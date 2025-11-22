# Nginx：深度内幕与实战指南

Nginx（发音为 "engine-x"）是世界上最流行的 Web 服务器、反向代理和负载均衡器。自 2004 年 Igor Sysoev 发布以来，它从根本上改变了互联网的运作方式，推动了 Web 架构从 90 年代笨重的“每连接一进程”模型向现代轻量级、异步、事件驱动的并发模型转变。

本指南是一份详尽的资源，专为需要从运维者和架构师角度深入理解 Nginx 的工程师设计。它涵盖了内部机制、高级配置、内核级调优、安全加固以及围绕 OpenResty 和 Lua 的生态系统。

## 架构与内部原理

要掌握 Nginx，首先必须理解它如何与操作系统交互。Nginx 的设计初衷就是以极小且可预测的内存占用处理数万个并发连接。

### 事件驱动模型

传统的 Web 服务器（如 Prefork 模式下的 Apache）为每个传入连接生成一个新的进程或线程。即使连接处于空闲状态（例如，用户网络缓慢，或 Keep-Alive 连接等待新请求），这也会占用内存和 CPU 资源。随着并发量的增长，上下文切换和内存消耗的开销成为瓶颈。这就是著名的 C10K 问题。

Nginx 使用**异步、非阻塞、事件驱动的架构**。

1.  **Worker 进程**：Nginx 通常运行固定数量的 Worker 进程（通常等于 CPU 核心数）。
2.  **事件循环**：每个 Worker 运行一个无限循环来处理事件。
3.  **多路复用**：它使用先进的操作系统机制（Linux 上的 `epoll`，FreeBSD/macOS 上的 `kqueue`，Windows 上的 `IOCP`）同时监控数千个文件描述符（网络套接字、文件）。

当网络数据包到达时，操作系统会唤醒 Worker。Worker 读取数据，进行处理（例如解析 HTTP），构建响应，将其写入缓冲区，并立即返回事件循环以处理下一个连接。如果可以避免，它不会等待网络或磁盘。

### 进程角色

*   **Master 进程（主进程）**：主管。它读取配置，绑定端口，并管理 Worker 进程。它处理信号（SIGHUP, SIGTERM）以执行热重载和二进制升级，而不会断开连接。Master 进程以 `root` 身份运行（以绑定 80/443 端口），但 Worker 进程会降权运行。
*   **Worker 进程（工作进程）**：主力。它们处理网络 I/O，读取/写入磁盘内容，并与上游服务器通信。它们以非特权用户身份运行（例如 `nginx` 或 `www-data`）。
*   **Cache Manager（缓存管理器）**：定期检查磁盘缓存以修剪过期项目或强制执行大小限制。
*   **Cache Loader（缓存加载器）**：在启动时短暂运行，将缓存元数据加载到共享内存中。

### 连接处理状态机

Nginx 实现了一个复杂的状态机。对于 HTTP：

1.  **ACCEPT**：接受新的 TCP 连接。
2.  **READ_HEADER**：数据被读入缓冲区。HTTP 状态机解析请求行和头部。
3.  **PROCESS**：Nginx 确定配置块（`server`，`location`）并处理请求（重写，认证，访问控制）。
4.  **CONTENT**：Nginx 生成响应（静态文件，上游代理，FastCGI）。
5.  **WRITE**：响应被写入套接字。
6.  **KEEPALIVE**：如果客户端请求保持连接，连接保持打开状态，但请求状态被清除，等待下一个请求。

### 阻塞操作与线程池

事件循环的阿喀琉斯之踵是**阻塞 I/O**。如果 Worker 阻塞（例如，从繁忙的机械硬盘读取文件），它将停止处理分配给它的*所有*数千个其他连接。

为了缓解这个问题，Nginx 引入了 **线程池 (Thread Pools)**（版本 1.7.11+）。
*   像 `sendfile()` 这样的操作通常是非阻塞的（它们触发 DMA 传输）。
*   但是，对不在页面缓存（Page Cache）中的文件进行 `read()` 可能会阻塞。
*   通过 `aio threads`，Nginx 可以将这些阻塞的磁盘操作卸载到单独的线程池中，允许主事件循环继续处理网络数据包。

```nginx
# 启用线程池
main {
    thread_pool default threads=32 max_queue=65536;
}

http {
    aio threads=default;
}
```

### 共享内存

Nginx 使用共享内存进行 Worker 之间的进程间通信 (IPC)。这对于以下方面至关重要：
*   **SSL 会话缓存**：共享 TLS 会话参数以避免昂贵的握手。
*   **Limit Zones**：存储限流计数器（`limit_req`，`limit_conn`）。
*   **Upstream Zones**：共享后端服务器的状态（健康检查，负载均衡计数器）。
*   **缓存键**：存储有关缓存文件的元数据。

Slab 分配器管理这块内存，将其切片为页面和块，以有效地存储各种大小的数据而不会产生碎片。

## 安装与编译选项

虽然 `apt-get install nginx` 足以满足基本使用，但高性能环境通常需要自定义构建。

### 源码编译

从源码编译允许你：
1.  静态链接特定的库（OpenSSL, PCRE, zlib）以确保版本兼容性和性能（例如，链接 OpenSSL 3.0 或 BoringSSL）。
2.  包含第三方模块（例如 `ngx_brotli`, `headers-more`, `vts`）。
3.  移除未使用的模块以减少攻击面和二进制文件大小。
4.  应用源代码补丁。

**先决条件 (Debian/Ubuntu):**
```bash
sudo apt-get install build-essential libpcre3-dev libssl-dev zlib1g-dev libgd-dev libgeoip-dev
```

**编译步骤:**

```bash
# 1. 下载源码
wget https://nginx.org/download/nginx-1.25.3.tar.gz
tar -zxvf nginx-1.25.3.tar.gz

# 2. 下载第三方模块（可选）
git clone https://github.com/google/ngx_brotli.git
cd ngx_brotli && git submodule update --init && cd ..

# 3. 配置
cd nginx-1.25.3
./configure \
    --prefix=/etc/nginx \
    --sbin-path=/usr/sbin/nginx \
    --conf-path=/etc/nginx/nginx.conf \
    --pid-path=/var/run/nginx.pid \
    --error-log-path=/var/log/nginx/error.log \
    --http-log-path=/var/log/nginx/access.log \
    --user=nginx \
    --group=nginx \
    --with-threads \
    --with-file-aio \
    --with-http_ssl_module \
    --with-http_v2_module \
    --with-http_v3_module \
    --with-http_realip_module \
    --with-http_addition_module \
    --with-http_sub_module \
    --with-http_stub_status_module \
    --with-http_gunzip_module \
    --with-http_gzip_static_module \
    --with-http_auth_request_module \
    --with-stream \
    --with-stream_ssl_module \
    --with-stream_realip_module \
    --add-module=../ngx_brotli \
    --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic' \
    --with-ld-opt='-Wl,-z,relro -Wl,-z,now'

# 4. 编译并安装
make -j$(nproc)
sudo make install
```

### 动态模块

自 1.9.11 版本起，Nginx 支持动态模块。这允许你将模块编译为共享对象 (`.so`) 并在运行时通过配置中的 `load_module` 加载，类似于 Apache 的 `LoadModule`。这将 Nginx 核心升级周期与模块更新分离开来。

```nginx
# nginx.conf
load_module modules/ngx_http_image_filter_module.so;
load_module modules/ngx_http_geoip_module.so;
```

要将模块编译为动态模块，请在配置期间使用 `--add-dynamic-module=/path/to/module`。

## 配置哲学与语法

配置文件 (`nginx.conf`) 是**指令 (Directives)** 和 **上下文 (Contexts)** 的树状结构。理解解析逻辑对于避免错误配置至关重要。

### 上下文层级

1.  **Main Context**：全局范围。指令影响整个应用程序（`user`, `worker_processes`, `pid`, `error_log`）。
2.  **Events Context**：配置网络模型（`worker_connections`, `use epoll`, `multi_accept`）。
3.  **HTTP Context**：Web 服务器层。
    *   **Upstream Context**：定义后端服务器组。
    *   **Server Context**：定义虚拟主机（IP/端口/域名组合）。
        *   **Location Context**：基于 URI 定义路由。
            *   **Nested Location Context**：进一步的路由细化。
4.  **Stream Context**：第 4 层 (TCP/UDP) 代理层。
5.  **Mail Context**：POP3/IMAP/SMTP 代理层。

### 指令类型

1.  **简单指令**：名称和参数，以分号结尾。
    *   `root /var/www;`
    *   `listen 80;`
2.  **块指令**：名称和参数，以 `{ ... }` 结尾。
    *   `server { ... }`
    *   `location / { ... }`

### 继承规则

大多数指令是向下继承的。如果你在 `http` 块中定义 `root /var/www;`，所有 `server` 块都会继承它，除非它们覆盖了它。

**关键例外：数组指令**
可以指定多次的指令（如 `proxy_set_header`, `add_header`, `access_log`）通常**不会**与父值合并。如果你在 `server` 块中定义了 `add_header`，然后在 `location` 块中再次定义，`location` 块将完全*替换*父级的头信息。

```nginx
server {
    add_header X-Server-Level "True";
    
    location / {
        add_header X-Location-Level "True";
        # 结果：仅发送 X-Location-Level。X-Server-Level 丢失。
    }
}
```
要解决此问题，如果需要，你必须重新声明父头信息，或者调整配置结构以在适当的级别应用通用头信息。

### "If is Evil" 原则

Nginx 中的 `if` 指令技术上属于 `rewrite` 模块。它在请求处理的重写阶段评估条件。

**为什么它很危险？**
Nginx 配置是声明式的。`if` 引入了命令式逻辑，这与通过声明式指令（如 `proxy_pass` 或 `try_files`）配合得不好。在 `if` 块内，Nginx 创建了一个隐式的匿名 location。请求可能会被困在这个 location 中，外部定义的指令可能无法按预期应用。

**错误配置示例：**
```nginx
# 危险！
location / {
    if ($http_user_agent ~* "Android") {
        add_header X-Device "Mobile";
    }
    try_files $uri $uri/ /index.php;
}
```
在这种情况下，如果条件匹配，`try_files` 可能会被忽略，因为它继承自外部块，但 `if` 块改变了上下文执行流。

**`if` 的安全用法：**
1.  `return ...`（立即停止处理）。
2.  `rewrite ... last;`（更改 URI）。

**替代方案：`map` 指令**
对于基于条件的变量赋值，请在 `http` 上下文中使用 `map`。它是高效且声明式的。

```nginx
# 更好：使用 map
map $http_user_agent $is_mobile {
    default       0;
    "~*Android"   1;
    "~*iPhone"    1;
}

server {
    if ($is_mobile) {
        return 301 http://m.example.com$request_uri;
    }
}
```

### 变量

Nginx 变量（`$host`, `$uri`, `$remote_addr`）是**按需计算**的（惰性求值）。
*   **内置变量**：由核心模块提供。
*   **自定义变量**：通过 `set`, `map`, 或 `geo` 创建。

性能说明：过度使用 `set` 和正则捕获会增加 CPU 开销。`map` 通常更优，因为它仅在访问变量时才进行评估。

## HTTP 核心机制

### 服务器选择

Nginx 如何决定使用哪个 `server` 块？

1.  **Listen 指令**：Nginx 首先按 IP 地址和端口过滤。
2.  **Server Name**：如果多个服务器监听同一端口，它会根据 `server_name` 检查 `Host` 头。
    *   **精确匹配**：`www.example.com`
    *   **通配符开头**：`*.example.com`
    *   **通配符结尾**：`www.example.*`
    *   **正则**：`~^(?<user>.+)\.example\.net$`
3.  **Default Server**：如果没有找到匹配项，它使用 `listen` 指令中标记为 `default_server` 的服务器，或者配置文件中定义的第一个服务器。

```nginx
server {
    listen 80 default_server;
    server_name _;
    return 444; # 关闭连接且不响应 (安全)
}
```

### Location 匹配深度解析

`location` 指令匹配 **标准化 URI**（解码后，相对路径，无查询字符串）。

**优先级顺序（牢记）：**

1.  **`=` (精确匹配)**：`location = /favicon.ico`。如果匹配，立即停止。
2.  **`^~` (优先前缀)**：`location ^~ /images/`。如果最长匹配前缀是 `^~` location，停止正则搜索。
3.  **`~` (区分大小写正则)**：按定义顺序检查。第一个匹配项获胜。
4.  **`~*` (不区分大小写正则)**：按定义顺序检查。第一个匹配项获胜。
5.  **前缀匹配**：`location /`。如果没有正则匹配，使用之前找到的最长前缀匹配。

**示例：**
```nginx
location / {
    # 规则 A (前缀)
}
location = / {
    # 规则 B (精确)
}
location /documents/ {
    # 规则 C (前缀)
}
location ^~ /images/ {
    # 规则 D (优先前缀)
}
location ~* \.(gif|jpg)$ {
    # 规则 E (正则)
}
```
*   请求 `/`：匹配 A 和 B。B 是精确的，所以 **B** 获胜。
*   请求 `/index.html`：匹配 A。无正则匹配。**A** 获胜。
*   请求 `/documents/document.html`：匹配 A 和 C。C 更长。无正则匹配。**C** 获胜。
*   请求 `/images/logo.jpg`：匹配 A 和 D。D 是优先前缀。停止搜索正则。**D** 获胜。（注意：E 被忽略）。
*   请求 `/documents/logo.jpg`：匹配 A 和 C。检查正则。匹配 E。**E** 获胜。

### URL 重写 (Rewriting)

`rewrite` 指令在内部更改 URI。它也可以发出外部重定向 (301/302)。

*   `rewrite ^/old/(.*)$ /new/$1 last;`：更改 URI 为 `/new/...` 并使用新 URI 重新开始 location 搜索。
*   `rewrite ^/old/(.*)$ /new/$1 break;`：更改 URI 但停留在当前 location 块中。
*   `rewrite ^/old/(.*)$ /new/$1 redirect;`：返回 302 Found (临时)。
*   `rewrite ^/old/(.*)$ /new/$1 permanent;`：返回 301 Moved Permanently。

## 反向代理与负载均衡

Nginx 可以说是最流行的反向代理。它接受请求并通过 HTTP, FastCGI, uWSGI, SCGI 或 Memcached 将其转发到后端服务器（Upstreams）。

### 基本代理

```nginx
location /api/ {
    proxy_pass http://backend_server;
    
    # 必要的 Header
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    
    # 缓冲 (默认开启)
    # Nginx 在发送给客户端之前读取后端的整个响应。
    proxy_buffering on;
    proxy_buffers 16 4k;
    proxy_buffer_size 4k;
    
    # 超时设置
    proxy_connect_timeout 60s;
    proxy_send_timeout 60s;
    proxy_read_timeout 60s;
}
```

**关于 `proxy_pass` 尾部斜杠的说明：**
*   `proxy_pass http://backend/`：将匹配的 location 部分替换为 `/`。
    *   请求 `/api/users` -> 上游 `/users`。
*   `proxy_pass http://backend`：追加完整的 URI。
    *   请求 `/api/users` -> 上游 `/api/users`。
*   **陷阱**：如果你在 `location` 中使用正则，`proxy_pass` **不能**有 URI 部分。必须是 `proxy_pass http://backend;`。

### 负载均衡策略

使用 `upstream` 定义服务器池。

```nginx
upstream backend_cluster {
    # 1. 轮询 (Round Robin - 默认)
    server 10.0.0.1;
    server 10.0.0.2;

    # 2. 最少连接 (Least Connections - 推荐用于处理时间不一的请求)
    least_conn;
    
    # 3. IP 哈希 (Sticky Sessions based on Client IP)
    # ip_hash;
    
    # 4. 通用哈希 (一致性哈希)
    # hash $request_uri consistent;
    
    # 参数
    server 10.0.0.3 weight=3;        # 接收 3 倍流量
    server 10.0.0.4 max_fails=3 fail_timeout=30s; # 被动健康检查 / 熔断器
    server 10.0.0.5 backup;          # 仅当其他服务器宕机时使用
    server 10.0.0.6 down;            # 手动标记下线
}
```

### 主动健康检查

开源版本的 Nginx 仅支持**被动**健康检查 (`max_fails`)。它仅在请求失败*后*才将服务器标记为宕机。
Nginx Plus（商业版）支持**主动**健康检查（定期探测端点）。

但是，你可以在开源版本中使用第三方模块（如来自 Tengine 的 `ngx_http_upstream_check_module`）或使用 Lua 探测端点来实现主动检查。

### 与上游的 Keep-Alive

默认情况下，Nginx 在每个请求后关闭与后端的连接。这对于 TLS 后端或高吞吐量来说效率低下（端口耗尽）。

启用 Keep-Alive：
```nginx
upstream backend {
    server 10.0.0.1:8080;
    keepalive 32;  # 每个 Worker 保持打开的空闲连接数
}

server {
    location / {
        proxy_pass http://backend;
        proxy_http_version 1.1;       # Keep-Alive 需要 HTTP/1.1
        proxy_set_header Connection ""; # 清除 "close" 头
    }
}
```

## 内容缓存

Nginx 可以使用 `proxy_cache` 充当强大的内容分发网络 (CDN) 节点。它将上游响应缓存到磁盘。

### 配置缓存区

必须在 `http` 上下文中完成。

```nginx
http {
    proxy_cache_path /var/cache/nginx 
                     levels=1:2 
                     keys_zone=my_cache:10m 
                     max_size=10g 
                     inactive=60m 
                     use_temp_path=off;
}
```

*   `levels=1:2`：目录哈希结构（例如 `/c/29/b7f54...`）。防止一个目录中文件过多。
*   `keys_zone`：名称和共享内存大小（1MB 存储约 8000 个键）。
*   `max_size`：最大磁盘大小。Cache Manager 在满时删除最近最少使用的项目。
*   `inactive`：在此时间内未访问的内容将被删除，无论是否过期。
*   `use_temp_path=off`：直接写入缓存目录，避免不必要的文件复制/重命名（性能优化）。

### 启用缓存

```nginx
location / {
    proxy_cache my_cache;
    proxy_cache_key "$scheme$request_method$host$request_uri";
    
    # 缓存规则
    proxy_cache_valid 200 302 10m;
    proxy_cache_valid 404      1m;
    proxy_cache_valid any      5m;
    
    # 绕过缓存（例如，用于调试或 cookie）
    proxy_cache_bypass $http_x_refresh;
    
    # 处理错误 (陈旧缓存)
    # 如果后端宕机 (500/502/504)，即使过期也提供缓存内容。
    proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
    
    # 后台更新 (旧版本中实验性，新版本稳定)
    proxy_cache_background_update on;
    
    # 锁定 (防止缓存击穿/Dog-pile effect)
    # 只有一个请求去后端填充缓存；其他的等待。
    proxy_cache_lock on;
    proxy_cache_lock_timeout 5s;
    
    add_header X-Cache-Status $upstream_cache_status; # HIT/MISS/BYPASS/EXPIRED
}
```

### 清除缓存

Nginx Plus 支持 `proxy_cache_purge`。对于开源版本，你可以使用 `ngx_cache_purge` 模块，或者简单地 `rm` 文件（如果你能将键映射到文件路径）。

## 安全与加固

Nginx 默认是安全的，但生产环境需要主动加固。

### 1. 隐藏元数据
防止向扫描器泄露版本号。
```nginx
server_tokens off;
```
（注意：要完全删除 "Server: nginx" 头，你需要 `headers-more` 模块）。

### 2. TLS 最佳实践 (2024/2025 版)
使用 Mozilla 的 SSL 配置生成器。

```nginx
ssl_protocols TLSv1.2 TLSv1.3; # 丢弃 SSLv3, TLS 1.0, 1.1
ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
ssl_prefer_server_ciphers off;

# 会话恢复 (性能)
ssl_session_cache shared:SSL:10m;
ssl_session_timeout 1d;
ssl_session_tickets off; # 如果有多个服务器，手动轮换票据或使用 Redis

# HSTS (严格传输安全)
add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;

# OCSP Stapling (隐私与性能)
ssl_stapling on;
ssl_stapling_verify on;
ssl_trusted_certificate /etc/nginx/ssl/chain.pem;
resolver 1.1.1.1;
```

### 3. 安全头 (Security Headers)
注入头部以保护客户端。

```nginx
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "no-referrer-when-downgrade" always;
add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;
```

### 4. 限流 (DDoS 缓解)
Nginx 使用 **漏桶 (Leaky Bucket)** 算法。

**区域定义：**
```nginx
# 基于二进制 IP 地址，每秒 10 个请求
# 区域名称: api_limit, 大小: 10MB (~160k IPs)
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;
```

**应用：**
```nginx
location /api/ {
    limit_req zone=api_limit burst=20 nodelay;
    # burst=20: 允许 20 个请求的队列。
    # nodelay: 不要慢下来，如果队列满了直接立即拒绝 (503)。
    
    limit_req_status 429; # 返回 429 Too Many Requests 而不是 503
}
```

### 5. 连接限制
限制每个 IP 的并发连接数。

```nginx
limit_conn_zone $binary_remote_addr zone=conn_limit:10m;

server {
    limit_conn conn_limit 10; # 每个 IP 最多 10 个并发连接
}
```

### 6. 防止缓冲区溢出
限制客户端可以发送的数据大小，以防止内存耗尽攻击。

```nginx
client_body_buffer_size 16k;
client_header_buffer_size 1k;
client_max_body_size 8m;      # 最大上传大小
large_client_header_buffers 2 1k;
```

## 性能调优与内核优化

配置 Nginx 只是战斗的一半。你必须调整 Linux 内核以处理高并发。

### Nginx 配置

```nginx
worker_processes auto; # 每个核心一个
worker_cpu_affinity auto; # 绑定到核心

# 增加限制
worker_rlimit_nofile 65535; 

events {
    worker_connections 65535; # 每个 Worker 的最大连接数
    multi_accept on; # 每次唤醒尽可能接受更多连接
    use epoll;
}

http {
    # 文件 I/O
    sendfile on;
    tcp_nopush on; # 在一个数据包中发送头部 + 文件开始 (仅限 Linux)
    tcp_nodelay on; # 禁用 Nagle 算法（Keep-alive 低延迟）
    
    # Keepalive
    keepalive_timeout 65;
    keepalive_requests 10000; 
    
    # 打开文件缓存 (元数据缓存)
    open_file_cache max=10000 inactive=20s;
    open_file_cache_valid 30s;
    open_file_cache_min_uses 2;
    open_file_cache_errors on;
    
    # 压缩 (Gzip)
    gzip on;
    gzip_comp_level 5; # 平衡 CPU 和大小 (1-9)
    gzip_min_length 256;
    gzip_proxied any;
    gzip_vary on;
    gzip_types
        application/atom+xml
        application/javascript
        application/json
        application/ld+json
        application/manifest+json
        application/rss+xml
        application/vnd.geo+json
        application/vnd.ms-fontobject
        application/x-font-ttf
        application/x-web-app-manifest+json
        application/xhtml+xml
        application/xml
        font/opentype
        image/bmp
        image/svg+xml
        image/x-icon
        text/cache-manifest
        text/css
        text/plain
        text/vcard
        text/vnd.rim.location.xloc
        text/vtt
        text/x-component
        text/x-cross-domain-policy;
        
    # Brotli (如果安装了模块 - 比 Gzip 更好)
    # brotli on;
    # brotli_comp_level 6;
    # brotli_types ...;
}
```

### Linux 内核调优 (`/etc/sysctl.conf`)
Nginx 打开的套接字数量不能超过内核允许的数量。

```ini
# 最大打开文件数 (系统级)
fs.file-max = 2097152

# 传入连接积压队列 (Syn Queue)
# 如果已满，客户端会收到 "Connection refused" 或超时
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535

# 临时端口 (增加向外代理连接的范围)
net.ipv4.ip_local_port_range = 1024 65535

# Time Wait 复用 (对于作为客户端连接上游的 Nginx 是安全的)
net.ipv4.tcp_tw_reuse = 1

# 减少连接在 FIN-WAIT-2 状态的时间
net.ipv4.tcp_fin_timeout = 15

# TCP 窗口缩放 (对于高带宽延迟积网络)
net.ipv4.tcp_window_scaling = 1

# 拥塞控制 (推荐新内核使用 BBR)
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
```

应用更改：`sysctl -p`。

此外，验证 `ulimit -n` 对运行 nginx 的用户是否足够高。编辑 `/etc/security/limits.conf`：
```
nginx soft nofile 65535
nginx hard nofile 65535
```

## 可观测性与调试

### 详细日志
默认的日志格式通常不足以调试性能问题。

```nginx
log_format perf '$remote_addr - $remote_user [$time_local] '
                '"$request" $status $body_bytes_sent '
                '"$http_referer" "$http_user_agent" '
                'rt=$request_time urt=$upstream_response_time '
                'uct=$upstream_connect_time uht=$upstream_header_time '
                'pipe=$pipe';

access_log /var/log/nginx/access.log perf buffer=32k flush=5s;
```
*   `rt`: 总请求时间 (Nginx + 上游)。
*   `urt`: 上游响应时间 (后端处理时间)。
*   `uct`: 建立 TCP 连接到后端的时间。
*   `buffer=32k flush=5s`: 通过缓冲日志写入减少磁盘 IOPS。

### 用于 ELK/Splunk 的 JSON 日志
结构化日志对于现代可观测性栈至关重要。

```nginx
log_format json_analytics escape=json
  '{'
    '"time_local": "$time_local",'
    '"remote_addr": "$remote_addr",'
    '"remote_user": "$remote_user",'
    '"request": "$request",'
    '"status": "$status",'
    '"body_bytes_sent": "$body_bytes_sent",'
    '"request_time": "$request_time",'
    '"upstream_response_time": "$upstream_response_time",'
    '"http_referrer": "$http_referer",'
    '"http_user_agent": "$http_user_agent",'
    '"request_id": "$request_id"'
  '}';
```

### 调试日志
要准确跟踪 Nginx 如何处理请求（头部解析，重写规则逻辑），请启用调试日志。**警告：产生海量日志。**

1.  确保 Nginx 编译时带有 `--with-debug`。
2.  配置：`error_log /var/log/nginx/debug.log debug;`
3.  或者，仅为特定 IP 启用以在不淹没日志的情况下调试生产问题：
    ```nginx
    events {
        debug_connection 192.168.1.1;
        debug_connection 10.0.0.0/24;
    }
    ```

### Stub Status
`ngx_http_stub_status_module` 提供实时指标。

```nginx
location /metrics {
    stub_status;
    allow 127.0.0.1;
    deny all;
}
```
输出解释：
*   `Active connections`: 打开的连接（包括空闲）。
*   `accepts`: 总接受连接数。
*   `handled`: 总处理连接数 (接受数 - 失败数)。
*   `requests`: 总请求数 (处理数 * 请求/连接)。
*   `Reading`: Nginx 正在读取请求头。
*   `Writing`: Nginx 正在将响应写入客户端。
*   `Waiting`: Keep-alive 连接 (Active - (Reading + Writing))。

要获取更详细的指标（如每个上游的统计信息），请使用 `ngx_vts_module` (Virtual Host Traffic Status) 编译。

## 使用 Lua 扩展 Nginx (OpenResty)

标准 Nginx 配置是声明式的，逻辑有限。**OpenResty** 将 **LuaJIT** 集成到 Nginx 中，允许你以高性能编写请求处理脚本。这使 Nginx 变成了功能齐全的 Web 应用服务器。

### 架构
OpenResty 挂钩到 Nginx 的阶段。
*   `init_by_lua`: Master 进程启动。
*   `init_worker_by_lua`: Worker 进程启动。
*   `ssl_certificate_by_lua`: 动态 SSL 证书加载。
*   `access_by_lua`: 认证和访问控制。
*   `content_by_lua`: 生成内容。
*   `header_filter_by_lua`: 修改响应头。
*   `body_filter_by_lua`: 修改响应体。
*   `log_by_lua`: 自定义日志。

### Cosockets (协程套接字)
OpenResty 的魔力在于 **Cosockets** (Coroutine Sockets)。它们允许 Lua 代码以**非阻塞**方式执行网络 I/O（连接到 Redis, MySQL, HTTP API）。Nginx 在 I/O 开始时挂起 Lua 协程，并在数据到达时恢复它，所有这些都不会阻塞 Worker 进程。

### 常见用例
1.  **复杂路由**：基于 Redis 键查找或 JWT 声明进行路由。
2.  **认证**：在到达后端之前直接在 Nginx 中验证 OAuth2/JWT 令牌。
3.  **动态上游**：无需重载即可更改后端目标。
4.  **WAF**：构建自定义防火墙规则（例如 Lua-Resty-WAF）。
5.  **API 聚合**：调用多个微服务并聚合结果。

### 示例：基于 Redis 的动态白名单

```nginx
http {
    lua_shared_dict cache 10m;
    
    server {
        location /protected {
            access_by_lua_block {
                local redis = require "resty.redis"
                local red = redis:new()
                red:set_timeout(1000)
                
                local ok, err = red:connect("127.0.0.1", 6379)
                if not ok then
                    ngx.log(ngx.ERR, "failed to connect to redis: ", err)
                    return ngx.exit(500)
                end
                
                local ip = ngx.var.remote_addr
                local is_allowed, err = red:sismember("allowed_ips", ip)
                
                -- Put connection back to pool
                red:set_keepalive(10000, 100)
                
                if is_allowed == 1 then
                    return -- Allow access
                else
                    ngx.status = 403
                    ngx.say("Access Denied")
                    return ngx.exit(403)
                end
            }
            
            proxy_pass http://backend;
        }
    }
}
```

## 从 Apache 迁移

从 Apache 迁移到 Nginx 需要转变思维，从“每个目录的 .htaccess”转向“集中式服务器块”。

### 主要区别

| 特性 | Apache | Nginx |
| :--- | :--- | :--- |
| **架构** | 基于进程/线程 | 事件驱动 |
| **静态文件** | 好 | 极好 (Sendfile/Async I/O) |
| **配置** | 分布式 (`.htaccess`) | 集中式 (`nginx.conf`) |
| **PHP** | `mod_php` (嵌入式) | `php-fpm` (Socket 上的 FastCGI) |
| **重写** | `mod_rewrite` (复杂) | `rewrite` / `return` (更简单) |
| **模块** | 可加载 `.so` | 动态 `.so` (自 1.9.11 起) |

### 重写规则转换

**Apache:**
```apache
RewriteEngine On
RewriteRule ^/user/([0-9]+)$ /profile.php?id=$1 [L]
```

**Nginx:**
```nginx
location ~ ^/user/([0-9]+)$ {
    # 方法 1: Rewrite
    rewrite ^/user/([0-9]+)$ /profile.php?id=$1 break;
    
    # 方法 2: FastCGI (性能更好)
    fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME /var/www/profile.php;
    fastcgi_param QUERY_STRING id=$1;
}
```

### 替换 .htaccess
Nginx 不支持 `.htaccess`。这是一个特性，而不是缺陷，因为扫描目录中的 `.htaccess` 文件会产生严重的 I/O 惩罚。你必须将这些规则迁移到主配置的 `server` 块中。

**Wordpress Nginx 配置（经典示例）：**
Nginx 使用 `try_files` 代替 Wordpress 复杂的 Apache 重写规则。

```nginx
location / {
    # 尝试查找文件，然后是目录，然后回退到 index.php
    try_files $uri $uri/ /index.php?$args;
}

location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    fastcgi_pass unix:/run/php/php8.1-fpm.sock;
}
```

## 故障排除常见错误

### 502 Bad Gateway
*   **原因**：Nginx 无法连接到上游（后端）。
*   **检查**：后端是否在运行？端口是否正确？防火墙问题？
*   **日志**：检查 `error.log` 中的 "connect() failed"。
*   **Socket 权限**：如果使用 Unix Sockets，检查 `nginx` 用户是否对 socket 文件有读写权限。

### 504 Gateway Timeout
*   **原因**：后端接受了连接但响应时间过长。
*   **修复**：优化后端代码（DB 查询）。增加 `proxy_read_timeout`。
    ```nginx
    proxy_read_timeout 300s;
    ```

### 413 Request Entity Too Large
*   **原因**：上传大小超过限制。
*   **修复**：增加 `client_max_body_size`。

### "Worker connections are not enough"
*   **原因**：高流量超过了 `worker_connections` 限制。
*   **修复**：在 `events` 块中增加 `worker_connections`（并检查 `ulimit`）。

### "Too many open files"
*   **原因**：达到系统或进程文件描述符限制。
*   **修复**：在 nginx.conf 中设置 `worker_rlimit_nofile` 以及 `ulimit -n` / `/etc/security/limits.conf`。

## 参考与资源

*   [Nginx 官方文档](http://nginx.org/en/docs/)
*   [Nginx Wiki](https://wiki.nginx.org/Main)
*   [OpenResty](https://openresty.org/en/)
*   [Mozilla SSL 配置生成器](https://ssl-config.mozilla.org/)
*   [Cloudflare 的 Nginx 调优博客文章](https://blog.cloudflare.com/tag/nginx/) - 内核调优深度文章。
*   [Agentzh 的 Nginx 教程](https://openresty.org/en/nginx-tutorials.html) - 由 OpenResty 作者撰写。

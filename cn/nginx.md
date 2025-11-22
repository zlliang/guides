# Nginx：深度内幕与实战指南

Nginx（发音为 "engine-x"）是世界上最流行的 Web 服务器、反向代理和负载均衡器。自 2004 年 Igor Sysoev 发布以来，它从根本上改变了互联网的运作方式，推动了 Web 架构从 90 年代笨重的“每连接一进程”模型向现代轻量级、异步、事件驱动的并发模型转变。

本指南是一份详尽的资源，专为需要从运维者和架构师角度深入理解 Nginx 的工程师设计。它涵盖了内部机制、高级配置、内核级调优、安全加固以及围绕 OpenResty 和 Lua 的生态系统。

## 架构与内部原理

要掌握 Nginx，首先必须理解它如何与操作系统交互。

### 事件驱动模型

传统的 Web 服务器（如 Prefork 模式下的 Apache）为每个传入连接生成一个新的进程或线程。即使连接处于空闲状态（例如，用户网络缓慢，或 Keep-Alive 连接等待新请求），这也会占用内存和 CPU 资源。

Nginx 使用**异步、非阻塞、事件驱动的架构**。

1.  **Worker 进程**：Nginx 通常运行固定数量的 Worker 进程（通常等于 CPU 核心数）。
2.  **事件循环**：每个 Worker 运行一个无限循环来处理事件。
3.  **多路复用**：它使用先进的操作系统机制（Linux 上的 `epoll`，FreeBSD/macOS 上的 `kqueue`，Windows 上的 `IOCP`）同时监控数千个文件描述符（网络套接字、文件）。

当网络数据包到达时，操作系统会唤醒 Worker。Worker 读取数据，进行处理（例如解析 HTTP），构建响应，将其写入缓冲区，并立即返回事件循环以处理下一个连接。如果可以避免，它不会等待网络或磁盘。

### 进程角色

*   **Master 进程（主进程）**：主管。它读取配置，绑定端口，并管理 Worker 进程。它处理信号（SIGHUP, SIGTERM）以执行热重载和二进制升级，而不会断开连接。
*   **Worker 进程（工作进程）**：主力。它们处理网络 I/O，读取/写入磁盘内容，并与上游服务器通信。它们以非特权用户身份运行（例如 `nginx`）。
*   **Cache Manager / Loader（缓存管理器/加载器）**：专门的进程，用于管理磁盘上的内容缓存（修剪过期项目，将元数据加载到内存中）。

### 阻塞操作与线程池

事件循环的阿喀琉斯之踵是**阻塞 I/O**。如果 Worker 阻塞（例如，从繁忙的机械硬盘读取文件），它将停止处理分配给它的*所有*数千个其他连接。

为了缓解这个问题，Nginx 引入了 **线程池 (Thread Pools)**（版本 1.7.11+）。
*   像 `sendfile()` 这样的操作通常是非阻塞的。
*   但是，对不在页面缓存（Page Cache）中的文件进行 `read()` 可能会阻塞。
*   通过 `aio threads`，Nginx 可以将这些阻塞的磁盘操作卸载到单独的线程池中，允许主事件循环继续处理网络数据包。

## 安装与编译选项

虽然 `apt-get install nginx` 足以满足基本使用，但高性能环境通常需要自定义构建。

### 源码编译

从源码编译允许你：
1.  静态链接特定的库（OpenSSL, PCRE, zlib）以确保版本兼容性和性能。
2.  包含第三方模块（例如 `ngx_brotli`, `headers-more`, `vts`）。
3.  移除未使用的模块以减少攻击面和二进制文件大小。

**先决条件 (Debian/Ubuntu):**
```bash
sudo apt-get install build-essential libpcre3-dev libssl-dev zlib1g-dev
```

**编译步骤:**
```bash
wget https://nginx.org/download/nginx-1.25.3.tar.gz
tar -zxvf nginx-1.25.3.tar.gz
cd nginx-1.25.3

# 配置常用优化标志
./configure \
    --prefix=/etc/nginx \
    --sbin-path=/usr/sbin/nginx \
    --conf-path=/etc/nginx/nginx.conf \
    --pid-path=/var/run/nginx.pid \
    --error-log-path=/var/log/nginx/error.log \
    --http-log-path=/var/log/nginx/access.log \
    --user=nginx \
    --group=nginx \
    --with-http_ssl_module \
    --with-http_v2_module \
    --with-http_realip_module \
    --with-http_stub_status_module \
    --with-threads \
    --with-file-aio \
    --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic'

make
sudo make install
```

### 动态模块

自 1.9.11 版本起，Nginx 支持动态模块。这允许你将模块编译为共享对象 (`.so`) 并在运行时通过配置中的 `load_module` 加载，类似于 Apache 的 `LoadModule`。

```nginx
# nginx.conf
load_module modules/ngx_http_image_filter_module.so;
```

## 配置哲学与语法

配置文件是**指令 (Directives)** 和 **上下文 (Contexts)** 的树状结构。

### 上下文层级

1.  **Main Context**：全局范围。指令影响整个应用程序（`user`, `worker_processes`, `pid`）。
2.  **Events Context**：配置网络模型（`worker_connections`, `use epoll`）。
3.  **HTTP Context**：Web 服务器层。
    *   **Upstream Context**：定义后端服务器组。
    *   **Server Context**：定义虚拟主机（IP/端口/域名组合）。
        *   **Location Context**：基于 URI 定义路由。
            *   **Nested Location Context**：进一步的路由细化。

### 继承规则

大多数指令是向下继承的。如果你在 `http` 块中定义 `root /var/www;`，所有 `server` 块都会继承它，除非它们覆盖了它。

*   **Action Directives（动作指令）**：触发出作的指令（如 `rewrite`, `return`, 或 `proxy_pass`）严格遵循继承；它们通常会停止执行或更改阶段。
*   **Array Directives（数组指令）**：接受多个值的指令（如 `proxy_set_header` 或 `add_header`）通常**不会**合并。如果你在 `server` 块中定义了 `add_header`，然后在 `location` 块中再次定义，`location` 块将完全*替换*父级的头信息。这是一个常见的陷阱。

### "If is Evil" 原则

Nginx 中的 `if` 指令技术上属于 `rewrite` 模块。它在请求处理的重写阶段评估条件。

**为什么它很危险？**
Nginx 配置是声明式的。`if` 引入了命令式逻辑，这与通过声明式指令（如 `proxy_pass` 或 `try_files`）配合得不好。

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
在这种情况下，如果条件匹配，`try_files` 可能会被忽略或表现出乎意料，因为 `if` 创建了一个隐式的嵌套 location 块。

**`if` 的安全用法：**
1.  `return ...`（立即停止处理）。
2.  `rewrite ... last;`（更改 URI）。

对于基于条件的变量赋值，请改用 `map` 指令。

```nginx
# 更好：使用 map
map $http_user_agent $is_mobile {
    default       0;
    "~*Android"   1;
    "~*iPhone"    1;
}
```

## HTTP 核心机制

### 服务器选择

Nginx 如何决定使用哪个 `server` 块？

1.  **Listen 指令**：Nginx 首先按 IP 地址和端口过滤。
2.  **Server Name**：如果多个服务器监听同一端口，它会根据 `server_name` 检查 `Host` 头。
    *   精确匹配：`www.example.com`
    *   通配符开头：`*.example.com`
    *   通配符结尾：`www.example.*`
    *   正则：`~^(?<user>.+)\.example\.net$`
3.  **Default Server**：如果没有找到匹配项，它使用 `listen` 指令中标记为 `default_server` 的服务器，或者定义的第一个服务器。

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

### 负载均衡策略

使用 `upstream` 定义服务器池。

```nginx
upstream backend_cluster {
    # 1. 轮询 (Round Robin - 默认)
    server 10.0.0.1;
    server 10.0.0.2;

    # 2. 最少连接 (Least Connections - 推荐用于处理时间不一的请求)
    least_conn;
    
    # 3. IP 哈希 (Sticky Sessions)
    # ip_hash;
    
    # 4. 通用哈希 (一致性哈希)
    # hash $request_uri consistent;
    
    # 参数
    server 10.0.0.3 weight=3;        # 接收 3 倍流量
    server 10.0.0.4 max_fails=3 fail_timeout=30s; # 熔断器
    server 10.0.0.5 backup;          # 仅当其他服务器宕机时使用
}
```

### 与上游的 Keep-Alive

默认情况下，Nginx 在每个请求后关闭与后端的连接。这对于 TLS 后端或高吞吐量来说效率低下。

启用 Keep-Alive：
```nginx
upstream backend {
    server 10.0.0.1:8080;
    keepalive 32;  # 保持打开的空闲连接数
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

Nginx 可以使用 `proxy_cache` 充当强大的内容分发网络 (CDN) 节点。

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
*   `inactive`：在此时间内未访问的内容将被删除。
*   `use_temp_path=off`：直接写入缓存目录，避免不必要的文件复制。

### 启用缓存

```nginx
location / {
    proxy_cache my_cache;
    proxy_cache_key "$scheme$request_method$host$request_uri";
    
    # 缓存规则
    proxy_cache_valid 200 302 10m;
    proxy_cache_valid 404      1m;
    
    # 处理错误 (陈旧缓存)
    # 如果后端宕机 (500/502/504)，即使过期也提供缓存内容。
    proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
    
    # 锁定 (防止缓存击穿/Dog-pile effect)
    # 只有一个请求去后端填充缓存；其他的等待。
    proxy_cache_lock on;
    
    add_header X-Cache-Status $upstream_cache_status; # HIT/MISS/BYPASS
}
```

## 安全与加固

Nginx 默认是安全的，但生产环境需要主动加固。

### 1. 隐藏元数据
防止向扫描器泄露版本号。
```nginx
server_tokens off;
```
（注意：要完全删除 "Server: nginx" 头，你需要 `headers-more` 模块或重新编译源码）。

### 2. TLS 最佳实践 (2024/2025 版)
使用 Mozilla 的 SSL 配置生成器（中间配置文件通常最好）。

```nginx
ssl_protocols TLSv1.2 TLSv1.3; # 丢弃 SSLv3, TLS 1.0, 1.1
ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
ssl_prefer_server_ciphers off;

# 会话恢复 (性能)
ssl_session_cache shared:SSL:10m;
ssl_session_timeout 1d;
ssl_session_tickets off;

# HSTS (严格传输安全)
add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
```

### 3. 限流 (DDoS 缓解)
Nginx 使用 **漏桶 (Leaky Bucket)** 算法。

**区域定义：**
```nginx
# 基于二进制 IP 地址，每秒 10 个请求
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;
```

**应用：**
```nginx
location /api/ {
    limit_req zone=api_limit burst=20 nodelay;
    # burst=20: 允许 20 个请求的队列。
    # nodelay: 不要慢下来，如果队列满了直接拒绝。
}
```

### 4. 防止缓冲区溢出
限制客户端可以发送的数据大小。

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
worker_processes auto;
worker_rlimit_nofile 65535; # 进程的文件描述符限制

events {
    worker_connections 65535;
    multi_accept on; # 每次唤醒尽可能接受更多连接
    use epoll;
}

http {
    # 文件 I/O
    sendfile on;
    tcp_nopush on; # 在一个数据包中发送头部 + 文件开始
    tcp_nodelay on; # 禁用 Nagle 算法（降低延迟）
    
    # Keepalive
    keepalive_timeout 65;
    keepalive_requests 1000; # 关闭前复用连接 1000 次
    
    # 压缩
    gzip on;
    gzip_comp_level 5; # 平衡 CPU 和大小 (1-9)
    gzip_min_length 256;
    gzip_types application/json application/javascript text/css text/xml;
}
```

### Linux 内核调优 (`/etc/sysctl.conf`)
Nginx 打开的套接字数量不能超过内核允许的数量。

```ini
# 最大打开文件数 (系统级)
fs.file-max = 2097152

# 传入连接积压队列 (Syn Queue)
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535

# 临时端口 (增加向外代理连接的范围)
net.ipv4.ip_local_port_range = 1024 65535

# Time Wait 复用 (对于作为客户端连接上游的 Nginx 是安全的)
net.ipv4.tcp_tw_reuse = 1

# Fin-Wait-2 超时
net.ipv4.tcp_fin_timeout = 15
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
                'uct=$upstream_connect_time uht=$upstream_header_time';

access_log /var/log/nginx/access.log perf;
```
*   `rt`: 总请求时间 (Nginx + 上游)。
*   `urt`: 上游响应时间 (后端处理时间)。
*   `uct`: 建立 TCP 连接到后端的时间。

### 调试日志
要准确跟踪 Nginx 如何处理请求（头部解析，重写规则逻辑），请启用调试日志。**警告：产生海量日志。**

1.  确保 Nginx 编译时带有 `--with-debug`。
2.  配置：`error_log /var/log/nginx/debug.log debug;`
3.  或者，仅为特定 IP 启用：
    ```nginx
    events {
        debug_connection 192.168.1.1;
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

## 使用 Lua 扩展 Nginx (OpenResty)

标准 Nginx 配置是声明式的，逻辑有限。**OpenResty** 将 **LuaJIT** 集成到 Nginx 中，允许你以高性能编写请求处理脚本。

### 常见用例
1.  **复杂路由**：基于 Redis 键查找进行路由。
2.  **认证**：在到达后端之前直接在 Nginx 中验证 OAuth2/JWT 令牌。
3.  **动态上游**：无需重载即可更改后端目标。
4.  **WAF**：构建自定义防火墙规则（例如 Lua-Resty-WAF）。

### 示例：Lua 中的 Hello World
```nginx
location /lua {
    default_type 'text/plain';
    content_by_lua_block {
        ngx.say("Hello from Lua!")
        ngx.log(ngx.ERR, "This is a log entry")
    }
}
```

### 示例：非阻塞数据库访问
OpenResty 中的 Lua 脚本在代码风格上是同步的，但在执行上是异步的（协程）。

```lua
content_by_lua_block {
    local redis = require "resty.redis"
    local red = redis:new()
    
    red:set_timeout(1000)
    local ok, err = red:connect("127.0.0.1", 6379)
    
    if not ok then
        ngx.say("failed to connect: ", err)
        return
    end
    
    local res, err = red:get("my_key")
    if not res then
        ngx.say("failed to get: ", err)
        return
    end
    
    if res == ngx.null then
        ngx.say("key not found")
        return
    end
    
    ngx.say("Key value: ", res)
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

### 重写规则转换

**Apache:**
```apache
RewriteEngine On
RewriteRule ^/user/([0-9]+)$ /profile.php?id=$1 [L]
```

**Nginx:**
```nginx
location ~ ^/user/([0-9]+)$ {
    rewrite ^/user/([0-9]+)$ /profile.php?id=$1 break;
    # 或者更好，简单地代理给 PHP
    fastcgi_pass ...;
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

## 总结

Nginx 是一个强大的工具，深度理解它会带来丰厚的回报。通过掌握事件循环、Location 搜索优先级以及静态服务和上游代理之间的区别，你可以构建出弹性、安全且极快的系统。

记住：
1.  **性能**来自非阻塞 I/O 和内核调优。
2.  **安全**来自最小化攻击面（仅编译所需内容）和严格的 HTTP 头。
3.  **可靠性**来自理解 Master 和 Worker 进程的区别，并避免在配置中进行阻塞操作。

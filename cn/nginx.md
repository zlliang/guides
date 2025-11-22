# Nginx：完整指南

Nginx（发音为 "engine-x"）是一个高性能的 HTTP 服务器、反向代理和邮件代理服务器。它由 Igor Sysoev 于 2004 年创建，旨在解决 C10K 问题（即同时处理 10,000 个并发连接），现已成为现代互联网的骨干。它以稳定性、丰富的功能集、简单的配置和低资源消耗而闻名。

本指南是对 Nginx 的深度剖析，涵盖从基本操作到内部架构、高级配置模式以及大规模工程团队使用的性能调优技巧。

## 快速参考

| 任务 | 命令 / 语法 | 说明 |
| :--- | :--- | :--- |
| **启动** | `nginx` | 启动服务器。 |
| **停止** | `nginx -s stop` | 快速关闭。 |
| **退出** | `nginx -s quit` | 优雅关闭（等待工作进程完成当前请求）。 |
| **重载** | `nginx -s reload` | 重新加载配置而不中断连接。 |
| **测试配置** | `nginx -t` | 检查配置文件语法的有效性。 |
| **版本** | `nginx -v` / `nginx -V` | 显示版本 / 显示构建详细信息（模块、参数）。 |
| **信号** | `kill -HUP <pid>` | 向主进程发送信号（等同于 reload）。 |

### 关键配置块

```nginx
user www-data;              # User 上下文（运行用户）
worker_processes auto;      # Main 上下文（全局设置）

events {                    # Events 上下文
    worker_connections 1024;
}

http {                      # HTTP 上下文
    server {                # Server 上下文（虚拟主机）
        listen 80;
        server_name example.com;

        location / {        # Location 上下文（URI 路由）
            root /var/www/html;
        }
    }
}
```

## 历史与 C10K 问题

在 Nginx 出现之前，占主导地位的 Web 服务器是 Apache HTTP Server。Apache 使用“每个连接一个进程”或“每个连接一个线程”的模型。虽然这种模型很稳健，但它难以扩展：随着并发连接数的增加，内存占用和上下文切换的开销呈线性增长，最终导致服务器饱和。这就是著名的 **C10K 问题**。

Igor Sysoev 设计 Nginx 时采用了根本不同的架构：**事件驱动和异步非阻塞**。Nginx 不会为每个客户端创建一个新进程，而是使用操作系统提供的高效事件多路复用机制（如 Linux 上的 `epoll` 或 BSD 上的 `kqueue`）在单个进程中处理数千个连接。

## 架构与内部原理

理解 Nginx 的底层工作原理对于调优和故障排查至关重要。

### Master-Worker 模型

Nginx 使用多进程架构，但与 Apache 的 Prefork 模块方式不同。

1.  **Master 进程（主进程）：**
    *   以 `root` 身份运行。
    *   读取并验证配置。
    *   绑定特权端口（如 80/443）。
    *   生成和管理 Worker 进程。
    *   处理信号（HUP, TERM, QUIT）。
    *   **关键点**：它本身不处理网络流量。

2.  **Worker 进程（工作进程）：**
    *   以非特权用户身份运行（如 `nginx` 或 `www-data`）。
    *   处理实际的网络连接。
    *   通常每个 CPU 核心配置一个 Worker（`worker_processes auto`）。
    *   使用共享内存区域进行进程间通信（共享缓存、限流计数器、会话持久化）。

### 事件循环 (The Event Loop)

每个 Worker 进程运行一个非阻塞的事件循环。

1.  **Accept**：Worker 接受来自监听套接字的新连接。
2.  **Multiplexing**：它使用 `epoll` (Linux)、`kqueue` (BSD) 或 `IOCP` (Windows) 同时监控数千个文件描述符（套接字）。
3.  **Events**：当套接字准备好读取或写入时，操作系统会通知 Worker。
4.  **Processing**：Worker 执行操作（例如读取请求头）并处理下一个事件。如果操作会阻塞（如从磁盘读取文件或等待上游响应），Nginx 会使用异步 I/O 或线程池（用于文件 I/O）来避免停滞循环。

这种架构解释了为什么 Nginx 如此之快：**上下文切换被最小化**。单个 Worker 停留在 CPU 上，并在紧凑的循环中处理许多请求。

### 连接处理阶段 (Phases)

当 Nginx 处理 HTTP 请求时，它会经过 11 个阶段。理解这些阶段有助于编写模块和调试复杂的配置。

1.  **Post-Read**：读取头部后立即进行的初始处理（例如 RealIP 模块）。
2.  **Server Rewrite**：在选择 server 块之前的 URL 修改。
3.  **Find Config**：Nginx 匹配 `location` 块。
4.  **Rewrite**：URI 操作（`rewrite` 指令）。
5.  **Post-Rewrite**：检查重写是否需要跳转到新位置。
6.  **Pre-Access**：限流（`limit_req`, `limit_conn`）。
7.  **Access**：认证和访问控制（`auth_basic`, `allow`/`deny`）。
8.  **Post-Access**：最终检查。
9.  **Try Files**：检查静态文件是否存在（`try_files`）。
10. **Content**：生成响应（提供文件、代理到上游、执行 FastCGI）。
11. **Log**：写入访问日志。

## 安装与生态系统

### 源码编译 vs. 软件包

*   **OS 软件包 (`apt`, `yum`)**：稳定，易于更新，与 systemd 集成。通常版本较旧。
*   **官方 Nginx 仓库**：更新更及时。
*   **源码编译**：如果需要以下情况则必须编译：
    *   特定的编译器优化。
    *   移除未使用的模块以减小体积。
    *   添加标准构建中未包含的第三方模块。
    *   修补源代码。

### 模块

Nginx 是模块化的，但与 Apache 不同，传统上模块是*静态编译*到二进制文件中的。从 1.9.11 版本开始，Nginx 支持 **动态模块**（`.so` 文件），可以通过 `load_module` 在运行时加载。

### OpenResty

Nginx 的一个流行发行版，捆绑了 **LuaJIT** 编译器和许多 Nginx 模块。它允许开发人员直接在 Nginx 内部使用 Lua 脚本编写高性能的 Web 应用程序和逻辑（路由、验证、复杂缓存）。

## 配置哲学

配置文件 (`nginx.conf`) 是分层级的。

### 上下文 (Contexts)

设置从上层上下文继承到下层上下文。
*   `main`：全局设置（工作进程、PID 文件）。
*   `events`：连接处理模型。
*   `http`：Web 服务器层。
    *   `server`：定义虚拟主机（域名/IP）。
        *   `location`：定义如何处理特定的 URI。

### Location 匹配逻辑

这是最常见的困惑来源。Nginx **不会**简单地匹配它找到的第一个块。它遵循特定的优先级顺序：

1.  **精确匹配 (`=`)**：`location = /path`（如果匹配，停止搜索）。
2.  **优先前缀 (`^~`)**：`location ^~ /static/`（如果匹配，停止正则搜索）。
3.  **正则表达式 (`~` 或 `~*`)**：按文件中出现的顺序检查。**第一个**匹配的正则获胜。
4.  **标准前缀**：`location /path`。如果没有正则匹配，则**最长**的匹配前缀获胜。

**"If 是邪恶的" (If is Evil) 规则：**
不鼓励且危险的做法是在 `location` 块内使用 `if`。这可能会导致不可预测的行为，因为 `if` 实际上是 Rewrite 阶段的一部分，发生在内容生成之前。
*   **坏习惯**：`if ($host = 'example.com') { ... }`
*   **好习惯**：使用单独的 `server` 块或 `map` 指令。

## 核心用例

### 1. 静态文件服务器

Nginx 非常擅长提供静态资源（图片、CSS、JS）。

*   **`sendfile on;`**：允许内核直接将数据从文件系统缓存复制到网络套接字，完全绕过用户空间（零拷贝）。
*   **`tcp_nopush on;`**：与 sendfile 一起使用，发送完整的数据包，减少网络开销。
*   **缓存头**：`expires max;`, `add_header Cache-Control "public";`。

### 2. 反向代理与负载均衡

Nginx 位于应用服务器（Node.js, Python, Go, PHP-FPM）之前。

```nginx
upstream backend_cluster {
    least_conn;                 # 负载均衡算法
    server 10.0.0.1:8080 weight=3;
    server 10.0.0.2:8080;
    keepalive 32;               # 保持与上游的连接打开
}

server {
    location /api/ {
        proxy_pass http://backend_cluster;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

**负载均衡算法：**
*   `round-robin`（默认）：顺序分配。
*   `least_conn`：发送给活动连接数最少的服务器。
*   `ip_hash`：基于客户端 IP 的会话粘性（Sticky Session）。
*   `hash $key`：通用哈希（例如按 URL）。

### 3. API 网关

Nginx 广泛用作 API 网关，在请求到达微服务之前处理横切关注点。
*   **限流**：`limit_req_zone`（漏桶算法）。
*   **认证**：`auth_request`（将认证委托给子请求），JWT 验证（商业版或第三方模块）。
*   **请求/响应转换**：修改头部或 JSON 体。

## 性能调优

为了在高流量硬件上充分利用 Nginx：

### Worker 配置
*   `worker_processes auto;`：每个 CPU 核心绑定一个 Worker。
*   `worker_cpu_affinity`：显式将 Worker 绑定到核心以防止缓存抖动（现代版本中 `auto` 会自动处理）。
*   `worker_rlimit_nofile`：增加 Worker 进程打开文件描述符的限制（必须 > `worker_connections`）。

### 连接限制
*   `events { worker_connections 10240; }`：每个 Worker 的最大同时连接数。
*   **最大客户端数公式**：`(worker_processes * worker_connections) / (若是反向代理 ? 2 : 1)`。
    *   除以 2 是因为代理需要保持一个与客户端的连接和一个与上游的连接。

### Keepalive 优化
建立 TCP 连接是昂贵的（三次握手）。
*   **客户端侧**：`keepalive_timeout 65;`（复用连接处理多个请求）。
*   **上游侧**：
    ```nginx
    upstream backend {
        server ...;
        keepalive 16;
    }
    ```
    并且在 location 中：`proxy_http_version 1.1; proxy_set_header Connection "";`

### 缓冲 (Buffering)
*   `client_body_buffer_size`：如果请求体适合缓冲区，则在内存中处理。如果不适合，则写入临时文件（慢）。
*   `proxy_buffer_size` / `proxy_buffers`：影响 Nginx 如何从上游读取响应。关闭缓冲（`proxy_buffering off;`）会立即将响应流式传输给客户端，但如果客户端很慢，则会占用 Worker 进程。

## 与替代方案的比较

### Nginx vs. Apache
*   **架构**：Apache (prefork/worker) 创建线程/进程。Nginx 是事件驱动的。
*   **性能**：Nginx 在静态文件和高并发方面更快。Apache 在动态内容处理方面具有可比性（因为瓶颈通常在应用运行时，如 PHP）。
*   **灵活性**：Apache 支持 `.htaccess` 文件进行动态的每目录配置。Nginx 不支持（出于性能原因），需要重新加载主配置。

### Nginx vs. HAProxy
*   **侧重**：HAProxy 是专用的 TCP/HTTP 负载均衡器。它缺乏 Web 服务器功能（如提供静态文件）。
*   **可观测性**：HAProxy 通常提供更细粒度的统计数据和开箱即用的详细健康检查。
*   **用例**：常一起使用。HAProxy 作为边缘入口（L4/L7 均衡），指向 Nginx 层进行终止和路由。

### Nginx vs. Envoy
*   **时代**：Envoy 诞生于容器/Kubernetes 时代（Service Mesh）。
*   **动态性**：Envoy 专为快速、API 驱动的配置更新（xDS 协议）而设计，无需重新加载进程。Nginx 需要重新加载配置（SIGHUP）或使用商业版 Nginx Plus API。
*   **复杂性**：Envoy 手动配置较复杂；通常由控制平面（如 Istio）管理。

## 调试与可观测性

### 日志
`log_format` 指令允许捕获特定变量。
```nginx
log_format detailed '$remote_addr - $remote_user [$time_local] '
                    '"$request" $status $body_bytes_sent '
                    '"$http_referer" "$http_user_agent" '
                    '$request_time $upstream_response_time $pipe';
```
*   `$request_time`：处理请求花费的总时间。
*   `$upstream_response_time`：等待后端花费的时间。

### Stub Status
一个简单的模块 (`ngx_http_stub_status_module`) 提供基本指标。
```nginx
location /nginx_status {
    stub_status;
    allow 127.0.0.1;
    deny all;
}
```
输出：活动连接数、接受数、处理的请求数。

### 调试
如果 Nginx 行为异常：
1.  检查日志：`/var/log/nginx/error.log`。
2.  启用调试日志（需要 `--with-debug` 编译标志）：`error_log /path/to/log debug;`。
3.  **配置转储**：`nginx -T` 将完整的解析配置转储到 stdout（有助于查看合并的包含文件）。

## 安全最佳实践

1.  **隐藏版本**：`server_tokens off;` 防止泄露 Nginx 版本号。
2.  **TLS 加固**：
    *   禁用 SSLv3/TLS1.0/TLS1.1。
    *   使用强加密套件 (Cipher Suites)。
    *   启用 HSTS (`Strict-Transport-Security`)。
3.  **缓冲区溢出保护**：
    *   `client_body_buffer_size`
    *   `client_header_buffer_size`
    *   `client_max_body_size`（限制上传大小）
    *   `large_client_header_buffers`
    限制这些可以防止攻击者通过大量请求耗尽内存。
4.  **以非特权用户运行**：永远不要以 root 身份运行 Worker。

## 参考与资源

*   [Nginx 官方文档](http://nginx.org/en/docs/)
*   [Nginx Wiki](https://wiki.nginx.org/Main)
*   [C10K 问题](http://www.kegel.com/c10k.html)
*   [OpenResty](https://openresty.org/en/)
*   [Mozilla SSL 配置生成器](https://ssl-config.mozilla.org/) - 安全 TLS 配置的必备工具。

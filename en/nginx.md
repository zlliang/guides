# Nginx: The Complete Guide

Nginx (pronounced "engine-x") is a high-performance HTTP server, reverse proxy, and mail proxy server. Created by Igor Sysoev in 2004 to solve the C10K problem (handling 10,000 concurrent connections), it has grown to become the backbone of the modern internet. It is renowned for its stability, rich feature set, simple configuration, and low resource consumption.

This guide serves as a deep dive into Nginx, moving from basic operation to internal architecture, advanced configuration patterns, and performance tuning used by high-scale engineering teams.

## Quick Reference

| Task | Command / Syntax | Description |
| :--- | :--- | :--- |
| **Start** | `nginx` | Start the server. |
| **Stop** | `nginx -s stop` | Fast shutdown. |
| **Quit** | `nginx -s quit` | Graceful shutdown (waits for workers). |
| **Reload** | `nginx -s reload` | Reload config without dropping connections. |
| **Test Config** | `nginx -t` | Check configuration syntax validity. |
| **Version** | `nginx -v` / `nginx -V` | Show version / Show build details. |
| **Signal** | `kill -HUP <pid>` | Send signal to master process (same as reload). |

### Key Configuration Blocks

```nginx
user www-data;              # User context
worker_processes auto;      # Main context

events {                    # Events context
    worker_connections 1024;
}

http {                      # HTTP context
    server {                # Server context (Virtual Host)
        listen 80;
        server_name example.com;

        location / {        # Location context (URI routing)
            root /var/www/html;
        }
    }
}
```

## History and The C10K Problem

Before Nginx, the dominant web server was Apache HTTP Server. Apache used a process-per-connection or thread-per-connection model. While robust, this model struggled to scale: as concurrent connections increased, the memory footprint and context-switching overhead grew linearly, eventually saturating the server. This was known as the **C10K problem**.

Igor Sysoev designed Nginx with a fundamentally different architecture: **event-driven and asynchronous**. Instead of creating a new process for every client, Nginx handles thousands of connections within a single process using efficient event multiplexing mechanisms provided by the OS (like `epoll` on Linux or `kqueue` on BSD).

## Architecture and Internals

Understanding how Nginx works under the hood is critical for tuning and debugging.

### The Master-Worker Model

Nginx uses a multi-process architecture, but not in the way Apache's Prefork module does.

1.  **Master Process:**
    *   Runs as `root`.
    *   Reads and validates configuration.
    *   Binds to privileged ports (like 80/443).
    *   Spawns and manages worker processes.
    *   Handles signals (HUP, TERM, QUIT).
    *   **Crucially**, it does not handle network traffic itself.

2.  **Worker Processes:**
    *   Run as a non-privileged user (e.g., `nginx` or `www-data`).
    *   Handle actual network connections.
    *   There are typically one worker per CPU core (`worker_processes auto`).
    *   Shared memory zones are used for inter-process communication (shared cache, rate limit counters, session persistence).

### The Event Loop

Each worker process runs a non-blocking event loop.

1.  **Accept:** The worker accepts new connections from the listening socket.
2.  **Multiplexing:** It uses `epoll` (Linux), `kqueue` (BSD), or `IOCP` (Windows) to monitor thousands of file descriptors (sockets) simultaneously.
3.  **Events:** When a socket is ready to read or write, the OS notifies the worker.
4.  **Processing:** The worker performs the operation (e.g., read request header) and moves to the next event. If an operation would block (like reading a file from disk or waiting for an upstream response), Nginx uses asynchronous I/O or thread pools (for file I/O) to avoid stalling the loop.

This architecture explains why Nginx is fast: **Context switches are minimized.** A single worker stays on the CPU and processes many requests in a tight loop.

### Connection Processing Phases

When Nginx handles an HTTP request, it passes through a series of 11 phases. Understanding these phases helps in writing modules and debugging complex configs.

1.  **Post-Read:** Initial processing immediately after reading the header (e.g., RealIP module).
2.  **Server Rewrite:** URL modification before server block selection.
3.  **Find Config:** Nginx matches the `location` block.
4.  **Rewrite:** URI manipulation (`rewrite` directive).
5.  **Post-Rewrite:** Checks if the rewrite requires a jump to a new location.
6.  **Pre-Access:** Rate limiting (`limit_req`, `limit_conn`).
7.  **Access:** Authentication and Access Control (`auth_basic`, `allow`/`deny`).
8.  **Post-Access:** Final checks.
9.  **Try Files:** Checking for static file existence (`try_files`).
10. **Content:** Generating the response (serving a file, proxying to upstream, executing FastCGI).
11. **Log:** Writing the access log.

## Installation and Ecosystem

### Source vs. Package

*   **OS Packages (`apt`, `yum`):** Stable, easy to update, integrated with systemd. Usually older versions.
*   **Official Nginx Repos:** More up-to-date.
*   **Compiling from Source:** Required if you need:
    *   Specific compiler optimizations.
    *   To remove unused modules for a smaller footprint.
    *   To add 3rd-party modules not included in standard builds.
    *   To patch the source code.

### Modules

Nginx is modular, but unlike Apache, modules were traditionally compiled *statically* into the binary. Since version 1.9.11, Nginx supports **Dynamic Modules** (`.so` files), which can be loaded at runtime via `load_module`.

### OpenResty

A popular distribution of Nginx that bundles the **LuaJIT** compiler and many Nginx modules. It allows developers to write high-performance web applications and logic (routing, validation, complex caching) directly within Nginx using Lua scripts.

## Configuration Philosophy

The configuration file (`nginx.conf`) is hierarchical.

### Contexts

Settings inherit from upper contexts to lower contexts.
*   `main`: Global settings (worker processes, PID file).
*   `events`: Connection processing models.
*   `http`: The web server layer.
    *   `server`: Defines a virtual host (domain/IP).
        *   `location`: Defines how to process specific URIs.

### Location Matching Logic

This is the most common source of confusion. Nginx does **not** simply match the first block it finds. It follows a specific priority order:

1.  **Exact Match (`=`)**: `location = /path` (Stops searching if matched).
2.  **Preferential Prefix (`^~`)**: `location ^~ /static/` (Stops regex search if matched).
3.  **Regular Expressions (`~` or `~*`)**: Checked in order of appearance in the file. The *first* matching regex wins.
4.  **Standard Prefix**: `location /path`. The *longest* matching prefix wins if no regex matches.

**The "If is Evil" Rule:**
Using `if` inside a `location` block is discouraged and dangerous. It can cause unpredictable behavior because `if` is actually part of the Rewrite phase, which happens before content generation.
*   **Bad:** `if ($host = 'example.com') { ... }`
*   **Good:** Use separate `server` blocks or `map` directives.

## Core Use Cases

### 1. Static File Server

Nginx excels at serving static assets (images, CSS, JS).

*   **`sendfile on;`**: Allows the kernel to copy data directly from the file system cache to the network socket, bypassing user space entirely (Zero Copy).
*   **`tcp_nopush on;`**: Used with sendfile to send full packets, reducing network overhead.
*   **Caching headers:** `expires max;`, `add_header Cache-Control "public";`.

### 2. Reverse Proxy & Load Balancer

Nginx sits in front of application servers (Node.js, Python, Go, PHP-FPM).

```nginx
upstream backend_cluster {
    least_conn;                 # Load balancing algorithm
    server 10.0.0.1:8080 weight=3;
    server 10.0.0.2:8080;
    keepalive 32;               # Keep connections to upstream open
}

server {
    location /api/ {
        proxy_pass http://backend_cluster;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

**Load Balancing Algorithms:**
*   `round-robin` (Default): Distributes sequentially.
*   `least_conn`: Sends to the server with fewest active connections.
*   `ip_hash`: Sticky session based on client IP.
*   `hash $key`: Generic hash (e.g., by URL).

### 3. API Gateway

Nginx is widely used as an API Gateway to handle cross-cutting concerns before requests hit microservices.
*   **Rate Limiting:** `limit_req_zone` (Leaky Bucket algorithm).
*   **Authentication:** `auth_request` (Delegates auth to a sub-request), JWT validation (commercial or 3rd party).
*   **Request/Response Transformation:** Modifying headers or JSON bodies.

## Performance Tuning

To get the most out of Nginx on high-traffic hardware:

### Worker Configuration
*   `worker_processes auto;`: Binds one worker per CPU core.
*   `worker_cpu_affinity`: Explicitly binds workers to cores to prevent cache thrashing (done automatically by `auto` in modern versions).
*   `worker_rlimit_nofile`: Increases the limit of open file descriptors for the worker process (must be > `worker_connections`).

### Connection Limits
*   `events { worker_connections 10240; }`: The max simultaneous connections per worker.
*   **Max Clients Formula:** `(worker_processes * worker_connections) / (Is_Reverse_Proxy ? 2 : 1)`.
    *   Divided by 2 because a proxy holds one connection to the client and one to the upstream.

### Keepalive Optimization
Creating TCP connections is expensive (3-way handshake).
*   **Client side:** `keepalive_timeout 65;` (Reuses connection for multiple requests).
*   **Upstream side:**
    ```nginx
    upstream backend {
        server ...;
        keepalive 16;
    }
    ```
    And in location: `proxy_http_version 1.1; proxy_set_header Connection "";`

### Buffering
*   `client_body_buffer_size`: If the request body fits in the buffer, it's handled in memory. If not, it's written to a temp file (slow).
*   `proxy_buffer_size` / `proxy_buffers`: Affects how Nginx reads responses from upstream. Turning buffering *off* (`proxy_buffering off;`) streams the response to the client immediately but ties up the worker process if the client is slow.

## Comparison with Alternatives

### Nginx vs. Apache
*   **Architecture:** Apache (prefork/worker) creates threads/processes. Nginx is event-driven.
*   **Performance:** Nginx is faster for static files and high concurrency. Apache is comparable for dynamic content processing (since the bottleneck is the app runtime, e.g., PHP).
*   **Flexibility:** Apache supports `.htaccess` files for dynamic, per-directory configuration. Nginx does not (for performance reasons), requiring a central reload.

### Nginx vs. HAProxy
*   **Focus:** HAProxy is a dedicated TCP/HTTP load balancer. It lacks web server features (serving static files).
*   **Observability:** HAProxy generally offers more granular stats and detailed health checks out-of-the-box.
*   **Use Case:** Often used together. HAProxy as the edge ingress (L4/L7 balancing), pointing to Nginx layers for termination and routing.

### Nginx vs. Envoy
*   **Era:** Envoy was born in the container/Kubernetes era (Service Mesh).
*   **Dynamism:** Envoy is designed for rapid, API-driven configuration updates (xDS protocol) without reloading the process. Nginx requires a config reload (SIGHUP) or the commercial Nginx Plus API.
*   **Complexity:** Envoy is more complex to configure manually; it is usually managed by a control plane (like Istio).

## Debugging and Observability

### Logging
The `log_format` directive allows capturing specific variables.
```nginx
log_format detailed '$remote_addr - $remote_user [$time_local] '
                    '"$request" $status $body_bytes_sent '
                    '"$http_referer" "$http_user_agent" '
                    '$request_time $upstream_response_time $pipe';
```
*   `$request_time`: Total time spent processing the request.
*   `$upstream_response_time`: Time spent waiting for the backend.

### Stub Status
A simple module (`ngx_http_stub_status_module`) provides basic metrics.
```nginx
location /nginx_status {
    stub_status;
    allow 127.0.0.1;
    deny all;
}
```
Output: Active connections, accepts, handled requests.

### Debugging
If Nginx behaves strangely:
1.  Check logs: `/var/log/nginx/error.log`.
2.  Enable debug logging (requires `--with-debug` compile flag): `error_log /path/to/log debug;`.
3.  **Config Dump:** `nginx -T` dumps the full resolved configuration to stdout (useful to see include files merged).

## Security Best Practices

1.  **Hide Version:** `server_tokens off;` prevents leaking the Nginx version number.
2.  **TLS Hardening:**
    *   Disable SSLv3/TLS1.0/TLS1.1.
    *   Use strong ciphers.
    *   Enable HSTS (`Strict-Transport-Security`).
3.  **Buffer Overflow Protection:**
    *   `client_body_buffer_size`
    *   `client_header_buffer_size`
    *   `client_max_body_size` (Limit upload size)
    *   `large_client_header_buffers`
    Limiting these prevents attackers from exhausting memory with massive requests.
4.  **Run as Non-Privileged User:** Never run workers as root.

## References & Resources

*   [Nginx Official Documentation](http://nginx.org/en/docs/)
*   [Nginx Wiki](https://wiki.nginx.org/Main)
*   [The C10K Problem](http://www.kegel.com/c10k.html)
*   [OpenResty](https://openresty.org/en/)
*   [Mozilla SSL Configuration Generator](https://ssl-config.mozilla.org/) - Essential for secure TLS configs.

# Nginx: The Comprehensive Insider's Guide

Nginx (pronounced "engine-x") is the world's most popular web server, reverse proxy, and load balancer. Since its release in 2004 by Igor Sysoev, it has fundamentally changed how the internet operates, enabling the transition from the heavy, process-per-connection model of the 90s to the lightweight, asynchronous, event-driven concurrency model of the modern web.

This guide is an exhaustive resource for engineers who need to understand Nginx not just as a user, but as an operator and architect. It covers internal mechanics, advanced configuration, kernel-level tuning, security hardening, and the ecosystem around OpenResty and Lua.

## Architecture and Internals

To master Nginx, one must first understand how it interacts with the operating system.

### The Event-Driven Model

Traditional web servers like Apache (in Prefork mode) spawn a new process or thread for every incoming connection. This blocks memory and CPU resources even when the connection is idle (e.g., a user on a slow network, or a Keep-Alive connection waiting for a new request).

Nginx uses an **asynchronous, non-blocking, event-driven architecture**.

1.  **The Worker Process:** Nginx typically runs a fixed number of worker processes (usually equal to the number of CPU cores).
2.  **The Event Loop:** Each worker runs an infinite loop that processes events.
3.  **Multiplexing:** It uses advanced OS mechanisms (`epoll` on Linux, `kqueue` on FreeBSD/macOS, `IOCP` on Windows) to monitor thousands of file descriptors (network sockets, files) simultaneously.

When a network packet arrives, the OS wakes up the worker. The worker reads the data, processes it (e.g., parses HTTP), constructs a response, writes it to the buffer, and immediately returns to the event loop to handle the next connection. It does not wait for the network or disk if it can avoid it.

### Process Roles

*   **Master Process:** The supervisor. It reads configuration, binds to ports, and manages worker processes. It handles signals (SIGHUP, SIGTERM) to perform hot reloads and binary upgrades without dropping connections.
*   **Worker Processes:** The workhorses. They handle network I/O, read/write content to disk, and communicate with upstream servers. They run as an unprivileged user (e.g., `nginx`).
*   **Cache Manager / Loader:** specialized processes that manage the on-disk content cache (pruning expired items, loading metadata into memory).

### Blocking Operations and Thread Pools

The Achilles' heel of an event loop is **blocking I/O**. If a worker blocks (e.g., reading a file from a spinning disk that is busy), it stops processing *all* other thousands of connections assigned to it.

To mitigate this, Nginx introduced **Thread Pools** (version 1.7.11+).
*   Operations like `sendfile()` are usually non-blocking.
*   However, `read()` on a file that isn't in the Page Cache can block.
*   With `aio threads`, Nginx can offload these blocking disk operations to a separate thread pool, allowing the main event loop to continue processing network packets.

## Installation and Build Options

While `apt-get install nginx` is sufficient for basic usage, high-performance environments often require a custom build.

### Building from Source

Compiling from source allows you to:
1.  Statically link specific libraries (OpenSSL, PCRE, zlib) to ensure version compatibility and performance.
2.  Include third-party modules (e.g., `ngx_brotli`, `headers-more`, `vts`).
3.  Remove unused modules to reduce attack surface and binary size.

**Prerequisites (Debian/Ubuntu):**
```bash
sudo apt-get install build-essential libpcre3-dev libssl-dev zlib1g-dev
```

**Compilation Steps:**
```bash
wget https://nginx.org/download/nginx-1.25.3.tar.gz
tar -zxvf nginx-1.25.3.tar.gz
cd nginx-1.25.3

# Configure with common optimization flags
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

### Dynamic Modules

Since version 1.9.11, Nginx supports dynamic modules. This allows you to compile a module as a shared object (`.so`) and load it at runtime via `load_module` in the configuration, similar to Apache's `LoadModule`.

```nginx
# nginx.conf
load_module modules/ngx_http_image_filter_module.so;
```

## Configuration Philosophy and Syntax

The configuration file is a tree-like structure of **Directives** and **Contexts**.

### The Context Hierarchy

1.  **Main Context:** The global scope. Directives here affect the entire application (`user`, `worker_processes`, `pid`).
2.  **Events Context:** Configures the network model (`worker_connections`, `use epoll`).
3.  **HTTP Context:** The web server layer.
    *   **Upstream Context:** Defines groups of backend servers.
    *   **Server Context:** Defines a Virtual Host (IP/Port/Domain combination).
        *   **Location Context:** Defines routing based on URI.
            *   **Nested Location Context:** Further routing refinement.

### Inheritance Rules

Most directives are inherited downwards. If you define `root /var/www;` in the `http` block, all `server` blocks inherit it unless they override it.

*   **Action Directives:** Directives that trigger an action (like `rewrite`, `return`, or `proxy_pass`) do strictly follow inheritance; they often stop execution or change the phase.
*   **Array Directives:** Directives that accept multiple values (like `proxy_set_header` or `add_header`) generally **do not** merge. If you define `add_header` in a `server` block and then again in a `location` block, the `location` block *replaces* the parent's headers entirely. This is a common pitfall.

### The "If is Evil" Principle

The `if` directive in Nginx is technically part of the `rewrite` module. It evaluates conditions at the rewrite phase of the request processing.

**Why is it dangerous?**
Nginx configuration is declarative. `if` introduces imperative logic that interacts poorly with other declarative directives (like `proxy_pass` or `try_files`).

**Example of Broken Config:**
```nginx
# DANGEROUS
location / {
    if ($http_user_agent ~* "Android") {
        add_header X-Device "Mobile";
    }
    try_files $uri $uri/ /index.php;
}
```
In this case, if the condition matches, `try_files` might be ignored or behave unexpectedly because `if` creates an implicit nested location block.

**Safe usage of `if`:**
1.  `return ...` (immediately stopping processing).
2.  `rewrite ... last;` (changing the URI).

For variable assignment based on conditions, use the `map` directive instead.

```nginx
# BETTER: Using map
map $http_user_agent $is_mobile {
    default       0;
    "~*Android"   1;
    "~*iPhone"    1;
}
```

## The HTTP Core

### Server Selection

How does Nginx decide which `server` block to use?

1.  **Listen Directive:** Nginx first filters by IP address and Port.
2.  **Server Name:** If multiple servers listen on the same port, it checks the `Host` header against `server_name`.
    *   Exact match: `www.example.com`
    *   Wildcard start: `*.example.com`
    *   Wildcard end: `www.example.*`
    *   Regex: `~^(?<user>.+)\.example\.net$`
3.  **Default Server:** If no match is found, it uses the server marked `default_server` in the `listen` directive, or the first one defined.

### Location Matching Deep Dive

The `location` directive matches the **Normalized URI** (decoded, relative path, no query string).

**Priority Order (Memorize This):**

1.  **`=` (Exact Match):** `location = /favicon.ico`. If matched, stop immediately.
2.  **`^~` (Preferential Prefix):** `location ^~ /images/`. If the longest matching prefix is a `^~` location, stop regex search.
3.  **`~` (Case-Sensitive Regex):** Checked in definition order. First match wins.
4.  **`~*` (Case-Insensitive Regex):** Checked in definition order. First match wins.
5.  **Prefix Match:** `location /`. If no regex matched, use the longest prefix match found earlier.

**Example:**
```nginx
location / {
    # Rule A (Prefix)
}
location = / {
    # Rule B (Exact)
}
location /documents/ {
    # Rule C (Prefix)
}
location ^~ /images/ {
    # Rule D (Preferential Prefix)
}
location ~* \.(gif|jpg)$ {
    # Rule E (Regex)
}
```
*   Request `/`: Matches A and B. B is exact, so **B** wins.
*   Request `/index.html`: Matches A. No regex matches. **A** wins.
*   Request `/documents/document.html`: Matches A and C. C is longer. No regex matches. **C** wins.
*   Request `/images/logo.jpg`: Matches A and D. D is preferential prefix. Stops searching regex. **D** wins. (Note: E is ignored).
*   Request `/documents/logo.jpg`: Matches A and C. Checks regex. Matches E. **E** wins.

## Reverse Proxying and Load Balancing

Nginx is arguably the most popular Reverse Proxy. It accepts requests and forwards them to backend servers (Upstreams) via HTTP, FastCGI, uWSGI, SCGI, or Memcached.

### Basic Proxying

```nginx
location /api/ {
    proxy_pass http://backend_server;
    
    # Essential Headers
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    
    # Timeouts
    proxy_connect_timeout 60s;
    proxy_send_timeout 60s;
    proxy_read_timeout 60s;
}
```

**Note on `proxy_pass` Trailing Slash:**
*   `proxy_pass http://backend/`: Replaces the matched location part with `/`.
    *   Request `/api/users` -> Upstream `/users`.
*   `proxy_pass http://backend`: Appends the full URI.
    *   Request `/api/users` -> Upstream `/api/users`.

### Load Balancing Strategies

Define a pool of servers using `upstream`.

```nginx
upstream backend_cluster {
    # 1. Round Robin (Default)
    server 10.0.0.1;
    server 10.0.0.2;

    # 2. Least Connections (Recommended for varying request times)
    least_conn;
    
    # 3. IP Hash (Sticky Sessions)
    # ip_hash;
    
    # 4. Generic Hash (Consistent Hashing)
    # hash $request_uri consistent;
    
    # Parameters
    server 10.0.0.3 weight=3;        # Receive 3x traffic
    server 10.0.0.4 max_fails=3 fail_timeout=30s; # Circuit breaker
    server 10.0.0.5 backup;          # Only used if others are down
}
```

### Keep-Alive to Upstream

By default, Nginx closes the connection to the backend after every request. This is inefficient for TLS backends or high throughput.

To enable Keep-Alive:
```nginx
upstream backend {
    server 10.0.0.1:8080;
    keepalive 32;  # Number of idle connections to keep open
}

server {
    location / {
        proxy_pass http://backend;
        proxy_http_version 1.1;       # Required for Keep-Alive
        proxy_set_header Connection ""; # Clear the "close" header
    }
}
```

## Content Caching

Nginx can act as a robust Content Delivery Network (CDN) node using `proxy_cache`.

### Configuring the Cache Zone

This must be done in the `http` context.

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

*   `levels=1:2`: Directory hashing structure (e.g., `/c/29/b7f54...`). Prevents too many files in one directory.
*   `keys_zone`: Name and shared memory size (1MB stores ~8000 keys).
*   `inactive`: Delete content not accessed within this time.
*   `use_temp_path=off`: Write directly to cache directory to avoid unnecessary file copying.

### Enabling Caching

```nginx
location / {
    proxy_cache my_cache;
    proxy_cache_key "$scheme$request_method$host$request_uri";
    
    # Caching Rules
    proxy_cache_valid 200 302 10m;
    proxy_cache_valid 404      1m;
    
    # Handling Errors (Stale Cache)
    # If backend is down (500/502/504), serve cached content even if expired.
    proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
    
    # Locking (Dog-pile effect prevention)
    # Only one request goes to backend to populate cache; others wait.
    proxy_cache_lock on;
    
    add_header X-Cache-Status $upstream_cache_status; # HIT/MISS/BYPASS
}
```

## Security and Hardening

Nginx is secure by default, but production environments require active hardening.

### 1. Hiding Metadata
Prevent leaking version numbers to scanners.
```nginx
server_tokens off;
```
(Note: To remove the "Server: nginx" header entirely, you need the `headers-more` module or recompile source).

### 2. TLS Best Practices (2024/2025 Edition)
Use Mozilla's SSL Configuration Generator (Intermediate profile is usually best).

```nginx
ssl_protocols TLSv1.2 TLSv1.3; # Drop SSLv3, TLS 1.0, 1.1
ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
ssl_prefer_server_ciphers off;

# Session Resumption (Performance)
ssl_session_cache shared:SSL:10m;
ssl_session_timeout 1d;
ssl_session_tickets off;

# HSTS (Strict Transport Security)
add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
```

### 3. Rate Limiting (DDoS Mitigation)
Nginx uses the **Leaky Bucket** algorithm.

**Zone Definition:**
```nginx
# 10 requests per second based on binary IP address
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;
```

**Application:**
```nginx
location /api/ {
    limit_req zone=api_limit burst=20 nodelay;
    # burst=20: Allow a queue of 20 requests.
    # nodelay: Don't slow them down, just reject if queue is full.
}
```

### 4. Preventing Buffer Overflows
Restrict the size of data clients can send.

```nginx
client_body_buffer_size 16k;
client_header_buffer_size 1k;
client_max_body_size 8m;      # Max upload size
large_client_header_buffers 2 1k;
```

## Performance Tuning and Kernel Optimization

Configuring Nginx is only half the battle. You must tune the Linux kernel to handle high concurrency.

### Nginx Configuration
```nginx
worker_processes auto;
worker_rlimit_nofile 65535; # File descriptors limit for the process

events {
    worker_connections 65535;
    multi_accept on; # Accept as many connections as possible per wake-up
    use epoll;
}

http {
    # File I/O
    sendfile on;
    tcp_nopush on; # Send headers + file start in one packet
    tcp_nodelay on; # Disable Nagle's algorithm (lower latency)
    
    # Keepalive
    keepalive_timeout 65;
    keepalive_requests 1000; # Reuse connection 1000 times before closing
    
    # Compression
    gzip on;
    gzip_comp_level 5; # Balance CPU and Size (1-9)
    gzip_min_length 256;
    gzip_types application/json application/javascript text/css text/xml;
}
```

### Linux Kernel Tuning (`/etc/sysctl.conf`)
Nginx cannot open more sockets than the kernel allows.

```ini
# Max open files (system wide)
fs.file-max = 2097152

# Backlog for incoming connections (Syn Queue)
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535

# Ephemeral Ports (Increase range for outgoing proxy connections)
net.ipv4.ip_local_port_range = 1024 65535

# Time Wait reuse (Safe for Nginx acting as client to upstream)
net.ipv4.tcp_tw_reuse = 1

# Timeout for Fin-Wait-2
net.ipv4.tcp_fin_timeout = 15
```

Apply with `sysctl -p`.

Also, verify `ulimit -n` is high enough for the user running nginx. Edit `/etc/security/limits.conf`:
```
nginx soft nofile 65535
nginx hard nofile 65535
```

## Observability and Debugging

### Detailed Logging
The default log format is often insufficient for debugging performance issues.

```nginx
log_format perf '$remote_addr - $remote_user [$time_local] '
                '"$request" $status $body_bytes_sent '
                '"$http_referer" "$http_user_agent" '
                'rt=$request_time urt=$upstream_response_time '
                'uct=$upstream_connect_time uht=$upstream_header_time';

access_log /var/log/nginx/access.log perf;
```
*   `rt`: Total Request Time (Nginx + Upstream).
*   `urt`: Upstream Response Time (Backend processing time).
*   `uct`: Time to establish TCP connection to backend.

### Debug Log
To trace exactly how Nginx processes a request (headers parsing, rewrite rules logic), enable debug logging. **Warning: Generates massive logs.**

1.  Ensure Nginx is compiled with `--with-debug`.
2.  Configure: `error_log /var/log/nginx/debug.log debug;`
3.  Or, enable only for specific IPs:
    ```nginx
    events {
        debug_connection 192.168.1.1;
    }
    ```

### Stub Status
The `ngx_http_stub_status_module` provides real-time metrics.

```nginx
location /metrics {
    stub_status;
    allow 127.0.0.1;
    deny all;
}
```
Output explanation:
*   `Active connections`: Open connections (including idle).
*   `accepts`: Total accepted connections.
*   `handled`: Total handled connections (accepts - failures).
*   `requests`: Total requests (handled * requests/conn).

## Extending Nginx with Lua (OpenResty)

Standard Nginx config is declarative and limited in logic. **OpenResty** integrates **LuaJIT** into Nginx, allowing you to script request handling with high performance.

### Common Use Cases
1.  **Complex Routing:** Route based on Redis key lookup.
2.  **Authentication:** Validate OAuth2/JWT tokens directly in Nginx before hitting the backend.
3.  **Dynamic Upstreams:** Change backend targets without reloading.
4.  **WAF:** Build custom firewall rules (e.g., Lua-Resty-WAF).

### Example: Hello World in Lua
```nginx
location /lua {
    default_type 'text/plain';
    content_by_lua_block {
        ngx.say("Hello from Lua!")
        ngx.log(ngx.ERR, "This is a log entry")
    }
}
```

### Example: Non-blocking Database Access
Lua scripts in OpenResty are synchronous in code style but asynchronous in execution (coroutines).

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

## Migration from Apache

Migrating from Apache to Nginx requires a mindset shift from ".htaccess per directory" to "Centralized Server Blocks".

### Key Differences

| Feature | Apache | Nginx |
| :--- | :--- | :--- |
| **Architecture** | Process/Thread based | Event-driven |
| **Static Files** | Good | Excellent (Sendfile/Async I/O) |
| **Configuration** | Distributed (`.htaccess`) | Centralized (`nginx.conf`) |
| **PHP** | `mod_php` (Embedded) | `php-fpm` (FastCGI over socket) |
| **Rewrites** | `mod_rewrite` (Complex) | `rewrite` / `return` (Simpler) |

### Rewrite Rule Translation

**Apache:**
```apache
RewriteEngine On
RewriteRule ^/user/([0-9]+)$ /profile.php?id=$1 [L]
```

**Nginx:**
```nginx
location ~ ^/user/([0-9]+)$ {
    rewrite ^/user/([0-9]+)$ /profile.php?id=$1 break;
    # Or better, simply proxy to PHP
    fastcgi_pass ...;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME /var/www/profile.php;
    fastcgi_param QUERY_STRING id=$1;
}
```

### Replacing .htaccess
Nginx does not support `.htaccess`. This is a feature, not a bug, as scanning directories for `.htaccess` files incurs significant I/O penalties. You must migrate these rules into the `server` block of your main config.

**Wordpress Nginx Config (The Classic Example):**
Instead of the complex Apache rewrite rules for Wordpress, Nginx uses `try_files`.

```nginx
location / {
    # Try finding the file, then the directory, then fall back to index.php
    try_files $uri $uri/ /index.php?$args;
}

location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    fastcgi_pass unix:/run/php/php8.1-fpm.sock;
}
```

## Summary

Nginx is a powerful tool that rewards deep understanding. By mastering the event loop, the location search priority, and the split between static serving and upstream proxying, you can architect systems that are resilient, secure, and incredibly fast.

Remember:
1.  **Performance** comes from non-blocking I/O and kernel tuning.
2.  **Security** comes from minimizing surface area (compiling only what you need) and strict HTTP headers.
3.  **Reliability** comes from understanding the distinction between the Master and Worker processes and avoiding blocking operations in the configuration.

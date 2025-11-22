# Nginx: The Comprehensive Insider's Guide

Nginx (pronounced "engine-x") is the world's most popular web server, reverse proxy, and load balancer. Since its release in 2004 by Igor Sysoev, it has fundamentally changed how the internet operates, enabling the transition from the heavy, process-per-connection model of the 90s to the lightweight, asynchronous, event-driven concurrency model of the modern web.

This guide is an exhaustive resource for engineers who need to understand Nginx not just as a user, but as an operator and architect. It covers internal mechanics, advanced configuration, kernel-level tuning, security hardening, and the ecosystem around OpenResty and Lua.

## Architecture and Internals

To master Nginx, one must first understand how it interacts with the operating system. Nginx is designed to handle thousands of concurrent connections with a small and predictable memory footprint.

### The Event-Driven Model

Traditional web servers like Apache (in Prefork mode) spawn a new process or thread for every incoming connection. This blocks memory and CPU resources even when the connection is idle (e.g., a user on a slow network, or a Keep-Alive connection waiting for a new request). As concurrency grows, the overhead of context switching and memory consumption becomes the bottleneck. This is known as the C10K problem.

Nginx uses an **asynchronous, non-blocking, event-driven architecture**.

1.  **The Worker Process:** Nginx typically runs a fixed number of worker processes (usually equal to the number of CPU cores).
2.  **The Event Loop:** Each worker runs an infinite loop that processes events.
3.  **Multiplexing:** It uses advanced OS mechanisms (`epoll` on Linux, `kqueue` on FreeBSD/macOS, `IOCP` on Windows) to monitor thousands of file descriptors (network sockets, files) simultaneously.

When a network packet arrives, the OS wakes up the worker. The worker reads the data, processes it (e.g., parses HTTP), constructs a response, writes it to the buffer, and immediately returns to the event loop to handle the next connection. It does not wait for the network or disk if it can avoid it.

### Process Roles

*   **Master Process:** The supervisor. It reads configuration, binds to ports, and manages worker processes. It handles signals (SIGHUP, SIGTERM) to perform hot reloads and binary upgrades without dropping connections. The master process runs as `root` (to bind to port 80/443) but drops privileges for workers.
*   **Worker Processes:** The workhorses. They handle network I/O, read/write content to disk, and communicate with upstream servers. They run as an unprivileged user (e.g., `nginx` or `www-data`).
*   **Cache Manager:** Periodically checks the on-disk cache to prune expired items or enforce size limits.
*   **Cache Loader:** Runs briefly at startup to load cache metadata into shared memory.

### Connection Processing State Machine

Nginx implements a complex state machine. For HTTP:

1.  **ACCEPT**: A new TCP connection is accepted.
2.  **READ_HEADER**: Data is read into a buffer. The HTTP state machine parses the request line and headers.
3.  **PROCESS**: Nginx determines the configuration block (`server`, `location`) and processes the request (rewrite, auth, access control).
4.  **CONTENT**: Nginx generates a response (static file, upstream proxy, FastCGI).
5.  **WRITE**: The response is written to the socket.
6.  **KEEPALIVE**: If the client requested keep-alive, the connection remains open but the request state is cleared, waiting for the next request.

### Blocking Operations and Thread Pools

The Achilles' heel of an event loop is **blocking I/O**. If a worker blocks (e.g., reading a file from a spinning disk that is busy), it stops processing *all* other thousands of connections assigned to it.

To mitigate this, Nginx introduced **Thread Pools** (version 1.7.11+).
*   Operations like `sendfile()` are usually non-blocking (they trigger DMA transfers).
*   However, `read()` on a file that isn't in the Page Cache can block.
*   With `aio threads`, Nginx can offload these blocking disk operations to a separate thread pool, allowing the main event loop to continue processing network packets.

```nginx
# Enabling Thread Pools
main {
    thread_pool default threads=32 max_queue=65536;
}

http {
    aio threads=default;
}
```

### Shared Memory

Nginx uses shared memory for inter-process communication (IPC) between workers. This is crucial for:
*   **SSL Session Cache:** Sharing TLS session parameters to avoid expensive handshakes.
*   **Limit Zones:** Storing counters for rate limiting (`limit_req`, `limit_conn`).
*   **Upstream Zones:** Sharing the state of backend servers (health checks, load balancing counters).
*   **Cache Keys:** Storing metadata about cached files.

The Slab Allocator manages this memory, slicing it into pages and chunks to efficiently store data of various sizes without fragmentation.

## Installation and Build Options

While `apt-get install nginx` is sufficient for basic usage, high-performance environments often require a custom build.

### Building from Source

Compiling from source allows you to:
1.  Statically link specific libraries (OpenSSL, PCRE, zlib) to ensure version compatibility and performance (e.g., linking against OpenSSL 3.0 or BoringSSL).
2.  Include third-party modules (e.g., `ngx_brotli`, `headers-more`, `vts`).
3.  Remove unused modules to reduce attack surface and binary size.
4.  Apply source code patches.

**Prerequisites (Debian/Ubuntu):**
```bash
sudo apt-get install build-essential libpcre3-dev libssl-dev zlib1g-dev libgd-dev libgeoip-dev
```

**Compilation Steps:**

```bash
# 1. Download Source
wget https://nginx.org/download/nginx-1.25.3.tar.gz
tar -zxvf nginx-1.25.3.tar.gz

# 2. Download 3rd Party Modules (Optional)
git clone https://github.com/google/ngx_brotli.git
cd ngx_brotli && git submodule update --init && cd ..

# 3. Configure
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

# 4. Compile and Install
make -j$(nproc)
sudo make install
```

### Dynamic Modules

Since version 1.9.11, Nginx supports dynamic modules. This allows you to compile a module as a shared object (`.so`) and load it at runtime via `load_module` in the configuration, similar to Apache's `LoadModule`. This separates the Nginx core upgrade cycle from module updates.

```nginx
# nginx.conf
load_module modules/ngx_http_image_filter_module.so;
load_module modules/ngx_http_geoip_module.so;
```

To compile a module as dynamic, use `--add-dynamic-module=/path/to/module` during configuration.

## Configuration Philosophy and Syntax

The configuration file (`nginx.conf`) is a tree-like structure of **Directives** and **Contexts**. Understanding the parsing logic is vital to avoid misconfiguration.

### The Context Hierarchy

1.  **Main Context:** The global scope. Directives here affect the entire application (`user`, `worker_processes`, `pid`, `error_log`).
2.  **Events Context:** Configures the network model (`worker_connections`, `use epoll`, `multi_accept`).
3.  **HTTP Context:** The web server layer.
    *   **Upstream Context:** Defines groups of backend servers.
    *   **Server Context:** Defines a Virtual Host (IP/Port/Domain combination).
        *   **Location Context:** Defines routing based on URI.
            *   **Nested Location Context:** Further routing refinement.
4.  **Stream Context:** The Layer 4 (TCP/UDP) proxy layer.
5.  **Mail Context:** The POP3/IMAP/SMTP proxy layer.

### Directive Types

1.  **Simple Directives:** Name and parameters, ending with a semicolon.
    *   `root /var/www;`
    *   `listen 80;`
2.  **Block Directives:** Name and parameters, ending with `{ ... }`.
    *   `server { ... }`
    *   `location / { ... }`

### Inheritance Rules

Most directives are inherited downwards. If you define `root /var/www;` in the `http` block, all `server` blocks inherit it unless they override it.

**Crucial Exception: Array Directives**
Directives that can be specified multiple times (like `proxy_set_header`, `add_header`, `access_log`) generally **do not** merge with parent values. If you define `add_header` in a `server` block and then again in a `location` block, the `location` block *replaces* the parent's headers entirely.

```nginx
server {
    add_header X-Server-Level "True";
    
    location / {
        add_header X-Location-Level "True";
        # Result: ONLY X-Location-Level is sent. X-Server-Level is lost.
    }
}
```
To fix this, you must redeclare the parent headers if needed, or structure your config to apply common headers at the appropriate level.

### The "If is Evil" Principle

The `if` directive in Nginx is technically part of the `rewrite` module. It evaluates conditions at the rewrite phase of the request processing.

**Why is it dangerous?**
Nginx configuration is declarative. `if` introduces imperative logic that interacts poorly with other declarative directives (like `proxy_pass` or `try_files`). Inside an `if` block, Nginx creates an implicit, anonymous location. Requests can get trapped in this location, and directives defined outside might not apply as expected.

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
In this case, if the condition matches, `try_files` might be ignored because it inherits from the outer block, but the `if` block changes the context execution flow.

**Safe usage of `if`:**
1.  `return ...` (immediately stopping processing).
2.  `rewrite ... last;` (changing the URI).

**Alternative: The `map` Directive**
For variable assignment based on conditions, use `map` in the `http` context. It is efficient and declarative.

```nginx
# BETTER: Using map
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

### Variables

Nginx variables (`$host`, `$uri`, `$remote_addr`) are calculated **on demand**, (lazy evaluation).
*   **Built-in variables:** Provided by core modules.
*   **Custom variables:** Created via `set`, `map`, or `geo`.

Performance note: Excessive use of `set` and regex captures can add CPU overhead. `map` is generally preferred as it is evaluated only when the variable is accessed.

## The HTTP Core

### Server Selection

How does Nginx decide which `server` block to use?

1.  **Listen Directive:** Nginx first filters by IP address and Port.
2.  **Server Name:** If multiple servers listen on the same port, it checks the `Host` header against `server_name`.
    *   **Exact match:** `www.example.com`
    *   **Wildcard start:** `*.example.com`
    *   **Wildcard end:** `www.example.*`
    *   **Regex:** `~^(?<user>.+)\.example\.net$`
3.  **Default Server:** If no match is found, it uses the server marked `default_server` in the `listen` directive, or the first one defined in the configuration file.

```nginx
server {
    listen 80 default_server;
    server_name _;
    return 444; # Close connection without response (security)
}
```

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

### URL Rewriting

The `rewrite` directive changes the URI internally. It can also issue external redirects (301/302).

*   `rewrite ^/old/(.*)$ /new/$1 last;`: Changes URI to `/new/...` and restarts the location search with the new URI.
*   `rewrite ^/old/(.*)$ /new/$1 break;`: Changes URI but stays in the current location block.
*   `rewrite ^/old/(.*)$ /new/$1 redirect;`: Returns 302 Found (Temporary).
*   `rewrite ^/old/(.*)$ /new/$1 permanent;`: Returns 301 Moved Permanently.

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
    
    # Buffering (Default On)
    # Nginx reads the whole response from backend before sending to client.
    proxy_buffering on;
    proxy_buffers 16 4k;
    proxy_buffer_size 4k;
    
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
*   **Trap:** If you use regex in `location`, `proxy_pass` CANNOT have a URI part. It must be `proxy_pass http://backend;`.

### Load Balancing Strategies

Define a pool of servers using `upstream`.

```nginx
upstream backend_cluster {
    # 1. Round Robin (Default)
    server 10.0.0.1;
    server 10.0.0.2;

    # 2. Least Connections (Recommended for varying request times)
    least_conn;
    
    # 3. IP Hash (Sticky Sessions based on Client IP)
    # ip_hash;
    
    # 4. Generic Hash (Consistent Hashing)
    # hash $request_uri consistent;
    
    # Parameters
    server 10.0.0.3 weight=3;        # Receive 3x traffic
    server 10.0.0.4 max_fails=3 fail_timeout=30s; # Passive Health Check / Circuit breaker
    server 10.0.0.5 backup;          # Only used if others are down
    server 10.0.0.6 down;            # Manually marked offline
}
```

### Active Health Checks

The open-source version of Nginx only supports **passive** health checks (`max_fails`). It marks a server as down only *after* a request fails.
Nginx Plus (Commercial) supports **active** health checks (periodically probing an endpoint).

However, you can achieve active checks in Open Source using third-party modules like `ngx_http_upstream_check_module` (from Tengine) or using Lua to probe endpoints.

### Keep-Alive to Upstream

By default, Nginx closes the connection to the backend after every request. This is inefficient for TLS backends or high throughput (port exhaustion).

To enable Keep-Alive:
```nginx
upstream backend {
    server 10.0.0.1:8080;
    keepalive 32;  # Number of idle connections to keep open per worker
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

Nginx can act as a robust Content Delivery Network (CDN) node using `proxy_cache`. It caches upstream responses to disk.

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
*   `max_size`: Max disk size. Cache Manager removes least recently used items when full.
*   `inactive`: Delete content not accessed within this time, regardless of expiration.
*   `use_temp_path=off`: Write directly to cache directory to avoid unnecessary file copying/renaming (Performance).

### Enabling Caching

```nginx
location / {
    proxy_cache my_cache;
    proxy_cache_key "$scheme$request_method$host$request_uri";
    
    # Caching Rules
    proxy_cache_valid 200 302 10m;
    proxy_cache_valid 404      1m;
    proxy_cache_valid any      5m;
    
    # Bypass Cache (e.g., for debugging or cookies)
    proxy_cache_bypass $http_x_refresh;
    
    # Handling Errors (Stale Cache)
    # If backend is down (500/502/504), serve cached content even if expired.
    proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
    
    # Background Update (Experimental in older versions, stable in new)
    proxy_cache_background_update on;
    
    # Locking (Dog-pile effect prevention)
    # Only one request goes to backend to populate cache; others wait.
    proxy_cache_lock on;
    proxy_cache_lock_timeout 5s;
    
    add_header X-Cache-Status $upstream_cache_status; # HIT/MISS/BYPASS/EXPIRED
}
```

### Purging Cache

Nginx Plus supports `proxy_cache_purge`. For Open Source, you can use the `ngx_cache_purge` module or simply `rm` the files (if you can map the key to the file path).

## Security and Hardening

Nginx is secure by default, but production environments require active hardening.

### 1. Hiding Metadata
Prevent leaking version numbers to scanners.
```nginx
server_tokens off;
```
(Note: To remove the "Server: nginx" header entirely, you need the `headers-more` module).

### 2. TLS Best Practices (2024/2025 Edition)
Use Mozilla's SSL Configuration Generator.

```nginx
ssl_protocols TLSv1.2 TLSv1.3; # Drop SSLv3, TLS 1.0, 1.1
ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
ssl_prefer_server_ciphers off;

# Session Resumption (Performance)
ssl_session_cache shared:SSL:10m;
ssl_session_timeout 1d;
ssl_session_tickets off; # If multiple servers, rotate tickets manually or use Redis

# HSTS (Strict Transport Security)
add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;

# OCSP Stapling (Privacy & Performance)
ssl_stapling on;
ssl_stapling_verify on;
ssl_trusted_certificate /etc/nginx/ssl/chain.pem;
resolver 1.1.1.1;
```

### 3. Security Headers
Inject headers to protect clients.

```nginx
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "no-referrer-when-downgrade" always;
add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;
```

### 4. Rate Limiting (DDoS Mitigation)
Nginx uses the **Leaky Bucket** algorithm.

**Zone Definition:**
```nginx
# 10 requests per second based on binary IP address
# Zone name: api_limit, size: 10MB (~160k IPs)
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;
```

**Application:**
```nginx
location /api/ {
    limit_req zone=api_limit burst=20 nodelay;
    # burst=20: Allow a queue of 20 requests.
    # nodelay: Don't slow them down, just reject immediately if queue is full (503).
    
    limit_req_status 429; # Return 429 Too Many Requests instead of 503
}
```

### 5. Connection Limiting
Limit the number of concurrent connections per IP.

```nginx
limit_conn_zone $binary_remote_addr zone=conn_limit:10m;

server {
    limit_conn conn_limit 10; # Max 10 concurrent connections per IP
}
```

### 6. Preventing Buffer Overflows
Restrict the size of data clients can send to prevent memory exhaustion attacks.

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
worker_processes auto; # One per core
worker_cpu_affinity auto; # Bind to cores

# Increase Limit
worker_rlimit_nofile 65535; 

events {
    worker_connections 65535; # Max connections per worker
    multi_accept on; # Accept as many connections as possible per wake-up
    use epoll;
}

http {
    # File I/O
    sendfile on;
    tcp_nopush on; # Send headers + file start in one packet (Linux only)
    tcp_nodelay on; # Disable Nagle's algorithm (Lower latency for keep-alive)
    
    # Keepalive
    keepalive_timeout 65;
    keepalive_requests 10000; 
    
    # Open File Cache (Metadata caching)
    open_file_cache max=10000 inactive=20s;
    open_file_cache_valid 30s;
    open_file_cache_min_uses 2;
    open_file_cache_errors on;
    
    # Compression (Gzip)
    gzip on;
    gzip_comp_level 5; # Balance CPU and Size (1-9)
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
        
    # Brotli (If module installed - better than Gzip)
    # brotli on;
    # brotli_comp_level 6;
    # brotli_types ...;
}
```

### Linux Kernel Tuning (`/etc/sysctl.conf`)
Nginx cannot open more sockets than the kernel allows.

```ini
# Max open files (system wide)
fs.file-max = 2097152

# Backlog for incoming connections (Syn Queue)
# If this is full, clients get "Connection refused" or timeouts
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535

# Ephemeral Ports (Increase range for outgoing proxy connections)
net.ipv4.ip_local_port_range = 1024 65535

# Time Wait reuse (Safe for Nginx acting as client to upstream)
net.ipv4.tcp_tw_reuse = 1

# Reduce time connections spend in FIN-WAIT-2
net.ipv4.tcp_fin_timeout = 15

# TCP Window Scaling (For high BDP networks)
net.ipv4.tcp_window_scaling = 1

# Congestion Control (BBR is recommended for newer kernels)
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
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
                'uct=$upstream_connect_time uht=$upstream_header_time '
                'pipe=$pipe';

access_log /var/log/nginx/access.log perf buffer=32k flush=5s;
```
*   `rt`: Total Request Time (Nginx + Upstream).
*   `urt`: Upstream Response Time (Backend processing time).
*   `uct`: Time to establish TCP connection to backend.
*   `buffer=32k flush=5s`: Reduces disk IOPS by buffering log writes.

### JSON Logging for ELK/Splunk
Structured logging is essential for modern observability stacks.

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

### Debug Log
To trace exactly how Nginx processes a request (headers parsing, rewrite rules logic), enable debug logging. **Warning: Generates massive logs.**

1.  Ensure Nginx is compiled with `--with-debug`.
2.  Configure: `error_log /var/log/nginx/debug.log debug;`
3.  Or, enable only for specific IPs to debug production issues without flooding logs:
    ```nginx
    events {
        debug_connection 192.168.1.1;
        debug_connection 10.0.0.0/24;
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
*   `Reading`: Nginx is reading request header.
*   `Writing`: Nginx is writing response to client.
*   `Waiting`: Keep-alive connections (Active - (Reading + Writing)).

For more detailed metrics (like per-upstream stats), compile with `ngx_vts_module` (Virtual Host Traffic Status).

## Extending Nginx with Lua (OpenResty)

Standard Nginx config is declarative and limited in logic. **OpenResty** integrates **LuaJIT** into Nginx, allowing you to script request handling with high performance. This turns Nginx into a full-fledged Web Application Server.

### Architecture
OpenResty hooks into Nginx's phases.
*   `init_by_lua`: Master process startup.
*   `init_worker_by_lua`: Worker process startup.
*   `ssl_certificate_by_lua`: Dynamic SSL cert loading.
*   `access_by_lua`: Auth and Access control.
*   `content_by_lua`: Generating content.
*   `header_filter_by_lua`: Modifying response headers.
*   `body_filter_by_lua`: Modifying response body.
*   `log_by_lua`: Custom logging.

### Cosockets
The magic of OpenResty is **Cosockets** (Coroutine Sockets). They allow Lua code to perform network I/O (connect to Redis, MySQL, HTTP APIs) in a **non-blocking** way. Nginx yields the Lua coroutine when I/O starts and resumes it when data arrives, all without blocking the worker process.

### Common Use Cases
1.  **Complex Routing:** Route based on Redis key lookup or JWT claims.
2.  **Authentication:** Validate OAuth2/JWT tokens directly in Nginx before hitting the backend.
3.  **Dynamic Upstreams:** Change backend targets without reloading.
4.  **WAF:** Build custom firewall rules (e.g., Lua-Resty-WAF).
5.  **API Aggregation:** Call multiple microservices and aggregate results.

### Example: Dynamic Redis-based Allowlist

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
| **Modules** | Loadable `.so` | Dynamic `.so` (since 1.9.11) |

### Rewrite Rule Translation

**Apache:**
```apache
RewriteEngine On
RewriteRule ^/user/([0-9]+)$ /profile.php?id=$1 [L]
```

**Nginx:**
```nginx
location ~ ^/user/([0-9]+)$ {
    # Approach 1: Rewrite
    rewrite ^/user/([0-9]+)$ /profile.php?id=$1 break;
    
    # Approach 2: FastCGI (Performance)
    fastcgi_pass unix:/run/php/php8.1-fpm.sock;
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

## Troubleshooting Common Errors

### 502 Bad Gateway
*   **Cause:** Nginx cannot connect to the upstream (backend).
*   **Check:** Is the backend running? Is the port correct? Firewall issues?
*   **Logs:** check `error.log` for "connect() failed".
*   **Socket Permissions:** If using Unix Sockets, check if the `nginx` user has RW permissions on the socket file.

### 504 Gateway Timeout
*   **Cause:** Backend accepted the connection but took too long to respond.
*   **Fix:** Optimize backend code (DB queries). Increase `proxy_read_timeout`.
    ```nginx
    proxy_read_timeout 300s;
    ```

### 413 Request Entity Too Large
*   **Cause:** Upload size exceeds limit.
*   **Fix:** Increase `client_max_body_size`.

### "Worker connections are not enough"
*   **Cause:** High traffic exceeded `worker_connections` limit.
*   **Fix:** Increase `worker_connections` in `events` block (and check `ulimit`).

### "Too many open files"
*   **Cause:** System or process file descriptor limit reached.
*   **Fix:** `worker_rlimit_nofile` in nginx.conf and `ulimit -n` / `/etc/security/limits.conf`.

## References & Resources

*   [Nginx Official Documentation](http://nginx.org/en/docs/)
*   [Nginx Wiki](https://wiki.nginx.org/Main)
*   [OpenResty](https://openresty.org/en/)
*   [Mozilla SSL Configuration Generator](https://ssl-config.mozilla.org/)
*   [Cloudflare's Nginx Tuning Blog Posts](https://blog.cloudflare.com/tag/nginx/) - Deep dives into kernel tuning.
*   [Agentzh's Nginx Tutorials](https://openresty.org/en/nginx-tutorials.html) - By the creator of OpenResty.

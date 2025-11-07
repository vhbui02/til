# Nginx Cheatsheet

<!-- tl;dr starts -->

My favorite [free and open source](https://github.com/nginx/nginx) HTTP web server, reverse proxy (a.k.a load balancer) with cache, TCP/UDP proxy server, mail proxy server. It's primarily used to serve static content, and connect with FastCGI applications (or Python's WSGI, ASGI, Java's Servlets, Ruby's Rack...) to serve dynamic content.

<!-- tl;dr ends -->

## CLI

```sh
# send signal to master process
# start new worker process with new config + gracefully shutdown old worker processes
nginx -s reload
nginx -s stop # shutdown quickly
nginx -s quit # shutdown gracefully
# NOTE: The user who starts the Nginx process must also be the one to stop it.

# Read nginx process ID
cat /usr/local/nginx/logs/nginx.pid
cat /var/run/nginx.pid
ps aux | grep nginx

kill -s QUIT $PROCESS_ID

# test config file
nginx -t

# test + dump config file to stdout
nginx -T
```

## Configuration File

Conventional locations:

- `/usr/local/nginx/conf`
- `/etc/nginx`
- `/usr/local/etc/nginx`

```conf
# feature-specific configuration files
# located in /etc/nginx/conf.d
include conf.d/http;    # NOTE: this is called "simple directive"
include conf.d/stream;
include conf.d/exchange-enhanced;

# NOTE: user directive makes sense only if the master process runs with
# super-user privileges
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /run/nginx.pid;

# "events" context, a.k.a top-level directive, reside in "main context"
# define general connection processing
events { # NOTE: this is called "block directive".

    # Connection types:
    # - Client-to-Nginx connections
    # - Internal connections (within the Nginx edge)
    # - Nginx-to-backend services upstream connections to.

    # Value depends on the hardware, software, whether you're serving static files or proxying to backends
    # - small site = <1024/process
    # - high traffic = 4096-8192/process
    # - very high traffic = >16384/process

    # Total theoretical capacity = # of Worker process * 1024
    # OS may limit # of file descriptors available, prevent reaching the theoretical capacity.

    # ONE Nginx Worker process can handle less than or equal to 1024 concurrent connections.
    worker_connections  1024;
}

# "http", a.k.a top-level directive, reside in "main context"
# define HTTP traffic to multiple virtual servers
http {
    # NOTE: 'server' directive is defined inside 'http' context

    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    # the server behind Nginx must have same value
    keepalive_timeout  65;

    # best practice is to move 'server' directive into separate file
    include /etc/nginx/conf.d/*.conf;

    # `server` blocks are diffrentiated by ports and server names
    # Two virtual servers can't listen on the same port
    # nginx use `server` to determine which server processes a request
    # nginx tests the URI specified in request's header
    # against the parameters of the `location` directives, which defined in `server` block
    server {
        listen              80;
        listen         [::]:80;
        server_name  localhost;

        #access_log  /var/log/nginx/host.access.log  main;

        # 1. prefix match
        location / {
            root    /usr/share/nginx/html;
            index   index.html index.htm;
        }

        # create internal redirect, `location` directive can refer to it by adding `internal` simple directive
        error_page 404 /custom_404.html;

        # 2. exact match
        location = /custom_404.html {

            # NOTE: usually `root` directive is ignored in the location block
            # NOTE: it automatically matches the `root` directive in the server block
            # NOTE: being explicit helps you anticipate future change from server block
            root /usr/share/nginx/html;

            # ensures custom 404 page must be accessed through internal redirects
            # e.g. via `error_page` directive, not direct external requests
            internal;
        }

        # Q: what if there are several maching `location` blocks?
        # A: the block with the longest prefix will be chosen
        location /images/ {
            # Q: why not `root /data/images` ?
            # A: the /images/ prefix will be implicitly used
            root /data;
        }

        # create internal redirect
        error_page 500 502 503 504 /custom_50x.html;

        # exact match syntax
        location = /custom_50x.html {
            root /usr/share/nginx/html;
            internal;
        }

        # Test 502 Bad Gateway error
        location /testing {
            # specify where to forward request for processing

            # unix domain sockets are a common way to communicate with FastCGI processes
            # for better performance, compared to TCP connections
            fastcgi_pass unix:/does/not/exist
            #fastcgi_pass unix:/var/run/php/php8.1-fpm.sock     # PHP-FPM, read-write permission
        }

        #location ~ <regex>         # case-sensitive regex
        #location ~* <regex>        # case-insensitive regex
        #location ^~ /path          # priority prefix match, blocks matching subsequent prefixes that're longer

        #rewrite ^/rewriteme/(.*)$ /$1 last;    # a request for /rewriteme/foobar will become a request to /foobar and a location is search...

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }

    server {
        # if the request can't match a single `server` and `location:` in other `server` directive, it goes into this directive
        listen 80 default_server;

        # nothing
        server_name _;

        # nuke
        return 444;
    }

    # ========================================================================= #
    # Gzip Compression                                                          #
    # ========================================================================= #
    gzip  on;

    # add "Vary: Accept-Encoding" HTTP header
    # return diff responses depends on whether the client supports compression
    gzip_vary on;

    # sets a minimum file size threshold of 1024 bytes (1 KB) before comporession is applied
    # not everytime compression is good
    gzip_min_length 1024;

    # 1-9
    # 6=good balance between compression ratio and CPU usage
    gzip_comp_level 6;

    # MIME type
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/json
        application/javascript;
        # NOTE: don't compress image/video, they're already compressed formats
        # NOTE: only text-based content benefits the most

    # ========================================================================= #
    # Optimal Buffer Size                                                       #
    # ========================================================================= #

    # if the client req's body < 128k, it's stored entirely in memory
    # if the client req's body > 128k, excess data is written to a tmp file on disk
    client_body_buffer_size 128k;

    # prevents clients from sending excessively large requests
    # if a client tries to send a req body > 50MB, Nginx return a 413 Request Entity Too Large
    client_max_body_size 50m;

    # ========================================================================= #
    # Example #1: HTTP Forward Proxy to Telegram server                         #
    # ========================================================================= #

    # Client will be AWS CloudFront

    server {
        listen       80;
        # listen  [::]:80;  # AWS CloudFront don't support IPv6 if using signed URLs/cookies
        server_name  iamhung.top;

        location /telegram {
            # gzip on;  # most are *.ts and *.vtt files

            # Forward request to Telegram API
            # Terminate the original request and create a new request
            # URL tranformation: http://iamhung.top/telegram/bot<token>/sendDocument
            # Proxied request: https://api.telegram.org/bot<token>/sendDocument
            # Use cases:
            # - Bypass reginal restrictions: if Telegram API is blocked in some regions, such as my country
            # - SSL Termination: clients can use HTTP to Nginx and Nginx uses HTTPS to Telegram. The latter connection must be secure.
            # - API Key protection: hide bot tokens from client-side code
            # - Rate Limiting: control access to Telegram API, appropriate to its rate limits
            # - Caching: buffer responses for better performance (
            proxy_pass https://api.telegram.org;

            # Without it, the Telegram server will receive `Host: iamhung.top` header
            # Telegram servers expect the requests are for them, not for my Nginx server
            # Ensure hostname validation + SSL certificate validation
            proxy_set_header Host api.telegram.org;

            # buffer the complete response from Telegram server before sending it to the client
            proxy_buffering on;

            # buffer response headers and the beginning of the response body
            proxy_buffer_size 128k;

            # the no. of buffers and their size for response body
            proxy_buffers 4 256k;

            # the size of buffers that are allocated for sending data to clients
            proxy_busy_buffers_size 256k;

            # Total buffer capacity: 4 * 256k + 128k = 1152k
            # Memory efficient: 256k can be sent to clients while still buffering
            # Validation: 256k < (1024k - 256k) = 256k < 768k
            # After filling 256k, it could be sent to the client immediately
            # The rest 768k will continue to receive data from Telegram
        }
    }
}

# ...
stream {
    # configuration specific to TCP/UDP affecting virtual servers
    server {
        # configuration of TCP virtual server 1
    }
}
```

## Reverse Proxy for PHP web application

```ini
server {
    listen      80;
    server_name example.org www.example.org;
    # global root, no per-location root
    root        /data/www;

    location / {
        index   index.html index.php;
    }

    location ~* \.(gif|jpg|png)$ {
        # set cache headers telling browsers and intermediate caches to store these images for 30 days
        # improve website performance by reducing server requests for static assets
        # best for assets that are unchanged for a long time
        expires 30d;
    }

    location ~ \.php$ {
        # where to forward FastCGI requests
        fastcgi_pass  localhost:9000;   # might be PHP-FPM

        # tells PHP-FPM: which script file to execute
        # set env var SCRIPT_FILENAME to the full filesystem path to the requested script
        # the absolute path made by: web root directory + script path from URL
        fastcgi_param SCRIPT_FILENAME
                    $document_root$fastcgi_script_name;

        # pulls in a standard set of FastCGI parameters from Nginx's configuration files
        # e.g. REQUEST_METHOD, QUERY_STRING, CONTENT_TYPE
        include       fastcgi_params;
    }
}
```

## Proxy traffic to a Jenkins app server

Benefits:

- Acts as a TLS termination proxy
- Static file serving
- Load balancing

```conf
# defint backend server pool
upstream app_server {
    server 127.0.0.1:8080 fail_timeout=0; # never mark the server as unavailable, even if the server becomes unresponsive
}

server {
    listen  80;
    listen [::]:80 default ipv6only=on; # make this virtual server handles IPv6 traffic by default, ensuring it only handles IPv6 requests
    server_name ci.yourcompanyname.com;

    location / {
        # proxy_set_header: modify HTTP headers when Nginx acts as a reverse proxy
        # proxy_redirect:

        # retain client's IP address
        # if the incoming request already has an X-Forwarded-For header (meaning it passed through other proxies) this variable will append the client's IP address to the existing list.
        # If no such header exists, it will create one with just the client's IP address.
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        # retain original "Host" header in client's request
        # a client's request specify "Host" as the domain name of Nginx proxy server (e.g. example.com)
        # when the request pass through Nginx proxy server, it changes the "Host" header to match the upstream server's address, which is "localhost:8080"
        # upstream server receive request from Nginx proxy server will see "Host" set to "localhost:8080", not "example.com".
        # Therefore create breakage on some features
        proxy_set_header Host $http_host;   #

        # When nginx acts as a reverse proxy, upstream servers sometimes send HTTP redirect responses (301, 302, ...) the contain URLs pointing back to themselves
        # By default, nginx automatically rewrites these redirect URLs to match the client's perspective: that is to replace the upstream server's address with Nginx proxy server's address
        # Setting it off will retain the upstream server's address
        proxy_redirect off;

        # Attempt serving files in a specific order
        # first, it tries to find and serve a file that matches the exact URI path requested by the client $uri (e.g. <root>/foobar)
        # if the file doesn't exist on the filesystem, Nginx will fallback to the second option @app_server, it will try to find it there
        try_files $uri @app_server;
    }

    # named location block
    # it's not a file or a directory, any directives use this will ref to this separately defined location block
    location @app_server {
        proxy_pass http://app_server;
    }
}
```

## AWS CloudFront

```conf
server {
    # ...
    # Optional: Add a header to verify requests (can be checked by CloudFront later if desired,
    # but the primary security here is the tunnel itself)
    # add_header X-Origin-Server "MyVideoOrigin" always;

    location ~* ^/streaming/.*\.(m3u8|ts|vtt)$ {
        add_header Cache-Control "public, max-age=315360000"; # 10 years, CloudFront will respect it
        try_files $uri =404;
        autoindex off; # disable directory listing, users can't browse directory contents
    }
    # ...
}
```

## `lua-resty-redis` API

```conf
location / {
    try_files $uri $uri/ =404;
    autoindex off; # disable directory listing, users can't browse directory contents
}

server {
    location / {
        content_by_lua_block {
            local redis = require "resty.redis"
            local red = redis:new()

            -- max time to connect to Redis server
            -- max time to send data to Redis server
            -- max time to wait for response from Redis server
            red:set_timeouts(100, 100, 100); -- local Redis
            -- red:set_timeouts(3000, 1000, 3000); -- remote Redis
            -- red:set_timeouts(500, 200, 1000); -- balanced

            local ok, err = red:connect("unix:/path/to/redis.sock");    -- UNIX domain socket
            -- local ok, err = red:connect("127.0.0.1", 6379);          -- IP address
            -- local ok, err = red:connect("redis.openresty.com", 6379);-- hostname, required resolver

        }
    }
}
```

## References

- [carlessanagustin/Nginx_Cheat_Sheet.md](https://gist.github.com/carlessanagustin/9509d0d31414804da03b)

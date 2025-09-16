Here’s a **detailed set of notes on NGINX** that cover fundamentals, advanced concepts, and production practices:

---

# 📘 NGINX Detailed Notes

## 1. Introduction to NGINX

* **NGINX** (pronounced “Engine-X”) is a **high-performance web server, reverse proxy, load balancer, and HTTP cache**.
* Originally built to solve the **C10k problem** (serving 10,000+ concurrent connections efficiently).
* Written in **C** → lightweight, fast, low resource usage.
* Commonly used as:

  * **Web server** (serving static/dynamic content).
  * **Reverse proxy** (sits in front of application servers).
  * **Load balancer** (distributes traffic across multiple backends).
  * **HTTP cache** (caching responses to reduce load).

---

## 2. Key Features

* Event-driven, asynchronous architecture → handles thousands of connections with low memory.
* High concurrency support.
* SSL/TLS termination (HTTPS).
* Gzip compression & Brotli support.
* URL rewriting & redirects.
* Virtual hosting (name-based & IP-based).
* Rate limiting & request throttling.
* Access control & security features (e.g., WAF integration).
* Integration with **FastCGI, uWSGI, SCGI** for dynamic content.

---

## 3. NGINX Architecture

* **Master process**:

  * Reads configuration.
  * Manages worker processes.
* **Worker processes**:

  * Handle client requests (event-driven).
  * Number of workers usually = number of CPU cores.
* **Modules**:

  * Core modules (basic HTTP, mail, stream).
  * Third-party modules (e.g., Lua, ModSecurity).
* **Event-driven model** → uses `epoll` (Linux), `kqueue` (BSD), IOCP (Windows).

---

## 4. Installation

### On Linux (Ubuntu/Debian)

```bash
sudo apt update
sudo apt install nginx -y
```

### On CentOS/RHEL

```bash
sudo yum install epel-release -y
sudo yum install nginx -y
```

### Start & Enable Service

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

### Check status

```bash
systemctl status nginx
```

---

## 5. Configuration Files

* Main config file: `/etc/nginx/nginx.conf`
* Virtual hosts/sites:

  * `/etc/nginx/sites-available/`
  * `/etc/nginx/sites-enabled/`
* Log files:

  * Access logs → `/var/log/nginx/access.log`
  * Error logs → `/var/log/nginx/error.log`

### Structure of `nginx.conf`

```nginx
user www-data;
worker_processes auto;
events {
    worker_connections 1024;
}
http {
    include       mime.types;
    sendfile      on;
    keepalive_timeout 65;

    server {
        listen 80;
        server_name example.com;
        root /var/www/html;

        location / {
            index index.html index.htm;
        }
    }
}
```

---

## 6. Serving Static Content

```nginx
server {
    listen 80;
    server_name mysite.com;

    location / {
        root /var/www/mysite;
        index index.html;
    }
}
```

---

## 7. Reverse Proxy

NGINX forwards client requests to backend servers (Node.js, Python, etc.).

```nginx
server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## 8. Load Balancing

Supports **Round Robin, Least Connections, IP Hash**.

### Example (Round Robin):

```nginx
upstream backend {
    server 192.168.1.10;
    server 192.168.1.11;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
    }
}
```

---

## 9. SSL/TLS Configuration

```nginx
server {
    listen 443 ssl;
    server_name secure.example.com;

    ssl_certificate /etc/ssl/certs/example.crt;
    ssl_certificate_key /etc/ssl/private/example.key;

    location / {
        root /var/www/secure;
        index index.html;
    }
}
```

Force HTTP → HTTPS:

```nginx
server {
    listen 80;
    server_name secure.example.com;
    return 301 https://$host$request_uri;
}
```

---

## 10. Security Best Practices

* Hide version:

  ```nginx
  server_tokens off;
  ```
* Enable HTTPS with strong ciphers.
* Use `limit_req_zone` for rate limiting.
* Restrict methods:

  ```nginx
  if ($request_method !~ ^(GET|POST)$ ) {
      return 405;
  }
  ```
* Disable directory listing:

  ```nginx
  autoindex off;
  ```

---

## 11. Caching

### Static file caching

```nginx
location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
    expires 30d;
    access_log off;
}
```

### Proxy cache

```nginx
proxy_cache_path /etc/nginx/cache levels=1:2 keys_zone=my_cache:10m max_size=1g inactive=60m;
server {
    location / {
        proxy_cache my_cache;
        proxy_pass http://backend;
    }
}
```

---

## 12. Logging

Default log formats:

```nginx
log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                '$status $body_bytes_sent "$http_referer" '
                '"$http_user_agent" "$http_x_forwarded_for"';

access_log /var/log/nginx/access.log main;
error_log  /var/log/nginx/error.log warn;
```

---

## 13. Performance Tuning

* Increase worker connections:

  ```nginx
  worker_processes auto;
  worker_connections 4096;
  ```
* Enable Gzip compression:

  ```nginx
  gzip on;
  gzip_types text/plain text/css application/json application/javascript;
  ```
* Use HTTP/2 for TLS:

  ```nginx
  listen 443 ssl http2;
  ```

---

## 14. Advanced Use Cases

* **API Gateway** (with JWT validation, rate limiting).
* **WAF** (Web Application Firewall) integration → ModSecurity, NAXSI.
* **Streaming** (RTMP module for live video).
* **Mail proxy** (IMAP/POP3/SMTP).
* **Service Mesh ingress** (NGINX Ingress Controller in Kubernetes).

---

## 15. NGINX vs Apache

| Feature           | NGINX         | Apache                   |
| ----------------- | ------------- | ------------------------ |
| Architecture      | Event-driven  | Process/thread-based     |
| Static content    | Faster        | Slower                   |
| Dynamic content   | Needs backend | Built-in (mod\_php etc.) |
| Memory usage      | Low           | Higher                   |
| Config complexity | Simple        | Flexible but heavy       |

---

✅ With this knowledge, you can explain **NGINX basics, architecture, reverse proxy, load balancing, SSL/TLS, caching, security, and tuning in interviews or real-world projects**.

---

Do you want me to also prepare **NGINX interview questions with scenarios** (like I did for Istio), so you can practice for DevOps interviews?

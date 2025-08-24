# Nginx Cheatsheet

## Basic Commands

```bash
# Start nginx
sudo systemctl start nginx
sudo nginx

# Stop nginx
sudo systemctl stop nginx
sudo nginx -s stop

# Restart nginx
sudo systemctl restart nginx
sudo nginx -s reload

# Reload configuration (graceful)
sudo systemctl reload nginx
sudo nginx -s reload

# Check nginx status
sudo systemctl status nginx

# Test configuration syntax
sudo nginx -t

# Show nginx version and configuration
nginx -v
nginx -V
```

## Configuration File Locations

```bash
# Main configuration file
/etc/nginx/nginx.conf

# Site configurations
/etc/nginx/sites-available/
/etc/nginx/sites-enabled/

# Log files
/var/log/nginx/access.log
/var/log/nginx/error.log

# Default document root
/var/www/html/
```

## Basic Server Block

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    root /var/www/example.com;
    index index.html index.php;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

## SSL/HTTPS Configuration

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    ssl_certificate /path/to/certificate.crt;
    ssl_certificate_key /path/to/private.key;
    
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512;
    ssl_prefer_server_ciphers off;
    
    root /var/www/example.com;
    index index.html;
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name example.com;
    return 301 https://$server_name$request_uri;
}
```

## Common Location Blocks

```nginx
# Static files with caching
location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}

# PHP processing
location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
}

# Deny access to hidden files
location ~ /\. {
    deny all;
}

# API proxy
location /api/ {
    proxy_pass http://backend-server/;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```

## Load Balancing

```nginx
# Define upstream servers
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080 backup;
}

# Load balancing methods
upstream backend_weighted {
    server 192.168.1.10:8080 weight=3;
    server 192.168.1.11:8080 weight=1;
}

upstream backend_ip_hash {
    ip_hash;
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
}

# Use upstream in location
location / {
    proxy_pass http://backend;
}
```

## Rate Limiting

```nginx
# Define rate limit zone
http {
    limit_req_zone $remote_addr zone=api:10m rate=10r/m;
}

# Apply rate limiting
server {
    location /api/ {
        limit_req zone=api burst=20 nodelay;
        proxy_pass http://backend;
    }
}
```

## Redirects and Rewrites

```nginx
# Permanent redirect
location /old-page {
    return 301 /new-page;
}

# Temporary redirect
location /temp {
    return 302 /temporary-location;
}

# Rewrite rules
location /product {
    rewrite ^/product/([0-9]+)/?$ /item.php?id=$1 last;
}

# Remove trailing slash
rewrite ^/(.*)/$ /$1 permanent;
```

## Security Headers

```nginx
# Security headers
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "no-referrer-when-downgrade" always;
add_header Content-Security-Policy "default-src 'self'" always;
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

# Hide nginx version
server_tokens off;
```

## Logging Configuration

```nginx
# Custom log format
log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                '$status $body_bytes_sent "$http_referer" '
                '"$http_user_agent" "$http_x_forwarded_for"';

# Access log
access_log /var/log/nginx/access.log main;

# Error log with level
error_log /var/log/nginx/error.log warn;

# Disable logging for specific locations
location /health {
    access_log off;
    return 200 "OK";
}
```

## Performance Tuning

```nginx
# Worker processes
worker_processes auto;
worker_connections 1024;

# Gzip compression
gzip on;
gzip_vary on;
gzip_min_length 1000;
gzip_types
    text/plain
    text/css
    application/json
    application/javascript
    text/xml
    application/xml
    application/xml+rss
    text/javascript;

# File caching
open_file_cache max=1000 inactive=20s;
open_file_cache_valid 30s;
open_file_cache_min_uses 2;
open_file_cache_errors on;

# Buffer sizes
client_body_buffer_size 128k;
client_max_body_size 50m;
client_header_buffer_size 1k;
large_client_header_buffers 4 4k;
```

## Useful Variables

```nginx
# Common nginx variables
$host                 # Host header
$server_name          # Server name
$request_uri          # Full original request URI
$uri                  # Current URI in request
$args                 # Query string arguments
$remote_addr          # Client IP address
$http_user_agent      # User agent string
$time_local           # Local time
$status               # Response status
$body_bytes_sent      # Bytes sent to client
$request_time         # Request processing time
$upstream_response_time # Backend response time
```

## Testing and Debugging

```bash
# Test configuration
sudo nginx -t

# Check which config files are loaded
sudo nginx -T

# Show nginx processes
ps aux | grep nginx

# Monitor access logs in real-time
sudo tail -f /var/log/nginx/access.log

# Monitor error logs
sudo tail -f /var/log/nginx/error.log

# Check listening ports
sudo netstat -tlnp | grep nginx
sudo ss -tlnp | grep nginx
```

## Common Issues & Solutions

### 1. 502 Bad Gateway
```bash
# Check if backend service is running
sudo systemctl status php8.1-fpm
# Check socket permissions
ls -la /var/run/php/
```

### 2. 403 Forbidden
```bash
# Check file permissions
ls -la /var/www/
# Check nginx user/group
ps aux | grep nginx
```

### 3. Configuration not reloading
```bash
# Hard restart instead of reload
sudo systemctl restart nginx
```

### 4. Check syntax errors
```bash
sudo nginx -t
```

## Enable/Disable Sites

```bash
# Enable site
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/

# Disable site
sudo rm /etc/nginx/sites-enabled/example.com

# Reload after changes
sudo nginx -s reload
```

## Quick Setup Commands

```bash
# Install nginx (Ubuntu/Debian)
sudo apt update && sudo apt install nginx

# Start and enable nginx
sudo systemctl start nginx
sudo systemctl enable nginx

# Create new site configuration
sudo nano /etc/nginx/sites-available/mysite.com

# Test and reload
sudo nginx -t && sudo systemctl reload nginx
```
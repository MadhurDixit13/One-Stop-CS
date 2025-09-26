# Reverse Proxy 🔄

[![Reverse Proxy](https://img.shields.io/badge/Reverse%20Proxy-Load%20Balancer-blue)](https://en.wikipedia.org/wiki/Reverse_proxy)
[![Nginx](https://img.shields.io/badge/Nginx-Web%20Server-009639)](https://nginx.org/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> **Load Balancer & Gateway** - A reverse proxy is a server that sits between clients and backend servers, handling requests, load balancing, SSL termination, and providing additional security and performance features.

## 🎯 Overview

Reverse proxies are intermediary servers that:
- **Load Balance** - Distribute requests across multiple backend servers
- **Terminate SSL** - Handle HTTPS encryption and decryption
- **Cache Content** - Store frequently requested content for faster delivery
- **Provide Security** - Filter requests and protect backend servers
- **Enable Scaling** - Route traffic to healthy backend instances
- **Simplify Architecture** - Hide backend complexity from clients

## 🚀 Key Features

### Core Capabilities
- **Request Routing** - Route requests to appropriate backend services
- **Load Balancing** - Distribute load across multiple servers
- **SSL Termination** - Handle HTTPS certificates and encryption
- **Content Caching** - Cache static and dynamic content
- **Request Filtering** - Block malicious requests and DDoS attacks
- **Health Monitoring** - Monitor backend server health

### Advanced Features
- **WebSocket Support** - Proxy WebSocket connections
- **HTTP/2 Support** - Modern protocol support
- **Compression** - Gzip/Brotli compression
- **Rate Limiting** - Control request rates per client
- **Authentication** - Basic auth, JWT validation
- **Logging & Monitoring** - Detailed access and error logs

## 🏗️ Architecture

### Reverse Proxy Components
```
Reverse Proxy Architecture
├── 🌐 Client Layer
│   ├── Web Browsers
│   ├── Mobile Apps
│   └── API Clients
├── 🔄 Reverse Proxy
│   ├── Load Balancer
│   ├── SSL Terminator
│   ├── Cache Layer
│   └── Security Filter
├── 🖥️ Backend Servers
│   ├── Web Servers
│   ├── API Servers
│   ├── Database Servers
│   └── Microservices
└── 🔧 Management
    ├── Health Checks
    ├── Configuration
    └── Monitoring
```

## 🛠️ Popular Reverse Proxies

### Nginx
```bash
# Install Nginx
# Ubuntu/Debian
sudo apt update
sudo apt install nginx

# macOS
brew install nginx

# Start Nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

```nginx
# /etc/nginx/sites-available/default
upstream backend {
    server 127.0.0.1:3000;
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
}

server {
    listen 80;
    server_name example.com;
    
    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com;
    
    # SSL Configuration
    ssl_certificate /etc/ssl/certs/example.com.crt;
    ssl_certificate_key /etc/ssl/private/example.com.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512;
    
    # Security Headers
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";
    
    # Gzip Compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;
    
    # Rate Limiting
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
    
    # API Routes
    location /api/ {
        limit_req zone=api burst=20 nodelay;
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket Support
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
    
    # Static Files
    location /static/ {
        alias /var/www/static/;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    # Health Check
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
}
```

### Apache HTTP Server
```apache
# /etc/apache2/sites-available/000-default.conf
<VirtualHost *:80>
    ServerName example.com
    Redirect permanent / https://example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName example.com
    
    # SSL Configuration
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/example.com.key
    
    # Security Headers
    Header always set X-Frame-Options DENY
    Header always set X-Content-Type-Options nosniff
    Header always set X-XSS-Protection "1; mode=block"
    
    # Load Balancing
    <Proxy balancer://backend>
        BalancerMember http://127.0.0.1:3000
        BalancerMember http://127.0.0.1:3001
        BalancerMember http://127.0.0.1:3002
    </Proxy>
    
    # Proxy Configuration
    ProxyPreserveHost On
    ProxyPass /api/ balancer://backend/
    ProxyPassReverse /api/ balancer://backend/
    
    # Static Files
    Alias /static/ /var/www/static/
    <Directory /var/www/static/>
        ExpiresActive On
        ExpiresDefault "access plus 1 year"
    </Directory>
</VirtualHost>
```

### HAProxy
```bash
# Install HAProxy
# Ubuntu/Debian
sudo apt install haproxy

# Configuration
sudo nano /etc/haproxy/haproxy.cfg
```

```haproxy
# /etc/haproxy/haproxy.cfg
global
    daemon
    maxconn 4096
    log stdout local0

defaults
    mode http
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms
    option httplog

# Frontend
frontend web_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/example.com.pem
    
    # Redirect HTTP to HTTPS
    redirect scheme https if !{ ssl_fc }
    
    # Rate Limiting
    stick-table type ip size 100k expire 30s store http_req_rate(10s)
    http-request track-sc0 src
    http-request deny if { sc_http_req_rate(0) gt 10 }
    
    # Backend Selection
    use_backend api_backend if { path_beg /api/ }
    use_backend static_backend if { path_beg /static/ }
    default_backend web_backend

# Backends
backend web_backend
    balance roundrobin
    option httpchk GET /health
    server web1 127.0.0.1:3000 check
    server web2 127.0.0.1:3001 check
    server web3 127.0.0.1:3002 check

backend api_backend
    balance roundrobin
    option httpchk GET /api/health
    server api1 127.0.0.1:8000 check
    server api2 127.0.0.1:8001 check

backend static_backend
    server static1 127.0.0.1:9000 check
```

### Traefik
```yaml
# docker-compose.yml
version: '3.8'

services:
  traefik:
    image: traefik:v2.10
    command:
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.myresolver.acme.tlschallenge=true"
      - "--certificatesresolvers.myresolver.acme.email=admin@example.com"
      - "--certificatesresolvers.myresolver.acme.storage=/letsencrypt/acme.json"
    ports:
      - "80:80"
      - "443:443"
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./letsencrypt:/letsencrypt
    labels:
      - "traefik.enable=true"

  web:
    image: nginx:alpine
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.web.rule=Host(`example.com`)"
      - "traefik.http.routers.web.entrypoints=websecure"
      - "traefik.http.routers.web.tls.certresolver=myresolver"
      - "traefik.http.services.web.loadbalancer.server.port=80"

  api:
    image: my-api:latest
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.api.rule=Host(`api.example.com`)"
      - "traefik.http.routers.api.entrypoints=websecure"
      - "traefik.http.routers.api.tls.certresolver=myresolver"
      - "traefik.http.services.api.loadbalancer.server.port=3000"
```

## 🔧 Load Balancing Strategies

### Round Robin
```nginx
upstream backend {
    server 127.0.0.1:3000;
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
}
```

### Least Connections
```nginx
upstream backend {
    least_conn;
    server 127.0.0.1:3000;
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
}
```

### IP Hash
```nginx
upstream backend {
    ip_hash;
    server 127.0.0.1:3000;
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
}
```

### Weighted
```nginx
upstream backend {
    server 127.0.0.1:3000 weight=3;
    server 127.0.0.1:3001 weight=2;
    server 127.0.0.1:3002 weight=1;
}
```

## 🚀 Advanced Features

### Health Checks
```nginx
upstream backend {
    server 127.0.0.1:3000 max_fails=3 fail_timeout=30s;
    server 127.0.0.1:3001 max_fails=3 fail_timeout=30s;
    server 127.0.0.1:3002 max_fails=3 fail_timeout=30s;
}

server {
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
}
```

### Caching
```nginx
# Cache Configuration
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m max_size=1g inactive=60m;

server {
    location /api/ {
        proxy_cache my_cache;
        proxy_cache_valid 200 302 10m;
        proxy_cache_valid 404 1m;
        proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
        proxy_cache_lock on;
        
        proxy_pass http://backend;
    }
}
```

### Rate Limiting
```nginx
# Rate Limiting Zones
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
limit_req_zone $binary_remote_addr zone=login:10m rate=1r/s;

server {
    location /api/ {
        limit_req zone=api burst=20 nodelay;
        proxy_pass http://backend;
    }
    
    location /login {
        limit_req zone=login burst=5 nodelay;
        proxy_pass http://backend;
    }
}
```

### WebSocket Support
```nginx
server {
    location /ws/ {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 🔐 Security Features

### SSL/TLS Configuration
```nginx
server {
    listen 443 ssl http2;
    
    # SSL Configuration
    ssl_certificate /etc/ssl/certs/example.com.crt;
    ssl_certificate_key /etc/ssl/private/example.com.key;
    
    # Modern SSL Configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    
    # HSTS
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
}
```

### Security Headers
```nginx
server {
    # Security Headers
    add_header X-Frame-Options DENY always;
    add_header X-Content-Type-Options nosniff always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline';" always;
}
```

### DDoS Protection
```nginx
# Limit connections per IP
limit_conn_zone $binary_remote_addr zone=conn_limit_per_ip:10m;

server {
    limit_conn conn_limit_per_ip 20;
    
    # Block suspicious requests
    if ($http_user_agent ~* (bot|crawler|spider)) {
        return 403;
    }
    
    # Block requests with no user agent
    if ($http_user_agent = "") {
        return 403;
    }
}
```

## 🔧 Best Practices

### Performance Optimization
1. **Enable Gzip Compression** - Reduce bandwidth usage
2. **Use HTTP/2** - Better multiplexing and performance
3. **Implement Caching** - Cache static and dynamic content
4. **Optimize SSL** - Use modern ciphers and protocols
5. **Monitor Performance** - Track response times and throughput

### Security Best Practices
1. **Use HTTPS** - Encrypt all traffic
2. **Implement Rate Limiting** - Prevent abuse and DDoS
3. **Add Security Headers** - Protect against common attacks
4. **Regular Updates** - Keep proxy software updated
5. **Monitor Logs** - Watch for suspicious activity

### High Availability
1. **Health Checks** - Monitor backend server health
2. **Failover** - Automatic failover to healthy servers
3. **Load Distribution** - Distribute load evenly
4. **Monitoring** - Track server performance and availability
5. **Backup Configuration** - Backup proxy configurations

## 🚀 Use Cases

### Microservices Gateway
```nginx
upstream user_service {
    server user-service:3000;
}

upstream order_service {
    server order-service:3001;
}

upstream payment_service {
    server payment-service:3002;
}

server {
    location /api/users/ {
        proxy_pass http://user_service/;
    }
    
    location /api/orders/ {
        proxy_pass http://order_service/;
    }
    
    location /api/payments/ {
        proxy_pass http://payment_service/;
    }
}
```

### API Gateway
```nginx
server {
    location /api/v1/ {
        # Authentication
        auth_request /auth;
        
        # Rate Limiting
        limit_req zone=api burst=20 nodelay;
        
        # Caching
        proxy_cache api_cache;
        proxy_cache_valid 200 5m;
        
        # Load Balancing
        proxy_pass http://api_backend/;
    }
    
    location = /auth {
        internal;
        proxy_pass http://auth_service/validate;
        proxy_pass_request_body off;
        proxy_set_header Content-Length "";
        proxy_set_header X-Original-URI $request_uri;
    }
}
```

### CDN Integration
```nginx
server {
    location /static/ {
        # Try local files first, then CDN
        try_files $uri @cdn;
    }
    
    location @cdn {
        proxy_pass https://cdn.example.com;
        proxy_cache static_cache;
        proxy_cache_valid 200 1y;
    }
}
```

## 🔍 Monitoring and Debugging

### Access Logs
```nginx
log_format detailed '$remote_addr - $remote_user [$time_local] '
                   '"$request" $status $body_bytes_sent '
                   '"$http_referer" "$http_user_agent" '
                   '$request_time $upstream_response_time';

access_log /var/log/nginx/access.log detailed;
```

### Error Logs
```nginx
error_log /var/log/nginx/error.log warn;
```

### Status Module
```nginx
server {
    location /nginx_status {
        stub_status on;
        access_log off;
        allow 127.0.0.1;
        deny all;
    }
}
```

## 🐛 Troubleshooting

### Common Issues
1. **502 Bad Gateway** - Backend server down or unreachable
2. **504 Gateway Timeout** - Backend server too slow
3. **SSL Certificate Errors** - Certificate issues or configuration
4. **Load Balancing Issues** - Uneven distribution or health check failures
5. **Performance Issues** - Slow response times or high CPU usage

### Debugging Tools
```bash
# Check Nginx configuration
nginx -t

# Reload configuration
nginx -s reload

# Check status
systemctl status nginx

# View logs
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log

# Test upstream connectivity
curl -I http://backend-server:3000/health
```

## 📚 Learning Resources

### Official Documentation
- [Nginx Documentation](https://nginx.org/en/docs/)
- [Apache HTTP Server](https://httpd.apache.org/docs/)
- [HAProxy Documentation](https://www.haproxy.org/download/2.8/doc/)
- [Traefik Documentation](https://doc.traefik.io/traefik/)

### Community Resources
- [Nginx Configuration Examples](https://github.com/nginxinc/docker-nginx)
- [Load Balancing Guide](https://www.nginx.com/resources/glossary/load-balancing/)
- [SSL Configuration Generator](https://ssl-config.mozilla.org/)

## 🔄 Updates and Roadmap

### Recent Updates
- **Nginx 1.24** - Enhanced HTTP/3 support and performance improvements
- **HAProxy 2.8** - Better observability and security features
- **Traefik 3.0** - Improved Kubernetes integration and performance

### Upcoming Features
- **Enhanced Security** - Better DDoS protection and threat detection
- **Performance Optimization** - Improved caching and compression
- **Cloud Integration** - Better cloud provider integration

## 🤝 Contributing

### How to Contribute
1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests and documentation
5. Submit a pull request

### Contribution Guidelines
- Follow configuration standards
- Write comprehensive documentation
- Test configurations thoroughly
- Ensure backward compatibility

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](https://opensource.org/licenses/MIT) file for details.

## 🙏 Acknowledgments

- **Nginx Team** for creating and maintaining Nginx
- **Apache Foundation** for Apache HTTP Server
- **HAProxy Team** for HAProxy load balancer
- **Traefik Team** for Traefik reverse proxy
- **Open Source Community** for contributions and feedback

---

<div align="center">

**🌟 Star this repository if you find it helpful!**

[⬆ Back to Top](#reverse-proxy-)

</div>

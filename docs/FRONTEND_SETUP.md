# Frontend Setup Guide (Nginx)

## Overview

The Expense frontend is a web application served by Nginx web server. It provides the user interface and communicates with the backend API via reverse proxy. This guide covers installation, configuration, and deployment.

## Prerequisites

- Linux OS (RHEL/CentOS/Fedora)
- Root or sudo access
- DNF package manager
- Backend API already installed and running (optional for initial setup)
- Internet connectivity for downloads

## Architecture

```
Nginx Frontend
├── Purpose: Serve web UI & reverse proxy
├── Port: 80 (HTTP)
├── Document Root: /usr/share/nginx/html
├── Config: /etc/nginx/nginx.conf
├── Service: SystemD (nginx.service)
└── Content: Static HTML, CSS, JavaScript
```

## Installation Steps

### Step 1: Install Nginx Web Server

```bash
dnf install nginx -y
```

**What this does:**
- Downloads and installs Nginx package
- Creates default configuration files
- Sets up standard directory structure
- `-y` flag automatically confirms installation

### Step 2: Enable Nginx Service

```bash
systemctl enable nginx
```

**What this does:**
- Enables Nginx to start automatically on system boot
- Ensures web server is always available
- Registers service with SystemD

### Step 3: Start Nginx Service

```bash
systemctl start nginx
```

**What this does:**
- Starts Nginx immediately
- Web server begins listening on port 80
- Ready to serve content

### Step 4: Verify Installation

Open browser and visit:
```
http://<nginx-server-ip>/
```

**Expected result:**
- Default Nginx welcome page appears
- Shows "Welcome to nginx!"
- Confirms installation was successful

## Configuration

### Step 1: Remove Default Content

```bash
rm -rf /usr/share/nginx/html/*
```

**What this does:**
- Removes default Nginx welcome page
- Clears directory for application content
- Prepares for expense application files

### Step 2: Download Frontend Application

```bash
curl -o /tmp/frontend.zip https://expense-joindevops.s3.us-east-1.amazonaws.com/expense-frontend-v2.zip
```

**What this does:**
- Downloads frontend source code from S3
- Saves to `/tmp/frontend.zip`
- Contains HTML, CSS, JavaScript files

### Step 3: Extract Frontend Content

```bash
cd /usr/share/nginx/html
unzip /tmp/frontend.zip
```

**What this does:**
- Navigates to Nginx document root
- Extracts all frontend files
- Creates directory structure for application
- Makes files accessible via web server

### Step 4: Verify Frontend Content

Open browser and visit:
```
http://<nginx-server-ip>/
```

**Expected result:**
- Expense application appears
- Web interface is functional
- Can see login/dashboard screens

### Step 5: Create Reverse Proxy Configuration

```bash
vim /etc/nginx/default.d/expense.conf
```

**File content:**
```nginx
proxy_http_version 1.1;

location /api/ {
    proxy_pass http://<BACKEND-SERVER-IP>:8080/;
}

location /health {
    stub_status on;
    access_log off;
}
```

**Configuration Details:**

| Parameter | Purpose |
|-----------|---------|
| `proxy_http_version 1.1` | Use HTTP/1.1 for backend |
| `location /api/` | Match all API requests |
| `proxy_pass` | Forward to backend server |
| `location /health` | Health check endpoint |
| `stub_status` | Return server status |
| `access_log off` | Don't log health checks |

**⚠️ Important:** Replace `<BACKEND-SERVER-IP>` with actual backend server IP

**Examples:**
```nginx
# For local deployment (same server)
proxy_pass http://127.0.0.1:8080/;

# For remote backend
proxy_pass http://192.168.1.20:8080/;
```

### Step 6: Reload Nginx Configuration

```bash
nginx -t
```

**What this does:**
- Tests configuration syntax
- Shows any errors in configuration
- Safe way to verify before applying

```bash
systemctl reload nginx
```

**What this does:**
- Reloads Nginx configuration
- Applies changes without stopping service
- Keeps existing connections active

**Alternative (harder restart):**
```bash
systemctl restart nginx
```

### Step 7: Verify Configuration

Open browser and test API proxy:
```
http://<nginx-server-ip>/api/
```

**Expected result:**
- Request proxied to backend
- Receives API response
- No error messages

## Nginx Configuration Details

### File Locations

| Path | Purpose |
|------|---------|
| `/etc/nginx/nginx.conf` | Main configuration file |
| `/etc/nginx/conf.d/` | Additional configuration files |
| `/etc/nginx/default.d/` | Default server configurations |
| `/usr/share/nginx/html` | Document root (static files) |
| `/var/log/nginx/access.log` | Access logs |
| `/var/log/nginx/error.log` | Error logs |

### Main Configuration Structure

```nginx
# Main context
http {
    # Global settings
    
    server {
        # Default server block
        listen 80;
        server_name _;
        
        # Document root
        root /usr/share/nginx/html;
        
        # Include custom configurations
        include /etc/nginx/default.d/*.conf;
    }
}
```

### Reverse Proxy Advanced Options

```nginx
location /api/ {
    proxy_pass http://backend:8080/;
    
    # Preserve client information
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    
    # Connection settings
    proxy_connect_timeout 60s;
    proxy_send_timeout 60s;
    proxy_read_timeout 60s;
}
```

## Management Commands

### Check Service Status
```bash
systemctl status nginx
```

**Output shows:**
- Running/Stopped status
- Process ID
- Memory usage
- Recent events

### View Configuration
```bash
cat /etc/nginx/nginx.conf
```

### View Custom Configuration
```bash
cat /etc/nginx/default.d/expense.conf
```

### View Access Logs
```bash
tail -f /var/log/nginx/access.log
```

**Shows real-time web traffic**

### View Error Logs
```bash
tail -f /var/log/nginx/error.log
```

**Shows any errors or warnings**

### Reload Configuration
```bash
systemctl reload nginx
```

**Applies changes without stopping service**

### Restart Service
```bash
systemctl restart nginx
```

**Hard stop and restart**

### Stop Service
```bash
systemctl stop nginx
```

### Start Service
```bash
systemctl start nginx
```

## Verification

### Check Port Listening
```bash
netstat -tlnp | grep :80
```

**Expected output:**
```
tcp    0    0    0.0.0.0:80    0.0.0.0:*    LISTEN    <pid>/nginx
```

### Check Nginx Process
```bash
ps aux | grep nginx
```

**Expected output shows:**
- Master process
- Worker processes

### Test Configuration Syntax
```bash
nginx -t
```

**Expected output:**
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

### Test Frontend Access
```bash
curl http://localhost/
```

**Should return HTML content**

### Test API Proxy
```bash
curl http://localhost/api/health
```

**Should return backend response**

## Troubleshooting

### Port 80 Already in Use

```bash
lsof -i :80
```

**If another process uses port 80:**
- Stop the other service
- Change Nginx port in configuration
- Or bind to different IP

### Nginx Won't Start

1. **Check logs:**
   ```bash
   journalctl -u nginx -n 50
   ```

2. **Check configuration:**
   ```bash
   nginx -t
   ```

3. **Check permissions:**
   ```bash
   ls -la /usr/share/nginx/html
   ```

### Backend Not Reachable

1. **Verify backend is running:**
   ```bash
   curl http://<backend-ip>:8080/
   ```

2. **Check firewall:**
   ```bash
   firewall-cmd --list-all
   ```

3. **Verify proxy configuration:**
   ```bash
   cat /etc/nginx/default.d/expense.conf
   ```

### Slow Performance

1. **Check server resources:**
   ```bash
   top
   ```

2. **Increase worker processes:**
   ```bash
   vim /etc/nginx/nginx.conf
   # Change: worker_processes auto;
   ```

3. **Enable caching:**
   ```nginx
   proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=cache:10m;
   proxy_cache cache;
   ```

## Security Considerations

### 1. Restrict Backend Access

Only allow localhost access to backend:
```nginx
location /api/ {
    proxy_pass http://127.0.0.1:8080/;
}
```

### 2. Add Security Headers

```nginx
add_header X-Frame-Options "SAMEORIGIN";
add_header X-Content-Type-Options "nosniff";
add_header X-XSS-Protection "1; mode=block";
```

### 3. Limit Request Size

```nginx
client_max_body_size 10m;
```

### 4. Enable HTTPS (Production)

```bash
dnf install certbot python3-certbot-nginx -y
certbot --nginx -d yourdomain.com
```

### 5. Set File Permissions

```bash
chmod 644 /usr/share/nginx/html/*
chmod 755 /usr/share/nginx/html
```

## Performance Tuning

### Worker Processes
```nginx
worker_processes auto;
```

### Worker Connections
```nginx
worker_connections 1024;
```

### Keepalive Timeout
```nginx
keepalive_timeout 65;
```

### Gzip Compression
```nginx
gzip on;
gzip_types text/plain text/css application/json;
gzip_min_length 1000;
```

## References

- [Nginx Official Documentation](https://nginx.org/en/docs/)
- [Nginx Reverse Proxy Guide](https://nginx.org/en/docs/http/ngx_http_proxy_module.html)
- [Nginx Security Best Practices](https://nginx.org/en/docs/)

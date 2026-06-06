# Quick Start Guide

## One-Command Summary

The Expense application has **3 main components** that need to be set up in order:

```
MySQL Database → NodeJS Backend → Nginx Frontend
```

## Quick Setup Timeline

### 1️⃣ DATABASE SETUP (5-10 minutes)
```bash
# Install MySQL
dnf install mysql-server -y
systemctl enable mysqld && systemctl start mysqld

# Set root password
mysql_secure_installation --set-root-pass ExpenseApp@1

# Verify
mysql -uroot -pExpenseApp@1 -e "show databases;"
```

### 2️⃣ BACKEND SETUP (10-15 minutes)
```bash
# Install NodeJS 20
dnf module disable nodejs -y
dnf module enable nodejs:20 -y
dnf install nodejs -y

# Create app user and directory
useradd expense
mkdir /app

# Download and setup application
curl -o /tmp/backend.zip https://expense-joindevops.s3.us-east-1.amazonaws.com/expense-backend-v2.zip
cd /app && unzip /tmp/backend.zip
npm install

# Configure systemd service (edit with your database IP)
cat > /etc/systemd/system/backend.service << 'EOF'
[Unit]
Description = Backend Service

[Service]
User=expense
Environment=DB_HOST="<MYSQL-IP>"
ExecStart=/bin/node /app/index.js
SyslogIdentifier=backend

[Install]
WantedBy=multi-user.target
EOF

# Load schema and start
mysql -h <MYSQL-IP> -uroot -pExpenseApp@1 < /app/schema/backend.sql
chown -R expense:expense /app
systemctl daemon-reload
systemctl start backend && systemctl enable backend

# Verify
curl http://localhost:8080/health
```

### 3️⃣ FRONTEND SETUP (5-10 minutes)
```bash
# Install Nginx
dnf install nginx -y
systemctl enable nginx && systemctl start nginx

# Download and extract frontend
rm -rf /usr/share/nginx/html/*
curl -o /tmp/frontend.zip https://expense-joindevops.s3.us-east-1.amazonaws.com/expense-frontend-v2.zip
cd /usr/share/nginx/html && unzip /tmp/frontend.zip

# Configure reverse proxy (edit with your backend IP)
cat > /etc/nginx/default.d/expense.conf << 'EOF'
proxy_http_version 1.1;

location /api/ {
    proxy_pass http://<BACKEND-IP>:8080/;
}

location /health {
    stub_status on;
    access_log off;
}
EOF

# Reload Nginx
nginx -t
systemctl reload nginx

# Verify
curl http://localhost/
```

## Verification Checklist

✅ **All systems running:**
```bash
systemctl status mysqld
systemctl status backend
systemctl status nginx
```

✅ **MySQL accessible:**
```bash
mysql -h <mysql-ip> -uroot -pExpenseApp@1 -e "show databases;"
```

✅ **Backend responding:**
```bash
curl http://<backend-ip>:8080/health
```

✅ **Frontend accessible:**
```bash
curl http://<frontend-ip>/
```

✅ **API proxy working:**
```bash
curl http://<frontend-ip>/api/
```

## Important IPs to Configure

| Component | Port | IP to Configure |
|-----------|------|-----------------|
| MySQL | 3306 | Database server IP |
| Backend | 8080 | Backend server IP (for Nginx) |
| Frontend | 80 | Any IP (public facing) |

## Troubleshooting Quick Tips

### Backend won't start
```bash
# Check logs
journalctl -u backend -n 20

# Verify MySQL connection
mysql -h <MYSQL-IP> -uroot -pExpenseApp@1 -e "select 1;"
```

### Frontend shows errors
```bash
# Check Nginx config
nginx -t

# Verify backend proxy
curl http://<backend-ip>:8080/

# Check Nginx logs
tail -f /var/log/nginx/error.log
```

### Port already in use
```bash
# Find what's using the port
lsof -i :<port-number>

# Kill process if needed
kill -9 <pid>
```

## Default Credentials

- **MySQL Root:** `root` / `ExpenseApp@1`
- **Application User:** `expense` (daemon user, no login)

## File Locations

```
Project Documentation:
├── README.md                    Main overview
├── docs/
│   ├── ARCHITECTURE.md         System design
│   ├── DATABASE_SETUP.md       MySQL guide
│   ├── BACKEND_SETUP.md        NodeJS guide
│   └── FRONTEND_SETUP.md       Nginx guide
└── config/
    ├── backend.service.example
    ├── expense.conf.example
    └── schema.sql.example
```

## Next Steps

1. **Read Architecture** - Understand how components connect
   ```
   See: docs/ARCHITECTURE.md
   ```

2. **Setup Database** - Install MySQL and configure
   ```
   See: docs/DATABASE_SETUP.md
   ```

3. **Setup Backend** - Deploy NodeJS application
   ```
   See: docs/BACKEND_SETUP.md
   ```

4. **Setup Frontend** - Deploy Nginx web server
   ```
   See: docs/FRONTEND_SETUP.md
   ```

## Support

For detailed information on each component, refer to individual setup guides:
- Database questions → `docs/DATABASE_SETUP.md`
- Backend questions → `docs/BACKEND_SETUP.md`
- Frontend questions → `docs/FRONTEND_SETUP.md`
- Architecture questions → `docs/ARCHITECTURE.md`

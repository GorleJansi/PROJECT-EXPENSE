# Troubleshooting Guide

## Overview

This guide covers common issues encountered during setup and operation of the Expense application and provides step-by-step solutions.

## Table of Contents

1. [Database (MySQL) Issues](#database-mysql-issues)
2. [Backend (NodeJS) Issues](#backend-nodejs-issues)
3. [Frontend (Nginx) Issues](#frontend-nginx-issues)
4. [Network & Connectivity Issues](#network--connectivity-issues)
5. [Application Not Working Issues](#application-not-working-issues)
6. [Performance Issues](#performance-issues)

---

## Database (MySQL) Issues

### Issue 1: MySQL Service Won't Start

**Symptoms:**
- `systemctl start mysqld` fails
- Service shows as "stopped" or "failed"

**Solutions:**

1. **Check service status:**
   ```bash
   systemctl status mysqld
   ```

2. **View error logs:**
   ```bash
   tail -f /var/log/mysql/error.log
   ```

3. **Common causes and fixes:**

   **Port already in use:**
   ```bash
   lsof -i :3306
   kill -9 <pid>
   systemctl start mysqld
   ```

   **Corrupted database files:**
   ```bash
   systemctl stop mysqld
   rm -rf /var/lib/mysql/*
   systemctl start mysqld
   ```

   **Permission issues:**
   ```bash
   chown -R mysql:mysql /var/lib/mysql
   chmod -R 755 /var/lib/mysql
   systemctl start mysqld
   ```

### Issue 2: Cannot Connect to MySQL

**Symptoms:**
- Connection timeout errors
- "Access denied" messages
- "Can't connect to MySQL server"

**Solutions:**

1. **Verify MySQL is running:**
   ```bash
   systemctl status mysqld
   ```

2. **Test local connection:**
   ```bash
   mysql -uroot -pExpenseApp@1
   ```

3. **Test remote connection:**
   ```bash
   mysql -h <mysql-ip> -uroot -pExpenseApp@1
   ```

4. **Check firewall rules:**
   ```bash
   firewall-cmd --list-all
   firewall-cmd --add-port=3306/tcp --permanent
   firewall-cmd --reload
   ```

5. **Verify password is correct:**
   - Default: `ExpenseApp@1`
   - If forgotten, reset:
   ```bash
   systemctl stop mysqld
   mysqld_safe --skip-grant-tables &
   mysql -u root
   FLUSH PRIVILEGES;
   ALTER USER 'root'@'localhost' IDENTIFIED BY 'ExpenseApp@1';
   EXIT;
   systemctl start mysqld
   ```

### Issue 3: Schema Not Loading

**Symptoms:**
- Error when running `mysql_secure_installation`
- Tables don't exist after loading schema
- "Access denied" when loading schema

**Solutions:**

1. **Check if schema file exists:**
   ```bash
   ls -la /app/schema/backend.sql
   ```

2. **Load schema with correct credentials:**
   ```bash
   mysql -h <mysql-ip> -uroot -pExpenseApp@1 < /app/schema/backend.sql
   ```

3. **Verify schema loaded:**
   ```bash
   mysql -h <mysql-ip> -uroot -pExpenseApp@1 -e "USE expense; SHOW TABLES;"
   ```

4. **Check database permissions:**
   ```bash
   mysql -h <mysql-ip> -uroot -pExpenseApp@1 -e "SHOW GRANTS FOR 'root'@'localhost';"
   ```

### Issue 4: Database Running Out of Space

**Symptoms:**
- Write errors in logs
- Application cannot save expenses
- Slow performance

**Solutions:**

1. **Check disk space:**
   ```bash
   df -h
   df -h /var/lib/mysql
   ```

2. **Check database size:**
   ```bash
   mysql -uroot -pExpenseApp@1 -e "SELECT table_schema, ROUND(SUM(data_length+index_length)/1024/1024, 2) as size_mb FROM information_schema.TABLES GROUP BY table_schema;"
   ```

3. **Clear old logs:**
   ```bash
   mysql -uroot -pExpenseApp@1 -e "PURGE BINARY LOGS BEFORE DATE_SUB(NOW(), INTERVAL 7 DAY);"
   ```

---

## Backend (NodeJS) Issues

### Issue 1: Backend Service Won't Start

**Symptoms:**
- `systemctl start backend` fails
- Service shows as "failed"
- Port 8080 not listening

**Solutions:**

1. **Check service status:**
   ```bash
   systemctl status backend
   ```

2. **View detailed logs:**
   ```bash
   journalctl -u backend -n 50
   ```

3. **Check configuration file:**
   ```bash
   cat /etc/systemd/system/backend.service
   ```

4. **Verify node is installed:**
   ```bash
   which node
   node --version
   ```

5. **Check if port is in use:**
   ```bash
   lsof -i :8080
   kill -9 <pid>
   ```

6. **Reload service configuration:**
   ```bash
   systemctl daemon-reload
   systemctl start backend
   ```

### Issue 2: Cannot Connect to Database from Backend

**Symptoms:**
- Backend starts but crashes immediately
- "ECONNREFUSED" errors in logs
- "Can't connect to MySQL"

**Solutions:**

1. **Verify DB_HOST in service file:**
   ```bash
   grep DB_HOST /etc/systemd/system/backend.service
   ```

2. **Test MySQL connectivity from backend server:**
   ```bash
   mysql -h <mysql-ip> -uroot -pExpenseApp@1 -e "SELECT 1;"
   ```

3. **Check if MySQL is reachable:**
   ```bash
   telnet <mysql-ip> 3306
   ping <mysql-ip>
   ```

4. **Update DB_HOST if incorrect:**
   ```bash
   vim /etc/systemd/system/backend.service
   # Update: Environment=DB_HOST="<correct-ip>"
   systemctl daemon-reload
   systemctl restart backend
   ```

5. **Check firewall on MySQL server:**
   ```bash
   firewall-cmd --add-source=<backend-ip>/32 --permanent
   firewall-cmd --reload
   ```

### Issue 3: NodeJS Modules Not Found

**Symptoms:**
- "Cannot find module" errors
- "npm ERR! code ENOENT"
- Application crashes on start

**Solutions:**

1. **Reinstall dependencies:**
   ```bash
   cd /app
   rm -rf node_modules package-lock.json
   npm install
   ```

2. **Check npm version:**
   ```bash
   npm --version
   npm update npm
   ```

3. **Clear npm cache:**
   ```bash
   npm cache clean --force
   npm install
   ```

4. **Check file permissions:**
   ```bash
   chown -R expense:expense /app
   chmod -R 755 /app
   ```

5. **Verify package.json exists:**
   ```bash
   cat /app/package.json
   ```

### Issue 4: Backend Port Already in Use

**Symptoms:**
- "EADDRINUSE" error
- Port 8080 in use by another process
- Service fails to start

**Solutions:**

1. **Find process using port 8080:**
   ```bash
   lsof -i :8080
   netstat -tlnp | grep 8080
   ```

2. **Kill the process:**
   ```bash
   kill -9 <pid>
   ```

3. **Change backend port (if needed):**
   ```bash
   vim /app/index.js
   # Change: const PORT = 8080; to desired port
   
   # Also update service file and Nginx config
   systemctl restart backend
   ```

### Issue 5: Backend Memory Issues

**Symptoms:**
- "JavaScript heap out of memory"
- Backend crashes after some time
- Slow response times

**Solutions:**

1. **Check memory usage:**
   ```bash
   ps aux | grep node
   top -p $(pgrep -f "node /app")
   ```

2. **Increase Node memory:**
   ```bash
   vim /etc/systemd/system/backend.service
   # Change ExecStart to:
   # ExecStart=/bin/node --max-old-space-size=2048 /app/index.js
   
   systemctl daemon-reload
   systemctl restart backend
   ```

3. **Check for memory leaks in code:**
   ```bash
   journalctl -u backend | grep -i memory
   ```

---

## Frontend (Nginx) Issues

### Issue 1: Nginx Won't Start

**Symptoms:**
- `systemctl start nginx` fails
- Port 80 shows error
- Service status shows "failed"

**Solutions:**

1. **Check service status:**
   ```bash
   systemctl status nginx
   ```

2. **Test Nginx configuration:**
   ```bash
   nginx -t
   ```

3. **View error logs:**
   ```bash
   tail -f /var/log/nginx/error.log
   ```

4. **Check if port 80 is in use:**
   ```bash
   lsof -i :80
   netstat -tlnp | grep :80
   ```

5. **Fix configuration syntax errors:**
   - Run `nginx -t` to identify the line with errors
   - Edit `/etc/nginx/nginx.conf` or `/etc/nginx/default.d/expense.conf`
   - Restart: `systemctl restart nginx`

### Issue 2: Port 80 Already in Use

**Symptoms:**
- "Address already in use" error
- Another service listening on port 80

**Solutions:**

1. **Find what's using port 80:**
   ```bash
   lsof -i :80
   ```

2. **Stop conflicting service:**
   ```bash
   systemctl stop httpd
   systemctl disable httpd
   ```

3. **Kill process and start Nginx:**
   ```bash
   kill -9 <pid>
   systemctl start nginx
   ```

### Issue 3: Cannot Access Frontend

**Symptoms:**
- Browser shows "Connection refused"
- "Unable to connect" error
- Timeout when accessing http://<ip>/

**Solutions:**

1. **Verify Nginx is running:**
   ```bash
   systemctl status nginx
   ```

2. **Check if port 80 is listening:**
   ```bash
   netstat -tlnp | grep :80
   curl http://localhost/
   ```

3. **Check firewall:**
   ```bash
   firewall-cmd --list-all
   firewall-cmd --add-port=80/tcp --permanent
   firewall-cmd --reload
   ```

4. **Verify content exists:**
   ```bash
   ls -la /usr/share/nginx/html/
   ```

5. **Check Nginx logs:**
   ```bash
   tail -f /var/log/nginx/error.log
   tail -f /var/log/nginx/access.log
   ```

### Issue 4: API Proxy Not Working

**Symptoms:**
- Frontend loads but API calls fail
- "Cannot GET /api/" errors
- CORS errors in browser console

**Solutions:**

1. **Verify reverse proxy config:**
   ```bash
   cat /etc/nginx/default.d/expense.conf
   ```

2. **Test backend connectivity:**
   ```bash
   curl http://<backend-ip>:8080/
   ```

3. **Check if backend IP is correct:**
   ```bash
   # Update expense.conf with correct IP
   vim /etc/nginx/default.d/expense.conf
   nginx -t
   systemctl reload nginx
   ```

4. **Test proxy directly:**
   ```bash
   curl http://localhost/api/
   ```

5. **Check Nginx error logs:**
   ```bash
   tail -f /var/log/nginx/error.log
   ```

6. **Enable debug logging:**
   ```bash
   vim /etc/nginx/nginx.conf
   # Change: error_log /var/log/nginx/error.log debug;
   systemctl reload nginx
   ```

### Issue 5: Frontend Shows Blank Page

**Symptoms:**
- Browser loads but shows blank/empty content
- 404 errors in console
- JavaScript errors

**Solutions:**

1. **Verify files are extracted:**
   ```bash
   ls -la /usr/share/nginx/html/
   ```

2. **Check file permissions:**
   ```bash
   chmod -R 644 /usr/share/nginx/html/*
   chmod 755 /usr/share/nginx/html
   ```

3. **Check browser console for errors:**
   - Open Developer Tools (F12)
   - Check Console and Network tabs
   - Look for 404 or CORS errors

4. **Verify index.html exists:**
   ```bash
   cat /usr/share/nginx/html/index.html | head -20
   ```

5. **Check Nginx access logs:**
   ```bash
   tail -f /var/log/nginx/access.log
   ```

### Issue 6: Slow Frontend Response

**Symptoms:**
- Page loading takes too long
- High latency to API
- Browser shows long wait times

**Solutions:**

1. **Enable gzip compression:**
   ```bash
   vim /etc/nginx/nginx.conf
   # Add:
   gzip on;
   gzip_types text/plain text/css application/json;
   
   systemctl reload nginx
   ```

2. **Add caching headers:**
   ```bash
   vim /etc/nginx/default.d/expense.conf
   # Add to location block:
   expires 7d;
   add_header Cache-Control "public, max-age=604800";
   
   systemctl reload nginx
   ```

3. **Check backend response time:**
   ```bash
   time curl http://<backend-ip>:8080/
   ```

---

## Network & Connectivity Issues

### Issue 1: Servers Cannot Communicate

**Symptoms:**
- Services on different servers can't connect
- Timeout errors
- "Connection refused"

**Solutions:**

1. **Test connectivity between servers:**
   ```bash
   ping <target-ip>
   telnet <target-ip> <port>
   nc -zv <target-ip> <port>
   ```

2. **Check firewall rules:**
   ```bash
   # On receiving server:
   firewall-cmd --list-all
   firewall-cmd --add-port=<port>/tcp --permanent
   firewall-cmd --add-rich-rule='rule family="ipv4" source address="<source-ip>/32" port protocol="tcp" port="<port>" accept' --permanent
   firewall-cmd --reload
   ```

3. **Check routing:**
   ```bash
   ip route
   traceroute <target-ip>
   ```

4. **Verify DNS resolution:**
   ```bash
   nslookup <hostname>
   dig <hostname>
   ```

### Issue 2: SSL/HTTPS Issues

**Symptoms:**
- SSL certificate errors
- "Insecure connection" warnings
- HTTPS requests fail

**Solutions:**

1. **Install SSL certificate:**
   ```bash
   dnf install certbot python3-certbot-nginx -y
   certbot --nginx -d yourdomain.com
   ```

2. **Check certificate expiration:**
   ```bash
   openssl s_client -connect yourdomain.com:443 -servername yourdomain.com | grep -i "Validity\|notAfter"
   ```

3. **Renew certificate:**
   ```bash
   certbot renew
   ```

---

## Application Not Working Issues

### Issue 1: Cannot Add/View Expenses

**Symptoms:**
- Clicking "Add Expense" shows error
- Expense list is empty
- Form submission fails

**Solutions:**

1. **Check browser console:**
   - Open DevTools (F12)
   - Check Console tab for JavaScript errors
   - Check Network tab for failed API calls

2. **Test backend API:**
   ```bash
   curl http://<backend-ip>:8080/api/expenses
   ```

3. **Check database connection:**
   ```bash
   mysql -h <mysql-ip> -uroot -pExpenseApp@1 expense -e "SHOW TABLES;"
   mysql -h <mysql-ip> -uroot -pExpenseApp@1 expense -e "SELECT * FROM expenses;"
   ```

4. **Check backend logs:**
   ```bash
   journalctl -u backend -f
   ```

5. **Verify table structure:**
   ```bash
   mysql -h <mysql-ip> -uroot -pExpenseApp@1 expense -e "DESCRIBE expenses;"
   ```

### Issue 2: Login Issues

**Symptoms:**
- Cannot login to application
- "Invalid credentials" error
- Session not persisting

**Solutions:**

1. **Check users table:**
   ```bash
   mysql -h <mysql-ip> -uroot -pExpenseApp@1 expense -e "SELECT * FROM users;"
   ```

2. **Verify backend health:**
   ```bash
   curl http://<backend-ip>:8080/health
   ```

3. **Check backend logs for auth errors:**
   ```bash
   journalctl -u backend | grep -i auth
   ```

4. **Ensure cookies are enabled:**
   - Check browser cookie settings
   - Test with incognito window

---

## Performance Issues

### Issue 1: High CPU Usage

**Symptoms:**
- CPU at 100%
- System sluggish
- Processes running slow

**Solutions:**

1. **Identify heavy processes:**
   ```bash
   top
   ps aux --sort=-%cpu | head
   ```

2. **Check database performance:**
   ```bash
   mysql -uroot -pExpenseApp@1 -e "SHOW PROCESSLIST;"
   ```

3. **Optimize slow queries:**
   ```bash
   mysql -uroot -pExpenseApp@1 expense -e "SELECT * FROM information_schema.PROCESSLIST WHERE TIME > 10;"
   ```

4. **Add indexes:**
   ```bash
   mysql -uroot -pExpenseApp@1 expense -e "CREATE INDEX idx_user_id ON expenses(user_id);"
   ```

### Issue 2: High Memory Usage

**Symptoms:**
- System running out of RAM
- OOM (Out of Memory) errors
- Swapping to disk

**Solutions:**

1. **Check memory usage:**
   ```bash
   free -h
   top -b -o %MEM | head
   ```

2. **Find memory hogs:**
   ```bash
   ps aux --sort=-%mem | head
   ```

3. **Restart services:**
   ```bash
   systemctl restart backend
   systemctl restart nginx
   # MySQL will clean up automatically
   ```

4. **Increase swap space:**
   ```bash
   fallocate -l 2G /swapfile
   chmod 600 /swapfile
   mkswap /swapfile
   swapon /swapfile
   ```

### Issue 3: Slow Queries

**Symptoms:**
- API responses slow
- Database queries timeout
- Application hangs

**Solutions:**

1. **Enable slow query log:**
   ```bash
   mysql -uroot -pExpenseApp@1 -e "SET GLOBAL slow_query_log = 'ON';"
   mysql -uroot -pExpenseApp@1 -e "SET GLOBAL long_query_time = 2;"
   ```

2. **Check slow query log:**
   ```bash
   tail -f /var/log/mysql/slow_query.log
   ```

3. **Analyze query:**
   ```bash
   mysql -uroot -pExpenseApp@1 expense -e "EXPLAIN SELECT * FROM expenses WHERE user_id = 1;"
   ```

4. **Add appropriate indexes:**
   ```bash
   mysql -uroot -pExpenseApp@1 expense -e "CREATE INDEX idx_user_expense ON expenses(user_id, expense_date);"
   ```

---

## Diagnostic Commands Reference

### System Diagnostics

```bash
# System info
uname -a
cat /etc/os-release

# Resource usage
top
free -h
df -h

# Network
netstat -tlnp
ss -tlnp
```

### Database Diagnostics

```bash
# MySQL status
systemctl status mysqld
mysql -uroot -pExpenseApp@1 -e "STATUS;"

# Database info
mysql -uroot -pExpenseApp@1 -e "SHOW DATABASES;"
mysql -uroot -pExpenseApp@1 expense -e "SHOW TABLES;"

# Performance
mysql -uroot -pExpenseApp@1 -e "SHOW PROCESSLIST;"
mysql -uroot -pExpenseApp@1 -e "SHOW STATUS LIKE 'Threads%';"
```

### Backend Diagnostics

```bash
# Service status
systemctl status backend
journalctl -u backend -n 50

# Port check
lsof -i :8080
netstat -tlnp | grep 8080

# Process info
ps aux | grep node
```

### Frontend Diagnostics

```bash
# Service status
systemctl status nginx
journalctl -u nginx -n 50

# Configuration
nginx -t
cat /etc/nginx/nginx.conf

# Port check
lsof -i :80
netstat -tlnp | grep :80

# Logs
tail -f /var/log/nginx/error.log
tail -f /var/log/nginx/access.log
```

---

## Getting Help

If you can't resolve an issue:

1. **Collect logs:**
   ```bash
   journalctl -u backend > backend.log
   journalctl -u nginx > nginx.log
   tail -100 /var/log/mysql/error.log > mysql.log
   ```

2. **Document the issue:**
   - What were you trying to do?
   - What error message did you see?
   - What OS and versions are you using?
   - Attach collected logs

3. **Contact support with:**
   - Screenshots of errors
   - Relevant log files
   - Output of diagnostic commands
   - Steps to reproduce the issue

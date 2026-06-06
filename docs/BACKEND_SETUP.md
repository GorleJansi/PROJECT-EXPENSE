# Backend Setup Guide (NodeJS)

## Overview

The Expense backend is a NodeJS application that provides REST APIs, handles business logic, and communicates with the MySQL database. This guide covers installation, configuration, and deployment.

## Prerequisites

- Linux OS (RHEL/CentOS/Fedora)
- Root or sudo access
- DNF package manager
- MySQL database already installed and running
- Internet connectivity for downloads

## Architecture

```
NodeJS Backend
├── Purpose: Business logic & APIs
├── Port: 8080
├── User: expense (daemon)
├── Location: /app
├── Service: SystemD (backend.service)
└── Dependencies: npm packages
```

## Installation Steps

### Step 1: Install NodeJS >20

#### Check Available NodeJS Modules
```bash
dnf module list nodejs
```

**Output shows available versions**

#### Disable Default NodeJS (16)
```bash
dnf module disable nodejs -y
```

**What this does:**
- Prevents installation of older NodeJS 16
- Allows us to select version 20 specifically

#### Enable NodeJS 20
```bash
dnf module enable nodejs:20 -y
```

**What this does:**
- Makes NodeJS version 20 available for installation
- `-y` flag auto-confirms

#### Install NodeJS
```bash
dnf install nodejs -y
```

**What this does:**
- Installs NodeJS version 20 and npm package manager
- Provides `node` and `npm` commands

#### Verify Installation
```bash
node --version
npm --version
```

**Expected output:**
```
v20.x.x
x.x.x
```

### Step 2: Create Application User

```bash
useradd expense
```

**What this does:**
- Creates a system daemon user called `expense`
- User has no login shell (cannot login directly)
- Used to run application in background
- Provides security isolation for the service

**Why daemon user:**
- Applications shouldn't run as root
- Limits privileges if application is compromised
- Better for system management and logging

### Step 3: Setup Application Directory

```bash
mkdir /app
```

**What this does:**
- Creates standard application directory
- Follows organization best practices
- Centralized location for all application code

### Step 4: Download Backend Application

```bash
curl -o /tmp/backend.zip https://expense-joindevops.s3.us-east-1.amazonaws.com/expense-backend-v2.zip
```

**What this does:**
- Downloads backend source code from S3
- Saves to `/tmp/backend.zip`
- `-o` flag specifies output filename

```bash
cd /app
```

```bash
unzip /tmp/backend.zip
```

**What this does:**
- Extracts application files
- Creates directory structure
- Prepares for configuration

### Step 5: Install Dependencies

```bash
cd /app
npm install
```

**What this does:**
- Reads `package.json` file
- Downloads all required npm packages
- Creates `node_modules` directory
- Builds dependency tree

**Time required:** 2-5 minutes depending on connection

## Configuration

### Step 1: Create SystemD Service File

```bash
sudo vim /etc/systemd/system/backend.service
```

**File content:**
```ini
[Unit]
Description = Backend Service

[Service]
User=expense
Environment=DB_HOST="<MYSQL-SERVER-IPADDRESS>"
ExecStart=/bin/node /app/index.js
SyslogIdentifier=backend

[Install]
WantedBy=multi-user.target
```

**Configuration Details:**

| Parameter | Purpose |
|-----------|---------|
| `Description` | Human-readable service name |
| `User=expense` | Run service as expense user |
| `DB_HOST` | Database server IP address |
| `ExecStart` | Command to start the application |
| `SyslogIdentifier` | Logs appear with this tag |
| `WantedBy` | Multi-user target (standard system mode) |

**⚠️ Important:** Replace `<MYSQL-SERVER-IPADDRESS>` with actual MySQL server IP

### Step 2: Set Correct Permissions

```bash
chown -R expense:expense /app
```

**What this does:**
- Changes ownership of `/app` directory to `expense` user
- Allows daemon user to read and execute files

### Step 3: Load Service Configuration

```bash
systemctl daemon-reload
```

**What this does:**
- Tells SystemD to re-read service files
- Registers new backend.service
- Makes service available to systemctl

### Step 4: Start Backend Service

```bash
systemctl start backend
```

**What this does:**
- Starts the backend service immediately
- Runs as `expense` user
- Listens on port 8080

### Step 5: Enable Auto-Start

```bash
systemctl enable backend
```

**What this does:**
- Enables service to start on system boot
- Ensures backend is always running after restart

## Database Schema

### Load Schema

```bash
mysql -h <MYSQL-SERVER-IPADDRESS> -uroot -pExpenseApp@1 < /app/schema/backend.sql
```

**What this does:**
- Connects to MySQL database
- Executes SQL script
- Creates necessary tables
- Populates initial data

**Prerequisites:**
- MySQL server must be running
- Root password must be `ExpenseApp@1`
- Replace `<MYSQL-SERVER-IPADDRESS>` with actual MySQL IP

### Restart Backend

```bash
systemctl restart backend
```

**What this does:**
- Restarts backend service
- Reconnects to database
- Applies any schema changes

## Management Commands

### Check Service Status
```bash
systemctl status backend
```

**Output shows:**
- Running/Stopped status
- Process ID
- Recent log entries

### View Service Logs
```bash
journalctl -u backend -f
```

**What this does:**
- Shows real-time logs from backend service
- `-f` flag follows new entries
- Press `Ctrl+C` to exit

### Stop Service
```bash
systemctl stop backend
```

### Restart Service
```bash
systemctl restart backend
```

### View Last 50 Log Lines
```bash
journalctl -u backend -n 50
```

## Verification

### Test Backend Health

```bash
curl http://localhost:8080/health
```

**Expected response:** Service health status

### Test API Endpoint (Example)
```bash
curl http://localhost:8080/api/expenses
```

### Check Port Listening
```bash
netstat -tlnp | grep 8080
```

**Expected output:**
```
tcp    0    0    127.0.0.1:8080    0.0.0.0:*    LISTEN    <pid>/node
```

### Check Process Running
```bash
ps aux | grep node
```

**Expected:** Shows backend.js process running as `expense` user

## Troubleshooting

### Service Won't Start

1. **Check logs:**
   ```bash
   journalctl -u backend -n 50
   ```

2. **Check if port is in use:**
   ```bash
   lsof -i :8080
   ```

3. **Verify configuration:**
   ```bash
   cat /etc/systemd/system/backend.service
   ```

### Database Connection Issues

1. **Test MySQL connectivity:**
   ```bash
   mysql -h <db-ip> -uroot -pExpenseApp@1 -e "select version();"
   ```

2. **Check DB_HOST in service file:**
   ```bash
   grep DB_HOST /etc/systemd/system/backend.service
   ```

3. **Verify MySQL is running:**
   ```bash
   systemctl status mysqld
   ```

### Node Modules Issues

1. **Reinstall dependencies:**
   ```bash
   cd /app
   rm -rf node_modules package-lock.json
   npm install
   ```

2. **Check Node version:**
   ```bash
   node --version
   ```

## Performance Optimization

### Increase File Descriptors
```bash
vim /etc/security/limits.conf
```

Add:
```
expense soft nofile 10000
expense hard nofile 10000
```

### Enable Process Manager (PM2 - Optional)

```bash
npm install -g pm2
pm2 start /app/index.js --name backend
pm2 save
```

## Environment Variables

### Available Variables
- `DB_HOST` - MySQL database host
- `NODE_ENV` - Environment (production/development)
- `PORT` - Backend port (default: 8080)

### Set in Service File
```ini
Environment=DB_HOST="192.168.1.10"
Environment=NODE_ENV="production"
Environment=PORT="8080"
```

## Security Considerations

1. **Run as non-root user** ✓ (expense user)
2. **Restrict file permissions** ✓ (expense user ownership)
3. **Use environment variables** - Don't hardcode credentials
4. **Limit network access** - Use firewall rules
5. **Regular backups** - Backup application code
6. **Monitor resource usage** - CPU, memory, disk

## References

- [NodeJS Documentation](https://nodejs.org/en/docs/)
- [NPM Documentation](https://docs.npmjs.com/)
- [SystemD Service Documentation](https://www.freedesktop.org/software/systemd/man/systemd.service.html)

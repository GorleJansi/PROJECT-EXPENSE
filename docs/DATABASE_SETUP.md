# MySQL Database Setup Guide

## Overview

The Expense application uses MySQL 8.0.x as its data storage layer. This guide covers installation, configuration, and verification.

## Prerequisites

- Linux OS (RHEL/CentOS/Fedora)
- Root or sudo access
- DNF package manager

## Installation

### Step 1: Install MySQL Server 8.0.x

```bash
dnf install mysql-server -y
```

**What this does:**
- Downloads and installs MySQL server package
- Prepares system for database operations
- `-y` flag automatically answers "yes" to prompts

### Step 2: Enable and Start MySQL Service

```bash
systemctl enable mysqld
```

**What this does:**
- Enables MySQL to start automatically on system boot
- Ensures database is always available

```bash
systemctl start mysqld
```

**What this does:**
- Starts the MySQL service immediately
- Database is now ready to accept connections

### Step 3: Set Root Password

```bash
mysql_secure_installation --set-root-pass ExpenseApp@1
```

**What this does:**
- Sets the root user password to `ExpenseApp@1`
- Replaces the default empty password
- Secures the installation
- `--set-root-pass` flag directly sets the password (non-interactive mode)

**Password Details:**
- Username: `root`
- Password: `ExpenseApp@1`
- Keep this secure and share only with authorized administrators

## Verification

### Connect to MySQL

#### Local Connection (Same Server)
```bash
mysql
```

#### Remote Connection (Different Server)
```bash
mysql -h <mysql-server-ip> -u root -pExpenseApp@1
```

**Connection Syntax:**
- `-h` : Host/IP address of MySQL server
- `-u` : Username (root)
- `-p` : Password flag (no space between `-p` and password)

### Verify Installation

Once connected to MySQL prompt (you'll see `mysql>`):

**List all databases:**
```sql
show databases;
```

**Expected output:**
```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```

## Database Schema

### Loading Application Schema

The application requires specific tables. Load the schema from backend application:

```bash
mysql -h <MYSQL-SERVER-IPADDRESS> -uroot -pExpenseApp@1 < /app/schema/backend.sql
```

**What this does:**
- Connects to MySQL server
- Executes SQL script from backend application
- Creates necessary tables and schemas
- Populates default data if needed

### Verify Schema Loaded

```bash
mysql -h <mysql-server-ip> -uroot -pExpenseApp@1
```

```sql
-- List application database
show databases;

-- Connect to expense database
use expense;

-- List tables
show tables;

-- View sample data
select * from <table_name>;
```

## Common MySQL Commands

### User Management

**Create new user:**
```sql
CREATE USER 'expense_app'@'<backend-ip>' IDENTIFIED BY 'password123';
```

**Grant privileges:**
```sql
GRANT ALL PRIVILEGES ON expense.* TO 'expense_app'@'<backend-ip>';
FLUSH PRIVILEGES;
```

**List users:**
```sql
SELECT user, host FROM mysql.user;
```

### Database Operations

**Create database:**
```sql
CREATE DATABASE expense;
```

**View database size:**
```sql
SELECT table_name, 
       ROUND(((data_length + index_length) / 1024 / 1024), 2) AS size_mb
FROM information_schema.TABLES
WHERE table_schema = 'expense';
```

### Backup and Restore

**Backup database:**
```bash
mysqldump -h <mysql-ip> -uroot -pExpenseApp@1 expense > expense_backup.sql
```

**Restore database:**
```bash
mysql -h <mysql-ip> -uroot -pExpenseApp@1 expense < expense_backup.sql
```

## Troubleshooting

### Check MySQL Service Status
```bash
systemctl status mysqld
```

### View MySQL Logs
```bash
tail -f /var/log/mysql/error.log
```

### Reset Root Password (If Forgotten)
```bash
systemctl stop mysqld
mysqld_safe --skip-grant-tables &
mysql -u root
```

## Security Considerations

1. **Change default password** - Done with `mysql_secure_installation`
2. **Restrict remote access** - Use firewall rules
3. **Use strong passwords** - `ExpenseApp@1` should be changed in production
4. **Limit user privileges** - Create limited user for application
5. **Regular backups** - Automate database backups
6. **Monitor connections** - Check for suspicious access

## Performance Tuning

### Basic Configuration
Edit `/etc/my.cnf`:

```ini
[mysqld]
# Connection settings
max_connections = 100
max_allowed_packet = 256M

# Performance settings
innodb_buffer_pool_size = 256M
innodb_log_file_size = 100M
```

Restart MySQL:
```bash
systemctl restart mysqld
```

## Monitoring

### Check running processes
```bash
mysql -h <mysql-ip> -uroot -pExpenseApp@1 -e "SHOW PROCESSLIST;"
```

### Check variable status
```bash
mysql -h <mysql-ip> -uroot -pExpenseApp@1 -e "SHOW STATUS;"
```

## References

- [MySQL Official Documentation](https://dev.mysql.com/doc/)
- [MySQL 8.0 Reference Manual](https://dev.mysql.com/doc/refman/8.0/en/)

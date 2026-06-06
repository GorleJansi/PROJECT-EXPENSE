# Expense Project - Complete Documentation Index

## 📚 Documentation Overview

Welcome to the Expense Project documentation. This index helps you find exactly what you need.

---

## 🚀 Getting Started

### For First-Time Users
1. **Start here:** [README.md](../README.md) - Project overview and structure
2. **Quick setup:** [QUICKSTART.md](../QUICKSTART.md) - One-page setup guide with commands

### Understanding the System
1. **System design:** [ARCHITECTURE.md](ARCHITECTURE.md) - How components work together
2. **Data flow:** [ARCHITECTURE.md](ARCHITECTURE.md#data-flow) - Request and response flow

---

## 🔧 Setup Guides

### Database Setup
📄 **File:** [DATABASE_SETUP.md](DATABASE_SETUP.md)

**Topics Covered:**
- MySQL Server 8.0.x installation
- Service configuration
- Root password setup
- Verification and testing
- User management
- Backup and restore
- Performance tuning
- Security considerations

**Key Commands:**
```bash
dnf install mysql-server -y
systemctl enable mysqld && systemctl start mysqld
mysql_secure_installation --set-root-pass ExpenseApp@1
```

---

### Backend Setup
📄 **File:** [BACKEND_SETUP.md](BACKEND_SETUP.md)

**Topics Covered:**
- NodeJS >20 installation
- Application user setup
- Code download and deployment
- NPM dependency installation
- SystemD service configuration
- Database schema loading
- Service management
- Environment variables
- Security hardening

**Key Commands:**
```bash
dnf module enable nodejs:20 -y && dnf install nodejs -y
useradd expense
curl -o /tmp/backend.zip https://expense-joindevops.s3.us-east-1.amazonaws.com/expense-backend-v2.zip
cd /app && unzip /tmp/backend.zip && npm install
```

---

### Frontend Setup
📄 **File:** [FRONTEND_SETUP.md](FRONTEND_SETUP.md)

**Topics Covered:**
- Nginx Web Server installation
- Static content deployment
- Reverse proxy configuration
- API routing setup
- Service management
- SSL/HTTPS setup
- Performance optimization
- Security headers

**Key Commands:**
```bash
dnf install nginx -y
systemctl enable nginx && systemctl start nginx
curl -o /tmp/frontend.zip https://expense-joindevops.s3.us-east-1.amazonaws.com/expense-frontend-v2.zip
cd /usr/share/nginx/html && unzip /tmp/frontend.zip
```

---

## 🐛 Troubleshooting

📄 **File:** [TROUBLESHOOTING.md](TROUBLESHOOTING.md)

### Issues by Component

| Component | Common Issues | Location in Guide |
|-----------|---------------|------------------|
| **MySQL** | Won't start, connection failed, schema issues | DATABASE_SETUP.md |
| **Backend** | Service won't start, DB connection error, port in use | BACKEND_SETUP.md |
| **Frontend** | Nginx won't start, API proxy not working, blank page | FRONTEND_SETUP.md |
| **General** | Network issues, SSL problems, performance | TROUBLESHOOTING.md |

### Quick Troubleshooting Links

**Database Issues:**
- [MySQL Won't Start](TROUBLESHOOTING.md#issue-1-mysql-service-wont-start)
- [Cannot Connect to MySQL](TROUBLESHOOTING.md#issue-2-cannot-connect-to-mysql)
- [Schema Not Loading](TROUBLESHOOTING.md#issue-3-schema-not-loading)
- [Database Full](TROUBLESHOOTING.md#issue-4-database-running-out-of-space)

**Backend Issues:**
- [Backend Won't Start](TROUBLESHOOTING.md#issue-1-backend-service-wont-start)
- [DB Connection Failed](TROUBLESHOOTING.md#issue-2-cannot-connect-to-database-from-backend)
- [Node Modules Missing](TROUBLESHOOTING.md#issue-3-nodejs-modules-not-found)
- [Port Already in Use](TROUBLESHOOTING.md#issue-4-backend-port-already-in-use)
- [Memory Issues](TROUBLESHOOTING.md#issue-5-backend-memory-issues)

**Frontend Issues:**
- [Nginx Won't Start](TROUBLESHOOTING.md#issue-1-nginx-wont-start)
- [Port 80 in Use](TROUBLESHOOTING.md#issue-2-port-80-already-in-use)
- [Cannot Access Frontend](TROUBLESHOOTING.md#issue-3-cannot-access-frontend)
- [API Proxy Not Working](TROUBLESHOOTING.md#issue-4-api-proxy-not-working)
- [Blank Page](TROUBLESHOOTING.md#issue-5-frontend-shows-blank-page)
- [Slow Performance](TROUBLESHOOTING.md#issue-6-slow-frontend-response)

**Application Issues:**
- [Cannot Add/View Expenses](TROUBLESHOOTING.md#issue-1-cannot-addview-expenses)
- [Login Problems](TROUBLESHOOTING.md#issue-2-login-issues)

---

## 🎨 User Interface

📄 **File:** [UI_GUIDE.md](UI_GUIDE.md)

**Topics Covered:**
- Application screenshots
- UI components and layout
- User workflows
- API endpoints
- Styling and theming
- Form fields and inputs
- Responsive design
- Accessibility features
- Performance optimization
- Browser compatibility
- Testing checklist

---

## 📋 Configuration Files

### Backend Service File
📄 **File:** [config/backend.service.example](../config/backend.service.example)

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

**Usage:** Copy to `/etc/systemd/system/backend.service` and update `DB_HOST`

---

### Nginx Configuration
📄 **File:** [config/expense.conf.example](../config/expense.conf.example)

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

**Usage:** Copy to `/etc/nginx/default.d/expense.conf` and update `<BACKEND-SERVER-IP>`

---

### Database Schema
📄 **File:** [config/schema.sql.example](../config/schema.sql.example)

Contains:
- Database creation
- User table schema
- Categories table
- Expenses table with relationships
- Indexes for performance
- Sample data

**Usage:** Reference for understanding database structure. Actual schema loads from `/app/schema/backend.sql`

---

## 🔍 Architecture Deep Dive

📄 **File:** [ARCHITECTURE.md](ARCHITECTURE.md)

### System Design
- 3-tier architecture overview
- Tier 1: Frontend (Nginx)
- Tier 2: Backend (NodeJS)
- Tier 3: Database (MySQL)

### Communication Flow
- User request → Frontend
- Frontend → Backend API
- Backend → Database
- Response chain back to user

### Integration Points
- Frontend ↔ Backend (HTTP/HTTPS)
- Backend ↔ Database (MySQL protocol)
- Network requirements and ports

### Deployment Considerations
- High availability
- Scalability
- Security architecture
- Firewall rules

---

## 📊 Quick Reference

### Important IPs and Ports

| Component | Port | Default IP | Purpose |
|-----------|------|-----------|---------|
| **MySQL** | 3306 | Database server | Data storage |
| **Backend** | 8080 | Backend server | API service |
| **Frontend** | 80 | Any (public) | Web UI |
| **HTTPS** | 443 | Any (public) | Secure web |

### Default Credentials

| Service | User | Password | Notes |
|---------|------|----------|-------|
| **MySQL** | root | ExpenseApp@1 | Change in production |
| **App User** | expense | (none) | Daemon user, no login |

### File Locations

| Purpose | Path |
|---------|------|
| Backend code | `/app/` |
| Database files | `/var/lib/mysql/` |
| Nginx config | `/etc/nginx/` |
| Static content | `/usr/share/nginx/html/` |
| SystemD service | `/etc/systemd/system/` |
| Logs - MySQL | `/var/log/mysql/` |
| Logs - Backend | `journalctl -u backend` |
| Logs - Frontend | `/var/log/nginx/` |

---

## 🛠️ Common Tasks

### Service Management

**Start services:**
```bash
systemctl start mysqld
systemctl start backend
systemctl start nginx
```

**Check status:**
```bash
systemctl status mysqld
systemctl status backend
systemctl status nginx
```

**View logs:**
```bash
journalctl -u backend -f
tail -f /var/log/nginx/error.log
tail -f /var/log/mysql/error.log
```

### Database Operations

**Connect to database:**
```bash
mysql -h <mysql-ip> -uroot -pExpenseApp@1
```

**Backup database:**
```bash
mysqldump -h <mysql-ip> -uroot -pExpenseApp@1 expense > backup.sql
```

**Restore database:**
```bash
mysql -h <mysql-ip> -uroot -pExpenseApp@1 expense < backup.sql
```

### Backend Operations

**Check API health:**
```bash
curl http://<backend-ip>:8080/health
```

**Restart service:**
```bash
systemctl restart backend
journalctl -u backend -f  # View logs
```

### Frontend Operations

**Test frontend:**
```bash
curl http://<frontend-ip>/
```

**Test API proxy:**
```bash
curl http://<frontend-ip>/api/health
```

**Reload config:**
```bash
nginx -t  # Test first
systemctl reload nginx
```

---

## 📈 Performance & Optimization

### Database Optimization
- See [DATABASE_SETUP.md - Performance Tuning](DATABASE_SETUP.md#performance-tuning)
- Index creation
- Query optimization
- Buffer pool configuration

### Backend Optimization
- See [BACKEND_SETUP.md - Performance Optimization](BACKEND_SETUP.md#performance-optimization)
- Worker processes
- Memory management
- Load balancing

### Frontend Optimization
- See [FRONTEND_SETUP.md - Performance Tuning](FRONTEND_SETUP.md#performance-tuning)
- Gzip compression
- Caching headers
- File optimization

---

## 🔒 Security Hardening

### Database Security
- ✅ Strong root password
- ✅ Limited user privileges
- ✅ Firewall rules
- ✅ Regular backups
- See [DATABASE_SETUP.md - Security](DATABASE_SETUP.md#security-considerations)

### Backend Security
- ✅ Non-root daemon user
- ✅ File permissions
- ✅ Environment variables for secrets
- See [BACKEND_SETUP.md - Security](BACKEND_SETUP.md#security-considerations)

### Frontend Security
- ✅ HTTPS/SSL setup
- ✅ Security headers
- ✅ Request size limits
- See [FRONTEND_SETUP.md - Security](FRONTEND_SETUP.md#security-considerations)

---

## 🧪 Testing & Verification

### Verification Checklist

- [ ] MySQL running and accessible
- [ ] Backend service started and responding
- [ ] Frontend accessible via web browser
- [ ] API proxy working (frontend can reach backend)
- [ ] Database schema loaded
- [ ] Expenses can be added/viewed
- [ ] No console errors in browser
- [ ] Logs show no errors

### Manual Testing
```bash
# Test each component
curl http://<backend-ip>:8080/health
curl http://<frontend-ip>/
curl http://<frontend-ip>/api/health

# Check database
mysql -h <mysql-ip> -uroot -pExpenseApp@1 -e "use expense; select * from expenses;"
```

---

## 📞 Getting Help

### Before Asking for Help

1. **Check logs:**
   ```bash
   journalctl -u backend -n 50 > backend.log
   journalctl -u nginx -n 50 > nginx.log
   tail -100 /var/log/mysql/error.log > mysql.log
   ```

2. **Run diagnostics:**
   - See [TROUBLESHOOTING.md - Diagnostic Commands](TROUBLESHOOTING.md#diagnostic-commands-reference)

3. **Check relevant guide:**
   - Find your issue in [TROUBLESHOOTING.md](TROUBLESHOOTING.md)
   - Review component setup guide

### Collect Information

When reporting issues, provide:
- Screenshots of errors
- Relevant log files
- OS and software versions
- Steps to reproduce
- Output of diagnostic commands
- System resources (top, free -h, df -h)

---

## 📖 Documentation Structure

```
docs/
├── INDEX.md (this file)           # Navigation and overview
├── README.md                       # Project introduction
├── QUICKSTART.md                   # Fast setup guide
├── ARCHITECTURE.md                 # System design
├── DATABASE_SETUP.md               # MySQL guide
├── BACKEND_SETUP.md                # NodeJS guide
├── FRONTEND_SETUP.md               # Nginx guide
├── TROUBLESHOOTING.md              # Problem solving
└── UI_GUIDE.md                     # Application features
```

---

## 🔄 Workflow Summary

```
1. Plan & Design
   └─→ Review ARCHITECTURE.md

2. Setup Components (in order)
   ├─→ Database: DATABASE_SETUP.md
   ├─→ Backend: BACKEND_SETUP.md
   └─→ Frontend: FRONTEND_SETUP.md

3. Verify Installation
   └─→ Run verification commands
   └─→ Test each component

4. Daily Operations
   └─→ SERVICE MANAGEMENT commands

5. Troubleshooting
   └─→ Find issue in TROUBLESHOOTING.md
   └─→ Apply solution
   └─→ Verify fix

6. Optimization
   └─→ Review performance tuning sections
   └─→ Apply optimizations
   └─→ Monitor and test
```

---

## 📚 External Resources

- [MySQL Documentation](https://dev.mysql.com/doc/)
- [NodeJS Documentation](https://nodejs.org/en/docs/)
- [Nginx Documentation](https://nginx.org/en/docs/)
- [SystemD Documentation](https://www.freedesktop.org/software/systemd/man/)
- [DNF Package Manager](https://dnf.readthedocs.io/)

---

## 📝 Documentation Updates

**Last Updated:** June 6, 2026

### Version History
- **v1.0** - Initial documentation with all setup guides, architecture, and troubleshooting

### Contributing
To update this documentation:
1. Edit relevant file
2. Test changes
3. Commit with clear message
4. Push to repository

---

## 📞 Support & Contact

For issues or questions:
1. Check [TROUBLESHOOTING.md](TROUBLESHOOTING.md)
2. Review relevant setup guide
3. Check [UI_GUIDE.md](UI_GUIDE.md) for feature questions
4. Collect logs and diagnostic info
5. Contact DevOps team with information

---

**Navigation:** [← Back to README](../README.md) | [Quick Start →](../QUICKSTART.md)

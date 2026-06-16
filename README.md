# Expense Project - Basic Flow

A three-tier application demonstrating DevOps practices with MySQL database, NodeJS backend, and Nginx frontend.

## Project Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Frontend (Nginx)                      │
│          Serves static web content on port 80           │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│              Backend (NodeJS) on port 8080               │
│        Handles API requests & database operations        │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│                  MySQL Database                          │
│              Stores application data                     │
└─────────────────────────────────────────────────────────┘
```

## Project Components

1. **Database (MySQL 8.0.x)** - Stores expense data
2. **Backend (NodeJS >20)** - REST API service
3. **Frontend (Nginx)** - Web UI for expense management

## Directory Structure

```
Expense-Basic-Flow/
├── README.md                          # Project overview
├── QUICKSTART.md                      # Quick start guide
├── docs/                              # Documentation
│   ├── ARCHITECTURE.md               # System design
│   ├── DATABASE_SETUP.md             # MySQL setup guide
│   ├── BACKEND_SETUP.md              # NodeJS backend guide
│   ├── FRONTEND_SETUP.md             # Nginx frontend guide
│   ├── TROUBLESHOOTING.md            # Troubleshooting guide
│   └── UI_GUIDE.md                   # UI features & screenshots
├── config/                            # Configuration files
│   ├── backend.service.example       # SystemD service file
│   ├── expense.conf.example          # Nginx configuration
│   └── schema.sql.example            # Database schema
├── screenshots/                       # UI screenshots
│   ├── expense-app-home.png          # Login/home page
│   └── add-view-expenses.png         # Expenses management
├── .gitignore                         # Git ignore file
└── .git/                              # Git repository
```

## Quick Start

### Prerequisites
- Linux OS (RedHat/CentOS/Fedora)
- Root or sudo access
- Internet connectivity

### Installation Steps

1. **Database Setup** - See [DATABASE_SETUP.md](docs/DATABASE_SETUP.md)
2. **Backend Setup** - See [BACKEND_SETUP.md](docs/BACKEND_SETUP.md)
3. **Frontend Setup** - See [FRONTEND_SETUP.md](docs/FRONTEND_SETUP.md)


## Ports

- Frontend: `80` (HTTP)
- Backend: `8080` (Internal API)
- MySQL: `3306` (Default)

## Verification

After setup, verify all components:
```bash
# Check MySQL
mysql -h <mysql-ip> -uroot -pExpenseApp@1 -e "show databases;"

# Check Backend
curl http://<backend-ip>:8080/health

# Check Frontend
curl http://<frontend-ip>/
```

## Troubleshooting

For common issues and solutions, see [TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md):
- MySQL/Database issues
- Backend (NodeJS) issues
- Frontend (Nginx) issues
- Network & connectivity issues
- Application problems
- Performance optimization

## UI & Screenshots

For information about the application UI and features, see [UI_GUIDE.md](docs/UI_GUIDE.md):
- Application screenshots
- UI components and features
- User workflows
- API integration points
- Browser compatibility

## Git Management

Track changes and maintain version control:
```bash
git add .
git commit -m "Initial Expense project setup"
git push origin main
```

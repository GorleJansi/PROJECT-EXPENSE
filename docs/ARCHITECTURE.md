# System Architecture

## Overview

The Expense application follows a classic three-tier architecture:

### Tier 1: Presentation Layer (Frontend)
- **Technology:** Nginx Web Server
- **Purpose:** Serves static web content (HTML, CSS, JavaScript)
- **Port:** 80 (HTTP)
- **Location:** `/usr/share/nginx/html`
- **Responsibilities:**
  - Serve UI to end users
  - Route API requests to backend via reverse proxy
  - Provide health check endpoint

### Tier 2: Application Layer (Backend)
- **Technology:** NodeJS (Version >20)
- **Purpose:** Business logic and API endpoints
- **Port:** 8080 (Internal)
- **Location:** `/app`
- **User:** `expense` (daemon user)
- **Responsibilities:**
  - Handle API requests from frontend
  - Process business logic
  - Communicate with database
  - Manage database transactions

### Tier 3: Data Layer (Database)
- **Technology:** MySQL 8.0.x
- **Purpose:** Persistent data storage
- **Port:** 3306 (Default)
- **Root User Password:** `ExpenseApp@1`
- **Responsibilities:**
  - Store expense records
  - Maintain data integrity
  - Handle data queries from backend

## Data Flow

1. **User Request:** User opens browser and accesses Frontend (Nginx)
2. **Static Content:** Nginx serves HTML/CSS/JavaScript to browser
3. **API Request:** Browser JavaScript sends API request to Nginx
4. **Reverse Proxy:** Nginx forwards request to Backend (NodeJS) on localhost:8080
5. **Backend Processing:** NodeJS processes the request
6. **Database Query:** Backend queries MySQL database
7. **Response Chain:** Data flows back: MySQL → Backend → Nginx → Browser

## Communication Diagram

```
┌─────────────────────┐
│   User Browser      │
│  (Port 80)          │
└──────────┬──────────┘
           │
           │ HTTP (Static Content)
           │ HTTP (API Requests)
           │
┌──────────▼──────────────┐
│   Nginx Frontend        │
│   Port 80               │
│   Reverse Proxy Config  │
└──────────┬──────────────┘
           │
           │ Proxy Pass to localhost:8080
           │
┌──────────▼──────────────┐
│   NodeJS Backend        │
│   Port 8080 (localhost) │
│   Daemon User: expense  │
└──────────┬──────────────┘
           │
           │ MySQL Protocol (Port 3306)
           │
┌──────────▼──────────────┐
│   MySQL Database        │
│   Port 3306             │
│   Root User: root       │
└─────────────────────────┘
```

## Key Integration Points

### Frontend ↔ Backend
- **Protocol:** HTTP/HTTPS
- **Endpoint:** `http://<backend-ip>:8080/`
- **Configuration:** Nginx reverse proxy in `/etc/nginx/default.d/expense.conf`
- **Path:** All `/api/*` requests proxied to backend

### Backend ↔ Database
- **Protocol:** MySQL native protocol
- **Host:** Database server IP address
- **User:** root (for administrative tasks)
- **Connection String:** `mysql -h <db-ip> -uroot -pExpenseApp@1`
- **Environment Variable:** `DB_HOST` in backend service file

## Service Dependencies

```
Nginx (Frontend) → depends on → Backend API
Backend → depends on → MySQL Database
```

All three services must be running for the application to function properly.

## Network Requirements

| Service | Port | Network | Access From |
|---------|------|---------|-------------|
| Nginx | 80 | Public | Internet |
| Backend | 8080 | Private | Nginx only |
| MySQL | 3306 | Private | Backend only |

## Deployment Considerations

- **High Availability:** Each tier can be deployed separately
- **Scalability:** Backend can have multiple instances behind load balancer
- **Security:** Database should not be exposed to frontend directly
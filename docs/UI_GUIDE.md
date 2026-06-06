# UI Screenshots & Visual Guide

## Application Screenshots

### 1. Login/Home Page
**File:** `screenshots/expense-app-home.png`

**Description:**
- Application title: "EXPENSE APP"
- Tagline: "Online Expense Tracker System"
- Description: "Track your expenses online"
- Dashboard with grid pattern background
- User icon for login/profile
- Professional dark theme with cyan/green accents

**Purpose:**
- Initial landing page
- Login/authentication screen
- Navigation hub to application features

---

### 2. Add/View Expenses Page
**File:** `screenshots/add-view-expenses.png`

**Description:**
- Header: "ADD/VIEW EXPENSES"
- Navigation menu on left side:
  - 🏠 HOME
  - 📋 ADD EXPENSE
- Main content area with table showing:
  - Columns: ID, AMOUNT, DESC
  - Row with sample data: ID: 5, AMOUNT: 10, DESC: ten
  - ADD button for adding new expenses
  - DEL button for deleting expenses
- Input fields for AMOUNT and description
- Blue/cyan color scheme for borders and highlights

**Purpose:**
- View existing expenses in table format
- Add new expenses via form
- Delete expenses via DEL button
- Manage expense records

---

## UI Features Overview

### Frontend Components

```
┌─────────────────────────────────────────┐
│         EXPENSE APP                     │
│  Online Expense Tracker System          │
│  Track your expenses online             │
│                                         │
│  [LOGIN / DASHBOARD]                    │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│  Navigation    │  Main Content          │
│  ──────────    │  ──────────────        │
│  🏠 HOME       │  ADD/VIEW EXPENSES     │
│  📋 ADD EXP    │                        │
│                │  ID | AMT | DESC | DEL│
│                │  ─────────────────────│
│                │  5  | 10  | ten  | ✓  │
│                │  ─────────────────────│
│                │  [Input Fields]       │
│                │  [ADD Button]         │
└─────────────────────────────────────────┘
```

### User Workflow

```
1. User visits application
         ↓
2. Sees login page (EXPENSE APP - Home)
         ↓
3. Authenticates with credentials
         ↓
4. Navigates to Add/View Expenses
         ↓
5. Views expense table
         ↓
6. Can:
   - Add new expense (fill form + click ADD)
   - Delete existing expense (click DEL)
   - View all expenses in table
```

---

## API Integration Points

### Frontend → Backend Communication

**API Endpoints Called by Frontend:**

| Page | Endpoint | Method | Purpose |
|------|----------|--------|---------|
| Home | `/api/auth/login` | POST | User authentication |
| Expenses | `/api/expenses` | GET | Fetch expense list |
| Expenses | `/api/expenses` | POST | Create new expense |
| Expenses | `/api/expenses/:id` | DELETE | Delete expense |
| Expenses | `/api/categories` | GET | Fetch categories |

### Backend → Database Flow

```
Frontend Form Input
        ↓
Backend API (/api/expenses)
        ↓
NodeJS Processing
        ↓
MySQL Database Query
        ↓
Return Response
        ↓
Frontend Displays Data
```

---

## Styling & Theme

### Color Scheme
- **Primary Background:** Dark navy/black (#001a33 or similar)
- **Accent Color:** Cyan/turquoise blue (#00BFFF or #00CED1)
- **Text Color:** White (#FFFFFF)
- **Borders:** Cyan/blue for table borders
- **Hover:** Slightly lighter cyan
- **Icons:** Simple minimalist design

### Typography
- **Font:** Clean, sans-serif (Arial, Helvetica, or similar)
- **Title Size:** Large and bold
- **Body Text:** Regular weight
- **Code/Data:** Monospace for table values

### Layout
- **Desktop First:** Responsive design
- **Sidebar Navigation:** Left-aligned menu
- **Content Area:** Main scrollable content
- **Mobile Friendly:** Stacks vertically on small screens

---

## Form Fields

### Login Form
```
┌─────────────────┐
│  Username: [ ]  │
│  Password: [ ]  │
│  [Login Button] │
└─────────────────┘
```

### Add Expense Form
```
┌──────────────────────────┐
│  Amount: [ ]             │
│  Category: [Dropdown]    │
│  Description: [ ]        │
│  Date: [Date Picker]     │
│  [ADD] [CANCEL]          │
└──────────────────────────┘
```

### Expense Table
```
┌─────────────────────────────────┐
│ ID | AMOUNT | CATEGORY | DESC   │
├─────────────────────────────────┤
│ 1  | 50.00  | Food     | Lunch  │
│ 2  | 25.50  | Transport| Uber   │
│ 3  | 100    | Bills    | Rent   │
└─────────────────────────────────┘
```

---

## Responsive Design

### Desktop View (>1200px)
- Sidebar + Content (side-by-side)
- Full table display
- Multiple columns visible

### Tablet View (768px - 1200px)
- Sidebar collapsible
- Table might scroll horizontally
- Touch-friendly buttons

### Mobile View (<768px)
- Full-width layout
- Hamburger menu for navigation
- Stack table vertically
- Single column display

---

## Accessibility Features

- ✅ Clear contrast ratios (WCAG AA)
- ✅ Keyboard navigation support
- ✅ ARIA labels on buttons
- ✅ Focus indicators on inputs
- ✅ Semantic HTML structure
- ✅ Form validation feedback
- ✅ Error messages clearly visible

---

## Performance Considerations

### Frontend Optimization
- Minimize CSS/JavaScript files
- Lazy load images
- Cache static assets
- Compress API responses
- Use CDN for static content

### Backend API Performance
- Pagination for large datasets
- Query optimization
- Database indexing
- Response compression
- Rate limiting

---

## Testing the UI

### Manual Testing Checklist

- [ ] Login page loads without errors
- [ ] Login with valid credentials works
- [ ] Login with invalid credentials shows error
- [ ] Home page displays after login
- [ ] Navigation menu items work
- [ ] Add expense form accepts input
- [ ] Submit adds expense to table
- [ ] Delete button removes expense
- [ ] Table paginates with many items
- [ ] Responsive design works on mobile
- [ ] Console has no JavaScript errors
- [ ] Network tab shows successful API calls
- [ ] All buttons are clickable and responsive
- [ ] Form validation prevents invalid input

---

## Troubleshooting UI Issues

### Blank Page
- Clear browser cache
- Hard refresh (Ctrl+F5 or Cmd+Shift+R)
- Check browser console for errors
- Verify backend is running: `curl http://localhost:8080/`

### Buttons Not Working
- Check browser console for JavaScript errors
- Verify API endpoint is accessible
- Check network requests in DevTools
- Ensure backend is responding

### API Calls Failing
- Verify backend IP in Nginx config
- Check CORS settings
- Verify API endpoints exist
- Check backend logs: `journalctl -u backend -f`

### Styling Issues
- Verify CSS files are loaded (Network tab)
- Check browser DevTools for CSS errors
- Clear browser cache
- Verify Nginx serving static files correctly

---

## Browser Compatibility

### Supported Browsers
- ✅ Chrome/Chromium (latest)
- ✅ Firefox (latest)
- ✅ Safari (latest)
- ✅ Edge (latest)
- ⚠️ IE 11 (limited support)

### Required Features
- JavaScript ES6 support
- Fetch API or XMLHttpRequest
- LocalStorage for session management
- CSS Grid/Flexbox support

---

## File Structure

```
screenshots/
├── expense-app-home.png
├── add-view-expenses.png
└── README.md (this file)
```

---

## Updating Screenshots

To update screenshots:

1. Capture screenshot using browser or tool
2. Save with clear name (e.g., `add-view-expenses.png`)
3. Update this documentation
4. Commit and push to repository

Tools:
- Snipping Tool (Windows)
- Screenshot (macOS)
- gnome-screenshot (Linux)
- Playwright/Puppeteer (automated)

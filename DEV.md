# Complyx Development Guide

## 🔐 Test User Credentials

### Admin User
- **Email:** `admin@complyx.com`
- **Password:** `Admin123!@#`
- **Role:** Admin
- **Access:** Full system access including admin panel
- **Permissions:** All permissions (users, assessments, organizations, content, admin)

### Manager User
- **Email:** `manager@complyx.com`
- **Password:** `Manager123!@#`
- **Role:** Manager
- **Access:** Can manage team and assessments
- **Permissions:** Most permissions except admin panel access

### Regular User
- **Email:** `user@complyx.com`
- **Password:** `User123!@#`
- **Role:** User
- **Access:** Can create and manage own assessments
- **Permissions:** Basic assessment permissions (create, read, update)

### Viewer User
- **Email:** `viewer@complyx.com`
- **Password:** `Viewer123!@#`
- **Role:** Viewer
- **Access:** Read-only access
- **Permissions:** Read-only permissions for all resources

---

## 🚀 Quick Start

### 1. Start Docker Services
```bash
cd server/docker
docker-compose up -d
```

This starts:
- PostgreSQL on port `5432`
- Redis on port `6379`

### 2. Initialize Database
```bash
cd server
pnpm db:generate
pnpm db:migrate
```

### 3. Seed Roles and Users
```bash
cd server
pnpm db:seed:auth
```

This will:
- Initialize default roles (Admin, Manager, User, Viewer)
- Create default permissions
- Create 4 test users (one for each role)

### 4. Start Server
```bash
cd server
pnpm dev
```

Server runs on `http://localhost:3001`

### 5. Start Client
```bash
cd client
pnpm dev
```

Client runs on `http://localhost:3000`

---

## 📍 Important URLs

### Frontend Routes
- **Home/Chat:** `http://localhost:3000/`
- **Login:** `http://localhost:3000/auth/login`
- **Register:** `http://localhost:3000/auth/register`
- **Forgot Password:** `http://localhost:3000/auth/forgot-password`
- **Dashboard:** `http://localhost:3000/dashboard`
- **Admin Panel:** `http://localhost:3000/admin`
- **Admin Users:** `http://localhost:3000/admin/users`
- **Admin Analytics:** `http://localhost:3000/admin/analytics`

### Backend API Endpoints
- **Health Check:** `http://localhost:3001/health`
- **API Base:** `http://localhost:3001/api`
- **Auth Endpoints:**
  - `POST /api/auth/register` - Register new user
  - `POST /api/auth/login` - Login
  - `POST /api/auth/logout` - Logout
  - `GET /api/auth/me` - Get current user
  - `GET /api/auth/verify-email?token=...` - Verify email
  - `POST /api/auth/forgot-password` - Request password reset
  - `POST /api/auth/reset-password` - Reset password
  - `POST /api/auth/refresh-token` - Refresh access token
- **Admin Endpoints:**
  - `GET /api/admin/stats` - Admin dashboard statistics
  - `GET /api/admin/users` - Get all users (paginated)
  - `GET /api/admin/users/:userId` - Get user by ID
  - `PUT /api/admin/users/:userId` - Update user
  - `DELETE /api/admin/users/:userId` - Delete user
  - `GET /api/admin/organizations` - Get all organizations
  - `GET /api/admin/analytics` - Get system analytics
  - `GET /api/admin/audit-logs` - Get audit logs

---

## 🔑 Authentication Flow

### Registration Flow
1. User registers at `/auth/register`
2. System sends verification email (check console in dev mode)
3. User clicks verification link
4. Email is verified, user can now login

### Login Flow
1. User logs in at `/auth/login`
2. System validates credentials
3. System generates JWT tokens (access + refresh)
4. Tokens stored in localStorage
5. User redirected to dashboard

### Admin Access Flow
1. User must have Admin role or `admin:access` permission
2. Access `/admin` routes
3. Admin layout checks permissions
4. If unauthorized, redirects to dashboard

---

## 🛠️ Development Scripts

### Server Scripts
```bash
# Development
pnpm dev                    # Start dev server with hot reload

# Database
pnpm db:generate            # Generate Prisma client
pnpm db:migrate             # Run database migrations
pnpm db:studio              # Open Prisma Studio
pnpm db:seed:auth           # Seed roles and test users

# Testing
pnpm test:auth              # Test authentication flow (requires server running)
```

### Client Scripts
```bash
# Development
pnpm dev                    # Start Next.js dev server
pnpm build                  # Build for production
pnpm start                  # Start production server
```

---

## 🗄️ Database Information

### Connection
- **Host:** `localhost`
- **Port:** `5432`
- **Database:** `ifrsbot`
- **Username:** `postgres`
- **Password:** `postgres`
- **Connection String:** `postgresql://postgres:postgres@localhost:5432/ifrsbot`

### Key Models
- **User** - User accounts with authentication
- **Role** - User roles (Admin, Manager, User, Viewer)
- **Permission** - Granular permissions
- **RolePermission** - Role-permission mapping
- **Organization** - Multi-tenant organizations
- **AuditLog** - System audit trail
- **Session** - User sessions
- **Assessment** - IFRS assessments
- **Question** - Assessment questions
- **Answer** - User answers

---

## 🔒 Security Features

### Rate Limiting
- **General API:** 100 requests per 15 minutes
- **Auth Endpoints:** 5 requests per 15 minutes
- **Admin Endpoints:** 50 requests per 15 minutes
- **Chat Endpoints:** 20 requests per minute

### Security Headers
- Content Security Policy (CSP)
- HTTP Strict Transport Security (HSTS)
- XSS Protection
- Input sanitization

### Password Requirements
- Minimum 8 characters
- At least one uppercase letter
- At least one lowercase letter
- At least one number
- At least one special character

---

## 📧 Email Configuration

### Development Mode
In development, emails are logged to console instead of being sent.

### Production Setup
Configure in `.env`:
```env
EMAIL_PROVIDER=smtp
EMAIL_FROM=noreply@complyx.com
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_SECURE=false
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password
```

---

## 🧪 Testing Authentication

### Manual Testing
1. Start server: `cd server && pnpm dev`
2. Start client: `cd client && pnpm dev`
3. Navigate to `http://localhost:3000/auth/login`
4. Login with any test user credentials above
5. Test admin access by navigating to `/admin`

### Automated Testing
```bash
# Make sure server is running first
cd server
pnpm test:auth
```

---

## 🎯 Role Permissions

### Admin
- Full system access
- All CRUD operations
- Admin panel access
- User management
- System configuration

### Manager
- Team management
- Assessment management
- Organization management
- Content management
- No admin panel access

### User
- Create assessments
- Read own assessments
- Update own assessments
- No delete permissions
- No admin access

### Viewer
- Read-only access
- View assessments
- View reports
- No modification permissions

---

## 🐛 Troubleshooting

### Database Connection Issues
- Ensure Docker containers are running: `docker-compose ps`
- Check DATABASE_URL in `.env`
- Verify PostgreSQL is accessible: `psql -h localhost -U postgres -d ifrsbot`

### Authentication Issues
- Check JWT_SECRET is set in `.env`
- Verify tokens are being stored in localStorage
- Check browser console for errors
- Verify user exists in database

### Admin Access Issues
- Ensure user has Admin role
- Check role-permission mappings in database
- Verify RBAC initialization completed
- Check server logs for permission errors

### Email Issues
- In dev mode, emails are logged to console
- Check email service configuration
- Verify SMTP credentials if using production email

---

## 📝 Notes

- All test users have `emailVerified: true` for easy testing
- Passwords follow the security requirements
- Roles are automatically initialized on server startup
- Admin panel requires Admin role or `admin:access` permission
- JWT tokens expire after 15 minutes (access) and 7 days (refresh)
- Session management tracks user activity

---

## 🔄 Resetting Database

To reset and reseed:
```bash
cd server
# Drop and recreate database (WARNING: Deletes all data)
pnpm db:migrate reset
# Reseed
pnpm db:seed:auth
```

---

*Last Updated: After Phase 5 Week 3 completion*

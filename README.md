<div align="center">

# 🚀 Complyx

### IFRS S1 & S2 Readiness Assessment Platform

**AI-Powered Conversational Assistant for Sustainability Compliance**

[![TypeScript](https://img.shields.io/badge/TypeScript-5.3-blue.svg)](https://www.typescriptlang.org/)
[![Next.js](https://img.shields.io/badge/Next.js-14.1-black.svg)](https://nextjs.org/)
[![Node.js](https://img.shields.io/badge/Node.js-20+-green.svg)](https://nodejs.org/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-blue.svg)](https://www.postgresql.org/)
[![License](https://img.shields.io/badge/License-Private-red.svg)](LICENSE)

</div>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Features](#-features)
- [Architecture](#-architecture)
- [Quick Start](#-quick-start)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [Testing](#-testing)
- [Deployment](#-deployment)
- [Contributing](#-contributing)
- [License](#-license)

---

## 🎯 Overview

**Complyx** is an enterprise-grade platform designed to help organizations assess and achieve compliance with **IFRS S1** (General Requirements for Disclosure of Sustainability-related Financial Information) and **IFRS S2** (Climate-related Disclosures) standards.

### Key Capabilities

- 🤖 **AI-Powered Assessment**: Conversational interface powered by Google Gemini
- 📊 **Real-Time Dashboard**: Comprehensive compliance tracking and gap analysis
- 🔍 **Gap Identification**: Automated identification of compliance gaps
- 📈 **Progress Tracking**: Visual progress indicators and readiness scores
- 📄 **Report Generation**: Exportable PDF and Excel compliance reports
- 🔐 **Enterprise Security**: Role-based access control and authentication

---

## ✨ Features

### Core Features

| Feature | Description | Status |
|---------|-------------|--------|
| **Conversational Assessment** | Interactive chat-based compliance assessment | ✅ Complete |
| **Multi-Phase Assessment** | Quick, detailed, and follow-up assessment phases | ✅ Complete |
| **Real-Time Dashboard** | Live compliance metrics and progress tracking | ✅ Complete |
| **Gap Analysis** | Automated identification and prioritization of gaps | ✅ Complete |
| **Compliance Matrix** | Visual representation of requirement compliance | ✅ Complete |
| **Report Export** | PDF and Excel report generation | ✅ Complete |
| **User Authentication** | JWT-based authentication with role management | ✅ Complete |
| **Admin Panel** | User management and system analytics | ✅ Complete |

### Advanced Features

- 🎨 **Dark Mode Support**: Full dark mode implementation across all components
- 📱 **Responsive Design**: Mobile-first, fully responsive UI
- 🔄 **Real-Time Updates**: Live data synchronization
- 🎯 **Industry Variations**: Industry-specific compliance guidance
- 📚 **Knowledge Base**: RAG-powered document search and retrieval
- 🔔 **Notifications**: Real-time alerts and updates

---

## 🏗️ Architecture

### System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Client (Vercel)                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   Next.js    │  │   React      │  │   Zustand    │     │
│  │   Frontend   │  │   Components │  │   State Mgmt  │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
└─────────────────────────────────────────────────────────────┘
                            │
                            │ HTTPS/REST API
                            │
┌─────────────────────────────────────────────────────────────┐
│                    Server (Railway)                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   Express    │  │   Prisma     │  │   Services   │     │
│  │   API        │  │   ORM        │  │   Layer      │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
└─────────────────────────────────────────────────────────────┘
         │                    │                    │
         │                    │                    │
    ┌─────────┐         ┌─────────┐         ┌─────────┐
    │PostgreSQL│         │  Redis  │         │ Gemini  │
    │Database │         │  Cache  │         │   API   │
    └─────────┘         └─────────┘         └─────────┘
```

### Technology Stack

#### Frontend
- **Framework**: Next.js 14 (App Router)
- **Language**: TypeScript 5.3
- **Styling**: Tailwind CSS 3.4
- **State Management**: Zustand 5.0
- **UI Components**: Custom component library
- **Charts**: Recharts 3.6
- **Animations**: Framer Motion 11.0

#### Backend
- **Runtime**: Node.js 20+
- **Framework**: Express 4.18
- **Language**: TypeScript 5.3
- **ORM**: Prisma 5.9
- **Database**: PostgreSQL 16
- **Cache**: Redis 7
- **AI**: Google Gemini API

#### Infrastructure
- **Client Hosting**: Vercel
- **Server Hosting**: Railway
- **Database**: Railway PostgreSQL
- **Containerization**: Docker
- **CI/CD**: GitHub Actions (optional)

---

## 🚀 Quick Start

### Prerequisites

Ensure you have the following installed:

- **Node.js** >= 18.0.0
- **pnpm** >= 8.0.0
- **Docker** (for local development)
- **Git**

### Installation

#### 1. Clone the Repository

```bash
git clone https://github.com/Atlas00000/complyx_main.git
cd complyx_main
```

#### 2. Install Dependencies

```bash
# Install root dependencies
pnpm install

# Install client dependencies
cd client && pnpm install && cd ..

# Install server dependencies
cd server && pnpm install && cd ..
```

#### 3. Environment Setup

```bash
# Copy environment files
cp server/.env.example server/.env
cp client/.env.example client/.env.local
```

**Required Environment Variables:**

**Server (`server/.env`):**
```env
# Database
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/ifrsbot

# Redis
REDIS_URL=redis://localhost:6379

# AI Provider
AI_PROVIDER=gemini
GEMINI_API_KEY=your_gemini_api_key_here

# JWT Secrets (generate with: openssl rand -base64 32)
JWT_SECRET=your_jwt_secret_here
JWT_REFRESH_SECRET=your_refresh_secret_here

# Application URLs
CLIENT_URL=http://localhost:3000
APP_URL=http://localhost:3001
```

**Client (`client/.env.local`):**
```env
NEXT_PUBLIC_API_URL=http://localhost:3001
```

#### 4. Start Docker Services

```bash
cd server/docker
docker-compose up -d
```

#### 5. Database Setup

```bash
cd server

# Generate Prisma Client
pnpm db:generate

# Run migrations
pnpm db:migrate

# Seed initial data (optional)
pnpm db:seed:auth
```

#### 6. Start Development Servers

**Terminal 1 - Client:**
```bash
cd client
pnpm dev
```

**Terminal 2 - Server:**
```bash
cd server
pnpm dev
```

### Access the Application

- **Client**: http://localhost:3000
- **Server API**: http://localhost:3001
- **API Health Check**: http://localhost:3001/health
- **Prisma Studio**: `cd server && pnpm db:studio`

---

## 📁 Project Structure

```
ifrsbot/
├── 📂 client/                    # Next.js Frontend Application
│   ├── 📂 app/                   # Next.js App Router
│   │   ├── 📂 dashboard/         # Dashboard pages
│   │   ├── 📂 auth/              # Authentication pages
│   │   └── 📂 admin/             # Admin panel pages
│   ├── 📂 components/            # React Components
│   │   ├── 📂 chat/              # Chat interface components
│   │   ├── 📂 dashboard/         # Dashboard components
│   │   ├── 📂 assessment/        # Assessment components
│   │   └── 📂 ui/                # UI component library
│   ├── 📂 lib/                   # Utilities and helpers
│   ├── 📂 stores/                # Zustand state management
│   ├── 📂 hooks/                 # Custom React hooks
│   └── 📄 package.json
│
├── 📂 server/                    # Express Backend Application
│   ├── 📂 src/
│   │   ├── 📂 routes/             # API routes
│   │   ├── 📂 controllers/        # Route controllers
│   │   ├── 📂 services/           # Business logic
│   │   │   ├── 📂 ai/             # AI service integrations
│   │   │   ├── 📂 assessment/     # Assessment services
│   │   │   ├── 📂 auth/           # Authentication services
│   │   │   └── 📂 knowledge/      # RAG and knowledge services
│   │   ├── 📂 middleware/         # Express middleware
│   │   └── 📂 utils/              # Utility functions
│   ├── 📂 prisma/                # Database schema and migrations
│   ├── 📂 docker/                # Docker configuration
│   └── 📄 package.json
│
├── 📄 README.md                  # This file
├── 📄 DEPLOYMENT.md              # Deployment guide
└── 📄 package.json              # Root package.json
```

---

## 🧪 Testing

### Testing Philosophy

We follow industry best practices for testing:

- **Unit Tests**: Test individual functions and components in isolation
- **Integration Tests**: Test interactions between components and services
- **E2E Tests**: Test complete user workflows
- **Performance Tests**: Ensure optimal performance under load
- **Security Tests**: Validate security measures and authentication

### Running Tests

#### Client Tests

```bash
cd client

# Run all tests
pnpm test

# Run tests in watch mode
pnpm test:watch

# Run tests with coverage
pnpm test:coverage

# Run E2E tests
pnpm test:e2e
```

#### Server Tests

```bash
cd server

# Run all tests
pnpm test

# Run integration tests
pnpm test:integration

# Run specific test suite
pnpm test:auth
pnpm test:rag
pnpm test:dashboard
```

### Test Coverage Goals

| Component | Target Coverage | Current Status |
|-----------|----------------|----------------|
| **Client Components** | > 80% | 🟡 In Progress |
| **Server Services** | > 85% | 🟡 In Progress |
| **API Routes** | > 90% | 🟡 In Progress |
| **Utilities** | > 95% | 🟡 In Progress |

### Testing Best Practices

1. **Test Isolation**: Each test should be independent and not rely on other tests
2. **Arrange-Act-Assert**: Follow the AAA pattern for test structure
3. **Mock External Dependencies**: Mock API calls, database queries, and external services
4. **Test Edge Cases**: Include tests for error conditions and boundary cases
5. **Meaningful Test Names**: Use descriptive test names that explain what is being tested
6. **Fast Execution**: Keep tests fast to enable quick feedback loops
7. **Continuous Integration**: All tests must pass before merging to main

### Example Test Structure

```typescript
describe('DashboardService', () => {
  describe('getDashboardData', () => {
    it('should return dashboard data for authenticated user', async () => {
      // Arrange
      const userId = 'user-123';
      const mockData = { /* ... */ };
      
      // Act
      const result = await dashboardService.getDashboardData(userId);
      
      // Assert
      expect(result).toEqual(mockData);
    });
    
    it('should throw error for unauthenticated user', async () => {
      // Arrange
      const userId = null;
      
      // Act & Assert
      await expect(
        dashboardService.getDashboardData(userId)
      ).rejects.toThrow('Unauthorized');
    });
  });
});
```

---

## 🚢 Deployment

### Production Deployment

See [DEPLOYMENT.md](./DEPLOYMENT.md) for comprehensive deployment instructions.

**Quick Deploy:**

1. **Server (Railway)**
   - Connect GitHub repository
   - Add PostgreSQL and Redis services
   - Configure environment variables
   - Deploy automatically on push

2. **Client (Vercel)**
   - Import GitHub repository
   - Set root directory to `client`
   - Configure environment variables
   - Deploy automatically on push

### Environment-Specific Configuration

- **Development**: Local Docker services
- **Staging**: Railway preview deployments
- **Production**: Railway + Vercel production

---

## 🤝 Contributing

### Development Workflow

1. **Create Feature Branch**
   ```bash
   git checkout -b feature/your-feature-name
   ```

2. **Make Changes**
   - Follow code style guidelines
   - Write tests for new features
   - Update documentation

3. **Test Your Changes**
   ```bash
   pnpm build:client
   pnpm build:server
   pnpm test
   ```

4. **Commit Changes**
   ```bash
   git commit -m "feat: add new feature"
   ```

5. **Push and Create PR**
   ```bash
   git push origin feature/your-feature-name
   ```

### Code Style

- **TypeScript**: Strict mode enabled
- **ESLint**: Configured for Next.js and React
- **Prettier**: Code formatting (if configured)
- **Conventional Commits**: Follow commit message conventions

---

## 📊 Project Status

### Completed Features ✅

- [x] Project setup and architecture
- [x] Authentication system
- [x] Chat interface
- [x] Assessment system
- [x] Dashboard with real-time metrics
- [x] Gap analysis
- [x] Compliance matrix
- [x] Report generation
- [x] Admin panel
- [x] Dark mode support
- [x] Docker containerization
- [x] Deployment configuration

### In Progress 🟡

- [ ] Comprehensive test coverage
- [ ] Performance optimization
- [ ] Advanced analytics
- [ ] Multi-language support

### Planned 🔵

- [ ] Mobile applications
- [ ] Advanced AI features
- [ ] Integration with external systems
- [ ] Enhanced reporting capabilities

---

## 📚 Documentation

- **[Deployment Guide](./DEPLOYMENT.md)**: Complete deployment instructions
- **[Development Guide](./DEV.md)**: Development setup and workflows
- **[Roadmap](./roadmap.md)**: Project roadmap and milestones

---

## 🔒 Security

### Security Features

- ✅ JWT-based authentication
- ✅ Password hashing with bcrypt
- ✅ Role-based access control (RBAC)
- ✅ Rate limiting
- ✅ CORS configuration
- ✅ Input validation and sanitization
- ✅ SQL injection prevention (Prisma)
- ✅ XSS protection
- ✅ HTTPS enforcement

### Security Best Practices

- Never commit secrets to version control
- Use environment variables for sensitive data
- Regularly update dependencies
- Monitor for security vulnerabilities
- Implement proper error handling
- Use secure session management

---

## 📈 Performance

### Performance Metrics

- **Client Load Time**: < 2s (First Contentful Paint)
- **API Response Time**: < 200ms (p95)
- **Database Query Time**: < 50ms (p95)
- **Page Load Time**: < 3s (Lighthouse Score: 90+)

### Optimization Strategies

- Code splitting and lazy loading
- Image optimization
- Database query optimization
- Caching strategies (Redis)
- CDN for static assets
- Bundle size optimization

---

## 📞 Support

For issues, questions, or contributions:

- **GitHub Issues**: [Create an issue](https://github.com/Atlas00000/complyx_main/issues)
- **Documentation**: See [DEPLOYMENT.md](./DEPLOYMENT.md) and [DEV.md](./DEV.md)

---

## 📄 License

**Private - All Rights Reserved**

This project is proprietary and confidential. Unauthorized copying, modification, distribution, or use of this software is strictly prohibited.

---

<div align="center">

**Built with ❤️ by the Complyx Team**

[⬆ Back to Top](#-complyx)

</div>

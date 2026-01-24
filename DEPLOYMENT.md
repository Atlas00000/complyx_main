# Deployment Guide

This guide covers deploying Complyx to production:
- **Server & Database**: Railway
- **Client**: Vercel

---

## Prerequisites

- GitHub repository with your code
- Railway account (https://railway.app)
- Vercel account (https://vercel.com)
- Google Gemini API key (https://makersuite.google.com/app/apikey)
- (Optional) Cloudflare R2 account for file storage

---

## Part 1: Deploy Server & Database to Railway

### Step 1: Create Railway Project

1. Go to [Railway Dashboard](https://railway.app/dashboard)
2. Click **"New Project"**
3. Select **"Deploy from GitHub repo"**
4. Choose your repository
5. Railway will detect the project structure

### Step 2: Add PostgreSQL Database

1. In your Railway project, click **"+ New"**
2. Select **"Database"** → **"Add PostgreSQL"**
3. Railway will automatically create a PostgreSQL database
4. Note the connection details (you'll need the `DATABASE_URL`)

### Step 3: Add Redis (Optional but Recommended)

1. In your Railway project, click **"+ New"**
2. Select **"Database"** → **"Add Redis"**
3. Note the connection details (you'll need the `REDIS_URL`)

### Step 4: Deploy Server

1. In your Railway project, click **"+ New"**
2. Select **"GitHub Repo"** → Choose your repository
3. Railway will detect the `server/` directory
4. Configure the service:
   - **Root Directory**: `server`
   - **Build Command**: `pnpm install && pnpm build`
   - **Start Command**: `pnpm start`
   - **Watch Paths**: `server/**`

### Step 5: Configure Environment Variables

In your Railway server service, go to **Variables** tab and add:

```env
# Database (provided by Railway PostgreSQL service)
DATABASE_URL=${{Postgres.DATABASE_URL}}

# Redis (provided by Railway Redis service, if added)
REDIS_URL=${{Redis.REDIS_URL}}

# AI Provider
AI_PROVIDER=gemini
GEMINI_API_KEY=your_gemini_api_key_here

# JWT Secrets (generate strong random strings)
JWT_SECRET=your_very_secure_random_string_here
JWT_REFRESH_SECRET=your_very_secure_random_string_here
JWT_ACCESS_EXPIRY=15m
JWT_REFRESH_EXPIRY=7d

# Application URLs (update after deploying client)
APP_URL=https://your-server-name.railway.app
CLIENT_URL=https://your-client-name.vercel.app

# Node Environment
NODE_ENV=production
PORT=3001

# Vector Database (optional - for production RAG)
VECTOR_DB_TYPE=memory
# Or use Pinecone:
# VECTOR_DB_TYPE=pinecone
# PINECONE_API_KEY=your_pinecone_key
# PINECONE_ENVIRONMENT=your_environment
# PINECONE_INDEX_NAME=complyx-knowledge
# EMBEDDING_DIMENSION=768

# Cloudflare R2 (optional - for file storage)
CLOUDFLARE_R2_ACCOUNT_ID=
CLOUDFLARE_R2_ACCESS_KEY_ID=
CLOUDFLARE_R2_SECRET_ACCESS_KEY=

# Email Configuration (optional)
EMAIL_PROVIDER=smtp
EMAIL_FROM=noreply@complyx.com
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_SECURE=false
SMTP_USER=your_email@gmail.com
SMTP_PASS=your_app_password

# OAuth (optional)
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
MICROSOFT_CLIENT_ID=
MICROSOFT_CLIENT_SECRET=
```

**Important Notes:**
- `DATABASE_URL` and `REDIS_URL` can use Railway's variable references: `${{Postgres.DATABASE_URL}}`
- Generate secure JWT secrets: `openssl rand -base64 32`
- Update `CLIENT_URL` after deploying to Vercel

### Step 6: Run Database Migrations

1. In Railway, go to your server service
2. Click on **"Deployments"** → **"View Logs"**
3. Or use Railway CLI:
   ```bash
   railway run pnpm db:generate
   railway run pnpm db:migrate
   ```

Alternatively, add a startup script to run migrations automatically:

1. Create `server/railway.json`:
```json
{
  "$schema": "https://railway.app/railway.schema.json",
  "build": {
    "builder": "NIXPACKS",
    "buildCommand": "pnpm install && pnpm build"
  },
  "deploy": {
    "startCommand": "pnpm db:generate && pnpm db:migrate && pnpm start",
    "restartPolicyType": "ON_FAILURE",
    "restartPolicyMaxRetries": 10
  }
}
```

### Step 7: Seed Initial Data (Optional)

If you need to seed roles and initial users:

```bash
railway run pnpm db:seed:auth
```

### Step 8: Get Server URL

1. Railway will assign a public URL to your server
2. Go to **Settings** → **Domains** to see your URL
3. Example: `https://complyx-server-production.up.railway.app`
4. Copy this URL - you'll need it for the client configuration

---

## Part 2: Deploy Client to Vercel

### Step 1: Import Project to Vercel

1. Go to [Vercel Dashboard](https://vercel.com/dashboard)
2. Click **"Add New..."** → **"Project"**
3. Import your GitHub repository
4. Vercel will auto-detect Next.js

### Step 2: Configure Project Settings

In the project configuration:

- **Framework Preset**: Next.js
- **Root Directory**: `client`
- **Build Command**: `pnpm build` (or leave default)
- **Output Directory**: `.next` (default)
- **Install Command**: `pnpm install`

### Step 3: Configure Environment Variables

In Vercel project settings → **Environment Variables**, add:

```env
# API URL - Your Railway server URL
NEXT_PUBLIC_API_URL=https://your-server-name.railway.app

# Cloudflare R2 Public URL (optional)
NEXT_PUBLIC_CLOUDFLARE_R2_PUBLIC_URL=https://your-r2-bucket.r2.dev

# Node Environment
NODE_ENV=production
```

**Important:**
- `NEXT_PUBLIC_API_URL` must match your Railway server URL
- Add these to **Production**, **Preview**, and **Development** environments
- Variables prefixed with `NEXT_PUBLIC_` are exposed to the browser

### Step 4: Deploy

1. Click **"Deploy"**
2. Vercel will:
   - Install dependencies
   - Build the Next.js application
   - Deploy to production
3. Wait for deployment to complete (usually 2-3 minutes)

### Step 5: Get Client URL

1. After deployment, Vercel provides a URL
2. Example: `https://complyx-client.vercel.app`
3. Copy this URL

### Step 6: Update Server Configuration

Go back to Railway and update the server's `CLIENT_URL` environment variable:

```env
CLIENT_URL=https://your-client-name.vercel.app
```

Redeploy the server service to apply the change.

---

## Part 3: Post-Deployment Setup

### 1. Update CORS Configuration

Ensure your Railway server allows requests from your Vercel domain. Check `server/src/server.ts` or middleware for CORS settings:

```typescript
cors({
  origin: process.env.CLIENT_URL || 'http://localhost:3000',
  credentials: true,
})
```

### 2. Test the Deployment

1. **Test Server Health:**
   ```bash
   curl https://your-server-name.railway.app/health
   ```
   Should return: `{"status":"ok","message":"Complyx API Server is running"}`

2. **Test Client:**
   - Visit your Vercel URL
   - Try logging in
   - Test API connectivity

### 3. Set Up Custom Domains (Optional)

**Railway:**
1. Go to server service → **Settings** → **Domains**
2. Add your custom domain
3. Configure DNS records as instructed

**Vercel:**
1. Go to project → **Settings** → **Domains**
2. Add your custom domain
3. Configure DNS records as instructed

---

## Part 4: Database Management

### Running Migrations

**Via Railway CLI:**
```bash
railway login
railway link
railway run pnpm db:migrate
```

**Via Railway Dashboard:**
1. Go to server service → **Deployments**
2. Create a new deployment with custom command: `pnpm db:migrate`

### Accessing Database

**Via Prisma Studio:**
```bash
railway run pnpm db:studio
```

**Via Railway Dashboard:**
1. Go to PostgreSQL service
2. Click **"Query"** tab
3. Run SQL queries directly

---

## Part 5: Monitoring & Logs

### Railway Logs

1. Go to your service → **Deployments**
2. Click on a deployment → **View Logs**
3. Monitor real-time logs

### Vercel Logs

1. Go to project → **Deployments**
2. Click on a deployment → **View Function Logs**
3. Monitor build and runtime logs

### Health Checks

Set up monitoring for:
- Server health endpoint: `/health`
- Database connectivity
- API response times

---

## Troubleshooting

### Server Issues

**Build Fails:**
- Check Railway logs for error messages
- Ensure `pnpm` is available (Railway auto-detects)
- Verify all dependencies in `package.json`

**Database Connection Errors:**
- Verify `DATABASE_URL` is set correctly
- Check PostgreSQL service is running
- Ensure migrations have run

**401 Unauthorized Errors:**
- Verify `JWT_SECRET` and `JWT_REFRESH_SECRET` are set
- Check token expiration settings
- Ensure CORS allows your client domain

### Client Issues

**Build Fails:**
- Check Vercel build logs
- Verify all environment variables are set
- Check for TypeScript errors: `pnpm build` locally

**API Connection Errors:**
- Verify `NEXT_PUBLIC_API_URL` matches Railway server URL
- Check CORS configuration on server
- Verify server is running and accessible

**Environment Variables Not Working:**
- Ensure variables are prefixed with `NEXT_PUBLIC_` for client-side access
- Redeploy after adding new variables
- Check variable names match exactly

---

## Environment Variables Reference

### Server (Railway)

| Variable | Required | Description |
|----------|----------|-------------|
| `DATABASE_URL` | Yes | PostgreSQL connection string |
| `REDIS_URL` | No | Redis connection string |
| `GEMINI_API_KEY` | Yes | Google Gemini API key |
| `JWT_SECRET` | Yes | JWT signing secret |
| `JWT_REFRESH_SECRET` | Yes | JWT refresh token secret |
| `CLIENT_URL` | Yes | Vercel client URL |
| `NODE_ENV` | Yes | Set to `production` |
| `PORT` | No | Server port (default: 3001) |

### Client (Vercel)

| Variable | Required | Description |
|----------|----------|-------------|
| `NEXT_PUBLIC_API_URL` | Yes | Railway server URL |
| `NEXT_PUBLIC_CLOUDFLARE_R2_PUBLIC_URL` | No | R2 public bucket URL |
| `NODE_ENV` | Yes | Set to `production` |

---

## Quick Deploy Checklist

### Railway (Server & DB)
- [ ] Create Railway project
- [ ] Add PostgreSQL database
- [ ] Add Redis (optional)
- [ ] Deploy server service
- [ ] Set all environment variables
- [ ] Run database migrations
- [ ] Seed initial data (optional)
- [ ] Test server health endpoint
- [ ] Note server URL

### Vercel (Client)
- [ ] Import GitHub repository
- [ ] Set root directory to `client`
- [ ] Set all environment variables
- [ ] Deploy project
- [ ] Test client URL
- [ ] Update server `CLIENT_URL`
- [ ] Test full integration

### Post-Deployment
- [ ] Test authentication flow
- [ ] Test API endpoints
- [ ] Verify CORS configuration
- [ ] Set up custom domains (optional)
- [ ] Configure monitoring
- [ ] Document production URLs

---

## Support & Resources

- **Railway Docs**: https://docs.railway.app
- **Vercel Docs**: https://vercel.com/docs
- **Next.js Deployment**: https://nextjs.org/docs/deployment
- **Prisma Deployment**: https://www.prisma.io/docs/guides/deployment

---

## Security Best Practices

1. **Never commit secrets** - Use environment variables
2. **Use strong JWT secrets** - Generate with `openssl rand -base64 32`
3. **Enable HTTPS** - Railway and Vercel provide this automatically
4. **Set up rate limiting** - Already configured in server
5. **Regular updates** - Keep dependencies updated
6. **Monitor logs** - Check for suspicious activity
7. **Database backups** - Railway provides automatic backups

---

**Last Updated**: January 2025

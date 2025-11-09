# Complete Railway Deployment Guide

This guide provides step-by-step instructions for deploying the MercurJS Railway Boilerplate to Railway.app with a multi-service architecture.

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Architecture](#architecture)
- [Step 1: Prepare Your Repository](#step-1-prepare-your-repository)
- [Step 2: Create Railway Project](#step-2-create-railway-project)
- [Step 3: Add PostgreSQL Database](#step-3-add-postgresql-database)
- [Step 4: Deploy Backend API](#step-4-deploy-backend-api)
- [Step 5: Deploy Vendor Panel](#step-5-deploy-vendor-panel)
- [Step 6: Deploy Storefront](#step-6-deploy-storefront)
- [Step 7: Configure Custom Domains](#step-7-configure-custom-domains-optional)
- [Step 8: Run Database Migrations](#step-8-run-database-migrations)
- [Troubleshooting](#troubleshooting)
- [Cost Estimation](#cost-estimation)

## Overview

This boilerplate deploys as **3 separate services** on Railway:

1. **Backend API** - MedusaJS API server (Port 9000)
2. **Vendor Panel** - Admin dashboard (Port 7000)
3. **Storefront** - Customer-facing Next.js app (Port 8000)

Each service is independently scalable and deployable, sharing a single PostgreSQL database.

## Prerequisites

Before deploying to Railway, ensure you have:

- ✅ **Railway Account** - Sign up at [railway.app](https://railway.app/)
- ✅ **GitHub Account** - Repository must be pushed to GitHub
- ✅ **Forked Repository** - Fork this repository to your GitHub account
- ✅ **Third-party Services** (if using):
  - Stripe account for payments
  - Resend account for emails
  - Algolia account for search
  - AWS S3 or Cloudflare R2 for file storage (optional)

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Railway Project                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   Backend    │  │    Vendor    │  │  Storefront  │     │
│  │     API      │  │    Panel     │  │              │     │
│  │  (Port 9000) │  │  (Port 7000) │  │  (Port 8000) │     │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘     │
│         │                 │                 │              │
│         └─────────────────┴─────────────────┘              │
│                           │                                │
│                  ┌────────▼────────┐                       │
│                  │   PostgreSQL    │                       │
│                  │    Database     │                       │
│                  └─────────────────┘                       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Step 1: Prepare Your Repository

### 1.1 Fork the Repository

1. Go to the [MercurJS Railway Boilerplate](https://github.com/dtoyoda10/mercur-railway-boilerplate)
2. Click **Fork** in the top right
3. Clone your fork locally:

```bash
git clone https://github.com/YOUR_USERNAME/mercur-railway-boilerplate.git
cd mercur-railway-boilerplate
```

### 1.2 Verify Local Setup (Optional but Recommended)

Test locally before deploying:

```bash
# Install dependencies
pnpm install

# Set up local .env files (see docs/environment-variables.md)

# Run database migrations
pnpm run db:init

# Test each service
pnpm run dev
```

## Step 2: Create Railway Project

### 2.1 Create New Project

1. Log in to [Railway Dashboard](https://railway.app/dashboard)
2. Click **New Project**
3. Select **Deploy from GitHub repo**
4. Authorize Railway to access your GitHub account (if first time)
5. Select your forked `mercur-railway-boilerplate` repository
6. Railway will create an empty project

### 2.2 Remove Default Service

Railway may auto-create a service. We'll create custom services instead:

1. If a service was auto-created, click on it
2. Click **Settings** → **Danger** → **Remove Service**

## Step 3: Add PostgreSQL Database

### 3.1 Add PostgreSQL Plugin

1. In your Railway project, click **+ New**
2. Select **Database** → **Add PostgreSQL**
3. Railway will provision a PostgreSQL instance
4. Note: The `DATABASE_URL` environment variable is automatically created

### 3.2 Verify Database Connection

1. Click on the PostgreSQL service
2. Go to **Variables** tab
3. You should see `DATABASE_URL` with a connection string like:
   ```
   postgresql://postgres:password@host:port/railway
   ```

## Step 4: Deploy Backend API

### 4.1 Create Backend Service

1. Click **+ New** → **Empty Service**
2. Name it `backend-api`
3. Click on the service to configure it

### 4.2 Connect to GitHub

1. Go to **Settings** tab
2. Under **Source**, click **Connect Repo**
3. Select your `mercur-railway-boilerplate` repository
4. Configure the following:

**Root Directory:**
```
apps/backend
```

**Watch Paths:**
```
apps/backend/**
packages/**
pnpm-workspace.yaml
pnpm-lock.yaml
```

### 4.3 Configure Build & Start Commands

Still in **Settings** tab:

**Build Command:**
```bash
cd ../.. && pnpm install && pnpm run build:api
```

**Start Command:**
```bash
cd ../.. && pnpm run start:api
```

### 4.4 Set Environment Variables

Go to **Variables** tab and add the following:

**Required Variables:**

| Variable | Value | Description |
|----------|-------|-------------|
| `DATABASE_URL` | `${{Postgres.DATABASE_URL}}` | Reference to PostgreSQL (auto-filled) |
| `NODE_ENV` | `production` | Environment mode |
| `JWT_SECRET` | `[Generate strong random string]` | JWT signing secret (use password generator) |
| `COOKIE_SECRET` | `[Generate strong random string]` | Cookie signing secret (use password generator) |
| `PORT` | `9000` | Backend API port |

**CORS Configuration:**

| Variable | Value | Description |
|----------|-------|-------------|
| `MEDUSA_ADMIN_CORS` | `https://your-vendor-panel.railway.app` | Vendor panel URL (update after deploying) |
| `MEDUSA_STORE_CORS` | `https://your-storefront.railway.app` | Storefront URL (update after deploying) |

**Optional Services (if configured):**

| Variable | Value | Description |
|----------|-------|-------------|
| `REDIS_URL` | `redis://host:port` | Redis connection (if using caching) |
| `MINIO_ENDPOINT` | `your-bucket.r2.cloudflarestorage.com` | S3/R2 endpoint for file storage |
| `MINIO_ACCESS_KEY` | `your-access-key` | S3/R2 access key |
| `MINIO_SECRET_KEY` | `your-secret-key` | S3/R2 secret key |
| `MINIO_BUCKET` | `your-bucket-name` | S3/R2 bucket name |
| `STRIPE_API_KEY` | `sk_live_...` | Stripe secret key |
| `RESEND_API_KEY` | `re_...` | Resend API key for emails |
| `ALGOLIA_APP_ID` | `...` | Algolia application ID |
| `ALGOLIA_ADMIN_KEY` | `...` | Algolia admin API key |

**Generate Secrets:**
```bash
# Use this command to generate secure random strings
openssl rand -base64 32
```

### 4.5 Deploy Backend

1. Click **Deploy** or wait for auto-deploy
2. Monitor the build logs in the **Deployments** tab
3. Wait for the deployment to complete (5-10 minutes for first deploy)
4. Once successful, you'll see a public URL like: `https://backend-api-production.up.railway.app`

### 4.6 Generate Domain

1. Go to **Settings** → **Networking**
2. Click **Generate Domain**
3. Copy the generated URL (e.g., `https://backend-api-production.up.railway.app`)

## Step 5: Deploy Vendor Panel

### 5.1 Create Vendor Service

1. Click **+ New** → **Empty Service**
2. Name it `vendor-panel`
3. Click on the service to configure it

### 5.2 Connect to GitHub

1. Go to **Settings** tab
2. Under **Source**, click **Connect Repo**
3. Select your `mercur-railway-boilerplate` repository
4. Configure:

**Root Directory:**
```
apps/vendor-panel
```

**Watch Paths:**
```
apps/vendor-panel/**
packages/**
pnpm-workspace.yaml
pnpm-lock.yaml
```

### 5.3 Configure Build & Start Commands

**Build Command:**
```bash
cd ../.. && pnpm install && pnpm run build:vendor
```

**Start Command:**
```bash
cd ../.. && pnpm run start:vendor
```

### 5.4 Set Environment Variables

Go to **Variables** tab and add:

| Variable | Value | Description |
|----------|-------|-------------|
| `MEDUSA_BACKEND_URL` | `https://your-backend-api.railway.app` | Backend API URL from Step 4.6 |
| `PORT` | `7000` | Vendor panel port |

### 5.5 Deploy Vendor Panel

1. Click **Deploy**
2. Monitor build logs
3. Once deployed, go to **Settings** → **Networking** → **Generate Domain**
4. Copy the vendor panel URL (e.g., `https://vendor-panel-production.up.railway.app`)

### 5.6 Update Backend CORS

Now that you have the vendor panel URL:

1. Go back to **backend-api** service
2. Go to **Variables** tab
3. Update `MEDUSA_ADMIN_CORS` to include your vendor panel URL:
   ```
   https://vendor-panel-production.up.railway.app
   ```
4. Redeploy backend for changes to take effect

## Step 6: Deploy Storefront

### 6.1 Create Storefront Service

1. Click **+ New** → **Empty Service**
2. Name it `storefront`
3. Click on the service to configure it

### 6.2 Connect to GitHub

1. Go to **Settings** tab
2. Under **Source**, click **Connect Repo**
3. Select your `mercur-railway-boilerplate` repository
4. Configure:

**Root Directory:**
```
apps/storefront
```

**Watch Paths:**
```
apps/storefront/**
packages/**
pnpm-workspace.yaml
pnpm-lock.yaml
```

### 6.3 Configure Build & Start Commands

**Build Command:**
```bash
cd ../.. && pnpm install && pnpm run build:store
```

**Start Command:**
```bash
cd ../.. && pnpm run start:store
```

### 6.4 Set Environment Variables

Go to **Variables** tab and add:

| Variable | Value | Description |
|----------|-------|-------------|
| `NEXT_PUBLIC_MEDUSA_BACKEND_URL` | `https://your-backend-api.railway.app` | Backend API URL |
| `NEXT_PUBLIC_STORE_URL` | `https://your-storefront.railway.app` | Storefront URL (update after generating) |
| `PORT` | `8000` | Storefront port |

### 6.5 Deploy Storefront

1. Click **Deploy**
2. Monitor build logs (Next.js builds may take 5-10 minutes)
3. Once deployed, go to **Settings** → **Networking** → **Generate Domain**
4. Copy the storefront URL

### 6.6 Update Backend CORS

1. Go back to **backend-api** service
2. Go to **Variables** tab
3. Update `MEDUSA_STORE_CORS` to include your storefront URL:
   ```
   https://storefront-production.up.railway.app
   ```
4. Redeploy backend

## Step 7: Configure Custom Domains (Optional)

### 7.1 Add Custom Domain to Each Service

For each service (backend, vendor, storefront):

1. Go to service **Settings** → **Networking**
2. Under **Custom Domains**, click **+ Custom Domain**
3. Enter your domain (e.g., `api.yourdomain.com`)
4. Railway provides DNS instructions (CNAME record)
5. Add the CNAME record to your DNS provider
6. Wait for DNS propagation (5-30 minutes)

**Recommended subdomains:**
- Backend: `api.yourdomain.com`
- Vendor: `admin.yourdomain.com`
- Storefront: `shop.yourdomain.com` or `www.yourdomain.com`

### 7.2 Update Environment Variables

After adding custom domains, update CORS variables in backend:

```
MEDUSA_ADMIN_CORS=https://admin.yourdomain.com
MEDUSA_STORE_CORS=https://shop.yourdomain.com
```

## Step 8: Run Database Migrations

### 8.1 Access Backend Shell

1. Go to **backend-api** service
2. Click **Deployments** tab
3. Click on the latest deployment
4. Click **View Logs** → **Shell** (or use Railway CLI)

### 8.2 Run Migrations

```bash
cd ../.. && pnpm run db:migrate
```

### 8.3 Create Admin User

```bash
cd apps/backend && npx medusa user -e admin@yourdomain.com -p your-secure-password
```

### 8.4 Seed Database (Optional)

If you want sample data:

```bash
cd apps/backend && npm run seed
```

This creates:
- Sample seller account: `seller@mercurjs.com` / `secret`
- Sample products and categories
- Default regions and sales channels

## Troubleshooting

### Build Fails with "Cannot find module"

**Solution:** Ensure root directory is set correctly and pnpm is installing dependencies.

Check build command includes:
```bash
cd ../.. && pnpm install && pnpm run build:api
```

### Backend Deployment Fails - Database Connection Error

**Symptoms:**
```
Error: connect ECONNREFUSED
Cannot connect to database
```

**Solution:**
1. Verify `DATABASE_URL` is set correctly in backend variables
2. Check PostgreSQL service is running
3. Ensure database reference is correct: `${{Postgres.DATABASE_URL}}`

### CORS Errors in Browser Console

**Symptoms:**
```
Access to fetch at 'https://backend.railway.app' from origin 'https://storefront.railway.app'
has been blocked by CORS policy
```

**Solution:**
1. Go to backend-api **Variables**
2. Update `MEDUSA_ADMIN_CORS` and `MEDUSA_STORE_CORS` with correct URLs
3. Ensure URLs have `https://` prefix
4. Redeploy backend service

### "Module not found" in Vendor Panel

**Solution:** Vendor panel needs to reference the backend URL correctly.

Verify `MEDUSA_BACKEND_URL` is set to your backend domain (without trailing slash):
```
https://backend-api-production.up.railway.app
```

### Next.js Build Timeout

**Symptoms:** Storefront deployment times out during build.

**Solution:**
1. Railway free tier has build time limits
2. Upgrade to Pro plan for longer build times
3. Optimize Next.js config for faster builds

### Database Migrations Don't Run Automatically

Railway doesn't auto-run migrations. You must run them manually via shell or add a release command.

**Option 1 - Manual (Recommended for first deploy):**
Use Railway shell (Step 8)

**Option 2 - Automated (Add to backend):**
Create `railway.json` in `apps/backend`:

```json
{
  "build": {
    "builder": "NIXPACKS"
  },
  "deploy": {
    "releaseCommand": "cd ../.. && pnpm run db:migrate"
  }
}
```

## Cost Estimation

### Railway Pricing (as of 2024)

**Free Tier (Trial):**
- $5 credit (one-time)
- Good for testing
- ~1-2 weeks of uptime for 3 services

**Hobby Plan:**
- $5/month base fee
- $0.000231 per GB-hour for memory
- $0.000463 per vCPU-hour
- Estimated: **$20-40/month** for this stack

**Pro Plan:**
- $20/month base fee
- Same per-resource pricing
- Better for production

**Typical Usage for This Boilerplate:**
- Backend: ~512MB RAM, 0.5 vCPU
- Vendor: ~256MB RAM, 0.25 vCPU
- Storefront: ~512MB RAM, 0.5 vCPU
- PostgreSQL: ~256MB RAM

**Monthly Estimate:** $25-45 depending on traffic

## Next Steps

After successful deployment:

1. ✅ **Test API**: Visit `https://your-backend.railway.app/health`
2. ✅ **Access Admin**: Visit `https://your-vendor.railway.app`
3. ✅ **Access Storefront**: Visit `https://your-storefront.railway.app`
4. 📝 **Configure Stripe**: See [docs/guides/stripe-setup.md](../guides/stripe-setup.md)
5. 📝 **Configure Email**: See [docs/guides/resend-setup.md](../guides/resend-setup.md)
6. 📝 **Set up Search**: See [docs/guides/algolia-setup.md](../guides/algolia-setup.md)
7. 🔐 **Secure Your App**: See [docs/security.md](../security.md)

## Additional Resources

- [Railway Documentation](https://docs.railway.app/)
- [MedusaJS Documentation](https://docs.medusajs.com/)
- [Environment Variables Reference](../environment-variables.md)
- [Troubleshooting Guide](../troubleshooting.md)
- [Database Management](../database-management.md)

---

**Need Help?** Open an issue on [GitHub](https://github.com/dtoyoda10/mercur-railway-boilerplate/issues)

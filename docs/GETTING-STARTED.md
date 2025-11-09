# Getting Started

This guide will help you set up the marketplace locally and deploy it to Railway.

## Local Development Setup

### 1. Install Requirements

You need these installed on your computer:
- Node.js 20 or higher
- PNPM 10 or higher (`npm install -g pnpm`)
- Docker Desktop
- Git

### 2. Clone and Install

```bash
git clone https://github.com/dtoyoda10/mercur-railway-boilerplate.git
cd mercur-railway-boilerplate
pnpm install
```

### 3. Start Database

```bash
docker-compose up -d
```

This starts PostgreSQL and Redis in Docker containers.

### 4. Configure Environment Variables

Copy the template files:

```bash
cp apps/backend/.env.template apps/backend/.env
cp apps/storefront/.env.template apps/storefront/.env
cp apps/vendor-panel/.env.template apps/vendor-panel/.env
```

Edit `apps/backend/.env` and set:

```env
DATABASE_URL=postgres://postgres:postgres@localhost:5432/medusa_db
JWT_SECRET=any-random-string-here
COOKIE_SECRET=another-random-string
STORE_CORS=http://localhost:8000
ADMIN_CORS=http://localhost:7000
```

For local development, you can use simple secrets. For production, generate strong ones with:
```bash
openssl rand -base64 32
```

Edit `apps/storefront/.env`:

```env
MEDUSA_BACKEND_URL=http://localhost:9000
```

Edit `apps/vendor-panel/.env`:

```env
VITE_MEDUSA_BACKEND_URL=http://localhost:9000
```

### 5. Initialize Database

```bash
pnpm run db:init
pnpm run db:admin
```

The second command creates an admin user. Use any email and password you want.

### 6. Start Development Servers

```bash
pnpm run dev
```

This starts all three services. Wait a minute for everything to compile.

Access your marketplace:
- Storefront: http://localhost:8000
- Admin Panel: http://localhost:7000
- Backend API: http://localhost:9000

### 7. Add Sample Data (Optional)

```bash
cd apps/backend
npm run seed
```

This creates sample products and a test vendor account (email: seller@mercurjs.com, password: secret).

## Deploy to Railway

### Prerequisites

Create accounts on:
- [Railway](https://railway.app/) - for hosting
- [Stripe](https://stripe.com/) - for payments
- [Resend](https://resend.com/) - for emails

Algolia and TalkJS are optional for now.

### Step 1: Fork the Repository

Go to GitHub and fork this repository to your account.

### Step 2: Create Railway Project

1. Log in to Railway
2. Click "New Project"
3. Select "Deploy from GitHub repo"
4. Choose your forked repository
5. Railway will create a project

### Step 3: Add PostgreSQL

1. In your Railway project, click "New"
2. Select "Database" then "PostgreSQL"
3. Railway provisions a database automatically

### Step 4: Create Backend Service

1. Click "New" then "Empty Service"
2. Name it "backend"
3. Click on the service
4. Go to Settings > Source
5. Click "Connect Repo" and select your fork
6. Set Root Directory: `apps/backend`
7. Set Build Command: `cd ../.. && pnpm install && pnpm run build:api`
8. Set Start Command: `cd ../.. && pnpm run start:api`

Add environment variables in the Variables tab:

```
DATABASE_URL=${{Postgres.DATABASE_URL}}
NODE_ENV=production
JWT_SECRET=<generate with: openssl rand -base64 32>
COOKIE_SECRET=<generate with: openssl rand -base64 32>
PORT=9000
STRIPE_SECRET_API_KEY=<your Stripe secret key>
RESEND_API_KEY=<your Resend API key>
```

Leave CORS variables empty for now, we'll add them after deploying the frontend.

Click "Deploy".

### Step 5: Create Vendor Panel Service

1. Click "New" then "Empty Service"
2. Name it "vendor-panel"
3. Click on it and go to Settings > Source
4. Connect to your repo
5. Set Root Directory: `apps/vendor-panel`
6. Set Build Command: `cd ../.. && pnpm install && pnpm run build:vendor`
7. Set Start Command: `cd ../.. && pnpm run start:vendor`

Add variables:

```
MEDUSA_BACKEND_URL=<your backend URL from step 4>
PORT=7000
```

Deploy.

### Step 6: Create Storefront Service

1. Click "New" then "Empty Service"
2. Name it "storefront"
3. Connect to your repo
4. Set Root Directory: `apps/storefront`
5. Set Build Command: `cd ../.. && pnpm install && pnpm run build:store`
6. Set Start Command: `cd ../.. && pnpm run start:store`

Add variables:

```
MEDUSA_BACKEND_URL=<your backend URL>
PORT=8000
```

Deploy.

### Step 7: Generate Domains

For each service:
1. Go to Settings > Networking
2. Click "Generate Domain"
3. Copy the URL

### Step 8: Update CORS Settings

Go back to your backend service and add these variables:

```
STORE_CORS=<your storefront URL>
ADMIN_CORS=<your vendor panel URL>
AUTH_CORS=<backend URL>,<vendor URL>,<storefront URL>
```

Redeploy the backend.

### Step 9: Run Database Migrations

1. Go to backend service
2. Click on Deployments
3. Click on the latest deployment
4. Open the Shell/Terminal
5. Run:

```bash
cd ../.. && pnpm run db:migrate
cd apps/backend && npx medusa user -e admin@yourdomain.com -p yourpassword
```

This creates the database tables and an admin user.

### You're Live!

Visit your storefront URL. Log in to the admin panel with the credentials you created.

## Next Steps

**Set up integrations:**
- [Stripe](guides/stripe-setup.md) - Required for accepting payments
- [Resend](guides/resend-setup.md) - Required for email notifications
- [Algolia](guides/algolia-setup.md) - Optional, for better search
- [TalkJS](guides/talkjs-setup.md) - Optional, for customer chat

**Before launching:**
- Read the [Production Checklist](deployment/production-checklist.md)
- Review [Security Best Practices](security.md)
- Set up [Monitoring](monitoring.md)

## Common Issues

**Build fails with "pnpm not found"**

Railway should detect pnpm automatically. If it doesn't, the build/start commands include pnpm installation.

**Services keep crashing**

Check the logs in Railway dashboard. Common causes:
- Missing environment variables
- Wrong DATABASE_URL
- Forgot to run migrations

**Can't access the site**

Make sure:
- All services show as "Active" in Railway
- Domain was generated in Settings > Networking
- No typos in environment variable URLs

More help in [Troubleshooting Guide](troubleshooting.md).

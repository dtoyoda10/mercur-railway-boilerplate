# Quickstart Guide

Get the marketplace running locally in 5 minutes.

## Step 1: Install

```bash
git clone https://github.com/dtoyoda10/mercur-railway-boilerplate.git
cd mercur-railway-boilerplate
pnpm install
```

## Step 2: Start Services

```bash
docker-compose up -d
```

This starts PostgreSQL and Redis.

## Step 3: Configure

```bash
cp apps/backend/.env.template apps/backend/.env
cp apps/storefront/.env.template apps/storefront/.env
cp apps/vendor-panel/.env.template apps/vendor-panel/.env
```

Edit `apps/backend/.env` - the important ones:

```env
DATABASE_URL=postgres://postgres:postgres@localhost:5432/medusa_db
JWT_SECRET=test-secret
COOKIE_SECRET=test-secret
STORE_CORS=http://localhost:8000
ADMIN_CORS=http://localhost:7000
```

Leave other variables for later. You can add Stripe, Resend, etc. when you need them.

## Step 4: Setup Database

```bash
pnpm run db:init
pnpm run db:admin
```

Create an admin user when prompted. Use any email/password.

## Step 5: Run

```bash
pnpm run dev
```

Wait about a minute for everything to compile.

Done! Visit:
- Storefront: http://localhost:8000
- Admin: http://localhost:7000 (log in with your admin credentials)

## Add Sample Products

```bash
cd apps/backend
npm run seed
```

This creates sample products you can browse on the storefront.

## What's Next?

**Add integrations when ready:**
- Stripe - for payments (see docs/guides/stripe-setup.md)
- Resend - for emails (see docs/guides/resend-setup.md)

**Deploy to production:**
- See docs/GETTING-STARTED.md for Railway deployment

**Customize:**
- Backend code is in `apps/backend/src/`
- Storefront is in `apps/storefront/src/`
- Admin customizations in `apps/vendor-panel/src/`

## Troubleshooting

**Docker won't start:** Make sure Docker Desktop is running

**Port already in use:** Something else is using ports 5432, 6379, 7000, 8000, or 9000. Stop those services first.

**Database connection error:** Check PostgreSQL is running with `docker ps`

**Still stuck?** See docs/troubleshooting.md or open an issue.

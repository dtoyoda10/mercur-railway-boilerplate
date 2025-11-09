# Troubleshooting Guide

Common issues and how to fix them.

## Table of Contents

- [Installation Issues](#installation-issues)
- [Database Problems](#database-problems)
- [Backend API Issues](#backend-api-issues)
- [Storefront Issues](#storefront-issues)
- [Vendor Panel Issues](#vendor-panel-issues)
- [Railway Deployment Issues](#railway-deployment-issues)
- [Integration Issues](#integration-issues)

---

## Installation Issues

### `pnpm: command not found`

You need to install pnpm first.

**Fix:**
```bash
npm install -g pnpm
```

Or use corepack (comes with Node 16+):
```bash
corepack enable
corepack prepare pnpm@10 --activate
```

### `pnpm install` fails with "EACCES" permission error

**Fix:**
```bash
# Change npm's default directory
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'

# Add to your ~/.bashrc or ~/.zshrc:
export PATH=~/.npm-global/bin:$PATH

# Reload shell
source ~/.bashrc  # or source ~/.zshrc
```

### Turbo cache errors

**Fix:**
```bash
# Clear turbo cache
rm -rf node_modules/.cache/turbo
pnpm run build --force
```

---

## Database Problems

### Can't connect to PostgreSQL

**Error:**
```
Error: connect ECONNREFUSED 127.0.0.1:5432
```

**Fixes:**

1. **PostgreSQL not running:**
   ```bash
   # If using Docker:
   docker-compose up -d postgres

   # Check it's running:
   docker ps | grep postgres
   ```

2. **Wrong DATABASE_URL:**
   ```env
   # Check your .env has correct format:
   DATABASE_URL=postgres://user:password@localhost:5432/dbname
   ```

3. **PostgreSQL not installed locally:**
   ```bash
   # macOS:
   brew install postgresql@16
   brew services start postgresql@16

   # Ubuntu:
   sudo apt install postgresql-16
   sudo systemctl start postgresql
   ```

### Migrations fail

**Error:**
```
Migration failed: relation "product" already exists
```

**Fix:**

1. **Drop and recreate database (WARNING: loses all data):**
   ```bash
   # Using Docker:
   docker-compose down -v
   docker-compose up -d

   # Or manually:
   dropdb medusa_db
   createdb medusa_db
   pnpm run db:init
   ```

2. **If you need to keep data, revert migrations:**
   ```bash
   cd apps/backend
   npx medusa migrations revert
   ```

### "too many connections" error

PostgreSQL has a connection limit (usually 100).

**Temporary fix:**
```bash
# Restart PostgreSQL
docker-compose restart postgres
```

**Permanent fix:**

Edit PostgreSQL config to increase `max_connections`:

```bash
# If using Docker, add to docker-compose.yml:
postgres:
  command: postgres -c max_connections=200
```

---

## Backend API Issues

### Backend won't start

**Error:**
```
Error: Missing environment variable: JWT_SECRET
```

**Fix:**

1. Check `.env` file exists in `apps/backend/`
2. Make sure all required vars are set:
   ```env
   DATABASE_URL=...
   JWT_SECRET=...
   COOKIE_SECRET=...
   ```

3. Restart the backend:
   ```bash
   cd apps/backend
   pnpm run dev
   ```

### Backend crashes on startup

**Check the logs** for the actual error. Common issues:

1. **Port already in use:**
   ```
   Error: listen EADDRINUSE: address already in use :::9000
   ```

   **Fix:**
   ```bash
   # Kill process on port 9000:
   lsof -ti:9000 | xargs kill -9
   # Or change port in .env:
   PORT=9001
   ```

2. **Database schema out of sync:**
   ```bash
   pnpm run db:migrate
   ```

### API returns 401 Unauthorized

**For admin endpoints:**

You need to authenticate first.

**Fix:**

1. Get admin auth cookie by logging in through vendor panel
2. Or use API key:
   ```bash
   curl http://localhost:9000/admin/products \
     -H "Authorization: Bearer YOUR_API_KEY"
   ```

3. Create API key:
   ```bash
   cd apps/backend
   npx medusa user -e admin@test.com -p password
   # Log in via vendor panel, then create API key in Settings
   ```

### CORS errors in browser console

**Error:**
```
Access to fetch at 'http://localhost:9000' from origin 'http://localhost:8000'
has been blocked by CORS policy
```

**Fix:**

Update CORS settings in backend `.env`:

```env
STORE_CORS=http://localhost:8000
ADMIN_CORS=http://localhost:7000
AUTH_CORS=http://localhost:9000,http://localhost:7000,http://localhost:8000
```

Restart backend after changing.

---

## Storefront Issues

### Storefront shows blank page

**Check browser console** (F12) for errors.

**Common fixes:**

1. **Backend not running:**
   ```bash
   # Make sure backend is up:
   curl http://localhost:9000/health
   # Should return: {"status": "ok"}
   ```

2. **Wrong backend URL in `.env`:**
   ```env
   # apps/storefront/.env
   MEDUSA_BACKEND_URL=http://localhost:9000  # No trailing slash!
   ```

3. **Missing publishable key:**
   ```env
   NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY=pk_...
   ```

   Get it by running:
   ```bash
   pnpm run db:init
   # Copy the publishable key from output
   ```

### Products not showing

**Checks:**

1. **Products exist in database:**
   - Log in to vendor panel
   - Go to Products
   - Create a test product

2. **Products are published:**
   - Edit product
   - Set status to "Published"
   - Add to a sales channel

3. **Storefront is using correct publishable key:**
   - Check `NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY`
   - Should match a key in backend Settings → Publishable API Keys

### Next.js build fails

**Error:**
```
Type error: Property 'products' does not exist on type...
```

**Fix:**

1. **Regenerate API types:**
   ```bash
   cd apps/backend
   pnpm run generate:oas

   cd ../..
   pnpm run codegen
   ```

2. **Clear Next.js cache:**
   ```bash
   cd apps/storefront
   rm -rf .next
   pnpm run build
   ```

---

## Vendor Panel Issues

### Can't log in to vendor panel

**Error:**
```
Invalid credentials
```

**Fix:**

1. **Create admin user:**
   ```bash
   cd apps/backend
   npx medusa user -e admin@test.com -p password
   ```

2. **Reset password:**
   - Go to login page
   - Click "Forgot password"
   - Or create a new admin user

### Vendor panel shows "Cannot connect to server"

**Fixes:**

1. **Backend not running:**
   ```bash
   curl http://localhost:9000/health
   ```

2. **Wrong backend URL:**
   ```env
   # apps/vendor-panel/.env
   VITE_MEDUSA_BACKEND_URL=http://localhost:9000
   ```

3. **Restart vendor panel:**
   ```bash
   cd apps/vendor-panel
   pnpm run dev
   ```

### Vendor panel build fails

**Error:**
```
Module not found: Can't resolve '@medusajs/ui'
```

**Fix:**

```bash
# Reinstall dependencies
rm -rf node_modules
pnpm install

# Try building again
pnpm run build:vendor
```

---

## Railway Deployment Issues

### Build fails on Railway

**Error:**
```
ERROR: No pnpm-lock.yaml found
```

**Fix:**

Make sure you committed `pnpm-lock.yaml`:
```bash
git add pnpm-lock.yaml
git commit -m "Add pnpm lockfile"
git push
```

### Service crashes after deployment

**Check Railway logs:**

1. Go to Railway Dashboard
2. Click on your service
3. Go to **Deployments** → **View Logs**

**Common issues:**

1. **Missing environment variables:**
   - Check all required vars are set in Railway Variables tab

2. **Database connection failed:**
   - Verify `DATABASE_URL` references PostgreSQL service: `${{Postgres.DATABASE_URL}}`

3. **Port binding error:**
   - Railway auto-sets `PORT` variable
   - Don't hardcode port in your app

### "Service Unavailable" 503 error

**Checks:**

1. **Service is running:**
   - Check Railway dashboard shows "Active"
   - Look at deployment logs for crashes

2. **Health check passing:**
   ```bash
   curl https://your-app.railway.app/health
   ```

3. **Start command is correct:**
   - Backend: `pnpm run start:api`
   - Vendor: `pnpm run start:vendor`
   - Storefront: `pnpm run start:store`

### CORS errors in production

**Fix:**

Update backend CORS vars to include your production URLs:

```env
# Railway backend variables
STORE_CORS=https://your-storefront.railway.app
ADMIN_CORS=https://your-vendor.railway.app
```

Redeploy backend after changing.

### Database migrations not running

Railway doesn't auto-run migrations.

**Fix:**

1. **Manual via Railway shell:**
   - Go to backend service
   - Click **Shell** (or **Deployments** → **Shell**)
   - Run:
     ```bash
     cd ../.. && pnpm run db:migrate
     ```

2. **Or add release command** to `apps/backend/railway.json`:
   ```json
   {
     "deploy": {
       "releaseCommand": "cd ../.. && pnpm run db:migrate"
     }
   }
   ```

---

## Integration Issues

### Stripe payments not working

**Error in browser:**
```
Stripe: No such payment_intent
```

**Fixes:**

1. **Wrong API keys:**
   - Test keys in development
   - Live keys in production
   - Check `STRIPE_SECRET_API_KEY` in backend

2. **Webhook not set up:**
   - Create webhook in Stripe Dashboard
   - Set `STRIPE_CONNECTED_ACCOUNTS_WEBHOOK_SECRET`

3. **CORS blocking Stripe:**
   - Add Stripe domain to CORS (usually not needed, but check)

### Algolia search not working

**No search results showing:**

**Fixes:**

1. **Products not indexed:**
   - Check Algolia dashboard → Browse index
   - Manually reindex:
     ```bash
     curl -X POST http://localhost:9000/admin/algolia/reindex
     ```

2. **Wrong API keys:**
   - Backend needs **Admin API Key**
   - Frontend needs **Search-Only API Key**

3. **Index name mismatch:**
   - Check index name in code matches Algolia dashboard

### Resend emails not sending

**Error:**
```
Resend: Invalid API key
```

**Fixes:**

1. **API key wrong:**
   - Regenerate in Resend dashboard
   - Update `RESEND_API_KEY` in backend

2. **From email not verified:**
   - Verify domain in Resend
   - Update `RESEND_FROM_EMAIL`

3. **Check Resend logs:**
   - Resend Dashboard → Logs
   - See what's failing

### TalkJS chat not loading

**Fixes:**

1. **Wrong App ID:**
   - Check `TALK_JS_APP_ID` in vendor panel `.env`

2. **User not authenticated:**
   - TalkJS needs logged-in users
   - Check user session exists

3. **JavaScript error:**
   - Open browser console
   - Look for TalkJS errors

---

## General Debugging Tips

### Enable debug logs

**Backend:**
```bash
# Run with debug output
DEBUG=medusa:* pnpm run dev
```

**Next.js:**
```bash
# Enable verbose logging
NODE_OPTIONS='--inspect' pnpm run dev
```

### Clear all caches

When weird things happen:

```bash
# Clear all node_modules
rm -rf node_modules
rm -rf apps/*/node_modules
rm -rf packages/*/node_modules

# Clear pnpm cache
pnpm store prune

# Reinstall
pnpm install

# Clear build caches
rm -rf .turbo
rm -rf apps/storefront/.next
rm -rf apps/backend/dist

# Rebuild
pnpm run build
```

### Check service health

**Quick health checks:**

```bash
# Backend
curl http://localhost:9000/health

# PostgreSQL
docker exec mercur-postgres pg_isready

# Redis
docker exec mercur-redis redis-cli ping
```

### Still stuck?

1. **Check GitHub Issues:** [github.com/dtoyoda10/mercur-railway-boilerplate/issues](https://github.com/dtoyoda10/mercur-railway-boilerplate/issues)
2. **Search MedusaJS Docs:** [docs.medusajs.com](https://docs.medusajs.com)
3. **Ask in Discussions:** GitHub Discussions tab
4. **Open a new issue:** Provide logs, error messages, and steps to reproduce

---

**Pro tip:** When asking for help, include:
- Error message (full stack trace)
- What you were trying to do
- Environment (Node version, OS, Railway/local)
- Relevant config files (.env with secrets redacted)
- Logs from the service that's failing

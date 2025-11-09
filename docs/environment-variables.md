# Environment Variables Reference

Complete reference for all environment variables used across the MercurJS Railway Boilerplate.

## Table of Contents

- [Overview](#overview)
- [Backend API Variables](#backend-api-variables)
- [Storefront Variables](#storefront-variables)
- [Vendor Panel Variables](#vendor-panel-variables)
- [Environment-Specific Configurations](#environment-specific-configurations)
- [Security Best Practices](#security-best-practices)

## Overview

This boilerplate uses environment variables to configure three applications:

| Application | Config File Location | Required Variables |
|-------------|---------------------|-------------------|
| **Backend API** | `apps/backend/.env` | 21 variables |
| **Storefront** | `apps/storefront/.env` | 10 variables |
| **Vendor Panel** | `apps/vendor-panel/.env` | 5 variables |

**Quick Start:**
```bash
# Copy template files
cp apps/backend/.env.template apps/backend/.env
cp apps/storefront/.env.template apps/storefront/.env
cp apps/vendor-panel/.env.template apps/vendor-panel/.env

# Edit each file with your values
```

---

## Backend API Variables

Location: `apps/backend/.env`

### Core Configuration

#### `NODE_ENV`
- **Type:** String
- **Required:** Yes
- **Default:** `development`
- **Valid Values:** `development`, `production`, `test`
- **Description:** Sets the application environment mode
- **Example:**
  ```env
  NODE_ENV=production
  ```

#### `DATABASE_URL`
- **Type:** String (PostgreSQL Connection URI)
- **Required:** Yes
- **Description:** PostgreSQL database connection string
- **Format:** `postgres://user:password@host:port/database`
- **Local Example:**
  ```env
  DATABASE_URL=postgres://postgres:password@localhost:5432/medusa_db
  ```
- **Railway Example:**
  ```env
  DATABASE_URL=${{Postgres.DATABASE_URL}}
  ```
- **Notes:**
  - Railway auto-populates this when using PostgreSQL plugin
  - Use SSL in production: append `?sslmode=require`

#### `PORT`
- **Type:** Number
- **Required:** No
- **Default:** `9000`
- **Description:** Port for the backend API server
- **Example:**
  ```env
  PORT=9000
  ```

### Authentication & Security

#### `JWT_SECRET`
- **Type:** String
- **Required:** Yes
- **Description:** Secret key for signing JWT tokens
- **Security:** ⚠️ **CRITICAL** - Must be a strong, random string
- **Generate:**
  ```bash
  openssl rand -base64 32
  ```
- **Example:**
  ```env
  JWT_SECRET=A7x9K2mP5nQ8wR4tY6uI1oP3aSdFgHjKlZxCvBnM
  ```
- **Notes:**
  - Change this in production
  - Never commit real secrets to git
  - Use different values for dev/staging/production

#### `COOKIE_SECRET`
- **Type:** String
- **Required:** Yes
- **Description:** Secret key for signing session cookies
- **Security:** ⚠️ **CRITICAL** - Must be a strong, random string
- **Generate:**
  ```bash
  openssl rand -base64 32
  ```
- **Example:**
  ```env
  COOKIE_SECRET=Z1mN8bV3cX5aS7dF9gH2jK4lP6qW0eR1tY3uI5oP
  ```

### CORS Configuration

#### `STORE_CORS`
- **Type:** String (Comma-separated URLs)
- **Required:** Yes
- **Description:** Allowed origins for storefront API requests
- **Local Example:**
  ```env
  STORE_CORS=http://localhost:8000
  ```
- **Production Example:**
  ```env
  STORE_CORS=https://shop.yourdomain.com,https://www.yourdomain.com
  ```
- **Notes:** Multiple origins separated by commas, no spaces

#### `ADMIN_CORS`
- **Type:** String (Comma-separated URLs)
- **Required:** Yes
- **Description:** Allowed origins for admin/vendor panel API requests
- **Local Example:**
  ```env
  ADMIN_CORS=http://localhost:7000
  ```
- **Production Example:**
  ```env
  ADMIN_CORS=https://admin.yourdomain.com
  ```

#### `VENDOR_CORS`
- **Type:** String (Comma-separated URLs)
- **Required:** Yes (for marketplace features)
- **Description:** Allowed origins for vendor-specific operations
- **Example:**
  ```env
  VENDOR_CORS=http://localhost:7000
  ```

#### `AUTH_CORS`
- **Type:** String (Comma-separated URLs)
- **Required:** Yes
- **Description:** Allowed origins for authentication endpoints
- **Example:**
  ```env
  AUTH_CORS=http://localhost:9000,http://localhost:7000,http://localhost:8000
  ```
- **Notes:** Should include all frontend URLs that need authentication

### Redis (Optional)

#### `REDIS_URL`
- **Type:** String (Redis Connection URI)
- **Required:** No (but recommended for production)
- **Description:** Redis connection for caching and job queues
- **Format:** `redis://user:password@host:port`
- **Local Example:**
  ```env
  REDIS_URL=redis://localhost:6379
  ```
- **Railway Example:**
  ```env
  REDIS_URL=${{Redis.REDIS_URL}}
  ```
- **Benefits:**
  - Faster API responses with caching
  - Background job processing
  - Session storage
  - Rate limiting

### Payment Integration (Stripe)

#### `STRIPE_SECRET_API_KEY`
- **Type:** String
- **Required:** Yes (if using payments)
- **Description:** Stripe secret API key for payment processing
- **Format:** `sk_test_...` (test) or `sk_live_...` (production)
- **Example:**
  ```env
  STRIPE_SECRET_API_KEY=sk_test_51Abc...xyz
  ```
- **Where to Find:** [Stripe Dashboard](https://dashboard.stripe.com/apikeys)
- **Security:** ⚠️ Never expose in client-side code

#### `STRIPE_CONNECTED_ACCOUNTS_WEBHOOK_SECRET`
- **Type:** String
- **Required:** Yes (if using Stripe Connect)
- **Description:** Webhook signing secret for Stripe Connect events
- **Format:** `whsec_...`
- **Example:**
  ```env
  STRIPE_CONNECTED_ACCOUNTS_WEBHOOK_SECRET=whsec_abc123xyz
  ```
- **Where to Find:** Stripe Dashboard → Developers → Webhooks
- **Setup Guide:** [docs/guides/stripe-setup.md](guides/stripe-setup.md)

### Email Service (Resend)

#### `RESEND_API_KEY`
- **Type:** String
- **Required:** Yes (if using email notifications)
- **Description:** Resend API key for sending transactional emails
- **Format:** `re_...`
- **Example:**
  ```env
  RESEND_API_KEY=re_abc123xyz
  ```
- **Where to Get:** [Resend Dashboard](https://resend.com/api-keys)
- **Setup Guide:** [docs/guides/resend-setup.md](guides/resend-setup.md)

#### `RESEND_FROM_EMAIL`
- **Type:** String (Email address)
- **Required:** Yes (if using Resend)
- **Description:** Sender email address for outgoing emails
- **Example:**
  ```env
  RESEND_FROM_EMAIL=noreply@yourdomain.com
  ```
- **Notes:**
  - Must be a verified domain in Resend
  - Default `onboarding@resend.dev` only works in test mode

### Search Integration (Algolia)

#### `ALGOLIA_APP_ID`
- **Type:** String
- **Required:** Yes (if using search)
- **Description:** Algolia application ID
- **Example:**
  ```env
  ALGOLIA_APP_ID=ABC123XYZ
  ```
- **Where to Find:** [Algolia Dashboard](https://www.algolia.com/apps) → Settings → API Keys

#### `ALGOLIA_API_KEY`
- **Type:** String
- **Required:** Yes (if using search)
- **Description:** Algolia Admin API Key (for indexing)
- **Security:** ⚠️ Use Admin API Key on backend only
- **Example:**
  ```env
  ALGOLIA_API_KEY=abc123xyz456def789
  ```
- **Setup Guide:** [docs/guides/algolia-setup.md](guides/algolia-setup.md)

### Chat Integration (TalkJS)

#### `TALK_JS_APP_ID`
- **Type:** String
- **Required:** Yes (if using chat)
- **Description:** TalkJS application ID for vendor-customer messaging
- **Example:**
  ```env
  TALK_JS_APP_ID=tXXXXXXX
  ```
- **Where to Get:** [TalkJS Dashboard](https://talkjs.com/dashboard/)

#### `TALK_JS_SECRET_API_KEY`
- **Type:** String
- **Required:** Yes (if using TalkJS)
- **Description:** TalkJS secret key for server-side operations
- **Example:**
  ```env
  TALK_JS_SECRET_API_KEY=sk_test_...
  ```
- **Security:** ⚠️ Never expose in frontend code

### File Storage (S3-Compatible)

These variables are optional if using local file storage. Required for production with AWS S3, Cloudflare R2, or MinIO.

#### `MINIO_ENDPOINT`
- **Type:** String (URL)
- **Required:** No (optional for file storage)
- **Description:** S3-compatible storage endpoint
- **Example (Cloudflare R2):**
  ```env
  MINIO_ENDPOINT=https://your-account.r2.cloudflarestorage.com
  ```
- **Example (AWS S3):**
  ```env
  MINIO_ENDPOINT=https://s3.amazonaws.com
  ```

#### `MINIO_ACCESS_KEY`
- **Type:** String
- **Required:** If using S3 storage
- **Description:** Access key for S3-compatible storage
- **Example:**
  ```env
  MINIO_ACCESS_KEY=AKIAIOSFODNN7EXAMPLE
  ```

#### `MINIO_SECRET_KEY`
- **Type:** String
- **Required:** If using S3 storage
- **Description:** Secret key for S3-compatible storage
- **Example:**
  ```env
  MINIO_SECRET_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
  ```

#### `MINIO_BUCKET`
- **Type:** String
- **Required:** If using S3 storage
- **Description:** Bucket name for storing files
- **Example:**
  ```env
  MINIO_BUCKET=my-ecommerce-files
  ```

---

## Storefront Variables

Location: `apps/storefront/.env`

### Backend Connection

#### `MEDUSA_BACKEND_URL`
- **Type:** String (URL)
- **Required:** Yes
- **Description:** URL of the MedusaJS backend API
- **Local Example:**
  ```env
  MEDUSA_BACKEND_URL=http://localhost:9000
  ```
- **Production Example:**
  ```env
  MEDUSA_BACKEND_URL=https://api.yourdomain.com
  ```
- **Notes:** No trailing slash

#### `NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY`
- **Type:** String
- **Required:** Yes
- **Description:** Publishable API key for client-side requests
- **How to Get:**
  1. Run backend API
  2. Run `pnpm run db:init` (creates default key)
  3. Or create via Admin UI: Settings → Publishable API Keys
- **Example:**
  ```env
  NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY=pk_abc123xyz
  ```
- **Notes:** Safe to expose in frontend (starts with `pk_`)

### Site Configuration

#### `NEXT_PUBLIC_BASE_URL`
- **Type:** String (URL)
- **Required:** Yes
- **Description:** Base URL of the storefront (for SEO, OG tags)
- **Local Example:**
  ```env
  NEXT_PUBLIC_BASE_URL=http://localhost:8000
  ```
- **Production Example:**
  ```env
  NEXT_PUBLIC_BASE_URL=https://shop.yourdomain.com
  ```

#### `NEXT_PUBLIC_DEFAULT_REGION`
- **Type:** String (Region code)
- **Required:** No
- **Default:** `us`
- **Description:** Default region/locale for the storefront
- **Example:**
  ```env
  NEXT_PUBLIC_DEFAULT_REGION=us
  ```
- **Valid Values:** Any region code configured in MedusaJS (e.g., `us`, `eu`, `uk`, `pl`)

#### `NEXT_PUBLIC_SITE_NAME`
- **Type:** String
- **Required:** No
- **Default:** `"Fleek Marketplace"`
- **Description:** Site name for meta tags and branding
- **Example:**
  ```env
  NEXT_PUBLIC_SITE_NAME="My Awesome Store"
  ```

#### `NEXT_PUBLIC_SITE_DESCRIPTION`
- **Type:** String
- **Required:** No
- **Description:** Site description for SEO meta tags
- **Example:**
  ```env
  NEXT_PUBLIC_SITE_DESCRIPTION="The best marketplace for everything"
  ```

### Payment (Client-side)

#### `NEXT_PUBLIC_STRIPE_KEY`
- **Type:** String
- **Required:** Yes (if using Stripe payments)
- **Description:** Stripe publishable key for client-side checkout
- **Format:** `pk_test_...` (test) or `pk_live_...` (production)
- **Example:**
  ```env
  NEXT_PUBLIC_STRIPE_KEY=pk_test_51Abc...xyz
  ```
- **Where to Find:** [Stripe Dashboard](https://dashboard.stripe.com/apikeys)
- **Security:** ✅ Safe to expose (publishable key)

### Search (Client-side)

#### `NEXT_PUBLIC_ALGOLIA_ID`
- **Type:** String
- **Required:** Yes (if using search)
- **Description:** Algolia application ID for client-side search
- **Example:**
  ```env
  NEXT_PUBLIC_ALGOLIA_ID=ABC123XYZ
  ```

#### `NEXT_PUBLIC_ALGOLIA_SEARCH_KEY`
- **Type:** String
- **Required:** Yes (if using search)
- **Description:** Algolia Search-Only API Key
- **Security:** ✅ Use Search-Only key (not Admin key)
- **Example:**
  ```env
  NEXT_PUBLIC_ALGOLIA_SEARCH_KEY=abc123xyz456def789
  ```

### Next.js Configuration

#### `REVALIDATE_SECRET`
- **Type:** String
- **Required:** No (recommended for ISR)
- **Description:** Secret for on-demand revalidation of static pages
- **Generate:**
  ```bash
  openssl rand -base64 16
  ```
- **Example:**
  ```env
  REVALIDATE_SECRET=my-secret-revalidate-token
  ```
- **Usage:** For webhooks to trigger page rebuilds

---

## Vendor Panel Variables

Location: `apps/vendor-panel/.env`

### Backend Connection

#### `VITE_MEDUSA_BACKEND_URL`
- **Type:** String (URL)
- **Required:** Yes
- **Description:** MedusaJS backend API URL for vendor panel
- **Local Example:**
  ```env
  VITE_MEDUSA_BACKEND_URL=http://localhost:9000
  ```
- **Production Example:**
  ```env
  VITE_MEDUSA_BACKEND_URL=https://api.yourdomain.com
  ```

#### `VITE_MEDUSA_BASE`
- **Type:** String
- **Required:** No
- **Default:** `'/'`
- **Description:** Base path for Medusa admin routes
- **Example:**
  ```env
  VITE_MEDUSA_BASE=/
  ```

### Integration URLs

#### `VITE_MEDUSA_STOREFRONT_URL`
- **Type:** String (URL)
- **Required:** No (for quick navigation)
- **Description:** Storefront URL for quick access from vendor panel
- **Example:**
  ```env
  VITE_MEDUSA_STOREFRONT_URL=http://localhost:8000
  ```

### Chat Integration

#### `VITE_TALK_JS_APP_ID`
- **Type:** String
- **Required:** Yes (if using vendor chat)
- **Description:** TalkJS application ID for vendor messaging
- **Example:**
  ```env
  VITE_TALK_JS_APP_ID=tXXXXXXX
  ```

### Feature Flags

#### `VITE_DISABLE_SELLERS_REGISTRATION`
- **Type:** Boolean
- **Required:** No
- **Default:** `false`
- **Description:** Disable public seller registration (admin-only seller creation)
- **Example:**
  ```env
  VITE_DISABLE_SELLERS_REGISTRATION=true
  ```
- **Use Case:** Set to `true` for closed marketplaces

---

## Environment-Specific Configurations

### Local Development

**Backend** (`apps/backend/.env`):
```env
NODE_ENV=development
DATABASE_URL=postgres://postgres:password@localhost:5432/medusa_db
REDIS_URL=redis://localhost:6379
JWT_SECRET=local-dev-jwt-secret-change-in-production
COOKIE_SECRET=local-dev-cookie-secret-change-in-production
STORE_CORS=http://localhost:8000
ADMIN_CORS=http://localhost:7000
VENDOR_CORS=http://localhost:7000
AUTH_CORS=http://localhost:9000,http://localhost:7000,http://localhost:8000
PORT=9000

# Optional: Test mode keys for development
STRIPE_SECRET_API_KEY=sk_test_...
RESEND_API_KEY=re_...
ALGOLIA_APP_ID=...
ALGOLIA_API_KEY=...
```

**Storefront** (`apps/storefront/.env`):
```env
MEDUSA_BACKEND_URL=http://localhost:9000
NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY=pk_...
NEXT_PUBLIC_BASE_URL=http://localhost:8000
NEXT_PUBLIC_DEFAULT_REGION=us
NEXT_PUBLIC_STRIPE_KEY=pk_test_...
NEXT_PUBLIC_SITE_NAME="Local Dev Store"
```

**Vendor Panel** (`apps/vendor-panel/.env`):
```env
VITE_MEDUSA_BACKEND_URL=http://localhost:9000
VITE_MEDUSA_STOREFRONT_URL=http://localhost:8000
VITE_MEDUSA_BASE=/
VITE_DISABLE_SELLERS_REGISTRATION=false
```

### Production (Railway)

Key differences for production:

1. **Use Railway variables:** `${{Postgres.DATABASE_URL}}`
2. **Use HTTPS URLs:** `https://` not `http://`
3. **Strong secrets:** Generate with `openssl rand -base64 32`
4. **Production API keys:** Use `pk_live_`, `sk_live_` for Stripe
5. **Verified domains:** Use your own domain for `RESEND_FROM_EMAIL`

**Backend** (Railway variables):
```env
NODE_ENV=production
DATABASE_URL=${{Postgres.DATABASE_URL}}
REDIS_URL=${{Redis.REDIS_URL}}  # if using Redis addon
JWT_SECRET=[strong-random-string]
COOKIE_SECRET=[strong-random-string]
STORE_CORS=https://shop.yourdomain.com
ADMIN_CORS=https://admin.yourdomain.com
VENDOR_CORS=https://admin.yourdomain.com
AUTH_CORS=https://api.yourdomain.com,https://admin.yourdomain.com,https://shop.yourdomain.com
PORT=9000
STRIPE_SECRET_API_KEY=sk_live_...
RESEND_API_KEY=re_...
RESEND_FROM_EMAIL=noreply@yourdomain.com
```

### Testing Environment

For running tests, create `apps/backend/.env.test`:

```env
NODE_ENV=test
DATABASE_URL=postgres://postgres:password@localhost:5432/medusa_test
JWT_SECRET=test-jwt-secret
COOKIE_SECRET=test-cookie-secret
STORE_CORS=http://localhost:8000
ADMIN_CORS=http://localhost:7000
VENDOR_CORS=http://localhost:7000
AUTH_CORS=http://localhost:9000
```

---

## Security Best Practices

### ✅ DO

1. **Generate Strong Secrets**
   ```bash
   # Use openssl to generate random secrets
   openssl rand -base64 32
   ```

2. **Use Different Secrets Per Environment**
   - Development secrets ≠ Production secrets
   - Each environment should have unique keys

3. **Never Commit Secrets**
   ```bash
   # .gitignore should include:
   .env
   .env.local
   .env.*.local
   ```

4. **Use Environment Variables in CI/CD**
   - Store secrets in Railway/GitHub Actions/CI secrets
   - Never hardcode in code

5. **Rotate Secrets Regularly**
   - Change `JWT_SECRET` and `COOKIE_SECRET` periodically
   - Rotate API keys for third-party services

6. **Use HTTPS in Production**
   - All URLs should use `https://` protocol
   - Enable SSL/TLS for database connections

7. **Restrict CORS Origins**
   - Only list necessary origins
   - Don't use wildcards (`*`) in production

### ❌ DON'T

1. **Never expose secrets in frontend**
   - Only use `NEXT_PUBLIC_` or `VITE_` prefixed vars in frontend
   - Backend secrets stay on backend only

2. **Don't use default/weak secrets**
   ```env
   # ❌ BAD
   JWT_SECRET=supersecret
   COOKIE_SECRET=secret123

   # ✅ GOOD
   JWT_SECRET=K7mP2nQ5wR8tY1uI4oP6aSdFgHjKlZxC
   ```

3. **Don't share `.env` files**
   - Use secure methods to share secrets (1Password, LastPass, Railway vars)
   - Don't send via email/Slack/Discord

4. **Don't log secrets**
   ```javascript
   // ❌ BAD
   console.log('JWT_SECRET:', process.env.JWT_SECRET)

   // ✅ GOOD
   console.log('JWT_SECRET:', '***')
   ```

5. **Don't commit `.env` files to git**
   - Always use `.env.template` for examples
   - Add `.env` to `.gitignore`

### Secret Rotation Checklist

When rotating secrets in production:

1. ✅ Generate new secret
2. ✅ Update Railway environment variables
3. ✅ Redeploy affected services
4. ✅ Monitor for errors (old tokens may be in use)
5. ✅ Wait 24-48 hours before removing old secret
6. ✅ Update team password manager

---

## Validation Scripts

### Validate Environment Variables

Create a validation script to ensure all required variables are set:

**`apps/backend/src/scripts/validate-env.ts`**:
```typescript
const required = [
  'DATABASE_URL',
  'JWT_SECRET',
  'COOKIE_SECRET',
  'STORE_CORS',
  'ADMIN_CORS'
]

required.forEach(key => {
  if (!process.env[key]) {
    console.error(`❌ Missing required environment variable: ${key}`)
    process.exit(1)
  }
})

console.log('✅ All required environment variables are set')
```

**Run validation:**
```bash
npx tsx apps/backend/src/scripts/validate-env.ts
```

---

## Additional Resources

- [Railway Environment Variables Docs](https://docs.railway.app/deploy/variables)
- [MedusaJS Configuration](https://docs.medusajs.com/development/backend/configurations)
- [Next.js Environment Variables](https://nextjs.org/docs/basic-features/environment-variables)
- [Vite Environment Variables](https://vitejs.dev/guide/env-and-mode.html)
- [Security Best Practices](security.md)

---

**Need Help?** Open an issue on [GitHub](https://github.com/dtoyoda10/mercur-railway-boilerplate/issues)

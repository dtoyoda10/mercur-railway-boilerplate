# Security Best Practices

Keeping your marketplace secure.

## Environment Variables & Secrets

### Never Hardcode Secrets

**Bad:**
```typescript
const stripeKey = 'sk_live_abc123'  // DON'T DO THIS
```

**Good:**
```typescript
const stripeKey = process.env.STRIPE_SECRET_API_KEY
```

### Generate Strong Secrets

```bash
# Generate secure random strings
openssl rand -base64 32

# For JWT and Cookie secrets
JWT_SECRET=$(openssl rand -base64 32)
COOKIE_SECRET=$(openssl rand -base64 32)
```

### Separate Secrets Per Environment

- Development: Use test API keys
- Staging: Use separate test keys
- Production: Use live keys

Never share secrets across environments.

### Rotate Secrets Regularly

Change your secrets every 3-6 months:

1. Generate new secret
2. Update in Railway
3. Redeploy services
4. Wait 24-48 hours (old tokens might still be valid)
5. Remove old secret

## Authentication & Authorization

### Admin Access

- Use strong passwords (16+ characters)
- Enable 2FA if available
- Limit admin accounts (only give access to who needs it)
- Audit admin users quarterly
- Remove inactive accounts

### API Keys

- Use publishable keys (`pk_`) in frontend only
- Never expose secret keys (`sk_`) in client-side code
- Rotate API keys periodically
- Delete unused API keys

### Vendor Accounts

- Require email verification
- Consider admin approval for new vendors
- Implement KYC (Know Your Customer) for payouts
- Set `VITE_DISABLE_SELLERS_REGISTRATION=true` for closed marketplaces

## CORS Configuration

### Restrict Origins

**Bad:**
```env
STORE_CORS=*  # Allows any domain - DON'T DO THIS
```

**Good:**
```env
STORE_CORS=https://shop.yourdomain.com,https://www.yourdomain.com
ADMIN_CORS=https://admin.yourdomain.com
```

### Production vs Development

Development:
```env
STORE_CORS=http://localhost:8000
```

Production:
```env
STORE_CORS=https://shop.yourdomain.com
```

Never mix localhost URLs in production.

## Database Security

### Connection Security

- Use SSL for database connections in production
- Add `?sslmode=require` to PostgreSQL URLs
- Don't expose database publicly (Railway handles this)

### SQL Injection Protection

Medusa uses an ORM (Objection.js) which prevents SQL injection. But if you write raw SQL:

**Bad:**
```typescript
await db.raw(`SELECT * FROM users WHERE email = '${userEmail}'`)
```

**Good:**
```typescript
await db.raw('SELECT * FROM users WHERE email = ?', [userEmail])
```

### Backup Database

- Enable automatic backups (Railway offers this)
- Test backup restoration regularly
- Keep backups encrypted
- Store backups off-site

## Payment Security

### Stripe Security

- Never log full credit card numbers
- Use Stripe's tokenization (Stripe.js handles this)
- Validate webhooks with signing secrets
- Monitor for unusual payment patterns

### Webhook Validation

Always verify webhooks:

```typescript
// Medusa does this automatically, but here's what it does:
const sig = request.headers['stripe-signature']
const event = stripe.webhooks.constructEvent(
  request.body,
  sig,
  process.env.STRIPE_WEBHOOK_SECRET
)
```

## API Security

### Rate Limiting

Protect against abuse:

```typescript
// Add rate limiting middleware (not included by default)
// Use express-rate-limit or similar
import rateLimit from 'express-rate-limit'

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // limit each IP to 100 requests per windowMs
})

app.use('/api/', limiter)
```

### Input Validation

Always validate user input:

```typescript
// Use Zod or similar
import { z } from 'zod'

const productSchema = z.object({
  title: z.string().min(1).max(255),
  price: z.number().positive(),
  description: z.string().max(5000)
})

// Validate before processing
const validated = productSchema.parse(req.body)
```

### Sanitize Output

Prevent XSS attacks:

- Next.js/React automatically escapes output
- Be careful with `dangerouslySetInnerHTML`
- Sanitize user-generated HTML (use DOMPurify)

## File Upload Security

### Validate File Types

```typescript
const allowedTypes = ['image/jpeg', 'image/png', 'image/webp']

if (!allowedTypes.includes(file.mimetype)) {
  throw new Error('Invalid file type')
}
```

### Limit File Size

```typescript
// In your upload handler
const MAX_SIZE = 5 * 1024 * 1024  // 5MB

if (file.size > MAX_SIZE) {
  throw new Error('File too large')
}
```

### Scan for Malware

For production, consider:
- ClamAV for virus scanning
- VirusTotal API integration
- AWS S3 malware scanning (if using S3)

### Store Files Securely

- Use S3/R2 with proper permissions
- Don't store files in public directories
- Use signed URLs for private files

## HTTPS/SSL

### Always Use HTTPS in Production

Railway handles this automatically, but ensure:

- All URLs use `https://`
- Redirect HTTP to HTTPS
- Set secure cookies:
  ```typescript
  res.cookie('session', value, {
    secure: true,  // Only send over HTTPS
    httpOnly: true,  // Not accessible via JavaScript
    sameSite: 'strict'  // CSRF protection
  })
  ```

## Logging & Monitoring

### What to Log

- Authentication attempts (successful and failed)
- Payment transactions
- API errors
- Database errors
- File uploads
- Admin actions

### What NOT to Log

- Passwords
- API keys
- Credit card numbers
- Full session tokens
- Personal data (GDPR compliance)

### Example

**Bad:**
```typescript
console.log('Login attempt:', { email, password })  // DON'T LOG PASSWORDS
```

**Good:**
```typescript
console.log('Login attempt:', { email, success: true })
```

## Error Handling

### Don't Leak Info in Errors

**Bad:**
```json
{
  "error": "Database connection failed at postgresql://user:password@host:5432/db"
}
```

**Good:**
```json
{
  "error": "Service temporarily unavailable. Please try again."
}
```

Log full errors server-side, but send generic messages to users.

## Dependency Security

### Regular Updates

```bash
# Check for vulnerabilities
pnpm audit

# Fix automatically
pnpm audit --fix

# Update dependencies
pnpm update
```

### Use Dependabot

GitHub Dependabot (already enabled in most repos) will:
- Scan for vulnerable dependencies
- Create PRs to update them

## Common Vulnerabilities to Avoid

### 1. SQL Injection
**Prevention:** Use ORM or parameterized queries

### 2. XSS (Cross-Site Scripting)
**Prevention:** React/Next.js auto-escapes output

### 3. CSRF (Cross-Site Request Forgery)
**Prevention:** Use SameSite cookies, CSRF tokens

### 4. Command Injection
**Prevention:** Never pass user input to shell commands

### 5. Path Traversal
**Prevention:** Validate file paths, don't allow `../`

### 6. Insecure Deserialization
**Prevention:** Validate JSON input, use TypeScript types

### 7. Broken Authentication
**Prevention:** Use strong passwords, 2FA, session management

### 8. Sensitive Data Exposure
**Prevention:** Encrypt data at rest and in transit, use HTTPS

## Security Headers

Add security headers (Railway doesn't do this automatically):

```typescript
// In your Express/Fastify middleware
app.use((req, res, next) => {
  res.setHeader('X-Content-Type-Options', 'nosniff')
  res.setHeader('X-Frame-Options', 'DENY')
  res.setHeader('X-XSS-Protection', '1; mode=block')
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains')
  next()
})
```

Or use helmet.js:
```typescript
import helmet from 'helmet'
app.use(helmet())
```

## GDPR Compliance

If serving EU customers:

- [ ] Add cookie consent banner
- [ ] Allow users to export their data
- [ ] Allow users to delete their accounts
- [ ] Have a privacy policy
- [ ] Don't store unnecessary personal data
- [ ] Encrypt personal data
- [ ] Report data breaches within 72 hours

## Incident Response Plan

### If You Get Hacked

1. **Don't panic**
2. **Isolate affected systems** - Take services offline if needed
3. **Preserve evidence** - Don't delete logs
4. **Rotate all secrets immediately**
5. **Notify users** if their data was compromised
6. **Fix the vulnerability**
7. **Document what happened** and how you fixed it
8. **Implement monitoring** to prevent it happening again

### Have These Ready

- [ ] Incident response contact list
- [ ] Backup restoration procedure
- [ ] Communication templates for users
- [ ] Legal contact (for data breach notifications)

## Security Checklist

Before going live:

- [ ] All secrets are strong and unique
- [ ] HTTPS enabled everywhere
- [ ] CORS restricted to your domains
- [ ] Database backups enabled
- [ ] Webhook signatures validated
- [ ] Rate limiting implemented
- [ ] Error messages don't leak info
- [ ] Dependencies are up to date
- [ ] Security headers configured
- [ ] Monitoring and alerting set up
- [ ] Admin accounts use strong passwords
- [ ] 2FA enabled for admin accounts

## Resources

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Stripe Security Best Practices](https://stripe.com/docs/security/guide)
- [Railway Security](https://docs.railway.app/reference/security)
- [MedusaJS Security](https://docs.medusajs.com/development/backend/security)

## Reporting Security Issues

Found a security vulnerability? Please **don't** open a public issue.

Instead, email: security@yourdomain.com (set this up!)

Or open a private security advisory on GitHub.

---

**Remember:** Security is not a one-time thing. It's an ongoing process. Stay vigilant, keep dependencies updated, and monitor your systems regularly.

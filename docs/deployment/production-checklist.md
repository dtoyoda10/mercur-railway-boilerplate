# Production Deployment Checklist

Use this checklist before going live. It'll save you from embarrassing bugs and security issues.

## Pre-Deployment

### Environment Configuration

- [ ] All environment variables set in Railway
- [ ] Using **production** API keys (not test keys)
  - [ ] Stripe: `sk_live_` not `sk_test_`
  - [ ] Stripe publishable: `pk_live_` not `pk_test_`
- [ ] Generated strong secrets (not defaults):
  - [ ] `JWT_SECRET` - 32+ random characters
  - [ ] `COOKIE_SECRET` - 32+ random characters
  - [ ] Use: `openssl rand -base64 32`
- [ ] CORS origins set correctly:
  - [ ] `STORE_CORS` - Your storefront domain
  - [ ] `ADMIN_CORS` - Your vendor panel domain
  - [ ] No `http://localhost` URLs in production vars
- [ ] Custom domain configured for each service
- [ ] SSL/HTTPS enabled (Railway does this automatically)

### Database

- [ ] PostgreSQL running on Railway (or managed DB)
- [ ] Database migrations run successfully
- [ ] Admin user created
- [ ] Seed data loaded (if needed)
- [ ] Database backups enabled (see [Database Management](../database-management.md))
- [ ] Connection pooling configured (if high traffic expected)

### Integrations

**Stripe:**
- [ ] Stripe account fully activated
- [ ] Live API keys configured
- [ ] Webhooks set up for production domain
- [ ] Webhook signing secret updated
- [ ] Stripe Connect enabled and configured
- [ ] Test payment processed successfully

**Resend (Email):**
- [ ] Domain verified in Resend
- [ ] `RESEND_FROM_EMAIL` using verified domain
- [ ] Test email sent successfully
- [ ] Email templates look good

**Algolia (Search):**
- [ ] Products indexed in Algolia
- [ ] Search working on production
- [ ] Searchable attributes configured
- [ ] Facets set up for filtering

**TalkJS (Chat):**
- [ ] Production App ID configured
- [ ] Chat tested between vendor and customer
- [ ] Notifications enabled

### Security

- [ ] No secrets hardcoded in code
- [ ] `.env` files not committed to git
- [ ] Using HTTPS for all URLs
- [ ] CORS restricted to your domains only
- [ ] Rate limiting enabled (if available)
- [ ] SQL injection protection (Medusa handles this)
- [ ] XSS protection enabled
- [ ] Admin panel only accessible to authorized users
- [ ] Vendor registration disabled or requires approval (set `VITE_DISABLE_SELLERS_REGISTRATION=true` if needed)

### Performance

- [ ] CDN configured for static assets (Railway auto-handles this)
- [ ] Redis enabled for caching (recommended)
- [ ] Database indices in place
- [ ] Image optimization enabled (Next.js handles this)
- [ ] Lazy loading implemented for images

### Testing

- [ ] All services deployed successfully
- [ ] Health checks passing:
  - [ ] Backend: `https://api.yourdomain.com/health`
  - [ ] Vendor panel loads
  - [ ] Storefront loads
- [ ] Test user flows:
  - [ ] Customer can browse products
  - [ ] Customer can add to cart
  - [ ] Customer can checkout (test payment)
  - [ ] Customer receives order confirmation email
  - [ ] Vendor can log in
  - [ ] Vendor can create products
  - [ ] Vendor can process orders
- [ ] Mobile responsiveness tested
- [ ] Cross-browser testing (Chrome, Safari, Firefox)

### Monitoring & Logging

- [ ] Error tracking set up (Sentry, LogRocket, etc.)
- [ ] Uptime monitoring configured (UptimeRobot, Pingdom, etc.)
- [ ] Performance monitoring enabled
- [ ] Database monitoring active
- [ ] Set up alerts for:
  - [ ] Service downtime
  - [ ] High error rates
  - [ ] Database connection issues
  - [ ] Payment failures

## Launch Day

### Final Checks

- [ ] Backup database before launch
- [ ] All team members have access to:
  - [ ] Railway dashboard
  - [ ] Stripe dashboard
  - [ ] Resend dashboard
  - [ ] Algolia dashboard
  - [ ] Admin panel
- [ ] Support email/chat ready
- [ ] Terms of Service published
- [ ] Privacy Policy published
- [ ] Return/Refund policy published
- [ ] Contact page working

### Go Live

- [ ] Switch DNS to production domains
- [ ] Wait for DNS propagation (15-30 minutes)
- [ ] Test all domains resolve correctly
- [ ] SSL certificates active (check for browser warnings)
- [ ] Place a real test order end-to-end
- [ ] Verify order confirmation emails
- [ ] Check payment processing works
- [ ] Monitor error logs for first hour

### Post-Launch

- [ ] Announce launch (social media, email, etc.)
- [ ] Monitor traffic and errors closely
- [ ] Watch for payment issues
- [ ] Check email deliverability
- [ ] Respond to any user issues immediately

## Week 1 After Launch

### Monitoring

- [ ] Review error logs daily
- [ ] Check payment success rates
- [ ] Monitor email delivery rates
- [ ] Review page load times
- [ ] Check database performance
- [ ] Monitor Railway resource usage
- [ ] Watch for security issues

### User Feedback

- [ ] Collect user feedback
- [ ] Fix critical bugs immediately
- [ ] Document any issues in GitHub
- [ ] Plan improvements for next sprint

### Performance

- [ ] Review slow API endpoints
- [ ] Optimize database queries if needed
- [ ] Check if Redis is helping
- [ ] Consider upgrading Railway plan if needed

## Monthly Maintenance

- [ ] Review and rotate secrets (every 3 months)
- [ ] Update dependencies (`pnpm update`)
- [ ] Test backups are working
- [ ] Review error rates and fix recurring issues
- [ ] Check for Medusa/Next.js updates
- [ ] Review Railway costs and optimize
- [ ] Audit user permissions
- [ ] Check for abandoned carts (implement recovery emails)

## Security Audit (Quarterly)

- [ ] Review access logs
- [ ] Audit admin users (remove inactive accounts)
- [ ] Check for suspicious payment patterns
- [ ] Review webhook security
- [ ] Test backup restoration process
- [ ] Update dependencies for security patches
- [ ] Review CORS and API security settings
- [ ] Penetration testing (if budget allows)

## Scaling Checklist

When you start getting real traffic:

**Database:**
- [ ] Enable connection pooling
- [ ] Add read replicas if needed
- [ ] Optimize slow queries
- [ ] Add database indices

**Backend:**
- [ ] Enable Redis for caching
- [ ] Implement job queues for background tasks
- [ ] Vertical scaling (upgrade Railway plan)
- [ ] Horizontal scaling (multiple instances)

**Storefront:**
- [ ] Implement ISR (Incremental Static Regeneration)
- [ ] Use CDN for static assets
- [ ] Optimize images (WebP, lazy loading)
- [ ] Code splitting for JS bundles

**Monitoring:**
- [ ] Set up APM (Application Performance Monitoring)
- [ ] Implement real user monitoring (RUM)
- [ ] Track Core Web Vitals
- [ ] Monitor conversion rates

## Rollback Plan

If something goes wrong:

1. **Immediate rollback:**
   ```bash
   # In Railway dashboard:
   # Go to Deployments → Select previous working deployment → Redeploy
   ```

2. **Database rollback:**
   - Restore from latest backup (see [Database Management](../database-management.md))
   - Only if database changes caused the issue

3. **Environment variable rollback:**
   - Railway keeps history of variable changes
   - Restore previous values if needed

4. **Communication:**
   - Update status page
   - Notify users via email/social
   - Post in Discord/Slack

## Emergency Contacts

Before launch, make sure you have:

- [ ] Railway support contact
- [ ] Stripe support (for payment issues)
- [ ] Database admin contact
- [ ] DevOps/infrastructure person on call
- [ ] Emergency communication plan

## Legal & Compliance

- [ ] GDPR compliance (if serving EU customers)
  - [ ] Cookie consent implemented
  - [ ] Privacy policy in place
  - [ ] Data export/deletion functionality
- [ ] PCI DSS compliance (Stripe handles most of this)
- [ ] Terms of Service published
- [ ] Refund policy clear
- [ ] Seller agreement (for marketplace vendors)
- [ ] Sales tax collection configured (if applicable)

## Documentation

- [ ] Internal runbook for common operations
- [ ] Vendor onboarding guide
- [ ] Customer support FAQs
- [ ] API documentation (if exposing public APIs)
- [ ] Incident response playbook

---

## Quick Reference

**Health Check URLs:**
```bash
# Backend
curl https://api.yourdomain.com/health

# Check all services are up
curl https://vendor.yourdomain.com
curl https://shop.yourdomain.com
```

**Emergency Commands:**
```bash
# Railway CLI rollback
railway rollback

# Check logs
railway logs --service backend-api

# Run migration
railway run pnpm run db:migrate
```

**Key Contacts:**
- Railway Support: [railway.app/help](https://railway.app/help)
- Stripe Support: [support.stripe.com](https://support.stripe.com)
- MedusaJS Discord: [discord.gg/medusajs](https://discord.gg/medusajs)

---

**Remember:** Production is never "done". It's an ongoing process of monitoring, optimizing, and improving. Start small, monitor closely, and scale as needed.

Good luck with your launch! 🚀

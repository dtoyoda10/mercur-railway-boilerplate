# Monitoring & Health Checks

Keep your marketplace healthy and catch issues before users do.

## Health Check Endpoints

### Backend API Health

**Endpoint:** `GET /health`

```bash
# Check if backend is up
curl https://api.yourdomain.com/health

# Response:
{"status": "ok"}
```

**What it checks:**
- API server is running
- Can accept requests

**Note:** This is a basic health check. It doesn't verify database connections or external services.

### Add Custom Health Checks

Create a more comprehensive health endpoint:

```typescript
// apps/backend/src/api/health/route.ts
import { MedusaRequest, MedusaResponse } from "@medusajs/medusa"

export async function GET(req: MedusaRequest, res: MedusaResponse) {
  const checks = {
    api: "ok",
    database: await checkDatabase(),
    redis: await checkRedis(),
    stripe: await checkStripe(),
  }

  const allHealthy = Object.values(checks).every(status => status === "ok")

  res.status(allHealthy ? 200 : 503).json({
    status: allHealthy ? "ok" : "degraded",
    checks,
    timestamp: new Date().toISOString()
  })
}

async function checkDatabase() {
  try {
    // Try a simple query
    await db.raw('SELECT 1')
    return "ok"
  } catch (error) {
    return "failed"
  }
}

async function checkRedis() {
  try {
    if (!process.env.REDIS_URL) return "not_configured"
    await redis.ping()
    return "ok"
  } catch (error) {
    return "failed"
  }
}

async function checkStripe() {
  try {
    // Simple Stripe API call
    await stripe.balance.retrieve()
    return "ok"
  } catch (error) {
    return "failed"
  }
}
```

## Uptime Monitoring

### Free Tools

**UptimeRobot** (recommended for small projects)
- Free tier: 50 monitors, 5-minute checks
- Setup:
  1. Sign up at [uptimerobot.com](https://uptimerobot.com)
  2. Add monitor for `https://api.yourdomain.com/health`
  3. Set alert contacts (email, SMS, Slack)
  4. Add monitors for storefront and vendor panel

**Pingdom**
- Free tier available
- More detailed reports

**Better Uptime**
- Clean UI
- Status page included

**StatusCake**
- Free tier: 10 uptime checks

### What to Monitor

Create monitors for:
- [ ] Backend API: `https://api.yourdomain.com/health`
- [ ] Storefront homepage: `https://shop.yourdomain.com`
- [ ] Vendor panel login: `https://admin.yourdomain.com`
- [ ] Database (if publicly accessible, but usually it's not)

### Alert Thresholds

- Check interval: 5 minutes (free tiers)
- Alert after: 2 consecutive failures
- Notification channels:
  - Email (always)
  - SMS (for critical issues)
  - Slack/Discord (for team awareness)

## Application Performance Monitoring (APM)

### Free APM Tools

**Sentry** (Error Tracking)
- Free tier: 5k events/month
- Tracks errors and exceptions
- Source maps for debugging
- [sentry.io](https://sentry.io)

Setup:
```bash
pnpm add @sentry/node @sentry/nextjs
```

```typescript
// apps/backend/src/instrumentation.ts
import * as Sentry from "@sentry/node"

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 0.1, // Sample 10% of transactions
})
```

**LogRocket** (Session Replay)
- See what users did before an error
- Free tier: 1k sessions/month
- [logrocket.com](https://logrocket.com)

**Datadog** (Full APM)
- Free tier available
- Metrics, traces, logs
- [datadoghq.com](https://www.datadoghq.com/)

**New Relic** (Full APM)
- 100 GB/month free
- [newrelic.com](https://newrelic.com/)

### What to Track

**Errors:**
- 500 Internal Server Errors
- Payment failures
- Failed webhooks
- Database connection errors
- Third-party API failures

**Performance:**
- API response times
- Database query times
- Page load times
- Checkout funnel

**Business Metrics:**
- Orders per hour/day
- Revenue
- Cart abandonment rate
- Vendor signups

## Logging

### What to Log

**Critical:**
- Authentication attempts (success/failure)
- Payment transactions
- Admin actions (product deletions, user bans)
- System errors

**Useful:**
- API requests (with sanitized data)
- Webhook events
- Background job results

**Never Log:**
- Passwords
- API keys
- Full credit card numbers
- Session tokens

### Logging Setup

**Basic Console Logs** (default)
```typescript
console.log('[INFO]', 'Order created', { orderId })
console.error('[ERROR]', 'Payment failed', { error: error.message })
```

**Structured Logging** (better for production)
```bash
pnpm add pino pino-pretty
```

```typescript
import pino from 'pino'

const logger = pino({
  level: process.env.LOG_LEVEL || 'info'
})

logger.info({ orderId }, 'Order created')
logger.error({ error }, 'Payment failed')
```

### Log Aggregation

For production, send logs to a central service:

**BetterStack** (formerly Logtail)
- Free tier: 1 GB/month
- [betterstack.com/logs](https://betterstack.com/logs)

**Papertrail**
- Free tier: 50 MB/month
- [papertrail.com](https://papertrailapp.com/)

**Railway Logs**
- Included with Railway
- 7-day retention
- Access via Dashboard or CLI

```bash
# View logs via Railway CLI
railway logs --service backend-api
```

## Metrics to Track

### Infrastructure

**Railway Dashboard shows:**
- CPU usage
- Memory usage
- Network bandwidth
- Request count

**Set up alerts:**
- High CPU (> 80% for 5 minutes)
- High memory (> 90%)
- Error rate spike

### Application Metrics

Track in your APM or custom dashboard:

**API:**
- Requests per minute
- Average response time
- Error rate (%)
- 95th percentile response time

**Database:**
- Query count
- Slow queries (> 1 second)
- Connection pool usage
- Database size

**Business:**
- Orders per day
- Revenue per day
- Active users
- Cart conversion rate

## Alerting Strategy

### Critical Alerts (Wake Me Up)

- [ ] Site is down (> 2 minutes)
- [ ] Payment processing broken
- [ ] Database connection lost
- [ ] High error rate (> 10%)

**Action:** Immediate response required

### Warning Alerts (Can Wait an Hour)

- [ ] Slow API responses (> 2 seconds)
- [ ] High memory usage (> 80%)
- [ ] Failed background jobs
- [ ] Webhook delivery failures

**Action:** Investigate during business hours

### Info Alerts (Daily Summary)

- [ ] Daily metrics summary
- [ ] Dependency updates available
- [ ] SSL certificate expiring (30 days)

**Action:** Review and plan

## Dashboard Recommendations

### Simple Setup (Free)

1. **UptimeRobot** - Uptime monitoring
2. **Sentry** - Error tracking
3. **Railway Dashboard** - Infrastructure metrics
4. **Google Analytics** - User behavior (storefront)

### Advanced Setup

1. **Datadog or New Relic** - Full APM
2. **Better Stack** - Logs
3. **Custom Grafana Dashboard** - Business metrics
4. **Stripe Dashboard** - Payment analytics

## Railway-Specific Monitoring

### Built-in Metrics

Railway provides:
- CPU and memory graphs
- Request metrics
- Deployment history
- Logs (7-day retention)

**Access:**
1. Railway Dashboard
2. Select service
3. Click **Metrics** or **Logs**

### Set Up Alerts in Railway

1. Click on service
2. Go to **Settings** → **Notifications**
3. Add webhook URL (e.g., Slack, Discord)
4. Configure triggers:
   - Deployment failed
   - Service crashed
   - High resource usage

## Creating a Status Page

Let users know if something's wrong:

**Free Options:**

**Statuspage.io** (by Atlassian)
- Free tier available
- [statuspage.io](https://www.statuspage.io/)

**Better Stack Status Pages**
- Free with Better Stack account
- Connects to your monitors

**Self-hosted:**
```bash
# Use Upptime (GitHub Actions + Issues)
https://github.com/upptime/upptime
```

## Monitoring Checklist

### Setup (One-time)

- [ ] Health check endpoint created
- [ ] Uptime monitoring configured
- [ ] Error tracking (Sentry) installed
- [ ] Structured logging implemented
- [ ] Alerts configured (email/Slack)
- [ ] Status page created

### Regular Checks (Weekly)

- [ ] Review error rates
- [ ] Check slow queries
- [ ] Review failed payments
- [ ] Check disk/memory usage
- [ ] Review dependency alerts

### Monthly Reviews

- [ ] Analyze performance trends
- [ ] Review and clean up logs
- [ ] Update alert thresholds
- [ ] Check backup restores work
- [ ] Security audit

## Quick Reference

**Check all services:**
```bash
# Backend health
curl https://api.yourdomain.com/health

# Storefront (should return HTML)
curl -I https://shop.yourdomain.com

# Vendor panel (should return HTML)
curl -I https://admin.yourdomain.com
```

**Railway CLI monitoring:**
```bash
# Live logs
railway logs --service backend-api

# Service status
railway status

# Resource usage
railway metrics
```

**Emergency response:**
1. Check status page / uptime monitor
2. View Railway logs for errors
3. Check database connections
4. Review recent deployments
5. Rollback if needed

## Resources

- [Railway Observability Docs](https://docs.railway.app/guides/observability)
- [Sentry Documentation](https://docs.sentry.io/)
- [Better Stack Guides](https://betterstack.com/docs)
- [UptimeRobot Docs](https://uptimerobot.com/help/)

---

**Remember:** You can't fix what you can't see. Set up monitoring early, before you have problems.

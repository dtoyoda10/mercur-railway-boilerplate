# Resend Email Setup

Get transactional emails working for order confirmations, password resets, and vendor notifications.

## Why Resend?

- Simple API, no complicated SMTP configs
- Great deliverability
- Generous free tier (3,000 emails/month)
- React Email integration (write emails as React components)

## Step 1: Create a Resend Account

1. Sign up at [resend.com](https://resend.com)
2. Verify your email

## Step 2: Get Your API Key

1. Go to [API Keys](https://resend.com/api-keys)
2. Click **Create API Key**
3. Give it a name like "Medusa Backend Production"
4. Copy the key (starts with `re_`)

Add to your backend `.env`:

```env
RESEND_API_KEY=re_abc123xyz
```

## Step 3: Set Up Your Domain

By default, emails come from `onboarding@resend.dev` which works for testing but looks unprofessional.

### Add Your Domain

1. Go to [Domains](https://resend.com/domains)
2. Click **Add Domain**
3. Enter your domain (e.g., `yourdomain.com`)
4. Add the DNS records Resend provides:
   - MX records
   - TXT records (SPF, DKIM)

### Verify DNS

It usually takes 5-30 minutes for DNS to propagate. Resend will automatically verify once it detects the records.

### Set From Email

Once verified, update your `.env`:

```env
RESEND_FROM_EMAIL=noreply@yourdomain.com
# or
RESEND_FROM_EMAIL=orders@yourdomain.com
# or
RESEND_FROM_EMAIL=hello@yourdomain.com
```

Choose something that makes sense for your brand.

## Step 4: Test Sending Emails

### Quick Test via Dashboard

1. Go to Resend Dashboard → [Emails](https://resend.com/emails)
2. Click **Send Test Email**
3. Send to your personal email
4. Check it arrives (check spam if not in inbox)

### Test from Your App

Trigger an email event in your marketplace:

- Place a test order → Should send order confirmation
- Create a vendor account → Should send welcome email
- Request password reset → Should send reset link

**Check email logs:**
- Resend Dashboard → [Logs](https://resend.com/logs)
- See all sent emails, delivery status, and any errors

## Email Templates in This Boilerplate

The backend uses Resend to send:

- **Order confirmations** - When customer places order
- **Shipping updates** - When vendor ships products
- **Vendor invitations** - When admin invites new vendor
- **Password resets** - When user forgets password
- **Payout notifications** - When vendor receives payment

Templates are in `apps/backend/src/emails/` (if using React Email) or sent as HTML strings.

## Common Issues

### Emails not sending

**Fixes:**
1. Check `RESEND_API_KEY` is set correctly
2. Restart backend after adding env var
3. Check Resend logs for errors
4. Make sure you're within your monthly limit

### Emails going to spam

**Fixes:**
1. Verify your domain (don't use `onboarding@resend.dev` in production)
2. Add all DNS records (SPF, DKIM, DMARC)
3. Avoid spam trigger words in subject lines
4. Don't send too many emails too fast
5. Set up [DMARC](https://resend.com/docs/dashboard/domains/dmarc)

### Wrong "from" address

Make sure `RESEND_FROM_EMAIL` matches a verified domain. You can't send from `@gmail.com` or other providers.

## Going Live

- [ ] Add and verify your custom domain
- [ ] Update `RESEND_FROM_EMAIL` to your domain
- [ ] Test all email flows (order, shipping, password reset)
- [ ] Check emails don't go to spam
- [ ] Set up email monitoring/alerts
- [ ] Consider upgrading plan if sending >3k emails/month

## Pro Tips

### Use Different Emails for Different Types

```env
# Instead of one generic email
RESEND_FROM_EMAIL=noreply@yourdomain.com

# Consider:
# - orders@yourdomain.com (for order emails)
# - support@yourdomain.com (for user support)
# - vendors@yourdomain.com (for vendor comms)
```

You'd need to modify the email module to support this.

### Preview Emails Before Sending

If using React Email, you can preview templates locally:

```bash
cd apps/backend
npx email dev
```

Opens a preview server at http://localhost:3000

### Track Email Opens (Optional)

Resend supports tracking, but respect user privacy. Enable in Resend settings if needed.

## Resources

- [Resend Documentation](https://resend.com/docs)
- [React Email](https://react.email) (for building email templates)
- [Email Best Practices](https://resend.com/docs/best-practices)

## Need Help?

- [Resend Support](https://resend.com/support)
- [Resend Discord](https://resend.com/discord)

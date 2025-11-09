# Stripe Setup Guide

Setting up Stripe for payments and multi-vendor payouts in your marketplace.

## What You'll Get

- **Stripe Checkout** - Handle customer payments
- **Stripe Connect** - Split payments between marketplace and vendors
- **Webhooks** - Keep your orders in sync with Stripe events

## Prerequisites

- A Stripe account (sign up at [stripe.com](https://stripe.com))
- Your backend API running locally or deployed

## Step 1: Get Your API Keys

1. Go to the [Stripe Dashboard](https://dashboard.stripe.com/)
2. Click **Developers** → **API keys**
3. You'll see two keys:
   - **Publishable key** (starts with `pk_`) - safe to use in frontend
   - **Secret key** (starts with `sk_`) - keep this server-side only

### For Development

Use the **test mode** keys while developing:

```env
# Backend .env
STRIPE_SECRET_API_KEY=sk_test_51Abc...xyz

# Storefront .env
NEXT_PUBLIC_STRIPE_KEY=pk_test_51Abc...xyz
```

### For Production

Switch to **live mode** (toggle in Stripe Dashboard) and use those keys:

```env
# Backend .env
STRIPE_SECRET_API_KEY=sk_live_51Abc...xyz

# Storefront .env
NEXT_PUBLIC_STRIPE_KEY=pk_live_51Abc...xyz
```

## Step 2: Set Up Stripe Connect

This marketplace uses Stripe Connect to split payments between you (the platform) and your vendors.

### Enable Stripe Connect

1. In Stripe Dashboard, go to **Connect** → **Settings**
2. Choose **Platform or marketplace** as your business type
3. Fill out the branding info (logo, business name, etc.)

### Create Connected Accounts for Vendors

When a vendor signs up on your marketplace, you need to create a Stripe Connect account for them. This boilerplate handles it, but here's what happens:

1. Vendor registers on your platform
2. Your backend creates a Stripe Connect account via API
3. Vendor completes Stripe onboarding (banking details, tax info)
4. Once verified, they can receive payouts

**Testing Connect in development:**

Stripe provides test accounts you can use. When onboarding, use these test details:

- **Business name:** Any name
- **Country:** United States
- **Routing number:** 110000000
- **Account number:** 000123456789

## Step 3: Configure Webhooks

Webhooks keep your database in sync when payments succeed, fail, or get refunded.

### Create a Webhook Endpoint

1. Go to **Developers** → **Webhooks**
2. Click **Add endpoint**
3. Set the endpoint URL:
   - **Local (for testing):** You'll need a tool like [ngrok](https://ngrok.com/) to expose localhost
     ```bash
     ngrok http 9000
     # Use the HTTPS URL: https://abc123.ngrok.io/webhooks/stripe
     ```
   - **Production:** `https://your-api-domain.com/webhooks/stripe`

4. Select events to listen for:
   - `checkout.session.completed`
   - `payment_intent.succeeded`
   - `payment_intent.payment_failed`
   - `charge.refunded`
   - `account.updated` (for Connect)
   - `payout.paid` (for Connect)
   - `payout.failed` (for Connect)

5. Click **Add endpoint**

### Add Webhook Secret to Your .env

After creating the webhook:

1. Click on your webhook endpoint
2. Click **Reveal** under **Signing secret**
3. Copy the secret (starts with `whsec_`)
4. Add to your backend `.env`:

```env
STRIPE_CONNECTED_ACCOUNTS_WEBHOOK_SECRET=whsec_abc123xyz
```

## Step 4: Test the Integration

### Test Payments

Use Stripe's test card numbers:

**Successful payment:**
- Card: `4242 4242 4242 4242`
- Expiry: Any future date (e.g., 12/25)
- CVC: Any 3 digits
- ZIP: Any 5 digits

**Payment requires authentication (3D Secure):**
- Card: `4000 0025 0000 3155`

**Payment fails:**
- Card: `4000 0000 0000 9995`

More test cards: [Stripe Testing Docs](https://stripe.com/docs/testing)

### Test Connect Onboarding

1. Create a vendor account on your marketplace
2. The vendor should see a "Connect with Stripe" button
3. Click it and complete the onboarding flow
4. Check Stripe Dashboard → **Connect** → **Accounts** to see the connected account

## Common Issues

### "No API key provided"

**Fix:** Make sure `STRIPE_SECRET_API_KEY` is set in your backend `.env` and you've restarted the server.

### Webhooks not triggering

**Fixes:**
1. Check the webhook URL is correct and publicly accessible
2. Look at **Developers** → **Webhooks** → **Logs** in Stripe Dashboard
3. Make sure you're using the right webhook secret in your `.env`
4. For local dev, ensure ngrok is running and the URL matches

### Connected account can't receive payouts

**Checks:**
1. Has the vendor completed Stripe onboarding?
2. Is their account verified? (Check Connect → Accounts)
3. Did they add valid banking details?

## Going Live Checklist

Before switching to live mode:

- [ ] Complete your Stripe account activation
- [ ] Add business details (tax ID, address, etc.)
- [ ] Switch API keys to live mode (`sk_live_` and `pk_live_`)
- [ ] Update webhook URL to production domain
- [ ] Create new webhook endpoint for live mode
- [ ] Update `STRIPE_CONNECTED_ACCOUNTS_WEBHOOK_SECRET` with live webhook secret
- [ ] Test a small transaction
- [ ] Set up [Stripe Radar](https://stripe.com/radar) for fraud prevention

## Resources

- [Stripe Connect Documentation](https://stripe.com/docs/connect)
- [Stripe Payment Intents](https://stripe.com/docs/payments/payment-intents)
- [Stripe Webhooks Guide](https://stripe.com/docs/webhooks)
- [Stripe Testing](https://stripe.com/docs/testing)

## Need Help?

- Check [Stripe's support docs](https://support.stripe.com/)
- Ask in the [Stripe Discord](https://discord.gg/stripe)
- Open an issue in this repo

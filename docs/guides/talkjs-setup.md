# TalkJS Chat Setup

Add real-time messaging between customers and vendors.

## Why TalkJS?

- Pre-built chat UI (no need to build from scratch)
- Real-time messaging
- File sharing
- Works on web and mobile
- Free for small teams

## Step 1: Create TalkJS Account

1. Sign up at [talkjs.com](https://talkjs.com/)
2. Create a new app
3. Choose a name for your marketplace

## Step 2: Get Your App ID

1. Go to **Settings** → **API Credentials**
2. Copy your **App ID** (looks like `t1234567`)
3. Copy your **Secret Key** (starts with `sk_`)

### Add to Backend

```env
# apps/backend/.env
TALK_JS_APP_ID=t1234567
TALK_JS_SECRET_API_KEY=sk_test_...
```

### Add to Vendor Panel

```env
# apps/vendor-panel/.env
VITE_TALK_JS_APP_ID=t1234567
```

**Note:** The storefront doesn't need TalkJS unless you want customer-to-customer chat (which is rare in marketplaces).

## Step 3: Configure Chat Roles

TalkJS uses "roles" to control who can do what in chats.

### Create Roles

1. Go to **Settings** → **Roles**
2. Create two roles:
   - **vendor** - For sellers
   - **customer** - For buyers

### Set Permissions

**Vendor role:**
- ✅ Can send messages
- ✅ Can upload files
- ✅ Can see customer info
- ❌ Can't delete conversations

**Customer role:**
- ✅ Can send messages
- ✅ Can upload files
- ❌ Can't see vendor's private info

## Step 4: Test the Chat

### From Vendor Panel

1. Log in to your vendor panel
2. Go to **Messages** or **Orders**
3. Click on a customer order
4. You should see a chat widget
5. Send a test message

### From Storefront (If Enabled)

1. Log in as a customer
2. Visit a product page
3. Click "Contact Seller"
4. Send a message to the vendor

**If chat doesn't load:**
- Check browser console for errors
- Verify `TALK_JS_APP_ID` is correct
- Make sure TalkJS script is loading (check Network tab)
- Check you're logged in (TalkJS needs user auth)

## Step 5: Customize Chat UI

### Basic Customization (Theme)

1. Go to **Settings** → **Themes**
2. Choose a pre-built theme or create custom
3. Adjust colors to match your brand:
   - Primary color
   - Text colors
   - Background colors

### Advanced Customization (CSS)

You can inject custom CSS:

1. Go to **Settings** → **Themes** → **Custom CSS**
2. Add your styles:

```css
/* Example: Make chat bubble rounder */
.message-bubble {
  border-radius: 20px;
}

/* Change primary color */
.send-button {
  background-color: #your-brand-color;
}
```

## Step 6: Enable Notifications

Keep users engaged with email/push notifications when they get messages.

### Email Notifications

1. Go to **Settings** → **Notifications**
2. Enable **Email notifications**
3. Customize email template
4. Set notification triggers:
   - New message received
   - Unread messages reminder

### Push Notifications (Advanced)

For push notifications, you'll need to:

1. Set up a service worker
2. Configure Firebase Cloud Messaging (FCM)
3. Follow [TalkJS Push Notifications Guide](https://talkjs.com/docs/Features/Notifications/Push_Notifications/)

Most marketplaces just use email, which works fine.

## Common Issues

### Chat widget not appearing

**Fixes:**
1. Check `TALK_JS_APP_ID` is correct
2. Make sure you're calling `Talk.ready()` in your code
3. Check browser console for JS errors
4. Verify user is logged in (TalkJS needs authenticated users)

### Messages not sending

**Fixes:**
1. Check user has the right role (vendor/customer)
2. Verify conversation was created properly
3. Check TalkJS Dashboard → **Logs** for errors
4. Make sure you're not rate limited (free tier has limits)

### "User not found" error

This means the user wasn't properly synced to TalkJS.

**Fix:** Ensure you're creating TalkJS users when they sign up:

```typescript
// This should be in your backend user creation flow
await talkjs.users.create({
  id: user.id,
  name: user.name,
  email: user.email,
  role: user.is_vendor ? 'vendor' : 'customer'
})
```

## Going Live

- [ ] Switch from test to live App ID
- [ ] Update `TALK_JS_APP_ID` and `TALK_JS_SECRET_API_KEY`
- [ ] Customize chat theme to match your brand
- [ ] Enable email notifications
- [ ] Test chat flows (customer to vendor, vendor to customer)
- [ ] Set up moderation rules (if needed)
- [ ] Consider upgrading plan for more users

## Advanced Features

### File Sharing

TalkJS supports file sharing out of the box. Users can:

- Upload images
- Upload documents (PDF, DOCX, etc.)
- Share order details

Configure max file size in TalkJS Settings.

### Message Moderation

For marketplaces, you might want to monitor chats:

1. Go to **Settings** → **Moderation**
2. Enable auto-flagging of inappropriate content
3. Set up keyword filters
4. Review flagged conversations in Dashboard

### Conversation Metadata

You can attach order info to chats:

```typescript
// Example: Link chat to an order
await talkjs.conversations.create({
  id: `order_${orderId}`,
  participants: [vendorId, customerId],
  subject: `Order #${orderNumber}`,
  custom: {
    orderId: orderId,
    orderTotal: total,
    orderStatus: status
  }
})
```

This lets vendors see order details right in the chat.

## Free Tier Limits

TalkJS free tier includes:

- 500 monthly active users (MAU)
- 100,000 messages/month
- 10 GB file storage

**MAU = unique users who send/receive at least one message**

If you hit limits, upgrade to a paid plan.

## Resources

- [TalkJS Documentation](https://talkjs.com/docs/)
- [TalkJS React Guide](https://talkjs.com/docs/Getting_Started/Frameworks/React/)
- [TalkJS API Reference](https://talkjs.com/docs/Reference/REST_API/)
- [Customization Guide](https://talkjs.com/docs/Features/Themes/)

## Need Help?

- [TalkJS Support](https://talkjs.com/support/)
- Email: support@talkjs.com

# Complete Instagram Automation Setup Guide

This guide will walk you through setting up the Instagram Comment-to-DM automation tool from scratch.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Instagram Business Account Setup](#instagram-business-account-setup)
3. [Facebook Developer App Setup](#facebook-developer-app-setup)
4. [Getting Your Access Token](#getting-your-access-token)
5. [Finding Your Account IDs](#finding-your-account-ids)
6. [Webhook Configuration](#webhook-configuration)
7. [Application Deployment](#application-deployment)
8. [Testing Your Setup](#testing-your-setup)

---

## Prerequisites

Before you begin, ensure you have:

- [ ] An Instagram account
- [ ] A Facebook account
- [ ] A Facebook Page
- [ ] Admin access to both the Instagram account and Facebook Page
- [ ] Basic understanding of APIs and webhooks

---

## Instagram Business Account Setup

### 1. Convert to Professional Account

1. Open the Instagram app on your phone
2. Go to **Settings** → **Account**
3. Select **Switch to Professional Account**
4. Choose **Business** or **Creator**
5. Select your category (e.g., "Digital Creator", "E-commerce")
6. Complete the setup process

### 2. Connect to Facebook Page

1. In Instagram settings, go to **Account** → **Linked Accounts**
2. Select **Facebook**
3. Log in and connect your Facebook Page
4. Make sure you're an admin of the Facebook Page

---

## Facebook Developer App Setup

### 1. Create Developer Account

1. Go to [developers.facebook.com](https://developers.facebook.com)
2. Click **Get Started** (top right)
3. Complete the registration process
4. Verify your email if needed

### 2. Create a New App

1. Click **My Apps** → **Create App**
2. Select **Business** as the app type
3. Fill in the details:
   - **App Name**: "Instagram Automation Tool" (or your preferred name)
   - **App Contact Email**: Your email
   - **Business Account**: Select or create one
4. Click **Create App**
5. Complete the security check

### 3. Add Instagram Product

1. In your app dashboard, scroll to **Add Products to Your App**
2. Find **Instagram** and click **Set Up**
3. The Instagram Graph API is now added to your app

### 4. Configure App Settings

1. Go to **Settings** → **Basic**
2. Note down your **App ID** and **App Secret** (click Show to reveal)
3. Add **App Domains**: Your deployment domain (e.g., `yourapp.com`)
4. Add **Privacy Policy URL**: Link to your privacy policy
5. Save changes

---

## Getting Your Access Token

### Option 1: Graph API Explorer (Recommended)

1. Go to [Graph API Explorer](https://developers.facebook.com/tools/explorer)
2. Select your app from the **Meta App** dropdown
3. Select your Facebook Page from the **User or Page** dropdown
4. Click **Permissions** and add these permissions:
   - `instagram_basic`
   - `instagram_manage_comments`
   - `instagram_manage_messages`
   - `pages_show_list`
   - `pages_read_engagement`
5. Click **Generate Access Token**
6. Authorize all permissions when prompted
7. **Copy the token** - this is a SHORT-LIVED token (1 hour)

### Option 2: Convert to Long-Lived Token (60 days)

Use this curl command to exchange for a long-lived token:

```bash
curl -X GET "https://graph.facebook.com/v21.0/oauth/access_token?\
grant_type=fb_exchange_token&\
client_id=YOUR_APP_ID&\
client_secret=YOUR_APP_SECRET&\
fb_exchange_token=SHORT_LIVED_TOKEN"
```

Replace:
- `YOUR_APP_ID`: Your Facebook App ID
- `YOUR_APP_SECRET`: Your Facebook App Secret
- `SHORT_LIVED_TOKEN`: The token from Graph API Explorer

**Response:**
```json
{
  "access_token": "EAALongLivedTokenHere...",
  "token_type": "bearer",
  "expires_in": 5184000
}
```

Save this `access_token` - it's valid for 60 days.

### Token Refresh Reminder

⚠️ **Important**: Long-lived tokens expire after 60 days. Set a calendar reminder to refresh your token every 50 days.

---

## Finding Your Account IDs

### 1. Get Facebook Page ID

**Method 1: Graph API Explorer**

```
GET /me/accounts
```

Look for your page in the response and note the `id`.

**Method 2: Page Settings**

1. Go to your Facebook Page
2. Click **Settings**
3. Click **Page Info**
4. Your Page ID is listed

### 2. Get Instagram Business Account ID

Using Graph API Explorer:

```
GET /PAGE_ID?fields=instagram_business_account
```

Replace `PAGE_ID` with your Facebook Page ID.

**Response:**
```json
{
  "instagram_business_account": {
    "id": "17841405793187218"
  },
  "id": "123456789012345"
}
```

Save the `instagram_business_account.id` value.

### 3. Get Instagram Post ID

To find a specific post's ID:

```
GET /INSTAGRAM_ACCOUNT_ID/media?fields=id,caption,media_type,permalink,timestamp
```

**Response:**
```json
{
  "data": [
    {
      "id": "17895695668004550",
      "caption": "Check out our new product!",
      "media_type": "IMAGE",
      "permalink": "https://www.instagram.com/p/ABC123/",
      "timestamp": "2024-01-15T12:00:00+0000"
    }
  ]
}
```

The `id` field is your post ID.

---

## Webhook Configuration

### 1. Deploy Your Application First

Your application must be publicly accessible before setting up webhooks. Deploy to:

- **Vercel**: `vercel deploy --prod`
- **Railway**: `railway up`
- **Render**: Connect repo and deploy

Note your deployment URL (e.g., `https://yourapp.vercel.app`)

### 2. Configure Webhook in Facebook App

1. Go to your app dashboard
2. Navigate to **Products** → **Webhooks**
3. Click **Configure** next to Instagram
4. Click **Edit Subscription**

**Enter webhook details:**
- **Callback URL**: `https://yourapp.vercel.app/api/webhook/instagram`
- **Verify Token**: Create a random string (e.g., `my_super_secret_verify_token_12345`)
  - ⚠️ Save this token - you'll need it in your environment variables

5. Click **Verify and Save**

### 3. Subscribe to Events

1. After verification, click **Add Field Subscription**
2. Select **comments**
3. Click **Save**

### 4. Test Webhook Locally (Optional)

For local development, use ngrok:

```bash
# Start your Next.js app
npm run dev

# In another terminal, start ngrok
npx ngrok http 3000
```

Use the ngrok HTTPS URL as your webhook callback URL.

---

## Application Deployment

### 1. Set Environment Variables

In your deployment platform, set these environment variables:

```env
DATABASE_URL=postgresql://user:password@host:5432/database
INSTAGRAM_ACCESS_TOKEN=your_long_lived_token_here
INSTAGRAM_BUSINESS_ACCOUNT_ID=17841405793187218
FACEBOOK_PAGE_ID=123456789012345
FACEBOOK_APP_SECRET=your_app_secret_here
WEBHOOK_VERIFY_TOKEN=my_super_secret_verify_token_12345
```

### 2. Deploy the Application

**Vercel:**
```bash
npm run build
vercel deploy --prod
```

**Railway:**
```bash
railway up
```

**Render:**
1. Connect your GitHub repository
2. Render auto-detects Next.js
3. Add environment variables
4. Deploy

### 3. Initialize Database

After deployment:

```bash
# SSH into your deployment or run locally against production DB
npx drizzle-kit push
```

This creates the necessary database tables.

---

## Testing Your Setup

### 1. Configure the Dashboard

1. Visit your deployed application
2. Go to the **Settings** tab
3. Enter all your credentials:
   - Instagram Access Token
   - Instagram Business Account ID
   - Facebook Page ID
   - Webhook Verify Token
   - Facebook App Secret
4. Click **Save Configuration**

### 2. Create a Test Campaign

1. Go to the **Campaigns** tab
2. Click **+ New Campaign**
3. Enter a Post ID from your Instagram account
4. Wait for the post preview to load
5. Enter test keywords: `test, bot, automation`
6. Enter a comment reply: `Thanks for commenting! I'll send you a DM.`
7. Enter a DM message: `Hi! Here's the information you requested.`
8. Make sure "Campaign is active" is checked
9. Click **Create Campaign**

### 3. Test the Automation

1. Go to your Instagram post (use a test account, not your main account)
2. Leave a comment with one of your keywords: `test`
3. Within seconds, you should see:
   - ✅ An automated reply to your comment
   - ✅ A DM sent to your test account (if you've messaged the business before)

### 4. Check Logs

Monitor your application logs for:

```
Webhook received: ...
Comment matches campaign ...
Replying to comment ...
Comment reply sent successfully
Sending DM to user ...
DM sent successfully
Successfully processed comment ...
```

### 5. Verify in Dashboard

Check that the comment isn't processed twice (deduplication works).

---

## Troubleshooting Common Issues

### Issue: Webhook Verification Failed

**Solutions:**
- Ensure `WEBHOOK_VERIFY_TOKEN` in your app matches the token in Facebook settings
- Check that your webhook endpoint is publicly accessible
- Verify the URL is exactly `https://yourdomain.com/api/webhook/instagram`

### Issue: DM Not Sending

**Solutions:**
- The user must have messaged your business first (Instagram restriction)
- Request `instagram_manage_messages` permission from Facebook
- Check that your access token has the correct permissions

### Issue: Post Preview Not Loading

**Solutions:**
- Verify the Post ID is correct
- Ensure the post belongs to your Instagram Business Account
- Check that your access token is valid and not expired

### Issue: Webhook Not Receiving Events

**Solutions:**
- Verify webhook subscription includes `comments` field
- Check webhook signature verification is working
- Ensure your server is publicly accessible (not localhost)
- Review webhook logs in Facebook App dashboard

### Issue: Access Token Expired

**Solutions:**
- Generate a new long-lived token (they expire after 60 days)
- Update the token in your dashboard Settings
- Consider setting up automated token refresh

---

## Security Best Practices

1. **Never commit secrets to Git**
   - Use environment variables
   - Add `.env` to `.gitignore`

2. **Rotate tokens regularly**
   - Set calendar reminders
   - Use strong, random verify tokens

3. **Monitor API usage**
   - Watch for unusual activity
   - Check Facebook App dashboard analytics

4. **Validate all webhook requests**
   - The app verifies HMAC signatures automatically
   - Never disable signature verification

5. **Use HTTPS only**
   - Never expose webhooks over HTTP
   - Use reputable hosting providers

---

## Next Steps

Now that your setup is complete:

1. ✅ Create campaigns for your real Instagram posts
2. ✅ Monitor performance and engagement
3. ✅ Adjust keywords and messages based on results
4. ✅ Set up token refresh reminders
5. ✅ Apply for additional Instagram permissions if needed

---

## Support

If you encounter issues:

1. Check the [README.md](README.md) for API documentation
2. Review Facebook's [Instagram Graph API documentation](https://developers.facebook.com/docs/instagram-api)
3. Check webhook logs in Facebook App dashboard
4. Open an issue on GitHub

---

**Happy Automating! 🚀**

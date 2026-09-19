# Troubleshooting Guide

Common issues and their solutions when setting up and running the Instagram Automation Tool.

## Table of Contents

- [Setup Issues](#setup-issues)
- [Webhook Issues](#webhook-issues)
- [API Issues](#api-issues)
- [Campaign Issues](#campaign-issues)
- [Database Issues](#database-issues)
- [Deployment Issues](#deployment-issues)

---

## Setup Issues

### Cannot Create Facebook Developer App

**Symptoms:**
- "You don't have permission to create apps" error
- Can't access developer.facebook.com

**Solutions:**
1. Verify your Facebook account is in good standing
2. Complete Facebook account verification if requested
3. Try using a different Facebook account
4. Check if your account has been restricted

---

### Instagram Account Not Showing Up

**Symptoms:**
- Can't find Instagram account in settings
- "No Instagram account found" error

**Solutions:**
1. Verify account is converted to Business/Creator
2. Ensure Instagram is connected to a Facebook Page
3. Check you're an admin of both the Instagram account and Facebook Page
4. Try disconnecting and reconnecting Instagram to Facebook Page
5. Wait 24-48 hours after conversion (caching delay)

---

### Cannot Generate Access Token

**Symptoms:**
- "Failed to generate token" error
- Permissions not showing in Graph API Explorer

**Solutions:**
1. Make sure you selected your app in Graph API Explorer
2. Verify app has Instagram product added
3. Check app is not in Development mode (or add test users)
4. Request permissions via App Review if needed
5. Try logging out and back in to Facebook

---

## Webhook Issues

### Webhook Verification Failed

**Symptoms:**
```
Error: Webhook verification failed
403 Forbidden
```

**Solutions:**

1. **Check verify token matches**
   ```env
   # In .env file
   WEBHOOK_VERIFY_TOKEN=my_secret_token
   
   # Must match token in Facebook App webhook settings
   ```

2. **Verify URL is correct**
   ```
   https://yourdomain.com/api/webhook/instagram
   ```
   Not: `/webhook` or `/api/webhooks` (note the 's')

3. **Check server is running**
   ```bash
   curl https://yourdomain.com/api/health
   # Should return: {"status":"ok"}
   ```

4. **Ensure HTTPS only**
   - Facebook requires HTTPS for webhooks
   - Use ngrok for local testing:
     ```bash
     npx ngrok http 3000
     ```

5. **Check environment variables are loaded**
   ```bash
   # In your deployment logs, verify:
   echo $WEBHOOK_VERIFY_TOKEN
   ```

---

### Webhook Not Receiving Events

**Symptoms:**
- Comments on Instagram don't trigger automation
- No webhook events in logs

**Solutions:**

1. **Verify webhook subscription**
   - Go to Facebook App → Webhooks
   - Check "comments" field is subscribed
   - Look for green checkmark

2. **Check webhook is active**
   - In Facebook App webhook settings
   - Ensure subscription is "Active" not "Inactive"

3. **Test webhook manually**
   ```bash
   # In Facebook App → Webhooks → Test
   # Send a test event
   ```

4. **Check application logs**
   ```bash
   # Should see:
   Webhook received: {...}
   ```

5. **Verify post is tracked**
   - Campaign must exist for that post ID
   - Campaign must be marked as "Active"

6. **Check webhook signature verification**
   - Ensure `FACEBOOK_APP_SECRET` is correct
   - If signature fails, requests are rejected

---

### Webhook Signature Verification Fails

**Symptoms:**
```
Error: Invalid webhook signature
403 Forbidden
```

**Solutions:**

1. **Verify app secret is correct**
   ```bash
   # Get from Facebook App → Settings → Basic → App Secret
   # Must match:
   FACEBOOK_APP_SECRET=your_app_secret_here
   ```

2. **Check environment variable is loaded**
   ```bash
   # In deployment logs:
   console.log(process.env.FACEBOOK_APP_SECRET)
   ```

3. **Ensure raw body is used**
   - Next.js handles this automatically
   - Don't parse body before signature verification

4. **Temporarily disable verification (testing only)**
   - Comment out signature verification in webhook route
   - ⚠️ **Never do this in production!**

---

## API Issues

### Access Token Expired

**Symptoms:**
```
Instagram API error: Error validating access token
OAuthException
```

**Solutions:**

1. **Generate new long-lived token**
   ```bash
   curl -X GET "https://graph.facebook.com/v21.0/oauth/access_token?\
   grant_type=fb_exchange_token&\
   client_id=YOUR_APP_ID&\
   client_secret=YOUR_APP_SECRET&\
   fb_exchange_token=SHORT_LIVED_TOKEN"
   ```

2. **Update token in dashboard**
   - Go to Settings tab
   - Paste new access token
   - Click Save

3. **Set up token refresh reminder**
   - Tokens expire after 60 days
   - Set calendar reminder for day 50

---

### Cannot Send DMs

**Symptoms:**
```
Error: Cannot send DM: User must message the business first
Error code: 10900
```

**Solutions:**

This is an Instagram restriction, not a bug!

**Option 1: User initiates conversation**
1. Ask followers to DM you first
2. Use Instagram Stories with "Send Message" sticker
3. Put "DM me for info" in your bio

**Option 2: Apply for messaging permissions**
1. Go to Facebook App → App Review
2. Request `instagram_manage_messages` permission
3. Provide detailed use case explanation
4. Wait for Facebook approval (1-7 days)

**Option 3: Accept DM limitation**
- Comment replies will still work
- DM will work for users who've messaged before
- This is acceptable for most use cases

---

### Rate Limit Errors

**Symptoms:**
```
Instagram API error: Rate limit exceeded
Error code: 32
```

**Solutions:**

1. **Wait for rate limit reset**
   - Instagram allows 200 calls per hour
   - Limit resets every hour

2. **Reduce API calls**
   - Don't refresh post preview repeatedly
   - Limit number of active campaigns on high-traffic posts

3. **Use retry logic**
   - App has built-in exponential backoff
   - Check logs for retry attempts

4. **Monitor API usage**
   - Check Facebook App dashboard analytics
   - Track API call patterns

---

### Cannot Fetch Post Details

**Symptoms:**
```
Failed to fetch post details
Error: Invalid media ID
```

**Solutions:**

1. **Verify post ID is correct**
   ```bash
   # Use Graph API Explorer:
   GET /YOUR_IG_ACCOUNT_ID/media?fields=id,caption
   ```

2. **Check post belongs to your account**
   - Can't fetch posts from other accounts
   - Must be your Instagram Business Account

3. **Verify access token has permissions**
   - Needs `instagram_basic` permission

4. **Check post type**
   - Works for IMAGE, VIDEO, CAROUSEL_ALBUM
   - May not work for IGTV or Reels (depends on API version)

---

## Campaign Issues

### Campaign Not Triggering

**Symptoms:**
- Comment posted with keyword
- No reply or DM sent

**Solutions:**

1. **Check campaign is active**
   - Green badge = Active
   - Gray badge = Inactive
   - Click play button to activate

2. **Verify keyword match**
   - Matching is case-insensitive
   - Partial match: "free" matches "free download"
   - Check for typos in keywords

3. **Verify post ID matches**
   ```bash
   # Post ID in campaign must match exactly
   Campaign Post ID: 17895695668004550
   Comment's Post ID: 17895695668004550  ✓
   ```

4. **Check comment wasn't already processed**
   - Each comment ID is only processed once
   - Check database `processed_comments` table

5. **Review application logs**
   ```bash
   # Look for:
   Comment matches campaign X
   Replying to comment...
   DM sent successfully
   
   # Or errors:
   No active campaigns for post
   Failed to send DM
   ```

---

### Campaign Not Showing in List

**Symptoms:**
- Created campaign but not visible
- Empty campaigns list

**Solutions:**

1. **Refresh the page**
   - Click browser refresh
   - Or navigate away and back to Campaigns tab

2. **Check database**
   ```bash
   # Connect to database and query:
   SELECT * FROM campaigns;
   ```

3. **Verify API call succeeded**
   - Open browser DevTools → Network tab
   - Check POST /api/campaigns returned 201

4. **Check for JavaScript errors**
   - Open browser Console
   - Look for errors loading campaigns

---

## Database Issues

### Cannot Connect to Database

**Symptoms:**
```
Error: Connection refused
ECONNREFUSED 127.0.0.1:5432
```

**Solutions:**

1. **Verify PostgreSQL is running**
   ```bash
   # Local:
   sudo systemctl status postgresql
   
   # Docker:
   docker ps | grep postgres
   ```

2. **Check connection string**
   ```env
   # Correct format:
   DATABASE_URL=postgresql://user:password@host:5432/database
   ```

3. **Verify credentials**
   - Username correct?
   - Password correct?
   - Database exists?

4. **Check network/firewall**
   - Can you ping the database host?
   - Is port 5432 open?

---

### Schema Push Failed

**Symptoms:**
```
Error: relation "campaigns" already exists
```

**Solutions:**

1. **Drop existing tables (development only)**
   ```bash
   # Connect to database:
   psql $DATABASE_URL
   
   # Drop tables:
   DROP TABLE IF EXISTS processed_comments CASCADE;
   DROP TABLE IF EXISTS campaigns CASCADE;
   DROP TABLE IF EXISTS config CASCADE;
   
   # Run push again:
   npx drizzle-kit push
   ```

2. **Use migrations instead**
   ```bash
   npx drizzle-kit generate
   npx drizzle-kit migrate
   ```

---

## Deployment Issues

### Build Failed

**Symptoms:**
```
Error: Build failed
Type errors found
```

**Solutions:**

1. **Run type check locally**
   ```bash
   npm run typecheck
   ```

2. **Fix TypeScript errors**
   - Check error messages
   - Ensure all imports are correct

3. **Verify all dependencies installed**
   ```bash
   rm -rf node_modules package-lock.json
   npm install
   ```

---

### Environment Variables Not Working

**Symptoms:**
```
Error: Instagram credentials not configured
process.env.VARIABLE is undefined
```

**Solutions:**

1. **Verify variables are set in deployment platform**
   - Vercel: Project Settings → Environment Variables
   - Railway: Project → Variables
   - Render: Environment → Environment Variables

2. **Check variable names match exactly**
   ```env
   # Correct:
   INSTAGRAM_ACCESS_TOKEN=xxx
   
   # Wrong:
   instagram_access_token=xxx
   IG_ACCESS_TOKEN=xxx
   ```

3. **Redeploy after adding variables**
   - Most platforms require redeploy
   - Vercel: Trigger new deployment
   - Railway: Redeploy from dashboard

4. **Check .env file locally**
   ```bash
   cat .env
   # Verify variables exist
   ```

---

### Health Check Failing

**Symptoms:**
```
Health check timeout
Deployment failed
```

**Solutions:**

1. **Verify /api/health endpoint works**
   ```bash
   curl https://yourdomain.com/api/health
   # Should return: {"status":"ok"}
   ```

2. **Check server is listening on correct port**
   ```javascript
   // Should use PORT from environment
   const port = process.env.PORT || 3000
   ```

3. **Increase health check timeout**
   - Railway: Update railway.toml
   - Render: Update render.yaml
   ```toml
   healthcheckTimeout = 300  # 5 minutes
   ```

4. **Check database connection**
   - Health check might fail if DB is unreachable
   - Verify DATABASE_URL is correct

---

### Docker Build Failed

**Symptoms:**
```
Error: Docker build failed
Cannot find module 'next'
```

**Solutions:**

1. **Verify Dockerfile is correct**
   - Check all COPY commands
   - Ensure standalone output is enabled

2. **Enable standalone in next.config.ts**
   ```typescript
   const nextConfig: NextConfig = {
     output: 'standalone',
   };
   ```

3. **Build locally to test**
   ```bash
   docker build -t instagram-automation .
   docker run -p 3000:3000 instagram-automation
   ```

4. **Check .dockerignore**
   - Ensure node_modules is ignored
   - Don't ignore package.json

---

## Still Having Issues?

### Debug Checklist

- [ ] Check application logs thoroughly
- [ ] Test each component individually
- [ ] Verify all environment variables
- [ ] Check Facebook App dashboard for errors
- [ ] Review webhook delivery logs
- [ ] Test with minimal campaign first
- [ ] Use ngrok for local webhook testing

### Get Help

1. **Review documentation**
   - [README.md](README.md)
   - [SETUP_GUIDE.md](SETUP_GUIDE.md)
   - [API_DOCUMENTATION.md](API_DOCUMENTATION.md)

2. **Check Instagram API docs**
   - [Instagram Graph API](https://developers.facebook.com/docs/instagram-api)
   - [Webhooks Reference](https://developers.facebook.com/docs/graph-api/webhooks)

3. **Open an issue**
   - GitHub Issues with:
     - Error message
     - Steps to reproduce
     - Environment details
     - Relevant logs

---

**Remember**: Most issues are due to incorrect configuration or expired tokens. Double-check credentials first!

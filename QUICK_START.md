# Quick Start Guide

Get your Instagram automation running in 10 minutes!

## 🎯 What This Tool Does

Automatically replies to Instagram comments AND sends DMs when someone comments with specific keywords on your posts.

**Example:**
- Someone comments "FREE" on your post
- Bot replies: "Check your DMs! 💬"
- Bot sends DM: "Here's your free guide: [link]"

---

## ⚡ 5-Minute Setup Checklist

### Step 1: Prerequisites (2 min)
- [ ] Instagram Business or Creator account
- [ ] Facebook Page connected to Instagram
- [ ] Facebook Developer account created

### Step 2: Get Your Access Token (3 min)
1. Go to [Graph API Explorer](https://developers.facebook.com/tools/explorer)
2. Select your app
3. Add permissions: `instagram_manage_comments`, `instagram_manage_messages`
4. Click "Generate Access Token"
5. Copy the token

### Step 3: Configure the App (1 min)
1. Open the app dashboard
2. Go to Settings tab
3. Paste your access token
4. Enter your Instagram Business Account ID
5. Save

### Step 4: Create Your First Campaign (2 min)
1. Go to Campaigns tab
2. Click "+ New Campaign"
3. Enter your Instagram post ID
4. Add keywords: `free, download, link`
5. Write your comment reply
6. Write your DM message
7. Click "Create Campaign"

### Step 5: Test It! (2 min)
1. Comment on your post with a keyword
2. Watch the magic happen! ✨

---

## 🔑 Where to Find Important IDs

### Instagram Post ID

**Option 1: From post URL**
```
https://www.instagram.com/p/ABC123DEF/
                              ^^^^^^^^
Post shortcode - NOT the ID you need
```

**Option 2: Use Graph API Explorer**
```
GET /YOUR_IG_ACCOUNT_ID/media?fields=id,caption
```

Copy the `id` field from the response.

### Instagram Business Account ID

```
GET /YOUR_PAGE_ID?fields=instagram_business_account
```

The response contains your Instagram Business Account ID.

### Facebook Page ID

1. Go to your Facebook Page
2. Settings → Page Info
3. Copy the Page ID

---

## 🚀 Quick Deploy

### Deploy to Vercel (Recommended)

```bash
# 1. Clone the repo
git clone <your-repo>
cd instagram-automation

# 2. Install dependencies
npm install

# 3. Set up database (if deploying to Vercel with PostgreSQL addon)
npx drizzle-kit push

# 4. Deploy
vercel deploy --prod
```

Add environment variables in Vercel dashboard:
- `DATABASE_URL`
- `INSTAGRAM_ACCESS_TOKEN`
- `INSTAGRAM_BUSINESS_ACCOUNT_ID`
- `FACEBOOK_APP_SECRET`
- `WEBHOOK_VERIFY_TOKEN`

### Deploy to Railway

```bash
railway up
```

Railway will auto-detect the Dockerfile and deploy.

---

## 🎨 Campaign Ideas

### E-commerce Store
- **Keyword**: `price, buy, shop`
- **Reply**: "Thanks for your interest! Check your DMs 💬"
- **DM**: "Here's the link to purchase: [your-shop-link]"

### Content Creator
- **Keyword**: `template, download, free`
- **Reply**: "I just sent you the download link! 📥"
- **DM**: "Here's your free template: [download-link]"

### Service Business
- **Keyword**: `info, booking, appointment`
- **Reply**: "I'll send you the details now! 📩"
- **DM**: "Book your appointment here: [booking-link]"

### Lead Magnet
- **Keyword**: `guide, ebook, pdf`
- **Reply**: "Check your DMs for the guide! 📚"
- **DM**: "Download your free guide: [link]"

---

## ⚠️ Common Gotchas

### DMs Not Working?

**Why**: Instagram requires users to message your business first before you can DM them.

**Solutions**:
1. Ask followers to send you a DM first
2. Apply for `instagram_manage_messages` permission from Facebook
3. Use stories with "Send Message" sticker to get users to initiate

### Webhook Not Triggering?

**Checklist**:
- ✅ Webhook URL is publicly accessible (not localhost)
- ✅ Subscribed to `comments` field in Facebook App
- ✅ Verify token matches in both places
- ✅ Campaign is marked as "Active"

### Access Token Expired?

**Solution**: Tokens expire after 60 days. Generate a new one and update in Settings.

---

## 💡 Pro Tips

1. **Test with a secondary account** - Don't spam your main followers during testing

2. **Use specific keywords** - "free" might trigger too often, "FREEGUIDE2024" is more specific

3. **Personalize messages** - Use emojis and friendly language in your replies

4. **Monitor performance** - Check which keywords get the most engagement

5. **Refresh tokens early** - Don't wait until day 60 to refresh your access token

6. **Multiple campaigns** - Create different campaigns for different posts/offers

7. **A/B test messages** - Try different reply messages to see what converts best

---

## 📊 Dashboard Overview

### Settings Tab
Configure your Instagram API credentials here. You only need to do this once.

### Campaigns Tab
- **Green badge** = Active campaign
- **Gray badge** = Inactive campaign
- **Play button** = Activate/Deactivate
- **Edit button** = Modify keywords/messages
- **Delete button** = Remove campaign

---

## 🆘 Need Help?

### Check the Logs

In your deployment platform, check application logs for errors:
```
Webhook received
Comment matches campaign
Replying to comment
DM sent successfully
```

### Webhook Logs

Facebook App Dashboard → Webhooks → View Events

See all webhook deliveries and responses.

### Test Your Token

Go to [Graph API Explorer](https://developers.facebook.com/tools/explorer) and try:
```
GET /me?fields=id,name
```

If this fails, your token is invalid.

---

## 📚 Resources

- **Full Setup Guide**: See [SETUP_GUIDE.md](SETUP_GUIDE.md)
- **API Documentation**: See [README.md](README.md)
- **Instagram Graph API Docs**: [developers.facebook.com/docs/instagram-api](https://developers.facebook.com/docs/instagram-api)

---

## ✅ Success Checklist

Before going live:

- [ ] Instagram account is Business/Creator
- [ ] Facebook App created with Instagram product
- [ ] Access token generated and saved
- [ ] Webhook configured and verified
- [ ] Database connected
- [ ] Test campaign created
- [ ] Test comment processed successfully
- [ ] Both comment reply and DM sent

---

**You're ready to automate! 🎉**

Start with one post, test thoroughly, then scale to more campaigns.

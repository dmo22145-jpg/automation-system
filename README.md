# 📸 Instagram Comment-to-DM Automation Tool

A full-stack web application that automates Instagram comment replies and direct messages based on keywords. When a user comments a specific keyword on a tracked Instagram post, the tool automatically replies to the comment AND sends the commenter a private DM.

## 🚀 Features

- **Automated Comment Replies**: Automatically reply to comments containing specific keywords
- **Direct Message Automation**: Send private DMs to commenters who trigger keywords
- **Campaign Management**: Create multiple campaigns for different posts with different keywords
- **Post Preview**: Auto-fetch Instagram post thumbnails and captions
- **Deduplication**: Prevents processing the same comment multiple times
- **Web Dashboard**: Clean, modern UI to manage campaigns and settings
- **Webhook Support**: Real-time processing via Facebook webhooks
- **Secure**: Webhook signature verification and credential encryption

## 🛠️ Tech Stack

- **Frontend**: Next.js 16, React 19, TypeScript, Tailwind CSS
- **Backend**: Next.js API Routes (serverless functions)
- **Database**: PostgreSQL with Drizzle ORM
- **Instagram API**: Official Instagram Graph API
- **Deployment**: Cloud-ready (Vercel, Railway, Render compatible)

## 📋 Prerequisites

Before setting up this application, you need:

1. An Instagram Business or Creator account
2. A Facebook Page connected to your Instagram account
3. A Facebook Developer account
4. Node.js 18+ installed
5. PostgreSQL database

## 🔧 Instagram API Setup (Complete Guide)

### Step 1: Convert Instagram Account

1. Go to your Instagram profile settings
2. Switch to Professional Account
3. Choose Business or Creator category
4. Complete the setup

### Step 2: Create Facebook Developer App

1. Visit [developers.facebook.com](https://developers.facebook.com)
2. Click "My Apps" → "Create App"
3. Select "Business" as the app type
4. Fill in app details and create

### Step 3: Add Instagram Graph API Product

1. In your app dashboard, click "Add Product"
2. Find "Instagram" and click "Set Up"
3. The Instagram Graph API is now added to your app

### Step 4: Get Required Permissions

You need the following permissions:
- `instagram_basic`
- `instagram_manage_comments`
- `instagram_manage_messages`
- `pages_show_list`
- `pages_read_engagement`

**To request permissions:**
1. Go to App Review → Permissions and Features
2. Request the permissions listed above
3. Provide use case details (comment automation, customer engagement)
4. Wait for Facebook approval (can take a few days)

### Step 5: Generate Long-Lived Access Token

**Using Graph API Explorer:**

1. Go to [developers.facebook.com/tools/explorer](https://developers.facebook.com/tools/explorer)
2. Select your app from the dropdown
3. Click "Generate Access Token"
4. Grant all requested permissions
5. This gives you a short-lived token (1 hour)

**Convert to Long-Lived Token (60 days):**

```bash
curl -X GET "https://graph.facebook.com/v21.0/oauth/access_token?grant_type=fb_exchange_token&client_id=YOUR_APP_ID&client_secret=YOUR_APP_SECRET&fb_exchange_token=SHORT_LIVED_TOKEN"
```

Save the `access_token` from the response - this is valid for 60 days.

**Note**: You'll need to refresh this token before it expires (60 days).

### Step 6: Get Your Instagram Business Account ID

```bash
curl -X GET "https://graph.facebook.com/v21.0/me/accounts?access_token=YOUR_ACCESS_TOKEN"
```

From the response, get your Facebook Page ID, then:

```bash
curl -X GET "https://graph.facebook.com/v21.0/PAGE_ID?fields=instagram_business_account&access_token=YOUR_ACCESS_TOKEN"
```

Save the `instagram_business_account.id` value.

### Step 7: Get Instagram Post ID

To get the ID of a specific post:

1. Go to [developers.facebook.com/tools/explorer](https://developers.facebook.com/tools/explorer)
2. Use this query: `YOUR_IG_ACCOUNT_ID/media?fields=id,caption,media_type,media_url,timestamp`
3. Find your post in the results and copy its ID

### Step 8: Configure Webhook

1. In your Facebook App dashboard, go to "Webhooks"
2. Click "Instagram" → "Edit Subscription"
3. Enter your callback URL: `https://yourdomain.com/api/webhook/instagram`
4. Enter a verify token (create a random string, save it for later)
5. Subscribe to the `comments` field
6. Save the subscription

**Important**: Your server must be publicly accessible for webhooks to work. Use ngrok for local testing:

```bash
npx ngrok http 3000
# Use the https URL provided as your webhook URL
```

## 🚀 Installation

### 1. Clone and Install

```bash
git clone <your-repo>
cd instagram-automation
npm install
```

### 2. Configure Environment Variables

Copy `.env.example` to `.env`:

```bash
cp .env.example .env
```

Edit `.env` with your credentials:

```env
DATABASE_URL=postgresql://user:password@localhost:5432/instagram_automation

INSTAGRAM_ACCESS_TOKEN=your_long_lived_token_here
INSTAGRAM_BUSINESS_ACCOUNT_ID=your_ig_business_account_id
FACEBOOK_PAGE_ID=your_facebook_page_id

FACEBOOK_APP_SECRET=your_app_secret_from_facebook
WEBHOOK_VERIFY_TOKEN=your_random_verify_token_string
```

### 3. Setup Database

```bash
# Push schema to database
npx drizzle-kit push
```

### 4. Run the Application

**Development:**
```bash
npm run dev
```

**Production:**
```bash
npm run build
npm start
```

Visit `http://localhost:3000` to access the dashboard.

## 📖 Usage Guide

### 1. Configure Instagram Credentials

1. Navigate to the **Settings** tab
2. Enter your Instagram Access Token
3. Enter your Instagram Business Account ID
4. Enter your Facebook Page ID (optional)
5. Enter your Webhook Verify Token
6. Enter your Facebook App Secret
7. Click **Save Configuration**

### 2. Create a Campaign

1. Go to the **Campaigns** tab
2. Click **+ New Campaign**
3. Enter the Instagram Post ID
4. The system will automatically fetch the post preview
5. Enter trigger keywords (comma-separated, e.g., "free, download, link")
6. Write your comment reply message
7. Write your DM message
8. Toggle "Campaign is active" if you want it to start immediately
9. Click **Create Campaign**

### 3. Test the Automation

1. Have someone comment on your tracked post with one of your trigger keywords
2. The webhook will receive the notification
3. The system will automatically:
   - Reply to the comment publicly
   - Send a DM to the commenter (if allowed)
   - Log the processed comment to prevent duplicates

### 4. Monitor Campaigns

- View all campaigns in the dashboard
- Toggle campaigns on/off with the play/pause button
- Edit campaigns to update keywords or messages
- Delete campaigns you no longer need

## 🔒 Security

- All webhook requests are verified using HMAC-SHA256 signatures
- Access tokens and secrets are stored in environment variables
- Credentials are never exposed to the frontend
- Database credentials are encrypted

## ⚠️ Important Limitations

### Direct Messages (DMs)

Instagram DMs via the Graph API have restrictions:

1. **User Must Initiate**: The user must have messaged your business first before you can send them a DM
2. **OR Get Approved**: Apply for `instagram_manage_messages` permission with a valid use case
3. **Messaging Window**: Without approval, you can only message users who messaged you in the last 24 hours

If DM sending fails, the comment reply will still work. The error is logged but doesn't stop the automation.

### Access Token Expiry

- Long-lived tokens expire after 60 days
- You'll need to manually refresh tokens before expiry
- Set up a calendar reminder to refresh tokens every 50 days

### Rate Limits

- Instagram API has rate limits (200 calls per user per hour)
- The application includes retry logic with exponential backoff
- For high-volume posts, monitor your API usage

## 🐛 Troubleshooting

### Webhook Not Receiving Events

1. Check that your server is publicly accessible
2. Verify the webhook URL in Facebook App settings
3. Ensure you've subscribed to the `comments` field
4. Check server logs for verification errors

### DMs Not Sending

1. Verify you have `instagram_manage_messages` permission
2. Check if the user has messaged your business before
3. Review error logs for specific error messages
4. Consider applying for messaging permissions from Facebook

### Post Preview Not Loading

1. Verify your access token is valid
2. Check that the post ID is correct
3. Ensure the post belongs to your Instagram account
4. Try refreshing your access token

### Campaign Not Triggering

1. Ensure the campaign is marked as "Active"
2. Verify the post ID matches exactly
3. Check that keywords are spelled correctly (case-insensitive)
4. Look at server logs for processing errors

## 📊 Database Schema

### `config`
- Stores Instagram API credentials
- Single row table

### `campaigns`
- Stores automation campaigns
- Links post ID to keywords and messages
- Supports active/inactive toggle

### `processed_comments`
- Tracks which comments have been processed
- Prevents duplicate processing
- References campaign ID

## 🔗 API Endpoints

### Webhooks
- `GET /api/webhook/instagram` - Webhook verification
- `POST /api/webhook/instagram` - Receive comment events

### Configuration
- `GET /api/config` - Get current config
- `POST /api/config` - Update config

### Campaigns
- `GET /api/campaigns` - List all campaigns
- `POST /api/campaigns` - Create campaign
- `PATCH /api/campaigns/[id]` - Update campaign
- `DELETE /api/campaigns/[id]` - Delete campaign

### Utilities
- `GET /api/post-preview?postId=xxx` - Fetch post details
- `GET /api/health` - Health check endpoint

## 🚢 Deployment

### Environment Variables

Make sure to set all required environment variables in your deployment platform:

```
DATABASE_URL
INSTAGRAM_ACCESS_TOKEN
INSTAGRAM_BUSINESS_ACCOUNT_ID
FACEBOOK_PAGE_ID (optional)
FACEBOOK_APP_SECRET
WEBHOOK_VERIFY_TOKEN
```

### Deploy to Vercel

```bash
vercel deploy --prod
```

### Deploy to Railway

```bash
railway up
```

### Deploy to Render

Create a new Web Service and connect your repository. Render will automatically detect the Next.js app.

## 📝 License

MIT License - feel free to use this for personal or commercial projects.

## 🤝 Contributing

Contributions are welcome! Please open an issue or submit a pull request.

## ⭐ Acknowledgments

Built with:
- [Next.js](https://nextjs.org/)
- [Instagram Graph API](https://developers.facebook.com/docs/instagram-api)
- [Drizzle ORM](https://orm.drizzle.team/)
- [Tailwind CSS](https://tailwindcss.com/)

---

**Need help?** Check the troubleshooting section or open an issue on GitHub.

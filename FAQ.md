# Frequently Asked Questions (FAQ)

Common questions about the Instagram Comment-to-DM Automation Tool.

## General Questions

### What does this tool do?

This tool automatically:
1. Monitors comments on your Instagram posts
2. Detects specific keywords in comments
3. Replies to the comment publicly
4. Sends a direct message (DM) to the commenter

Perfect for lead generation, customer service, and engagement automation.

### Is this legal and allowed by Instagram?

Yes! This tool uses the **official Instagram Graph API** provided by Facebook/Meta. It doesn't violate Instagram's Terms of Service when used properly. However:

- ✅ Using official API = Legal and safe
- ❌ Third-party scrapers = Against ToS
- ⚠️ Spamming = Can get you banned

Always provide value to users and follow Instagram's Community Guidelines.

### Do I need a business account?

Yes, you need an Instagram Business or Creator account. Personal accounts cannot access the Instagram Graph API.

### How much does it cost?

The tool itself is **free and open-source**. However, you may have costs for:

- Database hosting (free tier available on most platforms)
- Server hosting (free tier available: Vercel, Railway, Render)
- Domain name (optional, ~$10/year)

Most users can run this **completely free** using free tiers.

---

## Setup Questions

### How long does setup take?

- **First time**: 30-60 minutes (includes Facebook App setup)
- **Subsequent deployments**: 5-10 minutes

Follow the [QUICK_START.md](QUICK_START.md) guide for fastest setup.

### Do I need coding experience?

Basic technical knowledge is helpful, but detailed guides are provided. You should be comfortable:

- Using command line terminal
- Copying/pasting commands
- Setting environment variables
- Basic troubleshooting

No programming required to use the tool!

### What if I don't have a Facebook Developer account?

Creating one is free and takes 5 minutes:
1. Go to [developers.facebook.com](https://developers.facebook.com)
2. Click "Get Started"
3. Complete registration
4. Verify your email

### Can I test locally before deploying?

Yes! Run locally with:
```bash
npm run dev
```

Use [ngrok](https://ngrok.com/) to expose your local server for webhook testing:
```bash
npx ngrok http 3000
```

---

## Features & Functionality

### How many campaigns can I create?

Unlimited! Create as many campaigns as you need for different posts and keywords.

### Can I have multiple keywords per campaign?

Yes! Add keywords separated by commas:
```
free, download, link, guide
```

All keywords will trigger the same response.

### Is keyword matching case-sensitive?

No, matching is **case-insensitive**:
- "FREE" = "free" = "Free"

### Is it an exact match or partial match?

**Partial match**. The keyword just needs to appear anywhere in the comment:
- Keyword: "free"
- Matches: "free download", "Want this free", "is this free?"

### Can I use emojis in responses?

Absolutely! Emojis work in both comment replies and DMs:
```
Check your DMs! 💬✨
Here's your free guide 📚🎁
```

### How fast are the responses?

Nearly instant! Responses typically happen within:
- Comment reply: 1-3 seconds
- DM: 2-5 seconds

Depends on Instagram API response time and your server location.

### What happens if someone comments multiple times?

Each unique comment ID is processed only once (deduplication). If the same person comments again with a different comment, it will be processed as a new comment.

### Can I edit a campaign after creating it?

Yes! Click the edit button (✏️) to modify:
- Keywords
- Comment reply message
- DM message
- Active/inactive status

You cannot edit the Post ID (create a new campaign instead).

### How do I temporarily disable a campaign?

Click the pause button (⏸) next to the campaign. It will become inactive without deleting it. Click play (▶) to reactivate.

---

## Instagram API Questions

### Why can't I send DMs to everyone?

Instagram restricts DMs for privacy and anti-spam reasons. You can only DM users who:

1. Have messaged your business first, OR
2. You have special `instagram_manage_messages` approval from Facebook

**Solutions**:
- Encourage followers to DM you first
- Use Stories with "Send Message" sticker
- Apply for messaging permissions (requires valid business use case)

Comment replies always work regardless!

### How long do access tokens last?

- **Short-lived tokens**: 1 hour
- **Long-lived tokens**: 60 days

Always use long-lived tokens and set a reminder to refresh before day 60.

### How do I refresh an expired token?

Generate a new token using the same process as initial setup:
1. Go to Graph API Explorer
2. Select your app
3. Generate new access token
4. Update in Settings tab

See [SETUP_GUIDE.md](SETUP_GUIDE.md) for detailed instructions.

### What happens if my token expires?

The automation will stop working. You'll see errors in logs:
```
Error validating access token
```

Update the token in Settings to resume automation.

### Are there rate limits?

Yes, Instagram allows **200 API calls per hour** per access token.

For high-traffic posts, you may hit this limit. The app includes automatic retry logic with exponential backoff.

### Can I use this with Reels and IGTV?

Yes! Any Instagram post type works as long as you can get the media ID:
- Photos
- Videos
- Carousels
- Reels
- IGTV

---

## Technical Questions

### What technology stack is used?

- **Frontend**: Next.js, React, TypeScript, Tailwind CSS
- **Backend**: Next.js API Routes
- **Database**: PostgreSQL with Drizzle ORM
- **Deployment**: Docker, Vercel, Railway, or Render

### Where is data stored?

All data is stored in your PostgreSQL database:
- Campaign configurations
- Instagram credentials (encrypted)
- Processed comment IDs

Nothing is stored on third-party servers.

### Is my data secure?

Yes! Security measures include:
- HMAC signature verification for webhooks
- Environment variable encryption
- HTTPS-only communication
- No client-side credential exposure
- Database credential encryption

### Can I self-host this?

Absolutely! The tool is designed for self-hosting. Deploy to:
- Your own VPS (DigitalOcean, Linode, AWS, etc.)
- Docker container
- Kubernetes cluster
- Any Node.js-compatible host

### Does this work with Cloudflare?

Yes! Works great with Cloudflare. Just ensure:
- Webhooks can reach your origin server
- Don't cache webhook endpoints
- SSL/TLS is properly configured

### Can I use SQLite instead of PostgreSQL?

Not currently. The code uses PostgreSQL-specific features. However, you could adapt the schema for SQLite with minor changes to `src/db/schema.ts`.

---

## Deployment Questions

### Which hosting platform is best?

**Recommended**:
- **Vercel**: Easiest for Next.js, free tier available
- **Railway**: Good for full-stack apps, includes database
- **Render**: Similar to Railway, generous free tier

All three work great. Choose based on your preference.

### Do I need a custom domain?

No, you can use the default domain provided by your hosting platform:
- Vercel: `yourapp.vercel.app`
- Railway: `yourapp.up.railway.app`
- Render: `yourapp.onrender.com`

Custom domains are optional.

### How much traffic can it handle?

Depends on your hosting plan:
- **Free tiers**: ~100-500 campaigns/hour
- **Paid tiers**: Thousands of campaigns/hour

Most users never exceed free tier limits.

### What if my server goes down?

Instagram will retry webhook deliveries:
- Multiple retry attempts over 24 hours
- If server is down temporarily, events will be re-delivered
- Deduplication prevents double-processing

### Can I run multiple instances?

Yes, but be careful with webhook processing. Consider:
- Use a queue system for high volume
- Ensure database handles concurrent writes
- Monitor for duplicate processing

For most use cases, a single instance is sufficient.

---

## Campaign Strategy Questions

### What are the best keywords to use?

**Effective keywords**:
- Action-oriented: "download", "get", "claim", "send"
- Value words: "free", "discount", "offer", "bonus"
- Specific: Use unique codes like "SAVE2024"

**Avoid**:
- Too generic: "yes", "ok", "thanks"
- Too broad: "info", "help"

### How do I prevent spam triggering?

Use **specific, unique keywords**:

❌ Bad: "free" (triggers on "this is free shipping?")
✅ Good: "FREEGUIDE2024" (unlikely to appear in normal conversation)

### Can I track conversions?

Not built-in currently, but you can:
- Use unique URLs in DMs (e.g., with UTM parameters)
- Use unique discount codes
- Track clicks via link shorteners
- Check the `processed_comments` database table

Future versions may include analytics dashboard.

### Should I use the same message for everyone?

Depends on your goal:
- **Lead magnets**: Same message works (delivering promised content)
- **Customer service**: May want different campaigns for different questions
- **Sales**: Test different messages (A/B testing)

You can create multiple campaigns for the same post with different keywords.

### How do I A/B test messages?

Currently manual:
1. Create two campaigns with different keywords
2. Tell different audiences to use different keywords
3. Track which performs better
4. Update campaigns with winning message

Automated A/B testing may come in future updates.

---

## Compliance Questions

### Is this GDPR compliant?

The tool itself doesn't collect personal data beyond what Instagram provides. However:

**Your responsibility**:
- Have a privacy policy
- Inform users about automated messaging
- Provide opt-out mechanism
- Store only necessary data
- Delete data on request

Consult a lawyer for full GDPR compliance.

### Do I need to disclose this is automated?

**Best practice**: Yes, be transparent:
- Mention automated responses in your bio
- Include "This is an automated message" in DMs
- Provide way to reach a human

Transparency builds trust!

### Can I use this for political campaigns?

Check local laws and Instagram's policies. Generally:
- Political advertising has strict rules
- May require additional permissions
- Varies by country/region

Consult with a legal expert before using for political purposes.

---

## Support Questions

### Something isn't working. What should I do?

**Troubleshooting steps**:
1. Check [TROUBLESHOOTING.md](TROUBLESHOOTING.md)
2. Review application logs
3. Verify environment variables
4. Test with a simple campaign first
5. Check Facebook webhook logs

### Where can I get help?

1. **Documentation**:
   - [README.md](README.md) - Overview and features
   - [SETUP_GUIDE.md](SETUP_GUIDE.md) - Detailed setup
   - [QUICK_START.md](QUICK_START.md) - Fast setup
   - [TROUBLESHOOTING.md](TROUBLESHOOTING.md) - Fix issues
   - [API_DOCUMENTATION.md](API_DOCUMENTATION.md) - API reference

2. **Community**:
   - GitHub Issues
   - Discussions tab

3. **Facebook/Instagram**:
   - [Instagram API Docs](https://developers.facebook.com/docs/instagram-api)
   - [Webhooks Reference](https://developers.facebook.com/docs/graph-api/webhooks)

### Can I hire you to set this up?

This is an open-source project. For commercial support:
- Check if contributors offer paid services
- Hire a freelance developer familiar with Next.js
- Post on job boards (Upwork, Fiverr, etc.)

### How can I contribute?

See [CONTRIBUTING.md](CONTRIBUTING.md) for:
- Reporting bugs
- Suggesting features
- Contributing code
- Improving documentation

All contributions welcome! 🎉

---

## Future Features

### Will there be analytics?

Planned for version 1.1.0:
- Response rates
- Engagement metrics
- Keyword performance
- Campaign comparison

### Will you add multi-account support?

Yes, planned for version 1.2.0:
- Manage multiple Instagram accounts
- Switch between accounts easily
- Separate campaigns per account

### Can I suggest new features?

Absolutely! Open an issue on GitHub with:
- Feature description
- Use case
- Why it would be valuable

Most requested features get prioritized.

### Will there be a mobile app?

Not currently planned, but the web dashboard works great on mobile browsers. Progressive Web App (PWA) support may come in the future.

---

## Licensing Questions

### Is this free to use?

Yes! Licensed under MIT License. You can:
- Use commercially
- Modify the code
- Distribute it
- Use privately

No attribution required (but appreciated!).

### Can I sell this?

Yes, under MIT License you can:
- Sell hosting/setup services
- Create paid managed versions
- Offer commercial support
- White-label for clients

### Can I use this for client projects?

Absolutely! Perfect for:
- Social media agencies
- Marketing consultants
- Freelance developers
- Digital marketing services

---

## Performance Questions

### How many comments can it handle?

Depends on:
- Server resources
- Database performance
- Instagram API rate limits (200/hour)

**Typical capacity**:
- Free tier: 50-100 comments/hour
- Paid tier: 150-200 comments/hour (API limit)

For viral posts, you may need rate limit handling.

### Does it slow down my website?

No! The automation runs on:
- Separate API routes
- Triggered by webhooks
- Independent of your website

Your website performance is unaffected.

### Will it work on old posts?

Yes! Create a campaign for any post at any time. It will start processing new comments immediately, regardless of post age.

Old comments (before campaign creation) won't be processed.

---

**Have a question not answered here?** Open an issue on GitHub!

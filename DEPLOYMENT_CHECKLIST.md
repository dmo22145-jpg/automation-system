# Deployment Checklist

Use this checklist to ensure a smooth deployment of the Instagram Automation Tool.

## Pre-Deployment Checklist

### Instagram API Setup

- [ ] Instagram account converted to Business/Creator
- [ ] Facebook Page created and connected to Instagram
- [ ] Facebook Developer account created
- [ ] Facebook App created with Instagram product added
- [ ] Requested and approved necessary permissions:
  - [ ] `instagram_basic`
  - [ ] `instagram_manage_comments`
  - [ ] `instagram_manage_messages`
  - [ ] `pages_show_list`
  - [ ] `pages_read_engagement`
- [ ] Long-lived access token generated (60-day expiry)
- [ ] Instagram Business Account ID obtained
- [ ] Facebook Page ID obtained (optional)
- [ ] App Secret noted from Facebook App settings
- [ ] Webhook verify token created (random string)

### Local Testing

- [ ] Cloned repository
- [ ] Dependencies installed (`npm install`)
- [ ] Environment variables configured in `.env`
- [ ] Database connection string valid
- [ ] Schema pushed to database (`npx drizzle-kit push`)
- [ ] Application runs locally (`npm run dev`)
- [ ] Health endpoint responds (`curl http://localhost:3000/api/health`)
- [ ] Dashboard loads in browser
- [ ] Settings can be saved
- [ ] Test campaign created
- [ ] Post preview fetches successfully

### Code Quality

- [ ] TypeScript check passes (`npm run typecheck`)
- [ ] Build succeeds (`npm run build`)
- [ ] No console errors in browser
- [ ] No errors in terminal/logs
- [ ] All API endpoints respond correctly

## Deployment Platform Setup

Choose your platform and complete the relevant section:

### Option A: Vercel

- [ ] Vercel account created
- [ ] Vercel CLI installed (`npm i -g vercel`)
- [ ] Project linked to Vercel
- [ ] PostgreSQL addon added (or external database configured)
- [ ] Environment variables added in Vercel dashboard:
  - [ ] `DATABASE_URL`
  - [ ] `INSTAGRAM_ACCESS_TOKEN`
  - [ ] `INSTAGRAM_BUSINESS_ACCOUNT_ID`
  - [ ] `FACEBOOK_PAGE_ID`
  - [ ] `FACEBOOK_APP_SECRET`
  - [ ] `WEBHOOK_VERIFY_TOKEN`
- [ ] Domain configured (if using custom domain)
- [ ] Deployed (`vercel deploy --prod`)
- [ ] Deployment succeeded
- [ ] Health endpoint accessible online

### Option B: Railway

- [ ] Railway account created
- [ ] Railway CLI installed
- [ ] PostgreSQL database provisioned
- [ ] Environment variables configured in Railway dashboard
- [ ] `railway.toml` present in project root
- [ ] Deployed (`railway up`)
- [ ] Deployment succeeded
- [ ] Health endpoint accessible online

### Option C: Render

- [ ] Render account created
- [ ] GitHub repository connected
- [ ] PostgreSQL database provisioned
- [ ] Environment variables configured in Render dashboard
- [ ] `render.yaml` present in project root
- [ ] Auto-deploy enabled
- [ ] Deployment succeeded
- [ ] Health endpoint accessible online

### Option D: Self-Hosted

- [ ] VPS/server provisioned
- [ ] Docker installed on server
- [ ] PostgreSQL database accessible
- [ ] Environment variables configured
- [ ] Dockerfile present and tested
- [ ] Image built successfully
- [ ] Container running
- [ ] Reverse proxy configured (nginx/Caddy)
- [ ] SSL certificate installed (Let's Encrypt)
- [ ] Health endpoint accessible via HTTPS

## Database Setup

- [ ] Database connection successful
- [ ] Schema pushed/migrated
- [ ] Tables created:
  - [ ] `config`
  - [ ] `campaigns`
  - [ ] `processed_comments`
- [ ] Database backup configured (production)
- [ ] Connection pooling configured (if needed)

## Webhook Configuration

- [ ] Application deployed and accessible via HTTPS
- [ ] Webhook URL noted: `https://yourdomain.com/api/webhook/instagram`
- [ ] In Facebook App → Webhooks:
  - [ ] Callback URL entered
  - [ ] Verify token entered (matches `WEBHOOK_VERIFY_TOKEN`)
  - [ ] Verification successful (green checkmark)
  - [ ] Subscribed to `comments` field
- [ ] Test event sent from Facebook
- [ ] Test event received successfully (check logs)

## Application Configuration

- [ ] Accessed deployed dashboard
- [ ] Navigated to Settings tab
- [ ] Entered all credentials:
  - [ ] Instagram Access Token
  - [ ] Instagram Business Account ID
  - [ ] Facebook Page ID
  - [ ] Webhook Verify Token
  - [ ] Facebook App Secret
- [ ] Saved configuration successfully
- [ ] No error messages

## Functional Testing

### Test 1: Campaign Creation

- [ ] Clicked "New Campaign"
- [ ] Entered valid Instagram post ID
- [ ] Post preview loaded correctly
- [ ] Added test keywords: `test, automation, bot`
- [ ] Wrote comment reply message
- [ ] Wrote DM message
- [ ] Campaign created successfully
- [ ] Campaign appears in list with correct details

### Test 2: Comment Automation

- [ ] Used test Instagram account (not main account)
- [ ] Commented on tracked post with keyword
- [ ] Within 5 seconds:
  - [ ] Comment reply appeared
  - [ ] DM received (if user messaged first)
- [ ] Checked logs for successful processing
- [ ] Commented again with same keyword
- [ ] Deduplication worked (no second reply)

### Test 3: Campaign Management

- [ ] Edited campaign
- [ ] Changed keywords
- [ ] Saved successfully
- [ ] Toggled campaign inactive
- [ ] Badge changed to gray "Inactive"
- [ ] Commented on post (should not trigger)
- [ ] Toggled campaign active again
- [ ] Commented on post (should trigger)
- [ ] Deleted test campaign
- [ ] Campaign removed from list

## Monitoring Setup

- [ ] Application logs accessible
- [ ] Webhook delivery logs checked in Facebook App
- [ ] Error tracking configured (Sentry, LogRocket, etc.) (optional)
- [ ] Uptime monitoring configured (UptimeRobot, etc.) (optional)
- [ ] Database monitoring configured (optional)
- [ ] Performance metrics tracked (optional)

## Security Verification

- [ ] All endpoints use HTTPS
- [ ] Environment variables not exposed in frontend
- [ ] Webhook signature verification enabled
- [ ] `.env` file not committed to Git
- [ ] `.gitignore` includes sensitive files
- [ ] Database credentials secured
- [ ] Access tokens not in browser DevTools

## Documentation Review

- [ ] README.md reviewed
- [ ] Setup guide bookmarked
- [ ] Troubleshooting guide reviewed
- [ ] FAQ reviewed for common issues
- [ ] API documentation available for reference
- [ ] Team members have access to documentation (if applicable)

## Post-Deployment Tasks

### Immediate (Day 1)

- [ ] Monitor logs for any errors
- [ ] Test webhook delivery with real comments
- [ ] Verify all campaigns triggering correctly
- [ ] Check database connections stable
- [ ] Confirm no rate limit errors
- [ ] Document actual webhook URL for team
- [ ] Share dashboard URL with stakeholders

### Short-term (Week 1)

- [ ] Create additional campaigns for production posts
- [ ] Monitor API usage in Facebook App dashboard
- [ ] Check response times acceptable
- [ ] Review processed comments in database
- [ ] Optimize messages based on feedback
- [ ] A/B test different reply messages
- [ ] Document any issues encountered

### Medium-term (Month 1)

- [ ] Set calendar reminder for token refresh (day 50)
- [ ] Review campaign performance
- [ ] Adjust keywords based on usage
- [ ] Scale to more posts if successful
- [ ] Consider upgrading hosting plan if needed
- [ ] Backup database regularly
- [ ] Review and update documentation

## Token Refresh Reminder

⚠️ **Critical**: Access tokens expire after 60 days!

- [ ] Calendar reminder set for day 50
- [ ] Backup reminder set for day 55
- [ ] Token refresh process documented
- [ ] Team knows how to refresh tokens
- [ ] Test token refresh process before expiry

## Rollback Plan

In case of issues, know how to rollback:

- [ ] Previous deployment/version tagged
- [ ] Database backup available
- [ ] Rollback commands documented
- [ ] Team knows rollback procedure
- [ ] Webhook can be quickly disabled in Facebook App

## Success Criteria

Deployment is successful when:

- [ ] ✅ Health endpoint returns 200 OK
- [ ] ✅ Dashboard loads without errors
- [ ] ✅ Webhook verification successful
- [ ] ✅ Test campaign created
- [ ] ✅ Test comment triggers automation
- [ ] ✅ Comment reply posted successfully
- [ ] ✅ DM sent successfully (or expected error logged)
- [ ] ✅ Deduplication prevents double-processing
- [ ] ✅ No errors in application logs
- [ ] ✅ All team members can access dashboard

## Emergency Contacts

Document these for your team:

- **Deployment Platform Support**: _______________
- **Database Provider Support**: _______________
- **Domain Registrar Support**: _______________
- **Facebook Developer Support**: https://developers.facebook.com/support
- **Team Lead**: _______________
- **Technical Contact**: _______________

## Helpful Commands

```bash
# Check application health
curl https://yourdomain.com/api/health

# View logs (Vercel)
vercel logs

# View logs (Railway)
railway logs

# View logs (Docker)
docker logs container-name

# Connect to database
psql $DATABASE_URL

# Check database tables
psql $DATABASE_URL -c "\dt"

# Test webhook locally
npx ngrok http 3000

# Rebuild and redeploy
npm run build
vercel deploy --prod
```

## Notes Section

Use this space to document deployment-specific notes:

**Deployment Date**: _______________  
**Deployment URL**: _______________  
**Database URL**: _______________  
**Webhook URL**: _______________  
**Special Configurations**: _______________  
**Known Issues**: _______________  
**Next Steps**: _______________  

---

## Final Checklist Sign-off

Once ALL items above are checked, deployment is complete!

- [ ] All pre-deployment tasks complete
- [ ] Platform setup complete
- [ ] Database setup complete
- [ ] Webhook configured and verified
- [ ] Application configured
- [ ] Functional tests passed
- [ ] Monitoring setup complete
- [ ] Security verified
- [ ] Documentation reviewed
- [ ] Post-deployment tasks scheduled

**Deployed by**: _______________  
**Date**: _______________  
**Sign-off**: _______________  

---

**Congratulations! Your Instagram Automation Tool is live! 🎉**

Next: Start creating campaigns and automating your Instagram engagement!

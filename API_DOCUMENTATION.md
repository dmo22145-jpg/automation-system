# API Documentation

Complete API reference for the Instagram Comment-to-DM Automation Tool.

## Base URL

- **Development**: `http://localhost:3000`
- **Production**: `https://yourdomain.com`

---

## Authentication

Most API endpoints are internal and don't require authentication. The webhook endpoints verify requests using HMAC signatures.

---

## Endpoints

### Health Check

#### `GET /api/health`

Check if the application is running.

**Response:**
```json
{
  "status": "ok"
}
```

**Status Codes:**
- `200` - Application is healthy

---

### Configuration

#### `GET /api/config`

Retrieve current Instagram API configuration.

**Response:**
```json
{
  "instagramAccessToken": "••••••••",
  "instagramBusinessAccountId": "17841405793187218",
  "pageId": "123456789012345",
  "webhookVerifyToken": "••••••••",
  "facebookAppSecret": "••••••••",
  "hasConfig": true
}
```

**Notes:**
- Sensitive values are masked with `••••••••`
- `hasConfig` indicates if credentials are configured

**Status Codes:**
- `200` - Success
- `500` - Server error

---

#### `POST /api/config`

Update Instagram API configuration.

**Request Body:**
```json
{
  "instagramAccessToken": "EAAxxxx...",
  "instagramBusinessAccountId": "17841405793187218",
  "pageId": "123456789012345",
  "webhookVerifyToken": "my_secret_token",
  "facebookAppSecret": "abc123def456"
}
```

**Response:**
```json
{
  "success": true
}
```

**Status Codes:**
- `200` - Configuration updated
- `500` - Server error

---

### Campaigns

#### `GET /api/campaigns`

List all campaigns.

**Response:**
```json
[
  {
    "id": 1,
    "postId": "17895695668004550",
    "postThumbnail": "https://scontent.cdninstagram.com/...",
    "postCaption": "Check out our new product!",
    "keywords": "free, download, link",
    "commentReply": "Thanks for commenting! Check your DMs 💬",
    "dmMessage": "Here's the download link: https://example.com",
    "isActive": true,
    "createdAt": "2024-01-15T12:00:00.000Z",
    "updatedAt": "2024-01-15T12:00:00.000Z"
  }
]
```

**Status Codes:**
- `200` - Success
- `500` - Server error

---

#### `POST /api/campaigns`

Create a new campaign.

**Request Body:**
```json
{
  "postId": "17895695668004550",
  "keywords": "free, download, link",
  "commentReply": "Thanks for commenting! Check your DMs 💬",
  "dmMessage": "Here's the download link: https://example.com",
  "isActive": true
}
```

**Response:**
```json
{
  "id": 1,
  "postId": "17895695668004550",
  "postThumbnail": "https://scontent.cdninstagram.com/...",
  "postCaption": "Check out our new product!",
  "keywords": "free, download, link",
  "commentReply": "Thanks for commenting! Check your DMs 💬",
  "dmMessage": "Here's the download link: https://example.com",
  "isActive": true,
  "createdAt": "2024-01-15T12:00:00.000Z",
  "updatedAt": "2024-01-15T12:00:00.000Z"
}
```

**Notes:**
- Post details (thumbnail, caption) are automatically fetched from Instagram
- If Instagram API fails, campaign is still created without preview

**Status Codes:**
- `201` - Campaign created
- `400` - Missing required fields
- `500` - Server error

---

#### `PATCH /api/campaigns/[id]`

Update an existing campaign.

**URL Parameters:**
- `id` - Campaign ID (integer)

**Request Body (partial update allowed):**
```json
{
  "keywords": "new, keywords, here",
  "commentReply": "Updated reply message",
  "dmMessage": "Updated DM message",
  "isActive": false
}
```

**Response:**
```json
{
  "id": 1,
  "postId": "17895695668004550",
  "keywords": "new, keywords, here",
  "commentReply": "Updated reply message",
  "dmMessage": "Updated DM message",
  "isActive": false,
  "updatedAt": "2024-01-15T13:00:00.000Z"
}
```

**Status Codes:**
- `200` - Campaign updated
- `404` - Campaign not found
- `500` - Server error

---

#### `DELETE /api/campaigns/[id]`

Delete a campaign.

**URL Parameters:**
- `id` - Campaign ID (integer)

**Response:**
```json
{
  "success": true
}
```

**Notes:**
- Cascade deletes all associated processed comments

**Status Codes:**
- `200` - Campaign deleted
- `500` - Server error

---

### Post Preview

#### `GET /api/post-preview?postId=xxx`

Fetch Instagram post details for preview.

**Query Parameters:**
- `postId` - Instagram media ID (required)

**Response:**
```json
{
  "id": "17895695668004550",
  "caption": "Check out our new product! 🚀",
  "mediaType": "IMAGE",
  "thumbnail": "https://scontent.cdninstagram.com/...",
  "permalink": "https://www.instagram.com/p/ABC123/"
}
```

**Status Codes:**
- `200` - Success
- `400` - Missing postId or credentials not configured
- `500` - Failed to fetch from Instagram API

**Example Error:**
```json
{
  "error": "Instagram credentials not configured"
}
```

---

### Webhook

#### `GET /api/webhook/instagram`

Webhook verification endpoint (called by Facebook).

**Query Parameters:**
- `hub.mode` - Should be "subscribe"
- `hub.verify_token` - Your verification token
- `hub.challenge` - Challenge string to echo back

**Response:**
- Returns the `hub.challenge` value if verification succeeds
- Returns `403` if verification fails

**Example Request:**
```
GET /api/webhook/instagram?hub.mode=subscribe&hub.verify_token=my_token&hub.challenge=abc123
```

**Success Response:**
```
abc123
```

**Status Codes:**
- `200` - Verification successful
- `403` - Verification failed

---

#### `POST /api/webhook/instagram`

Webhook event receiver (called by Facebook when comments are posted).

**Headers:**
- `x-hub-signature-256` - HMAC signature for verification

**Request Body (example comment event):**
```json
{
  "entry": [
    {
      "id": "123456789",
      "time": 1234567890,
      "changes": [
        {
          "field": "comments",
          "value": {
            "id": "17895695668004550",
            "text": "free download please!",
            "media": {
              "id": "17841405793187218"
            },
            "from": {
              "id": "987654321",
              "username": "john_doe"
            }
          }
        }
      ]
    }
  ]
}
```

**Response:**
```json
{
  "success": true
}
```

**Processing Flow:**
1. Verify webhook signature
2. Extract comment details
3. Check if comment already processed (deduplication)
4. Find matching active campaigns
5. Check if comment contains trigger keywords
6. Reply to comment via Instagram API
7. Send DM to commenter via Instagram API
8. Store processed comment in database

**Status Codes:**
- `200` - Event processed successfully
- `403` - Invalid signature
- `500` - Processing error

**Notes:**
- Invalid signatures are rejected for security
- Failed DMs don't prevent comment replies
- Duplicate comments are silently ignored

---

## Instagram Graph API Integration

### Endpoints Used

This application makes calls to the following Instagram Graph API endpoints:

#### Reply to Comment
```
POST /{comment-id}/replies
```

**Parameters:**
- `message` - Reply text
- `access_token` - Instagram access token

#### Send DM
```
POST /me/messages
```

**Parameters:**
- `recipient.id` - Instagram-scoped user ID
- `message.text` - Message text
- `access_token` - Instagram access token

#### Get Post Details
```
GET /{media-id}?fields=id,caption,media_type,media_url,thumbnail_url,permalink
```

**Parameters:**
- `access_token` - Instagram access token

#### Get Comment Details
```
GET /{comment-id}?fields=id,text,username,from
```

**Parameters:**
- `access_token` - Instagram access token

---

## Error Handling

### Standard Error Response

```json
{
  "error": "Error message describing what went wrong"
}
```

### Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| `Instagram credentials not configured` | No access token set | Configure in Settings tab |
| `Failed to fetch post details` | Invalid post ID or expired token | Verify post ID and token |
| `Cannot send DM: User must message the business first` | DM restriction | User needs to initiate conversation |
| `Instagram API error: Rate limit exceeded` | Too many API calls | Wait and retry (auto-retry enabled) |
| `Invalid signature` | Webhook signature mismatch | Verify app secret matches |

---

## Rate Limits

### Instagram Graph API Limits

- **200 calls per user per hour**
- Rate limit applies per access token
- Exceeded limits return HTTP 429

### Retry Strategy

The application implements exponential backoff:
- Initial delay: 1 second
- Max retries: 3
- Delay multiplier: 2x

**Example:**
```
Attempt 1: Immediate
Attempt 2: 1 second delay
Attempt 3: 2 second delay
Attempt 4: 4 second delay
```

---

## Database Schema

### `config` Table

| Column | Type | Description |
|--------|------|-------------|
| `id` | serial | Primary key |
| `instagram_access_token` | text | Long-lived token |
| `instagram_business_account_id` | text | IG account ID |
| `page_id` | text | Facebook Page ID |
| `webhook_verify_token` | text | Webhook verification token |
| `facebook_app_secret` | text | App secret for signature verification |
| `updated_at` | timestamp | Last updated time |

### `campaigns` Table

| Column | Type | Description |
|--------|------|-------------|
| `id` | serial | Primary key |
| `post_id` | text | Instagram media ID |
| `post_thumbnail` | text | Post image URL (nullable) |
| `post_caption` | text | Post caption (nullable) |
| `keywords` | text | Comma-separated keywords |
| `comment_reply` | text | Public reply message |
| `dm_message` | text | Private DM message |
| `is_active` | boolean | Campaign enabled status |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last updated time |

### `processed_comments` Table

| Column | Type | Description |
|--------|------|-------------|
| `id` | serial | Primary key |
| `comment_id` | text | Instagram comment ID (unique) |
| `campaign_id` | serial | Foreign key to campaigns |
| `commenter_username` | text | Username who commented |
| `processed_at` | timestamp | Processing time |

---

## Security

### Webhook Signature Verification

All webhook requests are verified using HMAC-SHA256:

```javascript
const expectedSignature = 'sha256=' + 
  crypto.createHmac('sha256', APP_SECRET)
    .update(requestBody)
    .digest('hex');

if (signature !== expectedSignature) {
  // Reject request
}
```

### Environment Variables

Never expose these in client-side code:
- `INSTAGRAM_ACCESS_TOKEN`
- `FACEBOOK_APP_SECRET`
- `WEBHOOK_VERIFY_TOKEN`
- `DATABASE_URL`

### Best Practices

1. Use HTTPS only in production
2. Rotate access tokens before expiry
3. Monitor webhook logs for suspicious activity
4. Keep dependencies updated
5. Use environment-specific configurations

---

## Monitoring & Logging

### Application Logs

Check logs for:
```
✓ Webhook verified successfully
✓ Webhook received: {...}
✓ Comment matches campaign X with keywords: {...}
✓ Replying to comment...
✓ Comment reply sent successfully
✓ Sending DM to user...
✓ DM sent successfully
✓ Successfully processed comment

✗ Invalid webhook signature
✗ Failed to send DM (expected if user hasn't messaged first)
✗ Instagram API error: {...}
```

### Webhook Debugging

Facebook provides webhook debugging tools:
1. App Dashboard → Webhooks
2. Click "Test" to send sample events
3. View delivery logs and responses
4. Check for failed deliveries

---

## Testing

### Manual Testing

1. **Webhook Verification:**
   ```bash
   curl "https://yourdomain.com/api/webhook/instagram?hub.mode=subscribe&hub.verify_token=YOUR_TOKEN&hub.challenge=test123"
   # Should return: test123
   ```

2. **Post Preview:**
   ```bash
   curl "https://yourdomain.com/api/post-preview?postId=17895695668004550"
   ```

3. **Campaign Creation:**
   ```bash
   curl -X POST https://yourdomain.com/api/campaigns \
     -H "Content-Type: application/json" \
     -d '{
       "postId": "17895695668004550",
       "keywords": "test",
       "commentReply": "Test reply",
       "dmMessage": "Test DM"
     }'
   ```

### Integration Testing

Use ngrok for local webhook testing:
```bash
# Terminal 1
npm run dev

# Terminal 2
npx ngrok http 3000

# Use ngrok URL in Facebook webhook settings
```

---

## Support

For issues or questions:
- Check logs for errors
- Review [SETUP_GUIDE.md](SETUP_GUIDE.md)
- Consult [Instagram Graph API docs](https://developers.facebook.com/docs/instagram-api)
- Open an issue on GitHub

---

**Last Updated**: 2024-01-15

# System Architecture

Visual overview of the Instagram Comment-to-DM Automation Tool architecture.

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                          USER INTERFACE                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │
│  │   Settings   │  │  Campaigns   │  │  Dashboard   │             │
│  │    Panel     │  │     List     │  │   (Future)   │             │
│  └──────────────┘  └──────────────┘  └──────────────┘             │
│         │                  │                  │                     │
│         └──────────────────┴──────────────────┘                     │
│                            │                                        │
│                    Next.js Frontend                                 │
│                    (React Components)                               │
└─────────────────────────────┬───────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                       API LAYER (Next.js)                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐  │
│  │   Config   │  │ Campaigns  │  │   Post     │  │  Webhook   │  │
│  │    API     │  │    API     │  │  Preview   │  │    API     │  │
│  └────────────┘  └────────────┘  └────────────┘  └────────────┘  │
│         │                │                │              │          │
│         │                ▼                │              │          │
│         │         ┌─────────────┐         │              │          │
│         │         │  Instagram  │◄────────┘              │          │
│         │         │   API Lib   │                        │          │
│         │         └─────────────┘                        │          │
│         │                │                                │          │
└─────────┴────────────────┴────────────────────────────────┴──────────┘
          │                │                                │
          ▼                ▼                                ▼
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────────┐
│   PostgreSQL     │  │ Instagram Graph  │  │  Facebook Webhook   │
│    Database      │  │      API         │  │     Platform        │
└──────────────────┘  └──────────────────┘  └──────────────────────┘
```

## Detailed Component Architecture

### 1. Frontend Layer

```
┌─────────────────────────────────────────────────────────────┐
│                    Frontend (Client-Side)                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────────────────────────────────┐          │
│  │           page.tsx (Main Dashboard)          │          │
│  │  - Tab navigation (Settings/Campaigns)       │          │
│  │  - State management (campaigns, settings)    │          │
│  │  - Event handlers                            │          │
│  └──────────────────┬───────────────────────────┘          │
│                     │                                       │
│        ┌────────────┼────────────┐                         │
│        ▼            ▼            ▼                          │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐                   │
│  │ Config   │ │Campaign  │ │Campaign  │                   │
│  │  Panel   │ │  Form    │ │  List    │                   │
│  └──────────┘ └──────────┘ └──────────┘                   │
│                                                              │
│  Features:                                                  │
│  • Real-time post preview                                   │
│  • Campaign CRUD operations                                 │
│  • Status badges (Active/Inactive)                          │
│  • Character counters                                       │
│  • Responsive design                                        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 2. API Layer

```
┌─────────────────────────────────────────────────────────────┐
│                  API Routes (Server-Side)                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  /api/config (GET, POST)                                    │
│  ├─ Get current Instagram credentials                       │
│  └─ Update Instagram credentials                            │
│                                                              │
│  /api/campaigns (GET, POST)                                 │
│  ├─ List all campaigns                                      │
│  └─ Create new campaign                                     │
│                                                              │
│  /api/campaigns/[id] (PATCH, DELETE)                        │
│  ├─ Update campaign                                         │
│  └─ Delete campaign                                         │
│                                                              │
│  /api/post-preview (GET)                                    │
│  └─ Fetch Instagram post details                            │
│                                                              │
│  /api/webhook/instagram (GET, POST)                         │
│  ├─ Verify webhook (GET)                                    │
│  └─ Process events (POST)                                   │
│                                                              │
│  /api/health (GET)                                          │
│  └─ Health check for monitoring                             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 3. Instagram API Client

```
┌─────────────────────────────────────────────────────────────┐
│              Instagram API Client Library                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Class: InstagramApiClient                                  │
│                                                              │
│  Methods:                                                   │
│  ┌────────────────────────────────────────────┐            │
│  │ replyToComment(commentId, message)         │            │
│  │  → POST /{comment-id}/replies              │            │
│  └────────────────────────────────────────────┘            │
│                                                              │
│  ┌────────────────────────────────────────────┐            │
│  │ sendDm(recipientId, message)               │            │
│  │  → POST /me/messages                       │            │
│  └────────────────────────────────────────────┘            │
│                                                              │
│  ┌────────────────────────────────────────────┐            │
│  │ getPostDetails(postId)                     │            │
│  │  → GET /{media-id}                         │            │
│  └────────────────────────────────────────────┘            │
│                                                              │
│  ┌────────────────────────────────────────────┐            │
│  │ getCommentDetails(commentId)               │            │
│  │  → GET /{comment-id}                       │            │
│  └────────────────────────────────────────────┘            │
│                                                              │
│  Features:                                                  │
│  • Error handling                                           │
│  • Rate limit detection                                     │
│  • Exponential backoff retry                                │
│  • HMAC signature verification                              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 4. Database Schema

```
┌─────────────────────────────────────────────────────────────┐
│                   PostgreSQL Database                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────────────────────────────┐              │
│  │          config (1 row)                   │              │
│  ├──────────────────────────────────────────┤              │
│  │ id (PK)                                   │              │
│  │ instagram_access_token                   │              │
│  │ instagram_business_account_id            │              │
│  │ page_id                                   │              │
│  │ webhook_verify_token                     │              │
│  │ facebook_app_secret                       │              │
│  │ updated_at                                │              │
│  └──────────────────────────────────────────┘              │
│                                                              │
│  ┌──────────────────────────────────────────┐              │
│  │          campaigns (N rows)               │              │
│  ├──────────────────────────────────────────┤              │
│  │ id (PK)                                   │              │
│  │ post_id                                   │              │
│  │ post_thumbnail                            │              │
│  │ post_caption                              │              │
│  │ keywords (comma-separated)                │              │
│  │ comment_reply                             │              │
│  │ dm_message                                │              │
│  │ is_active                                 │              │
│  │ created_at                                │              │
│  │ updated_at                                │              │
│  └──────────────────────────────────────────┘              │
│                       │                                     │
│                       │ references                          │
│                       ▼                                     │
│  ┌──────────────────────────────────────────┐              │
│  │    processed_comments (N rows)            │              │
│  ├──────────────────────────────────────────┤              │
│  │ id (PK)                                   │              │
│  │ comment_id (UNIQUE)                       │              │
│  │ campaign_id (FK) ─────────────────────────┘              │
│  │ commenter_username                        │              │
│  │ processed_at                              │              │
│  └──────────────────────────────────────────┘              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Data Flow Diagrams

### Flow 1: Campaign Creation

```
User                  Frontend              API                Database          Instagram API
 │                       │                   │                    │                    │
 │  1. Click "New        │                   │                    │                    │
 │     Campaign"         │                   │                    │                    │
 ├──────────────────────►│                   │                    │                    │
 │                       │                   │                    │                    │
 │  2. Enter Post ID     │                   │                    │                    │
 ├──────────────────────►│                   │                    │                    │
 │                       │                   │                    │                    │
 │                       │  3. Fetch Preview │                    │                    │
 │                       ├──────────────────►│                    │                    │
 │                       │                   │                    │                    │
 │                       │                   │  4. Get Post       │                    │
 │                       │                   │     Details        │                    │
 │                       │                   ├────────────────────────────────────────►│
 │                       │                   │                    │                    │
 │                       │                   │  5. Return         │                    │
 │                       │                   │     Post Data      │                    │
 │                       │                   │◄────────────────────────────────────────┤
 │                       │                   │                    │                    │
 │                       │  6. Show Preview  │                    │                    │
 │                       │◄──────────────────┤                    │                    │
 │                       │                   │                    │                    │
 │  7. Fill form &       │                   │                    │                    │
 │     Click Create      │                   │                    │                    │
 ├──────────────────────►│                   │                    │                    │
 │                       │                   │                    │                    │
 │                       │  8. POST          │                    │                    │
 │                       │     Campaign      │                    │                    │
 │                       ├──────────────────►│                    │                    │
 │                       │                   │                    │                    │
 │                       │                   │  9. Insert         │                    │
 │                       │                   │     Campaign       │                    │
 │                       │                   ├───────────────────►│                    │
 │                       │                   │                    │                    │
 │                       │                   │ 10. Return ID      │                    │
 │                       │                   │◄───────────────────┤                    │
 │                       │                   │                    │                    │
 │                       │ 11. Success       │                    │                    │
 │                       │◄──────────────────┤                    │                    │
 │                       │                   │                    │                    │
 │ 12. Show Success      │                   │                    │                    │
 │◄──────────────────────┤                   │                    │                    │
```

### Flow 2: Comment Automation (Core Process)

```
Instagram         Facebook          Webhook           API               Database        Instagram API
   │              Webhook            Handler          Logic                │                 │
   │                 │                 │                │                  │                 │
   │  1. User        │                 │                │                  │                 │
   │     comments    │                 │                │                  │                 │
   │                 │                 │                │                  │                 │
   │  2. Send        │                 │                │                  │                 │
   │     webhook     │                 │                │                  │                 │
   ├────────────────►│                 │                │                  │                 │
   │                 │                 │                │                  │                 │
   │                 │  3. POST event  │                │                  │                 │
   │                 ├────────────────►│                │                  │                 │
   │                 │                 │                │                  │                 │
   │                 │                 │  4. Verify     │                  │                 │
   │                 │                 │     signature  │                  │                 │
   │                 │                 │                │                  │                 │
   │                 │                 │  5. Extract    │                  │                 │
   │                 │                 │     comment    │                  │                 │
   │                 │                 │     details    │                  │                 │
   │                 │                 │                │                  │                 │
   │                 │                 │  6. Check if   │                  │                 │
   │                 │                 │     processed  │                  │                 │
   │                 │                 ├────────────────────────────────►  │                 │
   │                 │                 │                │                  │                 │
   │                 │                 │  7. Not found  │                  │                 │
   │                 │                 │     (new)      │                  │                 │
   │                 │                 │◄────────────────────────────────  │                 │
   │                 │                 │                │                  │                 │
   │                 │                 │  8. Find       │                  │                 │
   │                 │                 │     matching   │                  │                 │
   │                 │                 │     campaign   │                  │                 │
   │                 │                 ├────────────────────────────────►  │                 │
   │                 │                 │                │                  │                 │
   │                 │                 │  9. Return     │                  │                 │
   │                 │                 │     campaign   │                  │                 │
   │                 │                 │◄────────────────────────────────  │                 │
   │                 │                 │                │                  │                 │
   │                 │                 │ 10. Check      │                  │                 │
   │                 │                 │     keywords   │                  │                 │
   │                 │                 │     match      │                  │                 │
   │                 │                 │                │                  │                 │
   │                 │                 │ 11. Get comment│                  │                 │
   │                 │                 │     author     │                  │                 │
   │                 │                 ├────────────────┼──────────────────────────────────►│
   │                 │                 │                │                  │                 │
   │                 │                 │ 12. Return     │                  │                 │
   │                 │                 │     user ID    │                  │                 │
   │                 │                 │◄───────────────┼──────────────────────────────────┤
   │                 │                 │                │                  │                 │
   │                 │                 │ 13. Reply to   │                  │                 │
   │                 │                 │     comment    │                  │                 │
   │                 │                 ├────────────────┼──────────────────────────────────►│
   │                 │                 │                │                  │                 │
   │                 │                 │ 14. Success    │                  │                 │
   │                 │                 │◄───────────────┼──────────────────────────────────┤
   │                 │                 │                │                  │                 │
   │                 │                 │ 15. Send DM    │                  │                 │
   │                 │                 ├────────────────┼──────────────────────────────────►│
   │                 │                 │                │                  │                 │
   │                 │                 │ 16. Success    │                  │                 │
   │                 │                 │     or Error   │                  │                 │
   │                 │                 │◄───────────────┼──────────────────────────────────┤
   │                 │                 │                │                  │                 │
   │                 │                 │ 17. Mark as    │                  │                 │
   │                 │                 │     processed  │                  │                 │
   │                 │                 ├────────────────────────────────►  │                 │
   │                 │                 │                │                  │                 │
   │                 │                 │ 18. Success    │                  │                 │
   │                 │                 │◄────────────────────────────────  │                 │
   │                 │                 │                │                  │                 │
   │                 │ 19. 200 OK      │                │                  │                 │
   │                 │◄────────────────┤                │                  │                 │
```

## Deployment Architecture

### Production Environment

```
┌─────────────────────────────────────────────────────────────────┐
│                        Cloud Platform                            │
│                   (Vercel/Railway/Render)                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              Next.js Application                          │  │
│  │                                                            │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │  │
│  │  │   Frontend   │  │   API Routes │  │   Webhooks   │   │  │
│  │  │  (Static)    │  │  (Serverless)│  │  (Functions) │   │  │
│  │  └──────────────┘  └──────────────┘  └──────────────┘   │  │
│  │                                                            │  │
│  └──────────────────────────────────────────────────────────┘  │
│                             │                                   │
│                             ▼                                   │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │         PostgreSQL Database (Managed)                     │  │
│  │  • config table                                           │  │
│  │  • campaigns table                                        │  │
│  │  • processed_comments table                               │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    External Services                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────────┐         ┌──────────────────────┐     │
│  │  Instagram Graph API │         │  Facebook Webhooks   │     │
│  │  • Comment replies   │◄────────┤  • Comment events    │     │
│  │  • DM sending        │         │  • Real-time notify  │     │
│  │  • Post details      │         │  • Retry logic       │     │
│  └──────────────────────┘         └──────────────────────┘     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Security Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                       Security Layers                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Layer 1: Transport Security                                    │
│  ┌────────────────────────────────────────────────────────┐    │
│  │  • HTTPS Only (TLS 1.2+)                               │    │
│  │  • SSL Certificate (Let's Encrypt / Platform managed)  │    │
│  │  • Secure WebSocket connections                         │    │
│  └────────────────────────────────────────────────────────┘    │
│                                                                  │
│  Layer 2: Request Validation                                    │
│  ┌────────────────────────────────────────────────────────┐    │
│  │  • Webhook Signature Verification (HMAC-SHA256)        │    │
│  │  • Input sanitization                                   │    │
│  │  • Rate limiting (Instagram API)                        │    │
│  └────────────────────────────────────────────────────────┘    │
│                                                                  │
│  Layer 3: Authentication & Authorization                        │
│  ┌────────────────────────────────────────────────────────┐    │
│  │  • Environment-based secrets                           │    │
│  │  • No client-side credential exposure                  │    │
│  │  • Secure token storage                                 │    │
│  └────────────────────────────────────────────────────────┘    │
│                                                                  │
│  Layer 4: Data Protection                                       │
│  ┌────────────────────────────────────────────────────────┐    │
│  │  • Database encryption at rest                          │    │
│  │  • Encrypted environment variables                      │    │
│  │  • Masked credentials in API responses                 │    │
│  └────────────────────────────────────────────────────────┘    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Scalability Considerations

### Current Architecture (v1.0)
- **Concurrent Users**: 100+ simultaneously
- **Comments/Hour**: ~150-200 (Instagram API limit)
- **Database**: PostgreSQL with connection pooling
- **Deployment**: Serverless functions (auto-scaling)

### Future Scaling Options (v2.0+)

```
┌─────────────────────────────────────────────────────────────────┐
│                  High-Volume Architecture                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐      ┌──────────────┐      ┌──────────────┐  │
│  │   Load       │──────►│  Next.js     │──────►│  Redis       │  │
│  │   Balancer   │      │  Instances   │      │  Cache       │  │
│  └──────────────┘      └──────────────┘      └──────────────┘  │
│                                │                                │
│                                ▼                                │
│                        ┌──────────────┐                         │
│                        │   Message    │                         │
│                        │   Queue      │                         │
│                        │  (Bull/SQS)  │                         │
│                        └──────────────┘                         │
│                                │                                │
│                                ▼                                │
│                        ┌──────────────┐                         │
│                        │   Workers    │                         │
│                        │  (Process    │                         │
│                        │   Comments)  │                         │
│                        └──────────────┘                         │
│                                │                                │
│                                ▼                                │
│                        ┌──────────────┐                         │
│                        │  PostgreSQL  │                         │
│                        │  (Primary +  │                         │
│                        │   Replicas)  │                         │
│                        └──────────────┘                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

**This architecture document provides a comprehensive overview of the system design. For implementation details, see the source code and API documentation.**

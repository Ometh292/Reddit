# Reddit Community Engagement Automation

A controlled Reddit community-engagement application designed to identify relevant discussions, prepare context-aware comment drafts, and assist with Reddit participation through the official Reddit API.

The system is intended for use by an authorized Reddit account and within predefined communities. It focuses on relevant, non-duplicative participation rather than indiscriminate or mass commenting.

---

## 🎯 Project Objective

The objective of this project is to streamline Reddit community engagement by:

- Identifying relevant Reddit discussions based on predefined topics and keywords
- Collecting the minimum information required to understand a discussion
- Preparing relevant comment drafts
- Providing a review/approval workflow before posting
- Preventing duplicate processing and repetitive comments
- Recording activity and outcomes
- Respecting subreddit-specific rules and Reddit platform policies

The application is designed to support a human-in-the-loop workflow rather than blindly posting automated comments.

---

## 🏗️ System Architecture

```text
                    ┌──────────────────────┐
                    │       Reddit API     │
                    │   Official OAuth     │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │   Backend Service    │
                    │      Node.js         │
                    └──────────┬───────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ▼                ▼                ▼
        Post Discovery    Relevance Check   Comment Draft
              │                │                │
              └────────────────┼────────────────┘
                               ▼
                    ┌──────────────────────┐
                    │      Supabase        │
                    │ Database + Logging   │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │   Review Interface   │
                    │   Human Approval     │
                    └──────────┬───────────┘
                               │
                         Approved?
                         /       \
                       Yes        No
                        │          │
                        ▼          ▼
                 Reddit API     SKIPPED
                  Comment
                        │
                        ▼
                    Activity Log
```

---

## 🔄 Workflow

### 1. Authentication

The application authenticates with Reddit using the official Reddit API and OAuth.

No Reddit account passwords are stored in the application.

---

### 2. Post Discovery

The application searches predefined communities for discussions matching configured topics or keywords.

Example topics may include:

```text
AI
automation
workflow automation
business automation
lead generation
CRM
```

The actual topics and subreddits should be configured according to the approved use case.

---

### 3. Relevance Filtering

Each discovered post is evaluated before any response is prepared.

Example criteria:

- Is the post related to the configured topic?
- Is the discussion appropriate for participation?
- Does the subreddit allow the intended type of participation?
- Has this post already been processed?
- Has a similar comment already been posted?

Irrelevant posts are marked as:

```text
SKIPPED
```

---

### 4. Comment Drafting

For relevant discussions, the system prepares a comment draft based on the context of the post.

The draft should:

- Directly address the discussion
- Provide useful information
- Avoid generic copy-paste responses
- Avoid repetitive promotional language
- Respect subreddit rules
- Avoid misleading claims

---

### 5. Human Review

Before posting, the generated comment enters a review state.

```text
PENDING
   ↓
REVIEW
   ↓
┌───────────┬───────────┐
│  APPROVE  │   SKIP    │
└─────┬─────┴───────────┘
      │
      ▼
   POSTED
```

A reviewer can:

- Approve the comment
- Edit the comment
- Reject/skip the comment

---

### 6. Comment Submission

After approval, the application submits the comment through the official Reddit API.

The application records:

- Reddit post ID
- Reddit post URL
- Account used
- Comment content
- Timestamp
- Result
- Comment URL when available

---

## 🗄️ Database

Supabase is used for persistent storage and activity tracking.

### `reddit_accounts`

| Field           | Description         |
| --------------- | ------------------- |
| id              | Internal identifier |
| reddit_username | Reddit account      |
| status          | Active/inactive     |
| created_at      | Creation timestamp  |

### `subreddits`

| Field       | Description                   |
| ----------- | ----------------------------- |
| id          | Internal identifier           |
| name        | Subreddit name                |
| enabled     | Whether processing is enabled |
| rules_notes | Internal notes                |
| created_at  | Creation timestamp            |

### `reddit_posts`

| Field          | Description         |
| -------------- | ------------------- |
| id             | Internal identifier |
| reddit_post_id | Reddit post ID      |
| subreddit      | Subreddit name      |
| title          | Post title          |
| body           | Post content        |
| url            | Reddit URL          |
| status         | Processing status   |
| created_at     | Discovery timestamp |

### `comment_tasks`

| Field         | Description         |
| ------------- | ------------------- |
| id            | Task ID             |
| post_id       | Related Reddit post |
| account_id    | Reddit account      |
| draft_comment | Generated draft     |
| final_comment | Approved version    |
| status        | Current status      |
| approved_at   | Approval timestamp  |
| posted_at     | Posting timestamp   |
| comment_url   | Posted comment URL  |
| error_message | Error information   |

---

## 📊 Status Flow

```text
DISCOVERED
    ↓
RELEVANCE_CHECK
    ↓
SKIPPED ───────────────┐
    │                  │
    ▼                  │
DRAFTED                │
    ↓                  │
REVIEW                 │
    │                  │
 ┌──┴───┐              │
 ▼      ▼              │
APPROVED  SKIPPED ─────┘
 │
 ▼
POSTING
 │
 ├── SUCCESS → POSTED
 │
 └── ERROR → FAILED
```

---

## 🔐 Security

Sensitive credentials must never be committed to GitHub.

Use environment variables:

```env
REDDIT_CLIENT_ID=your_client_id
REDDIT_CLIENT_SECRET=your_client_secret
REDDIT_REDIRECT_URI=your_redirect_uri
SUPABASE_URL=your_supabase_url
SUPABASE_ANON_KEY=your_supabase_key
```

Create a `.env.example` file containing placeholders only.

Add the following to `.gitignore`:

```gitignore
.env
.env.local
node_modules/
logs/
```

Never store:

- Reddit passwords
- OAuth secrets
- API keys
- Supabase service-role keys
- Dolphin credentials

inside the repository.

---

## 🧰 Technology Stack

### Backend

- Node.js
- Express.js

### Reddit

- Reddit Data API
- OAuth

### Database

- Supabase
- PostgreSQL

### Automation / Workflow

- n8n

### Development

- Git
- GitHub
- VS Code

---

## 📁 Suggested Project Structure

```text
reddit-community-engagement/
│
├── src/
│   ├── reddit/
│   │   ├── auth.js
│   │   ├── posts.js
│   │   ├── comments.js
│   │   └── subreddits.js
│   │
│   ├── database/
│   │   └── supabase.js
│   │
│   ├── services/
│   │   ├── relevance.js
│   │   ├── commentDraft.js
│   │   └── duplicateCheck.js
│   │
│   ├── routes/
│   │   ├── posts.js
│   │   └── comments.js
│   │
│   └── server.js
│
├── database/
│   └── schema.sql
│
├── .env.example
├── .gitignore
├── package.json
└── README.md
```

---

## 🚀 Initial Development Plan

### Phase 1 — Reddit API Authentication

- Create Reddit developer application
- Configure OAuth
- Test authentication
- Verify API access

### Phase 2 — Post Discovery

- Configure subreddit list
- Configure keywords
- Retrieve relevant posts
- Store posts in Supabase

### Phase 3 — Relevance Filtering

- Implement keyword/topic filtering
- Add duplicate detection
- Add subreddit allowlist

### Phase 4 — Comment Drafting

- Generate context-aware comment drafts
- Store drafts in Supabase
- Create review interface

### Phase 5 — Human Approval

- Approve/edit/reject comments
- Track reviewer and approval timestamp

### Phase 6 — Comment Posting

- Submit approved comments through Reddit API
- Verify result
- Store comment URL
- Update task status

### Phase 7 — n8n Integration

```text
Schedule
   ↓
Discover Posts
   ↓
Filter
   ↓
Store in Supabase
   ↓
Generate Draft
   ↓
Human Review
   ↓
Approved?
   ↓
Reddit API
   ↓
Log Result
```

---

## 🛡️ Compliance and Responsible Use

This application is intended to operate only within Reddit's approved API access and platform policies.

The system should:

- Use the official Reddit API
- Use OAuth authentication
- Operate only on authorized accounts
- Respect subreddit rules
- Respect Reddit API limits
- Avoid duplicate/repetitive comments
- Avoid indiscriminate posting
- Maintain appropriate logs
- Provide human oversight where required
- Stop when API or platform restrictions are encountered

The application must not:

- Circumvent Reddit security controls
- Bypass CAPTCHA
- Evade account restrictions
- Create fake engagement
- Spam communities
- Mass-post substantially identical comments
- Misrepresent the identity or affiliation of the account

---

## ⚠️ Product-Specific Restrictions

Before enabling a commercial use case, verify that the product, content, advertising activity, and intended subreddit participation are permitted under Reddit's current rules.

API approval does not automatically authorize prohibited transactions, prohibited products, spam, or other restricted activity.

---

## 🧪 Testing

Testing should initially be performed using:

1. A development Reddit account
2. A test subreddit where permitted
3. A small number of posts
4. Human approval for every comment

Example test:

```text
Reddit API
    ↓
Fetch one post
    ↓
Store in Supabase
    ↓
Generate comment draft
    ↓
Human approves
    ↓
Submit comment
    ↓
Verify comment
    ↓
Store result
```

---

## 📌 Current Status

```text
[ ] Reddit developer/API application
[ ] OAuth authentication
[ ] Reddit API connection
[ ] Subreddit configuration
[ ] Post discovery
[ ] Relevance filtering
[ ] Supabase database
[ ] Comment generation
[ ] Human review
[ ] Comment submission
[ ] Activity logging
[ ] n8n integration
```

---

## 🔮 Future Improvements

Potential future improvements include:

- Web-based review dashboard
- Advanced relevance scoring
- Better duplicate detection
- Subreddit-specific rules configuration
- Analytics dashboard
- Comment performance reporting
- Error monitoring
- Audit logs
- Role-based reviewer access

---

## 📄 License

This project is intended for authorized internal use. Any external deployment or commercial use should comply with Reddit's applicable developer/API terms and policies.

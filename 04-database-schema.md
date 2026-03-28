# Database Schema

## Overview

SQLite database with 5 main tables and relationships for storing Telegram group data.

## Table Definitions

### messages

Stores individual messages from the group.

```sql
CREATE TABLE messages (
    id INTEGER NOT NULL PRIMARY KEY,
    type TEXT NOT NULL,
    date TIMESTAMP NOT NULL,
    edit_date TIMESTAMP,
    content TEXT,
    reply_to INTEGER,
    user_id INTEGER,
    media_id INTEGER,
    topic_id INTEGER,
    is_deleted BOOLEAN DEFAULT 0,
    deleted_at TIMESTAMP,
    last_edit_date TIMESTAMP,
    edit_history TEXT,
    FOREIGN KEY(user_id) REFERENCES users(id),
    FOREIGN KEY(media_id) REFERENCES media(id)
);
```

| Column | Type | Purpose |
|--------|------|---------|
| `id` | INTEGER | Telegram message ID, unique identifier |
| `type` | TEXT | Message type: "text", "photo", "document", "poll", etc. |
| `date` | TIMESTAMP | When message was sent |
| `edit_date` | TIMESTAMP | When message was last edited (NULL if never edited) |
| `content` | TEXT | Message text content |
| `reply_to` | INTEGER | ID of parent message if this is a reply (NULL if not a reply) |
| `user_id` | INTEGER | Foreign key to users table |
| `media_id` | INTEGER | Foreign key to media table (NULL if no media) |
| `topic_id` | INTEGER | Forum topic ID (NULL if not in a topic) |
| `is_deleted` | BOOLEAN | Whether message was deleted |
| `deleted_at` | TIMESTAMP | When message was deleted |
| `last_edit_date` | TIMESTAMP | Latest edit timestamp |
| `edit_history` | TEXT | JSON array of edit snapshots |

---

### users

Stores user profiles and metadata.

```sql
CREATE TABLE users (
    id INTEGER NOT NULL PRIMARY KEY,
    username TEXT,
    first_name TEXT,
    last_name TEXT,
    tags TEXT,
    avatar TEXT,
    about TEXT,
    phone TEXT,
    photo_file TEXT,
    is_bot BOOLEAN DEFAULT 0,
    is_deleted BOOLEAN DEFAULT 0,
    deleted_display_name TEXT DEFAULT 'Deleted Account',
    last_updated TIMESTAMP DEFAULT NULL,
    raw_json TEXT,
    is_self BOOLEAN DEFAULT 0,
    contact BOOLEAN DEFAULT 0,
    mutual_contact BOOLEAN DEFAULT 0
);
```

| Column | Type | Purpose |
|--------|------|---------|
| `id` | INTEGER | Telegram user ID, unique identifier |
| `username` | TEXT | @username handle |
| `first_name` | TEXT | User's first name |
| `last_name` | TEXT | User's last name |
| `tags` | TEXT | Custom tags/labels (comma-separated) |
| `avatar` | TEXT | Avatar filename (relative path) |
| `about` | TEXT | User bio/about section |
| `phone` | TEXT | Phone number (if visible to self) |
| `photo_file` | TEXT | Avatar file path |
| `is_bot` | BOOLEAN | Whether user is a bot |
| `is_deleted` | BOOLEAN | Whether account is deleted |
| `deleted_display_name` | TEXT | Display name for deleted accounts |
| `last_updated` | TIMESTAMP | When profile was last fetched |
| `raw_json` | TEXT | Full user object as JSON string |
| `is_self` | BOOLEAN | Whether this is the logged-in account |
| `contact` | BOOLEAN | Whether user is in contacts |
| `mutual_contact` | BOOLEAN | Whether mutual contact |

---

### media

Stores media metadata and references.

```sql
CREATE TABLE media (
    id INTEGER NOT NULL PRIMARY KEY,
    type TEXT,
    url TEXT,
    title TEXT,
    description TEXT,
    thumb TEXT,
    md5 TEXT
);
```

| Column | Type | Purpose |
|--------|------|---------|
| `id` | INTEGER | Media ID, unique identifier |
| `type` | TEXT | Media type: "photo", "document", "video", "webpage", "poll", "sticker_pack" |
| `url` | TEXT | URL or local file path |
| `title` | TEXT | Media title or filename |
| `description` | TEXT | Media description or metadata |
| `thumb` | TEXT | Thumbnail file path |
| `md5` | TEXT | MD5 hash for deduplication |

**Media Types**:
- `"photo"`: Downloaded image/document/video
- `"webpage"`: Inline web link preview
- `"poll"`: Poll result
- `"sticker_pack"`: Sticker or custom emoji pack share link

---

### topics

Stores Telegram forum topics/channels.

```sql
CREATE TABLE topics (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    chat_id INTEGER NOT NULL,
    topic_id INTEGER NOT NULL,
    title TEXT NOT NULL,
    icon_emoji TEXT,
    icon_color INTEGER,
    created_date TIMESTAMP,
    UNIQUE(chat_id, topic_id)
);
```

| Column | Type | Purpose |
|--------|------|---------|
| `id` | INTEGER | Auto-increment primary key |
| `chat_id` | INTEGER | Group/channel ID |
| `topic_id` | INTEGER | Topic ID within the group |
| `title` | TEXT | Topic name/title |
| `icon_emoji` | TEXT | Emoji ID for topic icon |
| `icon_color` | INTEGER | Color code for topic |
| `created_date` | TIMESTAMP | When topic was created |

**Constraint**: `UNIQUE(chat_id, topic_id)` ensures one record per topic per group.

---

### self_privacy

Stores privacy settings snapshots for the logged-in account.

```sql
CREATE TABLE self_privacy (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL,
    snapshot_at TIMESTAMP NOT NULL,
    phone_visibility TEXT,
    last_seen_visibility TEXT,
    profile_photo_visibility TEXT,
    bio_visibility TEXT,
    forwards_visibility TEXT,
    calls_visibility TEXT,
    add_by_phone TEXT,
    has_fallback_photo BOOLEAN,
    read_dates_private BOOLEAN,
    contact_require_premium BOOLEAN,
    phone_calls_private BOOLEAN,
    FOREIGN KEY(user_id) REFERENCES users(id)
);
```

| Column | Type | Purpose |
|--------|------|---------|
| `id` | INTEGER | Auto-increment primary key |
| `user_id` | INTEGER | Foreign key to users (self-account) |
| `snapshot_at` | TIMESTAMP | When this snapshot was taken |
| `phone_visibility` | TEXT | Who can see phone: "everybody", "contacts", "nobody", "unknown" |
| `last_seen_visibility` | TEXT | Who can see last seen time |
| `profile_photo_visibility` | TEXT | Who can see profile photo |
| `bio_visibility` | TEXT | Who can see bio |
| `forwards_visibility` | TEXT | Who can see message forwards |
| `calls_visibility` | TEXT | Who can see call history |
| `add_by_phone` | TEXT | Who can add by phone |
| `has_fallback_photo` | BOOLEAN | Has fallback profile photo |
| `read_dates_private` | BOOLEAN | Read receipts disabled |
| `contact_require_premium` | BOOLEAN | Requires premium to contact |
| `phone_calls_private` | BOOLEAN | Phone calls disabled |

---

## Relationships

```
users (1) ──────────────── (N) messages
  ↑                            ↓
  │                        media (1)
  │                            ↑
  └────────────────────────────┘

topics (1) ──────────────── (N) messages

self_privacy (N) ──────────── (1) users
```

## Indexes

The database creates a custom SQL function:

```sql
CREATE FUNCTION PAGE(n, multiple)
  RETURNS INTEGER
  DETERMINISTIC
  LANGUAGE SQL
  AS SELECT CEIL(n / multiple)
```

This calculates page numbers for pagination queries.

## Migration Notes

The schema includes migration logic for backward compatibility:

1. **topic_id column**: Added to messages table if missing
2. **md5 column**: Added to media table if missing
3. **Message tracking columns**: Added if missing:
   - `is_deleted`
   - `deleted_at`
   - `last_edit_date`
   - `edit_history`

Migrations run silently on DB initialization if columns already exist.

## Query Patterns

### Get last synced message
```sql
SELECT id, date FROM messages
ORDER BY id DESC LIMIT 1
```

### Get messages for a page
```sql
SELECT * FROM messages
WHERE id > ?
ORDER BY id ASC
LIMIT ?
```

### Get messages by user
```sql
SELECT m.* FROM messages m
WHERE m.user_id = ?
ORDER BY m.id ASC
```

### Get messages by topic
```sql
SELECT * FROM messages
WHERE topic_id = ?
ORDER BY date DESC
```

### Get timeline (months)
```sql
SELECT
  strftime('%Y-%m', date) as month,
  COUNT(*) as count
FROM messages
GROUP BY month
ORDER BY month DESC
```

### Get dayline (days in month)
```sql
SELECT
  strftime('%Y-%m-%d', date) as day,
  COUNT(*) as count
FROM messages
WHERE strftime('%Y-%m', date) = ?
GROUP BY day
ORDER BY day ASC
```

### Get reply relationships
```sql
SELECT * FROM messages
WHERE reply_to IN (?, ?, ...)
```

### Get edit history
```sql
SELECT edit_history FROM messages
WHERE id = ? AND edit_history IS NOT NULL
```

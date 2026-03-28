# Core Modules

## __init__.py

**Entry point for CLI and orchestration**

### Main Components

- **`main()`**: CLI argument parser and command dispatcher
- **`get_config(path)`**: Load YAML config and merge with defaults
- **`_CONFIG`**: Default configuration dictionary

### CLI Commands

```python
# Initialize new site
tgae --new --path=mysite

# Sync messages
tgae --sync [--id MSG_IDS] [--from-id ID]

# Build static site
tgae --build [--template FILE] [--rss-template FILE] [--symlink]

# Export member archives
tgae --export [--export-dir DIR]
```

### Key Functions

- **`main()`**: Parses arguments and routes to appropriate handler
- **`get_config(path)`**: Loads YAML and merges with defaults
- **Sync handler**: Creates DB, initializes Sync, calls sync()
- **Build handler**: Creates DB, initializes Build, calls build()
- **Export handler**: Creates DB, initializes Export, calls export_all()

---

## sync.py

**Fetches messages from Telegram API and populates database**

### Main Class: `Sync`

```python
class Sync:
    def __init__(self, config, session_file, db)
    def sync(self, ids=None, from_id=None)
```

### Key Methods

- **`sync()`**: Main sync loop
  - Determines if group is a forum
  - Fetches topics if forum
  - Iterates through messages
  - Handles edits and deletions
  - Downloads media and avatars

- **`_get_group_id()`**: Resolve group name to ID

- **`_fetch_user()`**: Get full user profile with privacy settings

- **`_download_media()`**: Download and store media files

- **`_download_avatar()`**: Download user avatar

- **`_process_message()`**: Extract message data and store in DB

### Data Structures

- **`user_cache`**: In-memory cache of fetched users
- **`_sticker_pack_cache`**: Cache of sticker pack metadata
- **`_privacy_inspector`**: SelfPrivacyInspector instance

### Telegram API Integration

- Uses Telethon library for user account API
- Handles forum topics via `GetForumTopicsRequest`
- Fetches user privacy via `GetPrivacyRequest`
- Downloads media via Telethon's built-in methods

---

## build.py

**Generates static HTML site from database**

### Main Class: `Build`

```python
class Build:
    def __init__(self, config, db, symlink)
    def build(self)
```

### Build Modes

**Date Mode** (default):
- Organizes pages by year/month/day
- Creates index pages for each time period
- Supports deep linking via message ID

**Batch Mode** (`paginate_by_batch: true`):
- Creates flat numbered pages
- Ignores calendar structure
- Useful for large archives

### Key Methods

- **`build()`**: Main entry point, routes to date or batch mode

- **`_build_date_mode()`**: Generate date-organized pages
  - Creates year/month/day indexes
  - Paginates messages by date
  - Generates dayline for each page

- **`_build_batch_mode()`**: Generate batch pages
  - Creates numbered pages
  - Maintains fake "batch timeline"
  - Supports same pagination as date mode

- **`_render_page()`**: Render single page with Jinja2

- **`_generate_rss_feed()`**: Create RSS/Atom feed

- **`_generate_members_page()`**: Create member directory

- **`_generate_profile_page()`**: Create individual user profile

### Template Context

Pages receive context with:
- `messages`: List of Message objects for page
- `timeline`: OrderedDict of months/batches
- `dayline`: List of days with message counts
- `page_ids`: Map of message ID to page slug
- `config`: Configuration dictionary
- `site_url`: Base URL for links

### Media Handling

- Copies or symlinks media files to publish directory
- Generates thumbnails for images
- Handles missing media gracefully

---

## db.py

**SQLite database schema and data access layer**

### Schema

**messages table**:
- `id`: Message ID (primary key)
- `type`: Message type (text, photo, etc.)
- `date`: Message timestamp
- `edit_date`: When message was edited
- `content`: Message text
- `reply_to`: ID of parent message
- `user_id`: Foreign key to users
- `media_id`: Foreign key to media
- `topic_id`: Forum topic ID
- `is_deleted`: Deletion flag
- `deleted_at`: Deletion timestamp
- `last_edit_date`: Latest edit timestamp
- `edit_history`: JSON array of edits

**users table**:
- `id`: User ID (primary key)
- `username`: @username
- `first_name`, `last_name`: Name fields
- `tags`: User tags/labels
- `avatar`: Avatar filename
- `about`: Bio/about text
- `phone`: Phone number (if visible)
- `photo_file`: Avatar file path
- `is_bot`: Bot flag
- `is_deleted`: Deleted account flag
- `deleted_display_name`: Display name for deleted accounts
- `last_updated`: Last profile update
- `raw_json`: Full user object as JSON
- `is_self`: Self-account flag
- `contact`, `mutual_contact`: Contact flags

**media table**:
- `id`: Media ID (primary key)
- `type`: Media type (photo, document, webpage, poll, sticker_pack)
- `url`: Media URL or file path
- `title`: Media title
- `description`: Media description
- `thumb`: Thumbnail path
- `md5`: MD5 hash for deduplication

**topics table**:
- `id`: Auto-increment primary key
- `chat_id`: Group/channel ID
- `topic_id`: Topic ID
- `title`: Topic name
- `icon_emoji`: Emoji ID for topic icon
- `icon_color`: Color code for topic
- `created_date`: Topic creation timestamp

**self_privacy table**:
- Stores privacy settings snapshot for self-account
- Fields: phone_visibility, last_seen_visibility, etc.
- One record per sync (timestamped)

### Named Tuples

- **`User`**: User record with all fields
- **`Message`**: Message record with user and media objects
- **`Media`**: Media record
- **`Month`**: Timeline entry (date, slug, label, count)
- **`Day`**: Day entry (date, slug, label, count, page)
- **`SelfPrivacyRecord`**: Privacy settings snapshot

### Main Class: `DB`

```python
class DB:
    def __init__(self, dbfile, tz=None)
```

### Key Query Methods

- **`get_last_message_id()`**: Get ID and date of last synced message
- **`get_all_message_ids()`**: Get set of all message IDs
- **`get_all_messages(last_id, limit)`**: Paginated message retrieval
- **`get_messages_by_user(user_id)`**: Get all messages from user
- **`get_message_by_id_full(msg_id)`**: Get full message with user/media
- **`get_topics()`**: Get all topics
- **`get_topic_title(topic_id)`**: Get topic name
- **`get_dayline_for_messages(msg_ids)`**: Get day breakdown
- **`get_timeline()`**: Get month/year breakdown
- **`get_users()`**: Get all users
- **`get_reply_children_full(msg_ids)`**: Get messages replying to given IDs

### Insert Methods

- **`insert_message()`**: Store message
- **`insert_user()`**: Store user
- **`insert_media()`**: Store media
- **`insert_topic()`**: Store topic
- **`insert_self_privacy()`**: Store privacy snapshot

### Utility Methods

- **`get_total_message_count()`**: Total messages in DB
- **`get_latest_self_privacy(user_id)`**: Get latest privacy snapshot

---

## privacy.py

**Detects and stores privacy settings of self-account**

### Main Class: `SelfPrivacyInspector`

```python
class SelfPrivacyInspector:
    def __init__(self, client)
    def fetch(self) -> SelfPrivacySnapshot
```

### Privacy Settings Tracked

- `phone_visibility`: Who can see phone number
- `last_seen_visibility`: Who can see last seen time
- `profile_photo_visibility`: Who can see profile photo
- `bio_visibility`: Who can see bio/about
- `forwards_visibility`: Who can see message forwards
- `calls_visibility`: Who can see call history
- `add_by_phone`: Who can add by phone number
- `has_fallback_photo`: Has fallback profile photo
- `read_dates_private`: Read receipts disabled
- `contact_require_premium`: Requires premium to contact
- `phone_calls_private`: Phone calls disabled

### Privacy Values

- `"everybody"`: PrivacyValueAllowAll
- `"contacts"`: PrivacyValueAllowContacts
- `"nobody"`: PrivacyValueDisallowAll
- `"unknown"`: Failed to fetch or not supported

### Data Class: `SelfPrivacySnapshot`

Immutable snapshot of privacy settings with timestamp.

---

## export.py

**Creates per-member ZIP archives with messages and profiles**

### Main Class: `Export`

```python
class Export:
    def __init__(self, config, db, out_dir)
    def export_all(self)
```

### Export Format

Each member gets a ZIP file: `member_{user_id}.zip`

**Contents**:
- `profile.json`: User profile metadata
- `messages.ndjson`: Newline-delimited JSON of messages
- `avatar.{ext}`: User avatar (if available)

### Key Methods

- **`export_all()`**: Iterate through users and export each

- **`_export_member(user)`**: Create ZIP for single user
  - Collects all messages by user
  - Builds profile JSON
  - Includes avatar if available
  - Creates NDJSON message file

- **`_message_to_export_dict(msg)`**: Convert message to export format
  - Includes reply relationships
  - Includes topic information
  - Includes edit history
  - Includes deletion info

### Export Data Structure

**profile.json**:
```json
{
  "id": 123456,
  "username": "username",
  "first_name": "First",
  "last_name": "Last",
  "about": "Bio text",
  "phone": "+1234567890",
  "avatar": "avatar.jpg",
  "tags": "tag1,tag2",
  "is_bot": false,
  "is_deleted": false,
  "is_self": false,
  "contact": true,
  "mutual_contact": false,
  "last_updated": "2026-03-29T12:00:00",
  "raw_json": {...}
}
```

**messages.ndjson** (one JSON object per line):
```json
{
  "id": 999,
  "type": "text",
  "date": "2026-03-29T12:00:00",
  "content": "Message text",
  "topic_id": 1,
  "topic_title": "General",
  "effective_reply_to": 998,
  "reply_to_message": {...},
  "replied_by": [...],
  "is_deleted": false,
  "edit_history": [...],
  "user": {...},
  "media": {...}
}
```

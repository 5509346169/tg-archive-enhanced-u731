# Architecture

## System Design

```
┌─────────────────────────────────────────────────────────────┐
│                    Telegram API (Telethon)                  │
└────────────────────────┬────────────────────────────────────┘
                         │
                    ┌────▼─────┐
                    │   Sync    │  Fetch messages, users, topics
                    └────┬─────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
    ┌───▼──┐         ┌───▼──┐        ┌───▼──┐
    │Users │         │Topics│        │Media │
    └──────┘         └──────┘        └──────┘
        │                │                │
        └────────────────┼────────────────┘
                         │
                    ┌────▼──────┐
                    │  SQLite   │  Local database
                    │ data.db   │
                    └────┬──────┘
                         │
                    ┌────▼─────┐
                    │  Build    │  Generate HTML
                    └────┬─────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
    ┌───▼──────┐    ┌───▼──────┐    ┌───▼──────┐
    │HTML Pages│    │RSS/Atom  │    │Static    │
    │(Jinja2)  │    │Feed      │    │Assets    │
    └──────────┘    └──────────┘    └──────────┘
        │                │                │
        └────────────────┼────────────────┘
                         │
                    ┌────▼──────┐
                    │  site/    ��  Static website
                    │ directory │
                    └───────────┘
```

## Data Flow

### Sync Phase
1. **Authentication**: Load or create Telegram session
2. **Topic Discovery**: Fetch forum topics if group is a forum
3. **Message Fetching**: Iterate through messages from last synced ID
4. **User Enrichment**: Fetch full user profiles and privacy settings
5. **Media Download**: Download avatars and media based on config
6. **Database Insert**: Store messages, users, media, topics in SQLite

### Build Phase
1. **Template Loading**: Load Jinja2 templates
2. **Message Retrieval**: Query database for all messages
3. **Page Generation**: Create HTML pages (by date or batch)
4. **Link Resolution**: Map message IDs to page slugs for cross-linking
5. **Feed Generation**: Create RSS/Atom feeds
6. **Asset Copying**: Copy static files and media to publish directory

### Export Phase
1. **User Iteration**: Loop through all users
2. **Message Collection**: Get all messages by user
3. **ZIP Creation**: Create per-member ZIP with profile and messages
4. **Avatar Inclusion**: Include user avatar if available

## Module Responsibilities

| Module | Purpose |
|--------|---------|
| `__init__.py` | CLI entry point, argument parsing, orchestration |
| `sync.py` | Telegram API interaction, message fetching, user enrichment |
| `build.py` | HTML generation, pagination, feed creation |
| `export.py` | Per-member ZIP archive creation |
| `db.py` | SQLite schema, queries, data access layer |
| `privacy.py` | Self-account privacy settings detection |

## Key Design Patterns

### Incremental Sync
- Tracks last synced message ID
- Only fetches new messages on subsequent runs
- Can be interrupted and resumed
- Handles message edits and deletions

### Pagination
- **Date Mode**: Pages organized by year/month/day
- **Batch Mode**: Flat numbered pages with configurable size
- Both modes support deep linking via message ID mapping

### Template-Driven Generation
- Single Jinja2 template for flexibility
- Template receives context with messages, timeline, metadata
- Customizable without code changes

### Privacy-Aware
- Tracks user privacy settings
- Handles deleted accounts and deleted messages
- Respects message deletion timestamps
- Stores edit history for transparency

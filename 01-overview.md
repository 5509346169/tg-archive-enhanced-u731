# Project Overview

## What is tg-archive-enhanced-u731?

A tool for exporting Telegram group chats into static websites, preserving chat history like mailing list archives. This is an actively maintained fork of the original tg-archive project with enhanced support for Telegram Topics/Forums.

## Core Purpose

Convert Telegram group messages into a searchable, browsable static HTML archive that can be published anywhere without requiring a server.

## Key Features

### Message Archiving
- Periodically sync Telegram group messages to local SQLite database
- Download only new messages since last sync (incremental)
- Support for Telegram Topics/Forums with topic-aware organization
- Track message edits and deletion history
- Handle replies with cross-page linking

### Media Handling
- Download user avatars locally (configurable size)
- Download and embed media files (photos, documents, videos)
- Render poll results
- Use emoji alternatives for stickers
- Configurable media filtering by MIME type and file extension

### Site Generation
- Single-file Jinja2 HTML template for customization
- Year/Month/Day indexes with deep linking
- "In reply to" links to parent messages across pages
- RSS/Atom feed of recent messages
- Batch mode pagination or date-based pagination

### User Management
- Member directory with user profiles
- User privacy settings tracking
- Edit history per message
- Deleted message handling with timestamps
- Per-member export archives (ZIP files)

## Workflow

```
1. Initialize new site
   tgae --new --path=mysite

2. Configure Telegram credentials
   Edit config.yaml with API credentials

3. Sync messages from Telegram
   tgae --sync
   (Creates session.session on first run)

4. Build static site
   tgae --build
   (Generates HTML in site/ directory)

5. (Optional) Export per-member archives
   tgae --export
   (Creates ZIP files in export/ directory)

6. Publish site anywhere
   (Copy site/ directory to web server)
```

## Use Cases

- **Community Archives**: Preserve public group discussions
- **Documentation**: Archive important conversations for reference
- **Compliance**: Maintain searchable records of group communications
- **Offline Access**: Browse archives without internet connection
- **Data Portability**: Export member data as ZIP archives

## Limitations

- Requires Telegram user account API credentials (not bot API)
- Subject to Telegram API rate limits for large groups
- First sync of large groups may take time
- Media downloads depend on available disk space

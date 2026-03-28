# tg-archive-enhanced-u731 Wiki

Complete documentation for the Telegram Group Archive tool with Topics/Forums support.

## Quick Navigation

- **[Project Overview](./01-overview.md)** — What this project does and key features
- **[Architecture](./02-architecture.md)** — System design and data flow
- **[Core Modules](./03-core-modules.md)** — Detailed breakdown of each Python module
- **[Database Schema](./04-database-schema.md)** — SQLite structure and relationships
- **[CLI Commands](./05-cli-commands.md)** — Usage and command reference
- **[Configuration](./06-configuration.md)** — Config options and setup
- **[Development](./07-development.md)** — Building, testing, and extending

## Key Concepts

**Sync** → Fetch messages from Telegram API into SQLite database
**Build** → Generate static HTML site from database
**Export** → Create per-member ZIP archives with messages and profiles
**Topics** → Telegram Forums/Topics support for organized group chats

## Tech Stack

- **Language**: Python 3.13+
- **Telegram API**: Telethon (user account API)
- **Database**: SQLite3
- **Templating**: Jinja2
- **Feed Generation**: feedgen (RSS/Atom)
- **Media Processing**: Pillow, python-magic

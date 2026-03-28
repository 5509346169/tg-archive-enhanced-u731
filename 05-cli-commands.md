# CLI Commands

## Command: `tgae`

The main CLI command (also `tg-archive-enhanced-u731`).

```bash
tgae [global options] <command> [command options]
```

## Global Options

| Flag | Default | Description |
|------|---------|-------------|
| `-c, --config` | `config.yaml` | Path to config file |
| `-d, --data` | `data.sqlite` | Path to SQLite database file |
| `-se, --session` | `session.session` | Path to Telegram session file |
| `-v, --version` | — | Show version number and exit |

---

## Command: `--new`

Initialize a new site with template files.

```bash
tgae --new --path=mysite
```

| Flag | Default | Description |
|------|---------|-------------|
| `-n, --new` | — | Create new site (required) |
| `-p, --path` | `example` | Directory to create new site in |

**What it does**:
- Copies example directory from package to specified path
- Sets appropriate file permissions (755 for dirs, 644 for files)
- Creates config.yaml, template.html, static assets

**After running**:
1. `cd mysite`
2. Edit `config.yaml` with your credentials and settings

---

## Command: `--sync`

Sync messages from Telegram to local database.

```bash
tgae --sync
tgae --sync --id 12345
tgae --sync --id 12345 98765 111
tgae --sync --from-id 12345
```

| Flag | Default | Description |
|------|---------|-------------|
| `-s, --sync` | — | Run sync (required) |
| `-id, --id` | — | Sync specific message IDs |
| `-from-id, --from-id` | — | Sync all messages from this ID onward |

**Behavior**:
- Without `--id` or `--from-id`: resumes from last synced message
- With `--id`: fetches or updates specific message IDs
- With `--from-id`: syncs all messages from given ID to latest
- Can be interrupted with Ctrl+C and resumed later
- Creates `session.session` on first run (requires phone auth)

**First Run**:
- Prompts for phone number
- Sends auth code to Telegram app
- Creates session file

**Output**:
```
2026-03-29 12:00:00: starting Telegram sync (batch_size=2000, limit=0, wait=5, mode=standard)
2026-03-29 12:00:00: fetching from last message id=0 (None)
2026-03-29 12:00:00: synced 2000 messages...
```

---

## Command: `--build`

Build the static HTML site from database.

```bash
tgae --build
tgae --build --template path/to/template.html
tgae --build --rss-template path/to/rss_template.html
tgae --build --symlink
```

| Flag | Default | Description |
|------|---------|-------------|
| `-b, --build` | — | Run build (required) |
| `-t, --template` | `template.html` | Path to Jinja2 template |
| `--rss-template` | — | Path to RSS template |
| `--symlink` | — | Symlink instead of copy media/static files |

**What it produces**:
- `site/index.html`: Main archive index
- `site/YYYY-MM.html`: Monthly pages
- `site/index.atom` and `site/index.xml`: RSS/Atom feeds
- `site/static/`: Static assets (CSS, JS, etc.)
- `site/media/`: Media files (or symlinks)

**--symlink flag**: Useful during development — instead of copying files, creates symlinks (saves disk space and speeds up rebuilds).

---

## Command: `--export`

Export per-member archives.

```bash
tgae --export
tgae --export --export-dir /path/to/export
```

| Flag | Default | Description |
|------|---------|-------------|
| `-e, --export` | — | Run export (required) |
| `--export-dir` | `export` | Directory for ZIP files |

**Output format**: One ZIP file per user:
```
export/
  member_123456.zip
  member_789012.zip
```

Each ZIP contains:
- `profile.json`: User metadata
- `messages.ndjson`: All messages by user
- `avatar.jpg` (or `.png`): Avatar if available

---

## Examples

### Initial Setup

```bash
# Create new site
tgae --new --path=my-archive
cd my-archive

# Edit config
nano config.yaml  # Set api_id, api_hash, group

# Sync (first time, will prompt for auth)
tgae --sync

# Build
tgae --build
```

### Scheduled Updates

```bash
# Sync and rebuild (cron job)
cd /path/to/my-archive && tgae --sync && tgae --build
```

### Custom Config and Data Paths

```bash
tgae --config custom.yaml --data custom.sqlite --build
```

### Selective Sync

```bash
# Update specific messages
tgae --sync --id 12345 67890

# Sync from a specific point
tgae --sync --from-id 50000
```

### Development

```bash
# Build with symlinks (faster iteration)
tgae --build --symlink
```

### Export Member Data

```bash
tgae --export --export-dir /path/to/exports
```

---

## Error Handling

| Error | Cause | Fix |
|-------|-------|-----|
| "unable to find bundled example directory" | Package not installed correctly | Reinstall package |
| "the directory 'X' already exists" | Path conflict in --new | Use different path |
| "pass either --id or --from-id but not both" | Conflicting flags | Use one or neither |
| Telegram API errors | Rate limiting or auth issues | Wait and retry |
| "sync cancelled manually" | Ctrl+C during sync | Run sync again to resume |

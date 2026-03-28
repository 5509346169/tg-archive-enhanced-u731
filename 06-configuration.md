# Configuration

## Config File Format

The configuration file is YAML format, typically named `config.yaml`.

## All Configuration Options

### Telegram API Authentication

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `api_id` | string | `$API_ID` env var | Telegram API ID from my.telegram.org |
| `api_hash` | string | `$API_HASH` env var | Telegram API hash from my.telegram.org |
| `group` | string | `""` | Group username or ID to archive |

**Obtaining API credentials**:
1. Go to https://my.telegram.org/auth
2. Select "API development tools"
3. Create new application to get `api_id` and `api_hash`

Note: If the page shows "ERROR", disconnect from VPN/proxy and try a different browser.

---

### Sync Settings

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `fetch_batch_size` | int | `2000` | Number of messages to fetch per batch |
| `fetch_wait` | int | `5` | Seconds to wait between batches |
| `fetch_limit` | int | `0` | Max messages to fetch (0 = no limit) |
| `use_takeout` | bool | `false` | Use Telegram takeout mode for export |

**Batch size considerations**:
- Larger batch = fewer API calls but more memory
- Smaller batch = more API calls but lower memory
- Telegram rate limits may require smaller batches for large groups

---

### Avatar Settings

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `download_avatars` | bool | `true` | Whether to download user avatars |
| `avatar_size` | list | `[64, 64]` | Avatar thumbnail size in pixels |

---

### Media Settings

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `download_media` | bool | `false` | Whether to download media files |
| `media_dir` | string | `"media"` | Directory for downloaded media |
| `media_mime_types` | list | `[]` | Whitelist of MIME types to download |
| `skip_sticker_pack_media` | bool | `false` | Skip sticker pack media |
| `sticker_pack_link_text` | string | `"Sticker pack"` | Text for sticker pack links |
| `media_max_size` | int | `0` | Max file size in bytes (0 = no limit) |
| `media_deduplicate` | bool | `true` | Deduplicate media by MD5 hash |
| `media_exclude_mime_types` | list | `[]` | MIME types to skip (blacklist) |
| `media_exclude_extensions` | list | `[]` | File extensions to skip |

**Media whitelist example**:
```yaml
media_mime_types:
  - "image/jpeg"
  - "image/png"
  - "image/gif"
```

**Media blacklist example**:
```yaml
media_exclude_mime_types:
  - "video/mp4"
  - "application/zip"
media_exclude_extensions:
  - "exe"
  - "apk"
```

---

### Privacy Settings

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `fetch_self_privacy` | bool | `true` | Detect and store self-account privacy settings |

---

### Proxy Settings

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `proxy.enable` | bool | `false` | Enable proxy |

---

### Build Settings

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `publish_dir` | string | `"site"` | Output directory for built site |
| `static_dir` | string | `"static"` | Source static assets directory |
| `per_page` | int | `1000` | Messages per page |
| `paginate_by_batch` | bool | `false` | Use batch pagination instead of date-based |

**Pagination modes**:
- **Date mode** (default): Pages named `YYYY-MM.html`, organized by month
- **Batch mode**: Pages named `batch-0001.html`, flat numbered pagination

---

### RSS Feed Settings

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `publish_rss_feed` | bool | `true` | Generate RSS/Atom feeds |
| `rss_feed_entries` | int | `100` | Number of entries in RSS feed |

---

### Site Metadata

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `site_url` | string | `"https://mysite.com"` | Base URL for the site |
| `site_name` | string | `"@{group} (Telegram) archive"` | Site name |
| `site_description` | string | `"Public archive of @{group} Telegram messages."` | Site description |
| `meta_description` | string | `"@{group} {date} Telegram message archive."` | Per-page meta description template |
| `page_title` | string | `"{date} - @{group} Telegram message archive."` | Per-page title template |
| `telegram_url` | string | `"https://t.me/{id}"` | Template for Telegram group links |

**Template variables**: `{group}`, `{date}`, `{id}` are replaced with actual values.

---

### Display Settings

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `show_sender_fullname` | bool | `false` | Show full name instead of @username |
| `timezone` | string | `""` | Timezone for timestamps (e.g., "America/New_York") |

**Timezone examples**: `"UTC"`, `"US/Eastern"`, `"Asia/Kolkata"`, `"Europe/London"`

---

## Example config.yaml

```yaml
# Telegram API credentials
api_id: "12345678"
api_hash: "abcdef1234567890abcdef1234567890"

# Target group to archive
group: "mygroupname"

# Sync settings
fetch_batch_size: 2000
fetch_wait: 5
fetch_limit: 0

# Avatar settings
download_avatars: true
avatar_size: [64, 64]

# Media settings
download_media: true
media_dir: "media"
media_deduplicate: true
media_max_size: 50000000  # 50MB max
media_exclude_mime_types:
  - "video/mp4"
  - "application/zip"

# Privacy settings
fetch_self_privacy: true

# Build settings
publish_dir: "site"
static_dir: "static"
per_page: 1000

# RSS feed
publish_rss_feed: true
rss_feed_entries: 100

# Site metadata
site_url: "https://myarchive.example.com"
site_name: "My Telegram Archive"
site_description: "Archive of @mygroup Telegram messages"
timezone: "US/Eastern"
show_sender_fullname: false
```

## Environment Variables

For security, API credentials can be set via environment variables:

```bash
export API_ID="12345678"
export API_HASH="abcdef1234567890abcdef1234567890"
tgae --sync
```

## Security Notes

- **session.session**: Contains API auth — never commit or share
- **api_hash**: Treat as a secret, use environment variables
- **config.yaml**: May contain credentials — add to .gitignore
- **media/**: May contain sensitive images/files

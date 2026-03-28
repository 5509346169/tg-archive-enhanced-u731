# Development

## Prerequisites

- Python 3.12+ (tested with 3.13.2)
- uv (recommended) or pip
- SQLite3

## Setup

### Install with uv (recommended)

```bash
# Install package and dependencies
uv pip install tg-archive-enhanced-u731

# Or install from source in dev mode
git clone https://github.com/your-username/tg-archive-enhanced-u731
cd tg-archive-enhanced-u731
uv pip install -e "."
```

### Install with pip

```bash
pip install tg-archive-enhanced-u731
```

## Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| `telethon` | ~1.39 | Telegram API client |
| `jinja2` | ~3.1 | HTML template rendering |
| `feedgen` | ~1.0 | RSS/Atom feed generation |
| `pillow` | ~11.1 | Image processing (avatars) |
| `python-magic` | ~0.4 | MIME type detection |
| `pytz` | ~2025.1 | Timezone handling |
| `pyyaml` | ~6.0 | YAML config parsing |

## Project Structure

```
tg-archive-enhanced-u731/
├── tg_archive_enhanced_u731/      # Main package
│   ├── __init__.py                # CLI entry point
│   ├── __metadata__.py            # Version
│   ├── sync.py                    # Telegram sync
│   ├── build.py                   # Site builder
│   ├── db.py                      # Database layer
│   ├── export.py                  # Member export
│   ├── privacy.py                 # Privacy detection
│   └── example/                   # Default site template
│       ├── config.yaml            # Default config
│       ├── template.html          # Default HTML template
│       ├── rss_template.html      # Default RSS template
│       └── static/
│           └── style.css          # Default stylesheet
├── Test/                          # Test site
│   ├── config.yaml
│   ├── data.sqlite
│   ├── template.html
│   └── site/                      # Built test site
├── setup.py                       # Package setup
├── requirements.txt               # Pinned dependencies
└── README.md
```

## Entry Points

```python
# setup.py defines two CLI commands
entry_points={
    "console_scripts": [
        "tg-archive-enhanced-u731 = tg_archive_enhanced_u731:main",
        "tgae = tg_archive_enhanced_u731:main",  # Short alias
    ],
},
```

## Template Development

The Jinja2 template receives these context variables:

```python
{
    "config": config,              # Full config dict
    "timeline": timeline,          # OrderedDict of Month objects
    "dayline": dayline,            # OrderedDict of Day objects
    "messages": messages,          # List of Message namedtuples
    "page_ids": page_ids,          # Dict[msg_id, page_slug]
    "topics": topics,              # List of Topic namedtuples
    "site_url": site_url,          # Base URL string
}
```

### Month Named Tuple

```python
Month = namedtuple("Month", ["date", "slug", "label", "count"])
# slug: "2026-03" (used in page filenames)
# label: "Mar 2026"
```

### Day Named Tuple

```python
Day = namedtuple("Day", ["date", "slug", "label", "count", "page"])
# slug: "2026-03-29"
# label: "29 Mar 2026"
# page: which paginated page this day appears on
```

### Message Named Tuple

```python
Message = namedtuple("Message", [
    "id", "type", "date", "edit_date",
    "content", "reply_to", "user", "media",
    "topic_id", "is_deleted", "deleted_at",
    "last_edit_date", "edit_history",
])
```

## Working with the Database

### Query messages directly

```python
from tg_archive_enhanced_u731.db import DB

with DB("data.sqlite") as db:
    # Get all messages
    for msg in db.get_all_messages():
        print(msg.id, msg.content)

    # Get messages for a specific month
    for msg in db.get_messages(2026, 3):
        print(msg.date, msg.content)

    # Get messages by user
    for msg in db.get_messages_by_user(user_id=12345):
        print(msg.date, msg.content)
```

### Inspect topics

```python
with DB("data.sqlite") as db:
    topics = db.get_topics()
    for t in topics:
        print(t.topic_id, t.title, t.icon_emoji)
```

### Get users

```python
with DB("data.sqlite") as db:
    users = db.get_users()
    for u in users:
        print(u.id, u.username, u.first_name, u.last_name)
```

## Adding a New Feature

### Steps

1. **Database changes**: Add columns or tables in `db.py` schema and migration
2. **Sync changes**: Add fetching logic in `sync.py`
3. **Build changes**: Add rendering logic in `build.py` and template context
4. **Export changes**: Add export fields in `export.py`
5. **Config changes**: Add new config keys in `__init__.py`

### Add a new config option

```python
# In __init__.py, add to _CONFIG
_CONFIG = {
    ...
    "new_option": default_value,
}
```

### Add a new database column

```python
# In db.py, add to schema string
schema = """
CREATE table messages (
    ...
    new_column TEXT DEFAULT NULL,
    ...
)
```

Also add migration in `__init__` method:
```python
try:
    cur.execute("ALTER TABLE messages ADD COLUMN new_column TEXT DEFAULT NULL")
except sqlite3.OperationalError:
    pass
```

## Testing

The `Test/` directory contains a test site for development:

```bash
cd Test
tgae --build --template template.html
# Check Test/site/ directory
```

## Code Style

- Python 3.13+ compatible
- Type hints are used in new code
- Named tuples for data transfer objects
- Context managers for DB connections
- `logging` module for output

## Build / Package

```bash
# Build wheel
python setup.py bdist_wheel

# Or with uv
uv build

# Install locally
pip install dist/tg_archive_enhanced_u731-*.whl
```

## Version

Version is stored in `tg_archive_enhanced_u731/__metadata__.py`:

```python
__version__ = "x.y.z"
```

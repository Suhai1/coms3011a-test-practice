# STACK B (BACKUP) — Python + FastAPI + SQLite + HTML templates

Use this **only** if Stack A can't install or run (see 00-START-HERE → "When to switch").

Why this is a good backup: SQLite support (`sqlite3`) is **built into Python** — nothing to download for the database. It's also the language you use at work, so reading the code will feel familiar.

**What each part does:**
- **FastAPI** — Python web framework. You write functions; it turns them into web pages/API endpoints.
- **Uvicorn** — the server that runs FastAPI (`uvicorn app.main:app --reload`).
- **Jinja2** — HTML templates with `{{ variables }}` and `{% for %}` loops. The server builds the HTML; no React needed.
- **sqlite3** — Python's built-in SQLite library.
- **pytest** — runs tests.

---

## Exam: is Stack B even possible on this PC? (1 min check)
Ubuntu often ships Python **without** `venv` or `pip`, and you can't `apt install` them without admin rights. Check:
```bash
python3 --version                                  # 3.10+ is fine (Ubuntu 24.04 has 3.12)
python3 -m venv /tmp/vtest && echo "VENV OK"
python3 -m pip --version && echo "PIP OK"
```
| Result | What to do |
|---|---|
| `VENV OK` | Use the normal setup below |
| venv fails ("ensurepip is not available") but `PIP OK` | Skip the venv. Install with `python3 -m pip install --user --break-system-packages <packages>` and run with `python3 -m uvicorn ...` / `python3 -m pytest` |
| neither works | **Stack B is not available on this PC.** Stay on Stack A and use its database fallbacks |

---

## Exam: setup
```bash
cd ~ && mkdir exam-app && cd exam-app
python3 -m venv .venv
source .venv/bin/activate               # do this in EVERY new terminal (prompt shows (.venv))
pip install fastapi "uvicorn[standard]" jinja2 python-multipart pytest httpx
pip freeze > requirements.txt
mkdir -p app/templates app/static tests data
```

**Update AGENTS.md** — replace section 1 "Stack" with:
```
## 1. Stack (ACTIVE: Stack B)
- Python 3.11+, FastAPI, Jinja2 server-rendered templates, plain CSS in app/static/style.css (tokens from DESIGN.md).
- Database: SQLite via Python's built-in sqlite3, accessed ONLY through app/db.py. Raw SQL with ? placeholders. No ORM.
- Minimal JavaScript only where essential (vanilla JS in app/static/app.js). No CDN scripts.
- Tests: pytest + FastAPI TestClient, using a temporary database (DB_PATH env var), never data/app.db.
- Business logic lives in app/<resource>.py; routes in app/main.py stay thin.
```
And in section 2, use this layout:
```
app/
  main.py          # routes only
  db.py            # connection + migrations
  items.py         # logic + SQL per resource
  templates/       # base.html, index.html, ...
  static/style.css
tests/
data/app.db
```

### `app/db.py`
```python
import os, sqlite3
from pathlib import Path

DB_PATH = os.environ.get("DB_PATH", str(Path("data/app.db")))

MIGRATIONS = [
    # 1
    """
    CREATE TABLE items (
      id INTEGER PRIMARY KEY,
      title TEXT NOT NULL,
      status TEXT NOT NULL DEFAULT 'Todo' CHECK (status IN ('Todo','InProgress','Complete')),
      due_date TEXT,
      archived_at TEXT,
      created_at TEXT NOT NULL,
      updated_at TEXT NOT NULL
    );
    CREATE INDEX idx_items_archived ON items(archived_at);
    """,
    # 2 -> add new migrations at the end, never edit old ones
]

def get_conn() -> sqlite3.Connection:
    if DB_PATH != ":memory:":
        Path(DB_PATH).parent.mkdir(parents=True, exist_ok=True)
    conn = sqlite3.connect(DB_PATH, check_same_thread=False)
    conn.row_factory = sqlite3.Row          # rows behave like dicts: row["title"]
    conn.execute("PRAGMA foreign_keys = ON")
    conn.execute("PRAGMA journal_mode = WAL")
    migrate(conn)
    return conn

def migrate(conn: sqlite3.Connection):
    (version,) = conn.execute("PRAGMA user_version").fetchone()
    for v in range(version, len(MIGRATIONS)):
        with conn:                          # transaction: commits or rolls back
            conn.executescript(MIGRATIONS[v])
            conn.execute(f"PRAGMA user_version = {v + 1}")

conn = get_conn()
```

### `app/items.py`
```python
from datetime import datetime, date
from .db import conn

def now(): return datetime.now().isoformat(timespec="seconds")

def list_active():
    rows = conn.execute("SELECT * FROM items WHERE archived_at IS NULL ORDER BY due_date").fetchall()
    return [dict(r) for r in rows]

def create_item(title: str, due_date: str | None):
    if not title.strip():
        raise ValueError("Title is required")
    t = now()
    with conn:
        cur = conn.execute(
            "INSERT INTO items (title, due_date, created_at, updated_at) VALUES (?, ?, ?, ?)",
            (title.strip(), due_date or None, t, t))
    return dict(conn.execute("SELECT * FROM items WHERE id = ?", (cur.lastrowid,)).fetchone())

def archive_item(item_id: int):            # never DELETE
    with conn:
        conn.execute("UPDATE items SET archived_at = ?, updated_at = ? WHERE id = ?", (now(), now(), item_id))

def is_overdue(item) -> bool:              # derived, not stored
    return bool(item["due_date"]) and item["due_date"] < date.today().isoformat() and item["status"] != "Complete"
```

### `app/main.py`
```python
from fastapi import FastAPI, Request, Form
from fastapi.responses import HTMLResponse, RedirectResponse
from fastapi.staticfiles import StaticFiles
from fastapi.templating import Jinja2Templates
from . import items

app = FastAPI()
app.mount("/static", StaticFiles(directory="app/static"), name="static")
templates = Jinja2Templates(directory="app/templates")

@app.get("/", response_class=HTMLResponse)
def index(request: Request):
    rows = items.list_active()
    return templates.TemplateResponse(request, "index.html", {"items": rows, "is_overdue": items.is_overdue})

@app.post("/items")
def create(title: str = Form(...), due_date: str = Form("")):
    items.create_item(title, due_date or None)
    return RedirectResponse("/", status_code=303)

@app.post("/items/{item_id}/archive")
def archive(item_id: int):
    items.archive_item(item_id)
    return RedirectResponse("/", status_code=303)

# JSON API version (if the brief wants an API)
@app.get("/api/items")
def api_list():
    return {"data": items.list_active()}
```

### `app/templates/index.html`
```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8"><meta name="viewport" content="width=device-width, initial-scale=1">
  <title>App</title>
  <link rel="stylesheet" href="/static/style.css">
</head>
<body>
  <main class="wrap">
    <h1>Items</h1>
    <form method="post" action="/items" class="row">
      <input name="title" placeholder="Title" required>
      <input name="due_date" type="date">
      <button>Add</button>
    </form>
    {% if items %}
      <ul class="list">
      {% for i in items %}
        <li>
          <span>{{ i.title }}</span>
          <span class="mono">{{ i.due_date or "" }}</span>
          {% if is_overdue(i) %}<span class="warn">Overdue</span>{% endif %}
          <form method="post" action="/items/{{ i.id }}/archive"><button>Archive</button></form>
        </li>
      {% endfor %}
      </ul>
    {% else %}
      <p class="empty">Nothing here yet — add your first item above.</p>
    {% endif %}
  </main>
</body>
</html>
```
`app/static/style.css`: paste the `:root { ... }` tokens from DESIGN.md, then let Qoder style it with Prompt 8.

### `tests/test_items.py`
```python
import os
os.environ["DB_PATH"] = ":memory:"        # must be set BEFORE importing the app
from app import items

def test_archive_hides_but_keeps_row():
    it = items.create_item("Write report", None)
    items.archive_item(it["id"])
    assert all(r["id"] != it["id"] for r in items.list_active())

def test_rejects_empty_title():
    import pytest
    with pytest.raises(ValueError):
        items.create_item("   ", None)
```
Add an empty `app/__init__.py` and `tests/__init__.py` so imports work.

---

## Commands you'll use all day
| Command | What it does |
|---|---|
| `source .venv/bin/activate` | Use the project's Python (do in every new terminal) |
| `uvicorn app.main:app --reload` | Run the app at http://127.0.0.1:8000 (auto-reloads) |
| `pytest` | Run tests |
| `http://127.0.0.1:8000/docs` | FastAPI's free auto-generated API docs page |
| `sqlite3 data/app.db` | Look inside the database |
| `pip freeze > requirements.txt` | Save the installed package list (do before final commit) |

## Common errors
| Error | Fix |
|---|---|
| `ModuleNotFoundError: fastapi` | You forgot `source .venv/bin/activate` |
| `Form data requires "python-multipart"` | `pip install python-multipart` |
| `ImportError: attempted relative import` | Run from the project root as `uvicorn app.main:app`, not `python app/main.py` |
| `sqlite3.ProgrammingError: ... thread` | Make sure `check_same_thread=False` is in `get_conn()` |
| Template changes not showing | Hard refresh the browser (Ctrl+Shift+R) |

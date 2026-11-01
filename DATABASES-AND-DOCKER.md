# DATABASES-AND-DOCKER.md — The plain-English version

**Short version for tomorrow: use SQLite. You almost certainly don't need Docker.**
Read Part 1 and 2 tonight. Part 3 (Postgres/Docker) is only for emergencies or if the brief demands a "real" database server.

---

## Part 1 — What a database is (2-minute version)

A database is a set of **tables**, like spreadsheets:

`items` table
| id | title | status | due_date | archived_at |
|---|---|---|---|---|
| 1 | Write report | Todo | 2026-11-05 | NULL |
| 2 | Buy milk | Complete | 2026-11-01 | NULL |

- **Row** = one record. **Column** = one field.
- **Primary key** (`id`) = unique number for each row.
- **Foreign key** = a column pointing to another table's `id` (e.g. `items.project_id → projects.id`).
- **NULL** = "no value".
- **Index** = a lookup table that makes searching a column fast. Add one to any column you filter or sort by.
- **Transaction** = a group of changes that all succeed or all fail together.
- **Migration** = a numbered script that changes the table structure (add a column, add a table) without losing data.

**SQLite vs Postgres/MySQL:** SQLite is a single file on disk — your app opens it directly, nothing else to run. Postgres/MySQL are separate server programs your app connects to over the network. For a single-user exam app, SQLite is simpler and perfectly good.

---

## Part 2 — SQL cheat sheet

```sql
-- Create a table
CREATE TABLE projects (
  id INTEGER PRIMARY KEY,
  name TEXT NOT NULL UNIQUE,
  created_at TEXT NOT NULL
);

-- A table that belongs to another (foreign key)
CREATE TABLE items (
  id INTEGER PRIMARY KEY,
  project_id INTEGER REFERENCES projects(id),
  title TEXT NOT NULL,
  status TEXT NOT NULL CHECK (status IN ('Todo','InProgress','Complete')),
  archived_at TEXT
);
CREATE INDEX idx_items_project ON items(project_id);

-- Many-to-many (items <-> tags) = a join table, never "tag1,tag2" in a column
CREATE TABLE tags (id INTEGER PRIMARY KEY, name TEXT NOT NULL UNIQUE);
CREATE TABLE item_tags (
  item_id INTEGER NOT NULL REFERENCES items(id),
  tag_id  INTEGER NOT NULL REFERENCES tags(id),
  PRIMARY KEY (item_id, tag_id)
);

-- Insert
INSERT INTO items (title, status) VALUES ('Write report', 'Todo');

-- Read
SELECT * FROM items;
SELECT title, status FROM items WHERE archived_at IS NULL ORDER BY title;
SELECT * FROM items WHERE title LIKE '%report%';
SELECT status, COUNT(*) FROM items GROUP BY status;

-- Join (items with their project name)
SELECT items.title, projects.name
FROM items LEFT JOIN projects ON projects.id = items.project_id;

-- Items that have BOTH tag 1 and tag 2
SELECT item_id FROM item_tags WHERE tag_id IN (1, 2)
GROUP BY item_id HAVING COUNT(*) = 2;

-- Update
UPDATE items SET status = 'Complete' WHERE id = 1;

-- "Delete" the safe way (archive)
UPDATE items SET archived_at = datetime('now') WHERE id = 1;

-- Add a column later (in a NEW migration)
ALTER TABLE items ADD COLUMN priority INTEGER NOT NULL DEFAULT 2;
```

**Always use `?` placeholders in code** — never glue user input into SQL strings:
```ts
db.prepare("SELECT * FROM items WHERE id = ?").get(id);        // ✅ safe
db.prepare(`SELECT * FROM items WHERE id = ${id}`).get();       // ❌ SQL injection
```

### Looking inside your SQLite database
Check first: `sqlite3 --version`. Ubuntu doesn't always have the `sqlite3` command, and you can't `apt install` without admin rights.

**If `sqlite3` is missing,** use Python, which always has SQLite built in. From the project folder:
```bash
python3 -c "import sqlite3; c=sqlite3.connect('data/app.db'); print([r[0] for r in c.execute(\"select name from sqlite_master where type='table'\")])"
python3 -c "import sqlite3; c=sqlite3.connect('data/app.db'); [print(r) for r in c.execute('select * from items limit 10')]"
```
(The first lists tables. The second shows rows; change `items` to your table name.)

**If `sqlite3` exists:**
```bash
sqlite3 data/app.db
```
Then inside:
```
.tables                       -- list tables
.schema items                 -- show how the table is defined
.headers on
.mode column                  -- nicer output
SELECT * FROM items LIMIT 10;
PRAGMA user_version;          -- which migration you're on
.quit
```
Useful for: checking the agent actually saved something, checking archived rows still exist, proving to the demonstrator nothing gets deleted.

**Fresh start:** stop the app, `rm data/app.db*`, start the app again — migrations recreate it empty. Then `npm run seed`.

---

## Part 3 — Docker + Postgres (only if needed)

### What Docker is
Docker runs a program inside a sealed box ("container") with everything it needs, so you don't install it on your laptop directly. An **image** is the recipe (e.g. `postgres:16-alpine`); a **container** is a running copy. `docker compose` starts containers from a `docker-compose.yml` file.

On the lab PCs, check it works before relying on it:
```bash
docker run --rm hello-world
```
- `permission denied ... docker.sock` → your lab account isn't allowed to use Docker. Stick to SQLite.
- `docker: command not found` → not installed. Stick to SQLite.
- If `docker compose` doesn't work, try `docker-compose` (older install).

Images are downloaded on the lab PC on first use (the lecturer said Docker is allowed through the firewall). Pulling on your Mac tonight won't help the lab PC.

### Running Postgres
Create `docker-compose.yml` in the project root:
```yaml
services:
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: app
      POSTGRES_DB: app
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data   # keeps data when the container stops
volumes:
  pgdata:
```
Commands:
| Command | What it does |
|---|---|
| `docker compose up -d` | Start Postgres in the background |
| `docker compose ps` | Is it running? |
| `docker compose logs db` | See errors |
| `docker compose down` | Stop it (data kept) |
| `docker compose down -v` | Stop AND wipe the data |
| `docker compose exec db psql -U app` | Open a SQL prompt inside Postgres (`\dt` lists tables, `\q` quits) |

Connection string for your app:
```
postgresql://app:app@localhost:5432/app
```
- Node: `npm install pg` → `new Pool({ connectionString: process.env.DATABASE_URL })`. Note: queries become **async** (`await pool.query("SELECT ... WHERE id = $1", [id])`) and placeholders are `$1, $2` not `?`.
- Python: `pip install "psycopg[binary]"` → placeholders are `%s`.

### SQLite → Postgres differences that bite
| SQLite | Postgres |
|---|---|
| `INTEGER PRIMARY KEY` auto-increments | Use `id SERIAL PRIMARY KEY` or `id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY` |
| `?` placeholders | `$1, $2` (Node `pg`) / `%s` (Python) |
| Synchronous calls (better-sqlite3) | Everything is `async/await` |
| `datetime('now')` | `now()` |
| Dates as TEXT | Use `TIMESTAMPTZ` / `DATE` columns |
| `PRAGMA user_version` for migrations | Keep a `schema_migrations` table instead |

If you ever do switch, tell Qoder exactly that: *"Migrate src/lib/db.ts from SQLite to Postgres using the pg package and DATABASE_URL. Keep the same lib function names; make them async. Update all callers and tests."* — and do it in its own commit.

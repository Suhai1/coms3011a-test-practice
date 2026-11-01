# STACK A (MAIN) — Next.js 15 + TypeScript + Tailwind + SQLite, on the LAB machines

You used this stack in Lab 1, Lab 2 and the semester test, so it's the default.

> **The lab machines run Ubuntu with Node 18.19.1.** The latest versions of Next.js, Tailwind, better-sqlite3 and Vitest all need Node 20+, so the normal `@latest` setup **fails**. Everything below is pinned to versions that were **tested on Node 18.19.1**: build, tests and API all pass.

**What each part does:**
- **Next.js 15** — one project that is both the website (React pages) and the backend (API routes). One `npm run dev` runs everything.
- **TypeScript** — JavaScript with types; catches mistakes before you run the code.
- **Tailwind 4.1.18** — style with class names (`bg-accent p-4`). Pinned: 4.2+ breaks on Node 18.
- **SQLite** — the whole database is one file (`data/app.db`). No server to install, survives restarts.
- **Vitest 3** — runs tests. Pinned: v4 needs Node 20.

---

## Known-good versions (tested on Node 18.19.1)

| Package | Version | Why pinned |
|---|---|---|
| `next` | 15.5.27 | Next 16 needs Node 20 |
| `react`, `react-dom` | 19.1.0 | matches Next 15.5 |
| `tailwindcss`, `@tailwindcss/postcss` | **4.1.18 exact** | 4.2+ native binaries need Node 20 → `Cannot find module './tailwindcss-oxide.linux-x64-gnu.node'` |
| `better-sqlite3` | **11.10.0 exact** | v12 needs Node 20 |
| `sql.js` (fallback DB) | 1.15.2 | pure WebAssembly, no native binary |
| `vitest` | **3.2.4 exact** | v4 needs Node 20 |
| `tsx` | latest | works on Node 18 |

**Never run `npm install <pkg>@latest`, `npm update` or `npm audit fix --force`** — they will pull versions that break on Node 18. Tell Qoder the same.

---

## Exam: step 0 — check the machine (2 min, first thing)
```bash
node -v          # expect v18.19.1 — if it's 20+ the pins still work, nothing to change
npm -v
git --version
python3 --version
```
If `node -v` says **22 or higher**, you can also use Node's built-in `node:sqlite`, but the pinned setup below works everywhere, so just use it.

## Exam: setup (≈ 10 min)

```bash
cd ~                       # or wherever you're allowed to work
npx -y create-next-app@15.5.27 exam-app --ts --tailwind --eslint --app --src-dir --use-npm --import-alias "@/*" --turbopack --yes
cd exam-app

# 1. Pin Tailwind (create-next-app installs 4.3+, which FAILS on Node 18)
npm install -E -D tailwindcss@4.1.18 @tailwindcss/postcss@4.1.18

# 2. Database driver (Option 1 — see below if it fails)
npm install -E better-sqlite3@11.10.0

# 3. Testing tools
npm install -E -D @types/better-sqlite3 vitest@3.2.4 tsx

# 4. Folders
mkdir -p data src/lib src/tests scripts

# 5. Prove it works
npm run build
```
`npm run build` must end with a list of routes and no errors. If it fails with `tailwindcss-oxide`, step 1 didn't apply. Run it again.

Then:
1. Copy `AGENTS.md`, `COMMITS.md`, `DESIGN.md` and your `BRIEF.md` into `exam-app/`.
2. Add the lines from `COMMITS.md` → `.gitignore` to `.gitignore`.
3. Do the git setup in `COMMITS.md` and push.
4. Add the test and seed scripts to `package.json`:
```bash
npm pkg set scripts.test="DB_PATH=:memory: vitest run"
npm pkg set scripts.seed="tsx scripts/seed.ts"
```

Open the folder in Qoder and use **PROMPTS.md → Prompt 1**. Qoder will write the files below. They're here so you know what "correct" looks like and can paste them yourself if it goes wrong.

---

## The database: three options, in order

| Option | Use when | Code changes |
|---|---|---|
| **1. better-sqlite3@11.10.0** | Default. Fast, simple, synchronous. | none |
| **2. sql.js** | `npm install better-sqlite3` fails (it downloads a prebuilt binary from GitHub, which the firewall may block, and then tries to compile) | swap `db.ts`; lib functions become `async` |
| **3. JSON files** | Both of the above fail. This is what the semester test used, and it worked. | different storage; tell Qoder |

### `src/lib/db.ts` — Option 1: better-sqlite3 (default)
```ts
import Database from "better-sqlite3";
import fs from "node:fs";
import path from "node:path";
import { migrate } from "./migrations";

const DB_PATH = process.env.DB_PATH ?? path.join(process.cwd(), "data", "app.db");

function open() {
  if (DB_PATH !== ":memory:") fs.mkdirSync(path.dirname(DB_PATH), { recursive: true });
  const db = new Database(DB_PATH);
  db.pragma("journal_mode = WAL");
  db.pragma("foreign_keys = ON");
  migrate(db);
  return db;
}

// Reuse one connection during Next.js hot-reload
const g = globalThis as unknown as { __db?: Database.Database };
export const db = g.__db ?? (g.__db = open());
```

### `src/lib/db.ts` — Option 2: sql.js (if better-sqlite3 won't install)
sql.js is SQLite compiled to WebAssembly: a plain npm package with no native binary and no GitHub download. It works on any Node version. It keeps the DB in memory and writes the file to disk after every change. This wrapper gives it the same `prepare().run/get/all` shape as better-sqlite3, so `migrations.ts` works unchanged.

```bash
npm uninstall better-sqlite3 @types/better-sqlite3
npm install -E sql.js@1.15.2
npm install -E -D @types/sql.js
```
`next.config.ts` (required, or the build can't find the .wasm file):
```ts
import type { NextConfig } from "next";
const nextConfig: NextConfig = { serverExternalPackages: ["sql.js"] };
export default nextConfig;
```
`src/lib/db.ts`:
```ts
import initSqlJs, { type Database } from "sql.js";
import fs from "node:fs";
import path from "node:path";
import { migrate } from "./migrations";

// sql.js = SQLite compiled to WebAssembly. No native binary, works on any Node version.
// It runs in memory, so we write the whole DB file to disk after every change.
const DB_PATH = process.env.DB_PATH ?? path.join(process.cwd(), "data", "app.db");
type P = (string | number | null)[];

export type DB = {
  exec(sql: string): void;
  prepare(sql: string): {
    run(...params: P): { changes: number; lastInsertRowid: number };
    get(...params: P): unknown;
    all(...params: P): unknown[];
  };
};

function wrap(raw: Database): DB {
  let inTx = false;
  const save = () => {
    if (DB_PATH === ":memory:" || inTx) return;
    fs.mkdirSync(path.dirname(DB_PATH), { recursive: true });
    fs.writeFileSync(DB_PATH, Buffer.from(raw.export()));
  };
  const rows = (sql: string, params: P) => {
    const st = raw.prepare(sql);
    try {
      st.bind(params);
      const out: unknown[] = [];
      while (st.step()) out.push(st.getAsObject());
      return out;
    } finally {
      st.free();
    }
  };
  return {
    exec(sql) {
      raw.exec(sql);
      const s = sql.trim().toUpperCase();
      if (s.startsWith("BEGIN")) inTx = true;
      else if (s.startsWith("COMMIT") || s.startsWith("ROLLBACK")) { inTx = false; save(); }
      else save();
    },
    prepare(sql) {
      return {
        run: (...params) => {
          const st = raw.prepare(sql);
          try { st.run(params); } finally { st.free(); }
          const changes = raw.getRowsModified();
          const { id } = rows("SELECT last_insert_rowid() AS id", [])[0] as { id: number };
          save();
          return { changes, lastInsertRowid: id };
        },
        get: (...params) => rows(sql, params)[0],
        all: (...params) => rows(sql, params),
      };
    },
  };
}

async function open(): Promise<DB> {
  const SQL = await initSqlJs({
    locateFile: (f: string) => path.join(process.cwd(), "node_modules", "sql.js", "dist", f),
  });
  const raw = DB_PATH !== ":memory:" && fs.existsSync(DB_PATH)
    ? new SQL.Database(fs.readFileSync(DB_PATH))
    : new SQL.Database();
  raw.exec("PRAGMA foreign_keys = ON;");
  const db = wrap(raw);
  migrate(db);
  return db;
}

// One shared connection (survives Next.js hot reload)
const g = globalThis as unknown as { __db?: Promise<DB> };
export function getDb(): Promise<DB> {
  return (g.__db ??= open());
}
```
With Option 2, every lib function starts with `const db = await getDb();` and is `async`. Routes and tests `await` them. Tell Qoder:
> *"We're using sql.js via getDb() in src/lib/db.ts, not better-sqlite3. Every lib function must be async and start with `const db = await getDb()`. Don't use db.pragma or db.transaction — use db.exec('BEGIN') / 'COMMIT' / 'ROLLBACK'. Update AGENTS.md section 1 to say this."*

### Option 3: JSON files (last resort)
Tell Qoder:
> *"No database driver can be installed. Store data as JSON files in data/ using node:fs, behind the same lib functions. Serialise writes, so read-modify-write cycles don't overlap. Write to a temp file and rename, so a crash never leaves half a file. Archive with an archived_at field; never remove records."*

---

### `src/lib/migrations.ts` — versioned schema changes (works with Option 1 and 2)
```ts
// Each entry runs once, in order. NEVER edit an old one — add a new one at the end.
const MIGRATIONS: string[] = [
  /* 1 */ `
  CREATE TABLE items (
    id INTEGER PRIMARY KEY,
    title TEXT NOT NULL,
    status TEXT NOT NULL DEFAULT 'Todo' CHECK (status IN ('Todo','InProgress','Complete')),
    due_date TEXT,
    archived_at TEXT,
    created_at TEXT NOT NULL,
    updated_at TEXT NOT NULL
  );
  CREATE INDEX idx_items_status ON items(status);
  CREATE INDEX idx_items_due ON items(due_date);
  CREATE INDEX idx_items_archived ON items(archived_at);
  `,
  // /* 2 */ `ALTER TABLE items ADD COLUMN priority INTEGER NOT NULL DEFAULT 2;`,
];

type DB = { exec(sql: string): unknown; prepare(sql: string): { get(): unknown } };

export function migrate(db: DB) {
  const row = db.prepare("PRAGMA user_version").get() as { user_version: number };
  for (let v = row.user_version; v < MIGRATIONS.length; v++) {
    db.exec("BEGIN");
    try {
      db.exec(MIGRATIONS[v]);
      db.exec(`PRAGMA user_version = ${v + 1}`);
      db.exec("COMMIT");
    } catch (e) {
      db.exec("ROLLBACK");
      throw e;
    }
  }
}
```
The `items` table is just an example. Replace it with your schema from the planning stage.

### `src/lib/items.ts` — what a "lib" file looks like (Option 1)
```ts
import { db } from "./db";

export type Item = { id: number; title: string; status: "Todo" | "InProgress" | "Complete";
  due_date: string | null; archived_at: string | null; created_at: string; updated_at: string };

const now = () => new Date().toISOString();

export function listActive(): Item[] {
  return db.prepare("SELECT * FROM items WHERE archived_at IS NULL ORDER BY due_date").all() as Item[];
}

export function createItem(title: string, due_date: string | null): Item {
  if (!title.trim()) throw new Error("Title is required");
  const t = now();
  const res = db.prepare(
    "INSERT INTO items (title, due_date, created_at, updated_at) VALUES (?, ?, ?, ?)"
  ).run(title.trim(), due_date, t, t);
  return db.prepare("SELECT * FROM items WHERE id = ?").get(res.lastInsertRowid) as Item;
}

export function archiveItem(id: number) {           // never DELETE
  db.prepare("UPDATE items SET archived_at = ?, updated_at = ? WHERE id = ?").run(now(), now(), id);
}

// Derived, not stored:
export const isOverdue = (i: Item, today = new Date().toLocaleDateString("en-CA")) =>
  !!i.due_date && i.due_date < today && i.status !== "Complete";
```

### `src/app/api/items/route.ts` — a thin API route
```ts
import { NextResponse } from "next/server";
import { createItem, listActive } from "@/lib/items";

export const dynamic = "force-dynamic";

export async function GET() {
  return NextResponse.json({ data: listActive() });
}

export async function POST(req: Request) {
  try {
    const body = await req.json();
    return NextResponse.json({ data: createItem(body.title ?? "", body.due_date ?? null) }, { status: 201 });
  } catch (e) {
    return NextResponse.json({ error: (e as Error).message }, { status: 400 });
  }
}
```

### `src/tests/items.test.ts` — a real test
```ts
import { describe, it, expect } from "vitest";
import { createItem, archiveItem, listActive } from "@/lib/items";

describe("items", () => {
  it("archive hides from active list but keeps the row", () => {
    const item = createItem("Write report", null);
    archiveItem(item.id);
    expect(listActive().find(i => i.id === item.id)).toBeUndefined();
  });
  it("rejects empty title", () => {
    expect(() => createItem("   ", null)).toThrow();
  });
});
```
Vitest needs to understand the `@/` import. Create `vitest.config.ts`:
```ts
import { defineConfig } from "vitest/config";
import path from "node:path";
export default defineConfig({
  resolve: { alias: { "@": path.resolve(__dirname, "src") } },
  test: { environment: "node" },
});
```

---

## Commands you'll use all day
| Command | What it does |
|---|---|
| `npm run dev` | Start the app at http://localhost:3000 (auto-reloads when code changes) |
| `npm run build` | Production build. Catches type errors. Run before each commit |
| `npm test` | Run tests |
| `npm run seed` | Fill the DB with sample data |
| `rm -f data/app.db*` | Start with an empty database (it's recreated on next run) |
| Look inside the DB | see `DATABASES-AND-DOCKER.md` (the `sqlite3` command may not be installed on the lab PCs) |

---

## Node 18 gotchas (tell Qoder if it hits these)
| Symptom | Cause | Fix |
|---|---|---|
| `Cannot find module './tailwindcss-oxide.linux-x64-gnu.node'` | Tailwind 4.2+ on Node 18 | `npm install -E -D tailwindcss@4.1.18 @tailwindcss/postcss@4.1.18` |
| `ReferenceError: File is not defined` / `instanceof File` crashes on upload | Node 18 has no global `File` | Duck-type it: `typeof v === "object" && v !== null && typeof (v as Blob).arrayBuffer === "function"` (your semester test did exactly this) |
| `npm install` warns `EBADENGINE` or install fails for a package | that package version needs Node 20 | Install an older major version. Ask Qoder: "find the newest version of X that supports Node 18" |
| `Cannot find module 'node:sqlite'` | needs Node 22.5+ | Use Option 1 or 2 |
| `toSorted is not a function` / `Object.groupBy is not a function` | newer JS features | `toSorted`/`groupBy` aren't in Node 18; use `[...arr].sort()` and a manual reduce |

## Firewall problems and fixes
| Symptom | Likely cause | Fix |
|---|---|---|
| `npm install better-sqlite3` fails with `prebuild-install` / `node-gyp` / GitHub errors | It downloads a binary from GitHub, which may be blocked | **Option 2 (sql.js)** |
| Build hangs or fails mentioning `fonts.googleapis.com` | `next/font/google` (Geist) in `layout.tsx` | Delete the font imports and `className` font variables in `src/app/layout.tsx`, and use the system fonts from DESIGN.md |
| `npm install` hangs for everything | npm registry blocked/slow | Check the allowed-domains list, then try again. If it's still dead after 10 min → Stack B |
| Playwright install fails | Browser download is blocked | Skip end-to-end browser tests; use Vitest only |

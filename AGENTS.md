# AGENTS.md — Project rules for the AI agent

> Read this whole file before every task. These rules override any default habits.
> If a request from the user conflicts with a rule here, STOP and ask before proceeding.

## 0. How to work with me
- Do **one** task at a time. Do only what was asked. Do not "also improve" unrelated code.
- Before a non-trivial change: list the files you will create/modify and wait for my OK.
- After every change: run `npm run build` (or the stack's equivalent) and `npm test`. Report the result honestly. If something fails, say so — never claim it works without running it.
- If you are unsure what I want, ask ONE short question instead of guessing.
- If the same fix fails twice, stop and explain the root cause instead of trying a third variation.
- Never install a new library without asking. Use what is already installed. Never add a second library that does the same job as an existing one.
- Keep the app runnable at all times. Never leave half-finished features wired into the UI.

## 1. Stack (ACTIVE: Stack A)
- **Framework:** Next.js (App Router) + TypeScript (strict) + Tailwind CSS
- **Database:** SQLite, accessed ONLY through `src/lib/db.ts`. Raw SQL with prepared statements and `?` placeholders. No ORM.
- **Tests:** Vitest. Tests run against an in-memory database (`DB_PATH=:memory:`), never against `data/app.db`.
- **Package manager:** npm.
- **Fonts:** system font stacks only. Do NOT use `next/font/google` or any external font/CDN (the network is firewalled).
- **No external services:** no auth providers, no cloud databases, no CDN scripts, no analytics.

## 2. Folder layout
```
src/
  app/                 # pages and route handlers ONLY (thin: call lib functions)
    api/<resource>/route.ts
  components/          # React UI components
  lib/
    db.ts              # the ONLY place that opens the database
    migrations.ts      # schema changes, versioned
    <resource>.ts      # all business logic + SQL for one resource (e.g. tasks.ts)
    types.ts           # shared TypeScript types
  tests/               # *.test.ts
data/                  # app.db lives here (gitignored)
scripts/seed.ts        # seed data
```
- Business logic lives in `src/lib/`. Pages, API routes, and any CLI all call the same lib functions — one implementation, never two.

## 3. Database rules (these are the ones agents break — do not break them)
- **Never delete rows.** "Delete" in the UI means set `archived_at = <timestamp>`. Queries for active lists filter `WHERE archived_at IS NULL`. Archived items must still be viewable.
- **Never store derived values.** Anything computable from other columns (e.g. "overdue", progress %, counts, totals) is computed at read time, not stored in a column.
- **Fixed enums stay fixed.** If the brief defines a fixed set of statuses/types, enforce them with a `CHECK` constraint and a TypeScript union type. Never add extra values (e.g. "overdue" is NOT a status).
- **Schema changes go through `migrations.ts`** as a new numbered migration. Never drop and recreate a table that has data. Never edit an old migration.
- **Every table** has `id INTEGER PRIMARY KEY`, `created_at TEXT NOT NULL`, `updated_at TEXT NOT NULL` (ISO 8601 strings).
- **Foreign keys ON** and declared for every relationship. Many-to-many uses a proper join table, never comma-separated strings.
- **Add an index** for every column used in `WHERE`, `ORDER BY`, or joins.
- **No N+1 queries:** never run a query inside a loop over rows. Use a JOIN or a single `IN (...)` query.
- **Multi-step writes use a transaction.**
- **Dates:** store as ISO strings. Compare "today/overdue" using the user's local date, not the server's UTC midnight.
- **Graphs/trees** (dependencies, parent-child): every traversal has a visited-set cycle guard. Reject cycles at write time with a clear message.

## 4. Code rules
- TypeScript strict, no `any`. Validate all user input at the API boundary; return `400` with a clear message for bad input, `404` for missing items.
- API routes return JSON `{ data }` on success, `{ error: "message" }` on failure, with correct HTTP status codes.
- Small components. No file over ~250 lines — split it.
- Every async UI action shows a loading state and an error state. If a save fails, the UI must revert, not pretend it worked.
- Empty states: every list/view shows a helpful message when there is nothing to show — never a blank panel.

## 5. Tests
- Test **behaviour**, not rendering. Never mock the database — use the real in-memory SQLite.
- Each test sets up its own data. No test depends on another test or on `data/app.db`.
- Always include the awkward cases: empty input, invalid input, archive behaviour, date boundaries, cycles.

## 6. Design
- Follow `DESIGN.md` exactly. Use only its colour tokens and spacing scale. Do not invent new colours.
- Banned: purple/violet, gradients, glassmorphism/backdrop-blur, large drop shadows, emoji as icons, gradient text.

## 7. Git
- Follow `COMMITS.md` for message format.
- Commit only when the build and tests pass. One feature = one or more small commits.
- Never commit `data/*.db`, `.env`, or `node_modules`.
- Never run `git push --force`, `git reset --hard`, or delete branches unless I explicitly ask.

## 8. Project-specific notes
<!-- Fill in at the start of the exam from the brief -->
- App: [one-line description]
- Fixed rules from the brief: [e.g. "items can't be deleted, only archived"]
- Core features: [list]

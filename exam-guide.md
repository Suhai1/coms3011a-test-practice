# COMS3011A Test: Exam Playbook

<div class="hero" markdown="1">
**14:30 to 17:00 · Qoder only · marked from a fresh `git clone` of your last commit before 17:00**

Your three golden rules:

1. **Push after every feature.** Pushed code is the only code that counts.
2. **One small task at a time.** Ask, check in the browser, commit.
3. **Stop new work at 16:40.** The last 20 minutes are for proving the clone runs.
</div>

## 1. Timeline at a glance

| Time | Do this | Push? |
| --- | --- | --- |
| **14:30** | Read the whole brief (5 min). Log in to Qoder and GitHub. | No |
| **14:35** | Create the repo, clone it, scaffold Next.js (section 2). | Yes |
| **14:40** | Add the Qoder rules (section 3), then build Core. | Yes |
| **15:00** | Core working + README + start.sh. **This is your floor.** | Yes |
| **15:00–16:40** | Features from section 6, one at a time. | After each one |
| **16:40** | Stop. Final checklist (section 8). | Yes |
| **16:55** | Final push done. Leave it alone. | Done |

<div class="warn" markdown="1">
**If you're behind at 15:15 and Core isn't working:** stop adding anything. Fix Core only. A working Core scores far more than a broken app with features.
</div>

## 2. Setup (14:35, about 5 minutes)

### 2a. Create the repo on GitHub

- New repository → name `coms3011a-test` → **Public** → leave **everything unticked** (no README) → Create.
- Copy the HTTPS URL.

### 2b. Clone and configure (terminal)

```
git --version
node -v
git clone https://github.com/Suhai1/coms3011a-test.git
cd coms3011a-test
git config user.name "Suhail Seedat"
git config user.email "2586153@students.wits.ac.za"
```

When git asks for login: username **Suhai1**, password **your token** (from your phone note).

### 2c. Scaffold Next.js yourself (faster than asking Qoder)

```
npx create-next-app@latest . --yes
```

`--yes` accepts all defaults (TypeScript, App Router, Tailwind). If it errors on the flag, run it without `--yes` and press Enter for every question. If it complains the folder isn't empty, check for stray files with `ls -a` (only `.git` should be there).

```
npm run dev
```

Open http://localhost:3000. Leave this terminal running. Use a **second terminal** for git.

```
git add -A
git commit -m "chore: scaffold next.js app"
git push
```

## 3. Qoder setup (do this once, 2 minutes)

<div class="tip" markdown="1">
**Make the rules permanent.** Create a file `.qoder/rules/project.md` in the repo and paste the rules block below into it. Qoder applies project rules to every chat, so you never have to repaste them. If that doesn't seem to work, paste the block at the start of each new chat instead.
</div>

Use **Agent mode** (it edits files and runs commands), not Ask mode.

### The rules block (copy exactly; edit if the brief changes)

```
PROJECT RULES (apply to every change):
- Stack: Next.js App Router + TypeScript + SQLite. Local-first, single user, no auth.
- Use better-sqlite3 for the database. Keep ONE db module (lib/db.ts, or src/lib/db.ts if there is a src folder).
- The database file lives at ./data/app.db. On startup, create the folder,
  file and ALL tables/indexes if missing (CREATE TABLE IF NOT EXISTS).
  A fresh git clone must run with no manual setup.
- Tasks are NEVER deleted. Archive = an `archived` flag. No DELETE on tasks,
  no copying rows to another table.
- Statuses are fixed: Todo, InProgress, Complete. Never add a fourth.
- Overdue is DERIVED at read time from due_date in the user's local
  timezone (compute in the browser). Never store it, never make it a status.
- All data must survive a server restart.
- Add indexes on columns we filter or sort by. No N+1 queries (use joins).
- Do not add a new npm library if an installed one can do the job.
- Keep `npm install` and `npm run dev` working at all times.
- Do ONE feature per request. When finished, run `npm run build` and fix
  any errors, then tell me the exact steps to test it in the browser.
```

Also add `data/` to `.gitignore` so your database isn't committed (the app recreates it).

## 4. Copy-paste prompts

### Prompt A: Core (send right after the rules)

```
Build the Core of the todo app (Lab 1 rules). Requirements:
1. Create a task with Title, Description, Due Date and Topic; it appears in the list.
2. Edit a task; changes persist across a server restart.
3. Archive a task (flag, never delete). It leaves the active list but can be
   viewed in an "Archived" view.
4. Sort the list by topic, by status and by due date (all three correct).
5. Overdue tasks are visibly marked (derived, not stored). The status selector
   only offers Todo, InProgress, Complete.
Use server actions or API routes, keep the UI simple and clean.
When done, run npm run build and fix errors, then list how to test each item.
```

### Prompt B: Any feature

```
Next feature: <CODE + NAME>.
Requirements: <paste the feature's full text from the brief>.
DONE WHEN: <paste the "Done when" line>.
Follow the project rules. Don't change unrelated code.
Run npm run build, fix errors, then tell me exactly how to test the DONE WHEN.
```

### Prompt C: Fixing something

```
Bug: <what I did> → <what happened> (expected: <what should happen>).
Error text: <paste the exact error from terminal or browser console>.
Fix only this. Don't refactor other code.
```

### Prompt D: README (after every 2–3 features)

```
Update README.md: how to run (./start.sh or npm install && npm run dev),
Node version needed, and a "Features" list with one line each on WHERE to
find it in the UI (e.g. "Kanban: click Board in the top nav").
Only list features that work.
```

## 5. Qoder speed tips

<div class="tip" markdown="1">
**The fastest way through the exam is small, precise requests and checking every result.** Big vague requests cost more time in fixing than they save.
</div>

- **New chat per feature.** Long chats fill the context with old attempts ("context rot", Lecture 3) and the agent gets worse. Start fresh for each feature; the rules file carries over.
- **Paste the brief's exact wording**, including the "Done when" line. The agent aims at what you give it.
- **Point it at files with `@`** (e.g. `@lib/db.ts`) so it doesn't waste time searching or create a second db module.
- **Paste errors exactly.** Copy the full red error from the terminal or browser console. "It's broken" gets a guess.
- **Object in specifics.** "You stored overdue as a column; derive it from due_date at read time" gets the fix in one go.
- **Let it run the build itself.** Asking it to run `npm run build` and fix errors is the generate-test loop doing your checking for you.
- **While it works, prep the next prompt.** Copy the next feature's text from the brief so you can send it the second this one is committed.
- **Review the change list before accepting.** Watch for: new tables you didn't expect, a new library, DELETE statements, an `overdue` column.
- **If it goes round in circles twice, stop.** Discard (`git restore . && git clean -fd`), start a new chat, and ask for a smaller piece.
- **Don't run two agents on the same files.** Only run things in parallel if they touch completely different files.
- **Avoid Quest/long autonomous modes near the end.** Nothing big in flight after 16:30.

## 6. Feature plan (aim for 57 points, need 50)

Work down this list. Skip anything stuck for more than 10 minutes. If the test brief has a different menu, use the same rule: cheap, independent, clear "done when" first.

| # | Code | Feature | Pts | Total | Done when (short) |
| --- | --- | --- | --- | --- | --- |
| 1 | DS6 | Priority + effort | 3 | 3 | Sort by priority then due date is stable |
| 2 | QC3 | Seed script | 3 | 6 | `npm run seed` gives realistic data, not "Task 1" |
| 3 | VW3 | Today view | 4 | 10 | Overdue → today → in progress; proper empty state |
| 4 | QC5 | Dark mode + density | 3 | 13 | Theme survives restart; readable in both |
| 5 | DS3 | Projects | 4 | 17 | Project page lists only its tasks + progress |
| 6 | DS2 | Tags | 5 | 22 | Join table; rename updates all; 2-tag filter = intersection |
| 7 | TH1 | Activity log | 6 | 28 | Every change logged in the same transaction |
| 8 | TH3 | Task history | 4 | 32 | 3 edits → 3 readable entries (uses TH1) |
| 9 | VW1 | Kanban | 7 | 39 | Dragged card stays after reload; failed write reverts |
| 10 | IS3 | Full-text search | 5 | 44 | FTS5, highlighted snippets, updates on edit |
| 11 | VW2 | Calendar | 7 | 51 | Drag to another day changes due date everywhere |
| 12 | DS1 | Subtasks | 6 | 57 | 3-level tree; ticking a leaf moves root progress |

<div class="note" markdown="1">
**Feature-specific warnings to give Qoder:**

- **QC5 dark mode:** persist the setting in the **database**, not localStorage. Default to the system preference.
- **VW1 Kanban / VW2 Calendar:** ask for native HTML drag-and-drop (no extra library), and make the card revert if the save fails.
- **IS3 search:** must use SQLite **FTS5** with triggers to keep the index updated, not `LIKE`.
- **QC3 seed:** make the seed run against the same db file and print how many rows it created. Add `"seed"` to package.json scripts.
</div>

### After every feature: the 60-second loop

1. Test the "Done when" in the browser.
2. Stop the dev server (Ctrl+C), start it again, refresh. Is the data still there?
3. Commit and push:

```
git add -A
git commit -m "feat: add tags with filtering"
git push
```

## 7. When things go wrong

| Problem | Fix |
| --- | --- |
| `better-sqlite3` fails to install / build | Check `node -v`. Ask Qoder: "Switch to Node's built-in node:sqlite module" (Node 22.5+). |
| `create-next-app` says folder not empty | `ls -a`, remove anything except `.git`, rerun. |
| Port 3000 in use | It'll offer 3001; or close the other terminal running dev. |
| `git push` rejected (non-fast-forward) | `git pull --rebase` then `git push`. |
| Push asks for password and fails | Username `Suhai1`, password = **token**, not your GitHub password. |
| App broken, can't fix in 5 min | `git restore . && git clean -fd` → back to last commit. Move on. |
| Need to undo the last commit too | `git reset --hard HEAD~1` (only if not pushed yet). |
| Qoder edits the wrong thing | Reject the change, new chat, use `@file` to point it at the right file. |
| Page shows old data / weird errors | Stop dev server, delete `.next` folder, `npm run dev` again. |

## 8. Final checklist (16:40, do every box)

- [ ] `git status` shows nothing to commit
- [ ] `git push` done; latest commit visible on GitHub
- [ ] README has run instructions **and** a features list with where to find each
- [ ] `start.sh` exists and is executable (`chmod +x start.sh`, then commit)
- [ ] `.gitignore` has `node_modules`, `.next` and `data/`
- [ ] **Fresh-clone test:** new terminal, then:

```
cd /tmp && rm -rf check && git clone https://github.com/Suhai1/coms3011a-test.git check
cd check && npm install && npm run dev -- -p 3005
```

- [ ] Open http://localhost:3005: app loads, you can create a task, no setup needed
- [ ] Final push before **16:55**, then hands off

### start.sh (copy into the repo root)

```
#!/usr/bin/env bash
set -e
npm install
npm run dev
```

## 9. Mistakes that cost marks

<div class="warn" markdown="1">
- Overdue stored as a column or added as a fourth status
- Archive done as DELETE or by copying rows elsewhere
- Recurrence giving 31 February or drifting an hour over DST
- Overdue computed in the server timezone, not the user's
- One query per task to fetch tags (N+1); no indexes
- Tests that mock the database
- Dependency graph walk without a cycle guard
- Migrations that drop and recreate tables
- Three libraries installed for one job
- **Private repo, or nothing pushed = 0**
</div>

## 10. Room rules

- Lab PCs only, no laptops, no talking.
- **Only Qoder.** No claude.ai, ChatGPT, Codex or other AI on the PC **or your phone**. A tutor may check your phone.
- This guide, GitHub, docs, YouTube and music are fine.
- Fill in the feedback form afterwards.

<div class="hero end" markdown="1">
**You've already proven the hard part works: you can clone, commit and push with your token.** Stay calm, keep tasks small, push often. Good luck, Suhail!
</div>

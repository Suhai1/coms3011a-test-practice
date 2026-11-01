# START HERE — COMS3011A Exam Pack

Read this file first, tonight. It tells you what every other file is for, when to open it, and what to do with it.

---

## What's in the pack

| File | What it is | When you use it | How you use it |
|---|---|---|---|
| `00-START-HERE.md` | This guide + tonight's checklist + exam-day timeline | Tonight, and glance at it at 08:00 | Just read it |
| `AGENTS.md` | **The rulebook for Qoder.** Your stack, folder layout, and the rules the AI must never break | Minute 1 of coding | Copy into the **root** of your new project. Qoder reads it automatically on every request |
| `COMMITS.md` | How to write commits, when to commit/push, and git rescue commands | Minute 1 (setup), then every time you finish a feature | Copy into the project root too (Qoder will follow it when it commits). Use the cheat sheet yourself |
| `DESIGN.md` | Your visual style: colours, fonts, spacing, banned "AI look" patterns | Minute 1, and again in the "design pass" prompt | Copy into the project root. Pick ONE of the two directions tonight and delete the other |
| `PROMPTS.md` | Copy-paste prompt templates for every stage of the day | All day | Keep it open in a side tab. Fill in the `[brackets]` and paste into Qoder |
| `STACK-A-NEXTJS.md` | **Main stack.** Next.js + TypeScript + Tailwind + SQLite. Setup commands, folder layout, database code, tests, firewall fixes | First 30 min of coding | Follow the setup steps top to bottom |
| `STACK-B-PYTHON.md` | **Backup stack.** Python + FastAPI + SQLite + HTML templates | Only if Stack A won't install/build | Follow it instead of Stack A |
| `QODER-GUIDE.md` | How to actually drive Qoder: modes, context, accepting changes, avoiding loops | Tonight (10-min practice) | Read it, then do the practice drill at the bottom |
| `DATABASES-AND-DOCKER.md` | Plain-English SQL, SQLite, Postgres and Docker | Tonight (skim), during the exam when you need to inspect data | Skim tonight; use the cheat sheets on the day |

---

## The one idea behind all of this

The AI is very good at **doing** and bad at **guessing what you want**. Every file here exists to stop it guessing:

- `AGENTS.md` stops it guessing **how** to build (stack, rules).
- `DESIGN.md` stops it guessing **what it looks like**.
- `PROMPTS.md` stops *you* writing vague prompts when you're tired at 15:00.
- `COMMITS.md` makes sure that when it breaks something (it will), you can go back.

---

## Tonight's checklist (≈ 75 minutes, then SLEEP)

Tick these off in order. If one gets stuck for more than 10 minutes, skip it.

**1. Laptop basics (10 min)**
- [ ] `node -v` in Terminal → must be **22.13 or higher** (or 24.x). If not: `fnm install 22 && fnm default 22`
- [ ] `python3 --version` → 3.11 or higher (for the backup stack)
- [ ] `git --version` works
- [ ] `sqlite3 --version` works (built into macOS)

**2. GitHub works (10 min)** — this is the thing that ruins people in the last 5 minutes
- [ ] Make a throwaway repo on github.com, clone it, add a file, `git commit`, `git push`. If push asks for a password and fails, fix it now (see `COMMITS.md` → "Make sure push works").
- [ ] Delete the throwaway repo afterwards.

**3. Warm your caches (15 min)** — so installs work behind the firewall
- [ ] Follow **"Tonight: warm-up run"** in `STACK-A-NEXTJS.md`. It creates a project, installs everything once (including the SQLite driver's prebuilt binary) and then you delete the project. The downloads stay cached on your laptop.
- [ ] Do the same short warm-up in `STACK-B-PYTHON.md`.
- [ ] (Optional, only if Docker Desktop is already installed) `docker pull postgres:16-alpine`

**4. Qoder (15 min)**
- [ ] Open Qoder, make sure you're logged in.
- [ ] Do the practice drill at the bottom of `QODER-GUIDE.md`.

**5. Personalise (15 min)**
- [ ] `DESIGN.md`: choose Direction 1 or 2 (or change the colours), delete the other.
- [ ] Read `PROMPTS.md` once so you know what's there.

**6. Upload (5 min)**
- [ ] Upload all the `.md` files (or the zip) to the Moodle file area the lecturer mentioned, so you can download them in the exam.
- [ ] Also email them to yourself as a backup.

**7. Pack your bag:** laptop charger, phone charger, USB-C hub, earphones, water, snacks. Alarm for early arrival — aim to be at MSL by about 07:30.

---

## Exam-day timeline (08:00–17:00)

### 08:00–08:30 — Planning (NO coding)
1. **2 minutes of nothing.** Seriously.
2. Read the brief twice. Highlight every "must", "required", "done when".
3. Write `BRIEF.md` (just paste the brief into a text file — you'll give it to Qoder later).
4. On paper: list the **core** features, then extra features ranked by **marks ÷ effort**.
5. Sketch the data model on paper: tables, columns, which ones link to which.
6. Check the allowed-domains list. Decide: Stack A (default) or Stack B.
7. Use **PROMPTS.md → Prompt 0** in Qoder's **Ask mode** to sanity-check your plan (asking questions isn't coding).

### 08:30–09:30 — Setup + core
- Create the project (stack file), copy in `AGENTS.md`, `COMMITS.md`, `DESIGN.md`, `BRIEF.md`.
- First commit + push **before** writing any feature. (`COMMITS.md`)
- Build the core features one at a time with **Prompt 2**. Commit after each.

### Every hour, on the hour (set phone alarms for 10:00, 11:00, … 16:00)
1. `git push`
2. Click through the **whole** app like a user: start to finish.
3. Run the tests.
4. If anything is broken, fix it **before** starting a new feature.

### Lunch (1 hour)
Eat away from the screen. Before you leave: push. When you come back: walk through the app first.

### Feature blitz (until ~16:00)
- One feature at a time. Plan → build → check → commit.
- Every 2–3 features, run **Prompt 5 (critic review)**.
- One **design pass** with **Prompt 8** somewhere mid-afternoon.

### 16:00 — Feature freeze
- No new features after 16:00. Only fixes.
- **Prompt 10 (final checklist).** README with how to run it.
- 16:30: final full walkthrough, tests, push. Check on github.com that the latest commit is actually there.
- 16:45: done. Nothing in flight.

---

## When to switch to the backup stack

Switch to **Stack B** only if, in the first 30–40 minutes of coding:
- `npm install` fails or hangs and the fixes in `STACK-A-NEXTJS.md` → "Firewall problems" don't work, **or**
- the Next.js dev server will not start.

**Never switch stacks after 10:00.** At that point fix forward.

The database has its own fallback *inside* Stack A (better-sqlite3 → built-in `node:sqlite`), so a database install problem alone is **not** a reason to switch stacks.

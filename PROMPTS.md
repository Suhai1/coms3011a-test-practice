# PROMPTS.md — Copy-paste prompts for the whole day

How to use: copy a prompt, replace everything in `[square brackets]`, paste into Qoder.
Tip: start a **new chat** for each new feature (keeps the AI's context clean — old failed attempts in the chat make it worse, "context rot").

| # | Prompt | When | Qoder mode |
|---|---|---|---|
| 0 | Plan check | 08:00–08:30 planning | Ask |
| 1 | Scaffold | First thing when coding starts | Agent |
| 2 | Build one feature | Every feature | Agent |
| 3 | Fix a bug | Something's broken | Agent |
| 4 | Explain | You don't understand code/error | Ask |
| 5 | Critic review | Every 2–3 features | Ask (new chat) |
| 6 | Tests | After core, and after tricky features | Agent |
| 7 | Seed data | After core works | Agent |
| 8 | Design pass | Mid-afternoon | Agent |
| 9 | Loop breaker | AI going in circles | Agent |
| 10 | Final checklist | 16:00 | Agent |

---

## 0. Plan check (Ask mode — no code)
```
Here is the exam brief: @BRIEF.md
And my project rules: @AGENTS.md

Do NOT write any code. I want a plan only.
1. Summarise the required (core) features in one line each, quoting any "done when" conditions.
2. List the rules/constraints from the brief that are easy to break by accident.
3. Propose a database schema (tables, columns, types, relationships, indexes) as SQL CREATE TABLE statements.
4. List the pages/views and the API routes I'll need.
5. Split the work into small, independent tasks in build order, each checkable in under a minute.
Point out anything ambiguous in the brief.
```

## 1. Scaffold (Agent)
```
Read @AGENTS.md, @DESIGN.md and @BRIEF.md.
The Next.js project already exists. Set up the foundation only — no features yet:
1. src/lib/db.ts and src/lib/migrations.ts exactly as described in AGENTS.md, with this schema as migration 1:
[paste the CREATE TABLE statements from your plan]
2. Apply the DESIGN.md tokens in src/app/globals.css and a simple layout with a top nav: [list nav items].
3. A placeholder home page that reads from the database and shows an empty state.
4. Vitest set up with one passing test that creates a row in an in-memory DB.
Then run the build and the tests and tell me the results.
```

## 2. Build one feature (Agent) — the one you'll use most
```
Feature: [name]
What it does: [1–3 sentences, in user terms]
Done when: [the exact check, e.g. "I archive a task, it disappears from the list, and I can still find it under Archived"]
Constraints to respect: [e.g. "never delete rows — archive with archived_at", "status is one of Todo/InProgress/Complete only"]

First, list the files you will change and how (no code yet). Wait for my OK.
Then implement it: logic and SQL in src/lib, thin route/page, UI following DESIGN.md.
Add/extend tests for the logic. Run build + tests and report results.
```
> After it finishes: **click through the feature yourself.** Then commit: `feat(scope): ...`

## 3. Fix a bug (Agent) — be specific, not "it's broken"
```
Bug: [what happens]
Expected: [what should happen]
Steps to reproduce: [1, 2, 3]
Error message (if any): [paste it exactly]
Where I think it is: [file or "not sure"]

Find the root cause first and explain it in 2–3 sentences before changing anything.
Then make the smallest fix. Add a test that would have caught this. Run tests.
```

## 4. Explain (Ask)
```
Explain @[file] (or this error: [paste]) like I'm a final-year CS student who hasn't seen this library before.
What does it do, why is it done this way, and what could go wrong? Keep it short.
```

## 5. Critic review (Ask — start a NEW chat so it's a fresh reviewer)
```
You are a strict code reviewer and examiner. Review this project against @AGENTS.md and @BRIEF.md.
Do NOT change any code. Give me a list, most serious first:
1. Any rule in AGENTS.md that's violated (deleting rows, stored derived values, extra enum values, N+1 queries, missing indexes, no cycle guards, drop-and-recreate migrations, duplicate libraries).
2. Any "done when" condition in the brief that the current code would FAIL in a live demo.
3. Bugs and unhandled edge cases (empty input, invalid input, dates/timezones, restart persistence).
4. Tests that are fake (mocked DB, only check rendering).
For each: file, line/area, why it matters, suggested fix in one sentence.
```
> Then fix the top items one at a time with Prompt 3.

## 6. Tests (Agent)
```
Write behaviour tests for [feature/area] in src/tests/[name].test.ts.
Use the real in-memory SQLite database (no mocks). Each test creates its own data.
Must cover: normal case, empty/invalid input, [awkward cases: e.g. archive semantics, month-end dates, cycle rejection, timezone boundary].
Run them twice in a row and show me they pass both times.
```

## 7. Seed data (Agent)
```
Create scripts/seed.ts and an npm script "seed" that fills the database with [~200] realistic records for [describe domain].
Realistic names/titles (not "Task 1"), a spread of dates (past, today, future), all statuses, and some relationships.
It must use the lib functions (not raw inserts that bypass rules) and be safe to run on an empty database.
```

## 8. Design pass (Agent)
```
Do a visual pass on [page/all pages] following @DESIGN.md exactly.
- Use only the colour tokens and spacing scale in DESIGN.md.
- Remove anything on the BANNED list.
- Make every list have an empty state, loading state and error state.
- Visible keyboard focus on all interactive elements.
Do not change any behaviour or logic. List what you changed.
```

## 9. Loop breaker (Agent)
```
Stop. We've tried this [N] times and it's not working.
Don't write code yet. Explain:
1. What you think the root cause is.
2. What assumption you've been making that might be wrong.
3. Two different approaches, and which you recommend.
```
> If that still doesn't work: `git stash`, open a **new chat**, and re-ask with Prompt 2 including what you learned.

## 10. Final checklist (Agent, at 16:00)
```
Final pre-submission check. Do not add features.
1. Run the build and all tests; fix anything failing.
2. Write/update README.md: what the app does, features implemented, exact commands to install, run, seed and test.
3. Check .gitignore excludes node_modules, .next, data/*.db, .env.
4. Remove debug code, console.logs and dead files.
5. List any feature that is half-done so I can decide whether to hide it.
```

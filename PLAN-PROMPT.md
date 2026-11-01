# PLAN-PROMPT.md — The master planning prompt + the task prompt

## How to use
1. **08:00–08:30 (planning time, no coding):** open Qoder → new chat → **Ask** mode → model **Ultimate**.
2. Copy everything in the MASTER PLAN PROMPT box below. Paste the full exam brief at the very end, where it says `PASTE THE FULL BRIEF BELOW THIS LINE`. Send.
3. Read the plan. Correct anything wrong ("task 7 is in the wrong order", "the brief says X, not Y") until you're happy.
4. **At 08:30:** switch to **Agent** mode and say: *"Save the final plan exactly as written to PLAN.md in the project root."* (Or copy-paste it into a `PLAN.md` file yourself.)
5. Then, for every task: **new chat** → **Agent** mode → model **Auto** → paste the TASK PROMPT with the task number.

---

## MASTER PLAN PROMPT (copy everything inside the box)

```
You are my senior engineer and exam strategist. I have a 7.5-hour practical software exam, building an application with you, an AI coding agent. It is marked LIVE by a demonstrator at the end, against the brief's requirements and "done when" conditions. Your job right now is ONLY to plan. Do NOT write application code, and do not create or edit files.

## My environment (fixed: plan around it)
- Ubuntu lab PC, Node 18.19.1, firewalled internet (npm registry and docs are reachable; assume CDNs, Google Fonts and most other sites are blocked).
- Stack: Next.js 15.5.27 (App Router) + TypeScript (strict) + Tailwind 4.1.18 (pinned) + SQLite via better-sqlite3 11.10.0 (fallback: sql.js), Vitest 3.2.4 for tests. Raw SQL with ? placeholders, no ORM.
- Every package must support Node 18. No external services, no auth providers, no CDN scripts.
- Business logic lives in src/lib/<resource>.ts; API routes and pages are thin and call lib functions.
- My full rules are in AGENTS.md (I'll attach it during the build). Key ones: never delete rows (archive with archived_at), never store derived values, fixed enums enforced with CHECK constraints, versioned migrations (never drop/recreate), index filtered and sorted columns, no N+1 queries, cycle guards on any graph/tree, tests against an in-memory database (never mocked).

## How I will work
- One task per fresh chat. Each task must be finished, verified in the browser, tested and committed before the next starts.
- I will paste a task prompt like: "@PLAN.md @AGENTS.md Do task N only."
- So every task must be self-contained, small (10–25 minutes of agent work), and checkable by me in under 60 seconds.

## Produce the plan in EXACTLY this structure (markdown):

### 1. The brief in one paragraph
What the app is, who uses it, and how it will be marked.

### 2. Marks map
A table: requirement / feature | marks or weight (quote the brief) | required or optional | "done when" condition quoted word-for-word from the brief. If the brief gives no marks, rank by importance.

### 3. Hard rules and traps
Every constraint in the brief that an AI agent is likely to break by accident (e.g. deletion, stored derived values, fixed lists, time zones, persistence across restart). One line each, quoting the brief.

### 4. Data model
Complete SQLite CREATE TABLE and CREATE INDEX statements for migration 1, with a one-line reason for each table. Include archived_at, created_at, updated_at, CHECK constraints for fixed enums, foreign keys, and join tables for many-to-many.

### 5. Screens and API
- Pages/views: route, what's on it, which tasks build it.
- API routes: method + path, input, output, error cases.

### 6. Task list (the most important part)
Numbered tasks in build order. Rules:
- Task 1 = foundation: db.ts, migrations, design tokens from DESIGN.md, layout/nav, one passing test. No features.
- Then ALL required/core features, one per task, smallest working version first.
- Then optional features in order of (marks ÷ effort), highest first.
- Insert a "VERIFY" task after every 3–4 build tasks: run all tests, click through the full user journey, fix regressions only.
- Put a clear line "---- MINIMUM PASSING VERSION ENDS HERE ----" after the last core task.
- Final tasks: seed data with realistic content, design pass against DESIGN.md, README with run instructions, final check.

For EACH task use this exact format:
**Task N — <name>** (est. <minutes> min, <marks> marks, depends on: <task numbers or none>)
- Goal: <one sentence, user-visible outcome>
- Files: <files to create or change>
- Rules to respect: <the specific AGENTS.md / brief rules this task could break>
- Done when: <a concrete check I can do in the browser or terminal in under 60 seconds, mirroring the brief's wording where possible>
- Tests: <the behaviour tests to add, including the awkward case>

### 7. Time budget
Map the tasks onto the day: 08:30 start, 1-hour lunch around 12:30–13:30 (assume this unless the brief says otherwise), feature freeze at 16:00, finished by 16:45. Mark where I should be by 10:30, 12:30 and 15:00. If the task list can't fit, say which optional tasks to drop.

### 8. Risks and ambiguities
- Anything unclear or contradictory in the brief, with the interpretation you recommend and why (I may ask the lecturer).
- Anything that may not work on Node 18 or behind the firewall, with a fallback.

Be specific to THIS brief. No generic advice. If something isn't in the brief, don't invent requirements; list it under ambiguities instead.

=== PASTE THE FULL BRIEF BELOW THIS LINE ===

```

---

## TASK PROMPT (use for every task: new chat, Agent mode, Auto)

```
@PLAN.md @AGENTS.md @DESIGN.md
Do Task [N] from PLAN.md, and only that task.

1. First, restate the task's Goal, Rules to respect and Done-when in 3 lines, and list the files you'll change. Wait for my OK.
2. Then implement it. Logic and SQL in src/lib, thin routes/pages, UI following DESIGN.md.
3. Add the tests listed for this task. Run `npm run build` and `npm test` and show me the results honestly.
4. Tell me exactly how to check the Done-when myself (what to click / what to run).
5. When I confirm it works, tick Task [N] in PLAN.md as [x].
Do not start the next task. Do not change unrelated code. Do not install packages without asking (Node 18, pinned versions).
```

**After every task:** check it yourself → `git add . && git commit -m "feat(scope): ..."` → new chat → next task.

## VERIFY TASK PROMPT (when the plan says VERIFY)

```
@PLAN.md @AGENTS.md
VERIFY checkpoint. Don't add features.
1. Run npm run build and npm test; report results.
2. List every completed task's Done-when check from PLAN.md so I can click through them all now.
3. Review the code changed since the last checkpoint against AGENTS.md section 3 (database rules) and list any violations.
Fix only real breakages, one at a time, with my OK.
```

## IF THE PLAN NEEDS TO CHANGE (running behind, brief misread)

```
@PLAN.md
It's [time]. Tasks [x, y] are done. I'm behind / [what changed].
Re-plan the remaining tasks to finish by 16:00. Keep all core tasks. Drop or shrink optional tasks with the lowest marks ÷ effort, and tell me what you dropped. Update PLAN.md.
```

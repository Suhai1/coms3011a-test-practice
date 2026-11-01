# QODER-GUIDE.md — How to actually drive Qoder

Qoder is a code editor (it looks and works like VS Code) with an AI agent built in. You describe what you want; it reads your files, writes code, and runs terminal commands. **Your job is to direct and check, not to type.**

> Shortcuts below are for the **Ubuntu lab PCs** (Ctrl). On your Mac tonight, use Cmd where it says Ctrl.
> Menus move between versions. If a button isn't exactly where this says, look for the same word nearby, or press **Ctrl+Shift+P** and type what you want (this command search works for everything).

---

## 1. The basics

| Want to... | Do this |
|---|---|
| Open your project | File → Open Folder → pick your project folder (always open the **folder**, not a single file) |
| Open the AI chat panel | **Ctrl+L** (or the chat icon at top-right) |
| Open a terminal | **Ctrl+`** (backtick), or Terminal → New Terminal |
| Start a fresh chat | The **+** / new chat button in the chat panel, or type `/newChat` |
| See old chats | History (clock) icon in the chat panel |
| Find any command | **Ctrl+Shift+P** |
| Search files by name | **Ctrl+P** |

## 2. The two chat modes — pick the right one

| Mode | What it does | Use it for |
|---|---|---|
| **Ask** | Answers questions. Doesn't change your files. | Planning (allowed in the 30-min planning time), understanding errors, "explain this file", code review (Prompt 5) |
| **Agent** | Plans, edits multiple files, runs commands in the terminal. | Building features, fixing bugs, writing tests |

The mode switch is in the chat input box (a dropdown near the send button). If you see a **Quest** mode, that's for long fully-autonomous tasks — don't rely on it in the exam; small Agent tasks you can check are safer.

If there's a **model picker** next to the mode switch, use the strongest model available for planning and tricky bugs.

## 3. Giving it context with `@`
Type `@` in the chat box to attach things:
- `@AGENTS.md`, `@BRIEF.md`, `@DESIGN.md` — attach a file
- `@src/lib/items.ts` — attach the exact file you're talking about
- folders, rules, terminal output, images (screenshots of bugs work well)

**Always attach the specific file(s) involved.** "Fix the archive bug in @src/lib/items.ts" is much better than "fix the archive bug".

## 4. Rules: making it follow AGENTS.md every time
- Qoder automatically reads **`AGENTS.md` in the project root** — just copy the file in. No setup.
- Extra (optional): Settings (**Ctrl+Shift+,** → Qoder Settings) → **Rules** → Add. Rule types:
  - **Always Apply** — added to every request (good for "never delete rows").
  - **Manual** — only when you type `@rule` in chat.
  - **Model Decision** — the AI decides when it's relevant.
  - **Specific Files** — for files matching a pattern like `src/**/*.ts`.
- Even with rules, **restate the key constraint in the prompt** for features that touch it (PROMPTS.md does this for you).

## 5. Reviewing what it did
When the agent edits files, you'll see a list of changed files with diffs (green = added, red = removed).
- **Accept all:** Ctrl+Enter (or the Accept button)
- **Reject all:** Ctrl+Backspace (or Reject)
- You can open each file and accept/reject individual changes.

What to check in 30 seconds before accepting:
1. Did it touch files you didn't expect? (e.g. you asked for a button and it changed `db.ts`)
2. Any `DELETE FROM`, new dependencies in `package.json`, or a new status value? Those are red flags.
3. Then actually **run/click the feature**.

## 6. Terminal commands
In Agent mode it will want to run things like `npm install` or `npm test`. It shows the command and asks to run it.
- Read the command before approving. Approve `npm test`, `npm run build`, `npm install <thing you agreed to>`.
- Be careful with: `rm -rf`, `git reset --hard`, `git push --force`, anything deleting `data/`.
- If the dev server is already running in your terminal, tell it: *"The dev server is already running on port 3000, don't start another."*
- If it tries `npm install something@latest` or `npm audit fix --force`, **reject it**. On Node 18 that breaks the build. Say: *"Check AGENTS.md section 1: pinned versions, Node 18."*

## 7. The working loop (do this all day)
```
1. New chat for the feature
2. Prompt 2 (from PROMPTS.md) — it lists the plan → you say OK or correct it
3. It builds → you review the diff → accept
4. You click through the feature in the browser + run tests
5. Works? → git commit.  Broken? → Prompt 3 with the exact symptom
6. Next feature
```

## 8. Avoiding the classic agent traps
| Trap | What to do |
|---|---|
| Long chat, AI getting worse and repeating old mistakes ("context rot") | Start a **new chat**. Re-attach the relevant files. |
| It "fixes" something 3 times without success | Prompt 9 (loop breaker). If still stuck: `git stash`, new chat. |
| It says "Done! Everything works" | Don't believe it. Run it yourself. |
| It rewrites half the app for a small change | Reject, then: "Only change X in @file. Don't touch anything else." |
| It installs a library you didn't ask for | Reject and say "Don't add dependencies; use what's installed." |
| It breaks an earlier feature | Run tests after every feature; walk through the app hourly. |
| "That's wrong" → it guesses differently | Be specific: "You stored overdue in a column; compute it from due_date when reading instead." |

## 9. Running things in parallel (only when safe)
You can have two chats going at once, but only for features that **don't touch the same files** (e.g. a new page vs. the seed script). Anything that changes the database schema or the same lib file → one at a time. When in doubt, sequential.

## 10. If Qoder itself breaks
- Network error in chat → wait 30 s, retry. Check the terminal still works.
- Totally down → you can still run, test, commit with the terminal. Use Gemini (if available in the browser) as an Ask-mode helper and paste code by hand.

---

## Practice drill tonight (10 min)
1. Create an empty folder `qoder-practice`, open it in Qoder.
2. Copy `AGENTS.md` into it.
3. Open chat (Ctrl+L), **Ask** mode: *"Read @AGENTS.md and summarise the 5 most important rules in one line each."* → confirms it reads the file.
4. Switch to **Agent** mode: *"Create a Python file hello.py that prints the current date. Then run it."* → watch it ask to run a command; approve it.
5. Practise **reject**: ask it to "rename the function to greet", then reject the change.
6. Type `@` and see the context options.
7. Start a new chat. Delete the folder. Done — you've used every move you need.

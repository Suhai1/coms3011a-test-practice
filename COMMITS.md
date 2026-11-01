# COMMITS.md — Commit style and git safety net

## Commit message format (Conventional Commits)

```
<type>(<scope>): <what changed, present tense, lowercase, no full stop>
```

| type | use it for | example |
|---|---|---|
| `feat` | a new feature the user can see | `feat(tasks): add archive button` |
| `fix` | fixing a bug | `fix(tasks): show archived tasks in search` |
| `test` | adding/changing tests only | `test(tasks): cover archive edge cases` |
| `style` | visual/CSS only | `style(ui): apply design tokens to list view` |
| `refactor` | restructuring code, no behaviour change | `refactor(db): move queries into lib/tasks` |
| `chore` | setup, config, dependencies | `chore: scaffold next app with sqlite` |
| `docs` | README, comments | `docs: add run instructions` |

Rules:
- Scope = the area you touched (`tasks`, `db`, `ui`, `api`, `search`...). Optional but helpful.
- Under ~70 characters. Say **what changed**, not "updates" or "stuff".
- One commit = one logical change. If you need the word "and", it might be two commits.

## When to commit and push

| Moment | Do this |
|---|---|
| Project created, before any feature | commit + **push** (proves push works) |
| A feature works and tests pass | commit |
| About to ask the AI for something big/risky | commit first (so you can undo) |
| Every hour on the hour | **push** |
| Before lunch, and at 16:30 | **push**, then check github.com shows it |

---

## Setup at the start of the exam

**0. Log the lab PC into git (once, first thing):**
```bash
git config --global user.name "Suhail Seedat"
git config --global user.email "<the email on your GitHub account>"
git config --global credential.helper 'cache --timeout=36000'   # remembers your token in memory for 10 hours, not on disk
```
The first `git push` asks for a username and password. Username = your GitHub username. **Password = your Personal Access Token**, not your GitHub password (see "Make sure push works" below). After that, the cache remembers it for the day.

1. On github.com → **New repository** → name it → leave it **empty** (no README) → Create.
2. In your project folder:
```bash
git init
git add .
git commit -m "chore: scaffold project"
git branch -M main
git remote add origin https://github.com/<you>/<repo>.git
git push -u origin main
```
3. Refresh github.com — you should see your files. Done.

Make sure `.gitignore` contains:
```
node_modules/
.next/
data/*.db
data/*.db-*
.env
.venv/
__pycache__/
```

## Make sure push works (do this tonight)
The lab PC isn't logged into your GitHub, and GitHub doesn't accept your account password on the command line. You need a **Personal Access Token (PAT)**:

1. github.com → your avatar → **Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)** → **Generate new token (classic)**.
2. Note: `COMMS3011A exam`. Expiration: **7 days**. Scope: tick **`repo`**. Generate.
3. **Copy the token** (starts with `ghp_`) into your phone's notes or password manager. GitHub only shows it once.
4. Tomorrow, paste it as the "password" on the first `git push`. On Linux terminals, paste is **Ctrl+Shift+V**. Nothing appears while you paste a password; that's normal.

**At 17:00, before you leave:** run `git credential-cache exit` so the next person on that PC can't push as you. After the exam, delete the token on GitHub.

---

## Everyday cheat sheet

```bash
git status                      # what has changed?
git add .                       # stage everything
git commit -m "feat(x): ..."    # save a snapshot
git push                        # upload to GitHub
git log --oneline -10           # last 10 commits
git diff                        # see unstaged changes line by line
```

## Rescue commands (when the AI broke something)

| Situation | Command | Safe? |
|---|---|---|
| Undo uncommitted changes to ONE file | `git restore path/to/file` | loses that file's changes |
| Park all uncommitted changes, get back to last commit | `git stash` (bring back: `git stash pop`) | safe — nothing lost |
| Throw away ALL uncommitted changes | `git restore .` then `git clean -fd` (deletes new untracked files) | loses uncommitted work |
| Undo a commit that's already pushed | `git revert <commit-hash>` then `git push` | safe — makes a new "undo" commit |
| "What did it look like 3 commits ago?" | `git log --oneline` then `git show <hash>` | read-only |

**Rule of thumb:** if the AI has gone in circles for 10 minutes, `git stash` and start that feature again with a better prompt. It's faster than arguing.

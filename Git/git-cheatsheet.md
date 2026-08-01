# Git Cheatsheet — From Zero to Comfortable

## The Mental Model

Three areas your files move through:

```
Working Directory  →  Staging Area  →  Repository (.git)
   (edit files)          (git add)        (git commit)
```

- **Working directory** — your actual files, as you're editing them
- **Staging area** — a waiting room for changes you're about to save
- **Repository** — the permanent history of snapshots (commits)

Git ≠ GitHub. Git runs on your machine and tracks history. GitHub is a website that hosts Git repos online for sharing/collaboration.

---

## Setup (one-time)

```bash
git --version                              # check it's installed
sudo apt install git                       # install on Debian/Ubuntu if missing

git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main   # use 'main' instead of 'master'
git config --list                          # see all your settings
```

---

## Starting a Project

```bash
git init                        # turn current folder into a git repo
git clone <url>                 # copy an existing remote repo to your machine
```

---

## The Daily Loop

This is 90% of what you'll actually use:

```bash
git status                      # what's changed? what's staged?
git add <file>                  # stage a specific file
git add .                       # stage everything changed
git add -p                      # stage changes interactively, chunk by chunk
git commit -m "message"         # save a snapshot with a message
git commit -am "message"        # stage all tracked files AND commit (skips git add)
git log                         # view commit history
git log --oneline               # compact one-line-per-commit view
git log --oneline --graph --all # visual branch history
```

**Writing good commit messages:** short summary line (imperative mood: "Fix bug" not "Fixed bug"), under ~50 chars, blank line, then details if needed.

---

## Checking What Changed

```bash
git diff                        # unstaged changes vs last commit
git diff --staged               # staged changes vs last commit
git diff <commit1> <commit2>    # compare two commits
git show <commit>               # show what a specific commit changed
```

---

## Undoing Things

| Situation | Command |
|---|---|
| Unstage a file (keep changes) | `git restore --staged <file>` |
| Discard changes in working dir | `git restore <file>` |
| Change the last commit message | `git commit --amend -m "new message"` |
| Undo last commit, keep changes staged | `git reset --soft HEAD~1` |
| Undo last commit, keep changes unstaged | `git reset HEAD~1` |
| Undo last commit, discard changes ⚠️ | `git reset --hard HEAD~1` |
| Revert a commit safely (new commit that undoes it) | `git revert <commit>` |

⚠️ = destructive, can lose work. Everything else is safe.

**Rule of thumb:** `reset` rewrites history (don't use on shared/pushed commits). `revert` adds a new commit undoing an old one (safe for shared history).

---

## Branching

Branches let you work on something without touching your main code.

```bash
git branch                      # list branches
git branch <name>               # create a new branch
git switch <name>                # switch to a branch
git switch -c <name>              # create AND switch in one step
git branch -d <name>            # delete a branch (safe, only if merged)
git branch -D <name>            # force delete (⚠️ even if unmerged)
```

Older equivalent you'll see in tutorials: `git checkout <name>` and `git checkout -b <name>` — same thing, `switch` is the newer, clearer command.

---

## Merging

```bash
git switch main                 # go to the branch you want to merge INTO
git merge <branch>              # bring <branch>'s changes into current branch
```

If both branches changed the same lines, you get a **merge conflict**. Git marks it in the file like:

```
<<<<<<< HEAD
your version
=======
their version
>>>>>>> branch-name
```

Edit the file to keep what you want, delete the markers, then:

```bash
git add <file>
git commit
```

---

## Working with Remotes (GitHub, GitLab, etc.)

```bash
git remote add origin <url>     # connect local repo to a remote
git remote -v                   # see connected remotes
git push origin main            # upload your commits
git push -u origin main         # upload + remember this as the default (only need -u once)
git pull                        # download + merge latest changes
git fetch                       # download changes WITHOUT merging (safer, look first)
```

**`pull` vs `fetch`:** `fetch` shows you what changed remotely without touching your files. `pull` = `fetch` + `merge` immediately. When unsure, `fetch` then look, then decide.

---

## Stashing (save work without committing)

Useful when you need to switch branches but aren't ready to commit.

```bash
git stash                       # save uncommitted changes, clean working dir
git stash list                  # see stashed changes
git stash pop                   # reapply the most recent stash and remove it
git stash apply                 # reapply without removing from stash list
git stash drop                  # delete a stash without applying it
```

---

## Ignoring Files

Create a `.gitignore` file in your repo root and list patterns to skip:

```
node_modules/
*.log
.env
__pycache__/
```

Files matching these patterns won't show up in `git status` or get staged by `git add .`.

---

## Common Beginner Mistakes & Fixes

| Problem | Fix |
|---|---|
| Committed to the wrong branch | `git reset --soft HEAD~1`, switch branch, recommit |
| Forgot to add a file to last commit | `git add <file>` then `git commit --amend --no-edit` |
| Accidentally committed a secret/password | Remove it, commit, then rotate the secret immediately (history still has it until rewritten) |
| Merge conflict panic | Breathe. Edit the conflicted file, remove `<<<<`/`====`/`>>>>` markers, `git add`, `git commit` |
| "detached HEAD" message | You checked out a commit instead of a branch. `git switch main` (or your branch) to get back |

---

## Quick Reference: Command Cheat Sheet

```bash
git init                    # start tracking a folder
git clone <url>             # copy a remote repo
git status                  # what's going on right now
git add <file>              # stage a file
git commit -m "msg"         # save a snapshot
git log --oneline           # view history
git branch                  # list branches
git switch -c <name>        # new branch + switch to it
git merge <branch>          # merge branch into current
git push                    # upload commits
git pull                    # download + merge commits
git stash                   # temporarily shelve changes
git diff                    # see unstaged changes
```

---

## Suggested Practice Path

1. `git init` a test folder, make a file, `add` + `commit` it
2. Edit the file, `git diff`, then `add` + `commit` again
3. Check `git log --oneline` to see both commits
4. Create a branch, make a change, commit, `switch` back to main, `merge` it
5. Deliberately create a merge conflict (edit the same line on two branches) and resolve it
6. Connect to a real GitHub repo and practice `push`/`pull`

# Git Submodule Error: `fatal: in unpopulated submodule`

## The Error
```
fatal: in unpopulated submodule 'c-project-gomgom'
```

## What's Happening
`c-project-gomgom` is registered as a **git submodule** — a pointer to a specific commit in a separate repository — not a normal folder. When a repo with submodules is cloned, git creates the folder but doesn't automatically populate it with files. An "unpopulated" submodule is just an empty placeholder directory.

This also explains why `git add .` doesn't stage the folder's contents: git **never** tracks a submodule's files individually from the parent repo. It only ever stages one thing — a pointer to a commit SHA in the submodule's own repo.

## Case 1: Submodule is Unpopulated (matches the error)
Nothing has been checked out inside the folder yet.

**Fix:**
```bash
git submodule update --init --recursive
```
- `--init` initializes any submodules that were never set up
- `--recursive` handles nested submodules
- To scope to just this one: `git submodule update --init -- c-project-gomgom`

**Avoid this in future clones:**
```bash
git clone --recurse-submodules <url>
```

## Case 2: It's Populated, But You Don't Want It as a Submodule
If the goal is for `c-project-gomgom`'s files to be tracked as part of the main repo (not linked externally), it may have been added as a submodule by accident.

**Check if it's an intentional submodule:**
```bash
cat .gitmodules
```

**To fold it into the main repo as regular files:**
```bash
git rm --cached c-project-gomgom      # unstage the submodule reference
rm -rf c-project-gomgom/.git          # remove its embedded git repo
git add c-project-gomgom              # add it as normal files
```
Then remove its entry from `.gitmodules` if one exists.

## Quick Diagnostic
```bash
git submodule status
```
A `-` prefix before the commit hash confirms the submodule is uninitialized/unpopulated.

# Comparing Changes with Git

## Comparing Commits

**Compare two specific commits:**
```bash
git diff <commit1> <commit2>
```

**Compare a commit to its previous commit:**
```bash
git diff HEAD~1 HEAD
```

**See just the file names that changed, not the full diff:**
```bash
git diff --stat <commit1> <commit2>
```

**Compare a specific file across commits:**
```bash
git diff <commit1> <commit2> -- path/to/file.gd
```

**Compare working directory to a specific commit:**
```bash
git diff <commit-hash>
```

**See the full changeset a single commit introduced (vs. its parent):**
```bash
git show <commit-hash>
```

**Walk through di ffs of a range of commits one by one:**
```bash
git log -p
```

## Comparing Branches

**Basic branch comparison:**
```bash
git diff main feature-branch
```
Shows what would change if you merged `feature-branch` into `main` — i.e., full diff between the tips of both branches.

**Three-dot version (usually what you actually want):**
```bash
git diff main...feature-branch
```
Shows only the changes made *on* `feature-branch` since it diverged from `main` — ignores anything that happened on `main` in the meantime. This is almost always the more useful one when reviewing a feature branch, since it isolates just your work.

**Just the stat summary (files touched, +/- line counts):**
```bash
git diff --stat main...feature-branch
```

**See the actual commits that differ, not just line diffs:**
```bash
git log main..feature-branch          # commits on feature-branch not in main
git log --oneline --graph --all       # visual overview of all branches
```

**Compare a specific file between branches:**
```bash
git diff main feature-branch -- path/to/file.gd
```

**Check what a merge would actually bring in before doing it:**
```bash
git diff main...feature-branch
```

## Quick Reference

- Two dots (`..`): "what's different between these two tips right now"
- Three dots (`...`): "what happened on this branch since it split off from that one"
- For code review on a feature branch, three dots is almost always the one you want.
- For a more visual experience than raw terminal diffs, consider `git difftool` or a GUI diff viewer (VS Code, GitKraken, etc.)

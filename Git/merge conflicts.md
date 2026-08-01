# Syncing Your Branch with `main`

Your branch can't merge to `main` because it's behind by some number of commits. Follow the steps below to catch up.

## 1. Update your local `main`

```bash
git switch main
git pull
git switch YOUR_BRANCH_NAME_HERE
git merge main
```

## 2. If there are no merge conflicts

```bash
git push
```

You are now caught up with `main` and can merge if tests pass.

## 3. If there are merge conflicts

```bash
git status
```

Note all of the files that have conflicts, then go into each file and resolve them:

- Everything between `<<<<<<< HEAD` and `=======` is **your changes**
- Everything between `=======` and `>>>>>>> main` is **coming from `main`**

Once you're sure every file is fixed:

```bash
git add .
git commit -m "YOUR MESSAGE"
git push
```

You are now caught up with `main` and can merge if tests pass.

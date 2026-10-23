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

`git merge main` will stop and print something like:

```
CONFLICT (content): Merge conflict in src/player.gd
Automatic merge failed; fix conflicts and then commit the result.
```

Nothing is broken. Git has merged everything it could and is waiting for you to finish the rest.

### 3a. Find the conflicted files

```bash
git status
```

Look under **Unmerged paths**. Each conflicted file is marked `both modified` (or `deleted by us` / `deleted by them`, see 3d):

```
Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   src/player.gd
        both modified:   README.md
```

Files listed under **Changes to be committed** merged cleanly and need no attention.

To list only the conflicted files:

```bash
git diff --name-only --diff-filter=U
```

### 3b. Understand the conflict markers

Open each conflicted file. Every spot Git couldn't merge looks like this:

```
<<<<<<< HEAD
speed = 200
=======
speed = 250
>>>>>>> main
```

- Everything between `<<<<<<< HEAD` and `=======` is **your changes** (your branch)
- Everything between `=======` and `>>>>>>> main` is **coming from `main`**

A file can have several of these blocks. Search for `<<<<<<<` to find them all.

### 3c. Resolve each block

For every block, decide what the final code should be:

| Option | What to do |
| --- | --- |
| Keep yours | Delete the `main` side |
| Keep `main`'s | Delete your side |
| Keep both | Keep both sides, in the right order |
| Combine | Rewrite the lines by hand so they include both changes |

Then **delete all three marker lines** (`<<<<<<<`, `=======`, `>>>>>>>`). The result for the example above might just be:

```
speed = 250
```

Don't just pick a side automatically. Read what `main` changed: someone else's fix may need to be kept alongside yours. If you aren't sure why a line changed on `main`, check its history:

```bash
git log -p main -- path/to/file
```

**Shortcut for whole files:** if you want one side's version of an entire file:

```bash
git checkout --ours path/to/file     # keep your branch's version
git checkout --theirs path/to/file   # keep main's version
```

(This applies to `git merge`. During a `git rebase`, `--ours` and `--theirs` are swapped.)

Most editors (VS Code, JetBrains, etc.) highlight conflict blocks and offer "Accept Current / Accept Incoming / Accept Both" buttons, which do the same thing.

### 3d. Deleted-file conflicts

If `git status` shows `deleted by us` or `deleted by them`, one side deleted the file and the other changed it. Decide whether the file should exist:

```bash
git add path/to/file   # keep the file
git rm path/to/file    # delete the file
```

### 3e. Check that nothing was missed

Look for leftover markers before committing:

```bash
git diff --check
grep -rn '^<<<<<<<\|^=======\|^>>>>>>>' .
```

Then **build the project and run the tests locally**. A conflict can be resolved "cleanly" and still break the code.

### 3f. Mark resolved and commit

Stage each file you fixed. Staging is how you tell Git a conflict is resolved:

```bash
git add path/to/file
git status   # "Unmerged paths" should now be empty
```

Prefer adding files by name rather than `git add .`, so unrelated files don't slip into the merge commit.

Finish the merge:

```bash
git commit
```

Without `-m`, Git opens an editor with a ready-made message (`Merge branch 'main' into YOUR_BRANCH_NAME_HERE`) that lists the conflicted files. You can save it as is, or use `git commit -m "YOUR MESSAGE"` instead.

```bash
git push
```

You are now caught up with `main` and can merge if tests pass.

### Bailing out

If things get messy and you want to start over, this puts your branch back exactly as it was before `git merge main`:

```bash
git merge --abort
```

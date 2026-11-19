# SSH
ssh-keygen -t ed25519 -C "your_email@example.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
xclip -sel clip < ~/.ssh/id_ed25519.pub
# Setup
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# Start a repo
git init
git clone <repo_url>

# Check status & changes
git status
git diff                # unstaged changes
git diff --staged       # staged changes

# Stage & commit
git add <file>          # stage file
git add .               # stage all changes
git commit -m "msg"     # commit staged changes

# Undo changes
git restore <file>      # discard unstaged changes in file
git restore --staged <file> # unstage file
git reset --hard        # discard all local changes, reset to last commit

# Branching
git branch              # list branches
git branch <name>       # create branch
git checkout <branch>   # switch branch
git checkout -b <name>  # create & switch branch

# Merge & Pull
git merge <branch>
git pull                # fetch & merge remote changes
git push                # push commits to remote

# Log & history
git log
git log --oneline

# Remote
git remote -v           # list remotes
git fetch               # fetch remote changes (no merge)
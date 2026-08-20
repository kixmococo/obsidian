# The `.bashrc` File & Aliases — A Practical Guide

## What `.bashrc` actually is

`.bashrc` is a plain text file that lives in your home directory (`~/.bashrc`). Every time you open a new **interactive, non-login** shell — which in practice means every time you open a terminal window or tab — Bash reads this file and runs everything in it, top to bottom, as if you had typed it yourself.

That's the whole idea: it's a script that configures your shell before you start using it. Anything you find yourself typing at the start of every session probably belongs in here instead.

The name breaks down as **bash** + **rc**, where "rc" is an old Unix convention meaning "run commands" (it dates back to "run com" files in the 1960s). You'll see the same `rc` suffix on `.vimrc`, `.zshrc`, `.inputrc`, and many others.

### Where it fits among the other startup files

Bash has a few startup files and they trip people up constantly. Here's the short version:

- **`~/.bashrc`** — runs for interactive non-login shells (opening a normal terminal). This is the one you'll edit 95% of the time.
- **`~/.bash_profile`** (or `~/.profile`) — runs for **login** shells (logging in over SSH, or on the text console before a GUI starts). Common practice is to have `.bash_profile` simply source `.bashrc` so you get the same setup everywhere:

  ```bash
  # inside ~/.bash_profile
  if [ -f ~/.bashrc ]; then
      . ~/.bashrc
  fi
  ```

- **`/etc/bash.bashrc`** and **`/etc/profile`** — system-wide versions that apply to all users. Your personal `~/.bashrc` is the one you own and should edit for yourself.

If you set something and it works in one terminal but not another (e.g. works locally but not over SSH), this login-vs-non-login distinction is almost always why.

## What people put in `.bashrc`

A typical `.bashrc` collects a handful of things:

- **Aliases** — short names for longer commands (the focus of this guide).
- **Environment variables** — like `export EDITOR=vim` or adding a folder to your `PATH`.
- **Functions** — small reusable shell routines, for when an alias isn't powerful enough.
- **Prompt customization** — the `PS1` variable that controls what your prompt looks like.
- **Shell options** — tweaks to history behavior, tab completion, and so on.

## Aliases

### What an alias is

An alias is a shortcut. It tells Bash "whenever I type *this*, actually run *that* instead." It's the single most useful thing most people put in their `.bashrc`, because it turns commands you run dozens of times a day into two or three keystrokes.

### The basic syntax

```bash
alias name='command to run'
```

The rules that matter:

- **No spaces around the `=`.** `alias ll='ls -la'` works; `alias ll = 'ls -la'` does not.
- **Quote the right-hand side** whenever it contains spaces, flags, or special characters. Single quotes are the safe default.
- The `name` is what you'll type; everything inside the quotes is what actually runs.

### Making one on the fly (temporary)

You can define an alias directly in your terminal:

```bash
alias ll='ls -la'
```

This works immediately — but only for the current terminal session. Close the window and it's gone. That's fine for a quick experiment, but for anything you want to keep, it goes in `.bashrc`.

### Making one permanent (the real goal)

1. Open `.bashrc` in an editor:

   ```bash
   nano ~/.bashrc
   ```

2. Scroll to the bottom and add your alias on its own line:

   ```bash
   alias ll='ls -la'
   ```

3. Save and exit (in nano: `Ctrl+O`, `Enter`, then `Ctrl+X`).

4. Reload the file so the change takes effect in your current terminal without reopening it:

   ```bash
   source ~/.bashrc
   ```

   `source` (or its shorthand `.`) re-runs the file in your current shell. New terminals pick it up automatically; `source` is just so you don't have to open a new one.

### A batch of genuinely useful aliases

Drop any of these into your `.bashrc`. They're a good starting kit:

```bash
# Better listing
alias ll='ls -la'
alias la='ls -A'
alias l='ls -CF'

# Safer file operations — prompt before overwriting/deleting
alias cp='cp -i'
alias mv='mv -i'
alias rm='rm -i'

# Navigation shortcuts
alias ..='cd ..'
alias ...='cd ../..'
alias ....='cd ../../..'

# Quick edit + reload of this very file
alias bashrc='nano ~/.bashrc'
alias reload='source ~/.bashrc'

# Human-readable disk usage
alias df='df -h'
alias du='du -h'

# Grep with color
alias grep='grep --color=auto'

# Git shortcuts (if you use git)
alias gs='git status'
alias ga='git add'
alias gc='git commit -m'
alias gp='git push'
alias gl='git log --oneline --graph --decorate'
```

Notice the `cp`/`mv`/`rm` ones: you can alias a command to *itself plus a flag*. Now every `rm` asks for confirmation, which has saved many people from deleting the wrong thing.

### Aliases that take arguments — use a function instead

An alias just pastes your text in front of whatever you type, so arguments always land at the *end*. That's fine for `ll somedir`, but it breaks the moment you need the argument in the *middle*. For that, use a shell function — it lives in `.bashrc` right alongside your aliases:

```bash
# make a directory and cd into it in one step
mkcd() {
    mkdir -p "$1" && cd "$1"
}

# find a file by name under the current directory
ff() {
    find . -iname "*$1*"
}

# extract almost any archive with one command
extract() {
    case "$1" in
        *.tar.gz|*.tgz) tar xzf "$1" ;;
        *.tar.bz2)      tar xjf "$1" ;;
        *.zip)          unzip "$1" ;;
        *.gz)           gunzip "$1" ;;
        *)              echo "Don't know how to extract '$1'" ;;
    esac
}
```

`"$1"` is the first argument you pass, `"$2"` the second, and so on. The quotes matter — they keep filenames with spaces from breaking.

### Managing and inspecting aliases

- **List every alias currently defined:**

  ```bash
  alias
  ```

- **See what one specific alias expands to:**

  ```bash
  alias ll
  # -> alias ll='ls -la'
  ```

- **Remove an alias for the current session:**

  ```bash
  unalias ll
  ```

  (To remove it permanently, delete its line from `.bashrc`.)

- **Run the *real* command, bypassing an alias temporarily** — put a backslash in front:

  ```bash
  \rm file.txt   # runs rm without the -i confirmation, ignoring your alias
  ```

## Common pitfalls

- **"My alias isn't working."** You almost certainly edited `.bashrc` but didn't reload it. Run `source ~/.bashrc`, or open a fresh terminal.
- **Spaces around `=`.** Bash reads `alias ll = 'ls -la'` as a totally different (broken) command. No spaces.
- **It works locally but not over SSH.** That's the login-shell issue — make sure `.bash_profile` sources `.bashrc` (see the snippet near the top).
- **You're actually using Zsh, not Bash.** On recent macOS and some Linux setups the default shell is Zsh, whose config file is `~/.zshrc`. Alias syntax is identical, but you edit a different file. Check with `echo $SHELL`.
- **Naming an alias after a real command.** Usually fine and often intentional (like the `rm -i` trick), but be aware you're shadowing the original. Use `\command` or `command name` to reach past the alias when you need to.

## A minimal starter `.bashrc` section

If you just want something to paste in and tweak, add this block to the bottom of your `~/.bashrc`:

```bash
# ---- my custom aliases ----
alias ll='ls -la'
alias ..='cd ..'
alias ...='cd ../..'
alias grep='grep --color=auto'
alias df='df -h'
alias rm='rm -i'
alias reload='source ~/.bashrc'
alias bashrc='nano ~/.bashrc'

# ---- my custom functions ----
mkcd() { mkdir -p "$1" && cd "$1"; }
# ---------------------------
```

Then run `source ~/.bashrc`, type `ll`, and you're off.
```
```

## Quick reference

| Task | Command |
|---|---|
| Edit your config | `nano ~/.bashrc` |
| Apply changes now | `source ~/.bashrc` |
| Make a temporary alias | `alias name='command'` |
| List all aliases | `alias` |
| Inspect one alias | `alias name` |
| Remove an alias (session) | `unalias name` |
| Bypass an alias once | `\command` |
| Which shell am I in? | `echo $SHELL` |

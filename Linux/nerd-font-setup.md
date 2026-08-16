# Nerd Font Setup Guide

Nerd Fonts add extra glyphs (icons) used by terminal tools, LazyVim, and many
developer-focused websites. If you're seeing empty boxes instead of icons,
your terminal/app isn't using a Nerd Font yet.

Download fonts here: https://www.nerdfonts.com/font-downloads
Popular picks: JetBrainsMono Nerd Font, FiraCode Nerd Font, Hack Nerd Font

---

## 1. Terminal

Pick your platform/app and set the font:

- **Windows Terminal:**
  Settings → Profiles → (your shell) → Appearance → Font face →
  select your Nerd Font (e.g. "JetBrainsMono Nerd Font") → Save.

- **macOS Terminal.app:**
  Terminal → Settings → Profiles → Text → Change Font → pick the Nerd Font.

- **iTerm2:**
  iTerm2 → Settings → Profiles → Text → Font → select the Nerd Font.

- **Linux (GNOME Terminal):**
  Terminal → Preferences → Profile → Text → uncheck "Use system font" →
  choose the Nerd Font.

- **Alacritty** (`alacritty.toml` or `.yml`):
  ```toml
  [font.normal]
  family = "JetBrainsMono Nerd Font"
  ```

- **WezTerm** (`.wezterm.lua`):
  ```lua
  config.font = wezterm.font("JetBrainsMono Nerd Font")
  ```

**Important:** Fully quit and reopen the terminal app after changing the font —
a new tab/window isn't always enough.

---

## 2. LazyVim

LazyVim runs inside your terminal, so it automatically inherits whatever font
your terminal emulator uses. There's no separate font setting inside
Neovim/LazyVim for a terminal-based setup — once your terminal is using the
Nerd Font, LazyVim's icons should render correctly.

**Exception:** If you use a GUI Neovim frontend (Neovide, VimR, Goneovim,
etc.) instead of plain terminal `nvim`, that app has its own font setting.
Look for a `guifont` option in its config and set it there too, e.g.:

```vim
set guifont=JetBrainsMono\ Nerd\ Font:h12
```

---

## 3. Computer / System-wide (so browsers and other apps can use it too)

- **Windows:**
  Settings → Personalization → Fonts → confirm the font is listed.
  (If you double-clicked the `.ttf`/`.otf` file and clicked "Install," you're done.)

- **macOS:**
  Open the font file in Font Book and click "Install Font" if not already installed.

- **Linux:**
  Place the font file(s) in `~/.local/share/fonts/` or `/usr/share/fonts/`,
  then rebuild the font cache:
  ```bash
  fc-cache -fv
  ```

---

## 4. Verifying installation

- **Windows:** Settings → Personalization → Fonts → search "Nerd"
- **macOS:** Open Font Book → search "Nerd"
- **Linux:**
  ```bash
  fc-list | grep -i nerd
  ```

If the font doesn't show up, reinstall it and make sure you selected
"Install for all users" (Windows) or dragged it into Font Book (macOS).

---

## 5. Quick troubleshooting checklist

- [ ] Font installed and confirmed via Font Book / Settings / `fc-list`
- [ ] Terminal emulator's font explicitly set to the Nerd Font (not "system default")
- [ ] Terminal fully restarted (quit and reopened, not just new tab)
- [ ] If using a Neovim GUI frontend, `guifont` set there too
- [ ] Linux only: ran `fc-cache -fv` after adding font files

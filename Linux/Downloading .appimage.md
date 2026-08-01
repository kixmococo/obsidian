# AppImage → Desktop Icon Cheatsheet (Linux)

Generic steps to take any `.AppImage` file and turn it into a proper, clickable desktop application with an icon.

## 1. Unzip (if needed)

Some AppImages ship inside a `.zip`. If so:

```bash
cd ~/Downloads
unzip SomeApp.zip
```

Skip if it downloaded as a bare `.AppImage`.

## 2. Move it somewhere permanent

AppImages run from wherever they sit — they don't install. Pick a permanent home:

```bash
mkdir -p ~/Applications
mv SomeApp.AppImage ~/Applications/
```

## 3. Make it executable

```bash
chmod +x ~/Applications/SomeApp.AppImage
```

Test it:

```bash
~/Applications/SomeApp.AppImage
```

- **FUSE error?** → `sudo apt install libfuse2` (Debian/Ubuntu) or `sudo dnf install fuse fuse-libs` (Fedora), then retry.
- **SUID sandbox error?** (Electron apps) → append `--no-sandbox` to the run command:
  ```bash
  ~/Applications/SomeApp.AppImage --no-sandbox
  ```

## 4. Create a desktop entry

```bash
nano ~/.local/share/applications/someapp.desktop
```

Paste and edit:

```ini
[Desktop Entry]
Name=SomeApp
Exec=/home/YOURUSERNAME/Applications/SomeApp.AppImage
Icon=/home/YOURUSERNAME/Applications/someapp.png
Type=Application
Categories=Utility;
Terminal=false
```

**Field notes:**
- `Exec` — full path to the AppImage, no `~`. Append `--no-sandbox` here if step 3 needed it.
- `Icon` — path to a `.png`/`.svg`. Extract one from the AppImage itself with:
  ```bash
  ./SomeApp.AppImage --appimage-extract
  ```
  This dumps contents into `squashfs-root/` — look for an icon file inside, then move it wherever you want (e.g. `~/.local/share/icons/`).
- `Categories` — optional, controls which menu section it appears under.

Save: `Ctrl+O`, `Enter`, `Ctrl+X`.

## 5. Make the desktop file executable

```bash
chmod +x ~/.local/share/applications/someapp.desktop
```

## 6. Refresh the app menu

```bash
update-desktop-database ~/.local/share/applications/
```

The app should now appear in your launcher/menu with a real icon, same as any natively installed app.

---

## Worked Example: Obsidian

```bash
# Downloaded to ~/Apps/Obsidian-1.12.7.AppImage
chmod +x ~/Apps/Obsidian-1.12.7.AppImage

# Hit SUID sandbox error on launch -> fixed with --no-sandbox
```

`~/.local/share/applications/obsidian.desktop`:

```ini
[Desktop Entry]
Name=Obsidian
Exec=/home/YOURUSERNAME/Apps/Obsidian-1.12.7.AppImage --no-sandbox
Icon=/home/YOURUSERNAME/Apps/obsidian.png
Type=Application
Categories=Utility;Office;
Terminal=false
```

```bash
chmod +x ~/.local/share/applications/obsidian.desktop
update-desktop-database ~/.local/share/applications/
```

## Common Errors Quick Reference

| Error | Fix |
|---|---|
| `dlopen(): error loading libfuse.so.2` | `sudo apt install libfuse2` |
| `AppImages require FUSE to run` | Same as above |
| `SUID sandbox helper binary...not configured correctly` | Add `--no-sandbox` to Exec line |
| Icon not showing | Double check `Icon=` path is correct and file exists |
| Not appearing in menu | Run `update-desktop-database ~/.local/share/applications/` |

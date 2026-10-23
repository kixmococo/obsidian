# Running Battle.net on Linux with Lutris

A step-by-step guide for installing and launching Battle.net on Linux using
Lutris (with its bundled GE-Proton runner). Written for Ubuntu `resolute` on a
hybrid Intel + NVIDIA GPU laptop, but the steps apply broadly.

> **Why Lutris and not plain Wine?**
> The distro's Wine (`wine` 10.0) runs in *experimental WoW64 mode*, which has
> OpenGL bugs (`wglSetPixelFormatWINE failed`) that make Battle.net's embedded
> browser crash-loop. The WineHQ build that would fix it can't install on this
> release (it needs `libavcodec60:i386`, which `resolute` no longer ships).
> Lutris sidesteps both problems by downloading its own self-contained
> GE-Proton build that renders over Vulkan/DXVK instead of the broken GL path.

---

## Prerequisites

Install Lutris and the 32-bit graphics/Vulkan libraries it needs:

```bash
sudo apt install lutris
sudo apt install libgl1-mesa-dri:i386 libglx-mesa0:i386 libgl1:i386
sudo apt install mesa-vulkan-drivers:i386
sudo apt install winbind      # provides ntlm_auth, needed for Battle.net login
```

> If `apt` can't find `lutris` on a very new release, use Flatpak instead:
> ```bash
> sudo apt install flatpak
> flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
> flatpak install flathub net.lutris.Lutris
> ```

---

## Part 1 — Clean slate (only if a previous attempt failed)

Skip this if you're installing fresh.

1. Kill any leftover Wine/Battle.net processes:

   ```bash
   wineserver -k 2>/dev/null
   pkill -9 -f -i battle 2>/dev/null
   pkill -9 umu 2>/dev/null
   ```

2. In the Lutris window: right-click the **Battle.net** entry → **Remove** →
   tick **"Remove game files from disk"** → confirm.

3. Wipe leftover folders and caches:

   ```bash
   rm -rf ~/Games/battlenet
   rm -rf ~/.cache/lutris/installer/battlenet
   rm -rf ~/.cache/lutris/battlenet*
   ```

4. *(Optional)* Remove any old bare-Wine attempt. This deletes Wine *data*
   only, not Wine itself:

   ```bash
   rm -rf ~/.wine
   ```

---

## Part 2 — Install Battle.net through Lutris

1. Open Lutris. A clean launch log will show your GPUs detected and the
   GE-Proton runner in use. These are **harmless** and expected:
   - `WARNING: [Loader Message] ... does not export vkGetPhysicalDevice...KHR`
     — Vulkan loader noise, ignore it.
   - `Failed to parse JSON from ... Battle.net.config` and
     `ShellExecuteEx failed: File not found` during prefix creation — a
     cosmetic ordering hiccup; fine as long as the setup step that follows
     exits with return code 0.

2. Click the **`+`** (top-left) → **"Search the Lutris website for
   installers"** → search **Battle.net** → pick the official result →
   **Install** → accept the defaults.

3. Lutris builds a fresh prefix at `~/Games/battlenet` and downloads GE-Proton.
   Let it finish.

4. **If the Blizzard "Battle.net Setup" window does not appear on its own**
   (the scripted step sometimes silently skips it), drive it manually:
   right-click **Battle.net** → **Wine** section → **"Run EXE inside Wine
   prefix"** → select your installer:

   ```
   ~/Downloads/Battle.net-Setup.exe
   ```

   Let the Blizzard installer download and install fully. Don't close it.

5. Verify Battle.net actually installed (this folder must exist and contain
   `.exe` files):

   ```bash
   ls -la ~/Games/battlenet/drive_c/"Program Files (x86)"/Battle.net/
   ```

   From now on, launch with the Lutris **Play** button — never by hand — so it
   always starts in the correct prefix.

---

## Part 3 — Fix the blank / white login screen

Battle.net's login is an embedded Chromium browser. On this GPU setup it often
renders white, or flashes the real login for a split second then blanks. That's
a hardware-acceleration repaint bug. Fix it in order:

### Fix A — Disable hardware acceleration (try first)

1. Fully close Battle.net:

   ```bash
   wineserver -k
   ```

2. Edit Battle.net's config to render the login on the CPU:

   ```bash
   python3 - <<'EOF'
   import json, os
   p = os.path.expanduser("~/Games/battlenet/drive_c/users/u1/AppData/Roaming/Battle.net/Battle.net.config")
   with open(p) as f: cfg = json.load(f)
   cfg.setdefault("Client", {})["HardwareAcceleration"] = "false"
   with open(p, "w") as f: json.dump(cfg, f, indent=4)
   print("updated:", p)
   EOF
   ```

3. Confirm the setting landed:

   ```bash
   grep -i hardware ~/Games/battlenet/drive_c/users/u1/AppData/Roaming/Battle.net/Battle.net.config
   ```

   You should see `"HardwareAcceleration": "false"`.

4. Relaunch with the Lutris **Play** button.

### Fix B — Force a virtual desktop (if A isn't enough)

Gives the browser a stable framebuffer to paint into.

1. In Lutris: right-click **Battle.net** → **Configure** → **Runner options**
   tab.
2. Turn **on** "Emulate a virtual desktop" and set a resolution, e.g.
   `1280x720`.
3. **Save** → **Play**. Battle.net opens in its own bordered window and the
   login should render solid.

---

## Part 4 — Hybrid GPU (use the NVIDIA card)

On an Intel + NVIDIA laptop, make sure Battle.net and its games run on the
RTX 5070 Ti, not the weak Intel iGPU:

- In Lutris: right-click **Battle.net** → **Configure** → **System options** →
  enable **"Enable NVIDIA Prime render offload"** (and/or set the discrete GPU
  as preferred). Save.

---

## Handy commands

```bash
wineserver -k                 # kill everything Wine/Battle.net in this prefix
jobs                          # list background terminal jobs
kill %1                       # kill terminal job [1]
pkill -9 -f Battle.net        # force-kill anything named Battle.net
```

**Key paths**

| What | Where |
|------|-------|
| Lutris prefix (fake C: drive) | `~/Games/battlenet/drive_c/` |
| Battle.net install | `~/Games/battlenet/drive_c/Program Files (x86)/Battle.net/` |
| Battle.net config | `~/Games/battlenet/drive_c/users/u1/AppData/Roaming/Battle.net/Battle.net.config` |
| Installer stub | `~/Downloads/Battle.net-Setup.exe` |

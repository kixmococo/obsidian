# Remote Desktop Access to Ubuntu from Your Phone

Full graphical remote control of your Ubuntu machine from your phone, using **xrdp** (remote desktop protocol) + **Tailscale** (secure private network, no port forwarding needed).

---

## Part 1: Set Up Your Ubuntu Machine

### 1. Update your system
```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Install a desktop environment (skip if you already have Ubuntu Desktop)
Only needed if this is a headless server with no GUI installed:
```bash
sudo apt install xfce4 xfce4-goodies -y
```

### 3. Install xrdp
```bash
sudo apt install xrdp -y
sudo systemctl enable xrdp
sudo systemctl start xrdp
```

### 4. If your machine already has a desktop environment (GNOME, etc.), read this first
**Do not just point `.xsession` at your existing desktop environment.** If the Ubuntu machine is one you also use physically (not headless), GNOME will refuse to run two sessions for the same user — it errors with `A graphical session is already running!` and the remote session crashes ~2-3 seconds after login every time, regardless of network, client app, or config. See the **Troubleshooting** section at the bottom of this doc for the full story — the reliable fix is a **dedicated remote-login user running XFCE**, not trying to share your GNOME session:

```bash
# Install XFCE if it isn't already present
sudo apt install xfce4 xfce4-goodies -y

# Create a separate user just for remote logins
sudo adduser remoteuser

# Give that user XFCE as their session (not GNOME)
sudo -u remoteuser bash -c 'echo "xfce4-session" > ~/.xsession'

# Verify it actually saved correctly (watch for typos - it must read exactly "xfce4-session")
sudo cat /home/remoteuser/.xsession

# Confirm the binary actually exists before relying on it
which xfce4-session
```
Use `remoteuser` (not your everyday login) as the username when connecting via RDP going forward.

### 5. Open the RDP port in the firewall (if ufw is active)
```bash
sudo ufw allow 3389/tcp
```

### 6. Install Tailscale
```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```
This prints a login URL — open it in any browser and sign in (Google/Microsoft/GitHub) to authenticate the machine.

### 7. Get your Tailscale IP
```bash
tailscale ip -4
```
**Save this IP** — you'll need it to connect from your phone.

### 8. Reboot
```bash
sudo reboot
```

---

## Part 2: Set Up Your Phone

1. Install **Tailscale** from the App Store / Play Store and log into the **same account** you used on the Ubuntu machine.
2. Install an RDP client app — **Windows App** (formerly "Microsoft Remote Desktop" — same app, rebranded) works well and is available on both iOS and Android.
3. In the RDP app, add a new connection:
   - **PC name / Host**: the Tailscale IP from Part 1, step 7
   - **Port**: 3389
   - **Username / Password**: your Ubuntu login credentials
4. Connect — you should see your full Ubuntu desktop and have full mouse/keyboard control, as if sitting in front of it.

---

## Notes
- Tailscale must be running and connected on both devices for this to work.
- xrdp uses port `3389` by default.
- If the screen looks off after connecting, log out of any active local desktop session first — xrdp sometimes conflicts with an already-active graphical session.

---

## Tips & Tricks

### Performance
- **Lower the resolution/color depth** in the RDP app's connection settings if things feel laggy over mobile data — you don't need 4K color to edit a config file.
- **Use Wi-Fi when possible.** RDP over cellular works but is noticeably snappier on Wi-Fi, especially for anything video/animation heavy.
- **Close unused apps on the Ubuntu machine** before connecting — a lighter desktop session responds faster over RDP.

### The "black screen" or "session already active" problem
- xrdp creates a *new* session by default, separate from your physical desktop session, which can cause conflicts or a black screen. To fix this properly, install `xrdp` with **Xorg backend** support so it shares your actual desktop session instead of spawning a new one:
  ```bash
  sudo apt install xorgxrdp -y
  ```
- If you still hit a black screen, try logging out of the physical/local session entirely before connecting remotely.

### Reliability
- **Set Tailscale to launch on boot** on the Ubuntu machine so it's always reachable, even after a power outage or reboot:
  ```bash
  sudo systemctl enable tailscaled
  ```
- **Give the machine a static Tailscale hostname** (instead of memorizing the IP) via the Tailscale admin console — then just connect to `machine-name` instead of an IP.
- **Enable auto-login is NOT recommended** for security, but if xrdp keeps failing to connect to a locked screen, know that a locked/suspended Ubuntu machine can block incoming RDP sessions. Adjust power/sleep settings so the machine never suspends.

### Security
- Since Tailscale already encrypts and privately tunnels the connection, you generally do **not** need to expose port 3389 to the public internet — keep it firewalled off from anything except your Tailscale network (`ufw allow in on tailscale0 to any port 3389` is safer than a blanket allow).
- Use a strong Ubuntu account password since that's what authenticates the RDP session.
- Consider enabling **Tailscale SSH** too (`tailscale up --ssh`) as a lightweight terminal fallback if RDP ever misbehaves.

### Quality of life
- **Save the connection** in your RDP app so reconnecting later is a single tap.
- **Clipboard sharing** usually works out of the box with xrdp — copy on your phone, paste on the remote desktop, and vice versa.
- If you multitask between phone and other devices, Tailscale's device list (in the app or admin console) is a quick way to check the Ubuntu machine's current status/IP without SSHing in first.

---

## Using Windows App (Client Walkthrough)

> **Note:** Microsoft retired the old "Microsoft Remote Desktop" app and renamed/replaced it with **Windows App** (on iOS it's listed as "Windows App Mobile"). Same underlying functionality, but the interface is now organized into tabs, and it's built to manage multiple types of remote connections, not just a single PC.
>
> **Alternative: aRDP** (by developer iiordanov, open source, on Google Play) is a solid alternative built specifically for Android-to-Linux/xrdp connections. Same connection details apply (host, port 3389, username, password). Worth having installed as a second option if one client ever misbehaves — see the Troubleshooting section for why having two different clients was key to diagnosing a real issue.

### 1. First launch
On first open you'll get a short onboarding tour — tap **Skip** or **Got it** to reach the main screen. You'll land on a tabbed interface: typically **Devices**, **Apps**, and **Favorites**.

### 2. Add a connection
Tap the **"+"** icon (usually top-right corner), then choose **Add PC** (not "Add Workspace" — that's for enterprise/work accounts).

### 3. Fill in the connection details
- **PC name**: your Tailscale IP (from `tailscale ip -4`) — e.g. `100.x.x.x`
- **Port**: use `3389` in the port field, or append it directly: `100.x.x.x:3389`
- **User account**: your Ubuntu username and password — can be saved so you don't retype every time
- **Friendly name** (optional): e.g. "Home Ubuntu" for easy identification later

Leave gateway/display settings as default and save. The connection now appears under the **Devices** tab.

### 4. Connect
Tap the saved connection. On first connect, you'll likely see a **certificate warning** (normal — xrdp uses a self-signed certificate). Tap **Accept/Continue**.

### 5. What to expect
Briefly a black or xrdp login screen, then your full XFCE (or other) desktop loads with full mouse/keyboard control from your phone.

### 6. Day-to-day controls
- **Touch = mouse**: tap to click, tap-and-hold to right-click, drag to move windows
- **On-screen keyboard**: toggle via the keyboard icon in the app's toolbar
- **Trackpad mode**: some versions offer a virtual trackpad toggle for more precise clicking than direct touch
- **External keyboard/mouse**: pair Bluetooth peripherals to your phone — they work seamlessly through the session, great for real work
- **Favorites**: pin your Ubuntu connection to the **Favorites** tab for one-tap access if you end up adding more remote machines later
- **Disconnect**: swipe/tap to bring up the toolbar, then disconnect — your Ubuntu desktop and open programs keep running in the background

### Client Tips & Tricks
- **Certificate warning every time?** You can tell the app to trust the certificate permanently on first connect instead of confirming every session — look for a "Don't ask again" checkbox on the warning screen.
- **Screen too small/cramped?** Adjust the display resolution setting in the connection's properties — matching it closer to your phone's native resolution (or slightly lower) often looks and performs better than "auto."
- **Multiple machines?** Save each with a distinct friendly name — handy once you're managing more than one remote box.
- **Slow typing/input lag?** Switch off any "smooth scrolling" or high-quality graphics options in the app's per-connection performance settings — trading visual smoothness for responsiveness is usually worth it on mobile data.
- **App backgrounded mid-session?** Sessions typically survive a brief app switch, but a long backgrounding may drop the connection — just reconnect via the saved tile, and your remote desktop state (open apps, windows) will still be there since xrdp keeps the session alive server-side.
- **Landscape mode** is generally easier for real work — rotate your phone before connecting for a wider working view.

---

## Troubleshooting: "Connection lost" / session dies ~2-3 seconds after login

If you connect, log in successfully, and then the session immediately dies with an error like *"RDP Connection failed... network connectivity was interrupted... or your session was taken over"* (error code `0x904` on Windows App, or similar on other clients), work through these in order. This list reflects an actual multi-hour debugging session — most of these turned out to be **not** the cause, but are worth ruling out quickly before landing on the real fix at the bottom.

### 1. Wrong username
Check `whoami` on the Ubuntu machine and make sure it matches exactly what you're entering in the RDP client. A mismatched username causes an immediate auth failure that can *look* like a network error on the client side. Server-side, this shows up in `sudo journalctl -u xrdp -u xrdp-sesman` as `User does not exist, or could not be authenticated`.

### 2. Not actually a network issue (rule this out fast)
The generic "network connectivity was interrupted" error is misleading — it fires for almost any session-teardown reason, not just real network problems. Don't assume it's your Wi-Fi/cellular connection. Watch the live server logs while reconnecting to see the *real* reason:
```bash
sudo journalctl -u xrdp -u xrdp-sesman -f
```

### 3. Test with a second, different RDP client app
If the session dies consistently at the same point in time across **two structurally different clients** (e.g. Windows App and aRDP, which use different underlying engines), that rules out a client-app bug and confirms the problem is server-side.

### 4. Check for genuine network problems (cellular/CGNAT especially)
If both devices are on cellular data, run:
```bash
tailscale netcheck
```
Look for `MappingVariesByDestIP: true` (indicates CGNAT, which forces Tailscale to relay instead of connecting directly) and check DERP latency — uniformly high latency to every region (not just distant ones) can indicate cellular/satellite-specific routing issues. This is worth fixing for overall performance, but **in our case turned out not to be the actual cause of the disconnects** — confirmed by testing over a local hotspot connection (bypassing Tailscale and cellular entirely) and the exact same failure still happened.

### 5. Check for the specific channel error
```bash
sudo tail -100 /var/log/xrdp.log
```
Look for `xrdp_rdp_recv: xrdp_channel_process failed` right after login. This can be a red herring too (it appeared in our case but wasn't the root cause) — don't stop here, keep going to step 6.

### 6. THE ACTUAL FIX: Check if your window manager is crashing
This was the real root cause in our case, and is the single most likely explanation if:
- Your Ubuntu machine already has a desktop environment installed and in use (not headless)
- The RDP session consistently dies 2-3 seconds after successful login, no matter what client, network, or config you try

Check:
```bash
sudo tail -80 /var/log/xrdp-sesman.log
```
Look for:
```
Window manager (pid XXXXX, display 10) exited with signal SIGABRT
Window manager (pid XXXXX, display 10) exited quickly (2-3 secs)
```
Then confirm with:
```bash
cat ~/.xsession-errors
```
Look for: `ERROR: A graphical session is already running!`

**This means GNOME (or another DE with a single-session lock) is refusing to start a second session for the same user, because your physical/local desktop is already logged in as that user.** Locking the screen does NOT fix this — GNOME still considers the session active even when locked or the monitor is off.

**The fix:** create a dedicated user account just for remote logins, running a different, lighter desktop environment (XFCE) that doesn't have this single-session restriction:
```bash
# Install XFCE if not already present
sudo apt install xfce4 xfce4-goodies -y

# Create the dedicated remote user
sudo adduser remoteuser

# Set XFCE as that user's session (double-check spelling - "xfce4-session", not "xfce-session")
sudo -u remoteuser bash -c 'echo "xfce4-session" > ~/.xsession'

# Verify it saved correctly
sudo cat /home/remoteuser/.xsession

# Confirm the xfce4-session binary actually exists (if this returns nothing, the apt install didn't complete - rerun it)
which xfce4-session

# Restart xrdp
sudo systemctl restart xrdp
```
Connect using `remoteuser` (not your everyday account) going forward. This gives your remote sessions full isolation from your physical desktop — no conflicts, no shared state, and no risk of GNOME silently killing the connection again.

**Note:** this new account has its own separate home directory — your usual files, browser, and settings won't be there. If you want access to your regular files from the remote session, that's a separate step (shared folder or group permissions) — ask if you want help setting that up.

### Diagnostic dead ends we ruled out (for reference)
In case you hit similar red herrings troubleshooting this in the future, these were all tested and confirmed **not** to be the cause in this case:
- GPU/DRI permission warnings (`systemd-logind: failed to take device /dev/dri/card1`) — present in the logs throughout, but unrelated to the actual crash
- Dynamic virtual channels (`drdynvc`) / GFX negotiation — disabling these didn't fix it, though it's a reasonable thing to try for other xrdp issues
- TLS/security layer overhead — switching to plain `security_layer=rdp` didn't fix it
- Tailscale MTU — adjusting this didn't fix it
- Cellular network jitter/bufferbloat — real and measurable, but not the actual cause here (confirmed via local hotspot test)

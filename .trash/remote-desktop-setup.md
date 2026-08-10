 Remote Desktop Access to Ubuntu from Your Phone

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

### 4. If you installed XFCE in step 2, point xrdp to it
```bash
echo xfce4-session > ~/.xsession
sudo sed -i.bak '/fi/a xfce4-session' /etc/xrdp/startwm.sh
```

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
2. Install an RDP client app — **Microsoft Remote Desktop** works well (available on both iOS and Android).
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

# Creating SSH Keys

SSH keys are a pair of cryptographic keys (a private key and a public key) used to authenticate securely with remote servers and services like GitHub, GitLab, and cloud VMs — without typing a password every time.

## 1. Check for Existing Keys

Before generating a new key, check if you already have one:

```bash
ls -al ~/.ssh
```

Look for files like `id_ed25519.pub` or `id_rsa.pub`. If they exist, you may already have a usable key pair.

## 2. Generate a New Key Pair

The recommended algorithm today is **Ed25519** (faster and more secure than older RSA keys):

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

If you're on a system that doesn't support Ed25519 (rare, usually very old systems), use RSA with at least 4096 bits:

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

You'll be prompted for:
- **File location** — press Enter to accept the default (`~/.ssh/id_ed25519`)
- **Passphrase** — optional but strongly recommended; adds a layer of protection if your private key is ever stolen

## 3. Add Your Key to the SSH Agent

Start the agent and add your private key so you don't have to re-enter your passphrase every session:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

On macOS, you can persist this across reboots using the keychain:

```bash
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

## 4. Copy Your Public Key

You'll need the **public** key (never share the private key) to add to services like GitHub or a remote server.

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the output, or use a clipboard shortcut:

```bash
# macOS
pbcopy < ~/.ssh/id_ed25519.pub

# Linux (requires xclip)
xclip -sel clip < ~/.ssh/id_ed25519.pub

# Windows (Git Bash)
cat ~/.ssh/id_ed25519.pub | clip
```

## 5. Add the Public Key to a Service

**GitHub example:**
1. Go to Settings → SSH and GPG keys → New SSH key
2. Paste your public key
3. Give it a descriptive title (e.g., "Work Laptop")

**Remote server example:**

```bash
ssh-copy-id user@remote-host
```

Or manually append it to `~/.ssh/authorized_keys` on the server.

## 6. Test the Connection

```bash
ssh -T git@github.com
```

Or for a server:

```bash
ssh user@remote-host
```

## Tips

- **Never share your private key** (the file without `.pub`).
- Use a unique key per device/purpose if you want to be able to revoke access individually.
- Store a backup of your private key somewhere secure (e.g., a password manager) in case of device loss.
- If you use multiple keys, configure `~/.ssh/config` to map keys to specific hosts:

```
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_github

Host myserver
    HostName 192.0.2.10
    User u1
    IdentityFile ~/.ssh/id_ed25519_myserver
```

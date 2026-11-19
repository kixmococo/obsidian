# DSAC Annex B | Lab 1 — CLI Solution

**Installing and Configuring Windows Server 2022: the fast way.**

This is the CLI companion to `1-WindowsServer-Final.pdf`. It reaches **the same end state, checked the same way**, using one elevated PowerShell session per server and **one restart per server**. Expect about 2 minutes per server, compared with 15–20 minutes through the GUI.

> [!info] Who this is for
> Students who have finished the GUI lab at least once and understand *what* each setting does. The CLI replaces the clicking, not the understanding. Every command below is mapped back to its GUI step.

---

## End State (both servers)

| Setting | NDSACDC01 | NDSACDC02 |
|---|---|---|
| Hostname | `NDSACDC01` | `NDSACDC02` |
| Date/time | Matches laptop | Matches laptop |
| Firewall (Local GPO) | Domain/Private/Public **Off** | Domain/Private/Public **Off** |
| IPv4 | `192.168.40.2/24` | `192.168.40.3/24` |
| Gateway | `192.168.40.1` | `192.168.40.1` |
| DNS | `192.168.40.2`, `192.168.40.3` | `192.168.40.2`, `192.168.40.3` |

**Credentials:** `xadministrator` / `M@rines1M@rines1`

---

## 0. Open an Elevated PowerShell

1. Sign in to the VM.
2. Press **Win + X**, then **A**. This opens *Windows PowerShell (Admin)*. Accept UAC.
3. Confirm you're elevated. This must return `True`:

```powershell
([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole('Administrators')
```

> [!note] Section 1 (OS installation)
> The OS install is a GUI/boot-media task with no in-OS CLI equivalent. If your VM boots to a sign-in screen, it's already installed, so start here. (Production-scale builds automate installation with an `autounattend.xml` answer file, which is beyond this lab.)

---

## 1. The Whole Lab: One Block per Server

Edit the **three variables at the top**, paste or type the block, and run it. The server restarts once at the end.

```powershell
# ===== EDIT THESE PER SERVER =====
$Name = 'NDSACDC01'            # NDSACDC02 on the secondary
$IP   = '192.168.40.2'         # 192.168.40.3 on the secondary
$Now  = '11/17/2026 9:45 AM'   # read off your laptop immediately before running
# =================================

# --- Section 2: Date and time ---
Set-Date -Date $Now | Out-Null

# --- Section 3: Firewall OFF in the Local Group Policy Object (the same setting gpedit.msc writes) ---
Set-NetFirewallProfile -PolicyStore localhost -Profile Domain,Private,Public -Enabled False

# --- Section 4: Static IPv4 + DNS ---
$nic = Get-NetAdapter -Physical | Where-Object Status -eq 'Up' | Select-Object -First 1
Set-NetIPInterface -InterfaceIndex $nic.ifIndex -AddressFamily IPv4 -Dhcp Disabled
Get-NetIPAddress -InterfaceIndex $nic.ifIndex -AddressFamily IPv4 -ErrorAction SilentlyContinue |
    Remove-NetIPAddress -Confirm:$false
Get-NetRoute -InterfaceIndex $nic.ifIndex -DestinationPrefix '0.0.0.0/0' -ErrorAction SilentlyContinue |
    Remove-NetRoute -Confirm:$false
New-NetIPAddress -InterfaceIndex $nic.ifIndex -IPAddress $IP -PrefixLength 24 -DefaultGateway 192.168.40.1 | Out-Null
Set-DnsClientServerAddress -InterfaceIndex $nic.ifIndex -ServerAddresses 192.168.40.2,192.168.40.3

# --- Pre-restart check: review this output BEFORE the reboot ---
Get-Date
Get-NetFirewallProfile -PolicyStore localhost | Format-Table Name, Enabled
Get-NetIPConfiguration -InterfaceIndex $nic.ifIndex |
    Format-List InterfaceAlias, IPv4Address, IPv4DefaultGateway, DNSServer

# --- Section 5: Rename + single restart ---
Read-Host "Settings above correct? Press Enter to rename to $Name and restart (Ctrl+C to abort)"
Rename-Computer -NewName $Name -Force -Restart
```

### Why it's built this way

| Choice | Reason |
|---|---|
| `-PolicyStore localhost` | Writes to the **Local GPO**, exactly what the lab does in `gpedit.msc`. A plain `Set-NetFirewallProfile` (or `netsh advfirewall`) changes the *local firewall store* instead. That turns the firewall off, but it's a different setting from the one the lab asks for. |
| Remove old IP and default route first | `New-NetIPAddress` fails with *"Instance DefaultGateway already exists"* if a DHCP-learned route is still present. Clearing first makes the block safe to re-run. |
| `Get-NetAdapter -Physical … Up` | The adapter is named `Ethernet` on Hyper-V but often `Ethernet0` on VMware. Selecting by state avoids hard-coding the name. |
| Rename **last**, with `-Restart` | The name change is the only step that needs a reboot. Doing it last means **one** restart covers everything, instead of rebooting partway through. |
| `Read-Host` gate | Gives you one last look at the output before the VM goes down. |

---

## 2. Step-by-Step Mapping (GUI → CLI)

Every step in the Final PDF, and the command that replaces it.

### Section 2 — Date and Time

| GUI step | CLI |
|---|---|
| 1–3. Server Manager › Local Server › Time zone › Change date and time | *(not needed)* |
| 4. Set date/time/AM-PM to match laptop | `Set-Date '11/17/2026 9:45 AM'` |
| 4. (if needed) Match time zone | `Get-TimeZone` / `Set-TimeZone -Id 'Pacific Standard Time'`  (`Get-TimeZone -ListAvailable` lists IDs) |
| 5. OK | *(applies immediately)* |
| ✓ Verify | `Get-Date` |

> [!warning] Time reverts after a few seconds?
> The hypervisor's guest time-sync service is overriding you. That's an environment setting; tell your instructor. Don't fight it inside the guest.

### Section 3 — Firewall

| GUI step | CLI |
|---|---|
| 1. `gpedit.msc` | *(not needed)* |
| 2–4. Computer Config › Windows Settings › Security Settings › Windows Defender Firewall with Advanced Security › Properties | *(not needed)* |
| 5–7. Domain/Private/Public → Firewall state **Off** | `Set-NetFirewallProfile -PolicyStore localhost -Profile Domain,Private,Public -Enabled False` |
| 8. Apply / OK | *(applies immediately)* |
| ✓ Verify policy is set | `Get-NetFirewallProfile -PolicyStore localhost \| ft Name,Enabled` → all `False` |
| ✓ Verify it's in effect | `Get-NetFirewallProfile -PolicyStore ActiveStore \| ft Name,Enabled` → all `False` |
| ! Still shows on | `gpupdate /force` |

### Section 4 — Static IP

| GUI step | CLI |
|---|---|
| 1–3. Local Server › Ethernet › Properties › IPv4 › Properties | `$nic = Get-NetAdapter -Physical \| ? Status -eq Up \| select -First 1` |
| 4. IP / mask / gateway | `New-NetIPAddress -InterfaceIndex $nic.ifIndex -IPAddress 192.168.40.2 -PrefixLength 24 -DefaultGateway 192.168.40.1` |
| 5. DNS servers | `Set-DnsClientServerAddress -InterfaceIndex $nic.ifIndex -ServerAddresses 192.168.40.2,192.168.40.3` |
| 6–7. OK / Close | *(applies immediately)* |
| ✓ Verify | `ipconfig /all` → address, mask, gateway, both DNS, `DHCP Enabled . . : No` |

> `/24` **is** `255.255.255.0`: 24 one-bits of network mask. PowerShell takes prefix length, not dotted mask.

### Section 5 — Rename

| GUI step | CLI |
|---|---|
| 1–3. Local Server › Computer name › Change › `NDSACDC01` › OK | `Rename-Computer -NewName NDSACDC01 -Force -Restart` |
| 4–5. Acknowledge restart › Restart Now | *(included via `-Restart`)* |
| ✓ Verify after reboot | `hostname` |

### Section 6 — Secondary Server

On **NDSACDC02**, run the Section 1 block with:

```powershell
$Name = 'NDSACDC02'
$IP   = '192.168.40.3'
```

Everything else is the same: gateway, both DNS servers, and firewall.

---

## 3. End-State Verification (Section 7)

After each server restarts, sign back in, open an elevated PowerShell (**Win + X**, **A**), and run:

```powershell
$peer = if ((hostname) -eq 'NDSACDC01') { '192.168.40.3' } else { '192.168.40.2' }
"Hostname : $(hostname)"
"Time     : $(Get-Date)"
Get-NetFirewallProfile -PolicyStore ActiveStore | Format-Table Name, Enabled -AutoSize
ipconfig /all | Select-String 'Host Name|IPv4 Address|Subnet Mask|Default Gateway|DNS Servers|DHCP Enabled' -Context 0,1
"Ping $peer : $(Test-Connection $peer -Count 2 -Quiet)"
```

**Pass criteria:**

- [ ] Hostname is `NDSACDC01` / `NDSACDC02`
- [ ] Time matches your laptop
- [ ] Domain, Private, and Public all show `Enabled : False`
- [ ] IPv4 `192.168.40.2` / `.3`, mask `255.255.255.0`, gateway `192.168.40.1`
- [ ] DNS servers `192.168.40.2` and `192.168.40.3`, and `DHCP Enabled: No`
- [ ] Ping to the other server returns `True` (run this after **both** servers are finished)

---

## 4. Fewest-Keystrokes Variant

Use this if your console has no clipboard paste and you're typing everything by hand. It uses `netsh` and short aliases, so there's far less to type than in the main block. The end state is identical. *(Use `Get-NetAdapter` first if the adapter isn't named `Ethernet`.)*

```powershell
Set-Date '11/17/2026 9:45 AM'
Set-NetFirewallProfile -PolicyStore localhost -All -Enabled False
netsh int ipv4 set address "Ethernet" static 192.168.40.2 255.255.255.0 192.168.40.1
netsh int ipv4 set dns "Ethernet" static 192.168.40.2 primary
netsh int ipv4 add dns "Ethernet" 192.168.40.3 index=2
Rename-Computer NDSACDC01 -Force -Restart
```

For NDSACDC02, change `.40.2` to `.40.3` **in the address line only** (the DNS lines stay the same) and change the name to `NDSACDC02`.

> [!tip] Menu-driven alternative: `sconfig`
> Typing `sconfig` opens the Server Configuration tool. Option **2** sets the computer name, **8** sets network settings, and **9** sets date/time. It's faster than the GUI but slower than the block above, and it **can't** set the firewall GPO, so you'd still need the `Set-NetFirewallProfile` line.

---

## 5. Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| `Access is denied` / `PermissionDenied` | PowerShell isn't elevated | Close it and reopen with **Win + X**, **A** |
| `Instance DefaultGateway already exists` | Old DHCP route still present | Run the `Remove-NetRoute` line, then retry `New-NetIPAddress` |
| `Inconsistent parameters PolicyStore PersistentStore and Dhcp Enabled` | DHCP still enabled on the adapter | `Set-NetIPInterface -InterfaceIndex $nic.ifIndex -Dhcp Disabled`, then retry |
| `$nic` is empty | Adapter is disconnected / Status not `Up` | `Get-NetAdapter`; check the VM's network attachment |
| Address shows `169.254.x.x` | Duplicate IP on the network | Wrong `$IP` for this server; recheck the End State table |
| Firewall `ActiveStore` still `True` | Policy not refreshed yet | `gpupdate /force` |
| Ping fails, everything else correct | Other server not finished yet, or its firewall is still on | Finish/verify the peer, then ping again |
| Hostname unchanged after reboot | `Rename-Computer` errored and was missed | Rerun `Rename-Computer -NewName <name> -Force -Restart` and read the output |

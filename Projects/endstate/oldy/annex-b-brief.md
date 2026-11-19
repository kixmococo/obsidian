# Annex B Brief: DSA End State

**Sources:** `USMC/CTB/DSAC/b-annex/`, which holds the Lesson 1–3 and Messaging handouts, Facilitation Guides DS.02.01–02.08, Lab 1 (Windows Server, GUI and CLI), Lab 11 (Exchange post-deployment, GUI and CLI), and the Lesson 1 knowledge check.
**TLOs:** 0671-IMPL-1002 (*implement*) and 0671-OPER-1002 (*operate*) data systems **IAW STIGs** and local cybersecurity policy.

---

## Bottom Line

By the end of Annex B, the Marine should be able to take bare install media and **build, join, secure, mail-enable, back up, and keep running a small Windows enterprise**, then prove that it works.

## Target Build

| Host | OS | Roles / Apps | Lab example |
|---|---|---|---|
| DC01 | Server 2022 | AD DS (forest root, FSMO), DNS, GC | `NDSACDC01`, `192.168.40.2/24` |
| DC02 | Server 2022 | AD DS (additional DC), DNS, GC | `NDSACDC02`, `192.168.40.3/24` |
| EX01 / EX02 | Server 2022 | Exchange 2019 Mailbox role, DAG members | `NXXMEUEX01` / `NXXMEUEX02` |
| UTIL | Server 2022 | File Server (DAG witness). *Optional:* DHCP, WSUS, AD CS | `NXXMEUUTIL` |
| Client | Windows 10/11 | Domain-joined, RSAT, Outlook | — |

---

## End-Goal Checklist

### 1. Deploy OS (DS.02.01–02)
- [ ] Choose a server edition (Standard or Datacenter) and an install option (Desktop Experience or Core), and justify the choice
- [ ] Mount the media and install Windows Server on a VM
- [ ] Install a Windows workstation client
- [ ] Navigate the OS and the registry (`regedit`)

### 2. Base-Configure Each Server (DS.02.03, Lab 1)
- [ ] Set the hostname to the naming standard
- [ ] Set a static IPv4 address, mask, gateway, and both DNS servers
- [ ] Set the date, time, and time zone
- [ ] Set the firewall state (see the scrutiny section below)
- [ ] Verify connectivity: `ipconfig /all`, `Test-Connection`, peer ping
- [ ] Do the same build from the GUI, PowerShell, and `sconfig`

### 3. Domain Services (DS.02.04)
- [ ] Install the AD DS, DNS, and RSAT roles and features
- [ ] Create the forest and domain on DC01, then promote DC02 as an additional DC
- [ ] Create forward and reverse zones and A/PTR records
- [ ] Build an OU structure to the naming standard
- [ ] Use ADUC, AD Sites & Services, and ADAC
- [ ] View and transfer FSMO roles (`netdom query fsmo`, `Move-ADDirectoryServerOperationMasterRole`)

### 4. Configure Services and Policy (DS.02.05, Lesson 3)
- [ ] Create users and security groups, assign object permissions, and pre-stage computer objects
- [ ] Join the client and the member servers to the domain
- [ ] Add DNS conditional forwarders to neighbor sites
- [ ] Create, link, and enforce a GPO, and use Block Inheritance. Know the LSDOU processing order.
- [ ] Force a GPO onto a client and prove it applied (`gpupdate /force`, `gpresult /r`)
- [ ] *(Lesson 2 content, no lab)* Deploy DHCP: create a scope, set options 003/006/015, and authorize the server in AD

### 5. Messaging (DS.02.06, Lab 11)
- [ ] Install the prerequisites: .NET 4.8, VC++ 2012/2013, UCMA 4.0, IIS URL Rewrite, and the Windows features
- [ ] Prepare AD: `/PrepareSchema`, `/PrepareAD`, `/PrepareDomain`
- [ ] Install the Mailbox role through the GUI (EX01) and unattended (EX02)
- [ ] Rename the databases to MBDB01/02 and move the EDB and logs to `F:`
- [ ] Create the Internet Send Connector (scoped, both servers as sources) and review the receive connectors
- [ ] Add DNS records: round-robin A records (mail, owa, webmail, autodiscover), MX, and autodiscover SRV
- [ ] Set the Outlook Anywhere hostnames and the OWA/ECP internal and external URLs and authentication
- [ ] Create mailboxes, mail-enabled groups, and admin accounts in Organization Management
- [ ] Configure an Outlook profile, import a PST, and send and receive mail
- [ ] Migrate a mailbox (`New-MoveRequest`)
- [ ] Get mail flowing between sites through conditional forwarders on both sides
- [ ] Build the DAG: UTIL as witness, both members joined, passive copies Healthy, queues near 0

### 6. Maintain the OS (DS.02.07)
- [ ] Patch the servers (Windows Update, or WSUS if deployed)
- [ ] Back up and restore the server OS (Windows Server Backup, bare-metal or system state)
- [ ] Shut down Exchange gracefully (maintenance mode, then shutdown)
- [ ] Troubleshoot OS and network connectivity

### 7. Maintain Services (DS.02.08)
- [ ] Back up AD (system state) and restore it, both non-authoritative and authoritative (DSRM, `ntdsutil`)
- [ ] Monitor replication and DNS: `repadmin /replsummary`, `dcdiag`, DNS event log
- [ ] Restart and update domain services, and remove a client from the domain
- [ ] Exchange: dismount and mount a database, move the active copy (`Move-ActiveMailboxDatabase`), and back up the active database
- [ ] Exchange health checks: `Test-ServiceHealth`, `Test-ReplicationHealth`, `Get-MailboxDatabaseCopyStatus`
- [ ] Trace mail with `Get-MessageTrackingLog`
- [ ] Apply an Exchange CU and update its prerequisites
- [ ] Use Performance Monitor, Event Viewer, and Resource Monitor

---

## Final Proof of Function (Sign-off)
- [ ] `dcdiag /q` is clean and `repadmin /replsummary` shows 0 failures
- [ ] A domain user logs on at the client, and `gpresult` shows the GPO
- [ ] `nslookup` rotates mail and autodiscover records, and the MX and SRV records resolve
- [ ] A user on EX01 can mail a user on EX02 **and** a neighbor site
- [ ] Failing MBDB01 over to EX02 does not stop mail
- [ ] An AD object and a server backup have each been restored once

---

## Scrutiny: Gaps and Fixes

**R** = recommended, **O** = optional

- [ ] **R: Firewall is off on every profile (Lab 1).** This contradicts the TLO's "IAW STIGs" requirement, because the Windows Server STIG requires the firewall to be on. Keep the lab step, but add a domain-GPO step later that turns the firewall back on with the required AD, DNS, and Exchange rules. Also teach `Get-NetFirewallRule` and `Test-NetConnection -Port` for troubleshooting.
- [ ] **R: DNS order on the DCs.** Promote DC01 with DNS set to `127.0.0.1`. Once DC02 is up, follow Microsoft's guidance: **partner DC first, loopback last**. At the moment DC01 lists itself first.
- [ ] **R: Time is set by hand only.** In a domain, clients sync to the PDC emulator. Configure the PDCe with `w32tm /config /manualpeerlist:<NTP> /syncfromflags:manual /reliable:yes /update` and disable hypervisor time sync on the DCs. Lab 1's "time reverts" warning is this problem.
- [ ] **R: No certificates for HTTPS URLs.** Lab 11 sets `https://owa…` and autodiscover URLs but never issues a cert, so clients will see warnings and autodiscover prompts. Add a SAN cert (mail, owa, webmail, autodiscover) from AD CS or DoD PKI with `New-ExchangeCertificate` and `Enable-ExchangeCertificate -Services IIS,SMTP`. Lesson 2 already covers AD CS.
- [ ] **R: Support lifecycle.** Exchange 2019 and Windows 10 both reached end of support on **14 Oct 2025**. Keep them for training parity, but note that a fielded system needs **Exchange Server SE** (an in-place upgrade from 2019 CU15) and **Windows 11**. Server 2022 is now in extended support, and Server 2025 is current.
- [ ] **R: Many ELOs have no lab.** Receive connectors, mail-enabled groups, PST import, mailbox moves, message tracking, AD and OS backup/restore, FSMO transfer, and DHCP are all graded objectives with no hands-on lab. Write short mini-labs for each, starting with backup/restore.
- [ ] **R: Credential hygiene.** The lab docs embed the shared admin password. That is fine for an isolated range, but also teach named admin accounts, renaming or disabling the built-in Administrator (a STIG item), and **Windows LAPS** for local admin passwords.
- [ ] **R: File placement.** `Lab-Handouts/Final/ocr-Lab-11-gui.pdf` is titled *Annex D, Lab 2* but sits in Annex B. Move it or re-label it so students aren't confused.
- [ ] **O: STIG validation.** Run SCAP SCC or STIG Viewer against one DC and one Exchange server after the build. This closes the loop on the TLO.
- [ ] **O: Exchange HealthChecker.** Microsoft's `HealthChecker.ps1` (CSS-Exchange) checks prerequisites, TLS, and CU level in one run.
- [ ] **O: AD Recycle Bin.** Run `Enable-ADOptionalFeature 'Recycle Bin Feature'`. It makes deleted-object recovery quick and painless.
- [ ] **O: Use Server Core for DC02.** It is smaller and closer to a STIG baseline, and it reinforces the CLI labs.
- [ ] **O: DHCP failover and WSUS on UTIL.** These give the "update server OS" and DHCP objectives a real target.
- [ ] **O: Automate the builds.** Use an `autounattend.xml` for OS installs and save the CLI blocks as a re-runnable `.ps1` per role.

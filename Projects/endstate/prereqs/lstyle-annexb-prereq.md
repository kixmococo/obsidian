# Prerequisites and Software Selection: Linux/FOSS Annex B Build

Reference for [[lstyle-annex-b-brief]]. For each recommended component this covers **what it is, what it needs, how it installs, why it was picked, and what else to consider**, including paid options where they are clearly better.

> [!warning] Verify before fielding
> Versions, repositories, and support dates change. Check them against the vendor docs on the day you build. On a DoD network, every product also needs to be on your command's approved software list or covered by the system ATO.

---

## 0. How Each Tool Was Chosen

Each pick was scored on these criteria, in priority order:

1. **Compliance content exists.** There is a DISA STIG or SCAP Security Guide profile to harden and scan it. This is what TLO 0671-IMPL-1002 asks for.
2. **It's packaged in the distro or an official repo.** Security updates then arrive through `dnf` with everything else.
3. **Long support lifecycle** and an active upstream.
4. **Maps 1:1 to an Annex B skill**, so the lessons still transfer.
5. **Small footprint** that fits a lab laptop or range.
6. **A paid support path exists** if the system goes to production.

---

## 1. Lab Platform and Sizing

### Hypervisor
| Option | Type | Notes |
|---|---|---|
| **KVM/libvirt + virt-manager** (recommended for a single Linux host) | FOSS | Built into Linux. The fencing agent is `fence_xvm` or `fence_virsh`. |
| **Proxmox VE** (recommended for a shared range) | FOSS, with an optional paid subscription | Web UI, clustering, and snapshots. The fencing agent is `fence_pve`. |
| VMware vSphere/Workstation | Paid | Mature, but licensing is now expensive after the Broadcom acquisition. Fencing agent: `fence_vmware_rest`. |

### Suggested VM Sizing (whole lab ≈ 12 vCPU / 24–28 GB RAM)
| Host | vCPU | RAM | Disk |
|---|---|---|---|
| `ipa01`, `ipa02` | 2 | 4 GB | 30 GB |
| `mx01`, `mx02` | 2 | 4 GB | 30 GB OS + **second disk** of 20–50 GB for DRBD |
| `util01` | 2 | 4–8 GB | 100 GB+ (repo mirror and backups) |
| `ws01` | 2 | 4 GB | 40 GB |

### Network Prerequisites (decide these first)
- [ ] DNS domain and realm, for example `example.mil` / `EXAMPLE.MIL`. **Don't use `.local`**, and don't reuse a domain already served by another AD or IPA.
- [ ] Static IP plan, gateway, and an upstream DNS forwarder
- [ ] An upstream **NTP** source. Kerberos fails when clocks drift more than 5 minutes apart.
- [ ] *(Optional)* a second NIC or VLAN for cluster and DRBD replication traffic

### Build Order
1. `ipa01`
2. `ipa02`
3. `util01` (enroll it as an IPA client)
4. `mx01` and `mx02`
5. `ws01`

---

## 2. Operating System: Rocky Linux 9

**What:** a free, binary-compatible rebuild of Red Hat Enterprise Linux 9.

**Requirements**
- x86-64-v2 CPU (roughly anything from 2009 onward), or aarch64
- RAM: at least 1.5 GB for Minimal, and 2–4 GB is realistic for a server role
- Disk: at least 10 GB, and 20 GB or more is recommended
- Support runs until **May 2032**

**Install**
- Boot the DVD or minimal ISO.
- Software selection: **Minimal** or **Server** (≈ Server Core), or **Server with GUI** / **Workstation** (≈ Desktop Experience).
- On the **Security Policy** screen, pick the **DISA STIG** profile so the system is hardened at install time.
- Automate installs with **Kickstart** (≈ `autounattend.xml`).

**Why this one**
- The DISA STIG for RHEL 9 and the SCAP Security Guide (SSG) both apply.
- FreeIPA, Pacemaker, Cockpit, and Ansible system roles are first-class, tested together, and all in the distro repos.
- Most of the DoD Linux footprint is RHEL-family, so the skills transfer directly.

**Alternatives**
| Option | Consider when | Caveat |
|---|---|---|
| **AlmaLinux 9** | You want an equivalent with a different community or foundation | Effectively interchangeable with Rocky |
| **Rocky/Alma 10** | Building new and you want the longest lifecycle (into the mid-2030s) | Needs an **x86-64-v3** CPU (Haswell or newer). Check the STIG and SSG status for EL10 before relying on it. |
| **Ubuntu LTS** | You need Samba AD DC (see §3), or the team already knows Debian | There is a DISA STIG for Ubuntu. Canonical's automated hardening (USG) requires Ubuntu Pro, which is free for a few machines. |
| **Debian stable** | You want minimal, community-only software | There is **no DISA STIG**, which is weak for the compliance TLO |
| **RHEL** (paid) | Production, an ATO, or vendor support is needed | **Recommended for anything fielded.** DISA publishes the STIG for RHEL itself. A free Developer subscription covers lab use (up to 16 systems). |

---

## 3. Identity, DNS, and PKI: FreeIPA (replaces AD DS, DNS, and AD CS)

**What:** an integrated identity platform: 389 Directory Server (LDAP), MIT Kerberos (SSO), BIND (DNS), Dogtag (certificate authority), SSSD on clients, plus a Web UI and the `ipa` CLI. It is the upstream of Red Hat Identity Management (IdM).

**Requirements**
- RAM:
  - 4 GB is a good lab size. 2 GB will install but is slow with the CA.
  - Red Hat sizing is about **4 GB plus 4 GB swap per 10k users** and 16 GB for 100k users.
- Hostname must be a **FQDN** that resolves forward and reverse to the static IP. It must **not** resolve to `127.0.0.1` in `/etc/hosts`.
- The **IPv6 stack must be enabled in the kernel** (`::1` on loopback). It can go unused, but booting with `ipv6.disable=1` breaks the install.
- chrony must be syncing.
- Packages (AppStream): `dnf install ipa-server ipa-server-dns`. Clients need `ipa-client`.
- Firewall:
  - `firewall-cmd --permanent --add-service={freeipa-4,dns,ntp}`
  - These open TCP 80, 443, 389, 636; TCP/UDP 88, 464, 53; and UDP 123.

**Install notes**
- Run `ipa-server-install --setup-dns --forwarder=<upstream> --auto-reverse`.
- **DoD PKI option:** `--external-ca` generates a CSR so the IPA CA can be signed by an external or enterprise root CA instead of being self-signed.
- Run at least **two servers with CA and DNS**. Keep each server to **4 or fewer replication agreements**.

**Why this one**
- It gives one product for directory, Kerberos, DNS, and CA.
- **It fixes the certificate gap from the Windows brief:** certmonger automatically issues and renews certificates for Postfix, Dovecot, and the web services.
- HBAC, sudo rules, and SELinux user maps provide Linux-native policy.
- Built-in health checking (`ipa-healthcheck`), multi-master replication, CAC/PIV smart card login, and OTP.

**Gaps compared with AD**
| AD concept | FreeIPA |
|---|---|
| OUs | None. Use groups and host groups. |
| GPO | None. Use HBAC, sudo, and pwpolicy, plus Ansible. |
| FSMO | Only the CA renewal master and CRL master roles |
| Windows clients | Cannot join |

**Alternatives**
| Option | Consider when | Caveat |
|---|---|---|
| **Samba AD DC** (Debian/Ubuntu, or Fedora) | **Windows clients must stay**, or you want OUs, GPOs managed through RSAT, FSMO, and Sites exactly as taught in Annex B | Not shipped by RHEL or Rocky. It has no built-in CA. SYSVOL does **not** replicate automatically (use rsync or osync). Functional levels have historically lagged Windows. |
| **Univention Corporate Server (UCS)** | You want Samba AD with a polished web management UI | Built on Debian. The free core edition has paid tiers. |
| OpenLDAP + MIT Kerberos + BIND (DIY) | Learning the internals | A lot of hand-integration, and no unified management |
| **Red Hat IdM** (paid, included with RHEL) | Production | Same software as FreeIPA, with vendor support and STIG-backed RHEL underneath |
| **Microsoft AD** (paid) | The client estate is mostly Windows | Still **clearly superior for Windows fleets**. GPO depth has no FOSS equal. |

---

## 4. Policy and Configuration Management: Ansible (replaces GPO)

**What:** agentless automation over SSH using YAML playbooks.

**Requirements**
- Control node: Python 3.9 or later (`dnf install ansible-core`)
- Managed nodes: SSH and Python
- Useful collections:
  - `ansible-freeipa` (RPM): manage IPA objects as code
  - `rhel-system-roles` (RPM): `ha_cluster`, `postfix`, `firewall`, `timesync`, `selinux`, `crypto_policies`

**GPO concepts mapped**

| GPO concept | Ansible equivalent |
|---|---|
| Link a GPO to an OU | Run the playbook against an IPA **host group** (dynamic inventory) |
| Background refresh | `ansible-pull` on a systemd timer |
| `gpresult` / RSoP | `ansible-playbook --check --diff` |

**STIG tie-in:** `oscap xccdf generate fix --fix-type ansible --profile stig …` writes a remediation playbook straight from SSG. DISA also publishes Ansible content for its STIGs.

**Why this one:** it's agentless, has the largest community, is native to the RHEL ecosystem, and can produce STIG remediation automatically.

**Alternatives**
| Option | Notes |
|---|---|
| **AWX** (FOSS) | Web UI, RBAC, and schedules on top of Ansible (≈ GPMC). It needs Kubernetes, so it's heavier. |
| Puppet / Salt | Agent-based and continuously enforcing, which is closer to GPO's enforcement model but adds more moving parts |
| **Ansible Automation Platform** (paid) | Supported AWX for production |

---

## 5. Messaging (replaces Exchange)

### 5a. Postfix (MTA, ≈ Transport, Send and Receive Connectors)

**Requirements**
- Packages: `dnf install postfix postfix-ldap` (BaseOS/AppStream)
- Minimal RAM
- Ports: 25 (MTA), 587 (submission), 465 (submissions). Firewall services: `smtp`, `smtp-submission`, `smtps`.

**Why this one:** it's the RHEL default MTA, uses a privilege-separated design with a strong security record, has excellent documentation, and has native LDAP lookups.

**Alternatives**
| Option | Notes |
|---|---|
| Exim | Debian default. Very flexible, but has a heavier CVE history. |
| OpenSMTPD | Simple, but a smaller ecosystem |
| Sendmail | Legacy. Avoid. |

### 5b. Dovecot (IMAP and LMTP, ≈ Mailbox Role and Databases)

**Requirements**
- Packages: `dnf install dovecot dovecot-pigeonhole` (Sieve server-side rules ≈ Inbox rules)
- Ports: 993 (IMAPS) and LMTP (local socket)

**Why this one:** it's the de facto IMAP server, with quotas, Sieve, full-text search, and the `doveadm` admin tool (import, backup, sync, who, quota).

> [!note] Version matters
> EL9 ships Dovecot **2.3**. The **2.4** community edition changed the config syntax and **removed the director and dsync replicator** features. That is why this design gets HA from DRBD and Pacemaker instead of Dovecot replication.

**Alternatives**
| Option | Notes |
|---|---|
| Cyrus IMAP | Murder clustering. Heavier and less common. |
| **Stalwart Mail Server** | A modern all-in-one written in Rust (SMTP, IMAP, JMAP, spam filtering) with built-in clustering. Promising but younger, and has an enterprise paid tier. |

### 5c. Rspamd (≈ Exchange Anti-Spam / Transport Rules)

**Requirements**
- Install from the **official rspamd.com RPM repo**. It is not in base.
- Needs **Redis or Valkey**.
- About 512 MB to 1 GB RAM.
- Web UI on 11334. Keep it local or behind a proxy.

**What it does:** spam scoring, **DKIM signing**, SPF/DMARC/ARC checks, and rate limits in one daemon. It hooks into Postfix as a milter.

**Alternatives**
| Option | Notes |
|---|---|
| SpamAssassin + Amavis + OpenDKIM | The classic stack. More parts and slower. |
| **ClamAV** (add-on, FOSS) | Antivirus scanning, which may be required by policy. Adds 1–2 GB or more of RAM for signatures. |

### 5d. Roundcube (≈ OWA)

**Requirements**
- Packages: `roundcubemail` from **EPEL** (`dnf install epel-release`)
- PHP 8.x, nginx or httpd, and a database (SQLite for a lab, PostgreSQL or MariaDB otherwise)
- Port 443

**Why this one:** it's mature, light, and has a plugin ecosystem (password change, managesieve).

**Alternatives**
| Option | Notes |
|---|---|
| **SOGo** | Webmail plus calendar, contacts, and **ActiveSync** for phones. Pick it if groupware is required. |
| SnappyMail | Lighter, mail only |

### 5e. Thunderbird (≈ Outlook)

**Install:** it's in AppStream. It autoconfigures from `autoconfig.<domain>/mail/config-v1.1.xml` or from the RFC 6186 SRV records.

**Alternatives**
| Option | Notes |
|---|---|
| Evolution | Better calendar integration, and can also speak EWS |
| `readpst` (`libpst`) | Not a client. It converts **PST → mbox** for `doveadm import` (≈ "import data from personal folders"). |

### 5f. All-in-One and Groupware Alternatives to the Hand-Built Stack
| Option | License | Consider when | Caveat |
|---|---|---|---|
| **mailcow: dockerized** | FOSS (GPLv3) | Fastest path to a full mail server (Postfix, Dovecot, Rspamd, SOGo, ClamAV) behind a web UI | Docker-based and prefers Debian/Ubuntu. Needs about **6 GB RAM**. Hides the internals, so it's worse for teaching. |
| **grommunio** | FOSS (AGPLv3), with a paid subscription available | **Outlook must keep working natively** (MAPI/HTTP, EWS, ActiveSync) with LDAP/AD integration | Check its supported-OS list. It's a bigger stack. |
| iRedMail | FOSS core, with a paid Pro tier | Script-installed classic stack | The script does a lot of things you didn't see |
| Zimbra | Paid | — | The FOSS edition no longer gets official binaries. **Avoid for new builds.** |
| **Exchange SE / Microsoft 365 (DoD tenants)** | Paid | The organization is **Outlook- and Teams-centric** | **Still clearly superior** for full groupware (calendars, delegation, shared mailboxes, mobile management). Choose it unless the FOSS requirement is firm. |

---

## 6. High Availability: Pacemaker/Corosync + DRBD + qdevice (replaces the DAG)

**What**
- **DRBD** mirrors a block device (the `F:`-drive equivalent) between `mx01` and `mx02` in real time.
- **Pacemaker** moves the filesystem, the floating VIP, Postfix, and Dovecot as one group.
- **corosync-qnetd** on `util01` acts as the tie-breaker vote (≈ the file share witness).

**Requirements**
- Pacemaker:
  - `dnf config-manager --set-enabled highavailability`, then `dnf install pcs pacemaker fence-agents-all`
  - On the witness: `corosync-qnetd`. On the nodes: `corosync-qdevice`.
- DRBD 9:
  - Install from **ELRepo** (`kmod-drbd9x`, `drbd9x-utils`). The kernel module must match the running kernel, so re-check after kernel updates.
  - LINBIT's repos are the paid alternative.
- Storage: an identical, unused block device on each node (`/dev/sdb`)
- **Fencing (STONITH) is mandatory.** Without it, a 2-node cluster can split-brain and corrupt mail. Use the hypervisor fence agent (see §1) or `sbd` with a watchdog.
- Firewall:
  - `firewall-cmd --permanent --add-service=high-availability`, which opens 2224, 3121, 5403, 5404–5412, and others
  - Add DRBD's port separately (7788/tcp by default)

**Why this one**
- It's a supported RHEL-family HA stack.
- It works on **local disks**, so there's no SAN to become a single point of failure.
- Its concepts line up with the DAG: witness and quorum, active/passive, moving the active copy, and maintenance mode with `pcs node standby`.

**Alternatives**
| Option | Consider when | Caveat |
|---|---|---|
| Dovecot dsync replication | Running Dovecot 2.3 and you want active/active mailbox sync | **Removed in the 2.4 community edition.** It's a dead end. |
| Shared iSCSI or SAN + Pacemaker | Shared storage already exists | The storage itself becomes the single point of failure |
| Ceph | You're running a larger cluster (3+ nodes) | Overkill for 2 mail nodes |
| GlusterFS | — | Effectively unmaintained now. **Avoid.** |
| Two independent MX servers plus backups (no HA) | Simplicity matters more than uptime | A failover means a restore |
| **RHEL HA Add-On / LINBIT support** (paid) | Production | Vendor-supported versions of this same stack |

---

## 7. Network Services

### Kea DHCP (≈ DHCP Server)
**Requirements**
- Install: `dnf info kea`. It's in AppStream on current EL9 minor releases. If not, use ISC's official Cloudsmith repo.
- Tiny footprint.
- UDP 67 (firewalld service `dhcp`).

**Notes**
- The HA hook (`libdhcp_ha.so`) provides hot-standby or load-balancing pairs (≈ DHCP failover) and is free.
- For dynamic DNS, let SSSD on IPA clients update their own records (`ipa-client-install --enable-dns-updates`). That's easier than wiring Kea DDNS into IPA's GSS-TSIG.

**Why this one:** it's ISC's replacement for ISC dhcpd, which reached **end of life in 2022**. It has a JSON config, a REST control agent, and HA.

**Alternatives**
| Option | Notes |
|---|---|
| dnsmasq | Fine for a lab or a small segment, but limited |
| ISC dhcpd | EOL. Don't use it for new builds. |

### Cockpit (≈ Server Manager / Windows Admin Center)
- Ships with Rocky Server: `systemctl enable --now cockpit.socket`, then `https://host:9090`.
- Add-ons: `cockpit-machines`, `cockpit-storaged`, `cockpit-podman`.
- **Keep port 9090 internal-only.** Review it against the STIG, which flags unnecessary services.

---

## 8. Patching (≈ WSUS)

| Option | Requirements | Use when |
|---|---|---|
| **`dnf reposync` + httpd mirror** (recommended for the lab) | Disk space for the mirrored repos (about 50–100 GB for BaseOS+AppStream). Point `.repo` files at `util01`. | You want simple, offline-capable updates |
| **Foreman + Katello** | **4 vCPU, 20 GB RAM minimum**, and a large `/var/lib/pulp` | You want staged **content views** (Dev → Prod ≈ WSUS approvals), errata reporting, and provisioning |
| Uyuni | Similar to Foreman/Katello (it's the upstream of SUSE Manager) | Mixed SUSE and EL estates |
| **Red Hat Satellite** (paid) | Same as Katello | Production RHEL. **Clearly better if you are on RHEL subscriptions.** |

Always on: `dnf-automatic` with security-only updates on non-cluster hosts, and `needs-restarting -r` to see whether a reboot is required.

---

## 9. Backup and Recovery (≈ Windows Server Backup + System State)

| Tool | Role | Requirements | Notes |
|---|---|---|---|
| **ReaR** (Relax-and-Recover) | Bare-metal recovery ISO + OS archive | `dnf install rear xorriso syslinux`, plus an NFS or USB target | **Test the restore.** A ReaR ISO that has never been booted is not a backup. |
| **restic** | File and mailstore backups | EPEL. Target: `rest-server`, SFTP, or S3-compatible storage. | Encrypted and deduplicated, a single binary. Best for the lab. |
| **Bareos** | Enterprise backup (director, catalog, schedules, retention, tape) | Bareos repos, plus PostgreSQL for the catalog | Pick it when you need scheduled jobs and reporting (≈ a real backup product) |
| `ipa-backup` / `ipa-restore` | IdM "system state" | Built in | A full backup stops IPA briefly. Use `--data --online` for a hot data-only backup. |

**Alternatives**
- **BorgBackup** is similar to restic.
- **Bacula** is what Bareos forked from.
- **Paid: Veeam** (agents for Linux, plus hypervisor-level backups) is clearly easier at scale if you already have it.

---

## 10. Monitoring and Logging (≈ PerfMon, Event Viewer, SCOM)

| Tool | Requirements | Why or when |
|---|---|---|
| **Zabbix** (recommended) | Server: about 2 vCPU / 2–4 GB plus PostgreSQL. Install from the **repo.zabbix.com** LTS releases. Agent port 10050, server port 10051. | One product for agents, templates (Linux, DNS, Postfix), alerting, and dashboards. That's closest to SCOM and easiest to teach. |
| Prometheus + Grafana + node_exporter | Small, but you assemble the parts | Metrics-first and cloud-native. Good if Kubernetes is in your future. |
| Checkmk Raw / Icinga 2 | Moderate | Strong auto-discovery (Checkmk) or classic Nagios-style checks (Icinga) |
| Built-in tools | — | `journalctl`, `sar` (sysstat), `htop`, `iostat`, and Cockpit metrics |
| Central logging | **rsyslog** forwarding to `util01`. Add Graylog, OpenSearch, or Grafana Loki for search. | ≈ Event forwarding |
| **Splunk** (paid) | Heavy | **Clearly better log analytics**, and common across DoD |

---

## 11. Compliance (≈ SCAP Compliance Checker / STIG Viewer)

**Requirements:** `dnf install openscap-scanner scap-security-guide`. Add `scap-workbench` if you want a GUI.

**Steps**
1. Find the content file with `ls /usr/share/xml/scap/ssg/content/`. Use the `ssg-*9-ds.xml` file that matches your distro.
2. Scan: `oscap xccdf eval --profile stig --results res.xml --report report.html <ds.xml>`
3. Fix: `--remediate` applies fixes immediately, or `oscap xccdf generate fix --fix-type ansible` writes a playbook.

**Why this one:** it's built from the same ComplianceAsCode content that Red Hat ships and that DISA STIG automation is checked against.

**Government and paid tools to pair with it**
- DISA **SCC** and **STIG Viewer** are free for government use but not FOSS. Use them for the official checklist (`.ckl`/`.cklb`).
- **ACAS (Tenable)** is paid and DoD-mandated for vulnerability scanning.

---

## 12. Prerequisite Summary Checklist
- [ ] Hypervisor chosen, with a fence agent available for it
- [ ] Domain name, realm, IP plan, upstream DNS forwarder, and NTP source decided
- [ ] Rocky 9 ISO on hand. Kickstart file written (optional).
- [ ] Repos enabled where needed:
  - [ ] AppStream/BaseOS (default)
  - [ ] **EPEL** (Roundcube, restic)
  - [ ] **highavailability** (Pacemaker)
  - [ ] **ELRepo** (DRBD)
  - [ ] **rspamd.com** (Rspamd)
  - [ ] **repo.zabbix.com** (Zabbix)
- [ ] Second disk attached to `mx01` and `mx02` for DRBD
- [ ] IPv6 left enabled in the kernel on IPA servers. FQDN, forward, and reverse DNS resolve correctly.
- [ ] Firewall service list planned per host:
  - [ ] ipa: `freeipa-4 dns ntp`
  - [ ] mx: `smtp smtp-submission smtps imaps https high-availability` + 7788/tcp
  - [ ] util: `dhcp http high-availability` + 10051/tcp
- [ ] Decision recorded: **FreeIPA** (all-Linux) or **Samba AD** (Windows clients remain)
- [ ] Decision recorded: hand-built mail stack (teaching) or **mailcow/grommunio** (speed, Outlook support)

---

## 13. Supplementary: Software Quick Reference

Day-to-day reference for each tool: what it is, its license, where its config and logs live, and the commands you'll use most. Licenses are given so you can confirm the stack stays FOSS.

> [!tip] Two rules that apply to almost everything below
> - **Configs are text files under `/etc`.** Back them up or track them in git (`etckeeper`). Always run the tool's config test before restarting it.
> - **Logs go to the journal**: `journalctl -u <service> -f` (follow) or `-b` (this boot). Some services also write to `/var/log/…`.

### 13.1 Platform

| Tool | What it is (≈ Windows) | License | Key paths | Key commands |
|---|---|---|---|---|
| **Rocky Linux 9** | The OS (≈ Windows Server) | GPL and other OSI licenses | `/etc/os-release`, `/etc/yum.repos.d/` | `cat /etc/rocky-release`, `dnf repolist` |
| **dnf** | Package and update manager (≈ Windows Update + installers) | GPLv2+ | `/etc/dnf/dnf.conf`, `/var/log/dnf.log` | `dnf updateinfo list --security`, `dnf history`, `dnf history undo <id>` |
| **Kickstart / Anaconda** | Unattended installer (≈ `autounattend.xml`) | GPLv2+ | `/root/anaconda-ks.cfg` (written after every install, which makes it a good template) | `ksvalidator file.ks`, boot with `inst.ks=http://…/file.ks` |
| **systemd / journald** | Service manager and logs (≈ services.msc, Event Viewer, Task Scheduler) | LGPLv2.1+ | `/etc/systemd/system/` | `systemctl status\|enable --now\|restart <svc>`, `journalctl -u <svc> -f`, `systemctl list-timers` |
| **NetworkManager** | Network config (≈ adapter properties) | GPLv2+ | `/etc/NetworkManager/system-connections/*.nmconnection` | `nmcli dev status`, `nmcli con show`, `nmtui` |
| **firewalld** | Host firewall (≈ Defender Firewall). Zones play the role of Domain/Private/Public profiles. | GPLv2+ | `/etc/firewalld/` | `firewall-cmd --get-active-zones`, `--list-all`, `--set-log-denied=all` |
| **chrony** | NTP client and server (≈ w32time) | GPLv2 | `/etc/chrony.conf` | `chronyc sources -v`, `chronyc tracking`, `chronyc clients` |
| **SELinux** | Mandatory access control. No direct Windows equivalent; the closest are integrity levels and AppLocker. | GPL | `/etc/selinux/config`, `/var/log/audit/audit.log` | `getenforce`, `ausearch -m avc -ts recent`, `sealert -a /var/log/audit/audit.log`, `restorecon -Rv <path>` |
| **Cockpit** | Web admin console (≈ Server Manager / WAC) | LGPLv2.1+ | `/etc/cockpit/` | `systemctl enable --now cockpit.socket`, then `https://host:9090` |

### 13.2 Identity (FreeIPA and its parts)

| Tool | What it is (≈ Windows) | License | Key paths | Key commands |
|---|---|---|---|---|
| **FreeIPA** (framework) | Domain controller suite (≈ AD DS) | GPLv3+ | `/etc/ipa/default.conf`, `/var/log/ipaserver-install.log`, `/var/log/ipareplica-install.log` | `ipactl status\|restart`, `kinit admin`, `ipa <object>-find`, `ipa-healthcheck --failures-only` |
| **389 Directory Server** | The LDAP directory (≈ NTDS.dit) | GPLv3+ | `/etc/dirsrv/slapd-<REALM>/`, `/var/log/dirsrv/slapd-<REALM>/errors` | `dsctl <instance> status`, `ipa topologysegment-find domain` |
| **MIT Kerberos** | Authentication (≈ Kerberos in AD) | MIT | `/etc/krb5.conf`, `/var/log/krb5kdc.log` | `kinit`, `klist` (same as Windows), `kdestroy` |
| **BIND** (`named`) | DNS server. IPA stores its zones in LDAP. | MPL 2.0 | `/etc/named.conf`. Put customizations in `/etc/named/ipa-ext.conf` and `ipa-options-ext.conf`. | `rndc status`, `dig @localhost <name>`, `ipa dnszone-find` |
| **Dogtag PKI** | Certificate authority (≈ AD CS) | GPLv2 | `/var/log/pki/pki-tomcat/` | `ipa cert-find`, `ipa cert-show <serial>`, `ipa-cacert-manage renew` |
| **certmonger** | Certificate auto-request and renewal (≈ cert autoenrollment) | GPLv3+ | `/var/lib/certmonger/` | `getcert list`, `ipa-getcert request …`, `getcert resubmit -i <id>` |
| **SSSD** (clients) | Login, caching, and policy client (≈ Netlogon + the cached-credentials client) | GPLv3+ | `/etc/sssd/sssd.conf`, `/var/log/sssd/` | `sssctl domain-status example.mil`, `id <user>`, `sss_cache -E` |
| *(alt)* **Samba AD DC** | AD-compatible DC (Debian/Ubuntu) | GPLv3 | `/etc/samba/smb.conf`, `/var/lib/samba/` | `samba-tool domain provision`, `samba-tool drs showrepl`, `samba-tool fsmo show` |

### 13.3 Configuration Management

| Tool | What it is (≈ Windows) | License | Key paths | Key commands |
|---|---|---|---|---|
| **Ansible (ansible-core)** | Agentless automation (≈ GPO + PowerShell DSC) | GPLv3+ | `ansible.cfg`, `inventory`, `playbooks/`, `roles/` | `ansible all -m ping`, `ansible-playbook site.yml -C -D` (dry run with diff), `ansible-doc <module>` |
| **ansible-freeipa** | Modules to manage IPA users, groups, HBAC, and so on as code | GPLv3 | `/usr/share/ansible/collections/…/freeipa/` | `ansible-doc freeipa.ansible_freeipa.ipauser` |
| **rhel-system-roles** | Supported roles: `ha_cluster`, `postfix`, `firewall`, `timesync`, `selinux`, `crypto_policies` | MIT/GPL (varies by role) | `/usr/share/ansible/roles/`, with per-role `README.md` under `/usr/share/doc/` | Include the role in a playbook: `roles: [rhel-system-roles.timesync]` |
| **ansible-pull** | Pull-mode runs on a timer (≈ the background GPO refresh) | GPLv3+ | A git repo of playbooks, plus a systemd timer | `ansible-pull -U <git-url> local.yml` |

### 13.4 Messaging

| Tool | What it is (≈ Windows) | License | Key paths | Key commands |
|---|---|---|---|---|
| **Postfix** | MTA (≈ the Transport service plus send and receive connectors) | IPL 1.0 / EPL 2.0 | `/etc/postfix/main.cf`, `master.cf`, `ldap-*.cf`, `/var/log/maillog` | `postconf -n` (shows non-default settings), `postfix check`, `postqueue -p`, `postcat -q <id>`, `postsuper -d <id>` |
| **Dovecot** | IMAP, LMTP, and SASL auth (≈ the Mailbox role and databases) | MIT + LGPLv2.1 | `/etc/dovecot/dovecot.conf`, `conf.d/` | `doveconf -n`, `doveadm who`, `doveadm mailbox list -u <user>`, `doveadm quota get -u <user>`, `doveadm backup`, `doveadm import` |
| **Pigeonhole (Sieve)** | Server-side mail rules (≈ Inbox rules) | LGPLv2.1 | `~/sieve/`, `conf.d/90-sieve.conf` | Edited through Roundcube's managesieve plugin |
| **Rspamd** | Anti-spam, DKIM, and SPF/DMARC (≈ the anti-spam agents) | Apache 2.0 | **Only edit `/etc/rspamd/local.d/` and `override.d/`.** Log: `/var/log/rspamd/rspamd.log` | `rspamadm configtest`, `rspamc stat`, `rspamadm dkim_keygen -d example.mil -s mail` |
| **Redis / Valkey** | Key-value store used by Rspamd | EL9 `redis` 6.x is BSD. Redis 7.4+ changed its license, so for newer versions use **Valkey** (BSD-3). | `/etc/redis/` or `/etc/valkey/` | `redis-cli ping` / `valkey-cli ping` |
| **Roundcube** | Webmail (≈ OWA) | GPLv3+ | `/etc/roundcubemail/config.inc.php`, `/var/log/roundcubemail/` | `/usr/share/roundcubemail/bin/update.sh` after upgrades |
| **Thunderbird** | Mail client (≈ Outlook) | MPL 2.0 | `~/.thunderbird/<profile>/` | Autoconfig: `https://autoconfig.<domain>/mail/config-v1.1.xml` |
| **libpst (`readpst`)** | PST → mbox converter | GPLv2+ | — | `readpst -r -o out/ file.pst`, then `doveadm import` |
| **imapsync** | Mailbox migration between any two IMAP servers | NOLIMIT Public License (permissive) | — | `imapsync --host1 old --user1 u --host2 new --user2 u --dry` |
| **swaks** | SMTP test tool (≈ `Send-MailMessage` / telnet 25) | GPLv2 | — | `swaks --to u@example.mil --server mx01 --tls` |
| **pflogsumm** | Postfix log summary report | GPLv2+ | — | `pflogsumm -d today /var/log/maillog` |

### 13.5 High Availability

| Tool | What it is (≈ Windows) | License | Key paths | Key commands |
|---|---|---|---|---|
| **Corosync** | Cluster membership and quorum (≈ the Failover Cluster service) | BSD | `/etc/corosync/corosync.conf` (**managed by pcs, so don't hand-edit**) | `corosync-quorumtool -s`, `corosync-cfgtool -s` |
| **Pacemaker** | Resource manager (≈ DAG Active Manager) | GPLv2+ / LGPLv2.1+ | CIB at `/var/lib/pacemaker/cib/` (**never edit directly**), `/var/log/pacemaker/pacemaker.log` | `crm_mon -1`, `pcs status --full` |
| **pcs** | CLI and web UI for Corosync + Pacemaker (≈ Failover Cluster Manager) | GPLv2 | — | `pcs resource move\|clear\|cleanup`, `pcs constraint`, `pcs node standby\|unstandby`, `pcs config backup` |
| **DRBD 9** | Network-replicated block device (≈ database copy replication) | GPLv2 | `/etc/drbd.d/*.res` | `drbdadm status`, `drbdadm up\|down\|primary\|secondary <res>`, `drbdadm verify <res>` |
| **corosync-qnetd / qdevice** | Tie-breaker vote (≈ file share witness) | BSD | `/etc/corosync/qnetd/` | `corosync-qnetd-tool -l` (on the witness), `pcs quorum status` |
| **fence-agents** | STONITH power control (no Windows equivalent) | GPLv2+ / LGPLv2+ | Configured as `pcs stonith` resources | `fence_ipmilan -a <bmc> -l u -p p -o status`, `pcs stonith fence <node>` (**test it once**) |

### 13.6 Network Services, Patching, and Backup

| Tool | What it is (≈ Windows) | License | Key paths | Key commands |
|---|---|---|---|---|
| **Kea DHCPv4** | DHCP server | MPL 2.0 | `/etc/kea/kea-dhcp4.conf`, leases in `/var/lib/kea/kea-leases4.csv` | `kea-dhcp4 -t /etc/kea/kea-dhcp4.conf` (config test), `journalctl -u kea-dhcp4` |
| **dnf reposync + httpd** | Local update mirror (≈ WSUS) | GPLv2+ / Apache 2.0 | `/srv/repo/`, `/etc/httpd/conf.d/` | `dnf reposync --repoid=baseos --download-metadata -p /srv/repo` |
| **dnf-automatic** | Scheduled updates (≈ automatic updates by GPO) | GPLv2+ | `/etc/dnf/automatic.conf` (`upgrade_type = security`) | `systemctl enable --now dnf-automatic.timer` |
| **Foreman + Katello** | Content views, errata, provisioning (≈ WSUS/SCCM) | GPLv3 | `/etc/foreman/`, `/var/log/foreman/` | `hammer` CLI, `foreman-maintain health check` |
| **ReaR** | Bare-metal recovery (≈ Windows Server Backup bare-metal recovery) | GPLv3 | `/etc/rear/local.conf`, `/var/log/rear/` | `rear -v mkbackup`. Restore by booting the ISO and running `rear recover`. |
| **restic** | Encrypted, deduplicated file backup | BSD-2 | `RESTIC_REPOSITORY`, `RESTIC_PASSWORD_FILE` | `restic backup /srv/mail`, `restic snapshots`, `restic restore latest --target /tmp/r`, `restic check`, `restic forget --keep-daily 7 --prune` |
| **Bareos** | Enterprise backup with director, catalog, and tape | AGPLv3 | `/etc/bareos/` | `bconsole` (`status dir`, `run job=…`, `restore`), plus the Bareos WebUI |
| `ipa-backup` / `ipa-restore` | IdM system-state backup | GPLv3+ | `/var/lib/ipa/backup/` | `ipa-backup --data --online`, `ipa-restore <dir>` |

### 13.7 Monitoring, Logging, Compliance

| Tool | What it is (≈ Windows) | License | Key paths | Key commands |
|---|---|---|---|---|
| **Zabbix** (7.x) | Monitoring and alerting (≈ SCOM) | AGPLv3 (since 7.0) | `/etc/zabbix/zabbix_server.conf`, `zabbix_agent2.conf`, `/var/log/zabbix/` | `zabbix_get -s <host> -k agent.ping`, `zabbix_agent2 -t <key>` |
| **rsyslog** | Log forwarding and collection (≈ event forwarding) | GPLv3 / Apache 2.0 | `/etc/rsyslog.conf`, `/etc/rsyslog.d/` | Client: `*.* @@util01:514`. Test: `logger "test"`. Check config: `rsyslogd -N1`. |
| **sysstat (`sar`)** | Historical performance counters (≈ PerfMon data collector sets) | GPLv2+ | `/var/log/sa/` | `sar -u`, `sar -r`, `sar -n DEV`, `iostat -x 2` |
| **OpenSCAP** | SCAP scanner (≈ SCC) | LGPLv2.1+ | Results go wherever you point `--results`/`--report` | `oscap info <ds.xml>` (lists profiles), `oscap xccdf eval --profile stig …` |
| **SCAP Security Guide** | STIG and CIS content (ComplianceAsCode) | BSD-3 | `/usr/share/xml/scap/ssg/content/` | Also has Ansible and Bash remediations under `/usr/share/scap-security-guide/` |

### 13.8 All-in-One Alternatives

| Tool | License | Notes |
|---|---|---|
| mailcow: dockerized | GPLv3 | Managed through its web UI. Updates run with `./update.sh`. Everything lives in Docker volumes. |
| grommunio | AGPLv3 | Admin web UI. It runs its own MAPI/EWS stack (gromox). |
| SOGo | GPLv2 / LGPLv2 | Groupware, CalDAV/CardDAV, ActiveSync |

---

## 14. Supplementary: Linux Concepts for Windows Admins

| Windows habit | Linux equivalent | Gotcha |
|---|---|---|
| services.msc | `systemctl` | Services need `enable` to start at boot, as well as `start` to run now. `enable --now` does both. |
| Event Viewer | `journalctl` | Logs don't persist across reboots unless `/var/log/journal` exists. The STIG profile creates it. |
| Registry | Text files in `/etc` | Run the config test before restarting: `postfix check`, `named-checkconf`, `kea-dhcp4 -t`, `rsyslogd -N1`, `rspamadm configtest` |
| Run as administrator / UAC | `sudo` (rules come from IPA) | Never work as `root` interactively. sudo leaves an audit trail. |
| NTFS ACLs | POSIX permissions + ACLs + **SELinux labels** | Moving data (for example the mail store to `/srv/mail`) needs its SELinux label copied over: `semanage fcontext -a -e /var/spool/mail /srv/mail && restorecon -Rv /srv/mail` |
| `ipconfig` / `netstat` / `tracert` / `nslookup` | `ip a`, `ss -tulpn`, `tracepath`, `dig` | `dig` shows the full answer, including TTL and which server answered |
| Defender Firewall profiles | firewalld **zones** | `--permanent` changes only apply after `--reload`. Runtime-only changes are lost at reboot. |
| Task Scheduler | systemd **timers** (or cron) | `systemctl list-timers` |
| PowerShell remoting | SSH + Ansible | Use key-based or Kerberos (GSSAPI) SSH. No passwords. |
| Reboot after updates | `needs-restarting -r` | Restart only the services `needs-restarting -s` lists, unless the kernel or glibc changed |

### Official Documentation
- Rocky Linux: docs.rockylinux.org. Red Hat's RHEL 9 docs at docs.redhat.com also apply, including IdM, HA, and Security Hardening.
- FreeIPA: freeipa.org. Red Hat IdM guides at docs.redhat.com.
- Postfix: postfix.org/documentation.html
- Dovecot: doc.dovecot.org (choose the 2.3 docs for EL9)
- Rspamd: rspamd.com/doc
- Roundcube: github.com/roundcube/roundcubemail/wiki
- ClusterLabs (Pacemaker/Corosync): clusterlabs.org
- DRBD: linbit.com (DRBD 9 User Guide)
- Kea: kea.readthedocs.io
- Ansible: docs.ansible.com
- ReaR: relax-and-recover.org
- restic: restic.readthedocs.io
- Bareos: docs.bareos.org
- Zabbix: zabbix.com/documentation
- OpenSCAP and SSG: open-scap.org and github.com/ComplianceAsCode/content

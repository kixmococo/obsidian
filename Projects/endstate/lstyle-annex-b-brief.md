# Annex B Brief, Linux/FOSS Edition

Companion to [[annex-b-brief]]. It reaches the **same end goals** using only free and open-source software on Linux. For install requirements, why each tool was chosen, and the alternatives, see [[lstyle-annexb-prereq]].

---

## Bottom Line

From bare install media, **build, join, secure, mail-enable, back up, and keep running a small Linux enterprise**, then prove that it works. Compared with the Windows version, STIG hardening and certificates are built into the stack from day one instead of being bolted on later.

## Recommended Stack

| Windows (Annex B) | FOSS equivalent | Notes |
|---|---|---|
| Windows Server 2022 | **Rocky Linux 9** (Minimal / Server) | RHEL-compatible, so the RHEL 9 STIG content applies |
| Windows 10/11 client | Rocky Linux 9 Workstation (GNOME) | Fedora or Ubuntu LTS also work |
| AD DS + DNS + AD CS | **FreeIPA**: 389 DS (LDAP), MIT Kerberos, BIND, Dogtag CA, SSSD | Use **Samba AD DC** instead if Windows clients must stay (see [[lstyle-annexb-prereq]]) |
| Group Policy | **FreeIPA policy** (HBAC, sudo, password policy) + **Ansible** for config baselines | |
| Server Manager / `sconfig` | **Cockpit** web console / `nmtui` | |
| Exchange Mailbox / Transport | **Postfix** (SMTP) + **Dovecot** (IMAP/LMTP) + **Rspamd** (filtering and DKIM) | |
| OWA | **Roundcube** | |
| Outlook | **Thunderbird** | |
| DAG + file share witness | **Pacemaker/Corosync + DRBD**, with **corosync-qnetd** on UTIL as the witness | |
| DHCP Server | **Kea DHCP** | |
| WSUS | Local `dnf reposync` mirror (light) or **Foreman/Katello** (full) | |
| Windows Server Backup | **ReaR** (bare-metal recovery) + **restic** or **Bareos** (files) + `ipa-backup` | |
| PerfMon / Event Viewer | **Zabbix** (or Prometheus + Grafana), `journalctl`, `sar` | |
| SCAP / STIG Viewer | **OpenSCAP** + **SCAP Security Guide** | |

## Target Build

| Host | Replaces | Software | Example |
|---|---|---|---|
| `ipa01` | DC01 | FreeIPA server, DNS, CA (CA renewal master) | `192.168.40.2/24` |
| `ipa02` | DC02 | FreeIPA replica, DNS, CA | `192.168.40.3/24` |
| `mx01` / `mx02` | EX01 / EX02 | Postfix, Dovecot, Rspamd, Roundcube, Pacemaker, DRBD | `/dev/sdb` (DRBD) ≈ the `F:` drive |
| `util01` | UTIL | corosync-qnetd (witness), Kea, repo mirror, backup target, Zabbix | |
| `ws01` | Client | ipa-client, Thunderbird, `ipa` CLI (≈ RSAT) | |

---

## End-Goal Checklist

### 1. Deploy OS (≈ DS.02.01–02)
- [ ] Pick the install type, **Minimal** or **Server with GUI** (≈ Core or Desktop Experience), and justify it. Linux has no edition or licensing choice to make.
- [ ] Install Rocky 9 from the ISO. Under **Security Policy**, select the **DISA STIG** profile so the system is hardened during install.
- [ ] Install a workstation (Rocky Workstation)
- [ ] Navigate the system. The "registry" equivalents are `/etc/*`, `sysctl -a`, and `dconf` (desktop settings).

### 2. Base-Configure Each Server (≈ DS.02.03, Lab 1)
- [ ] `hostnamectl set-hostname ipa01.example.mil` (use the **FQDN**, which FreeIPA requires)
- [ ] `nmcli con mod <nic> ipv4.method manual ipv4.addresses 192.168.40.2/24 ipv4.gateway 192.168.40.1 ipv4.dns "192.168.40.3 127.0.0.1"`, then `nmcli con up <nic>`
- [ ] Set time with `timedatectl set-timezone …` and point **chrony** at the NTP source (`chronyc sources -v`)
- [ ] **Keep firewalld on.** Open only the services you need, for example `firewall-cmd --permanent --add-service={freeipa-4,dns,ntp}`, then `--reload`.
- [ ] Verify: `ip a`, `ip r`, `ping`, `dig`, `ss -tulpn`
- [ ] Repeat the build with `nmtui` and with Cockpit (`https://host:9090`)

### 3. Domain Services (≈ DS.02.04)
- [ ] On `ipa01`: `dnf install ipa-server ipa-server-dns`, then `ipa-server-install --setup-dns --forwarder=<upstream> --auto-reverse`
- [ ] On `ipa02`: run `ipa-client-install`, then `ipa-replica-install --setup-dns --setup-ca` (≈ promoting an additional DC)
- [ ] Add DNS records: `ipa dnsrecord-add example.mil host1 --a-rec=… --a-create-reverse`
- [ ] Organize objects (≈ OUs). FreeIPA has no OUs, so use **user groups**, **host groups**, and **automember** rules instead.
- [ ] Use the admin tools (≈ ADUC/ADAC): the Web UI (`https://ipa01`) and the `ipa` CLI
- [ ] Identify the role holders (≈ FSMO) with `ipa config-show` (CA renewal master) and `ipa-crlgen-manage status`. Move the role with `ipa config-mod --ca-renewal-master-server=ipa02…`
- [ ] Set up sites (≈ AD Sites): `ipa location-add`, `ipa server-mod --location=…`

### 4. Configure Services and Policy (≈ DS.02.05, Lesson 3)
- [ ] Create users and groups (`ipa user-add`, `ipa group-add`, `ipa group-add-member`) and delegate rights with RBAC (`ipa role-add`, `ipa privilege-*`)
- [ ] Pre-stage a host with `ipa host-add ws01 --random` (≈ pre-staging a computer object)
- [ ] Join hosts to the domain with `ipa-client-install --mkhomedir` (≈ "join a domain")
- [ ] Add a conditional forwarder: `ipa dnsforwardzone-add yymeu.usmc.mil --forwarder=10.x.x.x --forward-policy=only`
- [ ] **Policy (≈ GPO):**
  - [ ] Access control: `ipa hbacrule-add` (who can log into what). Disable the default `allow_all` rule.
  - [ ] Privilege: `ipa sudorule-add` (≈ restricted groups / admin rights)
  - [ ] Password and lockout policy: `ipa pwpolicy-mod --minlength=15 …`
  - [ ] Config baseline: an **Ansible** playbook targeted at IPA host groups (≈ linking a GPO to an OU)
  - [ ] Periodic refresh: run `ansible-pull` on a systemd timer (≈ the 90-minute GPO refresh)
- [ ] Force and verify policy (≈ `gpupdate` / `gpresult`): `sss_cache -E`, `ipa hbactest --user=u --host=h --service=sshd`, `ansible-playbook --check --diff`
- [ ] Set up DHCP on `util01` with **Kea**: a subnet and pool, with option 3 (routers), option 6 (DNS), and option 15 (domain). Use switch DHCP snooping in place of "authorize in AD".

### 5. Messaging (≈ DS.02.06, Lab 11)
- [ ] Install the prerequisites: `postfix postfix-ldap dovecot`, Rspamd + Redis/Valkey, Roundcube (EPEL), PHP, nginx/httpd, and `pcs pacemaker` + DRBD (see [[lstyle-annexb-prereq]])
- [ ] Issue a SAN certificate (mail, webmail, autoconfig, mx01, mx02) from the IPA CA: `ipa service-add smtp/mx01…`, then `ipa-getcert request …`. **certmonger auto-renews it.**
- [ ] Put the mail store on a DRBD volume on `/dev/sdb`, mounted at `/srv/mail` (≈ moving the DB and logs to `F:`)
- [ ] Look up recipients in FreeIPA: Postfix uses `virtual_mailbox_maps` = LDAP, and Dovecot uses `passdb`/`userdb` through PAM/SSSD or LDAP.
- [ ] Create mailboxes (≈ enable a mailbox): give each IPA user a `mail` attribute. Dovecot creates the Maildir on first delivery.
- [ ] Mail-enabled groups: a Postfix `virtual_alias_maps` LDAP query that expands IPA group members to their mail addresses
- [ ] Send connector: Postfix `relayhost` / `transport_maps` (scoped outbound)
- [ ] Receive connectors: `master.cf`, with `smtp` on port 25 (inbound) and `submission` on 587 or `submissions` on 465 (authenticated clients)
- [ ] DNS records:
  - [ ] Round-robin A records for `mail` and `webmail`
  - [ ] `ipa dnsrecord-add example.mil @ --mx-rec="10 mx01.example.mil."` (repeat for `mx02`)
  - [ ] SRV records `_imaps._tcp` and `_submission._tcp` (RFC 6186)
  - [ ] `autoconfig.example.mil`, serving `/mail/config-v1.1.xml` (≈ autodiscover for Thunderbird)
- [ ] Web access (≈ OWA/ECP): Roundcube at `https://webmail.example.mil`
- [ ] Clients (≈ Outlook): Thunderbird autoconfigures. Import a PST with `readpst` (libpst), then `doveadm import`.
- [ ] Migrate a mailbox (≈ `New-MoveRequest`): `doveadm backup` or `imapsync`
- [ ] Inter-site mail flow: a conditional forwarder and an MX record on **both** sides. Test with `swaks --to user@yymeu…`.
- [ ] **HA (≈ DAG):**
  - [ ] Create the cluster: `pcs cluster setup mailclu mx01 mx02`
  - [ ] Add the witness: `pcs quorum device add model net host=util01 algorithm=ffsplit` (≈ file share witness)
  - [ ] Create the resources: DRBD (promotable) → Filesystem → VIP → Postfix and Dovecot, all in one group
  - [ ] **Configure fencing (STONITH).** The cluster is not safe without it (`fence_vmware_rest`, `fence_xvm`, or `sbd`).

### 6. Maintain the OS (≈ DS.02.07)
- [ ] Patch: `dnf upgrade`, `dnf-automatic` for security updates, `needs-restarting -r`, and `dnf history undo <id>` to roll back
- [ ] Serve updates locally (≈ WSUS): point clients at the repo mirror or Foreman/Katello on `util01`
- [ ] Back up the OS with `rear mkbackup` (bootable ISO + archive) and **test** a bare-metal restore
- [ ] Shut down mail gracefully: `pcs node standby mx01` (≈ maintenance mode), patch, then `pcs node unstandby mx01`
- [ ] Troubleshoot with `journalctl -xe`, `systemctl status`, `ss`, `ip`, `tracepath`, `tcpdump`, `firewall-cmd --list-all`, and `ausearch` / `sealert` for SELinux denials

### 7. Maintain Services (≈ DS.02.08)
- [ ] Back up IdM (≈ system state): `ipa-backup` (full) or `ipa-backup --data --online`
- [ ] Restore IdM with `ipa-restore`. Non-authoritative restore ≈ re-initialize from a peer (`ipa-replica-manage re-initialize --from ipa01`).
- [ ] Undo a deletion (≈ AD Recycle Bin): `ipa user-del --preserve`, then `ipa user-undel`
- [ ] Check health and replication (≈ `dcdiag` / `repadmin`): `ipa-healthcheck --failures-only`, `ipa topologysuffix-verify domain`, `ipa-replica-manage list -v`
- [ ] Monitor DNS: `dig @ipa01 …`, `rndc status`, `journalctl -u named`
- [ ] Restart and update services: `ipactl restart|status`, and `dnf upgrade ipa-*` on one server at a time
- [ ] Remove a client from the domain: run `ipa-client-install --uninstall` on the client, then `ipa host-del ws01`
- [ ] Mail operations:
  - [ ] Move the active node (≈ moving the active DB): `pcs resource move mailgrp mx02`, then `pcs resource clear`
  - [ ] Health (≈ `Test-ServiceHealth`): `pcs status`, `drbdadm status`, `postfix check`, `doveadm who`
  - [ ] Trace mail (≈ message tracking): `postqueue -p`, `grep <queue-id> /var/log/maillog`, `pflogsumm`
  - [ ] Back up mail with `restic` (or Bareos) of `/srv/mail`, or `doveadm backup` to the backup target
- [ ] Monitor performance (≈ PerfMon / SCOM): Zabbix agents on every host, `sar`, `htop`, and Cockpit metrics

---

## Final Proof of Function (Sign-off)
- [ ] `ipa-healthcheck --failures-only` is clean on both IPA servers, and `ipa topologysuffix-verify domain` passes
- [ ] An IPA user logs into `ws01`, `ipa hbactest` shows the expected allow/deny, and the Ansible run reports **0 changed**
- [ ] `dig` returns the MX, both A records, and the SRV records, and the certificate validates in Thunderbird with **no warning**
- [ ] User A can mail User B internally **and** at a neighbor site (`swaks` + Thunderbird)
- [ ] `pcs node standby mx01` does not stop mail, and the service follows the VIP to `mx02`
- [ ] A preserved user is restored, and one ReaR or restic restore has been tested
- [ ] The OpenSCAP STIG scan is green or has documented exceptions: `oscap xccdf eval --profile stig --report r.html <ssg-*9-ds.xml>` (find the file with `ls /usr/share/xml/scap/ssg/content/`)

---

## Optional / Recommended Extras
- [ ] **R: Disable root SSH and shared passwords.** Use IPA sudo rules and SSH keys or Kerberos. This replaces LAPS and shared `xadministrator` credentials.
- [ ] **R: Centralize logs.** Forward rsyslog to `util01` (≈ event forwarding).
- [ ] **R: Automate the builds.** Use Kickstart (≈ `autounattend.xml`) and the `ansible-freeipa` and `rhel-system-roles` collections (`ha_cluster`, `postfix`, `firewall`, `timesync`, `selinux`).
- [ ] **O: Groupware.** If calendars, contacts, or ActiveSync are required, swap in **SOGo** (or run **mailcow**). For native Outlook/MAPI support, use **grommunio** (see [[lstyle-annexb-prereq]]).
- [ ] **O: MFA.** FreeIPA supports OTP/2FA and smart cards (CAC/PIV) out of the box.

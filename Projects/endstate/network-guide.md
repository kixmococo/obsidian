# Network Build Guide (FOSS)

Checklist for building the network that the [[lstyle-annex-b-brief]] servers run on. For hardware specs, software rationale, and the VLAN/IP plan, see [[network-prereqs]]. Each phase ends with a **✓ verify** step.

## Pick Your Path

| Step | **Part 1: Hardware-based** | **Part 2: VM-based** |
|---|---|---|
| 1 | S0 Plan | S0 Plan |
| 2 | H1 Rack, power, cabling | V1 Hypervisor install |
| 3 | H2 Switch | V2 Virtual networking |
| 4 | **FW-A *or* FW-B** on an appliance | V3 Firewall VM, then **FW-A *or* FW-B** |
| 5 | H3 Server BIOS/BMC prep | V4 Cluster and storage *(multi-host)* |
| 6 | H4 Provisioning server (PXE + Kickstart) | V5 Backup server (PBS) |
| 7 | H5 Install servers | V6 VM template and clones |
| 8 | H6 Fencing prep | V7 UPS |
| 9 | H7 Backup and UPS | — |
| 10 | S5 Ops services → S6 Hardening → S7 Sign-off | S5 Ops services → S6 Hardening → S7 Sign-off |

**Hybrid build:** use H1, H2, and FW-A/FW-B on an appliance, then V1 onward for the servers.

---

## S0. Plan and Document (shared)
- [ ] Draw the topology from [[network-prereqs]] (Part 1 §1.1 or Part 2 §2.1) in draw.io
- [ ] Finalize the VLAN/IP table, DNS/NTP chain, and firewall matrix ([[network-prereqs]] S1)
- [ ] Choose the firewall option: **FW-A (OPNsense)** or **FW-B (Linux router)** ([[network-prereqs]] S2)
- [ ] Choose a naming standard (`fw01`, `sw01`, `pve01`, `bkp01`, …) and fill in a port/NIC assignment sheet
- [ ] **✓ verify:** every device, IP, VLAN, and cable or virtual NIC is written down. Load it all into NetBox in S5.

---

## FW-A. Firewall / Router: OPNsense
*Works on a physical appliance (Part 1) or as a VM (Part 2, after V3).*

1. Install
   - [ ] Boot the OPNsense image (USB stick, or ISO in the VM), log in as `installer`, and install to disk on ZFS
   - [ ] On the console, assign **WAN** and **LAN** (LAN = the trunk NIC). Set LAN temporarily to `192.168.10.1/24` (MGMT).
   - [ ] *(VM only)* install the `os-qemu-guest-agent` plugin. Confirm that CRC, TSO, and LRO offloads are **disabled** under *Interfaces › Settings*.
2. Interfaces (*Interfaces › Other Types › VLAN*, then *Assignments*)
   - [ ] Create VLANs 10, 40, 50, and 60 on the trunk NIC. Assign them and give each the `.1` gateway address.
   - [ ] Do **not** create VLANs 70, 71, or 80. They stay non-routed.
3. Services
   - [ ] **Unbound DNS:**
     - [ ] Enable DNSSEC
     - [ ] Add query forwarding for `example.mil` and `40.168.192.in-addr.arpa` → `192.168.40.2` and `.3`
     - [ ] Answer only on the internal VLANs
   - [ ] **NTP:** add the upstream or GPS sources and serve on the internal VLANs
   - [ ] **DHCP relay** on VLAN 50 → `192.168.40.10` (Kea on util01)
4. Firewall rules
   - [ ] Create aliases (`IPA_SERVERS`, `MAIL_VIP`, `IPA_PORTS`) and build the rules from the [[network-prereqs]] S1 matrix
   - [ ] Default: **deny between VLANs**, and log the denies. Outbound NAT on automatic.
5. VPN
   - [ ] Site-to-site **WireGuard** (*VPN › WireGuard*), or IPsec to non-WireGuard peers. Allow only what the matrix lists.
   - [ ] *(Optional)* road-warrior WireGuard into MGMT
6. Security and operations
   - [ ] **Suricata** (*Services › Intrusion Detection*): IDS mode on WAN first, then IPS once tuned
   - [ ] NetFlow → `netops01:2055`. Remote syslog → `util01:514`.
   - [ ] Two-factor auth on the GUI, GUI restricted to MGMT, SSH keys only, DoD banner
   - [ ] *(Optional HA)* a second unit with **CARP** VIPs (.1) and the real IPs on .2/.3 of each VLAN, with pfsync on a dedicated link
   - [ ] **Back up the config** (*System › Configuration › Backups*) and store it off-box
7. **✓ verify:**
   - [ ] From MGMT, `ping` upstream and `dig @192.168.10.1 example.com` both work
   - [ ] CLIENTS → MGMT is **blocked**
   - [ ] Live log view shows the expected denies

---

## FW-B. Firewall / Router: Linux (Rocky 9; Debian 13 differences noted)
*Works on a physical box (Part 1) or as a VM (Part 2, after V3). The example names the WAN NIC `wan0` and the trunk NIC `lan0`; rename yours or substitute.*

1. Install
   - [ ] Rocky 9 **Minimal** with the **DISA STIG** security profile, on a 20 GB disk. Hostname `fw01.example.mil`.
   - [ ] *Debian:* use a netinst image with only "SSH server" selected
2. VLAN interfaces and zones
   ```bash
   # Rocky (NetworkManager). Repeat for 40, 50, 60.
   nmcli con add type vlan con-name vlan10 ifname vlan10 dev lan0 id 10 \
     ipv4.method manual ipv4.addresses 192.168.10.1/24 ipv6.method disabled connection.zone mgmt
   nmcli con mod wan0 connection.zone external
   ```
   - [ ] Create the zones first: `firewall-cmd --permanent --new-zone={mgmt,servers,clients,dmz}`, then `--reload`
   - [ ] *Debian:* in `/etc/network/interfaces`, add `iface vlan40 inet static` with `vlan-raw-device lan0` and `address 192.168.40.1/24` (needs the `vlan` package)
3. Turn on routing
   ```bash
   cat > /etc/sysctl.d/90-router.conf <<'EOF'
   net.ipv4.ip_forward = 1
   net.ipv4.conf.all.rp_filter = 1
   EOF
   sysctl --system
   ```
   - [ ] **Record a STIG exception** for `ip_forward`, since this host is a router
4. Firewall policy
   - **Rocky (firewalld policy objects):**
     ```bash
     # Services the router itself offers to internal zones
     for z in mgmt servers clients; do firewall-cmd --permanent --zone=$z --add-service={dns,ntp}; done
     firewall-cmd --permanent --zone=clients --add-service=dhcp
     firewall-cmd --permanent --zone=mgmt --add-service={ssh,cockpit}
     # Inter-VLAN: one policy per allowed flow (default = drop)
     firewall-cmd --permanent --new-policy clients-servers
     firewall-cmd --permanent --policy clients-servers --add-ingress-zone clients --add-egress-zone servers
     firewall-cmd --permanent --policy clients-servers \
       --add-service={dns,kerberos,kpasswd,ldap,ldaps,https,smtp-submission,smtps,imaps,ntp}
     firewall-cmd --permanent --new-policy clients-wan
     firewall-cmd --permanent --policy clients-wan --add-ingress-zone clients --add-egress-zone external --add-service={http,https}
     firewall-cmd --permanent --new-policy mgmt-any
     firewall-cmd --permanent --policy mgmt-any --add-ingress-zone mgmt --add-egress-zone ANY --set-target ACCEPT
     # …add servers-wan, vpn-servers, etc. from the matrix
     firewall-cmd --set-log-denied=unicast
     firewall-cmd --reload
     ```
     The `external` zone masquerades (NATs) by default.
   - **Debian (`/etc/nftables.conf`):**
     ```
     table inet filter {
       chain input { type filter hook input priority 0; policy drop;
         ct state established,related accept; iif lo accept
         iifname "vlan10" tcp dport { 22, 9090 } accept
         iifname { "vlan10", "vlan40", "vlan50" } meta l4proto { tcp, udp } th dport 53 accept
         iifname { "vlan10", "vlan40", "vlan50" } udp dport { 67, 123 } accept }
       chain forward { type filter hook forward priority 0; policy drop;
         ct state established,related accept
         iifname "vlan10" accept
         iifname "vlan50" oifname "vlan40" tcp dport { 53, 88, 389, 443, 464, 465, 587, 636, 993 } accept
         iifname "vlan50" oifname "vlan40" udp dport { 53, 88, 123, 464 } accept
         iifname "vlan50" oifname "wan0" tcp dport { 80, 443 } accept
         log prefix "fwd-drop " counter drop }
     }
     table ip nat { chain postrouting { type nat hook postrouting priority 100; oifname "wan0" masquerade; } }
     ```
     Apply it with `nft -c -f /etc/nftables.conf` (check), then `systemctl enable --now nftables`.
5. DNS: Unbound
   - [ ] `dnf install unbound` (or `apt install unbound`)
   - [ ] Add a drop-in at `/etc/unbound/conf.d/lab.conf` (on Debian, `/etc/unbound/unbound.conf.d/lab.conf`):
     ```
     server:
       interface: 192.168.10.1
       interface: 192.168.40.1
       interface: 192.168.50.1
       access-control: 192.168.0.0/16 allow
       domain-insecure: "example.mil"
       domain-insecure: "40.168.192.in-addr.arpa"
       local-zone: "168.192.in-addr.arpa." nodefault
     stub-zone:
       name: "example.mil"
       stub-addr: 192.168.40.2
       stub-addr: 192.168.40.3
     stub-zone:
       name: "40.168.192.in-addr.arpa"
       stub-addr: 192.168.40.2
       stub-addr: 192.168.40.3
     forward-zone:
       name: "."
       forward-addr: <upstream-resolver>
     ```
   - [ ] `unbound-checkconf`, then `unbound-control-setup`, then `systemctl enable --now unbound`. Add a `stub-zone` for each additional IPA reverse zone.
6. NTP: chrony
   - [ ] In `/etc/chrony.conf`, set `server <upstream> iburst` and `allow 192.168.0.0/16`
   - [ ] `systemctl restart chronyd`
7. DHCP relay (VLAN 50 → Kea on util01)
   - [ ] Rocky: `dnf install dhcp-relay`, then add a systemd override with `ExecStart=/usr/sbin/dhcrelay -d --no-pid -id vlan50 -iu vlan40 192.168.40.10`
   - [ ] Debian: if `isc-dhcp-relay` is no longer available, install `dnsmasq` and use `port=0` plus `dhcp-relay=192.168.50.1,192.168.40.10`
   - [ ] Or skip the relay and run **Kea on fw01** instead
8. VPN: WireGuard
   ```ini
   # /etc/wireguard/wg0.conf   (keys: wg genkey | tee priv | wg pubkey > pub)
   [Interface]
   Address    = 172.16.0.1/30
   ListenPort = 51820
   PrivateKey = <fw01-private>
   [Peer]                     # YYMEU site
   PublicKey  = <peer-public>
   Endpoint   = <peer-wan-ip>:51820
   AllowedIPs = 172.16.0.2/32, <peer-subnets>
   PersistentKeepalive = 25
   ```
   - [ ] `systemctl enable --now wg-quick@wg0`
   - [ ] Put `wg0` in a `vpn` zone and add a `vpn-servers` policy (25 to the mail VIP, 53 to IPA). Open 51820/udp on WAN.
   - [ ] On Rocky, WireGuard is a Technology Preview. **Libreswan** (IPsec) is the fully supported alternative and the one to use for DoD or vendor peers.
9. IDS: Suricata
   - [ ] Install: `dnf install epel-release && dnf install suricata` (Debian: `apt install suricata`)
   - [ ] Set the capture interface to `wan0`: `/etc/sysconfig/suricata` on Rocky, or the `af-packet` section of `/etc/suricata/suricata.yaml`
   - [ ] Run `suricata-update`, then `systemctl enable --now suricata`
   - [ ] Alerts go to `/var/log/suricata/fast.log` and `eve.json`. Move to IPS later with an nftables `queue` rule (NFQUEUE).
10. Operations
    - [ ] rsyslog: add `*.* @@192.168.40.10:514` → util01
    - [ ] Flow export: `softflowd -i vlan50 -n 192.168.40.30:2055` (ntopng)
    - [ ] Cockpit (`systemctl enable --now cockpit.socket`), reachable on MGMT only
    - [ ] SSH keys only. DoD banner in `/etc/issue` and `/etc/issue.net` (the STIG profile sets these).
    - [ ] Track the config with `etckeeper`, or back up `/etc` nightly
    - [ ] *(Optional HA)* run a second router with **keepalived** (a VRRP VIP of `.1` per VLAN, real IPs on .2/.3) and **conntrackd** on a dedicated sync link
    - [ ] *(Optional)* **FRR** for OSPF/BGP to neighbor sites instead of static routes
11. **✓ verify:**
    - [ ] Rules: `firewall-cmd --list-all-policies` (or `nft list ruleset`), and `ip -br a`
    - [ ] DNS: `dig @192.168.40.1 example.com` and `dig @192.168.40.1 ipa01.example.mil`, both after the IPA servers exist
    - [ ] NTP: `chronyc clients`
    - [ ] VPN: `wg show` shows a recent handshake
    - [ ] From VLAN 50, `nmap` toward MGMT shows everything filtered, and the denies appear in `journalctl -k`

---

# Part 1: Hardware-Based Build

## H1. Rack, Power, and Cabling
- [ ] Rack the UPS at the bottom and the heavy servers low. Leave airflow gaps. Install the metered PDU.
- [ ] Cable:
  - [ ] fw01 WAN → upstream, and fw01 LAN → sw01 trunk
  - [ ] Each server: 2× NIC → sw01 (LACP pair), plus BMC → a VLAN 10 access port
  - [ ] **mx01 ⇄ mx02:** a direct 10G DAC for DRBD and the second corosync ring
  - [ ] bkp01, ws01, and the admin laptop → access ports
- [ ] Label both ends of every cable and record them on the port sheet
- [ ] **✓ verify:** link lights everywhere, the UPS reports it's on line power, and the PDU shows the expected load

## H2. Managed Switch (`sw01`)
Configure over the **console cable** first. The example uses Cisco IOS syntax; translate it for other vendors.
```
hostname sw01
vlan 10,40,50,60,999
interface vlan 10
 ip address 192.168.10.2 255.255.255.0
ip default-gateway 192.168.10.1
ip domain-name example.mil
crypto key generate rsa modulus 2048
ip ssh version 2
line vty 0 15
 transport input ssh
no ip http server
no ip http secure-server
spanning-tree mode rapid-pvst
ip dhcp snooping
ip dhcp snooping vlan 40,50
! Uplink to fw01 (trusted: relay and PXE replies come from here)
interface Gi1/0/1
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,40,50,60
 ip dhcp snooping trust
! One LACP port-channel per server (example: ipa01 on Gi1/0/3-4)
interface range Gi1/0/3-4
 channel-group 3 mode active
interface Port-channel3
 switchport mode access
 switchport access vlan 40
 spanning-tree portfast
! Bond members boot as single NICs during PXE, so let the channel fall back
 no port-channel standalone-disable
! The util01 port-channel is trusted for DHCP snooping (Kea lives there)
interface Port-channel5
 ip dhcp snooping trust
! BMC ports
interface range Gi1/0/17-24
 switchport mode access
 switchport access vlan 10
! Client and admin access ports
interface Gi1/0/30
 switchport mode access
 switchport access vlan 50
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
! Unused ports
interface range Gi1/0/40-48
 switchport access vlan 999
 shutdown
```
- [ ] Set NTP to `192.168.10.1`, syslog to `util01`, **SNMPv3 only**, and the **DoD banner**
- [ ] Set up a local break-glass admin. RADIUS AAA gets added in S5.
- [ ] **✓ verify:**
  - [ ] `show vlan brief` and `show etherchannel summary` look right
  - [ ] SSH from MGMT works. Telnet and HTTP are refused.

## Firewall
Do **FW-A** or **FW-B** on the physical appliance now.
- [ ] In the appliance BIOS, enable AES-NI, set power-on after AC loss, and disable unused radios and ports
- [ ] *(HA)* rack and cable the second unit, then repeat the build with CARP (FW-A) or keepalived (FW-B)

## H3. Server BIOS and BMC Prep (every server)
- [ ] Update BIOS, BMC, NIC, and RAID firmware
- [ ] Boot mode **UEFI**, **Secure Boot on**, **TPM 2.0 on**
- [ ] Power-restore policy set to "last state" or "on"
- [ ] BMC: static IP on VLAN 10 (`.21` onward), **default password changed**, unique credentials stored in the vault, HTTPS/Redfish on, IPMI-over-LAN on only for the mx pair (fencing)
- [ ] Boot order: network (PXE/HTTP) once, then disk
- [ ] **✓ verify:** each BMC answers `ipmitool -I lanplus -H <bmc> -U <user> -P <pw> chassis status` from the admin laptop

## H4. Provisioning Server (`util01` first)
- [ ] Install **util01 by USB** with the Rocky 9 DVD: STIG profile, mdadm RAID1, static `192.168.40.10`
- [ ] Set up the PXE/UEFI boot service (≈ WDS):
  ```bash
  dnf install -y tftp-server httpd shim-x64 grub2-efi-x64 kea
  mkdir -p /var/www/html/rocky9 /var/www/html/ks
  mount -o loop,ro Rocky-9-x86_64-dvd.iso /var/www/html/rocky9  # (add to fstab)
  cp /boot/efi/EFI/rocky/{shimx64.efi,grubx64.efi} /var/lib/tftpboot/
  cp /var/www/html/rocky9/images/pxeboot/{vmlinuz,initrd.img} /var/lib/tftpboot/
  firewall-cmd --permanent --add-service={tftp,http,dhcp} && firewall-cmd --reload
  systemctl enable --now tftp.socket httpd
  ```
- [ ] Create `/var/lib/tftpboot/grub.cfg` with one menu entry per role:
  ```
  menuentry 'Rocky 9 - ipa' {
    linuxefi vmlinuz inst.repo=http://192.168.40.10/rocky9 inst.ks=http://192.168.40.10/ks/ipa.ks
    initrdefi initrd.img
  }
  ```
- [ ] Kea on VLAN 40: add **MAC reservations only** (no pool) for the servers, with `next-server 192.168.40.10` and `boot-file-name "shimx64.efi"`. Client VLAN 50 gets the normal pool from [[lstyle-annex-b-brief]] §4.
- [ ] Kickstart per role (`/var/www/html/ks/<role>.ks`). The essentials:
  ```
  text
  url --url=http://192.168.40.10/rocky9/BaseOS
  repo --name=AppStream --baseurl=http://192.168.40.10/rocky9/AppStream
  lang en_US.UTF-8
  keyboard us
  timezone UTC --utc
  network --bootproto=dhcp --hostname=CHANGEME.example.mil
  rootpw --lock
  user --name=admin --groups=wheel --iscrypted --password=<sha512-hash>
  sshkey --username=admin "<ssh-ed25519 public key>"
  ignoredisk --only-use=sda,sdb
  clearpart --all --initlabel
  part raid.e1 --size=600 --ondisk=sda
  part raid.e2 --size=600 --ondisk=sdb
  part raid.b1 --size=1024 --ondisk=sda
  part raid.b2 --size=1024 --ondisk=sdb
  part raid.p1 --size=1 --grow --ondisk=sda
  part raid.p2 --size=1 --grow --ondisk=sdb
  raid /boot/efi --level=1 --device=efi --fstype=efi --metadata=1.0 raid.e1 raid.e2
  raid /boot --level=1 --device=boot --fstype=xfs raid.b1 raid.b2
  raid pv.01 --level=1 --device=pv raid.p1 raid.p2
  volgroup vg0 pv.01
  # STIG wants separate filesystems:
  logvol /              --vgname=vg0 --name=root  --size=12288 --fstype=xfs
  logvol /home          --vgname=vg0 --name=home  --size=2048  --fstype=xfs
  logvol /tmp           --vgname=vg0 --name=tmp   --size=2048  --fstype=xfs
  logvol /var           --vgname=vg0 --name=var   --size=8192  --fstype=xfs
  logvol /var/log       --vgname=vg0 --name=log   --size=4096  --fstype=xfs
  logvol /var/log/audit --vgname=vg0 --name=audit --size=2048  --fstype=xfs
  logvol /var/tmp       --vgname=vg0 --name=vartmp --size=2048 --fstype=xfs
  %packages
  @^minimal-environment
  %end
  %addon com_redhat_oscap
    content-type = scap-security-guide
    profile = xccdf_org.ssgproject.content_profile_stig
  %end
  reboot
  ```
  Check every file with `ksvalidator` (from `pykickstart`).
- [ ] **✓ verify:** a test machine PXE-boots, picks up its reservation, and installs unattended to a login prompt

## H5. Install the Servers
- [ ] PXE-install in order: **ipa01 → ipa02 → mx01 → mx02 → netops01 → ws01**. Install `bkp01` per H7.
- [ ] After each install, convert the NIC pair to an LACP bond with the static IP from the plan:
  ```bash
  nmcli con add type bond con-name bond0 ifname bond0 \
    bond.options "mode=802.3ad,miimon=100,xmit_hash_policy=layer3+4" \
    ipv4.method manual ipv4.addresses 192.168.40.2/24 ipv4.gateway 192.168.40.1 \
    ipv4.dns "192.168.40.3 127.0.0.1" ipv4.dns-search example.mil
  nmcli con add type ethernet con-name bond0-p1 ifname eno1 master bond0
  nmcli con add type ethernet con-name bond0-p2 ifname eno2 master bond0
  nmcli con down "<old DHCP con>"; nmcli con up bond0
  hostnamectl set-hostname ipa01.example.mil
  ```
- [ ] On **mx01/mx02**, give the direct link its own address: `nmcli con add type ethernet ifname ens3f0 con-name repl ipv4.method manual ipv4.addresses 192.168.71.21/24 802-3-ethernet.mtu 9000` (use `.22` on mx02)
- [ ] Remove each server's PXE reservation from Kea once it's built
- [ ] **✓ verify:**
  - [ ] `cat /proc/mdstat` shows both mirrors `[UU]`
  - [ ] `cat /proc/net/bonding/bond0` shows 2 slaves up with an LACP partner
  - [ ] `oscap` STIG scan run
  - [ ] Ping across the bond still works after unplugging one cable

## H6. Fencing Prep (mx pair)
- [ ] On each mx BMC, create a dedicated **fencing user** with operator or power-control rights only
- [ ] Add a firewall rule allowing **mx01/mx02 → their BMC IPs on 623/udp** (IPMI) or 443 (Redfish)
- [ ] From each mx node: `fence_ipmilan -a <peer-bmc> -l fence -p <pw> --lanplus -o status` returns **Status: ON**
- [ ] For the cluster build in [[lstyle-annex-b-brief]] §5:
  ```bash
  pcs stonith create fence-mx01 fence_ipmilan ip=<mx01-bmc> username=fence password=<pw> lanplus=1 pcmk_host_list=mx01
  pcs stonith create fence-mx02 fence_ipmilan ip=<mx02-bmc> username=fence password=<pw> lanplus=1 pcmk_host_list=mx02
  ```
- [ ] **✓ verify:** the status check works both ways. Run the real fence test (`pcs stonith fence mx02`) after the cluster exists.

## H7. Backup Server and UPS
- [ ] `bkp01`: install **TrueNAS Community** with a RAIDZ2 pool. Create NFS shares for ReaR (`/mnt/tank/rear`) and an SFTP or `rest-server` target for restic. Give it a MGMT IP (`.30`).
- [ ] *(Alternative)* Rocky 9 + **Bareos** director and storage daemon, for scheduled jobs and a catalog
- [ ] On every server:
  - [ ] Set `/etc/rear/local.conf` to `OUTPUT=ISO`, `BACKUP=NETFS`, `BACKUP_URL=nfs://192.168.10.30/mnt/tank/rear`
  - [ ] Run `rear -v mkbackup` on a weekly systemd timer
  - [ ] Back up data and the mail store nightly with restic
- [ ] UPS:
  - [ ] On `util01`, run `dnf install nut` (EPEL). In `/etc/ups/ups.conf`, use the `usbhid-ups` or `snmp-ups` driver. Set `upsd` to listen on VLAN 40.
  - [ ] Every other server runs `upsmon` as a **secondary** pointing at `ups@util01`
  - [ ] Shutdown order: ws01 → mx → netops/util → **ipa last**. Boot order is the reverse, using BMC power-on delays.
- [ ] **✓ verify:**
  - [ ] `upsc ups@util01` shows status `OL`
  - [ ] One ReaR ISO **boots and restores** onto a spare disk or machine

Then continue to the shared sections **S5 → S6 → S7**.

---

# Part 2: VM-Based Build

## V1. Hypervisor Install (Proxmox VE 9)
1. BIOS and BMC
   - [ ] Enable **VT-x/AMD-V and VT-d/IOMMU**. Boot mode UEFI.
   - [ ] Give the BMC a static MGMT IP and change its default password
   - [ ] Update the firmware
2. Install
   - [ ] Boot the Proxmox ISO and choose **ZFS (RAID1)** on the two boot SSDs
   - [ ] Set the management IP `192.168.10.11/24` and FQDN `pve01.mgmt.example.mil`
   - [ ] Leave the gateway at `192.168.10.1`. It won't work until the firewall VM is up; that's expected.
3. Post-install
   - [ ] In *Updates › Repositories*, disable `enterprise` and add `pve-no-subscription` (or apply a subscription key). Then run `apt update && apt full-upgrade`.
   - [ ] Create named admin users with **TOTP or WebAuthn**, and stop using `root@pam` day to day
   - [ ] Enable the datacenter firewall, allowing 8006 and 22 from MGMT only
   - [ ] SSH: keys only, banner set
   - [ ] Cap the ZFS ARC if RAM is tight: `options zfs zfs_arc_max=<bytes>` in `/etc/modprobe.d/zfs.conf`
   - [ ] For a **laptop lab** (Fedora/Rocky Workstation): `dnf install @virtualization`, then build the same bridges with `nmcli` and use virt-manager in place of `qm`
4. **✓ verify:** the web UI at `https://192.168.10.11:8006` is reachable from the admin laptop on its direct or VLAN 10 link

## V2. Virtual Networking
**Single host, no physical switch** (`/etc/network/interfaces`):
```
auto vmbr0
iface vmbr0 inet manual            # WAN bridge: physical uplink, NO host IP
    bridge-ports enp1s0
    bridge-stp off
    bridge-fd 0

auto enp2s0
iface enp2s0 inet manual           # admin laptop plugs in here, untagged = VLAN 10
    bridge-access 10

auto vmbr1
iface vmbr1 inet manual            # internal VLAN-aware "switch"
    bridge-ports enp2s0
    bridge-stp off
    bridge-fd 0
    bridge-vlan-aware yes
    bridge-vids 10 40 50 60 71

auto vmbr1.10
iface vmbr1.10 inet static         # host management
    address 192.168.10.11/24
    gateway 192.168.10.1
```
**Cluster (hosts behind a physical switch):** make `bond0` an LACP bond (`bond-mode 802.3ad`, `bond-xmit-hash-policy layer3+4`) and set `vmbr1` `bridge-ports bond0` with `bridge-vids 10 40 50 60 70 71 80`. Add `vmbr1.70` (corosync) and `vmbr1.80` (storage, `mtu 9000`). Configure the switch as in **H2**, but with **trunk** port-channels to each host.
- [ ] Apply with `ifreload -a`, keeping a console or BMC session open while you do
- [ ] *(Optional)* define the VLANs once in **Datacenter › SDN** (VLAN zone + VNets) so every node gets them
- [ ] **✓ verify:** `bridge vlan show` lists the VIDs on `vmbr1`. On a cluster, `/proc/net/bonding/bond0` shows both slaves up.

## V3. Firewall VM
```bash
qm create 100 --name fw01 --machine q35 --cpu host --cores 2 --memory 4096 \
  --ostype other \
  --scsihw virtio-scsi-single --scsi0 local-zfs:32,iothread=1 \
  --net0 virtio,bridge=vmbr0 \
  --net1 virtio,bridge=vmbr1,trunks="10;40;50;60" \
  --ide2 local:iso/<opnsense-or-rocky>.iso,media=cdrom \
  --onboot 1 --startup order=1,up=30
```
- [ ] For FW-B, use `--ostype l26` and 2 GB of RAM
- [ ] `net1` has **no tag**, so it acts as a trunk (limited to the listed VLANs)
- [ ] Leave Proxmox's per-NIC firewall **off** on fw01
- [ ] *(Stronger isolation)* pass the physical WAN NIC through instead of using `vmbr0`: `qm set 100 --hostpci0 0000:03:00.0`
- [ ] Start the VM, open its console, and complete **FW-A** or **FW-B**
- [ ] **✓ verify:** the host can reach the internet through fw01 (`apt update` works), and the admin laptop reaches `192.168.10.1`

## V4. Cluster and Storage (multi-host only)
- [ ] Create the cluster: `pvecm create labclu --link0 192.168.70.11`. Join each other node with `pvecm add 192.168.70.11 --link0 192.168.70.1X`.
- [ ] *(2 nodes)* add a third vote: `pvecm qdevice setup <qnetd-host>`
- [ ] Storage, choose one:
  - [ ] **Ceph** on VLAN 80 (3 monitors, OSDs on the data NVMe)
  - [ ] **ZFS replication** jobs every 15 minutes
- [ ] HA for fw01 and the other critical VMs: `ha-manager add vm:100`. Add a **PVE 9 HA rule** (resource affinity, keep apart) for ipa01/ipa02 and mx01/mx02.
- [ ] **✓ verify:**
  - [ ] `pvecm status` shows **Quorate: Yes**
  - [ ] Jumbo frames work end to end: `ping -M do -s 8972 <peer-80-ip>`
  - [ ] Live-migrating a test VM works

## V5. Backup Server (PBS)
- [ ] Install **Proxmox Backup Server**, ideally on its **own box**. If you have only one host, run it as a VM with a dedicated disk and sync it off-host.
- [ ] Create a ZFS datastore. Add it in PVE (*Datacenter › Storage › Add › PBS*) with **client-side encryption**, and store the key offline.
- [ ] Schedule:
  - [ ] Nightly backups of all VMs (snapshot mode)
  - [ ] Retention: keep-daily 7, keep-weekly 4, keep-monthly 6
  - [ ] Weekly verify and garbage collection
- [ ] **✓ verify:** restore one backup **as a new VM ID** and boot it

## V6. VM Template and Clones
1. Build a Rocky 9 template (on `pve01`):
   ```bash
   qm create 9000 --name rocky9-tmpl --ostype l26 --machine q35 --bios ovmf \
     --efidisk0 local-zfs:1,efitype=4m,pre-enrolled-keys=1 \
     --cpu x86-64-v2-AES --cores 2 --memory 4096 --balloon 0 \
     --scsihw virtio-scsi-single --net0 virtio,bridge=vmbr1,tag=40 \
     --agent enabled=1 --serial0 socket --vga serial0
   qm disk import 9000 Rocky-9-GenericCloud-Base.latest.x86_64.qcow2 local-zfs
   qm config 9000 | grep unused   # note the imported disk name
   qm set 9000 --scsi0 local-zfs:vm-9000-disk-1,discard=on,iothread=1,ssd=1 \
     --ide2 local-zfs:cloudinit --boot order=scsi0
   qm resize 9000 scsi0 30G
   qm template 9000
   ```
2. Clone each VM (example: `ipa01`):
   ```bash
   qm clone 9000 101 --name ipa01 --full
   qm set 101 --ipconfig0 ip=192.168.40.2/24,gw=192.168.40.1 \
     --nameserver 192.168.40.1 --searchdomain example.mil --sshkeys ~/.ssh/admin.pub \
     --onboot 1 --startup order=2,up=60
   qm start 101
   ```
   - [ ] mx01/mx02 also get a data disk and a second NIC: `qm set 104 --scsi1 local-zfs:50,iothread=1 --net1 virtio,bridge=vmbr1,tag=71`
   - [ ] ws01: tag 50, a desktop image, and a SPICE display
3. On first boot of every guest:
   - [ ] `dnf -y upgrade`
   - [ ] `dnf -y install qemu-guest-agent`
   - [ ] Apply the STIG with `oscap … --remediate`, or with the SSG Ansible playbook ([[lstyle-annexb-prereq]] §11)
4. Fencing for the mx pair (≈ H6)
   - [ ] Create a PVE user `fence@pve` with the **VM.PowerMgmt** role on VMs 104 and 105
   - [ ] Allow mx → `192.168.10.11:8006` through the firewall
   - [ ] Later: `pcs stonith create fence-pve fence_pve ip=192.168.10.11 username=fence@pve password=<pw> pcmk_host_map="mx01:104;mx02:105"`

**VM best practices**

| Setting | Do | Why |
|---|---|---|
| Disk / NIC | **VirtIO SCSI single** with `iothread` and `discard`. **VirtIO** NIC. | Best performance. TRIM frees space on ZFS/Ceph. |
| Firmware | **q35 + UEFI (OVMF)**, Secure Boot keys enrolled | Modern platform. Secure Boot is a STIG item. |
| CPU type | `x86-64-v2-AES` (safe for live migration across mixed CPUs). Use **`x86-64-v3` for Rocky 10**, or `host` if all nodes are identical. | Rocky 10 will not boot on v2 |
| Memory | **Ballooning off** on IPA, mail, and database-like VMs. Don't overcommit RAM on those. | Prevents directory and database stalls |
| Guest agent | `qemu-guest-agent` installed and enabled | Clean shutdowns, consistent backups, IPs shown in the GUI |
| Unique identity | Use cloud-init clones only. If you copy a disk by hand, regenerate `/etc/machine-id` and the SSH host keys. | Duplicate IDs break DHCP, Kerberos, and monitoring |
| NICs and VLANs | Tag at the **VM NIC**. The firewall VM's NIC is an untagged trunk. Put replication on **VLAN 71**. | Keeps replication off the user VLANs |
| Placement | ipa01/02 and mx01/02 on **different hosts**, enforced by an HA rule | One host failure never takes down both copies |
| Boot order | fw01 (1) → ipa (2) → util01 (3) → mx (4) → netops01 (5) → clients (6), with `up=` delays | Services come up after their dependencies |
| Snapshots | Short-lived, before a change only. **Never revert an IPA server.** Re-initialize it from its replica instead. | Snapshots aren't backups. Reverting causes replication damage (like a USN rollback in AD). |
| Time | chrony in the guest pointed at the IPA servers or fw01 | Kerberos needs clocks within 5 minutes |
| Hardening | Remove unused virtual hardware. Enable the PVE firewall per VM, except fw01. | Smaller attack surface |
| Naming / tags | VM name = hostname. Add PVE tags (`ipa`, `mail`, `prod`). | Easier to search and to target with Ansible |

- [ ] **✓ verify:** every VM boots in order, `qm agent <id> ping` succeeds, and all VMs reach their gateway

## V7. UPS
- [ ] On `pve01`, run `apt install nut`. Configure the UPS driver and `upsd`, and run `upsmon` as the **primary**.
- [ ] Other hosts, PBS, and optionally the guests run as **secondaries**
- [ ] Set the guests to shut down cleanly through the guest agent before the host shuts down (`onboot` and `startup` orders handle the restart)
- [ ] **✓ verify:** `upsc ups@localhost` shows `OL`

Then continue to the shared sections **S5 → S6 → S7**.

---

## S5. Network Operations Services (shared)
On **`util01`** (alongside Kea and Zabbix):
- [ ] **FreeRADIUS:**
  - [ ] `dnf install freeradius freeradius-ldap freeradius-utils`
  - [ ] Point the LDAP module at FreeIPA, and add a client entry for `sw01` (and fw01 if it's FW-B)
  - [ ] Point the switch's AAA at it, keeping the local fallback account
  - [ ] Test with `radtest`, then SSH into the switch as an IPA user
- [ ] **rsyslog** central receiver on 514/tcp, with per-host files. Every device and server sends its logs here.

On **`netops01`** (Rocky 9, VLAN 40), run these as Podman containers or install them natively:
- [ ] **NetBox:** load the S0 data (VLANs, prefixes, IPs, devices, cables, VMs). This is the source of truth.
- [ ] **Oxidized:** pull configs nightly into Git. Use `ios` for sw01, `opnsense` for FW-A, and `linuxgeneric` for FW-B.
- [ ] **LibreNMS:** SNMPv3 discovery of the switch, firewall, hypervisors or servers, BMCs (Part 1), and the UPS
- [ ] **ntopng:** NetFlow from fw01
- [ ] **Apache Guacamole:** browser-based SSH and console access to management, exposed on MGMT only with 2FA
- [ ] **✓ verify:**
  - [ ] LibreNMS shows all devices green with traffic graphs
  - [ ] Oxidized has today's config committed
  - [ ] RADIUS login to the switch works, and the fallback works when RADIUS is stopped

## S6. Hardening Sweep (shared)
- [ ] Management planes (switch, firewall UI/SSH, BMCs, PVE/PBS, Cockpit) are reachable **only from VLAN 10**
- [ ] Every device has: the DoD banner, SSH keys or RADIUS, no default credentials, NTP synced, remote syslog, and SNMPv3
- [ ] Firewall:
  - [ ] FW-A: apply the **Firewall SRG / Network Device Management SRG** by hand
  - [ ] FW-B (Rocky): run the OpenSCAP STIG scan and document the router exceptions
  - [ ] Record all deviations in the POA&M
- [ ] Switch: apply the **vendor STIG**
- [ ] Rocky hosts and guests pass the OpenSCAP STIG scan. PVE hosts are hardened to CIS Debian guidance with documented exceptions.
- [ ] Vulnerability scan: **Greenbone/OpenVAS** in the lab, or ACAS on a DoD network
- [ ] Configs (fw, switch, `/etc/pve`, Kickstart files) are backed up off-site

## S7. Sign-off Tests (shared; `[HW]` = Part 1 only, `[VM]` = Part 2 only)
- [ ] **Reachability:** the port matrix from CLIENTS, SERVERS, and MGMT matches the policy. `nmap` from VLAN 50 proves the denies.
- [ ] **DNS chain:** `ws01` → IPA → fw01 Unbound → upstream resolves both internal and external names, including reverse lookups
- [ ] **NTP chain:** `chronyc tracking` shows the expected stratum with an offset under 100 ms
- [ ] **DHCP:** `ws01` gets a lease through the relay, and DHCP snooping blocks a rogue server
- [ ] **Link failure:** pull one cable from a bond, and service continues
- [ ] `[HW]` **Disk failure:** pull one RAID1 member, and the server keeps running. Rebuild with `mdadm --add`.
- [ ] `[HW]` **Server failure:** power off mx01 through its BMC. Pacemaker fences it and mail moves to mx02.
- [ ] `[VM]` **Host failure:** power off a PVE node. HA restarts its VMs elsewhere, and the cluster stays quorate.
- [ ] **Firewall HA** *(if built)*: fail the primary. Traffic and VPN move to the secondary.
- [ ] **Power:** simulate a power loss. NUT shuts everything down in order, and it comes back up in order.
- [ ] **Restore:** `[HW]` a ReaR bare-metal restore, or `[VM]` a PBS VM restore, plus one config restored from Oxidized
- [ ] **Monitoring:** LibreNMS and Zabbix all green, and a test alert is received

When all of this passes, continue to [[lstyle-annex-b-brief]] §1 to build the servers on this network.

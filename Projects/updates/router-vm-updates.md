# Updates: Router-VM Concept Applied to the Annex B Docs

**Source of the change:** [[vm-router-lab-guide]], which puts the lab behind a **single router VM** on isolated hypervisor networks. That router is the only path out (host → internet), so it doubles as a kill switch and a containment boundary.
**Docs compared:**
- [[lstyle-annex-b-brief]]
- [[annex-b-brief]]
- [[lstyle-annexb-prereq]]
- [[network-prereqs]]
- [[network-guide]]

> [!note] How to use this doc
> The original docs are **unchanged**. This is the **change log**. The merged copies are [[lstyle-annex-b-brief-v2]], [[lstyle-annexb-prereq-v2]], [[network-prereqs-v2]], and [[network-guide-v2]], all in this folder. Section §6 (fixes to [[vm-router-lab-guide]] itself) and §5 (Windows [[annex-b-brief]]) weren't merged anywhere, because those docs weren't in scope for v2 copies.

---

## 0. Verdict: Does It Change Anything?

**Yes, but it extends the design more than it replaces it.** The existing docs already have a router choke point: `fw01` routes every VLAN, and the Proxmox single-host design puts the WAN on `vmbr0` with internal VLANs on `vmbr1`. What the router-VM concept adds:

| # | Gap it exposes | Impact | Where it lands |
|---|---|---|---|
| 1 | No **desktop / Type-2 hypervisor** path. Part 2 assumes Proxmox on dedicated hardware, and the "laptop lab" tier is only one row. | **High.** This is the most likely way a student actually builds the lab. | New **Part 2-T** in both network docs |
| 2 | **Containment is not enforced.** `mgmt-any` (egress `ANY`) and `clients-wan 80/443` NAT straight into the host's real LAN. | **High.** Lab traffic can reach your home or unit LAN. | Firewall policy (S1), FW-A, FW-B |
| 3 | No **kill switch / offline mode**, and no test proving the lab runs with the WAN cut | Medium | S1, prereq §8, S7 sign-off |
| 4 | **Double NAT** is not covered. It breaks inter-site mail and VPNs between student labs. | Medium. "Mail a neighbor site" is a sign-off item. | New **range transit network** design |
| 5 | **Rogue DHCP** from the hypervisor's own NAT or host-only DHCP is not covered. It would fight Kea. | Medium | Part 2-T, S3 |
| 6 | **Fencing on Type-2 hypervisors.** `fence_pve` doesn't exist there, and host-reaching agents break the isolation. | Medium. A 2-node cluster with no fencing is unsafe. | prereq §1/§6, Part 2-T |
| 7 | **Host access to the lab.** Internal networks cut the host off, but the docs assume an "admin laptop" on MGMT. | Low–Medium | Part 2-T |
| 8 | **NAT subnet collisions.** The hypervisor NAT range could overlap the 192.168.x plan. | Low, but painful to debug | S1 |
| 9 | Layer-by-layer **verification method** (gateway → WAN → DNS → path) and `tcpdump` on both sides of the router | Low. Teaching value. | FW-A/FW-B verify, S7 |

The VLAN plan, IPs, server stack, and every Annex B end goal stay the same. Nothing is removed.

---

## 1. Changes to [[network-prereqs]]

### 1.1 CHANGE: Intro "Hardware or VMs?" table → add a third column
| | Part 1: Hardware | Part 2: VMs (Proxmox) | **Part 2-T: Desktop router-VM lab** |
|---|---|---|---|
| Cost and footprint | High | Low: 1–3 hosts | **Lowest:** one existing laptop or desktop |
| Isolation | Physical | VLAN bridges | **Isolated hypervisor networks behind one router VM**, with a kill switch |
| Skills taught | Cabling, RAID, BMC, PXE | Clustering, SDN, templates | Routing, NAT, DHCP, DNS, and firewall basics, watched packet by packet |
| Best for | Fixed sites | Shared ranges | **Individual student practice**, and as a containment sandbox |

### 1.2 ADD to S1, "Firewall Policy": containment and kill-switch rows
| From → To | Allow |
|---|---|
| **Any lab zone → RFC 1918 / CGNAT space on the WAN side** (`10/8`, `172.16/12`, `192.168/16`, `100.64/10`), except the WAN gateway itself | **deny and log.** This keeps lab traffic out of the host's real LAN. Put it **above** `mgmt-any` and `clients-wan`. |
| **WAN → any** | **deny.** No port forwards, except a DMZ or inter-site VPN if you build one. |
| **Kill switch** | Taking the router's WAN link down must leave every *internal* service working. That needs the util01 repo mirror and internal DNS/NTP (see 1.4). |

- **CHANGE `mgmt-any`:** narrow its egress from `ANY` to `{mgmt, servers, clients, dmz}`, plus WAN **only** for 80/443/53/123. As written it can reach the host's real LAN.

### 1.3 ADD to S1, "VLAN and IP Plan": a NAT-collision note
> [!warning] Check the WAN-side subnet
> The router VM's WAN address comes from the hypervisor's NAT network or the real LAN, and it **must not overlap** 192.168.10/40/50/60/71:
> - VirtualBox NAT: `10.0.2.0/24`
> - libvirt `default`: `192.168.122.0/24`
> - VMware VMnet8: a random `192.168.x.0/24`, so **check it** in the Virtual Network Editor
> - Home LANs are usually `192.168.0/1.x`
>
> If anything overlaps, change the hypervisor NAT range, not the lab plan.

### 1.4 CHANGE S1, "DNS and NTP Chains"
- **DNS:** add that, with the WAN cut, internal names must still resolve. IPA is authoritative for `example.mil`, so only external lookups fail. That is the expected result.
- **NTP:** with the WAN cut, fw01 loses its upstream source. Add `local stratum 10 orphan` to fw01's chrony (OPNsense: *Services › NTP*, "orphan mode") so the lab keeps **consistent** time. Kerberos needs the clocks to agree with each other, not to be correct.

### 1.5 CHANGE S3, "Network Services": DHCP row
- Add: **turn off every hypervisor-provided DHCP** on internal and host-only networks. Kea on util01 (through the fw01 relay) must be the only DHCP server on CLIENTS.
- Add a **bootstrap note:** servers are static, so Kea isn't needed until `ws01`. If you need DHCP earlier, turn on the router's DHCP for CLIENTS temporarily, then switch to the relay. The router-VM guide's `dnsmasq` setup is this temporary mode.

### 1.6 ADD to S2-A and S2-B gotchas
- [ ] **CARP / keepalived VIPs** need the hypervisor to accept frames from virtual MACs (`00:00:5e:00:01:xx`):
  - Hyper-V: *MAC address spoofing*
  - ESXi: *Forged transmits / MAC changes*, or *MAC Learning*
  - VirtualBox: *Promiscuous Mode: Allow All* on fw NICs

  Pacemaker's `IPaddr2` mail VIP moves only the IP, not the MAC, so it doesn't need this.
- [ ] **firewalld or Docker on a Linux router** can rewrite nftables. On FW-B, don't install Docker or Podman, and on Debian don't run firewalld next to `nftables.service`.
- [ ] **Bridged WAN over Wi-Fi** usually fails because the access point drops unknown MACs. Use **NAT** for the WAN on laptops.

### 1.7 ADD a new part, **Part 2-T: Desktop (Type-2) Router-VM Lab**. Insert it after Part 2 §2.7.

#### 2T.1 Topology
```
                      ┌────────────── laptop / desktop (host OS) ──────────────┐
 Internet ── host NIC ┤ hypervisor NAT ──[nic1 WAN]── fw01 (FW-A or FW-B)      │
                      │                                │ nic2  nic3  nic4 (nic5)│
                      │   isolated nets:  lab-mgmt ────┘   │     │     │        │
                      │                   lab-servers ─────┘     │     │        │
                      │                   lab-clients ───────────┘     │        │
                      │                   lab-dmz (opt) ───────────────┘        │
                      │                   lab-repl (mx01↔mx02, NOT on fw01)     │
                      └─────────────────────────────────────────────────────────┘
```
- **One isolated network per routed segment.** This replaces VLAN tags. It's simpler and works on every hypervisor. Desktop hypervisors handle 802.1Q trunks into VMs unevenly.
- Same subnets as S1. fw01 is `.1` on each network. **VLANs 70, 80, and 999 aren't used.** VLAN 71 becomes `lab-repl`, which only mx01 and mx02 attach to.
- Every server VM has **one NIC** on its segment, except mx01/02, which add `lab-repl`. They can't leave the lab except through fw01.

#### 2T.2 Host OS and hypervisor
| Option | License | Pick when | Notes |
|---|---|---|---|
| **KVM/libvirt + virt-manager** on Rocky/Fedora/Debian Workstation (**recommended**) | FOSS (GPL/LGPL) | Linux host | Truly isolated networks with no host IP (2T.3). It has `fence_virsh`/`fence_xvm` and a virtual watchdog (`i6300esb`). It's the same KVM as Proxmox, so the skills carry over. |
| **VirtualBox 7.x** | Base is **GPLv3**. The Extension Pack is proprietary (PUEL) and **not needed**. | Windows or macOS (Intel) hosts | The GUI shows only 4 NICs. Add **nic5–8 with `VBoxManage`**. It has no virtual watchdog, so use `softdog`. |
| VMware Workstation Pro | Proprietary, but free of charge since 2024 | It's already your standard | LAN Segments = internal networks. Check the VMnet8 subnet (1.3). |
| Hyper-V (Windows Pro/Ent) | Proprietary, included with Windows | Windows hosts, and matches the Windows [[annex-b-brief]] lab | Use a **Private** switch for the lab segments and **Default Switch** or External for the WAN. Watch the MAC-spoofing setting (1.6). |
| GNOME Boxes | FOSS | — | **Avoid.** It can't build isolated multi-NIC networks. |

#### 2T.3 Host sizing
The host OS needs its own **4–8 GB of RAM** on top of the VMs.

| Tier | RAM | Run |
|---|---|---|
| **Minimum** | 32 GB | FW-B fw01 (1 GB), ipa01/02 (3 GB each), mx01/02 (2–3 GB each), util01 (3 GB), ws01 (3 GB). Skip netops01 and pbs01. Run one IPA server while building mail if you need to. |
| **Comfortable** | 64 GB | The full 2.6 VM sizing |

- CPU: 8+ cores with VT-x/AMD-V, at x86-64-v3 or newer
- Disk: 1 TB NVMe. Thin-provisioned qcow2/VDI.
- Ethernet if you ever bridge the WAN

#### 2T.4 Host access (the host can't see isolated networks)
Pick one:
- **A (recommended, purest):** administer from **`ws01`** or a small admin VM inside the lab. Use the hypervisor console for everything before that exists.
- **B:** make **`lab-mgmt` a host-only network** with the hypervisor's DHCP **off**. Give the host `192.168.10.50` (the "admin laptop" in S1). This opens a host↔MGMT path, so record it.
- **Don't** add host-only NICs to servers or clients. That bypasses the router.

#### 2T.5 Part 2-T prerequisite checklist
- [ ] Host meets 2T.3, and virtualization is enabled in the firmware
- [ ] Hypervisor chosen (2T.2). Its NAT subnet has been checked for overlap (1.3).
- [ ] Isolated networks created with **no DHCP**: `lab-mgmt`, `lab-servers`, `lab-clients`, (`lab-dmz`), `lab-repl`
- [ ] Host-access option chosen (2T.4)
- [ ] WAN mode chosen: **NAT** (default, works on Wi-Fi) or **Bridged** (wired only, and needed for the range transit design in §2.3)
- [ ] Fencing method chosen (§3.2)
- [ ] ISOs downloaded: OPNsense *or* Rocky/Debian for fw01, and the Rocky 9 DVD or GenericCloud image

---

## 2. Changes to [[network-guide]]

### 2.1 CHANGE "Pick Your Path": add a third column
| Step | **Part 2-T: Desktop router-VM** |
|---|---|
| 1 | S0 Plan (+ the [[network-prereqs]] 2T checks) |
| 2 | T1 Hypervisor and isolated networks |
| 3 | T2 Router VM, then **FW-A *or* FW-B** (with the T3 deltas) |
| 4 | T4 Server/client VMs (reuse **V6** template practices) |
| 5 | T5 Fencing (SBD or a hypervisor agent) |
| 6 | S5 *(optional, if RAM allows)* → S6 → S7, plus the new T6 tests |

### 2.2 ADD to FW-A (OPNsense) and FW-B (Linux): the containment rule
- **OPNsense:**
  - [ ] *Firewalls › Aliases:* create `RFC1918_ALL` (`10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16, 100.64.0.0/10`)
  - [ ] *Firewall › Rules › Floating* (quick, direction out, interface WAN): **block** destination `RFC1918_ALL`, logged. If the WAN gateway is RFC 1918, add a pass rule above it for that gateway's IP (DNS/NTP only, if it's used upstream).
  - [ ] *Interfaces › WAN:* leave **Block private networks** and **Block bogon networks** on. These block *inbound* traffic, so the floating rule is still needed for outbound.
- **FW-B, Rocky (firewalld):**
  ```bash
  firewall-cmd --permanent --new-ipset=rfc1918 --type=hash:net
  for n in 10.0.0.0/8 172.16.0.0/12 192.168.0.0/16 100.64.0.0/10; do
    firewall-cmd --permanent --ipset=rfc1918 --add-entry=$n; done
  # Priority must be lower (evaluated earlier) than clients-wan / mgmt-any
  firewall-cmd --permanent --new-policy lab-contain
  firewall-cmd --permanent --policy lab-contain --set-priority -100
  firewall-cmd --permanent --policy lab-contain --add-ingress-zone={mgmt,servers,clients,dmz} --add-egress-zone external
  firewall-cmd --permanent --policy lab-contain --add-rich-rule='rule family=ipv4 destination ipset=rfc1918 log prefix="contain " drop'
  firewall-cmd --reload
  ```
  - [ ] Also narrow `mgmt-any` to `--add-egress-zone={servers,clients,dmz}`, and add a separate `mgmt-wan` policy for 53/80/123/443
- **FW-B, Debian (nftables):** put this as the **first** rule in `chain forward`:
  ```
  oifname "wan0" ip daddr { 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16, 100.64.0.0/10 } log prefix "contain " drop
  ```
  - [ ] Change `iifname "vlan10" accept` to `iifname "vlan10" oifname != "wan0" accept`, and add a scoped rule for vlan10 → wan0 on 53/80/123/443
- **Both:** chrony on fw01 → add `local stratum 10 orphan` (kill-switch time). OPNsense: set the NTP *orphan* stratum.
- [ ] **✓ verify:** from `ws01`, `curl -m3 http://<host's real LAN gateway>` **fails**, and the log shows `contain`. `curl https://example.com` succeeds.

### 2.3 ADD to FW-A and FW-B "verify": layer-by-layer checks
From `ws01`, in order:

| Step | Command | If it fails |
|---|---|---|
| 1. Own config | `ip a`, `ip r` | Wrong IP, mask, or gateway, or no lease from Kea |
| 2. Gateway | `ping 192.168.50.1` | Wrong network or segment attachment, or fw01's LAN IP |
| 3. Cross-segment | `ping 192.168.40.2` | Inter-zone policy, or rp_filter |
| 4. WAN | `ping 9.9.9.9` | Forwarding or NAT on fw01 |
| 5. DNS | `dig example.com`, then `dig ipa01.example.mil` | The IPA forwarder → fw01 Unbound chain |
| 6. Path | `tracepath 9.9.9.9` | The first hop must be `192.168.50.1` |

On fw01: `tcpdump -ni <lan-if>` shows lab source IPs, and `tcpdump -ni <wan-if>` shows them translated to fw01's WAN IP.

### 2.4 ADD a new part, **Part 2-T: Desktop Router-VM Build**. Insert it after V7.

#### T1. Hypervisor and isolated networks
**libvirt (recommended):**
- [ ] Install:
  - Rocky/Fedora: `dnf install @virtualization virt-manager`
  - Debian: `apt install virt-manager libvirt-daemon-system qemu-system-x86`
- [ ] Add your user to the `libvirt` group
- [ ] Create each segment as a network with **no `<ip>` and no `<forward>`**. That means no host address, no DHCP, and no routing, so it's a pure L2 segment:
  ```bash
  for n in mgmt:10 servers:40 clients:50 dmz:60 repl:71; do
    name=${n%%:*}; id=${n##*:}
    cat > /tmp/lab-$name.xml <<EOF
  <network><name>lab-$name</name><bridge name='virbr$id' stp='on' delay='0'/></network>
  EOF
    virsh net-define /tmp/lab-$name.xml && virsh net-autostart lab-$name && virsh net-start lab-$name
  done
  ```
- [ ] WAN: keep libvirt's `default` NAT network (192.168.122.0/24), or create a host bridge `br0` on wired Ethernet for a bridged WAN
- [ ] For **2T.4 option B only**, give `lab-mgmt` a host address: add `<ip address='192.168.10.50' prefix='24'/>` with **no `<dhcp>`** block

**VirtualBox:** internal networks are created when a VM first uses them, so there's nothing to define. The commands are in T2.

**Hyper-V:**
- [ ] `New-VMSwitch -Name lab-<seg> -SwitchType Private` for each segment
- [ ] WAN = `Default Switch` or an External switch

**✓ verify:**
- `virsh net-list --all` shows every `lab-*` network active with autostart on
- `ip -br a show virbr40` shows **no IPv4** (except `virbr10` under option B)

#### T2. Router VM (`fw01`)
**libvirt:**
```bash
virt-install --name fw01 --memory 2048 --vcpus 2 --cpu host-passthrough \
  --disk size=20,bus=virtio --cdrom <opnsense-or-rocky>.iso --os-variant <freebsd14.0|rocky9> \
  --network network=default,model=virtio \
  --network network=lab-mgmt,model=virtio \
  --network network=lab-servers,model=virtio \
  --network network=lab-clients,model=virtio \
  --network network=lab-dmz,model=virtio \
  --autostart
```

**VirtualBox:** use `VBoxManage` for NICs beyond 4.
```bash
VBoxManage createvm --name fw01 --ostype FreeBSD_64 --register   # RedHat9_64 for FW-B
VBoxManage modifyvm fw01 --memory 2048 --cpus 2 \
  --nic1 nat \
  --nic2 intnet --intnet2 lab-mgmt    --nictype2 virtio --nicpromisc2 allow-all \
  --nic3 intnet --intnet3 lab-servers --nictype3 virtio --nicpromisc3 allow-all \
  --nic4 intnet --intnet4 lab-clients --nictype4 virtio --nicpromisc4 allow-all \
  --nic5 intnet --intnet5 lab-dmz     --nictype5 virtio --nicpromisc5 allow-all
```
- [ ] `allow-all` is only needed if you'll use CARP. Otherwise use `deny`.

**Then (all hypervisors):**
- [ ] Write down each NIC's MAC → segment mapping **before** installing. NIC order inside the guest doesn't always match the hypervisor's slot order.
- [ ] Start fw01 **first** (libvirt `--autostart`; VirtualBox autostart service, or start it by hand)
- [ ] Complete FW-A or FW-B with the T3 deltas

#### T3. FW-A / FW-B deltas for Part 2-T (no VLAN tags)
- **OPNsense:**
  - [ ] Assign interfaces: `vtnet0`/`em0` = WAN, then one *physical* interface per segment (MGMT, SERVERS, CLIENTS, DMZ). Skip *Interfaces › Other Types › VLAN* entirely.
  - [ ] Disable hardware offloads, as in FW-A
- **FW-B (Rocky):** use plain Ethernet connections instead of `type vlan`:
  ```bash
  nmcli con add type ethernet con-name servers ifname enp3s0 \
    ipv4.method manual ipv4.addresses 192.168.40.1/24 ipv6.method disabled connection.zone servers
  # repeat: mgmt 192.168.10.1, clients 192.168.50.1, dmz 192.168.60.1
  ```
- **FW-B (Debian):** use `iface enp3s0 inet static` blocks without `vlan-raw-device`
- **Both:** the DHCP relay becomes `dhcrelay -id <clients-if> -iu <servers-if> 192.168.40.10`. In nftables, replace `vlanNN` with the real interface names.
- [ ] Apply the **2.2 containment rule** before connecting any other VM

#### T4. Server and client VMs
- [ ] One NIC each, on its segment. mx01/02 add a NIC on `lab-repl` (192.168.71.21/.22, **no gateway**).
- [ ] **No NAT, bridged, or host-only NIC on any server or client**
- [ ] Reuse the template practices from **V6**. On libvirt use `virt-clone` + `virt-sysprep --operations machine-id,ssh-hostkeys`. On VirtualBox, use "Generate new MAC addresses" and then reset the machine-id.
- [ ] Build order is unchanged: ipa01 → ipa02 → util01 → mx01/02 → ws01

#### T5. Fencing for mx01/02 (Type-2 hosts)
Pick one:
- **SBD with a watchdog (recommended, works on every hypervisor, keeps isolation):**
  - [ ] libvirt: add `<watchdog model='i6300esb' action='reset'/>` to each mx VM
  - [ ] VirtualBox or Hyper-V: `echo softdog > /etc/modules-load.d/softdog.conf`. It's lab-grade only.
  - [ ] `dnf install sbd`, then `pcs stonith sbd enable --watchdog=/dev/watchdog` with the cluster stopped. **Requires corosync-qdevice on util01**, which is already in the design.
  - [ ] Set `stonith-watchdog-timeout` to twice the SBD watchdog timeout
- **`fence_virsh` (libvirt):** mx01/02 SSH to the host and run `virsh destroy`. This needs a route from the lab to the host, which **breaks containment**. Allow it only to `host:22` from `lab-repl` (use 2T.4-B-style host addressing on `lab-repl`) and record the exception.
- **`fence_xvm`:** needs `fence_virtd` on the host listening on a lab bridge, so it has the same containment trade-off
- **`fence_vbox`:** exists upstream in ClusterLabs fence-agents but **isn't packaged in EL9's HA repo**. Treat it as unsupported.
- [ ] **✓ verify:** `pcs stonith fence mx02` (or `pcs stonith sbd status`) actually resets mx02, and resources move to mx01

#### T6. Kill-switch and containment tests (add to S7)
- [ ] **Containment:** from every segment, the host's real LAN gateway and other RFC 1918 hosts are unreachable, and fw01 logs `contain` drops
- [ ] **Kill switch:** cut the WAN.
  - Cut it with `virsh domif-setlink fw01 <wan-vnet> down`, or `VBoxManage controlvm fw01 setlinkstate1 off`
  - Then confirm all of these still work:
    - IPA login on ws01
    - internal mail, mx01 → mx02 user
    - `dnf makecache` against the util01 mirror
    - `chronyc tracking` (orphan stratum)
  - Only external DNS and HTTPS fail
  - Restore the link afterward
- [ ] **No bypass:** on each VM, `ip r` has exactly one default route, via `.1`. `tracepath` shows fw01 as the first hop.

### 2.5 ADD to S7, "Sign-off Tests"
- [ ] `[ALL]` The containment test (T6) passes on every build. It matters for Part 1 and Part 2 too whenever the WAN is a shared or real network.
- [ ] `[2-T]` The kill-switch test (T6) passes
- [ ] **DHCP:** CHANGE "DHCP snooping blocks a rogue server" to `[HW]` only. For `[VM]`/`[2-T]`, confirm with `virsh net-dumpxml` / `VBoxManage list dhcpservers` that **no hypervisor DHCP** is serving a lab segment.

---

## 3. Changes to [[lstyle-annexb-prereq]]

### 3.1 CHANGE §1, "Lab Platform and Sizing" → Hypervisor table
| Option | Type | Notes |
|---|---|---|
| KVM/libvirt + virt-manager | FOSS | **Recommended for a single Linux host.** Use isolated networks behind a router VM ([[network-prereqs]] Part 2-T). Fencing: **SBD + `i6300esb`** (keeps isolation), or `fence_virsh`/`fence_xvm` (needs a path to the host). |
| *(new row)* **VirtualBox 7.x** | FOSS base (GPLv3). The Extension Pack is proprietary and not needed. | Windows/macOS hosts. Internal networks + NAT WAN on the router VM. Fencing: **SBD + softdog** (lab-grade). `fence_vbox` isn't in EL9 HA. |
| Proxmox VE | (unchanged) | (unchanged) |
| VMware Workstation/vSphere | (unchanged) | ADD: "Workstation Pro is free of charge but proprietary. Check the VMnet8 subnet for overlap." |

### 3.2 ADD to §1, "Network Prerequisites"
- [ ] The gateway for every segment is **the router VM (`fw01`)**. Pick **FW-A** or **FW-B** and build it **before** ipa01. The upstream DNS forwarder for `ipa-server-install --forwarder=` is **`192.168.40.1`**, not a public resolver.
- [ ] The hypervisor's own DHCP is **off** on every lab network
- [ ] The WAN mode (NAT or bridged) has been chosen, and the hypervisor NAT subnet doesn't overlap the lab plan
- [ ] The containment rule is in place before any VM other than fw01 is connected

### 3.3 CHANGE §1, "Build Order"
0. **`fw01` (router VM)**, followed by the containment and verify tests
1. ipa01 … 5. ws01 (unchanged)

### 3.4 CHANGE §6, "High Availability" → Fencing bullet
Replace it with:
> **Fencing (STONITH) is mandatory.**
> - Proxmox: `fence_pve`
> - Bare metal: `fence_ipmilan`
> - Desktop / Type-2 lab: **watchdog-only SBD + corosync-qdevice** on util01. It needs no shared disk and no path to the host, so lab isolation holds. Use `i6300esb` on libvirt and `softdog` elsewhere.
> - `fence_virsh`/`fence_xvm` work too, but they need a firewall exception to reach the host.

### 3.5 CHANGE §8, "Patching"
- `dnf reposync` + httpd mirror on util01 moves from "recommended" to **required when the kill switch is used**. Without it, an offline lab can't install packages.
- Sync the mirror while the WAN is up. Add a systemd timer, e.g. nightly `dnf reposync --download-metadata -p /srv/repos`.

### 3.6 ADD to §10, "Monitoring" alternatives
| Option | Consider when | Caveat |
|---|---|---|
| **Zeek** on fw01 (Linux FW-B) or on a mirrored port | You want protocol logs (conn, dns, http, ssl) from the single choke point | Adds 2–4 GB of RAM to fw01. On OPNsense, use the Zenarmor (proprietary, free tier) or Suricata plugins instead. |

### 3.7 ADD to §13.6 quick reference
| Tool | What it is (≈ Windows) | License | Key paths | Key commands |
|---|---|---|---|---|
| **libvirt networks** | Hyper-V virtual switches | LGPL | `/etc/libvirt/qemu/networks/` | `virsh net-list --all`, `net-define`, `net-dumpxml`, `domif-setlink` |
| **VirtualBox** | Hyper-V / VMware Workstation | GPLv3 (base) | `~/VirtualBox VMs/` | `VBoxManage list vms`, `modifyvm --nicN intnet`, `controlvm … setlinkstate1 off` |
| **sbd** | (none; the closest is the DAG's own fencing) | GPLv2 | `/etc/sysconfig/sbd` | `pcs stonith sbd enable/status`, `sbd query-watchdog` |
| **tcpdump** | Wireshark CLI / pktmon | BSD | — | `tcpdump -ni <if> host <ip>` |

---

## 4. Changes to [[lstyle-annex-b-brief]]

- **§2 Base-Configure: ADD**
  - [ ] Confirm the path out: `ip r` shows one default route via `192.168.40.1` (fw01), and `tracepath 9.9.9.9` has fw01 as hop 1
- **§3 Domain Services: CHANGE**
  - [ ] `--forwarder=<upstream>` becomes `--forwarder=192.168.40.1` (Unbound on the router VM, per the [[network-prereqs]] DNS chain)
- **Target Build: ADD a row**

  | Host | Replaces | Software | Example |
  |---|---|---|---|
  | `fw01` | Lab range gateway (implied in Annex B) | OPNsense **or** Rocky/Debian router: NAT, Unbound, chrony, DHCP relay | `.1` on each segment; WAN = hypervisor NAT |

- **Final Proof of Function: ADD**
  - [ ] Containment: lab hosts can't reach the host's real LAN. Kill switch: with fw01's WAN down, logins, internal mail, and the repo mirror still work.
- **"Mail a neighbor site": ADD a note**
  - Behind double NAT, two student labs can't reach each other directly. Use the **range transit network** below, or build both sites on the same host.

### Range transit network (neighbor-site design)
This applies the router-VM concept at the range level. It replaces port forwards through double NAT.
```
 site A fw01 WAN ─┐
 site B fw01 WAN ─┼── "range-transit" network 172.16.255.0/24 ── isp01 router VM (NAT) ── Internet
 site C fw01 WAN ─┘        (static WAN IPs .11/.12/.13)
```
- [ ] `isp01` is a small Debian or OPNsense router VM: NAT to the internet, plus routes `192.168.x` per site **only if** sites use distinct ranges. Otherwise use VPN tunnels.
- [ ] Sites peer over **WireGuard or IPsec** between their WAN IPs on `172.16.255.0/24`. They're directly reachable there, so there's no double NAT between sites.
- [ ] Add an inter-site exception to each site's containment rule: allow `172.16.255.0/24` for the VPN only (UDP 51820, or 500/4500 + ESP)
- [ ] On one host, `range-transit` is just another isolated network. Across several hosts, make it a bridged network on a classroom switch or VLAN.
- [ ] Conditional forwarders between sites' IPA servers and Postfix `transport_maps` then work as described in [[lstyle-annex-b-brief]] §5

---

## 5. Changes to [[annex-b-brief]] (Windows edition)

Add these to "Scrutiny: Gaps and Fixes":
- [ ] **R: Isolate the range behind a router VM.** Lab 1 turns the Windows Firewall off. That's tolerable **only** if the range is contained.
  - On Hyper-V: put the DCs, Exchange, and UTIL on a **Private** switch, and route through a pfSense/OPNsense or Linux router VM (External/Default Switch WAN, Private LAN).
  - Block the router's outbound traffic to RFC 1918 space so the range can't reach the real network.
  - It also gives Lab 1's gateway field a real target (`192.168.40.1`).
- [ ] **O: Kill-switch drill.** Disconnect the router's WAN and prove AD logons, GPO, internal DNS, and DAG mail still work. That shows the services depend only on the internal infrastructure, which fits the DS.02.07/08 troubleshooting objectives.

---

## 6. Fixes for [[vm-router-lab-guide]] itself (if you keep it as a standalone handout)

| Section | Issue | Fix |
|---|---|---|
| §4B step 1 | Persists the setting into `/etc/sysctl.conf` | Use `/etc/sysctl.d/90-router.conf` (as in FW-B). `sysctl.conf` is legacy, and STIG/CIS tooling reads `sysctl.d`. |
| §4B step 2 | `ip addr add` is lost at reboot, and the persistence note is buried | Show the `nmcli` (Rocky) or `/etc/network/interfaces` (Debian) line inline |
| §4B step 3 | Masquerade with **no forward filter**. Every LAN host can reach the host's real LAN, which contradicts the §8 "Containment" claim. | Add `chain forward { policy drop; ct state established,related accept; iifname "enp0s8" oifname "enp0s3" ip daddr != { 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 } accept }` |
| §4A | pfSense/OPNsense defaults pass LAN → any, including the host's LAN | Add the floating RFC 1918 block rule (2.2 above) |
| §5 | Static clients fall back to `8.8.8.8` | That bypasses the router's DNS, and IPA if one exists. Use the router (or IPA) as the only resolver. |
| §6 step 3 | `ping 8.8.8.8` | Fine, but also add "ping the host's LAN gateway → **must fail**" as a containment check |
| §7 | No subnet-overlap gotcha | Add 1.3 above. VMware's VMnet8 is the usual culprit. |
| General | Uses `10.10.10.0/24` | Fine standalone. For the Annex B build, use the 192.168.x plan in [[network-prereqs]] S1. |

---

## 7. Verify Before Relying On

These details come from memory and haven't been tested against current releases:
- [ ] Whether watchdog-only SBD + qdevice is **supported** for 2-node clusters on Rocky 9 (RHEL documents it with caveats), and the exact `pcs stonith sbd enable` syntax
- [ ] That `fence_vbox` really isn't in the EL9 HA repo (`dnf list 'fence-agents-*'`)
- [ ] The libvirt network XML with no `<ip>`: confirm `virsh net-start` creates the bridge with no host address on your libvirt version
- [ ] The VirtualBox NIC limit per chipset (PIIX3 vs ICH9) and the `--nicpromiscN` value names
- [ ] chrony `local stratum 10 orphan` behavior with only one server in the lab (fw01)
- [ ] The OPNsense *Block private networks* direction (inbound on WAN only) on your version

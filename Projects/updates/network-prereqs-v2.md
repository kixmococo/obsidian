# Network Prerequisites: Hardware, Platform, and Software Selection

> [!info] v2: router-VM update
> This copy adds the [[vm-router-lab-guide]] concept (single router VM, containment, kill switch, desktop Type-2 lab). The original is unchanged in `endstate/`. Change list: [[router-vm-updates]].

Companion to [[network-guide-v2]] (the build checklist). It sits under [[lstyle-annex-b-brief-v2]] and [[lstyle-annexb-prereq-v2]]: the network built here is what the FreeIPA, mail, and utility servers run on.

**This doc is in two parts, and both use the shared sections.**

| Section | Covers |
|---|---|
| **Shared S1–S4** | Logical design, firewall/router choice (**OPNsense** and **Linux router** each get their own section), network services, guest OS |
| **Part 1: Hardware-based** | Every role on its own physical machine. No hypervisor. |
| **Part 2: VM-based** | Every role, including the firewall and the switching, runs as VMs on one or more hypervisor hosts |
| **Part 2-T: Desktop router-VM lab** | The whole lab on one laptop or desktop (Type-2 hypervisor), on isolated networks behind a single router VM |

**Assumptions**
- Physical devices are bought or issued separately. This doc gives the **requirements and recommendations**, not specific purchases.
- Everything that *runs on* that hardware is free and open-source, except where a paid option is called out as clearly better.
- Names and IPs match the earlier docs: `ipa01/02`, `mx01/02`, `util01`, `ws01`, and `192.168.40.0/24` for servers.

> [!warning] Fielding on a DoD network
> Hardware must be on the **DoDIN Approved Products List (APL)**, and all software must be covered by the system's ATO. The FOSS stack here is ideal for training and labs. A fielded system usually swaps the perimeter firewall and switches for APL-listed gear.

### Hardware or VMs?
| | Part 1: Hardware | Part 2: VMs (Proxmox) | Part 2-T: Desktop router-VM lab |
|---|---|---|---|
| Cost and footprint | High: about 8 servers, a rack, and a large UPS | Low: 1–3 hosts | **Lowest:** one existing laptop or desktop |
| Fault isolation | Strong. One box failing affects one role. | A host failure hits many VMs unless you cluster | None. One machine. |
| Lab containment | Physical | VLAN bridges + fw01 | **Isolated hypervisor networks behind one router VM**, with a kill switch |
| Recovery | ReaR, bare-metal restore, spare parts | Snapshots, PBS restore in minutes, live migration | Snapshots, exported VM images |
| Skills taught | Cabling, RAID, BMC/IPMI, PXE, physical fencing | Virtual networking, templates, clustering | Routing, NAT, DHCP, DNS, and firewall basics, watched packet by packet |
| Best for | Fixed sites and hands-on hardware training | Labs, ranges, and deployable kits | **Individual student practice** and contained sandboxes |

> [!tip] Production is usually a hybrid
> A common production layout uses Part 1's **physical firewall and switch** with Part 2's **virtualized servers**. Build each layer from whichever part fits it.

---

## S1. Logical Design (shared; decide before racking or clicking)

### VLAN and IP Plan
| VLAN | Name | Subnet | Gateway | Contents | Used in |
|---|---|---|---|---|---|
| 10 | MGMT | 192.168.10.0/24 | .1 (fw01) | switch .2, IPMI/BMC .21–.29, hypervisors .11–.13, backup server .30, admin laptop .50 | Both |
| 40 | SERVERS | 192.168.40.0/24 | .1 (fw01) | ipa01 .2, ipa02 .3, util01 .10, mail VIP .20, mx01 .21, mx02 .22, netops01 .30 | Both |
| 50 | CLIENTS | 192.168.50.0/24 | .1 (fw01) | DHCP pool .100–.200 (Kea on util01 via relay) | Both |
| 60 | DMZ *(optional)* | 192.168.60.0/24 | .1 (fw01) | SMTP relay or webmail reverse proxy | Both |
| 70 | HOST-CLUSTER | 192.168.70.0/24 | **none** | Proxmox corosync (non-routed) | Part 2 |
| 71 | GUEST-CLUSTER | 192.168.71.0/24 | **none** | mx01/02 DRBD + Pacemaker ring. In Part 1 this can be a direct cable instead. | Both |
| 80 | STORAGE | 192.168.80.0/24 | **none** | Ceph, NFS, and backup traffic. MTU 9000. | Mostly Part 2 |
| 999 | BLACKHOLE | — | — | Native VLAN on every trunk, and all unused ports | Both |

> [!note] Part 2-T
> On a desktop hypervisor, each routed VLAN (10/40/50/60) becomes its **own isolated network** (`lab-mgmt`, `lab-servers`, `lab-clients`, `lab-dmz`) with the same subnet. VLAN 71 becomes `lab-repl`, which only mx01/02 attach to. VLANs 70, 80, and 999 aren't used.

> [!warning] Check the WAN-side subnet
> fw01's WAN address comes from the hypervisor's NAT network or the real LAN, and it **must not overlap** 192.168.10/40/50/60/71:
> - VirtualBox NAT: `10.0.2.0/24`
> - libvirt `default`: `192.168.122.0/24`
> - VMware VMnet8: a random `192.168.x.0/24`, so **check it**
> - Home LANs are usually `192.168.0/1.x`
>
> If anything overlaps, change the hypervisor NAT range, not the lab plan.

### DNS and NTP Chains
- **DNS:**
  1. Clients and servers query **ipa01/ipa02**.
  2. The IPA servers forward to **fw01** (Unbound) at 192.168.40.1.
  3. fw01 sends queries to the upstream or inter-site resolvers.
  4. fw01 also forwards `example.mil` and its reverse zones back to the IPA servers, so management devices can resolve domain names.
- **NTP:**
  1. An upstream, DoD, or GPS source feeds **fw01**.
  2. fw01 serves the IPA servers, switches, hypervisors or servers, and BMCs.
  3. The IPA servers serve domain clients.
- **With the WAN cut (kill switch):**
  - Internal names keep resolving, because IPA is authoritative for `example.mil`. Only external lookups fail, which is expected.
  - fw01 runs chrony with `local stratum 10 orphan` (OPNsense: the NTP *orphan* stratum), so the lab keeps **consistent** time. Kerberos needs the clocks to agree, not to be correct.

### Firewall Policy (default deny between VLANs)
| From → To | Allow |
|---|---|
| CLIENTS → SERVERS | 53, 88/464 (TCP/UDP), 389/636, 443, 587/465, 993, 123/udp |
| CLIENTS → WAN | 80/443 (proxy optional) |
| CLIENTS → MGMT | **deny** |
| SERVERS → WAN | 53 (from IPA only), 123, 25 (from the mail VIP only), 443 (updates, or through util01's mirror) |
| **Any lab zone → RFC 1918 / CGNAT on the WAN side** (`10/8`, `172.16/12`, `192.168/16`, `100.64/10`), except the WAN gateway | **deny and log (containment).** Evaluate it **first**, above every allow rule. |
| MGMT → internal zones | 22, 443, 8006/8007 (Proxmox/PBS, Part 2), 623/udp (IPMI, Part 1), 161/udp (SNMP from netops01) |
| MGMT → WAN | 53, 80/443, 123 only |
| Inter-site VPN → SERVERS | 25 to the mail VIP, 53 to ipa01/02 |
| VLANs 70 / 71 / 80 | Not routed at all |
| WAN → any | **deny.** No port forwards, except the DMZ or inter-site VPN if you build one. |

**Kill switch:** taking fw01's WAN link down must leave every *internal* service working (logins, DNS, internal mail, the util01 repo mirror, time). It's tested in [[network-guide-v2]] S7.

---

## S2. Firewall / Router: Pick One

Both options deliver the same function: routing between the VLANs, NAT, the firewall policy above, Unbound DNS, NTP, DHCP relay, site-to-site VPN, IDS, and optional HA. Either runs on a physical box (Part 1) or as a VM (Part 2).

| | **S2-A OPNsense** | **S2-B Linux router (Rocky 9 / Debian 13)** |
|---|---|---|
| Base OS | FreeBSD (not Linux) | Linux |
| Management | Web GUI + API (CLI is limited) | CLI and text files (Cockpit optional) |
| Time to build | Hours | About a day |
| STIG story | No STIG. Apply the Firewall and Network Device SRGs by hand. | Rocky: the RHEL 9 STIG applies, with documented router exceptions. Debian: no STIG. |
| Teaches | Firewall concepts | The internals (nftables, FRR, Unbound, Kea) and the same skills as the servers |
| HA | CARP + pfsync, built in | keepalived (VRRP) + conntrackd |
| Pick when | You want speed, a GUI, and a training-friendly build | **Linux-only is required**, or the goal is deep understanding |

### S2-A. OPNsense
- **What it is:** a FreeBSD-based firewall and router distribution (2-clause BSD license). It is fully FOSS. Releases come twice a year (`YY.1` and `YY.7`), with frequent minor updates.
- **Built in:**
  - pf stateful firewall, VLANs, NAT, aliases
  - **Unbound** DNS
  - **Kea/ISC DHCP and relay**, NTP
  - **WireGuard, OpenVPN, and IPsec (strongSwan)**
  - **Suricata IDS/IPS**
  - CARP HA, NetFlow, two-factor auth for the GUI, and config history with backup
- **Requirements:**

  | | Physical (Part 1) | VM (Part 2) |
  |---|---|---|
  | CPU | 64-bit, 2+ cores (4+ with Suricata), **AES-NI** | 2–4 vCPU, CPU type `host` or `x86-64-v2-AES` |
  | RAM | 4 GB minimum, 8–16 GB with IDS/IPS | 4 GB (8 GB with IDS) |
  | Disk | 32 GB minimum, 120 GB+ SSD recommended | 32 GB virtio disk |
  | NICs | **4–6 Intel NICs** (i210/i225/i226/X710) | 2+ VirtIO NICs (WAN, plus a trunk) |
- **Gotchas:**
  - FreeBSD's best NIC drivers are Intel. Realtek NICs are a common source of trouble.
  - In a VM, keep hardware offloads (CRC, TSO, LRO) **disabled** under *Interfaces › Settings*.
  - There is no STIG. Document SRG compliance by hand.
- **Alternatives in the same family:**
  - **pfSense CE**: free, but Netgate's development effort goes into the paid Plus edition.
  - Paid **Deciso** appliances (OPNsense Business Edition) add vendor support.

### S2-B. Linux Router (Rocky Linux 9, or Debian 13)
- **What it is:** a minimal Linux install configured as a router out of individual FOSS components:

  | Function | Rocky 9 | Debian 13 | License |
  |---|---|---|---|
  | Packet filter / NAT | **firewalld** (zones + **policy objects**) on nftables | **nftables** (`/etc/nftables.conf`) | GPLv2 |
  | VLAN interfaces | NetworkManager (`nmcli`) | ifupdown or systemd-networkd | GPLv2+ / LGPLv2.1+ |
  | DNS resolver | **Unbound** (AppStream) | Unbound | BSD |
  | DHCP | **Kea** (or relay to util01 with `dhcrelay`/`dnsmasq`) | Kea | MPL 2.0 |
  | NTP | chrony | chrony | GPLv2 |
  | Dynamic routing *(optional)* | **FRR** (AppStream) | FRR | GPLv2+ |
  | Site-to-site VPN | WireGuard (`wireguard-tools`; **Technology Preview** in RHEL 9), or **Libreswan** IPsec (fully supported) | WireGuard (fully supported) or **strongSwan** | GPLv2 |
  | IDS/IPS | **Suricata** (EPEL) | Suricata (main repo) | GPLv2 |
  | HA | **keepalived** (VRRP) + **conntrackd** (state sync) | Same | GPLv2+ |
  | Flow export | softflowd / pmacct → ntopng | Same | BSD / GPLv2 |
  | Web admin *(optional)* | Cockpit | Cockpit | LGPLv2.1+ |
- **Requirements (physical or VM):**
  - 2+ cores (4+ with Suricata)
  - 2 GB RAM (8 GB with Suricata)
  - 20 GB disk
  - Physically, 2–6 Intel NICs. Linux drivers are broader than FreeBSD's, but Intel is still the safe choice.
- **Which distro:**
  - **Rocky 9** for STIG alignment and the same platform as every other server.
  - **Debian 13** if you want WireGuard as a fully supported feature, newer packages, and native nftables. The cost is no STIG.
- **Gotchas:**
  - The STIG profile sets `net.ipv4.ip_forward=0`. A router needs it set to `1`, so **document that as an approved exception**.
  - Unbound blocks private reverse zones by default. Forwarding `168.192.in-addr.arpa` to IPA needs `local-zone: "168.192.in-addr.arpa." nodefault`.
  - IPA's zones are unsigned, so add `domain-insecure: "example.mil"`. Otherwise DNSSEC validation fails.
  - ISC `dhcrelay` is EOL but still packaged. `dnsmasq --dhcp-relay` is an alternative, or run Kea on the router itself.
- **Alternatives in the same family:**
  - **VyOS** (Linux, with a Juniper-style CLI and commit/rollback; free "Stream" builds, but **LTS images require a subscription**)
  - OpenWrt x86, for small edge or tactical nodes

### S2-D. Gotchas for Either Option (as a VM)
- **CARP / keepalived VIPs** need the hypervisor to accept frames from virtual MACs (`00:00:5e:00:01:xx`):
  - Hyper-V: *MAC address spoofing*
  - ESXi: *Forged transmits / MAC changes*
  - VirtualBox: *Promiscuous Mode: Allow All* on fw NICs

  Pacemaker's `IPaddr2` mail VIP moves only the IP, not the MAC, so it doesn't need this.
- **Outbound containment isn't a default on either option.** OPNsense's *Block private networks* applies to **inbound** WAN traffic only, and Linux masquerade forwards everything. Add the RFC 1918 egress block from S1.
- **Bridged WAN over Wi-Fi** usually fails because the access point drops unknown MACs. Use **NAT** for the WAN on laptops.
- **Docker or firewalld on a Debian router** can rewrite nftables. Don't install container engines on fw01.

### S2-C. Paid Option (clearly better for fielded perimeters)
**Palo Alto, Fortinet, or Cisco Secure Firewall:** App-ID/NGFW features, threat intelligence feeds, DISA STIGs, APL listing, and vendor support.

---

## S3. Network Services (shared)
| Function                          | Recommended (FOSS)                                                         | License        | Alternatives / Paid                                                      |
| --------------------------------- | -------------------------------------------------------------------------- | -------------- | ------------------------------------------------------------------------ |
| Network device admin AAA (RADIUS) | **FreeRADIUS** on `util01`, with FreeIPA as the user backend               | GPLv2          | `tac_plus-ng` (TACACS+). Paid: **Cisco ISE**.                            |
| 802.1X / NAC                      | **PacketFence** (optional, heavy)                                          | GPLv2          | Paid: Cisco ISE, Aruba ClearPass                                         |
| IPAM / DCIM ("source of truth")   | **NetBox**                                                                 | Apache 2.0     | phpIPAM                                                                  |
| Network config backup             | **Oxidized**                                                               | Apache 2.0     | RANCID                                                                   |
| Network monitoring (SNMP)         | **LibreNMS** on `netops01` (Zabbix on `util01` keeps watching the servers) | GPLv3          | Paid: SolarWinds NPM                                                     |
| Flow analysis                     | **ntopng Community**                                                       | GPLv3          | pmacct / nfdump. ElastiFlow (free tier plus paid tiers).                 |
| IDS / NSM                         | **Suricata** on the firewall                                               | GPLv2          | **Zeek**, or **Security Onion** (needs 16–24 GB RAM and a SPAN port)     |
| VPN                               | **WireGuard**                                                              | GPLv2          | **IPsec** (strongSwan or Libreswan) for DoD and vendor interop. OpenVPN. |
| Edge DNS                          | **Unbound**                                                                | BSD            | BIND, PowerDNS Recursor                                                  |
| NTP                               | **chrony** (ntpd on OPNsense)                                              | GPLv2          | GPS + `gpsd` for stratum 1                                               |
| Bastion / jump access             | **Apache Guacamole** + an SSH bastion                                      | Apache 2.0     | Teleport Community                                                       |
| UPS shutdown                      | **NUT** (Network UPS Tools)                                                | GPLv2+         | Vendor agents (paid or proprietary)                                      |
| Packet capture                    | `tcpdump` and **Wireshark**                                                | BSD / GPLv2    | —                                                                        |
| Vulnerability scanning            | **Greenbone/OpenVAS**                                                      | AGPLv3 / GPLv2 | Paid: **ACAS (Tenable)**, which is DoD-mandated                          |
| Diagrams                          | draw.io                                                                    | Apache 2.0     | Visio (paid)                                                             |

**DHCP notes**
- Turn off **every hypervisor-provided DHCP** on internal and host-only networks. Kea on util01 (through the fw01 relay) must be the only DHCP server on CLIENTS.
- **Bootstrap:** servers are static, so Kea isn't needed until `ws01`. If you need DHCP earlier, turn on fw01's DHCP for CLIENTS temporarily, then switch to the relay.
- **Zeek** on fw01 (FW-B) is worth adding in Part 2-T. fw01 is the single choke point, so it sees every flow. Budget 2–4 GB of extra RAM.

## S4. Server and Client OS (shared)
Use **Rocky Linux 9** for every server, whether physical or virtual, for the reasons in [[lstyle-annexb-prereq-v2]] §2. Use Rocky Workstation for `ws01`.

---

# Part 1: Hardware-Based Build

Every role runs on its own physical machine, with no hypervisor.

## 1.1 Topology
```
              Upstream / ISP / inter-site
                        │ WAN
              ┌─────────┴─────────┐
              │ fw01  (S2-A or B) │  physical appliance (+ fw02 for HA)
              └─────────┬─────────┘
                        │ 802.1Q trunk
              ┌─────────┴─────────┐
              │  sw01  managed    │  (+ sw02 for redundancy, optional)
              └─┬──┬──┬──┬──┬──┬──┘
   access/LACP  │  │  │  │  │  └── ws01 (VLAN 50), admin laptop (VLAN 10)
       ipa01 ───┘  │  │  │  └──── BMC/IPMI ports of every server (VLAN 10)
       ipa02 ──────┘  │  └─────── util01, netops01, bkp01
       mx01 ═══════ mx02  (direct 10G cable: DRBD + corosync ring 2)
```

## 1.2 Hardware Requirements

### Network devices
- **Firewall:** see S2-A or S2-B, physical column. Buy **two identical units** if you want HA.
- **Managed switch (`sw01`):**
  - 24× 1GbE plus 4× SFP+ 10GbE
  - Required features: 802.1Q, **LACP**, RSTP, **DHCP snooping**, port security, BPDU guard, storm control, **802.1X**, RADIUS/TACACS+, SSH, **SNMPv3**, syslog, NTP, LLDP, and SPAN
  - **Buy what your unit fields.** Cisco Catalyst skills transfer furthest in the Marine Corps. Avoid "smart/web-managed" switches that have no CLI.

### Servers (one per role)
| Host | CPU | RAM | Storage | NICs | Must have |
|---|---|---|---|---|---|
| `ipa01`, `ipa02` | 4 cores | 8–16 GB ECC | 2× 480 GB SSD (RAID1) | 2× 1GbE (LACP) | BMC, TPM 2.0 |
| `mx01`, `mx02` | 4–8 cores | 16 GB ECC | 2× SSD (OS RAID1) + 2× SSD/HDD (data RAID1 → DRBD) | 2× 1GbE (LACP) + **1× 10GbE** (direct cable to its peer) | **BMC required**, because Pacemaker fences through IPMI (`fence_ipmilan`) |
| `util01` | 4 cores | 16 GB | OS RAID1 + 2–4 TB RAID1 (repo mirror, PXE content) | 2× 1GbE | BMC |
| `netops01` | 4 cores | 8–16 GB | 2× SSD (RAID1) | 2× 1GbE | BMC |
| `bkp01` | 4–8 cores | 16–32 GB ECC | SSD for the OS + 4+ HDD in RAIDZ2 or RAID6 | 1–10GbE | Separate rack or room if possible |
| `ws01` | 4 cores | 8–16 GB | 256 GB+ SSD | 1× 1GbE | TPM 2.0, UEFI Secure Boot |

**Form factor**
- Use small 1U servers (for example Dell R250/R350 or HPE DL20/DL320) for a fixed site.
- For a budget or deployable kit, use business SFF desktops with **Intel vPro/AMT**. The fence agent for AMT is `fence_amt_ws`.
- Always buy machines with an **out-of-band controller** (iDRAC, iLO, IPMI, or AMT).

### Facility
- **Rack:** about 12–18U. Put the UPS at the bottom and leave airflow gaps.
- **Power:** about 8 servers at 150–250 W each, plus the network gear, comes to roughly **1.5–2.5 kW**. Get a **3 kVA** online UPS with a network management card, plus a metered PDU.
- **Cooling:** plan for about 5,000–8,500 BTU/h.
- **Out-of-band access:** a console server or USB-serial cables for the switch and firewall. BMC ports go on VLAN 10.
- **Cabling:** Cat6, plus one 10G DAC per mx pair. Label both ends of everything.
- **Admin laptop:** Linux, with Wireshark, `picocom`, Ansible, and an SSH key or CAC.
- **Spares:** one switch, one PSU, and one disk of each type.
- *(Optional)* a **GPS NTP** receiver.

## 1.3 Software Specific to Bare Metal
| Need | FOSS choice | Notes |
|---|---|---|
| Disk redundancy | **mdadm** software RAID1 (set up by Anaconda or Kickstart) | Any hardware RAID controller works too. If you use one, monitor it through vendor tools (often proprietary). |
| NIC redundancy | NetworkManager **bond** (802.3ad/LACP) | Pairs with a port-channel on the switch |
| Mass OS install (≈ WDS/MDT) | **PXE/UEFI-HTTP boot + Kickstart** on `util01`: Kea (boot options), `tftp-server`, `httpd`, `shim`/`grub2-efi` | Alternatives: **Foreman** (full provisioning) or **Cobbler** |
| Firmware updates | **fwupd/LVFS** where the vendor supports it, otherwise vendor ISOs | Dell and HPE ship their own update tools, which are mostly proprietary |
| Hardware health | Redfish/IPMI polled by LibreNMS or Zabbix, plus `ipmitool sel list` | `smartctl` (smartmontools) for disk health |
| Cluster fencing (mail) | `fence_ipmilan` or `fence_redfish` against each mx BMC | Put the BMCs on VLAN 10 and give the cluster its own BMC account |
| Backup | **ReaR** (bare-metal recovery ISO) to an NFS share on `bkp01`, plus **Bareos** or **restic** for files | `bkp01` can run **TrueNAS Community** for ZFS storage |
| UPS | **NUT** server on `util01` (USB or SNMP to the UPS). Every server runs as a `netclient`. | Shutdown order: clients → mx → util/netops → ipa last. Boot in the reverse order. |

## 1.4 Part 1 Prerequisite Checklist
- [ ] All hardware in 1.2 is on hand. **Every server has a working BMC.** APL status is checked if the build will be fielded.
- [ ] Rack, power (3 kVA UPS), and cooling are in place. Console cables are on hand.
- [ ] S1 design (VLANs, DNS/NTP chain, policy) approved and entered in NetBox once it exists
- [ ] Firewall option chosen (S2-A or S2-B) and its install media ready
- [ ] Rocky 9 DVD ISO on hand, plus Kickstart files per role. Optionally set up the PXE content on `util01`.
- [ ] BMC credential plan made: unique per host, stored in a vault, and a dedicated fencing account
- [ ] Inter-site peers' VPN endpoint, subnets, and DNS forwarder IPs agreed

---

# Part 2: VM-Based Build

Every role, including the firewall and the "switch", runs as VMs on hypervisor host(s). The only physical pieces are the host(s), the WAN uplink, and optionally a switch.

## 2.1 Topology

**Single host (no physical switch):**
```
 WAN ──[nic1]── vmbr0 (WAN bridge, no host IP) ── fw01 VM (S2-A or B)
                                                    │ trunk (all VLANs)
 admin laptop ──[nic2]── vmbr1 (VLAN-aware internal "switch") ──┬── ipa01/02 (VLAN 40)
   (untagged = VLAN 10)    host mgmt IP on vmbr1.10              ├── mx01/02 (VLAN 40 + 71)
                                                                 ├── util01, netops01 (VLAN 40)
                                                                 └── ws01 (VLAN 50)
```

**Cluster (3 hosts):** the same bridges, but each host's `vmbr1` runs over an **LACP bond to a physical switch** carrying VLANs 10/40/50/60/70/71/80. The firewall VM gets HA. A physical firewall (the hybrid layout) is also common here.

## 2.2 Host Hardware Tiers
| Tier            | Hosts                   | Per host                                                                                                              | Use                                                                        |
| --------------- | ----------------------- | --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| **Laptop lab**  | 1 laptop or workstation | 8+ cores, **32 GB** (tight) to 64 GB, 1 TB NVMe                                                                       | Personal practice. Use **Part 2-T** (router-VM design) instead of Proxmox. |
| **Single host** | 1                       | 8 cores/16 threads, **64 GB** (128 GB is comfortable), 2× 1 TB NVMe (ZFS mirror), **2–4 NICs** (WAN, LAN/mgmt, spare) | The full lab on one box. No host HA.                                       |
| **Budget HA**   | 2 + witness             | As below                                                                                                              | QDevice on a small third box for quorum (the same idea as the DAG witness) |
| **Recommended** | 3                       | 8–16 cores, **128 GB ECC**, 2× SSD boot mirror + 1–2 NVMe for data/Ceph, **2× 10GbE + 2× 1GbE**, BMC                  | Real HA. Ceph or replicated ZFS.                                           |

**Must have**
- VT-x/AMD-V and **VT-d/IOMMU** (needed for NIC passthrough to the firewall)
- A CPU at **x86-64-v3** or newer (Haswell+), so EL10 guests are possible
- ECC RAM (with ZFS)
- A BMC on each host
- A UPS with USB or a network card

Form factor: used 1U/2U enterprise servers (for example R640/R650 or DL360 Gen10) for a fixed lab. For deployable kits, SFF or mini PCs with 2× 10GbE.

## 2.3 Hypervisor OS: **Proxmox VE 9** (recommended)
- **What it is:** a Debian 13-based distribution bundling KVM/QEMU, LXC, **ZFS**, **Ceph**, clustering, HA, a host firewall, **SDN** (VLAN zones and VNets), and a web UI and API. Licensed AGPLv3. Paid support subscriptions are optional, and the `pve-no-subscription` repo is free.
- **Requirements:**
  - 64-bit CPU with VT-x/AMD-V
  - RAM: 2 GB for the host, plus the ZFS ARC (capped by default at 10% of RAM), plus the guests
  - SSD or NVMe boot disks on a ZFS mirror
- **Why this one:**
  - It's the most complete FOSS replacement for vSphere: cluster, live migration, HA, backup integration (PBS), and SDN in one product.
  - It has a large community.
  - Commercial backup tools such as Veeam now support it.
- **Gap:** there is **no DISA STIG** for Proxmox or Debian. Harden it to CIS Debian guidance and record the exceptions.

| Alternative | Consider when | Caveat |
|---|---|---|
| **Rocky 9 + KVM/libvirt + Cockpit** (`cockpit-machines`) | **STIG coverage of the host is mandatory** | No built-in cluster or HA. Building that means Pacemaker plus shared storage. |
| **Fedora / Rocky Workstation + virt-manager** | Laptop lab | Single machine, but the best desktop experience. See **Part 2-T**. |
| **XCP-ng + Xen Orchestra** | You want a vSphere-like workflow on Xen | Full Xen Orchestra features come with paid support (building from source is free) |
| oVirt | — | Its commercial version (RHV) was discontinued. **Avoid.** |
| OpenStack / Harvester | Large or cloud-native environments | Overkill at this scale |
| **VMware vSphere** (paid) | DoD production with an existing licensing vehicle | **DISA ESXi/vCenter STIGs** and APL listing make it clearly better for compliance. It's expensive. |

## 2.4 Virtual Networking
- **Linux bridge, VLAN-aware** (the Proxmox default): one internal bridge acts as the switch. VM NICs are tagged per VLAN, and the firewall VM's NIC is **untagged** so it receives every VLAN (a trunk).
- **WAN bridge** has **no host IP**. Alternatively, pass the WAN NIC straight through to the firewall VM (PCIe passthrough) for the strongest isolation.
- **Host management** needs its own physical NIC or untagged VLAN 10 port. That way you can still reach the host when the firewall VM is down.
- *(Optional)* **Proxmox SDN:** define VLANs once at the datacenter level for every node. **Open vSwitch** if you need OpenFlow or VXLAN.
- **Firewall VM choice:** S2-A or S2-B. Both are fine as VMs. Give it `startup order=1` and `onboot=1`.

## 2.5 Storage and Backup
| Option | Use |
|---|---|
| **ZFS** (recommended) | Local mirror with checksums, snapshots, and `zfs send` replication between nodes |
| **Ceph** | 3+ nodes, **10GbE or faster**, 3+ OSDs per node. Shared storage that gives true live migration and HA. |
| **TrueNAS Community** | External NFS/iSCSI or backup target |
| **Proxmox Backup Server (PBS)** | Incremental, deduplicated, encrypted VM backups (GPLv3/AGPLv3). **Put it on a separate box.** On a single host it can run as a VM with its own disk, but then sync it off-host. Keep the in-guest backups from [[lstyle-annexb-prereq-v2]] §9 as well. |
| Paid | **Veeam** (supports Proxmox). NetApp, Pure, and other enterprise SANs. |

## 2.6 VM Sizing (whole lab ≈ 15 vCPU / 36 GB)
| VM | vCPU | RAM | Disk | NICs / VLANs |
|---|---|---|---|---|
| `fw01` (OPNsense / Linux) | 2 | 4 GB / 2 GB | 32 / 20 GB | WAN bridge + trunk |
| `ipa01`, `ipa02` | 2 | 4 GB | 30 GB | 40 |
| `mx01`, `mx02` | 2 | 4 GB | 30 GB + 50 GB data (DRBD) | 40 + 71 |
| `util01` | 2 | 8 GB | 100 GB+ | 40 |
| `netops01` | 2 | 4–8 GB | 40 GB | 40 |
| `ws01` | 2 | 4 GB | 40 GB | 50 |
| `pbs01` *(if virtual)* | 2–4 | 4–8 GB | 32 GB + datastore | 10 (+80) |

## 2.7 Part 2 Prerequisite Checklist
- [ ] The host(s) meet 2.2: VT-d enabled, ECC, enough NICs (at least WAN + LAN/mgmt), a BMC, and a UPS
- [ ] Hypervisor decision recorded: **Proxmox** (features) or **Rocky+KVM** (STIG on the host)
- [ ] Firewall option chosen (S2-A or S2-B). Decided whether to bridge or pass through the WAN NIC.
- [ ] S1 design approved
- [ ] ISOs and images downloaded and checksums verified:
  - [ ] Proxmox VE 9
  - [ ] PBS
  - [ ] OPNsense *or* Rocky/Debian (for the firewall)
  - [ ] Rocky 9 GenericCloud qcow2 + DVD
- [ ] Backup target decided (a separate PBS box is strongly preferred)
- [ ] Containment rule (S1) planned for fw01, and the hypervisor NAT/WAN subnet checked for overlap
- [ ] Admin credentials plan made: named accounts, TOTP/WebAuthn on the hypervisor UI, SSH keys, and a sealed break-glass account

---

# Part 2-T: Desktop (Type-2) Router-VM Lab

The whole lab runs on one laptop or desktop. Every VM sits on **isolated** hypervisor networks, and **fw01 is the only path out** (fw01 → hypervisor NAT → host → internet). It's built from [[vm-router-lab-guide]], scaled up to the S1 plan.

## 2T.1 Topology
```
                      ┌────────────── laptop / desktop (host OS) ──────────────┐
 Internet ── host NIC ┤ hypervisor NAT ──[nic1 WAN]── fw01 (S2-A or S2-B)      │
                      │                                │ nic2  nic3  nic4 (nic5)│
                      │   isolated nets:  lab-mgmt ────┘   │     │     │        │
                      │                   lab-servers ─────┘     │     │        │
                      │                   lab-clients ───────────┘     │        │
                      │                   lab-dmz (opt) ───────────────┘        │
                      │                   lab-repl (mx01↔mx02, NOT on fw01)     │
                      └─────────────────────────────────────────────────────────┘
```
- **One isolated network per routed segment.** This replaces VLAN tags. It's simpler and works on every hypervisor. Desktop hypervisors handle 802.1Q trunks into VMs unevenly.
- Same subnets as S1. fw01 is `.1` on each network.
- Every server and client VM has **one NIC** on its segment, except mx01/02, which add `lab-repl` (no gateway).
- **No server or client gets a NAT, bridged, or host-only NIC.** That would bypass the router.

## 2T.2 Host OS and Hypervisor
| Option | License | Pick when | Notes |
|---|---|---|---|
| **KVM/libvirt + virt-manager** on Rocky/Fedora/Debian Workstation (**recommended**) | FOSS (GPL/LGPL) | Linux host | Truly isolated networks with no host IP. It has a virtual watchdog (`i6300esb`) for SBD fencing. It's the same KVM as Proxmox, so the skills carry over. |
| **VirtualBox 7.x** | Base is **GPLv3**. The Extension Pack is proprietary (PUEL) and **not needed**. | Windows or macOS (Intel) hosts | The GUI shows only 4 NICs. Add **nic5+ with `VBoxManage`**. It has no virtual watchdog, so use `softdog`. |
| VMware Workstation Pro | Proprietary, but free of charge since 2024 | It's already your standard | LAN Segments = isolated networks. Check the VMnet8 subnet. |
| Hyper-V (Windows Pro/Ent) | Proprietary, included with Windows | Windows hosts, and matches the Windows [[annex-b-brief]] lab | **Private** switches for the lab and **Default Switch** or External for the WAN. Watch MAC spoofing (S2-D). |
| GNOME Boxes | FOSS | — | **Avoid.** It can't build isolated multi-NIC networks. |

## 2T.3 Host Sizing
The host OS needs its own **4–8 GB of RAM** on top of the VMs.

| Tier | Host RAM | Run |
|---|---|---|
| **Minimum** | 32 GB | FW-B fw01 (1–2 GB), ipa01/02 (3 GB each), mx01/02 (2–3 GB each), util01 (3 GB), ws01 (3 GB). Skip netops01 and pbs01. |
| **Comfortable** | 64 GB | The full 2.6 sizing, plus Zeek on fw01 |

- CPU: 8+ cores with VT-x/AMD-V, at x86-64-v3 or newer
- Disk: 1 TB NVMe, with thin-provisioned qcow2/VDI
- Ethernet only if you'll bridge the WAN

## 2T.4 Host Access (the host can't see isolated networks)
- **A (recommended, purest):** administer from **`ws01`** or a small admin VM inside the lab. Use the hypervisor console until that exists.
- **B:** make **`lab-mgmt` a host-only network** with the hypervisor's DHCP **off**. Give the host `192.168.10.50`, the "admin laptop" in S1. This opens a host↔MGMT path, so record it.

## 2T.5 Fencing (mx01/02)
| Method | Works on | Keeps isolation? | Notes |
|---|---|---|---|
| **Watchdog-only SBD + corosync-qdevice on util01** (**recommended**) | Every hypervisor | **Yes** | No shared disk needed. Use `i6300esb` on libvirt and `softdog` elsewhere (lab-grade). |
| `fence_virsh` | libvirt | No. The mx nodes SSH to the host. | Allow it only to `host:22`, and record the exception |
| `fence_xvm` | libvirt | No. It needs `fence_virtd` on a lab bridge. | Same trade-off |
| `fence_vbox` | VirtualBox | No | Upstream ClusterLabs only. **Not packaged in EL9 HA.** |

## 2T.6 WAN Mode and Neighbor Sites
- **NAT WAN (default):** works on Wi-Fi and is double-NATed, so it's outbound-only. Fine for a single site.
- **Bridged WAN:** wired only. fw01 gets a real-LAN address, and the containment rule matters even more.
- **Neighbor sites ("mail another site"):** double NAT blocks site-to-site traffic. Use a **range transit network**:
  ```
   site A fw01 WAN ─┐
   site B fw01 WAN ─┼── "range-transit" 172.16.255.0/24 ── isp01 router VM (NAT) ── Internet
   site C fw01 WAN ─┘        (static WAN IPs .11/.12/.13)
  ```
  - Sites peer over WireGuard or IPsec between their transit IPs, so there's no NAT between them.
  - Each site's containment rule gets one exception: `172.16.255.0/24` for the VPN only (UDP 51820, or 500/4500 + ESP).
  - On one host, `range-transit` is just another isolated network. Across hosts, bridge it onto a classroom switch or VLAN.

## 2T.7 Part 2-T Prerequisite Checklist
- [ ] Host meets 2T.3, and virtualization is enabled in the firmware
- [ ] Hypervisor chosen (2T.2), and its NAT subnet checked for overlap (S1)
- [ ] Isolated networks planned with **no DHCP**: `lab-mgmt`, `lab-servers`, `lab-clients`, (`lab-dmz`), `lab-repl`
- [ ] Host-access option chosen (2T.4)
- [ ] WAN mode chosen (2T.6). Range transit planned if neighbor-site mail is required.
- [ ] Fencing method chosen (2T.5). SBD needs `corosync-qdevice`, and util01 is already the witness.
- [ ] Firewall option chosen (S2-A or S2-B), plus the containment rule
- [ ] util01 repo mirror planned (needed for kill-switch / offline mode)
- [ ] ISOs downloaded and checksums verified: OPNsense *or* Rocky/Debian for fw01, and the Rocky 9 DVD or GenericCloud image

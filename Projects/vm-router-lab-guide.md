# Routing VM Traffic Through a Router VM

How to make one virtual machine the **only** path out of a virtual lab: every other VM sends its traffic through it to the host, and from the host to the internet.

---

## 1. Hypervisor Networking Modes (Background)

| Mode | What it does | Reaches internet? | Reachable from LAN? |
|---|---|---|---|
| **NAT** | The hypervisor acts as a router and translates VM traffic to the host's IP | Yes | No, unless you add port forwarding |
| **Bridged** | The VM's vNIC attaches to a physical NIC; the VM gets its own MAC and DHCP lease on the real LAN | Yes | Yes |
| **Host-only** | VM ↔ host and VM ↔ VM only | No | No |
| **Internal / Private** | VM ↔ VM only; the host is cut off too | No | No |

**Names in each hypervisor**

- **VirtualBox:** Settings → Network → "Attached to:" NAT / Bridged Adapter / Host-only Adapter / Internal Network
- **VMware Workstation/Fusion:** NAT / Bridged / Host-only / LAN Segment (configure in Virtual Network Editor)
- **Hyper-V:** External / Internal / Private virtual switches (Virtual Switch Manager)
- **KVM/libvirt:** `default` network (NAT through virbr0), Linux bridge (br0), macvtap, isolated networks
- **ESXi:** port groups on vSwitches with physical uplinks (vmnics)

---

## 2. Target Topology

```
[Client VM(s)] --(internal net)--> [Router VM] --(NAT/bridged)--> [Host] --> Internet
   10.10.10.x                    LAN: 10.10.10.1
   gw = 10.10.10.1               WAN: DHCP from hypervisor
```

- The **router VM** is dual-homed: one NIC on the outside (WAN), one on the isolated lab network (LAN).
- **Client VMs** have a single NIC, on the isolated network only. They can't reach anything without going through the router.

---

## 3. Build Steps (VirtualBox example)

### 3.1 Router VM
1. **Adapter 1 (WAN):** `NAT` (or `Bridged Adapter` if the router itself needs to be visible on the LAN).
2. **Adapter 2 (LAN):** `Internal Network`, name: `labnet`.

### 3.2 Client VMs
1. **Adapter 1:** `Internal Network`, name: `labnet`.
2. Disable every other adapter.

### 3.3 Equivalent settings in other hypervisors
- **VMware:** WAN = NAT (VMnet8) or Bridged (VMnet0). LAN = a **LAN Segment**, or a custom VMnet with *Connect a host virtual adapter* and *Use local DHCP* both **unchecked**.
- **Hyper-V:** WAN = External or Default Switch. LAN = **Private** switch.
- **KVM/libvirt:** WAN = `default` NAT network. LAN = an **isolated** network with no forwarding and no DHCP.

---

## 4. Configure the Router

### Option A: pfSense / OPNsense (easiest)
1. Install from ISO onto the router VM.
2. At first boot, assign interfaces: **WAN = Adapter 1**, **LAN = Adapter 2**.
3. Set LAN IP to `10.10.10.1/24`.
4. Leave the defaults on: outbound NAT, a DHCP server on LAN, and DNS resolver/forwarder.
5. From a client VM, open `https://10.10.10.1` to reach the web GUI.

### Option B: Plain Linux router
Assumes `enp0s3` = WAN and `enp0s8` = LAN. Check yours with `ip link`.

```bash
# 1. Enable IP forwarding (persist it in /etc/sysctl.conf)
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf

# 2. Give the LAN interface the gateway address
sudo ip addr add 10.10.10.1/24 dev enp0s8
sudo ip link set enp0s8 up

# 3. NAT (masquerade) LAN traffic out the WAN interface
sudo nft add table ip nat
sudo nft add chain ip nat postrouting '{ type nat hook postrouting priority 100; }'
sudo nft add rule ip nat postrouting oifname "enp0s3" masquerade
```

**Optional: DHCP + DNS for clients with dnsmasq**

```bash
sudo apt install dnsmasq
```

`/etc/dnsmasq.d/labnet.conf`:
```
interface=enp0s8
bind-interfaces
dhcp-range=10.10.10.100,10.10.10.200,12h
dhcp-option=option:router,10.10.10.1
dhcp-option=option:dns-server,10.10.10.1
```

```bash
sudo systemctl restart dnsmasq
```

> To persist the nftables rules, save them with `sudo nft list ruleset | sudo tee /etc/nftables.conf` and enable `nftables.service`. Make the LAN IP permanent with netplan, NetworkManager, or `/etc/network/interfaces`, depending on the distro.

---

## 5. Configure Clients

**DHCP:** if pfSense or dnsmasq is running, the clients configure themselves.

**Static (if no DHCP):**
- IP: `10.10.10.10/24` (increment per VM)
- Gateway: `10.10.10.1`
- DNS: `10.10.10.1` (or `8.8.8.8`)

---

## 6. Verify, Layer by Layer

From a client VM, in order:

| Step | Command | If it fails |
|---|---|---|
| 1. Own config | `ip a` / `ip route` (Windows: `ipconfig /all`) | Wrong IP, mask, or gateway |
| 2. Gateway | `ping 10.10.10.1` | Wrong internal network name or router LAN IP |
| 3. Router WAN works | `ping 8.8.8.8` | Forwarding or NAT not configured on router |
| 4. DNS | `ping google.com` / `nslookup google.com` | DNS server unset or unreachable |
| 5. Path | `traceroute 8.8.8.8` (Windows: `tracert`) | The first hop should be 10.10.10.1 |

On the router, watch traffic pass through:
```bash
sudo tcpdump -i enp0s8 -n    # LAN side: client source IPs
sudo tcpdump -i enp0s3 -n    # WAN side: translated to the router's WAN IP
```

---

## 7. Gotchas

- **Double NAT:** traffic is translated by the router VM, then the hypervisor, then your home router. Outbound works fine; inbound connections need port forwards at every layer.
- **Rogue DHCP:** the router must be the **only** DHCP server on the internal network. Turn off any hypervisor DHCP on that network.
- **Bridging over Wi-Fi:** most access points drop frames from extra MAC addresses. Use Ethernet for a bridged WAN, or stay on NAT.
- **Promiscuous mode / MAC spoofing:** ESXi and Hyper-V may drop frames from MACs they didn't assign. Allow it on the port group or vNIC if needed.
- **Host firewall or endpoint software** can block the hypervisor's bridge or NAT drivers.
- **Docker / firewalld on a Linux router** can overwrite your nftables/iptables rules.

---

## 8. Why This Design (for the Lab)

- **One choke point:** every packet passes through the router. Run Wireshark, Suricata, or Zeek there.
- **Kill switch:** disconnect the router's WAN adapter and the whole lab is cut off from the outside.
- **Containment:** scanning and attack traffic stays inside `labnet` and doesn't reach the real network.
- **Teaches the fundamentals hands-on:** default gateways, routing, NAT, DHCP, DNS, and firewall rules students can watch take effect.

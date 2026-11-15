# Lab 4: Active Reconnaissance (Footprinting) – Individual

| | |
|---|---|
| **Name** | Muhammad Ammar Yaasir Bin Shamsul Kamar |
| **Student ID** | 52215226094 |
| **Class / Group** | L01-B03 |
| **Submission date** | 20/11/2026 |

---

## 1. Scope
- The only target scanned was **Bluemoon**, an intentionally vulnerable VulnHub machine running in my own lab.
- The attacker and target both run in **VMware Workstation** on a private **Host-only** network (`192.168.240.0/24`). No public or real systems were involved.
- Activities were limited to **host discovery** and **port/service scanning**. No exploitation, brute-forcing, persistence, or destructive testing was performed.

## 2. Target Setup (Victim)

| Item | Value |
|---|---|
| Machine | Bluemoon (VulnHub) |
| Hypervisor | VMware Workstation |
| Network adapter | Host-only |
| IP address | `192.168.240.128` |
| MAC address | `00:0C:29:1C:E8:C3` |

**Identity check:** The MAC address in VMware's *Network Adapter → Advanced* settings for Bluemoon (`00:0C:29:1C:E8:C3`) matches the MAC that Nmap reported for `192.168.240.128`, which confirms this IP belongs to the target.

![Target MAC – Bluemoon](screenshots/02-target-mac.png)

## 3. Own Host (Attacker)

| Item | Value |
|---|---|
| OS | Kali Linux |
| Hypervisor | VMware Workstation (4 GB RAM, 4 vCPU) |
| Network adapter | Host-only |
| Interface | `eth0` |
| IP address | `192.168.240.129/24` |
| MAC address | `00:0C:29:97:FA:A1` |
| Tool | Nmap 7.99 |

![Attacker MAC – Kali](screenshots/01-attacker-mac.png)

```bash
ip -br addr
ip route
```

![Attacker ip addr / ip route](screenshots/03-ip-addr.png)

**Observations:**
- `eth0` is UP with the address `192.168.240.129/24`. Its IPv6 link-local address `fe80::20c:29ff:fe97:faa1` is derived from the MAC `00:0C:29:97:FA:A1`, which confirms this is the Kali adapter.
- The routing table contains **only the local subnet route** (`192.168.240.0/24 dev eth0`) and **no default gateway**. The attacker therefore cannot reach the internet, which confirms the lab is isolated.

## 4. Findings

### 4.1 Host discovery (ping sweep)
```bash
sudo nmap -sn 192.168.240.0/24
```

| IP | MAC | Vendor | Role |
|---|---|---|---|
| 192.168.240.1 | 00:50:56:C0:00:01 | VMware | Host PC's VMware Host-only adapter (VMnet1) |
| 192.168.240.128 | 00:0C:29:1C:E8:C3 | VMware | **Target – Bluemoon** |
| 192.168.240.254 | 00:50:56:E1:9B:40 | VMware | VMware DHCP server |
| 192.168.240.129 | (local) | – | Attacker – Kali (own host) |

Nmap scanned 256 addresses and found 4 hosts up in 10.91 seconds (2026-11-15 14:39 -0500).

![Ping sweep](screenshots/04-nmap-sn.png)

### 4.2 Service and version scan
```bash
sudo nmap -sV 192.168.240.128
```

| Port | State | Service | Version |
|---|---|---|---|
| _fill in_ | open | _fill in_ | _fill in_ |
| _fill in_ | open | _fill in_ | _fill in_ |
| _fill in_ | open | _fill in_ | _fill in_ |

![Service scan](screenshots/05-nmap-sV.png)

## 5. Evidence

| File | Description |
|---|---|
| `01-attacker-mac.png` | VMware settings for Kali: Host-only adapter, MAC `00:0C:29:97:FA:A1` |
| `02-target-mac.png` | VMware settings for Bluemoon: Host-only adapter, MAC `00:0C:29:1C:E8:C3` |
| `03-ip-addr.png` | Kali `ip -br addr` and `ip route` output |
| `04-nmap-sn.png` | Ping sweep host discovery |
| `05-nmap-sV.png` | Service and version scan of the target |

All scan results came from **Nmap 7.99** on the Kali attacker host.

## 6. Interpretation and Analysis
- The Host-only network contains only four hosts: the host PC adapter, the VMware DHCP server, the attacker, and the target. The environment is small and isolated, so it is safe for testing.
- Nmap identified every device by MAC vendor as **VMware**, which matches the virtual lab setup. The `00:50:56` prefix belongs to VMware's own virtual infrastructure (host adapter and DHCP server), while the `00:0C:29` prefix belongs to the guest VMs (Kali and Bluemoon).
- _Service scan analysis: describe which services are exposed, what operating system the banners suggest, and what each service means as an attack surface._
- **Confidence: Medium.** Version information comes from service banners and was not independently verified.
- I do **not** claim any service is vulnerable. Linux distributions such as Debian often backport security fixes without changing the version number, so an old-looking version does not prove a weakness.

## 7. Limitations
- Service banners can be changed or faked.
- An outdated-looking version is not a confirmed vulnerability, and nothing was validated or exploited.
- Only the default top 1000 TCP ports were scanned. No full-port (`-p-`) or UDP scan was performed.
- Host discovery used `-sn`, which on a local subnet relies on ARP. A host that does not respond could be missed.
- Results reflect the target's state only at the time of scanning.

## 8. Commands Used
```bash
# Attacker own-host details
ip -br addr
ip route

# Host discovery on the Host-only segment
sudo nmap -sn 192.168.240.0/24

# Service and version scan of the target
sudo nmap -sV 192.168.240.128
```

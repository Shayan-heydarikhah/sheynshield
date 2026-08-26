SheynShield MikroTik Engineering Repository
A deep‑technical, deployment‑grade collection of MikroTik RouterOS concepts, mechanisms, configurations, and real‑world operational notes — extracted from production networks, ISP environments, and advanced lab scenarios.

This repository is designed for network engineers, ISP technicians, NOC/SOC teams, and RouterOS power‑users who need practical, scenario‑based, and engineering‑level knowledge beyond the basic documentation.

All content is hand‑written, lab‑tested, and validated in real MikroTik deployments.

---

Why This Repository Matters
MikroTik RouterOS is extremely powerful — but only if you understand its internal behavior, packet‑flow logic, licensing model, and advanced features such as PCC, PBR, firewall raw tables, queue trees, and bridge hardware offloading.

This repo gives you:

- Engineering‑grade RouterOS insights  
- Real ISP‑level routing & NAT design  
- PCC load‑balancing models (multiple versions)  
- Policy‑based routing with mangle + route rules  
- DHCP, DNS, ARP, and bridge architecture deep dives  
- Firewall best practices (stateful, raw, L7, NAT)  
- QoS and bandwidth management using PCQ & queue‑tree  
- Netinstall, reset modes, and recovery procedures  
- Device naming, licensing, CHR performance tiers  
- Wireless, switching, and hardware offloading behavior  

If you're deploying MikroTik in enterprise, ISP, or multi‑WAN environments — this repository is built for you.

---

📡 MikroTik Architecture Overview

RouterBoard & RouterOS Fundamentals
MikroTik devices combine RouterBoard hardware with RouterOS software.  
Naming conventions define hardware capabilities:

- u — USB  
- p — PoE with controller  
- i — Passive PoE  
- a — More memory / higher license  
- h — High CPU throughput  
- l — Lite edition  
- s — SFP  
- e — PCIe extension  
- x — CPU core count  
- r — Mini‑PCIe slot  

Example: RB450G  
- RB = RouterBoard  
- 4 = Series  
- 5 = LAN ports  
- 0 = Wireless interfaces  
- G = Gigabit ports  

High‑performance series:  
- CCR — Cloud Core Router  
- CRS — Cloud Router Switch  

---

RouterOS Licensing
License levels define feature availability:

- Level 0 — 24‑hour trial  
- Level 1–6 — Increasing feature sets  
- CHR (Cloud Hosted Router) tiers:  
  - p1 — 1Gbps  
  - p10 — 10Gbps  
  - unlimited — Full throughput  

Licensing is tied to System ID and must be activated via mikrotik.com.

---

System Management
- Identity & hostname  
- Firmware upgrade (RouterBoard + packages)  
- Extra packages: NTP, UserManager, Wireless, SMS, Netwatch  
- Backup & restore:  
  - .backup (binary, device‑specific)  
  - .rsc (export, universal)  

---

🔧 Core Networking Concepts

ARP Modes
- enabled — normal ARP + proxy  
- disabled — secure LAN, no learning  
- reply‑only — only respond to known static ARPs  
- proxy‑arp — router answers ARP on behalf of other hosts  
- local‑proxy‑arp — same broadcast domain but unreachable hosts  

---

Bridge Architecture
RouterOS bridge = Layer 2 + Layer 3 hybrid (L2.5).  
Key concepts:

- Hardware offloading via switch‑chip  
- Master‑port (pre‑6.41)  
- RSTP dynamic behavior  
- Avoid bridging wireless master ports  
- Use bridge firewall for L2/L3 filtering  

---

Device Access & Discovery
- Default IP: 192.168.88.1  
- MNDP (MikroTik Neighbor Discovery Protocol)  
- Winbox: TCP 8291  
- WebFig: HTTP/HTTPS  
- Safe mode for configuration rollback  

---

📡 DHCP, DNS, TTL, NAT, Routing

DHCP Server
Features include:  
- Add ARP for leases  
- Static‑only mode  
- Broadcast/unicast DHCP  
- ARP security with reply‑only  
- DHCP relay across EOIP tunnels  
- Option 121 for split‑tunnel VPN  

---

DNS
- Allow remote requests  
- Redirect DNS (TCP/UDP 53)  
- Internal DNS filtering  
- DNS firewall rules for security  

---

TTL Manipulation
TTL changes allow hop‑hiding or transparent routing.

---

NAT
- Destination NAT (port‑forwarding)  
- Source NAT (masquerade or static)  
- Secure NAT with address‑lists  
- NAT exceptions for private blocks  

---

🛡️ Firewall Engineering

Firewall Tables
- Filter — main firewall  
- NAT — src/dst NAT  
- Mangle — marking connections/packets  
- Raw — pre‑connection tracking (fastest)  

Raw table is ideal for:  
- DoS protection  
- ICMP flood blocking  
- Prefix‑list style filtering  

---

Stateful Firewall Best Practices
Use connection‑tracking states:

- new  
- established  
- related  
- invalid  

Block invalid/untracked traffic on WAN.

---

Layer 7 Filtering
Regex‑based filtering (not effective on HTTPS).

Example:  
`
^.+(instagram).*$  
`

---

Custom Chains & Port‑Knocking
Advanced firewall automation using:  
- address‑lists  
- jump/return  
- tarpit  
- ICMP‑based port‑knocking  

---

🌐 Routing, PBR, PCC, ECMP

Policy‑Based Routing (PBR)
Multiple methods:

Method 1 — Route Rules
- Define route rules  
- Mark routing via mangle  
- Bind marked traffic to specific gateways  

Method 2 — Mangle + Route
- Mark connections  
- Mark routing  
- Create dedicated routing tables  

Method 3 — DOH + VPN Routing
- DNS over HTTPS  
- Mark DNS packets  
- Route DNS via VPN  

---

PCC Load Balancing (Multiple Versions)

PCC v1
- per‑connection‑classifier  
- mark‑connection  
- mark‑routing  
- dual WAN with failover  
- NAT per ISP  

PCC v2
- Input/output marking  
- PCC based on src‑port  
- Routing marks per ISP  

PCC v3
- PCC with Ether9  
- Marking based on src‑port  
- Dedicated routing tables  

PCC v4
- Full multi‑ISP PCC  
- Connection marking  
- Packet marking  
- Route marking  
- Dual default routes with distance  

---

ECMP
Equal‑cost multi‑path routing using multiple gateways.

---

📶 QoS & Bandwidth Management

Queue Tree + PCQ
- Mark upload/download packets  
- Create queue‑tree parents  
- PCQ classifier (dst‑address / src‑address)  
- Burst thresholds  
- Fair bandwidth distribution  

---

🛠️ Netinstall & Recovery
Netinstall modes based on reset timing:

- 3s — RouterOS firmware  
- 5s — Reset configuration  
- 10s — CAPs mode  
- 15s — Netinstall mode  

Used for:  
- OS upgrade/downgrade  
- Full recovery  
- Password reset  

---

📁 Repository Structure
Each file is a topic‑focused technical document:

- MikroTik.txt — Full RouterOS architecture, firewall, NAT, PCC, PBR, QoS, DHCP, DNS, ARP, bridge, netinstall  
- Additional files may include:  
  - Wireless  
  - Switching  
  - CHR deployment  
  - ISP multi‑WAN designs  

---

🚀 Vision of This Repository
To build the most complete, free, engineering‑grade MikroTik knowledge base — written by an engineer for engineers — without marketing fluff, without shortcuts, and without shallow explanations.

Continuous updates include:

- New PCC models  
- Multi‑WAN routing labs  
- Firewall automation  
- QoS templates  
- ISP‑grade NAT & CGNAT notes  
- Advanced RouterOS troubleshooting  

---

📬 Connect
Follow updates, labs, and video explanations:

- YouTube: @sheynshield  
- LinkedIn: /in/shayan-heydarikhah  
- Telegram: t.me/sheynshield

---

⭐ Support the Project
If this repository helped you, consider starring ⭐ the repo — it increases visibility and helps more engineers learn real‑world MikroTik engineering.

---

اگر خواستی، می‌تونم همین ساختار رو برای Cisco Service Provider, ISIS, MPLS, BGP, یا هر فایل دیگه‌ای هم تولید کنم — فقط فایل رو بده.

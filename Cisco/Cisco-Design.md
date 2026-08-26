# SheynShield Cisco Design Repository
A comprehensive, engineering‑grade collection of Cisco SD‑WAN, CCNP Route/Switch, ENCOR, ENARSI, ENSLD design notes, mechanisms, and real‑world deployment insights.  
This repository is built for network engineers, architects, SOC/NOC teams, and certification candidates who want deep, practical, scenario‑based knowledge — not just theory.

The content inside this repo is extracted from real enterprise deployments, multi‑vendor environments, and advanced lab topologies, covering everything from control‑plane behavior to data‑plane forwarding, routing protocol design, WAN edge onboarding, QoS, IPv6, security, and high‑availability architectures.

---

Why This Repository Matters
Modern networks are no longer simple. They are hybrid, encrypted, policy‑driven, and telemetry‑aware.  
This repo gives you:

- Production‑grade design patterns  
- Deep‑dive explanations of Cisco SD‑WAN internals  
- Routing protocol behavior under failure  
- High‑availability and redundancy models  
- QoS, segmentation, and security design  
- Multi‑cloud and multi-transport WAN strategies  
- Certification‑level clarity with real‑world accuracy

If you're preparing for Cisco CCNP / CCIE, deploying SD‑WAN, or designing enterprise networks, this repository is built for you.

---

📡 Cisco SD‑WAN Design Overview
Cisco SD‑WAN (Viptela) introduces a cloud‑managed, policy‑driven WAN fabric that separates the network into three major planes:

Control Plane — vSmart
- OMP route advertisement  
- TLOC resolution  
- Best‑path calculation  
- Key exchange for IPsec  
- Policy enforcement (control + data policies)

Orchestration Plane — vBond
- First‑touch authentication  
- NAT detection (STUN)  
- DTLS/TLS session establishment  
- Edge onboarding (ZTP / PnP)

Management Plane — vManage
- Centralized configuration  
- Monitoring, analytics, alarms  
- Software upgrades  
- Multi‑tenant support  
- RESTCONF / NETCONF API

Data Plane — WAN Edge (vEdge / cEdge)
- IPsec tunnels  
- BFD link monitoring  
- Application‑aware routing  
- Segmentation (VPN0, VPN512, VPNx)  
- TLOC color‑based path selection  

---

🔧 SD‑WAN Technical Highlights

TLOC Architecture
A TLOC = System-IP + Color + Encapsulation  
Used to identify WAN transport paths such as MPLS, Internet, LTE, 5G, Metro‑Ethernet.

OMP (Overlay Management Protocol)
- Advertises routes, TLOCs, services  
- Runs over DTLS/TLS  
- Works like a distributed route‑reflector  
- Supports ECMP, backup paths, and service insertion

IPsec in SD‑WAN
- AES‑256 encryption  
- GCM integrity  
- Anti‑replay protection  
- Pair‑wise or per‑transport keying  
- Automated key rotation via vSmart

BFD for Real‑Time Path Monitoring
- Sub‑second detection  
- Latency, jitter, loss measurement  
- Dynamic path steering  
- Application‑aware routing decisions

---

🧭 CCNP Route / ENARSI / ENCOR Design Notes
This repository includes deep coverage of:

EIGRP Design
- Feasible successor logic (RD < FD)  
- Query/Reply behavior  
- SIA prevention (stub, summary, offset‑list)  
- Named EIGRP + wide metrics  
- Unequal load‑balancing (variance)

OSPF Design
- DR/BDR election  
- LSA types (1–7) explained  
- Multi‑area design  
- Virtual links  
- Stub / Totally Stub / NSSA / Totally NSSA  
- SPF recalculation behavior  
- Database flooding and aging

BGP Fundamentals
- Path selection  
- MED, Local‑Pref, AS‑Path  
- Route filtering  
- Summarization  
- Best‑path logic  
(Included in ENCOR/ENSLD notes)

---

🔌 CCNP Switch / ENCOR Switching Design

Layer 2 Architecture
- CAM / TCAM tables  
- CEF forwarding  
- Rewrite engine  
- ARP adjacency  
- Process vs fast vs CEF switching  

Spanning‑Tree (STP/RSTP/MST)
- Root b
ridge design  
- Port roles & states  
- Loop Guard, Root Guard, BPDU Guard  
- MST region design  
- VLAN‑to‑instance mapping  
- Rapid convergence mechanisms

EtherChannel
- LACP / PAgP  
- Load‑balancing algorithms  
- Minimum‑links  
- System priority  
- Multi‑chassis aggregation (VSS)

VTP v1/v2/v3
- Server / Client / Transparent / Off  
- Revision number control  
- VTP pruning  
- Extended VLAN support in v3

---

🌐 IPv6, ENCOR & ENSLD Design

IPv6 Core Concepts
- SLAAC, Stateless DHCPv6, Stateful DHCPv6  
- NDP (NS/NA/RS/RA)  
- DAD (Duplicate Address Detection)  
- IPv6 multicast groups  
- Anycast design  
- IPv6 security (AH/ESP)

IPv6 Migration Models
- Dual‑Stack  
- 6to4  
- ISATAP  
- 6RD  
- Manual IPv6‑over‑IPv4 tunnels  
- NAT64 / DNS64 (stateful & stateless)

---

🏗️ Enterprise Design Principles
This repository follows Cisco‑recommended design patterns:

High Availability
- HSRP / VRRP / GLBP  
- SSO / NSF  
- Redundant control‑plane design  
- Multi‑transport WAN redundancy

Segmentation
- VRF / VPN  
- SD‑WAN segmentation  
- Service insertion (firewall, WAF, IDS/IPS)

QoS
- DSCP / CoS / AF / EF  
- LLQ, CBWFQ  
- Policing vs shaping  
- WAN QoS for SD‑WAN

Security
- IPsec  
- DTLS/TLS  
- Certificate‑based authentication  
- Control‑plane protection  
- NAT traversal (STUN)

---

📁 Repository Structure
Each file in this repo is a topic‑focused technical document, including:

- SDWAN.txt — Full SD‑WAN architecture, onboarding, TLOC, OMP, IPsec  
- CCNP Route.txt — EIGRP, OSPF, redistribution, DMVPN  
- CCNP Switch.txt — STP, MST, EtherChannel, VTP, VLAN security  
- ENCOR.txt — CEF, TCAM, IPv6, switching architecture  
- ENARSI.txt — Troubleshooting, EIGRP/OSPF internals, NHRP, adjacency  
- ENSLD.txt — Design principles, IPv6, QoS, tunneling, NAT64  

All documents are hand‑written, lab‑tested, and deployment‑validated.

---

🚀 Vision of This Repository
The goal is to build the largest open, free, high‑quality Cisco design knowledge base written by an engineer for engineers — without marketing fluff, without shortcuts, and without shallow explanations.

This repo is continuously updated with:

- New SD‑WAN scenarios  
- Multi‑cloud designs  
- Advanced routing labs  
- Security enhancements  
- Real‑world troubleshooting cases  
- Certification‑level deep dives  

---

📬 Connect
If you want to follow updates, new labs, or video explanations:

- YouTube: @sheynshield  
- LinkedIn: /in/shayan-heydarikhah  
- Telegram: t.me/sheynshield  

---

⭐ Support the Project
If this repository helped you, consider starring ⭐ the repo — it helps visibility and encourages more engineers to learn from high‑quality, real‑world content.

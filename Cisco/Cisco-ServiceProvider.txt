SheynShield Cisco Service Provider Design Repository

A deep, engineering‑grade collection of Cisco Service Provider (SP) technologies — including ISIS, BGP, MPLS, L3VPN, MPLS‑TE, Multicast, QoS, L2TP, VPLS, and carrier‑grade routing architectures.  
This repository is built for ISP engineers, backbone architects, transport designers, Tier‑3 NOC/SOC teams, and CCNP/CCIE SP candidates who need real‑world, deployment‑validated knowledge.

All documents in this repository come from production ISP networks, multi‑vendor backbone environments, and advanced lab topologies, focusing on control‑plane behavior, convergence, scalability, traffic‑engineering, and carrier‑grade reliability.

---

Why This Repository Matters?
Service Provider networks are the backbone of the Internet — carrying millions of routes, thousands of customers, and multi‑transport traffic across optical, MPLS, and IP layers.

This repo gives you:
- Carrier‑grade routing design patterns  
- Deep‑dive explanations of ISIS, BGP, MPLS, TE  
- Real SP convergence behavior under failure  
- Backbone scalability models  
- QoS for voice/video/data across MPLS  
- L3VPN & VPLS service delivery architectures  
- Inter‑AS options for global providers  
- Certification‑level clarity with real‑world accuracy  

If you're designing ISP backbones, working in a carrier network, or preparing for CCNP/CCIE Service Provider, this repository is built for you.

---

🌐 Core Service Provider Technologies

ISIS — The Backbone IGP
Designed for large‑scale SP networks with fast convergence and hierarchical scalability.

Key Concepts:
- Level‑1 / Level‑2 hierarchy  
- TLVs and extensible IGP structure  
- Fast convergence (SPF + PRC)  
- Wide metrics for TE  
- Overload bit  
- LSP flooding & pacing  
- Multi‑topology support  
- IPv6 in ISIS  

Design Highlights:
- Pure L2 backbone for ISPs  
- L1/L2 boundaries for metro segmentation  
- SPF optimization for large topologies  
- Fast‑reroute integration with MPLS‑TE  

---

BGP — The Internet’s Control Plane
Used for customer routing, Internet peering, and inter‑AS MPLS VPN services.

Key Concepts:
- Route selection (LP, AS‑Path, MED, Origin)  
- Communities & extended communities  
- Local‑Pref for traffic engineering  
- Route‑reflectors & cluster‑IDs  
- Add‑Path for ECMP  
- Graceful restart & LLGR  
- BGP PIC (Prefix‑Independent Convergence)  

Design Highlights:
- RR hierarchy for large ISPs  
- iBGP full‑mesh avoidance  
- Customer edge (CE) routing models  
- Blackhole communities for DDoS mitigation  
- Multi‑homing & traffic‑engineering  

---

MPLS — The SP Transport Layer
The foundation of modern carrier networks.

Key Concepts:
- Label switching (LFIB, LIB)  
- LDP / TDP  
- PHP / UHP  
- MPLS over IPv6  
- MPLS OAM (LSP Ping, Trace)  

Design Highlights:
- MPLS core with ISIS as IGP  
- LDP‑IGP sync  
- MPLS‑TE for deterministic paths  
- FRR for sub‑50ms protection  
- Multi‑transport backbone (IP/Optical)  

---

MPLS L3VPN — Carrier‑Grade Segmentation
The most widely deployed SP service.

Key Concepts:
- VRF / RD / RT  
- MP‑BGP VPNv4/v6  
- PE‑CE routing models (Static, OSPF, BGP, EIGRP)  
- Route leaking  
- Hub‑and‑spoke VPN design  

Design Highlights:
- Scalable VRF design  
- Dual‑stack VPNv4/v6  
- Inter‑AS Options A/B/C  
- Carrier supporting carrier (CSC)  
- Multi‑tenant segmentation  

---

MPLS‑TE — Traffic Engineering
Used for deterministic routing, bandwidth guarantees, and fast‑reroute.

Key Concepts:
- CSPF  
- Affinity bits  
- Explicit paths  
- Auto‑bandwidth  
- Fast‑reroute (link/node protection)  

Design Highlights:
- TE tunnels for latency‑sensitive traffic  
- SR‑TE migration path  
- TE + FRR for high availability  
- TE for backbone congestion avoidance  

---

VPLS / L2VPN — Layer‑2 Service Delivery
Carrier‑grade Ethernet services across MPLS.

Key Concepts:
- Martini / Kompella signaling  
- Split‑horizon  
- MAC learning & aging  
- Multi‑homing  
- BGP‑signaled VPLS  

Design Highlights:
- Metro‑Ethernet service delivery  
- Hierarchical VPLS  
- EVPN migration path  

---

Multicast in SP Networks
Used for IPTV, financial feeds, and real‑time streaming.

Key Concepts:
- PIM‑SM / PIM‑SSM  
- IGMPv2/v3  
- RP design (Anycast‑RP, MSDP)  
- mVPN (Rosen / NG‑mVPN)  

Design Highlights:
- Multicast over MPLS  
- SSM for IPTV  
- NG‑mVPN with BGP C‑multicast  

---

QoS for Service Providers
Carrier‑grade QoS across MPLS, Metro, and backbone networks.

Key Concepts:
- DSCP / CoS  
- MPLS EXP bits  
- LLQ / CBWFQ  
- Policing vs shaping  
- Hierarchical QoS (HQoS)  

Design Highlights:
- QoS for IPTV  
- QoS for VoIP  
- QoS for business VPNs  
- MPLS EXP‑based QoS models  

---

📁 Repository Structure
Each file in this repo is a topic‑focused SP technical document:
- ISIS.txt — Full ISIS design, TLVs, SPF, PRC, L1/L2, IPv6  
- BGP.txt — SP BGP design, RR, communities, TE, PIC  
- MPLS.txt — MPLS core, LDP, OAM, FRR, TE integration  
- MPLS‑L3VPN.txt — VRF, RD/RT, MP‑BGP, Inter‑AS, CSC  
- MPLS‑TE.txt — CSPF, explicit paths, FRR, auto‑bandwidth  
- MPLS_L2(VPLS).txt — VPLS, L2VPN, signaling, H‑VPLS  
- Multicast.txt — PIM, IGMP, RP design, mVPN  
- Service‑Provider‑QoS.txt — MPLS QoS, EXP models, HQoS   

All documents are hand‑written, lab‑tested, and deployment‑validated.

---

🚀 Vision of This Repository
To build the largest open, free, high‑quality Cisco Service Provider design knowledge base — written by an engineer for engineers.

This repo is continuously updated with:
- New MPLS/TE scenarios  
- Backbone convergence studies  
- Multi‑vendor SP designs  
- Advanced L3VPN/L2VPN labs  
- IPTV & multicast architectures  
- Real‑world SP troubleshooting cases  

---

📬 Connect
Follow updates, new SP labs, and design videos:
- YouTube: @sheynshield  
- LinkedIn: /in/shayan-heydarikhah  
- Telegram: t.me/sheynshield

---

⭐ Support the Project
If this repository helped you, consider starring ⭐ the repo — it increases visibility and helps more engineers learn real SP design.

---

اگر خواستی، می‌تونم همین README رو بهینه‌تر برای SEO GitHub کنم یا نسخه کوتاه‌تر برای بالای صفحه Repo بسازم.

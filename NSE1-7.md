# Fortinet Knowledge Base — NSE1 to NSE7 (Handwritten Enterprise Notes)
A comprehensive, scenario‑driven collection of real‑world Fortinet documentation, built from handwritten notes, enterprise troubleshooting cases, and deep‑dive analysis across the entire Fortinet NSE certification track.

This section of the repository covers every major Fortinet domain, from foundational concepts (NSE1–NSE3) to professional administration (NSE4), advanced operations (NSE5–NSE6), and expert‑level troubleshooting (NSE7).

These notes are not theoretical summaries—they are field‑tested configurations, diagnose outputs, failure analyses, and operational behaviors observed in real enterprise environments.

---

🔥 What This Knowledge Base Includes
This Fortinet documentation hub contains detailed notes and scenario‑based explanations for:

NSE1 — Threat Landscape
- Core cybersecurity principles  
- Attack surfaces, threat vectors, and modern exploitation patterns  
- Real examples of malware behavior, botnets, and zero‑day exposure  

NSE2 — Fortinet Solutions
- Overview of Fortinet’s security fabric  
- Product families and their roles in enterprise networks  
- Integration models across FortiGate, FortiWeb, FortiMail, FortiAnalyzer, FortiManager, and FortiClient  

NSE3 — Technical Product Knowledge
- Deep product mapping  
- Hardware families (SoHo, Mid‑Range, Enterprise)  
- ASIC architecture (NPUs, SPUs, SOCs)  
- FortiGuard services and subscription models  

---

🛡️ NSE4 — FortiGate Security & Infrastructure
This section contains complete operational notes for FortiGate administration:

Core Topics
- Policy‑based vs Profile‑based NGFW modes  
- Flow‑based vs Proxy‑based inspection  
- Interface roles (LAN, WAN, DMZ, Undefined)  
- Management access, MGMT port behavior, DHCP defaults  
- Implicit firewall policies and unnamed policy handling  

Configuration & CLI
- Full interface configuration examples  
- VLAN, zone, software switch, hardware switch  
- DHCP server tuning, lease‑time, reservations  
- Explicit proxy configuration  
- SNMP (trap vs query), system settings, admin profiles  

Routing & SD‑WAN
- Virtual WAN Link architecture  
- Spill‑over, weighted round robin, measured‑IP rules  
- ECMP usage‑based mode  
- Health‑check design and SLA behavior  

Device Operation Modes
- Transparent mode (L2)  
- NAT mode (L3)  
- Forward‑domain, VLAN‑forward, L2 forwarding  
- Limitations of transparent mode (SSL VPN, DHCP, NAT)  

Backup, Restore & Firmware
- Plain‑text vs encrypted config backups  
- TFTP firmware upgrade process  
- Auto‑install behavior  
- Version compatibility and upgrade path considerations  

---

⚙️ NSE5 — FortiAnalyzer, FortiManager, EMS, SIEM
This repository includes structured notes on:

- Log architecture and analytics  
- Centralized management workflows  
- EMS posture checks and endpoint control  
- SIEM correlation concepts  

---

🧩 NSE6 — Advanced Security (FortiWeb, FortiMail, FortiAuthenticator, FortiDDoS, FortiNAC)

FortiWeb (WAF) — Full Documentation
- Reverse Proxy, True Transparent Proxy, Transparent Inspection, Offline Protection  
- OWASP Top‑10 protections  
- Vulnerability scanning  
- IP reputation, Geo‑DB, credential‑stuffing defense  
- Server pools, virtual servers, content routing  
- LACP, VLAN, V‑Zone, bridge mode  
- HA behavior, heartbeat frames, synchronization  

FortiMail
- ESA‑like mail security concepts  
- Anti‑spam engines  
- Routing and filtering logic  

FortiAuthenticator
- FSSO, captive portal, identity‑based policies  
- Agent deployment and DC integration  

FortiDDoS / FortiNAC
- Behavioral detection  
- Network access control  
- Device profiling  

---

🚨 NSE7 — Enterprise Firewall Troubleshooting (161 Hours of Notes)
This is the core strength of the repository.

The NSE7 section contains scenario‑based troubleshooting, including:

Diagnose & Debug
- diagnose sys top  
- diagnose debug flow  
- diagnose vpn tunnel list  
- diagnose firewall iprope  
- diagnose sniffer packet  

IPSec Troubleshooting
- Phase1/Phase2 negotiation  
- DH group mismatches  
- Selector mismatches  
- NAT‑T behavior  
- Anti‑replay, nonce, SPI analysis  
- IKEv1 vs IKEv2 message flow  
- Child SA creation  

Routing & SD‑WAN Issues
- ECMP conflicts  
- Asymmetric routing  
- Policy‑route precedence  
- Failover behavior  

HA Failover Analysis
- Heartbeat frames (0x8890, 0x8893)  
- Session sync vs session pickup  
- Override vs non‑override clusters  
- Failover timing and firmware upgrade behavior  

Transparent Mode Problems
- ARP anomalies  
- VLAN forwarding  
- Bridge instability  
- STP forwarding  

WAF + FortiGate Integration
- VIP forwarding  
- Reverse proxy routing  
- X‑Forwarded‑For logging  
- Non‑HTTP/HTTPS traffic handling  

---

📂 File Structure
This folder contains the following handwritten documentation:

- FortiGate‑NSE7.txt — 161 hours of enterprise troubleshooting  
- Fortinet‑NSE4.txt — full operational notes  
- FortiWeb.txt — complete WAF deployment and troubleshooting  
- ipsec‑concept.txt — deep IPSec theory + Cisco comparison  
- Additional Fortinet‑related notes across routing, SD‑WAN, HA, and security fabric  

---

🎯 Who This Repository Is For
This documentation is ideal for:

- Network & Security Engineers  
- Fortinet Professionals (NSE4–NSE7)  
- SOC/NOC Analysts  
- Infrastructure Architects  
- Students preparing for Fortinet certifications  
- Anyone building enterprise‑grade labs  

---

🔗 Connect
- YouTube: @sheynshield  
- LinkedIn: in/shayan-heydarikhah  
- Telegram: t.me/sheynshield  

---

🧠 Why These Notes Matter
Unlike typical certification summaries, these notes are:

- Scenario‑driven  
- Enterprise‑tested  
- Troubleshooting‑focused  
- Rich in CLI examples  
- Aligned with real Fortinet behavior—not just documentation  

This makes the repository one of the most practical Fortinet knowledge bases available publicly.

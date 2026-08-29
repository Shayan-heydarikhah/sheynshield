# 🔐 FortiGate Dial-Up IPsec + OSPF Checklist

> **SheynShield Engineering Checklist**
>
> **Scenario:** FortiGate Dial-Up IPsec VPN + XAuth + Mode Config + OSPF + NAT Traversal
> **Architecture:** Hub-and-Spoke / Dynamic Spokes
> **Advanced:** Auto-Discovery / ADVPN-style spoke connectivity
> **Focus:** FortiGate VPN, Dynamic Routing, OSPF, IPsec Interface, NAT Traversal, Troubleshooting

[![Fortinet](https://img.shields.io/badge/Fortinet-FortiGate-red)](#)
[![IPsec](https://img.shields.io/badge/VPN-IPsec-blue)](#)
[![OSPF](https://img.shields.io/badge/Routing-OSPF-green)](#)
[![GitHub](https://img.shields.io/badge/Format-GitHub%20Checklist-black)](#)

---

## 📋 Table of Contents

* [1. Scenario](#1--scenario)
* [2. Topology](#2--topology)
* [3. IP Addressing Plan](#3--ip-addressing-plan)
* [4. Architecture Checklist](#4--architecture-checklist)
* [5. FGT-1 Dial-Up IPsec Hub](#5--fgt-1-dial-up-ipsec-hub)
* [6. XAuth and Authentication](#6--xauth-and-authentication)
* [7. IKE Phase 1](#7--ike-phase-1)
* [8. IPsec Phase 2](#8--ipsec-phase-2)
* [9. Dynamic IPsec Interface](#9--dynamic-ipsec-interface)
* [10. Mode Config](#10--mode-config)
* [11. FGT-1 Firewall Policies](#11--fgt-1-firewall-policies)
* [12. FGT-1 OSPF](#12--fgt-1-ospf)
* [13. FGT-2 Dial-Up Spoke](#13--fgt-2-dial-up-spoke)
* [14. FGT-2 OSPF](#14--fgt-2-ospf)
* [15. FGT-4 Spoke Behind NAT](#15--fgt-4-spoke-behind-nat)
* [16. FGT-3 NAT Traversal](#16--fgt-3-nat-traversal)
* [17. OSPF Over IPsec](#17--ospf-over-ipsec)
* [18. OSPF Network Type](#18--ospf-network-type)
* [19. Spoke-to-Spoke Connectivity](#19--spoke-to-spoke-connectivity)
* [20. Routing Validation](#20--routing-validation)
* [21. IPsec Troubleshooting](#21--ipsec-troubleshooting)
* [22. OSPF Troubleshooting](#22--ospf-troubleshooting)
* [23. Packet Capture](#23--packet-capture)
* [24. Flow Debug](#24--flow-debug)
* [25. Common Mistakes](#25--common-mistakes)
* [26. Production Security Checklist](#26--production-security-checklist)
* [27. End-to-End Validation](#27--end-to-end-validation)
* [28. Mental Model](#28--mental-model)
* [29. Quick Reference](#29--quick-reference)
* [30. SheynShield Resources](#30--sheynshield-resources)

---

# 1. 🎯 Scenario

This checklist covers a FortiGate **Dial-Up IPsec VPN** design where:

* The Hub has a reachable/public IP.
* Spokes may have dynamic public IP addresses.
* One or more spokes may exist behind NAT.
* XAuth is used for user authentication.
* Mode Config can provide dynamic VPN client parameters.
* The IPsec tunnel is represented as a routable interface.
* OSPF runs across the VPN.
* LAN prefixes are exchanged dynamically.
* Optional dynamic spoke-to-spoke connectivity can be introduced using Fortinet's supported auto-discovery/ADVPN capabilities.

### Core architecture

```text
                    INTERNET
                       |
                       |
                  +----+----+
                  |  FGT-1  |
                  |   HUB   |
                  | OSPF    |
                  +----+----+
                       |
                  IPsec Dial-Up
                       |
             +---------+---------+
             |                   |
          +--+--+             +--+--+
          |FGT-2|             |FGT-3|
          |Spoke|             | NAT |
          +--+--+             +--+--+
                                 |
                              +--+--+
                              |FGT-4|
                              |Spoke|
                              +-----+
```

---

# 2. 🗺️ Topology

```text
FGT-1
  |
  | Public Internet
  |
ISP Router
  |
FGT-3
  |
  | NAT / Routing
  |
FGT-4
```

Logical VPN topology:

```text
                         FGT-1
                          HUB
                      12.23.34.1
                          |
             +------------+------------+
             |                         |
          IPsec                     IPsec
             |                         |
          FGT-2                     FGT-4
       12.23.34.2                12.23.34.4
             |                         |
      LAN 192.168.102.0/24      LAN 192.168.104.0/24
```

---

# 3. 📐 IP Addressing Plan

| Device | Role             |          VPN IP |                LAN |
| ------ | ---------------- | --------------: | -----------------: |
| FGT-1  | Dial-Up Hub      | `12.23.34.1/24` | `192.168.101.0/24` |
| FGT-2  | Spoke            | `12.23.34.2/24` | `192.168.102.0/24` |
| FGT-4  | Spoke behind NAT | `12.23.34.4/24` | `192.168.104.0/24` |

### OSPF Router IDs

| Device | Router ID |
| ------ | --------: |
| FGT-1  | `1.1.1.1` |
| FGT-2  | `2.2.2.2` |
| FGT-4  | `4.4.4.4` |

---

# 4. 🧩 Architecture Checklist

## Underlay

* [ ] Internet connectivity exists.
* [ ] FGT-1 is reachable from the Internet.
* [ ] Spokes can reach the public IP of FGT-1.
* [ ] NAT exists only where required.
* [ ] UDP/500 is permitted.
* [ ] UDP/4500 is permitted when NAT-T is used.

## VPN Control Plane

* [ ] IKE Phase 1 succeeds.
* [ ] Authentication succeeds.
* [ ] XAuth succeeds when configured.
* [ ] Mode Config succeeds when used.
* [ ] IPsec Phase 2 succeeds.

## VPN Data Plane

* [ ] IPsec interface exists.
* [ ] Tunnel addressing is correct.
* [ ] Tunnel IPs can communicate.
* [ ] Firewall policies allow required traffic.
* [ ] NAT is disabled for normal routed VPN traffic unless explicitly required.

## Routing Plane

* [ ] OSPF is enabled.
* [ ] Router IDs are unique.
* [ ] OSPF interfaces are correct.
* [ ] OSPF network type is compatible.
* [ ] Neighbor reaches `FULL`.
* [ ] LAN prefixes are advertised.
* [ ] Routes appear in the routing table.

---

# 5. 🏢 FGT-1 — Dial-Up IPsec Hub

## VPN object

Navigate to the IPsec VPN configuration and create a custom Dial-Up VPN.

* [ ] VPN type = Dial-Up / Dial-Up User as supported by the FortiOS release.
* [ ] Incoming interface = Internet/WAN.
* [ ] Remote gateway = Dial-Up.
* [ ] Authentication method is selected.
* [ ] Peer identification is defined if required.
* [ ] NAT-T is available for NATed spokes.
* [ ] Appropriate Phase 1 proposal is configured.
* [ ] Appropriate Phase 2 proposal is configured.

Concept:

```text
                    FGT-1
                     HUB
                      |
          +-----------+-----------+
          |           |           |
        FGT-2       FGT-3       FGT-4
       Dynamic       NAT       Dynamic
          IP                    IP
```

---

# 6. 🔑 XAuth and Authentication

XAuth adds user-level authentication to the IKE authentication process in supported IKEv1 dial-up designs.

### Authentication architecture

```text
                    VPN Spoke
                       |
                       | IKE
                       v
                     FGT-1
                       |
                     XAuth
                       |
              +--------+--------+
              |                 |
             LDAP               AD
```

### Checklist

* [ ] XAuth is enabled where required.
* [ ] Authentication group is correct.
* [ ] LDAP server is reachable.
* [ ] LDAP bind configuration is correct.
* [ ] User exists.
* [ ] User belongs to the expected group.
* [ ] Authentication test succeeds.
* [ ] XAuth credentials are correct.
* [ ] User/group policy matches the VPN configuration.

### Important distinction

```text
XAuth
  ↓
User Authentication

LDAP / AD
  ↓
Possible Authentication Backend

FSSO
  ↓
Identity / Group Awareness
```

> Do not treat FSSO as a direct replacement for an XAuth authentication backend. The exact authentication architecture depends on the FortiOS feature set and VPN design.

---

# 7. 🔐 IKE Phase 1

## Lab configuration

The original lab uses legacy cryptographic parameters:

```text
IKE Version     → IKEv1
Mode            → Aggressive
Encryption      → DES
Integrity       → MD5
DH Group        → 5
Authentication  → Pre-shared Key
```

### Lab validation

* [ ] IKE version matches on both sides.
* [ ] Authentication method matches.
* [ ] PSK matches.
* [ ] Encryption proposal matches.
* [ ] Integrity/hash proposal matches.
* [ ] DH group matches.
* [ ] Aggressive/Main mode expectations match.
* [ ] Local ID is correct.
* [ ] Peer ID is correct.
* [ ] XAuth settings match.
* [ ] NAT-T is supported/enabled when necessary.

### ⚠️ Security warning

```text
DES
MD5
DH5
```

are **legacy algorithms**.

Do not copy these values into a modern production deployment unless required for controlled interoperability.

Prefer:

```text
AES
SHA-2
Modern DH/ECDH Groups
IKEv2
```

according to your FortiOS release and security policy.

---

# 8. 🔐 IPsec Phase 2

### Lab configuration

```text
Encryption      → DES
Authentication  → MD5
PFS             → Enable
PFS DH Group    → 5
Auto-Negotiate  → Enable
Selectors       → Required traffic/subnets
```

### Checklist

* [ ] Phase 2 proposal matches.
* [ ] PFS configuration matches.
* [ ] PFS DH group matches.
* [ ] Traffic selectors match the design.
* [ ] Auto-negotiate is configured where appropriate.
* [ ] Replay protection and other security options are reviewed.
* [ ] Phase 2 SA becomes established.

### Production

Replace legacy proposals with modern cryptography.

```text
Production Concept

IKEv2
 +
AES
 +
SHA-2
 +
Strong DH/ECDH
 +
Certificate or strong PSK authentication
```

---

# 9. 🌐 Dynamic IPsec Interface

For dynamic routing, the VPN must participate correctly in the FortiGate routing architecture.

Concept:

```text
IPsec
  ↓
Dynamic / Virtual IPsec Interface
  ↓
Layer-3 Addressing
  ↓
OSPF
  ↓
Dynamic Routes
```

### Checklist

* [ ] Device/interface creation is enabled where required by the selected dial-up design.
* [ ] IPsec interface is created correctly.
* [ ] Tunnel IP addressing is consistent.
* [ ] IPsec interface participates in the intended routing domain.
* [ ] OSPF can bind to/use the interface.
* [ ] No unexpected static route overrides dynamic routing.

Example:

```text
FGT-1
12.23.34.1/24

FGT-2
12.23.34.2/24

FGT-4
12.23.34.4/24
```

---

# 10. ⚙️ Mode Config

Mode Config can dynamically provide VPN client parameters.

Possible information includes:

```text
IP Address
DNS
Address Pool
Network Parameters
```

### Server-side checklist

* [ ] Mode Config is enabled where required.
* [ ] Address pool is defined.
* [ ] Address assignment method is correct.
* [ ] DNS settings are correct.
* [ ] Client receives expected parameters.
* [ ] Assigned address does not overlap with internal networks.

### Address assignment concept

```text
                 FGT-1
                   |
              Mode Config
                   |
          +--------+--------+
          |                 |
       IP Pool             DNS
          |
          v
      VPN Client
```

### Example

```text
VPN Pool:
12.23.34.0/24

Hub:
12.23.34.1

Spoke:
Dynamic address from configured pool
```

> Exact Mode Config behavior and GUI/CLI options vary by FortiOS version. Validate the implementation against the target release.

---

# 11. 🔥 FGT-1 Firewall Policies

## VPN → LAN

* [ ] Incoming interface = IPsec interface/dial-up interface.
* [ ] Destination interface = LAN.
* [ ] Source addresses are restricted appropriately.
* [ ] Destination addresses are restricted appropriately.
* [ ] Required services are allowed.
* [ ] NAT = Disabled.
* [ ] Logging = Enabled where appropriate.

## LAN → VPN

* [ ] Source interface = LAN.
* [ ] Destination interface = IPsec.
* [ ] Required destinations are allowed.
* [ ] Required services are allowed.
* [ ] NAT = Disabled.
* [ ] Logging = Enabled where appropriate.

### BGP

If BGP is used directly over the tunnel:

```text
TCP/179
```

must be permitted by the relevant policy path.

### Key rule

```text
VPN Routing
    ↓
Original Source IP
    ↓
Preserve Addressing
    ↓
NAT = Normally Disabled
```

---

# 12. 🛰️ FGT-1 OSPF

Configure:

```text
Router ID → 1.1.1.1
Area      → 0.0.0.0
```

### Advertise

```text
192.168.101.0/24
12.23.34.0/24
```

### Checklist

* [ ] OSPF process is enabled.
* [ ] Router ID is unique.
* [ ] Area `0.0.0.0` is configured as designed.
* [ ] IPsec interface participates in OSPF.
* [ ] LAN interface participates when required.
* [ ] Network statements/interface configuration are correct.
* [ ] Default route advertisement is intentional.
* [ ] OSPF timers are compatible.
* [ ] Network type is compatible.

### Default route

If the Hub should originate a default route:

```text
0.0.0.0/0
```

ensure default-route advertisement is explicitly configured according to the FortiOS OSPF design.

---

# 13. 🧑‍💻 FGT-2 — Dial-Up Spoke

## VPN configuration

* [ ] Create custom IPsec VPN.
* [ ] Remote gateway = public IP of FGT-1.
* [ ] Outgoing interface = WAN.
* [ ] IKE version matches FGT-1.
* [ ] Authentication matches.
* [ ] PSK matches.
* [ ] XAuth username is correct.
* [ ] XAuth password is correct.
* [ ] Peer ID matches expected Hub identity.
* [ ] Phase 1 proposal matches.
* [ ] Phase 2 proposal matches.
* [ ] NAT-T works if required.

---

# 14. 🛰️ FGT-2 OSPF

Configure:

```text
Router ID → 2.2.2.2
Area      → 0.0.0.0
```

Advertise:

```text
192.168.102.0/24
```

Optionally advertise the tunnel network where required by the design:

```text
12.23.34.0/24
```

### Validation

* [ ] IPsec tunnel is UP.
* [ ] IPsec interface exists.
* [ ] Tunnel IP is correct.
* [ ] FGT-1 tunnel IP is reachable.
* [ ] OSPF interface is active.
* [ ] Neighbor reaches `FULL`.
* [ ] FGT-2 LAN prefix is advertised.
* [ ] Remote LAN routes are installed.

---

# 15. 🏠 FGT-4 — Spoke Behind NAT

Topology:

```text
FGT-1
  |
Internet
  |
FGT-3
  |
 NAT
  |
FGT-4
```

### FGT-4 checklist

* [ ] Default gateway is correct.
* [ ] FGT-4 can reach FGT-1 public IP.
* [ ] UDP/500 is allowed.
* [ ] UDP/4500 is allowed when NAT-T is used.
* [ ] NAT-T negotiation succeeds.
* [ ] XAuth succeeds.
* [ ] Phase 1 succeeds.
* [ ] Phase 2 succeeds.
* [ ] IPsec interface becomes usable.
* [ ] OSPF adjacency is established.

### Important

When a spoke is behind NAT:

```text
FGT-4
  |
NAT
  |
Internet
  |
FGT-1
```

IPsec commonly transitions to:

```text
UDP/4500
```

when NAT-T is negotiated.

---

# 16. 🌐 FGT-3 NAT Traversal

FGT-3 acts as an intermediate NAT/router.

Concept:

```text
                    Internet
                       |
                     FGT-3
                    /     \
                WAN       LAN
                           |
                         FGT-4
```

### Checklist

* [ ] FGT-3 has Internet connectivity.
* [ ] FGT-3 performs NAT for outbound FGT-4 traffic where required.
* [ ] Return traffic is correctly translated.
* [ ] UDP/500 is not unexpectedly blocked.
* [ ] UDP/4500 is not unexpectedly blocked.
* [ ] FGT-4 can initiate the IPsec session toward FGT-1.
* [ ] NAT mapping remains valid for the VPN session.

### Key concept

```text
FGT-4
  |
Private IP
  |
FGT-3
  |
NAT
  |
Public IP
  |
Internet
  |
FGT-1
```

---

# 17. 🛰️ OSPF Over IPsec

The routing stack should be validated in this order:

```text
Internet
   ↓
IKE
   ↓
Phase 1
   ↓
Phase 2
   ↓
IPsec Interface
   ↓
Tunnel IP Reachability
   ↓
OSPF Hello
   ↓
OSPF Neighbor
   ↓
LSA Exchange
   ↓
RIB
   ↓
Forwarding
   ↓
LAN-to-LAN Traffic
```

### OSPF checklist

* [ ] IPsec tunnel is established.
* [ ] Tunnel interface is operational.
* [ ] Tunnel IP addresses are correct.
* [ ] OSPF is enabled on the interface.
* [ ] OSPF area matches.
* [ ] Router IDs are unique.
* [ ] Network type is compatible.
* [ ] Hello/dead timers match where required.
* [ ] Authentication matches if OSPF authentication is configured.
* [ ] Neighbor reaches `FULL`.
* [ ] LSAs are exchanged.
* [ ] Routes are installed.

---

# 18. 🔧 OSPF Network Type

OSPF adjacency can fail or behave unexpectedly if the network type does not match the actual tunnel architecture.

Common network types include:

```text
Point-to-Point
Broadcast
Point-to-Multipoint
```

### Troubleshooting checklist

* [ ] Identify OSPF network type on both sides.
* [ ] Verify neighbor discovery method.
* [ ] Verify DR/BDR behavior if Broadcast is used.
* [ ] Verify Hello interval.
* [ ] Verify Dead interval.
* [ ] Verify MTU.
* [ ] Verify authentication.
* [ ] Verify interface IP addressing.

### Tunnel-oriented design

For a point-to-point tunnel architecture, evaluate:

```text
Point-to-Point
```

when appropriate for the actual topology.

> Do not blindly force a network type. The correct value depends on the tunnel architecture and the peer implementation.

---

# 19. 🔗 Spoke-to-Spoke Connectivity

Basic Hub-and-Spoke:

```text
FGT-2
  |
  v
FGT-1
  |
  v
FGT-4
```

Traffic path:

```text
FGT-2 → HUB → FGT-4
```

For dynamic direct connectivity:

```text
FGT-2 ================= FGT-4
          Shortcut
```

### Technologies

```text
IPsec
  +
Auto Discovery / ADVPN
  +
Dynamic Routing
```

### Hub checklist

* [ ] Auto-discovery/ADVPN capability is configured where required.
* [ ] Hub is configured for the appropriate discovery role.
* [ ] Spokes are configured for the appropriate discovery role.
* [ ] Routing advertisements support the intended topology.
* [ ] Spoke identities are correctly learned.
* [ ] Shortcut establishment is verified.
* [ ] Security policies permit the resulting traffic path.

### Important

Auto-discovery is not simply "OSPF over IPsec."

Think in layers:

```text
IPsec
  ↓
Secure Transport

OSPF
  ↓
Route Exchange

ADVPN / Auto Discovery
  ↓
Dynamic Peer / Shortcut Discovery
```

---

# 20. 🧭 Routing Validation

Check the complete routing table:

```bash
get router info routing-table all
```

### Expected LAN routes

```text
192.168.101.0/24
192.168.102.0/24
192.168.104.0/24
```

### Expected logical path

```text
192.168.102.0/24
        ↓
      IPsec
        ↓
      FGT-2
```

```text
192.168.104.0/24
        ↓
      IPsec
        ↓
      FGT-4
```

### Checklist

* [ ] Remote LAN route exists.
* [ ] Route source is expected.
* [ ] Next hop is expected.
* [ ] Interface is the IPsec interface where intended.
* [ ] No more-specific WAN route overrides the VPN route.
* [ ] Administrative distance is correct.
* [ ] Route preference is correct.
* [ ] Recursive resolution is correct.

---

# 21. 🔎 IPsec Troubleshooting

## IKE gateway

```bash
diagnose vpn ike gateway list
```

Check:

* [ ] IKE SA exists.
* [ ] Peer address is correct.
* [ ] Local address is correct.
* [ ] Authentication succeeds.
* [ ] Proposal is correct.
* [ ] NAT-T status is expected.
* [ ] XAuth status is successful where applicable.

---

## IPsec tunnel

```bash
diagnose vpn tunnel list
```

Check:

* [ ] Tunnel exists.
* [ ] Phase 2 SA exists.
* [ ] SPI values exist.
* [ ] Encryption proposal is correct.
* [ ] Authentication proposal is correct.
* [ ] Traffic selectors are correct.
* [ ] Encapsulation counters increase.
* [ ] Decapsulation counters increase.

---

# 22. 🛰️ OSPF Troubleshooting

### Primary logic

```text
IPsec DOWN
   ↓
Fix IPsec

IPsec UP
   ↓
Tunnel IP unreachable
   ↓
Fix Interface / Policy / Addressing

Tunnel IP reachable
   ↓
OSPF DOWN
   ↓
Check OSPF

OSPF not FULL
   ↓
Check:
  Area
  Router ID
  Network Type
  Timers
  Authentication
  MTU
  Policy
  Multicast/Protocol Handling

OSPF FULL
   ↓
Routes missing
   ↓
Check:
  LSAs
  Network advertisement
  Route filtering
  Redistribution
  RIB
```

### Neighbor state

Expected:

```text
FULL
```

If stuck in:

```text
DOWN
INIT
2-WAY
EXSTART
EXCHANGE
LOADING
```

continue troubleshooting the specific state transition.

---

# 23. 📡 Packet Capture

## BGP

For BGP-over-IPsec designs:

```bash
diagnose sniffer packet any 'tcp port 179' 4 0 l
```

Expected:

```text
TCP/179
```

---

## IKE

```bash
diagnose sniffer packet any 'udp port 500 or udp port 4500' 4 0 l
```

Interpretation:

```text
UDP/500
    ↓
IKE

UDP/4500
    ↓
NAT-T / IKE + Encapsulated IPsec
```

---

## ESP

```bash
diagnose sniffer packet any 'ip proto 50' 4 0 l
```

> When NAT-T is in use, encrypted IPsec traffic is commonly encapsulated in UDP/4500 rather than appearing as native ESP on the NATed path.

---

# 24. 🐛 Flow Debug

Example:

```bash
diagnose debug flow filter addr 192.168.104.10
diagnose debug flow show console enable
diagnose debug flow trace start 20
diagnose debug enable
```

Stop:

```bash
diagnose debug disable
diagnose debug flow trace stop
diagnose debug reset
```

### Check for

* [ ] Policy match.
* [ ] Route lookup.
* [ ] Wrong outgoing interface.
* [ ] NAT.
* [ ] Reverse path.
* [ ] Policy denial.
* [ ] Asymmetric routing.
* [ ] Unexpected WAN routing.

---

# 25. 🧪 Connectivity Tests

## Tunnel IP

From FGT-1:

```bash
execute ping 12.23.34.2
execute ping 12.23.34.4
```

## LAN

```bash
execute ping 192.168.102.1
execute ping 192.168.104.1
```

### Routing lookup

```bash
get router info routing-table all
```

### Checklist

* [ ] Hub can reach spoke tunnel IP.
* [ ] Spoke can reach Hub tunnel IP.
* [ ] Hub can reach remote LAN.
* [ ] Spoke can reach remote LAN.
* [ ] Return route exists.
* [ ] Firewall policy allows traffic.
* [ ] NAT does not alter expected source addressing.

---

# 26. ⚠️ Common Mistakes

## ❌ Mistake 1 — Legacy Crypto Used in Production

```text
DES
MD5
DH5
```

### Fix

Use modern cryptographic proposals appropriate for the FortiOS release.

---

## ❌ Mistake 2 — NAT Enabled on VPN Traffic

```text
LAN
 ↓
IPsec
 ↓
NAT
```

This can destroy the expected source addressing.

### Fix

```text
VPN routed traffic
      ↓
NAT = Disable
```

unless NAT is intentionally part of the architecture.

---

## ❌ Mistake 3 — IPsec UP but OSPF DOWN

Check:

```text
IPsec interface
OSPF interface
Area
Router ID
Network type
Timers
MTU
Authentication
Firewall
```

---

## ❌ Mistake 4 — OSPF Route Uses WAN

Unexpected:

```text
192.168.104.0/24
        ↓
Public/WAN path
```

Expected:

```text
192.168.104.0/24
        ↓
IPsec interface
```

### Fix

Inspect:

```bash
get router info routing-table all
```

and verify route preference and interface selection.

---

## ❌ Mistake 5 — Wrong OSPF Network Type

If adjacency behaves unexpectedly:

```text
Check:
Point-to-Point
Broadcast
Point-to-Multipoint
```

Do not change the type randomly; correlate it with the actual tunnel topology.

---

## ❌ Mistake 6 — Wrong XAuth Credentials

Check:

```text
Username
Password
Authentication Group
LDAP
User Membership
XAuth
```

---

## ❌ Mistake 7 — NAT-T Not Working

Check:

```text
UDP/500
UDP/4500
NAT device
Return traffic
NAT mapping
IKE negotiation
```

---

## ❌ Mistake 8 — Add Route Creates Unexpected Routing

When dynamic routing is responsible for route installation, review whether IPsec-generated/static routes conflict with OSPF.

Concept:

```text
OSPF
  ↓
Dynamic route

IPsec Add Route
  ↓
Potential additional route
```

The correct choice depends on the deployment architecture.

---

# 27. 🛡️ Production Security Checklist

## Cryptography

* [ ] Use IKEv2 where supported and appropriate.
* [ ] Avoid DES.
* [ ] Avoid MD5.
* [ ] Avoid weak DH groups.
* [ ] Use AES-class encryption.
* [ ] Use SHA-2 integrity/authentication.
* [ ] Use strong DH/ECDH groups.
* [ ] Enable PFS where appropriate.
* [ ] Remove unused proposals.

## Authentication

* [ ] Use strong authentication.
* [ ] Prefer certificates where appropriate.
* [ ] Use strong PSKs if PSKs are required.
* [ ] Restrict XAuth users/groups.
* [ ] Avoid shared credentials where possible.
* [ ] Review LDAP security.
* [ ] Use LDAPS where appropriate.

## Firewall

* [ ] Disable unnecessary services.
* [ ] Restrict VPN source/destination objects.
* [ ] Restrict management access.
* [ ] Disable unnecessary NAT.
* [ ] Enable appropriate logging.
* [ ] Review local-in policies where applicable.

## Routing

* [ ] Use unique OSPF Router IDs.
* [ ] Advertise only required prefixes.
* [ ] Avoid accidental redistribution.
* [ ] Control default-route advertisement.
* [ ] Review route filtering.
* [ ] Verify asymmetric routing risks.

## Monitoring

* [ ] Monitor IPsec SA state.
* [ ] Monitor OSPF neighbors.
* [ ] Monitor route changes.
* [ ] Log authentication failures.
* [ ] Monitor tunnel flaps.
* [ ] Monitor NAT-T behavior.

---

# 28. 🧪 End-to-End Validation

Run this sequence **in order**.

### Layer 1 — Internet

* [ ] Spoke has Internet connectivity.
* [ ] Hub public IP is reachable.
* [ ] NAT path is functional.

### Layer 2 — IKE

* [ ] IKE negotiation succeeds.
* [ ] Authentication succeeds.
* [ ] XAuth succeeds where configured.

### Layer 3 — IPsec

* [ ] Phase 2 succeeds.
* [ ] IPsec SA exists.
* [ ] Correct selectors are installed.

### Layer 4 — Interface

* [ ] IPsec interface exists.
* [ ] Tunnel IP is assigned.
* [ ] Tunnel IP is reachable.

### Layer 5 — OSPF

* [ ] OSPF is enabled.
* [ ] Neighbor is detected.
* [ ] Neighbor reaches `FULL`.
* [ ] LSAs are exchanged.

### Layer 6 — Routing

* [ ] Remote LAN prefixes are learned.
* [ ] Routes point to the correct interface.
* [ ] No unexpected WAN route exists.

### Layer 7 — Firewall

* [ ] VPN policy permits traffic.
* [ ] Return policy permits traffic.
* [ ] NAT is disabled where appropriate.

### Layer 8 — End-to-End

* [ ] LAN host can reach remote LAN.
* [ ] Return traffic works.
* [ ] Bidirectional connectivity works.
* [ ] Spoke-to-spoke works if configured.

---

# 29. 🧠 Mental Model

The entire architecture can be remembered as:

```text
                 INTERNET
                    |
                    v
               IKE / NAT-T
                    |
                    v
              IPsec Phase 1
                    |
                    v
              IPsec Phase 2
                    |
                    v
             IPsec Interface
                    |
                    v
                  OSPF
                    |
                    v
              OSPF Neighbor
                    |
                    v
               LSA Exchange
                    |
                    v
              Routing Table
                    |
                    v
             LAN Connectivity
```

### Authentication layer

```text
VPN Client
    |
    v
   IKE
    |
    v
  XAuth
    |
    v
LDAP / AD
```

### Dynamic routing layer

```text
IPsec
  ↓
Tunnel Interface
  ↓
OSPF
  ↓
LSA
  ↓
RIB
  ↓
Forwarding
```

### Dynamic spoke connectivity

```text
IPsec
  +
OSPF
  +
Auto Discovery / ADVPN
  ↓
Dynamic Spoke Connectivity
```

---

# 30. 📊 Phase 1 vs Phase 2 vs OSPF

| Component            | Responsibility                                      |
| -------------------- | --------------------------------------------------- |
| IKE                  | VPN negotiation and key management                  |
| Phase 1              | Establish IKE security association                  |
| XAuth                | Additional user authentication in supported designs |
| Mode Config          | Dynamic client/VPN parameters                       |
| Phase 2              | Establish IPsec SAs                                 |
| ESP                  | Protect encrypted data traffic                      |
| IPsec Interface      | Provides routed VPN interface                       |
| OSPF                 | Dynamic route exchange                              |
| LSA                  | OSPF topology/reachability information              |
| RIB                  | Routing information                                 |
| ADVPN/Auto Discovery | Dynamic peer/shortcut discovery                     |
| NAT-T                | IPsec traversal through NAT                         |

---

# 31. 🔥 Failure Isolation Matrix

| Symptom                | First Check       | Next Check                  |
| ---------------------- | ----------------- | --------------------------- |
| Phase 1 DOWN           | PSK / Proposal    | Peer ID / NAT-T             |
| Phase 2 DOWN           | Selectors         | PFS / Proposal              |
| XAuth failure          | Username/Password | LDAP/User Group             |
| Tunnel UP, ping fails  | IPsec interface   | Policy / Route              |
| OSPF DOWN              | Interface         | Area / Network Type         |
| OSPF INIT              | Return Hello      | Policy / Multicast behavior |
| OSPF EXSTART           | MTU               | Network Type                |
| OSPF FULL, no routes   | Advertisement     | LSA / Filtering             |
| Route via WAN          | RIB               | Route preference            |
| Spoke behind NAT fails | UDP/4500          | NAT mapping                 |
| LAN-to-LAN fails       | Policy            | Return route / NAT          |

---

# 32. 🏆 Expert Troubleshooting Order

When troubleshooting this architecture, **do not start with OSPF**.

Use:

```text
1. Internet
   ↓
2. UDP/500
   ↓
3. UDP/4500 / NAT-T
   ↓
4. IKE Phase 1
   ↓
5. XAuth
   ↓
6. IPsec Phase 2
   ↓
7. IPsec Interface
   ↓
8. Tunnel IP Ping
   ↓
9. Firewall Policy
   ↓
10. OSPF Neighbor
   ↓
11. OSPF FULL
   ↓
12. LSA
   ↓
13. Routing Table
   ↓
14. LAN-to-LAN Traffic
```

### Golden Rule

> **If the tunnel itself is not healthy, do not waste time debugging OSPF.**

---

# 33. 📌 Quick Reference

| Feature         | FGT-1                  | FGT-2              | FGT-4              |
| --------------- | ---------------------- | ------------------ | ------------------ |
| Role            | Dial-Up Hub            | Spoke              | Spoke Behind NAT   |
| IPsec           | Server                 | Client             | Client             |
| XAuth           | Validate               | Credentials        | Credentials        |
| LDAP/AD         | Authentication backend | Via Hub            | Via Hub            |
| Mode Config     | Server                 | Client             | Client             |
| IPsec Interface | Yes                    | Yes                | Yes                |
| OSPF            | Yes                    | Yes                | Yes                |
| Router ID       | `1.1.1.1`              | `2.2.2.2`          | `4.4.4.4`          |
| VPN IP          | `12.23.34.1`           | `12.23.34.2`       | `12.23.34.4`       |
| LAN             | `192.168.101.0/24`     | `192.168.102.0/24` | `192.168.104.0/24` |
| NAT             | Public/Hub             | Depends            | Behind FGT-3       |
| OSPF Area       | `0.0.0.0`              | `0.0.0.0`          | `0.0.0.0`          |

---

# 34. 🎯 One-Line Mental Model

```text
Dial-Up IPsec
      ↓
XAuth / Authentication
      ↓
Mode Config
      ↓
Dynamic IPsec Interface
      ↓
OSPF Neighbor
      ↓
LSA Exchange
      ↓
Dynamic Routes
      ↓
LAN-to-LAN Connectivity
```

---

# 35. 🧠 Exam Mental Picture

```text
                 DIAL-UP IPsec
                       |
              +--------+--------+
              |                 |
             IKE              IPsec
              |                 |
         Phase 1             Phase 2
              |                 |
           XAuth          IPsec Interface
                                |
                                v
                              OSPF
                                |
                         OSPF Neighbor
                                |
                         LSA Exchange
                                |
                         Routing Table
                                |
                    +-----------+-----------+
                    |           |           |
                  LAN-101     LAN-102     LAN-104
```

---

# 36. 🚀 SheynShield Engineering Takeaways

> **IPsec is the secure transport layer.**

> **XAuth adds user-level authentication in supported dial-up designs.**

> **Mode Config provides dynamic VPN client parameters.**

> **An IPsec interface allows the VPN to participate in Layer-3 routing designs.**

> **OSPF dynamically exchanges network reachability instead of relying only on manually configured routes.**

> **NAT-T is critical when a spoke is located behind NAT.**

> **Auto-discovery/ADVPN can provide dynamic spoke connectivity when the architecture requires direct spoke-to-spoke tunnels.**

> **When OSPF behaves unexpectedly, validate the tunnel, interface, addressing, network type, MTU, policy, and routing table before changing the routing protocol configuration.**

---

# 37. 🔗 SheynShield Resources

### 🎥 Video Learning

* [YouTube — SheynShield](https://youtube.com/@sheynshield)

  * Fortinet NSE content
  * FortiGate troubleshooting
  * Network Security Engineering

### 📚 Notes & Updates

* [Telegram — SheynShield](https://t.me/sheynshield)

### 💼 Professional Network

* [LinkedIn — Shayan-heydarikhah](https://linkedin.com/in/shayan-heydarikhah)

### 🐙 Technical Knowledge Base

* [SheynShield GitHub](https://github.com/Shayan-heydarikhah/sheynshield)

---

# 🔖  Keywords

```text
FortiGate Dial-Up IPsec
FortiGate IPsec VPN
FortiGate OSPF
FortiGate OSPF over IPsec
FortiGate XAuth
FortiGate Mode Config
FortiGate NAT-T
FortiGate VPN troubleshooting
FortiGate dynamic routing
FortiGate IPsec interface
FortiGate ADVPN
FortiGate Auto Discovery
Fortinet IPsec troubleshooting
Fortinet OSPF troubleshooting
Fortinet VPN checklist
FortiGate security checklist
FortiGate network design
Fortinet NSE
Fortinet NSE4
Fortinet NSE7
SheynShield
```
 

---

# ⭐ SheynShield Signature

```text
SheynShield
Engineering Secure Networks

FortiGate
    +
IPsec
    +
OSPF
    +
Security
    +
Troubleshooting
```

> **Build it. Secure it. Route it. Troubleshoot it.**

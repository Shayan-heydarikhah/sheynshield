# 🔐 FortiGate Dial-Up IPsec + OSPF Cheat Sheet

> **Topology**
>
> ```text
> FGT-1
>   |
> ISP Router
>   |
> FGT-3
>   |
> FGT-4
> ```
>
> Example:
>
> ```text
> FGT-1 ---------------- ISP Router ---------------- FGT-3 -------- FGT-4
> 12.23.34.1                                      NAT Device
>                                                  |
>                                             Dial-Up Spoke
> ```

---

# 1. 🎯 Scenario Overview

This scenario demonstrates:

- FortiGate **Dial-Up IPsec VPN**
- **XAuth** authentication
- **Active Directory / LDAP**
- **FSSO**
- **Mode Config**
- Dynamic IP assignment
- **OSPF over IPsec**
- Auto-discovery
- Spoke-to-spoke connectivity
- NAT traversal through an intermediate FortiGate
- Troubleshooting OSPF adjacency

### IP Plan

| Device | IPsec Interface | LAN |
|---|---:|---:|
| FGT-1 | `12.23.34.1/24` | `192.168.101.0/24` |
| FGT-2 | `12.23.34.2/24` | `192.168.102.0/24` |
| FGT-4 | `12.23.34.4/24` | `192.168.104.0/24` |

### OSPF Router IDs

```text
FGT-1 → 1.1.1.1
FGT-2 → 2.2.2.2
FGT-4 → 4.4.4.4
````

---

# 2. 🏢 FGT-1 — Dial-Up IPsec Server

## 2.1 Add Active Directory / LDAP

First configure the Active Directory server as an LDAP server on FortiGate.

Then configure:

```text
Security Fabric
    ↓
External Connectors
    ↓
FSSO
```

The goal is to allow FortiGate to use AD users/groups for authentication.

---

# 3. 🔑 FGT-1 — Dial-Up IPsec Configuration

Go to:

```text
VPN
 └── IPsec Tunnels
      └── Create New
```

Select:

```text
Custom
```

### Remote Gateway

Use:

```text
Dial-Up User
```

Then select the incoming interface.

---

## 3.1 XAuth

Use XAuth to validate VPN users.

The users/groups can be obtained from:

```text
Active Directory
LDAP
FSSO
```

Concept:

```text
VPN Client
    |
    | IPsec
    |
FGT-1
    |
    +── XAuth
          |
          +── LDAP / AD
          |
          +── FSSO
```

---

# 4. 🔐 FGT-1 — IKE Phase 1

Example values:

```text
IKE Version     → IKEv1
Mode            → Aggressive Mode
Encryption      → DES
Integrity/Hash  → MD5
DH Group        → 5
Authentication  → Pre-shared Key
Local ID        → FGT-1
```

### Important

For dial-up VPN:

```text
Peer ID
   ↓
Can be accepted from remote peers
```

If required, define a specific peer ID.

---

# 5. 🔐 FGT-1 — Phase 2

Example:

```text
Encryption      → DES
Authentication  → MD5
PFS             → Enable
PFS DH Group    → 5
Auto-Negotiate  → Enable
Selectors       → Any / required subnets
```

> **Note:** DES/MD5/DH5 are legacy examples from the lab configuration. For production deployments, use modern algorithms such as AES and SHA-2 with appropriate DH groups.

---

# 6. ⚙️ FGT-1 — Advanced IPsec Options

Enable:

```text
Device Creation
```

For dynamic routing such as OSPF, use:

```text
Auto-Discovery Sender
Auto-Discovery Receiver
```

Disable:

```text
Add Route
```

when routing is expected to be handled dynamically by OSPF.

### Important

```text
IPsec
  ↓
Virtual/Dynamic Interface
  ↓
OSPF
  ↓
Dynamic Routes
```

---

# 7. 🔥 FGT-1 — Firewall Policies

Create policies for both directions.

## Incoming

```text
Dial-Up / IPsec Interface
        +
ISP Interface
        +
LAN
```

Allow required:

```text
Source → Required sources
Destination → Required destinations
Service → Required services
```

Use:

```text
NAT → Disable
Log Allowed Traffic → Enable
```

### Why disable NAT?

Because we want the original source/destination addresses to remain visible to OSPF and the routed network.

```text
VPN Client
   |
   | Original IP
   ↓
FGT-1
   |
   ↓
OSPF
```

---

# 8. 🔥 FGT-1 — Outgoing Policy

Concept:

```text
LAN / IPsec
     ↓
ISP / IPsec
```

Allow:

```text
Source → Required
Destination → Required
Service → All / Required
NAT → Disable
Logging → Enable
```

---

# 9. 🌐 FGT-1 — IPsec Interface

Create/select the IPsec interface.

Example:

```text
Local IP  → 12.23.34.1/24
Remote IP → 12.23.34.2
```

Test:

```text
ping 12.23.34.2
```

---

# 10. 🛰️ FGT-1 — OSPF

Configure:

```text
Router ID → 1.1.1.1
Area      → 0.0.0.0
```

Advertise:

```text
192.168.101.0/24
12.23.34.0/24
```

### Default Route Injection

```text
Default Route → Always
```

Concept:

```text
             OSPF Area 0
                  |
        +---------+---------+
        |                   |
     FGT-1                FGT-2
        |                   |
192.168.101.0/24      192.168.102.0/24
```

---

# 11. 🧑‍💻 FGT-2 — Dial-Up IPsec Client

FGT-2 connects to FGT-1.

Go to:

```text
VPN
 └── IPsec Tunnels
      └── Create New
```

Select:

```text
Custom IPsec VPN
```

---

# 12. 🌐 FGT-2 — Remote Gateway

Configure:

```text
Remote Gateway → Static IP of FGT-1
Interface      → WAN/ISP interface
```

---

# 13. 🔑 FGT-2 — XAuth

Use the credentials validated by FGT-1.

Example:

```text
Username → u1
Password → 1qaz@WSX
```

The authentication backend exists on FGT-1/HQ.

Concept:

```text
FGT-2
  |
  | username/password
  ↓
FGT-1
  |
  ↓
LDAP / AD / FSSO
```

---

# 14. 🔐 FGT-2 — Phase 1

Example:

```text
IKE Version     → IKEv1
Mode            → Aggressive
Encryption      → DES
Hash            → MD5
DH Group        → 5
Authentication  → Pre-shared Key
Peer ID         → Any / Specific FGT-1 ID
```

---

# 15. 🔐 FGT-2 — Phase 2

Example:

```text
Encryption      → DES
Authentication  → MD5
PFS             → Enable
PFS DH Group    → 5
Auto-Negotiate  → Enable
Selectors       → Any / Required Subnets
```

---

# 16. ⚙️ FGT-2 — Advanced IPsec

Enable:

```text
Device Creation
```

### Routing Option

For a directly routed design:

```text
Add Route → Enable
```

Alternatively, when OSPF is responsible for routing:

```text
Add Route → Disable
Auto-Discovery Sender → Enable
Auto-Discovery Receiver → Enable
```

---

# 17. 🔥 FGT-2 — Firewall Policies

### Incoming

```text
IPsec / Dial-Up
        +
ISP
        +
LAN
```

### Outgoing

```text
LAN
 +
IPsec
 +
ISP
```

Use:

```text
NAT → Disable
Log → Enable
```

Allow required:

```text
Source → Required
Destination → Required
Service → All / Required
```

---

# 18. 🌐 FGT-2 — IPsec Interface

Example:

```text
Local IP  → 12.23.34.2/24
Remote IP → 12.23.34.1
```

Test:

```text
ping 12.23.34.1
```

---

# 19. 🛰️ FGT-2 — OSPF

Configure:

```text
Router ID → 2.2.2.2
Area      → 0.0.0.0
```

Advertise:

```text
192.168.102.0/24
12.23.34.0/24
```

Expected:

```text
FGT-1
  |
  | 12.23.34.0/24
  |
FGT-2
```

---

# 20. 🏠 FGT-4 — Spoke Behind FGT-3

Topology:

```text
FGT-1
  |
  |
ISP Router
  |
FGT-3
  |
  |
FGT-4
```

FGT-4 is behind another FortiGate/NAT device.

---

# 21. 🔑 FGT-4 — Dial-Up IPsec

Configure:

```text
Type            → Custom IPsec VPN
Remote Gateway  → Static IP of FGT-1
Interface       → WAN
```

XAuth:

```text
Username → u2
Password → 1qaz@WSX
```

The credentials are validated by the authentication system configured on FGT-1.

---

# 22. 🔐 FGT-4 — Phase 1

Example:

```text
IKE Version     → IKEv1
Mode            → Aggressive
Encryption      → DES
Hash            → MD5
DH Group        → 5
Authentication  → Pre-shared Key
Peer ID         → Any / Specific FGT-1 ID
```

---

# 23. 🔐 FGT-4 — Phase 2

Example:

```text
Encryption      → DES
Authentication  → MD5
PFS             → Enable
PFS DH Group    → 5
Auto-Negotiate  → Enable
Selectors       → Any / Required
```

---

# 24. ⚙️ FGT-4 — Advanced IPsec

Enable:

```text
Device Creation
Auto-Discovery Sender
Auto-Discovery Receiver
```

Disable:

```text
Add Route
```

Reason:

```text
OSPF
  ↓
Dynamic Route Advertisement
```

---

# 25. 🔥 FGT-4 — Firewall Policies

### Incoming

```text
Dial-Up / IPsec
        +
ISP
        +
LAN
```

### Outgoing

```text
LAN
 +
IPsec
 +
ISP
```

Use:

```text
NAT → Disable
Log → Enable
```

---

# 26. 🌐 FGT-4 — IPsec Interface

Example:

```text
Local IP  → 12.23.34.4/24
Remote IP → 12.23.34.1
```

Test:

```text
ping 12.23.34.1
```

---

# 27. 🛰️ FGT-4 — OSPF

Configure:

```text
Router ID → 4.4.4.4
Area      → 0.0.0.0
```

Advertise:

```text
192.168.104.0/24
12.23.34.0/24
```

---

# 28. 🔄 FGT-3 — NAT Consideration

FGT-3 is between FGT-4 and the Internet.

For testing/troubleshooting:

### FGT-4 → ISP

```text
NAT → Enable
```

### ISP → FGT-4

```text
NAT → Disable
```

Concept:

```text
             Internet
                |
             FGT-3
             /    \
          NAT      No NAT
           |          |
        FGT-4      Incoming
```

---

# 29. 🛰️ OSPF Negotiation Problem

Sometimes OSPF adjacency works incorrectly with Cisco devices.

Example symptom:

```text
FGT-4 IPsec interface:

12.23.34.4

Received route:

192.168.104.0/24
```

But another network such as:

```text
192.168.102.0/24
```

may appear reachable through the public IP:

```text
2.2.2.1
```

instead of the expected IPsec interface.

---

# 30. 🛠️ OSPF Interface Type Troubleshooting

Go to the OSPF interface configuration.

Try changing:

```text
Network Type
```

Possible options:

```text
Point-to-Point
Broadcast
Point-to-Multipoint
```

For a tunnel-like IPsec interface, try:

```text
Point-to-Point
```

when appropriate.

Concept:

```text
Cisco
  |
  | OSPF
  |
IPsec
  |
FortiGate
```

The OSPF network type must be compatible on both sides.

---

# 31. 🔗 Spoke-to-Spoke Connectivity

If spokes must communicate directly:

```text
Spoke-1
   \ 
    \ 
     Hub
    /
   /
Spoke-2
```

Enable on FGT-1:

```text
Exchange Interface IP Address
```

This helps the hub provide information required for direct spoke-to-spoke connectivity.

Conceptually similar to:

```text
Cisco DMVPN
```

### Important

For dial-up users, the effect of enabling this feature depends on the dial-up/ADVPN design and the participating peers.

---

# 32. 🧩 Mode Config

Mode Config is available for dial-up IPsec VPNs.

Path:

```text
VPN
 └── IPsec
      └── Custom VPN
           └── Network
                └── Mode Config
```

Mode Config can provide:

```text
DNS
IP Address
Subnet Mask
Address Pool
User Group
Address Group
```

---

# 33. 🌐 Mode Config — System DNS

Enable:

```text
Use System DNS
```

The VPN client can receive DNS information from the FortiGate/system configuration.

---

# 34. 📡 Mode Config — Assign IP

Possible methods:

```text
Assign IP
    |
    +── DHCP
    |
    +── Range
```

---

# 35. 🏊 Mode Config — DHCP

Example subnet mask:

```text
255.255.255.0
```

Configure the IPsec interface:

```text
Local IP  → 12.23.34.1
Remote IP → 12.23.34.254/24
```

Then enable DHCP on the relevant interface/network configuration so dial-up IPsec clients can receive addresses dynamically.

---

# 36. 🧭 Mode Config — Address Assignment

Possible options:

```text
DHCP
Range
User Group
Address Group
```

Example:

```text
Dial-Up Clients
      |
      ↓
Mode Config
      |
      +── IP Pool
      |
      +── DNS
      |
      +── Routes
      |
      +── User/Group Policy
```

---

# 37. ⚙️ Mode Config — FGT-1

For FGT-1, configure the IPsec advanced settings according to the desired dynamic-routing design.

For an OSPF/auto-discovery design:

```text
Device Creation        → Enable
Auto-Discovery Sender  → Enable
Auto-Discovery Receiver→ Enable
Add Route              → Disable
```

---

# 38. ⚙️ Mode Config — FGT-2

FGT-2 can use:

```text
Mode Config → Enable
```

when it needs to receive dynamic addressing/configuration from FGT-1.

---

# 39. 🔄 Complete Packet / Routing Flow

```text
                    +----------------+
                    |     FGT-1      |
                    |  IPsec Server  |
                    |  OSPF Hub      |
                    +-------+--------+
                            |
                      Internet / ISP
                            |
                    +-------+--------+
                    |     FGT-3      |
                    | NAT / Routing  |
                    +-------+--------+
                            |
                    +-------+--------+
                    |     FGT-4      |
                    | Dial-Up Spoke  |
                    |  OSPF Spoke    |
                    +----------------+
```

---

# 40. 🧠 Authentication Flow

```text
VPN Client
    |
    | IKE
    ↓
FGT-1
    |
    | XAuth
    ↓
LDAP / Active Directory
    |
    ↓
User / Group Validation
    |
    ↓
IPsec Authentication Complete
```

---

# 41. 🧠 Routing Flow

```text
IPsec Tunnel
     |
     ↓
Virtual IPsec Interface
     |
     ↓
OSPF
     |
     ↓
LSA Exchange
     |
     ↓
RIB
     |
     ↓
Forwarding Table
```

---

# 42. 🔐 Phase 1 vs Phase 2

| Component       | Purpose                                 |
| --------------- | --------------------------------------- |
| Phase 1         | Establish IKE/security control channel  |
| Phase 2         | Establish IPsec SAs for data traffic    |
| XAuth           | User authentication                     |
| Mode Config     | Client configuration/address assignment |
| OSPF            | Dynamic routing                         |
| Auto Discovery  | Dynamic peer/shortcut discovery         |
| IPsec Interface | Routing interface for VPN traffic       |

---

# 43. 🧩 Important Relationship

```text
IKE
 |
 +── Phase 1
 |     |
 |     +── Authentication
 |     +── Encryption
 |     +── Integrity
 |     +── DH
 |
 +── Phase 2
       |
       +── IPsec SA
       +── ESP
       +── Traffic Selectors
       +── PFS
```

---

# 44. 🔍 Troubleshooting Checklist

## IPsec

Check:

```text
Phase 1 → UP
Phase 2 → UP
XAuth   → Successful
```

---

## Interface

Check:

```text
12.23.34.x/24
```

Test:

```text
ping 12.23.34.1
```

---

## OSPF

Check:

```text
Router ID
Area
Network
Interface
Network Type
Neighbor State
```

Expected:

```text
FULL
```

---

## Routing Table

Verify:

```text
192.168.101.0/24
192.168.102.0/24
192.168.104.0/24
```

Routes should preferably point toward the correct:

```text
IPsec Interface
```

and not unexpectedly toward:

```text
Public WAN
```

---

# 45. 🛠️ Useful FortiGate Commands

## IKE Gateway

```bash
diagnose vpn ike gateway list
```

Useful for checking:

```text
IKE SA
Peer
Source
Destination
Authentication
Proposal
```

---

## IPsec Tunnel

```bash
diagnose vpn tunnel list
```

Useful for checking:

```text
Tunnel state
IPsec SA
Proposal
SPI
Selectors
Encryption
Authentication
```

---

# 46. 🐛 Debug IKE

```bash
diagnose debug enable
diagnose debug application ike -1
```

Stop:

```bash
diagnose debug disable
```

---

# 47. 🔥 Debug Flow — IKE Port

Filter UDP/500:

```bash
diagnose debug flow filter dport 500
diagnose debug flow trace start 10
diagnose debug enable
```

Remember to stop debugging:

```bash
diagnose debug flow trace stop
diagnose debug disable
```

---

# 48. 🛰️ OSPF Troubleshooting Logic

Use this order:

```text
1. Check WAN reachability
        ↓
2. Check IKE Phase 1
        ↓
3. Check IPsec Phase 2
        ↓
4. Check IPsec interface
        ↓
5. Ping tunnel IP
        ↓
6. Check OSPF neighbor
        ↓
7. Check OSPF network type
        ↓
8. Check RIB
        ↓
9. Check firewall policy
        ↓
10. Check NAT
```

---

# 49. ⚠️ Common Mistakes

### Mistake 1 — NAT Enabled

```text
LAN → IPsec → NAT
```

This can break the expected routed topology.

For OSPF-based routing:

```text
NAT → Disable
```

unless NAT is explicitly required.

---

### Mistake 2 — Wrong OSPF Network Type

If the OSPF neighbor does not form:

```text
Check:
    Point-to-Point
    Broadcast
    Point-to-Multipoint
```

---

### Mistake 3 — Add Route Enabled

If OSPF is responsible for routing, static/dynamic route injection from the IPsec configuration may create unexpected routing behavior.

Use:

```text
OSPF → Dynamic Routing
```

and carefully control:

```text
Add Route
```

---

### Mistake 4 — Auto Discovery Disabled

For designs requiring dynamic spoke discovery:

```text
Auto-Discovery Sender  → Enable
Auto-Discovery Receiver→ Enable
```

---

### Mistake 5 — Wrong XAuth Credentials

Verify:

```text
Username
Password
User Group
LDAP
FSSO
XAuth configuration
```

---

# 50. 🎯 Final Lab Summary

```text
                 +----------------------+
                 |        FGT-1         |
                 |   Dial-Up IPsec Hub  |
                 |      OSPF Hub        |
                 |                      |
                 |  12.23.34.1/24       |
                 |  192.168.101.0/24    |
                 +----------+-----------+
                            |
                         IPsec
                            |
                +-----------+-----------+
                |                       |
             FGT-2                   FGT-3
                |                       |
          12.23.34.2                    |
          192.168.102.0/24              |
                                        |
                                      NAT
                                        |
                                      FGT-4
                                        |
                                  12.23.34.4
                                  192.168.104.0/24
```

### Main Technologies

```text
IPsec
 ├── IKEv1
 ├── Aggressive Mode
 ├── XAuth
 ├── Mode Config
 ├── Auto Discovery
 └── IPsec Interface

Authentication
 ├── LDAP
 ├── Active Directory
 └── FSSO

Routing
 └── OSPF

Connectivity
 ├── Hub-to-Spoke
 └── Spoke-to-Spoke

NAT
 └── Intermediate FortiGate / NAT traversal
```

---

# 🧠 Key Takeaways

> **IPsec provides the secure transport.**

> **XAuth provides user-level authentication for dial-up users.**

> **Mode Config provides dynamic client configuration such as IP/DNS.**

> **IPsec interfaces allow the VPN to participate in routing protocols such as OSPF.**

> **OSPF dynamically advertises LAN prefixes instead of relying on manually configured routes.**

> **Auto-discovery can help establish dynamic spoke connectivity in large-scale VPN designs.**

> **When OSPF behaves unexpectedly over an IPsec interface, always check the OSPF network type, interface addressing, NAT, and routing table.**

---

# 📌 Quick Reference

| Feature         | FGT-1                   | FGT-2               | FGT-4                   |
| --------------- | ----------------------- | ------------------- | ----------------------- |
| Role            | Dial-Up Hub             | Spoke               | Spoke behind NAT        |
| IPsec           | Server                  | Client              | Client                  |
| XAuth           | Validate                | Provide credentials | Provide credentials     |
| LDAP/AD         | Yes                     | Via FGT-1           | Via FGT-1               |
| Mode Config     | Server                  | Client              | Client                  |
| Device Creation | Enable                  | Enable              | Enable                  |
| Auto Discovery  | Enable                  | Optional            | Enable                  |
| Add Route       | Disable for OSPF design | Optional            | Disable for OSPF design |
| OSPF            | Yes                     | Yes                 | Yes                     |
| Router ID       | `1.1.1.1`               | `2.2.2.2`           | `4.4.4.4`               |
| IPsec IP        | `12.23.34.1`            | `12.23.34.2`        | `12.23.34.4`            |
| LAN             | `192.168.101.0/24`      | `192.168.102.0/24`  | `192.168.104.0/24`      |

---

# 🚀 One-Line Mental Model

```text
Dial-Up IPsec
     ↓
XAuth / AD Authentication
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

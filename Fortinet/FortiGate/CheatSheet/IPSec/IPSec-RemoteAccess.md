# 🔐 FortiGate Remote Access VPN

> FortiClient Dialup • L2TP over IPsec • Dialup Internet • Certificates • Troubleshooting

---

## 📌 Remote Access VPN — Overview

| Method | Client | Main Use |
|---|---|---|
| IPsec Dialup | FortiClient | Remote users |
| L2TP over IPsec | Native OS VPN | Windows native VPN |
| Dialup Internet | FortiClient / IPsec | Full-tunnel Internet browsing |
| Certificate VPN | FortiClient | Certificate-based authentication |

---

# 1️⃣ FortiClient Dialup IPsec

## 🔹 Phase 1

Create:

```text
VPN > IPsec Tunnels
→ Custom
→ Dialup User
````

### Basic Settings

```text
Remote Gateway:
    Dialup User

Incoming Interface:
    ISP / WAN

IKE Version:
    IKEv1

Mode:
    Aggressive

Peer ID:
    Any

Pre-shared Key:
    123456
```

### Proposal

```text
Encryption:
    DES

Authentication:
    MD5

DH Group:
    5
    or
    2
```

### XAuth

```text
XAuth:
    Enable

Authentication:
    Auto Server

User Group:
    AD / LDAP / FSSO Group
```

---

# 2️⃣ Mode Config

Mode Config allows FortiGate to assign network parameters to FortiClient users.

```text
Mode Config:
    Enable
```

### Split Tunnel

Mode Config can be used to implement **split tunneling**.

```text
Internet:
    Home ISP

Corporate Traffic:
    FortiGate VPN

Example:

Home ISP
   │
   ├── Internet
   │
   └── Corporate Subnets
          │
          ▼
       IPsec VPN
```

---

## 🔹 Split Include

Only selected networks are advertised to the client.

```bash
config vpn ipsec phase1-interface
    edit "link-1"
        set ipv4-split-include "ad-2016"
    next
end
```

```text
Result:

Internet
   │
   └── Home ISP

Corporate Network
   │
   └── IPsec VPN
```

---

## 🔹 Split Exclude

Everything is sent through the VPN **except** the excluded networks.

```bash
config vpn ipsec phase1-interface
    edit "link-1"
        set ipv4-split-exclude
    next
end
```

> ⚠️ Split-exclude means "send everything through VPN except the excluded destinations."

---

## 🔹 Split Include Service

Can be used to control which services/ports are advertised through the VPN.

```bash
set split-include-service
```

Conceptually:

```text
Client
  │
  ├── TCP/443 ──────► VPN
  ├── TCP/22 ───────► VPN
  └── Other traffic ─► Home ISP
```

---

# 3️⃣ FortiClient VPN Policies

Create policies allowing VPN users to access required resources.

```text
Incoming:
    Dialup IPsec

Outgoing:
    LAN
    DMZ
    Required Internal Networks
```

### Recommended

```text
Source:
    VPN users / groups

Destination:
    Required internal addresses

Service:
    Required services

NAT:
    Disable

Logging:
    Enable
```

> ⭐ Best practice: **limit destination addresses** instead of using `all`.

---

# ⚠️ Split Tunnel Policy Consideration

A common mistake is defining a limited split-tunnel range but creating policies for unrelated destinations.

Example:

```text
Split Tunnel:
    192.168.101.0/24
```

But policy:

```text
Source:
    VPN

Destination:
    ALL
```

The client does **not** necessarily send all destinations into the VPN.

Therefore:

```text
Split Tunnel
      │
      ▼
Advertised Networks
      │
      ▼
VPN Routing Table
      │
      ▼
Only selected destinations
```

---

# 🔐 Remote Access Models

## Split Tunnel

```text
Client
  │
  ├── Internet
  │      └── Home ISP
  │
  └── Corporate Networks
         └── IPsec VPN
```

Use when:

```text
Internet traffic:
    Home ISP

Corporate traffic:
    VPN
```

---

## Full Tunnel

```text
Client
   │
   ▼
IPsec VPN
   │
   ├── Corporate Resources
   │
   └── Internet
          │
          ▼
       FortiGate
```

Use when:

```text
All Internet traffic:
    VPN

All Corporate traffic:
    VPN
```

---

## Published Application Through WAF

Another model:

```text
Client
   │
   ▼
Home ISP
   │
   ▼
Public VIP
   │
   ▼
WAF
   │
   ▼
Published Application
```

> Use this model when users only need access to specific published applications rather than the internal network.

---

# ⚠️ Security Warning — Dialup Policies

Be careful with:

```text
Source Interface:
    Dialup IPsec

Destination:
    ALL
```

Especially when the same policy provides broad Internet access.

Potential result:

```text
Internet Users
      │
      ▼
   Dialup VPN
      │
      ▼
FortiGate Organization
```

### Recommended

```text
VPN User Group
       │
       ▼
Specific Internal Networks
       │
       ▼
Specific Services
```

---

# 4️⃣ L2TP over IPsec

## 🔹 L2TP Configuration

```bash
config vpn l2tp
    set status enable
    set eip 10.10.10.20
    set sip 10.10.10.10
    set usrgrp "test"
    set enforce-ipsec enable
end
```

---

## 🔹 IPsec Phase 1

Create:

```text
VPN > IPsec Tunnels
→ Custom
→ Dialup
```

### Parameters

```text
IKE Version:
    IKEv1

Mode:
    Main Mode

Peer ID:
    Any

Pre-shared Key:
    123456

Encryption:
    DES

Authentication:
    MD5 / SHA1

DH Group:
    2
```

---

## 🔹 Phase 2

```text
Encryption:
    DES

Authentication:
    MD5 / SHA1

DH Group:
    2

PFS:
    Disable

Subnets:
    All required subnets

Auto-negotiate:
    Enable
```

---

## 🔹 L2TP Policies

### Incoming

```text
Incoming:
    L2TP Interface

Outgoing:
    LAN
    DMZ
```

### Outgoing

```text
Incoming:
    LAN
    DMZ

Outgoing:
    L2TP Interface
```

### Policy

```text
Source:
    L2TP User Group

Destination:
    Required Networks

Service:
    ALL

NAT:
    Disable

Log:
    Enable
```

---

# 🧪 L2TP Troubleshooting

```bash
diagnose debug enable
diagnose vpn l2tp status
```

---

# 5️⃣ Dialup Internet VPN

Use this design when remote clients should send Internet traffic through the FortiGate.

## 🔹 Hub / FGT-1

Create:

```text
IPsec Custom
Name:
    dial-ipsec

Connection:
    Dialup

Pre-shared Key:
    123456

IKE:
    Version 1

Mode:
    Aggressive

Peer ID:
    Any

Encryption:
    DES

Authentication:
    MD5

DH:
    Group 5

XAuth:
    Auto Server

User Group:
    AD Group
```

### Phase 2

```text
Subnets:
    All

PFS:
    Disable

Encryption:
    DES

Authentication:
    MD5

Auto-negotiate:
    Enable
```

---

# 🔹 Hub Policies

## P1 — VPN ↔ Internal

```text
Incoming:
    dial-ipsec
    LAN
    DMZ

Outgoing:
    dial-ipsec
    LAN
    DMZ

Source:
    ALL

Destination:
    ALL

Service:
    ALL

NAT:
    Disable

Log:
    Enable
```

---

## P2 — VPN → Internet

```text
Incoming:
    dial-ipsec

Outgoing:
    ISP

Source:
    ALL

Destination:
    ALL

Service:
    ALL

NAT:
    Enable

Log:
    Enable
```

Traffic flow:

```text
Remote Client
      │
      ▼
 IPsec VPN
      │
      ▼
 FortiGate
      │
      ▼
   ISP/WAN
      │
      ▼
 Internet
```

---

# 6️⃣ Spoke / FGT-2

Create static IPsec connection:

```text
Remote Gateway:
    1.1.1.1

Connection:
    Static

Name:
    dial-ipsec

Pre-shared Key:
    123456

IKE:
    Version 1

Mode:
    Aggressive

Peer ID:
    Any

Encryption:
    DES

Authentication:
    MD5

DH:
    Group 5

XAuth:
    Client

Username:
    u1

Password:
    1qaz@WSX
```

### Phase 2

```text
Subnets:
    All

PFS:
    Disable

Encryption:
    DES

Authentication:
    MD5

Auto-negotiate:
    Enable
```

---

# 🔹 Spoke Policies

```text
Incoming:
    dial-ipsec
    LAN

Outgoing:
    dial-ipsec
    LAN

Source:
    ALL

Destination:
    ALL

Service:
    ALL

NAT:
    Disable

Log:
    Enable
```

---

# 7️⃣ Dialup Internet Routing

## ⚠️ Routing Problem

Suppose FGT-2 has:

```text
0.0.0.0/0 → ISP
```

Then Internet traffic will leave directly through the local ISP instead of the VPN.

```text
Client
  │
  ▼
FGT-2
  │
  └────► ISP
           │
           ▼
        Internet
```

The traffic will not reach the hub.

---

## 🔹 Required Routing Concept

### FGT-1 / Hub

First create a route to the spoke public IP:

```text
Spoke Public IP
      │
      ▼
     ISP
```

Then:

```text
0.0.0.0/0
    │
    ▼
   ISP
```

And add routes for spoke LAN networks:

```text
192.168.102.0/24
        │
        ▼
    IPsec Tunnel
```

---

## 🔹 FGT-2 / Spoke

Create a route for the Hub public IP:

```text
Hub Public IP
      │
      ▼
     ISP
```

Then route Internet traffic:

```text
0.0.0.0/0
      │
      ▼
 IPsec Tunnel
```

Conceptually:

```text
FGT-2

Hub Public IP
     │
     └────► ISP

Internet
     │
     └────► IPsec Tunnel
                 │
                 ▼
              FGT-1
                 │
                 ▼
                ISP
                 │
                 ▼
              Internet
```

> ⚠️ The route to the Hub's public IP must remain reachable through the local ISP. Otherwise the VPN tunnel itself can break due to recursive routing.

---

# 🔹 Return Traffic Problem

The Hub must know how to reach the spoke LAN.

Example:

```text
Spoke LAN:
    192.168.102.0/24
```

Hub:

```text
192.168.102.0/24
        │
        ▼
   IPsec Tunnel
```

Can be implemented with:

```text
Static Routes
```

or:

```text
Dynamic Routing
    ├── OSPF
    └── BGP
```

---

# 8️⃣ Dialup IPsec with Certificates

Instead of PSK:

```text
Pre-shared Key
```

use:

```text
X.509 Certificate
```

Certificate validation can involve:

```text
Common Name (CN)

Subject

Subject Alternative Name (SAN)

Certificate Authority (CA)
```

Concept:

```text
FortiClient
    │
    │ Certificate
    ▼
FortiGate
    │
    ├── CA Validation
    ├── Subject
    ├── SAN
    └── Certificate Identity
```

---

# 🧪 Certificate VPN Troubleshooting

### Filter VPN Logs

```bash
execute log filter category 1
execute log filter field subtype vpn
execute log display
```

---

# 9️⃣ IKE Debugging

Useful when Phase 1 does not establish.

```bash
diagnose debug enable
diagnose debug application ike -1
```

Stop debug:

```bash
diagnose debug disable
```

or:

```bash
diagnose debug reset
```

---

## 🔍 IKE Debug Checklist

Look for:

```text
IKE version
Mode
Peer ID
Pre-shared Key
Proposal
Encryption
Authentication
DH Group
Local Gateway
Remote Gateway
NAT-T
DPD
XAuth
Certificate
```

Typical negotiation:

```text
IKE Phase 1
     │
     ├── Proposal
     ├── Authentication
     ├── DH
     └── Identity
           │
           ▼
       IKE SA
           │
           ▼
     IKE Phase 2
           │
           ├── Encryption
           ├── Authentication
           ├── PFS
           └── Proxy IDs
                 │
                 ▼
              IPsec SA
```

---

# 🔎 FNBAMD Debug

For authentication-related problems:

```bash
diagnose debug application fnbamd -1
diagnose debug enable
```

Useful when troubleshooting:

```text
XAuth
LDAP
RADIUS
User Authentication
User Groups
Authentication Server
```

> ⚠️ `fnbamd` is the FortiGate authentication daemon. Do not confuse it with FortiGate's malware/botnet detection features.

---

# 🧪 Remote Access Troubleshooting Flow

```text
Client
  │
  ▼
Internet Reachability
  │
  ├── ❌
  │
  └── ✅
       │
       ▼
IKE Phase 1
       │
       ├── Proposal
       ├── PSK / Certificate
       ├── Peer ID
       ├── DH
       └── NAT-T
             │
             ▼
        IKE SA UP
             │
             ▼
        IKE Phase 2
             │
             ├── Proposal
             ├── PFS
             ├── Subnets
             └── Auto-negotiate
                   │
                   ▼
              IPsec SA UP
                   │
                   ▼
              XAuth / User
                   │
                   ▼
              Mode Config
                   │
                   ▼
               Routing
                   │
                   ▼
               Firewall
                   │
                   ▼
             Destination
```

---

# 🧠 Remote Access Design Summary

| Scenario                            | Recommended Design  |
| ----------------------------------- | ------------------- |
| Corporate resources only            | Split Tunnel        |
| Corporate + Internet through HQ     | Full Tunnel         |
| Only published applications         | VIP + WAF           |
| Windows native VPN                  | L2TP over IPsec     |
| FortiClient                         | IPsec Dialup        |
| Strong identity-based VPN           | Certificate + X.509 |
| AD authentication                   | LDAP / FSSO / XAuth |
| Remote Internet browsing            | Dialup IPsec + NAT  |
| Limited corporate access            | Split Include       |
| Everything except selected networks | Split Exclude       |

---

# ⚠️ Important Security Notes

* Avoid `Source = ALL` + `Destination = ALL` for remote-access policies unless intentionally required.
* Prefer specific **VPN user groups**.
* Prefer specific **destination address objects**.
* Use split tunneling when full-tunnel inspection is not required.
* For full-tunnel VPN, make sure routing and return routing are correct.
* Keep the route to the VPN peer's public IP outside the VPN tunnel.
* Verify NAT behavior carefully for Internet-bound VPN traffic.
* Verify NAT-T when clients are behind NAT.
* Make sure FortiClient Phase 1/Phase 2 proposals match FortiGate.
* Use certificate authentication when certificate-based identity is required.
* Use IKE/FNBAMD debug when authentication or negotiation fails.

```

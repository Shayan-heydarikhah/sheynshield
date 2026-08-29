# 🔗 FortiGate IPsec Aggregation & Redundancy

> IPsec Aggregate • Round-Robin • Redundant • IPsec Monitoring • OSPF • HA • ESP Sync

---

# 1️⃣ IPsec Aggregate — Overview

IPsec aggregation allows multiple IPsec tunnels to be combined into a single logical interface.

```text
                    FortiGate HUB
                         │
             ┌───────────┴───────────┐
             │                       │
         IPsec Link-1            IPsec Link-2
         ISP-1                    ISP-2
             │                       │
             └───────────┬───────────┘
                         │
                  IPsec Aggregate
                         │
                   12.12.12.1/24
                         │
                         │
                   12.12.12.2/24
                         │
                  IPsec Aggregate
             ┌───────────┴───────────┐
             │                       │
         IPsec Link-1            IPsec Link-2
         ISP-1                    ISP-2
````

---

# 2️⃣ Dual ISP — Default Routes

## 🔹 FGT-1 / Hub

Two ISP connections:

```text
ISP-1:
    Gateway: 1.1.1.2
    Distance: 10

ISP-2:
    Gateway: 11.11.11.2
    Distance: 11
```

Equivalent:

```text
0.0.0.0/0 → 1.1.1.2     AD 10
0.0.0.0/0 → 11.11.11.2  AD 11
```

---

## 🔹 FGT-2 / Spoke

```text
ISP-1:
    Gateway: 2.2.2.2
    Distance: 10

ISP-2:
    Gateway: 22.22.22.2
    Distance: 11
```

```text
0.0.0.0/0 → 2.2.2.2     AD 10
0.0.0.0/0 → 22.22.22.2  AD 11
```

---

# 3️⃣ Create IPsec Member — FGT-1

Create the first static IPsec tunnel:

```text
VPN
└── IPsec Tunnels
    └── Custom
```

### Link-1

```text
Name:
    link-1

Connection:
    Static IP

Remote Gateway:
    2.2.2.1

Interface:
    ISP-1

Authentication:
    Pre-shared Key

PSK:
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
```

### Phase 2

```text
Local:
    All Subnets

Remote:
    All Subnets

Encryption:
    DES

Authentication:
    MD5

DH:
    Group 5

PFS:
    Disable

Auto-negotiate:
    Enable
```

---

# 4️⃣ Create Second IPsec Member — FGT-1

Create another tunnel with the same parameters.

```text
Name:
    link-2

Remote Gateway:
    22.22.22.1

Interface:
    ISP-2
```

Use the same:

```text
PSK:
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
```

Phase 2:

```text
All Subnets
DES
MD5
DH Group 5
PFS Disable
Auto-negotiate Enable
```

---

# 5️⃣ IPsec Aggregate Interface

Go to:

```text
VPN
└── IPsec
    └── IPsec Aggregate
```

Create:

```text
Name:
    aggregate-links-12
```

Members:

```text
link-1
link-2
```

---

# 6️⃣ IPsec Aggregate Algorithms

Available algorithms:

```text
Weighted Round Robin
L3
L4
Redundant
Round-Robin
```

---

## 🔹 L3 Load Balancing

```text
Algorithm:
    L3
```

Concept:

```text
Client-1 ─────► Link-1
Client-2 ─────► Link-2
Client-3 ─────► Link-1
Client-4 ─────► Link-2
```

Load balancing is based on Layer-3 information.

---

## 🔹 L4 Load Balancing

```text
Algorithm:
    L4
```

Uses Layer-4 information such as:

```text
Source IP
Destination IP
Source Port
Destination Port
Protocol
```

Concept:

```text
TCP/443 flow-1 ──► Link-1
TCP/443 flow-2 ──► Link-2
UDP flow-1 ──────► Link-1
```

---

## 🔹 Weighted Round Robin

```text
Algorithm:
    Weighted Round Robin
```

Useful when links have different capacities.

Example:

```text
ISP-1:
    Weight = 3

ISP-2:
    Weight = 1
```

Concept:

```text
Traffic distribution:

ISP-1:
    ████████████████████

ISP-2:
    ███████
```

---

# 7️⃣ Round-Robin — Recommended Aggregate Mode

For FortiOS IPsec aggregation:

```bash
config system ipsec-aggregate
    edit "aggregate-links-12"
        set algorithm round-robin
    next
end
```

> ⭐ Round-robin is a practical choice when the goal is to distribute IPsec traffic across aggregate members.

---

# ⚠️ Important — Default Route Distance

After configuring IPsec aggregation with Round-Robin, the IPsec links may go down if the underlying default routes have different distances.

Example:

```text
ISP-1:
    AD 10

ISP-2:
    AD 11
```

For the aggregate scenario, use the same distance:

```text
ISP-1:
    AD 10

ISP-2:
    AD 10
```

Concept:

```text
0.0.0.0/0
    │
    ├── ISP-1 AD 10
    │
    └── ISP-2 AD 10
```

> ⚠️ If using this design, verify route selection and update-static-route behavior carefully rather than blindly making both routes equal.

---

# 8️⃣ Remove IPsec Monitor

If using IPsec Aggregate Round-Robin, remove the old monitor configuration.

```bash
config vpn ipsec phase1-interface
    edit "link-2"
        unset monitor
    next
end
```

Previously:

```bash
set monitor link-1
```

Remove it when the aggregate itself is responsible for member handling.

---

# 9️⃣ Aggregate Interface IP

Assign the logical IP to the aggregate interface.

## FGT-1

```text
Interface:
    aggregate-links-12

IP:
    12.12.12.1/24

Remote:
    12.12.12.2

Ping:
    Enable
```

## FGT-2

```text
Interface:
    aggregate-links-12

IP:
    12.12.12.2/24

Remote:
    12.12.12.1

Ping:
    Enable
```

Concept:

```text
FGT-1                         FGT-2

12.12.12.1/24                12.12.12.2/24
     │                              │
     ▼                              ▼
┌──────────────┐              ┌──────────────┐
│ IPsec        │              │ IPsec        │
│ Aggregate    │══════════════│ Aggregate    │
└──────────────┘              └──────────────┘
    │      │                      │      │
    │      │                      │      │
 Link-1  Link-2                Link-1  Link-2
```

---

# 🔟 Firewall Policies

## FGT-1 / Hub

### Incoming

```text
LAN
    +
aggregate-links-12
```

### Outgoing

```text
LAN
    +
aggregate-links-12
```

Policy:

```text
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

# 1️⃣1️⃣ OSPF over IPsec Aggregate

## FGT-1

```text
Router ID:
    1.1.1.1

Area:
    0.0.0.0
```

Networks:

```text
12.12.12.0/24
192.168.101.0/24
```

Concept:

```text
             OSPF
              │
              ▼
     IPsec Aggregate
              │
       ┌──────┴──────┐
       │             │
    Link-1         Link-2
```

---

## FGT-2

```text
Router ID:
    2.2.2.2

Area:
    0.0.0.0
```

Networks:

```text
12.12.12.0/24
192.168.102.0/24
```

---

# 1️⃣2️⃣ IPsec Monitoring

Without aggregation, one tunnel can monitor another.

Example:

```bash
config vpn ipsec phase1-interface
    edit "link-2"
        set monitor "link-1"
    next
end
```

Concept:

```text
             Link-1
               │
          Primary Link
               │
          ┌────▼────┐
          │ Monitor │
          └────┬────┘
               │
          Link-2
          Standby
```

If Link-1 fails:

```text
Link-1
   │
   ❌
   │
   ▼
Link-2
   │
   ▼
Forwarding
```

---

# ⚠️ Monitoring vs Aggregation

Do not mix the concepts blindly.

### Monitoring

```text
Link-1
  │
  └── monitor ──► Link-2

Primary / Backup
```

### Aggregation

```text
Link-1 ──┐
         ├──► Aggregate
Link-2 ──┘

Load balancing / redundancy
```

---

# 1️⃣3️⃣ OSPF Cost for Preferred ISP

If ISP-1 is more powerful:

```text
OSPF Interface Cost:

ISP-1:
    Cost 10

ISP-2:
    Cost 11
```

Concept:

```text
OSPF
 │
 ├── ISP-1
 │     Cost 10  ← Preferred
 │
 └── ISP-2
       Cost 11  ← Backup
```

This can influence OSPF path selection while both links remain available.

---

# ⚡ Fast Convergence Consideration

Switching between IPsec members may have some convergence delay.

Possible design:

```text
IPsec Aggregate
       +
IPsec Monitoring
       +
OSPF
```

But avoid unnecessary overlapping mechanisms.

Evaluate:

```text
Detection
   ↓
IKE/IPsec State
   ↓
Routing Update
   ↓
OSPF Convergence
   ↓
Forwarding
```

---

# 1️⃣4️⃣ HA + IPsec VPN

IPsec can be deployed on an HA pair.

Example:

```text
FGT-1:
    HA Priority = 129

FGT-2:
    HA Priority = 128
```

FGT-1 becomes primary.

---

# 🔐 HA IPsec Configuration

Create the same IPsec configuration on the HA pair.

```text
Remote Gateway:
    3.3.3.1

Name:
    ipsec-tun-1

PSK:
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
```

No XAuth:

```text
XAuth:
    Disable
```

---

## 🔹 Phase 2

```text
Subnets:
    All

PFS:
    Disable

Auto-negotiate:
    Enable
```

---

# 1️⃣5️⃣ HA IPsec Interface

```text
Interface:
    ipsec-tun-1

FGT-1:
    13.13.13.1/24

Remote:
    13.13.13.2
```

Enable:

```text
Ping
```

---

# 1️⃣6️⃣ HA Firewall Policies

Incoming:

```text
LAN
    +
ipsec-tun-1
```

Outgoing:

```text
LAN
    +
ipsec-tun-1
```

Policy:

```text
Source:
    ALL

Destination:
    ALL

Service:
    ALL

NAT:
    Disable

Logging:
    Enable
```

---

# 1️⃣7️⃣ HA Routing

Static:

```text
0.0.0.0/0
    │
    ▼
ISP-1
```

OSPF:

```text
Router ID:
    1.1.1.1
```

Networks:

```text
13.13.13.0/24
192.168.101.0/24
```

---

# 1️⃣8️⃣ HA + ESP Sequence Synchronization

In some HA IPsec failover scenarios, ESP sequence synchronization is important.

Configure:

```bash
config vpn ipsec phase1-interface
    edit "ipsec-tun-1"
        set ha-sync-esp-seqno enable
    next
end
```

Concept:

```text
             HA Cluster
        ┌─────────────────┐
        │                 │
        │  FGT-1 Primary  │
        │       │         │
        │       │ ESP     │
        │       ▼         │
        │  FGT-2 Standby  │
        │                 │
        └─────────────────┘
```

Without correct HA synchronization:

```text
Active FGT
    │
    ▼
IPsec / ESP
    │
Failover
    │
    ▼
New Active FGT
    │
    ❌
Potential ESP/IPsec problems
```

With ESP sequence synchronization:

```text
Active FGT
    │
    ▼
ESP State
    │
    ║ Sync
    ▼
Standby FGT
    │
    ▼
Failover
    │
    ▼
Continue IPsec forwarding
```

---

# ⚠️ HA IPsec Troubleshooting

If HA failover causes all IPsec negotiations to drop:

Check:

```text
HA Session Pickup
IPsec configuration synchronization
ESP sequence synchronization
IKE/IPsec state
Routing
```

Command:

```bash
config vpn ipsec phase1-interface
    edit "ipsec-tun-1"
        set ha-sync-esp-seqno enable
    next
end
```

---

# 1️⃣9️⃣ IPsec Location ID

When multiple IPsec VPNs are used with load balancing, location ID can help distinguish FortiGate locations.

```bash
config system settings
    set location-id 1.1.1.1
end
```

Example:

```text
HUB:
    location-id 1.1.1.1

SPOKE-1:
    location-id 2.2.2.2

SPOKE-2:
    location-id 3.3.3.3
```

---

# 2️⃣0️⃣ IPsec Aggregate Troubleshooting

Show aggregate information:

```bash
diagnose sys ipsec-aggregate list
```

Useful for checking:

```text
VRF
Members
Aggregate state
Member status
```

---

# 2️⃣1️⃣ IPsec over SD-WAN

If IPsec tunnels are transported over SD-WAN:

```text
SD-WAN
   │
   ├── ISP-1
   └── ISP-2
         │
         ▼
      IPsec
```

Do not automatically assume the IPsec interface itself must be added as an SD-WAN member.

Concept:

```text
Underlay:
    SD-WAN / ISP

Overlay:
    IPsec

Routing:
    OSPF / BGP / Static
```

The IPsec tunnel can use the appropriate underlay path while the overlay remains logically separate.

---

# 2️⃣2️⃣ Redundant IPsec Interface

Another design is to use redundant dialup IPsec tunnels rather than an IPsec aggregate.

---

## 🔹 Hub / FGT-1

Create:

```text
IPsec Custom
Connection:
    Dialup

Advanced:
    Device Creation:
        Enable

    Add Route:
        Enable
```

Do not assign an IP address to the IPsec interface.

```text
IPsec Interface:
    No IP Assignment
```

---

# 2️⃣3️⃣ Spoke / FGT-2

Create two dialup IPsec interfaces.

```text
IPsec-1
    Device Creation:
        Enable

IPsec-2
    Device Creation:
        Enable
```

Second tunnel can monitor the first:

```bash
config vpn ipsec phase1-interface
    edit "ipsec-2"
        set monitor "ipsec-1"
    next
end
```

Concept:

```text
                HUB
                 │
          ┌──────┴──────┐
          │             │
       IPsec-1       IPsec-2
          │             │
       Primary         Backup
          │             │
          └──────┬──────┘
                 │
               SPOKE
```

---

# 2️⃣4️⃣ Redundant IPsec Routing

Use two default routes.

Example:

```text
0.0.0.0/0
    │
    ├── IPsec-1
    │      AD 10
    │
    └── IPsec-2
           AD 11
```

When IPsec-1 fails:

```text
IPsec-1
   │
   ❌
   │
   ▼
IPsec-2
   │
   ▼
Routing Table Update
```

---

# 🔄 Route Update Options

Two common approaches:

```text
Option 1:
    Different administrative distances

Option 2:
    Update static route when tunnel becomes unavailable
```

Concept:

```text
Primary:
    AD 10

Backup:
    AD 11
```

or:

```text
Primary:
    Route active

Failure:
    Route removed/updated

Backup:
    Becomes active
```

---

# 2️⃣5️⃣ Redundant IPsec — Important Difference

Unlike routed IPsec interfaces with assigned tunnel addresses:

```text
IPsec-1:
    No IP

IPsec-2:
    No IP
```

Therefore:

```text
❌ Ping the IPsec interface
```

is not the main validation method.

Instead, define the required Phase-2 local/remote subnets and test actual traffic.

---

# 2️⃣6️⃣ Add-Route + Device Creation

These options provide different functionality.

### Device Creation

```text
Device Creation:
    Enable
```

Allows the tunnel to be represented as an interface.

### Add Route

```text
Add Route:
    Enable
```

Allows FortiGate to install routes associated with the VPN configuration.

Concept:

```text
IPsec Configuration
       │
       ├── Device Creation
       │       │
       │       ▼
       │   IPsec Interface
       │
       └── Add Route
               │
               ▼
          Routing Table
```

---

# 2️⃣7️⃣ Aggregate vs Redundant IPsec

| Feature                       | IPsec Aggregate               | Redundant IPsec             |
| ----------------------------- | ----------------------------- | --------------------------- |
| Multiple tunnels              | ✅                             | ✅                           |
| Logical interface             | ✅                             | Depends on design           |
| Load balancing                | ✅                             | ❌                           |
| Active/standby                | Possible                      | ✅                           |
| Round-robin                   | ✅                             | ❌                           |
| Monitoring                    | Usually not primary mechanism | ✅                           |
| Multiple ISP                  | ✅                             | ✅                           |
| OSPF/BGP overlay              | ✅                             | Depends on interface design |
| Best for traffic distribution | ✅                             | ❌                           |
| Best for simple failover      | ⚠️                            | ✅                           |

---

# 2️⃣8️⃣ Recommended Decision Matrix

```text
Need load balancing?
        │
       YES
        │
        ▼
 IPsec Aggregate
        │
        ├── Round-Robin
        ├── L3
        ├── L4
        └── Weighted RR


Need only primary/backup?
        │
       YES
        │
        ▼
 Redundant IPsec
        │
        └── Monitor


Need HA?
        │
       YES
        │
        ▼
HA IPsec
   +
ESP Sequence Sync


Need dynamic routing?
        │
       YES
        │
        ▼
IPsec Interface
   +
OSPF / BGP
```

---

# 🧪 2️⃣9️⃣ Verification Commands

## IPsec Aggregate

```bash
diagnose sys ipsec-aggregate list
```

---

## IKE Gateway

```bash
diagnose vpn ike gateway list
```

---

## IPsec Tunnel

```bash
diagnose vpn tunnel list
```

---

## IKE Debug

```bash
diagnose debug enable
diagnose debug application ike -1
```

Disable:

```bash
diagnose debug disable
```

---

# 🧠 3️⃣0️⃣ Troubleshooting Flow

```text
                 IPsec Problem
                       │
                       ▼
              Check Underlay
                       │
              ┌────────┴────────┐
              │                 │
           ISP-1             ISP-2
              │                 │
              └────────┬────────┘
                       ▼
                 IKE Status
                       │
                       ▼
                IPsec SA Status
                       │
                       ▼
              Aggregate Members
                       │
                       ▼
                  Routing
                       │
                       ▼
                    OSPF
                       │
                       ▼
                  Firewall
                       │
                       ▼
                  Traffic
```

---

# ⚠️ 3️⃣1️⃣ Common Mistakes

### ❌ Different ISP default-route distances

```text
ISP-1 AD 10
ISP-2 AD 11
```

while expecting both links to participate equally in aggregation.

---

### ❌ Keeping IPsec monitor with Aggregate

```bash
set monitor link-1
```

when the aggregate is intended to manage member behavior.

---

### ❌ Forgetting to remove monitor

```bash
unset monitor
```

when moving from monitor-based redundancy to aggregate-based design.

---

### ❌ NAT enabled

For routed VPN traffic:

```text
NAT:
    Disable
```

unless NAT is intentionally required.

---

### ❌ Missing return routes

Always verify:

```text
Local LAN
    │
    ▼
IPsec Aggregate
    │
    ▼
Remote LAN
```

and the reverse path:

```text
Remote LAN
    │
    ▼
IPsec Aggregate
    │
    ▼
Local LAN
```

---

### ❌ Assuming IPsec tunnel state = routing state

A tunnel can be:

```text
IPsec:
    UP
```

while:

```text
Route:
    Incorrect

OSPF:
    Not converged

Policy:
    Blocking

NAT:
    Incorrect
```

Therefore always validate the complete forwarding path.

---

# ⭐ 3️⃣2️⃣ Design Summary

```text
                    ┌───────────────┐
                    │   FortiGate   │
                    │      HUB      │
                    └───────┬───────┘
                            │
                    IPsec Aggregate
                            │
                 ┌──────────┴──────────┐
                 │                     │
             IPsec-1               IPsec-2
             ISP-1                  ISP-2
                 │                     │
                 └──────────┬──────────┘
                            │
                    Remote FortiGate
                            │
                         LAN
```

### Key Concepts

```text
IPsec Aggregate
    ↓
Multiple IPsec members
    ↓
One logical overlay
    ↓
Load balancing / redundancy


IPsec Monitor
    ↓
Primary / Backup
    ↓
Failure detection


OSPF
    ↓
Dynamic route exchange
    ↓
Fast path recalculation


HA + ESP Sync
    ↓
IPsec continuity during HA failover
```

---

# 🚀 Quick Reference

| Requirement                                     | Feature                             |
| ----------------------------------------------- | ----------------------------------- |
| Multiple IPsec tunnels as one logical interface | IPsec Aggregate                     |
| Load balancing                                  | Aggregate                           |
| Round-robin                                     | `set algorithm round-robin`         |
| Primary/backup VPN                              | IPsec Monitor                       |
| Dynamic routing over aggregate                  | OSPF / BGP                          |
| Preferred ISP                                   | OSPF interface cost                 |
| HA IPsec continuity                             | ESP sequence synchronization        |
| Identify IPsec aggregate members                | `diagnose sys ipsec-aggregate list` |
| Tunnel troubleshooting                          | `diagnose vpn tunnel list`          |
| IKE troubleshooting                             | `diagnose vpn ike gateway list`     |
| IKE debug                                       | `diagnose debug application ike -1` |
| SD-WAN underlay + IPsec overlay                 | SD-WAN + IPsec                      |
| Simple redundant dialup tunnels                 | Redundant IPsec                     |

```

## 🔗 SheynShield Resources

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

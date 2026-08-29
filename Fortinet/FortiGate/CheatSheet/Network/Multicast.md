# IPv4 Multicast — Complete Networking & FortiGate 

> **SheynShield | Network Security & Design Knowledge Base**
> IPv4 Multicast • IGMP • PIM • RP • SSM • MSDP • IGMP Snooping • FortiGate

---

## 📌 Table of Contents

* [1. Multicast Fundamentals](#1-multicast-fundamentals)
* [2. IPv4 Multicast Addressing](#2-ipv4-multicast-addressing)
* [3. Multicast MAC Address](#3-multicast-mac-address)
* [4. IGMP](#4-igmp)

  * [IGMPv1](#41-igmpv1)
  * [IGMPv2](#42-igmpv2)
  * [IGMPv3](#43-igmpv3)
  * [IGMPv3 Lite](#44-igmpv3-lite)
* [5. IGMP Snooping, CGMP & RGMP](#5-igmp-snooping-cgmp--rgmp)
* [6. IGMP Querier & PIM DR](#6-igmp-querier--pim-dr)
* [7. PIM Fundamentals](#7-pim-fundamentals)
* [8. PIM Dense Mode](#8-pim-dense-mode)
* [9. PIM Sparse Mode](#9-pim-sparse-mode)
* [10. Rendezvous Point](#10-rendezvous-point)
* [11. Auto-RP](#11-auto-rp)
* [12. BSR](#12-bsr)
* [13. Anycast RP](#13-anycast-rp)
* [14. MSDP](#14-msdp)
* [15. Bidirectional PIM](#15-bidirectional-pim)
* [16. PIM SSM](#16-pim-ssm)
* [17. Multicast Routing Tables](#17-multicast-routing-tables)
* [18. RPF](#18-rpf)
* [19. Multicast Troubleshooting](#19-multicast-troubleshooting)
* [20. FortiGate Multicast](#20-fortigate-multicast)
* [21. FortiGate Multicast Flow](#21-fortigate-multicast-flow)
* [22. FortiGate MFC](#22-fortigate-mfc)
* [23. Multicast Design Best Practices](#23-multicast-design-best-practices)
* [24. Quick Exam Reference](#24-quick-exam-reference)

---

# 1. Multicast Fundamentals

## What is Multicast?

Multicast is a **one-to-many** or **many-to-many** communication model where a sender transmits traffic to a multicast group rather than individually addressing every receiver.

### Traffic Models

| Model     | Description    | Example          |
| --------- | -------------- | ---------------- |
| Unicast   | One → One      | Client → Server  |
| Broadcast | One → Everyone | ARP broadcast    |
| Multicast | One → Many     | IPTV             |
| Multicast | Many → Many    | Video conference |
| Anycast   | One → Nearest  | DNS services     |

### Multicast Models

#### ASM — Any-Source Multicast

Receivers request a multicast **group** without specifying the source.

```text
(*,G)
```

Example:

```text
(*,239.1.1.1)
```

The receiver essentially says:

> "I want traffic for group 239.1.1.1 regardless of its source."

---

#### SSM — Source-Specific Multicast

The receiver specifies both the source and multicast group.

```text
(S,G)
```

Example:

```text
(192.168.1.50,232.1.1.1)
```

The receiver says:

> "I only want traffic from 192.168.1.50 for group 232.1.1.1."

### Key Difference

```text
ASM
(*,G)
    ↓
Any source → Group

SSM
(S,G)
    ↓
Specific source → Group
```

---

# 2. IPv4 Multicast Addressing

IPv4 multicast uses **Class D** addresses:

```text
224.0.0.0 – 239.255.255.255
```

Binary:

```text
1110.xxxxxxxx.xxxxxxxx.xxxxxxxx
```

### Important

IPv4 multicast addresses do **not use subnet masks in the same way as unicast addressing**.

The multicast address space is administered by IANA and various ranges have defined purposes.

---

## Multicast Address Ranges

| Range          | Purpose                             |
| -------------- | ----------------------------------- |
| `224.0.0.0/24` | Local-network control multicast     |
| `224.0.1.0/24` | Globally routed control multicast   |
| `232.0.0.0/8`  | SSM                                 |
| `233.0.0.0/8`  | GLOP                                |
| `239.0.0.0/8`  | Administratively scoped multicast   |
| Various ranges | Reserved / historical / special use |

---

## `224.0.0.0/24` — Local Multicast

Traffic in this range is intended for the **local network segment** and normally has TTL 1.

Routers generally do not forward these addresses.

### Important Addresses

| Multicast IP | Purpose                    |
| ------------ | -------------------------- |
| `224.0.0.1`  | All IPv4 multicast hosts   |
| `224.0.0.2`  | All IPv4 multicast routers |
| `224.0.0.4`  | DVMRP                      |
| `224.0.0.5`  | All OSPF routers           |
| `224.0.0.6`  | OSPF DR/BDR                |
| `224.0.0.9`  | RIPv2                      |
| `224.0.0.10` | EIGRP                      |
| `224.0.0.13` | All PIM routers            |
| `224.0.0.22` | IGMPv3 routers             |
| `224.0.0.25` | RGMP                       |
| `224.0.1.39` | Cisco Auto-RP announcement |
| `224.0.1.40` | Cisco Auto-RP discovery    |

> ⚠️ Do not use `224.0.0.0/24` as a normal application multicast range.

---

# 2.1 SSM Range

```text
232.0.0.0/8
```

SSM uses the source-specific model:

```text
(S,G)
```

Example:

```text
Source = 192.168.1.50
Group  = 232.1.1.1
```

---

# 2.2 GLOP Addressing

GLOP historically provides a mechanism for deriving multicast addresses from an organization's Autonomous System Number.

```text
233.0.0.0/8
```

Conceptually:

```text
233.AS.AS.x
```

The AS number is encoded into the middle portion of the multicast address.

Example:

```text
AS 10
    ↓
233.0.10.x
```

GLOP is primarily a historical multicast addressing concept and should not be confused with private multicast addressing.

---

# 2.3 Administratively Scoped Multicast

```text
239.0.0.0/8
```

Typical use:

* Enterprise multicast
* Internal applications
* IPTV
* Video distribution
* Lab environments
* Internal service discovery

Example:

```text
239.1.1.1
239.192.20.1
```

These addresses are intended for administrative/local use and are not globally routed across the public Internet.

---

# 2.4 Historical / Reserved Ranges

Some multicast ranges are reserved or were historically associated with protocols such as:

```text
224.0.2.0 – 224.0.255.255
224.2.0.0 – 224.2.255.255
225.0.0.0 – 231.255.255.255
234.0.0.0 – 238.255.255.255
```

Always verify the intended address allocation before deploying production multicast applications.

---

# 3. Multicast MAC Address

Multicast forwarding at Layer 2 uses a multicast destination MAC rather than the unicast MAC address of each receiver.

---

## IPv4 Multicast MAC

IPv4 multicast maps to:

```text
01:00:5E:xx:xx:xx
```

The first 24 bits are:

```text
01:00:5E
```

Only **23 bits** of the IPv4 multicast address are mapped into the Ethernet multicast MAC.

```text
IPv4 Multicast
       ↓
   23 bits
       ↓
01:00:5E:xx:xx:xx
```

### Important Consequence

Because an IPv4 multicast address contains 28 variable bits while the Ethernet mapping provides only 23 bits:

```text
2^28 multicast IPs
        ↓
2^23 multicast MAC values
```

Therefore:

> **32 different IPv4 multicast addresses can map to the same multicast MAC address.**

This creates a **multicast MAC aliasing problem**.

---

## Example

Multicast IP:

```text
239.1.1.1
```

Take the lower 23 bits:

```text
239.1.1.1
        ↓
01:00:5E:01:01:01
```

Therefore:

```text
IP   = 239.1.1.1
MAC  = 01:00:5E:01:01:01
```

---

## Another Example

```text
239.255.0.6
```

Maps to:

```text
01:00:5E:7F:00:06
```

---

## IPv6 Multicast MAC

IPv6 multicast Ethernet mapping uses:

```text
33:33:xx:xx:xx:xx
```

Example:

```text
IPv6 multicast
      ↓
33:33:xx:xx:xx:xx
```

---

# 3.1 Unicast vs Multicast MAC

### I/G Bit

The least-significant bit of the first MAC octet identifies Individual/Group addressing.

```text
I/G = 0 → Individual / Unicast
I/G = 1 → Group / Multicast
```

Therefore common multicast MAC addresses begin with an odd first octet.

Examples:

```text
Unicast:
00:xx:xx:xx:xx:xx
02:xx:xx:xx:xx:xx
04:xx:xx:xx:xx:xx

Multicast:
01:xx:xx:xx:xx:xx
03:xx:xx:xx:xx:xx
05:xx:xx:xx:xx:xx
```

---

# 4. IGMP

**IGMP — Internet Group Management Protocol**

IGMP operates between:

```text
Host ↔ Multicast Router
```

It tells multicast routers which multicast groups have interested receivers on a local network.

```text
          Multicast Source
                 |
                 |
          Multicast Router
                 |
        +--------+--------+
        |        |        |
       PC1      PC2      PC3
        |        |        |
       IGMP     IGMP     IGMP
```

### Key Facts

```text
Protocol:
IGMP

IP Protocol Number:
2

Version:
IPv4 multicast

IPv6 equivalent:
MLD
```

> IPv6 does not use IGMP. IPv6 uses **MLD — Multicast Listener Discovery**.

---

# 4.1 IGMPv1

IGMPv1 introduced basic multicast group membership management.

### Main Messages

| Type              |    Hex | Meaning                         |
| ----------------- | -----: | ------------------------------- |
| Membership Query  | `0x11` | Router asks who wants multicast |
| Membership Report | `0x12` | Host reports group membership   |

---

## IGMPv1 Query

Router periodically sends:

```text
0x11
```

Destination:

```text
224.0.0.1
```

Meaning:

> "Which hosts are interested in multicast?"

Hosts respond with:

```text
0x12
```

---

## IGMPv1 Leave

IGMPv1 has **no explicit Leave message**.

If a host stops listening, the router eventually determines that the group has no members after membership reports stop arriving.

Typical timing concept:

```text
Query interval ≈ 60 seconds
Multiple queries
        ↓
Membership timeout
        ↓
Group state removed
```

---

## Cisco Example

### Router

```cisco
ip multicast-routing

interface GigabitEthernet0/0
 ip pim dense-mode
 ip igmp version 1
 no shutdown
```

### Receiver

```cisco
interface GigabitEthernet0/0
 ip igmp join-group 224.1.1.1
 ip igmp version 1
```

### Verification

```cisco
show ip igmp groups
show ip igmp interface GigabitEthernet0/0
show ip mroute
```

---

# 4.2 IGMPv2

IGMPv2 extends IGMPv1 with explicit leave processing and querier election.

### Important Message Types

| Type              |    Hex | Meaning           |
| ----------------- | -----: | ----------------- |
| Membership Query  | `0x11` | Query             |
| Membership Report | `0x16` | Membership Report |
| Leave Group       | `0x17` | Leave             |

> `0x12` is the IGMPv1 Membership Report type. IGMPv2 uses `0x16`.

---

## IGMPv2 Header

```text
+--------+-------------------------+
| Type   | Max Response Time       |
+--------+-------------------------+
|       Checksum              |
+-----------------------------+
|       Group Address         |
+-----------------------------+
```

---

## Response Suppression

Without suppression:

```text
Router
  |
  +---- Query
        |
        +---- PC1 → Report
        +---- PC2 → Report
        +---- PC3 → Report
        +---- PC4 → Report
```

This can create unnecessary control traffic.

With IGMPv2 suppression:

```text
Router → General Query
             |
       Random timers
             |
        PC2 reaches 0
             |
         Report
             |
     Other hosts hear it
             |
      Cancel own report
```

Result:

```text
Many hosts
    ↓
One effective report
```

---

# 4.2.1 IGMPv2 Leave Process

When the last known member leaves:

```text
Host
 |
 | Leave
 ↓
Router
 |
 | Group-Specific Query
 ↓
LAN
 |
 +---- Are there still members?
```

If no response arrives:

```text
Group state
    ↓
Removed
```

---

## Last Member Query Parameters

Cisco-style examples:

```cisco
interface GigabitEthernet0/0
 ip igmp last-member-query-interval 2000
 ip igmp last-member-query-count 3
```

Conceptually:

```text
Leave
 ↓
Group-Specific Query
 ↓
Wait
 ↓
Another query
 ↓
No response
 ↓
Remove group
```

---

# 4.2.2 IGMP Query Interval

Example:

```cisco
interface GigabitEthernet0/0
 ip igmp query-interval 60
```

The default query interval is commonly 60 seconds in Cisco environments.

Verification:

```cisco
show ip igmp interface GigabitEthernet0/0
```

---

# 4.2.3 Immediate Leave

Useful when a port is known to have a single multicast receiver.

```cisco
interface GigabitEthernet0/0
 ip igmp immediate-leave group-list 1

access-list 1 permit 225.1.1.1
```

This avoids waiting for the normal group-specific query process.

> ⚠️ Do not blindly enable immediate leave on a shared port where multiple receivers may exist.

---

# 4.2.4 Static Group

A router can be configured as a permanent multicast group member:

```cisco
interface GigabitEthernet0/0
 ip igmp static-group 224.1.1.1
```

This creates a static membership rather than waiting for a dynamic host report.

---

# 4.2.5 IGMP Access Control

Example:

```cisco
interface GigabitEthernet0/0
 ip igmp access-group 1

access-list 1 permit 225.1.1.1
```

This limits which multicast groups can be joined.

---

# 4.3 IGMPv3

IGMPv3 adds **source filtering** and enables Source-Specific Multicast.

Main concept:

```text
IGMPv1/v2
(*,G)

IGMPv3
(S,G)
```

Example:

```text
Source:
192.168.2.10

Group:
232.1.1.1
```

Receiver requests:

```text
(192.168.2.10,232.1.1.1)
```

---

## IGMPv3 Report

IGMPv3 Membership Report:

```text
0x22
```

Destination:

```text
224.0.0.22
```

---

## IGMPv3 Source Filtering

Two important modes:

### INCLUDE

```text
INCLUDE
Source A
Source B
```

Meaning:

> Accept traffic from these sources.

### EXCLUDE

```text
EXCLUDE
Source A
Source B
```

Meaning:

> Accept sources except those specified.

---

## IGMPv3 Example

```cisco
interface GigabitEthernet0/0
 ip igmp version 3
 ip pim sparse-mode
```

SSM:

```cisco
ip pim ssm default
```

Or with an ACL:

```cisco
ip pim ssm range 10

access-list 10 permit 232.0.0.0 0.255.255.255
```

Receiver:

```cisco
interface GigabitEthernet0/0
 ip igmp version 3
 ip igmp join-group 232.1.1.1 source 192.168.2.10
```

---

# 4.3.1 IGMPv3 Query Fields

Important fields include:

```text
QRV
Querier's Robustness Variable

QQIC
Querier's Query Interval Code
```

### QRV

```text
3 bits
```

Range:

```text
0–7
```

The value communicates the robustness variable used by the querier.

### QQIC

```text
8 bits
```

Used to encode the query interval.

---

# 4.4 IGMPv3 Lite

IGMPv3 Lite is a Cisco-related compatibility mechanism designed to support applications that need source-specific membership information when the operating system/application environment does not natively provide full IGMPv3 functionality.

Example:

```cisco
interface GigabitEthernet0/0
 ip igmp v3lite
 ip pim sparse-mode
```

Conceptually:

```text
Application
    |
    | source + group
    ↓
IGMPv3 Lite daemon
    |
    ↓
Router
    |
    ↓
(S,G)
```

---

# 5. IGMP Snooping, CGMP & RGMP

Multicast has an important Layer 2 problem:

```text
Switch
  |
  +---- Receiver A
  +---- Receiver B
  +---- Receiver C
  +---- Receiver D
```

If the switch does not know which ports require a multicast stream, multicast traffic may be flooded.

---

# 5.1 IGMP Snooping

IGMP Snooping allows a Layer 2 switch to inspect IGMP messages and build multicast forwarding state.

```text
Host
  |
IGMP Join
  |
Switch
  |
Detect:
Group = 239.1.1.1
Port = Gi0/10
```

The switch then forwards multicast traffic only toward interested ports.

---

## Without IGMP Snooping

```text
Multicast
    |
 Switch
    |
 +--+--+--+--+
 |  |  |  |
PC PC PC PC

Flood
```

---

## With IGMP Snooping

```text
Multicast
    |
 Switch
    |
    +------ Receiver
```

Only required ports receive the stream.

---

## Useful Commands

```cisco
show ip igmp snooping
show ip igmp snooping groups
show ip igmp snooping querier
show ip igmp snooping monitor
show mac address-table multicast igmp-snooping
```

---

# 5.2 IGMP Snooping Router Port

The switch needs to identify the multicast-router-facing interface.

Router discovery can be based on multicast control traffic such as:

```text
OSPF:
224.0.0.5
224.0.0.6

PIM:
224.0.0.13

DVMRP:
224.0.0.4
```

Static configuration can also be used:

```cisco
ip igmp snooping vlan 1 mrouter interface FastEthernet0/0
```

---

# 5.3 Immediate Leave on IGMP Snooping

For single-host access ports:

```cisco
ip igmp snooping vlan 1 immediate-leave
```

Last-member parameters:

```cisco
ip igmp snooping vlan 1 last-member-query-count 2
ip igmp snooping vlan 1 last-member-query-interval 1000
```

### Design Rule

```text
One receiver per access port
        ↓
Immediate Leave can be useful

Multiple receivers behind one port
        ↓
Be careful with Immediate Leave
```

---

# 5.4 IGMP Report Suppression

Normally, report suppression reduces duplicate membership reports.

Avoid disabling it unless there is a specific interoperability/troubleshooting reason:

```cisco
no ip igmp snooping report-suppression
```

---

# 5.5 CGMP

**CGMP — Cisco Group Management Protocol**

CGMP was a Cisco proprietary mechanism used before IGMP Snooping became the common standard approach.

Basic concept:

```text
Host
 ↓
IGMP Join
 ↓
Router
 ↓
CGMP
 ↓
Switch
 ↓
Multicast MAC → Host Port
```

Important addresses:

```text
CGMP control:
01:00:0C:DD:DD:DD
```

---

## CGMP Join Concept

CGMP uses:

```text
USA = Unicast Source Address
GDA = Group Destination Address
```

Conceptually:

```text
USA → Client MAC
GDA → Multicast MAC
```

The switch learns:

```text
Multicast MAC
       ↓
Client port
```

---

## CGMP Verification

```cisco
debug ip cgmp
```

CGMP is considered a legacy Cisco technology and is generally replaced by IGMP Snooping.

---

# 5.6 RGMP

**RGMP — Router Group Management Protocol**

Multicast control mechanisms can also be used to prevent multicast traffic from being unnecessarily delivered to routers that do not need it.

RGMP uses:

```text
224.0.0.25
```

Example:

```cisco
interface GigabitEthernet0/0
 ip rgmp
```

Conceptual messages:

```text
HELLO
JOIN
LEAVE
BYE
```

---

# 6. IGMP Querier & PIM DR

Two concepts are often confused:

```text
IGMP Querier
        ≠
PIM Designated Router
```

---

## IGMP Querier

The IGMP querier sends IGMP queries to discover group membership.

For IGMPv2/v3, the router with the **lowest IP address** wins querier election.

Example:

```text
R1 = 10.1.1.1
R2 = 10.1.1.2
R3 = 10.1.1.3

IGMP Querier = R1
```

---

## PIM DR

PIM Designated Router is responsible for important multicast control-plane functions on a multi-access network.

PIM DR election can use:

```text
DR Priority
    ↓
Higher priority wins

If tied:
Higher IP wins
```

Example:

```cisco
interface GigabitEthernet0/0
 ip pim dr-priority 100
```

---

## Important Distinction

```text
IGMP Querier
    ↓
Lowest IP

PIM DR
    ↓
Highest DR priority
    ↓
Highest IP if tied
```

---

# 7. PIM Fundamentals

**PIM — Protocol Independent Multicast**

PIM uses the existing unicast routing table to perform multicast Reverse Path Forwarding decisions.

Therefore:

```text
OSPF
BGP
EIGRP
IS-IS
Static Route
       ↓
Unicast RIB
       ↓
RPF Decision
       ↓
Multicast Forwarding
```

---

## PIM Versions

### PIMv1

Legacy Cisco implementation.

Uses:

```text
224.0.0.2
```

### PIMv2

Standardized by RFC 4601.

Uses:

```text
224.0.0.13
```

PIM Hellos are typically sent every:

```text
30 seconds
```

Holdtime is commonly:

```text
105 seconds
```

PIM uses:

```text
IP protocol 103
```

---

# 7.1 PIM Operational Modes

```text
PIM-DM
PIM-SM
PIM-SSM
PIM-Bidir
```

---

# 8. PIM Dense Mode

PIM-DM follows a:

```text
Flood → Prune
```

model.

It is essentially a **push model**.

---

## PIM-DM Flow

```text
Source
  |
  ↓
Flood
  |
  +---- Router
  +---- Router
  +---- Router
  +---- Router
          |
          ↓
      No receivers
          |
        PRUNE
```

---

## Dense Mode Characteristics

### Initial Flood

Multicast traffic is initially forwarded across PIM-enabled interfaces except the incoming interface.

### RPF

The router validates the incoming interface using the unicast routing table.

### Prune

Routers without interested receivers prune themselves from the multicast tree.

### State Refresh

Prune state needs to be refreshed periodically.

### Graft

When a receiver appears on a previously pruned branch:

```text
GRAFT
  ↓
Upstream Router
  ↓
GRAFT-ACK
  ↓
Forwarding resumes
```

---

# 8.1 PIM-DM Assert

On multi-access networks, multiple routers may receive the same multicast traffic.

Example:

```text
          Source
             |
        +----+----+
        |         |
       R2        R3
        \         /
         \       /
          LAN
```

Both R2 and R3 may attempt forwarding.

PIM Assert resolves duplicate forwarding.

### Assert Selection

Winner:

1. Lowest Administrative Distance
2. Lowest metric
3. Highest IP address

Losers prune their forwarding state.

---

# 8.2 PIM-DM Prune Override

On shared LANs, a single router sending a prune should not immediately cause the entire segment to stop receiving traffic.

A short override window allows another router that still needs the traffic to maintain forwarding.

Conceptually:

```text
Router A → PRUNE
              |
           ~3 sec
              |
Router B → JOIN OVERRIDE
              |
         Continue traffic
```

---

# 8.3 PIM-DM State Refresh

State refresh periodically rebuilds/refreshes forwarding state and prune information.

Example Cisco configuration:

```cisco
interface GigabitEthernet0/0
 ip pim state-refresh origination-interval 60
```

Verification:

```cisco
show ip pim neighbor
show ip pim interface GigabitEthernet0/0 detail
```

---

# 8.4 PIM-DM Limitations

| Limitation  | Impact                                                                |
| ----------- | --------------------------------------------------------------------- |
| Flooding    | Wastes bandwidth                                                      |
| Pruning     | Generates control activity                                            |
| Scalability | Poor for large networks                                               |
| No RP       | Not suitable for ASM architectures requiring shared-tree coordination |
| Legacy      | Less common in modern deployments                                     |

> Modern enterprise multicast designs generally favor PIM-SM or SSM rather than PIM-DM.

---

# 9. PIM Sparse Mode

PIM-SM uses an:

```text
Explicit Join
```

model.

It is effectively a **pull model**.

```text
Receiver
   |
 IGMP Join
   |
Last-Hop Router
   |
PIM Join
   |
RP
```

No multicast traffic is normally forwarded toward a branch unless there is a reason for that branch to receive it.

---

# 9.1 PIM-SM Uses RP

For ASM:

```text
Receiver
    |
    | IGMP
    ↓
Last-Hop Router
    |
    | PIM Join
    ↓
   RP
```

The RP provides a shared-tree meeting point.

---

# 9.2 Shared Tree vs SPT

### RPT — Rendezvous Point Tree

Notation:

```text
(*,G)
```

Example:

```text
(*,239.1.1.1)
```

Tree is rooted at:

```text
RP
```

---

### SPT — Shortest Path Tree

Notation:

```text
(S,G)
```

Example:

```text
(10.1.1.10,239.1.1.1)
```

Tree is rooted at:

```text
Source
```

---

## Visual Model

```text
                 RP
                /  \
               /    \
              R2    R3
                    |
                  Receiver

              Shared Tree
                 (*,G)
```

After SPT switch:

```text
Source
  |
  R1
  |
  R2
  |
Receiver

       (S,G)
```

---

# 9.3 PIM-SM Source Registration

When a source starts transmitting:

```text
Source
  |
First-Hop Router / DR
  |
PIM Register
  |
  ↓
RP
```

The first multicast packet can be encapsulated in a PIM Register message and sent toward the RP.

If there are no receivers:

```text
RP
 ↓
REGISTER-STOP
 ↓
First-Hop Router
```

---

# 9.4 PIM-SM SPT Switchover

Initial path:

```text
Source
   ↓
RP
   ↓
Receiver
```

After source discovery:

```text
Source
   ↓
Shortest Path
   ↓
Receiver
```

This changes state from:

```text
(*,G)
```

to:

```text
(S,G)
```

The receiver-side router can send a PIM Join toward the source.

---

# 9.5 SPT Threshold

Cisco can control SPT switchover.

Example:

```cisco
ip pim spt-threshold 0
```

This can cause an immediate SPT transition when applicable.

---

# 10. Rendezvous Point

The RP is the coordination point for ASM multicast.

Think of RP as:

> **The meeting point for multicast sources and receivers.**

---

## RP Process

```text
Receiver
   |
 IGMP Report
   |
Last-Hop Router
   |
PIM Join
   |
   RP
```

Source:

```text
Source
   |
First-Hop Router
   |
PIM Register
   |
   RP
```

RP connects:

```text
Source
   ↕
  RP
   ↕
Receiver
```

---

# 10.1 Static RP

Simple static configuration:

```cisco
ip multicast-routing

interface GigabitEthernet0/0
 ip pim sparse-mode

ip pim rp-address 192.168.254.3
```

Verification:

```cisco
show ip pim rp mapping
show ip pim neighbor
show ip mroute
```

### Static RP Trade-off

```text
Simple
  +
Predictable
  -
Manual
  -
Limited redundancy
```

---

# 11. Auto-RP

Auto-RP is a Cisco mechanism for dynamic RP discovery.

Important addresses:

```text
224.0.1.39
224.0.1.40
```

### Roles

```text
RP Candidate
      ↓
RP Announcement
      ↓
Mapping Agent
      ↓
RP Discovery
      ↓
Multicast Routers
```

---

## Auto-RP Addresses

| Address      | Function        |
| ------------ | --------------- |
| `224.0.1.39` | RP announcement |
| `224.0.1.40` | RP discovery    |

---

## Auto-RP Example

RP candidate:

```cisco
access-list 1 permit 239.1.1.1

ip pim send-rp-announce Loopback0 scope 5 group-list 1
```

Mapping agent:

```cisco
ip pim send-rp-discovery Loopback0 scope 10
```

Interfaces may use:

```cisco
ip pim sparse-dense-mode
```

Auto-RP listener:

```cisco
ip pim autorp listener
```

Verification:

```cisco
show ip pim rp mapping
debug ip pim auto-rp
```

---

# 12. BSR

**BSR — Bootstrap Router**

BSR is a standards-based alternative to Cisco Auto-RP.

It is associated with PIMv2.

---

## BSR Roles

```text
Candidate BSR
      ↓
BSR Election
      ↓
Candidate RP
      ↓
RP-Set
      ↓
Multicast Routers
```

---

## BSR Election

Conceptually:

```text
Highest BSR priority
        ↓
Highest IP address if tied
```

---

## Candidate BSR

Example:

```cisco
ip pim bsr-candidate Loopback0
```

---

## Candidate RP

Example:

```cisco
access-list 1 permit 239.0.0.0 0.255.255.255

ip pim rp-candidate Loopback0 group-list 1
```

---

## Verification

```cisco
show ip pim bsr-router
show ip pim rp mapping
show ip pim rp mapping elected
show ip pim rp mapping inuse
debug ip pim bsr
```

---

# 12.1 RP Selection in BSR

When multiple RP candidates exist, selection considers:

1. Longest matching group prefix
2. RP priority
3. Hash value
4. IP address tie-breaker

### Longest Match Example

```text
RP1:
239.0.0.0/8

RP2:
239.1.1.1/32
```

For:

```text
239.1.1.1
```

The `/32` candidate is more specific.

---

## Hash-Based RP Selection

The hash mechanism provides deterministic RP selection and can distribute groups across multiple RPs.

Conceptually:

```text
Group
  +
Mask
  +
RP Candidate
  ↓
Hash
  ↓
Selected RP
```

Verification:

```cisco
show ip pim rp-hash 239.1.1.1
```

---

# 12.2 BSR Border

To prevent BSR information from escaping the intended multicast domain:

```cisco
interface GigabitEthernet0/0
 ip pim bsr-border
```

This is particularly important at domain boundaries.

---

# 13. Anycast RP

Anycast RP provides RP redundancy and locality.

Multiple physical routers use the same logical RP address.

Example:

```text
          Anycast RP
         10.10.10.10
          /        \
        R5          R6
        |            |
      Site A        Site B
```

Routers only need to know:

```text
10.10.10.10
```

Unicast routing determines which RP instance is closest.

---

## Anycast RP + MSDP

Multiple RP instances need to exchange information about active multicast sources.

This is where MSDP is used.

```text
R5                         R6
 |                          |
 +-------- MSDP ------------+
          TCP/639
```

---

# 14. MSDP

**MSDP — Multicast Source Discovery Protocol**

MSDP allows multicast domains/RPs to exchange source information.

Important for:

* Anycast RP
* Inter-domain multicast
* PIM-SM source discovery

---

## MSDP SA

MSDP uses **SA — Source Active** information.

An SA conveys information about:

```text
Source (S)
Group (G)
Originating RP
```

Conceptually:

```text
RP1
 |
 | SA
 ↓
RP2
 |
 | learns
 ↓
(S,G)
```

---

## MSDP Transport

MSDP uses:

```text
TCP port 639
```

---

## MSDP Peer Example

```cisco
ip msdp peer 6.6.6.6 connect-source Loopback0
```

Verification:

```cisco
show ip msdp peer
show ip msdp sa-cache
show ip msdp summary
show ip msdp count
```

---

# 14.1 MSDP Keepalive

Example:

```cisco
ip msdp keepalive 4.4.4.4 30 45
```

Conceptually:

```text
Keepalive interval = 30 sec
Holdtime = 45 sec
```

---

# 14.2 MSDP Security

MSDP peers can use authentication.

Example:

```cisco
ip msdp password 7.7.7.7 cisco
```

---

# 14.3 MSDP SA Filtering

Source-active information can be filtered.

Possible filtering mechanisms include:

```text
ACL
Route-map
RP-list
RP route-map
```

Example:

```cisco
ip msdp sa-filter out 5.5.5.5 1
```

---

# 14.4 MSDP SA Cache

Useful commands:

```cisco
show ip msdp sa-cache
show ip msdp accepted-sas
show ip msdp advertise-sas
show ip msdp rejected-sa
```

Optional cache controls:

```cisco
ip msdp cache-rejected-sa 2
```

---

# 14.5 MSDP Mesh Groups

With multiple MSDP peers, SA flooding can create unnecessary duplication.

Mesh groups reduce SA advertisement loops.

Example:

```cisco
ip msdp mesh-group A 4.4.4.4
ip msdp mesh-group A 7.7.7.7
```

---

# 14.6 MSDP + BGP

A common inter-domain design:

```text
        AS 65000
      +---------+
      |    RP   |
      +---------+
       /       \
     MSDP     MSDP
      /         \
 AS 65001      AS 65002
    RP            RP
```

Unicast reachability must still exist between multicast sources, receivers, RPs and relevant next hops.

---

# 15. Bidirectional PIM

Bidirectional PIM is designed for **many-to-many multicast applications**.

Typical examples:

* Video conferencing
* Financial market distribution
* Many-source multicast
* Collaborative applications

---

## Why Bidir-PIM?

PIM-SM may create:

```text
RPT
 +
SPT
 +
SPT Switchover
 +
(S,G) State
```

With many sources, this can become expensive.

Bidirectional PIM instead uses a shared bidirectional tree.

---

## PIM-SM vs Bidir-PIM

| Feature        | PIM-SM     | Bidir-PIM              |
| -------------- | ---------- | ---------------------- |
| ASM            | Yes        | Yes                    |
| RP             | Yes        | Yes                    |
| Register       | Yes        | No source registration |
| Register-Stop  | Yes        | No                     |
| SPT Switchover | Yes        | No                     |
| `(S,G)` state  | Common     | Avoided                |
| DF election    | No         | Yes                    |
| Many sources   | More state | More scalable          |

---

# 15.1 Bidirectional RP

The RP becomes a meeting point rather than a registration point.

```text
Source A
   \
    \
     RP
    /  \
   /    \
Source B Receiver
```

All sources can use the same bidirectional tree.

---

# 15.2 Designated Forwarder

Bidir-PIM uses **DF election** to prevent loops.

Conceptually:

```text
LAN
 |
 +--- R1
 |
 +--- R2
 |
 +--- R3
```

One router becomes:

```text
DF
```

The DF is responsible for forwarding multicast traffic toward the RP on the relevant segment.

---

## DF Election

General logic:

```text
Lowest metric
      ↓
If tie:
Highest IP address
```

DF election replaces the normal RPF-based forwarding behavior for bidirectional tree forwarding on the relevant interfaces.

---

## Bidirectional Configuration

Conceptual Cisco example:

```cisco
ip multicast-routing

interface GigabitEthernet0/0
 ip pim sparse-mode

ip pim bidir-enable
ip pim rp-address 3.3.3.3 bidir
```

---

# 16. PIM SSM

**Source-Specific Multicast**

SSM removes the requirement for an RP.

Architecture:

```text
Source
  |
  |
  +------ SPT ------ Receiver
```

No:

```text
RP
RPT
Register
Register-Stop
```

---

## SSM Address Range

```text
232.0.0.0/8
```

---

## SSM Join

Receiver specifies:

```text
(S,G)
```

Example:

```text
Source = 192.168.1.50
Group  = 232.1.1.1
```

---

## Cisco Example

```cisco
ip multicast-routing

interface GigabitEthernet0/0
 ip pim sparse-mode
 ip igmp version 3

ip pim ssm default
```

Or:

```cisco
ip pim ssm range 1

access-list 1 permit 232.0.0.0 0.255.255.255
```

Receiver:

```cisco
interface GigabitEthernet0/0
 ip igmp version 3
 ip igmp join-group 232.1.1.1 source 192.168.1.50
```

---

## SSM Tree

```text
             Source
             10.1.1.1
                |
               R1
                |
               R2
                |
             Receiver
```

Multicast state:

```text
(S,G)
```

Example:

```text
(10.1.1.1,232.1.1.1)
```

---

## SSM Advantages

```text
No RP
No RPT
No source registration
No MSDP
Direct source tree
Better source control
Reduced ASM complexity
```

---

# 17. Multicast Routing Tables

Multicast routing is different from normal unicast routing.

Typical Cisco command:

```cisco
show ip mroute
```

Example:

```text
(192.168.1.10,239.1.1.1)
```

Means:

```text
Source = 192.168.1.10
Group  = 239.1.1.1
```

---

## Common Multicast States

### `(*,G)`

```text
Any source → Group
```

Typical for:

```text
PIM-SM RPT
ASM
```

### `(S,G)`

```text
Specific source → Group
```

Typical for:

```text
SPT
SSM
```

---

# 17.1 Common Flags

Cisco multicast flags vary by platform/software release, but commonly indicate states such as:

```text
S → Sparse
D → Dense
S → SPT-related state depending on output/context
J → SPT Join
P → Pruned
T → SPT
F → Register/tunnel-related state
B → Bidirectional
M → MSDP-related source information
```

> Always interpret flags together with the full `show ip mroute` output and platform documentation.

---

# 18. RPF

**RPF — Reverse Path Forwarding**

RPF is one of the most important multicast loop-prevention mechanisms.

The router asks:

> "If I wanted to reach the multicast source using the unicast routing table, would I use this same interface?"

---

## RPF Example

```text
Source
  |
 R1
  |
 R2
  |
 R3
```

R3 receives multicast traffic from R2.

R3 checks its unicast route to:

```text
Source
```

If the best route says:

```text
Source → R2
```

and the multicast packet arrived from:

```text
R2
```

then:

```text
RPF PASS
```

Otherwise:

```text
RPF FAIL
```

---

## RPF Dependency

Multicast routing relies heavily on unicast reachability.

```text
Unicast RIB
     ↓
RPF lookup
     ↓
Multicast forwarding decision
```

Therefore:

> A multicast problem can actually be a unicast routing problem.

---

# 18.1 RPF Troubleshooting

Check:

```cisco
show ip route <source>
show ip route <rp>
show ip rpf <source>
show ip mroute
```

Common causes:

```text
Wrong unicast route
Asymmetric routing
Missing route
Incorrect next hop
Incorrect interface
Route redistribution issue
```

---

# 19. Multicast Troubleshooting

Use a layered approach.

```text
Application
    ↓
Host
    ↓
IGMP
    ↓
Switch / IGMP Snooping
    ↓
PIM
    ↓
RP / SSM
    ↓
RPF
    ↓
Multicast Routing Table
    ↓
Forwarding
```

---

## Troubleshooting Flow

### Step 1 — Is the source transmitting?

Check the application.

Example:

```text
VLC
UDP
239.192.20.1
Port 1234
```

---

### Step 2 — Is the receiver joining?

Check IGMP:

```cisco
show ip igmp groups
show ip igmp interface
```

---

### Step 3 — Is IGMP Snooping working?

```cisco
show ip igmp snooping
show ip igmp snooping groups
show ip igmp snooping querier
```

---

### Step 4 — Is PIM adjacency established?

```cisco
show ip pim neighbor
```

---

### Step 5 — Is the RP reachable?

```cisco
show ip pim rp mapping
show ip route <rp-address>
```

---

### Step 6 — Is the source reachable?

```cisco
show ip route <source>
show ip rpf <source>
```

---

### Step 7 — Check multicast state

```cisco
show ip mroute
```

Look for:

```text
(*,G)
(S,G)
Incoming Interface
Outgoing Interface List
RPF Neighbor
```

---

## Common Symptoms

| Symptom                      | Possible Cause                              |
| ---------------------------- | ------------------------------------------- |
| No multicast traffic         | No IGMP join                                |
| No `(S,G)`                   | Missing PIM/RPF/source state                |
| RPF failure                  | Wrong unicast route                         |
| Intermittent traffic         | RPF / topology / TTL                        |
| Traffic reaches wrong ports  | IGMP Snooping issue                         |
| ASM fails                    | RP problem                                  |
| SSM fails                    | IGMPv3 / source-specific configuration      |
| Inter-domain multicast fails | MSDP/BGP/unicast reachability               |
| Duplicate traffic            | PIM Assert problem                          |
| Traffic stops after leave    | Incorrect immediate-leave/snooping behavior |

---

# 20. FortiGate Multicast

FortiGate supports multicast routing and multicast forwarding.

The architecture should be clearly separated:

```text
Multicast Routing
        vs
Multicast Forwarding
```

These are not the same operation.

---

# 20.1 FortiGate Multicast Architecture

Example:

```text
          Multicast Source
                 |
              FGT-1
              /   \
             /     \
            RP     Network
             |
            FGT-2
             |
          Receivers
```

Example RP:

```text
22.22.22.1
```

---

# 20.2 Enable Multicast Routing

FortiGate multicast configuration is under:

```text
config router multicast
```

Example:

```cli
config router multicast
    set multicast-routing enable
end
```

---

# 20.3 PIM Interface

Conceptual configuration:

```cli
config router multicast
    config interface
        edit "port3"
            set pim-mode sparse-mode
            set igmp version 3
            set dr-priority 1
        next
    end
end
```

---

# 20.4 RP Configuration

Example:

```cli
config router multicast
    config rp-address
        edit 1
            set ip-address 192.168.254.252
        next
    end
end
```

---

# 20.5 FortiGate GUI

Multicast policy visibility:

```text
System
  ↓
Feature Visibility
  ↓
Multicast Policy
```

Then:

```text
Policy & Objects
  ↓
Multicast Policy
```

Multicast policies can be used to control:

```text
Source
Destination multicast group
Interface/path
```

---

# 20.6 FortiGate Multicast Policy

Example architecture:

```text
Multicast Router
       |
       ↓
   FortiGate
       |
       ↓
   Receiver
```

A multicast policy should explicitly allow the intended multicast flow.

Do not assume that ordinary unicast firewall policies automatically provide the desired multicast behavior.

---

# 20.7 FortiGate Verification

Useful commands:

```cli
get router info multicast table
get router info multicast table-count
get router info multicast igmp
get router info multicast igmp group
get router info multicast igmp group details
```

PIM:

```cli
get router info multicast pim sparse-mode neighbor
get router info multicast pim sparse-mode neighbor-detail
get router info multicast pim sparse-mode interface
get router info multicast pim sparse-mode interface-detail
get router info multicast pim sparse-mode next-hop
get router info multicast pim sparse-mode table
```

RP/BSR:

```cli
get router info multicast pim sparse-mode bsr-info
get router info multicast pim sparse-mode rp-mapping
```

---

# 20.8 FortiGate Multicast Diagnostics

Useful diagnostics:

```cli
diagnose ip multicast vif
diagnose ip multicast mac
diagnose ip multicast status
diagnose ip multicast mroute
```

Multicast Forwarding Cache:

```cli
diagnose ip multicast mfc-add
diagnose ip multicast mfc-del
```

---

# 20.9 Multicast Forwarding vs Multicast Routing

This distinction is critical.

### Multicast Forwarding

Used when FortiGate acts as a device forwarding multicast traffic between multicast routers/receivers.

### Multicast Routing

Used when FortiGate itself participates in multicast routing protocols such as PIM.

Conceptually:

```text
FortiGate as multicast router
        ↓
Multicast Routing
        ↓
PIM / IGMP
```

versus:

```text
FortiGate forwarding multicast
        ↓
Multicast Forwarding
```

Do not enable both mechanisms without understanding their roles.

---

# 20.10 FortiGate Multicast Forwarding

Configuration:

```cli
config system settings
    set multicast-forward enable
end
```

The exact behavior and interaction with multicast routing should be validated against the FortiOS version being deployed.

---

## Multicast TTL Behavior

FortiGate can preserve multicast TTL behavior using:

```cli
set multicast-ttl-notchange enable
```

Conceptually:

```text
TTL preservation
       ↓
Do not modify multicast TTL
```

This can matter when multicast traffic is unexpectedly expiring before reaching the destination.

---

# 20.11 Transparent Mode

When FortiGate operates transparently, multicast traffic can be forwarded at Layer 2.

A setting such as:

```cli
config system settings
    set multicast-skip-policy disable
end
```

can affect multicast policy handling in transparent mode.

> ⚠️ Validate the exact behavior for the FortiOS release and operating mode before applying in production.

---

# 20.12 Enhanced MAC VLAN

Multicast forwarding is not supported on enhanced MAC VLAN interfaces.

This is an important design constraint when selecting FortiGate interface types for multicast deployments.

---

# 21. FortiGate Multicast Global Configuration

Example structure:

```cli
config router multicast
    set route-limit 2147483647
    set multicast-routing enable

    config pim-sm-global
        set message-interval 60
        set join-prune-holdtime 210

        set accept-register-list ''
        set accept-source-list ''

        set bsr-candidate disable
        set bsr-allow-quick-refresh disable

        set cisco-register-checksum disable
        set cisco-crp-prefix disable
        set cisco-ignore-rp-set-priority disable

        set register-rp-reachability enable
        set register-source disable

        set register-supression 60
        set null-register-retries 1
        set rp-register-keepalive 185

        set register-rate-limit 0
        set spt-threshold enable
        set ssm disable
    end
end
```

---

# 21.1 Important FortiGate Parameters

| Parameter                  | Purpose                                       |
| -------------------------- | --------------------------------------------- |
| `multicast-routing`        | Enables multicast routing                     |
| `route-limit`              | Controls multicast route/resource limits      |
| `message-interval`         | PIM message interval                          |
| `join-prune-holdtime`      | Join/Prune state holdtime                     |
| `accept-register-list`     | Controls accepted PIM Register sources        |
| `accept-source-list`       | Controls accepted multicast sources           |
| `bsr-candidate`            | BSR candidate functionality                   |
| `register-rp-reachability` | Checks RP reachability                        |
| `register-source`          | Controls source-address registration behavior |
| `register-supression`      | Register suppression interval                 |
| `rp-register-keepalive`    | Register state keepalive                      |
| `ssm`                      | Enables/disables SSM functionality            |
| `spt-threshold`            | Controls SPT behavior                         |

---

# 21.2 FortiGate Multicast Interface

Example:

```cli
config router multicast
    config interface
        edit "port1"
            set ttl-threshold 1
            set pim-mode sparse-mode
            set passive disable
            set bfd disable
            set neighbour-filter ''
            set hello-interval 30
            set rp-candidate disable
            set multicast-flow ''
            set static-group ''
            set rpf-nbr-fail-back disable

            config join-group
                edit 239.120.20.1
                next
            end

            config igmp
                set access-group ''
                set version 3
                set query-max-response-time 10
                set query-interval 125
                set query-timeout 255
                set router-alert-check disable
                set immediate-leave-group ''
                set last-member-query-interval 1000
                set last-member-query-count 2
            end

            set dr-priority 1
        next
    end
end
```

---

# 21.3 TTL Threshold

Example:

```cli
set ttl-threshold 1
```

Concept:

```text
TTL threshold = 1
```

can be used when multicast sources/receivers are directly attached and you want to limit multicast propagation.

If multicast must cross several routed hops, a higher appropriate TTL/threshold design may be required.

---

# 21.4 Passive PIM

Example:

```cli
set passive disable
```

Active PIM operation allows neighbor discovery and PIM control exchange.

```text
Active
  ↓
PIM Hello
  ↓
Neighbor discovery
```

---

# 21.5 IGMP Parameters

Example:

```cli
config igmp
    set version 3
    set query-max-response-time 10
    set query-interval 125
    set query-timeout 255
    set last-member-query-interval 1000
    set last-member-query-count 2
end
```

Important concepts:

```text
Version
Query interval
Query response time
Querier timeout
Last-member query interval
Last-member query count
```

---

# 22. FortiGate MFC

**MFC — Multicast Forwarding Cache**

MFC represents the forwarding state used for multicast packet replication.

Conceptually:

```text
IGMP Join
    ↓
(*,G) state
    ↓
Source traffic
    ↓
(S,G) state
    ↓
RPF validation
    ↓
Outgoing VIFs
```

---

# 22.1 MFC Example

Suppose:

```text
Source:
192.168.20.200

Group:
239.192.20.1

Incoming VIF:
3

Outgoing VIF:
2
```

Conceptually:

```text
VIF 3
  ↓
Source 192.168.20.200
  ↓
239.192.20.1
  ↓
VIF 2
```

---

# 22.2 MFC vs MRIB

| Component | Function                                                       |
| --------- | -------------------------------------------------------------- |
| MRIB      | Multicast Routing Information Base / control-plane information |
| MFC       | Multicast Forwarding Cache / forwarding state                  |

Think:

```text
MRIB
 ↓
Control Plane
 ↓
Routing Decision

MFC
 ↓
Forwarding Plane
 ↓
Packet Replication
```

---

# 22.3 MFC Diagnostics

Inspect multicast forwarding state:

```cli
diagnose ip multicast list
```

Flush MFC:

```cli
diagnose ip multicast mfc-flush
```

Useful after:

```text
RP change
PIM change
IGMP state problem
Stale multicast state
Testing multicast reconstruction
```

> Flushing MFC can temporarily interrupt traffic while multicast state is rebuilt.

---

# 22.4 Manual MFC Entry

Advanced lab example:

```cli
diagnose ip multicast mfc-add 192.168.20.200 239.192.20.1 3 2
```

Meaning:

```text
Source:
192.168.20.200

Group:
239.192.20.1

Incoming VIF:
3

Outgoing VIF:
2
```

This can be useful for controlled testing.

> ⚠️ Manual MFC manipulation can bypass normal control-plane/RPF behavior and should generally be restricted to troubleshooting/lab scenarios.

---

# 22.5 MFC Deletion

```cli
diagnose ip multicast mfc-del
```

Use carefully because removing forwarding state can interrupt active multicast streams.

---

# 22.6 Multicast VIF

VIF means:

```text
Virtual Interface
```

Multicast routing uses VIF identifiers to represent multicast interfaces.

Useful command:

```cli
diagnose ip multicast vif
```

Use VIF information when interpreting MFC entries.

---

# 22.7 Multicast TTL Diagnostics

```cli
diagnose ip multicast ttl-threshold
```

Use this when investigating multicast TTL propagation and hop-limit-related forwarding behavior.

---

# 23. FortiGate Multicast Lab

## Topology

```text
                  Multicast Source
                  239.192.20.1
                        |
                        |
                  +-----------+
                  |   R1      |
                  +-----------+
                        |
                    22.22.22.1
                        |
                  +-----------+
                  |  FGT-1 RP |
                  +-----------+
                        |
                        |
                  +-----------+
                  |  FGT-2    |
                  +-----------+
                        |
                     LAN
                        |
                     Client
```

---

## Cisco Router

```cisco
enable
configure terminal

hostname R1

ip multicast-routing

interface GigabitEthernet0/0
 ip address dhcp
 ip pim sparse-mode
 ip igmp version 3
 no shutdown

interface GigabitEthernet0/1
 ip address 192.168.20.254 255.255.255.0
 ip pim sparse-mode
 ip igmp version 3
 no shutdown

ip route 0.0.0.0 0.0.0.0 22.22.22.1

ip pim rp-address 22.22.22.1
```

Verification:

```cisco
show ip pim neighbor
show ip igmp groups
show ip igmp membership
show ip mroute
```

---

# 23.1 VLC Multicast Server

Example:

```text
Multicast Group:
239.192.20.1

UDP Port:
1234
```

The server sends:

```text
UDP → 239.192.20.1:1234
```

---

# 23.2 VLC Receiver

Example URI:

```text
udp://@239.192.20.1:1234
```

Flow:

```text
Client
  |
IGMP Join
  |
FGT
  |
PIM
  |
Multicast Source
```

---

# 24. Multicast Design Best Practices

## 24.1 Prefer PIM-SM / SSM for Modern Designs

General design direction:

```text
ASM
 ↓
PIM-SM
 ↓
RP

SSM
 ↓
IGMPv3
 ↓
(S,G)
 ↓
No RP
```

---

## 24.2 Prefer SSM Where Possible

SSM reduces several ASM dependencies:

```text
No RP
No RPT
No Register
No Register-Stop
No MSDP
```

Architecture:

```text
Source
  |
  ↓
(S,G)
  |
  ↓
Receiver
```

---

## 24.3 Enable IGMP Snooping

Without snooping:

```text
Multicast
 ↓
Switch
 ↓
Potential flooding
```

With snooping:

```text
IGMP Join
 ↓
Switch learns interested ports
 ↓
Only required ports receive traffic
```

---

## 24.4 Keep Unicast Routing Healthy

Because RPF relies on unicast routing:

```text
Bad IGP/BGP
    ↓
Bad RPF
    ↓
Multicast failure
```

Always verify:

```text
Source reachability
RP reachability
RPF interface
Next hop
```

---

## 24.5 Design RP Redundancy

For ASM:

```text
Single RP
   ↓
Single failure domain
```

Consider:

```text
Anycast RP
+
MSDP
```

or an appropriate platform-supported RP redundancy mechanism.

---

## 24.6 Control Multicast Boundaries

Use:

```text
239.0.0.0/8
```

for administrative/internal multicast applications where appropriate.

At boundaries, control:

```text
PIM
BSR
Auto-RP
IGMP
Multicast groups
TTL
Source filtering
```

---

# 25. Multicast Protocol Interaction

The entire multicast architecture can be visualized as:

```text
                   APPLICATION
                       |
                       |
                 Multicast UDP
                       |
                       ↓
                    IGMP
                       |
              Host ←→ Router
                       |
                       ↓
                  PIM Control
                       |
            +----------+----------+
            |                     |
           ASM                   SSM
            |                     |
           RP                    (S,G)
            |                     |
       RPT (*,G)                  SPT
            |
       SPT Switchover
            |
        (S,G)
```

---

# 26. Multicast Control Plane vs Data Plane

## Control Plane

Responsible for determining:

```text
Who wants the traffic?
Which source?
Which group?
Which RP?
Which path?
Which interface?
```

Protocols:

```text
IGMP
PIM
MSDP
Auto-RP
BSR
```

---

## Data Plane

Responsible for:

```text
Receive
Validate
Replicate
Forward
```

Conceptually:

```text
Control Plane
     ↓
Build multicast state
     ↓
Data Plane
     ↓
Forward packets
```

---

# 27. High-Value Multicast Address Table

| Address       | Meaning               |
| ------------- | --------------------- |
| `224.0.0.1`   | All multicast hosts   |
| `224.0.0.2`   | All multicast routers |
| `224.0.0.4`   | DVMRP                 |
| `224.0.0.5`   | OSPF routers          |
| `224.0.0.6`   | OSPF DR/BDR           |
| `224.0.0.9`   | RIPv2                 |
| `224.0.0.10`  | EIGRP                 |
| `224.0.0.13`  | PIM routers           |
| `224.0.0.22`  | IGMPv3                |
| `224.0.0.25`  | RGMP                  |
| `224.0.1.39`  | Auto-RP announcement  |
| `224.0.1.40`  | Auto-RP discovery     |
| `232.0.0.0/8` | SSM                   |
| `233.0.0.0/8` | GLOP                  |
| `239.0.0.0/8` | Administrative scope  |

---

# 28. High-Value Protocol Numbers

| Protocol |             Value |
| -------- | ----------------: |
| IGMP     |   IP Protocol `2` |
| PIM      | IP Protocol `103` |
| MSDP     |         TCP `639` |

---

# 29. IGMP Version  

| Feature            | IGMPv1                | IGMPv2 | IGMPv3                    |
| ------------------ | --------------------- | ------ | ------------------------- |
| Membership Query   | Yes                   | Yes    | Yes                       |
| Membership Report  | Yes                   | Yes    | Yes                       |
| Leave Message      | No                    | Yes    | Yes                       |
| Querier Election   | Basic/legacy behavior | Yes    | Yes                       |
| Source Filtering   | No                    | No     | Yes                       |
| ASM                | Yes                   | Yes    | Yes                       |
| SSM                | No                    | No     | Yes                       |
| Report Suppression | Basic                 | Yes    | Advanced source filtering |

---

# 30. PIM Mode  

| Feature     | PIM-DM                  | PIM-SM                  | PIM-SSM              | Bidir-PIM            |
| ----------- | ----------------------- | ----------------------- | -------------------- | -------------------- |
| Model       | Push                    | Pull                    | Explicit source join | Shared bidirectional |
| Flood       | Yes                     | No                      | No                   | No                   |
| Prune       | Yes                     | Controlled joins/prunes | Join/prune           | Join/prune           |
| RP          | No                      | Yes                     | No                   | Yes                  |
| RPT         | No                      | Yes                     | No                   | Shared tree          |
| SPT         | Yes/source-based        | Yes                     | Yes                  | No                   |
| Register    | No                      | Yes                     | No                   | No                   |
| MSDP        | No                      | Can use                 | No                   | No                   |
| DF Election | No                      | No                      | No                   | Yes                  |
| Best use    | Legacy/specific designs | ASM                     | Source-specific apps | Many-to-many         |

---

# 31. Multicast State 

```text
(*,G)
```

Means:

```text
Any Source → Group
```

Usually associated with:

```text
ASM
RPT
RP
```

---

```text
(S,G)
```

Means:

```text
Specific Source → Group
```

Usually associated with:

```text
SPT
SSM
IGMPv3
```

---

# 32. Multicast Packet Flow — ASM

```text
                 SOURCE
                   |
                   |
              First-Hop Router
                   |
              PIM Register
                   |
                   ↓
                   RP
                   |
             Shared Tree
              (*,G)
                   |
          +--------+--------+
          |                 |
        Router            Router
          |                 |
      Receiver           Receiver
```

Then:

```text
Receiver Router
       |
       | (S,G) Join
       ↓
    Source Path
       |
       ↓
      SPT
```

---

# 33. Multicast Packet Flow — SSM

```text
SOURCE
192.168.1.50
     |
     |
     ↓
   Router
     |
     |
     ↓
 Receiver
```

Receiver requests:

```text
(S,G)
```

Example:

```text
(192.168.1.50,232.1.1.1)
```

No:

```text
RP
RPT
Register
MSDP
```

---

# 34. Multicast Packet Flow — Bidir-PIM

```text
             Source A
                 |
                 |
                 ↓
                RP
              /    \
             /      \
        Source B   Receiver
```

One shared bidirectional tree handles traffic from multiple sources.

```text
No Register
No SPT Switchover
No per-source tree requirement
```

DF election controls forwarding toward the RP.

---

# 35. Multicast Security Considerations

Multicast is not automatically "trusted" simply because it uses group addresses.

Consider controlling:

```text
Who can join?
Which groups can be joined?
Which sources are allowed?
Which interfaces can forward?
Which multicast boundaries exist?
```

Useful controls include:

```text
IGMP access-group
PIM source filtering
RP filtering
MSDP SA filtering
FortiGate multicast policies
TTL boundaries
239.0.0.0/8 administrative scope
```

---

# 36. Fast Troubleshooting Decision Tree

```text
                 Multicast not working
                         |
                         ↓
                Is source transmitting?
                    /          \
                  NO            YES
                  |              |
             Fix source      IGMP Join?
                                /    \
                              NO      YES
                              |        |
                         Fix IGMP    PIM Neighbor?
                                       /       \
                                     NO         YES
                                     |           |
                                Fix PIM       RP/RPF?
                                                 /   \
                                               NO     YES
                                               |       |
                                          Fix RP/RPF  Mroute?
                                                        |
                                                        ↓
                                                  OIF exists?
                                                   /       \
                                                 NO         YES
                                                 |           |
                                          Fix multicast   Check
                                            state/policy  firewall/L2
```

---

# 37. Cisco Verification Command Map

## IGMP

```cisco
show ip igmp groups
show ip igmp interface
show ip igmp membership
```

## PIM

```cisco
show ip pim neighbor
show ip pim interface
show ip pim interface detail
```

## Multicast Routing

```cisco
show ip mroute
show ip rpf <source>
```

## RP

```cisco
show ip pim rp mapping
show ip pim rp mapping inuse
```

## Auto-RP

```cisco
debug ip pim auto-rp
```

## BSR

```cisco
show ip pim bsr-router
show ip pim rp mapping elected
show ip pim rp mapping inuse
debug ip pim bsr
```

## MSDP

```cisco
show ip msdp peer
show ip msdp sa-cache
show ip msdp summary
show ip msdp count
```

## IGMP Snooping

```cisco
show ip igmp snooping
show ip igmp snooping groups
show ip igmp snooping querier
```

---

# 38. FortiGate Verification Command Map

## Multicast

```cli
get router info multicast table
get router info multicast table-count
```

## IGMP

```cli
get router info multicast igmp
get router info multicast igmp group
get router info multicast igmp group details
```

## PIM

```cli
get router info multicast pim sparse-mode neighbor
get router info multicast pim sparse-mode neighbor-detail
get router info multicast pim sparse-mode interface
get router info multicast pim sparse-mode interface-detail
get router info multicast pim sparse-mode next-hop
get router info multicast pim sparse-mode table
```

## RP / BSR

```cli
get router info multicast pim sparse-mode bsr-info
get router info multicast pim sparse-mode rp-mapping
```

## Kernel / Forwarding Diagnostics

```cli
diagnose ip multicast vif
diagnose ip multicast mac
diagnose ip multicast status
diagnose ip multicast mroute
diagnose ip multicast list
```

## MFC

```cli
diagnose ip multicast mfc-add
diagnose ip multicast mfc-del
diagnose ip multicast mfc-flush
```

---

# 39. Interview & NSE Quick Questions

### What is the IPv4 multicast range?

```text
224.0.0.0 – 239.255.255.255
```

### What is the SSM range?

```text
232.0.0.0/8
```

### What is the administratively scoped range?

```text
239.0.0.0/8
```

### What is the multicast MAC prefix for IPv4?

```text
01:00:5E
```

### How many IP bits map to an IPv4 multicast MAC?

```text
23 bits
```

### How many IPv4 multicast IP addresses can map to one MAC?

```text
32
```

### What protocol number does IGMP use?

```text
2
```

### What protocol number does PIM use?

```text
103
```

### What TCP port does MSDP use?

```text
639
```

### What is `(*,G)`?

```text
Any Source, Group
```

### What is `(S,G)`?

```text
Specific Source, Group
```

### Which IGMP version supports source filtering?

```text
IGMPv3
```

### Which multicast model does IGMPv3 enable?

```text
SSM
```

### Does SSM require an RP?

```text
No
```

### Does PIM-DM use an RP?

```text
No
```

### Does PIM-SM use an RP for ASM?

```text
Yes
```

### What does RPF protect against?

```text
Multicast forwarding loops
```

### What determines RPF?

```text
Unicast routing information
```

### Who wins IGMPv2/v3 querier election?

```text
Lowest IP address
```

### Who wins PIM DR election?

```text
Highest DR priority
→ Highest IP if tied
```

### Who wins PIM Assert?

```text
Lowest AD
→ Lowest metric
→ Highest IP
```

### What replaces RPF-based forwarding selection on bidirectional PIM segments?

```text
DF election
```

---

# 40. One-Page Mental Model

```text
                    MULTICAST
                        |
        +---------------+---------------+
        |                               |
      ASM                              SSM
        |                               |
   (*,G) / (S,G)                     (S,G)
        |                               |
     PIM-SM                         IGMPv3
        |                               |
       RP                              No RP
        |
   +----+----+
   |         |
  RPT       SPT
 (*,G)     (S,G)
   |
Register
Register-Stop
```

---

## Layer-by-Layer Model

```text
L7
Application
  |
  | UDP multicast
  ↓
L3
IGMP
  |
  | Group membership
  ↓
PIM
  |
  | Tree construction
  ↓
RPF
  |
  | Source-path validation
  ↓
Multicast Routing Table
  |
  ↓
MFC / Forwarding
  |
  ↓
L2
IGMP Snooping
  |
  ↓
Multicast MAC
```

---

# 41. Final Multicast 

```text
IPv4 Multicast
├── 224.0.0.0/24
│   └── Local control
│
├── 224.0.1.0/24
│   └── Routed control multicast
│
├── 232.0.0.0/8
│   └── SSM
│
├── 233.0.0.0/8
│   └── GLOP
│
└── 239.0.0.0/8
    └── Administrative scope
```

```text
IGMP
├── v1
│   ├── Query
│   └── Report
│
├── v2
│   ├── Query
│   ├── Report
│   └── Leave
│
└── v3
    ├── Source filtering
    ├── INCLUDE
    ├── EXCLUDE
    └── SSM
```

```text
PIM
├── Dense Mode
│   └── Flood → Prune
│
├── Sparse Mode
│   ├── RP
│   ├── RPT
│   ├── Register
│   └── SPT
│
├── SSM
│   └── (S,G)
│
└── Bidirectional
    ├── Shared tree
    ├── No register
    └── DF election
```

```text
RP Discovery
├── Static RP
├── Auto-RP
├── BSR
└── Anycast RP
      └── MSDP
```

```text
Layer 2 Multicast
├── Multicast MAC
├── IGMP Snooping
├── CGMP
└── RGMP
```

```text
FortiGate
├── Multicast Routing
├── PIM
├── IGMP
├── RP / BSR
├── Multicast Policy
├── VIF
├── MRIB
└── MFC
```

---

## ⭐ Core Takeaways

> **1. IGMP answers:**
> "Which receivers want this multicast group?"

> **2. PIM answers:**
> "How should routers build the multicast distribution tree?"

> **3. RP answers:**
> "Where do ASM sources and receivers initially meet?"

> **4. RPF answers:**
> "Did this multicast packet arrive from the correct path toward its source?"

> **5. IGMP Snooping answers:**
> "Which Layer 2 switch ports actually need this multicast stream?"

> **6. MFC answers:**
> "Where should this multicast packet be replicated and forwarded?"

> **7. SSM answers:**
> "I want traffic from this exact source for this exact multicast group."

> **8. Bidir-PIM answers:**
> "How can many sources efficiently share one bidirectional multicast tree?"

---

## 🔥 The Multicast Formula

```text
Receiver
   ↓
IGMP
   ↓
PIM
   ↓
RP / SSM
   ↓
RPF
   ↓
Multicast Routing State
   ↓
MFC / Forwarding
   ↓
IGMP Snooping
   ↓
Receiver
```

**If multicast breaks, troubleshoot exactly in this order.**
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

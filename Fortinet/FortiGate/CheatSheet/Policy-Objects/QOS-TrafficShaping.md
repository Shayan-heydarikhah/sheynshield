# FortiGate QoS, Traffic Shaping & DSCP  

> **FortiOS Focus:** 7.x
> **Level:** NSE 4 / NSE 7 / Network Security Engineer
> **Topics:** QoS · Traffic Shaping · Policing · Shapers · DSCP · TOS · WRED · Burst · Queueing · Per-IP · Multicast · NPU Offload

---

## 📌 Quick Navigation

* [1. QoS Fundamentals](#1-qos-fundamentals)
* [2. Bandwidth vs Link Rate vs Access Rate](#2-bandwidth-vs-link-rate-vs-access-rate)
* [3. Delay Components](#3-delay-components)
* [4. Packet Loss & Queueing](#4-packet-loss--queueing)
* [5. Shaping vs Policing](#5-shaping-vs-policing)
* [6. Token Bucket](#6-token-bucket)
* [7. Cisco Policing Examples](#7-cisco-policing-examples)
* [8. TOS / IP Precedence / DSCP](#8-tos--ip-precedence--dscp)
* [9. DSCP Classification](#9-dscp-classification)
* [10. FortiGate Traffic Shaping](#10-fortigate-traffic-shaping)
* [11. Shared Shaper](#11-shared-shaper)
* [12. Per-IP Shaper](#12-per-ip-shaper)
* [13. Reverse Shaping](#13-reverse-shaping)
* [14. Traffic Shaping Profiles](#14-traffic-shaping-profiles)
* [15. Burst & C-Burst](#15-burst--c-burst)
* [16. WRED](#16-wred)
* [17. DSCP Marking on FortiGate](#17-dscp-marking-on-fortigate)
* [18. Multi-Stage DSCP](#18-multi-stage-dscp)
* [19. DSCP Matching](#19-dscp-matching)
* [20. QoS Packet Processing](#20-qos-packet-processing)
* [21. Debug & Verification](#21-debug--verification)
* [22. Voice QoS](#22-voice-qos)
* [23. Video QoS](#23-video-qos)
* [24. LFI](#24-link-fragmentation-and-interleaving)
* [25. Multicast & Traffic Shaping](#25-multicast--traffic-shaping)
* [26. Global Traffic Priority](#26-global-traffic-priority)
* [27. NPU Acceleration](#27-npu-acceleration)
* [28. Common Design Rules](#28-common-design-rules)
* [29. Troubleshooting Checklist](#29-troubleshooting-checklist)
* [30. Exam Quick Recall](#30-exam-quick-recall)

---

# 1. QoS Fundamentals

## What is QoS?

**Quality of Service (QoS)** controls how network traffic is:

* Classified
* Marked
* Queued
* Prioritized
* Shaped
* Policed
* Dropped

The main QoS objectives are:

```text
Bandwidth
Latency
Jitter
Packet Loss
Congestion Management
Congestion Avoidance
```

### Typical priority

```text
Voice
  ↓
Video / Interactive
  ↓
Business-Critical Data
  ↓
Best-Effort Data
  ↓
Bulk / Background Traffic
```

---

# 2. Bandwidth vs Link Rate vs Access Rate

These terms are often confused.

| Term        | Meaning                                                   |
| ----------- | --------------------------------------------------------- |
| Bandwidth   | Configured logical value used by routing/QoS calculations |
| Link Rate   | Physical/interface transmission capability                |
| Access Rate | Provisioned/access speed of the circuit                   |
| CIR         | Committed Information Rate                                |
| PIR         | Peak Information Rate                                     |

### Important

A QoS bandwidth value does **not necessarily represent the physical interface speed**.

Example:

```text
Physical link = 1 Gbps
ISP CIR       = 100 Mbps
Required QoS  = 100 Mbps
```

The physical interface may be capable of 1 Gbps, while the ISP contract only permits 100 Mbps.

---

## Cisco bandwidth example

```cisco
interface Serial0/0/0
 clock rate 64000
 bandwidth 64
 no shutdown
```

Verify:

```cisco
show controllers serial 0/0/0
show interface serial 0/0/0
```

### Design rule

Routing protocols such as OSPF/EIGRP can use configured bandwidth values in their calculations.

Therefore:

> Don't confuse the physical/access rate with the logical `bandwidth` parameter.

---

# 3. Delay Components

Total network delay can be considered as:

```text
Total Delay ≈

Serialization Delay
+ Propagation Delay
+ Processing/Forwarding Delay
+ Queueing Delay
+ Shaping Delay
+ Network/ISP Delay
+ Codec Delay
```

---

## 3.1 Serialization Delay

Time required to place all packet bits onto the link.

### Formula

```text
Serialization Delay =
(Packet Size × 8) / Link Rate
```

Example:

```text
Packet = 1500 bytes
Link   = 64 kbps

(1500 × 8) / 64000
= 0.1875 sec
= 187.5 ms
```

This is extremely high for voice.

Smaller packets reduce serialization delay.

Example:

```text
60 bytes × 8 / 64000
= 7.5 ms
```

### Key point

```text
Higher link speed
        ↓
Lower serialization delay
```

---

## 3.2 Propagation Delay

Time required for the signal to physically travel through the medium.

```text
Propagation Delay =
Distance / Signal Propagation Speed
```

QoS generally cannot significantly reduce propagation delay.

---

## 3.3 Queueing Delay

Time spent waiting inside a queue.

Default queueing behavior may be FIFO.

```text
High Priority
     ↓
Priority Queue
     ↓
Normal Queue
```

Queueing delay is one of the major areas QoS can control.

---

## 3.4 Processing / Forwarding Delay

Time required by the device to process and forward traffic.

Modern hardware forwarding makes this delay very small.

---

## 3.5 Shaping Delay

Shaping intentionally delays packets to smooth traffic.

Example:

```text
Physical capacity = 128 kbps
Configured CIR     = 64 kbps
```

Traffic exceeding the shaping rate is queued and transmitted later.

```text
Burst
████████████

        ↓ shaping

████ ████ ████ ████
```

### Important

Shaping:

```text
QUEUE → DELAY → TRANSMIT
```

Policing:

```text
EXCEED → DROP / REMARK
```

---

# 4. Packet Loss & Queueing

Packet loss can occur because of:

* Physical errors
* FCS errors
* Congestion
* Queue overflow
* Tail drop
* Policing
* WRED
* Interface errors

---

## Tail Drop

If a queue is full:

```text
Queue size = 50 packets

Packet 1 → Queue
Packet 2 → Queue
...
Packet 50 → Queue
Packet 51 → DROP
```

Large queues can reduce packet loss but increase latency.

This creates the classic:

> **Bufferbloat problem**

---

# 5. Shaping vs Policing

This distinction is fundamental.

| Feature                   | Shaping            | Policing                             |
| ------------------------- | ------------------ | ------------------------------------ |
| Main action               | Delay/queue        | Drop/remark                          |
| Usually applied           | Egress             | Ingress or egress depending platform |
| Buffers excess traffic    | Yes                | No                                   |
| Smooths traffic           | Yes                | No                                   |
| Can increase latency      | Yes                | Usually no queueing delay            |
| Useful for WAN congestion | Yes                | Yes                                  |
| Can remark traffic        | Platform dependent | Common                               |

---

## Traffic Shaping

```text
Traffic
   ↓
Classifier
   ↓
Queue
   ↓
Controlled transmission
   ↓
WAN
```

Traffic above the configured rate is normally queued.

---

## Policing

```text
Traffic
   ↓
Rate Check
   ↓
Within limit → TRANSMIT
Above limit   → DROP / REMARK
```

### Typical ISP scenario

```text
Customer access = 1 Gbps
ISP CIR         = 500 Mbps
```

A policer can enforce:

```text
≤ 500 Mbps → transmit
> 500 Mbps → drop/remark
```

---

# 6. Token Bucket

Token bucket is a fundamental mechanism for traffic policing and shaping.

Imagine a bucket containing tokens.

```text
        Token Generator
              ↓
        ┌─────────────┐
        │   TOKENS    │
        │             │
        └─────────────┘
              ↓
           Packet
```

A packet needs enough tokens to transmit.

Typically:

```text
1 token ≈ 1 byte
```

depending on implementation.

---

## Token Bucket Parameters

| Parameter | Meaning         |
| --------- | --------------- |
| CIR       | Committed rate  |
| PIR       | Peak rate       |
| BC        | Committed Burst |
| BE / CBE  | Excess Burst    |

---

# 7. Cisco Policing Examples

## 7.1 Two-Color Policing

Two outcomes:

```text
CONFIRM → TRANSMIT
EXCEED  → DROP
```

Example:

```cisco
policy-map P1
 class class-default
  police 64000 confirm-action transmit exceed-action drop
```

Apply:

```cisco
interface GigabitEthernet0/0
 service-policy input P1
```

Verify:

```cisco
show policy-map
show policy-map interface GigabitEthernet0/0
```

---

## 7.2 Three-Color Policing

Three outcomes:

```text
CONFIRM   → TRANSMIT
EXCEED    → REMARK
VIOLATE   → DROP
```

Example:

```cisco
policy-map P2
 class class-default
  police 64000 4000 4000 \
   conform-action transmit \
   exceed-action set-dscp-transmit af13 \
   violate-action drop
```

Concept:

```text
Normal traffic
      ↓
  CONFIRM
      ↓
 TRANSMIT

Higher traffic
      ↓
  EXCEED
      ↓
 REMARK DSCP

Too much traffic
      ↓
 VIOLATION
      ↓
   DROP
```

---

## Single-Rate vs Dual-Rate

### Single Rate

Usually based on:

```text
CIR
BC
BE
```

### Dual Rate

Uses:

```text
CIR
PIR
BC
BE/CBE
```

Conceptually:

```text
CIR → committed traffic
PIR → peak traffic
```

---

# 8. TOS / IP Precedence / DSCP

The IPv4 TOS byte is historically used for QoS-related markings.

Modern interpretation:

```text
IPv4 TOS byte

  8 bits
┌──────────────┐
│ DSCP │  ECN  │
│ 6bit │ 2bit  │
└──────────────┘
```

---

## IP Precedence

IP Precedence uses the first 3 bits.

```text
Binary   Decimal   Meaning
000         0      Routine
001         1      Priority
010         2      Immediate
011         3      Flash
100         4      Flash Override
101         5      Critical
110         6      Internetwork Control
111         7      Network Control
```

---

# 9. DSCP Classification

DSCP uses:

```text
6 bits
```

Therefore:

```text
2^6 = 64 values
```

The remaining 2 bits are ECN.

```text
┌───────────────┬──────┐
│     DSCP      │ ECN  │
│     6 bits    │2 bit │
└───────────────┴──────┘
```

---

## DSCP Structure

```text
DSCP = XXXXX X
       │     │
       │     └─ Drop/selector bits depending class
       └────── Class Selector / forwarding information
```

---

# 10. DSCP Quick Reference

| DSCP | Name | Decimal |
| ---: | ---- | ------: |
|    0 | BE   |       0 |
|    8 | CS1  |       8 |
|   10 | AF11 |      10 |
|   12 | AF12 |      12 |
|   14 | AF13 |      14 |
|   16 | CS2  |      16 |
|   18 | AF21 |      18 |
|   20 | AF22 |      20 |
|   22 | AF23 |      22 |
|   24 | CS3  |      24 |
|   26 | AF31 |      26 |
|   28 | AF32 |      28 |
|   30 | AF33 |      30 |
|   32 | CS4  |      32 |
|   34 | AF41 |      34 |
|   36 | AF42 |      36 |
|   38 | AF43 |      38 |
|   40 | CS5  |      40 |
|   46 | EF   |      46 |
|   48 | CS6  |      48 |
|   56 | CS7  |      56 |

### Most important

```text
BE  = 0
CS1 = 8
AF11 = 10
AF12 = 12
AF13 = 14

CS2 = 16
AF21 = 18
AF22 = 20
AF23 = 22

CS3 = 24
AF31 = 26
AF32 = 28
AF33 = 30

CS4 = 32
AF41 = 34
AF42 = 36
AF43 = 38

CS5 = 40
EF  = 46

CS6 = 48
CS7 = 56
```

---

# 11. DSCP / AF Model

AF classes use:

```text
AFxy

x = forwarding class
y = drop precedence
```

Example:

```text
AF12
││
│└── Drop precedence
└─── Forwarding class
```

For the same forwarding class:

```text
AF11
AF12
AF13
```

Higher drop precedence means the packet is a stronger candidate for dropping during congestion.

---

## Example

```text
AF11 → lower drop precedence
AF12 → medium
AF13 → higher
```

Therefore, during congestion:

```text
AF13
  ↓
more likely to drop

AF12
  ↓
less likely

AF11
  ↓
least likely
```

---

# 12. EF — Expedited Forwarding

```text
EF = DSCP 46
```

Commonly associated with:

* VoIP
* Real-time traffic
* Low-latency traffic

Binary:

```text
46 decimal
= 101110
```

Full TOS byte with ECN = 0:

```text
10111000
   0xB8
```

---

# 13. DSCP ↔ IP Precedence Compatibility

Class Selector values preserve compatibility with IP Precedence.

```text
IP Precedence → CS

0 → CS0 → 0
1 → CS1 → 8
2 → CS2 → 16
3 → CS3 → 24
4 → CS4 → 32
5 → CS5 → 40
6 → CS6 → 48
7 → CS7 → 56
```

This is important when old and modern QoS mechanisms coexist.

---

# 14. Layer-2 QoS — CoS / 802.1Q

If the Layer-3 IP header cannot be used, QoS can be marked at Layer 2.

802.1Q includes:

```text
PCP / CoS = 3 bits
```

Therefore:

```text
2^3 = 8 values

0–7
```

Example:

```text
802.1Q Tag

┌──────────┬────────────┐
│ PCP/CoS  │ VLAN ID    │
│  3 bits  │ 12 bits    │
└──────────┴────────────┘
```

### Important

CoS is Layer 2.

DSCP is Layer 3.

A common mapping is:

```text
CoS
 ↓
Router
 ↓
DSCP
 ↓
WAN
```

---

# 15. Generic QoS Architecture

A scalable QoS architecture usually follows:

```text
CLASSIFY
   ↓
MARK
   ↓
QUEUE
   ↓
SCHEDULE
   ↓
SHAPE / POLICE
   ↓
FORWARD
```

Instead of every device repeatedly classifying traffic:

```text
Edge Device
   ↓
Classify + Mark
   ↓
Core
   ↓
Read DSCP
   ↓
Queue / Forward
```

This is one of the fundamental ideas behind differentiated services.

---

# 16. Traffic Shaping on FortiGate

FortiGate traffic shaping can be configured through:

```text
Policy & Objects
        ↓
Traffic Shaping
```

and CLI objects under:

```text
config firewall shaper traffic-shaper
```

or shaping profiles depending on the FortiOS feature/model.

---

# 17. Shared Shaper

A shared shaper is commonly used when multiple flows/users should consume a common bandwidth pool.

Concept:

```text
              Shared Shaper
             200 Mbps
                 │
       ┌─────────┼─────────┐
       ↓         ↓         ↓
     User A    User B    User C
       │         │         │
       └─────────┼─────────┘
                 ↓
             200 Mbps
```

Example:

```text
Name           = tr-shr-shp-test
Priority       = Medium
Maximum BW     = 200 Mbps
```

---

## Shared vs Per-IP

| Type   | Behavior                                     |
| ------ | -------------------------------------------- |
| Shared | Users/flows share one bandwidth pool         |
| Per-IP | Each source IP receives an independent limit |

---

# 18. Shared Shaper — Per-Policy Behavior

FortiGate can use a shared shaper in different sharing models.

Conceptually:

```text
Per-policy

Policy 1
   ↓
1 Gbps pool

Policy 2
   ↓
1 Gbps pool
```

versus:

```text
All policies

Policy 1 ─┐
Policy 2 ─┼──→ Shared 1 Gbps
Policy 3 ─┘
```

### Typical design

For a large number of users:

```text
ISP = 1 Gbps
Users = 5000
```

A naive equal-share calculation:

```text
1 Gbps / 5000
≈ 200 kbps/user
```

But shared shaping can use bandwidth dynamically rather than statically assigning 200 kbps to every user.

---

# 19. Per-IP Shaper

Per-IP shaping creates a bandwidth limit per source IP.

Example:

```text
Per-IP = 100 Mbps
```

Concept:

```text
Client A → 100 Mbps
Client B → 100 Mbps
Client C → 100 Mbps
```

Unlike a shared shaper:

```text
Shared = 100 Mbps TOTAL
```

### Good use cases

* Public FTP
* Guest networks
* User fairness
* Internet access control
* Preventing one client from consuming all bandwidth

---

# 20. Reverse Shaping

Traffic direction matters.

### LAN → WAN

```text
Upload
Client
  ↓
FortiGate
  ↓
ISP
```

### WAN → LAN

```text
Download
ISP
  ↓
FortiGate
  ↓
Client
```

---

## FortiGate Direction Concept

| Traffic   | Direction          |
| --------- | ------------------ |
| LAN → WAN | Outbound / Upload  |
| WAN → LAN | Inbound / Download |

Reverse shaping is useful when the shaping mechanism needs to control traffic in the opposite direction from the normal policy egress direction.

---

# 21. Interface Traffic Shaping

Example:

```cli
config system interface
    edit "port8"
        set ingress-shaping-profile "test"
    next
end
```

`ingress` refers to traffic entering the FortiGate interface.

For interface bandwidth:

```cli
config system interface
    edit "port3"
        set outbandwidth 4000
    next
end
```

> **Important:** Interface bandwidth values and traffic-shaper bandwidth values have different roles. Don't randomly configure low interface bandwidth values expecting them to act as the primary QoS policy.

---

# 22. Recommended FortiGate Shaping Design

A clean design is generally:

```text
ISP / WAN Interface
       ↓
Traffic Shaping Profile
       ↓
Firewall Policy
       ↓
Shared / Per-IP Shaper
       ↓
Class ID / DSCP
```

Avoid creating conflicting shaping definitions across:

```text
Interface
+
Firewall Policy
+
Shaping Policy
```

unless you deliberately understand how each stage interacts.

---

# 23. Traffic Shaping Profile

A shaping profile can be viewed conceptually as a highway.

```text
                SHAPING PROFILE
┌────────────────────────────────────────┐
│                                        │
│ Class 10 → Voice                       │
│ Class 20 → Business                    │
│ Class 30 → Best Effort                 │
│                                        │
└────────────────────────────────────────┘
```

Each class can have:

* Guaranteed bandwidth
* Maximum bandwidth
* Priority
* Burst
* Queue parameters
* WRED/RED behavior
* DSCP behavior

---

# 24. Guaranteed vs Maximum Bandwidth

Example:

```text
Class 10

Guaranteed = 50%
Maximum    = 100%
```

Meaning:

```text
Normal congestion:
at least 50% is reserved/guaranteed

Available bandwidth:
can potentially use up to 100%
```

---

## Example

```text
Link = 1 Gbps

Class 10
Guaranteed = 50%
Maximum    = 100%
```

Conceptually:

```text
Guaranteed:
500 Mbps

Maximum:
1 Gbps
```

If bandwidth is available, the class may use more than its guaranteed allocation.

---

# 25. Priority vs Bandwidth

These are different concepts.

```text
Priority
   ↓
Who gets served first?

Guaranteed bandwidth
   ↓
How much bandwidth is reserved/assured?

Maximum bandwidth
   ↓
How much can the class consume?

Burst
   ↓
How aggressively can traffic temporarily exceed the normal transmission pattern?
```

---

# 26. Voice QoS

Voice is sensitive to:

```text
Latency
Jitter
Packet Loss
```

Voice generally needs higher priority than ordinary data.

---

## Typical Voice Queue

```text
          WAN
           ↑
      ┌────┴─────┐
      │ Priority │
      │  Voice   │
      └────┬─────┘
           ↑
      ┌────┴─────┐
      │  Data    │
      │  Queue   │
      └──────────┘
```

---

# 27. Voice Delay

A commonly referenced target for one-way voice delay is approximately:

```text
≤ 150 ms
```

Values above this can noticeably affect conversational quality.

Some enterprise design guidance allows higher values, but:

> Lower latency is always preferable for interactive voice.

---

# 28. Codec & Packet Size

Common examples:

| Codec   | Approx. Payload Rate | Typical 20 ms Payload |
| ------- | -------------------: | --------------------: |
| G.711   |              64 kbps |             160 bytes |
| G.726   |             ~32 kbps |             ~80 bytes |
| G.729   |               8 kbps |              20 bytes |
| G.723.1 |             5.3 kbps |             ~20 bytes |

---

## Serialization Example

At:

```text
64 kbps
```

For a 160-byte packet:

```text
160 × 8 / 64000
= 20 ms
```

For a 1500-byte packet:

```text
1500 × 8 / 64000
= 187.5 ms
```

This illustrates why large data packets can create significant serialization delay on slow WAN circuits.

---

# 29. Jitter

Jitter = variation in packet delay.

Example:

```text
Packet 1 → 20 ms
Packet 2 → 21 ms
Packet 3 → 70 ms
Packet 4 → 22 ms
```

The variation creates jitter.

Real-time traffic such as:

* VoIP
* RTP
* Interactive video

is highly sensitive to jitter.

---

# 30. RTP

RTP commonly uses UDP because real-time traffic prioritizes timely delivery over retransmission.

Typical RTP port ranges in common implementations include:

```text
UDP 16384–32767
```

> Exact port ranges depend on the application/vendor configuration.

---

# 31. Call Admission Control

CAC prevents the network from accepting more real-time sessions than the available QoS resources can support.

Concept:

```text
Available Voice Capacity
          ↓
      CAC Check
       ↙     ↘
   Accept    Reject
```

Example:

```text
Voice Queue = 250 kbps

Existing sessions consume:
240 kbps

New call:
64 kbps

240 + 64 > 250

→ Reject / prevent admission
```

The objective is to preserve the quality of existing calls.

---

# 32. Link Fragmentation and Interleaving

LFI is important on slow WAN links.

Problem:

```text
Large data packet
████████████████████████████

Voice packet
   ↑
   must wait
```

On a slow link, the voice packet can experience a long serialization delay.

LFI breaks large packets into smaller fragments and allows higher-priority traffic to be inserted.

```text
Large packet

████████████████████

↓

████ ████ ████ ████

Voice

   VOICE

↓

████ VOICE ████ VOICE
```

### Goal

Reduce the serialization delay experienced by latency-sensitive traffic.

---

# 33. Video QoS

Video can be:

### Interactive

```text
Two-way
Low latency
Low jitter
```

Examples:

* Video conferencing
* Interactive meetings

### Non-interactive

```text
One-way
Delay less critical
Bandwidth intensive
```

Examples:

* Streaming
* Recorded video

---

## Video Characteristics

| Characteristic     | Voice             | Video          |
| ------------------ | ----------------- | -------------- |
| Packet size        | Relatively stable | Dynamic        |
| Packet rate        | Relatively stable | Dynamic        |
| Jitter sensitivity | High              | High           |
| Delay sensitivity  | High              | High           |
| Bandwidth          | Lower             | Higher         |
| Interactive        | Very sensitive    | Very sensitive |

---

# 34. Common Video Protocols

| Protocol        | Typical Port                          |
| --------------- | ------------------------------------- |
| H.323 H.225     | TCP 1720                              |
| H.323 H.225 RAS | TCP/UDP 1719 depending implementation |
| RTSP            | TCP/UDP 554                           |
| RTP             | UDP, application dependent            |
| MPEG traffic    | Application dependent                 |

> Don't build QoS policies from port numbers alone when reliable application identification is available.

---

# 35. NBAR

**NBAR = Network-Based Application Recognition**

Used to identify applications and classify traffic.

Concept:

```text
Packet
  ↓
NBAR
  ↓
Application Detection
  ↓
Classification
  ↓
QoS
```

Examples:

```text
Voice
Video
Web
FTP
Business Apps
```

---

# 36. IntServ vs DiffServ

## IntServ

Integrated Services provides per-flow resource reservation.

Concept:

```text
Flow
 ↓
RSVP
 ↓
Admission Control
 ↓
Reserved Resources
```

---

## DiffServ

Differentiated Services uses class-based treatment.

```text
Packet
 ↓
DSCP
 ↓
PHB
 ↓
Queue / Priority / Drop behavior
```

DiffServ scales better because core devices don't need to maintain individual reservation state for every flow.

---

# 37. Shaping Queue Architecture

Conceptual FortiGate processing:

```text
Incoming Packet
      ↓
Classification
      ↓
DSCP / TOS / Policy Match
      ↓
Class ID
      ↓
Traffic Shaper
      ↓
Shape Queue
      ↓
Software Queue
      ↓
Hardware Queue
      ↓
Interface
```

If the shaping queue has packets:

```text
Classify
   ↓
High / Medium / Low
   ↓
Software Queue
   ↓
Hardware Queue
```

If no shaping queue is required:

```text
Token availability
      ↓
Software Queue
      ↓
Hardware Queue
```

---

# 38. Generic Traffic Shaper

Conceptually:

```text
Traffic Shaper
    ↓
Rate = 64 Mbps
BC   = 8 MB
BE   = 8 MB
```

Example configuration concept:

```cli
traffic-shaper rate 64000 8000 8000
```

The exact CLI syntax is platform/version dependent; verify available parameters with:

```cli
config firewall shaper traffic-shaper
    edit "test"
        get
```

---

# 39. Class-Based Shaping

Class-based shaping allows traffic to be divided into classes.

Example:

```text
Class 10 → Voice
Class 20 → Business
Class 30 → Best Effort
```

Concept:

```text
                    1 Gbps
                      │
          ┌───────────┼───────────┐
          ↓           ↓           ↓
       Voice       Business       BE
        50%          30%          20%
```

---

# 40. Shaping with Priority

Example:

```text
Voice
Guaranteed = 50%
Maximum    = 100%
Priority   = High

Business
Guaranteed = 30%
Maximum    = 60%

Best Effort
Guaranteed = 20%
Maximum    = 50%
```

Traffic is then associated with classes using:

```text
Class ID
```

---

# 41. Example FortiGate Shaping Profile

```cli
config firewall shaping-profile
    edit "qa-prof"
        set type queuing

        config shaping-entries
            edit 1
                set class-id 10
                set guaranteed-bandwidth-percentage 50
                set maximum-bandwidth-percentage 100
            next

            edit 2
                set class-id 20
                set guaranteed-bandwidth-percentage 30
                set maximum-bandwidth-percentage 60
            next

            edit 3
                set class-id 30
                set guaranteed-bandwidth-percentage 20
                set maximum-bandwidth-percentage 50
            next
        end
    next
end
```

> CLI availability and exact field names vary by FortiOS release/model. Use `?` / `get` on the target appliance.

---

# 42. Traffic Shaping Policy

A shaping policy can classify traffic using:

```text
Source
Destination
Service
Interface
Schedule
Class ID
DSCP/TOS
```

Example:

```cli
config firewall shaping-policy
    edit 1
        set service "HTTP" "HTTPS"
        set dstintf "port1"
        set srcaddr "all"
        set dstaddr "all"
        set class-id 10
    next
end
```

---

# 43. DSCP Marking on Firewall Policy

FortiGate can modify DSCP/TOS values in policies.

Example:

```cli
config firewall policy
    edit 1
        set diffserv-forward enable
        set diffserv-reverse enable
        set diffservcode-forward 101110
        set diffservcode-reverse 101110
    next
end
```

`101110`:

```text
Binary = 46
Name   = EF
```

---

# 44. Forward vs Reverse DSCP

This is extremely important.

```text
Forward direction
---------------->
DSCP Forward

<----------------
Reverse direction
DSCP Reverse
```

Example:

```cli
set diffserv-forward enable
set diffservcode-forward 101110

set diffserv-reverse enable
set diffservcode-reverse 101110
```

---

# 45. DSCP Mapping Example

Topology:

```text
DMZ
192.168.20.0/24
      |
     FGT2
      |
192.168.100.0/24
      |
     FGT1
      |
192.168.101.0/24
      |
     LAN
```

Traffic classification can occur at FGT1:

```text
Client
  ↓
FGT1
  ↓
Mark EF
  ↓
FGT2
  ↓
Queue based on EF
```

---

# 46. Disable ASIC Offload During Troubleshooting

When troubleshooting QoS/DSCP behavior, hardware acceleration can hide packet processing details.

Example:

```cli
config firewall policy
    edit 1
        set auto-asic-offload disable
    next
end
```

Then capture traffic and inspect the packet.

> Don't disable hardware offload permanently without understanding the performance impact.

---

# 47. DSCP Match with TOS Mask

FortiGate can use TOS and TOS mask to match traffic.

Common DSCP-only mask:

```text
0xFC

Binary:

11111100
```

Meaning:

```text
DSCP → checked
ECN  → ignored
```

---

# 48. TOS / DSCP Matching Table

| DSCP | Name | TOS Hex |   Mask |
| ---: | ---- | ------: | -----: |
|    0 | BE   |  `0x00` | `0xFC` |
|    8 | CS1  |  `0x20` | `0xFC` |
|   10 | AF11 |  `0x28` | `0xFC` |
|   12 | AF12 |  `0x30` | `0xFC` |
|   14 | AF13 |  `0x38` | `0xFC` |
|   16 | CS2  |  `0x40` | `0xFC` |
|   18 | AF21 |  `0x48` | `0xFC` |
|   20 | AF22 |  `0x50` | `0xFC` |
|   22 | AF23 |  `0x58` | `0xFC` |
|   24 | CS3  |  `0x60` | `0xFC` |
|   26 | AF31 |  `0x68` | `0xFC` |
|   28 | AF32 |  `0x70` | `0xFC` |
|   30 | AF33 |  `0x78` | `0xFC` |
|   32 | CS4  |  `0x80` | `0xFC` |
|   34 | AF41 |  `0x88` | `0xFC` |
|   36 | AF42 |  `0x90` | `0xFC` |
|   38 | AF43 |  `0x98` | `0xFC` |
|   40 | CS5  |  `0xA0` | `0xFC` |
|   46 | EF   |  `0xB8` | `0xFC` |
|   48 | CS6  |  `0xC0` | `0xFC` |
|   56 | CS7  |  `0xE0` | `0xFC` |

---

# 49. Advanced TOS Masks

| Mask   | Binary     | Purpose                |
| ------ | ---------- | ---------------------- |
| `0xFF` | `11111111` | Match entire TOS byte  |
| `0xFC` | `11111100` | Match DSCP, ignore ECN |
| `0xF0` | `11110000` | Match first 4 bits     |
| `0xE0` | `11100000` | Match first 3 bits     |
| `0xC0` | `11000000` | Match first 2 bits     |
| `0x03` | `00000011` | Match ECN              |

---

# 50. Example: Match EF

EF:

```text
DSCP = 46
Binary = 101110
```

TOS:

```text
10111000
```

Hex:

```text
0xB8
```

Mask:

```text
11111100
0xFC
```

Example:

```cli
set tos 0xb8
set tos-mask 0xfc
```

This matches:

```text
DSCP = EF (46)
```

while ignoring ECN.

---

# 51. DSCP-Based Shaping

Example concept:

```text
Incoming Packet
      ↓
Read DSCP
      ↓
EF (46)?
   ↙     ↘
 YES      NO
  ↓        ↓
Voice     Other
Queue     Queue
```

---

# 52. Multi-Stage DSCP

Multi-stage DSCP marking can change the DSCP value based on traffic consumption.

Concept:

```text
Normal usage
    ↓
Normal DSCP

Exceeds threshold
    ↓
New DSCP

Exceeds higher threshold
    ↓
More aggressive treatment
```

Example concept:

```text
0–50 Mbps
    ↓
Normal DSCP

50–100 Mbps
    ↓
Exceed DSCP

>100 Mbps
    ↓
Maximum / higher drop treatment
```

---

# 53. Multi-Stage Example

```cli
config firewall shaper traffic-shaper
    edit "50k-100k-150k"
        set guaranteed-bandwidth 50
        set maximum-bandwidth 150

        set diffserv enable
        set dscp-marking-method multi-stage

        set exceed-bandwidth 100
        set exceed-dscp 111000
        set exceed-class-id 20

        set maximum-dscp 111111
        set diffservcode 100000

        set overhead 14
    next
end
```

Concept:

```text
0 ───── 50 ───── 100 ───── 150 Mbps
       G          E          MAX

       │          │           │
       │          │           └─ Maximum treatment
       │          └───────────── Exceed DSCP
       └──────────────────────── Guaranteed
```

---

# 54. Overhead

Traffic shaping may need to account for encapsulation overhead.

Examples:

```text
Ethernet/VLAN
MPLS
PPPoE
VPN
```

Approximate engineering considerations:

```text
PPPoE → additional overhead
MPLS   → label overhead
VLAN   → tag overhead
VPN    → tunnel/encryption overhead
```

The exact overhead depends on the encapsulation.

---

# 55. RED — Random Early Detection

RED is a congestion-avoidance mechanism.

Problem:

```text
TCP traffic
   ↓
Queue becomes full
   ↓
Tail Drop
   ↓
Many packets dropped at once
```

RED begins dropping packets before the queue is completely full.

```text
Queue utilization

LOW ──────── MEDIUM ───────── HIGH
              │
              ↓
          Early Drop
```

---

# 56. Why RED Helps TCP

TCP reacts to packet loss by reducing its congestion window.

Therefore:

```text
Early packet loss
      ↓
TCP detects congestion
      ↓
Window decreases
      ↓
Traffic becomes less aggressive
      ↓
Queue becomes more manageable
```

---

# 57. WRED

**Weighted Random Early Detection**

WRED extends RED by allowing different drop behavior for traffic classes/markings.

Concept:

```text
Low Drop Priority
       ↓
Keep longer

High Drop Priority
       ↓
Drop earlier
```

This works particularly well with DSCP/AF drop precedence.

---

# 58. FortiGate WRED Example

Conceptual configuration:

```cli
config firewall shaping-profile
    edit "tsh-x"
        set type queueing

        config shape-entries
            edit 1
                set limit 1000
                set red-probability 20
                set min 300
                set max 500
            next
        end
    next
end
```

Concept:

```text
Queue < 300
    ↓
No RED dropping

300–500
    ↓
0–20% probability

> 500
    ↓
Aggressive / maximum drop behavior
```

> Exact RED/WRED behavior depends on the FortiOS implementation and profile parameters.

---

# 59. WRED + NPU

Hardware acceleration can affect QoS features.

For troubleshooting:

```text
WRED
+
NPU offload
```

may not behave as expected if the traffic bypasses the software QoS path.

Therefore:

```text
Troubleshooting QoS
        ↓
Check offload
        ↓
Disable temporarily if required
```

---

# 60. Burst

Burst allows temporary transmission above the smooth average.

Example:

```text
Link = 1 Mbps
Guaranteed = 50%
Maximum = 100%
Burst interval = 100 ms
```

A 100 ms burst at 50% of 1 Mbps:

```text
1,000,000 × 0.5 × 0.1
= 50,000 bits
```

Convert to bytes:

```text
50,000 / 8
= 6,250 bytes
```

So approximately:

```text
100 ms burst
→ 6,250 bytes
```

---

# 61. C-Burst

If the committed burst interval is:

```text
200 ms
```

then:

```text
1,000,000 × 0.5 × 0.2
= 100,000 bits
```

Bytes:

```text
100,000 / 8
= 12,500 bytes
```

---

# 62. Burst Parameters

Example:

```cli
config firewall shaping-profile
    edit "tr-shape-prof-test"
        set type queuing

        config shaping-entries
            edit 1
                set class-id 1
                set guaranteed-bandwidth-percentage 50
                set maximum-bandwidth-percentage 100
                set burst-in-msec 100
                set cburst-in-msec 200
            next
        end
    next
end
```

Typical conceptual ranges may depend on FortiOS version/model.

---

# 63. Burst vs Average Rate

Without shaping:

```text
████████████████████
```

With shaping:

```text
███   ███   ███   ███
```

The objective is to keep the long-term average within the configured rate while permitting controlled bursts.

---

# 64. Adaptive Shaping

Adaptive shaping can react to congestion conditions.

Concept:

```text
ISP uncongested
       ↓
More bandwidth available

ISP congested
       ↓
Reduce sending rate
```

Example concept:

```cli
shape adaptive 300
```

Adaptive shaping is a specialized feature and should be used only when the environment and FortiOS implementation support it.

---

# 65. MIR — Minimum Information Rate

In adaptive shaping environments, MIR represents a minimum rate that should be maintained even under congestion.

Concept:

```text
Maximum Rate
     ↑
     │
Adaptive Region
     │
     ↓
MIR
```

---

# 66. Generic Shaping Example

Cisco-style conceptual example:

```cisco
policy-map SHAPE-ALL
 class class-default
  shape average 64000
```

Apply:

```cisco
interface GigabitEthernet0/0
 service-policy out SHAPE-ALL
```

Verify:

```cisco
show policy-map
show policy-map interface GigabitEthernet0/0
```

---

# 67. Shape + Priority Queue

Voice should be classified before normal data.

Example:

```cisco
class-map match-all VOIP-RTP
 match ip rtp 16384 16383
```

Then:

```cisco
policy-map QUEUE-VOIP
 class VOIP-RTP
  priority 32

 class class-default
  fair-queue
```

Then shape:

```cisco
policy-map SHAPE-ALL
 class class-default
  shape average 96000
  service-policy QUEUE-VOIP
```

Apply:

```cisco
interface GigabitEthernet0/0
 service-policy out SHAPE-ALL
```

Concept:

```text
                 96 kbps
                    │
                 SHAPER
                    │
          ┌─────────┴─────────┐
          ↓                   ↓
       VOICE                DATA
      Priority            Fair Queue
```

---

# 68. QoS Classification Example

```cisco
ip access-list extended USER1
 permit ip host 192.168.1.10 any

ip access-list extended USER2
 permit ip host 192.168.1.20 any

class-map USER1
 match access-group name USER1

class-map USER2
 match access-group name USER2

policy-map TRAFFIC-CONTROL
 class USER1
  police 8000

 class USER2
  police 10000
```

Apply:

```cisco
interface Serial0/0/0
 service-policy out TRAFFIC-CONTROL
```

---

# 69. QoS Design — Congestion Management

A useful mental model:

```text
             CONGESTION
                 │
       ┌─────────┴─────────┐
       ↓                   ↓
  Queue traffic        Drop traffic
       │                   │
    Shaping              Policing
       │                   │
       └─────────┬─────────┘
                 ↓
            QoS Policy
```

---

# 70. Traffic Direction on FortiGate

### LAN → WAN

```text
LAN
 ↓
FortiGate
 ↓
WAN
 ↓
ISP
```

Use:

```text
Outbound / Upload shaping
```

### WAN → LAN

```text
ISP
 ↓
WAN
 ↓
FortiGate
 ↓
LAN
```

Use:

```text
Inbound / Download shaping
Reverse shaping where appropriate
```

---

# 71. Multicast + QoS

Multicast traffic can also be subject to traffic handling and shaping.

Example multicast:

```text
239.192.20.1:5004
```

Server:

```bash
ffmpeg -re \
-i "video.mp4" \
-c copy \
-f mpegts \
"udp://239.192.20.1:5004?pkt_size=1316"
```

Client:

```bash
ffplay -fflags nobuffer \
udp://239.192.20.1:5004
```

---

# 72. Multicast FortiGate Concept

On both FortiGates, configure multicast appropriately.

Example topology:

```text
Server
  │
 FGT1
  │
 WAN / Transit
  │
 FGT2
  │
 Client
```

Receiver interfaces need to be included in the multicast design.

Verify:

```cli
diagnose sys mcast-session list
```

This displays multicast session information/counters.

---

# 73. Global Traffic Priority

FortiGate can use TOS/DSCP to determine traffic priority.

Concept:

```cli
config system global
    set traffic-priority dscp
    set traffic-priority-level high
end
```

Possible priority levels include:

```text
High
Medium
Low
```

---

# 74. TOS-Based Priority

Example:

```cli
config system tos-based-priority
    edit 1
        set tos 5
        set priority high
    next
end
```

TOS values are interpreted according to the FortiOS feature implementation.

---

# 75. DSCP-Based Priority

Concept:

```cli
config system dscp-based-priority
    edit 1
        set ds 46
        set priority high
    next
end
```

The exact command tree/field naming should be verified on the target FortiOS build using:

```cli
?
get
show
```

---

# 76. Interface Bandwidth

Example:

```cli
config system interface
    edit "port3"
        set outbandwidth 4000
    next
end
```

Values are commonly expressed in:

```text
Kbps
```

depending on the specific FortiOS parameter.

---

# 77. QoS Packet Processing Logic

A useful mental model:

```text
                PACKET
                   │
                   ↓
             Read IP Header
                   │
             DSCP / TOS
                   │
                   ↓
             Classification
                   │
                   ↓
             Match QoS Rule
                   │
          ┌────────┴─────────┐
          ↓                  ↓
       No Class             Class ID
          │                  │
      Default             Shaper/Profile
          │                  │
          └────────┬─────────┘
                   ↓
                Queue
                   ↓
             Scheduler
                   ↓
               Interface
                   ↓
                Forward
```

---

# 78. Class ID

Class ID is a key concept in class-based traffic shaping.

Example:

```text
Class ID 10 → Voice
Class ID 20 → Business
Class ID 30 → Default
```

Traffic is classified and mapped to the appropriate shaping class.

---

# 79. Traffic Shaper vs Shaping Policy

Don't confuse:

```text
Traffic Shaper
```

with:

```text
Traffic Shaping Policy
```

Conceptually:

```text
Traffic Shaper
    ↓
Defines HOW bandwidth is controlled

Shaping Policy
    ↓
Defines WHICH traffic uses that behavior
```

---

# 80. Shared Shaper Example

```cli
config firewall shaper traffic-shaper
    edit "tr-shr-shp-test"
        set per-policy enable
    next
end
```

`per-policy` determines how the shared resource is associated with policies.

Always verify the exact behavior on the FortiOS version in use.

---

# 81. DSCP Forwarding Example

Suppose:

```text
Source
192.168.20.200
```

needs EF treatment.

On the forwarding policy:

```cli
config firewall policy
    edit 1
        set diffserv-forward enable
        set diffservcode-forward 101110
    next
end
```

Result:

```text
DSCP = 46 / EF
```

---

# 82. Reverse DSCP Example

For reverse traffic:

```cli
config firewall policy
    edit 1
        set diffserv-reverse enable
        set diffservcode-reverse 101110
    next
end
```

---

# 83. DSCP-Based QoS Architecture

```text
             FGT1
              │
        Mark DSCP EF
              │
              ↓
      ┌───────────────┐
      │     WAN       │
      └───────────────┘
              │
              ↓
             FGT2
              │
       Match DSCP EF
              │
              ↓
        Voice Shaper
              │
              ↓
          Receiver
```

This is much more scalable than trying to classify the same application independently at every hop.

---

# 84. Debug Session

Filter sessions:

```cli
diagnose sys session filter dst 8.8.8.8
diagnose sys session list
```

---

# 85. Debug Flow

Enable useful flow information:

```cli
diagnose debug flow show function-name enable
diagnose debug flow show iprope enable
diagnose debug flow filter address 192.168.20.200
diagnose debug flow trace start 100
diagnose debug enable
```

Stop:

```cli
diagnose debug disable
diagnose debug reset
```

---

# 86. Interface / Class Verification

Useful commands:

```cli
diagnose netlink interface list port3
```

Useful for inspecting interface-related information including class/shaping details where supported.

Queue/class information:

```cli
diagnose netlink intf-class list port3
diagnose netlink intf-qdis list port3
```

---

# 87. Traffic Testing

Example:

```cli
diagnose traffictest run 8.8.8.8
```

Use traffic tests carefully; traffic-generation behavior and syntax can vary by FortiOS release.

---

# 88. NPU Offload & QoS

Hardware acceleration is excellent for performance, but some QoS features depend on software processing.

Important troubleshooting rule:

```text
QoS doesn't behave as expected
          ↓
Check NPU / ASIC offload
          ↓
Temporarily disable offload
          ↓
Test again
```

Example:

```cli
config firewall policy
    edit 1
        set auto-asic-offload disable
    next
end
```

---

# 89. NPU Interface Shaping

Some FortiGate platforms support NPU interface shaping.

Concept:

```cli
config system npu
    set intf-shaping-offload enable
end
```

> Availability is hardware/platform dependent. Do not assume every FortiGate supports every NPU QoS feature.

---

# 90. QoS Troubleshooting Workflow

When shaping isn't working:

```text
1. Is the traffic matching the correct firewall policy?
                  ↓
2. Is the shaping policy matching?
                  ↓
3. Is the correct interface selected?
                  ↓
4. Is the correct direction being shaped?
                  ↓
5. Is the class ID correct?
                  ↓
6. Is the shaper attached?
                  ↓
7. Is guaranteed/max bandwidth correct?
                  ↓
8. Is DSCP/TOS marking correct?
                  ↓
9. Is NPU offload bypassing the expected path?
                  ↓
10. Check queues/statistics
```

---

# 91. Common QoS Mistake — Wrong Direction

### Wrong mental model

```text
WAN → LAN
```

and expecting normal outbound shaping to control download traffic.

### Correct thinking

```text
LAN → WAN
    = Upload

WAN → LAN
    = Download
    = Reverse/inbound consideration
```

---

# 92. Common QoS Mistake — Shaping Too High

Suppose:

```text
ISP CIR = 100 Mbps
FortiGate shape = 110 Mbps
```

The ISP can still police at:

```text
100 Mbps
```

Result:

```text
FortiGate
110 Mbps
   ↓
ISP
100 Mbps policer
   ↓
DROP
```

Better design:

```text
ISP CIR
   ↓
FortiGate Shape slightly below CIR
   ↓
ISP
```

This gives the customer device control of the queue instead of allowing the ISP to become the first congestion point.

---

# 93. Common QoS Mistake — Guaranteed Bandwidth Too High

Example:

```text
ISP = 100 Mbps

Class A = 80 Mbps guaranteed
Class B = 30 Mbps guaranteed
```

Total:

```text
80 + 30 = 110 Mbps
```

This is impossible on a 100 Mbps link.

---

# 94. Common QoS Mistake — Huge Queue

Huge queues can reduce immediate packet loss but cause:

```text
Queue ↑
Latency ↑
Jitter ↑
Voice Quality ↓
```

Therefore:

> Bigger queue ≠ better QoS.

---

# 95. Common QoS Mistake — No Classification

If everything is treated equally:

```text
Voice = Data = Backup = Download
```

then during congestion:

```text
FIFO
 ↓
Everyone waits
```

QoS requires meaningful classification.

---

# 96. Common QoS Mistake — Marking Without Trust

DSCP marking only works if downstream devices:

```text
Trust
Read
Preserve
and/or Act on
```

the marking.

Architecture:

```text
Edge
 ↓
Mark EF
 ↓
Switch
 ↓
Trust DSCP
 ↓
Router
 ↓
Queue EF
```

If a device resets DSCP:

```text
EF
 ↓
BE
```

QoS intent is lost.

---

# 97. QoS Design Rule — Congestion Point

Always shape where you control the queue.

Bad:

```text
FGT → ISP
        ↓
     ISP Queue
```

Better:

```text
FGT Shape
    ↓
Controlled Queue
    ↓
ISP
```

The goal is to make congestion occur where you have visibility and policy control.

---

# 98. ISP CIR Design

Example:

```text
Physical access = 1 Gbps
ISP CIR = 500 Mbps
```

Recommended conceptual model:

```text
1 Gbps physical
      ↓
FortiGate
      ↓
~500 Mbps shaping
      ↓
ISP
      ↓
CIR 500 Mbps
```

This prevents the ISP from becoming the uncontrolled bottleneck.

---

# 99. Voice over Slow WAN

For a slow WAN:

```text
64 kbps
```

A 1500-byte packet takes:

```text
187.5 ms
```

This alone can exceed the desirable one-way voice delay budget.

Solutions include:

```text
Higher bandwidth
Smaller packets
Codec optimization
Priority queue
LFI
Traffic shaping
CAC
```

---

# 100. QoS Decision Tree

```text
Need to control traffic?
        │
        ├── Need to delay/smooth?
        │       └── SHAPING
        │
        ├── Need to drop above rate?
        │       └── POLICING
        │
        ├── Need traffic priority?
        │       └── QUEUE / PRIORITY
        │
        ├── Need application identification?
        │       └── CLASSIFICATION / NBAR
        │
        ├── Need end-to-end class information?
        │       └── DSCP
        │
        ├── Need Layer-2 marking?
        │       └── CoS / 802.1Q PCP
        │
        ├── Need congestion avoidance?
        │       └── RED / WRED
        │
        └── Need per-user fairness?
                └── PER-IP SHAPER
```

---

# 101. FortiGate QoS Mental Model

```text
                    TRAFFIC
                       │
                       ↓
                Firewall Policy
                       │
                       ↓
              Classification
                       │
          ┌────────────┴────────────┐
          │                         │
        DSCP                     Address/
        TOS                      Service
          │                         │
          └────────────┬────────────┘
                       ↓
                    Class ID
                       ↓
               Traffic Shaper
                       ↓
              ┌────────┴────────┐
              │                 │
         Guaranteed          Maximum
              │                 │
              └────────┬────────┘
                       ↓
                    Queue
                       ↓
                  Scheduler
                       ↓
                   Interface
                       ↓
                     WAN
```

---

# 102. QoS Reference — Delay

| Delay         | Main Cause               | QoS Can Control?      |
| ------------- | ------------------------ | --------------------- |
| Serialization | Packet size + link speed | Partially             |
| Propagation   | Distance/medium          | ❌                     |
| Processing    | Device processing        | Limited               |
| Queueing      | Congestion               | ✅                     |
| Shaping       | Intentional buffering    | ✅                     |
| ISP Network   | Provider network         | Usually ❌             |
| Codec         | Voice encoding           | Application dependent |

---

# 103. QoS Reference — Traffic Control

| Mechanism      | Main Purpose               |
| -------------- | -------------------------- |
| Classification | Identify traffic           |
| Marking        | Assign QoS value           |
| Queueing       | Decide service order       |
| Shaping        | Smooth traffic             |
| Policing       | Enforce/drop               |
| RED            | Early congestion avoidance |
| WRED           | Weighted early dropping    |
| CAC            | Protect real-time sessions |
| LFI            | Reduce serialization delay |
| DSCP           | Carry QoS class across L3  |

---

# 104. QoS Reference — FortiGate Objects

```text
Firewall Policy
      │
      ├── Traffic Shaper
      │
      ├── Shared Shaper
      │
      ├── Per-IP Shaper
      │
      └── DSCP/TOS Marking

Shaping Policy
      │
      ├── Match traffic
      ├── Class ID
      ├── DSCP/TOS
      └── Shaping Profile

Shaping Profile
      │
      ├── Classes
      ├── Guaranteed BW
      ├── Maximum BW
      ├── Burst
      ├── C-Burst
      ├── RED/WRED
      └── DSCP behavior
```

---

# 105. High-Value Commands

## Sessions

```cli
diagnose sys session filter dst 8.8.8.8
diagnose sys session list
```

## Flow Debug

```cli
diagnose debug flow show function-name enable
diagnose debug flow show iprope enable
diagnose debug flow filter address 192.168.20.200
diagnose debug flow trace start 100
diagnose debug enable
```

Stop:

```cli
diagnose debug disable
diagnose debug reset
```

## Interface

```cli
diagnose netlink interface list port3
```

## Classes

```cli
diagnose netlink intf-class list port3
```

## Queues

```cli
diagnose netlink intf-qdis list port3
```

## Multicast

```cli
diagnose sys mcast-session list
```

## Traffic Test

```cli
diagnose traffictest run 8.8.8.8
```

---

# 106. Configuration Verification

Before troubleshooting, collect:

```cli
show firewall policy
show firewall shaping-policy
show firewall shaper traffic-shaper
show firewall shaping-profile
show system interface
```

And inspect:

```cli
get
```

inside the relevant configuration object.

---

# 107. QoS Troubleshooting Checklist

```text
[ ] Confirm ISP CIR
[ ] Confirm physical interface rate
[ ] Confirm FortiGate shaping rate
[ ] Confirm correct direction
[ ] Confirm policy match
[ ] Confirm shaping-policy match
[ ] Confirm source/destination
[ ] Confirm service
[ ] Confirm class ID
[ ] Confirm guaranteed bandwidth
[ ] Confirm maximum bandwidth
[ ] Confirm burst
[ ] Confirm C-burst
[ ] Confirm DSCP marking
[ ] Confirm DSCP preservation
[ ] Confirm downstream device trusts DSCP
[ ] Check queue statistics
[ ] Check WRED/RED configuration
[ ] Check NPU/ASIC offload
[ ] Disable offload temporarily if required
[ ] Capture packets
[ ] Verify DSCP on the wire
[ ] Test with controlled traffic
```

---

# 108. Production QoS Design Checklist

### WAN

```text
[ ] Know ISP CIR
[ ] Know PIR
[ ] Know physical access speed
[ ] Shape slightly below ISP CIR where appropriate
[ ] Identify congestion point
```

### Voice

```text
[ ] Identify RTP
[ ] Mark EF
[ ] Give voice priority
[ ] Reserve bandwidth
[ ] Control jitter
[ ] Control packet loss
[ ] Consider CAC
[ ] Consider LFI on very slow links
```

### Data

```text
[ ] Separate business-critical traffic
[ ] Define minimum bandwidth
[ ] Define maximum bandwidth
[ ] Use fair queueing where appropriate
[ ] Protect critical applications
```

### Internet Users

```text
[ ] Shared shaper for aggregate control
[ ] Per-IP shaper for fairness
[ ] Avoid one user consuming entire WAN
```

---

# 109. Exam Quick Recall

> ### 🔥 NSE / Interview Memory Hooks

```text
SHAPING
= Queue + Delay

POLICING
= Drop / Remark

RED
= Early Random Drop

WRED
= Weighted Early Drop

DSCP
= 6 bits

ECN
= 2 bits

TOS
= 8 bits

IP Precedence
= 3 bits

802.1Q PCP / CoS
= 3 bits

EF
= DSCP 46
= 101110
= 0xB8 with ECN 00

CS1
= 8

CS5
= 40

CS6
= 48

CS7
= 56

DSCP mask
= 0xFC
= 11111100

EF TOS
= 0xB8

EF + 0xFC mask
= DSCP 46, ignore ECN
```

---

# 110. One-Minute QoS Summary

```text
                    QoS
                     │
       ┌─────────────┼─────────────┐
       ↓             ↓             ↓
 CLASSIFY          MARK          CONTROL
       │             │             │
       │           DSCP            │
       │           CoS             │
       │           TOS             │
       │                           │
       └─────────────┬─────────────┘
                     ↓
                   QUEUE
                     │
             ┌───────┴───────┐
             ↓               ↓
         PRIORITY         FAIRNESS
             │               │
             └───────┬───────┘
                     ↓
             SHAPE / POLICE
                     │
              ┌──────┴──────┐
              ↓             ↓
           SHAPE          POLICE
              │             │
          Queue/Delay    Drop/Remark
              │             │
              └──────┬──────┘
                     ↓
                  FORWARD
```

---

# 111. The Golden Rules

### Rule 1

> **Shaping controls the queue; policing controls the rate.**

### Rule 2

> **Shape before the congestion point you control.**

### Rule 3

> **DSCP is a marking mechanism; DSCP alone does not create QoS.**

### Rule 4

> **EF (46) is commonly used for real-time/voice traffic.**

### Rule 5

> **`0xFC` matches DSCP while ignoring ECN.**

### Rule 6

> **Large queues can trade packet loss for latency.**

### Rule 7

> **Higher bandwidth reduces serialization delay and usually reduces queueing pressure.**

### Rule 8

> **Voice needs low latency, low jitter, and low packet loss.**

### Rule 9

> **Per-IP shaping provides individual limits; shared shaping provides a shared pool.**

### Rule 10

> **When QoS behaves unexpectedly, always investigate hardware/NPU offloading.**

---

# 112. Final Architecture

```text
                    ┌─────────────────┐
                    │   Applications  │
                    └────────┬────────┘
                             │
                             ↓
                    ┌─────────────────┐
                    │ Classification  │
                    │ NBAR / Policy   │
                    └────────┬────────┘
                             │
                             ↓
                    ┌─────────────────┐
                    │     Marking     │
                    │ DSCP / TOS/CoS  │
                    └────────┬────────┘
                             │
                             ↓
                    ┌─────────────────┐
                    │     Queueing     │
                    │ Priority / WFQ  │
                    └────────┬────────┘
                             │
                    ┌────────┴────────┐
                    ↓                 ↓
               SHAPING            POLICING
                    │                 │
               Queue/Delay       Drop/Remark
                    │                 │
                    └────────┬────────┘
                             ↓
                    ┌─────────────────┐
                    │      WAN        │
                    │  Controlled     │
                    │    Traffic      │
                    └─────────────────┘
```

---

## ⭐ Core Takeaway

**QoS is not simply "giving more bandwidth to important traffic."**

A proper QoS design answers five questions:

```text
1. WHAT is the traffic?
        ↓
2. HOW do I identify it?
        ↓
3. HOW should I mark it?
        ↓
4. WHAT happens during congestion?
        ↓
5. WHERE should the queue exist?
```

If those five questions are correctly answered, the QoS architecture becomes predictable:

```text
Classification
      ↓
Marking
      ↓
Queueing
      ↓
Priority / Fairness
      ↓
Shaping / Policing
      ↓
Congestion Management
      ↓
Controlled Forwarding
```

> **SheynShield | Engineering Secure Networks**
>
> **FortiGate · QoS · Traffic Shaping · DSCP · Network Security**

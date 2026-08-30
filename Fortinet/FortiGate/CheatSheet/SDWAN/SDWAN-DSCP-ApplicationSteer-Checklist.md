# 🔗 SheynShield Resources

# FortiGate SD-WAN DSCP — Steering, ECMP, Application Routing & Traffic Shaping

> **FortiOS | SD-WAN DSCP Steering | TOS Matching | ECMP | FIB | Application-Based Routing | SLA | Traffic Shaping | QoS | Troubleshooting**

---

## 📑 Table of Contents

* [1. SD-WAN DSCP Steering Checklist](#1-sd-wan-dscp-steering-checklist)
* [2. DSCP Matching with TOS and TOS Mask](#2-dscp-matching-with-tos-and-tos-mask)
* [3. FGT-1 DSCP Steering](#3-fgt-1-dscp-steering)
* [4. FGT-4 Remote / ISP Side](#4-fgt-4-remote--isp-side)
* [5. DSCP Packet Capture](#5-dscp-packet-capture)
* [6. DSCP Traffic Shaping](#6-dscp-traffic-shaping)
* [7. SD-WAN ECMP](#7-sd-wan-ecmp)
* [8. Longest Prefix Match](#8-longest-prefix-match)
* [9. SD-WAN Tie-Break](#9-sd-wan-tie-break)
* [10. Lowest Cost vs Best Quality](#10-lowest-cost-vs-best-quality)
* [11. Application-Based SD-WAN](#11-application-based-sd-wan)
* [12. Firewall Policy vs SD-WAN Rule](#12-firewall-policy-vs-sd-wan-rule)
* [13. Application Steering Troubleshooting](#13-application-steering-troubleshooting)
* [14. End-to-End DSCP SD-WAN Flow](#14-end-to-end-dscp-sd-wan-flow)
* [15. SD-WAN Verification Checklist](#15-sd-wan-verification-checklist)
* [16. Quick Reference](#16-quick-reference)
* [17. Design Rules](#17-design-rules)
* [18. Core CLI](#18-core-cli)
* [19. Key Takeaways](#19-key-takeaways)
* [SheynShield Resources](#sheynshield-resources)

---

# 1. SD-WAN DSCP Steering Checklist

## 🎯 Objective

Use DSCP/TOS classification to influence SD-WAN path selection.

```text
Traffic
   |
   v
DSCP / TOS
   |
   v
SD-WAN Service Match
   |
   v
SLA / Quality Evaluation
   |
   v
Priority / Strategy
   |
   v
Selected WAN Member
```

### Checklist

* [ ] Identify the traffic class that requires steering.
* [ ] Identify the DSCP value carried by the traffic.
* [ ] Confirm whether the FortiGate should match DSCP/TOS.
* [ ] Configure `tos`.
* [ ] Configure `tos-mask`.
* [ ] Select the appropriate SD-WAN strategy.
* [ ] Configure SLA / health-check.
* [ ] Define eligible SD-WAN members.
* [ ] Verify firewall policy allows the traffic.
* [ ] Verify routing/FIB permits the intended egress path.
* [ ] Capture packets and confirm DSCP values.
* [ ] Verify the selected SD-WAN member.

---

# 2. DSCP Matching with TOS and TOS Mask

## Example

```cli
config system sdwan
    config service
        edit 5
            set name VoIP-Steer
            set mode priority
            set tos 0x70
            set tos-mask 0xf0
            set dst all
            set health-check Default_DNS
            set link-cost-factor jitter
            set priority-members 4 3
        next
    end
end
```

## Parameter Checklist

| Parameter          |       Example | Purpose                           |
| ------------------ | ------------: | --------------------------------- |
| `tos`              |        `0x70` | Value used for TOS/DSCP matching  |
| `tos-mask`         |        `0xf0` | Defines the relevant bits         |
| `mode`             |    `priority` | Selects members based on priority |
| `health-check`     | `Default_DNS` | SLA / health-check object         |
| `link-cost-factor` |      `jitter` | Quality metric                    |
| `priority-members` |         `4 3` | Member 4 preferred over member 3  |

### Verification

* [ ] Confirm DSCP/TOS value.
* [ ] Confirm mask.
* [ ] Confirm the SD-WAN service matches the intended traffic.
* [ ] Confirm the health-check is operational.
* [ ] Confirm priority members are correct.

> [!IMPORTANT]
> DSCP classification does not replace SD-WAN path selection. It acts as one of the criteria used to match traffic to an SD-WAN service.

---

# 3. FGT-1 DSCP Steering

## Rule 1 — Backend / Application Traffic

```text
Source:
    all

Destination:
    192.168.20.200

Strategy:
    Best Quality

Zones:
    1-back
    2-main

SLA:
    test

Quality:
    Packet Loss

Forward DSCP:
    001010

Reverse DSCP:
    001010
```

### Checklist

* [ ] Source is correct.
* [ ] Destination is correct.
* [ ] Correct SD-WAN zones are selected.
* [ ] SLA `test` exists.
* [ ] Packet-loss measurement is enabled.
* [ ] Forward DSCP is correct.
* [ ] Reverse DSCP is correct.
* [ ] Firewall policy permits the traffic.

---

## Rule 2 — Google / Internet Traffic

```text
Source:
    all

Destination:
    Wildcard Google

Strategy:
    Best Quality

Zones:
    1-main
    2-back

SLA:
    test

Quality:
    Latency

Forward DSCP:
    101110

Reverse DSCP:
    101110
```

### Checklist

* [ ] Internet destination object is correct.
* [ ] Application / Internet Service identification is correct.
* [ ] Latency SLA is configured.
* [ ] Main and backup members are ordered correctly.
* [ ] DSCP marking is confirmed with packet capture.
* [ ] Firewall policy permits the traffic.

---

## Firewall → SD-WAN Flow

```text
LAN
 |
 v
Firewall Policy
 |
 v
SD-WAN Service
 |
 v
DSCP / Application / Destination Match
 |
 v
SLA
 |
 v
Selected Member
```

---

# 4. FGT-4 Remote / ISP Side

## Rule 1

```text
Source:
    192.168.20.0/24

Destination:
    192.168.101.0/24

Strategy:
    Best Quality

Zones:
    1-back
    2-main

SLA:
    test

Quality:
    Latency

Forward DSCP:
    001010

Reverse DSCP:
    001010
```

### Checklist

* [ ] Source network is correct.
* [ ] Destination network is correct.
* [ ] Correct SD-WAN zones are selected.
* [ ] SLA is reachable.
* [ ] Latency measurement is operational.
* [ ] DSCP values are correct.
* [ ] Firewall policies allow both directions.

---

## Rule 2

```text
Source:
    all

Destination:
    192.168.101.0/24

Strategy:
    Best Quality

Zones:
    1-back
    2-main

SLA:
    test

Quality:
    Latency

Forward DSCP:
    101110

Reverse DSCP:
    101110
```

### Verification

* [ ] Confirm source/destination matching.
* [ ] Confirm DSCP classification.
* [ ] Confirm SLA state.
* [ ] Confirm selected member.
* [ ] Confirm return-path behavior.

---

# 5. DSCP Packet Capture

## Capture DSCP / TOS

```bash
diagnose sniffer packet any '(ip and ip[1] & 0xfc == 0x70)' 6 0 l
```

### Verify

```text
IP Header
   |
   +-- DS Field
        |
        +-- DSCP
        |
        +-- ECN
```

### Checklist

* [ ] Start packet capture.
* [ ] Generate matching traffic.
* [ ] Confirm the expected DSCP value.
* [ ] Confirm DSCP remains present across the required path.
* [ ] Verify forward direction.
* [ ] Verify reverse direction.
* [ ] Compare packet capture with SD-WAN rule configuration.

> [!TIP]
> If the SD-WAN rule does not match as expected, packet capture is one of the fastest ways to determine whether the traffic actually carries the DSCP value you configured.

---

# 6. DSCP Traffic Shaping

## Traffic Shaper

Example:

```text
Name:
    2k-shape

Type:
    Shared

Priority:
    Low

Application:
    Per Policy

Maximum Bandwidth:
    2 Kbps

DSCP:
    001010
```

### Checklist

* [ ] Create the traffic shaper.
* [ ] Select the correct shaping type.
* [ ] Define priority.
* [ ] Define maximum bandwidth.
* [ ] Configure DSCP classification if required.
* [ ] Apply the shaper to the correct policy.

---

## Traffic Shaping Policy

```text
Source:
    192.168.101.0/24

Destination:
    192.168.20.0/24

Service:
    HTTP

Outgoing Interface:
    FGT-2 Main
    FGT-2 Back

Shared Shaper:
    2k-shape

Reverse Shaper:
    2k-shape
```

### Traffic Flow

```text
192.168.101.0/24
       |
       | DSCP 001010
       v
   SD-WAN Rule
       |
       v
 Main / Backup
       |
       v
 Traffic Shaper
       |
       v
   2 Kbps Limit
       |
       v
192.168.20.0/24
```

### Checklist

* [ ] Traffic matches the firewall policy.
* [ ] Correct DSCP is present.
* [ ] Correct SD-WAN service is selected.
* [ ] Correct WAN member is selected.
* [ ] Traffic shaper is attached to the policy.
* [ ] Shared shaper is correct.
* [ ] Reverse shaper is configured if required.
* [ ] Bandwidth limit is verified.

---

# 7. SD-WAN ECMP

SD-WAN path selection can interact with the routing table and ECMP.

## Common SD-WAN Strategies

```text
Manual
Best Quality
Lowest Cost
```

### Checklist

* [ ] Identify all valid SD-WAN members.
* [ ] Check the routing table.
* [ ] Check for ECMP routes.
* [ ] Determine whether multiple paths have equal route preference.
* [ ] Check SD-WAN service mode.
* [ ] Check SLA-qualified members.
* [ ] Check tie-break behavior.
* [ ] Verify final FIB selection.

---

# 8. Longest Prefix Match

## Example Routing Table

```text
0.0.0.0/0
    -> sdwan-1
    AD 1

0.0.0.0/0
    -> sdwan-2
    AD 1

192.168.102.0/24
    -> port-1
    AD 1
```

The route:

```text
192.168.102.0/24
```

is more specific than:

```text
0.0.0.0/0
```

Therefore:

```text
192.168.102.0/24
        |
        v
Longest Prefix Match
        |
        v
port-1
```

### Checklist

* [ ] Check destination IP.
* [ ] Search for the most specific route.
* [ ] Check route preference.
* [ ] Check administrative distance.
* [ ] Check whether ECMP exists.
* [ ] Identify candidate egress interfaces.
* [ ] Verify the resulting FIB.
* [ ] Compare FIB with SD-WAN member availability.

> [!IMPORTANT]
> Do not troubleshoot SD-WAN selection in isolation. Routing and FIB decisions determine which forwarding paths are available to the SD-WAN decision process.

---

# 9. SD-WAN Tie-Break

## Example

```cli
config system sdwan
    config service
        edit 1
            set name 1
            set mode sla
            set dst all
            set src 192.168.102.0/24
            set priority-members 1 2 3 4
            set tie-break fib-best-match
        next
    end
end
```

## Possible Decision Inputs

```text
Priority
   |
Interface
   |
SLA
   |
FIB Best Match
   |
Input Device
```

### Checklist

* [ ] Confirm SD-WAN service match.
* [ ] Confirm SLA-qualified members.
* [ ] Confirm priority order.
* [ ] Check FIB best-match.
* [ ] Check interface selection.
* [ ] Check input device when relevant.
* [ ] Verify final member with `diagnose sys sdwan service`.

---

# 10. Lowest Cost vs Best Quality

## Lowest Cost

```text
Longest-Match Routes
        |
        v
Available Egress Members
        |
        v
SLA / Cost Evaluation
        |
        v
Lowest-Cost Decision
```

### Checklist

* [ ] Verify destination route.
* [ ] Verify longest-match route.
* [ ] Identify available egress members.
* [ ] Check SLA state.
* [ ] Check configured cost.
* [ ] Verify final selected member.

---

## Best Quality

```text
Latency
Packet Loss
Jitter
    |
    v
Quality Calculation
    |
    v
Best Quality Member
```

### Checklist

* [ ] Verify latency.
* [ ] Verify jitter.
* [ ] Verify packet loss.
* [ ] Confirm SLA thresholds.
* [ ] Confirm eligible members.
* [ ] Confirm selected member.

### Comparison

| Strategy          | Primary Focus                |
| ----------------- | ---------------------------- |
| Manual / Priority | Configured member preference |
| Best Quality      | SLA quality                  |
| Lowest Cost       | SLA/cost evaluation          |

> [!TIP]
> When troubleshooting an unexpected SD-WAN path, first determine the strategy, then determine the candidate members, and finally inspect the routing/FIB constraints.

---

# 11. Application-Based SD-WAN

SD-WAN services can use Internet Service / Application Control categories to classify traffic.

## Example

```cli
config system sdwan
    config service
        edit 1
            set mode sla
            set src 192.168.101.0/24
            set internet-service enable
            set internet-service-app-ctrl-category 5 21
        next
    end
end
```

Example application categories:

```text
Category 5
    Video / Audio

Category 21
    Email
```

## Decision Flow

```text
Client
  |
  v
Application Identification
  |
  +---- Video / Audio
  |
  +---- Email
  |
  v
SD-WAN Rule
  |
  v
SLA / Strategy
  |
  v
WAN Member
```

### Checklist

* [ ] Enable required application identification.
* [ ] Verify Internet Service configuration.
* [ ] Select correct application categories.
* [ ] Create SD-WAN service.
* [ ] Define source/destination.
* [ ] Define SLA.
* [ ] Select appropriate strategy.
* [ ] Verify firewall policy.
* [ ] Verify application classification.
* [ ] Verify selected WAN member.

---

# 12. Firewall Policy vs SD-WAN Rule

This distinction is critical.

## Firewall Policy

Responsible for:

```text
Allow / Deny
Application Identification
Security Profiles
Traffic Shaping
Access Control
```

## SD-WAN Rule

Responsible for:

```text
Traffic Matching
SLA Evaluation
Path Selection
WAN Member Selection
```

## Combined Flow

```text
Firewall Policy
      |
      +-- Allow / Deny
      +-- Application Detection
      |
      v
SD-WAN Rule
      |
      +-- Source
      +-- Destination
      +-- DSCP / TOS
      +-- Application
      +-- SLA
      +-- Strategy
      |
      v
Routing / FIB
      |
      v
WAN Member
```

### Troubleshooting Checklist

* [ ] Firewall policy matches.
* [ ] Firewall action is `accept`.
* [ ] Application is identified correctly.
* [ ] SD-WAN service matches.
* [ ] SLA is healthy.
* [ ] Routing table contains a valid path.
* [ ] FIB contains the expected forwarding decision.
* [ ] Correct WAN member is selected.

---

# 13. Application Steering Troubleshooting

## Policy Route Information

```bash
diagnose firewall proute list
```

### Checklist

* [ ] Check policy-route information.
* [ ] Check SD-WAN service matching.
* [ ] Check application identification.
* [ ] Check SLA state.
* [ ] Check routing table.
* [ ] Check FIB.
* [ ] Check selected SD-WAN member.
* [ ] Capture traffic if required.

---

# 14. End-to-End DSCP SD-WAN Flow

```text
                    Client
                       |
                       | DSCP
                       v
              +----------------+
              | Firewall       |
              | Policy         |
              +----------------+
                       |
                       v
              +----------------+
              | SD-WAN Rule    |
              |                |
              | Source         |
              | Destination    |
              | DSCP / TOS     |
              | Application    |
              +----------------+
                       |
                       v
                 SLA / Quality
                       |
          +------------+------------+
          |                         |
       Main WAN                  Backup WAN
          |                         |
          +------------+------------+
                       |
                       v
                 Traffic Shaper
                       |
                       v
                  Destination
```

### End-to-End Checklist

* [ ] Client generates the expected traffic.
* [ ] DSCP marking is present.
* [ ] Firewall policy matches.
* [ ] Application is identified if required.
* [ ] SD-WAN service matches.
* [ ] DSCP/TOS condition matches.
* [ ] SLA is healthy.
* [ ] Routing/FIB allows the expected path.
* [ ] Correct WAN member is selected.
* [ ] Traffic shaping is applied.
* [ ] Destination receives traffic.
* [ ] Return traffic follows the expected path.

---

# 15. SD-WAN Verification Checklist

## Service Selection

```bash
diagnose sys sdwan service
```

* [ ] Service exists.
* [ ] Rule ID is correct.
* [ ] Matching criteria are correct.
* [ ] SLA state is correct.
* [ ] Selected member is expected.

---

## SLA

```bash
diagnose sys sdwan health-check
```

Check:

```text
Member
SLA
Latency
Jitter
Packet Loss
Status
```

* [ ] SLA is UP.
* [ ] Correct members participate.
* [ ] Latency is acceptable.
* [ ] Jitter is acceptable.
* [ ] Packet loss is acceptable.

---

## Routing

```bash
get router info routing-table all
```

```bash
get router info routing-table static
```

```bash
get router info routing-table bgp
```

* [ ] Destination exists.
* [ ] Longest-prefix route is understood.
* [ ] Administrative distance is correct.
* [ ] ECMP is understood.
* [ ] FIB points to the expected egress.

---

## Policy Routing

```bash
diagnose firewall proute list
```

* [ ] Policy routes are understood.
* [ ] No unexpected policy route overrides forwarding.
* [ ] SD-WAN traffic is not being redirected unexpectedly.

---

## Policy Route Match

```bash
diagnose ip proute match 3.1.1.34 70:4c:a5:86:de:56 port3 22 6
```

* [ ] Source is correct.
* [ ] Destination is correct.
* [ ] MAC address is correct.
* [ ] Input interface is correct.
* [ ] Port is correct.
* [ ] Protocol is correct.

---

# 16. Quick Reference

| Requirement              | Mechanism / Command                    |
| ------------------------ | -------------------------------------- |
| Match traffic by DSCP    | `tos` + `tos-mask`                     |
| Prefer a WAN member      | `priority` / priority members          |
| Select best WAN          | Best Quality + SLA                     |
| Select lowest-cost path  | Lowest Cost + SLA                      |
| Match applications       | Internet Service / Application Control |
| Limit bandwidth          | Traffic Shaper                         |
| Apply DSCP-based shaping | Traffic Shaper + DSCP                  |
| Check SD-WAN service     | `diagnose sys sdwan service`           |
| Check SLA                | `diagnose sys sdwan health-check`      |
| Check routing            | `get router info routing-table all`    |
| Check static routes      | `get router info routing-table static` |
| Check BGP routes         | `get router info routing-table bgp`    |
| Check policy routes      | `diagnose firewall proute list`        |
| Test policy route        | `diagnose ip proute match ...`         |
| Capture DSCP             | `diagnose sniffer packet ...`          |

---

# 17. Design Rules

## 🧠 Memorize These

> **DSCP = Traffic Classification**

> **SLA = WAN Quality Measurement**

> **SD-WAN Rule = Path Selection**

> **Firewall Policy = Security / Access Control**

> **Traffic Shaper = Bandwidth Control**

> **Routing Table = Available Routes**

> **FIB = Final Forwarding Decision**

> **ECMP = Multiple Equal-Cost Paths**

> **TOS Mask = Defines Relevant Matching Bits**

> **Application Control = Application Identification**

---

# 18. Core CLI

## SD-WAN

```bash
diagnose sys sdwan service
diagnose sys sdwan health-check
diagnose sys sdwan neighbor
diagnose sys sdwan packet-history
```

## Routing

```bash
get router info routing-table all
get router info routing-table static
get router info routing-table bgp
```

## Policy Routing

```bash
diagnose firewall proute list
diagnose ip proute match 3.1.1.34 70:4c:a5:86:de:56 port3 22 6
```

## DSCP Capture

```bash
diagnose sniffer packet any '(ip and ip[1] & 0xfc == 0x70)' 6 0 l
```

---

# 19. Key Takeaways

> [!IMPORTANT]
> **Never troubleshoot SD-WAN path selection as a single feature.**

The forwarding decision can involve:

```text
                 Traffic
                    |
                    v
             Firewall Policy
                    |
          +---------+---------+
          |                   |
    Application ID       Security
          |                   |
          +---------+---------+
                    |
                    v
               SD-WAN Rule
                    |
          +---------+---------+
          |         |         |
         DSCP     SLA    Application
          |         |         |
          +---------+---------+
                    |
                    v
                Routing
                    |
                    v
                   FIB
                    |
                    v
             WAN Member
                    |
                    v
            Traffic Shaper
                    |
                    v
               Destination
```

## Final Troubleshooting Sequence

```text
1. Firewall Policy
        |
2. Application Identification
        |
3. SD-WAN Service Match
        |
4. DSCP / TOS Match
        |
5. SLA State
        |
6. Routing Table
        |
7. Longest Prefix Match
        |
8. ECMP / Tie-Break
        |
9. FIB
        |
10. Selected SD-WAN Member
        |
11. Traffic Shaping
        |
12. Packet Capture
```

### 🚦 Golden Rule

```text
MATCH
  ↓
SLA
  ↓
ROUTE
  ↓
FIB
  ↓
SD-WAN MEMBER
  ↓
SHAPER
  ↓
FORWARD
```

---

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

---

> **SheynShield | Engineering Secure Networks**
>
> FortiGate • FortiOS • SD-WAN • Network Security • Routing • Troubleshooting

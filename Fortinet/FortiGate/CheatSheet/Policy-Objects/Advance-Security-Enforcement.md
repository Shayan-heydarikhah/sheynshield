# FortiGate Advanced Security Enforcement  

> **FortiOS | Local-In Policy, TTL Policy, DoS Policy, Anomaly Detection, Quarantine, ACL Offloading, TCAM & Interface Policy**

---

## 📌 Table of Contents

* [1. Packet Processing Security Layers](#1-packet-processing-security-layers)
* [2. Local-In Policy](#2-local-in-policy)
* [3. Local-In vs Administrative Access](#3-local-in-vs-administrative-access)
* [4. Local-In Policy Example](#4-local-in-policy-example)
* [5. Local-In Troubleshooting](#5-local-in-troubleshooting)
* [6. HA Management Interface](#6-ha-management-interface)
* [7. TTL Policy](#7-ttl-policy)
* [8. TTL Policy Example](#8-ttl-policy-example)
* [9. DoS Policy](#9-dos-policy)
* [10. DoS Detection Logic](#10-dos-detection-logic)
* [11. Scan vs Flood](#11-scan-vs-flood)
* [12. ICMP Sweep](#12-icmp-sweep)
* [13. IPS Anomaly Mode](#13-ips-anomaly-mode)
* [14. Continuous vs Periodical Anomaly Mode](#14-continuous-vs-periodical-anomaly-mode)
* [15. Custom DoS Anomaly](#15-custom-dos-anomaly)
* [16. Quarantine Attackers](#16-quarantine-attackers)
* [17. Quarantine Architecture](#17-quarantine-architecture)
* [18. Access Control List (ACL)](#18-access-control-list-acl)
* [19. ACL Hardware Offloading](#19-acl-hardware-offloading)
* [20. TCAM / CAM](#20-tcam--cam)
* [21. FortiGate vs Cisco CEF](#21-fortigate-vs-cisco-cef)
* [22. Data Plane vs Control Plane](#22-data-plane-vs-control-plane)
* [23. CEF Switch vs Punt](#23-cef-switch-vs-punt)
* [24. ACL Troubleshooting](#24-acl-troubleshooting)
* [25. Interface Policy](#25-interface-policy)
* [26. Interface Policy Processing](#26-interface-policy-processing)
* [27. DSRI](#27-dsri)
* [28. DSRI Risks](#28-dsri-risks)
* [29. Interface Policy Example](#29-interface-policy-example)
* [30. Security Layer Comparison](#30-security-layer-comparison)
* [31. Troubleshooting Decision Tree](#31-troubleshooting-decision-tree)
* [32. NSE High-Value Notes](#32-nse-high-value-notes)

---

# 1. Packet Processing Security Layers

One of the most important FortiGate troubleshooting skills is knowing **where a packet is being stopped**.

A simplified model:

```text
                     INCOMING PACKET
                            │
                            ▼
                 ┌────────────────────┐
                 │ Interface / HW ACL │
                 └─────────┬──────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │ Interface Policy   │
                 └─────────┬──────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │ Local-In Policy    │
                 │ (traffic to FGT)   │
                 └─────────┬──────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │ DoS / Anomaly      │
                 └─────────┬──────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │ Firewall Policy    │
                 └─────────┬──────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │ Security Profiles  │
                 └─────────┬──────────┘
                           │
                           ▼
                       FORWARD
```

> ⚠️ This is a **conceptual troubleshooting model**, not a literal packet-processing sequence for every FortiOS feature/platform. Actual processing and hardware offload behavior depend on traffic type, FortiOS version, ASIC/NPU capabilities, and configuration.

---

# 2. Local-In Policy

A **Local-In Policy** controls traffic destined **to the FortiGate itself**, rather than traffic being forwarded through the FortiGate.

Typical targets:

```text
FortiGate Management
SSH
HTTPS
SNMP
PING
DNS
IPsec
BGP
OSPF
Other FortiGate-local services
```

Configuration:

```bash
config firewall local-in-policy
    edit 1
        set interface "port3"
        set srcaddr "kali-1"
        set dstaddr "all"
        set schedule "always"
        set service "ALL"
        set status enable
        set action deny
    next
end
```

Conceptually:

```text
Kali
192.168.101.2
      │
      │ HTTPS / SSH / ICMP / ...
      ▼
FortiGate port3
      │
      ▼
LOCAL-IN POLICY
      │
      └── DENY
```

---

# 3. Local-In vs Administrative Access

Do not confuse Local-In Policies with:

```text
config system interface
```

administrative access settings or administrator trusted-host controls.

### Administrative Access

Example concept:

```text
HTTPS
SSH
PING
SNMP
```

can be enabled/disabled on an interface.

This answers:

> **Which protocols are allowed to access a FortiGate interface?**

---

### Local-In Policy

Local-In Policy answers:

> **Under what source/interface/service conditions should traffic destined to the FortiGate be allowed or denied?**

Think:

```text
Administrative Access
        ↓
"Is HTTPS allowed on this interface?"

Local-In Policy
        ↓
"Which clients are allowed to use it?"
```

---

# 4. Local-In Policy Example

Goal:

```text
Block:
Kali → FortiGate

Allow:
Trusted administrators → FortiGate
```

A common design is:

```text
                 port3
                   │
       ┌───────────┴───────────┐
       │                       │
      Kali                 Admin PC
       │                       │
       ▼                       ▼
   Local-In DENY          Local-In ALLOW
```

This is much more granular than simply disabling:

```text
HTTPS
SSH
PING
```

on the interface.

---

# 5. Local-In Troubleshooting

For a packet destined to the FortiGate itself, debug the traffic.

Example:

```bash
diagnose debug flow filter addr 192.168.101.2
diagnose debug flow filter proto 1
diagnose debug enable
diagnose debug flow trace start 50
```

Here:

```text
proto 1 = ICMP
```

After testing:

```bash
diagnose debug disable
diagnose debug reset
```

### Mental Model

```text
Client
  │
  │ ICMP
  ▼
FortiGate
  │
  ├── Interface admin access?
  ├── Local-In policy?
  └── Other local-service processing?
```

---

# 6. HA Management Interface

For dedicated HA management interfaces, FortiGate supports:

```bash
set ha-mgmt-intf-only enable
```

Conceptually:

```text
             HA Cluster
          ┌───────────────┐
          │               │
       FGT-A            FGT-B
          │               │
          └──────┬────────┘
                 │
          Dedicated Mgmt
             Network
```

This is useful when management traffic should use a dedicated path rather than normal production interfaces.

---

# 7. TTL Policy

FortiGate can use TTL-based policy controls to match packets according to their IP TTL.

Example:

```bash
config firewall ttl-policy
    edit 1
        set action deny
        set status enable
        set srcintf "port3"
        set srcaddr "all"
        set service "ALL"
        set schedule "always"
        set ttl 64
    next
end
```

The important concept is:

```text
Incoming packet
      │
      ▼
TTL value
      │
      ▼
TTL Policy
      │
      ├── Match
      │
      └── Action
```

---

## ⚠️ TTL Is Not a Connection Lifetime

Do not confuse:

```text
TTL Policy
```

with:

```text
Session TTL
```

### IP TTL

```text
Layer 3 packet lifetime / hop limit
```

### Session TTL

```text
FortiGate session-table timeout
```

They solve completely different problems.

---

# 8. TTL Policy Example

Testing from Kali:

```bash
hping3 -c 10 -d 120 -S -p 1 --ttl 64 192.168.101.1
```

Useful for generating packets with a controlled TTL.

Then inspect:

```bash
diagnose debug flow filter addr 192.168.101.2
diagnose debug flow filter proto 1

diagnose debug enable
diagnose debug flow trace start 50
```

---

# 9. DoS Policy

A DoS Policy is designed to detect and mitigate abnormal traffic patterns.

Typical patterns include:

```text
TCP SYN Flood
UDP Flood
ICMP Flood
TCP Port Scan
IP Scan
ICMP Sweep
Other anomaly signatures
```

Conceptually:

```text
Traffic
   │
   ▼
Rate / Pattern Analysis
   │
   ├── Normal
   │
   └── Anomalous
          │
          ▼
      DoS Policy
          │
          ├── Block
          ├── Monitor
          └── Quarantine
```

---

# 10. DoS Detection Logic

DoS detection is not simply:

```text
"If traffic exists → block"
```

It evaluates traffic behavior.

Important dimensions can include:

```text
Packets per second
Connection rate
Source behavior
Destination behavior
Protocol
Port
Time interval
Traffic direction
Anomaly type
```

---

## Scan vs Flood

This distinction is important.

### Flood

Usually:

```text
One source
     │
     ├── Many packets
     ├── High rate
     └── Often one/few destinations
```

### Scan

Usually:

```text
One source
     │
     ├── Many destinations
     ├── Many ports
     └── Discovery behavior
```

---

# 11. Scan vs Flood

| Attack Pattern | Typical Behavior           |
| -------------- | -------------------------- |
| SYN Flood      | High SYN rate              |
| UDP Flood      | High UDP packet rate       |
| ICMP Flood     | High ICMP rate             |
| Port Scan      | Many ports                 |
| Host Scan      | Many destination IPs       |
| ICMP Sweep     | ICMP toward many hosts     |
| TCP Port Scan  | Many TCP destination ports |

### Memory Trick

```text
FLOOD
→ "How many packets?"

SCAN
→ "How many targets?"
```

---

# 12. ICMP Sweep

An ICMP sweep attempts to discover active hosts across an address range.

Conceptually:

```text
Attacker
   │
   ├── ICMP → 192.168.1.1
   ├── ICMP → 192.168.1.2
   ├── ICMP → 192.168.1.3
   ├── ICMP → 192.168.1.4
   └── ...
```

The important detection dimension is:

```text
Source
   ↓
Many destinations
```

This differs from an ICMP flood:

```text
Source
   ↓
One destination
   ↓
Huge packet rate
```

---

# 13. IPS Anomaly Mode

FortiGate IPS anomaly detection can use different handling behavior.

Conceptually:

```bash
config ips global
    set anomaly-mode ...
end
```

The exact available values and behavior depend on FortiOS version.

A threshold example:

```text
Threshold = 10 packets
Observed = 20 packets
```

The anomaly engine can trigger mitigation once the configured/default threshold is exceeded.

---

# 14. Continuous vs Periodical Anomaly Mode

## Continuous

Conceptually:

```text
Threshold = 10

Packets:
1 2 3 4 5 6 7 8 9 10
                  │
                  ▼
              Threshold
                  │
                  ▼
              Mitigation
```

Traffic continues to be treated as anomalous while the condition persists.

---

## Periodical

The system evaluates behavior over intervals rather than treating the entire event as one continuously blocked condition.

Conceptually:

```text
Time
│
├── Interval 1
│     └── Detect anomaly
│
├── Interval 2
│     └── Re-evaluate
│
├── Interval 3
│     └── Re-evaluate
│
└── ...
```

> ⚠️ Exact behavior should be verified against the FortiOS version because anomaly-mode implementation and available options can differ between releases.

---

# 15. Custom DoS Anomaly

You can inspect anomaly configuration and runtime state with diagnostic commands such as:

```bash
diagnose ips anomaly config
diagnose ips anomaly list
```

Example conceptual output:

```text
id=icmp_flood
ip=192.168.2.50
dos_id=1
pps=46
freq=50
exp=998
```

Useful interpretation:

```text
pps
↓
Packet-rate related measurement

freq
↓
Frequency / detection-related value

exp
↓
Expiration / accumulated anomaly state
```

> The exact meaning of individual fields is version-dependent. Do not treat a diagnostic field as a universal formula without checking the corresponding FortiOS release.

---

# 16. Quarantine Attackers

A DoS policy can quarantine detected attackers.

Example:

```bash
config firewall dos-policy
    edit 1
        config anomaly
            edit "tcp_port_scan"
                set quarantine attacker
                set quarantine-log enable
                set quarantine-expire 00d00h01m
            next
        end
    next
end
```

Conceptually:

```text
Attacker
   │
   ▼
Anomaly Detection
   │
   ▼
Threshold exceeded
   │
   ▼
Quarantine
   │
   ▼
Temporary block
```

---

# 17. Quarantine Architecture

Quarantine is more powerful than simply dropping one packet.

Conceptually:

```text
Attacker IP
     │
     ▼
Kernel Quarantine List
     │
     ├── IPS
     ├── AV
     ├── DLP
     └── DoS-related enforcement
```

Inspect quarantine:

```bash
diagnose user quarantine list
```

---

## 🧠 Important

A quarantined attacker can be blocked across traffic paths/policies where the relevant security enforcement features participate.

Therefore:

```text
Quarantine
≠
Only one firewall policy
```

It can become a broader enforcement mechanism.

---

# 18. Access Control List (ACL)

FortiGate platforms with supported switch-fabric/TCAM architectures can process ACLs in hardware.

Conceptually:

```text
Packet
  │
  ▼
Switch Fabric / TCAM
  │
  ├── Match
  │
  └── Action
```

This can reduce CPU involvement for supported ACL processing.

---

# 19. ACL Hardware Offloading

On supported platforms, interfaces connected to internal switch fabrics with TCAM capabilities can offload ACL processing.

Conceptually:

```text
Ingress Packet
      │
      ▼
Switch Fabric
      │
      ▼
TCAM lookup
      │
 ┌────┴────┐
 │         │
Match    No Match
 │         │
 ▼         ▼
Action    CPU / next stage
```

If hardware offloading is unavailable:

```text
Packet
  │
  ▼
CPU
  │
  ▼
ACL processing
```

---

## Supported Platform Warning

Do **not** memorize a universal model/port list from a  .

ACL hardware capabilities depend on:

```text
FortiGate model
FortiOS version
ASIC generation
Physical port architecture
Switch-fabric connection
Interface type
```

Always verify the exact platform in the corresponding FortiOS Hardware Acceleration / ACL documentation.

---

# 20. TCAM / CAM

### TCAM

**Ternary Content-Addressable Memory**

Supports:

```text
0
1
X = Don't Care
```

This makes TCAM highly useful for:

```text
ACL
IP prefix matching
Masks
Security rules
Hardware classification
```

---

## CAM vs TCAM

| Memory | Values    | Common Use                |
| ------ | --------- | ------------------------- |
| CAM    | 0 / 1     | Exact matching            |
| TCAM   | 0 / 1 / X | Masked / ternary matching |

Memory trick:

```text
CAM
→ Exact Match

TCAM
→ Flexible / Masked Match
```

---

# 21. FortiGate vs Cisco CEF

A useful conceptual comparison:

| Cisco               | FortiGate                           |
| ------------------- | ----------------------------------- |
| CEF                 | NP/NPU/ASIC forwarding architecture |
| FIB                 | Routing/forwarding information      |
| Adjacency Table     | Neighbor/adjacency information      |
| Hardware forwarding | NPU/ASIC offload                    |
| Punt to CPU         | Software/CPU processing             |

Do **not** assume:

```text
CEF = FortiGate NPU
```

They are not identical technologies.

The useful comparison is at the **architectural level**:

```text
Control Plane
      │
      ▼
Build forwarding information
      │
      ▼
Data Plane
      │
      ▼
Fast packet forwarding
```

---

# 22. Data Plane vs Control Plane

## Control Plane

Responsible for building/maintaining information such as:

```text
Routing
ARP / Neighbor Discovery
Protocol state
Control information
```

Conceptually:

```text
CPU
 │
 ├── Routing Protocols
 ├── ARP/ND
 └── Control Logic
```

---

## Data Plane

Responsible for forwarding packets at high speed.

Conceptually:

```text
Packet
  │
  ▼
FIB / Forwarding Information
  │
  ▼
ASIC / NPU
  │
  ▼
Forward
```

---

# 23. CEF Switch vs Punt

In Cisco terminology:

### CEF Switching

```text
Packet
  ↓
FIB
  ↓
Adjacency
  ↓
Fast forwarding
```

### Punt

```text
Packet
  ↓
Not suitable for fast path
  ↓
CPU
  ↓
Software processing
```

FortiGate has a similar high-level distinction:

```text
Fast Path
   ↓
ASIC / NPU

Exception / unsupported path
   ↓
CPU
```

This is one reason why packet captures and debug behavior can differ depending on whether traffic is hardware-offloaded.

---

# 24. ACL Troubleshooting

Useful commands:

```bash
diagnose firewall acl counter
```

Clear counters:

```bash
diagnose firewall acl clearcounter
```

Then:

```text
1. Clear counters
2. Generate test traffic
3. Check counters
4. Identify matching rule
5. Verify hardware/software path
```

---

# 25. Interface Policy

Interface Policies can provide an additional traffic-control layer associated with an interface.

Example:

```bash
config firewall interface-policy
    edit 1
        set status enable
        set logtraffic all
        set interface "port3"
        set srcaddr "all"
        set dstaddr "all"
        set service "ALL"
        set dsri disable

        set webfilter-profile-status enable
        set webfilter-profile "wf-test"

        set ips-sensor-status enable
        set ips-sensor "high"
    next
end
```

---

# 26. Interface Policy Processing

A useful conceptual model:

```text
Incoming Traffic
       │
       ▼
Interface Policy
       │
       ├── Match?
       │
       ├── Logging
       ├── Security checks
       └── Interface-level control
       │
       ▼
Normal packet processing
```

Interface Policies are particularly useful when traffic needs to be controlled at an earlier interface-associated stage.

---

## ⚠️ Important

Do not assume an Interface Policy is simply:

```text
"Firewall Policy before Firewall Policy"
```

The actual processing path depends on FortiOS traffic type and feature implementation.

Use packet-flow diagnostics to determine the real behavior.

---

# 27. DSRI

**DSRI = Dynamic Source Route Injection**

It can influence routing/forwarding behavior associated with traffic matching specific policies/features.

Conceptually:

```text
Traffic
   │
   ▼
Policy / Interface Policy
   │
   ▼
DSRI
   │
   ▼
Dynamic forwarding decision
```

Potential use cases include environments involving:

```text
Policy routing
SD-WAN
Asymmetric paths
Dynamic forwarding
```

---

# 28. DSRI Risks

DSRI should be treated carefully because it can affect forwarding behavior.

Particularly investigate:

```text
Asymmetric Routing
Policy Routes
SD-WAN
Routing Table
Adjacency / Neighbor Information
NPU Fast Path
Security Inspection
```

A useful troubleshooting principle:

```text
Routing looks correct
        │
        ▼
But packet takes unexpected path
        │
        ▼
Check:
Policy Route
SD-WAN
DSRI
NPU / ASIC
```

---

## ⚠️ Security Consideration

Do not assume every security feature behaves identically when traffic is moved into a different forwarding path.

When troubleshooting DSRI-related behavior, verify:

```text
Routing
+
Security Inspection
+
NPU Offload
+
Session State
```

as one system.

---

# 29. Interface Policy Example

Suppose:

```text
LAN → WAN
```

Normal firewall policy provides:

```text
LAN
 ↓
WAN
 ↓
Security Policy
```

An interface policy can provide additional controls associated with the ingress interface.

Example:

```text
Client
  │
  ▼
port3
  │
  ▼
Interface Policy
  │
  ├── IPS
  ├── Web Filter
  └── Logging
  │
  ▼
Normal Forwarding / Security Policy
```

This can be useful for specialized traffic-control scenarios.

---

# 30. Security Layer Comparison

| Feature                | Primary Purpose                  | Traffic Focus              |
| ---------------------- | -------------------------------- | -------------------------- |
| Interface Admin Access | Enable local services            | To FortiGate               |
| Local-In Policy        | Granular local-service filtering | To FortiGate               |
| TTL Policy             | TTL-based packet matching        | Inbound packets            |
| DoS Policy             | Detect/mitigate anomalies        | Forwarded traffic          |
| IPS Anomaly            | Detect abnormal patterns         | Traffic inspection         |
| Quarantine             | Temporarily isolate attacker     | Source-based               |
| Hardware ACL           | Fast L2/L3 filtering             | Supported interfaces       |
| Interface Policy       | Interface-associated filtering   | Traffic entering interface |
| Firewall Policy        | Main traffic authorization       | Forwarded traffic          |
| Security Profiles      | Deep security inspection         | Forwarded traffic          |

---

# 31. Troubleshooting Decision Tree

```text
                PACKET PROBLEM
                      │
                      ▼
            Is destination FortiGate?
                 │          │
                YES         NO
                 │           │
                 ▼           ▼
             Local-In     Forwarded
                 │           │
                 │           ▼
                 │      Interface Policy?
                 │           │
                 │           ▼
                 │       DoS / Anomaly?
                 │           │
                 │           ▼
                 │      Firewall Policy
                 │           │
                 │           ▼
                 │      Security Profiles
                 │           │
                 └──────┬────┘
                        ▼
                 ASIC / NPU Path?
                        │
                 ┌──────┴──────┐
                 ▼             ▼
               Fast          CPU
               Path          Path
                 │             │
                 └──────┬──────┘
                        ▼
                  Packet Capture
                  + Debug Flow
```

---

# 32. Attack-Type Recognition

## SYN Flood

```text
Many SYN packets
      ↓
Same / few destination(s)
      ↓
High connection rate
```

Test environment:

```bash
hping3 -S -p 80 --flood 192.168.20.200
```

> Use traffic-generation commands only in an isolated lab or an explicitly authorized test environment.

---

## TCP Port Scan

```text
One source
    ↓
Many destination ports
    ↓
Discovery behavior
```

---

## Host Scan

```text
One source
    ↓
Many destination IPs
```

---

## ICMP Sweep

```text
One source
    ↓
ICMP
    ↓
Many destination IPs
```

---

# 33. High-Value Diagnostic Commands

### Debug Local-In / Packet Flow

```bash
diagnose debug flow filter addr 192.168.101.2
diagnose debug flow filter proto 1
diagnose debug enable
diagnose debug flow trace start 50
```

Stop:

```bash
diagnose debug disable
diagnose debug reset
```

---

### IPS Anomaly

```bash
diagnose ips anomaly config
diagnose ips anomaly list
```

---

### Quarantine

```bash
diagnose user quarantine list
```

---

### ACL Counters

```bash
diagnose firewall acl counter
diagnose firewall acl clearcounter
```

---

### System Performance

```bash
get system performance status
```

---

# 34. NSE Exam Mental Model 🧠

```text
                TRAFFIC
                   │
          ┌────────┴────────┐
          │                 │
          ▼                 ▼
    TO FORTIGATE        THROUGH FORTIGATE
          │                 │
          ▼                 ▼
      LOCAL-IN         INTERFACE POLICY
          │                 │
          │                 ▼
          │              DoS/IPS
          │                 │
          │                 ▼
          │          FIREWALL POLICY
          │                 │
          │                 ▼
          │         SECURITY PROFILES
          │                 │
          └────────┬────────┘
                   ▼
             NPU / ASIC / CPU
                   │
                   ▼
                FORWARD
```

---

# 35. 🔥 Golden Rules

### Rule #1

```text
Traffic TO FortiGate
→ Think Local-In
```

### Rule #2

```text
Traffic THROUGH FortiGate
→ Think Firewall Policy
```

### Rule #3

```text
Need to control FortiGate services?
→ Check Interface Admin Access
   + Local-In Policy
```

### Rule #4

```text
Huge traffic rate?
→ Think DoS / Anomaly
```

### Rule #5

```text
Many ports?
→ Think Port Scan
```

### Rule #6

```text
Many destination IPs?
→ Think Host Scan / ICMP Sweep
```

### Rule #7

```text
Hardware ACL
→ TCAM / Switch Fabric
→ CPU may be bypassed for supported traffic
```

### Rule #8

```text
Unexpected forwarding path?
→ Check Policy Route
→ SD-WAN
→ DSRI
→ NPU/ASIC
```

### Rule #9

```text
Debug does not show expected packets?
→ Check hardware offload / fast path
```

### Rule #10

```text
Never confuse:

IP TTL
      ≠
Session TTL
      ≠
DoS threshold
```

---

# 36. Advanced Troubleshooting Matrix

| Symptom                              | First Things to Check                        |
| ------------------------------------ | -------------------------------------------- |
| Cannot ping FortiGate                | Interface ping access → Local-In Policy      |
| Cannot SSH to FortiGate              | Interface SSH access → Local-In Policy       |
| Traffic through FGT is flooded       | DoS Policy → IPS anomaly                     |
| Port scan not detected               | IPS/DoS anomaly configuration                |
| Attacker remains blocked             | Quarantine list                              |
| ACL rule does not increment          | ACL support/offload/interface architecture   |
| Unexpected CPU usage                 | ASIC/NPU offload + unsupported path          |
| Unexpected routing                   | Routing table → Policy Route → SD-WAN → DSRI |
| Packet missing from debug            | Fast path / hardware offload                 |
| TTL-based traffic blocked            | TTL Policy + actual packet TTL               |
| Security inspection appears bypassed | Forwarding path + DSRI + NPU/ASIC            |

---

# 37. Expert Architecture View

The most useful way to understand these features is **not by memorizing CLI commands**.

Think in terms of **where the decision happens**:

```text
                 SECURITY DECISION POINTS

                         PACKET
                           │
                           ▼
                 ┌──────────────────┐
                 │ Hardware ACL      │
                 │ TCAM / Fabric     │
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │ Interface Layer   │
                 │ Interface Policy  │
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │ Local-In          │
                 │ FortiGate itself  │
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │ DoS / Anomaly     │
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │ Firewall Policy   │
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │ Security Profiles │
                 └────────┬─────────┘
                          │
                          ▼
                    NPU / CPU
                          │
                          ▼
                       OUTPUT
```

> **Key engineering principle:** When troubleshooting FortiGate, first identify whether the packet is **to the FortiGate, through the FortiGate, or being handled in hardware**. That single distinction eliminates a huge amount of blind troubleshooting.

---

# 📚 Related  s

This topic connects directly with:

* **FortiGate Packet Flow**
* **FortiGate DoS Policy**
* **FortiGate IPS Anomaly Detection**
* **FortiGate Local-In Policy**
* **FortiGate NPU / ASIC Offloading**
* **FortiGate Policy Routing**
* **FortiGate SD-WAN**
* **FortiGate DSRI**
* **FortiGate Debug Flow**
* **FortiGate Hardware Acceleration**
* **FortiGate Session Troubleshooting**
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

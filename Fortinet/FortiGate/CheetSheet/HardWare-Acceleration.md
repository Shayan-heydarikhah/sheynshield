# FortiGate Hardware Acceleration Cheatsheet — Part 1 & Part 2

> **Scope:** SoC, SPU, NPU, CP, NTurbo, NP6/NP6Lite/NP6XLite, NP7, fast-path, ISF, NP Direct, IPsec offload, troubleshooting, HPE and optimization.

---

# 🧩 1. FortiGate Hardware Architecture

```text
FortiGate
   |
   +-- CPU
   |     |
   |     +-- SoC
   |           |
   |           +-- SPU
   |                 |
   |                 +-- NPU
   |                 +-- CP
   |
   +-- Memory
   |
   +-- Interfaces
```

### Entry-Level Devices

Some entry-level FortiGate platforms do not include a dedicated **Content Processor (CP)**.

### Processor Generations

| Processor  | Typical Position  | Notes                                |
| ---------- | ----------------- | ------------------------------------ |
| `NP6`      | Network Processor | High-performance packet acceleration |
| `NP6XLite` | Network Processor | Enhanced/lower-cost NP               |
| `NP6Lite`  | Network Processor | Lightweight NP                       |
| `NP7`      | Network Processor | Newer/high-throughput architecture   |
| `CP`       | Content Processor | Security/content processing          |
| `CP9`      | Content Processor | High-end content acceleration        |
| `CP9XLite` | Content Processor | Reduced-capability variant           |
| `CP9Lite`  | Content Processor | Entry/lower-capability variant       |

### CP / SoC Relationship

```text
CP9 / CP9XLite  → SoC4
CP9Lite         → SoC3
```

### IPsec Engine Capacity

| Processor | IPsec Engines |
| --------- | ------------: |
| CP9       |            16 |
| CP9XLite  |             5 |
| CP9Lite   |             1 |

---

# 🔍 2. Hardware Identification

```bash
get hardware status
```

Look for information such as:

```text
ASIC version
```

This is one of the first commands to use when determining the hardware acceleration architecture of a FortiGate.

---

# ⚡ 3. NTurbo / Fast Path

**NTurbo** accelerates eligible security-processing flows by creating a more direct data path through hardware.

Conceptually:

```text
Ingress
   |
   v
CPU
   |
   | First packet / policy decision
   v
Security Processing
   |
   v
NP / CP
   |
   v
Fast Path
   |
   v
Egress
```

Instead of processing every packet through the main CPU:

```text
CPU
 |
 +-- First packet
 |
 +-- Policy / session lookup
 |
 +-- Security decision
 |
 v
NP
 |
 +-- Subsequent packets
 |
 v
Egress
```

---

# 🛠️ 4. NTurbo Configuration

## NP Acceleration

```bash
config ips global
    set np-accel-mode basic
end
```

### `basic`

```text
Default mode
    |
    +-- Basic NP acceleration
    +-- Direct data path
    +-- Ingress → Egress through ASIC
    +-- IPS features can participate
```

---

## Advanced CP Acceleration

```bash
config ips global
    set cp-accel-mode advanced
end
```

### Advanced Mode

Provides more advanced pattern-matching / processing capabilities.

> Requires hardware supporting the corresponding CP architecture; availability depends on the FortiGate model.

### Quick Check

```text
If the CLI command does not exist
        ↓
The corresponding acceleration capability
may not exist on the platform / FortiOS build.
```

---

# 🚫 5. When NTurbo / NP Offloading Does Not Work

Even when acceleration is enabled, some sessions remain CPU-processed.

## NP Acceleration Disabled

Firewall policy:

```bash
set auto-asic-offload disable
```

Result:

```text
Traffic
   ↓
CPU
   ↓
No NP fast-path offload
```

---

## Proxy-Based Security Profiles

Proxy-based inspection can prevent normal NP offloading.

Examples:

```text
Proxy-based security profile
        ↓
CPU / proxy processing
        ↓
No normal fast-path NP offload
```

---

## Session Helpers

Some protocols require FortiOS session helpers.

### FTP

FTP sessions may not be offloaded because they use the FTP session helper.

```text
FTP
 |
 +-- Session Helper
       |
       └── CPU processing
```

---

## Interface Policies

Offloading can be affected when interface policies are configured on the ingress/egress path.

---

## DoS Policies

DoS policies attached to the ingress/egress path can affect normal acceleration behavior.

---

## Tunnels

Traffic to/from tunneled interfaces may not be eligible for NTurbo offloading.

Examples:

```text
IPsec
IP-in-IP
SSL VPN
GRE
CAPWAP
...
```

Conceptually:

```text
Tunneled Traffic
       |
       v
CPU / tunnel processing
       |
       v
Security processing
```

---

# 🧠 6. NP6 / NP7 VLAN MAC Consideration

On some NP6/NP7 + virtual-clustering configurations:

```text
VLAN MAC != Physical Interface MAC
```

can create an offloading problem.

Possible symptom:

```text
Traffic
   |
   v
VLAN Interface
   |
   v
NP6 / NP7
   |
   X
Packet dropped
```

### Potential Workaround

Disable NP offloading on the firewall policy accepting the traffic:

```bash
config firewall policy
    edit <policy-id>
        set auto-asic-offload disable
    next
end
```

### However

NP offloading can still work if:

* Other network devices learn the FortiGate MAC through ARP.
* Reply traffic's destination MAC matches the underlying interface MAC.

> **Troubleshooting rule:** Do not assume every VLAN/physical-MAC mismatch breaks offloading. Verify actual behavior with NPU diagnostics.

---

# 🔌 7. NP Port Mapping

## NP6

```bash
get hardware npu np6 port-list
```

```bash
diagnose npu np6 port-list
```

## NP6XLite

```bash
get hardware npu np6xlite port-list
```

## NP6Lite

```bash
get hardware npu np6lite port-list
```

```bash
diagnose npu np6lite port-list
```

Use these commands to determine:

```text
Interface
   ↓
NP assignment
   ↓
Ingress / Egress relationship
```

---

# 📡 8. NetFlow / sFlow and NP6 Offloading

### NetFlow

NP6 / NP6XLite / NP6Lite offloading can remain supported when NetFlow is configured on supported interfaces.

```text
NetFlow
   |
   +-- Firewall session information
   |
   +-- NP offloading can continue
```

### sFlow

Configuring sFlow on an interface can disable NP6/NP6XLite/NP6Lite offloading for traffic on that interface.

```text
sFlow
  ↓
NP offloading affected
```

> **Key distinction:** NetFlow and sFlow do not have the same impact on NP offloading.

---

# 🚀 9. NP6 vs NP6XLite vs NP6Lite

| Feature               | NP6                          | NP6XLite           | NP6Lite            |
| --------------------- | ---------------------------- | ------------------ | ------------------ |
| Position              | High-end                     | Mid-level          | Entry-level        |
| Approx. throughput    | ~40 Gbps                     | ~36 Gbps           | ~10 Gbps           |
| AES-GCM               | Limited / platform dependent | Supported          | Limited            |
| Processing capability | High                         | Enhanced           | Lightweight        |
| CPU dependency        | Lower                        | Lower              | Higher             |
| DPI capability        | Hardware dependent           | Hardware dependent | More CPU-dependent |

> Actual throughput and supported cryptographic/security features are model-dependent. Always verify the specific FortiGate datasheet.

---

# 🆕 10. NP7

NP7 provides newer hardware acceleration architecture and higher throughput compared with previous NP generations.

Examples of NP7-based hyperscale platforms include:

```text
FortiGate 1800F
FortiGate 1801F
FortiGate 2600F
FortiGate 2601F
FortiGate 3500F
FortiGate 3501F
FortiGate 4200F
FortiGate 4201F
FortiGate 4400F
FortiGate 4401F
```

---

# 🏎️ 11. Why Hardware Acceleration Matters

### Higher Throughput

```text
CPU-only
   ↓
CPU becomes bottleneck
```

versus:

```text
CPU
 ↓
NPU / CP
 ↓
Hardware forwarding
```

### Benefits

* Higher throughput
* Lower latency
* Lower CPU utilization
* Better scalability
* Efficient handling of very large session counts

---

# 🔄 12. Fast-Path Session Flow

```text
              First Packet
                   |
                   v
             +-----------+
             |    CPU    |
             +-----------+
                   |
          Policy / Session Check
                   |
                   v
             Fast-Path Check
                   |
            +------+------+
            |             |
          Eligible      Not Eligible
            |             |
            v             v
           NPU           CPU
            |
            v
       Subsequent Packets
            |
            v
          Egress
```

### Important Concept

Only the initial/session-establishment processing necessarily requires the CPU.

Eligible subsequent packets can be handled directly by hardware.

---

# ✅ 13. Fast-Path Requirements

A session generally needs to match supported traffic characteristics.

### Layer 2

```text
IPv4  → 0x0800
IPv6  → 0x86DD
```

### Layer 3

```text
IPv4
IPv6
```

### Layer 4

```text
TCP
UDP
ICMP
SCTP
```

### Common Blocker

```text
Proxy-based inspection
        ↓
CPU / proxy processing
        ↓
Normal NP fast-path unavailable
```

---

# 🔐 14. NP6 Offloaded Features

## IPsec

Depending on platform/cipher:

```text
AES
3DES
SHA-2
AES-GCM → NP6XLite and supported platforms
```

---

## Traffic Shaping

NP6XLite can provide enhanced statistics.

Other NP processors can offload traffic shaping while some drop information may not be logged in the same way.

---

## Anomaly Checks

```text
Malformed Packet
       |
       v
NPU
       |
       X
Drop before CPU
```

This protects CPU resources from malformed traffic.

---

## Multicast

NP acceleration can support:

```text
Multicast
Multicast over IPsec
```

---

## LAG

Link aggregation can distribute sessions across multiple NP processors where supported.

```text
               LAG
                |
       +--------+--------+
       |        |        |
      NP1      NP2      NP3
```

---

## Inter-VDOM Links

Inter-VDOM traffic can be offloaded in supported configurations.

> Some interface types, such as eMAC VLAN configurations, have additional limitations.

---

# ⚙️ 15. System NPU Configuration

## Fast Path

```bash
config system npu
    set fastpath enable
end
```

---

## Dedicated Management CPU

```bash
config system npu
    set dedicated-management-cpu enable
end
```

Concept:

```text
CPU0
 |
 +-- Reserved for management
```

---

## CAPWAP Offload

```bash
config system npu
    set capwap-offload enable
end
```

> Only useful where supported by the platform/configuration.

---

# 🔐 16. IPsec NPU Acceleration

```bash
config system npu
    set ipsec-asic-offload enable
end
```

---

## IPsec Inbound Cache

```bash
config system npu
    set ipsec-inbound-cache disable
end
```

Can be useful when troubleshooting IPsec anti-replay / packet-ordering problems.

> ⚠️ Disabling security-related caching should be done deliberately and validated against the specific problem.

---

## ESP-over-UDP / NAT-T

```bash
config system npu
    set uesp-offload enable
end
```

Used for supported ESP-over-UDP acceleration.

---

# 🎯 17. Hardware QoS / Shaping

```bash
config system npu
    set intf-shaping-offload enable
    set qos-mode priority
end
```

### Concept

```text
Traffic
   |
   v
NPU
   |
   +-- QoS
   +-- Shaping
   +-- Priority
   |
   v
Egress
```

---

# 🔎 18. Verify NPU Offloading

## Session Information

```bash
diagnose sys session list
```

Look for fields such as:

```text
npu_info
no_ofld_reason
```

### Interpretation

```text
npu_info
   ↓
Session is associated with hardware offloading
```

```text
no_ofld_reason
   ↓
Why session was not offloaded
```

---

## Packet Capture

```bash
diag sniffer packet port1 'host 1.1.1.1'
```

Useful for correlating:

```text
Packet arrival
    ↓
Policy
    ↓
Session
    ↓
Hardware forwarding
```

---

# 📊 19. NP Session Statistics

```bash
diagnose npu np6 session-stats 0
```

Shows session/offloading statistics for the NP.

---

## IPsec Offload Statistics

```bash
diagnose npu np6 ipsec-stats
```

Useful when troubleshooting:

```text
IPsec
  |
  +-- ASIC offload
  +-- Encryption/decryption
  +-- Hardware statistics
```

---

# 🛠️ 20. Troubleshooting Common Offload Problems

## Problem: TCP Sessions Fail Under Heavy Load

Potential optimization:

```bash
config firewall policy
    edit <policy-id>
        set delay-tcp-npu-session enable
    next
end
```

### Concept

```text
TCP Session
     |
     +-- CPU
     |
     +-- NPU
     |
     v
Session Search Engine
     |
     v
Cached session lookup
```

This can help in environments where TCP sessions are being dropped during high load.

---

# 🔐 21. IPsec Anti-Replay / Packet-Order Problems

### Symptom

```text
IPsec
   |
   +-- Packet delay / reordering
   |
   +-- Anti-replay drops
```

Potential troubleshooting action:

```bash
config system npu
    set ipsec-inbound-cache disable
end
```

> ⚠️ This changes IPsec acceleration behavior. Use it as a targeted troubleshooting/optimization step, not as a generic default.

---

# 🧟 22. Session Drift / Orphaned Sessions

### Symptom

```text
CPU session state
       ≠
NPU session state
```

This can create stale/orphaned sessions.

### Purge

```bash
diagnose npu np6 sse-purge-drift 0
```

Concept:

```text
CPU
 |
 +-- Session A
 |
 X
 |
NPU
 |
 +-- Stale Session A
```

The purge operation removes stale session state.

---

# 🔀 23. XAUI Load Balancing

When multiple XAUI ports are available, traffic can be balanced across interfaces.

```bash
config system npu
    config xaui-loadbalance
        ...
    end
end
```

Concept:

```text
                 Traffic
                    |
          +---------+---------+
          |                   |
       XAUI-1              XAUI-2
          |                   |
         NP                    NP
```

---

# 📈 24. Advanced NP Optimization

### Use LAG

```text
LAG
 |
 +-- NP1
 +-- NP2
 +-- NP3
 +-- NP4
```

Distribute sessions across available processors.

---

### Round-Robin Groups

For high-throughput flows, assign interfaces to suitable round-robin groups where supported.

---

# 🛡️ 25. Host Protection Engine — HPE

The **Host Protection Engine (HPE)** helps protect the CPU against high-rate traffic such as DDoS/SYN floods.

Conceptually similar to control-plane protection mechanisms on other vendors.

```text
Internet
   |
   | Massive SYN/UDP traffic
   v
+-------+
|  HPE  |
+-------+
   |
   +-- Rate limit
   +-- Protect CPU
   |
   v
FortiGate
```

---

## HPE Configuration

```bash
config system np6
    edit np6_0
        config hpe
            set tcpsyn-max 600000
            set udp-max 200000
            set enable-shaper enable
        end
    end
end
```

### Meaning

| Setting         |  Example | Purpose                 |
| --------------- | -------: | ----------------------- |
| `tcpsyn-max`    | `600000` | Maximum SYN packets/sec |
| `udp-max`       | `200000` | Maximum UDP packets/sec |
| `enable-shaper` | `enable` | Enable HPE shaping      |

### Monitor

```bash
diagnose npu np6 hpe
```

---

# 📡 26. Multicast Optimization

```bash
config system npu
    set mcast-session-accounting session-based
    set lag-out-port-select enable
end
```

### `mcast-session-accounting`

Useful when many multicast sessions exist.

### `lag-out-port-select`

Optimizes egress selection across LAG members.

---

# 🧩 27. Example NP6 Architectures

| Platform                | Architecture                       |
| ----------------------- | ---------------------------------- |
| FortiGate 300E / 400E   | Single NP6, 16 × 1G ports          |
| FortiGate 1500D         | Dual NP6 + ISF                     |
| FortiGate 3960E / 3980E | Multiple NP6 + round-robin support |
| FortiGate 1000D         | NP Direct                          |

---

# 🔗 28. What Is ISF?

**ISF = Integrated Switch Fabric**

ISF is a high-speed internal switching architecture connecting:

```text
          +----------------+
          |      CPU       |
          +-------+--------+
                  |
          +-------+-------+
          |      ISF      |
          +---+-------+---+
              |       |
             NP1     NP2
              |       |
          Interfaces / Ports
```

### Purpose

ISF allows:

* Any-to-any NP forwarding
* Multiple NP connectivity
* Cross-NP offloading
* Load distribution
* Flexible ingress/egress mapping

### Analogy

Think of ISF as an internal high-speed switching fabric.

```text
External Port
      |
      v
     NP1
      |
      v
     ISF
      |
      v
     NP2
      |
      v
External Port
```

---

# 🏗️ 29. ISF Platforms

Examples:

```text
FortiGate 1500D
FortiGate 3200D
FortiGate 5001D
```

### Verify

```bash
get hardware npu np6 port-list
```

Look for:

```text
Cross-chip offloading: yes
```

This indicates cross-NP forwarding through the internal fabric.

---

# ⚡ 30. What Is NP Direct?

**NP Direct** is a more direct architecture where there is no ISF between NP processors.

```text
Port
 |
 v
NP
 |
 v
Port
```

Instead of:

```text
Port
 |
 v
NP1
 |
 v
ISF
 |
 v
NP2
 |
 v
Port
```

---

# 🏎️ 31. NP Direct Characteristics

### Advantages

* Lowest possible latency
* Avoids ISF traversal
* Suitable for latency-sensitive workloads

### Requirement

Traffic must enter and exit through the appropriate/same NP.

```text
Ingress
   |
   v
 NP1
   |
   v
Egress
   |
   v
Same NP
```

If:

```text
Ingress → NP1
Egress  → NP2
```

the session may not receive the expected NP Direct offload.

---

# 🧮 32. NP Direct Platforms

Examples:

```text
FortiGate 1000D
FortiGate 2000E
FortiGate 2500E
FortiGate 3700D
```

On the FortiGate 3700D, certain ports can operate in low-latency mode.

---

# ⚙️ 33. Enable NP Direct / Low-Latency Mode

```bash
config system np6
    edit np6_0
        set low-latency-mode enable
    end
end
```

Concept:

```text
Low-Latency Mode
       |
       v
Bypass ISF
       |
       v
Direct NP path
```

---

# 🆚 34. ISF vs NP Direct

| Attribute           | ISF                        | NP Direct           |
| ------------------- | -------------------------- | ------------------- |
| Latency             | Higher                     | Lowest possible     |
| Internal path       | Uses ISF                   | Direct              |
| Traffic flexibility | Any-to-any                 | Same NP required    |
| Cross-NP forwarding | Yes                        | No                  |
| LAG flexibility     | Can mix supported NP ports | Same NP requirement |
| General deployment  | ✅                          |                     |
| Ultra-low latency   |                            | ✅                   |
| HFT / HPC           |                            | ✅                   |

> Exact latency values are platform-dependent; don't treat a fixed microsecond figure as universal.

---

# 🔍 35. Verify NP Mapping First

```bash
diagnose npu np6 port-list
```

Check:

```text
Ingress Port
     |
     v
   NP-X
     |
     ?
Egress Port
     |
     v
   NP-X
```

For NP Direct:

```text
Ingress NP == Egress NP
```

should be verified.

---

# 🚫 36. Do Not Mix NP Mapping in LAG

Example:

```bash
config system interface
    edit agg1
        set type aggregate
        set member port1 port2
    next
end
```

For NP Direct designs:

```text
port1 → NP1
port2 → NP2

      ↓

Avoid mixing these in a single latency-sensitive
NP Direct LAG design.
```

Prefer LAG members mapped appropriately to the same NP where the architecture requires it.

---

# 🐢 37. Symptom: Traffic Not Offloading in NP Direct

### Possible Root Cause

```text
Ingress interface → NP1
Egress interface  → NP2
```

### Verify

```bash
diagnose npu np6 port-list | grep portx
```

Check NP assignment.

### Corrective Action

Redesign the path/policy/interface selection so that the required traffic enters and exits through compatible NP resources.

---

# ⏱️ 38. Symptom: High Latency Despite NP Direct

### Possible Cause

Low-latency mode is disabled.

```bash
config system np6
    edit np6_0
        set low-latency-mode enable
    end
end
```

Verify again:

```bash
diagnose npu np6 port-list
```

---

# 🔀 39. FortiGate 3700D Hybrid Architecture

A hybrid design can use both ISF and NP Direct modes.

Conceptually:

```text
Ports 1-24
    |
    v
  ISF Mode
    |
    v
Flexible NP offload


Ports 25-32
    |
    v
NP Direct
    |
    v
Low latency
```

Example:

```bash
config system np6
    edit np6_0
        set low-latency-mode disable
    next

    edit np6_1
        set low-latency-mode enable
    end
end
```

---

# 🧪 40. Always Verify With Hardware Diagnostics

```bash
diagnose npu np6 port-list
```

Then verify the actual session:

```bash
diagnose sys session list
```

Look for:

```text
npu_info
no_ofld_reason
```

---

# 📊 41. Monitor NP Utilization

```bash
diagnose npu np6 session-stats
```

Use this when:

```text
CPU utilization is high
        |
        v
Check whether NP is actually carrying traffic
```

---

# 🧾 42. Session Accounting

If logging/statistics are required, enable the appropriate per-session accounting features supported by the platform.

The design goal is:

```text
Acceleration
     +
Visibility
     =
Performance + Troubleshooting
```

---

# 🚨 43. Configuration to Avoid

## Strict Protocol Header Checking

Avoid enabling:

```text
check-protocol-header strict
```

when hardware acceleration is required, because strict protocol-header checking can force processing through the main CPU and disable acceleration.

It may inspect things such as:

```text
Layer 4 header
IP header
SPI
Checksum
Header length
```

---

# ⚠️ 44. Proxy vs Flow-Based Processing

Avoid unnecessarily mixing:

```text
Proxy-based inspection
+
Flow-based acceleration
```

in a way that breaks the expected hardware offload path.

### Preferred Acceleration Model

```text
Flow-based policy
       |
       v
CPU — initial session processing
       |
       v
NPU / CP
       |
       v
Fast Path
       |
       v
Egress
```

---

# 🧭 45. Hardware Acceleration Troubleshooting Flow

```text
                 Traffic Problem
                       |
                       v
              Check hardware
                       |
                       v
            get hardware status
                       |
                       v
               Identify NPU
                       |
                       v
       diagnose npu np6 port-list
                       |
                       v
             Check port mapping
                       |
          +------------+------------+
          |                         |
          v                         v
      ISF design               NP Direct
          |                         |
          v                         v
   Check cross-NP             Check same-NP
     forwarding                 requirement
          |                         |
          +------------+------------+
                       |
                       v
             Check session state
                       |
                       v
          diagnose sys session list
                       |
             +---------+---------+
             |                   |
             v                   v
          npu_info          no_ofld_reason
             |                   |
             v                   v
        Offloaded?          Find blocker
                                 |
                  +--------------+--------------+
                  |              |              |
                  v              v              v
              Proxy          Tunnel       auto-asic-offload
             profile        interface          disabled
                  |              |              |
                  +--------------+--------------+
                                 |
                                 v
                        Check policy design
                                 |
                                 v
                         Check NPU statistics
                                 |
                                 v
                    diagnose npu np6 session-stats
```

---

# 🧠 46. Hardware Acceleration Mental Model

```text
                         FORTIGATE
                            |
              +-------------+-------------+
              |                           |
             CPU                         ASIC
              |                           |
       Control / Policy             Hardware Fast Path
       Session Setup               Packet Forwarding
       Management                  Crypto / Security
              |                           |
              +-------------+-------------+
                            |
                           NPU
                            |
             +--------------+--------------+
             |              |              |
            NP6          NP6XLite         NP7
             |
             +-- Fast Path
             +-- IPsec
             +-- QoS
             +-- Multicast
             +-- LAG
             +-- Anomaly checks
```

---

# 📌 47. Quick Reference

| Task                             | Command / Setting                          |
| -------------------------------- | ------------------------------------------ |
| Hardware information             | `get hardware status`                      |
| NP6 port mapping                 | `diagnose npu np6 port-list`               |
| NP6XLite port mapping            | `get hardware npu np6xlite port-list`      |
| NP6Lite port mapping             | `diagnose npu np6lite port-list`           |
| Session offload status           | `diagnose sys session list`                |
| NP session statistics            | `diagnose npu np6 session-stats`           |
| IPsec offload stats              | `diagnose npu np6 ipsec-stats`             |
| HPE status                       | `diagnose npu np6 hpe`                     |
| Packet capture                   | `diag sniffer packet port1 'host 1.1.1.1'` |
| Purge session drift              | `diagnose npu np6 sse-purge-drift 0`       |
| Enable fast path                 | `set fastpath enable`                      |
| Enable IPsec ASIC offload        | `set ipsec-asic-offload enable`            |
| Enable CAPWAP offload            | `set capwap-offload enable`                |
| Enable interface shaping offload | `set intf-shaping-offload enable`          |
| NP Direct                        | `set low-latency-mode enable`              |

---
---

# 🧠 Golden Rules

```text
1. First packet → CPU
2. Eligible established traffic → NPU fast path
3. Proxy inspection → often CPU/proxy path
4. Tunnel traffic → special offload limitations
5. ISF → flexible cross-NP forwarding
6. NP Direct → lowest latency, stricter NP mapping
7. LAG + NP Direct → verify all members carefully
8. npu_info → confirm offload
9. no_ofld_reason → find why not offloaded
10. Port mapping → always verify before redesigning policies
11. HPE → protect CPU from high-rate traffic
12. NetFlow and sFlow have different acceleration impacts
```

> ⚠️ **Version/model note:** Hardware acceleration behavior, CLI availability, ASIC capabilities, supported offloads, and exact throughput vary by FortiGate model and FortiOS release. Treat the commands above as a troubleshooting/learning reference and verify them against the target platform before using them in production.

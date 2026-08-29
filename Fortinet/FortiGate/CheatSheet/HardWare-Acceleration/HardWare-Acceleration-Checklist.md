# 🔥 FortiGate Hardware Acceleration Checklist — NPU, NP6, NP7, ISF & NP Direct

> **SheynShield | Engineering Secure Networks**
>
> A practical FortiGate Hardware Acceleration troubleshooting and optimization checklist covering **SoC, SPU, NPU, CP, NTurbo, NP6, NP6Lite, NP6XLite, NP7, Fast Path, ISF, NP Direct, IPsec offload, HPE, QoS, LAG and session offloading**.

---

## 📋 Checklist Overview

Use this checklist when:

* [ ] FortiGate CPU utilization is unexpectedly high
* [ ] NPU offloading is not working
* [ ] Sessions remain CPU-processed
* [ ] IPsec performance is lower than expected
* [ ] Network latency is higher than expected
* [ ] NP Direct is not providing the expected performance
* [ ] Cross-NP forwarding needs to be validated
* [ ] Hardware acceleration needs to be verified before a production change
* [ ] You are troubleshooting `no_ofld_reason`
* [ ] You need to validate an NP6/NP7 architecture

---

# 🧩 1. Identify the FortiGate Hardware Architecture

## Hardware Identification

* [ ] Check the hardware platform

```bash
get hardware status
```

* [ ] Identify the ASIC/NPU generation
* [ ] Identify whether the platform contains NP6, NP6Lite, NP6XLite, NP7 or another supported NPU
* [ ] Identify whether the platform contains a CP/CP9/CP9XLite/CP9Lite processor
* [ ] Verify the exact FortiGate model
* [ ] Verify the exact FortiOS version

### Architecture Mental Model

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

### Processor Reference

| Processor  | Role              | Validation                          |
| ---------- | ----------------- | ----------------------------------- |
| `NP6`      | Network Processor | Hardware forwarding / acceleration  |
| `NP6XLite` | Network Processor | Enhanced/lower-cost NP architecture |
| `NP6Lite`  | Network Processor | Lightweight NP architecture         |
| `NP7`      | Network Processor | Newer high-throughput architecture  |
| `CP`       | Content Processor | Security/content acceleration       |
| `CP9`      | Content Processor | High-end content acceleration       |
| `CP9XLite` | Content Processor | Reduced-capability variant          |
| `CP9Lite`  | Content Processor | Entry/lower-capability variant      |

### CP / SoC Validation

* [ ] Verify whether the platform uses the expected CP/SoC combination
* [ ] Remember that exact capabilities are model-dependent

```text
CP9 / CP9XLite → SoC4
CP9Lite        → SoC3
```

### IPsec Engine Reference

| Processor | IPsec Engines |
| --------- | ------------: |
| CP9       |            16 |
| CP9XLite  |             5 |
| CP9Lite   |             1 |

---

# ⚡ 2. Validate Fast Path / NTurbo

## Basic Fast-Path Concept

* [ ] Understand that the CPU may handle initial session processing
* [ ] Verify whether subsequent packets are eligible for hardware acceleration
* [ ] Verify whether the session is being transferred to the NPU/ASIC fast path

```text
First Packet
     |
     v
   CPU
     |
     +-- Policy lookup
     +-- Session creation
     +-- Security decision
     |
     v
 Fast-Path Check
     |
 +---+---+
 |       |
Yes      No
 |       |
 v       v
NPU     CPU
 |
 v
Subsequent Packets
```

---

# ⚙️ 3. Validate NP Acceleration Configuration

* [ ] Check whether NP acceleration is configured

```bash
config ips global
    set np-accel-mode basic
end
```

* [ ] Verify that `basic` mode is supported by the target platform
* [ ] Verify that the CLI option exists on the target FortiOS release

### Basic Mode

```text
Basic NP Acceleration
        |
        +-- Direct data path
        +-- ASIC forwarding
        +-- Supported IPS participation
```

---

# 🧠 4. Validate CP Acceleration

* [ ] Check whether advanced CP acceleration is available
* [ ] Enable only when supported and required

```bash
config ips global
    set cp-accel-mode advanced
end
```

* [ ] Verify CP architecture
* [ ] Verify FortiOS support
* [ ] Verify model-specific limitations

> ⚠️ If the CLI command does not exist, do not assume the configuration is simply disabled. The capability may not exist on that platform or FortiOS build.

---

# 🚫 5. Check Why Traffic Is NOT Being Offloaded

## Firewall Policy

* [ ] Inspect the firewall policy
* [ ] Check whether ASIC offloading has been disabled

```bash
set auto-asic-offload disable
```

* [ ] If disabled, determine whether this was intentional
* [ ] Re-enable where appropriate

```bash
set auto-asic-offload enable
```

---

## Proxy-Based Inspection

* [ ] Check whether the policy uses proxy-based inspection
* [ ] Determine whether the selected security profile requires CPU/proxy processing
* [ ] Verify whether flow-based inspection is appropriate

```text
Proxy-Based Inspection
        |
        v
Proxy / CPU Processing
        |
        X
Normal NP Fast Path
```

---

## Session Helpers

* [ ] Check whether the protocol uses a FortiOS session helper
* [ ] Pay special attention to FTP

```text
FTP
 |
 +-- Session Helper
       |
       v
     CPU
```

---

## Interface Policies

* [ ] Check for interface policies
* [ ] Verify whether they affect the ingress/egress processing path
* [ ] Validate acceleration after any interface-policy change

---

## DoS Policies

* [ ] Check whether a DoS policy is applied
* [ ] Determine whether ingress/egress processing is affected
* [ ] Re-test the session after policy changes

---

# 🔐 6. Check Tunnel-Based Traffic

Identify whether the affected traffic uses:

* [ ] IPsec
* [ ] IP-in-IP
* [ ] SSL VPN
* [ ] GRE
* [ ] CAPWAP
* [ ] Other tunnel interfaces

```text
Tunnel Traffic
      |
      v
Tunnel Processing
      |
      v
Security Processing
      |
      v
Potential Hardware Offload
```

* [ ] Do not assume tunnel traffic has the same offload behavior as ordinary IPv4/IPv6 forwarding
* [ ] Verify platform-specific tunnel acceleration support

---

# 🔌 7. Verify NP Port Mapping

## NP6

```bash
get hardware npu np6 port-list
```

```bash
diagnose npu np6 port-list
```

* [ ] Identify ingress interface
* [ ] Identify egress interface
* [ ] Identify NP assignment
* [ ] Determine whether traffic crosses multiple NP processors

---

## NP6XLite

```bash
get hardware npu np6xlite port-list
```

* [ ] Verify interface-to-NP mapping

---

## NP6Lite

```bash
diagnose npu np6lite port-list
```

* [ ] Verify interface-to-NP mapping

---

# 🧩 8. Validate NP6 / NP7 VLAN MAC Behavior

On supported NP6/NP7 and virtual-clustering designs:

* [ ] Check whether VLAN MAC differs from the physical interface MAC
* [ ] Verify ARP behavior
* [ ] Verify the destination MAC of return traffic
* [ ] Confirm actual NPU behavior before disabling offload

```text
VLAN MAC
   ≠
Physical Interface MAC
```

### Potential Workaround

If hardware forwarding is genuinely affected:

```bash
config firewall policy
    edit <policy-id>
        set auto-asic-offload disable
    next
end
```

### Do NOT Apply Blindly

* [ ] Verify the actual failure first
* [ ] Check ARP learning
* [ ] Check return-path MAC behavior
* [ ] Verify NPU session state

> **Rule:** A VLAN/physical-MAC difference does not automatically mean that NPU offloading is broken.

---

# 📡 9. Check NetFlow / sFlow Impact

## NetFlow

* [ ] Check whether NetFlow is configured
* [ ] Verify that supported NP offloading remains operational

```text
NetFlow
   |
   +-- Session information
   |
   +-- NP offload may continue
```

## sFlow

* [ ] Check whether sFlow is enabled on the affected interface
* [ ] Determine whether it is affecting NP offloading

```text
sFlow
   |
   v
NP Offloading
   |
   v
Potentially Disabled / Affected
```

> ⚠️ **Important:** NetFlow and sFlow do not necessarily have the same impact on NP offloading.

---

# 🚀 10. Validate NP Generation

## NP6

* [ ] Identify NP6 platforms
* [ ] Verify model-specific throughput
* [ ] Verify supported cryptographic acceleration

## NP6XLite

* [ ] Verify AES-GCM support where applicable
* [ ] Verify platform-specific capabilities
* [ ] Verify throughput against the exact model

## NP6Lite

* [ ] Verify lightweight NPU capabilities
* [ ] Expect greater platform-specific limitations

## NP7

* [ ] Identify NP7-based platforms
* [ ] Verify supported acceleration features
* [ ] Verify exact platform datasheet

### Example NP7 Platforms

* [ ] FortiGate 1800F
* [ ] FortiGate 1801F
* [ ] FortiGate 2600F
* [ ] FortiGate 2601F
* [ ] FortiGate 3500F
* [ ] FortiGate 3501F
* [ ] FortiGate 4200F
* [ ] FortiGate 4201F
* [ ] FortiGate 4400F
* [ ] FortiGate 4401F

> ⚠️ Do not use generic throughput numbers as a replacement for the exact FortiGate model datasheet.

---

# 🔐 11. Validate IPsec Hardware Acceleration

* [ ] Verify whether IPsec ASIC offloading is enabled

```bash
config system npu
    set ipsec-asic-offload enable
end
```

* [ ] Verify supported cipher
* [ ] Verify platform-specific IPsec acceleration
* [ ] Verify inbound/outbound acceleration
* [ ] Check IPsec NPU statistics

```bash
diagnose npu np6 ipsec-stats
```

---

# 🔄 12. Troubleshoot IPsec Packet Reordering / Anti-Replay

### Symptoms

* [ ] IPsec packet drops
* [ ] Anti-replay errors
* [ ] Packet reordering
* [ ] Unexpected tunnel instability
* [ ] High packet loss under load

### Troubleshooting Action

```bash
config system npu
    set ipsec-inbound-cache disable
end
```

* [ ] Re-test traffic
* [ ] Monitor anti-replay behavior
* [ ] Compare packet ordering
* [ ] Re-enable the feature if the troubleshooting test is complete and it is not required to remain disabled

> ⚠️ Do not disable security-related acceleration/caching as a generic optimization.

---

# 🌐 13. Validate ESP-over-UDP / NAT-T Offload

* [ ] Determine whether ESP-over-UDP/NAT-T is being used
* [ ] Check whether UESP offload is supported
* [ ] Verify configuration

```bash
config system npu
    set uesp-offload enable
end
```

* [ ] Validate the actual session
* [ ] Validate IPsec NPU statistics

---

# 🏎️ 14. Validate Fast Path Configuration

```bash
config system npu
    set fastpath enable
end
```

* [ ] Verify `fastpath`
* [ ] Confirm that the platform supports the configuration
* [ ] Re-test established sessions
* [ ] Verify NPU session information

---

# 🖥️ 15. Check Dedicated Management CPU

* [ ] Determine whether management CPU isolation is required
* [ ] Verify platform support
* [ ] Check configuration

```bash
config system npu
    set dedicated-management-cpu enable
end
```

Concept:

```text
CPU
 |
 +-- Management workload
 |
 +-- Reserved CPU resources
```

---

# 📡 16. Validate CAPWAP Offload

* [ ] Determine whether CAPWAP traffic is present
* [ ] Verify platform support
* [ ] Verify configuration

```bash
config system npu
    set capwap-offload enable
end
```

* [ ] Validate actual CAPWAP session behavior

---

# 🎯 17. Validate Hardware QoS / Traffic Shaping

```bash
config system npu
    set intf-shaping-offload enable
    set qos-mode priority
end
```

* [ ] Verify interface shaping requirements
* [ ] Verify QoS mode
* [ ] Confirm NPU support
* [ ] Check whether shaping is actually performed in hardware
* [ ] Verify statistics/logging behavior

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

# 🔎 18. Verify Individual Session Offloading

Run:

```bash
diagnose sys session list
```

* [ ] Locate the affected session
* [ ] Check `npu_info`
* [ ] Check `no_ofld_reason`
* [ ] Determine whether the session is offloaded
* [ ] Determine whether the session is partially offloaded
* [ ] Identify the reason for CPU processing

### Interpretation

```text
npu_info
   ↓
Hardware offload information
```

```text
no_ofld_reason
   ↓
Reason for missing offload
```

---

# 🧪 19. Capture the Actual Traffic

Use packet capture to correlate packet behavior:

```bash
diag sniffer packet port1 'host 1.1.1.1'
```

* [ ] Verify packet arrival
* [ ] Verify packet departure
* [ ] Verify source/destination
* [ ] Verify ingress interface
* [ ] Verify egress interface
* [ ] Compare packet capture with session information
* [ ] Determine whether the problem is forwarding, policy, session or hardware acceleration

---

# 📊 20. Check NPU Session Statistics

```bash
diagnose npu np6 session-stats 0
```

* [ ] Check session counters
* [ ] Check hardware forwarding activity
* [ ] Compare NPU activity with CPU utilization
* [ ] Determine whether traffic is actually using the NPU

---

# 🔐 21. Check IPsec NPU Statistics

```bash
diagnose npu np6 ipsec-stats
```

* [ ] Verify encryption statistics
* [ ] Verify decryption statistics
* [ ] Verify hardware acceleration counters
* [ ] Compare counters before/after traffic generation

---

# 🧟 22. Troubleshoot Session Drift

### Symptoms

* [ ] CPU session state does not match NPU session state
* [ ] Stale sessions appear to exist
* [ ] Unexpected forwarding behavior occurs
* [ ] Hardware session state appears orphaned

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

### Purge

```bash
diagnose npu np6 sse-purge-drift 0
```

* [ ] Confirm the affected NP
* [ ] Perform the purge deliberately
* [ ] Re-test the affected sessions

---

# 🚨 23. Troubleshoot TCP Session Drops Under Heavy Load

### Symptoms

* [ ] TCP sessions fail during high traffic
* [ ] TCP connection establishment becomes unstable
* [ ] Session search behavior becomes problematic
* [ ] CPU/NPU interaction is suspected

Potential configuration:

```bash
config firewall policy
    edit <policy-id>
        set delay-tcp-npu-session enable
    next
end
```

* [ ] Apply only to the affected policy
* [ ] Monitor session behavior
* [ ] Validate whether the change improves stability
* [ ] Compare CPU/NPU behavior before and after the change

---

# 🧠 24. Validate NP6 Offloaded Features

Check whether the deployment requires:

* [ ] IPsec offload
* [ ] Traffic shaping
* [ ] Anomaly checks
* [ ] Multicast
* [ ] Multicast over IPsec
* [ ] LAG
* [ ] Inter-VDOM offloading
* [ ] Hardware packet forwarding

### Anomaly Processing

```text
Malformed Packet
       |
       v
      NPU
       |
       X
Drop before CPU
```

* [ ] Verify whether malformed traffic is being handled by hardware

---

# 🔀 25. Validate LAG Architecture

* [ ] Identify all LAG members
* [ ] Identify the NP associated with each member
* [ ] Determine whether traffic crosses multiple NP processors
* [ ] Validate the architecture against the desired forwarding model

Example:

```text
LAG
 |
 +-- NP1
 +-- NP2
 +-- NP3
 +-- NP4
```

* [ ] Verify whether LAG distribution is expected
* [ ] Verify session distribution
* [ ] Verify platform-specific limitations

---

# 🔗 26. Validate ISF Architecture

**ISF = Integrated Switch Fabric**

* [ ] Determine whether the platform uses ISF
* [ ] Identify all NP processors
* [ ] Check whether cross-NP forwarding is required
* [ ] Verify cross-chip offloading

Concept:

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

### ISF Provides

* [ ] Any-to-any NP forwarding
* [ ] Cross-NP forwarding
* [ ] Multiple NP connectivity
* [ ] Load distribution
* [ ] Flexible ingress/egress mapping

### Example ISF Platforms

* [ ] FortiGate 1500D
* [ ] FortiGate 3200D
* [ ] FortiGate 5001D

### Verify

```bash
get hardware npu np6 port-list
```

Look for:

```text
Cross-chip offloading: yes
```

---

# ⚡ 27. Validate NP Direct Architecture

NP Direct provides a more direct forwarding path without requiring ISF traversal.

```text
Port
 |
 v
NP
 |
 v
Port
```

* [ ] Determine whether the platform supports NP Direct
* [ ] Identify the NP associated with ingress
* [ ] Identify the NP associated with egress
* [ ] Confirm that the forwarding path satisfies same-NP requirements

### Example Platforms

* [ ] FortiGate 1000D
* [ ] FortiGate 2000E
* [ ] FortiGate 2500E
* [ ] FortiGate 3700D

---

# ⏱️ 28. Validate NP Direct / Low-Latency Mode

```bash
config system np6
    edit np6_0
        set low-latency-mode enable
    end
end
```

* [ ] Verify platform support
* [ ] Verify the NP assignment
* [ ] Verify ingress NP
* [ ] Verify egress NP
* [ ] Confirm that both belong to the required NP path

### Required Concept

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
 NP1
```

For NP Direct:

```text
Ingress NP == Egress NP
```

should be validated where the architecture requires it.

---

# 🚫 29. Check NP Direct LAG Design

Example:

```text
port1 → NP1
port2 → NP2
```

* [ ] Determine whether LAG members belong to different NP processors
* [ ] Avoid assuming mixed-NP LAG behavior is equivalent to NP Direct
* [ ] Verify the architecture before deployment
* [ ] Prefer compatible NP mapping for latency-sensitive NP Direct designs

Example:

```bash
config system interface
    edit agg1
        set type aggregate
        set member port1 port2
    next
end
```

* [ ] Validate NP assignment of every member

---

# 🐢 30. Troubleshoot NP Direct Offloading

### Symptom

```text
Ingress → NP1
Egress  → NP2
```

### Check

```bash
diagnose npu np6 port-list
```

* [ ] Identify ingress NP
* [ ] Identify egress NP
* [ ] Check whether they match
* [ ] Check whether low-latency mode is enabled
* [ ] Check policy/interface design
* [ ] Check actual session offloading

### Corrective Action

* [ ] Redesign interface selection if necessary
* [ ] Re-evaluate LAG membership
* [ ] Align traffic with the required NP architecture
* [ ] Re-test session offloading

---

# ⏱️ 31. Troubleshoot High Latency

If NP Direct is expected:

* [ ] Verify `low-latency-mode`
* [ ] Verify NP mapping
* [ ] Verify LAG membership
* [ ] Verify ingress/egress interfaces
* [ ] Check whether traffic traverses ISF
* [ ] Check whether the session is actually hardware accelerated

```bash
diagnose npu np6 port-list
```

---

# 🔀 32. Validate Hybrid ISF / NP Direct Architecture

Some platforms can use different forwarding architectures for different NP resources.

Concept:

```text
Ports 1-24
    |
    v
  ISF Mode
    |
    v
Flexible NP Offload


Ports 25-32
    |
    v
NP Direct
    |
    v
Low Latency
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

* [ ] Identify which NP is used for flexible forwarding
* [ ] Identify which NP is used for low-latency traffic
* [ ] Validate port mapping
* [ ] Validate application requirements

---

# 🛡️ 33. Validate Host Protection Engine — HPE

HPE can protect CPU resources from high-rate traffic such as SYN floods and UDP floods.

```text
Internet
   |
   | High-rate traffic
   v
+-------+
|  HPE  |
+-------+
   |
   +-- Rate limiting
   +-- CPU protection
   |
   v
FortiGate
```

### Configuration

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

* [ ] Identify the correct NP
* [ ] Validate `tcpsyn-max`
* [ ] Validate `udp-max`
* [ ] Validate HPE shaper
* [ ] Verify that thresholds match the environment
* [ ] Avoid blindly copying example values into production

### Monitor

```bash
diagnose npu np6 hpe
```

---

# 📡 34. Validate Multicast Optimization

```bash
config system npu
    set mcast-session-accounting session-based
    set lag-out-port-select enable
end
```

* [ ] Determine whether multicast session accounting is required
* [ ] Check multicast session scale
* [ ] Validate LAG egress behavior
* [ ] Confirm platform support

---

# 🔄 35. Validate Multicast Over IPsec

* [ ] Determine whether multicast is transported over IPsec
* [ ] Verify platform support
* [ ] Verify IPsec ASIC acceleration
* [ ] Check NPU/IPsec statistics
* [ ] Validate actual forwarding behavior

---

# 🧱 36. Check Strict Protocol Header Inspection

Avoid enabling strict protocol header checking when it conflicts with the required hardware acceleration path.

Potential configuration:

```text
check-protocol-header strict
```

* [ ] Determine whether strict inspection is enabled
* [ ] Understand its impact on CPU processing
* [ ] Verify whether acceleration is required
* [ ] Compare security requirements against performance requirements

Potential checks include:

* [ ] Layer 4 header
* [ ] IP header
* [ ] SPI
* [ ] Checksum
* [ ] Header length

---

# 🔬 37. Validate Flow-Based vs Proxy-Based Inspection

### Preferred Hardware Acceleration Model

```text
Flow-Based Policy
       |
       v
CPU — Initial Session Processing
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

* [ ] Identify inspection mode
* [ ] Check whether proxy-based processing is required
* [ ] Check whether flow-based processing is sufficient
* [ ] Verify security-policy requirements
* [ ] Confirm the resulting offload behavior

---

# 🧪 38. Complete Hardware Acceleration Troubleshooting Workflow

Use this sequence before making configuration changes.

### Phase 1 — Hardware

* [ ] Run `get hardware status`
* [ ] Identify FortiGate model
* [ ] Identify NPU/ASIC generation
* [ ] Identify CP architecture
* [ ] Confirm FortiOS version

### Phase 2 — Port Mapping

* [ ] Run NPU port-list command
* [ ] Identify ingress NP
* [ ] Identify egress NP
* [ ] Determine whether ISF is involved
* [ ] Determine whether NP Direct is involved

### Phase 3 — Session

* [ ] Run `diagnose sys session list`
* [ ] Find the affected session
* [ ] Check `npu_info`
* [ ] Check `no_ofld_reason`

### Phase 4 — Policy

* [ ] Check `auto-asic-offload`
* [ ] Check inspection mode
* [ ] Check security profiles
* [ ] Check session helpers
* [ ] Check interface policies
* [ ] Check DoS policies

### Phase 5 — Traffic Type

* [ ] Check whether traffic is tunneled
* [ ] Check IPsec
* [ ] Check GRE
* [ ] Check SSL VPN
* [ ] Check CAPWAP
* [ ] Check multicast
* [ ] Check special protocols

### Phase 6 — NPU Statistics

* [ ] Check NPU session statistics
* [ ] Check IPsec statistics
* [ ] Check HPE statistics
* [ ] Compare hardware counters with CPU utilization

### Phase 7 — Packet Validation

* [ ] Run packet capture
* [ ] Verify ingress
* [ ] Verify egress
* [ ] Correlate packet behavior with session state
* [ ] Confirm the actual forwarding path

### Phase 8 — Change / Re-Test

* [ ] Make only one significant change at a time
* [ ] Re-test the same traffic
* [ ] Compare `npu_info`
* [ ] Compare `no_ofld_reason`
* [ ] Compare NPU statistics
* [ ] Compare CPU utilization
* [ ] Document the result

---

# 🚨 39. Hardware Acceleration Red Flags

If you see any of the following, stop and investigate before optimizing:

* [ ] `auto-asic-offload disable`
* [ ] Proxy-based inspection
* [ ] `no_ofld_reason`
* [ ] Unexpected tunnel processing
* [ ] Mixed-NP LAG
* [ ] NP Direct with different ingress/egress NPs
* [ ] Unexpected VLAN MAC behavior
* [ ] sFlow on an affected interface
* [ ] Strict protocol-header checking
* [ ] High CPU with low NPU utilization
* [ ] IPsec anti-replay drops
* [ ] Session drift
* [ ] HPE thresholds that do not match traffic patterns

---

# 📊 40. Final Validation Checklist

Before declaring the hardware acceleration problem solved:

* [ ] Hardware architecture identified
* [ ] FortiOS version verified
* [ ] NPU generation identified
* [ ] Port-to-NP mapping verified
* [ ] ISF/NP Direct architecture identified
* [ ] Firewall policy reviewed
* [ ] `auto-asic-offload` verified
* [ ] Inspection mode verified
* [ ] Tunnel requirements verified
* [ ] `diagnose sys session list` checked
* [ ] `npu_info` verified
* [ ] `no_ofld_reason` investigated
* [ ] NPU statistics checked
* [ ] IPsec statistics checked where applicable
* [ ] Packet capture performed where required
* [ ] HPE checked where applicable
* [ ] LAG/NP mapping validated
* [ ] Actual CPU utilization compared
* [ ] Actual throughput compared
* [ ] Latency measured where applicable
* [ ] Configuration change documented
* [ ] Before/after behavior compared

---

# 🧠 Golden Rules

> ### 01 — First Packet
>
> **CPU → Policy / Session Decision → Hardware Fast Path**

* [ ] Do not expect every packet to be processed identically.

### 02 — Verify, Don't Assume

* [ ] Use `npu_info`
* [ ] Use `no_ofld_reason`
* [ ] Use NPU diagnostics

### 03 — Port Mapping Matters

* [ ] Always understand ingress/egress NP assignment before redesigning the policy.

### 04 — ISF ≠ NP Direct

```text
ISF
→ Flexible cross-NP forwarding

NP Direct
→ Lower-latency direct path
→ Stricter NP mapping requirements
```

### 05 — Proxy Inspection Can Change the Path

* [ ] Check inspection mode before blaming the NPU.

### 06 — Tunnel Traffic Is Special

* [ ] Never assume tunnel traffic behaves like ordinary IPv4/IPv6 forwarding.

### 07 — LAG Requires Hardware Awareness

* [ ] Validate every LAG member's NP assignment.

### 08 — NetFlow ≠ sFlow

* [ ] Their impact on NP offloading can be different.

### 09 — HPE Protects CPU Resources

* [ ] Validate HPE when dealing with very high packet rates.

### 10 — Performance ≠ Blind Optimization

* [ ] Do not disable security features merely to increase hardware offloading.

---

# ⚡ Quick Command Reference

| Objective                 | Command                                    |
| ------------------------- | ------------------------------------------ |
| Hardware identification   | `get hardware status`                      |
| NP6 port mapping          | `diagnose npu np6 port-list`               |
| NP6XLite mapping          | `get hardware npu np6xlite port-list`      |
| NP6Lite mapping           | `diagnose npu np6lite port-list`           |
| Session offload           | `diagnose sys session list`                |
| NPU session statistics    | `diagnose npu np6 session-stats`           |
| IPsec statistics          | `diagnose npu np6 ipsec-stats`             |
| HPE monitoring            | `diagnose npu np6 hpe`                     |
| Packet capture            | `diag sniffer packet port1 'host 1.1.1.1'` |
| Session drift purge       | `diagnose npu np6 sse-purge-drift 0`       |
| Fast Path                 | `set fastpath enable`                      |
| IPsec ASIC offload        | `set ipsec-asic-offload enable`            |
| CAPWAP offload            | `set capwap-offload enable`                |
| Interface shaping offload | `set intf-shaping-offload enable`          |
| NP Direct / low latency   | `set low-latency-mode enable`              |

---

# 🏆 FortiGate Hardware Acceleration — Expert Validation Model

```text
                  TRAFFIC PROBLEM
                        |
                        v
                +---------------+
                | Identify HW   |
                +-------+-------+
                        |
                        v
                get hardware status
                        |
                        v
                +---------------+
                | NPU Mapping   |
                +-------+-------+
                        |
                        v
              diagnose npu ... port-list
                        |
             +----------+----------+
             |                     |
             v                     v
           ISF                  NP Direct
             |                     |
             v                     v
      Cross-NP possible       Same-NP critical
             |                     |
             +----------+----------+
                        |
                        v
                Session Analysis
                        |
                        v
             diagnose sys session list
                        |
              +---------+---------+
              |                   |
              v                   v
           npu_info        no_ofld_reason
              |                   |
              v                   v
          Offloaded?          Find blocker
                                  |
                    +-------------+-------------+
                    |             |             |
                    v             v             v
                 Policy        Proxy        Tunnel
                    |             |             |
                    +-------------+-------------+
                                  |
                                  v
                         NPU Statistics
                                  |
                                  v
                       Performance Validation
                                  |
                                  v
                           FINAL RESULT
```

---

# ⚠️ Version & Platform Disclaimer

Hardware acceleration behavior is highly dependent on:

* [ ] FortiGate model
* [ ] NPU generation
* [ ] CP generation
* [ ] FortiOS release
* [ ] Interface architecture
* [ ] Security profile
* [ ] Traffic type
* [ ] Encryption algorithm
* [ ] Network topology

> **Always validate CLI availability, ASIC capabilities, supported offloads, throughput and limitations against the exact FortiGate model and FortiOS release before applying a configuration in production.**

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

## 🔎 Keywords

`FortiGate Hardware Acceleration` · `FortiGate NPU` · `FortiGate NP6` · `FortiGate NP7` · `NP6 Offloading` · `NPU Offload` · `FortiGate Fast Path` · `FortiGate NTurbo` · `FortiGate ISF` · `FortiGate NP Direct` · `FortiGate IPsec Offload` · `FortiGate HPE` · `FortiGate CPU High` · `FortiGate no_ofld_reason` · `FortiGate npu_info` · `FortiGate Troubleshooting` · `Fortinet Hardware Acceleration` · `FortiGate Performance Optimization`

---

**SheynShield | Engineering Secure Networks**

> **Learn the architecture. Verify the hardware path. Troubleshoot from evidence. Optimize only after validation.**

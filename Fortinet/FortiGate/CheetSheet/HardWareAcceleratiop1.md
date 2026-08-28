# FortiGate Hardware Acceleration — Cheatsheet Part 1

> **Scope:** FortiGate ASIC architecture, NPU/CP, NTurbo, session offloading, NP6/NP6 Lite/NP6 XLite, and common offloading limitations.

---

## 1. FortiGate Hardware Architecture

Simplified hardware hierarchy:

```text
CPU
└── SoC
    ├── SPU
    │   ├── NPU
    │   └── CP
    └── Other SoC components
```

### Key Terms

| Component | Role                                                            |
| --------- | --------------------------------------------------------------- |
| **CPU**   | General-purpose processing                                      |
| **SoC**   | System-on-Chip containing multiple processing components        |
| **SPU**   | Security Processing Unit                                        |
| **NPU**   | Network Processing Unit; accelerates network/session processing |
| **CP**    | Content Processor; accelerates content/security inspection      |

---

# 2. Content Processor — CP

Some entry-level FortiGate models do **not** include a Content Processor (CP).

### CP / SoC Generations

| Processor     | SoC Generation |
| ------------- | -------------- |
| **CP9**       | SoC4           |
| **CP9 XLite** | SoC4           |
| **CP9 Lite**  | SoC3           |

### IPsec Engine Count

| Processor     | IPsec Engines |
| ------------- | ------------: |
| **CP9**       |            16 |
| **CP9 XLite** |             5 |
| **CP9 Lite**  |             1 |

> **Remember:** The number of IPsec engines affects the hardware's ability to accelerate IPsec processing.

---

# 3. Identify ASIC / Hardware Version

Use:

```bash
get hardware status
```

This can be used to identify hardware/ASIC information, including the ASIC version available on the platform.

---

# 4. Network Processors — NPU

Common NPU generations:

```text
NP6
NP6 XLite
NP6 Lite
```

### Quick Comparison

| NPU           | Description                                |
| ------------- | ------------------------------------------ |
| **NP6**       | Full-featured NP6 network processor        |
| **NP6 XLite** | Reduced-feature / smaller platform variant |
| **NP6 Lite**  | Entry-level NP6 variant                    |

---

# 5. NTurbo / Fast Path

**NTurbo** is an acceleration mechanism that moves supported processing into ASICs instead of processing everything through the FortiGate CPU.

It can be thought of as a **fast path** for eligible traffic.

### Basic Concept

Without acceleration:

```text
Ingress
   ↓
CPU
   ↓
Security Processing
   ↓
CPU
   ↓
Egress
```

With ASIC acceleration:

```text
Ingress
   ↓
ASIC / NPU / CP
   ↓
Egress
```

The goal is to reduce CPU involvement and increase packet/session processing performance.

---

# 6. IPS Acceleration

Check/configure IPS acceleration:

```bash
config ips global
    set np-accel-mode basic
end
```

### NP Acceleration Modes

```text
basic
advanced
```

### Basic Mode

```bash
config ips global
    set np-accel-mode basic
end
```

**Basic** is the default mode.

Conceptually:

```text
Ingress
   ↓
NP acceleration
   ↓
IPS processing
   ↓
Egress
```

It provides a relatively direct accelerated data path while still applying supported IPS functionality.

---

## 7. CP Acceleration

Advanced CP acceleration can be configured with:

```bash
config ips global
    set cp-accel-mode advanced
end
```

### Advanced Mode

The advanced mode supports more sophisticated pattern-matching/inspection acceleration.

> Advanced CP acceleration requires platforms with sufficient CP capability, such as systems with multiple CP8/CP9 processors.

### Important

If the command is not available on the FortiGate:

```bash
set cp-accel-mode
```

it may indicate that the platform does not have the required CP/IPS acceleration capability.

> **Default:** `basic`

---

# 8. When NTurbo / NP Offloading Does NOT Work

Even when NTurbo is enabled, some sessions cannot be offloaded and must be processed by the FortiGate CPU.

---

## 8.1 NP Acceleration Disabled

If NP acceleration is disabled in the firewall policy:

```bash
config firewall policy
    edit <policy-id>
        set auto-asic-offload disable
    next
end
```

the session will not use ASIC offloading.

### Check

```text
auto-asic-offload = disable
        ↓
CPU processing
```

---

# 9. Proxy-Based Security Profiles

Sessions using **proxy-based security profiles** may not be eligible for NTurbo offloading.

Why?

Because proxy-based inspection can require additional FortiOS processing.

```text
Traffic
   ↓
Proxy-based inspection
   ↓
CPU / FortiOS processing
```

---

# 10. Session Helpers

Some protocols require FortiOS session helpers.

### Example: FTP

FTP sessions are not offloaded to NP processors when the FTP session helper is required.

```text
FTP
 ↓
FTP Session Helper
 ↓
CPU processing
```

> **Exam / troubleshooting point:** If a session depends on a FortiOS session helper, do not assume it can be fully ASIC-offloaded.

---

# 11. Interface Policies / DoS Policies

NTurbo offloading may not occur when **interface policies** or **DoS policies** are applied to the ingress/egress interfaces.

```text
Ingress
   ↓
Interface / DoS policy
   ↓
CPU processing
```

Always consider interface-level inspection when troubleshooting unexpected CPU utilization.

---

# 12. Tunnel Interfaces

Traffic to or from tunnel interfaces cannot be offloaded by NTurbo in the listed cases.

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
Client
  ↓
Tunnel
  ↓
FortiOS processing
  ↓
Security processing
  ↓
CPU
```

Therefore:

> **Tunnel traffic ≠ automatically ASIC-offloaded traffic.**

---

# 13. Virtual Clustering + VLAN MAC Address Issue

There is an important special case when using:

```text
NP7 / NP6
+
Virtual Clustering
+
VLAN interfaces
```

If the MAC address of the VLAN interface accepting traffic is different from the MAC address of the underlying physical interface, NP offloading may have problems in certain configurations.

### Problem

```text
Physical Interface
MAC-A
    │
    └── VLAN Interface
        MAC-B
```

If the relevant MAC-learning/ARP conditions are not satisfied, NP7/NP6 processing can potentially drop traffic.

---

## 13.1 Troubleshooting Solution

If traffic is being dropped, disable NP offloading on the firewall policy accepting the traffic:

```bash
config firewall policy
    edit <policy-id>
        set auto-asic-offload disable
    next
end
```

This forces the traffic away from NP offloading and can be used as a troubleshooting/workaround approach.

---

## 13.2 When Offloading Can Still Work

NP offloading can still work in some configurations where the VLAN and physical interface have different MAC addresses.

For example, if other network devices correctly learn the FortiGate MAC addresses through ARP:

```text
ARP learning
     ↓
Correct destination MAC
     ↓
NP offloading may continue
```

Offloading can also work when the destination MAC of the return traffic matches the MAC address of the underlying interface.

### Practical Rule

```text
Different VLAN/Physical MAC
        │
        ├── Correct ARP/MAC learning
        │       └── Offloading may work
        │
        └── Incorrect MAC learning
                └── NP drops / offload problem
```

---

# 14. Check NPU Port Information

## NP6

```bash
get hardware npu np6 port-list
```

or:

```bash
diagnose npu np6 port-list
```

---

## NP6 XLite

```bash
get hardware npu np6xlite port-list
```

---

## NP6 Lite

```bash
get hardware npu np6lite port-list
```

or:

```bash
diagnose npu np6lite port-list
```

---

# 15. NetFlow and NP6 Offloading

A useful point:

> Configuring **NetFlow** on interfaces connected to NP6, NP6 XLite, or NP6 Lite does **not** disable NP offloading.

```text
NP6
 │
 ├── NetFlow enabled
 │
 └── Other eligible sessions
        ↓
   NP offloading can continue
```

Full NetFlow information can be supported through information maintained in the firewall session.

---

# 16. sFlow vs NetFlow

This distinction is important.

### NetFlow

For NP6 / NP6 XLite / NP6 Lite:

```text
NetFlow
   ↓
NP offloading
   ↓
Can continue
```

Configuring NetFlow does not disable all NP offloading on the interface.

### sFlow

Configuring sFlow on an interface disables **NP6 / NP6 XLite / NP6 Lite offloading for all traffic on that interface**.

```text
sFlow enabled
      ↓
NP6 offloading disabled
      ↓
Traffic processed without NP6 offload
```

### Quick Comparison

| Feature     | Effect on NP6 Offloading                                       |
| ----------- | -------------------------------------------------------------- |
| **NetFlow** | ✅ Offloading can continue                                      |
| **sFlow**   | ❌ Disables NP6/NP6 XLite/NP6 Lite offloading on that interface |

---

# 17. NTurbo Troubleshooting Checklist

When you expect ASIC acceleration but CPU utilization is high:

```text
1. Check hardware / ASIC
       ↓
2. Identify NPU / CP
       ↓
3. Check firewall policy
       ↓
4. Check auto-asic-offload
       ↓
5. Check security profile mode
       ↓
6. Check session helpers
       ↓
7. Check interface policies
       ↓
8. Check DoS policies
       ↓
9. Check tunnel interfaces
       ↓
10. Check VLAN / physical MAC behavior
       ↓
11. Check NetFlow / sFlow
       ↓
12. Verify actual session/NPU offloading
```

---

# 18. Fast Memory Map

```text
                    FortiGate
                       │
                      SoC
                       │
                      SPU
                  ┌────┴────┐
                  │         │
                 NPU        CP
                  │         │
             NP6/XLite/   CP9/
                Lite      CP9 XLite
                           / Lite
```

### NTurbo

```text
NTurbo = Fast Path
         ↓
ASIC acceleration
         ↓
Reduce CPU processing
         ↓
Increase throughput / performance
```

### Common Offload Blockers

```text
auto-asic-offload disable
        │
proxy-based profile
        │
session helper
        │
interface policy
        │
DoS policy
        │
tunnel interface
        │
MAC / VLAN / virtual-cluster issue
        │
sFlow
        ↓
CPU / non-NP processing
```

---

# 19. Commands — Quick Reference

| Purpose              | Command                               |
| -------------------- | ------------------------------------- |
| Hardware information | `get hardware status`                 |
| NP6 ports            | `get hardware npu np6 port-list`      |
| NP6 diagnostic ports | `diagnose npu np6 port-list`          |
| NP6 XLite ports      | `get hardware npu np6xlite port-list` |
| NP6 Lite ports       | `get hardware npu np6lite port-list`  |
| NP6 Lite diagnostics | `diagnose npu np6lite port-list`      |

### IPS acceleration

```bash
config ips global
    set np-accel-mode basic
    set cp-accel-mode advanced
end
```

### Disable ASIC offload for troubleshooting

```bash
config firewall policy
    edit <policy-id>
        set auto-asic-offload disable
    next
end
```

---

# 20. Key Takeaways

> ### Remember these 8 points

1. **NPU = network acceleration**
2. **CP = content/security inspection acceleration**
3. **NTurbo = fast-path acceleration**
4. **`auto-asic-offload disable` prevents policy-level ASIC offloading**
5. **Session helpers can prevent NP offloading**
6. **Tunnel traffic is a major offloading limitation**
7. **NetFlow does not necessarily disable NP6 offloading**
8. **sFlow disables NP6/NP6 XLite/NP6 Lite offloading on the interface**

---

## ⚡ One-Line Troubleshooting Rule

```text
High CPU ≠ No ASIC

First determine:
Hardware → NPU/CP → Policy → Security Profile → Session Helper
→ Interface/DoS Policy → Tunnel → MAC/ARP → Flow Monitoring
```

# 🔗 SheynShield Resources

# FortiGate Advanced Networking & Policy Behavior Checklist

> **FortiOS Focus:** 7.2.x  
> **Topic:** MTU, TCP MSS, ASIC Offload, Session TTL, NGFW Modes, Land Attack Protection, Application Bandwidth Tracking, Learning Mode  
> **Level:** Advanced / NSE4–NSE7  
> **Brand:** SheynShield | Engineering Secure Networks

---

# 📌 Table of Contents

- [MTU vs TCP MSS](#1-mtu-vs-tcp-mss)
- [TCP MSS Clamping](#2-tcp-mss-clamping)
- [Interface MSS vs Policy MSS](#3-interface-mss-vs-policy-mss)
- [MTU Override](#4-mtu-override)
- [Path MTU Discovery](#5-path-mtu-discovery)
- [MTU/MSS Validation](#6-mtumss-validation)
- [ASIC Offload](#7-asic-offload)
- [Session TTL](#8-session-ttl)
- [NGFW Modes](#9-ngfw-modes)
- [Policy-Based vs Profile-Based](#10-policy-based-vs-profile-based)
- [Central NAT](#11-central-nat)
- [SSL Inspection & Redirect](#12-ssl-inspection--redirect)
- [Land Attack Protection](#13-land-attack-protection)
- [Application Bandwidth Tracking](#14-application-bandwidth-tracking)
- [Learning Mode](#15-learning-mode)
- [Advanced Troubleshooting](#16-advanced-troubleshooting)
- [NSE Memory Map](#17-nse-memory-map)

---

# 1. MTU vs TCP MSS Checklist

## ✅ MTU Concept

- [ ] Understand MTU definition

```text
MTU =
Maximum IP packet size transmitted
without fragmentation
````

* [ ] Remember MTU is an interface/path property

```text
MTU
 |
 └── Interface / Path
```

---

## ✅ MSS Concept

* [ ] Understand MSS definition

```text
MSS =
Maximum TCP payload size
```

* [ ] Remember MSS affects TCP negotiation only

```text
MSS
 |
 └── TCP Segment Size
```

---

## ✅ MSS Formula

IPv4 TCP:

```text
MSS = MTU - IP Header - TCP Header
```

Normally:

```text
MSS = MTU - 40
```

Example:

```text
MTU 1500

1500 - 40

MSS 1460
```

---

## ⚠️ Important Difference

* [ ] Do not confuse MSS with MTU

Example:

```bash
config firewall policy
    edit 1
        set tcp-mss-sender 1448
        set tcp-mss-receiver 1448
    next
end
```

This means:

```text
TCP MSS = 1448
```

NOT:

```text
MTU = 1488
```

---

# 2. TCP MSS Clamping Checklist

## When To Use

* [ ] IPsec tunnel problems
* [ ] GRE overhead issues
* [ ] PPPoE MTU reduction
* [ ] TCP application stalls
* [ ] PMTUD failures

---

## MSS Adjustment Flow

```text
Original MSS
      |
      v
FortiGate MSS Clamp
      |
      v
Reduced TCP Segment
```

---

## Example

```bash
config firewall policy
    edit 1
        set tcp-mss-sender 1448
        set tcp-mss-receiver 1448
    next
end
```

Validation:

* [ ] Capture TCP SYN
* [ ] Check advertised MSS
* [ ] Compare client/server MSS

---

# 3. Interface MSS vs Policy MSS Checklist

## Interface MSS

Use when:

* [ ] Applying broad MSS behavior
* [ ] All traffic requires adjustment

Concept:

```text
Interface
    |
    +-- MSS Adjustment
```

---

## Policy MSS

Use when:

* [ ] Only specific applications need tuning
* [ ] Specific VPN path requires adjustment

Example:

```bash
config firewall policy
    edit 1
        set tcp-mss-sender 1448
    next
end
```

---

## Design Rule

```text
Interface MSS
        |
        v
Global Behavior


Policy MSS
        |
        v
Granular Control
```

---

# 4. MTU Override Checklist

## Verify Before Change

* [ ] Identify problematic path
* [ ] Check encapsulation overhead
* [ ] Confirm required MTU

---

## Configure MTU Override

```bash
config system interface
    edit "port1"
        set mtu-override enable
        set mtu 1480
    next
end
```

---

Expected:

```text
Interface MTU

1500
 |
 v
1480
```

---

## Remember

```text
Changing MTU
=
Interface change


Changing MSS
=
TCP behavior change
```

---

# 5. Path MTU Discovery Checklist

## Verify PMTUD

* [ ] Check ICMP behavior
* [ ] Check tunnel overhead
* [ ] Check asymmetric routing
* [ ] Check firewall filtering

---

Enable:

```bash
config system interface
    edit "port1"
        set pmtu-discovery enable
    next
end
```

---

# 6. MTU/MSS Validation Checklist

## Reference Values

| Environment |     MTU |        MSS |
| ----------- | ------: | ---------: |
| Ethernet    |    1500 |       1460 |
| PPPoE       |    1492 |       1452 |
| Reduced MTU |    1480 |       1440 |
| IPsec       | Depends | ~1350-1400 |

---

## MPLS Validation

Check:

* [ ] Ethernet frame size
* [ ] IP MTU
* [ ] MPLS label overhead

Remember:

```text
MPLS MTU ≠ Always 1508
```

---

# 7. ASIC Offload Checklist

## Understand ASIC Path

Normal:

```text
Packet
 ↓
FortiGate
 ↓
ASIC
 ↓
Fast Forwarding
```

---

## Disable Temporarily

```bash
config firewall policy
    edit 1
        set auto-asic-offload disable
    next
end
```

---

Use during:

* [ ] Packet capture
* [ ] Flow debug
* [ ] Session troubleshooting
* [ ] Forwarding investigation

---

## Production Rule

```text
Disable ASIC

ONLY:
Specific Policy
+
Temporary Troubleshooting
```

---

# 8. Session TTL Checklist

## Understand Timeout

Session TTL controls:

```text
How long sessions remain active
```

---

Typical values:

```text
TCP
≈ 3600 sec

UDP
≈ 180 sec

ICMP
≈ 60 sec
```

---

# Global Session TTL

```bash
config system session-ttl
    set default 3600
end
```

---

# Policy TTL

```bash
config firewall policy
    edit 1
        set session-ttl 7200
    next
end
```

---

## TTL Troubleshooting

Checklist:

* [ ] Check session table
* [ ] Identify timeout value
* [ ] Confirm application requirement
* [ ] Clear old session if required
* [ ] Test new connection

Command:

```bash
diagnose sys session list
```

---

# 9. NGFW Modes Checklist

FortiGate supports:

```text
Policy-Based

Profile-Based
```

---

# 10. Policy-Based vs Profile-Based Checklist

## Profile-Based

Traditional model:

```text
Firewall Policy
 |
 +-- IPS
 +-- Antivirus
 +-- Web Filter
 +-- Application Control
 +-- SSL Inspection
```

---

## Policy-Based

Application/security focused:

```text
Traffic

↓

Application Identification

↓

Security Policy

↓

Decision
```

---

## Validate Before Changing

* [ ] Backup configuration
* [ ] Understand policy impact
* [ ] Test in lab

---

# 11. Central NAT Checklist

Central NAT separates:

```text
Security Policy

from

NAT Decision
```

Flow:

```text
Firewall Policy

↓

Central NAT

↓

Translation
```

---

Verify:

* [ ] NAT rules
* [ ] Matching order
* [ ] Source translation
* [ ] Destination translation

---

# 12. SSL Inspection & Redirect Checklist

Verify:

* [ ] Proxy inspection requirement
* [ ] SSL inspection profile
* [ ] Certificate trust
* [ ] Authentication integration

---

HTTP redirect:

```bash
set http-policy-redirect enable
```

SSH redirect:

```bash
set ssh-policy-redirect enable
```

---

Flow:

```text
Traffic

↓

Proxy

↓

Proxy Policy

↓

Security Inspection
```

---

# 13. Land Attack Protection Checklist

## Land Attack Definition

Condition:

```text
Source IP
=
Destination IP
```

Example:

```text
192.168.1.10

↓

192.168.1.10
```

---

Enable:

```bash
config system settings
    set block-land-attack enable
end
```

---

Verify:

* [ ] Protection enabled
* [ ] Logs monitored
* [ ] Attack signatures understood

---

# 14. Application Bandwidth Tracking Checklist

Enable:

```bash
config system settings
    set application-bandwidth-tracking enable
end
```

---

Useful for:

* [ ] Application visibility
* [ ] SD-WAN decisions
* [ ] QoS analysis
* [ ] Bandwidth reporting

---

Check:

* [ ] Device performance impact
* [ ] CPU utilization
* [ ] Session scale

---

# 15. Learning Mode Checklist

## Learning Workflow

```text
Traffic

↓

Logs

↓

FortiAnalyzer

↓

Policy Analyzer

↓

Recommendations

↓

Deployment
```

---

## Validate

* [ ] Traffic visibility
* [ ] FortiAnalyzer integration
* [ ] Policy recommendation accuracy
* [ ] Security event analysis

---

## Common Limitations

Check:

* [ ] Interface requirements
* [ ] NAT64/NAT46 compatibility
* [ ] User/group limitations
* [ ] Internet Service limitations

---

# 16. Advanced Troubleshooting Checklist

## MTU/MSS Problem

```text
Application Issue

↓

TCP Handshake

↓

Check MSS

↓

Check MTU

↓

Check PMTUD

↓

Check Tunnel Overhead

↓

Apply MSS Clamp
```

---

## ASIC Troubleshooting

```text
Identify Policy

↓

Check Session

↓

Check ASIC

↓

Disable Temporarily

↓

Capture Traffic

↓

Restore
```

---

## Session Troubleshooting

```bash
diagnose sys session list
```

Check:

* [ ] Source
* [ ] Destination
* [ ] Port
* [ ] Timeout
* [ ] Policy ID
* [ ] State

---

# 17. NSE High-Value Memory Map 🧠

## MTU

```text
Maximum IP Packet Size
```

---

## MSS

```text
Maximum TCP Payload
```

---

## MSS Formula

```text
MSS = MTU - 40
```

---

## ASIC

```text
Performance

↓

Hardware Acceleration
```

Troubleshooting:

```text
Temporary Disable
```

---

## Session TTL

```text
Global

↓

Policy Override

↓

Service Override
```

---

## NGFW

```text
Policy-Based

Application Centric


Profile-Based

Policy + Security Profiles
```

---

## Land Attack

```text
Source == Destination
```

---

## Learning Mode

```text
Observe

↓

Analyze

↓

Recommend
```

---

# 🔥 Final SheynShield Takeaway

```text
MTU
→ Packet Size

MSS
→ TCP Payload Size

MSS Clamp
→ Fix TCP Size Problems

ASIC
→ Performance Optimization

Session TTL
→ Connection Lifetime Control

NGFW Mode
→ Security Policy Architecture

Land Attack
→ Source/Destination Validation

Learning Mode
→ Traffic Intelligence → Policy Recommendation
```

---

# ⚠️ Production Checklist

* [ ] Backup configuration before major changes
* [ ] Avoid global changes without validation
* [ ] Test MTU/MSS before deployment
* [ ] Disable ASIC only temporarily
* [ ] Tune TTL based on application behavior
* [ ] Validate NGFW mode migration impact
* [ ] Monitor performance after enabling tracking features
* [ ] Restore troubleshooting settings after testing

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

**SheynShield | Engineering Secure Networks**


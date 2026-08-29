# 🔗 SheynShield Resources

# FortiGate VNE — MAP-E, DS-Lite & IPv4/IPv6 Transition Checklist

> **FortiOS:** 7.2.x  
> **Topic:** IPv4/IPv6 Transition, CGNAT, MAP-E, DS-Lite, VNE  
> **Level:** NSE 6/7 — Advanced Networking & Security Engineering  
> **Brand:** SheynShield | Engineering Secure Networks

---

# 📌 Table of Contents

- [1. VNE Fundamentals](#1-vne-fundamentals)
- [2. IPv4/IPv6 Transition Architecture](#2-ipv4ipv6-transition-architecture)
- [3. CGNAT Checklist](#3-cgnat-checklist)
- [4. MAP-E Checklist](#4-map-e-checklist)
- [5. DS-Lite Checklist](#5-ds-lite-checklist)
- [6. FortiGate VNE Configuration](#6-fortigate-vne-configuration)
- [7. Routing Validation](#7-routing-validation)
- [8. Firewall Policy Validation](#8-firewall-policy-validation)
- [9. Port Block Allocation](#9-port-block-allocation)
- [10. Troubleshooting Workflow](#10-troubleshooting-workflow)
- [11. Common Design Mistakes](#11-common-design-mistakes)
- [12. NSE Exam Notes](#12-nse-exam-notes)
- [13. Quick Command Reference](#13-quick-command-reference)

---

# 1. VNE Fundamentals Checklist

## What is VNE?

**VNE (Virtual Network Enabler/Encapsulator)** provides IPv4/IPv6 transition mechanisms on FortiGate.

Use cases:

- [ ] IPv6-only provider networks
- [ ] IPv4 service delivery over IPv6 transport
- [ ] Carrier-grade NAT architectures
- [ ] Broadband subscriber networks
- [ ] Mobile network transition scenarios

---

## VNE Supported Modes

| Mode | Purpose |
|---|---|
| `ds-lite` | IPv4 over IPv6 toward AFTR |
| `fixed-ip` | Provider-defined IPv4 over IPv6 mapping |
| `map-e` | Mapping of IPv4/ports with IPv6 encapsulation |

Checklist:

- [ ] Understand DS-Lite architecture
- [ ] Understand MAP-E architecture
- [ ] Understand IPv6 transport dependency
- [ ] Understand CGNAT relationship

---

# 2. IPv4/IPv6 Transition Architecture Checklist

## Core Mental Model

```text
IPv4 Service
      |
      v
IPv6 Transport Network
      |
      v
Transition Mechanism
      |
      v
IPv4 Internet
````

Verify:

* [ ] IPv4 addressing plan exists
* [ ] IPv6 transport is reachable
* [ ] Transition mechanism is selected
* [ ] Routing supports both protocols
* [ ] Security policies allow required traffic

---

# 3. CGNAT Checklist

## CGNAT Purpose

CGNAT allows multiple subscribers to share limited public IPv4 addresses.

Architecture:

```text
Subscriber
192.168.1.10
      |
      v
     CE
      |
      v
100.64.x.x
      |
      v
   CGNAT
      |
      v
Public IPv4
      |
      v
Internet
```

---

## Shared Address Space

Validate:

* [ ] Understand `100.64.0.0/10`
* [ ] Do not confuse with RFC1918 space

RFC1918:

```text
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
```

Carrier Shared Space:

```text
100.64.0.0/10
```

---

# 4. MAP-E Checklist

## MAP-E Definition

**MAP-E = Mapping of Address and Port using Encapsulation**

Checklist:

* [ ] IPv4 packets are encapsulated inside IPv6
* [ ] IPv4 address mapping is understood
* [ ] Port mapping is understood
* [ ] Border Relay role is understood

---

## MAP-E Traffic Flow

```text
IPv4 Client

     |
     v

FortiGate VNE

     |
     |
     | IPv4 inside IPv6
     |
     v

IPv6 Provider Network

     |
     v

Border Relay

     |
     v

IPv4 Internet
```

Verify:

* [ ] IPv6 connectivity exists
* [ ] Mapping parameters are correct
* [ ] Border Relay is reachable

---

# 5. DS-Lite Checklist

## DS-Lite Definition

DS-Lite transports IPv4 traffic over IPv6 toward an AFTR.

Architecture:

```text
IPv4 Client

     |
     v

CPE

     |
     |
 IPv4-in-IPv6

     |
     v

IPv6 Network

     |
     v

AFTR

     |
     v

IPv4 Internet
```

Checklist:

* [ ] Understand IPv4-in-IPv6 tunneling
* [ ] Understand AFTR function
* [ ] Understand centralized CGNAT
* [ ] Validate IPv6 reachability

---

# 6. DS-Lite vs MAP-E Checklist

| Feature             | DS-Lite     | MAP-E                   |
| ------------------- | ----------- | ----------------------- |
| Transport           | IPv6        | IPv6                    |
| IPv4 Handling       | Tunnel      | Mapping + Encapsulation |
| Main Component      | AFTR        | Border Relay            |
| NAT Location        | Centralized | Mapping-based           |
| Subscriber Identity | CGNAT state | Address/port mapping    |

Memory:

```text
DS-Lite
=
IPv4 over IPv6 + AFTR


MAP-E
=
IPv4 Mapping + IPv4 over IPv6 + BR
```

---

# 7. FortiGate VNE Configuration Checklist

## Enable VNE

```bash
config system vne
    set status enable
end
```

Checklist:

* [ ] VNE enabled
* [ ] Correct mode selected
* [ ] Correct parent interface selected
* [ ] Correct IPv6 transport configured

---

# 8. DS-Lite Configuration Checklist

Example:

```bash
config system vne
    set status enable
    set mode ds-lite
    set interface port2
    set br 2001:db8:12::2
    set ipv4-address 192.168.12.1/30
end
```

Validate:

* [ ] Mode = ds-lite
* [ ] BR address configured
* [ ] IPv4 tunnel address configured
* [ ] IPv6 connectivity available

---

# 9. MAP-E Configuration Checklist

Example:

```bash
config system vne
    set status enable
    set mode map-e
    set interface port2
end
```

Validate:

* [ ] MAP-E mode enabled
* [ ] Provider mapping parameters available
* [ ] BMR information understood
* [ ] Border Relay reachable

---

# 10. Fixed-IP Mode Checklist

Example:

```bash
config system vne
    set status enable
    set mode fixed-ip
    set interface port2
    set br 2001:db8:12::1
end
```

Validate:

* [ ] Provider parameters received
* [ ] Mapping information correct
* [ ] Update mechanism reachable

---

# 11. Routing Validation Checklist

## IPv4 Routing

```bash
get router info routing-table all
```

Check:

* [ ] IPv4 destination uses VNE interface
* [ ] Next-hop is correct
* [ ] Route exists

---

## IPv6 Routing

```bash
get router info6 routing-table
```

Check:

* [ ] IPv6 transport route exists
* [ ] Provider path works
* [ ] BR/AFTR reachable

---

# 12. Firewall Policy Validation Checklist

Verify:

* [ ] Source interface
* [ ] Destination interface
* [ ] Source address
* [ ] Destination address
* [ ] Service
* [ ] NAT requirement
* [ ] Logging enabled

Example:

```bash
config firewall policy
    edit 1
        set name "VNE-WAN"
        set action accept
        set nat enable
        set logtraffic all
    next
end
```

---

# 13. Port Block Allocation Checklist

CGNAT shares public IPv4 addresses.

Example:

```text
Public IPv4

+---------+---------+---------+
| User A  | User B  | User C  |
| Ports   | Ports   | Ports   |
+---------+---------+---------+
```

Validate:

* [ ] Port allocation strategy
* [ ] Subscriber scalability
* [ ] Application requirements
* [ ] Port exhaustion risk

Applications requiring attention:

* [ ] VoIP
* [ ] WebRTC
* [ ] P2P
* [ ] IoT
* [ ] High concurrency applications

---

# 14. VNE Diagnostic Checklist

## Check VNE Status

```bash
diagnose test application vned 1
```

---

## Check Interfaces

```bash
show system interface
```

Validate:

* [ ] VNE interface exists
* [ ] IPv4 address
* [ ] IPv6 address
* [ ] Interface state

---

## Check Sessions

IPv4:

```bash
diagnose sys session list
```

IPv6:

```bash
diagnose sys session6 list
```

---

# 15. Troubleshooting Workflow

```text
Client Problem

      |
      v

Check VNE Status

      |
      v

Check Interface

      |
      v

Check IPv6 Transport

      |
      v

Check Route

      |
      v

Check Firewall Policy

      |
      v

Check NAT / Mapping

      |
      v

Check Session

      |
      v

Debug Flow
```

---

# 16. Flow Debug Checklist

Start:

```bash
diagnose debug reset

diagnose debug flow filter addr <IP>

diagnose debug flow show function-name enable

diagnose debug enable

diagnose debug flow trace start 50
```

Stop:

```bash
diagnose debug disable

diagnose debug reset
```

---

# 17. Common Design Mistakes Checklist

## ❌ Treating MAP-E as Normal NAT

Wrong:

```text
Private IPv4
      |
      v
Public IPv4
```

Correct:

```text
IPv4 Mapping
+
IPv6 Encapsulation
+
Border Relay
```

---

## ❌ Confusing DS-Lite and MAP-E

Remember:

```text
DS-Lite
AFTR


MAP-E
Border Relay
```

---

## ❌ Forgetting IPv6 Routing

A working tunnel does not guarantee working transport.

Check:

```text
VNE
 ↓
IPv6 Route
 ↓
Provider
 ↓
AFTR / BR
```

---

## ❌ Using Production IPv6 Prefixes in Lab

Use:

```text
2001:db8::/32
```

---

# 18. NSE Exam High Value Notes 🧠

## DS-Lite

```text
IPv4
 ↓
IPv6 Tunnel
 ↓
AFTR
 ↓
Internet
```

---

## MAP-E

```text
IPv4 + Port Mapping
 ↓
IPv6 Encapsulation
 ↓
Border Relay
 ↓
Internet
```

---

## CGNAT

```text
Many Subscribers
        |
        v
One Public IPv4
        |
        v
Port Sharing
```

---

# 19. Golden Troubleshooting Order 🔥

```text
1. VNE Status

2. Interface

3. IPv6 Transport

4. Routing

5. Firewall Policy

6. NAT / Mapping

7. Session

8. Packet Debug
```

---

# 20. Quick Command Reference

| Purpose      | Command                             |
| ------------ | ----------------------------------- |
| VNE Config   | `config system vne`                 |
| VNE Test     | `diagnose test application vned 1`  |
| IPv4 Session | `diagnose sys session list`         |
| IPv6 Session | `diagnose sys session6 list`        |
| IPv4 Route   | `get router info routing-table all` |
| IPv6 Route   | `get router info6 routing-table`    |
| Debug Flow   | `diagnose debug flow`               |

---

# 🎯 Engineer Final Mental Model

```text
IPv4 Service

      |
      v

VNE

      |
      +----------------+
      |                |
      v                v

DS-Lite             MAP-E

AFTR                Mapping + BR

      |                |

      +-------IPv6-----+

              |

              v

        IPv4 Internet
```

---

# 🔥 One-Line Memory Hook

```text
DS-Lite
=
IPv4 over IPv6 + AFTR


MAP-E
=
IPv4 Mapping + Encapsulation + Border Relay


VNE
=
FortiGate IPv4/IPv6 Transition Engine
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

**SheynShield | Engineering Secure Networks**

#FortiGate #FortiOS #Fortinet #VNE #MAPE #DSLite #IPv6 #CGNAT #NetworkSecurity #NSE7 #NSE6 #CyberSecurity


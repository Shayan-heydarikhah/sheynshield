# 🔗 SheynShield Resources

# FortiGate Advanced SNAT, IP Pools, Central NAT & CGNAT Checklist

> **FortiOS Networking | Source NAT, IP Pool Design, Central SNAT, Deterministic NAT, Virtual Wire Pair & CGNAT Architecture**

---

# 📌 Table of Contents

- [1. SNAT Architecture Checklist](#1-snat-architecture-checklist)
- [2. Source Port Preservation Checklist](#2-source-port-preservation-checklist)
- [3. IP Pool Type Selection Checklist](#3-ip-pool-type-selection-checklist)
- [4. Overload IP Pool Checklist](#4-overload-ip-pool-checklist)
- [5. NAT Port Capacity Planning Checklist](#5-nat-port-capacity-planning-checklist)
- [6. Fixed Port Range Checklist](#6-fixed-port-range-checklist)
- [7. Port Block Allocation (PBA) Checklist](#7-port-block-allocation-pba-checklist)
- [8. CGNAT Design Checklist](#8-cgnat-design-checklist)
- [9. One-to-One NAT Checklist](#9-one-to-one-nat-checklist)
- [10. IP Pool ARP Reply Checklist](#10-ip-pool-arp-reply-checklist)
- [11. Central NAT Checklist](#11-central-nat-checklist)
- [12. Central SNAT Map Checklist](#12-central-snat-map-checklist)
- [13. Source Port Based NAT Matching Checklist](#13-source-port-based-nat-matching-checklist)
- [14. Virtual Wire Pair SNAT Checklist](#14-virtual-wire-pair-snat-checklist)
- [15. NAT Troubleshooting Workflow](#15-nat-troubleshooting-workflow)
- [16. NAT Exhaustion Checklist](#16-nat-exhaustion-checklist)
- [17. SIP / VoIP NAT Checklist](#17-sip--voip-nat-checklist)
- [18. NAT Session Lifecycle Checklist](#18-nat-session-lifecycle-checklist)
- [19. CGNAT VXLAN Design Checklist](#19-cgnat-vxlan-design-checklist)
- [20. NSE High Value Notes](#20-nse-high-value-notes)

---

# 1. SNAT Architecture Checklist

## Source NAT Flow Validation

Source NAT changes:

```text
Source IP
+
Source Port
````

Example:

```text
Before NAT

192.168.101.10:52341


        ↓ SNAT


After NAT

192.168.254.252:5377
```

---

## Deployment Checklist

* [ ] Identify internal source networks
* [ ] Identify external/public addresses
* [ ] Define NAT requirement
* [ ] Confirm firewall policy path
* [ ] Confirm return routing
* [ ] Validate session translation
* [ ] Validate NAT scalability

---

## NAT Mental Model

```text
Private Client

      |
      |
      ▼

FortiGate SNAT

      |
      |
      ▼

Public Address + Port
```

---

# 2. Source Port Preservation Checklist

## Validate Application Requirement

Check:

* [ ] SIP/VoIP dependency
* [ ] Legacy application dependency
* [ ] Source-port-sensitive application
* [ ] Vendor requirement

---

## Port Preservation Example

Preferred:

```text
10.10.10.10:5060

↓

203.0.113.10:5060
```

Possible alternative:

```text
10.10.10.10:5060

↓

203.0.113.10:49152
```

---

## Design Rule

* [ ] Preserve source port only when required
* [ ] Verify collision handling
* [ ] Avoid unnecessary fixed-port design

---

# 3. IP Pool Type Selection Checklist

Choose the correct NAT model:

| Requirement                | Recommended Type      |
| -------------------------- | --------------------- |
| Normal Internet Access     | Overload              |
| Dedicated Public Mapping   | One-to-One            |
| Deterministic Port Mapping | Fixed Port Range      |
| CGNAT Subscriber Scaling   | Port Block Allocation |

---

Checklist:

* [ ] Business requirement identified
* [ ] Number of users calculated
* [ ] Public IP capacity calculated
* [ ] Traceability requirement evaluated

---

# 4. Overload IP Pool Checklist

## Use Case

Multiple users share public IP addresses.

```text
10.10.10.10 ─┐
10.10.10.11 ─┤
10.10.10.12 ─┼── FortiGate ── 203.0.113.10
10.10.10.13 ─┘
```

---

## Configuration Validation

```bash
config firewall ippool
    edit "SNAT-OVERLOAD"
        set type overload
        set startip 192.168.254.252
        set endip 192.168.254.252
    next
end
```

---

Checklist:

* [ ] IP Pool created
* [ ] Pool assigned to policy
* [ ] NAT enabled
* [ ] Session creation verified
* [ ] Public IP utilization monitored

---

# 5. NAT Port Capacity Planning Checklist

## Important Planning Value

```text
60416 usable NAT ports
per IPv4 address
```

Do not assume:

```text
65536 ports
```

---

## Capacity Calculation

Example:

One public IP:

```text
203.0.113.10

≈ 60416 translations
```

Two public IPs:

```text
60416 × 2

≈ 120832 ports
```

---

Checklist:

* [ ] Concurrent sessions estimated
* [ ] Public IP count calculated
* [ ] NAT exhaustion risk evaluated
* [ ] Timeout values reviewed

---

# 6. Fixed Port Range Checklist

## Purpose

Deterministic:

```text
Internal IP

+

External IP

+

Port Range
```

---

## Validation

* [ ] Source ranges defined
* [ ] Public ranges defined
* [ ] Port allocation understood
* [ ] Application compatibility verified

---

Example:

```text
User A

↓

Public IP

↓

Dedicated Port Range
```

---

## Capacity Formula

```text
Ports Per User

=

Available Ports

÷

Number Of Internal IPs
```

---

Example:

```text
60416 / 10

≈ 6041 ports per user
```

---

# 7. Port Block Allocation (PBA) Checklist

## CGNAT Recommended Model

Use for:

* [ ] ISP environments
* [ ] Large subscriber networks
* [ ] Deterministic logging
* [ ] Subscriber traceability

---

## Key Parameters

```text
Block Size

Blocks Per User

PBA Timeout
```

---

## Capacity Formula

### Ports/User

```text
Block Size × Blocks/User
```

---

### Users/Public IP

```text
60416 / Ports Per User
```

---

Example:

```text
Block Size = 128

Blocks/User = 8


Ports/User:

128 × 8

=1024
```

---

Approximate:

```text
60416 / 1024

≈59 users
```

---

## PBA Checklist

* [ ] Block size selected
* [ ] Subscriber quota defined
* [ ] Public IP capacity calculated
* [ ] Logging requirement considered

---

# 8. CGNAT Design Checklist

## Architecture

```text
Internet

    |
    |
Public IP Pool

    |
    |
FortiGate CGNAT

    |
    |
Subscribers
```

---

Validate:

* [ ] Subscriber network defined
* [ ] RFC6598 range considered
* [ ] Public pool calculated
* [ ] Port allocation model selected

---

Common CGNAT range:

```text
100.64.0.0/10
```

---

# 9. One-to-One NAT Checklist

## Mapping Model

```text
Private              Public

10.10.10.10  <--> 203.0.113.10

10.10.10.11  <--> 203.0.113.11
```

---

Checklist:

* [ ] Dedicated mapping required
* [ ] Public addresses available
* [ ] PAT requirement removed
* [ ] Policy uses correct pool

---

# 10. IP Pool ARP Reply Checklist

## Validate When Pool Exists on Connected Network

Configuration:

```bash
set arp-reply enable
```

---

Check:

* [ ] Upstream router expects ARP response
* [ ] ARP interface configured
* [ ] Public IP ownership verified

---

Flow:

```text
ISP

↓

ARP Request

↓

FortiGate

↓

IP Pool Address
```

---

# 11. Central NAT Checklist

## Enable Central NAT

```bash
config system settings
    set central-nat enable
end
```

---

Architecture:

```text
Firewall Policy

(Security Decision)

        |

        ▼

Central SNAT

        |

        ▼

Translation
```

---

Checklist:

* [ ] Central NAT requirement confirmed
* [ ] Firewall policy separated from NAT logic
* [ ] SNAT rules ordered correctly

---

# 12. Central SNAT Map Checklist

Validate:

* [ ] Source interface
* [ ] Destination interface
* [ ] Original address
* [ ] Destination address
* [ ] NAT action
* [ ] IP Pool selection

---

Example:

```bash
config firewall central-snat-map
    edit 1
        set srcintf "LAN"
        set dstintf "WAN"
        set orig-addr "all"
        set dst-addr "all"
        set nat enable
    next
end
```

---

# 13. Source Port Based NAT Matching Checklist

Central SNAT can match source ports.

Example:

```bash
set origin-port 100-2000
```

---

Validate:

* [ ] Source port requirement exists
* [ ] Application dependency confirmed
* [ ] Rule does not block normal traffic

---

Avoid:

```text
Blind source-port restriction
```

---

# 14. Virtual Wire Pair SNAT Checklist

## Validate Topology

```text
Internet

 |

Mikrotik

 |

FortiGate VWP

 |

Internal Network
```

---

Checklist:

* [ ] VWP created
* [ ] Interfaces assigned
* [ ] Central NAT configured
* [ ] IP Pool subnet verified
* [ ] Return path tested

---

Important:

```text
IP Pool subnet

must be different

from VWP peer subnet
```

---

# 15. NAT Troubleshooting Workflow

## Step 1

Traffic Arrival:

* [ ] Packet reaches FortiGate
* [ ] Correct interface
* [ ] Correct source/destination

---

## Step 2

Policy:

* [ ] Firewall policy matches
* [ ] Action = Accept
* [ ] Logging enabled

---

## Step 3

SNAT:

* [ ] Correct NAT rule
* [ ] Correct IP Pool
* [ ] Available address

---

## Step 4

Session:

Check:

```bash
diagnose sys session list
```

Verify:

* [ ] Original source
* [ ] Translated source
* [ ] NAT information

---

## Step 5

Return Path:

* [ ] Routing exists
* [ ] Reply reaches FortiGate
* [ ] Session completes

---

# 16. NAT Exhaustion Checklist

Symptoms:

* [ ] New connections fail
* [ ] Existing sessions work
* [ ] NAT allocation errors
* [ ] High session count

---

Solutions:

* [ ] Add public IPs
* [ ] Enable PBA
* [ ] Optimize timeouts
* [ ] Review application behavior
* [ ] Scale CGNAT design

---

# 17. SIP / VoIP NAT Checklist

Validate:

* [ ] Source port requirement
* [ ] SIP helper behavior
* [ ] ALG impact
* [ ] RTP range
* [ ] Session timeout

---

Architecture:

```text
SIP Client

↓

FortiGate NAT

↓

SIP Provider
```

---

# 18. NAT Session Lifecycle Checklist

```text
New Connection

↓

NAT Allocation

↓

Session Creation

↓

Traffic

↓

Timeout / FIN / RST

↓

Resource Release
```

---

Validate:

* [ ] Session timeout
* [ ] Resource cleanup
* [ ] Long-lived applications

---

# 19. CGNAT VXLAN Design Checklist

Validate:

* [ ] IPv6 underlay available
* [ ] VXLAN tunnel established
* [ ] VNI configured
* [ ] VLAN mapped
* [ ] SNAT pool assigned

---

Architecture:

```text
IPv6 Underlay

        |

      VXLAN

        |

 Subscriber LAN

        |

      CGNAT
```

---

# 20. NSE High Value Notes 🧠

## Overload

```text
Normal PAT

Many users

One public IP
```

---

## One-to-One

```text
Dedicated public mapping
```

---

## Fixed Port Range

```text
Deterministic IP + Port allocation
```

---

## PBA

```text
Subscriber

↓

Port Block

↓

Public IP
```

---

## Central NAT

```text
Security Policy

+

Separate SNAT Logic
```

---

## Golden NAT Formula

```text
Ports/User

=

Block Size × Blocks/User
```

---

## Final SheynShield NAT Mental Model 🔥

```text
                NAT DESIGN

                     |

        ----------------------------

        Address

          |
          |
     Which Public IP?


        Port

          |
          |
     Which Port Range?


        Time

          |
          |
     How Long Session Exists?
```

---

# ⚡ 30 Second Review

```text
OVERLOAD

↓

Normal Internet PAT


ONE-TO-ONE

↓

Dedicated Mapping


FIXED PORT RANGE

↓

Deterministic Allocation


PBA

↓

Large Scale CGNAT


CENTRAL NAT

↓

Separate SNAT Decision Layer
```

---

# 🔥 Production Checklist

```text
[ ] Correct IP Pool Type Selected

[ ] NAT Capacity Calculated

[ ] Public IP Requirement Estimated

[ ] Port Exhaustion Risk Checked

[ ] Central NAT Order Verified

[ ] Return Routing Validated

[ ] Session Translation Verified

[ ] CGNAT Traceability Designed

[ ] VWP Requirements Checked

[ ] Logging Enabled
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

⭐ **SheynShield | Engineering Secure Networks**


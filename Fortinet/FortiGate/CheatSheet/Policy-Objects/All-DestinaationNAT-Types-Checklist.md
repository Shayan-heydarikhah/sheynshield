# 🔗 SheynShield Resources

# FortiGate VIP, DNAT, Virtual Server & ZTNA Access Proxy Checklist

> **FortiOS | VIP Matching, DNAT, VIP Types, Load Balancing, FQDN VIP, DNS Translation & ZTNA Access Proxy**

---

# ✅ FortiGate VIP / DNAT / ZTNA Deployment Checklist

## 📌 Table of Contents

- [1. VIP Architecture Validation](#1-vip-architecture-validation)
- [2. VIP Matching & match-vip](#2-vip-matching--match-vip)
- [3. Static NAT VIP Checklist](#3-static-nat-vip-checklist)
- [4. Port Forwarding Checklist](#4-port-forwarding-checklist)
- [5. Virtual Server & Load Balance Checklist](#5-virtual-server--load-balance-checklist)
- [6. Load Balance Algorithm Checklist](#6-load-balance-algorithm-checklist)
- [7. Session Persistence Checklist](#7-session-persistence-checklist)
- [8. Multiplexing & SSL Checklist](#8-multiplexing--ssl-checklist)
- [9. Gratuitous ARP Checklist](#9-gratuitous-arp-checklist)
- [10. DNS Translation Checklist](#10-dns-translation-checklist)
- [11. FQDN VIP Checklist](#11-fqdn-vip-checklist)
- [12. ZTNA Access Proxy Checklist](#12-ztna-access-proxy-checklist)
- [13. EMS Tag Based Access Control Checklist](#13-ems-tag-based-access-control-checklist)
- [14. VIP Troubleshooting Workflow](#14-vip-troubleshooting-workflow)
- [15. NSE Exam Key Points](#15-nse-exam-key-points)
- [16. Production Security Checklist](#16-production-security-checklist)

---

# 1. VIP Architecture Validation

## ✅ Before Deployment

- [ ] Identify external/public IP
- [ ] Identify mapped/internal server IP
- [ ] Confirm incoming interface
- [ ] Confirm backend routing
- [ ] Confirm return path through FortiGate
- [ ] Confirm required service ports
- [ ] Confirm application dependency

---

## Packet Flow Validation

```text
Client
   |
   |
External IP:Port
   |
   ▼
FortiGate VIP
   |
   ▼
DNAT Processing
   |
   ▼
Internal Server
   |
   ▼
Return Traffic
````

Checklist:

* [ ] Client reaches FortiGate
* [ ] VIP matches traffic
* [ ] Firewall policy matches
* [ ] DNAT occurs
* [ ] Backend responds
* [ ] Reply path is correct

---

# 2. VIP Matching & match-vip

## ✅ Understanding

VIP changes destination evaluation.

Example:

```text
Client
192.168.101.50

Destination:
192.168.101.100

        |
        ▼

VIP

        |
        ▼

Internal Server
192.168.20.200
```

---

## match-vip Checklist

Enable when policy must explicitly match VIP traffic.

```bash
set match-vip enable
```

Verify:

* [ ] Deny policies can identify VIP traffic
* [ ] VIP access control is intentional
* [ ] Policy order is correct
* [ ] VIP policy is not bypassed

---

## Security Validation

Example:

```text
Kali
 |
 ▼
VIP IIS

Should:
DENY
```

Verify:

* [ ] Source restriction exists
* [ ] VIP matching works
* [ ] Logs confirm correct policy hit

---

# 3. Static NAT VIP Checklist

## Architecture

```text
External IP

192.168.101.100

        |
        |
       VIP

        |
        |

Internal Server

192.168.20.200
```

---

## Configuration Checklist

* [ ] VIP object created
* [ ] External IP configured
* [ ] Mapped IP configured
* [ ] Interface selected
* [ ] Firewall policy created
* [ ] NAT disabled in policy

---

## Policy Validation

Recommended:

```text
Destination:

VIP Object
```

Avoid:

```text
Destination:

all
```

---

## Service Security

Prefer:

```text
HTTP
HTTPS
SSH
RDP
```

Avoid:

```text
ALL
```

---

# 4. Port Forwarding Checklist

## Port Translation Example

```text
External:

192.168.101.100:8081


Internal:

192.168.20.200:80
```

---

## Validation

* [ ] External port defined
* [ ] Internal port defined
* [ ] Custom service created
* [ ] Policy matches external port
* [ ] Backend service is running

---

## Common Error

Wrong:

```text
VIP:

8081 → 80


Policy:

HTTP
```

Correct:

```text
VIP:

8081 → 80


Policy:

TCP/8081
```

---

# 5. Virtual Server & Load Balance Checklist

## Identify Requirement

Choose:

### Static VIP

```text
One External IP

↓

One Backend Server
```

---

### Load Balance VIP

```text
One External IP

↓

Multiple Backend Servers
```

---

Checklist:

* [ ] Backend pool defined
* [ ] Health checks configured
* [ ] Load algorithm selected
* [ ] Persistence requirement evaluated

---

# 6. Load Balance Algorithm Checklist

## Least Sessions

Validate:

* [ ] Session count monitoring enabled
* [ ] Backend distribution checked

Example:

```text
Server A
100 sessions

Server B
20 sessions

Server C
40 sessions
```

Traffic preference:

```text
Server B
```

---

## Least RTT

Validate:

* [ ] Response time monitored
* [ ] Health measurement works

Example:

```text
Server A
80ms


Server B
20ms
```

Preferred:

```text
Server B
```

---

# 7. Session Persistence Checklist

Required when applications maintain local sessions.

Validate:

* [ ] Application requires stickiness
* [ ] Persistence method selected
* [ ] Cookie/session behavior tested

Without persistence:

```text
Client

Request 1 → Server A

Request 2 → Server B

Request 3 → Server C
```

With persistence:

```text
Client

Request 1 → Server A

Request 2 → Server A

Request 3 → Server A
```

---

# 8. Multiplexing & SSL Checklist

## Multiplexing

Validate:

* [ ] Proxy architecture supports reuse
* [ ] Backend connection optimization required
* [ ] Application compatibility tested

Concept:

```text
Many Clients

      |
      ▼

FortiGate

      |
      ▼

Backend Connection Reuse
```

---

## SSL Rekey

Checklist:

* [ ] SSL inspection/termination requirement confirmed
* [ ] Certificate configured
* [ ] FortiOS version behavior verified

Example:

```bash
set ssl-client-rekey-count 0
```

Always verify version-specific behavior.

---

# 9. Gratuitous ARP Checklist

Validate:

* [ ] VIP ownership changes considered
* [ ] Neighbor ARP updates required
* [ ] GARP behavior verified

Concept:

```text
VIP Change

↓

Gratuitous ARP

↓

Updated ARP Tables
```

---

# 10. DNS Translation Checklist

## Validation

* [ ] DNS feature enabled
* [ ] DNS record exists
* [ ] VIP DNS mapping configured
* [ ] TTL behavior verified

Example:

```text
shayan.test.co

↓

192.168.101.104

↓

VIP

↓

192.168.20.200
```

---

Troubleshoot:

* [ ] DNS server
* [ ] DNS records
* [ ] Interface DNS settings
* [ ] VIP configuration
* [ ] Firewall policy

---

# 11. FQDN VIP Checklist

## Flow

```text
Client

↓

FQDN

↓

DNS Resolution

↓

VIP

↓

Backend Server
```

---

Validation:

* [ ] External FQDN resolves
* [ ] DNS record correct
* [ ] VIP resolves correctly
* [ ] Backend reachable

---

Example:

```text
public.example.com

↓

192.168.101.105
```

---

# 12. ZTNA Access Proxy Checklist

## Enable Features

* [ ] ZTNA enabled
* [ ] Certificate configured
* [ ] Access proxy created
* [ ] Application published

---

Architecture:

```text
Remote User

↓

FortiGate

↓

ZTNA Access Proxy

↓

Identity Check

↓

EMS Tag Check

↓

Internal Application
```

---

Validate:

* [ ] External IP
* [ ] External Port
* [ ] Certificate
* [ ] Virtual Host
* [ ] Backend Server

---

# 13. EMS Tag Based Access Control Checklist

ZTNA Decision:

```text
Endpoint

↓

FortiClient

↓

EMS

↓

Security Tag

↓

FortiGate

↓

Policy Decision
```

---

Validate:

* [ ] Endpoint registered
* [ ] EMS connected
* [ ] Tag synchronized
* [ ] Proxy policy matches tag

---

Example:

Trusted:

```text
Low Risk
```

Action:

```text
ALLOW
```

---

Malicious:

```text
Malicious-File-Detected
```

Action:

```text
DENY
```

---

# 14. VIP Troubleshooting Workflow

## Step 1 — Packet Arrival

Check:

* [ ] Source IP
* [ ] Destination IP
* [ ] Destination port
* [ ] Incoming interface

---

## Step 2 — VIP Match

Verify:

* [ ] External IP
* [ ] External interface
* [ ] External port
* [ ] VIP type

---

## Step 3 — Firewall Policy

Check:

* [ ] Source
* [ ] Destination VIP
* [ ] Service
* [ ] Schedule
* [ ] Policy order

---

## Step 4 — DNAT

Confirm:

```text
External:

192.168.101.100:8081


After DNAT:

192.168.20.200:80
```

---

## Step 5 — Backend Reachability

Verify:

* [ ] Server listening
* [ ] Server firewall
* [ ] Default gateway
* [ ] Return route

---

## Diagnostic Commands

Endpoint:

```bash
diagnose endpoint record list
```

ZTNA UID:

```bash
diagnose endpoint lls-comm send ztna find-uid <UID>
```

FCNACD:

```bash
diagnose test application fcnacd 7
```

---

# 15. NSE Exam Key Points 🧠

## VIP

Remember:

```text
VIP = Destination NAT Publication
```

---

## match-vip

```bash
set match-vip enable
```

Meaning:

```text
Policy explicitly understands VIP destination
```

---

## Port Forwarding

Remember:

```text
External Port != Internal Port
```

Example:

```text
8081 → 80
```

---

## Load Balance

Remember:

```text
Least Sessions

Least RTT

Persistence
```

---

## ZTNA

Remember:

```text
Identity

+

Certificate

+

EMS Tag

+

Application Policy
```

---

# 16. Production Security Checklist

## VIP Security

* [ ] Use specific services
* [ ] Avoid ALL service
* [ ] Restrict source addresses
* [ ] Enable logging
* [ ] Review policy order
* [ ] Validate match-vip

---

## Application Security

* [ ] Protect exposed services
* [ ] Patch backend servers
* [ ] Monitor logs
* [ ] Validate certificates

---

## ZTNA Security

Avoid:

```text
IP Based Trust Only
```

Prefer:

```text
Identity

+

Device Posture

+

Certificate

+

EMS Tag

+

Application Control
```

---

# 🔥 Golden Mental Model

```text
                 FORTIGATE VIP

                      |
     -------------------------------------

     Static VIP       Load Balance       ZTNA

        |                  |               |

       DNAT          Backend Pool     Identity

                          |               |

                  Session / RTT       EMS Tags

                          |               |

                     Servers        Applications
```

---

# ⚡ 30 Second Review

```text
VIP
 |
 |-- Static NAT
 |      External IP → Internal IP
 |
 |-- Port Forwarding
 |      External Port → Internal Port
 |
 |-- FQDN VIP
 |      DNS → VIP
 |
 |-- Load Balance
 |      Multiple Backend Servers
 |
 |-- DNS Translation
 |      DNS Mapping
 |
 |-- Access Proxy
        ZTNA Application Access
```

---

# ⭐ Five Things To Remember

```text
1.
VIP publishes internal services.


2.
match-vip controls VIP-aware policy matching.


3.
External service and backend service can be different.


4.
Load balancing depends on sessions, RTT and persistence.


5.
ZTNA replaces IP trust with identity-based access.
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

```

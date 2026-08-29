# FortiGate VIP, DNAT, Virtual Server & ZTNA Access Proxy  

> **FortiOS | VIP Matching, DNAT, VIP Types, Server Load Balancing, FQDN VIP, DNS Translation & ZTNA Access Proxy**

---

## 📌 Table of Contents

* [1. VIP Matching — The Critical Concept](#1-vip-matching--the-critical-concept)
* [2. Why `match-vip` Matters](#2-why-match-vip-matters)
* [3. VIP Access Control Lab](#3-vip-access-control-lab)
* [4. VIP + DNAT Architecture](#4-vip--dnat-architecture)
* [5. Static NAT VIP](#5-static-nat-vip)
* [6. Port Forwarding](#6-port-forwarding)
* [7. Custom Service for Port Forwarding](#7-custom-service-for-port-forwarding)
* [8. Virtual Server / Server Load Balance](#8-virtual-server--server-load-balance)
* [9. VIP Load-Balance](#9-vip-load-balance)
* [10. Server Load-Balance Algorithms](#10-server-load-balance-algorithms)
* [11. Session Persistence](#11-session-persistence)
* [12. Multiplexing](#12-multiplexing)
* [13. SSL Client Rekey](#13-ssl-client-rekey)
* [14. Gratuitous ARP](#14-gratuitous-arp)
* [15. DNS Translation](#15-dns-translation)
* [16. FQDN VIP](#16-fqdn-vip)
* [17. Access Proxy / ZTNA](#17-access-proxy--ztna)
* [18. ZTNA Tags & EMS](#18-ztna-tags--ems)
* [19. ZTNA Policy Logic](#19-ztna-policy-logic)
* [20. Endpoint Troubleshooting](#20-endpoint-troubleshooting)
* [21. VIP Troubleshooting Workflow](#21-vip-troubleshooting-workflow)
* [22. NSE High-Value Notes](#22-nse-high-value-notes)
* [23. Quick Reference](#23-quick-reference)

---

# 1. VIP Matching — The Critical Concept

A very important FortiGate behavior appears when **VIP/DNAT** is used.

Consider:

```text
Internet / WAN
      │
      ▼
FortiGate
      │
      ▼
DMZ
```

A VIP might expose:

```text
External IP
192.168.101.100
      │
      │ DNAT
      ▼
Internal Server
192.168.20.200
```

The client does **not** directly request:

```text
192.168.20.200
```

Instead:

```text
Client
  │
  ▼
192.168.101.100
  │
  ▼
VIP
  │
  ▼
192.168.20.200
```

This distinction becomes extremely important when firewall policies are used to restrict access to VIPs.

---

# 2. Why `match-vip` Matters

Suppose you have:

```text
Policy #1
Kali → DMZ
Action = DENY
```

and:

```text
Policy #2
Any → VIP-IIS
Action = ACCEPT
```

Without careful VIP matching, the intended security boundary can become confusing because VIP/DNAT processing changes how destination matching is evaluated.

The critical option is:

```bash
set match-vip enable
```

---

## What Does `match-vip` Solve?

Conceptually:

```text
Kali
 │
 │ Request VIP
 ▼
FortiGate
 │
 ├── VIP Matching
 │
 ├── DNAT
 │
 └── Firewall Policy Matching
```

With:

```bash
set match-vip enable
```

the deny policy can specifically match traffic destined for a VIP.

This is particularly useful when you want to ensure that a restricted source cannot bypass your intended VIP access-control logic.

---

# 3. VIP Access Control Lab

## Address Objects

Example:

```text
KALI-LINUX
192.168.101.101
```

VIP:

```text
VIP-IIS
External IP:
192.168.101.100

Mapped IP:
192.168.20.200
```

---

## Policy #1 — Deny Kali

```bash
config firewall policy
    edit 1
        set name "deny-kali"
        set srcintf "wan1"
        set dstintf "dmz"
        set srcaddr "192.168.101.101-kali"
        set dstaddr "all"
        set action deny
        set schedule "always"
        set service "ALL"
        set match-vip enable
    next
end
```

This makes the policy capable of matching VIP-destined traffic for the intended restriction.

---

## Policy #2 — Allow VIP

```bash
config firewall policy
    edit 2
        set name "vip-access"
        set srcintf "wan1"
        set dstintf "dmz"
        set srcaddr "all"
        set dstaddr "vip-iis"
        set action accept
        set schedule "always"
        set service "ALL"
    next
end
```

Conceptually:

```text
                 Traffic
                    │
                    ▼
             Source = Kali?
                    │
              ┌─────┴─────┐
             YES           NO
              │             │
              ▼             ▼
        Policy #1       Policy #2
        DENY            VIP ACCEPT
              │
              ▼
             DROP
```

---

# 4. VIP + DNAT Architecture

A normal VIP performs destination NAT:

```text
Client
192.168.101.x
     │
     │ Destination:
     │ 192.168.101.100
     ▼
 FortiGate
     │
     │ DNAT
     ▼
192.168.20.200
```

Think:

```text
External IP
     │
     │ VIP
     ▼
Mapped IP
```

---

## Example

```text
VIP:
External IP = 192.168.101.100
Mapped IP   = 192.168.20.200
```

Request:

```text
192.168.101.50
      ↓
192.168.101.100:80
```

After DNAT:

```text
192.168.101.50
      ↓
192.168.20.200:80
```

---

# 5. Static NAT VIP

A basic VIP can operate as static destination NAT.

Example:

```text
VIP Name:
vip-test

Interface:
port3

External IP:
192.168.101.100

Mapped IP:
192.168.20.200
```

Conceptually:

```text
                 VIP
                  │
        ┌─────────┴─────────┐
        │                   │
External IP             Mapped IP
192.168.101.100  →  192.168.20.200
```

---

## Firewall Policy

```text
Incoming Interface
        ↓
port3

Source
        ↓
all

Destination
        ↓
vip-test

Service
        ↓
Preferably specific service

NAT
        ↓
Disabled
```

For a simple HTTP publication:

```text
Client
  ↓
VIP
  ↓
IIS
```

prefer:

```text
Service = HTTP
```

rather than:

```text
Service = ALL
```

when operationally possible.

---

# 6. Port Forwarding

A VIP can also translate the destination port.

Example:

```text
External:
192.168.101.100:8081

Mapped:
192.168.20.200:80
```

Architecture:

```text
Client
   │
   │ 192.168.101.100:8081
   ▼
FortiGate VIP
   │
   │ DNAT
   ▼
192.168.20.200:80
```

This is:

```text
Destination IP Translation
+
Destination Port Translation
```

---

# 7. Custom Service for Port Forwarding

This is an important operational detail.

If the external service is:

```text
TCP/8081
```

while the backend service is:

```text
TCP/80
```

the firewall policy must match the **incoming/client-side service**, not simply the backend port.

Conceptually:

```text
Client
192.168.101.100:8081
       │
       ▼
     VIP
       │
       │ DNAT
       ▼
192.168.20.200:80
```

The policy sees the original incoming destination port as part of the traffic classification.

---

## Recommended Approach

Create a custom service:

```text
Name:
HTTP-8081

Protocol:
TCP

Destination Port:
8081
```

Then use:

```text
Policy Service = HTTP-8081
```

rather than assuming the backend HTTP service definition is equivalent to the external port.

---

## ⚠️ Common Mistake

```text
VIP:
8081 → 80

Policy:
HTTP
```

This can create a mismatch because:

```text
HTTP = TCP/80
```

while the client is connecting to:

```text
TCP/8081
```

Better:

```text
VIP:
8081 → 80

Policy:
TCP/8081
```

---

# 8. Virtual Server / Server Load Balance

FortiGate also supports more advanced VIP functionality for load balancing.

Important distinction:

```text
Basic VIP
   ↓
Destination NAT

Server Load Balance / Virtual Server
   ↓
Traffic Distribution
+
Advanced Application Handling
```

Virtual-server style deployments can operate at:

```text
Layer 3
Layer 4
```

depending on the VIP type and configuration.

For application-aware virtual-server behavior, proxy-based inspection may be required depending on the specific feature/configuration.

---

# 9. VIP Load-Balance

Example:

```bash
config firewall vip
    edit "lb-test"
        set type load-balance
        set extintf "port3"
        set extip 192.168.101.102-192.168.101.103
        set mappedip 192.168.20.200-192.168.20.201
    next
end
```

Conceptually:

```text
                 VIP
                  │
         ┌────────┴────────┐
         │                 │
         ▼                 ▼
   Server 1            Server 2
192.168.20.200      192.168.20.201
```

The FortiGate distributes sessions between backend servers.

---

# 10. Server Load-Balance Algorithms

Load balancing requires a decision mechanism.

Common concepts include:

```text
Least Sessions
Least RTT
```

---

## Least Sessions

FortiGate considers the number of active sessions.

```text
Server A
Sessions = 100

Server B
Sessions = 20

Server C
Sessions = 40
```

Traffic can preferentially be sent toward:

```text
Server B
```

because it currently has fewer sessions.

Conceptually:

```text
FortiGate
   │
   ├── Server A → 100 sessions
   ├── Server B → 20 sessions  ← selected
   └── Server C → 40 sessions
```

---

## Least RTT

The selection can use measured response-time information/health-check behavior.

Example:

```text
Server A → RTT 80 ms
Server B → RTT 20 ms
Server C → RTT 50 ms
```

Preferred:

```text
Server B
```

---

# 11. Session Persistence

Web applications often require the same client to continue reaching the same backend server.

This is commonly called:

> **Session Persistence / Stickiness**

Without persistence:

```text
Client
  │
  ├── Request 1 → Server A
  ├── Request 2 → Server B
  ├── Request 3 → Server C
```

This can break applications that maintain server-local session state.

---

## With Persistence

```text
Client
  │
  ├── Request 1 → Server A
  ├── Request 2 → Server A
  ├── Request 3 → Server A
```

Common persistence mechanisms include:

```text
Cookie-based persistence
Session persistence
```

---

# 12. Multiplexing

Multiplexing allows multiple client-side requests/connections to reuse connections toward backend servers where supported by the relevant proxy/load-balancing architecture.

Conceptually:

```text
Client A ──┐
Client B ──┼──> FortiGate ───> Backend Connection
Client C ──┘
```

Instead of creating an independent backend connection for every client-side interaction, the proxy can reuse backend connections where supported.

Think:

```text
Many Client Connections
        ↓
     FortiGate
        ↓
Connection Reuse
        ↓
Backend
```

This can reduce backend connection overhead.

---

# 13. SSL Client Rekey

In SSL-terminating/load-balancing scenarios, FortiGate can participate in TLS processing.

A rekey mechanism can be used to periodically establish fresh cryptographic keys.

Conceptually:

```text
Client
  │
  │ TLS
  ▼
FortiGate
  │
  │ TLS / Backend
  ▼
Server
```

A configuration such as:

```bash
set ssl-client-rekey-count 0
```

must be interpreted according to the exact FortiOS release and feature context.

> ⚠️ Do not treat `0` as a universal security recommendation. Always verify the semantics for the target FortiOS version.

---

# 14. Gratuitous ARP

VIP/load-balancing environments may use **Gratuitous ARP (GARP)** to update neighboring devices about IP/MAC reachability changes.

Conceptually:

```text
VIP / Address Ownership Change
           │
           ▼
    Gratuitous ARP
           │
           ▼
Neighbor ARP Tables
           │
           ▼
Updated Reachability
```

A configuration such as:

```bash
set gratuitous-arps-interval 0
```

changes the GARP behavior according to FortiOS semantics.

> ⚠️ Verify the exact behavior in the target FortiOS release before changing this value in production.

---

# 15. DNS Translation

VIP functionality can also support DNS translation.

Example:

```bash
config firewall vip
    edit "dns-test"
        set type dns-translation
        set extip 192.168.101.104
        set extintf "port3"
        set mappedip 192.168.20.200
        set dns-mapping-ttl 0
    next
end
```

Conceptually:

```text
Client
  │
  │ DNS / Name
  ▼
FortiGate DNS Translation
  │
  ▼
Translated Address
```

---

## DNS Mapping TTL

Example:

```bash
set dns-mapping-ttl 0
```

The TTL controls DNS caching behavior according to the configured value.

When troubleshooting DNS translation, inspect:

```text
DNS Server
DNS Role
Interface DNS settings
DNS Records
VIP
Firewall Policy
```

---

# 16. DNS Translation — Lab Model

Enable the required DNS server functionality through Feature Visibility when applicable.

Conceptual setup:

```text
Interface:
port3

DNS name:
shayan.test.co

VIP IP:
192.168.101.104
```

Then ensure DNS resolution maps the intended name/address.

Example:

```text
shayan.test.co
        ↓
192.168.101.104
```

and the VIP maps the external address to:

```text
192.168.20.200
```

---

# 17. FQDN VIP

An FQDN-based VIP uses a DNS name rather than a static destination IP.

Conceptually:

```text
Client
  │
  │ FQDN
  ▼
DNS
  │
  ▼
Resolved IP
  │
  ▼
FortiGate VIP
  │
  ▼
Mapped Server
```

Example objects:

```text
External FQDN:
shayan.test.co

Mapped FQDN:
s.test.co
```

DNS:

```text
shayan.test.co → 192.168.101.105

s.test.co → 192.168.20.200
```

---

## FQDN VIP Flow

```text
Client
   │
   ▼
shayan.test.co
   │
   ▼
DNS Resolution
   │
   ▼
192.168.101.105
   │
   ▼
FortiGate VIP
   │
   ▼
s.test.co
   │
   ▼
192.168.20.200
```

> 🧠 **NSE Tip:** FQDN VIP functionality depends on DNS resolution and the relevant FortiOS VIP behavior. Always troubleshoot both the DNS layer and firewall layer.

---

# 18. Access Proxy / ZTNA

Another VIP type is:

```text
access-proxy
```

This is associated with **ZTNA / application access** use cases.

Enable the relevant feature:

```text
System
  ↓
Feature Visibility
  ↓
ZTNA
```

Conceptually:

```text
Remote Client
      │
      ▼
FortiGate
      │
      ▼
ZTNA Access Proxy
      │
      ├── Identity
      ├── Certificate
      ├── EMS Tags
      ├── Policy
      └── Application
      │
      ▼
Internal Server
```

---

# 19. ZTNA Server

A ZTNA server can define:

```text
Name
External Interface
External IP
External Port
Certificate
Protocol
Path
Virtual Host
```

Example:

```text
Name:
zttest

Interface:
port3

External IP:
192.168.101.107

External Port:
8443

Protocol:
HTTPS
```

The external endpoint becomes:

```text
https://192.168.101.107:8443
```

---

# 20. ZTNA VIP

Example:

```bash
config firewall vip
    edit "winsrv-2016"
        set type access-proxy
        set extip 192.168.101.107
        set extintf "port3"
        set server-type https
        set extport 8443
        set ssl-certificate "test.com"
    next
end
```

Conceptually:

```text
Internet Client
      │
      │ HTTPS :8443
      ▼
192.168.101.107
      │
      ▼
ZTNA Access Proxy
      │
      ▼
ZTNA Policy
      │
      ▼
Internal Application
192.168.20.200
```

---

# 21. ZTNA Tags & EMS

ZTNA can use endpoint information obtained from **FortiClient / FortiClient EMS**.

Example:

```text
EMS
 │
 ├── Endpoint
 │
 ├── Identity
 │
 ├── Certificate
 │
 └── Security Tags
          │
          ▼
       FortiGate
```

A tag can represent endpoint state.

Example:

```text
Malicious-File-Detected
```

The FortiGate can use the tag as part of access-control logic.

---

# 22. ZTNA Policy Logic

Example deny policy:

```bash
config firewall proxy-policy
    edit 3
        set name "ztna-deny-malware"
        set proxy access-proxy
        set access-proxy "winsrv-2016"
        set srcintf "port3"
        set srcaddr "all"
        set dstaddr "all"
        set ztna-ems-tag "FCTEMS0000109188_Malicious-File-Detected"
        set schedule "always"
        set logtraffic "all"
    next
end
```

Then an allow policy:

```bash
config firewall proxy-policy
    edit 2
        set name "proxy-winsrv-2016"
        set proxy access-proxy
        set access-proxy "winsrv-2016"
        set srcintf "port3"
        set srcaddr "all"
        set dstaddr "all"
        set ztna-ems-tag "FCTEMS0000109188_Low"
        set action accept
        set schedule "always"
        set logtraffic "all"
    next
end
```

---

# 23. ZTNA Policy Decision

Think about ZTNA as:

```text
                Client
                  │
                  ▼
             Authentication
                  │
                  ▼
            Client Identity
                  │
                  ▼
             EMS Endpoint
                  │
                  ▼
                Tag
                  │
          ┌───────┴────────┐
          ▼                ▼
     Malicious          Trusted
          │                │
          ▼                ▼
        DENY             ALLOW
```

---

# 24. Endpoint Security Tag Example

Suppose EMS detects a condition such as:

```text
c:/virus.txt
```

and assigns a security tag:

```text
Malicious-File-Detected
```

The flow becomes:

```text
Endpoint
   │
   ▼
FortiClient
   │
   ▼
FortiClient EMS
   │
   ▼
Security Tag
   │
   ▼
FortiGate ZTNA
   │
   ▼
ZTNA Proxy Policy
   │
   ▼
DENY
```

This is a key ZTNA concept:

> **Access can depend on endpoint posture, not merely source IP.**

---

# 25. Endpoint Troubleshooting

To inspect endpoint records:

```bash
diagnose endpoint record list
```

Useful information can include:

```text
IP address
MAC address
VDOM
EMS serial
Client certificate
Public IP
Online status
Registration status
On-net status
FortiClient version
UID
Hostname
OS
```

Example conceptual output:

```text
IP Address:
10.10.10.20

EMS:
FCTEMS0000109188

Registration:
registered

Online:
online

On-Net:
on-net
```

---

# 26. FortiClient Endpoint UID

For endpoint-specific troubleshooting:

```bash
diagnose endpoint lls-comm send ztna find-uid <UID>
```

Example:

```bash
diagnose endpoint lls-comm send ztna find-uid F4F3263AEBE54777A6509A8FCCDF9284
```

`lls` refers to the low-level service communication mechanism used in this troubleshooting context.

---

# 27. FCNACD Diagnostic

A useful diagnostic command for endpoint/ZTNA troubleshooting:

```bash
diagnose test application fcnacd 7
```

Use it when investigating:

* Endpoint communication
* Endpoint registration
* ZTNA endpoint state
* FortiClient/EMS interaction

---

# 28. VIP Troubleshooting Workflow

When a VIP does not work, follow the packet path.

```text
                 CLIENT
                    │
                    ▼
             External IP/Port
                    │
                    ▼
                Interface
                    │
                    ▼
                 VIP Match
                    │
                    ▼
              Firewall Policy
                    │
                    ▼
                  DNAT
                    │
                    ▼
              Internal Server
                    │
                    ▼
                Reply Path
```

---

## Step 1 — Is the Client Reaching FortiGate?

Capture:

```text
Incoming packet
Source IP
Destination IP
Destination Port
Interface
```

---

## Step 2 — Does the VIP Match?

Check:

```text
External IP
External Interface
External Port
VIP Type
```

---

## Step 3 — Does the Firewall Policy Match?

Verify:

```text
Source
Destination
Service
Schedule
Policy Order
Action
```

---

## Step 4 — Is the Service Correct?

For:

```text
8081 → 80
```

verify that the policy matches the **external/client-side port 8081** where required.

---

## Step 5 — Does DNAT Occur?

Confirm that:

```text
192.168.101.100:8081
```

becomes:

```text
192.168.20.200:80
```

---

## Step 6 — Does the Server Reply?

Check:

```text
Server
  ↓
Default Gateway
  ↓
FortiGate
  ↓
Client
```

A broken return route can make a correctly configured VIP appear broken.

---

# 29. VIP Troubleshooting Decision Tree

```text
                 VIP NOT WORKING
                       │
                       ▼
                Packet reaches FG?
                  │            │
                 NO           YES
                  │            │
                  ▼            ▼
             Check WAN     VIP matched?
                              │
                         ┌────┴────┐
                        NO        YES
                         │          │
                         ▼          ▼
                    Check VIP   Policy matched?
                                   │
                              ┌────┴────┐
                             NO        YES
                              │          │
                              ▼          ▼
                         Check policy  DNAT?
                                         │
                                    ┌────┴────┐
                                   NO        YES
                                    │          │
                                    ▼          ▼
                               Check VIP   Server reachable?
                                                  │
                                             ┌────┴────┐
                                            NO        YES
                                             │          │
                                             ▼          ▼
                                       Routing/ACL   Check reply
```

---

# 30. VIP Type Comparison

| VIP Type            | Main Purpose              | Typical Layer / Behavior |
| ------------------- | ------------------------- | ------------------------ |
| Static NAT          | Basic destination NAT     | L3/L4                    |
| FQDN                | Name-based VIP            | DNS-dependent            |
| Load-Balance        | Basic load distribution   | L3-oriented              |
| Server Load Balance | Advanced server balancing | L3/L4                    |
| DNS Translation     | DNS-related translation   | DNS + VIP                |
| Access Proxy        | ZTNA/application access   | Proxy/application        |

> ⚠️ Exact capabilities and limits depend on FortiOS release and platform.

---

# 31. Load Balancing vs Static VIP

```text
Static VIP

Client
  │
  ▼
VIP
  │
  ▼
Server
```

---

```text
Load-Balance VIP

Client
  │
  ▼
VIP
  │
  ├────> Server A
  │
  ├────> Server B
  │
  └────> Server C
```

---

# 32. High-End Server Load-Balance Capacity

Some high-end FortiGate platforms can support large numbers of server-load-balance objects.

For example, platform documentation may specify capacities around:

```text
10,000 Server Load-Balance objects
```

> ⚠️ **Do not memorize 10K as a universal FortiGate limit.** VIP/server-load-balance capacity is platform- and FortiOS-version-specific. Always check the relevant maximum-value table for the exact model/version.

---

# 33. Security Best Practices

### ✅ Prefer specific services

Instead of:

```text
Service = ALL
```

use:

```text
Service = HTTP
```

or:

```text
Service = HTTPS
```

where appropriate.

---

### ✅ Restrict trusted clients

For sensitive VIPs:

```text
Source
   ↓
Trusted Client Group
   ↓
VIP
```

rather than:

```text
Source = all
```

---

### ✅ Control VIP discovery/access

Use appropriate:

```text
Source restrictions
Policy ordering
match-vip
Logging
Service restrictions
```

---

### ✅ For ZTNA

Do not rely solely on:

```text
Source IP
```

Combine:

```text
Identity
+
Certificate
+
EMS Tag
+
Application
+
Policy
```

---

# 34. NSE High-Value Notes 🧠

## `match-vip`

```text
set match-vip enable
```

Think:

> **Make the policy explicitly participate in VIP destination matching.**

---

## Port Forwarding

```text
External:
8081

Mapped:
80
```

Remember:

```text
Client connects to 8081
Backend receives 80
```

Policy service matching must account for the **incoming service/port**.

---

## Least Sessions

```text
FortiGate
   ↓
Session Table
   ↓
Count backend sessions
   ↓
Select less-loaded server
```

---

## Least RTT

```text
Health/response measurement
       ↓
RTT
       ↓
Select lower-latency server
```

---

## Session Persistence

```text
Client
   ↓
Same backend
   ↓
Application session continuity
```

---

## FQDN VIP

```text
FQDN
 ↓
DNS
 ↓
Resolved IP
 ↓
VIP
```

---

## Access Proxy

```text
Client
 ↓
ZTNA Access Proxy
 ↓
Identity / Certificate / EMS
 ↓
Proxy Policy
 ↓
Internal Application
```

---

# 35. Golden Mental Model 🔥

```text
                         FORTIGATE VIP
                              │
          ┌───────────────────┼────────────────────┐
          │                   │                    │
          ▼                   ▼                    ▼
       Static              Load Balance         Access Proxy
          │                   │                    │
          ▼                   ▼                    ▼
        DNAT              Backend Pool           ZTNA
                              │                    │
                     ┌────────┴────────┐      ┌────┴────┐
                     ▼                 ▼      ▼         ▼
                  Session          RTT/Load  Identity  EMS
                  Count            Health    Cert      Tags
                     │                 │      │         │
                     └────────┬────────┘      └────┬────┘
                              ▼                    ▼
                         Backend Server       Application
```

---

# 36. Final   — 30 Second Review

```text
VIP
 │
 ├── Static NAT
 │      └── External IP → Internal IP
 │
 ├── Port Forwarding
 │      └── External Port → Internal Port
 │
 ├── FQDN
 │      └── DNS name → VIP resolution
 │
 ├── Load Balance
 │      └── Distribute traffic
 │
 ├── Server Load Balance
 │      ├── L3/L4
 │      ├── Least Sessions
 │      ├── Least RTT
 │      ├── Persistence
 │      └── Multiplexing
 │
 ├── DNS Translation
 │      └── DNS mapping / translation
 │
 └── Access Proxy
        └── ZTNA / application access
```

### 🔥 Remember These Five

```text
1. VIP = Destination NAT / Publication

2. match-vip = Important for explicit VIP matching behavior

3. External Port ≠ Backend Port
   → Policy must account for client-side service

4. Load Balance
   → Session count / RTT / persistence matter

5. Access Proxy
   → ZTNA + Identity + Certificate + EMS Tags
```

> **Production rule:** When troubleshooting a VIP, never look only at the firewall policy. Follow the complete chain: **Client → External IP/Port → VIP Match → Policy → DNAT → Backend → Return Path**. Most VIP problems become obvious once this packet path is traced systematically.
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

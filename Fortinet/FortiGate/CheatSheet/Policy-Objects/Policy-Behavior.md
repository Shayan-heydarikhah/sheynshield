# FortiGate Advanced Networking & Policy Behavior  

> **FortiOS | MTU, TCP MSS, ASIC Offload, Session TTL, NGFW Modes, Land Attack Protection, Application Bandwidth Tracking & Learning Mode**

---

## 📌 Table of Contents

* [1. MTU vs TCP MSS](#1-mtu-vs-tcp-mss)
* [2. TCP MSS Clamping](#2-tcp-mss-clamping)
* [3. Interface MSS vs Policy MSS](#3-interface-mss-vs-policy-mss)
* [4. MTU Override](#4-mtu-override)
* [5. Path MTU Discovery](#5-path-mtu-discovery)
* [6. MTU/MSS Reference Values](#6-mtumss-reference-values)
* [7. Testing MTU with DF Bit](#7-testing-mtu-with-df-bit)
* [8. Hardware / ASIC Acceleration](#8-hardware--asic-acceleration)
* [9. Session TTL](#9-session-ttl)
* [10. Custom Session TTL](#10-custom-session-ttl)
* [11. Session TTL Troubleshooting](#11-session-ttl-troubleshooting)
* [12. NGFW Modes](#12-ngfw-modes)
* [13. Policy-Based vs Profile-Based](#13-policy-based-vs-profile-based)
* [14. Policy-Based Mode](#14-policy-based-mode)
* [15. Enforce Default Application Port](#15-enforce-default-application-port)
* [16. Central NAT](#16-central-nat)
* [17. SSL Inspection & Authentication](#17-ssl-inspection--authentication)
* [18. HTTP/SSH Policy Redirect](#18-httpssh-policy-redirect)
* [19. Land Attack Protection](#19-land-attack-protection)
* [20. Application Bandwidth Tracking](#20-application-bandwidth-tracking)
* [21. Learning Mode](#21-learning-mode)
* [22. Learning Mode Limitations](#22-learning-mode-limitations)
* [23. Advanced Troubleshooting Workflow](#23-advanced-troubleshooting-workflow)
* [24. NSE High-Value Notes](#24-nse-high-value-notes)

---

# 1. MTU vs TCP MSS

Two parameters are frequently confused:

```text
MTU
↓
Maximum IP packet size that can be transmitted
over an interface/path without fragmentation.

MSS
↓
Maximum TCP payload size carried inside an IP packet.
```

The basic relationship for normal IPv4 TCP is:

```text
MSS = MTU - IP Header - TCP Header
```

Normally:

```text
MSS = MTU - 40
```

because:

```text
IPv4 Header = 20 bytes
TCP Header  = 20 bytes
```

Therefore:

```text
MTU 1500
    ↓
MSS 1460
```

---

## ⚠️ Important

**Changing MSS does NOT automatically change the interface MTU.**

For example:

```bash
config firewall policy
    edit 1
        set tcp-mss-sender 1448
        set tcp-mss-receiver 1448
    next
end
```

This performs **TCP MSS adjustment/clamping** for traffic matching the policy.

It does **not** mean:

```text
MTU = 1488
```

or that the physical/interface MTU has been changed.

Think:

```text
MTU
 │
 ├── Interface / Path property
 │
 └── Maximum packet size

MSS
 │
 ├── TCP property
 │
 └── Maximum TCP payload
```

---

# 2. TCP MSS Clamping

MSS manipulation is particularly useful when encapsulation adds overhead.

Typical problem:

```text
Client
   │
   │ MTU 1500
   ▼
FortiGate
   │
   │ IPsec / GRE / PPPoE / Tunnel
   ▼
Internet
```

The original 1500-byte packet may no longer fit after encapsulation.

Instead of allowing oversized TCP packets to create fragmentation or PMTUD problems, FortiGate can advertise a smaller MSS.

Example:

```bash
config firewall policy
    edit 1
        set tcp-mss-sender 1448
        set tcp-mss-receiver 1448
    next
end
```

Conceptually:

```text
Normal TCP MSS
      │
      ▼
   1460
      │
      │ MSS Clamp
      ▼
   1448
```

---

# 3. Interface MSS vs Policy MSS

MSS can be controlled at different configuration scopes depending on FortiOS version and interface/policy capabilities.

## Interface-Level

Use interface-level TCP MSS behavior when you want a broader rule for traffic entering/leaving an interface.

Conceptually:

```text
Interface
   │
   └── TCP MSS adjustment
          │
          └── Applies broadly
```

---

## Policy-Level

Example:

```bash
config firewall policy
    edit 1
        set tcp-mss-sender 1448
        set tcp-mss-receiver 1448
    next
end
```

Conceptually:

```text
Policy #1
    │
    ├── Client A → MSS 1448
    └── Client B → MSS 1448
```

while another policy can use:

```text
Policy #2
    │
    └── Different MSS
```

### 🧠 Design Rule

```text
Interface-level
    ↓
Broad behavior

Policy-level
    ↓
Granular / traffic-specific behavior
```

This makes policy-level MSS adjustment particularly useful when only one application, tunnel, WAN path, or policy has an MTU problem.

---

# 4. MTU Override

MTU is an interface property.

Example:

```bash
config system interface
    edit "port1"
        set mtu-override enable
        set mtu 1480
    next
end
```

Now:

```text
port1
MTU = 1480
```

instead of the normal interface MTU.

Therefore the theoretical IPv4 TCP MSS becomes approximately:

```text
1480 - 40 = 1440
```

---

## MTU vs MSS

```text
Interface:

MTU = 1480
        │
        ▼
TCP MSS ≈ 1440
```

Whereas:

```text
Policy:

MSS = 1448
```

only changes TCP MSS negotiation/adjustment.

---

# 5. Path MTU Discovery

PMTUD attempts to determine the largest packet size that can traverse the path without fragmentation.

Conceptually:

```text
Source
  │
  │ Packet
  ▼
Router
  │
  │ Smaller MTU
  ▼
Router
  │
  ▼
Destination
```

If the path cannot carry the packet, PMTUD mechanisms can signal the sender to reduce packet size.

---

## FortiGate Configuration

Example:

```bash
config system interface
    edit "port1"
        set pmtu-discovery enable
    next
end
```

Use this carefully in environments involving:

* Tunnels
* VPN
* MPLS
* Firewalls
* Asymmetric paths
* ICMP filtering
* Legacy applications

---

# 6. MTU/MSS Reference Values

These are **typical starting points**, not universal FortiGate requirements.

| Environment            |              Typical MTU |               Typical TCP MSS |
| ---------------------- | -----------------------: | ----------------------------: |
| Standard Ethernet      |                     1500 |                          1460 |
| PPPoE                  |                     1492 |                          1452 |
| Reduced-MTU path       |                     1480 |                          1440 |
| VPN/IPsec              |      Depends on overhead |              Often ~1350–1400 |
| GRE/IPsec combinations | Depends on encapsulation | Often requires MSS adjustment |
| Satellite links        |     Often path-dependent |           MSS may need tuning |

---

## Important MPLS Note

A value such as:

```text
1508
```

is commonly associated with **provider-side Ethernet frames / baby jumbo requirements** used to preserve a 1500-byte IP MTU after adding MPLS-related overhead.

Do **not** assume:

```text
MPLS MTU = 1508
```

for every deployment.

Always distinguish:

```text
Ethernet Frame Size
vs
IP MTU
vs
MPLS Label Stack Overhead
```

---

# 7. Testing MTU with DF Bit

The DF (Don't Fragment) bit is useful for testing the maximum packet size across a path.

Example concept:

```text
execute ping-option df-bit
```

Then use different packet sizes.

Conceptually:

```text
Packet = 1500
DF = ON
       │
       ▼
Path cannot fragment
       │
       ▼
Failure
```

Reduce the packet size:

```text
1500
 ↓
1490
 ↓
1480
 ↓
1472
```

until the packet succeeds.

This helps identify the effective path MTU.

---

## Practical Calculation

For a normal IPv4 ICMP ping:

```text
IP packet size
=
ICMP payload
+
IP header
```

Therefore a common test payload of:

```text
1472 bytes
```

corresponds approximately to:

```text
1472 + 20 = 1492-byte IP packet
```

---

# 8. Hardware / ASIC Acceleration

FortiGate uses hardware acceleration/ASIC offloading to improve performance.

Normally:

```text
Packet
  ↓
FortiGate
  ↓
ASIC
  ↓
Fast forwarding
```

But during troubleshooting, ASIC offload can make packet behavior harder to observe.

---

## Disable ASIC Offload for a Policy

Example:

```bash
config firewall policy
    edit 1
        set auto-asic-offload disable
    next
end
```

Useful during:

* Packet capture
* Flow debugging
* Session troubleshooting
* Unexpected forwarding behavior
* Policy debugging
* ASIC-related troubleshooting

---

## ⚠️ Production Warning

Do not disable ASIC offload globally just because troubleshooting is easier.

Prefer:

```text
Specific Policy
       ↓
Temporary ASIC Disable
       ↓
Troubleshooting
       ↓
Restore acceleration
```

---

# 9. Session TTL

Session TTL determines how long a session can remain in the session table without being removed according to the relevant timeout behavior.

Default timeout values vary by protocol and FortiOS configuration.

Typical examples include:

```text
TCP
≈ 3600 seconds

UDP
≈ 180 seconds

ICMP
≈ 60–80 seconds
```

> ⚠️ Exact defaults are FortiOS/version/context dependent. Verify the actual value on the target FortiGate rather than relying on memorized numbers.

---

## Global Session TTL

Example:

```bash
config system session-ttl
    set default 3600
end
```

Think:

```text
Global Default
       │
       ▼
Session Timeout
```

---

# 10. Custom Session TTL

Some legacy applications maintain long-lived TCP sessions and do not reconnect correctly.

Examples:

```text
Legacy applications
Medical applications
Old database clients
Fixed-port applications
Persistent TCP applications
```

---

## Policy-Level TTL

```bash
config firewall policy
    edit 1
        set session-ttl 7200
    next
end
```

This gives that policy a different session timeout.

---

## Service-Level TTL

Example:

```bash
config firewall service custom
    edit "TCP-23"
        set tcp-portrange 23
        set session-ttl never
    next
end
```

This can be useful for a specific service rather than changing the behavior globally.

---

## Protocol/Port Timeout

Conceptually:

```bash
config system session-ttl
    ...
end
```

and custom service timeout settings can be used to tune particular traffic.

Always verify the exact syntax available in your FortiOS release.

---

# 11. Session TTL Troubleshooting

Before changing TTL values, inspect current sessions.

Conceptually:

```bash
diagnose sys session list
```

Look for:

```text
Source
Destination
Protocol
Port
Timeout
State
Policy ID
```

---

## Clear Existing Sessions

If an old session is using the previous timeout/configuration, changing the policy may not immediately affect that existing session.

Therefore:

```text
Change Configuration
        ↓
Existing Session?
        ↓
Clear / expire session if appropriate
        ↓
Create NEW session
        ↓
Test
```

> ⚠️ Be careful with session clearing in production. Clearing sessions can immediately disrupt active connections.

---

# 12. NGFW Modes

FortiGate supports different NGFW policy modes.

The two important concepts are:

```text
Policy-Based
Profile-Based
```

---

## High-Level Comparison

| Feature                     | Policy-Based                          | Profile-Based                            |
| --------------------------- | ------------------------------------- | ---------------------------------------- |
| Security model              | Application/security-centric policies | Traditional firewall policies + profiles |
| Granularity                 | Direct policy logic                   | Reusable security profiles               |
| Policy count                | Can consolidate logic                 | May require more policies                |
| Modern deployments          | Strong fit                            | Common in legacy/migration environments  |
| ZTNA/security integration   | Strong                                | Supported depending on feature           |
| Inspection mode flexibility | More feature-dependent                | Flow/Proxy commonly configurable         |
| IPsec Wizard                | Not supported in policy-based mode    | Available depending on version           |

> **Fortinet recommendation and feature behavior can change by FortiOS release. Verify against the target version.**

---

# 13. Policy-Based vs Profile-Based

## Profile-Based Model

Traditional model:

```text
Firewall Policy
      │
      ├── Source
      ├── Destination
      ├── Service
      ├── NAT
      │
      └── Security Profiles
             ├── IPS
             ├── AV
             ├── Web Filter
             ├── Application Control
             └── SSL Inspection
```

---

## Policy-Based Model

Security/application logic becomes more integrated into policy matching.

Conceptually:

```text
Traffic
   ↓
Application
   ↓
Security Policy
   ↓
Security Decision
```

This can simplify application-centric policy design.

---

# 14. Policy-Based Mode

Example VDOM configuration:

```bash
config vdom
    edit "vd-test"
        config system settings
            set ngfw-mode policy-based
        end
    next
end
```

> ⚠️ Changing NGFW mode is a major configuration change. Policies and feature visibility can change, and depending on FortiOS/version/context, existing policy configuration may be removed or transformed.

**Back up the configuration before changing NGFW mode.**

---

# 15. Enforce Default Application Port

One powerful application-control concept is enforcing the application's default port.

Example:

```bash
config firewall security-policy
    edit 1
        set enforce-default-app-port enable
    next
end
```

The objective is to prevent an application from simply using an unexpected port to bypass policy expectations.

Example:

```text
HTTP
Expected:
TCP/80

Traffic:
HTTP over TCP/8080
```

With appropriate application-port enforcement:

```text
Application = HTTP
Expected Port = 80
        │
        ▼
TCP/8080
        │
        ▼
Potentially Blocked
```

---

## Default Application Port as Service

The system can also use application default ports as service information:

```bash
config system settings
    set default-app-port-as-service enable
end
```

Then:

```bash
config firewall security-policy
    edit 1
        set enforce-default-app-port enable
    next
end
```

This allows application default-port information to participate earlier in policy matching.

---

# 16. Central NAT

Central NAT separates NAT logic from individual firewall policies.

Conceptually:

```text
Firewall Policy
      │
      │ Security decision
      ▼
Central NAT
      │
      ▼
NAT Decision
```

This can provide more granular and centralized control over address translation.

---

# 17. SSL Inspection & Authentication

Advanced NGFW deployments can combine:

```text
User Authentication
+
Interface
+
Application
+
SSL Inspection
+
Security Policy
```

Conceptually:

```text
Client
  │
  ▼
Authentication
  │
  ▼
SSL Inspection
  │
  ▼
Application Identification
  │
  ▼
Security Policy
```

This is particularly important when the application is hidden inside TLS encryption.

---

# 18. HTTP/SSH Policy Redirect

In profile-based deployments, HTTP and SSH traffic can be redirected toward proxy-based policy handling.

Example:

```bash
config firewall policy
    edit 1
        set http-policy-redirect enable
        set ssh-policy-redirect enable
    next
end
```

This generally requires the relevant policy/inspection configuration to use the appropriate **proxy-based** behavior.

Conceptually:

```text
HTTP
 │
 ▼
Proxy
 │
 ▼
Proxy Policy
```

and:

```text
SSH
 │
 ▼
SSH Proxy / Inspection
 │
 ▼
Policy
```

---

## Explicit Proxy

The explicit proxy must also be configured appropriately.

Example concept:

```text
Client
   │
   │ Proxy Request
   ▼
FortiGate Explicit Proxy
   │
   ▼
Proxy Policy
   │
   ▼
Internet
```

---

# 19. Land Attack Protection

A **Land attack** uses the same address as both source and destination.

Conceptually:

```text
Source IP
192.168.1.10

Destination IP
192.168.1.10

Source Port
X

Destination Port
X
```

This abnormal packet can cause systems to waste resources processing traffic that appears to originate from themselves.

---

## Enable Protection

```bash
config system settings
    set block-land-attack enable
end
```

Concept:

```text
Source = Destination
        │
        ▼
Land Attack
        │
        ▼
Block
```

---

# 20. Application Bandwidth Tracking

FortiGate can track application bandwidth usage.

This can support use cases such as:

```text
Application Visibility
SD-WAN Decisions
QoS
Bandwidth Analysis
```

Example:

```bash
config system settings
    set application-bandwidth-tracking enable
end
```

> ⚠️ The exact CLI spelling and availability can vary by FortiOS release. Verify with `?` or the relevant FortiOS CLI reference.

---

## Performance Consideration

Application tracking adds processing overhead.

Conceptually:

```text
Low-end / constrained appliance
        │
        ▼
Consider performance impact

High-end appliance
        │
        ▼
More suitable for intensive tracking
```

Do not enable features blindly on resource-constrained platforms.

---

# 21. Learning Mode

Learning mode is associated with policy analysis workflows.

A policy can operate in a learning-oriented workflow to observe traffic and generate policy recommendations.

Conceptually:

```text
Client Traffic
      │
      ▼
FortiGate
      │
      ▼
Logs
      │
      ▼
FortiAnalyzer
      │
      ▼
Policy Analyzer
      │
      ▼
Learned Traffic
      │
      ▼
Recommended Policies
```

---

## Policy Analyzer

A Policy Analyzer workflow can analyze traffic observed from managed FortiGates and use FortiAnalyzer logs to determine:

```text
Applications
Services
Destinations
Traffic Patterns
Security Events
```

The resulting policy recommendations can then be reviewed and deployed through the FortiManager workflow.

---

# 22. Learning Modes

Depending on the workflow/version, learned traffic can be interpreted using different approaches.

### Permissive

```text
Allowed learned traffic
        ↓
Use analyzed traffic/logs
        ↓
Generate broader policy recommendations
```

### Restricted

```text
Allowed learned traffic
        ↓
Focus more heavily on UTM/security logs
        ↓
Generate more restrictive recommendations
```

---

## Malicious Traffic

Learning workflows can also identify traffic that should be blocked.

Conceptually:

```text
Observed Traffic
      │
      ├── Legitimate
      │      ↓
      │   Learn / Recommend
      │
      └── Malicious
             ↓
           Block
```

---

# 23. Learning Mode Limitations

Important restrictions can apply when learning mode is enabled.

Common limitations include:

```text
✓ Source interfaces may require device identification

✗ Incoming interface = any
  may not be supported

✗ Outgoing interface = any
  may not be supported

✗ Internet Service may not be supported

✗ NAT46 may not be supported

✗ NAT64 may not be supported

✗ Users/groups may not be supported

✗ Some negate options may not be supported
```

Always verify limitations against the exact FortiOS/FortiManager version.

---

# 24. Learning Mode Mental Model

A useful way to remember it:

```text
                  LEARNING MODE
                       │
                       ▼
                  Observe Traffic
                       │
                       ▼
                     Logs
                       │
                       ▼
                 FortiAnalyzer
                       │
                       ▼
                  Policy Analyzer
                       │
                       ▼
              Learned Applications
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
        Legitimate             Malicious
             │                   │
             ▼                   ▼
      Policy Recommendation     Block
```

---

# 25. Advanced Troubleshooting Workflow

When dealing with MTU/MSS problems:

```text
                Application Problem
                       │
                       ▼
                 TCP Connection?
                       │
                       ▼
                  SYN / SYN-ACK
                       │
                       ▼
                  MSS Values
                       │
                       ▼
                   MTU Path
                       │
                       ▼
                    PMTUD
                       │
                       ▼
                 DF-bit Testing
                       │
                       ▼
                Tunnel Overhead
                       │
                       ▼
                 MSS Clamping
```

---

## Check TCP MSS

Capture the TCP three-way handshake:

```text
SYN
 └── MSS = ?

SYN/ACK
 └── MSS = ?
```

Compare:

```text
Client MSS
vs
Server MSS
vs
Expected Path MTU
```

---

# 26. MTU Troubleshooting Example

Suppose:

```text
Physical MTU = 1500
```

but traffic goes through:

```text
IPsec
```

The effective available payload becomes smaller.

Without adjustment:

```text
Client
  │
  │ MSS 1460
  ▼
FortiGate
  │
  │ IPsec overhead
  ▼
Packet too large
```

With MSS clamping:

```text
Client
  │
  │ MSS 1360
  ▼
FortiGate
  │
  │ IPsec overhead
  ▼
Packet fits
```

---

# 27. Session TTL — Design Rules

### Security Optimization

Shorter timeouts can reduce stale sessions for:

```text
Guest networks
High-risk services
Short-lived applications
Certain UDP/ICMP traffic
```

---

### Performance / Application Compatibility

Longer timeouts may be appropriate for:

```text
Long-lived TCP applications
Backup systems
Persistent database sessions
Certain VPN-related traffic
Legacy applications
```

---

### ⚠️ Do Not Use `never` Blindly

Example:

```bash
set session-ttl never
```

can keep sessions alive indefinitely according to the relevant timeout behavior.

That means:

```text
More persistent sessions
        ↓
More session-table resource consumption
        ↓
Potential scalability impact
```

Use it only for genuinely long-lived applications.

---

# 28. Session TTL Decision Tree

```text
Application Disconnects?
          │
          ▼
Check Session Timeout
          │
          ▼
Does application require
long-lived connection?
       │           │
      YES          NO
       │           │
       ▼           ▼
Tune TTL       Investigate
       │        application,
       │        routing,
       │        TCP state
       ▼
Retest with NEW session
```

---

# 29. ASIC + Session Troubleshooting

A powerful troubleshooting sequence:

```text
1. Identify policy
       ↓
2. Check session
       ↓
3. Check ASIC offload
       ↓
4. Temporarily disable ASIC
       ↓
5. Capture packet
       ↓
6. Compare ingress/egress
       ↓
7. Check NAT
       ↓
8. Check MTU/MSS
       ↓
9. Restore ASIC acceleration
```

---

# 30. Useful Commands

### Session Table

```bash
diagnose sys session list
```

---

### Policy / Traffic Debugging

Use the relevant:

```text
diagnose debug flow
```

workflow when investigating policy decisions and packet handling.

---

### MTU Testing

```bash
execute ping-options df-bit
```

Then test different packet sizes.

---

### Interface Configuration

```bash
show system interface
```

or inspect a specific interface.

---

### Session TTL

```bash
show system session-ttl
```

and inspect the relevant firewall policy/service.

---

# 31. Golden Mental Model

```text
                    FORTIGATE
                        │
        ┌───────────────┼────────────────┐
        │               │                │
        ▼               ▼                ▼
       MTU             MSS            Session TTL
        │               │                │
        │               │                │
 Interface/Path     TCP Payload      Session Lifetime
        │               │                │
        ▼               ▼                ▼
   Packet Size      TCP Segment       Session Table
        │               │                │
        └───────────────┼────────────────┘
                        ▼
                 Traffic Behavior
```

---

# 32. NSE High-Value Notes 🧠

### MTU

```text
MTU = Maximum IP packet size
```

### MSS

```text
MSS = Maximum TCP payload
```

### IPv4 TCP

```text
MSS ≈ MTU - 40
```

### Standard Ethernet

```text
MTU 1500
MSS 1460
```

### PPPoE

```text
MTU 1492
MSS 1452
```

### IPsec

```text
Effective MTU
=
Physical MTU
-
Encapsulation Overhead
```

### MSS Clamping

```text
Does NOT change interface MTU.
```

### ASIC

```text
Hardware acceleration
        ↓
Performance

Troubleshooting
        ↓
Temporarily disable on specific policy
```

### Session TTL

```text
Global default
        ↓
Policy/service override
```

### NGFW

```text
Policy-Based
    ↓
Application/security-centric

Profile-Based
    ↓
Traditional policy + security profiles
```

### Land Attack

```text
Source IP == Destination IP
        ↓
Potential Land Attack
```

### Learning Mode

```text
Traffic
 ↓
Logs
 ↓
FortiAnalyzer
 ↓
Policy Analyzer
 ↓
Policy Recommendation
```

---

# 33. One-Page Quick Reference

| Topic                  | Key Point                            | Example              |
| ---------------------- | ------------------------------------ | -------------------- |
| MTU                    | Maximum IP packet size               | `1500`               |
| MSS                    | TCP payload limit                    | `1460`               |
| MSS formula            | IPv4 TCP                             | `MTU - 40`           |
| MSS Clamp              | Changes TCP MSS                      | `1448`               |
| MTU Override           | Changes interface MTU                | `1480`               |
| PMTUD                  | Discovers path MTU                   | DF/ICMP mechanism    |
| ASIC                   | Hardware acceleration                | `auto-asic-offload`  |
| Session TTL            | Session lifetime                     | `3600 sec`           |
| TTL `never`            | Very long/unbounded timeout behavior | Use carefully        |
| Policy-Based           | Application/security-centric         | Modern NGFW design   |
| Profile-Based          | Policy + security profiles           | Traditional model    |
| Default App Port       | Enforce expected application port    | HTTP/80              |
| Central NAT            | Separate NAT logic                   | Centralized NAT      |
| Land Attack            | Source = Destination                 | Block                |
| App Bandwidth Tracking | Application usage tracking           | SD-WAN/QoS use cases |
| Learning Mode          | Observe → Analyze → Recommend        | Policy Analyzer      |

---

# 🔥 Final Takeaways

```text
MTU ≠ MSS

MTU
→ Interface/path packet size

MSS
→ TCP payload size

MSS Clamping
→ Fix TCP packet-size problems without necessarily
   changing interface MTU

ASIC Offload
→ Disable temporarily when deep troubleshooting
   requires software-path visibility

Session TTL
→ Tune only for specific application requirements

NGFW Policy-Based
→ Application/security-centric policy architecture

Profile-Based
→ Traditional firewall policy + reusable profiles

Land Attack
→ Source and destination address are identical

Learning Mode
→ Observe traffic → Analyze logs → Build policy recommendations
```

> **Production mindset:** Before changing MTU, MSS, TTL, ASIC offload, or NGFW mode, identify the exact problem first. These settings can solve very specific problems—but changing them globally can create a much larger one.
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

# FortiGate Application Control 

> **FortiGate Application Control | IPS Protocol Decoders | Application Signatures | Port Enforcement | QUIC | Quarantine | SCADA/ICS**
>
> Practical **FortiOS 7.x** reference for NSE preparation, troubleshooting, secure deployments, and real-world application visibility.

---

## 📚 Table of Contents

* [1. Application Control Overview](#1-application-control-overview)
* [2. How Application Control Detects Applications](#2-how-application-control-detects-applications)
* [3. Application Control Processing](#3-application-control-processing)
* [4. Application Control Profile](#4-application-control-profile)
* [5. Application Control Actions](#5-application-control-actions)
* [6. Application Attributes](#6-application-attributes)
* [7. Other vs Unknown Applications](#7-other-vs-unknown-applications)
* [8. Category vs Application Matching](#8-category-vs-application-matching)
* [9. Application Exclusions](#9-application-exclusions)
* [10. Port Enforcement](#10-port-enforcement)
* [11. Network Protocol Enforcement](#11-network-protocol-enforcement)
* [12. QUIC and Application Control](#12-quic-and-application-control)
* [13. Quarantine](#13-quarantine)
* [14. Deep Application Inspection](#14-deep-application-inspection)
* [15. SSL-Based Application Detection](#15-ssl-based-application-detection)
* [16. Application Control in Sandwich SSL Topology](#16-application-control-in-sandwich-ssl-topology)
* [17. SCADA and Industrial Protocol Decoders](#17-scada-and-industrial-protocol-decoders)
* [18. GTP Decoder](#18-gtp-decoder)
* [19. Modbus Decoder](#19-modbus-decoder)
* [20. DNP3 Decoder](#20-dnp3-decoder)
* [21. Multiple Application Parameters](#21-multiple-application-parameters)
* [22. Application Control CLI](#22-application-control-cli)
* [23. Troubleshooting](#23-troubleshooting)
* [24. Deployment Checklist](#24-deployment-checklist)
* [25. NSE Exam Traps](#25-nse-exam-traps)
* [26. Quick Revision Card](#26-quick-revision-card)

---

# 1. Application Control Overview

FortiGate **Application Control** identifies applications in network traffic and allows administrators to:

* Allow applications
* Block applications
* Monitor applications
* Log application activity
* Quarantine clients
* Filter by application category
* Filter by risk
* Filter by vendor
* Filter by technology
* Filter by behavior
* Filter by popularity
* Enforce default application ports
* Control unknown applications
* Control other applications

### Core Architecture

```text
                    Client
                       |
                       v
                Firewall Policy
                       |
                       v
              Application Control
                       |
             +---------+---------+
             |                   |
             v                   v
      IPS Protocol Decoder   Application DB
             |                   |
             +---------+---------+
                       |
                       v
                Application ID
                       |
                       v
             Matching / Filtering
                       |
          +------------+------------+
          |            |            |
         PASS         BLOCK      QUARANTINE
```

---

# 2. How Application Control Detects Applications

Application Control uses **IPS protocol decoders and application signatures** to identify applications.

The detection engine does not rely only on the destination port.

For example:

```text
Application:
    HTTPS application

Port:
    TCP/443

Detection:
    Application signature / protocol analysis
```

An application can potentially be detected even when it uses a **non-standard port**, provided the relevant decoder/signature can identify it.

### Key Concept

```text
Port Number
    ≠
Application Identity
```

FortiGate can inspect protocol behavior and application signatures rather than assuming:

```text
TCP/443 = HTTPS
TCP/80  = HTTP
```

---

# 3. Application Control Processing

A simplified processing model:

```text
Traffic
   |
   v
Firewall Policy
   |
   +--> Inspection Mode
   |
   +--> SSL Inspection
   |
   +--> Application Control
           |
           +--> Protocol Decoder
           |
           +--> Application Signature
           |
           +--> Category
           |
           +--> Risk
           |
           +--> Vendor
           |
           +--> Technology
           |
           +--> Behavior
           |
           +--> Popularity
           |
           v
       Policy Action
```

### Application Detection Can Use

* Protocol characteristics
* Application signatures
* IPS protocol decoders
* Application metadata
* Application categories
* Application behavior
* Application technology
* Vendor information
* Popularity
* Risk level

---

# 4. Application Control Profile

Create an Application Control profile under:

```text
Security Profiles
└── Application Control
```

Typical design:

```text
Application Control Profile
        |
        +── Allow trusted applications
        |
        +── Monitor unknown applications
        |
        +── Block risky categories
        |
        +── Quarantine malicious clients
        |
        +── Enforce application ports
```

### Example Policy Design

A common starting point:

```text
Default:
    Allow / Monitor

Specific categories:
    Block

High-risk applications:
    Block

Unknown applications:
    Block or Monitor

Other applications:
    Pass or Monitor
```

Attach the profile to the firewall policy:

```text
Policy & Objects
└── Firewall Policy
     ├── Inspection Mode
     ├── SSL/SSH Inspection
     └── Application Control
```

---

# 5. Application Control Actions

Typical actions include:

| Action       | Meaning                                                           |
| ------------ | ----------------------------------------------------------------- |
| `pass`       | Allow the application traffic                                     |
| `block`      | Block the application traffic                                     |
| `monitor`    | Allow while logging the application activity                      |
| `reset`      | Reset the connection                                              |
| `quarantine` | Block/quarantine the violating client for the configured duration |

### Action Concept

```text
Application detected
       |
       +---- PASS
       |
       +---- MONITOR
       |
       +---- BLOCK
       |
       +---- RESET
       |
       +---- QUARANTINE
```

---

# 6. Application Attributes

Application Control can match applications using multiple attributes.

## Protocol

Filters applications by application protocol.

```text
protocol
```

---

## Risk

Risk represents the potential impact associated with allowing the application.

| Risk | Level    |
| ---: | -------- |
|  `1` | Low      |
|  `2` | Elevated |
|  `3` | Medium   |
|  `4` | High     |
|  `5` | Critical |

Example:

```text
Risk 4–5
    ↓
High/Critical
    ↓
Consider blocking or strict monitoring
```

---

## Vendor

Filters applications according to the application vendor.

```text
vendor
```

---

## Technology

Application technology classification can include:

| Value | Technology       |
| ----: | ---------------- |
|   `0` | Network Protocol |
|   `1` | Browser-Based    |
|   `2` | Client-Server    |
|   `4` | Peer-to-Peer     |

---

## Behavior

Application behavior classification can include:

| Value | Behavior            |
| ----: | ------------------- |
|   `2` | Botnet              |
|   `3` | Evasive             |
|   `5` | Excessive Bandwidth |
|   `6` | Tunneling           |
|   `9` | Cloud               |

---

## Popularity

Application popularity ranges from:

```text
1 → Least popular
5 → Most popular
```

Example:

```text
1 2 3 4 5
│       │
Low    High
```

---

# 7. Other vs Unknown Applications

This distinction is an important Application Control concept.

### Other Application

FortiGate has an application signature/database match, but the application does not fall into the specific matching conditions configured by the rule.

```text
Signature exists
      |
      v
Does not match configured rule
      |
      v
OTHER
```

### Unknown Application

FortiGate cannot identify the application with an available application signature.

```text
No matching signature
       |
       v
UNKNOWN
```

### Important

```text
Known but not matched
        ↓
      OTHER

Not identified
        ↓
     UNKNOWN
```

---

# 8. Category vs Application Matching

You can match either:

```text
Application
```

or:

```text
Application Category
```

### Category-Based Rule

```bash
config application list
    edit "app-cont-test"

        config entries
            edit 1
                set category 2 6
                set action block
            next
        end

    next
end
```

When categories are specified, individual application IDs do not necessarily need to be listed in that same entry.

### Application-Based Rule

```bash
config application list
    edit "app-cont-test"

        config entries
            edit 1
                set application 15893 40568
                set action block
            next
        end

    next
end
```

---

# 9. Application Exclusions

Application exclusions are useful when you want to apply a broad category rule while allowing or handling specific applications differently.

### Concept

```text
Category
   |
   +---- Application A → Match
   |
   +---- Application B → Excluded
   |
   +---- Application C → Match
```

Example:

```bash
config application list
    edit "app-cont-test"

        set other-application-action pass
        set unknown-application-action pass

        config entries
            edit 1
                set action block
                set category 2 3 5 6 7 8 12 15 17 21 22 23 25 26 28 29 30 31
                set exclusion 15893 40568
            next
        end

    next
end
```

### Result

Conceptually:

```text
Category
   |
   +---- Application 15893 → Excluded
   |
   +---- Application 40568 → Excluded
   |
   +---- Other applications → Block
```

---

# 10. Port Enforcement

Application Control can enforce the expected/default ports for applications.

### Enable

```bash
config application list
    edit "app-cont-test"
        set enforce-default-app-port enable
    next
end
```

### Concept

Without port enforcement:

```text
Application
      |
      +---- TCP/443
      |
      +---- TCP/8443
      |
      +---- TCP/8080
```

With default-port enforcement:

```text
Application
      |
      +---- Expected Port → ALLOW
      |
      +---- Non-default Port → VIOLATION
```

### ⭐ Important

> **Application detection and port enforcement are different functions.**

Application Control can identify an application on a non-standard port, while port enforcement can be used to prevent applications from operating outside their expected/default ports.

---

# 11. Network Protocol Enforcement

Application Control can also enforce expected network services/ports.

Example:

```bash
config application list
    edit "app-cont-test"

        set enforce-default-app-port enable

        config default-network-services
            edit 2
                set port 53
                set services dns
                set violation-action monitor
            next
        end

    next
end
```

### Concept

```text
DNS
 |
 +---- UDP/53 → expected
 |
 +---- Other Port → violation
```

The exact enforcement behavior depends on the configured application/network-service rules.

---

# 12. QUIC and Application Control

QUIC uses:

```text
UDP
```

and HTTP/3 commonly uses:

```text
UDP/443
```

Therefore, application detection can encounter QUIC traffic that does not behave like traditional TCP-based HTTP.

### Application Control QUIC Option

Conceptually:

```text
QUIC
 |
 +---- Allow
 |
 +---- Block
```

### Example

```text
QUIC = Allow
    ↓
QUIC traffic can continue
    ↓
Application Control can process applicable traffic

QUIC = Block
    ↓
QUIC traffic is blocked
```

### Recommended Enterprise Approach

When web traffic uses HTTP/3:

```text
Firewall Policy
   |
   +── SSL/Deep Inspection
   |
   +── Web Filter
   |
   +── Application Control
   |
   +── QUIC policy
```

> ⚠️ Do not assume that blocking TCP/443 controls all HTTPS traffic. HTTP/3 commonly uses **UDP/443**.

---

# 13. Quarantine

When an Application Control rule uses quarantine:

```text
Violation
    ↓
Client identified
    ↓
Client quarantined
    ↓
Traffic blocked
    ↓
Quarantine timer expires
```

Example:

```bash
set action block
set quarantine attacker
set quarantine-expiry 5
set quarantine-log enable
```

### Default Example

```text
quarantine-expiry = 5 minutes
```

### Monitoring

Quarantined clients can be investigated through:

```text
FortiView
```

and:

```text
Quarantine Monitor
```

### Key Concept

```text
BLOCK
→ Current violating traffic is blocked

QUARANTINE
→ Client is placed into a quarantine state
```

---

# 14. Deep Application Inspection

Application Control can use deeper application inspection when required.

Example:

```bash
config application list
    edit "app-cont-test"
        set deep-app-inspection enable
    next
end
```

### Why?

Some applications cannot be reliably identified from a simple initial packet exchange.

```text
Initial Traffic
      |
      v
Basic Identification
      |
      v
More Application Data
      |
      v
Deep Application Inspection
      |
      v
More Accurate Identification
```

### Resource Consideration

Deeper inspection can increase:

* CPU usage
* Memory usage
* Inspection latency
* Overall processing load

This is especially important for high-volume environments and protocols such as QUIC/HTTP/3.

---

# 15. SSL-Based Application Detection

Some Application Control signatures are specifically marked for SSL-decrypted traffic.

A relevant option is:

```bash
set force-inclusion-ssl-di-sigs enable
```

Example:

```bash
config application list
    edit "app-cont-test"
        set force-inclusion-ssl-di-sigs enable
    next
end
```

### Concept

```text
Encrypted Traffic
       |
       v
SSL Inspection Device
       |
       v
Decrypted Traffic
       |
       v
FortiGate
       |
       v
SSL-Based Application Signatures
```

### `require_ssl_di`

Some signatures contain a predefined:

```text
require_ssl_di
```

tag.

These signatures are intended for applicable decrypted traffic.

> **Important:** These predefined signatures are not equivalent to ordinary user-customizable signatures.

---

# 16. Application Control in Sandwich SSL Topology

A **sandwich topology** can place FortiGate between SSL encryption/decryption devices.

```text
              SSL Decryption
                    |
                    v
            +---------------+
            |   FortiGate   |
            | Application   |
            |    Control    |
            +---------------+
                    |
                    v
              SSL Encryption
```

FortiGate receives already-decrypted traffic and can use applicable SSL-based application signatures.

### Enable

```bash
config application list
    edit "app-cont-test"
        set force-inclusion-ssl-di-sigs enable
    next
end
```

### Processing

```text
Encrypted
   ↓
Decryption Device
   ↓
Clear Traffic
   ↓
FortiGate
   ↓
Application Detection
   ↓
Security Decision
   ↓
Encryption Device
   ↓
Destination
```

---

# 17. SCADA and Industrial Protocol Decoders

Application Control and IPS protocol decoders are especially important for industrial environments.

Examples:

```text
GTP
Modbus
DNP3
```

These protocols have application-specific structures that can be inspected beyond simple IP/port matching.

### SCADA/ICS Model

```text
SCADA / ICS Traffic
        |
        v
IPS Protocol Decoder
        |
        +── Protocol Validation
        |
        +── Function Code Analysis
        |
        +── Anomaly Detection
        |
        +── Exploit Detection
        |
        v
Security Decision
```

---

# 18. GTP Decoder

## What is GTP?

**GPRS Tunneling Protocol (GTP)** is widely used in mobile carrier networks.

Common ports:

```text
UDP/2123 → GTP-C
UDP/2152 → GTP-U
```

### GTP Components

```text
GTP-C
→ Control plane

GTP-U
→ User plane
```

### FortiGate GTP Decoder Capabilities

Potential inspection areas include:

* GTP message validation
* Malformed header detection
* Tunnel flooding protection
* Fraud detection
* IMSI/IMEI validation
* Protocol anomaly detection
* Exploit detection

### IMSI

**International Mobile Subscriber Identity**

Conceptually:

```text
IMSI
 |
 +── MCC
 |    └── Mobile Country Code
 |
 +── MNC
 |    └── Mobile Network Code
 |
 └── MSIN
      └── Mobile Subscriber Identification Number
```

### IMEI

**International Mobile Equipment Identity**

Used to identify mobile equipment.

```text
IMEI
→ Device identity
```

---

# 19. Modbus Decoder

## What is Modbus?

Modbus is widely used in:

* PLCs
* Industrial automation
* SCADA
* Factory equipment

Common Modbus TCP port:

```text
TCP/502
```

### Modbus Types

```text
Modbus RTU
→ Serial

Modbus TCP
→ TCP/IP
```

### FortiGate Modbus Inspection

Potential security controls include:

* Illegal function codes
* Malformed packets
* Request flooding
* Unauthorized write commands
* Protocol anomalies

### Security Concept

```text
PLC
 |
 | Modbus Write
 v
FortiGate
 |
 +--> Function Code Validation
 |
 +--> Protocol Validation
 |
 +--> IPS Signature
 |
 v
ALLOW / BLOCK
```

> ⚠️ Industrial protocol inspection should be tested carefully because blocking legitimate write operations can directly affect production systems.

---

# 20. DNP3 Decoder

## What is DNP3?

**Distributed Network Protocol 3 (DNP3)** is commonly used in:

* Electrical utilities
* Water treatment
* SCADA
* Oil and gas
* Industrial control systems

Common TCP port:

```text
TCP/20000
```

DNP3 can also operate over serial communication.

### FortiGate DNP3 Inspection

Potential inspection capabilities include:

* Malformed packet detection
* Protocol violation detection
* Function-code analysis
* Read/write command inspection
* Authentication-related inspection
* Exploit/fuzzing detection

### Security Model

```text
DNP3 Traffic
     |
     v
DNP3 Decoder
     |
     +── Packet Validation
     |
     +── Function Analysis
     |
     +── Anomaly Detection
     |
     +── Exploit Detection
     |
     v
Security Action
```

---

# 21. Multiple Application Parameters

Some application signatures support multiple parameters.

This is particularly useful for:

* SCADA
* ICS
* Industrial protocols
* Function-code inspection

### Concept

```text
Application Signature
       |
       +── Parameter Group 1
       |
       +── Parameter Group 2
       |
       +── Parameter Group 3
```

Traffic can be flagged when it matches the configured parameter groups according to the signature/override logic.

### Example

A Modbus signature may inspect a specific function code and protocol parameter simultaneously.

```text
Modbus
 +
Function Code
 +
Parameter
 ↓
Signature Match
 ↓
Action
```

---

# 22. Application Control CLI

## Basic Application Control Profile

```bash
config application list
    edit "app-cont-test"

        set extended-log enable

        set other-application-action pass
        set other-application-log enable

        set unknown-application-action block
        set unknown-application-log enable

        set enforce-default-app-port enable

        set force-inclusion-ssl-di-sigs disable

        set deep-app-inspection enable

        set options allow-dns

        set control-default-network-services disable

    next
end
```

---

## Category-Based Rule

```bash
config application list
    edit "app-cont-test"

        config entries
            edit 1
                set category 2 6
                set protocols all
                set vendor all
                set technology all
                set behavior all
                set popularity 1 2 3 4 5
                set action block
                set log enable
                set log-packet disable
                set session-ttl 0
                set quarantine attacker
                set quarantine-expiry 5
                set quarantine-log enable
            next
        end

    next
end
```

---

# 23. Troubleshooting

## Check FortiGuard Status

```bash
get system fortiguard
```

Use this to verify:

* FortiGuard connectivity
* Subscription/license status
* Service availability

> Application signatures and related FortiGuard services depend on the platform, FortiOS version, and subscription/service status.

---

## WAD Diagnostic

```bash
diagnose test app wad 1000
```

Useful for inspecting WAD worker/process information.

Conceptually:

```text
WAD
 |
 +── Web Proxy
 |
 +── Web Filtering
 |
 +── Video Filtering
 |
 +── Application-related processing
```

---

## WAD Debug

```bash
diagnose wad debug enable level verbose
diagnose wad debug enable category video
```

For Application Control troubleshooting, combine WAD/flow diagnostics with application-control and IPS diagnostics appropriate to the FortiOS release.

---

# 24. Deployment Checklist

## Firewall Policy

* [ ] Correct firewall policy selected.
* [ ] Correct inspection mode configured.
* [ ] Application Control profile attached.
* [ ] SSL inspection configured where required.
* [ ] Deep Inspection CA deployed to clients where required.
* [ ] Application logging enabled where necessary.

## Application Detection

* [ ] FortiGuard connectivity verified.
* [ ] Application signatures updated.
* [ ] Application categories reviewed.
* [ ] Risk levels reviewed.
* [ ] Unknown applications policy defined.
* [ ] Other applications policy defined.

## Port Enforcement

* [ ] Default application ports reviewed.
* [ ] Non-standard application ports tested.
* [ ] Network protocol enforcement reviewed.
* [ ] Legitimate exceptions documented.

## QUIC / HTTP/3

* [ ] UDP/443 behavior reviewed.
* [ ] QUIC policy defined.
* [ ] HTTP/3 applications tested.
* [ ] Application Control tested with QUIC traffic.
* [ ] Web Filter compatibility tested.
* [ ] Deep Inspection impact evaluated.

## Quarantine

* [ ] Quarantine action reviewed.
* [ ] Quarantine duration configured.
* [ ] Quarantine logging enabled.
* [ ] FortiView monitoring tested.

## Industrial Networks

* [ ] SCADA protocols identified.
* [ ] Modbus traffic tested.
* [ ] DNP3 traffic tested.
* [ ] GTP requirements evaluated where applicable.
* [ ] Industrial signatures reviewed.
* [ ] False positives tested before enforcement.

---

# 25. NSE Exam Traps

## 🧠 Trap #1 — Port ≠ Application

```text
TCP/443
    ≠
Always HTTPS

TCP/80
    ≠
Always HTTP
```

Application Control can use protocol/application signatures to identify applications.

---

## 🧠 Trap #2 — Other vs Unknown

```text
Known signature
but not matching configured rule
        ↓
      OTHER

No application identification
        ↓
     UNKNOWN
```

---

## 🧠 Trap #3 — Category vs Application

```text
Category Rule
→ Multiple applications

Application Rule
→ Specific application IDs
```

---

## 🧠 Trap #4 — Port Enforcement

```text
Application Detection
        ≠
Port Enforcement
```

Detection identifies the application.

Port enforcement controls whether the application is allowed to operate on expected/default ports.

---

## 🧠 Trap #5 — QUIC

```text
HTTP/3
  ↓
QUIC
  ↓
UDP
```

Do not assume that TCP/443 inspection covers all modern web traffic.

---

## 🧠 Trap #6 — Quarantine

```text
BLOCK
→ Block matching traffic

QUARANTINE
→ Put violating client into quarantine
```

---

## 🧠 Trap #7 — Deep Inspection

```text
Encrypted Application
        ↓
SSL Deep Inspection
        ↓
Decrypted Payload
        ↓
Application Inspection
```

Without decryption, many encrypted application details may not be visible to the same extent.

---

## 🧠 Trap #8 — SSL DI Signatures

```text
require_ssl_di
        ↓
Predefined SSL-decryption-related signature
```

`force-inclusion-ssl-di-sigs` controls whether applicable predefined SSL-based signatures are included for decrypted traffic.

---

## 🧠 Trap #9 — SCADA

Do not rely only on:

```text
IP + Port
```

Industrial protocol decoders can inspect protocol-specific behavior.

```text
Modbus
→ Function Codes

DNP3
→ Function / Protocol Parameters

GTP
→ Tunnel / Subscriber Parameters
```

---

## 🧠 Trap #10 — Unknown Applications

If a custom or previously unidentified tool cannot be recognized by the available application signatures:

```text
No Signature Match
       ↓
UNKNOWN
       ↓
Can be blocked using:
unknown-application-action
```

Example:

```bash
set unknown-application-action block
```

---

# 26. Quick Revision Card

```text
APPLICATION CONTROL
│
├── DETECTION
│   ├── IPS Protocol Decoder
│   ├── Application Signature
│   └── Application Database
│
├── MATCHING
│   ├── Application
│   ├── Category
│   ├── Protocol
│   ├── Risk
│   ├── Vendor
│   ├── Technology
│   ├── Behavior
│   └── Popularity
│
├── ACTION
│   ├── Pass
│   ├── Monitor
│   ├── Block
│   ├── Reset
│   └── Quarantine
│
├── SPECIAL FEATURES
│   ├── Deep App Inspection
│   ├── Port Enforcement
│   ├── Network Protocol Enforcement
│   ├── QUIC Control
│   └── SSL DI Signatures
│
├── APPLICATION STATES
│   ├── Known
│   ├── Other
│   └── Unknown
│
└── INDUSTRIAL DECODERS
    ├── GTP
    ├── Modbus
    └── DNP3
```

---

## ⚡ 60-Second NSE Memory Map

```text
APPLICATION CONTROL
        |
        v
"WHO IS THIS APPLICATION?"
        |
        +── IPS Decoder
        +── Signature
        +── Protocol
        |
        v
"WHAT TYPE IS IT?"
        |
        +── Category
        +── Risk
        +── Vendor
        +── Technology
        +── Behavior
        +── Popularity
        |
        v
"WHAT SHOULD FORTIGATE DO?"
        |
        +── Pass
        +── Monitor
        +── Block
        +── Reset
        +── Quarantine
        |
        v
"IS THE TRAFFIC USING THE RIGHT PORT?"
        |
        +── Default App Port Enforcement
        +── Network Protocol Enforcement
        |
        v
"IS IT ENCRYPTED?"
        |
        +── SSL Inspection
        +── Deep Application Inspection
        +── SSL-DI Signatures
        |
        v
"IS IT MODERN WEB TRAFFIC?"
        |
        +── QUIC
        +── HTTP/3
        +── UDP/443
        |
        v
"IS IT INDUSTRIAL?"
        |
        +── GTP
        +── Modbus
        +── DNP3
```

---

## 🔥 One-Line NSE Memory Aid

> **Application Control identifies applications using protocol decoders and signatures, classifies them by attributes such as category/risk/technology, and then applies actions such as pass, monitor, block, reset, or quarantine—while port enforcement, SSL inspection, QUIC handling, and industrial protocol decoders provide additional control layers.**

---

## 🔑 Most Important Commands

```bash
# FortiGuard status
get system fortiguard

# Application Control profile
config application list
    edit "app-cont-test"

# Unknown applications
set unknown-application-action block

# Other applications
set other-application-action pass

# Application port enforcement
set enforce-default-app-port enable

# Deep application inspection
set deep-app-inspection enable

# SSL DI signatures
set force-inclusion-ssl-di-sigs enable

# Application categories / signatures
config entries

# WAD diagnostic
diagnose test app wad 1000

# WAD verbose debugging
diagnose wad debug enable level verbose
```

---

## 🏷️ Suggested GitHub  Metadata

**Title:**

```text
FortiGate Application Control   — FortiOS 7.x | NSE
```

**Suggested filename:**

```text
fortigate-application-control-.md
```

**Suggested repository topics:**

```text
fortigate
fortios
fortinet
application-control
ips
network-security
cybersecurity
nse
nse4
nse7
scada
ics-security
modbus
dnp3
gtp
quic
http3
firewall
```

**keywords:**

```text
FortiGate Application Control
FortiOS Application Control
FortiGate IPS protocol decoder
FortiGate application signatures
FortiGate application control quarantine
FortiGate application port enforcement
FortiGate QUIC application control
FortiGate HTTP/3
FortiGate Modbus inspection
FortiGate DNP3 inspection
FortiGate GTP decoder
FortiOS NSE Application Control
```

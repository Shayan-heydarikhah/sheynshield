# 🔐 FortiGate TACACS+ & 802.1X — Security Engineering Checklist

> **SheynShield | Security & Design Knowledge Base**
> **Platform:** FortiGate / FortiOS
> **Level:** NSE4 → NSE7
> **Focus:** TACACS+ AAA • RADIUS • 802.1X • EAP • PEAP • EAP-TLS • NPS • Authentication • Authorization

---

## 📌 Table of Contents

* [1. Architecture Checklist](#1-architecture-checklist)
* [2. TACACS+ Fundamentals](#2-tacacs-fundamentals)
* [3. TACACS+ Server Configuration](#3-tacacs-server-configuration)
* [4. TACACS+ Administrator Authentication](#4-tacacs-administrator-authentication)
* [5. TACACS+ Authorization](#5-tacacs-authorization)
* [6. TACACS+ Security Hardening](#6-tacacs-security-hardening)
* [7. 802.1X Fundamentals](#7-8021x-fundamentals)
* [8. EAP Authentication Methods](#8-eap-authentication-methods)
* [9. FortiGate as 802.1X Supplicant](#9-fortigate-as-8021x-supplicant)
* [10. RADIUS & NPS Validation](#10-radius--nps-validation)
* [11. Certificate / PKI Validation](#11-certificate--pki-validation)
* [12. PEAP Checklist](#12-peap-checklist)
* [13. EAP-TLS Checklist](#13-eap-tls-checklist)
* [14. Troubleshooting Checklist](#14-troubleshooting-checklist)
* [15. NSE Gotchas](#15-nse-gotchas)
* [16. Production Checklist](#16-production-checklist)
* [17. Lab Validation](#17-lab-validation)
* [18. Quick CLI Reference](#18-quick-cli-reference)
* [19. One-Page Memory Map](#19-one-page-memory-map)
* [20. Expert Takeaway](#20-expert-takeaway)

---

# 1. Architecture Checklist

## Enterprise Identity Architecture

```text
                         Active Directory
                                │
                ┌───────────────┴───────────────┐
                │                               │
                ▼                               ▼
          TACACS+ Server                   NPS / RADIUS
                │                               │
                │                               ├── 802.1X
                │                               ├── PEAP
                │                               └── EAP-TLS
                │
                ▼
           FortiGate
                │
        ┌───────┴────────┐
        │                │
        ▼                ▼
   Admin AAA        Network Access
        │                │
   TACACS+             802.1X
```

### Architecture Validation

* [ ] Identity source is clearly defined
* [ ] Active Directory integration is validated
* [ ] TACACS+ is used for administrative AAA where appropriate
* [ ] RADIUS/NPS is used for network authentication where appropriate
* [ ] 802.1X role is clearly identified
* [ ] EAP authentication method is selected
* [ ] Certificate requirements are documented
* [ ] Authentication and authorization paths are documented
* [ ] Redundancy is designed for critical authentication services

---

# 2. TACACS+ Fundamentals

## Core Concept

**TACACS+ = centralized Authentication, Authorization and Accounting (AAA).**

Typical FortiGate use:

```text
Administrator
      │
      ▼
  FortiGate
      │
   TCP/49
      ▼
 TACACS+ Server
      │
      ▼
 Identity Backend
```

### TACACS+ Checklist

* [ ] Authentication requirement identified
* [ ] Authorization requirement identified
* [ ] Accounting requirement identified
* [ ] TACACS+ server deployed
* [ ] FortiGate configured as TACACS+ client
* [ ] TCP/49 permitted
* [ ] Shared secret configured
* [ ] Remote groups defined
* [ ] Administrative profiles mapped

## TACACS+ vs RADIUS

| Feature           | TACACS+               | RADIUS               |
| ----------------- | --------------------- | -------------------- |
| Primary use       | Device administration | Network/user access  |
| Transport         | TCP                   | UDP                  |
| Default port      | `49`                  | `1812/1813`          |
| Authentication    | ✅                     | ✅                    |
| Authorization     | Strongly separated    | Attribute-based      |
| Accounting        | ✅                     | ✅                    |
| FortiGate example | Admin login           | VPN / Wi-Fi / 802.1X |

### NSE Memory

```text
TACACS+ → Device Administration
RADIUS  → Network/User Authentication
```

---

# 3. TACACS+ Server Configuration

## FortiGate CLI

```bash
config user tacacs+
    edit "tac-1"
        set server 192.168.20.200
        set key <TACACS_SECRET>
        set authorization enable
        set authen-type auto
    next
end
```

### Configuration Checklist

* [ ] TACACS+ object created
* [ ] Correct server IP configured
* [ ] Correct shared secret configured
* [ ] Authorization enabled when required
* [ ] Authentication type matches server capability
* [ ] TCP/49 is reachable
* [ ] Primary TACACS+ server tested
* [ ] Secondary server configured where required

> 🔐 Never commit real TACACS+ shared secrets to GitHub.

---

# 4. TACACS+ Administrator Authentication

## Administrator Flow

```text
Administrator
      │
      │ Username + Password
      ▼
  FortiGate
      │
      │ TACACS+
      ▼
TACACS+ Server
      │
      ├── Authentication
      │
      └── Authorization
              │
              ▼
       FortiGate Admin Profile
```

### Administrator Checklist

* [ ] Remote authentication enabled
* [ ] TACACS+ group created
* [ ] Administrator references correct remote group
* [ ] Correct administrative profile assigned
* [ ] Login tested
* [ ] Failed login behavior tested
* [ ] Local emergency administrator retained
* [ ] Break-glass access secured
* [ ] Remote authentication failure does not lock out emergency access

---

# 5. TACACS+ Authorization

## Authentication vs Authorization

```text
Authentication
      ↓
"Who are you?"

Authorization
      ↓
"What are you allowed to do?"
```

### Group Mapping

```text
TACACS+ Group
      │
      ├── Network-Admins
      │       ↓
      │   Super_Admin
      │
      └── Network-Operators
              ↓
          Read_Only
```

### Authorization Checklist

* [ ] TACACS+ authorization enabled
* [ ] Remote groups returned correctly
* [ ] Remote groups mapped to FortiGate groups
* [ ] Administrative profiles mapped correctly
* [ ] Least privilege applied
* [ ] `Super_Admin` restricted to required administrators
* [ ] Read-only operators use restricted profiles
* [ ] Authorization tested independently from authentication

### Critical NSE Concept

```text
Authentication SUCCESS
        ≠
Authorization SUCCESS
```

A user may successfully authenticate but receive the wrong administrative permissions.

---

# 6. TACACS+ Security Hardening

* [ ] Strong TACACS+ shared secret configured
* [ ] TACACS+ communication restricted to trusted clients
* [ ] Management source IP controlled
* [ ] Redundant TACACS+ servers considered
* [ ] Least-privilege administrator profiles configured
* [ ] Emergency local account protected
* [ ] Administrator login auditing enabled
* [ ] Failed authentication monitored
* [ ] Shared secrets excluded from Git repositories
* [ ] Passwords excluded from documentation
* [ ] Production credentials replaced with placeholders in examples

### Recommended Documentation Format

```text
set key <TACACS_SECRET>
```

Never:

```text
set key MyProductionSecret123
```

---

# 7. 802.1X Fundamentals

**802.1X provides port-based network access control.**

## Three Primary Roles

```text
┌─────────────────┐
│   Supplicant    │
│     Client      │
└────────┬────────┘
         │
         │ EAP
         ▼
┌─────────────────┐
│  Authenticator  │
│ Network Access  │
└────────┬────────┘
         │
         │ RADIUS
         ▼
┌─────────────────┐
│ Authentication  │
│     Server      │
│   NPS / RADIUS  │
└─────────────────┘
```

### Role Checklist

* [ ] Supplicant identified
* [ ] Authenticator identified
* [ ] Authentication server identified
* [ ] RADIUS communication validated
* [ ] EAP method selected
* [ ] Certificate requirements documented

### Important

802.1X is **not itself the credential mechanism**.

It provides the access-control framework.

```text
802.1X
   │
   └── EAP
        ├── PEAP
        ├── EAP-TLS
        └── Other EAP methods
```

---

# 8. EAP Authentication Methods

## PEAP

```text
Client
  │
  │ TLS tunnel
  ▼
RADIUS / NPS
  │
  └── Inner Authentication
```

Typical model:

```text
Username + Password
        ↓
      PEAP
        ↓
       TLS
        ↓
   RADIUS / NPS
```

### PEAP Checklist

* [ ] Server certificate deployed
* [ ] Client trusts server certificate chain
* [ ] Inner authentication method configured
* [ ] Username format validated
* [ ] Password authentication validated
* [ ] NPS policy allows selected PEAP method
* [ ] FortiGate EAP method matches NPS

---

## EAP-TLS

```text
Client Certificate
       │
       ▼
      TLS
       │
       ▼
RADIUS / NPS
       │
       ▼
Certificate Validation
```

### EAP-TLS Checklist

* [ ] Client certificate issued
* [ ] Client private key available
* [ ] RADIUS server certificate valid
* [ ] CA trust established
* [ ] Certificate chain complete
* [ ] SAN configured correctly
* [ ] EKU appropriate
* [ ] NPS policy allows EAP-TLS
* [ ] Client validates RADIUS certificate
* [ ] Certificate revocation requirements considered

---

# 9. FortiGate as 802.1X Supplicant

## RADIUS Configuration

```text
User & Authentication
└── RADIUS Servers
```

Example:

```text
Name:
    rad-1

Server:
    192.168.20.200

NAS IP:
    192.168.20.254

Authentication:
    MSCHAPv2

Secret:
    <RADIUS_SECRET>
```

### Validation

* [ ] RADIUS server IP correct
* [ ] NAS IP correct
* [ ] Shared secret correct
* [ ] Authentication method supported
* [ ] UDP/1812 reachable
* [ ] RADIUS server recognizes FortiGate as NAS

---

## Interface EAP Supplicant

Example:

```bash
config system interface
    edit "port3"
        set eap-supplicant enable
        set eap-method peap
        set eap-identity <EAP_IDENTITY>
        set eap-password <EAP_PASSWORD>
        set eap-ca-cert <CA_CERTIFICATE>
    next
end
```

### Interface Checklist

* [ ] Correct interface selected
* [ ] `eap-supplicant` enabled
* [ ] EAP method configured
* [ ] EAP identity configured
* [ ] EAP credential configured where required
* [ ] CA certificate configured where required
* [ ] Switch/authenticator configuration matches
* [ ] EAP authentication tested

> 🔐 Never publish real EAP credentials or private certificates.

---

# 10. RADIUS & NPS Validation

## Standard Ports

```text
UDP/1812 → Authentication
UDP/1813 → Accounting
```

### Connectivity Checklist

* [ ] UDP/1812 reachable
* [ ] UDP/1813 reachable when accounting is required
* [ ] Routing is correct
* [ ] Firewall rules permit traffic
* [ ] RADIUS shared secret matches
* [ ] NAS IP matches NPS configuration
* [ ] NPS client definition is correct
* [ ] NPS network policy is correct
* [ ] NPS authentication method matches FortiGate
* [ ] AD authentication succeeds

---

# 11. Certificate / PKI Validation

Certificate-based authentication introduces another dependency chain.

```text
Client Certificate
       │
       ▼
Certificate Chain
       │
       ▼
Trusted CA
       │
       ▼
RADIUS / NPS
       │
       ▼
Identity Validation
```

### Certificate Checklist

* [ ] Certificate is present
* [ ] Certificate is not expired
* [ ] Certificate is issued by trusted CA
* [ ] Certificate chain is complete
* [ ] CA is trusted
* [ ] SAN is correct
* [ ] EKU is appropriate
* [ ] Subject is correct where required
* [ ] Private key is available
* [ ] RADIUS server certificate is valid
* [ ] Client trusts RADIUS server CA
* [ ] Certificate revocation requirements are understood

### NSE Reminder

```text
RADIUS reachable
        ≠
EAP-TLS working
```

Certificate validation can fail even when RADIUS connectivity is perfect.

---

# 12. PEAP Checklist

## Client

* [ ] EAP method = PEAP
* [ ] Username correct
* [ ] Password correct
* [ ] Server certificate trusted
* [ ] CA chain trusted

## FortiGate

* [ ] RADIUS server configured
* [ ] Shared secret correct
* [ ] EAP supplicant enabled
* [ ] PEAP selected
* [ ] EAP identity configured
* [ ] Required CA certificate configured

## NPS

* [ ] NPS client configured
* [ ] Network Policy enabled
* [ ] PEAP enabled
* [ ] Inner authentication method configured
* [ ] AD credentials valid
* [ ] Certificate binding correct

---

# 13. EAP-TLS Checklist

## PKI

* [ ] Root CA available
* [ ] Issuing CA available
* [ ] Client certificate issued
* [ ] RADIUS server certificate issued
* [ ] Private keys available
* [ ] Certificate chain complete
* [ ] SAN correct
* [ ] EKU correct
* [ ] Validity dates checked

## Trust

* [ ] Client trusts issuing CA
* [ ] RADIUS trusts required client CA
* [ ] Client validates RADIUS certificate
* [ ] FortiGate trusts required CA where applicable

## NPS

* [ ] EAP-TLS enabled
* [ ] Correct certificate selected
* [ ] NPS policy matches client
* [ ] AD identity mapping validated

---

# 14. Troubleshooting Checklist

## Layer 1 — Connectivity

* [ ] Interface is up
* [ ] Link is stable
* [ ] Correct VLAN configured
* [ ] Correct switch port configuration
* [ ] Routing is correct
* [ ] Required firewall rules exist

---

## Layer 2 — 802.1X

* [ ] Supplicant enabled
* [ ] Authenticator configured
* [ ] EAP exchange begins
* [ ] EAP method matches
* [ ] Identity is sent correctly

---

## Layer 3 — RADIUS

* [ ] UDP/1812 reachable
* [ ] Shared secret correct
* [ ] NAS IP correct
* [ ] NPS client configured
* [ ] RADIUS Access-Request received
* [ ] Access-Accept generated

---

## Layer 4 — Authentication

* [ ] Username correct
* [ ] Password correct for PEAP
* [ ] Certificate valid for EAP-TLS
* [ ] CA trusted
* [ ] Certificate chain valid
* [ ] SAN/EKU correct
* [ ] NPS policy allows authentication

---

## Layer 5 — Authorization

* [ ] Correct group returned
* [ ] Remote group mapped
* [ ] Correct FortiGate profile assigned
* [ ] Firewall policy references correct identity/group
* [ ] User receives intended access

---

# 15. NSE Gotchas

## 🔥 Gotcha 1 — TACACS+ Uses TCP/49

```text
TACACS+ → TCP/49
```

Not:

```text
UDP/1812
```

---

## 🔥 Gotcha 2 — RADIUS Uses UDP

```text
1812 → Authentication
1813 → Accounting
```

---

## 🔥 Gotcha 3 — Authentication ≠ Authorization

```text
Authentication
      ↓
Identity verified

Authorization
      ↓
Permissions assigned
```

---

## 🔥 Gotcha 4 — 802.1X ≠ EAP

```text
802.1X
   ↓
Access-control framework

EAP
   ↓
Authentication framework
```

---

## 🔥 Gotcha 5 — PEAP ≠ EAP-TLS

```text
PEAP
 └── TLS tunnel
      └── Inner authentication

EAP-TLS
 └── Client certificate
      └── Certificate-based authentication
```

---

## 🔥 Gotcha 6 — FortiGate Role Matters

Always determine whether FortiGate is acting as:

```text
Supplicant
```

or:

```text
Authenticator
```

The troubleshooting path is different.

---

## 🔥 Gotcha 7 — RADIUS Success Does Not Guarantee Policy Success

```text
RADIUS Authentication
        ↓
       SUCCESS
        ↓
Group / Attribute Mapping
        ↓
Firewall Policy
        ↓
       ACCESS
```

Authentication can succeed while policy matching fails.

---

# 16. Production Checklist

## TACACS+

* [ ] Strong shared secret configured
* [ ] TCP/49 restricted
* [ ] Trusted TACACS+ servers only
* [ ] Redundant servers configured where appropriate
* [ ] Remote groups mapped
* [ ] Least-privilege profiles configured
* [ ] `Super_Admin` restricted
* [ ] Emergency local administrator protected
* [ ] Administrator authentication logging enabled

## 802.1X

* [ ] Correct supplicant/authenticator roles defined
* [ ] RADIUS server redundant where required
* [ ] EAP method documented
* [ ] Certificates managed centrally
* [ ] CA trust validated
* [ ] SAN/EKU validated
* [ ] Dedicated authentication identities used
* [ ] EAP credentials protected
* [ ] Authentication failures monitored

## Secrets

* [ ] No RADIUS shared secrets in GitHub
* [ ] No TACACS+ secrets in GitHub
* [ ] No passwords in examples
* [ ] No OTP seeds published
* [ ] No private keys published
* [ ] Production IPs sanitized when necessary

---

# 17. Lab Validation

## TACACS+ Lab

```text
[ ] TACACS+ server = 192.168.20.200
[ ] TCP/49 reachable
[ ] Shared secret configured
[ ] Authentication succeeds
[ ] Remote group returned
[ ] FortiGate group matches
[ ] Administrator login succeeds
[ ] Correct admin profile assigned
[ ] Authorization verified
```

## 802.1X Lab

```text
[ ] RADIUS server configured
[ ] NAS IP configured
[ ] UDP/1812 reachable
[ ] Shared secret matches
[ ] EAP supplicant enabled
[ ] PEAP tested
[ ] EAP-TLS tested if required
[ ] CA certificate installed
[ ] EAP identity configured
[ ] EAP authentication succeeds
[ ] NPS policy verified
[ ] Authentication failure tested
```

---

# 18. Quick CLI Reference

## TACACS+

```bash
config user tacacs+
    edit "tac-1"
        set server 192.168.20.200
        set key <TACACS_SECRET>
        set authorization enable
        set authen-type auto
    next
end
```

---

## 802.1X / EAP Supplicant

```bash
config system interface
    edit "port3"
        set eap-supplicant enable
        set eap-method peap
        set eap-identity <EAP_IDENTITY>
        set eap-password <EAP_PASSWORD>
        set eap-ca-cert <CA_CERTIFICATE>
    next
end
```

---

## EAP Diagnostics

```bash
diagnose test app eap_supp 2
```

> ⚠️ Diagnostic syntax and available test options can vary by FortiOS release.

---

# 19. One-Page Memory Map

```text
                 FORTIGATE AUTHENTICATION
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
      TACACS+            RADIUS            802.1X
          │                │                │
          │                │                ▼
          │                │               EAP
          │                │                │
          │                │         ┌──────┴──────┐
          │                │         ▼             ▼
          │                │       PEAP          EAP-TLS
          │                │         │             │
          │                │      TLS +         Client
          │                │      Inner          Cert
          │                │      Auth
          │                │
          ▼                ▼                ▼
    Device AAA        User AAA         Port Access
          │                │                │
          ▼                ▼                ▼
   Admin Login      Authentication    Network Admission
   Authorization    Accounting
```

---

# 20. Expert Takeaway

## The NSE Mental Model

```text
TACACS+
   │
   └── Device Administration
          ├── Authentication
          └── Authorization
                  │
                  ▼
             Admin Profile


RADIUS
   │
   ├── Authentication
   ├── Accounting
   ├── AVP / VSA
   ├── Session Information
   └── Dynamic Authorization


802.1X
   │
   └── Port-Based Network Access
            │
            └── EAP
                 ├── PEAP
                 │    └── Username / Password inside TLS
                 │
                 └── EAP-TLS
                      └── Certificate-Based Authentication
```

### 🔥 Five Things to Remember

* [ ] **TACACS+ → FortiGate administrator AAA**
* [ ] **RADIUS → centralized user/network authentication and AAA attributes**
* [ ] **802.1X → port-based network access control**
* [ ] **PEAP → TLS tunnel + inner authentication**
* [ ] **EAP-TLS → certificate-based client authentication**

### 🔥 Troubleshooting Golden Chain

```text
Connectivity
     ↓
802.1X / EAP
     ↓
RADIUS
     ↓
Authentication
     ↓
Authorization
     ↓
Group / Attribute Mapping
     ↓
Firewall Policy
     ↓
Access
```

> **Expert Rule:** When authentication succeeds but access behavior is wrong, don't stop at the authentication server. Trace the complete **identity → attribute → group → policy → session** chain.

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

`FortiGate` • `TACACS+` • `RADIUS` • `802.1X` • `EAP` • `PEAP` • `EAP-TLS` • `NPS` • `AAA` • `Network Access Control` • `Network Security` • `Fortinet` • `NSE4` • `NSE7`

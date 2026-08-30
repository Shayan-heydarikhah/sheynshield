# 🔐 FortiGate FSSO Checklist

> **FortiOS FSSO (Fortinet Single Sign-On) | Active Directory | FSSO Collector Agent | Identity-Based Firewall Policy | Troubleshooting**
>
> Practical **FortiGate FSSO checklist** for configuration, Active Directory integration, user/group mapping, identity-based policies, verification, troubleshooting, and NSE4/NSE7 exam preparation.

---

## 📋 Table of Contents

* [1. FSSO Fundamentals](#1-fsso-fundamentals)
* [2. FSSO Architecture](#2-fsso-architecture)
* [3. FSSO Deployment Checklist](#3-fsso-deployment-checklist)
* [4. Active Directory Prerequisites](#4-active-directory-prerequisites)
* [5. FSSO Collector Agent](#5-fsso-collector-agent)
* [6. FortiGate FSSO Configuration](#6-fortigate-fsso-configuration)
* [7. FSSO Groups](#7-fsso-groups)
* [8. Identity-Based Firewall Policy](#8-identity-based-firewall-policy)
* [9. FSSO Identity Verification](#9-fsso-identity-verification)
* [10. FSSO Troubleshooting](#10-fsso-troubleshooting)
* [11. Troubleshooting Decision Tree](#11-troubleshooting-decision-tree)
* [12. FSSO Performance Checklist](#12-fsso-performance-checklist)
* [13. Security Hardening](#13-security-hardening)
* [14. FSSO Limitations & Exam Traps](#14-fsso-limitations--exam-traps)
* [15. NSE Exam Memory Map](#15-nse-exam-memory-map)
* [16. Quick CLI Reference](#16-quick-cli-reference)
* [17. Final FSSO Audit Checklist](#17-final-fsso-audit-checklist)
* [18. SheynShield Quick Recall](#18-sheynshield-quick-recall)

---

# 1. FSSO Fundamentals

## What is FSSO?

**FSSO — Fortinet Single Sign-On** allows FortiGate to identify users based on Windows/Active Directory authentication events without requiring users to manually authenticate to FortiGate.

### Core identity flow

```text
Windows User
     │
     │ Logon
     ▼
Active Directory / DC
     │
     │ Logon Event
     ▼
FSSO Collector / Mechanism
     │
     │ User + IP + Group
     ▼
FortiGate
     │
     ▼
Identity-Based Firewall Policy
     │
     ▼
Network Access
```

## FSSO Core Checklist

* [ ] Understand that FSSO provides **user identity awareness**
* [ ] Understand that users do not normally need to manually authenticate to FortiGate
* [ ] Identify the Active Directory environment
* [ ] Identify Domain Controllers
* [ ] Identify FSSO Collector Agent architecture
* [ ] Identify FortiGate as the policy enforcement point
* [ ] Verify user/IP mapping
* [ ] Verify AD group membership
* [ ] Verify identity-based firewall policy

---

# 2. FSSO Architecture

```text
                     Active Directory
                    ┌────────┬────────┐
                    │        │        │
                   DC1      DC2      DC3
                    │        │        │
                    └────┬───┴───┬────┘
                         │
                    Logon Events
                         │
                         ▼
                 ┌─────────────────┐
                 │ FSSO Collector  │
                 │     Agent       │
                 └────────┬────────┘
                          │
                     User/IP/Group
                          │
                          ▼
                 ┌─────────────────┐
                 │    FortiGate    │
                 └────────┬────────┘
                          │
                    Identity Match
                          │
                          ▼
                 Firewall Policy
```

## Component Checklist

| Component         | Verify                                |
| ----------------- | ------------------------------------- |
| Active Directory  | User/group repository                 |
| Domain Controller | Windows authentication/logon events   |
| FSSO Collector    | Identity collection                   |
| FortiGate         | Identity mapping + policy enforcement |
| DNS               | Name resolution                       |
| NTP               | Time synchronization                  |
| Firewall Policy   | Identity-based access control         |

---

# 3. FSSO Deployment Checklist

## Collector Agent Architecture

```text
AD / DC
   │
   │ Logon Events
   ▼
FSSO Collector Agent
   │
   │ Identity Information
   ▼
FortiGate
```

* [ ] Collector Agent installed
* [ ] Collector Agent service running
* [ ] Domain Controllers configured
* [ ] AD monitoring configured
* [ ] FortiGate can reach Collector Agent
* [ ] Shared secret configured
* [ ] Collector Agent shows connected/verified
* [ ] User identity reaches FortiGate
* [ ] AD groups are available

## Local / FortiGate-Oriented Identity Retrieval

```text
FortiGate
   │
   ▼
Identity Repository / AD
   │
   ▼
Identity Retrieval / Validation
```

* [ ] Verify whether local identity retrieval is being used
* [ ] Verify required AD/LDAP connectivity
* [ ] Verify required Windows security events
* [ ] Verify polling configuration where applicable

### Exam Memory

```text
Collector Agent
     ↓
Collect + Push

Local
     ↓
FortiGate retrieves/checks identity
```

---

# 4. Active Directory Prerequisites

## DNS Checklist

DNS is one of the first dependencies to verify.

* [ ] FortiGate can resolve Domain Controllers
* [ ] FortiGate can resolve AD-related hostnames
* [ ] Windows clients use appropriate AD DNS
* [ ] Forward DNS works
* [ ] Reverse resolution is correct where required
* [ ] No stale workstation DNS records exist
* [ ] DNS latency is acceptable

### DNS Flow

```text
FortiGate
    │
    ▼
Internal DNS
    │
    ▼
Active Directory
```

---

## NTP / Time Checklist

* [ ] FortiGate time is correct
* [ ] Domain Controller time is correct
* [ ] Windows client time is correct
* [ ] NTP is configured
* [ ] Time synchronization is stable
* [ ] Time zone configuration is correct

```text
FortiGate
    ≈
Domain Controller
    ≈
Windows Client
```

> **High-value troubleshooting rule:** Authentication and identity problems should not be debugged while ignoring time synchronization.

---

## Domain Client Checklist

* [ ] Client is domain joined
* [ ] Client can reach Domain Controller
* [ ] Client uses correct DNS
* [ ] Windows authentication succeeds
* [ ] User can log on normally
* [ ] Domain authentication events are generated
* [ ] Workstation hostname resolves correctly
* [ ] Workstation IP can be determined

---

# 5. FSSO Collector Agent

## Installation Checklist

* [ ] Install a Fortinet-supported FSSO Collector version
* [ ] Verify FortiOS/FSSO compatibility
* [ ] Install on an appropriate Windows Server
* [ ] Verify Domain Controller reachability
* [ ] Configure AD monitoring
* [ ] Configure FortiGate connection
* [ ] Configure shared secret
* [ ] Verify service status

### Architecture

```text
Windows Server
├── FSSO Collector Agent
├── AD Integration
└── DC Monitoring
```

## Service Account Checklist

* [ ] Use a dedicated service account where practical
* [ ] Avoid unnecessary Domain Administrator privileges
* [ ] Grant minimum required permissions
* [ ] Protect service-account credentials
* [ ] Document account ownership
* [ ] Monitor account expiration/lockout

---

# 6. FortiGate FSSO Configuration

## GUI

Typical navigation:

```text
Security Fabric
└── External Connectors
    └── FSSO
```

## Configuration Checklist

* [ ] Create FSSO connector
* [ ] Configure Collector Agent IP/FQDN
* [ ] Configure authentication/shared secret
* [ ] Configure LDAP/AD integration where required
* [ ] Verify connector status
* [ ] Verify Collector Agent connectivity
* [ ] Verify identity information

### Example

```text
Name:
    fsso-ldap

Primary FSSO Agent:
    192.168.20.200

Key:
    <FSSO_SHARED_SECRET>
```

> ⚠️ **Never publish real FSSO passwords, shared secrets, certificates, or production IP information in a public GitHub repository.**

---

## CLI Configuration

```bash
config user fsso
    edit "fsso-ldap"
        set server "192.168.20.200"
        set password <FSSO_SHARED_SECRET>
    next
end
```

### With LDAP

```bash
config user fsso
    edit "fsso-ldap"
        set ldap-server "winsrv-2016"
        set server "192.168.20.200"
        set password <FSSO_SHARED_SECRET>
    next
end
```

> **Version note:** Always verify the exact CLI syntax against the target FortiOS release.

---

# 7. FSSO Groups

FSSO becomes useful for firewall authorization when AD/FSSO groups are mapped to FortiGate user groups.

## Group Configuration Checklist

* [ ] Create FortiGate FSSO user group
* [ ] Select FSSO remote server
* [ ] Select required remote AD groups
* [ ] Verify group membership
* [ ] Verify group synchronization
* [ ] Test a real domain user
* [ ] Confirm the expected group is visible

### Example

```text
Active Directory
│
├── IT
├── HR
├── Finance
└── Guest
       │
       ▼
FortiGate FSSO Groups
       │
       ▼
Firewall Policies
```

---

# 8. Identity-Based Firewall Policy

## Policy Checklist

* [ ] Incoming interface is correct
* [ ] Outgoing interface is correct
* [ ] Source address is correct
* [ ] Source user/group is correct
* [ ] Destination is correct
* [ ] Service is correct
* [ ] Schedule is correct
* [ ] Action is correct
* [ ] FSSO group is referenced
* [ ] Policy order is correct
* [ ] Identity policy is actually being matched

### Identity-Based Policy Model

```text
Source IP
    +
User Identity
    +
AD Group
    +
Destination
    +
Service
    ↓
Firewall Policy
    ↓
ALLOW / DENY
```

### Critical Concept

```text
FSSO configured
      ≠
Every policy becomes identity-aware
```

The firewall policy must reference the appropriate FSSO/user group.

---

# 9. FSSO Identity Verification

## Identity Chain

```text
[1] User Login
      ↓
[2] Domain Controller
      ↓
[3] Logon Event
      ↓
[4] FSSO Detection
      ↓
[5] Workstation Resolution
      ↓
[6] User + IP Mapping
      ↓
[7] AD Group Resolution
      ↓
[8] FortiGate Identity
      ↓
[9] Firewall Policy
```

## Verify Every Layer

* [ ] User login succeeds
* [ ] DC receives authentication event
* [ ] FSSO detects event
* [ ] Username is correct
* [ ] Domain is correct
* [ ] Workstation is correct
* [ ] IP address is correct
* [ ] AD groups are correct
* [ ] User appears on FortiGate
* [ ] Policy matches the FSSO group
* [ ] Traffic log shows expected identity

---

# 10. FSSO Troubleshooting

## Step 1 — Check FSSO Server Status

```bash
diagnose debug enable
diagnose debug authd fsso server-status
```

Verify:

* [ ] FSSO server is reachable
* [ ] Collector Agent is connected
* [ ] Status is healthy
* [ ] No connection errors exist

---

## Step 2 — Display FSSO Users

```bash
diagnose debug authd fsso list
```

Check:

* [ ] Username
* [ ] IP address
* [ ] Domain
* [ ] Group information
* [ ] Current identity mappings

---

## Step 3 — Polling Debug

```bash
diagnose debug fsso-polling detail 1
```

Use when investigating polling/local identity retrieval behavior.

---

## Step 4 — FSSO Daemon Debug

```bash
diagnose debug application fssod -1
```

Use for deeper FSSO daemon troubleshooting.

---

## Step 5 — Disable Debugging

Always clean up debugging after testing:

```bash
diagnose debug disable
```

### Operational Rule

```text
Enable Debug
     ↓
Collect Required Information
     ↓
Analyze
     ↓
Disable Debug
```

---

# 11. Troubleshooting Decision Tree

```text
                  FSSO NOT WORKING
                         │
                         ▼
              Is Collector Connected?
                    /          \
                  NO            YES
                  │              │
                  ▼              ▼
           Check network,    Is user visible?
           port, secret,        /       \
           DNS, service       NO         YES
                              │           │
                              ▼           ▼
                         Check AD       Is group
                         events and     correct?
                         collection       / \
                                       NO   YES
                                       │     │
                                       ▼     ▼
                                  Check AD   Check
                                  groups     firewall
                                             policy
                                               │
                                               ▼
                                         Check policy
                                         order/session
```

---

# 12. FSSO Performance Checklist

Large environments require additional planning.

## Capacity Factors

* [ ] Number of users
* [ ] Number of simultaneous logons
* [ ] Number of Domain Controllers
* [ ] Number of domains
* [ ] Number of forests
* [ ] Collector placement
* [ ] Network latency
* [ ] DNS reliability
* [ ] Event processing load
* [ ] Collector performance

### High Authentication Volume

```text
Large Login Burst
       ↓
Many Windows Events
       ↓
FSSO Processing Load
       ↓
Potential Event Processing Pressure
       ↓
Identity Detection Issues
```

For large deployments, validate the FSSO architecture and Collector capacity rather than assuming that a single Collector is always sufficient.

---

# 13. Security Hardening

## Shared Secret

* [ ] Use a strong shared secret
* [ ] Never commit production secrets to GitHub
* [ ] Store secrets securely
* [ ] Rotate secrets according to policy
* [ ] Restrict access to configuration backups

Use:

```bash
set password <FSSO_SHARED_SECRET>
```

Never publish:

```bash
set password "ProductionPassword123"
```

---

## Service Account

* [ ] Dedicated service account
* [ ] Minimum required privileges
* [ ] No unnecessary Domain Admin privileges
* [ ] Strong password
* [ ] Account lifecycle documented
* [ ] Monitor account status

---

## Network Security

* [ ] Restrict FortiGate ↔ Collector communication
* [ ] Restrict Collector ↔ DC communication
* [ ] Verify required ports only
* [ ] Avoid unnecessary exposure
* [ ] Monitor unexpected connection attempts

---

# 14. FSSO Limitations & Exam Traps

## FSSO ≠ Manual Authentication

```text
FSSO
 ↓
Identity Awareness

Manual Authentication
 ↓
User explicitly authenticates
```

FSSO is designed to provide identity information without requiring users to repeatedly authenticate to FortiGate.

---

## FSSO ≠ Firewall Policy

```text
FSSO
 ↓
Provides Identity
```

```text
Firewall Policy
 ↓
Makes Access Decision
```

Together:

```text
Identity
   +
Policy
   ↓
Access Decision
```

---

## Collector Agent vs Local

| Concept            | Collector Agent  | Local                        |
| ------------------ | ---------------- | ---------------------------- |
| Primary model      | Event collection | FortiGate retrieval/checking |
| Identity direction | Push-oriented    | FortiGate-oriented           |
| Main dependency    | Collector        | FortiGate + identity source  |
| Exam keyword       | Collect + Push   | Retrieve / Poll              |

---

## FSSO vs RSSO

| Feature                   | FSSO                           | RSSO                             |
| ------------------------- | ------------------------------ | -------------------------------- |
| Identity source/mechanism | Windows/AD identity mechanisms | RADIUS authentication/accounting |
| Typical environment       | AD/Windows                     | RADIUS-based environments        |
| Result                    | User identity awareness        | RADIUS-derived identity          |
| Policy usage              | Identity-based policy          | Identity-based policy            |

### Memory Rule

```text
FSSO
→ Windows / AD identity

RSSO
→ RADIUS-derived identity
```

---

## NTLM Consideration

For the referenced FSSO scenario:

```text
Kerberos
   ✅ Supported scenario

NTLM
   ⚠️ Not supported in that referenced scenario
```

> Always distinguish a specific FortiOS/FSSO implementation scenario from a blanket statement about every Fortinet identity feature.

---

## Kerberos Event Consideration

The referenced local/polling scenario identifies:

```text
4768
4769
```

as relevant Kerberos authentication events.

Conceptually:

```text
User
 ↓
Kerberos Authentication
 ↓
Domain Controller
 ↓
Security Event
 ↓
FSSO Detection
 ↓
FortiGate Identity
```

---

# 15. NSE Exam Memory Map

```text
                         FSSO
                          │
       ┌──────────────────┼──────────────────┐
       │                  │                  │
       ▼                  ▼                  ▼
    Identity          Architecture       Firewall
       │                  │                  │
       ▼                  ▼                  ▼
 User + IP +        Collector / Local    Identity Policy
 Domain + Group
       │
       ▼
      AD/DC
       │
       ▼
 Logon Events
       │
       ▼
 FSSO Detection
       │
       ▼
 FortiGate Mapping
```

## Five Things to Remember

* [ ] **FSSO = Identity Awareness**
* [ ] **Collector Agent = Collect + Push**
* [ ] **Identity = User + IP + Domain + Groups**
* [ ] **DNS + NTP + AD = Critical Dependencies**
* [ ] **Firewall Policy = Final Access Decision**

---

# 16. Quick CLI Reference

## Configure FSSO

```bash
config user fsso
    edit "fsso-ldap"
        set server "192.168.20.200"
        set password <FSSO_SHARED_SECRET>
    next
end
```

## Configure FSSO with LDAP

```bash
config user fsso
    edit "fsso-ldap"
        set ldap-server "winsrv-2016"
        set server "192.168.20.200"
        set password <FSSO_SHARED_SECRET>
    next
end
```

## Check Server Status

```bash
diagnose debug enable
diagnose debug authd fsso server-status
```

## Display FSSO Users

```bash
diagnose debug authd fsso list
```

## Polling Debug

```bash
diagnose debug fsso-polling detail 1
```

## FSSO Daemon Debug

```bash
diagnose debug application fssod -1
```

## Disable Debug

```bash
diagnose debug disable
```

---

# 17. Final FSSO Audit Checklist

## 🏗️ Architecture

* [ ] FSSO architecture documented
* [ ] Collector Agent identified
* [ ] Domain Controllers identified
* [ ] Identity source documented
* [ ] FortiGate role documented

## 🌐 Network

* [ ] FortiGate → Collector connectivity verified
* [ ] Collector → DC connectivity verified
* [ ] Required communication ports verified
* [ ] DNS resolution verified
* [ ] Reverse resolution verified where required
* [ ] Network latency checked

## 🕐 Time

* [ ] FortiGate NTP verified
* [ ] Domain Controller time verified
* [ ] Client time verified
* [ ] Time synchronization stable

## 👤 Identity

* [ ] User login verified
* [ ] Username detected
* [ ] IP mapping verified
* [ ] Domain verified
* [ ] AD groups verified
* [ ] FSSO user visible on FortiGate

## 🔥 Firewall Policy

* [ ] FSSO group created
* [ ] FSSO group referenced by policy
* [ ] Source interface verified
* [ ] Destination interface verified
* [ ] Source address verified
* [ ] Destination verified
* [ ] Service verified
* [ ] Schedule verified
* [ ] Policy order verified
* [ ] Traffic matches expected policy

## 🛠️ Troubleshooting

* [ ] `server-status` checked
* [ ] `fsso list` checked
* [ ] Polling debug checked where applicable
* [ ] `fssod` debug checked where required
* [ ] AD events verified
* [ ] Group membership verified
* [ ] Session state verified
* [ ] Debugging disabled after testing

## 🔐 Security

* [ ] Shared secret protected
* [ ] No credentials committed to Git
* [ ] Dedicated service account used
* [ ] Least privilege applied
* [ ] Collector communication restricted
* [ ] Configuration backups protected

---

# 18. SheynShield Quick Recall

```text
                 FSSO
                  │
                  ▼
          Windows / AD Login
                  │
                  ▼
             Logon Event
                  │
                  ▼
          FSSO Collector
                  │
                  ▼
          User + IP + Group
                  │
                  ▼
              FortiGate
                  │
                  ▼
       Identity-Based Policy
                  │
             ┌────┴────┐
             ▼         ▼
           ALLOW      DENY
```

### The Core Formula

```text
AD Authentication
       +
FSSO Identity Detection
       +
User/IP Mapping
       +
AD Group Membership
       +
Firewall Policy
       =
Identity-Based Network Access
```

### Troubleshooting Formula

```text
DNS
 ↓
NTP
 ↓
AD
 ↓
Collector
 ↓
Connectivity
 ↓
Identity
 ↓
Group
 ↓
Policy
 ↓
Session
```

### One-Line Takeaways

> **FSSO provides identity awareness; the firewall policy provides the access decision.**

> **Collector Agent architecture is event-driven and push-oriented.**

> **The most important FSSO troubleshooting question is: "At which stage did the identity chain fail?"**

> **If the user exists but the policy does not match, investigate group mapping and policy configuration before blaming AD.**

> **If the FSSO server is disconnected, start with connectivity, DNS, service status, ports, and shared secret.**

---

# 🔎 Keywords

`FortiGate FSSO` · `Fortinet Single Sign-On` · `FortiOS FSSO` · `FortiGate FSSO configuration` · `FortiGate FSSO troubleshooting` · `FSSO Collector Agent` · `FortiGate Active Directory authentication` · `FortiGate AD integration` · `FortiGate identity based firewall policy` · `FortiGate FSSO groups` · `FSSO CLI commands` · `FortiGate FSSO debug` · `FortiGate FSSO server status` · `FortiGate AD SSO` · `FortiGate user identity` · `FortiGate NSE4 FSSO` · `Fortinet FSSO NSE7` · `FSSO troubleshooting checklist` · `FortiGate identity based policy` · `FortiGate Active Directory SSO`

---

# 🏷️ SheynShield Reference

| Field                  | Value                                             |
| ---------------------- | ------------------------------------------------- |
| **Topic**              | FortiGate FSSO                                    |
| **Category**           | User & Authentication / Identity                  |
| **Level**              | NSE4 → NSE7                                       |
| **Primary Technology** | Fortinet Single Sign-On                           |
| **Identity Source**    | Active Directory / Windows                        |
| **Core Components**    | FortiGate + FSSO Collector + AD/DC                |
| **Primary Use Case**   | Identity-Based Firewall Policies                  |
| **Format**             | Checklist + Lab Reference + Troubleshooting Guide |

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

> **SheynShield | Engineering Secure Networks**
>
> **Learn the identity flow. Verify the mapping. Prove the policy match.**

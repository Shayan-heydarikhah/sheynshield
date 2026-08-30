# 🔐 FortiGate LDAP Server & Authentication — Checklist

> **SheynShield | Engineering Secure Networks**
>
> Practical FortiOS checklist for **LDAP, Active Directory, LDAP Authentication, LDAPS, Recursive Search, AD Attributes, LDAP Groups, Wildcard Administrators, Polling AD, FSSO, Identity-Based Policies & Troubleshooting**.

---

## 📌 Table of Contents

- [1. LDAP Architecture](#1-ldap-architecture)
- [2. LDAP Server Configuration](#2-ldap-server-configuration)
- [3. LDAP Bind Types](#3-ldap-bind-types)
- [4. LDAP Connectivity Validation](#4-ldap-connectivity-validation)
- [5. LDAPS & Certificate Validation](#5-ldaps--certificate-validation)
- [6. LDAP Search & Recursive Search](#6-ldap-search--recursive-search)
- [7. LDAP User Attributes](#7-ldap-user-attributes)
- [8. AD Dial-In Attribute](#8-ad-dial-in-attribute)
- [9. LDAP User Groups](#9-ldap-user-groups)
- [10. LDAP Group Matching](#10-ldap-group-matching)
- [11. LDAP Authentication Testing](#11-ldap-authentication-testing)
- [12. LDAP + Firewall Policy](#12-ldap--firewall-policy)
- [13. Wildcard LDAP Administrator](#13-wildcard-ldap-administrator)
- [14. User Device Store](#14-user-device-store)
- [15. Explicit Proxy Authentication](#15-explicit-proxy-authentication)
- [16. LDAP Polling AD](#16-ldap-polling-ad)
- [17. FSSO vs LDAP vs Polling AD](#17-fsso-vs-ldap-vs-polling-ad)
- [18. Large AD Environment](#18-large-ad-environment)
- [19. Redundant Identity Architecture](#19-redundant-identity-architecture)
- [20. LDAP Troubleshooting](#20-ldap-troubleshooting)
- [21. Security Checklist](#21-security-checklist)
- [22. Production Validation Checklist](#22-production-validation-checklist)
- [23. NSE Exam Memory Map](#23-nse-exam-memory-map)
- [24. Golden Rules](#24-golden-rules)
- [25. Quick CLI Reference](#25-quick-cli-reference)
- [26. One-Minute Troubleshooting](#26-one-minute-troubleshooting)
- [27. SheynShield Mental Model](#27-sheynshield-mental-model)
- [28. Resources](#28-resources)

---

# 1. LDAP Architecture

LDAP allows FortiGate to authenticate users and query identity information from an external directory such as Microsoft Active Directory.

```text
                    ACTIVE DIRECTORY
                         / LDAP
                           │
                           │ TCP/389
                           │
                           ▼
                    ┌──────────────┐
                    │   FortiGate  │
                    └──────┬───────┘
                           │
                    LDAP User / Group
                           │
                           ▼
                    Firewall Policy
                           │
                           ▼
                         ACCESS
````

### Core LDAP Checklist

* [ ] LDAP server is reachable
* [ ] Correct LDAP IP/FQDN configured
* [ ] Correct LDAP port configured
* [ ] Correct Base DN configured
* [ ] Bind credentials are valid
* [ ] LDAP search scope is correct
* [ ] User attribute is correct
* [ ] LDAP group mapping is correct
* [ ] Firewall policy references the correct user/group

---

# 2. LDAP Server Configuration

Example environment:

```text
LDAP / Active Directory
        │
        │
WIN-SRV-2016
192.168.20.200
        │
        │ LDAP
        │ TCP/389
        ▼
    FortiGate
```

### Basic Parameters

| Parameter   | Example                           |
| ----------- | --------------------------------- |
| Server Name | `winsrv-2016`                     |
| IP Address  | `192.168.20.200`                  |
| LDAP Port   | `389`                             |
| Base DN     | `OU=publish-users,DC=test,DC=com` |
| Bind Type   | `Regular`                         |
| Username    | `ssoadmin`                        |
| Password    | `<LDAP_PASSWORD>`                 |

### Configuration Checklist

* [ ] Define LDAP server object
* [ ] Configure IP/FQDN
* [ ] Configure LDAP/LDAPS port
* [ ] Configure Base DN
* [ ] Select appropriate bind type
* [ ] Configure service account if required
* [ ] Verify credentials
* [ ] Test connection

> ⚠️ **Security:** Never publish production LDAP credentials, passwords, activation secrets, or service-account credentials in GitHub.

---

# 3. LDAP Bind Types

FortiGate LDAP configuration can use different bind behaviors depending on the directory architecture.

## Regular Bind

```text
Bind Type
   ↓
Regular
   ↓
Configured LDAP Credentials
   ↓
LDAP Search
```

Checklist:

* [ ] Bind username configured
* [ ] Bind password configured
* [ ] Bind DN is valid
* [ ] Search Base DN is correct
* [ ] LDAP server accepts the bind

---

## Simple Bind

```text
DN + Password
      ↓
LDAP Server
      ↓
Authentication
```

Checklist:

* [ ] Correct DN configured
* [ ] Correct password configured
* [ ] LDAP server permits simple bind
* [ ] Transport security is evaluated

---

## Anonymous Bind

```text
FortiGate
   │
   │ No Bind Credentials
   ▼
LDAP Server
```

Checklist:

* [ ] LDAP server explicitly permits anonymous access
* [ ] Anonymous directory queries are intentional
* [ ] Security implications have been reviewed

> 🔐 **Production recommendation:** Avoid anonymous LDAP access unless it is explicitly required and secured by the directory design.

---

# 4. LDAP Connectivity Validation

Before troubleshooting authentication, prove basic connectivity.

### GUI Validation

```text
LDAP Server
    ↓
Test Connection
```

Then validate:

```text
Browse
   ↓
Query
   ↓
Authentication Test
```

### Checklist

* [ ] LDAP server responds
* [ ] TCP/389 works for LDAP
* [ ] LDAPS port is reachable when applicable
* [ ] DNS resolution works
* [ ] FortiGate can resolve the LDAP hostname
* [ ] Bind operation succeeds
* [ ] LDAP browse works
* [ ] LDAP query works
* [ ] User authentication succeeds

### Troubleshooting Principle

```text
Connectivity
     ↓
Bind
     ↓
Search
     ↓
User
     ↓
Group
     ↓
Policy
```

**Do not start with firewall policy debugging before proving LDAP connectivity.**

---

# 5. LDAPS & Certificate Validation

For secure LDAP deployments:

```text
FortiGate
    │
    │ TLS
    ▼
LDAP Server
    │
    ▼
Certificate
```

### Certificate Identity

Check:

```text
Subject Alternative Name (SAN)
```

and, where applicable:

```text
Common Name (CN)
```

### LDAPS Checklist

* [ ] LDAP server certificate is valid
* [ ] Certificate is not expired
* [ ] Correct CA chain is available
* [ ] SAN matches the LDAP server identity
* [ ] CN is appropriate where applicable
* [ ] FortiGate trusts the issuing CA
* [ ] Correct LDAPS port is configured
* [ ] TLS negotiation succeeds

> 🔐 **Best Practice:** Prefer certificate identity validation using a correct SAN rather than relying on legacy CN-only assumptions.

---

# 6. LDAP Search & Recursive Search

LDAP search determines where and how FortiGate looks for directory objects.

### Search Model

```text
Base DN
   │
   ▼
LDAP Directory
   │
   ├── OU
   │    ├── User
   │    └── User
   │
   └── Nested OU
        └── User
```

### Recursive Search

Example:

```bash
config user ldap
    edit "winsrv-2016"
        set search-type recursive
    next
end
```

### Checklist

* [ ] Base DN is correct
* [ ] Search scope is appropriate
* [ ] Recursive search is enabled where required
* [ ] Nested OUs are reachable through the search
* [ ] LDAP server supports the required search behavior
* [ ] FortiOS version supports the configured syntax

### Mental Model

```text
Base DN
   ↓
Search Scope
   ↓
LDAP Objects
   ↓
User
   ↓
Group
```

---

# 7. LDAP User Attributes

User identification depends on the directory schema.

### Common Attributes

| Attribute        | Typical Use                       |
| ---------------- | --------------------------------- |
| `cn`             | Common Name                       |
| `sAMAccountName` | Traditional Windows/AD logon name |
| `uid`            | Common LDAP user identifier       |

### Attribute Checklist

* [ ] Correct username attribute identified
* [ ] Attribute exists on the directory
* [ ] Attribute contains expected value
* [ ] FortiGate LDAP configuration matches the directory schema
* [ ] User search returns the expected object

### Important AD Attribute

```text
sAMAccountName
        ↓
Traditional Windows Logon Identity
```

> 💡 **NSE memory:** LDAP authentication can fail even when connectivity is perfect if FortiGate is searching for the wrong user attribute.

---

# 8. AD Dial-In Attribute

Active Directory can expose:

```text
msNPAllowDialin
```

This attribute is associated with Network Access Permission / dial-in access behavior.

### Concept

```text
AD User
   │
   ▼
msNPAllowDialin
   │
   ▼
Network Access Permission
```

### Configuration Example

```bash
config user ldap
    edit "winsrv-2016"
        set member-attr "msNPAllowDialin"
    next
end
```

### Checklist

* [ ] Attribute exists in AD
* [ ] Correct attribute is configured
* [ ] User has expected dial-in state
* [ ] LDAP query returns the expected value

---

# 9. LDAP User Groups

Create a FortiGate user group backed by LDAP.

Example:

```bash
config user group
    edit "ldap-g-local"
        set member "winsrv-2016"
    next
end
```

### Authentication Chain

```text
LDAP Server
     ↓
LDAP User
     ↓
FortiGate LDAP Group
     ↓
Firewall Policy
     ↓
Access
```

### Checklist

* [ ] LDAP server is configured
* [ ] FortiGate user group exists
* [ ] LDAP server is assigned to the group
* [ ] Correct remote groups are mapped
* [ ] User belongs to the expected AD group
* [ ] Policy references the correct FortiGate group

---

# 10. LDAP Group Matching

Example:

```bash
config user group
    edit "ldap-g-local"
        set member "winsrv-2016"
        config match
            edit 1
                set server-name "winsrv-2016"
                set group-name "true"
            next
        end
    next
end
```

### Group Matching Checklist

* [ ] LDAP server name is correct
* [ ] Remote group is correct
* [ ] AD group membership is correct
* [ ] Group DN is correct where required
* [ ] Nested group behavior is understood
* [ ] FortiGate user group matches the intended AD group

### Identity Mapping

```text
AD User
   ↓
AD Group
   ↓
LDAP
   ↓
FortiGate User Group
   ↓
Firewall Policy
```

---

# 11. LDAP Authentication Testing

Use an authentication test to isolate LDAP authentication from policy behavior.

Example:

```bash
diagnose test authserver ldap winsrv-2016 <username> <password>
```

### Test Checklist

* [ ] Correct LDAP server name
* [ ] Correct username
* [ ] Correct test password
* [ ] LDAP connectivity works
* [ ] Bind succeeds
* [ ] User is found
* [ ] User authentication succeeds

> ⚠️ Never store real passwords in GitHub commands, screenshots, documentation, or public labs.

### Authentication Isolation

```text
LDAP Authentication Test
          │
          ├── FAIL
          │    ↓
          │ LDAP problem
          │
          └── SUCCESS
               ↓
          Check Group
               ↓
          Check Policy
```

---

# 12. LDAP + Firewall Policy

LDAP authentication becomes useful when identity is incorporated into access control.

```text
                    LDAP / AD
                       │
                       ▼
                  LDAP User
                       │
                       ▼
                 User Group
                       │
                       ▼
               Firewall Policy
                       │
                       ▼
                     ACCESS
```

### Policy Checklist

* [ ] Incoming interface is correct
* [ ] Source address is correct
* [ ] LDAP-backed user group is selected
* [ ] Destination is correct
* [ ] Service is correct
* [ ] Schedule is correct
* [ ] Policy order is correct
* [ ] Authentication method is correct
* [ ] Traffic actually reaches the intended policy

### Critical Rule

> **LDAP configuration alone does not make every firewall policy identity-aware.**

---

# 13. Wildcard LDAP Administrator

FortiGate can use remote LDAP users for administrator authentication.

Concept:

```text
LDAP User
    │
    ▼
Remote LDAP Group
    │
    ▼
FortiGate Administrator
    │
    ▼
Admin Profile
```

### Example

```text
Remote Server Group
        ↓
winsrv-2016

Admin Profile
        ↓
Super Admin / Restricted Profile
```

### Checklist

* [ ] LDAP authentication works
* [ ] Remote LDAP group is correct
* [ ] Administrator object exists
* [ ] Correct admin profile is assigned
* [ ] Trusted hosts are configured where appropriate
* [ ] MFA/2FA is enabled for privileged access
* [ ] Least-privilege administration is used

---

# 14. User Device Store

### User Count

```bash
diagnose user-device-store user-count list 1
```

Use this to inspect user-count information stored by the FortiGate user-device store.

### Query

```bash
diagnose user-device-store user-count query "<LDAP-DN>"
```

### Statistics

```bash
diagnose user-device-store user-stats query <date> <value>
```

### Checklist

* [ ] User count can be displayed
* [ ] LDAP DN is correct
* [ ] User statistics are available
* [ ] Time range is appropriate
* [ ] Retention behavior is understood for the target FortiOS release

---

# 15. Explicit Proxy Authentication

Explicit proxy can be used to create an authentication workflow.

```text
Client
   │
   ▼
Explicit Proxy
   │
   ▼
Authentication
   │
   ▼
LDAP
   │
   ▼
Active Directory
```

### Authentication Checklist

* [ ] Explicit proxy is configured
* [ ] Authentication scheme exists
* [ ] LDAP server is reachable
* [ ] LDAP user group is configured
* [ ] Proxy policy references the group
* [ ] Client traffic reaches the proxy
* [ ] Authentication challenge is generated
* [ ] Authentication succeeds

### Failure Pattern

```text
Client
   ↓
Explicit Proxy
   ↓
No Authentication Rule
   ↓
Authentication Failure / Traffic Denied
```

---

# 16. LDAP Polling AD

Polling AD allows FortiGate to query Active Directory through LDAP.

```text
             Active Directory
                    │
                    │ LDAP
                    ▼
                FortiGate
                    │
                    ▼
                Polling AD
```

### Requirements

* [ ] LDAP server configured
* [ ] LDAP connectivity works
* [ ] Directory credentials configured
* [ ] Required permissions exist
* [ ] Base DN is correct
* [ ] User search works
* [ ] Group lookup works

### Architecture

```text
FortiGate
    │
    │ Direct LDAP Query
    ▼
Active Directory
    │
    ▼
User Information
```

> **Key concept:** Polling AD does not require a DC Agent for the direct LDAP query mechanism.

---

# 17. FSSO vs LDAP vs Polling AD

| Feature               | LDAP                             | FSSO                   | Polling AD                                        |
| --------------------- | -------------------------------- | ---------------------- | ------------------------------------------------- |
| Primary purpose       | Authentication / directory query | User logon awareness   | AD user information retrieval                     |
| Direct LDAP query     | Yes                              | Not the core mechanism | Yes                                               |
| User ↔ IP mapping     | Not inherently                   | Core capability        | Limited                                           |
| Transparent identity  | Limited                          | Strong                 | Limited                                           |
| DC Agent              | No                               | Common architecture    | No                                                |
| Collector             | No                               | Common                 | No                                                |
| Authentication        | Yes                              | Identity integration   | User retrieval/authentication depending on design |
| Large AD environments | Depends on design                | Well suited            | Less scalable                                     |
| Main mental model     | Authenticate/query               | Detect logon           | Poll/query                                        |

### Fast Memory

```text
LDAP
→ "Can I authenticate/query this user?"

FSSO
→ "Which user is associated with this IP?"

Polling AD
→ "Can FortiGate query AD directly?"
```

---

# 18. Large AD Environment

Large environments may contain:

```text
                 Active Directory
                  /      |      \
                DC1     DC2     DC3
                 \       |       /
                  \      |      /
                   FSSO Architecture
                         │
                         ▼
                      FortiGate
```

### Design Checklist

* [ ] Multiple DCs are considered
* [ ] Collector placement is appropriate
* [ ] Network latency is acceptable
* [ ] DNS is reliable
* [ ] Authentication volume is understood
* [ ] Event processing capacity is considered
* [ ] Redundancy is implemented where required
* [ ] FSSO architecture matches environment scale

### Practical Rule

```text
Small / simple environment
        ↓
Direct LDAP may be sufficient

Large / complex AD
        ↓
FSSO architecture may be more appropriate
```

---

# 19. Redundant Identity Architecture

A hybrid architecture can provide multiple identity mechanisms.

```text
                 Active Directory
                  /            \
                 /              \
             FSSO               LDAP
              │                  │
              │                  │
              └──────┬───────────┘
                     ▼
                  FortiGate
                     │
                     ▼
                   Policy
```

### Resilience Checklist

* [ ] Primary identity mechanism identified
* [ ] Secondary authentication path identified
* [ ] LDAP remains available if FSSO is unavailable
* [ ] User groups are consistently mapped
* [ ] Failure behavior is documented
* [ ] Break-glass administrative access exists
* [ ] Recovery procedure is tested

> ⚠️ Do not assume that configuring LDAP automatically provides transparent FSSO behavior. The two mechanisms solve different identity problems.

---

# 20. LDAP Troubleshooting

## Troubleshooting Chain

```text
LDAP Failure
    │
    ▼
[1] Connectivity
    │
    ▼
[2] Port
    │
    ▼
[3] Bind
    │
    ▼
[4] Base DN
    │
    ▼
[5] Search
    │
    ▼
[6] User Attribute
    │
    ▼
[7] User Authentication
    │
    ▼
[8] Group Membership
    │
    ▼
[9] FortiGate User Group
    │
    ▼
[10] Firewall Policy
```

---

## Connectivity Failure

Check:

* [ ] LDAP server IP/FQDN
* [ ] Routing
* [ ] TCP/389
* [ ] LDAPS port where applicable
* [ ] DNS
* [ ] Firewall rules
* [ ] Network ACLs

---

## Bind Failure

Check:

* [ ] Bind type
* [ ] Username
* [ ] Password
* [ ] Bind DN
* [ ] Account status
* [ ] LDAP server permissions

---

## Search Failure

Check:

* [ ] Base DN
* [ ] Search scope
* [ ] Recursive search
* [ ] User DN
* [ ] LDAP attributes
* [ ] Directory structure

---

## Authentication Failure

Check:

* [ ] Username attribute
* [ ] User account status
* [ ] Password
* [ ] LDAP bind
* [ ] Authentication test
* [ ] LDAP group mapping

---

## Policy Failure

If LDAP authentication succeeds but traffic is denied:

```text
LDAP Authentication
        ↓
       PASS
        ↓
User Group
        ↓
       PASS?
        ↓
Firewall Policy
        ↓
       MATCH?
```

Check:

* [ ] User appears correctly
* [ ] User belongs to correct LDAP group
* [ ] FortiGate group is correct
* [ ] Policy references correct group
* [ ] Policy order is correct
* [ ] Traffic matches expected policy
* [ ] Session information is correct

---

# 21. Security Checklist

## Identity Security

* [ ] Use centralized identity where appropriate
* [ ] Avoid shared administrator accounts
* [ ] Enable MFA/2FA for privileged accounts
* [ ] Use least-privilege administrator profiles
* [ ] Restrict administrative access to trusted networks
* [ ] Configure trusted hosts where appropriate
* [ ] Maintain a protected break-glass account

## LDAP Security

* [ ] Prefer secure LDAP where appropriate
* [ ] Validate LDAP certificates
* [ ] Verify certificate SAN
* [ ] Protect LDAP credentials
* [ ] Use dedicated service accounts
* [ ] Apply least privilege
* [ ] Avoid anonymous LDAP unless justified

## Documentation Security

* [ ] Replace real passwords with placeholders
* [ ] Remove production IPs when necessary
* [ ] Remove API keys
* [ ] Remove shared secrets
* [ ] Remove private certificates
* [ ] Sanitize screenshots
* [ ] Sanitize debug output

---

# 22. Production Validation Checklist

Before deploying LDAP authentication:

### Infrastructure

* [ ] DNS is operational
* [ ] NTP is operational
* [ ] Routing to AD is correct
* [ ] LDAP ports are reachable
* [ ] LDAPS certificates are valid where applicable

### LDAP

* [ ] LDAP server configured
* [ ] Bind succeeds
* [ ] Base DN verified
* [ ] Search works
* [ ] User lookup works
* [ ] Group lookup works
* [ ] Authentication test succeeds

### FortiGate

* [ ] LDAP user group configured
* [ ] Remote group mapping verified
* [ ] Firewall policy references correct group
* [ ] Policy order verified
* [ ] Authentication logs verified
* [ ] Session behavior verified

### Recovery

* [ ] Break-glass account exists
* [ ] Recovery procedure documented
* [ ] LDAP outage scenario tested
* [ ] Administrative access recovery tested

---

# 23. NSE Exam Memory Map

```text
LDAP
│
├── Server
│   ├── IP / FQDN
│   ├── Port
│   ├── Base DN
│   └── Bind Type
│
├── Bind
│   ├── Regular
│   ├── Simple
│   └── Anonymous
│
├── Search
│   ├── Base DN
│   ├── Search Scope
│   └── Recursive
│
├── Attributes
│   ├── cn
│   ├── sAMAccountName
│   └── uid
│
├── AD
│   └── msNPAllowDialin
│
├── Groups
│   ├── LDAP Server
│   ├── Remote Group
│   └── FortiGate User Group
│
├── Authentication
│   ├── LDAP
│   ├── Explicit Proxy
│   └── Identity-Based Policy
│
├── Polling AD
│   ├── Direct LDAP Query
│   ├── No DC Agent
│   └── User Information
│
└── FSSO
    ├── Logon Detection
    ├── User ↔ IP
    ├── Collector
    └── DC Agent
```

---

# 24. Golden Rules

* [ ] **LDAP = authentication + directory lookup**
* [ ] **FSSO = logon awareness + user/IP mapping**
* [ ] **Polling AD = FortiGate directly queries AD**
* [ ] **Base DN determines where LDAP searches**
* [ ] **Recursive search enables subtree-oriented searches where supported**
* [ ] **`sAMAccountName` is a common AD logon attribute**
* [ ] **`cn` is a common LDAP directory attribute**
* [ ] **`uid` is common in many non-AD LDAP implementations**
* [ ] **`msNPAllowDialin` is an AD dial-in related attribute**
* [ ] **LDAP connectivity must be proven before authentication troubleshooting**
* [ ] **LDAP authentication success does not guarantee policy matching**
* [ ] **The FortiGate user group must map correctly to the directory**
* [ ] **The firewall policy must reference the appropriate identity group**
* [ ] **LDAPS requires correct certificate validation**
* [ ] **SAN should correctly identify the LDAP server**
* [ ] **Dedicated least-privileged service accounts are preferred**
* [ ] **Never publish production credentials**
* [ ] **Large AD environments require appropriate identity architecture**
* [ ] **Maintain a recovery path for administrative authentication**

---

# 25. Quick CLI Reference

## Recursive LDAP Search

```bash
config user ldap
    edit "winsrv-2016"
        set search-type recursive
    next
end
```

---

## LDAP Authentication Test

```bash
diagnose test authserver ldap winsrv-2016 <username> <password>
```

---

## LDAP Member Attribute

```bash
config user ldap
    edit "winsrv-2016"
        set member-attr "msNPAllowDialin"
    next
end
```

---

## LDAP User Group

```bash
config user group
    edit "ldap-g-local"
        set member "winsrv-2016"
    next
end
```

---

## User Count

```bash
diagnose user-device-store user-count list 1
```

---

## User Count Query

```bash
diagnose user-device-store user-count query "<LDAP-DN>"
```

---

## User Statistics

```bash
diagnose user-device-store user-stats query <date> <value>
```

> ⚠️ **Version note:** FortiOS CLI syntax can change between releases. Verify commands against the target FortiOS version before using them in production.

---

# 26. One-Minute Troubleshooting

When LDAP authentication fails, ask these questions in order:

```text
1. Can FortiGate reach LDAP?
        ↓
2. Is the correct port reachable?
        ↓
3. Does the bind succeed?
        ↓
4. Is the Base DN correct?
        ↓
5. Can FortiGate find the user?
        ↓
6. Is the username attribute correct?
        ↓
7. Does authentication succeed?
        ↓
8. Is the user in the correct AD group?
        ↓
9. Is the FortiGate user group correct?
        ↓
10. Does the firewall policy reference it?
```

### Fast Decision Tree

```text
              LDAP AUTH FAILURE
                      │
                      ▼
              Connectivity OK?
                /          \
              NO            YES
              │              │
              ▼              ▼
        Fix routing,      Bind OK?
        DNS, port         /      \
                         NO       YES
                         │         │
                         ▼         ▼
                      Check      User Found?
                      creds       /     \
                                NO       YES
                                │         │
                                ▼         ▼
                             Base DN    Auth OK?
                             Search      /   \
                                        NO    YES
                                        │      │
                                        ▼      ▼
                                     Attribute Group OK?
                                     / Bind       /   \
                                                NO     YES
                                                │       │
                                                ▼       ▼
                                             Group   Policy
                                             Mapping Matching
```

---

# 27. SheynShield Mental Model

## LDAP

```text
              ACTIVE DIRECTORY
                     │
                     │ LDAP
                     ▼
                FortiGate
                     │
          ┌──────────┼──────────┐
          │          │          │
         Bind       Search    Attribute
          │          │          │
          ▼          ▼          ▼
      Credentials    DN     sAMAccountName
                              / cn / uid
          │          │          │
          └──────────┼──────────┘
                     ▼
                 LDAP User
                     │
                     ▼
                 LDAP Group
                     │
                     ▼
              Firewall Policy
                     │
                     ▼
                   ACCESS
```

---

## FSSO

```text
User Login
    ↓
Domain Controller
    ↓
FSSO Collector / Agent
    ↓
User ↔ IP Mapping
    ↓
FortiGate
    ↓
Identity Policy
```

---

## Polling AD

```text
FortiGate
    │
    │ Direct LDAP Query
    ▼
Active Directory
    │
    ▼
User Information
    │
    ▼
FortiGate Policy
```

---

## Core Distinction

```text
LDAP
→ "Can I authenticate/query this user?"

FSSO
→ "Which user is associated with this IP?"

Polling AD
→ "Can FortiGate query AD directly?"

LDAP Group
→ "Which users belong to this authorization group?"

Firewall Policy
→ "What is this authenticated identity allowed to access?"
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

# 🎯 Final Checklist

```text
LDAP
│
├── Connectivity
│   ├── DNS
│   ├── Routing
│   └── Port
│
├── Authentication
│   ├── Bind
│   ├── Username
│   └── Password
│
├── Directory Search
│   ├── Base DN
│   ├── Search Scope
│   └── Recursive Search
│
├── Identity
│   ├── cn
│   ├── sAMAccountName
│   └── uid
│
├── Authorization
│   ├── AD Group
│   ├── LDAP Group
│   └── Firewall Policy
│
├── Secure LDAP
│   ├── TLS
│   ├── CA
│   └── SAN
│
├── Alternative Identity
│   ├── FSSO
│   └── Polling AD
│
└── Operations
    ├── Troubleshooting
    ├── Monitoring
    ├── Recovery
    └── Documentation Security
```

---

## 🧠 SheynShield Golden Mental Model

> **LDAP → Bind → Search → User → Group → Policy → Access**

> **FSSO → Logon Detection → User/IP Mapping → Identity Policy**

> **Polling AD → Direct LDAP Query → User Information → Policy**

---

## 🔎 Keywords

`FortiGate LDAP` · `FortiGate LDAP configuration` · `FortiGate LDAP authentication` · `FortiOS LDAP` · `FortiGate Active Directory` · `FortiGate AD authentication` · `FortiGate LDAPS` · `FortiGate LDAP troubleshooting` · `FortiGate LDAP group` · `FortiGate LDAP recursive search` · `FortiGate FSSO` · `FortiGate Polling AD` · `FortiGate identity based policy` · `Fortinet LDAP` · `FortiGate authentication troubleshooting` · `FortiGate wildcard administrator` · `FortiGate LDAP CLI` · `Fortinet Active Directory integration`

---

**SheynShield | Engineering Secure Networks**

`FortiGate` • `FortiOS` • `LDAP` • `Active Directory` • `LDAPS` • `FSSO` • `Polling AD` • `Identity Security` • `Network Security`


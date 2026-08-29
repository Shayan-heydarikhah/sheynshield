# FortiGate LDAP Server & Authentication

> **FortiOS Focus:** User Authentication / LDAP / Active Directory / FSSO
> **Scope:** LDAP Server, LDAP Authentication, Recursive Search, AD Attributes, Dial-In Attributes, Wildcard Admin, Authentication Troubleshooting, Polling AD, FSSO
> **Brand:** SheynShield — Engineering Secure Networks

---

# 1. LDAP Server — Basic Configuration

LDAP authentication allows FortiGate to authenticate users against an external LDAP/Active Directory server.

### Example Environment

```text
LDAP / Active Directory
        |
        |
WIN-SRV-2016
192.168.20.200
        |
      LDAP
      TCP/389
        |
    FortiGate
```

### LDAP Server Parameters

```text
Server Name       → winsrv-2016
IP Address        → 192.168.20.200
Port              → 389
Common Name       → cn
Distinguished Name → OU=publish-users,DC=test,DC=com
Bind Type         → Regular
Username          → ssoadmin
Password          → <LDAP-password>
```

> **Security Rule:** Never publish real LDAP credentials in documentation, GitHub repositories, screenshots, or training material.

---

# 2. LDAP Bind Types

FortiGate can use different LDAP bind/search behaviors depending on the authentication design.

## Regular

```text
Bind Type → Regular
```

# Regular bind authenticates to LDAP using configured credentials and can perform DN-based subtree searches.

Example:

```text
Username → ssoadmin
Password → <password>
DN       → OU=publish-users,DC=test,DC=com
```

---

## Simple

```text
Bind Type → Simple
```

# Simple bind uses a straightforward DN/password authentication mechanism without the same recursive subtree search behavior.

Concept:

```text
DN + Password
     ↓
LDAP Server
     ↓
Authentication
```

---

## Anonymous

```text
Bind Type → Anonymous
```

# Anonymous bind connects to LDAP without configured bind credentials when the LDAP server permits anonymous access.

> **Security Note:** Anonymous LDAP access should only be used when explicitly supported and justified by the directory security design.

---

# 3. LDAP Connection Test

After configuring the LDAP server, verify connectivity.

```text
Test Connection
```

# Tests whether FortiGate can establish the LDAP connection successfully.

Also verify:

```text
Browse
Query
```

# Browse and query operations help validate DN structure, LDAP objects, and searchable directory information.

---

# 4. LDAP over Certificate

When LDAP communication uses certificates, certificate identity validation becomes important.

```text
Subject Alternative Name (SAN)
```

# FortiGate validates the LDAP server identity using the certificate SAN when available.

```text
Common Name (CN)
```

# CN can be used as the certificate identity when SAN-based validation is unavailable or when supported by the validation process.

### Concept

```text
FortiGate
    |
    | TLS
    ↓
LDAP Server
    |
Certificate
    |
    +── SAN
    |
    └── CN
```

> **Production Rule:** Prefer a valid certificate with the correct SAN matching the LDAP server identity.

---

# 5. Recursive LDAP Search

Recursive search allows FortiGate to search through LDAP directory subtrees.

```bash
config user ldap
    edit "winsrv-2016"
        set search-type recursive
    end
end
```

# Enables recursive LDAP searching for the configured LDAP server.

### Important

```text
Recursive Search
       ↓
Search Base DN
       ↓
Subtree
       ↓
Nested LDAP Objects
```

> **Compatibility Note:** Recursive LDAP search behavior depends on the LDAP implementation and FortiOS version. Verify support for the target directory service.

---

# 6. LDAP Authentication Diagnostic

```bash
diagnose test authserver ldap winsrv-2016 u1 1qaz@WSX
```

# Tests user authentication against the configured LDAP authentication server.

Example concept:

```text
FortiGate
   |
   | LDAP Authentication Request
   ↓
winsrv-2016
   |
   ↓
User Authentication
```

> **Security Rule:** Never use real production passwords in commands stored in documentation.

---

# 7. LDAP User Identification Attributes

LDAP/Active Directory users can be identified using directory attributes.

Common attributes include:

```text
cn
sAMAccountName
uid
```

### CN

```text
cn
```

# Common Name identifies the LDAP directory object.

### sAMAccountName

```text
sAMAccountName
```

# Active Directory commonly uses sAMAccountName as the traditional Windows logon username.

### UID

```text
uid
```

# UID is commonly used as a user identifier in LDAP environments, especially in non-AD LDAP implementations.

---

# 8. LDAP Dial-In Attribute

Active Directory can contain the `msNPAllowDialin` attribute.

```text
msNPAllowDialin
```

# Controls the user's Network Access Permission / dial-in access state in Active Directory.

Example concept:

```text
AD User
   |
   └── Dial-in
         |
         └── Network Access Permission
                |
                └── Allow Access
```

---

# 9. Configure LDAP Member Attribute

```bash
config user ldap
    edit "winsrv-2016"
        set member-attr "msNPAllowDialin"
    end
end
```

# Defines the LDAP member attribute FortiGate uses for the configured LDAP authentication logic.

---

# 10. LDAP User Group

Create a FortiGate user group and associate it with the LDAP server.

```bash
config user group
    edit "ldap-g-local"
        set member "winsrv-2016"
    end
end
```

# Creates a FortiGate user group that uses the configured LDAP authentication server.

---

# 11. LDAP Group Matching

```bash
config user group
    edit "ldap-g-local"
        set member "winsrv-2016"
        config match
            edit 1
                set server-name "winsrv-2016"
                set group-name "true"
            end
        end
    end
end
```

# Matches users against the configured LDAP server/group criteria.

> **Design Rule:** LDAP group membership and FortiGate user-group membership must be designed consistently with the directory structure.

---

# 12. LDAP + Firewall Policy

LDAP authentication must be associated with the firewall policy when identity-based authentication is required.

```text
Firewall Policy
       |
       ├── Source Interface
       ├── Source Address
       ├── Source User/User Group
       └── Destination
```

# The firewall policy can use LDAP-backed user groups as an identity-based source condition.

### Important

```text
LDAP Server
     ↓
LDAP User Group
     ↓
Firewall Policy
     ↓
User Authentication
     ↓
Access
```

---

# 13. Wildcard Administrative Accounts

FortiGate can allow remote LDAP users to authenticate as administrators.

Concept:

```text
System
  └── Administrators
       └── Match All Users from Remote Server Group
```

Example:

```text
Remote User Group → winsrv-2016
Admin Profile     → Super Admin
Username          → u1
```

---

# 14. Wildcard Admin Username

```text
Username → u1
```

# Defines the local FortiGate administrator entry used for matching the remote LDAP authentication identity.

### Important

```text
FortiGate Username
       =
LDAP Username
```

# The administrator identity must match the remote directory username used during authentication.

Example:

```text
FortiGate
    u1

LDAP
    u1
```

If the identities do not match:

```text
Authentication
       ↓
Username mismatch
       ↓
Administrative login failure
```

---

# 15. User Device Store — User Count

```bash
diagnose user-device-store user-count list 1
```

# Displays user-count information stored in the FortiGate user-device store.

---

# 16. User Device Store — Query

```bash
diagnose user-device-store user-count query "CN=callcenter,OU=information-tech,OU=publish-users,DC=test,DC=com"
```

# Queries user-count information for the specified LDAP distinguished name.

---

# 17. User Device Store — Statistics

```bash
diagnose user-device-store user-stats query 2026-03-01 20
```

# Queries stored user statistics for the specified time range.

> **Operational Note:** Retention and available historical statistics depend on FortiOS behavior and configuration.

---

# 18. Testing LDAP Authentication with Explicit Proxy

For testing user authentication, an explicit web proxy can be configured.

Concept:

```text
Client
  |
  ↓
Explicit Web Proxy
  |
  ↓
Authentication
  |
  ↓
LDAP
  |
  ↓
Active Directory
```

# Explicit proxy authentication can be used to force users through an authentication workflow.

---

# 19. Authentication Schema

```text
Authentication Schema
```

# Defines how FortiGate presents and performs the authentication process.

### Important

If the authentication schema or required explicit proxy policy is missing:

```text
Client
  ↓
Explicit Proxy
  ↓
No Authentication Rule
  ↓
Traffic Denied
```

# The explicit proxy authentication workflow requires the appropriate authentication configuration and policy.

---

# 20. Explicit Proxy Policy

```text
Proxy Policy
```

# Controls which users, destinations, schedules, and actions are permitted through the explicit proxy.

Basic example:

```text
Incoming/Source → Authenticated Clients
Outgoing        → WAN
Destination     → Internet
Action           → Accept
```

---

# 21. Identity-Based Firewall Policy

```bash
config firewall policy
    edit 1
        set identity-based enable
    end
end
```

# Enables identity-based behavior for the firewall policy where supported by the FortiOS release and policy design.

> **Important:** Authentication behavior depends on the exact FortiOS version, proxy configuration, and policy structure.

---

# 22. LDAP Polling AD

FortiGate can use LDAP polling to retrieve user authentication information directly from Active Directory.

```text
Security Fabric
    ↓
External Connectors
    ↓
Polling AD
```

# Polling AD allows FortiGate to query Active Directory through LDAP without requiring a DC Agent.

---

# 23. Polling AD — Architecture

```text
             Active Directory
                    |
                    |
                 LDAP
                    |
                    ↓
              FortiGate
                    |
              Polling AD
```

# FortiGate directly communicates with Active Directory to obtain user information.

### Requirements

```text
LDAP Server
LDAP Connectivity
Directory Credentials
```

# The FortiGate needs valid LDAP connectivity and an account with sufficient directory permissions.

---

# 24. Polling AD Credentials

A dedicated directory account can be used.

Example:

```text
Username → ssoadmin
```

# The LDAP service account is used by FortiGate to query Active Directory.

> **Best Practice:** Use a dedicated least-privileged service account instead of a highly privileged domain administrator account whenever possible.

---

# 25. Polling AD + Firewall Policy

When users access resources such as the Internet:

```text
User
 ↓
FortiGate
 ↓
Authentication
 ↓
LDAP / AD
 ↓
Firewall Policy
 ↓
Internet
```

# The firewall policy can use LDAP-backed user groups to enforce identity-based access.

---

# 26. FSSO Local Mode vs Polling Mode

These two mechanisms have different authentication architectures.

| Feature                    | FSSO Local Mode        | Polling AD                     |
| -------------------------- | ---------------------- | ------------------------------ |
| DC Agent                   | Commonly used          | Not required                   |
| Authentication information | Forwarded to FortiGate | Queried by FortiGate           |
| Automatic logon awareness  | Yes                    | Limited                        |
| User experience            | Transparent            | Authentication may be required |
| Large AD environments      | Better suited          | Less scalable                  |
| Simple AD environment      | Possible               | Simple option                  |

---

# 27. FSSO Local Mode

Concept:

```text
User Login
    ↓
Domain Controller
    ↓
FSSO Collector / Agent
    ↓
FortiGate
    ↓
User Identity
```

# FSSO local mode receives user logon information from the FSSO infrastructure and associates users with IP addresses.

---

# 28. Polling Mode

Concept:

```text
User
 ↓
FortiGate
 ↓
LDAP / AD
 ↓
User Information
```

# Polling mode allows FortiGate to query Active Directory directly instead of relying on a collector agent to forward logon information.

---

# 29. FSSO Local Mode — Main Advantage

```text
Automatic User Identification
```

# FSSO can provide transparent user identification based on domain logon information.

Example:

```text
User Login
     ↓
FSSO Detects Login
     ↓
User ↔ IP Mapping
     ↓
FortiGate
     ↓
Identity-Based Policy
```

---

# 30. Polling AD — Main Limitation

```text
Polling AD
     ↓
FortiGate queries AD
     ↓
Authentication / User information
```

# Polling mode is simpler but provides less flexible automatic logon awareness than a full FSSO deployment.

---

# 31. Large Active Directory Environment

For environments with many domain controllers or large user repositories:

```text
Multiple DCs
    ↓
FSSO Collector
    ↓
FortiGate
```

# A DC Agent/FSSO architecture is generally better suited to large environments where centralized logon collection is required.

---

# 32. Single Active Directory Environment

For a smaller environment:

```text
Single AD
    ↓
LDAP
    ↓
FortiGate
```

# Direct LDAP polling can be a simpler architecture when the environment is small and transparent FSSO behavior is not required.

---

# 33. Redundant Authentication Strategy

A hybrid design can provide an additional authentication path.

```text
             Active Directory
              /            \
             /              \
        FSSO/DC Agent      LDAP
             |              |
             ↓              ↓
             +---- FortiGate+
```

# LDAP polling can provide an alternative authentication mechanism when FSSO logon information is temporarily unavailable.

### Example Failure

```text
AD
 ↓
DC Agent
 ↓
FSSO Collector
 ↓
FortiGate

Collector unavailable
       ↓
FSSO information unavailable
       ↓
LDAP polling can provide
an alternative authentication path
```

---

# 34. LDAP Authentication — Troubleshooting Flow

```text
LDAP Authentication Failure
          |
          +── Connectivity?
          |
          +── TCP/389 or LDAPS?
          |
          +── Bind credentials?
          |
          +── Base DN?
          |
          +── Search type?
          |
          +── Username attribute?
          |
          +── LDAP group?
          |
          +── Firewall policy?
          |
          +── Authentication schema?
          |
          +── Certificate/SAN?
```

# Use this flow to isolate LDAP connectivity, directory search, identity matching, and policy authentication problems.

---

# 35. LDAP Troubleshooting Checklist

```text
✓ Verify LDAP server IP
✓ Verify LDAP port
✓ Verify bind credentials
✓ Verify Base DN
✓ Verify LDAP search type
✓ Verify username attribute
✓ Test connection
✓ Test browse
✓ Test query
✓ Test user authentication
✓ Verify LDAP group
✓ Verify firewall policy
✓ Verify authentication schema
✓ Verify certificate/SAN for LDAPS
```

# These checks cover the main LDAP authentication failure points.

---

# 36. Core LDAP Mental Model

```text
                    LDAP
                     │
          ┌──────────┼──────────┐
          │          │          │
       Server       Bind       Search
          │          │          │
       IP/Port    Credentials   DN
          │          │          │
          └──────────┼──────────┘
                     │
                  User
                     │
             LDAP Attribute
                     │
              User / Group
                     │
             FortiGate Policy
                     │
                  Access
```

# LDAP authentication is a chain of connectivity, binding, directory search, identity matching, and policy authorization.

---

# 37. FSSO vs LDAP — Fast Memory Map

```text
LDAP
│
├── Authentication
├── Directory Query
├── User/Group Lookup
└── Direct FortiGate → AD communication


FSSO
│
├── User Logon Detection
├── User ↔ IP Mapping
├── Transparent Authentication
└── Collector/DC Agent


Polling AD
│
├── No DC Agent required
├── FortiGate queries AD
├── Simple architecture
└── Less transparent than FSSO
```

# Memorize the architectural difference between LDAP authentication, FSSO logon awareness, and AD polling.

---

# 38. Golden Rules

> **1. LDAP provides directory-based authentication and user/group lookup.**

> **2. Always verify LDAP connectivity before troubleshooting authentication logic.**

> **3. Base DN determines where FortiGate searches in the directory.**

> **4. Recursive search allows subtree-oriented LDAP searches where supported.**

> **5. `sAMAccountName`, `cn`, and `uid` are common identity attributes.**

> **6. LDAP group matching must align with the Active Directory structure.**

> **7. LDAP authentication alone does not automatically mean transparent user identification.**

> **8. FSSO is primarily about detecting user logons and mapping users to IP addresses.**

> **9. Polling AD allows FortiGate to query Active Directory directly without a DC Agent.**

> **10. Use dedicated, least-privileged LDAP service accounts.**

> **11. Never publish real LDAP passwords or production credentials.**

> **12. For LDAPS, certificate identity validation and SAN configuration are critical.**

> **13. Firewall policies must reference the appropriate LDAP-backed user/group when identity-based access is required.**

> **14. Large AD environments generally benefit from an FSSO/DC Agent architecture.**

> **15. LDAP polling can be considered as an alternative authentication path when FSSO information is unavailable.**

---

# 39. Ultra-Fast LDAP

```text
LDAP
│
├── Server
│   ├── IP
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
│   ├── DN
│   ├── Subtree
│   └── Recursive
│
├── Attributes
│   ├── cn
│   ├── sAMAccountName
│   └── uid
│
├── Groups
│   ├── LDAP Server
│   ├── Remote Group
│   └── Firewall Policy
│
├── Authentication
│   ├── LDAP
│   ├── Explicit Proxy
│   └── Identity-Based Policy
│
├── AD Polling
│   ├── No DC Agent
│   ├── Direct LDAP Query
│   └── Simple Architecture
│
└── FSSO
    ├── DC Agent
    ├── Collector
    ├── User ↔ IP
    └── Transparent Authentication
```

# The complete LDAP mental model connects the directory server, search mechanism, user attributes, groups, authentication, and FortiGate security policies.

---

# 40. Most Important LDAP Commands

```bash
config user ldap
    edit "winsrv-2016"
        set search-type recursive
    end
```

# Enables recursive LDAP directory searching for the selected LDAP server.

```bash
diagnose test authserver ldap winsrv-2016 <username> <password>
```

# Tests authentication of a specific user against the LDAP authentication server.

```bash
config user ldap
    edit "winsrv-2016"
        set member-attr "msNPAllowDialin"
    end
```

# Defines the Active Directory member attribute used for the LDAP authentication logic.

```bash
config user group
    edit "ldap-g-local"
        set member "winsrv-2016"
    end
end
```

# Associates the LDAP authentication server with a FortiGate user group.

```bash
diagnose user-device-store user-count list 1
```

# Displays stored user-count information from the user-device store.

```bash
diagnose user-device-store user-count query "<LDAP-DN>"
```

# Queries user-count information for a specific LDAP distinguished name.

```bash
diagnose user-device-store user-stats query <date> <value>
```

# Queries stored user statistics for the requested time range.

---

# 41. SheynShield LDAP Mental Model

```text
                 ACTIVE DIRECTORY
                        │
                        │ LDAP
                        ↓
              ┌─────────────────┐
              │    FortiGate    │
              └─────────────────┘
                        │
             ┌──────────┼──────────┐
             │          │          │
           Bind       Search    Attributes
             │          │          │
             ↓          ↓          ↓
         Credentials    DN      cn/sAMAccountName
                                  /uid
             │          │          │
             └──────────┼──────────┘
                        ↓
                  LDAP User/Group
                        │
                        ↓
                 Firewall Policy
                        │
                        ↓
                      ACCESS
```

### Authentication Architecture

```text
              USER
                │
        ┌───────┴────────┐
        │                │
       FSSO             LDAP
        │                │
  User ↔ IP          Authentication
        │                │
        └───────┬────────┘
                ↓
             FortiGate
                │
                ↓
             POLICY
                │
                ↓
              ACCESS
```

### Core Distinction

```text
LDAP
→ "Can I authenticate/query this user?"

FSSO
→ "Which user is associated with this IP?"

Polling AD
→ "Can FortiGate directly query AD for user information?"

LDAP Group
→ "Which users belong to this authorization group?"

Firewall Policy
→ "What is this authenticated user allowed to access?"
```

---

## SheynShield — Engineering Secure Networks

**LDAP → Identity → Group → Policy → Access**

**FSSO → Logon Detection → User/IP Mapping → Identity Policy**

**Polling AD → Direct LDAP Query → User Information → Policy**

# 🔐 FortiGate RADIUS Server — Authentication, RSSO, VSA & Dynamic Shaping

> **SheynShield | Security & Design Knowledge Base**
> **FortiGate / FortiOS | NSE4 → NSE7**
>
> Practical **RADIUS Server checklist** covering **RADIUS authentication, NPS, Active Directory, VSA, AVP, RSSO, RADIUS Accounting, CoA, group mapping, dynamic shaping, wireless attributes, troubleshooting, and FortiGate CLI**.

---

## 📌 Table of Contents

* [1. RADIUS Architecture](#1-radius-architecture)
* [2. RADIUS Ports](#2-radius-ports)
* [3. FortiGate RADIUS Configuration](#3-fortigate-radius-configuration)
* [4. RADIUS User Groups](#4-radius-user-groups)
* [5. Username Encoding & Case Sensitivity](#5-username-encoding--case-sensitivity)
* [6. AVP vs VSA](#6-avp-vs-vsa)
* [7. Fortinet VSA](#7-fortinet-vsa)
* [8. NPS Group Mapping](#8-nps-group-mapping)
* [9. RADIUS Accounting](#9-radius-accounting)
* [10. RSSO](#10-rsso)
* [11. RSSO vs FSSO](#11-rsso-vs-fsso)
* [12. RADIUS CoA](#12-radius-coa)
* [13. Dynamic Traffic Shaping](#13-dynamic-traffic-shaping)
* [14. Wireless RADIUS Attributes](#14-wireless-radius-attributes)
* [15. Termination Action](#15-termination-action)
* [16. Filter-Id & Group Mapping](#16-filter-id--group-mapping)
* [17. RSA SecurID / ACE](#17-rsa-securid--ace)
* [18. Troubleshooting Checklist](#18-troubleshooting-checklist)
* [19. RADIUS Troubleshooting Matrix](#19-radius-troubleshooting-matrix)
* [20. End-to-End NPS → FortiGate](#20-end-to-end-nps--fortigate)
* [21. NSE Exam Gotchas](#21-nse-exam-gotchas)
* [22. Production Security Checklist](#22-production-security-checklist)
* [23. Quick CLI Reference](#23-quick-cli-reference)
* [24. One-Page Memory Map](#24-one-page-memory-map)

---

# 1. RADIUS Architecture

**RADIUS — Remote Authentication Dial-In User Service** provides centralized:

* Authentication
* Authorization
* Accounting

Typical FortiGate architecture:

```text
                    ┌─────────────────────┐
                    │ Active Directory    │
                    │       / LDAP        │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │        NPS          │
                    │      RADIUS         │
                    └──────────┬──────────┘
                               │
                 ┌─────────────┴─────────────┐
                 │                           │
              UDP 1812                   UDP 1813
            Authentication               Accounting
                 │                           │
                 └─────────────┬─────────────┘
                               ▼
                    ┌─────────────────────┐
                    │     FortiGate       │
                    │                     │
                    │ Authentication      │
                    │ Group Mapping       │
                    │ RSSO                │
                    │ Dynamic Shaping      │
                    └──────────┬──────────┘
                               │
                               ▼
                       Firewall Policy
                               │
                               ▼
                            Access
```

### RADIUS Deployment Checklist

* [ ] RADIUS/NPS server is available
* [ ] Active Directory integration is working
* [ ] FortiGate is defined as a RADIUS client/NAS
* [ ] Shared secret is configured consistently
* [ ] Authentication port is reachable
* [ ] Accounting port is reachable when required
* [ ] NPS policies are configured
* [ ] FortiGate user group is configured
* [ ] Firewall policy references the correct group

---

# 2. RADIUS Ports

| Function                       | UDP Port |
| ------------------------------ | -------: |
| Authentication / Authorization |   `1812` |
| Accounting                     |   `1813` |
| Legacy Authentication          |   `1645` |
| Legacy Accounting              |   `1646` |

### Port Validation

* [ ] UDP/1812 allowed from FortiGate to RADIUS
* [ ] UDP/1813 allowed when accounting is required
* [ ] Legacy ports are used only when required
* [ ] Routing between FortiGate and RADIUS is correct
* [ ] Intermediate firewalls permit required traffic

### Memory Rule

```text
1812 → Authentication
1813 → Accounting

1645 → Legacy Authentication
1646 → Legacy Accounting
```

> ⚠️ **NSE Gotcha:** Authentication can work perfectly while accounting-dependent features fail because UDP/1813 is unavailable.

---

# 3. FortiGate RADIUS Configuration

### GUI

```text
User & Authentication
└── RADIUS Servers
    └── Create New
```

Example:

```text
Name:              rad-1
Server IP:         192.168.20.200
Authentication:    MSCHAPv2
NAS IP:            192.168.20.254
Auth Port:         1812
Accounting Port:   1813
```

### Configuration Checklist

* [ ] RADIUS object created
* [ ] Correct server IP configured
* [ ] Correct authentication method selected
* [ ] Shared secret configured
* [ ] NAS IP configured correctly when required
* [ ] Authentication port verified
* [ ] Accounting port verified
* [ ] NPS recognizes FortiGate as a RADIUS client

### CLI

```bash
config user radius
    edit "rad-1"
        set server "192.168.20.200"
        set secret <RADIUS_SECRET>
    next
end
```

> 🔐 Never commit a real RADIUS shared secret to GitHub.

---

# 4. RADIUS User Groups

Create a FortiGate remote authentication group:

```text
User & Authentication
└── User Groups
    └── Create New
```

Example:

```text
Group Name:      rad-g-1
Remote Server:   rad-1
```

### Group Checklist

* [ ] RADIUS server selected
* [ ] Correct remote group configuration
* [ ] Returned RADIUS attributes understood
* [ ] Group referenced by firewall policy
* [ ] NPS group membership verified

### Identity Chain

```text
Active Directory
      ↓
AD Group
      ↓
NPS Policy
      ↓
RADIUS Response
      ↓
FortiGate Group
      ↓
Firewall Policy
      ↓
Access
```

> 💡 **Critical troubleshooting rule:**
> `Authentication = SUCCESS` does not necessarily mean `Firewall Policy Match = SUCCESS`.

---

# 5. Username Encoding & Case Sensitivity

Example:

```bash
config user radius
    edit "rad-1"
        set password-encoding auto
        set username-case-sensitive enable
    next
end
```

### Checklist

* [ ] Password encoding matches the deployment
* [ ] Username format is consistent
* [ ] Username case behavior is understood
* [ ] AD username format is verified
* [ ] NPS username handling is verified

### Case Sensitivity Example

```text
User01
user01
USER01
```

Depending on the FortiOS/RADIUS/backend behavior, these may not be treated identically.

### Recommended Design

```text
FortiGate
   ↓
RADIUS
   ↓
NPS
   ↓
Active Directory
```

Keep username normalization and case handling consistent across the entire authentication chain.

---

# 6. AVP vs VSA

## AVP — Attribute-Value Pair

Standard RADIUS attributes follow:

```text
Attribute = Value
```

Common attributes:

| Attribute          | Code | Purpose                        |
| ------------------ | ---: | ------------------------------ |
| User-Name          |  `1` | Username                       |
| NAS-IP-Address     |  `4` | NAS address                    |
| Framed-IP-Address  |  `8` | Client IP                      |
| Class              | `25` | Session/accounting correlation |
| Vendor-Specific    | `26` | Vendor attribute container     |
| NAS-Identifier     | `32` | NAS identifier                 |
| Acct-Input-Octets  | `42` | Received octets                |
| Acct-Output-Octets | `43` | Transmitted octets             |
| Acct-Session-Id    | `44` | Session identifier             |
| Event-Timestamp    | `55` | Event timestamp                |

### AVP Checklist

* [ ] Attribute is supported
* [ ] Attribute code is correct
* [ ] Attribute value is correct
* [ ] FortiGate interprets the attribute as expected
* [ ] NPS policy returns the required attribute

---

# 7. Fortinet VSA

**VSA = Vendor-Specific Attribute**

Fortinet-specific RADIUS information can be carried using the RADIUS Vendor-Specific Attribute mechanism.

```text
RADIUS Attribute 26
        │
        ▼
Fortinet VSA
        │
        ├── Group
        ├── Client IP
        ├── VDOM
        ├── IPv6 information
        ├── Interface
        └── Access Profile
```

### Fortinet Vendor Information

```text
Vendor:
Fortinet

Vendor-ID:
12356
```

### Common Fortinet VSA Concepts

| VSA | Attribute                      | Purpose                       |
| --: | ------------------------------ | ----------------------------- |
| `1` | `fortinet-group-name`          | Remote group mapping          |
| `2` | `fortinet-client-ip-address`   | Client IP                     |
| `3` | `fortinet-vdom-name`           | VDOM information              |
| `4` | `fortinet-client-ipv6-address` | IPv6 client information       |
| `5` | `fortinet-interface-name`      | Interface information         |
| `6` | `fortinet-access-profile`      | Administrative access profile |

### VSA Checklist

* [ ] Fortinet vendor ID is correct
* [ ] Correct VSA is returned by RADIUS
* [ ] NPS vendor-specific attribute is configured correctly
* [ ] Attribute value matches FortiGate expectations
* [ ] Returned VSA is visible during troubleshooting
* [ ] Group mapping uses the correct returned value

---

# 8. NPS Group Mapping

A common enterprise design is:

```text
Active Directory Group
          ↓
      NPS Policy
          ↓
    RADIUS Response
          ↓
    Fortinet VSA
          ↓
     FortiGate Group
          ↓
    Firewall Policy
```

Example:

```text
AD User:
    alice

AD Group:
    IT

NPS:
    Match IT group

RADIUS:
    Fortinet-Group-Name = IT

FortiGate:
    Remote Group = IT
```

### NPS Mapping Checklist

* [ ] User belongs to expected AD group
* [ ] NPS policy matches the user
* [ ] NPS policy returns expected attribute
* [ ] Fortinet VSA is correctly configured
* [ ] Returned group name is correct
* [ ] FortiGate remote group matches returned value
* [ ] Firewall policy references the expected group

### Troubleshooting Priority

If:

```text
Authentication = SUCCESS
Policy Match    = FAILURE
```

check:

```text
NPS Policy
   ↓
RADIUS Attributes
   ↓
VSA / Filter-Id
   ↓
FortiGate Group
   ↓
Firewall Policy
```

before assuming the RADIUS password authentication itself is broken.

---

# 9. RADIUS Accounting

RADIUS Accounting provides session information such as:

* Session start
* Interim updates
* Session stop
* User/session correlation
* Usage information
* RSSO information
* Dynamic policy context

```text
FortiGate
   │
   │ Accounting-Request
   ▼
RADIUS Server
   │
   ├── Start
   ├── Interim-Update
   └── Stop
```

### Accounting Checklist

* [ ] UDP/1813 reachable
* [ ] Accounting server configured
* [ ] Accounting secret matches
* [ ] Start packets received
* [ ] Interim updates configured where required
* [ ] Stop packets received
* [ ] Session IDs are consistent
* [ ] Accounting data is visible on the RADIUS server

---

# 10. RSSO

**RSSO = RADIUS Single Sign-On**

RSSO allows FortiGate to obtain user/session information associated with RADIUS authentication and use that identity for policy enforcement.

### GUI

```text
Security Fabric
└── External Connectors
    └── RADIUS Single Sign-On
```

Example:

```text
Name:
    rsso-agent
```

Then create/use a corresponding FortiGate user group.

```text
User & Authentication
└── User Groups
    └── rsso-f
```

### RSSO Flow

```text
User
  ↓
Network Access Device
  ↓
RADIUS Authentication
  ↓
RADIUS Server
  ↓
Accounting / Session Information
  ↓
FortiGate RSSO
  ↓
User ↔ IP Association
  ↓
FortiGate User Group
  ↓
Firewall Policy
```

### RSSO Checklist

* [ ] RSSO connector configured
* [ ] RADIUS authentication works
* [ ] RADIUS accounting works
* [ ] User/session information reaches FortiGate
* [ ] User-to-IP association is learned
* [ ] RSSO group is configured
* [ ] Firewall policy references RSSO identity

---

# 11. RSSO vs FSSO

| Feature                  | RSSO                | FSSO Polling | FSSO Collector  |
| ------------------------ | ------------------- | ------------ | --------------- |
| Primary source           | RADIUS              | AD           | AD/DC           |
| Authentication awareness | RADIUS-based        | Polling      | Logon events    |
| Main identity source     | RADIUS session      | AD           | AD              |
| Best fit                 | RADIUS environments | Simple AD    | Enterprise AD   |
| Session awareness        | RADIUS-based        | AD polling   | AD logon events |

### Design Rule

```text
RADIUS infrastructure
        ↓
       RSSO

Simple AD environment
        ↓
   FSSO Polling

Enterprise AD / multiple DCs
        ↓
 FSSO Collector / DC Agent
```

---

# 12. RADIUS CoA

**CoA = Change of Authorization**

CoA allows the RADIUS infrastructure to request changes to an existing authenticated session.

```text
RADIUS
   │
   │ CoA
   ▼
FortiGate
   │
   ├── Update authorization
   ├── Update session
   └── Apply supported attributes
```

### CoA Use Cases

* [ ] Dynamic authorization
* [ ] User policy changes
* [ ] Bandwidth changes
* [ ] Wireless environments
* [ ] Session updates
* [ ] Post-authentication authorization

### CoA Checklist

* [ ] CoA supported by target FortiOS release
* [ ] RADIUS server supports required CoA behavior
* [ ] FortiGate CoA configuration verified
* [ ] Required network path is available
* [ ] Shared secret/configuration is correct
* [ ] Session is eligible for the requested update

---

# 13. Dynamic Traffic Shaping

RADIUS attributes can participate in dynamic traffic-shaping designs.

Example:

```bash
config user radius
    edit "rad-1"
        set radius-coa enable
        set radius-all-server enable
    next
end
```

> ⚠️ Exact CLI availability and behavior can vary by FortiOS release. Validate commands against the target FortiOS version.

### Firewall Policy

Example:

```bash
config firewall policy
    edit 1
        set dynamic-shaping enable
    next
end
```

### Dynamic Shaping Flow

```text
RADIUS User
     ↓
Identity
     ↓
RADIUS Attributes
     ↓
FortiGate Session
     ↓
Firewall Policy
     ↓
Dynamic Shaper
     ↓
User Traffic
```

Example:

```text
User A → 20 Mbps
User B → 10 Mbps
User C →  5 Mbps
```

### Dynamic Shaping Checklist

* [ ] User identity is correctly learned
* [ ] Required RADIUS attributes are returned
* [ ] Accounting is working when required
* [ ] Policy supports dynamic shaping
* [ ] Dynamic shaping is enabled
* [ ] User/session is associated correctly
* [ ] Shaper statistics confirm enforcement

---

# 14. Wireless RADIUS Attributes

Wireless environments may use additional attributes to carry information related to:

* SSID
* Wireless controller/device
* Wireless termination point
* Association time

Fortinet-specific wireless information may include:

| Code | Concept                    |
| ---: | -------------------------- |
|  `7` | Fortinet SSID              |
| `23` | Wireless device/controller |
| `24` | Wireless termination point |
| `25` | Association time           |

### Wireless Troubleshooting Checklist

* [ ] RADIUS authentication works
* [ ] Accounting is received
* [ ] Wireless client association is correct
* [ ] AP/controller information is correct
* [ ] Session state is synchronized
* [ ] Roaming behavior is validated
* [ ] Dynamic shaping is verified

---

# 15. Termination Action

Termination behavior affects how session state changes are handled.

Conceptual values:

| Value | Concept                                      |
| ----: | -------------------------------------------- |
|   `0` | Default/normal behavior                      |
|   `1` | RADIUS request/session verification behavior |
|   `2` | Resource/session management and cleanup      |

> ⚠️ Treat these values as **FortiOS-version and deployment dependent**. Validate the exact behavior against the target release and RADIUS integration.

### Deployment Checklist

* [ ] Required termination behavior is identified
* [ ] FortiOS version is verified
* [ ] RADIUS server behavior is verified
* [ ] Wireless session behavior is tested
* [ ] Stale-session scenarios are tested

---

# 16. Filter-Id & Group Mapping

RADIUS can return attributes used by FortiGate for identity/group decisions.

Example:

```text
RADIUS
   │
   └── Filter-Id = IT
             │
             ▼
      FortiGate Group
             │
             ▼
      Firewall Policy
```

### Mapping Checklist

* [ ] Returned attribute is expected
* [ ] Attribute value is correct
* [ ] FortiGate group expects the same value
* [ ] NPS policy returns the attribute
* [ ] Firewall policy references the correct group

### Important

```text
AD Group
   ↓
NPS Policy
   ↓
RADIUS Attribute
   ↓
FortiGate Group
   ↓
Firewall Policy
```

Any mismatch can cause policy selection failure.

---

# 17. RSA SecurID / ACE

RSA SecurID provides OTP-based authentication.

Conceptual architecture:

```text
User
 │
 ├── Username
 └── OTP
      │
      ▼
FortiGate
      │
      ▼
RADIUS
      │
      ▼
RSA Authentication Infrastructure
```

### Diagnostic Test

```bash
diagnose test authserver radius <server> <username> <password>
```

### Security Checklist

* [ ] OTP integration is validated
* [ ] RADIUS authentication works
* [ ] Token/OTP policy is correct
* [ ] No OTP secrets are stored in documentation
* [ ] Production credentials are never committed to GitHub

> 🔐 **Never publish real passwords, OTP seeds, RADIUS secrets, certificates containing private keys, or production credentials.**

---

# 18. Troubleshooting Checklist

## Step 1 — Network Reachability

* [ ] FortiGate can reach RADIUS server
* [ ] Correct routing exists
* [ ] UDP/1812 is allowed
* [ ] UDP/1813 is allowed when required
* [ ] No intermediate firewall blocks RADIUS

---

## Step 2 — RADIUS Server

* [ ] RADIUS server is online
* [ ] FortiGate is configured as a RADIUS client
* [ ] Shared secret matches
* [ ] NAS IP is correct
* [ ] Authentication method matches
* [ ] NPS policy is enabled

---

## Step 3 — Authentication

Run:

```bash
diagnose test authserver radius <server> <username> <password>
```

Validate:

* [ ] Authentication request reaches server
* [ ] Server returns Access-Accept
* [ ] No Access-Reject
* [ ] No unexpected Access-Challenge
* [ ] Authentication method is compatible

---

## Step 4 — Group Mapping

If authentication succeeds:

```text
Authentication
      ↓
      ?
FortiGate Group
```

Check:

* [ ] NPS policy
* [ ] AD group membership
* [ ] VSA
* [ ] Filter-Id
* [ ] Returned attributes
* [ ] FortiGate remote group
* [ ] Firewall policy

---

## Step 5 — RSSO

If RSSO fails:

* [ ] Authentication works
* [ ] Accounting works
* [ ] UDP/1813 is reachable
* [ ] RSSO connector is enabled
* [ ] Session information is received
* [ ] User ↔ IP association exists
* [ ] RSSO group is correct

---

## Step 6 — Dynamic Shaping

Check:

```bash
diagnose firewall shaper dynamic-shaper stats
```

Then:

```bash
diagnose firewall shaper dynamic-shaper list
```

Specific client:

```bash
diagnose firewall shaper dynamic-shaper list ip 192.168.20.20
```

Validate:

* [ ] Correct user identity
* [ ] Correct policy
* [ ] Required RADIUS attributes
* [ ] Correct shaper
* [ ] Active session
* [ ] Shaper statistics

---

## Step 7 — Session

Inspect:

```bash
diagnose sys session list
```

Then correlate:

```text
User
 ↓
Authentication
 ↓
Group
 ↓
Policy
 ↓
Session
 ↓
Shaper
```

---

# 19. RADIUS Troubleshooting Matrix

| Symptom                                      | Primary Checks                                      |
| -------------------------------------------- | --------------------------------------------------- |
| Authentication fails                         | Server IP, secret, port, authentication method      |
| RADIUS timeout                               | Routing, firewall, UDP/1812                         |
| Accounting missing                           | UDP/1813, accounting configuration                  |
| User authenticates but wrong policy matches  | Group mapping, VSA, Filter-Id                       |
| RSSO user not detected                       | Accounting, RSSO configuration, user/IP association |
| Dynamic shaping fails                        | RADIUS attributes, accounting, policy, shaper       |
| Wrong group                                  | NPS policy, VSA, Filter-Id                          |
| Wireless session remains stale               | Accounting, termination behavior                    |
| CoA fails                                    | CoA configuration, reachability, server support     |
| Username mismatch                            | Case sensitivity, username format                   |
| MSCHAPv2 fails                               | NPS policy, AD integration, authentication method   |
| Policy does not match after successful login | Group mapping and returned attributes               |

---

# 20. End-to-End NPS → FortiGate

```text
                    Active Directory
                           │
                           ▼
                    ┌─────────────┐
                    │     NPS     │
                    │   RADIUS    │
                    └──────┬──────┘
                           │
              ┌────────────┴────────────┐
              │                         │
          UDP 1812                  UDP 1813
       Authentication             Accounting
              │                         │
              └────────────┬────────────┘
                           ▼
                    ┌─────────────┐
                    │  FortiGate  │
                    └──────┬──────┘
                           │
          ┌────────────────┼─────────────────┐
          │                │                 │
          ▼                ▼                 ▼
       Groups             RSSO        Dynamic Shaping
          │                │                 │
          └────────────────┼─────────────────┘
                           ▼
                    Firewall Policy
                           │
                           ▼
                          User
```

### End-to-End Validation

* [ ] AD user exists
* [ ] AD group membership is correct
* [ ] NPS receives request
* [ ] NPS policy matches
* [ ] Access-Accept is returned
* [ ] Required AVP/VSA is returned
* [ ] FortiGate recognizes the user
* [ ] Correct FortiGate group is selected
* [ ] Correct firewall policy matches
* [ ] Accounting starts
* [ ] RSSO learns the session when required
* [ ] Dynamic shaping applies when configured

---

# 21. NSE Exam Gotchas

## ⚠️ Gotcha #1 — Authentication ≠ Authorization

```text
Authentication
    ↓
Who are you?

Authorization
    ↓
What can you access?
```

Therefore:

```text
RADIUS Authentication = SUCCESS
```

does **not** guarantee:

```text
Firewall Policy Match = SUCCESS
```

---

## ⚠️ Gotcha #2 — 1812 vs 1813

```text
1812 → Authentication
1813 → Accounting
```

If authentication works but RSSO/accounting-dependent behavior fails:

```text
Check UDP/1813
```

---

## ⚠️ Gotcha #3 — AVP vs VSA

```text
AVP
 ↓
Standard RADIUS Attribute

VSA
 ↓
Vendor-Specific Attribute
```

Fortinet-specific information commonly uses the vendor-specific mechanism.

---

## ⚠️ Gotcha #4 — Authentication Success Can Still Mean Policy Failure

Always inspect:

```text
AD
 ↓
NPS
 ↓
RADIUS Attribute
 ↓
FortiGate Group
 ↓
Firewall Policy
```

---

## ⚠️ Gotcha #5 — RSSO Is Session-Aware

RSSO is not simply:

```text
Username + Password
```

Think:

```text
RADIUS Authentication
       +
Accounting / Session Information
       ↓
FortiGate User ↔ IP Awareness
```

---

## ⚠️ Gotcha #6 — Dynamic Shaping Depends on Identity

If authentication works but shaping does not:

```text
RADIUS Attributes
        ↓
Accounting
        ↓
User / Session
        ↓
Firewall Policy
        ↓
Dynamic Shaper
```

Trace the complete chain.

---

# 22. Production Security Checklist

## RADIUS Server

* [ ] RADIUS server IP is correct
* [ ] Shared secret is strong
* [ ] Shared secret matches on both sides
* [ ] UDP/1812 is restricted to trusted clients
* [ ] UDP/1813 is restricted to trusted clients
* [ ] NAS IP is correct
* [ ] NPS policies are least-privilege
* [ ] Authentication method is appropriate
* [ ] AD integration is functioning
* [ ] Authentication logging is enabled

## FortiGate

* [ ] RADIUS server object configured
* [ ] Correct RADIUS group configured
* [ ] Firewall policy references correct group
* [ ] Required VSA/AVP mappings validated
* [ ] Accounting configured where required
* [ ] RSSO configured where required
* [ ] CoA configured where required
* [ ] Dynamic shaping configured where required
* [ ] Authentication troubleshooting logs available

## Security

* [ ] Never publish RADIUS shared secrets
* [ ] Never publish production passwords
* [ ] Never publish OTP seeds
* [ ] Never publish private keys
* [ ] Restrict RADIUS clients to trusted NAS devices
* [ ] Monitor authentication failures
* [ ] Monitor abnormal accounting traffic
* [ ] Review NPS policies regularly
* [ ] Use secure management access
* [ ] Validate supported secure RADIUS options for the target environment

---

# 23. Quick CLI Reference

## RADIUS Server

```bash
config user radius
    edit "rad-1"
        set server "192.168.20.200"
        set secret <RADIUS_SECRET>
    next
end
```

## Username Handling

```bash
config user radius
    edit "rad-1"
        set password-encoding auto
        set username-case-sensitive enable
    next
end
```

## Accounting

```bash
config user radius
    edit "rad-1"
        config accounting-server
            edit 1
                set status enable
                set server 192.168.20.200
                set port 1813
                set secret <RADIUS_SECRET>
            next
        end
    next
end
```

## CoA

```bash
config user radius
    edit "rad-1"
        set radius-coa enable
        set radius-all-server enable
    next
end
```

> ⚠️ Verify exact syntax against the target FortiOS release.

## Authentication Test

```bash
diagnose test authserver radius <server> <username> <password>
```

## Authenticated Users

```bash
diagnose firewall auth list
```

## Sessions

```bash
diagnose sys session list
```

## Dynamic Shaper Statistics

```bash
diagnose firewall shaper dynamic-shaper stats
```

## Dynamic Shaper List

```bash
diagnose firewall shaper dynamic-shaper list
```

## Specific Client

```bash
diagnose firewall shaper dynamic-shaper list ip 192.168.20.20
```

## User Device Store

```bash
diagnose user-device-store user-count list 1
```

Example:

```bash
diagnose user-device-store user-count query \
"CN=callcenter,OU=information-tech,OU=publish-users,DC=test,DC=com"
```

## Wireless Clients

```bash
diagnose wireless-controller wlac-d sta online
```

---

# 24. One-Page Memory Map

```text
                         RADIUS
                           │
             ┌─────────────┼─────────────┐
             │             │             │
            1812          1813          VSA
             │             │             │
      Authentication   Accounting    Vendor Data
             │             │             │
             └─────────────┼─────────────┘
                           │
                           ▼
                       FortiGate
                           │
              ┌────────────┼────────────┐
              │            │            │
           Groups         RSSO         CoA
              │            │            │
              │       User ↔ IP     Authorization
              │
              ▼
       Firewall Policy
              │
              ▼
           Session
              │
              ▼
      Dynamic Shaping
```

---

# 🧠 NSE4 → NSE7 Mental Model

```text
                    RADIUS
                       │
       ┌───────────────┼────────────────┐
       │               │                │
    1812/AAA         1813             VSA
       │               │                │
 Authentication    Accounting      Vendor Data
       │               │                │
       └───────────────┼────────────────┘
                       │
                       ▼
                    NPS/AD
                       │
                       ▼
                 Group Mapping
                       │
                       ▼
                  FortiGate
                       │
          ┌────────────┼────────────┐
          │            │            │
        Policy        RSSO         CoA
          │            │            │
          ▼            ▼            ▼
       Access       User/IP      Dynamic Auth
          │
          ▼
       Session
          │
          ▼
   Dynamic Shaping
```

---

# 🔥 Final SheynShield Checklist

## RADIUS Authentication

* [ ] RADIUS server reachable
* [ ] UDP/1812 reachable
* [ ] Shared secret correct
* [ ] NAS identity correct
* [ ] Authentication method correct
* [ ] NPS policy matches
* [ ] AD authentication succeeds

## Identity

* [ ] Username format verified
* [ ] Case sensitivity understood
* [ ] Required AVP/VSA returned
* [ ] Group mapping verified
* [ ] FortiGate group matches returned identity

## Accounting

* [ ] UDP/1813 reachable
* [ ] Accounting enabled
* [ ] Start packet received
* [ ] Interim update received when required
* [ ] Stop packet received

## RSSO

* [ ] RSSO configured
* [ ] Accounting working
* [ ] User/IP association learned
* [ ] RSSO group configured
* [ ] Policy references RSSO identity

## Dynamic Authorization

* [ ] CoA supported and configured
* [ ] Required attributes returned
* [ ] Session can receive updates
* [ ] Authorization change is verified

## Dynamic Shaping

* [ ] Dynamic shaping enabled
* [ ] Correct user/session identified
* [ ] Required RADIUS attributes present
* [ ] Correct firewall policy matched
* [ ] Shaper statistics verified

---

# 🎯 SheynShield Takeaway

> **FortiGate RADIUS is much more than username/password authentication.**

The enterprise identity chain is:

```text
Active Directory
      ↓
NPS
      ↓
RADIUS Authentication
      ↓
AVP / VSA
      ↓
FortiGate Group
      ↓
Firewall Policy
      ↓
Session
      ↓
Accounting / RSSO
      ↓
CoA / Dynamic Authorization
      ↓
Dynamic Shaping
```

### 🔥 The 7 Things to Remember

```text
1. 1812 = Authentication
2. 1813 = Accounting
3. AVP = Standard RADIUS Attribute
4. VSA = Vendor-Specific Attribute
5. RSSO = RADIUS-based user/session awareness
6. CoA = Change of Authorization
7. Authentication Success ≠ Policy Match Success
```

### 🚨 Golden Troubleshooting Rule

```text
If RADIUS authentication succeeds
but the user receives the wrong access:

DO NOT STOP AT RADIUS.

Trace:

AD
 ↓
NPS Policy
 ↓
RADIUS Response
 ↓
AVP / VSA
 ↓
FortiGate Group
 ↓
Firewall Policy
 ↓
Session
 ↓
Accounting / RSSO
 ↓
Shaper / Authorization
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

`FortiGate` · `RADIUS` · `NPS` · `Active Directory` · `RSSO` · `VSA` · `AVP` · `CoA` · `Dynamic Shaping` · `Fortinet` · `Network Security` · `NSE4` · `NSE7`

# 🔐 RADIUS Server — FortiGate  

> **SheynShield | Security & Design Knowledge Base**
> **Topic:** RADIUS Authentication, RSSO, VSA/AVP & Dynamic Shaping
> **Platform:** FortiGate / FortiOS
> **Level:** NSE4 → NSE7
> **Keywords:** RADIUS, Remote Authentication, NPS, VSA, AVP, RSSO, Dynamic Shaping, Accounting, Fortinet VSA

---

## 1. RADIUS — Core Concept

**RADIUS (Remote Authentication Dial-In User Service)** provides centralized authentication, authorization, and accounting for network devices and security systems.

### Standard Ports

| Function                       | UDP Port |
| ------------------------------ | -------: |
| Authentication / Authorization |   `1812` |
| Accounting                     |   `1813` |
| Legacy Authentication          |   `1645` |
| Legacy Accounting              |   `1646` |

### Authentication Flow

```text
User
  │
  ▼
FortiGate
  │
  │ RADIUS Access-Request
  ▼
RADIUS / NPS
  │
  ├── Access-Accept
  ├── Access-Reject
  └── Access-Challenge
```

For accounting:

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

---

# 2. FortiGate RADIUS Server Configuration

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
Groups:            All Groups
```

### Important Parameters

| Parameter       | Example          | Purpose                             |
| --------------- | ---------------- | ----------------------------------- |
| Name            | `rad-1`          | RADIUS object name                  |
| Server          | `192.168.20.200` | RADIUS/NPS server                   |
| Authentication  | `MSCHAPv2`       | Authentication protocol             |
| NAS IP          | `192.168.20.254` | IP presented as NAS/client identity |
| Auth Port       | `1812`           | Authentication                      |
| Accounting Port | `1813`           | Accounting                          |

---

# 3. RADIUS Firewall Group

Create a local FortiGate group:

```text
User & Authentication
└── User Groups
    └── Create New
```

Example:

```text
Group Name: rad-g-1
Remote Server: rad-1
```

Depending on the authentication design, the group can reference the RADIUS server and/or remote group information returned by RADIUS.

### Policy Integration

The RADIUS group must be referenced by the firewall policy when user-based authentication is required.

```text
Policy & Objects
└── Firewall Policy
    └── Source User / User Group
        └── rad-g-1
```

> ⚠️ **Key Point:** Unlike explicit proxy authentication, a normal RADIUS-based identity policy does not use a separate "authentication schema" object. The firewall policy itself becomes part of the authentication flow when identity-based matching is configured.

---

# 4. Global RADIUS Authentication Port

CLI:

```bash
config system global
    set radius-port 1812
end
```

Use this when changing the default RADIUS authentication port.

---

# 5. RADIUS Username Encoding & Case Sensitivity

```bash
config user radius
    edit "rad-1"
        set password-encoding auto
        set username-case-sensitive enable
    next
end
```

### Password Encoding

```text
auto
```

Supports modern character encoding such as UTF-8 where applicable.

Legacy encoding options may exist for compatibility with older environments.

### Username Case Sensitivity

```text
username-case-sensitive enable
```

When enabled:

```text
User01
user01
USER01
```

may be treated as different usernames depending on the RADIUS server and authentication backend.

> 💡 **Best Practice:** Keep username handling consistent between FortiGate, RADIUS/NPS, Active Directory, and downstream authentication systems.

---

# 6. RADIUS VSA vs AVP

One of the most important concepts when integrating FortiGate with NPS/RADIUS is understanding **AVP** and **VSA**.

## AVP — Attribute-Value Pair

An AVP represents a standard RADIUS attribute:

```text
Attribute = Value
```

Examples:

| Attribute          | Code | Purpose                         |
| ------------------ | ---: | ------------------------------- |
| User-Name          |  `1` | Username                        |
| NAS-IP-Address     |  `4` | NAS IP                          |
| Framed-IP-Address  |  `8` | Assigned client IP              |
| Class              | `25` | Session/accounting correlation  |
| Vendor-Specific    | `26` | Container for vendor attributes |
| NAS-Identifier     | `32` | NAS identifier                  |
| Acct-Input-Octets  | `42` | Received octets                 |
| Acct-Output-Octets | `43` | Transmitted octets              |
| Acct-Session-Id    | `44` | Session identifier              |
| Event-Timestamp    | `55` | Event timestamp                 |

---

# 7. VSA — Vendor-Specific Attributes

**VSA = Vendor-Specific Attribute**

VSA extends standard RADIUS functionality with vendor-specific information.

For Fortinet:

```text
Vendor ID: 12356
Vendor: Fortinet
```

Conceptually:

```text
RADIUS Attribute 26
        │
        └── Fortinet Vendor-Specific Attributes
                  │
                  ├── Group Name
                  ├── Client IP
                  ├── VDOM
                  ├── IPv6 Address
                  ├── Interface
                  └── Access Profile
```

### Example Fortinet VSA Definitions

```text
Vendor: Fortinet
Vendor-ID: 12356
```

| VSA | Attribute                    | Type       | Purpose                       |
| --- | ---------------------------- | ---------- | ----------------------------- |
| `1` | fortinet-group-name          | String     | Remote group mapping          |
| `2` | fortinet-client-ip-address   | IP Address | Client IP                     |
| `3` | fortinet-vdom-name           | String     | VDOM information              |
| `4` | fortinet-client-ipv6-address | Octets     | IPv6 client information       |
| `5` | fortinet-interface-name      | String     | Interface information         |
| `6` | fortinet-access-profile      | String     | Administrative access profile |

---

# 8. VSA Group Mapping with NPS

A common use case is dynamically returning a group from the RADIUS server.

Example:

```text
NPS
│
├── User = alice
│
└── RADIUS Attribute
      │
      └── Fortinet Group = IT
```

FortiGate can then use the returned attribute for group matching.

Conceptually:

```text
NPS Policy
   │
   │ Return VSA
   ▼
FortiGate
   │
   │ Match returned group
   ▼
FortiGate User Group
   │
   ▼
Firewall Policy
```

### Example

NPS returns:

```text
Fortinet-Group-Name = IT
```

FortiGate group/policy can then match:

```text
IT
```

This creates a dynamic relationship between:

```text
AD Group
     ↓
NPS Policy
     ↓
RADIUS Attribute / VSA
     ↓
FortiGate Remote Group
     ↓
Firewall Policy
```

> 💡 **NSE Tip:** When authentication succeeds but the expected firewall policy does not match, inspect the returned RADIUS attributes before troubleshooting the firewall policy itself.

---

# 9. RADIUS Single Sign-On — RSSO

**RSSO = RADIUS Single Sign-On**

RSSO allows FortiGate to learn authenticated user/session information from RADIUS accounting or an RSSO mechanism and use that identity in firewall policies.

### GUI

```text
Security Fabric
└── External Connectors
    └── RADIUS Single Sign-On
```

Example:

```text
Name: rsso-agent
Enable required RSSO features
```

Then create a user group:

```text
User & Authentication
└── User Groups
    └── rsso-f
```

Remote server:

```text
rsso-agent
```

---

# 10. RSSO Policy Flow

```text
User
  │
  ▼
Network Access Device
  │
  │ RADIUS Authentication
  ▼
RADIUS Server
  │
  │ Accounting / Session Information
  ▼
FortiGate RSSO
  │
  │ User ↔ IP association
  ▼
FortiGate User Group
  │
  ▼
Firewall Policy
```

### Firewall Policy

Instead of matching only an IP address:

```text
Source:
    rsso-f
```

FortiGate can enforce identity-based access.

---

# 11. RSSO vs FSSO Polling vs FSSO Agent

| Feature                      | RADIUS RSSO              | FSSO Polling           | FSSO Collector |
| ---------------------------- | ------------------------ | ---------------------- | -------------- |
| Main source                  | RADIUS                   | AD                     | AD/DC Agent    |
| Authentication awareness     | RADIUS-based             | Polling                | Logon events   |
| DC Agent required            | No                       | No                     | Usually yes    |
| Automatic AD logon awareness | Limited by RADIUS design | No                     | Yes            |
| Best use case                | RADIUS environments      | Simple AD environments | Enterprise AD  |

### Design Rule

```text
Single / simple AD repository
        ↓
FSSO Polling can be suitable

Large AD environment / multiple DCs
        ↓
FSSO Collector / DC Agent

RADIUS-based authentication infrastructure
        ↓
RSSO
```

---

# 12. RADIUS Accounting

Accounting is important for:

* Session tracking
* User login/logout
* Dynamic shaping
* Session correlation
* Wireless environments
* RSSO-related identity information

Example:

```text
Authentication
UDP/1812
     ↓
Access-Accept

Accounting
UDP/1813
     ↓
Start / Update / Stop
```

---

# 13. Dynamic Traffic Shaping with RADIUS

FortiGate can use RADIUS attributes to dynamically influence traffic shaping.

### Step 1 — Configure RADIUS

```bash
config user radius
    edit "rad-1"
        set radius-coa enable
        set radius-all-server enable
    next
end
```

> ⚠️ Exact command availability and behavior can vary by FortiOS release. Verify against the target FortiOS version before deployment.

---

# 14. RADIUS Accounting Server

Example:

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

### Important

```text
1812 → Authentication
1813 → Accounting
```

---

# 15. Group Override vs Filter-Id

A RADIUS server can return attributes that FortiGate uses for group or policy decisions.

Example concept:

```text
RADIUS
   │
   └── Filter-Id = IT
              │
              ▼
        FortiGate Group
```

`Filter-Id` can be useful for identifying or mapping users/groups.

Other RADIUS attributes can be used for dynamic behavior such as traffic shaping depending on the FortiOS feature and integration design.

---

# 16. Dynamic Shaping in Firewall Policy

Example:

```bash
config firewall policy
    edit 1
        set dynamic-shaping enable
    next
end
```

Concept:

```text
RADIUS User
     │
     ├── Identity
     ├── Group
     └── Dynamic Attributes
             │
             ▼
       FortiGate Policy
             │
             ▼
      Dynamic Shaper
             │
             ▼
        User Traffic
```

### Why Dynamic Shaping?

It allows bandwidth behavior to follow user/session identity instead of relying only on static IP-based shaping.

Example:

```text
User A → 20 Mbps
User B → 10 Mbps
User C → 5 Mbps
```

without requiring each user to have a dedicated static IP-based policy.

---

# 17. Wireless RADIUS Attributes

In wireless environments, additional RADIUS attributes can carry information related to:

* SSID
* Wireless controller/device
* Access point
* Wireless termination point
* Association time

Example attributes from the Fortinet-specific wireless context:

| Code | Concept                                |
| ---: | -------------------------------------- |
|  `7` | Fortinet SSID                          |
| `23` | Wireless device/controller information |
| `24` | Wireless termination point             |
| `25` | Association time                       |

These attributes can become particularly important when troubleshooting:

```text
Roaming
Wireless identity
Dynamic shaping
Session tracking
AP/controller correlation
```

---

# 18. Termination Action

Termination behavior can determine what FortiGate does when a user's RADIUS/session state changes.

Conceptual values:

| Value | Meaning                                         |
| ----: | ----------------------------------------------- |
|   `0` | Default behavior / normal idle-timeout handling |
|   `1` | RADIUS request / session verification behavior  |
|   `2` | Resource/session management and cleanup         |

### Practical Design

For wireless environments:

```text
Termination Action = 1
```

can be useful when active authentication/session state must be synchronized more aggressively.

For wired environments:

```text
Termination Action = 0
```

may be more appropriate depending on the deployment.

> ⚠️ Treat these values as deployment-specific. Validate the exact FortiOS and RADIUS/NPS behavior before using them in production.

---

# 19. RADIUS CoA / Dynamic Authorization

**CoA = Change of Authorization**

CoA allows the RADIUS infrastructure to request a change to an existing authenticated session without forcing the user to completely reconnect.

Conceptually:

```text
RADIUS/NPS
    │
    │ CoA
    ▼
FortiGate
    │
    ├── Change authorization
    ├── Update session
    └── Apply new attributes
```

Useful for:

* Dynamic access control
* User policy changes
* Bandwidth changes
* Wireless environments
* Post-authentication authorization updates

---

# 20. RSA SecurID / ACE

RSA SecurID uses OTP-based authentication.

Concept:

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

Example diagnostic test:

```bash
diagnose test authserver radius <server> <username> <password>
```

For an OTP deployment, the authentication credential may include the OTP/token according to the RSA/RADIUS integration design.

> 🔐 **Security Rule:** Never store or publish real RADIUS shared secrets, user passwords, OTP seeds, or production credentials in documentation or GitHub.

---

# 21. Troubleshooting — Authentication

### Test RADIUS Authentication

```bash
diagnose test authserver radius <server> <username> <password>
```

Use this to answer:

```text
Can FortiGate reach the RADIUS server?
        ↓
Does RADIUS accept the credentials?
        ↓
Is the authentication method correct?
```

---

# 22. Troubleshooting — User Database

Useful commands:

```bash
diagnose firewall auth list
```

This helps inspect authenticated users/sessions known to the firewall.

For user/device database troubleshooting:

```bash
diagnose user-device-store user-count list 1
```

Example query:

```bash
diagnose user-device-store user-count query \
"CN=callcenter,OU=information-tech,OU=publish-users,DC=test,DC=com"
```

Historical statistics may also be queried depending on the FortiOS feature/version:

```bash
diagnose user-device-store user-stats query <date> <value>
```

---

# 23. Troubleshooting — Dynamic Shaper

### Dynamic Shaper Statistics

```bash
diagnose firewall shaper dynamic-shaper stats
```

### Dynamic Shaper List

```bash
diagnose firewall shaper dynamic-shaper list
```

### Specific Client

```bash
diagnose firewall shaper dynamic-shaper list ip 192.168.20.20
```

---

# 24. Troubleshooting — Sessions

Inspect active sessions:

```bash
diagnose sys session list
```

Inspect firewall policy processing:

```bash
diagnose firewall iprope list 100004
```

These commands help correlate:

```text
User
 ↓
Authentication
 ↓
Policy
 ↓
Session
 ↓
Shaper
```

---

# 25. Wireless Session Troubleshooting

For wireless controller/session information:

```bash
diagnose wireless-controller wlac-d sta online
```

Useful when troubleshooting:

* Wireless clients
* AP association
* Roaming
* Session state
* Dynamic shaping
* Authentication state

---

# 26. RADIUS Troubleshooting Matrix

| Symptom                                      | Check                                               |
| -------------------------------------------- | --------------------------------------------------- |
| Authentication fails                         | RADIUS IP / secret / port / authentication method   |
| Timeout                                      | Routing / firewall / UDP 1812                       |
| Accounting missing                           | UDP 1813 / accounting configuration                 |
| User authenticates but policy does not match | Group mapping / returned attributes                 |
| RSSO user not detected                       | Accounting / RSSO configuration                     |
| Dynamic shaping not applied                  | RADIUS attributes / accounting / policy             |
| Wrong group                                  | NPS policy / VSA / Filter-Id                        |
| Wireless session remains stale               | Accounting / termination behavior                   |
| CoA not working                              | CoA configuration / RADIUS reachability             |
| Username mismatch                            | Case sensitivity / AD username format               |
| MSCHAPv2 failure                             | Authentication method / NPS policy / AD integration |

---

# 27. End-to-End NPS → FortiGate Design

```text
                    Active Directory
                           │
                           ▼
                    ┌─────────────┐
                    │    NPS      │
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
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
       Groups            RSSO         Dynamic Shaper
          │                │                │
          └────────────────┼────────────────┘
                           ▼
                    Firewall Policy
                           │
                           ▼
                         User
```

---

# 28. NSE Exam Memory Map

```text
RADIUS
│
├── 1812
│    └── Authentication
│
├── 1813
│    └── Accounting
│
├── AVP
│    └── Standard RADIUS attributes
│
├── VSA
│    └── Vendor-specific attributes
│
├── Fortinet Vendor ID
│    └── 12356
│
├── RSSO
│    └── RADIUS-based user/session awareness
│
├── CoA
│    └── Dynamic authorization changes
│
└── Dynamic Shaping
     └── Identity/session-based bandwidth control
```

---

# 29. High-Value NSE Gotchas ⚠️

### 1. Authentication ≠ Authorization

```text
Authentication
    ↓
Who are you?

Authorization
    ↓
What are you allowed to access?
```

A successful RADIUS login does **not** automatically mean the user will match the intended FortiGate firewall policy.

---

### 2. 1812 ≠ 1813

```text
1812 → Authentication
1813 → Accounting
```

If authentication works but accounting-dependent features fail, check **UDP/1813**.

---

### 3. AVP ≠ VSA

```text
AVP → Standard RADIUS attribute
VSA → Vendor-specific attribute
```

Fortinet-specific information is commonly carried through the RADIUS Vendor-Specific Attribute mechanism.

---

### 4. RADIUS Group Mapping Matters

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

A broken link anywhere in this chain can result in:

```text
Authentication = SUCCESS
Policy Match    = FAILURE
```

---

### 5. RSSO Requires Session Awareness

RSSO is not simply:

```text
Username + Password
```

The important concept is:

```text
Authentication
      +
Session / Accounting Information
      ↓
FortiGate User ↔ IP Awareness
```

---

### 6. Dynamic Shaping Depends on Correct Identity/Attributes

If the user is authenticated but dynamic bandwidth is not applied, inspect:

```text
RADIUS attributes
        ↓
Accounting
        ↓
FortiGate user/session
        ↓
Dynamic shaper
        ↓
Firewall policy
```

---

# 30. Production Checklist

## RADIUS Server

* [ ] RADIUS server IP is correct
* [ ] Shared secret matches
* [ ] UDP/1812 reachable
* [ ] UDP/1813 reachable if accounting is required
* [ ] NAS IP is correctly defined
* [ ] Authentication method matches NPS
* [ ] Username format is consistent
* [ ] Username case sensitivity is understood
* [ ] NPS policy is correctly configured

## FortiGate

* [ ] RADIUS object created
* [ ] RADIUS group created
* [ ] Firewall policy references correct group
* [ ] Required VSA/AVP mappings configured
* [ ] Accounting server configured when required
* [ ] RSSO configured when required
* [ ] CoA configured when required
* [ ] Dynamic shaping enabled when required
* [ ] Logging enabled for troubleshooting

## Security

* [ ] Never publish RADIUS shared secrets
* [ ] Never publish real user passwords
* [ ] Use secure management access
* [ ] Restrict RADIUS server communication to trusted NAS devices
* [ ] Prefer protected RADIUS deployments where supported
* [ ] Monitor authentication failures
* [ ] Monitor unexpected RADIUS accounting behavior

---

# 31. One-Page Quick Reference

```text
╔════════════════════════════════════════════════════╗
║                FORTIGATE RADIUS                  ║
╠════════════════════════════════════════════════════╣
║ Authentication          UDP 1812                  ║
║ Accounting              UDP 1813                  ║
║ Legacy Auth             UDP 1645                  ║
║ Legacy Accounting       UDP 1646                  ║
║                                                    ║
║ AVP                     Standard Attribute         ║
║ VSA                     Vendor-Specific Attribute  ║
║ Fortinet Vendor ID      12356                     ║
║                                                    ║
║ RSSO                    RADIUS Single Sign-On      ║
║ CoA                     Change of Authorization    ║
║                                                    ║
║ Dynamic Shaping         Identity/session based     ║
║                                                    ║
║ Main troubleshooting:                             ║
║ diagnose test authserver radius                  ║
║ diagnose firewall auth list                      ║
║ diagnose sys session list                        ║
║ diagnose firewall shaper dynamic-shaper stats   ║
║ diagnose firewall shaper dynamic-shaper list     ║
╚════════════════════════════════════════════════════╝
```

---

## 🎯 SheynShield Takeaway

> **RADIUS on FortiGate is not just username/password authentication.**

The complete architecture can combine:

```text
RADIUS Authentication
        +
RADIUS Accounting
        +
AVP / VSA
        +
Group Mapping
        +
RSSO
        +
CoA
        +
Dynamic Shaping
        ↓
Identity-Aware Network Security
```

For NSE-level troubleshooting, always follow the chain:

```text
User
 ↓
RADIUS Authentication
 ↓
NPS Policy
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
Dynamic Authorization / Shaping
```

**If authentication succeeds but access behavior is wrong, do not stop at RADIUS authentication. Trace the entire identity-to-policy chain.**


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

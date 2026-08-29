# 🔐 TACACS+ & 802.1X — FortiGate  

> **SheynShield | Security & Design Knowledge Base**
> **Topic:** TACACS+ Remote Administration & 802.1X Network Access Control
> **Platform:** FortiGate / FortiOS
> **Level:** NSE4 → NSE7
> **Keywords:** TACACS+, AAA, Remote Authentication, TACACS Authorization, 802.1X, EAP, PEAP, EAP-TLS, RADIUS, Supplicant

---

# 1. TACACS+ — Core Concept

**TACACS+** is primarily used for centralized **AAA (Authentication, Authorization, and Accounting)**, especially for network-device administration.

```text
Administrator
      │
      ▼
  FortiGate
      │
      │ TACACS+
      ▼
 TACACS+ Server
      │
      ▼
 Identity / AAA Backend
```

### TACACS+ vs RADIUS

| Feature              | TACACS+                  | RADIUS                           |
| -------------------- | ------------------------ | -------------------------------- |
| Primary use          | Device administration    | Network/user access              |
| Transport            | TCP                      | UDP                              |
| Authentication       | Supported                | Supported                        |
| Authorization        | Strongly separated       | Attribute-based                  |
| Accounting           | Supported                | Supported                        |
| Common FortiGate use | Admin login              | VPN / Wi-Fi / 802.1X / user auth |
| Password encryption  | More complete separation | Password field protection        |
| Typical port         | TCP `49`                 | UDP `1812/1813`                  |

> 🎯 **NSE Memory:**
> **TACACS+ → Device Administration / AAA**
> **RADIUS → Network Access / Authentication / Accounting**

---

# 2. Legacy .NET Framework Requirement

Some TACACS+ server implementations or management software may require older Windows/.NET components.

For example, enabling **.NET Framework 3.5** from Windows installation media:

```powershell
DISM /Online /Enable-Feature `
 /FeatureName:NetFx3 `
 /All `
 /LimitAccess `
 /Source:D:\sources\sxs
```

### Parameters

| Parameter             | Purpose                                 |
| --------------------- | --------------------------------------- |
| `/Online`             | Modify the running Windows installation |
| `/Enable-Feature`     | Enable Windows feature                  |
| `/FeatureName:NetFx3` | .NET Framework 3.x                      |
| `/All`                | Enable required parent features         |
| `/LimitAccess`        | Do not contact Windows Update           |
| `/Source`             | Installation source                     |

> ⚠️ **Production Note:** This is a Windows/TACACS+ server prerequisite, not a FortiGate requirement. Use only when the selected TACACS+ implementation actually requires it.

---

# 3. Configure TACACS+ Server on FortiGate

### CLI

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

### Key Parameters

| Parameter           | Example           | Purpose                                   |
| ------------------- | ----------------- | ----------------------------------------- |
| Name                | `tac-1`           | FortiGate TACACS+ object                  |
| Server              | `192.168.20.200`  | TACACS+ server                            |
| Key                 | `<TACACS_SECRET>` | Shared secret                             |
| Authorization       | `enable`          | Enable TACACS+ authorization              |
| Authentication Type | `auto`            | Automatic authentication method selection |

> 🔐 **Security:** Never commit the real TACACS+ shared key to GitHub. Use placeholders such as `<TACACS_SECRET>`.

---

# 4. TACACS+ User Group

After creating the remote TACACS+ server, create a FortiGate user group that references the remote server/group.

### GUI

```text
User & Authentication
└── User Groups
    └── Create New
```

Example:

```text
Group Name:
    tac-group-1

Remote Server:
    tac-1

Remote Groups:
    All required TACACS+ groups
```

Conceptual authentication chain:

```text
TACACS+ Server
      │
      ├── Network-Admins
      ├── Security-Admins
      └── Read-Only
             │
             ▼
       FortiGate Group
       tac-group-1
```

---

# 5. TACACS+ for FortiGate Administrators

TACACS+ becomes especially useful when FortiGate administrator accounts must be centrally controlled.

### GUI

```text
System
└── Administrators
    └── Create New
```

Example concept:

```text
Username:
    u1

User Type:
    Remote Authentication

Remote User Group:
    tac-group-1

Admin Profile:
    Super_Admin
```

### Authentication Flow

```text
Admin
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

---

# 6. TACACS+ Authorization

Authentication answers:

```text
"Who are you?"
```

Authorization answers:

```text
"What are you allowed to do?"
```

A common enterprise design is:

```text
TACACS+ Group
      │
      ▼
FortiGate Remote Group
      │
      ▼
FortiGate Administrator
      │
      ▼
Administrator Profile
```

Example:

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

> 💡 **Best Practice:** Avoid assigning `Super_Admin` to every remote administrator. Map TACACS+ groups to the minimum FortiGate administrator profile required.

---

# 7. TACACS+ Troubleshooting Checklist

```text
[ ] TACACS+ server reachable
[ ] TCP/49 permitted
[ ] Shared secret matches
[ ] Authentication method matches
[ ] Remote group exists
[ ] FortiGate user group references TACACS+
[ ] Administrator references correct remote group
[ ] TACACS+ authorization is enabled
[ ] TACACS+ server returns expected authorization
[ ] FortiGate admin profile is correct
```

### Troubleshooting Logic

```text
Login fails
   │
   ├── Server reachable?
   │
   ├── TCP/49 open?
   │
   ├── Shared secret correct?
   │
   ├── Authentication accepted?
   │
   ├── Group returned?
   │
   └── Authorization/profile mapped?
```

---

# 8. 802.1X — Core Concept

**802.1X** provides port-based network access control.

The three main components are:

```text
┌──────────────┐
│ Supplicant   │
│ Client       │
└──────┬───────┘
       │ EAP
       ▼
┌──────────────┐
│ Authenticator│
│ FortiGate    │
└──────┬───────┘
       │ RADIUS
       ▼
┌──────────────┐
│ Auth Server  │
│ NPS/RADIUS   │
└──────────────┘
```

### Roles

| Component             | Role                                               |
| --------------------- | -------------------------------------------------- |
| Supplicant            | Client requesting network access                   |
| Authenticator         | Controls access to the network                     |
| Authentication Server | Validates the identity                             |
| RADIUS                | AAA communication between authenticator and server |
| EAP                   | Authentication framework                           |

---

# 9. 802.1X Authentication Methods

Common EAP methods include:

```text
PEAP
EAP-TLS
```

### PEAP

```text
Client
  │
  │ Outer TLS tunnel
  ▼
RADIUS Server
  │
  └── Inner authentication
```

PEAP commonly protects an inner username/password-based authentication exchange inside a TLS tunnel.

### EAP-TLS

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

EAP-TLS requires certificates and provides certificate-based authentication.

---

# 10. Certificates

Certificate-based EAP methods require proper PKI planning.

Typical components:

```text
CA
 │
 ├── RADIUS/NPS Server Certificate
 │
 └── Client Certificate
```

### Certificate Checklist

```text
[ ] Certificate issued by trusted CA
[ ] Certificate is valid
[ ] Certificate is not expired
[ ] SAN is correctly configured
[ ] Client trusts the CA
[ ] RADIUS server trusts required CA
[ ] Certificate chain is complete
[ ] EAP method matches certificate deployment
```

> 🎯 **NSE Tip:** EAP-TLS authentication problems are often certificate-chain, trust, SAN, or EKU problems rather than simple RADIUS connectivity problems.

---

# 11. FortiGate as 802.1X Supplicant

A FortiGate interface can be configured to participate in an 802.1X/EAP authentication process.

### RADIUS Server

First define the RADIUS authentication server:

```text
User & Authentication
└── RADIUS Servers
```

Example:

```text
Name:
    rad-1

NAS IP:
    192.168.20.254

Authentication:
    MSCHAPv2

Server:
    192.168.20.200

Secret:
    <RADIUS_SECRET>
```

---

# 12. Interface 802.1X Configuration

The relevant interface must be configured for EAP/802.1X behavior.

Example:

```bash
config system interface
    edit "port3"
        set allowaccess <required-management-services>
        set stpforward enable
        set eap-supplicant enable
        set eap-method peap
        set eap-identity <EAP_IDENTITY>
        set eap-password <EAP_PASSWORD>
        set eap-ca-cert <CA_CERTIFICATE>
    next
end
```

### PEAP Example

```text
eap-supplicant:
    enable

eap-method:
    peap

eap-identity:
    <AD_USERNAME>

eap-password:
    <PASSWORD>

eap-ca-cert:
    <CA_CERTIFICATE>
```

> 🔐 **Security:** Never publish the real EAP username/password or private certificates in a public repository.

---

# 13. EAP Identity

The EAP identity identifies the supplicant to the authentication infrastructure.

Example:

```text
EAP Identity:
    test
```

In an AD-integrated environment, the identity can correspond to a dedicated authentication account.

### Recommended Design

```text
FortiGate
   │
   └── Dedicated 802.1X Identity
            │
            ▼
       NPS / RADIUS
            │
            ▼
       Authentication
```

> 💡 Avoid using a personal administrator account as the long-term EAP supplicant identity.

---

# 14. PEAP vs EAP-TLS

| Feature                                 | PEAP      | EAP-TLS   |
| --------------------------------------- | --------- | --------- |
| TLS tunnel                              | ✅         | ✅         |
| Client certificate                      | Usually ❌ | ✅         |
| Server certificate                      | ✅         | ✅         |
| Username/password                       | Common    | ❌         |
| Certificate-based client authentication | ❌         | ✅         |
| Deployment complexity                   | Lower     | Higher    |
| Enterprise PKI dependency               | Lower     | High      |
| Security strength                       | High      | Very High |

### Quick Decision

```text
Need username/password inside TLS?
        ↓
      PEAP

Need certificate-based authentication?
        ↓
     EAP-TLS
```

---

# 15. Interface Access & 802.1X

After enabling EAP behavior on the interface, FortiGate can expose an authentication/security mode associated with the interface.

Conceptually:

```text
Interface
   │
   ├── Normal access
   │
   ├── Captive Portal
   │
   └── 802.1X / EAP authentication
```

The exact GUI options depend on the FortiOS release and interface type.

---

# 16. EAP Supplicant Diagnostics

Useful diagnostic command:

```bash
diagnose test app eap_supp 2
```

The diagnostic interface can be used to test or inspect EAP supplicant behavior.

In the relevant diagnostic context, a special restart action may be available:

```text
99 → Restart EAP supplicant daemon
```

> ⚠️ Diagnostic command syntax and available test numbers can vary between FortiOS releases. Always verify against the target release.

---

# 17. 802.1X Troubleshooting Flow

```text
                802.1X Failure
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
    Link / Interface          Authentication
          │                       │
          │                       ├── RADIUS reachable?
          │                       ├── Secret correct?
          │                       ├── EAP method correct?
          │                       ├── Certificate valid?
          │                       └── Credentials valid?
          │
          ▼
     EAP Supplicant
          │
          ▼
       RADIUS/NPS
          │
          ▼
      Access-Accept
```

---

# 18. PEAP Troubleshooting

If PEAP fails:

```text
[ ] RADIUS server reachable
[ ] UDP/1812 reachable
[ ] Shared secret correct
[ ] MSCHAPv2 supported
[ ] NPS policy matches
[ ] Username correct
[ ] Password correct
[ ] Server certificate valid
[ ] Client/FortiGate trusts CA
[ ] EAP method matches on both sides
```

---

# 19. EAP-TLS Troubleshooting

For EAP-TLS:

```text
[ ] Client certificate exists
[ ] Certificate is valid
[ ] Certificate is not expired
[ ] Private key is available
[ ] CA is trusted
[ ] Server certificate is trusted
[ ] SAN is correct
[ ] EKU is appropriate
[ ] Certificate chain is complete
[ ] NPS policy allows EAP-TLS
[ ] FortiGate EAP method = TLS
```

---

# 20. TACACS+ vs RADIUS vs 802.1X

| Technology | Main Purpose               | Typical FortiGate Role            |
| ---------- | -------------------------- | --------------------------------- |
| TACACS+    | Device AAA                 | Administrator authentication      |
| RADIUS     | Centralized AAA            | User/network authentication       |
| 802.1X     | Port-based access control  | EAP authentication                |
| PEAP       | EAP authentication method  | Username/password over TLS        |
| EAP-TLS    | Certificate-based EAP      | Strong certificate authentication |
| RSSO       | Identity/session awareness | User-aware firewall policies      |

### Remember

```text
TACACS+
   ↓
"Who can administer my device?"

RADIUS
   ↓
"Who is this user and what attributes apply?"

802.1X
   ↓
"Should this endpoint receive network access?"

PEAP / EAP-TLS
   ↓
"How should the endpoint authenticate?"
```

---

# 21. Enterprise Authentication Architecture

A common enterprise architecture can look like:

```text
                         Active Directory
                                │
                                ▼
                        ┌──────────────┐
                        │ NPS / RADIUS │
                        └──────┬───────┘
                               │
                 ┌─────────────┼─────────────┐
                 │             │             │
                 ▼             ▼             ▼
             RADIUS         802.1X       Wireless
                 │             │             │
                 │             │             │
                 ▼             ▼             ▼
             FortiGate     FortiGate      AP/WLC
                 │
                 │
                 ▼
              Users
```

For administrator AAA:

```text
Administrator
     │
     ▼
FortiGate
     │
     │ TCP/49
     ▼
TACACS+
     │
     ▼
AD / Identity Store
```

---

# 22. High-Value NSE Gotchas ⚠️

### TACACS+ is not RADIUS

```text
TACACS+ → TCP/49
RADIUS  → UDP/1812 / UDP/1813
```

Do not troubleshoot them as if they use the same transport.

---

### 802.1X is not an authentication protocol by itself

802.1X provides the framework for port-based access control.

The actual authentication method can be:

```text
PEAP
EAP-TLS
EAP-...
```

---

### PEAP ≠ EAP-TLS

```text
PEAP
    └── TLS tunnel + inner authentication

EAP-TLS
    └── Certificate-based client authentication
```

---

### Authentication and Authorization are different

```text
Authentication
       ↓
Identity verified

Authorization
       ↓
Permissions determined
```

A TACACS+ user can authenticate successfully but still receive the wrong FortiGate administrator permissions if group/profile mapping is incorrect.

---

### Don't confuse FortiGate's role

In a particular 802.1X deployment, determine whether FortiGate is acting as:

```text
Supplicant
      or
Authenticator
```

The configuration and troubleshooting path are different.

---

# 23. Production Security Checklist

## TACACS+

* [ ] Use a strong shared secret
* [ ] Restrict TACACS+ server access to trusted FortiGate management IPs
* [ ] Use remote groups for centralized authorization
* [ ] Map groups to least-privilege admin profiles
* [ ] Avoid unnecessary `Super_Admin`
* [ ] Configure redundant TACACS+ servers where appropriate
* [ ] Monitor administrator authentication
* [ ] Never publish shared secrets

## 802.1X

* [ ] Define RADIUS/NPS correctly
* [ ] Validate EAP method
* [ ] Deploy required certificates
* [ ] Validate CA trust
* [ ] Verify SAN/EKU
* [ ] Use dedicated authentication identities
* [ ] Protect EAP credentials
* [ ] Test failover
* [ ] Monitor authentication failures

---

# 24. Lab Validation Checklist

### TACACS+

```text
[ ] TACACS+ server = 192.168.20.200
[ ] TCP/49 reachable
[ ] Shared secret configured
[ ] TACACS+ authentication succeeds
[ ] Remote group returned
[ ] FortiGate group matches
[ ] Remote administrator login succeeds
[ ] Correct admin profile assigned
```

### 802.1X

```text
[ ] RADIUS server configured
[ ] NAS IP configured
[ ] UDP/1812 reachable
[ ] Shared secret matches
[ ] PEAP tested
[ ] EAP-TLS tested if required
[ ] CA certificate installed
[ ] EAP identity configured
[ ] EAP credentials validated
[ ] Interface EAP supplicant enabled
[ ] EAP diagnostics tested
```

---

# 25. Quick Command Reference

### TACACS+

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

### EAP Supplicant

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

### EAP Diagnostic

```bash
diagnose test app eap_supp 2
```

---

# 26. One-Page Memory Map

```text
                 AUTHENTICATION TECHNOLOGIES
                              │
          ┌───────────────────┼───────────────────┐
          │                   │                   │
          ▼                   ▼                   ▼
      TACACS+              RADIUS               802.1X
          │                   │                   │
          │                   │                   └── EAP
          │                   │                         │
          │                   │                  ┌──────┴──────┐
          │                   │                  ▼             ▼
          │                   │                PEAP         EAP-TLS
          │                   │                  │             │
          │                   │                  │        Certificates
          │                   │                  │
          ▼                   ▼                  ▼
     Device AAA          User AAA          Port Access
          │                   │                  │
          ▼                   ▼                  ▼
    Admin Login        Authentication       Network Access
    Authorization      Accounting
```

---

# 🎯 SheynShield Expert Takeaway

The easiest way to remember this entire section is:

```text
TACACS+
   │
   └── FortiGate Administration
          ├── Authentication
          └── Authorization
                  │
                  ▼
             Admin Profile


RADIUS
   │
   ├── Authentication
   ├── Accounting
   ├── VSA / AVP
   ├── RSSO
   └── Dynamic Authorization


802.1X
   │
   └── Port-Based Network Access
            │
            └── EAP
                 ├── PEAP
                 │    └── Username/Password inside TLS
                 │
                 └── EAP-TLS
                      └── Certificate-Based Authentication
```

> **The NSE mental model:**
> **TACACS+ controls administrators. RADIUS authenticates users and carries AAA attributes. 802.1X controls network admission. EAP defines how the endpoint authenticates.**


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

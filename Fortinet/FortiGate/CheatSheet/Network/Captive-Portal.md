# FortiGate Captive Portal  

> **FortiGate Captive Portal** forces users to authenticate before accessing permitted network resources.
> It is commonly used for **guest Wi-Fi, hotspot access, guest networks, and user-based network access control**.

---

## 1. Captive Portal — Core Concept

```text
Client
   |
   | HTTP/HTTPS Request
   v
FortiGate
   |
   | Authentication Required
   v
Captive Portal
   |
   | Username / Password
   v
Authentication Server
   |
   | LDAP / RADIUS / Local User
   v
Authenticated
   |
   v
Internet / Network Resources
```

### Key Components

| Component              | Purpose                                        |
| ---------------------- | ---------------------------------------------- |
| Captive Portal         | Authentication page presented to users         |
| Local User             | Local FortiGate authentication                 |
| LDAP                   | Directory-based authentication                 |
| RADIUS                 | Remote authentication                          |
| User Group             | Defines which users can authenticate           |
| Firewall Policy        | Enforces authenticated access                  |
| Authentication Timeout | Controls how long authentication remains valid |

---

# 2. Captive Portal Authentication Modes

FortiGate can use different authentication sources:

```text
Captive Portal
      |
      +---- Local Users
      |
      +---- LDAP
      |
      +---- RADIUS
      |
      +---- User Groups
```

Typical deployment:

```text
Guest Client
     |
     v
Guest VLAN / Wi-Fi
     |
     v
FortiGate
     |
 Captive Portal
     |
     +---- Local User
     |
     +---- LDAP
     |
     +---- RADIUS
     |
     v
Internet
```

---

# 3. Captive Portal User Types

FortiGate user groups can be used to control access.

Typical logical structure:

```text
Users & Devices
      |
      +---- Users
      |
      +---- User Groups
                |
                +---- Firewall Authentication
                |
                +---- LDAP Users
                |
                +---- RADIUS Users
```

Example concept:

```text
guest-users
     |
     +---- guest01
     +---- guest02
     +---- guest03
```

---

# 4. Guest Users

FortiGate Guest Management can be used to create temporary guest accounts.

Typical workflow:

```text
Administrator
     |
     v
Guest Manager
     |
     +---- Create User
     +---- Set Username
     +---- Set Password
     +---- Set Expiration
     |
     v
Guest User
     |
     v
Captive Portal
```

Guest accounts are useful when users require **temporary or limited-time access**.

---

# 5. Captive Portal Firewall Policy

Authentication is normally enforced through a firewall policy.

Example:

```bash
config firewall policy
    edit 10
        set name "Guest-Captive-Portal"
        set srcintf "guest"
        set dstintf "wan"
        set srcaddr "all"
        set dstaddr "all"
        set schedule "always"
        set service "ALL"
        set action "accept"
        set nat enable
    next
end
```

Logical flow:

```text
Guest Interface
      |
      v
Firewall Policy
      |
      v
Authentication
      |
      +---- NOT AUTHENTICATED
      |          |
      |          v
      |    Captive Portal
      |
      +---- AUTHENTICATED
                 |
                 v
              Internet
```

> **Important:** Captive Portal authentication is not simply a GUI feature. The traffic must be processed by the appropriate firewall policy and authentication mechanism.

---

# 6. Authentication Timeout

Authentication sessions can be controlled using authentication timeout settings.

```bash
config user setting
    set auth-timeout-type idle-timeout
    set auth-timeout 5
end
```

`auth-timeout` is measured in **minutes**.

### Timeout Types

| Type              | Behavior                                                                   |
| ----------------- | -------------------------------------------------------------------------- |
| `idle-timeout`    | Session expires after the configured idle period                           |
| `hard-timeout`    | Authentication expires after a fixed period                                |
| `session-timeout` | New sessions are denied after timeout while existing sessions can continue |

### Example

```bash
set auth-timeout-type idle-timeout
set auth-timeout 5
```

Conceptually:

```text
User Login
    |
    v
Authenticated
    |
    | No traffic for 5 minutes
    v
Authentication Expires
```

For guest networks, **idle timeout** is often more practical than forcing every user to re-authenticate after a fixed period.

---

# 7. HTTP → HTTPS Authentication

FortiGate can redirect HTTP authentication requests toward HTTPS.

Relevant authentication settings include:

```text
HTTP
HTTPS
HTTP Redirect
```

Concept:

```text
Client
  |
  | HTTP
  v
FortiGate
  |
  | Redirect
  v
HTTPS Captive Portal
```

### Recommended

Use a valid certificate for HTTPS captive portal authentication.

```text
HTTP
 ↓
HTTPS Redirect
 ↓
Captive Portal
 ↓
Authentication
```

---

# 8. Captive Portal Authentication Methods

Common authentication backends:

### Local

```text
Client
  ↓
Captive Portal
  ↓
FortiGate Local User Database
```

### LDAP

```text
Client
  ↓
Captive Portal
  ↓
FortiGate
  ↓
LDAP
  ↓
Active Directory / LDAP Directory
```

### RADIUS

```text
Client
  ↓
Captive Portal
  ↓
FortiGate
  ↓
RADIUS
  ↓
NPS / RADIUS Server
```

---

# 9. Captive Portal + LDAP

Example LDAP server:

```bash
config user ldap
    edit "LDAP"
        set server 10.10.20.3
        set cnid "sAMAccountName"
        set dn "dc=example,dc=local"
        set type regular
        set username "administrator@example.local"
        set password <PASSWORD>
    next
end
```

Then create a firewall user group using the LDAP server.

Logical structure:

```text
LDAP Server
    |
    +---- Users
    |
    +---- Groups
            |
            v
       FortiGate User Group
            |
            v
       Firewall Policy
            |
            v
       Captive Portal
```

---

# 10. Captive Portal + RADIUS

Typical RADIUS flow:

```text
Client
   |
   v
Captive Portal
   |
   v
FortiGate
   |
   | RADIUS Authentication
   v
RADIUS Server
   |
   v
Access-Accept / Access-Reject
```

Common RADIUS ports:

```text
UDP/1812 = Authentication
UDP/1813 = Accounting
```

---

# 11. Captive Portal + Guest Wi-Fi

Typical enterprise guest architecture:

```text
                Internet
                   |
                   |
              FortiGate
                   |
             Guest VLAN
                   |
          +--------+--------+
          |                 |
       FortiAP           Guest Client
          |                 |
          +--------+--------+
                   |
             Captive Portal
                   |
             Authentication
```

The guest network should normally be isolated from internal networks.

Example:

```text
Guest VLAN
    |
    +---- Internet       ✅
    |
    +---- Internal LAN   ❌
    |
    +---- Servers        ❌
    |
    +---- Management     ❌
```

---

# 12. Guest Network Security Model

Recommended policy structure:

```text
Guest → Internet
       ACCEPT
       NAT ENABLED

Guest → Internal Networks
       DENY

Guest → Management
       DENY
```

Example:

```text
                 FortiGate
                    |
          +---------+---------+
          |                   |
       Internal             Guest
          |                   |
       Servers             Captive
       Users               Portal
                              |
                           Internet
```

---

# 13. Email Collection Captive Portal

FortiGate can also use an email-collection portal for guest access scenarios.

Example configuration concept:

```bash
config system global
    set switch-controller enable
end
```

Example captive portal VAP:

```bash
config wireless-controller.vap
    edit "freewifi"
        set security captive-portal
        set portal-type email-collect
    next
end
```

Firewall policy:

```bash
config firewall policy
    edit 1
        set email-collect enable
    next
end
```

Typical use cases:

```text
Retail
Shopping Centers
Hotels
Guest Wi-Fi
Marketing Wi-Fi
Public Wi-Fi
```

---

# 14. Captive Portal Authentication Monitoring

Monitor authenticated firewall users:

```bash
diagnose firewall auth list
```

This can help identify:

```text
Authenticated users
Authentication state
User sessions
Authentication-related information
```

For MAC-based authentication information:

```bash
diagnose firewall auth mac list
```

---

# 15. Authentication Troubleshooting

### Step 1 — Verify User

```text
User exists?
        |
        +---- Local
        +---- LDAP
        +---- RADIUS
```

### Step 2 — Verify Group

```text
User
 ↓
User Group
 ↓
Firewall Policy
```

### Step 3 — Verify Policy

Check:

```text
Source Interface
Source Address
Source User / Group
Destination Interface
Destination Address
Service
NAT
```

### Step 4 — Check Authentication

```bash
diagnose firewall auth list
```

### Step 5 — Test LDAP

```bash
diagnose test authserver ldap <server> <username> <password>
```

### Step 6 — Check RADIUS

Verify:

```text
UDP/1812
NAS IP
Shared Secret
Authentication Method
RADIUS Group
```

---

# 16. Common Captive Portal Problems

| Problem                                | Possible Cause                         |
| -------------------------------------- | -------------------------------------- |
| Portal does not appear                 | Policy / interface / redirect issue    |
| Login rejected                         | Wrong credentials                      |
| LDAP authentication fails              | LDAP configuration / DN / connectivity |
| RADIUS authentication fails            | NAS IP / secret / port                 |
| User authenticates but has no Internet | Routing / policy / NAT                 |
| User repeatedly gets login page        | Timeout / session handling             |
| HTTPS warning                          | Certificate problem                    |
| Guest can access internal network      | Missing isolation policy               |
| Portal works but applications fail     | DNS / policy / service restrictions    |

---

# 17. Captive Portal Security Checklist

```text
[ ] Dedicated Guest VLAN / Interface
[ ] Guest users isolated from internal networks
[ ] Captive Portal enabled
[ ] HTTPS enabled
[ ] Valid certificate configured
[ ] Authentication source configured
[ ] User group configured
[ ] Firewall policy configured
[ ] NAT enabled for Internet access
[ ] Internal network access denied
[ ] Authentication timeout configured
[ ] Guest account expiration configured where required
[ ] Authentication logs monitored
[ ] LDAP/RADIUS connectivity tested
```

---

# 18. Captive Portal — Quick Mental Model

```text
                    INTERNET
                       |
                       |
                    FortiGate
                       |
                Firewall Policy
                       |
                 Authentication
                       |
                +------+------+
                |             |
           NOT AUTHENTICATED  AUTHENTICATED
                |             |
                v             v
         Captive Portal     Internet
                |
                v
       Local / LDAP / RADIUS
                |
                v
        Authentication Result
                |
        +-------+-------+
        |               |
      ACCEPT           REJECT
        |               |
        v               v
    Internet          Denied
```

---

## Related Topics

```text
Network
├── Captive Portal
├── VLAN
├── Guest Network
├── 802.1X
├── RADIUS
├── LDAP
└── Network Access Control
```

> **Core idea:** Captive Portal controls **network access at the point of connection**, while LDAP/RADIUS provide the **identity source** and the firewall policy determines **what the authenticated user is allowed to access**.

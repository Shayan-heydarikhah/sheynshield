# 🔐 FortiGate Captive Portal Checklist

> **FortiOS | Captive Portal | Guest Access | LDAP | RADIUS | Authentication | Guest Wi-Fi | Network Access Control**
>
> **SheynShield — Engineering Secure Networks**

---

## 📋 Table of Contents

* [ ] [1. Captive Portal Architecture](#1-captive-portal-architecture)
* [ ] [2. Network Design](#2-network-design)
* [ ] [3. Firewall Policy](#3-firewall-policy)
* [ ] [4. Authentication Source](#4-authentication-source)
* [ ] [5. Local User Authentication](#5-local-user-authentication)
* [ ] [6. LDAP Authentication](#6-ldap-authentication)
* [ ] [7. RADIUS Authentication](#7-radius-authentication)
* [ ] [8. User Groups](#8-user-groups)
* [ ] [9. Guest Management](#9-guest-management)
* [ ] [10. HTTPS Captive Portal](#10-https-captive-portal)
* [ ] [11. Authentication Timeout](#11-authentication-timeout)
* [ ] [12. Guest Network Security](#12-guest-network-security)
* [ ] [13. Email Collection Portal](#13-email-collection-portal)
* [ ] [14. Authentication Monitoring](#14-authentication-monitoring)
* [ ] [15. Troubleshooting](#15-troubleshooting)
* [ ] [16. Common Failure Scenarios](#16-common-failure-scenarios)
* [ ] [17. Security Hardening](#17-security-hardening)
* [ ] [18. NSE4 / NSE7 High-Value Notes](#18-nse4--nse7-high-value-notes)
* [ ] [19. Quick CLI Reference](#19-quick-cli-reference)
* [ ] [20. Final Validation](#20-final-validation)

---

# 1. Captive Portal Architecture

### Core Authentication Flow

```text
                    Guest Client
                         │
                         ▼
                   Guest VLAN
                         │
                         ▼
                     FortiGate
                         │
                         ▼
                 Firewall Policy
                         │
                         ▼
                Captive Portal
                         │
              ┌──────────┼──────────┐
              │          │          │
              ▼          ▼          ▼
            Local       LDAP      RADIUS
              │          │          │
              └──────────┼──────────┘
                         │
                         ▼
                  Authentication
                         │
                  ┌──────┴──────┐
                  │             │
                ACCEPT         REJECT
                  │             │
                  ▼             ▼
               Internet       Denied
```

### Architecture Checklist

* [ ] Captive Portal requirement has been identified.
* [ ] Guest users are separated from internal users.
* [ ] Dedicated Guest VLAN/interface is available.
* [ ] Firewall policy is responsible for enforcing access.
* [ ] Authentication source has been selected.
* [ ] User group has been defined.
* [ ] Guest-to-Internet access has been designed.
* [ ] Guest-to-internal access has been explicitly denied.
* [ ] Authentication timeout has been defined.
* [ ] Logging and monitoring requirements have been defined.

---

# 2. Network Design

## Guest VLAN

* [ ] Dedicated Guest VLAN has been created.
* [ ] Guest subnet has been assigned.
* [ ] DHCP scope is configured.
* [ ] DNS configuration is available.
* [ ] Default gateway points to FortiGate.
* [ ] Guest clients cannot directly access internal VLANs.
* [ ] Guest clients cannot access management networks.
* [ ] Guest clients can reach required authentication services.

### Recommended Design

```text
                    FortiGate
                       │
          ┌────────────┴────────────┐
          │                         │
       Internal                  Guest VLAN
          │                         │
       Servers                  Guest Wi-Fi
       Users                         │
       Mgmt                     Captive Portal
                                    │
                                    ▼
                                 Internet
```

### Isolation Checklist

* [ ] Guest → Internet = **ALLOW**
* [ ] Guest → Internal LAN = **DENY**
* [ ] Guest → Server VLAN = **DENY**
* [ ] Guest → Management VLAN = **DENY**
* [ ] Guest → Security infrastructure = **DENY unless required**
* [ ] Guest → Authentication services = **ALLOW where required**

> **Security principle:** Guest access should be designed as **Internet access first**, not as another internal user segment.

---

# 3. Firewall Policy

Captive Portal authentication must be integrated with the appropriate firewall policy.

### Policy Checklist

* [ ] Source interface = Guest interface/VLAN.
* [ ] Destination interface = WAN or required destination.
* [ ] Source address is correctly defined.
* [ ] Destination address is correctly defined.
* [ ] User/group requirements are configured.
* [ ] Schedule is appropriate.
* [ ] Required services are permitted.
* [ ] NAT is enabled where Internet access requires it.
* [ ] Authentication is enforced through the intended policy.
* [ ] More-specific deny policies exist where required.

### Example

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

### Policy Validation

```text
Guest Client
     │
     ▼
Ingress Interface
     │
     ▼
Firewall Policy Match
     │
     ▼
Authentication
     │
 ┌───┴────┐
 ▼        ▼
Reject   Accept
          │
          ▼
       Internet
```

* [ ] Correct policy is matched.
* [ ] No earlier policy bypasses authentication.
* [ ] NAT is working.
* [ ] Routing is correct.
* [ ] Return traffic is permitted.

---

# 4. Authentication Source

Select the authentication backend based on the deployment.

| Authentication Source | Typical Use                         |
| --------------------- | ----------------------------------- |
| Local Users           | Small environments / testing        |
| LDAP                  | Enterprise directory authentication |
| RADIUS                | Central authentication / NPS        |
| Guest Management      | Temporary guest accounts            |
| User Groups           | Identity-based policy control       |

### Checklist

* [ ] Authentication source selected.
* [ ] Authentication server reachable.
* [ ] Credentials verified.
* [ ] User group created.
* [ ] Group linked to the required policy.
* [ ] Authentication logging enabled.
* [ ] Failure behavior tested.

---

# 5. Local User Authentication

### Local User Flow

```text
Client
  │
  ▼
Captive Portal
  │
  ▼
FortiGate Local Database
  │
 ┌┴──────────┐
 ▼           ▼
Accept      Reject
```

### Checklist

* [ ] Local user created.
* [ ] Username configured.
* [ ] Strong password configured.
* [ ] User assigned to appropriate group.
* [ ] User is not unnecessarily privileged.
* [ ] Expiration is configured where appropriate.
* [ ] Login tested from Guest network.
* [ ] Logout/re-authentication tested.

---

# 6. LDAP Authentication

### LDAP Flow

```text
Guest Client
     │
     ▼
Captive Portal
     │
     ▼
FortiGate
     │
     ▼
LDAP Server
     │
     ▼
Directory
```

### LDAP Checklist

* [ ] LDAP server IP/FQDN configured.
* [ ] LDAP connectivity verified.
* [ ] Correct LDAP type selected.
* [ ] Base DN configured correctly.
* [ ] CN/username attribute is correct.
* [ ] Bind account configured where required.
* [ ] Bind password is valid.
* [ ] LDAP user exists.
* [ ] LDAP group mapping is correct.
* [ ] Firewall can reach LDAP server.
* [ ] Authentication test succeeds.

### Example

```bash
config user ldap
    edit "LDAP"
        set server <LDAP-SERVER>
        set cnid "sAMAccountName"
        set dn "dc=example,dc=local"
        set type regular
        set username <BIND-USER>
        set password <PASSWORD>
    next
end
```

> **Never commit real LDAP passwords or bind credentials to GitHub.**

### LDAP Test

```bash
diagnose test authserver ldap <server> <username> <password>
```

* [ ] Authentication returns success.
* [ ] User/group membership is correct.
* [ ] LDAP authentication is not blocked by routing/firewall rules.

---

# 7. RADIUS Authentication

### RADIUS Flow

```text
Client
  │
  ▼
Captive Portal
  │
  ▼
FortiGate
  │
  │ RADIUS
  ▼
RADIUS Server
  │
 ┌┴───────────────┐
 ▼                ▼
Access-Accept   Access-Reject
```

### RADIUS Checklist

* [ ] RADIUS server configured.
* [ ] Correct server IP/FQDN configured.
* [ ] Correct authentication port configured.
* [ ] Shared secret configured correctly.
* [ ] FortiGate/NAS IP is recognized by RADIUS.
* [ ] Authentication method is compatible.
* [ ] RADIUS policy permits the request.
* [ ] User exists.
* [ ] User/group attributes are correct.
* [ ] Authentication test succeeds.

### Common Ports

```text
UDP/1812 → Authentication
UDP/1813 → Accounting
```

> Always verify the exact RADIUS implementation and ports used by the deployment.

---

# 8. User Groups

User groups connect authentication identity to firewall access control.

### Identity Chain

```text
User
 │
 ▼
Authentication Server
 │
 ▼
User Group
 │
 ▼
Firewall Policy
 │
 ▼
Network Access
```

### Checklist

* [ ] User group created.
* [ ] Correct authentication server added.
* [ ] LDAP/RADIUS membership verified.
* [ ] Group is referenced by the intended firewall policy.
* [ ] Unauthenticated users cannot bypass the policy.
* [ ] Different guest groups have appropriate access levels.

### Example Logical Model

```text
guest-users
├── guest01
├── guest02
└── guest03
```

---

# 9. Guest Management

Guest accounts are useful for temporary access.

### Guest Account Checklist

* [ ] Guest account created.
* [ ] Username generated.
* [ ] Strong/random password configured.
* [ ] Expiration time configured.
* [ ] Maximum access duration defined.
* [ ] Guest account owner/operator is known.
* [ ] Expired accounts are removed or disabled.
* [ ] Guest credentials are distributed securely.
* [ ] Guest activity is logged.

### Guest Lifecycle

```text
Create
  │
  ▼
Activate
  │
  ▼
Authenticate
  │
  ▼
Access
  │
  ▼
Expire
  │
  ▼
Disable / Remove
```

---

# 10. HTTPS Captive Portal

Authentication pages should use HTTPS where supported and appropriate.

### Checklist

* [ ] HTTPS captive portal is enabled.
* [ ] Appropriate certificate is configured.
* [ ] Certificate is valid.
* [ ] Certificate is trusted by clients where possible.
* [ ] Certificate hostname/SAN matches the intended name.
* [ ] Certificate is not expired.
* [ ] HTTP-to-HTTPS behavior has been tested.
* [ ] Mobile clients have been tested.
* [ ] Browser certificate warnings have been investigated.

### Authentication Flow

```text
HTTP Request
     │
     ▼
FortiGate
     │
     ▼
HTTPS Redirect
     │
     ▼
Captive Portal
     │
     ▼
Authentication
```

### Certificate Validation

* [ ] Validity period checked.
* [ ] Issuer checked.
* [ ] SAN checked.
* [ ] Trust chain checked.
* [ ] Client trust verified.

---

# 11. Authentication Timeout

Authentication timeout controls how long authenticated access remains valid.

### Example

```bash
config user setting
    set auth-timeout-type idle-timeout
    set auth-timeout 5
end
```

### Timeout Checklist

* [ ] Timeout requirement identified.
* [ ] Idle timeout considered.
* [ ] Hard timeout considered.
* [ ] Session timeout behavior understood.
* [ ] Guest experience tested after timeout.
* [ ] Re-authentication tested.

### Timeout Model

```text
User Login
    │
    ▼
Authenticated
    │
    ├── Activity continues
    │
    └── Idle period reached
              │
              ▼
        Authentication expires
```

### Timeout Types

| Type              | Concept                                                                    |
| ----------------- | -------------------------------------------------------------------------- |
| `idle-timeout`    | Authentication expires after inactivity                                    |
| `hard-timeout`    | Authentication expires after a fixed period                                |
| `session-timeout` | New sessions are denied after timeout while existing sessions can continue |

> **NSE memory:** Know the behavioral difference between **idle**, **hard**, and **session** timeout.

---

# 12. Guest Network Security

### Mandatory Isolation Checklist

* [ ] Guest → Internal LAN denied.
* [ ] Guest → Server VLAN denied.
* [ ] Guest → Management VLAN denied.
* [ ] Guest → Firewall management denied.
* [ ] Guest → Infrastructure services restricted.
* [ ] Guest → Internet permitted only as required.
* [ ] DNS access controlled.
* [ ] DHCP access controlled.
* [ ] Administrative interfaces are unreachable from Guest VLAN.
* [ ] Inter-client communication is restricted where required.

### Security Model

```text
                   Guest Network
                         │
              ┌──────────┼──────────┐
              │          │          │
              ▼          ▼          ▼
           Internet    Internal    Mgmt
             ✅          ❌          ❌
```

---

# 13. Email Collection Portal

Email collection can be used for specific guest Wi-Fi/hotspot scenarios.

### Checklist

* [ ] Email-collection requirement identified.
* [ ] Captive Portal VAP configured where applicable.
* [ ] Portal type configured appropriately.
* [ ] Firewall policy supports the required behavior.
* [ ] Guest consent/privacy requirements considered.
* [ ] Collected information is handled according to organizational policy.

### Example

```bash
config wireless-controller.vap
    edit "freewifi"
        set security captive-portal
        set portal-type email-collect
    next
end
```

Policy example:

```bash
config firewall policy
    edit 1
        set email-collect enable
    next
end
```

> **Security/privacy note:** Collect only information that is actually required.

---

# 14. Authentication Monitoring

### Check Authenticated Users

```bash
diagnose firewall auth list
```

### Check MAC Authentication Information

```bash
diagnose firewall auth mac list
```

### Monitoring Checklist

* [ ] Authenticated users are visible.
* [ ] Expected users appear.
* [ ] Authentication state is correct.
* [ ] Unexpected users are investigated.
* [ ] Authentication failures are logged.
* [ ] Guest sessions are monitored.
* [ ] Expired sessions disappear as expected.

---

# 15. Troubleshooting

## Step 1 — Client Connectivity

* [ ] Client received an IP address.
* [ ] DHCP is working.
* [ ] Default gateway is correct.
* [ ] DNS is working.
* [ ] Client can reach FortiGate.
* [ ] Client is actually connected to the Guest VLAN.

---

## Step 2 — Firewall Policy

* [ ] Correct policy exists.
* [ ] Correct source interface.
* [ ] Correct source address.
* [ ] Correct destination interface.
* [ ] Correct destination address.
* [ ] Correct service.
* [ ] Correct user/group configuration.
* [ ] NAT is enabled where required.
* [ ] Policy order does not bypass authentication.

---

## Step 3 — Authentication

```bash
diagnose firewall auth list
```

* [ ] User authentication exists.
* [ ] User belongs to the expected group.
* [ ] Authentication has not expired.
* [ ] Credentials are correct.

---

## Step 4 — LDAP

* [ ] LDAP server reachable.
* [ ] Base DN correct.
* [ ] Username attribute correct.
* [ ] Bind credentials valid.
* [ ] LDAP group membership correct.

Test:

```bash
diagnose test authserver ldap <server> <username> <password>
```

---

## Step 5 — RADIUS

* [ ] RADIUS server reachable.
* [ ] UDP/1812 reachable.
* [ ] NAS IP is correct.
* [ ] Shared secret matches.
* [ ] Authentication method matches.
* [ ] RADIUS policy allows the request.
* [ ] Access-Accept is returned.

---

## Step 6 — Certificate

* [ ] Certificate is valid.
* [ ] Certificate is not expired.
* [ ] SAN is correct.
* [ ] Client trusts the CA.
* [ ] Complete certificate chain is trusted.
* [ ] HTTPS portal loads without unexpected warnings.

---

## Step 7 — Internet Access

* [ ] Authentication succeeds.
* [ ] Firewall policy permits traffic.
* [ ] NAT works.
* [ ] Default route exists.
* [ ] DNS resolution works.
* [ ] Return traffic is allowed.
* [ ] Security profiles are not unintentionally blocking required traffic.

---

# 16. Common Failure Scenarios

| Symptom                                          | Investigation                                             |
| ------------------------------------------------ | --------------------------------------------------------- |
| Portal does not appear                           | VLAN, DHCP, policy, redirect and client connectivity      |
| Login page appears but authentication fails      | Credentials, LDAP/RADIUS and group membership             |
| LDAP fails                                       | DN, server reachability, bind account and LDAP settings   |
| RADIUS fails                                     | NAS IP, shared secret, port and RADIUS policy             |
| User authenticates but Internet fails            | Policy, NAT, routing, DNS                                 |
| User repeatedly sees login page                  | Timeout/session handling                                  |
| HTTPS certificate warning                        | Certificate, SAN, trust chain and expiration              |
| Guest reaches internal network                   | Missing or incorrect isolation policy                     |
| Some applications fail                           | DNS, policy, service restrictions or application behavior |
| Authentication suddenly expires                  | Timeout configuration                                     |
| Guest account still works after expiration       | Guest lifecycle/expiration configuration                  |
| Authentication works but wrong policy is applied | User/group mapping and policy order                       |

---

# 17. Security Hardening

### Network Segmentation

* [ ] Guest network is isolated from corporate networks.
* [ ] Guest VLAN is dedicated.
* [ ] Management interfaces are inaccessible.
* [ ] Internal DNS is not unnecessarily exposed.
* [ ] Internal services are explicitly denied.

### Authentication

* [ ] Strong credentials are used.
* [ ] Temporary accounts have expiration.
* [ ] Authentication timeout is configured.
* [ ] Unused accounts are disabled.
* [ ] LDAP/RADIUS is preferred where centralized identity is required.

### HTTPS

* [ ] HTTPS portal is enabled where appropriate.
* [ ] Trusted certificate is deployed.
* [ ] Certificate expiration is monitored.
* [ ] Certificate hostname/SAN is correct.

### Logging

* [ ] Authentication events are logged.
* [ ] Failed logins are monitored.
* [ ] Guest access is auditable.
* [ ] Security events are forwarded to the central logging platform where required.

### Least Privilege

* [ ] Guest users receive only required access.
* [ ] Guest users cannot reach administrative interfaces.
* [ ] Authentication groups map only to intended policies.
* [ ] Temporary access automatically expires.

---

# 18. NSE4 / NSE7 High-Value Notes

## 🧠 Remember the Authentication Chain

```text
Client
  ↓
Interface / VLAN
  ↓
Firewall Policy
  ↓
Authentication
  ↓
User / Group
  ↓
Access Decision
  ↓
Internet / Resource
```

---

## 🧠 Captive Portal ≠ Authentication Server

```text
Captive Portal
       │
       ▼
Authentication Interface
       │
       ▼
Local / LDAP / RADIUS
       │
       ▼
Identity Verification
```

The Captive Portal is the **access/authentication mechanism**.

LDAP and RADIUS are **identity/authentication backends**.

---

## 🧠 Captive Portal ≠ Authorization

Authentication answers:

> **Who are you?**

Authorization answers:

> **What are you allowed to access?**

```text
Authentication
      │
      ▼
Identity
      │
      ▼
User Group
      │
      ▼
Firewall Policy
      │
      ▼
Authorization
```

---

## 🧠 Guest Network Design

```text
Guest
  │
  ├── Internet       ✅
  ├── Internal LAN   ❌
  ├── Servers        ❌
  └── Management     ❌
```

---

## 🧠 Timeout

```text
idle-timeout
    ↓
Inactivity-based expiration

hard-timeout
    ↓
Fixed authentication lifetime

session-timeout
    ↓
New sessions denied after timeout
```

---

## 🧠 Troubleshooting Order

When Captive Portal fails, do **not** immediately blame authentication.

Use:

```text
1. Client
   ↓
2. DHCP
   ↓
3. DNS
   ↓
4. VLAN / Interface
   ↓
5. Firewall Policy
   ↓
6. Authentication
   ↓
7. LDAP / RADIUS
   ↓
8. NAT
   ↓
9. Routing
   ↓
10. Certificate
```

---

# 19. Quick CLI Reference

## Authentication Users

```bash
diagnose firewall auth list
```

## MAC Authentication

```bash
diagnose firewall auth mac list
```

## LDAP Authentication Test

```bash
diagnose test authserver ldap <server> <username> <password>
```

## Authentication Timeout

```bash
config user setting
    set auth-timeout-type idle-timeout
    set auth-timeout 5
end
```

## LDAP

```bash
config user ldap
    edit "LDAP"
        set server <LDAP-SERVER>
        set cnid "sAMAccountName"
        set dn "dc=example,dc=local"
    next
end
```

## Guest Firewall Policy

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

> **⚠️ Version note:** Exact CLI options can differ between FortiOS releases, FortiGate models and deployment modes. Always validate the command tree on the target FortiOS version.

---

# 20. Final Validation

## 🚀 Production Readiness Checklist

### Network

* [ ] Guest VLAN configured.
* [ ] DHCP working.
* [ ] DNS working.
* [ ] Routing verified.
* [ ] Internet connectivity verified.

### Captive Portal

* [ ] Captive Portal enabled.
* [ ] Authentication flow tested.
* [ ] HTTP/HTTPS behavior tested.
* [ ] HTTPS certificate validated.

### Authentication

* [ ] Local/LDAP/RADIUS source tested.
* [ ] User group tested.
* [ ] Authentication success tested.
* [ ] Authentication failure tested.
* [ ] Timeout tested.
* [ ] Guest expiration tested.

### Security

* [ ] Guest → Internal denied.
* [ ] Guest → Management denied.
* [ ] Guest → Servers denied.
* [ ] Required authentication services allowed.
* [ ] Least-privilege access implemented.

### Monitoring

* [ ] Authentication logs available.
* [ ] Failed authentication monitored.
* [ ] Guest sessions visible.
* [ ] Central logging configured where required.

### Operational

* [ ] Policy order reviewed.
* [ ] NAT verified.
* [ ] Certificate expiration monitored.
* [ ] Guest account lifecycle documented.
* [ ] Troubleshooting procedure documented.

---

# 🎯 Final Mental Model

```text
                         GUEST CLIENT
                              │
                              ▼
                         GUEST VLAN
                              │
                              ▼
                          FORTIGATE
                              │
                              ▼
                     FIREWALL POLICY
                              │
                              ▼
                      CAPTIVE PORTAL
                              │
                 ┌────────────┼────────────┐
                 │            │            │
                 ▼            ▼            ▼
               LOCAL         LDAP        RADIUS
                 │            │            │
                 └────────────┼────────────┘
                              │
                              ▼
                      AUTHENTICATION
                              │
                     ┌────────┴────────┐
                     │                 │
                  REJECT             ACCEPT
                     │                 │
                     ▼                 ▼
                   DENY             AUTHORIZED
                                       │
                                       ▼
                                    INTERNET
```

## 🔥 SheynShield Golden Rules

> **Rule #1 — Captive Portal is not the identity database.**

> **Rule #2 — Authentication tells you who the user is; firewall policy determines what that user can access.**

> **Rule #3 — Guest networks should be isolated from internal and management networks by design.**

> **Rule #4 — Never troubleshoot Captive Portal authentication before verifying VLAN, DHCP, DNS and firewall policy.**

> **Rule #5 — HTTPS captive portals require a certificate strategy, not merely an HTTPS toggle.**

> **Rule #6 — Temporary guest access should have a defined lifecycle: Create → Authenticate → Access → Expire → Disable.**

> **Rule #7 — LDAP/RADIUS failures are often connectivity, identity, group-mapping or policy problems—not necessarily a Captive Portal problem.**

---

# 📚 Related SheynShield Topics

* [ ] FortiGate Firewall Policy
* [ ] FortiGate User Authentication
* [ ] FortiGate LDAP
* [ ] FortiGate RADIUS
* [ ] FortiGate Guest Management
* [ ] FortiGate VLAN
* [ ] FortiGate Wireless
* [ ] FortiAP
* [ ] Network Access Control
* [ ] 802.1X
* [ ] FortiGate Security Profiles
* [ ] FortiGate Logging
* [ ] FortiGate FortiView
* [ ] FortiGate Troubleshooting
* [ ] FortiGate HA

---

# 🔎 Topics

`FortiGate` `FortiOS` `Fortinet` `Captive Portal` `FortiGate Captive Portal` `Guest WiFi` `Guest Network` `LDAP` `RADIUS` `FortiAP` `Network Access Control` `NSE4` `NSE7` `Firewall` `Network Security` `Cyber Security` `Authentication` `Guest Access` `Hotspot` `FortiGate Authentication` `Fortinet Security`

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

**SheynShield — Engineering Secure Networks**

> **Learn it. Verify it. Secure it.**

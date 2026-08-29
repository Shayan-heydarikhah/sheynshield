# 🔐 FortiGate Captive Portal Checklist

> **FortiGate Captive Portal | FortiOS | Guest Wi-Fi | Guest Network | LDAP | RADIUS | Authentication | Network Access Control**
>
> **SheynShield — Engineering Secure Networks**

[![Fortinet](https://img.shields.io/badge/Fortinet-FortiGate-red)](https://www.fortinet.com/products/next-generation-firewall)
[![FortiOS](https://img.shields.io/badge/FortiOS-Captive%20Portal-blue)](https://docs.fortinet.com/)
[![Type](https://img.shields.io/badge/Type-Checklist-green)](#-final-validation)
[![Security](https://img.shields.io/badge/Focus-Network%20Security-orange)](#-security-hardening)

---

## 📋 Table of Contents

* [1. Captive Portal Architecture](#1-captive-portal-architecture)
* [2. Network Design](#2-network-design)
* [3. Firewall Policy](#3-firewall-policy)
* [4. Authentication Source](#4-authentication-source)
* [5. Local User Authentication](#5-local-user-authentication)
* [6. LDAP Authentication](#6-ldap-authentication)
* [7. RADIUS Authentication](#7-radius-authentication)
* [8. User Groups](#8-user-groups)
* [9. Guest Management](#9-guest-management)
* [10. HTTPS Captive Portal](#10-https-captive-portal)
* [11. Authentication Timeout](#11-authentication-timeout)
* [12. Guest Network Security](#12-guest-network-security)
* [13. Email Collection Portal](#13-email-collection-portal)
* [14. Authentication Monitoring](#14-authentication-monitoring)
* [15. Troubleshooting](#15-troubleshooting)
* [16. Common Failure Scenarios](#16-common-failure-scenarios)
* [17. Security Hardening](#17-security-hardening)
* [18. NSE4 / NSE7 High-Value Notes](#18-nse4--nse7-high-value-notes)
* [19. Quick CLI Reference](#19-quick-cli-reference)
* [20. Final Validation](#20-final-validation)
* [21. SheynShield Golden Rules](#21-sheynshield-golden-rules)
* [Related Topics](#-related-topics)

---

# 1. Captive Portal Architecture

## Core Authentication Flow

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
                ┌─────────────┼─────────────┐
                │             │             │
                ▼             ▼             ▼
              Local          LDAP         RADIUS
                │             │             │
                └─────────────┼─────────────┘
                              │
                              ▼
                       Authentication
                              │
                    ┌─────────┴─────────┐
                    │                   │
                  ACCEPT              REJECT
                    │                   │
                    ▼                   ▼
                 Internet              DENY
```

## Architecture Checklist

* [ ] Captive Portal requirement has been identified.
* [ ] Guest users are separated from internal users.
* [ ] Dedicated Guest VLAN/interface is available.
* [ ] Guest subnet has been defined.
* [ ] Firewall policy is responsible for enforcing access.
* [ ] Authentication source has been selected.
* [ ] User group has been defined.
* [ ] Guest-to-Internet access has been designed.
* [ ] Guest-to-internal access has been explicitly denied.
* [ ] Authentication timeout has been defined.
* [ ] Logging and monitoring requirements have been defined.
* [ ] Guest account lifecycle has been defined.

---

# 2. Network Design

## Guest VLAN Checklist

* [ ] Dedicated Guest VLAN has been created.
* [ ] Guest subnet has been assigned.
* [ ] DHCP scope is configured.
* [ ] DNS configuration is available.
* [ ] Default gateway points to FortiGate.
* [ ] Guest clients can reach the FortiGate gateway.
* [ ] Guest clients cannot directly access internal VLANs.
* [ ] Guest clients cannot access management networks.
* [ ] Guest clients can reach required authentication services.
* [ ] Routing has been verified.
* [ ] Internet connectivity has been verified.

## Recommended Architecture

```text
                         FortiGate
                             │
              ┌──────────────┴──────────────┐
              │                             │
          Internal                      Guest VLAN
              │                             │
       ┌──────┼──────┐                 Guest Wi-Fi
       │      │      │                      │
    Users  Servers   Mgmt              Captive Portal
                                             │
                                             ▼
                                          Internet
```

## Guest Isolation Checklist

* [ ] Guest → Internet = **ALLOW**
* [ ] Guest → Internal LAN = **DENY**
* [ ] Guest → Server VLAN = **DENY**
* [ ] Guest → Management VLAN = **DENY**
* [ ] Guest → Security infrastructure = **DENY unless required**
* [ ] Guest → Authentication services = **ALLOW where required**
* [ ] Guest → FortiGate management = **DENY**
* [ ] Guest → Other guest clients = **RESTRICT where required**

> **Security Principle:** Treat the Guest network as an **untrusted network**. Design access from **deny-by-default** rather than assuming guest users are trusted.

---

# 3. Firewall Policy

Captive Portal authentication must be integrated with the appropriate firewall policy and traffic flow.

## Firewall Policy Checklist

* [ ] Source interface = Guest interface/VLAN.
* [ ] Destination interface = WAN or required destination.
* [ ] Source address is correctly defined.
* [ ] Destination address is correctly defined.
* [ ] User/group requirements are correctly configured.
* [ ] Schedule is appropriate.
* [ ] Required services are permitted.
* [ ] NAT is enabled where Internet access requires it.
* [ ] Authentication is enforced through the intended policy.
* [ ] More-specific deny policies exist where required.
* [ ] Policy order has been reviewed.
* [ ] No earlier policy unintentionally bypasses authentication.
* [ ] Return traffic is permitted.
* [ ] Routing is correct.

## Example Policy

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

> **⚠️ Note:** The exact policy requirements depend on the FortiOS release, authentication design and deployment mode. Validate the configuration against the target FortiOS version.

## Policy Flow

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

---

# 4. Authentication Source

Choose the authentication backend based on the deployment requirements.

| Authentication Source | Typical Use                         |
| --------------------- | ----------------------------------- |
| Local Users           | Small environments, labs, testing   |
| LDAP                  | Enterprise directory authentication |
| RADIUS                | Centralized authentication, NPS     |
| Guest Management      | Temporary guest accounts            |
| User Groups           | Identity-based access control       |

## Authentication Checklist

* [ ] Authentication source selected.
* [ ] Authentication server is reachable.
* [ ] Credentials have been verified.
* [ ] User group has been created.
* [ ] Authentication server is associated with the correct group.
* [ ] Group is linked to the intended firewall policy.
* [ ] Authentication logging is enabled.
* [ ] Authentication failure behavior has been tested.

---

# 5. Local User Authentication

## Authentication Flow

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

## Local User Checklist

* [ ] Local user has been created.
* [ ] Username has been configured.
* [ ] Strong password has been configured.
* [ ] User has been assigned to the appropriate group.
* [ ] User is not unnecessarily privileged.
* [ ] Expiration has been configured where appropriate.
* [ ] Login has been tested from the Guest network.
* [ ] Logout has been tested.
* [ ] Re-authentication has been tested.
* [ ] Expired accounts are disabled or removed.

---

# 6. LDAP Authentication

## LDAP Authentication Flow

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
Directory / Active Directory
```

## LDAP Checklist

* [ ] LDAP server IP/FQDN is configured.
* [ ] LDAP connectivity is verified.
* [ ] Correct LDAP type is selected.
* [ ] Base DN is configured correctly.
* [ ] Username/CN attribute is correct.
* [ ] Bind account is configured where required.
* [ ] Bind password is valid.
* [ ] LDAP user exists.
* [ ] LDAP group mapping is correct.
* [ ] Firewall can reach the LDAP server.
* [ ] Authentication test succeeds.

## Example LDAP Configuration

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

> **🔐 Security:** Never commit real LDAP passwords, bind credentials, API keys or secrets to GitHub.

## LDAP Authentication Test

```bash
diagnose test authserver ldap <server> <username> <password>
```

### Validate

* [ ] Authentication returns success.
* [ ] User exists in the directory.
* [ ] User/group membership is correct.
* [ ] LDAP server is reachable.
* [ ] DNS resolution works if FQDN is used.
* [ ] Firewall rules permit the required LDAP traffic.

---

# 7. RADIUS Authentication

## RADIUS Authentication Flow

```text
Client
  │
  ▼
Captive Portal
  │
  ▼
FortiGate
  │
  │ RADIUS Authentication
  ▼
RADIUS Server
  │
 ┌┴───────────────┐
 ▼                ▼
Access-Accept   Access-Reject
```

## RADIUS Checklist

* [ ] RADIUS server is configured.
* [ ] Correct server IP/FQDN is configured.
* [ ] Correct authentication port is configured.
* [ ] Shared secret is configured correctly.
* [ ] FortiGate/NAS IP is recognized by the RADIUS server.
* [ ] Authentication method is compatible.
* [ ] RADIUS policy permits the request.
* [ ] User exists.
* [ ] User/group attributes are correct.
* [ ] Authentication test succeeds.
* [ ] RADIUS logs have been reviewed when troubleshooting.

## Common RADIUS Ports

```text
UDP/1812 → Authentication
UDP/1813 → Accounting
```

> **⚠️ Always verify the exact RADIUS implementation, ports and authentication method used by the deployment.**

---

# 8. User Groups

User groups connect authentication identity to firewall access control.

## Identity Chain

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

## User Group Checklist

* [ ] User group has been created.
* [ ] Correct authentication server has been added.
* [ ] LDAP/RADIUS membership has been verified.
* [ ] Group is referenced by the intended firewall policy.
* [ ] Unauthenticated users cannot bypass the policy.
* [ ] Different guest groups have appropriate access levels.
* [ ] Group-based authorization has been tested.

## Example

```text
guest-users
├── guest01
├── guest02
└── guest03
```

---

# 9. Guest Management

Guest Management is useful for temporary or controlled guest access.

## Guest Account Checklist

* [ ] Guest account has been created.
* [ ] Username has been generated.
* [ ] Strong/random password has been configured.
* [ ] Expiration time has been configured.
* [ ] Maximum access duration has been defined.
* [ ] Guest account owner/operator is known.
* [ ] Expired accounts are removed or disabled.
* [ ] Guest credentials are distributed securely.
* [ ] Guest activity is logged.

## Guest Lifecycle

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

HTTPS should be used for captive portal authentication where supported and appropriate.

## HTTPS Checklist

* [ ] HTTPS captive portal is enabled.
* [ ] Appropriate certificate is configured.
* [ ] Certificate is valid.
* [ ] Certificate is not expired.
* [ ] Certificate hostname/SAN matches the intended name.
* [ ] Certificate trust chain is valid.
* [ ] Client devices trust the issuing CA where possible.
* [ ] HTTP-to-HTTPS behavior has been tested.
* [ ] Mobile clients have been tested.
* [ ] Browser certificate warnings have been investigated.

## Authentication Flow

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

## Certificate Validation

* [ ] Validity period checked.
* [ ] Issuer checked.
* [ ] SAN checked.
* [ ] Trust chain checked.
* [ ] Client trust verified.
* [ ] Certificate expiration monitoring is configured.

> **🔐 Security Principle:** HTTPS captive portal security depends on a correct **certificate and trust strategy**, not simply enabling HTTPS.

---

# 11. Authentication Timeout

Authentication timeout controls how long authenticated access remains valid.

## Example

```bash
config user setting
    set auth-timeout-type idle-timeout
    set auth-timeout 5
end
```

## Timeout Checklist

* [ ] Timeout requirement has been identified.
* [ ] Idle timeout has been considered.
* [ ] Hard timeout has been considered.
* [ ] Session timeout behavior is understood.
* [ ] Guest experience has been tested after timeout.
* [ ] Re-authentication has been tested.
* [ ] Timeout values match the security requirement.

## Timeout Model

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
       Authentication Expires
```

## Timeout Types

| Type              | Behavior                                                                   |
| ----------------- | -------------------------------------------------------------------------- |
| `idle-timeout`    | Authentication expires after inactivity                                    |
| `hard-timeout`    | Authentication expires after a fixed period                                |
| `session-timeout` | New sessions are denied after timeout while existing sessions can continue |

> **🧠 NSE Memory:** Understand the behavioral difference between **idle**, **hard**, and **session** timeout.

---

# 12. Guest Network Security

## Mandatory Isolation Checklist

* [ ] Guest → Internal LAN = **DENY**
* [ ] Guest → Server VLAN = **DENY**
* [ ] Guest → Management VLAN = **DENY**
* [ ] Guest → Firewall management = **DENY**
* [ ] Guest → Security infrastructure = restricted
* [ ] Guest → Internet = allowed only as required
* [ ] DNS access is controlled.
* [ ] DHCP access is controlled.
* [ ] Administrative interfaces are unreachable from the Guest VLAN.
* [ ] Inter-client communication is restricted where required.
* [ ] East-west guest traffic has been evaluated.

## Security Model

```text
                   Guest Network
                         │
              ┌──────────┼──────────┐
              │          │          │
              ▼          ▼          ▼
           Internet    Internal    Mgmt
             ✅          ❌          ❌
```

> **Golden Principle:** A Guest VLAN should be treated as **untrusted**.

---

# 13. Email Collection Portal

Email collection can be used in specific guest Wi-Fi and hotspot scenarios.

## Checklist

* [ ] Email collection requirement has been identified.
* [ ] Captive Portal VAP is configured where applicable.
* [ ] Portal type is configured appropriately.
* [ ] Firewall policy supports the required behavior.
* [ ] Guest consent/privacy requirements have been considered.
* [ ] Only required information is collected.
* [ ] Data handling follows organizational requirements.

## Example

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

### Typical Use Cases

* [ ] Hotels
* [ ] Retail
* [ ] Shopping Centers
* [ ] Guest Wi-Fi
* [ ] Public Wi-Fi
* [ ] Marketing Wi-Fi
* [ ] Hotspot deployments

---

# 14. Authentication Monitoring

## Authenticated Users

```bash
diagnose firewall auth list
```

## MAC Authentication Information

```bash
diagnose firewall auth mac list
```

## Monitoring Checklist

* [ ] Authenticated users are visible.
* [ ] Expected users appear.
* [ ] Authentication state is correct.
* [ ] Unexpected users are investigated.
* [ ] Authentication failures are logged.
* [ ] Guest sessions are monitored.
* [ ] Expired sessions disappear as expected.
* [ ] Authentication events are available for investigation.

---

# 15. Troubleshooting

## Step 1 — Client Connectivity

* [ ] Client received an IP address.
* [ ] DHCP is working.
* [ ] Default gateway is correct.
* [ ] DNS is working.
* [ ] Client can reach FortiGate.
* [ ] Client is actually connected to the Guest VLAN.
* [ ] VLAN tagging is correct.
* [ ] Wireless SSID/VAP mapping is correct where applicable.

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
* [ ] Policy order has been reviewed.
* [ ] Authentication is actually being enforced.

---

## Step 3 — Authentication

```bash
diagnose firewall auth list
```

* [ ] User authentication exists.
* [ ] User belongs to the expected group.
* [ ] Authentication has not expired.
* [ ] Credentials are correct.
* [ ] Authentication failure reason has been investigated.

---

## Step 4 — LDAP

* [ ] LDAP server is reachable.
* [ ] Base DN is correct.
* [ ] Username attribute is correct.
* [ ] Bind credentials are valid.
* [ ] LDAP group membership is correct.
* [ ] DNS resolution works if required.

Test:

```bash
diagnose test authserver ldap <server> <username> <password>
```

---

## Step 5 — RADIUS

* [ ] RADIUS server is reachable.
* [ ] UDP/1812 is reachable.
* [ ] NAS IP is correct.
* [ ] Shared secret matches.
* [ ] Authentication method matches.
* [ ] RADIUS policy allows the request.
* [ ] Access-Accept is returned.
* [ ] RADIUS logs have been reviewed.

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
* [ ] Upstream connectivity is healthy.

---

# 16. Common Failure Scenarios

| Symptom                                     | Primary Investigation Areas                             |
| ------------------------------------------- | ------------------------------------------------------- |
| Portal does not appear                      | VLAN, DHCP, policy, redirect, client connectivity       |
| Login page appears but authentication fails | Credentials, LDAP/RADIUS, group membership              |
| LDAP authentication fails                   | DN, server reachability, bind account, LDAP settings    |
| RADIUS authentication fails                 | NAS IP, shared secret, port, RADIUS policy              |
| User authenticates but Internet fails       | Policy, NAT, routing, DNS                               |
| User repeatedly sees login page             | Timeout, session handling, authentication state         |
| HTTPS certificate warning                   | Certificate, SAN, trust chain, expiration               |
| Guest reaches internal network              | Missing/incorrect isolation policy                      |
| Some applications fail                      | DNS, policy, service restrictions, application behavior |
| Authentication suddenly expires             | Timeout configuration                                   |
| Guest account still works after expiration  | Guest lifecycle/expiration configuration                |
| Wrong firewall policy is applied            | User/group mapping, policy order                        |
| Authentication server cannot be reached     | Routing, DNS, firewall policy, server availability      |

---

# 17. Security Hardening

## Network Segmentation

* [ ] Guest network is isolated from corporate networks.
* [ ] Guest VLAN is dedicated.
* [ ] Management interfaces are inaccessible.
* [ ] Internal DNS is not unnecessarily exposed.
* [ ] Internal services are explicitly denied.
* [ ] Guest-to-guest communication is restricted where required.

## Authentication

* [ ] Strong credentials are used.
* [ ] Temporary accounts have expiration.
* [ ] Authentication timeout is configured.
* [ ] Unused accounts are disabled.
* [ ] Centralized authentication is used where appropriate.
* [ ] Authentication groups follow least privilege.

## HTTPS

* [ ] HTTPS portal is enabled where appropriate.
* [ ] Trusted certificate is deployed.
* [ ] Certificate expiration is monitored.
* [ ] Certificate hostname/SAN is correct.
* [ ] Certificate chain is valid.

## Logging

* [ ] Authentication events are logged.
* [ ] Failed logins are monitored.
* [ ] Guest access is auditable.
* [ ] Central logging is configured where required.
* [ ] Suspicious authentication behavior is investigated.

## Least Privilege

* [ ] Guest users receive only required access.
* [ ] Guest users cannot reach administrative interfaces.
* [ ] Authentication groups map only to intended policies.
* [ ] Temporary access automatically expires.
* [ ] Guest access is reviewed periodically.

---

# 18. NSE4 / NSE7 High-Value Notes

## 🧠 Authentication Chain

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

### Exam Checklist

* [ ] Understand where Captive Portal fits in the traffic flow.
* [ ] Understand the relationship between User → Group → Policy.
* [ ] Understand Local vs LDAP vs RADIUS authentication.
* [ ] Understand Guest Management.
* [ ] Understand authentication timeout behavior.
* [ ] Understand Guest VLAN isolation.
* [ ] Understand HTTPS/certificate requirements.
* [ ] Know the key authentication troubleshooting commands.

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

**Captive Portal** provides the access/authentication mechanism.

**LDAP/RADIUS** provide external identity/authentication services.

---

## 🧠 Authentication ≠ Authorization

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

> **NSE troubleshooting principle:** Start with the lower layers and traffic path before assuming the authentication backend is the problem.

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

> **⚠️ Version Note:** Exact CLI options can differ between FortiOS releases, FortiGate models and deployment modes. Always validate the command tree on the target FortiOS version.

---

# 20. Final Validation

## 🚀 Production Readiness Checklist

### Network

* [ ] Guest VLAN configured.
* [ ] DHCP working.
* [ ] DNS working.
* [ ] Routing verified.
* [ ] Internet connectivity verified.
* [ ] Guest-to-internal connectivity tested and denied.

### Captive Portal

* [ ] Captive Portal enabled.
* [ ] Authentication flow tested.
* [ ] HTTP/HTTPS behavior tested.
* [ ] HTTPS certificate validated.
* [ ] Mobile client behavior tested.

### Authentication

* [ ] Local/LDAP/RADIUS source tested.
* [ ] User group tested.
* [ ] Authentication success tested.
* [ ] Authentication failure tested.
* [ ] Timeout tested.
* [ ] Guest expiration tested.
* [ ] Re-authentication tested.

### Security

* [ ] Guest → Internal denied.
* [ ] Guest → Management denied.
* [ ] Guest → Servers denied.
* [ ] Required authentication services allowed.
* [ ] Least-privilege access implemented.
* [ ] Guest-to-guest access reviewed.

### Monitoring

* [ ] Authentication logs available.
* [ ] Failed authentication monitored.
* [ ] Guest sessions visible.
* [ ] Central logging configured where required.
* [ ] Authentication anomalies can be investigated.

### Operational

* [ ] Policy order reviewed.
* [ ] NAT verified.
* [ ] Certificate expiration monitored.
* [ ] Guest account lifecycle documented.
* [ ] Troubleshooting procedure documented.
* [ ] Configuration backup/change record maintained.
* [ ] FortiOS version documented.

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

---

# 🔥 SheynShield Golden Rules

> **Rule #1 — Captive Portal is not the identity database.**

> **Rule #2 — Authentication tells you who the user is; authorization determines what the user can access.**

> **Rule #3 — Guest networks should be isolated from internal and management networks by design.**

> **Rule #4 — Never troubleshoot Captive Portal authentication before verifying VLAN, DHCP, DNS and firewall policy.**

> **Rule #5 — HTTPS captive portals require a certificate strategy, not merely an HTTPS toggle.**

> **Rule #6 — Temporary guest access should have a defined lifecycle: Create → Authenticate → Access → Expire → Disable.**

> **Rule #7 — LDAP/RADIUS failures can originate from connectivity, identity, group mapping, policy or server-side configuration—not necessarily the Captive Portal itself.**

> **Rule #8 — Treat the Guest network as untrusted and apply least-privilege access.**

---

# 📚 Related Topics

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
* [ ] FortiView
* [ ] FortiGate Troubleshooting
* [ ] FortiGate HA

---

# 🔎 Keywords

`FortiGate Captive Portal`
`FortiOS Captive Portal`
`Fortinet Captive Portal`
`FortiGate Guest WiFi`
`FortiGate Guest Network`
`FortiGate LDAP Authentication`
`FortiGate RADIUS Authentication`
`FortiGate User Authentication`
`FortiGate Guest Management`
`FortiAP Captive Portal`
`FortiGate Authentication Troubleshooting`
`FortiGate Guest VLAN`
`FortiGate Network Access Control`
`FortiGate Firewall Policy`
`NSE4 Captive Portal`
`NSE7 FortiGate Authentication`
`Fortinet Network Security`
`FortiGate Security Checklist`

---

# 🔗 SheynShield Resources

### 🎥 Video Learning

* **YouTube — SheynShield**

  * Fortinet NSE content
  * FortiGate troubleshooting
  * Network Security Engineering

### 📚 Notes & Updates

* **Telegram — SheynShield**

  * Technical notes
  * Security content
  * Fortinet updates

### 💼 Professional Network

* **LinkedIn — Shayan Heydari**

  * Network Security Engineering
  * Fortinet
  * Network Design
  * Cybersecurity

### 🐙 Technical Knowledge Base

* **SheynShield GitHub**

  * Fortinet Cheat Sheets
  * Security Checklists
  * Network Engineering Notes
  * FortiGate Troubleshooting

---

## 🏷️ Tags

`#FortiGate` `#FortiOS` `#Fortinet` `#CaptivePortal` `#GuestWiFi` `#GuestNetwork` `#LDAP` `#RADIUS` `#FortiAP` `#NetworkSecurity` `#CyberSecurity` `#NSE4` `#NSE7` `#Firewall` `#Authentication` `#NetworkAccessControl`

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


**SheynShield — Engineering Secure Networks**

> **Learn it. Verify it. Secure it.**

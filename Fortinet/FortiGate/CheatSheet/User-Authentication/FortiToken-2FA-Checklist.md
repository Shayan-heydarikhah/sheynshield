# 🔐 FortiGate 2FA & FortiToken Checklist

> **FortiOS 2FA | FortiToken | FortiToken Mobile | OTP | FTM Push | Token Lifecycle | Synchronization | SSL-VPN | Administrator Security | Troubleshooting**

A practical **FortiGate 2FA and FortiToken checklist** for designing, configuring, validating, securing, and troubleshooting **FortiToken, FortiToken Mobile (FTM), OTP authentication, administrator 2FA, SSL-VPN 2FA, token provisioning, synchronization, and recovery**.

---

## 📌 Table of Contents

- [1. 2FA Architecture](#1-2fa-architecture)
- [2. FortiToken Components](#2-fortitoken-components)
- [3. FortiToken Deployment Checklist](#3-fortitoken-deployment-checklist)
- [4. FortiToken Mobile Provisioning](#4-fortitoken-mobile-provisioning)
- [5. Administrator 2FA](#5-administrator-2fa)
- [6. Remote User 2FA](#6-remote-user-2fa)
- [7. Authentication Sources](#7-authentication-sources)
- [8. Token Lifecycle](#8-token-lifecycle)
- [9. Token Provisioning Expiry](#9-token-provisioning-expiry)
- [10. OTP Synchronization](#10-otp-synchronization)
- [11. Time & NTP Validation](#11-time--ntp-validation)
- [12. Token Expiry Settings](#12-token-expiry-settings)
- [13. FortiToken Mobile Push](#13-fortitoken-mobile-push)
- [14. FortiToken Connectivity](#14-fortitoken-connectivity)
- [15. SSL-VPN + 2FA](#15-ssl-vpn--2fa)
- [16. FortiToken RMA & Recovery](#16-fortitoken-rma--recovery)
- [17. Lost Token Procedure](#17-lost-token-procedure)
- [18. Break-Glass Administrator](#18-break-glass-administrator)
- [19. FortiToken Import Troubleshooting](#19-fortitoken-import-troubleshooting)
- [20. Authentication Debugging](#20-authentication-debugging)
- [21. Security Hardening Checklist](#21-security-hardening-checklist)
- [22. Production Validation](#22-production-validation)
- [23. Troubleshooting Decision Tree](#23-troubleshooting-decision-tree)
- [24. NSE Exam Memory Map](#24-nse-exam-memory-map)
- [25. Quick CLI Reference](#25-quick-cli-reference)
- [26. One-Minute Troubleshooting](#26-one-minute-troubleshooting)
- [27. Final Security Checklist](#27-final-security-checklist)
- [28. Keywords](#28-keywords)

---

# 1. 2FA Architecture

## Two-Factor Authentication Flow

```text
                 USER
                   │
                   ▼
          Username + Password
                   │
                   ▼
          Primary Authentication
                   │
             ┌─────┴─────┐
             │           │
            LDAP       RADIUS
             │           │
             └─────┬─────┘
                   │
                   ▼
              FortiToken
                   │
                   ▼
                OTP / Push
                   │
                   ▼
             Second Factor
                   │
                   ▼
            Authentication
                Success
                   │
                   ▼
           Authorization Policy
                   │
                   ▼
                Access
````

### Authentication vs Authorization

* [ ] Verify primary authentication
* [ ] Verify second-factor authentication
* [ ] Verify user/group authorization
* [ ] Verify firewall/VPN policy
* [ ] Verify destination/resource permissions

> **Core concept:**
> **Authentication = Who are you?**
> **Authorization = What are you allowed to access?**

---

# 2. FortiToken Components

| Component                   | Purpose                                                  |
| --------------------------- | -------------------------------------------------------- |
| **FortiToken**              | Hardware OTP token                                       |
| **FortiToken Mobile (FTM)** | Mobile OTP application                                   |
| **Seed**                    | Cryptographic secret used by the token                   |
| **OTP**                     | One-time authentication code                             |
| **Provisioning**            | Mobile token assignment and activation                   |
| **FortiGuard/FDS**          | Fortinet services involved in supported token operations |

### Token Checklist

* [ ] Identify hardware vs mobile token
* [ ] Verify token serial
* [ ] Verify token ownership
* [ ] Verify token assignment
* [ ] Verify token status
* [ ] Protect token/activation information
* [ ] Never publish real seeds or activation codes

---

# 3. FortiToken Deployment Checklist

## GUI

Navigate to:

```text
User & Authentication
└── FortiTokens
```

### Deployment

* [ ] Open FortiToken management
* [ ] Import/create the required token
* [ ] Verify token appears in inventory
* [ ] Verify token status
* [ ] Assign token to the correct user
* [ ] Enable 2FA for the user
* [ ] Verify authentication policy references the user/group
* [ ] Test OTP authentication

---

# 4. FortiToken Mobile Provisioning

Typical workflow:

```text
FortiGate
   │
   ▼
Token Assignment
   │
   ▼
Provisioning
   │
   ▼
Activation Information
   │
   ▼
FortiToken Mobile
   │
   ▼
User Activates Token
   │
   ▼
Provisioned
   │
   ▼
OTP Authentication
```

### Provisioning Checklist

* [ ] FortiToken Mobile is available
* [ ] Token is assigned to the correct user
* [ ] User email is correct
* [ ] User mobile number is correct where required
* [ ] Provisioning service is reachable
* [ ] Activation information is delivered
* [ ] FTM application is installed
* [ ] User completes activation
* [ ] Token status changes to provisioned/active
* [ ] OTP authentication is tested

---

# 5. Administrator 2FA

Administrator accounts are high-value authentication targets.

## Recommended Design

```text
Trusted Management Network
            │
            ▼
      Administrator
            │
     ┌──────┴──────┐
     │             │
 Strong Password  FortiToken
     │             │
     └──────┬──────┘
            ▼
           2FA
            │
            ▼
      Admin Profile
            │
            ▼
    FortiGate Management
```

### Administrator Checklist

* [ ] Enable 2FA for privileged administrators
* [ ] Assign individual tokens
* [ ] Avoid shared administrator accounts
* [ ] Use least-privilege admin profiles
* [ ] Restrict management interfaces
* [ ] Restrict administrative source IPs
* [ ] Verify administrator email/mobile information
* [ ] Test token authentication
* [ ] Maintain emergency recovery access

### Avoid

```text
Internet
   │
   ▼
FortiGate GUI
   │
   ▼
Super Admin
   │
   └── No 2FA
```

### Prefer

```text
Trusted Management Network
          │
          ▼
     Admin Account
          │
          ▼
         2FA
          │
          ▼
  Least-Privilege Profile
```

---

# 6. Remote User 2FA

Common architecture:

```text
Remote User
     │
     ▼
SSL-VPN
     │
     ▼
Primary Authentication
     │
     ├── LDAP
     └── RADIUS
     │
     ▼
FortiToken
     │
     ▼
OTP
     │
     ▼
Authentication Success
     │
     ▼
VPN Access
```

### Remote User Checklist

* [ ] Verify LDAP/RADIUS authentication
* [ ] Verify user exists
* [ ] Verify user/group mapping
* [ ] Assign FortiToken
* [ ] Enable 2FA
* [ ] Verify SSL-VPN authentication configuration
* [ ] Verify OTP validation
* [ ] Verify VPN group/policy
* [ ] Test successful login
* [ ] Test failed OTP
* [ ] Test expired OTP

---

# 7. Authentication Sources

Common authentication sources:

```text
                    Authentication
                          │
             ┌────────────┼────────────┐
             │            │            │
           Local        LDAP        RADIUS
             │            │            │
             └────────────┼────────────┘
                          │
                          ▼
                     FortiToken
                          │
                          ▼
                         2FA
```

| Source                 | Typical Use                                             |
| ---------------------- | ------------------------------------------------------- |
| **Local**              | Local FortiGate users                                   |
| **LDAP**               | Directory-based authentication                          |
| **RADIUS**             | Centralized remote authentication                       |
| **FortiAuthenticator** | Centralized authentication and Fortinet token workflows |

### Checklist

* [ ] Identify primary authentication source
* [ ] Verify authentication server connectivity
* [ ] Verify username/group mapping
* [ ] Verify token assignment
* [ ] Verify 2FA enforcement
* [ ] Verify authorization policy

---

# 8. Token Lifecycle

Understanding token state is critical during troubleshooting.

| State                 | Meaning                               |
| --------------------- | ------------------------------------- |
| **New**               | Newly added token                     |
| **Available**         | Available for assignment              |
| **Active**            | Assigned and active                   |
| **Provisioning**      | Mobile token awaiting activation      |
| **Provisioned**       | Mobile token successfully activated   |
| **Provision Timeout** | Provisioning window expired           |
| **Locked**            | Token is locked                       |
| **Already Activated** | Token/seed has already been activated |

## Lifecycle

```text
New
 │
 ▼
Available
 │
 ▼
Assigned
 │
 ▼
Provisioning
 │
 ▼
Activated
 │
 ▼
Provisioned / Active
 │
 ├──────────────┐
 │              │
 ▼              ▼
Locked       Expired
 │              │
 ▼              ▼
Recovery     Re-provision
```

### Token State Checklist

* [ ] Token exists
* [ ] Token is assigned
* [ ] Token is active
* [ ] Token is not locked
* [ ] Mobile token is provisioned
* [ ] Provisioning has not expired
* [ ] Token is not already activated elsewhere

---

# 9. Token Provisioning Expiry

Example:

```bash
config system global
    set two-factor-ftm-expire 72
end
```

### Interpretation

```text
72 hours
   │
   ▼
Provisioning Window
   │
   ├── Activated → Continue
   │
   └── Not Activated → Provisioning Timeout
```

### Checklist

* [ ] Verify FTM provisioning expiry
* [ ] Confirm user activates within the allowed period
* [ ] Check token status after assignment
* [ ] Re-provision expired tokens
* [ ] Verify target FortiOS syntax

> **Important:** CLI names and available options can vary between FortiOS releases. Validate commands against the target version.

---

# 10. OTP Synchronization

Time-based OTP depends on synchronized time.

```text
FortiGate
    +
FortiToken
    +
Mobile Device
    │
    ▼
Correct Time
    │
    ▼
Valid OTP
```

### Synchronization Checklist

* [ ] FortiGate system time is correct
* [ ] FortiGate timezone is correct
* [ ] NTP is configured
* [ ] NTP synchronization is healthy
* [ ] Mobile device time is correct
* [ ] Token is not experiencing excessive drift
* [ ] OTP generation is functioning
* [ ] Current/next OTP values are available when synchronization is required

### Synchronize Token

```bash
execute fortitoken sync <token-id> <current-token-code> <next-token-code>
```

### Concept

```text
Token ID
   +
Current OTP
   +
Next OTP
   │
   ▼
Synchronization
   │
   ▼
OTP Validation
```

---

# 11. Time & NTP Validation

OTP troubleshooting should always include time validation.

### Verify

```text
FortiGate Time
      ≈
Mobile Device Time
      ≈
Trusted NTP Time Source
```

### Checklist

* [ ] Verify system clock
* [ ] Verify timezone
* [ ] Verify NTP server
* [ ] Verify NTP synchronization
* [ ] Verify mobile device time
* [ ] Verify no significant clock drift
* [ ] Re-test OTP after correcting time

### Common Symptom

```text
Correct Password
       │
       ▼
OTP Rejected
       │
       ▼
Check Time
       │
       ▼
Check NTP
       │
       ▼
Check Token Synchronization
```

---

# 12. Token Expiry Settings

Example configuration:

```bash
config system global
    set two-factor-ftk-expiry <seconds>
    set two-factor-ftm-expiry <seconds>
    set two-factor-sms-expiry <seconds>
    set two-factor-fac-expiry <seconds>
    set two-factor-email-expiry <seconds>
end
```

| Setting                   | Purpose                                   |
| ------------------------- | ----------------------------------------- |
| `two-factor-ftk-expiry`   | FortiToken hardware authentication expiry |
| `two-factor-ftm-expiry`   | FortiToken Mobile expiry                  |
| `two-factor-sms-expiry`   | SMS OTP expiry                            |
| `two-factor-fac-expiry`   | FortiAuthenticator-related 2FA expiry     |
| `two-factor-email-expiry` | Email OTP expiry                          |

### Security Checklist

* [ ] Avoid unnecessarily long OTP validity
* [ ] Balance usability and security
* [ ] Validate expiry behavior
* [ ] Test expired OTP
* [ ] Document production values

---

# 13. FortiToken Mobile Push

Supported Fortinet architectures can use FortiToken Mobile push authentication.

Typical architecture:

```text
SSL-VPN
   │
   ▼
FortiGate
   │
   ▼
FortiAuthenticator
   │
   ▼
FTM Push
   │
   ▼
User Approval
   │
   ▼
Authentication Success
```

### Push Checklist

* [ ] FortiAuthenticator is reachable
* [ ] RADIUS configuration is correct where applicable
* [ ] FTM is provisioned
* [ ] Push service is enabled where required
* [ ] FortiGate can reach required services
* [ ] User receives push notification
* [ ] User approval is recorded
* [ ] Authentication completes successfully

---

# 14. FortiToken Connectivity

Token provisioning and management may require connectivity to Fortinet services.

### Example Tests

```bash
execute ping fds1.fortinet.com
execute ping directregistration.fortinet.com
execute ping globalftm.fortinet.net
```

### Connectivity Checklist

* [ ] DNS resolution works
* [ ] Internet connectivity works
* [ ] FortiGate can reach required Fortinet services
* [ ] HTTPS connectivity is allowed
* [ ] Routing is correct
* [ ] Firewall policies permit required traffic
* [ ] FortiGuard connectivity is healthy
* [ ] System time is correct
* [ ] NTP is working

> **Note:** `ping` verifies ICMP reachability, not necessarily the application-layer service. Validate the actual required service connectivity as appropriate.

---

# 15. SSL-VPN + 2FA

## Authentication Flow

```text
SSL-VPN Client
      │
      ▼
Username + Password
      │
      ▼
LDAP / RADIUS
      │
      ▼
FortiToken
      │
      ▼
OTP
      │
      ▼
Authentication Success
      │
      ▼
SSL-VPN Session
      │
      ▼
Firewall Policy
      │
      ▼
Protected Resource
```

### Remote Authentication Timeout

Example:

```bash
config system global
    set remoteauthtimeout 300
end
```

```text
300 seconds
     =
5 minutes
```

### SSL-VPN Checklist

* [ ] Verify primary authentication
* [ ] Verify FortiToken assignment
* [ ] Verify OTP validation
* [ ] Verify user group
* [ ] Verify VPN portal
* [ ] Verify firewall policy
* [ ] Verify `remoteauthtimeout`
* [ ] Test successful authentication
* [ ] Test invalid OTP
* [ ] Test timeout behavior

---

# 16. FortiToken RMA & Recovery

A FortiToken should not be treated as an ordinary configuration object that can simply be copied between FortiGate devices.

### RMA / Migration Checklist

* [ ] Identify affected FortiGate
* [ ] Identify token ownership
* [ ] Identify token serials
* [ ] Verify token status
* [ ] Confirm target FortiGate
* [ ] Preserve required configuration
* [ ] Follow supported Fortinet transfer/RMA procedures
* [ ] Verify token reassignment
* [ ] Test authentication
* [ ] Document the operation

### Concept

```text
Old FortiGate
      │
      ▼
Token Lifecycle / Supported Transfer
      │
      ▼
New FortiGate
      │
      ▼
Token Reassignment
      │
      ▼
Authentication Test
```

---

# 17. Lost Token Procedure

If a FortiToken is lost:

```text
Lost Token
    │
    ▼
Identify User
    │
    ▼
Lock / Disable Token
    │
    ▼
Verify Recovery Path
    │
    ▼
Assign Replacement
    │
    ▼
Provision Replacement
    │
    ▼
Test Authentication
```

### Lost Token Checklist

* [ ] Identify affected user
* [ ] Lock/disable the lost token
* [ ] Verify the user identity
* [ ] Verify alternate authentication path
* [ ] Assign replacement token
* [ ] Provision replacement token
* [ ] Test OTP
* [ ] Remove old token assignment where required
* [ ] Document incident/recovery

---

# 18. Break-Glass Administrator

A production FortiGate should have a documented emergency access strategy.

### Recommended Concept

```text
Primary Admin
     │
     └── Strong Password + 2FA

Break-Glass Admin
     │
     └── Emergency Recovery Procedure
```

### Checklist

* [ ] Maintain emergency administrative access
* [ ] Protect break-glass credentials
* [ ] Restrict access to trusted management paths
* [ ] Monitor use
* [ ] Document recovery procedure
* [ ] Test recovery periodically
* [ ] Avoid dependency on a single token infrastructure

> **Security principle:** Do not create an architecture where every administrator becomes inaccessible if the same token infrastructure fails.

---

# 19. FortiToken Import Troubleshooting

Example errors may include:

```text
Import FortiToken license error: -7571
```

or:

```text
Import FortiToken license error: -7566
```

### Investigation Checklist

* [ ] Verify token serial
* [ ] Verify activation information
* [ ] Verify token ownership
* [ ] Verify FortiGate registration
* [ ] Verify FortiGuard connectivity
* [ ] Verify DNS
* [ ] Verify system time
* [ ] Verify token status
* [ ] Retry supported provisioning/import process
* [ ] Check FortiOS version compatibility

> **Important:** Error-code interpretation can depend on the FortiOS/token workflow and version. Confirm against the relevant Fortinet documentation before treating a numeric code as definitive.

---

# 20. Authentication Debugging

## FortiToken Information

```bash
diagnose fortitoken info
```

## Token Database

```bash
show user fortitoken
```

## Token Synchronization

```bash
execute fortitoken sync <token-id> <current-code> <next-code>
```

## Authentication Debug

```bash
diagnose debug enable
diagnose debug application authd -1
```

### Stop Debugging

```bash
diagnose debug disable
```

### Debugging Checklist

* [ ] Reproduce the authentication problem
* [ ] Enable only required debugging
* [ ] Capture relevant output
* [ ] Identify primary authentication failure
* [ ] Identify second-factor failure
* [ ] Check token state
* [ ] Check time synchronization
* [ ] Check connectivity
* [ ] Disable debugging after testing

---

# 21. Security Hardening Checklist

## Identity

* [ ] Use centralized authentication where appropriate
* [ ] Use individual user identities
* [ ] Avoid shared administrator accounts
* [ ] Use strong passwords
* [ ] Enable 2FA for privileged users
* [ ] Use least-privilege administrator profiles

## FortiToken

* [ ] Assign tokens to the correct users
* [ ] Monitor token status
* [ ] Lock lost tokens immediately
* [ ] Protect activation information
* [ ] Protect token seeds
* [ ] Document token ownership
* [ ] Establish token recovery procedures

## Management Plane

* [ ] Restrict administrative access
* [ ] Use trusted management networks
* [ ] Disable unnecessary administrative services
* [ ] Use secure management protocols
* [ ] Monitor administrator authentication
* [ ] Maintain break-glass access

## Infrastructure

* [ ] Configure reliable NTP
* [ ] Configure correct DNS
* [ ] Verify FortiGuard connectivity
* [ ] Verify Fortinet service reachability
* [ ] Maintain supported FortiOS/token versions

---

# 22. Production Validation

Before production deployment:

### Authentication

* [ ] Username/password authentication works
* [ ] LDAP/RADIUS authentication works
* [ ] 2FA is enforced
* [ ] Correct user group is selected

### FortiToken

* [ ] Token is assigned
* [ ] Token is active
* [ ] Mobile token is provisioned
* [ ] OTP is accepted
* [ ] Invalid OTP is rejected
* [ ] Expired OTP is rejected
* [ ] Lost token procedure is documented

### Time

* [ ] FortiGate clock is correct
* [ ] NTP is synchronized
* [ ] Mobile device time is correct
* [ ] Token synchronization is healthy

### Recovery

* [ ] Break-glass account exists
* [ ] Emergency recovery procedure is documented
* [ ] Token replacement procedure is documented
* [ ] RMA/migration process is documented
* [ ] Recovery path has been tested

---

# 23. Troubleshooting Decision Tree

```text
                    2FA FAILURE
                        │
             ┌──────────┴──────────┐
             │                     │
       Password Fails          OTP Fails
             │                     │
             ▼                     ▼
       Check LDAP/RADIUS      Check Token
                                   │
                     ┌─────────────┼─────────────┐
                     │             │             │
                  Locked        Expired       Wrong OTP
                     │             │             │
                     ▼             ▼             ▼
                  Unlock       Re-provision    Check Time
                                                   │
                                                   ▼
                                                  NTP
                                                   │
                                                   ▼
                                           Token Synchronize
```

---

## Fast Troubleshooting Sequence

```text
1. Primary Authentication
        ↓
2. User / Group
        ↓
3. Token Assignment
        ↓
4. Token Status
        ↓
5. Provisioning Status
        ↓
6. FortiGate Time
        ↓
7. Mobile Device Time
        ↓
8. NTP
        ↓
9. Fortinet Connectivity
        ↓
10. OTP Synchronization
        ↓
11. VPN / Firewall Policy
        ↓
12. Authentication Debug
```

### Diagnostic Questions

| Question                                   | Check                    |
| ------------------------------------------ | ------------------------ |
| Does password authentication work?         | LDAP/RADIUS/local        |
| Is the token assigned?                     | FortiToken database      |
| Is the token active?                       | Token status             |
| Is FTM provisioned?                        | Provisioning state       |
| Is token locked?                           | Token status             |
| Did provisioning expire?                   | FTM expiry               |
| Is the OTP wrong?                          | Time/NTP/token sync      |
| Can FortiGate reach Fortinet services?     | DNS/routing/connectivity |
| Does the user belong to the correct group? | User/group mapping       |
| Does SSL-VPN reference the correct group?  | VPN configuration        |
| Is authentication timing out?              | `remoteauthtimeout`      |
| Is deeper debugging required?              | `authd` debug            |

---

# 24. NSE Exam Memory Map

```text
                    FORTIGATE 2FA
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
    Primary Auth      FortiToken        Policy
        │                │                │
   LDAP/RADIUS       OTP / Push      Authorization
        │                │                │
        └────────────────┼────────────────┘
                         │
                         ▼
                       Access
```

### Remember

```text
FortiToken
    ↓
Second Factor

FortiToken Mobile
    ↓
Mobile OTP / Push

Seed
    ↓
Cryptographic Secret

Provisioning
    ↓
Assign + Activate

Provision Timeout
    ↓
Re-provision

Clock Drift
    ↓
OTP Problems

NTP
    ↓
Critical Time Dependency

LDAP / RADIUS
    ↓
Primary / Remote Authentication

2FA
    ↓
Authentication ≠ Authorization
```

---

# 25. Quick CLI Reference

## Configure FTM Provisioning Expiry

```bash
config system global
    set two-factor-ftm-expire 72
end
```

## Synchronize FortiToken

```bash
execute fortitoken sync <token-id> <current-code> <next-code>
```

## Display FortiToken Information

```bash
diagnose fortitoken info
```

## Display Token Database

```bash
show user fortitoken
```

## Test Fortinet Service Reachability

```bash
execute ping fds1.fortinet.com
execute ping directregistration.fortinet.com
execute ping globalftm.fortinet.net
```

## Configure Remote Authentication Timeout

```bash
config system global
    set remoteauthtimeout 300
end
```

## FTM Push

```bash
config system ftm-push
    set status enable
    set server-ip <FTM-SERVER>
    set server-port 4433
end
```

## Authentication Debug

```bash
diagnose debug enable
diagnose debug application authd -1
```

## Disable Debug

```bash
diagnose debug disable
```

> **Version note:** Always validate CLI syntax and behavior against the target FortiOS release.

---

# 26. One-Minute Troubleshooting

```text
[ ] Password authentication works
[ ] Correct user is being authenticated
[ ] Correct user group is selected
[ ] FortiToken is assigned
[ ] Token is active/provisioned
[ ] Token is not locked
[ ] Provisioning has not expired
[ ] FortiGate time is correct
[ ] Mobile device time is correct
[ ] NTP is synchronized
[ ] FortiGate DNS works
[ ] Fortinet services are reachable
[ ] OTP is generated correctly
[ ] Token synchronization is healthy
[ ] SSL-VPN/VPN policy references correct group
[ ] Authorization policy is correct
[ ] Remote authentication timeout is appropriate
[ ] Debugging was disabled after testing
```

---

# 27. Final Security Checklist

## 🔐 Identity

* [ ] Individual identities are used
* [ ] Strong passwords are enforced
* [ ] Centralized authentication is used where appropriate
* [ ] Privileged users require 2FA

## 🔑 FortiToken

* [ ] Every privileged account has an appropriate token
* [ ] Token ownership is documented
* [ ] Token lifecycle is monitored
* [ ] Lost tokens can be revoked
* [ ] Replacement procedure exists
* [ ] Provisioning procedure exists

## ⏱️ Time

* [ ] NTP is configured
* [ ] FortiGate time is correct
* [ ] Mobile device time is correct
* [ ] Token synchronization is verified

## 🌐 Connectivity

* [ ] DNS works
* [ ] Routing works
* [ ] FortiGuard connectivity works
* [ ] Required Fortinet services are reachable
* [ ] Required firewall rules are permitted

## 🛡️ Authorization

* [ ] User groups are correct
* [ ] VPN policies are correct
* [ ] Firewall policies are correct
* [ ] Least privilege is applied

## 🚨 Recovery

* [ ] Break-glass administrator exists
* [ ] Lost-token procedure exists
* [ ] RMA/recovery procedure exists
* [ ] Emergency authentication path is documented
* [ ] Recovery has been tested

---

# 28. Keywords

`FortiGate 2FA` · `FortiToken` · `FortiToken Mobile` · `FTM` · `FortiOS 2FA` · `FortiGate OTP` · `FortiToken troubleshooting` · `FortiToken synchronization` · `FortiToken provisioning` · `FortiToken Mobile push` · `FortiGate SSL-VPN 2FA` · `FortiGate administrator 2FA` · `FortiGate LDAP 2FA` · `FortiGate RADIUS 2FA` · `FortiToken RMA` · `FortiToken recovery` · `FortiGate authentication` · `FortiOS authentication` · `FortiToken NTP` · `FortiGate OTP troubleshooting` · `Fortinet two factor authentication`

---

# 🧠 SheynShield Quick Recall

```text
              FORTIGATE 2FA
                    │
       ┌────────────┼────────────┐
       │            │            │
       ▼            ▼            ▼
   Identity      FortiToken   Authorization
       │            │            │
 LDAP/RADIUS      OTP/Push     Policy
       │            │            │
       └────────────┼────────────┘
                    │
                    ▼
                  ACCESS
```

### Critical Relationships

```text
Password
   +
FortiToken
   =
2FA
```

```text
Wrong Time
   ↓
Wrong OTP
```

```text
Provisioning Expired
   ↓
Re-provision
```

```text
Lost Token
   ↓
Lock
   ↓
Replace
   ↓
Re-provision
```

```text
2FA
   ≠
Authorization
```

```text
Authentication
      +
Authorization
      +
Recovery
      =
Production-Ready Security
```

---

# 🎯 Core Takeaway

> **FortiToken 2FA is not simply an OTP feature. A production-ready deployment requires identity management, token lifecycle control, time synchronization, service connectivity, authorization, monitoring, and a tested recovery strategy.**

```text
Identity
   +
Primary Authentication
   +
FortiToken
   +
OTP / Push
   +
Time Synchronization
   +
Connectivity
   +
Authorization
   +
Recovery
   │
   ▼
Secure Access
```

---

## 📚 SheynShield Reference

| Field            | Value                                             |
| ---------------- | ------------------------------------------------- |
| **Topic**        | FortiGate 2FA & FortiToken                        |
| **Category**     | User & Authentication                             |
| **Level**        | NSE4 → NSE7                                       |
| **Technologies** | FortiToken, FTM, LDAP, RADIUS, SSL-VPN            |
| **Use Cases**    | Admin 2FA, VPN 2FA, Remote Authentication         |
| **Format**       | Checklist + Lab Reference + Troubleshooting Guide |

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

`FortiGate` `FortiToken` `FortiToken Mobile` `2FA` `OTP` `LDAP` `RADIUS` `SSL-VPN` `Identity Security` `Network Security`

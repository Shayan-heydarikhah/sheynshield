# FortiGate Administrator, Authentication & Management Security 

> **FortiOS 7.2.x — Administrator Accounts, Admin Profiles, LDAP, MFA, Trusted Hosts, REST API, SSO, Password Policy & SSH Public-Key Authentication**

---

## Table of Contents

* [1. System Settings Overview](#1-system-settings-overview)
* [2. Administrator Management](#2-administrator-management)

  * [2.1 Local Administrators](#21-local-administrators)
  * [2.2 Remote Administrators](#22-remote-administrators)
  * [2.3 LDAP Remote Authentication](#23-ldap-remote-authentication)
  * [2.4 Wildcard Remote Administrators](#24-wildcard-remote-administrators)
  * [2.5 Restrict Local Administrator Logins](#25-restrict-local-administrator-logins)
* [3. Administrator Profiles](#3-administrator-profiles)
* [4. Multi-Factor Authentication](#4-multi-factor-authentication)
* [5. Trusted Hosts](#5-trusted-hosts)
* [6. VDOM Administrator Scope](#6-vdom-administrator-scope)
* [7. REST API Administrators](#7-rest-api-administrators)
* [8. SSO Administrators](#8-sso-administrators)
* [9. FortiCloud SSO](#9-forticloud-sso)
* [10. Administrator Password Policy](#10-administrator-password-policy)
* [11. SSH Public-Key Authentication](#11-ssh-public-key-authentication)
* [12. Operating Modes](#12-operating-modes)
* [13. Administrator Security Hardening Checklist](#13-administrator-security-hardening-checklist)
* [14. NSE Exam Quick Review](#14-nse-exam-quick-review)

---

# 1. System Settings Overview

FortiGate administrator and management configuration is primarily organized under:

```text
System
└── Settings
    ├── Basic
    │   ├── Administrators
    │   ├── Admin Profiles
    │   ├── Password Policies
    │   └── Interfaces
    │
    └── Advanced
        ├── HA
        ├── VDOM
        ├── SNMP
        └── Certificates
```

### Management Operating Modes

FortiGate can operate in:

| Mode                 | Description                                             |
| -------------------- | ------------------------------------------------------- |
| **NAT/Route Mode**   | FortiGate operates as a Layer-3 security gateway/router |
| **Transparent Mode** | FortiGate operates transparently at Layer 2             |

---

# 2. Administrator Management

FortiGate supports multiple administrator authentication models:

```text
Administrator
│
├── Local Authentication
│   └── Local username + password
│
├── Remote Authentication
│   ├── LDAP
│   ├── Other configured authentication servers
│   └── Remote groups
│
├── MFA
│   ├── FortiToken
│   └── SMS
│
├── REST API
│   └── API users
│
├── SSO
│   └── SAML / Security Fabric
│
└── FortiCloud SSO
```

---

## 2.1 Local Administrators

A local administrator is authenticated directly by the FortiGate.

Typical configuration:

```text
System
└── Administrators
    └── Create New
        ├── Username
        ├── Password
        ├── Administrator Profile
        ├── Trusted Hosts
        ├── VDOM
        └── Two-Factor Authentication
```

### ⚠️ Administrator Username Security

Do **not** use these characters in a local administrator username:

```text
< > ( ) # " '
```

Using these characters in administrator usernames can introduce security risks, including potential **cross-site scripting (XSS)** issues.

### Recommended

```text
admin-sec
network-admin
fw-admin
readonly-admin
```

### Avoid

```text
admin<script>
admin#
admin"name
admin()
```

---

# 2.2 Remote Administrators

Remote administrators authenticate against an external authentication server such as LDAP.

### Authentication Flow

```text
Administrator
      │
      │ Login
      ▼
 FortiGate
      │
      │ Remote Authentication
      ▼
 LDAP / Authentication Server
      │
      │ Validate credentials
      ▼
 User / Group
      │
      ▼
FortiGate Administrator Profile
      │
      ▼
Access Granted
```

### Important Prerequisite

Before creating a remote administrator:

> **Configure the remote authentication server first and define the required user groups.**

For LDAP-based administration:

```text
FortiGate
   │
   ├── LDAP Server
   │
   └── LDAP User Group
          │
          └── Administrator Account/Profile
```

---

# 2.3 LDAP Remote Authentication

When using LDAP for administrator authentication, the FortiGate must be able to communicate with the LDAP server.

### Basic Design

```text
                    LDAP Server
                 ┌───────────────┐
                 │ Users         │
                 │ Groups        │
                 │ Passwords     │
                 └───────┬───────┘
                         │
                         │ LDAP
                         ▼
                 ┌───────────────┐
                 │   FortiGate   │
                 │               │
                 │ Remote Admin  │
                 └───────────────┘
```

### Critical Point

When configuring a remote administrator, the username should normally correspond to the username available on the remote authentication source.

For example:

```text
LDAP:
    username = sheyn

FortiGate Remote Admin:
    username = sheyn
```

---

# 2.4 Wildcard Remote Administrators

A wildcard administrator allows **multiple remote users** from a configured remote group to authenticate using the same administrator configuration.

Instead of creating:

```text
user1
user2
user3
user4
...
```

you can create one wildcard administrator:

```text
LDAP Group
     │
     ├── user1
     ├── user2
     ├── user3
     └── user4
          │
          ▼
Wildcard Administrator
          │
          ▼
Administrator Profile
```

### Match All Users

If all users belonging to a remote server group should be permitted, configure the administrator as a wildcard administrator.

Example:

```bash
config system admin
    edit "x-u1"
        set remote-auth enable
        set accprofile "super_admin"
        set wildcard enable
        set remote-group "ldap-local-fw"
    next
end
```

### Meaning

```text
set remote-auth enable
```

Enables remote authentication.

```text
set wildcard enable
```

Allows the administrator entry to match remote users rather than a single specific username.

```text
set remote-group "ldap-local-fw"
```

Restricts authentication to members of the configured remote group.

### Security Principle

Do **not** automatically assign:

```text
super_admin
```

to an LDAP group unless every member of that group genuinely requires full administrative access.

Prefer:

```text
LDAP Group
      │
      ├── Read-only users
      │       └── read-only profile
      │
      ├── Network admins
      │       └── network administration profile
      │
      └── Security admins
              └── security administration profile
```

### 🔐 Preferred Authentication

For highly privileged administrative access, consider stronger authentication mechanisms such as:

```text
PKI / Certificate Authentication
+
MFA
+
Trusted Hosts
+
Least-Privilege Administrator Profile
```

---

# 2.5 Restrict Local Administrator Logins

FortiOS can restrict local administrator logins while configured remote authentication servers are available.

### Feature

```text
admin-restrict-local
```

When enabled:

> FortiOS checks whether the remote authentication servers used by administrators are unavailable before allowing a local administrator to authenticate.

### CLI

```bash
config system global
    set admin-restrict-local enable
end
```

### Default

```text
Disabled
```

### Concept

```text
Remote Authentication Servers
          │
          ▼
      Available?
       /      \
     YES       NO
      │         │
      ▼         ▼
 Local Admin   Local Admin
 Restricted    Allowed
```

### Why Use It?

This can reduce the possibility of bypassing centralized authentication by using local administrator accounts while the remote authentication infrastructure is operational.

### ⚠️ Operational Consideration

Always maintain a carefully controlled emergency/break-glass strategy.

Do not design an authentication architecture where a failure of the external identity infrastructure results in complete administrative lockout.

---

# 3. Administrator Profiles

An **Administrator Profile** determines what an administrator is allowed to do on the FortiGate.

Think of it as:

```text
Authentication
      │
      ▼
"Who are you?"
      │
      ▼
Administrator Account
      │
      ▼
Administrator Profile
      │
      ▼
"What are you allowed to do?"
```

### Typical Privilege Model

```text
Super Admin
    │
    ├── Full system administration
    └── Global configuration

Network Admin
    │
    ├── Interfaces
    ├── Routing
    └── Network-related configuration

Security Admin
    │
    ├── Firewall policies
    ├── Security profiles
    └── Security configuration

Read Only
    │
    └── Monitoring / viewing configuration
```

### Least Privilege

Use the smallest administrator profile required.

```text
❌ Everyone → super_admin

✅ Helpdesk → read-only
✅ Network team → network-related profile
✅ Security team → security-related profile
✅ Automation → dedicated API profile
```

---

# 4. Multi-Factor Authentication

FortiGate administrators can use MFA to increase authentication security.

Common options include:

```text
FortiToken
SMS
```

For privileged administrators, token-based MFA is generally preferred over weaker out-of-band methods.

---

## 4.1 FortiToken Administrator MFA

Example:

```bash
config system admin
    edit "u1"
        set password "1qaz@WSX"
        set two-factor fortitoken
        set fortitoken "123-456-789"
        set email-to "admin@example.com"
    next
end
```

> Replace example credentials and token values with real, securely generated values. Never commit real credentials or tokens to Git.

### MFA Flow

```text
Username
   │
   ▼
Password
   │
   ▼
FortiToken
   │
   ▼
Authentication Success
   │
   ▼
FortiGate GUI / CLI
```

---

## 4.2 NTP Is Critical for Token-Based MFA

FortiToken-based authentication depends on accurate time synchronization.

Therefore:

```text
FortiGate
   │
   └── NTP
        │
        └── Accurate Time
              │
              ▼
        Token Validation
```

### Operational Checklist

```text
[ ] NTP configured
[ ] Time synchronized
[ ] Time zone configured correctly
[ ] Token assigned to correct administrator
[ ] Token status verified
```

### Important

If the FortiGate clock drifts significantly, time-based token authentication can fail.

---

## 4.3 Backup / Emergency Administrator

For a critical firewall, maintain a controlled emergency access strategy.

A common design is:

```text
Normal Admin
    │
    ├── MFA
    ├── LDAP / centralized authentication
    └── Daily operations

Break-Glass Admin
    │
    ├── Local account
    ├── Strong unique password
    ├── Highly restricted access
    └── Securely stored credentials
```

### ⚠️ Important

Do not treat a break-glass account as a normal daily-use administrator.

Its purpose is:

```text
Identity infrastructure failure
        +
Authentication failure
        +
Emergency recovery
```

---

# 4.4 FortiToken Cloud

**FortiToken Cloud** is Fortinet's cloud-based identity and access management service that can provide MFA capabilities for FortiGate and FortiAuthenticator environments.

Conceptually:

```text
User
 │
 ▼
FortiGate
 │
 ▼
FortiToken Cloud
 │
 ▼
MFA Verification
```

---

# 5. Trusted Hosts

Trusted Hosts restrict where an administrator can log in from.

Example:

```text
192.168.20.200/32
192.168.254.0/24
```

### Example Design

```text
Management Network
192.168.254.0/24
        │
        │
        ▼
   FortiGate
        │
        ├── Admin Access
        │
        └── Trusted Host Restriction
```

### Host vs Subnet

Single host:

```text
192.168.20.200/32
```

Subnet:

```text
192.168.254.0/24
```

### Maximum

Up to **10 trusted hosts** can be specified for an administrator.

---

## 5.1 Trusted Host Behavior

When trusted hosts are defined for all administrators, administrative access on each interface is restricted according to the trusted-host configuration.

Conceptually:

```text
Administrator
     │
     ▼
Source IP
     │
     ▼
Trusted Host Check
     │
 ┌───┴────┐
 │        │
MATCH   NO MATCH
 │        │
 ▼        ▼
ALLOW    DENY
```

### ⚠️ Ping Exception

If `PING` administrative access is enabled on an interface, ping can still work regardless of trusted-host restrictions.

Therefore:

```text
Trusted Host ≠ ICMP restriction
```

If you want to restrict management exposure:

```text
Trusted Hosts
+
Interface Administrative Access
+
Network ACL / Firewall Design
+
Dedicated Management Network
```

---

# 6. VDOM Administrator Scope

When VDOMs are enabled, administrator access can be scoped to a specific VDOM.

Conceptually:

```text
FortiGate
│
├── root
│   ├── admin-root
│   └── network-admin
│
├── VDOM-A
│   └── admin-a
│
└── VDOM-B
    └── admin-b
```

### Security Principle

Use VDOM-level administration when administrators should manage only a specific virtual domain.

```text
Global Administrator
        │
        ▼
Multiple VDOMs

VDOM Administrator
        │
        ▼
Specific VDOM
```

---

# 7. REST API Administrators

FortiGate provides API-based administration for:

* Automated configuration
* Configuration backup
* Monitoring
* Automation
* Integration with external systems

### API Architecture

```text
Automation System
       │
       │ REST API
       ▼
 FortiGate API User
       │
       ▼
Administrator Profile
       │
       ▼
VDOM / Global Scope
```

### Important Security Principle

By default, REST API users/groups should be treated as **read-only** unless explicitly granted additional permissions.

### API Administrator Configuration

Example:

```bash
config system api-user
    edit "api-admin"
        set api-key <API-KEY>
        set accprofile "api-prof-test"
    next
end
```

### Generate API Key

Example:

```bash
execute api-user generate-key test-user-api
```

> **Never publish a real API key in GitHub, documentation, screenshots, tickets, or chat.**

---

## 7.1 API Administrator Scope

When creating an API administrator profile, define the required scope:

```text
Global
   OR
Specific VDOM
```

### Prefer Least Privilege

```text
Automation Job
      │
      ▼
Dedicated API User
      │
      ▼
Dedicated API Profile
      │
      ▼
Required VDOM
      │
      ▼
Required permissions only
```

Avoid:

```text
Automation
    ↓
super_admin
```

unless there is a specific operational requirement.

---

## 7.2 FNDN

Fortinet Developer Network (**FNDN**) is relevant when developing and integrating solutions around Fortinet products and APIs.

Typical API workflow:

```text
Developer
   │
   ▼
FNDN / API Documentation
   │
   ▼
API User
   │
   ▼
FortiGate REST API
   │
   ▼
Automation / Integration
```

---

# 8. SSO Administrators

FortiGate can support administrator Single Sign-On using SAML-based authentication.

When FortiGate acts as a **SAML Service Provider (SP)** with SAML SSO enabled in the Security Fabric environment, SSO administrator accounts can be created automatically after successful authentication.

### SSO Flow

```text
Administrator
      │
      ▼
FortiGate Login
      │
      ▼
SAML Authentication
      │
      ▼
Identity Provider
      │
      ▼
Authentication Success
      │
      ▼
FortiGate
      │
      ▼
SSO Administrator
```

### First Successful Login

After the first successful login, the SSO user can be added to the administrator table under:

```text
System
└── Administrators
    └── SSO Administrators
```

---

## 8.1 Default Administrator Profile

The administrator profile assigned to an SSO administrator is based on the configured SAML Service Provider settings.

Conceptually:

```text
SAML Authentication
       │
       ▼
SP Settings
       │
       ▼
Default Admin Profile
       │
       ▼
SSO Administrator
```

### Administrator Configuration

Within the SSO administrator configuration, define:

```text
VDOM
User
Administrator Profile
```

---

# 9. FortiCloud SSO

FortiGate can allow administrators to authenticate using **FortiCloud Single Sign-On**.

### Supported Account Types

FortiCloud SSO can support:

```text
IAM users
Non-IAM users
```

For non-IAM users, the account must correspond to the FortiCloud account associated with the FortiGate registration.

### Enable FortiCloud SSO

```bash
config system global
    set admin-forticloud-sso-login enable
end
```

### Conceptual Flow

```text
Administrator
      │
      ▼
FortiGate
      │
      ▼
FortiCloud SSO
      │
      ▼
Identity / IAM
      │
      ▼
Authentication
      │
      ▼
FortiGate Administrator Access
```

### IAM Portal Permissions

For IAM-based administration, appropriate portal permissions must be configured, including permissions related to:

```text
Support Site
IAM Portal
FortiOS SSO
```

---

# 10. Administrator Password Policy

FortiGate supports password policies for stronger authentication.

Password policy can control:

| Parameter            | Purpose                                              |
| -------------------- | ---------------------------------------------------- |
| Minimum length       | Minimum password size                                |
| Lowercase characters | Require lowercase letters                            |
| Uppercase characters | Require uppercase letters                            |
| Numbers              | Require numeric characters                           |
| Non-alphanumeric     | Require special characters                           |
| Password expiration  | Force periodic password changes                      |
| Change characters    | Require sufficient difference from previous password |
| Password reuse       | Prevent reuse of previous passwords                  |

---

## 10.1 Password Security Recommendations

### Avoid

```text
companyname
administrator
password
Password123
passw0rd
admin123
password1
```

### Prefer

```text
Long
+
Unique
+
Unpredictable
+
Not reused
```

Use a password manager or secure password generator where appropriate.

### Password Length

Administrator passwords can be up to **64 characters**.

> **Exam Note:** Do not confuse password maximum length with the CLI range of individual password-policy parameters.

---

## 10.2 Password Policy CLI

Example:

```bash
config system password-policy
    set status enable
    set apply-to admin-password
    set minimum-length <8-128>
    set min-lower-case-letter <0-128>
    set min-upper-case-letter <0-128>
    set min-non-alphanumeric <0-128>
    set min-number <0-128>
    set min-change-characters <0-128>
    set expire-status enable
    set expire-day <1-999>
    set reuse-password disable
end
```

### `apply-to`

Possible targets include:

```text
admin-password
ipsec-preshared-key
```

or both, depending on the configuration.

---

## 10.3 Password Policy Example

A stronger policy could conceptually require:

```text
Minimum Length        → Long enough
Lowercase             → Required
Uppercase             → Required
Number                → Required
Special Character     → Required
Expiration            → Enabled if required by policy
Password Reuse        → Disabled
Change Characters     → Enforced
```

### Password Lifecycle

```text
Strong Password
      │
      ▼
Unique
      │
      ▼
Not Reused
      │
      ▼
Securely Stored
      │
      ▼
Rotated according to policy
```

### ⚠️ Important

Do not make passwords predictable through rotation:

```text
Password1
Password2
Password3
```

This is not meaningful password hygiene.

---

## 10.4 Centralized Authentication

For enterprise environments, consider integrating administrator authentication with centralized identity infrastructure:

```text
Active Directory
       │
       ▼
LDAP / Authentication Service
       │
       ▼
FortiGate
```

Benefits include:

* Centralized user management
* Centralized group membership
* Easier onboarding/offboarding
* Reduced local-account sprawl
* Consistent authentication policy

---

# 11. SSH Public-Key Authentication

SSH public-key authentication provides an alternative to password-based SSH authentication.

### Key Pair

```text
SSH Key Pair
│
├── Public Key
│   └── Stored on FortiGate
│
└── Private Key
    └── Stored securely on administrator/client system
```

### Authentication Flow

```text
SSH Client
   │
   ├── Username
   │
   └── Private Key
          │
          ▼
       FortiGate
          │
          ▼
    Public Key Match
          │
          ▼
       Login
```

---

## 11.1 Public-Key Authentication Advantages

### Pros

* More resistant to password guessing
* Private key does not need to be transmitted to FortiGate
* Authentication can be restricted to systems possessing the private key
* Can be combined with a passphrase/password
* Excellent for automation and scripts

### Cons

* More complex to deploy
* Private-key security becomes critical
* Requires operational knowledge
* More difficult for inexperienced users to manage

---

# 11.2 Password Authentication vs SSH Keys

| Feature                         | Password | SSH Public Key |
| ------------------------------- | -------: | -------------: |
| Easy to deploy                  |        ✅ |              ❌ |
| Easy to remember                |        ✅ |              ❌ |
| Resistant to brute force        |        ❌ |              ✅ |
| Private key required            |        ❌ |              ✅ |
| Suitable for automation         |       ⚠️ |              ✅ |
| Depends on private-key security |        ❌ |              ✅ |
| Can use passphrase              |        — |              ✅ |

### Key Security Principle

> **The security of SSH public-key authentication depends heavily on protecting the private key.**

---

# 11.3 SSH Keys for Automation

Public-key authentication is particularly useful when connecting to FortiGate without human interaction.

Example:

```text
Automation Script
       │
       ▼
SSH Private Key
       │
       ▼
FortiGate
       │
       ▼
SSH Public Key
       │
       ▼
Authentication
```

This avoids embedding reusable administrator passwords directly into scripts.

---

# 11.4 Generate SSH Key with PuTTYgen

A common Windows workflow is:

```text
PuTTYgen
   │
   ├── Select key type
   │
   ├── Select key length
   │
   ├── Generate key pair
   │
   ├── Save private key
   │
   └── Save/copy public key
```

### RSA

RSA is a commonly used key type in traditional SSH deployments.

### Key Length

Use a security-appropriate key length supported by the target FortiOS/SSH implementation.

### Comment Field

If the comment is not required for identification or management, it can be cleared before copying the public key.

---

# 11.5 Configure SSH Public Key on FortiGate

Example:

```bash
config system admin
    edit "adminx"
        set ssh-public-key1 "<SSH-PUBLIC-KEY>"
    next
end
```

The public key is associated with the administrator account.

### Concept

```text
adminx
  │
  └── ssh-public-key1
          │
          └── Public Key
```

---

# 12. Operating Modes

FortiGate supports different operating modes.

## NAT/Route Mode

```text
Client
  │
  ▼
FortiGate
  │
  ├── Routing
  ├── NAT
  ├── Firewall
  └── Security Inspection
  │
  ▼
Internet / WAN
```

Typical use cases:

* Internet gateway
* Enterprise edge firewall
* Routing
* NAT
* Inter-VLAN security

---

## Transparent Mode

```text
Network A
    │
    ▼
FortiGate
    │
    ▼
Network B
```

FortiGate operates transparently rather than functioning as the primary Layer-3 gateway.

Typical concept:

```text
Existing L3 Gateway
       │
       ▼
FortiGate
       │
       ▼
Existing Network
```

---

# 13. Administrator Security Hardening Checklist

Use this as a deployment checklist.

## Identity & Authentication

```text
[ ] Remove unnecessary administrator accounts
[ ] Use unique administrator usernames
[ ] Avoid dangerous/special characters in usernames
[ ] Use centralized authentication where appropriate
[ ] Configure LDAP / authentication servers correctly
[ ] Use administrator groups
[ ] Use least-privilege administrator profiles
[ ] Enable MFA for privileged accounts
[ ] Configure NTP
```

---

## Local Accounts

```text
[ ] Minimize local administrator accounts
[ ] Maintain controlled emergency access
[ ] Use strong unique passwords
[ ] Do not reuse passwords
[ ] Do not use predictable password rotation
[ ] Securely store break-glass credentials
```

---

## Trusted Hosts

```text
[ ] Restrict administrator source IPs
[ ] Prefer dedicated management networks
[ ] Use /32 for individual management hosts
[ ] Use management subnets where appropriate
[ ] Review trusted hosts regularly
[ ] Remember that trusted hosts do not restrict PING
```

---

## Remote Authentication

```text
[ ] Configure LDAP/authentication server first
[ ] Configure remote groups
[ ] Map groups to administrator profiles
[ ] Avoid unnecessary super_admin privileges
[ ] Consider PKI/MFA for privileged access
[ ] Define an emergency access strategy
```

---

## API Security

```text
[ ] Create dedicated API users
[ ] Use dedicated API profiles
[ ] Restrict VDOM/global scope
[ ] Use least privilege
[ ] Protect API keys
[ ] Never publish API keys
[ ] Rotate/revoke compromised credentials
```

---

## SSH Security

```text
[ ] Prefer SSH public-key authentication where appropriate
[ ] Protect private keys
[ ] Use secure key storage
[ ] Use passphrases where appropriate
[ ] Avoid embedding passwords in scripts
[ ] Use dedicated automation accounts
[ ] Apply least privilege to automation
```

---

# 14. NSE Exam Quick Review

## ⭐ Administrator Authentication Matrix

| Method         | Authentication Source   | Typical Use                      |
| -------------- | ----------------------- | -------------------------------- |
| Local          | FortiGate               | Emergency / local administration |
| Remote         | LDAP / external server  | Centralized administration       |
| Wildcard       | Remote group            | Multiple remote administrators   |
| MFA            | FortiToken / SMS        | Additional authentication factor |
| SSO            | SAML                    | Centralized SSO                  |
| FortiCloud SSO | FortiCloud              | Cloud-based administrator SSO    |
| API User       | API key                 | Automation / integration         |
| SSH Key        | Public/private key pair | Secure CLI / automation          |

---

## ⭐ Remote Administrator Sequence

```text
1. Configure LDAP/authentication server
             ↓
2. Configure remote user groups
             ↓
3. Create administrator profile
             ↓
4. Create remote administrator
             ↓
5. Map remote group
             ↓
6. Configure VDOM scope
             ↓
7. Configure trusted hosts
             ↓
8. Enable MFA where appropriate
```

---

## ⭐ Wildcard Administrator

```bash
config system admin
    edit "x-u1"
        set remote-auth enable
        set accprofile "super_admin"
        set wildcard enable
        set remote-group "ldap-local-fw"
    next
end
```

### Remember

```text
wildcard = multiple remote users can match
remote-group = controls which remote users/group
accprofile = controls permissions
```

---

## ⭐ Local Login Restriction

```bash
config system global
    set admin-restrict-local enable
end
```

### Remember

```text
Remote servers available
        ↓
Local administrator login restricted

Remote servers unavailable
        ↓
Local administrator can authenticate
```

---

## ⭐ Trusted Hosts

```text
192.168.20.200/32
        │
        └── Single trusted host

192.168.254.0/24
        │
        └── Trusted management subnet
```

### Exam Trap

```text
Trusted Hosts
      ≠
PING restriction
```

---

## ⭐ API Users

```bash
config system api-user
    edit "api-admin"
        set api-key <API-KEY>
        set accprofile "api-prof-test"
    next
end
```

Generate key:

```bash
execute api-user generate-key test-user-api
```

### Remember

```text
API User
   ↓
API Key
   ↓
API Profile
   ↓
VDOM / Global Scope
```

---

## ⭐ Password Policy

```bash
config system password-policy
    set status enable
    set apply-to admin-password
    set minimum-length <8-128>
    set min-lower-case-letter <0-128>
    set min-upper-case-letter <0-128>
    set min-non-alphanumeric <0-128>
    set min-number <0-128>
    set min-change-characters <0-128>
    set expire-status enable
    set expire-day <1-999>
    set reuse-password disable
end
```

### Remember

```text
Length
+
Complexity
+
Uniqueness
+
Expiration
+
No Reuse
```

---

## ⭐ SSH Public Key

```text
Private Key
    │
    │ NEVER send to FortiGate
    ▼
Administrator / Automation System

Public Key
    │
    │ Stored on FortiGate
    ▼
FortiGate Administrator
```

CLI:

```bash
config system admin
    edit "adminx"
        set ssh-public-key1 "<SSH-PUBLIC-KEY>"
    next
end
```

---

# 🔥 SheynShield Golden Rules

> **1. Authentication ≠ Authorization**

```text
Authentication
    ↓
Who are you?

Authorization
    ↓
What can you do?
```

---

> **2. Remote Authentication ≠ Administrator Profile**

```text
LDAP
 ↓
Identity

Admin Profile
 ↓
Permissions
```

Both must be configured correctly.

---

> **3. Wildcard ≠ Super Admin**

```text
Wildcard
   =
Multiple remote users can match

Super Admin
   =
Maximum administrative privilege
```

Combining both can create a **very high-impact privilege boundary**.

---

> **4. Trusted Hosts ≠ Interface Security**

Trusted hosts restrict administrator source addresses, but management exposure should also be controlled through:

```text
Trusted Hosts
+
Interface Administrative Access
+
Dedicated Management Network
+
Network Security Controls
```

---

> **5. MFA ≠ Password Policy**

```text
Password Policy
    ↓
Something you know

FortiToken
    ↓
Something you have
```

Combining them provides stronger authentication.

---

> **6. API Keys Are Credentials**

Treat an API key like a password:

```text
❌ GitHub repository
❌ README
❌ Screenshot
❌ Chat message
❌ Source code
❌ Public documentation

✅ Secret store
✅ Protected automation environment
```

---

> **7. Private SSH Key Never Goes to FortiGate**

```text
FortiGate → Public Key
Client    → Private Key
```

---

> **8. Least Privilege Everywhere**

```text
Human Admin
      ↓
Minimum required profile

API User
      ↓
Minimum required permissions

VDOM Admin
      ↓
Minimum required VDOM

Remote Group
      ↓
Minimum required access
```

---

# Quick Memory Map

```text
                    FORTIGATE ADMINISTRATION
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
     AUTHENTICATION      AUTHORIZATION       SECURITY
          │                  │                  │
    ┌─────┼─────┐            │          ┌───────┼────────┐
    │     │     │            │          │       │        │
  Local LDAP   SSO       Admin Profile MFA  Trusted   Password
    │     │     │            │          │     Hosts    Policy
    │     │     │            │          │
    │  Wildcard             VDOM       FortiToken
    │                        Scope
    │
    └───────────────┐
                    │
              API / SSH
                    │
             ┌──────┴──────┐
             │             │
          REST API      SSH Key
             │             │
          API Key      Public/Private
```

---

## Final Security Architecture

For a mature enterprise FortiGate deployment, a strong administrator design can look like:

```text
                    Management Network
                           │
                           ▼
                      FortiGate
                           │
        ┌──────────────────┼───────────────────┐
        │                  │                   │
        ▼                  ▼                   ▼
     LDAP/AD             SAML             FortiCloud
        │                  │                   │
        └──────────────────┼───────────────────┘
                           │
                           ▼
                    Administrator
                           │
                 ┌─────────┼─────────┐
                 │         │         │
                 ▼         ▼         ▼
             Profile     MFA    Trusted Hosts
                 │
                 ▼
             VDOM Scope
                 │
                 ▼
          Least-Privilege Access
```

### 🛡️ SheynShield Principle

**Secure FortiGate administration is not just about protecting the password.**

It is a combination of:

```text
Strong Authentication
        +
MFA
        +
Centralized Identity
        +
Least Privilege
        +
Trusted Hosts
        +
VDOM Scope
        +
Secure API Design
        +
SSH Key Security
        +
Emergency Access
        +
Accurate Time / NTP
```

---

### SEO Keywords

`FortiGate administrator` · `FortiOS administrator configuration` · `FortiGate admin profile` · `FortiGate LDAP administrator` · `FortiGate remote authentication` · `FortiGate wildcard administrator` · `FortiGate MFA` · `FortiToken administrator` · `FortiGate trusted hosts` · `FortiGate REST API` · `FortiGate API user` · `FortiGate SSO` · `FortiCloud SSO` · `FortiGate password policy` · `FortiGate SSH public key` · `FortiOS 7.2` · `FortiGate security hardening` · `Fortinet NSE4` · `Fortinet NSE7`

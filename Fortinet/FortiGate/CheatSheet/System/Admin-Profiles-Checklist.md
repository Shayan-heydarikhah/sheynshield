# 🔐 FortiGate Administrator & Management Security Checklist

> **FortiOS 7.2.x — Administrator Accounts, Authentication, LDAP, MFA, Trusted Hosts, REST API, SSO, Password Policy & SSH Public-Key Authentication**

**SheynShield | Engineering Secure Networks**

---

## 📋 Table of Contents

* [1. Management Security Baseline](#1-management-security-baseline)
* [2. Administrator Account Checklist](#2-administrator-account-checklist)
* [3. Remote Authentication & LDAP](#3-remote-authentication--ldap)
* [4. Wildcard Administrators](#4-wildcard-administrators)
* [5. Local Administrator Restriction](#5-local-administrator-restriction)
* [6. Administrator Profiles & Least Privilege](#6-administrator-profiles--least-privilege)
* [7. MFA & FortiToken](#7-mfa--fortitoken)
* [8. Trusted Hosts](#8-trusted-hosts)
* [9. VDOM Administrator Scope](#9-vdom-administrator-scope)
* [10. REST API Administrator Security](#10-rest-api-administrator-security)
* [11. SSO Administrator Security](#11-sso-administrator-security)
* [12. FortiCloud SSO](#12-forticloud-sso)
* [13. Administrator Password Policy](#13-administrator-password-policy)
* [14. SSH Public-Key Authentication](#14-ssh-public-key-authentication)
* [15. Management Interface Security](#15-management-interface-security)
* [16. Emergency / Break-Glass Access](#16-emergency--break-glass-access)
* [17. Configuration Validation](#17-configuration-validation)
* [18. Administrator Security Hardening](#18-administrator-security-hardening)
* [19. NSE Exam Quick Review](#19-nse-exam-quick-review)
* [20. SheynShield Golden Rules](#20-sheynshield-golden-rules)
* [21. Final Enterprise Architecture](#21-final-enterprise-architecture)
* [22. SEO Keywords](#22-seo-keywords)

---

# 1. Management Security Baseline

## 1.1 FortiGate Operating Mode

* [ ] Confirm whether the FortiGate operates in **NAT/Route Mode**
* [ ] Confirm whether the FortiGate operates in **Transparent Mode**
* [ ] Document the selected operating mode
* [ ] Verify that the operating mode matches the network architecture

### NAT/Route Mode

```text
Clients
   │
   ▼
FortiGate
   │
   ├── Routing
   ├── NAT
   ├── Firewall Policies
   └── Security Inspection
   │
   ▼
WAN / Internet
```

### Transparent Mode

```text
Network A
   │
   ▼
FortiGate
   │
   ▼
Network B
```

---

## 1.2 Management Security Baseline

* [ ] Use a dedicated management network where possible
* [ ] Restrict administrative access to trusted management sources
* [ ] Minimize exposed management interfaces
* [ ] Disable unnecessary administrative protocols
* [ ] Use HTTPS instead of HTTP for GUI management
* [ ] Use SSH instead of insecure CLI protocols
* [ ] Restrict administrative access by source IP
* [ ] Use MFA for privileged administrators
* [ ] Use centralized authentication where appropriate
* [ ] Apply least-privilege administrator profiles
* [ ] Configure accurate time synchronization
* [ ] Maintain an emergency access strategy

---

# 2. Administrator Account Checklist

## 2.1 Local Administrators

A local administrator is authenticated directly by the FortiGate.

* [ ] Review all local administrator accounts
* [ ] Remove unused accounts
* [ ] Remove accounts belonging to former administrators
* [ ] Use unique administrator usernames
* [ ] Use strong unique passwords
* [ ] Avoid shared administrator accounts
* [ ] Assign the minimum required administrator profile
* [ ] Restrict VDOM scope where applicable
* [ ] Configure trusted hosts
* [ ] Enable MFA for privileged accounts
* [ ] Document emergency accounts separately

Typical configuration areas:

```text
System
└── Administrators
    ├── Username
    ├── Password
    ├── Administrator Profile
    ├── Trusted Hosts
    ├── VDOM
    └── Two-Factor Authentication
```

---

## 2.2 Administrator Username Security

* [ ] Avoid dangerous characters in administrator usernames
* [ ] Do not use HTML/script-like content in usernames
* [ ] Use predictable but controlled naming conventions
* [ ] Do not expose unnecessary personal information in usernames

Avoid:

```text
admin<script>
admin#
admin"name
admin()
```

Prefer:

```text
admin-sec
network-admin
fw-admin
readonly-admin
```

---

## 2.3 Administrator Naming Convention

Recommended enterprise convention:

```text
<role>-<purpose>

Examples:

network-admin
security-admin
fw-readonly
automation-api
breakglass-admin
```

* [ ] Define a naming standard
* [ ] Apply the standard consistently
* [ ] Document account ownership
* [ ] Document account purpose
* [ ] Review naming periodically

---

# 3. Remote Authentication & LDAP

## 3.1 Remote Authentication Prerequisites

Before creating a remote administrator:

* [ ] Configure the remote authentication server
* [ ] Verify connectivity from FortiGate
* [ ] Configure the required remote groups
* [ ] Verify user/group membership
* [ ] Test authentication independently
* [ ] Create the required administrator profile
* [ ] Map the remote group to the correct administrator profile

Recommended sequence:

```text
1. Authentication Server
          ↓
2. Remote User / Group
          ↓
3. Administrator Profile
          ↓
4. Remote Administrator
          ↓
5. Trusted Hosts
          ↓
6. MFA
          ↓
7. Validation
```

---

## 3.2 LDAP Connectivity

* [ ] Verify LDAP server reachability
* [ ] Verify DNS resolution where required
* [ ] Verify LDAP server address
* [ ] Verify authentication credentials
* [ ] Verify LDAP base DN
* [ ] Verify user search configuration
* [ ] Verify group search configuration
* [ ] Verify LDAP group membership
* [ ] Test authentication using a dedicated test account

Architecture:

```text
LDAP / AD
   │
   ├── Users
   ├── Groups
   └── Authentication
          │
          ▼
       FortiGate
          │
          ▼
   Remote Administrator
          │
          ▼
   Administrator Profile
```

---

## 3.3 Remote Administrator Validation

* [ ] Confirm the remote username exists
* [ ] Confirm the remote group is correct
* [ ] Confirm the administrator profile is correct
* [ ] Confirm VDOM scope
* [ ] Confirm trusted hosts
* [ ] Confirm MFA requirements
* [ ] Test successful login
* [ ] Test unauthorized user rejection
* [ ] Test unauthorized group rejection
* [ ] Test privilege boundaries

---

# 4. Wildcard Administrators

A wildcard administrator allows multiple remote users to match a single administrator configuration.

## 4.1 Wildcard Checklist

* [ ] Confirm wildcard administration is actually required
* [ ] Configure remote authentication
* [ ] Configure the remote group
* [ ] Enable wildcard matching
* [ ] Assign an appropriate administrator profile
* [ ] Restrict VDOM scope
* [ ] Configure trusted hosts
* [ ] Enable MFA where appropriate
* [ ] Test authorized group members
* [ ] Test unauthorized users
* [ ] Review the group membership regularly

Example:

```bash
config system admin
    edit "remote-network-admin"
        set remote-auth enable
        set wildcard enable
        set remote-group "ldap-local-fw"
        set accprofile "network-admin"
    next
end
```

### Key Parameters

```text
remote-auth
    ↓
Remote authentication enabled

wildcard
    ↓
Multiple remote users can match

remote-group
    ↓
Controls which remote group is allowed

accprofile
    ↓
Controls administrative permissions
```

---

## 4.2 Wildcard Security Review

* [ ] Do not automatically assign `super_admin`
* [ ] Use dedicated LDAP groups
* [ ] Separate network and security administrators
* [ ] Separate read-only users
* [ ] Review group membership
* [ ] Review administrator privileges
* [ ] Document why wildcard access is required

Recommended:

```text
LDAP Group
   │
   ├── Network Admins
   │       └── Network Profile
   │
   ├── Security Admins
   │       └── Security Profile
   │
   └── Read Only
           └── Read-Only Profile
```

---

# 5. Local Administrator Restriction

FortiOS provides `admin-restrict-local` to restrict local administrator authentication while configured remote authentication servers are available.

## 5.1 Configuration Checklist

* [ ] Determine whether local-login restriction is required
* [ ] Identify all remote authentication dependencies
* [ ] Configure emergency access appropriately
* [ ] Enable the feature if required

Example:

```bash
config system global
    set admin-restrict-local enable
end
```

---

## 5.2 Behavior Validation

Expected concept:

```text
Remote Authentication Server
          │
          ▼
       Available?
       /       \
     YES        NO
      │          │
      ▼          ▼
Local Login    Local Login
Restricted     Allowed
```

* [ ] Test remote authentication available → local login restricted
* [ ] Test remote authentication unavailable → emergency local access works
* [ ] Document break-glass procedure
* [ ] Ensure emergency credentials are securely stored

> ⚠️ **Operational Warning:** Never implement centralized authentication in a way that can permanently lock administrators out during an identity-service outage.

---

# 6. Administrator Profiles & Least Privilege

Administrator profiles answer:

> **What is this administrator allowed to do?**

Authentication answers:

> **Who is this user?**

Authorization answers:

> **What can this user do?**

---

## 6.1 Administrator Profile Checklist

* [ ] Review all administrator profiles
* [ ] Remove unnecessary custom profiles
* [ ] Identify profiles with excessive permissions
* [ ] Separate read-only access
* [ ] Separate network administration
* [ ] Separate security administration
* [ ] Restrict system administration
* [ ] Restrict VDOM scope
* [ ] Apply least privilege

Example:

```text
Authentication
      │
      ▼
Administrator
      │
      ▼
Administrator Profile
      │
      ▼
Permissions
      │
      ▼
VDOM Scope
```

---

## 6.2 Privilege Model

| Administrator  | Recommended Scope                        |
| -------------- | ---------------------------------------- |
| Read Only      | Monitoring / viewing                     |
| Helpdesk       | Minimum operational permissions          |
| Network Admin  | Interfaces / routing / network functions |
| Security Admin | Firewall / security configuration        |
| Automation     | Dedicated API permissions                |
| Super Admin    | Exceptional full administrative access   |

* [ ] Avoid giving `super_admin` to ordinary users
* [ ] Avoid giving `super_admin` to automation unless absolutely required
* [ ] Review privileged accounts regularly

---

# 7. MFA & FortiToken

## 7.1 MFA Checklist

* [ ] Enable MFA for privileged administrators
* [ ] Prefer strong token-based authentication where appropriate
* [ ] Assign the correct token to the correct administrator
* [ ] Verify token status
* [ ] Test token authentication
* [ ] Document token ownership
* [ ] Define recovery procedures
* [ ] Protect token enrollment information

Common methods:

```text
FortiToken
SMS
```

---

## 7.2 FortiToken Authentication

Conceptual flow:

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

Example structure:

```bash
config system admin
    edit "admin-sec"
        set two-factor fortitoken
        set fortitoken "<TOKEN>"
    next
end
```

> ⚠️ Never commit real passwords, token values, secrets, or credentials to Git repositories.

---

## 7.3 NTP & MFA

Accurate time is critical for time-based authentication.

* [ ] Configure NTP
* [ ] Verify NTP synchronization
* [ ] Verify system time
* [ ] Verify timezone
* [ ] Verify token status
* [ ] Test MFA

Architecture:

```text
FortiGate
   │
   ▼
NTP
   │
   ▼
Accurate Time
   │
   ▼
Token Validation
```

---

## 7.4 FortiToken Cloud

* [ ] Determine whether FortiToken Cloud is required
* [ ] Verify cloud service integration
* [ ] Verify administrator identity mapping
* [ ] Verify MFA enrollment
* [ ] Test authentication
* [ ] Document recovery procedures

Concept:

```text
Administrator
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

# 8. Trusted Hosts

Trusted Hosts restrict administrator access based on the source IP address.

## 8.1 Trusted Host Checklist

* [ ] Configure trusted hosts for privileged administrators
* [ ] Use `/32` for individual management hosts
* [ ] Use subnet definitions where appropriate
* [ ] Prefer dedicated management networks
* [ ] Avoid broad source ranges
* [ ] Review trusted hosts periodically
* [ ] Remove obsolete management addresses
* [ ] Validate access from allowed sources
* [ ] Validate rejection from unauthorized sources

Examples:

```text
192.168.20.200/32
    ↓
Single management host

192.168.254.0/24
    ↓
Management subnet
```

---

## 8.2 Trusted Host Validation

```text
Administrator
      │
      ▼
Source IP
      │
      ▼
Trusted Host Check
      │
 ┌────┴────┐
 │         │
MATCH    NO MATCH
 │         │
 ▼         ▼
ALLOW     DENY
```

* [ ] Test an allowed source
* [ ] Test an unauthorized source
* [ ] Verify every administrator has the intended restriction

---

## 8.3 Trusted Host ≠ Interface Security

Remember:

```text
Trusted Hosts
      ≠
Complete Management Interface Security
```

A secure management design should combine:

```text
Trusted Hosts
      +
Interface Administrative Access
      +
Dedicated Management Network
      +
Network Security Controls
```

### ICMP/PING Trap

* [ ] Review whether PING is enabled
* [ ] Do not assume trusted hosts block ICMP
* [ ] Restrict unnecessary interface administrative services

> ⚠️ **Exam Trap:** Trusted-host restrictions are not equivalent to ICMP/PING restrictions.

---

# 9. VDOM Administrator Scope

When VDOMs are enabled, administrator privileges can be scoped to specific virtual domains.

## 9.1 VDOM Checklist

* [ ] Identify administrators requiring global access
* [ ] Identify administrators requiring VDOM-only access
* [ ] Assign the minimum required VDOM
* [ ] Avoid unnecessary global privileges
* [ ] Validate cross-VDOM access
* [ ] Review VDOM permissions periodically

Architecture:

```text
FortiGate
│
├── root
│   ├── Global Admin
│   └── Network Admin
│
├── VDOM-A
│   └── VDOM-A Admin
│
└── VDOM-B
    └── VDOM-B Admin
```

---

# 10. REST API Administrator Security

FortiGate REST API access should be treated as privileged administrative access.

## 10.1 API Security Checklist

* [ ] Identify all API users
* [ ] Remove unused API users
* [ ] Create dedicated API identities
* [ ] Assign dedicated API profiles
* [ ] Restrict API permissions
* [ ] Restrict VDOM/global scope
* [ ] Protect API keys
* [ ] Rotate/revoke compromised API keys
* [ ] Avoid hard-coding secrets in source code
* [ ] Never publish API keys
* [ ] Review API users periodically

---

## 10.2 API Architecture

```text
Automation System
       │
       ▼
FortiGate REST API
       │
       ▼
API User
       │
       ▼
API Key
       │
       ▼
API Profile
       │
       ▼
VDOM / Global Scope
```

---

## 10.3 API Least Privilege

Preferred:

```text
Automation
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
Required Permissions
```

Avoid:

```text
Automation
     │
     ▼
super_admin
```

unless there is a documented operational requirement.

---

## 10.4 API Key Protection

Treat API keys as credentials.

Never place them in:

```text
❌ GitHub
❌ README files
❌ Public documentation
❌ Screenshots
❌ Tickets
❌ Chat messages
❌ Source code
❌ Tutorials
```

Prefer:

```text
✅ Secret manager
✅ Protected CI/CD variables
✅ Secure automation environment
✅ Restricted configuration store
```

---

## 10.5 Example API Configuration

```bash
config system api-user
    edit "api-admin"
        set api-key "<API-KEY>"
        set accprofile "api-profile"
    next
end
```

Generate a key when required:

```bash
execute api-user generate-key <api-user>
```

* [ ] Never publish the generated key
* [ ] Store it securely
* [ ] Revoke it if exposed

---

# 11. SSO Administrator Security

FortiGate can support SAML-based administrator Single Sign-On.

## 11.1 SSO Checklist

* [ ] Identify the Identity Provider
* [ ] Configure SAML correctly
* [ ] Verify FortiGate Service Provider configuration
* [ ] Configure the required SAML attributes
* [ ] Define the appropriate administrator profile
* [ ] Define VDOM scope
* [ ] Configure trusted hosts where applicable
* [ ] Test successful authentication
* [ ] Test unauthorized identity rejection
* [ ] Document SSO dependencies

Architecture:

```text
Administrator
      │
      ▼
FortiGate
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
SSO Administrator
```

---

## 11.2 SSO Administrator Profile

Review:

* [ ] Default administrator profile
* [ ] SAML Service Provider settings
* [ ] User mapping
* [ ] VDOM scope
* [ ] Authorization attributes
* [ ] Privilege level

Concept:

```text
SAML Authentication
       │
       ▼
Service Provider Settings
       │
       ▼
Administrator Profile
       │
       ▼
SSO Administrator
```

---

# 12. FortiCloud SSO

## 12.1 FortiCloud SSO Checklist

* [ ] Determine whether FortiCloud SSO is required
* [ ] Verify FortiGate registration
* [ ] Verify FortiCloud identity
* [ ] Verify IAM user permissions where applicable
* [ ] Verify required portal permissions
* [ ] Test SSO authentication
* [ ] Document dependency on FortiCloud services

Example:

```bash
config system global
    set admin-forticloud-sso-login enable
end
```

Conceptual flow:

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
Administrator Access
```

---

# 13. Administrator Password Policy

## 13.1 Password Policy Checklist

* [ ] Enable administrator password policy
* [ ] Define minimum password length
* [ ] Require lowercase characters
* [ ] Require uppercase characters
* [ ] Require numbers
* [ ] Require non-alphanumeric characters
* [ ] Define password expiration if required
* [ ] Prevent password reuse where appropriate
* [ ] Require sufficient password change characters
* [ ] Review password policy periodically

---

## 13.2 Password Policy Parameters

| Parameter         | Security Purpose              |
| ----------------- | ----------------------------- |
| Minimum Length    | Prevent short passwords       |
| Lowercase         | Increase complexity           |
| Uppercase         | Increase complexity           |
| Number            | Increase complexity           |
| Non-Alphanumeric  | Increase complexity           |
| Expiration        | Password lifecycle control    |
| Change Characters | Prevent trivial modifications |
| Password Reuse    | Prevent reuse                 |

---

## 13.3 Password Policy Example

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

> **Note:** Validate exact CLI ranges and supported parameters against the FortiOS 7.2.x build being deployed.

---

## 13.4 Password Security

Avoid predictable passwords:

```text
Password1
Password2
Password3
```

Prefer:

```text
Long
+
Unique
+
Unpredictable
+
Not Reused
```

* [ ] Use a password manager where appropriate
* [ ] Never reuse privileged credentials
* [ ] Never share administrator passwords
* [ ] Never store credentials in Git

---

# 14. SSH Public-Key Authentication

SSH public-key authentication can reduce reliance on password-based SSH authentication.

## 14.1 SSH Key Checklist

* [ ] Generate a dedicated SSH key pair
* [ ] Protect the private key
* [ ] Use a passphrase where appropriate
* [ ] Store the private key securely
* [ ] Install only the public key on FortiGate
* [ ] Never transfer the private key to FortiGate
* [ ] Use dedicated accounts for automation
* [ ] Apply least privilege
* [ ] Remove obsolete public keys

---

## 14.2 SSH Key Architecture

```text
SSH Key Pair
│
├── Public Key
│      │
│      └── FortiGate
│
└── Private Key
       │
       └── Administrator / Automation System
```

Authentication:

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

## 14.3 Configure Public Key

Example:

```bash
config system admin
    edit "admin-sec"
        set ssh-public-key1 "<SSH-PUBLIC-KEY>"
    next
end
```

* [ ] Verify public key is correct
* [ ] Verify private key remains on the client
* [ ] Test SSH authentication
* [ ] Test invalid key rejection
* [ ] Remove obsolete keys

---

## 14.4 SSH Keys for Automation

Recommended:

```text
Automation
    │
    ▼
Dedicated SSH Account
    │
    ▼
Private Key
    │
    ▼
FortiGate
    │
    ▼
Public Key
```

* [ ] Avoid embedding passwords in automation scripts
* [ ] Protect automation private keys
* [ ] Use dedicated automation identities
* [ ] Restrict permissions
* [ ] Monitor automation activity

---

# 15. Management Interface Security

## 15.1 Interface Administrative Access

For every management interface:

* [ ] Identify enabled administrative protocols
* [ ] Disable unnecessary services
* [ ] Restrict HTTPS access
* [ ] Restrict SSH access
* [ ] Review PING exposure
* [ ] Review HTTP exposure
* [ ] Review TELNET exposure
* [ ] Review SNMP exposure
* [ ] Restrict access to management VLANs

Recommended concept:

```text
Dedicated Management Network
             │
             ▼
        Management VLAN
             │
             ▼
          FortiGate
             │
      ┌──────┴──────┐
      │             │
     HTTPS          SSH
      │             │
      └──────┬──────┘
             ▼
        Admin Access
```

---

# 16. Emergency / Break-Glass Access

A mature FortiGate deployment should have a controlled emergency-access strategy.

## 16.1 Break-Glass Checklist

* [ ] Maintain a dedicated emergency administrator
* [ ] Do not use the account for daily administration
* [ ] Use a strong unique credential
* [ ] Store credentials securely
* [ ] Restrict access as much as operationally possible
* [ ] Document emergency procedures
* [ ] Test emergency access periodically
* [ ] Log emergency account usage
* [ ] Rotate credentials according to policy
* [ ] Review the account after every use

Concept:

```text
Normal Administration
       │
       ├── LDAP / SSO
       ├── MFA
       ├── Trusted Hosts
       └── Least Privilege

Emergency Administration
       │
       ├── Local Account
       ├── Strong Credential
       ├── Restricted Access
       └── Secure Storage
```

---

# 17. Configuration Validation

## 17.1 Identity Validation

* [ ] Verify administrator accounts
* [ ] Verify remote authentication
* [ ] Verify LDAP connectivity
* [ ] Verify remote groups
* [ ] Verify wildcard configuration
* [ ] Verify SSO
* [ ] Verify FortiCloud SSO
* [ ] Verify MFA

---

## 17.2 Authorization Validation

* [ ] Verify administrator profiles
* [ ] Verify least privilege
* [ ] Verify VDOM scope
* [ ] Verify API profile permissions
* [ ] Verify automation permissions
* [ ] Verify super-admin assignments

---

## 17.3 Network Access Validation

* [ ] Verify trusted hosts
* [ ] Verify management VLAN
* [ ] Verify administrative services
* [ ] Verify HTTPS
* [ ] Verify SSH
* [ ] Verify PING exposure
* [ ] Verify unauthorized-source rejection

---

## 17.4 Credential Validation

* [ ] Verify password policy
* [ ] Verify MFA
* [ ] Verify API key protection
* [ ] Verify SSH private-key protection
* [ ] Verify break-glass credentials
* [ ] Search repositories for accidentally exposed secrets

---

# 18. Administrator Security Hardening

## 🔐 Identity

* [ ] Remove unused administrators
* [ ] Use unique usernames
* [ ] Use centralized authentication where appropriate
* [ ] Use dedicated administrator groups
* [ ] Enable MFA
* [ ] Configure NTP

---

## 🔑 Authentication

* [ ] Use LDAP/centralized authentication where appropriate
* [ ] Use SAML SSO where appropriate
* [ ] Use FortiCloud SSO where appropriate
* [ ] Use SSH public-key authentication where appropriate
* [ ] Maintain emergency authentication

---

## 🛡️ Authorization

* [ ] Apply least privilege
* [ ] Avoid unnecessary `super_admin`
* [ ] Use dedicated API profiles
* [ ] Restrict VDOM scope
* [ ] Separate network and security administration
* [ ] Review privileged accounts regularly

---

## 🌐 Management Exposure

* [ ] Use a dedicated management network
* [ ] Configure trusted hosts
* [ ] Disable unnecessary administrative protocols
* [ ] Restrict HTTPS
* [ ] Restrict SSH
* [ ] Review PING exposure
* [ ] Minimize Internet-facing management

---

## 🤖 API Security

* [ ] Use dedicated API users
* [ ] Use dedicated API profiles
* [ ] Restrict API scope
* [ ] Protect API keys
* [ ] Rotate/revoke exposed credentials
* [ ] Never publish secrets

---

## 🔐 SSH Security

* [ ] Use public-key authentication where appropriate
* [ ] Protect private keys
* [ ] Use passphrases where appropriate
* [ ] Remove obsolete keys
* [ ] Use dedicated automation identities
* [ ] Apply least privilege

---

# 19. NSE Exam Quick Review

## ⭐ Authentication vs Authorization

```text
AUTHENTICATION
      ↓
Who are you?

AUTHORIZATION
      ↓
What can you do?
```

---

## ⭐ Administrator Authentication Matrix

| Method         | Authentication Source  | Typical Use                      |
| -------------- | ---------------------- | -------------------------------- |
| Local          | FortiGate              | Local / emergency administration |
| Remote         | LDAP / external server | Centralized administration       |
| Wildcard       | Remote group           | Multiple remote administrators   |
| MFA            | FortiToken / SMS       | Additional authentication factor |
| SSO            | SAML                   | Centralized SSO                  |
| FortiCloud SSO | FortiCloud             | Cloud-based administrator SSO    |
| API User       | API key                | Automation / integration         |
| SSH Key        | Public/private key     | CLI / automation                 |

---

## ⭐ Remote Administrator Sequence

```text
Authentication Server
        ↓
Remote Group
        ↓
Administrator Profile
        ↓
Remote Administrator
        ↓
VDOM Scope
        ↓
Trusted Hosts
        ↓
MFA
        ↓
Validation
```

---

## ⭐ Wildcard Administrator

```text
wildcard
    ↓
Multiple remote users can match

remote-group
    ↓
Defines allowed remote group

accprofile
    ↓
Defines permissions
```

### Exam Trap

```text
Wildcard
    ≠
Super Admin
```

A wildcard administrator can match many users, but the assigned administrator profile determines what those users can do.

---

## ⭐ Local Administrator Restriction

```bash
config system global
    set admin-restrict-local enable
end
```

Remember:

```text
Remote Authentication Available
        ↓
Local Login Restricted

Remote Authentication Unavailable
        ↓
Local Login Can Authenticate
```

---

## ⭐ Trusted Hosts

```text
192.168.20.200/32
        ↓
Single Host

192.168.254.0/24
        ↓
Management Subnet
```

### Exam Trap

```text
Trusted Hosts
      ≠
PING Restriction
```

---

## ⭐ API Security

```text
API User
   ↓
API Key
   ↓
API Profile
   ↓
VDOM / Global Scope
   ↓
Required Permissions
```

### Golden Rule

```text
API Key = Credential
```

Protect it like a password.

---

## ⭐ Password Policy

```text
Length
   +
Complexity
   +
Uniqueness
   +
No Reuse
   +
Appropriate Lifecycle
```

---

## ⭐ SSH Public Key

```text
FortiGate
    ↓
Public Key

Client
    ↓
Private Key
```

### Exam Trap

```text
Private Key
      ≠
Stored on FortiGate
```

Only the public key should be configured on the FortiGate administrator account.

---

# 20. SheynShield Golden Rules

> ### 🔥 Rule #1 — Authentication ≠ Authorization

```text
Authentication
    ↓
Who are you?

Authorization
    ↓
What can you do?
```

---

> ### 🔥 Rule #2 — LDAP ≠ Administrator Profile

```text
LDAP
 ↓
Identity

Administrator Profile
 ↓
Permissions
```

Both layers must be correctly configured.

---

> ### 🔥 Rule #3 — Wildcard ≠ Super Admin

```text
Wildcard
    =
Multiple remote users

super_admin
    =
Maximum administrative privilege
```

Combining a broad remote group with `super_admin` can create a very large privilege boundary.

---

> ### 🔥 Rule #4 — Trusted Hosts ≠ Complete Management Security

```text
Trusted Hosts
      +
Interface Administrative Access
      +
Management Network
      +
Network Security Controls
```

---

> ### 🔥 Rule #5 — MFA ≠ Password Policy

```text
Password
    ↓
Something you know

FortiToken
    ↓
Something you have
```

---

> ### 🔥 Rule #6 — API Keys Are Credentials

```text
API Key
    =
Credential
```

Never publish it.

---

> ### 🔥 Rule #7 — Private SSH Key Never Goes to FortiGate

```text
FortiGate → Public Key
Client    → Private Key
```

---

> ### 🔥 Rule #8 — Least Privilege Everywhere

```text
Human Admin
     ↓
Minimum Required Profile

API User
     ↓
Minimum Required Permissions

VDOM Admin
     ↓
Minimum Required VDOM

Remote Group
     ↓
Minimum Required Access
```

---

> ### 🔥 Rule #9 — NTP Matters for MFA

```text
Accurate Time
      ↓
Reliable Token Validation
```

---

> ### 🔥 Rule #10 — Emergency Access Must Be Designed

```text
Centralized Authentication
          ↓
Identity Infrastructure Failure
          ↓
Controlled Break-Glass Access
          ↓
Emergency Recovery
```

---

# 21. Final Enterprise Architecture

A mature FortiGate administrator security architecture can be represented as:

```text
                     MANAGEMENT NETWORK
                             │
                             ▼
                        FORTIGATE
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ▼                    ▼                    ▼
     LDAP / AD             SAML              FortiCloud
        │                    │                    │
        └────────────────────┼────────────────────┘
                             │
                             ▼
                      ADMINISTRATOR
                             │
             ┌───────────────┼───────────────┐
             │               │               │
             ▼               ▼               ▼
        Admin Profile       MFA        Trusted Hosts
             │               │               │
             └───────────────┼───────────────┘
                             │
                             ▼
                         VDOM Scope
                             │
                             ▼
                      Least Privilege
                             │
             ┌───────────────┴───────────────┐
             │                               │
             ▼                               ▼
          REST API                         SSH
             │                               │
          API Key                        Public Key
             │                               │
             ▼                               ▼
       Automation                       CLI / Automation
```

---

# 22. SEO Keywords

`FortiGate administrator` · `FortiOS administrator configuration` · `FortiGate administrator security` · `FortiGate admin profile` · `FortiGate LDAP administrator` · `FortiGate remote authentication` · `FortiGate wildcard administrator` · `FortiGate MFA` · `FortiToken administrator` · `FortiGate trusted hosts` · `FortiGate REST API` · `FortiGate API user` · `FortiGate API security` · `FortiGate SSO` · `FortiGate SAML administrator` · `FortiCloud SSO` · `FortiGate password policy` · `FortiGate SSH public key` · `FortiOS 7.2` · `FortiGate management security` · `FortiGate security hardening` · `Fortinet administrator security` · `Fortinet NSE4` · `Fortinet NSE7`

---

# ✅ Final Deployment Checklist

Use this section as the final sign-off before putting a FortiGate into production.

## Identity

* [ ] All administrator accounts reviewed
* [ ] Unused accounts removed
* [ ] Administrator naming convention applied
* [ ] Centralized authentication configured where appropriate
* [ ] Remote groups reviewed

## Authentication

* [ ] Strong passwords configured
* [ ] MFA enabled for privileged administrators
* [ ] NTP synchronized
* [ ] LDAP authentication tested
* [ ] SSO authentication tested
* [ ] Emergency authentication tested

## Authorization

* [ ] Administrator profiles reviewed
* [ ] Least privilege applied
* [ ] `super_admin` assignments justified
* [ ] VDOM scope reviewed
* [ ] API permissions reviewed

## Network Security

* [ ] Dedicated management network used where possible
* [ ] Trusted hosts configured
* [ ] Administrative services minimized
* [ ] HTTPS restricted
* [ ] SSH restricted
* [ ] PING exposure reviewed
* [ ] Unauthorized management access tested

## API

* [ ] API users documented
* [ ] API profiles restricted
* [ ] API keys protected
* [ ] No API keys committed to Git
* [ ] Compromised keys can be revoked

## SSH

* [ ] Public-key authentication configured where appropriate
* [ ] Private keys securely stored
* [ ] Passphrases used where appropriate
* [ ] Obsolete keys removed
* [ ] Automation accounts use least privilege

## Emergency Access

* [ ] Break-glass account exists
* [ ] Break-glass credentials securely stored
* [ ] Emergency procedure documented
* [ ] Emergency access tested
* [ ] Emergency account usage monitored

## Final Review

* [ ] Configuration reviewed by another administrator
* [ ] Authentication failure scenarios tested
* [ ] Privilege boundaries tested
* [ ] Management exposure reviewed
* [ ] Credentials and secrets checked before publishing configuration/documentation
* [ ] Configuration backup securely stored

---

## 🛡️ SheynShield Security Principle

> **Secure FortiGate administration is not about protecting a password alone.**

A mature administrator-security architecture combines:

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
        +
Secure Management Network
```

**Authentication identifies the administrator.
Authorization limits the administrator.
Network controls restrict where the administrator can connect from.
MFA strengthens identity assurance.
Least privilege limits the blast radius.**

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


> **SheynShield | Engineering Secure Networks**
>
> *Fortinet • Network Security • Firewall Engineering • Secure Network Design*

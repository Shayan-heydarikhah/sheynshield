# 🔐 FortiGate PKI & Certificate-Based Authentication Checklist

> **SheynShield | Engineering Secure Networks**
>
> Practical **FortiGate PKI & Certificate Authentication Checklist** covering **X.509 certificates, CA trust, client certificate validation, User Peer, CN/Subject matching, SAN/UPN, LDAP integration, Active Directory, certificate-based firewall authentication, and troubleshooting**.
>
> **FortiOS Focus:** PKI Authentication • Certificate Validation • LDAP • Active Directory • UPN • SAN • User Peer • Firewall Policy

---

## 📌 Table of Contents

* [1. PKI Authentication Overview](#1-pki-authentication-overview)
* [2. PKI Components](#2-pki-components)
* [3. PKI Prerequisites](#3-pki-prerequisites)
* [4. Certificate Authentication Flow](#4-certificate-authentication-flow)
* [5. Certificate Authority Trust](#5-certificate-authority-trust)
* [6. FortiGate User Peer](#6-fortigate-user-peer)
* [7. Mandatory CA Verification](#7-mandatory-ca-verification)
* [8. Certificate Subject and DN](#8-certificate-subject-and-dn)
* [9. CN-Based Authentication](#9-cn-based-authentication)
* [10. Certificate Matching](#10-certificate-matching)
* [11. Subject Matching](#11-subject-matching)
* [12. CN Matching](#12-cn-matching)
* [13. Multiple CN Values](#13-multiple-cn-values)
* [14. LDAP-Integrated PKI Authentication](#14-ldap-integrated-pki-authentication)
* [15. UPN / Principal Name](#15-upn--principal-name)
* [16. UPN in SAN](#16-upn-in-san)
* [17. LDAP Password vs UPN Authentication](#17-ldap-password-vs-upn-authentication)
* [18. LDAP Username Mapping](#18-ldap-username-mapping)
* [19. PKI Authentication in Firewall Policy](#19-pki-authentication-in-firewall-policy)
* [20. Authentication vs Authorization](#20-authentication-vs-authorization)
* [21. PKI Security Checklist](#21-pki-security-checklist)
* [22. PKI Troubleshooting Checklist](#22-pki-troubleshooting-checklist)
* [23. Troubleshooting Matrix](#23-troubleshooting-matrix)
* [24. PKI Deployment Models](#24-pki-deployment-models)
* [25. Enterprise PKI Architecture](#25-enterprise-pki-architecture)
* [26. NSE Exam Memory Map](#26-nse-exam-memory-map)
* [27. Quick CLI Reference](#27-quick-cli-reference)
* [28. One-Minute PKI Checklist](#28-one-minute-pki-checklist)
* [29. Golden Rules](#29-golden-rules)
* [30. Final PKI Mental Model](#30-final-pki-mental-model)
* [31. SheynShield Resources](#31-sheynshield-resources)

---

# 1. PKI Authentication Overview

**PKI (Public Key Infrastructure)** allows FortiGate to authenticate users or clients using **digital certificates**.

Instead of relying only on:

```text
Username + Password
```

the authentication process can use:

```text
Client Certificate
        ↓
Certificate Validation
        ↓
Identity Extraction
        ↓
LDAP / AD Lookup
        ↓
User / Group Mapping
        ↓
Authorization
```

### PKI Deployment Checklist

* [ ] Enterprise CA exists
* [ ] Client certificates are issued
* [ ] FortiGate trusts the appropriate CA
* [ ] Certificate identity fields are correctly populated
* [ ] LDAP/AD integration is available where required
* [ ] User/group mapping is configured
* [ ] Certificate authentication is enabled in the required policy
* [ ] Authorization is restricted using appropriate policies

### Enterprise Architecture

```text
                    ┌────────────────────┐
                    │   Active Directory │
                    │        LDAP        │
                    └─────────┬──────────┘
                              │
                              │ Identity
                              ▼
                    ┌────────────────────┐
                    │    Enterprise CA   │
                    │  Root / Issuing CA │
                    └─────────┬──────────┘
                              │
                       Issue Certificate
                              │
                              ▼
                    ┌────────────────────┐
                    │   Client / User    │
                    │ X.509 Certificate  │
                    └─────────┬──────────┘
                              │
                         TLS / PKI
                              │
                              ▼
                    ┌────────────────────┐
                    │     FortiGate      │
                    │ PKI Authentication │
                    └─────────┬──────────┘
                              │
                              ▼
                       LDAP / Identity
                              │
                              ▼
                        User / Group
                              │
                              ▼
                       Security Policy
                              │
                              ▼
                            Access
```

---

# 2. PKI Components

| Component              | Role                                          |
| ---------------------- | --------------------------------------------- |
| **CA**                 | Issues and signs certificates                 |
| **Root CA**            | Establishes the trust anchor                  |
| **Intermediate CA**    | Issues certificates under the Root CA         |
| **Client Certificate** | Represents the user/device identity           |
| **Private Key**        | Proves possession of the certificate identity |
| **X.509 Certificate**  | Contains identity and trust information       |
| **SAN**                | Contains additional certificate identities    |
| **UPN**                | Common AD identity carried in SAN             |
| **FortiGate**          | Validates and processes the certificate       |
| **LDAP / AD**          | Provides centralized user identity            |
| **User Peer**          | Defines certificate-based peer matching       |
| **Firewall Policy**    | Provides authorization                        |

### Component Checklist

* [ ] Root CA is trusted
* [ ] Intermediate CA is correctly deployed
* [ ] Client certificate is valid
* [ ] Client private key is available
* [ ] Certificate identity is correct
* [ ] SAN/UPN is populated where required
* [ ] LDAP identity exists
* [ ] FortiGate has the required CA certificate

---

# 3. PKI Prerequisites

Before deploying certificate-based authentication:

### Certificate Infrastructure

* [ ] Enterprise CA is operational
* [ ] CA certificate is available
* [ ] Certificate chain is complete
* [ ] Client certificates are issued
* [ ] Certificates are not expired
* [ ] Certificates contain the required identity fields
* [ ] Certificate identities are unique

### Active Directory / LDAP

* [ ] AD/LDAP is reachable
* [ ] LDAP server is configured
* [ ] Base DN is correct
* [ ] Required user attributes exist
* [ ] User/group mapping is defined
* [ ] UPN values are consistent where used

### FortiGate

* [ ] CA certificate is imported
* [ ] User Peer is configured
* [ ] Certificate validation is enabled
* [ ] Matching method is selected
* [ ] LDAP integration is configured where required
* [ ] Firewall policy references the correct authentication mechanism

### Security

* [ ] Certificate private keys are protected
* [ ] LDAP credentials are not exposed
* [ ] Production certificates are not committed to GitHub
* [ ] Real usernames/passwords are removed from examples
* [ ] Certificate revocation strategy is defined

> **Security Rule:** Never publish production certificates, private keys, LDAP passwords, activation secrets, or other authentication material in documentation.

---

# 4. Certificate Authentication Flow

A typical FortiGate certificate authentication workflow:

```text
Client
  │
  │ Client Certificate
  ▼
FortiGate
  │
  ├── Verify Certificate
  │
  ├── Validate CA
  │
  ├── Inspect Subject / CN / SAN
  │
  ├── Extract Identity
  │
  └── LDAP Lookup
          │
          ▼
      Active Directory
          │
          ▼
      User Validation
          │
          ▼
      Group Membership
          │
          ▼
      Firewall Policy
          │
          ▼
         ACCESS
```

### Authentication Flow Checklist

* [ ] Client presents certificate
* [ ] FortiGate validates certificate
* [ ] Trusted CA is confirmed
* [ ] Certificate identity is extracted
* [ ] LDAP lookup succeeds if configured
* [ ] User identity matches
* [ ] User/group authorization succeeds
* [ ] Firewall policy permits access

---

# 5. Certificate Authority Trust

The first major security decision is **certificate trust**.

Typical PKI hierarchy:

```text
Root CA
   │
   └── Intermediate CA
          │
          └── User Certificate
```

FortiGate must trust the appropriate CA chain required to validate the client certificate.

### CA Checklist

* [ ] Correct CA certificate is imported
* [ ] CA is trusted by FortiGate
* [ ] Certificate chain is valid
* [ ] Intermediate CA is available where required
* [ ] CA certificate has not expired
* [ ] Client certificate was issued by a trusted CA
* [ ] Certificate validation succeeds

### Core Rule

```text
Client Certificate
        ↓
CA Validation
        ↓
Identity Matching
```

Do not treat identity matching as a replacement for certificate trust.

---

# 6. FortiGate User Peer

A **User Peer** defines certificate-based user/peer matching behavior.

Example:

```bash
config user peer
    edit "u1"
        set mandatory-ca-verify enable
        set ca "ca-test.com"
        set cn-type string
        set cn "u1"
        set ldap-server "winsrv-2016"
        set ldap-mode password
    next
end
```

### User Peer Checklist

* [ ] User Peer exists
* [ ] Correct CA is selected
* [ ] CA verification is enabled
* [ ] CN type is appropriate
* [ ] CN value is correct when CN matching is used
* [ ] LDAP server is correct
* [ ] LDAP mode matches the authentication design
* [ ] User identity matches the expected directory identity

### Identity Example

```text
Active Directory
    │
    └── User: u1

Certificate
    │
    └── CN = u1
```

Expected relationship:

```text
Certificate Identity
        =
Directory Identity
```

---

# 7. Mandatory CA Verification

Enable mandatory CA verification:

```bash
set mandatory-ca-verify enable
```

Conceptually:

```text
Client Certificate
        │
        ▼
Trusted CA?
   │         │
  YES        NO
   │         │
   ▼         ▼
Continue    Reject
```

### Security Checklist

* [ ] `mandatory-ca-verify` is enabled where required
* [ ] Correct CA is configured
* [ ] Certificate chain is trusted
* [ ] Expired certificates are rejected
* [ ] Untrusted certificates are rejected

> **Production Principle:** Certificate identity matching should occur within a trusted certificate-validation process.

---

# 8. Certificate Subject and DN

A certificate Subject can contain multiple components.

Example:

```text
CN=u1
OU=Publish-Users
O=Test
C=US
```

The Subject represents the certificate's distinguished identity.

### Common Fields

| Field       | Meaning                   |
| ----------- | ------------------------- |
| **CN**      | Common Name               |
| **OU**      | Organizational Unit       |
| **O**       | Organization              |
| **C**       | Country                   |
| **DC**      | Domain Component          |
| **Email**   | Email identity            |
| **Subject** | Complete Subject identity |

### Mental Model

```text
Subject
 ├── CN
 ├── OU
 ├── O
 ├── C
 └── DC
```

### Checklist

* [ ] Required Subject fields exist
* [ ] CN is correct
* [ ] OU/O/DC values are expected
* [ ] Subject matching rules are understood
* [ ] Case sensitivity is considered

---

# 9. CN-Based Authentication

The Common Name can be used as a certificate identity.

Example:

```bash
set cn-type string
set cn "u1"
```

Certificate:

```text
Subject:
CN=u1
OU=Publish-Users
DC=test
DC=com
```

FortiGate extracts:

```text
CN → u1
```

Then:

```text
CN
 │
 ▼
Identity
 │
 ▼
LDAP Lookup
 │
 ▼
AD User
 │
 ▼
Authentication
```

### CN Checklist

* [ ] CN exists
* [ ] CN value is correct
* [ ] CN format matches the configured `cn-type`
* [ ] CN matches the expected identity
* [ ] Case sensitivity is considered
* [ ] LDAP mapping is correct where LDAP is used

---

# 10. Certificate Matching

Important certificate identity sources include:

```text
Subject
CN
SAN
UPN
LDAP Identity
```

| Method      | Identity Source                |
| ----------- | ------------------------------ |
| **Subject** | Certificate Distinguished Name |
| **CN**      | Certificate Common Name        |
| **SAN**     | Subject Alternative Name       |
| **UPN**     | AD Principal Name in SAN       |
| **LDAP**    | Directory identity lookup      |

### Important

```text
Subject ≠ CN
```

CN is only one component of the Subject.

### Matching Checklist

* [ ] Correct certificate field is selected
* [ ] Matching mode is understood
* [ ] Case sensitivity is considered
* [ ] SAN is inspected where required
* [ ] UPN is present where required
* [ ] LDAP identity matches the certificate identity

---

# 11. Subject Matching

Example:

```bash
config vpn certificate setting
    set subject-match substring
    set subject-set superset
    set cn-match substring
    set cn-allow-multi enable
end
```

### Substring Matching

Certificate:

```text
CN=varzesh3.com
```

Configured search:

```text
varzesh
```

Conceptually:

```text
varzesh
   ↓
varzesh3.com
```

Result:

```text
MATCH
```

### Exact-Value Concept

If the matching logic requires the complete value:

```text
Configured:
varzesh

Certificate:
varzesh3.com
```

Result:

```text
NO MATCH
```

### Checklist

* [ ] Understand whether matching is substring-based
* [ ] Verify complete identity when required
* [ ] Test the actual certificate field
* [ ] Avoid overly broad matching rules

> **Security Rule:** Avoid unnecessarily broad certificate matching expressions.

---

# 12. CN Matching

CN matching focuses specifically on the Common Name.

Example:

```text
Certificate:
CN=User1
```

Configured identity:

```text
User1
```

Expected:

```text
MATCH
```

But:

```text
Certificate:
CN=User1
```

Configured:

```text
user1
```

Do not assume:

```text
User1 = user1
```

### CN Matching Checklist

* [ ] CN exists
* [ ] CN is the intended identity field
* [ ] Matching mode is correct
* [ ] Case sensitivity is considered
* [ ] Wildcards/substrings are used only when justified
* [ ] CN does not unintentionally match another identity

---

# 13. Multiple CN Values

Where supported by the target FortiOS version:

```bash
set cn-allow-multi enable
```

This allows certificate processing to accommodate certificates containing multiple CN values.

Concept:

```text
Certificate
 ├── CN = user1
 ├── CN = user1@example.com
 └── ...
```

### Checklist

* [ ] Confirm multiple CN behavior in the target FortiOS release
* [ ] Verify which CN is expected
* [ ] Avoid ambiguous identity mappings
* [ ] Test certificates containing multiple identities

---

# 14. LDAP-Integrated PKI Authentication

LDAP integration can connect certificate identity to centralized Active Directory identity.

Two common approaches:

```text
Certificate
    │
    ├── Username + Password
    │
    └── Principal Name / UPN
```

### LDAP Integration Checklist

* [ ] LDAP server is configured
* [ ] LDAP connectivity works
* [ ] Base DN is correct
* [ ] User identity attribute is known
* [ ] LDAP lookup succeeds
* [ ] Certificate identity can be mapped to the directory identity
* [ ] User/group authorization is configured

---

# 15. UPN / Principal Name

For Active Directory environments, the **User Principal Name (UPN)** can provide a scalable certificate-to-user identity mapping.

Example:

```text
UPN:
u1@test.com
```

Concept:

```text
Client Certificate
        │
        ▼
      UPN
        │
        ▼
u1@test.com
        │
        ▼
      LDAP
        │
        ▼
 Active Directory
        │
        ▼
      User
```

### Why UPN Matters

Without scalable identity mapping:

```text
User 1 → Certificate Mapping
User 2 → Certificate Mapping
User 3 → Certificate Mapping
...
User 1000 → Certificate Mapping
```

With centralized LDAP/UPN mapping:

```text
Certificate
     ↓
UPN
     ↓
LDAP
     ↓
AD Identity
     ↓
Group Membership
```

### UPN Checklist

* [ ] UPN exists in AD
* [ ] UPN is unique
* [ ] Certificate contains the expected UPN identity
* [ ] UPN is placed in the expected certificate extension
* [ ] LDAP lookup can resolve the UPN
* [ ] AD identity matches the certificate identity

---

# 16. UPN in SAN

In Microsoft AD-integrated PKI deployments, UPN is commonly carried in the **Subject Alternative Name (SAN)**.

Concept:

```text
SAN
 └── Other Name
      └── UPN
           └── u1@test.com
```

Authentication flow:

```text
Client Certificate
        │
        ▼
       SAN
        │
        ▼
       UPN
        │
        ▼
  u1@test.com
        │
        ▼
      LDAP
        │
        ▼
      AD User
```

### SAN / UPN Checklist

* [ ] SAN extension exists
* [ ] UPN identity is present
* [ ] UPN value is correct
* [ ] UPN matches the intended AD account
* [ ] FortiGate is configured to use the required identity mapping
* [ ] LDAP lookup succeeds

> **Exam Tip:** For AD-integrated certificate authentication, distinguish the **certificate Subject/CN** from **UPN carried in SAN**.

---

# 17. LDAP Password vs UPN Authentication

## Model A — Username + Password

```text
Certificate
    │
    ▼
Extract Identity
    │
    ▼
Username
    │
    ├── Username
    └── Password
    │
    ▼
LDAP
    │
    ▼
Authentication
```

Example:

```bash
set ldap-server "winsrv-2016"
set ldap-mode password
```

### Checklist

* [ ] LDAP server is configured
* [ ] Username mapping is correct
* [ ] Password authentication is expected
* [ ] LDAP credentials are protected

---

## Model B — Certificate + UPN

```text
Certificate
    │
    ▼
SAN / UPN
    │
    ▼
LDAP Lookup
    │
    ▼
Active Directory
    │
    ▼
User / Group
```

### Enterprise Advantage

This architecture can preserve the passwordless characteristic of certificate authentication.

### Design Comparison

| Model           | Identity            | Password Dependency | Scalability |
| --------------- | ------------------- | ------------------: | ----------: |
| CN Mapping      | CN                  |            Low/None |       Lower |
| Subject Mapping | Subject             |            Low/None |      Medium |
| LDAP Password   | Username + Password |                 Yes |        High |
| LDAP + UPN      | Certificate UPN     | Can be passwordless |        High |

---

# 18. LDAP Username Mapping

Some designs can map a local certificate identity to another LDAP username.

Example:

```bash
config user peer
    edit "u1"
        set ldap-username "u2"
        set ldap-password <password>
        set ldap-mode password
    next
end
```

Concept:

```text
Certificate / Local Identity
            │
            ▼
           u1
            │
            │ Mapping
            ▼
           u2
            │
            ▼
       LDAP Authentication
```

### Checklist

* [ ] Mapping is intentional
* [ ] Source identity is correct
* [ ] LDAP target identity is correct
* [ ] Password is securely stored
* [ ] Mapping is documented
* [ ] Special-case mapping is not confused with scalable UPN-based identity

> **Operational Rule:** Use explicit identity mapping only when the architecture requires it.

---

# 19. PKI Authentication in Firewall Policy

Certificate authentication must ultimately connect to the appropriate authorization policy.

Example:

```bash
config firewall policy
    edit 1
        set auth-cert "ca-test.com"
    next
end
```

Conceptual flow:

```text
Client
  │
  ▼
Client Certificate
  │
  ▼
Certificate Validation
  │
  ▼
Identity
  │
  ▼
User / Group
  │
  ▼
Firewall Policy
  │
  ▼
Authorization
  │
  ▼
ACCESS
```

### Firewall Policy Checklist

* [ ] Correct source interface
* [ ] Correct source address
* [ ] Correct authentication mechanism
* [ ] Correct certificate/CA configuration
* [ ] Correct user/group mapping
* [ ] Correct destination
* [ ] Least-privilege access is enforced
* [ ] Logging is enabled where appropriate

---

# 20. Authentication vs Authorization

This is one of the most important PKI concepts.

### Authentication

```text
Is the certificate trusted?
        +
Who does it represent?
```

### Authorization

```text
What is this identity allowed to access?
```

Therefore:

```text
Certificate
    ↓
Certificate Validation
    ↓
Identity
    ↓
LDAP / AD
    ↓
Group Membership
    ↓
Firewall Policy
    ↓
Authorization
    ↓
Access
```

### Golden Rule

```text
Valid Certificate
       ≠
Unlimited Access
```

A certificate proves identity/trust within the authentication design.

The firewall policy determines what that authenticated identity can access.

---

# 21. PKI Security Checklist

## 🔐 CA Security

* [ ] Root CA is protected
* [ ] Issuing CA is protected
* [ ] CA certificates are valid
* [ ] Certificate chain is complete
* [ ] Expired certificates are rejected
* [ ] Certificate revocation is considered
* [ ] Only trusted CAs are configured

## 👤 Client Certificate

* [ ] Certificate is valid
* [ ] Certificate is not expired
* [ ] Certificate is issued by a trusted CA
* [ ] Private key is protected
* [ ] Required Subject fields exist
* [ ] Required SAN fields exist
* [ ] UPN is correct where required
* [ ] Identity is unique

## 🔎 Certificate Matching

* [ ] CN matching is intentional
* [ ] Subject matching is intentional
* [ ] SAN matching is understood
* [ ] UPN mapping is correct
* [ ] Case sensitivity is considered
* [ ] Broad substring matching is avoided where possible
* [ ] Multiple CN behavior is understood

## 🗂 LDAP / AD

* [ ] LDAP server is reachable
* [ ] Base DN is correct
* [ ] LDAP identity attribute is correct
* [ ] User lookup succeeds
* [ ] Group membership is correct
* [ ] Dedicated service accounts are used
* [ ] LDAP credentials are protected

## 🔥 FortiGate

* [ ] User Peer is configured
* [ ] Correct CA is selected
* [ ] CA verification is enabled
* [ ] LDAP server is correct
* [ ] LDAP mode is appropriate
* [ ] Certificate authentication is enabled
* [ ] Correct user/group is referenced
* [ ] Firewall policy provides least-privilege authorization

---

# 22. PKI Troubleshooting Checklist

When certificate authentication fails, troubleshoot in this order:

### Step 1 — Certificate

* [ ] Is the certificate present?
* [ ] Is the certificate expired?
* [ ] Is the certificate valid?
* [ ] Is the private key available?
* [ ] Is the certificate chain complete?

### Step 2 — CA

* [ ] Is the issuing CA trusted?
* [ ] Is the correct CA configured?
* [ ] Is the intermediate CA available?
* [ ] Does mandatory CA verification succeed?

### Step 3 — Identity

* [ ] Is CN present?
* [ ] Is Subject correct?
* [ ] Is SAN present?
* [ ] Is UPN present where required?
* [ ] Is the identity case correct?
* [ ] Does the certificate identity match AD?

### Step 4 — LDAP

* [ ] Is LDAP reachable?
* [ ] Is Base DN correct?
* [ ] Is the correct user attribute used?
* [ ] Does LDAP lookup succeed?
* [ ] Does UPN resolve to the correct AD user?

### Step 5 — Authorization

* [ ] Is the user in the correct group?
* [ ] Is the correct certificate authentication method referenced?
* [ ] Does the firewall policy match?
* [ ] Is the destination allowed?

---

# 23. Troubleshooting Matrix

| Symptom                                   | Primary Checks                             |
| ----------------------------------------- | ------------------------------------------ |
| Certificate rejected                      | Expiry, chain, CA trust                    |
| CA validation fails                       | Trusted CA / certificate chain             |
| Client certificate not accepted           | Certificate validity and CA                |
| CN mismatch                               | CN value and matching mode                 |
| Subject mismatch                          | Subject structure and matching rules       |
| SAN/UPN lookup fails                      | SAN, UPN and LDAP mapping                  |
| LDAP lookup fails                         | Connectivity, Base DN, attributes          |
| User found but access denied              | Group and firewall policy                  |
| Password unexpectedly requested           | LDAP authentication mode                   |
| Multiple CN issue                         | `cn-allow-multi` and certificate structure |
| PKI deployment does not scale             | Consider centralized LDAP/UPN mapping      |
| Authentication succeeds but traffic fails | Authorization / firewall policy            |

---

# 24. PKI Deployment Models

## Model A — CN Matching

```text
Certificate
    │
    ▼
CN = u1
    │
    ▼
FortiGate User Peer
    │
    ▼
Authentication
```

### Suitable For

* [ ] Labs
* [ ] Testing
* [ ] Small environments
* [ ] Explicit certificate-to-user mappings

---

## Model B — Subject Matching

```text
Certificate
    │
    ▼
Subject
    │
    ▼
FortiGate Matching
    │
    ▼
Authentication
```

### Suitable For

* [ ] Structured certificate environments
* [ ] Predictable Subject formats
* [ ] Controlled certificate templates

---

## Model C — LDAP + UPN

```text
Certificate
    │
    ▼
SAN / UPN
    │
    ▼
LDAP
    │
    ▼
Active Directory
    │
    ▼
User / Groups
```

### Suitable For

* [ ] Large enterprises
* [ ] Large PKI deployments
* [ ] Centralized identity management
* [ ] Passwordless authentication designs
* [ ] Scalable certificate-to-user mapping

### Design Rule

```text
Small / Explicit
     ↓
CN / Subject Mapping

Large / Centralized
     ↓
LDAP + UPN
```

---

# 25. Enterprise PKI Architecture

Recommended conceptual architecture:

```text
                     ┌──────────────────┐
                     │       AD         │
                     │ LDAP / Identity  │
                     └────────┬─────────┘
                              │
                              │ Identity
                              ▼
                     ┌──────────────────┐
                     │       CA         │
                     │ Root / Issuing   │
                     └────────┬─────────┘
                              │
                       Issue Certificate
                              │
                              ▼
                     ┌──────────────────┐
                     │      Client      │
                     │ Certificate      │
                     │ SAN / UPN        │
                     └────────┬─────────┘
                              │
                              │ TLS
                              ▼
                     ┌──────────────────┐
                     │    FortiGate     │
                     │                  │
                     │ CA Validation    │
                     │       ↓          │
                     │ Identity Match   │
                     │       ↓          │
                     │ LDAP Lookup      │
                     └────────┬─────────┘
                              │
                              ▼
                       User / Group
                              │
                              ▼
                      Firewall Policy
                              │
                              ▼
                           ACCESS
```

### Enterprise Deployment Checklist

* [ ] Centralized CA infrastructure
* [ ] Standardized certificate templates
* [ ] Consistent UPN format
* [ ] LDAP integration
* [ ] Centralized user/group management
* [ ] Certificate lifecycle management
* [ ] Certificate revocation process
* [ ] FortiGate certificate validation
* [ ] Least-privilege authorization
* [ ] Monitoring and logging
* [ ] Recovery procedure

---

# 26. NSE Exam Memory Map

```text
                         PKI
                          │
          ┌───────────────┼────────────────┐
          │               │                │
         CA             Client           LDAP
          │           Certificate          │
          │               │                │
     Trust Chain          │             Identity
          │               │                │
          └───────┬───────┘                │
                  ▼                        │
             FortiGate                    │
                  │                        │
          ┌───────┼────────┐              │
          │       │        │              │
       Subject    CN      SAN ◄───────────┘
          │       │        │
          │       │       UPN
          │       │        │
          └───────┴────────┘
                  │
                  ▼
            Authentication
                  │
                  ▼
              User / Group
                  │
                  ▼
           Firewall Policy
                  │
                  ▼
                Access
```

### Remember

```text
CA
↓
Trust

CN / Subject / SAN
↓
Certificate Identity

UPN
↓
AD Identity

LDAP
↓
Directory Lookup

User / Group
↓
Authorization Context

Firewall Policy
↓
Access Control
```

---

# 27. Quick CLI Reference

## User Peer

```bash
config user peer
    edit "u1"
        set mandatory-ca-verify enable
        set ca "ca-test.com"
        set cn-type string
        set cn "u1"
        set ldap-server "winsrv-2016"
        set ldap-mode password
    next
end
```

* [ ] Verify CA
* [ ] Verify CN
* [ ] Verify LDAP server
* [ ] Verify LDAP mode

---

## LDAP Username Mapping

```bash
config user peer
    edit "u1"
        set ldap-username "u2"
        set ldap-password <password>
        set ldap-mode password
    next
end
```

> Never store real passwords in public documentation.

---

## Certificate Matching

```bash
config vpn certificate setting
    set subject-match substring
    set subject-set superset
    set cn-match substring
    set cn-allow-multi enable
end
```

* [ ] Verify matching mode
* [ ] Verify subject behavior
* [ ] Verify CN behavior
* [ ] Verify multiple-CN requirements

---

## Firewall Certificate Authentication

```bash
config firewall policy
    edit 1
        set auth-cert "ca-test.com"
    next
end
```

* [ ] Verify policy
* [ ] Verify CA
* [ ] Verify source/destination
* [ ] Verify authorization

---

# 28. One-Minute PKI Checklist

```text
[ ] Is the client certificate valid?
[ ] Is the certificate trusted by the configured CA?
[ ] Is the certificate chain complete?
[ ] Is mandatory CA verification enabled?
[ ] Is the expected CN/Subject/SAN present?
[ ] Is the identity case correct?
[ ] Is UPN present where required?
[ ] Does UPN map to the correct AD user?
[ ] Is LDAP reachable?
[ ] Is LDAP lookup successful?
[ ] Is the correct User Peer configured?
[ ] Is the correct user/group matched?
[ ] Is certificate authentication enabled?
[ ] Does the firewall policy permit the authenticated identity?
[ ] Is authorization limited to the required resources?
```

---

# 29. Golden Rules

> **1. CA validation establishes certificate trust.**

> **2. Certificate identity matching does not replace CA validation.**

> **3. Subject and CN are not the same thing.**

> **4. CN is only one component of the certificate Subject.**

> **5. SAN can contain additional identity information such as UPN.**

> **6. UPN provides a powerful identity mapping mechanism for AD-integrated PKI.**

> **7. LDAP provides centralized directory lookup and identity integration.**

> **8. Certificate authentication should not automatically imply unrestricted access.**

> **9. Authentication proves identity; authorization determines access.**

> **10. Avoid overly broad certificate matching rules.**

> **11. Protect client private keys.**

> **12. Never publish production certificates or LDAP credentials.**

> **13. Use standardized certificate templates and identity formats.**

> **14. Large PKI deployments benefit from centralized identity mapping.**

> **15. Always validate PKI behavior against the exact FortiOS version being deployed.**

---

# 30. Final PKI Mental Model

```text
┌─────────────────────────────────────────────┐
│            FORTIGATE PKI AUTH               │
├─────────────────────────────────────────────┤
│                                             │
│  Certificate Authority                      │
│          ↓                                  │
│  Client Certificate                         │
│          ↓                                  │
│  CA / Chain Validation                      │
│          ↓                                  │
│  Subject / CN / SAN                         │
│          ↓                                  │
│  Identity Extraction                        │
│          ↓                                  │
│  UPN / LDAP Lookup                           │
│          ↓                                  │
│  Active Directory                           │
│          ↓                                  │
│  User / Group                               │
│          ↓                                  │
│  Firewall Policy                            │
│          ↓                                  │
│  AUTHORIZATION                               │
│          ↓                                  │
│  AUTHORIZED ACCESS                           │
│                                             │
└─────────────────────────────────────────────┘
```

## 🔥 The 5 Things to Remember

```text
1. CA validation establishes certificate trust.

2. Subject, CN and SAN are different certificate identity locations.

3. UPN in SAN can provide scalable AD identity mapping.

4. LDAP connects certificate identity to centralized directory identity.

5. Authentication proves identity; firewall policy provides authorization.
```

---

# 🧠 SheynShield PKI Mental Model

```text
PKI
│
├── CA
│   └── Trust
│
├── Certificate
│   ├── Subject
│   ├── CN
│   └── SAN
│       └── UPN
│
├── FortiGate
│   ├── User Peer
│   ├── CA Verification
│   └── Certificate Matching
│
├── LDAP / AD
│   ├── Identity Lookup
│   ├── User
│   └── Group
│
└── Firewall Policy
    └── Authorization
```

### The Core Chain

```text
CA
 ↓
Certificate
 ↓
Validation
 ↓
Identity
 ↓
UPN / LDAP
 ↓
AD User
 ↓
Group
 ↓
Firewall Policy
 ↓
Access
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

## 🏷️ Keywords

`FortiGate PKI` • `FortiGate Certificate Authentication` • `FortiGate PKI Authentication` • `FortiGate LDAP Certificate Authentication` • `FortiGate User Peer` • `FortiGate CA Certificate` • `FortiGate Client Certificate` • `FortiGate UPN` • `FortiGate SAN` • `FortiGate Active Directory` • `FortiGate LDAP` • `FortiOS Certificate Authentication` • `X.509 Authentication` • `Certificate Based Authentication` • `Fortinet PKI` • `Fortinet Certificate Authentication`

---

**SheynShield | Engineering Secure Networks**

`FortiGate` • `PKI` • `Certificate Authentication` • `LDAP` • `Active Directory` • `CA` • `UPN` • `SAN` • `X.509` • `Identity Security` • `Network Security`

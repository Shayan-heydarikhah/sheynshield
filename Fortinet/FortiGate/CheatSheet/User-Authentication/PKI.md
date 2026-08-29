# 🔐 FortiGate PKI & Certificate-Based Authentication  

> **SheynShield | FortiGate Security Engineering**
>
> Practical reference for **PKI authentication, CA validation, client certificates, certificate field matching, LDAP integration, CN/Subject matching, UPN/Principal Name, and certificate-based firewall authentication**.

---

## 1. PKI Authentication — Big Picture

**PKI (Public Key Infrastructure)** allows FortiGate to authenticate users using **digital certificates** instead of relying only on passwords.

A typical enterprise architecture:

```text
                    ┌────────────────────┐
                    │   Active Directory │
                    │        LDAP        │
                    └─────────┬──────────┘
                              │
                              │ User Identity
                              ▼
                    ┌────────────────────┐
                    │   Certificate CA   │
                    │  Enterprise CA     │
                    └─────────┬──────────┘
                              │
                       Issue Certificate
                              │
                              ▼
                     ┌─────────────────┐
                     │ Client / User   │
                     │ Certificate     │
                     └────────┬────────┘
                              │
                              │ TLS / Certificate
                              ▼
                    ┌────────────────────┐
                    │     FortiGate      │
                    │ PKI Authentication │
                    └─────────┬──────────┘
                              │
                              ▼
                    LDAP Identity Check
                              │
                              ▼
                         User Group
                              │
                              ▼
                      Firewall / VPN
```

### Core Components

| Component              | Responsibility                          |
| ---------------------- | --------------------------------------- |
| **AD / LDAP**          | User identity repository                |
| **CA**                 | Issues and signs certificates           |
| **Client Certificate** | Represents the user/device identity     |
| **FortiGate**          | Validates certificate and maps identity |
| **Firewall Policy**    | Enforces authorization                  |

---

# 2. PKI Prerequisites

Before implementing certificate-based authentication:

```text
[ ] Active Directory / LDAP available
[ ] Enterprise CA available
[ ] Client certificates issued
[ ] CA certificate imported/trusted by FortiGate
[ ] Unique domain namespace configured
[ ] Certificate contains the required identity fields
[ ] LDAP server configured on FortiGate
[ ] User/group mapping configured
[ ] Firewall policy configured for certificate authentication
```

### Unique Domain

Use a properly designed, unique domain namespace for the PKI/AD environment.

Example:

```text
test.com
```

Avoid designing certificate identity around ambiguous or duplicated namespaces.

---

# 3. Certificate-Based Authentication Flow

The high-level authentication process:

```text
Client
  │
  │ Client Certificate
  ▼
FortiGate
  │
  ├── Verify CA
  │
  ├── Inspect Certificate
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
     User / Group Match
          │
          ▼
       Firewall Policy
          │
          ▼
         Access
```

---

# 4. FortiGate User Peer

A **user peer** defines how FortiGate identifies and validates a certificate-based user.

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

### Important Identity Rule

The configured certificate identity must correspond to the identity expected by the authentication system.

For example:

```text
AD Username
     │
     ▼
u1

Certificate Identity
     │
     ▼
CN = u1
```

Therefore:

```text
Certificate CN = u1
AD Username    = u1
```

---

# 5. Mandatory CA Verification

```bash
set mandatory-ca-verify enable
```

This forces FortiGate to verify the client certificate against the configured trusted CA.

Conceptually:

```text
Client Certificate
        │
        ▼
Is certificate signed by trusted CA?
        │
    ┌───┴───┐
   YES      NO
    │        │
    ▼        ▼
Continue    Reject
```

### Security Principle

For production PKI authentication:

> **Do not rely only on certificate field matching. Establish trust in the certificate chain first.**

---

# 6. CA Configuration

Example:

```bash
set ca "ca-test.com"
```

The CA object should correspond to the trusted CA certificate configured on FortiGate.

Typical hierarchy:

```text
Root CA
   │
   └── Intermediate CA
          │
          └── User Certificate
```

FortiGate must trust the appropriate CA chain required to validate the client certificate.

---

# 7. Certificate Subject / Distinguished Name

A certificate's **Subject** contains multiple identity components.

Example:

```text
CN=u1
OU=Publish-Users
O=Test
C=PL
emailAddress=u1@test.com
```

This entire structure is commonly referred to as the **Distinguished Name (DN)**.

Typical certificate fields:

| Field   | Meaning                   |
| ------- | ------------------------- |
| CN      | Common Name               |
| OU      | Organizational Unit       |
| O       | Organization              |
| C       | Country                   |
| Email   | Email identity            |
| Subject | Complete subject identity |

---

# 8. CN-Based Authentication

Example:

```bash
set cn-type string
set cn "u1"
```

FortiGate extracts the **Common Name (CN)** from the certificate and compares it with the configured value.

Example:

```text
Certificate:

Subject:
CN=u1
OU=publish-users
DC=test
DC=com
```

FortiGate:

```text
CN → u1
```

Then:

```text
u1
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

---

# 9. CN Type

The `cn-type` setting controls how the CN value is interpreted.

Example:

```bash
set cn-type string
```

Conceptually:

```text
CN = u1
```

can be treated as a normal string identity.

Depending on the selected CN type, FortiGate can apply different format expectations to the CN.

### Example — Email Identity

A certificate could contain:

```text
CN = jcarrey@fortinet.com
```

If the configured CN type expects an email-formatted identity, the CN must comply with that format.

---

# 10. Certificate Matching Methods

FortiGate can identify a client using different certificate fields.

The three important approaches are:

```text
1. Subject
2. CN
3. LDAP
```

Conceptual comparison:

| Method      | Identity Source                    |
| ----------- | ---------------------------------- |
| **Subject** | Certificate Distinguished Name     |
| **CN**      | Certificate Common Name            |
| **LDAP**    | Certificate identity → LDAP lookup |

> **Certificate field comparisons are case-sensitive.**

---

# 11. Subject Matching

Example configuration:

```bash
config vpn certificate setting
    set subject-match substring
    set subject-set superset
    set cn-match substring
    set cn-allow-multi enable
end
```

### Subject Match

`substring` means FortiGate can search for a configured value inside the received certificate field.

Example:

```text
Certificate:

CN=varzesh3.com
```

Searching for:

```text
varzesh
```

can match:

```text
varzesh3.com
```

because `varzesh` is a substring.

---

# 12. Substring vs Exact Value

This distinction is important.

### Substring

```text
Certificate:
CN=varzesh3.com

Search:
varzesh
```

Result:

```text
MATCH
```

Because:

```text
varzesh
└─────────────┐
              ▼
        varzesh3.com
```

### Exact / Value-Oriented Matching

If the configuration expects the actual value rather than a substring, the complete relevant identity must match.

```text
Configured:
varzesh

Certificate:
varzesh3.com

Result:
NO MATCH
```

> **Exam tip:** Always distinguish **substring matching** from matching the complete field value.

---

# 13. Subject Set — Superset vs Subset

The certificate Subject can contain many components:

```text
CN
OU
O
DC
Email
...
```

### Superset

Conceptually requires the relevant complete set of Subject components.

```text
Certificate Subject
 ├── CN
 ├── OU
 ├── O
 └── DC
```

The configured subject representation must cover the required Subject structure.

### Subset

Allows only part of the Subject components to be used for matching.

```text
Certificate Subject
 ├── CN      ← Match
 ├── OU
 ├── O
 └── DC
```

### Quick Memory

```text
SUPerset → broader/full Subject representation
SUBset   → selected portion of Subject
```

---

# 14. Multiple CN Values

```bash
set cn-allow-multi enable
```

This allows certificate processing to support multiple CN values where the certificate contains them.

Example concept:

```text
Certificate
 ├── CN = user1
 ├── CN = user1@example.com
 └── ...
```

Useful when certificate structures contain multiple identity representations.

---

# 15. LDAP-Integrated Certificate Authentication

LDAP integration provides a scalable approach for certificate-based user authentication.

There are two important approaches:

```text
Certificate
    │
    ├── Username + Password
    │
    └── Principal Name / UPN
```

---

# 16. LDAP Password Method

Example:

```bash
set ldap-server "winsrv-2016"
set ldap-mode password
```

In this mode, FortiGate uses a username/password mechanism against LDAP.

Conceptually:

```text
Certificate
    │
    ▼
Extract User Identity
    │
    ▼
Username
    │
    ▼
LDAP
    │
    ├── Username
    └── Password
    │
    ▼
Authentication
```

### Limitation

For large PKI deployments, requiring a password for every PKI user reduces the benefit of certificate-based authentication.

---

# 17. LDAP Principal Name / UPN Method ⭐

The more scalable PKI model is to use the **principal name / UPN** contained in the certificate.

Example:

```text
Certificate SAN:

UPN = u1@test.com
```

FortiGate:

```text
Certificate UPN
      │
      ▼
LDAP Lookup
      │
      ▼
AD User
      │
      ▼
Match
      │
      ▼
Authentication Success
```

### Why It Scales

Instead of creating an individual certificate-authentication object for every user:

```text
1000 users
   ↓
1000 individual mappings
```

the LDAP-integrated approach can use the certificate identity to locate the corresponding AD user.

```text
1000 users
   ↓
One LDAP integration
   ↓
UPN-based lookup
```

---

# 18. UPN in Subject Alternative Name

The **User Principal Name (UPN)** is commonly represented in the certificate's **Subject Alternative Name (SAN)**.

Example:

```text
SAN
 └── Other Name
      └── UPN
           └── u1@test.com
```

FortiGate can use this identity to search the LDAP directory.

Conceptually:

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
 AD User Match
```

---

# 19. Why Principal Name Is Preferred for PKI

### Username + Password

```text
User
 │
 ├── Certificate
 ├── Username
 └── Password
```

### Principal Name

```text
User
 │
 └── Certificate
        │
        └── UPN
              │
              ▼
             LDAP
```

The second architecture better preserves the passwordless nature of certificate authentication.

### Enterprise Advantage

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
     ↓
Authorization
```

This model scales significantly better in environments with many PKI users.

---

# 20. Mapping PKI Users to LDAP

A typical enterprise flow:

```text
                    Certificate Authority
                            │
                            │ Issue certificate
                            ▼
                         User u1
                            │
                    UPN = u1@test.com
                            │
                            ▼
                       FortiGate
                            │
                     Certificate Check
                            │
                            ▼
                         LDAP
                            │
                            ▼
                    AD: u1@test.com
                            │
                            ▼
                       User Group
                            │
                            ▼
                     Firewall Policy
```

---

# 21. LDAP Username Override

In some designs, the local FortiGate identity can be mapped to another LDAP username.

Example:

```bash
set ldap-username "u2"
set ldap-password <password>
set ldap-mode password
```

Conceptually:

```text
FortiGate User
      │
      ▼
     u1
      │
      │ LDAP Mapping
      ▼
     u2
      │
      ▼
LDAP Authentication
```

This is a special mapping scenario and should not be confused with scalable UPN-based PKI authentication.

---

# 22. Certificate Authentication in Firewall Policy

After configuring the certificate identity/user group, the firewall policy must reference the appropriate certificate authentication mechanism.

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
Internet / VPN / Client
          │
          ▼
    Client Certificate
          │
          ▼
       FortiGate
          │
          ▼
   Certificate Validation
          │
          ▼
      User Identity
          │
          ▼
    Firewall Policy
          │
          ▼
        Access
```

---

# 23. Authentication vs Authorization

This distinction is extremely important.

### Authentication

```text
Is this certificate valid?
        +
Who does this certificate represent?
```

### Authorization

```text
What is this authenticated user allowed to access?
```

Therefore:

```text
Certificate
    ↓
Authentication
    ↓
LDAP Identity
    ↓
Group Membership
    ↓
Firewall Policy
    ↓
Authorization
```

> A valid certificate does **not** automatically mean unrestricted network access.

---

# 24. PKI Security Checklist

### CA

* [ ] Trusted CA is configured
* [ ] CA certificate is valid
* [ ] Certificate chain is complete
* [ ] CA is not expired
* [ ] Correct CA is selected

### Client Certificate

* [ ] Certificate is valid
* [ ] Certificate has not expired
* [ ] Certificate is issued by the trusted CA
* [ ] Required Subject/CN/SAN fields exist
* [ ] UPN is correctly populated where required
* [ ] Identity matches the expected LDAP user

### LDAP

* [ ] LDAP server is reachable
* [ ] Correct Base DN
* [ ] Correct user attributes
* [ ] Correct group mapping
* [ ] LDAP lookup succeeds

### FortiGate

* [ ] User peer configured
* [ ] CA verification enabled
* [ ] Correct matching method selected
* [ ] Certificate authentication enabled in policy
* [ ] Correct user group referenced

---

# 25. PKI Troubleshooting Flow

```text
                 PKI Authentication Failed
                           │
                           ▼
                 Is certificate valid?
                    /             \
                  NO               YES
                  │                 │
                  ▼                 ▼
             Fix certificate    Is CA trusted?
                                  /       \
                                NO         YES
                                │           │
                                ▼           ▼
                           Configure CA   Identity found?
                                           /       \
                                         NO         YES
                                         │           │
                                         ▼           ▼
                                     Check CN/    Check LDAP
                                     Subject/UPN   lookup
                                                     │
                                                     ▼
                                               Group mapping
                                                     │
                                                     ▼
                                              Firewall Policy
```

---

# 26. Troubleshooting Matrix

| Symptom                          | Check                                |
| -------------------------------- | ------------------------------------ |
| Certificate rejected             | CA trust, expiry, certificate chain  |
| CA validation fails              | Trusted CA configuration             |
| CN does not match                | CN value and matching mode           |
| Subject matching fails           | Subject structure / case sensitivity |
| LDAP lookup fails                | LDAP connectivity and Base DN        |
| UPN lookup fails                 | SAN/UPN value and LDAP mapping       |
| User authenticated but no access | User group / firewall policy         |
| Multiple CN problem              | `cn-allow-multi`                     |
| Password unexpectedly requested  | LDAP mode                            |
| PKI doesn't scale                | Consider LDAP + Principal Name/UPN   |

---

# 27. Important Certificate Matching Rules 🧠

### Rule 1 — Certificate trust first

```text
Certificate
     ↓
CA Validation
     ↓
Identity Matching
```

Not:

```text
Certificate
     ↓
Identity Matching
     ↓
CA Validation
```

---

### Rule 2 — Matching is case-sensitive

If:

```text
Certificate:
CN=User1
```

and configuration expects:

```text
user1
```

do not assume they are equivalent.

```text
User1 ≠ user1
```

---

### Rule 3 — Subject ≠ CN

```text
Subject
 ├── CN
 ├── OU
 ├── O
 ├── DC
 └── ...
```

CN is only one component of the Subject.

---

### Rule 4 — SAN can carry UPN

For scalable AD-integrated PKI:

```text
Certificate
   ↓
SAN / UPN
   ↓
LDAP
   ↓
AD User
```

---

### Rule 5 — Authentication ≠ Authorization

```text
Valid Certificate
       ≠
Full Network Access
```

Access is ultimately controlled by authorization policies.

---

# 28. PKI Deployment Models

### Model A — CN Matching

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

**Good for:**

* Labs
* Small deployments
* Explicit certificate-to-user mapping

---

### Model B — Subject Matching

```text
Certificate
    │
    ▼
Subject
    │
    ▼
FortiGate Matching Rules
    │
    ▼
Authentication
```

**Good for:**

* Structured certificate identities
* Organizations using predictable certificate Subject formats

---

### Model C — LDAP + UPN ⭐

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

**Best suited for:**

* Large enterprises
* Large PKI deployments
* Passwordless authentication
* Centralized identity management

---

# 29. PKI Architecture — Enterprise Recommendation

```text
                     ┌──────────────────┐
                     │       AD         │
                     │ LDAP / Identity  │
                     └────────┬─────────┘
                              │
                              │
                     ┌────────▼─────────┐
                     │       CA         │
                     │ Root / Issuing   │
                     └────────┬─────────┘
                              │
                       User Certificate
                              │
                              ▼
                     ┌──────────────────┐
                     │      Client      │
                     │ Cert + UPN/SAN   │
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
                           Access
```

---

# 30. NSE Exam Memory Map

```text
                         PKI
                          │
          ┌───────────────┼────────────────┐
          │               │                │
         CA             Client           LDAP
          │           Certificate          │
          │               │                │
     Trust Chain          │              Identity
          │               │                │
          └───────┬───────┘                │
                  ▼                        │
             FortiGate                    │
                  │                        │
          ┌───────┼────────┐              │
          │       │        │              │
       Subject    CN      LDAP ◄──────────┘
          │       │        │
          └───────┴────────┘
                  │
                  ▼
             Authentication
                  │
                  ▼
              User Group
                  │
                  ▼
           Firewall Policy
                  │
                  ▼
                Access
```

---

# 31. Quick CLI Reference

### User Peer

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

### LDAP Username Mapping

```bash
config user peer
    edit "u1"
        set ldap-username "u2"
        set ldap-password <password>
        set ldap-mode password
    next
end
```

### Certificate Matching

```bash
config vpn certificate setting
    set subject-match substring
    set subject-set superset
    set cn-match substring
    set cn-allow-multi enable
end
```

### Firewall Policy

```bash
config firewall policy
    edit 1
        set auth-cert "ca-test.com"
    next
end
```

---

# 32. Final PKI  

```text
┌─────────────────────────────────────────────┐
│              FORTIGATE PKI                  │
├─────────────────────────────────────────────┤
│                                             │
│  CA                                          │
│   ↓                                          │
│  Client Certificate                          │
│   ↓                                          │
│  CA Verification                             │
│   ↓                                          │
│  Subject / CN / SAN                          │
│   ↓                                          │
│  Identity Extraction                         │
│   ↓                                          │
│  LDAP / UPN Lookup                            │
│   ↓                                          │
│  AD User                                     │
│   ↓                                          │
│  Group Membership                            │
│   ↓                                          │
│  Firewall Policy                             │
│   ↓                                          │
│  AUTHORIZED ACCESS                           │
│                                             │
└─────────────────────────────────────────────┘
```

### 🔥 The 5 Things to Remember

```text
1. CA validation establishes certificate trust.
2. Subject and CN are certificate identity fields.
3. Certificate matching is case-sensitive.
4. UPN in SAN can provide scalable LDAP-based identity mapping.
5. Authentication proves identity; firewall policy provides authorization.
```

---

**SheynShield | Engineering Secure Networks**

`FortiGate` • `PKI` • `Certificate Authentication` • `LDAP` • `Active Directory` • `CA` • `UPN` • `X.509` • `Identity Security`


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

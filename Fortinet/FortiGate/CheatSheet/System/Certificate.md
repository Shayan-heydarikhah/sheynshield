# FortiGate Certificates & PKI 

> **FortiOS | Certificates, PKI, CSR, CA, Remote Certificates, Local Certificates, ACME & Let's Encrypt**
>
> **SheynShield | Engineering Secure Networks**

---

## 1. Certificate Overview

FortiOS uses digital certificates in multiple security features:

* 🔐 Administrative HTTPS access
* 🔑 SSH certificate authentication
* 🌐 SSL VPN
* 🛡️ SSL/Deep Packet Inspection
* 🔒 IPsec VPN
* ⚖️ Virtual Server / Load Balancing
* 🤝 SAML authentication
* 🏗️ Fortinet Security Fabric
* 📜 Certificate Authority validation
* 🔄 Automated certificate management
* 🌍 ACME / Let's Encrypt

The foundation is **PKI (Public Key Infrastructure)**.

```text
                    PKI
                     │
        ┌────────────┼────────────┐
        │            │            │
       CA           CSR       Certificate
        │            │            │
   Trust Anchor   Request     Identity
        │            │            │
        └────────────┴────────────┘
                     │
                  FortiGate
```

---

# 2. Public Key Infrastructure — PKI

A certificate is used to establish:

1. **Identity**
2. **Trust**
3. **Secure communication**

Typical PKI flow:

```text
FortiGate
   │
   │ Generate Private/Public Key
   │
   ▼
Generate CSR
   │
   │ Certificate Signing Request
   ▼
Certificate Authority (CA)
   │
   │ Validate + Sign
   ▼
Signed Certificate
   │
   ▼
Import into FortiGate
   │
   ▼
Trusted Identity / Encryption
```

### Important

A certificate contains the **public key**.

The **private key must remain protected** by the entity that owns the certificate.

```text
Certificate
├── Subject
├── Issuer
├── Public Key
├── Validity
├── SAN
└── CA Signature

Private Key
└── MUST remain protected
```

---

# 3. Certificate Types in FortiGate

FortiGate primarily works with:

| Certificate Type       | Purpose                                                     |
| ---------------------- | ----------------------------------------------------------- |
| **Local Certificate**  | Identify the FortiGate itself                               |
| **CA Certificate**     | Trust certificates signed by a CA                           |
| **Remote Certificate** | Identify/trust a remote entity using its public certificate |
| **CRL**                | Certificate Revocation List                                 |
| **OCSP Server**        | Online certificate status validation                        |

---

# 4. Local Certificate

A **local certificate** represents the FortiGate or a service hosted by FortiGate.

Common use cases:

* HTTPS administrative GUI
* SSL VPN portal
* Virtual Server / Load Balancer
* TLS services
* Other services where FortiGate must present its identity

```text
Client
   │
   │ TLS
   ▼
FortiGate
   │
   └── Presents Local Certificate
```

---

# 5. CA Certificate

A **CA certificate** establishes trust.

```text
Root CA
   │
   └── Intermediate CA
           │
           └── Server Certificate
                    │
                    ▼
                 FortiGate
```

FortiGate can use trusted CA certificates to validate certificates presented by remote entities.

---

# 6. Remote Certificate

A **remote certificate** contains the remote entity's public certificate.

The private key is not required by FortiGate for this use case.

Typical use cases include:

* Authentication
* SAML
* Fortinet Fabric connectivity
* Certificate-based trust
* Certificate inspection/validation

### Example — SAML

When FortiGate acts as an **Identity Provider (IdP)**, an SP certificate can optionally be specified.

When FortiGate acts as a **Service Provider (SP)**, the certificate used by the IdP must be specified.

These certificates can be imported as **remote certificates** because FortiGate does not need the remote entity's private key.

---

# 7. Local vs CA vs Remote

| Type                   | Represents                |         Private Key |
| ---------------------- | ------------------------- | ------------------: |
| **Local Certificate**  | FortiGate / local service |    Usually required |
| **CA Certificate**     | Trusted CA                | ❌ No private CA key |
| **Remote Certificate** | Remote entity             |                ❌ No |
| **CRL**                | Revocation information    |                   ❌ |

### Memory Trick

```text
LOCAL  → "I am FortiGate"
CA     → "I trust this CA"
REMOTE → "I trust/identify this remote entity"
CRL    → "These certificates are revoked"
```

---

# 8. CSR — Certificate Signing Request

A **CSR (Certificate Signing Request)** is generated when the FortiGate wants a CA to issue a certificate.

```text
FortiGate
   │
   ├── Generate private key
   │
   ├── Generate public key
   │
   └── Generate CSR
           │
           ▼
          CA
           │
       Sign CSR
           │
           ▼
     Signed Certificate
           │
           ▼
       FortiGate
```

### CSR Contains Information Such As

* Subject
* Public key
* Organization
* Country
* State/Province
* City
* Organizational Unit
* Email
* SANs

> **Critical:** The CSR does **not** contain the private key.

---

# 9. Generate Certificate with CMP

FortiGate supports automated certificate management using **CMP**.

```bash
execute vpn certificate local generate cmp \
    <certificate-name> \
    <key-size> \
    <server> \
    <path> \
    <server-certificate> \
    <auth-certificate> \
    <user> \
    <password> \
    <subject> \
    [sans] \
    [options]
```

### CMP

```text
CMP
└── Certificate Management Protocol
```

CMP can be used for automated certificate enrollment and management, particularly in larger PKI environments.

---

# 10. Built-in SSL Certificates

FortiOS can generate built-in certificates used by FortiGate.

### Default SSL CA

```bash
execute vpn certificate local generate default-ssl-ca
```

Used with built-in SSL inspection certificate functionality.

### Default SSL Key Certificates

```bash
execute vpn certificate local generate default-ssl-key-certs
```

### Default SSL Server Key

```bash
execute vpn certificate local generate default-ssl-serv-key
```

---

# 11. RSA Certificate Generation

Example syntax:

```bash
execute vpn certificate local generate rsa \
    <certificate_name> \
    <key_size> \
    <subject> \
    <country> \
    <state/province> \
    <city> \
    <organization> \
    <ou> \
    <email> \
    [sans] \
    [options]
```

Typical concept:

```text
RSA
 └── Public Key
      └── CSR
           └── CA signing
                └── Certificate
```

---

# 12. ECC Certificate Generation

FortiGate can generate elliptic-curve certificates.

```bash
execute vpn certificate local generate ec \
    <certificate_name> \
    <curve_name> \
    <subject> \
    <country> \
    <state/province> \
    <city> \
    <organization> \
    <ou> \
    <email> \
    [sans] \
    [options]
```

### Important ECC Curve IDs

|   ID | Curve       |
| ---: | ----------- |
| `19` | `secp256r1` |
| `20` | `secp384r1` |
| `21` | `secp521r1` |

> ⚠️ **Exam Trap:** `21` corresponds to **secp521r1**, not secp512r1.

---

# 13. Importing Certificates

A certificate does not necessarily need to originate from a FortiGate-generated CSR.

You can import externally generated certificates.

Common scenarios:

* Wildcard certificate
* Certificate generated by another PKI system
* Existing organizational certificate
* Certificate shared between multiple services/devices

Example wildcard:

```text
*.example.com
```

Can potentially be used for:

```text
fgt.example.com
faz.example.com
fmg.example.com
```

provided the certificate's scope and deployment requirements are appropriate.

---

# 14. Certificate Import Types

FortiGate supports importing:

```text
Import
├── Local Certificate
├── CA Certificate
├── Remote Certificate
└── CRL
```

### CLI Example

```bash
execute vpn certificate local import tftp \
    <file_name> \
    <server_address> \
    <cert_type> \
    [password]
```

---

# 15. Certificate + Private Key

When importing a certificate that was **not generated by the FortiGate CSR process**, the private key may also need to be imported.

Common formats:

### PKCS#12

```text
.p12
.pfx
```

A PKCS#12 bundle can contain:

```text
Certificate
+
Private Key
+
Certificate Chain
```

It is normally password protected.

---

### Separate Certificate + Key

Common examples:

```text
certificate.cer
private-key.pem
```

The private key must be protected.

---

# 16. Certificate Visibility and VDOM

Certificate visibility depends on where the certificate is uploaded.

```text
Certificate uploaded to VDOM
        │
        └── Accessible only by that VDOM
```

```text
Certificate uploaded to Global VDOM
        │
        └── Globally accessible
            by VDOMs
```

### Memory Rule

```text
VDOM Certificate
      ↓
VDOM Scope

Global Certificate
      ↓
Global Scope
```

---

# 17. Signed Certificate Generated from FortiGate CSR

When FortiGate generates a CSR:

```text
FortiGate
   │
   ├── Private Key → remains on FortiGate
   │
   └── CSR → CA
                │
                ▼
             Signed Cert
```

The CA returns the **signed certificate**, not the private key.

Therefore, the signed certificate can be imported back into FortiGate.

---

# 18. ACME — Automated Certificate Management Environment

**ACME** is defined by:

```text
RFC 8555
```

It allows automated certificate enrollment and management.

A major public CA using ACME is:

```text
Let's Encrypt
```

[Let's Encrypt](https://letsencrypt.org/?utm_source=chatgpt.com)

FortiGate can use ACME-managed certificates for services such as secure administrative access.

---

# 19. ACME Requirements

For public ACME enrollment, the FortiGate generally needs:

* Public IP address
* Public DNS hostname
* DNS hostname resolving to the public IP
* Public-facing ACME interface
* Ability to receive ACME validation traffic

Example:

```text
test.example.com
       │
       ▼
 Public IP
       │
       ▼
 FortiGate
```

---

# 20. ACME Interface Requirements

The configured ACME interface must be reachable from the Internet for the required validation process.

Avoid conflicting:

* VIPs
* Port forwarding
* NAT rules

on the required validation ports.

In particular, ensure that HTTP/HTTPS validation traffic is not intercepted or redirected in a way that prevents ACME validation.

---

# 21. ACME SAN Behavior

For FortiGate ACME certificates:

```text
SAN = FortiGate DNS hostname
```

The SAN is automatically populated from the configured ACME hostname.

Important limitations from the notes:

* SAN is automatically populated
* It cannot be manually edited
* Wildcard SANs are not supported in this workflow
* Multiple SANs cannot be added

---

# 22. Configure ACME Certificate

Example:

```bash
config vpn certificate local
    edit "acme-test"
        set enroll-protocol acme2
        set acme-domain "test.example.com"
        set acme-email "admin@example.com"
    next
end
```

Check certificate details:

```bash
get vpn certificate local details acme-test
```

Check ACME status:

```bash
diagnose sys acme status-full test.example.com
```

---

# 23. Test ACME Connectivity

Before troubleshooting ACME enrollment, verify reachability to the ACME service.

Example:

```bash
execute ping acme-v02.api.letsencrypt.org
```

> ⚠️ **Operational Tip:** If a proxy, firewall, DNS security system, or upstream filtering device blocks ACME endpoints, certificate enrollment can fail.

---

# 24. Use ACME Certificate for Admin GUI

Once the certificate has been successfully enrolled:

```bash
config system global
    set admin-server-cert "acme-test"
end
```

Flow:

```text
Internet Client
      │
      │ HTTPS
      ▼
FortiGate
      │
      └── ACME Certificate
              │
              └── Trusted CA
```

This removes the need to use the default/self-signed certificate for administrative HTTPS access.

---

# 25. Secure Administrative GUI

You can assign a certificate to the FortiGate HTTPS administrative interface.

```bash
config system global
    set admin-server-cert "fortisslvpndemo"
end
```

Recommended architecture:

```text
Administrator
      │
      │ HTTPS
      ▼
FortiGate
      │
      └── Trusted Server Certificate
```

### Best Practice

Use a certificate whose:

```text
CN / SAN
     ↓
matches
     ↓
FQDN used by administrator
```

Example:

```text
https://fgt.example.com
```

Certificate:

```text
SAN = fgt.example.com
```

---

# 26. SSH Certificate Authentication

FortiGate can also use certificates for administrator SSH authentication.

First import the remote certificate:

```bash
execute vpn certificate remote import tftp \
    certificate.pem \
    192.168.20.200
```

Then configure the remote certificate:

```bash
config certificate remote
    edit "REMOTE_Cert_1"
    next
end
```

Assign it to the administrator:

```bash
config system admin
    edit "admin1"
        set accprofile "prof_admin"
        set vdom "root"
        set ssh-certificate "REMOTE_Cert_1"
    next
end
```

---

# 27. SSH Certificate Authentication Flow

```text
Linux Client
     │
     │ SSH + Client Certificate
     ▼
FortiGate
     │
     ├── Validate Certificate
     │
     ├── Match configured Remote Certificate
     │
     └── Map to Administrator
              │
              ▼
         Access Granted
```

Example client:

```bash
ssh -i certificate-private.pem \
    admin1@192.168.20.200
```

### Important

The client keeps:

```text
Private Key
```

while FortiGate needs the corresponding:

```text
Public Certificate
```

for authentication/trust.

---

# 28. Certificate Authentication — Key Concept

```text
Client
├── Private Key 🔐
└── Certificate 📜
          │
          ▼
      FortiGate
          │
          └── Trusted Remote Certificate
```

The private key should **never be uploaded to FortiGate merely for validating the client's certificate identity**.

---

# 29. REST API Test Environment

A REST API administrator can be created under:

```text
System
└── Administrators
    └── New REST API User
```

Example:

```text
Username: api-test
Profile: RW
```

The API token can then be used as a bearer token.

Example API endpoint:

```text
/api/v2/cmdb/firewall/policy/1
```

Conceptual request:

```http
GET /api/v2/cmdb/firewall/policy/1
Authorization: Bearer <API-TOKEN>
```

Expected successful HTTP status:

```text
200 OK
```

> 🔐 **Security Tip:** Treat REST API tokens like passwords. Never place real API tokens in public GitHub repositories, screenshots, documentation, or scripts.

---

# 30. HTTPS Daemon Troubleshooting

For troubleshooting FortiGate HTTPS administrative access:

```bash
diagnose debug reset
diagnose debug enable
diagnose debug application httpsd -1
```

Stop debugging:

```bash
diagnose debug disable
```

### Debug Flow

```text
Client
  │
  │ HTTPS
  ▼
httpsd
  │
  ├── TLS negotiation
  ├── Certificate selection
  ├── Authentication
  └── HTTP processing
```

Useful when investigating:

* HTTPS login problems
* Certificate selection
* TLS negotiation
* Administrative GUI issues

---

# 31. Certificate Management Protocols

Important protocols/concepts:

| Technology | Purpose                           |
| ---------- | --------------------------------- |
| **CSR**    | Request certificate from CA       |
| **CMP**    | Certificate management/enrollment |
| **ACME**   | Automated certificate management  |
| **OCSP**   | Online certificate status         |
| **CRL**    | Certificate revocation list       |
| **PKI**    | Overall trust infrastructure      |

---

# 32. CRL vs OCSP

### CRL

Certificate Revocation List:

```text
CA
 │
 └── CRL
      ├── Cert A → revoked
      ├── Cert B → revoked
      └── Cert C → revoked
```

The client downloads a list of revoked certificates.

### OCSP

Online Certificate Status Protocol:

```text
FortiGate
    │
    │ "Is this certificate valid?"
    ▼
OCSP Responder
    │
    └── Good / Revoked / Unknown
```

### Memory Trick

```text
CRL  → Download the list
OCSP → Ask the CA/status responder
```

---

# 33. Certificate Chain

A typical certificate chain:

```text
                    Root CA
                       │
                 ┌─────┴─────┐
                 │           │
          Intermediate CA    ...
                 │
                 ▼
          Server Certificate
                 │
                 ▼
             FortiGate
```

Trust is established by validating the chain back to a trusted CA.

---

# 34. Certificate Validation

When FortiGate validates a certificate, important checks may include:

```text
Certificate
     │
     ├── Validity Period
     ├── Subject / SAN
     ├── Issuer
     ├── Signature
     ├── Certificate Chain
     ├── Revocation Status
     └── Trusted CA
```

### Common Failure Reasons

* Expired certificate
* Not-yet-valid certificate
* Wrong SAN
* Unknown CA
* Broken certificate chain
* Revoked certificate
* Missing intermediate CA
* Incorrect private key
* Unsupported algorithm
* Incorrect hostname

---

# 35. Certificate + Private Key Relationship

One of the most important PKI concepts:

```text
Public Certificate
       │
       │ corresponds to
       ▼
Private Key
```

The certificate contains the public key.

The private key is secret.

```text
Public Key
   └── Can be distributed

Private Key
   └── MUST remain secret
```

If the private key is compromised:

```text
Certificate Security
        ↓
       ❌
```

---

# 36. Wildcard Certificates

Example:

```text
*.example.com
```

Can cover multiple hosts such as:

```text
fgt.example.com
faz.example.com
fmg.example.com
```

This can be useful when the same certificate needs to be deployed across multiple devices.

### Important

Wildcard certificates do **not** mean:

```text
example.com
```

is automatically covered in every certificate validation context.

Always verify the actual SANs and certificate scope.

---

# 37. FortiGate Certificate CLI Map

```text
Certificate
│
├── Local
│   ├── RSA
│   ├── EC
│   ├── CMP
│   ├── ACME
│   └── Import
│
├── CA
│   └── Trust
│
├── Remote
│   └── Remote identity
│
├── CRL
│   └── Revocation
│
└── OCSP
    └── Online status
```

---

# 38. Useful CLI Commands

### Generate RSA

```bash
execute vpn certificate local generate rsa ...
```

### Generate ECC

```bash
execute vpn certificate local generate ec ...
```

### Generate CMP Certificate

```bash
execute vpn certificate local generate cmp ...
```

### Generate Default SSL CA

```bash
execute vpn certificate local generate default-ssl-ca
```

### Import Certificate

```bash
execute vpn certificate local import tftp ...
```

### Import Remote Certificate

```bash
execute vpn certificate remote import tftp ...
```

### Display ACME Details

```bash
get vpn certificate local details acme-test
```

### ACME Troubleshooting

```bash
diagnose sys acme status-full test.example.com
```

### HTTPS Daemon Debug

```bash
diagnose debug reset
diagnose debug enable
diagnose debug application httpsd -1
diagnose debug disable
```

---

# 39. Certificate Deployment Decision Tree

```text
Need certificate on FortiGate?
          │
          ▼
Does FortiGate need to identify itself?
          │
       ┌──┴──┐
      YES    NO
       │      │
       ▼      ▼
    LOCAL   Remote/CA
       │
       ▼
Need trusted public certificate?
       │
      YES
       │
       ▼
     ACME / CA
       │
       ▼
Need automated enrollment?
       │
   ┌───┴────┐
  YES       NO
   │         │
   ▼         ▼
 ACME/CMP    CSR
               │
               ▼
              CA
               │
               ▼
          Signed Certificate
```

---

# 40. NSE Exam Traps ⚠️

> [!IMPORTANT]
> **CSR ≠ Certificate.**
> CSR is a request sent to the CA; the CA signs it and returns a certificate.

> [!IMPORTANT]
> **CSR does not contain the private key.**

> [!IMPORTANT]
> A **local certificate** is used when FortiGate needs to identify itself.

> [!IMPORTANT]
> A **remote certificate** contains the remote entity's public certificate; FortiGate does not need the remote private key for this purpose.

> [!TIP]
> `*.example.com` is a wildcard certificate and can cover multiple subdomains depending on its SAN/scope.

> [!IMPORTANT]
> **CRL = list-based revocation.**
> **OCSP = online certificate status query.**

> [!IMPORTANT]
> ACME is standardized by **RFC 8555**.

> [!WARNING]
> ACME public certificate enrollment requires appropriate Internet reachability and DNS configuration.

> [!TIP]
> For administrative HTTPS, the certificate's hostname/SAN should match the FQDN administrators use to access FortiGate.

> [!IMPORTANT]
> The private key must remain protected. A certificate alone cannot substitute for possession of the corresponding private key where private-key proof is required.

> [!TIP]
> Certificate uploaded to a VDOM → VDOM scope.
> Certificate uploaded globally → globally accessible according to FortiOS certificate scope.

---

# 41. Quick Comparison

| Feature                  | Local Cert |             CA Cert | Remote Cert |
| ------------------------ | ---------: | ------------------: | ----------: |
| Represents FortiGate     |          ✅ |                   ❌ |           ❌ |
| Represents remote entity |          ❌ |                   ❌ |           ✅ |
| Establishes CA trust     |          ❌ |                   ✅ |           ❌ |
| Requires private key     |    Usually |                   ❌ |           ❌ |
| Admin HTTPS              |          ✅ |                   ❌ |           ❌ |
| SSL VPN server identity  |          ✅ |                   ❌ |           ❌ |
| Remote authentication    |          ❌ | ✅/context-dependent |           ✅ |
| SAML trust               |   Possible |            Possible |           ✅ |

---

# 42. Final Mental Model 🧠

```text
                         PKI
                          │
              ┌───────────┴───────────┐
              │                       │
             CA                     Entity
              │                       │
              │                 Public / Private Key
              │                       │
              ▼                       ▼
             CSR ◄────────────── FortiGate
              │
              │ Sign
              ▼
        Certificate
              │
       ┌──────┼────────┐
       │      │        │
     HTTPS   SSL VPN   Other TLS
       │
       ▼
    FortiGate Identity
```

---

# 🔥 43. One-Minute NSE Revision

```text
CSR
↓
Request certificate from CA

LOCAL CERT
↓
FortiGate identifies itself

CA CERT
↓
Trust CA / certificate chain

REMOTE CERT
↓
Identify/trust remote entity

CRL
↓
Revocation list

OCSP
↓
Online revocation/status check

CMP
↓
Certificate management/enrollment protocol

ACME
↓
Automated certificate management

Let's Encrypt
↓
Public ACME CA

RSA / ECC
↓
Public-key cryptography

Private Key
↓
KEEP SECRET 🔐

SAN
↓
Hostname identity

Wildcard
↓
*.example.com

admin-server-cert
↓
Certificate used by FortiGate HTTPS GUI
```

---

## 📌 SheynShield Security Rule

> **A certificate proves an identity through a trusted PKI chain; the private key proves possession of that identity.**

**SheynShield | Security & Design Knowledge Base**
**Engineering Secure Networks**

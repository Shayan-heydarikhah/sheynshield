# 🔐 FortiGate Certificates & PKI Checklist

> **FortiOS 7.2.x — Certificate Management, PKI, CSR, CA, Local & Remote Certificates, ACME, Let's Encrypt, CRL, OCSP, SSH Certificate Authentication & HTTPS Administration**
>
> **SheynShield | Engineering Secure Networks**

---

## 📋 Table of Contents

* [1. PKI Fundamentals Checklist](#1-pki-fundamentals-checklist)
* [2. Certificate Types Checklist](#2-certificate-types-checklist)
* [3. Local Certificate Checklist](#3-local-certificate-checklist)
* [4. CA Certificate Checklist](#4-ca-certificate-checklist)
* [5. Remote Certificate Checklist](#5-remote-certificate-checklist)
* [6. CSR Checklist](#6-csr-checklist)
* [7. RSA Certificate Checklist](#7-rsa-certificate-checklist)
* [8. ECC Certificate Checklist](#8-ecc-certificate-checklist)
* [9. Certificate Import Checklist](#9-certificate-import-checklist)
* [10. Private Key & PKCS#12 Checklist](#10-private-key--pkcs12-checklist)
* [11. Certificate Chain Checklist](#11-certificate-chain-checklist)
* [12. Certificate Validation Checklist](#12-certificate-validation-checklist)
* [13. Certificate Scope & VDOM Checklist](#13-certificate-scope--vdom-checklist)
* [14. ACME & Let's Encrypt Checklist](#14-acme--lets-encrypt-checklist)
* [15. ACME DNS & Connectivity Checklist](#15-acme-dns--connectivity-checklist)
* [16. ACME Certificate Deployment Checklist](#16-acme-certificate-deployment-checklist)
* [17. HTTPS Administrative Certificate Checklist](#17-https-administrative-certificate-checklist)
* [18. SSH Certificate Authentication Checklist](#18-ssh-certificate-authentication-checklist)
* [19. SAML Certificate Checklist](#19-saml-certificate-checklist)
* [20. CRL & OCSP Checklist](#20-crl--ocsp-checklist)
* [21. Certificate Security Hardening Checklist](#21-certificate-security-hardening-checklist)
* [22. Certificate Troubleshooting Checklist](#22-certificate-troubleshooting-checklist)
* [23. Useful FortiGate CLI](#23-useful-fortigate-cli)
* [24. Certificate Decision Tree](#24-certificate-decision-tree)
* [25. NSE Exam Traps](#25-nse-exam-traps)
* [26. Certificate Quick Comparison](#26-certificate-quick-comparison)
* [27. One-Minute NSE Revision](#27-one-minute-nse-revision)
* [28. SheynShield Golden Rules](#28-sheynshield-golden-rules)
* [29. Final PKI Architecture](#29-final-pki-architecture)

---

# 1. PKI Fundamentals Checklist

## Public Key Infrastructure

* [ ] Understand the purpose of PKI.
* [ ] Understand certificate-based identity.
* [ ] Understand certificate-based trust.
* [ ] Understand public/private key relationships.
* [ ] Understand certificate signing.
* [ ] Understand certificate validation.
* [ ] Understand certificate revocation.
* [ ] Understand certificate chains.
* [ ] Understand CA trust anchors.
* [ ] Understand the difference between authentication and encryption.

### PKI Flow

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

### Core PKI Workflow

```text
FortiGate
   │
   ├── Generate Private Key
   │
   ├── Generate Public Key
   │
   └── Generate CSR
           │
           ▼
          CA
           │
      Validate + Sign
           │
           ▼
    Signed Certificate
           │
           ▼
       FortiGate
```

---

# 2. Certificate Types Checklist

Verify that you understand the role of each certificate object.

| Certificate Object     | Primary Purpose                    | Private Key on FortiGate |
| ---------------------- | ---------------------------------- | -----------------------: |
| **Local Certificate**  | FortiGate/local service identity   |                ✅ Usually |
| **CA Certificate**     | Establish CA trust                 |                        ❌ |
| **Remote Certificate** | Remote entity certificate/trust    |                        ❌ |
| **CRL**                | Certificate revocation information |                        ❌ |
| **OCSP**               | Online certificate status checking |                        ❌ |

### Memory Model

* [ ] `LOCAL` → **I am FortiGate**
* [ ] `CA` → **I trust this CA**
* [ ] `REMOTE` → **I trust/identify this remote entity**
* [ ] `CRL` → **These certificates are revoked**
* [ ] `OCSP` → **Ask whether this certificate is valid**

---

# 3. Local Certificate Checklist

A local certificate identifies FortiGate or a service hosted by FortiGate.

### Common Use Cases

* [ ] Administrative HTTPS GUI.
* [ ] SSL VPN.
* [ ] Virtual Server / Load Balancing.
* [ ] TLS services.
* [ ] Other services where FortiGate presents its identity.

### Local Certificate Flow

```text
Client
   │
   │ TLS
   ▼
FortiGate
   │
   └── Presents Local Certificate
```

### Verify

* [ ] Certificate is intended for the correct service.
* [ ] Certificate is not expired.
* [ ] SAN contains the required hostname.
* [ ] Private key matches the certificate.
* [ ] Certificate chain is complete.
* [ ] Trusted CA is available to clients.
* [ ] Correct certificate is assigned to the service.

---

# 4. CA Certificate Checklist

A CA certificate establishes trust for certificates issued by that CA.

### Typical Chain

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

### Checklist

* [ ] Import the required root CA.
* [ ] Import required intermediate CA certificates.
* [ ] Verify certificate validity.
* [ ] Verify issuer relationships.
* [ ] Verify the chain is complete.
* [ ] Remove obsolete/untrusted CA certificates.
* [ ] Avoid blindly trusting unknown CAs.
* [ ] Document why each custom CA is trusted.

---

# 5. Remote Certificate Checklist

Remote certificates contain the public certificate of a remote entity.

### Checklist

* [ ] Determine which remote entity is being trusted.
* [ ] Obtain the remote public certificate.
* [ ] Verify the certificate issuer.
* [ ] Verify certificate validity.
* [ ] Verify the certificate chain where applicable.
* [ ] Import only the public certificate.
* [ ] Do **not** import the remote private key when it is not required.
* [ ] Assign the remote certificate to the appropriate authentication/trust function.

### Example Use Cases

* [ ] Certificate-based authentication.
* [ ] SAML trust.
* [ ] Security Fabric trust.
* [ ] Remote identity validation.
* [ ] Certificate inspection/validation.

---

# 6. CSR Checklist

A **CSR — Certificate Signing Request** is a request sent to a CA for certificate issuance.

### Before Generating CSR

* [ ] Define the required hostname/FQDN.
* [ ] Define Subject information.
* [ ] Define SAN requirements.
* [ ] Select RSA or ECC as appropriate.
* [ ] Select an appropriate key size/curve.
* [ ] Verify organizational certificate requirements.
* [ ] Confirm the CA requirements.

### CSR Contains

* [ ] Subject information.
* [ ] Public key.
* [ ] Organization information where applicable.
* [ ] Country where applicable.
* [ ] State/Province where applicable.
* [ ] City where applicable.
* [ ] Organizational Unit where applicable.
* [ ] Email where applicable.
* [ ] SANs where required.

### Critical Security Check

* [ ] Confirm that the CSR does **not** contain the private key.
* [ ] Confirm that the private key remains protected by the entity generating the CSR.

```text
Private Key
     │
     └── KEEP SECRET 🔐

CSR
     │
     └── Send to CA

CA
     │
     └── Signs CSR

Certificate
     │
     └── Return to FortiGate
```

> [!IMPORTANT]
> **CSR ≠ Certificate.** A CSR is a certificate request. The CA signs the request and issues the certificate.

---

# 7. RSA Certificate Checklist

FortiGate can generate RSA-based certificates.

### CLI

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

### Checklist

* [ ] Select an appropriate RSA key size.
* [ ] Define the correct Subject.
* [ ] Define required SANs.
* [ ] Verify hostname requirements.
* [ ] Protect the generated private key.
* [ ] Submit CSR to the appropriate CA if required.
* [ ] Import the signed certificate.
* [ ] Validate the resulting certificate chain.

---

# 8. ECC Certificate Checklist

FortiGate supports elliptic-curve certificate generation.

### CLI

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

### Important Curve IDs

|   ID | Curve       |
| ---: | ----------- |
| `19` | `secp256r1` |
| `20` | `secp384r1` |
| `21` | `secp521r1` |

### NSE Exam Checklist

* [ ] Memorize `19 → secp256r1`.
* [ ] Memorize `20 → secp384r1`.
* [ ] Memorize `21 → secp521r1`.
* [ ] Do not confuse `secp521r1` with `secp512r1`.

> [!WARNING]
> **Exam Trap:** Curve ID `21` corresponds to **secp521r1**.

---

# 9. Certificate Import Checklist

FortiGate can import certificates generated outside FortiGate.

### Possible Sources

* [ ] Enterprise PKI.
* [ ] Public CA.
* [ ] Wildcard certificate.
* [ ] Certificate generated on another system.
* [ ] Existing organizational certificate.
* [ ] Certificate shared across appropriately designed services.

### Import Types

```text
Import
├── Local Certificate
├── CA Certificate
├── Remote Certificate
└── CRL
```

### CLI

```bash
execute vpn certificate local import tftp \
    <file_name> \
    <server_address> \
    <cert_type> \
    [password]
```

### Validation

* [ ] Correct certificate type selected.
* [ ] Certificate format supported.
* [ ] Private key is available when required.
* [ ] Certificate and private key match.
* [ ] Certificate chain is available.
* [ ] Certificate password is known when required.
* [ ] Certificate scope is correct.

---

# 10. Private Key & PKCS#12 Checklist

When importing a certificate that was not generated from a FortiGate CSR, the private key may also need to be imported.

## PKCS#12

Common extensions:

```text
.p12
.pfx
```

A PKCS#12 bundle may contain:

```text
Certificate
+
Private Key
+
Certificate Chain
```

### Checklist

* [ ] Verify the PKCS#12 password.
* [ ] Protect the PKCS#12 file.
* [ ] Protect the private key.
* [ ] Verify the certificate matches the private key.
* [ ] Verify the certificate chain.
* [ ] Remove temporary certificate files from insecure locations.
* [ ] Do not commit `.p12`, `.pfx`, or private keys to Git repositories.

## Separate Certificate + Key

Example:

```text
certificate.cer
private-key.pem
```

### Security

* [ ] Certificate may be distributed according to its intended use.
* [ ] Private key must remain secret.
* [ ] Private key must have appropriate filesystem/secret-store protection.

> [!CAUTION]
> **A leaked private key can compromise the identity represented by the corresponding certificate.**

---

# 11. Certificate Chain Checklist

Verify the complete trust chain.

```text
Root CA
   │
   ▼
Intermediate CA
   │
   ▼
Server Certificate
   │
   ▼
FortiGate
```

### Checklist

* [ ] Root CA is trusted.
* [ ] Intermediate CA is available.
* [ ] Server/local certificate is valid.
* [ ] Issuer relationships are correct.
* [ ] Certificate chain is complete.
* [ ] No expired CA exists in the required chain.
* [ ] No revoked certificate exists in the chain.
* [ ] Client trusts the appropriate root CA.

### Common Chain Problem

```text
Client
   │
   ▼
Server Certificate
   │
   X
Missing Intermediate CA
```

Result:

```text
Certificate Validation Failure
```

---

# 12. Certificate Validation Checklist

When validating a certificate, check:

* [ ] Validity period.
* [ ] Not-before date.
* [ ] Not-after date.
* [ ] Subject.
* [ ] SAN.
* [ ] Issuer.
* [ ] Digital signature.
* [ ] Certificate chain.
* [ ] Trusted CA.
* [ ] Revocation status.
* [ ] Key usage.
* [ ] Extended key usage where applicable.
* [ ] Supported algorithm.
* [ ] Hostname match.

### Validation Model

```text
Certificate
     │
     ├── Validity
     ├── Subject / SAN
     ├── Issuer
     ├── Signature
     ├── Chain
     ├── Revocation
     └── Trusted CA
```

### Common Failure Causes

* [ ] Expired certificate.
* [ ] Certificate is not yet valid.
* [ ] Wrong SAN.
* [ ] Unknown CA.
* [ ] Missing intermediate CA.
* [ ] Broken certificate chain.
* [ ] Revoked certificate.
* [ ] Incorrect private key.
* [ ] Unsupported algorithm.
* [ ] Hostname mismatch.

---

# 13. Certificate Scope & VDOM Checklist

Certificate visibility can depend on where the certificate is uploaded.

### VDOM Scope

```text
Certificate
     │
     ▼
Specific VDOM
     │
     ▼
VDOM Scope
```

### Global Scope

```text
Certificate
     │
     ▼
Global VDOM
     │
     ▼
Global Availability
```

### Checklist

* [ ] Identify which VDOM owns the service.
* [ ] Upload the certificate to the appropriate scope.
* [ ] Verify certificate visibility.
* [ ] Verify the service can access the certificate.
* [ ] Avoid unnecessary global certificate deployment.
* [ ] Document certificate ownership.

> [!TIP]
> **VDOM certificate → VDOM scope.**
> **Global certificate → Global scope.**

---

# 14. ACME & Let's Encrypt Checklist

**ACME — Automated Certificate Management Environment** enables automated certificate enrollment and management.

```text
ACME
└── RFC 8555
```

A major public CA supporting ACME is:

**Let's Encrypt**

### Checklist

* [ ] Understand ACME.
* [ ] Understand certificate automation.
* [ ] Understand ACME validation.
* [ ] Configure the public DNS hostname.
* [ ] Configure the required FortiGate interface.
* [ ] Verify Internet reachability.
* [ ] Verify DNS resolution.
* [ ] Verify ACME validation traffic can reach FortiGate.
* [ ] Verify there is no conflicting NAT/VIP configuration.
* [ ] Verify certificate enrollment status.
* [ ] Verify certificate validity after enrollment.

---

# 15. ACME DNS & Connectivity Checklist

For public ACME enrollment, verify:

```text
FQDN
  │
  ▼
Public DNS
  │
  ▼
Public IP
  │
  ▼
FortiGate
```

### DNS

* [ ] Public hostname exists.
* [ ] Public DNS resolves correctly.
* [ ] DNS points to the expected public IP.
* [ ] No stale DNS record exists.
* [ ] FQDN matches the certificate requirement.

### Connectivity

* [ ] Required ACME validation traffic can reach FortiGate.
* [ ] Required interface is reachable from the Internet.
* [ ] No conflicting VIP exists.
* [ ] No conflicting port-forwarding rule exists.
* [ ] No NAT rule interferes with validation.
* [ ] HTTP/HTTPS redirection does not break validation.
* [ ] Upstream firewall allows required traffic.
* [ ] Proxy/security gateway does not block required ACME communication.

### Connectivity Test

```bash
execute ping acme-v02.api.letsencrypt.org
```

> [!WARNING]
> DNS resolution and Internet connectivity alone do not guarantee successful ACME validation. The validation path must also be reachable and correctly handled.

---

# 16. ACME Certificate Deployment Checklist

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

### Checklist

* [ ] Configure ACME enrollment.
* [ ] Configure the correct domain.
* [ ] Configure an operational email address where required.
* [ ] Verify public DNS.
* [ ] Verify Internet reachability.
* [ ] Verify ACME validation.
* [ ] Verify certificate enrollment.
* [ ] Verify certificate details.
* [ ] Verify certificate validity.
* [ ] Assign certificate to the intended FortiGate service.

### Certificate Details

```bash
get vpn certificate local details acme-test
```

### ACME Status

```bash
diagnose sys acme status-full test.example.com
```

---

# 17. HTTPS Administrative Certificate Checklist

The FortiGate administrative HTTPS interface can use a trusted server certificate.

### CLI

```bash
config system global
    set admin-server-cert "fortisslvpndemo"
end
```

### Recommended Design

```text
Administrator
      │
      │ HTTPS
      ▼
FortiGate
      │
      └── Trusted Server Certificate
```

### Certificate Validation

* [ ] Certificate is issued by a trusted CA.
* [ ] Certificate is not expired.
* [ ] SAN matches the administrator FQDN.
* [ ] Certificate chain is valid.
* [ ] Private key matches the certificate.
* [ ] Certificate is assigned to `admin-server-cert`.

### Example

Administrator accesses:

```text
https://fgt.example.com
```

Certificate should contain:

```text
SAN = fgt.example.com
```

> [!TIP]
> Prefer certificate identity matching the **FQDN administrators actually use** to access the FortiGate.

---

# 18. SSH Certificate Authentication Checklist

FortiGate can use certificates for administrator SSH authentication.

### Remote Certificate Import

```bash
execute vpn certificate remote import tftp \
    certificate.pem \
    192.168.20.200
```

### Configure Remote Certificate

```bash
config certificate remote
    edit "REMOTE_Cert_1"
    next
end
```

### Assign to Administrator

```bash
config system admin
    edit "admin1"
        set accprofile "prof_admin"
        set vdom "root"
        set ssh-certificate "REMOTE_Cert_1"
    next
end
```

### Checklist

* [ ] Generate or obtain the client certificate.
* [ ] Protect the client private key.
* [ ] Import the required remote certificate.
* [ ] Configure the remote certificate object.
* [ ] Assign the certificate to the administrator.
* [ ] Verify administrator profile.
* [ ] Verify VDOM scope.
* [ ] Verify SSH access is enabled.
* [ ] Test certificate authentication.

### Authentication Model

```text
SSH Client
    │
    ├── Private Key 🔐
    └── Certificate
           │
           ▼
       FortiGate
           │
           ├── Validate Certificate
           │
           ├── Match Remote Certificate
           │
           └── Map Administrator
```

### Security Rule

* [ ] Keep private key on the client.
* [ ] Store only the required public certificate on FortiGate.
* [ ] Protect the client private key with appropriate controls.

---

# 19. SAML Certificate Checklist

Certificate usage is important when FortiGate participates in SAML authentication.

### If FortiGate Acts as IdP

* [ ] Understand whether an SP certificate is required.
* [ ] Import the required remote certificate where appropriate.
* [ ] Verify SAML trust relationships.
* [ ] Verify certificate validity.
* [ ] Verify certificate chain where applicable.

### If FortiGate Acts as SP

* [ ] Identify the IdP certificate.
* [ ] Import the required remote certificate.
* [ ] Verify SAML configuration.
* [ ] Verify certificate trust.
* [ ] Verify certificate expiration.

### Mental Model

```text
FortiGate
   │
   ├── IdP
   │
   └── SP
```

Certificate requirements depend on the role FortiGate performs in the SAML relationship.

---

# 20. CRL & OCSP Checklist

## CRL

**Certificate Revocation List**

```text
CA
 │
 └── CRL
      ├── Certificate A → Revoked
      ├── Certificate B → Revoked
      └── Certificate C → Revoked
```

### CRL Checklist

* [ ] Understand that CRL is a list.
* [ ] Verify CRL source.
* [ ] Verify CRL validity.
* [ ] Verify certificate revocation status when required.
* [ ] Ensure required CRL information is available.

## OCSP

**Online Certificate Status Protocol**

```text
FortiGate
    │
    │ Certificate status query
    ▼
OCSP Responder
    │
    └── Good / Revoked / Unknown
```

### OCSP Checklist

* [ ] Understand online status checking.
* [ ] Identify the OCSP responder.
* [ ] Verify connectivity.
* [ ] Verify certificate status response.
* [ ] Understand `Good`, `Revoked`, and `Unknown`.

### Memory Trick

```text
CRL
↓
Download the list

OCSP
↓
Ask the responder
```

---

# 21. Certificate Security Hardening Checklist

## Private Key Protection

* [ ] Protect all private keys.
* [ ] Never publish private keys.
* [ ] Never commit private keys to GitHub.
* [ ] Never place private keys in public documentation.
* [ ] Never expose private keys in screenshots.
* [ ] Use secure secret storage.
* [ ] Restrict filesystem access.
* [ ] Protect backup copies.
* [ ] Revoke/replace certificates when private keys are compromised.

## Certificate Lifecycle

* [ ] Inventory certificates.
* [ ] Record certificate owner.
* [ ] Record certificate purpose.
* [ ] Record certificate expiration date.
* [ ] Monitor expiration.
* [ ] Define renewal procedures.
* [ ] Remove obsolete certificates.
* [ ] Review trusted CA certificates periodically.

## Hostname Security

* [ ] Use correct FQDN.
* [ ] Verify SAN.
* [ ] Avoid certificate hostname mismatches.
* [ ] Avoid unnecessary wildcard certificates.
* [ ] Verify wildcard scope before deployment.

## GitHub Security

* [ ] Never publish private keys.
* [ ] Never publish `.p12` files.
* [ ] Never publish `.pfx` files containing private keys.
* [ ] Never publish real API tokens alongside certificate examples.
* [ ] Never publish real passwords.
* [ ] Use placeholders in technical documentation.

Example:

```text
<PRIVATE-KEY>
<API-TOKEN>
<DOMAIN>
<USERNAME>
```

---

# 22. Certificate Troubleshooting Checklist

When certificate authentication or TLS fails, troubleshoot systematically.

## Step 1 — Certificate Validity

* [ ] Is the certificate expired?
* [ ] Is the certificate not-yet-valid?
* [ ] Is the system clock correct?

## Step 2 — Hostname

* [ ] Does SAN contain the requested hostname?
* [ ] Is the administrator using the correct FQDN?
* [ ] Is the certificate issued for the correct service?

## Step 3 — Trust

* [ ] Does the client trust the CA?
* [ ] Is the root CA trusted?
* [ ] Is the intermediate CA available?
* [ ] Is the chain complete?

## Step 4 — Key Pair

* [ ] Does the private key match the certificate?
* [ ] Is the private key available?
* [ ] Is the private key protected?
* [ ] Was the correct certificate imported?

## Step 5 — Revocation

* [ ] Is the certificate revoked?
* [ ] Is CRL checking involved?
* [ ] Is OCSP checking involved?
* [ ] Is the revocation responder reachable?

## Step 6 — FortiGate Scope

* [ ] Is the certificate uploaded to the correct VDOM?
* [ ] Is the certificate globally available where required?
* [ ] Does the target service have access to the certificate?

## Step 7 — Service Configuration

* [ ] Is the correct certificate assigned?
* [ ] Is `admin-server-cert` configured correctly where applicable?
* [ ] Is HTTPS administrative access enabled?
* [ ] Is SSH certificate authentication configured correctly?

---

# 23. Useful FortiGate CLI

## Generate RSA Certificate

```bash
execute vpn certificate local generate rsa ...
```

## Generate ECC Certificate

```bash
execute vpn certificate local generate ec ...
```

## Generate CMP Certificate

```bash
execute vpn certificate local generate cmp ...
```

## Generate Default SSL CA

```bash
execute vpn certificate local generate default-ssl-ca
```

## Generate Default SSL Key Certificates

```bash
execute vpn certificate local generate default-ssl-key-certs
```

## Generate Default SSL Server Key

```bash
execute vpn certificate local generate default-ssl-serv-key
```

## Import Local Certificate

```bash
execute vpn certificate local import tftp ...
```

## Import Remote Certificate

```bash
execute vpn certificate remote import tftp ...
```

## Check ACME Certificate

```bash
get vpn certificate local details acme-test
```

## Check ACME Status

```bash
diagnose sys acme status-full test.example.com
```

## Test ACME Connectivity

```bash
execute ping acme-v02.api.letsencrypt.org
```

---

# 24. Certificate Decision Tree

```text
Need a certificate on FortiGate?
              │
              ▼
Does FortiGate need to identify itself?
              │
         ┌────┴────┐
        YES        NO
         │          │
         ▼          ▼
      LOCAL      CA / REMOTE
         │
         ▼
Need a trusted public certificate?
         │
        YES
         │
         ▼
   Public CA / ACME
         │
         ▼
Need automated enrollment?
         │
    ┌────┴────┐
   YES        NO
    │          │
    ▼          ▼
  ACME/CMP     CSR
                │
                ▼
                CA
                │
                ▼
       Signed Certificate
                │
                ▼
            FortiGate
```

---

# 25. NSE Exam Traps

> [!IMPORTANT]
> **CSR is not a certificate.**
> CSR = certificate signing request.

> [!IMPORTANT]
> **CSR does not contain the private key.**

> [!IMPORTANT]
> **Local Certificate** is primarily used when FortiGate needs to identify itself.

> [!IMPORTANT]
> **CA Certificate** is used to establish trust in a CA/certificate chain.

> [!IMPORTANT]
> **Remote Certificate** represents the public certificate of a remote entity.

> [!IMPORTANT]
> The remote entity's private key is not required merely for FortiGate to validate/trust its public certificate.

> [!WARNING]
> ECC curve ID `21` = **secp521r1**.

> [!IMPORTANT]
> **CRL = list-based revocation.**

> [!IMPORTANT]
> **OCSP = online certificate status query.**

> [!IMPORTANT]
> **ACME = Automated Certificate Management Environment.**

> [!IMPORTANT]
> ACME is standardized by **RFC 8555**.

> [!WARNING]
> Public ACME enrollment requires correct DNS and appropriate Internet reachability.

> [!TIP]
> Administrative HTTPS certificates should match the FQDN administrators use to access FortiGate.

> [!IMPORTANT]
> A certificate contains a public key; the corresponding private key must remain protected.

> [!TIP]
> `*.example.com` is a wildcard certificate and can cover applicable subdomains within its certificate scope.

---

# 26. Certificate Quick Comparison

| Feature                        | Local Cert |  CA Cert |     Remote Cert     |
| ------------------------------ | :--------: | :------: | :-----------------: |
| Represents FortiGate           |      ✅     |     ❌    |          ❌          |
| Represents remote entity       |      ❌     |     ❌    |          ✅          |
| Establishes CA trust           |      ❌     |     ✅    |          ❌          |
| Usually requires private key   |      ✅     |     ❌    |          ❌          |
| Admin HTTPS                    |      ✅     |     ❌    |          ❌          |
| SSL VPN server identity        |      ✅     |     ❌    |          ❌          |
| Certificate-based remote trust |  Possible  | Possible |          ✅          |
| SAML-related use               |  Possible  | Possible | ✅/context-dependent |

### Memory

```text
LOCAL
↓
FortiGate Identity

CA
↓
Trust

REMOTE
↓
Remote Identity / Trust
```

---

# 27. One-Minute NSE Revision

```text
CSR
↓
Request certificate from CA

LOCAL CERT
↓
FortiGate identifies itself

CA CERT
↓
Establish CA trust

REMOTE CERT
↓
Remote certificate / trust

CRL
↓
Revocation list

OCSP
↓
Online status query

CMP
↓
Certificate management/enrollment

ACME
↓
Automated certificate management

Let's Encrypt
↓
Public ACME CA

RSA / ECC
↓
Public-key cryptography

PRIVATE KEY
↓
KEEP SECRET 🔐

SAN
↓
Certificate hostname identity

WILDCARD
↓
*.example.com

admin-server-cert
↓
Certificate for FortiGate administrative HTTPS
```

---

# 28. SheynShield Golden Rules

> [!IMPORTANT]
>
> ### 1. Certificate ≠ Private Key
>
> ```text
> Certificate
>     ↓
> Public Identity
>
> Private Key
>     ↓
> Proof of Possession
> ```

---

> [!IMPORTANT]
>
> ### 2. CSR ≠ Certificate
>
> ```text
> CSR
> ↓
> Request
>
> CA
> ↓
> Sign
>
> Certificate
> ↓
> Identity
> ```

---

> [!IMPORTANT]
>
> ### 3. Local ≠ Remote ≠ CA
>
> ```text
> LOCAL
> ↓
> "I am FortiGate"
>
> CA
> ↓
> "I trust this CA"
>
> REMOTE
> ↓
> "I trust/identify this remote entity"
> ```

---

> [!IMPORTANT]
>
> ### 4. Public Key Can Be Shared — Private Key Cannot
>
> ```text
> Public Key
> ↓
> Can be distributed
>
> Private Key
> ↓
> MUST remain secret
> ```

---

> [!IMPORTANT]
>
> ### 5. SAN Matters
>
> ```text
> Administrator
>       ↓
> https://fgt.example.com
>       ↓
> SAN = fgt.example.com
> ```
>
> Hostname mismatch can cause certificate validation failures.

---

> [!IMPORTANT]
>
> ### 6. CRL vs OCSP
>
> ```text
> CRL
> ↓
> Download the revocation list
>
> OCSP
> ↓
> Ask the responder
> ```

---

> [!IMPORTANT]
>
> ### 7. ACME Automates Certificate Management
>
> ```text
> DNS
> ↓
> Validation
> ↓
> ACME
> ↓
> Certificate
> ↓
> Automated Renewal
> ```

---

> [!WARNING]
>
> ### 8. Never Publish Private Keys
>
> Never put these in a public GitHub repository:
>
> ```text
> ❌ private-key.pem
> ❌ server.key
> ❌ certificate.p12
> ❌ certificate.pfx
> ❌ Real credentials
> ```
>
> Use:
>
> ```text
> <PRIVATE-KEY>
> <CERTIFICATE>
> <DOMAIN>
> ```

---

> [!IMPORTANT]
>
> ### 9. Certificate Scope Matters
>
> ```text
> VDOM Certificate
> ↓
> VDOM Scope
>
> Global Certificate
> ↓
> Global Scope
> ```

---

> [!TIP]
>
> ### 10. Certificate Security Is a Lifecycle
>
> ```text
> Generate
>    ↓
> Issue
>    ↓
> Deploy
>    ↓
> Validate
>    ↓
> Monitor
>    ↓
> Renew
>    ↓
> Revoke / Replace
> ```

---

# 29. Final PKI Architecture

A mature FortiGate PKI architecture can be represented as:

```text
                         PKI
                          │
             ┌────────────┴────────────┐
             │                         │
            CA                       Entity
             │                         │
             │                  Public / Private Key
             │                         │
             ▼                         ▼
            CSR ◄──────────────── FortiGate
             │
             │ Sign
             ▼
       Certificate
             │
       ┌─────┼─────────┐
       │     │         │
     HTTPS  SSL VPN   Other TLS
       │
       ▼
FortiGate Identity
```

## Enterprise Administrative Certificate Architecture

```text
                    Management Network
                           │
                           ▼
                       FortiGate
                           │
            ┌──────────────┼──────────────┐
            │              │              │
            ▼              ▼              ▼
           CA            ACME          Enterprise PKI
            │              │              │
            └──────────────┼──────────────┘
                           │
                           ▼
                    Local Certificate
                           │
             ┌─────────────┼─────────────┐
             │             │             │
             ▼             ▼             ▼
          HTTPS          SSL VPN      TLS Services
             │
             ▼
        Administrator
             │
             ▼
      Trusted Certificate
```

---

# 🛡️ Final Certificate Security Checklist

## PKI

* [ ] PKI architecture documented.
* [ ] Trusted CAs identified.
* [ ] Certificate ownership documented.
* [ ] Certificate lifecycle documented.

## Certificates

* [ ] Local certificates reviewed.
* [ ] CA certificates reviewed.
* [ ] Remote certificates reviewed.
* [ ] Expiration dates monitored.
* [ ] SANs verified.
* [ ] Certificate chains validated.

## Private Keys

* [ ] Private keys protected.
* [ ] Private keys not stored in Git.
* [ ] Private keys not exposed in documentation.
* [ ] Backup copies protected.
* [ ] Compromised keys replaced.

## ACME

* [ ] DNS validated.
* [ ] Public reachability validated.
* [ ] ACME validation path validated.
* [ ] Certificate enrollment verified.
* [ ] Renewal process monitored.

## Administrative HTTPS

* [ ] Trusted certificate configured.
* [ ] FQDN matches SAN.
* [ ] Certificate chain is valid.
* [ ] Certificate expiration is monitored.

## SSH

* [ ] SSH certificate authentication reviewed.
* [ ] Remote certificate configured where required.
* [ ] Private client key protected.
* [ ] Administrator profile follows least privilege.

## Revocation

* [ ] CRL requirements reviewed.
* [ ] OCSP requirements reviewed.
* [ ] Revocation responders reachable where required.

---

# 🧠 SheynShield Mental Model

```text
                         FORTIGATE PKI
                               │
        ┌──────────────────────┼──────────────────────┐
        │                      │                      │
     IDENTITY                TRUST                REVOCATION
        │                      │                      │
   Local Cert              CA Cert                 CRL
        │                      │                    OCSP
        │                      │
        └──────────────┬───────┘
                       │
                       ▼
                 Certificate
                       │
              ┌────────┴────────┐
              │                 │
            Public Key       Private Key
              │                 │
           Shareable        KEEP SECRET 🔐
              │                 │
              └────────┬────────┘
                       │
                       ▼
              FortiGate Services
                       │
        ┌──────────────┼──────────────┐
        │              │              │
      HTTPS         SSL VPN         SSH
        │              │              │
        └──────────────┼──────────────┘
                       │
                       ▼
                 Secure Identity
```

---

# 🔥 SheynShield Security Principle

> **A certificate establishes identity through a trusted PKI chain; possession of the corresponding private key provides proof of that identity.**

```text
Strong PKI
    +
Correct Certificate
    +
Valid Chain
    +
Correct SAN
    +
Private-Key Protection
    +
Revocation Checking
    +
Automated Renewal
    =
Secure Certificate Infrastructure
```

---

## 🔎 Keywords

`FortiGate certificates` · `FortiOS certificates` · `FortiGate PKI` · `FortiGate CSR` · `FortiGate certificate authority` · `FortiGate CA certificate` · `FortiGate local certificate` · `FortiGate remote certificate` · `FortiGate ACME` · `FortiGate Let's Encrypt` · `FortiGate HTTPS certificate` · `FortiGate admin certificate` · `FortiGate SSH certificate authentication` · `FortiGate SAML certificate` · `FortiGate CRL` · `FortiGate OCSP` · `FortiGate certificate chain` · `FortiGate certificate troubleshooting` · `FortiOS 7.2 certificate` · `Fortinet PKI` · `Fortinet NSE4` · `Fortinet NSE7` · `FortiGate security hardening`

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

**SheynShield | Security & Design Knowledge Base**

**Engineering Secure Networks**

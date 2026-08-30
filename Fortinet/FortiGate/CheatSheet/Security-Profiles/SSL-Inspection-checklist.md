# 🔐 FortiGate SSL/SSH Inspection Checklist

> **FortiOS 7.x | SSL/TLS Inspection | SSH Inspection | Deep Inspection | Certificate Inspection | HTTPS | SCP | SFTP | DLP | Antivirus | IPS**
>
> A practical **FortiGate SSL/SSH Inspection Checklist** for **NSE 4 / NSE 7 preparation, production deployment, security hardening, troubleshooting, and encrypted traffic inspection**.

---

## 📚 Table of Contents

* [1. SSL/SSH Inspection Fundamentals](#1-sslssh-inspection-fundamentals)
* [2. Certificate Inspection Checklist](#2-certificate-inspection-checklist)
* [3. Deep Inspection Checklist](#3-deep-inspection-checklist)
* [4. FortiGate CA Certificate Checklist](#4-fortigate-ca-certificate-checklist)
* [5. SNI Validation Checklist](#5-sni-validation-checklist)
* [6. TLS Security Checklist](#6-tls-security-checklist)
* [7. SSL Offloading Checklist](#7-ssl-offloading-checklist)
* [8. FTP/FTPS Inspection Checklist](#8-ftpftps-inspection-checklist)
* [9. SSH/SCP/SFTP Inspection Checklist](#9-sshs cpsftp-inspection-checklist)
* [10. SSH File Scanning Checklist](#10-ssh-file-scanning-checklist)
* [11. SSH Filter Checklist](#11-ssh-filter-checklist)
* [12. DLP + SSH Checklist](#12-dlp--ssh-checklist)
* [13. Antivirus + SSH Checklist](#13-antivirus--ssh-checklist)
* [14. Quarantine Checklist](#14-quarantine-checklist)
* [15. Firewall Policy Checklist](#15-firewall-policy-checklist)
* [16. Proxy Inspection Checklist](#16-proxy-inspection-checklist)
* [17. Troubleshooting Checklist](#17-troubleshooting-checklist)
* [18. Common Configuration Mistakes](#18-common-configuration-mistakes)
* [19. Production Hardening Checklist](#19-production-hardening-checklist)
* [20. NSE 4 / NSE 7 Exam Traps](#20-nse-4--nse-7-exam-traps)
* [21. Quick Reference](#21-quick-reference)
* [22. Final Validation Checklist](#22-final-validation-checklist)

---

# 1. SSL/SSH Inspection Fundamentals

## 🎯 Why SSL/SSH Inspection?

Encrypted traffic can hide malicious content from security controls.

Check which encrypted protocols require inspection:

* [ ] HTTPS
* [ ] SSL/TLS applications
* [ ] IMAPS
* [ ] POP3S
* [ ] FTPS
* [ ] SSH
* [ ] SCP
* [ ] SFTP

Without payload inspection:

```text
Encrypted Traffic
        ↓
FortiGate
        ↓
Security Profiles
        ↓
Limited Payload Visibility
```

With deep inspection:

```text
Encrypted Traffic
        ↓
TLS Decryption
        ↓
Clear-Text Inspection
        ↓
AV / IPS / DLP / Web Filter / Application Control
        ↓
Re-encryption
        ↓
Destination
```

### ✅ Architecture Checklist

* [ ] Identify encrypted protocols.
* [ ] Identify which applications require payload inspection.
* [ ] Determine whether certificate inspection is sufficient.
* [ ] Determine whether deep inspection is required.
* [ ] Identify endpoints that must trust the inspection CA.
* [ ] Identify applications that may not tolerate TLS interception.
* [ ] Plan exemptions before production deployment.
* [ ] Monitor resource consumption after enabling deep inspection.

---

# 2. Certificate Inspection Checklist

Certificate inspection provides visibility into the TLS handshake and certificate information without performing the same full payload decryption provided by deep inspection.

### 🔎 Certificate Inspection

* [ ] Certificate inspection is selected where payload inspection is unnecessary.
* [ ] Server certificate validity is evaluated.
* [ ] Certificate issuer is evaluated.
* [ ] Certificate chain is evaluated.
* [ ] Certificate expiration is evaluated.
* [ ] SNI information is considered where required.
* [ ] Certificate identity validation is configured appropriately.
* [ ] Applications requiring full payload inspection are not incorrectly placed behind certificate-only inspection.

### 🧠 Remember

```text
Certificate Inspection
        ↓
TLS Handshake
        ↓
Certificate / Identity Information
        ↓
No equivalent full payload visibility
```

### ⭐ NSE Memory Hook

> **Certificate Inspection = inspect certificate information.**

---

# 3. Deep Inspection Checklist

Deep inspection is used when FortiGate needs visibility into encrypted application payloads.

### 🔬 Deep Inspection

* [ ] Deep inspection is required by the security policy.
* [ ] Correct SSL/SSH inspection profile is attached.
* [ ] Appropriate inspection mode is configured.
* [ ] Inspection CA is selected.
* [ ] Managed clients trust the inspection CA.
* [ ] Business-critical applications have been tested.
* [ ] TLS compatibility has been validated.
* [ ] Required security profiles are attached.
* [ ] Logging is enabled.
* [ ] Resource impact has been evaluated.

### 🔄 Deep Inspection Flow

```text
Client
   ↓
Encrypted HTTPS
   ↓
FortiGate
   ↓
Decrypt
   ↓
Security Inspection
   ├── Antivirus
   ├── IPS
   ├── Web Filter
   ├── DLP
   └── Application Control
   ↓
Re-encrypt
   ↓
Server
```

### Production Checklist

* [ ] Do not blindly decrypt all traffic.
* [ ] Define inspection scope.
* [ ] Define trusted applications.
* [ ] Define privacy-sensitive exclusions where appropriate.
* [ ] Test certificate-pinning applications.
* [ ] Monitor CPU and memory.
* [ ] Monitor session behavior.

---

# 4. FortiGate CA Certificate Checklist

During deep inspection, FortiGate can act as a TLS interception point.

```text
Original Server Certificate
            ↓
        FortiGate
            ↓
     Re-signed Certificate
            ↓
          Client
```

### `Fortinet_CA_SSL`

* [ ] Correct FortiGate inspection CA is identified.
* [ ] CA certificate is exported when required.
* [ ] CA is installed into managed endpoint trust stores.
* [ ] Browser trust has been validated.
* [ ] Certificate chain has been tested.

### ⚠️ Critical Security Check

* [ ] **Never install `Fortinet_CA_Untrusted` as a trusted CA.**
* [ ] Verify endpoint trust stores after deployment.
* [ ] Remove unintended inspection CAs from unmanaged systems.

### Certificate Validation

```text
Client
  ↓
FortiGate-generated Certificate
  ↓
Trusted CA?
  ├── YES → Continue
  └── NO  → Certificate Warning / Failure
```

---

# 5. SNI Validation Checklist

**SNI = Server Name Indication**

SNI allows a TLS client to indicate the requested hostname during TLS negotiation.

Example:

```text
ClientHello
    |
    └── SNI = example.com
```

Certificate identity may contain:

```text
CN  = example.com
SAN = example.com
```

### SNI Checklist

* [ ] SNI behavior is understood.
* [ ] Certificate identity is validated where required.
* [ ] SAN/CN behavior is understood.
* [ ] SNI mismatch behavior has been tested.
* [ ] Strict validation is enabled only where compatible.
* [ ] Certificate-pinning applications have been tested.

### Mental Model

```text
SNI
 ↓
Certificate Identity
 ↓
Compare
 ↓
Allow / Block
```

---

# 6. TLS Security Checklist

### 🔐 Cryptographic Hardening

* [ ] Strong cryptography requirements have been evaluated.
* [ ] Legacy TLS versions have been identified.
* [ ] TLS 1.0 usage has been identified.
* [ ] TLS 1.1 usage has been identified.
* [ ] TLS 1.2 support has been verified.
* [ ] TLS 1.3 compatibility has been verified.
* [ ] Legacy applications have been tested before enforcement.

Example:

```cli
config system global
    set strong-crypto enable
end
```

### TLS Version Example

```cli
config firewall ssl-ssh-profile
    edit "ssl-test"
        config https
            set min-allowed-ssl-version tls-1.2
            set unsupported-ssl-version block
        end
    next
end
```

### Validation

```text
TLS 1.0
   ↓
Block

TLS 1.1
   ↓
Block

TLS 1.2
   ↓
Allow

TLS 1.3
   ↓
Allow*
```

> `*` TLS 1.3 behavior depends on the FortiOS release, inspection mode, platform, and application compatibility.

---

# 7. SSL Offloading Checklist

SSL offloading means another device performs TLS decryption before traffic reaches FortiGate.

```text
Client
   ↓
External TLS Device
   ↓
TLS Decryption
   ↓
Clear Text
   ↓
FortiGate
   ↓
Server
```

### Architecture Checklist

* [ ] Identify where TLS decryption occurs.
* [ ] Identify whether FortiGate receives encrypted or clear-text traffic.
* [ ] Identify whether an ADC/load balancer performs TLS termination.
* [ ] Configure protocol handling accordingly.
* [ ] Avoid assuming FortiGate receives the original TLS session.
* [ ] Validate protocol detection after TLS offloading.

### Key Rule

> **The FortiGate protocol configuration must match the actual traffic state.**

---

# 8. FTP/FTPS Inspection Checklist

For FTP/FTPS, TLS negotiation can change the protection state.

### Important Commands / Concepts

```text
AUTH TLS
    ↓
TLS Negotiation

PBSZ
    ↓
Protection Buffer Size

PROT
    ↓
Protection Level
```

### `PROT`

Common concepts:

```text
PROT P
→ Protected

PROT C
→ Clear
```

### Checklist

* [ ] FTP vs FTPS architecture is identified.
* [ ] TLS termination point is identified.
* [ ] `AUTH TLS` behavior is understood.
* [ ] `PBSZ` behavior is understood.
* [ ] `PROT` behavior is understood.
* [ ] SSL offloading configuration matches the actual topology.
* [ ] File inspection is tested.

---

# 9. SSH/SCP/SFTP Inspection Checklist

SSH-based file transfer can carry sensitive or malicious content.

```text
SSH
├── SCP
└── SFTP
```

### Checklist

* [ ] SSH traffic is identified.
* [ ] SCP traffic is identified.
* [ ] SFTP traffic is identified.
* [ ] SSH inspection is enabled where required.
* [ ] File scanning is enabled where required.
* [ ] Antivirus is attached.
* [ ] DLP is attached where required.
* [ ] SSH filter is configured.
* [ ] File logging is enabled.
* [ ] Blocked activity is logged.

### Security Stack

```text
SSH
 │
 ├── Protocol Inspection
 ├── File Extraction
 ├── Antivirus
 ├── DLP
 ├── SSH Filter
 └── Logging
```

---

# 10. SSH File Scanning Checklist

Example:

```cli
config firewall profile-protocol-options
    edit "proto-test"

        config ssh
            set options oversize clientcomfort servercomfort
            set comfort-interval 100
            set comfort-amount 100
            set oversize-limit 100
            set uncompressed-oversize-limit 100
            set uncompressed-nest-limit 2
            set scan-bzip2 enable
        end

    next
end
```

### Parameter Checklist

| Parameter                     | Validation                 |
| ----------------------------- | -------------------------- |
| `comfort-interval`            | [ ] Reviewed               |
| `comfort-amount`              | [ ] Reviewed               |
| `oversize-limit`              | [ ] Reviewed               |
| `uncompressed-oversize-limit` | [ ] Reviewed               |
| `uncompressed-nest-limit`     | [ ] Reviewed               |
| `scan-bzip2`                  | [ ] Enabled where required |

### Production Validation

* [ ] Large files have been tested.
* [ ] Compressed files have been tested.
* [ ] Nested archives have been tested.
* [ ] Oversize behavior is understood.
* [ ] Uncompressed size limits are understood.
* [ ] Inspection resource consumption is monitored.

---

# 11. SSH Filter Checklist

Example:

```cli
config ssh-filter profile
    edit "ssh-prof-filt"
        set block scp
        set log scp
    next
end
```

### Checklist

* [ ] SSH filter profile exists.
* [ ] SCP policy is defined.
* [ ] Block actions are configured where required.
* [ ] Logging is enabled.
* [ ] Legitimate administrative transfers are considered.
* [ ] Exceptions are documented.
* [ ] Policy attachment is verified.

### Example

```text
SSH
 │
 ├── SCP
 │    ├── Block
 │    └── Log
 │
 └── Other SSH
      └── Continue
```

---

# 12. DLP + SSH Checklist

DLP can inspect SSH-carried data for sensitive information when the applicable inspection path supports it.

Example:

```cli
config dlp profile
    edit "dlp-prof"

        set full-archive-proto ssh
        set summary-proto ssh

        config filter
            edit 1
                set proto ssh
            next
        end

    next
end
```

### DLP Validation

* [ ] DLP profile exists.
* [ ] SSH is included in the required DLP scope.
* [ ] DLP sensor/patterns are configured.
* [ ] Sensitive-data test files are prepared.
* [ ] SCP/SFTP test transfers are performed.
* [ ] DLP matches are logged.
* [ ] DLP actions are validated.

### Flow

```text
SSH / SCP / SFTP
        ↓
Data / File Extraction
        ↓
DLP Inspection
        ↓
Sensitive Match?
   ┌────┴────┐
   │         │
  No        Yes
   │         │
Continue    Action
```

---

# 13. Antivirus + SSH Checklist

Example:

```cli
config antivirus profile
    edit "av-prof-test"

        config ssh
            set av-scan block
            set outbreak-prevention block
            set external-blocklist block
            set fortindr block
            set quarantine enable
            set emulator enable
        end

    next
end
```

### Antivirus Checklist

* [ ] AV scanning is enabled for SSH where required.
* [ ] Malware blocking is enabled.
* [ ] Outbreak prevention behavior is reviewed.
* [ ] External blocklist behavior is reviewed.
* [ ] FortiDR integration/behavior is understood.
* [ ] Emulator behavior is evaluated where applicable.
* [ ] Quarantine behavior is configured.
* [ ] Logs are enabled.

### Archive Handling

```cli
set archive-block encrypted corrupted partiallycorrupted multipart nested mailbomb timeout unhandled
set archive-log encrypted corrupted partiallycorrupted multipart nested mailbomb timeout unhandled
```

* [ ] Encrypted archives are tested.
* [ ] Corrupted archives are tested.
* [ ] Nested archives are tested.
* [ ] Multipart archives are tested.
* [ ] Oversize/timeout behavior is understood.

---

# 14. Quarantine Checklist

Example:

```cli
config antivirus quarantine
    set drop-infected ssh
    set store-infected ssh
    set drop-blocked ssh
    set store-blocked ssh
    set drop-machine-learning ssh
    set store-machine-learning ssh
end
```

### Validate

* [ ] Infected files are dropped where required.
* [ ] Infected files are stored where required.
* [ ] Blocked files are dropped where required.
* [ ] Blocked files are stored where required.
* [ ] ML-detected files follow the intended policy.
* [ ] Quarantine storage is monitored.
* [ ] Retention requirements are documented.

### Drop vs Store

| Action    | Meaning                               |
| --------- | ------------------------------------- |
| **Drop**  | Prevent delivery                      |
| **Store** | Retain a copy for analysis/quarantine |

---

# 15. Firewall Policy Checklist

Encrypted traffic inspection depends on correct policy architecture.

### Policy

* [ ] Correct source interface is selected.
* [ ] Correct destination interface is selected.
* [ ] Correct addresses are configured.
* [ ] Correct service/application scope is configured.
* [ ] Policy action is `ACCEPT`.
* [ ] Correct inspection mode is selected.
* [ ] SSL/SSH inspection profile is attached.
* [ ] Protocol options profile is attached.
* [ ] Antivirus profile is attached where required.
* [ ] IPS profile is attached where required.
* [ ] Web Filter profile is attached where required.
* [ ] DLP profile is attached where required.
* [ ] Application Control is attached where required.
* [ ] Logging is enabled.

### Architecture

```text
Firewall Policy
      │
      ├── Inspection Mode
      │
      ├── SSL/SSH Inspection
      │
      ├── Protocol Options
      │
      ├── Antivirus
      │
      ├── IPS
      │
      ├── Web Filter
      │
      ├── DLP
      │
      └── Application Control
```

---

# 16. Proxy Inspection Checklist

Proxy-based inspection gives FortiGate greater control over application sessions.

### Validate

* [ ] Proxy inspection is appropriate for the application.
* [ ] Proxy-based security profiles are understood.
* [ ] TLS interception behavior is understood.
* [ ] Application compatibility has been tested.
* [ ] Proxy resource consumption has been evaluated.
* [ ] Session logs are available.

### `proxy-after-tcp-handshake`

Example:

```cli
config firewall ssl-ssh-profile
    edit "ssl-test-prof"

        config https
            set ports 443
            set status certificate-inspection
            set proxy-after-tcp-handshake enable
        end

    next
end
```

HTTP example:

```cli
config firewall profile-protocol-options
    edit "proto-test-prof"

        config http
            set ports 80
            set proxy-after-tcp-handshake enable
            unset options
            unset post-lang
        end

    next
end
```

### Mental Model

```text
TCP Handshake
      ↓
FortiGate Proxy
      ↓
Server-side Connection
      ↓
Protocol Inspection
```

---

# 17. Troubleshooting Checklist

## 🔴 Certificate Errors

* [ ] Client trusts the FortiGate inspection CA.
* [ ] `Fortinet_CA_SSL` is correctly deployed.
* [ ] `Fortinet_CA_Untrusted` is **not** installed as a trusted CA.
* [ ] Certificate chain is valid.
* [ ] Server certificate is valid.
* [ ] Certificate has not expired.
* [ ] SNI validation is not incorrectly blocking the connection.
* [ ] Certificate is not blocked by configured security policy.
* [ ] Browser/application trust store has been checked.

---

## 🔴 HTTPS Not Being Inspected

* [ ] Correct firewall policy is matching traffic.
* [ ] SSL/SSH inspection profile is attached.
* [ ] Deep inspection is actually enabled where required.
* [ ] Correct inspection mode is selected.
* [ ] Security profiles are attached.
* [ ] Traffic is actually entering the expected policy.
* [ ] Application is not bypassing the intended path.
* [ ] TLS version is supported.
* [ ] Certificate trust is working.
* [ ] Exemptions are not unintentionally bypassing inspection.

---

## 🔴 Application Breakage

Check for:

* [ ] Certificate pinning.
* [ ] Legacy TLS behavior.
* [ ] Custom trust stores.
* [ ] Client certificate authentication.
* [ ] SNI incompatibility.
* [ ] Unsupported TLS features.
* [ ] Inspection exemptions.
* [ ] Incorrect CA deployment.

### Troubleshooting Flow

```text
Application Failure
       ↓
Check Policy Match
       ↓
Check SSL/SSH Profile
       ↓
Check CA Trust
       ↓
Check TLS Version
       ↓
Check SNI
       ↓
Check Application Compatibility
       ↓
Check Exemptions
       ↓
Check Logs
```

---

# 18. Common Configuration Mistakes

| Mistake                                                         | Expected Problem                        |
| --------------------------------------------------------------- | --------------------------------------- |
| Certificate inspection used when payload inspection is required | Limited encrypted-content visibility    |
| Deep inspection without CA deployment                           | Certificate warnings/failures           |
| `Fortinet_CA_Untrusted` trusted by clients                      | Incorrect and risky trust configuration |
| Deep inspection applied globally without testing                | Application compatibility issues        |
| Legacy TLS ignored                                              | Client connectivity failures            |
| SSL offloading misunderstood                                    | Incorrect protocol/inspection behavior  |
| SSH inspection enabled without resource planning                | Performance impact                      |
| Oversize limits ignored                                         | Files may not be fully inspected        |
| Archive nesting ignored                                         | Incomplete file inspection              |
| DLP configured but not attached                                 | No expected DLP enforcement             |
| AV configured but not attached                                  | No expected malware inspection          |
| SSH filter created but not applied                              | No effective SSH filtering              |
| Logging disabled                                                | Difficult troubleshooting               |
| Broad trusted/exemption rules                                   | Reduced inspection coverage             |

---

# 19. Production Hardening Checklist

## 🔐 SSL/TLS

* [ ] Use TLS 1.2+ where application compatibility allows.
* [ ] Disable obsolete TLS versions where practical.
* [ ] Use deep inspection only where business/security requirements justify it.
* [ ] Deploy inspection CA through managed endpoint infrastructure.
* [ ] Never trust `Fortinet_CA_Untrusted`.
* [ ] Test critical applications before enforcement.
* [ ] Create documented inspection exemptions.
* [ ] Review exemptions periodically.
* [ ] Monitor CPU utilization.
* [ ] Monitor memory utilization.
* [ ] Monitor session counts.
* [ ] Monitor inspection-related errors.

## 🛡️ Security Profiles

* [ ] Antivirus is enabled where required.
* [ ] IPS is enabled where required.
* [ ] Web Filter is enabled where required.
* [ ] DLP is enabled where required.
* [ ] Application Control is enabled where required.
* [ ] Protocol Options are correctly configured.
* [ ] Logging is enabled.

## 🔒 SSH

* [ ] SCP/SFTP inspection is evaluated.
* [ ] SSH filtering is configured where required.
* [ ] DLP is applied where appropriate.
* [ ] Antivirus is applied where appropriate.
* [ ] File scanning limits are reviewed.
* [ ] Archive scanning is reviewed.
* [ ] Quarantine behavior is tested.
* [ ] Blocked SSH activity is logged.

---

# 20. NSE 4 / NSE 7 Exam Traps

## 🧠 Trap #1 — Certificate vs Deep Inspection

```text
Certificate Inspection
→ Certificate / TLS metadata

Deep Inspection
→ Decrypt
→ Inspect Payload
→ Re-encrypt
```

---

## 🧠 Trap #2 — CA Trust

```text
Deep Inspection
      ↓
FortiGate re-signs certificate
      ↓
Client must trust inspection CA
```

> **Never trust `Fortinet_CA_Untrusted`.**

---

## 🧠 Trap #3 — SSL Offloading

```text
External Device
      ↓
TLS Decryption
      ↓
Clear Text
      ↓
FortiGate
```

The FortiGate configuration must match the actual traffic state.

---

## 🧠 Trap #4 — SSH

```text
SSH
├── SCP
└── SFTP
```

SSH inspection can involve:

```text
Protocol Inspection
+
File Scanning
+
AV
+
DLP
+
SSH Filter
```

---

## 🧠 Trap #5 — `PROT`

```text
PROT P
→ Protected

PROT C
→ Clear
```

---

## 🧠 Trap #6 — Security Profile ≠ Decryption

Attaching:

```text
AV
IPS
DLP
Web Filter
```

does **not automatically mean** FortiGate can inspect encrypted payloads.

The inspection architecture must provide the required visibility.

---

## 🧠 Trap #7 — Oversize

If a file exceeds configured inspection limits:

```text
Large File
    ↓
Oversize Handling
    ↓
May not receive complete content inspection
```

---

## 🧠 Trap #8 — Proxy Inspection

Proxy-based inspection gives FortiGate greater control over application sessions and content processing.

---

# 21. Quick Reference

| Feature                      | Certificate Inspection | Deep Inspection |
| ---------------------------- | :--------------------: | :-------------: |
| TLS handshake visibility     |            ✅           |        ✅        |
| Certificate inspection       |            ✅           |        ✅        |
| SNI-related inspection       |            ✅           |        ✅        |
| Full payload visibility      |            ❌           |        ✅        |
| HTTPS content inspection     |            ❌           |        ✅        |
| Encrypted payload AV         |            ❌           |        ✅        |
| Encrypted payload IPS        |            ❌           |        ✅        |
| Encrypted payload DLP        |            ❌           |        ✅        |
| Requires inspection CA trust |            ❌           |        ✅        |
| Resource consumption         |          Lower         |      Higher     |
| Compatibility impact         |          Lower         |      Higher     |

---

# 22. Final Validation Checklist

Before deploying SSL/SSH inspection into production:

### Architecture

* [ ] Traffic flow is documented.
* [ ] TLS termination point is known.
* [ ] Inspection point is known.
* [ ] SSL offloading is documented.
* [ ] Proxy vs flow architecture is understood.

### Certificates

* [ ] Inspection CA is selected.
* [ ] Managed clients trust the CA.
* [ ] Certificate chain is valid.
* [ ] `Fortinet_CA_Untrusted` is not trusted.
* [ ] Certificate errors have been tested.

### TLS

* [ ] Minimum TLS version is defined.
* [ ] Legacy TLS clients are identified.
* [ ] SNI behavior is validated.
* [ ] Cipher/security requirements are reviewed.

### Security Profiles

* [ ] AV attached.
* [ ] IPS attached.
* [ ] Web Filter attached.
* [ ] DLP attached.
* [ ] Application Control attached.
* [ ] Protocol Options attached.

### SSH

* [ ] SSH inspection tested.
* [ ] SCP tested.
* [ ] SFTP tested.
* [ ] File scanning tested.
* [ ] AV tested.
* [ ] DLP tested.
* [ ] SSH filter tested.
* [ ] Quarantine tested.
* [ ] Oversize behavior tested.

### Operations

* [ ] Logging enabled.
* [ ] Monitoring configured.
* [ ] Performance baseline established.
* [ ] Exceptions documented.
* [ ] Application compatibility validated.
* [ ] Rollback plan documented.

---

# ⚡ Final Mental Model

```text
                 FORTIGATE SSL/SSH INSPECTION
                            │
                            ▼
                    Encrypted Traffic
                            │
              ┌─────────────┴─────────────┐
              │                           │
              ▼                           ▼
       Certificate                    Deep Inspection
       Inspection                          │
              │                           ▼
              │                        Decrypt
              │                           │
              │                           ▼
              │                   Security Profiles
              │                    ┌──────┼──────┐
              │                    │      │      │
              │                   AV     IPS    DLP
              │                    │      │      │
              │                    └──────┼──────┘
              │                           │
              │                       Re-encrypt
              │                           │
              └──────────────┬────────────┘
                             ▼
                           Client
```

---

# 🧠 One-Line NSE Memory Aid

> **Certificate Inspection validates TLS/certificate information; Deep Inspection decrypts encrypted traffic so FortiGate can inspect the payload; SSL Offloading means another device already terminated/decrypted TLS; SSH inspection extends security controls to SSH, SCP, and SFTP traffic.**

---

# 🏷️ Tags

`fortigate` `fortios` `ssl-inspection` `ssh-inspection` `deep-inspection` `certificate-inspection` `tls-inspection` `https-inspection` `ssl-offloading` `ssh` `scp` `sftp` `fortinet` `nse4` `nse7` `network-security` `cybersecurity` `firewall` `utm` `proxy-inspection` `dlp` `antivirus` `ips`

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

> **SheynShield Engineering Principle:**
> **Decrypt only where necessary, inspect deeply where justified, trust only the correct CA, validate application compatibility, monitor resource impact, and build encrypted-traffic security as a layered control rather than a single feature.**

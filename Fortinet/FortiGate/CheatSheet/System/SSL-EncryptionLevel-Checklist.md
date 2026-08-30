# 🔐 FortiGate Security Hardening Checklist

> **FortiOS | TLS Versions | Cipher Suites | Admin HTTPS | SSH | SSL-VPN | TPM | Private Data Encryption**
>
> **SheynShield | Engineering Secure Networks**

[![FortiOS](https://img.shields.io/badge/FortiOS-Security%20Hardening-red)](https://www.fortinet.com/products/next-generation-firewall)
[![NSE4](https://img.shields.io/badge/NSE4-Exam%20Ready-blue)](https://training.fortinet.com/)
[![NSE7](https://img.shields.io/badge/NSE7-Advanced-purple)](https://training.fortinet.com/)
[![GitHub](https://img.shields.io/badge/GitHub-SheynShield-black)](https://github.com/Shayan-heydarikhah/sheynshield)

---

## 📌 Table of Contents

* [1. Security Hardening Baseline](#1--security-hardening-baseline)
* [2. TLS Version Hardening](#2--tls-version-hardening)
* [3. Global TLS Minimum](#3--global-tls-minimum)
* [4. Admin HTTPS Hardening](#4--admin-https-hardening)
* [5. TLS 13 Cipher Suites](#5--tls-13-cipher-suites)
* [6. Weak Cipher Technologies](#6--weak-cipher-technologies)
* [7. Strong Crypto](#7--strong-crypto)
* [8. SSH Hardening](#8--ssh-hardening)
* [9. SSL-VPN Hardening](#9--ssl-vpn-hardening)
* [10. SSL-VPN Security Level](#10--ssl-vpn-security-level)
* [11. SSL-VPN TLS Range](#11--ssl-vpn-tls-range)
* [12. TPM](#12--tpm)
* [13. Private Data Encryption](#13--private-data-encryption)
* [14. TPM + Configuration Restore](#14--tpm--configuration-restore)
* [15. TPM + HA](#15--tpm--ha)
* [16. Protected Data](#16--protected-data)
* [17. TLS-Dependent Services](#17--tls-dependent-services)
* [18. Production Hardening Checklist](#18--production-hardening-checklist)
* [19. Troubleshooting Checklist](#19--troubleshooting-checklist)
* [20. Verification Commands](#20--verification-commands)
* [21. NSE Exam Checklist](#21--nse-exam-checklist)
* [22. Common Exam Traps](#22--common-exam-traps)
* [23. One-Minute Revision](#23--one-minute-revision)
* [24. Final Security Checklist](#24--final-security-checklist)
* [25. SheynShield Resources](#25--sheynshield-resources)

---

# 1. 🛡️ Security Hardening Baseline

Before changing cryptographic settings:

* [ ] Identify the FortiOS version.
* [ ] Identify the FortiGate model/platform.
* [ ] Check supported TLS versions.
* [ ] Check supported cipher suites.
* [ ] Identify legacy administrative clients.
* [ ] Identify SSL-VPN client compatibility.
* [ ] Identify external TLS-dependent services.
* [ ] Create a configuration backup.
* [ ] Document the rollback procedure.
* [ ] Test the change in a controlled environment.
* [ ] Confirm HA compatibility if applicable.

> [!WARNING]
> **Never treat a cryptographic hardening change as a simple configuration change.** Protocol and cipher restrictions can break administrative access, VPN clients, authentication systems, monitoring, and external integrations.

---

# 2. 🔒 TLS Version Hardening

TLS protocol version and cipher suite are **different security controls**.

### Security Model

```text
TLS Security
     │
     ├── Protocol Version
     │      ├── TLS 1.2
     │      └── TLS 1.3
     │
     └── Cryptography
            └── Cipher Suite
```

### Hardening Checklist

* [ ] Avoid TLS 1.0 where possible.
* [ ] Avoid TLS 1.1 where possible.
* [ ] Prefer TLS 1.2+.
* [ ] Prefer TLS 1.3 where supported.
* [ ] Validate client compatibility.
* [ ] Validate Fortinet-service compatibility.
* [ ] Validate third-party integration compatibility.

### Recommended Baseline

```text
Preferred:
TLS 1.3
   ↓
TLS 1.2

Avoid:
TLS 1.1
TLS 1.0
```

> [!TIP]
> **TLS version = protocol generation.**
>
> **Cipher suite = cryptographic algorithms used by the protocol.**

---

# 3. 🌐 Global TLS Minimum

Applicable system TLS communications can be controlled using:

```cli
config system global
    set ssl-min-proto-version tlsv1-2
end
```

### Checklist

* [ ] Review the current `ssl-min-proto-version`.
* [ ] Set TLS 1.2 as the minimum where compatibility permits.
* [ ] Consider TLS 1.3 where supported.
* [ ] Identify services affected by the global setting.
* [ ] Test external integrations.
* [ ] Monitor logs after implementation.

### Concept

```text
ssl-min-proto-version
          │
          ▼
Minimum TLS version
          │
          ├── TLS 1.2
          └── TLS 1.3
```

> [!WARNING]
> Raising the global TLS minimum may break legacy systems.

---

# 4. 🔐 Admin HTTPS Hardening

Administrative HTTPS has dedicated cryptographic controls.

```cli
config system global
    set strong-crypto enable
    set admin-https-ssl-versions tlsv1-2
    set admin-https-ssl-ciphersuites <cipher-list>
    set admin-https-ssl-banned-ciphers <cipher-list>
end
```

### Checklist

* [ ] Enable strong cryptography where appropriate.
* [ ] Allow only modern TLS versions.
* [ ] Prefer TLS 1.2/1.3.
* [ ] Remove obsolete TLS versions.
* [ ] Review TLS 1.3 cipher suites.
* [ ] Review banned cipher technologies.
* [ ] Test all administrative clients.
* [ ] Confirm GUI access after the change.
* [ ] Confirm API access after the change.

---

# 5. 🧬 TLS 1.3 Cipher Suites

Administrative HTTPS TLS 1.3 cipher suites are controlled by:

```cli
set admin-https-ssl-ciphersuites
```

### Critical Distinction

```text
Admin HTTPS
     │
     ├── TLS 1.3
     │      └── admin-https-ssl-ciphersuites
     │
     └── TLS 1.2 and lower
            └── admin-https-ssl-banned-ciphers
```

### Checklist

* [ ] Identify whether TLS 1.3 is enabled.
* [ ] Review available TLS 1.3 cipher suites.
* [ ] Select only required strong suites.
* [ ] Validate browser compatibility.
* [ ] Validate automation/API compatibility.
* [ ] Confirm TLS 1.2 behavior separately.

> [!IMPORTANT]
> `admin-https-ssl-ciphersuites` is specifically associated with **TLS 1.3**. Do not use it as if it were a universal TLS 1.2-and-below cipher control.

---

# 6. 🚫 Weak Cipher Technologies

Use:

```cli
set admin-https-ssl-banned-ciphers <cipher-list>
```

to prevent unwanted cipher technologies for TLS 1.2 and lower.

### Hardening Workflow

* [ ] Identify weak/legacy cipher technologies.
* [ ] Review current administrative client support.
* [ ] Configure banned cipher technologies.
* [ ] Test HTTPS administration.
* [ ] Test API clients.
* [ ] Monitor authentication failures.
* [ ] Document exceptions.

```text
Strong Crypto
      ↓
Modern TLS
      ↓
Cipher Restriction
      ↓
Client Compatibility Test
      ↓
Production
```

---

# 7. 💪 Strong Crypto

Enable:

```cli
config system global
    set strong-crypto enable
end
```

### Checklist

* [ ] Determine whether `strong-crypto` is supported.
* [ ] Enable it on hardened deployments.
* [ ] Check resulting CLI options.
* [ ] Validate administrative HTTPS.
* [ ] Validate SSH.
* [ ] Validate dependent services.
* [ ] Test legacy clients before enforcement.

### Important

```text
strong-crypto
      ≠
All cryptographic controls
```

> [!IMPORTANT]
> Exact behavior and available algorithms can vary by FortiOS version and platform.

---

# 8. 🔑 SSH Hardening

SSH has cryptographic controls separate from HTTPS/TLS.

Example:

```cli
config system global
    set strong-crypto enable
    set ssh-enc-algo <algorithm-list>
end
```

### SSHv1

```text
SSHv1
  ↓
Legacy
  ↓
Avoid
```

### Checklist

* [ ] Disable SSHv1 where supported.
* [ ] Use modern SSH algorithms.
* [ ] Review `ssh-enc-algo`.
* [ ] Restrict SSH to management interfaces.
* [ ] Restrict administrative source IPs.
* [ ] Avoid exposing SSH to untrusted networks.
* [ ] Test administrative access before enforcing restrictions.

> [!WARNING]
> Do not confuse SSH cryptographic settings with HTTPS/TLS settings.

---

# 9. 🌐 SSL-VPN Hardening

SSL-VPN has its own cryptographic controls.

Example:

```cli
config vpn ssl setting
    set algorithm high
    set ssl-min-proto-ver tls1-3
    set ssl-max-proto-ver tls1-3
    set ciphersuite TLS-AES-128-GCM-SHA256
end
```

### Checklist

* [ ] Set an appropriate SSL-VPN security level.
* [ ] Prefer `algorithm high`.
* [ ] Set an appropriate minimum TLS version.
* [ ] Set an appropriate maximum TLS version.
* [ ] Review TLS 1.3 cipher suites.
* [ ] Test all supported VPN clients.
* [ ] Confirm browser-based VPN compatibility if applicable.
* [ ] Validate authentication integrations.

---

# 10. 🎚️ SSL-VPN Security Level

The SSL-VPN `algorithm` setting controls the cryptographic security level.

| Setting  | Concept                  |
| -------- | ------------------------ |
| `high`   | High-security cipher set |
| `medium` | High + medium security   |
| `low`    | Broadest compatibility   |

### Recommended

```cli
config vpn ssl setting
    set algorithm high
end
```

### Checklist

* [ ] Use `high` where compatibility allows.
* [ ] Avoid `low` unless a documented compatibility requirement exists.
* [ ] Document any exception.
* [ ] Test VPN clients after changing the value.

---

# 11. 🔢 SSL-VPN TLS Range

Minimum TLS:

```cli
set ssl-min-proto-ver tls1-3
```

Maximum TLS:

```cli
set ssl-max-proto-ver tls1-3
```

### Hardened Example

```cli
config vpn ssl setting
    set ssl-min-proto-ver tls1-3
    set ssl-max-proto-ver tls1-3
end
```

### Compatibility Example

```cli
config vpn ssl setting
    set ssl-min-proto-ver tls1-2
    set ssl-max-proto-ver tls1-3
end
```

### Checklist

* [ ] Determine required client TLS support.
* [ ] Set the minimum supported secure version.
* [ ] Set the maximum supported version.
* [ ] Avoid unnecessary legacy protocols.
* [ ] Test VPN clients.
* [ ] Monitor SSL-VPN failures.

---

# 12. 🧩 TPM

A **Trusted Platform Module (TPM)** provides hardware-backed protection for cryptographic material on supported FortiGate platforms.

### TPM Can Help Protect

* [ ] Cryptographic keys.
* [ ] Encryption secrets.
* [ ] Sensitive configuration data.
* [ ] Device-bound protected information.

### Concept

```text
FortiGate
    │
    ▼
 TPM Hardware
    │
    ▼
Hardware-backed Key
    │
    ▼
Protected Encryption Material
```

### TPM Verification

```cli
diagnose hardware test info
```

```cli
diagnose hardware deviceinfo tpm
```

```cli
diagnose tpm
```

---

# 13. 🔐 Private Data Encryption

Enable on supported systems:

```cli
config system global
    set private-data-encryption enable
end
```

### Concept

```text
Private Data Encryption
          │
          ▼
Sensitive Configuration Data
          │
          ▼
Cryptographic Protection
          │
          ▼
TPM-backed Protection
```

### Checklist

* [ ] Verify TPM support.
* [ ] Verify FortiOS compatibility.
* [ ] Understand the protected-data scope.
* [ ] Establish the master encryption credential.
* [ ] Back up configuration.
* [ ] Document recovery requirements.
* [ ] Test restoration.
* [ ] Secure the encryption credential.

> [!CAUTION]
> Treat the master encryption credential as **critical recovery material**. Losing required encryption information can make protected data unrecoverable.

---

# 14. 💾 TPM + Configuration Restore

TPM-backed encryption introduces recovery dependencies.

### Scenario Matrix

| Scenario                                      | Expected Result                         |
| --------------------------------------------- | --------------------------------------- |
| TPM unavailable                               | Protected data cannot be used normally  |
| Wrong encryption credential                   | Protected data cannot be decrypted/used |
| Required TPM + matching encryption conditions | Protected configuration can be restored |

### Recovery Checklist

* [ ] Backup configuration before enabling encryption.
* [ ] Secure encryption credentials.
* [ ] Document TPM requirements.
* [ ] Document hardware dependencies.
* [ ] Test restore on supported hardware.
* [ ] Document disaster recovery procedures.
* [ ] Validate replacement-device procedures.

### Mental Model

```text
Configuration Backup
        │
        ▼
TPM-Protected Data
        │
        ├── TPM unavailable
        │       └── ❌
        │
        ├── Wrong encryption condition
        │       └── ❌
        │
        └── Required conditions satisfied
                └── ✅
```

> [!IMPORTANT]
> TPM protection should **not** be interpreted as full-disk encryption.

---

# 15. 🔗 TPM + HA

Before enabling private-data encryption in HA:

* [ ] Verify TPM support on every member.
* [ ] Verify compatible FortiOS versions.
* [ ] Verify platform compatibility.
* [ ] Back up the cluster configuration.
* [ ] Establish encryption credentials.
* [ ] Configure members consistently.
* [ ] Confirm synchronization behavior.
* [ ] Test failover.
* [ ] Test recovery.
* [ ] Document replacement-member procedures.

### HA Concept

```text
                 HA Cluster
                     │
          ┌──────────┴──────────┐
          │                     │
        FGT-1                 FGT-2
          │                     │
        TPM-1                 TPM-2
          │                     │
          └──────────┬──────────┘
                     │
              Protected Data
```

---

# 16. 🔒 Protected Data

Depending on FortiOS version and feature configuration, private-data encryption may protect sensitive credentials and cryptographic material.

Potential examples include:

* [ ] IPsec pre-shared keys
* [ ] Certificate private keys
* [ ] HA passwords
* [ ] FortiGuard proxy credentials
* [ ] LDAP credentials
* [ ] RADIUS credentials
* [ ] SNMP credentials
* [ ] Local authentication secrets
* [ ] FortiToken-related secrets
* [ ] Link Monitor credentials
* [ ] PPPoE/modem credentials
* [ ] SDN connector credentials
* [ ] Wireless security credentials
* [ ] Other feature-specific secrets

> [!NOTE]
> The exact protected-data scope is **FortiOS-release and feature dependent**.

---

# 17. 🔌 TLS-Dependent Services

TLS hardening can affect external integrations.

Review:

* [ ] FortiGuard
* [ ] FortiAnalyzer
* [ ] FortiSandbox
* [ ] LDAP
* [ ] SMTP
* [ ] POP3
* [ ] Exchange
* [ ] Syslog over TLS
* [ ] Authentication services
* [ ] External APIs
* [ ] Monitoring systems
* [ ] Certificate-based integrations

### TLS Troubleshooting Model

```text
FortiGate
   │
   ├── TLS Version
   │
   ├── Cipher Compatibility
   │
   ├── Certificate Validation
   │
   ├── SNI / Hostname
   │
   └── Remote Server Support
              │
              ▼
        TLS Handshake
```

### If an Integration Breaks

* [ ] Check TLS version compatibility.
* [ ] Check cipher compatibility.
* [ ] Check certificate chain.
* [ ] Check certificate validity.
* [ ] Check hostname/SNI.
* [ ] Check remote server configuration.
* [ ] Check FortiOS release-specific behavior.
* [ ] Check logs/debug output.

> [!TIP]
> A TLS handshake failure does **not automatically mean the certificate is wrong**.

---

# 18. 🏭 Production Hardening Checklist

## TLS

* [ ] TLS 1.0 disabled where possible.
* [ ] TLS 1.1 disabled where possible.
* [ ] TLS 1.2 available.
* [ ] TLS 1.3 enabled where supported.
* [ ] Minimum TLS version documented.
* [ ] Legacy exceptions documented.

## Admin HTTPS

* [ ] Strong cryptography enabled where appropriate.
* [ ] Modern TLS versions enabled.
* [ ] Weak cipher technologies restricted.
* [ ] TLS 1.3 cipher suites reviewed.
* [ ] Administrative clients tested.

## SSH

* [ ] SSHv1 disabled.
* [ ] Modern encryption algorithms selected.
* [ ] SSH exposed only to management networks.
* [ ] Administrative source restrictions applied.

## SSL-VPN

* [ ] `algorithm high` used where supported.
* [ ] Minimum TLS version hardened.
* [ ] Maximum TLS version defined.
* [ ] TLS 1.3 cipher suites reviewed.
* [ ] VPN clients tested.

## TPM

* [ ] TPM support verified.
* [ ] Private-data encryption evaluated.
* [ ] Encryption credentials securely stored.
* [ ] Configuration backup created.
* [ ] Recovery procedure tested.

---

# 19. 🧪 Troubleshooting Checklist

When HTTPS, SSH, SSL-VPN, or an external TLS service fails after hardening:

### Step 1 — Identify the Service

* [ ] Admin HTTPS?
* [ ] SSH?
* [ ] SSL-VPN?
* [ ] FortiGuard?
* [ ] LDAP?
* [ ] SMTP?
* [ ] FortiAnalyzer?
* [ ] FortiSandbox?
* [ ] External API?

### Step 2 — Check TLS Version

* [ ] What TLS version does FortiGate offer?
* [ ] What TLS version does the remote endpoint support?
* [ ] Is the configured minimum too high?
* [ ] Is the maximum version compatible?

### Step 3 — Check Cipher Compatibility

* [ ] Is the required cipher allowed?
* [ ] Was it banned?
* [ ] Is the cipher supported by the FortiOS release?
* [ ] Is the remote endpoint compatible?

### Step 4 — Check Certificates

* [ ] Certificate valid?
* [ ] Certificate chain trusted?
* [ ] CN/SAN correct?
* [ ] Certificate expired?
* [ ] SNI/hostname correct?

### Step 5 — Check Service Configuration

* [ ] `strong-crypto`
* [ ] `admin-https-ssl-versions`
* [ ] `admin-https-ssl-ciphersuites`
* [ ] `admin-https-ssl-banned-ciphers`
* [ ] SSL-VPN `algorithm`
* [ ] SSL-VPN minimum TLS
* [ ] SSL-VPN maximum TLS
* [ ] SSL-VPN `ciphersuite`

---

# 20. 🔎 Verification Commands

### Global Configuration

```cli
show system global
```

### Search TLS Settings

```cli
show system global | grep -f ssl
```

### SSL-VPN

```cli
show vpn ssl setting
```

### TPM

```cli
diagnose hardware test info
```

```cli
diagnose hardware deviceinfo tpm
```

```cli
diagnose tpm
```

### Recommended Verification Workflow

```text
Configuration
     ↓
Supported CLI Options
     ↓
Runtime Status
     ↓
Client Compatibility
     ↓
Logs
     ↓
Production Validation
```

> [!NOTE]
> Exact diagnostic commands and output can vary between FortiOS releases and platforms.

---

# 21. 🎯 NSE Exam Checklist

## TLS

* [ ] `ssl-min-proto-version` = global minimum TLS version.
* [ ] TLS version ≠ cipher suite.
* [ ] TLS 1.2/1.3 are preferred modern versions.

## Admin HTTPS

* [ ] `admin-https-ssl-versions` = administrative HTTPS TLS versions.
* [ ] `admin-https-ssl-ciphersuites` = TLS 1.3 cipher suites.
* [ ] `admin-https-ssl-banned-ciphers` = banned cipher technologies for TLS 1.2 and lower.
* [ ] `strong-crypto` = stronger cryptographic requirements for applicable system services.

## SSL-VPN

* [ ] `algorithm` = SSL-VPN cryptographic security level.
* [ ] `algorithm high` = strongest security level among the listed options.
* [ ] `ssl-min-proto-ver` = minimum SSL-VPN TLS version.
* [ ] `ssl-max-proto-ver` = maximum SSL-VPN TLS version.
* [ ] `ciphersuite` = SSL-VPN TLS 1.3 cipher-suite selection.

## TPM

* [ ] TPM = hardware-backed cryptographic protection.
* [ ] TPM ≠ full-disk encryption.
* [ ] Private-data encryption protects sensitive data/credentials.
* [ ] Recovery requirements must be documented.
* [ ] HA compatibility must be validated.

---

# 22. 🚨 Common Exam Traps

### Trap #1 — TLS vs Cipher Suite

```text
TLS 1.3
   ≠
Cipher Suite
```

* [ ] Remember that protocol version and cryptographic algorithm selection are separate concepts.

---

### Trap #2 — `strong-crypto` Controls Everything

```text
strong-crypto
      ≠
All SSL-VPN Cryptography
```

* [ ] Remember SSL-VPN has dedicated cryptographic settings.

---

### Trap #3 — `algorithm high`

```text
algorithm high
      ↓
SSL-VPN security level
```

* [ ] Do not interpret it as a global FortiGate cryptography setting.

---

### Trap #4 — TLS 1.3 Cipher Setting

```text
admin-https-ssl-ciphersuites
      ↓
TLS 1.3
```

* [ ] Do not confuse it with TLS 1.2-and-below cipher restrictions.

---

### Trap #5 — TPM = Full Disk Encryption

```text
TPM
 ≠
Full Disk Encryption
```

* [ ] TPM provides hardware-backed protection for cryptographic material.

---

### Trap #6 — Hardening Without Compatibility Testing

```text
Security ↑
   +
Compatibility ↓
```

* [ ] Always test clients and external integrations.

---

### Trap #7 — TPM Without Recovery Planning

```text
TPM
 +
Encryption
 +
No Recovery Plan
 =
Operational Risk
```

* [ ] Back up.
* [ ] Document.
* [ ] Test restore.

---

# 23. ⚡ One-Minute Revision

```text
FORTIGATE SECURITY HARDENING
│
├── TLS
│   ├── Minimum Version
│   ├── TLS 1.2
│   └── TLS 1.3
│
├── ADMIN HTTPS
│   ├── strong-crypto
│   ├── admin-https-ssl-versions
│   ├── admin-https-ssl-ciphersuites
│   └── admin-https-ssl-banned-ciphers
│
├── SSH
│   ├── SSH Algorithms
│   └── Disable SSHv1
│
├── SSL-VPN
│   ├── algorithm
│   ├── ssl-min-proto-ver
│   ├── ssl-max-proto-ver
│   └── ciphersuite
│
└── TPM
    ├── Hardware-backed Protection
    ├── Private Data Encryption
    ├── Sensitive Credentials
    └── Recovery Planning
```

---

# 24. 🏆 Final Security Checklist

### Protocol Hardening

* [ ] TLS 1.0 avoided.
* [ ] TLS 1.1 avoided.
* [ ] TLS 1.2 supported.
* [ ] TLS 1.3 enabled where appropriate.
* [ ] Minimum TLS version documented.

### Administrative Access

* [ ] HTTPS hardened.
* [ ] Strong cryptography enabled where appropriate.
* [ ] Weak cipher technologies restricted.
* [ ] SSHv1 disabled.
* [ ] Modern SSH algorithms configured.
* [ ] Management access restricted.

### SSL-VPN

* [ ] `algorithm high` configured where appropriate.
* [ ] Minimum TLS version hardened.
* [ ] Maximum TLS version defined.
* [ ] TLS 1.3 cipher suites reviewed.
* [ ] Client compatibility validated.

### TPM / Data Protection

* [ ] TPM availability verified.
* [ ] Private-data encryption evaluated.
* [ ] Encryption credentials secured.
* [ ] Configuration backup created.
* [ ] Restore tested.
* [ ] HA compatibility verified.
* [ ] Disaster recovery procedure documented.

### Validation

* [ ] Administrative access tested.
* [ ] VPN access tested.
* [ ] Monitoring tested.
* [ ] Authentication tested.
* [ ] FortiGuard connectivity tested.
* [ ] External TLS integrations tested.
* [ ] Logs reviewed.
* [ ] Rollback procedure verified.

---

# 🧠 SheynShield Mental Model

```text
                  FORTIGATE CRYPTO
                         │
        ┌────────────────┼────────────────┐
        │                │                │
       TLS              SSH              TPM
        │                │                │
   ┌────┴────┐           │          Hardware-backed
   │         │           │          Key Protection
Version   Cipher         │                │
   │         │           │                ▼
   ▼         ▼           ▼       Private Data Encryption
TLS 1.2   TLS 1.3      SSH       ┌───────┼───────┐
TLS 1.3   Ciphers    Algorithms   │       │       │
                                Keys  Credentials  Secrets
```

---

# 🔑 SheynShield One-Liner

> **TLS version defines the protocol generation, cipher suites define cryptographic algorithms, `strong-crypto` strengthens applicable system cryptography, SSL-VPN has its own security controls, and TPM provides hardware-backed protection for sensitive cryptographic material.**

---

## 🔎 Keywords

`FortiGate Security Hardening` · `FortiOS Security Hardening` · `FortiGate TLS` · `FortiGate TLS 1.2` · `FortiGate TLS 1.3` · `FortiGate Cipher Suites` · `FortiGate Strong Crypto` · `FortiGate HTTPS Hardening` · `FortiGate SSH Hardening` · `FortiGate SSL VPN Security` · `FortiGate SSL VPN TLS` · `FortiGate TPM` · `FortiGate Private Data Encryption` · `FortiOS TPM` · `FortiGate Encryption` · `FortiGate Hardening Checklist` · `Fortinet Security` · `Fortinet NSE4` · `Fortinet NSE7` · `Network Security` · `FortiGate Administration`

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

`Engineering Secure Networks`

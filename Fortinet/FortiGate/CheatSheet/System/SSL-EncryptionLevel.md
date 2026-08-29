# 🔐 FortiGate Security Hardening 

## TLS Versions • Cipher Suites • TPM • Private Data Encryption

> **FortiOS Focus:** Security Hardening / Administration / Cryptography
> **Level:** NSE4 → NSE7
> **Use Case:** Exam Review + Production Hardening + Troubleshooting

---

## 📌 Quick Reference

| Area                   | Configuration                             | Key Point                                                          |
| ---------------------- | ----------------------------------------- | ------------------------------------------------------------------ |
| Global TLS minimum     | `ssl-min-proto-version`                   | Controls minimum TLS version for applicable FortiOS communications |
| Admin HTTPS            | `admin-https-ssl-versions`                | Controls TLS versions for GUI/API HTTPS                            |
| Admin HTTPS ciphers    | `admin-https-ssl-ciphersuites`            | TLS 1.3 cipher suites                                              |
| Banned ciphers         | `admin-https-ssl-banned-ciphers`          | Removes unwanted TLS 1.2-and-below cipher technologies             |
| Strong crypto          | `strong-crypto`                           | Enforces stronger cryptographic settings                           |
| SSL-VPN algorithm      | `algorithm`                               | `high / medium / low` security level                               |
| SSL-VPN TLS range      | `ssl-min-proto-ver` / `ssl-max-proto-ver` | Defines allowed TLS versions                                       |
| SSL-VPN TLS 1.3 cipher | `ciphersuite`                             | Available when maximum TLS version is TLS 1.3                      |
| TPM                    | `private-data-encryption`                 | Protects sensitive credentials/keys using TPM-backed encryption    |
| TPM status             | `diagnose hardware deviceinfo tpm`        | Hardware TPM information                                           |

---

# 1. 🔒 TLS Version Hardening

## Global Minimum TLS Version

A FortiGate can define a minimum TLS protocol version for applicable SSL/TLS communications.

```bash
config system global
    set ssl-min-proto-version tlsv1-2
end
```

### Recommended Baseline

```text
TLS 1.2+
```

If the platform and connected services support it, TLS 1.3 can provide an even stronger baseline.

> ⚠️ **Compatibility matters:** Increasing the minimum TLS version can break communication with legacy systems or services that only support older TLS versions.

---

## 🧠 Why TLS Version Matters

TLS provides:

* Confidentiality
* Integrity
* Server/client authentication
* Protection against several legacy cryptographic weaknesses

### General Security Preference

```text
TLS 1.3
   ↓
TLS 1.2
   ↓
TLS 1.1  ← Legacy
   ↓
TLS 1.0  ← Legacy / Avoid
```

> **Exam Tip:** Do not confuse the **minimum TLS version** with the **cipher suite**.
> TLS version determines the protocol generation; cipher suites determine the cryptographic algorithms used within that protocol.

---

# 2. 📧 Email Server TLS

The FortiGate email-server configuration has its own TLS version control.

```bash
config system email-server
    set ssl-min-proto-version tlsv1
end
```

Check the configuration:

```bash
get | grep ssl
```

### ⚠️ Production Consideration

Do **not** automatically lower TLS security globally just because a legacy external service requires an older protocol.

Instead:

1. Identify the incompatible service.
2. Confirm its supported TLS versions.
3. Upgrade the remote service if possible.
4. Apply the least-permissive compatibility setting required.
5. Monitor logs after the change.

---

# 3. 🔗 TLS-Dependent Services

TLS version compatibility can matter for multiple FortiGate integrations, including:

* Email servers
* Certificates
* FortiSandbox
* FortiGuard
* FortiAnalyzer
* Syslog over TLS
* User authentication
* LDAP
* POP3
* Exchange
* Other external authentication/integration services

### Troubleshooting Mindset

If an integration suddenly fails after TLS hardening:

```text
FortiGate
   │
   ├── TLS version
   ├── Cipher compatibility
   ├── Certificate validation
   ├── SNI / hostname
   └── Remote server support
             │
             ▼
        TLS handshake
```

A TLS failure does **not** automatically mean the certificate is the problem.

---

# 4. 🛡️ Admin HTTPS Cryptography

Administrative HTTPS settings can be controlled under:

```bash
config system global
```

### Example

```bash
config system global
    set strong-crypto enable
    set admin-https-ssl-versions tlsv1-2
    set admin-https-ssl-ciphersuites <cipher-list>
    set admin-https-ssl-banned-ciphers <cipher-list>
end
```

---

# 5. 💪 `strong-crypto`

Enable stronger cryptographic requirements:

```bash
config system global
    set strong-crypto enable
end
```

### Important Concept

When strong cryptography is enabled, FortiOS restricts administrative cryptographic options toward stronger protocol/cipher configurations.

> ⚠️ Exact available options can vary by **FortiOS version and platform**. Always verify the CLI options on the target release.

---

# 6. 🔐 Admin HTTPS TLS Versions

Example:

```bash
config system global
    set admin-https-ssl-versions tlsv1-2
end
```

Depending on the FortiOS release, available versions can include:

```text
TLS 1.2
TLS 1.3
```

Legacy versions may exist for compatibility on some releases, but should generally be avoided in hardened deployments.

### Security Rule

```text
Prefer:
TLS 1.3
   +
TLS 1.2

Avoid:
TLS 1.0
TLS 1.1
```

---

# 7. 🧬 TLS 1.3 Cipher Suites

The setting:

```bash
set admin-https-ssl-ciphersuites
```

controls the TLS 1.3 cipher suites offered by administrative HTTPS.

### Key Exam Point

> **TLS 1.2 and lower are not controlled by this specific TLS 1.3 cipher-suite setting.**

Conceptually:

```text
Admin HTTPS
     │
     ├── TLS 1.3
     │     └── admin-https-ssl-ciphersuites
     │
     └── TLS 1.2 and lower
           └── admin-https-ssl-banned-ciphers
```

---

# 8. 🚫 Banning Weak Cipher Technologies

Use:

```bash
set admin-https-ssl-banned-ciphers
```

to prevent unwanted cipher technologies from being offered for TLS 1.2 and lower.

### Hardening Strategy

```text
1. Enable strong cryptography
2. Allow modern TLS versions
3. Remove weak cipher technologies
4. Test all administrative clients
5. Monitor authentication/access failures
```

---

# 9. ❌ Disabling TLS 1.3

If TLS 1.3 cipher suites need to be completely disabled for administrative HTTPS, remove TLS 1.3 from the configured protocol versions.

Conceptually:

```text
TLS 1.3 enabled
    ↓
TLS 1.3 cipher-suite configuration applies

TLS 1.3 removed
    ↓
No TLS 1.3 administrative HTTPS
```

---

# 10. 🔑 SSH Cryptography

SSH administrative access has separate cryptographic controls.

```bash
config system global
    set admin-ssh-v1 enable
    set strong-crypto enable
    set ssh-enc-algo <algorithm-list>
end
```

### ⚠️ SSHv1

SSH version 1 is legacy technology and should generally **not** be enabled in a modern secure deployment.

If the FortiOS release provides a way to disable SSHv1, prefer disabling it.

> **Exam trap:** SSH cryptographic settings are separate from HTTPS/TLS settings.

---

# 11. 🌐 SSL-VPN Cryptography

SSL-VPN has its own TLS and cipher controls.

```bash
config vpn ssl setting
    set algorithm high
    set ssl-min-proto-ver tls1-3
    set ssl-max-proto-ver tls1-3
    set ciphersuite TLS-AES-128-GCM-SHA256
end
```

---

# 12. 🎚️ SSL-VPN Security Level

The `algorithm` setting controls the SSL-VPN cryptographic security level.

```text
high
medium
low
```

### Behavior

| Algorithm Level | Allowed Cipher Security Levels |
| --------------- | ------------------------------ |
| `high`          | High only                      |
| `medium`        | High + Medium                  |
| `low`           | Broadest set                   |

### Production Recommendation

```bash
set algorithm high
```

when compatibility requirements allow it.

---

# 13. 🔢 SSL-VPN TLS Minimum / Maximum

Minimum TLS version:

```bash
set ssl-min-proto-ver tls1-3
```

Maximum TLS version:

```bash
set ssl-max-proto-ver tls1-3
```

Possible versions depend on FortiOS release:

```text
TLS 1.0
TLS 1.1
TLS 1.2
TLS 1.3
```

### Hardened Example

```bash
config vpn ssl setting
    set ssl-min-proto-ver tls1-3
    set ssl-max-proto-ver tls1-3
end
```

### Compatibility Example

```bash
config vpn ssl setting
    set ssl-min-proto-ver tls1-2
    set ssl-max-proto-ver tls1-3
end
```

---

# 14. 🧪 SSL-VPN TLS 1.3 Cipher Suites

Example:

```bash
set ciphersuite TLS-AES-128-GCM-SHA256
```

Other TLS 1.3 cipher suites may include:

```text
TLS-AES-128-GCM-SHA256
TLS-AES-256-GCM-SHA384
TLS-CHACHA20-POLY1305-SHA256
TLS-AES-128-CCM-SHA256
TLS-AES-128-CCM-8-SHA256
```

> ⚠️ Availability depends on FortiOS version/platform and the SSL-VPN implementation.

---

# 15. 🧠 Critical SSL-VPN Exam Point

The SSL-VPN `ciphersuite` setting is associated with **TLS 1.3** and is selectable when the maximum SSL/TLS version is TLS 1.3.

Think:

```text
SSL-VPN
   │
   ├── algorithm
   │      └── Security level
   │
   ├── ssl-min-proto-ver
   │      └── Minimum TLS version
   │
   ├── ssl-max-proto-ver
   │      └── Maximum TLS version
   │
   └── ciphersuite
          └── TLS 1.3 cipher suites
```

---

# 16. ⚠️ `strong-crypto` ≠ SSL-VPN `algorithm`

One of the most important distinctions:

```text
config system global
    set strong-crypto enable
```

is **not equivalent to**:

```bash
config vpn ssl setting
    set algorithm high
end
```

### Remember

| Setting             | Purpose                                                       |
| ------------------- | ------------------------------------------------------------- |
| `strong-crypto`     | Global cryptographic hardening for applicable system services |
| `algorithm high`    | SSL-VPN cryptographic security level                          |
| `ssl-min-proto-ver` | SSL-VPN minimum TLS version                                   |
| `ssl-max-proto-ver` | SSL-VPN maximum TLS version                                   |
| `ciphersuite`       | SSL-VPN TLS 1.3 cipher-suite selection                        |

> **Exam Trap:** `strong-crypto` does **not** directly control the SSL-VPN `algorithm` security level or SSL-VPN cipher selection.

---

# 17. 🔐 Trusted Platform Module — TPM

## What Is TPM?

A Trusted Platform Module is a dedicated hardware security component available on supported FortiGate hardware.

It can help protect sensitive cryptographic material by:

* Generating cryptographic keys
* Storing cryptographic keys
* Protecting encryption secrets
* Providing hardware-backed security
* Binding protected data to a specific FortiGate device

---

# 18. 🧩 TPM Architecture

Conceptually:

```text
Master Encryption Password
          │
          ▼
       TPM Module
          │
          ▼
     RSA-2048 Primary Key
          │
          ▼
Protects the Master Encryption Password
          │
          ▼
Sensitive FortiGate Data
```

The master encryption password is used to protect sensitive data, while the TPM-backed primary key protects the master encryption password.

---

# 19. 🔑 Master Encryption Password

On supported platforms, enabling private-data encryption requires a master encryption password.

The password is represented as a **32-hexadecimal-digit value**.

Conceptually:

```text
32 hexadecimal digits
        ↓
128 bits
        ↓
Master encryption password/key material
```

Sensitive data can then be protected using the FortiGate private-data encryption mechanism.

---

# 20. 🧠 TPM Does NOT Mean Disk Encryption

> ❗ **Important:** TPM protection should not be confused with full disk encryption.

The TPM does **not** mean that the FortiGate's entire storage device is encrypted.

Instead, it protects cryptographic material and sensitive configuration data according to FortiOS's private-data-encryption mechanism.

---

# 21. 🔗 Configuration Binding

A TPM-backed primary key is tied to the specific FortiGate hardware and does not leave the TPM.

This provides protection against simply copying protected cryptographic material to another device.

Conceptually:

```text
FortiGate A
   │
   └── TPM A
        │
        └── Primary Key A
                │
                └── Protected configuration
```

Restoring that configuration requires the appropriate TPM/encryption conditions to be satisfied.

---

# 22. 💾 Configuration Restore Behavior

Consider a backup containing TPM-protected private data.

### Scenario 1 — TPM Disabled

```text
Backup
  ↓
TPM-protected data
  ↓
TPM unavailable
  ↓
❌ Restore cannot use the protected data
```

### Scenario 2 — TPM Enabled + Wrong Master Encryption Password

```text
Backup
  ↓
Different encryption password
  ↓
❌ Protected data cannot be decrypted/used
```

### Scenario 3 — TPM Enabled + Matching Encryption Password

```text
Backup
  ↓
Matching TPM/encryption conditions
  ↓
✅ Configuration can be restored
```

> **Critical:** Before enabling TPM-backed private-data encryption, establish a documented backup and recovery procedure.

---

# 23. 🔐 Data Protected by Private Data Encryption

Depending on FortiOS features and configuration, protected sensitive data can include credentials and cryptographic secrets such as:

* Alert email credentials
* Routing-related credentials
* External resource credentials
* FortiGuard proxy credentials
* FortiToken seeds
* HA passwords
* IPsec pre-shared keys
* Link Monitor server credentials
* Certificate private keys
* Local user-related credentials
* LDAP credentials
* RADIUS credentials
* FSSO-related credentials
* Modem/PPPoE credentials
* NTP credentials
* SDN connector credentials
* SNMP credentials
* Wireless security credentials

> ⚠️ The exact protected-data scope is **FortiOS-version and feature dependent**.

---

# 24. 🏢 TPM + HA

In an HA cluster, TPM/private-data encryption introduces an important requirement:

```text
FGT-1
  │
  └── Master Encryption Key
          │
          ├──────────────┐
          ▼              ▼
        FGT-1          FGT-2
          │              │
        TPM            TPM
```

The cluster members must be configured consistently so that protected configuration data can be synchronized and the HA cluster can operate correctly.

### ⚠️ Operational Rule

Before enabling private-data encryption in HA:

1. Verify TPM support on every member.
2. Verify FortiOS compatibility.
3. Establish the encryption key/password.
4. Back up configuration.
5. Ensure all cluster members use the required matching encryption configuration.
6. Test restore/recovery procedures.

---

# 25. 🛠️ TPM Troubleshooting Commands

### Hardware Information

```bash
diagnose hardware test info
```

### TPM Device Information

```bash
diagnose hardware deviceinfo tpm
```

### TPM Diagnostics

```bash
diagnose tpm
```

---

# 26. 🔒 Enable Private Data Encryption

Example:

```bash
config system global
    set private-data-encryption enable
end
```

> ⚠️ **Production Warning:** Treat the master encryption password as critical recovery material. Losing the required encryption credentials can make protected configuration data unrecoverable.

---

# 27. 🧪 Security Hardening Workflow

A practical hardening sequence:

```text
                FortiGate
                   │
                   ▼
          ┌─────────────────┐
          │ TLS Version     │
          │ Hardening       │
          └────────┬────────┘
                   ▼
          ┌─────────────────┐
          │ Strong Crypto   │
          └────────┬────────┘
                   ▼
          ┌─────────────────┐
          │ Cipher Control  │
          └────────┬────────┘
                   ▼
          ┌─────────────────┐
          │ Admin HTTPS     │
          │ + SSH           │
          └────────┬────────┘
                   ▼
          ┌─────────────────┐
          │ SSL-VPN         │
          │ TLS Hardening   │
          └────────┬────────┘
                   ▼
          ┌─────────────────┐
          │ TPM / Private   │
          │ Data Encryption │
          └────────┬────────┘
                   ▼
          ┌─────────────────┐
          │ Backup +        │
          │ Recovery Test   │
          └─────────────────┘
```

---

# 28. 🎯 NSE Exam 

### TLS

```text
ssl-min-proto-version
        ↓
Minimum TLS version for applicable system communications
```

### Admin HTTPS

```text
admin-https-ssl-versions
        ↓
Allowed TLS versions

admin-https-ssl-ciphersuites
        ↓
TLS 1.3 cipher suites

admin-https-ssl-banned-ciphers
        ↓
Banned cipher technologies for TLS 1.2 and lower
```

### SSL-VPN

```text
algorithm
        ↓
Security level

ssl-min-proto-ver
        ↓
Minimum TLS version

ssl-max-proto-ver
        ↓
Maximum TLS version

ciphersuite
        ↓
TLS 1.3 cipher suites
```

### TPM

```text
TPM
 ↓
Hardware-backed key protection
 ↓
Private-data encryption
 ↓
Protected credentials + cryptographic material
```

---

# 29. 🚨 Common Mistakes

### ❌ Mistake 1 — Confusing TLS Version and Cipher Suite

```text
TLS 1.3 ≠ Cipher Suite
```

TLS defines the protocol version.

Cipher suites define cryptographic algorithms within the protocol.

---

### ❌ Mistake 2 — Assuming `strong-crypto` Controls Everything

```text
strong-crypto
      ≠
all SSL-VPN cryptographic settings
```

SSL-VPN has its own controls.

---

### ❌ Mistake 3 — Assuming TPM Encrypts the Entire Disk

```text
TPM
≠
Full Disk Encryption
```

TPM provides hardware-backed protection for cryptographic material and sensitive data.

---

### ❌ Mistake 4 — Enabling TLS 1.3 Without Testing Clients

Legacy clients may fail after protocol hardening.

Always test:

```text
Admins
VPN Clients
Monitoring
FortiAnalyzer
FortiGuard
LDAP
SMTP
External APIs
Authentication Systems
```

---

### ❌ Mistake 5 — Enabling TPM Without a Recovery Plan

Before enabling private-data encryption:

```text
Backup
   +
Document encryption credentials
   +
Verify HA compatibility
   +
Test restore
```

---

# 30. 🔥 Production Hardening Baseline

A reasonable modern baseline is:

```text
✅ TLS 1.2 minimum where compatibility requires it
✅ TLS 1.3 where supported
✅ Strong cryptography enabled where appropriate
✅ Disable legacy SSHv1
✅ Restrict administrative HTTPS ciphers
✅ Avoid TLS 1.0 / TLS 1.1
✅ Use SSL-VPN high security where supported
✅ Prefer modern SSL-VPN TLS versions
✅ Protect private configuration data
✅ Use TPM where supported and operationally appropriate
✅ Maintain tested configuration backups
✅ Test recovery before relying on encrypted backups
```

---

# 🧠 Final Memory Map

```text
                    FORTIGATE CRYPTO
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
       TLS               SSH               TPM
        │                 │                 │
   ┌────┴────┐       SSH algorithms    Hardware
   │         │             │           key protection
 Version   Cipher          │                 │
   │         │             │                 ▼
   ▼         ▼             ▼        Private Data Encryption
TLS 1.2/1.3 Cipher       SSH        ┌────────┼────────┐
                                     │        │        │
                                  Passwords  Keys   Credentials
```

## ⭐ One-Minute Revision

> **TLS version = protocol generation**
> **Cipher suite = cryptographic algorithms**
> **`strong-crypto` = stronger system cryptography**
> **`algorithm high` = SSL-VPN security level**
> **`ssl-min/max-proto-ver` = SSL-VPN TLS range**
> **`ciphersuite` = SSL-VPN TLS 1.3 cipher selection**
> **TPM = hardware-backed protection of cryptographic material**
> **Private-data encryption ≠ full disk encryption**
> **Always validate compatibility before disabling legacy protocols.**

---

## 🔎 Keywords

`FortiGate` · `FortiOS` · `NSE4` · `NSE7` · `TLS 1.2` · `TLS 1.3` · `SSL VPN` · `HTTPS` · `SSH` · `Cipher Suite` · `Strong Crypto` · `TPM` · `Private Data Encryption` · `Fortinet Security` · `Network Security` · `FortiGate Hardening`


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

# 🔐 FortiGate SSL/SSH Inspection  

> **FortiOS Focus:** SSL/SSH Inspection, Certificate Inspection, Deep Inspection, SSL Offloading, SSH File Scanning
> **Level:** NSE 4 / NSE 7
> **Use Case:** HTTPS inspection, certificate validation, encrypted traffic security profiles, SSH/SCP/SFTP inspection

---

## 📌 1. Why SSL/SSH Inspection?

Encrypted traffic can hide malicious content from security profiles such as:

* Antivirus
* IPS
* Web Filter
* Email Filter
* DLP
* Application Control
* File scanning

Therefore:

```text
Client
   │
   │ Encrypted HTTPS / SSH
   ▼
┌───────────────┐
│   FortiGate   │
│               │
│ Decrypt       │
│ Inspect       │
│ Re-encrypt    │
└───────────────┘
   │
   ▼
 Server
```

For HTTPS deep inspection:

```text
Encrypted Traffic
       ↓
     Decrypt
       ↓
 Inspect Clear Text
       ↓
 Security Profiles
       ↓
    Re-encrypt
       ↓
     Client
```

---

# 🧩 2. SSL Inspection Modes

## Certificate Inspection

Certificate inspection is the lightweight option.

It primarily examines the TLS handshake and certificate information rather than decrypting the complete application payload.

Typical checks include:

* Certificate validity
* Certificate issuer
* Certificate chain
* Certificate expiration
* Certificate-related policy checks
* SNI / certificate name matching

### Key Point

```text
Certificate Inspection
        │
        ├── TLS handshake
        ├── Certificate information
        └── No full payload decryption
```

Therefore, certificate inspection **cannot provide the same visibility into encrypted application content as deep inspection**.

---

# 🔬 3. Deep / Custom SSL Inspection

Deep inspection is used when FortiGate must inspect the actual encrypted payload.

```text
Client
  │
  │ TLS
  ▼
FortiGate
  │
  ├── TLS Decryption
  ├── Content Inspection
  │     ├── IPS
  │     ├── Antivirus
  │     ├── Web Filter
  │     ├── DLP
  │     └── Application Control
  │
  ├── Re-encryption
  ▼
Server
```

### Deep Inspection Requirements

The FortiGate acts as a TLS interception point and uses a CA certificate to generate/re-sign certificates for inspected destinations.

Common certificates include:

```text
Fortinet_CA_SSL
Fortinet_CA_Untrusted
Custom/Enterprise CA
```

---

# 🏛️ 4. FortiGate CA Certificate Behavior

During deep inspection:

```text
Original Server Certificate
            ↓
       FortiGate
            ↓
Re-signed Certificate
            ↓
         Client
```

The client must trust the CA used by FortiGate to re-sign inspected certificates.

### `Fortinet_CA_SSL`

If the FortiGate CA is not trusted by the client/browser:

```text
Browser
   ↓
Certificate Warning
```

To prevent this warning:

```text
Install Fortinet_CA_SSL
        ↓
Trusted CA Store
        ↓
Browser trusts FortiGate-generated certificates
```

### ⚠️ Important

**Do NOT install `Fortinet_CA_Untrusted` as a trusted CA.**

It is intended for certificates that FortiGate considers untrusted and should not be added to the browser's trusted CA store.

---

# 🛡️ 5. SSL Inspection Security Options

Navigate to:

```text
Security Profiles
└── SSL/SSH Inspection
```

Important options include:

| Option                       | Purpose                                                                |
| ---------------------------- | ---------------------------------------------------------------------- |
| Certificate Inspection       | Inspect TLS certificate/handshake information                          |
| Deep Inspection              | Decrypt and inspect encrypted content                                  |
| Blocked Certificates         | Block certificates listed in FortiGuard's blocked certificate database |
| Untrusted SSL Certificates   | Control handling of certificates that are not trusted                  |
| Server Certificate SNI Check | Compare SNI information with certificate identity                      |
| SSL Cipher Compliance        | Enforce stronger/allowed TLS cipher requirements                       |
| RPC over HTTP                | Supports inspection handling for relevant Microsoft/RPC traffic        |

---

# 🌐 6. Server Certificate SNI Check

SNI = **Server Name Indication**

During TLS negotiation, the client can indicate the requested hostname through SNI.

Example:

```text
Client
  │
  │ ClientHello
  │ SNI = example.com
  ▼
FortiGate
```

The server certificate may contain:

```text
CN  = example.com
SAN = example.com
```

### SNI Check Modes

| Mode    | Behavior                                            |
| ------- | --------------------------------------------------- |
| Disable | No SNI/certificate-name validation                  |
| Enable  | Uses SNI and certificate identity during validation |
| Strict  | Requires stricter SNI/SAN matching                  |

### Practical Idea

```text
SNI
 ↓
Certificate SAN/CN
 ↓
Compare Identity
 ↓
Allow / Block
```

---

# 🔐 7. Strong Cryptography

Enable stronger cryptographic requirements globally:

```cli
config system global
    set strong-crypto enable
end
```

> **Best Practice:** Evaluate compatibility before enabling stricter cryptographic requirements in production, especially with legacy clients and applications.

---

# 🔒 8. TLS Version Control

Example SSL/SSH profile:

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

### Result

With an appropriate configuration:

```text
TLS 1.0  → Block
TLS 1.1  → Block
TLS 1.2  → Allow
TLS 1.3  → Allow*
```

> `*` Actual TLS 1.3 behavior depends on the FortiOS version, inspection mode, and platform support.

---

# ⚡ 9. SSL Offloading / External TLS Decryption

Sometimes FortiGate is positioned **behind another SSL/TLS decryption device**, such as an ADC or external TLS inspection/decryption appliance.

```text
Client
   │
   ▼
External TLS Device
   │
   │ Clear Text
   ▼
FortiGate
   │
   ▼
Server
```

In this architecture, FortiGate must know that the incoming traffic has already been decrypted.

### SSL-Offloaded Concept

```text
External Device
       │
       │ TLS Decrypt
       ▼
     Clear Text
       │
       ▼
   FortiGate
```

If FortiGate expects encrypted traffic while receiving clear text, protocol detection/inspection can behave incorrectly.

---

# 🧰 10. SSL-Offloaded Protocol Configuration

Example:

```cli
config firewall profile-protocol-options
    edit "ssl-test"

        config http
            set ports 80
            unset options
            unset post-lang
            set ssl-offloaded yes
        end

        config ftp
            set ports 21
            set options splice
            set ssl-offloaded yes
        end

        config imap
            set ports 143
            set options fragmail
            set ssl-offloaded yes
        end

        config pop3
            set ports 110
            set options fragmail
            set ssl-offloaded yes
        end

        config smtp
            set ports 25
            set options fragmail splice
            set ssl-offloaded yes
        end

    next
end
```

### Key Rule

> If an external device performs SSL decryption, configure FortiGate so the protocol engine understands that the traffic is already SSL-offloaded.

---

# 🚨 11. `AUTH TLS`, `PBSZ`, and `PROT`

For protocols such as FTP/FTPS, protocol negotiation can change the protection state.

Important concepts:

```text
AUTH TLS
    ↓
TLS negotiation

PBSZ
    ↓
Protection Buffer Size

PROT
    ↓
Protection Level
```

`PROT` can determine whether subsequent data is protected or sent in clear text.

Common concept:

```text
PROT P = Protected
PROT C = Clear
```

### SSL-Offloaded Consideration

When SSL offloading is enabled, FortiGate may treat traffic differently because the external device has already performed the TLS decryption.

> **Important:** Do not blindly apply `ssl-offloaded` to all environments. The configuration must match the actual traffic path and where TLS encryption/decryption occurs.

---

# 🖥️ 12. SSH Traffic Inspection

FortiGate can inspect SSH-based file transfers and related traffic.

Important protocols:

```text
SSH
├── SCP
└── SFTP
```

Security controls can include:

* Antivirus
* DLP
* SSH filtering
* File scanning
* Protocol options
* Quarantine
* File logging

---

# 📁 13. SSH File Scanning

Example protocol options:

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

### Important Parameters

| Parameter                     | Purpose                                       |
| ----------------------------- | --------------------------------------------- |
| `comfort-interval`            | Controls interval for comfort traffic         |
| `comfort-amount`              | Controls amount of comfort traffic            |
| `oversize-limit`              | Controls maximum size considered for scanning |
| `uncompressed-oversize-limit` | Controls uncompressed content size            |
| `uncompressed-nest-limit`     | Controls archive/compression nesting          |
| `scan-bzip2`                  | Enables BZIP2 scanning                        |

---

# 🚫 14. SSH Filter

Example:

```cli
config ssh-filter profile
    edit "ssh-prof-filt"
        set block scp
        set log scp
    next
end
```

This allows SCP activity to be controlled independently through the SSH filter.

### Example Policy Logic

```text
SSH
 │
 ├── SCP → Block + Log
 │
 └── Other SSH → Continue inspection
```

---

# 🧪 15. DLP + SSH

DLP can be applied to SSH traffic to detect sensitive information.

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

### DLP Inspection

```text
SSH/SCP/SFTP
     ↓
File/Data Extraction
     ↓
DLP Sensor
     ↓
Sensitive Data Match?
     ├── No  → Continue
     └── Yes → Action
```

---

# 🦠 16. Antivirus + SSH

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

Archive handling can also be configured:

```cli
set archive-block encrypted corrupted partiallycorrupted multipart nested mailbomb timeout unhandled
set archive-log encrypted corrupted partiallycorrupted multipart nested mailbomb timeout unhandled
```

---

# 📦 17. SSH Quarantine

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

Conceptually:

```text
Detected File
     │
     ├── Infected
     │     ├── Drop
     │     └── Store
     │
     ├── Blocked
     │     ├── Drop
     │     └── Store
     │
     └── ML Detection
           ├── Drop
           └── Store
```

### Drop vs Store

| Action | Meaning                                     |
| ------ | ------------------------------------------- |
| Drop   | Prevent delivery                            |
| Store  | Keep a copy for further analysis/quarantine |

---

# 🔥 18. Security Profile Policy Requirements

For encrypted traffic inspection, the firewall policy must use the appropriate inspection mode and security profiles.

Typical architecture:

```text
Firewall Policy
│
├── Proxy-based inspection
│
├── SSL/SSH Inspection
│      └── Deep Inspection / Custom Inspection
│
├── IPS
├── Antivirus
├── Web Filter
├── DLP
├── Application Control
└── Protocol Options
```

### Example

```text
Policy
 ├── Action: ACCEPT
 ├── Inspection: Proxy
 ├── SSL/SSH Profile: Custom Deep Inspection
 ├── IPS Sensor
 ├── AV Profile
 ├── Web Filter
 ├── DLP Profile
 └── Protocol Options
```

> **Key Point:** Simply enabling a security profile does not guarantee visibility into encrypted payloads. The inspection architecture must allow FortiGate to decrypt and inspect the relevant traffic.

---

# 🔄 19. Proxy After TCP Handshake

FortiGate can be configured to start proxy behavior after the TCP three-way handshake.

Normal TCP:

```text
Client                  FortiGate                 Server
  │                        │                        │
  │-------- SYN ---------->│                        │
  │                        │-------- SYN --------->│
  │                        │<------ SYN/ACK --------│
  │<------ SYN/ACK --------│                        │
  │-------- ACK ---------->│                        │
  │                        │                        │
```

With proxy behavior, FortiGate can establish/control the server-side connection independently.

---

# 🧠 20. `proxy-after-tcp-handshake`

Example HTTPS configuration:

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

And HTTP:

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

### Concept

```text
TCP Handshake
      ↓
FortiGate Proxy
      ↓
Server Connection
      ↓
Protocol Inspection
```

If disabled, the connection behavior can be more directly established with the server depending on the inspection configuration.

---

# 🎯 21. Why Proxy-Based Inspection Matters

Proxy inspection provides FortiGate with more control over the application session.

```text
Client
  │
  ▼
FortiGate Proxy
  │
  ├── Decode
  ├── Inspect
  ├── Apply Security Profiles
  ├── Block / Allow
  └── Rebuild / Forward
  │
  ▼
Server
```

This is particularly important for:

* HTTPS inspection
* SSH file inspection
* DLP
* Antivirus
* Web filtering
* Content inspection

---

# 🧭 22. Inspection Decision Tree

```text
Encrypted Traffic
       │
       ▼
Is TLS/SSL inspection required?
       │
   ┌───┴───┐
   │       │
  No      Yes
   │       │
   ▼       ▼
Forward   Certificate
           Inspection
              │
              ▼
       Need payload visibility?
              │
          ┌───┴───┐
          │       │
         No      Yes
          │       │
          ▼       ▼
      Continue   Deep
                 Inspection
                    │
                    ▼
               Decrypt
                    │
                    ▼
             Security Profiles
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
         IPS        AV        DLP
          │         │         │
          └─────────┼─────────┘
                    ▼
                Re-encrypt
                    │
                    ▼
                  Client
```

---

# ⚠️ 23. Common SSL/SSH Inspection Mistakes

| Mistake                                                       | Result                                                 |
| ------------------------------------------------------------- | ------------------------------------------------------ |
| Using certificate inspection expecting payload scanning       | Encrypted content remains unavailable                  |
| Forgetting client CA trust                                    | Browser certificate warnings                           |
| Installing `Fortinet_CA_Untrusted` as trusted                 | **Security risk / incorrect trust configuration**      |
| Using deep inspection without planning exclusions             | Application compatibility issues                       |
| Ignoring legacy TLS clients                                   | Connectivity problems                                  |
| Forgetting proxy-based requirements                           | Some security profiles cannot inspect expected content |
| Enabling SSL offloading without matching traffic architecture | Protocol/inspection problems                           |
| Blocking SSH/SCP without logging                              | Difficult troubleshooting                              |
| Ignoring archive/oversize limits                              | Files may not be fully inspected                       |
| Assuming HTTPS = one inspection behavior                      | TLS version/application behavior can differ            |

---

# 🧪 24. Troubleshooting Checklist

### Certificate Problems

```text
□ Is the client trusting the FortiGate inspection CA?
□ Is the certificate chain valid?
□ Is the server certificate trusted?
□ Is SNI validation enabled?
□ Is the certificate blocked?
□ Is Fortinet_CA_Untrusted involved?
```

### HTTPS Inspection

```text
□ Is the firewall policy using the correct SSL/SSH profile?
□ Is deep inspection required?
□ Is the policy using the expected inspection mode?
□ Are IPS/AV/DLP profiles attached?
□ Is the traffic actually being decrypted?
□ Is TLS 1.2/1.3 supported by the client/application?
```

### SSH Inspection

```text
□ Is SSH protocol inspection enabled?
□ Is SCP/SFTP traffic being identified?
□ Is the SSH filter attached?
□ Is DLP attached?
□ Is Antivirus attached?
□ Are oversize limits appropriate?
□ Is archive scanning enabled where required?
□ Is quarantine configured?
```

---

# 🔎 25. Useful Mental Model

Remember this distinction:

```text
Certificate Inspection
        ↓
"Who are you?"
        ↓
Certificate / TLS metadata
```

versus:

```text
Deep Inspection
        ↓
"What are you sending?"
        ↓
Decrypt
        ↓
Inspect Payload
        ↓
Security Profiles
        ↓
Re-encrypt
```

And for SSH:

```text
SSH
 │
 ├── Protocol Inspection
 ├── File Extraction
 ├── Antivirus
 ├── DLP
 └── SSH Filter
```

---

# 🚀 26. Production Best Practices

### SSL/TLS

* Prefer **TLS 1.2+** where application compatibility allows.
* Use **Deep Inspection** when payload-level security inspection is required.
* Deploy the inspection CA to managed endpoints.
* Never trust `Fortinet_CA_Untrusted`.
* Test business-critical applications before enforcing deep inspection globally.
* Create targeted SSL inspection policies rather than blindly decrypting everything.
* Monitor CPU, memory, and session impact.

### SSH

* Inspect SCP/SFTP where data-loss or malware risk justifies it.
* Combine SSH inspection with **AV + DLP + SSH filtering** where appropriate.
* Tune oversize and archive limits based on available resources.
* Log blocked SSH activity.
* Validate inspection behavior with controlled test transfers.

---

# 📌 27. Quick Reference

| Feature                          | Certificate Inspection | Deep Inspection |
| -------------------------------- | ---------------------: | --------------: |
| Certificate validation           |                      ✅ |               ✅ |
| SNI inspection                   |                      ✅ |               ✅ |
| Full payload visibility          |                      ❌ |               ✅ |
| HTTPS content inspection         |                      ❌ |               ✅ |
| IPS on encrypted payload         |                      ❌ |               ✅ |
| AV on encrypted payload          |                      ❌ |               ✅ |
| DLP on encrypted payload         |                      ❌ |               ✅ |
| Requires trusted inspection CA   |                      ❌ |               ✅ |
| Higher resource consumption      |                  Lower |          Higher |
| Application compatibility impact |                  Lower |          Higher |

---

# 🧠 NSE Exam Memory Hook

> **Certificate Inspection = inspect the certificate.**
> **Deep Inspection = decrypt → inspect → re-encrypt.**
> **SSL Offloading = another device already decrypted the traffic.**
> **SSH inspection = protocol + file + AV/DLP/filter controls.**

```text
               FORTIGATE SSL/SSH INSPECTION

                         ┌──────────────┐
                         │    Traffic   │
                         └──────┬───────┘
                                │
                    ┌───────────▼───────────┐
                    │ SSL/TLS / SSH Traffic │
                    └───────────┬───────────┘
                                │
                    ┌───────────▼───────────┐
                    │   Inspection Profile  │
                    └───────────┬───────────┘
                                │
                 ┌──────────────┴──────────────┐
                 │                             │
          Certificate                    Deep Inspection
          Inspection                          │
                 │                            ▼
                 │                       Decrypt
                 │                            │
                 │                            ▼
                 │                    Security Profiles
                 │                     ┌──────┼──────┐
                 │                     │      │      │
                 │                    IPS     AV     DLP
                 │                     │      │      │
                 │                     └──────┼──────┘
                 │                            │
                 │                         Encrypt
                 │                            │
                 └──────────────┬─────────────┘
                                ▼
                              Client
```

---

## 🏷️  /  Tags

`fortigate` `fortios` `ssl-inspection` `ssh-inspection` `deep-inspection` `certificate-inspection` `tls-inspection` `https-inspection` `ssh` `scp` `sftp` `fortinet` `nse4` `nse7` `network-security` `cybersecurity` `firewall` `utm` `proxy-inspection` `dlp` `antivirus` `ips`

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

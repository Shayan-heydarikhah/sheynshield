# FortiGate SSH Filter & SSL Inspection  

> **FortiOS | SSH Filtering, Shell Blocking, Command Logging & SSL Inspection**

---

## 📌 Table of Contents

* [1. SSH Filter Profiles](#1-ssh-filter-profiles)
* [2. SSH Shell Control](#2-ssh-shell-control)
* [3. SSH Command Logging](#3-ssh-command-logging)
* [4. SSH Filter + Firewall Policy](#4-ssh-filter--firewall-policy)
* [5. SSL SSH Inspection Profile](#5-ssl-ssh-inspection-profile)
* [6. SSL Logging](#6-ssl-logging)
* [7. Certificate Inspection](#7-certificate-inspection)
* [8. End-to-End Traffic Flow](#8-end-to-end-traffic-flow)
* [9. Security Design Recommendations](#9-security-design-recommendations)
* [10. NSE High-Value Notes](#10-nse-high-value-notes)
* [11. Quick Reference](#11-quick-reference)

---

# 1. SSH Filter Profiles

FortiGate provides **SSH Filter Profiles** to control and monitor SSH sessions passing through the firewall.

Configuration:

```bash
config ssh-filter profile
    edit ssh-test
        set block shell
        set log shell
        set default-command-log disable
    next
end
```

### Main Controls

| Setting               | Purpose                              |
| --------------------- | ------------------------------------ |
| `block shell`         | Blocks interactive SSH shell         |
| `log shell`           | Logs shell-related activity          |
| `default-command-log` | Controls default SSH command logging |

---

# 2. SSH Shell Control

## Block Interactive Shell

```bash
set block shell
```

This prevents the SSH client from obtaining an interactive shell through the inspected SSH session.

Conceptually:

```text
SSH Client
    │
    ▼
SSH Session
    │
    ▼
FortiGate SSH Filter
    │
    ├── Shell request
    │       │
    │       ▼
    │    BLOCK
    │
    └── Allowed SSH functionality
```

### Why Block Shell?

Useful when the SSH connection is required for a limited purpose but interactive shell access should not be allowed.

For example:

```text
Allowed:
    SSH-based application/function

Blocked:
    Interactive shell
```

> **Security principle:** Allow the protocol only for the functionality that is actually required.

---

# 3. SSH Command Logging

## Enable Shell Logging

```bash
set log shell
```

This enables logging related to shell activity.

---

## Default Command Logging

```bash
set default-command-log disable
```

When command logging is enabled, FortiGate can record commands executed through the inspected SSH session.

Conceptually:

```text
SSH Client
    │
    ├── Command 1
    ├── Command 2
    ├── Command 3
    └── Command 4
          │
          ▼
     SSH Filter
          │
          ▼
       Logging
```

### Security Use Case

Command logging can be valuable when:

* SSH access must be audited
* Administrators execute scripts remotely
* Privileged operations need visibility
* You need forensic evidence
* SSH activity must be correlated with security events

---

## ⚠️ Important Design Consideration

If clients are trusted and command-level auditing is required, command logging can be enabled.

However, a stronger security model is often:

```text
Trusted Client
      │
      ▼
Command Logging
      +
Shell Restrictions
```

rather than allowing unrestricted interactive shell access.

> **Don't confuse:**
> **Logging a shell** does not mean **allowing a shell**.
> `block shell` and `log shell` can be used together.

---

# 4. SSH Filter + Firewall Policy

An SSH Filter Profile is applied through a firewall policy.

Example:

```bash
config firewall policy
    edit 1
        set srcintf port3
        set dstintf port1
        set srcaddr all
        set dstaddr all
        set action accept
        set schedule always
        set service all
        set utm-status enable
        set inspection-mode proxy
        set ssh-filter-profile ssh-test
        set profile-protocol-options protocol
        set ssl-ssh-profile ssl
        set nat enable
    next
end
```

---

## 🔗 Important Policy Requirements

The policy must have UTM inspection enabled:

```bash
set utm-status enable
```

And proxy inspection:

```bash
set inspection-mode proxy
```

Then attach the SSH filter:

```bash
set ssh-filter-profile ssh-test
```

---

## Why Proxy Mode?

SSH filtering requires FortiGate to inspect and understand the application protocol.

Conceptually:

```text
Client
  │
  │ SSH
  ▼
FortiGate
  │
  ├── Proxy / Protocol Inspection
  │
  ├── SSH Filter
  │
  └── Logging
  │
  ▼
Server
```

> **NSE tip:** When troubleshooting an SSH filter that does not appear to work, check the **firewall policy**, **UTM status**, and **inspection mode** before blaming the SSH profile.

---

# 5. SSL SSH Inspection Profile

SSH and SSL/HTTPS inspection settings can be associated with the firewall policy through an SSL/SSH profile.

Example:

```bash
config firewall ssl-ssh-profile
    edit cus-deep-test
        set server-cert-mode re-sign
        set caname Fortinet_CA_SSL
        set untrusted-caname Fortinet_CA_Untrusted
        set ssl-anomaly-log enable
        set ssl-exemption-log enable
        set ssl-negotiation-log enable
        set rpc-over-https disable
        set mapi-over-https disable
        set use-ssl-server disable
        set ssl-server-cert-log enable
        set ssl-handshake-log enable
    next
end
```

---

## 🔐 Re-Sign Mode

```bash
set server-cert-mode re-sign
```

In re-sign inspection, FortiGate terminates the TLS session from the client side and establishes a separate TLS session toward the server.

Conceptually:

```text
              TLS Session 1
Client ─────────────────────> FortiGate
                                │
                                │ TLS Session 2
                                ▼
                              Server
```

FortiGate acts as the inspection point between the two TLS sessions.

---

## CA Configuration

Trusted CA:

```bash
set caname Fortinet_CA_SSL
```

Untrusted CA:

```bash
set untrusted-caname Fortinet_CA_Untrusted
```

The CA certificate presented to clients must be trusted by those clients for seamless certificate re-signing.

---

# 6. SSL Logging

For advanced troubleshooting and visibility, enable the available SSL logging options relevant to the inspection profile.

### SSL Anomaly Logging

```bash
set ssl-anomaly-log enable
```

Useful for detecting SSL/TLS anomalies.

---

### SSL Exemption Logging

```bash
set ssl-exemption-log enable
```

Useful for understanding when traffic is exempted from SSL inspection.

---

### SSL Negotiation Logging

```bash
set ssl-negotiation-log enable
```

Useful for investigating TLS negotiation behavior.

---

### SSL Server Certificate Logging

```bash
set ssl-server-cert-log enable
```

Useful for visibility into the server certificates observed during SSL inspection.

---

### SSL Handshake Logging

```bash
set ssl-handshake-log enable
```

Useful when troubleshooting TLS handshake problems.

---

## Recommended Troubleshooting Visibility

For deep SSL/TLS troubleshooting:

```text
SSL Anomaly Log
       +
SSL Exemption Log
       +
SSL Negotiation Log
       +
SSL Server Certificate Log
       +
SSL Handshake Log
```

> ⚠️ Enabling extensive SSL logging can significantly increase log volume. Enable it intentionally, especially in production environments.

---

# 7. Certificate Inspection

When SSL server certificate logging is enabled, certificate information can be exposed for investigation.

A certificate record can contain fields such as:

```text
notbefore
notafter
issuer
cn
san
sn
ski
certhash
keyalgo
keysize
```

Example:

```text
notbefore="2021-03-13T00:00:00Z"
notafter="2022-04-13T23:59:59Z"
issuer="DigiCert TLS RSA SHA256 2020 CA1"
cn="*.fortinet.com"
san="*.fortinet.com;www.fortinet.com;fortinet.com"
sn="000aa00a00000a00000a00a00aa000a0"
ski="df9152b605cc18b346efb34de6907275dbdb2b3c"
certhash="1d55cd34a1ed5d3f69bd825a45e04fbd2efba937"
keyalgo="rsa"
keysize=2048
```

---

## 🔎 Certificate Fields

| Field       | Meaning                      |
| ----------- | ---------------------------- |
| `notbefore` | Certificate validity start   |
| `notafter`  | Certificate expiration       |
| `issuer`    | Certificate issuer / CA      |
| `cn`        | Common Name                  |
| `san`       | Subject Alternative Name     |
| `sn`        | Certificate serial number    |
| `ski`       | Subject Key Identifier       |
| `certhash`  | Certificate hash/fingerprint |
| `keyalgo`   | Public-key algorithm         |
| `keysize`   | Public-key size              |

---

## Certificate Investigation

When investigating SSL problems, check:

```text
1. Certificate validity
2. Expiration
3. Issuer
4. CN
5. SAN
6. Serial number
7. Key algorithm
8. Key size
9. Certificate hash
```

---

# 8. End-to-End Traffic Flow

A simplified architecture:

```text
                         FortiGate
                            │
                            ▼
                    Firewall Policy
                            │
                ┌───────────┴───────────┐
                │                       │
          UTM Enabled             Proxy Inspection
                │                       │
                └───────────┬───────────┘
                            │
                ┌───────────┴───────────┐
                │                       │
                ▼                       ▼
          SSH Filter             SSL/SSH Profile
                │                       │
        ┌───────┴───────┐       ┌───────┴────────┐
        │               │       │                │
   Shell Block      Command   SSL Logs       Certificate
                    Logging
        │               │       │                │
        └───────────────┴───────┴────────────────┘
                            │
                            ▼
                          Logs
```

---

# 9. SSH Security Design

A good SSH security architecture separates three concepts:

```text
        SSH Security
             │
     ┌───────┼────────┐
     │       │        │
     ▼       ▼        ▼
  Access   Control   Audit
     │       │        │
     ▼       ▼        ▼
 Policy   Shell     Logs
          Commands
```

### Access

Control who can establish SSH connections:

```text
Source
Destination
Service
Policy
```

### Control

Restrict what the SSH session can do:

```text
SSH Filter
    ↓
Shell restrictions
```

### Audit

Record what happened:

```text
Shell logs
Command logs
SSL logs
Certificate logs
```

---

# 10. NSE High-Value Notes 🧠

### SSH Filter Location

```bash
config ssh-filter profile
```

---

### Block Interactive Shell

```bash
set block shell
```

> **Block shell ≠ block SSH entirely.**

It specifically targets shell functionality.

---

### Log Shell Activity

```bash
set log shell
```

---

### Firewall Policy Integration

```bash
set utm-status enable
set inspection-mode proxy
set ssh-filter-profile <profile>
```

---

### SSL Inspection

```bash
config firewall ssl-ssh-profile
```

Re-sign:

```bash
set server-cert-mode re-sign
```

CA:

```bash
set caname <CA>
```

---

### SSL Troubleshooting Logs

```bash
set ssl-anomaly-log enable
set ssl-exemption-log enable
set ssl-negotiation-log enable
set ssl-server-cert-log enable
set ssl-handshake-log enable
```

---

# 11. Quick Reference

| Objective                      | Command                            |
| ------------------------------ | ---------------------------------- |
| Create SSH filter              | `config ssh-filter profile`        |
| Block shell                    | `set block shell`                  |
| Log shell                      | `set log shell`                    |
| Configure command logging      | `set default-command-log ...`      |
| Enable UTM                     | `set utm-status enable`            |
| Proxy inspection               | `set inspection-mode proxy`        |
| Attach SSH filter              | `set ssh-filter-profile <profile>` |
| Configure SSL/SSH profile      | `config firewall ssl-ssh-profile`  |
| Re-sign certificates           | `set server-cert-mode re-sign`     |
| Configure trusted CA           | `set caname <CA>`                  |
| SSL anomaly logging            | `set ssl-anomaly-log enable`       |
| SSL exemption logging          | `set ssl-exemption-log enable`     |
| SSL negotiation logging        | `set ssl-negotiation-log enable`   |
| SSL server certificate logging | `set ssl-server-cert-log enable`   |
| SSL handshake logging          | `set ssl-handshake-log enable`     |

---

# 🎯 Final Mental Model

```text
                 SSH SESSION
                     │
                     ▼
              Firewall Policy
                     │
                     ▼
             UTM / Proxy Mode
                     │
                     ▼
               SSH Filter
                │        │
                │        └── Command/Shell Logging
                │
                └── Shell Blocking
                     │
                     ▼
                  Logging
```

For encrypted web/TLS traffic:

```text
                  TLS SESSION
                      │
                      ▼
               SSL/SSH Profile
                      │
                      ▼
                Re-Sign Mode
                      │
              ┌───────┴────────┐
              ▼                ▼
       SSL Inspection      Certificate
              │              Analysis
              ▼
       SSL/TLS Logging
```

> **Core security principle:**
> **SSH filtering controls what the session can do; SSL/SSH inspection determines how encrypted traffic is inspected; logging provides the evidence needed for auditing and troubleshooting.**
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

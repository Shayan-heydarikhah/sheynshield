# 🔐 FortiGate SSH Filter & SSL Inspection Checklist

> **FortiOS 7.2.x | SSH Filtering | Shell Control | Command Logging | SSL/TLS Inspection | Certificate Analysis | Troubleshooting**

[![FortiOS](https://img.shields.io/badge/FortiOS-7.2.x-red)](#)
[![Fortinet](https://img.shields.io/badge/Vendor-Fortinet-orange)](#)
[![Security](https://img.shields.io/badge/Focus-Network%20Security-blue)](#)
[![SheynShield](https://img.shields.io/badge/SheynShield-Security%20Engineering-black)](#)

---

## 📌 Table of Contents

* [1. SSH Filter Pre-Deployment Checklist](#1-ssh-filter-pre-deployment-checklist)
* [2. SSH Filter Profile Checklist](#2-ssh-filter-profile-checklist)
* [3. SSH Shell Control Checklist](#3-ssh-shell-control-checklist)
* [4. SSH Command Logging Checklist](#4-ssh-command-logging-checklist)
* [5. Firewall Policy Integration Checklist](#5-firewall-policy-integration-checklist)
* [6. SSL/SSH Inspection Profile Checklist](#6-sslssh-inspection-profile-checklist)
* [7. SSL Re-Sign Checklist](#7-ssl-re-sign-checklist)
* [8. SSL/TLS Logging Checklist](#8-ssltls-logging-checklist)
* [9. Certificate Inspection Checklist](#9-certificate-inspection-checklist)
* [10. End-to-End Validation Checklist](#10-end-to-end-validation-checklist)
* [11. SSH Troubleshooting Checklist](#11-ssh-troubleshooting-checklist)
* [12. SSL/TLS Troubleshooting Checklist](#12-ssltls-troubleshooting-checklist)
* [13. Security Hardening Checklist](#13-security-hardening-checklist)
* [14. NSE High-Value Recall](#14-nse-high-value-recall)
* [15. CLI Quick Reference](#15-cli-quick-reference)
* [16. Production Checklist](#16-production-checklist)
* [17. Final Mental Model](#17-final-mental-model)
* [18. SheynShield Resources](#18-sheynshield-resources)

---

# 1. SSH Filter Pre-Deployment Checklist

Before configuring SSH filtering, verify the traffic path.

### Traffic Identification

* [ ] Identify SSH clients.
* [ ] Identify SSH servers.
* [ ] Identify source interface.
* [ ] Identify destination interface.
* [ ] Identify source addresses.
* [ ] Identify destination addresses.
* [ ] Confirm TCP/22 or the required SSH service.
* [ ] Confirm the traffic actually crosses the FortiGate.
* [ ] Identify the firewall policy processing the traffic.
* [ ] Confirm the policy is using the expected inspection mode.

### Policy Inspection

* [ ] Confirm UTM inspection is enabled.
* [ ] Confirm proxy inspection is being used where required.
* [ ] Confirm the SSH filter profile is attached to the correct policy.
* [ ] Confirm there is no earlier policy matching the traffic.
* [ ] Confirm the SSH traffic is not bypassing the intended inspection path.

### Baseline

* [ ] Test SSH before enabling filtering.
* [ ] Record normal SSH behavior.
* [ ] Record expected shell behavior.
* [ ] Record expected logging behavior.
* [ ] Define what should be blocked.
* [ ] Define what should be logged.

---

# 2. SSH Filter Profile Checklist

Create and configure the SSH filter profile.

```bash
config ssh-filter profile
    edit ssh-test
        set block shell
        set log shell
        set default-command-log disable
    next
end
```

### Profile Validation

* [ ] SSH filter profile exists.
* [ ] Correct profile name is used.
* [ ] Shell blocking requirement is defined.
* [ ] Shell logging requirement is defined.
* [ ] Command logging behavior is explicitly reviewed.
* [ ] Profile is attached to the intended firewall policy.

### Core Parameters

| Parameter             | Purpose                             |
| --------------------- | ----------------------------------- |
| `block shell`         | Block interactive SSH shell         |
| `log shell`           | Log shell-related activity          |
| `default-command-log` | Control default SSH command logging |

---

# 3. SSH Shell Control Checklist

## Block Interactive Shell

```bash
set block shell
```

Validate:

* [ ] Interactive shell access is actually blocked.
* [ ] SSH connectivity itself remains available if required.
* [ ] Required SSH functionality still works.
* [ ] Administrative/application requirements have been tested.
* [ ] The restriction does not unintentionally block legitimate SSH use.

### Critical Concept

```text
block shell
     ≠
block SSH
```

`block shell` targets the interactive shell functionality rather than necessarily blocking the entire SSH session.

### Security Model

```text
SSH
 │
 ├── Required Functionality
 │       └── ALLOW
 │
 └── Interactive Shell
         └── BLOCK
```

* [ ] Permit only required SSH functionality.
* [ ] Avoid unrestricted SSH capability where unnecessary.
* [ ] Test both permitted and prohibited behavior.

---

# 4. SSH Command Logging Checklist

## Shell Logging

```bash
set log shell
```

* [ ] Shell activity logging requirement defined.
* [ ] Shell logging enabled where required.
* [ ] Generated logs are visible at the configured destination.
* [ ] Log volume has been evaluated.

## Command Logging

Review:

```bash
set default-command-log <value>
```

* [ ] Command logging behavior understood.
* [ ] Required commands are captured where supported.
* [ ] Logging requirements comply with organizational policy.
* [ ] Log retention is sufficient for auditing.
* [ ] Command logs can be correlated with the originating session.

### Audit Model

```text
SSH Connection
      │
      ├── User
      ├── Session
      ├── Shell Activity
      ├── Commands
      └── Timestamp
              │
              ▼
            Logs
```

### Security Use Cases

* [ ] Privileged SSH activity auditing.
* [ ] Remote administration monitoring.
* [ ] Forensic investigation.
* [ ] Insider-threat investigation.
* [ ] Compliance auditing.
* [ ] Security-event correlation.

> **Remember:** `log shell` and `block shell` are independent concepts. A shell can be restricted while relevant activity is still logged.

---

# 5. Firewall Policy Integration Checklist

SSH filtering must be integrated into the firewall policy processing path.

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

### Mandatory Validation

* [ ] Correct firewall policy identified.
* [ ] `utm-status` enabled where required.
* [ ] `inspection-mode proxy` configured where required.
* [ ] Correct SSH filter profile attached.
* [ ] Correct SSL/SSH profile attached when TLS inspection is required.
* [ ] Policy order verified.
* [ ] Policy hit counters verified.
* [ ] Test traffic matches the expected policy.

### Key CLI

```bash
set utm-status enable
set inspection-mode proxy
set ssh-filter-profile <profile>
set ssl-ssh-profile <profile>
```

### Troubleshooting Priority

If SSH filtering does not work:

```text
1. Check Policy Match
       ↓
2. Check UTM Status
       ↓
3. Check Inspection Mode
       ↓
4. Check SSH Filter Profile
       ↓
5. Check SSH Traffic
```

> **NSE Tip:** Do not start troubleshooting by changing the SSH profile. First prove that the traffic is actually being inspected by the intended firewall policy.

---

# 6. SSL/SSH Inspection Profile Checklist

Create or validate the SSL/SSH inspection profile.

```bash
config firewall ssl-ssh-profile
    edit cus-deep-test
        set server-cert-mode re-sign
        set caname Fortinet_CA_SSL
        set untrusted-caname Fortinet_CA_Untrusted
        set ssl-anomaly-log enable
        set ssl-exemption-log enable
        set ssl-negotiation-log enable
        set ssl-server-cert-log enable
        set ssl-handshake-log enable
    next
end
```

### Profile Checklist

* [ ] SSL/SSH profile exists.
* [ ] Correct inspection mode selected.
* [ ] Certificate handling reviewed.
* [ ] Trusted CA configured.
* [ ] Untrusted CA behavior reviewed.
* [ ] SSL logging requirements reviewed.
* [ ] SSL profile attached to the correct firewall policy.
* [ ] Client trust requirements validated.

---

# 7. SSL Re-Sign Checklist

## Re-Sign Mode

```bash
set server-cert-mode re-sign
```

Re-sign inspection concept:

```text
             TLS Session #1
Client ──────────────────────► FortiGate
                                │
                                │ TLS Session #2
                                ▼
                              Server
```

FortiGate becomes the inspection point between the client and server.

### Certificate Authority Checklist

```bash
set caname Fortinet_CA_SSL
set untrusted-caname Fortinet_CA_Untrusted
```

* [ ] Trusted CA exists.
* [ ] CA is trusted by client devices.
* [ ] CA certificate is deployed where required.
* [ ] Certificate chain is valid.
* [ ] Client certificate warnings tested.
* [ ] Untrusted certificate behavior tested.
* [ ] Certificate replacement behavior understood.

### Production Validation

* [ ] Windows clients tested.
* [ ] Linux clients tested where applicable.
* [ ] Mobile clients tested where applicable.
* [ ] Browsers tested.
* [ ] Applications using certificate pinning tested.
* [ ] Exceptions documented where required.

> **Important:** TLS inspection can break applications that use certificate pinning, custom trust stores, mutual TLS, or other non-standard certificate validation mechanisms.

---

# 8. SSL/TLS Logging Checklist

Enable logging intentionally.

## SSL Anomaly

```bash
set ssl-anomaly-log enable
```

* [ ] TLS anomalies need visibility.
* [ ] Anomaly logs are reaching the logging destination.
* [ ] Log volume evaluated.

## SSL Exemption

```bash
set ssl-exemption-log enable
```

* [ ] SSL inspection exemptions are logged.
* [ ] Exemption reasons can be investigated.
* [ ] Unexpected bypasses can be identified.

## SSL Negotiation

```bash
set ssl-negotiation-log enable
```

* [ ] TLS negotiation events available.
* [ ] Negotiation failures can be investigated.

## SSL Server Certificate

```bash
set ssl-server-cert-log enable
```

* [ ] Server certificate information logged.
* [ ] Certificate-related failures can be correlated.

## SSL Handshake

```bash
set ssl-handshake-log enable
```

* [ ] TLS handshake failures can be investigated.
* [ ] Handshake logging enabled during troubleshooting where appropriate.

### Deep TLS Troubleshooting Set

```text
[ ] SSL Anomaly Log
[ ] SSL Exemption Log
[ ] SSL Negotiation Log
[ ] SSL Server Certificate Log
[ ] SSL Handshake Log
```

> **Production Warning:** Extensive SSL logging can increase log volume significantly. Enable detailed logging deliberately rather than permanently on high-volume systems.

---

# 9. Certificate Inspection Checklist

When investigating certificate-related problems, inspect:

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

### Certificate Validation

* [ ] `notbefore` valid.
* [ ] `notafter` has not expired.
* [ ] Issuer trusted.
* [ ] CN reviewed.
* [ ] SAN reviewed.
* [ ] Serial number recorded.
* [ ] SKI reviewed where necessary.
* [ ] Certificate fingerprint/hash recorded.
* [ ] Public-key algorithm reviewed.
* [ ] Key size reviewed.
* [ ] Certificate chain validated.

### Quick Reference

| Field       | Check                        |
| ----------- | ---------------------------- |
| `notbefore` | Validity start               |
| `notafter`  | Expiration                   |
| `issuer`    | Issuing CA                   |
| `cn`        | Common Name                  |
| `san`       | Subject Alternative Name     |
| `sn`        | Serial number                |
| `ski`       | Subject Key Identifier       |
| `certhash`  | Certificate fingerprint/hash |
| `keyalgo`   | Public-key algorithm         |
| `keysize`   | Public-key size              |

### Certificate Investigation Workflow

```text
Certificate Error
       │
       ▼
Validity
       │
       ▼
Expiration
       │
       ▼
Issuer
       │
       ▼
CN / SAN
       │
       ▼
Certificate Chain
       │
       ▼
Algorithm / Key Size
       │
       ▼
Client Trust
```

---

# 10. End-to-End Validation Checklist

## SSH Validation

* [ ] SSH connection can be established.
* [ ] Correct firewall policy is matched.
* [ ] UTM inspection is active.
* [ ] Proxy inspection is active where required.
* [ ] SSH filter profile is applied.
* [ ] Interactive shell behaves as expected.
* [ ] Required SSH functionality remains available.
* [ ] Shell activity is logged where configured.
* [ ] Command logging behaves as expected.
* [ ] Logs reach the configured destination.

## SSL/TLS Validation

* [ ] HTTPS/TLS traffic matches the intended policy.
* [ ] SSL/SSH profile is attached.
* [ ] Re-sign certificate is presented when expected.
* [ ] Client trusts the inspection CA.
* [ ] TLS handshake succeeds.
* [ ] SSL logs are generated.
* [ ] SSL exemptions behave as expected.
* [ ] Certificate information is visible.
* [ ] Applications continue to function.

---

# 11. SSH Troubleshooting Checklist

## Symptom: SSH Filter Has No Effect

Check in this order:

```text
[ ] Is traffic crossing the FortiGate?
[ ] Is the correct firewall policy matched?
[ ] Is UTM enabled?
[ ] Is proxy inspection being used?
[ ] Is the SSH filter attached?
[ ] Is the SSH profile configured correctly?
[ ] Is the traffic actually identified as SSH?
[ ] Is another policy matching first?
```

### Troubleshooting Model

```text
SSH Not Filtered
      │
      ▼
Policy Match?
  │         │
 NO        YES
  │         │
Fix       UTM?
Policy      │
           ▼
       Proxy Mode?
           │
           ▼
      SSH Filter?
           │
           ▼
       Test Again
```

---

# 12. SSL/TLS Troubleshooting Checklist

## Symptom: HTTPS/TLS Application Fails

Check:

* [ ] Correct firewall policy.
* [ ] SSL/SSH profile attached.
* [ ] Inspection mode correct.
* [ ] CA trusted by client.
* [ ] Certificate chain valid.
* [ ] Certificate not expired.
* [ ] SAN matches expected hostname.
* [ ] TLS negotiation succeeds.
* [ ] SSL handshake logs checked.
* [ ] SSL anomaly logs checked.
* [ ] SSL exemption logs checked.
* [ ] Application certificate pinning evaluated.
* [ ] Mutual TLS requirements evaluated.
* [ ] Application-specific exceptions evaluated.

### TLS Troubleshooting Flow

```text
Application Failure
        │
        ▼
Check Policy
        │
        ▼
Check SSL/SSH Profile
        │
        ▼
Check CA Trust
        │
        ▼
Check Certificate
        │
        ▼
Check TLS Negotiation
        │
        ▼
Check Handshake Logs
        │
        ▼
Check SSL Anomaly
        │
        ▼
Check Application Compatibility
```

---

# 13. Security Hardening Checklist

## SSH Access Control

* [ ] Restrict SSH source networks.
* [ ] Restrict SSH destination networks.
* [ ] Avoid unnecessary `service all`.
* [ ] Use dedicated firewall policies where practical.
* [ ] Restrict administrative SSH access.
* [ ] Apply least privilege.
* [ ] Avoid unrestricted interactive shells when unnecessary.

## SSH Monitoring

* [ ] Enable shell logging when required.
* [ ] Enable command logging where justified.
* [ ] Forward security-relevant logs to centralized logging.
* [ ] Define retention requirements.
* [ ] Correlate SSH activity with user identity and timestamps.

## SSL Inspection

* [ ] Use trusted enterprise CA infrastructure.
* [ ] Protect private CA keys.
* [ ] Deploy CA certificates securely.
* [ ] Document SSL inspection exceptions.
* [ ] Review certificate-pinning exceptions.
* [ ] Monitor SSL anomalies.
* [ ] Monitor unexpected exemptions.
* [ ] Avoid excessive SSL logging in production.

## Privacy & Compliance

* [ ] Define who can access SSH command logs.
* [ ] Define retention periods.
* [ ] Protect sensitive command information.
* [ ] Apply organizational privacy requirements.
* [ ] Review applicable regulatory requirements.

---

# 14. NSE High-Value Recall 🧠

### SSH Filter

```bash
config ssh-filter profile
```

### Block Shell

```bash
set block shell
```

> **Block shell ≠ Block SSH**

### Log Shell

```bash
set log shell
```

### Firewall Policy

```bash
set utm-status enable
set inspection-mode proxy
set ssh-filter-profile <profile>
```

### SSL/SSH Profile

```bash
config firewall ssl-ssh-profile
```

### Re-Sign

```bash
set server-cert-mode re-sign
```

### Trusted CA

```bash
set caname <CA>
```

### SSL Logging

```bash
set ssl-anomaly-log enable
set ssl-exemption-log enable
set ssl-negotiation-log enable
set ssl-server-cert-log enable
set ssl-handshake-log enable
```

### NSE Mental Triggers

| Question                        | Recall                          |
| ------------------------------- | ------------------------------- |
| Block interactive SSH shell?    | `block shell`                   |
| Log shell activity?             | `log shell`                     |
| SSH filter not working?         | Check policy + UTM + proxy mode |
| Inspect TLS?                    | SSL/SSH profile                 |
| Re-sign certificate?            | `server-cert-mode re-sign`      |
| Trust inspection CA?            | `caname`                        |
| Investigate TLS handshake?      | `ssl-handshake-log`             |
| Investigate TLS anomaly?        | `ssl-anomaly-log`               |
| Investigate exemptions?         | `ssl-exemption-log`             |
| Investigate server certificate? | `ssl-server-cert-log`           |

---

# 15. CLI Quick Reference

## SSH Filter

```bash
config ssh-filter profile
    edit ssh-test
        set block shell
        set log shell
        set default-command-log disable
    next
end
```

## Firewall Policy

```bash
config firewall policy
    edit <policy-id>
        set utm-status enable
        set inspection-mode proxy
        set ssh-filter-profile <profile>
        set ssl-ssh-profile <profile>
    next
end
```

## SSL/SSH Profile

```bash
config firewall ssl-ssh-profile
    edit <profile>
        set server-cert-mode re-sign
        set caname <CA>
        set untrusted-caname <CA>
        set ssl-anomaly-log enable
        set ssl-exemption-log enable
        set ssl-negotiation-log enable
        set ssl-server-cert-log enable
        set ssl-handshake-log enable
    next
end
```

---

# 16. Production Checklist 🚀

## SSH Filtering

```text
[ ] SSH traffic identified
[ ] Correct firewall policy identified
[ ] Policy order verified
[ ] UTM enabled
[ ] Proxy inspection verified
[ ] SSH filter profile created
[ ] SSH filter attached
[ ] Shell restriction requirement defined
[ ] Shell blocking tested
[ ] Shell logging evaluated
[ ] Command logging evaluated
[ ] SSH logs verified
```

## SSL Inspection

```text
[ ] SSL/SSH profile created
[ ] Correct inspection mode selected
[ ] Re-sign requirement evaluated
[ ] Trusted CA configured
[ ] CA deployed to clients
[ ] Untrusted certificate behavior tested
[ ] SSL anomaly logging evaluated
[ ] SSL exemption logging evaluated
[ ] SSL negotiation logging evaluated
[ ] SSL certificate logging evaluated
[ ] SSL handshake logging evaluated
[ ] Application compatibility tested
[ ] Certificate-pinning exceptions documented
```

## Operational Validation

```text
[ ] Policy hit verified
[ ] SSH session tested
[ ] Interactive shell tested
[ ] Command logging tested
[ ] TLS session tested
[ ] Certificate validated
[ ] SSL logs verified
[ ] Central logging verified
[ ] Log volume evaluated
[ ] Troubleshooting procedure documented
```

---

# 17. Final Mental Model

## SSH

```text
                  SSH SESSION
                       │
                       ▼
                Firewall Policy
                       │
                ┌──────┴──────┐
                │             │
             UTM ON       Proxy Mode
                │             │
                └──────┬──────┘
                       │
                       ▼
                  SSH Filter
                 ┌─────┴─────┐
                 │           │
                 ▼           ▼
            Shell Block   Logging
                             │
                             ▼
                      Command/Shell
                           Logs
```

## SSL/TLS

```text
                  TLS SESSION
                       │
                       ▼
                Firewall Policy
                       │
                       ▼
                 SSL/SSH Profile
                       │
                       ▼
                   Re-Sign
                       │
              ┌────────┴────────┐
              ▼                 ▼
       TLS Inspection      Certificate
              │              Analysis
              ▼                 │
        SSL/TLS Logs ◄──────────┘
```

## Security Model

```text
              ENCRYPTED / SSH TRAFFIC
                        │
                        ▼
                 Firewall Policy
                        │
            ┌───────────┴───────────┐
            ▼                       ▼
        SSH Filter             SSL/SSH Profile
            │                       │
       ┌────┴────┐             ┌────┴────┐
       ▼         ▼             ▼         ▼
    Control    Audit        Inspect    Validate
       │         │             │         │
       └─────────┴─────────────┴─────────┘
                         │
                         ▼
                       Logs
```

### 🔥 Golden Rule

> **SSH Filter = Control SSH session behavior**
>
> **SSL/SSH Profile = Control encrypted-traffic inspection**
>
> **Logging = Create the evidence required for troubleshooting, auditing and detection**

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

## 🧠 SheynShield Takeaway

> **Secure SSH and TLS traffic by separating three responsibilities:**
>
> **ACCESS → CONTROL → VISIBILITY**
>
> **Firewall Policy** controls *who can communicate.*
>
> **SSH Filter** controls *what an SSH session can do.*
>
> **SSL/SSH Inspection** controls *how encrypted traffic is inspected.*
>
> **Logging** provides *the evidence required to troubleshoot, audit and investigate.*
>
> The production mindset is therefore:
>
> **Identify → Inspect → Restrict → Log → Validate → Monitor**

# FortiGate System Settings — FortiOS 7.2.x

> **SheynShield | Engineering Secure Networks**
> **FortiGate NSE4 / NSE7 Technical **
>
> A practical reference for **FortiOS system settings, NTP/PTP, administrative hardening, Auxiliary Sessions, TPM, SNMP, FortiGuard, certificates, REST API, Workspace Mode and configuration management**.

---

## 📌 Table of Contents

* [1. Initial System Settings](#1-initial-system-settings)
* [2. NTP & PTP](#2-ntp--ptp)
* [3. Administrative Ports](#3-administrative-ports)
* [4. Admin Lockout & Login Security](#4-admin-lockout--login-security)
* [5. TLS / SSL Minimum Version](#5-tls--ssl-minimum-version)
* [6. Auxiliary Sessions](#6-auxiliary-sessions)
* [7. Configuration Save / Auto-Revert](#7-configuration-save--auto-revert)
* [8. Trusted Platform Module — TPM](#8-trusted-platform-module--tpm)
* [9. SNMP Monitoring](#9-snmp-monitoring)
* [10. Replacement Messages](#10-replacement-messages)
* [11. CLI Configuration Scripts](#11-cli-configuration-scripts)
* [12. FortiGate Cipher Suites](#12-fortigate-cipher-suites)
* [13. FortiGuard](#13-fortiguard)
* [14. FortiGuard Anycast & Certificate Validation](#14-fortiguard-anycast--certificate-validation)
* [15. FortiManager as Local FortiGuard Server](#15-fortimanager-as-local-fortiguard-server)
* [16. FortiGuard Through Explicit Proxy](#16-fortiguard-through-explicit-proxy)
* [17. FDS-Only ISDB](#17-fds-only-isdb)
* [18. Air-Gap Licensing](#18-air-gap-licensing)
* [19. IoT Detection Service](#19-iot-detection-service)
* [20. Certificates & PKI](#20-certificates--pki)
* [21. ACME / Let's Encrypt](#21-acme--lets-encrypt)
* [22. REST API](#22-rest-api)
* [23. HTTPS Daemon Troubleshooting](#23-https-daemon-troubleshooting)
* [24. Workspace Mode](#24-workspace-mode)
* [25. High-Value Troubleshooting Commands](#25-high-value-troubleshooting-commands)
* [26. NSE Exam Quick Review](#26-nse-exam-quick-review)

---

# 1. Initial System Settings

## 🔐 Default Administrator Password

At initial setup, immediately change the default administrator password.

### Important

* The new password normally cannot be identical to the old password.
* Password reuse behavior can be affected by the configured password policy.
* Never leave the default administrative credential active on a production FortiGate.

### Recommended hardening

```text
Default admin
      ↓
Change password
      ↓
Restrict management interfaces
      ↓
Use trusted management networks
      ↓
Enable MFA where appropriate
      ↓
Monitor authentication logs
```

---

# 2. NTP & PTP

## ⏱️ Why Accurate Time Matters

Accurate system time is foundational for many FortiOS functions:

* Logging
* Scheduled policies
* Certificate validation
* SSL/TLS
* Authentication
* FortiGuard communication
* Security event correlation
* HA troubleshooting
* SIEM correlation

> **Golden Rule:** Set the correct timezone and synchronize time immediately after the initial FortiGate deployment.

---

## NTP

**NTP — Network Time Protocol**

Typical NTP synchronization operates on the order of minutes depending on configuration and environment.

### Check system time

```bash
execute time
execute date
```

### Configure timezone

```bash
config system global
    set timezone 41
    set dst enable
end
```

> `timezone 41` is commonly used for Tehran in FortiOS timezone mappings. Verify the correct timezone ID for your deployment.

---

## Configure Custom NTP Server

```bash
config system ntp
    set type custom

    config ntpserver
        edit 1
            set server ntp.day.ir
            set ntpv3 enable
            set authentication enable
            set key-type md5
            set key-id 123
        next
    end
end
```

### NTP Version Note

```text
ntpv3 enable
    ↓
NTPv3

ntpv3 disable
    ↓
NTPv4
```

### NTP Authentication

Can be used to authenticate NTP communication.

Common algorithm examples:

```text
MD5
SHA-based authentication
```

> Prefer authenticated NTP when appropriate, especially in controlled enterprise environments. Avoid exposing unnecessary NTP services to untrusted networks.

---

## PTP — Precision Time Protocol

**PTP** provides significantly more precise clock synchronization than traditional NTP and is useful in highly time-sensitive environments.

Example:

```bash
config system ptp
    set interface port1
end
```

### PTP Delay Mechanisms

Two important concepts:

```text
End-to-End
Peer-to-Peer
```

### End-to-End

The delay is calculated using timestamps exchanged between the master and slave.

Simplified delay calculation:

```text
Delay ≈ ((t2 - t1) + (t4 - t3)) / 2
```

Where:

```text
t1 = Sync transmission
t2 = Sync reception
t3 = Delay Request transmission
t4 = Delay Response reception
```

---

## NTP Troubleshooting

```bash
diagnose sys ntp status
```

### PTP debug

```bash
diagnose debug application ptpd -1
```

### Test UDP/123 reachability

```bash
execute telnet 192.168.20.200 123
```

> UDP/123 is used by NTP. A generic telnet test is not a definitive NTP protocol test, so use protocol-specific diagnostics where possible.

---

## Windows NTP Server Checklist

On a Windows NTP server:

### Service

```text
services.msc
    ↓
Windows Time
    ↓
Restart / verify running
```

### Check listening sockets

```cmd
netstat -nao
```

### Windows Group Policy

```text
Computer Configuration
  >
Administrative Templates
  >
System
  >
Windows Time Service
  >
Time Providers
```

Verify:

* Windows NTP Client
* NTP configuration
* Announce Flags
* Time Provider settings

---

# 3. Administrative Ports

FortiGate administrative ports can be customized.

```bash
config system global
    set admin-port 80
    set admin-sport 443
    set admin-https-redirect enable

    set admin-ssh-port 2142
    set admin-telnet-port 2323
end
```

### Recommended

```text
HTTP     → redirect to HTTPS
HTTPS    → custom management port if required
SSH      → custom port if required
TELNET   → avoid in production
```

> Changing the port is **not a replacement for access control**. Restrict the management plane using trusted interfaces, source IPs and administrative access policies.

---

## Service Source Port

```bash
config system global
    set default-service-source-port 20-30
end
```

This controls the default source-port range used for applicable service traffic.

---

# 4. Admin Lockout & Login Security

Protect the management plane against password guessing.

```bash
config system global
    set admin-lockout-threshold 2
    set admin-lockout-duration 60
end
```

### Meaning

```text
2 failed attempts
      ↓
Admin account locked
      ↓
60-second lockout
```

### Check login events

```text
Log & Report
  >
System Events
  >
General System Events
```

---

## Session Timeout

For administrative security, use an appropriate idle timeout.

```text
Recommended operational target:
~10 minutes
```

Balance security with administrator usability.

---

# 5. TLS / SSL Minimum Version

## Global Minimum TLS

```bash
config system global
    set ssl-min-proto-version tlsv1-2
end
```

### Security principle

Prefer:

```text
TLS 1.2+
```

where supported.

> TLS requirements are feature-specific. Do not assume changing the global minimum TLS setting changes every FortiOS service in exactly the same way.

---

## Email Server TLS

Example:

```bash
config system email-server
    set ssl-min-proto-version tlsv1
end
```

> Use the strongest protocol supported by the remote service. Legacy TLS versions should only be enabled when required for compatibility.

---

## Services Using TLS / Certificates

Examples include:

* Email
* FortiGuard
* FortiSandbox
* FortiAnalyzer
* Syslog
* LDAP
* User Authentication
* POP3
* Exchange
* Certificate-dependent services

---

# 6. Auxiliary Sessions

## 🔥 What Is an Auxiliary Session?

Auxiliary Sessions allow FortiGate to handle traffic where the **incoming or return interface/path of an existing session changes**.

This becomes especially important with:

* ECMP
* Multiple ISPs
* Asymmetric routing
* Policy-based routing
* SD-WAN
* Load balancing
* Complex multi-path topologies

Fortinet documents Auxiliary Sessions as a mechanism for handling changes to incoming/outgoing interfaces and traffic paths.

---

## Version History

| FortiOS         | Auxiliary Session                   |
| --------------- | ----------------------------------- |
| 6.0 and earlier | Not supported                       |
| 6.2.0–6.2.2     | Permanently enabled                 |
| 6.2.3+          | Disabled by default; can be enabled |

---

## Configuration

```bash
config system settings
    set auxiliary-session enable
end
```

Disable:

```bash
config system settings
    set auxiliary-session disable
end
```

### Important

Do not enable Auxiliary Sessions blindly.

Fortinet specifically warns that return-path symmetry must be considered in environments such as **SD-WAN Hub-and-Spoke and ADVPN**.

---

# Auxiliary Session vs Symmetric Routing

### Without Auxiliary Sessions

A session may become:

```text
Session
   ↓
Interface changes
   ↓
Session becomes dirty
   ↓
Session refresh/update
   ↓
Potential CPU processing increase
```

### With Auxiliary Sessions

```text
Existing Session
      ↓
Path/interface changes
      ↓
Auxiliary Session created
      ↓
Traffic can continue
      ↓
Potential NPU offload remains possible
```

---

## Why This Matters for NPU

A major performance concept:

```text
Initial packet
      ↓
CPU
      ↓
Session lookup / creation
      ↓
Policy
      ↓
Routing
      ↓
Session established
      ↓
NPU may offload eligible traffic
      ↓
High-speed forwarding
```

The CPU is still involved in control/session establishment, while the NPU can accelerate eligible data-plane forwarding.

---

## Session Diagnostic

```bash
diagnose sys session list
```

Look for:

```text
state=dirty
```

A dirty session indicates that FortiGate needs to update session-related information, such as interface/path state.

---

## NPU Information

Example:

```text
npu info:
flag=0x91/0x81
offload=8/8
```

Interpretation depends on the complete session output and FortiOS/model.

Look at:

```text
npu_state
npu info
offload
in_npu
out_npu
```

---

# Auxiliary Session Scenarios

## Scenario 1 — Return Traffic Uses Original Interface

```text
Client
  |
port1
  |
FortiGate
  |
port3
  |
Server
```

Return:

```text
Server
  |
port3
  |
FortiGate
  |
port1
  |
Client
```

If the return path remains symmetric, Auxiliary Sessions may not be necessary.

---

## Scenario 2 — Return Traffic Uses Another Interface

```text
Client
   |
 port1
   ↓
FortiGate
   ↓
 port3
   |
 Server
```

Return:

```text
Server
   |
 port4
   ↓
FortiGate
```

### Without Auxiliary Session

```text
Existing session
      ↓
Interface changed
      ↓
Session dirty/refresh
      ↓
CPU involvement increases
```

### With Auxiliary Session

```text
Existing session
      ↓
Interface changed
      ↓
Auxiliary session
      ↓
Traffic continues
```

---

## Scenario 3 — Incoming Interface Changes

```text
Original:
port1 → FortiGate → port3

New packet:
port2 → FortiGate → port3
```

With Auxiliary Sessions:

```text
Existing session
      ↓
Different ingress
      ↓
Auxiliary session
      ↓
Forward traffic
```

---

## Scenario 4 — Routing Table Changes

Suppose:

```text
Original:
port1 → FortiGate → port3
```

A better route becomes available:

```text
port4 → destination
```

### Auxiliary disabled

The existing session can continue using the original path as long as the destination remains reachable.

### Auxiliary enabled

The session may be refreshed to account for changed routing/interface conditions.

---

# Auxiliary Session + ECMP

This is one of the important NSE concepts.

```text
                 ISP-1
                  |
                  |
Client ── FortiGate
                  |
                  |
                 ISP-2
```

The same logical traffic flow may encounter different ingress/egress interfaces because of ECMP.

Fortinet documents Auxiliary Sessions as a mechanism that can allow TCP traffic to continue when ECMP causes traffic for the same session to enter/exit through different interfaces.

---

# Asymmetric Routing

FortiOS also has asymmetric-routing-related settings.

In FortiOS 7.2 CLI reference:

```bash
config system settings
    set auxiliary-session enable
    set asymroute enable
end
```

The CLI reference identifies `auxiliary-session` and `asymroute` as separate settings.

> **Exam trap:** Auxiliary Session ≠ Asymmetric Routing. They solve related but different problems.

---

# Recommended Mental Model

```text
                 ┌──────────────┐
Incoming Packet ─►     CPU      │
                 │              │
                 │ Session      │
                 │ Policy       │
                 │ Routing      │
                 └──────┬───────┘
                        │
                 Session Established
                        │
                        ▼
                 ┌──────────────┐
                 │     NPU      │
                 │ Fast Forward │
                 └──────┬───────┘
                        │
                        ▼
                   Outgoing
```

With changing paths:

```text
Existing Session
       │
       ├── Same path
       │      ↓
       │   Normal forwarding
       │
       └── Path/interface changes
              ↓
       Auxiliary Session
              ↓
       New path tracking
```

---

# 7. Configuration Save / Auto-Revert

FortiGate supports configuration save/revert behavior.

```bash
config system global
    set cfg-save revert
    set cfg-revert-timeout 600
end
```

### Meaning

```text
Configuration change
       ↓
Temporary configuration
       ↓
600-second timer
       ↓
If not confirmed
       ↓
Configuration reverts
```

This is particularly useful for **remote administration** where a configuration change could accidentally disconnect management access.

---

## Save Configuration

```bash
execute cfg save
```

> Always verify the exact behavior of `cfg-save` for your FortiOS release before using it as a production change-control mechanism.

---

# 8. Trusted Platform Module — TPM

## What Is TPM?

A supported FortiGate hardware platform may use a **Trusted Platform Module** to protect sensitive cryptographic material.

Fortinet documents TPM as a hardware security mechanism that can generate, store and authenticate cryptographic keys.

---

## Private Data Encryption

Enable:

```bash
config system global
    set private-data-encryption enable
end
```

FortiOS prompts for a **32-hexadecimal-digit private/master encryption key**.

Fortinet documents AES-128-CBC protection of sensitive data and a TPM-generated RSA-2048 primary key on supported hardware.

---

## Important TPM Concepts

```text
Master Encryption Password
          ↓
      AES-128-CBC
          ↓
Sensitive configuration data

TPM
 ↓
RSA-2048 Primary Key
 ↓
Protects master encryption password
```

### Critical distinction

> **TPM does NOT encrypt the FortiGate disk drive.**

---

## Data Protected by Private Encryption

Examples include:

* IPsec PSKs
* HA passwords
* Local certificate private keys
* LDAP credentials
* RADIUS credentials
* FSSO credentials
* NTP authentication password
* SNMP credentials
* PPPoE credentials
* Modem credentials
* FortiGuard proxy credentials
* SDN connector credentials
* Link Monitor credentials
* Wireless security credentials

---

## TPM + Configuration Backup

Critical concept:

```text
TPM enabled
+
Protected configuration
+
Correct encryption key
        ↓
Configuration can be restored
```

If the required TPM/private encryption conditions are not met, restoration can fail.

---

## HA + TPM

In an HA cluster:

```text
FGT-A
  |
  | Same private encryption key
  |
FGT-B
```

All cluster members must use compatible/same encryption credentials so the cluster can synchronize protected configuration data.

---

## TPM Diagnostics

```bash
diagnose hardware test info
diagnose hardware deviceinfo tpm
diagnose tpm
```

---

# 9. SNMP Monitoring

## What Can SNMP Monitor?

Common resources:

* CPU
* RAM
* Interface status
* Interface bandwidth
* Uplinks
* Latency-related metrics
* Temperature
* System information
* Interface statistics
* Device health

---

## Enable SNMP on Management Interface

The relevant interface must permit SNMP management access:

```bash
config system interface
    edit port1
        set allowaccess ping https ssh snmp
    next
end
```

---

# MIB Files

## Fortinet Core MIB

```text
FORTINET-CORE-MIB
```

Contains information common across Fortinet products.

---

## FortiGate MIB

```text
FORTINET-FORTIGATE-MIB
```

Contains FortiGate-specific objects and traps.

---

## MIB-II

```text
RFC 1213
```

FortiGate supports MIB-II groups with documented limitations.

> For FortiGate-specific monitoring, the Fortinet MIBs generally provide more useful device-specific information than generic MIB-II statistics.

---

## Ethernet-like MIB

```text
RFC 2665
```

Provides Ethernet-like interface information.

---

# SNMPv1 / SNMPv2c

Example:

```bash
config system snmp community
    edit 2
        set name pub-com
        set status enable

        config hosts
            edit 1
                set ip 192.168.254.253
                set source-ip <source-ip>
                set ha-direct enable
                set host-type any
            next
        end

        set query-v1-status enable
        set query-v2c-status enable

        set trap-v1-status enable
        set trap-v2c-status enable
    next
end
```

### Host Type

Typical concepts:

```text
query
trap
any
```

---

# SNMPv3

SNMPv3 provides stronger security than community-string-based SNMPv1/v2c.

Example:

```bash
config system snmp user
    edit fgt-v3
        set status enable
        set trap-status enable
        set queries enable

        set notify-hosts 192.168.254.253
        set source-ip <source-ip>
        set ha-direct enable

        set security-level auth-priv

        set auth-proto sha256
        set auth-pwd <AUTH_PASSWORD>

        set priv-proto aes
        set priv-pwd <PRIV_PASSWORD>
    next
end
```

### Security Levels

```text
no-auth-no-priv
        ↓
auth-no-priv
        ↓
auth-priv
```

---

## Authentication Algorithms

Depending on FortiOS version:

```text
MD5
SHA
SHA224
SHA256
SHA384
SHA512
```

---

## Privacy Algorithms

Examples:

```text
DES
AES
AES256
AES256Cisco
```

> Availability is version/platform dependent.

---

# SNMP MIB Views

MIB Views restrict which OIDs an SNMP user/community can access.

```bash
config system snmp mib-view
    edit view1
        set include 1.3.6.1.2
    next

    edit view2
        set include 1.3.6.1.2.1
        set exclude 1.3.6.1.2.1.2.1
        set exclude 1.3.6.1.2.1.4.31
    next
end
```

### Concept

```text
MIB Tree
   |
   +── Include
   |
   +── Exclude
```

Useful for:

* Least privilege
* Monitoring segmentation
* Limiting visibility
* Delegated monitoring

---

# SNMP + VDOM

SNMP objects can be restricted by VDOM.

Concept:

```text
SNMP Community/User
       |
       +── MIB View
       |
       +── VDOM access
```

This is useful in multi-tenant environments.

---

# DHCP SNMP Events

SNMP can report DHCP-related events such as:

* DHCP pool usage reaching a threshold
* Duplicate IP detection
* DHCP NAK events

Example:

```bash
config system snmp community
    edit 1
        set name regr-sys

        config hosts
            edit 1
                set ip 10.1.100.11 255.255.255.255
            next
        end

        set events dhcp
    next
end
```

---

# 10. Replacement Messages

FortiGate replacement messages can customize messages presented to users by security features.

Supported image formats include:

```text
GIF
JPEG
TIFF
PNG
```

Maximum image size:

```text
24 KB
```

---

## Replacement Message Categories

Examples:

```text
UTM
Admin
Alert Mail
Custom Message
FortiGuard Web Filter
FTP
HTTP
ICAP
Mail
NAC Quarantine
Spam
SSL VPN
Traffic Quota
Web Proxy
Authentication
```

---

## Enable GUI Replacement Message Groups

```bash
config system settings
    set gui-replacement-message-groups enable
end
```

---

## Example

```bash
config emailfilter profile
    edit newmsgs
        set replacemsg-group newutm
    next
end
```

Firewall policy:

```bash
config firewall policy
    edit 1
        set replacemsg-override-group newauth
        set inspection-mode proxy
        set emailfilter-profile newmsgs
    next
end
```

---

# 11. CLI Configuration Scripts

FortiGate GUI provides a CLI/script mechanism for deploying configuration commands.

### Comment Syntax

```bash
# This is a comment
```

Anything after `#` on a comment line is not executed.

Example:

```bash
# Configure DNS
config system dns
    set primary 8.8.8.8
end
```

---

# 12. FortiGate Cipher Suites

FortiGate uses cryptographic protocols for multiple services, including:

* HTTPS administration
* SSH
* SSL VPN
* TLS-based integrations

---

# HTTPS Administration

```bash
config system global
    set strong-crypto enable
end
```

Depending on FortiOS version, additional HTTPS cipher/version controls are available.

Example concepts:

```bash
set admin-https-ssl-versions
set admin-https-ssl-ciphersuites
set admin-https-ssl-banned-ciphers
```

---

## Strong Crypto

With strong cryptography enabled, FortiOS restricts administrative HTTPS to stronger TLS versions according to the release's supported configuration.

> Always verify the exact cipher/version behavior against the FortiOS release because cipher controls evolve between versions.

---

# SSH

Example:

```bash
config system global
    set strong-crypto enable
end
```

SSH encryption algorithms can also be controlled through the relevant SSH configuration.

> **Security rule:** Do not enable SSHv1 unless there is a specific legacy requirement.

---

# SSL VPN

Example:

```bash
config vpn ssl setting
    set algorithm high
    set ssl-max-proto-ver tls1-3
    set ssl-min-proto-ver tls1-3
end
```

Cipher suites may be configured when TLS 1.3 is selected.

Example:

```bash
set ciphersuite TLS-AES-128-GCM-SHA256
```

Possible examples:

```text
TLS-AES-128-GCM-SHA256
TLS-AES-256-GCM-SHA384
TLS-CHACHA20-POLY1305-SHA256
TLS-AES-128-CCM-SHA256
TLS-AES-128-CCM-8-SHA256
```

---

## SSL VPN Algorithm Levels

```text
high
medium
low
```

Conceptually:

```text
HIGH
 ↓
High-strength algorithms

MEDIUM
 ↓
High + Medium

LOW
 ↓
Broader compatibility
```

> `strong-crypto` is not a universal control for SSL VPN cipher policy.

---

# 13. FortiGuard

FortiGuard provides security intelligence and subscription services such as:

* Antivirus
* IPS
* Application Control
* AntiSpam
* Web Filtering
* WAF-related services
* Threat intelligence
* IoT/device intelligence

FortiGate normally communicates with FortiGuard to validate licensing and retrieve updates.

---

# FortiGuard Update Architecture

```text
FortiGate
    |
    | HTTPS / other supported protocols
    ↓
FortiGuard
    |
    +── AV
    +── IPS
    +── Web Filtering
    +── AntiSpam
    +── Application signatures
    +── Threat intelligence
```

---

## Package Authentication

AV and IPS packages are signed by Fortinet.

Concept:

```text
Package
   ↓
Fortinet CA signature
   ↓
FortiGate validates package
   ↓
Accept / Warn / Reject
```

---

# Immediate Update

On supported hardware/platform configurations, FortiGate can maintain communication with FortiGuard to receive notifications of new updates and then retrieve them.

---

# Improve IPS Quality

This option can send attack-related information to FortiGuard to help improve threat intelligence and IPS signatures.

---

# Antivirus Grayware / PUP / PUA

Potentially unwanted applications can be detected using the appropriate AV grayware/PUP/PUA functionality where supported.

---

# FortiGuard Proxy Tunneling

Example:

```bash
config system autoupdate tunneling
    set address 1.2.3.4
    set port 1344
    set username <username>
    set password <password>
    set status enable
end
```

> Proxy tunneling support is service-specific. Do not assume every FortiGuard service can traverse the same proxy path.

---

# FortiGuard Update Scheduling

Example:

```bash
config system autoupdate schedule
    set status enable
    set frequency automatic
end
```

Update frequency can depend on FortiGuard service behavior, contract status, platform and configured scheduling.

---

## Automatic vs Scheduled

```text
Automatic
   ↓
FortiOS determines update timing

Scheduled
   ↓
Administrator controls update schedule
```

---

# Manual AV / IPS Update

Example:

```bash
execute restore ips tftp nids-720-19.261.pkg 192.168.20.200
```

Debug:

```bash
diagnose debug application updated -1
diagnose debug enable
```

Stop debugging:

```bash
diagnose debug disable
```

---

# Package Validation Levels

Conceptually:

```text
Level 0
    Accept unsigned package

Level 1
    Warning / confirmation

Level 2
    Reject unsigned package
```

> Security-sensitive environments should avoid weakening package verification merely to force an update.

---

# FortiGuard Update Server Location

FortiGuard can use:

```text
Automatic / Lowest Latency
USA
EU
```

Example:

```bash
config system fortiguard
    set update-server-location automatic
end
```

Fortinet documents Anycast and regional FortiGuard domains for these modes.

---

## Anycast

Common Anycast domains:

```text
globalupdate.fortinet.net
globalguardservice.fortinet.net
```

US:

```text
usupdate.fortinet.net
usguardservice.fortinet.net
```

EU:

```text
euupdate.fortinet.net
euguardservice.fortinet.net
```

---

# FortiGuard Protocol

Example:

```bash
config system fortiguard
    set protocol https
    set port 443
end
```

Commonly relevant ports/services include:

```text
HTTPS 443
DNS / service-specific UDP
```

> Exact ports depend on the FortiGuard service and FortiOS configuration.

---

# FortiGuard Cache

Web filtering and anti-spam can use local caches.

Example concepts:

```bash
config system fortiguard
    set antispam-cache enable
    set webfilter-cache enable
end
```

Caching can reduce repeated external rating requests.

---

# Sending Malware Statistics

FortiGate can periodically send encrypted malware/security statistics to FortiGuard.

Example:

```bash
config system global
    set fds-statistics enable
    set fds-statistics-period 60
end
```

`60` represents the configured period in minutes.

---

# FortiGuard Service Diagnostics

```bash
diagnose sys service-communication
```

Useful for understanding communication between FortiGate and external services.

Also inspect:

```text
Log & Report
  >
System Events
```

and:

```text
System
  >
FortiGuard
  >
Licenses
```

---

# 14. FortiGuard Anycast & Certificate Validation

Anycast improves global FortiGuard connectivity by using a shared Anycast destination and routing traffic toward an appropriate FortiGuard location.

Fortinet documents Anycast as the default FortiGuard access mode on FortiGate in the relevant 7.2 documentation.

---

## Anycast TLS Validation

Concept:

```text
FortiGate
   |
   | TLS ClientHello
   | OCSP status request
   ↓
FortiGuard
   |
   | Certificate + OCSP status
   ↓
FortiGate
   |
   +── Validate certificate
   +── Validate OCSP
   +── Validate hostname
   |
   ↓
TLS established
```

---

## Possible Abort Conditions

Examples:

```text
Certificate CN/SAN does not match expected hostname
        ↓
Abort

OCSP status invalid
        ↓
Abort

Issuer/CA trust validation fails
        ↓
Abort
```

---

## Configure Anycast

```bash
config system fortiguard
    set fortiguard-anycast enable
    set fortiguard-anycast-source fortinet
end
```

Depending on release/options, an alternate source such as AWS may be available.

---

# 15. FortiManager as Local FortiGuard Server

FortiManager can provide local FortiGuard services for environments where FortiGates should retrieve updates/rating locally.

Fortinet documents separate server types for **update** and **rating** services.

---

## Architecture

```text
                FortiManager
             ┌────────────────┐
             │ Update Server  │
             │ Rating Server  │
             └───────┬────────┘
                     |
                     |
                FortiGate
```

---

## Configuration

```bash
config system central-management
    set type fortimanager
    set fmg 192.168.254.200

    config server-list
        edit 1
            set server-type update
            set server-address 192.168.254.200
        next

        edit 2
            set server-type rating
            set server-address 192.168.254.200
        next
    end

    set fmg-update-port 443
    set include-default-servers enable
end
```

### Important

Fortinet documents that when `fmg-update-port` is `443`, the local update server uses HTTPS/443. If not configured, the traditional update mechanism may use port 8890 depending on the setup.

---

## Failback

```text
FortiManager
     |
     | Available?
     |
    YES ──► Local FortiGuard
     |
    NO
     ↓
Default FortiGuard servers
```

---

# 16. FortiGuard Through Explicit Proxy

FortiGate can use an explicit proxy for supported FortiGuard/FortiCloud communication.

Example:

```bash
config system fortiguard
    set proxy-server-ip 192.168.254.251
    set proxy-server-port 8080
    set proxy-username <username>
    set proxy-password <password>
end
```

---

## Example Architecture

```text
FortiGate
   |
   | Explicit Proxy
   ↓
Proxy FortiGate
   |
   ↓
Internet
   |
   ↓
FortiGuard
```

> Not every FortiGuard service uses the same proxy mechanism.

---

# 17. FDS-Only ISDB

FortiOS firmware includes a built-in Internet Service Database component.

The built-in package allows FortiGate policies referencing Fortinet services to remain functional before the full updated ISDB is downloaded.

Concept:

```text
Firmware
  |
  +── Lightweight ISDB
  |       ↓
  |    Basic services
  |
  +── Full ISDB
          ↓
      Full FortiGuard update
```

---

## View ISDB

```bash
diagnose firewall internet-service list
```

---

## Post-Upgrade Behavior

After a firmware upgrade/reboot, FortiGate can automatically retrieve an updated ISDB package.

Example:

```bash
diagnose autoupdate versions | grep internet -a 6
```

---

# 18. Air-Gap Licensing

Air-gapped environments may not have direct Internet access.

Typical use cases:

* OT
* Industrial networks
* Critical infrastructure
* Isolated labs
* High-security environments

---

## Air-Gap Model

```text
Internet-connected system
        |
        | Download license/update
        ↓
Offline transfer
        |
        ↓
Air-gapped FortiGate
```

Manual licensing support is platform and FortiOS-version dependent. Fortinet documents support for manual licensing on supported FortiGate hardware appliances running FortiOS 7.2.0+.

---

## Manual License

Example:

```bash
execute restore manual-license ftp a.lic 192.168.20.200
```

> The exact transport syntax depends on the file-transfer method available in the release.

---

# 19. IoT Detection Service

FortiGate IoT Detection can use FortiGuard intelligence to identify devices that cannot be fully identified using the local device database.

Architecture:

```text
Unknown Device
      ↓
FortiGate Device Detection
      ↓
Local CIDB
      |
      ├── Known → Identify
      |
      └── Unknown / incomplete
               ↓
          FortiGuard
               ↓
        Device intelligence
               ↓
            FortiGate
```

---

## Requirements

Typically:

* IoT Detection subscription
* FortiCare registration
* FortiGuard connectivity
* Device Detection enabled

---

## Force FortiGuard Query

```bash
diagnose cid sigs disable
```

This disables the local device signature database for testing so queries can be forced toward FortiGuard.

---

## IoT Debug

```bash
diagnose debug application iotd -1
diagnose debug enable
```

Device list:

```bash
diagnose user device list
```

---

# FortiAP + FortiGuard IoT

FortiAP can collect device information and FortiGate can query FortiGuard for additional identification intelligence.

Potential data sources include:

```text
DHCP
MAC
HTTP
Device behavior
```

---

## Device Weight

Example:

```bash
config wireless-controller setting
    edit ap-1
        set device-weight 1
        set device-holdoff 5
        set device-idle 1440
    next
end
```

Conceptually:

```text
FortiGuard intelligence
        +
DHCP information
        +
Behavioral analysis
        ↓
Device confidence
```

---

# 20. Certificates & PKI

FortiOS uses certificates for:

* HTTPS administration
* SSL VPN
* IPsec-related authentication
* Deep Inspection
* SAML
* Fabric communication
* Load balancing
* TLS services
* Device identity

---

# Certificate Types

## Local Certificate

Contains:

```text
Certificate
+
Private Key
```

Used when FortiGate must identify itself.

Examples:

```text
HTTPS admin
SSL VPN
Virtual Server
TLS service
```

---

## Remote Certificate

Normally contains the **public certificate/key information** required to identify a remote entity.

The private key is not required.

Example:

```text
SAML
Remote identity
Trust relationships
```

---

## CA Certificate

Used to establish trust.

```text
Root CA
   ↓
Intermediate CA
   ↓
Server Certificate
```

---

# CSR

A **Certificate Signing Request** is generated when a certificate should be signed by a CA.

Concept:

```text
FortiGate
   |
   | Generate private/public key
   ↓
CSR
   |
   ↓
Certificate Authority
   |
   ↓
Signed Certificate
   |
   ↓
Import into FortiGate
```

---

# Certificate Generation

Examples:

```bash
execute vpn certificate local generate rsa \
    <certificate_name> \
    <key_size> \
    <subject> \
    <country> \
    <state> \
    <city> \
    <organization> \
    <ou> \
    <email>
```

EC example:

```bash
execute vpn certificate local generate ec \
    <certificate_name> \
    <curve_name> \
    <subject> \
    <country> \
    <state> \
    <city> \
    <organization> \
    <ou> \
    <email>
```

---

# Default SSL Certificates

Examples:

```bash
execute vpn certificate local generate default-ssl-ca

execute vpn certificate local generate default-ssl-key-certs

execute vpn certificate local generate default-ssl-serv-key
```

---

# Import Certificate

Example:

```bash
execute vpn certificate local import tftp \
    <file_name> \
    <server_address> \
    <cert_type>
```

Possible certificate categories include:

```text
CA
Local
Remote
CRL
```

---

## VDOM Certificate Scope

Important:

```text
Certificate uploaded to VDOM
        ↓
Accessible to that VDOM

Certificate uploaded globally
        ↓
Potentially available across VDOMs
```

---

# PKCS#12

PKCS#12 commonly bundles:

```text
Certificate
+
Private Key
+
Optional certificate chain
```

Typical extensions:

```text
.p12
.pfx
```

A password is normally required to protect the private key material.

---

# ACME / Let's Encrypt

ACME automates certificate enrollment.

```text
FortiGate
   ↓
ACME
   ↓
Let's Encrypt / ACME CA
   ↓
Certificate
   ↓
HTTPS / SSL VPN / other service
```

---

## ACME Requirements

Typical requirements include:

* Publicly reachable service
* Public IP
* DNS hostname resolving to that IP
* Appropriate ACME interface
* Port 80/443 availability according to challenge method and FortiOS implementation

FortiOS ACME support has specific interface and hostname requirements; verify them for the exact release before deployment.

---

## Test Connectivity

```bash
execute ping acme-v02.api.letsencrypt.org
```

---

## ACME Configuration

```bash
config vpn certificate local
    edit acme-test
        set enroll-protocol acme2
        set acme-domain test.example.com
        set acme-email admin@example.com
    next
end
```

Check:

```bash
get vpn certificate local details acme-test
```

Debug:

```bash
diagnose sys acme status-full test.example.com
```

---

# Use ACME Certificate for Admin GUI

```bash
config system global
    set admin-server-cert acme-test
end
```

---

# 21. Certificate-Based SSH Authentication

A remote certificate can be associated with an administrator.

Concept:

```text
SSH Client
   |
Private Key
   ↓
FortiGate
   |
Certificate authentication
   ↓
Admin account
```

Example:

```bash
config certificate remote
    edit REMOTE_Cert_1
    next
end
```

Then:

```bash
config system admin
    edit admin1
        set accprofile prof_admin
        set vdom root
        set ssh-certificate REMOTE_Cert_1
    next
end
```

---

# 22. REST API

FortiGate provides a REST API for automation.

Typical workflow:

```text
Create REST API administrator
        ↓
Generate API token
        ↓
API Client
        ↓
HTTPS
        ↓
FortiGate REST API
```

---

## Example API Endpoint

```text
/api/v2/cmdb/firewall/policy/1
```

Example:

```text
GET
https://<FORTIGATE>/api/v2/cmdb/firewall/policy/1
```

---

## Postman

Use:

```text
Authorization
    ↓
Bearer Token
    ↓
API Token
```

Expected successful HTTP response:

```text
200 OK
```

---

## Security Best Practices

Never:

```text
Hard-code API tokens
Share tokens
Use unrestricted admin profiles
Expose API management publicly
```

Prefer:

```text
Least privilege
Trusted source IPs
Dedicated API administrators
Token rotation
HTTPS
Audit logging
```

---

# 23. HTTPS Daemon Troubleshooting

FortiGate HTTPS administration is handled by the HTTPS daemon.

Useful debug commands:

```bash
diagnose debug reset
diagnose debug enable
diagnose debug application httpsd -1
```

Stop:

```bash
diagnose debug disable
```

---

# FortiDDNS

FortiDDNS functionality may depend on Fortinet licensing/service availability and FortiOS/platform version.

> Verify current FortiDDNS entitlement before deploying it as a production dependency.

---

# Elliptic Curves

Common curve identifiers may include:

```text
19 → secp256r1
20 → secp384r1
21 → secp521r1
```

> The third value is commonly associated with **secp521r1**, not secp512r1.

---

# 24. Workspace Mode

## What Is Workspace Mode?

Workspace Mode allows administrators to make a batch of configuration changes that are **not applied to the live configuration until commit**.

Fortinet documents that edited objects are locked during a transaction, preventing conflicting administrators from modifying the same objects.

---

## Normal Mode

```text
Admin changes config
       ↓
Configuration applied
```

---

## Workspace Mode

```text
Admin
  ↓
Start transaction
  ↓
Make changes
  ↓
Review
  ↓
Commit
  ↓
Live configuration
```

Or:

```text
Abort
  ↓
Discard changes
```

---

# Start Transaction

```bash
execute config-transaction
```

---

# Commit

```bash
execute config-transaction commit
```

---

# Abort

```bash
execute config-transaction abort
```

---

# Transaction Timeout

FortiOS workspace transactions have a timeout when inactive.

Fortinet's 7.2 documentation describes a **five-minute inactivity timeout** for workspace transactions, after which changes are discarded.

Typical warnings:

```text
config transaction id=1 will expire in 30 seconds
config transaction id=1 will expire in 20 seconds
config transaction id=1 will expire in 10 seconds
config transaction id=1 has expired
```

---

# Workspace Mode Locking

If another administrator is modifying an object:

```text
Can not config the object since either the object
or the referenced objects are being configured
by other transactions.

Command fail.
Return code 14
```

This prevents conflicting configuration changes.

---

# Commands Not Available in Workspace Transactions

Important examples include:

```text
config system console
config system resource-limits
config system elbc
config system global
set split-port
set vdom-admin
set management-vdom
set wireless-mode
set internal-switch-mode
```

Also several system-wide configuration areas are excluded.

Fortinet's 7.2 documentation lists the unsupported transaction areas explicitly.

---

# Workspace Diagnostics

## Transaction Metadata

```bash
diagnose sys config-transaction show txn-meta
```

Shows transaction metadata.

---

## Transaction Information

```bash
diagnose sys config-transaction show txn-info
```

Shows active transaction information.

---

## Transaction Entities

```bash
diagnose sys config-transaction show txn-entity
```

Shows affected configuration entities.

---

## Transaction Locks

```bash
diagnose sys config-transaction show txn-lock
```

Shows locked objects.

---

## CLI Commands

```bash
diagnose sys config-transaction show txn-cli-commands
```

Useful to see commands associated with transactions where supported.

---

## Memory / Transaction Context

```bash
diagnose sys config-transaction show mctx
```

Useful for inspecting transaction memory context.

> Do not interpret a single `free_pages=0` observation without understanding the exact FortiOS release/output semantics.

---

# 25. High-Value Troubleshooting Commands

## System

```bash
get system status
get system performance status
diagnose sys top
```

---

## Time

```bash
execute time
execute date
diagnose sys ntp status
```

---

## Sessions

```bash
diagnose sys session list
```

Look for:

```text
state
npu_state
npu info
offload
in_npu
out_npu
```

---

## FortiGuard

```bash
diagnose sys service-communication
diagnose debug rating
get webfilter status
```

---

## IoT

```bash
diagnose user device list
diagnose debug application iotd -1
```

---

## HTTPS

```bash
diagnose debug reset
diagnose debug enable
diagnose debug application httpsd -1
diagnose debug disable
```

---

## PTP

```bash
diagnose debug application ptpd -1
```

---

## Autoupdate

```bash
diagnose debug application updated -1
diagnose debug enable
```

---

# 26. NSE Exam Quick Review

## 🧠 Must Remember

| Topic              | Key Point                                |
| ------------------ | ---------------------------------------- |
| Default Admin      | Change immediately                       |
| NTP                | Accurate time is critical                |
| PTP                | High-precision time synchronization      |
| TLS                | Prefer TLS 1.2+                          |
| Admin Lockout      | Protect management plane                 |
| Auxiliary Session  | Handles changing traffic paths           |
| ECMP               | Can cause different ingress/egress paths |
| NPU                | Accelerates eligible data-plane traffic  |
| CPU                | Initial/session/control processing       |
| TPM                | Protects sensitive cryptographic data    |
| TPM                | Does **not** encrypt disk                |
| SNMPv3             | Authentication + privacy                 |
| MIB View           | Limits OID visibility                    |
| FortiGuard         | Security intelligence + updates          |
| Anycast            | Optimized FortiGuard connectivity        |
| FortiManager       | Can act as local FortiGuard server       |
| FDS-only ISDB      | Lightweight built-in ISDB                |
| Air Gap            | Manual licensing/update workflow         |
| Local Certificate  | FortiGate identity + private key         |
| Remote Certificate | Remote identity/public certificate       |
| CA Certificate     | Establishes trust                        |
| CSR                | Request signed certificate               |
| ACME               | Automated certificate enrollment         |
| REST API           | Automation via HTTPS                     |
| Workspace Mode     | Changes require commit                   |
| Transaction Abort  | Discards uncommitted changes             |

---

# 🔥 NSE Trap Questions

## Trap 1 — Auxiliary Session

> **"Auxiliary Session is the same thing as asymmetric routing."**

❌ False.

They are related to traffic-path behavior but are separate FortiOS mechanisms/settings.

---

## Trap 2 — Auxiliary Session Default

For FortiOS 6.2.3+:

```text
Disabled by default
```

unless platform/release-specific behavior differs.

---

## Trap 3 — TPM

> "TPM encrypts the FortiGate disk."

❌ False.

TPM protects cryptographic material/configuration secrets; Fortinet explicitly states that the TPM does not encrypt the disk drive.

---

## Trap 4 — Workspace Mode

> "Configuration changes immediately affect the live system."

❌ Not before commit.

```text
Start transaction
      ↓
Modify
      ↓
Commit
      ↓
Live configuration
```

---

## Trap 5 — Workspace Timeout

Inactive workspace transactions expire and their changes are discarded. Fortinet's 7.2 documentation specifies a five-minute inactivity timeout.

---

## Trap 6 — MIB View

MIB View does **not** create a new MIB.

It controls which portions of the MIB tree are accessible.

```text
Include
   +
Exclude
   ↓
Accessible OID tree
```

---

## Trap 7 — Local vs Remote Certificate

```text
Local Certificate
    ↓
FortiGate identity
    ↓
Private key required

Remote Certificate
    ↓
Remote entity identity
    ↓
Private key normally not required
```

---

## Trap 8 — FortiGuard Anycast

Anycast does not mean:

```text
One physical FortiGuard server
```

It means:

```text
Same logical destination
       ↓
Multiple geographically distributed servers
       ↓
Routing chooses an appropriate endpoint
```

Fortinet documents Anycast as the default FortiGuard access mechanism in the relevant FortiOS 7.2 documentation.

---

# 🧩 Production Hardening Checklist

```text
[ ] Change default admin password
[ ] Restrict administrative interfaces
[ ] Disable unnecessary HTTP/Telnet access
[ ] Use HTTPS/SSH
[ ] Use strong TLS settings
[ ] Configure admin lockout
[ ] Configure accurate timezone
[ ] Configure reliable NTP
[ ] Monitor NTP synchronization
[ ] Protect configuration backups
[ ] Consider private-data encryption
[ ] Validate TPM support where applicable
[ ] Configure SNMPv3 instead of v1/v2c where possible
[ ] Restrict SNMP source addresses
[ ] Restrict SNMP MIB views
[ ] Validate FortiGuard connectivity
[ ] Verify license status
[ ] Validate FortiGuard update path
[ ] Use FortiManager locally where architecture requires it
[ ] Protect API administrators
[ ] Restrict REST API source addresses
[ ] Use trusted certificates
[ ] Replace default admin certificate where appropriate
[ ] Use ACME for automated certificate lifecycle where appropriate
[ ] Use Workspace Mode for controlled configuration changes
[ ] Verify configuration backups
[ ] Monitor system events
```

---

# 🎯 One-Page Mental Model

```text
                         FORTIGATE SYSTEM
                                |
        ┌───────────────────────┼────────────────────────┐
        │                       │                        │
      TIME                   SECURITY                 MANAGEMENT
        │                       │                        │
   ┌────┴────┐            ┌─────┴─────┐          ┌──────┴──────┐
   │   NTP   │            │    TLS    │          │ HTTPS / SSH │
   │   PTP   │            │    TPM    │          │    REST API │
   └─────────┘            │   SNMP    │          │  Workspace  │
                          └───────────┘          └─────────────┘

                                |
                        ┌───────┴────────┐
                        │  DATA PLANE    │
                        └───────┬────────┘
                                │
                         ┌──────┴──────┐
                         │             │
                        CPU           NPU
                         │             │
                  Session / Policy    Fast
                  Routing / Control   Forwarding
                         │             │
                         └──────┬──────┘
                                │
                       Auxiliary Sessions
                                │
                       ECMP / SD-WAN /
                     Asymmetric Paths
                                │
                                ▼
                         Traffic Continuity
```

---

# 🔗 Official Fortinet References

* [FortiOS 7.2 Hardening Guide](https://docs.fortinet.com/document/fortigate/7.2.0/best-practices/555436/hardening)
* [Auxiliary Sessions — FortiOS 7.2](https://docs.fortinet.com/document/fortigate/7.2.7/administration-guide/14295/controlling-return-path-with-auxiliary-session)
* [Workspace Mode — FortiOS 7.2](https://docs.fortinet.com/document/fortigate/7.2.8/administration-guide/530847/workspace-mode)
* [TPM Support — FortiOS 7.2](https://docs.fortinet.com/document/fortigate/7.2.12/administration-guide/893277/trusted-platform-module-support)
* [FortiGuard — FortiOS 7.2](https://docs.fortinet.com/document/fortigate/7.2.4/administration-guide/42459/fortiguard)
* [FortiManager as Local FortiGuard Server](https://docs.fortinet.com/document/fortigate/7.2.11/administration-guide/179018/using-fortimanager-as-a-local-fortiguard-server)

---

## 🏷️ Suggested GitHub Tags

`fortigate` `fortios` `fortinet` `nse4` `nse7` `network-security` `firewall` `cybersecurity` `fortiguard` `snmp` `ntp` `ptp` `auxiliary-session` `npu` `tpm` `workspace-mode` `ssl` `tls` `certificate` `acme` `rest-api` `sdwan` `ecmp`

---

## #️⃣ Suggested SEO Keywords

**Primary:**

```text
FortiGate system settings
FortiOS 7.2  
FortiGate NSE4  
FortiGate NSE7 
FortiGate configuration guide
```

**Technical:**

```text
FortiGate auxiliary session
FortiGate NPU
FortiGate SNMP
FortiGate TPM
FortiGate FortiGuard
FortiGate Anycast
FortiGate Workspace Mode
FortiGate ACME certificate
FortiGate REST API
FortiGate asymmetric routing
FortiGate ECMP
```

**Long-tail:**

```text
how FortiGate auxiliary session works
FortiOS workspace mode explained
FortiGate FortiGuard troubleshooting
FortiGate NPU session offloading
FortiGate SNMPv3 configuration
FortiGate TPM private data encryption
FortiGate ACME Let's Encrypt configuration
FortiGate FortiManager local FortiGuard
```

---

> **SheynShield | Engineering Secure Networks**
> Practical Network Security • Fortinet • Network Design • Security Engineering

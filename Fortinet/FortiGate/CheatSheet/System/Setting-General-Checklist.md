# 🔐 FortiGate System Hardening Checklist

> **FortiOS 7.2.x | NSE4 / NSE7 | Production Hardening | Troubleshooting**
>
> **SheynShield | Engineering Secure Networks**
>
> Practical checklist for **Administrator Security, NTP/PTP, Management Ports, Session Processing, Auxiliary Sessions, NPU Offloading, Link Monitor, Configuration Save Modes, TPM, Private Data Encryption & Workspace Transactions**.

---

## 📌 Quick Navigation

* [ ] [Administrator Security](#1-administrator-security)
* [ ] [System Time — NTP & PTP](#2-system-time--ntp--ptp)
* [ ] [Administrative Ports](#3-administrative-ports)
* [ ] [Administrator Session Timeout & Lockout](#4-administrator-session-timeout--lockout)
* [ ] [Auxiliary Sessions](#5-auxiliary-sessions)
* [ ] [Auxiliary Session vs NPU](#6-auxiliary-session-vs-npu-processing)
* [ ] [Asymmetric Routing & Multi-ISP](#7-asymmetric-routing--multi-isp)
* [ ] [Session & NPU Troubleshooting](#8-session--npu-troubleshooting)
* [ ] [Link Monitor](#9-link-monitor)
* [ ] [Configuration Save Modes](#10-configuration-save-modes)
* [ ] [TPM & Private Data Encryption](#11-tpm--private-data-encryption)
* [ ] [Replacement Messages](#12-replacement-messages)
* [ ] [CLI Configuration Scripts](#13-cli-configuration-scripts)
* [ ] [Workspace / Configuration Transactions](#14-workspace--configuration-transactions)
* [ ] [NSE Exam Checklist](#15-nse-exam-checklist)
* [ ] [Troubleshooting Command Map](#16-troubleshooting-command-map)
* [ ] [Production Hardening Checklist](#17-production-hardening-checklist)

---

# 1. Administrator Security

## 🔑 Initial Administrator Hardening

* [ ] Change the factory/default administrator credential
* [ ] Create named administrator accounts
* [ ] Avoid using the shared `admin` account for daily administration
* [ ] Restrict management access to trusted interfaces
* [ ] Use HTTPS instead of HTTP
* [ ] Use SSH instead of Telnet
* [ ] Enable MFA where supported
* [ ] Review administrator profiles and permissions
* [ ] Apply least privilege
* [ ] Review administrator authentication logs

### Password Policy

```bash
config system password-policy
    set status enable
    set minimum-length 12
    set min-upper-case-letter 1
    set min-lower-case-letter 1
    set min-number 1
    set min-non-alphanumeric 1
    set reuse-password disable
end
```

### Verify

* [ ] Password policy enabled
* [ ] Minimum password length defined
* [ ] Uppercase requirement configured
* [ ] Lowercase requirement configured
* [ ] Numeric requirement configured
* [ ] Special-character requirement configured
* [ ] Password reuse restricted
* [ ] `min-change-characters` understood when configured

### 🧠 NSE Memory

```text
reuse-password enable
        ↓
Password reuse allowed

reuse-password disable
        ↓
Password must change

min-change-characters > 0
        ↓
Overrides reuse-password
```

---

# 2. System Time — NTP & PTP

## ⏱️ Time Hardening

* [ ] Configure the correct timezone
* [ ] Verify daylight-saving configuration
* [ ] Configure reliable NTP
* [ ] Verify synchronization
* [ ] Use trusted/internal NTP infrastructure where appropriate
* [ ] Configure NTP authentication when required
* [ ] Avoid unnecessary Internet-exposed NTP services
* [ ] Verify time-dependent security features
* [ ] Verify logging timestamps
* [ ] Verify SIEM correlation timestamps

### Manual Time

```bash
execute date
execute time
```

Manual configuration:

```bash
execute date 2026-08-29
execute time 12:30:00
```

### Timezone Example

```bash
config system global
    set timezone 41
end
```

For Tehran:

```text
timezone 41 = GMT+03:30
```

### DST

```bash
config system global
    set timezone 41
    set dst disable
end
```

* [ ] Confirm actual jurisdictional DST requirements
* [ ] Do not blindly copy old `dst enable` configurations

---

## 🕐 NTP Configuration

```bash
config system ntp
    set ntpsync enable
    set type custom
    set syncinterval 60

    config ntpserver
        edit 1
            set server ntp.example.com
            set ntpv3 disable
        next
    end
end
```

### Verify

```bash
diagnose sys ntp status
```

### NTP Version

* [ ] Understand `ntpv3 enable`
* [ ] Understand `ntpv3 disable`

```text
ntpv3 enable
    =
NTPv3

ntpv3 disable
    =
NTPv4
```

### NTP Authentication

* [ ] Understand NTP authentication
* [ ] Understand MD5/SHA1 options
* [ ] Validate compatibility with the NTP infrastructure

```text
NTPv3 → MD5
NTPv4 → SHA1
```

> ⚠️ Always verify the exact supported behavior against the target FortiOS release.

---

## 🔄 NTP Sync Interval

```bash
config system ntp
    set syncinterval 1
end
```

* [ ] Remember the unit is **minutes**
* [ ] Verify the configured interval
* [ ] Avoid unnecessarily aggressive synchronization

---

## 📡 FortiGate as NTP Server

```bash
config system ntp
    set server-mode enable
    set interface "LAN"
end
```

* [ ] Enable only where required
* [ ] Restrict NTP service to intended interfaces
* [ ] Verify downstream clients
* [ ] Verify upstream synchronization

---

# 2.1 PTP — Precision Time Protocol

* [ ] Determine whether NTP precision is sufficient
* [ ] Use PTP for environments requiring tighter synchronization
* [ ] Identify PTP master/slave architecture
* [ ] Select appropriate delay mechanism
* [ ] Verify interface configuration

Example:

```bash
config system ptp
    set status enable
    set interface "port1"
    set delay-mechanism E2E
end
```

### PTP Delay Models

```text
E2E = End-to-End path delay

P2P = Peer-to-Peer link delay
```

### Exam Check

* [ ] E2E understood
* [ ] P2P understood
* [ ] Difference between NTP and PTP understood

---

# 2.2 Time Troubleshooting

### Basic Verification

```bash
execute date
execute time
diagnose sys ntp status
```

### PTP Debug

```bash
diagnose debug application ptpd -1
```

### Important Troubleshooting Rule

* [ ] Do not treat TCP telnet to UDP/123 as a definitive NTP test
* [ ] Use FortiGate NTP diagnostics
* [ ] Use packet capture when required
* [ ] Verify routing
* [ ] Verify firewall policy
* [ ] Verify DNS if hostname is used
* [ ] Verify upstream NTP availability

---

# 3. Administrative Ports

## 🔐 Management Port Hardening

Recommended:

```text
HTTPS
SSH
```

Avoid unless specifically required:

```text
HTTP
TELNET
```

Example:

```bash
config system global
    set admin-port 80
    set admin-sport 443
    set admin-https-redirect enable
    set admin-ssh-port 2142
    set admin-telnet-port 2323
end
```

### Checklist

* [ ] HTTPS enabled
* [ ] HTTP exposure reviewed
* [ ] HTTPS redirect enabled where appropriate
* [ ] SSH port reviewed
* [ ] Telnet disabled/unexposed
* [ ] Management services restricted by interface
* [ ] Management services restricted by trusted source IP where possible
* [ ] Unused management protocols disabled

### HTTPS Redirect

```bash
set admin-https-redirect enable
```

```text
HTTP :80
   ↓
Redirect
   ↓
HTTPS :443
```

---

## 🔌 Default Service Source Port

```bash
config system global
    set default-service-source-port 20-30
end
```

* [ ] Understand this controls source-port behavior for system-generated service traffic
* [ ] Do not confuse it with firewall-policy destination ports

---

# 4. Administrator Session Timeout & Lockout

## ⏳ Administrator Idle Timeout

```bash
config system global
    set admintimeout 10
end
```

Recommended operational baseline:

```text
5–10 minutes
```

* [ ] Configure idle timeout
* [ ] Avoid unnecessarily long administrative sessions
* [ ] Balance security and operational requirements

---

## 🚫 Login Lockout

```bash
config system global
    set admin-lockout-threshold 2
    set admin-lockout-duration 60
end
```

Concept:

```text
Failed attempts
      ↓
Threshold reached
      ↓
Account locked
      ↓
Lockout duration
```

* [ ] Configure failed-login threshold
* [ ] Configure lockout duration
* [ ] Monitor failed authentication events
* [ ] Verify the setting matches operational requirements

---

## 📋 Authentication Logs

Review:

```text
Log & Report
    ↓
System Events
    ↓
General System Events
```

Check for:

* [ ] Failed logins
* [ ] Successful logins
* [ ] Lockouts
* [ ] Administrative activity
* [ ] Authentication anomalies
* [ ] Unexpected administrator source addresses

---

# 5. Auxiliary Sessions

## 🧠 What Is Auxiliary Session?

Auxiliary sessions provide additional session handling when traffic associated with an existing connection changes its incoming/outgoing interface or path.

Relevant environments:

* [ ] ECMP
* [ ] Multiple WAN links
* [ ] Load balancing
* [ ] Policy-Based Routing
* [ ] SD-WAN
* [ ] ADVPN
* [ ] Asymmetric routing
* [ ] Dynamic path changes

---

## 📌 Version History

| FortiOS         | Auxiliary Session   |
| --------------- | ------------------- |
| 6.0 and earlier | Not supported       |
| 6.2.0–6.2.2     | Permanently enabled |
| 6.2.3+          | Disabled by default |

### FortiOS 7.2.x

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

* [ ] Verify whether auxiliary sessions are actually required
* [ ] Do not enable blindly
* [ ] Evaluate traffic symmetry
* [ ] Evaluate SD-WAN/ADVPN behavior
* [ ] Validate CPU/NPU behavior after changes

---

# 5.1 Auxiliary Session ≠ Asymmetric Routing

### Asymmetric Routing

```bash
config system settings
    set asymroute enable
end
```

IPv6:

```bash
set asymroute6 enable
```

ICMP:

```bash
set asymroute-icmp enable
set asymroute6-icmp enable
```

### Remember

```text
asymroute
    ↓
Controls asymmetric-routing behavior

auxiliary-session
    ↓
Controls additional session handling
```

* [ ] Do not treat them as identical settings
* [ ] Understand their relationship
* [ ] Evaluate both independently

---

# 6. Auxiliary Session vs NPU Processing

## 🧠 Simplified Processing Model

```text
Incoming Packet
       ↓
      CPU
       ↓
Session Lookup
       ↓
New / Existing Session
       ↓
Policy + Routing
       ↓
NPU Eligibility
       ↓
      NPU
       ↓
Fast Data-Plane Forwarding
```

### CPU

* [ ] Session establishment
* [ ] Initial packet processing
* [ ] Policy processing
* [ ] Routing decisions
* [ ] Session-state management
* [ ] Control-plane functions

### NPU

* [ ] Understand hardware-dependent NPU support
* [ ] Understand eligible data-plane offloading
* [ ] Verify offload status
* [ ] Understand that CPU is not necessarily absent from the session

### Core Concept

```text
CPU
 ↓
Session establishment/control

NPU
 ↓
Eligible data-plane forwarding
```

---

## 🧹 Dirty Sessions

A session may become:

```text
DIRTY
```

Potential triggers:

* [ ] Incoming interface changed
* [ ] Outgoing interface changed
* [ ] Routing changed
* [ ] ECMP path changed
* [ ] Asymmetric traffic
* [ ] Session state requires update

Concept:

```text
Path Change
    ↓
Session Update Required
    ↓
DIRTY
    ↓
Additional Processing
```

---

# 6.1 NPU Offload Verification

```bash
diagnose sys session list
```

Look for:

```text
npu info:
offload=8/8
```

or:

```text
offload=8/0
```

Conceptually:

```text
8/8 → both directions offloaded
8/0 → one direction offloaded
```

> ⚠️ Exact flags and interpretation can vary by FortiOS release and hardware platform.

---

# 7. Asymmetric Routing & Multi-ISP

## 🌐 Design Checklist

* [ ] Identify all WAN interfaces
* [ ] Identify primary route
* [ ] Identify backup route
* [ ] Identify ECMP
* [ ] Identify PBR
* [ ] Identify SD-WAN
* [ ] Identify ADVPN
* [ ] Identify expected return path
* [ ] Determine whether routing is symmetric
* [ ] Determine whether return traffic can arrive through another interface
* [ ] Evaluate auxiliary sessions
* [ ] Evaluate `asymroute`
* [ ] Verify NPU offloading
* [ ] Monitor dirty sessions

---

## Example

```text
                 ISP-1
                   |
                 port1
                   |
             +-----------+
             | FortiGate |
             +-----------+
                   |
                 port3
                   |
                 LAN
                   
                 ISP-2
                   |
                 port2
```

Primary route:

```text
0.0.0.0/0
distance 10
```

Backup:

```text
0.0.0.0/0
distance 250
```

---

## 🔄 Path-Change Analysis

### Symmetric

```text
Forward:
ISP-1 → FortiGate → LAN

Return:
LAN → FortiGate → ISP-1
```

* [ ] Auxiliary session requirement evaluated
* [ ] NPU offload verified

### Asymmetric

```text
Forward:
ISP-1 → FortiGate → LAN

Return:
LAN → FortiGate → ISP-2
```

* [ ] `asymroute` evaluated
* [ ] `auxiliary-session` evaluated
* [ ] Session table inspected
* [ ] NPU behavior inspected
* [ ] CPU utilization monitored

---

## ⚠️ SD-WAN / ADVPN Warning

Do **not** assume:

```text
SD-WAN = auxiliary-session enable
```

Instead:

* [ ] Determine whether return traffic is expected to be symmetric
* [ ] Understand hub/spoke behavior
* [ ] Understand ADVPN path selection
* [ ] Test failover
* [ ] Test return traffic
* [ ] Verify session behavior

---

# 8. Session & NPU Troubleshooting

## 🔍 Session Table

```bash
diagnose sys session list
```

Inspect:

* [ ] `state`
* [ ] `dev`
* [ ] `npu_state`
* [ ] `npu info`
* [ ] `offload`
* [ ] VLAN information
* [ ] Queue information
* [ ] Interface information

---

## 🔎 Dirty Session Investigation

Search for:

```text
state=dirty
```

Investigate:

* [ ] Path changes
* [ ] Interface changes
* [ ] Routing changes
* [ ] ECMP behavior
* [ ] Asymmetric routing
* [ ] SD-WAN decisions
* [ ] CPU impact

---

## 🚀 NPU Investigation

Check:

```text
npu info:
offload=8/8
```

or:

```text
offload=8/0
```

* [ ] Determine whether forward direction is offloaded
* [ ] Determine whether reverse direction is offloaded
* [ ] Compare behavior before/after routing changes
* [ ] Check platform-specific NPU capabilities

---

## 🔁 Auxiliary / Reflection Information

Example:

```text
reflect info 0:
dev=37->38/38->37
npu_state=0x000400
npu info:
flag=0x91/0x00
offload=8/0
```

* [ ] Correlate session with interface
* [ ] Correlate session with NPU state
* [ ] Correlate offload status
* [ ] Check auxiliary/reflect information

---

# 9. Link Monitor

## 🔗 Link Monitor Validation

For multi-WAN designs:

* [ ] Configure health-check destination
* [ ] Select correct source interface
* [ ] Select appropriate probe protocol
* [ ] Configure gateway
* [ ] Configure interval
* [ ] Configure timeout
* [ ] Configure failure threshold
* [ ] Configure recovery threshold
* [ ] Configure route update behavior
* [ ] Configure policy-route update behavior
* [ ] Configure cascade-interface behavior
* [ ] Verify HA interaction where applicable

Example:

```bash
config system link-monitor
    edit "WAN1"
        set addr-mode ipv4
        set srcintf "port1"
        set server-type static
        set server 192.168.254.2
        set protocol ping
        set gateway-ip 192.168.254.2
        set source-ip 0.0.0.0
        set interval 500
        set probe-timeout 500
        set failtime 5
        set recoverytime 5
        set probe-count 30
        set ha-priority 1
        set update-cascade-interface enable
        set update-static-route enable
        set update-policy-route enable
        set status enable
    next
end
```

### Key Parameters

| Parameter                  | Purpose                          |
| -------------------------- | -------------------------------- |
| `srcintf`                  | Probe source interface           |
| `server`                   | Health-check destination         |
| `protocol`                 | Probe protocol                   |
| `gateway-ip`               | Gateway associated with monitor  |
| `interval`                 | Probe interval                   |
| `probe-timeout`            | Probe timeout                    |
| `failtime`                 | Failure threshold                |
| `recoverytime`             | Recovery threshold               |
| `probe-count`              | Probe count                      |
| `update-static-route`      | Static-route update              |
| `update-policy-route`      | Policy-route update              |
| `update-cascade-interface` | Cascades interface state changes |

---

# 10. Configuration Save Modes

## 💾 Save Behavior

FortiOS provides:

```text
automatic
manual
revert
```

---

## Automatic

```bash
config system global
    set cfg-save automatic
end
```

Concept:

```text
Configuration Change
        ↓
RAM
        ↓
Configuration Saved
```

* [ ] Understand automatic save behavior

---

## Manual

```bash
config system global
    set cfg-save manual
end
```

Save explicitly:

```bash
execute cfg save
```

* [ ] Remember manual mode requires explicit save
* [ ] Verify changes before saving

---

## Revert

```bash
config system global
    set cfg-save revert
    set cfg-revert-timeout 600
end
```

Concept:

```text
Change
  ↓
Temporary Configuration
  ↓
   ┌───────────────┐
   │               │
Commit          Timeout
   │               │
   ↓               ↓
Keep            Revert
```

* [ ] Use revert mode for risky remote changes
* [ ] Configure an appropriate timeout
* [ ] Confirm access before committing

---

## 🚨 Save vs Reload

```bash
execute cfg save
```

= Save configuration

```bash
execute cfg reload
```

= Reload/reboot behavior

### NSE Trap

* [ ] Do not confuse `save` with `reload`

---

# 11. TPM & Private Data Encryption

## 🔐 TPM Checklist

* [ ] Verify TPM hardware support
* [ ] Understand TPM is hardware-backed security
* [ ] Understand TPM is not full-disk encryption
* [ ] Understand the master encryption key/password
* [ ] Document recovery requirements
* [ ] Test backup/restore
* [ ] Verify HA requirements

---

## Private Data Encryption

```bash
config system global
    set private-data-encryption enable
end
```

* [ ] Enable only after recovery planning
* [ ] Protect the master encryption key
* [ ] Document key ownership
* [ ] Verify backup procedures
* [ ] Verify restore procedures

---

## 🔑 Encryption Concept

```text
Master Encryption Password
          ↓
      AES-128-CBC
          ↓
Sensitive Data

          +

         TPM
          ↓
      RSA-2048
          ↓
Protects Master Encryption Password
```

### Core Memory

```text
TPM
 ↓
Hardware-backed key protection
 ↓
Private Data Encryption
 ↓
Sensitive credentials / cryptographic material
```

---

## ❌ TPM ≠ Full Disk Encryption

* [ ] Understand TPM does not mean the entire FortiGate disk is encrypted
* [ ] Understand TPM protects cryptographic material according to FortiOS private-data-encryption mechanisms

```text
TPM
≠
Full Disk Encryption
```

---

## 💾 Backup / Restore

Before enabling private-data encryption:

* [ ] Create configuration backup
* [ ] Protect encryption credentials
* [ ] Document recovery process
* [ ] Verify TPM state
* [ ] Test restore
* [ ] Test failure/recovery scenario
* [ ] Verify HA compatibility

### Recovery Dependency

```text
TPM State
    +
Master Encryption Password
    +
Compatible Device/Configuration
    ↓
Protected Data Recovery
```

---

## 🏢 HA + TPM

* [ ] Verify TPM support on every HA member
* [ ] Verify FortiOS compatibility
* [ ] Use the required matching master encryption configuration
* [ ] Verify synchronization
* [ ] Test failover
* [ ] Test restore
* [ ] Document recovery procedure

---

## 🛠️ TPM Diagnostics

```bash
diagnose hardware test info
```

```bash
diagnose hardware deviceinfo tpm
```

```bash
diagnose tpm
```

---

## 🔐 Sensitive Data Categories

Potential protected data includes:

* [ ] Service credentials
* [ ] Authentication credentials
* [ ] IPsec PSKs
* [ ] Certificate private keys
* [ ] HA passwords
* [ ] LDAP credentials
* [ ] RADIUS credentials
* [ ] FSSO credentials
* [ ] NTP credentials
* [ ] SNMP credentials
* [ ] FortiGuard credentials
* [ ] FortiToken-related secrets
* [ ] PPPoE/modem credentials
* [ ] SDN connector credentials
* [ ] Link Monitor credentials

> ⚠️ Exact protected-data scope is FortiOS/version/feature dependent.

---

# 12. Replacement Messages

## 📨 Replacement Message Checklist

* [ ] Determine which security feature generates the message
* [ ] Identify replacement-message group
* [ ] Verify inspection mode
* [ ] Verify security profile dependency
* [ ] Verify policy association
* [ ] Test the resulting client-facing message

Enable GUI replacement-message groups where required:

```bash
config system settings
    set gui-replacement-message-groups enable
end
```

### Example

```bash
config emailfilter profile
    edit "newmsgs"
        set replacemsg-group "newutm"
    next
end
```

Firewall policy:

```bash
config firewall policy
    edit 1
        set replacemsg-override-group "newauth"
        set inspection-mode proxy
        set emailfilter-profile "newmsgs"
    next
end
```

### Dependency Chain

```text
Firewall Policy
      ↓
Proxy Inspection
      ↓
Security Profile
      ↓
Replacement Message
```

---

# 13. CLI Configuration Scripts

## 🧑‍💻 CLI Script Checklist

* [ ] Validate syntax before execution
* [ ] Understand configuration hierarchy
* [ ] Backup before major changes
* [ ] Use comments for operational clarity
* [ ] Test in lab where possible
* [ ] Review output/errors
* [ ] Verify resulting configuration

Comments begin with:

```bash
#
```

Example:

```bash
# Configure administrator timeout

config system global
    set admintimeout 10
end
```

---

# 14. Workspace / Configuration Transactions

## 🧩 Workspace Mode

Use Workspace/transaction mode when configuration changes require controlled deployment.

Concept:

```text
Start
  ↓
Modify
  ↓
Validate
  ↓
Commit
```

or:

```text
Start
  ↓
Modify
  ↓
Abort
  ↓
Discard
```

---

## Start Transaction

```bash
execute config-transaction start
```

With timeout:

```bash
execute config-transaction start 5
```

* [ ] Understand timeout is in minutes
* [ ] Understand default timeout
* [ ] Understand transaction expiration
* [ ] Avoid leaving transactions unnecessarily open

---

## Commit

```bash
execute config-transaction commit
```

* [ ] Validate changes
* [ ] Commit intentionally
* [ ] Verify resulting global configuration

---

## Abort

```bash
execute config-transaction abort
```

* [ ] Abort failed/unsafe transactions
* [ ] Verify changes were discarded

---

## ⏰ Transaction Expiration

```text
Transaction
     ↓
Timeout
     ↓
Expiration
     ↓
Uncommitted changes discarded
```

* [ ] Monitor transaction timeout
* [ ] Commit before expiration when changes are intended
* [ ] Understand automatic expiration behavior

---

## 🔒 Transaction Locks

Possible error:

```text
Can not config the object since either the object or the referenced objects
are being configured by other transactions.

Command fail.
Return code 14
```

Interpretation:

```text
Another Transaction
       ↓
Object / Reference Locked
       ↓
Configuration Blocked
```

* [ ] Identify active transaction
* [ ] Identify locked object
* [ ] Coordinate with other administrators

---

## 🛠️ Workspace Troubleshooting

### Transaction Metadata

```bash
diagnose sys config-transaction show txn-meta
```

### Transaction Information

```bash
diagnose sys config-transaction show txn-info
```

### Transaction Entities

```bash
diagnose sys config-transaction show txn-entity
```

### Transaction Locks

```bash
diagnose sys config-transaction show txn-lock
```

### Transaction CLI Commands

```bash
diagnose sys config-transaction show txn-cli-commands
```

### Memory Context

```bash
diagnose sys config-transaction show mctx
```

---

## 🚫 Workspace Restrictions

Before using Workspace Mode:

* [ ] Check whether the target configuration object is supported
* [ ] Review workspace restrictions
* [ ] Avoid assuming every global/system setting is transaction-compatible

Examples of restricted areas can include:

```text
config system console
config system resource-limits
config system elbc
config system global
config system npu
config system np6
config system wireless
config system vdom-property
config system storage
```

> ⚠️ Exact restrictions are FortiOS-version dependent. Verify the target release CLI reference.

---

# 15. NSE Exam Checklist

## 🔐 Administrator Security

* [ ] `admin-lockout-threshold` = failed login threshold
* [ ] `admin-lockout-duration` = lockout duration
* [ ] `admintimeout` = administrator idle timeout
* [ ] `reuse-password` = password reuse behavior
* [ ] `min-change-characters` can override reuse-password behavior

---

## ⏱️ Time

* [ ] NTP = network time synchronization
* [ ] PTP = precision time synchronization
* [ ] `ntpv3 enable` = NTPv3
* [ ] `ntpv3 disable` = NTPv4
* [ ] E2E = End-to-End
* [ ] P2P = Peer-to-Peer

---

## 🔄 Auxiliary Sessions

```text
≤ 6.0
    ↓
Not supported

6.2.0–6.2.2
    ↓
Permanently enabled

6.2.3+
    ↓
Disabled by default
```

* [ ] Understand auxiliary sessions
* [ ] Understand asymmetric routing
* [ ] Do not confuse them
* [ ] Evaluate SD-WAN/ADVPN symmetry

---

## 🚀 NPU

```text
First Packet
     ↓
CPU
     ↓
Session Establishment
     ↓
NPU Eligibility
     ↓
NPU
     ↓
Fast Forwarding
```

* [ ] Understand CPU role
* [ ] Understand NPU role
* [ ] Understand offload status
* [ ] Understand dirty sessions

---

## 💾 Configuration Save

```text
automatic
    ↓
Automatic save

manual
    ↓
execute cfg save

revert
    ↓
Temporary configuration
+
Timeout
```

---

## 🔐 TPM

```text
TPM
 ↓
Hardware-backed protection
 ↓
Private Data Encryption
 ↓
Sensitive data
```

* [ ] TPM ≠ disk encryption
* [ ] Understand master encryption password
* [ ] Understand backup/restore dependency
* [ ] Understand HA requirements

---

## 🧩 Workspace

```text
start
  ↓
modify
  ↓
commit
```

or:

```text
start
  ↓
modify
  ↓
abort
```

* [ ] Understand transaction timeout
* [ ] Understand transaction locks
* [ ] Understand commit vs abort
* [ ] Know transaction diagnostic commands

---

# 16. Troubleshooting Command Map

| Objective                  | Command                                                 |
| -------------------------- | ------------------------------------------------------- |
| Current date               | `execute date`                                          |
| Current time               | `execute time`                                          |
| NTP status                 | `diagnose sys ntp status`                               |
| PTP debug                  | `diagnose debug application ptpd -1`                    |
| Session table              | `diagnose sys session list`                             |
| TPM hardware info          | `diagnose hardware deviceinfo tpm`                      |
| TPM diagnostics            | `diagnose tpm`                                          |
| Hardware information       | `diagnose hardware test info`                           |
| Transaction metadata       | `diagnose sys config-transaction show txn-meta`         |
| Transaction info           | `diagnose sys config-transaction show txn-info`         |
| Transaction entities       | `diagnose sys config-transaction show txn-entity`       |
| Transaction locks          | `diagnose sys config-transaction show txn-lock`         |
| Transaction CLI commands   | `diagnose sys config-transaction show txn-cli-commands` |
| Transaction memory context | `diagnose sys config-transaction show mctx`             |

---

# 17. Production Hardening Checklist

## 🟢 Administrator

* [ ] Factory/default credential changed
* [ ] Named administrator accounts created
* [ ] Least-privilege profiles assigned
* [ ] MFA enabled where supported
* [ ] Management access restricted
* [ ] HTTPS preferred
* [ ] SSH preferred
* [ ] HTTP exposure reviewed
* [ ] Telnet disabled/unexposed
* [ ] Administrator idle timeout configured
* [ ] Login lockout configured
* [ ] Password policy enabled
* [ ] Password reuse restricted

---

## 🟢 Time & Logging

* [ ] Correct timezone configured
* [ ] DST behavior validated
* [ ] Reliable NTP configured
* [ ] NTP synchronization verified
* [ ] PTP evaluated where required
* [ ] Logs use accurate timestamps
* [ ] SIEM time correlation verified
* [ ] Authentication/security logs reviewed

---

## 🟢 Routing & Sessions

* [ ] Routing symmetry documented
* [ ] ECMP documented
* [ ] PBR documented
* [ ] SD-WAN documented
* [ ] ADVPN documented
* [ ] Multiple WAN paths documented
* [ ] Asymmetric routing evaluated
* [ ] `auxiliary-session` evaluated
* [ ] `asymroute` evaluated
* [ ] Dirty sessions investigated
* [ ] NPU offloading verified
* [ ] CPU impact measured

---

## 🟢 WAN Failover

* [ ] Link Monitor configured
* [ ] Probe destination validated
* [ ] Probe source interface validated
* [ ] Failure threshold validated
* [ ] Recovery threshold validated
* [ ] Static route update behavior validated
* [ ] Policy-route update behavior validated
* [ ] Cascade behavior validated
* [ ] Failover tested
* [ ] Recovery tested
* [ ] Session behavior tested during failover

---

## 🟢 Configuration Safety

* [ ] Configuration backup created
* [ ] Save mode understood
* [ ] `automatic`/`manual`/`revert` selected intentionally
* [ ] Risky remote changes use revert mode where appropriate
* [ ] Revert timeout configured
* [ ] Workspace Mode evaluated for sensitive changes
* [ ] Transaction timeout understood
* [ ] Commit procedure documented
* [ ] Abort procedure documented
* [ ] Transaction locks understood
* [ ] Recovery procedure tested

---

## 🟢 TPM & Private Data

* [ ] TPM support verified
* [ ] Private Data Encryption requirement evaluated
* [ ] Master encryption credential securely stored
* [ ] Recovery procedure documented
* [ ] Configuration backup tested
* [ ] Restore procedure tested
* [ ] HA requirements verified
* [ ] TPM diagnostics verified
* [ ] Team understands TPM ≠ full disk encryption

---

# 🚨 High-Value Exam Traps

## Trap 1 — Auxiliary Session = Asymmetric Routing

❌ Incorrect.

```text
asymroute
    ↓
Asymmetric-routing behavior

auxiliary-session
    ↓
Additional session handling
```

---

## Trap 2 — NPU Processes Everything From Packet One

❌ Oversimplified.

```text
CPU
 ↓
Session establishment/control

NPU
 ↓
Eligible data-plane forwarding
```

---

## Trap 3 — TPM Encrypts the Disk

❌ Incorrect.

```text
TPM
≠
Full Disk Encryption
```

---

## Trap 4 — `ntpv3 disable` Means NTPv3

❌ Incorrect.

```text
ntpv3 enable
    =
NTPv3

ntpv3 disable
    =
NTPv4
```

---

## Trap 5 — Workspace Changes Are Immediately Global

❌ Incorrect.

```text
start
 ↓
modify
 ↓
commit
 ↓
global configuration
```

---

## Trap 6 — `execute cfg save` Reboots the FortiGate

❌ Incorrect.

```text
execute cfg save
    =
Save configuration

execute cfg reload
    =
Reload/reboot behavior
```

---

## Trap 7 — Auxiliary Sessions Should Always Be Enabled in SD-WAN

❌ Incorrect.

```text
SD-WAN
   ↓
Check routing symmetry
   ↓
Evaluate auxiliary-session
   ↓
Test real traffic behavior
```

---

# 🧠 One-Minute Memory Map

```text
                 FORTIGATE SYSTEM HARDENING
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
     SECURITY              TIME              SESSION
        │                   │                   │
 Password Policy          NTP              Auxiliary
 Lockout                  PTP              Asymroute
 Admin Timeout            TZ               ECMP
 Mgmt Ports               Logs             SD-WAN
        │                   │                   │
        └───────────────────┼───────────────────┘
                            │
                         PROCESSING
                            │
                    ┌───────┴───────┐
                    │               │
                   CPU             NPU
                    │               │
              Session Setup     Fast Path
              Policy/Routing     Offload
                    │               │
                    └───────┬───────┘
                            │
                       CONFIGURATION
                            │
                ┌───────────┴───────────┐
                │                       │
             Save Modes             Workspace
             automatic             start
             manual                commit
             revert                abort
                │                   locks
                └───────────┬───────────┘
                            │
                           TPM
                            │
                 Private Data Encryption
                            │
                  Sensitive Credentials
```

---

# 🎯 Final Production Validation

Before declaring the FortiGate hardened:

* [ ] Administrator security reviewed
* [ ] Management exposure minimized
* [ ] Password policy validated
* [ ] Login lockout validated
* [ ] Administrator timeout validated
* [ ] Timezone validated
* [ ] NTP synchronization verified
* [ ] PTP requirement evaluated
* [ ] Routing symmetry documented
* [ ] Auxiliary session requirement evaluated
* [ ] Asymmetric routing requirement evaluated
* [ ] NPU offloading verified
* [ ] Dirty sessions investigated
* [ ] Link Monitor failover tested
* [ ] Configuration save mode validated
* [ ] Workspace transaction behavior tested
* [ ] TPM capability verified
* [ ] Private Data Encryption recovery tested
* [ ] HA/TPM behavior validated
* [ ] Configuration backup verified
* [ ] Disaster-recovery procedure documented

---

# 🔥 SheynShield Takeaway

> **Don't memorize FortiGate commands in isolation.**
>
> Understand the chain:

```text
Administrator
     ↓
Security
     ↓
Time
     ↓
Routing
     ↓
Session State
     ↓
CPU
     ↓
NPU
     ↓
Auxiliary Session
     ↓
Configuration State
     ↓
TPM / Private Data
```

The real NSE4/NSE7 skill is understanding **why FortiGate behaves the way it does**, not simply remembering CLI syntax.

**SheynShield | Engineering Secure Networks**

---

## 🔎 Keywords

`FortiGate system hardening` · `FortiOS 7.2` · `FortiGate NSE4` · `FortiGate NSE7` · `FortiGate security checklist` · `FortiGate hardening checklist` · `FortiGate NTP` · `FortiGate PTP` · `FortiGate auxiliary session` · `FortiGate asymmetric routing` · `FortiGate NPU offloading` · `FortiGate dirty session` · `FortiGate SD-WAN` · `FortiGate ADVPN` · `FortiGate Link Monitor` · `FortiGate configuration save` · `FortiGate workspace mode` · `FortiGate configuration transaction` · `FortiGate TPM` · `FortiGate private data encryption` · `FortiOS CLI` · `FortiGate troubleshooting`

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

> ⭐ **SheynShield Engineering Note**
>
> This checklist is designed for **FortiOS 7.2.x**, NSE4/NSE7 revision, lab validation and production-hardening workflows.
>
> Always validate CLI syntax, feature availability, hardware behavior and version-specific restrictions against the exact FortiOS release deployed in production.

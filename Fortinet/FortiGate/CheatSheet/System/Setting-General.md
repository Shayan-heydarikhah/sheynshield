# FortiGate System Hardening, Time, Session Processing & Configuration Management  

> **FortiOS 7.2.x | NSE4 / NSE7 Practical Reference**
>
> **SheynShield | Engineering Secure Networks**
>
> A practical  covering **Administrator Security, NTP/PTP, Management Ports, Session Processing, Auxiliary Sessions, NPU Offloading, Link Monitor, Configuration Save Modes, TPM and Workspace Transactions**.

---

## Table of Contents

* [1. Administrator Security](#1-administrator-security)
* [2. System Time — NTP & PTP](#2-system-time--ntp--ptp)
* [3. Administrative Ports](#3-administrative-ports)
* [4. Administrator Session Timeout & Lockout](#4-administrator-session-timeout--lockout)
* [5. Auxiliary Sessions](#5-auxiliary-sessions)
* [6. Auxiliary Session vs NPU Processing](#6-auxiliary-session-vs-npu-processing)
* [7. Auxiliary Session Traffic Scenarios](#7-auxiliary-session-traffic-scenarios)
* [8. Session & NPU Troubleshooting](#8-session--npu-troubleshooting)
* [9. Multi-ISP / Asymmetric Routing Example](#9-multi-isp--asymmetric-routing-example)
* [10. Configuration Save Modes](#10-configuration-save-modes)
* [11. TPM & Private Data Encryption](#11-tpm--private-data-encryption)
* [12. Replacement Messages](#12-replacement-messages)
* [13. CLI Configuration Scripts](#13-cli-configuration-scripts)
* [14. Workspace / Configuration Transactions](#14-workspace--configuration-transactions)
* [15. High-Value NSE Exam Points](#15-high-value-nse-exam-points)
* [16. Troubleshooting Command Map](#16-troubleshooting-command-map)
* [17. Quick Hardening Checklist](#17-quick-hardening-checklist)

---

# 1. Administrator Security

## 1.1 Default Administrator

On a factory-default FortiGate, the initial `admin` account behavior depends on the FortiOS/model generation.

### Golden Rule

> **Never leave the factory/default administrator credential unchanged.**

Immediately after initial access:

1. Change the administrator password.
2. Create named administrator accounts.
3. Avoid daily administration with the shared `admin` account.
4. Restrict management access to trusted interfaces.
5. Prefer HTTPS/SSH over insecure management protocols.
6. Enable MFA where supported.

---

## 1.2 Password Reuse

FortiOS password policy can control whether an administrator is allowed to reuse the previous password.

```bash
config system password-policy
    set status enable
    set reuse-password disable
end
```

### Important Logic

| Setting                                    | Result                                            |
| ------------------------------------------ | ------------------------------------------------- |
| `reuse-password enable`                    | Password reuse is allowed                         |
| `reuse-password disable`                   | New password must differ from old password        |
| `min-change-characters > 0`                | Requires a minimum number of characters to differ |
| `min-change-characters` + `reuse-password` | `min-change-characters` takes precedence          |

Example:

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

Fortinet documents `reuse-password` as the control determining whether administrators may reuse the same password; `min-change-characters` overrides it when both are enabled.

---

# 2. System Time — NTP & PTP

## 2.1 Why Time Matters

Accurate system time is **not optional** on a production FortiGate.

Incorrect time can affect:

* Logs
* Event correlation
* Scheduled policies
* Certificates
* SSL-dependent features
* Authentication
* Security investigations
* Troubleshooting
* SIEM correlation

> **First-day rule:** Configure the timezone and a reliable time source during initial deployment.

Fortinet specifically notes that accurate FortiOS system time is required for features such as scheduling, logging and SSL-dependent functionality.

---

## 2.2 Manual Time

```bash
execute date 2026-08-29
execute time 12:30:00
```

Check:

```bash
execute date
execute time
```

---

## 2.3 Timezone

Example for Tehran:

```bash
config system global
    set timezone 41
end
```

FortiOS timezone `41` corresponds to **GMT+03:30 Tehran**.

### DST Warning

```bash
config system global
    set timezone 41
    set dst disable
end
```

`dst` controls daylight-saving-time behavior.

> **Important:** Do not blindly enable DST because an old configuration example contains `set dst enable`. Timezone/DST behavior should match the actual jurisdiction and current FortiOS/timezone rules.

FortiOS exposes `dst` as the CLI control for daylight-saving time.

---

# 2.4 NTP

FortiGate can synchronize its clock using:

* FortiGuard NTP
* Custom NTP servers
* NTP authentication
* NTP server mode

Basic configuration:

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

### NTP Version

```bash
set ntpv3 enable
```

means:

```text
NTPv3
```

while:

```bash
set ntpv3 disable
```

means:

```text
NTPv4
```

FortiOS documents `ntpv3` as the switch for NTPv3; when disabled, NTPv4 is used.

---

## 2.5 NTP Authentication

FortiOS supports NTP authentication using:

* MD5
* SHA1

Example:

```bash
config system ntp
    set ntpsync enable
    set type custom

    config ntpserver
        edit 1
            set server ntp.example.com
            set authentication enable
            set key-id 123
            set key "SECRET"
            set ntpv3 enable
        next
    end
end
```

### Important

| NTP Version | Authentication |
| ----------- | -------------- |
| NTPv3       | MD5            |
| NTPv4       | SHA1           |

Fortinet's CLI reference documents MD5/SHA1 authentication and the `ntpv3` behavior.

> **Security note:** NTP authentication is useful for trusted/internal NTP infrastructure. Do not expose unnecessary NTP services to the Internet.

---

## 2.6 NTP Sync Interval

```bash
config system ntp
    set syncinterval 1
end
```

The value is in **minutes**.

FortiOS supports an NTP synchronization interval from **1 to 1440 minutes**.

---

## 2.7 FortiGate as NTP Server

FortiGate can also provide NTP service to downstream devices.

```bash
config system ntp
    set server-mode enable
    set interface "LAN"
end
```

Conceptually:

```text
                    Internet / Internal NTP
                              |
                              v
                       +-------------+
                       |  FortiGate  |
                       | NTP Client  |
                       +-------------+
                              |
                       NTP Server Mode
                              |
             +----------------+----------------+
             |                |                |
             v                v                v
           PC-01            SW-01            Server
```

FortiGate relays NTP requests to its configured upstream NTP server when server mode is enabled.

---

# 2.8 PTP — Precision Time Protocol

PTP is designed for environments requiring much tighter time synchronization than ordinary NTP.

Typical use cases:

* Telecom
* Industrial systems
* Financial systems
* Time-sensitive applications
* Specialized data-center environments

FortiOS supports PTP configuration including:

* Interface
* Multicast/hybrid mode
* End-to-End delay
* Peer-to-Peer delay

Example:

```bash
config system ptp
    set status enable
    set interface "port1"
    set delay-mechanism E2E
end
```

---

## 2.9 PTP Delay Mechanisms

### End-to-End — E2E

The delay is measured across the complete path between the master and slave.

Simplified concept:

```text
Master                         Slave
  |                              |
  | -------- Sync -------------->|
  |                              |
  | <------ Delay_Req -----------|
  |                              |
  | -------- Delay_Resp -------->|
```

The slave uses timestamps to estimate path delay.

### Peer-to-Peer — P2P

Each link/path segment measures its own propagation delay through peer-delay exchanges.

```text
Master ---- Switch ---- Switch ---- Slave
           P2P delay    P2P delay
```

### Exam Memory

```text
E2E = End-to-End path delay

P2P = Peer-to-Peer link delay
```

---

## 2.10 NTP/PTP Troubleshooting

Check system time:

```bash
execute date
execute time
```

Check NTP:

```bash
diagnose sys ntp status
```

Test UDP/123 reachability:

```bash
execute telnet 192.168.20.200 123
```

> **Note:** TCP telnet to UDP/123 is not a definitive NTP test. It only proves whether something is listening/reachable using the tested transport. For real NTP validation, use FortiGate NTP diagnostics and packet capture.

PTP debugging:

```bash
diagnose debug application ptpd -1
```

---

# 2.11 Windows NTP Server Troubleshooting

On a Windows NTP server, verify:

```cmd
netstat -nao
```

Check the Windows Time service:

```text
services.msc
    Windows Time
```

Restart:

```cmd
net stop w32time
net start w32time
```

### Group Policy

Typical policy location:

```text
Computer Configuration
 └── Administrative Templates
     └── System
         └── Windows Time Service
             ├── Global Configuration Settings
             └── Time Providers
```

Verify:

* Windows NTP Client enabled
* NTP Server configured
* Appropriate AnnounceFlags
* Windows Time service running

---

# 3. Administrative Ports

## 3.1 Change Management Ports

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

### Recommended Security Position

Prefer:

```text
HTTPS
SSH
```

Avoid:

```text
HTTP
TELNET
```

unless there is a specific operational requirement.

---

## 3.2 HTTPS Redirect

```bash
set admin-https-redirect enable
```

Concept:

```text
HTTP :80
   |
   | redirect
   v
HTTPS :443
```

---

## 3.3 Default Service Source Port

```bash
config system global
    set default-service-source-port 20-30
end
```

This affects the source-port range used for system-generated TCP/UDP service traffic.

> Do not confuse this setting with the destination service port of firewall policies.

---

# 4. Administrator Session Timeout & Lockout

## 4.1 Administrator Idle Timeout

FortiOS controls administrator idle timeout through:

```bash
config system global
    set admintimeout 10
end
```

### Security Recommendation

For administrative environments:

```text
5–10 minutes
```

is generally preferable to leaving long idle sessions active.

FortiOS defines `admintimeout` as the number of minutes before an idle administrator session times out.

---

## 4.2 Login Lockout

Example:

```bash
config system global
    set admin-lockout-threshold 2
    set admin-lockout-duration 60
end
```

Meaning:

```text
2 failed attempts
        |
        v
Account locked
        |
        v
60 seconds
```

FortiOS supports up to **10 failed attempts**, while the lockout duration is configured in seconds.

---

## 4.3 Login Logs

GUI:

```text
Log & Report
    >
System Events
    >
General System Events
```

Useful for identifying:

* Failed logins
* Successful logins
* Lockouts
* Administrative activity
* Authentication anomalies

---

# 5. Auxiliary Sessions

## 5.1 What Is an Auxiliary Session?

**Auxiliary Session** allows FortiGate to create additional session state when traffic associated with an existing connection uses a different incoming or outgoing interface/path.

It is particularly relevant to:

* ECMP
* Asymmetric routing
* Multiple WAN links
* Load balancing
* Policy-based routing
* SD-WAN
* ADVPN
* Complex multi-path topologies

Fortinet describes auxiliary sessions as a mechanism for handling changes in incoming, outgoing, or return interfaces in environments such as ECMP and load balancing.

---

# 5.2 Version History

| FortiOS         | Auxiliary Session                   |
| --------------- | ----------------------------------- |
| 6.0 and earlier | Not supported                       |
| 6.2.0–6.2.2     | Permanently enabled                 |
| 6.2.3+          | Disabled by default; can be enabled |

---

# 5.3 Configuration

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

FortiOS 7.2 CLI documents the default as **disabled**.

---

# 5.4 Asymmetric Routing

IPv4:

```bash
config system settings
    set asymroute enable
end
```

IPv6:

```bash
config system settings
    set asymroute6 enable
end
```

ICMP-specific controls also exist:

```bash
set asymroute-icmp enable
set asymroute6-icmp enable
```

FortiOS documents these as separate controls for IPv4, IPv6 and ICMP asymmetric routing.

---

# 5.5 Auxiliary Session ≠ Asymmetric Routing

These are related but **not identical**.

```text
Asymmetric Routing
        |
        +---- Traffic uses different paths
        |
        v
Auxiliary Session
        |
        +---- Helps FortiGate maintain session handling
              when interfaces/paths change
```

### Remember

> `asymroute` controls acceptance/handling of asymmetric routing.

> `auxiliary-session` controls creation/use of auxiliary sessions for changing traffic paths.

---

# 5.6 When Auxiliary Sessions Help

### Scenario

```text
             ISP-1
               |
               v
Client ---- FortiGate ---- Server
               ^
               |
             ISP-2
```

Suppose:

```text
Forward path  = ISP-1
Return path   = ISP-2
```

The original session was created through one interface, but return traffic arrives through another.

Without auxiliary sessions, FortiGate may need to modify/refresh the session state.

With auxiliary sessions:

```text
Main Session
      |
      +---- Auxiliary Session
                   |
                   v
             Alternate Path
```

This can allow traffic to continue while preserving appropriate session handling.

---

# 6. Auxiliary Session vs NPU Processing

## 6.1 Initial Packet Processing

A simplified FortiGate packet-processing model:

```text
             Incoming Packet
                    |
                    v
                  CPU
                    |
             Session Lookup
                    |
             +------+------+
             |             |
           New           Existing
             |             |
             v             v
       Create Session   Session Check
             |             |
             v             v
       Policy Lookup    NPU Eligibility
             |             |
             v             v
        Routing/PBR       NPU
                           |
                           v
                    Fast Forwarding
```

---

# 6.2 CPU Role

The CPU is responsible for control-plane/session-establishment work such as:

* Initial packet processing
* Session lookup
* New session creation
* Policy decisions
* Routing decisions
* Session-state management
* Control-plane functions

---

# 6.3 NPU Role

On supported FortiGate platforms, the NPU can offload eligible data-plane processing.

Typical benefit:

```text
CPU
 |
 | session establishment
 v
NPU
 |
 | high-speed forwarding
 v
Output Interface
```

### Key Concept

> NPU offloading does **not** mean the CPU disappears from the session.

The CPU generally performs the initial/session-control work, while the NPU can handle subsequent eligible packet forwarding.

---

# 6.4 Without Auxiliary Sessions

When traffic changes interface/path:

```text
Existing Session
      |
      v
Interface Changed
      |
      v
Session becomes DIRTY
      |
      v
Session/interface update
      |
      v
CPU processing
```

This can become expensive in environments with:

* ECMP
* Interface flapping
* Multiple WAN paths
* Frequent path changes

---

# 6.5 With Auxiliary Sessions

```text
Existing Session
      |
      v
Path/interface changed
      |
      v
Auxiliary Session
      |
      v
NPU can continue eligible forwarding
```

This can reduce the need to repeatedly modify the original session.

---

# 6.6 Why `dirty` Matters

A session may appear as:

```text
state = dirty
```

A dirty session indicates that the session state requires additional processing/update.

Typical triggers include:

* Path change
* Incoming interface change
* Return interface change
* Routing changes
* ECMP behavior
* Asymmetric traffic

---

# 6.7 NPU Offload Example

Example session output:

```text
npu info:
flag=0x91/0x81
offload=8/8
```

Both directions show offload.

Another example:

```text
npu info:
flag=0x91/0x00
offload=8/0
```

Only one direction is offloaded.

### Memory Trick

```text
offload=x/y

x = one direction
y = reverse direction
```

Always interpret the exact output according to the FortiOS/platform version.

---

# 7. Auxiliary Session Traffic Scenarios

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

### Auxiliary Disabled

Return traffic can use the original session/path.

### Auxiliary Enabled

FortiGate evaluates the applicable route/PBR/SD-WAN decision.

If the best path is:

```text
port1
```

traffic uses port1.

If the best path is:

```text
port2
```

traffic may use port2.

### Priority

When both PBR and SD-WAN-related routing decisions are involved, the exact decision process depends on the configuration and FortiOS version.

> **Do not memorize "PBR always wins" without checking the specific FortiOS routing decision order.**

---

# Scenario 2 — Return Traffic Arrives on a Different Interface

Original:

```text
Session:

port1 ---> FortiGate ---> port3
```

Return arrives:

```text
port4 ---> FortiGate
```

### Auxiliary Disabled

The existing session may become:

```text
DIRTY
```

FortiGate updates the session/interface state.

High traffic or repeated path changes can increase CPU processing.

### Auxiliary Enabled

FortiGate can create:

```text
Original Session
       +
Auxiliary Session
```

allowing traffic to continue using the alternate path.

---

# Scenario 3 — Incoming Traffic Uses a Different Interface

Original:

```text
port1 ---> FortiGate ---> port3
```

New incoming packet:

```text
port2 ---> FortiGate
```

### Auxiliary Disabled

```text
Existing Session
      |
      v
DIRTY
      |
      v
Session update
```

### Auxiliary Enabled

```text
Existing Session
      |
      v
Auxiliary Session
      |
      v
Continue forwarding
```

---

# Scenario 4 — Routing Table Changes

Original:

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

A new route appears:

```text
Server -> port4
```

### Auxiliary Disabled

If the original route remains valid, an established session may continue using the original path rather than immediately moving to the newly preferred route.

### Auxiliary Enabled

A route/interface change can trigger session handling that refreshes the path/interface information.

---

# 7.5 SD-WAN / ADVPN Warning

Auxiliary sessions should **not** be enabled blindly.

Fortinet specifically warns that routing symmetry should be considered in topologies such as:

* SD-WAN Hub-and-Spoke
* ADVPN
* Other environments where return traffic is expected to be symmetric

### Practical Rule

```text
Symmetric traffic expected?
        |
        +---- YES ---> Prefer disabling auxiliary-session
        |
        +---- NO ----> Evaluate auxiliary-session
```

> The correct setting depends on the actual topology and traffic behavior.

---

# 8. Session & NPU Troubleshooting

## 8.1 Session Table

```bash
diagnose sys session list
```

Useful information includes:

```text
state
dev
npu_state
npu info
offload
vlan
qid
```

---

## 8.2 Look for DIRTY Sessions

```bash
diagnose sys session list
```

Search for:

```text
state=dirty
```

or:

```text
state = dirty
```

### Interpretation

```text
DIRTY
  |
  +-- Path/interface changed
  +-- Session state requires update
  +-- Possible CPU processing increase
```

---

## 8.3 NPU Offload

Look for:

```text
npu info:
offload=8/8
```

or:

```text
offload=8/0
```

### Concept

```text
8/8 = both directions offloaded
8/0 = one direction offloaded
```

---

## 8.4 Reflection / Auxiliary Information

Example:

```text
reflect info 0:
dev=37->38/38->37
npu_state=0x000400
npu info:
flag=0x91/0x00
offload=8/0
...
total reflect session num: 1
```

The exact fields are platform/version dependent.

Use the output to correlate:

```text
Session
   |
   +-- Interface
   +-- NPU state
   +-- Offload state
   +-- Auxiliary/reflect information
```

---

# 9. Multi-ISP / Asymmetric Routing Example

## 9.1 Topology

```text
                 ISP-1
              192.168.254.2
                   |
                 port1
                   |
             +-----------+
             |  FGT-1    |
             +-----------+
                   |
                 port3
                   |
             192.168.20.0/24
                   |
                Server


                 ISP-2
              192.168.200.1
                   |
                 port2
                   |
             +-----------+
             |  FGT-1    |
             +-----------+
```

---

# 9.2 Default Routes

Primary:

```text
0.0.0.0/0
    via 192.168.254.2
    distance 10
```

Backup:

```text
0.0.0.0/0
    via 12.12.12.2
    distance 250
```

---

# 9.3 Policy Route

Concept:

```text
Incoming Interface:
    WAN

Destination:
    192.168.20.0/24

Outgoing Interface:
    port3

Gateway:
    12.12.12.2
```

---

# 9.4 Link Monitor

Example:

```bash
config system link-monitor

    edit "WAN1"
        set addr-mode ipv4
        set srcintf "port1"
        set server-config default
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

    edit "WAN2"
        set addr-mode ipv4
        set srcintf "port2"
        set server-config default
        set server-type static
        set server 192.168.200.1
        set protocol ping
        set gateway-ip 192.168.200.1
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

### Important Parameters

| Parameter                  | Meaning                         |
| -------------------------- | ------------------------------- |
| `srcintf`                  | Interface used for the probe    |
| `server`                   | Health-check destination        |
| `protocol`                 | Probe protocol                  |
| `gateway-ip`               | Gateway associated with monitor |
| `interval`                 | Probe interval                  |
| `probe-timeout`            | Probe timeout                   |
| `failtime`                 | Failure threshold               |
| `recoverytime`             | Recovery threshold              |
| `probe-count`              | Probe count                     |
| `update-static-route`      | Update static route state       |
| `update-policy-route`      | Update policy route state       |
| `update-cascade-interface` | Cascade interface state changes |

---

# 9.5 Firewall Policies

For a routing/failover lab:

```text
Firewall policies
    |
    +-- WAN access
    +-- Internal access
```

NAT should be enabled/disabled according to the actual traffic design.

> **Do not blindly use `NAT disabled` in production.** The correct setting depends on whether the traffic requires source NAT.

---

# 9.6 Auxiliary Session Configuration

On both FortiGates:

```bash
config system settings
    set auxiliary-session enable
    set asymroute enable
end
```

### Version Matching

For multi-FortiGate designs:

> **Keep participating FortiGate devices on compatible and preferably identical FortiOS releases when reproducing/operating a feature-sensitive design.**

---

# 9.7 MikroTik Side

Example conceptual Netwatch logic:

```text
Netwatch
 |
 +-- UP
 |    |
 |    +-- Enable primary route
 |    +-- Disable backup route
 |
 +-- DOWN
      |
      +-- Disable primary route
      +-- Enable backup route
```

Example:

```text
Host:
192.168.20.200
```

Route logic:

```text
Primary:
0.0.0.0/0 -> 192.168.254.2

Backup:
0.0.0.0/0 -> 192.168.200.1
```

Masquerade:

```text
IP
 └── Firewall
      └── NAT
           └── masquerade
```

---

# 10. Configuration Save Modes

FortiGate provides configuration save behavior through:

```bash
config system global
    set cfg-save automatic
end
```

Available modes include:

```text
automatic
manual
revert
```

Fortinet documents these configuration-save modes under `config system global`.

---

# 10.1 Automatic

```bash
config system global
    set cfg-save automatic
end
```

Changes are automatically saved.

Concept:

```text
CLI/GUI Change
      |
      v
RAM
      |
      v
Flash
```

---

# 10.2 Manual

```bash
config system global
    set cfg-save manual
end
```

Changes are applied but must be explicitly saved.

```bash
execute cfg save
```

---

# 10.3 Revert Mode

```bash
config system global
    set cfg-save revert
    set cfg-revert-timeout 600
end
```

Concept:

```text
Change configuration
       |
       v
Temporary state
       |
       +---- Commit/save ---> Keep changes
       |
       +---- Timeout/reboot -> Revert
```

`600` means **600 seconds**.

---

# 10.4 Save vs Reload

Save:

```bash
execute cfg save
```

Reload/reboot:

```bash
execute cfg reload
```

### Memory Trick

```text
SAVE   = write configuration

RELOAD = reboot/reload device
```

Fortinet documents `execute cfg save` and `execute cfg reload` in configuration save mode.

---

# 11. TPM & Private Data Encryption

## 11.1 Trusted Platform Module

On supported FortiGate hardware, TPM can protect sensitive cryptographic material.

TPM can help protect:

* Passwords
* Keys
* Sensitive configuration data

Fortinet states that TPM is disabled by default on supported FortiGate hardware.

---

# 11.2 Private Data Encryption

Enable:

```bash
config system global
    set private-data-encryption enable
end
```

FortiGate then prompts for a:

```text
32 hexadecimal digit
master/private data encryption key
```

---

# 11.3 Cryptographic Hierarchy

Simplified:

```text
32-hex-digit
Master Encryption Password
          |
          v
       AES-128-CBC
          |
          v
Sensitive Data

          +
          |
          v
        TPM
          |
          v
     RSA-2048
          |
          v
Primary Key
```

Fortinet describes the master-encryption-password as protecting sensitive data using AES-128-CBC and the TPM-generated 2048-bit primary key as protecting that master password using RSA-2048.

---

# 11.4 TPM Does NOT Mean Disk Encryption

> **Critical exam point:**

```text
TPM != Full Disk Encryption
```

Fortinet explicitly notes that the TPM module does **not** encrypt the disk drive.

---

# 11.5 Backup / Restore Dependency

TPM-protected configuration introduces an important dependency.

```text
Backup
  |
  v
TPM-protected encryption information
  |
  v
Restore
```

Restore behavior depends on:

```text
TPM state
+
Master Encryption Password
```

### Key Rules

| Restore Condition                 | Result                                     |
| --------------------------------- | ------------------------------------------ |
| TPM disabled                      | Protected configuration cannot be restored |
| TPM enabled + wrong master key    | Cannot restore protected configuration     |
| TPM enabled + matching master key | Configuration can be restored              |

---

# 11.6 HA + TPM

For an HA cluster:

```text
FGT-A
  |
  +-- Master Encryption Key
  |
  v
FGT-B
```

### Important

> HA members must use the **same master encryption key** for the cluster to form correctly and synchronize protected configuration.

---

# 11.7 TPM Troubleshooting

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

# 11.8 Sensitive Data Protected by Private Data Encryption

Examples include:

* Alert email credentials
* BGP/routing-related credentials
* External resource credentials
* FortiGuard proxy password
* FortiToken seeds
* HA password
* IPsec PSK
* Link Monitor server password
* Local certificate private keys
* LDAP/RADIUS/FSSO credentials
* Modem/PPPoE credentials
* NTP credentials
* SDN connector credentials
* SNMP credentials
* Wireless security credentials

### Memory Trick

```text
Private Data Encryption
        |
        +-- Passwords
        +-- PSKs
        +-- Private Keys
        +-- Authentication Secrets
        +-- Service Credentials
```

---

# 12. Replacement Messages

FortiGate replacement messages can customize user-facing messages generated by security features.

Common groups/categories include:

```text
UTM
 ├── admin
 ├── alertmail
 ├── custom-message
 ├── fortiguard-wf
 ├── ftp
 ├── http
 ├── icap
 ├── mail
 ├── nac-quar
 ├── spam
 ├── sslvpn
 ├── traffic-quota
 ├── utm
 └── webproxy

AUTH
 ├── auth
 └── webproxy
```

Enable GUI replacement-message groups:

```bash
config system settings
    set gui-replacement-message-groups enable
end
```

---

# 12.1 Email Filter Example

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

### Key Dependency

```text
Firewall Policy
       |
       +-- Proxy inspection
       |
       +-- Email Filter Profile
       |
       +-- Replacement Message
```

---

# 13. CLI Configuration Scripts

FortiGate GUI provides a CLI/script interface for deploying configuration commands.

Typical workflow:

```text
GUI
 |
 v
CLI / Script
 |
 v
Paste commands
 |
 v
Execute
```

---

## 13.1 Comments

A CLI script comment begins with:

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

Everything after `#` on that line is treated as a comment.

---

# 14. Workspace / Configuration Transactions

## 14.1 Why Workspace Mode?

Workspace/transaction mode is useful when configuration changes need controlled deployment.

Concept:

```text
Administrator A
       |
       v
Transaction
       |
       +---- Validate
       |
       +---- Commit
       |
       +---- Abort
```

Instead of immediately exposing every change to the rest of the system, configuration changes are held in a transaction until committed.

Fortinet documents Workspace Mode as a change-control mechanism where changes are made in a local CLI process and become available to other processes after commit.

---

# 14.2 Start Transaction

```bash
execute config-transaction start
```

Or specify a timeout:

```bash
execute config-transaction start 5
```

The timeout is specified in **minutes**.

FortiOS supports a transaction timeout from **1 to 60 minutes**, with a default of 5 minutes.

---

# 14.3 Commit

```bash
execute config-transaction commit
```

Concept:

```text
Transaction
    |
    v
Commit
    |
    v
Configuration becomes active globally
```

---

# 14.4 Abort

```bash
execute config-transaction abort
```

Concept:

```text
Transaction
    |
    v
Abort
    |
    v
Discard transaction changes
```

---

# 14.5 Transaction Expiration

Example:

```text
config transaction id=1 will expire in 30 seconds
config transaction id=1 will expire in 20 seconds
config transaction id=1 will expire in 10 seconds
config transaction id=1 has expired
```

### Exam Point

> If the administrator does not commit the transaction before expiration, the transaction expires.

---

# 14.6 Objects Locked by Another Transaction

You may encounter:

```text
Can not config the object since either the object or the referenced objects
are being configured by other transactions.

Command fail.
Return code 14
```

Meaning:

```text
Another active transaction
        |
        v
Object/reference locked
        |
        v
Your configuration cannot modify it
```

---

# 14.7 Transaction Troubleshooting

### Transaction Metadata

```bash
diagnose sys config-transaction show txn-meta
```

Useful for:

```text
Active transaction information
Concurrent transaction/session information
```

---

### Transaction Information

```bash
diagnose sys config-transaction show txn-info
```

Useful for:

```text
Transaction changes
Modification information
```

---

### Transaction Entities

```bash
diagnose sys config-transaction show txn-entity
```

Shows affected entities.

---

### Transaction Locks

```bash
diagnose sys config-transaction show txn-lock
```

Shows locked objects.

Useful when another administrator is modifying configuration.

---

### Transaction CLI Commands

```bash
diagnose sys config-transaction show txn-cli-commands
```

Shows CLI commands associated with the transaction.

---

### Memory Context

```bash
diagnose sys config-transaction show mctx
```

Useful for examining memory/context information related to the transaction.

> If memory resources become exhausted, configuration transaction operations may be affected.

---

# 14.8 Commands Restricted in Workspace Mode

Some configuration areas cannot be changed through a workspace transaction.

Examples include:

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
end

config system settings
set opmode
end

config system npu
config system np6
config system wireless
set mode
end

config system vdom-property
config system storage
```

Fortinet documents these restrictions for Workspace Mode.

---

# 15. High-Value NSE Exam Points

## Administrator

```text
admin-lockout-threshold
        |
        v
Failed login count

admin-lockout-duration
        |
        v
Lockout time
```

---

## Password Policy

```text
reuse-password enable
        |
        v
Password reuse allowed
```

```text
reuse-password disable
        |
        v
New password required
```

```text
min-change-characters > 0
        |
        v
Overrides reuse-password
```

---

## Time

```text
NTP
 |
 +-- Normal network time synchronization

PTP
 |
 +-- Precision/time-sensitive environments
```

---

## NTP Version

```text
ntpv3 enable
    =
NTPv3

ntpv3 disable
    =
NTPv4
```

---

## PTP

```text
E2E = End-to-End

P2P = Peer-to-Peer
```

---

## Auxiliary Session

```text
FortiOS <= 6.0
    -> Not supported

6.2.0 - 6.2.2
    -> Permanently enabled

6.2.3+
    -> Disabled by default
```

---

## Auxiliary Session

```text
ECMP
SD-WAN
Multiple paths
Asymmetric traffic
       |
       v
Auxiliary Session
       |
       v
Additional session handling
```

---

## Asymmetric Routing

```bash
config system settings
    set asymroute enable
end
```

---

## NPU

```text
First packet / session setup
            |
            v
           CPU
            |
            v
       Session created
            |
            v
       NPU eligibility
            |
            v
           NPU
            |
            v
    High-speed forwarding
```

---

## Dirty Session

```text
Path/interface change
        |
        v
Session update required
        |
        v
DIRTY
```

---

## Configuration Save

```text
automatic
    |
    +-- Automatically save

manual
    |
    +-- execute cfg save

revert
    |
    +-- Save point + timeout
```

---

## Workspace Transaction

```text
start
  |
  v
modify
  |
  +---- abort ---> discard
  |
  +---- commit --> apply
```

---

## TPM

```text
Private Data Encryption
          |
          v
Master Encryption Password
          |
          +---- AES-128-CBC
          |
          v
Sensitive Data

TPM
 |
 +---- RSA-2048 Primary Key
 |
 v
Protects Master Encryption Password
```

---

# 16. Troubleshooting Command Map

| Goal                       | Command                                                 |
| -------------------------- | ------------------------------------------------------- |
| Current date               | `execute date`                                          |
| Current time               | `execute time`                                          |
| NTP status                 | `diagnose sys ntp status`                               |
| PTP debug                  | `diagnose debug application ptpd -1`                    |
| Session table              | `diagnose sys session list`                             |
| TPM hardware info          | `diagnose hardware deviceinfo tpm`                      |
| TPM diagnostics            | `diagnose tpm`                                          |
| Hardware test              | `diagnose hardware test info`                           |
| Transaction metadata       | `diagnose sys config-transaction show txn-meta`         |
| Transaction info           | `diagnose sys config-transaction show txn-info`         |
| Transaction entities       | `diagnose sys config-transaction show txn-entity`       |
| Transaction locks          | `diagnose sys config-transaction show txn-lock`         |
| Transaction CLI commands   | `diagnose sys config-transaction show txn-cli-commands` |
| Transaction memory context | `diagnose sys config-transaction show mctx`             |

---

# 17. Quick Hardening Checklist

## Initial Deployment

* [ ] Change factory/default admin credential
* [ ] Create named administrator accounts
* [ ] Enable appropriate password policy
* [ ] Disable unnecessary password reuse
* [ ] Configure administrator idle timeout
* [ ] Configure login lockout
* [ ] Restrict management interfaces
* [ ] Prefer HTTPS/SSH
* [ ] Disable unnecessary Telnet/HTTP exposure
* [ ] Configure correct timezone
* [ ] Configure NTP
* [ ] Verify NTP synchronization
* [ ] Configure logging destination
* [ ] Verify certificate/time-dependent features

---

## Routing / Session Design

* [ ] Identify whether routing is symmetric
* [ ] Identify ECMP
* [ ] Identify SD-WAN
* [ ] Identify PBR
* [ ] Identify ADVPN
* [ ] Identify multiple ISP paths
* [ ] Determine whether return traffic can change interface
* [ ] Evaluate `auxiliary-session`
* [ ] Evaluate `asymroute`
* [ ] Verify NPU offload
* [ ] Investigate `dirty` sessions
* [ ] Compare CPU/NPU behavior under load

---

## Configuration Safety

* [ ] Use Workspace Mode for sensitive changes
* [ ] Understand transaction timeout
* [ ] Commit deliberately
* [ ] Abort failed changes
* [ ] Check transaction locks
* [ ] Use `cfg-save revert` for risky remote changes
* [ ] Set a reasonable revert timeout
* [ ] Keep configuration backups

---

## TPM / Sensitive Data

* [ ] Verify TPM support
* [ ] Understand Private Data Encryption
* [ ] Protect the master encryption key
* [ ] Document HA key requirements
* [ ] Test backup/restore procedures
* [ ] Understand TPM restore dependencies
* [ ] Remember: TPM is **not** disk encryption

---

# 🔥 One-Minute Memory Map

```text
                    FORTIGATE SYSTEM BASICS
                             |
       +---------------------+----------------------+
       |                     |                      |
    SECURITY                TIME                 SESSION
       |                     |                      |
 Password Policy            NTP                 Auxiliary
 Lockout                    PTP                 Asymroute
 Admin Timeout              TZ                  ECMP
 Management Ports           Logs                SD-WAN
       |                     |                      |
       +---------------------+----------------------+
                             |
                         PROCESSING
                             |
                     +-------+-------+
                     |               |
                    CPU             NPU
                     |               |
              Session Setup      Fast Path
              Policy/Routing     Offloading
                             |
                             v
                      CONFIGURATION
                             |
                +------------+------------+
                |                         |
          Save Modes                 Transactions
          automatic                 start/commit
          manual                    abort/timeout
          revert                    locks
                             |
                             v
                            TPM
                             |
                    Private Data Encryption
```

---

# 🎯 Interview / Exam Traps

### Trap 1

> **Auxiliary Session and Asymmetric Routing are the same thing.**

❌ Wrong.

```text
asymroute          = asymmetric routing behavior
auxiliary-session  = additional session handling
```

---

### Trap 2

> **NPU handles the entire session from the first packet.**

❌ Oversimplified.

```text
CPU -> session establishment/control
NPU -> eligible data-plane forwarding
```

---

### Trap 3

> **TPM encrypts the FortiGate disk.**

❌ Wrong.

```text
TPM protects cryptographic material.
TPM != Full Disk Encryption
```

---

### Trap 4

> **NTPv3 is used when `ntpv3 disable` is configured.**

❌ Wrong.

```text
ntpv3 enable  = NTPv3
ntpv3 disable = NTPv4
```

---

### Trap 5

> **Workspace changes are immediately committed globally.**

❌ Wrong.

```text
start
  |
modify
  |
commit
```

Changes become available globally after commit.

---

### Trap 6

> **`execute cfg save` means reboot.**

❌ Wrong.

```text
execute cfg save   = save configuration

execute cfg reload = reload/reboot
```

---

### Trap 7

> **Auxiliary sessions should always be enabled in SD-WAN.**

❌ Wrong.

Routing symmetry must be considered. Fortinet specifically recommends evaluating the effect on symmetric return traffic in SD-WAN hub/spoke and ADVPN-type topologies.

---

# 🔎 SEO Keywords

`FortiGate ` · `FortiOS 7.2 ` · `FortiGate NSE4` · `FortiGate NSE7` · `Fortinet NSE4 notes` · `Fortinet NSE7 notes` · `FortiGate auxiliary session` · `FortiGate asymmetric routing` · `FortiGate NPU offloading` · `FortiGate NTP` · `FortiGate PTP` · `FortiGate TPM` · `FortiGate workspace mode` · `FortiGate configuration transaction` · `FortiGate troubleshooting commands` · `FortiGate hardening` · `FortiGate session troubleshooting` · `FortiOS CLI `

---

# 📌 SheynShield Takeaway

> **The real skill is not memorizing CLI commands.**
>
> Understand the relationship between:
>
> **Routing → Session State → CPU → NPU → Auxiliary Session → Configuration State**
>
> Once this chain is clear, troubleshooting complex FortiGate behavior becomes much easier.

**SheynShield | Engineering Secure Networks**


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

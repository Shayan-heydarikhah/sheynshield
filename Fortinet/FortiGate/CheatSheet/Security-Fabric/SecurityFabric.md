# 🛡️ FortiOS 7.2.0 — Security Fabric

> **FortiOS Version:** `7.2.0`
> **Focus:** Security Fabric · Automation · External Connectors · SAML · FortiManager · FortiAnalyzer Cloud · EMS · OT/Purdue · Security Rating
> **Style:** Quick Reference / CLI / Troubleshooting
>
> **Lab Note:** Commands and behaviors below are based on the **FortiOS 7.2.0 training environment** used for this reference.

---

# 🧭 01 — Security Fabric Mental Model

Think of Security Fabric as:

```text
                         ┌──────────────────────┐
                         │    ROOT FORTIGATE    │
                         │  Fabric Coordinator  │
                         └──────────┬───────────┘
                                    │
             ┌──────────────────────┼──────────────────────┐
             │                      │                      │
             ▼                      ▼                      ▼
        Downstream FGT         FortiSwitch / AP         FortiNAC
             │
             ├── EMS
             ├── FortiAnalyzer
             ├── FortiManager
             ├── FortiSandbox
             └── FortiDR
```

### The rule to remember

```text
Root device
   ↓
Fabric synchronization
   ↓
Downstream devices
   ↓
Shared objects / identities / security information
```

But **not everything must be inherited**.

Security Fabric can operate with:

| Mode                 | Behavior                                                               |
| -------------------- | ---------------------------------------------------------------------- |
| `Default / Unified`  | Objects can be inherited from the root                                 |
| `Local`              | Device maintains its own values                                        |
| `Configuration Sync` | Used for FortiManager / FortiAnalyzer / FortiSandbox-type integrations |

---

# ⚙️ 02 — Fabric Synchronization

### Main configuration

```bash
config system csf
    set fabric-object-unification enable
    set configuration-sync local
    set fabric-workers 2
end
```

### What matters?

```text
fabric-object-unification
        ↓
FortiGate objects
        ↓
Root → downstream synchronization
```

```text
configuration-sync
        ↓
FortiManager
FortiAnalyzer
FortiSandbox
EMS-related synchronization
```

### Fabric workers

```text
1 ─────────────── 4
        ↑
      default
```

Controls the number of processes handling synchronization.

---

# ⚠️ 03 — The "Local Device" Trap

Consider:

```text
FGT-1 (Default)
       │
       ▼
FGT-2 (Local)
       │
       ▼
FGT-3 (Default)
```

FGT-2 may not display inherited objects in the normal object view as expected.

But it can still **advertise objects toward downstream devices**.

### Important behavior

If an object from the root is referenced by another object downstream:

```text
Root Object
     ↓
Referenced Object
     ↓
Downstream Device
```

Changing the original root object later does **not necessarily mean every referenced downstream value changes as you expect**.

### VDOM warning

> ⚠️ Fabric synchronization behavior changes when VDOMs are involved.

Before enabling VDOMs:

```text
BACKUP
  ↓
Enable VDOM
  ↓
Validate NPU / addressing
  ↓
Validate Fabric
```

Also remember:

> Fabric synchronization cannot be run on multi-mode VDOMs.

---

# 🔐 04 — Security Fabric over IPsec

A Fabric connection can be built over an IPsec tunnel.

### Basic flow

```text
FGT-A
  │
  │ IPsec
  ▼
FGT-B
```

### IPsec baseline

```text
VPN
 └── IPsec Tunnels
      └── Site-to-Site
           ├── Static IP
           ├── Interface
           ├── Pre-shared Key
           ├── Main Mode
           └── Encryption
```

### After tunnel creation

Do not stop at Phase 1/2.

Validate:

```text
IPsec
  ↓
Firewall Policy
  ↓
Routing
  ↓
IPsec Interface IP
  ↓
Administrative Access
  ↓
Fabric Connectivity
```

Example interface addressing:

```text
FGT-A
12.23.34.1

       IPsec

FGT-B
12.23.34.2
```

Then:

```bash
execute ping 12.23.34.2
```

### Security principle

> Keep the allowed interfaces and address objects as restricted as possible when using IPsec for Fabric communication.

---

# 🔗 05 — Fabric Connectivity Diagnostics

### Pending authorization requests

```bash
diagnose sys csf authorization pending-list
```

### Downstream devices

```bash
diagnose sys csf downstream
```

### Upstream / root information

```bash
diagnose sys csf upstream
```

### Security Fabric status

```bash
diagnose sys csf
```

### Show current Fabric configuration

```bash
show system csf
```

---

# 🤖 06 — Automation

Automation can operate in two different contexts:

```text
                 AUTOMATION
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
   Security Fabric         Standalone
          │                     │
   Inherits root          Own commands
   behavior/context       / automation
```

### Maximum concurrent stitches

```bash
config system automation-setting
    set max-concurrent-stitches 128
end
```

Range:

```text
32 ───────────── 256
```

### Operational recommendation

When restoring images / signatures through scheduled automation:

```text
Parallel
   ↓
Fast
   ↓
Potentially harder to control

Sequential
   ↓
Controlled execution
   ↓
Better when commands require delay/order
```

> 💡 For scheduled restore operations, sequential execution can be preferable when command ordering and delay matter.

---

# 🧪 07 — Automation Troubleshooting

### Test automation schedule

```bash
diagnose automation test x-schedule
```

Use this when you want to verify the automation execution path.

### Check automation daemon

```bash
diagnose test application autod
```

### Deep automation debugging

```bash
diagnose debug enable
diagnose debug application autod -1
```

Stop debugging when finished:

```bash
diagnose debug disable
```

---

# 🌐 08 — External Connectors

External connectors provide external identity, endpoint, and intelligence information.

### Identity / Endpoint examples

```text
FSSO Agent
Symantec Endpoint Protection
RADIUS Single Sign-On Agent
Exchange Server
```

### Threat Feed sources

```text
IP Addresses
Domain Names
Malware Hashes
STIX / TAXII
HTTP-based block lists
```

---

# 🧱 09 — Threat Feed: Think "External Data → Firewall Object"

The concept is simple:

```text
External Source
      │
      ▼
Threat Feed
      │
      ▼
Dynamic Object
      │
      ▼
Firewall Policy
      │
      ▼
Enforcement
```

A plain-text feed:

```text
192.168.10.10
192.168.20.20
192.168.30.0/24
```

Each line represents one entry.

---

# 📌 10 — Threat Feed Practical Rules

### Supported data examples

```text
IPv4
IPv6
Domains
URLs
Malware Hashes
```

### IPv4 ranges

Prefer:

```text
192.168.254.0/24
```

### IPv6

Do not use square brackets for a plain IPv6 address.

For URLs containing IPv6:

```text
http://[IPv6-address]/feed.txt
```

### Wildcards

For domain-based feeds:

```text
*.google.com
```

### URL encoding

IDN and UTF-related URL encoding can be supported.

---

# 🚨 11 — Threat Feed Capacity Notes

From the 7.2.0 training notes:

```text
Maximum file size:
10 MB

Maximum entries:
131072
```

Resource limits can vary by FortiGate model.

Check the actual table size instead of assuming a universal hardware limit.

```bash
print tablesize
```

### External resource table

```text
system.external-resource

0
256
512
```

Reference from the training notes:

```text
512 → global limit
256 → per VDOM
```

> ⚠️ Avoid duplicate entries. Large or duplicated feeds can increase resource consumption.

---

# 🧩 12 — Threat Feed Categories & VDOM Objects

When VDOMs are involved, object/category identifiers can appear differently.

Training reference:

```text
g-cat-192
   ↓
Global object

cat-192
   ↓
Root / VDOM object
```

### FortiGuard threat-feed style

```text
Name:
x

URL:
stix://<feed-server>/...
```

---

# 🔍 13 — Dynamic List Verification

To inspect dynamically resolved objects:

```bash
diagnose firewall dynamic list
```

Think:

```text
Connector
   ↓
Feed / Identity
   ↓
Dynamic List
   ↓
Policy Match
```

---

# ☸️ 14 — Kubernetes Connector Troubleshooting

For Kubernetes-related connector communication:

```bash
diagnose debug application kubed -1
diagnose debug enable
```

Use this to inspect Kubernetes verification and transmission activity.

---

# 🛡️ 15 — Symantec Connector Troubleshooting

Symantec connector communication:

```text
Port:
8446
```

Debug:

```bash
diagnose debug application sepmd -1
diagnose debug enable
```

---

# 👤 16 — Users With No GUI Authentication Tracking

Sometimes authenticated-user traffic is not visible through the expected GUI tracking view.

Use:

```bash
diagnose firewall auth list
```

Think:

```text
GUI visibility
      ↓
Not enough?
      ↓
CLI
      ↓
diagnose firewall auth list
```

---

# 🧭 17 — KDC Auto-Discovery

Enable WAD debugging:

```bash
diagnose wad debug enable category all
diagnose wad debug enable level verbose
diagnose debug enable
```

Then test:

```bash
diagnose wad user exchange test-auto-discover
```

### WAD

```text
WAD
 ↓
Web Application Daemon
 ↓
Handles proxy / web-related processes
```

---

# 🧬 18 — STIX vs TAXII

| Technology | Think About It As                                      |
| ---------- | ------------------------------------------------------ |
| **STIX**   | Threat intelligence representation / expression        |
| **TAXII**  | Transport / exchange mechanism for threat intelligence |

### Mental shortcut

```text
STIX
"What is the intelligence?"

TAXII
"How do we exchange it?"
```

---

# 🔑 19 — SAML in Security Fabric

SAML can provide:

```text
Authentication
+
Authorization
+
SSO
```

Typical architecture:

```text
                    ┌──────────────┐
                    │     IDP      │
                    │              │
                    │ AD / ADFS /  │
                    │ Directory    │
                    └──────┬───────┘
                           │
                         SAML
                           │
                           ▼
                    ┌──────────────┐
                    │     SP       │
                    │  FortiGate   │
                    └──────────────┘
```

### Security Fabric model

```text
ROOT FORTIGATE
      │
      │ Identity Provider
      ▼
DOWNSTREAM FORTIGATE
      │
      │ Service Provider
      ▼
      User
```

> ⭐ Remember this one:
>
> **Root = IDP**
> **Downstream = SP**

---

# 🏢 20 — Microsoft ADFS + FortiGate SAML

Training topology:

```text
Active Directory
       │
       ▼
   AD CS / CA
       │
       ▼
     ADFS
       │
       │ SAML
       ▼
FortiGate Security Fabric
```

Example domain:

```text
test.com
```

Example ADFS names:

```text
ADFS Service:
adfs.test.com

Certificate CN:
shayan.adfs.test.com
```

---

# 🔏 21 — AD CS Certificate Preparation

Windows Certificate Authority:

```text
Certificate Templates
        ↓
Manage
        ↓
Web Server
        ↓
Duplicate Template
```

Training configuration notes:

```text
Request Handling
 └── Allow private key export

General
 └── Publish / identify template

Cryptography
 └── Minimum key size: 512

Security
 └── Allow enrollment
```

Then create the required certificate template based on these settings.

---

# 🖥️ 22 — MMC Certificate Enrollment

Open:

```text
MMC
 ↓
Add/Remove Snap-in
 ↓
Certificates
 ↓
Computer Account
 ↓
Personal
 ↓
Certificates
```

Certificate example:

```text
Subject:
CN=shayan.adfs.test.com

DNS:
adfs.test.com
```

Private key:

```text
Exportable
```

---

# 📜 23 — Export CA Certificate

Windows Certificate Services web enrollment:

```text
http://127.0.0.1/certsrv
```

Export the required CA certificate and import it into FortiGate using:

```text
Base64
```

Then:

```text
FortiGate
   ↓
Generate CSR
   ↓
Sign CSR with CA
   ↓
Import signed certificate
   ↓
SAML integration
```

---

# 🧑‍💻 24 — ADFS Configuration Flow

Install:

```text
ADFS Role
```

Configure:

```text
SSL Certificate
    ↓
shayan.adfs.test.com

Federation Service Name
    ↓
adfs.test.com
```

Then configure:

```text
Groups
   ↓
Administrators
```

SAML settings can be tuned through:

```text
Edit Federation Service
    ↓
SAML-related values
```

Create:

```text
Relying Party Trust
```

Enable:

```text
SAML 2.0 Web SSO Protocol
```

---

# 🔐 25 — FortiAuthenticator as SAML IDP

Basic architecture:

```text
Active Directory
       │
       ▼
FortiAuthenticator
       │
       │ SAML
       ▼
FortiGate
```

Example interface:

```bash
config system interface
    edit "port1"
        set ip 192.168.254.230 255.255.255.0
        set allowaccess ssh https
    next
end
```

System settings from the training lab:

```bash
config system global
    set timezone 41
    set allowed-hops 192.168.254.254
end
```

---

# 🧩 26 — FortiAuthenticator SAML Components

### Remote Authentication

```text
Authentication
 └── Remote Auth Server
      └── LDAP
```

### User Source

```text
User Management
 └── Realms
      └── New User Source
```

### SAML IDP

```text
SAML IDP
 ├── General
 ├── Service Provider
 ├── Certificates
 └── Attributes
```

Useful SAML attributes:

```text
username
userprincipalname
```

Single Logout:

```text
Enable
```

---

# 🌍 27 — Internet-Based SAML IDP

If the IDP is exposed through the Internet:

```text
Internet
   │
   ▼
Public IP
   │
   ▼
SAML IDP
   │
   ▼
Certificate
```

The IDP requires:

```text
Reachable address
+
Valid certificate
```

---

# 🧱 28 — FortiGate SAML: SP vs IDP

### Root FortiGate

```text
Security Fabric SSO
       ↓
Identity Provider
```

Configure:

```text
IP Address
Certificate
```

> ⚠️ Prefer the **specific management IP address** rather than an ambiguous interface/address.

### Downstream FortiGate

```text
Security Fabric
       ↓
SAML SSO
       ↓
Service Provider
```

Define:

```text
Root device
IDP prefix
Remote certificate / IDP certificate
```

---

# 🚪 29 — Downstream SAML Login Behavior

Two important modes:

### Normal

```text
SSO
+
Normal Login
```

Useful when you want a local administrative fallback.

### SSO

```text
Downstream
     ↓
Redirect
     ↓
Root FortiGate
     ↓
SAML Authentication
```

### Recommended operational mindset

```text
SSO enabled
   +
Local admin fallback
   =
Better recovery path
```

---

# 👑 30 — SAML Administrator Mapping

On the root FortiGate:

```text
System
 └── Administrators
      └── New SSO Administrator
```

Assign the appropriate:

```text
Remote Group
+
Administrator Profile
```

Also define required:

```text
Firewall Groups
FSSO Groups
Remote Groups
```

---

# ☁️ 31 — FortiAnalyzer Cloud / FortiCloud Logging

Think of cloud logging as:

```text
FortiGate
   │
   ├── UTM Logs
   └── Event Logs
          │
          ▼
   FortiAnalyzer Cloud
```

Training notes:

```text
FortiAnalyzer Cloud
 └── Cloud-based logging / analytics
```

The required subscription depends on the service.

> ⚠️ In the **FortiOS 7.2.0 training environment**, the noted FortiAnalyzer Cloud limitations include DLP and IPS archive limitations.

---

# 📝 32 — FortiCloud Logging Configuration

```bash
config log fortiguard setting
    set status enable
    set upload-option realtime
end
```

### Important

Use the appropriate default gateway / routing path for cloud communication.

---

# 🧩 33 — VDOM Override for FortiAnalyzer Cloud

If the global configuration disables the service, a VDOM cannot simply override it without the relevant global setting.

Example:

```bash
config log setting
    set faz-override enable
end
```

Then:

```bash
config log fortianalyzer-cloud override-setting
    set status enable
end
```

Filtering:

```bash
config log fortianalyzer-cloud override-filter
    set severity information
    set forward-traffic disable
    set local-traffic disable
    set multicast-traffic disable
    set sniffer-traffic disable
    set anomaly disable
    set voip disable
    set dlp-archive disable
end
```

---

# 🔎 34 — FortiAnalyzer Cloud CLI Verification

When GUI visibility is insufficient:

```bash
execute log filter device fortianalyzer-cloud
execute log filter category event
execute log display
```

### Remember

```text
GUI
 ↓
Limited visibility

CLI
 ↓
Verification
```

---

# ☁️ 35 — FortiCloud vs FortiAnalyzer Cloud

Don't mix them up.

```text
FortiAnalyzer Cloud
        ↓
Logging
Analytics
```

```text
FortiGate Cloud
        ↓
Hosted FortiGate management
Logging / retention
Cloud services
```

> ⚠️ From the training notes: use the intended cloud service according to the required feature set rather than enabling overlapping services unnecessarily.

---

# 🏢 36 — FortiManager

Mental model:

```text
FortiManager
     │
     │ Central Management
     ▼
FortiGate Devices
```

Similar concept to:

```text
WSUS
  ↓
Central Update
  ↓
Managed Clients
```

Management communication:

```text
TCP/541
```

---

# ☁️ 37 — FortiManager Cloud

FortiManager Cloud uses the relevant FortiCloud account/subscription relationship.

The service endpoint must resolve:

```text
fortimanager.forticloud.com
```

Verify central management status:

```bash
diagnose fdsm central-mgmt-status
```

Think:

```text
DNS
 ↓
Cloud reachability
 ↓
Registration
 ↓
Central Management
```

---

# 🧪 38 — FortiGuard / Subscription Diagnostics

Check subscription and support information:

```bash
diagnose test update info
```

Useful when you need to distinguish:

```text
Configuration problem
        vs
Subscription problem
        vs
Connectivity problem
```

---

# 🧪 39 — FortiCloud Sandbox

Training notes distinguish:

```text
FortiGate AV License
        +
FortiCloud / Sandbox entitlement
        ↓
Cloud Sandbox functionality
```

Check sandbox region:

```bash
execute forticloud-sandbox region
```

Configure:

```bash
config system fortiguard
    set sandbox-region <REGION>
end
```

Region values from the lab:

```text
0 → Europe
1 → Global
2 → Japan
3 → US
```

Update:

```bash
execute forticloud-sandbox update
```

---

# 🔌 40 — EMS Integration

Typical architecture:

```text
FortiGate
    │
    ▼
FortiClient EMS
    │
    ├── Tags
    ├── Vulnerabilities
    ├── Malware Hashes
    └── Endpoint Information
```

---

# 🚨 41 — EMS Certificate Trust Problem

If EMS connection fails because the certificate is not trusted:

```bash
execute fctems verify <EMS_NAME>
```

Example:

```bash
execute fctems verify wind2016-ems
```

Training workaround:

```text
Export certificate as Base64
        ↓
Re-import certificate
        ↓
Retry EMS integration
```

---

# 🤫 42 — EMS Silent Approval in Security Fabric

Example configuration:

```bash
config endpoint-control fctems
    edit "ems1"
        set fortinetone-cloud-authentication disable
        set server <EMS_SERVER>
        set https-port 443
        set source-ip 0.0.0.0
        set pull-sysinfo enable
        set pull-vulnerabilities enable
        set pull-avatars enable
        set pull-tags enable
        set pull-malware-hash enable
        set capabilities fabric-auth silent-approval websocket push-ca-cert
        set call-timeout 30
        set websocket-override disable
    next
end
```

### Why `push-ca-cert`?

```text
Root / Fabric
      ↓
CA Certificate
      ↓
FortiClient / EMS
```

It helps distribute the required CA certificate.

---

# 🔍 43 — EMS Cluster Synchronization

Check certificate synchronization:

```bash
diagnose endpoint fctems json deep-inspect-cert-sync
```

Useful when:

```text
HA / Fabric
   +
EMS
   +
Certificate trust
```

do not behave consistently.

---

# ⚡ 44 — FortiClient ZTNA Tag Synchronization

ZTNA tag synchronization can use:

```text
REST API
TCP/8013

+

WebSockets
```

The goal:

```text
FortiClient
    ↓
Real-time attributes
    ↓
EMS / FortiGate
    ↓
ZTNA Tags
    ↓
Policy Enforcement
```

---

# 🔌 45 — FortiClient Network Access Daemon

Check WebSocket status:

```bash
diagnose test application fcnacd 2
```

Then verify synchronized dynamic information:

```bash
diagnose firewall dynamic list
```

Mental model:

```text
FortiClient
   ↓
WebSocket / REST
   ↓
FortiGate
   ↓
Dynamic Tag
   ↓
Policy
```

---

# 🧩 46 — FortiNAC

Check downstream FortiNAC information:

```bash
diagnose sys csf downstream-devices fortinac
```

Useful for:

```text
Device status
+
Downstream information
```

---

# 🔌 47 — FortiSwitch / FortiAP

Within Security Fabric:

```text
FortiSwitch
FortiAP
```

can behave as:

```text
Auto-Authentication / Extension Devices
```

The training notes indicate these do not require the same manual authorization workflow as ordinary downstream FortiGate devices.

From the Fabric, you can perform operations such as:

```text
FortiAP
 ├── Deauthorize
 └── Restart

FortiSwitch
 ├── CLI access
 ├── Firmware upgrade
 ├── Restart
 └── Deauthorize
```

---

# 🚫 48 — User Quarantine

Display quarantine information:

```bash
show user quarantine
```

Example quarantine target:

```bash
config firewall-groups x-quarantine
    config targets
        edit 1
            config macs
                edit "12:34:56:78:90:aa"
                next
            end
        next
    end
end
```

Concept:

```text
Compromised User / Endpoint
          ↓
       Quarantine
          ↓
Restricted Access
```

---

# 🏭 49 — Purdue Model + OT Security

The Purdue model provides a layered architecture for industrial environments.

Think:

```text
                 ENTERPRISE
                     │
                  Level 4
                     │
                  ┌──┴──┐
                  │ DMZ │
                  └──┬──┘
                     │
                  Level 3
                     │
              Manufacturing
                     │
                  Level 2
                     │
                SCADA / HMI
                     │
                  Level 1
                     │
                 PLC / Control
                     │
                  Level 0
                     │
            Physical Process
```

---

# 🏭 50 — Purdue Levels Quick Reference

| Level   | Main Function            | Examples                          |
| ------- | ------------------------ | --------------------------------- |
| **0**   | Physical Process         | Sensors, actuators, field devices |
| **1**   | Basic Control            | PLCs, controllers                 |
| **2**   | Supervisory Control      | SCADA, HMI                        |
| **3**   | Manufacturing Operations | MES, historians                   |
| **DMZ** | Security Buffer          | Firewalls, IPS, security systems  |
| **4**   | Enterprise               | Business applications, Internet   |

---

# 🛡️ 51 — Why Purdue Helps Security

### Defense in Depth

```text
Multiple layers
     ↓
Multiple security checkpoints
```

### Risk Mitigation

```text
Segmentation
     ↓
Reduced unauthorized access
     ↓
Reduced lateral movement
```

### Visibility

```text
Clear zones
     ↓
Better monitoring
     ↓
Better detection
```

### Compliance

```text
Purdue
   +
IEC 62443
   ↓
Structured OT security
```

---

# 🔥 52 — OT Attack Surface

OT environments must consider threats such as:

```text
APT
Malware
Ransomware
Network Attacks
Vulnerabilities
Encryption-related attacks
```

The security objective is not only:

```text
Confidentiality
```

but strongly emphasizes:

```text
Safety
+
Availability
+
Business Continuity
+
Security
```

---

# 🧱 53 — OT Security Technology Stack

```text
                OT SECURITY
                     │
      ┌──────────────┼──────────────┐
      │              │              │
      ▼              ▼              ▼
 Network          Endpoint       Identity
 Security         Security       & Access
      │              │              │
     NGFW           AV/EDR         MFA
     IPS            HIPS           RBAC
     VPN            App Control    PAM
      │
      ▼
 Segmentation
      │
      ▼
 Monitoring / SIEM
      │
      ▼
 SOAR / Threat Intelligence
```

---

# 🧩 54 — Purdue + Fortinet Security Fabric

FortiGate feature visibility:

```text
System
  ↓
Feature Visibility
  ↓
OT
```

Then inspect:

```text
Security Fabric
   ↓
Asset Identity Center
   ↓
OT View
```

Training model reference:

```text
FortiGate / FortiSwitch
        ↓
Level 2

Other OT devices
        ↓
Level 3

Special / external assets
        ↓
Other Purdue levels
```

---

# 🔄 55 — Change Purdue Level

Training CLI reference:

```bash
diagnose user-device-store device memory ot-prudue-set max-ip-level
```

Use this when adjusting the maximum Purdue/IP level classification.

---

# 🧠 56 — Security Fabric Device Database

The root FortiGate maintains information about:

```text
Devices
+
Users
```

Training references:

```text
user_info
user_info_history
```

Useful when investigating:

```text
Who?
Where?
Which device?
Which identity?
```

---

# 🔎 57 — Fabric Discovery & Status — Fast Commands

| Goal                  | Command                                        |
| --------------------- | ---------------------------------------------- |
| Fabric status         | `diagnose sys csf`                             |
| Pending authorization | `diagnose sys csf authorization pending-list`  |
| Downstream devices    | `diagnose sys csf downstream`                  |
| Upstream/root         | `diagnose sys csf upstream`                    |
| Downstream FortiNAC   | `diagnose sys csf downstream-devices fortinac` |
| Dynamic objects       | `diagnose firewall dynamic list`               |
| Authenticated users   | `diagnose firewall auth list`                  |
| Subscription/update   | `diagnose test update info`                    |

---

# 🧪 58 — Security Rating

Security Rating can be triggered manually instead of waiting for the scheduled execution.

### Disable scheduled security rating

```bash
config system global
    set security-rating-run-on-schedule disable
end
```

### Trigger manually

```bash
diagnose report-runner trigger
```

Mental model:

```text
Configuration
      ↓
Security Rating
      ↓
Report
      ↓
Find weaknesses
      ↓
Remediation
```

---

# 🔐 59 — SAML Authentication vs Authorization

Keep these concepts separate:

```text
Authentication
      ↓
"Who are you?"
```

```text
Authorization
      ↓
"What are you allowed to do?"
```

In the Fabric SAML workflow:

```text
SAML
 ↓
Identity
 ↓
Group
 ↓
Administrator Mapping
 ↓
Profile
 ↓
Access
```

---

# 🧭 60 — Security Fabric Troubleshooting Flow

When Fabric is not working, don't randomly debug everything.

Use this order:

```text
01. Interface
      ↓
02. IP connectivity
      ↓
03. Routing
      ↓
04. Firewall Policy
      ↓
05. Certificate / Trust
      ↓
06. Authorization
      ↓
07. Fabric Status
      ↓
08. Connector Status
      ↓
09. Process Debug
```

---

# 🧪 61 — Universal Debug Pattern

When debugging a FortiOS process:

```bash
diagnose debug enable
diagnose debug application <PROCESS> -1
```

Examples:

```bash
diagnose debug application autod -1
```

```bash
diagnose debug application kubed -1
```

```bash
diagnose debug application sepmd -1
```

After troubleshooting:

```bash
diagnose debug disable
```

> ⚠️ Do not leave verbose debugging enabled longer than necessary.

---

# 🧠 62 — The 10 Commands Worth Memorizing

If you only remember a small set:

```bash
diagnose sys csf
```

```bash
diagnose sys csf downstream
```

```bash
diagnose sys csf upstream
```

```bash
diagnose sys csf authorization pending-list
```

```bash
diagnose firewall dynamic list
```

```bash
diagnose firewall auth list
```

```bash
diagnose test update info
```

```bash
diagnose fdsm central-mgmt-status
```

```bash
diagnose report-runner trigger
```

```bash
diagnose automation test x-schedule
```

---

# ⚡ 63 — Fast Decision Tree

```text
Fabric problem?
      │
      ├── Can I reach the peer?
      │       └── NO → Interface / Route / Policy
      │
      ├── Is authorization pending?
      │       └── YES → Check Fabric authorization
      │
      ├── Is Fabric visible?
      │       └── NO → diagnose sys csf
      │
      ├── Is object/identity missing?
      │       └── Check synchronization / connector
      │
      ├── Is certificate involved?
      │       └── Check CA / trust / EMS / SAML
      │
      └── Still unclear?
              ↓
        Enable process-level debug
```

---

# 🧨 64 — High-Value Gotchas

> ### ⚠️ GOTCHA #1
>
> **Fabric synchronization ≠ every device using identical local configuration.**

```text
Default / Unified
       ≠
Local mode
```

---

> ### ⚠️ GOTCHA #2
>
> **VDOMs change the Fabric synchronization model.**

Always validate:

```text
Backup
NPU
IP addressing
Fabric behavior
```

---

> ### ⚠️ GOTCHA #3
>
> **SAML direction matters.**

```text
ROOT
 ↓
IDP

DOWNSTREAM
 ↓
SP
```

---

> ### ⚠️ GOTCHA #4
>
> **Threat Feed is not just a text file.**

```text
Feed
 ↓
Dynamic Object
 ↓
Policy
 ↓
Enforcement
```

---

> ### ⚠️ GOTCHA #5
>
> **Cloud troubleshooting often needs CLI.**

Do not assume the GUI exposes every useful verification point.

---

> ### ⚠️ GOTCHA #6
>
> **Automation debugging is process-oriented.**

```text
Automation
   ↓
autod
   ↓
diagnose debug application autod -1
```

---

> ### ⚠️ GOTCHA #7
>
> **OT security is not only about blocking traffic.**

Think:

```text
Safety
+
Availability
+
Segmentation
+
Visibility
+
Incident Response
```

---

# 📚 65 — One-Page Memory Map

```text
                         SECURITY FABRIC
                               │
        ┌──────────────────────┼──────────────────────┐
        │                      │                      │
        ▼                      ▼                      ▼
    AUTOMATION             CONNECTORS               SAML
        │                      │                      │
        │                      ├── Identity          ├── IDP
        │                      ├── Endpoint          └── SP
        │                      └── Threat Feed
        │
        ▼
      autod
                               │
        ┌──────────────────────┼──────────────────────┐
        │                      │                      │
        ▼                      ▼                      ▼
   FORTIMANAGER          FORTIANALYZER            EMS
        │                      │                      │
        ▼                      ▼                      ▼
   Central Mgmt             Logging              Tags / ZTNA
        │
        └──────────────────────┬──────────────────────┘
                               ▼
                              OT
                               │
                               ▼
                         PURDUE MODEL
                               │
          ┌────────────┬───────┼────────┬────────────┐
          ▼            ▼       ▼        ▼            ▼
       Level 0      Level 1  Level 2  Level 3     Level 4
       Physical      PLC      SCADA     MES       Enterprise
```

---

# 🏁 Final Rule

When troubleshooting **Security Fabric on FortiOS 7.2.0**, think in layers:

```text
CONNECTIVITY
     ↓
TRUST
     ↓
AUTHORIZATION
     ↓
SYNCHRONIZATION
     ↓
IDENTITY
     ↓
ENFORCEMENT
     ↓
DEBUG
```

> **Don't debug the symptom first. Find the layer that failed.**

---

## 📌 Version Note

This is intentionally aligned with the **FortiOS 7.2.0 training material** used for the associated course/lab environment.

```text
FortiOS
  └── 7.2.0
       ├── Security Fabric
       ├── Automation
       ├── External Connectors
       ├── SAML
       ├── Cloud Services
       ├── EMS
       ├── OT / Purdue
       └── Security Rating
```

> **Lab values are examples. Replace IP addresses, hostnames, usernames, certificates, and credentials with your own environment before deployment.**

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

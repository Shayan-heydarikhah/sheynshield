# FortiGate LENC, GUI CDN & FortiGate Cloud — CLI 

> **SheynShield | Engineering Secure Networks**
> FortiOS operational reference for **NSE4 / NSE7**, troubleshooting, licensing, encryption restrictions, GUI optimization, and FortiGate Cloud connectivity.

---

# 1. Low Encryption Models (LENC)

## What is LENC?

Some FortiGate models support a **Low Encryption (LENC)** license.

When an LENC license is applied, the FortiGate becomes a **Low Encryption Model** and is identified by the `-LENC` suffix.

Example:

```text
FG-100E-LENC
FG-60F-LENC
FG-400F-LENC
```

### Core Concept

```text
Normal FortiGate
      │
      │ Apply LENC license
      ▼
Low Encryption Model
      │
      ├── Restricted cryptographic algorithms
      ├── Restricted VPN encryption
      └── No SSL Inspection
```

---

# 2. LENC Cryptographic Restrictions

LENC models cannot use or inspect high-encryption protocols such as:

```text
3DES
AES
```

According to the supplied FortiOS notes, LENC models use:

```text
56-bit DES
```

for:

* SSL VPN
* IPsec VPN

and cannot perform:

```text
SSL Inspection
```

### Quick Comparison

| Feature        | LENC                  |
| -------------- | --------------------- |
| 3DES           | ❌ Not supported       |
| AES            | ❌ Not supported       |
| DES 56-bit     | ✅ Supported           |
| SSL VPN        | ⚠️ Limited encryption |
| IPsec VPN      | ⚠️ Limited encryption |
| SSL Inspection | ❌ Not supported       |

> ⚠️ **Exam Point:** Do not confuse an LENC license with a performance limitation. LENC is primarily a **cryptographic capability/restriction**.

---

# 3. LENC and BIOS Security Level

Applying an LENC license also affects the device's **BIOS and startup security settings**.

According to the notes:

```text
LENC applied
    ↓
FortiOS boots with BIOS security level 0
```

If a valid crypto license upgrades the appliance back to a **High Encryption Model**:

```text
Crypto license applied
    ↓
High Encryption Model
    ↓
BIOS security level 2
```

### Conceptual Flow

```text
High Encryption
      │
      │ LENC license
      ▼
Low Encryption
      │
      └── BIOS Security Level 0

Low Encryption
      │
      │ Crypto license
      ▼
High Encryption
      │
      └── BIOS Security Level 2
```

> 🔎 BIOS-level signatures and file-integrity mechanisms are part of the platform's boot/security architecture.

---

# 4. LENC Model Examples

The supplied model list includes:

```text
FG-60F-LENC
FG-Rugged-60F-LENC
FG-61F-LENC
FG-80F-LENC
FG-100F-LENC
FG-101F-LENC

FG-200E-LENC
FG-201E-LENC
FG-201F-LENC

FG-400E-BYPASS-LENC
FG-400F-LENC
FG-401E-LENC

FG-600E-LENC
FG-600F-LENC
FG-601E-LENC

FG-800D-LENC
FG-900D-LENC
FG-1000D-LENC

FG-1100E-LENC
FG-1101E-LENC

FG-1800F-LENC
FG-1801F-LENC

FG-2000E-LENC
```

> ⚠️ **Important:** LENC model availability is platform/region/license dependent. For a production deployment, verify the exact model and current Fortinet support matrix rather than assuming every model has an LENC variant.

---

# 5. Display the System Interface Tree

Use:

```bash
tree system interface
```

### Purpose

Displays the objects and values available under:

```text
system interface
```

Useful when exploring the FortiOS configuration tree and discovering available parameters.

---

# 6. GUI CDN Usage

FortiGate can load static GUI artifacts through a **Content Delivery Network (CDN)**.

Static GUI resources can include:

* JavaScript
* Images
* Other static GUI components

Instead of loading these resources directly from the FortiGate, the administrator can retrieve them from CDN infrastructure closer to the administrator.

---

## Why Use GUI CDN?

Without CDN:

```text
Administrator
      │
      │ High latency
      ▼
FortiGate
      │
      └── GUI static resources
```

With CDN:

```text
Administrator
      │
      │ Lower latency
      ▼
Fortinet CDN
      │
      └── Static GUI resources
```

### Benefit

Especially useful when:

```text
Administrator
      │
      │ Long-distance / high-latency connection
      ▼
FortiGate
```

For example:

```text
Administrator → USA
       │
       │ Internet
       │
       ▼
FortiGate → Remote region
```

Static GUI artifacts can be retrieved from a closer CDN location, improving GUI responsiveness.

---

# 7. Enable GUI CDN

```bash
config system global
    set gui-cdn-usage enable
end
```

### Important Behavior

The CDN is used **after a successful administrator login**.

If CDN access fails, the FortiGate can fall back to loading the required files from the FortiGate itself.

```text
Admin Login
    ↓
Authentication successful?
    │
    ├── No → CDN not used
    │
    └── Yes
         ↓
      CDN request
         │
         ├── Success → Load static artifacts from CDN
         │
         └── Failure → Fall back to FortiGate
```

> 🔐 **Security Note:** The CDN is intended for Fortinet GUI static artifacts; it does not mean administrative traffic or configuration data is being hosted on a public CDN.

---

# 8. FortiGate Cloud

FortiGate can communicate with **FortiGate Cloud / Fortinet cloud services** for services such as:

* Cloud management
* Licensing
* Updates
* Logging-related services
* FortiClient integration
* FortiAP / FortiSwitch information
* Firmware/image operations
* Configuration synchronization
* Reports

---

# 9. Register FortiGate with FortiGate Cloud

```bash
execute fortiguard-log login <account_id> <password>
```

Example:

```bash
execute fortiguard-log login <account_id> <password>
```

### Purpose

Establishes the connection/login required to register or associate the FortiGate with the cloud service.

> 🔐 **Security:** Never place real production credentials in shared notes, screenshots, scripts, or public Git repositories.

---

# 10. FDSM Diagnostics

`fdsm` diagnostics can be used to inspect different aspects of communication between the FortiGate and Fortinet cloud/FortiGuard services.

General syntax:

```bash
diagnose fdsm <operation>
```

---

## Contract Controller

```bash
diagnose fdsm contract-controller-update
```

### Purpose

Checks/updates contract-related information obtained through the cloud/FortiGuard infrastructure.

---

# 11. FortiGuard/Firmware Database Update

```bash
diagnose fdsm fds-update
```

Useful for checking update status related to services such as:

```text
IPS
AV
Malware protection
```

---

# 12. Log Controller

```bash
diagnose fdsm log-controller-update
```

### Purpose

Useful when troubleshooting cloud logging / log-controller communication.

Can help determine whether:

```text
FortiGate
   ↓
Cloud / Log Controller
   ↓
Logging destination
```

is functioning as expected.

---

# 13. GUI Message Updates

```bash
diagnose fdsm message-update
```

Used for checking/updating certain cloud-delivered messages and alerts displayed through the FortiGate GUI.

---

# 14. FortiClient / EMS Connectivity

```bash
diagnose fdsm forticlient-update
```

```bash
diagnose fdsm forticlient-net-info
```

> **Note:** The exact command spelling can vary by FortiOS release; verify the available command on the target version.

Useful when FortiGate is integrated with:

```text
FortiClient
FortiClient EMS
```

Potential troubleshooting areas include:

* Connectivity
* Synchronization
* Tags
* Endpoint/network information

---

# 15. Modem Information

```bash
diagnose fdsm modem-list
```

Useful when a FortiGate has a supported modem and you need to inspect modem-related information or failures.

---

# 16. Firmware/Image Information

### Image List

```bash
diagnose fdsm image-list
```

Useful for viewing available image information and version relationships.

### Download Image

```bash
diagnose fdsm image-download
```

Can be useful when GUI-based firmware/image operations are experiencing problems.

---

# 17. FortiClient Installer

```bash
diagnose fdsm fc-installer-download
```

Used to retrieve/download FortiClient installer information from Fortinet infrastructure.

---

# 18. SSL VPN Package

```bash
diagnose fdsm sslvpn-package-download
```

Useful for troubleshooting SSL VPN package retrieval.

---

# 19. FortiAP / FortiSwitch Updates

### FortiAP

```bash
diagnose fdsm fortiap-latest-ver
diagnose fdsm fortiap-download
```

### FortiSwitch

```bash
diagnose fdsm fortisw-latest-ver
diagnose fdsm fortisw-download
```

These commands can help troubleshoot:

```text
FortiGate
   │
   ├── FortiAP
   │
   └── FortiSwitch
```

software/version information and downloads.

---

# 20. Central Management Status

```bash
diagnose fdsm central-mgmt-status
```

Useful for checking the state of centralized management communication.

---

# 21. FortiManager Auto-Discovery

```bash
diagnose fdsm fmg-auto-discovery-status
```

Useful when troubleshooting FortiManager discovery and management connectivity.

---

# 22. FortiAP / FortiSwitch Contract Download

```bash
diagnose fdsm fap-fsw-contract-download
```

Used to inspect/download contract-related information for:

```text
FortiAP
FortiSwitch
```

---

# 23. Reports

### List Reports

```bash
diagnose fdsm report-list
```

### Download Report

```bash
diagnose fdsm report-download
```

Useful when investigating cloud-generated reports.

---

# 24. Configuration Operations

FDSM also provides configuration-related operations.

### List Configuration

```bash
diagnose fdsm cfg-list
```

### Upload Configuration

```bash
diagnose fdsm cfg-upload
```

### Download Configuration

```bash
diagnose fdsm cfg-download
```

### Configuration Difference

```bash
diagnose fdsm cfg-diff
```

### Concept

```text
FortiGate
    │
    │ Configuration information
    ▼
Fortinet Cloud Infrastructure
    │
    ├── Store / retrieve
    ├── Compare
    └── Support management workflows
```

> ⚠️ Treat cloud configuration operations carefully in production environments. Always understand whether an operation is **read-only, upload, download, or synchronization-related** before executing it.

---

# 25. FortiToken Activation

```bash
diagnose fdsm ftk-activate
```

Used for FortiToken-related activation workflows.

> **Note:** Verify the exact command available on your FortiOS release because diagnostic command names can change between releases.

---

# 26. FortiGate Cloud Troubleshooting Mindset

When FortiGate Cloud-related functions fail, don't immediately assume that the FortiGate itself is the problem.

Think in layers:

```text
                ┌──────────────────────┐
                │    FortiGate Cloud   │
                └──────────▲───────────┘
                           │
                    HTTPS / Services
                           │
                ┌──────────┴───────────┐
                │     FortiGuard/FDS   │
                └──────────▲───────────┘
                           │
                     Internet Path
                           │
                ┌──────────┴───────────┐
                │       FortiGate      │
                └──────────────────────┘
```

Check:

```text
1. Internet connectivity
2. DNS resolution
3. Routing
4. Firewall policies
5. Proxy requirements
6. FortiGuard connectivity
7. License/contract status
8. FortiGate Cloud registration
9. Service-specific diagnostics
10. System event logs
```

---

# 27. High-Value Diagnostic Matrix

| Requirement              | Diagnostic                                 |
| ------------------------ | ------------------------------------------ |
| Contract information     | `diagnose fdsm contract-controller-update` |
| FortiGuard/FDS updates   | `diagnose fdsm fds-update`                 |
| Cloud logging            | `diagnose fdsm log-controller-update`      |
| GUI/cloud messages       | `diagnose fdsm message-update`             |
| FortiClient              | `diagnose fdsm forticlient-update`         |
| FortiClient network info | `diagnose fdsm forticlient-net-info`       |
| Modem                    | `diagnose fdsm modem-list`                 |
| Firmware images          | `diagnose fdsm image-list`                 |
| Firmware download        | `diagnose fdsm image-download`             |
| FortiClient installer    | `diagnose fdsm fc-installer-download`      |
| SSL VPN package          | `diagnose fdsm sslvpn-package-download`    |
| FortiAP version          | `diagnose fdsm fortiap-latest-ver`         |
| FortiAP download         | `diagnose fdsm fortiap-download`           |
| FortiSwitch version      | `diagnose fdsm fortisw-latest-ver`         |
| FortiSwitch download     | `diagnose fdsm fortisw-download`           |
| Central management       | `diagnose fdsm central-mgmt-status`        |
| FMG discovery            | `diagnose fdsm fmg-auto-discovery-status`  |
| AP/Switch contracts      | `diagnose fdsm fap-fsw-contract-download`  |
| Reports                  | `diagnose fdsm report-list`                |
| Report download          | `diagnose fdsm report-download`            |
| Configuration list       | `diagnose fdsm cfg-list`                   |
| Configuration upload     | `diagnose fdsm cfg-upload`                 |
| Configuration download   | `diagnose fdsm cfg-download`               |
| Configuration diff       | `diagnose fdsm cfg-diff`                   |
| FortiToken               | `diagnose fdsm ftk-activate`               |

---

# 28. NSE4 / NSE7 Fast Recall

```text
LENC
│
├── Low Encryption Model
├── Model suffix → -LENC
├── Restricted cryptography
├── DES 56-bit
├── No AES / 3DES
├── SSL VPN → limited encryption
├── IPsec VPN → limited encryption
└── SSL Inspection → NOT supported
```

```text
LENC BIOS
│
├── LENC applied
│      ↓
│   BIOS Security Level 0
│
└── Crypto license
       ↓
    High Encryption
       ↓
    BIOS Security Level 2
```

```text
GUI CDN
│
├── Static GUI artifacts
├── Lower latency for remote administrators
├── Used after successful admin login
├── CDN failure → FortiGate fallback
└── config system global
       set gui-cdn-usage enable
```

```text
FortiGate Cloud
│
├── Registration
│
├── Contract
├── FDS / AV / IPS updates
├── Logging
├── FortiClient / EMS
├── Firmware
├── FortiAP
├── FortiSwitch
├── Reports
├── Configuration
└── FortiToken
```

---

# 29. ⚠️ Exam Traps

### Trap #1 — LENC ≠ Normal Encryption

```text
LENC
≠
High Encryption
```

LENC specifically restricts cryptographic capabilities.

---

### Trap #2 — LENC Does Not Encrypt the Disk

The LENC concept should not be confused with full disk encryption.

---

### Trap #3 — GUI CDN Is Not Cloud Management

```text
GUI CDN
→ Static GUI resources / performance

FortiGate Cloud
→ Cloud services / management / logging / updates / other services
```

They are separate concepts.

---

### Trap #4 — CDN Does Not Mean Admin Authentication Happens in the CDN

The administrator first authenticates to the FortiGate. CDN usage applies to eligible static GUI artifacts afterward.

---

### Trap #5 — `fds-update` Is Not a Generic "Everything Is Working" Test

A successful FDS update check does not automatically prove that:

```text
FortiGate Cloud
FortiManager
FortiClient EMS
Logging
FortiAP
FortiSwitch
```

are all functioning.

Use the **service-specific diagnostic**.

---

# SheynShield One-Minute Revision

```text
LENC
────────────────────────
-LENC
Low Encryption
DES 56-bit
No AES
No 3DES
No SSL Inspection
BIOS Security Level → 0

Crypto license
High Encryption
BIOS Security Level → 2


GUI CDN
────────────────────────
config system global
    set gui-cdn-usage enable
end

Purpose:
Static GUI artifacts
Lower latency
Remote administration
Fallback → FortiGate


FORTIGATE CLOUD
────────────────────────
execute fortiguard-log login <account> <password>

diagnose fdsm contract-controller-update
diagnose fdsm fds-update
diagnose fdsm log-controller-update
diagnose fdsm message-update

diagnose fdsm image-list
diagnose fdsm image-download

diagnose fdsm central-mgmt-status
diagnose fdsm fmg-auto-discovery-status

diagnose fdsm cfg-list
diagnose fdsm cfg-upload
diagnose fdsm cfg-download
diagnose fdsm cfg-diff

diagnose fdsm report-list
diagnose fdsm report-download
```

> **Golden Rule:** For FortiGate Cloud troubleshooting, identify the **specific service first**, then use its corresponding `diagnose fdsm` command instead of treating cloud connectivity as one single service.

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

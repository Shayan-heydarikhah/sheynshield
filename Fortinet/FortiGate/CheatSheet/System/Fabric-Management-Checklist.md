# 🔥 FortiGate Fabric Management & Firmware Upgrade Checklist

> **FortiOS 7.2.0 | Security Fabric | Fabric Management | Federated Upgrade | Firmware Upgrade | TFTP | USB | Controlled Upgrade | Firmware Verification | Recovery**
>
> **SheynShield | Engineering Secure Networks**

---

## 📌 Table of Contents

- [1. Fabric Management Checklist](#1-fabric-management-checklist)
- [2. Pre-Upgrade Checklist](#2-pre-upgrade-checklist)
- [3. Firmware Maturity Checklist](#3-firmware-maturity-checklist)
- [4. Upgrade Path Checklist](#4-upgrade-path-checklist)
- [5. Configuration Backup Checklist](#5-configuration-backup-checklist)
- [6. FortiGuard & Security Definitions](#6-fortiguard--security-definitions)
- [7. Fabric Upgrade Checklist](#7-fabric-upgrade-checklist)
- [8. Auto-Update Tunnel Checklist](#8-auto-update-tunnel-checklist)
- [9. Federated Upgrade Checklist](#9-federated-upgrade-checklist)
- [10. TFTP Firmware Upgrade Checklist](#10-tftp-firmware-upgrade-checklist)
- [11. Controlled Upgrade Checklist](#11-controlled-upgrade-checklist)
- [12. Firmware Signature Verification](#12-firmware-signature-verification)
- [13. Boot Menu Firmware Verification](#13-boot-menu-firmware-verification)
- [14. USB Firmware Upgrade](#14-usb-firmware-upgrade)
- [15. USB Auto-Detection](#15-usb-auto-detection)
- [16. Firmware Downgrade Checklist](#16-firmware-downgrade-checklist)
- [17. Post-Upgrade Validation](#17-post-upgrade-validation)
- [18. Production Golden Checklist](#18-production-golden-checklist)
- [19. Troubleshooting Checklist](#19-troubleshooting-checklist)
- [20. High-Value CLI Reference](#20-high-value-cli-reference)
- [21. NSE Exam Traps](#21-nse-exam-traps)
- [22. Expert Operational Rules](#22-expert-operational-rules)
- [23. One-Minute NSE Revision](#23-one-minute-nse-revision)
- [24. SheynShield Security Rule](#24-sheynshield-security-rule)

---

# 1. Fabric Management Checklist

## 🎯 Purpose

FortiGate **Fabric Management** provides centralized firmware and device management for devices participating in the Security Fabric.

### Managed Device Checklist

- [ ] FortiGate devices identified.
- [ ] FortiAP devices identified.
- [ ] FortiSwitch devices identified.
- [ ] Fabric topology verified.
- [ ] Fabric root FortiGate identified.
- [ ] Fabric members authorized.
- [ ] Device registration status verified.
- [ ] Device operational status verified.
- [ ] Current firmware versions documented.
- [ ] Target firmware version documented.
- [ ] Upgrade status reviewed.
- [ ] Firmware maturity reviewed.

### GUI Location

```text
System
└── Fabric Management
````

### Fabric Management Can Provide

* [ ] Device authorization.
* [ ] Device registration.
* [ ] Device monitoring.
* [ ] Firmware version visibility.
* [ ] Firmware maturity visibility.
* [ ] Upgrade management.
* [ ] Upgrade scheduling.
* [ ] Multi-device upgrade.
* [ ] Fabric upgrade status.

---

# 2. Pre-Upgrade Checklist

> [!CAUTION]
> **Never start a production firmware upgrade without a rollback/recovery plan.**

## 🔍 Environment Validation

* [ ] Current FortiOS version documented.
* [ ] Current build number documented.
* [ ] Hardware model verified.
* [ ] HA topology documented.
* [ ] Security Fabric topology documented.
* [ ] Managed FortiAP devices documented.
* [ ] Managed FortiSwitch devices documented.
* [ ] Critical services documented.
* [ ] Critical VPNs documented.
* [ ] Routing dependencies documented.
* [ ] External dependencies documented.

## 🌐 Connectivity

* [ ] FortiGuard connectivity verified.
* [ ] DNS resolution verified.
* [ ] Internet connectivity verified where required.
* [ ] Routing verified.
* [ ] Management connectivity verified.
* [ ] Out-of-band management available where possible.
* [ ] Console access available.

## ⏱️ Time Synchronization

* [ ] NTP configured.
* [ ] NTP synchronization verified.
* [ ] System time verified.
* [ ] Time zone verified.

## 🔐 Licensing

* [ ] FortiCare status verified.
* [ ] FortiGuard services verified.
* [ ] Required licenses verified.
* [ ] Security service subscriptions verified.

## 💾 Backup

* [ ] Configuration backup created.
* [ ] Backup stored outside the device.
* [ ] Backup tested/reviewed.
* [ ] Recovery configuration available.
* [ ] Previous firmware image available.

## 📦 Firmware

* [ ] Target firmware downloaded.
* [ ] Correct hardware image selected.
* [ ] Firmware source verified.
* [ ] Firmware authenticity verified.
* [ ] Firmware integrity verified.
* [ ] Upgrade path verified.
* [ ] Firmware maturity reviewed.
* [ ] Feature/Mature classification reviewed.

## 🧪 Testing

* [ ] Firmware tested in lab where possible.
* [ ] Pilot device identified.
* [ ] Low-risk devices identified.
* [ ] Production devices grouped by risk.
* [ ] Critical devices scheduled last where possible.

## 🕐 Maintenance

* [ ] Maintenance window approved.
* [ ] Expected reboot time estimated.
* [ ] Stakeholders notified.
* [ ] Rollback procedure documented.
* [ ] Recovery equipment available.
* [ ] TFTP server prepared if required.
* [ ] USB recovery media prepared if required.

---

# 3. Firmware Maturity Checklist

Starting with **FortiOS 7.2.0**, FortiOS firmware images use maturity tags. ([Fortinet Documentation][2])

## Feature Firmware

* [ ] Feature tag identified.
* [ ] New functionality requirements reviewed.
* [ ] Production risk evaluated.
* [ ] Feature release warning reviewed.

## Mature Firmware

* [ ] Mature tag identified.
* [ ] Release contains no new major features.
* [ ] Bug fixes reviewed.
* [ ] Vulnerability patches reviewed.
* [ ] Production suitability evaluated.

### GUI Indicators

```text
Feature → Gray
Mature  → Green
```

### CLI Verification

```bash
get system status
```

Look for:

```text
GA.F → Feature
GA.M → Mature
```

### Maturity Checklist

* [ ] Current firmware maturity identified.
* [ ] Target firmware maturity identified.
* [ ] Mature → Feature transition evaluated.
* [ ] Feature firmware warning reviewed.

> [!IMPORTANT]
> **Feature vs Mature is a release maturity classification, not simply "new vs old".**

---

# 4. Upgrade Path Checklist

## 🔎 Verify Upgrade Path

* [ ] Current FortiOS version identified.
* [ ] Target FortiOS version identified.
* [ ] Official upgrade path checked.
* [ ] Intermediate versions identified.
* [ ] Direct upgrade availability verified.
* [ ] Hardware compatibility verified.
* [ ] Configuration migration impact reviewed.
* [ ] Feature changes reviewed.
* [ ] HA dependencies reviewed.
* [ ] Fabric dependencies reviewed.

### Upgrade Decision

```text
Current Version
      │
      ▼
Target Version
      │
      ▼
Supported Upgrade Path?
      │
   ┌──┴──┐
  YES    NO
   │      │
   ▼      ▼
Proceed  STOP
         Review
         Intermediate
         Versions
```

> [!WARNING]
> **Do not assume that every FortiOS release can be upgraded directly to every newer release.**

### Fabric Upgrade

When the upgrade path contains multiple builds, FortiGate can automatically follow the path during a federated update. ([Fortinet Documentation][3])

* [ ] Follow upgrade path option reviewed.
* [ ] Intermediate versions reviewed.
* [ ] Multiple reboot requirement understood.
* [ ] Direct upgrade risk evaluated.

---

# 5. Configuration Backup Checklist

## 💾 Backup Before Upgrade

* [ ] Local configuration backup created.
* [ ] External backup created.
* [ ] FortiManager backup verified if applicable.
* [ ] Backup timestamp recorded.
* [ ] Backup location documented.
* [ ] Recovery configuration available.
* [ ] Configuration changes frozen before upgrade.

### Fabric Scheduled Upgrade

For custom/scheduled Fabric upgrades:

* [ ] Configuration backup timing understood.
* [ ] Pending configuration changes reviewed.
* [ ] Final configuration backup performed if necessary.
* [ ] No uncommitted production changes remain.

> [!CAUTION]
> In a custom scheduled upgrade, the configuration backup is created when the upgrade is scheduled. Configuration changes made afterward may not be included in that scheduled backup. ([Fortinet Documentation][4])

---

# 6. FortiGuard & Security Definitions

## 🌐 FortiGuard Checklist

* [ ] FortiGuard connectivity verified.
* [ ] DNS resolution verified.
* [ ] Routing verified.
* [ ] Licensing verified.
* [ ] Auto-update functionality verified.

## 🔄 Post-Upgrade Security Updates

After firmware installation:

```bash
execute update-now
```

### Verify

* [ ] AV definitions updated.
* [ ] Attack definitions updated.
* [ ] FortiGuard connectivity verified.
* [ ] Security services operational.
* [ ] System processes healthy.

### Recommended Flow

```text
Firmware Upgrade
      │
      ▼
Reboot
      │
      ▼
get system status
      │
      ▼
execute update-now
      │
      ▼
Verify Definitions
      │
      ▼
Verify Services
      │
      ▼
Validate Traffic
```

> [!IMPORTANT]
> **A successful firmware installation does not automatically mean the complete system is healthy.**

---

# 7. Fabric Upgrade Checklist

## GUI

```text
System
└── Fabric Management
    └── Fabric Upgrade
```

### Checklist

* [ ] Logged into correct FortiGate.
* [ ] Root FortiGate identified.
* [ ] Fabric topology verified.
* [ ] Target firmware selected.
* [ ] Latest firmware reviewed.
* [ ] All available upgrades reviewed.
* [ ] Maturity level reviewed.
* [ ] Upgrade path reviewed.
* [ ] Immediate vs Custom schedule selected.
* [ ] Configuration backup confirmed.
* [ ] Upgrade schedule reviewed.
* [ ] Device list reviewed.
* [ ] Upgrade started during approved window.

Fortinet documents Fabric Upgrade as the centralized mechanism for upgrading the root/Fabric devices and managed devices. ([Fortinet Documentation][4])

---

# 8. Auto-Update Tunnel Checklist

Before Fabric firmware-management workflows:

* [ ] FortiGuard Auto-Update Tunnel requirements reviewed.
* [ ] FortiGuard connectivity verified.
* [ ] DNS verified.
* [ ] Routing verified.
* [ ] NTP verified.
* [ ] Licensing verified.
* [ ] Fabric connectivity verified.

### Concept

```text
Fabric Device
      │
      ▼
FortiGate / Fabric Root
      │
      ▼
FortiGuard
      │
      ▼
Firmware / Security Services
```

---

# 9. Federated Upgrade Checklist

## 🎯 Purpose

A federated upgrade can coordinate firmware upgrades across Fabric devices and can follow a multi-build upgrade path where supported. ([Fortinet Documentation][3])

### CLI

```bash
execute federated-upgrade initialize
```

### Status

```bash
execute federated-upgrade status
```

### Cancel

```bash
execute federated-upgrade cancel
```

### Restart

```bash
execute federated-upgrade restart
```

> [!NOTE]
> `restart` is available in later 7.2.x documentation. Always verify the exact CLI options against the FortiOS build being used.

## Federated Upgrade Checklist

* [ ] Fabric topology verified.
* [ ] Root FortiGate identified.
* [ ] Target version selected.
* [ ] Upgrade path reviewed.
* [ ] Intermediate versions reviewed.
* [ ] Upgrade schedule selected.
* [ ] Configuration backup confirmed.
* [ ] Fabric devices reviewed.
* [ ] FortiGate upgrade sequence reviewed.
* [ ] FortiAP behavior reviewed.
* [ ] FortiSwitch behavior reviewed.
* [ ] Multiple reboot requirement understood.

> [!IMPORTANT]
> In FortiOS 7.2.0, FortiAP and FortiSwitch devices cannot follow the same multi-build upgrade path as FortiGate devices; they upgrade directly to the target version. ([Fortinet Documentation][3])

---

# 10. TFTP Firmware Upgrade Checklist

## 🖥️ TFTP Preparation

* [ ] TFTP server installed.
* [ ] Firmware copied to TFTP root.
* [ ] Correct firmware filename confirmed.
* [ ] Correct hardware image confirmed.
* [ ] TFTP server IP confirmed.
* [ ] Network connectivity confirmed.
* [ ] Firewall rules verified.

Example:

```text
TFTP Root
└── a.pkg
```

## 🔌 Connectivity

```bash
execute ping 192.168.20.200
```

* [ ] Ping successful.
* [ ] TFTP server reachable.
* [ ] Firmware file available.

## 🚀 Upgrade

```bash
execute restore image tftp a.pkg 192.168.20.200
```

### Confirm

```text
This operation will replace the current firmware version!
Do you want to continue? (y/n)
```

* [ ] Upgrade warning reviewed.
* [ ] Firmware source verified.
* [ ] Correct image confirmed.
* [ ] Upgrade approved.
* [ ] Confirmation entered.

## 🔐 Firmware Validation

FortiGate performs operations including:

* [ ] Firmware download.
* [ ] Signature verification.
* [ ] Firmware integrity verification.
* [ ] Firmware installation.
* [ ] Reboot.

### Flow

```text
Ping
 ↓
TFTP
 ↓
Download
 ↓
Verify Signature
 ↓
Verify Integrity
 ↓
Install
 ↓
Reboot
 ↓
New FortiOS
```

---

# 11. Controlled Upgrade Checklist

Controlled upgrades allow a firmware image to be staged in a separate partition before activation. ([Fortinet Documentation][1])

## 📦 Stage Firmware

```bash
execute restore secondary-image tftp a.pkg
```

Possible sources:

```text
FTP
TFTP
USB
```

## 🔀 Select Next Boot Partition

```bash
execute set-next-reboot secondary
```

> [!IMPORTANT]
> `secondary` must be used when the newly staged firmware is located in the secondary partition. The `primary`/`secondary` value refers to the partition containing the firmware that should be loaded on the next reboot. ([Fortinet Documentation][1])

## Controlled Upgrade Checklist

* [ ] Firmware image prepared.
* [ ] Firmware image verified.
* [ ] Secondary partition available.
* [ ] Firmware staged.
* [ ] Staging completed successfully.
* [ ] Correct partition identified.
* [ ] `set-next-reboot` configured.
* [ ] Maintenance window approved.
* [ ] Reboot scheduled.
* [ ] Post-reboot validation planned.

### Flow

```text
Current Firmware
      │
      ▼
Secondary Partition
      │
      ├── Load New Firmware
      │
      ▼
Verify
      │
      ▼
set-next-reboot secondary
      │
      ▼
Maintenance Window
      │
      ▼
Reboot
      │
      ▼
New Firmware
```

### Production Use Cases

* [ ] Staged upgrades.
* [ ] Maintenance-window optimization.
* [ ] Multiple FortiGate deployment.
* [ ] Script-based deployment.
* [ ] FortiManager-controlled workflow.

---

# 12. Firmware Signature Verification

## 🔐 Firmware Authenticity Checklist

* [ ] Firmware downloaded from trusted Fortinet source.
* [ ] Correct hardware model verified.
* [ ] Firmware build verified.
* [ ] Signature verification performed.
* [ ] Integrity verification performed.
* [ ] Unexpected signature warning investigated.

### Verification Model

```text
Firmware Image
      │
      ▼
Digital Signature
      │
      ▼
FortiGate Verification
      │
   ┌──┴──┐
  PASS   FAIL
   │      │
   ▼      ▼
Continue STOP
          Investigate
```

> [!CAUTION]
> **Never ignore firmware authenticity warnings in production.**

### If Signature Warning Appears

* [ ] Stop upgrade if unexpected.
* [ ] Verify firmware source.
* [ ] Verify file.
* [ ] Verify model compatibility.
* [ ] Verify build.
* [ ] Re-download firmware if necessary.
* [ ] Investigate before proceeding.

---

# 13. Boot Menu Firmware Verification

## 🎯 Purpose

Use console-based boot testing when you need to verify a firmware image without immediately making it the permanent default firmware.

### Required

* [ ] Console cable.
* [ ] TFTP server.
* [ ] Firmware image.
* [ ] Local management information.
* [ ] TFTP server IP.
* [ ] Firmware filename.

## Reboot

```bash
execute reboot
```

Confirm:

```text
y
```

## Interrupt Boot

Watch for:

```text
Press any key to display configuration menu ..
```

* [ ] Press key during boot window.
* [ ] Enter boot menu.
* [ ] Avoid missing the short interruption window.

### Boot Flow

```text
Boot
 │
 ▼
Press Any Key
 │
 ├── YES → Boot Menu
 │
 └── NO  → Normal Boot
```

---

# 14. Boot Menu Checklist

Typical options:

| Key | Function                                |
| --- | --------------------------------------- |
| `g` | Get firmware image from TFTP            |
| `f` | Format boot device                      |
| `b` | Boot backup firmware and set as default |
| `c` | Configuration/information               |
| `q` | Continue normal boot                    |
| `h` | Help                                    |

## Safe Verification Workflow

* [ ] Enter boot menu.
* [ ] Select `g`.
* [ ] Enter TFTP server address.
* [ ] Enter local FortiGate address.
* [ ] Enter firmware filename.
* [ ] Download image.
* [ ] Verify image.
* [ ] Select run-without-saving option where supported.
* [ ] Test firmware.
* [ ] Validate basic functionality.

> [!DANGER]
> `f` can format the boot device. **Never select it casually in production.**

---

# 15. Boot Menu TFTP Configuration Checklist

Select:

```text
c
```

Potential configuration options:

| Key | Function                |
| --- | ----------------------- |
| `p` | Firmware download port  |
| `d` | DHCP mode               |
| `i` | Local IP address        |
| `s` | Local subnet mask       |
| `g` | Local gateway           |
| `v` | VLAN ID                 |
| `t` | TFTP server             |
| `f` | Firmware filename       |
| `e` | Reset TFTP parameters   |
| `r` | Review parameters       |
| `n` | Network diagnostic/ping |
| `q` | Quit                    |
| `h` | Help                    |

## Checklist

* [ ] Download port verified.
* [ ] Local IP configured.
* [ ] Subnet mask configured.
* [ ] Gateway configured if required.
* [ ] VLAN configured if required.
* [ ] TFTP server configured.
* [ ] Firmware filename configured.
* [ ] Parameters reviewed.
* [ ] Network test completed.

---

# 16. USB Firmware Upgrade

## 💾 Preparation

* [ ] USB drive available.
* [ ] Firmware image copied to USB root.
* [ ] Correct filename verified.
* [ ] Correct hardware image verified.
* [ ] USB connected to FortiGate.

## Restore

```bash
execute restore image usb <filename>
```

Example:

```bash
execute restore image usb a.pkg
```

## Confirmation

```text
This operation will replace the current firmware version!
Do you want to continue? (y/n)
```

* [ ] Warning reviewed.
* [ ] Correct firmware confirmed.
* [ ] Upgrade approved.
* [ ] Reboot expected.

## Post-USB Upgrade

```bash
get system status
```

```bash
execute update-now
```

### Validate

* [ ] FortiOS version.
* [ ] Build number.
* [ ] Device model.
* [ ] System status.
* [ ] FortiGuard connectivity.
* [ ] AV definitions.
* [ ] Attack definitions.
* [ ] Critical services.

---

# 17. USB Auto-Detection Checklist

GUI location:

```text
System
└── Settings
    └── Startup Settings
```

### Checklist

* [ ] Detect Firmware requirement identified.
* [ ] Firmware filename configured.
* [ ] Firmware copied to USB root.
* [ ] USB connected.
* [ ] Startup behavior understood.
* [ ] Upgrade performed if required.
* [ ] Auto-detection disabled after temporary use if no longer required.

> [!WARNING]
> Avoid leaving automatic firmware detection enabled unnecessarily after maintenance or testing.

---

# 18. Firmware Downgrade Checklist

> [!CAUTION]
> **Firmware downgrade is a high-risk operation.**

## Before Downgrade

* [ ] Operational reason documented.
* [ ] Downgrade path verified.
* [ ] Target firmware verified.
* [ ] Hardware compatibility verified.
* [ ] Configuration backup created.
* [ ] Recovery plan prepared.
* [ ] Previous firmware available.
* [ ] Console access available.
* [ ] Maintenance window approved.

## TFTP Downgrade

```bash
execute ping 192.168.20.200
```

Then:

```bash
execute restore image tftp a.pkg 192.168.20.200
```

### Downgrade Warning

FortiGate may display:

```text
This operation will downgrade the current firmware version!
```

* [ ] Downgrade warning reviewed.
* [ ] Configuration impact understood.
* [ ] Confirmation approved.

## After Downgrade

```bash
get system status
```

```bash
execute update-now
```

### Validate

* [ ] Firmware version.
* [ ] Build.
* [ ] Configuration.
* [ ] Interfaces.
* [ ] Routing.
* [ ] Firewall policies.
* [ ] VPN.
* [ ] HA.
* [ ] Security Fabric.
* [ ] FortiGuard.
* [ ] Critical services.

---

# 19. Post-Upgrade Validation Checklist

## 🖥️ System

```bash
get system status
```

* [ ] Correct FortiOS version.
* [ ] Correct build.
* [ ] Correct hardware model.
* [ ] System healthy.

## 🌐 Network

* [ ] Interfaces operational.
* [ ] VLANs operational.
* [ ] Routing verified.
* [ ] Default route verified.
* [ ] DNS verified.
* [ ] Internet connectivity verified.

## 🔥 Security

* [ ] Firewall policies verified.
* [ ] Security profiles verified.
* [ ] IPS operational.
* [ ] Antivirus operational.
* [ ] Web filtering operational.
* [ ] Application control operational.

## 🔐 VPN

* [ ] IPsec tunnels verified.
* [ ] SSL VPN verified if used.
* [ ] Authentication verified.
* [ ] VPN traffic tested.

## 🛡️ HA

* [ ] HA status verified.
* [ ] Primary/secondary roles verified.
* [ ] Synchronization verified.
* [ ] Session synchronization verified where applicable.
* [ ] HA monitoring verified.

## 🌐 Security Fabric

* [ ] Fabric root healthy.
* [ ] Fabric members connected.
* [ ] FortiAP status verified.
* [ ] FortiSwitch status verified.
* [ ] Fabric topology healthy.

## 📡 FortiGuard

* [ ] FortiGuard connectivity verified.
* [ ] AV definitions updated.
* [ ] Attack definitions updated.
* [ ] License status verified.

## 📊 Services

* [ ] Critical services verified.
* [ ] Logs reviewed.
* [ ] Authentication tested.
* [ ] Monitoring restored.
* [ ] Alerts reviewed.

## 🚦 Traffic

* [ ] Internal traffic tested.
* [ ] Internet traffic tested.
* [ ] Critical applications tested.
* [ ] NAT verified.
* [ ] VIPs verified.
* [ ] Routing verified.
* [ ] VPN traffic verified.

---

# 20. Production Golden Checklist

## 🔴 BEFORE UPGRADE

* [ ] Backup configuration.
* [ ] Verify upgrade path.
* [ ] Verify target firmware.
* [ ] Verify hardware compatibility.
* [ ] Verify firmware authenticity.
* [ ] Verify firmware integrity.
* [ ] Verify FortiGuard.
* [ ] Verify licenses.
* [ ] Verify NTP.
* [ ] Verify HA.
* [ ] Verify Fabric.
* [ ] Verify VPN.
* [ ] Verify routing.
* [ ] Verify critical services.
* [ ] Prepare rollback plan.
* [ ] Prepare console access.
* [ ] Prepare TFTP/USB recovery.
* [ ] Approve maintenance window.

## 🟡 DURING UPGRADE

* [ ] Monitor firmware transfer.
* [ ] Monitor signature verification.
* [ ] Monitor integrity verification.
* [ ] Monitor reboot.
* [ ] Do not interrupt power.
* [ ] Do not interrupt firmware transfer.
* [ ] Monitor HA state.
* [ ] Monitor Fabric state.
* [ ] Monitor management connectivity.

## 🟢 AFTER UPGRADE

* [ ] `get system status`
* [ ] `execute update-now`
* [ ] Verify AV definitions.
* [ ] Verify attack definitions.
* [ ] Verify interfaces.
* [ ] Verify routing.
* [ ] Verify firewall policies.
* [ ] Verify NAT.
* [ ] Verify VPNs.
* [ ] Verify HA.
* [ ] Verify Fabric.
* [ ] Verify FortiGuard.
* [ ] Verify logs.
* [ ] Verify monitoring.
* [ ] Test critical applications.
* [ ] Confirm production traffic.
* [ ] Document final state.

---

# 21. Troubleshooting Checklist

## Firmware Upgrade Failed

```text
Firmware Upgrade Failed
        │
        ▼
Can FortiGate Boot?
   ┌────┴────┐
  YES       NO
   │         │
   ▼         ▼
Check OS   Console
   │         │
   │         ▼
   │      Boot Menu
   │         │
   │         ▼
   │      TFTP / USB
   │         │
   └────┬────┘
        ▼
Verify Firmware
        │
        ▼
Verify Signature
        │
        ▼
Verify Integrity
        │
        ▼
Verify Upgrade Path
        │
        ▼
Recover / Retry
```

## If FortiGate Boots

* [ ] Run `get system status`.
* [ ] Verify firmware version.
* [ ] Verify build.
* [ ] Verify interfaces.
* [ ] Verify routing.
* [ ] Verify services.
* [ ] Verify FortiGuard.
* [ ] Verify security definitions.
* [ ] Review logs.

## If FortiGate Does Not Boot

* [ ] Connect console.
* [ ] Interrupt boot.
* [ ] Enter boot menu.
* [ ] Verify TFTP connectivity.
* [ ] Verify firmware image.
* [ ] Verify image filename.
* [ ] Verify boot partition.
* [ ] Use recovery workflow.
* [ ] Avoid destructive formatting unless explicitly required.

---

# 22. High-Value CLI Reference

| Purpose                      | Command                                       |
| ---------------------------- | --------------------------------------------- |
| System status                | `get system status`                           |
| Test connectivity            | `execute ping <IP>`                           |
| Update definitions           | `execute update-now`                          |
| Reboot                       | `execute reboot`                              |
| TFTP upgrade                 | `execute restore image tftp <file> <server>`  |
| USB upgrade                  | `execute restore image usb <file>`            |
| Stage firmware               | `execute restore secondary-image tftp <file>` |
| Select secondary boot        | `execute set-next-reboot secondary`           |
| Select primary boot          | `execute set-next-reboot primary`             |
| Initialize federated upgrade | `execute federated-upgrade initialize`        |
| Federated status             | `execute federated-upgrade status`            |
| Cancel federated upgrade     | `execute federated-upgrade cancel`            |
| Restart federated upgrade    | `execute federated-upgrade restart`           |

> [!IMPORTANT]
> Always verify command availability and exact behavior against the specific FortiOS build and hardware platform before production execution.

---

# 23. Complete TFTP Upgrade Checklist

```bash
# Test TFTP server reachability
execute ping 192.168.20.200

# Start firmware upgrade
execute restore image tftp a.pkg 192.168.20.200

# Confirm the firmware replacement
y

# Confirm additional warning if displayed
y

# After reboot
get system status

# Update security definitions
execute update-now
```

### Checklist

* [ ] Ping successful.
* [ ] Firmware file exists.
* [ ] Correct model verified.
* [ ] Upgrade path verified.
* [ ] Firmware signature verified.
* [ ] Upgrade confirmed.
* [ ] FortiGate rebooted.
* [ ] New firmware verified.
* [ ] Security definitions updated.
* [ ] Production traffic tested.

---

# 24. Controlled Upgrade Example

```bash
# Stage new firmware into secondary partition
execute restore secondary-image tftp a.pkg

# Configure the partition containing the staged firmware
# to be loaded at the next reboot
execute set-next-reboot secondary

# Reboot during the maintenance window
execute reboot
```

### Checklist

* [ ] Firmware staged successfully.
* [ ] Secondary partition contains target firmware.
* [ ] `set-next-reboot secondary` configured.
* [ ] Maintenance window reached.
* [ ] Reboot performed.
* [ ] New firmware loaded.
* [ ] System status verified.
* [ ] Production traffic validated.

> [!WARNING]
> Do not blindly use `primary` after staging an image to `secondary`. `set-next-reboot` selects the partition that will be booted next. ([Fortinet Documentation][1])

---

# 25. Upgrade Method Comparison

| Method             | Source                      | Activation             | Primary Use                |
| ------------------ | --------------------------- | ---------------------- | -------------------------- |
| GUI Upgrade        | FortiGuard / uploaded image | Immediate or scheduled | Normal upgrade             |
| Fabric Upgrade     | FortiGuard                  | Immediate or scheduled | Fabric-wide upgrade        |
| Federated Upgrade  | FortiGuard                  | Coordinated            | Multi-build/Fabric upgrade |
| TFTP Restore       | TFTP                        | Immediate              | Manual/recovery            |
| USB Restore        | USB                         | Immediate              | Offline/recovery           |
| Controlled Upgrade | Secondary partition         | Next reboot            | Staged activation          |
| Boot Menu TFTP     | TFTP                        | Boot-time              | Recovery/testing           |

---

# 26. Firmware Recovery Checklist

## 🧰 Recovery Kit

* [ ] Console cable.
* [ ] Laptop.
* [ ] TFTP server.
* [ ] Known-good firmware.
* [ ] Previous firmware.
* [ ] USB recovery media.
* [ ] Configuration backup.
* [ ] Management IP information.
* [ ] TFTP server IP.
* [ ] Firmware filename.

## Recovery Sequence

```text
Failed Upgrade
      │
      ▼
Console Access
      │
      ▼
Boot Menu
      │
      ├── TFTP
      │
      ├── Backup Firmware
      │
      └── USB / Alternative Recovery
      │
      ▼
Boot Known-Good Image
      │
      ▼
Verify Configuration
      │
      ▼
Verify Network
      │
      ▼
Verify Services
      │
      ▼
Restore Production State
```

---

# 27. Security Checklist

## 🔐 Firmware Security

* [ ] Firmware downloaded from trusted source.
* [ ] Firmware authenticity verified.
* [ ] Signature warnings investigated.
* [ ] Firmware model compatibility verified.
* [ ] Firmware file protected.
* [ ] TFTP access restricted.
* [ ] USB media protected.
* [ ] Configuration backups protected.
* [ ] Recovery images protected.

## 🛡️ Operational Security

* [ ] Management access restricted.
* [ ] Console access controlled.
* [ ] Firmware upgrade permissions restricted.
* [ ] Maintenance window approved.
* [ ] Upgrade activity logged.
* [ ] Recovery procedure documented.

---

# 28. Reliability Checklist

* [ ] OOB management available.
* [ ] Console access available.
* [ ] HA state verified.
* [ ] Fabric topology verified.
* [ ] NTP synchronized.
* [ ] FortiGuard operational.
* [ ] Licenses valid.
* [ ] Backup available.
* [ ] Previous firmware available.
* [ ] TFTP server ready.
* [ ] USB recovery ready.
* [ ] Maintenance window available.
* [ ] Rollback plan documented.

---

# 29. NSE Exam Traps ⚠️

> [!IMPORTANT]
> **Fabric Management** is the central GUI area used for firmware management in FortiOS 7.2.x.

> [!IMPORTANT]
> **FortiOS 7.2.0 introduced Feature and Mature firmware maturity tags.** ([Fortinet Documentation][2])

> [!TIP]
> Feature firmware is associated with newer functionality; Mature firmware does not introduce new major features and focuses on maintenance, fixes, and applicable vulnerability patches. ([Fortinet Documentation][5])

> [!IMPORTANT]
> `get system status` can be used to identify the FortiOS version, build, and maturity notation.

> [!IMPORTANT]
> A **federated update** can automatically follow multiple firmware builds in an upgrade path for supported FortiGate devices. ([Fortinet Documentation][3])

> [!WARNING]
> FortiAP and FortiSwitch do not follow the same multi-build upgrade path behavior as FortiGate in the FortiOS 7.2.0 federated-update workflow. ([Fortinet Documentation][3])

> [!IMPORTANT]
> `execute restore image tftp` replaces the current firmware and causes a reboot after installation.

> [!IMPORTANT]
> Firmware signature verification is a security control. Do not ignore unexpected authenticity warnings.

> [!IMPORTANT]
> `execute restore secondary-image` stages firmware in another partition for later activation. ([Fortinet Documentation][1])

> [!IMPORTANT]
> `execute set-next-reboot secondary` selects the secondary partition for the next boot. ([Fortinet Documentation][1])

> [!WARNING]
> `execute set-next-reboot primary` does **not** mean "boot the newly uploaded firmware." It means boot the firmware stored in the primary partition.

> [!IMPORTANT]
> `execute update-now` should be considered during post-upgrade security-definition validation.

> [!WARNING]
> A firmware downgrade can introduce configuration migration/reversion problems.

> [!CAUTION]
> The boot-menu `f` option can format the boot device. Treat it as destructive.

---

# 30. Expert Operational Rules 🧠

### Rule 01 — Backup First

```text
NO BACKUP
   ↓
NO PRODUCTION UPGRADE
```

### Rule 02 — Verify the Path

```text
Current
   ↓
Supported Path
   ↓
Target
```

Never assume:

```text
Current → Latest
```

is supported.

### Rule 03 — Verify the Image

```text
Source
 ↓
Model
 ↓
Build
 ↓
Signature
 ↓
Integrity
```

### Rule 04 — Separate Staging From Activation

Controlled upgrades allow:

```text
Stage
 ↓
Validate
 ↓
Schedule
 ↓
Reboot
 ↓
Activate
```

This can reduce maintenance-window pressure.

### Rule 05 — Post-Upgrade ≠ Reboot Complete

```text
Reboot
 ↓
System Status
 ↓
Definitions
 ↓
Services
 ↓
HA
 ↓
Fabric
 ↓
Traffic
```

### Rule 06 — Recovery Must Exist Before Failure

```text
Firmware
+
Config Backup
+
Console
+
TFTP
+
USB
=
Recovery Capability
```

### Rule 07 — Production Upgrades Should Be Staged

```text
LAB
 ↓
PILOT
 ↓
LOW-RISK
 ↓
PRODUCTION
 ↓
CRITICAL
```

### Rule 08 — Protect Firmware Sources

Firmware images and configuration backups are operationally sensitive assets.

* [ ] Restrict access.
* [ ] Use trusted sources.
* [ ] Protect backups.
* [ ] Protect recovery media.
* [ ] Document version provenance.

---

# 31. One-Minute NSE Revision

```text
Fabric Management
↓
Central firmware/device management
```

```text
Feature
↓
New features
↓
Gray
```

```text
Mature
↓
Maintenance-oriented release
↓
Green
```

```text
Upgrade Path
↓
Current
↓
Intermediate
↓
Target
```

```text
TFTP Upgrade
↓
execute ping
↓
execute restore image tftp
↓
Signature Verification
↓
Integrity Verification
↓
Install
↓
Reboot
```

```text
Controlled Upgrade
↓
restore secondary-image
↓
Stage Firmware
↓
set-next-reboot secondary
↓
Maintenance Window
↓
Reboot
```

```text
Federated Upgrade
↓
Fabric Management
↓
Follow Upgrade Path
↓
Intermediate Builds
↓
Multiple Reboots
↓
Target Firmware
```

```text
Recovery
↓
Console
↓
Boot Menu
↓
TFTP / Backup / USB
↓
Known-Good Firmware
```

```text
Post Upgrade
↓
get system status
↓
execute update-now
↓
Check Services
↓
Check HA
↓
Check Fabric
↓
Check Traffic
```

---

# 32. Quick Command Cheat Sheet

```bash
# System
get system status

# Connectivity
execute ping <IP>

# Security definitions
execute update-now

# Reboot
execute reboot

# TFTP firmware
execute restore image tftp <file> <server>

# USB firmware
execute restore image usb <file>

# Controlled firmware staging
execute restore secondary-image tftp <file>

# Boot secondary partition next
execute set-next-reboot secondary

# Boot primary partition next
execute set-next-reboot primary

# Federated upgrade
execute federated-upgrade initialize

# Federated status
execute federated-upgrade status

# Cancel federated upgrade
execute federated-upgrade cancel
```

---

# 33. Final Production Checklist 🚀

Before clicking **Upgrade**:

* [ ] I know the current firmware.
* [ ] I know the target firmware.
* [ ] I verified the upgrade path.
* [ ] I verified hardware compatibility.
* [ ] I verified firmware authenticity.
* [ ] I created a configuration backup.
* [ ] I have console/OOB access.
* [ ] I have a recovery firmware image.
* [ ] I have a recovery method.
* [ ] I verified HA/Fabric state.
* [ ] I verified FortiGuard.
* [ ] I verified NTP.
* [ ] I verified licensing.
* [ ] I have an approved maintenance window.
* [ ] I have a rollback plan.
* [ ] I know what I will validate after reboot.

### After Upgrade:

* [ ] `get system status`
* [ ] `execute update-now`
* [ ] Verify security definitions.
* [ ] Verify interfaces.
* [ ] Verify routing.
* [ ] Verify policies.
* [ ] Verify NAT.
* [ ] Verify VPN.
* [ ] Verify HA.
* [ ] Verify Security Fabric.
* [ ] Verify FortiGuard.
* [ ] Verify critical applications.
* [ ] Review logs.
* [ ] Confirm production traffic.
* [ ] Document final firmware/build.

---

# 34. Keywords

`FortiGate Firmware Upgrade` ·
`FortiOS Upgrade` ·
`FortiOS 7.2.0 Firmware Upgrade` ·
`FortiGate Upgrade Path` ·
`FortiGate Fabric Management` ·
`Fortinet Security Fabric Upgrade` ·
`FortiGate Federated Upgrade` ·
`FortiGate TFTP Upgrade` ·
`FortiGate USB Firmware Upgrade` ·
`FortiGate Controlled Upgrade` ·
`FortiOS Firmware Verification` ·
`FortiGate Firmware Signature` ·
`FortiGate Firmware Recovery` ·
`FortiGate Boot Menu` ·
`FortiGate Downgrade` ·
`FortiGuard Update` ·
`execute update-now` ·
`execute restore image tftp` ·
`execute restore secondary-image` ·
`execute set-next-reboot` ·
`FortiAP Firmware Upgrade` ·
`FortiSwitch Firmware Upgrade` ·
`FortiOS Upgrade Path` ·
`FortiGate Maintenance` ·
`FortiGate Firmware Troubleshooting`

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

# 36. SheynShield Security Rule 🔐

> **Never treat a firmware upgrade as a single reboot operation.**
>
> **A professional FortiGate upgrade is a controlled lifecycle:**
>
> `PLAN → BACKUP → VERIFY → STAGE → UPGRADE → REBOOT → UPDATE → VALIDATE → DOCUMENT`
>
> **Engineering Secure Networks**
>
> **SheynShield | Security & Design Knowledge Base**


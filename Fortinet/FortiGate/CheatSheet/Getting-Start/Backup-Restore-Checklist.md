# FortiGate Backup, Restore & Factory Reset Checklist

> **FortiOS CLI / NSE Quick Reference**
>
> Backup • Restore • Configuration Revision • YAML • Logs • IPS Signatures • Certificates • Factory Reset
>
> **Focus:** FortiOS configuration backup and recovery, configuration revisions, YAML configuration, log backup, certificate export, and factory-reset operations.

---

## Table of Contents

- [Pre-Change Safety Checklist](#pre-change-safety-checklist)
- [Backup Type Selection](#backup-type-selection)
- [Standard Configuration Backup](#standard-configuration-backup)
- [Full Configuration Backup](#full-configuration-backup)
- [YAML Configuration Backup & Restore](#yaml-configuration-backup--restore)
- [Flash Configuration Revision](#flash-configuration-revision)
- [Configuration Revision Management](#configuration-revision-management)
- [Central Management Backup](#central-management-backup)
- [Configuration Restore Validation](#configuration-restore-validation)
- [Certificate Export](#certificate-export)
- [Custom IPS Signature Backup](#custom-ips-signature-backup)
- [Log Backup](#log-backup)
- [Factory Reset Checklist](#factory-reset-checklist)
- [Factory Reset Command Comparison](#factory-reset-command-comparison)
- [Backup & Recovery Decision Tree](#backup--recovery-decision-tree)
- [NSE Exam Checklist](#nse-exam-checklist)
- [One-Minute CLI Recall](#one-minute-cli-recall)
- [Golden Rules](#golden-rules)

---

# Pre-Change Safety Checklist

Before performing a risky configuration change:

- [ ] Confirm the target FortiGate hostname.
- [ ] Confirm the FortiGate model.
- [ ] Confirm the FortiOS version/build.
- [ ] Confirm the current VDOM configuration.
- [ ] Confirm management access.
- [ ] Confirm out-of-band/console access if available.
- [ ] Create a configuration backup.
- [ ] Create a configuration revision before major changes.
- [ ] Verify that the backup was successfully created.
- [ ] Store the backup outside the FortiGate when required.
- [ ] Record the backup filename/comment.
- [ ] Record the configuration revision number.
- [ ] Document the change and rollback plan.

> **Operational Rule:**  
> Never start a destructive or high-risk operation without a verified recovery path.

---

# Backup Type Selection

Use the correct backup mechanism for the requirement.

| Requirement | Command / Method |
|---|---|
| Standard configuration | `execute backup config` |
| Complete configuration | `execute backup full-config` |
| YAML configuration | `execute backup yaml-config ...` |
| Restore YAML | `execute restore yaml-config ...` |
| Flash configuration revision | `execute backup config flash <comment>` |
| Restore flash revision | `execute restore config flash <number>` |
| Central management backup | `execute backup config management-station` |
| Disk log backup | `execute backup disk ...` |
| Memory log backup | `execute backup memory ...` |
| Custom IPS signatures | `execute backup ipsuserdefsig ...` |
| Certificate export | `execute vpn certificate local export ...` |

---

# Standard Configuration Backup

## Checklist

- [ ] Determine whether a normal configuration backup is sufficient.
- [ ] Verify the destination.
- [ ] Execute the configuration backup.
- [ ] Verify the resulting backup.
- [ ] Store the backup securely.

### Command

```bash
execute backup config
````

### Use When

* [ ] A normal configuration backup is required.
* [ ] The operational configuration needs to be preserved.
* [ ] A pre-change recovery point is required.

---

# Full Configuration Backup

## Checklist

* [ ] Determine whether a complete recovery backup is required.
* [ ] Prefer `full-config` for scenarios requiring more complete configuration data.
* [ ] Protect the resulting backup appropriately.
* [ ] Verify compatibility before restoring it to another device.

### Command

```bash
execute backup full-config
```

### NSE Concept

```text
config
   │
   └── Standard configuration backup

full-config
   │
   └── More complete configuration/recovery data
```

> **NSE Exam Tip:**
> Do not automatically treat `config` and `full-config` as interchangeable.

---

# YAML Configuration Backup & Restore

## Backup Checklist

* [ ] Confirm the destination server.
* [ ] Confirm TFTP connectivity.
* [ ] Select a meaningful filename.
* [ ] Export the YAML configuration.
* [ ] Verify the file on the destination server.
* [ ] Preserve the YAML metadata headers.

### Backup

```bash
execute backup yaml-config tftp fgt.yml 192.168.254.254
```

---

## Restore Checklist

Before restoring YAML:

* [ ] Verify the target FortiGate model.
* [ ] Verify FortiOS compatibility.
* [ ] Verify VDOM requirements.
* [ ] Verify the YAML file.
* [ ] Confirm the restore operation is intentional.
* [ ] Ensure a rollback/recovery path exists.

### Restore

```bash
execute restore yaml-config tftp fgt.yml 192.168.254.254
```

Alternative filename:

```bash
execute restore yaml-config tftp fgt.yaml 192.168.254.254
```

---

## YAML Metadata

Do **NOT** remove configuration metadata headers such as:

```text
#config-version=FGVMK6-7.2.0-FW-build1157-220331 : opmode=0: vdom=0: user=admin
#conf_file_ver=284434209243482
#buildno=1157
#global_vdom=1
```

### Metadata Checklist

* [ ] FortiGate model information preserved.
* [ ] FortiOS version/build information preserved.
* [ ] Operating mode information preserved.
* [ ] VDOM information preserved.
* [ ] Configuration-file version preserved.
* [ ] Global VDOM state preserved.

---

# Flash Configuration Revision

Configuration revisions can be created on flash where supported.

## Create Revision

```bash
execute backup config flash test
```

### Checklist

* [ ] Choose a meaningful revision/comment.
* [ ] Create the revision before risky changes.
* [ ] Record the revision number.
* [ ] Verify the revision exists.

Example:

```text
flash → Storage location
test  → Revision/comment
```

---

# Restore Flash Revision

## Command

```bash
execute restore config flash 2
```

### Checklist

* [ ] Identify the correct revision number.
* [ ] Confirm the revision content.
* [ ] Confirm rollback is intentional.
* [ ] Restore the selected revision.
* [ ] Verify system behavior after restoration.

---

# Configuration Revision Management

Configuration revisions provide historical configuration states.

### GUI

```text
GUI
 └── Top-right Admin menu
      └── Config
           └── Revision
```

## Revision Checklist

* [ ] Review available revisions.
* [ ] Identify the current configuration.
* [ ] Use **Diff** before rollback.
* [ ] Use **Revert** when rollback is required.
* [ ] Verify the device after reverting.
* [ ] Confirm newer revisions still exist if required.

---

## Diff vs Revert

| Function   | Purpose                                          |
| ---------- | ------------------------------------------------ |
| **Diff**   | Compare configuration revisions                  |
| **Revert** | Restore/revert to a previous configuration state |

### Mental Model

```text
Revision 1
    │
Revision 2
    │
Revision 3 ← Current
    │
    ├── Diff
    │     └── Compare configurations
    │
    └── Revert
          └── Return to previous state
```

> **Important:**
> **Revert does not mean "delete all newer revisions."**

---

# Configuration Revision Availability

Before relying on local revision storage:

* [ ] Verify the FortiGate model.
* [ ] Verify FortiOS support.
* [ ] Verify available flash/storage.
* [ ] Verify the required configuration-storage mechanism.
* [ ] Verify central management requirements where applicable.

### Storage Concept

```text
Configuration Revision
        │
        ├── Local storage / flash
        │
        └── Central management
              ├── FortiManager
              └── FortiGate Cloud
```

> **Note:**
> Exact revision-storage availability can depend on the FortiGate model, FortiOS release, and configured management/storage capabilities.

---

# Central Management Backup

FortiGate can send configuration backups to a management station.

## Command

```bash
execute backup config management-station
```

### Checklist

* [ ] Verify central management configuration.
* [ ] Verify FortiManager/FortiGate Cloud connectivity where applicable.
* [ ] Execute the backup.
* [ ] Verify the backup reached the management system.
* [ ] Confirm the backup timestamp/version.

### Conceptual Flow

```text
FortiGate
    │
    │ Configuration Backup
    ▼
Management Station
    │
    ├── FortiManager
    │
    └── FortiGate Cloud
```

---

# Configuration Restore Validation

A configuration backup should **not** automatically be assumed portable.

## Before Restore

* [ ] Confirm source FortiGate model.
* [ ] Confirm target FortiGate model.
* [ ] Confirm source FortiOS version.
* [ ] Confirm target FortiOS version.
* [ ] Confirm VDOM configuration.
* [ ] Confirm operating mode.
* [ ] Confirm configuration compatibility.
* [ ] Confirm backup integrity.
* [ ] Confirm configuration-file password if applicable.
* [ ] Confirm management access after restore.

---

## Common Restore Failure: Configuration File Error

### Possible Causes

```text
Wrong FortiGate model
        OR
Incompatible FortiOS version
        OR
Configuration incompatibility
```

Example:

```text
Source:
FG-100F
FortiOS 7.2.x

Target:
FG-60F
FortiOS 7.4.x
```

### Checklist

* [ ] Verify model compatibility.
* [ ] Verify FortiOS compatibility.
* [ ] Verify configuration format.
* [ ] Use an appropriate configuration migration procedure if required.
* [ ] Do not blindly restore an incompatible backup.

> **NSE Exam Trap:**
> A configuration backup is **not automatically portable between arbitrary FortiGate models or firmware versions**.

---

# Invalid Password

If the configuration backup is password-protected:

```text
Correct password → Restore succeeds
Wrong password   → Invalid password
```

## Checklist

* [ ] Determine whether the backup is password-protected.
* [ ] Obtain the correct backup password.
* [ ] Enter the correct password.
* [ ] Do not modify or guess the protected configuration.
* [ ] Verify the restore result.

---

# Certificate Export

Certificates can be exported from FortiGate.

### Example

```bash
execute vpn certificate local export tftp fortinet_ca_ssl cer cassl.cer 192.168.254.254
```

### Checklist

* [ ] Identify the correct local certificate.
* [ ] Verify the export format.
* [ ] Verify the TFTP server.
* [ ] Export the certificate.
* [ ] Verify the resulting certificate file.
* [ ] Protect certificate-related material appropriately.

### Flow

```text
FortiGate
    │
    │ Certificate Export
    ▼
TFTP Server
192.168.254.254
    │
    └── cassl.cer
```

---

# Custom IPS Signature Backup

Custom IPS signatures should be backed up separately when required.

## FTP

```bash
execute backup ipsuserdefsig ftp
```

## TFTP

```bash
execute backup ipsuserdefsig tftp
```

### Checklist

* [ ] Identify whether custom IPS signatures exist.
* [ ] Select the backup destination.
* [ ] Export user-defined IPS signatures.
* [ ] Verify the backup.
* [ ] Store the backup with the configuration recovery set.

---

# Log Backup

## Disk Logs

### All Logs

```bash
execute backup disk alllogs
```

### IPS Archive

```bash
execute backup disk ipsarchive
```

### Log Data

```bash
execute backup disk log
```

---

## Memory Logs

### All Logs

```bash
execute backup memory alllogs
```

### Log Data

```bash
execute backup memory log
```

---

## Log Backup Checklist

* [ ] Determine whether logs are stored on disk or memory.
* [ ] Select the required log category.
* [ ] Execute the backup.
* [ ] Verify the exported data.
* [ ] Store the backup securely.
* [ ] Record the backup date/time.

---

# Factory Reset Checklist

> ⚠️ **HIGH-RISK OPERATION**

Factory-reset commands can remove configuration/data and may make the FortiGate inaccessible.

## Before Factory Reset

* [ ] Confirm the reset is intentional.
* [ ] Confirm the target FortiGate.
* [ ] Take a configuration backup.
* [ ] Take a full configuration backup if required.
* [ ] Preserve required certificates.
* [ ] Preserve custom IPS signatures.
* [ ] Preserve required logs.
* [ ] Record the current FortiOS version.
* [ ] Record the current configuration.
* [ ] Confirm console/OOB access.
* [ ] Confirm the post-reset access procedure.
* [ ] Confirm the device can safely be taken offline.
* [ ] Document the change.

---

# `execute factoryreset`

## Command

```bash
execute factoryreset
```

### Checklist

* [ ] Confirm destructive operation.
* [ ] Confirm backup exists.
* [ ] Confirm recovery access.
* [ ] Execute factory reset.
* [ ] Wait for reset process.
* [ ] Verify reboot.
* [ ] Verify factory-default state.
* [ ] Reconfigure management access.
* [ ] Restore configuration only when appropriate.

### Conceptual Flow

```text
execute factoryreset
        │
        ├── Reset configuration/data
        │
        ├── Restore factory state
        │
        └── Reboot
```

---

# `execute factoryresetshutdown`

## Command

```bash
execute factoryresetshutdown
```

### Checklist

* [ ] Confirm factory reset is required.
* [ ] Confirm shutdown behavior is desired.
* [ ] Confirm backup exists.
* [ ] Confirm physical/OOB access.
* [ ] Execute the command.
* [ ] Verify the device shuts down.

### Key Difference

```text
factoryreset
      │
      └── Reset → Reboot

factoryresetshutdown
      │
      └── Reset → Shutdown
```

---

# `execute factoryreset2`

## Command

```bash
execute factoryreset2
```

### Key Concept

`factoryreset2` is **not identical** to a complete factory reset.

It resets many system components while preserving specific configuration areas.

Examples of preserved configuration areas include:

```text
system.global
system.global.vdom-mode
system.global.long-vdom-name
VDOMS
system.virtual-switch
system.interface
system.settings
router.static
router.static6
```

### Checklist

* [ ] Understand exactly what will be reset.
* [ ] Identify configuration sections that will be preserved.
* [ ] Verify the command behavior for the target FortiOS release.
* [ ] Create a backup before execution.
* [ ] Confirm the desired post-reset state.

> **NSE Exam Tip:**
> Do not memorize `factoryreset2` as simply another name for `factoryreset`.

---

# Factory Reset Command Comparison

| Command                        | Reset Behavior                                    | Final State                      |
| ------------------------------ | ------------------------------------------------- | -------------------------------- |
| `execute factoryreset`         | Full factory reset                                | Reboot                           |
| `execute factoryresetshutdown` | Full factory reset                                | Shutdown                         |
| `execute factoryreset2`        | Reset defaults while preserving specific sections | Version/implementation dependent |

---

# Backup & Recovery Decision Tree

```text
What do you need to preserve?
              │
              ├── Normal configuration
              │       └── execute backup config
              │
              ├── More complete recovery data
              │       └── execute backup full-config
              │
              ├── YAML configuration
              │       └── execute backup yaml-config ...
              │
              ├── Flash revision
              │       └── execute backup config flash <comment>
              │
              ├── Logs
              │       ├── execute backup disk alllogs
              │       ├── execute backup disk ipsarchive
              │       ├── execute backup disk log
              │       ├── execute backup memory alllogs
              │       └── execute backup memory log
              │
              ├── Custom IPS signatures
              │       └── execute backup ipsuserdefsig ...
              │
              └── Certificate
                      └── execute vpn certificate local export ...
```

---

# NSE Exam Checklist

## Configuration Backup

* [ ] `execute backup config` = standard configuration backup
* [ ] `execute backup full-config` = more complete configuration backup
* [ ] `execute backup yaml-config ...` = YAML export
* [ ] `execute restore yaml-config ...` = YAML restore

## Revision Control

* [ ] `execute backup config flash <comment>` = create flash revision
* [ ] `execute restore config flash <number>` = restore flash revision
* [ ] **Diff** = compare revisions
* [ ] **Revert** = rollback to previous revision
* [ ] Revert does **not** automatically mean deleting newer revisions

## Central Management

* [ ] `execute backup config management-station`
* [ ] FortiManager
* [ ] FortiGate Cloud

## Logs

* [ ] `execute backup disk alllogs`
* [ ] `execute backup disk ipsarchive`
* [ ] `execute backup disk log`
* [ ] `execute backup memory alllogs`
* [ ] `execute backup memory log`

## Security Data

* [ ] `execute backup ipsuserdefsig ftp`
* [ ] `execute backup ipsuserdefsig tftp`
* [ ] Certificate export command

## Factory Reset

* [ ] `execute factoryreset` → reset + reboot
* [ ] `execute factoryresetshutdown` → reset + shutdown
* [ ] `execute factoryreset2` → reset while preserving specific configuration areas

---

# NSE Exam Traps

### Trap 1 — `config` vs `full-config`

```text
config
   ≠
full-config
```

Do not assume both backups contain exactly the same information.

---

### Trap 2 — YAML Metadata

```text
YAML metadata
      ↓
DO NOT DELETE
```

Preserve the configuration metadata headers.

---

### Trap 3 — Diff vs Revert

```text
Diff   → Compare
Revert → Rollback
```

---

### Trap 4 — Revert Does Not Mean Delete

```text
Revision 1
Revision 2
Revision 3 ← Current

Revert → Revision 1

Revision 2/3
may still remain stored
```

---

### Trap 5 — Factory Reset Commands

```text
factoryreset
       ↓
Reset + Reboot

factoryresetshutdown
       ↓
Reset + Shutdown

factoryreset2
       ↓
Selective/default reset behavior
```

---

### Trap 6 — Configuration Portability

Before restoring:

```text
MODEL
  +
FORTIOS VERSION
  +
VDOM / OPERATION MODE
  +
CONFIGURATION COMPATIBILITY
```

must be considered.

---

# One-Minute CLI Recall

```bash
# ==========================================
# CONFIGURATION BACKUP
# ==========================================

execute backup config
execute backup full-config


# ==========================================
# YAML BACKUP / RESTORE
# ==========================================

execute backup yaml-config tftp fgt.yml 192.168.254.254

execute restore yaml-config tftp fgt.yml 192.168.254.254


# ==========================================
# FLASH CONFIGURATION REVISION
# ==========================================

execute backup config flash test
execute restore config flash 2


# ==========================================
# CENTRAL MANAGEMENT
# ==========================================

execute backup config management-station


# ==========================================
# LOG BACKUP
# ==========================================

execute backup disk alllogs
execute backup disk ipsarchive
execute backup disk log

execute backup memory alllogs
execute backup memory log


# ==========================================
# CUSTOM IPS SIGNATURE
# ==========================================

execute backup ipsuserdefsig ftp
execute backup ipsuserdefsig tftp


# ==========================================
# CERTIFICATE EXPORT
# ==========================================

execute vpn certificate local export tftp \
fortinet_ca_ssl cer cassl.cer 192.168.254.254


# ==========================================
# FACTORY RESET
# ==========================================

execute factoryreset

execute factoryresetshutdown

execute factoryreset2
```

---

# Golden Rules

> ### 🔥 Backup Before Change
>
> Create a verified recovery point before risky configuration changes.

> ### 🔥 Revision Before Risk
>
> Use configuration revisions when you need a fast rollback point.

> ### 🔥 Verify Before Restore
>
> Always validate FortiGate model, FortiOS version, VDOM/mode, and configuration compatibility.

> ### 🔥 Preserve YAML Metadata
>
> Do not remove YAML configuration metadata headers.

> ### 🔥 Diff Before Revert
>
> Understand the configuration difference before performing rollback.

> ### 🔥 Factory Reset Means Destruction
>
> Treat factory-reset commands as destructive operations and verify your recovery path first.

---

# Quick Reference Matrix

| Task                      | CLI                                        |
| ------------------------- | ------------------------------------------ |
| Standard config backup    | `execute backup config`                    |
| Full config backup        | `execute backup full-config`               |
| YAML backup               | `execute backup yaml-config ...`           |
| YAML restore              | `execute restore yaml-config ...`          |
| Create flash revision     | `execute backup config flash <comment>`    |
| Restore flash revision    | `execute restore config flash <number>`    |
| Central management backup | `execute backup config management-station` |
| Disk all logs             | `execute backup disk alllogs`              |
| Disk IPS archive          | `execute backup disk ipsarchive`           |
| Disk logs                 | `execute backup disk log`                  |
| Memory all logs           | `execute backup memory alllogs`            |
| Memory logs               | `execute backup memory log`                |
| IPS signatures via FTP    | `execute backup ipsuserdefsig ftp`         |
| IPS signatures via TFTP   | `execute backup ipsuserdefsig tftp`        |
| Certificate export        | `execute vpn certificate local export ...` |
| Factory reset + reboot    | `execute factoryreset`                     |
| Factory reset + shutdown  | `execute factoryresetshutdown`             |
| Selective/default reset   | `execute factoryreset2`                    |

---

# SheynShield Resources

## 🎥 Video Learning

* [YouTube — SheynShield](https://youtube.com/@sheynshield)

  * Fortinet NSE content
  * FortiGate troubleshooting
  * Network Security Engineering

## 📚 Notes & Updates

* [Telegram — SheynShield](https://t.me/sheynshield)

## 💼 Professional Network

* [LinkedIn — Shayan-heydarikhah](https://linkedin.com/in/shayan-heydarikhah)

## 🐙 Technical Knowledge Base

* [SheynShield GitHub](https://github.com/Shayan-heydarikhah/sheynshield)

---

## Tags

`FortiGate` `FortiOS` `Fortinet` `NSE4` `NSE7` `Backup` `Restore` `Configuration` `YAML` `Configuration Revision` `Factory Reset` `IPS` `Certificates` `Network Security` `Cyber Security`

```

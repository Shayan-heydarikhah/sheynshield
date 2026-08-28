# FortiGate Backup, Restore & Factory Reset 

> **FortiOS CLI Quick Reference**
> Backup & Restore • Configuration Revision • YAML Config • Certificates • Factory Reset
>
> 🎯 **NSE Focus:** Know the difference between `config`, `full-config`, `yaml-config`, revision control, and factory-reset commands.

---

## 1. Backup Types — Quick Map

| Command                             | Purpose                               | Key Point                                   |
| ----------------------------------- | ------------------------------------- | ------------------------------------------- |
| `execute backup disk alllogs`       | Backup all logs to disk               | Log backup                                  |
| `execute backup disk ipsarchive`    | Backup IPS archive                    | Useful for IPS-related archive data         |
| `execute backup disk log`           | Backup log data                       | Log backup                                  |
| `execute backup memory alllogs`     | Backup all logs from memory           | Memory-based logs                           |
| `execute backup memory log`         | Backup log from memory                | Memory-based logs                           |
| `execute backup ipsuserdefsig ftp`  | Backup custom IPS signatures via FTP  | User-defined signatures                     |
| `execute backup ipsuserdefsig tftp` | Backup custom IPS signatures via TFTP | User-defined signatures                     |
| `execute backup config`             | Backup configuration                  | Standard configuration backup               |
| `execute backup full-config`        | Full configuration backup             | Includes additional sensitive/complete data |
| `execute backup yaml-config ...`    | Export YAML configuration             | Structured configuration format             |

### ⭐ Recommended Log Backups

For practical log preservation, these are especially useful:

```bash
execute backup disk alllogs
execute backup disk ipsarchive
```

---

# 2. Standard Configuration Backup

```bash
execute backup config
```

Use this for a normal FortiGate configuration backup.

### Configuration vs Full Configuration

```bash
execute backup config
```

**Standard configuration backup**

```bash
execute backup full-config
```

**Full configuration backup**

The `full-config` backup contains additional information required for a more complete recovery scenario.

> ⚠️ **Exam Tip:**
> Think of `full-config` when the requirement is **complete/disaster recovery**, rather than simply exporting the operational configuration.

---

# 3. YAML Configuration Backup

FortiOS also supports exporting the configuration in YAML format.

```bash
execute backup yaml-config tftp fgt.yml 192.168.254.254
```

### Restore YAML Configuration

```bash
execute restore yaml-config tftp fgt.yml 192.168.254.254
```

or:

```bash
execute restore yaml-config tftp fgt.yaml 192.168.254.254
```

### ⚠️ YAML Header Metadata

Exported YAML configuration files can contain metadata comments such as:

```text
#config-version=FGVMK6-7.2.0-FW-build1157-220331 : opmode=0: vdom=0: user=admin
#conf_file_ver=284434209243482
#buildno=1157
#global_vdom=1
```

**Do NOT remove these metadata lines.**

They provide important information about:

* FortiGate model
* FortiOS version/build
* Operating mode
* VDOM information
* Configuration file version
* Global VDOM state

---

# 4. Backup Configuration to Flash

A configuration revision can be created directly on flash:

```bash
execute backup config flash test
```

Here:

```text
flash → storage location
test  → revision/comment name
```

This creates a configuration revision with the comment:

```text
test
```

---

# 5. Restore a Configuration Revision

```bash
execute restore config flash 2
```

The number represents the stored configuration revision.

### GUI Location

```text
GUI
 └── Top-right Admin menu
      └── Config
           └── Revision
```

The Revision panel provides functions such as:

| Function   | Purpose                                                      |
| ---------- | ------------------------------------------------------------ |
| **Revert** | Restore/revert to a previous configuration revision          |
| **Diff**   | Compare a revision against the current/running configuration |

### 🧠 Important Concept

If you create a new revision and later revert to an older version:

```text
Revision 1
Revision 2
Revision 3  ← current
      ↓
Revert
      ↓
Revision 1
```

The newer revision can still remain stored on flash.

So:

> **Revert ≠ delete the newer revision**

---

# 6. Configuration Revision Control

Configuration revision management allows multiple configuration versions to be retained.

### Requirements / Availability

Revision control can be available on models with:

```text
512 MB flash or higher
```

depending on the specific FortiGate model and FortiOS support.

Revision control requires an available configuration storage mechanism, such as:

* Central management server
* Local hard drive/storage where supported

### Central Management Options

The central management server can be:

* **FortiManager**
* **FortiGate Cloud**

If the required central-management/storage capability is not configured, FortiGate may prompt you to:

1. Enable central management
2. Obtain the required license

---

# 7. Configuration Backup on Cloud / Central Management

FortiGate can send configuration backups to its management station:

```bash
execute backup config management-station
```

Possible centralized destinations include:

```text
FortiManager
FortiGate Cloud
```

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

# 8. Configuration Restore Troubleshooting

## ❌ Configuration File Error

### Meaning

The configuration file is incompatible with the FortiGate.

Common causes:

```text
Wrong FortiGate model
        OR
Different FortiOS firmware version
```

### Example

```text
Backup:
FG-100F
FortiOS 7.2.x

Target:
FG-60F
FortiOS 7.4.x
```

The configuration may not be directly compatible.

### Fix

Use a configuration file matching:

* Correct FortiGate model
* Correct FortiOS firmware version

> ⚠️ **NSE Exam Trap:**
> A configuration backup is not automatically portable between arbitrary FortiGate models or firmware versions.

---

# 9. ❌ Invalid Password

A configuration file may be protected with a password.

If the password supplied during restore does not match the password used to protect the backup:

```text
Invalid password
```

### Fix

Use the correct configuration-file password.

```text
Configuration Backup
        │
        ├── Password protected
        │
        ▼
Restore
        │
        ├── Correct password → ✅
        │
        └── Wrong password   → ❌
```

---

# 10. Certificate Export

Certificates can also be exported from the FortiGate.

Example:

```bash
execute vpn certificate local export tftp fortinet_ca_ssl cer cassl.cer 192.168.254.254
```

Conceptually:

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

# 11. IPS User-Defined Signature Backup

Custom IPS signatures should be backed up separately when required.

### FTP

```bash
execute backup ipsuserdefsig ftp
```

### TFTP

```bash
execute backup ipsuserdefsig tftp
```

These backups are particularly useful when custom IPS signatures have been developed for the environment.

---

# 12. Log Backup

### Disk

```bash
execute backup disk alllogs
execute backup disk ipsarchive
execute backup disk log
```

### Memory

```bash
execute backup memory alllogs
execute backup memory log
```

### Quick Comparison

```text
                 Backup
                   │
          ┌────────┴────────┐
          │                 │
        Disk              Memory
          │                 │
    ┌─────┴─────┐      ┌────┴────┐
    │           │      │         │
 all logs   IPS archive all logs  log
```

---

# 13. Factory Reset

⚠️ **HIGH-RISK COMMANDS**

Factory reset removes configuration/data and can make the device inaccessible.

---

## `execute factoryreset`

```bash
execute factoryreset
```

### Behavior

Performs a factory reset and reboots the FortiGate after the reset process.

Conceptually:

```text
Factory Reset
     │
     ├── Purge configuration/data
     ├── Restore factory state
     ├── Reformat/reset required storage
     └── Reboot
```

---

# 14. `execute factoryresetshutdown`

```bash
execute factoryresetshutdown
```

Similar to factory reset, but instead of rebooting into the factory-default state, the FortiGate **shuts down** after the reset process.

### Difference

| Command                        | After Reset |
| ------------------------------ | ----------- |
| `execute factoryreset`         | Reboots     |
| `execute factoryresetshutdown` | Shuts down  |

### 🧠 Memory Trick

```text
factoryreset
      ↓
   RESET
      ↓
   REBOOT

factoryresetshutdown
      ↓
   RESET
      ↓
  SHUTDOWN
```

---

# 15. `execute factoryreset2`

```bash
execute factoryreset2
```

This factory-reset variant restores default values for many system components while preserving specific configuration areas.

The preserved areas include configuration such as:

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

> ⚠️ **Exam Tip:**
> Do not treat `factoryreset2` as identical to a complete `factoryreset`.
> The key distinction is that **specific configuration sections are preserved**.

---

# 16. Factory Reset — Comparison

| Command                        | Main Behavior                                     | Reboot / Shutdown                 |
| ------------------------------ | ------------------------------------------------- | --------------------------------- |
| `execute factoryreset`         | Full factory reset                                | Reboot                            |
| `execute factoryresetshutdown` | Full factory reset                                | Shutdown                          |
| `execute factoryreset2`        | Reset defaults while preserving specific sections | Depends on implementation/version |

---

# 17. Backup & Recovery Decision Tree

```text
Need to preserve configuration?
          │
          ├── Normal configuration
          │       └── execute backup config
          │
          ├── Complete recovery
          │       └── execute backup full-config
          │
          ├── Structured YAML
          │       └── execute backup yaml-config ...
          │
          ├── Configuration revision
          │       └── execute backup config flash <comment>
          │
          ├── Logs
          │       ├── execute backup disk alllogs
          │       └── execute backup disk ipsarchive
          │
          └── Custom IPS signatures
                  └── execute backup ipsuserdefsig ...
```

---

# 18. ⭐ NSE Exam Quick Recall

| Topic                   | Remember                                   |
| ----------------------- | ------------------------------------------ |
| Normal config backup    | `execute backup config`                    |
| Complete backup         | `execute backup full-config`               |
| YAML export             | `execute backup yaml-config ...`           |
| YAML restore            | `execute restore yaml-config ...`          |
| Flash revision          | `execute backup config flash <comment>`    |
| Revision restore        | `execute restore config flash <number>`    |
| Log backup              | `execute backup disk alllogs`              |
| IPS archive             | `execute backup disk ipsarchive`           |
| Custom IPS signatures   | `execute backup ipsuserdefsig`             |
| Certificate export      | `execute vpn certificate local export ...` |
| Factory reset           | `execute factoryreset`                     |
| Reset + shutdown        | `execute factoryresetshutdown`             |
| Selective/default reset | `execute factoryreset2`                    |
| Revision GUI            | `Admin → Config → Revision`                |
| Revision comparison     | **Diff**                                   |
| Revision rollback       | **Revert**                                 |
| Revision storage        | Flash ≥ 512 MB where supported             |
| Cloud/Central backup    | `execute backup config management-station` |

---

# 19. ⚡ One-Minute 

```bash
# ==============================
# CONFIG BACKUP
# ==============================

execute backup config
execute backup full-config

# YAML
execute backup yaml-config tftp fgt.yml 192.168.254.254

# YAML RESTORE
execute restore yaml-config tftp fgt.yml 192.168.254.254


# ==============================
# FLASH REVISION
# ==============================

execute backup config flash test
execute restore config flash 2


# ==============================
# CENTRAL MANAGEMENT
# ==============================

execute backup config management-station


# ==============================
# LOG BACKUP
# ==============================

execute backup disk alllogs
execute backup disk ipsarchive
execute backup disk log

execute backup memory alllogs
execute backup memory log


# ==============================
# CUSTOM IPS SIGNATURE
# ==============================

execute backup ipsuserdefsig ftp
execute backup ipsuserdefsig tftp


# ==============================
# CERTIFICATE EXPORT
# ==============================

execute vpn certificate local export tftp \
fortinet_ca_ssl cer cassl.cer 192.168.254.254


# ==============================
# FACTORY RESET
# ==============================

execute factoryreset

execute factoryresetshutdown

execute factoryreset2
```

---

## 🔥 Core Mental Model

```text
                 FORTIGATE BACKUP
                       │
       ┌───────────────┼────────────────┐
       │               │                │
   Configuration      Logs           Security
       │               │                │
   ┌───┼────┐      ┌───┴────┐       ┌───┴────┐
   │   │    │      │        │       │        │
 config full YAML  disk   memory   IPS      Cert
   │
   └── Revision / Flash
           │
           ├── Diff
           └── Revert
```

> **Golden Rule:**
> **Backup before change → Revision before risky operation → Verify compatibility before restore → Factory reset only when you explicitly intend to destroy/reset configuration.**

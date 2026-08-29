# 💽 FortiGate RAID Management & Troubleshooting Checklist — FortiOS

> **SheynShield | Engineering Secure Networks**  
> Production-ready checklist for **FortiGate internal disk RAID management, verification, rebuild operations, hardware troubleshooting, and maintenance planning**.

---

# 📌 Table of Contents

- [RAID Planning Checklist](#raid-planning-checklist)
- [Pre-Change Safety Checklist](#pre-change-safety-checklist)
- [Hardware Compatibility Checklist](#hardware-compatibility-checklist)
- [Disk Inventory Checklist](#disk-inventory-checklist)
- [RAID Status Verification](#raid-status-verification)
- [RAID Configuration Checklist](#raid-configuration-checklist)
- [RAID Rebuild Checklist](#raid-rebuild-checklist)
- [Disk Hardware Troubleshooting](#disk-hardware-troubleshooting)
- [Maintenance Workflow](#maintenance-workflow)
- [Troubleshooting Decision Tree](#troubleshooting-decision-tree)
- [Exam & Interview Memory Notes](#exam--interview-memory-notes)

---

# 🏗️ RAID Planning Checklist

Before managing RAID on FortiGate:

- [ ] Confirm FortiGate model supports RAID
- [ ] Verify internal disks are RAID-capable
- [ ] Check current RAID configuration
- [ ] Identify current RAID level
- [ ] Identify disk members
- [ ] Verify disk health status
- [ ] Schedule maintenance window if required

---

# ⚠️ Pre-Change Safety Checklist

Before any RAID modification:

- [ ] Take FortiGate configuration backup
- [ ] Backup important logs/data
- [ ] Document current RAID status
- [ ] Export required reports
- [ ] Verify replacement/recovery plan
- [ ] Confirm maintenance approval
- [ ] Inform stakeholders about possible downtime

---

# 🚨 Critical Warning

RAID operations can be destructive.

Before executing RAID changes:

```text
Backup
  +
Maintenance Window
  +
Platform Validation
  +
Rollback Plan
````

---

# 🖥️ Hardware Compatibility Checklist

Verify:

* [ ] Device is factory-built Fortinet appliance
* [ ] Internal storage supports RAID
* [ ] FortiOS version supports required operation
* [ ] Hardware documentation reviewed

Important:

```text
Not every FortiGate model supports RAID.
```

RAID availability depends on:

* [ ] Hardware platform
* [ ] Disk configuration
* [ ] FortiOS support

---

# 💽 Disk Inventory Checklist

## List Available Disks

Command:

```bash
execute disk list
```

Purpose:

* [ ] Detect installed disks
* [ ] Verify disk visibility
* [ ] Confirm disk count
* [ ] Identify disk members

Expected information:

```text
Disk Inventory

    |
    ├── Disk ID
    ├── Capacity
    ├── State
    └── Availability
```

---

# 🔍 RAID Status Verification Checklist

## Check Current RAID Status

Command:

```bash
execute disk raid status
```

Validate:

* [ ] RAID enabled/disabled state
* [ ] RAID level
* [ ] Disk members
* [ ] Array health
* [ ] Rebuild status
* [ ] Failed disk condition

Example workflow:

```text
Current RAID

      ↓

execute disk raid status

      ↓

Analyze:

- RAID Level
- Disk State
- Rebuild Progress
- Health
```

---

# ⚙️ RAID Configuration Checklist

## Enable RAID

Command:

```bash
execute disk raid enable
```

Validate:

* [ ] Platform supports RAID
* [ ] Backup completed
* [ ] Maintenance window available
* [ ] Data impact understood

⚠️ Possible impact:

* Reboot may be required
* Existing disk data may be affected

---

## Change RAID Level

Command:

```bash
execute disk raid rebuild-level
```

Validate:

* [ ] Current RAID documented
* [ ] Target RAID level confirmed
* [ ] Backup completed
* [ ] Downtime planned

Possible impact:

* [ ] RAID rebuild required
* [ ] Disk data may be erased

---

## Disable RAID

Command:

```bash
execute disk raid disable
```

Validate:

* [ ] Business approval obtained
* [ ] Backup completed
* [ ] Data migration completed if required

⚠️ Important:

```text
Disable RAID

        ↓

Possible reboot

        ↓

Disk data loss
```

---

# 🔄 RAID Rebuild Checklist

Before rebuild:

* [ ] Verify current RAID state
* [ ] Verify disk health
* [ ] Confirm target RAID configuration
* [ ] Backup configuration/data

During rebuild:

* [ ] Monitor RAID status
* [ ] Monitor disk health
* [ ] Avoid unnecessary reboot/power interruption

After rebuild:

* [ ] Verify RAID state
* [ ] Verify disk members
* [ ] Confirm system operation

---

# 🔧 Hardware Disk Troubleshooting Checklist

## Hardware Disk Information

Command:

```bash
diagnose hardware deviceinfo disk
```

Use for:

* [ ] Disk detection issues
* [ ] Hardware information
* [ ] Disk state verification
* [ ] Hardware troubleshooting

---

# 📋 Quick Command Reference

| Command                             | Purpose                           |
| ----------------------------------- | --------------------------------- |
| `execute disk list`                 | Display detected disks            |
| `execute disk raid status`          | Display RAID configuration/status |
| `execute disk raid enable`          | Enable RAID                       |
| `execute disk raid rebuild-level`   | Change/rebuild RAID level         |
| `execute disk raid disable`         | Disable RAID                      |
| `diagnose hardware deviceinfo disk` | Display hardware disk information |

---

# 🧭 RAID Operational Workflow

```text
              Current System
                    |
                    ▼
          Check Disk Inventory
                    |
                    |
          execute disk list
                    |
                    ▼
            Check RAID Status
                    |
                    |
       execute disk raid status
                    |
                    ▼
          Backup Configuration
                    |
                    ▼
        Schedule Maintenance
                    |
                    ▼
        RAID Modification
                    |
          ┌─────────┴─────────┐
          │                   │
          ▼                   ▼
    Enable RAID        Disable RAID
          │                   │
          └─────────┬─────────┘
                    |
                    ▼
                Reboot
                    |
                    ▼
          Verify Final Status
                    |
        ┌───────────┴───────────┐
        ▼                       ▼

execute disk raid status

diagnose hardware deviceinfo disk
```

---

# 🚨 RAID Troubleshooting Checklist

## RAID Not Detected

Check:

* [ ] Hardware platform support
* [ ] Disk installation
* [ ] Disk visibility

Commands:

```bash
execute disk list
```

```bash
diagnose hardware deviceinfo disk
```

---

## RAID Degraded

Check:

* [ ] Failed disk member
* [ ] RAID status
* [ ] Rebuild progress
* [ ] Hardware health

Command:

```bash
execute disk raid status
```

---

## Disk Failure Investigation

Validate:

* [ ] Physical disk state
* [ ] RAID membership
* [ ] Hardware information
* [ ] Replacement requirement

---

# 🔐 Production Best Practices

* [ ] Never modify RAID without backup
* [ ] Verify platform compatibility first
* [ ] Document current RAID state
* [ ] Perform changes during maintenance window
* [ ] Monitor rebuild process
* [ ] Validate after reboot
* [ ] Keep hardware documentation available

---

# 🧠 Exam & Interview Memory Notes

Remember:

* [ ] RAID support depends on FortiGate hardware platform
* [ ] Disk inventory command:

```bash
execute disk list
```

* [ ] RAID status command:

```bash
execute disk raid status
```

* [ ] RAID enable command:

```bash
execute disk raid enable
```

* [ ] RAID rebuild/change level:

```bash
execute disk raid rebuild-level
```

* [ ] RAID disable:

```bash
execute disk raid disable
```

* [ ] Hardware disk troubleshooting:

```bash
diagnose hardware deviceinfo disk
```

---

# 🎯 Golden Rule

> RAID configuration changes are **maintenance operations with possible destructive impact**.
> Always verify hardware support, create backups, schedule downtime, and validate the final RAID state.

---

# 🧩 Final Mental Model

```text
          FortiGate Storage

                 |
                 ▼

          Disk Inventory

                 |
                 ▼

          RAID Status

                 |
                 ▼

       Configuration Decision

          |
   ┌──────┴──────┐
   ▼             ▼

Enable RAID   Disable RAID

   |
   ▼

Rebuild / Verify

   |
   ▼

Healthy Storage
```

---

# 🔗 SheynShield Resources

## 🎥 Video Learning

* [YouTube — SheynShield](https://youtube.com/@sheynshield)

  * Fortinet NSE content
  * FortiGate troubleshooting
  * Network Security Engineering

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

⭐ **SheynShield | Engineering Secure Networks**
**FortiGate RAID = Verify → Backup → Change → Rebuild → Validate**

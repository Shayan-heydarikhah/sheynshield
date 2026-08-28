# FortiGate RAID — Cheat Sheet

> **Purpose:** FortiGate internal disk RAID management, verification, and rebuild commands.

---

## ⚠️ Important Notes

* RAID is supported only on **Fortinet factory-built devices/series that include RAID-capable disks**.
* **Changing RAID configuration can require a reboot and can erase existing data on the disks.**
* Before changing RAID:

  * Take a configuration backup.
  * Back up important logs/data.
  * Verify the current RAID status.
  * Schedule maintenance downtime.

---

## 💽 Disk & RAID Commands

### List Available Disks

```bash
execute disk list
```

**Purpose:**
Display the disks detected by the FortiGate.

---

### Check RAID Status

```bash
execute disk raid status
```

**Purpose:**
Display the current RAID configuration and status.

Useful for checking:

* RAID level
* Disk members
* RAID state
* Rebuild status
* Disk health/state

---

## 🛠️ Enable RAID

```bash
execute disk raid enable
```

**Purpose:**
Enable RAID on supported FortiGate platforms.

> ⚠️ RAID configuration may require a reboot and can result in data loss.

---

## 🔄 Change RAID Level

```bash
execute disk raid rebuild-level
```

**Purpose:**
Change/rebuild the RAID level.

> ⚠️ Changing the RAID level can require rebuilding the RAID array and may erase existing disk data.

---

## 🚫 Disable RAID

```bash
execute disk raid disable
```

**Purpose:**
Disable the RAID configuration.

> ⚠️ According to the lab notes, disabling RAID requires a reboot and **erases all data on the disks**.

---

## 🔍 Hardware Disk Information

```bash
diagnose hardware deviceinfo disk
```

**Purpose:**
Display low-level hardware information about the installed disks.

Useful for troubleshooting:

* Disk detection
* Disk hardware information
* Disk state
* Hardware-level disk diagnostics

---

# 🧠 Quick Command Table

| Command                             | Purpose                        |
| ----------------------------------- | ------------------------------ |
| `execute disk list`                 | List detected disks            |
| `execute disk raid status`          | Show RAID status               |
| `execute disk raid enable`          | Enable RAID                    |
| `execute disk raid rebuild-level`   | Rebuild/change RAID level      |
| `execute disk raid disable`         | Disable RAID                   |
| `diagnose hardware deviceinfo disk` | Show disk hardware information |

---

# 🚨 RAID Change Workflow

```text
        Current RAID
             │
             ▼
    Check Disk Inventory
             │
             │ execute disk list
             ▼
      Check RAID Status
             │
             │ execute disk raid status
             ▼
       Backup Data/Config
             │
             ▼
     Maintenance Window
             │
             ▼
   Change RAID Configuration
             │
      ┌──────┴──────┐
      │             │
      ▼             ▼
 RAID Enable    RAID Disable
      │             │
      └──────┬──────┘
             ▼
          Reboot
             │
             ▼
       Verify RAID/Disk
             │
             ├── execute disk raid status
             └── diagnose hardware deviceinfo disk
```

---

## 🎯 Exam / Interview Notes

* **RAID support:** Depends on the FortiGate hardware platform.
* **Before RAID changes:** Always take a backup.
* **RAID status:**
  `execute disk raid status`
* **Disk inventory:**
  `execute disk list`
* **Disable RAID:** May require reboot and **disk data loss**.
* **RAID level change:** Can require a rebuild and **data loss**.
* **Hardware-level disk troubleshooting:**
  `diagnose hardware deviceinfo disk`

> 🔥 **Golden Rule:** Treat RAID configuration changes as a **destructive maintenance operation** unless the specific FortiOS/platform documentation confirms otherwise.

# 📋 FortiGate Disk Management & RAID Operational Checklist

A step-by-step production verification checklist for managing internal storage drives, configuring RAID arrays, and diagnosing hardware disks on FortiGate appliances.

---

## 🛠️ Phase 1: Pre-Deployment & Compatibility Checks

- [ ] **Factory Drive Parity:** Confirm all installed storage drives are **Fortinet Factory-Build Series** certified units.
- [ ] **Data Backup Verification:** Ensure all system logs, PCAP files, and configurations are fully backed up before executing RAID operations.
- [ ] **Hardware Capacity Verification:** Confirm the FortiGate model features multiple physical disk slots supporting hardware or software RAID architectures.

---

## ⚠️ Phase 2: RAID Configuration & Array Modifications

> [!CAUTION]
> **Data Destruction & Reboot Warning:**
> Enabling, disabling, or modifying RAID levels **WILL ERASE ALL DATA** on all attached drives and automatically reboot the FortiGate unit.

- [ ] **Enable RAID Array:** Execute `execute disk raid enable` (Triggers system reboot and full disk format).
- [ ] **Disable RAID Array:** Execute `execute disk raid disable` to revert to standalone disk mode (Triggers system reboot and full disk format).
- [ ] **Configure Rebuild Level:** Set the target rebuild parameters using `execute disk raid rebuild-level` if modifying array performance or redundancy characteristics.

---

## 🧪 Phase 3: Post-Deployment Verification & Diagnostics

- [ ] **Physical Disk Inventory:** Run `execute disk list` to verify all physical drives are detected and online.
- [ ] **RAID Health Verification:** Run `execute disk raid status` to verify array synchronization, member status, and health metrics.
- [ ] **SMART & Hardware Inspection:** Run `diagnose hardware deviceinfo disk` to review low-level SMART attributes and drive health counters.

---

## ⚡ Quick Reference Commands Table

| Task | Command | Operational Impact |
| :--- | :--- | :--- |
| **List Installed Physical Disks** | `execute disk list` | Non-disruptive |
| **Check RAID Array Health** | `execute disk raid status` | Non-disruptive |
| **Inspect Low-Level Hardware Info** | `diagnose hardware deviceinfo disk` | Non-disruptive |
| **Enable RAID Array** | `execute disk raid enable` | ⚠️ **Reboot & Erase Data** |
| **Disable RAID Array** | `execute disk raid disable` | ⚠️ **Reboot & Erase Data** |
| **Change RAID Rebuild Level** | `execute disk raid rebuild-level` | ⚠️ **Reboot & Erase Data** |

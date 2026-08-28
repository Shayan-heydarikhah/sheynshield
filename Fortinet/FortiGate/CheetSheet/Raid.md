# 💾 FortiGate Disk Management & RAID Configuration — Operational Cheat Sheet

A production-ready technical reference for managing internal storage drives, RAID arrays, and disk diagnostics on storage-equipped FortiGate appliances.

---

## ⚠️ Critical Prerequisites & Operations Notice

> [!CAUTION]
> **Data Destruction Warning:**
> Enabling, disabling, or changing the rebuild level of a RAID array **will format all attached physical disks, destroy all saved log data/pcap archives, and reboot the FortiGate device**. Always perform a full backup before modifying storage properties.

* **Disk Compatibility:** Only Fortinet factory-certified hard drives and SSDs (Fortinet Factory-Build Series) are supported. Non-factory drives will be flagged as uncertified or ignored by the storage controller.
* **Scope:** Applies to FortiGate models with multiple internal storage drives (e.g., FortiGate 100E/100F/200E/200F with dual SSDs or enterprise rack-mount models).

---

## ⚙️ 1. RAID Operations & Level Modifications

All RAID configuration changes are executed from the administrative CLI and trigger an immediate disk initialization and system reboot sequence.

```text
# Enable RAID on supported multi-disk FortiGate units
execute disk raid enable

# Disable RAID array (Reverts disks to standalone mode and erases all data)
execute disk raid disable

# Modify RAID rebuild level or change array configuration
execute disk raid rebuild-level <level_option>

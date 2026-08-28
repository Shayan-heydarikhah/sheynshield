# FortiGate Fabric Management & Firmware Upgrade  

> **FortiOS 7.2.0 — Security Fabric | Firmware Management | Controlled Upgrade | TFTP | USB | Firmware Verification**

---

## 1. Fabric Management

**Fabric Management** is used to centrally manage FortiGate, FortiAP, and FortiSwitch devices that belong to the **Fortinet Security Fabric**.

### What Can Be Managed?

From:

```text
System
 └── Fabric Management
```

Administrators can:

* Authorize Fabric devices
* Register Fabric devices
* Monitor device status
* Manage firmware versions
* Upgrade multiple Fabric devices
* Organize devices into upgrade groups
* Review firmware maturity
* Monitor upgrade status

### Fabric Management Dashboard

The page provides information such as:

| Information         | Description                              |
| ------------------- | ---------------------------------------- |
| Total Devices       | Number of devices in the Security Fabric |
| Device Type         | FortiGate, FortiAP, FortiSwitch, etc.    |
| Device Name         | Fabric device hostname/name              |
| Device Status       | Current operational status               |
| Registration Status | Registration state                       |
| Firmware Version    | Current FortiOS/Forti firmware           |
| Maturity Level      | Feature or Mature release                |
| Upgrade Status      | Current firmware upgrade state           |

---

# 2. Fabric Firmware Upgrade — Pre-Upgrade Checklist

Before upgrading multiple devices through Security Fabric, perform a structured validation.

### Recommended Pre-Upgrade Workflow

```text
Current Environment
        │
        ▼
Check Fabric Connectivity
        │
        ▼
Verify FortiGuard Connectivity
        │
        ▼
Configure Auto-Update Tunneling
        │
        ▼
Check Upgrade Path
        │
        ▼
Synchronize NTP
        │
        ▼
Verify Licenses
        │
        ▼
Backup Configurations
        │
        ▼
Download Required Firmware Images
        │
        ▼
Test Firmware
        │
        ▼
Upgrade Fabric Devices
        │
        ▼
Update AV / Attack Definitions
        │
        ▼
Post-Upgrade Validation
```

### Before Upgrade

* Enable/configure **Auto-Update Tunneling** where required.
* Verify FortiGuard connectivity.
* Check the supported **upgrade path**.
* Synchronize NTP/time across devices.
* Verify FortiGuard/FortiCare licensing status.
* Back up configurations.
* Download required firmware images before starting.
* Test the target firmware before production deployment.
* Review the target firmware maturity level.
* Plan for device reboot and service interruption.
* For large deployments, consider controlled or staged upgrades.

> ⚠️ **Operational recommendation:** Do not begin a large Fabric upgrade without having the required firmware images and recovery plan available.

---

# 3. FortiOS Firmware Maturity

FortiOS firmware can have different maturity classifications.

| Tag         | Meaning                      | Typical Interpretation              |
| ----------- | ---------------------------- | ----------------------------------- |
| **Feature** | New feature release          | New functionality / newer code      |
| **Mature**  | Maintenance-oriented release | More mature patch/maintenance state |
| **GA**      | General Availability         | Generally available release         |

### GUI Indicators

```text
Feature  → Gray
Mature   → Green
```

### CLI

```bash
get system status
```

Use this to verify:

* FortiOS version
* Build number
* Firmware information
* System status

> **Exam Tip:** Do not confuse **Feature** and **Mature** releases. Maturity indicates the release stage, not simply whether the firmware is "new" or "old."

---

# 4. Upgrade Path

Before upgrading FortiOS:

```text
Current Version
      │
      ▼
Check Supported Upgrade Path
      │
      ▼
Intermediate Version?
      │
 ┌────┴────┐
 │         │
 NO       YES
 │         │
 ▼         ▼
Target    Upgrade
Version   Step-by-Step
```

### Why Upgrade Path Matters?

A direct upgrade to the latest release is not always supported.

Always consider:

* Current FortiOS version
* Target FortiOS version
* Intermediate releases
* Configuration migration
* Feature changes
* Hardware platform
* HA/Fabric dependencies

> 🔎 **Best Practice:** Use Fortinet's upgrade-path information before scheduling a production upgrade.

---

# 5. Configuration Backup

Before firmware upgrades:

```text
BACKUP CONFIGURATION
        │
        ├── Local Backup
        ├── FortiManager Backup
        └── External Backup
```

### Important

Always maintain a recoverable copy of the configuration before:

* Firmware upgrade
* Firmware downgrade
* HA upgrade
* Fabric-wide upgrade
* Major configuration changes

> ⚠️ **Custom Upgrade Warning:** In a scheduled custom upgrade, the configuration backup is created when the administrator schedules the upgrade. If additional configuration changes are made afterward, those later changes may not be included in that scheduled backup.

---

# 6. Antivirus & Attack Definitions

Installing a new firmware image replaces the current antivirus and attack definitions with the definitions included in the firmware package.

Therefore, **after upgrading FortiOS**, verify that security definitions are current.

```bash
execute update-now
```

### Recommended Post-Upgrade Sequence

```text
Firmware Upgrade
      │
      ▼
FortiGate Reboot
      │
      ▼
Verify FortiOS
      │
      ▼
execute update-now
      │
      ▼
Verify AV / Attack Definitions
      │
      ▼
Check Processes & Services
      │
      ▼
Validate Traffic
```

> 🔎 After upgrading, verify system processes and services rather than assuming that a successful firmware installation means the entire system is healthy.

---

# 7. Fabric Management — Firmware Deployment

Navigate to:

```text
System
 └── Fabric Management
```

From Fabric Management, administrators can manage operations such as:

* Device registration
* Device authorization
* Firmware management
* Group upgrades
* Fabric device administration

### Multi-Device Upgrade Concept

```text
                Security Fabric
                      │
          ┌───────────┴───────────┐
          │                       │
      FortiGate                FortiGate
          │                       │
      FortiSwitch              FortiAP
          │
      Upgrade Group
          │
          ▼
   Coordinated Upgrade
```

---

# 8. Auto-Update Tunneling

Before using certain Fabric firmware-management workflows, verify that the required **FortiGuard Auto-Update Tunnel** functionality is configured.

Conceptually:

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
Firmware / Signature Services
```

### Verify

* FortiGuard connectivity
* Licensing
* DNS
* Routing
* NTP
* Auto-update tunnel configuration

---

# 9. Federated Upgrade

FortiOS provides a mechanism for coordinated firmware upgrades.

### Initialize

```bash
execute federated-upgrade initialize
```

Possible operations include:

```text
initialize
status
cancel
```

### Concept

```text
               Fabric / HA Environment
                       │
                       ▼
              Federated Upgrade
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       Device A      Device B     Device C
          │            │            │
          ▼            ▼            ▼
       Upgrade       Upgrade      Upgrade
```

### Important

The federated upgrade process can coordinate devices through an upgrade sequence rather than treating every device as an isolated upgrade.

> ⚠️ **Important:** FortiAP and FortiSwitch devices do not currently follow the same FortiGate upgrade path behavior; they upgrade directly to the target version.

---

# 10. Firmware Upgrade Warning

To enable firmware upgrade warning prompts:

```bash
config system global
    set gui-firmware-upgrade-warning enable
end
```

This can help prevent accidental firmware upgrades through the GUI.

---

# 11. Manual Firmware Upgrade — TFTP

A FortiGate can retrieve a firmware image from a TFTP server.

### Step 1 — Prepare TFTP Server

Copy the firmware image to the **root directory** of the TFTP server.

Example:

```text
TFTP Root
└── a.pkg
```

### Step 2 — Test Connectivity

```bash
execute ping 192.168.20.200
```

### Step 3 — Upgrade

```bash
execute restore image tftp a.pkg 192.168.20.200
```

Where:

```text
a.pkg                  → Firmware image filename
192.168.20.200         → TFTP server IP
```

### Warning

The command replaces the current firmware.

You will see a confirmation similar to:

```text
This operation will replace the current firmware version!
Do you want to continue? (y/n)
```

Enter:

```text
y
```

The FortiGate then:

1. Downloads the firmware.
2. Verifies the firmware signature.
3. Checks firmware integrity.
4. Installs the image.
5. Reboots.
6. Starts the new FortiOS version.

---

# 12. TFTP Upgrade — Expected Flow

```text
execute ping 192.168.20.200
             │
             ▼
TFTP Connectivity OK
             │
             ▼
execute restore image tftp ...
             │
             ▼
Download Image
             │
             ▼
Verify Signature
             │
             ▼
Verify Integrity
             │
             ▼
Install Firmware
             │
             ▼
Reboot
             │
             ▼
New FortiOS
```

### Typical Messages

```text
Get image from tftp server OK.
Verifying the signature of the firmware image.
Checking new firmware integrity ... pass
Please wait for system to restart.
Firmware upgrade in progress ...
Done.
The system is going down NOW !!
```

---

# 13. Feature Firmware Warning

When installing a firmware image classified as **Feature**, FortiGate may display a warning such as:

```text
Warning: Upgrading to an image with Feature maturity notation.
Image file uploaded is marked as a Feature image,
are you sure you want to upgrade?
```

This is an important operational signal.

### Decision Point

```text
Target Image
     │
     ▼
Feature Image?
     │
 ┌───┴───┐
 │       │
 YES     NO
 │       │
 ▼       ▼
Review   Continue
Risk
```

---

# 14. Update Security Definitions After Upgrade

After firmware installation:

```bash
execute update-now
```

Use this to trigger updates for FortiGuard-related security definitions and restart/update applicable services as required.

### Validation

```text
Firmware
   │
   ├── FortiOS Version
   ├── AV Definitions
   ├── Attack Definitions
   ├── FortiGuard Connectivity
   └── Running Services
```

---

# 15. Controlled Firmware Upgrade

Controlled upgrades allow a firmware image to be loaded into another firmware partition **before** the actual reboot/activation.

This is useful for:

* Staged upgrades
* Maintenance windows
* Multiple FortiGate deployments
* Script-based upgrades
* FortiManager-controlled workflows

### Load Secondary Image

```bash
execute restore secondary-image tftp a.pkg
```

The image can be staged for later use.

### Select Next Boot Image

```bash
execute set-next-reboot primary
```

Possible partition choices include:

```text
primary
secondary
```

### Concept

```text
Current Running Image
        │
        │
        ▼
┌───────────────────────┐
│ FortiGate Storage     │
│                       │
│ Primary    → Current  │
│ Secondary  → New OS   │
└───────────────────────┘
        │
        ▼
Set Next Reboot
        │
        ▼
Reboot
        │
        ▼
Selected Firmware
```

> 💡 **Use Case:** Firmware can be staged in advance, allowing the actual activation to occur during a controlled maintenance window.

---

# 16. Multistage Upgrade Strategy

For large environments:

```text
Stage 1
  │
  ├── Download firmware
  ├── Validate signature
  └── Stage image
          │
          ▼
Stage 2
  │
  ├── Test
  └── Validate
          │
          ▼
Stage 3
  │
  ├── Upgrade pilot device
  └── Monitor
          │
          ▼
Stage 4
  │
  └── Upgrade remaining devices
```

### Recommended Strategy

```text
LAB
 ↓
PILOT
 ↓
LOW-RISK DEVICES
 ↓
PRODUCTION
 ↓
CRITICAL SYSTEMS
```

---

# 17. Firmware Image Signing & Verification

FortiOS firmware images are digitally signed.

The purpose is to help verify that the firmware image is authentic and has not been improperly modified.

### High-Level Verification

```text
Firmware Image
      │
      ▼
Signature Attached
      │
      ▼
FortiGate Verification
      │
      ▼
Signature Valid?
    ┌─┴─┐
   YES  NO
    │    │
    ▼    ▼
Install  Warning / Reject
```

### Key Concept

The firmware image contains a signature associated with the code.

During firmware handling, FortiGate verifies the image.

If the signature does not validate, the device can warn that the firmware authenticity cannot be verified.

---

# 18. Firmware Signature Warning

A manually uploaded image that fails signature verification may generate a warning similar to:

```text
Fortinet cannot verify the authenticity of this firmware
and therefore there may be a risk that the firmware contains
code unknown to Fortinet.
```

### Security Rule

> 🚨 **Never ignore firmware authenticity warnings in production.**

Verify:

* Firmware source
* Model compatibility
* File integrity
* Firmware build
* Signature status

---

# 19. Firmware Verification Without Permanent Upgrade

A useful recovery/verification workflow is to boot a firmware image **without making it the permanent default firmware**.

### Why?

You may want to verify:

* OS boots correctly
* Hardware compatibility
* Firmware functionality
* Basic system operation

without immediately replacing the existing default image.

---

# 20. Console-Based Firmware Verification

### Required

* Console cable
* RJ-45-to-USB, DB-9, or null-modem connection as applicable
* TFTP server
* Firmware image

### Preparation

1. Start the TFTP server.
2. Copy firmware image to TFTP root.
3. Connect console to FortiGate.
4. Verify TFTP connectivity.
5. Reboot FortiGate.

```bash
execute ping 192.168.20.200
```

Then:

```bash
execute reboot
```

Confirm:

```text
This operation will reboot the system!
Do you want to continue? (y/n)
```

Enter:

```text
y
```

---

# 21. Interrupt FortiGate Boot Process

During startup, FortiGate displays:

```text
Press any key to display configuration menu ..
```

### Critical Timing

You have only a short time to interrupt the boot process.

```text
Boot
 │
 ▼
Press any key
 │
 ├── Key pressed → Boot Menu
 │
 └── No key → Normal boot
```

If you miss the window:

```text
Normal Boot
   │
   ▼
execute reboot
   │
   ▼
Try Again
```

---

# 22. FortiGate Boot Menu

A typical boot menu provides options such as:

```text
[g]: Get firmware image from TFTP server.
[f]: Format boot device.
[b]: Boot with backup firmware and set as default.
[c]: Configuration and information.
[q]: Quit menu and continue to boot with default firmware.
[h]: Display this list of options.
```

### Key Options

| Option | Function                                 |
| ------ | ---------------------------------------- |
| `g`    | Retrieve firmware from TFTP              |
| `f`    | Format boot device                       |
| `b`    | Boot backup firmware and make it default |
| `c`    | Configure / display information          |
| `q`    | Continue normal boot                     |
| `h`    | Help                                     |

> ⚠️ **Danger:** `f` can format the boot device. Do not use it casually in production.

---

# 23. Boot Firmware From TFTP — Verification Mode

Select:

```text
g
```

Then enter the TFTP server address:

```text
Enter TFTP server address [192.168.20.200]:
```

Enter the FortiGate local IP when requested:

```text
Enter local address [192.168.20.200]:
```

Then provide the firmware filename:

```text
Enter file name [image.out]:
```

---

# 24. Firmware Save Options

After downloading the firmware, FortiGate can provide options similar to:

```text
Save as default firmware / backup firmware / run image without saving:
[d/b/r]
```

### Meaning

| Option | Meaning                                        |
| ------ | ---------------------------------------------- |
| `d`    | Save as default firmware                       |
| `b`    | Save as backup firmware                        |
| `r`    | Run image without saving as permanent firmware |

### Verification

For testing only:

```text
r
```

This allows the image to be run without making it the permanent default firmware.

### Key Concept

```text
Run Test Image
      │
      ▼
Validate OS
      │
      ▼
Existing Default Remains
      │
      ▼
Permanent Upgrade Not Performed
```

> 💡 This is useful when you want to verify an image before committing to a firmware replacement.

---

# 25. Installing Firmware Through Boot Menu

If the firmware has been verified and you want to install it:

```text
Reboot
  │
  ▼
Interrupt Boot
  │
  ▼
Configure TFTP
  │
  ▼
Download Image
  │
  ▼
Select Save Option
  │
  ▼
Install
  │
  ▼
Restart
```

---

# 26. TFTP Configuration Menu

If required, select:

```text
c
```

The configuration menu can contain options such as:

```text
[p]: Set firmware download port.
[d]: Set DHCP mode.
[i]: Set local IP address.
[s]: Set local subnet mask.
[g]: Set local gateway.
[v]: Set local VLAN ID.
[t]: Set remote TFTP server IP address.
[f]: Set firmware file name.
[e]: Reset TFTP parameters to factory defaults.
[r]: Review TFTP parameters.
[n]: Diagnose networking (ping).
[q]: Quit this menu.
[h]: Display this list of options.
```

### Practical Workflow

```text
c
 │
 ├── p → Download Port
 ├── i → Local IP
 ├── s → Subnet Mask
 ├── g → Gateway
 ├── v → VLAN ID
 ├── t → TFTP Server
 ├── f → Firmware File
 └── n → Network Test
```

---

# 27. USB Firmware Restore

Firmware can also be restored from a USB drive.

### Preparation

1. Copy firmware image to the USB root directory.
2. Connect USB to FortiGate.
3. Connect to CLI/console.
4. Execute restore command.

```bash
execute restore image usb <filename>
```

Example:

```bash
execute restore image usb a.pkg
```

FortiGate warns:

```text
This operation will replace the current firmware version!
Do you want to continue? (y/n)
```

Enter:

```text
y
```

The FortiGate:

```text
Restore Image
     │
     ▼
Install Firmware
     │
     ▼
Restart
```

---

# 28. Post-USB Upgrade

After the firmware restore:

```bash
execute update-now
```

Then verify:

```bash
get system status
```

Check:

* FortiOS version
* Build
* Device model
* System status
* FortiGuard connectivity
* Security definitions

---

# 29. USB Auto-Detection

FortiGate can be configured to detect a firmware image on USB during startup.

Navigate to:

```text
System
 └── Settings
      └── Startup Settings
```

Enable:

```text
Detect Firmware
```

and specify the firmware filename.

Then:

1. Copy firmware to USB root.
2. Connect USB.
3. Reboot FortiGate.

> ⚠️ **Operational recommendation:** Avoid leaving automatic firmware detection enabled unnecessarily. Disable it after temporary maintenance/testing workflows when it is no longer required.

---

# 30. Firmware Downgrade

Downgrading firmware should be treated as a **high-risk operation**.

> ⚠️ **Recommendation:** Do not downgrade unless there is a validated operational reason and the downgrade path is supported.

### TFTP Downgrade

Prepare:

```text
TFTP Server
└── Old FortiOS Image
```

Test connectivity:

```bash
execute ping 192.168.20.200
```

Then:

```bash
execute restore image tftp a.pkg 192.168.20.200
```

FortiGate will indicate:

```text
This operation will replace the current firmware version!
```

Confirm:

```text
y
```

If the image represents a downgrade, FortiGate can display a warning similar to:

```text
This operation will downgrade the current firmware version!
```

Confirm again:

```text
y
```

---

# 31. Downgrade Workflow

```text
Current FortiOS
      │
      ▼
Backup Configuration
      │
      ▼
Verify Downgrade Path
      │
      ▼
Prepare Old Firmware
      │
      ▼
Test TFTP
      │
      ▼
restore image tftp
      │
      ▼
Downgrade Warning
      │
      ▼
Confirm
      │
      ▼
Reboot
      │
      ▼
execute update-now
      │
      ▼
Validate
```

### After Downgrade

Run:

```bash
execute update-now
```

Then verify:

```bash
get system status
```

> ⚠️ Firmware downgrades can involve configuration migration/reversion issues. Always verify the configuration and supported downgrade path before proceeding.

---

# 32. Upgrade vs Controlled Upgrade

| Method            | Firmware Location         | Activation          | Best Use                |
| ----------------- | ------------------------- | ------------------- | ----------------------- |
| GUI Upgrade       | FortiGate                 | Immediate/scheduled | Normal upgrades         |
| TFTP Restore      | TFTP server               | Immediate           | Recovery/manual upgrade |
| USB Restore       | USB                       | Immediate           | Offline/recovery        |
| Secondary Image   | Local secondary partition | Next reboot         | Controlled staging      |
| Fabric Management | Security Fabric           | Coordinated         | Multiple devices        |
| Federated Upgrade | Fabric/upgrade workflow   | Coordinated         | Multi-device upgrade    |

---

# 33. Production Upgrade Golden Rules

### Before

```text
☑ Backup configuration
☑ Verify upgrade path
☑ Download firmware
☑ Verify firmware authenticity
☑ Check device compatibility
☑ Test firmware
☑ Verify FortiGuard
☑ Verify licenses
☑ Synchronize NTP
☑ Review HA/Fabric dependencies
☑ Prepare rollback/recovery plan
```

### During

```text
☑ Monitor device status
☑ Do not interrupt power
☑ Do not interrupt firmware transfer
☑ Monitor HA/Fabric state
☑ Monitor connectivity
☑ Track reboot progress
```

### After

```text
☑ get system status
☑ execute update-now
☑ Verify AV definitions
☑ Verify attack definitions
☑ Verify interfaces
☑ Verify routing
☑ Verify policies
☑ Verify VPNs
☑ Verify HA
☑ Verify Security Fabric
☑ Verify logs
☑ Verify critical services
```

---

# 34. Troubleshooting Decision Tree

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
   │     Boot Menu
   │         │
   │         ▼
   │      TFTP/USB
   │         │
   └────┬────┘
        ▼
Verify Firmware
        │
        ▼
Check Signature
        │
        ▼
Check Integrity
        │
        ▼
Check Upgrade Path
        │
        ▼
Restore / Retry
```

---

# 35. High-Value CLI Reference

| Purpose                      | Command                                       |
| ---------------------------- | --------------------------------------------- |
| Check system status          | `get system status`                           |
| Test connectivity            | `execute ping <IP>`                           |
| Update security definitions  | `execute update-now`                          |
| Reboot                       | `execute reboot`                              |
| TFTP firmware restore        | `execute restore image tftp <file> <server>`  |
| USB firmware restore         | `execute restore image usb <file>`            |
| Stage secondary firmware     | `execute restore secondary-image tftp <file>` |
| Select next reboot image     | `execute set-next-reboot primary`             |
| Initialize federated upgrade | `execute federated-upgrade initialize`        |

---

# 36. Example — Complete TFTP Upgrade

```bash
# Verify connectivity
execute ping 192.168.20.200

# Upgrade FortiOS
execute restore image tftp a.pkg 192.168.20.200

# Confirm prompts
y
y

# After reboot
get system status

# Update security definitions
execute update-now
```

---

# 37. Example — Controlled Upgrade

```bash
# Stage firmware in secondary partition
execute restore secondary-image tftp a.pkg

# Select partition for next reboot
execute set-next-reboot primary

# Reboot during maintenance window
execute reboot
```

> **Note:** The exact partition state and command behavior should be validated against the specific FortiOS release and hardware platform before production use.

---

# 38. Security & Reliability Checklist

### 🔐 Security

* [ ] Obtain firmware from trusted Fortinet sources.
* [ ] Verify firmware authenticity/signature.
* [ ] Do not ignore signature warnings.
* [ ] Restrict access to the TFTP server.
* [ ] Protect USB firmware media.
* [ ] Protect configuration backups.

### 🛡️ Reliability

* [ ] Maintain console access.
* [ ] Maintain out-of-band management where possible.
* [ ] Verify HA state before upgrade.
* [ ] Verify Fabric topology.
* [ ] Synchronize NTP.
* [ ] Verify licenses.
* [ ] Maintain recovery firmware.
* [ ] Keep required firmware images locally available.

### 📦 Recovery

* [ ] Configuration backup available.
* [ ] Previous firmware available.
* [ ] TFTP server ready.
* [ ] USB recovery media ready.
* [ ] Console cable available.
* [ ] Maintenance window approved.

---

# 39. Expert Notes

> **1. Never treat "firmware downloaded successfully" as "upgrade completed successfully."**
> Installation, reboot, service health, FortiGuard connectivity, security definitions, HA state, and traffic forwarding should all be validated.

> **2. Pre-stage firmware when the maintenance window is critical.**
> Controlled upgrades can reduce the amount of work that must happen during the actual maintenance window.

> **3. Firmware availability is part of disaster recovery.**
> For large Fabric environments, having only an internet-based firmware workflow can create unnecessary operational risk during an outage.

> **4. Time synchronization matters.**
> NTP should be healthy before authentication, FortiToken/MFA, certificates, FortiGuard services, and coordinated upgrade operations.

> **5. Backup is not optional.**
> Always know exactly where the rollback configuration and firmware are before changing the operating system.

> **6. Test before production.**
> A firmware image should be validated against the actual hardware model, configuration, HA design, Security Fabric topology, and critical services.

---

# 40. Exam Quick Review 🧠

```text
Fabric Management
    ↓
Manage FortiGate / FortiAP / FortiSwitch
    ↓
Registration + Authorization + Firmware
```

```text
Feature
    ↓
New functionality
    ↓
Gray
```

```text
Mature
    ↓
Maintenance / mature release
    ↓
Green
```

```text
TFTP Upgrade
    ↓
execute ping
    ↓
execute restore image tftp
    ↓
Verify Signature
    ↓
Install
    ↓
Reboot
    ↓
execute update-now
```

```text
Controlled Upgrade
    ↓
restore secondary-image
    ↓
Stage firmware
    ↓
set-next-reboot
    ↓
Reboot
```

```text
Firmware Verification
    ↓
Reboot
    ↓
Interrupt Boot
    ↓
g
    ↓
TFTP
    ↓
r
    ↓
Run Without Saving
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
Check Traffic
    ↓
Check HA / Fabric
```

---

## 🔑 Keywords

`FortiOS 7.2.0` · `FortiGate Firmware Upgrade` · `FortiGate Upgrade` · `FortiOS Upgrade Path` · `Security Fabric` · `Fabric Management` · `FortiGate Federated Upgrade` · `FortiGate TFTP Upgrade` · `FortiGate USB Firmware Upgrade` · `FortiOS Controlled Upgrade` · `FortiGate Firmware Verification` · `FortiOS Firmware Signature` · `FortiGuard Update` · `execute update-now` · `execute restore image tftp` · `execute restore secondary-image` · `FortiGate Downgrade` · `FortiGate Recovery` · `FortiGate Boot Menu`

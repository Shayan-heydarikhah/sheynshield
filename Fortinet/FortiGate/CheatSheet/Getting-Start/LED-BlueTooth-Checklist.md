# FortiGate LED & Hardware Indicators — Troubleshooting Checklist

> **SheynShield | Engineering Secure Networks**
>
> A practical **FortiGate LED & Hardware Troubleshooting Checklist** for **NSE4 / NSE7**, field engineers, network administrators, and FortiGate troubleshooting.
>
> Covers **STATUS, POWER, HA, ALARM, BYPASS, PoE, FAN, Ethernet, SFP/SFP+/SFP28/QSFP28, PSU, IPMI, BLE, and hardware diagnostics**.

---

## 📌 Quick Navigation

* [ ] [LED Severity](#-1-led-alarm-severity)
* [ ] [LOGO](#-2-logo-led)
* [ ] [PWR](#-3-power--pwr)
* [ ] [STA](#-4-status--sta)
* [ ] [BYP](#-5-bypass--byp)
* [ ] [ALARM](#-6-alarm-led)
* [ ] [HA](#-7-ha-led)
* [ ] [PoE](#-8-max-poe)
* [ ] [SVC](#-10-svc-led)
* [ ] [3G/4G](#-11-3g--4g-led)
* [ ] [Wi-Fi](#-12-wi-fi-led)
* [ ] [PSU](#-13-power-supply-led)
* [ ] [Fan](#-18-fan-led)
* [ ] [Ethernet](#-19-ethernet-leds)
* [ ] [SFP Family](#-22-sfp)
* [ ] [Hardware Diagnostics](#-27-hardware-troubleshooting-commands)
* [ ] [IPMI](#-28-ipmi)
* [ ] [BLE](#-29-bluetooth--ble-diagnostics)
* [ ] [Troubleshooting Workflows](#-30-led-troubleshooting)
* [ ] [NSE Exam Memory Map](#-nse-exam-memory-map)
* [ ] [Exam Traps](#-exam--real-world-traps)

---

# 1. LED Alarm Severity

Use the LED color as the **first hardware-health signal**, not as the final diagnosis.

| Severity        | Meaning                                        | Action                          |
| --------------- | ---------------------------------------------- | ------------------------------- |
| 🟢 **Normal**   | Device operating normally                      | [ ] Continue monitoring         |
| 🟠 **Minor**    | Warning / non-critical condition               | [ ] Investigate                 |
| 🔴 **Major**    | Serious condition affecting operation          | [ ] Troubleshoot immediately    |
| 🔴 **Critical** | Severe / potentially non-recoverable condition | [ ] Escalate / inspect hardware |

### Troubleshooting Checklist

* [ ] Identify the LED.
* [ ] Identify steady vs flashing state.
* [ ] Identify the FortiGate model.
* [ ] Check system/event logs.
* [ ] Correlate LED state with FortiOS state.
* [ ] Check hardware-specific documentation.

> ⚠️ **Important:** LED behavior is **model dependent**. Never assume that the same LED behavior applies to every FortiGate platform.

---

# 2. LOGO LED

| LED      | Meaning      | Check                 |
| -------- | ------------ | --------------------- |
| 🟢 Green | Unit ON      | [ ] Normal            |
| 🔵 Blue  | FortiWiFi ON | [ ] Wireless platform |
| ⚫ Off    | Device OFF   | [ ] Power             |

### Checklist

* [ ] Confirm device has power.
* [ ] Confirm the platform is FortiGate vs FortiWiFi.
* [ ] Check PWR/PSU indicators.
* [ ] Check boot state if the unit is starting.

---

# 3. Power — PWR

| LED               | Meaning                              |
| ----------------- | ------------------------------------ |
| 🟢 Green          | Unit ON / power supplies functioning |
| 🟠 Amber          | One power supply functioning         |
| 🟠 Flashing Amber | Power-supply failure                 |
| 🔴 Red            | Unit ON but only one PSU functional  |
| 🔴 Flashing Red   | Power failure                        |
| ⚫ Off             | Unit OFF                             |

### PWR Troubleshooting Checklist

* [ ] Check PWR LED.
* [ ] Check individual PSU LEDs if available.
* [ ] Check PSU INPUT.
* [ ] Check PSU OUTPUT.
* [ ] Check PSU FAIL.
* [ ] Check AC power.
* [ ] Check power cable.
* [ ] Check redundant PSU.
* [ ] Check hardware alarms.

> 🔥 **Exam Point:** `PWR` does not necessarily represent the health of every individual PSU. On redundant-power platforms, inspect the dedicated PSU indicators.

---

# 4. Status — STA

| LED               | Meaning                |
| ----------------- | ---------------------- |
| 🟢 Green          | Normal operation       |
| 🟢 Flashing Green | Booting                |
| 🟠 Amber          | Major/minor alarm      |
| 🟠 Flashing Amber | BLE ON                 |
| 🔴 Red            | Major alarm            |
| 🔴 Flashing Red   | BLE ON / pairing state |
| ⚫ Off             | Unit OFF               |

### Checklist

* [ ] Check whether the unit is booting.
* [ ] Check for major/minor alarms.
* [ ] Check BLE state if applicable.
* [ ] Check system event logs.
* [ ] Correlate with PWR and ALARM LEDs.

---

# 5. Bypass — BYP

> ⚠️ **Model dependent.**

| LED      | Meaning                      |
| -------- | ---------------------------- |
| 🟠 Amber | Bypass active / Fail Open    |
| ⚫ Off    | Bypass inactive / Fail Close |

### Fast Recall

```text
BYP Amber
    ↓
Bypass ACTIVE
    ↓
Fail Open
```

```text
BYP Off
    ↓
Bypass INACTIVE
    ↓
Fail Close
```

### Checklist

* [ ] Confirm the model supports hardware bypass.
* [ ] Check BYP LED.
* [ ] Identify bypass port pair.
* [ ] Determine Fail Open vs Fail Close.
* [ ] Investigate why bypass was activated.

---

# 6. ALARM LED

| LED      | Meaning     |
| -------- | ----------- |
| 🔴 Red   | Major alarm |
| 🟠 Amber | Minor alarm |
| ⚫ Off    | No alarm    |

### Fast Recall

```text
RED
 ↓
Major

AMBER
 ↓
Minor

OFF
 ↓
No Alarm
```

### Checklist

* [ ] Identify alarm severity.
* [ ] Check system logs.
* [ ] Check event logs.
* [ ] Check hardware health.
* [ ] Check PSU/FAN/temperature state.

---

# 7. HA LED

| LED               | Meaning                 |
| ----------------- | ----------------------- |
| 🟢 Green          | Operating in HA cluster |
| 🟠 Amber / 🔴 Red | HA failover condition   |
| ⚫ Off             | HA disabled             |

### HA Troubleshooting Checklist

* [ ] Check HA LED.
* [ ] Check cluster status.
* [ ] Check cluster members.
* [ ] Check heartbeat interfaces.
* [ ] Check HA synchronization.
* [ ] Check member state.
* [ ] Check failover events.
* [ ] Check system/event logs.

### Mental Model

```text
HA LED abnormal
      ↓
Check HA status
      ↓
Check members
      ↓
Check heartbeat
      ↓
Check synchronization
      ↓
Check failover events
      ↓
Check logs
```

---

# 8. MAX POE

Indicates the PoE power-budget condition.

| LED      | Meaning                                                |
| -------- | ------------------------------------------------------ |
| 🟢 Green | Maximum PoE power allocated                            |
| 🟠 Amber | PoE usage higher than normal                           |
| 🔴 Red   | Power cannot be delivered although device is connected |
| ⚫ Off    | PoE power available / normal                           |

### Checklist

* [ ] Check PoE budget.
* [ ] Check connected powered devices.
* [ ] Check total wattage.
* [ ] Check MAX POE LED.
* [ ] Check individual PoE ports.
* [ ] Investigate overload condition.

### Fast Recall

```text
Amber → High PoE consumption
Red   → Cannot deliver required PoE power
```

---

# 9. PoE LED

| LED               | Meaning                             |
| ----------------- | ----------------------------------- |
| 🟢 Green          | Power being delivered               |
| 🟢 Flashing Green | Error / PoE device requesting power |
| ⚫ Off             | No PoE device / no power delivered  |

### Checklist

* [ ] Confirm PoE is enabled.
* [ ] Check PoE LED.
* [ ] Check connected device.
* [ ] Check power budget.
* [ ] Check for PoE errors.
* [ ] Check cable and physical connection.

---

# 10. SVC LED

| LED               | Meaning      |
| ----------------- | ------------ |
| 🟢 Green          | SVC ON       |
| 🟢 Flashing Green | SVC activity |
| ⚫ Off             | SVC OFF      |

### Checklist

* [ ] Identify SVC function on the platform.
* [ ] Check LED state.
* [ ] Check service state.
* [ ] Check logs if abnormal.

---

# 11. 3G / 4G LED

| LED               | Meaning           |
| ----------------- | ----------------- |
| 🟢 Green          | 3G/4G service ON  |
| 🟢 Flashing Green | 3G/4G activity    |
| ⚫ Off             | 3G/4G service OFF |

### Checklist

* [ ] Confirm cellular hardware exists.
* [ ] Check service state.
* [ ] Check activity.
* [ ] Check modem status.
* [ ] Check SIM/connectivity if applicable.

---

# 12. Wi-Fi LED

| LED               | Meaning         |
| ----------------- | --------------- |
| 🟢 Green          | Wi-Fi connected |
| 🟢 Flashing Green | Wi-Fi activity  |
| ⚫ Off             | Wi-Fi OFF       |

> ⚠️ Primarily applicable to **FortiWiFi / wireless-capable platforms**.

### Checklist

* [ ] Confirm FortiWiFi platform.
* [ ] Check Wi-Fi state.
* [ ] Check client connectivity.
* [ ] Check wireless activity.

---

# 13. Power Supply LED

For platforms with dedicated PSU indicators:

| LED               | Meaning                                                  |
| ----------------- | -------------------------------------------------------- |
| 🟢 Green          | PSU operating normally                                   |
| 🟢 Flashing Green | Power detected, PSU not providing output / standby       |
| 🟠 Amber          | Output OFF / PSU error / no input while redundant PSU ON |
| 🟠 Flashing Amber | PSU warning/error                                        |
| 🔴 Red            | Power cord unplugged / power lost                        |
| 🔴 Flashing Red   | PSU warning event                                        |
| ⚫ Off             | Power not detected                                       |

### PSU Checklist

* [ ] Check PSU status.
* [ ] Check input power.
* [ ] Check output power.
* [ ] Check redundancy.
* [ ] Check PSU fan.
* [ ] Check hardware alarm.
* [ ] Replace PSU if required.

---

# 14. Power Supply OK

| LED               | Meaning                          |
| ----------------- | -------------------------------- |
| 🟢 Green          | Standby rail + main output ON    |
| 🟢 Flashing Green | Standby rail ON, main output OFF |
| ⚫ Off             | Error / no AC input              |

### Checklist

* [ ] Check standby rail.
* [ ] Check main output.
* [ ] Check AC input.
* [ ] Check PSU FAIL indicator.

---

# 15. Power Supply FAIL

| LED               | Meaning               |
| ----------------- | --------------------- |
| 🟠 Amber          | Main output/fan error |
| 🟠 Flashing Amber | PSU warning           |
| ⚫ Off             | No error / no power   |

### Checklist

* [ ] Check FAIL LED.
* [ ] Check PSU output.
* [ ] Check PSU fan.
* [ ] Check temperature.
* [ ] Check hardware logs.
* [ ] Replace PSU if required.

---

# 16. Power Supply INPUT

| LED               | Meaning                    |
| ----------------- | -------------------------- |
| 🟢 Green          | Input voltage normal       |
| 🟢 Flashing Green | Over/under-voltage warning |
| ⚫ Off             | No input power             |

### Checklist

* [ ] Verify AC input.
* [ ] Check input LED.
* [ ] Investigate voltage warning.
* [ ] Check power source/circuit.

---

# 17. Power Supply OUTPUT

| LED               | Meaning               |
| ----------------- | --------------------- |
| 🟢 Green          | Output voltage normal |
| 🟢 Flashing Green | Standby mode          |
| 🟠 Amber          | Critical error        |
| 🟠 Flashing Amber | Warning               |
| ⚫ Off             | No output             |

### Checklist

* [ ] Check output state.
* [ ] Check standby state.
* [ ] Check critical error.
* [ ] Check warning.
* [ ] Correlate with PSU FAIL.

---

# 18. Fan LED

| LED               | Meaning                        |
| ----------------- | ------------------------------ |
| 🟢 Green          | Fan(s) operating normally      |
| 🟢 Flashing Green | Fan switching / initialization |
| 🟠 Amber          | Fan failure                    |
| 🔴 Red            | Fan error / abnormal RPM       |
| 🔴 Flashing Red   | One fan set has an alert       |
| ⚫ Off             | Fan error / fan OFF            |

### Fan Troubleshooting Checklist

* [ ] Check FAN LED.
* [ ] Check fan RPM.
* [ ] Check fan sets.
* [ ] Check temperature.
* [ ] Check airflow.
* [ ] Check physical obstruction.
* [ ] Check system/event logs.
* [ ] Replace failed fan if required.

### Mental Model

```text
GREEN
  ↓
Normal

AMBER
  ↓
Fan Failure

RED
  ↓
Serious Fan/RPM Condition

FLASHING RED
  ↓
Fan Set Alert
```

---

# 19. Ethernet LEDs

> ⚠️ Ethernet LED behavior varies by hardware generation.

## Link + Speed

| LED               | Meaning              |
| ----------------- | -------------------- |
| 🟢 Green          | 1 Gbps               |
| 🟢 Flashing Green | TX/RX at 1 Gbps      |
| 🟠 Amber          | 10/100 Mbps          |
| 🟠 Flashing Amber | TX/RX at 10/100 Mbps |
| ⚫ Off             | No link              |

## Link / Activity

| LED               | Meaning           |
| ----------------- | ----------------- |
| 🟢 Green          | Link established  |
| 🟢 Flashing Green | Data transmission |
| ⚫ Off             | No link           |

## Speed

| LED      | Speed                   |
| -------- | ----------------------- |
| 🟢 Green | 1 Gbps                  |
| 🟠 Amber | 100 Mbps                |
| ⚫ Off    | Not connected / 10 Mbps |

### Ethernet Troubleshooting Checklist

* [ ] Check LED state.
* [ ] Determine link state.
* [ ] Determine speed.
* [ ] Check activity.
* [ ] Check cable.
* [ ] Check peer device.
* [ ] Check interface configuration.
* [ ] Check interface errors.

---

# 20. 10G Ethernet

## Link / Activity

| LED               | Meaning           |
| ----------------- | ----------------- |
| 🟢 Green          | Link established  |
| 🟢 Flashing Green | Data transmission |
| ⚫ Off             | No link           |

## Speed

| LED      | Speed                    |
| -------- | ------------------------ |
| 🟢 Green | 10 Gbps                  |
| 🟠 Amber | 5 / 2.5 / 1 Gbps         |
| ⚫ Off    | Not connected / 100 Mbps |

### Checklist

* [ ] Verify transceiver/cable.
* [ ] Verify peer speed.
* [ ] Check autonegotiation where applicable.
* [ ] Check interface status.
* [ ] Check errors and drops.

---

# 21. Ethernet PoE

| LED      | Meaning                          |
| -------- | -------------------------------- |
| 🟢 Green | PoE ON / device receiving power  |
| 🟠 Amber | Port providing power             |
| 🔴 Red   | Device connected but not powered |
| ⚫ Off    | PoE OFF / no powered device      |

### Checklist

* [ ] Confirm PoE enabled.
* [ ] Check power delivery.
* [ ] Check device compatibility.
* [ ] Check total PoE budget.
* [ ] Check cable.
* [ ] Check port state.

---

# 22. SFP

| LED               | Meaning       |
| ----------------- | ------------- |
| 🟢 Green          | 1 Gbps        |
| 🟢 Flashing Green | Data activity |
| ⚫ Off             | No link       |

### Checklist

* [ ] Check SFP module.
* [ ] Check fiber/cable.
* [ ] Check peer.
* [ ] Check link.
* [ ] Check activity.
* [ ] Verify supported transceiver.

---

# 23. SFP+

| LED               | Meaning       |
| ----------------- | ------------- |
| 🟢 Green          | 10 / 1 Gbps   |
| 🟢 Flashing Green | Data activity |
| ⚫ Off             | No link       |

### Checklist

* [ ] Check transceiver.
* [ ] Check fiber/DAC.
* [ ] Check peer.
* [ ] Verify negotiated/configured speed.
* [ ] Check interface errors.

---

# 24. SFP28

| LED               | Meaning          |
| ----------------- | ---------------- |
| 🟢 Green          | 25 / 10 / 1 Gbps |
| 🟢 Flashing Green | Data activity    |
| ⚫ Off             | No link          |

### Checklist

* [ ] Verify SFP28 compatibility.
* [ ] Verify supported speed.
* [ ] Check transceiver.
* [ ] Check fiber/DAC.
* [ ] Check peer configuration.

---

# 25. QSFP28

| LED               | Meaning       |
| ----------------- | ------------- |
| 🟢 Green          | 100 / 40 Gbps |
| 🟢 Flashing Green | Data activity |
| ⚫ Off             | No link       |

### Checklist

* [ ] Verify QSFP28 module.
* [ ] Verify 40/100G compatibility.
* [ ] Check breakout configuration if applicable.
* [ ] Check peer.
* [ ] Check optical/cable condition.

---

# 26. SFP Family — Fast Memorization

```text
SFP
 └── 1G

SFP+
 └── 10G / 1G

SFP28
 └── 25G / 10G / 1G

QSFP28
 └── 100G / 40G
```

> ⚠️ **Important:** Actual supported speeds depend on the **FortiGate model, hardware generation, transceiver, cable, and port configuration**.

### Exam Checklist

* [ ] Know SFP.
* [ ] Know SFP+.
* [ ] Know SFP28.
* [ ] Know QSFP28.
* [ ] Remember that port capability is model dependent.

---

# 27. Hardware Troubleshooting Commands

## Linux Bridge / Netlink

```bash
diagnose netlink brctl name host root.b
```

### Checklist

* [ ] Identify whether the issue involves bridge behavior.
* [ ] Run the diagnostic command where applicable.
* [ ] Correlate output with interface/bridge state.
* [ ] Check logs and configuration.

---

# 28. IPMI — Hardware Management

**IPMI (Intelligent Platform Management Interface)** provides hardware-level management and monitoring capabilities.

Common monitored conditions include:

* [ ] Temperature
* [ ] Fan status
* [ ] Power
* [ ] Voltage
* [ ] Hardware health
* [ ] Environmental conditions

### Alarm Classification

| Condition                | Classification |
| ------------------------ | -------------- |
| Non-critical             | Minor          |
| Critical but recoverable | Major          |
| Non-recoverable          | Critical       |

### Mental Model

```text
IPMI
 │
 ├── Temperature
 ├── Fan
 ├── Voltage
 ├── Power
 └── Hardware Health
```

### Checklist

* [ ] Check temperature.
* [ ] Check fan health.
* [ ] Check power.
* [ ] Check voltage.
* [ ] Check hardware alarms.
* [ ] Correlate with FortiGate LED state.

---

# 29. Bluetooth / BLE Diagnostics

> ⚠️ BLE capabilities and commands are **platform/model dependent**.

## Diagnostic Commands

```bash
diagnose hardware test ble
```

```bash
diagnose bluetooth status
```

```bash
diagnose bluetooth get_bt_version
```

```bash
diagnose bluetooth clean_bt_mode
```

### BLE Troubleshooting Checklist

* [ ] Check Bluetooth status.
* [ ] Check Bluetooth version.
* [ ] Run BLE hardware test.
* [ ] Check pairing/device state.
* [ ] Clean Bluetooth mode if required.
* [ ] Re-test BLE functionality.

### Troubleshooting Flow

```text
BLE Problem
    ↓
diagnose bluetooth status
    ↓
diagnose bluetooth get_bt_version
    ↓
diagnose hardware test ble
    ↓
Check pairing/device state
    ↓
diagnose bluetooth clean_bt_mode
    ↓
Re-test
```

> ⚠️ Always verify command availability on the target FortiOS release.

---

# 30. LED Troubleshooting

## 🔴 Device Appears Unhealthy

* [ ] Check `STA`.
* [ ] Check `ALARM`.
* [ ] Check `PWR`.
* [ ] Check individual `PSU`.
* [ ] Check `FAN`.
* [ ] Check `HA`.
* [ ] Check system logs.
* [ ] Check event logs.
* [ ] Correlate hardware and FortiOS state.

```text
STA / ALARM
      ↓
PWR / PSU
      ↓
FAN
      ↓
HA
      ↓
System/Event Logs
      ↓
Hardware Guide
```

---

## ⚡ Power Problem

* [ ] Check PWR.
* [ ] Check individual PSU.
* [ ] Check PSU INPUT.
* [ ] Check PSU OUTPUT.
* [ ] Check PSU FAIL.
* [ ] Check AC source.
* [ ] Check power cable.
* [ ] Check hardware logs.

```text
PWR Abnormal
     ↓
Individual PSU
     ↓
INPUT
     ↓
OUTPUT
     ↓
FAIL
     ↓
AC / Power Cable
```

---

## 🌐 Network Port Problem

* [ ] Check Ethernet/SFP LED.
* [ ] Is LED OFF?
* [ ] Check physical link.
* [ ] Check cable/transceiver.
* [ ] Check peer.
* [ ] Check speed.
* [ ] Check activity.
* [ ] Check interface status.
* [ ] Check configuration.
* [ ] Check errors/drops.

```text
No Connectivity
      ↓
LED Check
      ↓
LED OFF?
 ┌────┴────┐
 YES       NO
  ↓         ↓
Physical   Check Speed
Problem       ↓
           Activity
              ↓
       Interface State
              ↓
          Configuration
```

---

## 🔄 HA Problem

* [ ] Check HA LED.
* [ ] Check cluster status.
* [ ] Check members.
* [ ] Check heartbeat interfaces.
* [ ] Check synchronization.
* [ ] Check failover events.
* [ ] Check system/event logs.

```text
HA LED Abnormal
      ↓
Cluster Status
      ↓
Members
      ↓
Heartbeat
      ↓
Synchronization
      ↓
Failover Events
      ↓
Logs
```

---

## 🌡️ Thermal / Fan Problem

* [ ] Check FAN LED.
* [ ] Check temperature.
* [ ] Check fan RPM.
* [ ] Check fan set status.
* [ ] Check airflow.
* [ ] Check physical obstruction.
* [ ] Check hardware alarms.
* [ ] Check event logs.

```text
FAN / Temperature Alarm
          ↓
      Fan Status
          ↓
       Fan RPM
          ↓
      Airflow
          ↓
    Hardware Logs
          ↓
    Replace Fan?
```

---

# 🧠 NSE Exam Memory Map

```text
LOGO
 ├── Green → ON
 ├── Blue  → FortiWiFi
 └── Off   → OFF
```

```text
PWR
 ├── Green → Normal power
 ├── Amber → PSU/reduced-power condition
 ├── Red   → Power failure condition
 └── Off   → OFF
```

```text
STA
 ├── Green       → Normal
 ├── Flash Green → Boot
 ├── Amber       → Alarm
 └── Red         → Major alarm
```

```text
ALARM
 ├── Red   → Major
 ├── Amber → Minor
 └── Off   → None
```

```text
HA
 ├── Green     → HA active
 ├── Amber/Red → Failover condition
 └── Off       → HA disabled
```

```text
FAN
 ├── Green → Normal
 ├── Amber → Failure
 └── Red   → Serious fan condition
```

---

# ⚠️ Exam & Real-World Traps

### Trap #1 — LED ≠ Complete Diagnosis

```text
LED
 ↓
Symptom
 ↓
NOT root cause
```

Always correlate the LED with:

* [ ] FortiOS state
* [ ] Hardware state
* [ ] System logs
* [ ] Event logs
* [ ] Model-specific hardware guide

---

### Trap #2 — Hardware LEDs Are Model Dependent

Do **not** assume every FortiGate has:

* [ ] BYP
* [ ] MAX POE
* [ ] 3G/4G
* [ ] WIFI
* [ ] BLE
* [ ] IPMI
* [ ] Individual PSU indicators

---

### Trap #3 — Flashing Does Not Always Mean "Activity"

Depending on the LED:

```text
Flashing
   ├── Activity
   ├── Boot
   ├── Initialization
   ├── Warning
   ├── Pairing
   └── Transition
```

Always interpret the flashing state according to the **specific LED and model**.

---

### Trap #4 — SFP Family Does Not Guarantee Every Speed

```text
SFP
SFP+
SFP28
QSFP28
```

are port/module families.

Actual speed support depends on:

* [ ] Hardware model
* [ ] Port
* [ ] Transceiver
* [ ] Cable
* [ ] Peer
* [ ] Port configuration

---

### Trap #5 — PWR ≠ Individual PSU Health

A redundant-power FortiGate may require checking:

```text
PWR
 ↓
PSU 1
 ↓
PSU 2
 ↓
INPUT
 ↓
OUTPUT
 ↓
FAIL
```

---

### Trap #6 — HA LED Does Not Replace HA Diagnostics

If the HA LED is abnormal:

```text
HA LED
 ↓
HA status
 ↓
Members
 ↓
Heartbeat
 ↓
Synchronization
 ↓
Failover events
 ↓
Logs
```

---

# 🔥 One-Minute FortiGate Hardware Checklist

Before escalating a hardware issue:

* [ ] Identify exact FortiGate model.
* [ ] Identify affected LED.
* [ ] Record LED color.
* [ ] Record steady/flashing state.
* [ ] Check LOGO.
* [ ] Check PWR.
* [ ] Check PSU.
* [ ] Check STA.
* [ ] Check ALARM.
* [ ] Check FAN.
* [ ] Check HA.
* [ ] Check BYP if available.
* [ ] Check PoE if available.
* [ ] Check Ethernet/SFP LEDs.
* [ ] Check physical cables/transceivers.
* [ ] Check temperature.
* [ ] Check hardware diagnostics.
* [ ] Check system/event logs.
* [ ] Correlate with the model-specific hardware guide.

---

# 🎯 SheynShield Hardware Troubleshooting Formula

```text
LED
 ↓
Identify State
 ↓
Identify Severity
 ↓
Identify Hardware Component
 ↓
Run Diagnostic
 ↓
Check FortiOS State
 ↓
Check System/Event Logs
 ↓
Correlate With Model
 ↓
Find Root Cause
 ↓
Fix
 ↓
Verify Recovery
```

> **Golden Rule:**
> **Never troubleshoot a FortiGate hardware LED in isolation.**
>
> Treat the LED as the **symptom**, then correlate it with **hardware state + FortiOS diagnostics + logs + model-specific documentation**.

---

# 📋 Field Engineer Final Checklist

### Before Touching Configuration

* [ ] Confirm exact model.
* [ ] Confirm FortiOS version.
* [ ] Identify affected hardware component.
* [ ] Record LED state.
* [ ] Record alarm severity.
* [ ] Check recent events.
* [ ] Check whether HA is involved.
* [ ] Check whether redundant hardware exists.

### Before Replacing Hardware

* [ ] Verify physical power.
* [ ] Verify PSU.
* [ ] Verify cable/transceiver.
* [ ] Verify fan/temperature.
* [ ] Verify interface state.
* [ ] Verify diagnostic output.
* [ ] Verify event logs.
* [ ] Rule out configuration/software causes.

### After Remediation

* [ ] LED returned to normal.
* [ ] Hardware state normal.
* [ ] Interface/link restored.
* [ ] HA synchronization restored if applicable.
* [ ] Alarm cleared.
* [ ] Logs show recovery.
* [ ] Traffic/service verified.

---

# 🔎 Keywords

`FortiGate LED` · `FortiGate status LED` · `FortiGate PWR LED` · `FortiGate STA LED` · `FortiGate HA LED` · `FortiGate alarm LED` · `FortiGate fan failure` · `FortiGate PSU troubleshooting` · `FortiGate PoE` · `FortiGate SFP` · `FortiGate SFP+` · `FortiGate SFP28` · `FortiGate QSFP28` · `FortiGate IPMI` · `FortiGate BLE diagnostics` · `FortiGate hardware troubleshooting` · `FortiGate hardware indicators` · `FortiGate LED troubleshooting` · `FortiGate hardware failure` · `FortiGate HA troubleshooting`

---

# 📚 SheynShield Quick Reference

> **LED → Hardware State → Diagnostic → Logs → Model-Specific Guide → Root Cause**

```text
Abnormal LED
     ↓
Identify LED
     ↓
Identify Color / Flash State
     ↓
Determine Severity
     ↓
Identify Hardware Component
     ↓
Run Diagnostic
     ↓
Check FortiOS State
     ↓
Check Logs
     ↓
Check Model Documentation
     ↓
Troubleshoot Root Cause
     ↓
Verify Recovery
```

---

## 🔗 SheynShield Resources

### 🎥 Video Learning

* [ ] [YouTube — SheynShield](https://youtube.com/@sheynshield)

  * Fortinet NSE content
  * FortiGate troubleshooting
  * Network Security Engineering

### 📚 Notes & Updates

* [ ] [Telegram — SheynShield](https://t.me/sheynshield)

### 💼 Professional Network

* [ ] [LinkedIn — Shayan-heydarikhah](https://linkedin.com/in/shayan-heydarikhah)

### 🐙 Technical Knowledge Base

* [ ] [SheynShield GitHub](https://github.com/Shayan-heydarikhah/sheynshield)

---

**SheynShield | Engineering Secure Networks**

> **Learn → Configure → Troubleshoot → Engineer**

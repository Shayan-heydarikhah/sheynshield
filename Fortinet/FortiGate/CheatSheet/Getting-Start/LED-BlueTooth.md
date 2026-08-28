# FortiGate LED & Hardware Indicators  

> **FortiGate Hardware LEDs — Quick Reference**
>
> Use this   to quickly interpret **system status, alarms, HA, power, PoE, wireless, Ethernet/SFP, fan, and hardware-management indicators**.

---

## 📌 Quick Mental Model

| Indicator                 | Main Purpose                    |
| ------------------------- | ------------------------------- |
| **LOGO**                  | Unit / FortiWiFi state          |
| **PWR**                   | Power-supply status             |
| **STA**                   | System / alarm status           |
| **BYP**                   | Hardware bypass state           |
| **ALARM**                 | Major / minor alarm             |
| **HA**                    | HA cluster / failover state     |
| **MAX POE**               | PoE power budget                |
| **POE**                   | PoE delivery/activity           |
| **SVC**                   | Service status/activity         |
| **3G/4G**                 | Cellular service/activity       |
| **WIFI**                  | Wi-Fi status/activity           |
| **Fan**                   | Fan health                      |
| **Ethernet**              | Link / activity / speed         |
| **SFP/SFP+/SFP28/QSFP28** | Optical link / activity / speed |
| **IPMI**                  | Hardware-management health      |

> ⚠️ **Model-dependent:** Not every FortiGate has all of these LEDs. Always correlate the LED legend with the specific hardware model.

---

# 1. LED Alarm Severity

A useful troubleshooting mental model:

| Severity        | Meaning                                                  |
| --------------- | -------------------------------------------------------- |
| 🟢 **Normal**   | Device operating normally                                |
| 🟠 **Minor**    | Non-critical condition / warning                         |
| 🔴 **Major**    | Serious condition that may affect operation              |
| 🔴 **Critical** | Severe or potentially non-recoverable hardware condition |

### Typical examples

* **Major:** OS crash, serious system failure
* **Minor:** High temperature, warning condition, abnormal behavior
* **Critical:** Hardware condition that may be non-recoverable

---

# 2. LOGO LED

| LED      | Meaning         |
| -------- | --------------- |
| 🟢 Green | Unit is ON      |
| 🔵 Blue  | FortiWiFi is ON |
| ⚫ Off    | Device is OFF   |

---

# 3. Power — PWR

| LED               | Meaning                                               |
| ----------------- | ----------------------------------------------------- |
| 🟢 Green          | Unit is ON and/or both power supplies are functioning |
| 🟠 Amber          | One power supply is functioning                       |
| 🟠 Flashing Amber | Power-supply failure                                  |
| 🔴 Red            | Unit is ON, but only one power supply is functional   |
| 🔴 Flashing Red   | Power failure                                         |
| ⚫ Off             | Unit is OFF                                           |

### 🔥 Exam Point

> **PWR ≠ individual PSU health.**
> On redundant-power platforms, inspect the individual power-supply indicators when available.

---

# 4. Status — STA

| LED               | Meaning                   |
| ----------------- | ------------------------- |
| 🟢 Green          | Normal operation          |
| 🟢 Flashing Green | Booting                   |
| 🟠 Amber          | Major or minor alarm      |
| 🟠 Flashing Amber | BLE is ON                 |
| 🔴 Red            | Major alarm               |
| 🔴 Flashing Red   | BLE is ON / pairing state |
| ⚫ Off             | Unit is OFF               |

---

# 5. Bypass — BYP

Hardware bypass is model-dependent.

| LED      | Meaning                                       |
| -------- | --------------------------------------------- |
| 🟠 Amber | Bypass port pair is active — **Fail Open**    |
| ⚫ Off    | Bypass port pair is inactive — **Fail Close** |

### Remember

```text
BYP Amber → Bypass ACTIVE
BYP Off   → Bypass INACTIVE
```

---

# 6. Alarm LED

| LED      | Meaning     |
| -------- | ----------- |
| 🔴 Red   | Major alarm |
| 🟠 Amber | Minor alarm |
| ⚫ Off    | No alarm    |

### Quick Severity

```text
Red   → Major
Amber → Minor
Off   → Normal
```

---

# 7. HA LED

| LED               | Meaning                    |
| ----------------- | -------------------------- |
| 🟢 Green          | Operating in an HA cluster |
| 🟠 Amber / 🔴 Red | HA failover condition      |
| ⚫ Off             | HA disabled                |

### Troubleshooting

If the HA LED indicates failover:

```text
HA LED
  ↓
Check HA status
  ↓
Check cluster members
  ↓
Check heartbeat interfaces
  ↓
Check synchronization
  ↓
Check event/system logs
```

---

# 8. MAX POE

Indicates the PoE power budget condition.

| LED      | Meaning                                                  |
| -------- | -------------------------------------------------------- |
| 🟢 Green | Maximum PoE power allocated                              |
| 🟠 Amber | PoE wattage usage is higher than normal                  |
| 🔴 Red   | Power cannot be delivered although a device is connected |
| ⚫ Off    | PoE power is available / normal                          |

### Key Point

```text
Amber → High PoE consumption
Red   → Cannot deliver required PoE power
```

---

# 9. PoE LED

| LED               | Meaning                                       |
| ----------------- | --------------------------------------------- |
| 🟢 Green          | Power is being delivered                      |
| 🟢 Flashing Green | Error or PoE device requesting power          |
| ⚫ Off             | No PoE device connected or no power delivered |

---

# 10. SVC LED

| LED               | Meaning      |
| ----------------- | ------------ |
| 🟢 Green          | SVC is ON    |
| 🟢 Flashing Green | SVC activity |
| ⚫ Off             | SVC is OFF   |

---

# 11. 3G / 4G LED

| LED               | Meaning              |
| ----------------- | -------------------- |
| 🟢 Green          | 3G/4G service is ON  |
| 🟢 Flashing Green | 3G/4G activity       |
| ⚫ Off             | 3G/4G service is OFF |

---

# 12. Wi-Fi LED

| LED               | Meaning         |
| ----------------- | --------------- |
| 🟢 Green          | Wi-Fi connected |
| 🟢 Flashing Green | Wi-Fi activity  |
| ⚫ Off             | Wi-Fi OFF       |

> ⚠️ Wi-Fi indicators are relevant primarily to **FortiWiFi / wireless-capable platforms**.

---

# 13. Power Supply LED

For platforms with individual PSU indicators:

| LED               | Meaning                                                            |
| ----------------- | ------------------------------------------------------------------ |
| 🟢 Green          | PSU operating normally                                             |
| 🟢 Flashing Green | Power detected, but PSU is not providing power or is in standby    |
| 🟠 Amber          | Output OFF, PSU error, or no input power while redundant PSU is ON |
| 🟠 Flashing Amber | PSU error/warning; PSU may need replacement                        |
| 🔴 Red            | Power cord unplugged or power lost                                 |
| 🔴 Flashing Red   | PSU warning event                                                  |
| ⚫ Off             | Power not detected                                                 |

---

# 14. Power Supply OK

| LED               | Meaning                        |
| ----------------- | ------------------------------ |
| 🟢 Green          | Standby rail + main output ON  |
| 🟢 Flashing Green | Standby rail + main output OFF |
| ⚫ Off             | Error or no AC input           |

---

# 15. Power Supply FAIL

| LED               | Meaning                           |
| ----------------- | --------------------------------- |
| 🟠 Amber          | Main output or fan error detected |
| 🟠 Flashing Amber | PSU warning event                 |
| ⚫ Off             | No error or no power              |

---

# 16. Power Supply INPUT

| LED               | Meaning                              |
| ----------------- | ------------------------------------ |
| 🟢 Green          | Input voltage within normal range    |
| 🟢 Flashing Green | Over-voltage / under-voltage warning |
| ⚫ Off             | No input power                       |

---

# 17. Power Supply OUTPUT

| LED               | Meaning               |
| ----------------- | --------------------- |
| 🟢 Green          | Output voltage normal |
| 🟢 Flashing Green | Standby mode          |
| 🟠 Amber          | Critical error        |
| 🟠 Flashing Amber | Warning               |
| ⚫ Off             | No output             |

---

# 18. Fan LED

| LED               | Meaning                                                     |
| ----------------- | ----------------------------------------------------------- |
| 🟢 Green          | Fan(s) operating normally                                   |
| 🟢 Flashing Green | Fan switching / initialization                              |
| 🟠 Amber          | Fan failure                                                 |
| 🔴 Red            | Fan error; RPM too low/high, or both fan sets have an alert |
| 🔴 Flashing Red   | One fan set has at least one alert                          |
| ⚫ Off             | Fan error or fan is OFF                                     |

### Troubleshooting Logic

```text
Fan LED
   │
   ├── Green
   │     └── Normal
   │
   ├── Amber
   │     └── Fan failure
   │
   ├── Red
   │     └── Serious fan/RPM condition
   │
   └── Flashing Red
         └── One fan set has an alert
```

---

# 19. Ethernet LEDs

Ethernet LED behavior varies by hardware design.

## Ethernet — Link + Speed

| LED               | Meaning                  |
| ----------------- | ------------------------ |
| 🟢 Green          | Connected at 1 Gbps      |
| 🟢 Flashing Green | TX/RX at 1 Gbps          |
| 🟠 Amber          | Connected at 10/100 Mbps |
| 🟠 Flashing Amber | TX/RX at 10/100 Mbps     |
| ⚫ Off             | No link                  |

---

## Ethernet Link / Activity

| LED               | Meaning           |
| ----------------- | ----------------- |
| 🟢 Green          | Link established  |
| 🟢 Flashing Green | Data transmission |
| ⚫ Off             | No link           |

---

## Ethernet Speed

| LED      | Speed                    |
| -------- | ------------------------ |
| 🟢 Green | 1 Gbps                   |
| 🟠 Amber | 100 Mbps                 |
| ⚫ Off    | Not connected or 10 Mbps |

---

# 20. 10G Ethernet

## 10G Link / Activity

| LED               | Meaning           |
| ----------------- | ----------------- |
| 🟢 Green          | Link established  |
| 🟢 Flashing Green | Data transmission |
| ⚫ Off             | No link           |

## 10G Speed

| LED      | Speed                     |
| -------- | ------------------------- |
| 🟢 Green | 10 Gbps                   |
| 🟠 Amber | 5 / 2.5 / 1 Gbps          |
| ⚫ Off    | Not connected or 100 Mbps |

---

# 21. Ethernet PoE

| LED      | Meaning                               |
| -------- | ------------------------------------- |
| 🟢 Green | PoE power ON / device receiving power |
| 🟠 Amber | Port is providing power               |
| 🔴 Red   | Device connected but not powered      |
| ⚫ Off    | PoE OFF / no device receiving power   |

---

# 22. SFP

| LED               | Meaning             |
| ----------------- | ------------------- |
| 🟢 Green          | Connected at 1 Gbps |
| 🟢 Flashing Green | Data activity       |
| ⚫ Off             | No link             |

---

# 23. SFP+

| LED               | Meaning                   |
| ----------------- | ------------------------- |
| 🟢 Green          | Connected at 10 or 1 Gbps |
| 🟢 Flashing Green | Data activity             |
| ⚫ Off             | No link                   |

---

# 24. SFP28

| LED               | Meaning                       |
| ----------------- | ----------------------------- |
| 🟢 Green          | Connected at 25 / 10 / 1 Gbps |
| 🟢 Flashing Green | Data activity                 |
| ⚫ Off             | No link                       |

---

# 25. QSFP28

| LED               | Meaning                    |
| ----------------- | -------------------------- |
| 🟢 Green          | Connected at 100 / 40 Gbps |
| 🟢 Flashing Green | Data activity              |
| ⚫ Off             | No link                    |

---

# 26. SFP Family — Fast Memorization

```text
SFP
 └── 1 Gbps

SFP+
 └── 10 / 1 Gbps

SFP28
 └── 25 / 10 / 1 Gbps

QSFP28
 └── 100 / 40 Gbps
```

> ⚠️ Actual supported speeds depend on the specific FortiGate model, transceiver, port configuration, and hardware generation.

---

# 27. Hardware Troubleshooting Commands

## Linux Bridge / Netlink

```bash
diagnose netlink brctl name host root.b
```

Useful when investigating bridge-related hardware/network behavior.

---

# 28. IPMI — Intelligent Platform Management Interface

**IPMI** is a common datacenter hardware-management standard used for monitoring hardware-level conditions such as:

* Temperature
* Fan status
* Power
* Voltage
* Hardware health
* Environmental conditions

### Alarm Classification

| Condition                | Classification |
| ------------------------ | -------------- |
| Non-critical             | Minor alarm    |
| Critical but recoverable | Major alarm    |
| Non-recoverable          | Critical alarm |

### Mental Model

```text
IPMI
 │
 ├── Temperature
 ├── Fan
 ├── Voltage
 ├── Power
 └── Hardware health
```

---

# 29. Bluetooth / BLE Diagnostics

Some FortiGate/FortiWiFi platforms provide Bluetooth/BLE functionality.

### Diagnostics

```bash
diagnose hardware test ble
```

Check Bluetooth status:

```bash
diagnose bluetooth status
```

Display Bluetooth version:

```bash
diagnose bluetooth get_bt_version
```

Clean Bluetooth mode:

```bash
diagnose bluetooth clean_bt_mode
```

### BLE Troubleshooting Flow

```text
BLE problem
    ↓
diagnose bluetooth status
    ↓
diagnose bluetooth get_bt_version
    ↓
diagnose hardware test ble
    ↓
Check pairing / device state
    ↓
diagnose bluetooth clean_bt_mode
```

> ⚠️ BLE commands and capabilities are **platform/model dependent**.

---

# 30. LED Troubleshooting  

## Device appears unhealthy

```text
Check STA
   ↓
Check ALARM
   ↓
Check PWR / PSU
   ↓
Check FAN
   ↓
Check HA
   ↓
Check system/event logs
```

---

## Power Problem

```text
PWR abnormal
     ↓
Check individual PSU
     ↓
Check PSU INPUT
     ↓
Check PSU OUTPUT
     ↓
Check PSU FAIL
     ↓
Check AC/power cable
```

---

## Network Port Problem

```text
No connectivity
     ↓
Check Ethernet/SFP LED
     ↓
LED OFF?
     ├── YES → Physical/link problem
     │
     └── NO
          ↓
      Check speed
          ↓
      Check activity
          ↓
      Check interface status/config
```

---

## HA Problem

```text
HA LED abnormal
      ↓
Check cluster status
      ↓
Check heartbeat links
      ↓
Check member status
      ↓
Check synchronization
      ↓
Check failover events
```

---

# 🧠 NSE Exam Memory Map

```text
LOGO
 ├── Green  → ON
 ├── Blue   → FortiWiFi
 └── Off    → OFF

PWR
 ├── Green  → Normal power
 ├── Amber  → Reduced/PSU condition
 ├── Red    → Power failure condition
 └── Off    → OFF

STA
 ├── Green        → Normal
 ├── Flash Green  → Boot
 ├── Amber        → Alarm
 └── Red          → Major alarm

ALARM
 ├── Red    → Major
 ├── Amber  → Minor
 └── Off    → None

HA
 ├── Green       → HA active
 ├── Amber/Red   → Failover
 └── Off         → HA disabled

FAN
 ├── Green → Normal
 ├── Amber → Failure
 └── Red   → Serious fan condition
```

---

# ⚠️ Important Exam / Real-World Notes

1. **LED definitions are hardware/model dependent.**
2. Do not assume every FortiGate has `BYP`, `MAX POE`, `3G/4G`, `WIFI`, `BLE`, IPMI, or individual PSU indicators.
3. **Flashing usually indicates activity, transition, initialization, or warning**, depending on the specific LED.
4. Always correlate the LED with:

   * FortiGate model
   * Hardware guide
   * System/event logs
   * `diagnose` output
5. A physical LED condition should not be interpreted independently from the actual FortiOS state.

---

# 🔎 High-Value Keywords

`FortiGate LED` · `FortiGate status LED` · `FortiGate PWR LED` · `FortiGate STA LED` · `FortiGate HA LED` · `FortiGate alarm LED` · `FortiGate fan failure` · `FortiGate PSU` · `FortiGate PoE` · `FortiGate SFP` · `FortiGate SFP28` · `FortiGate QSFP28` · `FortiGate IPMI` · `FortiGate BLE diagnostics` · `FortiGate hardware troubleshooting`

---

## 📚 SheynShield Quick Reference

> **LED → Hardware State → Diagnose → Logs → Hardware Guide**

```text
LED abnormal
    ↓
Identify LED
    ↓
Determine severity
    ↓
Check hardware state
    ↓
Run diagnose command
    ↓
Check system/event logs
    ↓
Correlate with FortiGate model
    ↓
Troubleshoot root cause
```

**SheynShield | Engineering Secure Networks**

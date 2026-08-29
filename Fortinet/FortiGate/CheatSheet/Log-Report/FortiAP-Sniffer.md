# FortiGate Advanced Logging & Firewall Sniffer  

> **FortiOS | FortiAP WiFi Telemetry + Firewall Sniffer + IPS-Based Traffic Inspection**

---

## 📌 Table of Contents

* [1. FortiAP WiFi Signal Logging](#1-fortiap-wifi-signal-logging)
* [2. WiFi Event Logs](#2-wifi-event-logs)
* [3. Historical Client Signal Analysis](#3-historical-client-signal-analysis)
* [4. Firewall Sniffer](#4-firewall-sniffer)
* [5. Firewall Sniffer Configuration](#5-firewall-sniffer-configuration)
* [6. Firewall Sniffer vs Diagnose Sniffer](#6-firewall-sniffer-vs-diagnose-sniffer)
* [7. Practical Use Cases](#7-practical-use-cases)
* [8. Troubleshooting Workflow](#8-troubleshooting-workflow)
* [9. NSE High-Value Notes](#9-nse-high-value-notes)
* [10. Quick Reference](#10-quick-reference)

---

# 1. FortiAP WiFi Signal Logging

When a **FortiAP** is managed by a **FortiGate**, wireless client telemetry can be recorded in FortiGate logs.

For clients connecting to an SSID configured in:

* **Tunnel mode**
* **Local-Bridge mode**

FortiGate can record wireless signal information such as:

```text
Signal Strength
Signal-to-Noise Ratio (SNR)
```

This provides historical visibility into the quality of the wireless client connection.

---

## 📡 What Gets Logged?

### Signal Strength

Indicates how strong the received wireless signal is from the client/AP perspective.

Conceptually:

```text
Strong Signal
     │
     ▼
Better wireless coverage
```

### Signal-to-Noise Ratio (SNR)

Represents the relationship between the desired wireless signal and background noise.

Conceptually:

```text
Higher SNR
    ↓
Cleaner RF environment
    ↓
Better wireless communication
```

> **Important:** A client can have relatively good signal strength but poor SNR because of RF interference/noise.

---

# 2. WiFi Event Logs

Navigate to:

```text
Log & Report
    └── System Events
         └── WiFi Event
```

These logs provide historical information about wireless client events.

### Log Location

```text
Log & Report
      │
      ▼
System Events
      │
      ▼
WiFi Event
```

---

# 3. Historical Client Signal Analysis

One of the major benefits of logging these values is **historical wireless troubleshooting**.

Instead of asking:

> "What is the client's signal right now?"

you can investigate:

> "What was the client's signal quality when the problem occurred?"

---

## 🔍 Example Investigation

Imagine a user reports:

```text
10:35 → WiFi disconnect
10:40 → Slow connection
10:45 → Authentication issue
```

Historical WiFi event logs can help correlate:

```text
Client
  │
  ├── Signal Strength
  │
  ├── SNR
  │
  ├── Authentication Event
  │
  └── Timestamp
```

This allows correlation between:

```text
Wireless Event
       +
Signal Quality
       +
Authentication
       +
Time
```

---

## Tunnel vs Local-Bridge Logging

The location of the relevant information differs depending on the SSID forwarding mode.

| FortiAP SSID Mode | Relevant Log                                      |
| ----------------- | ------------------------------------------------- |
| **Local Bridge**  | WiFi Event logs / local-bridge traffic statistics |
| **Tunnel**        | Forward Traffic logs / tunnel traffic             |
| Authentication    | WiFi Event logs                                   |

### Simplified Flow

```text
                   FortiAP
                     │
             ┌───────┴────────┐
             │                │
       Local Bridge         Tunnel
             │                │
             ▼                ▼
       Local Traffic     FortiGate Traffic
             │                │
             ▼                ▼
       WiFi Event       Forward Traffic
```

> **Key point:** Signal strength and SNR information can be associated with the relevant logged wireless/traffic events, allowing historical analysis.

---

# 4. Firewall Sniffer

FortiGate also provides a **Firewall Sniffer** feature for inspecting and logging traffic using a configured sniffer policy/profile.

This is different from simply running an on-demand diagnostic packet capture.

Conceptually:

```text
Traffic
   │
   ▼
Firewall Sniffer
   │
   ├── Logging
   ├── IPS inspection
   └── Sniffer Profile
```

### Why use Firewall Sniffer?

It can be useful when you want to:

* Detect hidden traffic patterns
* Inspect suspicious traffic
* Apply IPS inspection to selected traffic
* Generate logs
* Use a defined sniffer profile
* Monitor traffic continuously according to configuration

---

# 5. Firewall Sniffer Configuration

Example configuration:

```bash
config firewall sniffer
    edit 1
        set status enable
        set logtraffic utm
        set interface port1
        set application-list-status disable
        set ips-sensor-status enable
        set ips-sensor sniffer-profile x-sniff
        set dsri disable
        set av-profile-status disable
        set webfilter-profile-status disable
        set emailfilter-profile-status disable
        set dlp-profile-status disable
        set ip-threatfeed-status disable
        set file-filter-profile-status disable
        set ips-dos-status disable
    next
end
```

---

## 🧩 Configuration Breakdown

### Enable the Sniffer

```bash
set status enable
```

Activates the configured firewall sniffer entry.

---

### Enable UTM Logging

```bash
set logtraffic utm
```

Enables UTM-style traffic logging for the sniffer.

---

### Select Interface

```bash
set interface port1
```

Specifies the interface where the sniffer operates.

---

### Disable Application List

```bash
set application-list-status disable
```

Application-list processing is disabled.

---

### Enable IPS

```bash
set ips-sensor-status enable
```

Enables IPS inspection.

Then specify the IPS sensor/profile:

```bash
set ips-sensor sniffer-profile x-sniff
```

Conceptually:

```text
Firewall Sniffer
       │
       ▼
   IPS Enabled
       │
       ▼
 sniffer-profile
       │
       ▼
   Inspection
```

---

### Disable DSRI

```bash
set dsri disable
```

Disables DSRI for this sniffer configuration.

---

## Disable Other Security Profiles

The example intentionally disables several inspection modules:

```bash
set av-profile-status disable
set webfilter-profile-status disable
set emailfilter-profile-status disable
set dlp-profile-status disable
set ip-threatfeed-status disable
set file-filter-profile-status disable
set ips-dos-status disable
```

### Result

The sniffer configuration focuses primarily on:

```text
Traffic Logging
       +
IPS Inspection
```

rather than enabling every available UTM inspection feature.

---

# 6. Firewall Sniffer vs Diagnose Sniffer

This distinction is extremely important for troubleshooting.

## Firewall Sniffer

```text
config firewall sniffer
```

is a **configured firewall feature**.

It can provide:

* Persistent/configured inspection
* Logging
* IPS inspection
* Configurable sniffer profile
* Visibility into traffic patterns

---

## Diagnose Sniffer

The diagnostic packet sniffer is typically used as an **on-demand troubleshooting tool**.

Conceptually:

```text
diagnose sniffer packet
```

is used when an engineer wants to capture traffic interactively.

---

## Comparison

| Feature                   | Firewall Sniffer          | Diagnose Sniffer          |
| ------------------------- | ------------------------- | ------------------------- |
| Configuration object      | ✅ Yes                     | ❌ No                      |
| Persistent configuration  | ✅                         | ❌                         |
| On-demand troubleshooting | ⚠️                        | ✅                         |
| Live packet visibility    | Limited/feature dependent | ✅                         |
| IPS profile               | ✅                         | ❌                         |
| Logging                   | ✅                         | Diagnostic output/capture |
| Use case                  | Monitoring / inspection   | Packet troubleshooting    |
| Sniffer profile           | ✅                         | ❌                         |
| Network diagnostics       | Limited                   | Strong                    |

### Mental Model

```text
                 FortiGate Traffic
                        │
            ┌───────────┴───────────┐
            │                       │
            ▼                       ▼
    Firewall Sniffer        Diagnose Sniffer
            │                       │
            ▼                       ▼
     Configured Model        On-Demand Model
            │                       │
            ├── Logging             └── Live Capture
            ├── IPS
            └── Sniffer Profile
```

> **Remember:**
> **Firewall Sniffer = configured inspection/logging feature**
> **Diagnose Sniffer = on-demand packet troubleshooting**

---

# 7. Practical Use Cases

## 🛜 Use Case 1 — Poor WiFi Performance

User reports:

> "WiFi is slow."

Investigate:

```text
Client
  ↓
WiFi Event
  ↓
Signal Strength
  ↓
SNR
  ↓
Timestamp
```

Then correlate with:

* Authentication events
* Traffic logs
* AP events
* Client movement
* RF conditions

---

## 🔐 Use Case 2 — Suspicious Traffic

You suspect hidden or unusual traffic patterns.

Use:

```text
Firewall Sniffer
      ↓
Traffic Logging
      ↓
IPS Inspection
      ↓
Sniffer Profile
      ↓
Log Analysis
```

This is useful when normal firewall-policy logging does not provide enough visibility.

---

## 🔬 Use Case 3 — Packet-Level Troubleshooting

You need to answer:

> "Is the packet reaching the interface?"

Use the diagnostic sniffer:

```text
diagnose sniffer
       ↓
Live packet capture
       ↓
Analyze packet path
```

---

# 8. Troubleshooting Workflow

## WiFi Problem

```text
             WiFi Problem
                  │
                  ▼
            Identify Client
                  │
                  ▼
            Check WiFi Event
                  │
                  ▼
       Check Signal Strength
                  │
                  ▼
              Check SNR
                  │
                  ▼
        Check Authentication
                  │
                  ▼
       Check Traffic Correlation
                  │
                  ▼
             Root Cause
```

---

## Traffic Visibility Problem

```text
       Traffic Investigation
                │
                ▼
      Do normal logs provide
          enough visibility?
          │             │
         YES           NO
          │             │
          ▼             ▼
       Analyze      Firewall Sniffer
       Logs              │
                         ▼
                  Sniffer Profile
                         │
                         ▼
                    IPS Inspection
                         │
                         ▼
                      Logs
```

---

# 9. NSE High-Value Notes 🧠

### FortiAP

```text
FortiAP
   ↓
FortiGate Managed AP
   ↓
WiFi Event Logs
```

Historical wireless telemetry can help correlate:

```text
Signal Strength
+
SNR
+
Authentication
+
Traffic
```

---

### Where to Look?

```text
Log & Report
    ↓
System Events
    ↓
WiFi Event
```

For tunnel traffic, relevant information can also appear in:

```text
Forward Traffic Logs
```

---

### Firewall Sniffer

Main configuration:

```bash
config firewall sniffer
```

Enable:

```bash
set status enable
```

Select interface:

```bash
set interface port1
```

Enable IPS:

```bash
set ips-sensor-status enable
```

Select IPS sniffer profile:

```bash
set ips-sensor sniffer-profile x-sniff
```

Enable UTM logging:

```bash
set logtraffic utm
```

---

### Most Important Concept

```text
Firewall Sniffer
      ≠
Diagnose Sniffer
```

**Firewall Sniffer:**

```text
Configured
      +
Logging
      +
Inspection
      +
IPS Profile
```

**Diagnose Sniffer:**

```text
On-Demand
      +
Live Packet Capture
      +
Network Troubleshooting
```

---

# 10. Quick Reference

| Task                          | Configuration / Location                    |
| ----------------------------- | ------------------------------------------- |
| FortiAP wireless events       | `Log & Report → System Events → WiFi Event` |
| Historical signal information | WiFi Event / relevant traffic logs          |
| Local-bridge traffic          | Local-bridge traffic statistics             |
| Tunnel traffic                | Forward Traffic logs                        |
| Configure Firewall Sniffer    | `config firewall sniffer`                   |
| Enable sniffer                | `set status enable`                         |
| Enable UTM logging            | `set logtraffic utm`                        |
| Select interface              | `set interface <interface>`                 |
| Enable IPS                    | `set ips-sensor-status enable`              |
| Select IPS profile            | `set ips-sensor <profile>`                  |
| Disable AV                    | `set av-profile-status disable`             |
| Disable Web Filter            | `set webfilter-profile-status disable`      |
| Disable DLP                   | `set dlp-profile-status disable`            |
| Disable File Filter           | `set file-filter-profile-status disable`    |
| Disable IPS DoS               | `set ips-dos-status disable`                |
| Live packet troubleshooting   | `diagnose sniffer`                          |

---

# 🎯 Final Mental Model

```text
                         FORTIGATE
                            │
          ┌─────────────────┴─────────────────┐
          │                                   │
       FortiAP                           Firewall Traffic
          │                                   │
          ▼                                   ▼
    WiFi Telemetry                     Firewall Sniffer
          │                                   │
     ┌────┴────┐                         ┌────┴────┐
     │         │                         │         │
 Signal       SNR                    Logging      IPS
 Strength                             │           │
     │                                └────┬──────┘
     ▼                                     ▼
Historical Logs                      Sniffer Profile
     │
     ▼
Correlation & Troubleshooting
```

> **Production mindset:**
> Wireless troubleshooting becomes significantly more powerful when **signal strength + SNR + authentication + traffic logs** are correlated against the exact timestamp of the incident. For traffic investigations, use **Firewall Sniffer** when you need configured inspection/logging, and **diagnose sniffer** when you need immediate packet-level troubleshooting.

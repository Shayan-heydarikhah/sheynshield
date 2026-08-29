# FortiGate Advanced Logging & Firewall Sniffer Checklist

> **FortiOS | FortiAP WiFi Telemetry | WiFi Event Logs | Firewall Sniffer | IPS Inspection | Traffic Visibility | Troubleshooting**

[![FortiOS](https://img.shields.io/badge/FortiOS-FortiGate-red)](https://www.fortinet.com/products/next-generation-firewall)
[![Security](https://img.shields.io/badge/Focus-Network%20Security-blue)](https://github.com/Shayan-heydarikhah/sheynshield)
[![GitHub](https://img.shields.io/badge/Format-GitHub%20Markdown-black)](https://github.com/Shayan-heydarikhah/sheynshield)

---

## 📋 Table of Contents

* [Overview](#overview)
* [1. FortiAP WiFi Telemetry Checklist](#1-fortiap-wifi-telemetry-checklist)
* [2. WiFi Event Log Checklist](#2-wifi-event-log-checklist)
* [3. Historical Signal Analysis](#3-historical-signal-analysis)
* [4. Tunnel vs Local-Bridge Logging](#4-tunnel-vs-local-bridge-logging)
* [5. Firewall Sniffer Checklist](#5-firewall-sniffer-checklist)
* [6. Firewall Sniffer Configuration](#6-firewall-sniffer-configuration)
* [7. IPS-Based Sniffer Inspection](#7-ips-based-sniffer-inspection)
* [8. Firewall Sniffer vs Diagnose Sniffer](#8-firewall-sniffer-vs-diagnose-sniffer)
* [9. WiFi Troubleshooting Workflow](#9-wifi-troubleshooting-workflow)
* [10. Traffic Visibility Workflow](#10-traffic-visibility-workflow)
* [11. Packet-Level Troubleshooting](#11-packet-level-troubleshooting)
* [12. NSE High-Value Checklist](#12-nse-high-value-checklist)
* [13. Production Checklist](#13-production-checklist)
* [14. Quick Command Reference](#14-quick-command-reference)
* [15. Final Mental Model](#15-final-mental-model)
* [SheynShield Resources](#sheynshield-resources)

---

# Overview

FortiGate logging can provide visibility into two very different troubleshooting domains:

```text
                         FortiGate
                            |
              +-------------+-------------+
              |                           |
           FortiAP                    Firewall
              |                           |
              v                           v
       WiFi Telemetry              Traffic Visibility
              |                           |
        +-----+-----+               +-----+------+
        |           |               |            |
     Signal       SNR          Firewall       Diagnose
     Strength                 Sniffer        Sniffer
        |                       |              |
        +----------+------------+--------------+
                   |
                   v
              Correlation
                   |
                   v
             Root Cause
```

### Core Principle

```text
WiFi Problem
    |
    +--> Signal Strength
    +--> SNR
    +--> Authentication
    +--> Traffic
    +--> Timestamp
```

For traffic visibility:

```text
Configured inspection
        |
        +--> Firewall Sniffer

Immediate packet capture
        |
        +--> Diagnose Sniffer
```

---

# 1. FortiAP WiFi Telemetry Checklist

When a **FortiAP is managed by FortiGate**, wireless client telemetry can be available in FortiGate logging.

### Verify

* [ ] FortiAP is managed by FortiGate
* [ ] Client is connected to the expected SSID
* [ ] SSID is operating in a supported forwarding mode
* [ ] WiFi events are being logged
* [ ] Signal information is available
* [ ] SNR information is available
* [ ] Event timestamps are available
* [ ] Client identity/MAC can be correlated with the event

### Key Metrics

| Metric                   | What it tells you                        |
| ------------------------ | ---------------------------------------- |
| **Signal Strength**      | Received wireless signal level           |
| **SNR**                  | Relationship between signal and RF noise |
| **Timestamp**            | When the condition occurred              |
| **Authentication Event** | Authentication/connectivity activity     |
| **Client Identity**      | Which endpoint generated the event       |

### Important

```text
Good Signal
    ≠
Good WiFi

Good Signal + Poor SNR
        |
        v
Possible RF interference/noise
```

---

# 2. WiFi Event Log Checklist

Navigate to:

```text
Log & Report
    └── System Events
         └── WiFi Event
```

### Investigation Checklist

* [ ] Open **System Events**
* [ ] Select **WiFi Event**
* [ ] Identify the affected client
* [ ] Check event timestamp
* [ ] Check signal strength
* [ ] Check SNR
* [ ] Check authentication events
* [ ] Correlate with traffic events
* [ ] Compare events before and after the incident

### Investigation Model

```text
Client
  |
  v
WiFi Event
  |
  +--> Signal Strength
  |
  +--> SNR
  |
  +--> Authentication
  |
  +--> Timestamp
  |
  v
Historical Correlation
```

---

# 3. Historical Signal Analysis

One of the biggest advantages of logging is the ability to investigate the **condition at the time of failure**, rather than only the current state.

### Troubleshooting Checklist

* [ ] Record incident timestamp
* [ ] Identify client MAC/IP
* [ ] Locate WiFi event
* [ ] Check signal strength
* [ ] Check SNR
* [ ] Check authentication events
* [ ] Check AP events
* [ ] Check traffic logs
* [ ] Compare multiple timestamps
* [ ] Determine whether the problem is persistent or intermittent

### Example

```text
10:35
WiFi disconnect
    |
    +--> Check signal
    +--> Check SNR
    +--> Check authentication

10:40
Slow connection
    |
    +--> Check traffic
    +--> Check RF metrics

10:45
Authentication issue
    |
    +--> Correlate WiFi events
```

### Golden Rule

> **Always correlate wireless metrics with the exact timestamp of the incident.**

---

# 4. Tunnel vs Local-Bridge Logging

The SSID forwarding mode affects where relevant traffic information can be observed.

| SSID Mode          | Primary Investigation Area            |
| ------------------ | ------------------------------------- |
| **Local Bridge**   | WiFi Events / local-bridge traffic    |
| **Tunnel**         | Forward Traffic Logs / tunnel traffic |
| **Authentication** | WiFi Event Logs                       |

### Local Bridge

```text
FortiAP
   |
   v
Local Bridge
   |
   v
Local Network
   |
   v
Traffic
```

### Tunnel Mode

```text
FortiAP
   |
   v
CAPWAP Tunnel
   |
   v
FortiGate
   |
   v
Forward Traffic
```

### Checklist

* [ ] Identify SSID forwarding mode
* [ ] Determine whether traffic is local-bridged
* [ ] Determine whether traffic is tunneled
* [ ] Check WiFi Event logs
* [ ] Check Forward Traffic logs when applicable
* [ ] Correlate authentication and traffic events

---

# 5. Firewall Sniffer Checklist

FortiGate provides a configured **Firewall Sniffer** feature for traffic visibility and inspection.

Configuration begins with:

```bash
config firewall sniffer
```

### Verify

* [ ] Firewall Sniffer object exists
* [ ] Sniffer status is enabled
* [ ] Correct interface is selected
* [ ] Logging is configured
* [ ] Required inspection modules are enabled
* [ ] Unnecessary inspection modules are disabled
* [ ] IPS sensor is configured when required
* [ ] Sniffer profile exists
* [ ] Logs are being generated
* [ ] Sniffer scope matches the troubleshooting requirement

### Concept

```text
Traffic
   |
   v
Firewall Sniffer
   |
   +--> Logging
   |
   +--> IPS Inspection
   |
   +--> Sniffer Profile
   |
   v
Log Analysis
```

---

# 6. Firewall Sniffer Configuration

Example:

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

## Configuration Validation

### Sniffer Status

```bash
set status enable
```

* [ ] Sniffer is enabled

### Logging

```bash
set logtraffic utm
```

* [ ] UTM logging is enabled when required

### Interface

```bash
set interface port1
```

* [ ] Correct interface selected
* [ ] Interface receives the traffic being investigated

### Application List

```bash
set application-list-status disable
```

* [ ] Application processing is intentionally enabled/disabled

### IPS

```bash
set ips-sensor-status enable
set ips-sensor sniffer-profile x-sniff
```

* [ ] IPS inspection enabled when required
* [ ] Correct IPS sensor/profile selected
* [ ] Sniffer profile exists
* [ ] IPS events are being logged

---

# 7. IPS-Based Sniffer Inspection

Firewall Sniffer can be used with IPS inspection for selected traffic.

### Flow

```text
Traffic
   |
   v
Firewall Sniffer
   |
   v
IPS Enabled?
   |
  YES
   |
   v
Sniffer IPS Profile
   |
   v
Inspection
   |
   v
Security Event / Log
```

### Checklist

* [ ] `ips-sensor-status` enabled
* [ ] Correct IPS sensor configured
* [ ] Sniffer profile exists
* [ ] Relevant signatures are enabled
* [ ] Traffic reaches the configured interface
* [ ] Expected IPS logs are generated
* [ ] False positives are considered
* [ ] Performance impact is evaluated

### Example

```bash
set ips-sensor-status enable
set ips-sensor sniffer-profile x-sniff
```

---

## Other Security Profiles

The example configuration disables several inspection engines:

```bash
set av-profile-status disable
set webfilter-profile-status disable
set emailfilter-profile-status disable
set dlp-profile-status disable
set ip-threatfeed-status disable
set file-filter-profile-status disable
set ips-dos-status disable
```

### Verify Intentionally

* [ ] Antivirus disabled intentionally
* [ ] Web Filter disabled intentionally
* [ ] Email Filter disabled intentionally
* [ ] DLP disabled intentionally
* [ ] IP Threat Feed disabled intentionally
* [ ] File Filter disabled intentionally
* [ ] IPS DoS disabled intentionally

> **Do not disable security inspection simply to make troubleshooting easier unless the operational impact is understood.**

---

# 8. Firewall Sniffer vs Diagnose Sniffer

This is a high-value FortiGate troubleshooting distinction.

## Firewall Sniffer

```bash
config firewall sniffer
```

Think:

```text
Configured
   +
Persistent
   +
Logging
   +
Inspection
   +
IPS Profile
```

## Diagnose Sniffer

Think:

```text
On-Demand
   +
Live Packet Capture
   +
Packet-Level Troubleshooting
```

Typical diagnostic command:

```bash
diagnose sniffer packet
```

---

## Comparison

| Capability                       |            Firewall Sniffer |  Diagnose Sniffer |
| -------------------------------- | --------------------------: | ----------------: |
| Configuration object             |                           ✅ |                 ❌ |
| Persistent configuration         |                           ✅ |                 ❌ |
| On-demand capture                |                          ⚠️ |                 ✅ |
| Live packet visibility           | Limited / feature-dependent |                 ✅ |
| IPS profile                      |                           ✅ |                 ❌ |
| Logging                          |                           ✅ | Diagnostic output |
| Sniffer profile                  |                           ✅ |                 ❌ |
| Packet-level troubleshooting     |                     Limited |                 ✅ |
| Continuous configured monitoring |                           ✅ |                 ❌ |

### Memory Trick

```text
FIREWALL SNIFFER
      =
Configured Inspection + Logging

DIAGNOSE SNIFFER
      =
Immediate Packet Capture
```

---

# 9. WiFi Troubleshooting Workflow

Use this workflow when a wireless user reports poor performance or disconnects.

```text
                  WiFi Problem
                       |
                       v
                 Identify Client
                       |
                       v
                 Record Timestamp
                       |
                       v
                Check WiFi Event
                       |
             +---------+---------+
             |                   |
             v                   v
       Signal Strength          SNR
             |                   |
             +---------+---------+
                       |
                       v
              Check Authentication
                       |
                       v
               Check AP Events
                       |
                       v
              Check Traffic Logs
                       |
                       v
                 Correlate
                       |
                       v
                  Root Cause
```

### WiFi Incident Checklist

* [ ] Client identified
* [ ] MAC address recorded
* [ ] IP address recorded
* [ ] AP identified
* [ ] SSID identified
* [ ] Incident timestamp recorded
* [ ] WiFi Event checked
* [ ] Signal Strength checked
* [ ] SNR checked
* [ ] Authentication events checked
* [ ] Traffic logs checked
* [ ] AP events checked
* [ ] Client movement considered
* [ ] RF interference considered
* [ ] Root cause documented

---

# 10. Traffic Visibility Workflow

When standard firewall logs do not provide enough information:

```text
Traffic Investigation
        |
        v
Are normal logs sufficient?
        |
   +----+----+
   |         |
  YES        NO
   |         |
   v         v
Analyze    Firewall
Logs       Sniffer
             |
             v
        Sniffer Profile
             |
             v
        IPS Inspection
             |
             v
           Logs
             |
             v
          Analysis
```

### Checklist

* [ ] Confirm traffic exists
* [ ] Check standard Forward Traffic logs
* [ ] Identify source
* [ ] Identify destination
* [ ] Identify interface
* [ ] Determine whether more visibility is required
* [ ] Configure Firewall Sniffer if appropriate
* [ ] Select correct interface
* [ ] Enable required logging
* [ ] Enable IPS if required
* [ ] Analyze generated logs

---

# 11. Packet-Level Troubleshooting

When the question is:

> **"Is the packet actually reaching the interface?"**

Use the diagnostic sniffer.

```text
Client
   |
   v
Network
   |
   v
FortiGate Interface
   |
   v
diagnose sniffer
   |
   v
Packet Capture
   |
   v
Packet Path Analysis
```

### Packet Capture Checklist

* [ ] Identify source IP
* [ ] Identify destination IP
* [ ] Identify ingress interface
* [ ] Identify expected egress interface
* [ ] Start diagnostic capture
* [ ] Generate test traffic
* [ ] Verify packet arrival
* [ ] Verify packet departure
* [ ] Check protocol/port
* [ ] Compare expected vs actual packet path
* [ ] Stop capture after troubleshooting

### Mental Model

```text
Firewall Sniffer
    |
    +--> "What traffic should I monitor/inspect?"

Diagnose Sniffer
    |
    +--> "What packets are actually moving right now?"
```

---

# 12. NSE High-Value Checklist

## FortiAP

* [ ] FortiAP managed by FortiGate
* [ ] WiFi Event logging available
* [ ] Signal Strength checked
* [ ] SNR checked
* [ ] Authentication events correlated
* [ ] Timestamp correlated
* [ ] Traffic logs correlated

### Remember

```text
Signal Strength
       +
SNR
       +
Authentication
       +
Traffic
       +
Timestamp
       =
Better WiFi Troubleshooting
```

---

## Firewall Sniffer

Main configuration:

```bash
config firewall sniffer
```

Enable:

```bash
set status enable
```

Logging:

```bash
set logtraffic utm
```

Interface:

```bash
set interface port1
```

IPS:

```bash
set ips-sensor-status enable
set ips-sensor sniffer-profile x-sniff
```

### Checklist

* [ ] `config firewall sniffer`
* [ ] `set status enable`
* [ ] `set logtraffic utm`
* [ ] Correct interface
* [ ] IPS status correct
* [ ] Correct IPS profile
* [ ] Required logs generated

---

## Diagnose Sniffer

Typical syntax:

```bash
diagnose sniffer packet
```

### Remember

```text
Firewall Sniffer
    =
Configured feature

Diagnose Sniffer
    =
On-demand troubleshooting
```

---

# 13. Production Checklist

Before deploying advanced logging/sniffer functionality in production:

### Scope

* [ ] Monitoring scope is clearly defined
* [ ] Correct interface selected
* [ ] Required traffic identified
* [ ] Logging requirements documented

### Security

* [ ] IPS profile reviewed
* [ ] Inspection modules intentionally enabled/disabled
* [ ] Security impact assessed
* [ ] False-positive impact considered

### Performance

* [ ] Expected traffic volume estimated
* [ ] Logging volume considered
* [ ] CPU impact considered
* [ ] Memory impact considered
* [ ] Disk/log storage considered
* [ ] IPS inspection impact considered

### Troubleshooting

* [ ] Incident timestamp available
* [ ] Client/source identified
* [ ] Destination identified
* [ ] Relevant logs enabled
* [ ] Capture scope minimized
* [ ] Debugging disabled after investigation

### Operational Hygiene

* [ ] Temporary configuration documented
* [ ] Temporary sniffer rules removed when no longer needed
* [ ] Excessive logging avoided
* [ ] Security profiles restored if temporarily changed
* [ ] Findings documented

---

# 14. Quick Command Reference

| Task                       | Command / Location                          |
| -------------------------- | ------------------------------------------- |
| WiFi Event Logs            | `Log & Report → System Events → WiFi Event` |
| Configure Firewall Sniffer | `config firewall sniffer`                   |
| Enable Firewall Sniffer    | `set status enable`                         |
| Enable UTM logging         | `set logtraffic utm`                        |
| Select interface           | `set interface <interface>`                 |
| Enable IPS                 | `set ips-sensor-status enable`              |
| Select IPS profile         | `set ips-sensor <profile>`                  |
| Disable AV                 | `set av-profile-status disable`             |
| Disable Web Filter         | `set webfilter-profile-status disable`      |
| Disable DLP                | `set dlp-profile-status disable`            |
| Disable File Filter        | `set file-filter-profile-status disable`    |
| Disable IPS DoS            | `set ips-dos-status disable`                |
| Diagnostic packet capture  | `diagnose sniffer packet`                   |

---

# 15. Final Mental Model

```text
                         FORTIGATE
                            |
             +--------------+--------------+
             |                             |
           FortiAP                       Traffic
             |                             |
             v                             v
      WiFi Telemetry                Visibility Tools
             |                             |
       +-----+-----+                 +-----+-----+
       |           |                 |           |
    Signal        SNR          Firewall       Diagnose
    Strength                  Sniffer        Sniffer
       |                         |              |
       v                         v              v
 Historical                  Logging        Live Capture
    Events                       |              |
       |                         +------+-------+
       |                                |
       +------------+-------------------+
                    |
                    v
               Correlation
                    |
                    v
               Root Cause
```

## 🎯 Golden Rules

```text
Good Signal
    ≠
Good WiFi
```

```text
High Signal + Low SNR
    |
    +--> Investigate RF noise/interference
```

```text
Firewall Sniffer
    ≠
Diagnose Sniffer
```

```text
Firewall Sniffer
    =
Configured Inspection + Logging
```

```text
Diagnose Sniffer
    =
On-Demand Packet Capture
```

### Troubleshooting Priority

```text
1. Identify the problem
        ↓
2. Record exact timestamp
        ↓
3. Identify client/source
        ↓
4. Check relevant logs
        ↓
5. Correlate signal/SNR/authentication/traffic
        ↓
6. Use Firewall Sniffer if configured inspection is required
        ↓
7. Use Diagnose Sniffer for packet-level analysis
        ↓
8. Identify root cause
        ↓
9. Remove temporary troubleshooting configuration
```

> **Production Mindset:** Don't collect logs simply because more logs exist. Collect the **right telemetry**, correlate it with the **right timestamp**, and choose the right tool for the question you are trying to answer.

---

# SheynShield Resources

### 🎥 Video Learning

* [YouTube — SheynShield](https://youtube.com/@sheynshield)

  * Fortinet NSE content
  * FortiGate troubleshooting
  * Network Security Engineering

### 📚 Notes & Updates

* [Telegram — SheynShield](https://t.me/sheynshield)

### 💼 Professional Network

* [LinkedIn — Shayan Heydari](https://linkedin.com/in/shayan-heydarikhah)

### 🐙 Technical Knowledge Base

* [SheynShield GitHub](https://github.com/Shayan-heydarikhah/sheynshield)

---

## 🔎 Keywords

`FortiGate Firewall Sniffer` · `FortiOS Firewall Sniffer` · `FortiAP WiFi Troubleshooting` · `FortiGate WiFi Logs` · `FortiAP Signal Strength` · `FortiAP SNR` · `FortiGate IPS Sniffer` · `FortiGate Diagnose Sniffer` · `FortiGate Packet Capture` · `FortiOS Logging` · `FortiGate Traffic Inspection` · `Fortinet Troubleshooting` · `FortiGate NSE` · `Network Security`

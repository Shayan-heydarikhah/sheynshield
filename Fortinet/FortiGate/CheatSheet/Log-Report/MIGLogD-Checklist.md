# FortiGate Logging, `miglogd` & `fgtlogd` Checklist

> **FortiOS Focus:** 7.2.x
> **Scope:** FortiGate Logging, Local Disk, FortiAnalyzer, FortiAnalyzer Cloud, Syslog, VDOM Logging Overrides, `miglogd`, `fgtlogd`, Log Filtering, Log Retention & Troubleshooting
> **SheynShield:** Security & Network Engineering Knowledge Base

---

## 📋 Table of Contents

* [1. Logging Architecture Checklist](#1-logging-architecture-checklist)
* [2. Local Logging Checklist](#2-local-logging-checklist)
* [3. Log Buffer & Historical FortiView Checklist](#3-log-buffer--historical-fortiview-checklist)
* [4. `miglogd` Checklist](#4-miglogd-checklist)
* [5. `fgtlogd` Checklist](#5-fgtlogd-checklist)
* [6. FortiAnalyzer Checklist](#6-fortianalyzer-checklist)
* [7. FortiAnalyzer Upload & Reliability Checklist](#7-fortianalyzer-upload--reliability-checklist)
* [8. FortiAnalyzer Cloud Checklist](#8-fortianalyzer-cloud-checklist)
* [9. Syslog & SIEM Checklist](#9-syslog--siem-checklist)
* [10. Log Filtering Checklist](#10-log-filtering-checklist)
* [11. Log Disk Management Checklist](#11-log-disk-management-checklist)
* [12. Log Retention Checklist](#12-log-retention-checklist)
* [13. Disk Full Protection Checklist](#13-disk-full-protection-checklist)
* [14. VDOM Logging Override Checklist](#14-vdom-logging-override-checklist)
* [15. Security Fabric Logging Checklist](#15-security-fabric-logging-checklist)
* [16. Logging Security Checklist](#16-logging-security-checklist)
* [17. Production Logging Architecture Checklist](#17-production-logging-architecture-checklist)
* [18. Local Logging Troubleshooting Checklist](#18-local-logging-troubleshooting-checklist)
* [19. FortiAnalyzer Troubleshooting Checklist](#19-fortianalyzer-troubleshooting-checklist)
* [20. Syslog Troubleshooting Checklist](#20-syslog-troubleshooting-checklist)
* [21. Log Rate Limiting Checklist](#21-log-rate-limiting-checklist)
* [22. NSE Exam & Interview Checklist](#22-nse-exam--interview-checklist)
* [23. Final Production Checklist](#23-final-production-checklist)
* [24. CLI Quick Reference](#24-cli-quick-reference)
* [25. SheynShield Mental Model](#25-sheynshield-mental-model)

---

# 1. Logging Architecture Checklist

## 🏗️ Core Architecture

* [ ] Identify all required log sources.
* [ ] Identify all required log destinations.
* [ ] Define local logging requirements.
* [ ] Define remote logging requirements.
* [ ] Define centralized logging requirements.
* [ ] Define SIEM/SOC integration requirements.
* [ ] Define log retention requirements.
* [ ] Define compliance/audit requirements.
* [ ] Define expected log volume.
* [ ] Define acceptable logging latency.
* [ ] Define failure behavior when a logging destination becomes unavailable.

### Logging Pipeline

```text
Traffic / Event
      │
      ▼
Log Generation
      │
      ▼
Log Filtering
      │
      ▼
Log Processing
      │
      ├──────────────┐
      ▼              ▼
Local Buffer      Remote Queue
      │              │
      ▼              ▼
Local Disk        FortiAnalyzer
                     │
                     ▼
                    SIEM
                     │
                     ▼
                   SOC
```

### Core Design Rule

* [ ] Do not treat FortiGate local logging as the only source of historical security evidence.
* [ ] Use centralized logging when long-term retention is required.
* [ ] Monitor both local storage and remote forwarding.
* [ ] Design logging around **visibility + investigation + retention**, not simply maximum log volume.

---

# 2. Local Logging Checklist

## 💾 Local Storage

* [ ] Verify whether the FortiGate platform supports local disk logging.
* [ ] Confirm the disk is detected.
* [ ] Confirm the disk is initialized/formatted where required.
* [ ] Confirm disk logging is enabled.
* [ ] Verify available disk capacity.
* [ ] Verify logging quota.
* [ ] Verify maximum log age.
* [ ] Verify disk-full behavior.
* [ ] Verify warning thresholds.
* [ ] Verify log rotation.
* [ ] Verify local report requirements.

### Disk Diagnostics

```bash
diagnose sys logdisk status
diagnose sys logdisk usage
diagnose sys logdisk quota
diagnose sys logdisk smart
```

### Verify

```text
[ ] Disk detected
[ ] Disk healthy
[ ] Disk available
[ ] Disk logging enabled
[ ] Quota understood
[ ] Retention understood
[ ] Rotation configured
[ ] Disk-full behavior reviewed
```

---

# 3. Log Buffer & Historical FortiView Checklist

## 🧠 Log Buffer

* [ ] Understand that FortiGate can use system memory for log buffering.
* [ ] Evaluate memory impact of high-volume logging.
* [ ] Avoid assuming memory buffering provides long-term retention.
* [ ] Verify local disk requirements separately from memory buffering.

> Log buffering behavior depends on FortiOS version, platform, logging destinations and configuration.

## 📊 Historical FortiView

* [ ] Confirm local disk logging when historical local data is required.
* [ ] Confirm sufficient storage for the required history.
* [ ] Confirm retention requirements.
* [ ] Confirm old logs are not being removed earlier than required.

### FortiView Diagnostics

```bash
diagnose fortiview result event-log
diagnose fortiview result security-log
diagnose fortiview result security-log action-block
```

### Time-Based Query

```bash
diagnose fortiview time 2026-11-20 00:00:00 2026-11-21 00:00:00
```

---

# 4. `miglogd` Checklist

## 🔎 Role

* [ ] Understand `miglogd` as a key component of local log processing.
* [ ] Investigate `miglogd` when local logging behaves unexpectedly.
* [ ] Check log buffers.
* [ ] Check log statistics.
* [ ] Check disk-related information.
* [ ] Check relevant caches.
* [ ] Check shared memory when required.

### Basic Diagnostics

```bash
diagnose test application miglogd 1
```

### VDOM-Specific Diagnostics

```bash
diagnose test application miglogd 1 vdom root
```

### High-Value Test IDs

|    ID | Check                   |
| ----: | ----------------------- |
|   `1` | Global log settings     |
|   `2` | VDOM log settings       |
|   `3` | Log buffer size         |
|   `4` | Log statistics          |
|   `6` | Statistics dump         |
|   `9` | Policy sniffer files    |
|  `11` | UTM traffic cache       |
|  `12` | Policy cache            |
|  `16` | Log disk usage          |
|  `18` | Network interface cache |
|  `19` | Application cache       |
|  `24` | WLAN AP cache           |
|  `27` | DNS cache               |
|  `28` | Shared memory           |
|  `31` | Log dump file content   |
|  `33` | Log dump files          |
|  `36` | Memory log file lists   |
|  `42` | UUID cache              |
|  `43` | ISDB cache              |
|  `47` | Remote sockets          |
|  `50` | Threat Feed cache       |
| `101` | Root VDOM log settings  |

### `miglogd` Troubleshooting

```text
[ ] Logging configuration checked
[ ] Log buffer checked
[ ] Log statistics checked
[ ] Disk usage checked
[ ] Relevant caches checked
[ ] Shared memory checked
[ ] VDOM-specific settings checked
```

---

# 5. `fgtlogd` Checklist

## 🔎 Role

Investigate `fgtlogd` when the problem involves:

* [ ] Remote log forwarding
* [ ] FortiAnalyzer synchronization
* [ ] Remote queues
* [ ] Log subscriptions
* [ ] Log filters
* [ ] Dropped logs
* [ ] Remote sockets
* [ ] FortiCloud logging state

### Basic Diagnostics

```bash
diagnose test application fgtlogd 1
diagnose test application fgtlogd 3
diagnose test application fgtlogd 5
diagnose test application fgtlogd 30
```

### High-Value Test IDs

|    ID | Purpose                           |
| ----: | --------------------------------- |
|   `1` | Global log settings/report        |
|   `2` | VDOM log settings                 |
|   `3` | Detailed log statistics           |
|   `4` | Statistics                        |
|   `5` | Dropped logs due to rate limiting |
|   `6` | Subscribed log information        |
|   `7` | Subscribed log filters            |
|   `9` | Remote socket                     |
|  `10` | Log packet dump                   |
|  `20` | FortiCloud log state              |
|  `30` | Remote queues/items               |
|  `37` | FAZ/FDS packet dumping            |
|  `38` | Delete FAZ/FDS dump files         |
|  `39` | Backup FAZ/FDS dump files         |
|  `41` | Remote queues                     |
| `101` | Root VDOM log settings            |

### Remote Logging Health

```text
[ ] Remote destination configured
[ ] Remote destination reachable
[ ] Authentication/authorization verified
[ ] Remote queue checked
[ ] Dropped-log counters checked
[ ] Subscription checked
[ ] Filter checked
[ ] Remote socket checked
[ ] Rate limiting checked
```

---

# 6. FortiAnalyzer Checklist

## 🏢 Basic Configuration

Navigate to:

```text
Log & Report
└── Log Settings
    └── Remote Logging and Archiving
```

### CLI

```bash
config log fortianalyzer setting
    set status enable
    set server 192.168.254.200
end
```

### Verify

* [ ] FortiAnalyzer IP/FQDN is correct.
* [ ] FortiAnalyzer is reachable.
* [ ] FortiGate is authorized where required.
* [ ] Certificate verification is configured appropriately.
* [ ] Correct source/interface is selected.
* [ ] Upload mode is appropriate.
* [ ] Log filters are appropriate.
* [ ] Remote queues are healthy.
* [ ] FortiAnalyzer compatibility is verified.

### Connectivity Test

```bash
execute log fortianalyzer test-connectivity
```

---

# 7. FortiAnalyzer Upload & Reliability Checklist

## 🚚 Upload Modes

| Mode               | Use                                     |
| ------------------ | --------------------------------------- |
| `realtime`         | Continuous log forwarding               |
| `1-minute`         | Approximate one-minute upload interval  |
| `5-minute`         | Approximate five-minute upload interval |
| `store-and-upload` | Store locally before upload             |

### Verify

* [ ] Upload mode selected based on operational requirements.
* [ ] Logging latency requirement documented.
* [ ] Local storage capacity evaluated when using `store-and-upload`.
* [ ] Remote connectivity failure scenario tested.
* [ ] Queue growth monitored.
* [ ] Log synchronization behavior understood.

### Store-and-Upload

```text
Traffic
   │
   ▼
FortiGate
   │
   ▼
Local Storage
   │
   │ Cache
   ▼
FortiAnalyzer
   │
   ▼
Centralized Logs
```

## 🔐 Reliable Logging

```bash
config log fortianalyzer setting
    set reliable enable
end
```

Verify:

* [ ] Reliable transmission requirement evaluated.
* [ ] Remote logging reliability requirements documented.
* [ ] Performance impact considered.

---

# 8. FortiAnalyzer Cloud Checklist

## ☁️ Cloud Logging

* [ ] FortiAnalyzer Cloud subscription verified.
* [ ] Contract/entitlement verified.
* [ ] Required logging features verified.
* [ ] Required retention verified.
* [ ] Cloud connectivity verified.
* [ ] Certificate verification evaluated.
* [ ] TLS requirements reviewed.
* [ ] Upload interval selected.
* [ ] Log filters reviewed.

### Example Configuration

```bash
config log fortianalyzer-cloud setting
    set status enable
    set certificate-verification enable
    set interface-select-method auto
    set upload-option 5-minute
    set max-log-rate 0
end
```

### Connectivity

```bash
execute log fortianalyzer-cloud test-connectivity
```

### Contract / Update Information

```bash
diagnose test update info
```

### Cloud Log Verification

```bash
execute log filter device fortianalyzer-cloud
execute log filter category traffic
execute log filter dump
execute log display
```

### Verify

```text
[ ] Subscription valid
[ ] Feature entitlement verified
[ ] Connectivity successful
[ ] TLS/security settings reviewed
[ ] Upload mode selected
[ ] Filters reviewed
[ ] Retention verified
```

> **Important:** FortiAnalyzer Cloud capabilities can change with subscription and product release. Verify the active entitlement before designing the architecture.

---

# 9. Syslog & SIEM Checklist

## 📡 Syslog

Example:

```bash
config log syslogd
    set status enable
    set format default
    set server 192.168.254.254
end
```

### Verify

* [ ] Syslog enabled.
* [ ] Correct server configured.
* [ ] Correct format selected.
* [ ] Facility configured where required.
* [ ] Required traffic categories enabled.
* [ ] Network path available.
* [ ] Transport requirements verified.
* [ ] Syslog listener verified.

## SIEM Integration

* [ ] SIEM requirements defined.
* [ ] Required log categories identified.
* [ ] Log format verified.
* [ ] CEF requirement evaluated.
* [ ] Parsing tested.
* [ ] Timestamp synchronization verified.
* [ ] Severity mapping verified.
* [ ] Duplicate logging evaluated.
* [ ] SIEM ingestion rate monitored.

### Common Architecture

```text
FortiGate
    │
    ▼
 Syslog
    │
    ▼
  SIEM
    │
 ┌──┼───────────┐
 ▼  ▼           ▼
SOC Correlation Alerting
```

---

# 10. Log Filtering Checklist

## 🎯 Filtering Strategy

* [ ] Define which logs are operationally required.
* [ ] Define which logs are security-critical.
* [ ] Define which logs are compliance-required.
* [ ] Avoid forwarding unnecessary high-volume categories.
* [ ] Verify severity filtering.
* [ ] Verify traffic-category filtering.
* [ ] Review filters after major configuration changes.

### FortiAnalyzer Filter

```bash
config log fortianalyzer filter
    set severity information
    set forward-traffic enable
    set local-traffic enable
    set multicast-traffic enable
    set sniffer-traffic enable
end
```

### Syslog Filter

```bash
config log syslogd filter
    set severity information
    set forward-traffic enable
    set local-traffic enable
    set multicast-traffic enable
    set sniffer-traffic enable
end
```

### Log Categories

Evaluate:

```text
[ ] Forward traffic
[ ] Local traffic
[ ] Multicast traffic
[ ] Sniffer traffic
[ ] IPS
[ ] Antivirus
[ ] Web Filter
[ ] Application Control
[ ] DNS
[ ] SSH
[ ] SSL
[ ] WAF
[ ] DLP
[ ] VoIP
[ ] Anomaly
[ ] Authentication
[ ] VPN
[ ] HA
[ ] System events
[ ] Administrative events
```

> **Production principle:** Log what you need to **detect, investigate, audit and prove** — not everything the firewall can generate.

---

# 11. Log Disk Management Checklist

## 💽 Disk Configuration

Example:

```bash
config log disk setting
    set status enable
    set ips-archive enable
    set log-quota 0
    set maximum-log-age 7
    set upload disable
    set full-first-warning-threshold 75
    set full-second-warning-threshold 90
    set full-final-warning-threshold 95
    set max-log-file-size 20
    set roll-schedule daily
    set roll-time 00:00
    set diskfull overwrite
end
```

### Verify

* [ ] Disk logging enabled.
* [ ] IPS archive requirement evaluated.
* [ ] Log quota understood.
* [ ] Maximum log age defined.
* [ ] Warning thresholds defined.
* [ ] Maximum file size reviewed.
* [ ] Roll schedule reviewed.
* [ ] Disk-full behavior defined.
* [ ] Upload requirement evaluated.

### Disk Diagnostics

```bash
diagnose sys logdisk usage
diagnose sys logdisk quota
diagnose sys logdisk smart
diagnose sys logdisk status
```

---

# 12. Log Retention Checklist

## ⏱️ Retention

* [ ] Define required retention period.
* [ ] Identify regulatory requirements.
* [ ] Identify security investigation requirements.
* [ ] Calculate expected daily log volume.
* [ ] Calculate required storage capacity.
* [ ] Account for growth.
* [ ] Account for temporary buffering.
* [ ] Account for upload failures.
* [ ] Verify FortiAnalyzer retention.
* [ ] Verify SIEM retention.
* [ ] Verify cloud retention.

### Retention Architecture

```text
FortiGate
   │
   ├── Short-Term Local Logs
   │
   ▼
FortiAnalyzer
   │
   ├── Medium/Long-Term Retention
   │
   ▼
SIEM / Archive
   │
   └── Compliance / SOC / Investigation
```

---

# 13. Disk Full Protection Checklist

## ⚠️ Disk Thresholds

Example:

```text
75%  → First Warning
90%  → Second Warning
95%  → Final Warning
100% → Disk-Full Action
```

### Verify

* [ ] First warning configured.
* [ ] Second warning configured.
* [ ] Final warning configured.
* [ ] Disk-full action documented.
* [ ] Alerting tested.
* [ ] Centralized logging remains available.

## Disk-Full Behavior

### Overwrite

```bash
set diskfull overwrite
```

* [ ] Older log data can be overwritten.
* [ ] Centralized logging is available if historical retention is required.

### No Logging

```bash
set diskfull nolog
```

* [ ] Logging interruption risk evaluated.
* [ ] Security visibility gap understood.
* [ ] Centralized logging configured as protection.

> **Security consideration:** A local disk reaching capacity should not become a single point of failure for security visibility.

---

# 14. VDOM Logging Override Checklist

## 🧩 Multi-VDOM Design

* [ ] Determine whether VDOM-specific destinations are required.
* [ ] Determine global logging destinations.
* [ ] Determine VDOM-specific destinations.
* [ ] Verify override configuration.
* [ ] Verify VDOM-specific filters.
* [ ] Document each VDOM destination.

### Enable Overrides

```bash
config log setting
    set faz-override enable
    set syslog-override enable
end
```

### Concept

```text
Global Logging
      │
      ├── VDOM-A
      │      └── FAZ Override
      │
      ├── VDOM-B
      │      └── Syslog Override
      │
      └── VDOM-C
             └── Global Destination
```

## FortiAnalyzer Override

```bash
config log fortianalyzer3 override-setting
    set status enable
    set server 123.12.123.123
    set reliable enable
end
```

## Syslog Override

```bash
config log syslogd4 override-setting
    set status enable
    set server 123.12.123.12
    set facility local1
end
```

### Verify

```text
[ ] Override enabled where required
[ ] Correct destination configured
[ ] VDOM filter configured
[ ] Destination reachable
[ ] Logs received
[ ] Global vs local behavior documented
```

> Platform and FortiOS release limits should always be verified before designing multiple override destinations.

---

# 15. Security Fabric Logging Checklist

## 🌐 Fabric Architecture

* [ ] Identify Security Fabric root FortiGate.
* [ ] Identify downstream FortiGates.
* [ ] Define centralized logging architecture.
* [ ] Define FortiAnalyzer integration.
* [ ] Verify reporting requirements.
* [ ] Avoid unnecessarily duplicating centralized logging functions.
* [ ] Verify root FortiGate reporting role.

### Recommended Model

```text
                    FortiAnalyzer
                         ▲
                         │
                  Central Logging
                         │
                  Root FortiGate
                         ▲
                         │
                 Security Fabric
             ┌───────────┼───────────┐
             ▼           ▼           ▼
           FGT-2       FGT-3       FGT-4
```

### Verify

```text
[ ] Root FortiGate identified
[ ] FortiAnalyzer connected
[ ] Downstream devices sending required logs
[ ] Central reporting verified
[ ] Retention verified
[ ] SIEM integration evaluated
```

---

# 16. Logging Security Checklist

## 🔐 Secure the Logging Infrastructure

* [ ] Protect FortiAnalyzer administrative access.
* [ ] Restrict access to log infrastructure.
* [ ] Use strong encryption where supported.
* [ ] Verify certificate validation requirements.
* [ ] Review TLS minimum version.
* [ ] Protect SIEM ingestion endpoints.
* [ ] Protect Syslog infrastructure.
* [ ] Synchronize system time.
* [ ] Protect log integrity.
* [ ] Define access controls for sensitive logs.
* [ ] Define privacy requirements for user-identifying logs.
* [ ] Define audit requirements.

### User Anonymization

```bash
set user-anonymize enable
```

Verify:

* [ ] Privacy requirements evaluated.
* [ ] SOC investigation requirements evaluated.
* [ ] User-identifying data handling documented.

---

# 17. Production Logging Architecture Checklist

## 🏢 Small Environment

```text
FortiGate
   │
   ├── Local Disk
   │
   └── Syslog
```

Checklist:

```text
[ ] Local logging configured
[ ] Syslog configured
[ ] Retention defined
[ ] Disk monitored
```

## 🏢 Medium Environment

```text
FortiGate
   │
   ├── Local Disk
   │
   └── FortiAnalyzer
```

Checklist:

```text
[ ] Local buffer/storage
[ ] Centralized logging
[ ] Retention
[ ] Reporting
[ ] Monitoring
```

## 🏢 Enterprise Environment

```text
                     FortiAnalyzer
                          ▲
                          │
                   Central Logging
                          │
                   Root FortiGate
                          │
                  Security Fabric
            ┌─────────────┼─────────────┐
            ▼             ▼             ▼
          FGT-2         FGT-3         FGT-4
            │             │             │
            └─────────────┼─────────────┘
                          ▼
                         SIEM
                          │
                         SOC
```

### Enterprise Checklist

* [ ] Central FortiAnalyzer deployed.
* [ ] Root FortiGate identified.
* [ ] Security Fabric logging validated.
* [ ] SIEM integration implemented where required.
* [ ] Retention policy documented.
* [ ] Local storage used as controlled short-term storage/buffer.
* [ ] Remote logging monitored.
* [ ] Logging failures alert the SOC/NOC.

---

# 18. Local Logging Troubleshooting Checklist

## 🚨 Problem: Local Logs Missing

### Step 1 — Configuration

```text
[ ] Is logging enabled?
[ ] Is the required log category enabled?
[ ] Is a log filter blocking the category?
[ ] Is the correct VDOM being checked?
```

### Step 2 — Disk

```bash
diagnose sys logdisk status
diagnose sys logdisk usage
diagnose sys logdisk quota
diagnose sys logdisk smart
```

Check:

```text
[ ] Disk detected
[ ] Disk healthy
[ ] Disk not full
[ ] Quota not exhausted
[ ] Retention not too short
```

### Step 3 — `miglogd`

```bash
diagnose test application miglogd 1
diagnose test application miglogd 4
diagnose test application miglogd 16
```

Check:

```text
[ ] Log settings
[ ] Log statistics
[ ] Disk usage
[ ] Buffer behavior
```

### Step 4 — Root Cause

```text
Configuration
      ↓
Filter
      ↓
Disk
      ↓
miglogd
      ↓
Log Generation
```

---

# 19. FortiAnalyzer Troubleshooting Checklist

## 🚨 Problem: FortiAnalyzer Not Receiving Logs

### Configuration

```text
[ ] FAZ IP/FQDN correct
[ ] Status enabled
[ ] Upload option correct
[ ] Required log categories enabled
[ ] Filter allows required logs
```

### Connectivity

```bash
execute log fortianalyzer test-connectivity
```

Check:

```text
[ ] Network connectivity
[ ] Authorization
[ ] Certificate
[ ] Interface selection
[ ] Routing
```

### `fgtlogd`

```bash
diagnose test application fgtlogd 1
diagnose test application fgtlogd 3
diagnose test application fgtlogd 5
diagnose test application fgtlogd 30
```

Check:

```text
[ ] Global logging state
[ ] Log statistics
[ ] Dropped logs
[ ] Remote queues
[ ] Remote sockets
```

### Troubleshooting Model

```text
FortiGate
   │
   ▼
Log Generation
   │
   ▼
Filter
   │
   ▼
fgtlogd
   │
   ▼
Remote Queue
   │
   ▼
Network
   │
   ▼
FortiAnalyzer
```

---

# 20. Syslog Troubleshooting Checklist

## 🚨 Problem: Syslog Not Receiving Logs

```text
[ ] Syslog status enabled
[ ] Correct server IP
[ ] Correct format
[ ] Correct facility
[ ] Correct filter
[ ] Required categories enabled
[ ] Network path available
[ ] Transport requirements verified
[ ] Syslog listener active
[ ] Firewall path allows traffic
[ ] SIEM/parser accepts the selected format
```

### Example

```bash
config log syslogd
    set status enable
    set format default
    set server 192.168.254.254
end
```

---

# 21. Log Rate Limiting Checklist

## 🚦 `max-log-rate`

Check:

```bash
diagnose test application fgtlogd 5
```

### Verify

* [ ] `max-log-rate` configured?
* [ ] Transmission limit appropriate?
* [ ] Dropped logs detected?
* [ ] Remote destination capacity evaluated?
* [ ] Log burst behavior understood?

### Example

```bash
set max-log-rate 0
```

Conceptually:

```text
0
│
└── No configured transmission-rate limit
```

> Always verify parameter semantics for the exact FortiOS release.

---

# 22. NSE Exam & Interview Checklist

## 🧠 `miglogd`

* [ ] Can explain the role of `miglogd`.
* [ ] Can identify local logging troubleshooting scenarios.
* [ ] Know how to inspect log statistics.
* [ ] Know how to inspect log disk usage.
* [ ] Know relevant `miglogd` diagnostic IDs.

### Remember

```text
miglogd
   ↓
Local Log Processing
   +
Buffers
   +
Caches
   +
Disk-related Operations
```

---

## 🧠 `fgtlogd`

* [ ] Can explain the role of `fgtlogd`.
* [ ] Can troubleshoot remote logging.
* [ ] Can inspect remote queues.
* [ ] Can inspect dropped logs.
* [ ] Can inspect subscriptions.
* [ ] Can inspect remote sockets.

### Remember

```text
fgtlogd
   ↓
Remote Logging
   +
Synchronization
   +
Queues
   +
Forwarding
```

---

## 🧠 Upload Modes

* [ ] `realtime` = continuous forwarding.
* [ ] `1-minute` = approximately one-minute upload cycle.
* [ ] `5-minute` = approximately five-minute upload cycle.
* [ ] `store-and-upload` = local storage/cache before upload.

---

## 🧠 VDOM Overrides

* [ ] `faz-override` = VDOM-specific FortiAnalyzer destination behavior.
* [ ] `syslog-override` = VDOM-specific Syslog destination behavior.

---

## 🧠 Disk Thresholds

```text
75%
 ↓
First Warning

90%
 ↓
Second Warning

95%
 ↓
Final Warning
```

---

## 🧠 Disk Full

* [ ] `overwrite` = older log data can be overwritten.
* [ ] `nolog` = new logging can stop when storage reaches the configured condition.

---

# 23. Final Production Checklist

## 🔥 SheynShield Production Logging Checklist

### Architecture

* [ ] Logging architecture documented.
* [ ] Log sources identified.
* [ ] Log destinations identified.
* [ ] Retention requirements defined.
* [ ] SIEM requirements defined.
* [ ] Compliance requirements defined.

### Local Logging

* [ ] Local disk detected.
* [ ] Disk health verified.
* [ ] Disk logging enabled where required.
* [ ] Log quota reviewed.
* [ ] Maximum log age reviewed.
* [ ] Disk-full behavior configured.
* [ ] Warning thresholds configured.
* [ ] Log rotation reviewed.

### FortiAnalyzer

* [ ] FortiAnalyzer configured.
* [ ] Connectivity verified.
* [ ] Authorization verified.
* [ ] Certificate validation reviewed.
* [ ] Upload mode selected.
* [ ] Reliable logging evaluated.
* [ ] Remote queue monitored.
* [ ] Log filters optimized.
* [ ] FortiOS/FortiAnalyzer compatibility verified.

### Syslog / SIEM

* [ ] Syslog configured where required.
* [ ] Correct format selected.
* [ ] Facility configured.
* [ ] SIEM parser tested.
* [ ] Required security logs forwarded.
* [ ] Log volume monitored.

### VDOM

* [ ] VDOM logging requirements documented.
* [ ] FAZ override configured where required.
* [ ] Syslog override configured where required.
* [ ] VDOM filters validated.
* [ ] Global/local logging behavior documented.

### Security Fabric

* [ ] Root FortiGate identified.
* [ ] Centralized logging architecture validated.
* [ ] Downstream FortiGate logging validated.
* [ ] FortiAnalyzer reporting verified.
* [ ] SIEM integration evaluated.

### Troubleshooting

* [ ] `miglogd` health checked.
* [ ] `fgtlogd` health checked.
* [ ] Disk usage checked.
* [ ] Disk health checked.
* [ ] Remote queues checked.
* [ ] Dropped logs checked.
* [ ] Filters checked.
* [ ] Connectivity checked.

---

# 24. CLI Quick Reference

## 💾 Local Disk

```bash
diagnose sys logdisk usage
diagnose sys logdisk quota
diagnose sys logdisk smart
diagnose sys logdisk status
```

## 🏢 FortiAnalyzer

```bash
execute log fortianalyzer test-connectivity

diagnose test application fgtlogd 1
diagnose test application fgtlogd 3
diagnose test application fgtlogd 5
diagnose test application fgtlogd 30
```

## ☁️ FortiAnalyzer Cloud

```bash
execute log fortianalyzer-cloud test-connectivity
diagnose test update info
```

## 🧠 `miglogd`

```bash
diagnose test application miglogd 1
diagnose test application miglogd 2
diagnose test application miglogd 3
diagnose test application miglogd 4
diagnose test application miglogd 16
diagnose test application miglogd 47
```

## 📊 FortiView

```bash
diagnose fortiview result event-log
diagnose fortiview result security-log
diagnose fortiview result security-log action-block
```

---

# 25. SheynShield Mental Model

## 🧠 The Complete Logging Pipeline

```text
                    FORTIGATE
                        │
                        ▼
                 Log Generation
                        │
                        ▼
                   Log Filter
                        │
                        ▼
                 Log Processing
                        │
             ┌──────────┴──────────┐
             │                     │
             ▼                     ▼
          miglogd               fgtlogd
             │                     │
             ▼                     ▼
       Buffer / Disk        Remote Queue
             │                     │
             ▼             ┌───────┼────────┐
         FortiView         ▼       ▼        ▼
                         FAZ     Cloud    Syslog
                           │       │        │
                           └───────┼────────┘
                                   ▼
                                  SIEM
                                   │
                                   ▼
                                  SOC
```

## 🎯 Golden Rule

```text
Generate
   ↓
Filter
   ↓
Process
   ↓
Buffer / Store
   ↓
Forward
   ↓
Analyze
   ↓
Retain
   ↓
Monitor
```

## 🔥 The SheynShield Rule

> **FortiGate logging is not simply "enable logs and send them to FortiAnalyzer."**

A production logging design must answer:

```text
What should be logged?
        ↓
Where should it be stored?
        ↓
What should be forwarded?
        ↓
How should it be filtered?
        ↓
What happens if the remote destination fails?
        ↓
How are dropped logs detected?
        ↓
How long must logs be retained?
        ↓
Who analyzes them?
        ↓
How are they integrated with SIEM/SOC?
```

### Preferred Enterprise Model

```text
FortiGate
    │
    ▼
FortiAnalyzer
    │
    ▼
SIEM
    │
    ▼
SOC
```

with:

```text
Local Disk
    =
Short-Term Storage / Buffer
```

rather than the only source of historical security evidence.

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

## ⭐ SheynShield

**SheynShield | Engineering Secure Networks**

> Practical Network Security • Fortinet • Firewall Engineering • Troubleshooting • Secure Network Design

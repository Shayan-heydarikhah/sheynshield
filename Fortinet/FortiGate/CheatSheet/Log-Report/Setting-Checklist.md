# FortiGate Logging & Log Management Checklist

> **FortiOS | Logging • Log Storage • FortiAnalyzer • Syslog • FortiSIEM • FortiView • Troubleshooting**
>
> **SheynShield — Engineering Secure Networks**

---

## 📋 Checklist Overview

* [ ] Understand FortiGate logging architecture
* [ ] Select appropriate log destinations
* [ ] Configure local disk logging where required
* [ ] Configure FortiAnalyzer centralized logging
* [ ] Configure Syslog / SIEM integration
* [ ] Configure log filters
* [ ] Configure retention and disk thresholds
* [ ] Validate log forwarding
* [ ] Validate FortiView visibility
* [ ] Monitor logging health
* [ ] Test failure and recovery scenarios
* [ ] Document the logging architecture

---

# 1. 🧠 Logging Architecture

### Identify the required logging destinations

* [ ] System memory
* [ ] Local disk
* [ ] FortiAnalyzer
* [ ] FortiManager
* [ ] FortiCloud
* [ ] Syslog
* [ ] FortiSIEM
* [ ] Other SIEM / logging platform

### Verify the architecture

```text
                         FortiGate
                            │
                 Traffic / Event / UTM
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
          Memory        Local Disk     Remote Logs
                                           │
                    ┌──────────┬───────────┼──────────┐
                    ▼          ▼           ▼          ▼
                   FAZ       FMG        Syslog     SIEM
```

* [ ] Logging requirements are documented
* [ ] Centralized logging is available for production
* [ ] Local storage is not the only logging destination
* [ ] Log retention requirements are documented
* [ ] Compliance requirements are considered
* [ ] Incident-response requirements are considered

> **Golden Rule:** Don't design logging around storage alone. Design it around **security visibility and investigation requirements**.

---

# 2. 💾 Memory Logging

* [ ] Determine whether memory logging is enabled
* [ ] Understand that memory logging is temporary
* [ ] Do not use memory as the only long-term log repository
* [ ] Verify remote forwarding for security-critical logs
* [ ] Confirm expected log visibility after reboot

### Mental Model

```text
Generate Log
     │
     ▼
Memory Buffer
     │
     ▼
Remote Destination
```

* [ ] Temporary buffering requirement identified
* [ ] Persistent storage requirement identified

---

# 3. 💽 Local Disk Logging

### GUI

```text
Log & Report
└── Log Settings
    └── Local Logs
        └── Disk
```

### CLI

```bash
config log disk setting
    set status enable
end
```

* [ ] Local disk logging enabled when required
* [ ] Disk capacity verified
* [ ] Disk health verified
* [ ] Retention period configured
* [ ] Disk-full behavior configured
* [ ] Warning thresholds configured
* [ ] Log rotation reviewed

### Verify

```bash
diagnose sys logdisk status
diagnose sys logdisk usage
diagnose sys logdisk quota
diagnose sys logdisk smart
```

---

# 4. 🏢 Security Fabric Logging

* [ ] Identify the Security Fabric root FortiGate
* [ ] Identify downstream FortiGates
* [ ] Determine where logs are stored
* [ ] Determine where logs are centrally analyzed
* [ ] Avoid unnecessary independent logging architectures
* [ ] Verify FortiAnalyzer integration
* [ ] Verify local-storage requirements for downstream devices

### Recommended Model

```text
                 Root FortiGate
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
        FGT-2        FGT-3        FGT-4
          │            │            │
          └────────────┼────────────┘
                       ▼
                 FortiAnalyzer
```

---

# 5. 📊 Local Reports

* [ ] Determine whether local reports are required
* [ ] Verify local log data exists
* [ ] Check available disk space
* [ ] Check CPU impact
* [ ] Check memory impact
* [ ] Check disk I/O impact
* [ ] Avoid unnecessary local report generation when centralized reporting already exists

> **Design Principle:** If FortiAnalyzer already provides centralized reporting, don't create unnecessary reporting workload on every FortiGate.

---

# 6. 📈 Historical FortiView

* [ ] Verify local disk logging
* [ ] Verify historical log data exists
* [ ] Verify retention period
* [ ] Confirm historical FortiView requirements for the target FortiOS release
* [ ] Verify enough disk capacity exists

### Flow

```text
Logs
  │
  ▼
Local Storage
  │
  ▼
Historical FortiView
```

---

# 7. 🟢 FortiAnalyzer Integration

### Configure

```bash
config log fortianalyzer setting
    set status enable
    set server <FORTIANALYZER-IP>
end
```

* [ ] FortiAnalyzer IP/FQDN is correct
* [ ] Routing to FortiAnalyzer works
* [ ] Required network connectivity exists
* [ ] FortiGate is authorized
* [ ] Certificate requirements are satisfied
* [ ] Time synchronization is correct
* [ ] Logs are arriving at FortiAnalyzer
* [ ] Historical search works
* [ ] Reports work
* [ ] Retention is sufficient

### Validation

```text
FortiGate
   │
   ├── Reachable?
   ├── Authorized?
   ├── Time synchronized?
   ├── Certificate valid?
   └── Logs arriving?
          │
          ▼
      FortiAnalyzer
```

---

# 8. 🔄 FortiAnalyzer Upload Mode

Identify the configured upload behavior:

* [ ] Real-time
* [ ] Every minute
* [ ] Every 5 minutes
* [ ] Store & Upload

### Store & Upload

```bash
config log fortianalyzer setting
    set upload-option store-and-upload
end
```

* [ ] Local caching requirement identified
* [ ] Temporary FAZ connectivity loss considered
* [ ] Local disk capacity checked
* [ ] Recovery behavior tested

### Store & Upload Model

```text
FortiGate
    │
    ▼
Local Cache
    │
    │ Connection restored
    ▼
FortiAnalyzer
```

> **NSE Memory:** `store-and-upload` introduces a local storage/cache stage before forwarding.

---

# 9. 📦 Local Disk Retention

Review:

* [ ] Disk size
* [ ] Daily log volume
* [ ] Enabled log categories
* [ ] DLP archives
* [ ] IPS archives
* [ ] Packet captures
* [ ] Reports
* [ ] Log quotas
* [ ] Maximum log age
* [ ] Rotation schedule
* [ ] Disk-full behavior

### Retention Model

```text
Log Volume
    +
Disk Capacity
    +
Retention
    +
Archives
    ↓
Actual Retention Period
```

> Never assume a specific number of days of retention without considering actual log volume and storage capacity.

---

# 10. ⏳ Maximum Log Age

Example:

```bash
set maximum-log-age 7
```

* [ ] Required retention period identified
* [ ] Compliance requirement checked
* [ ] Incident-response requirement checked
* [ ] FortiAnalyzer retention checked
* [ ] Local disk capacity checked
* [ ] Backup/archive requirements checked

### Mental Model

```text
Day 1 ─ Day 2 ─ Day 3 ─ ... ─ Day 7
                                │
                                ▼
                         Rotation / Removal
```

---

# 11. 🚨 Disk-Full Behavior

Review:

```bash
set diskfull overwrite
```

* [ ] Disk-full behavior explicitly reviewed
* [ ] Overwrite behavior understood
* [ ] Logging continuity requirement identified
* [ ] Compliance requirement considered
* [ ] Alerting configured before disk exhaustion

### Decision

```text
Disk Full
   │
   ├── Overwrite
   │      └── Preserve new logging
   │          but lose older data
   │
   └── Stop / No Logging
          └── Preserve existing data
              but lose new events
```

> **Security decision:** Decide whether preserving historical logs or maintaining continuous logging is more important for the specific environment.

---

# 12. ⚠️ Disk Warning Thresholds

Example:

```bash
set full-first-warning-threshold 75
set full-second-warning-threshold 90
set full-final-warning-threshold 95
```

* [ ] First warning configured
* [ ] Second warning configured
* [ ] Critical warning configured
* [ ] Alert destination verified
* [ ] SOC/NOC receives alerts
* [ ] Escalation process documented

### Recommended Monitoring

```text
0%
 │
 ├────────────── 75%  → Warning
 │
 ├──────────────────── 90%  → Serious
 │
 ├────────────────────────── 95%  → Critical
 │
 └────────────────────────────── 100% → Capacity
```

---

# 13. 🔁 Log Rotation

Example:

```bash
set roll-schedule daily
set roll-time 00:00
```

* [ ] Rotation schedule reviewed
* [ ] Rotation time selected
* [ ] Storage impact reviewed
* [ ] Retention behavior tested

### Flow

```text
Current Log
     │
     ▼
Rotation
     │
     ▼
New Log File
     │
     ▼
Continue Logging
```

---

# 14. 🗃️ Log Quota

Review:

```bash
set log-quota 0
```

* [ ] Log quota understood
* [ ] Available physical disk checked
* [ ] Other archive quotas reviewed
* [ ] Report quota reviewed
* [ ] DLP archive quota reviewed

> **NSE Warning:** Do not automatically interpret `0` as "infinite physical storage." Actual storage remains limited by the platform and available disk.

---

# 15. 📡 Syslog Integration

### Configure

```bash
config log syslogd setting
    set status enable
    set server <SYSLOG-SERVER-IP>
end
```

* [ ] Syslog server IP/FQDN verified
* [ ] Routing verified
* [ ] Port verified
* [ ] Protocol verified
* [ ] Format verified
* [ ] Log filter reviewed
* [ ] Logs received by Syslog server
* [ ] SIEM parser validated

### Typical Architecture

```text
FortiGate
    │
    ▼
Syslog
    │
    ▼
SIEM
    │
    ├── Parsing
    ├── Correlation
    ├── Detection
    ├── Alerting
    └── Investigation
```

---

# 16. 🔤 Syslog Format

Review supported formats for the target FortiOS release:

* [ ] Default
* [ ] CSV
* [ ] CEF
* [ ] Other supported format

### CEF Integration

```text
FortiGate
    │
    ▼
CEF
    │
    ▼
SIEM
```

* [ ] SIEM parser supports selected format
* [ ] Fields map correctly
* [ ] Timestamp is parsed correctly
* [ ] Source/destination fields are parsed
* [ ] Action field is parsed
* [ ] Policy information is retained
* [ ] Severity mapping is validated

> Always verify exact format and field support against the target FortiOS release.

---

# 17. 🎯 Log Filters

* [ ] FortiAnalyzer filter reviewed
* [ ] Syslog filter reviewed
* [ ] Severity configured
* [ ] Traffic categories configured
* [ ] Event categories configured
* [ ] Security categories configured
* [ ] Local traffic requirement reviewed
* [ ] Multicast traffic requirement reviewed
* [ ] Sniffer traffic requirement reviewed

### Filtering Model

```text
All Generated Logs
        │
        ▼
    Log Filter
        │
   ┌────┼────┐
   ▼    ▼    ▼
  FAZ  SIEM Email
```

### Avoid

```text
Everything
   │
   ▼
SIEM
   │
   ├── High storage
   ├── High parsing load
   ├── High network usage
   └── Alert noise
```

### Prefer

```text
Relevant Logs
     │
     ▼
SIEM
     │
     ▼
Better Signal / Noise Ratio
```

---

# 18. 🔐 Security Event Logging

Verify required security categories:

* [ ] Antivirus

* [ ] IPS

* [ ] Web Filter

* [ ] Application Control

* [ ] DNS Filter

* [ ] SSL Inspection

* [ ] SSH

* [ ] Authentication

* [ ] Administrator activity

* [ ] HA events

* [ ] System events

* [ ] Security-relevant logs are not disabled only to reduce volume

* [ ] Required categories are documented

---

# 19. 🔀 Forward vs Local Traffic

Understand the difference:

| Log Type          | Meaning                                       |
| ----------------- | --------------------------------------------- |
| Forward Traffic   | Traffic traversing FortiGate                  |
| Local Traffic     | Traffic to/from FortiGate itself              |
| Multicast Traffic | Multicast-related traffic                     |
| Sniffer Traffic   | Traffic associated with sniffer functionality |

### Forward Traffic

```text
Client
  │
  ▼
FortiGate
  │
  ▼
Server / Internet
```

### Local Traffic

```text
Client
  │
  ▼
FortiGate
  │
  └── FortiGate itself
```

* [ ] Forward traffic requirement identified
* [ ] Local traffic requirement identified
* [ ] Log volume impact considered

---

# 20. 🆔 Policy UUID Logging

Example:

```bash
config log setting
    set policy-uuid enable
end
```

* [ ] Policy UUID logging requirement identified
* [ ] UUID appears in traffic logs
* [ ] Logs can be correlated with firewall policies
* [ ] SIEM parser handles UUID fields

### Correlation

```text
Traffic Log
     │
     └── Policy UUID
              │
              ▼
       Firewall Policy
```

---

# 21. 🆔 Address UUID Correlation

Where supported:

* [ ] Address UUID logging reviewed
* [ ] Source object correlation tested
* [ ] Destination object correlation tested
* [ ] Automation requirements considered
* [ ] SIEM/API parsing validated

Useful for:

* [ ] Automation
* [ ] Troubleshooting
* [ ] Log analysis
* [ ] API correlation
* [ ] Security investigations

---

# 22. 📊 FortiView Validation

Useful diagnostics:

```bash
diagnose fortiview result event-log
```

```bash
diagnose fortiview result security-log
```

```bash
diagnose fortiview result security-log action-block
```

* [ ] Event logs visible
* [ ] Security logs visible
* [ ] Blocked actions visible
* [ ] Correct timeframe selected
* [ ] Source filtering works
* [ ] Destination filtering works
* [ ] Policy filtering works

### Investigation Flow

```text
Identify Time
      ↓
FortiView
      ↓
Source / Destination
      ↓
Action
      ↓
Policy
      ↓
Security Profile
      ↓
FortiAnalyzer / SIEM
```

---

# 23. 🔎 Log Forwarding Verification

Use:

```bash
execute log list 0
```

Then verify:

* [ ] Logging configuration
* [ ] Destination status
* [ ] Log generation
* [ ] Log filtering
* [ ] Network connectivity
* [ ] Remote reception

### Complete Chain

```text
Generate
   ↓
Filter
   ↓
Store / Buffer
   ↓
Forward
   ↓
Receive
   ↓
Index
   ↓
Search
   ↓
Analyze
   ↓
Alert / Report
```

> **Troubleshooting principle:** Find the exact stage where the logging chain breaks.

---

# 24. ⏰ Time Synchronization

* [ ] NTP configured
* [ ] NTP server reachable
* [ ] Timezone correct
* [ ] System clock correct
* [ ] FortiAnalyzer time synchronized
* [ ] SIEM time synchronized

### Why?

Incorrect time can break:

* [ ] Event correlation
* [ ] Incident timelines
* [ ] Authentication analysis
* [ ] Certificate validation
* [ ] SIEM correlation
* [ ] Historical investigation

---

# 25. 📧 Email Alerting

Review alert configuration:

```bash
config alertemail setting
```

Potential categories:

* [ ] IPS

* [ ] HA

* [ ] Antivirus

* [ ] Web Filter

* [ ] SSH

* [ ] FDS updates

* [ ] Administrator login

* [ ] Disk usage

* [ ] Other critical system events

* [ ] SMTP configuration verified

* [ ] Recipient verified

* [ ] Alert category selected

* [ ] Test alert generated

* [ ] Alert received

---

# 26. 💽 Disk Usage Alerting

* [ ] Warning threshold configured
* [ ] Critical threshold configured
* [ ] Email/SIEM/SOC notification configured
* [ ] Escalation procedure documented
* [ ] Test alert performed

### Example

```text
75% → Warning
90% → Serious
95% → Critical
100% → Capacity reached
```

> Never wait for 100% before discovering a logging problem.

---

# 27. 📦 FTP Archival

Where supported and required:

```bash
config log disk setting
    set upload enable
    set uploadport 21
    set upload-destination ftp-server
    set uploadtype traffic event virus webfilter ips
    set uploadsched enable
    set uploadtime 00:00
    set upload-delete-files enable
end
```

* [ ] FTP requirement identified
* [ ] FTP server reachable
* [ ] Upload schedule verified
* [ ] Uploaded categories verified
* [ ] File deletion behavior reviewed
* [ ] Security of the archival path reviewed

### Important

```text
FTP
 │
 └── File Archival

FortiAnalyzer
 │
 ├── Search
 ├── Analytics
 ├── Reports
 ├── Investigation
 └── Centralized Storage
```

> FTP archival should not automatically be treated as a replacement for centralized security analytics.

---

# 28. 🏗️ Enterprise Logging Architecture

### Recommended baseline

```text
                    FortiGate
                       │
            ┌──────────┼──────────┐
            ▼          ▼          ▼
       Local Cache     FAZ      Syslog
                          │         │
                          ▼         ▼
                     Analytics    SIEM
                          │         │
                          └────┬────┘
                               ▼
                              SOC
```

Checklist:

* [ ] Centralized logging deployed
* [ ] Local cache configured where appropriate
* [ ] FortiAnalyzer deployed
* [ ] SIEM integration deployed where required
* [ ] Redundant destination considered
* [ ] Retention documented
* [ ] Monitoring implemented
* [ ] Alerting implemented

---

# 29. 🛡️ High-Availability Logging

For critical environments:

* [ ] Primary logging destination configured
* [ ] Secondary logging destination considered
* [ ] Failure scenario tested
* [ ] Log buffering verified
* [ ] Recovery synchronization verified
* [ ] No single logging destination creates a complete visibility gap

### Resilient Model

```text
                 FortiGate
                /        \
               /          \
              ▼            ▼
          FortiAnalyzer   Syslog/SIEM
              │              │
              ▼              ▼
           Storage          SOC
```

---

# 30. 🚨 Missing Logs — Troubleshooting Checklist

If expected logs are missing:

### Logging

* [ ] Is logging enabled?
* [ ] Is the correct log category enabled?
* [ ] Is the relevant firewall policy generating logs?

### Filter

* [ ] Is severity filtering the event?
* [ ] Is traffic filtering the event?
* [ ] Is event/security filtering the event?

### Storage

* [ ] Is local disk available?
* [ ] Is disk full?
* [ ] Is quota exhausted?
* [ ] Is retention deleting the expected data?

### Network

* [ ] Can FortiGate reach FortiAnalyzer?
* [ ] Can FortiGate reach Syslog?
* [ ] Is routing correct?
* [ ] Are required ports/services reachable?

### Destination

* [ ] Is FortiAnalyzer authorized?
* [ ] Is Syslog receiving data?
* [ ] Is SIEM parsing the data?

### Time

* [ ] Is NTP working?
* [ ] Is timezone correct?
* [ ] Is the system clock correct?

### Application

* [ ] Is FortiView showing the event?
* [ ] Is FortiAnalyzer showing the event?
* [ ] Is SIEM showing the event?

---

# 31. 🧪 Logging Failure Test

Perform controlled testing:

* [ ] Generate known traffic
* [ ] Confirm traffic log generation
* [ ] Confirm local visibility
* [ ] Confirm FortiAnalyzer reception
* [ ] Confirm Syslog reception
* [ ] Confirm SIEM parsing
* [ ] Temporarily interrupt remote logging path
* [ ] Confirm expected local buffering behavior
* [ ] Restore connectivity
* [ ] Verify recovery/upload
* [ ] Confirm no unexpected visibility gap

### Test Model

```text
Generate Event
      ↓
FortiGate
      ↓
Local / Buffer
      ↓
Remote Destination
      ↓
SIEM / FAZ
      ↓
Search
      ↓
Confirm
```

---

# 32. 📋 Logging Deployment Validation

Before production:

* [ ] Logging requirements documented
* [ ] Log destinations selected
* [ ] Local disk requirement determined
* [ ] FortiAnalyzer configured
* [ ] Syslog/SIEM configured
* [ ] Filters configured
* [ ] Retention configured
* [ ] Disk thresholds configured
* [ ] NTP configured
* [ ] Alerting configured
* [ ] FortiView validated
* [ ] Remote log reception validated
* [ ] Failure scenario tested
* [ ] Recovery scenario tested
* [ ] Documentation completed

---

# 33. 🧠 NSE4 / NSE7 High-Value Memory

### Remember the logging destinations

```text
Memory
Disk
FAZ
FMG
FortiCloud
Syslog
FortiSIEM
```

### Remember the forwarding modes

```text
Real-Time
Periodic
Store & Upload
```

### Remember the logging chain

```text
Generate
  ↓
Filter
  ↓
Buffer / Store
  ↓
Forward
  ↓
Receive
  ↓
Index
  ↓
Search
  ↓
Analyze
```

### Remember the disk controls

```text
Quota
Log Age
Rotation
Warning Threshold
Disk-Full Behavior
```

---

# 34. ⚡ Quick CLI Checklist

### Local Disk

```bash
config log disk setting
    set status enable
end
```

### FortiAnalyzer

```bash
config log fortianalyzer setting
    set status enable
    set server <FAZ-IP>
    set upload-option store-and-upload
end
```

### Syslog

```bash
config log syslogd setting
    set status enable
    set server <SYSLOG-IP>
end
```

### FortiAnalyzer Filter

```bash
config log fortianalyzer filter
    set severity <LEVEL>
    set forward-traffic enable
    set local-traffic enable
end
```

### Syslog Filter

```bash
config log syslogd filter
    set severity <LEVEL>
    set forward-traffic enable
    set local-traffic enable
end
```

### Disk Diagnostics

```bash
diagnose sys logdisk usage
diagnose sys logdisk quota
diagnose sys logdisk smart
diagnose sys logdisk status
```

### FortiView

```bash
diagnose fortiview result event-log
diagnose fortiview result security-log
diagnose fortiview result security-log action-block
```

### Log Information

```bash
execute log list 0
```

---

# 35. 🎯 Logging Decision Matrix

| Requirement                    | Preferred Approach                |
| ------------------------------ | --------------------------------- |
| Temporary troubleshooting      | Memory                            |
| Short-term local storage       | Local Disk                        |
| Centralized enterprise logging | FortiAnalyzer                     |
| SIEM integration               | Syslog / FortiSIEM                |
| Cloud logging                  | FortiCloud                        |
| Centralized reporting          | FortiAnalyzer                     |
| Temporary remote failure       | Store & Upload                    |
| Long-term retention            | FortiAnalyzer / external platform |
| High logging resilience        | Multiple destinations             |
| Simple file archival           | FTP where appropriate             |

---

# 36. 🔥 SheynShield Golden Rules

> ### Rule #1
>
> **A firewall without usable logs is difficult to investigate.**

> ### Rule #2
>
> **Logging ≠ Reporting ≠ SIEM.**

```text
Logging
  ↓
Collect

FortiAnalyzer
  ↓
Store + Search + Analyze + Report

SIEM
  ↓
Correlate + Detect + Alert + Investigate
```

> ### Rule #3
>
> **Don't send everything to the SIEM blindly. Control log volume without sacrificing security visibility.**

> ### Rule #4
>
> **Store & Upload provides a useful local-cache model when centralized logging is temporarily unavailable.**

> ### Rule #5
>
> **Monitor disk usage before it reaches a critical state.**

> ### Rule #6
>
> **Always troubleshoot the complete logging chain instead of checking only the destination.**

> ### Rule #7
>
> **Never publish real passwords, API keys, certificates with private keys, tokens, or other secrets in GitHub.**

---

# 37. 🚀 Production Readiness Checklist

### Architecture

* [ ] Central logging selected
* [ ] Logging destinations documented
* [ ] Redundancy evaluated
* [ ] Retention requirement documented

### FortiGate

* [ ] Required log categories enabled
* [ ] Local storage configured where required
* [ ] Disk thresholds configured
* [ ] Disk-full behavior reviewed
* [ ] Log filters configured

### FortiAnalyzer

* [ ] Connectivity verified
* [ ] Authorization verified
* [ ] Logs received
* [ ] Search validated
* [ ] Reports validated
* [ ] Retention validated

### SIEM

* [ ] Syslog connectivity verified
* [ ] Format selected
* [ ] Parser validated
* [ ] Fields mapped
* [ ] Correlation tested
* [ ] Alerts tested

### Operations

* [ ] NTP verified
* [ ] Disk alerts verified
* [ ] Monitoring configured
* [ ] Failure test completed
* [ ] Recovery test completed
* [ ] Runbook documented

---

# 38. 🎤 60-Second Interview Answer

> **How would you design FortiGate logging for an enterprise?**

```text
I would avoid relying exclusively on FortiGate memory or local storage.

I would use FortiAnalyzer as the centralized logging and reporting platform,
configure appropriate log filters to control unnecessary log volume, and use
local storage as a cache or short-term repository where required.

For SIEM integration, I would forward the required events through Syslog
using an appropriate supported format.

I would also monitor disk utilization, retention, FortiAnalyzer
authorization and connectivity, NTP synchronization, and logging filters.

For critical environments, I would consider redundant logging paths and
test both failure and recovery scenarios to prevent a complete visibility
gap.
```

---

# 🔗 SheynShield Resources

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

## 🔗 Related SheynShield Topics

* [ ] FortiGate Firewall Policy
* [ ] FortiGate Policy & Objects
* [ ] FortiGate Authentication
* [ ] FortiGate FSSO
* [ ] FortiGate RADIUS
* [ ] FortiGate LDAP
* [ ] FortiGate HA
* [ ] FortiGate Security Profiles
* [ ] FortiGate FortiView
* [ ] FortiAnalyzer
* [ ] FortiManager
* [ ] FortiSIEM
* [ ] Syslog & SIEM Integration
* [ ] FortiGate Troubleshooting

---

## 🏷️ Tags

`FortiGate` `FortiOS` `Fortinet` `FortiGate-Logging` `Log-Management` `FortiAnalyzer` `FortiManager` `FortiCloud` `Syslog` `FortiSIEM` `FortiView` `SIEM` `NetworkSecurity` `CyberSecurity` `NSE4` `NSE7` `Firewall` `SecurityOperations` `SOC` `SecurityMonitoring`

---

**SheynShield — Engineering Secure Networks**

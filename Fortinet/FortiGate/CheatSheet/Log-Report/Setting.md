# FortiGate Logging & Log Management  

> **FortiOS Focus:** Logging, Local Disk, FortiAnalyzer, FortiManager, FortiCloud, Syslog, FortiSIEM, Log Filters, FortiView
> **Brand:** SheynShield — Engineering Secure Networks
> **Audience:** NSE4 / NSE7 / Network Security Engineers / FortiGate Administrators

---

## 1. Logging Architecture — Mental Model

FortiGate can generate and store/forward logs through several destinations:

```text
                         ┌─────────────────────┐
                         │      FortiGate      │
                         │                     │
                         │  Traffic / Event    │
                         │  Security / UTM     │
                         └──────────┬──────────┘
                                    │
              ┌─────────────────────┼─────────────────────┐
              │                     │                     │
              ▼                     ▼                     ▼
       System Memory          Local Disk          Remote Logging
                                                      │
                            ┌──────────────┬──────────┼──────────┐
                            ▼              ▼          ▼          ▼
                       FortiAnalyzer  FortiManager  Syslog   FortiSIEM
                       FortiAnalyzer
                          Cloud
                            │
                            ▼
                       Reporting /
                       Analytics
```

### Core destinations

| Destination       | Main purpose                                    |
| ----------------- | ----------------------------------------------- |
| **System Memory** | Temporary log buffering                         |
| **Local Disk**    | Local persistent log storage                    |
| **FortiAnalyzer** | Centralized logging, analytics and reporting    |
| **FortiManager**  | Centralized management and logging capabilities |
| **FortiCloud**    | Cloud-based logging/reporting                   |
| **Syslog**        | External log collection / SIEM integration      |
| **FortiSIEM**     | SIEM / security analytics                       |

> **Design principle:** For production environments, centralized logging is generally preferred over relying exclusively on local FortiGate storage.

---

# 2. Log Buffering & Memory

A portion of system memory can be used for log buffering and forwarding.

```text
FortiGate
   │
   ├── Generate Log
   │
   ├── Memory Buffer
   │
   └── Forward
         │
         ├── FortiAnalyzer
         ├── FortiManager
         ├── Syslog
         └── Other logging destinations
```

### Important

* Memory-based logging is not a replacement for persistent log storage.
* Memory logs are temporary.
* Logging behavior depends on the enabled log destination and configuration.
* Performance statistics are not necessarily stored as ordinary disk logs.

> **Operational rule:** If logs are required for investigation, compliance, or long-term historical analysis, use centralized log storage.

---

# 3. Local Disk Logging

Local disk logging allows FortiGate to store logs on its internal/local storage.

## Enable Local Disk Logging

```text
Log & Report
└── Log Settings
    └── Local Logs
        └── Disk
            └── Enable
```

### CLI

```bash
config log disk setting
    set status enable
end
```

---

## Local Disk Logging — Important Behavior

When enabled:

```text
Traffic/Event
     │
     ▼
FortiGate
     │
     ▼
Local Disk
```

When disabled:

```text
Traffic/Event
     │
     ▼
Remote Logging
     │
     ├── FortiAnalyzer
     ├── FortiManager
     ├── FortiCloud
     └── Syslog
```

### Security Fabric consideration

When Security Fabric is enabled, local disk logging behavior and GUI availability can differ between the **root FortiGate** and downstream FortiGates.

A common architecture is:

```text
                  Root FortiGate
                ┌───────────────┐
                │ Local Storage │
                │ + FAZ         │
                └───────┬───────┘
                        │
          ┌─────────────┴─────────────┐
          ▼                           ▼
    Downstream FGT              Downstream FGT
```

### Recommended architecture

For larger Security Fabric deployments:

* Use the root FortiGate as the primary logging point where appropriate.
* Prefer centralized FortiAnalyzer storage.
* Avoid designing the entire logging architecture around local disk on every downstream device.

---

# 4. Local Reports

Local reports allow FortiGate to generate reports from locally available log data.

```text
Log Data
   │
   ▼
Local Storage
   │
   ▼
Local Reports
```

### Enable

```text
Log & Report
└── Local Reports
```

Depending on the FortiOS version and Security Fabric configuration, local report visibility may require enabling the relevant feature.

### Resource consideration

Local reporting can consume:

* CPU
* Memory
* Disk I/O
* Disk space

> **Best practice:** If FortiAnalyzer is already being used for centralized reporting, avoid unnecessary local report generation unless there is a specific operational requirement.

---

# 5. Historical FortiView

Historical FortiView provides historical visualization of log data.

```text
Logs
  │
  ▼
Local Disk
  │
  ▼
Historical FortiView
```

### Requirements

Typically:

* Local disk logging must be enabled.
* Historical log data must exist locally.
* Retention depends on available storage and configured log retention.

### Important

Historical data retention is not unlimited.

A common default behavior is that older local historical data is removed after a defined retention period, depending on the FortiOS release/configuration.

---

# 6. FortiAnalyzer / FortiManager Remote Logging

For centralized logging:

```text
FortiGate
    │
    │ Remote Logging
    ▼
FortiAnalyzer / FortiManager
    │
    ├── Log Search
    ├── Reports
    ├── Analytics
    ├── Historical Investigation
    └── Security Monitoring
```

### GUI

```text
Log & Report
└── Log Settings
    └── Remote Logging and Archiving
```

### CLI example

```bash
config log fortianalyzer setting
    set status enable
    set server <FORTIANALYZER-IP>
end
```

---

# 7. FortiAnalyzer Connection Status

The connection status indicates whether FortiGate can communicate with and authorize against FortiAnalyzer.

### Successful state

```text
FortiGate
   │
   │ Authorized
   ▼
FortiAnalyzer
```

Check:

* Network reachability
* Correct FortiAnalyzer IP
* Authorization
* Certificates
* Time synchronization
* Routing
* Firewall policies
* Required ports/services

### Connectivity test

Use the GUI connectivity test or appropriate diagnostic commands for the specific FortiOS release.

---

# 8. Upload Options

FortiGate can use different mechanisms to send logs to remote logging systems.

| Upload Option       | Behavior                                     |
| ------------------- | -------------------------------------------- |
| **Real-time**       | Logs are forwarded as they are generated     |
| **Every minute**    | Logs are sent periodically                   |
| **Every 5 minutes** | Logs are sent periodically                   |
| **Store & Upload**  | Logs are stored locally first, then uploaded |

---

## Real-Time

```text
Log
 │
 └──────► FortiAnalyzer
```

### Advantage

* Low forwarding latency
* Suitable for near-real-time monitoring

### Consideration

The local device is not necessarily being used as the primary persistent log repository.

---

# 9. Store & Upload

This mode provides a local cache/store before forwarding logs.

```text
                ┌──────────────┐
Log ───────────►│ Local Disk   │
                └──────┬───────┘
                       │
                       ▼
                FortiAnalyzer
```

### CLI

```bash
config log fortianalyzer setting
    set upload-option store-and-upload
end
```

### Why use it?

Useful when:

* Temporary connectivity loss exists.
* Local buffering is desired.
* Logs should be retained locally before upload.

### Important

Once cached logs are successfully uploaded, local cached data can be removed according to the configured behavior.

---

# 10. Local Disk Retention

Disk logging is not unlimited.

Typical behavior:

```text
Disk
 │
 ├── Available
 │
 ├── Warning threshold
 │
 ├── Critical threshold
 │
 └── Full
```

The actual retention period depends on:

* Disk size
* Log volume
* Enabled log types
* Packet capture/archive usage
* Reports
* DLP archives
* IPS archives
* Log rotation settings

---

# 11. Disk Usage Diagnostics

Useful commands:

```bash
diagnose sys logdisk usage
```

Check quota:

```bash
diagnose sys logdisk quota
```

Check disk health:

```bash
diagnose sys logdisk smart
```

Check status:

```bash
diagnose sys logdisk status
```

### Operational workflow

```text
Check Usage
    │
    ▼
Check Quota
    │
    ▼
Check SMART / Health
    │
    ▼
Check Status
    │
    ▼
Tune Retention / Upload
```

---

# 12. Local Disk Configuration

Example:

```bash
config log disk setting
    set status enable
    set ips-archive enable
    set max-policy-packet-capture-size 100

    set log-quota 0
    set dlp-archive-quota 0
    set report-quota 0

    set maximum-log-age 7

    set full-first-warning-threshold 75
    set full-second-warning-threshold 90
    set full-final-warning-threshold 95

    set max-log-file-size 20

    set roll-schedule daily
    set roll-time 00:00

    set diskfull overwrite
end
```

> **Note:** Exact CLI syntax and available options can vary between FortiOS releases and hardware platforms. Verify the command tree with `?` / `show full-configuration` on the target version.

---

# 13. Log Quota

Example:

```bash
set log-quota 0
```

Depending on the relevant FortiOS implementation, `0` can represent an unlimited/default allocation rather than a literal unlimited amount of physical disk.

### Important

Do not interpret:

```text
0 = infinite physical storage
```

Instead:

```text
Configured quota
       │
       ▼
Available disk
       │
       ▼
Actual storage limitation
```

---

# 14. Log Age

Example:

```bash
set maximum-log-age 7
```

This controls the configured maximum age for retained local logs.

```text
Day 1 ── Day 2 ── ... ── Day 7 ──► Rotation / Removal
```

### Production consideration

Do not choose retention based only on the default.

Consider:

* Compliance requirements
* Incident response requirements
* Average daily log volume
* Disk capacity
* FortiAnalyzer retention
* Backup requirements

---

# 15. Disk Full Behavior

Example:

```bash
set diskfull overwrite
```

Possible design:

```text
Disk reaches capacity
        │
        ▼
Old log data
        │
        ▼
Overwrite / rotate
```

Another possible behavior is:

```text
Disk reaches capacity
        │
        ▼
Stop logging
```

### Critical distinction

**Overwrite** protects ongoing logging availability but sacrifices older logs.

**No-log behavior** preserves existing data but can result in loss of new logs.

> Choose based on operational and compliance requirements.

---

# 16. Disk Warning Thresholds

Example:

```bash
set full-first-warning-threshold 75
set full-second-warning-threshold 90
set full-final-warning-threshold 95
```

Concept:

```text
0% ─────────────── 75% ───────── 90% ───── 95% ─── 100%
                   │               │          │
                   ▼               ▼          ▼
                Warning         Warning    Critical
```

### Recommended monitoring

Alert before reaching 100%.

A logging system that reaches 100% without an appropriate rotation/overwrite policy can create a blind spot during an incident.

---

# 17. Log File Rotation

Example:

```bash
set roll-schedule daily
set roll-time 00:00
```

Concept:

```text
00:00
 │
 ▼
Close current log file
 │
 ▼
Create / rotate log file
 │
 ▼
Continue logging
```

Log rotation helps organize local storage and manage retention.

---

# 18. FTP Log Upload

FortiGate can also upload selected log files to an external FTP destination where supported.

Example structure:

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

### Design consideration

Do not treat FTP archival as equivalent to FortiAnalyzer.

```text
FTP
 │
 └── File archival

FortiAnalyzer
 │
 ├── Centralized logging
 ├── Search
 ├── Analytics
 ├── Reports
 └── Investigation
```

For security operations, centralized log analytics is generally more useful than simple file archival.

---

# 19. FortiAnalyzer Priority

If multiple external logging mechanisms are configured, understand which mechanism is responsible for:

* Centralized storage
* Real-time forwarding
* Local caching
* Reporting
* Archival

A common production design is:

```text
FortiGate
   │
   ├── Local cache
   │
   └──────► FortiAnalyzer
               │
               ├── Storage
               ├── Analytics
               └── Reports
```

---

# 20. Syslog

FortiGate can forward logs to external Syslog servers.

Typical use cases:

* SIEM
* SOC
* Centralized logging
* Splunk
* Security analytics
* Compliance

Example:

```bash
config log syslogd setting
    set status enable
    set server <SYSLOG-SERVER-IP>
end
```

### Multiple Syslog Servers

The supported number depends on the FortiOS version/platform.

Do not assume a fixed number across every FortiGate model and FortiOS release.

---

# 21. Syslog Formats

Common formats include:

```text
Default
CSV
CEF
```

### CEF

**CEF — Common Event Format**

Commonly useful when integrating with SIEM platforms.

Concept:

```text
FortiGate
    │
    ▼
CEF
    │
    ▼
SIEM
    │
    ├── Parsing
    ├── Correlation
    ├── Alerting
    └── Investigation
```

> Always verify the exact supported formats and fields on the target FortiOS release.

---

# 22. Syslog Example

```bash
config log syslogd setting
    set status enable
    set format default
    set server <SYSLOG-SERVER-IP>
end
```

For production:

```text
FortiGate
   │
   ├── Primary Syslog
   ├── Secondary Syslog
   └── FortiAnalyzer
```

Use redundancy where log availability is critical.

---

# 23. Log Filters

Log filters determine which events are forwarded to a particular logging destination.

This is extremely important in large environments.

```text
                 All Generated Logs
                         │
                         ▼
                    Log Filter
                         │
            ┌────────────┼────────────┐
            ▼            ▼            ▼
         FAZ          Syslog        Email
```

---

# 24. FortiAnalyzer Log Filter

Example:

```bash
config log fortianalyzer filter
    set severity <LEVEL>
    set forward-traffic enable
    set local-traffic enable
    set multicast-traffic enable
    set sniffer-traffic enable
end
```

### Concept

You can control:

* Severity
* Forward traffic
* Local traffic
* Multicast traffic
* Sniffer traffic
* Other available log categories

---

# 25. Syslog Log Filter

Example:

```bash
config log syslogd filter
    set severity <LEVEL>
    set forward-traffic enable
    set local-traffic enable
    set multicast-traffic enable
    set sniffer-traffic enable
end
```

### Why filters matter

Without filtering:

```text
Huge Log Volume
      │
      ▼
Syslog / SIEM
      │
      ├── Storage consumption
      ├── Parsing load
      ├── Network overhead
      └── Alert noise
```

With filtering:

```text
Useful Logs
    │
    ▼
SIEM
    │
    ▼
Better signal-to-noise ratio
```

---

# 26. Event Logging

Event logging controls which system/security events are recorded.

```text
Log Settings
└── Event Logging
      │
      ├── All
      └── Customize
```

### All

Records all available event log categories selected by the configuration.

### Customize

Allows specific event categories to be selected.

> **Best practice:** Avoid disabling security-relevant event logs merely to reduce log volume.

---

# 27. Local Traffic Logging

Local traffic means traffic destined to or generated by the FortiGate itself rather than ordinary transit/forward traffic.

```text
Client
  │
  ▼
FortiGate
  │
  └──► FortiGate itself
```

versus:

```text
LAN
 │
 ▼
FortiGate
 │
 ▼
Internet
```

which is typically **forward traffic**.

### Important

Local traffic logging can generate substantial log volume.

Evaluate whether it is required before enabling everything in high-volume environments.

---

# 28. Forward Traffic vs Local Traffic

| Log Type              | Meaning                                       |
| --------------------- | --------------------------------------------- |
| **Forward Traffic**   | Traffic traversing FortiGate                  |
| **Local Traffic**     | Traffic to/from FortiGate itself              |
| **Multicast Traffic** | Multicast-related traffic                     |
| **Sniffer Traffic**   | Traffic associated with sniffer functionality |

This distinction is important during troubleshooting.

---

# 29. Traffic Log UUIDs

FortiGate can include UUIDs in traffic logs.

## Policy UUID

```bash
config log setting
    set policy-uuid enable
end
```

Policy UUIDs help correlate traffic logs with firewall policies.

```text
Traffic Log
     │
     └── Policy UUID
             │
             ▼
        Firewall Policy
```

---

## Address UUID

Address UUIDs can help correlate logged source/destination objects with the corresponding address objects.

Useful for:

* Log filtering
* Automation
* API-based analysis
* Correlation
* Troubleshooting

> Exact CLI syntax can vary by FortiOS version. Verify with the target release's CLI reference.

---

# 30. FortiView Diagnostics

FortiView provides visibility into traffic, events and security information.

Useful diagnostic examples:

```bash
diagnose fortiview result event-log
```

```bash
diagnose fortiview result security-log
```

```bash
diagnose fortiview result security-log action-block
```

Historical time-range example:

```bash
diagnose fortiview time <START-DATETIME> <END-DATETIME>
```

Example:

```bash
diagnose fortiview time 2026-11-20 00:00:00 2026-11-21 00:00:00
```

> **Note:** Use dates appropriate to the current investigation. Do not copy example dates blindly into production troubleshooting.

---

# 31. FortiView Investigation Workflow

When investigating a security event:

```text
1. Identify timeframe
        │
        ▼
2. Open FortiView
        │
        ▼
3. Filter by source / destination
        │
        ▼
4. Check action
        │
        ▼
5. Check policy
        │
        ▼
6. Check security profile
        │
        ▼
7. Correlate with FAZ / SIEM
```

---

# 32. Checking Log Forwarding

Useful command:

```bash
execute log list 0
```

Use it to inspect available/logging information according to the FortiOS version.

For deeper troubleshooting, inspect:

```text
Log Settings
FortiAnalyzer status
Syslog status
Disk status
Network connectivity
Time synchronization
```

---

# 33. Performance Statistics

Performance statistics should not be confused with ordinary traffic/security logs.

Conceptually:

```text
FortiGate Statistics
       │
       ├── Runtime / performance data
       │
       └── Logging / event data
```

Performance statistics may be exported/consumed through supported external mechanisms such as Syslog or FortiAnalyzer depending on the FortiOS release and configuration.

---

# 34. Email Alerting

FortiGate can send selected events through email alerting.

Example:

```bash
config alertemail setting
    set username <SMTP-USER>
    set mailto1 <ADMIN-EMAIL>
    set filter-mode category

    set ips-logs enable
    set ha-logs enable
    set antivirus-logs enable
    set webfilter-logs enable
    set log-disk-usage-warning enable
    set ssh-logs enable
    set fds-update-log enable
    set admin-login-log enable

    set local-disk-usage 75
end
```

### Common alert categories

* IPS
* HA
* Antivirus
* Web Filter
* SSH
* FDS updates
* Administrator login
* Disk usage

---

# 35. Email Filter Modes

Common concepts include:

```text
Category
Threshold
```

### Category

Send alerts based on selected event categories.

### Threshold

Generate alerts based on defined thresholds where supported.

---

# 36. Disk Usage Alerting

A useful operational threshold:

```text
75% ── Warning
90% ── Serious
95% ── Critical
100% ── Capacity reached
```

Do not wait for 100%.

A healthy logging design should detect the problem before the disk becomes full.

---

# 37. Local Disk + FortiAnalyzer Design

### Recommended enterprise architecture

```text
                     Internet
                        │
                        ▼
                 ┌─────────────┐
                 │  FortiGate  │
                 └──────┬──────┘
                        │
             ┌──────────┴──────────┐
             │                     │
             ▼                     ▼
       Local Disk Cache       FortiAnalyzer
             │                     │
             │                     ├── Search
             │                     ├── Reports
             │                     ├── Analytics
             │                     └── Retention
             │
             └── Temporary Buffer
```

### Why?

If the connection to FortiAnalyzer is temporarily unavailable:

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

This provides better resilience than relying exclusively on real-time forwarding.

---

# 38. Enterprise Logging Design

For an enterprise environment:

```text
                    ┌──────────────────┐
                    │    FortiGate     │
                    └────────┬─────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
         FortiAnalyzer     Syslog        FortiCloud
              │              │
              ▼              ▼
          Reporting         SIEM
                             │
                             ▼
                         SOC / SIEM
```

### Design goals

* Centralized logging
* Reliable forwarding
* Adequate retention
* Security event visibility
* SIEM integration
* Alerting
* Historical investigation
* Disk monitoring

---

# 39. Security Fabric Logging

In a Security Fabric environment:

```text
                    Root FortiGate
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
          FGT-2         FGT-3         FGT-4
             │            │            │
             └────────────┼────────────┘
                          ▼
                    FortiAnalyzer
```

### Recommended thinking

Do not configure every FortiGate independently without considering:

* Fabric topology
* Root FortiGate
* Downstream devices
* Centralized logging
* FortiAnalyzer
* Local disk requirements
* Reporting workload

---

# 40. Logging Troubleshooting Checklist

When logs are missing:

### Step 1 — Is logging enabled?

```text
Log & Report
└── Log Settings
```

---

### Step 2 — Is the required log category enabled?

Check:

```text
Traffic
Event
Security
UTM
Local Traffic
```

---

### Step 3 — Is the destination reachable?

```text
FortiGate
   │
   ├── FortiAnalyzer
   ├── Syslog
   └── FortiCloud
```

---

### Step 4 — Check local disk

```bash
diagnose sys logdisk status
diagnose sys logdisk usage
diagnose sys logdisk quota
diagnose sys logdisk smart
```

---

### Step 5 — Check filters

```text
Severity
Traffic category
Event category
Security category
```

A filter can silently prevent expected logs from being forwarded.

---

### Step 6 — Check time

Incorrect time can break:

* Event correlation
* Certificates
* Authentication
* SIEM correlation
* Historical investigation

Verify:

```text
NTP
Timezone
System clock
```

---

### Step 7 — Check FortiAnalyzer authorization

```text
FortiGate
    │
    ├── Reachable?
    ├── Authorized?
    ├── Certificate valid?
    └── Logs arriving?
```

---

### Step 8 — Check Syslog

Verify:

```text
Server IP
Port
Protocol
Format
Filter
Network path
```

---

# 41. Log Storage Decision Matrix

| Requirement                         | Recommended                       |
| ----------------------------------- | --------------------------------- |
| Temporary troubleshooting           | Memory                            |
| Local short-term logging            | Local Disk                        |
| Enterprise centralized logs         | FortiAnalyzer                     |
| SIEM integration                    | Syslog / FortiSIEM                |
| Cloud logging                       | FortiCloud                        |
| Centralized reporting               | FortiAnalyzer                     |
| High availability of log collection | Multiple destinations             |
| Long-term retention                 | FortiAnalyzer / external platform |
| Offline/local cache before upload   | Store & Upload                    |

---

# 42. Real-Time vs Store & Upload

| Feature                                   | Real-Time | Store & Upload |
| ----------------------------------------- | --------: | -------------: |
| Immediate forwarding                      |         ✅ |              ❌ |
| Local caching                             |   Limited |              ✅ |
| Useful during temporary connectivity loss |   Limited |              ✅ |
| Lower forwarding latency                  |         ✅ |              ❌ |
| Local disk dependency                     |     Lower |         Higher |
| Good for centralized monitoring           |         ✅ |              ✅ |

---

# 43. Logging Best Practices

### ✅ Do

* Use FortiAnalyzer for centralized logging in enterprise deployments.
* Monitor local disk usage.
* Configure meaningful retention.
* Use log filters to reduce unnecessary noise.
* Synchronize time with reliable NTP.
* Monitor FortiAnalyzer authorization status.
* Use Syslog/CEF when integrating with SIEM where appropriate.
* Keep security-relevant event logging enabled.
* Use local storage as a cache/backup mechanism where appropriate.
* Test the logging path after deployment.

### ❌ Avoid

* Relying exclusively on memory logs.
* Sending every possible log type to the SIEM without planning.
* Ignoring disk utilization.
* Waiting until disk usage reaches 100%.
* Disabling security logs simply to reduce volume.
* Assuming all FortiOS versions have identical logging CLI options.
* Publishing real passwords, API keys or secrets in GitHub.
* Treating FTP archival as a replacement for centralized security analytics.

---

# 44. Quick CLI Reference

### Local disk

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

### FortiAnalyzer filter

```bash
config log fortianalyzer filter
    set severity <LEVEL>
    set forward-traffic enable
    set local-traffic enable
end
```

### Syslog filter

```bash
config log syslogd filter
    set severity <LEVEL>
    set forward-traffic enable
    set local-traffic enable
end
```

### Disk diagnostics

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

---

# 45. NSE Exam Memory Map

```text
LOGGING
│
├── Where?
│   ├── Memory
│   ├── Local Disk
│   ├── FortiAnalyzer
│   ├── FortiManager
│   ├── FortiCloud
│   ├── Syslog
│   └── FortiSIEM
│
├── How?
│   ├── Real-time
│   ├── Periodic
│   └── Store & Upload
│
├── What?
│   ├── Traffic
│   ├── Event
│   ├── Security
│   ├── Local Traffic
│   └── UTM
│
├── Filter?
│   ├── Severity
│   ├── Traffic type
│   └── Log category
│
├── Storage?
│   ├── Quota
│   ├── Log age
│   ├── Rotation
│   └── Disk-full behavior
│
└── Troubleshoot?
    ├── Disk status
    ├── Disk usage
    ├── Disk quota
    ├── Disk health
    ├── FAZ connectivity
    ├── Syslog connectivity
    └── FortiView
```

---

# 46. 60-Second Interview Answer

> **How would you design FortiGate logging for an enterprise?**

```text
I would avoid relying only on FortiGate memory or local storage.

I would use FortiAnalyzer as the centralized logging and reporting platform,
configure appropriate log filters to control log volume, and use local disk
as a cache or short-term storage where required.

For SIEM integration, I would forward the required events through Syslog
using an appropriate supported format such as CEF.

I would also monitor disk utilization, log retention, FortiAnalyzer
authorization/connectivity, NTP synchronization, and logging filters.

For critical environments, I would design redundant logging paths so that
a temporary failure of one logging destination does not create a complete
visibility gap.
```

---

# 47. SheynShield Golden Rules

> ### 🔥 Rule #1
>
> **A firewall without usable logs is difficult to investigate.**

> ### 🔥 Rule #2
>
> **Logging ≠ Reporting ≠ SIEM.**

```text
Logging
   ↓
Collect data

FortiAnalyzer
   ↓
Store + Search + Analyze + Report

SIEM
   ↓
Correlate + Detect + Alert + Investigate
```

> ### 🔥 Rule #3
>
> **Do not optimize logging only for storage consumption. Optimize for security visibility.**

> ### 🔥 Rule #4
>
> **Store & Upload provides a useful local-cache model when centralized logging is temporarily unreachable.**

> ### 🔥 Rule #5
>
> **Always check the complete logging chain:**

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
Report / Alert
```

---

## Related SheynShield Topics

* FortiGate Firewall Policy
* FortiGate Policy & Objects
* FortiGate Authentication
* FortiGate FSSO
* FortiGate RADIUS
* FortiGate LDAP
* FortiGate HA
* FortiGate Security Profiles
* FortiGate FortiView
* FortiAnalyzer
* FortiManager
* FortiSIEM
* Syslog & SIEM Integration
* FortiGate Troubleshooting

---

## Tags

`FortiGate` `FortiOS` `Fortinet` `Logging` `FortiAnalyzer` `FortiManager` `FortiCloud` `Syslog` `FortiSIEM` `FortiView` `SIEM` `NetworkSecurity` `CyberSecurity` `NSE4` `NSE7` `Firewall` `SecurityOperations`

---

**SheynShield — Engineering Secure Networks**

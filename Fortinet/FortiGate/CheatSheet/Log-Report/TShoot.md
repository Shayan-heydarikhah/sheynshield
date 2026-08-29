# FortiGate `miglogd` Troubleshooting  

> **FortiOS | miglogd Diagnostics, Log Forwarding, Buffering, Backup, Dumping & Failed Log Analysis**

---

## 📌 Table of Contents

* [1. What is miglogd?](#1-what-is-miglogd)
* [2. miglogd Troubleshooting Strategy](#2-miglogd-troubleshooting-strategy)
* [3. miglogd Debug Flags](#3-miglogd-debug-flags)
* [4. miglogd Diagnostic Commands](#4-miglogd-diagnostic-commands)
* [5. Log Forwarding Statistics](#5-log-forwarding-statistics)
* [6. Understanding Syslog Failures](#6-understanding-syslog-failures)
* [7. Understanding FortiAnalyzer Statistics](#7-understanding-fortianalyzer-statistics)
* [8. Log Backup](#8-log-backup)
* [9. Log Dumping](#9-log-dumping)
* [10. Local Log Management](#10-local-log-management)
* [11. SNMP & Failed Log Monitoring](#11-snmp--failed-log-monitoring)
* [12. Troubleshooting Workflow](#12-troubleshooting-workflow)
* [13. High-Value NSE Notes](#13-high-value-nse-notes)
* [14. Quick Reference](#14-quick-reference)

---

# 1. What is `miglogd`?

`miglogd` is the FortiGate logging daemon responsible for processing and forwarding logs to configured destinations such as:

```text
FortiGate
   │
   ▼
 miglogd
   │
   ├── Local Disk
   ├── Syslog
   ├── FortiAnalyzer
   ├── FortiGuard / Cloud services
   └── Other configured log destinations
```

When troubleshooting missing or delayed logs, `miglogd` is one of the most important processes to investigate.

---

# 2. miglogd Troubleshooting Strategy

A useful troubleshooting hierarchy is:

```text
                    Log Problem
                        │
                        ▼
                  Is log generated?
                        │
                        ▼
                    miglogd
                        │
          ┌─────────────┼──────────────┐
          ▼             ▼              ▼
       Queue         Forwarding      Storage
          │             │              │
          ▼             ▼              ▼
       Memory         Syslog          Disk
       / Disk           │
                        └── FAZ
```

When logs are not reaching the destination, investigate in this order:

1. **Is the FortiGate generating the log?**
2. **Is `miglogd` processing it?**
3. **Is the queue building up?**
4. **Is the remote destination reachable?**
5. **Are logs being transmitted?**
6. **Are logs being cached?**
7. **Are logs being dropped?**
8. **Is local storage full?**

---

# 3. miglogd Debug Flags

FortiGate provides debug flags for investigating different areas of `miglogd`.

---

## 🔎 `0x100` — Log Transmission / Filtering

```bash
diagnose debug application miglogd 0x100
```

Useful for investigating:

* Log filtering
* Live log transmission
* Forwarding behavior

Conceptually:

```text
Log
 ↓
miglogd
 ↓
Filter
 ↓
Transmission
 ↓
Remote Collector
```

For FortiAnalyzer communication, FortiGate uses **OFTP** for log forwarding.

---

## 📦 `0x2000` — Queue Management

```bash
diagnose debug application miglogd 0x2000
```

Useful when investigating:

* Log queues
* Queued messages
* Queue processing
* Delayed forwarding

Mental model:

```text
Incoming Logs
      │
      ▼
    Queue
      │
      ▼
   Forward
```

If the queue continuously grows, investigate the downstream destination and forwarding path.

---

## 🌐 `0x4000` — Remote Connection Handling

```bash
diagnose debug application miglogd 0x4000
```

Useful for investigating:

* Remote collector connections
* Connection handling
* Remote forwarding behavior

Use this when the problem appears to be related to the connection between FortiGate and a remote logging destination.

---

# 4. miglogd Diagnostic Commands

## Show Live Log Forwarding

```bash
diagnose test application miglogd 4
```

Useful for viewing:

* Live log forwarding
* Forwarding counters
* Current forwarding state

---

## Show miglogd Child Daemons

```bash
diagnose test application miglogd 6 1
```

and:

```bash
diagnose test application miglogd 6 2
```

These commands can be used to inspect individual `miglogd` child instances.

Conceptually:

```text
                 miglogd
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
       Child 1   Child 2   Child N
```

---

## Show miglog ID

```bash
diagnose test application miglogd 15
```

Useful when identifying the `miglogd` instance/environment involved in logging operations.

---

## Change Number of miglogd Children

### Increase

```bash
diagnose test application miglogd 13
```

### Decrease

```bash
diagnose test application miglogd 14
```

> ⚠️ Changing the number of logging child processes should be done carefully and according to the platform/version behavior. More processes do not automatically mean better logging performance.

---

## Check Buffer Memory

```bash
diagnose test application miglogd 14
```

> **Version note:** `miglogd` test commands are highly version/platform dependent. Always validate the exact command behavior against the FortiOS release in production.

---

# 5. Log Forwarding Statistics

A particularly useful diagnostic is:

```bash
diagnose test application miglogd 6
```

Example output:

```text
mem=404, disk=657, alert=0, alarm=0, sys=920, faz=555, webt=0, fds=0

interface-missed=460

Queues in all miglogds:
cur:0
total-so-far:526
```

---

## 🧠 Reading the Counters

| Counter            | Indicates                                 |
| ------------------ | ----------------------------------------- |
| `mem`              | Memory-related logging statistics         |
| `disk`             | Disk logging statistics                   |
| `alert`            | Alert logging statistics                  |
| `alarm`            | Alarm logging statistics                  |
| `sys`              | System logging statistics                 |
| `faz`              | FortiAnalyzer-related statistics          |
| `webt`             | WebTrends-related statistics              |
| `fds`              | FortiGuard/FDS-related statistics         |
| `interface-missed` | Logs/events missed at the interface stage |
| `cur`              | Current queue count                       |
| `total-so-far`     | Total queue-related count                 |

> Exact counter meanings can vary by FortiOS version and platform. Treat the output as a diagnostic snapshot rather than a universal fixed schema.

---

# 6. Understanding Syslog Failures

Example:

```text
syslog 0:
    sent=254
    failed=139
    relayed=0

syslog 1:
    sent=220
    failed=139
    relayed=0

syslog 2:
    sent=95
    failed=73
    relayed=0
```

The important relationship is:

```text
sent
  +
failed
  +
relayed
```

---

## 🚨 High `failed` Counter

A growing `failed` value indicates that log forwarding attempts to the corresponding Syslog destination are not succeeding.

Possible areas to investigate:

```text
FortiGate
   │
   ├── Network connectivity
   ├── Routing
   ├── Firewall policy
   ├── Syslog destination
   ├── Transport
   └── Remote Syslog performance
```

If the remote Syslog server is slow or unable to process messages fast enough, buffered messages can accumulate and eventually be lost.

---

# 7. Understanding FortiAnalyzer Statistics

Example:

```text
faz 0:
    sent=282
    failed=0
    cached=0
    dropped=0
    relayed=0
```

These counters are extremely useful.

---

## `sent`

```text
sent=282
```

Logs successfully forwarded.

---

## `failed`

```text
failed=0
```

Failed forwarding attempts.

A growing value should trigger investigation.

---

## `cached`

```text
cached=0
```

Logs currently being cached for later forwarding.

A growing cache can indicate:

```text
FAZ unavailable
       OR
Slow/unstable connection
       OR
Forwarding bottleneck
```

---

## `dropped`

```text
dropped=0
```

Logs discarded instead of successfully reaching the destination.

This is one of the most important counters when investigating **log loss**.

---

## `relayed`

```text
relayed=0
```

Indicates logs being relayed through another logging path/device, where applicable.

---

## 🔥 Practical Interpretation

```text
FAZ:

sent ↑
failed = 0
cached = 0
dropped = 0
```

Generally indicates healthy forwarding.

---

### Problem Pattern #1

```text
sent ↑
cached ↑
dropped = 0
```

Think:

```text
Temporary forwarding delay
       │
       ├── Latency
       ├── Connectivity instability
       ├── Bandwidth limitation
       └── Destination processing delay
```

---

### Problem Pattern #2

```text
sent ↑
failed ↑
```

Investigate:

```text
Connectivity
Routing
Remote service
Transport
Authentication/configuration
```

---

### Problem Pattern #3

```text
cached ↑
dropped ↑
```

High risk of log loss.

Investigate immediately:

```text
Buffer capacity
Bandwidth
Destination performance
Disk
Forwarding configuration
```

---

# 8. Log Backup

FortiGate can back up local disk log files.

## Full Log Backup

```bash
execute log backup
```

This backs up the available disk log files.

> **Important:** This functionality is available on FortiGates with an SSD disk.

---

## ⚠️ Before Full Backup

For large/full log backups, it may be desirable to temporarily stop the processes that actively handle logging/reporting.

Relevant processes:

```text
miglogd
reportd
```

One approach is to disable automatic restart before stopping the processes:

```bash
diagnose sys process daemon-auto-restart disable miglogd
diagnose sys process daemon-auto-restart disable reportd
```

> ⚠️ Process manipulation is disruptive. Perform it only during an approved maintenance/troubleshooting window.

---

# 9. Finding miglogd / reportd Processes

## Process Monitor

FortiGate GUI:

```text
Admin
  ↓
System
  ↓
Process Monitor
```

The process monitor can be used to identify or terminate processes.

---

## CLI Process View

```bash
diagnose sys top 10 99
```

This provides process information including CPU/memory-related statistics.

---

### Search for miglogd

```bash
diagnose sys top | grep miglog
```

### Search for reportd

```bash
diagnose sys top | grep report
```

---

## Kill a Process

Example:

```bash
diagnose sys kill 9 690
```

Where:

```text
9   = kill signal
690 = process ID
```

> 🚨 **Danger:** Never blindly copy a PID from an example. Always identify the current PID on the actual FortiGate first.

---

# 10. Log Backup Destinations

### Local

```bash
execute log backup local
```

### USB

```bash
execute log backup /usb/log.tar
```

Depending on FortiOS version/platform, additional destinations such as FTP/TFTP may be supported.

---

## Backup Workflow

```text
                    Log Backup
                        │
                        ▼
                 Check SSD support
                        │
                        ▼
               Schedule maintenance
                        │
                        ▼
          Consider stopping miglogd/reportd
                        │
                        ▼
                Execute log backup
                        │
              ┌─────────┴─────────┐
              ▼                   ▼
            Local               USB
              │                   │
              ▼                   ▼
        Local storage       /usb/log.tar
```

---

# 11. Log Dumping

Log dumping is a lighter-weight alternative to a full log backup.

> **Use log dumping when you need diagnostic log data without performing a complete disk-log backup.**

---

## Enable miglogd Dumping

For instance `1`:

```bash
diagnose test application miglogd 26 1
```

Expected concept:

```text
miglogd(1) log dumping is enabled
```

---

## Check Dumping Status

```bash
diagnose test application miglogd 26 0 255
```

This can display the dumping state of multiple `miglogd` instances.

Example:

```text
miglogd(0) log dumping is disabled
miglogd(1) log dumping is enabled
miglogd(2) log dumping is disabled
```

---

## Enable Individual Instances

### Instance 0

```bash
diagnose test application miglogd 26 0
```

### Instance 1

```bash
diagnose test application miglogd 26 1
```

### Instance 2

```bash
diagnose test application miglogd 26 2
```

---

# 12. Collect Dumped Log Information

Allow FortiGate to run and collect log messages.

Then:

```bash
diagnose test application miglogd 33
```

Example:

```text
2019-04-17 15:50:02 20828 log-1-0.dat
2019-04-17 15:48:31 4892  log-2-0.dat
```

This shows the generated dump files.

---

# 13. Copy Log Dump to USB

```bash
diagnose test application miglogd 34
```

Example:

```text
dumping file miglog1_index0.dat copied to usb disk ok
dumping file miglog2_index0.dat copied to usb disk ok
```

This allows the collected diagnostic dump files to be moved to removable storage.

---

# 14. Disable Log Dumping

Disable instance 0:

```bash
diagnose test application miglogd 26 0
```

Disable instance 1:

```bash
diagnose test application miglogd 26 1
```

Disable instance 2:

```bash
diagnose test application miglogd 26 2
```

Or inspect/control the configured instance range using:

```bash
diagnose test application miglogd 26 0 255
```

---

## 🔄 Log Dump Workflow

```text
Enable Dump
     │
     ▼
miglogd collects logs
     │
     ▼
Let system run
     │
     ▼
diagnose test application miglogd 33
     │
     ▼
Identify dump files
     │
     ▼
diagnose test application miglogd 34
     │
     ▼
Copy to USB
     │
     ▼
Analyze
     │
     ▼
Disable Dump
```

---

# 15. SNMP & Failed Log Monitoring

A slow Syslog server can create a serious logging problem.

Conceptually:

```text
FortiGate
   │
   ▼
Syslog
   │
   ▼
Remote Server
   │
   └── Slow response
          │
          ▼
    Retransmissions
          │
          ▼
   Kernel buffering
          │
          ▼
       Overflow
          │
          ▼
     Lost messages
```

Therefore, monitoring failed/queued/dropped log counters is important.

---

## SNMP Monitoring

For monitoring purposes, use the appropriate FortiGate SNMP OIDs for:

* Logging status
* Disk usage
* Log storage
* Failed forwarding
* Relevant system health counters

> **Important:** Exact OIDs depend on the FortiOS release and MIB version. Do not hard-code an OID from a different FortiOS version without validating it against the device's MIB.

---

# 16. Disk Monitoring

Local disk health is critical for FortiGate logging.

Monitor:

```text
Disk utilization
Log storage
Disk-full state
Log queue
Buffer utilization
```

A practical monitoring model:

```text
                 Logging Health
                       │
       ┌───────────────┼────────────────┐
       ▼               ▼                ▼
     Disk            Queue          Forwarding
       │               │                │
       ▼               ▼                ▼
   Capacity         Backlog       Failed/Dropped
```

---

# 17. Log Filtering During Troubleshooting

Before investigating `miglogd`, isolate the relevant log category.

Example:

```bash
execute log filter device disk
```

Select event logs:

```bash
execute log filter category event
```

Filter login events:

```bash
execute log filter field action login
```

Display:

```bash
execute log display
```

### Investigation Flow

```text
Disk
 ↓
Event
 ↓
Action = Login
 ↓
Display
```

This is useful for verifying that logs are actually being generated locally before troubleshooting forwarding.

---

# 18. Local Log Management

## Delete Logs from One Category

```bash
execute log delete
```

> Deletes local logs according to the selected log category/context.

---

## Delete All Local Reports / Logs

```bash
execute log delete-all
```

This removes local report/log database information and recreates the report database.

> 🚨 **Destructive operation.** Use only when you intentionally want to remove local log/report data.

---

## Flush Current Category Cache

```bash
execute log flush-cache
```

Writes the current category's disk-log cache to disk in compressed form.

---

## Flush All Log Caches

```bash
execute log flush-cache-all
```

Writes disk-log caches for all categories to disk in compressed form.

---

# 19. Raw Log Backup

For forensic analysis:

```bash
execute log raw-backup
```

Raw backup is useful when the goal is to provide log data to forensic/analysis tools.

Compared with a conventional full backup, raw backup can be:

```text
Lighter to process
Faster to read
Not compressed
FortiGate-specific/raw format
```

The resulting archive can be larger than a normal compressed backup.

### Comparison

| Method                   | Main Use             |               Compression | Typical Goal               |
| ------------------------ | -------------------- | ------------------------: | -------------------------- |
| `execute log backup`     | Full disk log backup | Depends on implementation | Complete backup            |
| `execute log raw-backup` | Forensic analysis    |                        No | Faster/raw analysis        |
| miglogd dump             | Troubleshooting      |         Diagnostic format | Investigate logging daemon |

---

# 20. Legacy WebTrends Logs

```bash
show log webtrends
```

This is associated with older/legacy log formats.

> **NSE / Operational Note:** Treat WebTrends-related logging as legacy functionality and verify relevance against the FortiOS version being investigated.

---

# 21. Advanced Troubleshooting Workflow

## Scenario: Logs Are Not Reaching Syslog

```text
                 Logs Missing
                     │
                     ▼
              Check local logs
                     │
                     ▼
               Is log generated?
                 │          │
                NO         YES
                 │          │
                 ▼          ▼
          Check logging   Check miglogd
          configuration       │
                              ▼
                       diagnose test
                       application
                       miglogd 6
                              │
                              ▼
                       Check syslog
                       sent/failed
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
                 failed ↑            sent ↑
                    │                   │
                    ▼                   ▼
             Check destination     Check filtering
             / connectivity        / forwarding
```

---

# 22. Scenario: FAZ Logs Are Delayed

Start with:

```bash
diagnose test application miglogd 6
```

Look for:

```text
faz:
    sent
    failed
    cached
    dropped
```

Then:

```text
cached ↑
   │
   ├── Check bandwidth
   ├── Check latency/jitter
   ├── Check FAZ availability
   ├── Check forwarding configuration
   └── Check local buffer capacity
```

If:

```text
dropped ↑
```

treat it as a potential **log-loss condition** and investigate immediately.

---

# 23. Scenario: miglogd Queue Is Growing

Use:

```bash
diagnose debug application miglogd 0x2000
```

and:

```bash
diagnose test application miglogd 6
```

Look for:

```text
Queue
  │
  ├── Current count
  ├── Total count
  └── Destination-specific backlog
```

Then investigate the slowest destination.

---

# 24. Scenario: Remote Connection Problem

Use:

```bash
diagnose debug application miglogd 0x4000
```

Then correlate with:

```text
Network connectivity
Routing
Destination availability
Transport
Latency
Packet loss
```

---

# 25. Scenario: Need Detailed Live Forwarding Debug

Use:

```bash
diagnose debug application miglogd 0x100
```

This is useful when you need to observe:

```text
Filter
  ↓
Log selection
  ↓
Forwarding
  ↓
Transmission
```

Remember to control the debug session carefully in production because verbose debugging can generate substantial output.

---

# 26. miglogd Diagnostic Matrix

| Command                                     | Primary Purpose                                     |
| ------------------------------------------- | --------------------------------------------------- |
| `diagnose debug application miglogd 0x100`  | Log filtering / transmission                        |
| `diagnose debug application miglogd 0x2000` | Queue management                                    |
| `diagnose debug application miglogd 0x4000` | Remote connection handling                          |
| `diagnose test application miglogd 4`       | Live forwarding information                         |
| `diagnose test application miglogd 6`       | Logging statistics / counters                       |
| `diagnose test application miglogd 6 1`     | Child daemon instance                               |
| `diagnose test application miglogd 6 2`     | Child daemon instance                               |
| `diagnose test application miglogd 15`      | miglog ID                                           |
| `diagnose test application miglogd 13`      | Increase child count                                |
| `diagnose test application miglogd 14`      | Decrease child count / version-specific diagnostics |
| `diagnose test application miglogd 20`      | Logging/collector diagnostic information            |
| `diagnose test application miglogd 26`      | Log dumping control                                 |
| `diagnose test application miglogd 33`      | Show dumped log files                               |
| `diagnose test application miglogd 34`      | Copy dump files to USB                              |

> ⚠️ **Version warning:** `diagnose test application miglogd <id>` subcommands are implementation-specific and can change between FortiOS releases. Validate the exact command on the target release before using it operationally.

---

# 27. NSE High-Value Notes 🧠

### `miglogd`

```text
miglogd
   ↓
Log processing
   ↓
Local storage / Remote forwarding
```

---

### Debug Flag `0x100`

```bash
diagnose debug application miglogd 0x100
```

Think:

> **Filtering + transmission**

---

### Debug Flag `0x2000`

```bash
diagnose debug application miglogd 0x2000
```

Think:

> **Queue management**

---

### Debug Flag `0x4000`

```bash
diagnose debug application miglogd 0x4000
```

Think:

> **Remote connection handling**

---

### FAZ Counters

```text
sent
failed
cached
dropped
relayed
```

The two counters that deserve immediate attention during forwarding problems:

```text
cached
dropped
```

---

### Syslog

```text
failed ↑
```

→ investigate the destination and forwarding path.

A slow Syslog server can contribute to buffering and eventual message loss.

---

### Local Log Verification

```bash
execute log filter device disk
execute log filter category event
execute log filter field action login
execute log display
```

First prove:

> **The log exists locally.**

Then troubleshoot:

> **Why was it not forwarded?**

---

# 28. Production Troubleshooting Checklist

```text
[ ] Confirm log generation
[ ] Check local disk logs
[ ] Verify log filtering
[ ] Check miglogd process
[ ] Check queue state
[ ] Check memory buffer
[ ] Check disk buffer
[ ] Check Syslog sent/failed
[ ] Check FAZ sent/failed/cached/dropped
[ ] Check remote connectivity
[ ] Check bandwidth / latency / packet loss
[ ] Check disk capacity
[ ] Check FortiAnalyzer availability
[ ] Check FortiGate logging configuration
[ ] Collect miglogd diagnostics
[ ] Perform log dump if required
[ ] Perform full backup if required
[ ] Preserve evidence before destructive operations
```

---

# 29. Golden Troubleshooting Rule 🔥

When someone says:

> **"FortiGate is not sending logs."**

Do **not** immediately assume the problem is FortiAnalyzer or Syslog.

Use this chain:

```text
                 LOG PROBLEM
                     │
                     ▼
             Is the log generated?
                     │
                     ▼
              Does local disk
              contain the log?
                     │
                     ▼
                  miglogd
                     │
                     ▼
                  Queue?
                     │
                     ▼
               Forwarding?
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
       Syslog                  FAZ
          │                     │
     sent / failed        sent / failed
          │                cached / dropped
          └──────────┬──────────┘
                     ▼
             Network / Server
                     │
                     ▼
                Root Cause
```

> **The most important troubleshooting principle:**
> **First prove the log exists, then prove `miglogd` processed it, then prove it entered the forwarding queue, and finally investigate the remote collector.**

---

# 30. Quick Command  

```bash
# miglogd live transmission / filtering
diagnose debug application miglogd 0x100

# miglogd queue management
diagnose debug application miglogd 0x2000

# miglogd remote connection handling
diagnose debug application miglogd 0x4000

# Live forwarding statistics
diagnose test application miglogd 4

# Detailed miglogd statistics
diagnose test application miglogd 6

# Child daemon diagnostics
diagnose test application miglogd 6 1
diagnose test application miglogd 6 2

# miglog ID
diagnose test application miglogd 15

# miglogd diagnostic / collector information
diagnose test application miglogd 20

# Enable log dumping
diagnose test application miglogd 26 1

# Show dumping status
diagnose test application miglogd 26 0 255

# Show generated dump files
diagnose test application miglogd 33

# Copy dump files to USB
diagnose test application miglogd 34

# Local disk log investigation
execute log filter device disk
execute log filter category event
execute log filter field action login
execute log display

# Full log backup
execute log backup

# Raw forensic backup
execute log raw-backup

# Delete selected logs
execute log delete

# Delete all local report/log data
execute log delete-all

# Flush current log cache
execute log flush-cache

# Flush all log caches
execute log flush-cache-all

# Legacy WebTrends information
show log webtrends
```

---

# 🎯 Final Mental Model

```text
                       FORTIGATE
                           │
                           ▼
                         LOG
                           │
                           ▼
                       miglogd
                           │
              ┌────────────┼────────────┐
              │            │            │
              ▼            ▼            ▼
           Memory        Queue         Disk
              │            │            │
              └────────────┼────────────┘
                           │
                    ┌──────┴──────┐
                    ▼             ▼
                 Syslog          FAZ
                    │             │
              sent/failed    sent/failed
                              cached
                              dropped
```

### 🔥 One-Line Memory Hook

```text
0x100  → Transmission
0x2000 → Queue
0x4000 → Remote Connection
6      → Statistics
4      → Live Forwarding
26     → Log Dump
33     → Show Dump Files
34     → Copy Dump Files
```

> **FortiGate Logging Troubleshooting = Generate → Process → Queue → Store → Forward → Confirm → Preserve.**

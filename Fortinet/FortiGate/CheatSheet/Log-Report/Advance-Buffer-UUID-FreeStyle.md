# FortiGate Advanced & Specialized Logging  

> **FortiOS | Advanced Logging, Audit Tracking, Log Buffering, UUID & Free-Style Filters**

---

## 📌 Table of Contents

* [1. CLI Audit Logging](#1-cli-audit-logging)
* [2. System Event & Local Traffic Investigation](#2-system-event--local-traffic-investigation)
* [3. FortiAnalyzer Disk Buffer](#3-fortianalyzer-disk-buffer)
* [4. Memory vs Disk Buffer Architecture](#4-memory-vs-disk-buffer-architecture)
* [5. Disk Buffer Overflow & FIFO Behavior](#5-disk-buffer-overflow--fifo-behavior)
* [6. Monitoring miglogd](#6-monitoring-miglogd)
* [7. UUID Logging](#7-uuid-logging)
* [8. Free-Style Logging Filters](#8-free-style-logging-filters)
* [9. Operational Recommendations](#9-operational-recommendations)
* [10. Quick Reference](#10-quick-reference)

---

# 1. CLI Audit Logging

FortiGate can record **administrative CLI activity** to provide an audit trail of configuration changes and administrative actions.

### Enable CLI Audit Logging

```bash
config system global
    set cli-audit-log enable
end
```

### Scope

> `cli-audit-log` is a **global setting** and affects **all VDOMs**.

This allows administrators to track:

```text
Administrator
     ↓
CLI Command
     ↓
FortiGate
     ↓
Audit Log
```

### What does CLI Audit provide?

CLI audit logging is useful for:

* **Auditing**
* **Accounting**
* Tracking administrative actions
* Investigating configuration changes
* Determining **who did what**
* Determining **when an action occurred**

> **Key idea:** CLI audit logs answer the question:
>
> **"Who changed what on the FortiGate?"**

---

# 2. System Event & Local Traffic Investigation

From the GUI:

```text
Log & Report
    ├── System Events
    └── Local Traffic
```

These logs can help identify:

* Administrative actions
* System events
* Local traffic activity
* Configuration-related events
* Operational problems

Selecting a log and opening the **Details** panel provides additional fields and information.

---

## CLI Log Investigation

### Select logs stored on disk

```bash
execute log filter device disk
```

### Filter by category

```bash
execute log filter category event
```

### Filter by subtype

```bash
execute log filter field subtype system
```

### Filter by Log ID

```bash
execute log filter field logid <LOG-ID>
```

### Display filtered logs

```bash
execute log display
```

### Investigation Flow

```text
Disk Logs
   ↓
Category = Event
   ↓
Subtype = System
   ↓
Log ID
   ↓
Display
```

---

# 3. FortiAnalyzer Disk Buffer

When FortiGate forwards logs to **FortiAnalyzer**, temporary log buffering becomes important.

Consider this scenario:

```text
FortiGate ───────────────> FortiAnalyzer
             connection lost
```

If the connection to FortiAnalyzer is temporarily unavailable, FortiGate can cache logs locally and forward them later when connectivity is restored.

### Buffering Architecture

```text
                    FortiGate
                       │
                 ┌─────┴─────┐
                 │  miglogd  │
                 └─────┬─────┘
                       │
              ┌────────┴────────┐
              │                 │
           Memory              Disk
              │                 │
              └────────┬────────┘
                       ↓
                 FortiAnalyzer
```

---

## Configure FAZ Disk Buffer

```bash
config system global
    set faz-disk-buffer-size 200
end
```

### Default

```text
100 MB
```

### Example

```bash
config system global
    set faz-disk-buffer-size 200
end
```

This increases the configured FortiAnalyzer disk buffer to:

```text
200 MB
```

> ⚠️ Changing the FAZ disk buffer size can reset `miglogd` and therefore temporarily change logging behavior.

---

# 4. Memory vs Disk Buffer Architecture

FortiGate uses different `miglogd` processes for buffering.

### High-Level Flow

```text
Incoming Logs
     │
     ▼
Memory Buffer
     │
     ▼
Disk Buffer
     │
     ▼
FortiAnalyzer
```

### Processes

| Process            | Responsibility                       |
| ------------------ | ------------------------------------ |
| `miglogd`          | Main logging daemon / disk buffering |
| `miglogd-children` | Memory buffering                     |

### Important Diagnostic Detail

Disk-buffer statistics:

```text
main-miglogd
```

Memory-buffer statistics:

```text
miglogd-children
```

### Conceptual Architecture

```text
                    Logs
                     │
                     ▼
              miglogd-children
                     │
                Memory Buffer
                     │
                     ▼
                 miglogd
                     │
                 Disk Buffer
                     │
                     ▼
               FortiAnalyzer
```

> **Remember:**
> **Memory → miglogd-children**
> **Disk → main miglogd**

---

# 5. Disk Buffer Overflow & FIFO Behavior

The buffer has a finite capacity.

If the total available buffer becomes full:

```text
New Log
   ↓
Buffer Full?
   │
  YES
   ↓
Old Logs Removed
   ↓
New Logs Stored
```

This results in **FIFO-style overwrite behavior**.

### Example

Assume:

```text
FAZ disk buffer = 200 MB
```

If FortiAnalyzer is unreachable long enough for the buffer to reach capacity:

```text
200 MB
████████████████████████████

New Log
   ↓
Oldest Log Removed
   ↓
New Log Stored
```

Therefore, older cached logs can be lost.

---

## Disable Logging When Disk Is Full

The disk-full behavior can be controlled under:

```bash
config log disk setting
    set diskfull nolog
end
```

### Meaning

```text
diskfull nolog
```

means FortiGate does **not continue writing logs to a full disk by overwriting older logs**.

Conceptually:

```text
Disk Full
   ↓
No more local disk logging
```

> ⚠️ This is an operational trade-off: preserving existing logs comes at the cost of dropping new logs once the storage condition is reached.

---

# 6. Monitoring `miglogd`

For troubleshooting logging behavior, inspect the `miglogd` process.

```bash
diagnose test application miglogd 4
```

This can help investigate:

* Logging process behavior
* Memory buffering
* Disk buffering
* Queued logs
* FortiAnalyzer forwarding
* Buffer utilization

### Troubleshooting Mental Model

```text
FortiGate generates log
        │
        ▼
     miglogd
        │
        ├── Memory Queue
        │
        └── Disk Queue
               │
               ▼
          FortiAnalyzer
```

If logs are not reaching FortiAnalyzer:

```text
1. Check connectivity
2. Check logging configuration
3. Check miglogd
4. Check memory buffer
5. Check disk buffer
6. Check FAZ status
```

---

# 7. UUID Logging

UUID logging improves correlation and tracking of objects referenced by logs.

Enable UUID logging globally:

```bash
config system global
    set log-uuid-address enable
    set log-uuid-policy enable
end
```

### Configuration

```bash
config system global
    set log-uuid-address enable
    set log-uuid-policy enable
end
```

### Why UUIDs Matter

FortiGate configurations contain objects such as:

* Firewall policies
* Addresses
* Address groups
* Other referenced objects

Object names can change over time.

UUIDs provide a more stable identifier for:

```text
Object
   ↓
Policy
   ↓
Log
   ↓
SIEM / FortiAnalyzer
```

### Useful for

* Log correlation
* API-based tracking
* Automation
* Auditing
* Configuration analysis
* SIEM integration

> **Practical tip:** UUID-based correlation becomes especially valuable when automation/API consumers need to track objects independently from their human-readable names.

---

# 8. Free-Style Logging Filters

Free-style filters provide advanced filtering expressions for logs.

> ⚠️ **Use carefully.** Free-style filters are powerful, but they are generally less readable and maintainable than structured filtering.

---

## Configure Free-Style Syslog Filter

```bash
config log syslogd filter
    config free-style
        edit 1
            set category traffic
            set filter logid <ID1> <ID2>
            set filter-type include
        next
    end
end
```

### Example

```bash
config log syslogd filter
    config free-style
        edit 1
            set category traffic
            set filter logid 0102043039 0102043040
            set filter-type include
        next
    end
end
```

---

## Supported Categories

Depending on FortiOS version and logging configuration, free-style filtering can target categories such as:

```text
event
virus
webfilter
attack
spam
anomaly
voip
dlp
app-ctrl
waf
dns
ssh
ssl
file-filter
icap
ztna
traffic
```

> **Version note:** Available categories and syntax can vary between FortiOS releases. Always validate against the target FortiOS version.

---

# 9. Include vs Exclude

Free-style filters support different filtering behaviors.

### Include

```bash
set filter-type include
```

Only matching logs are selected.

Conceptually:

```text
All Logs
   │
   ▼
[ Match Condition ]
   │
   ├── Match → Keep
   └── No Match → Drop
```

### Exclude

```text
Exclude matching logs
```

Conceptually:

```text
All Logs
   │
   ▼
[ Match Condition ]
   │
   ├── Match → Drop
   └── No Match → Keep
```

---

# 10. CLI Free-Style Log Filtering

Free-style filters can also be used directly during CLI log investigation.

---

## Filter by Log ID

```bash
execute log filter free-style "logid <ID1> <ID2>"
```

Example:

```bash
execute log filter free-style "logid 0102043039 0102043040"
```

---

## Filter by Source IP

```bash
execute log filter free-style "srcip <IP1> <IP2>"
```

Example:

```bash
execute log filter free-style "srcip 192.168.2.5 192.168.2.205"
```

---

## OR Logic

Use parentheses with `or`:

```bash
execute log filter free-style "(logid <ID>) or (srcip <IP1> <IP2>)"
```

Example:

```bash
execute log filter free-style "(logid 0102043039) or (srcip 192.168.2.5 192.168.2.205)"
```

Meaning:

```text
Log ID matches
        OR
Source IP matches
```

---

## AND Logic

```bash
execute log filter free-style "(srcip <IP>) and (dstip <IP>)"
```

Example:

```bash
execute log filter free-style "(srcip 192.168.2.5) and (dstip 10.10.10.10)"
```

Meaning:

```text
Source IP = 192.168.2.5
        AND
Destination IP = 10.10.10.10
```

---

# 11. Dump the Active Filter

After applying a free-style filter:

```bash
execute log filter dump
```

This is useful to verify what filtering conditions are currently active.

### Example

```bash
execute log filter free-style "(logid 0102043039) or (srcip 192.168.2.5 192.168.2.205)"

execute log filter dump
```

Then inspect the resulting filter configuration.

---

# 12. Advanced Filtering Workflow

A practical troubleshooting workflow:

```text
                 Start
                   │
                   ▼
            Identify Log Source
                   │
                   ▼
            Select Log Device
                   │
                   ▼
          Define Log Category
                   │
                   ▼
          Apply Structured Filter
                   │
                   ▼
         Need Complex Filtering?
              │          │
             NO         YES
              │          │
              │          ▼
              │    Free-Style Filter
              │          │
              └────┬─────┘
                   ▼
             Dump Filter
                   │
                   ▼
             Display Logs
                   │
                   ▼
              Analyze
```

---

# 13. Advanced Logging: What to Use When?

| Requirement                      | Recommended Mechanism                 |
| -------------------------------- | ------------------------------------- |
| Track administrator CLI commands | `cli-audit-log`                       |
| Investigate system events        | System Event logs                     |
| Investigate local traffic        | Local Traffic logs                    |
| Temporarily cache FAZ logs       | FAZ disk buffer                       |
| Investigate logging daemon       | `diagnose test application miglogd 4` |
| Correlate policy/address objects | UUID logging                          |
| Simple log filtering             | Structured `execute log filter`       |
| Complex conditions               | Free-style filtering                  |
| Verify active filter             | `execute log filter dump`             |

---

# 14. High-Value NSE Exam Notes 🧠

### CLI Audit

```bash
config system global
    set cli-audit-log enable
end
```

**Remember:**

> Global configuration → affects all VDOMs.

---

### FAZ Disk Buffer

```bash
config system global
    set faz-disk-buffer-size 200
end
```

**Default:**

```text
100 MB
```

**Important:**

> Changing the buffer size can reset `miglogd`.

---

### Buffer Architecture

```text
Memory Buffer
     ↓
miglogd-children

Disk Buffer
     ↓
main miglogd
```

---

### Buffer Full

```text
Buffer Full
     ↓
FIFO behavior
     ↓
Old logs may be overwritten
```

If disk-full logging is configured as:

```bash
config log disk setting
    set diskfull nolog
end
```

new disk logs are not written once the disk is full.

---

### UUID

```bash
config system global
    set log-uuid-address enable
    set log-uuid-policy enable
end
```

Useful for:

```text
Correlation
Automation
API
SIEM
Auditing
```

---

### Free-Style

```bash
execute log filter free-style "..."
```

Examples:

```bash
execute log filter free-style "logid 0102043039 0102043040"
```

```bash
execute log filter free-style "srcip 192.168.2.5 192.168.2.205"
```

```bash
execute log filter free-style "(logid 0102043039) or (srcip 192.168.2.5 192.168.2.205)"
```

```bash
execute log filter free-style "(srcip 192.168.2.5) and (dstip 10.10.10.10)"
```

Then:

```bash
execute log filter dump
```

---

# 15. Quick Command Reference

| Task                      | Command                                     |
| ------------------------- | ------------------------------------------- |
| Enable CLI audit          | `set cli-audit-log enable`                  |
| Configure FAZ disk buffer | `set faz-disk-buffer-size <MB>`             |
| Enable address UUID       | `set log-uuid-address enable`               |
| Enable policy UUID        | `set log-uuid-policy enable`                |
| Select disk logs          | `execute log filter device disk`            |
| Filter event category     | `execute log filter category event`         |
| Filter system subtype     | `execute log filter field subtype system`   |
| Filter by Log ID          | `execute log filter field logid <ID>`       |
| Display logs              | `execute log display`                       |
| Dump active filter        | `execute log filter dump`                   |
| Free-style Log ID         | `execute log filter free-style "logid ..."` |
| Free-style source IP      | `execute log filter free-style "srcip ..."` |
| Free-style OR             | `"(condition) or (condition)"`              |
| Free-style AND            | `"(condition) and (condition)"`             |
| Inspect miglogd           | `diagnose test application miglogd 4`       |

---

# 🎯 Key Takeaways

```text
CLI Audit
    ↓
Who did what?

FAZ Disk Buffer
    ↓
What happens when FAZ is unavailable?

miglogd
    ↓
Who handles logging/buffering?

UUID Logging
    ↓
How can objects be correlated reliably?

Free-Style Filter
    ↓
How can complex log conditions be investigated?
```

> **Operational mindset:**
> Advanced logging is not just about collecting more logs. The real objective is to create a reliable chain from **event → administrator/object → policy → log → storage → analysis → correlation**.

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

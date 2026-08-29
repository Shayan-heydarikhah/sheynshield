# FortiGate Logging, `miglogd` & `fgtlogd` —  

> **FortiOS Focus:** 7.2.x
> **Scope:** Local Logging, FortiAnalyzer, FortiAnalyzer Cloud, Syslog, VDOM Overrides, `miglogd`, `fgtlogd`, Log Filtering & Troubleshooting

---

## 1. Logging Architecture — Big Picture

FortiGate can generate, buffer, store, forward, and analyze logs through several destinations:

```text
                         ┌──────────────────────┐
                         │      FortiGate       │
                         │                      │
Traffic / Events ───────►│ Logging Subsystem    │
                         └──────────┬───────────┘
                                    │
             ┌──────────────────────┼──────────────────────┐
             │                      │                      │
             ▼                      ▼                      ▼
        System Memory          Local Disk             Remote
                                                       Logging
                                                         │
                          ┌──────────────────────────────┼──────────────┐
                          │                              │              │
                          ▼                              ▼              ▼
                    FortiAnalyzer                 FortiAnalyzer     Syslog
                    / FortiManager                   Cloud
```

### Main destinations

| Destination             | Purpose                                        |
| ----------------------- | ---------------------------------------------- |
| **System Memory**       | Temporary log buffering                        |
| **Local Disk**          | Persistent local log storage                   |
| **FortiAnalyzer**       | Centralized logging, analytics & reporting     |
| **FortiManager**        | Remote log management in supported deployments |
| **FortiAnalyzer Cloud** | Cloud-based centralized logging                |
| **FortiGate Cloud**     | Cloud logging services                         |
| **Syslog**              | External SIEM/log collection                   |
| **FortiSIEM**           | SIEM integration                               |

---

# 2. `miglogd` vs `fgtlogd`

Understanding the logging daemons is important when troubleshooting FortiGate logging.

| Daemon    | Main Role                                                              |
| --------- | ---------------------------------------------------------------------- |
| `miglogd` | Handles log processing/storage and local log-related operations        |
| `fgtlogd` | Handles log forwarding/reporting and remote logging-related operations |

A simplified model:

```text
Traffic / Event
      │
      ▼
 FortiGate Logging
      │
      ├──────────────► miglogd
      │                  │
      │                  ├── Log buffers
      │                  ├── Local disk
      │                  ├── Log caches
      │                  └── Local log processing
      │
      └──────────────► fgtlogd
                         │
                         ├── FortiAnalyzer
                         ├── FortiAnalyzer Cloud
                         ├── FortiCloud
                         └── Remote logging mechanisms
```

> **Troubleshooting rule:** If the problem is related to **local log storage, log cache, disk usage or local log processing**, investigate `miglogd`.
> If the problem is related to **remote log synchronization/forwarding**, investigate `fgtlogd`.

---

# 3. Log Buffering

A portion of FortiGate memory is assigned for log buffering.

> Approximately **5% of available memory** can be used for log buffering.

The exact behavior depends on the FortiOS version, hardware platform, enabled logging destinations and configuration.

### Important

Performance statistics are not stored on disk as normal traffic logs.

They can be obtained through:

* Syslog
* FortiAnalyzer
* FortiView / monitoring mechanisms

---

# 4. FortiView Log Diagnostics

Useful commands:

```bash
diagnose fortiview result event-log
```

```bash
diagnose fortiview result security-log
```

```bash
diagnose fortiview result security-log action-block
```

### Time-based FortiView query

```bash
diagnose fortiview time 2026-11-20 00:00:00 2026-11-21 00:00:00
```

Useful when investigating:

* Security events
* Blocked traffic
* Threat activity
* Historical FortiView information

---

# 5. Local Log Storage

FortiGate can store logs locally using:

```text
System Memory
      │
      └── Local Disk
```

## Local Disk

When disk logging is enabled:

```text
Traffic/Event
     │
     ▼
Local Disk
     │
     └── Optional upload
             │
             ▼
        FortiAnalyzer
```

### Important Design Notes

* The disk used for logging should be properly initialized/formatted.
* Do not assume that an installed disk automatically means it is configured for logging.
* In Security Fabric deployments, logging behavior differs between the **root FortiGate** and downstream devices.
* Local disk logging availability in the GUI can change when Security Fabric is enabled.
* The root FortiGate is the preferred location for centralized local reporting/storage in a Fabric design.

### Recommended enterprise design

```text
                    FortiAnalyzer
                         ▲
                         │
                         │
              ┌──────────┴──────────┐
              │     Root FortiGate  │
              │                     │
              │  SSD #1 → Logs      │
              │  SSD #2 → Storage   │
              └──────────┬──────────┘
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
            FGT-2      FGT-3      FGT-4
          downstream downstream downstream
```

> **Best practice:** In a Security Fabric architecture, centralize reporting and analytics through the root FortiGate + FortiAnalyzer rather than designing every downstream FortiGate as an independent reporting platform.

---

# 6. Local Reports

Local reports define whether FortiGate generates reports locally.

### Enable

```text
Log & Report
   └── Local Reports
```

Reports can be reviewed directly on the FortiGate.

### Disable

No local report generation.

### Design Recommendation

For resource-constrained FortiGate devices:

```text
FortiGate
   │
   └── Forward logs
           │
           ▼
     FortiAnalyzer
           │
           └── Reporting
```

> **Recommendation:** Disable local reporting when centralized FortiAnalyzer reporting is available, unless there is a specific operational reason to keep local reports.

---

# 7. Historical FortiView

Historical FortiView depends on local log storage.

### Requirements

```text
Disk Logging
     │
     └── Enabled
           │
           ▼
   Historical FortiView
```

By default, historical log information older than approximately **7 days** is removed.

---

# 8. Remote Logging — FortiAnalyzer / FortiManager

Navigate to:

```text
Log & Report
└── Log Settings
    └── Remote Logging and Archiving
```

Enable:

```text
Send Logs to FortiAnalyzer/FortiManager
```

### Basic CLI

```bash
config log fortianalyzer setting
    set status enable
    set server 192.168.254.200
end
```

Test connectivity:

```bash
execute log fortianalyzer test-connectivity
```

---

# 9. FortiAnalyzer Upload Modes

The upload option determines how logs are sent to the remote FortiAnalyzer/FortiManager.

| Mode               | Behavior                                       |
| ------------------ | ---------------------------------------------- |
| `realtime`         | Logs are forwarded continuously                |
| `1-minute`         | Logs are sent approximately every minute       |
| `5-minute`         | Logs are sent approximately every five minutes |
| `store-and-upload` | Logs are stored locally first, then uploaded   |

### Example

```bash
config log fortianalyzer setting
    set upload-option store-and-upload
end
```

---

# 10. Store-and-Upload Architecture

`store-and-upload` is useful when you want local caching before forwarding.

```text
Traffic
   │
   ▼
FortiGate
   │
   ▼
Local Disk
   │
   │ cache
   ▼
FortiAnalyzer
   │
   ▼
Uploaded logs
```

### Important characteristics

* Logs are temporarily stored locally.
* Logs are uploaded to FortiAnalyzer according to the configured mechanism.
* Uploaded temporary/cached files can be deleted after successful upload.
* Disk capacity must be monitored.
* Log sequence numbers are important when synchronizing logs.

> **Version compatibility:** For a FortiOS 7.2.x deployment, maintain compatible FortiAnalyzer versions, particularly when using advanced logging/synchronization mechanisms.

---

# 11. Reliable Logging

`reliable` controls reliable log transmission behavior.

Example:

```bash
config log fortianalyzer setting
    set reliable enable
end
```

Conceptually:

```text
FortiGate
   │
   │ reliable transmission
   ▼
FortiAnalyzer
```

Useful when log delivery reliability is more important than simply forwarding logs as quickly as possible.

---

# 12. FortiAnalyzer Advanced Configuration

Example reference configuration:

```bash
config log fortianalyzer setting
    set status enable
    set server 192.168.254.200

    set reliable enable
    set ips-archive enable

    set certificate-verification enable

    set access-config enable

    set enc-algorithm low
    set ssl-min-proto-version default

    set conn-timeout 10
    set monitor-keepalive-period 5
    set monitor-failure-retry-period 5

    set interface-select-method auto

    set upload-option realtime

    set priority default

    set max-log-rate 0
end
```

### Important parameters

| Parameter                      | Meaning                                                        |
| ------------------------------ | -------------------------------------------------------------- |
| `reliable`                     | Reliable remote log transmission                               |
| `ips-archive`                  | Archive IPS-related information when supported                 |
| `certificate-verification`     | Verify remote certificate                                      |
| `access-config`                | Allows supported FortiAnalyzer management/configuration access |
| `enc-algorithm`                | Encryption algorithm selection                                 |
| `ssl-min-proto-version`        | Minimum TLS protocol                                           |
| `conn-timeout`                 | Connection timeout                                             |
| `monitor-keepalive-period`     | Keepalive monitoring interval                                  |
| `monitor-failure-retry-period` | Retry interval                                                 |
| `source-ip`                    | Source address for connection                                  |
| `interface-select-method`      | Interface selection behavior                                   |
| `upload-option`                | Log upload mode                                                |
| `priority`                     | Remote server priority                                         |
| `max-log-rate`                 | Maximum log transmission rate                                  |

### `max-log-rate`

```bash
set max-log-rate 0
```

Means:

```text
0 = No configured transmission rate limit
```

---

# 13. FortiAnalyzer Priority

When multiple FortiAnalyzer destinations are configured, priority can influence failover behavior.

Conceptually:

```text
Priority: default
        │
        ▼
Primary FAZ
        │
        X
        │
        ▼
Secondary / Lower Priority FAZ
```

A lower-priority destination can be used as a disaster-recovery path depending on the configuration.

---

# 14. FortiAnalyzer Reports Through Security Fabric

In a Security Fabric topology:

```text
                 FortiAnalyzer
                      ▲
                      │ REST API / Reporting
                      │
                Root FortiGate
                      ▲
                      │
             Security Fabric
             ┌────────┼────────┐
             │        │        │
            FGT-2    FGT-3    FGT-4
```

### Key concept

The **root FortiGate** has the central reporting role.

Downstream FortiGates can consume/view information, but they do not provide the same centralized reporting role as the root device.

---

# 15. FortiAnalyzer REST API Integration

After connecting FortiAnalyzer to FortiGate, FortiAnalyzer reports can become accessible through the FortiGate GUI.

Relevant configuration:

```bash
config log setting
    set rest-api-set enable
    set rest-api-get enable
end
```

### Important

These settings enable the required REST API interaction for the logging/reporting integration.

You do not necessarily need to create a separate REST API user on the FortiGate for this specific integration.

---

# 16. Log Forwarding Statistics

Useful command:

```bash
execute log list 0
```

Use it to inspect information related to logs and forwarding.

---

# 17. Local Disk Usage

Useful commands:

```bash
diagnose sys logdisk usage
```

```bash
diagnose sys logdisk quota
```

```bash
diagnose sys logdisk smart
```

```bash
diagnose sys logdisk status
```

### Quick interpretation

| Command  | Purpose                   |
| -------- | ------------------------- |
| `usage`  | Disk utilization          |
| `quota`  | Logging quota information |
| `smart`  | Disk health information   |
| `status` | Disk/logging status       |

---

# 18. Log Disk Configuration

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

> Parameter names can vary by FortiOS release. Always verify the exact CLI syntax on the target FortiOS version with `?` / CLI reference.

---

# 19. Log Quota

Example:

```bash
set log-quota 0
```

A value of `0` can represent an unlimited/default quota depending on the specific quota parameter and FortiOS release.

For example:

```text
1 TB disk
└── Log quota
      └── 50%
          └── ~500 GB allocation
```

> Do not interpret every `0` value universally. Verify the parameter semantics for the exact FortiOS release.

---

# 20. Log Age

Example:

```bash
set maximum-log-age 7
```

Conceptually:

```text
Day 1 ───────────── Day 7 ─────────────►
                         │
                         └── Old logs rotated/deleted
```

If logs must be retained longer:

```text
FortiGate
   │
   └──► FortiAnalyzer
          │
          └── Long-term retention
```

---

# 21. Disk Full Behavior

Two important behaviors:

```text
diskfull overwrite
```

or:

```text
diskfull nolog
```

### Overwrite

Older logs can be overwritten as disk capacity is exhausted.

### No-log

New logging stops when the configured storage reaches its limit.

> **Security implication:** `nolog` can create a visibility gap. For most environments, centralized remote logging is preferable so local disk exhaustion does not become the only logging point.

---

# 22. Disk Warning Thresholds

Example:

```text
75% ──► First warning
90% ──► Second warning
95% ──► Final warning
100% ─► Disk full behavior
```

Example configuration:

```bash
set full-first-warning-threshold 75
set full-second-warning-threshold 90
set full-final-warning-threshold 95
```

---

# 23. Log File Rolling

Example:

```bash
set roll-schedule daily
set roll-time 00:00
```

Maximum file size:

```bash
set max-log-file-size 20
```

Concept:

```text
Log File
   │
   ├── reaches size limit
   │
   └──► New log file
```

or:

```text
00:00
 │
 └──► Scheduled roll
```

---

# 24. Local Disk Upload to FTP

Example:

```bash
config log disk setting
    set status enable
    set upload enable

    set uploadport 21
    set upload-destination ftp-server

    set uploadtype traffic event virus webfilter ips emailfilter \
        dlp-archive anomaly voip dlp app-ctrl waf gtp dns ssh ssl

    set uploaddir /
    set uploadsched enable
    set uploadtime 00:00

    set upload-delete-files enable
end
```

### Important

Synchronize:

```text
Log Rolling
      +
FTP Upload Schedule
```

Example design:

```text
00:00 ─── Log Roll
00:10 ─── Upload
```

A small delay between rolling and upload can provide a safer operational window.

---

# 25. FortiAnalyzer vs FTP

When both are configured:

```text
FortiGate
   │
   ├── FortiAnalyzer
   │
   └── FTP
```

FortiAnalyzer is generally the preferred enterprise logging destination because it provides centralized:

* Analytics
* Search
* Reporting
* Log management
* Correlation
* Retention

> **Best practice:** Use FortiAnalyzer as the primary enterprise log platform rather than treating FTP as the main analytics solution.

---

# 26. Syslog Integration

FortiGate can forward logs to external Syslog servers.

Example:

```bash
config log syslogd
    set status enable
    set format default
    set server 192.168.254.254
end
```

Depending on the platform/release, multiple Syslog destinations can be configured.

### Common formats

```text
default
CSV
CEF
```

### CEF

CEF is particularly useful for SIEM integrations.

```text
FortiGate
   │
   │ CEF
   ▼
Syslog
   │
   ▼
SIEM
   ├── Splunk
   ├── ArcSight
   └── Other SIEM platforms
```

---

# 27. Syslog Filtering

Example:

```bash
config log syslogd filter
    set severity information
    set forward-traffic enable
    set local-traffic enable
    set multicast-traffic enable
    set sniffer-traffic enable
end
```

Filtering prevents unnecessary log categories from being forwarded.

---

# 28. FortiAnalyzer Log Filtering

Example:

```bash
config log fortianalyzer filter
    set severity information
    set forward-traffic enable
    set local-traffic enable
    set multicast-traffic enable
    set sniffer-traffic enable
end
```

### Logging pipeline

```text
Generated Logs
      │
      ▼
Log Filter
      │
      ├── Severity
      ├── Traffic
      ├── Security
      ├── UTM
      └── Event categories
      │
      ▼
FortiAnalyzer / Syslog
```

---

# 29. Traffic Log Categories

Common categories include:

* Forward traffic
* Local traffic
* Multicast traffic
* Sniffer traffic
* Security events
* IPS
* Antivirus
* Web Filter
* Application Control
* DNS
* SSH
* SSL
* WAF
* DLP
* VoIP
* Anomaly

Do not blindly enable every category on high-volume systems.

> **Design principle:** Log what you need to detect, investigate, audit and prove — not simply everything the firewall can generate.

---

# 30. Local Traffic Logging

Local traffic logging can generate a significant volume of logs.

For this reason, it may be disabled by default depending on the FortiOS configuration/platform.

Example:

```bash
config log setting
    set local-traffic-log enable
end
```

> Verify the exact parameter name for your FortiOS release.

---

# 31. UUIDs in Traffic Logs

UUIDs can make log correlation easier.

## Policy UUID

```text
Policy
  │
  └── UUID
       │
       └── Traffic Log
```

Useful for:

* SIEM correlation
* Policy identification
* Automation
* API-based analysis

## Address UUID

Address UUIDs can also be included in traffic logs.

Useful when correlating:

```text
Source Address
Destination Address
Policy
Traffic Log
```

---

# 32. VDOM Logging Overrides

In multi-VDOM deployments, individual VDOMs can override global logging destinations.

Example:

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
      │      └── Override → FAZ-A
      │
      ├── VDOM-B
      │      └── Override → Syslog-B
      │
      └── VDOM-C
             └── Global destination
```

If override is disabled, the VDOM uses the global FortiAnalyzer/Syslog settings.

---

# 33. Multiple FortiAnalyzer / Syslog Servers per VDOM

A VDOM can support multiple override destinations depending on FortiOS release/platform.

Typical design:

```text
VDOM
 ├── FAZ Override 1
 ├── FAZ Override 2
 ├── FAZ Override 3
 │
 ├── Syslog Override 1
 ├── Syslog Override 2
 ├── Syslog Override 3
 └── Syslog Override 4
```

> Always verify platform-specific limits on the target FortiOS release.

---

# 34. VDOM Override — FortiAnalyzer

Example:

```bash
config log fortianalyzer3 override-setting
    set status enable
    set server 123.12.123.123
    set reliable enable
end
```

Override filtering:

```bash
config log fortianalyzer3 override-filter
    set severity information
    set forward-traffic enable
    set local-traffic enable
    set multicast-traffic enable
    set sniffer-traffic enable
    set anomaly enable
    set voip enable
    set dlp-archive enable
    set dns enable
    set ssh enable
    set ssl enable
end
```

---

# 35. VDOM Override — Syslog

Example:

```bash
config log syslogd4 override-setting
    set status enable
    set server 123.12.123.12
    set facility local1
end
```

Override filter:

```bash
config log syslogd4 override-filter
    set severity information
    set forward-traffic enable
    set local-traffic enable
    set multicast-traffic enable
    set sniffer-traffic enable
    set anomaly enable
    set voip enable
    set dns enable
    set ssh enable
    set ssl enable
end
```

---

# 36. Useful VDOM Log Settings

Example:

```bash
config log setting
    set faz-override enable
    set syslog-override enable

    set resolve-ip enable
    set resolve-port enable

    set log-user-in-upper disable

    set fwpolicy-implicit-log disable
    set fwpolicy6-implicit-log disable

    set log-invalid-packet disable

    set local-in-allow disable
    set local-in-deny-unicast disable
    set local-in-deny-broadcast disable

    set local-out enable
    set local-out-ioc-detection enable

    set daemon-log disable

    set neighbor-event disable

    set brief-traffic-format disable

    set user-anonymize disable

    set expolicy-implicit-log disable

    set log-policy-comment disable

    set rest-api-set disable
    set rest-api-get disable
end
```

### Important settings

| Setting                  | Purpose                             |
| ------------------------ | ----------------------------------- |
| `resolve-ip`             | Resolve/log IP-related information  |
| `resolve-port`           | Resolve service/port information    |
| `fwpolicy-implicit-log`  | Log implicit IPv4 policy traffic    |
| `fwpolicy6-implicit-log` | Log implicit IPv6 policy traffic    |
| `log-invalid-packet`     | Log invalid packets                 |
| `local-in-allow`         | Local-in allowed traffic logging    |
| `local-in-deny-*`        | Local-in denied traffic logging     |
| `local-out`              | Local-out traffic logging           |
| `daemon-log`             | Background daemon logging           |
| `neighbor-event`         | Neighbor/discovery-related events   |
| `user-anonymize`         | Anonymize logged users              |
| `rest-api-*`             | Logging/reporting REST API behavior |

---

# 37. IP & Port Resolution

Recommended when human-readable logs are important:

```bash
set resolve-ip enable
set resolve-port enable
```

Concept:

```text
10.10.10.10
     │
     ▼
Hostname / resolved representation

TCP/443
   │
   ▼
HTTPS
```

> Be aware that name/port resolution can add processing overhead and may influence log presentation.

---

# 38. User Anonymization

```bash
set user-anonymize enable
```

Conceptually:

```text
Real User
   │
   ▼
Anonymized / hashed representation
```

Disable:

```bash
set user-anonymize disable
```

This allows real user identity to appear in applicable logs.

> **Privacy requirement:** User-identifying logs should be handled according to organizational policy and applicable privacy regulations.

---

# 39. FortiAnalyzer Cloud

FortiAnalyzer Cloud provides cloud-based log storage and analytics.

Two subscription concepts in the notes:

| Subscription | Description                          |
| ------------ | ------------------------------------ |
| **FAZC**     | Standard FortiAnalyzer Cloud         |
| **AFAC**     | Advanced/Premium FortiAnalyzer Cloud |

Capabilities depend on the active contract/subscription and FortiOS release.

---

# 40. FortiAnalyzer Cloud — Configuration

Example:

```bash
config log fortianalyzer-cloud setting
    set status enable
    set ips-archive disable

    set certificate-verification enable

    set enc-algorithm low
    set ssl-min-proto-version default

    set conn-timeout 10
    set monitor-keepalive-period 5
    set monitor-failure-retry-period 5

    set interface-select-method auto

    set upload-option 5-minute

    set priority default

    set max-log-rate 0
end
```

> For Internet/cloud-based communication, prefer the strongest encryption setting supported by your FortiOS release and deployment requirements.

---

# 41. FortiAnalyzer Cloud Connectivity

Test:

```bash
execute log fortianalyzer-cloud test-connectivity
```

Check contract/update information:

```bash
diagnose test update info
```

---

# 42. FortiAnalyzer Cloud Log Filtering

Traffic logs:

```bash
execute log filter device fortianalyzer-cloud
execute log filter category traffic
execute log filter dump
execute log display
```

UTM / Antivirus example:

```bash
execute log filter device fortianalyzer-cloud
execute log filter category utm-virus
execute log filter dump
execute log display
```

Useful for confirming whether specific categories are being generated and forwarded.

---

# 43. FortiAnalyzer Cloud — Important Limitation

Capabilities depend on subscription.

The notes identify:

### Standard / FAZC

May provide categories such as:

* UTM
* IPS
* Security profiles
* Event logs

Advanced traffic/log capabilities can depend on the subscription and release.

### Premium / AFAC

Provides broader cloud logging/analytics capabilities and may include advanced analytics/AI features.

> **Important:** FortiAnalyzer Cloud feature availability changes over time. Verify the active contract and FortiOS/FortiAnalyzer Cloud documentation before designing retention or archive requirements.

---

# 44. `fgtlogd` Troubleshooting

Start with:

```bash
diagnose test application fgtlogd 1
```

Useful test modes:

|    ID | Purpose                                    |
| ----: | ------------------------------------------ |
|   `1` | Show global log settings/report            |
|   `2` | Show VDOM log settings                     |
|   `3` | Show detailed log statistics               |
|   `4` | Dump statistics                            |
|   `5` | Show dropped logs due to log-rate limiting |
|   `6` | Show subscribed log information            |
|   `7` | Show subscribed log filters                |
|   `9` | Show remote socket                         |
|  `10` | Enable/disable log packet dump             |
|  `20` | Show FortiCloud log state                  |
|  `30` | Show remote queues/items                   |
|  `37` | Enable/disable FAZ/FDS packet dumping      |
|  `38` | Delete FAZ/FDS dump files                  |
|  `39` | Backup FAZ/FDS dump files to USB           |
|  `41` | Show remote queues                         |
| `101` | Show root VDOM log settings                |

### Critical troubleshooting commands

```bash
diagnose test application fgtlogd 1
```

```bash
diagnose test application fgtlogd 3
```

```bash
diagnose test application fgtlogd 5
```

```bash
diagnose test application fgtlogd 30
```

---

# 45. `logsync-enable`

When inspecting `fgtlogd` global settings:

```text
logsync-enable 1
```

indicates that log synchronization with a remote logging destination such as FortiAnalyzer is enabled.

Concept:

```text
FortiGate
   │
   └── logsync-enable = 1
          │
          ▼
     Remote Sync
```

---

# 46. `miglogd` Troubleshooting

Basic:

```bash
diagnose test application miglogd 1
```

VDOM-specific:

```bash
diagnose test application miglogd 1 vdom root
```

### Important `miglogd` test IDs

|    ID | Function                            |
| ----: | ----------------------------------- |
|   `1` | Show global log settings            |
|   `2` | Show VDOM log settings              |
|   `3` | Show log buffer size                |
|   `4` | Show log statistics                 |
|   `5` | Show maximum file descriptor number |
|   `6` | Dump statistics                     |
|   `9` | Delete policy sniffer files         |
|  `10` | Show CID cache                      |
|  `11` | Show UTM traffic cache              |
|  `12` | Show policy cache                   |
|  `13` | Increase number of miglog children  |
|  `14` | Decrease number of miglog children  |
|  `15` | Show miglog ID                      |
|  `16` | Show log disk usage                 |
|  `18` | Show network interface cache        |
|  `19` | Show application cache              |
|  `24` | Show WLAN AP cache                  |
|  `26` | Enable/disable log dumping          |
|  `27` | Show DNS cache                      |
|  `28` | Show miglog shared memory           |
|  `31` | Show log dump file content          |
|  `32` | Delete log dump file                |
|  `33` | List log dump files                 |
|  `34` | Backup log dump files to USB        |
|  `36` | Show memory log file lists          |
|  `42` | Show UUID cache                     |
|  `43` | Show ISDB cache                     |
|  `44` | Show ISDB country cache             |
|  `45` | Show ISDB region cache              |
|  `46` | Show ISDB city cache                |
|  `47` | Show remote sockets                 |
|  `48` | Show publish information            |
|  `49` | Show published logs                 |
|  `50` | Show Threat Feed cache              |
|  `51` | Show dynamic interface cache        |
| `101` | Show root VDOM log settings         |
| `102` | Show application custom cache       |
| `103` | Show application list cache         |
| `104` | Show UTM traffic cache              |
| `105` | Show reputation traffic cache       |
| `106` | Show firewall service cache         |

---

# 47. `miglogd` Troubleshooting Flow

Use this sequence when local logging behaves unexpectedly:

```text
START
  │
  ▼
Is logging enabled?
  │
  ├── NO ──► Enable required log destination
  │
  └── YES
       │
       ▼
Check disk
       │
       ├── diagnose sys logdisk usage
       ├── diagnose sys logdisk quota
       ├── diagnose sys logdisk smart
       └── diagnose sys logdisk status
       │
       ▼
Check miglogd
       │
       ├── buffer size
       ├── statistics
       ├── disk usage
       ├── caches
       └── shared memory
       │
       ▼
Check fgtlogd
       │
       ├── remote queues
       ├── dropped logs
       ├── subscriptions
       └── remote sockets
       │
       ▼
Check FortiAnalyzer/Syslog
       │
       ▼
END
```

---

# 48. Log Rate Limiting

If a maximum log transmission rate is configured:

```bash
set max-log-rate <value>
```

then logs can be dropped when the configured rate is exceeded.

Check:

```bash
diagnose test application fgtlogd 5
```

This helps identify logs dropped because of the configured transmission rate.

---

# 49. Security Fabric Logging Model

Recommended enterprise architecture:

```text
                         ┌──────────────────┐
                         │  FortiAnalyzer   │
                         └────────▲─────────┘
                                  │
                            Central Logs
                                  │
                         ┌────────┴────────┐
                         │ Root FortiGate  │
                         │                 │
                         │ Central Report  │
                         └────────┬────────┘
                                  │
                    ┌─────────────┼─────────────┐
                    ▼             ▼             ▼
                  FGT-2         FGT-3         FGT-4
               Downstream     Downstream     Downstream
```

### Design principle

```text
Local FortiGate logging
        +
Central FortiAnalyzer
        =
Better visibility + retention + analytics
```

---

# 50. Threat Weight

Navigate to:

```text
Log & Report
└── Threat Weight
```

For IPS-related environments, assign appropriate threat severity/weighting.

### Recommended approach

Critical IPS events should have a sufficiently high threat weight so they are clearly prioritized in:

* Reports
* Dashboards
* Security monitoring
* Incident investigation

---

# 51. Practical Enterprise Logging Blueprint

### Small Environment

```text
FortiGate
   │
   ├── Local Disk
   │
   └── Syslog
```

### Medium Environment

```text
FortiGate
   │
   ├── Local Disk
   │
   └── FortiAnalyzer
```

### Enterprise Environment

```text
                    FortiAnalyzer
                         ▲
                         │
                 Centralized Logging
                         │
               ┌─────────┴─────────┐
               │   Root FortiGate  │
               └─────────┬─────────┘
                         │
             Security Fabric
          ┌──────────┬────┴────┬──────────┐
          ▼          ▼         ▼          ▼
        FGT-2      FGT-3     FGT-4      FGT-5
          │          │         │          │
          └──────────┴─────────┴──────────┘
                         │
                       Logs
```

Optional:

```text
FortiAnalyzer
      │
      └──► SIEM / SOC
```

---

# 52. Recommended Log Strategy

### Tier 1 — FortiGate

Enable the logs required for:

* Traffic visibility
* Security events
* Authentication
* Administration
* IPS
* Antivirus
* Web Filtering
* Application Control
* VPN
* HA
* System events

### Tier 2 — FortiAnalyzer

Use for:

* Central storage
* Search
* Reporting
* Correlation
* Long-term retention
* Incident investigation

### Tier 3 — SIEM

Forward selected high-value logs for:

* Correlation
* SOC monitoring
* Alerting
* Threat detection
* Compliance

---

# 53. Logging Best Practices

> **Do**

* Use FortiAnalyzer for centralized enterprise logging.
* Monitor local disk usage.
* Configure disk warning thresholds.
* Use appropriate log filters.
* Keep FortiOS and FortiAnalyzer versions compatible.
* Monitor remote logging queues.
* Verify FortiAnalyzer authorization/connectivity.
* Use reliable logging when required.
* Separate high-volume logs from critical security logs where appropriate.
* Protect log infrastructure from unauthorized access.
* Define retention requirements before choosing local/cloud storage.

> **Don't**

* Enable every possible log category without evaluating volume.
* Depend exclusively on local FortiGate logs for long-term retention.
* Ignore disk usage.
* Ignore dropped-log counters.
* Use weak encryption when stronger supported options are available.
* Treat FTP as a replacement for a centralized log analytics platform.
* Assume every FortiAnalyzer Cloud feature is included in every subscription.

---

# 54. Fast Troubleshooting Checklist

## Local Logs Missing

```text
[ ] Is local disk logging enabled?
[ ] Is the disk detected?
[ ] Is the disk healthy?
[ ] Is the disk full?
[ ] Is log quota configured?
[ ] Is maximum log age appropriate?
[ ] Is logging filtered?
[ ] Is miglogd processing logs?
```

Commands:

```bash
diagnose sys logdisk status
diagnose sys logdisk usage
diagnose sys logdisk smart
diagnose test application miglogd 1
diagnose test application miglogd 4
diagnose test application miglogd 16
```

---

## FortiAnalyzer Not Receiving Logs

```text
[ ] FortiAnalyzer IP correct?
[ ] FortiAnalyzer reachable?
[ ] FortiGate authorized?
[ ] Certificate verification successful?
[ ] Upload option correct?
[ ] Reliable mode required?
[ ] Log filter allowing the category?
[ ] Remote queue growing?
[ ] Logs dropped because of rate limit?
```

Commands:

```bash
execute log fortianalyzer test-connectivity
diagnose test application fgtlogd 1
diagnose test application fgtlogd 3
diagnose test application fgtlogd 5
diagnose test application fgtlogd 30
```

---

## Syslog Not Receiving Logs

```text
[ ] Syslog status enabled?
[ ] Correct server IP?
[ ] Correct format?
[ ] Correct facility?
[ ] Correct log filter?
[ ] Network path available?
[ ] UDP/TCP transport requirements verified?
[ ] Syslog server listening?
```

Example:

```bash
config log syslogd
    set status enable
    set format default
    set server 192.168.254.254
end
```

---

# 55. Exam / Interview Quick Recall

### `miglogd`

> **Local log processing, storage, caches and disk-related operations.**

### `fgtlogd`

> **Remote logging, synchronization, subscriptions, queues and forwarding.**

### `realtime`

> Forward logs continuously.

### `store-and-upload`

> Store/cache logs locally before uploading.

### `max-log-rate 0`

> No configured transmission-rate limit.

### `resolve-ip`

> Resolve IP-related information for logging/presentation.

### `resolve-port`

> Resolve service/port information.

### `faz-override`

> Allows a VDOM to use its own FortiAnalyzer logging destination.

### `syslog-override`

> Allows a VDOM to use its own Syslog destination.

### `75% / 90% / 95%`

> Typical first/second/final disk warning thresholds in the reference configuration.

### `diskfull overwrite`

> Continue logging by overwriting older log data when the disk is full.

### `FortiAnalyzer`

> Preferred enterprise platform for centralized FortiGate logging, analytics and reporting.

---

# 56. One-Minute Mental Model

```text
                  FORTIGATE LOGGING
                         │
          ┌──────────────┴──────────────┐
          │                             │
       miglogd                        fgtlogd
          │                             │
    Local Processing              Remote Processing
          │                             │
    ┌─────┼─────┐              ┌───────┼────────┐
    │     │     │              │       │        │
 Buffer  Disk  Cache          FAZ    Cloud    Syslog
    │     │     │
    └─────┴─────┘
          │
       FortiView
          │
          ▼
       Reporting
```

### Golden Rule

```text
Generate
   ↓
Filter
   ↓
Process
   ↓
Store / Buffer
   ↓
Forward
   ↓
Analyze
   ↓
Retain
```

---

## 🔥 SheynShield Production Checklist

```text
[ ] Local logging requirement defined
[ ] Local disk detected and healthy
[ ] Disk logging configured
[ ] Log retention defined
[ ] Disk warning thresholds configured
[ ] FortiAnalyzer connected
[ ] FortiAnalyzer authorization verified
[ ] Upload mode selected
[ ] Reliable logging evaluated
[ ] Log filters optimized
[ ] Syslog/SIEM integration evaluated
[ ] VDOM overrides configured where required
[ ] Historical FortiView requirement evaluated
[ ] Local reports enabled only when required
[ ] Log rate limits reviewed
[ ] Dropped logs monitored
[ ] miglogd health checked
[ ] fgtlogd queues checked
[ ] FortiAnalyzer compatibility verified
[ ] Cloud subscription verified
[ ] Security Fabric root logging architecture validated
[ ] Long-term retention delegated to centralized platform
```

---

## ⚡ CLI Quick Reference

```bash
# Local disk
diagnose sys logdisk usage
diagnose sys logdisk quota
diagnose sys logdisk smart
diagnose sys logdisk status

# FortiAnalyzer
execute log fortianalyzer test-connectivity
diagnose test application fgtlogd 1
diagnose test application fgtlogd 3
diagnose test application fgtlogd 5
diagnose test application fgtlogd 30

# miglogd
diagnose test application miglogd 1
diagnose test application miglogd 2
diagnose test application miglogd 3
diagnose test application miglogd 4
diagnose test application miglogd 16
diagnose test application miglogd 47

# FortiView
diagnose fortiview result event-log
diagnose fortiview result security-log
diagnose fortiview result security-log action-block
```

---

## 🧠 SheynShield Takeaway

> **FortiGate logging is not simply “enable logs and send them to FortiAnalyzer.”**
>
> A production logging design must consider:
>
> **Log generation → filtering → buffering → local storage → remote forwarding → queue health → retention → analytics → SIEM integration.**
>
> For enterprise deployments, the strongest architecture is generally:
>
> **FortiGate → FortiAnalyzer → SIEM/SOC**
>
> with local disk used as a controlled buffer/cache rather than the only source of historical security evidence.
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

# FortiGate Tabs & FortiView 

> **FortiOS CLI Quick Reference**
> Log GUI Display • Application Visibility • FortiView • Historical Data • Report Database
>
> 🎯 **NSE Focus:** Understand what controls application/hostname visibility, where FortiView data comes from, and how to troubleshoot stale or inconsistent report data.

---

## 1. Log GUI Display

FortiGate can resolve additional information before displaying traffic in the GUI, including:

* Hostnames
* Application names
* Previously unidentified applications

### Configuration

```bash
config log gui-display
    set resolve-hosts enable
    set resolve-apps enable
    set fortiview-unscanned-apps enable
end
```

### Key Options

| Setting                    | Purpose                                                        |
| -------------------------- | -------------------------------------------------------------- |
| `resolve-hosts`            | Resolve IP addresses to hostnames in GUI log display           |
| `resolve-apps`             | Resolve application information for log display                |
| `fortiview-unscanned-apps` | Display applications that were not identified/scanned normally |

---

## 2. `resolve-hosts`

```bash
config log gui-display
    set resolve-hosts enable
end
```

### Purpose

Allows FortiGate to resolve destination/source IP addresses into hostnames for display.

### Concept

```text
IP Address
    │
    ▼
Hostname Resolution
    │
    ▼
GUI Logs
    │
    └── More readable host information
```

> 💡 **Operational Tip:**
> Hostname resolution improves readability but can introduce additional DNS-related activity.

---

# 3. `resolve-apps`

```bash
config log gui-display
    set resolve-apps enable
end
```

This controls whether FortiGate attempts to resolve application information for GUI log display.

### Why Is It Useful?

Application Control may encounter traffic where the application cannot immediately be retrieved from the application database.

Enabling application resolution can provide more useful application information in displayed logs.

```text
Traffic
   │
   ▼
Application Detection
   │
   ├── Application found
   │       └── Display application
   │
   └── Application not immediately identified
           │
           ▼
      Application resolution
           │
           ▼
      More useful GUI log
```

---

# 4. `fortiview-unscanned-apps`

```bash
config log gui-display
    set fortiview-unscanned-apps enable
end
```

### Purpose

Allows FortiView to display applications that were not scanned/identified through the normal application inspection process.

### ⭐ Exam Concept

```text
Application Control
        │
        ├── Known / scanned application
        │        └── Application information available
        │
        └── Unscanned / unidentified application
                 │
                 └── fortiview-unscanned-apps
                            │
                            ▼
                    Can appear in FortiView
```

> **Remember:** This setting is primarily about **visibility in FortiView**, not magically turning unidentified traffic into fully inspected application traffic.

---

# 5. FortiView

**FortiView** provides a graphical view of traffic, applications, users, destinations, sources, and other traffic-related information.

Its available historical visibility depends heavily on:

* FortiGate model
* Storage type/capacity
* Logging configuration
* Traffic being logged
* Report database/history settings

---

# 6. FortiView Historical Data & Storage

Historical FortiView retention varies by platform and storage capability.

### General Model Classes

| Platform                                      | Approximate Historical FortiView Data |
| --------------------------------------------- | ------------------------------------- |
| Desktop models / 100-series with SSD          | ~5 minutes and ~1 hour views          |
| Medium models with SSD                        | Up to ~24 hours                       |
| Large models such as 1500D and above with SSD | Up to ~7 days                         |

> ⚠️ **Important:** Exact retention depends on the specific FortiGate model, FortiOS version, storage configuration, logging volume, and enabled features. Treat these values as a study reference rather than a universal guarantee.

---

# 7. Weekly FortiView Data

Weekly FortiView data can be enabled through:

```bash
config log setting
    set fortiview-weekly-data enable
end
```

### Concept

```text
Normal FortiView Data
        │
        ├── Short-term historical data
        │
        └── Weekly data
               │
               ▼
       fortiview-weekly-data
```

### Why Enable It?

Useful when you need broader historical visibility and reporting beyond short-term FortiView data.

---

# 8. Enable Disk Logging

FortiView/reporting relies on available logging data.

Disk logging can be enabled with:

```bash
config log disk setting
    set status enable
end
```

### Conceptual Flow

```text
Traffic
   │
   ▼
FortiGate Logging
   │
   ▼
Disk Storage
   │
   ├── Logs
   ├── FortiView data
   └── Report data
```

> 🧠 **NSE Tip:**
> If you expect historical FortiView/report information, verify that the required logging/storage configuration is actually enabled.

---

# 9. Report Data Sources

FortiGate reports can use different traffic sources.

```bash
config report setting
    set report-source forward-traffic sniffer-traffic local-deny-traffic
end
```

### Report Sources

| Source               | Meaning                                        |
| -------------------- | ---------------------------------------------- |
| `forward-traffic`    | Forwarded traffic                              |
| `sniffer-traffic`    | Traffic captured through sniffer functionality |
| `local-deny-traffic` | Locally denied traffic                         |

### Traffic Model

```text
                 Report Source
                      │
        ┌─────────────┼─────────────┐
        │             │             │
        ▼             ▼             ▼
 Forward Traffic   Sniffer      Local Deny
        │          Traffic       Traffic
        └─────────────┬─────────────┘
                      ▼
                Report Database
                      │
                      ▼
                   Reports
```

---

# 10. Report Cache & Database Troubleshooting

If FortiView or reports show inconsistent/irregular information, the report cache/database can be rebuilt.

### Flush Report Cache

```bash
execute report flush-cache
```

### Recreate Report Database

```bash
execute report recreate-db
```

These commands can help clean up irregularities caused by:

* Firmware upgrades
* Cached report data
* Report database inconsistencies

---

# 11. `flush-cache` vs `recreate-db`

| Command                      | Purpose                      |
| ---------------------------- | ---------------------------- |
| `execute report flush-cache` | Flush report cache           |
| `execute report recreate-db` | Recreate the report database |

### Troubleshooting Flow

```text
FortiView / Report looks abnormal
              │
              ▼
       Check logging/storage
              │
              ▼
       Check report sources
              │
              ▼
       Flush report cache
              │
              ▼
       Still inconsistent?
              │
              ▼
       Recreate report DB
```

> ⚠️ **Operational Warning:**
> Rebuilding a report database is a troubleshooting operation. Use it deliberately, especially on production devices where reporting data is important.

---

# 12. Full Configuration Example

```bash
# ==========================================
# LOG GUI DISPLAY
# ==========================================

config log gui-display
    set resolve-hosts enable
    set resolve-apps enable
    set fortiview-unscanned-apps enable
end


# ==========================================
# FORTIVIEW WEEKLY DATA
# ==========================================

config log setting
    set fortiview-weekly-data enable
end


# ==========================================
# DISK LOGGING
# ==========================================

config log disk setting
    set status enable
end


# ==========================================
# REPORT SOURCES
# ==========================================

config report setting
    set report-source forward-traffic sniffer-traffic local-deny-traffic
end


# ==========================================
# REPORT TROUBLESHOOTING
# ==========================================

execute report flush-cache

execute report recreate-db
```

---

# 13. ⭐ NSE Quick Recall

| Requirement                      | Command / Setting                                   |
| -------------------------------- | --------------------------------------------------- |
| Resolve hostnames in GUI logs    | `set resolve-hosts enable`                          |
| Resolve application information  | `set resolve-apps enable`                           |
| Show unscanned apps in FortiView | `set fortiview-unscanned-apps enable`               |
| Enable weekly FortiView data     | `set fortiview-weekly-data enable`                  |
| Enable disk logging              | `set status enable` under `config log disk setting` |
| Configure report sources         | `config report setting`                             |
| Flush report cache               | `execute report flush-cache`                        |
| Recreate report database         | `execute report recreate-db`                        |

---

# 14. 🧠 High-Value Exam Distinctions

### GUI Log Display

```text
config log gui-display
```

Think:

> **"How should information be resolved/displayed in the GUI?"**

---

### FortiView Historical Data

Think:

> **"How much historical information can the platform retain?"**

Main factors:

```text
FortiGate Model
      +
Storage
      +
Logging
      +
FortiView Settings
      +
Traffic Volume
```

---

### Report Database

Think:

> **"Where do reports get their processed historical information?"**

Troubleshooting:

```bash
execute report flush-cache
execute report recreate-db
```

---

# 15. 🔥 30-Second Memory Map

```text
                 FORTIGATE VISIBILITY
                         │
          ┌──────────────┴──────────────┐
          │                             │
      GUI LOGS                       FORTIVIEW
          │                             │
          │                    ┌────────┴────────┐
          │                    │                 │
          ▼                    ▼                 ▼
 resolve-hosts          Historical Data    Weekly Data
 resolve-apps                │           fortiview-weekly-data
 unscanned-apps              │
          │                  ▼
          │              Storage
          │                  │
          └────────────┬─────┘
                       ▼
                  REPORTING
                       │
             ┌─────────┴─────────┐
             │                   │
          Sources             Database
             │                   │
      forward-traffic      flush-cache
      sniffer-traffic      recreate-db
      local-deny
```

---

## 🎯 Final NSE Takeaways

1. **`config log gui-display`** → controls how information is resolved/displayed in GUI logs.
2. **`resolve-hosts`** → hostname visibility.
3. **`resolve-apps`** → application information resolution.
4. **`fortiview-unscanned-apps`** → visibility of unscanned applications in FortiView.
5. **`fortiview-weekly-data`** → enables weekly FortiView data.
6. **Disk logging** must be considered when troubleshooting historical visibility.
7. **`forward-traffic`, `sniffer-traffic`, `local-deny-traffic`** are report data sources.
8. **`flush-cache`** → clear report cache.
9. **`recreate-db`** → rebuild the report database.
10. **FortiView retention is platform/storage dependent** — don't memorize a single retention value as universal.

> **SheynShield Rule:**
> **Visibility ≠ Inspection ≠ Logging ≠ Reporting.**
>
> FortiGate may be able to detect traffic, log it, process it for FortiView, and include it in reports through different mechanisms. Always identify **which layer the question is asking about**.

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

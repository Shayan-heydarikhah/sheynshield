# 🛡️ FortiGate Antivirus (AV) & File Inspection  

> **FortiOS Security Engineering Reference — AV Engine, Scan Modes, CDR, CIFS, FortiSandbox & Malware Feeds**

---

## 📌 Quick Mental Model

FortiGate can inspect files using different processing architectures depending on:

* Inspection mode: **Flow vs Proxy**
* AV scan mode: **Flow / Default / Legacy** depending on FortiOS release
* File size
* File type
* Archive/compression type
* Enabled security features
* Available system resources
* FortiSandbox configuration

### Simplified architecture

```text
                    ┌──────────────────────────┐
                    │       Client             │
                    └────────────┬─────────────┘
                                 │
                                 ▼
                    ┌──────────────────────────┐
                    │       FortiGate           │
                    │                          │
                    │  Protocol Options        │
                    │        ↓                 │
                    │  AV Engine               │
                    │        ↓                 │
                    │  ┌────────────────────┐  │
                    │  │ Flow / Stream      │  │
                    │  │ Proxy / Scanunit   │  │
                    │  └────────────────────┘  │
                    │        ↓                 │
                    │  CDR / Sandbox /        │
                    │  Threat Feeds           │
                    └────────────┬─────────────┘
                                 │
                                 ▼
                            Internet
```

---

# 1. AV Scan Modes

## 🔹 Flow-Based AV

Flow-based AV scanning is designed for **high-speed, low-resource inspection**.

```text
Client
  │
  │ file stream
  ▼
FortiGate
  │
  ├── pattern matching
  ├── DFA
  ├── IPS/AV engine
  └── on-the-fly inspection
  │
  ▼
Destination
```

### Characteristics

| Feature                 | Flow-Based                         |
| ----------------------- | ---------------------------------- |
| Full file buffering     | ❌                                  |
| On-the-fly inspection   | ✅                                  |
| Resource consumption    | Lower                              |
| Performance             | High                               |
| Large file handling     | Limited by inspection architecture |
| Deep archive processing | More limited                       |
| Scanunit dependency     | Depends on feature/file type       |
| Best use                | High-throughput inspection         |

### Important

Flow-based inspection generally does **not buffer the complete file** before forwarding traffic.

Instead:

```text
Receive chunk
     ↓
Inspect
     ↓
Forward / Drop
     ↓
Receive next chunk
```

This allows FortiGate to maintain high throughput with relatively low memory consumption.

---

# 2. Proxy-Based AV

Proxy inspection uses a more stateful file-processing architecture.

```text
Client
   │
   │ Download
   ▼
FortiGate
   │
   │ Buffer / Reassemble
   ▼
AV / Scanunit
   │
   ├── Antivirus
   ├── Archive extraction
   ├── CDR
   ├── DLP
   ├── Quarantine
   └── Sandbox
   │
   ▼
Client
```

### Key difference

> In proxy-based inspection, FortiGate can buffer and process the file before releasing it to the client.

This makes proxy mode suitable for features that require access to the **complete file**.

Examples:

* Content Disarm & Reconstruction
* Quarantine
* DLP
* Advanced AV processing
* Some archive inspection
* FortiSandbox inline scanning

---

# 3. Flow vs Proxy — Fast Comparison

| Capability                   |    Flow   |  Proxy |
| ---------------------------- | :-------: | :----: |
| Low latency                  |     ✅     |   ⚠️   |
| High throughput              |     ✅     |   ⚠️   |
| Full file buffering          |     ❌     |    ✅   |
| CDR                          | ❌/limited |    ✅   |
| Advanced file processing     |  Limited  |    ✅   |
| Inline Sandbox               |     ❌     |    ✅   |
| Memory consumption           |   Lower   | Higher |
| Large-file processing        |  Limited  | Better |
| Complete file reconstruction |     ❌     |    ✅   |

> **Rule of thumb:**
> **Flow = speed**
> **Proxy = deeper file processing**

---

# 4. Historical AV Scan Architecture

FortiOS versions have used different terminology and architectures.

A useful conceptual mapping is:

```text
Older architecture
───────────────────

Default
   ├── Streaming / in-process scanning
   └── WAD-assisted processing

Legacy
   └── Buffer complete content
        ↓
      Scanunit


Newer architecture
───────────────────

Flow
   └── On-the-fly / DFA-oriented inspection

Proxy
   └── Stateful buffering
        ↓
      Scanunit
```

> ⚠️ Exact scan-mode behavior and available options depend on FortiOS release and platform.

---

# 5. WAD vs Scanunit

## WAD

**WAD = Web Application Daemon**

WAD participates heavily in proxy/web traffic processing and can perform relatively lightweight in-process inspection.

Conceptually:

```text
Traffic
   ↓
WAD
   ↓
Quick inspection
   ↓
Forward / Drop
```

Useful when:

* Low-latency inspection is desired
* Streaming inspection is sufficient
* The file/feature does not require full buffering

---

## Scanunit

`scanunit` is used for heavier file-scanning operations.

```text
File
 ↓
Buffer
 ↓
Scanunit
 ├── AV
 ├── decompression
 ├── advanced analysis
 └── file inspection
```

### Why Scanunit matters

Large or complex files may require:

* Complete buffering
* Archive extraction
* Deep scanning
* Advanced AV processing

Therefore:

> **Scanunit = heavier file processing**

---

# 6. Stream-Based Scanning

Stream-based scanning inspects data while the file is being transferred instead of requiring the entire file to be buffered first.

### Supported archive formats

Depending on FortiOS version, stream-based scanning supports formats such as:

```text
ZIP
GZIP
BZIP
BZIP2
TAR
ISO 9660
```

### Concept

```text
          File Stream
              │
              ▼
        ┌────────────┐
        │ AV Engine  │
        └─────┬──────┘
              │
       ┌──────┴──────┐
       │             │
    malicious       clean
       │             │
       ▼             ▼
     DROP          FORWARD
```

### Major advantage

A large archive can potentially be inspected **without buffering the entire archive in memory**.

---

# 7. Stream-Based Scanning Limitations

Stream-based scanning does not support every protocol/feature combination.

For example, depending on FortiOS version:

* HTTP/HTTPS support
* FTP/FTPS support
* SCP/SFTP support

may vary.

Some features can disable or limit stream-based processing, including:

* DLP
* Quarantine
* FortiGuard Outbreak Prevention
* External Block List
* Content Disarm & Reconstruction

When a file cannot be processed using stream-based scanning:

```text
Stream inspection
       │
       ▼
Unsupported file/feature
       │
       ▼
Buffer file
       │
       ▼
Scanunit
```

---

# 8. Oversized Files

FortiGate has finite resources for file buffering and scanning.

Therefore, an **oversize threshold** can be configured.

Example:

```cli
config firewall profile-protocol-options
    edit "proto-test"
        config ftp
            set oversize-limit 10000
        end
    next
end
```

> Check the actual unit and supported range for your FortiOS release/model before deployment.

### Concept

```text
File size
   │
   ├── <= threshold
   │      ↓
   │    Normal inspection
   │
   └── > threshold
          ↓
      Oversized handling
```

---

# 9. Oversized Archive vs Uncompressed Oversize

There are two different concepts to understand.

### Oversize

Size of the received/compressed object.

```text
ZIP = 500 MB
```

### Uncompressed oversize

Size after archive extraction.

```text
ZIP = 100 MB
        ↓
Extract
        ↓
Uncompressed content = 2 GB
```

This is especially important for protection against **archive/ZIP bomb attacks**.

Example:

```cli
config firewall profile-protocol-options
    edit "proto-test"

        config ftp
            set oversize-limit 300
            set uncompressed-oversize-limit 150
        end

    next
end
```

Concept:

```text
Compressed file
      │
      ▼
Extract
      │
      ▼
Uncompressed size
      │
      ├── within limit → inspect
      │
      └── exceeds limit → configured action
```

---

# 10. Legacy Scan Mode

Legacy mode can be useful for:

* Troubleshooting
* Compatibility
* Environments where stream-based scanning is unsuitable

Conceptually:

```text
Client
  ↓
FortiGate
  ↓
Buffer
  ↓
Scanunit
  ↓
AV inspection
  ↓
Release / Block
```

### Trade-off

| Benefit                    | Cost                      |
| -------------------------- | ------------------------- |
| More complete buffering    | More memory               |
| Better compatibility       | Higher latency            |
| Useful for troubleshooting | Lower throughput          |
| Advanced processing        | More resource consumption |

---

# 11. TCP Windowing & File Inspection

Large TCP windows can increase the amount of data that FortiGate must handle during stream inspection.

Conceptually:

```text
Initial window
     ↓
128 KB
     ↓
256 KB
     ↓
512 KB
     ↓
...
```

If packet loss or resource pressure occurs, TCP behavior may reduce the effective window.

### Why security engineers care

Large transfer windows + many simultaneous downloads can increase:

* Memory pressure
* Buffer requirements
* Inspection workload
* Conserve-mode risk

---

# 12. TCP Window Configuration

Example:

```cli
config firewall profile-protocol-options
    edit "proto-test"

        config ssh
            set stream-based-uncompressed-limit 1000

            set tcp-window-type static
            set tcp-window-size 1024

            set tcp-window-minimum 512
            set tcp-window-maximum 1500
        end

    next
end
```

### TCP window types

| Type          | Meaning                              |
| ------------- | ------------------------------------ |
| `system`      | System default                       |
| `static`      | Manually specified window            |
| `dynamic`     | Adjust according to available memory |
| `auto-tuning` | Automatically adapts values          |

> For production, choose the behavior based on actual memory/throughput testing rather than blindly using static values.

---

# 13. AV Machine-Learning Detection

FortiOS AV can integrate machine-learning-based detection into traditional antivirus processing.

The goal is to identify potentially malicious files that may not yet have a traditional signature.

```text
File
 ↓
Traditional AV
 ↓
ML / AI analysis
 ↓
File characteristics
 ↓
Malicious / Suspicious / Clean
```

### Why it matters

Traditional signatures are strongest against **known malware**.

Machine learning can help identify suspicious characteristics associated with previously unseen malware.

---

## Enable ML Detection

```cli
config antivirus settings
    set machine-learning-detection enable
end
```

> Availability and default state can vary by FortiOS version/model.

### Important

The AV machine-learning package is delivered through FortiGuard services and requires the appropriate subscription/support.

---

# 14. AI/ML vs Traditional Heuristics

### Traditional signature detection

```text
File
 ↓
Hash / signature
 ↓
Known malware?
 ├── YES → BLOCK
 └── NO  → continue
```

### ML-assisted detection

```text
File
 ↓
Extract features
 ↓
ML model
 ↓
Behavior / structural characteristics
 ↓
Risk classification
```

This is particularly valuable for detecting malware that does not yet have a traditional signature.

---

# 15. AV Database Modes

FortiGate can use different AV database sizes depending on platform/resources.

Conceptually:

### Extended DB

Suitable for platforms where resource efficiency is important.

```text
Recent / relevant signatures
        ↓
Lower resource footprint
```

### Extreme DB

Larger historical malware database.

```text
Large historical database
        ↓
Broader detection coverage
        ↓
Higher resource requirements
```

Example:

```cli
config antivirus settings
    set use-extreme-db enable
end
```

> Always verify the supported database modes for the specific FortiOS release and hardware platform.

---

# 16. Check AV / FortiGuard Database Status

Useful diagnostics:

```cli
diagnose autoupdate versions
```

```cli
diagnose autoupdate status
```

These help verify:

* AV database version
* FortiGuard update state
* Signature update information
* Update timestamps

---

# 17. Content Disarm & Reconstruction (CDR)

CDR is designed to sanitize potentially dangerous active content inside supported files.

Example:

```text
Office Document
      │
      ▼
   CDR engine
      │
      ├── inspect
      ├── identify active content
      ├── remove dangerous components
      └── reconstruct
              │
              ▼
       sanitized document
```

### Common example

A Microsoft Office document contains:

```text
Macro
External link
Embedded object
Active content
```

CDR can remove potentially dangerous components and reconstruct a safer version.

---

# 18. CDR Workflow

```text
Client
  │
  ▼
FortiGate
  │
  ▼
File received
  │
  ▼
AV / CDR inspection
  │
  ├── malicious/unsafe content
  │       ↓
  │    remove/sanitize
  │
  ▼
Reconstructed file
  │
  ▼
Client
```

### Important

CDR generally requires **proxy-based processing**, because FortiGate must access and reconstruct the file.

---

# 19. CDR + SMTP

SMTP processing can behave differently from HTTP/FTP because of protocol handling and streaming/splicing.

When CDR must inspect and modify email attachments, make sure the SMTP inspection architecture allows the complete content to be processed.

Typical concept:

```text
SMTP
 ↓
Proxy / appropriate scan mode
 ↓
Attachment extraction
 ↓
AV
 ↓
CDR
 ↓
Rebuild message
 ↓
Deliver
```

> Test SMTP behavior carefully in a lab because changing scan/splice behavior can affect mail-client compatibility and message handling.

---

# 20. CDR Configuration Concept

Example structure:

```cli
config firewall profile-protocol-options
    edit "proto-test"

        set scan-mode default

        config smtp
            ...
        end

        config ftp
            ...
        end

        config content-disarm
            set error-action block
            set detect-only disable
            set cover-page enable
        end

    next
end
```

### `detect-only`

If enabled, FortiGate can detect/report without necessarily performing the intended sanitization action.

For actual CDR enforcement, verify that the deployment is configured for **sanitization rather than detect-only behavior**.

---

# 21. FortiGuard Virus Outbreak Prevention (VOS)

Virus Outbreak Prevention provides an additional layer beyond the local AV signature database.

Concept:

```text
Local AV Database
        +
FortiGuard intelligence
        +
Threat intelligence
        ↓
Virus Outbreak Prevention
```

A newly identified malware hash can be distributed through FortiGuard intelligence so that protected FortiGate devices can react before traditional signature databases fully catch up.

---

## Diagnostics

```cli
diagnose debug rating
```

Useful for checking FortiGuard connectivity/licensing-related information.

---

# 22. Scanunit Debugging

To inspect scanunit behavior:

```cli
diagnose sys scanunit debug all
```

Useful when troubleshooting:

* File scanning
* Scanunit processing
* Hash lookups
* File-processing behavior

Additional debugging may be enabled with:

```cli
diagnose debug enable
```

> Use debug commands carefully in production; debugging can generate significant output.

---

# 23. External Malware Block List

FortiGate can consume external malware hash feeds.

Typical hash types include:

```text
MD5
SHA-1
SHA-256
```

Concept:

```text
External Threat Feed
        │
        ▼
   Malware Hash
        │
        ▼
FortiGate AV Engine
        │
        ├── Match → BLOCK
        └── No match → Continue
```

---

# 24. External Malware Feed + Security Fabric

Threat feeds can be integrated through the Security Fabric / external connectors.

Conceptually:

```text
Threat Intelligence
       │
       ▼
External Connector
       │
       ▼
FortiGate
       │
       ▼
AV Profile
       │
       ▼
Hash Matching
```

### Performance warning

Large external hash lists can increase lookup workload.

Avoid blindly importing enormous feeds without testing:

* CPU
* Memory
* Lookup latency
* Update frequency

---

# 25. External Malware Block List — Policy Example

```cli
config antivirus profile
    edit "av-test"

        config http
            set external-blocklist "malw-test"
        end

    next
end
```

Depending on FortiOS version and configuration, external blocklists may be applied globally or selectively.

---

# 26. EMS Malware Hash Feed

FortiClient EMS can provide malware intelligence to FortiGate.

Example:

```cli
config endpoint-control fctems
    edit "ems-test"
        set fortinetone-cloud-authentication disable
        set server 192.168.20.200
        set https-port 443
        set source-ip 0.0.0.0

        set pull-sysinfo enable
        set pull-vulnerabilities enable
        set pull-avatars enable
        set pull-tags enable
        set pull-malware-hash enable

        set call-timeout 30
        set websocket-override disable
    next
end
```

Enable EMS threat-feed usage in the AV profile:

```cli
config antivirus profile
    edit "av-test"
        set ems-threat-feed enable
    next
end
```

---

# 27. AV Detection Order — Mental Model

A useful troubleshooting model is:

```text
Incoming file
      │
      ▼
Local AV intelligence
      │
      ▼
EMS threat intelligence
      │
      ▼
External malware blocklist
      │
      ▼
FortiGuard / outbreak intelligence
      │
      ▼
Advanced analysis / Sandbox
```

> Exact processing order can vary by FortiOS release, feature configuration, and protocol.

---

# 28. AV Statistics

Useful commands:

```cli
diagnose ips av stats show
```

Clear statistics:

```cli
diagnose ips av stats clear
```

Use these during controlled testing to determine whether AV engines are actually processing traffic.

---

# 29. CIFS / SMB Antivirus Inspection

CIFS/SMB can be inspected using flow or proxy mechanisms depending on the traffic and feature requirements.

```text
SMB/CIFS
   │
   ├── Normal unencrypted traffic
   │       ├── Flow
   │       └── Proxy
   │
   └── Certain compression/encryption mechanisms
           ↓
       Proxy may be required
```

### File magic number

The initial bytes of a file can identify its type.

Conceptually:

```text
File
 ↓
First bytes
 ↓
Magic number
 ↓
File type identification
```

This is important for file filtering and content identification.

---

# 30. CIFS Archive Limitations

Proxy-based CIFS AV inspection can still have limitations, particularly around:

* Certain archive formats
* Oversized files
* Complex nested archives

Always verify supported archive/file types for the exact FortiOS release.

---

# 31. CIFS Credential / Decryption Considerations

For protected SMB/CIFS traffic, additional credentials/certificates may be required depending on the authentication/encryption architecture.

Concept:

```text
Encrypted CIFS
      │
      ▼
FortiGate
      │
      ├── authentication / credentials
      ├── certificate / key material
      └── protocol inspection
```

Do not assume that ordinary AV inspection can decrypt every SMB encryption scenario.

---

# 32. CIFS AV Configuration Example

```cli
config antivirus profile
    edit "av-test"

        config cifs
            set quarantine enable

            set archive-log encrypted corrupted partiallycorrupted multipart nested mailbomb timeout unhandled

            set archive-block encrypted corrupted partiallycorrupted multipart nested mailbomb timeout unhandled
        end

    next
end
```

---

# 33. File Filter + AV

File filtering can complement AV.

Example:

```cli
config file-filter profile
    edit "ff-test"

        set feature-set proxy

        config rules
            edit "ff-test-r"
                set protocol http ftp smtp imap pop3 mapi cifs ssh
                set action block
                set file-type pdf
            next
        end

    next
end
```

> The exact available protocols/file types depend on FortiOS release.

---

# 34. Security Policy — Complete File Inspection Stack

For deep inspection of encrypted web traffic, the policy may need:

```text
Firewall Policy
 │
 ├── Source
 ├── Destination
 ├── Service
 │
 ├── SSL/SSH Inspection
 │      └── Deep Inspection
 │
 ├── Protocol Options
 │      └── File-size / protocol behavior
 │
 ├── Antivirus
 │
 ├── File Filter
 │
 └── Other Security Profiles
        ├── Web Filter
        ├── Application Control
        ├── IPS
        └── DLP
```

### Important

> **HTTPS + AV without Deep Inspection ≠ full file visibility.**

Encrypted payloads must first become inspectable.

---

# 35. FortiSandbox — Post-Transfer Scanning

FortiSandbox adds behavioral analysis for suspicious files.

### Post-transfer model

```text
Client
  │
  │ Download
  ▼
FortiGate
  │
  └──────────────► FortiSandbox
                         │
                         ▼
                   Behavioral Analysis
                         │
                         ▼
                    Verdict / IOC
```

In post-transfer mode, the client can receive the file before the sandbox verdict is available.

Therefore:

> **Post-transfer = less user latency, but the file may reach the endpoint before sandbox verdict.**

---

# 36. FortiSandbox — Inline Scanning

Inline scanning holds the file while sandbox analysis takes place.

```text
Client
  │
  │ File
  ▼
FortiGate
  │
  ├── Hold
  │
  └──────────────► FortiSandbox
                         │
                    Analysis
                         │
                ┌────────┴────────┐
                │                 │
              Clean            Malicious
                │                 │
                ▼                 ▼
             Release             Block
```

### Main advantage

> **Inline = stronger pre-delivery protection**

### Main trade-off

> **Inline = additional latency and dependency on Sandbox availability.**

---

# 37. FortiSandbox Inline Configuration

Example:

```cli
config system fortisandbox
    set status enable
    set inline-scan enable
    set server 192.168.20.200
end
```

AV profile:

```cli
config antivirus profile
    edit "av-test"

        set fortisandbox-mode inline
        set fortisandbox-error-action block
        set fortisandbox-timeout-action block
        set fortisandbox-max-upload 10

    next
end
```

### Error action

Possible behaviors depend on FortiOS version:

```text
block
monitor/log
ignore/pass
```

### Security recommendation

For high-security environments:

```text
Sandbox failure
      ↓
BLOCK
```

This is a **fail-closed** approach.

---

# 38. FortiSandbox Timeout

Inline scanning introduces a timeout.

Conceptually:

```text
File
 ↓
Sandbox
 ↓
Wait
 │
 ├── Verdict before timeout → action
 │
 └── Timeout → configured timeout action
```

Possible policy:

```text
Timeout → BLOCK
```

This provides stronger security at the expense of availability/user experience.

---

# 39. FortiSandbox Upload Size

The AV profile can restrict the maximum file size submitted to FortiSandbox.

Example:

```cli
set fortisandbox-max-upload 10
```

Use an appropriate value based on:

* Internet bandwidth
* Sandbox capacity
* FortiGate resources
* Typical file sizes
* Security requirements

---

# 40. FortiSandbox File Types

Common supported categories include:

```text
7z
ARJ
BZIP
BZIP2
CAB
HTML
JS
LZH
LZW
RTF
TAR
VBA
VBS
WinPE / EXE
ELF
GZIP
Microsoft Office
PDF
RAR
XZ
ZIP
```

> Exact supported formats vary by FortiSandbox/FortiOS release and deployment.

---

# 41. APT Protection

APT protection can send suspicious or selected files to FortiSandbox.

Typical logic:

```text
File
 ↓
AV
 ↓
APT Protection
 ↓
FortiSandbox
 ↓
Behavioral analysis
 ↓
Verdict
```

### Scan strategies

```text
Inline
Post-transfer
```

### Submission selection

Possible approaches include:

```text
Suspicious files
All supported files
Exclude specific file types
Exclude file-name patterns
Use Sandbox database
```

---

# 42. FortiSandbox Diagnostics

Useful commands during troubleshooting include:

```cli
diagnose test application quarantined 7
```

```cli
diagnose test application quarantined -1
```

FortiCloud/Sandbox related testing:

```cli
diagnose test application forticldd 3
```

Sandbox submission statistics:

```cli
diagnose test application quarantined 2
```

For deeper debugging:

```cli
diagnose debug enable
```

and, where applicable:

```cli
diagnose debug device <device-id>
```

> Diagnostic command availability can vary by FortiOS release.

---

# 43. FortiGuard + FortiSandbox — Defense in Depth

Do not think of Sandbox as a replacement for AV.

A stronger architecture is:

```text
                 ┌──────────────────┐
                 │ Local AV Database│
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │ ML / AI Detection│
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │ Threat Feeds     │
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │ FortiSandbox     │
                 │ Behavioral Scan  │
                 └────────┬─────────┘
                          │
                          ▼
                     Final Verdict
```

---

# 44. Flow vs Proxy — Security Engineering Decision

## Choose Flow when:

* High throughput is critical
* Low latency matters
* Basic/on-the-fly inspection is sufficient
* Resource consumption must remain low
* Full file reconstruction is unnecessary

## Choose Proxy when:

* Deep file inspection is required
* CDR is required
* Full file buffering is acceptable
* Inline Sandbox is required
* Complex archive inspection is needed
* DLP/quarantine features are required

---

# 45. Troubleshooting Decision Tree

```text
             File not detected?
                    │
                    ▼
             Is traffic HTTPS?
                    │
             ┌──────┴──────┐
             │             │
            YES            NO
             │             │
             ▼             ▼
      Deep inspection?   Check policy
             │
        ┌────┴────┐
        │         │
       NO        YES
        │         │
        ▼         ▼
   No payload   Continue
   visibility   AV analysis
                    │
                    ▼
             Inspection mode?
                    │
             ┌──────┴──────┐
             │             │
            Flow          Proxy
             │             │
             ▼             ▼
       File supported?  Buffer/scanunit?
             │             │
             ▼             ▼
       Oversize limit?   Sandbox/CDR?
             │             │
             ▼             ▼
       Check diagnostics
```

---

# 46. High-Value Diagnostic Commands

### FortiGuard / AV database

```cli
diagnose autoupdate versions
diagnose autoupdate status
```

### AV statistics

```cli
diagnose ips av stats show
diagnose ips av stats clear
```

### Scanunit

```cli
diagnose sys scanunit debug all
```

### General debug

```cli
diagnose debug enable
```

### FortiGuard connectivity/rating

```cli
diagnose debug rating
```

---

# 47. Lab Validation Checklist

Before deploying advanced AV inspection:

* [ ] Confirm FortiGuard AV subscription
* [ ] Confirm AV database is updated
* [ ] Confirm FortiSandbox connectivity
* [ ] Confirm SSL Deep Inspection CA deployment
* [ ] Confirm firewall policy uses correct AV profile
* [ ] Confirm protocol options are attached
* [ ] Test normal files
* [ ] Test oversized files
* [ ] Test compressed archives
* [ ] Test nested archives
* [ ] Test corrupted archives
* [ ] Test password-protected archives
* [ ] Test Office documents
* [ ] Test PDF
* [ ] Test executable files
* [ ] Test CDR
* [ ] Test CIFS/SMB
* [ ] Test SMTP attachments
* [ ] Test Sandbox inline mode
* [ ] Test Sandbox timeout behavior
* [ ] Test Sandbox connectivity failure
* [ ] Monitor memory/CPU
* [ ] Check conserve-mode events

---

# 48. Performance Engineering Checklist

Before enabling every AV feature globally:

```text
                    ┌──────────────┐
                    │ AV Features  │
                    └──────┬───────┘
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
        CDR             Sandbox            DLP
          │                │                │
          └────────────────┼────────────────┘
                           ▼
                    CPU / Memory
                           │
                           ▼
                    Throughput
```

Measure:

* CPU utilization
* Memory utilization
* Concurrent sessions
* File concurrency
* Average file size
* Large-file frequency
* Sandbox submission rate
* Sandbox latency
* Conserve-mode events

### Golden rule

> **More inspection ≠ automatically more security.**

Poorly sized inspection policies can create:

```text
More scanning
     ↓
More memory pressure
     ↓
Conserve mode
     ↓
Reduced performance
     ↓
Potential availability problems
```

---

# 49. Recommended Security Architecture

For a high-security enterprise:

```text
                    INTERNET
                       │
                       ▼
                ┌──────────────┐
                │  FortiGate   │
                └──────┬───────┘
                       │
             ┌─────────┼─────────┐
             ▼         ▼         ▼
         SSL Deep    IPS/App    Web/DNS
         Inspection  Control    Filtering
             │
             ▼
             AV
             │
       ┌─────┼─────────┐
       ▼     ▼         ▼
      ML    CDR    Threat Feeds
       │     │         │
       └─────┼─────────┘
             ▼
       FortiSandbox
             │
             ▼
        Final Verdict
             │
             ▼
          CLIENT
```

---

# 50. NSE Exam Memory Map 🧠

```text
FLOW
│
├── Fast
├── Low resource
├── On-the-fly
├── No full-file buffering
└── Limited advanced processing

PROXY
│
├── Buffer
├── Deep file inspection
├── CDR
├── DLP
├── Quarantine
└── Inline Sandbox

STREAM-BASED
│
├── On-the-fly
├── Archive-aware
├── Lower memory
└── Unsupported cases → buffering/scanunit

SCANUNIT
│
├── Heavy processing
├── Large/complex files
├── Buffering
└── Advanced inspection

WAD
│
├── Fast/in-process processing
├── Web/proxy traffic
└── Lightweight inspection

CDR
│
├── Sanitize active content
├── Reconstruct files
└── Proxy-oriented

SANDBOX
│
├── Post-transfer
│     └── Client may receive file first
│
└── Inline
      └── Hold → analyze → release/block

THREAT FEEDS
│
├── FortiGuard
├── EMS
└── External malware hashes
```

---

# ⚡ 60-Second Interview / Exam Summary

| Question                    | Answer                                                                  |
| --------------------------- | ----------------------------------------------------------------------- |
| Flow AV?                    | Fast, on-the-fly, low resource                                          |
| Proxy AV?                   | Buffering + deeper file processing                                      |
| WAD?                        | Lightweight/in-process traffic processing                               |
| Scanunit?                   | Heavy file scanning/processing                                          |
| Stream-based scanning?      | Inspect stream without buffering entire file                            |
| Legacy scanning?            | Buffer-oriented processing                                              |
| Oversize limit?             | Controls oversized file handling                                        |
| Uncompressed oversize?      | Protects against expanded archive size                                  |
| CDR?                        | Removes potentially dangerous active content and reconstructs file      |
| ML AV?                      | Helps identify suspicious/malicious file characteristics                |
| Virus Outbreak Prevention?  | Rapid threat intelligence/signature protection                          |
| External malware blocklist? | Hash-based malware blocking                                             |
| FortiSandbox post-transfer? | File may reach client before sandbox verdict                            |
| FortiSandbox inline?        | Hold file until sandbox decision                                        |
| Inline Sandbox failure?     | Depends on configured error/timeout action                              |
| HTTPS AV visibility?        | Requires appropriate SSL inspection                                     |
| CIFS encrypted traffic?     | May require proxy/decryption capabilities depending on SMB architecture |
| Large files?                | Carefully tune buffering/oversize/resource limits                       |
| Main performance risk?      | Buffering + concurrent large/complex files                              |

---

# 🔥 SheynShield Field Rules

> **Rule #1:** `Flow = speed`, `Proxy = depth`.

> **Rule #2:** If the security feature needs the **whole file**, expect **buffering/proxy-oriented processing**.

> **Rule #3:** `Oversize-limit` is not just a performance setting — it is also a **security/resource trade-off**.

> **Rule #4:** Large archives can become a memory problem after decompression. Always think about **compressed size vs uncompressed size**.

> **Rule #5:** CDR requires access to the actual file content and therefore is fundamentally different from simple stream signature matching.

> **Rule #6:** FortiSandbox **post-transfer** and **inline** have different security/latency characteristics.

> **Rule #7:** AV signatures alone are not enough for modern malware. Combine **AV + ML + FortiGuard intelligence + Sandbox + file filtering** where the platform/resources justify it.

> **Rule #8:** Don't enable every advanced inspection feature globally without measuring **CPU, memory, concurrency and file-size distribution**.

> **Rule #9:** For encrypted web traffic, remember:

```text
No Decryption
     ↓
No Payload Visibility
     ↓
Limited AV/File Inspection
```

> **Rule #10:** Always validate behavior against the **exact FortiOS version and FortiGate model** before turning a lab configuration into a production standard.

---

## 🔗 Related SheynShield Topics

* FortiGate SSL/SSH Deep Inspection
* FortiGate Protocol Options
* FortiGate File Filter
* FortiGate Web Filter
* FortiGate Application Control
* FortiGate IPS
* FortiSandbox
* FortiGuard Antivirus
* Security Fabric
* FortiClient EMS Threat Feeds
* Conserve Mode & FortiGate Memory Troubleshooting

# 🛡️ FortiGate Antivirus (AV) & File Inspection Checklist

> **SheynShield | FortiOS Security Engineering**
>
> **FortiGate Antivirus, File Inspection, Flow vs Proxy, Stream Scanning, Scanunit, CDR, FortiSandbox, Malware Feeds, CIFS/SMB & AV Troubleshooting**

---

## 📌 Table of Contents

* [🎯 Quick Mental Model](#-quick-mental-model)
* [1. AV Inspection Architecture](#1-av-inspection-architecture)
* [2. Flow-Based AV Checklist](#2-flow-based-av-checklist)
* [3. Proxy-Based AV Checklist](#3-proxy-based-av-checklist)
* [4. Flow vs Proxy Validation](#4-flow-vs-proxy-validation)
* [5. WAD vs Scanunit](#5-wad-vs-scanunit)
* [6. Stream-Based Scanning](#6-stream-based-scanning)
* [7. Stream-Based Limitations](#7-stream-based-limitations)
* [8. Oversized File Protection](#8-oversized-file-protection)
* [9. Archive & Uncompressed Oversize](#9-archive--uncompressed-oversize)
* [10. Legacy Scan Mode](#10-legacy-scan-mode)
* [11. TCP Window & File Inspection](#11-tcp-window--file-inspection)
* [12. AV Machine Learning](#12-av-machine-learning)
* [13. AV Database](#13-av-database)
* [14. FortiGuard AV Validation](#14-fortiguard-av-validation)
* [15. Content Disarm & Reconstruction](#15-content-disarm--reconstruction)
* [16. CDR + SMTP](#16-cdr--smtp)
* [17. Virus Outbreak Prevention](#17-virus-outbreak-prevention)
* [18. External Malware Blocklist](#18-external-malware-blocklist)
* [19. FortiClient EMS Malware Feed](#19-forticlient-ems-malware-feed)
* [20. CIFS/SMB Antivirus Inspection](#20-cifssmb-antivirus-inspection)
* [21. File Filter + Antivirus](#21-file-filter--antivirus)
* [22. HTTPS File Inspection](#22-https-file-inspection)
* [23. FortiSandbox Post-Transfer](#23-fortisandbox-post-transfer)
* [24. FortiSandbox Inline](#24-fortisandbox-inline)
* [25. FortiSandbox Timeout & Failure](#25-fortisandbox-timeout--failure)
* [26. FortiSandbox File Submission](#26-fortisandbox-file-submission)
* [27. AV Statistics & Diagnostics](#27-av-statistics--diagnostics)
* [28. Complete Security Policy Validation](#28-complete-security-policy-validation)
* [29. Troubleshooting Decision Tree](#29-troubleshooting-decision-tree)
* [30. Performance Engineering](#30-performance-engineering)
* [31. Production Deployment](#31-production-deployment)
* [32. Lab Validation](#32-lab-validation)
* [33. NSE Exam Memory Map](#33-nse-exam-memory-map)
* [34. 60-Second Interview Summary](#34-60-second-interview-summary)
* [🔥 SheynShield Field Rules](#-sheynshield-field-rules)
* [🔗 Related SheynShield Topics](#-related-sheynshield-topics)

---

# 🎯 Quick Mental Model

FortiGate file inspection depends on multiple variables:

* [ ] FortiOS version identified
* [ ] FortiGate model identified
* [ ] Inspection mode identified
* [ ] AV scan mode identified
* [ ] Protocol identified
* [ ] File size evaluated
* [ ] Archive/compression type evaluated
* [ ] Required security profiles identified
* [ ] FortiSandbox requirement identified
* [ ] Available CPU/memory evaluated

### Core model

```text
                    Client
                       │
                       ▼
                ┌─────────────┐
                │  FortiGate  │
                └──────┬──────┘
                       │
                 Firewall Policy
                       │
                       ▼
               SSL/SSH Inspection
                       │
                       ▼
                 Protocol Options
                       │
                       ▼
                    AV Engine
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
        Flow         Proxy       Scanunit
          │            │            │
          └────────────┼────────────┘
                       ▼
              CDR / Threat Feed
                       │
                       ▼
                 FortiSandbox
                       │
                       ▼
                  Final Action
```

---

# 1. AV Inspection Architecture

## Pre-Deployment Checklist

* [ ] Confirm FortiOS version
* [ ] Confirm FortiGate hardware model
* [ ] Confirm FortiGuard AV entitlement
* [ ] Confirm AV database status
* [ ] Identify inspected protocols
* [ ] Determine whether traffic is encrypted
* [ ] Determine whether Deep Inspection is required
* [ ] Determine whether CDR is required
* [ ] Determine whether FortiSandbox is required
* [ ] Estimate average file size
* [ ] Estimate maximum file size
* [ ] Estimate concurrent file transfers
* [ ] Review available memory
* [ ] Review expected throughput

---

# 2. Flow-Based AV Checklist

### Characteristics

```text
File Stream
     │
     ▼
AV Engine
     │
     ├── Inspect
     ├── Match
     └── Forward / Block
```

* [ ] Understand that flow inspection is stream-oriented
* [ ] Confirm full-file buffering is not required
* [ ] Confirm low latency is a priority
* [ ] Confirm high throughput is required
* [ ] Validate supported file types
* [ ] Validate supported archive types
* [ ] Validate oversized-file behavior
* [ ] Validate interaction with other security profiles
* [ ] Test large-file transfers
* [ ] Monitor CPU usage
* [ ] Monitor memory usage

### Flow Mental Model

```text
Receive chunk
     ↓
Inspect
     ↓
Forward / Drop
     ↓
Receive next chunk
```

### Best Fit

* [ ] High-throughput environments
* [ ] Low-latency environments
* [ ] Basic/on-the-fly file inspection
* [ ] Environments where full-file reconstruction is unnecessary

---

# 3. Proxy-Based AV Checklist

Proxy processing is appropriate when FortiGate needs more complete access to the file.

* [ ] Confirm proxy inspection is enabled where required
* [ ] Confirm full-file buffering is acceptable
* [ ] Validate memory requirements
* [ ] Validate archive extraction
* [ ] Validate CDR requirements
* [ ] Validate DLP requirements
* [ ] Validate quarantine requirements
* [ ] Validate Inline FortiSandbox requirements
* [ ] Test complete file delivery
* [ ] Test large files
* [ ] Test nested archives

### Proxy Mental Model

```text
Client
  │
  ▼
FortiGate
  │
  ▼
Buffer / Reassemble
  │
  ▼
Scanunit
  │
  ├── AV
  ├── CDR
  ├── DLP
  ├── Quarantine
  └── Sandbox
  │
  ▼
Release / Block
```

---

# 4. Flow vs Proxy Validation

| Requirement                |    Flow    | Proxy |
| -------------------------- | :--------: | :---: |
| Low latency                |     [x]    |  [ ]  |
| High throughput            |     [x]    |  [ ]  |
| Full-file buffering        |     [ ]    |  [x]  |
| Deep file processing       |   Limited  |  [x]  |
| CDR                        | Limited/No |  [x]  |
| Quarantine                 |   Limited  |  [x]  |
| Inline Sandbox             | Limited/No |  [x]  |
| Lower memory consumption   |     [x]    |  [ ]  |
| Complex archive processing |   Limited  |  [x]  |

### Decision Checklist

* [ ] Need speed → evaluate **Flow**
* [ ] Need deep file processing → evaluate **Proxy**
* [ ] Need CDR → evaluate **Proxy**
* [ ] Need Inline Sandbox → evaluate **Proxy-oriented architecture**
* [ ] Need full-file reconstruction → evaluate **Proxy**
* [ ] Need low resource consumption → evaluate **Flow**

> **Rule:** `Flow = speed`
> **Rule:** `Proxy = depth`

---

# 5. WAD vs Scanunit

## WAD

**WAD = Web Application Daemon**

* [ ] Understand WAD's role in proxy/web processing
* [ ] Use WAD mental model for lightweight/in-process processing
* [ ] Consider WAD when low-latency processing is sufficient

```text
Traffic
   ↓
WAD
   ↓
Lightweight inspection
   ↓
Forward / Drop
```

## Scanunit

* [ ] Understand Scanunit as heavier file-processing architecture
* [ ] Expect buffering for relevant processing paths
* [ ] Consider archive extraction requirements
* [ ] Consider memory consumption
* [ ] Consider concurrent file-processing load

```text
File
 ↓
Buffer
 ↓
Scanunit
 ├── AV
 ├── Decompression
 ├── Advanced inspection
 └── File analysis
```

---

# 6. Stream-Based Scanning

Stream-based inspection attempts to inspect content while it is being transferred.

### Validation Checklist

* [ ] Confirm protocol supports stream-based scanning
* [ ] Confirm file type is supported
* [ ] Confirm archive type is supported
* [ ] Confirm required security features are compatible
* [ ] Validate behavior with large files
* [ ] Validate behavior with compressed files
* [ ] Validate behavior with nested archives
* [ ] Test fallback behavior

### Concept

```text
File Stream
     │
     ▼
 AV Engine
     │
     ├── Malicious → DROP
     │
     └── Clean → FORWARD
```

### Common Archive Formats

Depending on FortiOS release:

* [ ] ZIP
* [ ] GZIP
* [ ] BZIP
* [ ] BZIP2
* [ ] TAR
* [ ] ISO 9660

> Always validate the exact supported formats against the target FortiOS release.

---

# 7. Stream-Based Limitations

Verify whether the following features affect stream-based processing:

* [ ] DLP
* [ ] Quarantine
* [ ] FortiGuard Outbreak Prevention
* [ ] External Block List
* [ ] CDR
* [ ] Protocol-specific limitations
* [ ] Archive limitations
* [ ] File-size limitations

### Possible Fallback

```text
Stream Inspection
       │
       ▼
Unsupported Case
       │
       ▼
Buffer
       │
       ▼
Scanunit
```

---

# 8. Oversized File Protection

FortiGate must balance file inspection depth against available resources.

### Checklist

* [ ] Define acceptable maximum file size
* [ ] Review `oversize-limit`
* [ ] Verify unit for the target release
* [ ] Verify supported range
* [ ] Define action for oversized content
* [ ] Test normal-size files
* [ ] Test oversized files
* [ ] Monitor memory during testing

### Example

```cli
config firewall profile-protocol-options
    edit "proto-test"
        config ftp
            set oversize-limit 10000
        end
    next
end
```

---

# 9. Archive & Uncompressed Oversize

Do **not** confuse:

```text
Compressed Size
        vs
Uncompressed Size
```

Example:

```text
ZIP = 100 MB
       │
       ▼
    Extract
       │
       ▼
Content = 2 GB
```

### Security Checklist

* [ ] Configure compressed-file limits where appropriate
* [ ] Configure uncompressed limits where supported
* [ ] Test nested archives
* [ ] Test archive bombs
* [ ] Test corrupted archives
* [ ] Test password-protected archives
* [ ] Monitor memory consumption

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

> **Security principle:** archive size alone does not represent the actual resource cost after decompression.

---

# 10. Legacy Scan Mode

Use legacy/buffer-oriented processing only when justified by the target FortiOS release and requirements.

### Checklist

* [ ] Confirm legacy mode is supported
* [ ] Identify why legacy processing is required
* [ ] Test compatibility
* [ ] Test memory consumption
* [ ] Test throughput
* [ ] Test large files
* [ ] Compare against flow/proxy processing
* [ ] Document operational reason

### Trade-Off

| Benefit                 | Cost                      |
| ----------------------- | ------------------------- |
| More complete buffering | Higher memory             |
| Compatibility           | Higher latency            |
| Troubleshooting value   | Lower throughput          |
| Advanced processing     | More resource consumption |

---

# 11. TCP Window & File Inspection

Large TCP windows can influence how much data is in flight during file transfers.

### Checklist

* [ ] Review TCP window behavior
* [ ] Review concurrent file transfers
* [ ] Monitor memory
* [ ] Monitor packet loss
* [ ] Monitor retransmissions
* [ ] Monitor conserve-mode events
* [ ] Avoid blindly increasing TCP window values
* [ ] Test under realistic concurrency

### Concept

```text
TCP Window
     ↓
More data in flight
     ↓
More inspection workload
     ↓
Potential resource pressure
```

---

# 12. AV Machine Learning

Machine-learning-based AV detection can complement traditional signature detection.

### Checklist

* [ ] Confirm ML detection support
* [ ] Confirm FortiGuard entitlement
* [ ] Confirm required packages/services
* [ ] Verify configuration
* [ ] Test known malware
* [ ] Test suspicious/unknown samples in a controlled lab
* [ ] Monitor CPU/memory impact
* [ ] Document expected behavior

Example:

```cli
config antivirus settings
    set machine-learning-detection enable
end
```

### Mental Model

```text
File
 ↓
Traditional AV
 ↓
ML Analysis
 ↓
File Characteristics
 ↓
Risk Classification
```

---

# 13. AV Database

### Database Checklist

* [ ] Determine database mode supported by platform
* [ ] Verify current AV database
* [ ] Verify update status
* [ ] Verify FortiGuard connectivity
* [ ] Evaluate resource impact
* [ ] Document selected database mode

Example:

```cli
config antivirus settings
    set use-extreme-db enable
end
```

> Verify support and behavior for the exact FortiOS release and hardware model.

---

# 14. FortiGuard AV Validation

### Commands

```cli
diagnose autoupdate versions
```

```cli
diagnose autoupdate status
```

### Checklist

* [ ] AV database is current
* [ ] FortiGuard connectivity is healthy
* [ ] Updates are occurring
* [ ] Subscription is valid
* [ ] Update timestamp is recent
* [ ] No FortiGuard communication errors exist

---

# 15. Content Disarm & Reconstruction

CDR sanitizes supported files by removing potentially dangerous active content and reconstructing the document.

### CDR Checklist

* [ ] Confirm CDR is supported
* [ ] Confirm proxy-oriented processing
* [ ] Confirm target file types
* [ ] Confirm CDR policy
* [ ] Confirm `detect-only` behavior
* [ ] Define CDR error action
* [ ] Test Office documents
* [ ] Test embedded objects
* [ ] Test macros/active content
* [ ] Verify reconstructed output
* [ ] Verify user experience

### Workflow

```text
Office Document
      │
      ▼
     CDR
      │
      ├── Inspect
      ├── Detect active content
      ├── Remove risky components
      └── Reconstruct
             │
             ▼
       Sanitized File
```

---

# 16. CDR + SMTP

SMTP attachment inspection requires careful validation because email protocols and message handling differ from ordinary web downloads.

### Checklist

* [ ] Confirm SMTP inspection architecture
* [ ] Confirm attachment extraction
* [ ] Confirm AV scanning
* [ ] Confirm CDR processing
* [ ] Test Office attachments
* [ ] Test PDF attachments
* [ ] Test archives
* [ ] Test malformed messages
* [ ] Test mail-client compatibility
* [ ] Validate splice/scan behavior where applicable

---

# 17. Virus Outbreak Prevention

FortiGuard outbreak intelligence provides protection beyond the local signature database.

### Checklist

* [ ] Confirm FortiGuard subscription
* [ ] Confirm FortiGuard connectivity
* [ ] Confirm outbreak-prevention functionality
* [ ] Validate policy interaction
* [ ] Test logging
* [ ] Test blocked-file behavior
* [ ] Review update status

### Mental Model

```text
Local AV
   +
FortiGuard Intelligence
   +
Threat Intelligence
   ↓
Additional Malware Protection
```

### Diagnostic

```cli
diagnose debug rating
```

---

# 18. External Malware Blocklist

External malware feeds can provide hashes such as:

* [ ] MD5
* [ ] SHA-1
* [ ] SHA-256

### Architecture

```text
External Threat Feed
        │
        ▼
    Malware Hash
        │
        ▼
      FortiGate
        │
        ├── Match → BLOCK
        └── No Match → Continue
```

### Checklist

* [ ] Validate feed source
* [ ] Validate hash format
* [ ] Validate update frequency
* [ ] Validate feed size
* [ ] Test known malicious hash
* [ ] Test non-matching hash
* [ ] Monitor CPU
* [ ] Monitor memory
* [ ] Monitor lookup latency

Example:

```cli
config antivirus profile
    edit "av-test"
        config http
            set external-blocklist "malw-test"
        end
    next
end
```

---

# 19. FortiClient EMS Malware Feed

FortiClient EMS can provide malware intelligence to FortiGate.

### EMS Checklist

* [ ] Confirm EMS connectivity
* [ ] Confirm HTTPS connectivity
* [ ] Configure EMS server
* [ ] Enable malware-hash synchronization
* [ ] Verify synchronization
* [ ] Enable EMS threat-feed usage in AV profile
* [ ] Test known malware hash
* [ ] Verify logs

Example:

```cli
config system endpoint-control fctems
    edit "ems-test"
        set server 192.168.20.200
        set https-port 443
        set pull-malware-hash enable
    next
end
```

AV profile:

```cli
config antivirus profile
    edit "av-test"
        set ems-threat-feed enable
    next
end
```

> CLI structure and available options vary by FortiOS release.

---

# 20. CIFS/SMB Antivirus Inspection

### Checklist

* [ ] Identify SMB/CIFS traffic
* [ ] Determine whether traffic is encrypted
* [ ] Determine whether proxy inspection is required
* [ ] Validate supported archive formats
* [ ] Validate oversized-file behavior
* [ ] Validate nested archives
* [ ] Validate file-type detection
* [ ] Validate authentication/decryption requirements
* [ ] Test normal SMB downloads
* [ ] Test large SMB transfers

### File Magic Number

```text
File
 ↓
Initial Bytes
 ↓
Magic Number
 ↓
File Type Identification
```

### Important

Do not assume FortiGate can inspect every encrypted SMB implementation.

---

# 21. File Filter + Antivirus

File filtering can complement AV by enforcing file-type policies.

### Checklist

* [ ] Define prohibited file types
* [ ] Define allowed file types
* [ ] Select required protocols
* [ ] Select correct action
* [ ] Attach file-filter profile
* [ ] Attach AV profile
* [ ] Test allowed files
* [ ] Test blocked files
* [ ] Verify logging

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

---

# 22. HTTPS File Inspection

HTTPS encryption can prevent FortiGate from seeing the actual payload unless appropriate SSL inspection is configured.

### Checklist

* [ ] Confirm traffic is HTTPS
* [ ] Confirm SSL/SSH Inspection profile
* [ ] Determine whether Deep Inspection is required
* [ ] Deploy trusted CA to clients where required
* [ ] Verify certificate validation
* [ ] Test HTTPS download
* [ ] Verify AV logs
* [ ] Verify file inspection
* [ ] Verify application compatibility

### Critical Mental Model

```text
HTTPS
  ↓
Encrypted Payload
  ↓
No Decryption
  ↓
No Payload Visibility
  ↓
Limited File Inspection
```

> **HTTPS + AV without appropriate decryption does not provide full file visibility.**

---

# 23. FortiSandbox Post-Transfer

Post-transfer scanning prioritizes user experience and lower delivery latency.

### Checklist

* [ ] Confirm FortiSandbox connectivity
* [ ] Enable appropriate AV/Sandbox integration
* [ ] Define submission criteria
* [ ] Test suspicious files
* [ ] Verify submission
* [ ] Verify verdict
* [ ] Verify logging
* [ ] Understand that file may reach endpoint before verdict

### Mental Model

```text
Client
  │
  ▼
FortiGate
  │
  ├──────────────► FortiSandbox
  │                     │
  ▼                     ▼
Client receives      Analysis
file                    │
                        ▼
                     Verdict
```

> **Post-transfer = lower latency, but the endpoint may receive the file before the Sandbox verdict.**

---

# 24. FortiSandbox Inline

Inline Sandbox scanning holds the file until the configured Sandbox decision is available.

### Checklist

* [ ] Confirm FortiSandbox availability
* [ ] Confirm inline scanning support
* [ ] Enable inline scanning
* [ ] Configure Sandbox server
* [ ] Configure error action
* [ ] Configure timeout action
* [ ] Configure maximum upload size
* [ ] Test clean file
* [ ] Test malicious/suspicious file
* [ ] Test Sandbox failure
* [ ] Test Sandbox timeout
* [ ] Monitor latency
* [ ] Monitor resource usage

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

### Inline Flow

```text
File
 ↓
FortiGate
 ↓
HOLD
 ↓
FortiSandbox
 ↓
Analysis
 ├── Clean → RELEASE
 └── Malicious → BLOCK
```

---

# 25. FortiSandbox Timeout & Failure

Inline protection introduces dependency on Sandbox availability.

### Checklist

* [ ] Define Sandbox error action
* [ ] Define Sandbox timeout action
* [ ] Decide fail-open vs fail-closed
* [ ] Test Sandbox unavailable
* [ ] Test network failure
* [ ] Test timeout
* [ ] Test high Sandbox latency
* [ ] Verify user experience
* [ ] Verify security impact

### High-Security Model

```text
Sandbox Failure
      ↓
    BLOCK
```

### Security Trade-Off

| Strategy    | Security | Availability |
| ----------- | -------- | ------------ |
| Fail-closed | Higher   | Lower        |
| Fail-open   | Lower    | Higher       |

> Choose according to business requirements and risk tolerance.

---

# 26. FortiSandbox File Submission

### Checklist

* [ ] Define which files are submitted
* [ ] Define suspicious-file criteria
* [ ] Review supported formats
* [ ] Configure exclusions if required
* [ ] Configure maximum upload size
* [ ] Review Sandbox capacity
* [ ] Review Internet bandwidth
* [ ] Review FortiGate resources
* [ ] Monitor submission rate
* [ ] Monitor verdict latency

### Common Categories

* [ ] ZIP
* [ ] RAR
* [ ] 7z
* [ ] Office
* [ ] PDF
* [ ] EXE
* [ ] DLL
* [ ] Script files
* [ ] HTML/JS
* [ ] Archives

> Supported formats depend on the FortiSandbox/FortiOS release.

---

# 27. AV Statistics & Diagnostics

## AV Database

```cli
diagnose autoupdate versions
diagnose autoupdate status
```

* [ ] Confirm AV database version
* [ ] Confirm update timestamp
* [ ] Confirm FortiGuard status

## AV Statistics

```cli
diagnose ips av stats show
```

```cli
diagnose ips av stats clear
```

* [ ] Clear statistics before controlled testing
* [ ] Generate test traffic
* [ ] Re-check statistics
* [ ] Verify AV engine activity

## Scanunit

```cli
diagnose sys scanunit debug all
```

* [ ] Enable only when required
* [ ] Reproduce the issue
* [ ] Capture relevant output
* [ ] Disable unnecessary debugging

## General Debug

```cli
diagnose debug enable
```

> Avoid uncontrolled debugging on production systems.

## FortiGuard

```cli
diagnose debug rating
```

---

# 28. Complete Security Policy Validation

A complete file-inspection policy may include:

```text
Firewall Policy
      │
      ├── Source
      ├── Destination
      ├── Service
      │
      ├── SSL/SSH Inspection
      │
      ├── Protocol Options
      │
      ├── Antivirus
      │
      ├── File Filter
      │
      ├── Web Filter
      │
      ├── Application Control
      │
      ├── IPS
      │
      └── DLP
```

### Checklist

* [ ] Correct firewall policy matched
* [ ] Correct source selected
* [ ] Correct destination selected
* [ ] Correct service selected
* [ ] SSL inspection attached where required
* [ ] Protocol options attached
* [ ] AV profile attached
* [ ] File Filter attached where required
* [ ] IPS configured
* [ ] Application Control configured
* [ ] DLP configured where required
* [ ] Logging enabled
* [ ] Policy order verified

---

# 29. Troubleshooting Decision Tree

```text
                 File not detected
                        │
                        ▼
                 Is traffic HTTPS?
                   /          \
                 YES           NO
                  │             │
                  ▼             ▼
          Deep Inspection?   Check Policy
             /      \
           NO       YES
           │         │
           ▼         ▼
     No Payload   Continue
      Visibility     │
                     ▼
               Inspection Mode
                  /       \
               Flow       Proxy
                │           │
                ▼           ▼
          File Supported?  Buffer?
                │           │
                ▼           ▼
          Oversize Limit?  Scanunit?
                │           │
                └─────┬─────┘
                      ▼
                FortiSandbox?
                      │
                      ▼
                 Check Logs
```

### Troubleshooting Checklist

* [ ] Confirm policy match
* [ ] Confirm traffic direction
* [ ] Confirm protocol
* [ ] Confirm HTTPS decryption
* [ ] Confirm AV profile
* [ ] Confirm protocol options
* [ ] Confirm inspection mode
* [ ] Confirm file type
* [ ] Confirm archive type
* [ ] Confirm file size
* [ ] Confirm oversize limits
* [ ] Confirm AV database
* [ ] Confirm FortiGuard connectivity
* [ ] Confirm Sandbox connectivity
* [ ] Check AV statistics
* [ ] Check Scanunit behavior
* [ ] Check logs
* [ ] Check CPU
* [ ] Check memory
* [ ] Check conserve mode

---

# 30. Performance Engineering

Advanced file inspection can increase CPU and memory requirements.

### Measure

* [ ] CPU utilization
* [ ] Memory utilization
* [ ] Concurrent sessions
* [ ] Concurrent file transfers
* [ ] Average file size
* [ ] Maximum file size
* [ ] Archive frequency
* [ ] Nested archive frequency
* [ ] Sandbox submission rate
* [ ] Sandbox latency
* [ ] CDR workload
* [ ] DLP workload
* [ ] Conserve-mode events

### Risk Model

```text
More Inspection
      ↓
More Processing
      ↓
More CPU / Memory
      ↓
Resource Pressure
      ↓
Conserve Mode
      ↓
Performance / Availability Impact
```

### Performance Checklist

* [ ] Do not enable every advanced feature blindly
* [ ] Establish baseline performance
* [ ] Test under realistic concurrency
* [ ] Test large files
* [ ] Test compressed archives
* [ ] Test Sandbox load
* [ ] Test CDR workload
* [ ] Test DLP workload
* [ ] Monitor memory
* [ ] Monitor CPU
* [ ] Document capacity limits

---

# 31. Production Deployment

## Before Deployment

* [ ] Confirm exact FortiOS release
* [ ] Confirm hardware platform
* [ ] Confirm FortiGuard subscription
* [ ] Confirm AV database
* [ ] Confirm SSL inspection design
* [ ] Confirm protocol options
* [ ] Confirm AV profile
* [ ] Confirm File Filter requirements
* [ ] Confirm CDR requirements
* [ ] Confirm Sandbox requirements
* [ ] Confirm logging
* [ ] Confirm rollback plan

## Security Validation

* [ ] Test known malicious sample in a controlled environment
* [ ] Test clean file
* [ ] Test executable
* [ ] Test Office file
* [ ] Test PDF
* [ ] Test archive
* [ ] Test nested archive
* [ ] Test oversized file
* [ ] Test encrypted traffic
* [ ] Test Sandbox verdict
* [ ] Test Sandbox failure
* [ ] Test CDR
* [ ] Test SMB/CIFS
* [ ] Test SMTP

---

# 32. Lab Validation

### Basic AV

* [ ] Clean HTTP file
* [ ] Clean HTTPS file
* [ ] Known malware test sample
* [ ] Executable file
* [ ] PDF
* [ ] Office document

### Archive

* [ ] ZIP
* [ ] GZIP
* [ ] TAR
* [ ] Nested archive
* [ ] Corrupted archive
* [ ] Password-protected archive
* [ ] Archive bomb simulation

### Advanced Inspection

* [ ] CDR
* [ ] File Filter
* [ ] External malware feed
* [ ] EMS malware feed
* [ ] FortiSandbox post-transfer
* [ ] FortiSandbox inline
* [ ] Sandbox timeout
* [ ] Sandbox failure

### Protocols

* [ ] HTTP
* [ ] HTTPS
* [ ] FTP
* [ ] SMTP
* [ ] CIFS/SMB
* [ ] SSH/SFTP where supported

### Resource Validation

* [ ] CPU
* [ ] Memory
* [ ] Session count
* [ ] Concurrent transfers
* [ ] Sandbox latency
* [ ] Conserve mode

---

# 33. NSE Exam Memory Map

```text
FLOW
│
├── Fast
├── Low resource
├── On-the-fly
├── Stream-oriented
└── Limited deep processing

PROXY
│
├── Buffer
├── Deep file processing
├── CDR
├── DLP
├── Quarantine
└── Inline-oriented workflows

STREAM-BASED
│
├── On-the-fly
├── Archive-aware
├── Lower memory
└── Unsupported cases may require buffering

SCANUNIT
│
├── Heavy processing
├── File analysis
├── Decompression
└── Buffer-oriented processing

WAD
│
├── Web/proxy processing
├── In-process processing
└── Lightweight inspection

CDR
│
├── Detect active content
├── Remove risky content
└── Reconstruct file

SANDBOX
│
├── Post-transfer
│     └── File may reach endpoint first
│
└── Inline
      └── Hold → Analyze → Release/Block

THREAT INTELLIGENCE
│
├── FortiGuard
├── EMS
└── External Hash Feeds
```

---

# 34. 60-Second Interview Summary

| Question               | Answer                                                               |
| ---------------------- | -------------------------------------------------------------------- |
| Flow AV?               | Fast, stream-oriented, lower resource consumption                    |
| Proxy AV?              | Buffering and deeper file processing                                 |
| WAD?                   | Web/proxy-oriented processing component                              |
| Scanunit?              | Heavy file-processing/scanning component                             |
| Stream scanning?       | Inspect content while transferring                                   |
| Legacy mode?           | Buffer-oriented processing                                           |
| Oversize limit?        | Controls handling of oversized files                                 |
| Uncompressed oversize? | Controls expanded archive size                                       |
| CDR?                   | Sanitizes and reconstructs supported files                           |
| ML AV?                 | Adds ML-assisted malware detection                                   |
| FortiGuard?            | Provides threat/signature intelligence                               |
| External blocklist?    | Hash-based malware blocking                                          |
| EMS threat feed?       | Endpoint-derived malware intelligence                                |
| Sandbox post-transfer? | File may reach endpoint before verdict                               |
| Sandbox inline?        | Holds file for Sandbox decision                                      |
| Sandbox failure?       | Depends on configured error action                                   |
| HTTPS AV visibility?   | Requires appropriate SSL inspection                                  |
| SMB encrypted traffic? | Depends on protocol/decryption capabilities                          |
| Main resource risk?    | Large/complex files + concurrency + buffering                        |
| Main design rule?      | Match inspection depth to security requirement and platform capacity |

---

# 🔥 SheynShield Field Rules

> **Rule #1:** `Flow = Speed`
> `Proxy = Depth`

> **Rule #2:** If a feature requires the **complete file**, expect a buffering/proxy-oriented workflow.

> **Rule #3:** `oversize-limit` is both a **security control** and a **resource-management control**.

> **Rule #4:** Always distinguish:

```text
Compressed Size
       ≠
Uncompressed Size
```

> **Rule #5:** CDR is fundamentally different from simple stream signature matching because the file must be processed and reconstructed.

> **Rule #6:** FortiSandbox **Post-Transfer** and **Inline** provide different security/latency trade-offs.

> **Rule #7:** AV should be viewed as a defense-in-depth layer:

```text
AV
 +
ML
 +
FortiGuard
 +
Threat Feeds
 +
CDR
 +
FortiSandbox
```

> **Rule #8:** Never enable every advanced inspection feature globally without capacity testing.

> **Rule #9:** For encrypted web traffic:

```text
No Decryption
     ↓
No Payload Visibility
     ↓
Limited File Inspection
```

> **Rule #10:** Always validate behavior against the **exact FortiOS version + FortiGate model**.

---

# ⚡ Fast Troubleshooting Checklist

```text
[ ] 1. Correct firewall policy?
[ ] 2. Correct AV profile?
[ ] 3. Correct protocol options?
[ ] 4. HTTPS traffic decrypted?
[ ] 5. Correct inspection mode?
[ ] 6. File type supported?
[ ] 7. Archive type supported?
[ ] 8. File within size limit?
[ ] 9. Uncompressed size within limit?
[ ] 10. AV database current?
[ ] 11. FortiGuard reachable?
[ ] 12. Sandbox reachable?
[ ] 13. CDR configured correctly?
[ ] 14. File Filter affecting traffic?
[ ] 15. Application/security profile affecting traffic?
[ ] 16. AV statistics incrementing?
[ ] 17. Scanunit processing correctly?
[ ] 18. CPU normal?
[ ] 19. Memory normal?
[ ] 20. Conserve mode inactive?
```

---

# 🧠 One-Line Architecture

```text
Decrypt → Identify → Inspect → Scan → Sanitize → Sandbox → Verdict → Deliver
```

---

# 🔗 Related SheynShield Topics

* [ ] FortiGate SSL/SSH Deep Inspection
* [ ] FortiGate Protocol Options
* [ ] FortiGate File Filter
* [ ] FortiGate Web Filter
* [ ] FortiGate Application Control
* [ ] FortiGate IPS
* [ ] FortiSandbox
* [ ] FortiGuard Antivirus
* [ ] FortiClient EMS Threat Feeds
* [ ] Security Fabric
* [ ] FortiGate Conserve Mode
* [ ] FortiGate Memory Troubleshooting

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

## 🏷️ Topics

`fortigate` `fortios` `fortinet` `antivirus` `file-inspection` `fortisandbox` `cdr` `content-disarm-reconstruction` `av` `network-security` `cybersecurity` `nse4` `nse7` `security-engineering` `malware-detection` `threat-intelligence` `ssl-inspection` `cifs` `smb` `network-security-checklist`

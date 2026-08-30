# 🔐 FortiGate DLP Checklist — Data Leak Prevention

> **FortiOS 7.x | DLP | Data Types | Dictionaries | Sensors | DLP Profiles | Fingerprinting | Proxy Inspection | SSL Deep Inspection**
>
> **SheynShield | Engineering Secure Networks**

---

## 📋 Table of Contents

* [Quick Audit](#-quick-audit)
* [1. DLP Fundamentals](#1-dlp-fundamentals)
* [2. DLP Architecture](#2-dlp-architecture)
* [3. Proxy Inspection](#3-proxy-inspection)
* [4. SSL Deep Inspection](#4-ssl-deep-inspection)
* [5. Data Types](#5-data-types)
* [6. Dictionaries](#6-dictionaries)
* [7. Sensors](#7-sensors)
* [8. DLP Profiles](#8-dlp-profiles)
* [9. DLP Rules](#9-dlp-rules)
* [10. File Type Filtering](#10-file-type-filtering)
* [11. File Size Filtering](#11-file-size-filtering)
* [12. DLP Actions](#12-dlp-actions)
* [13. Fingerprinting](#13-fingerprinting)
* [14. Fingerprint Document Sources](#14-fingerprint-document-sources)
* [15. Fingerprint Matching](#15-fingerprint-matching)
* [16. Cloud Storage DLP](#16-cloud-storage-dlp)
* [17. Performance](#17-performance)
* [18. Troubleshooting](#18-troubleshooting)
* [19. Production Deployment](#19-production-deployment)
* [20. Practical Validation](#20-practical-validation)
* [21. NSE Exam Traps](#21-nse-exam-traps)
* [22. 60-Second Revision](#22-60-second-revision)
* [23. CLI Quick Reference](#23-cli-quick-reference)

---

# ⚡ Quick Audit

Use this section for a **60-second DLP deployment review**.

### Core Configuration

* [ ] DLP feature is enabled
* [ ] Correct DLP profile exists
* [ ] DLP profile uses the required proxy feature set
* [ ] Firewall policy uses proxy-based inspection where required
* [ ] Correct DLP sensor is configured
* [ ] Required dictionaries are configured
* [ ] Required data types are configured
* [ ] DLP rules contain the correct protocol
* [ ] Required file types are selected
* [ ] File-size limits are appropriate
* [ ] Correct DLP action is configured
* [ ] Required logging is enabled
* [ ] DLP profile is attached to the correct firewall policy

### HTTPS

* [ ] SSL inspection is configured where required
* [ ] Deep Inspection CA is trusted by clients
* [ ] HTTPS traffic is actually decrypted
* [ ] DLP can inspect the decrypted content
* [ ] SSL inspection exclusions are reviewed

### Fingerprinting

* [ ] Internal storage requirement verified
* [ ] Document source configured
* [ ] Repository connectivity verified
* [ ] Fingerprint database populated
* [ ] Fingerprint daemon operational
* [ ] Match percentage configured
* [ ] Fingerprint action tested

### Validation

* [ ] HTTP upload tested
* [ ] HTTP download tested
* [ ] HTTPS upload tested
* [ ] HTTPS download tested
* [ ] Keyword detection tested
* [ ] Regex detection tested
* [ ] Dictionary matching tested
* [ ] Sensor threshold tested
* [ ] File-type filtering tested
* [ ] File-size filtering tested
* [ ] Fingerprinting tested
* [ ] Cloud-storage behavior tested
* [ ] False positives reviewed
* [ ] CPU/memory impact measured

---

# 1. DLP Fundamentals

## What is DLP?

**DLP — Data Leak Prevention** detects and controls sensitive information moving through network traffic.

### Typical objectives

* [ ] Detect sensitive information
* [ ] Log sensitive-data violations
* [ ] Allow legitimate traffic
* [ ] Block sensitive-data transfers
* [ ] Quarantine violating activity
* [ ] Restrict sensitive files
* [ ] Protect confidential documents
* [ ] Reduce accidental data leakage
* [ ] Support compliance requirements

### Typical sensitive information

* [ ] Credentials
* [ ] Personal information
* [ ] Credit-card data
* [ ] Social Security Numbers
* [ ] Confidential documents
* [ ] Internal keywords
* [ ] Source code
* [ ] Sensitive files

### Basic flow

```text
Client
   |
   | Upload / Download
   v
FortiGate
   |
   v
DLP Inspection
   |
   +--> Detect
   +--> Log
   +--> Allow
   +--> Block
   +--> Quarantine
   |
   v
Destination
```

---

# 2. DLP Architecture

## Core DLP hierarchy

```text
Data Type
    |
    v
Dictionary
    |
    v
Sensor
    |
    v
DLP Profile
    |
    v
Firewall Policy
```

### Verify each layer

* [ ] Data Type defines what should be detected
* [ ] Dictionary groups related detection entries
* [ ] Sensor defines matching logic
* [ ] DLP Profile defines inspection/action behavior
* [ ] Firewall Policy determines where DLP is applied

### Mental model

```text
WHAT?
  |
  +--> Data Type

WHAT GROUP?
  |
  +--> Dictionary

HOW TO EVALUATE?
  |
  +--> Sensor

WHAT SHOULD HAPPEN?
  |
  +--> DLP Profile

WHERE?
  |
  +--> Firewall Policy
```

---

# 3. Proxy Inspection

> ⭐ **NSE KEY CONCEPT**

Many advanced DLP capabilities require **proxy-based inspection**.

### Verify

* [ ] Firewall policy uses the required inspection mode
* [ ] DLP profile is configured for proxy processing
* [ ] Traffic is actually traversing the intended policy
* [ ] Required security profiles are attached

### Concept

```text
LAN
 |
 v
Firewall Policy
 |
 +--> Proxy Inspection
 |
 +--> DLP
 |
 v
Internet
```

### DLP rule of thumb

```text
Advanced DLP
     |
     v
Proxy Processing
```

> ⚠️ Do not assume a flow-based policy provides every DLP capability available in proxy mode.

---

# 4. SSL Deep Inspection

DLP cannot inspect sensitive information hidden inside encrypted HTTPS payloads unless the traffic is available for inspection.

### HTTPS flow

```text
Client
   |
   | HTTPS
   v
FortiGate
   |
   v
SSL Deep Inspection
   |
   v
Decrypted Content
   |
   v
DLP
   |
   v
Security Decision
```

### HTTPS DLP checklist

* [ ] SSL Deep Inspection is configured where required
* [ ] Client trusts the inspection CA
* [ ] HTTPS traffic is decrypted
* [ ] DLP receives inspectable content
* [ ] Necessary SSL exclusions are reviewed
* [ ] Applications affected by certificate pinning are tested

### Critical concept

```text
Encrypted Payload
      |
      X
DLP cannot inspect hidden content

Decrypted Payload
      |
      v
DLP Inspection
      |
      v
Detection
```

---

# 5. Data Types

Data Types define the content that DLP should detect.

### Common detection types

* [ ] Keyword
* [ ] Regex
* [ ] Hex
* [ ] Credit Card
* [ ] Social Security Number
* [ ] Custom patterns

---

## Keyword

Use keywords for exact words or phrases.

Example:

```text
CONFIDENTIAL
INTERNAL
SECRET
```

Checklist:

* [ ] Keyword is correctly defined
* [ ] Case/normalization behavior is understood
* [ ] False-positive keywords are reviewed
* [ ] Keyword is added to the appropriate dictionary

---

## Regex

Use regular expressions for structured patterns.

Example:

```regex
\b\d{3}-\d{2}-\d{4}\b
```

Checklist:

* [ ] Regex syntax is valid
* [ ] Expected input format is tested
* [ ] False positives are tested
* [ ] Multiple representations are considered

---

## Hex

Hex patterns can be used when detection requires binary/hexadecimal matching.

* [ ] Hex pattern defined
* [ ] Expected binary representation verified
* [ ] Test data created
* [ ] False positives validated

---

## Credit Card

* [ ] Built-in credit-card detection evaluated
* [ ] Custom pattern required?
* [ ] Verification logic considered
* [ ] Test card data used safely
* [ ] False positives validated

---

## SSN

* [ ] Built-in detection evaluated
* [ ] Custom regex required?
* [ ] Verification logic considered
* [ ] Test values validated
* [ ] False positives reviewed

---

# 6. Dictionaries

A **DLP Dictionary** groups related data-detection entries.

### Example

```text
Dictionary: Confidential-Data
 |
 +--> CONFIDENTIAL
 +--> INTERNAL
 +--> SECRET
 +--> Custom Pattern
```

### Checklist

* [ ] Dictionary created
* [ ] Correct data types added
* [ ] Keywords reviewed
* [ ] Regex patterns reviewed
* [ ] Logical relationship defined
* [ ] False positives tested

---

## ANY Logic

```text
Entry A
   OR
Entry B
   OR
Entry C
```

* [ ] Understand that one matching entry can satisfy the logical relationship
* [ ] Use ANY when independent indicators should trigger detection

---

## ALL Logic

```text
Entry A
   AND
Entry B
   AND
Entry C
```

* [ ] Confirm all required entries must match
* [ ] Use ALL when multiple indicators must coexist

---

# 7. Sensors

A **DLP Sensor** determines how dictionaries and entries are evaluated.

### Architecture

```text
Data Type
   |
   v
Dictionary
   |
   v
Sensor
   |
   v
DLP Profile
```

### Checklist

* [ ] Sensor created
* [ ] Correct dictionary selected
* [ ] Match type configured
* [ ] Entry status enabled
* [ ] Count threshold configured
* [ ] Sensor attached to the intended DLP configuration
* [ ] Sensor tested with known data

### Example

```cli
config dlp sensor
    edit "sen-test"
        set match-type match-any

        config entries
            edit 1
                set dictionary "dlp-dic-test"
                set count 1
                set status enable
            next
        end
    next
end
```

---

# 8. DLP Profiles

The DLP Profile determines the inspection and enforcement behavior.

### Checklist

* [ ] DLP profile created
* [ ] Correct feature set selected
* [ ] Required rules configured
* [ ] Protocol selected
* [ ] File type selected
* [ ] File size configured
* [ ] Action configured
* [ ] Logging configured
* [ ] Profile attached to firewall policy

### Example

```cli
config dlp profile
    edit "dlp-prof-test"

        set feature-set proxy

    next
end
```

### Remember

```text
Data Type
     ↓
Dictionary
     ↓
Sensor
     ↓
DLP Profile
     ↓
Firewall Policy
```

---

# 9. DLP Rules

A DLP rule can define:

```text
Protocol
Filter
File Type
File Size
Action
```

### Checklist

* [ ] Protocol matches actual traffic
* [ ] Filter type is correct
* [ ] File type matches the requirement
* [ ] File-size threshold is appropriate
* [ ] Action is appropriate
* [ ] Logging is enabled where required

Example:

```text
DLP Rule
 |
 +--> Protocol
 +--> Filter
 +--> File Type
 +--> File Size
 +--> Action
```

---

# 10. File Type Filtering

Use file-type classification when the requirement is actually based on file type.

### Examples

```text
Documents
Archives
Images
Executables
Other
```

### Checklist

* [ ] Required file type identified
* [ ] FortiOS file-type classification verified
* [ ] Numeric IDs verified against target FortiOS release
* [ ] Test file created
* [ ] Upload tested
* [ ] Download tested

> ⚠️ Avoid blindly memorizing numeric file-type IDs across FortiOS releases.

---

# 11. File Size Filtering

File-size restrictions can reduce unnecessary inspection and resource consumption.

Example:

```cli
set file-size 500
```

### Checklist

* [ ] Maximum/configured size understood
* [ ] Units verified
* [ ] Boundary condition tested
* [ ] Files below threshold tested
* [ ] Files above threshold tested
* [ ] Performance impact evaluated

---

# 12. DLP Actions

Common actions include:

| Action         | Purpose                                               |
| -------------- | ----------------------------------------------------- |
| **Allow**      | Permit activity                                       |
| **Log Only**   | Detect and log without blocking                       |
| **Block**      | Deny matching activity                                |
| **Quarantine** | Apply stronger restriction according to configuration |

### Validate action behavior

* [ ] Allow tested
* [ ] Log Only tested
* [ ] Block tested
* [ ] Quarantine tested where required
* [ ] Logs confirm expected action

### Remember

```text
LOG ONLY
  |
  +--> Detect + Log
  +--> Traffic continues
```

```text
BLOCK
  |
  +--> Matching activity denied
```

```text
QUARANTINE
  |
  +--> Stronger restriction behavior
```

> Exact behavior can vary with FortiOS release and configuration.

---

# 13. Fingerprinting

DLP Fingerprinting is designed to detect known documents/files by comparing file fingerprints.

### Concept

```text
Sensitive Document
       |
       v
Fingerprint Generation
       |
       v
Fingerprint Database
       |
       v
Future Network File
       |
       v
Fingerprint Comparison
       |
   +---+---+
   |       |
 Match   No Match
   |       |
   v       v
Action   Continue
```

### Fingerprinting checklist

* [ ] Sensitive documents identified
* [ ] Document repository prepared
* [ ] Document source configured
* [ ] Fingerprint database populated
* [ ] Matching tested
* [ ] Match percentage reviewed
* [ ] Action configured
* [ ] Logs validated

---

# 14. Fingerprint Document Sources

A document source can provide files used to build the fingerprint database.

Example architecture:

```text
SMB Repository
     |
     v
FortiGate
     |
     v
Document Source
     |
     v
Fingerprint Database
```

### Important

> Document fingerprinting requires a FortiGate with the necessary internal storage capability.

### Configuration checklist

* [ ] Server type selected
* [ ] Server address configured
* [ ] Repository path configured
* [ ] File pattern configured
* [ ] Scan subdirectories configured where required
* [ ] Schedule configured
* [ ] Deleted-file behavior reviewed
* [ ] Modified-file behavior reviewed
* [ ] Sensitivity configured
* [ ] Service account created
* [ ] Credentials protected

### Safe documentation

```text
<USERNAME>
<PASSWORD>
<SECRET>
```

Never publish:

```text
admin
RealPassword123
API-Key
PrivateKey
```

---

# 15. Fingerprint Matching

Fingerprint matching can use a configured match percentage.

Concept:

```text
Captured File
      |
      v
Fingerprint Comparison
      |
      v
Match Percentage
      |
      +---- Below Threshold --> No Match
      |
      +---- At/Above Threshold --> Match
                                      |
                                      v
                                    Action
```

Example:

```cli
set match-percentage 40
```

### Checklist

* [ ] Threshold defined
* [ ] Exact-match behavior tested
* [ ] Partial-match behavior tested
* [ ] Threshold boundary tested
* [ ] False positives reviewed
* [ ] Action verified

---

# 16. Cloud Storage DLP

Cloud storage can introduce additional DLP challenges.

Examples:

* [ ] Cloud drives
* [ ] SaaS applications
* [ ] SharePoint-like platforms
* [ ] Web-based storage
* [ ] Proprietary upload APIs

### Why cloud applications are different

A file upload may use:

```text
Multiple HTTP Requests
       +
Custom API
       +
Chunked Upload
       +
Encoding
       +
Separate Metadata
```

rather than:

```text
Simple HTTP POST
      |
      +--> filename
      +--> file content
```

### Cloud DLP checklist

* [ ] Cloud application identified
* [ ] Upload mechanism tested
* [ ] HTTPS decrypted where required
* [ ] DLP detection verified
* [ ] Block action verified
* [ ] Log accuracy verified
* [ ] Filename accuracy verified
* [ ] API/encoding behavior investigated

### Important troubleshooting principle

```text
File blocked successfully
        |
        +--> DLP detection works

Filename inaccurate
        |
        +--> Metadata/API behavior may be responsible
```

Do not automatically conclude that DLP detection failed because a cloud application's logged filename is incomplete or inaccurate.

---

# 17. Performance

DLP introduces additional processing overhead.

### Processing model

```text
Traffic
   |
   v
Proxy
   |
   v
Content Processing
   |
   v
DLP
   |
   v
Additional Security Profiles
   |
   v
Forwarding
```

### Production checklist

* [ ] FortiGate sizing reviewed
* [ ] Expected traffic volume measured
* [ ] File-size limits configured
* [ ] Unnecessary protocols excluded
* [ ] Targeted DLP rules used
* [ ] CPU monitored
* [ ] Memory monitored
* [ ] Throughput tested
* [ ] Latency measured
* [ ] Cloud applications tested separately

### Optimization principle

```text
More inspection
      |
      v
More processing
      |
      v
Potential throughput impact
```

---

# 18. Troubleshooting

## DLP Is Not Detecting Data

* [ ] DLP feature enabled
* [ ] Correct DLP profile selected
* [ ] Correct firewall policy matched
* [ ] Proxy inspection enabled where required
* [ ] SSL Deep Inspection configured for HTTPS
* [ ] Correct sensor configured
* [ ] Correct dictionary configured
* [ ] Correct data type configured
* [ ] Sensor match type verified
* [ ] Sensor threshold verified
* [ ] Rule protocol matches traffic
* [ ] File type matches traffic
* [ ] File size is within configured limit
* [ ] DLP action is correct
* [ ] Logging is enabled
* [ ] Test traffic contains the expected data

---

## HTTPS Upload Not Blocked

Trace the flow:

```text
HTTPS
  |
  v
SSL Deep Inspection
  |
  v
Decrypted Content?
  |
  +---- NO ---> DLP cannot inspect hidden payload
  |
  +---- YES
          |
          v
         DLP
          |
          v
       Detection
```

Checklist:

* [ ] SSL policy matches traffic
* [ ] Client trusts inspection CA
* [ ] Traffic is decrypted
* [ ] DLP profile is attached
* [ ] DLP sensor matches content
* [ ] Rule matches protocol/file
* [ ] Block action is configured
* [ ] Logs confirm processing

---

## Fingerprint Not Matching

```text
Fingerprint Failure
       |
       +--> Document Source
       |
       +--> Repository Access
       |
       +--> Fingerprint Database
       |
       +--> Refresh
       |
       +--> Match Percentage
       |
       +--> DLP Profile
       |
       +--> Test Traffic
```

Checklist:

* [ ] Document source reachable
* [ ] Repository credentials valid
* [ ] Correct file path configured
* [ ] Correct file pattern configured
* [ ] Database populated
* [ ] Database refreshed
* [ ] Fingerprint daemon operational
* [ ] Match percentage reviewed
* [ ] DLP rule uses fingerprint filtering
* [ ] Action configured
* [ ] Logs reviewed

---

## Cloud File Blocked but Filename Is Wrong

* [ ] Confirm DLP detection actually occurred
* [ ] Confirm traffic was blocked
* [ ] Review cloud API behavior
* [ ] Check whether filename is transferred separately
* [ ] Check encoding/chunking behavior
* [ ] Compare FortiGate logs with the original filename
* [ ] Do not confuse metadata accuracy with detection accuracy

---

# 19. Production Deployment

## Phase 1 — Discovery

* [ ] Identify sensitive information
* [ ] Identify sensitive files
* [ ] Identify business-critical applications
* [ ] Identify cloud-storage services
* [ ] Identify encrypted traffic
* [ ] Identify compliance requirements
* [ ] Identify acceptable false-positive rate

---

## Phase 2 — Detection

Start conservatively:

```text
Detect
  |
  v
Log
  |
  v
Analyze
  |
  v
Tune
```

* [ ] Start with logging/monitoring
* [ ] Identify false positives
* [ ] Tune dictionaries
* [ ] Tune sensors
* [ ] Tune file filters
* [ ] Review logs

---

## Phase 3 — Enforcement

After validation:

```text
Monitor
   |
   v
Tune
   |
   v
Block
   |
   v
Quarantine where justified
```

* [ ] Block high-confidence violations
* [ ] Define exceptions
* [ ] Document business requirements
* [ ] Test legitimate workflows
* [ ] Implement quarantine only where appropriate
* [ ] Monitor incidents after enforcement

---

## Phase 4 — Continuous Monitoring

* [ ] Review DLP logs
* [ ] Review violations
* [ ] Review false positives
* [ ] Review cloud application behavior
* [ ] Review resource utilization
* [ ] Update dictionaries
* [ ] Update detection logic
* [ ] Review sensitive-document repository
* [ ] Re-test after FortiOS upgrades

---

# 20. Practical Validation

## Test Matrix

| Test              | Expected Result                   | Status |
| ----------------- | --------------------------------- | ------ |
| Keyword detection | Match                             | [ ]    |
| Regex detection   | Match                             | [ ]    |
| Dictionary ANY    | Match when one entry matches      | [ ]    |
| Dictionary ALL    | Match when required entries match | [ ]    |
| Sensor count      | Threshold enforced                | [ ]    |
| HTTP upload       | DLP processes                     | [ ]    |
| HTTPS upload      | DLP processes after decryption    | [ ]    |
| File type         | Correct classification            | [ ]    |
| File size         | Threshold enforced                | [ ]    |
| Log Only          | Detect + log                      | [ ]    |
| Block             | Activity denied                   | [ ]    |
| Quarantine        | Restriction applied               | [ ]    |
| Fingerprint       | Known file detected               | [ ]    |
| Cloud storage     | Detection validated               | [ ]    |

---

## Controlled DLP Testing

Use a controlled test environment and approved test data.

Example:

```text
Client
  |
  v
HTTPS
  |
  v
FortiGate
  |
  +--> SSL Inspection
  |
  +--> DLP
  |
  +--> IPS
  |
  v
Test Destination
```

### Test cases

* [ ] Keyword
* [ ] Regex
* [ ] Dictionary
* [ ] Sensor threshold
* [ ] File type
* [ ] File size
* [ ] Log Only
* [ ] Block
* [ ] Fingerprint
* [ ] HTTPS upload
* [ ] HTTPS download
* [ ] Cloud-storage upload

---

# 21. NSE Exam Traps

## 🧠 Trap #1 — DLP ≠ Keyword Blocking

```text
DLP
 |
 +--> Data Types
 +--> Dictionaries
 +--> Sensors
 +--> Fingerprinting
 +--> Profiles
```

DLP is a content-inspection architecture, not simply keyword filtering.

---

## 🧠 Trap #2 — Proxy Mode

```text
Advanced DLP
      |
      v
Proxy Processing
```

Do not automatically assume flow mode provides all DLP capabilities.

---

## 🧠 Trap #3 — HTTPS

```text
Encrypted HTTPS
      |
      X
Content hidden from inspection
```

vs.

```text
HTTPS
  |
  v
SSL Deep Inspection
  |
  v
Decrypted Content
  |
  v
DLP
```

---

## 🧠 Trap #4 — Data Type vs Dictionary

```text
Data Type
  |
  +--> Defines detectable content
```

```text
Dictionary
  |
  +--> Groups related detection entries
```

---

## 🧠 Trap #5 — Dictionary vs Sensor

```text
Dictionary
  |
  +--> What entries belong together?
```

```text
Sensor
  |
  +--> How should entries be evaluated?
```

---

## 🧠 Trap #6 — Sensor Count

```text
count = 1
```

means the configured threshold can be satisfied when the required matching count reaches one.

---

## 🧠 Trap #7 — File Type vs Filename

If the requirement is:

```text
Block PDF files
```

prefer appropriate **file-type classification** rather than relying only on filename strings.

Cloud applications can make filename metadata unreliable.

---

## 🧠 Trap #8 — Fingerprinting

```text
Keyword
   |
   +--> What does the content contain?

Fingerprint
   |
   +--> Is this a known document/file?
```

---

## 🧠 Trap #9 — Internal Storage

Document fingerprinting has a storage requirement.

```text
Fingerprinting
      |
      v
Required internal storage capability
```

---

## 🧠 Trap #10 — Detection vs Logging Metadata

```text
File blocked
    +
Filename inaccurate
```

does **not automatically mean**:

```text
DLP detection failed
```

Cloud APIs may affect metadata presented in logs.

---

## 🧠 Trap #11 — Action

```text
Log Only
   |
   +--> Detect + Log
   +--> Do not confuse with Block
```

---

## 🧠 Trap #12 — DLP Architecture

Memorize:

```text
DATA TYPE
    ↓
DICTIONARY
    ↓
SENSOR
    ↓
DLP PROFILE
    ↓
FIREWALL POLICY
```

---

# 22. 60-Second Revision

```text
                 FORTIGATE DLP
                       |
                       v
                  WHAT TO FIND?
                       |
                       v
                   DATA TYPE
                       |
                       v
                HOW TO GROUP IT?
                       |
                       v
                  DICTIONARY
                       |
                       v
               HOW TO EVALUATE?
                       |
                       v
                    SENSOR
                       |
                       v
              WHAT SHOULD HAPPEN?
                       |
                       v
                 DLP PROFILE
                       |
                       v
               WHERE TO APPLY?
                       |
                       v
              FIREWALL POLICY
```

### Detection methods

```text
DLP
 |
 +--> Keyword
 +--> Regex
 +--> Hex
 +--> Credit Card
 +--> SSN
 +--> Dictionary
 +--> Sensor
 +--> Fingerprinting
 +--> Watermarking
```

### Enforcement

```text
Detection
    |
    +--> Allow
    +--> Log
    +--> Block
    +--> Quarantine
```

### Encrypted traffic

```text
HTTPS
  |
  v
SSL Deep Inspection
  |
  v
Decrypted Content
  |
  v
DLP
```

### Fingerprinting

```text
Document Repository
       |
       v
Fingerprint Database
       |
       v
Network File
       |
       v
Fingerprint Match
       |
       v
DLP Action
```

---

# 23. CLI Quick Reference

## DLP Profile

```bash
config dlp profile
    edit "dlp-prof-test"
        set feature-set proxy
    next
end
```

## DLP Sensor

```bash
config dlp sensor
    edit "sen-test"
        set match-type match-any

        config entries
            edit 1
                set dictionary "dlp-dic-test"
                set count 1
                set status enable
            next
        end
    next
end
```

## Data Type

```bash
config dlp data-type
    edit "custom-pattern"
        set pattern "<REGEX_OR_PATTERN>"
    next
end
```

## Fingerprint Document Source

```bash
config dlp fp-doc-source
    edit "dlp-doc-test"
        set server-type smb
        set server <SERVER_IP>
        set username "<USERNAME>"
        set password "<SECRET>"
        set file-path "<PATH>"
        set file-pattern "<PATTERN>"
    next
end
```

> ⚠️ Always verify exact CLI syntax and available options against the FortiOS release being deployed.

---

# 🔥 DLP One-Line Memory Aid

> **Data Type defines what to detect → Dictionary groups detection entries → Sensor evaluates the match → DLP Profile defines the action → Firewall Policy determines where DLP is applied. For encrypted traffic, SSL Deep Inspection exposes the content to DLP; for known documents, Fingerprinting compares the traffic against a fingerprint database.**

---

# 🎯 DLP Architecture at a Glance

```text
                         FORTIGATE DLP
                              |
          +-------------------+-------------------+
          |                   |                   |
          v                   v                   v
      DATA TYPES         FINGERPRINTING       WATERMARK
          |                   |                   |
          v                   v                   |
     DICTIONARIES       DOCUMENT SOURCE          |
          |                   |                   |
          v                   v                   |
       SENSORS          FINGERPRINT DB           |
          |                   |                   |
          +-------------------+-------------------+
                              |
                              v
                        DLP PROFILE
                              |
                              v
                       FIREWALL POLICY
                              |
                              v
                     SSL DEEP INSPECTION
                              |
                              v
                           TRAFFIC
```

---

# 🚨 Production Readiness Checklist

```text
DLP Production Readiness
|
+-- [ ] Business requirements documented
+-- [ ] Sensitive data identified
+-- [ ] Sensitive files identified
+-- [ ] Proxy inspection validated
+-- [ ] SSL inspection validated
+-- [ ] Data Types configured
+-- [ ] Dictionaries configured
+-- [ ] Sensors configured
+-- [ ] DLP Profile configured
+-- [ ] Protocol filters validated
+-- [ ] File types validated
+-- [ ] File sizes validated
+-- [ ] Actions tested
+-- [ ] Logging validated
+-- [ ] Fingerprinting tested
+-- [ ] Cloud applications tested
+-- [ ] False positives reviewed
+-- [ ] CPU/memory impact measured
+-- [ ] Exceptions documented
+-- [ ] Monitoring process defined
+-- [ ] Rollback plan prepared
+-- [ ] Credentials/secrets removed from documentation
```

---

# 🔗 Related FortiGate Topics

* [ ] SSL Deep Inspection
* [ ] Proxy-Based Inspection
* [ ] IPS
* [ ] Application Control
* [ ] Web Filter
* [ ] File Filter
* [ ] Firewall Policy
* [ ] FortiGuard
* [ ] Logging & FortiView
* [ ] Security Profiles

---

## 🧠 SheynShield Engineering Note

> Don't think of FortiGate DLP as simply **"block a keyword."**
>
> Think in layers:
>
> **Data Type → Dictionary → Sensor → DLP Profile → Firewall Policy**
>
> For encrypted traffic:
>
> **HTTPS → SSL Deep Inspection → DLP → Action**
>
> For known sensitive documents:
>
> **Document Source → Fingerprint Database → Fingerprint Match → DLP Action**
>
> And when troubleshooting cloud-storage DLP, always separate:
>
> **Detection accuracy ≠ Metadata/filename accuracy**

---

## 🔑 Keywords

```text
FortiGate DLP
FortiOS DLP
FortiGate Data Leak Prevention
FortiGate DLP profile
FortiGate DLP sensor
FortiGate DLP dictionary
FortiGate DLP data type
FortiGate DLP fingerprinting
FortiGate DLP fingerprint
FortiGate proxy inspection
FortiGate SSL deep inspection DLP
FortiGate HTTPS DLP
FortiGate sensitive data detection
FortiGate file inspection
FortiGate data leakage prevention
FortiOS NSE DLP
NSE4 DLP
NSE7 DLP
FortiGate troubleshooting
FortiGate security profiles
```

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

**SheynShield — Engineering Secure Networks**

*Practical Network Security Knowledge Base for Fortinet, Firewalls, Network Design & Security Engineering.*

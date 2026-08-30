حتماً — این نسخه را به‌صورت **GitHub-ready GFM Checklist** بازطراحی کردم؛ یعنی به‌جای صرفاً تبدیل متن به checklist، ساختار را برای **NSE revision، troubleshooting، production deployment، GitHub SEO و scanability** بهینه کردم. موارد مفهومی، CLI، validation و operational checks از هم تفکیک شده‌اند.

````markdown
# 🛡️ FortiGate IPS — Engineering & Tuning Checklist

> **FortiOS IPS | Custom IPS Signatures | Protocol Decoders | PCRE | Rate-Based Detection | Tags | Hold Time | Hardware Acceleration | Performance Tuning**

[![FortiOS](https://img.shields.io/badge/FortiOS-IPS-red?style=flat-square)](#)
[![Fortinet](https://img.shields.io/badge/Fortinet-Security-orange?style=flat-square)](#)
[![Security](https://img.shields.io/badge/Domain-Network%20Security-blue?style=flat-square)](#)
[![Checklist](https://img.shields.io/badge/Format-Checklist-success?style=flat-square)](#)

> **SheynShield | Engineering Secure Networks**

---

## 📌 Table of Contents

- [1. IPS Architecture](#-1-ips-architecture)
- [2. Deep Packet Inspection](#-2-deep-packet-inspection)
- [3. Protocol Decoder](#-3-protocol-decoder)
- [4. Custom IPS Signature](#-4-custom-ips-signature)
- [5. Signature Metadata](#-5-signature-metadata)
- [6. Service vs Port](#-6-service-vs-port)
- [7. Flow Direction](#-7-flow-direction)
- [8. Pattern Matching](#-8-pattern-matching)
- [9. PCRE](#-9-pcre)
- [10. Context](#-10-context)
- [11. Range Modifiers](#-11-range-modifiers)
- [12. Byte Matching](#-12-byte-matching)
- [13. Parsed Type](#-13-parsed-type)
- [14. Rate-Based Detection](#-14-rate-based-detection)
- [15. IPS Tags](#-15-ips-tags)
- [16. Signature Weight](#-16-signature-weight)
- [17. Malicious URL Protection](#-17-malicious-url-protection)
- [18. Signature Deployment](#-18-signature-deployment)
- [19. IPS Filter Logic](#-19-ips-filter-logic)
- [20. Signature Hold Time](#-20-signature-hold-time)
- [21. Hardware Acceleration](#-21-hardware-acceleration)
- [22. IPS Database](#-22-ips-database)
- [23. Industrial IPS](#-23-industrial-ips)
- [24. IPS Engine Count](#-24-ips-engine-count)
- [25. Socket Size](#-25-socket-size)
- [26. Fail-Open](#-26-fail-open)
- [27. Resource Accuracy](#-27-resource-accuracy)
- [28. Decoder Configuration](#-28-decoder-configuration)
- [29. Brute-Force Detection](#-29-brute-force-detection)
- [30. Custom Signature Testing](#-30-custom-signature-testing)
- [31. Performance Optimization](#-31-performance-optimization)
- [32. Troubleshooting](#-32-troubleshooting)
- [33. NSE Exam Checklist](#-33-nse-exam-checklist)
- [34. Production Design](#-34-production-design)
- [35. Quick CLI Reference](#-35-quick-cli-reference)
- [36. One-Minute Revision](#-36-one-minute-revision)
- [37. Final Engineering Principle](#-37-final-engineering-principle)
- [SheynShield Resources](#-sheynshield-resources)

---

# 🔥 1. IPS Architecture

## IPS Engine Components

- [ ] Signature matching
- [ ] Protocol decoding
- [ ] Pattern matching
- [ ] PCRE matching
- [ ] Heuristic detection
- [ ] Threat intelligence
- [ ] Anomaly detection
- [ ] Application awareness
- [ ] CVE/security intelligence
- [ ] Rate-based detection
- [ ] Session tracking
- [ ] Custom signatures

## Inspection Flow

```text
Incoming Traffic
       │
       ▼
Protocol Decoder
       │
       ▼
Service Identification
       │
       ▼
IPS Signatures
       │
       ├── Pattern Matching
       ├── PCRE
       ├── Behavioral Detection
       ├── Rate Detection
       └── Threat Intelligence
              │
              ▼
         IPS Decision
              │
       ┌──────┼──────┐
       ▼      ▼      ▼
     Allow   Log   Block
````

### Engineering Checks

* [ ] Understand that IPS is more than signature matching
* [ ] Confirm protocol identification
* [ ] Confirm application/service context
* [ ] Confirm signature matching scope
* [ ] Confirm behavioral/rate detection where required
* [ ] Confirm final IPS action

---

# 🔎 2. Deep Packet Inspection

## Verify Inspection Visibility

* [ ] Packet headers are visible
* [ ] Protocol fields are visible
* [ ] Application-layer data is visible
* [ ] HTTP URI is inspectable
* [ ] HTTP headers are inspectable
* [ ] HTTP methods are inspectable
* [ ] HTTP cookies are inspectable
* [ ] HTTP body is inspectable
* [ ] File content is inspectable where supported
* [ ] Email content is inspectable where supported
* [ ] Protocol-specific structures are decoded

### Example

```text
HTTP
├── Method
├── Host
├── URI
├── Header
├── Cookie
└── Body
```

### Encrypted Traffic

* [ ] Determine whether required payload is encrypted
* [ ] Verify SSL/TLS inspection requirements
* [ ] Verify application visibility
* [ ] Verify the selected IPS context is compatible with the available visibility
* [ ] Do not assume TCP/443 automatically means HTTP payload is inspectable

---

# 🧩 3. Protocol Decoder

> **Protocol decoder = protocol structure awareness.**

### Verify

* [ ] Correct protocol is identified
* [ ] Protocol syntax is validated
* [ ] Malformed protocol behavior can be detected
* [ ] Decoder anomalies are considered
* [ ] Relevant service tree is selected

```text
Traffic
   │
   ▼
Protocol Decoder
   │
   ├── Valid protocol
   │      ↓
   │   Continue IPS
   │
   └── Malformed protocol
          ↓
       Anomaly
```

### Key Principle

* [ ] Remember: **Decoder identifies protocol structure**
* [ ] Remember: **Signature identifies suspicious behavior/content**

---

# 🛠️ 4. Custom IPS Signature

## Basic F-SBID Structure

```text
F-SBID(
    --name "Block.SMTP.VRFY.CMD";
    --protocol tcp;
    --service SMTP;
    --pattern "vrfy";
    --no_case;
    --context header;
)
```

### Signature Checklist

* [ ] `F-SBID` is uppercase
* [ ] Signature syntax is valid
* [ ] `--name` is defined
* [ ] `--protocol` is appropriate
* [ ] `--service` is appropriate
* [ ] `--flow` is appropriate
* [ ] `--pattern` is specific
* [ ] `--context` is correct
* [ ] PCRE is used only when necessary
* [ ] `--attack_id` is unique where required
* [ ] Weight is intentionally selected

---

## CLI Deployment

```bash
config ips custom
    edit "ips-cus-test"
        set signature 'F-SBID( --name "Block.SMTP.VRFY.CMD"; --protocol tcp; --service SMTP; --pattern "vrfy"; --no_case; --context header; )'
    next
end
```

### Validation

* [ ] CLI accepts the signature
* [ ] Signature appears in configuration
* [ ] Signature is enabled
* [ ] Signature is visible to the IPS sensor
* [ ] Signature is attached to the required policy

---

# 🏷️ 5. Signature Metadata

## Important Fields

| Field         | Purpose                  |
| ------------- | ------------------------ |
| `--name`      | Signature name           |
| `--attack_id` | Signature identifier     |
| `--protocol`  | Transport protocol       |
| `--service`   | Application/service tree |
| `--flow`      | Traffic direction        |
| `--pattern`   | Literal matching         |
| `--pcre`      | Regex matching           |
| `--context`   | Inspection buffer        |
| `--weight`    | Signature priority       |

### Naming Checklist

* [ ] Name is unique
* [ ] Name is descriptive
* [ ] Name follows organizational naming convention
* [ ] Name stays within supported length
* [ ] Vendor/product/component information is included where useful

Recommended:

```text
vendor.product.component.vulnerability
```

Example:

```text
IBM.Domino.iNotes.Foldername.Buffer.Overflow
```

---

# 🌐 6. Service vs Port

> **Service identification is generally preferable to relying only on a destination port when the IPS engine supports the relevant service.**

### Prefer

```text
--service HTTP;
```

### Instead of unnecessarily relying on

```text
--dst_port 80;
```

### Service Tree

```text
IPS Engine
    │
    ├── HTTP
    ├── DNS
    ├── SMTP
    ├── SSL
    ├── SSH
    └── ...
```

### Checklist

* [ ] Application/service is known
* [ ] Correct IPS service tree is selected
* [ ] Port-only matching is avoided when unnecessary
* [ ] Generic signatures are avoided where possible
* [ ] Signature scope is minimized

> ⚠️ **Engineering Rule:** Broad signatures can increase CPU consumption and false positives.

---

# 🔄 7. Flow Direction

## Client → Server

```text
--flow from_client;
```

* [ ] Use for client-originated attack traffic

## Server → Client

```text
--flow from_server;
```

* [ ] Use for server-originated attack traffic

## Both Directions

```text
--flow bi_direction;
```

* [ ] Use when either direction is relevant

### Special Case

* [ ] Understand `reversed` when attack location needs to be represented differently in the GUI
* [ ] Consider server-generated authentication failures in brute-force detection

---

# 🧱 8. Pattern Matching

## Basic Pattern

```text
--pattern "vrfy";
```

### Checklist

* [ ] Pattern is enclosed in quotes
* [ ] Pattern ends with `;`
* [ ] Pattern is sufficiently specific
* [ ] Pattern is not unnecessarily short
* [ ] Pattern is resilient to legitimate variations
* [ ] `--no_case` is used only when required

### Case Sensitive

```text
--pattern "Admin";
```

### Case Insensitive

```text
--pattern "Admin";
--no_case;
```

Matches:

```text
Admin
admin
ADMIN
AdMiN
```

### Pattern Quality

Avoid:

```text
--pattern "GET";
```

when a more discriminating pattern is possible.

Prefer:

```text
--pattern "GET /admin/login";
```

where appropriate.

---

# ⚠️ 9. PCRE

## Example

```text
--pcre "/admin/i";
```

### Checklist

* [ ] PCRE is actually required
* [ ] Simpler pattern matching cannot solve the requirement
* [ ] Regex is deterministic
* [ ] Wildcards are minimized
* [ ] Backtracking is controlled
* [ ] Inspection context is limited
* [ ] Production performance has been tested

### Prefer

```text
--pattern "specific-value";
```

### Before

```text
--pcre "/.*/";
```

> ⚠️ **PCRE can be significantly more expensive than simple pattern matching.**

---

# 📍 10. Context

Context defines **where** the IPS engine searches.

### Common Contexts

* [ ] `uri`
* [ ] `header`
* [ ] `host`
* [ ] `body`
* [ ] `file`
* [ ] `packet`
* [ ] `packet_origin`

### Examples

HTTP URI:

```text
--context uri;
```

HTTP Host:

```text
--context host;
```

HTTP Header:

```text
--context header;
```

File:

```text
--context file;
```

### Best Practice

* [ ] Select the narrowest useful context
* [ ] Avoid scanning irrelevant buffers
* [ ] Verify the context exists for the selected service

---

# 📦 11. Range Modifiers

## Available Concepts

| Modifier       | Purpose                |
| -------------- | ---------------------- |
| `distance`     | Relative search start  |
| `within`       | Relative search window |
| `distance_abs` | Absolute search start  |
| `within_abs`   | Absolute search window |
| `offset`       | Start from an offset   |
| `depth`        | Limit search depth     |
| `isdataat`     | Verify data exists     |
| `pcre`         | Regex matching         |

### Distance + Within

```text
--pattern "login";
--distance 10;
--within 20;
--pattern "fail";
```

Concept:

```text
Previous Match
      │
      │<── distance 10 ──>
      ▼
    Search
      │
      │<──── within 20 ────>
      ▼
   Next Match
```

### Checklist

* [ ] Understand the reference point
* [ ] Use `distance + within` when appropriate
* [ ] Use `distance_abs + within_abs` when absolute positioning is required
* [ ] Avoid unnecessarily broad search windows
* [ ] Validate negative-pattern behavior

---

# 🧮 12. Byte Matching

## `byte_test`

Used to evaluate numeric protocol fields.

```text
--byte_test 2,>,100,0;
```

Concept:

```text
Read 2 bytes
     │
     ▼
Compare with 100
     │
     ▼
Condition satisfied?
```

### Use Cases

* [ ] Protocol length validation
* [ ] Numeric field matching
* [ ] Malformed packet detection
* [ ] Protocol-specific field validation

---

## `byte_jump`

Moves the inspection reference according to a value in packet data.

```text
--byte_jump 4,0,relative;
```

Concept:

```text
Read numeric field
       ↓
Calculate offset
       ↓
Move reference
       ↓
Continue matching
```

### Checklist

* [ ] Correct byte count
* [ ] Correct offset
* [ ] Correct relative/absolute behavior
* [ ] Protocol structure verified

---

## Endianness

Example:

```text
AA BB CC DD
```

Big-endian:

```text
AA BB CC DD
```

Little-endian:

```text
DD CC BB AA
```

* [ ] Verify protocol byte order
* [ ] Do not assume endianness

---

## `align`

* [ ] Use `align` only when required by protocol structure
* [ ] Understand 32-bit boundary rounding

Example:

```text
7 bytes  → 8 bytes
33 bytes → 36 bytes
```

---

# 🧩 13. Parsed Type

`--parsed_type` allows matching against a specific parsed protocol/application structure.

Example:

```text
--parsed_type HTTP-GET;
```

### Checklist

* [ ] Parsed protocol structure is known
* [ ] Correct parsed type is selected
* [ ] Signature is not unnecessarily broad

---

# 📊 14. Rate-Based Detection

Useful for behavioral attacks such as:

* [ ] Brute force
* [ ] Credential attacks
* [ ] Excessive requests
* [ ] Repeated protocol abuse
* [ ] Abnormal request rates
* [ ] Application abuse

Concept:

```text
--rate <count>,<duration>[,<limit>];
```

Example:

```text
3 events / 30 seconds
```

### Detection Model

```text
Event
  ↓
Event
  ↓
Event
  ↓
Threshold
  ↓
Action
```

---

## Track-Based Detection

Possible tracking dimensions can include:

* [ ] `src_ip`
* [ ] `dst_ip`
* [ ] DHCP client
* [ ] DNS domain
* [ ] DNS domain + IP

### Example

```text
Track = src_ip
```

Concept:

```text
Source A
 ├── Event 1
 ├── Event 2
 ├── Event 3
 └── Event 4
       ↓
 Threshold
       ↓
 Action
```

### Checklist

* [ ] Select the correct tracking key
* [ ] Understand what contributes to the counter
* [ ] Validate behavior with multiple clients
* [ ] Test threshold behavior

---

# ⏱️ Continuous vs Periodical

## Continuous

```text
set rate-mode continuous
```

Useful for:

* [ ] Brute force
* [ ] Persistent attacks
* [ ] Repeated authentication failures
* [ ] Continuing suspicious behavior

Concept:

```text
Attack
  ↓
Counter
  ↓
Threshold
  ↓
Action
```

---

## Periodical

Useful for:

* [ ] Crawlers
* [ ] DNS patterns
* [ ] Mail traffic
* [ ] APIs
* [ ] Repeated access attempts

Concept:

```text
Time Window
     ↓
Count Events
     ↓
Threshold
     ↓
Evaluate
     ↓
New Period
```

---

# 🏷️ 15. IPS Tags

Tags allow signatures to maintain state across packets within a session.

### Set

```text
--tag pset,Tag.Login;
```

### Test

```text
--tag test,Tag.Login;
```

### Test NOT

```text
--tag test,!Tag.Login;
```

### Operations

| Operation | Purpose           |
| --------- | ----------------- |
| `pset`    | Set tag/reference |
| `toggle`  | Toggle tag state  |
| `test`    | Test tag          |
| `test,!`  | Test tag absence  |

### Multi-Stage Detection

```text
Packet 1
   ↓
Detect Stage 1
   ↓
Set Tag
   ↓
Packet 2
   ↓
Test Tag
   ↓
Detect Stage 2
```

### Checklist

* [ ] Tag naming is unique
* [ ] Stage 1 sets the tag
* [ ] Stage 2 tests the tag
* [ ] Session state is understood
* [ ] Multi-packet behavior is tested

---

# ⚖️ 16. Signature Weight

Example:

```text
--weight 40;
```

Supported range:

```text
0 - 255
```

### Checklist

* [ ] Understand signature priority behavior
* [ ] Avoid arbitrary weight assignment
* [ ] Define an organizational convention
* [ ] Test interactions with other signatures

### Important

> Weight affects signature priority where applicable; it should not be treated as a universal "more secure" setting.

---

# 🌐 17. Malicious URL Protection

Example:

```bash
config ips sensor
    edit "ips-test"
        set block-malicious-url enable
    next
end
```

### Checklist

* [ ] Feature is supported on target FortiOS
* [ ] IPS sensor is identified
* [ ] Malicious URL protection is enabled where required
* [ ] FortiGuard connectivity/status is verified
* [ ] Logs are monitored

### Drive-By Protection Model

```text
User
  ↓
Malicious / Compromised Website
  ↓
HTTP Response
  ├── Redirect
  ├── JavaScript
  ├── Exploit
  └── Malicious Download
           ↓
          IPS
           ↓
       Detect / Block
```

---

# 🛡️ 18. Signature Deployment

> **Creating a custom signature does NOT automatically activate it.**

Required chain:

```text
Custom Signature
      ↓
IPS Sensor
      ↓
Firewall Policy
      ↓
Traffic
```

### Deployment Checklist

* [ ] Create custom signature
* [ ] Validate syntax
* [ ] Add signature to IPS sensor
* [ ] Select action
* [ ] Attach IPS sensor to firewall policy
* [ ] Verify policy is actually matching traffic
* [ ] Generate test traffic
* [ ] Verify IPS event logs

---

## GUI Deployment Flow

```text
Security Profiles
      ↓
Intrusion Prevention
      ↓
Create / Edit IPS Profile
      ↓
Signature / Filter
      ↓
Select Custom Signature
      ↓
Set Action
      ↓
Apply to Firewall Policy
```

---

# 🧩 19. IPS Filter Logic

When multiple attributes are configured in a filter, they generally narrow the matching scope.

Example:

```text
Server
AND
TCP
AND
Medium → High Severity
```

Concept:

```text
Role = Server
      AND
Protocol = TCP
      AND
Severity = Medium..High
```

### Checklist

* [ ] Understand AND-style filtering
* [ ] Avoid excessively broad filters
* [ ] Separate filters by security requirement
* [ ] Apply granular actions
* [ ] Validate resulting signature set

### Better Design

```text
Role 1 → High Severity
Role 2 → Medium Severity
Role 3 → Specific Protocol
```

---

# ⏳ 20. Signature Hold Time

FortiGate can temporarily hold newly introduced/updated IPS signatures.

Example:

```bash
config system ips
    set signature-hold-time 3d12h
end
```

### Disable

```bash
config system ips
    set signature-hold-time 0
end
```

### Concept

```text
FortiGuard
    ↓
New / Updated Signature
    ↓
Hold Period
    ↓
Validation / Monitoring
    ↓
Active Enforcement
```

### Why Use Hold Time?

* [ ] Controlled signature rollout
* [ ] Reduce operational risk
* [ ] Validate potential false positives
* [ ] Monitor new signature behavior

### Critical Reminder

* [ ] Do NOT interpret hold time as "IPS disabled"
* [ ] Existing active signatures remain relevant
* [ ] Custom signatures can continue providing protection
* [ ] Other security controls remain important

---

## Override Hold by ID

```bash
config system ips
    set override-signature-hold-by-id enable
end
```

### Checklist

* [ ] Enable only when required
* [ ] Identify exceptional signatures
* [ ] Document overrides
* [ ] Monitor resulting behavior

---

## Troubleshoot Held Signatures

```bash
diagnose ips signature on-hold vd1
```

Specific signature:

```bash
diagnose ips signature on-hold vd1 29844
```

* [ ] Verify signature is on hold
* [ ] Verify correct VDOM
* [ ] Verify signature ID
* [ ] Confirm expected hold behavior

---

# ⚡ 21. Hardware Acceleration

Supported FortiGate platforms can use hardware acceleration for IPS workloads.

Potential processing components include:

```text
CPU
 │
 ├── IPS Engine
 │
 └── Acceleration
      ├── CP
      ├── NP
      └── Platform-specific IPS acceleration
```

### Checklist

* [ ] Verify platform capabilities
* [ ] Verify supported acceleration path
* [ ] Check CPU utilization
* [ ] Check IPS engine utilization
* [ ] Check NP/CP activity where applicable
* [ ] Avoid assuming every model supports the same features

---

# ⚙️ 22. IPS Acceleration Mode

Example:

```bash
config ips global
    set np-accel-mode basic
    set cp-accel-mode basic
end
```

Possible CP modes may include:

```text
none
basic
advanced
```

## `none`

```text
CPU
 ↓
IPS Processing
```

Useful for:

* [ ] Troubleshooting
* [ ] Controlled testing
* [ ] Environments where acceleration is intentionally disabled

## `basic`

* [ ] Supported acceleration for common workloads

## `advanced`

* [ ] Use only where supported
* [ ] Validate platform behavior
* [ ] Benchmark before/after

> ⚠️ Never select acceleration mode from a generic model list. Verify the exact FortiGate platform and FortiOS release.

---

# 🧠 23. IPS Database

Depending on platform/configuration, FortiGate may provide different IPS database options.

Concept:

```text
Regular / Slim Database
        vs
Extended Database
```

Example:

```bash
config ips global
    set database extended
end
```

### Engineering Trade-off

```text
More Signatures
      ↕
More Coverage
      ↕
More Resources
```

### Checklist

* [ ] Verify database support
* [ ] Select appropriate database
* [ ] Check memory usage
* [ ] Check CPU impact
* [ ] Validate required signature coverage

---

# 🏭 24. Industrial IPS

Industrial/OT signatures may require specific inclusion behavior.

Example:

```bash
config ips global
    set exclude-signature none
end
```

### OT Deployment Checklist

* [ ] Inventory OT protocols
* [ ] Identify critical assets
* [ ] Identify communication patterns
* [ ] Enable required industrial signatures
* [ ] Monitor false positives
* [ ] Validate signatures in maintenance windows
* [ ] Monitor resource impact
* [ ] Document exceptions

### OT Security Model

```text
Inventory
   ↓
Protocol Identification
   ↓
Required Signatures
   ↓
Controlled Deployment
   ↓
Monitoring
   ↓
Tuning
```

---

# 🧠 25. IPS Engine Count

Example:

```bash
config ips global
    set engine-count 0
end
```

`0` can allow FortiOS to optimize the value according to the platform.

Concept:

```text
Traffic
 ├── IPS Engine 1
 ├── IPS Engine 2
 ├── IPS Engine 3
 └── IPS Engine N
```

### Checklist

* [ ] Understand platform-specific engine behavior
* [ ] Check CPU
* [ ] Check memory
* [ ] Check concurrent sessions
* [ ] Check traffic volume
* [ ] Check inspection workload
* [ ] Avoid assuming more engines always means better performance

---

# 💾 26. Socket Size

Example:

```bash
config ips global
    set socket-size 128
end
```

Concept:

```text
Kernel
   ↓
Socket Buffer
   ↓
IPS Engine
```

### Too Small

```text
Small Buffer
    ↓
IPS processing pressure
    ↓
Potential fail-open/resource events
```

### Too Large

```text
Large Buffer
    ↓
Higher Memory Consumption
    ↓
Conserve Mode Pressure
```

### Checklist

* [ ] Check memory utilization
* [ ] Check IPS engine load
* [ ] Check concurrent sessions
* [ ] Check traffic rate
* [ ] Check fail-open events
* [ ] Change socket size only with measured evidence

> **Golden Rule:** Never blindly increase `socket-size`.

---

# 🚨 27. Fail-Open

Example:

```bash
config ips global
    set fail-open disable
end
```

### Concept

```text
IPS Resource Pressure
        │
        ├── Fail-Open Enabled
        │       ↓
        │   Forward Traffic
        │
        └── Fail-Open Disabled
                ↓
        Traffic May Be Dropped
```

### Security vs Availability

| Configuration        | Primary Bias         |
| -------------------- | -------------------- |
| Fail-open            | Availability         |
| Fail-closed behavior | Security enforcement |

### Checklist

* [ ] Define business availability requirement
* [ ] Define security risk tolerance
* [ ] Understand failure behavior
* [ ] Test under resource pressure
* [ ] Document the decision

---

# ⚠️ 28. Fail-Open ≠ Hardware Acceleration Overload

Do not automatically equate:

```text
IPS fail-open
```

with:

```text
NP / CP / IPS acceleration overload
```

### Troubleshooting Checklist

* [ ] Check CPU
* [ ] Check memory
* [ ] Check IPS engine state
* [ ] Check acceleration state
* [ ] Check socket/buffer usage
* [ ] Check conserve mode
* [ ] Check fail-open events
* [ ] Identify the actual bottleneck before changing configuration

---

# 📈 29. Resource Accuracy

IPS resource/session handling can use optimized heuristics.

Concept:

```text
Maximum Resource
       ↓
Safety / Heuristic Margin
       ↓
Operational IPS Capacity
```

### Checklist

* [ ] Understand current resource limits
* [ ] Monitor high-load behavior
* [ ] Check CPU
* [ ] Check memory
* [ ] Check sessions
* [ ] Check buffer usage
* [ ] Check acceleration
* [ ] Check conserve mode
* [ ] Change resource-related parameters only after measurement

---

# 🔬 30. Decoder Configuration

Some FortiOS versions/platforms allow decoder-specific configuration.

Example:

```bash
config ips decoder
    config dns
        config parameter
            edit "port list"
                set value "100,200,300"
            next
        end
    end
end
```

### Checklist

* [ ] Verify decoder support
* [ ] Verify FortiOS version
* [ ] Check whether decoder uses automatic detection
* [ ] Confirm supported port configuration
* [ ] Avoid assuming arbitrary port assignment is supported

---

# 🧨 31. Brute-Force Detection

Strong brute-force detection can combine:

```text
Signature
    +
Rate
    +
Track
    +
Action
```

### Detection Flow

```text
Authentication Failure
       ↓
Signature Match
       ↓
Track Source IP
       ↓
3 Events / 30 Seconds
       ↓
Threshold
       ↓
Block / Quarantine
```

### Checklist

* [ ] Identify authentication-failure pattern
* [ ] Create specific signature
* [ ] Select correct service
* [ ] Select correct context
* [ ] Select correct flow
* [ ] Configure rate
* [ ] Configure tracking
* [ ] Define action
* [ ] Test legitimate authentication
* [ ] Test brute-force behavior
* [ ] Verify logs

---

# 🧪 32. Custom Signature Testing

## Syntax

* [ ] `F-SBID` syntax validated
* [ ] Signature accepted by FortiGate
* [ ] No malformed keywords
* [ ] Correct quotation/escaping

## Identification

* [ ] Unique attack ID
* [ ] Unique name
* [ ] Correct protocol
* [ ] Correct service
* [ ] Correct flow
* [ ] Correct context

## Matching

* [ ] Pattern is sufficiently specific
* [ ] `--no_case` is necessary
* [ ] PCRE is justified
* [ ] Range modifiers are minimized
* [ ] Byte operations are validated
* [ ] Tags are tested if used
* [ ] Rate logic is tested if used

## Security

* [ ] Correct action
* [ ] False positives tested
* [ ] False negatives considered
* [ ] Legitimate traffic tested
* [ ] Malicious traffic tested

## Deployment

* [ ] Signature added to IPS sensor
* [ ] IPS sensor attached to firewall policy
* [ ] SSL inspection configured if required
* [ ] Logging enabled
* [ ] Event generated successfully

## Performance

* [ ] CPU tested
* [ ] Memory tested
* [ ] Session impact tested
* [ ] IPS engine load checked
* [ ] Production traffic volume tested

---

# 🚀 33. Performance Optimization

## Preferred Design

```text
Specific Service
      ↓
Specific Context
      ↓
Strong Pattern
      ↓
Minimal Search Range
      ↓
Simple Matching
      ↓
Controlled PCRE
```

## Avoid

```text
Generic Service
      +
No Context
      +
Short Pattern
      +
Broad Range
      +
Complex PCRE
```

### Optimization Checklist

* [ ] Use the most specific service possible
* [ ] Use the narrowest useful context
* [ ] Use longer unique patterns
* [ ] Minimize search ranges
* [ ] Prefer literal patterns
* [ ] Use PCRE only when necessary
* [ ] Avoid excessive wildcards
* [ ] Avoid broad generic signatures
* [ ] Benchmark custom signatures
* [ ] Monitor CPU/memory impact

---

# 🧭 34. IPS Troubleshooting

## Signature Not Matching

* [ ] Is the firewall policy correct?
* [ ] Is the IPS profile attached?
* [ ] Is IPS enabled?
* [ ] Is traffic actually passing through the expected policy?
* [ ] Is the correct inspection mode configured?
* [ ] Is SSL/TLS visibility available?
* [ ] Is the protocol correctly identified?
* [ ] Is the correct service tree selected?
* [ ] Is the flow direction correct?
* [ ] Is the context correct?
* [ ] Is the pattern actually present?
* [ ] Is the signature enabled?
* [ ] Is the signature on hold?
* [ ] Is the signature excluded/filtered?
* [ ] Has the rate threshold been reached?
* [ ] Is the configured action correct?

### Troubleshooting Flow

```text
Signature Not Matching
        │
        ▼
Firewall Policy?
        │
        ▼
IPS Profile?
        │
        ▼
Inspection Mode?
        │
        ▼
SSL Visibility?
        │
        ▼
Protocol?
        │
        ▼
Service Tree?
        │
        ▼
Flow?
        │
        ▼
Context?
        │
        ▼
Pattern?
        │
        ▼
Signature Hold?
        │
        ▼
Filter?
        │
        ▼
Rate Threshold?
        │
        ▼
Action?
```

---

# 🎓 35. NSE Exam Checklist

## Custom Signature

* [ ] `F-SBID`
* [ ] `--name`
* [ ] `--attack_id`
* [ ] `--protocol`
* [ ] `--service`
* [ ] `--flow`
* [ ] `--pattern`
* [ ] `--pcre`
* [ ] `--context`

## Matching

* [ ] Pattern
* [ ] PCRE
* [ ] Distance
* [ ] Within
* [ ] Byte Test
* [ ] Byte Jump

## State

* [ ] Tags
* [ ] Rate
* [ ] Track

## Priority

* [ ] Weight

## Deployment

* [ ] Custom Signature
* [ ] IPS Sensor
* [ ] Firewall Policy
* [ ] Required SSL inspection

## Performance

* [ ] Engine count
* [ ] Socket size
* [ ] CP acceleration
* [ ] NP acceleration
* [ ] IPS database
* [ ] Fail-open
* [ ] CPU
* [ ] Memory

## Lifecycle

* [ ] Signature Hold Time
* [ ] Signature override
* [ ] On-hold diagnostics

---

# 🏗️ 36. Production Design

```text
                         Internet
                            │
                            ▼
                  ┌───────────────────┐
                  │     FortiGate     │
                  │                   │
                  │  SSL Inspection   │
                  │        ↓          │
                  │ Protocol Decoder  │
                  │        ↓          │
                  │    IPS Engine      │
                  │                   │
                  │ ├─ Signatures     │
                  │ ├─ PCRE           │
                  │ ├─ Rate           │
                  │ ├─ Decoder        │
                  │ └─ Intelligence   │
                  └─────────┬─────────┘
                            │
                            ▼
                           LAN
```

## Defense-in-Depth Checklist

* [ ] Firewall policy
* [ ] SSL inspection
* [ ] IPS
* [ ] Application Control
* [ ] Web Filtering
* [ ] DNS Security
* [ ] DoS Protection
* [ ] Malware Protection
* [ ] Sandbox where appropriate
* [ ] Logging/Monitoring
* [ ] Incident response integration

> **IPS is one security layer — not the entire security architecture.**

---

# 🧰 37. Quick CLI Reference

## Custom IPS Signature

```bash
config ips custom
    edit "custom-signature"
        set signature 'F-SBID( --name "Example.Custom"; --protocol tcp; --service HTTP; --flow from_client; --pattern "malicious"; --context uri; )'
    next
end
```

## IPS Global

```bash
config ips global
    set engine-count 0
    set socket-size 128
    set fail-open disable
end
```

## IPS Database

```bash
config ips global
    set database extended
end
```

## Industrial Signatures

```bash
config ips global
    set exclude-signature none
end
```

## Signature Hold Time

```bash
config system ips
    set signature-hold-time 3d12h
    set override-signature-hold-by-id enable
end
```

## Malicious URL

```bash
config ips sensor
    edit "ips-test"
        set block-malicious-url enable
    next
end
```

## Signature Hold Diagnostics

```bash
diagnose ips signature on-hold vd1
```

```bash
diagnose ips signature on-hold vd1 29844
```

---

# 📊 38. IPS Quick Decision Table

| Requirement           | Preferred Mechanism                   |
| --------------------- | ------------------------------------- |
| Known attack string   | `--pattern`                           |
| Complex matching      | `--pcre`                              |
| HTTP URI              | `--context uri`                       |
| HTTP Host             | `--context host`                      |
| HTTP Header           | `--context header`                    |
| File content          | `--context file`                      |
| Client → Server       | `--flow from_client`                  |
| Server → Client       | `--flow from_server`                  |
| Both directions       | `--flow bi_direction`                 |
| Repeated behavior     | `--rate`                              |
| Per-source tracking   | `--track`                             |
| Multi-stage attack    | `--tag`                               |
| Numeric field         | `--byte_test`                         |
| Dynamic offset        | `--byte_jump`                         |
| Relative search       | `distance + within`                   |
| Signature priority    | `--weight`                            |
| New signature rollout | Hold Time                             |
| High throughput       | Hardware acceleration where supported |
| Resource pressure     | Tune based on measurement             |
| Encrypted payload     | Appropriate SSL visibility            |

---

# 🧠 39. NSE Exam Memory Map

```text
CUSTOM IPS
│
├── F-SBID
│   ├── name
│   ├── attack_id
│   ├── protocol
│   ├── service
│   ├── flow
│   ├── pattern
│   ├── pcre
│   └── context
│
├── MATCHING
│   ├── pattern
│   ├── pcre
│   ├── distance
│   ├── within
│   ├── byte_test
│   └── byte_jump
│
├── STATE
│   ├── tag
│   └── rate / track
│
├── PRIORITY
│   └── weight
│
└── DEPLOYMENT
    ├── Custom Signature
    ├── IPS Sensor
    ├── Firewall Policy
    └── Required Inspection
```

---

# ⚡ 40. One-Minute Revision

```text
Service       = Where/application tree
Port          = Transport endpoint
Context       = Where to inspect
Pattern       = Simple matching
PCRE          = Powerful regex matching
Flow          = Direction
Rate          = Frequency
Track         = What is counted separately
Tag           = Session/multi-stage state
Distance      = Relative search start
Within        = Relative search window
Byte_Test     = Numeric comparison
Byte_Jump     = Dynamic reference movement
Weight        = Signature priority
Decoder       = Protocol structure
Hold Time     = Controlled signature activation
Socket Size   = IPS buffering
Engine Count  = IPS processing instances
Acceleration  = Hardware-assisted processing
Fail-Open     = Availability vs enforcement
```

---

# 🎯 41. High-Value Engineering Rules

> ### Rule 01 — Service
>
> **Use the appropriate service tree instead of relying only on a port when the application protocol is known.**

> ### Rule 02 — Context
>
> **Use the narrowest practical inspection context.**

> ### Rule 03 — Pattern
>
> **Prefer deterministic literal matching whenever possible.**

> ### Rule 04 — PCRE
>
> **Use PCRE only when simpler matching cannot satisfy the requirement.**

> ### Rule 05 — Deployment
>
> **A custom signature must be deployed through an IPS sensor/profile and then applied to the relevant firewall policy.**

> ### Rule 06 — Encryption
>
> **Encrypted payload inspection requires appropriate traffic visibility.**

> ### Rule 07 — Rate
>
> **Rate + Track is powerful for detecting repeated behavior such as brute force.**

> ### Rule 08 — Search Range
>
> **Distance controls the relative search start; within controls the search window.**

> ### Rule 09 — State
>
> **Tags allow multi-stage signatures to maintain state across packets in a session.**

> ### Rule 10 — Performance
>
> **IPS tuning must consider CPU, memory, engine count, buffers, acceleration, sessions, and traffic volume together.**

---

# 🧪 42. Production Readiness Checklist

Before deploying a custom IPS signature:

* [ ] Requirement documented
* [ ] Attack behavior understood
* [ ] Protocol identified
* [ ] Service identified
* [ ] Flow identified
* [ ] Inspection context identified
* [ ] Pattern minimized and strengthened
* [ ] PCRE avoided unless necessary
* [ ] Search range minimized
* [ ] Rate logic validated
* [ ] Tracking behavior validated
* [ ] Tags validated if required
* [ ] Attack ID documented
* [ ] Signature name follows standard
* [ ] Action documented
* [ ] IPS sensor updated
* [ ] Firewall policy verified
* [ ] SSL inspection verified where required
* [ ] Legitimate traffic tested
* [ ] Attack traffic tested
* [ ] False positives tested
* [ ] CPU impact measured
* [ ] Memory impact measured
* [ ] Logs verified
* [ ] Rollback plan prepared
* [ ] Change documented

---

# 🔐 43. Final Engineering Principle

> **The best IPS signature is not the most complex signature. It is the signature that detects the intended behavior with the smallest possible inspection scope, lowest computational cost, and lowest false-positive rate.**

```text
Specific Service
      +
Correct Context
      +
Strong Pattern
      +
Minimal Range
      +
Correct Flow
      +
Controlled PCRE
      ↓
High-Quality IPS Signature
```

### Final Mental Model

```text
Decode
  ↓
Identify
  ↓
Match
  ↓
Correlate
  ↓
Decide
  ↓
Log / Block
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

## 🔖 Topics

```text
fortigate
fortios
fortinet
fortigate-ips
fortios-ips
ips
intrusion-prevention
ips-signature
custom-ips-signature
network-security
cybersecurity
nse4
nse7
fortinet-nse
security-engineering
network-security-engineering
pcap
pcre
threat-detection
```

---

## 🔎 Keywords

```text
FortiGate IPS
FortiOS IPS
FortiGate custom IPS signature
Fortinet custom IPS signature
FortiGate IPS troubleshooting
FortiGate IPS performance tuning
FortiOS F-SBID
FortiGate PCRE
FortiGate IPS rate based detection
FortiGate IPS signature
FortiGate IPS protocol decoder
FortiGate IPS hold time
FortiGate IPS hardware acceleration
FortiGate IPS engine count
FortiGate IPS fail open
FortiGate IPS socket size
Fortinet NSE IPS
FortiGate NSE4 IPS
FortiGate NSE7 IPS
```

---

> **SheynShield — Engineering Secure Networks**
>
> Practical FortiGate knowledge for **network security engineers, Fortinet administrators, NSE candidates, SOC teams, and security architects.**

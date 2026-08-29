# 🛡️ FortiGate IPS — Engineering & Tuning  

> **FortiOS IPS | Custom Signatures | Protocol Decoders | Rate-Based Detection | Hold Time | Acceleration | Performance Tuning**

---

## 📌 1. IPS Architecture — Big Picture

FortiGate IPS is not simply a signature-matching engine.

The IPS engine can combine:

* **Signature matching**
* **Protocol decoders**
* **Pattern matching**
* **PCRE**
* **Heuristics**
* **Threat intelligence**
* **Anomaly detection**
* **Application awareness**
* **CVE/security intelligence**
* **Rate-based detection**
* **Session tracking**
* **Custom signatures**

### Simplified inspection flow

```text
                Incoming Traffic
                       │
                       ▼
              ┌─────────────────┐
              │ Protocol Decoder│
              └────────┬────────┘
                       │
             Identify protocol
                       │
                       ▼
              ┌─────────────────┐
              │ Service Tree    │
              │ HTTP / DNS /    │
              │ SMTP / SSL ...  │
              └────────┬────────┘
                       │
                       ▼
             ┌──────────────────┐
             │ IPS Signatures   │
             │ Pattern / PCRE   │
             └────────┬─────────┘
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
    Static Match            Behavioral /
    Signature               Rate Detection
          │                       │
          └───────────┬───────────┘
                      ▼
                 IPS Action
            ┌────────┼─────────┐
            ▼        ▼         ▼
          Allow     Log       Block
```

---

# 🔎 2. IPS Deep Packet Inspection

IPS requires sufficient visibility into the traffic.

Depending on the protocol and inspection method, the engine can inspect:

* Packet headers
* Protocol fields
* Payload
* Application-layer fields
* HTTP URI
* HTTP headers
* HTTP methods
* HTTP cookies
* HTTP body
* Files
* Email content
* Protocol-specific structures

### Example

An HTTP attack may not be identifiable only from:

```text
Source IP
Destination IP
Destination Port
```

The IPS engine may need to inspect:

```text
HTTP
 ├── Method
 ├── Host
 ├── URI
 ├── Header
 ├── Cookie
 └── Body
```

---

# 🧠 3. Protocol Decoder vs Signature

One of the most important IPS concepts:

> **The protocol decoder identifies the protocol structure first; signatures are then evaluated in the appropriate inspection context.**

Example:

```text
TCP/443
   │
   ▼
Protocol / SSL detection
   │
   ▼
SSL service tree
   │
   ▼
Relevant IPS signatures
```

Therefore, using the correct `--service` is usually better than relying only on a port.

---

# 🧩 4. Custom IPS Signature — Basic Structure

Fortinet custom signatures use the `F-SBID` format.

### Basic example

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

### Important

`F-SBID` must be written in uppercase.

```text
F-SBID(...)
```

---

# 🛠️ 5. Create Custom IPS Signature

CLI:

```bash
config ips custom
    edit "ips-cus-test"
        set signature 'F-SBID( --name "Block.SMTP.VRFY.CMD"; --protocol tcp; --service SMTP; --pattern "vrfy"; --no_case; --context header; )'
    next
end
```

### Recommended signature metadata

For FortiManager / centralized management, use an explicit `attack_id`.

```text
F-SBID(
    --attack_id 8151;
    --name "Example.Custom.Attack";
    --protocol tcp;
    --service HTTP;
    --flow from_client;
    --pattern "malicious";
    --context uri;
)
```

> **Best practice:** Keep custom attack IDs unique within your organization's custom-signature namespace.

---

# 🎯 6. Signature Definition — Mandatory Concepts

A custom IPS signature should normally define the following:

| Component     | Purpose                     |
| ------------- | --------------------------- |
| `--name`      | Signature name              |
| `--protocol`  | Transport protocol          |
| `--service`   | Application/service tree    |
| `--flow`      | Traffic direction           |
| `--pattern`   | Literal pattern             |
| `--context`   | Inspection buffer           |
| `--pcre`      | Regex-based matching        |
| `--attack_id` | Custom signature identifier |
| `--weight`    | Signature priority          |

---

# 🏷️ 7. Signature Name Rules

Example:

```text
--name "IBM.Domino.iNotes.Foldername.Buffer.Overflow";
```

Rules:

* Maximum length: **64 characters**
* Use double quotation marks
* Signature name must be unique
* Periods can be used instead of spaces

### Recommended naming convention

```text
vendor.product.component.vulnerability
```

Example:

```text
IBM.Domino.iNotes.Foldername.Buffer.Overflow
```

---

# 🌐 8. Service vs Port — VERY IMPORTANT

Prefer:

```text
--service HTTP;
```

over:

```text
--dst_port 80;
```

when the IPS engine supports the relevant service.

### Why?

FortiGate IPS uses service trees.

For example:

```text
HTTP
SMTP
DNS
POP3
SSL
SSH
...
```

A packet identified as HTTP is evaluated against the HTTP service tree.

Therefore:

```text
--service HTTP
```

is generally more appropriate than:

```text
--dst_port 80
```

for an HTTP-specific signature.

### Service-tree concept

```text
                 IPS Engine
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
         HTTP       DNS        SMTP
          │          │          │
       HTTP DB     DNS DB     SMTP DB
```

### Unknown service

If a signature has:

* `--service` → assigned to that service tree
* only port → unknown-service tree
* neither service nor port → generic signature

Generic signatures can therefore have a broader inspection scope.

> **Engineering rule:** Avoid unnecessarily broad signatures because they can increase CPU consumption and false positives.

---

# 🔄 9. Flow Direction

### Client → Server

```text
--flow from_client;
```

Matches traffic sent from the client toward the server.

### Server → Client

```text
--flow from_server;
```

Matches traffic sent from the server toward the client.

### Both directions

```text
--flow bi_direction;
```

Example:

```text
F-SBID(
    --name "Example.TCP.Signature";
    --protocol tcp;
    --flow bi_direction;
)
```

### Reversed

`reversed` is primarily used to describe the attack location correctly in the GUI.

A classic case is brute-force detection where the interesting packet is:

```text
Server → Client
```

because the server reports a failed authentication attempt.

---

# 🔐 10. HTTP Host Custom Signature

Example:

```text
F-SBID(
    --attack_id 1468;
    --name "block-iran-gov-fqdn";
    --pattern "iran.gov.ir";
    --service HTTP;
    --no_case;
    --flow from_client;
    --context host;
)
```

### HTTPS / encrypted traffic

A critical distinction:

> A signature cannot inspect an encrypted hostname simply because the traffic uses TCP/443.

If the required application-layer data is encrypted, the FortiGate must have the necessary visibility.

Depending on the scenario, this may involve:

* SSL inspection
* Protocol identification
* SNI/host visibility
* Appropriate IPS inspection context

Do **not** blindly assume that `HTTP`, `SSL`, `HTTPS`, and TCP/443 are interchangeable IPS service definitions.

---

# 🧱 11. Pattern Matching

Basic pattern:

```text
--pattern "vrfy";
```

The pattern must:

* Be enclosed in `" "`
* Be followed by `;`

### Case-sensitive

```text
--pattern "Admin";
```

### Case-insensitive

```text
--pattern "Admin";
--no_case;
```

Result:

```text
Admin
admin
ADMIN
AdMiN
```

---

# ⚠️ 12. Pattern Escaping

Special characters must be escaped according to Fortinet IPS custom-signature syntax.

Examples of special characters include:

```text
"
;
\
|
:
```

Hexadecimal representation can be used where appropriate.

Example concept:

```text
|22|
|3b|
|5c|
|7c|
|3a|
```

### Recommendation

Prefer explicit hexadecimal encoding for problematic special characters rather than relying on complicated escaping.

---

# 🧪 13. PCRE

PCRE provides regular-expression matching:

```text
--pcre "/admin/i";
```

Example:

```text
F-SBID(
    --name "Example.PCRE";
    --protocol tcp;
    --service HTTP;
    --flow from_client;
    --pcre "/admin/i";
    --context uri;
)
```

### PCRE is powerful — but expensive

> **PCRE is generally less efficient than simple pattern matching.**

Therefore:

```text
Simple pattern
      ↓
Prefer this

PCRE
      ↓
Use when required
```

Avoid unnecessarily complex expressions such as:

```text
.*
.+ 
```

when a deterministic pattern can solve the problem.

---

# 🚨 14. PCRE Performance Rules

### Avoid

```text
--pcre "/.*/";
```

### Prefer

```text
--pattern "specific-value";
```

when possible.

### Rules

* Use the simplest matching engine possible.
* Use `--pattern` before PCRE when sufficient.
* Avoid catastrophic/backtracking-heavy expressions.
* Avoid unnecessary wildcards.
* Limit the inspection context.
* Use range modifiers when they genuinely reduce the search area.

> **A custom signature that works in a lab can still become a production performance problem at scale.**

---

# 📏 15. Pattern Length

Searching for extremely short patterns is inefficient.

### Avoid

```text
--pattern "GET";
```

if the pattern is not sufficiently discriminating.

Prefer longer and more unique patterns:

```text
--pattern "GET /admin/login";
```

where appropriate.

### Rule of thumb

> Use the **longest practical unique pattern** while keeping the signature resilient to legitimate variations.

---

# 🧭 16. Context

Context tells the IPS engine **where to search**.

Common contexts include:

```text
uri
header
host
body
file
packet
packet_origin
```

### Examples

HTTP URI:

```text
--context uri;
```

HTTP header:

```text
--context header;
```

HTTP host:

```text
--context host;
```

File:

```text
--context file;
```

---

# 📦 17. File Context

For FTP:

```text
--context file;
```

For HTTP, file/body contexts may represent the relevant transferred content depending on protocol decoding.

Email protocols support MIME parsing.

Common protocols:

```text
SMTP
IMAP
POP3
NNTP
```

Common encodings may be decoded, including:

```text
Base64
UUEncode
7bit
8bit
Quoted-Printable
```

### Important limitation

If a file is compressed or archived, do not assume the IPS engine will automatically decompress every archive format.

---

# 📍 18. Packet-Based Inspection

FortiGate IPS supports packet-based inspection for many packet/header-related keywords.

Conceptually:

```text
Ethernet Header
       │
IP Header
       │
TCP/UDP Header
       │
Payload
       │
Trailer
```

If no specific range modifier is used, matching can occur across the applicable inspection buffer.

---

# 📏 19. Range Modifiers

| Modifier       | Purpose                                   |
| -------------- | ----------------------------------------- |
| `distance`     | Start searching after a relative distance |
| `within`       | Limit search range                        |
| `distance_abs` | Absolute distance reference               |
| `within_abs`   | Absolute search range                     |
| `offset`       | Start from an offset                      |
| `depth`        | Limit search depth                        |
| `isdataat`     | Verify data exists at an offset           |
| `pcre`         | Regular-expression matching               |

---

# 🎯 20. Distance + Within

Example:

```text
--pattern "login";
--distance 10;
--within 20;
--pattern "fail";
```

Conceptually:

```text
last reference
     │
     │<------ 10 bytes ------>│
                             login
                               │
                               │<------ 20 bytes ------>│
                                                       fail
```

### Important

`distance` determines **where searching starts**.

`within` determines **how far the engine searches**.

---

# 🔗 21. Reference Point

Possible reference concepts include:

```text
match
packet
context
reverse
lasttag
```

### Default

If no reference is specified, the previous match is normally used where applicable.

### First pattern

If you use `distance` / `within` with the first pattern, there is no previous match.

Therefore specify the appropriate reference, commonly:

```text
--within 100,context;
```

---

# ⚠️ 22. Distance / Within Performance

Be careful when combining:

```text
distance
within
distance_abs
within_abs
```

Use matching pairs:

```text
distance + within
```

or:

```text
distance_abs + within_abs
```

Avoid unnecessarily broad ranges.

### Negative matching

For negative patterns:

```text
--pattern !"|0a|";
```

absolute range modifiers may be useful in specific cases.

---

# 🚫 23. Deprecated Signature Keywords

Some legacy keywords remain for backward compatibility.

Examples include:

```text
uri
header
body
content
raw
offset
offset_abs
depth
depth_abs
```

For new signatures:

> Prefer the current `--context` model rather than deprecated context keywords.

---

# 🧮 24. `byte_test`

`byte_test` checks a numeric value in packet data.

Concept:

```text
--byte_test <bytes>,<operator>,<value>,<offset>;
```

Example:

```text
--byte_test 2,>,100,0;
```

Meaning:

> Read 2 bytes and alert if the value is greater than 100.

Useful for:

* Protocol fields
* Length validation
* Numeric protocol values
* Malformed packet detection

---

# 🚀 25. `byte_jump`

`byte_jump` changes the reference point based on a value found in the packet.

Example:

```text
--byte_jump 4,0,relative;
```

Conceptually:

```text
Read numeric field
      │
      ▼
Calculate jump
      │
      ▼
Move reference point
      │
      ▼
Continue inspection
```

---

# 🔢 26. Big-Endian vs Little-Endian

Example bytes:

```text
AA BB CC DD
```

Big-endian interpretation:

```text
AA BB CC DD
```

Little-endian interpretation:

```text
DD CC BB AA
```

Always verify the protocol specification before choosing the byte order.

---

# 📐 27. `align`

`align` rounds converted byte counts to a 32-bit boundary when used with `byte_jump`.

Concept:

```text
7 bytes → 8 bytes
```

```text
33 bytes → 36 bytes
```

Use only when required by the protocol structure.

---

# 🧩 28. Parsed Type

`--parsed_type` allows matching against a specific parsed application/protocol structure.

Example:

```text
--parsed_type HTTP-GET;
```

This can be useful when an application contains multiple similar protocol elements and the signature should target a particular parsed type.

---

# 📊 29. Rate-Based IPS Detection

Rate detection is useful for identifying behavior such as:

* Brute force
* Credential attacks
* Excessive requests
* Abnormal repeated traffic
* Protocol abuse

Concept:

```text
--rate <count>,<duration>[,<limit>];
```

Example:

```text
3 events / 30 seconds
```

---

# 👤 30. Track-Based Detection

Possible tracking dimensions include:

```text
src_ip
dst_ip
DHCP client
DNS domain
DNS domain + IP
```

Concept:

```text
User A
  │
  ├── Request 1
  ├── Request 2
  ├── Request 3
  └── Request 4
        │
        ▼
   Threshold reached
        │
        ▼
      Action
```

### Why tracking matters

Without a tracking key, matching traffic may contribute collectively to the rate counter.

With tracking:

```text
Track = src_ip
```

the engine can maintain counters per source.

---

# ⏱️ 31. Continuous vs Periodical Rate Mode

### Continuous

```text
set rate-mode continuous
```

Useful when the threshold should represent continuing suspicious behavior.

Typical examples:

* Brute force
* Repeated credential failures
* Persistent attack behavior

Concept:

```text
Attack
  ↓
Counter increases
  ↓
Threshold reached
  ↓
Action
```

The counter does not simply reset after every enforcement interval in the same way as periodical behavior.

---

### Periodical

Used for periodic threshold evaluation.

Example:

```text
4 attempts
within
30 seconds
```

After the defined period, the counter can reset for a new evaluation period.

Useful for:

* Crawlers
* DNS patterns
* Mail traffic
* APIs
* Repeated access attempts

---

# 🧨 32. Rate + Quarantine

When using rate detection with quarantine, understand the enforcement behavior and timers carefully.

```text
Detection
    ↓
Threshold reached
    ↓
Quarantine / enforcement
    ↓
Timer
    ↓
Client can retry
```

Always validate the resulting behavior in the target FortiOS release.

---

# 🏷️ 33. IPS Tags

Tags allow signatures to communicate state across packets in a session.

Example:

```text
--tag pset,Tag.Rsync.Argument;
```

Then:

```text
--tag test,Tag.Rsync.Argument;
```

### Operations

| Operation | Meaning                              |
| --------- | ------------------------------------ |
| `pset`    | Set tag and remember reference point |
| `toggle`  | Toggle tag state                     |
| `test`    | Test whether tag exists              |
| `test,!`  | Test that tag does not exist         |

Example:

```text
--tag pset,Tag.Login;
```

```text
--tag test,Tag.Login;
```

```text
--tag test,!Tag.Login;
```

### Why tags?

Useful when:

```text
Packet 1
   ↓
Detect stage 1
   ↓
Set tag
   ↓
Packet 2
   ↓
Check tag
   ↓
Detect stage 2
```

This is particularly useful for multi-stage attack patterns.

---

# ⚖️ 34. Signature Weight

Weight influences signature priority.

```text
--weight 40;
```

Range:

```text
0 - 255
```

Higher weight can give a signature priority over a lower-weight signature where applicable.

### Recommended custom-signature range

```text
20 - 50
```

Typical concept:

```text
Normal signatures → lower weight
Custom important signatures → 20–50
Botnet signatures → potentially much higher
```

---

# 🧪 35. Example — Windows Signature

```text
F-SBID(
    --attack_id 8151;
    --vuln_id 8151;
    --name "windows.nt.5.web.surfing";
    --default_action drop_session;
    --service HTTP;
    --protocol tcp;
    --flow from_client;
    --pattern !"fct";
    --pattern "windows nt 5.1";
    --no_case;
    --context header;
    --weight 40;
)
```

---

# 🌐 36. Malicious URL / Drive-by Protection

Drive-by attacks can abuse compromised or malicious websites to deliver:

* JavaScript
* Executables
* Exploit code
* Redirects
* Malicious downloads

FortiGate IPS can use its malicious-URL intelligence for additional protection where supported.

CLI concept:

```bash
config ips sensor
    edit "ips-test"
        set block-malicious-url enable
    next
end
```

> **Model-specific support and FortiOS behavior must be verified for the target appliance/release.**

---

# 🧬 37. Drive-by Detection Concept

```text
User
 │
 ▼
Malicious / Compromised Website
 │
 ▼
HTTP Response
 │
 ├── Redirect
 ├── JavaScript
 ├── Exploit
 └── Malicious Download
        │
        ▼
      IPS
        │
   ┌────┴────┐
   ▼         ▼
 Detect     Block
```

---

# 🛡️ 38. Deploy Custom Signature into IPS Profile

Creating the signature is **not enough**.

The signature must be referenced by an IPS sensor/profile.

Concept:

```text
Custom Signature
       │
       ▼
IPS Sensor / Profile
       │
       ▼
Firewall Policy
       │
       ▼
Traffic
```

### GUI workflow

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

# 🔥 39. Firewall Policy Deployment

Typical design:

```text
LAN
 │
 ▼
Firewall Policy
 │
 ├── Proxy / Flow inspection
 ├── Deep Inspection where required
 └── IPS Profile
       │
       ▼
    Internet
```

For encrypted traffic requiring payload inspection:

```text
SSL/TLS
   ↓
Inspection
   ↓
Decryption / visibility where configured
   ↓
IPS
```

---

# 🧩 40. IPS Filter Logic

When multiple filter attributes are configured in the same filter, they generally narrow the match.

Example:

```text
Server
AND
TCP
AND
Medium → High severity
```

Conceptually:

```text
Severity = Medium..High
      AND
Protocol = TCP
      AND
Role = Server
```

### Better profile design

Instead of one excessively broad filter:

```text
Role 1 → High severity
Role 2 → Medium severity
Role 3 → ...
```

This gives more granular policy control.

---

# ⏳ 41. IPS Signature Hold Time

FortiGate can temporarily hold newly introduced/updated IPS signatures.

Concept:

```text
FortiGuard
    │
    ▼
New / Updated Signature
    │
    ▼
Hold Period
    │
    ├── Monitor / Validate
    │
    ▼
Signature becomes active
```

CLI:

```bash
config system ips
    set signature-hold-time 3d12h
end
```

### Disable hold time

```bash
config system ips
    set signature-hold-time 0
end
```

---

# 🧠 42. Why Hold Time Exists?

A new signature may require additional validation before becoming fully active.

The operational idea is:

```text
New signature
     ↓
Behavior / validation period
     ↓
Potential false-positive analysis
     ↓
Updated signature intelligence
     ↓
Production enforcement
```

This can be useful for organizations that prioritize stability and controlled signature deployment.

---

# ⚠️ 43. Hold Time ≠ Protection Pause

A common operational mistake is assuming:

> "If new signatures are held, IPS is disabled."

That is incorrect.

Existing active signatures and other security controls continue to provide protection.

During a hold period, maintain additional defenses such as:

* Existing IPS signatures
* Custom IPS signatures
* DoS policies
* Rate-based protection
* Firewall policies
* Web filtering
* Application control
* Other security profiles

---

# 🧰 44. Override Signature Hold by ID

CLI:

```bash
config system ips
    set override-signature-hold-by-id enable
end
```

This provides more granular control over signature hold behavior.

Useful when a specific signature needs exceptional handling.

---

# 🔍 45. Troubleshoot Signature Hold

Example diagnostic:

```bash
diagnose ips signature on-hold vd1
```

Specific signature:

```bash
diagnose ips signature on-hold vd1 29844
```

Use these commands when investigating why a signature has not yet entered normal enforcement.

---

# ⚡ 46. IPS Hardware Acceleration

FortiGate can use hardware acceleration for supported IPS workloads.

Conceptual components include:

```text
CPU
 │
 ├── IPS engine
 │
 └── CP acceleration
        │
        └── Pattern matching
```

Depending on the platform:

* CPU
* CP
* NP
* IPSA / acceleration mechanisms

may participate in processing.

---

# ⚙️ 47. IPS Acceleration Mode

CLI:

```bash
config ips global
    set np-accel-mode basic
    set cp-accel-mode basic
end
```

Possible CP acceleration modes can include:

```text
none
basic
advanced
```

### `none`

```text
Main CPU
   ↓
IPS processing
```

Useful primarily for troubleshooting or environments where acceleration is intentionally disabled.

### `basic`

Provides supported acceleration for common workloads.

### `advanced`

Designed for higher-performance environments and more advanced pattern-matching workloads where supported by the platform.

> **Do not select an acceleration mode solely from a generic model list. Hardware capabilities are platform-specific.**

---

# 🧠 48. IPS Database

FortiGate IPS signatures can be provided through different database configurations depending on the FortiGate platform.

Conceptually:

```text
Regular / Slim database
        vs
Extended database
```

CLI example:

```bash
config ips global
    set database extended
end
```

### Engineering consideration

A larger signature database can provide broader coverage but may consume more resources.

Always balance:

```text
Detection Coverage
       ↕
Memory / CPU
       ↕
Throughput
```

---

# 🏭 49. Industrial IPS Signatures

Industrial / OT signatures may be excluded depending on the configuration.

To include them:

```bash
config ips global
    set exclude-signature none
end
```

### OT security principle

Before enabling large industrial signature sets:

```text
Inventory
   ↓
Identify OT protocols
   ↓
Understand traffic patterns
   ↓
Enable required signatures
   ↓
Monitor
   ↓
Tune false positives
```

---

# 🧠 50. IPS Engine Count

The IPS engine can use multiple parallel processing instances.

Example:

```bash
config ips global
    set engine-count 0
end
```

`0` allows FortiOS to optimize the value according to the platform.

### Concept

```text
Traffic
  │
  ├── IPS Engine 1
  ├── IPS Engine 2
  ├── IPS Engine 3
  └── IPS Engine N
```

More engines do **not** automatically mean better performance.

The correct value depends on:

* CPU
* Memory
* Platform
* Traffic volume
* Concurrent sessions
* Inspection workload

---

# 💾 51. IPS Socket Size / Buffer

Example:

```bash
config ips global
    set socket-size 128
end
```

The socket buffer controls how much data the kernel provides to the IPS engine during sampling/processing.

### Too small

```text
Small buffer
    ↓
IPS engine overloaded more easily
    ↓
Higher chance of fail-open events
```

### Too large

```text
Large buffer
    ↓
Higher memory consumption
    ↓
Potential conserve-mode pressure
```

### Golden rule

> **Do not blindly increase socket-size.**

Tune it together with:

* Memory utilization
* IPS engine load
* Concurrent sessions
* Traffic rate
* Fail-open events

---

# 🚨 52. IPS Fail-Open

Fail-open determines what happens when the IPS processing path cannot continue normally because of resource pressure or overload.

Example:

```bash
config ips global
    set fail-open disable
end
```

### Concept

```text
IPS overloaded
      │
      ├── Fail-open enabled
      │        ↓
      │     Forward traffic
      │
      └── Fail-open disabled
               ↓
            Traffic may be dropped
```

### Security vs Availability

```text
Fail-open
   ↑
Availability

Fail-closed
   ↑
Security enforcement
```

Choose according to business requirements and risk tolerance.

---

# ⚠️ 53. Fail-Open ≠ Hardware Acceleration Overload

Do not confuse:

```text
IPS fail-open
```

with:

```text
NP / CP / IPS acceleration overload
```

They are related to different processing/resource conditions.

Always inspect the actual bottleneck before changing fail-open behavior.

---

# 📈 54. IPS Session Count Accuracy

IPS resource/session handling can use optimized heuristics rather than consuming the absolute maximum theoretical resource capacity.

Conceptually:

```text
Maximum resource
       │
       ▼
Safety / heuristic margin
       │
       ▼
Operational IPS capacity
```

An **accurate** mode can use resource limits more aggressively, but this should be evaluated carefully under high-load conditions.

### Troubleshooting principle

Check:

```text
CPU
Memory
IPS engines
Sessions
Buffer usage
Acceleration
Conserve mode
Fail-open events
```

before changing resource-related parameters.

---

# 🔬 55. Protocol Decoder

Protocol decoders identify malformed or abnormal protocol behavior.

Example:

```text
HTTP Decoder
     ↓
Does packet follow HTTP protocol requirements?
     │
     ├── YES → Continue inspection
     │
     └── NO  → Decoder anomaly
```

This allows IPS to detect protocol violations even when no traditional exploit signature directly matches.

---

# 🌐 56. DNS Decoder Example

Some FortiOS versions/platforms allow decoder-specific port configuration.

Concept:

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

### Important

Some decoders operate in an automatic mode and cannot simply be assigned arbitrary individual ports.

Always verify decoder behavior for the exact FortiOS release.

---

# 🔐 57. Custom Signature — Host Blocking Example

### HTTP

```text
F-SBID(
    --attack_id 1468;
    --name "block.iran.gov.fqdn";
    --pattern "iran.gov.ir";
    --service HTTP;
    --no_case;
    --flow from_client;
    --context host;
)
```

### Important HTTPS consideration

For HTTPS, distinguish between:

```text
TCP/443
```

and:

```text
Decrypted HTTP content
```

If the hostname is visible only through TLS metadata, use the appropriate FortiOS-supported inspection mechanism rather than assuming an HTTP host context will inspect encrypted payload.

---

# 🧨 58. Brute-Force Detection Design

A strong brute-force IPS strategy can combine:

```text
Signature
   +
Rate
   +
Track
   +
Action
```

Example design:

```text
Authentication failure
       ↓
Signature match
       ↓
Track source IP
       ↓
3 events / 30 seconds
       ↓
Threshold reached
       ↓
Block / Quarantine
```

---

# 🎯 59. Rate-Based Brute Force

Conceptual configuration:

```text
Threshold = 3
Duration  = 30 seconds
Track     = Source IP
```

Meaning:

> Detect repeated matching events from the same source within the defined time window.

This is generally more meaningful than simply matching one authentication failure.

---

# 🧪 60. Custom Signature Testing Checklist

Before production deployment:

* [ ] Signature syntax validated
* [ ] Unique attack ID
* [ ] Unique signature name
* [ ] Correct protocol
* [ ] Correct service
* [ ] Correct flow direction
* [ ] Correct context
* [ ] Pattern sufficiently specific
* [ ] `--no_case` used only when required
* [ ] PCRE avoided unless necessary
* [ ] Range modifiers minimized
* [ ] False positives tested
* [ ] Legitimate traffic tested
* [ ] Performance tested
* [ ] Logging enabled
* [ ] Correct action selected
* [ ] Signature added to IPS sensor
* [ ] IPS sensor attached to firewall policy
* [ ] Required SSL inspection configured

---

# 🚀 61. IPS Performance Optimization Checklist

### Prefer

```text
Specific service
        ↓
Specific context
        ↓
Longer unique pattern
        ↓
Minimal search range
        ↓
Simple pattern engine
```

### Avoid

```text
Generic signature
        +
No context
        +
Short pattern
        +
Broad range
        +
Complex PCRE
```

This combination can significantly increase the inspection workload.

---

# 🧭 62. IPS Troubleshooting Flow

```text
Signature not matching
        │
        ▼
Is IPS profile attached?
        │
        ▼
Is policy inspecting traffic?
        │
        ▼
Is required SSL visibility available?
        │
        ▼
Correct protocol?
        │
        ▼
Correct service tree?
        │
        ▼
Correct flow?
        │
        ▼
Correct context?
        │
        ▼
Pattern actually present?
        │
        ▼
Signature on hold?
        │
        ▼
Signature filtered?
        │
        ▼
Rate threshold reached?
        │
        ▼
Action configured?
```

---

# 🧠 63. Most Important NSE-Level Rules

> ### Rule #1
>
> **`--service` is generally more meaningful than simply matching a TCP port when the protocol/service is known.**

> ### Rule #2
>
> **Use the most specific context possible.**

> ### Rule #3
>
> **Prefer simple `--pattern` matching over PCRE whenever possible.**

> ### Rule #4
>
> **PCRE can become a performance problem when used carelessly.**

> ### Rule #5
>
> **Creating a custom signature does not activate it. Add it to an IPS sensor/profile and attach that profile to the firewall policy.**

> ### Rule #6
>
> **Encrypted traffic requires appropriate visibility before payload-based signatures can inspect it.**

> ### Rule #7
>
> **Rate + Track is powerful for detecting repeated behavior such as brute force.**

> ### Rule #8
>
> **`distance` defines where the search begins; `within` limits the search window.**

> ### Rule #9
>
> **Tags allow multi-packet / multi-stage signatures to maintain state across a session.**

> ### Rule #10
>
> **Performance tuning must consider CPU, memory, engine count, socket size, acceleration and traffic volume together.**

---

# 🏗️ 64. Production IPS Design Pattern

```text
                    Internet
                       │
                       ▼
              ┌─────────────────┐
              │    FortiGate    │
              │                 │
              │ SSL Inspection  │
              │       ↓         │
              │ Protocol Decode │
              │       ↓         │
              │ IPS Engine       │
              │  ├─ Signatures   │
              │  ├─ PCRE         │
              │  ├─ Rate         │
              │  ├─ Decoder      │
              │  └─ Intelligence │
              └────────┬────────┘
                       │
                       ▼
                    LAN
```

### Defense layers

```text
Firewall Policy
      +
SSL Inspection
      +
IPS
      +
Application Control
      +
Web Filtering
      +
DNS Security
      +
DoS Protection
      +
Sandbox / Malware Protection
```

> **IPS should be treated as one layer of defense, not the entire security architecture.**

---

# 📝 65. Quick CLI Reference

### Custom IPS signature

```bash
config ips custom
    edit "custom-signature"
        set signature 'F-SBID( --name "Example.Custom"; --protocol tcp; --service HTTP; --flow from_client; --pattern "malicious"; --context uri; )'
    next
end
```

### IPS global

```bash
config ips global
    set engine-count 0
    set socket-size 128
    set fail-open disable
end
```

### IPS database

```bash
config ips global
    set database extended
end
```

### Industrial signatures

```bash
config ips global
    set exclude-signature none
end
```

### Hold time

```bash
config system ips
    set signature-hold-time 3d12h
    set override-signature-hold-by-id enable
end
```

### Malicious URL

```bash
config ips sensor
    edit "ips-test"
        set block-malicious-url enable
    next
end
```

---

# 🔥 66. IPS Quick  Table

| Requirement                  | Recommended Direction                 |
| ---------------------------- | ------------------------------------- |
| Known attack string          | `--pattern`                           |
| Complex pattern              | `--pcre`                              |
| HTTP URL detection           | `--context uri`                       |
| HTTP Host detection          | `--context host`                      |
| HTTP header detection        | `--context header`                    |
| File detection               | `--context file`                      |
| Client → Server              | `--flow from_client`                  |
| Server → Client              | `--flow from_server`                  |
| Both directions              | `--flow bi_direction`                 |
| Repeated attack              | `--rate`                              |
| Per-user/source tracking     | `--track`                             |
| Multi-stage attack           | `--tag`                               |
| Numeric protocol field       | `--byte_test`                         |
| Dynamic reference movement   | `--byte_jump`                         |
| Search window                | `distance` + `within`                 |
| Signature priority           | `--weight`                            |
| Newly introduced signatures  | Hold Time                             |
| High IPS throughput          | Hardware acceleration where supported |
| IPS resource pressure        | Tune engine/buffer carefully          |
| Encrypted payload inspection | Appropriate SSL visibility            |

---

# 🎓 67. Exam Memory Map

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

## ⚡ One-Minute Revision

```text
Service ≠ Port
Context = Where to inspect
Pattern = Fast/simple matching
PCRE = Powerful but expensive
Flow = Direction
Rate = Frequency
Track = Who/what is counted
Tag = Multi-stage session state
Distance = Where search starts
Within = Search window
Byte_Test = Compare numeric field
Byte_Jump = Move reference point
Weight = Signature priority
Decoder = Protocol correctness
Hold Time = Controlled signature activation
Socket Size = IPS buffering
Engine Count = Parallel IPS processing
Fail-Open = Availability vs enforcement
```

---

## 🔐 Final Engineering Principle

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
High-quality IPS Signature
```

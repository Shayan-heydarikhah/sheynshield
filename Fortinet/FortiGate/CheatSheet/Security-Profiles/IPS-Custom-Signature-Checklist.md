# 🔗 SheynShield Resources

# FortiGate Custom IPS Signature — Checklist

> **FortiOS / IPS Engine:** Custom IPS Signature / F-SBID
> **Focus:** IPS, F-SBID, Pattern, PCRE, Context, Range, Byte Operations, Tags, Rate, Track & Deployment
> **Level:** NSE 4 → NSE 7
> **Brand:** SheynShield — Engineering Secure Networks

---

## 📋 Table of Contents

* [1. Signature Architecture Checklist](#1-signature-architecture-checklist)
* [2. F-SBID Structure Checklist](#2-f-sbid-structure-checklist)
* [3. Signature Identity Checklist](#3-signature-identity-checklist)
* [4. Protocol and Traffic Checklist](#4-protocol-and-traffic-checklist)
* [5. Pattern Matching Checklist](#5-pattern-matching-checklist)
* [6. PCRE Checklist](#6-pcre-checklist)
* [7. Context Checklist](#7-context-checklist)
* [8. Range Modifier Checklist](#8-range-modifier-checklist)
* [9. Reference Point Checklist](#9-reference-point-checklist)
* [10. Byte Test and Byte Jump Checklist](#10-byte-test-and-byte-jump-checklist)
* [11. Parsed Type Checklist](#11-parsed-type-checklist)
* [12. Protocol Header Checklist](#12-protocol-header-checklist)
* [13. Rate and Track Checklist](#13-rate-and-track-checklist)
* [14. Session Tag Checklist](#14-session-tag-checklist)
* [15. Weight and ID Checklist](#15-weight-and-id-checklist)
* [16. Custom IPS Deployment Checklist](#16-custom-ips-deployment-checklist)
* [17. Traffic Shaping Checklist](#17-traffic-shaping-checklist)
* [18. Signature Troubleshooting Checklist](#18-signature-troubleshooting-checklist)
* [19. False Positive Checklist](#19-false-positive-checklist)
* [20. Performance Optimization Checklist](#20-performance-optimization-checklist)
* [21. Production Readiness Checklist](#21-production-readiness-checklist)
* [22. NSE Memory Checklist](#22-nse-memory-checklist)
* [23. Quick Reference](#23-quick-reference)
* [24. 30-Second Interview Answer](#24-30-second-interview-answer)

---

# 1. Signature Architecture Checklist

## Detection Pipeline

* [ ] Identify exactly what attack or behavior must be detected.
* [ ] Identify the protocol.
* [ ] Identify the application/service.
* [ ] Identify the traffic direction.
* [ ] Select the narrowest possible inspection context.
* [ ] Determine whether a simple pattern is sufficient.
* [ ] Use PCRE only when simple pattern matching cannot provide sufficient accuracy.
* [ ] Use range modifiers when the location of the second match matters.
* [ ] Use `byte_test` for numeric comparisons.
* [ ] Use `byte_jump` when the protocol contains dynamic offsets.
* [ ] Use `tag` when detection requires correlation across packets.
* [ ] Use `rate` + `track` when detection is behavioral.
* [ ] Assign an appropriate signature weight.
* [ ] Test before production deployment.

### Detection Model

```text
Protocol
   ↓
Service
   ↓
Flow
   ↓
Context
   ↓
Pattern / PCRE
   ↓
Range
   ↓
Byte Operations
   ↓
Tag / Rate / Track
   ↓
Weight
   ↓
IPS Action
```

---

# 2. F-SBID Structure Checklist

## Basic Structure

* [ ] Use the `F-SBID` wrapper.
* [ ] Write `F-SBID` in uppercase.
* [ ] Separate signature options with `;`.
* [ ] Validate the complete syntax before deployment.
* [ ] Verify quotation marks around string values.
* [ ] Verify special-character escaping.

### Basic Template

```text
F-SBID(
    --name "Vendor.Product.Attack";
    --protocol tcp;
    --service HTTP;
    --flow from_client;
    --pattern "malicious";
    --context uri;
)
```

### Syntax Checklist

* [ ] `F-SBID(` exists.
* [ ] All options are inside the block.
* [ ] Each option ends with `;`.
* [ ] Strings use `"..."`.
* [ ] Closing `)` exists.
* [ ] Signature name is unique.

---

# 3. Signature Identity Checklist

## `--name`

* [ ] Define a unique signature name.
* [ ] Keep the name within the supported length.
* [ ] Avoid spaces.
* [ ] Use `.` to separate logical components.
* [ ] Use a predictable naming convention.

### Recommended Naming Model

```text
vendor.product.component.vulnerability
```

Example:

```text
IBM.Domino.iNotes.Foldername.Buffer.Overflow
```

### Naming Checklist

* [ ] Vendor identified.
* [ ] Product identified.
* [ ] Component identified.
* [ ] Attack/vulnerability identified.
* [ ] Name is unique.
* [ ] Name is descriptive enough for troubleshooting.

---

# 4. Protocol and Traffic Checklist

## Protocol

* [ ] Identify the correct transport protocol.
* [ ] Use `--protocol tcp` when the signature targets TCP traffic.
* [ ] Use the appropriate protocol for UDP/ICMP/etc.
* [ ] Confirm the actual traffic matches the selected protocol.

Example:

```text
--protocol tcp;
```

---

## Service

* [ ] Identify whether FortiGate has a service/protocol decoder.
* [ ] Prefer `--service` when an appropriate service tree exists.
* [ ] Avoid relying only on destination ports for application identification.
* [ ] Confirm the traffic is actually classified as the selected service.

Example:

```text
--service HTTP;
```

### Service vs Port

```text
Application-aware approach
        ↓
--service HTTP
        ↓
HTTP service tree
```

Instead of relying only on:

```text
--dst_port 80
```

---

## Flow

* [ ] Determine whether the attack originates from the client.
* [ ] Determine whether the attack originates from the server.
* [ ] Select the correct flow direction.
* [ ] Verify the signature against a packet capture.

Example:

```text
--flow from_client;
```

---

# 5. Pattern Matching Checklist

## `--pattern`

* [ ] Use `--pattern` when simple matching is sufficient.
* [ ] Use a distinctive pattern.
* [ ] Avoid unnecessarily short patterns.
* [ ] Consider case sensitivity.
* [ ] Use `--no_case` when appropriate.
* [ ] Verify the exact bytes present in the inspected buffer.
* [ ] Consider encoding and normalization.

### Basic Syntax

```text
--pattern "value";
```

### Case-Insensitive Matching

```text
--pattern "VRFY";
--no_case;
```

---

## Pattern Selection

* [ ] Is the pattern sufficiently unique?
* [ ] Is it long enough to avoid excessive matches?
* [ ] Can the pattern be restricted to a specific context?
* [ ] Can a service restriction be added?
* [ ] Can the pattern replace a more expensive PCRE?

### Preferred Design

```text
Specific service
      ↓
Specific context
      ↓
Distinctive pattern
```

---

## Binary Pattern Checklist

* [ ] Use hexadecimal representation when binary matching is required.
* [ ] Verify byte order.
* [ ] Verify exact byte sequence.
* [ ] Verify the offset/reference point.

Example:

```text
--pattern "|05 00|";
```

---

## Escaping Checklist

* [ ] Check quotation marks.
* [ ] Check semicolons.
* [ ] Check backslashes.
* [ ] Check pipe characters.
* [ ] Verify special characters against the FortiOS syntax.

---

## Line Ending Checklist

When matching protocol line endings:

* [ ] Verify whether `LF` or `CRLF` is actually required.
* [ ] Use explicit hexadecimal representation where appropriate.

Example:

```text
|0a|
```

---

# 6. PCRE Checklist

## When to Use PCRE

* [ ] Confirm simple `pattern` matching is insufficient.
* [ ] Define exactly what structure the expression must match.
* [ ] Restrict PCRE to the smallest possible context.
* [ ] Avoid unnecessarily broad expressions.
* [ ] Test the expression against legitimate traffic.
* [ ] Test the expression against malicious traffic.
* [ ] Evaluate CPU/performance impact.

### Basic Syntax

```text
--pcre "/expression/";
```

Example:

```text
--pcre "/admin/i";
```

---

## PCRE Modifier Checklist

* [ ] `i` required for case-insensitive matching?
* [ ] `s`/`S` required for newline matching?
* [ ] `m` required for multi-line behavior?
* [ ] `x`/`X` required for readable expressions?
* [ ] Anchor behavior understood?
* [ ] Greedy vs non-greedy behavior understood?

### Example

```text
--pcre "/user\s*=\s*[^;]+/x";
```

---

## PCRE Performance Checklist

Avoid unnecessary:

```text
/.*/
```

```text
/.+/
```

* [ ] Remove unnecessary wildcards.
* [ ] Avoid excessive backtracking.
* [ ] Restrict the service.
* [ ] Restrict the context.
* [ ] Narrow the search range.
* [ ] Replace PCRE with `pattern` if possible.

### Golden Rule

```text
Simple condition
      ↓
pattern

Complex structured condition
      ↓
PCRE
```

---

# 7. Context Checklist

## Context Selection

* [ ] Identify where the malicious data actually exists.
* [ ] Select the narrowest available context.
* [ ] Avoid unrestricted payload matching when a specific buffer exists.
* [ ] Verify the protocol decoder exposes the expected data in that context.

### Common Contexts

| Context         | Typical Use              |
| --------------- | ------------------------ |
| `uri`           | HTTP URI                 |
| `header`        | HTTP/protocol headers    |
| `body`          | HTTP/message body        |
| `file`          | File/attachment content  |
| `packet`        | Packet-level inspection  |
| `packet_origin` | Original packet data     |
| `banner`        | Protocol banners         |
| `content`       | Compatibility/legacy use |
| `origin`        | Protocol-origin data     |

---

## HTTP URI

```text
--pattern "/admin";
--context uri;
```

Checklist:

* [ ] Attack exists in URI.
* [ ] URI context is selected.
* [ ] URI normalization/decoding is understood.
* [ ] Pattern is not unnecessarily broad.

---

## HTTP Header

```text
--pattern "User-Agent:";
--context header;
```

Checklist:

* [ ] Header field identified.
* [ ] Header context selected.
* [ ] Case sensitivity considered.

---

## HTTP Body

```text
--pattern "SELECT";
--context body;
```

Checklist:

* [ ] Payload exists in body.
* [ ] Body inspection is available.
* [ ] SSL inspection provides visibility where required.

---

## File Context

```text
--context file;
```

Checklist:

* [ ] File inspection is available.
* [ ] Target content exists inside the decoded file.
* [ ] Appropriate security inspection is enabled.

---

# 8. Range Modifier Checklist

Use range modifiers when the relative position between matching operations matters.

## Available Concepts

* [ ] `distance`
* [ ] `within`
* [ ] `distance_abs`
* [ ] `within_abs`
* [ ] `data_at`

---

## `distance`

* [ ] Identify the reference point.
* [ ] Define the minimum distance.
* [ ] Confirm the next match occurs after the required offset.

Example:

```text
--pattern "login";
--distance 10;
```

Concept:

```text
MATCH
  ↓
distance
  ↓
SEARCH START
```

---

## `within`

* [ ] Define the maximum search window.
* [ ] Confirm the target must occur inside the window.
* [ ] Keep the range as narrow as practical.

Example:

```text
--pattern "login";
--distance 10;
--within 20;
--pattern "fail";
```

Concept:

```text
login
  │
  └──── search window ────┐
                          ↓
                         fail
```

---

## First Pattern Range Checklist

When range modifiers are applied to the first pattern:

* [ ] Verify whether a previous match exists.
* [ ] Explicitly specify the reference point if required.
* [ ] Use `context` when the range must start from the selected buffer.

Example:

```text
--pattern "/disp_album.php?";
--context uri;
--within 50,context;
```

---

## Range Optimization

* [ ] Avoid unnecessarily large `distance`.
* [ ] Avoid unnecessarily large `within`.
* [ ] Avoid broad absolute ranges.
* [ ] Use the smallest range that still detects the attack.
* [ ] Test boundary conditions.

---

# 9. Reference Point Checklist

Before using range modifiers:

* [ ] Identify the previous pattern match.
* [ ] Identify whether the reference is the packet.
* [ ] Identify whether the reference is the context.
* [ ] Identify whether a tag stores the reference.
* [ ] Verify whether reverse searching is required.

### Common References

| Reference | Purpose                       |
| --------- | ----------------------------- |
| `match`   | Last matching pattern         |
| `packet`  | Packet beginning              |
| `context` | Beginning of selected context |
| `reverse` | Search relative to end        |
| `lasttag` | Reference stored by tag       |

### Design Checklist

```text
Reference
   ↓
Minimum distance
   ↓
Search window
   ↓
Target pattern
```

---

# 10. Byte Test and Byte Jump Checklist

Use byte operations when the protocol contains structured numeric fields.

---

## `byte_test`

* [ ] Identify the field.
* [ ] Determine the number of bytes.
* [ ] Determine byte order.
* [ ] Determine the comparison operator.
* [ ] Determine the expected value.
* [ ] Determine the offset.
* [ ] Verify relative/absolute behavior.

Example:

```text
--byte_test 2,>,100,0;
```

Concept:

```text
Read bytes
    ↓
Convert value
    ↓
Compare
    ↓
Match / No Match
```

---

## `byte_jump`

Use `byte_jump` when a packet field determines where the next structure starts.

* [ ] Identify the length/offset field.
* [ ] Determine the number of bytes.
* [ ] Determine the offset.
* [ ] Determine whether relative addressing is required.
* [ ] Determine byte order.
* [ ] Check alignment requirements.

Example:

```text
--byte_jump 4,0,relative;
```

Concept:

```text
Length Field
     ↓
Calculate Offset
     ↓
JUMP
     ↓
Continue Matching
```

---

## Advanced Byte Operations

Verify requirements for:

```text
--byte_jump 4,20,relative,align;
```

```text
--byte_jump 4,0,4,relative,little;
```

```text
--byte_test 4,>,0x7FFF,4,relative;
```

---

## Endianness Checklist

* [ ] Determine whether the protocol is Big Endian.
* [ ] Determine whether the protocol is Little Endian.
* [ ] Confirm the byte sequence from packet capture.

Example:

```text
0x12345678
```

Big Endian:

```text
12 34 56 78
```

Little Endian:

```text
78 56 34 12
```

---

## Alignment Checklist

When `align` is used:

* [ ] Confirm the protocol requires aligned boundaries.
* [ ] Understand 32-bit alignment behavior.
* [ ] Test values that cross alignment boundaries.

---

# 11. Parsed Type Checklist

Use `--parsed_type` when detection should target a decoded protocol/application structure.

Example:

```text
--parsed_type HTTP-GET;
```

Checklist:

* [ ] Confirm the protocol decoder supports the parsed type.
* [ ] Identify the correct logical structure.
* [ ] Avoid unrestricted raw-payload matching when parsed data is available.
* [ ] Verify the parsed type against real traffic.

Concept:

```text
Raw Packet
    ↓
Protocol Decoder
    ↓
Parsed Structure
    ↓
Targeted Detection
```

---

# 12. Protocol Header Checklist

## IP

* [ ] Determine whether IP header fields are relevant.
* [ ] Validate IP ID requirements.
* [ ] Avoid header matching when payload/context matching provides a more reliable detection method.

---

## TCP

* [ ] Confirm TCP traffic.
* [ ] Verify sequence-number requirements.
* [ ] Determine whether relative comparison is needed.
* [ ] Validate the condition against packet captures.

Example:

```text
--seq =,12345;
```

Operators:

| Operator | Meaning      |
| -------- | ------------ |
| `=`      | Equal        |
| `>`      | Greater than |
| `<`      | Less than    |
| `!`      | Not equal    |

Relative example:

```text
--seq <,12345,relative;
```

---

## ICMP

* [ ] Verify ICMP traffic.
* [ ] Identify ICMP ID requirements.
* [ ] Identify ICMP sequence requirements.

Examples:

```text
--icmp_id <number>;
```

```text
--icmp_seq <number>;
```

---

# 13. Rate and Track Checklist

Use behavioral detection when one packet is insufficient.

Suitable use cases:

* [ ] Brute force.
* [ ] Flooding.
* [ ] Excessive requests.
* [ ] Repeated suspicious activity.
* [ ] Authentication abuse.
* [ ] Abnormal request frequency.

---

## Rate

* [ ] Define the event count.
* [ ] Define the time interval.
* [ ] Determine whether `limit` is required.
* [ ] Test below-threshold traffic.
* [ ] Test threshold traffic.
* [ ] Test above-threshold traffic.

Example:

```text
--rate 20,60;
```

Concept:

```text
20 events
    ↓
60 seconds
    ↓
Threshold evaluation
```

---

## Track

Determine what entity should be counted:

* [ ] Source IP.
* [ ] Destination IP.
* [ ] DHCP client.
* [ ] DNS domain.
* [ ] DNS domain + IP.
* [ ] Other supported tracking entity.

Example:

```text
--track src_ip;
```

---

## Rate + Track

Validate that:

```text
Client A
   ↓
20 requests
   ↓
Threshold reached
```

does not incorrectly cause:

```text
Client B
   ↓
2 requests
```

to inherit Client A's count.

---

# 14. Session Tag Checklist

Use tags when detection requires multi-stage correlation.

## Tag Operations

* [ ] `set`
* [ ] `pset`
* [ ] `clear`
* [ ] `toggle`
* [ ] `test`

---

## Set

```text
--tag set,Tag.Attack;
```

Checklist:

* [ ] Tag name is unique.
* [ ] Tag contains valid printable characters.
* [ ] Tag is initially expected to be unset.

---

## Test

```text
--tag test,Tag.Attack;
```

Test absence:

```text
--tag test,!Tag.Attack;
```

---

## Clear

```text
--tag clear,Tag.Attack;
```

---

## Tag Workflow

```text
Packet #1
    ↓
Signature A
    ↓
TAG SET
    ↓
Packet #2
    ↓
Signature B
    ↓
TAG TEST
    ↓
Attack Confirmed
```

---

## Tag Naming Checklist

Avoid unsupported/problematic characters:

* [ ] No spaces.
* [ ] No comma.
* [ ] No `!`.
* [ ] No `;`.
* [ ] Use descriptive names.
* [ ] Keep names consistent across related signatures.

---

# 15. Weight and ID Checklist

## Weight

* [ ] Determine whether signature priority needs adjustment.
* [ ] Assign a deliberate weight.
* [ ] Avoid arbitrarily using very high values.
* [ ] Test behavior when multiple signatures match.

Example:

```text
--weight 40;
```

Range:

```text
0 - 255
```

Concept:

```text
Low confidence
      ↓
Lower weight

High confidence
      ↓
Higher weight
```

---

## Attack ID

* [ ] Determine whether an explicit attack ID is required.
* [ ] Ensure the ID is predictable and unique.
* [ ] Consider centralized management requirements.

Example:

```text
--attack_id 8151;
```

---

## Vulnerability ID

* [ ] Determine whether a vulnerability ID is required.
* [ ] Keep identification consistent.

Example:

```text
--vuln_id 8151;
```

---

# 16. Custom IPS Deployment Checklist

## Configuration

* [ ] Create the custom IPS signature.
* [ ] Validate the signature syntax.
* [ ] Confirm the signature appears in the configuration.
* [ ] Confirm the custom signature is enabled.
* [ ] Attach the signature to the appropriate IPS sensor/profile.
* [ ] Attach the IPS sensor to the correct firewall policy.

Concept:

```text
Client
   ↓
Firewall Policy
   ↓
IPS Sensor
   ↓
Built-in Signatures
       +
Custom Signatures
   ↓
Detection
   ↓
Action
```

---

## CLI Concept

```text
config ips custom
    edit "ips-cus-test"
        ...
    next
end
```

Checklist:

* [ ] Correct custom signature object.
* [ ] Correct signature content.
* [ ] Correct IPS sensor.
* [ ] Correct firewall policy.
* [ ] Correct traffic direction.
* [ ] Logging enabled where required.

---

## Encrypted Traffic

When detecting application content inside encrypted traffic:

* [ ] Determine whether SSL inspection is required.
* [ ] Verify the appropriate SSL inspection profile.
* [ ] Verify the traffic is actually being decrypted.
* [ ] Confirm IPS receives the required application visibility.
* [ ] Test detection after SSL inspection.

Concept:

```text
HTTPS
  ↓
SSL Inspection
  ↓
Decrypted Application Data
  ↓
IPS
  ↓
Signature Matching
```

---

# 17. Traffic Shaping Checklist

Custom IPS detection can be part of a broader application-security architecture.

Validate:

* [ ] Application Control identifies the application.
* [ ] Application Group contains the intended applications.
* [ ] Traffic Shaper is configured.
* [ ] Policy matches the intended traffic.
* [ ] Traffic direction is correct.
* [ ] Application identification occurs before shaping is expected.
* [ ] Services do not unintentionally exclude the application.

Concept:

```text
Application Control
       ↓
Application Group
       ↓
Firewall Policy
       ↓
Traffic Shaper
       ↓
Bandwidth Control
```

---

## Application Group Checklist

* [ ] Create/identify the correct application group.
* [ ] Add the intended application signatures.
* [ ] Remove unrelated applications.
* [ ] Confirm applications are actually identified.

---

## Traffic Shaper Checklist

* [ ] Define maximum bandwidth.
* [ ] Define minimum guaranteed bandwidth if required.
* [ ] Apply the shaper to the correct policy.
* [ ] Validate traffic direction.
* [ ] Verify actual bandwidth behavior.

Example concept:

```text
Application Group
       ↓
Traffic Shaper
       ↓
100 Kbps Maximum
```

---

# 18. Signature Troubleshooting Checklist

## Signature Does Not Match

```text
[ ] F-SBID syntax is correct
[ ] Signature name is valid
[ ] Signature name is unique
[ ] Protocol is correct
[ ] Service is correct
[ ] Flow direction is correct
[ ] Context is correct
[ ] Pattern exists in the inspected buffer
[ ] Pattern encoding is correct
[ ] URI normalization is considered
[ ] PCRE expression is correct
[ ] Range modifiers are correct
[ ] Byte offsets are correct
[ ] Byte order is correct
[ ] Parsed type is correct
[ ] IPS sensor contains the signature
[ ] Firewall policy uses the correct IPS sensor
[ ] Traffic is actually passing through the policy
[ ] SSL inspection provides required visibility
```

---

# 19. False Positive Checklist

If the signature matches legitimate traffic:

```text
[ ] Pattern is too short
[ ] Pattern is too generic
[ ] Context is missing
[ ] Service restriction is missing
[ ] Flow direction is too broad
[ ] PCRE is too generic
[ ] PCRE contains excessive wildcards
[ ] Range is too large
[ ] Case sensitivity is incorrect
[ ] Encoding assumptions are incorrect
[ ] Normalization assumptions are incorrect
[ ] Negative matching is required
[ ] Protocol decoder assumptions are incorrect
```

---

## False Positive Reduction Strategy

```text
Broad Pattern
      ↓
Specific Pattern
      ↓
Specific Context
      ↓
Specific Service
      ↓
Specific Flow
      ↓
Restricted Range
      ↓
Validated Signature
```

---

# 20. Performance Optimization Checklist

## Signature Optimization

* [ ] Prefer `pattern` over PCRE when possible.
* [ ] Use the most specific service.
* [ ] Use the narrowest context.
* [ ] Use distinctive patterns.
* [ ] Restrict `distance`.
* [ ] Restrict `within`.
* [ ] Use byte operations for structured numeric fields.
* [ ] Avoid unnecessary wildcards.
* [ ] Avoid unnecessarily broad PCRE.
* [ ] Avoid inspecting the entire packet when a specific buffer is available.
* [ ] Test CPU impact under realistic traffic.

---

## Optimization Hierarchy

```text
Specific Service
       ↓
Specific Context
       ↓
Distinctive Pattern
       ↓
Small Search Range
       ↓
Byte Operations
       ↓
PCRE only when necessary
```

---

## Matching Cost Awareness

| Detection Method    | Relative Cost | Preferred Use        |
| ------------------- | ------------: | -------------------- |
| `pattern`           |        🟢 Low | Simple strings/bytes |
| `pattern + context` |   🟢 Very Low | Preferred            |
| `pattern + range`   |        🟢 Low | Structured matching  |
| `byte_test`         |  🟢 Efficient | Numeric comparison   |
| `byte_jump`         |   🟡 Moderate | Dynamic offsets      |
| PCRE                |     🔴 Higher | Complex expressions  |

---

# 21. Production Readiness Checklist

Before deploying a custom signature globally:

## Detection

* [ ] Attack condition is clearly defined.
* [ ] Protocol is verified.
* [ ] Service is verified.
* [ ] Flow direction is verified.
* [ ] Context is verified.
* [ ] Pattern is distinctive.
* [ ] PCRE is necessary only when required.
* [ ] Range restrictions are appropriate.
* [ ] Byte operations are validated.
* [ ] Tags/rate tracking are validated where required.

## Accuracy

* [ ] Tested against malicious traffic.
* [ ] Tested against legitimate traffic.
* [ ] False positives investigated.
* [ ] False negatives investigated.
* [ ] Case sensitivity tested.
* [ ] Encoding tested.
* [ ] Normalization tested.
* [ ] Boundary conditions tested.

## Performance

* [ ] CPU impact evaluated.
* [ ] High-volume traffic tested.
* [ ] PCRE complexity evaluated.
* [ ] Search range minimized.
* [ ] Context minimized.
* [ ] Service restriction enabled where possible.

## Deployment

* [ ] Custom signature installed.
* [ ] IPS sensor updated.
* [ ] Firewall policy verified.
* [ ] Logging verified.
* [ ] Monitoring enabled.
* [ ] Rollback procedure prepared.

---

# 22. NSE Memory Checklist

## Custom IPS

* [ ] `F-SBID`
* [ ] `--name`
* [ ] `--protocol`
* [ ] `--service`
* [ ] `--flow`
* [ ] `--pattern`
* [ ] `--pcre`
* [ ] `--context`
* [ ] `--distance`
* [ ] `--within`
* [ ] `--distance_abs`
* [ ] `--within_abs`
* [ ] `--data_at`
* [ ] `--byte_test`
* [ ] `--byte_jump`
* [ ] `--parsed_type`
* [ ] `--rate`
* [ ] `--track`
* [ ] `--tag`
* [ ] `--weight`
* [ ] `--attack_id`
* [ ] `--vuln_id`

---

## Pattern vs PCRE

```text
Simple match
     ↓
pattern

Complex expression
     ↓
PCRE
```

* [ ] Remember: don't use PCRE when pattern is enough.

---

## Context

```text
URI    → uri
Header → header
Body   → body
File   → file
```

* [ ] Always ask: "Where exactly does the malicious data exist?"

---

## Behavioral Detection

```text
Single packet
     ↓
pattern / PCRE

Multiple events
     ↓
rate + track

Multiple stages
     ↓
tag
```

---

## Structured Protocol

```text
Numeric field
     ↓
byte_test

Dynamic offset
     ↓
byte_jump
```

---

# 23. Quick Reference

## Minimal Signature

```text
F-SBID(
    --name "Vendor.Product.Attack";
    --protocol tcp;
    --service HTTP;
    --flow from_client;
    --pattern "malicious";
    --context uri;
)
```

---

## Pattern

```text
--pattern "value";
```

Case insensitive:

```text
--pattern "value";
--no_case;
```

---

## PCRE

```text
--pcre "/expression/i";
```

---

## Context

```text
--context uri;
--context header;
--context body;
--context file;
```

---

## Range

```text
--distance 10;
--within 50;
```

Context reference:

```text
--distance 10,context;
--within 50,context;
```

---

## Byte Test

```text
--byte_test 2,>,100,0;
```

---

## Byte Jump

```text
--byte_jump 4,0,relative;
```

---

## Rate

```text
--rate 20,60;
```

---

## Track

```text
--track src_ip;
```

---

## Tags

```text
--tag set,Tag.Attack;
--tag test,Tag.Attack;
--tag test,!Tag.Attack;
--tag clear,Tag.Attack;
```

---

## Weight

```text
--weight 40;
```

---

## IDs

```text
--attack_id 8151;
--vuln_id 8151;
```

---

# 🔥 Production Custom IPS Formula

Use this decision process when creating a new signature:

```text
┌────────────────────────────┐
│ What am I detecting?       │
└──────────────┬─────────────┘
               ↓
┌────────────────────────────┐
│ Which protocol?            │
└──────────────┬─────────────┘
               ↓
┌────────────────────────────┐
│ Which service tree?        │
└──────────────┬─────────────┘
               ↓
┌────────────────────────────┐
│ Which traffic flow?        │
└──────────────┬─────────────┘
               ↓
┌────────────────────────────┐
│ Which context?             │
└──────────────┬─────────────┘
               ↓
       ┌───────┴────────┐
       ↓                ↓
┌──────────────┐  ┌──────────────┐
│ Simple Match │  │ Complex Logic│
└──────┬───────┘  └──────┬───────┘
       ↓                  ↓
   --pattern          --pcre
       └────────┬─────────┘
                ↓
┌────────────────────────────┐
│ Restrict search range      │
└──────────────┬─────────────┘
               ↓
┌────────────────────────────┐
│ Need numeric comparison?   │
│        byte_test           │
└──────────────┬─────────────┘
               ↓
┌────────────────────────────┐
│ Need dynamic offset?       │
│        byte_jump           │
└──────────────┬─────────────┘
               ↓
┌────────────────────────────┐
│ Multi-packet correlation?  │
│           tag              │
└──────────────┬─────────────┘
               ↓
┌────────────────────────────┐
│ Behavioral threshold?      │
│       rate + track         │
└──────────────┬─────────────┘
               ↓
┌────────────────────────────┐
│ Set appropriate weight     │
└──────────────┬─────────────┘
               ↓
┌────────────────────────────┐
│ Test → Tune → Deploy       │
└────────────────────────────┘
```

---

# 🧠 Golden Rules Checklist

* [ ] Prefer `--pattern` over `--pcre` whenever possible.
* [ ] Select the most specific `--context`.
* [ ] Use `--service` when an appropriate service decoder exists.
* [ ] Avoid short and generic patterns.
* [ ] Keep `distance` and `within` as narrow as practical.
* [ ] Use `byte_test` for numeric comparisons.
* [ ] Use `byte_jump` for dynamic-length structures.
* [ ] Use `tag` for multi-stage/session correlation.
* [ ] Use `rate` + `track` for behavioral detection.
* [ ] Treat PCRE as a powerful but potentially expensive mechanism.
* [ ] Ensure SSL inspection provides visibility when encrypted content must be inspected.
* [ ] Test against both malicious and legitimate traffic.
* [ ] Evaluate performance before global deployment.
* [ ] Verify IPS sensor and firewall-policy integration.
* [ ] Keep a rollback plan for production changes.

---

# 🎯 NSE Exam Memory Map

```text
CUSTOM IPS
│
├── F-SBID
│
├── Identity
│   ├── name
│   ├── attack_id
│   └── vuln_id
│
├── Traffic
│   ├── protocol
│   ├── service
│   └── flow
│
├── Detection
│   ├── pattern
│   ├── pcre
│   ├── context
│   ├── byte_test
│   └── byte_jump
│
├── Range
│   ├── distance
│   ├── within
│   ├── distance_abs
│   └── within_abs
│
├── Correlation
│   ├── tag
│   ├── rate
│   └── track
│
├── Priority
│   └── weight
│
└── Deployment
    ├── Custom IPS
    ├── IPS Sensor
    └── Firewall Policy
```

---

# ⚡ 30-Second Interview Checklist

**Question:** How do you design an efficient FortiGate custom IPS signature?

> **Answer:**

```text
[ ] Identify the protocol
[ ] Identify the service
[ ] Identify traffic direction
[ ] Select the narrowest context
[ ] Prefer pattern over PCRE
[ ] Use PCRE only for complex matching
[ ] Use distance/within to restrict searching
[ ] Use byte_test for numeric conditions
[ ] Use byte_jump for dynamic offsets
[ ] Use tag for multi-packet correlation
[ ] Use rate + track for behavioral detection
[ ] Assign appropriate weight
[ ] Test false positives/negatives
[ ] Evaluate performance
[ ] Deploy through the IPS sensor
```

### Interview Answer

> First identify the protocol, service, traffic direction and exact attack condition. Then select the narrowest possible inspection context. Use a distinctive `--pattern` whenever possible because it is generally more efficient than PCRE. Use PCRE only when complex matching is required. For structured binary protocols, use `byte_test` and `byte_jump`. Use `distance` and `within` to constrain the search range, and `tag`, `rate` and `track` when detection depends on multi-packet correlation or behavioral thresholds. Finally, validate false positives, performance and policy integration before production deployment.

---

# 🔗 Related SheynShield Topics

* [ ] FortiGate IPS
* [ ] FortiGate Custom IPS Signature
* [ ] FortiGate IPS Sensor
* [ ] FortiGate Application Control
* [ ] FortiGate Traffic Shaping
* [ ] FortiGate SSL Deep Inspection
* [ ] FortiGate DLP
* [ ] FortiGate WAF
* [ ] FortiGate ICAP
* [ ] FortiGate Security Profiles

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

```text
fortigate
fortinet
fortios
ips
intrusion-prevention
custom-ips
f-sbid
network-security
cybersecurity
nse4
nse7
fortinet-nse
security-engineering
firewall
ids-ips
pcre
packet-inspection
```

---

## 🔎 Keywords

```text
FortiGate Custom IPS Signature
FortiOS Custom IPS Signature
FortiGate F-SBID
FortiGate IPS Pattern Matching
FortiGate IPS PCRE
FortiGate IPS Context
FortiGate byte_test
FortiGate byte_jump
FortiGate IPS rate track
FortiGate IPS tag
Fortinet custom IPS signature example
FortiGate IPS troubleshooting
FortiGate IPS performance optimization
NSE7 Custom IPS Signature
NSE4 FortiGate IPS
```

---

> **SheynShield Principle:**
> **Detect precisely → Restrict the context → Minimize the search range → Prefer efficient matching → Validate before production.**

**SheynShield — Engineering Secure Networks**

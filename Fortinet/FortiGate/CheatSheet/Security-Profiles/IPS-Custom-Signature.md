# FortiGate Custom IPS Signature  

> **FortiOS / IPS Engine:** Custom IPS Signature Reference
> **Focus:** FortiGate IPS, F-SBID, Pattern Matching, PCRE, Context, Range Modifiers, Byte Operations, Tags, Rate Tracking & Traffic Shaping
> **Level:** NSE 4 → NSE 7
> **Brand:** SheynShield — Engineering Secure Networks

---

## Table of Contents

* [1. IPS Signature Architecture](#1-ips-signature-architecture)
* [2. F-SBID Signature Structure](#2-f-sbid-signature-structure)
* [3. Mandatory Signature Components](#3-mandatory-signature-components)
* [4. Pattern Matching](#4-pattern-matching)
* [5. PCRE Matching](#5-pcre-matching)
* [6. Context Selection](#6-context-selection)
* [7. Range Modifiers](#7-range-modifiers)
* [8. Reference Points](#8-reference-points)
* [9. PCRE Buffer Modifiers](#9-pcre-buffer-modifiers)
* [10. Protocol & Header Matching](#10-protocol--header-matching)
* [11. TCP / IP / ICMP Matching](#11-tcp--ip--icmp-matching)
* [12. Byte Test & Byte Jump](#12-byte-test--byte-jump)
* [13. Parsed Types](#13-parsed-types)
* [14. Rate & Track](#14-rate--track)
* [15. Session Tags](#15-session-tags)
* [16. Signature Weight](#16-signature-weight)
* [17. Attack ID & Vulnerability ID](#17-attack-id--vulnerability-id)
* [18. Custom Signature Examples](#18-custom-signature-examples)
* [19. Common Signature Design Errors](#19-common-signature-design-errors)
* [20. Performance Optimization](#20-performance-optimization)
* [21. Custom IPS Deployment](#21-custom-ips-deployment)
* [22. Traffic Shaping with Application Groups](#22-traffic-shaping-with-application-groups)
* [23. Troubleshooting Checklist](#23-troubleshooting-checklist)
* [24. Quick Reference](#24-quick-reference)

---

# 1. IPS Signature Architecture

FortiGate IPS signatures are used to identify malicious or suspicious traffic by inspecting packet headers, protocol structures, payloads, application data and behavioral patterns.

A signature can combine:

```text
Protocol
   ↓
Service
   ↓
Flow Direction
   ↓
Context / Buffer
   ↓
Pattern / PCRE
   ↓
Range Modifiers
   ↓
Byte Operations
   ↓
Tags / Rate Tracking
   ↓
Action
```

### IPS Detection Building Blocks

| Mechanism        | Purpose                                     |
| ---------------- | ------------------------------------------- |
| `pattern`        | Fast string / byte pattern matching         |
| `pcre`           | Regular-expression based matching           |
| Protocol decoder | Understand protocol structure               |
| Context          | Restrict inspection to a specific buffer    |
| `byte_test`      | Compare numeric values                      |
| `byte_jump`      | Move the reference point dynamically        |
| `tag`            | Correlate multiple packets/signatures       |
| `rate`           | Detect abnormal request frequency           |
| `track`          | Define what entity is counted               |
| `weight`         | Influence signature priority                |
| `service`        | Place signature in the correct service tree |
| `flow`           | Define traffic direction                    |

---

# 2. F-SBID Signature Structure

Fortinet custom IPS signatures use the **F-SBID** format.

> **F-SBID = Fortinet Signature Block ID**

Basic structure:

```text
F-SBID(
    --name "Example.Signature";
    --protocol tcp;
    --service SMTP;
    --flow from_client;
    --pattern "VRFY";
    --no_case;
    --context header;
)
```

### Important

`F-SBID` should be written in uppercase.

A signature consists of multiple options separated by semicolons:

```text
--option value;
```

---

# 3. Mandatory Signature Components

For a practical custom IPS signature, define at minimum:

```text
name
protocol
service
flow
```

Example:

```text
F-SBID(
    --name "SMTP.VRFY.Command";
    --protocol tcp;
    --service SMTP;
    --flow from_client;
    --pattern "VRFY";
    --no_case;
    --context header;
)
```

---

## Signature Name

The name should:

* Be enclosed in `"`.
* Be unique.
* Be no longer than 64 characters.
* Use `.` instead of spaces.

Example:

```text
--name "IBM.Domino.iNotes.Foldername.Buffer.Overflow";
```

A useful naming convention:

```text
vendor.product.component.vulnerability
```

Example:

```text
IBM.Domino.iNotes.Foldername.Buffer.Overflow
```

---

# 4. Pattern Matching

`--pattern` is the preferred matching mechanism when a simple byte/string pattern is sufficient.

Syntax:

```text
--pattern "string";
```

Example:

```text
--pattern "VRFY";
```

Case-sensitive by default.

For case-insensitive matching:

```text
--pattern "VRFY";
--no_case;
```

### Performance Rule

> Prefer `pattern` over `pcre` whenever a simple pattern can achieve the same detection.

Why?

```text
pattern
   ↓
fast matching
   ↓
lower CPU cost
```

versus:

```text
PCRE
   ↓
regular-expression processing
   ↓
higher CPU cost
   ↓
potential performance impact
```

---

## Binary / Hex Patterns

Binary values can be represented using hexadecimal notation.

Example:

```text
--pattern "|05 00|";
```

Multiple patterns can be chained:

```text
--pattern "|05 00|";
--distance 0;
--pattern "|6e 00|";
--distance 5;
--within 2;
```

---

## Escaping Special Characters

Special characters used by the signature parser must be escaped correctly.

Common representations include:

| Character | Hex representation |    |   |
| --------- | ------------------ | -- | - |
| `"`       | `                  | 22 | ` |
| `;`       | `                  | 3b | ` |
| `\`       | `                  | 5c | ` |
| `\|`      | `                  | 7c | ` |
| `:`       | `                  | 3a | ` |

### Example

Instead of relying on complicated escaping:

```text
--pattern "|22|";
```

can represent:

```text
"
```

---

## Line Endings

When detecting line endings in protocol headers, use:

```text
|0a|
```

rather than relying on:

```text
|0d 0a|
```

This is particularly relevant when inspecting HTTP headers.

---

# 5. PCRE Matching

PCRE = **Perl Compatible Regular Expressions**

Syntax:

```text
--pcre "/expression/";
```

Example:

```text
F-SBID(
    --name "HTTP.Admin.Path";
    --protocol tcp;
    --service HTTP;
    --flow from_client;
    --pcre "/admin/i";
    --context uri;
)
```

This matches:

```text
/admin
/Admin
/ADMIN
```

because of:

```text
i
```

---

## PCRE Modifiers

| Modifier | Meaning                               |
| -------- | ------------------------------------- |
| `i`      | Case-insensitive                      |
| `S`      | Dot matches newline                   |
| `m`      | Multi-line mode                       |
| `X`      | Ignore whitespace                     |
| `a`      | Force matching at beginning of buffer |
| `e`      | `$` matches only end of buffer        |
| `g`      | Non-greedy behavior by default        |

---

## PCRE Examples

### Case-insensitive

```text
--pcre "/admin/i";
```

### Multi-line content

```text
--pcre "/<script>.*?<\/script>/s";
```

### Beginning of lines

```text
--pcre "/^login=/m";
```

### Readable complex expression

```text
--pcre "/user\s*=\s*[^;]+/x";
```

---

## PCRE Performance Warning

> PCRE is significantly more expensive than normal pattern matching.

Avoid unnecessarily broad expressions such as:

```text
/.*/
```

or:

```text
/.+/
```

Especially dangerous when the signature:

* Applies to many protocols.
* Has no service restriction.
* Inspects server-side traffic.
* Runs on high-throughput interfaces.
* Uses complex backtracking expressions.

### Rule

```text
Simple condition → pattern

Complex structured condition → PCRE
```

---

# 6. Context Selection

`--context` tells the IPS engine **where to search**.

Examples:

```text
--context uri;
--context header;
--context body;
--context file;
```

Choosing the correct context is one of the most important IPS signature optimization techniques.

---

## Common Contexts

| Context         | Typical use              |
| --------------- | ------------------------ |
| `uri`           | HTTP URI                 |
| `header`        | HTTP/protocol headers    |
| `body`          | HTTP body / message body |
| `file`          | Files / attachments      |
| `packet`        | Packet-level inspection  |
| `packet_origin` | Original packet data     |
| `banner`        | Protocol banners         |
| `content`       | Legacy compatibility     |
| `origin`        | Protocol-origin data     |

---

## HTTP URI Example

Bad:

```text
--pattern "/../../";
```

Better:

```text
--pattern "/../..";
--context uri;
```

The context limits inspection to the URI buffer instead of forcing unnecessary payload-wide matching.

---

## HTTP Header Example

```text
--pattern "User-Agent:";
--context header;
```

---

## HTTP Body Example

```text
--pattern "SELECT";
--context body;
```

---

## File Example

```text
F-SBID(
    --protocol tcp;
    --pattern "X5O!P%@";
    --context file;
)
```

This is useful for detecting content inside decoded files.

---

# 7. Range Modifiers

Range modifiers control **where** the IPS engine searches.

| Modifier       | Purpose                                                      |
| -------------- | ------------------------------------------------------------ |
| `distance`     | Start searching a specified distance after a reference point |
| `within`       | Limit search range after a reference point                   |
| `distance_abs` | Absolute distance from a reference point                     |
| `within_abs`   | Absolute search boundary                                     |
| `data_at`      | Verify data exists at a specified position                   |

---

## Packet Structure

Conceptually:

```text
[ Ethernet Header ]
        ↓
[ IP Header ]
        ↓
[ TCP / UDP Header ]
        ↓
[ Payload / Data ]
        ↓
[ Trailer ]
```

Payload matching normally begins after protocol headers.

---

# 8. Reference Points

Range modifiers operate relative to a reference point.

Common references:

| Reference | Meaning                       |
| --------- | ----------------------------- |
| `match`   | Last matched pattern          |
| `packet`  | Beginning of packet           |
| `context` | Beginning of selected context |
| `reverse` | Search relative to the end    |
| `lasttag` | Reference stored by a tag     |

If no reference is specified, the default reference is generally the previous match.

---

## Distance

```text
--distance <range>,<refer>;
```

Conceptually:

```text
MATCH
  ↓
[ distance ]
  ↓
SEARCH START
```

Example:

```text
--pattern "login";
--distance 10;
```

The next matching operation starts after the defined distance from the reference point.

---

## Within

```text
--within <range>,<refer>;
```

Example:

```text
--pattern "login";
--distance 10;
--within 20;
--pattern "fail";
```

Conceptually:

```text
0                 10                 30
|-----------------|------------------|
                  login
                  |<--- within 20 --->|
                         fail
```

The `fail` pattern must be found inside the defined search window.

---

## First Pattern + Range Modifier

If `distance` or `within` is used with the **first pattern**, there is no previous match.

Therefore explicitly specify:

```text
context
```

Example:

```text
--pattern "/disp_album.php?";
--context uri;
--within 50,context;
```

---

## Distance + Within

A common pattern is:

```text
--distance X;
--within Y;
```

Together they define:

```text
reference
    ↓
minimum distance
    ↓
search window
    ↓
maximum boundary
```

Use them carefully because overly broad ranges can increase processing cost and false positives.

---

# 9. PCRE Buffer Modifiers

Legacy Snort-style PCRE buffer modifiers should not be confused with FortiGate's modern `--context` approach.

Conceptually, historical modifiers included:

| Modifier | Buffer             |
| -------- | ------------------ |
| `b`      | URI/raw URI buffer |
| `u`      | URI                |
| `p`      | HTTP POST body     |
| `h`      | HTTP header        |
| `m`      | HTTP method        |
| `c`      | HTTP cookie        |
| `i`      | HTTP response body |
| `d`      | DNP3 data          |
| `k`      | SMTP body          |
| `s`      | Sticky buffer      |
| `y`      | SSI                |

### FortiGate Best Practice

Prefer explicit context:

```text
--context uri;
```

rather than relying on deprecated or legacy buffer modifiers.

---

# 10. Protocol & Header Matching

IPS signatures can inspect:

* IP headers
* TCP/UDP headers
* ICMP fields
* Protocol payloads
* Application-layer protocol structures

The protocol decoder identifies traffic and allows the IPS engine to select the appropriate service tree.

---

## Service vs Port

Prefer:

```text
--service HTTP;
```

over:

```text
--dst_port 80;
```

when detecting HTTP.

Why?

FortiGate identifies services based on protocol/content detection rather than simply trusting port numbers.

Therefore HTTP can run on:

```text
80
8080
8000
custom ports
```

and still be classified as HTTP.

---

## Service Trees

FortiGate IPS maintains service-specific signature trees such as:

```text
HTTP
SMTP
POP3
DNS
SSH
...
```

A signature with:

```text
--service HTTP;
```

is assigned to the HTTP service tree.

A signature using only a port may fall into the unknown-service tree.

### Rule

> If the protocol/service has a supported service decoder, use `--service` instead of relying on destination ports.

---

# 11. TCP / IP / ICMP Matching

## IP ID

The IP ID field can be useful for detecting specific packet patterns and some spoofing/scanning behavior.

Example concept:

```text
--ip_id <value>;
```

---

## TCP Sequence Number

TCP sequence numbers can be matched using:

```text
--seq <operator>,<number>[,relative];
```

Examples:

```text
--seq =,12345;
```

```text
--seq >,12345;
```

```text
--seq <,12345;
```

```text
--seq !,12345;
```

Relative mode:

```text
--seq <,12345,relative;
```

### Operators

| Operator | Meaning      |
| -------- | ------------ |
| `=`      | Equal        |
| `>`      | Greater than |
| `<`      | Less than    |
| `!`      | Not equal    |

---

## ICMP ID

```text
--icmp_id <number>;
```

Used for ICMP Echo Request / Echo Reply packets.

---

## ICMP Sequence

```text
--icmp_seq <number>;
```

Used for ICMP Echo Request / Echo Reply packets.

---

# 12. Byte Test & Byte Jump

These keywords are extremely useful for structured binary protocols.

---

## `byte_test`

`byte_test` reads a specified number of bytes and compares the value.

Concept:

```text
read bytes
    ↓
compare value
    ↓
match / no match
```

Example:

```text
--byte_test 2,>,100,0;
```

Meaning conceptually:

> Read 2 bytes and check whether the value is greater than 100.

---

## `byte_jump`

`byte_jump` uses a value from the packet to dynamically move the reference point.

Concept:

```text
read length
     ↓
calculate offset
     ↓
jump
     ↓
continue inspection
```

Example:

```text
--byte_jump 4,0,relative;
```

---

## Examples

```text
--byte_jump 4,0,relative;
```

```text
--byte_test 4,>,3536,0,relative;
```

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

## Byte Test Syntax

Conceptually:

```text
--byte_test <bytes>,<operator>,<value>,<offset>[,<multiplier>[,<modifiers>]];
```

---

## Byte Jump Syntax

Conceptually:

```text
--byte_jump <bytes>,<offset>[,<multiplier>[,<modifiers>]];
```

---

# Big Endian vs Little Endian

Byte order determines how multi-byte values are interpreted.

### Big Endian

Most significant byte first:

```text
AA BB CC DD
```

### Little Endian

Least significant byte first:

```text
DD CC BB AA
```

Example value:

```text
0x12345678
```

Big endian:

```text
12 34 56 78
```

Little endian:

```text
78 56 34 12
```

---

## `align`

`align` rounds converted byte counts to the next 32-bit boundary.

Conceptually:

```text
7 bytes
   ↓
8-byte aligned boundary
```

Useful when protocol structures require aligned offsets.

---

# 13. Parsed Types

`--parsed_type` can be used when inspection needs to target a decoded protocol/application structure.

Example:

```text
--parsed_type HTTP-GET;
```

Another example:

```text
--parsed_type SSL_PCT;
```

This is useful when a protocol contains multiple logical structures and the signature needs to inspect a particular parsed type rather than blindly searching the entire payload.

---

## Example

```text
F-SBID(
    --name "SSL.PCT.Overflow";
    --protocol tcp;
    --flow from_client;
    --service SSL;
    --parsed_type SSL_PCT;
    --pattern "|01|";
    --within 1,packet;
    --distance 2,packet;
    --byte_test 2,>,1,3;
    --byte_test 2,<,0x301,3;
    --data_size >300;
)
```

### Why Parsed Types Matter

```text
Raw packet
    ↓
Protocol decoder
    ↓
Parsed structure
    ↓
Targeted signature matching
```

This can be much more precise than unrestricted payload matching.

---

# 14. Rate & Track

Not every attack can be detected by a single packet.

Examples:

* Brute-force authentication
* Flooding
* Excessive requests
* Repeated suspicious activity
* Abnormal request rates

For these cases:

```text
rate + track
```

can be used.

---

## Rate Syntax

Conceptually:

```text
--rate <count>,<duration>[,<limit>];
```

Example:

```text
--rate 20,60;
```

Meaning:

> Detect approximately 20 matching events during a 60-second interval.

---

## `limit`

`limit` enables stricter threshold handling.

Concept:

```text
Without limit
    ↓
average-based rate calculation

With limit
    ↓
stricter threshold enforcement
```

This can improve accuracy when the exact threshold matters.

---

## Track Options

Traffic can be tracked by entities such as:

```text
src_ip
dst_ip
dhcp_client
dns_domain
dns_domain + ip
```

Example concept:

```text
--track src_ip;
```

This means matching activity is counted per source IP.

---

## Rate + Track

Conceptually:

```text
Attacker A
   ↓
20 requests
   ↓
threshold reached
   ↓
signature action
```

while:

```text
Attacker B
   ↓
2 requests
   ↓
below threshold
```

This is useful for detecting behavior rather than a single malicious packet.

---

# 15. Session Tags

`--tag` allows signatures to correlate activity across packets within a session.

This becomes important when:

> The complete attack pattern is distributed across multiple packets or directions.

---

## Tag Operations

| Operation      | Purpose                                             |
| -------------- | --------------------------------------------------- |
| `set` / `pset` | Set a tag and optionally remember a reference point |
| `clear`        | Remove the tag                                      |
| `toggle`       | Toggle tag state                                    |
| `test`         | Check whether a tag exists                          |

---

## Tag Examples

Set:

```text
--tag set,Tag.Rsync.Argument;
```

Test:

```text
--tag test,Tag.Rsync.Argument;
```

Test absence:

```text
--tag test,!DHTML.EDIT.CONTROL.CLSID;
```

Clear:

```text
--tag clear,Tag.Login;
```

---

## Tag Workflow

```text
Packet #1
   ↓
Signature A matches
   ↓
TAG SET
   ↓
Packet #2
   ↓
Signature B checks TAG
   ↓
TAG exists
   ↓
Continue detection
```

This enables multi-stage detection.

---

## Tag Naming Rules

Tag names should contain printable characters.

Avoid:

```text
space
comma
!
;
```

A newly created tag starts in the unset state.

---

# 16. Signature Weight

`--weight` controls signature priority.

Syntax:

```text
--weight <value>;
```

Range:

```text
0 - 255
```

Example:

```text
--weight 40;
```

A higher weight can give a signature priority over a lower-weight signature when multiple signatures match.

Typical guidance:

```text
Custom signatures:
20 - 50
```

Application-control signatures commonly use lower values, while some high-priority botnet signatures may use much higher weights.

---

## Weight Strategy

```text
Low confidence signature
        ↓
lower weight

High confidence signature
        ↓
higher weight
```

Do not automatically assign extremely high weights to every custom signature.

---

# 17. Attack ID & Vulnerability ID

Custom signatures can use:

```text
--attack_id
--vuln_id
```

Example:

```text
--attack_id 8151;
--vuln_id 8151;
```

### Attack ID

Provides a unique identifier for the attack/signature.

For environments involving centralized management such as FortiManager, assigning explicit IDs is useful for predictable identification.

Example:

```text
--attack_id 8151;
```

---

# 18. Custom Signature Examples

## Example 1 — SMTP VRFY Detection

```text
F-SBID(
    --name "Block.SMTP.VRFY.CMD";
    --protocol tcp;
    --service SMTP;
    --flow from_client;
    --pattern "VRFY";
    --no_case;
    --context header;
)
```

---

## Example 2 — HTTP Admin Detection

```text
F-SBID(
    --name "HTTP.Admin.Access";
    --protocol tcp;
    --service HTTP;
    --flow from_client;
    --pattern "/admin";
    --no_case;
    --context uri;
)
```

---

## Example 3 — PCRE Detection

```text
F-SBID(
    --name "HTTP.Login.Pattern";
    --protocol tcp;
    --service HTTP;
    --flow from_client;
    --pcre "/user\s*=\s*[^;]+/x";
    --context body;
)
```

---

## Example 4 — SQL Injection Pattern

A simplified example:

```text
F-SBID(
    --name "HTTP.SQL.Injection";
    --protocol tcp;
    --service HTTP;
    --flow from_client;
    --pattern "SELECT";
    --no_case;
    --context uri;
)
```

For more complex detection, PCRE can be used:

```text
F-SBID(
    --name "HTTP.SQLi.Special.Characters";
    --protocol tcp;
    --service HTTP;
    --flow from_client;
    --pcre "/(%27)|(')|(%23)|(#)|(--)/i";
    --context uri;
)
```

> **Design principle:** Prefer precise, context-aware detection over broad payload matching.

---

# 19. Common Signature Design Errors

## ❌ 1. Matching an encoded URI instead of the decoded representation

Bad:

```text
--pattern "/..%c0%af../";
--context uri;
```

Prefer matching the normalized/decoded representation whenever possible:

```text
--pattern "/../..";
--context uri;
```

---

## ❌ 2. Using PCRE when `pattern` is enough

Bad:

```text
--pcre "/admin/i";
```

when a simple pattern works.

Better:

```text
--pattern "admin";
--no_case;
```

---

## ❌ 3. Missing Context

Bad:

```text
--pattern "User-Agent";
```

Better:

```text
--pattern "User-Agent";
--context header;
```

---

## ❌ 4. Using Port Instead of Service

Less optimal:

```text
--dst_port 80;
```

Better:

```text
--service HTTP;
```

---

## ❌ 5. Using Short Patterns

Patterns shorter than approximately four characters can be inefficient for matching engines.

Avoid unnecessary short patterns such as:

```text
--pattern "GET";
```

when they can cause excessive matches.

Prefer a more distinctive pattern:

```text
--pattern "GET /admin/";
```

when appropriate.

---

## ❌ 6. Broad PCRE

Avoid expressions such as:

```text
/.*/
```

or:

```text
/.+/
```

unless there is a very specific reason.

---

## ❌ 7. Wrong Context

For HTTP:

```text
URI → uri
Header → header
Body → body
File → file
```

Do not inspect the entire packet when a specific buffer is available.

---

## ❌ 8. Excessive Range

Avoid unnecessarily large:

```text
distance
within
distance_abs
within_abs
```

ranges.

Large search windows increase processing requirements and can increase false positives.

---

# 20. Performance Optimization

Custom IPS signatures execute inside a high-speed inspection engine.

Poorly designed signatures can impact performance.

## Optimization Hierarchy

Use this preference:

```text
Specific service
      ↓
Specific context
      ↓
Long distinctive pattern
      ↓
Small search range
      ↓
Byte operations when appropriate
      ↓
PCRE only when necessary
```

---

## Pattern vs PCRE

| Method              |  Performance | Use                             |
| ------------------- | -----------: | ------------------------------- |
| `pattern`           |      🟢 Fast | Simple strings / byte sequences |
| `pattern + context` | 🟢 Very fast | Preferred                       |
| `pattern + range`   |      🟢 Fast | Structured matching             |
| `byte_test`         | 🟢 Efficient | Binary values                   |
| `byte_jump`         |  🟡 Moderate | Dynamic offsets                 |
| `PCRE`              | 🔴 Expensive | Complex expressions             |

---

## Aho-Corasick / Fast Pattern Matching

Modern IPS engines use optimized pattern-matching techniques to efficiently search large signature databases.

The principle is:

```text
Large signature database
        ↓
optimized pattern matching
        ↓
parallel candidate detection
        ↓
protocol / context validation
        ↓
action
```

The practical lesson:

> Design signatures so the engine can eliminate non-matching traffic as early as possible.

---

# 21. Custom IPS Deployment

Create a custom IPS signature:

```text
config ips custom
    edit "ips-cus-test"
        ...
    next
end
```

Then attach the custom signature through an IPS sensor/profile.

Conceptual deployment:

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
Action
   ↓
Allow / Monitor / Block
```

---

## Inspection Requirement

IPS requires traffic inspection.

For encrypted traffic, consider the appropriate SSL inspection strategy when the goal is to inspect the encrypted application content.

Concept:

```text
HTTPS
  ↓
SSL inspection
  ↓
decrypted traffic
  ↓
IPS inspection
```

Without visibility into encrypted content, IPS cannot inspect what it cannot see.

---

# 22. Traffic Shaping with Application Groups

Custom IPS is not the only place where application identification matters.

FortiGate can combine:

```text
Application Control
        +
Application Group
        +
Traffic Shaping
```

---

## Application Group

Example:

```text
Security Profiles
    ↓
Application Signatures
    ↓
Application Group
    ↓
app-g-test
```

Example members:

```text
HTTP Browsers
Web Clients
```

---

## Traffic Shaping Concept

Example:

```text
Application Group
       ↓
Traffic Shaper
       ↓
Maximum Bandwidth = 100 Kbps
```

---

## Example Policy

Conceptually:

```text
config firewall security-policy
    edit 1
        set srcintf "port3"
        set dstintf "port1"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set status enable
        set schedule "always"
        set service "ALL"
        set app-group "risk_1"
        set logtraffic all
    next
end
```

### Important

Verify that:

* Application group members are correct.
* Services do not conflict with the policy.
* Traffic direction is correct.
* The shaper is applied to the intended direction.
* The application is actually identified before shaping is expected to occur.

---

# 23. Troubleshooting Checklist

## Signature Does Not Match

Check:

```text
[ ] F-SBID is uppercase
[ ] Signature syntax is valid
[ ] Name is unique
[ ] Protocol is correct
[ ] Service is correct
[ ] Flow direction is correct
[ ] Context is correct
[ ] Pattern is present in the inspected buffer
[ ] Encoding/normalization is considered
[ ] SSL inspection provides visibility
[ ] IPS sensor contains the custom signature
[ ] Firewall policy uses the correct IPS sensor
```

---

## Signature Matches Too Much

Check:

```text
[ ] Pattern is too short
[ ] Context is missing
[ ] Service is too broad
[ ] PCRE is too generic
[ ] Range is too large
[ ] Case sensitivity is incorrect
[ ] Negative pattern is missing
[ ] Protocol decoder assumptions are wrong
```

---

## Performance Problem

Check:

```text
[ ] Can PCRE be replaced with pattern?
[ ] Can context be narrowed?
[ ] Can service be specified?
[ ] Can pattern length be increased?
[ ] Can distance/within be reduced?
[ ] Is the signature applied globally?
[ ] Is the signature matching high-volume traffic?
[ ] Is an unnecessary wildcard being used?
```

---

# 24. Quick Reference

## F-SBID

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
--distance 10,context;
--within 50,context;
```

---

## Negative Pattern

```text
--pattern !"bad";
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

# 🔥 Custom IPS Signature Design Formula

For production-grade custom signatures, think in this order:

```text
                ┌─────────────────────┐
                │ What am I detecting?│
                └──────────┬──────────┘
                           ↓
                ┌─────────────────────┐
                │ Which protocol?     │
                └──────────┬──────────┘
                           ↓
                ┌─────────────────────┐
                │ Which service tree? │
                └──────────┬──────────┘
                           ↓
                ┌─────────────────────┐
                │ Which flow?         │
                └──────────┬──────────┘
                           ↓
                ┌─────────────────────┐
                │ Which context?      │
                └──────────┬──────────┘
                           ↓
             ┌─────────────┴─────────────┐
             ↓                           ↓
       Simple pattern              Complex logic
             ↓                           ↓
         --pattern                    --pcre
             │                           │
             └─────────────┬─────────────┘
                           ↓
                 Add range restriction
                           ↓
                 Add byte operations
                     if required
                           ↓
                    Add tag/rate
                     if behavioral
                           ↓
                     Set weight
                           ↓
                    Test + tune
```

---

# 🧠 Golden Rules

> **1. Prefer `--pattern` over `--pcre` whenever possible.**

> **2. Always select the most specific `--context` available.**

> **3. Use `--service` instead of relying only on ports when the service decoder exists.**

> **4. Avoid short and generic patterns.**

> **5. Keep `distance` and `within` ranges as narrow as practical.**

> **6. Use `byte_test` for numeric comparisons instead of forcing complex PCRE expressions.**

> **7. Use `byte_jump` when the protocol contains dynamic-length fields.**

> **8. Use tags when an attack is distributed across multiple packets.**

> **9. Use `rate` + `track` for behavioral attacks such as brute force and flooding.**

> **10. Treat PCRE as a powerful but expensive tool.**

> **11. For encrypted traffic, ensure the firewall has the required visibility before expecting IPS content inspection.**

> **12. Test custom signatures under realistic traffic conditions before deploying them globally.**

---

# 📌 NSE Exam Memory Map

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

## ⚡ 30-Second Interview Answer

**How do you design an efficient FortiGate custom IPS signature?**

> First identify the protocol, service and traffic direction. Then select the narrowest possible inspection context. Use a distinctive `--pattern` whenever possible because it is more efficient than PCRE. If the detection requires complex matching, use PCRE carefully. For structured binary protocols, use `byte_test` and `byte_jump`. Use `distance` and `within` to constrain the search range, and use `tag`, `rate` and `track` when detection depends on multiple packets or behavioral thresholds. Finally, assign an appropriate weight and test the signature under realistic traffic before production deployment.

---

## 🔗 Related SheynShield Topics

* FortiGate IPS
* FortiGate Custom IPS Signature
* FortiGate IPS Sensor
* FortiGate Application Control
* FortiGate Traffic Shaping
* FortiGate SSL Deep Inspection
* FortiGate DLP
* FortiGate WAF
* FortiGate ICAP
* FortiGate Security Profiles

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

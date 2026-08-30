# 🔗 SheynShield Resources

# FortiGate Web Filter & HTTP/3/QUIC — Checklist

> **FortiOS 7.x | Web Filtering | URL Filter | FortiGuard | Content Filter | Anti-Phishing | Safe Search | WISP | SSL Inspection | HTTP/3 | QUIC**
>
> A practical **FortiGate Web Filtering Checklist** for deployment, troubleshooting, NSE preparation, and real-world network security operations.

---

## 📚 Table of Contents

* [1. Web Filter Architecture](#1-web-filter-architecture)
* [2. Web Filter Processing Order](#2-web-filter-processing-order)
* [3. URL Filter](#3-url-filter)
* [4. URL Filter Actions](#4-url-filter-actions)
* [5. FortiGuard Web Filtering](#5-fortiguard-web-filtering)
* [6. FortiGuard Categories](#6-fortiguard-categories)
* [7. Category Authentication](#7-category-authentication)
* [8. Web Filter Override](#8-web-filter-override)
* [9. Credential Anti-Phishing](#9-credential-anti-phishing)
* [10. Web Content Filter](#10-web-content-filter)
* [11. Content Filter Weight & Threshold](#11-content-filter-weight--threshold)
* [12. FortiSandbox Malicious URL Blocking](#12-fortisandbox-malicious-url-blocking)
* [13. FortiGuard Rating Failure](#13-fortiguard-rating-failure)
* [14. Domain vs IP Rating](#14-domain-vs-ip-rating)
* [15. Invalid URL Protection](#15-invalid-url-protection)
* [16. Safe Search](#16-safe-search)
* [17. YouTube & Vimeo Restrictions](#17-youtube--vimeo-restrictions)
* [18. Google Account Restriction](#18-google-account-restriction)
* [19. HTTP POST Control](#19-http-post-control)
* [20. ActiveX & Java Applets](#20-activex--java-applets)
* [21. Cookie Filtering](#21-cookie-filtering)
* [22. WAD Diagnostics](#22-wad-diagnostics)
* [23. SSL Certificate Validation](#23-ssl-certificate-validation)
* [24. Websense WISP](#24-websense-wisp)
* [25. HTTP/3 & QUIC](#25-http3--quic)
* [26. HTTP/2 vs HTTP/3](#26-http2-vs-http3)
* [27. QUIC Packet Structure](#27-quic-packet-structure)
* [28. QUIC Header](#28-quic-header)
* [29. QUIC Frames & Streams](#29-quic-frames--streams)
* [30. QUIC vs TLS 1.3](#30-quic-vs-tls-13)
* [31. Web Filter Deployment Checklist](#31-web-filter-deployment-checklist)
* [32. Troubleshooting Checklist](#32-troubleshooting-checklist)
* [33. NSE Exam Traps](#33-nse-exam-traps)
* [34. Quick Revision](#34-quick-revision)
* [35. Final Mental Model](#35-final-mental-model)

---

# 1. Web Filter Architecture

FortiGate Web Filtering can combine multiple inspection mechanisms to make a web-access decision.

```text
                         Internet
                            |
                            v
                    +---------------+
                    |   FortiGate   |
                    +---------------+
                            |
              +-------------+-------------+
              |             |             |
              v             v             v
        URL Filtering   FortiGuard    Content Filter
              |         Web Rating          |
              |             |               |
              +-------------+---------------+
                            |
                            v
                    Proxy-Based Processing
                            |
                            v
                    Additional Inspection
```

## Architecture Checklist

* [ ] Firewall policy identified.
* [ ] Correct inspection mode selected.
* [ ] Web Filter profile attached.
* [ ] URL filtering requirements identified.
* [ ] FortiGuard Web Filtering enabled where required.
* [ ] Content filtering requirements identified.
* [ ] Anti-phishing requirements identified.
* [ ] SSL inspection requirements identified.
* [ ] Logging requirements defined.
* [ ] Application compatibility tested.

## Main Components

| Component                     | Primary Function                                   |
| ----------------------------- | -------------------------------------------------- |
| **URL Filter**                | Matches administrator-defined URLs and patterns    |
| **FortiGuard Web Filter**     | Uses FortiGuard categories and ratings             |
| **Web Content Filter**        | Searches web content for configured words/patterns |
| **Web Script Filter**         | Inspects supported web scripts                     |
| **Antivirus**                 | Scans applicable web content/downloads             |
| **Anti-Phishing**             | Detects suspicious credential-submission behavior  |
| **FortiSandbox URL Blocking** | Uses malicious URL intelligence                    |

> **NSE Memory:**
> **URL Filter = administrator-defined patterns**
> **FortiGuard = category/rating intelligence**

---

# 2. Web Filter Processing Order

A simplified proxy web-filtering workflow can be represented as:

```text
URL Filter
    |
    v
FortiGuard Web Filtering
    |
    v
Web Content Filter
    |
    v
Web Script / Additional Proxy Inspection
    |
    v
Antivirus / Other Security Engines
```

## Processing Checklist

* [ ] URL filtering behavior understood.
* [ ] FortiGuard rating behavior verified.
* [ ] Content filtering behavior tested.
* [ ] Anti-phishing processing requirements reviewed.
* [ ] SSL inspection requirements verified.
* [ ] Blocking decisions tested.
* [ ] Logging verified.

### Important

An earlier blocking decision can prevent later inspection stages from processing the request.

```text
URL Filter
    |
    +---- BLOCK
            |
            +----> Request denied
            |
            +----> Later inspection may not occur
```

> **NSE Memory:**
> Do not assume every security engine evaluates every request.

---

# 3. URL Filter

URL Filter provides administrator-defined URL matching.

## URL Filter Checklist

* [ ] Required URLs identified.
* [ ] Required wildcard patterns created.
* [ ] Required regex patterns created.
* [ ] Matching type verified.
* [ ] Allow rules tested.
* [ ] Block rules tested.
* [ ] Monitor rules tested.
* [ ] Exempt rules reviewed.
* [ ] Rule behavior validated with logs.

## Example

```text
URL:
*.example.com

Type:
Wildcard

Action:
Block
```

## Regex Example

```text
.*ya*
```

> **Exam Trap:**
> **Wildcard ≠ Regex.**
> Always verify the matching engine before designing the pattern.

---

# 4. URL Filter Actions

| Action      | Result                                             |
| ----------- | -------------------------------------------------- |
| **Exempt**  | Bypasses remaining applicable web/proxy inspection |
| **Block**   | Denies the request                                 |
| **Allow**   | Allows processing to continue                      |
| **Monitor** | Allows processing and generates matching logs      |

## Allow vs Exempt

### Allow

```text
URL Match
    |
    v
  ALLOW
    |
    v
Continue applicable inspection
```

### Exempt

```text
URL Match
    |
    v
 EXEMPT
    |
    v
Bypass remaining applicable inspection
```

## Checklist

* [ ] Allow and Exempt behavior understood.
* [ ] Exempt rules limited to trusted requirements.
* [ ] Exempt rules reviewed for security impact.
* [ ] Monitor action used where visibility is required.
* [ ] Block action tested with replacement/block page.

> ### 🧠 NSE Memory
>
> **ALLOW ≠ EXEMPT**

---

# 5. FortiGuard Web Filtering

FortiGuard Web Filtering provides centrally maintained URL/domain classification and reputation information.

```text
Client
  |
  v
FortiGate
  |
  +----> URL Filter
  |
  +----> FortiGuard Rating
  |
  +----> Category
  |
  v
Final Action
```

## Checklist

* [ ] FortiGuard service availability verified.
* [ ] FortiGuard connectivity tested.
* [ ] Web Filter license/service status checked.
* [ ] Required categories identified.
* [ ] Category actions configured.
* [ ] Rating failures tested.
* [ ] FortiGuard logs reviewed.

---

# 6. FortiGuard Categories

Common category examples include:

|    ID | Category                       |
| ----: | ------------------------------ |
|  `26` | Social Networking              |
|  `31` | Streaming Media                |
|  `33` | Gambling                       |
|  `34` | Pornography                    |
|  `37` | Proxy Avoidance / Anonymizers  |
|  `38` | Malicious / Dangerous Websites |
|  `39` | Phishing                       |
|  `40` | Botnet                         |
|  `95` | Advertising / Trackers         |
| `140` | Unused / Redirection Domains   |
| `141` | Hacking / Crack-related Sites  |
| `149` | Guns / Drugs                   |

## Category Checklist

* [ ] Required categories identified.
* [ ] High-risk categories blocked.
* [ ] Business-required categories reviewed.
* [ ] Category overrides reviewed.
* [ ] Category IDs verified against deployed FortiOS/FortiGuard version.

> **Important:** Category IDs and names can change across releases. Verify them against the documentation for the deployed version.

---

# 7. Category Authentication

FortiGate can require authentication before granting access to selected categories.

## Checklist

* [ ] Authentication source configured.
* [ ] User/group mapping verified.
* [ ] Category action set to authentication.
* [ ] Authentication group configured.
* [ ] Warning duration reviewed.
* [ ] Authentication logs verified.
* [ ] Allowed and denied users tested.

## Example

```bash
config webfilter profile
    edit "wb-f-test"
        set feature-set proxy

        config ftgd-wf
            config filters
                edit 95
                    set action authentication
                    set auth-usr-grp "grp-test-fsso"
                    set log enable
                    set warn-duration 5
                next
            end
        end
    next
end
```

## Traffic Flow

```text
User
 |
 v
Restricted Category
 |
 v
Authentication Required
 |
 v
User / Group Validation
 |
 +----> ALLOW
 |
 +----> DENY
```

---

# 8. Web Filter Override

Web Filter Override allows different users/groups to receive different web-filtering behavior.

```text
Web Filter Profile
        |
   +----+----+----+
   |         |    |
   v         v    v
Group A   Group B Group C
Strict    Monitor Override
```

## Checklist

* [ ] Override requirement identified.
* [ ] Authorized users/groups defined.
* [ ] Override permissions configured.
* [ ] Override profile configured.
* [ ] Category override behavior tested.
* [ ] Override logs reviewed.
* [ ] Privileged override access restricted.

## Example

```bash
config webfilter profile
    edit "web-filt-test"

        set ovrd-perm \
            bannedword-override \
            urlfilter-override \
            fortiguard-wf-override \
            contenttype-check-override

        config override
            set ovrd-user-group "ldap-test"
            set profile "monitor-all"
        end

    next
end
```

> **Security Principle:**
> Web Filter Override should be treated as a controlled privilege, not a general bypass mechanism.

---

# 9. Credential Anti-Phishing

Credential Anti-Phishing can inspect web requests for credential-submission behavior.

```text
Client
  |
  v
Web Request
  |
  v
FortiGate Proxy
  |
  +----> URI Check
  |
  +----> FortiGuard Classification
  |
  +----> Credential Analysis
  |
  v
Action
```

## Checklist

* [ ] Anti-phishing enabled where required.
* [ ] URI inspection configured.
* [ ] Basic authentication inspection reviewed.
* [ ] Username-only inspection reviewed.
* [ ] Credential patterns reviewed.
* [ ] FortiGuard categories configured.
* [ ] Block/default action tested.
* [ ] Anti-phishing logging enabled.

## Main Options

```text
check-uri
check-basic-auth
check-username-only
```

## Example

```bash
config webfilter profile
    edit "wf-test"

        config antiphish
            set status enable
            set default-action block
            set check-uri enable
            set check-basic-auth enable
            set check-username-only enable
        end

    next
end
```

## Processing Priority

```text
URL Filter
    |
    +---- BLOCK
    |      |
    |      +----> Anti-Phishing may not process
    |
    +---- ALLOW
           |
           v
      FortiGuard
           |
           +---- BLOCK
           |      |
           |      +----> Anti-Phishing may not process
           |
           +---- ALLOW
                  |
                  v
             Anti-Phishing
```

> **NSE Memory:**
> If traffic is denied before anti-phishing processing, the anti-phishing engine does not get an opportunity to inspect that request.

---

# 10. Web Content Filter

Content filtering evaluates content within web pages rather than simply matching the URL.

```text
Web Content
    |
    +---- Word / Pattern
    |
    +---- Word / Pattern
    |
    v
Weighted Score
    |
    v
Threshold
    |
    +---- BLOCK
    |
    +---- ALLOW
```

## Checklist

* [ ] Banned words identified.
* [ ] Patterns configured.
* [ ] Scores configured.
* [ ] Threshold configured.
* [ ] False positives tested.
* [ ] Legitimate applications tested.
* [ ] Logging enabled where required.

---

# 11. Content Filter Weight & Threshold

A configured word/pattern can contribute a score toward the content-filter threshold.

```text
Detected Content
       |
       v
Score Calculation
       |
       v
Threshold Comparison
       |
 +-----+-----+
 |           |
 v           v
>= Threshold < Threshold
 |           |
 v           v
BLOCK       ALLOW
```

## Example

| Pattern       | Score |
| ------------- | ----: |
| `drug`        |    10 |
| `drug deal`   |    20 |
| `"drug deal"` |    10 |

Example threshold:

```text
10
```

## Checklist

* [ ] Score values reviewed.
* [ ] Threshold reviewed.
* [ ] Multiple-match behavior tested.
* [ ] False positives tested.
* [ ] Regex behavior verified.
* [ ] Wildcard behavior verified.

## Example

```bash
config webfilter content
    edit 1
        config entries
            edit "keyboard*"
                set score 1
                set action block
            next
        end
    next
end
```

---

# 12. FortiSandbox Malicious URL Blocking

FortiGate can use malicious URL intelligence associated with FortiSandbox.

## Checklist

* [ ] FortiSandbox integration verified.
* [ ] Malicious URL blocking requirement identified.
* [ ] Web Filter profile reviewed.
* [ ] Blocklist behavior tested.
* [ ] Logs verified.

## Example

```bash
config webfilter profile
    edit "wf-test"

        config web
            set blocklist enable
        end

    next
end
```

---

# 13. FortiGuard Rating Failure

A FortiGate may fail to obtain a current FortiGuard rating because of connectivity or service issues.

## Rating Failure Checklist

* [ ] FortiGuard connectivity tested.
* [ ] DNS resolution verified.
* [ ] FortiGuard service status verified.
* [ ] Rating failure behavior identified.
* [ ] `error-allow` reviewed.
* [ ] Security/availability trade-off documented.

## Example

```bash
config webfilter profile
    edit "wf-test"

        config ftgd-wf
            set options error-allow
        end

    next
end
```

## Behavior

```text
FortiGuard Rating
       |
 +-----+------+
 |            |
 v            v
Available   Unavailable
 |            |
 v            v
Normal       error-allow?
Decision       |
              +----> ALLOW
```

| Behavior                  | Availability | Security |
| ------------------------- | ------------ | -------- |
| **Allow on rating error** | Higher       | Lower    |
| **Fail closed / block**   | Lower        | Higher   |

> ### 🧠 NSE Memory
>
> `error-allow` prioritizes **availability** when FortiGuard rating cannot be obtained.

---

# 14. Domain vs IP Rating

Depending on configuration and FortiOS capabilities, FortiGate can use domain and destination-IP information when determining web ratings.

```text
Requested Domain
       +
Destination IP
       |
       v
Rating Resolution
       |
       v
Final Category
```

## Checklist

* [ ] Domain-based rating verified.
* [ ] IP-rating behavior understood.
* [ ] `rate-server-ip` requirement evaluated.
* [ ] Different domain/IP classifications considered.
* [ ] Final category/action tested.

## Example

```bash
config webfilter profile
    edit "wf-test"

        config ftgd-wf
            set options rate-server-ip
        end

    next
end
```

> **Important:** Rating precedence/weights are version-dependent. Do not memorize conceptual rating numbers as universal FortiGuard values.

---

# 15. Invalid URL Protection

Invalid URL protection can block malformed or invalid URL requests.

## Checklist

* [ ] Invalid URL behavior reviewed.
* [ ] `block-invalid-url` requirement evaluated.
* [ ] URL encoding tested.
* [ ] Application compatibility verified.

## Example

```bash
config webfilter profile
    edit "wf-test"
        set options block-invalid-url
    next
end
```

## URL Encoding

A space can be URL-encoded as:

```text
%20
```

Example:

```text
https://example.com/my%20file
```

---

# 16. Safe Search

FortiGate can enforce Safe Search for supported search engines.

Common examples:

* [ ] Google
* [ ] Bing
* [ ] Yahoo
* [ ] Yandex

## Safe Search Checklist

* [ ] Safe Search requirement identified.
* [ ] Supported search engines tested.
* [ ] Search enforcement configured.
* [ ] Search logging evaluated.
* [ ] Privacy requirements reviewed.
* [ ] FortiView visibility tested.

## Search Logging

```bash
config webfilter profile
    edit "wf-test"

        config web
            set log-search enable
        end

    next
end
```

> **Privacy Note:** Search queries can contain sensitive information. Enable search logging only when operationally justified.

---

# 17. YouTube & Vimeo Restrictions

## YouTube

FortiGate can enforce supported YouTube restriction modes.

```text
YouTube
   |
   +---- Strict
   |
   +---- Moderate
```

## Checklist

* [ ] YouTube restriction requirement identified.
* [ ] Restriction mode selected.
* [ ] Video filtering visibility enabled where required.
* [ ] FortiGuard classification tested.
* [ ] User experience tested.

## Feature Visibility

```bash
config system feature-visibility
    set video-filter enable
end
```

## Vimeo

Example:

```bash
config webfilter profile
    edit "wf-test"

        config web
            set vimeo-restrict 7
        end

    next
end
```

> **Important:** Verify supported values against the deployed FortiOS version.

---

# 18. Google Account Restriction

Google account restrictions can be used to limit access to approved Google account domains.

## Example

```text
user@gmail.com
     |
     +---- BLOCK

user@company.example
     |
     +---- ALLOW
```

## Checklist

* [ ] Organizational domain identified.
* [ ] Allowed domains configured.
* [ ] Personal account behavior tested.
* [ ] Corporate account behavior tested.
* [ ] Google service compatibility verified.

---

# 19. HTTP POST Control

HTTP POST is commonly used for:

* [ ] Web forms
* [ ] Credential submission
* [ ] File uploads
* [ ] Application data
* [ ] API requests

## Example

```bash
config webfilter profile
    edit "wf-test"
        set post-action normal
    next
end
```

Possible behavior:

```text
normal
block
```

## Deployment Checklist

* [ ] POST requirements identified.
* [ ] Legitimate applications tested.
* [ ] File uploads tested.
* [ ] Authentication portals tested.
* [ ] API functionality tested.
* [ ] Blocking impact documented.

> **Security vs Compatibility:**
> Blocking POST can reduce certain data-submission paths but can also break legitimate applications.

---

# 20. ActiveX & Java Applets

ActiveX and Java Applets are legacy browser technologies.

Potential legacy applications include:

* [ ] DVR portals
* [ ] CCTV systems
* [ ] Legacy management interfaces
* [ ] Internet Explorer-based applications

## Checklist

* [ ] Legacy dependencies identified.
* [ ] ActiveX requirements identified.
* [ ] Java Applet requirements identified.
* [ ] Blocking tested before global enforcement.
* [ ] Legacy applications migrated where possible.

> **Security Principle:**
> Prefer modern browser/application technologies and eliminate legacy components when operationally possible.

---

# 21. Cookie Filtering

Cookies can support:

* [ ] Session management
* [ ] Authentication
* [ ] User preferences
* [ ] Application state
* [ ] Tracking

## Checklist

* [ ] Cookie requirements identified.
* [ ] Authentication applications tested.
* [ ] Session behavior tested.
* [ ] Tracking requirements reviewed.
* [ ] Application compatibility tested.

> Aggressive cookie filtering can break authentication and modern web applications.

---

# 22. WAD Diagnostics

**WAD (Web Application Daemon)** is strongly associated with FortiGate proxy-based web processing.

## Diagnostic Checklist

* [ ] WAD process investigated.
* [ ] Correct VDOM selected.
* [ ] Web Filter statistics checked.
* [ ] FortiGuard statistics checked.
* [ ] FortiGuard cache checked.
* [ ] Override information checked.

## WAD Filters

### Root VDOM

```bash
diagnose wad filter vd root
```

### All VDOMs

```bash
diagnose wad filter vd all
```

## Web Filter Statistics

```bash
diagnose webfilter stats list root
```

## FortiGuard Statistics

```bash
diagnose webfilter fortiguard statistics
```

## FortiGuard Override

```bash
diagnose webfilter fortiguard override
```

## FortiGuard Cache

```bash
diagnose webfilter fortiguard cache
```

---

# 23. SSL Certificate Validation

SSL/TLS inspection can provide certificate visibility and, depending on the inspection mode, payload visibility.

Potential certificate problems include:

* [ ] Revoked certificate
* [ ] Untrusted certificate
* [ ] Invalid certificate
* [ ] Self-signed certificate
* [ ] Certificate name mismatch

## Inspection Model

```text
Client
  |
  v
FortiGate
  |
  +---- SSL/TLS Inspection
  |
  +---- Certificate Validation
  |
  +---- Web Filter
  |
  +---- IPS / AV / Other Inspection
  |
  v
Internet
```

## Certificate Inspection

```text
Certificate Inspection
        |
        +---- Certificate metadata visibility
        |
        +---- Limited payload visibility
```

## Deep Inspection

```text
Deep Inspection
      |
      +---- Decrypt
      |
      +---- Inspect
      |
      +---- Re-encrypt
```

## Checklist

* [ ] Correct SSL inspection profile selected.
* [ ] CA certificate configured.
* [ ] CA certificate trusted by managed clients.
* [ ] Certificate validation behavior tested.
* [ ] Invalid certificate behavior reviewed.
* [ ] Self-signed certificate requirements reviewed.
* [ ] Application compatibility tested.
* [ ] Privacy/legal requirements reviewed.

> **Important:** Deep Inspection requires appropriate client trust configuration and can introduce application compatibility issues.

---

# 24. Websense WISP

**Websense Integrated Services Protocol (WISP)** can integrate FortiGate with an external web-filtering service.

```text
Client
  |
  v
FortiGate
  |
  +---- Local URL Filter
  |
  +---- WISP
  |
  +---- FortiGuard
  |
  v
Final Decision
```

## WISP Checklist

* [ ] External filtering requirement identified.
* [ ] WISP servers configured.
* [ ] Connectivity tested.
* [ ] Algorithm selected.
* [ ] Logging configured.
* [ ] Failover behavior tested.
* [ ] Added latency evaluated.

## Example

```bash
config web-proxy wisp

    edit "wisp1"
        set server-ip 192.168.20.200
    next

    edit "wisp2"
        set server-ip 192.168.20.201
    next

end
```

## Enable WISP

```bash
config webfilter profile
    edit "wf-test"

        set feature-set flow
        set wisp enable
        set wisp-servers "wisp1" "wisp2"
        set wisp-algorithm round-robin
        set log-all-url enable

    next
end
```

## WISP Algorithms

```text
Primary-Secondary
    |
    +---- Primary
    |
    +---- Backup

Round-Robin
    |
    +---- WISP1
    +---- WISP2
    +---- WISP1
    +---- WISP2

Auto-Learning
    |
    +---- Learn performance
    |
    +---- Prefer suitable analysis path
```

> **Design Consideration:**
> External filtering introduces additional dependencies, latency, and failure domains.

---

# 25. HTTP/3 & QUIC

HTTP/3 operates over **QUIC**, while QUIC operates over **UDP**.

## HTTP/1.1

```text
HTTP/1.1
   |
   v
TCP
   |
   v
IP
```

## HTTP/2

```text
HTTP/2
   |
   v
TLS
   |
   v
TCP
   |
   v
IP
```

## HTTP/3

```text
HTTP/3
   |
   v
QUIC + TLS 1.3
   |
   v
UDP
   |
   v
IP
```

## Checklist

* [ ] HTTP/3 requirement identified.
* [ ] UDP/443 behavior reviewed.
* [ ] QUIC traffic identified.
* [ ] SSL inspection compatibility tested.
* [ ] Application behavior tested.
* [ ] QUIC blocking impact evaluated.

> ### 🧠 NSE Memory
>
> **HTTP/3 ≠ HTTP/2 over UDP**
>
> HTTP/3 is specifically designed to operate over QUIC.

---

# 26. HTTP/2 vs HTTP/3

| Feature                | HTTP/2    | HTTP/3           |
| ---------------------- | --------- | ---------------- |
| Transport              | TCP       | UDP              |
| Security               | TLS       | TLS 1.3 via QUIC |
| Multiplexing           | Yes       | Yes              |
| TCP-level HOL blocking | Yes       | No               |
| Transport handshake    | TCP + TLS | QUIC + TLS       |
| Common port            | TCP/443   | UDP/443          |
| Protocol               | HTTP/2    | HTTP/3           |

## HTTP/2 Stack

```text
HTTP/2
  |
TLS
  |
TCP
  |
IP
```

## HTTP/3 Stack

```text
HTTP/3
  |
QUIC
  |
UDP
  |
IP
```

## Exam Checklist

* [ ] HTTP/2 mapped to TCP.
* [ ] HTTP/3 mapped to QUIC.
* [ ] QUIC mapped to UDP.
* [ ] HTTP/3 mapped to TLS 1.3.
* [ ] TCP-level HOL distinction understood.

---

# 27. QUIC Packet Structure

A QUIC packet contains a header and one or more frames.

```text
+--------------------------------+
| QUIC Header                    |
+--------------------------------+
| Frame 1                        |
+--------------------------------+
| Frame 2                        |
+--------------------------------+
| Frame N                        |
+--------------------------------+
```

## Checklist

* [ ] Header identified.
* [ ] Frames understood.
* [ ] Packet numbering understood.
* [ ] Connection IDs understood.
* [ ] Streams understood.

---

# 28. QUIC Header

Important QUIC concepts include:

## Destination Connection ID

Used to identify the destination QUIC connection.

```text
Destination Connection ID
          |
          v
QUIC Connection
```

## Packet Number

Used as part of QUIC packet identification, loss detection, and recovery.

```text
Packet 1
Packet 2
Packet 3
Packet 4
```

## Spin Bit

The spin bit can assist passive RTT/latency measurement where implemented and enabled.

```text
Spin Bit
   |
   +---- Passive Measurement
   |
   +---- Network Monitoring
```

## Key Phase

The Key Phase bit indicates the packet-protection key phase.

```text
TLS 1.3
   |
   v
Packet Protection Keys
   |
   v
Key Phase
```

> **Exam Correction:**
> The QUIC header does **not** contain the actual encryption secret. TLS 1.3 establishes the cryptographic keys; QUIC uses protected packet headers and payloads.

---

# 29. QUIC Frames & Streams

## Frames

A QUIC packet can contain multiple frames.

```text
QUIC Packet
     |
     +---- Frame 1
     |
     +---- Frame 2
     |
     +---- Frame 3
```

## STREAM Frame

A STREAM frame can contain information such as:

```text
Stream ID
Offset
Stream Data
```

## Streams

A QUIC connection can support multiple independent streams.

```text
QUIC Connection
 |
 +---- Stream 1
 |
 +---- Stream 2
 |
 +---- Stream 3
 |
 +---- Stream N
```

Streams can be:

* [ ] Bidirectional
* [ ] Unidirectional

## Head-of-Line Behavior

If one stream experiences packet loss:

```text
Stream 1
   |
   +---- Packet Loss
```

other streams are not blocked by **TCP-level** head-of-line blocking.

> **NSE Memory:**
> QUIC provides independent streams within a connection, avoiding TCP's connection-wide transport-level HOL behavior.

---

# 30. QUIC vs TLS 1.3

| Feature             | QUIC                             | TLS 1.3                           |
| ------------------- | -------------------------------- | --------------------------------- |
| Primary role        | Secure transport + streams       | Cryptographic handshake/security  |
| Transport           | UDP                              | TCP or QUIC                       |
| Multiplexing        | Yes                              | No                                |
| Reliability         | Yes                              | No                                |
| Congestion control  | Yes                              | No                                |
| Packet protection   | Uses TLS-derived keys            | Provides cryptographic mechanisms |
| 0-RTT               | Supported when conditions permit | Supports 0-RTT mechanisms         |
| TCP HOL elimination | Yes, because QUIC is not TCP     | Not a transport feature           |

## QUIC Architecture

```text
QUIC
 |
 +---- Connection Management
 |
 +---- Reliability
 |
 +---- Congestion Control
 |
 +---- Streams
 |
 +---- Packet Protection
 |
 +---- TLS 1.3
```

> ### 🧠 Key Concept
>
> **QUIC uses TLS 1.3 for cryptographic protection, but QUIC itself provides transport functions such as streams, reliability, congestion control, and connection management.**

---

# 31. Web Filter Deployment Checklist

## Firewall Policy

* [ ] Correct source interface selected.
* [ ] Correct destination interface selected.
* [ ] Correct source addresses configured.
* [ ] Correct destination addresses configured.
* [ ] Correct service/application requirements reviewed.
* [ ] Inspection mode verified.
* [ ] Web Filter profile attached.
* [ ] SSL inspection profile attached where required.
* [ ] Logging enabled.
* [ ] Policy order verified.

## URL Filtering

* [ ] Block URLs configured.
* [ ] Allow URLs configured.
* [ ] Monitor rules configured where required.
* [ ] Wildcard syntax verified.
* [ ] Regex syntax verified.
* [ ] Rule order reviewed.
* [ ] Exempt rules reviewed.
* [ ] Anti-phishing interactions tested.

## FortiGuard

* [ ] FortiGuard service status verified.
* [ ] FortiGuard connectivity verified.
* [ ] Categories reviewed.
* [ ] Category actions configured.
* [ ] Rating failure behavior reviewed.
* [ ] IP rating evaluated.
* [ ] Category overrides reviewed.
* [ ] FortiGuard logs verified.

## Content Filter

* [ ] Banned words configured.
* [ ] Pattern types verified.
* [ ] Scores reviewed.
* [ ] Threshold configured.
* [ ] False positives tested.
* [ ] Legitimate applications tested.

## Anti-Phishing

* [ ] Authentication integration verified.
* [ ] Anti-phishing enabled.
* [ ] URI inspection reviewed.
* [ ] Basic authentication inspection reviewed.
* [ ] Username inspection reviewed.
* [ ] Custom patterns reviewed.
* [ ] Logging enabled.
* [ ] Block behavior tested.

## Safe Search

* [ ] Safe Search configured.
* [ ] Supported engines tested.
* [ ] Search logging evaluated.
* [ ] YouTube restriction tested.
* [ ] Vimeo restriction tested where required.
* [ ] Privacy requirements reviewed.

## SSL Inspection

* [ ] Certificate Inspection evaluated.
* [ ] Deep Inspection evaluated.
* [ ] CA certificate configured.
* [ ] CA certificate trusted by clients.
* [ ] Certificate errors tested.
* [ ] Application compatibility tested.
* [ ] Privacy requirements reviewed.

## HTTP/3 / QUIC

* [ ] UDP/443 reviewed.
* [ ] HTTP/3 application requirements identified.
* [ ] QUIC traffic identified.
* [ ] Deep Inspection compatibility tested.
* [ ] Application behavior validated.
* [ ] QUIC blocking impact assessed.
* [ ] Blocking QUIC used only when operationally justified.

---

# 32. Troubleshooting Checklist

## Step 1 — Firewall Policy

* [ ] Is the expected firewall policy matching?
* [ ] Is the correct source/destination selected?
* [ ] Is the correct inspection mode enabled?
* [ ] Is the Web Filter profile attached?

## Step 2 — SSL Inspection

* [ ] Is HTTPS traffic visible?
* [ ] Is the correct SSL profile applied?
* [ ] Is the client trusting the inspection CA?
* [ ] Are certificate errors present?

## Step 3 — URL Filter

* [ ] Does the URL match a local rule?
* [ ] Is the matching type correct?
* [ ] Is the rule action Allow/Block/Monitor/Exempt?
* [ ] Is rule ordering correct?

## Step 4 — FortiGuard

* [ ] Can FortiGate reach FortiGuard?
* [ ] Is the domain rated?
* [ ] Is the expected category returned?
* [ ] Is rating failure occurring?
* [ ] Is `error-allow` configured?

## Step 5 — Content Filter

* [ ] Is content filtering enabled?
* [ ] Is the word/pattern matched?
* [ ] Is the score sufficient?
* [ ] Is the threshold correct?

## Step 6 — Anti-Phishing

* [ ] Is anti-phishing enabled?
* [ ] Did an earlier engine block the request?
* [ ] Is credential inspection configured?
* [ ] Are logs available?

## Step 7 — WAD

```bash
diagnose wad filter vd root
diagnose wad filter vd all
```

## Step 8 — Web Filter Statistics

```bash
diagnose webfilter stats list root
```

## Step 9 — FortiGuard

```bash
diagnose webfilter fortiguard statistics
diagnose webfilter fortiguard override
diagnose webfilter fortiguard cache
```

## Troubleshooting Flow

```text
Client
  |
  v
Firewall Policy
  |
  +---- Inspection Mode?
  |
  +---- Web Filter Profile?
  |
  +---- SSL Inspection?
  |
  +---- URL Filter?
  |
  +---- FortiGuard Rating?
  |
  +---- Content Filter?
  |
  +---- Anti-Phishing?
  |
  +---- WISP?
  |
  +---- WAD?
  |
  v
ALLOW / BLOCK
```

---

# 33. NSE Exam Traps

## Trap #1 — Allow vs Exempt

```text
ALLOW
  |
  +---- Continue applicable inspection

EXEMPT
  |
  +---- Bypass remaining applicable inspection
```

* [ ] Can you explain the difference without looking at notes?

---

## Trap #2 — URL Filter vs FortiGuard

```text
URL Filter
  |
  +---- Administrator-defined URL patterns

FortiGuard
  |
  +---- Fortinet-maintained category/rating intelligence
```

* [ ] Can you distinguish local URL matching from category-based filtering?

---

## Trap #3 — Content Filter

```text
URL Filter
  |
  +---- URL / URL Pattern

Content Filter
  |
  +---- Web Content
```

* [ ] Do not confuse destination matching with content inspection.

---

## Trap #4 — Rating Error

```text
error-allow
    |
    +---- Allow when rating cannot be obtained
```

* [ ] Remember the availability/security trade-off.

---

## Trap #5 — HTTP/3

```text
HTTP/3
  |
  +---- QUIC
          |
          +---- UDP
```

While:

```text
HTTP/2
  |
  +---- TCP
```

* [ ] HTTP/3 is not simply HTTP/2 over UDP.

---

## Trap #6 — QUIC Encryption

```text
QUIC
  +
TLS 1.3
```

* [ ] QUIC uses TLS 1.3 cryptographic mechanisms.
* [ ] TLS 1.3 is not equivalent to QUIC.
* [ ] QUIC provides transport functionality.

---

## Trap #7 — QUIC Streams

```text
1 QUIC Connection
       |
       +---- Stream 1
       +---- Stream 2
       +---- Stream 3
       +---- Stream N
```

* [ ] Understand independent streams.
* [ ] Understand TCP-level HOL avoidance.

---

## Trap #8 — Deep Inspection

```text
Certificate Inspection
        |
        +---- Certificate visibility

Deep Inspection
        |
        +---- Decrypt
        +---- Inspect
        +---- Re-encrypt
```

* [ ] Client trust requirements understood.
* [ ] Compatibility impact understood.

---

## Trap #9 — Proxy Mode

```text
Flow-Based
   |
   +---- Lower processing overhead
   +---- More limited proxy capabilities

Proxy-Based
   |
   +---- Full proxy processing
   +---- Required by some advanced web-filtering features
```

* [ ] Know which web-filtering functions require proxy processing.

---

## Trap #10 — Security vs Compatibility

Aggressive web filtering can affect:

* [ ] Legacy applications
* [ ] Google services
* [ ] YouTube
* [ ] HTTP/3
* [ ] QUIC
* [ ] DVR/CCTV portals
* [ ] Authentication portals
* [ ] File uploads
* [ ] APIs

> **Engineering Principle:**
> A security control is not production-ready until its compatibility impact has been tested.

---

# 34. Quick Revision

```text
WEB FILTER
|
+-- URL FILTER
|   |
|   +-- Allow
|   +-- Block
|   +-- Monitor
|   +-- Exempt
|
+-- FORTIGUARD
|   |
|   +-- Category
|   +-- Rating
|   +-- Reputation
|
+-- CONTENT FILTER
|   |
|   +-- Words
|   +-- Score
|   +-- Threshold
|
+-- ANTI-PHISHING
|   |
|   +-- URI
|   +-- Username
|   +-- Password
|
+-- SAFE SEARCH
|   |
|   +-- Google
|   +-- Bing
|   +-- Yahoo
|   +-- Yandex
|
+-- FORTISANDBOX
|   |
|   +-- Malicious URL Intelligence
|
+-- WISP
|   |
|   +-- External Web Filtering
|
+-- SSL INSPECTION
    |
    +-- Certificate Inspection
    +-- Deep Inspection
```

---

# ⚡ HTTP/3 Quick Card

```text
HTTP/3
   |
   v
QUIC
   |
   v
UDP
   |
   v
IP
```

## QUIC

```text
QUIC
|
+-- Connection ID
|
+-- Packet Number
|
+-- Frames
|
+-- Streams
|
+-- Reliability
|
+-- Congestion Control
|
+-- TLS 1.3
```

---

# 🧠 One-Line NSE Memory Aid

> **URL Filter matches administrator-defined URL patterns; FortiGuard provides category/rating intelligence; Content Filter evaluates web content; Anti-Phishing analyzes credential-related behavior; SSL Deep Inspection provides visibility into encrypted traffic; HTTP/3 operates over QUIC, and QUIC operates over UDP with TLS 1.3 cryptographic protection.**

---

# 🔥 Fast Comparison

| Feature                       | Main Question                                              |
| ----------------------------- | ---------------------------------------------------------- |
| **URL Filter**                | Does this URL/pattern match?                               |
| **FortiGuard**                | What category/rating does this destination have?           |
| **Content Filter**            | What words/patterns exist inside web content?              |
| **Anti-Phishing**             | Is credential-related behavior suspicious?                 |
| **Safe Search**               | Should search results be restricted?                       |
| **FortiSandbox URL Blocking** | Is the URL associated with malicious intelligence?         |
| **WISP**                      | Should an external filtering service evaluate the request? |
| **SSL Inspection**            | Can FortiGate inspect encrypted traffic?                   |
| **HTTP/3**                    | Is HTTP operating over QUIC?                               |
| **QUIC**                      | Is the transport using UDP with TLS 1.3-based protection?  |

---

# 35. Final Mental Model

```text
                         WEB REQUEST
                              |
                              v
                     +----------------+
                     | Firewall Policy|
                     +----------------+
                              |
                              v
                     +----------------+
                     | SSL Inspection |
                     +----------------+
                              |
                              v
                     +----------------+
                     |   URL Filter   |
                     +----------------+
                              |
                              v
                     +----------------+
                     |   FortiGuard   |
                     |     Rating     |
                     +----------------+
                              |
                              v
                     +----------------+
                     | Content Filter |
                     +----------------+
                              |
                              v
                     +----------------+
                     | Anti-Phishing  |
                     +----------------+
                              |
                              v
                     +----------------+
                     |      WAD       |
                     +----------------+
                              |
                              v
                       ALLOW / BLOCK
```

---

# 📌 Related FortiGate Topics

* [ ] FortiGate Firewall Policy
* [ ] FortiGate Proxy-Based Inspection
* [ ] FortiGate SSL Deep Inspection
* [ ] FortiGuard Web Filtering
* [ ] FortiGate Anti-Phishing
* [ ] FortiGate Antivirus
* [ ] FortiGate FortiSandbox
* [ ] FortiGate WAD
* [ ] HTTP/2
* [ ] HTTP/3
* [ ] QUIC
* [ ] TLS 1.3

---

# 🎯 Final Deployment Validation

Before considering the Web Filter deployment complete:

* [ ] Firewall policy validated.
* [ ] Inspection mode validated.
* [ ] SSL inspection validated.
* [ ] URL Filter validated.
* [ ] FortiGuard categorization validated.
* [ ] Content Filter validated.
* [ ] Anti-phishing validated.
* [ ] Safe Search validated.
* [ ] YouTube/Vimeo behavior validated.
* [ ] WISP behavior validated where applicable.
* [ ] HTTP/3/QUIC behavior validated.
* [ ] Logging validated.
* [ ] WAD diagnostics tested.
* [ ] False positives reviewed.
* [ ] Application compatibility tested.
* [ ] Security exceptions documented.
* [ ] Troubleshooting procedure documented.

> ## 🛡️ SheynShield Engineering Principle
>
> **Do not measure a Web Filter deployment only by what it blocks. Measure it by whether it provides the required security visibility and control without breaking legitimate business traffic.**

---

# 🔗 SheynShield Resources

# FortiGate WAF Security Checklist

> **FortiOS Focus:** Web Application Firewall (WAF) · VIP · DNAT · Reverse Proxy · HTTP/HTTPS · Proxy Inspection · SSL Inspection · IPS · Security Fabric
> **Audience:** FortiGate · NSE4 · NSE5 · NSE6 · NSE7 · Network & Security Engineers
> **Primary Use Cases:** Web Application Protection · SQL Injection · HTTP Attacks · Brute-Force Mitigation · Internet-Facing Applications
> **Inspection Mode:** Proxy-Based
> **Methodology:** Design → Configure → Secure → Verify → Troubleshoot

---

## 📋 Table of Contents

* [1. WAF Deployment Readiness](#1-waf-deployment-readiness)
* [2. Enable WAF Feature](#2-enable-waf-feature)
* [3. Web Application Architecture](#3-web-application-architecture)
* [4. VIP / DNAT Checklist](#4-vip--dnat-checklist)
* [5. Firewall Policy Checklist](#5-firewall-policy-checklist)
* [6. Proxy Inspection Checklist](#6-proxy-inspection-checklist)
* [7. HTTPS / SSL Inspection Checklist](#7-https--ssl-inspection-checklist)
* [8. WAF Profile Checklist](#8-waf-profile-checklist)
* [9. WAF Security Controls](#9-waf-security-controls)
* [10. WAF vs IPS](#10-waf-vs-ips)
* [11. SQL Injection Protection](#11-sql-injection-protection)
* [12. Brute-Force Protection](#12-brute-force-protection)
* [13. Security Fabric Integration](#13-security-fabric-integration)
* [14. Logging & Monitoring](#14-logging--monitoring)
* [15. Production Hardening](#15-production-hardening)
* [16. WAF Troubleshooting](#16-waf-troubleshooting)
* [17. False Positive Troubleshooting](#17-false-positive-troubleshooting)
* [18. HTTPS Troubleshooting](#18-https-troubleshooting)
* [19. VIP Troubleshooting](#19-vip-troubleshooting)
* [20. WAF Verification Checklist](#20-waf-verification-checklist)
* [21. Common WAF Design Mistakes](#21-common-waf-design-mistakes)
* [22. Production Reference Architecture](#22-production-reference-architecture)
* [23. Fast NSE Exam Checklist](#23-fast-nse-exam-checklist)
* [24. One-Minute Revision](#24-one-minute-revision)
* [25. Final WAF Mental Model](#25-final-waf-mental-model)

---

# 1. WAF Deployment Readiness

Before configuring WAF, verify the application architecture.

### Environment

* [ ] Identify the protected web application
* [ ] Identify the backend web server
* [ ] Identify application IP address
* [ ] Identify application FQDN
* [ ] Identify public IP
* [ ] Identify FortiGate ingress interface
* [ ] Identify FortiGate egress/backend interface
* [ ] Confirm routing
* [ ] Confirm return routing
* [ ] Identify HTTP/HTTPS services
* [ ] Identify application-specific ports
* [ ] Identify application owners
* [ ] Identify expected client sources

### Application Requirements

* [ ] HTTP required?
* [ ] HTTPS required?
* [ ] HTTP → HTTPS redirect required?
* [ ] WebSocket required?
* [ ] Large file uploads required?
* [ ] API endpoints present?
* [ ] Authentication/login portal present?
* [ ] Special HTTP methods required?
* [ ] Custom headers required?
* [ ] Large request bodies required?

> **Production Rule:** Understand the application before enabling aggressive WAF controls. Security policy must protect the application without breaking legitimate business traffic.

---

# 2. Enable WAF Feature

If WAF configuration is not visible in the GUI:

```text
System
   ↓
Feature Visibility
   ↓
Web Application Firewall
```

Checklist:

* [ ] Open Feature Visibility
* [ ] Locate Web Application Firewall
* [ ] Enable WAF
* [ ] Confirm WAF configuration is visible
* [ ] Confirm the deployed FortiOS release supports the required WAF features

> **NSE Note:** Feature availability and exact GUI/CLI behavior can vary between FortiOS releases.

---

# 3. Web Application Architecture

A common Internet-facing WAF design:

```text
                         INTERNET
                            │
                            │ HTTP / HTTPS
                            ▼
                    ┌─────────────────┐
                    │    FortiGate    │
                    │                 │
                    │      VIP        │
                    │       │         │
                    │  Firewall Policy│
                    │       │         │
                    │     Proxy       │
                    │       │         │
                    │ SSL Inspection  │
                    │       │         │
                    │      WAF        │
                    │       │         │
                    │      IPS        │
                    └───────┬─────────┘
                            │
                            ▼
                    Web Application
                      192.168.20.200
```

### Architecture Checklist

* [ ] Internet-facing interface identified
* [ ] Public address identified
* [ ] VIP configured
* [ ] Backend server reachable
* [ ] Firewall policy created
* [ ] Proxy inspection selected
* [ ] WAF profile attached
* [ ] SSL inspection evaluated for HTTPS
* [ ] IPS evaluated
* [ ] Logging enabled
* [ ] Backend return path verified

---

# 4. VIP / DNAT Checklist

The VIP provides the publishing mechanism for the internal application.

### VIP Checklist

* [ ] External IP is correct
* [ ] Mapped IP is correct
* [ ] Correct interface is selected
* [ ] Correct port mapping is configured
* [ ] HTTP mapping verified if required
* [ ] HTTPS mapping verified if required
* [ ] VIP is referenced by the correct firewall policy
* [ ] No conflicting VIP exists
* [ ] VIP matching behavior verified
* [ ] Backend server is listening
* [ ] Return route is correct

Example:

```text
Public IP
192.168.101.101
       │
       │ VIP / DNAT
       ▼
Private Web Server
192.168.20.200
```

> The IP addresses above are documentation/lab examples.

### VIP Security Checklist

* [ ] Do not expose unnecessary ports
* [ ] Do not publish unnecessary backend services
* [ ] Use dedicated VIPs where practical
* [ ] Avoid overlapping/conflicting VIP objects
* [ ] Restrict access at the firewall-policy layer

---

# 5. Firewall Policy Checklist

The firewall policy is the access-control layer.

```text
Client
   │
   ▼
VIP
   │
   ▼
Firewall Policy
   │
   ├── Proxy Inspection
   ├── WAF
   ├── SSL Inspection
   ├── IPS
   └── Logging
   │
   ▼
Web Server
```

### Source

* [ ] Source is restricted where business requirements permit
* [ ] Known partner networks identified
* [ ] Trusted application consumers identified
* [ ] Internet exposure is intentional
* [ ] `ALL` source is avoided when unnecessary

### Destination

* [ ] Destination is the specific Web VIP
* [ ] No unrestricted internal destination
* [ ] VIP is correctly matched

### Services

* [ ] HTTP allowed only if required
* [ ] HTTPS allowed only if required
* [ ] Application-specific services identified
* [ ] `ALL` services avoided unless explicitly required

### Inspection

* [ ] Proxy inspection selected
* [ ] WAF profile attached
* [ ] IPS profile evaluated
* [ ] SSL inspection evaluated
* [ ] Logging enabled

### Policy Security

* [ ] Policy order verified
* [ ] More-specific policies placed appropriately
* [ ] Shadowing policies checked
* [ ] Implicit deny understood
* [ ] Unused policies reviewed

---

# 6. Proxy Inspection Checklist

WAF relies on application-aware processing.

```text
HTTP / HTTPS
     │
     ▼
Proxy Processing
     │
     ├── HTTP Parsing
     ├── Application Inspection
     └── WAF
```

Checklist:

* [ ] Firewall policy uses supported inspection mode
* [ ] Proxy-based inspection verified
* [ ] WAF profile compatible with inspection mode
* [ ] Application traffic successfully reaches proxy
* [ ] HTTP parsing verified
* [ ] HTTPS handling verified
* [ ] Resource utilization monitored

### Proxy Mental Model

```text
Flow
 │
 └── IPS-oriented processing

Proxy
 │
 └── Proxy-based application processing
```

> **Important:** Exact feature support depends on FortiOS version and configuration.

---

# 7. HTTPS / SSL Inspection Checklist

HTTPS encrypts the application payload.

```text
Client
   │
   │ HTTPS
   ▼
FortiGate
   │
   ▼
SSL Inspection
   │
   ▼
Application Visibility
   │
   ▼
WAF
   │
   ▼
Web Server
```

### HTTPS Checklist

* [ ] HTTPS certificate requirements identified
* [ ] SSL inspection requirement evaluated
* [ ] Appropriate SSL inspection profile selected
* [ ] Certificate trust requirements understood
* [ ] Client compatibility tested
* [ ] Backend TLS requirements tested
* [ ] Application functionality tested
* [ ] CPU impact monitored
* [ ] Memory impact monitored
* [ ] Latency impact evaluated

### Security Trade-Off

```text
SSL Inspection
      │
      ▼
More Visibility
      │
      ▼
More Processing
      │
      ▼
Potential Resource Impact
```

> Do not enable deep inspection blindly in production. Validate certificate behavior, application compatibility, latency, and resource consumption.

---

# 8. WAF Profile Checklist

Create a dedicated WAF profile for the application.

### Profile

* [ ] WAF profile created
* [ ] Profile name is descriptive
* [ ] Profile is attached to the correct firewall policy
* [ ] Appropriate protection categories enabled
* [ ] Logging configured
* [ ] Application-specific exceptions identified
* [ ] False-positive strategy defined

### Policy Model

```text
Firewall Policy
      │
      ├── Proxy Inspection
      │
      ├── WAF Profile
      │
      ├── SSL Inspection
      │
      └── IPS
```

---

# 9. WAF Security Controls

Evaluate the following controls according to the application and FortiOS release:

* [ ] HTTP protocol validation
* [ ] Malformed request detection
* [ ] SQL injection protection
* [ ] HTTP attack protection
* [ ] Parameter inspection
* [ ] Header inspection
* [ ] Request-body inspection
* [ ] URL inspection
* [ ] Request-size controls
* [ ] Rate/abuse controls where supported
* [ ] Brute-force protections where applicable
* [ ] Logging
* [ ] Alerting
* [ ] Exception handling

### WAF Processing Model

```text
HTTP Request
      │
      ▼
HTTP Parsing
      │
      ▼
WAF Inspection
      │
 ┌────┴────┐
 ▼         ▼
Allow     Block
 │         │
 ▼         X
Web       Log
Server
```

---

# 10. WAF vs IPS

Do not treat WAF and IPS as identical security controls.

| Control                  | Primary Question                               |
| ------------------------ | ---------------------------------------------- |
| **Firewall**             | Who can connect?                               |
| **VIP**                  | Where should traffic be translated/published?  |
| **SSL Inspection**       | Can encrypted traffic be inspected?            |
| **IPS**                  | Does traffic match malicious/exploit patterns? |
| **WAF**                  | Is this HTTP/HTTPS request malicious?          |
| **Application Security** | Is the application securely designed?          |

### Mental Model

```text
Firewall
   ↓
Can the connection pass?

SSL Inspection
   ↓
Can encrypted content be inspected?

WAF
   ↓
Is the web request malicious?

IPS
   ↓
Does traffic match supported malicious patterns?

Application
   ↓
Is the application itself secure?
```

### Checklist

* [ ] Firewall used for access control
* [ ] VIP used for publishing
* [ ] SSL inspection used where appropriate
* [ ] WAF used for web-layer protection
* [ ] IPS evaluated as complementary protection
* [ ] Secure coding remains an application responsibility

---

# 11. SQL Injection Protection

SQL Injection is a major WAF use case.

Example:

```http
GET /product?id=100
```

Potentially malicious:

```http
GET /product?id=100' OR '1'='1
```

### SQLi Checklist

* [ ] SQL injection protection enabled where supported
* [ ] Request parameters inspected
* [ ] WAF events logged
* [ ] False positives monitored
* [ ] Application-specific exceptions documented
* [ ] Exceptions scoped as narrowly as possible
* [ ] Application-side parameterized queries implemented
* [ ] Database permissions minimized

### Protection Flow

```text
HTTP Request
      │
      ▼
     WAF
      │
      ├── Parse
      ├── Inspect
      ├── Match
      └── Action
           │
      ┌────┴────┐
      ▼         ▼
    Allow     Block
```

> **Important:** WAF reduces risk but does not replace secure application development.

---

# 12. Brute-Force Protection

Web applications frequently expose authentication endpoints.

Example:

```text
Attacker
   │
   ├── Login Attempt
   ├── Login Attempt
   ├── Login Attempt
   ├── Login Attempt
   └── ...
        │
        ▼
       WAF
        │
        ▼
 Detection / Policy
```

### Checklist

* [ ] Login endpoint identified
* [ ] Repeated authentication attempts monitored
* [ ] Application rate limiting evaluated
* [ ] WAF controls evaluated
* [ ] MFA enabled where appropriate
* [ ] Account lockout/rate limiting evaluated
* [ ] Authentication logs monitored
* [ ] Dedicated authentication protection considered

> WAF can contribute to application-layer abuse protection, but authentication controls should primarily be designed as part of the application/security architecture.

---

# 13. Security Fabric Integration

For Security Fabric deployments:

```text
                  Security Fabric
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
      FortiGate        Fabric       Security
        WAF           Components     Services
          │              │              │
          └──────────────┼──────────────┘
                         ▼
                  Central Visibility
```

Checklist:

* [ ] FortiGate integrated into Security Fabric where required
* [ ] Relevant Fabric components identified
* [ ] Security telemetry evaluated
* [ ] Event visibility verified
* [ ] Centralized monitoring evaluated
* [ ] Integration permissions reviewed
* [ ] FortiOS/product compatibility verified

> Available Fabric integrations vary by FortiOS version and deployed Fortinet products.

---

# 14. Logging & Monitoring

A WAF without visibility is difficult to operate securely.

### Logging Checklist

* [ ] WAF events logged
* [ ] Blocked requests logged
* [ ] Violations logged
* [ ] Security events correlated
* [ ] Firewall policy logs enabled where required
* [ ] SSL inspection events monitored where required
* [ ] IPS events monitored
* [ ] Application errors correlated with WAF events

### Investigate

```text
Timestamp
Source IP
Destination VIP
URL
HTTP Method
HTTP Headers
Parameters
Request Body
Matched Rule
Action
Policy
WAF Profile
```

### Operational Rule

```text
Detect
  ↓
Log
  ↓
Investigate
  ↓
Tune
  ↓
Verify
```

---

# 15. Production Hardening

## Exposure

* [ ] Public exposure is business-required
* [ ] Source restriction implemented where possible
* [ ] Only required services published
* [ ] Unused VIPs removed
* [ ] Unused policies removed

## Firewall

* [ ] Least privilege applied
* [ ] Specific VIP used
* [ ] Specific services used
* [ ] Policy order verified
* [ ] Logging enabled

## WAF

* [ ] WAF enabled
* [ ] Appropriate protections enabled
* [ ] False positives monitored
* [ ] Exceptions minimized
* [ ] Exceptions documented

## SSL

* [ ] HTTPS inspection requirement evaluated
* [ ] Certificate deployment tested
* [ ] Application compatibility tested
* [ ] Resource impact monitored

## IPS

* [ ] IPS profile evaluated
* [ ] Appropriate signatures/policies configured
* [ ] Events monitored

## Application

* [ ] Secure coding practices implemented
* [ ] Patching maintained
* [ ] Authentication hardened
* [ ] MFA evaluated
* [ ] Database security implemented
* [ ] Application logging enabled

---

# 16. WAF Troubleshooting

## Web Application Is Not Reachable

Follow the traffic chain:

```text
Client
  │
  ▼
Public IP
  │
  ▼
VIP
  │
  ▼
Firewall Policy
  │
  ▼
Proxy
  │
  ▼
SSL Inspection
  │
  ▼
WAF
  │
  ▼
IPS
  │
  ▼
Web Server
```

### Checklist

* [ ] Client can reach public IP
* [ ] VIP exists
* [ ] External IP is correct
* [ ] Mapped IP is correct
* [ ] VIP matches traffic
* [ ] Firewall policy matches
* [ ] Service is allowed
* [ ] Proxy inspection is correct
* [ ] WAF profile is attached
* [ ] SSL inspection is correct
* [ ] IPS is not blocking
* [ ] Routing is correct
* [ ] Return traffic is correct
* [ ] Web server is listening

---

# 17. False Positive Troubleshooting

If legitimate traffic is blocked:

```text
Client
   │
   ▼
 WAF
   │
   X
Blocked
```

### Investigate

* [ ] WAF event identified
* [ ] Matched rule identified
* [ ] Source identified
* [ ] Destination identified
* [ ] URL identified
* [ ] HTTP method identified
* [ ] Header inspected
* [ ] Parameter inspected
* [ ] Request body inspected
* [ ] Application behavior checked

### Remediation

```text
Identify Rule
     ↓
Understand Why It Matched
     ↓
Validate Application Behavior
     ↓
Tune / Exception
     ↓
Retest
     ↓
Monitor
```

### Avoid

```text
False Positive
      ↓
Disable WAF
```

### Prefer

```text
False Positive
      ↓
Identify Specific Rule
      ↓
Create Narrow Exception
      ↓
Retest
```

---

# 18. HTTPS Troubleshooting

If HTTP works but HTTPS fails:

### Checklist

* [ ] HTTPS VIP exists
* [ ] TCP/443 is allowed
* [ ] Correct certificate configured
* [ ] Client trusts certificate where required
* [ ] SSL inspection policy is correct
* [ ] TLS version compatibility checked
* [ ] Backend TLS checked
* [ ] WAF receives decrypted application traffic where required
* [ ] SSL inspection logs reviewed
* [ ] WAF logs reviewed

### Troubleshooting Flow

```text
HTTPS Failure
     │
     ▼
TCP/443
     │
     ▼
Certificate
     │
     ▼
TLS Handshake
     │
     ▼
SSL Inspection
     │
     ▼
HTTP Visibility
     │
     ▼
WAF
```

---

# 19. VIP Troubleshooting

When traffic never reaches the backend:

```text
Internet
   │
   ▼
Public IP
   │
   ▼
VIP
   │
   ▼
DNAT
   │
   ▼
Backend Server
```

### Checklist

* [ ] Public IP correct
* [ ] VIP enabled
* [ ] VIP interface correct
* [ ] External port correct
* [ ] Mapped port correct
* [ ] Mapped IP correct
* [ ] Firewall policy references VIP
* [ ] Policy order correct
* [ ] Backend listening
* [ ] Backend route correct
* [ ] Return traffic correct

---

# 20. WAF Verification Checklist

After deployment, perform controlled tests.

### Connectivity

* [ ] HTTP test completed
* [ ] HTTPS test completed
* [ ] Correct VIP reached
* [ ] Backend response verified

### WAF

* [ ] WAF profile confirmed
* [ ] Legitimate request allowed
* [ ] Controlled malicious test generated in an authorized environment
* [ ] WAF detection verified
* [ ] Block/log action verified
* [ ] Event appears in logs

### SSL

* [ ] TLS handshake verified
* [ ] Certificate behavior verified
* [ ] Application functionality verified

### Performance

* [ ] CPU monitored
* [ ] Memory monitored
* [ ] Latency monitored
* [ ] Application response time verified

### Operations

* [ ] Logging verified
* [ ] Monitoring verified
* [ ] Alerting verified
* [ ] Exception documentation completed

---

# 21. Common WAF Design Mistakes

## ❌ Mistake 1 — Publishing a Web Server Without WAF

```text
Internet
   │
   ▼
VIP
   │
   ▼
Web Server
```

Prefer:

```text
Internet
   │
   ▼
VIP
   │
   ▼
Firewall Policy
   │
   ▼
Proxy
   │
   ▼
WAF
   │
   ▼
Web Server
```

---

## ❌ Mistake 2 — `ALL` Services

```text
Source      : ALL
Destination : Web VIP
Service     : ALL
```

Prefer:

```text
Source      : Required
Destination : Web VIP
Service     : HTTP / HTTPS
```

---

## ❌ Mistake 3 — Ignoring HTTPS Encryption

```text
HTTPS
  │
  ▼
Encrypted Application Data
  │
  ▼
Limited Visibility
```

For deployments requiring application inspection:

```text
HTTPS
  │
  ▼
SSL Inspection
  │
  ▼
WAF
```

---

## ❌ Mistake 4 — Disabling WAF Because of One False Positive

```text
False Positive
      ↓
Disable WAF
```

Prefer:

```text
False Positive
      ↓
Identify Rule
      ↓
Tune Narrowly
      ↓
Retest
```

---

## ❌ Mistake 5 — Treating WAF as Complete Security

WAF does **not** replace:

* [ ] Secure coding
* [ ] Patch management
* [ ] Authentication security
* [ ] MFA
* [ ] IPS
* [ ] Firewall policy
* [ ] Endpoint security
* [ ] Database security
* [ ] Logging
* [ ] Monitoring

---

# 22. Production Reference Architecture

```text
                         INTERNET
                            │
                            ▼
                     Public Web IP
                            │
                            ▼
                     ┌────────────┐
                     │ FortiGate  │
                     │            │
                     │    VIP     │
                     │     │      │
                     │ Firewall   │
                     │   Policy   │
                     │     │      │
                     │   Proxy    │
                     │     │      │
                     │ SSL Inspect│
                     │     │      │
                     │    WAF     │
                     │     │      │
                     │    IPS     │
                     └─────┬──────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │ Web Application │
                  │ 192.168.20.200  │
                  └─────────────────┘
```

### Security Layers

```text
Public IP
    ↓
VIP
    ↓
Firewall Policy
    ↓
Proxy Inspection
    ↓
SSL Inspection
    ↓
WAF
    ↓
IPS
    ↓
Web Application
    ↓
Logging / Monitoring
```

---

# 23. Fast NSE Exam Checklist

| Topic               | Remember                                                    |
| ------------------- | ----------------------------------------------------------- |
| **WAF**             | Application-layer web protection                            |
| **VIP**             | Publishes/maps external address to backend                  |
| **DNAT**            | Translates destination address/port                         |
| **Proxy**           | Proxy-based application processing                          |
| **HTTPS**           | Encrypted application traffic                               |
| **SSL Inspection**  | Provides visibility into encrypted traffic where applicable |
| **SQLi**            | Major WAF use case                                          |
| **IPS**             | Complementary security control                              |
| **Firewall**        | Access control                                              |
| **WAF**             | HTTP/HTTPS request inspection                               |
| **False Positive**  | Investigate and tune instead of disabling WAF               |
| **Security Fabric** | Security integration and visibility                         |
| **Production**      | Least privilege + defense in depth                          |

---

# 24. One-Minute Revision

```text
PUBLIC WEB APPLICATION
        │
        ▼
      PUBLIC IP
        │
        ▼
       VIP
        │
        ▼
 FIREWALL POLICY
        │
        ▼
      PROXY
        │
        ▼
 SSL INSPECTION
        │
        ▼
       WAF
        │
        ▼
       IPS
        │
        ▼
   WEB SERVER
```

Remember:

```text
Firewall
   ↓
WHO can connect?

VIP
   ↓
WHERE does traffic go?

SSL Inspection
   ↓
CAN encrypted traffic be inspected?

WAF
   ↓
WHAT is the web request trying to do?

IPS
   ↓
DOES traffic match malicious patterns?

Application
   ↓
IS the application securely designed?
```

---

# 25. Final WAF Mental Model

```text
                         FORTIGATE WAF
                                │
                                ▼
                           PUBLIC IP
                                │
                                ▼
                               VIP
                                │
                                ▼
                        FIREWALL POLICY
                                │
                                ▼
                         PROXY INSPECTION
                                │
                                ▼
                         SSL INSPECTION
                                │
                                ▼
                               WAF
                                │
                    ┌───────────┴───────────┐
                    ▼                       ▼
                  ALLOW                    BLOCK
                    │                       │
                    ▼                       X
              WEB APPLICATION             LOG
                    │
                    ▼
                   IPS
                    │
                    ▼
             LOGGING / MONITORING
```

## 🧠 Golden Troubleshooting Rule

When a web application fails, **do not immediately blame WAF**.

Follow the chain:

```text
Client
  ↓
Public IP
  ↓
VIP
  ↓
Firewall Policy
  ↓
Proxy
  ↓
SSL Inspection
  ↓
WAF
  ↓
IPS
  ↓
Web Server
  ↓
Return Traffic
```

Find the **first layer where the expected behavior stops**.

---

# 🎯 SheynShield Engineering Rule

> **WAF is not another firewall rule.**

The firewall answers:

```text
"WHO can connect?"
```

The WAF answers:

```text
"WHAT is this HTTP/HTTPS request trying to do?"
```

For production troubleshooting, remember:

```text
CLIENT
  ↓
VIP
  ↓
FIREWALL
  ↓
PROXY
  ↓
SSL INSPECTION
  ↓
WAF
  ↓
IPS
  ↓
WEB SERVER
```

**A successful TCP connection does not prove that the web application is secure.**

**A successful HTTP response does not prove that WAF is correctly protecting the application.**

Always verify:

```text
Connectivity
     ↓
Inspection
     ↓
Detection
     ↓
Blocking
     ↓
Logging
     ↓
Application Functionality
```

---

## 🔗 Topics

* [ ] VIP / DNAT
* [ ] Firewall Policy
* [ ] Proxy Inspection
* [ ] SSL Inspection
* [ ] IPS
* [ ] Security Fabric
* [ ] Web Application Security
* [ ] HTTP/HTTPS Security
* [ ] Application-Layer Protection
* [ ] FortiGate Troubleshooting

---

## 🔗 SheynShield Resources

### 🎥 Video Learning

* [ ] [YouTube — SheynShield](https://youtube.com/@sheynshield)

  * Fortinet NSE content
  * FortiGate troubleshooting
  * Network Security Engineering

### 📚 Notes & Updates

* [ ] [Telegram — SheynShield](https://t.me/sheynshield)

### 💼 Professional Network

* [ ] [LinkedIn — Shayan-heydarikhah](https://linkedin.com/in/shayan-heydarikhah)

### 🐙 Technical Knowledge Base

* [ ] [SheynShield GitHub](https://github.com/Shayan-heydarikhah/sheynshield)

---

## 🔎 Keywords

`FortiGate WAF` · `FortiGate Web Application Firewall` · `FortiOS WAF` · `FortiGate WAF configuration` · `FortiGate WAF checklist` · `FortiGate VIP WAF` · `FortiGate reverse proxy` · `FortiGate SQL injection protection` · `FortiGate HTTP security` · `FortiGate HTTPS inspection` · `FortiGate proxy inspection` · `FortiGate IPS WAF` · `FortiGate NSE4 WAF` · `FortiGate NSE7 WAF` · `FortiGate troubleshooting` · `Web Application Firewall checklist`

---

> **SheynShield | Engineering Secure Networks**
>
> **Build it. Secure it. Verify it. Troubleshoot it.**
>
> This checklist is designed as a practical reference for FortiGate WAF deployment, security hardening, NSE preparation, and production troubleshooting.


---

> **SheynShield | Engineering Secure Networks**
>
> **FortiGate • Network Security • Secure Network Design • NSE Knowledge Base**
>
> *Practical knowledge. Engineering mindset. Secure networks.*

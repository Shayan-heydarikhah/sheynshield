# FortiGate Web Filter & HTTP/3/QUIC  

> **FortiGate Web Filtering | URL Filtering | FortiGuard | Content Filter | Anti-Phishing | Safe Search | WISP | HTTP/3 | QUIC**
>
> Practical reference for **FortiOS 7.x**, NSE preparation, troubleshooting, and real-world deployments.

---

## 📚 Table of Contents

- [1. Web Filter Architecture](#1-web-filter-architecture)
- [2. Web Filter Processing Order](#2-web-filter-processing-order)
- [3. URL Filter](#3-url-filter)
- [4. URL Filter Actions](#4-url-filter-actions)
- [5. FortiGuard Web Filtering](#5-fortiguard-web-filtering)
- [6. FortiGuard Categories](#6-fortiguard-categories)
- [7. Category Authentication](#7-category-authentication)
- [8. Web Filter Override](#8-web-filter-override)
- [9. Credential Anti-Phishing](#9-credential-anti-phishing)
- [10. Web Content Filter](#10-web-content-filter)
- [11. Content Filter Weight & Threshold](#11-content-filter-weight--threshold)
- [12. FortiSandbox Malicious URL Blocking](#12-fortisandbox-malicious-url-blocking)
- [13. FortiGuard Rating Failure](#13-fortiguard-rating-failure)
- [14. Domain vs IP Rating](#14-domain-vs-ip-rating)
- [15. Invalid URL Protection](#15-invalid-url-protection)
- [16. Safe Search](#16-safe-search)
- [17. YouTube Restricted Access](#17-youtube-restricted-access)
- [18. Google Account Restriction](#18-google-account-restriction)
- [19. HTTP POST Control](#19-http-post-control)
- [20. ActiveX & Java Applets](#20-activex--java-applets)
- [21. Cookie Filtering](#21-cookie-filtering)
- [22. WAD Diagnostics](#22-wad-diagnostics)
- [23. SSL Certificate Validation](#23-ssl-certificate-validation)
- [24. Websense WISP](#24-websense-wisp)
- [25. HTTP/3 & QUIC](#25-http3--quic)
- [26. HTTP/2 vs HTTP/3](#26-http2-vs-http3)
- [27. QUIC Packet Structure](#27-quic-packet-structure)
- [28. QUIC Header](#28-quic-header)
- [29. QUIC Frames & Streams](#29-quic-frames--streams)
- [30. QUIC vs TLS 1.3](#30-quic-vs-tls-13)
- [31. Web Filter Deployment Checklist](#31-web-filter-deployment-checklist)
- [32. Troubleshooting Commands](#32-troubleshooting-commands)
- [33. NSE Exam Traps](#33-nse-exam-traps)
- [34. Quick Revision](#34-quick-revision)

---

# 1. Web Filter Architecture

FortiGate Web Filtering can make decisions using multiple inspection mechanisms.

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
                     Web Script Filter
                            |
                            v
                    Antivirus Scanning
                            |
                            v
                          Client
````

## Main Components

| Component                     | Purpose                                                           |
| ----------------------------- | ----------------------------------------------------------------- |
| **URL Filter**                | Match administrator-defined URLs and URL patterns                 |
| **FortiGuard Web Filter**     | Category-based URL/domain rating                                  |
| **Web Content Filter**        | Inspect web content for configured words/patterns                 |
| **Web Script Filter**         | Inspect/control supported web scripts                             |
| **Antivirus**                 | Scan web downloads/content where applicable                       |
| **Anti-Phishing**             | Detect credential phishing behavior                               |
| **FortiSandbox URL Blocking** | Block malicious URLs identified through FortiSandbox intelligence |

> **Key concept:**
> URL filtering is primarily based on administrator-defined URL patterns, while FortiGuard Web Filtering uses FortiGuard-maintained ratings and categories.

---

# 2. Web Filter Processing Order

A simplified proxy web-filtering flow is:

```text
1. URL Filter
       |
       v
2. FortiGuard Web Filtering
       |
       v
3. Web Content Filter
       |
       v
4. Web Script Filter
       |
       v
5. Antivirus / Other Proxy Inspection
```

A blocking decision can prevent subsequent inspection stages from being reached.

```text
URL Filter
    |
    +---- BLOCK
             |
             +----> Request denied
```

> **Important:**
> Do not assume that every feature is evaluated for every request. Earlier decisions can prevent later engines from processing the traffic.

---

# 3. URL Filter

URL Filter allows administrators to manually define URL patterns.

## Common Use Cases

* Block specific websites
* Allow specific websites
* Match URL patterns
* Exempt trusted destinations
* Create custom web-access policies
* Trigger anti-phishing actions

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

> **Important:** Regex and wildcard matching are different mechanisms. Always select the correct matching type for the intended pattern.

---

# 4. URL Filter Actions

| Action      | Behavior                                                                            |
| ----------- | ----------------------------------------------------------------------------------- |
| **Exempt**  | Allows the traffic to bypass remaining applicable web/proxy inspection              |
| **Block**   | Denies the matching URL and displays a replacement/block page                       |
| **Allow**   | Allows the request to continue through subsequent applicable inspection             |
| **Monitor** | Processes the request similarly to Allow and generates logging for matching traffic |

## Allow vs Exempt

### Allow

```text
URL matches
     |
     v
   ALLOW
     |
     v
Continue remaining applicable inspection
```

### Exempt

```text
URL matches
     |
     v
  EXEMPT
     |
     v
Bypass remaining applicable inspection
```

> ### NSE Memory
>
> **Allow != Exempt**

---

# 5. FortiGuard Web Filtering

FortiGuard Web Filtering provides centrally maintained URL/domain categories and ratings.

```text
Client
  |
  v
FortiGate
  |
  +----> Local URL Filter
  |
  +----> FortiGuard Rating
  |
  +----> Category Decision
  |
  v
Allow / Block / Monitor / Warn / Authentication
```

FortiGuard provides continuously updated website categorization and reputation intelligence.

---

# 6. FortiGuard Categories

Examples of FortiGuard category IDs from the reference:

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

> **Note:** Category IDs and names can change between FortiOS/FortiGuard releases. Verify category definitions against the FortiGuard documentation for the deployed version.

---

# 7. Category Authentication

FortiGate can require authentication before allowing access to selected web categories.

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
FortiGate
 |
 v
Authentication Required
 |
 v
User / Group Validation
 |
 +----> Allowed
 |
 +----> Denied
```

---

# 8. Web Filter Override

Web Filter Override allows selected users/groups to use different web-filtering behavior.

```text
                    Web Filter Profile
                           |
              +------------+------------+
              |            |            |
              v            v            v
           Group A      Group B      Group C
           Strict       Monitor      Override
```

## Override Permissions

```bash
config webfilter profile
    edit "web-filt-test"
        set feature-set proxy

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

## Category Override

```bash
config webfilter profile
    edit "web-filt-test"

        config ftgd-wf
            unset options
            set ovrd g01 g02 g04 g05 g06 g07 g21 140 141
        end

    next
end
```

### Example Category Rules

```bash
config filters
    edit 140
        set category 140
        set action block
    next

    edit 141
        set category 141
        set action block
    next
end
```

> **Key concept:**
> Override is useful when the same web category must have different behavior for different users or groups.

---

# 9. Credential Anti-Phishing

Credential Anti-Phishing can inspect web requests for credential-submission behavior and compare the destination against configured/known phishing conditions.

```text
Client
  |
  | Username / Password
  v
Web Application
  |
  v
FortiGate Proxy
  |
  +----> URI Check
  |
  +----> FortiGuard Classification
  |
  +----> Anti-Phishing Inspection
  |
  +----> Credential Analysis
```

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

## Inspection Entries

```bash
config webfilter profile
    edit "wf-test"

        config antiphish
            config inspection-entries
                edit "all-test"
                    set action block
                    set fortiguard-categ all
                next
            end
        end

    next
end
```

## Custom Credential Patterns

```bash
config custom-pattern

    edit "username-id"
        set category username
        set type regex
    next

    edit "password"
        set category password
        set type regex
    next

end
```

## Logging

```bash
config webfilter profile
    edit "wf-test"

        set web-content-log enable
        set web-antiphishing-log enable
        set log-all-url enable

        config web
            set log-search enable
        end

    next
end
```

---

## Anti-Phishing Processing Priority

Web URL/category filtering can prevent anti-phishing inspection from occurring.

```text
Web URL Filter
      |
      +---- BLOCK
      |      |
      |      +----> No further Anti-Phishing inspection
      |
      +---- ALLOW
             |
             v
      FortiGuard Filtering
             |
             +---- BLOCK
             |      |
             |      +----> No Anti-Phishing inspection
             |
             +---- ALLOW
                    |
                    v
              Anti-Phishing
```

> **NSE Memory:**
> If traffic is blocked before Anti-Phishing processing, Anti-Phishing does not get an opportunity to inspect that request.

---

# 10. Web Content Filter

Web Content Filter examines content within web pages rather than simply matching the URL.

```text
Web Page
   |
   +---- "drug"
   |
   +---- "deal"
   |
   +---- "drug deal"
   |
   v
Weighted Score
   |
   v
Threshold
   |
   +---- Allow
   |
   +---- Block
```

## Example

If the page contains:

```text
drug
deal
```

The resulting score depends on the configured banned-word entities and their weights.

---

# 11. Content Filter Weight & Threshold

Each banned word/pattern can have a configured score.

## Example

| Term          | Weight | Threshold |
| ------------- | -----: | --------: |
| `drug`        |     10 |        10 |
| `drug deal`   |     20 |        10 |
| `"drug deal"` |     10 |        10 |

Conceptually:

```text
Total Score >= Threshold
        |
        v
      BLOCK
```

```text
Total Score < Threshold
        |
        v
      ALLOW
```

## Configure Word Score

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

> CLI syntax can vary by FortiOS release. Verify the exact hierarchy and available fields on the target FortiOS version.

## Configure Threshold

```bash
config webfilter profile
    edit "wf-test"

        config web
            set bword-threshold 10
        end

    next
end
```

---

## Wildcard vs Regex

### Wildcard

```text
forti*.com
```

Example matches may include:

```text
fortinet.com
fortiguard.com
```

### Regex

Regex uses regular-expression semantics.

For example:

```text
ch.ck
```

can represent a pattern where `.` is a regex metacharacter.

> **Exam Trap:**
> Never assume that a wildcard pattern and regex pattern have identical syntax.

---

# 12. FortiSandbox Malicious URL Blocking

FortiGate can use malicious URL intelligence associated with FortiSandbox.

## GUI Concept

```text
Security Profiles
    |
    +---- Web Filter
            |
            +---- Static URL Filter
                    |
                    +---- Block malicious URLs
                         discovered by FortiSandbox
```

## CLI

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

FortiGate may be unable to obtain a current FortiGuard rating because of connectivity or service problems.

## Allow on Rating Error

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
       +---- Available
       |       |
       |       +---- Normal category decision
       |
       +---- Unavailable
               |
               +---- error-allow
               |       |
               |       +---- Allow
               |
               +---- Fail-closed behavior
                       |
                       +---- Block
```

## Security Trade-Off

| Behavior              | Availability | Security |
| --------------------- | ------------ | -------- |
| Allow on rating error | Higher       | Lower    |
| Block on rating error | Lower        | Higher   |

> ### NSE Memory
>
> `error-allow` prioritizes availability when FortiGuard rating is unavailable.

---

# 14. Domain vs IP Rating

FortiGate can use both the requested domain and destination IP for web rating in supported configurations.

```text
Requested Domain
       +
Destination IP
       |
       v
FortiGuard Rating
       |
       v
Rating Resolution
       |
       v
Final Category / Action
```

A domain and its IP address may receive different ratings.

## Conceptual Example

```text
Domain:
test.com
    |
    +---- Games = 10

Destination IP:
192.0.2.10
    |
    +---- Social Media = 50
```

If the higher-weight rating is selected:

```text
Social Media = 50
        |
        v
Final Rating
```

## Conceptual Rating Weights

```text
Malware / Phishing     100
Adult                   90
Gambling                80
Social Media            50
News                    40
Business                10
```

> **Important:** These numbers should be treated as conceptual examples unless verified against the exact FortiOS/FortiGuard release.

## Enable IP Rating

```bash
config webfilter profile
    edit "wf-test"

        config ftgd-wf
            set options rate-server-ip
        end

    next
end
```

---

# 15. Invalid URL Protection

Invalid URL protection can help block malformed or invalid URL requests.

```bash
config webfilter profile
    edit "wf-test"
        set options block-invalid-url
    next
end
```

## URL Encoding

A space in a URL path should normally be encoded as:

```text
space
  |
  v
%20
```

Example:

```text
https://example.com/my%20file
```

---

# 16. Safe Search

FortiGate can enforce Safe Search for supported search engines.

Common examples include:

* Google
* Bing
* Yahoo
* Yandex

```text
Client
  |
  v
FortiGate
  |
  v
Search Engine
  |
  v
Safe Search Enforcement
  |
  v
Restricted Search Results
```

## Enable Search Logging

```bash
config webfilter profile
    edit "wf-test"

        config web
            set log-search enable
        end

    next
end
```

## FortiView

Search activity can be investigated through FortiView where supported:

```text
FortiView
   |
   +---- Search Phrases
```

> **Security Consideration:** Search queries may contain sensitive information. Enable search logging only when operationally justified and handle logs according to organizational privacy requirements.

---

# 17. YouTube Restricted Access

FortiGate can enforce YouTube restriction modes.

```text
YouTube Restriction
        |
        +---- Strict
        |
        +---- Moderate
```

## Video Filter Visibility

```bash
config system feature-visibility
    set video-filter enable
end
```

Video filtering can use FortiGuard classification and supported YouTube controls.

---

## Vimeo Restriction

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

Example values:

```text
7
→ Do not show mature content

134
→ Do not show unrated and mature content
```

> Verify the available values on the deployed FortiOS release.

---

# 18. Google Account Restriction

Google account restrictions can limit which Google accounts/domains users can access.

## Concept

Allowed domain:

```text
fortinet.com
```

Example:

```text
user@gmail.com
      |
      +---- BLOCK

user@fortinet.com
      |
      +---- ALLOW
```

This can be useful when an organization wants users to access only organization-controlled Google accounts.

---

# 19. HTTP POST Control

HTTP POST is commonly used by browsers to submit data to web servers.

Examples:

* Forms
* Credentials
* File uploads
* Application data

## Configure

```bash
config webfilter profile
    edit "wf-test"
        set post-action normal
    next
end
```

Possible behaviors include:

```text
normal
block
```

## Security Consideration

Blocking HTTP POST can reduce certain data-submission paths but may break legitimate applications.

```text
Security
   +
Compatibility
   =
Required testing
```

---

# 20. ActiveX & Java Applets

ActiveX and Java Applets are legacy web technologies.

Historically they were used by many browser-based applications.

Common legacy examples:

```text
DVR
CCTV
Legacy Management Systems
Internet Explorer Applications
```

Modern security architectures generally avoid these technologies where possible because they can increase the attack surface.

> **Important:** Blocking legacy browser components may break old applications. Test before enforcing the policy globally.

---

# 21. Cookie Filtering

Cookies are stored by browsers and commonly used for:

* Session management
* Authentication
* User preferences
* Tracking
* Application state

```text
Browser
   |
   +---- Cookie
   |
   v
Web Server
```

Cookie controls must be balanced against application compatibility.

---

# 22. WAD Diagnostics

The **WAD (Web Application Daemon)** is strongly associated with FortiGate proxy-based web processing.

## Check WAD Filter — Root VDOM

```bash
diagnose wad filter vd root
```

## Check WAD Filter — All VDOMs

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

SSL/TLS inspection can validate certificates and detect certificate-related problems.

Potential certificate conditions include:

```text
Revoked Certificate
Untrusted Certificate
Invalid Certificate
Self-Signed Certificate
Certificate Name Mismatch
```

## Enterprise Inspection Model

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
  +---- IPS / AV / Other Security Engines
  |
  v
Internet
```

## Deep Inspection

For encrypted web traffic where payload inspection is required:

```text
Certificate Inspection
        |
        +---- Limited visibility

Deep Inspection
        |
        +---- Decrypt
        +---- Inspect
        +---- Re-encrypt
```

> **Important:** Deep Inspection requires appropriate certificate deployment/trust on managed clients and can introduce compatibility issues.

> **Security Note:** Do not automatically block all self-signed certificates in every environment. Internal applications may legitimately use certificates issued by a private PKI.

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

## Configure WISP Servers

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
primary-secondary
round-robin
auto-learning
```

### Concept

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
        +---- Prefer faster analysis path
```

> **Design consideration:** External web-filtering services introduce additional dependencies, latency, and resource requirements.

---

# 25. HTTP/3 & QUIC

HTTP/3 uses **QUIC**, and QUIC uses **UDP**.

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

> ### Key Concept
>
> **HTTP/3 is not simply HTTP/2 over UDP.**
>
> HTTP/3 is designed specifically to operate over QUIC.

---

# 26. HTTP/2 vs HTTP/3

| Feature                   | HTTP/2    | HTTP/3                           |
| ------------------------- | --------- | -------------------------------- |
| Transport                 | TCP       | UDP                              |
| Security                  | TLS       | TLS 1.3 integrated with QUIC     |
| Multiplexing              | Yes       | Yes                              |
| TCP Head-of-Line Blocking | Yes       | No TCP-level HOL between streams |
| Transport Handshake       | TCP + TLS | QUIC + TLS                       |
| Common Port               | TCP/443   | UDP/443                          |
| Protocol                  | HTTP/2    | HTTP/3                           |

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

Conceptually:

```text
QUIC Packet
     |
     +---- Header
     |
     +---- Frame
     |
     +---- Frame
     |
     +---- Frame
```

---

# 28. QUIC Header

Important QUIC concepts include:

## Destination Connection ID

Identifies the destination QUIC connection.

```text
Destination Connection ID
          |
          v
Correct QUIC Connection
```

## Packet Number

Used by QUIC for packet identification and loss/recovery processing.

```text
Packet 1
Packet 2
Packet 3
Packet 4
```

Packet numbers are used as part of QUIC's reliability and loss-detection mechanisms.

---

## Spin Bit

The spin bit is a QUIC header bit that can assist passive measurement of network RTT/latency.

```text
Spin Bit
   |
   +---- Passive Measurement
   |
   +---- Network Monitoring
```

> Its use is subject to QUIC implementation behavior and endpoint configuration.

---

## Key Phase

The Key Phase bit identifies the packet-protection key phase being used.

```text
TLS 1.3
   |
   v
Packet Protection Keys
   |
   v
Key Phase
```

> **Exam correction:** The QUIC header does **not** contain the actual secret encryption key. TLS 1.3 establishes the cryptographic keys; the QUIC header carries metadata such as key phase.

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

A STREAM frame can contain information such as:

```text
Stream ID
Offset
Stream Data
```

---

## Streams

A single QUIC connection can support multiple independent streams.

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

```text
Bidirectional
Unidirectional
```

### Why Streams Matter

HTTP/3 uses QUIC streams for multiplexing.

If one stream experiences packet loss:

```text
Stream 1
   |
   +---- Packet Loss
```

other streams are not subject to TCP-level head-of-line blocking.

---

# 30. QUIC vs TLS 1.3

| Feature         | QUIC / HTTP/3                            | TLS 1.3                           |
| --------------- | ---------------------------------------- | --------------------------------- |
| Primary Purpose | Transport + multiplexing                 | Encryption + authentication       |
| Transport       | UDP                                      | Can operate over TCP or QUIC      |
| Handshake       | Uses TLS 1.3                             | Cryptographic handshake           |
| Encryption      | Uses TLS 1.3 packet protection           | Provides encryption               |
| Multiplexing    | Yes                                      | No                                |
| 0-RTT           | Supported by QUIC when conditions permit | Supports 0-RTT mechanisms         |
| TCP HOL         | Avoided because QUIC is not TCP          | Does not itself eliminate TCP HOL |

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

### Key Concept

```text
QUIC
   +
TLS 1.3
   |
   v
Secure Transport
```

---

# 31. Web Filter Deployment Checklist

## Firewall Policy

* [ ] Correct firewall policy selected.
* [ ] Required inspection mode configured.
* [ ] Proxy-based inspection enabled where required.
* [ ] Web Filter profile attached.
* [ ] SSL inspection profile configured.
* [ ] Deep Inspection enabled where encrypted payload inspection is required.
* [ ] Client trust chain deployed.
* [ ] Logging configured.

## URL Filtering

* [ ] Required block URLs configured.
* [ ] Required allow URLs configured.
* [ ] Wildcard/regex syntax verified.
* [ ] Rule order reviewed.
* [ ] Exempt rules reviewed carefully.
* [ ] Anti-phishing URL actions tested where required.

## FortiGuard

* [ ] FortiGuard license/service status verified.
* [ ] FortiGuard connectivity verified.
* [ ] Required categories configured.
* [ ] Rating failure behavior reviewed.
* [ ] IP rating evaluated where required.
* [ ] Category overrides reviewed.

## Content Filter

* [ ] Banned words configured.
* [ ] Word scores reviewed.
* [ ] Threshold configured.
* [ ] False positives tested.
* [ ] Regex/wildcard behavior verified.

## Anti-Phishing

* [ ] LDAP/AD integration configured.
* [ ] Anti-phishing enabled.
* [ ] URI inspection configured.
* [ ] Basic authentication inspection reviewed.
* [ ] Username inspection configured if required.
* [ ] Custom patterns reviewed.
* [ ] Logging enabled.

## Safe Search

* [ ] Safe Search configured.
* [ ] Search logging enabled only when required.
* [ ] YouTube restriction tested.
* [ ] Vimeo restrictions tested where required.

## SSL Inspection

* [ ] Appropriate SSL inspection profile selected.
* [ ] CA certificate deployed to clients.
* [ ] Invalid certificate handling reviewed.
* [ ] Untrusted certificate handling reviewed.
* [ ] Certificate compatibility tested.

## HTTP/3 / QUIC

* [ ] UDP/443 behavior reviewed.
* [ ] HTTP/3 application requirements identified.
* [ ] Deep Inspection compatibility tested.
* [ ] QUIC-related troubleshooting performed where necessary.
* [ ] Blocking QUIC is used only when operationally justified.

---

# 32. Troubleshooting Commands

## WAD

```bash
diagnose wad filter vd root
diagnose wad filter vd all
```

## Web Filter Statistics

```bash
diagnose webfilter stats list root
```

## FortiGuard

```bash
diagnose webfilter fortiguard statistics
diagnose webfilter fortiguard override
diagnose webfilter fortiguard cache
```

---

## Recommended Troubleshooting Flow

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
Final Decision
```

---

# 33. NSE Exam Traps

## Trap #1 — Allow vs Exempt

```text
ALLOW
  |
  +---- Continue remaining applicable inspection

EXEMPT
  |
  +---- Bypass remaining applicable inspection
```

---

## Trap #2 — URL Filter vs FortiGuard

```text
URL Filter
  |
  +---- Administrator-defined URL patterns

FortiGuard
  |
  +---- Fortinet-maintained categories and ratings
```

---

## Trap #3 — Content Filter

```text
URL Filter
  |
  +---- URL / URL Pattern

Content Filter
  |
  +---- Content inside web pages
```

---

## Trap #4 — Rating Error

```text
error-allow
  |
  +---- Allow when FortiGuard rating cannot be obtained
```

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

---

## Trap #6 — QUIC Encryption

QUIC uses TLS 1.3.

```text
QUIC
  +
TLS 1.3
```

Do not describe QUIC as using a completely separate encryption protocol.

---

## Trap #7 — QUIC Streams

One QUIC connection can carry many independent streams.

```text
1 QUIC Connection
       |
       +---- Stream 1
       +---- Stream 2
       +---- Stream 3
       +---- Stream N
```

---

## Trap #8 — Deep Inspection

If encrypted HTTP payload inspection is required:

```text
Certificate Inspection
        |
        +---- Limited payload visibility

Deep Inspection
        |
        +---- Decrypt
        +---- Inspect
        +---- Re-encrypt
```

---

## Trap #9 — Proxy Mode

Some web-filtering functions require proxy-based processing.

```text
Flow-Based
   |
   +---- Lower processing overhead
   +---- More limited proxy/content capabilities

Proxy-Based
   |
   +---- Full web proxy processing
   +---- Required for some advanced web-filtering features
```

---

## Trap #10 — Security vs Compatibility

Aggressive web filtering can affect:

```text
Legacy Applications
Google Services
YouTube
HTTP/3
QUIC
DVR / CCTV Portals
Authentication Portals
File Uploads
```

Always test security policies against real application traffic.

---

# 34. Quick Revision

## Web Filter

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

> **URL Filter decides based on URL patterns, FortiGuard decides based on category/rating, Content Filter evaluates web content, Anti-Phishing inspects credential-related behavior, and Deep Inspection provides visibility into encrypted application traffic.**

---

# 🔥 Fast Comparison

| Feature                       | Main Question                                                  |
| ----------------------------- | -------------------------------------------------------------- |
| **URL Filter**                | Does this URL/pattern match?                                   |
| **FortiGuard**                | What category/rating does this destination have?               |
| **Content Filter**            | What words/patterns exist inside the web content?              |
| **Anti-Phishing**             | Is the user submitting credentials in a suspicious context?    |
| **Safe Search**               | Should search results be restricted?                           |
| **FortiSandbox URL Blocking** | Is the URL associated with malicious intelligence?             |
| **WISP**                      | Should an external web-filtering service evaluate the request? |
| **SSL Inspection**            | Can FortiGate see and inspect encrypted traffic?               |
| **HTTP/3**                    | Is the application using HTTP over QUIC?                       |
| **QUIC**                      | Is the transport using UDP with integrated TLS 1.3?            |

---

## 🎯 Final Mental Model

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

## 📌 Related FortiGate Topics

* FortiGate Firewall Policy
* FortiGate Proxy-Based Inspection
* FortiGate SSL Deep Inspection
* FortiGuard Web Filtering
* FortiGate Anti-Phishing
* FortiGate Antivirus
* FortiGate FortiSandbox
* FortiGate WAD
* HTTP/2
* HTTP/3
* QUIC
* TLS 1.3

---

> **SheynShield | Engineering Secure Networks**
>
> FortiGate • Network Security • Secure Network Design • NSE Knowledge Base

 

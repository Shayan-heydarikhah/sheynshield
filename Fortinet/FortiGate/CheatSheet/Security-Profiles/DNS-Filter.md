# FortiGate DNS Filter —  

> **FortiGate DNS Filter | FortiGuard Secure DNS | DNS Proxy | Botnet C&C | DNS Translation | DoH/DoT | DNS Troubleshooting**
>
> Practical reference for **FortiOS 7.x**, NSE preparation, troubleshooting, and enterprise deployments.

---

## 📚 Table of Contents

- [1. DNS Filter Overview](#1-dns-filter-overview)
- [2. Security Profile Processing Concept](#2-security-profile-processing-concept)
- [3. DNS Filter vs Web Filter vs Antivirus](#3-dns-filter-vs-web-filter-vs-antivirus)
- [4. DNS Proxy](#4-dns-proxy)
- [5. DNS Cache](#5-dns-cache)
- [6. System DNS Configuration](#6-system-dns-configuration)
- [7. DNS Server Selection](#7-dns-server-selection)
- [8. FortiGuard Secure DNS](#8-fortiguard-secure-dns)
- [9. DNS Filter Profile](#9-dns-filter-profile)
- [10. DNS Filter Actions](#10-dns-filter-actions)
- [11. Local Domain Filter](#11-local-domain-filter)
- [12. Wildcard vs Regex](#12-wildcard-vs-regex)
- [13. FortiGuard DNS Rating](#13-fortiguard-dns-rating)
- [14. Botnet C&C DNS Protection](#14-botnet-c-c-dns-protection)
- [15. Safe Search](#15-safe-search)
- [16. DNS Translation](#16-dns-translation)
- [17. DNS Database](#17-dns-database)
- [18. DNS over TLS](#18-dns-over-tls)
- [19. DNS over HTTPS](#19-dns-over-https)
- [20. DNS Query Types](#20-dns-query-types)
- [21. DNS Logs](#21-dns-logs)
- [22. DNSProxy Diagnostics](#22-dnsproxy-diagnostics)
- [23. FortiGuard DNSProxy Diagnostics](#23-fortiguard-dnsproxy-diagnostics)
- [24. DNS Cache Troubleshooting](#24-dns-cache-troubleshooting)
- [25. DNS Troubleshooting Flow](#25-dns-troubleshooting-flow)
- [26. Deployment Checklist](#26-deployment-checklist)
- [27. NSE Exam Traps](#27-nse-exam-traps)
- [28. Quick Revision Card](#28-quick-revision-card)

---

# 1. DNS Filter Overview

FortiGate DNS Filter allows the firewall to inspect DNS requests and apply security decisions before the client establishes a connection to the destination.

```text
Client
   |
   | DNS Query
   v
FortiGate
   |
   +---- Local DNS Filter
   |
   +---- FortiGuard DNS Rating
   |
   +---- Botnet / C&C Intelligence
   |
   +---- DNS Cache
   |
   v
Allow / Monitor / Block / Redirect
   |
   v
DNS Response
````

### Why DNS Filtering?

DNS filtering can stop malicious or unwanted domains **before the client connects to the destination**.

Typical use cases:

* Malicious domain blocking
* Phishing domain blocking
* Botnet C&C protection
* Category-based DNS filtering
* Safe Search
* Local domain policies
* DNS redirection
* DNS translation

---

# 2. Security Profile Processing Concept

A simplified security inspection concept is:

```text
Client
  |
  v
DNS Request
  |
  v
DNS Filter
  |
  +---- Block?
  |      |
  |      +---- YES → Stop
  |
  +---- Allow
  |
  v
Web Connection
  |
  v
Web Filter
  |
  v
Antivirus
```

### Key Idea

DNS filtering works at the **domain-resolution stage**.

```text
DNS Filter
    ↓
Domain

Web Filter
    ↓
URL / Web Category / Content

Antivirus
    ↓
Files / Malware Signatures
```

---

# 3. DNS Filter vs Web Filter vs Antivirus

| Security Feature | Main Inspection Target         | Example                        |
| ---------------- | ------------------------------ | ------------------------------ |
| **DNS Filter**   | Domain / DNS request           | `malicious.example`            |
| **Web Filter**   | URL / category / web content   | `https://example.com/download` |
| **Antivirus**    | Files / content / signatures   | Malicious executable           |
| **IPS**          | Network/application signatures | Exploit / attack pattern       |

### Simple Memory Model

```text
DNS
 ↓
"Where am I going?"

Web Filter
 ↓
"What website/URL am I accessing?"

AV
 ↓
"What file/content am I receiving?"
```

---

# 4. DNS Proxy

FortiGate uses the **DNS Proxy** process to handle DNS-related functions.

```text
Client
   |
   | DNS Query
   v
DNS Proxy
   |
   +---- Cache
   |
   +---- DNS Filter
   |
   +---- FortiGuard Secure DNS
   |
   +---- Upstream DNS
   |
   v
DNS Response
```

### Important Process

```text
dnsproxy
```

### Diagnostic Command

```bash
diagnose test application dnsproxy
```

---

# 5. DNS Cache

DNS caching improves performance by avoiding repeated upstream DNS queries.

```text
Client
  |
  | Query example.com
  v
FortiGate
  |
  +---- Cache HIT
  |       |
  |       +---- Return cached IP
  |
  +---- Cache MISS
          |
          +---- Query upstream DNS
```

### Benefits

* Faster DNS resolution
* Reduced upstream DNS traffic
* Reduced latency
* Reduced repeated DNS queries

### Important

DNS caching is especially useful when many clients repeatedly request the same domains.

---

# 6. System DNS Configuration

Basic DNS configuration:

```bash
config system dns
    set timeout 5
    set retry 2
    set cache-notfound-response disable
    set dns-cache-limit 5000
    set dns-cache-ttl 1000
    set alt-primary 178.22.122.101
    set alt-secondary 8.8.8.8
    set server-select-method least-rtt
    set log all
end
```

> ⚠️ **Security Note:** Replace public/placeholder DNS addresses with approved enterprise resolvers in production.

---

## DNS Timeout

```bash
set timeout 5
```

Concept:

```text
DNS request
    |
    +---- Wait up to 5 seconds
    |
    +---- Retry / fail according to configuration
```

---

## DNS Retry

```bash
set retry 2
```

Controls the number of retry attempts when resolving through configured DNS servers.

---

## Negative DNS Cache

```bash
set cache-notfound-response disable
```

Controls caching of unsuccessful DNS responses.

Concept:

```text
Domain does not exist
        |
        v
Negative DNS response
        |
        +---- Cache according to configuration
```

---

# 7. DNS Server Selection

FortiOS can select an upstream DNS server based on different methods.

Example:

```bash
set server-select-method least-rtt
```

### `least-rtt`

Selects the DNS server with the lowest measured round-trip time.

```text
DNS-1 → 40 ms
DNS-2 → 15 ms
DNS-3 → 80 ms

Selected:
DNS-2
```

### Conceptual Comparison

| Method        | Concept                                        |
| ------------- | ---------------------------------------------- |
| **least-rtt** | Prefer server with lowest latency              |
| **failover**  | Use another server when preferred server fails |

> Always verify the exact available methods for the deployed FortiOS release.

---

# 8. FortiGuard Secure DNS

FortiGate can use FortiGuard DNS services for DNS resolution and security rating.

```text
Client
   |
   v
FortiGate
   |
   v
FortiGuard Secure DNS
   |
   +---- DNS Resolution
   |
   +---- Domain Rating
   |
   +---- Security Intelligence
```

### Anycast

FortiGuard Anycast can provide a nearby service endpoint.

```bash
config system fortiguard
    set fortiguard-anycast enable
end
```

Additional Secure DNS parameters may include:

```bash
config system fortiguard
    set fortiguard-anycast enable
    set fortiguard-anycast-source fortinet
    set anycast-sdns-server-ip 0.0.0.0
    set anycast-sdns-server-port 853
end
```

### Port `853`

```text
TCP/UDP 853
    ↓
DNS over TLS / Secure DNS
```

> The exact transport behavior depends on the FortiOS feature and configuration. Verify the deployed release documentation before assuming a specific protocol.

---

# 9. DNS Filter Profile

DNS Filter is configured under:

```text
Security Profiles
    └── DNS Filter
```

Typical functions include:

* Domain filtering
* FortiGuard category filtering
* Botnet/C&C protection
* Safe Search
* DNS redirection
* Logging
* Rating-error handling

---

# 10. DNS Filter Actions

Common domain-filter actions include:

| Action      | Behavior                       |
| ----------- | ------------------------------ |
| **Allow**   | Permit the DNS request         |
| **Monitor** | Permit while logging the match |
| **Block**   | Deny the DNS request           |

### Processing Concept

```text
DNS Query
   |
   v
Local Domain Filter
   |
   +---- BLOCK → Stop
   |
   +---- ALLOW
   |
   v
FortiGuard Rating
   |
   +---- BLOCK
   |
   +---- ALLOW
```

---

# 11. Local Domain Filter

A local/static domain filter allows administrators to define their own domain rules.

```text
Client
   |
   v
Local Domain Filter
   |
   +---- Match → Action
   |
   +---- No Match
          |
          v
     FortiGuard
```

### Processing Order

```text
Local Domain Filter
        ↓
FortiGuard DNS Service
```

### Example

```bash
config dnsfilter domain-filter
    ...
end
```

> Exact CLI hierarchy can vary by FortiOS version. Verify the command tree with `?` or the release-specific CLI reference.

---

# 12. Wildcard vs Regex

FortiOS supports different matching methods for domain filters.

## Wildcard

Example:

```text
*.host
```

Conceptually matches domains under the specified suffix.

Example:

```text
wp36.host
wp36.host.pressdns.com
```

However:

```text
wp36.host123.pressdns.com
```

may not match because the expected word boundary is absent.

---

## Regex

For strict domain matching, use an explicit regular expression.

Example concept:

```text
^.*\.host$
```

This means:

```text
Start
  ↓
Any characters
  ↓
.host
  ↓
End
```

### Comparison

| Pattern      | Purpose                 |
| ------------ | ----------------------- |
| `*.host`     | Wildcard matching       |
| `^.*\.host$` | Explicit regex matching |

### NSE Memory Aid

```text
Wildcard
→ easier

Regex
→ precise
```

---

# 13. FortiGuard DNS Rating

FortiGuard can classify domains based on its cloud security intelligence.

```text
DNS Query
   |
   v
FortiGate
   |
   v
FortiGuard Rating
   |
   +---- Category
   |
   +---- Reputation
   |
   +---- Security Intelligence
   |
   v
Policy Decision
```

Possible security categories can include:

* Malware
* Phishing
* Botnet
* Adult
* Gambling
* Social Media
* Other FortiGuard categories

---

# 14. Botnet C&C DNS Protection

DNS filtering can help detect and block domains associated with **Command & Control (C&C)** infrastructure.

```text
Client
   |
   | DNS Query
   v
FortiGate
   |
   v
Botnet / C&C Intelligence
   |
   +---- Malicious IP / Domain?
          |
          +---- YES → BLOCK
          |
          +---- NO → Continue
```

### Concept

```text
DNS Resolution
      |
      v
Destination IP
      |
      v
Botnet Intelligence
      |
      +---- Known C&C → Block
```

### Diagnostics

```bash
diagnose sys botnet list 9000 10
```

Conceptually:

```text
9000 → starting index/context
10   → number of entries
```

### Hit Information

```bash
diagnose sys botnet-ip hit
```

Useful for investigating botnet IP database matches/hits.

---

# 15. Safe Search

FortiGate can enforce Safe Search for supported search engines.

Typical search engines include:

* Google
* Bing
* Yahoo
* Yandex

Concept:

```text
Client
  |
  | Search Query
  v
FortiGate
  |
  +---- Safe Search Enforcement
  |
  v
Search Engine
  |
  v
Restricted Results
```

### Why DNS Filter Can Help

DNS-level filtering can control access to domains/services involved in search and security enforcement, while web/application inspection can provide additional control.

---

# 16. DNS Translation

DNS translation allows FortiGate to translate a DNS result to another IP address.

Example concept:

```text
Client asks:

iran.ir
   |
   v
FortiGate
   |
   +---- Original:
   |     185.143.233.120
   |
   +---- Translated:
         192.168.20.200
```

### Example Use Case

```text
Internet DNS
     |
     v
Public IP
     |
     v
FortiGate DNS Translation
     |
     v
Internal IP
```

This can be useful in controlled DNS/NAT architectures where a domain should resolve differently for internal clients.

---

## DNS Translation Architecture

```text
                    Internet
                       |
                       v
                 Public Service
                       |
                  Public IP
                       |
                       v
                 FortiGate
                       |
                DNS Translation
                       |
                       v
                Internal Server
                192.168.20.200
```

### Security Policy Consideration

A typical design may require separate policies:

```text
Policy 1
LAN → WAN
    |
    +---- DNS Filter
    +---- Inspection

Policy 2
LAN → DMZ/Internal
    |
    +---- Allow translated destination
```

---

# 17. DNS Database

FortiGate maintains DNS-related databases and caches used by the DNS proxy.

Useful concepts include:

```text
DNS Cache
DNS Database
FQDN Database
Secure DNS Database
Botnet Database
FortiGuard Rating Cache
Hostname Cache
```

### DNS Server Configuration

DNS functionality can also be associated with interface-level DNS server settings.

Typical modes include:

```text
Recursive
System DNS
```

Conceptually:

```text
Client
   |
   v
FortiGate DNS Server
   |
   +---- DNS Filter
   |
   +---- Cache
   |
   +---- System DNS
   |
   v
Upstream Resolver
```

---

# 18. DNS over TLS

**DoT = DNS over TLS**

Instead of sending traditional plaintext DNS:

```text
Client
   |
   | DNS
   v
Resolver
```

DoT encrypts DNS traffic using TLS.

```text
Client
   |
   | TLS
   | TCP/853
   v
DNS Resolver
```

### Security Benefits

* Encrypts DNS queries
* Reduces passive DNS visibility
* Protects DNS traffic against some forms of interception

### FortiGate Considerations

When inspecting secure DNS:

```text
DoT
 ↓
TLS inspection / protocol handling
 ↓
Certificate validation
 ↓
Security policy
```

The exact inspection capability depends on the FortiOS release and inspection profile.

---

# 19. DNS over HTTPS

**DoH = DNS over HTTPS**

DNS queries are transported inside HTTPS.

```text
Client
   |
   | HTTPS
   | TCP/443
   v
DoH Resolver
```

Compared with traditional DNS:

```text
Traditional DNS
→ UDP/TCP 53

DoT
→ TLS
→ 853

DoH
→ HTTPS
→ 443
```

### Security Challenge

DoH can blend DNS traffic with normal HTTPS traffic.

```text
HTTPS/443
   |
   +---- Normal Web
   |
   +---- DoH
```

Therefore, identifying and controlling DoH may require additional application, web, or TLS inspection capabilities.

---

# 20. DNS Query Types

Important DNS record/query types:

| QTYPE | Record | Purpose                |
| ----: | ------ | ---------------------- |
|   `1` | A      | IPv4 address           |
|  `28` | AAAA   | IPv6 address           |
|   `5` | CNAME  | Canonical name / alias |
|  `15` | MX     | Mail exchanger         |
|  `16` | TXT    | Text information       |

### Quick Memory

```text
A
→ IPv4

AAAA
→ IPv6

CNAME
→ Alias

MX
→ Mail

TXT
→ Text
```

### DNS Class

```text
IN
```

means:

```text
Internet
```

---

# 21. DNS Logs

DNS filtering can log DNS queries and responses.

Typical information may include:

```text
Client
Domain
Query Type
Response
Action
Category
Policy
Timestamp
```

### Log Filter

Example:

```bash
execute log filter category utm-dns
execute log display
```

Useful for investigating:

* Blocked domains
* Suspicious DNS requests
* Botnet activity
* DNS filtering decisions
* Client behavior

---

# 22. DNSProxy Diagnostics

Main diagnostic entry point:

```bash
diagnose test application dnsproxy
```

This provides multiple DNS proxy diagnostic operations.

---

## Diagnostic Menu

Conceptual operations include:

```text
1   Clear DNS cache
2   Show statistics
3   Dump DNS settings
4   Reload FQDN
5   Requery FQDN
6   Dump FQDN
7   Dump DNS cache
8   Dump DNS database
9   Reload DNS database
10  Dump secure DNS policy/profile
11  Dump botnet domains
12  Reload secure DNS settings
13  Show hostname cache
14  Clear hostname cache
15  Show Secure DNS rating cache
16  Clear Secure DNS rating cache
17  DNS debug bit mask
18  DNS debug object memory
99  Restart DNSProxy worker
```

> Exact menu items and behavior can vary by FortiOS release.

---

# 23. FortiGuard DNSProxy Diagnostics

A useful command for Secure DNS/FortiGuard-related information is:

```bash
diagnose test application dnsproxy 3
```

This can expose diagnostic information related to:

* Secure DNS
* FortiGuard rating
* FortiGuard server
* Service status
* License information
* Response time
* Failures
* Timeouts
* Pending requests

---

## Useful Fields

When reviewing output, pay attention to fields such as:

```text
type
expire
tls
request
response
timeout
ready
rt
probe
times
failure
last-fail
```

### Conceptual Interpretation

| Field            | Concept                     |
| ---------------- | --------------------------- |
| `tls`            | Secure DNS/TLS state        |
| `request`        | Pending request information |
| `response`       | Response state              |
| `to` / `timeout` | Timeout information         |
| `ready`          | Service readiness           |
| `rt`             | Response time               |
| `failure`        | Failure count               |
| `last-fail`      | Last failure information    |
| `probe`          | Health/probe state          |

> Always validate field meanings against the exact FortiOS build because diagnostic output is implementation-specific.

---

# 24. DNS Cache Troubleshooting

## Clear DNS Cache

```text
diagnose test application dnsproxy 1
```

Use when:

* DNS cache contains stale information
* DNS changes are not reflected
* Troubleshooting resolution problems

---

## Show DNS Statistics

```text
diagnose test application dnsproxy 2
```

Useful for:

* Latency
* Forwarding
* Retries
* DNS server health
* Resolution behavior

### Important Indicator

If latency is reported as:

```text
-1
```

it can indicate that the DNS server is unreachable or no valid measurement is available.

---

## Dump DNS Settings

```text
diagnose test application dnsproxy 3
```

Useful for checking:

```text
DNS servers
FortiGuard servers
Response time
Failures
Timeouts
Secure DNS state
```

---

# 25. DNS Troubleshooting Flow

Use this sequence when a client cannot resolve a domain.

```text
                Client
                   |
                   v
             DNS Request
                   |
                   v
             FortiGate
                   |
                   v
              DNS Filter
                   |
          +--------+--------+
          |                 |
        BLOCK              ALLOW
          |                 |
          v                 v
         STOP          Local Cache
                              |
                       +------+------+
                       |             |
                      HIT           MISS
                       |             |
                       v             v
                    Answer      Upstream DNS
                                      |
                              +-------+-------+
                              |               |
                            Success          Fail
                              |               |
                              v               v
                           Answer        Alternate DNS
                                              |
                                              v
                                          FortiGuard
```

---

## Troubleshooting Checklist

### Step 1 — Check Firewall Policy

```text
LAN → WAN
```

Verify:

* DNS traffic is allowed
* Correct security profile is attached
* DNS Filter is enabled
* Inspection settings are appropriate

---

### Step 2 — Check DNS Server

Verify:

```text
Primary DNS
Secondary DNS
Alternative DNS
Timeout
Retry
```

---

### Step 3 — Check DNS Cache

```bash
diagnose test application dnsproxy 7
```

---

### Step 4 — Check FortiGuard

```bash
get system fortiguard
```

Verify:

```text
License
Connectivity
FortiGuard status
```

---

### Step 5 — Check DNSProxy

```bash
diagnose test application dnsproxy
```

---

### Step 6 — Check Secure DNS

```bash
diagnose test application dnsproxy 3
```

Look for:

```text
RT
Failures
Timeouts
Ready state
TLS state
```

---

### Step 7 — Check Logs

```bash
execute log filter category utm-dns
execute log display
```

---

# 26. Deployment Checklist

## DNS Filter

* [ ] DNS Filter profile created.
* [ ] Correct domain categories configured.
* [ ] Local domain filters reviewed.
* [ ] Block/Allow/Monitor actions tested.
* [ ] FortiGuard connectivity verified.
* [ ] Rating-error behavior reviewed.
* [ ] DNS logging enabled where required.

---

## DNS Cache

* [ ] Cache size reviewed.
* [ ] Cache TTL reviewed.
* [ ] Negative response caching reviewed.
* [ ] Timeout configured.
* [ ] Retry count configured.
* [ ] DNS server selection method reviewed.

---

## FortiGuard

* [ ] FortiGuard license verified.
* [ ] FortiGuard connectivity tested.
* [ ] Anycast evaluated.
* [ ] Secure DNS status verified.
* [ ] Rating service operational.

---

## Botnet Protection

* [ ] Botnet/C&C filtering enabled where required.
* [ ] FortiGuard intelligence available.
* [ ] Botnet hit logs monitored.
* [ ] False positives reviewed.

---

## Safe Search

* [ ] Safe Search requirement defined.
* [ ] Supported search engines tested.
* [ ] User experience validated.

---

## DNS Translation

* [ ] Translation entry configured.
* [ ] Original destination verified.
* [ ] Translated IP verified.
* [ ] Firewall policy allows translated traffic.
* [ ] DMZ/internal policy reviewed.

---

## DoT / DoH

* [ ] Secure DNS requirements documented.
* [ ] TLS inspection requirements evaluated.
* [ ] Certificate trust verified.
* [ ] Application control evaluated.
* [ ] DoH/DoT bypass scenarios tested.

---

# 27. NSE Exam Traps

## Trap #1 — DNS Filter vs Web Filter

```text
DNS Filter
→ Domain-level control

Web Filter
→ URL/category/content-level control
```

---

## Trap #2 — DNS Filter Happens Early

```text
DNS Query
    ↓
DNS Filter
    ↓
Connection
```

DNS filtering can stop the connection before the client reaches the web destination.

---

## Trap #3 — Local Filter vs FortiGuard

```text
Local Domain Filter
        ↓
FortiGuard
```

A local match can determine the action before the request proceeds to subsequent rating logic.

---

## Trap #4 — Monitor Is Not Block

```text
MONITOR
→ Allow + Log

BLOCK
→ Deny
```

---

## Trap #5 — DNS Cache

DNS cache improves performance but can introduce stale data depending on TTL/cache behavior.

```text
Cache
→ Faster

But

Cache
→ Potentially stale
```

---

## Trap #6 — `least-rtt`

```text
least-rtt
→ Prefer DNS server with lowest measured RTT
```

It is not the same concept as simple sequential failover.

---

## Trap #7 — DoT vs DoH

```text
DoT
→ TLS
→ Port 853

DoH
→ HTTPS
→ Port 443
```

---

## Trap #8 — DNS Record Types

```text
A
→ IPv4

AAAA
→ IPv6

CNAME
→ Alias

MX
→ Mail

TXT
→ Text
```

---

## Trap #9 — Botnet C&C

DNS filtering can detect/block domains associated with known C&C infrastructure.

```text
DNS Query
    ↓
Botnet Intelligence
    ↓
Known C&C?
    ↓
BLOCK
```

---

## Trap #10 — DNS Translation

DNS translation changes the DNS answer presented to the client.

```text
Original IP
     ↓
FortiGate
     ↓
Translated IP
```

It does **not automatically replace the need for the corresponding security policy/NAT/routing design**.

---

## Trap #11 — DNS Cache vs DNS Database

Do not assume every DNS-related entry is the same cache.

```text
DNS Cache
→ Cached resolution results

DNS Database
→ DNS-related database/state

FortiGuard Rating Cache
→ Cached security-rating information
```

---

## Trap #12 — Secure DNS

Secure DNS does not mean the domain is automatically safe.

```text
Encrypted DNS
≠
Safe Domain
```

Encryption protects the DNS communication channel; DNS filtering/reputation determines whether the requested domain should be trusted.

---

# 28. Quick Revision Card

```text
                         DNS FILTER
                             |
          +------------------+------------------+
          |                  |                  |
          v                  v                  v
     LOCAL FILTER       FORTIGUARD         BOTNET/C&C
          |                  |                  |
          +------------------+------------------+
                             |
                             v
                    ALLOW / MONITOR / BLOCK
                             |
                             v
                       DNS PROXY
                             |
          +------------------+------------------+
          |                  |                  |
          v                  v                  v
        CACHE           UPSTREAM DNS       SECURE DNS
          |                  |                  |
          +------------------+------------------+
                             |
                             v
                         CLIENT
```

---

## DNS Resolution Model

```text
Client
  |
  v
DNS Query
  |
  v
FortiGate DNSProxy
  |
  +---- DNS Filter
  |
  +---- Local Domain Filter
  |
  +---- Cache
  |
  +---- FortiGuard Rating
  |
  +---- Botnet Intelligence
  |
  +---- Upstream Resolver
  |
  v
DNS Response
```

---

## DNS Protocol Memory

```text
Traditional DNS
    |
    +---- UDP/TCP 53

DNS over TLS
    |
    +---- TLS
    +---- 853

DNS over HTTPS
    |
    +---- HTTPS
    +---- 443
```

---

## DNS Record Memory

```text
A       → IPv4
AAAA    → IPv6
CNAME   → Alias
MX      → Mail
TXT     → Text
IN      → Internet Class
```

---

## Core Commands

```bash
# FortiGuard status
get system fortiguard

# DNSProxy diagnostic menu
diagnose test application dnsproxy

# Secure DNS / FortiGuard DNSProxy information
diagnose test application dnsproxy 3

# Clear DNS cache
diagnose test application dnsproxy 1

# DNS statistics
diagnose test application dnsproxy 2

# Dump FQDN information
diagnose test application dnsproxy 6

# Dump DNS cache
diagnose test application dnsproxy 7

# Dump DNS database
diagnose test application dnsproxy 8

# Reload DNS database
diagnose test application dnsproxy 9

# Dump secure DNS policy/profile
diagnose test application dnsproxy 10

# Dump botnet domains
diagnose test application dnsproxy 11

# Show hostname cache
diagnose test application dnsproxy 13

# Clear hostname cache
diagnose test application dnsproxy 14

# Show Secure DNS rating cache
diagnose test application dnsproxy 15

# Clear Secure DNS rating cache
diagnose test application dnsproxy 16

# DNS debug
diagnose test application dnsproxy 17

# DNS memory diagnostics
diagnose test application dnsproxy 18

# Restart DNSProxy worker
diagnose test application dnsproxy 99

# Botnet database
diagnose sys botnet list 9000 10

# Botnet IP hits
diagnose sys botnet-ip hit

# DNS UTM logs
execute log filter category utm-dns
execute log display
```

---

# 🔥 Final NSE Memory Aid

> **DNS Filter controls domains at the DNS-resolution stage; Web Filter controls web destinations/content; Antivirus inspects files/content; FortiGuard provides cloud-based rating and security intelligence; DNSProxy manages DNS processing, caching, and forwarding; and Botnet/C&C intelligence can stop malicious resolution before the client establishes the connection.**

```text
DNS FILTER
    ↓
DOMAIN DECISION

WEB FILTER
    ↓
URL / CATEGORY / CONTENT

ANTIVIRUS
    ↓
FILE / CONTENT

FORTIGUARD
    ↓
RATING / SECURITY INTELLIGENCE

DNSPROXY
    ↓
CACHE / FORWARD / RESOLUTION

BOTNET
    ↓
C&C DETECTION
```

---

## 🧠 30-Second Exam Review

```text
DNS Filter
→ Domain-level filtering

DNSProxy
→ DNS processing engine

DNS Cache
→ Faster repeated resolution

FortiGuard
→ Rating + Security Intelligence

Botnet DB
→ C&C intelligence

A
→ IPv4

AAAA
→ IPv6

CNAME
→ Alias

MX
→ Mail

TXT
→ Text

DoT
→ TLS / 853

DoH
→ HTTPS / 443

least-rtt
→ Lowest measured DNS latency

Monitor
→ Allow + Log

Block
→ Deny

DNS Translation
→ Modify DNS answer

Safe Search
→ Restrict search results
```

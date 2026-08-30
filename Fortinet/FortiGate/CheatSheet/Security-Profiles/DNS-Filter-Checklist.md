# 🔗 SheynShield Resources

# FortiGate DNS Filter — Security & Troubleshooting Checklist

> **FortiOS 7.x | DNS Filter | DNSProxy | FortiGuard Secure DNS | Botnet C&C | DNS Translation | DoH | DoT | DNS Cache | NSE4 | NSE7**

**Audience:** FortiGate / NSE4–NSE7 / Network & Security Engineers
**Goal:** Design, deploy, validate, troubleshoot, and optimize FortiGate DNS filtering.

---

## 📋 Table of Contents

* [1. DNS Filter Fundamentals](#1-dns-filter-fundamentals)
* [2. DNS Security Architecture](#2-dns-security-architecture)
* [3. DNS Filter vs Other Security Profiles](#3-dns-filter-vs-other-security-profiles)
* [4. DNSProxy](#4-dnsproxy)
* [5. DNS Cache](#5-dns-cache)
* [6. System DNS](#6-system-dns)
* [7. DNS Server Selection](#7-dns-server-selection)
* [8. FortiGuard Secure DNS](#8-fortiguard-secure-dns)
* [9. DNS Filter Profile](#9-dns-filter-profile)
* [10. DNS Filter Actions](#10-dns-filter-actions)
* [11. Local Domain Filtering](#11-local-domain-filtering)
* [12. Wildcard vs Regex](#12-wildcard-vs-regex)
* [13. FortiGuard DNS Rating](#13-fortiguard-dns-rating)
* [14. Botnet C&C Protection](#14-botnet-c-c-protection)
* [15. Safe Search](#15-safe-search)
* [16. DNS Translation](#16-dns-translation)
* [17. DNS Database & FQDN State](#17-dns-database--fqdn-state)
* [18. DNS over TLS](#18-dns-over-tls)
* [19. DNS over HTTPS](#19-dns-over-https)
* [20. DNS Query Types](#20-dns-query-types)
* [21. DNS Logging](#21-dns-logging)
* [22. DNSProxy Diagnostics](#22-dnsproxy-diagnostics)
* [23. FortiGuard DNS Diagnostics](#23-fortiguard-dns-diagnostics)
* [24. DNS Cache Troubleshooting](#24-dns-cache-troubleshooting)
* [25. End-to-End Troubleshooting](#25-end-to-end-troubleshooting)
* [26. Production Deployment Checklist](#26-production-deployment-checklist)
* [27. DoH / DoT Security Checklist](#27-doh--dot-security-checklist)
* [28. Performance & Capacity Checklist](#28-performance--capacity-checklist)
* [29. NSE Exam Traps](#29-nse-exam-traps)
* [30. Quick Revision Card](#30-quick-revision-card)
* [31. Core CLI Reference](#31-core-cli-reference)

---

# 1. DNS Filter Fundamentals

## ✅ Core Concepts

* [ ] Understand that **DNS Filter operates at the DNS-resolution stage**.
* [ ] Understand that DNS filtering can block a domain before the client establishes the subsequent connection.
* [ ] Understand the difference between domain-level filtering and URL/content inspection.
* [ ] Understand that DNS filtering depends on the DNS traffic actually being visible to FortiGate.
* [ ] Understand that encrypted DNS protocols such as DoH and DoT can change the inspection model.

### Basic Flow

```text
Client
   |
   | DNS Query
   v
FortiGate
   |
   +---- Local Domain Filter
   |
   +---- FortiGuard Rating
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
```

## ✅ Typical Use Cases

* [ ] Malicious-domain blocking
* [ ] Phishing-domain protection
* [ ] Botnet C&C protection
* [ ] Category-based DNS filtering
* [ ] Safe Search enforcement
* [ ] Local domain policies
* [ ] DNS redirection
* [ ] DNS translation
* [ ] DNS security monitoring
* [ ] Enterprise DNS policy enforcement

---

# 2. DNS Security Architecture

## ✅ Understand the Processing Model

```text
                 CLIENT
                    |
                    v
                DNS Query
                    |
                    v
               DNSProxy
                    |
       +------------+------------+
       |            |            |
       v            v            v
   DNS Filter     Cache      FortiGuard
       |            |            |
       +------------+------------+
                    |
                    v
              Policy Decision
                    |
          +---------+---------+
          |                   |
        BLOCK                ALLOW
          |                   |
         STOP             DNS Response
```

## ✅ Security Architecture

* [ ] DNS Filter is enabled.
* [ ] DNSProxy operation is understood.
* [ ] FortiGuard connectivity is verified.
* [ ] Local domain filtering is reviewed.
* [ ] Botnet intelligence is evaluated.
* [ ] DNS cache behavior is understood.
* [ ] DNS logging is enabled where required.
* [ ] Secure DNS requirements are documented.

---

# 3. DNS Filter vs Other Security Profiles

| Security Feature        | Primary Target                      | Example                        |
| ----------------------- | ----------------------------------- | ------------------------------ |
| **DNS Filter**          | DNS/domain                          | `malicious.example`            |
| **Web Filter**          | URL/category/content                | `https://example.com/download` |
| **Antivirus**           | Files/content                       | Malware                        |
| **IPS**                 | Network/application attack patterns | Exploit                        |
| **Application Control** | Applications                        | Cloud storage / messaging      |

## 🧠 Mental Model

```text
DNS Filter
    ↓
"Where am I going?"

Web Filter
    ↓
"What website / URL am I accessing?"

Antivirus
    ↓
"What file/content am I receiving?"

IPS
    ↓
"Is this traffic matching an attack pattern?"
```

## ✅ Verify

* [ ] DNS filtering is not confused with Web Filtering.
* [ ] DNS filtering is not treated as malware/file inspection.
* [ ] DNS filtering is not considered a replacement for IPS.
* [ ] Multiple security layers are used when appropriate.

---

# 4. DNSProxy

FortiGate uses the **DNSProxy** process for DNS-related processing.

```text
Client
   |
   v
DNSProxy
   |
   +---- DNS Cache
   |
   +---- DNS Filter
   |
   +---- FortiGuard
   |
   +---- Upstream DNS
   |
   v
DNS Response
```

## ✅ DNSProxy Checklist

* [ ] Know the `dnsproxy` process.
* [ ] Know how to access DNSProxy diagnostics.
* [ ] Know how to inspect DNS cache.
* [ ] Know how to inspect DNS database/state.
* [ ] Know how to inspect Secure DNS state.
* [ ] Know how to clear DNS cache.
* [ ] Know how to reload DNS information.
* [ ] Know how to inspect DNS statistics.

### Primary Diagnostic Command

```bash
diagnose test application dnsproxy
```

---

# 5. DNS Cache

DNS caching prevents unnecessary repeated upstream lookups.

## ✅ Cache Flow

```text
Client
  |
  | Query example.com
  v
FortiGate
  |
  +---- Cache HIT
  |       |
  |       +---- Return cached result
  |
  +---- Cache MISS
          |
          +---- Query upstream DNS
```

## ✅ Validate

* [ ] DNS cache is enabled/configured appropriately.
* [ ] Cache size is reviewed.
* [ ] Cache TTL is reviewed.
* [ ] Negative-response caching is understood.
* [ ] Stale-record behavior is considered during troubleshooting.
* [ ] Cache can be cleared during DNS troubleshooting.

### Important Trade-off

```text
DNS Cache
    ↓
Lower latency
    ↓
Fewer upstream queries

BUT

Cached information
    ↓
Can become stale
```

---

# 6. System DNS

## ✅ Review DNS Configuration

Example:

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

> **Production note:** Replace example/public DNS servers with approved enterprise resolvers where required.

## ✅ DNS Timeout

```bash
set timeout 5
```

* [ ] Timeout value is appropriate.
* [ ] High timeout values are not hiding upstream DNS failures.
* [ ] DNS latency is monitored.

## ✅ DNS Retry

```bash
set retry 2
```

* [ ] Retry behavior is understood.
* [ ] Excessive retries are avoided.
* [ ] Upstream DNS availability is verified.

## ✅ Negative DNS Responses

```bash
set cache-notfound-response disable
```

* [ ] Negative caching behavior is reviewed.
* [ ] NXDOMAIN troubleshooting considers negative caching.

---

# 7. DNS Server Selection

FortiOS can select upstream DNS servers according to the configured method.

Example:

```bash
set server-select-method least-rtt
```

## ✅ `least-rtt`

* [ ] Understand that `least-rtt` prefers the DNS server with the lowest measured latency.
* [ ] Understand that it is not equivalent to simple sequential failover.

Example:

```text
DNS-1 → 40 ms
DNS-2 → 15 ms
DNS-3 → 80 ms

Preferred:
DNS-2
```

## 🧠 Exam Memory

```text
least-rtt
    ↓
Lowest measured DNS latency
```

* [ ] Verify supported server-selection methods against the deployed FortiOS version.

---

# 8. FortiGuard Secure DNS

FortiGuard can provide DNS resolution and security intelligence.

```text
Client
   |
   v
FortiGate
   |
   v
FortiGuard Secure DNS
   |
   +---- Resolution
   +---- Rating
   +---- Security Intelligence
```

## ✅ FortiGuard Checklist

* [ ] FortiGuard license is valid.
* [ ] FortiGuard connectivity is working.
* [ ] DNS rating service is reachable.
* [ ] Secure DNS state is verified.
* [ ] Anycast requirements are evaluated.

Example:

```bash
config system fortiguard
    set fortiguard-anycast enable
end
```

Additional parameters may include:

```bash
config system fortiguard
    set fortiguard-anycast enable
    set fortiguard-anycast-source fortinet
    set anycast-sdns-server-ip 0.0.0.0
    set anycast-sdns-server-port 853
end
```

> Always verify exact syntax and supported parameters against the target FortiOS release.

---

# 9. DNS Filter Profile

## ✅ Profile Checklist

* [ ] DNS Filter profile created.
* [ ] Appropriate domain categories selected.
* [ ] Local filters configured.
* [ ] FortiGuard rating enabled where required.
* [ ] Botnet/C&C protection evaluated.
* [ ] Safe Search configured if required.
* [ ] Rating-error behavior reviewed.
* [ ] Logging enabled where required.

Typical GUI location:

```text
Security Profiles
    └── DNS Filter
```

---

# 10. DNS Filter Actions

## ✅ Understand Actions

| Action      | Result         |
| ----------- | -------------- |
| **Allow**   | Permit request |
| **Monitor** | Permit + log   |
| **Block**   | Deny request   |

## 🧠 Critical Difference

```text
MONITOR
    ↓
ALLOW + LOG
```

```text
BLOCK
    ↓
DENY
```

## ✅ Verify

* [ ] Monitor is not confused with Block.
* [ ] Block behavior is tested.
* [ ] Logging confirms the selected action.
* [ ] False positives are reviewed before aggressive blocking.

---

# 11. Local Domain Filtering

Local/static domain filtering allows administrator-defined rules.

```text
DNS Query
    |
    v
Local Domain Filter
    |
    +---- Match → Configured Action
    |
    +---- No Match
             |
             v
        FortiGuard
```

## ✅ Checklist

* [ ] Local domain filter exists where required.
* [ ] Exact domains are reviewed.
* [ ] Wildcard behavior is understood.
* [ ] Regex behavior is understood.
* [ ] Action is explicitly configured.
* [ ] Test domains are used to validate matching.

Conceptual configuration:

```bash
config dnsfilter domain-filter
    ...
end
```

> Exact CLI structure can vary by FortiOS version.

---

# 12. Wildcard vs Regex

## ✅ Wildcard

Example:

```text
*.host
```

Use when a broad suffix-style match is appropriate.

## ✅ Regex

Example:

```text
^.*\.host$
```

Use when more explicit pattern matching is required.

## 🧠 Comparison

| Method       | Best Use                 |
| ------------ | ------------------------ |
| **Wildcard** | Simple domain matching   |
| **Regex**    | Precise pattern matching |

### Memory Aid

```text
Wildcard
→ Simple

Regex
→ Precise
```

## ✅ Validate

* [ ] Test positive matches.
* [ ] Test negative matches.
* [ ] Test parent/child domains.
* [ ] Test unexpected suffixes.
* [ ] Confirm behavior on the exact FortiOS release.

---

# 13. FortiGuard DNS Rating

FortiGuard can classify domains using cloud-based security intelligence.

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
    +---- Reputation
    +---- Security Intelligence
    |
    v
Policy Decision
```

## ✅ Verify

* [ ] FortiGuard service is reachable.
* [ ] Rating requests are working.
* [ ] Category policies are configured.
* [ ] Rating failures are understood.
* [ ] Logging provides enough information for troubleshooting.

Potential categories include:

* [ ] Malware
* [ ] Phishing
* [ ] Botnet
* [ ] Adult
* [ ] Gambling
* [ ] Social Media
* [ ] Other FortiGuard categories

---

# 14. Botnet C&C Protection

DNS filtering can help detect domains associated with known **Command & Control infrastructure**.

```text
Client
   |
   | DNS Query
   v
FortiGate
   |
   v
Botnet Intelligence
   |
   +---- Known C&C?
          |
          +---- YES → BLOCK
          |
          +---- NO → Continue
```

## ✅ Botnet Checklist

* [ ] Botnet/C&C protection is enabled where required.
* [ ] FortiGuard intelligence is available.
* [ ] Botnet database status is reviewed.
* [ ] Botnet hits are monitored.
* [ ] False positives are investigated.

### Diagnostic Commands

```bash
diagnose sys botnet list 9000 10
```

```bash
diagnose sys botnet-ip hit
```

## 🧠 Exam Concept

```text
DNS Query
    ↓
Botnet Intelligence
    ↓
Known C&C?
    ↓
Block
```

---

# 15. Safe Search

FortiGate can enforce Safe Search for supported search engines.

Potentially supported services include:

* [ ] Google
* [ ] Bing
* [ ] Yahoo
* [ ] Yandex

## ✅ Checklist

* [ ] Safe Search requirement is documented.
* [ ] Supported search engines are identified.
* [ ] Safe Search enforcement is enabled where required.
* [ ] User experience is tested.
* [ ] Bypass scenarios are tested.

```text
Client
   |
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

---

# 16. DNS Translation

DNS translation modifies the DNS answer returned to the client.

Example:

```text
Client asks:

service.example
      |
      v
FortiGate
      |
      +---- Original IP
      |     185.143.233.120
      |
      +---- Translated IP
            192.168.20.200
```

## ✅ DNS Translation Checklist

* [ ] Original domain is identified.
* [ ] Original DNS response is verified.
* [ ] Translation rule is configured.
* [ ] Translated IP is correct.
* [ ] Client receives the expected DNS answer.
* [ ] Routing to the translated destination exists.
* [ ] Firewall policy permits the resulting traffic.
* [ ] NAT requirements are evaluated separately.

## ⚠️ Critical Concept

```text
DNS Translation
    ≠
Automatic Security Policy
```

DNS translation changes the DNS answer. It does **not automatically replace the required firewall policy, routing, or NAT design**.

---

# 17. DNS Database & FQDN State

DNS-related state can include:

```text
DNS Cache
DNS Database
FQDN Database
Secure DNS State
Botnet Database
FortiGuard Rating Cache
Hostname Cache
```

## ✅ Understand the Difference

* [ ] DNS cache is understood as cached resolution information.
* [ ] DNS database/state is not assumed to be identical to the DNS cache.
* [ ] FQDN state is understood separately.
* [ ] FortiGuard rating cache is understood separately.
* [ ] Hostname cache is understood separately.

---

# 18. DNS over TLS

**DoT = DNS over TLS**

Traditional DNS:

```text
Client
   |
   | DNS
   v
Resolver
```

DoT:

```text
Client
   |
   | TLS
   | TCP/853
   v
DNS Resolver
```

## ✅ DoT Checklist

* [ ] Understand DNS over TLS.
* [ ] Understand TLS encryption.
* [ ] Understand port `853`.
* [ ] Identify approved DoT resolvers.
* [ ] Evaluate certificate validation requirements.
* [ ] Evaluate FortiGate inspection capabilities for the deployed FortiOS version.
* [ ] Test bypass scenarios.

### Memory

```text
DoT
→ TLS
→ 853
```

---

# 19. DNS over HTTPS

**DoH = DNS over HTTPS**

```text
Client
   |
   | HTTPS
   | TCP/443
   v
DoH Resolver
```

## ✅ Understand the Security Challenge

```text
HTTPS/443
   |
   +---- Web Traffic
   |
   +---- DoH
```

Because DoH uses HTTPS:

* [ ] Identify potential DoH bypass paths.
* [ ] Evaluate Application Control.
* [ ] Evaluate Web Filtering.
* [ ] Evaluate TLS inspection where appropriate.
* [ ] Test browser-specific DoH behavior.
* [ ] Define approved DNS resolvers.
* [ ] Monitor unauthorized secure DNS usage.

## 🧠 Memory

```text
Traditional DNS
→ 53

DoT
→ TLS / 853

DoH
→ HTTPS / 443
```

---

# 20. DNS Query Types

## ✅ Know Common QTYPEs

| QTYPE | Record | Purpose                |
| ----: | ------ | ---------------------- |
|   `1` | A      | IPv4 address           |
|  `28` | AAAA   | IPv6 address           |
|   `5` | CNAME  | Canonical name / alias |
|  `15` | MX     | Mail exchanger         |
|  `16` | TXT    | Text information       |

## 🧠 Memory

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
→ Internet
```

---

# 21. DNS Logging

## ✅ Logging Checklist

* [ ] DNS logging is enabled where required.
* [ ] Blocked domains are visible.
* [ ] Client identity is available where supported.
* [ ] Domain name is logged.
* [ ] Query type is available where supported.
* [ ] Action is logged.
* [ ] Category/rating information is reviewed.
* [ ] Timestamp is available.
* [ ] Logs are correlated with client activity.

### CLI

```bash
execute log filter category utm-dns
execute log display
```

## Useful for Investigating

* [ ] Blocked domains
* [ ] Suspicious DNS activity
* [ ] Botnet activity
* [ ] DNS filtering decisions
* [ ] Client behavior
* [ ] False positives

---

# 22. DNSProxy Diagnostics

## ✅ Primary Command

```bash
diagnose test application dnsproxy
```

## Diagnostic Areas

Know how to investigate:

* [ ] DNS cache
* [ ] DNS statistics
* [ ] DNS settings
* [ ] FQDN state
* [ ] DNS database
* [ ] Secure DNS state
* [ ] Botnet domains
* [ ] Hostname cache
* [ ] Secure DNS rating cache
* [ ] DNS debugging
* [ ] DNS memory
* [ ] DNSProxy worker state

---

# 23. FortiGuard DNS Diagnostics

A useful diagnostic command is:

```bash
diagnose test application dnsproxy 3
```

Depending on FortiOS version, diagnostic output may expose information related to:

* [ ] Secure DNS
* [ ] FortiGuard server
* [ ] Service state
* [ ] License/service status
* [ ] Response time
* [ ] Failures
* [ ] Timeouts
* [ ] Pending requests

## Fields to Investigate

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

## ⚠️ Important

* [ ] Do not assume diagnostic field meanings are identical across all FortiOS releases.
* [ ] Verify implementation-specific output against the target build.

---

# 24. DNS Cache Troubleshooting

## Clear DNS Cache

```bash
diagnose test application dnsproxy 1
```

Use when:

* [ ] DNS records appear stale.
* [ ] A DNS change is not reflected.
* [ ] Cached resolution is suspected.
* [ ] DNS troubleshooting requires a clean cache.

## DNS Statistics

```bash
diagnose test application dnsproxy 2
```

Review:

* [ ] DNS latency
* [ ] Forwarding
* [ ] Retries
* [ ] Server health
* [ ] Resolution behavior

### Important Indicator

```text
Latency = -1
```

may indicate:

* [ ] DNS server is unreachable.
* [ ] No valid measurement is available.
* [ ] Further diagnostics are required.

---

# 25. End-to-End Troubleshooting

## 🔍 Step 1 — Client

* [ ] Verify client DNS configuration.
* [ ] Identify configured DNS server.
* [ ] Test resolution directly from the client.
* [ ] Determine whether the client uses traditional DNS, DoT, or DoH.

---

## 🔍 Step 2 — Firewall Policy

Verify:

```text
LAN → WAN
```

* [ ] DNS traffic is allowed.
* [ ] Correct policy is being matched.
* [ ] DNS Filter is attached where required.
* [ ] Inspection settings are appropriate.
* [ ] No earlier policy bypasses the intended security policy.

---

## 🔍 Step 3 — DNS Filter

* [ ] Correct DNS Filter profile is attached.
* [ ] Local domain filters are reviewed.
* [ ] Category policy is correct.
* [ ] Action is correct.
* [ ] Monitor vs Block behavior is understood.

---

## 🔍 Step 4 — FortiGuard

```bash
get system fortiguard
```

Verify:

* [ ] FortiGuard connectivity.
* [ ] Service status.
* [ ] License/service availability.
* [ ] Rating functionality.

---

## 🔍 Step 5 — DNSProxy

```bash
diagnose test application dnsproxy
```

Check:

* [ ] DNS cache.
* [ ] DNS settings.
* [ ] DNS database.
* [ ] FQDN state.
* [ ] Statistics.
* [ ] Secure DNS.
* [ ] Hostname cache.

---

## 🔍 Step 6 — Secure DNS

```bash
diagnose test application dnsproxy 3
```

Review:

* [ ] RTT.
* [ ] Failures.
* [ ] Timeouts.
* [ ] Ready state.
* [ ] TLS state.
* [ ] Pending requests.

---

## 🔍 Step 7 — Logs

```bash
execute log filter category utm-dns
execute log display
```

Check:

* [ ] Client.
* [ ] Domain.
* [ ] Action.
* [ ] Category.
* [ ] Timestamp.
* [ ] Policy/profile.

---

# 26. Production Deployment Checklist

## 🛡️ DNS Filter

* [ ] DNS Filter profile created.
* [ ] Required categories configured.
* [ ] Local domain filters reviewed.
* [ ] Actions tested.
* [ ] FortiGuard connectivity verified.
* [ ] Rating-error behavior reviewed.
* [ ] DNS logging enabled.
* [ ] False positives reviewed.

## ⚡ DNS Cache

* [ ] Cache size reviewed.
* [ ] Cache TTL reviewed.
* [ ] Negative caching reviewed.
* [ ] Timeout configured.
* [ ] Retry configured.
* [ ] DNS server selection reviewed.

## ☁️ FortiGuard

* [ ] License verified.
* [ ] Connectivity tested.
* [ ] Anycast evaluated.
* [ ] Secure DNS verified.
* [ ] Rating service operational.

## 🤖 Botnet Protection

* [ ] C&C protection enabled where required.
* [ ] Botnet intelligence available.
* [ ] Botnet hits monitored.
* [ ] False positives reviewed.

## 🔎 Safe Search

* [ ] Requirement documented.
* [ ] Supported search engines identified.
* [ ] Enforcement tested.
* [ ] User experience validated.

## 🔄 DNS Translation

* [ ] Translation rule configured.
* [ ] Original DNS answer verified.
* [ ] Translated IP verified.
* [ ] Routing verified.
* [ ] Firewall policy verified.
* [ ] NAT requirements evaluated.

---

# 27. DoH / DoT Security Checklist

## 🔐 Secure DNS

* [ ] Enterprise DNS architecture is documented.
* [ ] Approved DNS resolvers are documented.
* [ ] Traditional DNS is monitored.
* [ ] DoT usage is understood.
* [ ] DoH usage is understood.
* [ ] Unauthorized DNS resolvers are identified.
* [ ] Application Control is evaluated.
* [ ] TLS inspection requirements are evaluated.
* [ ] Browser bypass scenarios are tested.
* [ ] IPv6 DNS bypass is considered.
* [ ] Endpoint-level DNS policies are considered.

## 🧠 Security Model

```text
Traditional DNS
      ↓
FortiGate DNS visibility

DoT
      ↓
Encrypted DNS / 853

DoH
      ↓
HTTPS / 443
      ↓
May blend with normal HTTPS
```

---

# 28. Performance & Capacity Checklist

DLP-style content inspection is not the only feature that consumes resources; DNS security also introduces processing overhead.

## ✅ Capacity Planning

* [ ] Expected DNS query rate is known.
* [ ] CPU utilization is monitored.
* [ ] Memory utilization is monitored.
* [ ] FortiGuard latency is monitored.
* [ ] DNS cache effectiveness is evaluated.
* [ ] Upstream DNS latency is measured.
* [ ] Excessive logging is avoided.
* [ ] DNS failures are correlated with resource usage.
* [ ] High-volume clients are identified.
* [ ] Production load testing is performed where appropriate.

## Performance Model

```text
DNS Request
    ↓
DNSProxy
    ↓
Filter / Rating
    ↓
Cache / Upstream Resolver
    ↓
Response
```

---

# 29. NSE Exam Traps

## ❌ Trap 1 — DNS Filter ≠ Web Filter

```text
DNS Filter
→ Domain-level decision

Web Filter
→ URL/category/content
```

---

## ❌ Trap 2 — Monitor ≠ Block

```text
Monitor
→ Allow + Log

Block
→ Deny
```

---

## ❌ Trap 3 — Secure DNS ≠ Safe Domain

```text
Encrypted DNS
≠
Trusted Domain
```

Encryption protects the DNS communication channel.

Filtering/reputation determines whether the domain should be allowed.

---

## ❌ Trap 4 — DoT vs DoH

```text
DoT
→ TLS
→ 853

DoH
→ HTTPS
→ 443
```

---

## ❌ Trap 5 — `least-rtt`

```text
least-rtt
→ Lowest measured DNS latency
```

It is not simply sequential failover.

---

## ❌ Trap 6 — DNS Cache

```text
Cache
→ Faster

Cache
→ Potentially stale
```

---

## ❌ Trap 7 — DNS Translation

```text
DNS Translation
→ Changes DNS answer
```

It does not automatically create the required routing/security/NAT policy.

---

## ❌ Trap 8 — Botnet C&C

```text
DNS Query
    ↓
Botnet Intelligence
    ↓
Known C&C?
    ↓
Block
```

---

## ❌ Trap 9 — DNS Record Types

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

## ❌ Trap 10 — DNS Database ≠ DNS Cache

```text
DNS Cache
→ Cached resolution information

DNS Database
→ DNS-related database/state

Rating Cache
→ Cached FortiGuard rating information
```

---

# 30. Quick Revision Card

## 🧠 30-Second DNS Review

```text
DNS FILTER
    ↓
DOMAIN-LEVEL DECISION

DNSPROXY
    ↓
DNS PROCESSING

DNS CACHE
    ↓
FASTER RESOLUTION

FORTIGUARD
    ↓
RATING + SECURITY INTELLIGENCE

BOTNET
    ↓
C&C INTELLIGENCE

LOCAL FILTER
    ↓
ADMIN-DEFINED DOMAIN RULES

MONITOR
    ↓
ALLOW + LOG

BLOCK
    ↓
DENY

DNS TRANSLATION
    ↓
MODIFY DNS ANSWER

DoT
    ↓
TLS / 853

DoH
    ↓
HTTPS / 443
```

---

## 🧠 DNS Architecture

```text
                    CLIENT
                       |
                       v
                  DNS QUERY
                       |
                       v
                   DNSPROXY
                       |
        +--------------+--------------+
        |              |              |
        v              v              v
   DNS FILTER        CACHE        FORTIGUARD
        |              |              |
        |              |              |
        +--------------+--------------+
                       |
                       v
                POLICY DECISION
                       |
             +---------+---------+
             |                   |
           BLOCK                ALLOW
             |                   |
            STOP             DNS RESPONSE
```

---

## 🧠 DNS Protocol Memory

```text
Traditional DNS
→ UDP/TCP 53

DNS over TLS
→ TLS / 853

DNS over HTTPS
→ HTTPS / 443
```

---

## 🧠 DNS Record Memory

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

IN
→ Internet Class
```

---

# 31. Core CLI Reference

## FortiGuard

```bash
get system fortiguard
```

## DNSProxy Diagnostic Menu

```bash
diagnose test application dnsproxy
```

## Clear DNS Cache

```bash
diagnose test application dnsproxy 1
```

## DNS Statistics

```bash
diagnose test application dnsproxy 2
```

## DNS / Secure DNS Diagnostic Information

```bash
diagnose test application dnsproxy 3
```

## Dump FQDN

```bash
diagnose test application dnsproxy 6
```

## Dump DNS Cache

```bash
diagnose test application dnsproxy 7
```

## Dump DNS Database

```bash
diagnose test application dnsproxy 8
```

## Reload DNS Database

```bash
diagnose test application dnsproxy 9
```

## Dump Secure DNS Policy/Profile

```bash
diagnose test application dnsproxy 10
```

## Dump Botnet Domains

```bash
diagnose test application dnsproxy 11
```

## Show Hostname Cache

```bash
diagnose test application dnsproxy 13
```

## Clear Hostname Cache

```bash
diagnose test application dnsproxy 14
```

## Show Secure DNS Rating Cache

```bash
diagnose test application dnsproxy 15
```

## Clear Secure DNS Rating Cache

```bash
diagnose test application dnsproxy 16
```

## DNS Debug

```bash
diagnose test application dnsproxy 17
```

## DNS Memory Diagnostics

```bash
diagnose test application dnsproxy 18
```

## Restart DNSProxy Worker

```bash
diagnose test application dnsproxy 99
```

## Botnet Database

```bash
diagnose sys botnet list 9000 10
```

## Botnet IP Hits

```bash
diagnose sys botnet-ip hit
```

## DNS UTM Logs

```bash
execute log filter category utm-dns
execute log display
```

---

# 🔥 Final NSE Memory Aid

> **FortiGate DNS Filter controls domains at the DNS-resolution stage. DNSProxy manages DNS processing, caching, forwarding, and related DNS state. FortiGuard provides domain rating and security intelligence, while Botnet/C&C intelligence can identify malicious infrastructure. DoT encrypts DNS using TLS, while DoH transports DNS through HTTPS.**

```text
DNS FILTER
    ↓
DOMAIN DECISION

DNSPROXY
    ↓
PROCESS / CACHE / FORWARD

FORTIGUARD
    ↓
RATING / SECURITY INTELLIGENCE

BOTNET
    ↓
C&C DETECTION

DNS TRANSLATION
    ↓
MODIFY DNS ANSWER

DoT
    ↓
TLS / 853

DoH
    ↓
HTTPS / 443
```

---

# 🚨 Final Production Validation

Before declaring a FortiGate DNS Filter deployment production-ready:

* [ ] DNS Filter profile validated.
* [ ] Local domain rules validated.
* [ ] FortiGuard rating validated.
* [ ] Botnet/C&C protection validated.
* [ ] DNS cache behavior validated.
* [ ] DNS server selection validated.
* [ ] DNS logging validated.
* [ ] DNSProxy diagnostics tested.
* [ ] DNS translation tested where applicable.
* [ ] DoH bypass scenarios tested.
* [ ] DoT behavior tested.
* [ ] IPv6 DNS paths tested.
* [ ] False positives reviewed.
* [ ] Performance monitored.
* [ ] Failure scenarios tested.
* [ ] Recovery procedures documented.
* [ ] FortiOS-specific CLI syntax verified.
* [ ] Configuration backup completed.

---

> ## 🛡️ SheynShield Engineering Note
>
> Don't think of FortiGate DNS Filter as simply **"block malicious domains."**
>
> The real architecture is:
>
> **Client → DNSProxy → Local Filter / Cache / FortiGuard → Policy Decision → DNS Response**
>
> For modern encrypted DNS:
>
> **DoT → TLS/853**
>
> **DoH → HTTPS/443**
>
> And for troubleshooting:
>
> **Client → Policy → DNS Filter → DNSProxy → Cache → FortiGuard → Upstream DNS → Logs**
>
> The most important NSE mental model is:
>
> **DNS Filter = Domain Decision**
>
> **DNSProxy = DNS Processing**
>
> **FortiGuard = Rating & Intelligence**
>
> **Botnet = C&C Intelligence**
>
> **DoT = DNS over TLS**
>
> **DoH = DNS over HTTPS**

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

## 🔎 Keywords

`FortiGate DNS Filter` · `FortiOS DNS Filter` · `FortiGate DNSProxy` · `FortiGuard Secure DNS` · `FortiGate DNS troubleshooting` · `FortiGate DoH` · `FortiGate DoT` · `FortiGate Botnet C&C` · `FortiGate DNS Translation` · `FortiGate DNS Cache` · `Fortinet NSE4 DNS Filter` · `Fortinet NSE7 DNS` · `FortiGate security profiles` · `FortiOS 7.x DNS`

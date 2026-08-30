# 🔐 FortiGate Email Filter Checklist — SMTP, ESMTP, SPF, DKIM, DMARC & Anti-Spam

> **SheynShield | Engineering Secure Networks**
> FortiGate Email Filtering • SMTP • ESMTP • Anti-Spam • SPF • DKIM • DMARC • MIME • DNSBL • ORBL • FortiGuard

[![FortiOS](https://img.shields.io/badge/FortiOS-7.x-red)](#) [![Security](https://img.shields.io/badge/Network-Security-blue)](#) [![NSE](https://img.shields.io/badge/NSE-4%20%7C%207-orange)](#) [![Format](https://img.shields.io/badge/Format-GitHub%20Checklist-black)](#)

> **Practical FortiGate Email Filter checklist for FortiOS 7.x, NSE preparation, troubleshooting, deployment validation, and real-world email security architecture.**

---

## 📌 Table of Contents

* [1. Email Security Architecture](#1-email-security-architecture)
* [2. File Filter vs Email Filter vs DLP](#2-file-filter-vs-email-filter-vs-dlp)
* [3. Email Filter Deployment](#3-email-filter-deployment)
* [4. SMTP & ESMTP Checklist](#4-smtp--esmtp-checklist)
* [5. SMTP Command Flow](#5-smtp-command-flow)
* [6. SPF Checklist](#6-spf-checklist)
* [7. DKIM Checklist](#7-dkim-checklist)
* [8. DMARC Checklist](#8-dmarc-checklist)
* [9. SMTP Envelope vs Message](#9-smtp-envelope-vs-message)
* [10. Local Spam Filters](#10-local-spam-filters)
* [11. HELO DNS Check](#11-helo-dns-check)
* [12. Return Email DNS Check](#12-return-email-dns-check)
* [13. Block & Allow Lists](#13-block--allow-lists)
* [14. Local Override](#14-local-override)
* [15. Banned Word Filtering](#15-banned-word-filtering)
* [16. Trusted IP Addresses](#16-trusted-ip-addresses)
* [17. MIME Header Filtering](#17-mime-header-filtering)
* [18. FortiGuard Anti-Spam](#18-fortiguard-anti-spam)
* [19. DNSBL & ORBL](#19-dnsbl--orbl)
* [20. SMTP Filtering Logic](#20-smtp-filtering-logic)
* [21. IMAP & POP3 Filtering](#21-imap--pop3-filtering)
* [22. Webmail Visibility](#22-webmail-visibility)
* [23. Email Security CLI Checklist](#23-email-security-cli-checklist)
* [24. Troubleshooting Checklist](#24-troubleshooting-checklist)
* [25. Production Security Checklist](#25-production-security-checklist)
* [26. NSE Exam Traps](#26-nse-exam-traps)
* [27. Quick Revision Card](#27-quick-revision-card)

---

# 1. Email Security Architecture

## 🧭 High-Level Flow

```text
                         Internet
                            │
                            ▼
                    ┌───────────────┐
                    │   FortiGate   │
                    └───────┬───────┘
                            │
              ┌─────────────┼─────────────┐
              │             │             │
              ▼             ▼             ▼
            SMTP          IMAP          POP3
              │             │             │
              └─────────────┼─────────────┘
                            ▼
                    Email Filtering
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
        ▼                   ▼                   ▼
   Local Filters        FortiGuard          DNSBL/ORBL
        │               Anti-Spam               │
        │                   │                   │
        └───────────────────┼───────────────────┘
                            ▼
                    Content Inspection
                            │
                 ┌──────────┴──────────┐
                 │                     │
                 ▼                     ▼
            MIME Headers         Banned Words
                 │                     │
                 └──────────┬──────────┘
                            ▼
                     Final Decision
                       /        \
                   ALLOW       SPAM/BLOCK
```

## ✅ Architecture Checklist

* [ ] Identify all protected mail flows.
* [ ] Identify SMTP traffic.
* [ ] Identify IMAP/IMAPS traffic.
* [ ] Identify POP3/POP3S traffic.
* [ ] Identify mail gateways in front of FortiGate.
* [ ] Determine whether FortiGate sees the original sender IP.
* [ ] Determine whether SSL inspection is required.
* [ ] Enable only the required email-filter features.
* [ ] Verify FortiGuard connectivity.
* [ ] Define logging requirements.
* [ ] Define false-positive handling.
* [ ] Define trusted mail infrastructure.

---

# 2. File Filter vs Email Filter vs DLP

## 🔎 Security Function Comparison

| Technology       | Primary Purpose                                       |
| ---------------- | ----------------------------------------------------- |
| **File Filter**  | File type / metadata control                          |
| **Email Filter** | Spam, sender, reputation and email-specific filtering |
| **DLP**          | Sensitive-content and pattern inspection              |

### File Filter

```text
File
 │
 ├── File Type
 ├── Metadata
 └── Archive Content
       │
       ▼
   Allow / Block / Log
```

### Email Filter

```text
Email
 │
 ├── Sender / Recipient
 ├── SMTP Behavior
 ├── DNS Reputation
 ├── FortiGuard
 ├── MIME
 └── Banned Words
       │
       ▼
 Spam / Allow / Block
```

### DLP

```text
Content
 │
 ├── Sensitive Information
 ├── Patterns
 ├── Data Types
 └── Size / Context
       │
       ▼
   DLP Decision
```

## ⚠️ Exam Checklist

* [ ] Do not confuse File Filter with DLP.
* [ ] File Filter primarily controls file characteristics.
* [ ] DLP is designed for sensitive-data inspection.
* [ ] Email Filter is focused on email security controls.

---

# 3. Email Filter Deployment

## ⚙️ Feature Visibility

Depending on FortiOS/platform:

```text
System
   ↓
Feature Visibility
   ↓
Email Filter
```

## ✅ Deployment Checklist

* [ ] Email Filter feature is visible.
* [ ] Required email filtering features are enabled.
* [ ] Email Filter profile exists.
* [ ] Correct firewall policy is identified.
* [ ] Email Filter profile is applied.
* [ ] Protocol options are correct.
* [ ] SSL inspection is configured when required.
* [ ] Logging is enabled.
* [ ] FortiGuard services are reachable.
* [ ] Test messages are available.

---

# 4. SMTP & ESMTP Checklist

## SMTP

Common SMTP commands:

```text
HELO
MAIL FROM
RCPT TO
DATA
RSET
```

## ESMTP

Extended SMTP introduces additional capabilities through extensions.

Common examples:

```text
EHLO
AUTH
STARTTLS
SIZE
HELP
```

## 🧠 NSE Memory

```text
HELO
  ↓
SMTP

EHLO
  ↓
ESMTP
```

## ✅ Checklist

* [ ] Identify whether the client uses HELO or EHLO.
* [ ] Check whether ESMTP extensions are advertised.
* [ ] Check whether AUTH is used.
* [ ] Check whether STARTTLS is offered.
* [ ] Check whether STARTTLS is actually negotiated.
* [ ] Check the MAIL FROM value.
* [ ] Check the RCPT TO value.
* [ ] Check DATA processing.
* [ ] Use RSET to reset a transaction when troubleshooting SMTP sessions.

---

# 5. SMTP Command Flow

```text
Client
  │
  │ HELO / EHLO
  ▼
Server
  │
  │ 250
  ▼
Client
  │
  │ MAIL FROM:<sender>
  ▼
Server
  │
  │ 250
  ▼
Client
  │
  │ RCPT TO:<recipient>
  ▼
Server
  │
  │ 250
  ▼
Client
  │
  │ DATA
  ▼
Server
  │
  │ Message
  │ .
  ▼
Server
  │
  │ 250
  ▼
Transaction Complete
```

## 🔥 SMTP Checklist

* [ ] HELO/EHLO succeeds.
* [ ] MAIL FROM is accepted.
* [ ] RCPT TO is accepted.
* [ ] DATA is accepted.
* [ ] Message terminator `.` is received.
* [ ] Server returns successful response.
* [ ] SMTP transaction can be reset with `RSET`.

---

# 6. SPF Checklist

**SPF = Sender Policy Framework**

SPF determines whether a sending IP is authorized by the sender domain's published SPF policy.

## 🔄 SPF Flow

```text
Incoming Email
      │
      ▼
Envelope Sender Domain
      │
      ▼
DNS SPF Record
      │
      ▼
Sending IP
      │
      ▼
Authorized?
   /       \
 YES       NO
  │         │
PASS       FAIL
```

## Example

```text
v=spf1 ip4:1.2.3.4 include:_spf.example.com -all
```

## Common SPF Mechanisms

```text
ip4:
ip6:
include:
a
mx
all
```

## ✅ SPF Checklist

* [ ] SPF record exists.
* [ ] SPF syntax is valid.
* [ ] Sending IP is authorized.
* [ ] `include:` mechanisms are valid.
* [ ] SPF lookup behavior is understood.
* [ ] SPF failure handling is defined.
* [ ] Envelope sender domain is identified.
* [ ] Do not assume SPF validates the visible `From:` header.

### 🧠 Memory Aid

> **SPF → Is this sending IP authorized for the domain?**

---

# 7. DKIM Checklist

**DKIM = DomainKeys Identified Mail**

DKIM uses cryptographic signatures to allow receiving systems to verify message authenticity and integrity for signed content.

## 🔐 DKIM Flow

```text
Sending Server
      │
      │ Private Key
      ▼
DKIM Signature
      │
      ▼
Email
      │
      ▼
Receiving Server
      │
      │ DNS Query
      ▼
Public Key
      │
      ▼
Signature Verification
```

## DNS Structure

```text
selector._domainkey.example.com
```

## ✅ DKIM Checklist

* [ ] DKIM signing is enabled.
* [ ] Selector is known.
* [ ] Public key exists in DNS.
* [ ] DNS selector is correct.
* [ ] Signature is present.
* [ ] Signature validation succeeds.
* [ ] Message modification behavior is understood.
* [ ] DKIM domain is identified.

### 🧠 Memory Aid

> **DKIM → Was the message cryptographically signed, and can the signature be verified?**

---

# 8. DMARC Checklist

**DMARC = Domain-based Message Authentication, Reporting, and Conformance**

DMARC builds on SPF and DKIM and adds domain policy, alignment requirements, and reporting.

## 🔄 DMARC Flow

```text
Incoming Email
      │
      ├──────────► SPF
      │
      └──────────► DKIM
                     │
                     ▼
                   DMARC
                     │
          ┌──────────┼──────────┐
          │          │          │
          ▼          ▼          ▼
         None    Quarantine   Reject
```

## DMARC Policies

```text
p=none
p=quarantine
p=reject
```

## ✅ DMARC Checklist

* [ ] DMARC record exists.
* [ ] SPF result is understood.
* [ ] DKIM result is understood.
* [ ] SPF alignment is understood.
* [ ] DKIM alignment is understood.
* [ ] DMARC policy is defined.
* [ ] Reporting configuration is reviewed.
* [ ] Third-party senders are considered.
* [ ] Subdomain policy is considered.

### 🧠 Memory Aid

```text
SPF
→ Sending authorization

DKIM
→ Cryptographic signature

DMARC
→ Policy + alignment + reporting
```

---

# 9. SMTP Envelope vs Message

## SMTP Envelope

```text
MAIL FROM:<bob@example.com>
RCPT TO:<alice@example.net>
```

## Message Headers

```text
From: Bob <bob@example.com>
To: Alice <alice@example.net>
Subject: Meeting
```

## Message Body

```text
Hi Alice,

Do you want to meet tonight?

Regards,
Bob
```

## ⭐ Security Concept

```text
SMTP Envelope
    │
    ├── MAIL FROM
    └── RCPT TO
         │
         ▼
      Message
         │
         ├── Headers
         └── Body
```

## ✅ Checklist

* [ ] Identify envelope sender.
* [ ] Identify visible `From:` header.
* [ ] Do not assume they are identical.
* [ ] Understand the impact on SPF.
* [ ] Understand the impact on DKIM.
* [ ] Understand the impact on DMARC.
* [ ] Check spoofing scenarios.

---

# 10. Local Spam Filters

## Six Core Local Checks

```text
1. HELO DNS Check
2. Return Email DNS Check
3. Block / Allow List
4. Banned Words
5. Trusted IP
6. MIME Header
```

## Filtering Matrix

| Filter           | Primary Input         |
| ---------------- | --------------------- |
| HELO DNS         | SMTP client identity  |
| Return Email DNS | Sender domain         |
| Block/Allow      | IP / domain / address |
| Banned Words     | Message content       |
| Trusted IP       | Source IP             |
| MIME Header      | MIME/header fields    |

## ✅ Checklist

* [ ] HELO DNS behavior is understood.
* [ ] Return Email DNS behavior is understood.
* [ ] Block/Allow lists are configured.
* [ ] Banned-word table is reviewed.
* [ ] Trusted IP table is reviewed.
* [ ] MIME header rules are reviewed.

---

# 11. HELO DNS Check

Example:

```text
HELO mail.example.com
          │
          ▼
      DNS Lookup
          │
      ┌───┴───┐
      ▼       ▼
    Valid   Invalid
      │       │
      ▼       ▼
 Continue   Filter
```

## DNS Concepts

```text
FQDN
 ↓
A / AAAA

Reverse DNS
 ↓
PTR
```

## ✅ Checklist

* [ ] Capture the HELO/EHLO hostname.
* [ ] Resolve the hostname.
* [ ] Check A/AAAA where applicable.
* [ ] Check reverse DNS where applicable.
* [ ] Investigate mismatches.
* [ ] Do not confuse HELO DNS with Return Email DNS.

---

# 12. Return Email DNS Check

Example:

```text
MAIL FROM:<bob@example.com>
              │
              ▼
        example.com
              │
              ▼
         DNS Checks
              │
              ▼
       Filtering Decision
```

## ✅ Checklist

* [ ] Identify the envelope sender.
* [ ] Extract sender domain.
* [ ] Validate relevant DNS records.
* [ ] Check sender-domain reputation.
* [ ] Investigate invalid DNS behavior.
* [ ] Distinguish this from HELO validation.

---

# 13. Block & Allow Lists

```text
Incoming Email
      │
      ▼
Local List
   /     \
Allow   Block
 │        │
 ▼        ▼
Pass    Filter
```

## ✅ Checklist

* [ ] Define trusted senders.
* [ ] Define blocked senders.
* [ ] Define trusted IP addresses.
* [ ] Avoid overly broad allow rules.
* [ ] Review exceptions periodically.
* [ ] Document business justification.
* [ ] Monitor false positives.

---

# 14. Local Override

Local override changes the precedence of local block/allow decisions relative to other filtering checks.

## CLI

```cli
config emailfilter profile
    edit "ef-test"
        config smtp
            set local-override enable
        end
    next
end
```

## Concept

### Local Override Disabled

```text
DNS / Reputation
       ↓
Local Lists
       ↓
Other Filters
```

### Local Override Enabled

```text
Local Lists
       ↓
Other Filtering
       ↓
DNS / Reputation
```

## ✅ Checklist

* [ ] Determine whether `local-override` is enabled.
* [ ] Understand which local rules need precedence.
* [ ] Review allow-list behavior.
* [ ] Review block-list behavior.
* [ ] Test both matching and non-matching traffic.
* [ ] Document exceptions.

### 🧠 NSE Memory

> **Local Override Enabled → Local decisions receive earlier precedence.**

---

# 15. Banned Word Filtering

Banned-word filtering evaluates configured words or patterns and associates matching entries with a score/action.

## Logic

```text
Pattern
   +
Score
   +
Threshold
   ↓
Spam Decision
```

## Example

```text
Pattern             Score
-------------------------
word                   10
suspicious*             10
```

## CLI

```cli
config emailfilter bword
    edit 1
        set name "test-bword"

        config entries
            edit 1
                set pattern "word"
                set pattern-type wild-card
                set action spam
                set where all
                set score 10
            next
        end
    next
end
```

## Apply Profile

```cli
config emailfilter profile
    edit "ef-test"
        set options bannedword
        set spam-bword-threshold 20
        set spam-bword-table 1
    next
end
```

## ✅ Checklist

* [ ] Define required patterns.
* [ ] Select the correct pattern type.
* [ ] Define action.
* [ ] Define search location.
* [ ] Configure appropriate score.
* [ ] Configure threshold.
* [ ] Test Subject matching.
* [ ] Test Body matching.
* [ ] Monitor false positives.
* [ ] Validate behavior against the deployed FortiOS release.

---

# 16. Trusted IP Addresses

Trusted IP addresses can exempt known trusted infrastructure from applicable IP-based anti-spam checks.

## Common Architecture

```text
Internet
   │
   ▼
Mail Gateway
   │
   ▼
FortiGate
   │
   ▼
Mail Server
```

FortiGate may see:

```text
Mail Gateway IP
```

rather than:

```text
Original Sender IP
```

## CLI

```cli
config emailfilter iptrust
    edit 1
        set name "ip-tr-test"

        config entries
            edit 1
                set addr-type ipv4
                set ipv4-subnet 192.168.20.200/32
            next
        end
    next
end
```

## Apply

```cli
config emailfilter profile
    edit "ef-test"
        set spam-rbl-table 1
    next
end
```

## ✅ Checklist

* [ ] Identify mail gateways.
* [ ] Identify source IP visible to FortiGate.
* [ ] Add only genuinely trusted infrastructure.
* [ ] Avoid unnecessarily broad subnets.
* [ ] Review trusted IP entries regularly.
* [ ] Understand which IP-based checks are affected.

> ⚠️ Trusted IP does **not** mean all security inspection is bypassed.

---

# 17. MIME Header Filtering

**MIME = Multipurpose Internet Mail Extensions**

MIME describes message content types and multipart structures.

## Common MIME Types

```text
text
 ├── plain
 └── enriched

image
 ├── gif
 └── jpeg

audio
 └── basic

video
 └── mpeg

application
 ├── octet-stream
 └── postscript

message
 ├── rfc822
 ├── partial
 └── external-body

multipart
 ├── mixed
 ├── alternative
 ├── parallel
 └── digest
```

## CLI

```cli
config emailfilter mheader
    edit 1
        set name "mheader-test"

        config entries
            edit 1
                set fieldname "message"
                set fieldbody "test"
                set pattern-type wildcard
                set action spam
            next
        end
    next
end
```

## Apply

```cli
config emailfilter profile
    edit "ef-test"
        set options spamhdrcheck
        set spam-mheader-table 1
    next
end
```

## ✅ Checklist

* [ ] Identify required MIME headers.
* [ ] Define matching fields.
* [ ] Define matching patterns.
* [ ] Define action.
* [ ] Attach MIME table to the email profile.
* [ ] Test multipart messages.
* [ ] Monitor false positives.

---

# 18. FortiGuard Anti-Spam

FortiGuard can provide cloud-based reputation and anti-spam intelligence.

## Common Intelligence Sources

```text
IP Reputation
URL Reputation
Phishing Detection
Email Checksum / Fingerprint
Spam Intelligence
```

## Flow

```text
Email
  │
  ├── Local Filters
  │
  ├── IP Reputation
  │
  ├── URL Reputation
  │
  ├── Phishing Detection
  │
  └── Email Checksum
          │
          ▼
      Final Verdict
```

## Spam Submission

```text
Incorrectly Classified Email
            │
            ▼
       User Submission
            │
            ▼
         FortiGuard
```

## ✅ Checklist

* [ ] Verify FortiGuard connectivity.
* [ ] Verify subscription/service status where applicable.
* [ ] Check IP reputation.
* [ ] Check URL reputation.
* [ ] Check phishing detection.
* [ ] Understand email checksum/fingerprint behavior.
* [ ] Review incorrectly classified messages.
* [ ] Follow organizational privacy requirements before submitting email data.

---

# 19. DNSBL & ORBL

## DNSBL

**DNS-based Block List**

Used to identify IP addresses associated with spam or other undesirable activity.

## ORBL

**Open Relay Block List**

Used to identify systems associated with open-relay behavior.

## Architecture

```text
SMTP Source IP
      │
      ▼
DNSBL / ORBL
      │
   ┌──┴──┐
   ▼     ▼
Listed  Clean
   │      │
   ▼      ▼
 Spam   Continue
```

## CLI

```cli
config emailfilter dnsbl
    edit 1
        set name "dnsbl-test"

        config entries
            edit 1
                set server 192.168.20.200
                set action spam
            next
        end
    next
end
```

## Apply

```cli
config emailfilter profile
    edit "ef-test"
        set options spamrbl
        set spam-rbl-table 1
    next
end
```

## ✅ Checklist

* [ ] Identify DNSBL providers.
* [ ] Validate provider availability.
* [ ] Review provider reputation quality.
* [ ] Understand false-positive risk.
* [ ] Configure appropriate action.
* [ ] Test listed IP behavior.
* [ ] Test clean IP behavior.
* [ ] Review current provider policies before production deployment.

---

# 20. SMTP Filtering Logic

> **Note:** Exact processing behavior depends on FortiOS version, enabled options, protocol, and profile configuration. Treat the following as a conceptual troubleshooting model rather than a universal packet-processing order.

## Local Override Disabled

```text
SMTP Connection
      │
      ▼
HELO / DNS Checks
      │
      ▼
Sender / Reputation Checks
      │
      ▼
FortiGuard
      │
      ▼
Local Lists
      │
      ▼
MIME / Header Checks
      │
      ▼
Banned Words
      │
      ▼
Final Verdict
```

## Local Override Enabled

```text
SMTP Connection
      │
      ▼
Local Lists
      │
      ▼
Other Local Checks
      │
      ▼
DNS / Reputation
      │
      ▼
Content Checks
      │
      ▼
Final Verdict
```

## ✅ Troubleshooting Checklist

* [ ] Determine whether local override is enabled.
* [ ] Identify which filter generated the verdict.
* [ ] Check local lists.
* [ ] Check DNS validation.
* [ ] Check FortiGuard reputation.
* [ ] Check DNSBL/ORBL.
* [ ] Check MIME filtering.
* [ ] Check banned words.
* [ ] Review logs for the final action.

---

# 21. IMAP & POP3 Filtering

## Protocol Roles

```text
SMTP
 ↓
Mail Transfer

IMAP
 ↓
Mailbox Access / Synchronization

POP3
 ↓
Mail Retrieval
```

Secure variants:

```text
IMAPS
POP3S
```

## Conceptual Filtering Flow

```text
Incoming Mail
      │
      ▼
MIME Header
      │
      ▼
Email Address Lists
      │
      ▼
Banned Words
      │
      ▼
IP Lists
      │
      ▼
DNS Validation
      │
      ▼
FortiGuard
      │
      ▼
DNSBL / ORBL
```

## ✅ Checklist

* [ ] Identify IMAP traffic.
* [ ] Identify IMAPS traffic.
* [ ] Identify POP3 traffic.
* [ ] Identify POP3S traffic.
* [ ] Configure appropriate SSL inspection for encrypted protocols.
* [ ] Verify email filtering profile.
* [ ] Verify logging.
* [ ] Test mailbox retrieval.

---

# 22. Webmail Visibility

Supported webmail features depend on FortiOS version and protocol visibility.

Example configuration:

```cli
config emailfilter profile
    edit "ef-test"
        set spam-filtering enable

        config msn-hotmail
            set log-all enable
        end

        config gmail
            set log-all enable
        end
    next
end
```

## Concept

```text
Webmail
   │
   ▼
FortiGate
   │
   ├── Gmail
   └── Hotmail / MSN
          │
          ▼
      Visibility / Logs
```

## ⚠️ Checklist

* [ ] Verify feature availability on the deployed FortiOS version.
* [ ] Verify encrypted traffic visibility.
* [ ] Verify SSL inspection requirements.
* [ ] Do not assume webmail logging equals full email security inspection.
* [ ] Consider dedicated email-security architecture for advanced requirements.

---

# 23. Email Security CLI Checklist

## 🔹 Local Override

```cli
config emailfilter profile
    edit "ef-test"
        config smtp
            set local-override enable
        end
    next
end
```

* [ ] Correct profile selected.
* [ ] Local override requirement documented.
* [ ] Allow/block precedence tested.

## 🔹 Banned Words

```cli
config emailfilter bword
    edit 1
        set name "test-bword"

        config entries
            edit 1
                set pattern "word"
                set pattern-type wild-card
                set action spam
                set where all
                set score 10
            next
        end
    next
end
```

* [ ] Pattern configured.
* [ ] Pattern type configured.
* [ ] Score configured.
* [ ] Action configured.
* [ ] Search location configured.

## 🔹 Trusted IP

```cli
config emailfilter iptrust
    edit 1
        set name "ip-tr-test"

        config entries
            edit 1
                set addr-type ipv4
                set ipv4-subnet 192.168.20.200/32
            next
        end
    next
end
```

* [ ] Only trusted infrastructure included.
* [ ] Subnet scope minimized.
* [ ] IP ownership verified.

## 🔹 DNSBL

```cli
config emailfilter dnsbl
    edit 1
        set name "dnsbl-test"

        config entries
            edit 1
                set server 192.168.20.200
                set action spam
            next
        end
    next
end
```

* [ ] DNSBL source is valid.
* [ ] Action is appropriate.
* [ ] False positives are monitored.

---

# 24. Troubleshooting Checklist

## 🔍 SMTP Connectivity

* [ ] TCP/25 connectivity works where applicable.
* [ ] Submission ports are understood.
* [ ] SMTP server responds.
* [ ] HELO/EHLO succeeds.
* [ ] MAIL FROM succeeds.
* [ ] RCPT TO succeeds.
* [ ] DATA succeeds.
* [ ] STARTTLS negotiation succeeds where required.

## 🔍 Sender Validation

* [ ] Envelope sender identified.
* [ ] Sender domain identified.
* [ ] SPF record checked.
* [ ] SPF result understood.
* [ ] DKIM signature checked.
* [ ] DKIM selector resolved.
* [ ] DMARC result checked.
* [ ] Alignment evaluated.

## 🔍 Reputation

* [ ] Source IP reputation checked.
* [ ] Sender/domain reputation checked.
* [ ] FortiGuard status checked.
* [ ] DNSBL status checked.
* [ ] ORBL status checked.
* [ ] Trusted IP configuration checked.

## 🔍 Content

* [ ] MIME headers checked.
* [ ] Banned-word rules checked.
* [ ] Threshold checked.
* [ ] False positives investigated.
* [ ] Attachment/file filtering checked separately.

## 🔍 Policy

* [ ] Correct firewall policy matched.
* [ ] Email Filter profile attached.
* [ ] Protocol options correct.
* [ ] SSL inspection profile correct.
* [ ] Logging enabled.
* [ ] Final action verified.

---

# 25. Production Security Checklist

## 🛡️ Architecture

* [ ] Mail flow documented.
* [ ] Mail gateways documented.
* [ ] Internal mail servers documented.
* [ ] Public MX infrastructure documented.
* [ ] TLS requirements documented.
* [ ] Logging requirements documented.

## 🔐 Authentication

* [ ] SPF configured.
* [ ] DKIM configured.
* [ ] DMARC configured.
* [ ] Alignment requirements understood.
* [ ] Third-party senders reviewed.

## 🧠 Reputation

* [ ] FortiGuard enabled where appropriate.
* [ ] IP reputation evaluated.
* [ ] URL reputation evaluated.
* [ ] DNSBL providers reviewed.
* [ ] ORBL behavior understood.

## 🎯 Local Controls

* [ ] Allow list documented.
* [ ] Block list documented.
* [ ] Trusted IPs documented.
* [ ] Banned words reviewed.
* [ ] MIME rules reviewed.
* [ ] Local override requirements documented.

## 📊 Monitoring

* [ ] Spam logs enabled.
* [ ] Security events monitored.
* [ ] False positives tracked.
* [ ] False negatives investigated.
* [ ] Trusted IP entries reviewed periodically.
* [ ] DNSBL behavior reviewed periodically.
* [ ] FortiGuard connectivity monitored.

## 🔄 Maintenance

* [ ] FortiOS version documented.
* [ ] Configuration changes documented.
* [ ] Email filtering behavior tested after upgrades.
* [ ] Third-party reputation providers reviewed.
* [ ] Exceptions periodically removed when no longer required.

---

# 26. NSE Exam Traps

## 🧠 Trap #1 — HELO vs EHLO

```text
HELO
→ SMTP

EHLO
→ ESMTP
```

---

## 🧠 Trap #2 — SPF vs DKIM

```text
SPF
→ Sending authorization

DKIM
→ Cryptographic signature
```

---

## 🧠 Trap #3 — DMARC

```text
DMARC
=
SPF/DKIM authentication
+
Alignment
+
Policy
+
Reporting
```

---

## 🧠 Trap #4 — Envelope vs From Header

```text
MAIL FROM:
→ SMTP envelope sender

From:
→ Message header
```

They are **not necessarily identical**.

---

## 🧠 Trap #5 — File Filter vs DLP

```text
File Filter
→ File characteristics

DLP
→ Sensitive content / patterns
```

---

## 🧠 Trap #6 — Local Override

```text
local-override enable
→ Local block/allow decisions receive earlier precedence.
```

---

## 🧠 Trap #7 — Trusted IP

```text
Trusted IP
≠
Disable all security inspection
```

It relates primarily to applicable IP-based email filtering checks.

---

## 🧠 Trap #8 — DNSBL

```text
DNSBL
→ DNS-based reputation/block-list mechanism
```

DNSBL is **not SPF**.

---

## 🧠 Trap #9 — FortiGuard vs DNSBL

```text
FortiGuard
→ Fortinet security intelligence

DNSBL / ORBL
→ External reputation/list sources
```

---

## 🧠 Trap #10 — FortiGate vs FortiMail

```text
FortiGate Email Filter
→ Integrated firewall email-filtering capability

FortiMail
→ Dedicated email-security platform
```

Architecture should be selected according to scale, requirements, and required email-security capabilities.

---

## 🧠 Trap #11 — IMAP vs POP3

```text
IMAP
→ Mailbox access / synchronization

POP3
→ Mail retrieval
```

---

## 🧠 Trap #12 — SSL Inspection

```text
IMAPS / POP3S / encrypted mail traffic
→ May require appropriate SSL inspection/content-scanning capability
```

---

# 27. Quick Revision Card

```text
EMAIL FILTER
│
├── SMTP / ESMTP
│   ├── HELO
│   ├── EHLO
│   ├── MAIL FROM
│   ├── RCPT TO
│   ├── DATA
│   └── RSET
│
├── LOCAL FILTERS
│   ├── HELO DNS
│   ├── Return Email DNS
│   ├── Block / Allow
│   ├── Banned Words
│   ├── Trusted IP
│   └── MIME Header
│
├── FORTIGUARD
│   ├── IP Reputation
│   ├── URL Reputation
│   ├── Phishing Detection
│   └── Email Checksum
│
├── THIRD PARTY
│   ├── DNSBL
│   └── ORBL
│
├── AUTHENTICATION
│   ├── SPF
│   ├── DKIM
│   └── DMARC
│
└── PROTOCOLS
    ├── SMTP
    ├── ESMTP
    ├── IMAP
    ├── IMAPS
    ├── POP3
    └── POP3S
```

---

# ⚡ SMTP Authentication Memory Map

```text
SPF
│
└── "Is this sending IP authorized?"

DKIM
│
└── "Can the cryptographic signature be verified?"

DMARC
│
└── "Does authentication/alignment satisfy
    the domain's published policy?"
```

---

# ⚡ Email Filtering Decision Flow

```text
                    Incoming Email
                          │
                          ▼
                  SMTP / IMAP / POP3
                          │
                          ▼
                   Protocol Parsing
                          │
             ┌────────────┴────────────┐
             │                         │
             ▼                         ▼
       Local Filtering             Reputation
             │                         │
       ┌─────┼─────┐          ┌───────┼───────┐
       │     │     │          │       │       │
       ▼     ▼     ▼          ▼       ▼       ▼
     Lists  MIME  Words    FortiGuard DNSBL  ORBL
       │     │     │          │       │       │
       └─────┴─────┴──────────┴───────┴───────┘
                          │
                          ▼
                    Final Decision
                     /          \
                    ▼            ▼
                 ALLOW       SPAM / BLOCK
```

---

# 🔥 One-Line NSE Memory Aid

> **SMTP transfers email, SPF validates sending authorization, DKIM validates a cryptographic signature, DMARC applies authentication/alignment policy, FortiGuard provides security intelligence, DNSBL/ORBL provide reputation-list checks, and local filters provide administrator-controlled email decisions.**

---

# 🛡️ Production Design Principle

```text
                  EMAIL SECURITY
                        │
        ┌───────────────┼───────────────┐
        │               │               │
        ▼               ▼               ▼
 Authentication     Reputation       Content
        │               │               │
   SPF / DKIM        FortiGuard     Banned Words
      DMARC           DNSBL             MIME
                      ORBL              DLP
        │               │               │
        └───────────────┼───────────────┘
                        ▼
                 Policy Enforcement
                        │
                        ▼
                  Logging / SOC
                        │
                        ▼
                  Final Verdict
```

### Layered Security Model

```text
Authentication
      +
Reputation
      +
Content Inspection
      +
Policy Enforcement
      +
Logging
      =
Layered Email Security
```

---

# 🔎 Keywords

```text
FortiOS
FortiGate Email Filter
FortiGate SMTP
FortiGate ESMTP
SMTP Commands
HELO
EHLO
MAIL FROM
RCPT TO
DATA
RSET
SPF
DKIM
DMARC
Email Authentication
Email Security
Anti-Spam
FortiGuard Anti-Spam
IP Reputation
URL Reputation
DNSBL
ORBL
Open Relay Block List
Banned Words
MIME Header
Trusted IP
Local Override
IMAP
IMAPS
POP3
POP3S
SSL Inspection
FortiMail
NSE4
NSE7
Network Security
Email Filtering
```

---

# 📋 Final FortiGate Email Security Audit

* [ ] FortiOS version documented.
* [ ] Email Filter feature enabled where required.
* [ ] Correct firewall policy identified.
* [ ] Correct Email Filter profile attached.
* [ ] SMTP flow tested.
* [ ] ESMTP behavior verified.
* [ ] HELO/EHLO behavior verified.
* [ ] MAIL FROM verified.
* [ ] RCPT TO verified.
* [ ] SPF validated.
* [ ] DKIM validated.
* [ ] DMARC validated.
* [ ] Envelope/header differences understood.
* [ ] Local block/allow lists reviewed.
* [ ] Local override reviewed.
* [ ] Banned-word rules reviewed.
* [ ] Trusted IPs reviewed.
* [ ] MIME rules reviewed.
* [ ] FortiGuard connectivity verified.
* [ ] IP reputation tested.
* [ ] URL reputation tested.
* [ ] DNSBL/ORBL configuration reviewed.
* [ ] IMAP/POP3 filtering tested where required.
* [ ] SSL inspection validated for encrypted protocols.
* [ ] Logging verified.
* [ ] False positives monitored.
* [ ] Exceptions documented.
* [ ] Production configuration backed up.
* [ ] Post-upgrade validation plan prepared.

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

> **Remember:** Modern email security should be designed as a layered architecture. SPF, DKIM, DMARC, reputation intelligence, DNSBL/ORBL, local filtering, content inspection, SSL inspection, logging, and policy enforcement solve different parts of the email-security problem.

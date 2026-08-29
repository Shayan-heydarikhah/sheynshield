# FortiGate Email Filter & SMTP Commands  

> **FortiGate Email Filtering | SMTP | ESMTP | Anti-Spam | SPF | DKIM | DMARC | MIME | DNSBL | ORBL | Banned Words | FortiGuard**
>
> Practical reference for **FortiOS 7.x**, NSE preparation, troubleshooting, and real-world email security deployments.

---

## 📚 Table of Contents

* [1. Email Filter Architecture](#1-email-filter-architecture)
* [2. File Filter vs Email Filter](#2-file-filter-vs-email-filter)
* [3. File Filter](#3-file-filter)
* [4. Email Filter](#4-email-filter)
* [5. SMTP vs ESMTP](#5-smtp-vs-esmtp)
* [6. SMTP Command Flow](#6-smtp-command-flow)
* [7. SPF](#7-spf)
* [8. DKIM](#8-dkim)
* [9. DMARC](#9-dmarc)
* [10. SMTP Envelope vs Message](#10-smtp-envelope-vs-message)
* [11. Local Spam Filters](#11-local-spam-filters)
* [12. HELO DNS Check](#12-helo-dns-check)
* [13. Return Email DNS Check](#13-return-email-dns-check)
* [14. Block/Allow Lists](#14-blockallow-lists)
* [15. Local Override](#15-local-override)
* [16. Banned Word Filtering](#16-banned-word-filtering)
* [17. Trusted IP Addresses](#17-trusted-ip-addresses)
* [18. MIME Header Filtering](#18-mime-header-filtering)
* [19. FortiGuard Anti-Spam](#19-fortiguard-anti-spam)
* [20. Third-Party DNSBL / ORBL](#20-third-party-dnsbl--orbl)
* [21. SMTP Filtering Order](#21-smtp-filtering-order)
* [22. IMAP / POP3 Filtering](#22-imap--pop3-filtering)
* [23. Webmail Filtering](#23-webmail-filtering)
* [24. Email Filter Deployment Checklist](#24-email-filter-deployment-checklist)
* [25. NSE Exam Traps](#25-nse-exam-traps)
* [26. Quick Revision Card](#26-quick-revision-card)

---

# 1. Email Filter Architecture

FortiGate email filtering can inspect email protocols and apply multiple layers of local, FortiGuard, and third-party anti-spam checks.

```text
                         Internet
                            |
                            v
                    +---------------+
                    |   FortiGate   |
                    +---------------+
                            |
                  +---------+---------+
                  |                   |
                  v                   v
             SMTP/ESMTP          IMAP/POP3
                  |                   |
                  +---------+---------+
                            |
                            v
                    Email Filtering
                            |
       +--------------------+--------------------+
       |                    |                    |
       v                    v                    v
  Local Filters       FortiGuard          Third-Party
       |               Anti-Spam             DNSBL
       |                    |                    |
       +--------------------+--------------------+
                            |
                            v
                    Final Email Decision
                            |
                 +----------+----------+
                 |                     |
               ALLOW                 SPAM/BLOCK
```

### Main Filtering Sources

| Source                   | Purpose                                          |
| ------------------------ | ------------------------------------------------ |
| **Local Filters**        | Locally configured email filtering rules         |
| **FortiGuard Anti-Spam** | FortiGuard reputation and anti-spam intelligence |
| **DNSBL**                | DNS-based reputation/block lists                 |
| **ORBL**                 | Open-relay reputation/block lists                |
| **Banned Words**         | Detect configured words or phrases               |
| **MIME Header**          | Inspect MIME-related headers                     |
| **Trusted IP**           | Exclude trusted source IPs from IP-based checks  |

---

# 2. File Filter vs Email Filter

These are related but fundamentally different security functions.

```text
File Filter
    |
    +--> File Type
    +--> File Metadata
    +--> Archive Content
    |
    v
Block / Log

Email Filter
    |
    +--> Sender / Recipient
    +--> SMTP behavior
    +--> DNS reputation
    +--> FortiGuard Anti-Spam
    +--> Banned Words
    +--> MIME headers
    |
    v
Spam / Allow / Block
```

### ⭐ Important

**File Filter does not inspect file content to identify sensitive information such as:**

* Social Security Numbers
* Credit card numbers
* Sensitive strings
* Regex-based sensitive data

For content/size-based data protection, use **DLP**.

---

# 3. File Filter

File Filter can control files passing through supported protocols based primarily on **file type and metadata**.

### Example Profile

```bash
config file-filter profile
    edit "ff-test"
        set feature-set flow
        set log enable
        set scan-archive-contents enable

        config rules
            edit "ff-r-test"
                set protocol http ftp smtp imap pop3 cifs
                set action block
                set direction outgoing
                set password-protected any
                set file-type tar zip
            next

            edit "ff-r2-test"
                set protocol http ftp smtp imap pop3 cifs
                set action log-only
                set direction any
                set password-protected any
                set file-type 7z
            next
        end
    next
end
```

### Apply to Firewall Policy

```bash
config firewall policy
    edit 1
        set srcintf "port3"
        set dstintf "port1"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"

        set utm-status enable
        set profile-protocol-options "protocol"
        set ssl-ssh-profile "protocols"
        set file-filter-profile "ff-test"

        set auto-asic-offload disable
        set np-acceleration disable

        set nat enable
    next
end
```

### ⭐ File Filter Limitation

```text
File Filter
    ↓
File type / metadata
```

Not:

```text
File Filter
    ↓
Sensitive content inspection
```

For content-based inspection:

```text
DLP
    ↓
Content / Pattern / Size
```

---

# 4. Email Filter

FortiGate Email Filter provides anti-spam filtering for supported email protocols.

For larger or more specialized email-security deployments, **FortiMail** is generally the dedicated email-security platform.

### Feature Visibility

Depending on FortiOS/platform:

```text
System
 └── Feature Visibility
      └── Email Filter
```

### Email Filter Architecture

```text
Incoming Email
      |
      v
+-------------------+
| SMTP / IMAP / POP |
+-------------------+
      |
      v
Local Checks
      |
      +----> Block/Allow List
      |
      +----> DNS Checks
      |
      +----> Banned Words
      |
      +----> MIME Headers
      |
      v
FortiGuard Anti-Spam
      |
      v
Third-Party Reputation
      |
      v
Final Decision
```

---

# 5. SMTP vs ESMTP

## SMTP

SMTP provides the basic protocol commands required to transfer email.

Common commands include:

| Command     | Purpose                        |
| ----------- | ------------------------------ |
| `HELO`      | Identify the SMTP client       |
| `MAIL FROM` | Specify envelope sender        |
| `RCPT TO`   | Specify envelope recipient     |
| `DATA`      | Begin message transmission     |
| `RSET`      | Reset current mail transaction |

---

## ESMTP

**Extended SMTP (ESMTP)** adds extensions to traditional SMTP.

Common extensions include:

```text
EHLO
AUTH
STARTTLS
SIZE
HELP
```

### Example

```text
Client
  |
  | EHLO mail.example.com
  |
  | AUTH ...
  |
  | STARTTLS
  |
  | MAIL FROM
  |
  | RCPT TO
  |
  | DATA
  |
  v
Server
```

### ⭐ Exam Point

```text
HELO → SMTP
EHLO → ESMTP
```

---

# 6. SMTP Command Flow

A simplified SMTP transaction:

```text
Client
  |
  | HELO / EHLO
  |-------------------->
  |
  |<-------------------- 250
  |
  | MAIL FROM:
  |-------------------->
  |
  |<-------------------- 250
  |
  | RCPT TO:
  |-------------------->
  |
  |<-------------------- 250
  |
  | DATA
  |-------------------->
  |
  | Message
  |-------------------->
  |
  | .
  |-------------------->
  |
  |<-------------------- 250
```

### Reset Transaction

```text
RSET
```

resets the current SMTP mail transaction.

---

# 7. SPF

**SPF = Sender Policy Framework**

SPF helps receiving mail systems determine whether a sending IP is authorized to send mail for a domain.

### SPF DNS Record

Example:

```text
v=spf1 ip4:1.2.3.4 include:_spf.example.com -all
```

### Concept

```text
Email arrives
     |
     v
Envelope Sender Domain
     |
     v
SPF DNS Record
     |
     v
Authorized Sending IP?
     |
 +---+---+
 |       |
YES      NO
 |       |
Pass   Fail
```

### SPF Common Mechanisms

```text
ip4:
ip6:
include:
a
mx
all
```

### ⭐ Important

SPF primarily validates the **sending IP against the domain's published SPF policy**.

It does not by itself prove that the visible `From:` header is trustworthy.

---

# 8. DKIM

**DKIM = DomainKeys Identified Mail**

DKIM uses public-key cryptography to allow receiving mail systems to verify that a message was signed by the sending domain and that signed content has not been improperly modified.

### Architecture

```text
Sending Server
     |
     | Private Key
     v
DKIM Signature
     |
     v
Email
     |
     v
Receiving Server
     |
     | DNS Query
     v
DKIM Public Key
     |
     v
Signature Verification
```

### DNS

The public key is published in DNS under a selector.

Conceptually:

```text
selector._domainkey.example.com
```

### Key Concept

```text
Private Key
    ↓
Sign

Public Key
    ↓
Verify
```

---

# 9. DMARC

**DMARC = Domain-based Message Authentication, Reporting, and Conformance**

DMARC builds on SPF and DKIM and allows a domain owner to publish a policy for messages that fail authentication/alignment requirements.

### Basic Flow

```text
Incoming Email
      |
      +----> SPF
      |
      +----> DKIM
      |
      v
    DMARC
      |
      +----> Pass
      |
      +----> Quarantine
      |
      +----> Reject
```

### DMARC Policies

```text
p=none
p=quarantine
p=reject
```

### DMARC Reporting

DMARC can also provide reporting mechanisms that help domain administrators understand authentication results.

### ⭐ Exam Point

```text
SPF
→ Authorized sending source

DKIM
→ Cryptographic message signature

DMARC
→ Policy + alignment + reporting
```

---

# 10. SMTP Envelope vs Message

This distinction is critical for email filtering.

## SMTP Envelope

The SMTP envelope contains transaction-level sender/recipient information.

```text
HELO mail.example.com

MAIL FROM:<bob@example.com>

RCPT TO:<alice@example.net>
```

---

## Message Headers

The message itself can contain:

```text
From: Bob <bob@example.com>
To: Alice <alice@example.net>
Subject: Meeting
```

---

## Message Body

```text
Hi Alice,

Do you want to meet tonight?

Regards,
Bob
```

### Complete Structure

```text
SMTP Envelope
    |
    +--> MAIL FROM
    +--> RCPT TO
    |
    v
Message
    |
    +--> Headers
    |
    +--> Body
```

### ⭐ Security Point

The SMTP envelope sender and the visible `From:` header are **not necessarily the same**.

This distinction is important for:

* SPF
* DKIM
* DMARC
* Anti-spam analysis
* Spoofing detection

---

# 11. Local Spam Filters

FortiGate can perform several local anti-spam checks.

### Six Important Categories

```text
1. HELO DNS Lookup
2. Return Email DNS Check
3. Block/Allow List
4. Banned Words
5. Trusted IP Addresses
6. MIME Header
```

### Classification

| Filter           | Primary Input               |
| ---------------- | --------------------------- |
| HELO DNS         | Sending host identity / DNS |
| Return Email DNS | Sender domain DNS           |
| Block/Allow List | IP/domain/email             |
| Banned Words     | Message content             |
| Trusted IP       | Source IP                   |
| MIME Header      | MIME/header fields          |

---

# 12. HELO DNS Check

The HELO/EHLO identity supplied by the SMTP client can be checked against DNS information.

Conceptually:

```text
HELO mail.example.com
          |
          v
DNS Lookup
          |
     +----+----+
     |         |
   Valid     Invalid
     |         |
   Continue   Spam/Block
```

Typical DNS relationships can include:

```text
FQDN
  ↓
A / AAAA

Reverse DNS
  ↓
PTR
```

### ⭐ Exam Trap

Do not confuse:

```text
HELO DNS check
```

with:

```text
Return Email DNS check
```

They evaluate different parts of the SMTP transaction.

---

# 13. Return Email DNS Check

The sender domain can be checked against DNS information.

Conceptually:

```text
MAIL FROM:<bob@example.com>
              |
              v
       example.com
              |
              v
        DNS validation
              |
              v
         Reputation
```

Relevant DNS records may include:

```text
A / AAAA
MX
PTR
```

depending on the specific validation mechanism.

---

# 14. Block/Allow Lists

Local block/allow lists provide administrator-defined decisions.

```text
Incoming Email
      |
      v
Local List
      |
 +----+----+
 |         |
Allow     Block
 |         |
Pass      Spam/Block
```

### ⭐ High-Priority Use Case

If an administrator wants a trusted sender or destination to bypass additional filtering, the local override behavior becomes important.

---

# 15. Local Override

By default, certain DNS/reputation checks can occur before local block/allow list processing.

Local override changes the precedence so local block/allow decisions can be evaluated earlier.

### Configure

```bash
config emailfilter profile
    edit "ef-test"
        config smtp
            set local-override enable
        end
    next
end
```

### Concept

#### Local Override Disabled

```text
DNS / Reputation Checks
        ↓
Local Block/Allow
        ↓
Other Filters
```

#### Local Override Enabled

```text
Local Block/Allow
        ↓
Other Local Checks
        ↓
DNS / Reputation Checks
```

### ⭐ Exam Point

Use local override when a local allow/block decision needs precedence over subsequent filtering checks.

---

# 16. Banned Word Filtering

Banned-word filtering evaluates configured words or phrases.

The matching model uses:

```text
Pattern
   +
Score
   +
Threshold
   ↓
Spam Decision
```

### Default Concept

```text
Score = 10
Threshold = 10
```

> Exact defaults and supported syntax can vary by FortiOS release; verify the deployed version before relying on them operationally.

### Example

```text
Term             Score
----------------------
word              20
word phrase       20
mail*age          20
```

### Important Behavior

A matching term's score is generally counted **once per configured term**, rather than increasing indefinitely simply because the same term appears repeatedly.

### Configure

```bash
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

### Apply to Profile

```bash
config emailfilter profile
    edit "ef-test"
        set options bannedword
        set spam-bword-threshold 20
        set spam-bword-table 1
    next
end
```

### Search Locations

```text
Subject
Body
```

Depending on the configured `where` behavior.

---

# 17. Trusted IP Addresses

Trusted IP addresses allow administrators to exempt known trusted sources from certain IP-based anti-spam checks.

### Typical Use Case

If FortiGate is behind an internal mail-transfer infrastructure:

```text
Internet
   |
   v
Mail Gateway
   |
   v
FortiGate
   |
   v
Internal Mail Server
```

The FortiGate may see the mail gateway's IP rather than the original external sender.

Trusted IP configuration can therefore prevent unnecessary IP reputation checks against trusted infrastructure.

### Configure

```bash
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

### Apply

```bash
config emailfilter profile
    edit "ef-test"
        set spam-rbl-table 1
    next
end
```

### Trusted IP Concept

Trusted sources can be exempt from checks such as:

```text
DNSBL / RBL
FortiGuard IP reputation
Local IP block lists
```

> ⚠️ Do not trust broad IP ranges unless they are genuinely controlled and trusted.

---

# 18. MIME Header Filtering

**MIME = Multipurpose Internet Mail Extensions**

MIME allows email messages to contain different content types and multipart structures.

### Common MIME Categories

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

### Configure MIME Header Filter

```bash
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

### Apply

```bash
config emailfilter profile
    edit "ef-test"
        set options spamhdrcheck
        set spam-mheader-table 1
    next
end
```

---

# 19. FortiGuard Anti-Spam

FortiGuard provides cloud-based email reputation and anti-spam intelligence.

### Common FortiGuard-Based Checks

```text
IP Reputation
URL Reputation
Phishing URL Detection
Email Checksum / Fingerprint
Spam Intelligence
```

### Concept

```text
Email
  |
  +----> Local Filters
  |
  +----> FortiGuard IP Reputation
  |
  +----> FortiGuard URL Reputation
  |
  +----> Phishing Detection
  |
  +----> Email Checksum
  |
  v
Final Verdict
```

---

## Spam Submission

Spam submission can help improve FortiGuard anti-spam intelligence when legitimate email is incorrectly classified as spam.

Conceptually:

```text
Email incorrectly marked as Spam
             |
             v
       User Submission
             |
             v
       FortiGuard
```

> Use this feature according to the organization's privacy and operational requirements.

---

# 20. Third-Party DNSBL / ORBL

FortiGate can integrate with external reputation services.

### ORBL

**Open Relay Block List**

Used to identify systems associated with open-relay behavior.

### DNSBL

**DNS-based Block List**

Used to identify IP addresses associated with spam or other undesirable activity.

Examples of public reputation services include:

```text
Spamhaus
DNSBL.info
MultiRBL
```

> ⚠️ Third-party list availability, policies, and licensing can change. Verify the service's current terms before deployment.

### Configure DNSBL

```bash
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

### Apply

```bash
config emailfilter profile
    edit "ef-test"
        set options spamrbl
        set spam-rbl-table 1
    next
end
```

---

# 21. SMTP Filtering Order

The exact processing behavior depends on the enabled options and profile configuration, but the key distinction is whether **local override** is enabled.

## Local Override Disabled

A simplified model:

```text
1. HELO DNS Lookup
        ↓
2. Last-Hop IP / Reputation Checks
        ↓
3. Return Email DNS Check
        ↓
4. FortiGuard Reputation Checks
        ↓
5. Local Block/Allow Lists
        ↓
6. Header / MIME Checks
        ↓
7. Banned Words
```

Conceptually:

```text
SMTP Connection
      |
      v
HELO DNS
      |
      v
Return Email DNS
      |
      v
IP Reputation
      |
      v
Local Lists
      |
      v
MIME Headers
      |
      v
Banned Words
      |
      v
Final Verdict
```

---

## Local Override Enabled

The local block/allow decision gets earlier precedence.

```text
SMTP Connection
      |
      v
Local IP / Address Lists
      |
      v
Header / MIME Lists
      |
      v
Banned Words
      |
      v
DNS / Reputation Checks
      |
      v
Final Verdict
```

### ⭐ NSE Memory Aid

```text
local-override disable
→ reputation/DNS checks can occur before local lists

local-override enable
→ local lists get precedence
```

---

# 22. IMAP / POP3 Filtering

For IMAP/POP3-based spam filtering, the inspection sequence differs from SMTP.

A simplified order is:

```text
1. MIME Header Check
        ↓
2. Email Address Block/Allow List
        ↓
3. Banned Word → Subject
        ↓
4. IP Block/Allow List
        ↓
5. Banned Word → Body
        ↓
6. Return Email DNS
        ↓
7. FortiGuard Email Checksum
        ↓
8. FortiGuard URL Check
        ↓
9. DNSBL / ORBL
```

### Supported Secure Protocols

```text
IMAPS
POP3S
```

require appropriate SSL content scanning/inspection support.

### ⭐ Important

```text
SMTP
→ Mail transfer

IMAP
→ Mail access / mailbox synchronization

POP3
→ Mail retrieval
```

---

# 23. Webmail Filtering

FortiGate can provide limited visibility/logging for supported webmail services.

Example:

```bash
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

### Concept

```text
Webmail
   |
   v
FortiGate
   |
   +--> Gmail
   |
   +--> Hotmail / MSN
   |
   v
Logging / Visibility
```

> ⚠️ Webmail filtering is not equivalent to full dedicated mail-server inspection. For advanced email-security requirements, a dedicated email-security platform may be more appropriate.

---

# 24. Email Filter Deployment Checklist

## Firewall Policy

* [ ] Email filtering is enabled.
* [ ] Correct inspection mode is configured.
* [ ] Required SSL inspection is configured for encrypted protocols.
* [ ] Email Filter profile is attached to the policy.
* [ ] Protocol options are appropriate.
* [ ] Logging is enabled where required.

## SMTP

* [ ] HELO/EHLO behavior is understood.
* [ ] Return email DNS checks are evaluated.
* [ ] Local block/allow lists are configured.
* [ ] Local override behavior is understood.
* [ ] FortiGuard Anti-Spam connectivity is verified.
* [ ] DNSBL/ORBL sources are reviewed.
* [ ] Trusted mail gateways are configured appropriately.

## SPF

* [ ] Sender domain has a valid SPF record.
* [ ] Sending IP is authorized.
* [ ] `include:` mechanisms are validated.
* [ ] SPF failure handling is understood.

## DKIM

* [ ] DKIM signing is enabled on the sender.
* [ ] Public key exists in DNS.
* [ ] Selector is correct.
* [ ] Signature validation works.

## DMARC

* [ ] DMARC record exists.
* [ ] SPF/DKIM alignment is understood.
* [ ] Policy is appropriate.
* [ ] Reporting configuration is reviewed.

## Banned Words

* [ ] Patterns are defined.
* [ ] Scores are appropriate.
* [ ] Threshold is tested.
* [ ] Subject/body matching is understood.
* [ ] False positives are monitored.

## MIME

* [ ] MIME header filtering is configured where required.
* [ ] Multipart messages are tested.
* [ ] Legitimate attachments are not unintentionally blocked.

## Trusted IP

* [ ] Internal mail gateways are identified.
* [ ] Only trusted IP ranges are exempted.
* [ ] Trusted sources are regularly reviewed.

---

# 25. NSE Exam Traps

## 🧠 Trap #1 — SMTP vs ESMTP

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
→ validates sending authorization based on IP/domain policy

DKIM
→ validates a cryptographic signature
```

---

## 🧠 Trap #3 — DMARC

DMARC is not simply another DNS reputation mechanism.

```text
SPF
+
DKIM
+
Alignment
+
Policy
+
Reporting
=
DMARC
```

---

## 🧠 Trap #4 — Envelope vs Header

```text
MAIL FROM
→ SMTP envelope sender

From:
→ Message header
```

They are not necessarily identical.

---

## 🧠 Trap #5 — File Filter vs DLP

```text
File Filter
→ File type / metadata

DLP
→ Content / sensitive information / patterns / size
```

---

## 🧠 Trap #6 — Local Override

```text
local-override enable
→ local block/allow decisions receive precedence
```

Useful when trusted or explicitly blocked sources must be handled before other filtering checks.

---

## 🧠 Trap #7 — Trusted IP

Trusted IP does **not** mean:

```text
All security inspection = bypass
```

It is primarily related to exemption from applicable **IP-based anti-spam checks**.

---

## 🧠 Trap #8 — DNSBL

```text
DNSBL
→ reputation based on DNS lookups
```

It is not the same as SPF.

---

## 🧠 Trap #9 — FortiMail

```text
FortiGate Email Filter
→ integrated/basic email filtering capabilities

FortiMail
→ dedicated email security platform
```

For large-scale enterprise mail security, architecture and feature requirements should determine the product choice.

---

## 🧠 Trap #10 — IMAPS / POP3S

Encrypted mail protocols require appropriate SSL inspection/content-scanning support.

```text
IMAP
→ IMAPS

POP3
→ POP3S
```

---

# 26. Quick Revision Card

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
├── EMAIL AUTHENTICATION
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

## ⚡ SMTP Authentication Memory Map

```text
SPF
│
└── "Is this sending IP authorized?"

DKIM
│
└── "Was this message cryptographically signed?"

DMARC
│
└── "What should I do when SPF/DKIM authentication
    and alignment do not satisfy the domain policy?"
```

---

## ⚡ Email Filtering Decision Flow

```text
                 Incoming Email
                       |
                       v
                SMTP / IMAP / POP
                       |
                       v
                Protocol Parsing
                       |
          +------------+------------+
          |                         |
          v                         v
     Local Filters            Reputation
          |                         |
          |                 +-------+-------+
          |                 |       |       |
          |                FG      DNSBL   ORBL
          |                 |       |       |
          +-----------------+-------+-------+
                            |
                            v
                      Content Checks
                            |
                  +---------+---------+
                  |                   |
                  v                   v
             MIME Header         Banned Words
                  |                   |
                  +---------+---------+
                            |
                            v
                      Final Decision
                            |
                  +---------+---------+
                  |                   |
                ALLOW               SPAM/BLOCK
```

---

## 🔥 One-Line NSE Memory Aid

> **SMTP transfers the message, SPF validates sending authorization, DKIM validates a cryptographic signature, DMARC applies domain policy and alignment, FortiGuard provides reputation intelligence, DNSBL/ORBL provide external reputation data, and local filters provide administrator-controlled email decisions.**

---

## 🛡️ Production Design Principle

```text
                EMAIL SECURITY
                      |
        +-------------+-------------+
        |             |             |
        v             v             v
 Authentication   Reputation    Content
        |             |             |
     SPF/DKIM       FortiGuard    Banned Words
     DMARC          DNSBL         MIME
                                  DLP
        |             |             |
        +-------------+-------------+
                      |
                      v
                Final Verdict
```

### Best-Practice Mindset

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

> **Remember:** No single mechanism is sufficient for modern email security. SPF, DKIM, DMARC, reputation services, local filtering, content controls, and appropriate SSL inspection should be designed as complementary layers.

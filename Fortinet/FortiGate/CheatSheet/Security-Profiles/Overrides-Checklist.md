# 🔐 FortiGate SSL Inspection & Web Rating Override — Engineering Checklist

> **SheynShield | Engineering Secure Networks**
>
> **FortiOS SSL Inspection • HTTPS • HTTP/2 • ALPN • SNI • Certificate Replacement • FortiGuard Web Rating • IP Reputation • Web Rating Override • Temporary Web Access**

![FortiGate SSL Inspection](https://img.shields.io/badge/FortiGate-SSL%20Inspection-red?style=flat-square)
![FortiOS](https://img.shields.io/badge/FortiOS-Security%20Profiles-blue?style=flat-square)
![NSE4](https://img.shields.io/badge/NSE4-Ready-success?style=flat-square)
![NSE7](https://img.shields.io/badge/NSE7-Engineering-orange?style=flat-square)

---

## 📌 Table of Contents

* [1. SSL Inspection Architecture](#1-ssl-inspection-architecture)
* [2. SSL Inspection Pre-Deployment Checklist](#2-ssl-inspection-pre-deployment-checklist)
* [3. ALPN & HTTP/2 Checklist](#3-alpn--http2-checklist)
* [4. SNI Identification Checklist](#4-sni-identification-checklist)
* [5. Certificate Replacement Checklist](#5-certificate-replacement-checklist)
* [6. Multiple Certificate Selection](#6-multiple-certificate-selection)
* [7. SSL IP Rating Exemption](#7-ssl-ip-rating-exemption)
* [8. HTTP Address IP Rating](#8-http-address-ip-rating)
* [9. Domain vs IP Reputation](#9-domain-vs-ip-reputation)
* [10. Web Rating Override](#10-web-rating-override)
* [11. Alternate Web Filter Profiles](#11-alternate-web-filter-profiles)
* [12. External Resources & Local Categories](#12-external-resources--local-categories)
* [13. Web Override Scope](#13-web-override-scope)
* [14. Authentication & Temporary Access](#14-authentication--temporary-access)
* [15. Firewall Policy Deployment](#15-firewall-policy-deployment)
* [16. HTTPS Inspection Decision Tree](#16-https-inspection-decision-tree)
* [17. SSL Inspection Troubleshooting](#17-ssl-inspection-troubleshooting)
* [18. Web Rating Troubleshooting](#18-web-rating-troubleshooting)
* [19. Production Hardening Checklist](#19-production-hardening-checklist)
* [20. NSE Exam Checklist](#20-nse-exam-checklist)
* [21. One-Minute Revision](#21-one-minute-revision)
* [22. Quick CLI Reference](#22-quick-cli-reference)
* [23. Final Engineering Checklist](#23-final-engineering-checklist)
* [SheynShield Resources](#sheynshield-resources)

---

# 1. SSL Inspection Architecture

## 🧠 Core Mental Model

FortiGate SSL inspection is not simply "decrypt HTTPS."

The inspection workflow can involve:

```text
Client
  │
  │ TCP
  ▼
FortiGate
  │
  ├── TLS Detection
  │
  ├── SNI
  │
  ├── Certificate Processing
  │
  ├── ALPN
  │
  ├── SSL Inspection
  │
  ├── Application Identification
  │
  ├── FortiGuard Rating
  │
  ├── Web Filtering
  │
  └── Security Decision
  │
  ▼
Internet Server
```

### ✅ Architecture Checklist

* [ ] Identify whether traffic is HTTP or HTTPS.
* [ ] Identify whether SSL inspection is required.
* [ ] Determine whether certificate inspection is sufficient.
* [ ] Determine whether deep/custom inspection is required.
* [ ] Verify the firewall policy uses the intended SSL/SSH profile.
* [ ] Verify the required Web Filter profile is attached.
* [ ] Verify FortiGuard connectivity.
* [ ] Verify client trust of the inspection CA where certificate replacement is used.
* [ ] Verify application visibility requirements.
* [ ] Verify performance impact before production deployment.

---

# 2. SSL Inspection Pre-Deployment Checklist

## 🔍 Visibility Requirements

Before enabling deep inspection:

* [ ] Identify applications that require encrypted-payload inspection.
* [ ] Identify applications that must remain exempt.
* [ ] Identify certificate-pinning applications.
* [ ] Identify applications incompatible with interception.
* [ ] Identify internal/private applications.
* [ ] Identify regulatory or privacy exclusions.
* [ ] Identify managed endpoints.
* [ ] Verify the FortiGate inspection CA can be deployed to clients.
* [ ] Test browsers and operating systems.
* [ ] Test mobile devices where applicable.

## 🛡️ Security Profile Checklist

* [ ] SSL/SSH Inspection profile selected.
* [ ] Web Filter profile selected.
* [ ] Antivirus profile selected where required.
* [ ] Application Control profile selected where required.
* [ ] IPS profile selected where required.
* [ ] DNS Security controls evaluated.
* [ ] Logging enabled.
* [ ] Certificate errors monitored.
* [ ] FortiGuard service availability verified.

---

# 3. ALPN & HTTP/2 Checklist

## 🧠 ALPN

**ALPN = Application-Layer Protocol Negotiation**

ALPN allows TLS peers to negotiate the application protocol carried over the TLS connection.

Common examples:

```text
h2
http/1.1
```

### TLS Flow

```text
Client
   │
   │ TCP
   ▼
FortiGate
   │
   │ TLS
   ▼
ClientHello
   │
   ├── TLS version
   ├── SNI
   └── ALPN
        ├── h2
        └── http/1.1
```

### ⚠️ Important

ALPN is **not encryption**.

It is a protocol-negotiation mechanism carried as part of TLS negotiation.

### CLI

```cli
config firewall ssl-ssh-profile
    edit <profile>
        set supported-alpn all
    next
end
```

### ✅ ALPN Checklist

* [ ] Identify whether the application uses HTTP/1.1.
* [ ] Identify whether the application uses HTTP/2.
* [ ] Verify ALPN support in the SSL inspection profile.
* [ ] Verify `supported-alpn` behavior for the target FortiOS release.
* [ ] Confirm `h2` negotiation where required.
* [ ] Confirm HTTP/1.1 fallback behavior where applicable.
* [ ] Test browsers after changing ALPN behavior.
* [ ] Monitor applications that depend on HTTP/2.

### 🎓 NSE Rule

```text
TCP
 ↓
TLS
 ↓
ALPN negotiation
 ↓
HTTP/1.1 / HTTP/2
```

> **ALPN ≠ HTTP/2.**
> ALPN is the negotiation mechanism; HTTP/2 is one of the protocols that can be negotiated.

---

# 4. SNI Identification Checklist

## 🔎 Server Name Indication

SNI allows a TLS client to indicate the hostname it is attempting to reach.

Example:

```text
ClientHello
    │
    └── SNI = www.example.com
```

FortiGate can use hostname-related information during SSL inspection and certificate handling.

### ✅ SNI Checklist

* [ ] Verify the client sends SNI.
* [ ] Verify the expected hostname is visible.
* [ ] Verify the hostname matches the intended certificate.
* [ ] Check CN/SAN values.
* [ ] Check certificate replacement behavior.
* [ ] Consider clients that do not send SNI.
* [ ] Do not assume TCP/443 alone identifies a website.

### Mental Model

```text
TCP/443
   ↓
TLS
   ↓
SNI
   ↓
Hostname Identification
```

---

# 5. Certificate Replacement Checklist

## 🎯 Replace Mode

Certificate replacement allows FortiGate to present a replacement certificate to the client.

Conceptually:

```text
Original Server
      │
      │ Certificate
      ▼
  FortiGate
      │
      │ Replacement Certificate
      ▼
    Client
```

### Configuration

```cli
config firewall ssl-ssh-profile
    edit multi-cert
        set server-cert-mode replace
        set server-cert bbb aaa
    next
end
```

### ✅ Certificate Replacement Checklist

* [ ] `server-cert-mode` is configured correctly.
* [ ] Required certificates exist on FortiGate.
* [ ] Certificate names are correct.
* [ ] Certificate CN is correct.
* [ ] Certificate SAN is correct.
* [ ] SNI matching has been tested.
* [ ] Client trusts the FortiGate CA.
* [ ] Browser certificate warnings have been tested.
* [ ] Non-browser applications have been tested.
* [ ] Certificate-pinning applications have been tested.
* [ ] Fallback behavior has been validated.

### ⚠️ Key Principle

Certificate replacement is fundamentally different from simply observing the original server certificate.

```text
Certificate Inspection
        ↓
Inspect certificate metadata

Certificate Replacement
        ↓
FortiGate presents a replacement certificate
```

---

# 6. Multiple Certificate Selection

Multiple HTTPS services can share infrastructure or protected addresses.

Example:

```text
                 ┌── site-a.example.com
                 │
Client ──────────┼── site-b.example.com
                 │
                 └── site-c.example.com
```

FortiGate can use a certificate list and select an appropriate certificate based on available server identity information.

### Selection Concept

```text
ClientHello
    │
    ▼
   SNI
    │
    ▼
FortiGate
    │
    ├── Certificate List
    │
    ├── Match CN/SAN
    │
    └── Select Certificate
```

### ✅ Checklist

* [ ] Multiple certificates are imported.
* [ ] Certificate names are correct.
* [ ] Certificates contain appropriate SAN values.
* [ ] SNI values match expected hostnames.
* [ ] The certificate list is attached to the SSL profile.
* [ ] Matching has been tested with multiple sites.
* [ ] No-match behavior has been validated.
* [ ] Client trust has been validated.

---

# 7. SSL IP Rating Exemption

FortiGate may consider IP reputation as part of security/rating decisions.

Configuration example:

```cli
config firewall ssl-ssh-profile
    edit <profile>
        set ssl-exemption-ip-rating enable
    next
end
```

### Concept

```text
example.com
     │
     ├── Domain Rating
     │
     └── IP Rating
```

Depending on the configuration, IP rating can participate in the decision.

### ✅ Checklist

* [ ] Determine whether IP reputation should influence SSL rating.
* [ ] Check domain reputation.
* [ ] Check resolved IP reputation.
* [ ] Identify shared-hosting scenarios.
* [ ] Identify CDN usage.
* [ ] Identify cloud-hosting infrastructure.
* [ ] Determine whether IP rating creates false positives.
* [ ] Configure SSL IP-rating exemption only when justified.
* [ ] Test the resulting Web Filter behavior.

---

# 8. HTTP Address IP Rating

HTTP protocol options can include address/IP rating behavior.

Example:

```cli
config firewall profile-protocol-options
    edit <profile>
        config http
            set address-ip-rating enable
        end
    next
end
```

### ✅ Checklist

* [ ] Identify the relevant protocol-options profile.
* [ ] Check HTTP rating behavior.
* [ ] Check domain classification.
* [ ] Check destination IP reputation.
* [ ] Evaluate shared hosting.
* [ ] Evaluate CDN environments.
* [ ] Test domain-based access.
* [ ] Test direct-IP access.
* [ ] Confirm Web Filter logs.

---

# 9. Domain vs IP Reputation

## 🧠 Why They Can Conflict

A single IP address may host many domains:

```text
203.0.113.10
 ├── company-a.com
 ├── company-b.com
 ├── company-c.com
 └── example.com
```

The domain can be legitimate while the IP has a poor reputation.

### Example

```text
example.com
     ↓
Business Category

203.0.113.10
     ↓
Malicious / Suspicious
```

This creates a policy-design question.

### ✅ Reputation Checklist

* [ ] Check domain reputation.
* [ ] Check destination IP reputation.
* [ ] Determine whether the IP hosts multiple domains.
* [ ] Check CDN involvement.
* [ ] Check cloud hosting.
* [ ] Check DNS resolution.
* [ ] Determine which reputation source should have operational priority.
* [ ] Test the final security decision.

### Mental Model

```text
Hostname
   │
   ├── Domain Intelligence
   │
   └── IP Intelligence
          │
          ▼
     Security Decision
```

> **Do not assume that a domain and its destination IP always have the same reputation.**

---

# 10. Web Rating Override

## 🎯 Purpose

A Web Rating Override manually changes the classification applied to a website.

Typical use cases:

* [ ] Incorrect FortiGuard category.
* [ ] Business application incorrectly classified.
* [ ] Internal policy requires custom categorization.
* [ ] Temporary organizational classification.
* [ ] Trusted business resource requires a different policy treatment.

### Concept

```text
Website
   ↓
FortiGuard Rating
   ↓
Web Rating Override
   ↓
Local / Alternate Classification
   ↓
Web Filter Policy
   ↓
Allow / Block
```

### ✅ Web Rating Override Checklist

* [ ] Identify the incorrectly categorized URL/domain.
* [ ] Verify FortiGuard's current classification.
* [ ] Define the required local category.
* [ ] Create the appropriate override.
* [ ] Confirm the override scope.
* [ ] Confirm the Web Filter profile uses the resulting category.
* [ ] Test from an affected client.
* [ ] Verify Web Filter logs.
* [ ] Document the business justification.
* [ ] Review overrides periodically.

---

# 11. Alternate Web Filter Profiles

Selected users or source addresses can require different web-access policies.

Concept:

```text
Normal Users
     │
     ▼
Web Filter A
     │
     └── Social Media → Block


Authorized Users
     │
     ▼
Web Filter B
     │
     └── Social Media → Allow
```

### Possible use cases

* [ ] Administrators.
* [ ] Developers.
* [ ] Security team.
* [ ] Temporary troubleshooting.
* [ ] Business-required access.
* [ ] Controlled testing.

### ✅ Checklist

* [ ] Identify users requiring an alternate profile.
* [ ] Define the business reason.
* [ ] Prefer identity-based scope where appropriate.
* [ ] Avoid unnecessarily broad source-IP exceptions.
* [ ] Assign the alternate Web Filter profile.
* [ ] Test normal users.
* [ ] Test authorized users.
* [ ] Verify logs.
* [ ] Review exceptions periodically.

---

# 12. External Resources & Local Categories

FortiGate can consume externally maintained resources where supported.

Example:

```cli
config system external-resource
    edit 1
        set category 192
        set resource http://192.168.20.200/lists/blocklist.txt
    next
end
```

### Architecture

```text
External List
     │
     ▼
FortiGate
     │
     ▼
External Resource
     │
     ▼
Category / Security Decision
     │
     ▼
Web Filtering
```

### ✅ Checklist

* [ ] Identify the external list owner.
* [ ] Validate list format.
* [ ] Validate list update frequency.
* [ ] Confirm FortiGate can retrieve the resource.
* [ ] Confirm the configured category.
* [ ] Test resource synchronization.
* [ ] Monitor stale-list conditions.
* [ ] Document ownership.
* [ ] Define rollback behavior.

---

# 13. Web Override Scope

## 👤 User vs User Group

Scope is critical.

Example:

```text
local_user
 ├── user1
 └── user2
```

A group-level override may affect all users in the group.

### Risk

```text
User 1
   ↓
Override
   ↓
User Group
   ↓
User 2 may also receive the effect
```

### ✅ Scope Checklist

* [ ] Determine whether the exception is for one user.
* [ ] Determine whether the exception is for a group.
* [ ] Determine whether IP-based scope is appropriate.
* [ ] Verify authenticated identity.
* [ ] Verify group membership.
* [ ] Confirm the override does not affect unintended users.
* [ ] Test with another member of the same group.
* [ ] Document the scope.

### Golden Rule

> **The narrower the required exception, the narrower the override scope should be.**

---

# 14. Authentication & Temporary Access

Temporary web access can be safer than permanent policy exceptions.

Concept:

```text
Blocked Website
      │
      ▼
   Block Page
      │
      ▼
Override Request
      │
      ▼
Authentication
      │
      ├── Username
      ├── Password
      └── Profile
      │
      ▼
Temporary Access
      │
      ▼
Timer Expires
      │
      ▼
Normal Policy
```

### ⏱️ Temporary Access Checklist

* [ ] Authentication is required.
* [ ] Correct user identity is validated.
* [ ] Appropriate Web Filter profile is selected.
* [ ] Override duration is defined.
* [ ] Temporary access is logged.
* [ ] Expiration behavior is tested.
* [ ] Permanent exceptions are avoided where possible.
* [ ] Business justification is recorded.

### Example

```text
Override Duration
     ↓
15 minutes
     ↓
Temporary Access
     ↓
Timer Expiration
     ↓
Original Policy
```

---

# 15. Firewall Policy Deployment

Creating an SSL inspection profile is not sufficient.

The profile must participate in the traffic policy.

### Concept

```text
SSL Inspection Profile
        │
        ▼
Firewall Policy
        │
        ▼
Client Traffic
```

### Example

```cli
config firewall policy
    edit 1
        set srcintf port3
        set dstintf port1
        set srcaddr all
        set dstaddr all
        set action accept
        set schedule always
        set service ALL
        set utm-status enable
        set ssl-ssh-profile multi-cert
        set av-profile default
        set webfilter-profile default
        set logtraffic all
        set nat enable
    next
end
```

### ✅ Policy Checklist

* [ ] Correct source interface.
* [ ] Correct destination interface.
* [ ] Correct source address.
* [ ] Correct destination address.
* [ ] Correct service.
* [ ] Policy is enabled.
* [ ] `utm-status` is enabled where required.
* [ ] Correct SSL/SSH inspection profile.
* [ ] Correct Web Filter profile.
* [ ] Correct security profiles.
* [ ] Logging enabled.
* [ ] Policy order verified.

---

# 16. HTTPS Inspection Decision Tree

Use this decision process before choosing an SSL inspection strategy:

```text
HTTPS Traffic
     │
     ▼
Do we need certificate metadata only?
     │
   YES ─────────► Certificate Inspection
     │
    NO
     │
     ▼
Do we need encrypted payload visibility?
     │
   YES
     │
     ▼
Deep / Custom Inspection
     │
     ▼
Does application support interception?
     │
   YES ─────────► Inspect
     │
    NO
     │
     ▼
Consider SSL exemption
```

### ✅ Decision Checklist

* [ ] Certificate metadata requirement identified.
* [ ] Payload inspection requirement identified.
* [ ] Application compatibility tested.
* [ ] Certificate trust deployed.
* [ ] SSL exceptions documented.
* [ ] Business requirements reviewed.
* [ ] Privacy requirements reviewed.
* [ ] Performance impact evaluated.

---

# 17. SSL Inspection Troubleshooting

## 🔎 Connectivity

* [ ] Is the firewall policy matching the traffic?
* [ ] Is the policy enabled?
* [ ] Is the SSL inspection profile attached?
* [ ] Is TLS negotiation successful?
* [ ] Is SNI present?
* [ ] Is the server certificate valid?
* [ ] Does the client trust the inspection CA?
* [ ] Is certificate pinning involved?
* [ ] Is the application compatible with inspection?

## 🔎 HTTP/2

* [ ] Is ALPN being negotiated?
* [ ] Is `h2` available?
* [ ] Is HTTP/1.1 fallback working?
* [ ] Does changing ALPN behavior change the application result?
* [ ] Is the FortiOS version behavior verified?

## 🔎 Certificate Replacement

* [ ] `server-cert-mode` checked.
* [ ] Certificate list checked.
* [ ] CN checked.
* [ ] SAN checked.
* [ ] SNI checked.
* [ ] CA trust checked.
* [ ] Certificate chain checked.
* [ ] Client warnings checked.

---

# 18. Web Rating Troubleshooting

When a website is unexpectedly blocked or allowed:

### Step 1 — Identify the hostname

* [ ] Record requested FQDN.
* [ ] Verify DNS resolution.
* [ ] Identify destination IP.

### Step 2 — Check reputation

* [ ] Domain category checked.
* [ ] IP reputation checked.
* [ ] Shared hosting identified.
* [ ] CDN identified.

### Step 3 — Check Web Filter

* [ ] Correct Web Filter profile.
* [ ] Category action.
* [ ] Override status.
* [ ] Local category.
* [ ] User/group scope.

### Step 4 — Check policy

* [ ] Correct firewall policy.
* [ ] Correct source identity.
* [ ] Correct Web Filter profile.
* [ ] Correct SSL inspection profile.

### Step 5 — Verify logs

* [ ] Web Filter log.
* [ ] Security event log.
* [ ] Policy log.
* [ ] SSL inspection-related information.

---

# 19. Production Hardening Checklist

## 🔐 Security

* [ ] Use the least permissive Web Filter policy required.
* [ ] Avoid permanent exceptions when temporary access is sufficient.
* [ ] Restrict override scope.
* [ ] Review user-group permissions.
* [ ] Review external resource sources.
* [ ] Document Web Rating Overrides.
* [ ] Review exceptions periodically.

## ⚡ Performance

* [ ] Evaluate SSL inspection CPU usage.
* [ ] Evaluate memory utilization.
* [ ] Monitor concurrent sessions.
* [ ] Identify high-bandwidth encrypted applications.
* [ ] Avoid unnecessary deep inspection.
* [ ] Use exemptions for justified incompatible applications.
* [ ] Monitor FortiGate resource usage after deployment.

## 📊 Monitoring

* [ ] Enable appropriate logging.
* [ ] Monitor Web Filter events.
* [ ] Monitor certificate failures.
* [ ] Monitor SSL inspection failures.
* [ ] Monitor FortiGuard connectivity.
* [ ] Monitor unexpected category changes.
* [ ] Review temporary override events.

---

# 20. NSE Exam Checklist

## 🧠 Memorize These

* [ ] **ALPN** = Application-Layer Protocol Negotiation.
* [ ] ALPN can negotiate `h2` and `http/1.1`.
* [ ] **SNI** identifies the requested hostname during TLS negotiation.
* [ ] Certificate replacement causes FortiGate to present a replacement certificate.
* [ ] Clients must trust the CA used for certificate replacement.
* [ ] Multiple certificates can be configured for replacement scenarios.
* [ ] Domain reputation and IP reputation can differ.
* [ ] Shared hosting can create domain/IP reputation differences.
* [ ] Web Rating Override changes website categorization.
* [ ] Alternate Web Filter profiles can provide differentiated policy.
* [ ] User-group scope can affect multiple users.
* [ ] Temporary overrides reduce the need for permanent exceptions.
* [ ] External resources can provide externally maintained lists where supported.

---

# 21. One-Minute Revision

```text
ALPN
 ↓
Protocol negotiation

SNI
 ↓
Hostname identification

Certificate Inspection
 ↓
Certificate metadata

Certificate Replacement
 ↓
FortiGate presents replacement certificate

Deep Inspection
 ↓
Encrypted payload visibility

FortiGuard
 ↓
Domain/IP reputation

Web Rating Override
 ↓
Change classification

Web Filter
 ↓
Allow / Block

User Override
 ↓
Identity-based exception

Timer
 ↓
Temporary access
```

---

# 22. Quick CLI Reference

## ALPN

```cli
config firewall ssl-ssh-profile
    edit <profile>
        set supported-alpn all
    next
end
```

## Multiple Certificates

```cli
config firewall ssl-ssh-profile
    edit multi-cert
        set server-cert-mode replace
        set server-cert bbb aaa
    next
end
```

## SSL IP Rating Exemption

```cli
config firewall ssl-ssh-profile
    edit <profile>
        set ssl-exemption-ip-rating enable
    next
end
```

## HTTP Address IP Rating

```cli
config firewall profile-protocol-options
    edit <profile>
        config http
            set address-ip-rating enable
        end
    next
end
```

## External Resource

```cli
config system external-resource
    edit 1
        set category 192
        set resource http://192.168.20.200/lists/blocklist.txt
    next
end
```

> ⚠️ **CLI syntax and feature availability can vary by FortiOS release and FortiGate model. Validate with the target FortiOS CLI help before production deployment.**

---

# 23. Final Engineering Checklist

## 🔐 SSL Inspection

* [ ] Correct SSL inspection mode selected.
* [ ] Required traffic visibility identified.
* [ ] SSL exemptions documented.
* [ ] Inspection CA trusted by clients.
* [ ] Application compatibility tested.
* [ ] Performance tested.

## 🌐 ALPN / HTTP/2

* [ ] ALPN requirement identified.
* [ ] `supported-alpn` verified.
* [ ] HTTP/2 tested.
* [ ] HTTP/1.1 fallback tested.

## 🪪 Certificates

* [ ] Required certificates imported.
* [ ] CN/SAN values validated.
* [ ] SNI matching tested.
* [ ] Multiple certificate selection tested.
* [ ] Client trust verified.

## 🛡️ FortiGuard Rating

* [ ] Domain rating checked.
* [ ] IP rating checked.
* [ ] Reputation conflict evaluated.
* [ ] Shared-hosting scenario evaluated.
* [ ] FortiGuard connectivity verified.

## 🎯 Web Rating Override

* [ ] Incorrect category identified.
* [ ] Override created.
* [ ] Local category defined.
* [ ] Web Filter profile verified.
* [ ] Override scope verified.
* [ ] Logs validated.

## 👤 Temporary Access

* [ ] Authentication configured.
* [ ] User/group scope verified.
* [ ] Duration configured.
* [ ] Temporary access tested.
* [ ] Expiration behavior verified.

## 📊 Operations

* [ ] Logging enabled.
* [ ] Exceptions documented.
* [ ] Performance monitored.
* [ ] Overrides reviewed periodically.
* [ ] FortiOS version-specific behavior verified.

---

# 🎯 Final Engineering Principle

> **Do not treat HTTPS as simply TCP/443.**

A reliable FortiGate SSL/Web Filtering design follows the complete chain:

```text
TCP
 ↓
TLS
 ↓
SNI
 ↓
Certificate
 ↓
ALPN
 ↓
Application Protocol
 ↓
SSL Inspection
 ↓
FortiGuard Intelligence
 ↓
Domain/IP Rating
 ↓
Web Rating Override
 ↓
Web Filter
 ↓
Security Decision
```

### The Golden Rule

```text
Correct Visibility
      +
Correct Identity
      +
Correct Inspection
      +
Correct Reputation
      +
Correct Scope
      +
Correct Policy
      ↓
Predictable Web Security
```

> **FortiGate SSL Inspection mindset:**
>
> **Identify → Inspect → Classify → Override when justified → Enforce → Log → Review**

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
FortiOS
FortiGate
SSL Inspection
Deep Inspection
Certificate Inspection
Certificate Replacement
HTTPS Inspection
HTTP/2
ALPN
SNI
TLS
FortiGuard
Web Filtering
Web Rating
Web Rating Override
IP Reputation
Domain Reputation
Security Profiles
Firewall Policy
Network Security
NSE4
NSE7
Temporary Web Access
```

---

> **SheynShield | Engineering Secure Networks**
>
> Practical FortiGate knowledge for **Network Security Engineers, Fortinet Administrators, NSE4 candidates, NSE7 candidates, and security professionals.**

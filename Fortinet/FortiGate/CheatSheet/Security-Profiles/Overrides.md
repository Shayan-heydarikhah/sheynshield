# 🔐 FortiGate SSL Inspection & Web Rating Overrides 

> **SheynShield | Engineering Secure Networks**
> FortiGate Security Profiles • SSL Inspection • ALPN • SNI • Certificate Replacement • FortiGuard Rating • Web Rating Override

---

## 📌 Table of Contents

* [1. SSL Inspection — HTTP/2 & ALPN](#1-ssl-inspection--http2--alpn)
* [2. Multiple Certificates in Replace Mode](#2-multiple-certificates-in-replace-mode)
* [3. FortiGuard IP Rating Exemption](#3-fortiguard-ip-rating-exemption)
* [4. HTTP Address IP Rating](#4-http-address-ip-rating)
* [5. Domain vs IP Reputation](#5-domain-vs-ip-reputation)
* [6. Web Rating Overrides](#6-web-rating-overrides)
* [7. Alternate Web Filter Profiles](#7-alternate-web-filter-profiles)
* [8. External Resources & Local Categories](#8-external-resources--local-categories)
* [9. Web Override Scope](#9-web-override-scope)
* [10. Web Override Authentication & Timers](#10-web-override-authentication--timers)
* [11. Quick Decision Matrix](#11-quick-decision-matrix)
* [12. Troubleshooting Checklist](#12-troubleshooting-checklist)

---

# 1. SSL Inspection — HTTP/2 & ALPN

## 🧠 ALPN

**ALPN = Application-Layer Protocol Negotiation**

ALPN is a TLS extension that allows the client and server to negotiate which application-layer protocol will be used after the TLS session is established.

Typical examples:

```text
HTTP/1.1
HTTP/2
```

### TLS Flow

```text
Client
  │
  │ TCP SYN
  ▼
FortiGate
  │
  │ TCP connection
  ▼
Server
  │
  │ TLS negotiation
  ▼
ClientHello
  │
  ├── TLS version
  ├── SNI
  └── ALPN
       ├── h2
       └── http/1.1
```

### Important Concept

ALPN does **not** itself provide encryption.

It is a negotiation mechanism inside the TLS handshake that tells the peers which application protocol should run over the TLS connection.

---

## ⚙️ Configure Supported ALPN

```cli
config firewall ssl-ssh-profile
    edit <profile>
        set supported-alpn all
    next
end
```

### `supported-alpn`

Controls which ALPN protocols the SSL inspection profile supports.

Conceptually:

```text
all
 ├── HTTP/1.1
 └── HTTP/2
```

If ALPN support is restricted, only the supported protocol(s) can be negotiated.

### 💡 Exam Point

> **TCP connection → TLS handshake → ALPN negotiation → HTTP protocol selection**

Do not confuse:

```text
TCP
TLS
ALPN
HTTP/1.1
HTTP/2
```

They belong to different layers/stages of the connection.

---

# 2. Multiple Certificates in Replace Mode

## 🎯 Use Case

Multiple HTTPS sites can share the **same protected server IP address**.

The FortiGate can use multiple certificates in an SSL inspection profile and select the appropriate replacement certificate based on the requested server name.

Example:

```text
                ┌── site-a.example.com
Client ────────►│
                ├── site-b.example.com
       Same IP ─┤
                └── site-c.example.com
```

The FortiGate examines information such as:

* SNI
* Certificate CN
* Certificate SAN

and attempts to find a matching certificate.

---

## 🔄 Certificate Selection Logic

```text
ClientHello
     │
     │ SNI
     ▼
 FortiGate
     │
     ├── Compare SNI
     │
     ├── Compare certificate CN/SAN
     │
     ▼
Certificate List
     │
     ├── Match found
     │      ↓
     │   Use matched certificate
     │
     └── No match
            ↓
       Use first certificate
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

---

## 🔥 Policy Example

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

### ⚠️ Important

Certificate replacement is different from merely inspecting the server certificate.

In replacement mode:

```text
Original Server Certificate
          │
          ▼
      FortiGate
          │
          │ Select replacement certificate
          ▼
Client receives FortiGate-presented certificate
```

The client must trust the CA used by FortiGate to avoid certificate warnings.

---

# 3. FortiGuard IP Rating Exemption

FortiGate can use FortiGuard reputation/rating information for IP addresses during security inspection.

In some environments, you may want the decision to rely on the **domain rating rather than IP reputation**.

### Configuration

```cli
config firewall ssl-ssh-profile
    edit ssh-prof-test
        set ssl-exemption-ip-rating enable
    next
end
```

### Concept

```text
Client requests:
        │
        ▼
example.com
        │
        ├── Domain Rating
        │
        └── IP Rating
```

If IP rating is excluded from the relevant SSL inspection decision:

```text
Domain Rating
      │
      ▼
Security Decision
```

instead of allowing an IP reputation result to override/conflict with the domain-based result.

---

# 4. HTTP Address IP Rating

A similar concept exists for HTTP protocol inspection.

```cli
config firewall profile-protocol-options
    edit proto-test
        config http
            set address-ip-rating enable
        end
    next
end
```

### Concept

FortiGate may evaluate both:

```text
Domain reputation
+
IP reputation
```

This can become important when multiple domains resolve to the same hosting infrastructure.

---

# 5. Domain vs IP Reputation

## 🧠 Why Can Domain and IP Ratings Conflict?

Consider:

```text
https://example.com
       │
       ▼
DNS Resolution
       │
       ▼
203.0.113.10
```

FortiGuard may have separate intelligence for:

```text
example.com → Business
203.0.113.10 → Malware
```

Now two intelligence sources provide different classifications.

### Scenario A — Shared Hosting

```text
203.0.113.10
 ├── company-a.com
 ├── company-b.com
 ├── company-c.com
 └── example.com
```

The IP reputation may not accurately represent every domain hosted on that IP.

### Scenario B — IP-only Access

```text
Client
  │
  ▼
https://203.0.113.10
```

There is no meaningful domain name to evaluate.

In this situation, IP reputation can become more important.

---

## 💡 Design Principle

When domain identity is more trustworthy for your use case:

```text
Domain
  ↓
FortiGuard Category
  ↓
Web Policy
```

When IP reputation is important:

```text
Domain
  +
IP Reputation
  ↓
Security Decision
```

### Practical Example

If a business wants to trust:

```text
*.adobe.com
```

but one Adobe-related IP is classified differently, disabling IP-based rating can prevent an IP reputation conflict from overriding the domain-based classification.

> **Always validate the actual FortiOS version and inspection profile behavior before applying this globally.**

---

# 6. Web Rating Overrides

## 🎯 Purpose

A **Web Rating Override** allows an administrator to manually assign a website to a different Fortinet category.

This is useful when:

* The FortiGuard category is incorrect for your environment.
* A business application is classified incorrectly.
* An internal policy requires a custom classification.
* A site needs to belong to a locally defined category.

Conceptually:

```text
Website
   │
   ▼
FortiGuard Rating
   │
   ▼
Override
   │
   ▼
Local / Alternate Category
   │
   ▼
Web Filter Policy
```

---

## Example

FortiGuard:

```text
example.com
    ↓
Unknown / Incorrect Category
```

Administrator:

```text
example.com
    ↓
Local Category: Business-Trusted
```

The Web Filter can then apply policy based on the overridden classification.

---

# 7. Alternate Web Filter Profiles

Another override mechanism allows selected users or IP addresses to use an **alternative Web Filter profile**.

Example:

```text
Normal users
    │
    ▼
Web Filter Profile A
    │
    └── Social Media → Block
```

Selected users:

```text
Authorized user
    │
    ▼
Web Filter Profile B
    │
    └── Social Media → Allow
```

### Use Cases

Useful for:

* Administrators
* Developers
* Security teams
* Temporary exceptions
* Business-required access

---

# 8. External Resources & Local Categories

FortiGate can use external resources as part of security policy design.

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
FortiGate External Resource
      │
      ▼
Category / Security Decision
      │
      ▼
Web Filtering
```

This can be useful for maintaining externally managed lists without manually entering every item into the FortiGate configuration.

---

# 9. Web Override Scope

## ⚠️ User vs User Group Scope

The scope of a Web Rating Override matters.

Suppose:

```text
local_user
 ├── user1
 └── user2
```

If the override is created at the **user-group scope**, the resulting behavior can affect the group rather than only the individual user.

Conceptually:

```text
User 1
  │
  │ Override
  ▼
local_user group
  │
  ├── User 1
  └── User 2
```

Therefore, User 2 may also inherit the override behavior when accessing the same resource.

### 💡 Important

> If the exception is intended for **one specific person**, carefully select the scope. A group-level override can have a much wider impact than expected.

---

# 10. Web Override Authentication & Timers

Web override mechanisms can use authentication and predefined time periods.

Typical authentication fields include:

```text
Username
Password
Web Filter Profile
Duration
```

Example workflow:

```text
User requests blocked website
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
     Override Granted
          │
          ▼
     Timer Starts
```

---

## ⏱️ Predefined Timer

Example:

```text
15 minutes
```

The user receives temporary access instead of creating a permanent policy exception.

### Recommended Use Cases

Temporary override is useful for:

* Troubleshooting
* Emergency business access
* Temporary research
* Administrator testing
* Short-lived application requirements

---

## 🔐 Authentication Scope

When using **user/user-group based override**, the relevant user group must be available to the firewall policy/authentication design.

A useful mental model:

```text
IP-based override
        │
        └── Device / source IP

User-based override
        │
        └── Authenticated identity

User-group override
        │
        └── Group-level scope
```

---

# 11. Quick Decision Matrix

| Requirement                                       | Recommended Mechanism           |
| ------------------------------------------------- | ------------------------------- |
| Inspect certificate validity                      | Certificate Inspection          |
| Inspect encrypted payload                         | Deep / Custom SSL Inspection    |
| Support HTTP/2 negotiation                        | ALPN / `supported-alpn`         |
| Present different certificates for multiple sites | Replace mode + certificate list |
| Ignore IP reputation during SSL rating            | SSL IP-rating exemption         |
| Control HTTP IP reputation                        | HTTP `address-ip-rating`        |
| Correct an incorrect FortiGuard category          | Web Rating Override             |
| Give selected users different web access          | Alternate Web Filter Profile    |
| Temporarily allow blocked access                  | Web Override + Timer            |
| Maintain external IP/domain lists                 | External Resource               |
| Apply exception to one person                     | User scope                      |
| Apply exception to a group                        | User-group scope                |

---

# 12. Troubleshooting Checklist

## 🔎 SSL / HTTP/2

* [ ] Is the correct SSL inspection profile attached to the firewall policy?
* [ ] Is the traffic actually HTTPS?
* [ ] Is TLS negotiation successful?
* [ ] Is SNI visible?
* [ ] Is ALPN being negotiated?
* [ ] Is HTTP/2 supported by the inspection profile?
* [ ] Is the client trusting the FortiGate inspection CA?
* [ ] Are certificate replacement rules matching the expected SNI/CN/SAN?

---

## 🔎 Certificate Replacement

* [ ] `server-cert-mode` is correctly configured.
* [ ] Required certificates exist on FortiGate.
* [ ] Certificate names are correct.
* [ ] SNI matches the intended certificate.
* [ ] CN/SAN values are correct.
* [ ] The client trusts the issuing CA.

---

## 🔎 FortiGuard Rating

* [ ] Check domain reputation.
* [ ] Check IP reputation.
* [ ] Determine whether the domain and IP classifications conflict.
* [ ] Check whether IP rating should participate in the decision.
* [ ] Validate FortiGuard connectivity.
* [ ] Check the Web Filter profile and category action.

---

## 🔎 Web Override

* [ ] Verify the Web Rating Override exists.
* [ ] Verify the correct category is assigned.
* [ ] Verify the correct Web Filter profile is being used.
* [ ] Check whether the override is user-, IP-, or group-scoped.
* [ ] Verify authentication requirements.
* [ ] Check the override duration/timer.
* [ ] Confirm the firewall policy uses the intended Web Filter profile.

---

# 🧠 NSE Exam & Real-World Takeaways

> ### 1️⃣ ALPN ≠ HTTP/2
>
> ALPN is the **negotiation mechanism** used to select protocols such as HTTP/1.1 or HTTP/2.

> ### 2️⃣ SNI is critical for HTTPS site identification
>
> SNI allows FortiGate to identify the requested hostname during TLS negotiation.

> ### 3️⃣ Replace mode can use multiple certificates
>
> FortiGate can select a replacement certificate based on the requested server identity.

> ### 4️⃣ Domain reputation and IP reputation can disagree
>
> Shared hosting is one important reason why IP reputation may not accurately represent a specific domain.

> ### 5️⃣ Web Rating Override is a classification override
>
> It changes how a website is categorized for Web Filter decisions.

> ### 6️⃣ User-group scope matters
>
> A group-level override can affect other members of the same group.

> ### 7️⃣ Temporary access is safer than permanent exceptions
>
> Use timed overrides when the business requirement is temporary.

---

# ⚡ One-Minute Mental Model

```text
                    ┌──────────────────────┐
                    │      Client          │
                    └──────────┬───────────┘
                               │
                         TCP + TLS
                               │
                               ▼
                    ┌──────────────────────┐
                    │      FortiGate       │
                    └──────────┬───────────┘
                               │
                ┌──────────────┼──────────────┐
                │              │              │
                ▼              ▼              ▼
              SNI            ALPN       Certificate
                │              │         Validation/
                │              │         Replacement
                ▼              ▼              │
             Domain        HTTP/1.1          │
             Identity       /HTTP2            │
                │                             │
                └──────────────┬──────────────┘
                               ▼
                       FortiGuard Rating
                               │
                    ┌──────────┴──────────┐
                    │                     │
               Domain Rating          IP Rating
                    │                     │
                    └──────────┬──────────┘
                               ▼
                       Web Rating Override
                               │
                               ▼
                       Web Filter Profile
                               │
                    ┌──────────┴──────────┐
                    │                     │
                  Allow                 Block
                    │
                    ▼
                 Client
```

---

## 🧩 Core CLI Snippets

### ALPN

```cli
config firewall ssl-ssh-profile
    edit <profile>
        set supported-alpn all
    next
end
```

### Multiple Certificates

```cli
config firewall ssl-ssh-profile
    edit multi-cert
        set server-cert-mode replace
        set server-cert bbb aaa
    next
end
```

### SSL IP Rating Exemption

```cli
config firewall ssl-ssh-profile
    edit <profile>
        set ssl-exemption-ip-rating enable
    next
end
```

### HTTP IP Rating

```cli
config firewall profile-protocol-options
    edit <profile>
        config http
            set address-ip-rating enable
        end
    next
end
```

### External Resource

```cli
config system external-resource
    edit 1
        set category 192
        set resource http://192.168.20.200/lists/blocklist.txt
    next
end
```

---
 
**  topics:**

```text
FortiOS
SSL Inspection
Deep Inspection
Certificate Inspection
HTTP2
ALPN
SNI
Certificate Replacement
FortiGuard
Web Filtering
Web Rating Override
IP Reputation
Domain Reputation
Security Profiles
Network Security
NSE4
NSE7 
 
temporary web access controls.
```

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

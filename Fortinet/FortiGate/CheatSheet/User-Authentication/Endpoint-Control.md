# FortiGate Endpoint Control & Compliance  

> **FortiOS Endpoint Control, Compliance & Authentication**
>
> Practical reference for endpoint identity, compliance verification, disclaimer messages, Web Single Sign-On, FortiClient/EMS integration, authentication timeout, and captive-portal email collection.

---

## 1. Endpoint Control & Compliance — Overview

FortiGate can integrate **endpoint identity and compliance information** into authentication and access-control decisions.

The major components covered in this are:

* Disclaimer messages
* Captive portal
* Web Single Sign-On (WSO)
* VM uniqueness
* Hardware uniqueness
* File-system integrity checking
* FortiClient EMS integration
* Endpoint identity
* Authentication timeout
* Email collection through captive portal
* Username/password authentication source order

### High-Level Architecture

```text
                         FortiGate
                            │
             ┌──────────────┼──────────────┐
             │              │              │
             ▼              ▼              ▼
        Authentication   Endpoint       Compliance
             │            Control            │
       ┌─────┼─────┐        │          ┌─────┴─────┐
       │     │     │        │          │           │
      LDAP RADIUS TACACS   EMS       VM          Hardware
       │     │     │        │       Identity     Identity
       └─────┼─────┘        │
             │              ▼
             ▼         Device Information
        User Identity
```

---

# 2. Disclaimer Message

A **disclaimer message** allows FortiGate to present a notice or agreement before permitting access.

The feature can be useful for:

* Captive portal access
* FortiClient installation/registration workflows
* User acknowledgement
* Acceptable-use policies
* Terms and conditions

### VDOM Scope

Disclaimer messages can be configured **per VDOM**.

---

## 3. Enable Policy Disclaimer

First enable the required GUI/feature visibility options.

### Global GUI Replacement Message Groups

```bash
config system global
    set gui-replacement-message-groups enable
end
```

### Enable Policy Disclaimer

```bash
config system settings
    set gui-policy-disclaimer enable
end
```

Then enable the disclaimer on the required firewall policy:

```bash
config firewall policy
    edit 1
        set disclaimer enable
    next
end
```

### Concept

```text
Client
  │
  ▼
Firewall Policy
  │
  ▼
Disclaimer
  │
  ├── Accept ──► Continue
  │
  └── Reject ──► Access denied
```

---

# 4. Web Single Sign-On (WSO)

WSO can be used with disclaimer-based authentication to control whether clients must authenticate again after their browser session changes.

### Example

```bash
config firewall policy
    edit 1
        set wsa disable
        set disclaimer enable
    next
end
```

### Important Concept

With:

```bash
set wsa disable
```

the client may be required to authenticate again after closing the browser.

The enabled behavior can be used when session/tracking functionality is required.

### Practical Flow

```text
Client
  │
  ▼
Captive Portal
  │
  ▼
Disclaimer
  │
  ▼
Authentication
  │
  ▼
Access
  │
  ▼
Browser closed
  │
  ▼
New authentication may be required
```

> **Exam Tip:** Do not confuse the disclaimer itself with authentication. The disclaimer is an acknowledgement/control mechanism; authentication determines the user's identity.

---

# 5. Compliance & Device Uniqueness

FortiGate can use device characteristics to establish endpoint uniqueness.

Two important concepts are:

```text
Device Uniqueness
       │
       ├── VM uniqueness
       │
       └── Hardware uniqueness
```

---

## 6. VM Uniqueness

For virtual FortiGate deployments, uniqueness can be associated with identifiers such as:

* Certificate information
* Serial-related identifiers
* Firmware/version-related binding

The purpose is to prevent a virtual instance from being treated as an arbitrary duplicate of another instance.

### Mental Model

```text
Virtual FortiGate
       │
       ├── Certificate
       ├── Serial-related identity
       └── Firmware identity
                │
                ▼
          Instance uniqueness
```

---

# 7. Hardware Uniqueness

For hardware appliances, uniqueness can be associated with hardware-specific identifiers.

Common concepts include:

* BIOS identity
* Hardware serial number

```text
Physical FortiGate
       │
       ├── BIOS
       └── Serial Number
              │
              ▼
       Hardware Identity
```

### VM vs Hardware

| Environment        | Primary Uniqueness Concept |
| ------------------ | -------------------------- |
| Virtual FortiGate  | VM-specific identity       |
| Physical FortiGate | BIOS / hardware serial     |

---

# 8. Automatic File-System Check

FortiGate can perform a file-system check during startup.

This can be particularly useful after situations such as:

* Unsafe reboot
* Unexpected power loss
* File-system inconsistency
* Abnormal shutdown

### GUI

```text
System
└── Settings
    └── Startup Settings
        └── Auto File System Check
```

Enable:

```text
Auto File System Check
```

### Concept

```text
Unexpected / Unsafe Reboot
          │
          ▼
      FortiGate
          │
          ▼
 Startup File-System Check
          │
          ▼
 Check FortiOS File System
```

> **Operational Tip:** Consider enabling this when the appliance is exposed to frequent abnormal shutdowns or unstable power conditions.

---

# 9. FortiClient EMS Integration

FortiGate can integrate endpoint information from **FortiClient EMS**.

This allows FortiGate to obtain endpoint/device information that can be used for endpoint-control decisions.

### Diagnostic Command

```bash
diagnose user-device-store device memory list
```

---

## 10. Configure EMS Server

Example:

```bash
config endpoint-control fctems
    edit ems-test
        set server 172.18.62.12
        set certificate-fingerprint 4F:A6:76:E2:00:4F:A6:76:E2:00:4F:A6:76:E2:00:E0
    next
end
```

### Certificate Fingerprint

The EMS certificate fingerprint provides a way for FortiGate to verify the identity of the EMS server.

### Endpoint Identity

Depending on the environment, endpoint identity/compliance can use device-specific information such as:

```text
BIOS
Serial Number
Certificate / EMS Identity
```

---

# 11. Authentication Source Order

FortiGate supports multiple external authentication repositories.

Common sources include:

```text
Local Users
     ↓
RADIUS
     ↓
LDAP
     ↓
TACACS+
```

The important point is that authentication source configuration determines **where FortiGate obtains/validates user identity**.

### Quick Comparison

| Source  | Typical Use                               |
| ------- | ----------------------------------------- |
| Local   | Local FortiGate accounts                  |
| RADIUS  | Central authentication / network access   |
| LDAP    | Directory-based authentication            |
| TACACS+ | Centralized administrative authentication |

> **NSE Tip:** Do not assume that every authentication source provides the same features. Group mapping, SSO, authorization and accounting capabilities vary by protocol.

---

# 12. Username Case Sensitivity

Username comparison can be controlled through user configuration.

Example:

```bash
config user local
    edit 1
        set username-sensetivity disable
        set ldap-server win-srv-2016
    next
end
```

### Concept

With username sensitivity disabled:

```text
User1
user1
USER1
```

can be treated without case-sensitive distinction depending on the authentication mechanism/configuration.

> **Important:** Always validate the exact behavior against the authentication backend and FortiOS version being used.

---

# 13. FSSO vs RSSO

### FSSO

FSSO is primarily used for user identity information associated with environments such as:

* Microsoft Windows
* Citrix
* Novell / directory environments

The FortiGate can use the identity information to create policy decisions without requiring users to manually authenticate to every resource.

### RSSO

**RADIUS Single Sign-On (RSSO)** uses RADIUS authentication/accounting information.

Conceptually:

```text
Client
  │
  ▼
RADIUS
  │
  ▼
Authentication / Accounting Information
  │
  ▼
FortiGate RSSO
  │
  ▼
User Identity
  │
  ▼
Firewall Policy
```

> **Key distinction:** FSSO and RSSO both provide identity awareness, but they obtain identity information through different mechanisms.

---

# 14. Email Collection for Retail / Guest Access

FortiGate can collect an email address through a captive portal.

Typical use cases:

* Retail Wi-Fi
* Guest Wi-Fi
* Free Wi-Fi
* Marketing consent
* Guest identification

---

## 15. Enable Email Collection

### GUI

```text
System
└── Feature Visibility
    └── Email Collection
```

Enable:

```text
Email Collection
```

---

# 16. Enable Switch Controller / Wireless Features

Example:

```bash
config system global
    set switch-controller enable
end
```

After enabling the required functionality, additional wireless/switch-controller options may become available.

---

# 17. Configure Captive Portal Email Collection

Example:

```bash
config wireless-controller vap
    edit freewifi
        set security captive-portal
        set portal-type email-collect
    next
end
```

Then configure the corresponding wireless interface security mode to use the email-collection portal.

### Firewall Policy

```bash
config firewall policy
    edit 1
        set email-collect enable
    next
end
```

### Authentication Flow

```text
Guest Client
     │
     ▼
Wi-Fi
     │
     ▼
Captive Portal
     │
     ▼
Email Collection
     │
     ▼
Email Submitted
     │
     ▼
Firewall Policy
     │
     ▼
Internet Access
```

---

# 18. Diagnose Email / Captive Portal Authentication

Useful command:

```bash
diagnose firewall auth mac list
```

This can help inspect authentication information associated with MAC addresses.

---

# 19. Authentication Timeout

FortiGate can control how long authenticated users remain valid.

### CLI

```bash
config user setting
    set auth-timeout-type idle-timeout
    set auth-timeout 5
end
```

`auth-timeout` is specified in **minutes**.

---

# 20. Authentication Timeout Types

There are three important concepts:

```text
Authentication Timeout
       │
       ├── Idle
       ├── Hard
       └── Session
```

---

## 21. Idle Timeout

```text
User authenticates
       │
       ▼
Traffic/activity
       │
       ▼
No activity
       │
       ▼
Idle timer expires
       │
       ▼
Authenticated sessions terminated
```

Example:

```bash
set auth-timeout-type idle-timeout
set auth-timeout 5
```

After the configured idle period, authentication/session state can expire.

### Typical Use

**Idle timeout** is generally the preferred option when you want authentication to expire after a period of inactivity rather than after a fixed wall-clock duration.

---

# 22. Hard Timeout

Hard timeout is based on the elapsed time since authentication.

```text
Authentication
     │
     ▼
5 minutes
     │
     ▼
Authentication expires
```

Even if the user remains active, the authentication lifetime reaches its configured limit.

---

# 23. Session Timeout

Session-based behavior applies to new session establishment while existing sessions may continue according to the configured behavior.

Conceptually:

```text
Authentication
     │
     ▼
Timeout reached
     │
     ├── New sessions → blocked
     │
     └── Existing sessions → may continue
```

> **NSE Tip:** The practical difference between **idle**, **hard**, and **session** timeout is critical when troubleshooting why a user is disconnected versus why only new connections are blocked.

---

# 24. User Group Authentication Timeout

A group can have its own authentication timeout.

Example:

```bash
config user group
    edit grp-test
        set authtimeout 0
    next
end
```

### `authtimeout 0`

```text
0
│
▼
Use global/default authentication timeout
```

The valid timeout range is:

```text
0–43200 minutes
```

---

# 25. RADIUS Group Exception

When multiple RADIUS groups are involved, group-level `authtimeout` behavior can be different.

In the described configuration:

```text
Multiple RADIUS groups
        │
        ▼
Group authtimeout may be ignored
        │
        ▼
Global user authentication settings apply
        │
        ▼
auth-timeout
```

Therefore, when troubleshooting authentication expiration, check both:

```bash
config user setting
```

and:

```bash
config user group
```

---

# 26. Authentication Timeout Decision Tree

```text
                    User authenticated
                           │
                           ▼
                 Authentication Timeout
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
        Idle             Hard           Session
          │                │                │
          ▼                ▼                ▼
   No activity      Fixed lifetime    New session
          │                │             behavior
          ▼                ▼
       Expire           Expire
```

---

# 27. Endpoint Control & Compliance Checklist

### Disclaimer

* [ ] Enable required disclaimer feature visibility
* [ ] Enable policy disclaimer
* [ ] Configure disclaimer message
* [ ] Enable disclaimer on the firewall policy
* [ ] Test accept/reject behavior

### WSO

* [ ] Understand WSO/session tracking behavior
* [ ] Test browser close/reopen behavior
* [ ] Verify whether re-authentication is required

### Endpoint Identity

* [ ] Verify VM uniqueness
* [ ] Verify hardware uniqueness
* [ ] Verify serial/BIOS-based identity where applicable
* [ ] Verify certificate-based identity where applicable

### File System

* [ ] Enable automatic file-system check when required
* [ ] Test startup behavior after controlled reboot

### EMS

* [ ] Configure FortiClient EMS
* [ ] Verify EMS certificate fingerprint
* [ ] Verify endpoint information
* [ ] Check endpoint/device store

### Authentication

* [ ] Verify authentication source
* [ ] Verify username case sensitivity
* [ ] Verify FSSO/RSSO requirements
* [ ] Verify authentication timeout
* [ ] Verify group timeout

### Captive Portal

* [ ] Enable email collection
* [ ] Configure captive portal
* [ ] Configure wireless/VAP
* [ ] Enable email collection on policy
* [ ] Test guest authentication
* [ ] Verify MAC authentication state

---

# 28. High-Value CLI Reference

### Authentication Settings

```bash
config user setting
    set auth-timeout-type idle-timeout
    set auth-timeout 5
end
```

### User Group Timeout

```bash
config user group
    edit grp-test
        set authtimeout 0
    next
end
```

### Policy Disclaimer

```bash
config system global
    set gui-replacement-message-groups enable
end

config system settings
    set gui-policy-disclaimer enable
end

config firewall policy
    edit 1
        set disclaimer enable
    next
end
```

### EMS

```bash
config endpoint-control fctems
    edit ems-test
        set server <EMS-IP>
        set certificate-fingerprint <FINGERPRINT>
    next
end
```

### File-System Check

```text
System
└── Settings
    └── Startup Settings
        └── Auto File System Check
```

---

# 29. Troubleshooting Commands

### Endpoint Device Store

```bash
diagnose user-device-store device memory list
```

### Authentication / MAC Information

```bash
diagnose firewall auth mac list
```

---

# 30. NSE Exam Memory Map

```text
             ENDPOINT CONTROL & COMPLIANCE
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
    Disclaimer          EMS          Authentication
        │                │                │
        ▼                ▼                ▼
     Captive           Endpoint       Timeout
     Portal            Identity           │
        │                │          ┌──────┼──────┐
        ▼                ▼          │      │      │
   Email Collect      VM/Hardware  Idle   Hard  Session
```

### Remember

```text
Disclaimer
    ≠
Authentication
```

```text
FSSO
    → Identity from SSO mechanisms

RSSO
    → Identity from RADIUS-related information

EMS
    → Endpoint/device information

Authentication Timeout
    → Controls authentication lifetime/behavior
```

---

# 31. One-Minute Interview Answer

**Q: What does FortiGate Endpoint Control & Compliance provide?**

> FortiGate can use endpoint identity and authentication information to make more granular access-control decisions. Depending on the deployment, this can involve FortiClient EMS, device identifiers, captive portals, disclaimers, FSSO/RSSO, and authentication timeout policies.

**Q: What is the difference between idle and hard authentication timeout?**

> Idle timeout expires authentication after the configured period of inactivity, while hard timeout expires authentication after a fixed amount of elapsed time regardless of activity.

**Q: What is the purpose of a disclaimer?**

> A disclaimer presents an acknowledgement or policy message before access and is commonly used with captive portals and controlled access workflows. It should not be confused with the authentication mechanism itself.

---

## 🔥 SheynShield Quick Recall

```text
DISCLAIMER
    ↓
User acknowledgement

CAPTIVE PORTAL
    ↓
Controlled client access

EMAIL COLLECTION
    ↓
Guest / retail identification

EMS
    ↓
Endpoint information

FSSO
    ↓
User identity

RSSO
    ↓
RADIUS-derived identity

VM / HARDWARE UNIQUENESS
    ↓
Device identity

AUTH TIMEOUT
    ↓
Control authentication lifetime
```

> **Core principle:**
> **Identity + Endpoint State + Authentication State + Policy = Controlled Network Access**


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

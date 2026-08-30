# 🔐 FortiGate Authentication Hardening Checklist

> **FortiOS 7.2.x | NSE4 / NSE7 | Authentication Settings | User Authentication | FSSO | Authentication Lockout**
>
> **SheynShield | Engineering Secure Networks**
>
> A practical **FortiGate Authentication Hardening Checklist** for securing authentication behavior, login protection, HTTP-to-HTTPS authentication, authentication policy extension, FSSO integration, and troubleshooting.

---

## 📌 Table of Contents

* [1. Authentication Security Baseline](#1-authentication-security-baseline)
* [2. Authentication Settings](#2-authentication-settings)
* [3. HTTP Authentication Redirect](#3-http-authentication-redirect)
* [4. Authentication Lockout](#4-authentication-lockout)
* [5. Authentication Policy Extension](#5-authentication-policy-extension)
* [6. `auth-on-demand` Modes](#6-auth-on-demand-modes)
* [7. Authentication and Firewall Policies](#7-authentication-and-firewall-policies)
* [8. FSSO Validation](#8-fsso-validation)
* [9. Authentication Troubleshooting](#9-authentication-troubleshooting)
* [10. Configuration Validation](#10-configuration-validation)
* [11. NSE4 / NSE7 Exam Traps](#11-nse4--nse7-exam-traps)
* [12. Quick CLI Reference](#12-quick-cli-reference)
* [13. Production Authentication Checklist](#13-production-authentication-checklist)
* [14. One-Minute Memory Map](#14-one-minute-memory-map)
* [15. SEO Keywords](#15-seo-keywords)
* [16. SheynShield Resources](#16-sheynshield-resources)

---

# 1. Authentication Security Baseline

## 🎯 Pre-Deployment Checklist

* [ ] Identify every FortiGate authentication method in use.
* [ ] Identify local users.
* [ ] Identify LDAP authentication.
* [ ] Identify RADIUS authentication.
* [ ] Identify TACACS+ authentication where applicable.
* [ ] Identify FSSO authentication.
* [ ] Identify firewall policies requiring authentication.
* [ ] Identify administrator authentication separately from user authentication.
* [ ] Document authentication dependencies.
* [ ] Document authentication server IP addresses/FQDNs.
* [ ] Verify routing toward authentication servers.
* [ ] Verify DNS resolution where FQDN-based authentication is used.
* [ ] Verify authentication server availability.
* [ ] Verify time synchronization.
* [ ] Verify certificates where HTTPS authentication is involved.

> ⚠️ **Do not treat "authentication failure" as automatically meaning "authentication server failure."**

The failure can occur anywhere in the authentication chain.

```text
Client
   ↓
Interface
   ↓
Firewall Policy
   ↓
Authentication Requirement
   ↓
Authentication Method
   ↓
Authentication Server
   ↓
User / Group
   ↓
Authentication Result
   ↓
Security Policy
   ↓
Destination
```

---

# 2. Authentication Settings

## 📍 GUI Location

```text
User & Authentication
└── Authentication Settings
```

Before changing authentication behavior:

* [ ] Review current authentication settings.
* [ ] Record the existing configuration.
* [ ] Identify lockout settings.
* [ ] Identify invalid-attempt behavior.
* [ ] Identify `auth-on-demand`.
* [ ] Identify HTTP authentication redirect behavior.
* [ ] Identify policies using authentication.
* [ ] Identify policies using FSSO.

---

## 🔎 Core Authentication Parameters

| Parameter                | Purpose                                                 |
| ------------------------ | ------------------------------------------------------- |
| `auth-lockout-threshold` | Number of failed authentication attempts before lockout |
| `auth-lockout-duration`  | Duration of authentication lockout                      |
| `auth-invalid-max`       | Maximum invalid authentication attempts                 |
| `auth-on-demand`         | Controls authentication-on-demand behavior              |
| `fsso`                   | Controls FSSO behavior on a firewall policy             |

---

# 3. HTTP Authentication Redirect

## 🔐 Authentication Transport Security

When HTTP authentication is redirected to HTTPS, the logical flow becomes:

```text
HTTP Request
     ↓
FortiGate Authentication
     ↓
HTTPS Redirect
     ↓
Secure Authentication
     ↓
Credential Validation
```

## ✅ Hardening Checklist

* [ ] Prefer HTTPS for authentication.
* [ ] Enable HTTP-to-HTTPS authentication redirect when appropriate.
* [ ] Verify required HTTP/HTTPS protocols are enabled.
* [ ] Verify the authentication interface is reachable.
* [ ] Install a valid certificate where required.
* [ ] Verify certificate validity.
* [ ] Verify certificate hostname/SAN requirements.
* [ ] Test authentication from a supported client.
* [ ] Test the redirect behavior.
* [ ] Verify that authentication succeeds after redirection.
* [ ] Check logs if the redirect fails.

### Security Principle

```text
HTTP
 ↓
Redirect
 ↓
HTTPS
 ↓
Authentication
```

Avoid transmitting authentication credentials unnecessarily over cleartext HTTP.

---

## ⚠️ Replacement Message Limitation

FortiGate does **not** provide customizable authentication replacement messages for:

```text
FTP authentication
Telnet authentication
```

Do not assume every authentication protocol supports the same replacement-message customization.

---

# 4. Authentication Lockout

Authentication lockout helps protect against repeated invalid authentication attempts.

## 🔧 Example Configuration

```bash
config user setting
    set auth-lockout-threshold 2
    set auth-lockout-duration 30
    set auth-invalid-max 2
end
```

---

## 🧠 Parameter Checklist

### `auth-lockout-threshold`

Controls the failed-attempt threshold associated with authentication lockout.

* [ ] Define an appropriate threshold.
* [ ] Avoid unnecessarily high values.
* [ ] Avoid excessively aggressive values that could lock out legitimate users.
* [ ] Test the configured threshold.

---

### `auth-lockout-duration`

Controls how long the authentication lockout remains active.

* [ ] Configure an appropriate duration.
* [ ] Verify the unit used by the FortiOS release.
* [ ] Test the actual lockout duration.
* [ ] Confirm the behavior in authentication logs.

---

### `auth-invalid-max`

Controls the maximum invalid authentication attempts used by FortiOS authentication handling.

* [ ] Understand how it interacts with authentication lockout.
* [ ] Configure according to the authentication design.
* [ ] Test repeated invalid credentials.
* [ ] Verify the resulting behavior.

---

## 🔥 Authentication Lockout Flow

```text
Attempt #1
   ↓
Invalid
   ↓
Continue

Attempt #2
   ↓
Invalid
   ↓
Threshold reached
   ↓
Lockout
   ↓
Configured duration
   ↓
Authentication available again
```

---

## 🛡️ Production Checklist

* [ ] Enable authentication lockout.
* [ ] Define a documented threshold.
* [ ] Define a documented duration.
* [ ] Test legitimate login behavior.
* [ ] Test invalid credentials.
* [ ] Test repeated failed authentication.
* [ ] Verify lockout.
* [ ] Verify lockout expiration.
* [ ] Monitor authentication logs.
* [ ] Monitor for repeated attacks.
* [ ] Ensure the threshold does not create an unnecessary denial-of-service condition for legitimate users.

---

# 5. Authentication Policy Extension

Authentication policy extension controls authentication requirements when accessing protected resources through firewall-policy processing.

## 🔧 Configuration

```bash
config user setting
    set auth-on-demand always
end
```

---

## 🔐 Authentication Flow

```text
User
 ↓
Request Protected Resource
 ↓
Authentication Required
 ↓
Credential Validation
 ↓
Authentication Successful?
 ├── YES → Continue
 └── NO  → Deny
```

---

## ✅ Validation Checklist

* [ ] Identify resources requiring authentication.
* [ ] Identify policies protecting those resources.
* [ ] Verify `auth-on-demand`.
* [ ] Verify authentication method.
* [ ] Verify user/group membership.
* [ ] Test unauthenticated access.
* [ ] Test authenticated access.
* [ ] Verify authentication failure behavior.
* [ ] Verify authentication success behavior.

---

# 6. `auth-on-demand` Modes

The `auth-on-demand` setting controls how authentication state is handled.

## `always`

```text
Connection / Resource Request
          ↓
Authentication Required
          ↓
Credentials Verified
          ↓
Access Continues
```

Think:

> **Authenticate before the protected connection/session proceeds.**

---

## `implicit`

Conceptually:

```text
Initial Authentication
       ↓
Authentication State
       ↓
Subsequent Connection
       ↓
Existing Authentication State Reused
```

Think:

> **An established authentication state can be reused implicitly.**

---

## 📊 Quick Comparison

| Mode       | Concept                                                                             |
| ---------- | ----------------------------------------------------------------------------------- |
| `always`   | Authentication is explicitly required whenever the authentication condition applies |
| `implicit` | Existing authentication state can be reused for subsequent connections              |

### NSE Memory Trick

```text
always
  ↓
Authenticate again when required

implicit
  ↓
Reuse authentication state
```

---

# 7. Authentication and Firewall Policies

Authentication settings do **not** replace firewall-policy processing.

The firewall policy still determines whether traffic is allowed.

## 🧠 Decision Model

```text
Client
  ↓
Incoming Interface
  ↓
Firewall Policy Match
  ↓
Authentication Requirement
  ↓
Authentication
  ↓
Identity / Group
  ↓
Policy Decision
  ↓
Destination
```

---

## 🔐 Firewall Policy Checklist

For every authentication-dependent policy:

* [ ] Verify source interface.
* [ ] Verify destination interface.
* [ ] Verify source address.
* [ ] Verify destination address.
* [ ] Verify service.
* [ ] Verify schedule.
* [ ] Verify source identity/group.
* [ ] Verify authentication requirement.
* [ ] Verify authentication method.
* [ ] Verify FSSO configuration.
* [ ] Verify NAT requirements.
* [ ] Verify security profiles.
* [ ] Verify policy order.
* [ ] Verify logs.

---

# 8. FSSO Validation

FortiGate can use **Fortinet Single Sign-On (FSSO)** to associate users with network identities.

## 🔧 Policy Example

```bash
config firewall policy
    edit 1
        set fsso disable
    next
end
```

This explicitly disables FSSO for the selected firewall policy.

---

## 🧩 FSSO Decision Checklist

* [ ] Determine whether the policy requires FSSO.
* [ ] Verify `set fsso`.
* [ ] Verify FSSO agent/collector connectivity where applicable.
* [ ] Verify user identity information.
* [ ] Verify group membership.
* [ ] Verify identity-to-policy mapping.
* [ ] Verify authentication logs.
* [ ] Verify the expected firewall policy is matched.
* [ ] Test with an authenticated user.
* [ ] Test with an unauthenticated user.

---

## ⚠️ Important Distinction

Do not confuse:

```text
Local Authentication
LDAP
RADIUS
TACACS+
FSSO
```

They represent different authentication/identity mechanisms and may require different troubleshooting approaches.

---

# 9. Authentication Troubleshooting

## 🚨 Troubleshooting Workflow

When authentication fails, troubleshoot from the client toward the destination.

```text
1. Client
   ↓
2. Network Connectivity
   ↓
3. FortiGate Interface
   ↓
4. Firewall Policy
   ↓
5. Authentication Requirement
   ↓
6. Authentication Method
   ↓
7. Authentication Server
   ↓
8. User / Group
   ↓
9. Authentication Result
   ↓
10. Security Policy
   ↓
11. Destination
```

---

## 🔍 Layer 1 — Client

* [ ] Confirm client IP address.
* [ ] Confirm client can reach FortiGate.
* [ ] Confirm correct gateway.
* [ ] Confirm DNS where required.
* [ ] Confirm browser/application compatibility.
* [ ] Confirm system time where certificates/authentication depend on it.

---

## 🔍 Layer 2 — Firewall Policy

* [ ] Identify the matched firewall policy.
* [ ] Verify source interface.
* [ ] Verify destination interface.
* [ ] Verify source address.
* [ ] Verify destination address.
* [ ] Verify service.
* [ ] Verify schedule.
* [ ] Verify authentication settings.
* [ ] Verify FSSO behavior.

---

## 🔍 Layer 3 — Authentication

* [ ] Verify authentication method.
* [ ] Verify username.
* [ ] Verify password.
* [ ] Verify account status.
* [ ] Verify user/group membership.
* [ ] Verify authentication server reachability.
* [ ] Verify authentication server response.
* [ ] Check for account lockout.
* [ ] Check invalid-attempt counters.
* [ ] Check authentication logs.

---

## 🔍 Layer 4 — HTTPS Redirect

If HTTP authentication is redirected:

* [ ] Confirm HTTP request reaches FortiGate.
* [ ] Confirm redirect occurs.
* [ ] Confirm HTTPS listener is reachable.
* [ ] Verify certificate.
* [ ] Verify certificate hostname.
* [ ] Verify browser trust.
* [ ] Verify authentication page loads.
* [ ] Verify credentials are accepted.

---

# 10. Configuration Validation

## 📋 Before Change

* [ ] Backup current configuration.
* [ ] Record current authentication settings.
* [ ] Record firewall policies affected.
* [ ] Record authentication server configuration.
* [ ] Document the change.
* [ ] Define rollback procedure.
* [ ] Identify expected behavior after the change.

---

## 📋 After Change

* [ ] Verify configuration.
* [ ] Test valid credentials.
* [ ] Test invalid credentials.
* [ ] Test lockout behavior.
* [ ] Test authentication recovery.
* [ ] Test protected-resource access.
* [ ] Test unauthorized access.
* [ ] Verify firewall policy logs.
* [ ] Verify authentication logs.
* [ ] Verify FSSO behavior where applicable.

---

# 11. NSE4 / NSE7 Exam Traps

## 🚨 Trap #1 — Lockout Threshold vs Duration

```text
auth-lockout-threshold
        ↓
How many failed attempts?

auth-lockout-duration
        ↓
How long is the lockout?
```

Do not confuse these two settings.

---

## 🚨 Trap #2 — `auth-invalid-max`

```text
auth-invalid-max
        ↓
Maximum invalid authentication attempts
```

Do not automatically assume it is identical to `auth-lockout-threshold`.

---

## 🚨 Trap #3 — `auth-on-demand`

```text
auth-on-demand
        ↓
Authentication behavior
```

It is not a firewall-policy replacement.

---

## 🚨 Trap #4 — FSSO

```bash
set fsso disable
```

means:

```text
FSSO disabled
for that firewall policy
```

It does not mean:

```text
All FortiGate authentication disabled
```

---

## 🚨 Trap #5 — HTTP Redirect

```text
HTTP Redirect
      ↓
HTTPS
```

Redirecting HTTP to HTTPS improves transport security, but certificate validation and HTTPS configuration still matter.

---

## 🚨 Trap #6 — Authentication Server Failure

An authentication failure does **not** automatically indicate an LDAP/RADIUS/FSSO server problem.

Check:

```text
Policy
 ↓
Authentication Requirement
 ↓
Authentication Method
 ↓
Server Reachability
 ↓
Credentials
 ↓
Group Membership
 ↓
Policy Result
```

---

## 🚨 Trap #7 — Authentication ≠ Authorization

Authentication answers:

> **Who are you?**

Authorization answers:

> **What are you allowed to access?**

FortiGate firewall policies participate in the authorization decision.

---

# 12. Quick CLI Reference

## Authentication Lockout

```bash
config user setting
    set auth-lockout-threshold 2
    set auth-lockout-duration 30
    set auth-invalid-max 2
end
```

---

## Authentication-On-Demand

```bash
config user setting
    set auth-on-demand always
end
```

---

## Disable FSSO for a Policy

```bash
config firewall policy
    edit 1
        set fsso disable
    next
end
```

---

## 🔎 Configuration Verification

After modifying the configuration:

* [ ] Verify the resulting configuration.
* [ ] Confirm the intended values are active.
* [ ] Test authentication.
* [ ] Test lockout.
* [ ] Test firewall policy matching.
* [ ] Review logs.

> ⚠️ Exact CLI availability and behavior can vary by FortiOS release and platform. Validate commands against the target FortiOS version before applying them in production.

---

# 13. Production Authentication Checklist

## 🔐 Authentication Security

* [ ] Use strong authentication controls.
* [ ] Configure authentication lockout.
* [ ] Define an appropriate failed-attempt threshold.
* [ ] Define an appropriate lockout duration.
* [ ] Configure invalid-attempt handling.
* [ ] Monitor repeated authentication failures.
* [ ] Review authentication logs regularly.
* [ ] Avoid unnecessarily permissive authentication behavior.

---

## 🌐 HTTP / HTTPS

* [ ] Prefer HTTPS authentication.
* [ ] Enable HTTP-to-HTTPS redirect when appropriate.
* [ ] Use a valid certificate.
* [ ] Verify certificate hostname.
* [ ] Verify browser/client trust.
* [ ] Test the authentication redirect.
* [ ] Avoid unnecessary cleartext authentication.

---

## 🧑‍💻 User / Group Authentication

* [ ] Validate user identity.
* [ ] Validate group membership.
* [ ] Validate LDAP/RADIUS/FSSO connectivity.
* [ ] Validate authentication server reachability.
* [ ] Validate authentication responses.
* [ ] Verify account lockout status.
* [ ] Verify authentication state.

---

## 🔥 Firewall Policy

* [ ] Verify policy order.
* [ ] Verify source interface.
* [ ] Verify destination interface.
* [ ] Verify source address.
* [ ] Verify destination address.
* [ ] Verify service.
* [ ] Verify schedule.
* [ ] Verify identity/group.
* [ ] Verify authentication requirement.
* [ ] Verify FSSO configuration.
* [ ] Verify security profiles.
* [ ] Verify NAT behavior.
* [ ] Verify logging.

---

## 🧪 Validation

* [ ] Test successful authentication.
* [ ] Test invalid credentials.
* [ ] Test repeated invalid credentials.
* [ ] Verify lockout.
* [ ] Verify lockout expiration.
* [ ] Test HTTP-to-HTTPS redirect.
* [ ] Test unauthorized access.
* [ ] Test authorized access.
* [ ] Test FSSO where applicable.
* [ ] Review logs after testing.

---

# 14. One-Minute Memory Map

```text
                  FORTIGATE AUTHENTICATION
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
       Redirect          Lockout       Policy Extension
          │                │                │
          ▼                ▼                ▼
       HTTPS          Failed Attempts   auth-on-demand
                           │                │
                           ▼                ▼
                       Threshold       always / implicit
                           │
                           ▼
                        Duration
                           │
                           ▼
                     Firewall Policy
                           │
                 ┌─────────┴─────────┐
                 │                   │
              Identity             FSSO
                 │                   │
                 └─────────┬─────────┘
                           ▼
                    Access Decision
```

---

# 🎯 Core Exam Memory

```text
auth-lockout-threshold
        ↓
Failed-attempt threshold

auth-lockout-duration
        ↓
Lockout duration

auth-invalid-max
        ↓
Maximum invalid authentication attempts

auth-on-demand
        ↓
Authentication behavior

always
        ↓
Authentication required when applicable

implicit
        ↓
Existing authentication state can be reused

fsso disable
        ↓
FSSO disabled on the selected firewall policy
```

---

# 🔥 Authentication Troubleshooting Formula

When authentication fails, remember:

```text
CLIENT
  ↓
CONNECTIVITY
  ↓
INTERFACE
  ↓
FIREWALL POLICY
  ↓
AUTHENTICATION REQUIREMENT
  ↓
AUTHENTICATION METHOD
  ↓
AUTH SERVER
  ↓
USER / GROUP
  ↓
AUTHENTICATION RESULT
  ↓
AUTHORIZATION
  ↓
RESOURCE
```

> **Never jump directly to the authentication server.**

---

# 🧠 SheynShield Takeaway

> **FortiGate authentication troubleshooting is not simply "check the username and password."**
>
> The real troubleshooting chain is:
>
> **Client → Policy → Authentication Requirement → Authentication Method → Identity → Authentication Result → Authorization → Resource**
>
> Understanding this chain lets you distinguish an authentication problem from a policy, identity, connectivity, FSSO, or authorization problem.

---

# 🔎 Keywords

`FortiGate Authentication Settings`
`FortiGate authentication hardening`
`FortiOS authentication`
`FortiGate authentication lockout`
`FortiGate auth-lockout-threshold`
`FortiGate auth-lockout-duration`
`FortiGate auth-invalid-max`
`FortiGate auth-on-demand`
`FortiGate authentication policy`
`FortiGate HTTP HTTPS authentication redirect`
`FortiGate FSSO authentication`
`FortiGate firewall authentication`
`FortiGate user authentication`
`FortiGate LDAP authentication`
`FortiGate RADIUS authentication`
`FortiGate authentication troubleshooting`
`FortiOS NSE4 authentication`
`FortiOS NSE7 authentication`
`FortiGate CLI authentication`
`FortiGate security hardening`

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

**SheynShield | Engineering Secure Networks**

> **Learn the command. Understand the behavior. Troubleshoot the flow.**

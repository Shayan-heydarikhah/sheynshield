# FortiGate Authentication Settings  

> **FortiOS Authentication Settings | User Authentication | Authentication Policy Extension**
>
> Practical reference for authentication behavior, login protection, HTTP-to-HTTPS redirection, lockout controls, and policy authentication.

---

## 1. Authentication Settings — Overview

FortiGate **Authentication Settings** controls how users are handled during authentication and how authentication-related behavior is enforced.

Typical areas include:

* Authentication lockout
* Invalid credential handling
* HTTP authentication redirection
* Authentication timing and enforcement
* Authentication policy extension behavior
* FSSO interaction with firewall policies

**GUI path:**

```text
User & Authentication
└── Authentication Settings
```

---

# 2. HTTP Authentication Redirect

FortiGate can redirect HTTP authentication requests toward HTTPS.

### HTTP Redirect

```text
HTTP request
     │
     ▼
FortiGate authentication
     │
     ▼
HTTPS redirect
     │
     ▼
Secure authentication
```

Enable:

```text
HTTP Redirect
```

### Recommended

If HTTP authentication redirection is enabled:

* Prefer HTTPS for authentication.
* Ensure the required HTTP/HTTPS protocol support is enabled.
* Use a valid certificate where applicable.
* Verify that the required FortiGate feature/license or supported hardware capability is available.

### Important

FortiGate does **not** allow customization of authentication replacement messages for:

* FTP authentication
* Telnet authentication

---

# 3. Authentication Lockout

Authentication lockout protects FortiGate resources against repeated invalid authentication attempts.

### CLI

```bash
config user setting
    set auth-lockout-threshold 2
    set auth-lockout-duration 30
    set auth-invalid-max 2
end
```

### Parameters

| Parameter                | Example | Purpose                                                         |
| ------------------------ | ------: | --------------------------------------------------------------- |
| `auth-lockout-threshold` |     `2` | Number of failed attempts before lockout                        |
| `auth-lockout-duration`  |    `30` | Lockout duration in seconds                                     |
| `auth-invalid-max`       |     `2` | Maximum invalid authentication attempts before lockout handling |

### Example Flow

```text
Authentication attempt #1
        │
        ├── Invalid → continue
        │
Authentication attempt #2
        │
        ├── Invalid
        ▼
Lockout threshold reached
        │
        ▼
User/client is locked
        │
        ▼
Wait for configured duration
        │
        ▼
Authentication can be attempted again
```

### Security Recommendation

Use authentication lockout to reduce the risk of:

* Password guessing
* Brute-force authentication
* Repeated invalid login attempts

> **Design note:** Do not use an excessively low threshold in environments where legitimate users may frequently mistype credentials.

---

# 4. Authentication Policy Extension

Authentication policy extension controls when authentication is required before access to another firewall policy/resource is permitted.

### CLI

```bash
config user setting
    set auth-on-demand always
end
```

---

## 5. `auth-on-demand`

### `always`

```text
User requests resource
        │
        ▼
Authentication required
        │
        ▼
Credentials verified
        │
        ▼
Resource access permitted
```

With:

```bash
set auth-on-demand always
```

the user must complete the required authentication before continuing to access protected resources.

### Concept

```text
NO VALID AUTHENTICATION
        │
        ▼
Access to protected resource
        │
        X
      DENIED
        │
        ▼
Authenticate
        │
        ▼
Credentials validated
        │
        ▼
Access permitted
```

---

## 6. Authentication Behavior Modes

| Mode       | Behavior                                                                         | Typical Concept                         |
| ---------- | -------------------------------------------------------------------------------- | --------------------------------------- |
| `always`   | Authentication is required whenever the authentication condition applies         | Per-session / connection authentication |
| `implicit` | One authentication mechanism can be reused implicitly for subsequent connections | Authentication state reused             |

### `always`

Think:

> **"Authenticate before the connection/session is allowed to proceed."**

### `implicit`

Think:

> **"Authentication has already established an authentication state that can be reused."**

---

# 7. Authentication + Firewall Policy

Authentication settings do not replace firewall-policy logic.

A firewall policy can determine whether authentication is required and what authenticated users are allowed to access.

### Example

```bash
config firewall policy
    edit 1
        set fsso disable
    next
end
```

### `fsso`

```text
set fsso disable
```

explicitly disables FSSO authentication for the selected firewall policy.

This is important when distinguishing between:

* Explicit authentication
* Local firewall authentication
* FSSO-based identity
* LDAP/RADIUS/TACACS+ authentication

---

# 8. Authentication Decision Model

A useful mental model for FortiGate authentication:

```text
                    Client
                      │
                      ▼
               Firewall Policy
                      │
          ┌───────────┴───────────┐
          │                       │
      Identity required?       No identity
          │                       │
         YES                      ▼
          │                    Continue
          ▼
 Authentication mechanism
          │
 ┌────────┼───────────┐
 │        │           │
LDAP    RADIUS       FSSO
 │        │           │
 └────────┼───────────┘
          │
          ▼
    Authentication
       successful?
       │        │
      YES       NO
       │         │
       ▼         ▼
 Continue      Deny
       │
       ▼
Security Policy
       │
       ▼
    Resource
```

---

# 9. Authentication Security Checklist

### Lockout

* [ ] Configure `auth-lockout-threshold`
* [ ] Configure an appropriate `auth-lockout-duration`
* [ ] Configure `auth-invalid-max`
* [ ] Test repeated invalid credentials
* [ ] Verify lockout behavior

### HTTP Authentication

* [ ] Prefer HTTPS authentication
* [ ] Enable HTTP redirect when appropriate
* [ ] Verify HTTP/HTTPS protocol support
* [ ] Install a valid certificate where required
* [ ] Verify the authentication page after redirect

### Authentication Policy

* [ ] Understand `auth-on-demand`
* [ ] Select the appropriate authentication behavior
* [ ] Verify whether authentication is required per session/connection
* [ ] Test access before and after authentication

### Firewall Policy

* [ ] Verify source interface
* [ ] Verify destination interface
* [ ] Verify source identity/group
* [ ] Verify destination
* [ ] Verify service
* [ ] Verify schedule
* [ ] Verify authentication requirements
* [ ] Check whether FSSO is enabled or disabled

---

# 10. NSE Exam / Troubleshooting Quick Notes

### Remember

```text
auth-lockout-threshold
        ↓
Failed authentication attempts
        ↓
Lockout
        ↓
auth-lockout-duration
```

```text
auth-invalid-max
        ↓
Maximum invalid authentication attempts
        ↓
Lockout handling
```

```text
auth-on-demand always
        ↓
Authentication required
        ↓
Credentials verified
        ↓
Access continues
```

```text
set fsso disable
        ↓
FSSO disabled for that firewall policy
```

---

# 11. Quick CLI Reference

```bash
# Authentication settings
config user setting
    set auth-lockout-threshold 2
    set auth-lockout-duration 30
    set auth-invalid-max 2
    set auth-on-demand always
end
```

```bash
# Disable FSSO on a firewall policy
config firewall policy
    edit 1
        set fsso disable
    next
end
```

---

# 12. Troubleshooting Mindset

When a user cannot access a protected resource, do **not** immediately assume that the authentication server is the problem.

Check the chain:

```text
1. Client
   ↓
2. Interface / Zone
   ↓
3. Firewall Policy
   ↓
4. Authentication Requirement
   ↓
5. Authentication Server
   ↓
6. User / Group Membership
   ↓
7. Authentication Result
   ↓
8. Security Policy
   ↓
9. Destination Resource
```

### Fast Troubleshooting Questions

| Question                                | Check                                  |
| --------------------------------------- | -------------------------------------- |
| Is the traffic reaching FortiGate?      | Traffic logs / FortiView               |
| Is the correct policy matched?          | Policy lookup / policy ID              |
| Is authentication required?             | Firewall policy + user settings        |
| Is the user authenticated?              | Authentication/session state           |
| Is the account locked?                  | Lockout settings / authentication logs |
| Is FSSO involved?                       | `set fsso` / FSSO status               |
| Is HTTP being redirected?               | HTTP/HTTPS settings + certificate      |
| Is the authentication server reachable? | Routing + connectivity                 |
| Is the user/group correct?              | LDAP/RADIUS/FSSO group mapping         |

---

## 🔥 One-Minute Memory Map

```text
                 AUTHENTICATION SETTINGS
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
   HTTP Redirect     Lockout        Policy Extension
        │                │                │
        ▼                ▼                ▼
      HTTPS       Failed Attempts     auth-on-demand
                         │                │
                         ▼                ▼
                     Threshold        always / implicit
                         │
                         ▼
                      Duration
```

### Core Commands

```bash
config user setting
    set auth-lockout-threshold <attempts>
    set auth-lockout-duration <seconds>
    set auth-invalid-max <attempts>
    set auth-on-demand always
end
```

### Core Exam Concept

> **Authentication settings determine how FortiGate handles authentication attempts and authentication state; firewall policies determine what authenticated or unauthenticated traffic is ultimately allowed to access.**

---

## Keywords

`FortiGate Authentication Settings` · `FortiOS authentication` · `FortiGate authentication lockout` · `auth-on-demand FortiGate` · `FortiGate HTTP redirect authentication` · `FortiGate firewall authentication` · `FortiGate FSSO` · `FortiOS user authentication` · `FortiGate NSE4 authentication` · `FortiGate CLI authentication`

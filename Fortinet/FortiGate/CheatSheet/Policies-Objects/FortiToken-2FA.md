# 🔐 FortiGate 2-Factor Authentication (2FA) & FortiToken  

> **SheynShield | FortiGate Security Engineering**
>
> Practical reference for **FortiToken, FortiToken Mobile, 2FA, token lifecycle, token synchronization, FTM push, RMA/recovery, and troubleshooting**.

---

## 1. 2FA Overview

**Two-Factor Authentication (2FA)** adds a second authentication factor to the normal username/password authentication process.

Typical flow:

```text
Username + Password
        │
        ▼
Primary Authentication
        │
        ▼
FortiToken / OTP
        │
        ▼
Second-Factor Validation
        │
        ▼
       Access
```

FortiOS supports FortiToken and FortiToken Mobile for OTP-based authentication.

### Common Use Cases

* FortiGate administrator login
* SSL-VPN authentication
* Remote user authentication
* LDAP users
* RADIUS users
* Local users
* FortiAuthenticator-based authentication
* FortiToken Mobile push authentication

---

# 2. FortiToken Components

| Component                   | Purpose                                                |
| --------------------------- | ------------------------------------------------------ |
| **FortiToken**              | Hardware OTP token                                     |
| **FortiToken Mobile (FTM)** | Mobile application generating OTP                      |
| **Seed**                    | Secret cryptographic material used to generate OTPs    |
| **OTP**                     | One-time password                                      |
| **FTM Provisioning**        | Process of assigning/activating a mobile token         |
| **FortiGuard FDS**          | Fortinet backend services involved in token management |

### OTP

Depending on the token/application configuration, OTP codes can be:

```text
6 digits
or
8 digits
```

A token registration/provisioning process contains a seed that is used by the token to generate time-based OTPs.

---

# 3. FortiToken Mobile — Basic Deployment

GUI:

```text
User & Authentication
        ↓
FortiTokens
        ↓
Create New
```

Then assign the token to the required user.

For administrator 2FA:

```text
System
  → Administrators
      → Select Administrator
          → Enable Two-Factor Authentication
          → Select FortiToken
```

### Administrator Contact Information

For FortiToken Mobile activation, make sure the administrator account contains the required:

* Email address
* Mobile number

The FortiGate may use these services to deliver the activation/provisioning information.

---

# 4. Email / SMS Service

Depending on the FortiOS version and deployment:

```text
System
  → Settings / Config
      → Advanced
          → Email Service
          → SMS Service
```

Configure the required notification service before relying on email/SMS-based token provisioning.

---

# 5. Assigning FortiToken to Remote Users

Example:

```text
User & Authentication
    ↓
User Definition
    ↓
Remote LDAP User
    ↓
Select User / Group
    ↓
Enable Two-Factor Authentication
    ↓
Assign FortiToken
```

Then place the user into the appropriate firewall user group.

Example:

```text
LDAP User
   │
   ├── Password Authentication
   │
   └── FortiToken OTP
          │
          ▼
      Firewall Group
          │
          ▼
      Firewall Policy
```

> **Important:** The user must actually be referenced by the authentication mechanism/policy that requires the second factor.

---

# 6. FortiToken CLI

### Import FortiToken Mobile Token

```bash
execute fortitoken-mobile import <activation-code>
```

### Display Token Information

```bash
show user fortitoken
```

### Diagnose FortiToken

```bash
diagnose fortitoken info
```

---

# 7. FortiToken Lifecycle / Status

Understanding token states is critical when troubleshooting provisioning.

| State                 | Meaning                                                               |
| --------------------- | --------------------------------------------------------------------- |
| **New**               | Newly added token; not activated or assigned                          |
| **Available**         | Token is available for assignment                                     |
| **Active**            | Token is assigned and active                                          |
| **Provisioning**      | FortiToken Mobile is assigned and waiting for activation              |
| **Provisioned**       | Token has been activated successfully                                 |
| **Provision Timeout** | Token was provisioned but was not activated within the allowed period |
| **Locked**            | Token has been manually or automatically locked                       |
| **Already Activated** | Token/seed was already activated and cannot be reused normally        |

### Provisioning State

```text
Token Assigned
      ↓
Provisioning
      ↓
User activates FTM
      ↓
Provisioned
```

If activation does not occur within the configured provisioning window:

```text
Provisioning
      ↓
Timeout
      ↓
Re-provision required
```

---

# 8. FortiToken Mobile Provisioning Expiry

Configure the FortiToken Mobile provisioning expiration period:

```bash
config system global
    set two-factor-ftm-expire 72
end
```

> `72` represents **hours**.

The exact available token-expiration options can vary by FortiOS release, so verify the command in the target FortiOS version.

---

# 9. Token Synchronization / Clock Drift

FortiToken OTP generation is time-dependent.

Therefore, synchronization between:

```text
FortiGate
      +
FortiToken
      +
User Device
```

is critical.

### Check

Verify:

* Correct timezone
* Correct system time
* Reliable NTP
* Correct time on mobile/device
* No significant clock drift

### Token Synchronization

Example:

```bash
execute fortitoken sync <token-id> <current-token-code> <next-token-code>
```

Conceptually:

```text
FortiGate Token ID
        +
Current OTP
        +
Next OTP
        ↓
Token Synchronization
```

This can be useful when the FortiToken and FortiGate are out of synchronization.

---

# 10. Token Expiration Timers

FortiOS provides global controls for different second-factor mechanisms.

```bash
config system global
    set two-factor-ftk-expiry <seconds>
    set two-factor-ftm-expiry <seconds>
    set two-factor-sms-expiry <seconds>
    set two-factor-fac-expiry <seconds>
    set two-factor-email-expiry <seconds>
end
```

| Setting                   | Purpose                                   |
| ------------------------- | ----------------------------------------- |
| `two-factor-ftk-expiry`   | FortiToken hardware authentication expiry |
| `two-factor-ftm-expiry`   | FortiToken Mobile expiry                  |
| `two-factor-sms-expiry`   | SMS OTP expiry                            |
| `two-factor-fac-expiry`   | FortiAuthenticator-related 2FA expiry     |
| `two-factor-email-expiry` | Email OTP expiry                          |

> **Operational rule:** Keep OTP expiration windows reasonably short while allowing enough time for users to receive and enter the code.

---

# 11. FortiToken Mobile Push

FortiOS can integrate with FortiAuthenticator for FortiToken Mobile push notifications in supported authentication scenarios.

Example use cases include:

```text
SSL-VPN
   ↓
FortiAuthenticator
   ↓
FortiToken Mobile Push
   ↓
User Approval
   ↓
Authentication
```

RADIUS-based authentication can also participate in this architecture when FortiAuthenticator acts as the RADIUS server.

---

# 12. FortiToken Mobile Connectivity

When FortiToken Mobile provisioning or token management does not work, verify connectivity to the required Fortinet services.

Useful tests:

```bash
execute ping fds1.fortinet.com
execute ping directregistration.fortinet.com
execute ping globalftm.fortinet.net
```

### Troubleshooting Checklist

```text
[ ] DNS resolution works
[ ] Internet connectivity works
[ ] FortiGate can reach Fortinet services
[ ] HTTPS connectivity is allowed
[ ] System time is correct
[ ] NTP is working
[ ] Token is registered
[ ] Token is assigned
[ ] Token is not locked
[ ] Token has not expired
[ ] Mobile activation was completed
```

---

# 13. FTM Push Configuration

Example:

```bash
config system ftm-push
    set status enable
    set server-ip <Fortinet-FTM-server>
    set server-port 4433
end
```

> The actual FTM service endpoint should match the FortiOS/FortiToken architecture and version being deployed.

Also verify that the required FortiGate interface has the appropriate administrative access to the FTM service where applicable.

---

# 14. FortiToken RMA / Token Transfer

**RMA** is relevant when a token needs to be moved/recovered as part of hardware replacement or another supported lifecycle operation.

A token should not simply be copied between FortiGate devices as if it were a normal configuration object.

### Operational Principle

```text
Old FortiGate
     │
     │ Token lifecycle / transfer
     ▼
New FortiGate
     │
     ▼
Token reassignment
```

Before performing token recovery or migration:

```text
[ ] Verify token ownership
[ ] Verify token status
[ ] Confirm target FortiGate
[ ] Preserve required configuration
[ ] Follow Fortinet-supported transfer/RMA procedure
```

---

# 15. Emergency Administrative Access

A critical operational best practice is to maintain a **break-glass administrative account**.

Example concept:

```text
Primary Admin
    │
    └── 2FA enabled

Break-Glass Admin
    │
    └── Emergency access procedure
```

Do not design an environment where **every administrative account depends on the same token infrastructure**.

> Keep emergency access protected, documented, monitored, and used only when necessary.

---

# 16. Lost FortiToken Scenario

If a token is lost:

```text
Lost Token
    ↓
Identify affected user
    ↓
Lock / disable token
    ↓
Verify alternate authentication
    ↓
Assign replacement token
    ↓
Re-provision
    ↓
Test authentication
```

### Important

Before performing token recovery procedures, ensure that you have an appropriate administrative recovery path.

A break-glass administrator is generally safer than relying on undocumented recovery techniques during an outage.

---

# 17. FortiToken Import Errors

Common examples:

```text
Import FortiToken license error: -7571
```

Possible meaning:

```text
Device/token registration problem
```

Another example:

```text
Import FortiToken license error: -7566
```

Possible meaning:

```text
Incorrect token / serial / activation information
```

### Troubleshooting

```text
1. Verify token information
2. Verify activation code
3. Verify token serial
4. Verify FortiGate registration
5. Verify FortiGuard connectivity
6. Verify DNS
7. Verify system time
8. Retry provisioning
```

---

# 18. Trial Token Recovery Considerations

Before attempting supported recovery procedures for trial tokens:

```text
[ ] Identify the tokens
[ ] Remove the affected token assignments where required
[ ] Confirm the management VDOM
[ ] Verify FortiGate registration/connectivity
```

If VDOMs are enabled, token-related management operations may involve the **management VDOM** (commonly `root`, depending on configuration).

---

# 19. SSL-VPN + 2FA Timeout

For remote authentication scenarios, the authentication timeout can be controlled globally.

Example:

```bash
config system global
    set remoteauthtimeout 300
end
```

Example interpretation:

```text
300 seconds
    ↓
5 minutes
```

This controls how long the FortiGate waits for remote authentication in relevant scenarios.

### SSL-VPN Authentication Flow

```text
SSL-VPN Client
      │
      ▼
Username + Password
      │
      ▼
Remote Authentication
      │
      ▼
FortiToken / OTP
      │
      ▼
Authentication Success
      │
      ▼
SSL-VPN Session
```

---

# 20. FortiToken + LDAP

A common enterprise architecture:

```text
                 ┌───────────────┐
                 │ Microsoft AD  │
                 │    / LDAP     │
                 └───────┬───────┘
                         │
                  Username/Password
                         │
                         ▼
┌──────────────┐   ┌───────────────┐
│ FortiToken   │──▶│   FortiGate   │
│ Mobile       │   │               │
└──────────────┘   └───────┬───────┘
                           │
                           ▼
                    Firewall / VPN
```

### Authentication Layers

| Layer           | Function                        |
| --------------- | ------------------------------- |
| LDAP            | Primary identity authentication |
| FortiToken      | Second authentication factor    |
| User Group      | Authorization scope             |
| Firewall Policy | Network access control          |
| SSL-VPN         | Remote access                   |

---

# 21. FortiToken + RADIUS

Another common design:

```text
User
 │
 ▼
FortiGate
 │
 ├── RADIUS ────────► RADIUS/NPS
 │
 └── FortiToken ────► OTP validation
```

This architecture is useful when authentication is centralized through RADIUS.

---

# 22. 2FA for FortiGate Administrators

Recommended structure:

```text
Admin Account
     │
     ├── Strong Password
     │
     ├── FortiToken
     │
     ├── Trusted Management Network
     │
     └── Restricted Admin Profile
```

### Best Practice

Avoid:

```text
Internet
   ↓
FortiGate GUI
   ↓
Super Admin
   ↓
No 2FA
```

Prefer:

```text
Management Network
        ↓
Trusted Admin
        ↓
2FA
        ↓
Least-Privilege Admin Profile
```

---

# 23. Security Design Checklist

### Identity

* [ ] Use centralized authentication where appropriate
* [ ] Use LDAP/RADIUS for enterprise identity
* [ ] Enable 2FA for privileged accounts
* [ ] Avoid shared administrator accounts
* [ ] Maintain a protected break-glass account

### FortiToken

* [ ] Token assigned to correct user
* [ ] Token status is healthy
* [ ] Token is not locked
* [ ] Mobile token is provisioned
* [ ] Activation completed
* [ ] Token expiry is configured

### Infrastructure

* [ ] DNS works
* [ ] NTP works
* [ ] Timezone is correct
* [ ] FortiGuard connectivity works
* [ ] Required Fortinet services are reachable
* [ ] Required firewall policies allow authentication traffic

### SSL-VPN

* [ ] Remote authentication works
* [ ] OTP works
* [ ] Authentication timeout is appropriate
* [ ] User group is correctly mapped
* [ ] VPN policy references the correct user group

---

# 24. Troubleshooting Decision Tree

```text
                 2FA Failure
                     │
          ┌──────────┴──────────┐
          │                     │
      Password fails         OTP fails
          │                     │
          ▼                     ▼
   Check LDAP/RADIUS       Check Token Status
                                │
                  ┌─────────────┼─────────────┐
                  │             │             │
               Locked        Timeout      Wrong OTP
                  │             │             │
                  ▼             ▼             ▼
              Unlock       Re-provision   Check Time
                                              │
                                              ▼
                                           NTP/Drift
                                              │
                                              ▼
                                      Token Synchronize
```

---

# 25. Useful Diagnostic Commands

### FortiToken

```bash
diagnose fortitoken info
```

### Token Database

```bash
show user fortitoken
```

### Token Synchronization

```bash
execute fortitoken sync <token-id> <current-code> <next-code>
```

### FortiGuard / FTM Connectivity

```bash
execute ping fds1.fortinet.com
execute ping directregistration.fortinet.com
execute ping globalftm.fortinet.net
```

### General Debugging

```bash
diagnose debug enable
diagnose debug application authd -1
```

> Use debugging carefully in production and disable it after troubleshooting.

```bash
diagnose debug disable
```

---

# 26. High-Value NSE Exam Notes 🧠

> **Remember these relationships:**

```text
FortiToken
    ↓
OTP / Second Factor

FortiToken Mobile
    ↓
Mobile OTP / Push

Seed
    ↓
Cryptographic secret used by token

Provisioning
    ↓
Assign + Activate Mobile Token

Provision Timeout
    ↓
Re-provision required

Clock Drift
    ↓
OTP validation problems

NTP
    ↓
Critical for time-based OTP

LDAP / RADIUS
    ↓
Primary / Remote Authentication

2FA
    ↓
Authentication ≠ Authorization
```

---

# 27. Production Best Practices

### 🔐 Identity

> **Never treat 2FA as a replacement for authorization.**

Authentication answers:

```text
Who are you?
```

Authorization answers:

```text
What are you allowed to access?
```

A secure design requires both.

### Recommended Architecture

```text
                 Identity Provider
                LDAP / RADIUS / AD
                       │
                       ▼
                 Primary Auth
                       │
                       ▼
                  FortiToken
                       │
                       ▼
                    2FA
                       │
                       ▼
              User / Admin Group
                       │
                       ▼
              Authorization Policy
                       │
                       ▼
                    Resource
```

---

# 28. Quick Reference Table

| Feature                 | Key Point                                                   |
| ----------------------- | ----------------------------------------------------------- |
| FortiToken              | Hardware OTP                                                |
| FortiToken Mobile       | Mobile OTP / push-capable ecosystem                         |
| OTP                     | One-time authentication code                                |
| Seed                    | Token secret                                                |
| Provisioning            | Mobile token activation process                             |
| Provision Timeout       | Token must be re-provisioned                                |
| NTP                     | Important for OTP time synchronization                      |
| LDAP                    | Centralized identity source                                 |
| RADIUS                  | Remote authentication                                       |
| SSL-VPN                 | Common 2FA use case                                         |
| Admin 2FA               | Strongly recommended                                        |
| FTM Push                | Push authentication through supported Fortinet architecture |
| `remoteauthtimeout`     | Remote authentication timeout                               |
| `two-factor-ftm-expire` | FortiToken Mobile provisioning expiry                       |

---

# 29. One-Minute Troubleshooting Checklist

```text
[ ] Is the user authenticated by LDAP/RADIUS?
[ ] Is the FortiToken assigned to the correct user?
[ ] Is the token Active/Provisioned?
[ ] Is the token locked?
[ ] Has provisioning expired?
[ ] Is FortiGate time correct?
[ ] Is NTP working?
[ ] Is the user's mobile time correct?
[ ] Can FortiGate reach Fortinet services?
[ ] Is DNS working?
[ ] Is the correct user group used?
[ ] Is the firewall/VPN policy referencing the correct group?
[ ] Is remote authentication timeout sufficient?
[ ] Can token synchronization resolve OTP drift?
```

---

## 🎯 Core Takeaway

**FortiToken 2FA is not simply “enable OTP.”**

A production-ready implementation requires:

```text
Identity
   +
Authentication
   +
FortiToken Lifecycle
   +
Time Synchronization
   +
Connectivity
   +
Authorization
   +
Recovery Strategy
```

If any one of these components is incorrectly designed, the authentication experience can fail even when the FortiToken itself is healthy.

---

**SheynShield | Engineering Secure Networks**

`FortiGate` • `FortiToken` • `2FA` • `LDAP` • `RADIUS` • `SSL-VPN` • `Identity Security` • `Network Security`

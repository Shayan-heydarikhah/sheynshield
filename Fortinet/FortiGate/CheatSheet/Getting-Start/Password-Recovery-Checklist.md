# FortiGate Password Recovery & Login Security Checklist

> **SheynShield | Engineering Secure Networks**
>
> **FortiGate • FortiOS • Administrator Security • Password Recovery • Maintainer • Admin Lockout • Trusted Hosts • MFA • RBAC • Hardening • NSE 4 • NSE 7**

---

## 🎯 Purpose

Use this checklist to **secure, audit, troubleshoot, and recover FortiGate administrator access**.

It covers:

* Administrator authentication
* Authorization and admin profiles
* Password recovery
* Console recovery
* Maintainer/recovery mechanisms
* Admin lockout
* Trusted hosts
* Password policy
* MFA
* Management-plane hardening
* Break-glass recovery
* NSE 4 fundamentals
* NSE 7 troubleshooting

> ⚠️ **Version warning:** Password-recovery behavior, CLI syntax, defaults, and available security controls can vary by FortiOS release and FortiGate model. Validate recovery procedures against the exact target version before production use.

---

# 🔐 1. Administrator Authentication Checklist

### Authentication

* [ ] Administrator username identified
* [ ] Authentication method documented
* [ ] Strong administrator password configured
* [ ] Individual administrator accounts used
* [ ] Shared administrative credentials avoided
* [ ] MFA enabled where supported
* [ ] Authentication events logged
* [ ] Failed authentication events monitored

### Authentication vs Authorization

```text
Authentication
    =
WHO ARE YOU?

Authorization
    =
WHAT ARE YOU ALLOWED TO DO?
```

* [ ] Authentication and authorization are understood separately
* [ ] Admin profile assigned according to job role
* [ ] Least privilege applied

---

# 👤 2. Administrator Account Checklist

## Default `admin` Account

* [ ] Default administrative account reviewed
* [ ] Initial password changed
* [ ] Password meets organizational requirements
* [ ] Trusted hosts configured
* [ ] MFA evaluated
* [ ] Administrative access limited to management networks
* [ ] Unnecessary administrative access disabled

### Recommended Account Model

```text
                    ADMIN ACCESS
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
        Individual Admin       Break-Glass Admin
              │                     │
              ▼                     ▼
             RBAC              Emergency Use
              │                     │
              ▼                     ▼
           Auditing              Controlled
```

* [ ] Normal operations use individual accounts
* [ ] Break-glass account/process documented
* [ ] Emergency credentials securely controlled

---

# 🔑 3. Password Security Checklist

* [ ] Minimum password length defined
* [ ] Password complexity requirements reviewed
* [ ] Password history/reuse controls reviewed
* [ ] Password expiration policy reviewed
* [ ] Weak/default passwords eliminated
* [ ] Passwords are unique
* [ ] Passwords are not stored in documentation
* [ ] Passwords are not committed to Git
* [ ] Passwords are not included in screenshots
* [ ] Password manager/PAM used where appropriate

### Strong Credential Model

```text
LONG
 +
UNIQUE
 +
UNPREDICTABLE
 +
MFA
```

> Password rotation should not be treated as a substitute for MFA, privileged-access controls, or good credential hygiene.

---

# 🚨 4. Password Recovery Checklist

Password recovery is a **break-glass administrative process**, not a normal login mechanism.

### Before Recovery

* [ ] Administrator identity verified
* [ ] Authorization confirmed
* [ ] Device ownership confirmed
* [ ] FortiGate model identified
* [ ] FortiOS version identified
* [ ] Existing administrator accounts checked
* [ ] Alternative authorized administrator checked
* [ ] Configuration backup located if available
* [ ] Console access available
* [ ] Official Fortinet recovery procedure reviewed

### Recovery Flow

```text
Lost Administrator Password
            ↓
      Verify Authorization
            ↓
      Identify FortiGate
            ↓
      Identify FortiOS Version
            ↓
       Obtain Console Access
            ↓
    Follow Supported Recovery
            ↓
      Set New Admin Password
            ↓
      Validate Management
            ↓
       Audit Configuration
```

---

# 🖥️ 5. Console Access Checklist

* [ ] Console port identified
* [ ] Correct console connection established
* [ ] Console session verified
* [ ] Device physically secured
* [ ] Recovery operator authorized
* [ ] Console credentials/session protected
* [ ] Recovery activity documented

### Important

```text
Network Access
      ≠
Console Recovery Access
```

If normal administrator authentication is unavailable, console-based recovery may be required depending on the platform and FortiOS release.

---

# ⚠️ 6. Maintainer / Recovery Mechanism Checklist

FortiOS releases have historically included special recovery mechanisms for authorized administrator password recovery.

* [ ] Current FortiOS recovery mechanism identified
* [ ] Current FortiGate model behavior verified
* [ ] Historical recovery instructions not assumed to be current
* [ ] Official documentation checked
* [ ] Recovery mechanism tested in a controlled environment where appropriate
* [ ] Recovery process documented internally

### Historical Concept

Some older recovery workflows used a credential derived from the device serial number, for example:

```text
bcpb<DEVICE_SERIAL_NUMBER>
```

> ⚠️ **Do not treat this as a universal current credential.** Recovery behavior has changed across FortiOS releases. Never publish or rely on a historical maintainer format without validating the exact release.

---

# 🔒 7. Disable Maintainer Recovery — Security Review

If the FortiOS version supports disabling the administrator-maintainer recovery mechanism:

* [ ] Business requirement evaluated
* [ ] Threat model evaluated
* [ ] Physical security evaluated
* [ ] Alternative recovery procedure documented
* [ ] Secure configuration backup available
* [ ] Disaster-recovery procedure tested
* [ ] Authorized personnel know the recovery process
* [ ] Factory-reset implications understood

Concept:

```text
Disable Recovery
       │
       ├── Security
       │     ↓
       │  Stronger protection
       │
       └── Recoverability
             ↓
          More difficult
```

### Engineering Rule

```text
Security Control
       +
Recoverability
       +
Business Continuity
       =
Complete Design
```

---

# 🚫 8. Admin Lockout Checklist

Admin lockout helps protect administrative authentication against repeated failed login attempts.

### Verify

* [ ] Lockout mechanism enabled/available
* [ ] Lockout threshold reviewed
* [ ] Lockout duration reviewed
* [ ] Failed login events logged
* [ ] Monitoring configured
* [ ] SOC/NOC knows expected behavior
* [ ] Lockout behavior tested where appropriate

---

## 🔢 Lockout Threshold

```text
admin-lockout-threshold
        =
HOW MANY FAILED ATTEMPTS?
```

Conceptual configuration:

```cli
config system global
    set admin-lockout-threshold <VALUE>
end
```

---

## ⏱️ Lockout Duration

```text
admin-lockout-duration
        =
HOW LONG IS THE LOCKOUT?
```

Conceptual configuration:

```cli
config system global
    set admin-lockout-duration <SECONDS>
end
```

> Verify valid values and defaults against the exact FortiOS release.

---

# 🛡️ 9. Brute-Force Protection Checklist

* [ ] Admin lockout configured
* [ ] Strong passwords configured
* [ ] MFA enabled
* [ ] Trusted hosts configured
* [ ] Management interfaces restricted
* [ ] Administrative services limited
* [ ] Failed-login logging enabled
* [ ] Security monitoring integrated
* [ ] Suspicious login attempts investigated

### Defense-in-Depth

```text
Strong Password
       +
MFA
       +
Trusted Host
       +
Management Network Isolation
       +
Lockout
       +
Logging
```

---

# 🌍 10. Trusted Host Checklist

Trusted hosts restrict administrator login sources.

### Verify

* [ ] Trusted host configured
* [ ] Management subnet identified
* [ ] Jump host/PAM source identified
* [ ] Only required source IPs allowed
* [ ] Public/untrusted source ranges excluded
* [ ] Backup administrator access path documented
* [ ] Trusted host configuration reviewed after IP changes

### Security Model

```text
Administrator
      ↓
Trusted Source IP?
      │
 ┌────┴────┐
YES        NO
 │          │
 ▼          ▼
ALLOW      DENY
```

---

# 🏢 11. Management Network Checklist

Preferred architecture:

```text
                 ADMIN
                   │
                   ▼
             Jump Host / PAM
                   │
                   ▼
            Management VLAN
                   │
                   ▼
               FortiGate
```

* [ ] Dedicated management network available
* [ ] Management VLAN isolated
* [ ] Administrative source networks documented
* [ ] Jump host/PAM considered
* [ ] OOB management considered where appropriate
* [ ] Internet-based management avoided
* [ ] Management traffic monitored

---

# 🚫 12. Internet-Facing Management Checklist

Avoid:

```text
Internet
    ↓
HTTPS / SSH
    ↓
FortiGate Management
```

Prefer:

```text
Administrator
      ↓
Secure Remote Access
      ↓
Management Network
      ↓
FortiGate
```

* [ ] HTTPS exposed only where required
* [ ] SSH exposed only where required
* [ ] HTTP disabled where unnecessary
* [ ] Telnet disabled
* [ ] Source restrictions configured
* [ ] MFA configured where supported
* [ ] Remote administration architecture reviewed
* [ ] Administrative services monitored

---

# 👑 13. Admin Profile / RBAC Checklist

### Authorization

* [ ] Admin profiles reviewed
* [ ] Super-admin access restricted
* [ ] Role-based permissions defined
* [ ] Read-only access used where appropriate
* [ ] Security administrators separated from unrelated roles
* [ ] Network administrators granted only required privileges
* [ ] Unused administrative accounts removed

### Mental Model

```text
Authentication
      ↓
WHO?
      ↓
Administrator
      ↓
Admin Profile
      ↓
WHAT?
      ↓
Permissions
```

### Least Privilege

```text
Give administrators
ONLY
the permissions they need.
```

---

# 🔐 14. MFA Checklist

Where supported and appropriate:

* [ ] MFA enabled for privileged administrators
* [ ] MFA enrollment controlled
* [ ] Recovery process documented
* [ ] Break-glass process documented
* [ ] MFA bypass mechanisms reviewed
* [ ] Emergency access tested
* [ ] MFA events logged
* [ ] Administrative identities centrally managed where appropriate

### Security Model

```text
Password
    +
Second Factor
    +
Trusted Source
    +
Least Privilege
```

---

# 📋 15. Administrator Lifecycle Checklist

## Provisioning

* [ ] Business justification exists
* [ ] Named administrator created
* [ ] Appropriate admin profile assigned
* [ ] Trusted hosts configured
* [ ] MFA enabled
* [ ] Logging enabled

## Review

* [ ] Privileges periodically reviewed
* [ ] Account activity reviewed
* [ ] Trusted hosts reviewed
* [ ] MFA status reviewed
* [ ] Unused accounts identified

## Deprovisioning

* [ ] Former administrator access removed
* [ ] Credentials revoked
* [ ] MFA enrollment removed
* [ ] Trusted-host entries reviewed
* [ ] API/service credentials reviewed where applicable
* [ ] Audit trail retained

---

# 🧪 16. Password Recovery Operational Runbook

### Phase 1 — Identify

* [ ] Device model confirmed
* [ ] FortiOS version confirmed
* [ ] Device serial information available
* [ ] Current administrators identified
* [ ] Recovery authorization confirmed

### Phase 2 — Prepare

* [ ] Console access prepared
* [ ] Configuration backup located
* [ ] Maintenance/change record created
* [ ] Business impact assessed
* [ ] Recovery procedure verified

### Phase 3 — Recover

* [ ] Supported recovery procedure followed
* [ ] Administrator password reset
* [ ] Management access tested
* [ ] Existing configuration preserved where possible

### Phase 4 — Harden

* [ ] Trusted hosts reviewed
* [ ] MFA reviewed
* [ ] Admin profiles reviewed
* [ ] Lockout settings reviewed
* [ ] Management services reviewed
* [ ] Suspicious accounts checked
* [ ] Configuration changes audited

### Phase 5 — Close

* [ ] New credentials securely stored
* [ ] Recovery event documented
* [ ] Security review completed
* [ ] Backup updated if appropriate
* [ ] Incident/change ticket closed

---

# 🔍 17. Admin Lockout Troubleshooting

When an administrator cannot log in:

```text
Admin Login Failure
        ↓
Correct Username?
        │
        ▼
Correct Password?
        │
        ▼
Account Locked?
        │
        ▼
Trusted Host Match?
        │
        ▼
Correct Management Interface?
        │
        ▼
Authentication Source?
        │
        ▼
MFA Successful?
        │
        ▼
Admin Profile Valid?
        │
        ▼
Login Successful
```

### Checklist

* [ ] Username verified
* [ ] Password verified
* [ ] Lockout state considered
* [ ] Source IP checked
* [ ] Trusted host configuration checked
* [ ] Management interface checked
* [ ] Authentication method checked
* [ ] MFA checked
* [ ] Admin profile checked
* [ ] Relevant logs reviewed

---

# 🚨 18. Suspected Administrator Compromise

If administrator credentials may be compromised:

* [ ] Treat the account as compromised
* [ ] Disable/restrict the account where operationally appropriate
* [ ] Rotate credentials
* [ ] Revoke/reconfigure MFA where necessary
* [ ] Review trusted hosts
* [ ] Review recently created administrators
* [ ] Review configuration changes
* [ ] Review login history
* [ ] Review management access logs
* [ ] Review firewall-policy changes
* [ ] Review routing changes
* [ ] Review VPN changes
* [ ] Review security-profile changes
* [ ] Review system/global configuration
* [ ] Preserve relevant logs
* [ ] Escalate according to incident-response procedures

### NSE 7 Thinking

Do not ask only:

```text
"Did the attacker log in?"
```

Also ask:

```text
What changed?
       ↓
Who changed it?
       ↓
When?
       ↓
From where?
       ↓
What configuration objects changed?
       ↓
Can persistence exist?
```

---

# 🧾 19. Configuration Audit Checklist

Review:

### Administrators

* [ ] Known administrators only
* [ ] No unexpected accounts
* [ ] No unnecessary super-admin accounts
* [ ] Profiles follow least privilege

### Authentication

* [ ] Strong authentication
* [ ] MFA
* [ ] Trusted hosts
* [ ] Lockout
* [ ] Password policy

### Management

* [ ] HTTPS
* [ ] SSH
* [ ] HTTP disabled where unnecessary
* [ ] Telnet disabled
* [ ] Management source restrictions

### Recovery

* [ ] Recovery mechanism documented
* [ ] Recovery policy documented
* [ ] Break-glass access secured
* [ ] Configuration backups protected

---

# 🔐 20. GitHub Security Checklist

Before publishing FortiGate security documentation:

* [ ] No real administrator passwords
* [ ] No device serial numbers
* [ ] No API tokens
* [ ] No private keys
* [ ] No private certificates
* [ ] No production IP addresses
* [ ] No VPN secrets
* [ ] No recovery credentials
* [ ] No real customer configuration
* [ ] No sensitive screenshots
* [ ] No backup configuration containing secrets

Use placeholders:

```text
<DEVICE_SERIAL_NUMBER>
<ADMIN_USERNAME>
<STRONG_PASSWORD>
<MANAGEMENT_IP>
<TRUSTED_HOST_IP>
<API_TOKEN>
```

### Git Hygiene

* [ ] Repository scanned for secrets
* [ ] `.gitignore` reviewed
* [ ] Historical commits checked for accidentally exposed credentials
* [ ] Sensitive files removed before publication

---

# 🧠 21. NSE 4 Knowledge Checklist

A FortiGate administrator should be able to explain:

* [ ] Administrator authentication
* [ ] Administrator authorization
* [ ] `admin` account
* [ ] Admin profiles
* [ ] Password policy
* [ ] Password recovery concept
* [ ] Console access
* [ ] Maintainer/recovery concept
* [ ] Admin lockout
* [ ] Lockout threshold
* [ ] Lockout duration
* [ ] Trusted hosts
* [ ] Management protocols
* [ ] MFA concept
* [ ] Least privilege

---

# 🧠 22. NSE 7 Engineering Checklist

An advanced engineer should be able to reason about:

* [ ] Management-plane attack surface
* [ ] Privileged account compromise
* [ ] Break-glass access
* [ ] Recoverability vs security
* [ ] Trusted-host architecture
* [ ] MFA architecture
* [ ] RBAC
* [ ] Authentication troubleshooting
* [ ] Lockout troubleshooting
* [ ] Configuration auditing
* [ ] Administrative logging
* [ ] Incident investigation
* [ ] Secure management architecture
* [ ] Disaster recovery
* [ ] Configuration backup security

---

# 🔥 23. High-Value Exam Traps

### Trap 1 — Authentication ≠ Authorization

```text
Authentication
    =
WHO?

Authorization
    =
WHAT CAN THEY DO?
```

* [ ] Do not confuse credentials with permissions

---

### Trap 2 — Lockout Threshold ≠ Duration

```text
Threshold
    =
HOW MANY?

Duration
    =
HOW LONG?
```

* [ ] Know the difference

---

### Trap 3 — Recovery ≠ Normal Login

```text
Recovery Mechanism
       ≠
Production Authentication
```

* [ ] Treat recovery as break-glass access

---

### Trap 4 — Trusted Host ≠ Firewall Policy

```text
Trusted Host
    =
Administrator Source Restriction
```

```text
Firewall Policy
    =
Traffic Enforcement
```

* [ ] Do not substitute one for the other

---

### Trap 5 — MFA ≠ Least Privilege

```text
MFA
    =
Stronger Authentication
```

```text
RBAC / Admin Profile
    =
Authorization Control
```

* [ ] Use both

---

### Trap 6 — Security ≠ Recoverability

```text
Maximum Security
       +
No Recovery Plan
       =
Operational Risk
```

* [ ] Always design recovery before disabling recovery mechanisms

---

# 🛡️ 24. Complete FortiGate Management Hardening Checklist

## Identity

* [ ] Individual administrator accounts
* [ ] Strong passwords
* [ ] MFA
* [ ] Least privilege
* [ ] RBAC
* [ ] Super-admin access restricted

## Source Restriction

* [ ] Trusted hosts
* [ ] Dedicated management network
* [ ] Jump host/PAM
* [ ] Remote administration controlled

## Services

* [ ] HTTPS required
* [ ] SSH required
* [ ] HTTP disabled where unnecessary
* [ ] Telnet disabled
* [ ] Unnecessary management services disabled

## Authentication Security

* [ ] Admin lockout
* [ ] Password policy
* [ ] Password history
* [ ] Password expiration reviewed
* [ ] MFA recovery documented

## Monitoring

* [ ] Administrative login logging
* [ ] Failed-login monitoring
* [ ] Configuration-change logging
* [ ] Security monitoring
* [ ] Alerting for suspicious administrative activity

## Recovery

* [ ] Console access controlled
* [ ] Recovery procedure documented
* [ ] Break-glass process documented
* [ ] Configuration backup protected
* [ ] Disaster recovery tested

## Physical Security

* [ ] FortiGate physically protected
* [ ] Console access restricted
* [ ] Data-center access controlled
* [ ] Recovery hardware secured

---

# ⚡ 25. 60-Second Revision

```text
              FORTIGATE ADMIN SECURITY
                         │
          ┌──────────────┼──────────────┐
          │              │              │
          ▼              ▼              ▼
   AUTHENTICATION   AUTHORIZATION    RECOVERY
          │              │              │
       Password       Admin Profile   Console
       MFA            RBAC            Break-Glass
       Lockout                        Supported Recovery
       Trusted Host
          │              │              │
          └──────────────┼──────────────┘
                         ▼
                  MANAGEMENT PLANE
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
       HTTPS            SSH          Logging
                         │
                         ▼
                   HARDENING
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
             MFA       RBAC      Monitoring
```

---

# 🏁 Final Mental Model

Think about FortiGate administrator security using **six questions**:

```text
WHO?
 ↓
Authentication

WHAT?
 ↓
Authorization / Admin Profile

WHERE?
 ↓
Trusted Hosts / Management Network

HOW MANY FAILURES?
 ↓
Lockout Threshold

HOW LONG?
 ↓
Lockout Duration

WHAT IF THE PASSWORD IS LOST?
 ↓
Authorized Recovery
```

Then add:

```text
MFA
 +
Least Privilege
 +
Management Isolation
 +
Logging
 +
Secure Recovery
```

### Engineer's Rule

> **Protect the FortiGate management plane as seriously as the data plane.**

A compromised administrator account can allow an attacker to modify the very security controls that are supposed to protect the network.

---

# 🔖 Keywords

`FortiGate Password Recovery`
`FortiGate Admin Password`
`FortiGate Password Recovery Checklist`
`FortiGate Maintainer`
`FortiGate Admin Lockout`
`FortiGate Admin Lockout Threshold`
`FortiGate Admin Lockout Duration`
`FortiGate Trusted Hosts`
`FortiGate Trusted Host`
`FortiGate Administrator Security`
`FortiGate Management Security`
`FortiGate Management Plane Security`
`FortiGate Password Policy`
`FortiGate MFA`
`FortiGate RBAC`
`FortiGate Admin Profile`
`FortiGate Console Recovery`
`FortiGate Hardening Checklist`
`FortiOS Administrator Security`
`Fortinet NSE4`
`Fortinet NSE7`
`FortiGate NSE4`
`FortiGate NSE7`
`FortiGate Troubleshooting`
`Fortinet Firewall Security`
`FortiGate Security Checklist`

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

## 📌 SheynShield Reference

**Topic:** FortiGate Password Recovery & Login Security
**Level:** NSE 4 → NSE 7
**Category:** Administration / Security / Hardening / Troubleshooting
**Format:** GitHub Markdown Checklist
**Brand:** SheynShield — Engineering Secure Networks

# FortiGate FSSO  

### Fortinet Single Sign-On (FSSO) — Authentication, Architecture, Configuration & Troubleshooting

> **FortiGate FSSO  ** — A practical reference for understanding how FortiGate identifies Windows users through Active Directory logon events and uses user/group identity in firewall policies.

---

## 1. What Is FSSO?

**FSSO (Fortinet Single Sign-On)** allows FortiGate to identify users based on their Windows/Active Directory authentication events **without requiring users to manually authenticate to the firewall**.

The general workflow is:

```text
User
  │
  │ Windows Logon
  ▼
Active Directory / Domain Controller
  │
  │ Logon Event
  ▼
FSSO Collector / FSSO Mechanism
  │
  │ User + IP + Groups
  ▼
FortiGate
  │
  │ Identity-based Policy
  ▼
Network Access
```

### Core FSSO workflow

FSSO generally:

1. Detects the user's **logon event**.
2. Records:

   * Username
   * Workstation name
   * Domain
3. Resolves the workstation name to an **IP address**.
4. Determines the **AD groups** associated with the user.
5. Sends user identity information to FortiGate.
6. FortiGate creates an identity/session mapping.
7. Firewall policies can then use the user's **AD/FSSO group** as a source.

---

# 2. FSSO Architecture

```text
                  Active Directory
                  Domain Controller
                         │
                         │ Windows Logon Events
                         ▼
                ┌──────────────────┐
                │   FSSO Collector │
                │      Agent       │
                └────────┬─────────┘
                         │
                         │ User/IP/Group
                         ▼
                ┌──────────────────┐
                │     FortiGate    │
                └────────┬─────────┘
                         │
                         │ Identity Match
                         ▼
                Firewall Policy
```

### Important components

| Component            | Role                                                    |
| -------------------- | ------------------------------------------------------- |
| Active Directory     | Authentication and user/group repository                |
| Domain Controller    | Generates Windows logon events                          |
| FSSO Collector Agent | Collects and forwards user logon information            |
| FortiGate            | Maps identity to IP and applies identity-based policies |

---

# 3. FSSO Deployment Modes

FortiGate environments can use different mechanisms for obtaining user identity.

## Collector Agent Mode

```text
AD / DC
   │
   │ Logon Events
   ▼
FSSO Collector Agent
   │
   │ Push identity
   ▼
FortiGate
```

The Collector Agent monitors domain authentication activity and **pushes user identity information to FortiGate**.

### Advantages

* Near real-time identity updates
* Suitable for Windows/AD environments
* Centralized collection
* Supports AD user/group information

---

## Local Mode

In local mode, FortiGate obtains user information directly rather than relying on the traditional Collector Agent architecture.

Conceptually:

```text
FortiGate
    │
    │ Query / Validate
    ▼
LDAP / AD
```

This model can be useful when the environment requires FortiGate to communicate directly with the user repository.

> **Key distinction:** Collector Agent mode is primarily **event-driven/push-based**, while local/polling-style mechanisms rely more on FortiGate obtaining or checking identity information itself.

---

# 4. FSSO vs DC Agent

| Feature                       | FSSO                                        | DC Agent                      |
| ----------------------------- | ------------------------------------------- | ----------------------------- |
| Main purpose                  | SSO identity integration                    | AD logon monitoring           |
| User repositories             | Can integrate with broader identity sources | Primarily Active Directory    |
| Multi-domain/forest scenarios | More flexible                               | More limited                  |
| Fortinet ecosystem            | Broader integration                         | Mainly FortiGate-focused      |
| Logon information             | Sends identity to FortiGate                 | Collects AD logon information |

### Practical rule

```text
Only AD + straightforward environment
        │
        └──► DC Agent can be sufficient

Multiple domains / forests / broader identity integration
        │
        └──► FSSO is generally more flexible
```

---

# 5. FSSO Configuration — GUI

Navigate to:

```text
Security Fabric
└── External Connectors
    └── FSSO
```

Create an FSSO connector.

Example:

```text
Name:
    fsso-ldap

Primary FSSO Agent:
    192.168.20.200

Key:
    <shared-secret>
```

> **Security:** Never use real production passwords or shared secrets in documentation, GitHub repositories, screenshots, or public labs.

---

# 6. Verify FSSO Collector Connection

After configuration, verify that the Collector Agent connection is shown as:

```text
Connected / Verified
```

If the Collector Agent is not verified, troubleshoot:

* IP connectivity
* TCP port
* Shared key
* DNS
* NTP/time synchronization
* Windows firewall
* FSSO service
* Collector Agent configuration

---

# 7. FSSO CLI Configuration

Example:

```bash
config user fsso
    edit "fsso-ldap"
        set server "192.168.20.200"
        set password <shared-secret>
    next
end
```

If using LDAP integration:

```bash
config user fsso
    edit "fsso-ldap"
        set ldap-server "winsrv-2016"
        set server "192.168.20.200"
        set password <shared-secret>
    next
end
```

> Exact CLI options can vary by FortiOS release. Always verify syntax against the target FortiOS version.

---

# 8. FSSO Communication Ports

Common FSSO-related communication includes:

```text
TCP/8000
UDP/8002
```

The exact ports depend on the FSSO architecture/version and configuration.

### Troubleshooting checklist

```text
FortiGate
   │
   ├── TCP connectivity
   ├── UDP connectivity where applicable
   ├── Shared secret
   ├── FSSO service status
   └── Time synchronization
             │
             ▼
       Collector Agent
             │
             ▼
             Domain Controller
```

---

# 9. FSSO Groups

After configuring FSSO, create a user group:

```text
User & Authentication
└── User Groups
```

Example:

```text
Group Name:
    fsso-group

Remote Server:
    fsso-ldap

Remote Groups:
    Domain users / Security groups
```

The purpose is to make the AD/FSSO identity available to firewall policies.

---

# 10. Identity-Based Firewall Policy

FSSO becomes useful when the identity is referenced inside a firewall policy.

Example:

```text
Policy
├── Incoming Interface
│   └── LAN
├── Outgoing Interface
│   └── WAN / SD-WAN
├── Source
│   └── Network
├── Source User
│   └── FSSO Group
├── Destination
│   └── Internet
├── Service
│   └── HTTP/HTTPS
├── Schedule
│   └── Always
└── Action
    └── Accept
```

### Key concept

```text
IP Address
     +
User Identity
     +
AD Group
     ↓
Identity-Based Firewall Policy
```

Without referencing the FSSO/user group in the appropriate policy, simply configuring FSSO does **not** automatically make every firewall policy identity-aware.

---

# 11. AD Preparation

Before troubleshooting FSSO, verify:

### DNS

FortiGate should be able to resolve the AD infrastructure correctly.

```text
FortiGate
   │
   ▼
Internal DNS
   │
   ▼
Active Directory
```

### Time / NTP

Time synchronization is critical for authentication-related mechanisms.

Verify:

```text
FortiGate time
      ≈
Domain Controller time
```

Check:

* FortiGate NTP
* Domain Controller time
* Time zone
* DNS resolution

---

# 12. Domain Join Considerations

For Windows clients to generate usable domain authentication information:

```text
Client
  │
  ├── DNS → AD DNS
  │
  ├── Authentication → DC
  │
  └── Logon Event → DC
```

Ensure that:

* Clients can reach the Domain Controller.
* DNS points toward the correct AD DNS infrastructure.
* Required AD authentication traffic is permitted.
* Domain membership is healthy.

---

# 13. FSSO Collector Agent Installation

For FortiOS environments using the Collector Agent architecture:

```text
Windows Server
├── FSSO Collector Agent
├── AD integration
└── Domain Controller monitoring
```

When installing the FSSO software:

* Use a dedicated service account where practical.
* Avoid using the built-in/domain Administrator account unnecessarily.
* Grant only the permissions required by the deployment.
* Configure the Collector Agent appropriately for the AD environment.

### Version consideration

For environments using newer FortiOS releases, verify the compatibility between:

```text
FortiOS
   ↕
FSSO version
   ↕
Windows Server / AD
```

Do not blindly deploy an old FSSO package against a newer FortiOS version.

---

# 14. Collector Agent vs Local Identity Retrieval

### Collector Agent

```text
User Login
    ↓
Domain Controller
    ↓
FSSO Collector
    ↓
FortiGate
```

**Push-oriented identity detection**

### Local

```text
User
  ↓
FortiGate
  ↓
Identity Repository
  ↓
Validation / Retrieval
```

**FortiGate-oriented identity retrieval**

### Exam memory

> **Collector Agent = collect and push logon information.**

> **Local = FortiGate performs the identity retrieval/checking mechanism.**

---

# 15. FSSO Security Groups

FSSO is especially useful when policies must be based on AD security groups.

Example:

```text
AD
├── IT
├── HR
├── Finance
└── Guest
```

FortiGate can create different policy behavior:

```text
IT
 └── Full Internet

HR
 └── Business Internet

Finance
 └── Restricted Internet

Guest
 └── Internet Only
```

This eliminates the need to build policies only around static IP addresses.

---

# 16. FSSO Identity Flow

A useful troubleshooting model:

```text
[1] User logs in
        ↓
[2] DC records authentication event
        ↓
[3] FSSO detects event
        ↓
[4] Workstation is resolved to IP
        ↓
[5] User's AD groups are determined
        ↓
[6] Identity information sent to FortiGate
        ↓
[7] FortiGate creates user/IP mapping
        ↓
[8] Firewall policy matches FSSO group
        ↓
[9] Traffic is allowed/denied
```

If identity-based policy is not working, determine **which step failed**.

---

# 17. FSSO Troubleshooting Commands

## Check FSSO server status

```bash
diagnose debug enable
diagnose debug authd fsso server-status
```

Useful for determining whether FortiGate can communicate with the FSSO server/agent.

---

## Display FSSO user information

```bash
diagnose debug authd fsso list
```

Use this to inspect current FSSO identity information.

---

## FSSO Polling Debug

```bash
diagnose debug fsso-polling detail 1
```

Useful when investigating polling/local identity retrieval behavior.

---

## FSSO daemon debug

```bash
diagnose debug application fssod -1
```

Use when deeper FSSO daemon troubleshooting is required.

Disable debugging after troubleshooting:

```bash
diagnose debug disable
```

---

# 18. Recommended Troubleshooting Sequence

Don't immediately start with packet captures.

Use this order:

```text
1. DNS
   ↓
2. NTP / Time
   ↓
3. AD connectivity
   ↓
4. FSSO Agent service
   ↓
5. FortiGate ↔ FSSO connectivity
   ↓
6. Shared secret
   ↓
7. FSSO server status
   ↓
8. FSSO user list
   ↓
9. AD group membership
   ↓
10. Firewall policy
   ↓
11. Session / traffic verification
```

### Why?

Because FSSO problems are frequently **identity-chain problems**, not firewall-policy problems.

---

# 19. Common FSSO Failure Scenarios

| Symptom                                    | Likely Area                                   |
| ------------------------------------------ | --------------------------------------------- |
| FSSO Agent disconnected                    | Connectivity / port / key                     |
| Agent connected but no users               | DC monitoring / logon events                  |
| User detected but wrong IP                 | DNS / workstation resolution                  |
| User detected but wrong group              | AD group membership / synchronization         |
| FSSO user visible but policy doesn't match | Firewall policy                               |
| Random identity changes                    | DHCP / IP reuse / workstation resolution      |
| Authentication works but identity missing  | FSSO event collection                         |
| Intermittent user detection                | Event load / Collector performance            |
| Multiple users behind same IP              | NAT / shared workstation / identity ambiguity |

---

# 20. Important FSSO Limitations

### NTLM

The provided FSSO mechanism does **not support NTLM-based authentication** in the referenced scenario.

Therefore:

```text
Kerberos
   ✅ Supported scenario

NTLM
   ⚠️ Not supported in this FSSO scenario
```

---

## High Login Volume

If a very large number of users authenticate simultaneously, the FSSO daemon may potentially miss some events depending on the deployment.

For environments with high authentication volume:

```text
Large authentication bursts
          ↓
Potential event processing pressure
          ↓
Consider appropriate FSSO Agent architecture
```

---

# 21. Kerberos Event Consideration

The referenced local/polling scenario relies on specific Windows security events.

The notes identify:

```text
4768
4769
```

as supported Kerberos logon-related events for that scenario.

### Concept

```text
User
 ↓
Kerberos Authentication
 ↓
Domain Controller
 ↓
Security Event
 ↓
FSSO Detection
 ↓
FortiGate Identity
```

If these events are not generated/available as expected, identity detection can fail.

---

# 22. FSSO Performance Considerations

For large environments:

```text
                    ┌── DC1
                    │
Users ──► AD/DC ────┼── DC2
                    │
                    └── DC3
                         │
                         ▼
                  FSSO Architecture
                         │
                         ▼
                     FortiGate
```

Consider:

* Number of users
* Number of simultaneous logons
* Number of Domain Controllers
* Number of domains/forests
* Collector placement
* Network latency
* DNS reliability
* Event processing load

---

# 23. Security Best Practices

### Use a dedicated service account

Prefer:

```text
ssoadmin
```

or another dedicated account with the minimum required permissions.

Avoid:

```text
Domain Administrator
```

unless specifically required.

---

### Protect the shared secret

Never publish:

```text
set password "real-password"
```

Use:

```text
set password <FSSO_SHARED_SECRET>
```

in public documentation.

---

### Synchronize time

```text
FortiGate
   │
   ├── NTP
   │
   ▼
Network Time Source
   │
   ▼
AD / Domain Controllers
```

Time inconsistencies can break authentication-related workflows.

---

# 24. FSSO Verification Checklist

```text
[ ] AD is reachable
[ ] Internal DNS is configured correctly
[ ] FortiGate resolves AD/DC names
[ ] NTP/time is synchronized
[ ] FSSO Agent is installed
[ ] FSSO service is running
[ ] FortiGate can reach FSSO Agent
[ ] FSSO shared secret matches
[ ] FSSO server shows Connected/Verified
[ ] User login event is generated
[ ] Workstation resolves to correct IP
[ ] User appears in FSSO list
[ ] Correct AD groups are detected
[ ] FSSO group is created on FortiGate
[ ] Firewall policy references the FSSO group
[ ] Policy order is correct
[ ] Traffic matches the identity policy
[ ] Logs show expected username/group
[ ] Debug is disabled after testing
```

---

# 25. NSE Exam Memory Map

```text
FSSO
│
├── Purpose
│   └── Identify users without manual firewall login
│
├── Identity Source
│   └── AD / Windows authentication
│
├── Collector Agent
│   └── Collect + Push logon information
│
├── Local
│   └── FortiGate retrieves/checks identity
│
├── Identity Data
│   ├── Username
│   ├── IP
│   ├── Domain
│   └── Groups
│
├── Firewall
│   └── Identity-based policies
│
├── Critical Dependencies
│   ├── DNS
│   ├── NTP
│   ├── AD
│   ├── FSSO Agent
│   └── Connectivity
│
└── Troubleshooting
    ├── server-status
    ├── fsso list
    ├── fsso-polling
    └── fssod debug
```

---

# 26. Quick CLI Reference

```bash
# Configure FSSO
config user fsso
    edit "fsso-ldap"
        set server "192.168.20.200"
        set password <shared-secret>
    next
end
```

```bash
# FSSO status
diagnose debug enable
diagnose debug authd fsso server-status
```

```bash
# Show FSSO users
diagnose debug authd fsso list
```

```bash
# Polling debug
diagnose debug fsso-polling detail 1
```

```bash
# FSSO daemon debug
diagnose debug application fssod -1
```

```bash
# Stop debugging
diagnose debug disable
```

---

# 27. Fast Troubleshooting Decision Tree

```text
                 FSSO NOT WORKING
                        │
                        ▼
             Is FSSO Agent connected?
                  /             \
                NO               YES
                │                 │
                ▼                 ▼
        Check network,       Is user visible?
        port, key, DNS          /       \
                              NO         YES
                              │           │
                              ▼           ▼
                         Check AD,     Is correct
                         events,       group shown?
                         polling          /    \
                                        NO      YES
                                        │        │
                                        ▼        ▼
                                   Check AD    Check
                                   groups      firewall
                                               policy
                                                 │
                                                 ▼
                                           Check policy
                                           order/session
```

---

# 28. One-Line Takeaways

> **FSSO = identity-based access without forcing users to authenticate manually to FortiGate.**

> **Collector Agent = detect Windows logon information and push identity toward FortiGate.**

> **FSSO identity = User + IP + Domain + Group information.**

> **DNS + NTP + AD connectivity are foundational dependencies.**

> **Configuring FSSO alone is not enough — the firewall policy must reference the appropriate FSSO/user group.**

> **When troubleshooting, first prove identity visibility on FortiGate, then troubleshoot policy matching.**

---

## 🔎  Keywords

`FortiGate FSSO` · `Fortinet Single Sign-On` · `FortiGate FSSO configuration` · `FortiGate FSSO troubleshooting` · `FortiOS FSSO` · `FortiGate Active Directory authentication` · `FortiGate AD integration` · `FSSO Collector Agent` · `FortiGate identity based firewall policy` · `FortiGate SSO` · `Fortinet FSSO CLI` · `FSSO troubleshooting commands` · `FortiGate LDAP FSSO`

---

## SheynShield Reference

**Topic:** FortiGate FSSO
**Category:** User & Authentication / Identity
**Level:** NSE4 → NSE7
**Use Cases:** AD SSO, Identity-Based Firewall Policies, User/Group-Based Access Control, FSSO Troubleshooting
**Format:**   + Lab Reference + Troubleshooting Guide

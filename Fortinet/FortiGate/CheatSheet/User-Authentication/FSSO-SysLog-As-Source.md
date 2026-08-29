# FortiGate FSSO Using Syslog as Source — Enterprise  

> **FortiOS / FSSO Practical Reference**
> **Topic:** FSSO with Syslog as Authentication Source
> **Focus:** Windows AD → Syslog → FSSO Agent → FortiGate
> **Use Case:** Enterprise environments where installing the FSSO agent directly on the Domain Controller is not desirable.

---

## 1. What Is FSSO with Syslog?

**Fortinet Single Sign-On (FSSO)** allows FortiGate to identify authenticated users and apply identity-based security policies without requiring users to authenticate separately to every resource.

In a **Syslog-based FSSO architecture**, Windows authentication events are exported to a centralized Syslog server. A separate FSSO Agent reads and parses those events, maps the detected username/IP to the corresponding LDAP/AD groups, and forwards the resulting identity information to FortiGate.

### Logical Flow

```text
Windows Domain Controller
        │
        │ Windows Authentication Events
        ▼
   Syslog Collector
        │
        │ Syslog
        ▼
   FSSO Agent
        │
        │ LDAP Group Lookup
        ▼
   Active Directory
        │
        │ User / Group Mapping
        ▼
    FortiGate
        │
        ▼
 Identity-Based Firewall Policy
```

### Enterprise Architecture

```text
                 ┌──────────────────────┐
                 │   Active Directory   │
                 │   DC / LDAP Server    │
                 │   192.168.20.200      │
                 └──────────┬───────────┘
                            │
                     LDAP / AD Lookup
                            │
                            ▼
┌──────────────┐     ┌──────────────┐
│ Windows DC   │────►│ Syslog       │
│ Auth Events  │     │ Server       │
└──────────────┘     │ 192.168.20.1 │
                     └──────┬───────┘
                            │
                         Syslog
                            │
                            ▼
                     ┌──────────────┐
                     │ FSSO Agent   │
                     │ Server-2     │
                     │ 192.168.20.201│
                     └──────┬───────┘
                            │
                         FSSO
                            │
                            ▼
                     ┌──────────────┐
                     │  FortiGate   │
                     │    FGT-2     │
                     └──────────────┘
```

---

# 2. When Should You Use Syslog-Based FSSO?

Syslog-based FSSO is particularly useful when:

* The AD server should not run the FSSO Agent.
* Authentication logs must be centralized.
* An enterprise already has a centralized Syslog/SIEM architecture.
* Multiple Windows systems generate authentication events.
* You need to integrate Windows Event Logs with an external identity-processing system.
* You want the FSSO Agent to run on a separate Windows server.

### Typical Enterprise Design

```text
AD / Domain Controllers
        ↓
Windows Event Logs
        ↓
Syslog
        ↓
Central Syslog Server
        ↓
FSSO Agent
        ↓
LDAP Group Resolution
        ↓
FortiGate
        ↓
Identity-Based Policies
```

---

# 3. FSSO Collector Agent vs Syslog-Based FSSO

Do not confuse the **FSSO Agent architecture** with the **source of authentication events**.

The FSSO Agent can consume authentication information through different mechanisms.

| Architecture                    | Authentication Source                   | Typical Use                                          |
| ------------------------------- | --------------------------------------- | ---------------------------------------------------- |
| Collector Agent / DC monitoring | Windows / AD events                     | Enterprise                                           |
| LDAP polling                    | Active Directory                        | Environments where event monitoring is not preferred |
| Syslog source                   | Centralized Syslog                      | Enterprise / SIEM integration                        |
| Local FSSO                      | FortiGate directly communicates with AD | Smaller environments                                 |

### Key Concept

```text
Authentication Source
        │
        ├── Windows Events
        ├── LDAP
        └── Syslog
                │
                ▼
             FSSO
                │
                ▼
           FortiGate
```

---

# 4. FortiGate — Configure FSSO as Collector Agent

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
    fsso-syslog

Mode:
    Collector Agent

Primary FSSO Agent:
    <FSSO_AGENT_IP>

Key:
    <FSSO_SHARED_KEY>
```

The FortiGate should show the FSSO Agent connection as **verified/connected**.

---

## CLI Example

```bash
config user fsso
    edit "fsso-syslog"
        set server "<FSSO_AGENT_IP>"
        set password "<FSSO_SHARED_KEY>"
    next
end
```

> **Security:** Never publish real FSSO shared keys, LDAP passwords, Syslog secrets, or administrative credentials in a public GitHub repository.

---

# 5. Syslog Server Architecture

The Syslog server acts as the central authentication-event collector.

Example:

```text
Syslog Server
IP: 192.168.20.1
```

Windows authentication events are forwarded to this server.

The FSSO Agent then reads these Syslog messages.

```text
Windows Authentication
        ↓
Windows Event Log
        ↓
Syslog Forwarder
        ↓
192.168.20.1:514
        ↓
FSSO Agent
```

---

# 6. Windows AD → Syslog

The Windows Domain Controller should forward relevant security events to the Syslog infrastructure.

Important Windows authentication events include:

| Event ID | Meaning                         | FSSO Relevance                     |
| -------: | ------------------------------- | ---------------------------------- |
|   `4624` | Successful logon                | ⭐ Critical                         |
|   `4625` | Failed logon                    | Useful for monitoring              |
|   `4634` | Logoff                          | Useful for session tracking        |
|   `4647` | User initiated logoff           | Useful                             |
|   `4776` | NTLM authentication             | Depends on authentication design   |
|   `4768` | Kerberos TGT request            | Useful for Kerberos-based tracking |
|   `4769` | Kerberos service ticket request | Useful                             |

### Most Important Event

```text
4624
Successful Logon
```

Example logical event:

```xml
<Event>
    <System>
        <EventID>4624</EventID>
    </System>

    <EventData>
        <Data Name="TargetUserName">u1</Data>
        <Data Name="TargetDomainName">HQ</Data>
        <Data Name="IpAddress">192.168.1.10</Data>
        <Data Name="LogonType">3</Data>
        <Data Name="WorkstationName">PC-02</Data>
    </EventData>
</Event>
```

The FSSO parsing logic must extract at least:

```text
Username
Client IP
Domain
Group
```

---

# 7. Syslog Transport

A common Syslog listener is:

```text
UDP/514
```

Example:

```text
Syslog Server
192.168.20.1
UDP 514
```

On the FSSO Agent:

```text
Advanced Settings
└── Syslog Source List
    ├── Enable
    └── Listen Port: 514
```

### Verification Checklist

```text
[ ] Syslog service is running
[ ] UDP/514 is reachable
[ ] Windows events are generated
[ ] Windows events arrive at Syslog server
[ ] Syslog messages arrive at FSSO Agent
[ ] FSSO parsing rule matches the message
[ ] Username is extracted
[ ] Client IP is extracted
[ ] Group is extracted
[ ] LDAP lookup succeeds
[ ] FSSO sends identity to FortiGate
```

---

# 8. FSSO Agent — Syslog Source

On the FSSO Agent:

```text
Advanced Settings
└── Syslog Source List
```

Enable:

```text
Syslog Source: Enabled

Listen Port:
    514
```

Add the centralized Syslog source:

```text
Source IP:
    192.168.20.1

Match Rule:
    <SYSLOG_LOGON_RULE>

User Type:
    Remote / LDAP
```

---

# 9. LDAP Configuration on FSSO Agent

The FSSO Agent needs LDAP access to Active Directory so it can resolve the detected username and determine group membership.

Example:

```text
LDAP Server:
    192.168.20.200

LDAP Port:
    389

Base DN:
    DC=test,DC=com

Bind Type:
    Regular

Username:
    ssoadmin

Password:
    <LDAP_PASSWORD>

User Object Class:
    user

Username Attribute:
    sAMAccountName

Group Membership:
    User Attribute

Group Membership Attribute:
    memberOf
```

### LDAP Mapping

```text
Syslog
  │
  ├── username = u1
  └── client_ip = 192.168.20.20
          │
          ▼
       LDAP Query
          │
          ▼
       AD User
          │
          ▼
      memberOf
          │
          ▼
      itgroup
          │
          ▼
       FSSO Mapping
          │
          ▼
FortiGate Identity
```

---

# 10. Syslog Parsing Rules

The most important part of this architecture is the **mapping between raw Syslog messages and FSSO identity fields**.

A parsing rule should extract:

```text
Username
Client IPv4
Group
```

Example conceptual rule:

```text
Logon:
    status="logon"

Username:
    user="{{:username}}"

Client IPv4:
    srcip={{:client_ip=([0-9.]+)}}

Group:
    group="{{:group}}"
```

> The exact parser syntax depends on the FSSO Agent version and the format of the Syslog message. Always validate the generated rule against the actual Syslog payload.

---

# 11. Test the Parsing Rule

Do not create the rule only from assumptions.

First capture a real authentication event.

Example:

```text
User:
    u1

Client IP:
    192.168.101.2

Group:
    itgroup
```

Example normalized message:

```text
type="custom_logon"
custom_user="u1"
custom_ip="192.168.101.2"
custom_group="itgroup"
```

Then test the parser.

### Expected Result

```text
Username:
    u1

IP:
    192.168.101.2

Group:
    itgroup
```

### Rule Validation Checklist

```text
[ ] Username detected correctly
[ ] IP detected correctly
[ ] Domain detected correctly
[ ] Group detected correctly
[ ] No extra spaces break parsing
[ ] Multiple users can be parsed
[ ] Multiple IP addresses do not cause false matches
[ ] Logoff events are handled correctly
```

---

# 12. Local Windows Syslog Forwarding

If the Windows environment does not already have a Syslog forwarder, a Windows-to-Syslog utility can be used.

Possible approaches include:

```text
Evtsys
NXLog
Winlogbeat
Windows Event Forwarding
```

Example concept:

```text
Windows Security Event
        ↓
Event ID 4624
        ↓
Windows Syslog Forwarder
        ↓
Syslog Server
        ↓
FSSO Agent
```

---

# 13. Event ID 4624 — The Critical Event

For successful Windows authentication, focus heavily on:

```text
Event ID:
    4624
```

Useful fields:

```text
TargetUserName
TargetDomainName
IpAddress
LogonType
WorkstationName
```

Example:

```text
TargetUserName:
    u1

TargetDomainName:
    HQ

IpAddress:
    192.168.1.10

LogonType:
    3

WorkstationName:
    PC-02
```

### Why This Matters

FSSO ultimately needs to answer:

> **Which user is currently associated with which client IP?**

```text
192.168.1.10
      ↓
     u1
      ↓
   itgroup
      ↓
FortiGate Policy
      ↓
Internet Access
```

---

# 14. FSSO User Group on FortiGate

After the FSSO connector is working, create a user group.

Navigate to:

```text
User & Authentication
└── User Groups
```

Example:

```text
Group Name:
    fsso-group

Remote Server:
    fsso-syslog

Remote Groups:
    itgroup
```

Conceptually:

```text
AD
 │
 └── itgroup
       │
       ▼
   FSSO Agent
       │
       ▼
 fsso-syslog
       │
       ▼
 FortiGate User Group
       │
       ▼
 Firewall Policy
```

---

# 15. Identity-Based Firewall Policy

FSSO becomes useful when the identity information is consumed by a firewall policy.

Example:

```text
Incoming Interface:
    LAN

Outgoing Interface:
    SD-WAN

Source:
    LAN_SUBNET

Source User:
    fsso-group

Destination:
    Internet

Service:
    Web Access

Schedule:
    Always

NAT:
    Enable
```

### Critical Point

The firewall policy must actually reference the FSSO user/group.

```text
Authentication Source
        ↓
FSSO Group
        ↓
Firewall Policy
        ↓
Identity-Based Access
```

Simply configuring FSSO does **not** automatically mean every firewall policy will use user identity.

---

# 16. FSSO Timeout

FSSO maintains a relationship between the user and client IP.

Example:

```bash
config user fsso
    edit "fsso-syslog"
        set logon-timeout 5
    next
end
```

Where:

```text
5 = minutes
```

### Design Consideration

A timeout that is too short can cause:

```text
User
 ↓
Identity expires
 ↓
FortiGate no longer sees expected identity
 ↓
Policy matching changes
```

A timeout that is too long can leave stale identity information.

Choose the value according to the authentication and session behavior of the environment.

---

# 17. FSSO Collector Agent vs Local FSSO

| Feature                                 | FSSO Collector Agent | Local FSSO              |
| --------------------------------------- | -------------------- | ----------------------- |
| Separate Windows Agent                  | ✅                    | ❌                       |
| FortiGate directly communicates with AD | Not necessarily      | ✅                       |
| Multiple DCs                            | Better suited        | More limited            |
| Multiple domains                        | Better suited        | More limited            |
| Enterprise deployment                   | ⭐ Recommended        | Usually less suitable   |
| Additional Windows component            | Required             | Not required            |
| Centralized event processing            | ✅                    | Limited                 |
| Syslog integration                      | ✅                    | Depends on architecture |

### Enterprise Recommendation

Use the **FSSO Collector Agent architecture** when:

```text
Multiple Domain Controllers
        OR
Multiple AD Domains
        OR
Large Enterprise
        OR
Centralized Authentication Logging
        OR
High Reliability Requirements
```

Use Local FSSO when:

```text
Small Environment
+
Simple AD Architecture
+
No Agent Installation Requirement
```

---

# 18. Why Syslog-Based FSSO?

### Advantages

* Centralized authentication event collection
* Separates FSSO processing from Domain Controllers
* Integrates naturally with enterprise Syslog/SIEM architecture
* Can process authentication events from multiple sources
* Useful when direct FSSO Agent deployment on a DC is undesirable

### Disadvantages

* More components
* More troubleshooting points
* Requires reliable Syslog delivery
* Parser/rule design becomes critical
* Incorrect log formatting can break identity detection
* Additional latency may exist between authentication and identity availability

---

# 19. Troubleshooting Workflow

When FSSO does not detect a user, **do not start with FortiGate policy debugging**.

Follow the chain from the source.

```text
1. Did Windows generate the authentication event?
                    ↓
2. Did the Syslog forwarder send it?
                    ↓
3. Did the Syslog server receive it?
                    ↓
4. Did the FSSO Agent receive it?
                    ↓
5. Did the parser extract the username?
                    ↓
6. Did the parser extract the client IP?
                    ↓
7. Did LDAP resolve the user?
                    ↓
8. Did LDAP return group membership?
                    ↓
9. Did FSSO send the identity to FortiGate?
                    ↓
10. Does the firewall policy reference the FSSO group?
```

---

# 20. FortiGate Debug Commands

### Enable Debug

```bash
diagnose debug enable
```

### FSSO Server Status

```bash
diagnose debug application authd -1
```

### FSSO Daemon Debug

```bash
diagnose debug application fssod -1
```

### Authentication Information

```bash
diagnose firewall auth list
```

This is useful for checking currently recognized authenticated users.

---

# 21. FSSO Verification Checklist

## FortiGate

```text
[ ] FSSO connector created
[ ] Collector Agent mode configured
[ ] FSSO Agent IP reachable
[ ] Shared key matches
[ ] Connection shows verified/connected
[ ] FSSO user group created
[ ] Correct remote group selected
[ ] Firewall policy references FSSO group
```

## Active Directory

```text
[ ] LDAP service available
[ ] LDAP port reachable
[ ] Bind account valid
[ ] Base DN correct
[ ] Username attribute correct
[ ] Group membership attribute correct
[ ] Required authentication events generated
```

## Syslog

```text
[ ] Syslog server running
[ ] UDP/514 reachable
[ ] Windows logs arrive
[ ] Event ID 4624 exists
[ ] Source IP is present
[ ] Username is present
[ ] Domain is present
```

## FSSO Agent

```text
[ ] Syslog source enabled
[ ] Listen port configured
[ ] Syslog source IP configured
[ ] Parsing rule created
[ ] Parsing rule tested
[ ] Username extracted
[ ] Client IP extracted
[ ] Group extracted
[ ] LDAP lookup succeeds
[ ] User mapping created
```

---

# 22. Common Failure Points

| Problem                                      | Likely Cause                               |
| -------------------------------------------- | ------------------------------------------ |
| No FSSO users                                | Syslog events not reaching FSSO            |
| User detected but no group                   | LDAP/group mapping problem                 |
| Wrong client IP                              | Incorrect Syslog parsing rule              |
| Authentication event exists but user missing | Username field parser mismatch             |
| FSSO connected but no identity               | Source/event parsing issue                 |
| User identity disappears quickly             | FSSO timeout too short                     |
| Policy does not match user                   | FSSO group not referenced by policy        |
| Multiple users map incorrectly               | IP/username parsing ambiguity              |
| Syslog works but FSSO fails                  | Parser or LDAP lookup problem              |
| LDAP authentication fails                    | Base DN / bind credentials / connectivity  |
| Intermittent identity                        | Event loss, Syslog reliability, or timeout |

---

# 23. High-Level Troubleshooting Model

```text
                 USER LOGIN
                     │
                     ▼
              Windows AD Event
                     │
                     ▼
                Event ID 4624
                     │
                     ▼
              Syslog Forwarder
                     │
                     ▼
             Central Syslog
                     │
                     ▼
               FSSO Agent
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
     Parse Username          Parse IP
          │                     │
          └──────────┬──────────┘
                     ▼
                LDAP Lookup
                     │
                     ▼
               Group Mapping
                     │
                     ▼
              FSSO Identity
                     │
                     ▼
                 FortiGate
                     │
                     ▼
           Identity-Based Policy
                     │
                     ▼
                ALLOW / DENY
```

---

# 24. Enterprise Design — Recommended Separation

A clean enterprise architecture separates these functions:

```text
                ┌─────────────────┐
                │ Active Directory│
                └────────┬────────┘
                         │
                  Authentication
                         │
                         ▼
                ┌─────────────────┐
                │ Windows Events  │
                └────────┬────────┘
                         │
                         ▼
                ┌─────────────────┐
                │ Syslog Platform │
                └────────┬────────┘
                         │
                         ▼
                ┌─────────────────┐
                │   FSSO Agent    │
                └────────┬────────┘
                         │
                    Identity
                         │
                         ▼
                ┌─────────────────┐
                │    FortiGate    │
                └────────┬────────┘
                         │
                         ▼
              Identity-Based Security
```

This provides a clean separation between:

```text
Authentication
      +
Log Collection
      +
Identity Resolution
      +
Security Enforcement
```

---

# 25. Security Best Practices

### Protect the FSSO Shared Key

Never expose:

```text
FSSO password
FSSO shared key
LDAP password
Syslog secret
Administrator password
```

Use placeholders in documentation:

```text
<FSSO_SHARED_KEY>
<LDAP_PASSWORD>
<SYSLOG_SECRET>
```

### Use a Dedicated LDAP Service Account

Prefer:

```text
ssoadmin
```

or another dedicated least-privilege account rather than the domain administrator.

### Protect LDAP

Where supported by the environment, prefer secure LDAP rather than transmitting credentials over unencrypted LDAP.

### Restrict Syslog Sources

Only trusted systems should be able to send authentication events to the Syslog listener.

```text
Allowed Sources
      │
      ▼
Syslog Server
      │
      ▼
FSSO Agent
```

### Synchronize Time

FSSO troubleshooting becomes significantly harder when:

```text
AD Time
≠
Syslog Time
≠
FSSO Agent Time
≠
FortiGate Time
```

Verify:

```text
NTP
Timezone
System Time
```

across all components.

---

# 26. Switch / Network Considerations

If the architecture includes switches between the servers and FortiGate:

```text
[ ] Verify VLAN configuration
[ ] Verify routing
[ ] Verify gateway
[ ] Verify DNS
[ ] Verify NTP
[ ] Verify Syslog connectivity
[ ] Verify TCP/UDP requirements
[ ] Verify firewall rules
```

For server-facing switch ports, use an appropriate edge/PortFast configuration according to the switching platform and topology.

> Do not blindly enable PortFast on links that can form Layer-2 loops.

---

# 27. Quick CLI Reference

### FSSO Configuration

```bash
config user fsso
    edit "fsso-syslog"
        set server "<FSSO_AGENT_IP>"
        set password "<FSSO_SHARED_KEY>"
        set logon-timeout 5
    next
end
```

### Debug

```bash
diagnose debug enable

diagnose debug application authd -1

diagnose debug application fssod -1

diagnose firewall auth list
```

---

# 28. FSSO Syslog Deployment Checklist

```text
PHASE 1 — AD
[ ] Verify AD
[ ] Verify LDAP
[ ] Verify authentication events
[ ] Verify Event ID 4624

PHASE 2 — SYSLOG
[ ] Deploy Syslog server
[ ] Configure Windows log forwarding
[ ] Verify UDP/514
[ ] Verify received events

PHASE 3 — FSSO AGENT
[ ] Install FSSO Agent
[ ] Configure Syslog source
[ ] Configure LDAP
[ ] Configure parsing rules
[ ] Test username extraction
[ ] Test IP extraction
[ ] Test group extraction

PHASE 4 — FORTIGATE
[ ] Configure FSSO connector
[ ] Verify Agent connection
[ ] Create FSSO user group
[ ] Map remote AD group
[ ] Create identity-based policy

PHASE 5 — VALIDATION
[ ] Login as test user
[ ] Verify Event ID 4624
[ ] Verify Syslog message
[ ] Verify FSSO mapping
[ ] Verify FortiGate user
[ ] Verify firewall policy hit
[ ] Verify internet/resource access

PHASE 6 — HARDENING
[ ] Protect shared keys
[ ] Protect LDAP credentials
[ ] Restrict Syslog sources
[ ] Verify NTP
[ ] Review FSSO timeout
[ ] Enable appropriate logging
```

---

# 29. Interview / NSE Exam Quick Hits

> ### Remember These

**FSSO = Identity → IP → Group → Policy**

```text
User Login
   ↓
Authentication Event
   ↓
FSSO
   ↓
Username + IP + Groups
   ↓
FortiGate
   ↓
Identity-Based Policy
```

### Syslog FSSO

```text
Windows Event
    ↓
Syslog
    ↓
FSSO Agent
    ↓
LDAP Group Resolution
    ↓
FortiGate
```

### Critical Event

```text
4624 = Successful Logon
```

### Critical Syslog Port

```text
UDP/514
```

### Critical LDAP Information

```text
Base DN
Bind Account
Username Attribute
Group Membership Attribute
```

### Critical FSSO Debug

```bash
diagnose debug application authd -1
diagnose debug application fssod -1
diagnose firewall auth list
```

### Critical Troubleshooting Rule

> **Trace the identity from the source to the policy.**

```text
AD
 ↓
Event
 ↓
Syslog
 ↓
FSSO
 ↓
LDAP
 ↓
Group
 ↓
FortiGate
 ↓
Policy
```

---

# 30. Final Mental Model

The most important concept is that **FSSO is not simply an authentication server**.

It is an **identity correlation mechanism**.

FSSO correlates:

```text
WHO?
  ↓
Username

WHERE?
  ↓
Client IP

WHICH GROUP?
  ↓
AD / LDAP Group

WHEN?
  ↓
Logon / Timeout

WHAT POLICY?
  ↓
FortiGate Firewall Policy
```

Therefore:

```text
Authentication Event
        +
Client IP
        +
LDAP Group
        ↓
      FSSO
        ↓
FortiGate Identity
        ↓
Security Policy
```

That is the core architecture behind **FortiGate FSSO with Syslog as the authentication-event source**.

---

## 🔎  Keywords

`FortiGate FSSO`, `Fortinet FSSO`, `FSSO Syslog`, `FortiGate Syslog FSSO`, `FortiGate Active Directory SSO`, `FortiGate LDAP FSSO`, `FSSO Collector Agent`, `FortiGate identity based policy`, `Windows Event ID 4624`, `FortiGate AD authentication`, `FortiOS FSSO`, `FortiGate single sign-on`, `Fortinet Active Directory integration`, `FSSO troubleshooting`, `FortiGate enterprise authentication`

---

## 📌 SheynShield   Series

**Category:** FortiGate → User & Authentication → FSSO

**Related topics:**

* FSSO Collector Agent
* FSSO Local Mode
* FSSO LDAP Polling
* FSSO Syslog Integration
* LDAP Authentication
* RADIUS Authentication
* TACACS+
* Identity-Based Firewall Policies
* FSSO Troubleshooting
* FortiGate User Authentication

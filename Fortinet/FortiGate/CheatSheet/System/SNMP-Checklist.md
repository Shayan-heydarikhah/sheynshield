# 🔗 SheynShield Resources

# FortiGate SNMP Checklist

> **FortiOS | SNMP Monitoring | SNMPv1 | SNMPv2c | SNMPv3 | MIB | OID | MIB Views | Traps | HA | DHCP Events | Troubleshooting**
>
> **SheynShield | Engineering Secure Networks**

---

## 📋 Table of Contents

* [1. SNMP Architecture Checklist](#1-snmp-architecture-checklist)
* [2. Interface SNMP Access](#2-interface-snmp-access)
* [3. MIB Checklist](#3-mib-checklist)
* [4. RFC 1213 MIB-II](#4-rfc-1213-mib-ii)
* [5. RFC 2665 Ethernet-like MIB](#5-rfc-2665-ethernet-like-mib)
* [6. SNMP Version Selection](#6-snmp-version-selection)
* [7. SNMPv1/v2c Configuration](#7-snmpv1v2c-configuration)
* [8. SNMPv3 Configuration](#8-snmpv3-configuration)
* [9. SNMPv3 Security Levels](#9-snmpv3-security-levels)
* [10. Authentication & Privacy](#10-authentication--privacy)
* [11. MIB Views](#11-mib-views)
* [12. MIB View Include/Exclude](#12-mib-view-includeexclude)
* [13. MIB View + Community](#13-mib-view--community)
* [14. MIB View + SNMPv3](#14-mib-view--snmpv3)
* [15. SNMP + HA](#15-snmp--ha)
* [16. DHCP SNMP Events](#16-dhcp-snmp-events)
* [17. Query vs Trap](#17-query-vs-trap)
* [18. SNMP Ports](#18-snmp-ports)
* [19. Production Security Checklist](#19-production-security-checklist)
* [20. Troubleshooting Checklist](#20-troubleshooting-checklist)
* [21. Quick Configuration Card](#21-quick-configuration-card)
* [22. NSE Exam Traps](#22-nse-exam-traps)
* [23. SNMP Mental Model](#23-snmp-mental-model)
* [24. Final Memory Map](#24-final-memory-map)

---

# 1. SNMP Architecture Checklist

### Core Concept

**SNMP (Simple Network Management Protocol)** allows an external **SNMP Manager / NMS** to monitor FortiGate and receive event notifications.

### Monitoring Checklist

* [ ] CPU utilization
* [ ] Memory utilization
* [ ] Interface status
* [ ] Interface bandwidth
* [ ] Packet counters
* [ ] System health
* [ ] Temperature
* [ ] HA status
* [ ] DHCP events
* [ ] Security-related events
* [ ] FortiGate-specific statistics

### Basic Architecture

```text
                ┌─────────────────────┐
                │     SNMP Manager    │
                │        / NMS        │
                └──────────┬──────────┘
                           │
                ┌──────────┴──────────┐
                │                     │
              QUERY                  TRAP
                │                     │
                ▼                     ▲
         ┌────────────────────────────────┐
         │           FortiGate            │
         │          SNMP Agent             │
         └────────────────────────────────┘
```

### Remember

| Operation        | Direction       | Initiator | Purpose              |
| ---------------- | --------------- | --------- | -------------------- |
| **Query / Poll** | NMS → FortiGate | NMS       | Retrieve information |
| **Trap**         | FortiGate → NMS | FortiGate | Report an event      |

* [ ] Query = NMS asks
* [ ] Trap = FortiGate tells

---

# 2. Interface SNMP Access

Creating an SNMP community or SNMPv3 user **does not automatically enable SNMP access on the interface**.

The relevant interface must permit SNMP administrative access.

### Checklist

* [ ] Identify the interface used by the NMS.
* [ ] Enable `snmp` under `allowaccess`.
* [ ] Confirm routing between NMS and FortiGate.
* [ ] Confirm UDP/161 reachability.
* [ ] Confirm UDP/162 reachability if traps are used.

### Configuration

```cli
config system interface
    edit "port1"
        set allowaccess ping https ssh snmp
    next
end
```

### Minimal Example

```cli
config system interface
    edit "port1"
        set allowaccess snmp
    next
end
```

> [!IMPORTANT]
> **SNMP configuration and interface `allowaccess snmp` are separate configuration requirements.**

---

# 3. MIB Checklist

## What is MIB?

**MIB — Management Information Base** defines the objects and OIDs that an SNMP Manager can interpret.

### Fortinet MIBs

* [ ] `fortinet-core-mib.mib`
* [ ] `fortinet-fortigate-mib.mib`

### Fortinet Core MIB

```text
fortinet-core-mib.mib
```

Contains common Fortinet information such as:

* [ ] Common system information
* [ ] Common configuration information
* [ ] Common traps

### FortiGate MIB

```text
fortinet-fortigate-mib.mib
```

Contains FortiGate-specific:

* [ ] System information
* [ ] Configuration objects
* [ ] Traps
* [ ] FortiGate-specific monitoring information

### Memory Trick

```text
Fortinet Core MIB
        ↓
Common Fortinet information

FortiGate MIB
        ↓
FortiGate-specific information
```

> [!TIP]
> When generic MIB-II information is insufficient, prefer the **Fortinet-specific MIB**.

---

# 4. RFC 1213 MIB-II

FortiGate supports **MIB-II / RFC 1213** with exceptions.

### Unsupported Group

* [ ] EGP group → **Not supported**

```text
RFC 1213
└── MIB-II
    └── EGP → ❌ Not supported
```

### Accuracy Consideration

Generic MIB-II protocol statistics may not accurately represent all Fortinet traffic activity.

Therefore:

* [ ] Use MIB-II for generic monitoring.
* [ ] Use Fortinet MIB for FortiGate-specific statistics.
* [ ] Do not assume generic counters represent every Fortinet feature accurately.

> [!WARNING]
> **MIB-II is not always an exact representation of FortiGate-specific traffic statistics.**

---

# 5. RFC 2665 Ethernet-like MIB

FortiGate supports Ethernet-like MIB information with exceptions.

### Unsupported Groups

* [ ] `dot3Tests` → Not supported
* [ ] `dot3Errors` → Not supported

```text
RFC 2665
│
├── Ethernet-like MIB
│       └── Supported
│
├── dot3Tests
│       └── ❌ Not supported
│
└── dot3Errors
        └── ❌ Not supported
```

---

# 6. SNMP Version Selection

| Feature              | SNMPv1 | SNMPv2c | SNMPv3 |
| -------------------- | -----: | ------: | -----: |
| Community-based      |      ✅ |       ✅ |      ❌ |
| Authentication       |      ❌ |       ❌ |      ✅ |
| Encryption / Privacy |      ❌ |       ❌ |      ✅ |
| Queries              |      ✅ |       ✅ |      ✅ |
| Traps                |      ✅ |       ✅ |      ✅ |
| Security level       |    Low |     Low |   High |

### Production Recommendation

```text
SNMPv3
  |
  └── auth-priv
        ├── Authentication
        └── Privacy / Encryption
```

### Selection Checklist

* [ ] Legacy environment → v1/v2c may be required.
* [ ] Modern environment → prefer v3.
* [ ] Sensitive management network → use v3.
* [ ] Need encryption → use v3.
* [ ] Need authentication + encryption → use `auth-priv`.

> [!IMPORTANT]
> **SNMPv1/v2c use community strings. SNMPv3 uses users with security levels.**

---

# 7. SNMPv1/v2c Configuration

Configuration:

```cli
config system snmp community
```

### Example

```cli
config system snmp community
    edit 2
        set name "pub-com"
        set status enable

        config hosts
            edit "fgt-1"
                set ip 192.168.254.253
                set source-ip <SOURCE-IP>
                set ha-direct enable
                set host-type any
            next
        end

        set query-v1-port <PORT>
        set query-v1-status enable

        set query-v2c-port <PORT>
        set query-v2c-status enable

        set trap-v1-lport <PORT>
        set trap-v1-rport <PORT>
        set trap-v1-status enable

        set trap-v2c-lport <PORT>
        set trap-v2c-rport <PORT>
        set trap-v2c-status enable

        set events <EVENTS>
    next
end
```

### Validation Checklist

* [ ] Community exists.
* [ ] Community is enabled.
* [ ] NMS IP is configured.
* [ ] Query status is enabled.
* [ ] Trap status is enabled if required.
* [ ] Correct ports are configured.
* [ ] Correct events are selected.
* [ ] Host type matches the requirement.

---

# 8. SNMPv3 Configuration

Configuration:

```cli
config system snmp user
```

### Example

```cli
config system snmp user
    edit "monitor"
        set status enable
        set trap-status enable

        set queries enable
        set query-port <PORT>

        set trap-lport <PORT>
        set trap-rport <PORT>

        set notify-hosts 192.168.254.253

        set source-ip <SOURCE-IP>
        set ha-direct enable

        set events <EVENTS>

        set security-level auth-priv

        set auth-proto sha256
        set auth-pwd <AUTH_PASSWORD>

        set priv-proto aes
        set priv-pwd <PRIV_PASSWORD>
    next
end
```

### Deployment Checklist

* [ ] SNMPv3 user created.
* [ ] User enabled.
* [ ] Queries enabled if polling is required.
* [ ] Traps enabled if event notification is required.
* [ ] NMS/notification host configured.
* [ ] Authentication protocol configured.
* [ ] Authentication password configured.
* [ ] Privacy protocol configured.
* [ ] Privacy password configured.
* [ ] Security level configured.
* [ ] Source IP configured where required.

> [!WARNING]
> Never commit real SNMP authentication or privacy passwords to GitHub.

---

# 9. SNMPv3 Security Levels

SNMPv3 provides three security levels.

| Security Level    | Authentication | Encryption |
| ----------------- | -------------: | ---------: |
| `no-auth-no-priv` |              ❌ |          ❌ |
| `auth-no-priv`    |              ✅ |          ❌ |
| `auth-priv`       |              ✅ |          ✅ |

### Security Model

```text
no-auth-no-priv
    |
    └── No authentication
        No encryption

auth-no-priv
    |
    └── Authentication
        No encryption

auth-priv ⭐
    |
    ├── Authentication
    └── Encryption / Privacy
```

### Production Checklist

* [ ] Avoid `no-auth-no-priv` unless there is a specific legacy requirement.
* [ ] Use `auth-no-priv` only when encryption is not required.
* [ ] Prefer `auth-priv` for production monitoring.

> [!IMPORTANT]
> **`auth-priv` = Authentication + Privacy.**

---

# 10. Authentication & Privacy

## Authentication Protocols

Example:

```cli
set auth-proto sha256
```

Possible algorithms can include:

```text
MD5
SHA
SHA224
SHA256
SHA384
SHA512
```

### Checklist

* [ ] Choose an algorithm supported by the FortiOS version.
* [ ] Confirm NMS compatibility.
* [ ] Prefer stronger algorithms where supported.

---

## Privacy Protocols

Example:

```cli
set priv-proto aes
```

Possible options can include:

```text
DES
AES
AES256
AES256CISCO
```

### Secure SNMPv3 Model

```text
SNMPv3
   |
   ├── Username
   |
   ├── Authentication
   │      └── SHA256
   |
   └── Privacy
          └── AES
```

---

# 11. MIB Views

A **MIB View** controls which parts of the MIB/OID tree an SNMP identity can access.

### Why Use MIB Views?

* [ ] Least privilege
* [ ] Limit sensitive information
* [ ] Restrict monitoring scope
* [ ] Separate monitoring teams
* [ ] Reduce unnecessary MIB exposure
* [ ] Build role-based monitoring access

Configuration:

```cli
config system snmp mib-view
```

### Security Model

```text
SNMP Identity
      |
      ├── Authentication
      ├── Privacy
      ├── VDOM restriction
      │
      └── MIB View
             |
             ▼
        Allowed OIDs
```

> [!TIP]
> MIB views are an SNMP equivalent of **least-privilege access control for the MIB tree**.

---

# 12. MIB View Include/Exclude

### Example

```cli
config system snmp mib-view
    edit "monitor-view"
        set include 1.3.6.1.2.1
        set exclude 1.3.6.1.2.1.1.9.1
    next
end
```

### Concept

```text
MIB Tree
   |
   +── INCLUDE
   |      |
   |      └── Accessible
   |
   └── EXCLUDE
          |
          └── Blocked
```

### Advanced Example

```cli
config system snmp mib-view
    edit "view2"
        set include 1.3.6.1.2.1

        set exclude \
            1.3.6.1.2.1.2.1 \
            1.3.6.1.2.1.4.31 \
            1.3.6.1.2.1.1.9.1
    next
end
```

### Checklist

* [ ] Define the root OID to expose.
* [ ] Include required OID branches.
* [ ] Exclude unnecessary branches.
* [ ] Test the final accessible OID tree.
* [ ] Confirm NMS dashboards still receive required objects.

---

# 13. MIB View + Community

A MIB view can be assigned to an SNMP community.

```cli
config system snmp community

    edit 1
        set name "monitor-full"
        set vdoms "vdom1"
    next

    edit 2
        set name "monitor-limited"
        set mib-view "view2"
    next

    edit 3
        set name "monitor-vdom"
        set mib-view "view1"
        set vdoms root vdom1
    next

end
```

### Checklist

* [ ] Community configured.
* [ ] Correct MIB view assigned.
* [ ] Correct VDOM scope assigned.
* [ ] NMS can query allowed OIDs.
* [ ] Restricted OIDs are inaccessible.

---

# 14. MIB View + SNMPv3

MIB views can also be assigned to SNMPv3 users.

```cli
config system snmp user

    edit "v3user"
        set mib-view "view1"
    next

    edit "v3user-vdom"
        set vdom "vdom1"
    next

    edit "v3user-restricted"
        set mib-view "view1"
        set vdoms root vdom1
    next

end
```

### Security Stack

```text
SNMPv3 User
     |
     ├── Authentication
     |
     ├── Privacy
     |
     ├── VDOM restriction
     |
     └── MIB View
            |
            ▼
       Allowed OIDs
```

---

# 15. SNMP + HA

SNMP monitoring becomes more interesting in FortiGate HA environments.

### Important Option

```cli
set ha-direct enable
```

### Concept

Without appropriate HA handling:

```text
NMS
 |
 ▼
HA Cluster
 |
 ▼
FortiGate
```

With HA-direct:

```text
             NMS
              |
       ┌──────┴──────┐
       │             │
       ▼             ▼
    FGT-01         FGT-02
```

### HA Checklist

* [ ] Determine whether the NMS monitors the cluster or individual members.
* [ ] Understand `ha-direct`.
* [ ] Configure the appropriate source IP.
* [ ] Verify each HA member can be monitored as intended.
* [ ] Test failover behavior.
* [ ] Confirm traps remain useful after HA role changes.

> [!IMPORTANT]
> **Always define whether your monitoring requirement is cluster-level or member-level.**

---

# 16. DHCP SNMP Events

FortiGate can generate SNMP events related to DHCP.

### Important Events

* [ ] DHCP pool usage reaches approximately **90%**.
* [ ] Duplicate IP detected.
* [ ] DHCP NAK occurs.

### DHCP Pool Usage

```text
DHCP Pool
    |
    +── < 90%
    |      └── Normal
    |
    └── ≈ 90%
           └── SNMP event
```

### DHCP Query OID

```text
1.3.6.1.4.1.12356.101.23
```

This can provide DHCP lease usage information.

---

# 17. Query vs Trap

## Query / Polling

The NMS actively requests information.

```text
NMS
 |
 | SNMP GET
 ▼
FortiGate
 |
 | Response
 ▼
NMS
```

### Typical Query Data

* [ ] CPU
* [ ] Memory
* [ ] Interface statistics
* [ ] Packet counters
* [ ] Device information
* [ ] DHCP lease usage

---

## Trap

FortiGate sends an event notification to the NMS.

```text
FortiGate
    |
    | SNMP Trap
    ▼
   NMS
```

### Typical Trap Events

* [ ] Interface events
* [ ] HA events
* [ ] DHCP events
* [ ] System events
* [ ] Security notifications

### Exam Shortcut

```text
QUERY = NMS asks

TRAP = FortiGate tells
```

> [!TIP]
> If the question says **"NMS periodically retrieves CPU utilization"**, think **Query**.
>
> If it says **"FortiGate immediately notifies NMS about an event"**, think **Trap**.

---

# 18. SNMP Ports

| Function   | Default UDP Port |
| ---------- | ---------------: |
| SNMP Query |            `161` |
| SNMP Trap  |            `162` |

```text
NMS
 │
 └── UDP/161 ───────► FortiGate
                         │
                         │
FortiGate ── UDP/162 ──► NMS
```

### Connectivity Checklist

* [ ] UDP/161 allowed for polling.
* [ ] UDP/162 allowed for traps.
* [ ] Routing exists.
* [ ] Firewall policy permits the traffic where applicable.
* [ ] NMS is listening on the expected port.
* [ ] FortiGate interface allows SNMP.

> [!IMPORTANT]
> **161 = Query**
>
> **162 = Trap**

---

# 19. Production Security Checklist

### SNMP Security

* [ ] Prefer SNMPv3.
* [ ] Prefer `auth-priv`.
* [ ] Use strong authentication.
* [ ] Use encryption/privacy.
* [ ] Restrict NMS source IP addresses.
* [ ] Use a dedicated management network.
* [ ] Avoid exposing SNMP to untrusted networks.
* [ ] Use MIB views where full MIB access is unnecessary.
* [ ] Limit VDOM access appropriately.
* [ ] Use unique credentials.
* [ ] Rotate credentials according to organizational policy.
* [ ] Never store real SNMP passwords in GitHub.

### Interface Security

* [ ] Enable `allowaccess snmp` only where required.
* [ ] Avoid enabling SNMP on unnecessary interfaces.
* [ ] Prefer management interfaces/networks.

### Monitoring Design

* [ ] Define polling requirements.
* [ ] Define trap requirements.
* [ ] Define HA monitoring requirements.
* [ ] Define required OIDs.
* [ ] Test NMS compatibility.
* [ ] Test failover behavior.

---

# 20. Troubleshooting Checklist

When SNMP monitoring fails, troubleshoot in this order.

## Layer 1 — Interface

* [ ] Does the interface allow SNMP?

```cli
show system interface
```

Look for:

```text
allowaccess ... snmp ...
```

---

## Layer 2 — Routing

* [ ] Is there a route to the NMS?
* [ ] Is the return path correct?
* [ ] Is the configured source IP reachable?

---

## Layer 3 — UDP

For polling:

```text
UDP/161
```

For traps:

```text
UDP/162
```

* [ ] UDP/161 reachable?
* [ ] UDP/162 reachable?
* [ ] NMS listening?
* [ ] Intermediate firewall permitting traffic?

---

## Layer 4 — SNMP Version

* [ ] Is the NMS using v1?
* [ ] Is the NMS using v2c?
* [ ] Is the NMS using v3?
* [ ] Does FortiGate configuration match?

---

## Layer 5 — Authentication

### SNMPv1/v2c

* [ ] Correct community string?
* [ ] Correct NMS IP?
* [ ] Correct host configuration?

### SNMPv3

* [ ] Correct username?
* [ ] Correct security level?
* [ ] Correct authentication protocol?
* [ ] Correct authentication password?
* [ ] Correct privacy protocol?
* [ ] Correct privacy password?

---

## Layer 6 — MIB View

* [ ] Is the requested OID inside the MIB view?
* [ ] Is the OID excluded?
* [ ] Is the correct MIB view assigned?
* [ ] Is VDOM restriction blocking access?

---

## Layer 7 — OID

* [ ] Is the requested OID valid?
* [ ] Is the correct Fortinet MIB loaded into the NMS?
* [ ] Is the OID supported by the target FortiOS release?

---

## Layer 8 — HA

* [ ] Is `ha-direct` required?
* [ ] Is the NMS targeting the correct member?
* [ ] Is the source IP correct?
* [ ] Does SNMP behavior remain correct after failover?

---

## Layer 9 — Traps

If polling works but traps do not:

* [ ] Trap status enabled?
* [ ] Correct notify host configured?
* [ ] UDP/162 allowed?
* [ ] NMS listening on UDP/162?
* [ ] Correct event configured?

---

# 21. Quick Configuration Card

## Interface

```cli
config system interface
    edit "port1"
        set allowaccess snmp
    next
end
```

---

## SNMPv2c

```cli
config system snmp community
    edit 1
        set name "monitoring"
        set status enable

        config hosts
            edit "NMS"
                set ip 192.168.254.253
                set host-type any
            next
        end
    next
end
```

---

## SNMPv3

```cli
config system snmp user
    edit "monitor"
        set status enable

        set queries enable
        set trap-status enable

        set security-level auth-priv

        set auth-proto sha256
        set auth-pwd <AUTH_PASSWORD>

        set priv-proto aes
        set priv-pwd <PRIV_PASSWORD>

        set notify-hosts 192.168.254.253
    next
end
```

---

## MIB View

```cli
config system snmp mib-view
    edit "monitor-view"
        set include 1.3.6.1.2.1
        set exclude 1.3.6.1.2.1.1.9.1
    next
end
```

> [!WARNING]
> Treat these CLI blocks as **reference templates**, not universal copy/paste configurations. CLI availability and syntax can vary by FortiOS release and platform.

---

# 22. NSE Exam Traps

> [!IMPORTANT]
> **SNMPv1/v2c = Community-based**

> [!IMPORTANT]
> **SNMPv3 = User + Security Level**

> [!IMPORTANT]
> **`auth-priv` = Authentication + Privacy**

> [!TIP]
> **Query = NMS → FortiGate**

> [!TIP]
> **Trap = FortiGate → NMS**

> [!IMPORTANT]
> **UDP/161 = SNMP Query**

> [!IMPORTANT]
> **UDP/162 = SNMP Trap**

> [!WARNING]
> Creating an SNMP community/user does **not** automatically mean the interface accepts SNMP.

> [!IMPORTANT]
> Interface access must include:

```cli
set allowaccess snmp
```

> [!TIP]
> Use **MIB Views** to restrict accessible OID branches.

> [!IMPORTANT]
> `fortinet-core-mib` = common Fortinet information.

> [!IMPORTANT]
> `fortinet-fortigate-mib` = FortiGate-specific information.

> [!WARNING]
> RFC 1213 MIB-II **EGP** is not supported.

> [!WARNING]
> RFC 2665 `dot3Tests` and `dot3Errors` are not supported.

> [!TIP]
> Generic MIB-II statistics may not represent all FortiGate traffic accurately.

> [!IMPORTANT]
> Understand `ha-direct` when monitoring FortiGate HA members.

---

# 23. SNMP Mental Model

```text
                         SNMP
                           |
              ┌────────────┴────────────┐
              │                         │
            QUERY                      TRAP
              │                         │
          NMS → FGT                 FGT → NMS
              │                         │
              └────────────┬────────────┘
                           │
                        MIB / OID
                           │
              ┌────────────┴────────────┐
              │                         │
      Fortinet Core MIB          FortiGate MIB
              │                         │
      Common information        FGT-specific data
                           │
                           ▼
                       MIB View
                           │
                    ┌──────┴──────┐
                    │             │
                 INCLUDE       EXCLUDE
                    │             │
                    └──────┬──────┘
                           ▼
                    Allowed OIDs
```

---

# 24. Final Memory Map

```text
SNMP
│
├── Interface
│   └── allowaccess snmp
│
├── Versions
│   ├── SNMPv1
│   ├── SNMPv2c
│   └── SNMPv3 ⭐
│
├── Security
│   ├── v1/v2c
│   │   └── Community
│   │
│   └── v3
│       ├── no-auth-no-priv
│       ├── auth-no-priv
│       └── auth-priv ⭐
│
├── MIB
│   ├── fortinet-core-mib
│   ├── fortinet-fortigate-mib
│   ├── RFC 1213
│   └── RFC 2665
│
├── MIB View
│   ├── Include
│   └── Exclude
│
├── Monitoring
│   ├── CPU
│   ├── Memory
│   ├── Interfaces
│   ├── Bandwidth
│   ├── Temperature
│   └── System Health
│
├── Events
│   └── DHCP
│       ├── Pool ≈ 90%
│       ├── Duplicate IP
│       └── DHCP NAK
│
├── HA
│   └── ha-direct
│
└── Ports
    ├── UDP/161 → Query
    └── UDP/162 → Trap
```

---

# ⚡ 60-Second SNMP Checklist

Before deploying FortiGate SNMP, verify:

* [ ] SNMP version selected.
* [ ] NMS IP identified.
* [ ] Interface allows `snmp`.
* [ ] UDP/161 allowed for polling.
* [ ] UDP/162 allowed for traps.
* [ ] Community configured for v1/v2c.
* [ ] User configured for v3.
* [ ] `auth-priv` selected where appropriate.
* [ ] Authentication protocol configured.
* [ ] Privacy protocol configured.
* [ ] MIB loaded into NMS.
* [ ] MIB View configured if required.
* [ ] VDOM scope verified.
* [ ] HA behavior verified.
* [ ] Required DHCP events configured.
* [ ] Required OIDs tested.
* [ ] NMS receives polling data.
* [ ] NMS receives traps.
* [ ] No real credentials committed to GitHub.

---

# 🧠 SheynShield One-Liner

> **SNMP tells the NMS what is happening on FortiGate; MIB defines what can be understood, MIB View controls what can be exposed, and SNMPv3 protects how that information is exchanged.**

---

# 🔥 FortiGate SNMP Cheat Sheet — Ultra Quick Recall

```text
SNMP
 │
 ├── v1/v2c
 │     └── Community
 │
 ├── v3
 │     ├── Authentication
 │     ├── Privacy
 │     └── auth-priv ⭐
 │
 ├── Query
 │     └── UDP/161
 │
 ├── Trap
 │     └── UDP/162
 │
 ├── Interface
 │     └── allowaccess snmp
 │
 ├── MIB
 │     ├── Core
 │     └── FortiGate
 │
 ├── MIB View
 │     ├── Include
 │     └── Exclude
 │
 ├── HA
 │     └── ha-direct
 │
 └── DHCP Events
       ├── ~90% pool usage
       ├── Duplicate IP
       └── DHCP NAK
```

---

# 🔎 Keywords

`FortiGate SNMP` · `FortiOS SNMP` · `FortiGate SNMPv3` · `FortiGate SNMP configuration` · `FortiGate SNMP monitoring` · `FortiGate SNMP traps` · `FortiGate MIB` · `Fortinet MIB` · `FortiGate MIB view` · `FortiGate OID` · `FortiGate SNMPv2c` · `FortiGate SNMPv1` · `FortiGate SNMP troubleshooting` · `FortiGate HA SNMP` · `FortiGate DHCP SNMP` · `SNMP auth-priv` · `FortiOS SNMP CLI` · `Fortinet SNMP MIB` · `NSE4 SNMP` · `NSE7 FortiGate` · `Fortinet Security` · `Network Security Engineering`

---

# 🏷️ Topics

```text
fortigate
fortios
fortinet
snmp
snmpv3
network-monitoring
network-security
fortinet-security
fortigate-monitoring
fortigate-snmp
fortinet-mib
nse4
nse7
cybersecurity
network-engineering
```

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

**SheynShield | Security & Design Knowledge Base**

`Engineering Secure Networks`

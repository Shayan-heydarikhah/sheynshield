# FortiGate SNMP  

> **FortiOS | SNMP Monitoring, MIB, SNMPv1/v2c/v3, MIB Views & Traps**
>
> **SheynShield | Engineering Secure Networks**

---

## 1. SNMP Overview

**SNMP (Simple Network Management Protocol)** allows an external **SNMP Manager/NMS** to monitor and receive notifications from FortiGate.

Typical monitoring targets:

* CPU utilization
* Memory utilization
* Interface status
* Interface bandwidth
* Packet statistics
* Latency / delay
* Temperature
* System health
* HA status
* DHCP events
* Security events
* Other FortiGate-specific statistics

### Basic Architecture

```text
                ┌─────────────────────┐
                │     SNMP Manager    │
                │       / NMS         │
                └──────────┬──────────┘
                           │
                    SNMP Query / Trap
                           │
                           ▼
                ┌─────────────────────┐
                │      FortiGate      │
                │     SNMP Agent      │
                └─────────────────────┘
```

### Two Main SNMP Operations

| Operation        | Direction       | Purpose                         |
| ---------------- | --------------- | ------------------------------- |
| **Query / Poll** | NMS → FortiGate | Retrieve monitoring information |
| **Trap**         | FortiGate → NMS | Notify NMS about an event       |

---

# 2. Enable SNMP Access on the Interface

The interface used by the SNMP Manager must permit SNMP administrative access.

Example:

```bash
config system interface
    edit "port1"
        set allowaccess ping https ssh snmp
    next
end
```

> ⚠️ **Exam Tip:** Configuring the SNMP community/user alone is not enough. The relevant FortiGate interface must also allow **SNMP administrative access**.

---

# 3. MIB — Management Information Base

A **MIB** defines the objects/OIDs that an SNMP Manager can query.

FortiGate supports several important MIBs.

---

## 3.1 Fortinet Core MIB

```text
fortinet-core-mib.mib
```

Contains information common to Fortinet products, including:

* Common system information
* Common configuration information
* Common trap information

The SNMP Manager requires this MIB to correctly interpret information and traps common across Fortinet products.

---

## 3.2 FortiGate MIB

```text
fortinet-fortigate-mib.mib
```

Contains information specific to FortiGate devices.

Includes:

* FortiGate-specific system information
* FortiGate-specific configuration objects
* FortiGate-specific traps

> **Remember:**
> `fortinet-core-mib.mib` → common Fortinet information
> `fortinet-fortigate-mib.mib` → FortiGate-specific information

---

# 4. Standard MIB Support

## RFC 1213 — MIB-II

FortiGate supports MIB-II groups with some exceptions.

### Important Exception

The **EGP group** is not supported.

```text
RFC 1213
└── MIB-II
    └── EGP → Not supported
```

### Important Accuracy Note

Protocol statistics returned through MIB-II may **not accurately represent all Fortinet traffic activity**.

For more accurate FortiGate-specific information, prefer the **Fortinet MIB**.

---

# 5. RFC 2665 — Ethernet-like MIB

FortiGate supports Ethernet-like MIB information.

However, the following groups are not supported:

* `dot3Tests`
* `dot3Errors`

Therefore:

```text
RFC 2665
├── Ethernet-like MIB → Supported
├── dot3Tests         → Not supported
└── dot3Errors        → Not supported
```

---

# 6. SNMP Versions

| Feature         | SNMPv1 | SNMPv2c | SNMPv3 |
| --------------- | -----: | ------: | -----: |
| Community-based |      ✅ |       ✅ |      ❌ |
| Authentication  |      ❌ |       ❌ |      ✅ |
| Encryption      |      ❌ |       ❌ |      ✅ |
| Queries         |      ✅ |       ✅ |      ✅ |
| Traps           |      ✅ |       ✅ |      ✅ |
| Security        |    Low |     Low |   High |

### Recommended

For modern deployments:

```text
SNMPv3
    ↓
Authentication
    +
Privacy/Encryption
```

---

# 7. SNMPv1 / SNMPv2c Configuration

Configuration is performed under:

```bash
config system snmp community
```

Example:

```bash
config system snmp community
    edit 2
        set name "pub-com"
        set status enable

        config hosts
            edit "fgt-1"
                set ip 192.168.254.253
                set source-ip <source-ip>
                set ha-direct enable
                set host-type any
            next
        end

        set query-v1-port <port_number>
        set query-v1-status enable

        set query-v2c-port <port_number>
        set query-v2c-status enable

        set trap-v1-lport <port_number>
        set trap-v1-rport <port_number>
        set trap-v1-status enable

        set trap-v2c-lport <port_number>
        set trap-v2c-rport <port_number>
        set trap-v2c-status enable

        set events <events>
    next
end
```

---

## 7.1 Host Type

The host configuration determines what the SNMP Manager is allowed to do.

```bash
set host-type any
```

Possible behavior includes:

```text
query
trap
any
```

### Concept

```text
SNMP Manager
     │
     ├── Query ───────► FortiGate
     │
     └── Trap ◄──────── FortiGate
```

---

# 8. SNMPv3 Configuration

SNMPv3 configuration is performed under:

```bash
config system snmp user
```

Example:

```bash
config system snmp user
    edit "fgt-1"
        set status enable
        set trap-status enable

        set trap-lport <port_number>
        set trap-rport <port_number>

        set queries enable
        set query-port <port_number>

        set notify-hosts <NMS-IP>

        set source-ip <source-ip>
        set ha-direct enable

        set events <events>

        set security-level auth-priv

        set auth-proto sha256
        set auth-pwd <auth-password>

        set priv-proto aes
        set priv-pwd <privacy-password>
    next
end
```

---

# 9. SNMPv3 Security Levels

SNMPv3 supports three security levels:

| Security Level    | Authentication | Encryption |
| ----------------- | -------------: | ---------: |
| `no-auth-no-priv` |              ❌ |          ❌ |
| `auth-no-priv`    |              ✅ |          ❌ |
| `auth-priv`       |              ✅ |          ✅ |

### Recommended

```text
auth-priv
```

provides:

```text
Authentication
      +
Privacy / Encryption
```

---

# 10. SNMPv3 Authentication Protocols

Example:

```bash
set auth-proto sha256
```

Supported examples:

```text
MD5
SHA
SHA224
SHA256
SHA384
SHA512
```

> **Security Tip:** Prefer stronger authentication algorithms where supported by your NMS and FortiOS version.

---

# 11. SNMPv3 Privacy Protocols

Example:

```bash
set priv-proto aes
```

Possible options include:

```text
DES
AES
AES256
AES256CISCO
```

For encrypted SNMP communication:

```text
security-level auth-priv
            +
authentication protocol
            +
privacy protocol
```

---

# 12. MIB Views

A **MIB view** controls which OID branches an SNMP user/community can access.

This is useful for:

* Least privilege
* Monitoring segmentation
* Limiting sensitive information
* Different monitoring permissions
* Multi-team NMS environments

Configuration:

```bash
config system snmp mib-view
```

---

## 12.1 Include / Exclude

```bash
config system snmp mib-view
    edit "mib-view-smp"
        set include <OID>
        set exclude <OID>
    next
end
```

Concept:

```text
                 MIB Tree
                    │
          ┌─────────┴─────────┐
          │                   │
       INCLUDE              EXCLUDE
          │                   │
       Allowed             Blocked
       OIDs                  OIDs
```

---

# 13. MIB View Example

```bash
config system snmp mib-view
    edit "view1"
        set include 1.3.6.1.2
    next

    edit "view2"
        set include 1.3.6.1.2.1

        set exclude \
            1.3.6.1.2.1.2.1 \
            1.3.6.1.2.1.4.31 \
            1.3.6.1.2.1.1.9.1
    next
end
```

### Key Concept

```text
include
   ↓
Defines the accessible subtree

exclude
   ↓
Removes specific branches from the accessible subtree
```

> **Important:** MIB views are useful when you want to expose only a controlled portion of the MIB tree to an SNMP monitoring system.

---

# 14. MIB View + SNMP Community

A MIB view can be assigned to an SNMP community.

```bash
config system snmp community
    edit 1
        set name "regr-sys"
        set vdoms "vdom1"
    next

    edit 2
        set name "regr-sys1"
        set mib-view "view2"
    next

    edit 3
        set name "regr-sys2"
        set mib-view "view1"
        set vdoms root vdom1
    next
end
```

---

# 15. MIB View + SNMPv3 User

MIB views can also be assigned to SNMPv3 users.

```bash
config system snmp user
    edit "v3user"
        set mib-view "view1"
    next

    edit "v3user1"
        set vdom "vdom1"
    next

    edit "v3user2"
        set mib-view "view1"
        set vdoms root vdom1
    next
end
```

### Security Model

```text
SNMP User
    │
    ├── Authentication
    ├── Privacy
    ├── VDOM restriction
    └── MIB View
             │
             ▼
       Allowed OIDs only
```

---

# 16. SNMP + HA

For HA environments, SNMP can be configured to monitor the HA cluster.

Important option:

```bash
set ha-direct enable
```

### Concept

Without appropriate HA handling:

```text
NMS
 │
 ▼
HA management path
 │
 ▼
FortiGate
```

With HA-direct:

```text
NMS
 │
 ├────────► HA Member 1
 │
 └────────► HA Member 2
```

> **Exam Tip:** Understand whether the SNMP monitoring traffic should target the cluster or individual HA members.

---

# 17. SNMP DHCP Events

FortiGate can generate SNMP events related to DHCP.

Important DHCP events include:

### 1. DHCP Pool Usage

A DHCP server IP pool reaches approximately:

```text
90%
```

usage.

### 2. Duplicate IP Detection

The DHCP server detects that an IP address is already in use.

### 3. DHCP NAK

A DHCP client receives a DHCP NAK.

---

# 18. DHCP SNMP Query

FortiGate also supports SNMP queries for DHCP lease usage information.

OID:

```text
1.3.6.1.4.1.12356.101.23
```

The returned information is based on the percentage of the DHCP pool that has been leased.

---

# 19. SNMPv2c DHCP Event Example

```bash
config system snmp community
    edit 1
        set name "regr-sys"

        config hosts
            edit 1
                set ip 10.1.100.11 255.255.255.255
            next

            edit 2
                set ip 172.16.200.55 255.255.255.255
            next
        end

        set events dhcp
    next
end
```

This allows the configured SNMP hosts to receive DHCP-related SNMP events.

---

# 20. SNMPv3 DHCP Event Example

```bash
config system snmp user
    edit 1
        set notify-hosts 192.168.21.0 192.168.20.0

        set events dhcp

        set security-level auth-priv

        set auth-proto sha256
        set auth-pwd <auth-password>

        set priv-proto aes256
        set priv-pwd <privacy-password>
    next
end
```

---

# 21. Query vs Trap

A common exam distinction:

### Query / Polling

The NMS actively asks FortiGate:

```text
NMS
 │
 │ SNMP GET
 ▼
FortiGate
 │
 │ Response
 ▼
NMS
```

Used for:

* CPU
* RAM
* Interface statistics
* Counters
* Device information
* DHCP lease usage

---

### Trap

FortiGate sends an event without waiting for a query:

```text
FortiGate
    │
    │ SNMP Trap
    ▼
   NMS
```

Useful for:

* Interface events
* HA events
* DHCP events
* System events
* Security-related notifications

---

# 22. SNMP Security Model

### SNMPv1 / SNMPv2c

```text
Community String
       ↓
Access
```

Security limitations:

* No native encryption
* Community string is not equivalent to strong authentication
* Should not be exposed unnecessarily

---

### SNMPv3

```text
Username
   │
   ├── Authentication
   │
   └── Privacy / Encryption
```

Recommended security level:

```text
auth-priv
```

---

# 23. Recommended Production Design

For a production FortiGate:

```text
                    Management Network
                           │
                           ▼
                    ┌─────────────┐
                    │  SNMP / NMS │
                    └──────┬──────┘
                           │
                     SNMPv3
                     auth-priv
                           │
                           ▼
                    ┌─────────────┐
                    │  FortiGate  │
                    └─────────────┘
```

### Recommended Practices

* Prefer **SNMPv3**
* Use `auth-priv`
* Restrict SNMP source addresses
* Use a dedicated management network where possible
* Apply MIB views when full MIB access is unnecessary
* Avoid exposing SNMP to untrusted networks
* Use strong authentication and privacy algorithms
* Monitor HA members appropriately
* Allow SNMP only on required interfaces

---

# 24. Quick Configuration Reference

### SNMP Interface Access

```bash
config system interface
    edit "port1"
        set allowaccess snmp
    next
end
```

### SNMPv2c

```bash
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

### SNMPv3

```bash
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

### MIB View

```bash
config system snmp mib-view
    edit "monitor-view"
        set include 1.3.6.1.2.1
        set exclude 1.3.6.1.2.1.1.9.1
    next
end
```

---

# 25. NSE Exam Quick Hits ⚡

> [!IMPORTANT]
> **SNMPv1/v2c = Community-based**
> **SNMPv3 = User + Authentication + Privacy**

> [!TIP]
> **Query** is initiated by the NMS.
> **Trap** is initiated by FortiGate.

> [!IMPORTANT]
> `auth-priv` provides both authentication and privacy.

> [!TIP]
> Use **MIB views** to restrict which portions of the MIB tree an SNMP user/community can access.

> [!IMPORTANT]
> Fortinet MIBs provide more FortiGate-specific information than generic MIB-II statistics.

> [!WARNING]
> RFC 1213 MIB-II does not provide an exact representation of all Fortinet traffic statistics.

> [!IMPORTANT]
> MIB-II **EGP** group is not supported.

> [!IMPORTANT]
> RFC 2665 `dot3Tests` and `dot3Errors` groups are not supported.

> [!TIP]
> SNMP configuration and interface `allowaccess snmp` are separate considerations.

---

# 26. Troubleshooting Checklist

When SNMP monitoring fails:

```text
1. Is SNMP enabled on the FortiGate?
          │
          ▼
2. Does the interface allow SNMP?
          │
          ▼
3. Is the NMS IP configured?
          │
          ▼
4. Is the correct SNMP version configured?
          │
          ▼
5. SNMPv2c → Check community
   SNMPv3  → Check user/auth/privacy
          │
          ▼
6. Check UDP/161 for queries
          │
          ▼
7. Check UDP/162 for traps
          │
          ▼
8. Check MIB view restrictions
          │
          ▼
9. Check VDOM / HA configuration
          │
          ▼
10. Verify the requested OID
```

---

# 27. Ports to Remember

| Function   | Default UDP Port |
| ---------- | ---------------: |
| SNMP Query |            `161` |
| SNMP Trap  |            `162` |

```text
NMS ──UDP/161──► FortiGate
FortiGate ──UDP/162──► NMS
```

---

# 28. Mental Model

```text
                    SNMP
                     │
         ┌───────────┴───────────┐
         │                       │
       QUERY                    TRAP
         │                       │
      NMS → FGT               FGT → NMS
         │                       │
         └───────────┬───────────┘
                     │
                  MIB / OID
                     │
          ┌──────────┴──────────┐
          │                     │
    Fortinet Core MIB      FortiGate MIB
          │                     │
    Common information      FGT-specific
```

---

## 🔑 Final Memory Map

```text
SNMP
│
├── Interface
│   └── allowaccess snmp
│
├── Versions
│   ├── v1
│   ├── v2c
│   └── v3
│
├── Security
│   ├── v1/v2c → Community
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
│   ├── RAM
│   ├── Interfaces
│   ├── Bandwidth
│   ├── Temperature
│   └── System health
│
└── Events
    └── DHCP
        ├── Pool ≥ 90%
        ├── Duplicate IP
        └── DHCP NAK
```

---

## 📌 SheynShield One-Liner

> **SNMP tells the NMS what is happening on FortiGate; MIB defines what can be understood, MIB View controls what can be exposed, and SNMPv3 protects how that information is exchanged.**

**SheynShield | Security & Design Knowledge Base**
`Engineering Secure Networks`

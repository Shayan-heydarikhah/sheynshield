# FortiGate HA / FGCP / FGSP / VRRP / VDOM Cheat Sheet

> **FortiGate High Availability & Redundancy Quick Reference**
> Topics: **FGCP · HA · A-P · A-A · VDOM · VCluster · FGSP · VRRP · Session Pickup · HA Management · Failover · VMAC · Troubleshooting**

---

## 1. HA Fundamentals

### HA Components

FortiGate HA is primarily based on **FGCP — FortiGate Clustering Protocol**.

```text
                HA Cluster
        ┌───────────────────────┐
        │                       │
      FGT-1                   FGT-2
     Primary                 Secondary
        │                       │
        └────── Heartbeat ──────┘
```

### Main HA Concepts

| Concept             | Meaning                                  |
| ------------------- | ---------------------------------------- |
| FGCP                | FortiGate clustering protocol            |
| HA heartbeat        | Cluster communication                    |
| Primary             | Device currently controlling the cluster |
| Secondary           | Other cluster member                     |
| A-P                 | Active-Passive                           |
| A-A                 | Active-Active                            |
| Override            | Influences primary/master election       |
| Priority            | One of the election factors              |
| Virtual Cluster     | Per-VDOM HA election/load distribution   |
| Session Pickup      | Synchronize sessions between HA members  |
| VMAC                | Virtual MAC used by HA                   |
| Reserved Management | Per-unit management interface            |

---

# 2. Recommended HA Design

### Out-of-Band Management

Prefer a dedicated management network/VLAN for HA members.

```text
              Management Network
                     │
          ┌──────────┴──────────┐
          │                     │
      FGT-1 Mgmt             FGT-2 Mgmt
      10.10.10.11            10.10.10.12
```

Advantages:

* Individual access to each cluster member
* Easier troubleshooting
* SNMP monitoring per unit
* Remote logging
* FortiAnalyzer/FortiCloud communication
* Management remains independent from traffic interfaces

---

# 3. HA Primary Election

A simplified election view:

```text
HA Election
    │
    ├── Monitor / health state
    │
    ├── Priority
    │
    ├── Uptime
    │
    └── Serial number / tie-break information
```

### Priority

In general:

```text
Higher HA priority
       ↓
Higher election preference
```

Example:

```text
FGT-1 = priority 200
FGT-2 = priority 150

Preferred primary = FGT-1
```

> ⚠️ Exact election behavior depends on the HA configuration and FortiOS version. Do not reduce HA election to priority alone.

---

# 4. HA Override

Enable override:

```bash
config system ha
    set override enable
end
```

### Important

If using override, configure the intended HA behavior consistently across the cluster.

```text
FGT-1
override = enable
priority = 200

FGT-2
override = enable
priority = 150
```

### Why Override Matters?

Without override, after a failover the new primary may remain primary even after the original device returns.

With override, the preferred unit can regain the primary role when HA election conditions favor it.

---

## 5. Override + Priority Scenario

Initial state:

```text
FGT-1
Priority = 200
Primary

FGT-2
Priority = 150
Secondary
```

Failure:

```text
FGT-1  ✕
       ↓
FGT-2 becomes Primary
```

When FGT-1 returns:

### With appropriate override configuration

```text
FGT-1
Priority 200
     ↓
Preferred Primary
```

### Without override

The currently active device may remain primary depending on the election conditions.

---

## 6. Important HA Rule

Do **not** manually change HA priority randomly during an incident unless you understand the election state.

Bad operational sequence:

```text
Failover
   ↓
Change priority on secondary
   ↓
Enable/disable override inconsistently
   ↓
Change priority again
   ↓
Unclear election state
```

Better:

```text
1. Identify current primary
2. Check HA status
3. Check priority
4. Check override
5. Check monitor links
6. Correct both members consistently
7. Allow cluster to stabilize
```

---

# 7. HA Status / Troubleshooting Commands

### General Status

```bash
get system ha
```

```bash
get system ha status
```

```bash
diagnose sys ha status
```

Useful for checking:

* Primary/secondary state
* Cluster members
* HA group information
* Priority
* HA synchronization
* Virtual cluster state
* Heartbeat information

---

# 8. HA Management

### Reserved Management Interface

Reserved management interfaces provide:

* Direct management access to each cluster unit
* Individual IP identity
* SNMP monitoring
* Remote logging
* FortiAnalyzer communication
* FortiCloud communication
* FortiSandbox communication
* NetFlow / sFlow
* Remote authentication/certificate communication

### Important Properties

Reserved management interfaces:

* Do **not** use HA VMAC
* Normally retain the physical interface MAC
* Configuration is **not synchronized**
* Each cluster member can have a different management IP

Example:

```text
FGT-1
mgmt1 = 10.10.10.11/24

FGT-2
mgmt1 = 10.10.10.12/24
```

---

## 9. HA Management Configuration

Example structure:

```bash
config system ha
    set ha-direct enable
    set ha-mgmt-status enable

    config ha-mgmt-interfaces
        edit 1
            set interface "mgmt1"
            set dst "10.10.10.0/24"
            set gateway 10.10.10.1
        next
    end
end
```

> ⚠️ Exact options and syntax can vary by FortiOS release and platform.

---

## 10. HA-Direct

`ha-direct` allows management-related traffic to use the reserved management path.

Typical services:

```text
SNMP
Syslog
FortiAnalyzer
FortiCloud
FortiSandbox
NetFlow
sFlow
Remote Authentication
Certificate Validation
```

---

# 11. SNMP + HA Management

For SNMP traffic using the HA management interface:

```bash
config system snmp
    edit 1
        config hosts
            edit 1
                set ha-direct enable
            next
        end
    next
end
```

### Important Concept

```text
Normal interface
      ↓
Configuration synchronized

Reserved management interface
      ↓
Per-device configuration
      ↓
Not synchronized
```

---

# 12. Logs in HA

When connecting to the cluster IP:

```text
              HA IP
                │
        ┌───────┴───────┐
        │               │
      FGT-1           FGT-2
```

You normally see logs associated with the unit currently handling the traffic.

Always check the **device name / hostname** in the log.

### Important

There is no magical "single cluster log database" simply because the devices are in HA.

For centralized logging use:

```text
FortiGate
   │
   ├── FortiAnalyzer
   ├── FortiCloud
   └── Syslog
```

---

# 13. HA Synchronization

After building the HA pair:

```text
FGT-1
  │
  │ Configuration Sync
  ↓
FGT-2
```

Recommended workflow:

```text
1. Configure primary device
2. Connect secondary
3. Establish heartbeat
4. Allow synchronization to complete
5. Verify HA status
6. Test failover
```

### Why?

If:

```text
FGT-1 = fully configured
FGT-2 = empty
```

and you immediately force a role change, you may create unexpected behavior.

---

# 14. HA Synchronization Commands

Access another HA member:

```bash
execute ha manage 0 admin <password>
```

Check HA:

```bash
diagnose sys ha status
```

Synchronize configuration where appropriate:

```bash
execute ha sync start
```

> ⚠️ Command availability/behavior should be checked against the FortiOS version.

---

# 15. Removing a Device from HA

Recommended sequence:

```text
1. Verify cluster is healthy
2. Verify heartbeat links
3. Verify monitor links
4. Verify synchronization
5. Remove secondary/slave
6. Keep primary stable
```

Do **not** remove a member while:

```text
Heartbeat = unstable
Monitor = failed
Synchronization = incomplete
Cluster = unhealthy
```

---

# 16. HA Failover Triggers

Typical failover conditions include:

```text
Device failure
Power loss
Monitored interface failure
SSD failure
Memory-based failover
Other configured health conditions
```

---

# 17. A-P vs A-A

## Active-Passive

```text
              Traffic
                 │
                 ▼
             FGT-1
             ACTIVE
                 │
              HA Sync
                 │
                 ▼
             FGT-2
            PASSIVE
```

Only one unit normally handles production traffic.

---

## Active-Active

```text
                 Traffic
                    │
             ┌──────┴──────┐
             ▼             ▼
          FGT-1          FGT-2
          ACTIVE         ACTIVE
             └──────┬──────┘
                    │
                 FGCP
```

A-A can distribute processing/load, depending on traffic and configuration.

### Important

A-A is **not simply "two independent FortiGates forwarding 50/50 traffic."**

FGCP manages cluster state and traffic distribution.

---

# 18. FGCP

**FGCP = FortiGate Clustering Protocol**

Used for:

* Cluster formation
* HA election
* Configuration synchronization
* Session synchronization
* Cluster state
* Active/Passive operation
* Active/Active operation
* Virtual cluster behavior

---

# 19. FGSP

**FGSP = FortiGate Session Life Support Protocol**

FGSP is useful when FortiGates are **not directly forming one traditional HA cluster**.

Typical topology:

```text
              Load Balancer
             /             \
            /               \
        FGT-1               FGT-2
       Standalone           Standalone
            \               /
             Session Sync
                 FGSP
```

Useful for:

* Session synchronization
* Asymmetric traffic designs
* Load-balancer deployments
* Multiple standalone FortiGates
* Inter-cluster synchronization

---

# 20. FGSP vs FGCP

| Feature                      | FGCP                     | FGSP                        |
| ---------------------------- | ------------------------ | --------------------------- |
| Main purpose                 | HA clustering            | Session synchronization     |
| Cluster formation            | Yes                      | No                          |
| HA election                  | Yes                      | No                          |
| Configuration sync           | Yes                      | Limited/different mechanism |
| Session synchronization      | Yes                      | Yes                         |
| Standalone FortiGate support | No                       | Yes                         |
| Load-balancer topology       | Possible                 | Common use case             |
| Virtual cluster              | Yes                      | No                          |
| Asymmetric-routing designs   | Limited/design-dependent | Strong use case             |

---

# 21. Session Pickup

Example:

```bash
config system ha
    set session-pickup enable
end
```

Useful options may include:

```bash
set session-pickup-expectation enable
set session-pickup-connectionless enable
set session-pickup-nat enable
```

### Connectionless

Useful for:

```text
UDP
ICMP
```

### Expectation Sessions

Important for protocols such as:

```text
FTP
SIP
Other related/expected sessions
```

> ⚠️ Availability of individual options depends on FortiOS version.

---

# 22. FGSP Session Types

Depending on platform/version/configuration, session synchronization can include:

```text
IPv4
IPv6
TCP
UDP
ICMP
SCTP
NAT sessions
Expectation sessions
```

---

# 23. FGSP Sync Configuration

Example:

```bash
config system cluster-sync
    edit 1
        set peer-ip 1.2.3.4
        set peervd "root"
        set syncvd "root"
    next
end
```

Concept:

```text
Peer IP
   ↓
Peer VDOM
   ↓
Which VDOMs synchronize
```

---

# 24. Standalone Cluster Session Sync

Example:

```bash
config system standalone-cluster
    set session-sync-dev port3 port4
end
```

Session synchronization can use:

```text
Layer 2
or
Layer 3
```

depending on design and FortiOS support.

---

# 25. Advanced Topology

```text
       HA Cluster 1
      ┌─────────────┐
      │ FGT-1 FGT-2 │
      └──────┬──────┘
             │
       Standalone/
       Session Sync
             │
      ┌──────┴──────┐
      │             │
   FGT Cluster    FGT Cluster
       #2             #3
```

Large-scale designs require careful consideration of:

* Session synchronization
* CPU
* Memory
* Session count
* Synchronization bandwidth
* Failure domains
* Asymmetric routing

---

# 26. Standalone Config Sync

Example:

```bash
config system ha
    set standalone-config-sync enable
    set override enable
    set priority 255
    set hbdev port3 0 port4 0
end
```

Possible synchronized objects include:

```text
Firewall Policies
Firewall Addresses
UTM/Security Profiles
```

> ⚠️ Standalone configuration synchronization is not equivalent to normal FGCP HA.

---

# 27. Unicast HA / Sync Peer

Example structure:

```bash
config system ha
    set group-name "TEST"
    set hbdev port3 50
    set standalone-config-sync enable

    config unicast-peers
        edit 1
            set peer-ip 10.1.100.72
        next
    end

    set override enable
    set priority 200
    set unicast-status enable
    set unicast-gateway 172.16.200.74
end
```

Use only when the topology requires unicast HA communication.

---

# 28. Heartbeat

Heartbeat links are critical for:

```text
Cluster communication
HA state
Election
Synchronization
Health detection
```

Recommended:

```text
FGT-1 port3 ───────── FGT-2 port3
FGT-1 port4 ───────── FGT-2 port4
```

Prefer redundant heartbeat links.

### Best Practice

Direct back-to-back heartbeat connections can reduce dependency on intermediate switching infrastructure.

---

# 29. HA Heartbeat Timing

Example:

```bash
config system ha
    set hb-interval-in-milliseconds 10
    set hb-lost-threshold 2
end
```

Conceptually:

```text
Heartbeat interval
       +
Lost heartbeat threshold
       ↓
Failover detection time
```

> ⚠️ CLI names and supported ranges differ between FortiOS releases. Verify the exact command on your version.

---

# 30. Split Brain

### What is Split Brain?

Both FortiGates believe they should be primary.

```text
       Network
       /     \
     FGT-1  FGT-2
      MASTER MASTER
```

Potential causes:

* Broken heartbeat links
* Incorrect heartbeat topology
* Network isolation
* Insufficient redundant heartbeat paths

### Prevention

```text
Heartbeat #1 ───────────────
Heartbeat #2 ───────────────
```

Use:

* Multiple heartbeat interfaces
* Direct connections where appropriate
* Reliable cabling
* Correct HA configuration
* Monitoring

---

# 31. HA Failover Time

Conceptual model:

```text
Heartbeat Interval
        ×
Lost Threshold
        ↓
Failure Detection
        ↓
Election
        ↓
GARP / MAC Update
        ↓
Traffic Recovery
```

Reducing heartbeat timers too aggressively can increase sensitivity to transient network issues.

---

# 32. SSD Failover

Enable SSD-based failover where supported:

```bash
config system ha
    set ssd-failover enable
end
```

Useful when SSD failure should cause HA failover.

Important for features that depend heavily on local storage, depending on platform/configuration.

---

# 33. Memory-Based Failover

Example structure:

```bash
config system ha
    set memory-based-failover enable

    set memory-failover-threshold <value>
    set memory-failover-monitor-period <value>
    set memory-failover-sample-rate <value>
    set memory-failover-flip-timeout <value>
end
```

Concept:

```text
Memory Usage
     │
     ▼
Threshold exceeded
     │
     ▼
Repeated samples
     │
     ▼
Failover condition
     │
     ▼
Other HA member becomes active
```

### Important

Memory-based failover is designed to protect against severe memory pressure/conserve-mode conditions.

---

# 34. Failover ≠ Role Recovery

Important concept:

```text
Device failure
     ↓
FGT-2 becomes primary
     ↓
FGT-1 recovers
```

Recovery of FGT-1 does **not automatically mean** it immediately becomes primary.

Election behavior depends on:

```text
Override
Priority
Uptime
Health
Cluster state
Election rules
```

---

# 35. HA Configuration Not Synchronized

Important examples:

```text
Hostname
GUI dashboard widgets
HA override
HA device priority
Virtual cluster priority
Ping-server / dead-gateway HA priority settings
Reserved management interface configuration
Reserved management default route
HA management gateway
```

Also remember:

```text
Licensing / subscriptions
```

must be compatible between HA members.

> ⚠️ Exact synchronization exceptions can change by FortiOS version. Always check the release-specific HA synchronization documentation.

---

# 36. VDOM

**VDOM = Virtual Domain**

Conceptually similar to:

```text
Cisco VRF
```

but VDOM provides much broader administrative/security isolation.

```text
FortiGate
│
├── root / Management VDOM
├── VDOM-A
├── VDOM-B
└── VDOM-C
```

Each VDOM can have independent:

```text
Routing
Firewall Policies
VPN
Interfaces
Security Policies
Administrators
Traffic
```

---

# 37. Enable Multi-VDOM

Example:

```bash
config system global
    set vdom-mode multi-vdom
end
```

Depending on FortiOS version, changing VDOM mode may require confirmation/reboot.

---

# 38. VDOM Resources

VDOMs are isolated logically, but they still share physical resources:

```text
CPU
Memory
ASIC/NPU
Interfaces
Disk
```

Use resource monitoring:

```text
System → Global Resources
```

### Important

Do not interpret VDOM isolation as:

> "Every VDOM gets a completely independent physical FortiGate."

---

# 39. Management VDOM

Common design:

```text
                 FortiGate
                    │
             ┌──────┴──────┐
             │             │
        Management       Traffic
          VDOMs           VDOMs
```

Recommended design:

```text
One dedicated management VDOM
+
Traffic VDOMs
```

Management VDOM can be used for:

* Device administration
* Central management
* Management interfaces
* Administrative services

---

# 40. Admin VDOM vs Traffic VDOM

Conceptually:

### Management/Admin VDOM

```text
Management
Administration
Internal access
Security management
```

### Traffic VDOM

```text
Internet
LAN
WAN
VPN
Firewall traffic
```

---

# 41. VDOM Prompt

Useful when managing multiple VDOMs:

```bash
config global
config system global
    set edit-vdom-prompt enable
end
```

This helps prevent accidentally configuring the wrong VDOM.

---

# 42. Inter-VDOM Routing

Common methods:

```text
VDOM Link
    OR
Inter-VDOM routing
    OR
NPU-related architecture
```

Example:

```text
VDOM-A
   │
   │ VDOM Link
   ▼
VDOM-B
```

### Important

Inter-VDOM traffic may have hardware acceleration/offload limitations depending on architecture and platform.

Always verify:

```text
NPU
ASIC
Offload
VDOM Link
```

behavior for your FortiGate model.

---

# 43. VDOM Link Type

Example:

```bash
config global
    config system vdom-link
        edit "vd-link0"
            set type ethernet
        end
    end
end
```

> ⚠️ Exact supported VDOM-link types depend on FortiOS version.

---

# 44. Virtual Clustering

Virtual clustering allows HA roles to be distributed per VDOM.

Useful especially with:

```text
A-A HA
+
Multiple VDOMs
```

Example:

```text
FGT-1                 FGT-2

VDOM-A → PRIMARY      VDOM-A → SECONDARY

VDOM-B → SECONDARY    VDOM-B → PRIMARY
```

This allows traffic processing to be distributed.

---

# 45. VCluster

Concept:

```text
Physical HA Cluster
        │
        ├── VCluster 1
        │     ├── VDOM-A
        │     └── VDOM-C
        │
        └── VCluster 2
              ├── VDOM-B
              └── VDOM-D
```

Each virtual cluster can have its own:

```text
Priority
Override
VDOM membership
Monitor interfaces
Ping-server monitoring
```

---

# 46. VCluster Configuration

Example:

```bash
config system ha
    set vcluster-status enable

    config vcluster
        edit 2
            set override enable
            set priority <integer>
            set vdom "VDOM-A"
            set monitor "port3"
            set pingserver-monitor-interface "port3"
        next
    end
end
```

> ⚠️ Exact VCluster syntax/options should be checked against the FortiOS release.

---

# 47. VCluster Benefits

Useful when:

```text
VDOM-A traffic → FGT-1
VDOM-B traffic → FGT-2
```

instead of:

```text
All VDOMs → FGT-1
```

This can improve resource utilization in A-A environments.

---

# 48. Group ID

HA `group-id` identifies the HA group.

It is important for:

* Separating HA groups
* VMAC generation
* Cluster identification
* Avoiding conflicts between HA groups

Example:

```bash
config system ha
    set group-id 2
end
```

### Important

When upgrading/migrating FortiOS, review HA group IDs and VMAC behavior.

---

# 49. HA Virtual MAC

HA interfaces can use virtual MAC addresses.

Concept:

```text
Physical MAC
      ↓
HA VMAC
      ↓
Network sees virtual identity
```

This allows traffic to move between HA members without requiring hosts to learn a completely different gateway MAC.

---

# 50. VMAC Structure

Conceptual structure:

```text
Group Prefix
     +
Group ID
     +
VCluster
     +
Interface Index
```

Example:

```text
00:09:0f:09:00:03
│           │  │
│           │  └── Interface index
│           └───── VCluster portion
└──────────────── Group prefix / HA information
```

> ⚠️ Do not rely on manually calculating VMACs across FortiOS versions/platforms without checking the release-specific VMAC format.

---

# 51. VMAC Troubleshooting

Useful command:

```bash
diagnose sys ha mac
```

Useful for checking:

```text
HA VMAC
Physical MAC
Cluster information
Interface mapping
```

---

# 52. NIC Hardware Information

```bash
diagnose hardware deviceinfo nic port1
```

Useful information can include:

```text
MAC
Speed
Duplex
Frames
Errors
Hardware state
```

In HA environments distinguish:

```text
Physical/HW MAC
vs
HA Virtual MAC
```

---

# 53. GARP / ARP After Failover

After failover, the network needs to learn that the active gateway is now reachable through the new HA member.

Concept:

```text
Failover
   ↓
New primary
   ↓
GARP
   ↓
Switch updates MAC/FDB
   ↓
Traffic returns
```

### Gratuitous ARP

```bash
config system ha
    set gratuitous-arps enable
end
```

> Usually enabled by default depending on release/configuration.

---

# 54. ARP Tuning

HA may provide controls for:

```text
ARP advertisements
ARP intervals
GARP behavior
Link-failed signaling
```

Example:

```bash
config system ha
    set arps <value>
    set arps-interval <value>
    set gratuitous-arps enable
end
```

> ⚠️ Exact parameter names/defaults vary by FortiOS version. Use `show full-configuration system ha` on the actual device.

---

# 55. Link-Failed Signal

Concept:

```text
Monitor link failure
       ↓
HA detects failure
       ↓
Traffic interfaces can be signaled/downed
       ↓
Peer takes traffic
```

Example:

```bash
config system ha
    set link-failed-signal disable
end
```

Use carefully because this can affect multiple traffic interfaces.

---

# 56. Hardware Switch vs HA Monitoring

Hardware/software switch interfaces require careful design.

Potential weakness:

```text
FortiGate power loss
       ↓
Entire switch function disappears
       ↓
Clients connected through that switch may lose connectivity
```

Also verify whether the interface type can participate in the HA monitoring design you require.

### Design principle

Prefer explicit physical/logical links when you need:

```text
HA monitoring
Redundancy
Failure detection
```

---

# 57. Force HA Failover

For testing/troubleshooting only:

```bash
execute ha failover set 1
```

Restore:

```bash
execute ha failover unset 1
```

Concept:

```text
Normal
  ↓
Force Failover
  ↓
Test
  ↓
Unset
  ↓
Normal
```

> ⚠️ **Do not use this as a normal production failover mechanism.**

---

# 58. Firmware Upgrade — HA

### Before Upgrade

Always:

```text
1. Backup configuration
2. Check FortiOS upgrade path
3. Verify HA health
4. Verify synchronization
5. Check licenses
6. Check release notes
7. Check known issues
```

Use the official **Fortinet Upgrade Path** for the exact source/target versions.

---

# 59. Uninterruptible Upgrade

Concept:

```text
FGT-1 active
FGT-2 active/standby

      ↓

Upgrade one member

      ↓

Traffic continues

      ↓

Upgrade remaining member
```

The exact procedure depends on HA mode, FortiOS version, model and upgrade path.

---

# 60. Interruptible Upgrade

For planned downtime:

```bash
config system ha
    set uninterruptible-upgrade disable
end
```

This allows an upgrade process that does not prioritize uninterrupted traffic operation.

> ⚠️ Always follow the version-specific Fortinet upgrade procedure.

---

# 61. HA Backup

Before major changes:

```text
BACKUP CONFIGURATION
        ↓
Change
        ↓
Verify
        ↓
Test
```

For production:

```text
Current Config
+
Revision Backup
+
External Backup
```

is preferable.

---

# 62. VDOM Exceptions

Used when a configuration object should not synchronize in the normal way.

Concept:

```text
ALL
│
├── Synchronize everywhere
│
INCLUSIVE
│
└── Synchronize only with selected scope

EXCLUSIVE
│
└── Synchronize with everyone except selected scope
```

Example:

```bash
config global

config system vdom-exception
    edit 1
        set object <object>
        set scope all
        set vdom <VDOM>
    next
end
```

Possible use cases can include:

```text
FortiAnalyzer
Management-related objects
VIP/local-subnet related objects
IP pools / NAT pools
```

> ⚠️ Supported objects and exact syntax depend on FortiOS version.

---

# 63. NetFlow / sFlow on HA Management

Example management interface:

```bash
config system interface
    edit "mgmt1"
        set dedicated-to management
        set netflow-sampler both
        set sflow-sampler enable
    next
end
```

NetFlow example:

```bash
config system netflow
    set collector-ip 192.168.20.1
    set collector-port 9996
    set active-flow-timeout 60
end
```

If using HA management:

```text
HA
 ↓
ha-direct
 ↓
Management Interface
 ↓
NetFlow / sFlow
```

---

# 64. HA Reserved Management — Key Rule

Remember:

```text
Traffic Interface
    ↓
HA VMAC
    ↓
Cluster identity

Reserved Management Interface
    ↓
Physical MAC
    ↓
Individual device identity
```

This is one of the most important differences in HA management design.

---

# 65. A-P → A-A Migration

Before changing mode:

```text
FGT-1 = Primary
FGT-2 = Secondary
```

Example priorities:

```text
FGT-1 = 130
FGT-2 = 129
```

If the election conditions change and override is not enabled, the other unit may become primary.

### Safe principle

Before changing:

```text
1. Verify active member
2. Verify synchronization
3. Verify priority
4. Verify override
5. Change HA mode
6. Monitor election
7. Verify traffic
```

---

# 66. A-A Load Distribution

A-A can distribute traffic processing through FGCP.

```text
                Traffic
                   │
          ┌────────┴────────┐
          ▼                 ▼
       FGT-1              FGT-2
       Worker             Worker
          │                 │
          └────── FGCP ─────┘
```

With VDOMs, virtual clustering can further influence which unit is primary for specific VDOMs.

---

# 67. A-A + VDOM + NPU

Potential problem:

```text
VDOM-A
   │
   │ NPU / VDOM link traversal
   ▼
VDOM-B
```

Traffic may not receive the same acceleration behavior as ordinary interface-to-interface forwarding.

Possible design considerations:

```text
Inter-VDOM routing
+
External routing device
+
NPU/ASIC design
```

Always validate with:

```bash
diagnose debug flow
diagnose sys session list
```

and platform-specific NPU commands.

---

# 68. SCTP Session Handling

SCTP supports:

```text
Multihoming
Multiple paths
Transport-level redundancy
```

FortiOS has an option related to SCTP sessions without INIT:

```bash
config system settings
    set sctp-session-without-init enable
end
```

Potential use cases include environments where maintaining SCTP continuity across path changes is important.

> ⚠️ Do not enable this simply because SCTP exists. Validate the exact application and FortiOS behavior first.

---

# 69. SCTP Security Concepts

SCTP includes:

```text
Verification Tag
Checksum
Association
Multihoming
```

The verification tag helps associate packets with the correct SCTP association.

SCTP is different from traditional TCP three-way handshake behavior.

---

# 70. TLS Session / Handshake Continuity

Some FortiGate proxy/security functions can synchronize information needed to reduce disruption after HA failover.

Example certificate configuration:

```bash
config web-proxy global
    set ssl-ca-cert <certificate>
end
```

The exact synchronization behavior depends on the feature and FortiOS release.

---

# 71. HA Diagnostics

### HA State

```bash
get system ha
```

```bash
diagnose sys ha status
```

### HA MAC

```bash
diagnose sys ha mac
```

### Session Table

```bash
diagnose sys session list
```

### IPS Shared Information

```bash
diagnose ips share list
```

### IPS Sessions

```bash
diagnose ips session list
```

### HA Statistics

```bash
get test hasync 50
```

> ⚠️ Diagnostic commands can vary between FortiOS releases.

---

# 72. HA Checksum

Useful for checking synchronization consistency.

Example:

```bash
diagnose sys ha checksum autoscale-cluster
```

Concept:

```text
FGT-1
  │
  ├── Configuration
  ├── HA attributes
  └── License/state information
           ↓
        Checksum
           ↓
FGT-2
```

---

# 73. HA Sync Architecture

Heartbeat links may carry:

```text
HA control information
Configuration synchronization
Session synchronization
Cluster state
Health information
```

Heartbeat communication uses FortiGate HA mechanisms and is not simply ordinary management traffic.

> ⚠️ Avoid memorizing a single TCP/UDP port as the entire HA protocol definition; protocol behavior is version/platform dependent.

---

# 74. VMAC + VLAN

When VMAC is used:

```text
Physical Interface
       │
       ├── VLAN 10
       ├── VLAN 11
       └── VLAN 12
```

VMAC behavior depends on the HA/VDOM/interface architecture.

Always verify with:

```bash
diagnose sys ha mac
```

rather than assuming a VMAC from memory.

---

# 75. VRRP

**VRRP = Virtual Router Redundancy Protocol**

FortiOS supports VRRP versions depending on FortiOS/platform support.

Basic topology:

```text
              Clients
                 │
        Virtual Gateway
        192.168.10.1
                 │
          ┌──────┴──────┐
          │             │
        FGT-1         FGT-2
        .252           .253
```

---

# 76. Basic VRRP Configuration

Example:

```bash
config system interface
    edit "vlan10"
        set vrrp-virtual-mac enable

        config vrrp
            edit 1
                set vrip 192.168.10.1
                set priority 255
            next
        end
    next
end
```

Second device:

```bash
config system interface
    edit "vlan10"

        config vrrp
            edit 1
                set vrip 192.168.10.1
                set priority 200
            next
        end
    next
end
```

Result:

```text
FGT-1 priority 255 → MASTER
FGT-2 priority 200 → BACKUP
```

---

# 77. VRRP Virtual MAC

VRRP virtual MAC typically follows:

```text
00:00:5E:00:01:<VRID>
```

For example:

```text
VRID 1

00:00:5E:00:01:01
```

The virtual gateway keeps the same virtual MAC when the VRRP master changes.

---

# 78. Why VRRP Virtual MAC Matters

Without virtual MAC behavior, hosts/switches may need to relearn different physical MAC addresses after failover.

With VMAC:

```text
Gateway IP
192.168.10.1
     │
     ▼
VRRP VMAC
00:00:5E:00:01:01
     │
     ▼
Current Master
```

This improves convergence behavior.

---

# 79. VRRP Priority

Example:

```text
FGT-1 = 255
FGT-2 = 200
```

Higher priority generally wins master election.

```text
255 > 200
```

---

# 80. VRRP Monitoring

VRRP can use a destination address for tracking depending on supported FortiOS features.

Example:

```bash
config system interface
    edit "vlan10"

        config vrrp
            edit 1
                set vrip 192.168.10.1
                set priority 255
                set vrdst 8.8.8.8
                set vrdst-priority 10
                set adv-interval 10
                set start-time 10
                set vrgrp 10
                set preempt enable
            next
        end
    next
end
```

Concept:

```text
VRRP Master
    │
    └── Monitor destination
             │
          Failure
             ↓
      Priority adjustment
             ↓
      Another router can win
```

> ⚠️ Exact tracking semantics and commands vary by FortiOS version.

---

# 81. VRRP Preempt

Conceptually similar to HA override:

```text
Preferred device recovers
        ↓
Higher priority
        ↓
Preempt
        ↓
Becomes Master again
```

Example:

```bash
set preempt enable
```

---

# 82. VRRP Group

Example:

```bash
set vrgrp 10
```

This can be used to group VRRP routers for election/tracking behavior.

---

# 83. Single VRRP Domain

Example:

```text
VRID 1

FGT-1
192.168.100.253
Priority 255

FGT-2
192.168.100.254
Priority 200

VIP:
192.168.100.1
```

---

# 84. Multiple VRRP Domains / Instances

Example:

```text
Interface: port8

VRID 1
VIP = 192.168.100.1
Priority = 255

VRID 2
VIP = 192.168.100.100
Priority = 155
```

Each VRID can have its own virtual MAC:

```text
VRID 1
00:00:5E:00:01:01

VRID 2
00:00:5E:00:01:02
```

---

# 85. VRRP Verification

Useful commands:

```bash
get router info vrrp
```

```bash
get system vrrp
```

Look for:

```text
VRID
VRIP
Priority
State
VMAC
Advertisement interval
Preempt
Tracking destination
```

---

# 86. VRRP + DHCP

DHCP clients should normally receive the **virtual gateway IP**:

```text
DHCP Gateway:
192.168.10.1
```

NOT:

```text
FGT-1 physical IP:
192.168.10.252

FGT-2 physical IP:
192.168.10.253
```

Correct:

```text
                DHCP
                  │
                  ▼
         Default Gateway
          192.168.10.1
                  │
            VRRP Master
```

---

# 87. VRRP vs FortiGate HA

| Feature                 | FortiGate HA  | VRRP                 |
| ----------------------- | ------------- | -------------------- |
| Protocol                | FGCP          | VRRP                 |
| Full FortiGate cluster  | Yes           | No                   |
| Config synchronization  | Yes           | No                   |
| Session synchronization | HA mechanisms | No native equivalent |
| Virtual gateway         | Yes           | Yes                  |
| Primary election        | HA election   | VRRP election        |
| VMAC                    | HA VMAC       | VRRP VMAC            |
| VDOM virtual clustering | Yes           | No                   |
| Standalone routers      | No            | Yes                  |

---

# 88. EMAC-VLAN

Enhanced MAC VLAN can be used in specific FortiGate interface architectures.

Concept:

```text
Physical Interface
       │
    EMAC VLAN
       │
      VRRP
```

Example structure:

```bash
config system interface
    edit "port8"
        set type emac
        set ip 192.168.254.200/24
        set allowaccess ping http ssh
        set vrrp-virtual-mac enable

        config vrrp
            edit 1
                set vrip 192.168.254.1
                set priority 100
            next
        end
    next
end
```

> ⚠️ EMAC-VLAN behavior is platform and version dependent. Validate before using it in production.

---

# 89. VRRP Trunk Design

When FortiGate connects to switches:

```text
              Switch
             /      \
         trunk      trunk
           │          │
         FGT-1      FGT-2
           │          │
           └── VRRP ──┘
```

Ensure the switch side correctly carries the required VLANs.

---

# 90. HA + VRRP Design Decision

Use **FortiGate HA** when you need:

```text
FortiGate clustering
Configuration synchronization
Session pickup
HA failover
A-P/A-A
VDOM clustering
```

Use **VRRP** when you need:

```text
Virtual gateway redundancy
Standalone devices
Router redundancy
Multiple independent FortiGates
```

Use **FGSP** when you need:

```text
Session synchronization
between standalone FortiGates
or more advanced load-balancer architectures
```

---

# 91. Advanced HA / FGSP Architecture

```text
                 Clients
                    │
                    ▼
              Load Balancer
               /         \
              /           \
             ▼             ▼
       HA Cluster-1   HA Cluster-2
       ┌──────────┐   ┌──────────┐
       │ FGT1 FGT2│   │ FGT3 FGT4│
       └────┬─────┘   └────┬─────┘
            │              │
            └── FGSP Sync ─┘
```

Possible use cases:

* Large environments
* Load-balanced firewalls
* Multiple HA clusters
* Session continuity
* Asymmetric routing architectures

---

# 92. FGSP Encryption

For Layer-3 session synchronization, encryption may be appropriate.

Example:

```bash
config system standalone-cluster
    set encryption enable
    set psksecret <secret>
end
```

Use a strong secret in production.

---

# 93. FGSP + IKE Monitoring

When IPsec VPNs are involved, session synchronization alone may not be enough.

Example:

```bash
config system cluster-sync
    edit 1
        set peer-ip 1.2.3.4
        set ike-monitor disable
        set ike-monitor-interval 15
        set ike-heartbeat-interval <value>
        set ike-seqjump-speed 10
    next
end
```

Concept:

```text
IPsec
  │
  ├── IKE state
  ├── Session state
  └── Packet/session continuity
```

> ⚠️ Exact IKE-monitor behavior/options must be checked for the FortiOS version.

---

# 94. FGSP Firmware Upgrade

Important compatibility rule:

```text
FGSP synchronization
        ↓
Firmware compatibility matters
```

FortiOS 7.0.2 introduced changes related to the HA virtual MAC range and session synchronization packet structures.

Therefore:

```text
FGT-A 7.0.1
       ↕
FGT-B 7.0.2+
```

should **not** be assumed to have full session-sync compatibility.

Always follow Fortinet's supported upgrade path.

---

# 95. PFCP Compatibility

PFCP support and session information changed across FortiOS releases.

Example problem:

```text
New FortiOS
     │
     │ session sync
     ▼
Older FortiOS
```

New session metadata may not be understood by the older peer.

### Rule

```text
Do not assume FGSP is firmware-version agnostic.
```

---

# 96. Configuration Sync vs Session Sync

### Configuration Sync

```text
Policies
Addresses
Profiles
Configuration
```

### Session Sync

```text
TCP sessions
UDP/ICMP sessions
NAT state
Expectation sessions
Protocol state
```

They are **not the same mechanism**.

---

# 97. Troubleshooting Decision Tree

```text
HA Problem
   │
   ├── Cluster not formed?
   │      ├── Check group-id
   │      ├── Check group-name
   │      ├── Check heartbeat
   │      └── Check model/version/config
   │
   ├── Wrong primary?
   │      ├── Check priority
   │      ├── Check override
   │      ├── Check uptime
   │      ├── Check monitor links
   │      └── Check VCluster
   │
   ├── Failover not happening?
   │      ├── Check monitor
   │      ├── Check heartbeat
   │      ├── Check failover settings
   │      └── Check device health
   │
   ├── Sessions dropped?
   │      ├── Check session pickup
   │      ├── Check FGSP
   │      ├── Check asymmetric routing
   │      └── Check protocol support
   │
   └── Management problem?
          ├── Check reserved management
          ├── Check ha-direct
          ├── Check route/gateway
          └── Check per-device configuration
```

---

# 98. Split-Brain Troubleshooting

Check:

```bash
diagnose sys ha status
```

Then verify:

```text
Heartbeat interface
Heartbeat counters
Cluster group ID
Cluster group name
Primary/secondary state
Monitor links
Network connectivity
```

Physical design:

```text
FGT-1 port3 ───────── FGT-2 port3
FGT-1 port4 ───────── FGT-2 port4
```

Avoid having both heartbeat paths depend on a single switch.

---

# 99. Wrong Primary Troubleshooting

Checklist:

```text
[ ] Is override enabled?
[ ] Is override consistent?
[ ] What is priority on each unit?
[ ] Which unit is currently primary?
[ ] What is uptime?
[ ] Are monitor links healthy?
[ ] Is VCluster enabled?
[ ] Which VDOM owns the VCluster?
[ ] Is this A-P or A-A?
[ ] Is this normal HA or FGSP?
```

---

# 100. HA Lab Checklist

Before testing:

```text
[ ] Same FortiGate model/platform family as required
[ ] Compatible FortiOS version
[ ] Same port/interface requirements
[ ] Compatible licenses/subscriptions
[ ] Heartbeat links connected
[ ] Management access configured
[ ] Backup completed
[ ] HA group configured
[ ] Priority configured
[ ] Override understood
[ ] Monitor links configured
[ ] Session pickup configured if required
```

---

# 101. HA Lab Testing

Test one failure at a time.

### Test 1 — Device Failure

```text
FGT-1
  ↓
Power off
  ↓
FGT-2
  ↓
Should become active
```

### Test 2 — Monitor Failure

```text
Disconnect monitored interface
        ↓
Observe HA
        ↓
Failover
```

### Test 3 — Heartbeat Failure

```text
Disconnect HB link
        ↓
Observe cluster
        ↓
Check split-brain protection
```

### Test 4 — Recovery

```text
Restore FGT-1
        ↓
Wait for synchronization
        ↓
Verify role
```

---

# 102. Golden HA Workflow

```text
                 BACKUP
                    │
                    ▼
          Configure Primary
                    │
                    ▼
          Configure Heartbeat
                    │
                    ▼
          Join Secondary
                    │
                    ▼
          Wait for Sync
                    │
                    ▼
            Verify HA State
                    │
                    ▼
          Configure Monitoring
                    │
                    ▼
            Test Failover
                    │
                    ▼
          Test Recovery
                    │
                    ▼
              Production
```

---

# 103. Production HA Best Practices

### Hardware

```text
✓ Same/supported models
✓ Same/compatible FortiOS
✓ Redundant power
✓ Redundant heartbeat
✓ Redundant network paths
✓ Compatible interfaces
```

### Network

```text
✓ Dedicated management
✓ Direct/redundant heartbeat
✓ Proper switch redundancy
✓ Correct VLAN trunking
✓ Correct routing
```

### HA

```text
✓ Consistent override strategy
✓ Correct priority
✓ Correct monitor interfaces
✓ Session pickup where required
✓ Backup before changes
```

### Operations

```text
✓ Test failover
✓ Test recovery
✓ Document primary/secondary roles
✓ Monitor both units
✓ Centralize logs
✓ Verify upgrade path
```

---

# 104. Quick Command Reference

## HA

```bash
get system ha
get system ha status
diagnose sys ha status
diagnose sys ha mac
```

## HA Configuration

```bash
config system ha
    set override enable
    set priority <value>
    set group-id <value>
end
```

## HA Management

```bash
config system ha
    set ha-direct enable
    set ha-mgmt-status enable
end
```

## HA Member Access

```bash
execute ha manage 0 admin <password>
```

## HA Synchronization

```bash
execute ha sync start
```

## Sessions

```bash
diagnose sys session list
```

## IPS

```bash
diagnose ips share list
diagnose ips session list
```

## HA Statistics

```bash
get test hasync 50
```

## NIC

```bash
diagnose hardware deviceinfo nic port1
```

## VRRP

```bash
get router info vrrp
get system vrrp
```

## HA Failover Testing

```bash
execute ha failover set 1
execute ha failover unset 1
```

---

# 105. One-Page Mental Model

```text
                         FORTIGATE REDUNDANCY
                                  │
              ┌───────────────────┼───────────────────┐
              │                   │                   │
             FGCP                FGSP                VRRP
              │                   │                   │
         HA Cluster         Session Sync       Gateway Redundancy
              │                   │                   │
        ┌─────┴─────┐       Standalone FGTs      Standalone Devices
        │           │
       A-P         A-A
        │           │
        │       Virtual Cluster
        │           │
        │         VDOMs
        │
   Session Pickup
        │
        ▼
   HA Failover
```

---

# 106. The 10 Things to Remember

```text
01. FGCP = FortiGate clustering mechanism

02. FGSP = session synchronization between
    standalone/advanced FortiGate designs

03. VRRP = virtual gateway redundancy

04. Higher priority generally means stronger
    election preference

05. Override controls preferred-role recovery
    behavior

06. Reserved management interfaces have
    individual device identity and are not
    treated like normal HA traffic interfaces

07. Heartbeat redundancy is critical to avoid
    split-brain scenarios

08. Session Pickup is not the same thing as
    configuration synchronization

09. A-A + VDOM requires understanding of
    Virtual Clusters and traffic distribution

10. ALWAYS check the FortiOS version before
    applying HA/FGSP/VRRP commands copied
    from another release
```

---

# 107. Fast Troubleshooting Card

```text
WRONG MASTER?
→ priority
→ override
→ uptime
→ monitor
→ vcluster

NO FAILOVER?
→ heartbeat
→ monitor interface
→ device health
→ failover settings

SESSION LOST?
→ session-pickup
→ FGSP
→ asymmetric routing
→ expectation sessions
→ NAT sync

MANAGEMENT LOST?
→ reserved mgmt IP
→ gateway
→ ha-direct
→ ha-mgmt-status
→ per-device config

SPLIT BRAIN?
→ heartbeat links
→ redundant HB
→ group-id
→ cluster connectivity

UPGRADE?
→ backup
→ upgrade path
→ release notes
→ HA health
→ sync status
```

---

# 108. Final Architecture Map

```text
                         ┌─────────────────────┐
                         │     FortiGate       │
                         └──────────┬──────────┘
                                    │
              ┌─────────────────────┼─────────────────────┐
              │                     │                     │
             HA                   VDOM                  VRRP
              │                     │                     │
             FGCP               Isolation            Gateway HA
              │                     │
        ┌─────┴─────┐        ┌─────┴─────┐
        │           │        │           │
       A-P         A-A     VDOM-A      VDOM-B
        │           │
        │       VCluster
        │           │
        └─────┬─────┘
              │
        Session Pickup
              │
              ▼
         Session State


              Advanced
                 │
                FGSP
                 │
        ┌────────┴────────┐
        │                 │
   Standalone FGT     Standalone FGT
        │                 │
        └── Session Sync ─┘
```

> **Core rule:**
> **FGCP builds the FortiGate HA cluster. FGSP synchronizes sessions between FortiGates/clusters in advanced designs. VRRP provides virtual-router/gateway redundancy. VDOM provides logical isolation, and VCluster distributes HA roles per VDOM.**

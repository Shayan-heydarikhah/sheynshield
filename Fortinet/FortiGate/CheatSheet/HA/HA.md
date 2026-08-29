# FortiGate HA  

## High Availability — FGCP, FGSP, VRRP, VDOM & HA Operations

> **FortiOS Focus:** NSE 4 / NSE 7
> **Scope:** HA architecture, election, failover, session pickup, heartbeat, vCluster, reserved management, FGSP, VRRP, troubleshooting and upgrade
> **Brand:** SheynShield — Engineering Secure Networks

---

# 1. HA at a Glance

FortiGate High Availability provides redundancy by grouping multiple FortiGate units into a cluster.

### Main HA Technologies

| Technology   | Purpose                                                        | Typical Deployment                      |
| ------------ | -------------------------------------------------------------- | --------------------------------------- |
| **FGCP**     | Native FortiGate clustering                                    | FortiGate HA cluster                    |
| **FGSP**     | Session synchronization between standalone FortiGates/clusters | Load balancer / asymmetric environments |
| **VRRP**     | Gateway redundancy                                             | Independent FortiGates/routers          |
| **vCluster** | VDOM-level HA partitioning                                     | Active-active / multi-VDOM environments |

---

# 2. HA Operating Modes

## Active-Passive — A-P

One FortiGate actively processes traffic while another is standby.

```text
             Users
               |
             Switch
               |
        +------+------+
        |             |
     FGT-1          FGT-2
     ACTIVE         STANDBY
        |
      Traffic
```

### Characteristics

* One unit is Active/Primary.
* The other unit is Standby/Secondary.
* Configuration is synchronized.
* Session synchronization can preserve sessions during failover.
* Simpler operational model.
* Common choice for traditional firewall HA.

---

## Active-Active — A-A

Multiple cluster members can actively process traffic.

```text
                 Traffic
                    |
              +-----+-----+
              |           |
            FGT-1       FGT-2
             ACTIVE      ACTIVE
              |           |
              +-----+-----+
                    |
                 Network
```

### Characteristics

* Both units can process traffic.
* Traffic distribution can increase aggregate throughput.
* Uses **FGCP**.
* Session ownership/load distribution becomes more important.
* VDOM/vCluster configuration can influence traffic distribution.

> **Exam Tip:**
> Active-active does **not** simply mean "both firewalls independently route everything." FGCP determines cluster roles and traffic processing behavior.

---

# 3. HA Prerequisites

Before forming an HA cluster, verify:

* Same FortiGate model/platform where required by the supported HA design.
* Compatible/same FortiOS firmware version.
* Compatible hardware configuration.
* HA interfaces are physically connected.
* Heartbeat interfaces are reliable.
* Licenses and registration are completed.
* Network topology is understood.
* Configuration backup exists.

### Before adding a new FortiGate

```text
1. Register FortiGate
2. Activate required licenses
3. Verify FortiOS version
4. Verify hardware/model
5. Configure required HA parameters
6. Connect heartbeat links
7. Join cluster
8. Verify synchronization
```

> **Production Rule:**
> Always take a configuration backup before changing HA membership or performing firmware operations.

---

# 4. Basic HA Configuration

GUI:

```text
System
  └── HA
```

CLI:

```bash
config system ha
    set mode a-p
    set group-id 2
    set group-name "FGT-HA"
    set priority 200
    set override enable
    set hbdev "port3" 50 "port4" 50
end
```

### Device Priority

Range:

```text
0–255
```

Higher priority value = higher preference for becoming primary when priority is considered by the election process.

Default:

```text
128
```

---

# 5. How FortiGate Determines the Primary Unit

HA election should be understood as a **tie-breaking process**, not simply "highest priority always wins."

The exact election behavior depends on HA settings and FortiOS version.

## Without Override

The general election logic considers factors such as:

1. Monitored interface status
2. Uptime
3. Device priority
4. Serial number

The exact ordering is version/implementation dependent, so always verify the behavior for the target FortiOS release.

---

## With Override Enabled

Override changes the election preference so that configured HA priority has greater influence.

Conceptually:

```text
Monitored interface availability
        ↓
Device priority
        ↓
Uptime
        ↓
Serial number
```

### Important

```text
Higher priority value
        ↓
Higher preference
```

Do **not** confuse this with statements such as:

> "0 has the highest priority."

That is incorrect for normal FortiGate HA device-priority interpretation.

---

# 6. HA Override

## What is Override?

FortiGate HA Override is conceptually similar to **Cisco HSRP preempt**.

It controls whether the preferred unit should regain the primary role after recovery.

```text
Override disabled
    ↓
Current primary can remain primary
after another unit returns

Override enabled
    ↓
Preferred unit can reclaim primary role
according to election criteria
```

### Configuration

```bash
config system ha
    set override enable
end
```

### Critical Production Rule

If using override:

> **Configure the HA election strategy consistently across the cluster.**

Do not intentionally leave one member with different override behavior unless you understand the resulting election behavior.

---

# 7. HA Election Example

Assume:

```text
FGT-1
Priority = 200

FGT-2
Priority = 100
```

With appropriate override configuration:

```text
FGT-1 → preferred Primary
FGT-2 → Secondary
```

If FGT-1 fails:

```text
FGT-2 → Primary
```

When FGT-1 returns:

### Override enabled

FGT-1 can reclaim the preferred role.

### Override disabled

FGT-2 may remain Primary depending on the election state.

---

# 8. Changing HA Roles for Testing

You may manipulate HA uptime for testing/election behavior.

Example diagnostic command:

```bash
diagnose sys ha reset-uptime
```

> **Important:**
> HA election manipulation is primarily a troubleshooting/testing technique. Avoid unnecessary manual role manipulation in production.

---

# 9. HA Heartbeat Interfaces

Heartbeat interfaces carry HA control and synchronization information.

They can be used for:

* HA election information
* Configuration synchronization
* Session synchronization
* Cluster state
* Health information

Example:

```bash
config system ha
    set hbdev "port3" 50 "port4" 50
end
```

### Recommended Design

Use redundant heartbeat links:

```text
FGT-1 port3 <----------> FGT-2 port3
FGT-1 port4 <----------> FGT-2 port4
```

Prefer:

* Direct back-to-back links
* Separate physical paths
* Multiple heartbeat interfaces

### Why?

To reduce the probability of:

```text
Heartbeat failure
       ↓
Cluster members cannot see each other
       ↓
Both believe they should become Primary
       ↓
Split Brain
```

---

# 10. Split Brain

## Definition

Split brain occurs when cluster members lose reliable communication with each other and independently believe they should become Primary.

```text
        Heartbeat failure
              ↓
       +------+------+
       |             |
     FGT-1          FGT-2
     PRIMARY        PRIMARY
```

### Prevention

Use:

* Multiple heartbeat interfaces
* Physically diverse paths
* Reliable L2 connectivity
* Direct heartbeat connections where appropriate
* Correct HA monitoring
* Correct network design

---

# 11. Heartbeat Timing

HA heartbeat parameters can influence failure detection.

Example:

```bash
config system ha
    set hb-interval-in-milliseconds 10
    set hb-lost-threshold 2
end
```

Depending on FortiOS release, command spelling/availability may differ.

Conceptually:

```text
Heartbeat interval
       +
Allowed missed heartbeats
       =
Failure detection behavior
```

### Trade-off

Very aggressive timers:

```text
Fast detection
       +
Higher sensitivity to transient network problems
```

Do not blindly reduce timers in production.

---

# 12. Monitored Interfaces

HA can monitor critical interfaces.

Typical examples:

```text
WAN
DMZ
LAN
Critical uplink
Internet-facing interface
```

Example concept:

```bash
config system ha
    set monitor "wan1" "dmz"
end
```

If a monitored interface fails:

```text
Interface failure
      ↓
HA health degradation
      ↓
Failover
```

### Important

Do not rely only on heartbeat connectivity.

A FortiGate can still have a healthy heartbeat while its WAN/uplink is broken.

---

# 13. Failover Triggers

Common HA failover conditions include:

```text
FortiGate device failure
        ↓
Power loss
        ↓
Critical interface failure
        ↓
SSD failure (when configured)
        ↓
Memory-based failover
        ↓
Other HA health conditions
```

---

# 14. Session Pickup

Session pickup allows session state to be synchronized between HA members.

Without session synchronization:

```text
Client
  ↓
FGT-1
  ↓
Session exists only on FGT-1

FGT-1 fails
  ↓
FGT-2 receives packet
  ↓
Session may need to be rebuilt
```

With session pickup:

```text
Client
  ↓
FGT-1
  │
  ├── Session state
  └──────────────→ FGT-2
                       ↓
                    Session
```

Configuration:

```bash
config system ha
    set session-pickup enable
end
```

---

# 15. Session Pickup Types

Depending on FortiOS/version and platform, additional session synchronization options can include:

```bash
set session-pickup enable
set session-pickup-connectionless enable
set session-pickup-expectation enable
set session-pickup-nat enable
```

### Connectionless

Useful for protocols such as:

```text
UDP
ICMP
```

### Expectation Sessions

Useful for protocols/applications involving related sessions, such as:

```text
FTP
SIP
```

---

# 16. Session Pickup vs Configuration Synchronization

These are different concepts.

| Function           | Purpose                                                        |
| ------------------ | -------------------------------------------------------------- |
| Configuration sync | Synchronize configuration                                      |
| Session pickup     | Synchronize active session state                               |
| FGCP               | Native FortiGate clustering                                    |
| FGSP               | Session synchronization between standalone FortiGates/clusters |

---

# 17. FGCP

## FortiGate Clustering Protocol

FGCP is Fortinet's native clustering mechanism.

Used for:

* HA membership
* Primary/Secondary election
* Configuration synchronization
* Cluster state
* Session synchronization mechanisms
* HA failover

Conceptually:

```text
        FGCP
          |
   +------+------+
   |             |
 FGT-1          FGT-2
 Primary       Secondary
```

---

# 18. HA Reserved Management Interface

Reserved management interfaces allow administrators to reach individual cluster members independently.

Example:

```text
              Management Network
                 |
        +--------+--------+
        |                 |
      FGT-1             FGT-2
    10.10.10.11       10.10.10.12
        |                 |
        +------ HA -------+
```

This is extremely useful for:

* Individual device management
* SNMP
* Syslog
* FortiAnalyzer
* FortiCloud
* FortiSandbox communication
* NetFlow
* sFlow
* Remote authentication/certificate verification

---

# 19. Reserved Management Interface Properties

Reserved management interfaces:

* Are individually addressable.
* Keep their physical MAC address rather than HA virtual MAC behavior.
* Configuration is **not synchronized** like normal cluster interface configuration.
* Should be placed on an appropriate management network.

### Important

Do not use the reserved management IPs as the normal FortiManager management address for the HA cluster.

Use the cluster's appropriate management/interface address according to the FortiManager design.

---

# 20. HA Direct

For services that must originate from the individual reserved management interface:

```bash
config system ha
    set ha-mgmt-status enable
    set ha-direct enable
end
```

Example:

```bash
config system ha
    set ha-direct enable
    set ha-mgmt-status enable
    config ha-mgmt-interfaces
        edit 1
            set interface "mgmt"
            set gateway 192.168.20.1
            set dst 192.168.20.0 255.255.255.0
        next
    end
end
```

These management settings are not treated like ordinary synchronized cluster configuration.

---

# 21. SNMP with HA Management

When using reserved management interfaces, HA-direct can be required.

Concept:

```text
SNMP Manager
     |
Management Network
     |
+----+----+
|         |
FGT-1    FGT-2
```

Example:

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

---

# 22. Logs in HA

When connected to the cluster's normal HA/management address, logs generally represent the active processing unit and cluster context.

When directly connecting to a reserved management address:

```text
FGT-1 management IP
        ↓
FGT-1 local perspective/logs

FGT-2 management IP
        ↓
FGT-2 local perspective/logs
```

### Operational Rule

For troubleshooting individual members, use their dedicated management addresses.

---

# 23. SSD Failover

SSD-related failure can trigger HA failover when configured.

Example:

```bash
config system ha
    set ssd-failover enable
end
```

This can be particularly relevant when features depend heavily on local storage.

---

# 24. Memory-Based Failover

FortiGate can use memory conditions as an HA failover trigger.

Conceptual parameters include:

```bash
set memory-based-failover enable
set memory-failover-threshold
set memory-failover-monitor-period
set memory-failover-sample-rate
set memory-failover-flip-timeout
```

### Concept

```text
Memory usage
     ↓
Threshold exceeded
     ↓
Condition monitored
     ↓
Repeated/qualified condition
     ↓
HA failover
```

> **Important:** Exact behavior and parameter names can vary by FortiOS release.

---

# 25. HA Failover Timing

Failover time consists of multiple stages:

```text
Failure
  ↓
Detection
  ↓
Election
  ↓
Role transition
  ↓
MAC/ARP convergence
  ↓
Traffic forwarding
```

Reducing heartbeat timers is only one part of failover optimization.

---

# 26. Gratuitous ARP — GARP

After failover, downstream devices may still have the old MAC/IP association cached.

FortiGate can use gratuitous ARP to accelerate convergence.

Relevant HA settings include:

```bash
config system ha
    set gratuitous-arps enable
end
```

Concept:

```text
Before failover:

Gateway IP → FGT-1 MAC

After failover:

Gateway IP → FGT-2/HA VMAC

        ↓
      GARP
        ↓
Switches/hosts update ARP/FDB
```

---

# 27. ARP Parameters

Relevant parameters can include:

```bash
set arps
set arps-interval
set gratuitous-arps enable
```

Use the values appropriate to the target FortiOS release and production design.

---

# 28. Link-Failed Signaling

HA can use link-failure signaling to react more aggressively to certain interface failures.

Conceptually:

```text
Monitored interface failure
       ↓
Signal cluster
       ↓
Traffic transition
```

This should be designed carefully because aggressive propagation of failures can create unnecessary traffic interruption.

---

# 29. HA Virtual MAC

In HA, interfaces may use virtual MAC addresses so the active unit can assume the cluster identity.

Concept:

```text
Normal:

Client → MAC of FGT-1

Failover:

Client → Same HA/virtual MAC
             ↓
           FGT-2
```

This minimizes the need for network devices to relearn a completely different gateway identity.

---

# 30. VMAC Troubleshooting

Useful command:

```bash
diagnose sys ha mac
```

Hardware information:

```bash
diagnose hardware deviceinfo nic <interface>
```

Useful information includes:

* Physical MAC
* Virtual MAC
* Speed
* Errors
* Frames
* Interface state

---

# 31. HA VMAC Structure

A FortiGate HA VMAC contains information derived from:

```text
Group prefix
+
Group ID
+
Virtual cluster
+
Interface index
```

Conceptual format:

```text
00:09:0f:09 : XX : YY
```

Where the final components encode HA-related identifiers.

### Example

For:

```text
Group ID = 0
Virtual Cluster = 1
Interface index = 3
```

An example VMAC can be:

```text
00:09:0f:09:00:03
```

> **Exam Tip:**
> VMAC structure is useful for troubleshooting and understanding HA behavior, but do not memorize a simplified formula without checking the exact FortiOS release/documentation.

---

# 32. HA Group ID

The Group ID separates HA clusters.

Example:

```text
Cluster A
Group ID = 10

Cluster B
Group ID = 20
```

This prevents unrelated FortiGate devices from accidentally forming the same HA cluster.

### Production Rule

Do not blindly leave every HA deployment at the same default Group ID when multiple clusters share the same L2 environment.

---

# 33. HA Management — Useful Commands

### HA Configuration

```bash
show system ha
```

### HA Information

```bash
get system ha
```

### Detailed HA Status

```bash
get system ha status
```

### HA Synchronization

```bash
execute ha sync start
```

### HA Check/Recalculation

```bash
diagnose sys ha check recalc
```

### Enter another HA member

```bash
execute ha manage <index> <admin>
```

Example:

```bash
execute ha manage 0 admin
```

> Never place real production passwords in documentation or shared  s.

---

# 34. HA Synchronization Troubleshooting

Basic workflow:

```text
1. Check HA status
       ↓
2. Check cluster membership
       ↓
3. Check heartbeat interfaces
       ↓
4. Check firmware versions
       ↓
5. Check configuration checksum
       ↓
6. Check session synchronization
       ↓
7. Check monitored interfaces
       ↓
8. Check logs/events
```

Useful commands:

```bash
get system ha status
diagnose sys ha checksum
diagnose sys ha checksum autoscale-cluster
```

---

# 35. What Does NOT Synchronize?

Important HA exceptions include settings such as:

* Hostname
* GUI dashboard widgets/layout
* HA override setting
* Device priority
* Virtual cluster priority
* Certain HA-specific monitoring/election parameters
* Reserved management interface settings
* Reserved management default route/gateway
* Individual licensing/registration state

### Golden Rule

> **Cluster configuration ≠ identical local device identity.**

Some settings must remain unique per member.

---

# 36. Safe HA Deployment Sequence

Recommended workflow:

```text
                    START
                      |
                      ↓
             Backup configuration
                      |
                      ↓
          Verify model + FortiOS
                      |
                      ↓
             Register/license
                      |
                      ↓
          Configure base connectivity
                      |
                      ↓
          Configure A-P HA first
                      |
                      ↓
          Configure heartbeat links
                      |
                      ↓
          Configure monitored links
                      |
                      ↓
          Configure reserved Mgmt
                      |
                      ↓
          Verify synchronization
                      |
                      ↓
           Test controlled failover
                      |
                      ↓
             Configure A-A/vCluster
                 if required
```

---

# 37. Adding a New FortiGate to a Cluster

Recommended order:

```text
Primary FGT
    ↓
Fully configured
    ↓
Heartbeat connected
    ↓
Secondary joins
    ↓
Configuration synchronization
    ↓
Verify cluster state
    ↓
Only then perform role/failover testing
```

### Why?

If the new unit is empty and you immediately switch traffic:

```text
FGT-1 = Fully configured

FGT-2 = Empty/not synchronized

Failover
    ↓
FGT-2 becomes active
    ↓
Unexpected behavior
```

---

# 38. Removing a FortiGate

When removing a cluster member:

```text
Remove Secondary/Slave first
        ↓
Verify remaining cluster
        ↓
Remove HA connections
```

Avoid removing the current Primary first unless the migration procedure specifically requires it.

---

# 39. HA Backup Strategy

Before:

* HA configuration changes
* Firmware upgrades
* Cluster membership changes
* Major VDOM changes
* FGSP changes

Take:

```text
Configuration backup
```

Think of backup as the **rollback mechanism**, not simply a documentation step.

---

# 40. Firmware Upgrade — HA

Always check the supported:

> **Fortinet Firmware Upgrade Path**

before upgrading.

Two broad upgrade strategies exist.

---

## Uninterruptible Upgrade

Used when supported by the platform/configuration.

Concept:

```text
Cluster
  |
  +-- FGT-1 upgrades
  |
  +-- FGT-2 continues traffic
  |
  +-- FGT-1 returns
  |
  +-- FGT-2 upgrades
```

Goal:

```text
Minimal/no user-visible interruption
```

---

## Interrupted Upgrade

Disable uninterruptible upgrade when required:

```bash
config system ha
    set uninterruptible-upgrade disable
end
```

Then perform a controlled maintenance-window upgrade.

### Production Checklist

```text
✓ Backup
✓ Verify firmware compatibility
✓ Verify upgrade path
✓ Check release notes
✓ Check HA status
✓ Check cluster synchronization
✓ Upgrade according to supported procedure
✓ Verify HA after upgrade
```

---

# 41. Hardware Switch and HA Monitoring

Hardware/software switching can introduce design limitations.

Do not assume that a hardware switch automatically provides HA-level redundancy.

Example problem:

```text
FortiGate power failure
        ↓
Hardware switch still exists
        ↓
Clients may continue using
the same local switching path
```

HA interface monitoring must be designed around actual physical/logical failure domains.

---

# 42. VDOM

VDOM = Virtual Domain.

Conceptually similar to:

```text
Cisco VRF
```

but VDOM provides broader administrative/security isolation than a simple VRF.

Enable multi-VDOM:

```bash
config system global
    set vdom-mode multi-vdom
end
```

---

# 43. VDOM Isolation

Each VDOM can have its own:

* Routing table
* Firewall policies
* VPN configuration
* Security policies
* Interfaces/logical resources
* Administrative context

Global configuration includes things such as:

* Physical interfaces
* DNS/global settings
* Firmware
* Global logging
* Global system behavior

---

# 44. Admin VDOM vs Traffic VDOM

A FortiGate can have:

### Management/Admin VDOM

Used primarily for:

```text
Management
Administration
System services
```

### Traffic VDOM

Used for:

```text
Production traffic
Internet access
Security policies
Routing
VPN
```

A common design principle is:

> Keep management logically separated from production traffic when the architecture requires it.

---

# 45. VDOM HA / Virtual Clustering

With VDOMs, HA can distribute roles at the VDOM level.

This is commonly associated with:

> **Virtual Cluster / vCluster**

Concept:

```text
FGT-1                     FGT-2
------                    ------
VDOM-A → Primary          VDOM-A → Secondary
VDOM-B → Secondary        VDOM-B → Primary
```

This allows different VDOMs to prefer different cluster members.

---

# 46. vCluster

Conceptual configuration:

```bash
config system ha
    set vcluster-status enable

    config vcluster
        edit 1
            set override enable
            set priority <value>
            set vdom "root"
            set monitor "port3"
        next
    end
end
```

Depending on FortiOS release, syntax and available options may differ.

### Why use vCluster?

To distribute active traffic between HA members.

```text
VDOM-A → FGT-1
VDOM-B → FGT-2
VDOM-C → FGT-1
VDOM-D → FGT-2
```

---

# 47. vCluster Design Considerations

Each virtual cluster can have:

* Its own priority
* Its own override behavior
* Its own monitored interfaces
* VDOM membership

This provides granular HA behavior.

### Important

Heartbeat interfaces remain part of the global HA mechanism.

---

# 48. vCluster + A-A

A-A designs become more powerful with VDOM partitioning.

Example:

```text
                  Traffic
                     |
           +---------+---------+
           |                   |
        FGT-1                FGT-2
           |                   |
       VDOM-A                VDOM-B
       ACTIVE                ACTIVE
```

This is fundamentally different from simply running two independent firewalls.

---

# 49. Inter-VDOM Routing

VDOMs can communicate through:

```text
VDOM Links
```

Concept:

```text
VDOM-A
   |
VDOM Link
   |
VDOM-B
```

Depending on platform and architecture, traffic through inter-VDOM links may have hardware-offload limitations.

### Design Rule

When using:

```text
VDOM
+
NPU
+
HA
+
A-A
```

always verify the hardware acceleration/offload behavior for the exact platform.

---

# 50. FGSP

## FortiGate Session Life Support Protocol

FGSP is **not the same as FGCP**.

FGSP is designed for session synchronization between FortiGate devices that are not necessarily members of one traditional FGCP HA cluster.

Typical topology:

```text
                 Load Balancer
                /             \
             FGT-1           FGT-2
             Standalone      Standalone

                ↕ FGSP
          Session Synchronization
```

---

# 51. FGSP Use Cases

Useful when:

* Load balancers are upstream/downstream.
* Asymmetric routing exists.
* FortiGates are standalone.
* Multiple FortiGate clusters need session synchronization.
* Traffic can enter one FortiGate and return through another.

---

# 52. FGSP vs FGCP

| Feature                       | FGCP      | FGSP                        |
| ----------------------------- | --------- | --------------------------- |
| Native HA cluster             | Yes       | No                          |
| Primary/Secondary election    | Yes       | No                          |
| Configuration synchronization | Yes       | Limited/separate mechanisms |
| Session synchronization       | Yes       | Yes                         |
| Standalone FortiGates         | No        | Yes                         |
| Load-balancer architectures   | Sometimes | Excellent use case          |
| vCluster                      | Yes       | No                          |
| Asymmetric-routing designs    | Limited   | Strong use case             |

### Remember

```text
FGCP = Cluster

FGSP = Session synchronization
```

---

# 53. FGSP Session Pickup

Example:

```bash
config system ha
    set session-pickup enable
    set session-pickup-expectation enable
    set session-pickup-connectionless enable
    set session-pickup-nat enable
    set sync-packet-balance enable
end
```

Potentially synchronized session types include:

* IPv4
* IPv6
* TCP
* UDP
* SCTP
* ICMP
* NAT sessions
* Expectation sessions

Exact support depends on FortiOS/platform.

---

# 54. FGSP Session Synchronization Link

A dedicated synchronization interface/path is recommended.

Concept:

```text
FGT-1
  |
  | Session Sync
  |
FGT-2
```

The synchronization path may use L2 or L3 connectivity depending on the design and supported configuration.

---

# 55. Cluster Sync

FGSP can use:

```bash
config system cluster-sync
    edit 1
        set peer-ip 1.2.3.4
        set peervd "root"
        set syncvd "root"
    next
end
```

The exact syntax varies by FortiOS version.

---

# 56. Standalone Cluster

FortiGate also supports advanced synchronization architectures involving standalone clusters.

Concept:

```text
        HA Cluster 1
        FGT-1 + FGT-2
              |
           FGSP/Sync
              |
        HA Cluster 2
        FGT-3 + FGT-4
```

This allows session state to be synchronized across larger distributed architectures.

---

# 57. Standalone Cluster Example

Conceptual configuration:

```bash
config system standalone-cluster
    set standalone-group-id 1
    set group-member-id 0
    set session-sync-dev "port5"
end
```

### Important

All members participating in the same standalone synchronization group need compatible group identifiers and appropriate member identities.

---

# 58. FGSP Encryption

For Layer-3 synchronization paths, encryption can be configured where supported:

```bash
config system standalone-cluster
    set encryption enable
    set psksecret <secret>
end
```

Never publish a real PSK in documentation.

---

# 59. FGSP Scale

FGSP can support multiple FortiGate devices/clusters depending on FortiOS/platform limits.

Do not treat:

```text
16 devices
```

or similar values as universal.

Always verify the exact limit for:

```text
FortiOS version
+
FortiGate model
+
FGSP topology
```

---

# 60. FGSP and Load Balancers

Classic architecture:

```text
                 Load Balancer
                /             \
               /               \
            FGT-1             FGT-2
               \               /
                \             /
                 Server Farm
```

The load balancer may distribute new connections.

FGSP synchronizes session state so that:

```text
Packet enters FGT-1
        ↓
Session synchronized
        ↓
Packet later enters FGT-2
        ↓
FGT-2 understands session
```

---

# 61. Sync Packet Balance

For high-volume environments:

```bash
set sync-packet-balance enable
```

This can distribute synchronization processing.

Large MTU/jumbo frames may also reduce packet-processing overhead where supported end-to-end.

Example:

```text
MTU ≈ 9216
```

> Only use jumbo frames when every device/path in the synchronization path supports them.

---

# 62. IKE Session Synchronization in FGSP

For IPsec environments, IKE state synchronization may be required.

Conceptual configuration:

```bash
config system cluster-sync
    edit 1
        set peer-ip 1.2.3.4
        set ike-monitor enable
        set ike-monitor-interval 15
        set ike-heartbeat-interval <value>
        set ike-seqjump-speed 10
    next
end
```

Exact values and syntax are FortiOS/version dependent.

---

# 63. FGSP Firmware Compatibility

Session synchronization is sensitive to FortiOS versions.

Example historical compatibility issue:

```text
FortiOS 7.0.2+
       ↕
FortiOS 7.0.1-
```

Changes to synchronization packet structures/features can make cross-version session synchronization incompatible.

### Rule

> Do not assume FGSP session synchronization works across arbitrary FortiOS versions.

Always check the release notes and upgrade documentation.

---

# 64. PFCP and Session Compatibility

Modern FortiOS session structures can contain additional information such as PFCP-related state.

If a newer FortiOS synchronizes sessions to an older version that does not understand the additional information:

```text
Session structure mismatch
        ↓
State may be lost/ignored
        ↓
Session behavior can be affected
```

---

# 65. SCTP Session Support

SCTP is useful in environments requiring:

* Multihoming
* Multiple paths
* Telecommunications/signaling
* Resilient path switching

FortiOS can support SCTP session behavior and, depending on release/configuration, session synchronization mechanisms.

Example:

```bash
config system setting
    set sctp-session-without-init enable
end
```

### Concept

```text
SCTP
  |
Multiple paths
  |
Path failure
  ↓
Alternate path
```

This can be useful for highly available streaming/telecom applications.

---

# 66. SCTP Security Concepts

SCTP includes mechanisms such as:

* Verification Tag
* Checksum
* Multi-homing
* Multi-streaming

These help maintain session integrity and path management.

---

# 67. VRRP

## Virtual Router Redundancy Protocol

VRRP provides gateway redundancy between independent devices.

```text
             Clients
                |
          Virtual Gateway
             10.1.1.1
                |
        +-------+-------+
        |               |
      FGT-1           FGT-2
    Priority 255     Priority 200
      MASTER          BACKUP
```

---

# 68. VRRP Virtual MAC

With VRRP virtual MAC:

```text
00:00:5e:00:01:<VRID>
```

Example:

```text
VRID = 1

00:00:5e:00:01:01
```

This provides a shared gateway identity.

---

# 69. VRRP Configuration

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

---

# 70. VRRP Priority

Higher priority wins the VRRP master role.

Example:

```text
FGT-1 = 255
FGT-2 = 200
```

Result:

```text
FGT-1 → MASTER
FGT-2 → BACKUP
```

---

# 71. VRRP Preemption

VRRP preemption is conceptually similar to HA override.

```text
Primary fails
    ↓
Backup becomes Master
    ↓
Original Primary returns
    ↓
Preemption enabled
    ↓
Higher-priority router can become Master again
```

Example:

```bash
config system interface
    edit "vlan10"
        config vrrp
            edit 1
                set preempt enable
            next
        end
    next
end
```

---

# 72. VRRP Monitoring

VRRP can monitor a destination.

Conceptual configuration:

```bash
config system interface
    edit "vlan10"
        config vrrp
            edit 1
                set vrip 192.168.10.1
                set vrdst 8.8.8.8
                set vrdst-priority 10
            next
        end
    next
end
```

This allows gateway priority to be influenced by destination reachability.

---

# 73. VRRP Multiple Virtual Routers

A single interface can participate in multiple VRRP instances/virtual IPs.

Example:

```text
VRID 1
VIP = 192.168.100.1
Priority = 255

VRID 2
VIP = 192.168.100.100
Priority = 155
```

Each VRID has its own virtual MAC.

```text
VRID 1
→ 00:00:5e:00:01:01

VRID 2
→ 00:00:5e:00:01:02
```

---

# 74. VRRP Troubleshooting

Useful commands:

```bash
get router info vrrp
```

and:

```bash
get system vrrp
```

Look for:

```text
VRID
VRIP
Priority
State
Master
Backup
Advertisement interval
Preempt
Virtual MAC
```

---

# 75. VRRP vs FGCP

| Feature                       | FGCP HA | VRRP                             |
| ----------------------------- | ------- | -------------------------------- |
| FortiGate cluster             | Yes     | No                               |
| Configuration synchronization | Yes     | No                               |
| Session pickup                | Yes     | No native FGCP session mechanism |
| Virtual gateway               | Yes     | Yes                              |
| Primary election              | Yes     | Yes                              |
| FortiGate-specific            | Yes     | No                               |
| VDOM/vCluster                 | Yes     | No                               |
| Independent devices           | No      | Yes                              |

### Remember

```text
FGCP
= FortiGate cluster

VRRP
= Virtual gateway redundancy

FGSP
= Session synchronization
```

---

# 76. HA Forced Failover

For controlled testing:

```bash
execute ha failover set <cluster-id>
```

To remove the forced condition:

```bash
execute ha failover unset <cluster-id>
```

### Production Warning

Use forced failover for:

```text
Testing
Troubleshooting
Maintenance validation
```

Not as a routine production operation.

---

# 77. HA Status Verification

Always start troubleshooting with:

```bash
get system ha status
```

Check:

```text
Cluster state
Primary/Secondary role
Serial numbers
Priority
Uptime
Heartbeat
Monitored interfaces
Synchronization
Virtual cluster state
```

---

# 78. Useful HA Troubleshooting Commands

```bash
get system ha
show system ha
get system ha status

diagnose sys ha check recalc
diagnose sys ha mac
diagnose sys ha checksum

diagnose sys session list
diagnose ips session list
diagnose ips share list

execute log display
```

For HA synchronization statistics:

```bash
get test hasync 50
```

---

# 79. HA Checksum

Configuration synchronization can be validated using HA checksum information.

Concept:

```text
FGT-1 configuration checksum
          ≠
FGT-2 configuration checksum
          ↓
Investigate synchronization
```

Useful for:

* Configuration mismatch
* Sync troubleshooting
* Cluster verification

---

# 80. HA and APIPA

FortiGate HA heartbeat communication can use link-local addressing internally.

Do not treat these addresses as ordinary management IP addresses.

Heartbeat links are primarily:

```text
Cluster communication paths
```

---

# 81. HA + NPU + VDOM

One of the more advanced NSE7 topics:

```text
HA
 +
A-A
 +
VDOM
 +
NPU
 +
Inter-VDOM traffic
```

can introduce asymmetric routing/session-distribution considerations.

Potential design:

```text
             Router
           /        \
        FGT-1      FGT-2
          |          |
        VDOM        VDOM
          \          /
           Internal
            routing
```

Depending on the platform and topology, a router or appropriate inter-VDOM routing design may be needed to control traffic paths.

---

# 82. VDOM Exception

Some configuration objects can be selectively synchronized.

Conceptual scopes:

```text
all
inclusive
exclusive
```

Example:

```bash
config global

config system vdom-exception
    edit 1
        set object <object>
        set scope inclusive
        set vdom "test"
    next
end
```

Potential use cases can include:

* FortiAnalyzer-related objects
* Management-related objects
* VIPs
* IP pools
* NAT pools

> Verify the exact supported object types for the FortiOS version in use.

---

# 83. Software Switch / Hardware Switch

Useful command:

```bash
show system switch-interface
```

This shows software switch-related configuration.

### HA Design Warning

Do not automatically assume a hardware switch provides interface-level HA monitoring.

A physical uplink failure inside/behind a switching construct may not produce the HA event you expect.

---

# 84. HA + DHCP

Be careful when DHCP and HA roles are involved.

Gateway configuration should normally reference the **virtual gateway address** where the design requires gateway redundancy.

Example:

```text
DHCP Gateway
      ↓
192.168.10.1
      ↓
VRRP/HA virtual gateway
```

rather than hard-coding one physical FortiGate's IP when that address is not intended to be the client gateway.

---

# 85. HA Design Best Practices

## Heartbeat

```text
✓ Use at least two heartbeat links
✓ Prefer physically diverse paths
✓ Direct links are excellent
✓ Avoid single points of failure
```

## Management

```text
✓ Use reserved/out-of-band management
✓ Separate management network
✓ Give each cluster member unique management IP
```

## Monitoring

```text
✓ Monitor critical WAN links
✓ Monitor important uplinks
✓ Do not monitor every interface blindly
```

## Synchronization

```text
✓ Verify configuration sync
✓ Enable session pickup when required
✓ Test failover
```

## Firmware

```text
✓ Backup first
✓ Check upgrade path
✓ Check release notes
✓ Verify HA health before upgrade
```

---

# 86. HA Failure-Domain Design

Bad design:

```text
          Same Switch
        +-------------+
        |             |
      FGT-1          FGT-2
        |             |
      HB-1            HB-1
```

Better:

```text
FGT-1 HB1 ───────── FGT-2 HB1

FGT-1 HB2 ───────── FGT-2 HB2
     \                /
      Different physical
           paths
```

Best design depends on physical infrastructure and failure domains.

---

# 87. HA Monitoring Strategy

Do not monitor only the heartbeat.

```text
Heartbeat
    +
WAN
    +
Critical LAN
    +
DMZ
    +
Relevant health checks
```

This provides a better representation of the FortiGate's actual service availability.

---

# 88. HA vs Link Monitor

These are related but different.

### HA Interface Monitoring

Answers:

> "Is this cluster member healthy enough to remain active?"

### Link Monitor

Answers:

> "Is this network path/service reachable?"

Link monitoring can be used for health detection and failover-related behavior depending on the design.

---

# 89. HA + Link Monitor

Concept:

```text
FortiGate
   |
Link Monitor
   |
Probe target
   |
Internet/Upstream
```

If the monitored path becomes unavailable:

```text
Health failure
      ↓
HA decision
      ↓
Possible failover
```

Do not confuse this with simply detecting whether the physical interface is electrically up.

---

# 90. Production HA Testing Matrix

Before declaring HA production-ready:

| Test                       | Expected Result                          |
| -------------------------- | ---------------------------------------- |
| Primary power loss         | Secondary takes over                     |
| Primary WAN failure        | Expected failover                        |
| Primary LAN/uplink failure | Expected behavior                        |
| Heartbeat link 1 failure   | Cluster remains healthy                  |
| Heartbeat link 2 failure   | Cluster remains healthy                  |
| Both heartbeat links fail  | Split-brain protection behavior verified |
| Session pickup             | Existing sessions behave as designed     |
| Management access          | Both members reachable                   |
| SNMP                       | Individual members monitored             |
| Syslog                     | Logs received correctly                  |
| FortiAnalyzer              | Connectivity verified                    |
| Firmware upgrade           | Supported procedure succeeds             |
| Primary recovery           | Expected role behavior verified          |
| vCluster failover          | Correct VDOM ownership                   |
| FGSP session sync          | Session survives path change             |

---

# 91. Troubleshooting Decision Tree

```text
HA problem
   |
   +-- Cluster not forming?
   |      |
   |      +-- Model?
   |      +-- Firmware?
   |      +-- Group ID?
   |      +-- Group name?
   |      +-- Heartbeat?
   |      +-- HA mode?
   |
   +-- Wrong Primary?
   |      |
   |      +-- Override?
   |      +-- Priority?
   |      +-- Uptime?
   |      +-- Monitored interfaces?
   |      +-- Serial?
   |
   +-- Failover not happening?
   |      |
   |      +-- Monitor configured?
   |      +-- Interface really failed?
   |      +-- Heartbeat healthy?
   |      +-- Health-check behavior?
   |
   +-- Sessions dropped?
   |      |
   |      +-- Session pickup?
   |      +-- Connectionless pickup?
   |      +-- NAT pickup?
   |      +-- Expectation pickup?
   |
   +-- Config mismatch?
          |
          +-- Checksum
          +-- Synchronization
          +-- HA exceptions
```

---

# 92. NSE Exam Memory Map

## FGCP

```text
Cluster
Election
Primary/Secondary
Config Sync
Session Pickup
Heartbeat
VMAC
vCluster
```

## FGSP

```text
Standalone
Session Sync
Asymmetric Routing
Load Balancer
Multiple FortiGates
```

## VRRP

```text
Virtual Gateway
VRID
Priority
Preempt
Virtual MAC
```

## Reserved Management

```text
Individual Member
Out-of-Band
HA Direct
SNMP
Syslog
FortiAnalyzer
NetFlow
sFlow
```

## vCluster

```text
VDOM
A-A
Per-VDOM Priority
Per-VDOM Monitoring
Traffic Distribution
```

---

# 93. Most Important HA Commands

```bash
# HA configuration
show system ha
get system ha

# HA status
get system ha status

# HA synchronization
execute ha sync start

# HA election/check
diagnose sys ha check recalc

# HA MAC addresses
diagnose sys ha mac

# HA checksum
diagnose sys ha checksum

# HA member access
execute ha manage <index> <admin>

# Session information
diagnose sys session list

# Shared IPS information
diagnose ips share list

# IPS/session information
diagnose ips session list

# HA statistics
get test hasync 50

# VRRP
get router info vrrp
get system vrrp

# Interface hardware
diagnose hardware deviceinfo nic <interface>

# Switch interface
show system switch-interface
```

---

# 94. Ultra-Fast HA  

```text
FGCP
│
├── A-P
│   ├── Active
│   └── Standby
│
├── A-A
│   ├── Multiple active processing
│   └── FGCP traffic distribution
│
├── Election
│   ├── Monitored interfaces
│   ├── Priority
│   ├── Uptime
│   └── Serial number
│
├── Override
│   └── Similar concept to preempt
│
├── Heartbeat
│   ├── HA control
│   ├── Sync
│   └── State
│
├── Session Pickup
│   ├── TCP
│   ├── UDP/ICMP where configured
│   ├── NAT
│   └── Expectations
│
├── VMAC
│   └── Cluster identity
│
├── vCluster
│   └── VDOM-level HA
│
└── Reserved Management
    └── Individual member access


FGSP
│
├── Standalone FortiGates
├── Session synchronization
├── Load balancer
├── Asymmetric routing
└── Multi-cluster designs


VRRP
│
├── Virtual gateway
├── VRID
├── Priority
├── Preempt
└── Virtual MAC
```

---

# 95. Golden Rules

> **1. FGCP = FortiGate clustering.**

> **2. FGSP = session synchronization between FortiGates/clusters.**

> **3. VRRP = virtual gateway redundancy.**

> **4. Higher HA device priority means higher preference when priority is used in the election.**

> **5. Override is conceptually similar to preemption.**

> **6. Heartbeat failure and service failure are not the same thing.**

> **7. Monitor critical interfaces, not every interface blindly.**

> **8. Use redundant heartbeat paths to reduce split-brain risk.**

> **9. Reserved management IPs provide individual member identity.**

> **10. Reserved management configuration is not synchronized like normal HA configuration.**

> **11. Session pickup and configuration synchronization are different mechanisms.**

> **12. vCluster allows HA behavior to be distributed per VDOM.**

> **13. VRRP does not provide FGCP-style configuration/session synchronization.**

> **14. Always verify the exact FortiOS version before applying a command from a different release.**

> **15. Backup + upgrade path + release notes = mandatory before HA firmware changes.**

---

# 96. Production Golden Architecture

A strong production design commonly looks like:

```text
                       Internet
                          |
                    Edge / Router
                          |
                    +-----+-----+
                    |           |
                 WAN1          WAN2
                    |           |
              +-----+-----------+-----+
              |                     |
            FGT-1                 FGT-2
           PRIMARY              SECONDARY
              |                     |
              +==== HB1 ============+
              +==== HB2 ============+
              |                     |
           Reserved              Reserved
          Management            Management
              |                     |
              +-------- Mgmt -------+
                       |
                  Management
                    Network
```

With:

```text
FGCP
+
Redundant Heartbeat
+
Session Pickup
+
Critical Interface Monitoring
+
Reserved Management
+
Configuration Backup
+
Controlled Failover Testing
```

you have the foundation of a robust FortiGate HA deployment.

---

## SheynShield — HA Mental Model

```text
                 FORTIGATE HA
                      │
        ┌─────────────┼─────────────┐
        │             │             │
       FGCP          FGSP          VRRP
        │             │             │
     CLUSTER       SESSION       GATEWAY
        │            SYNC       REDUNDANCY
        │             │             │
   ┌────┼────┐        │        VRID/VMAC
   │    │    │        │
  A-P  A-A vCluster   │
   │    │    │        │
   └────┴────┴────────┘
             │
       Session Pickup
             │
       Heartbeat/VMAC
             │
    Reserved Management
```

**Core distinction to memorize:**

```text
FGCP → "Who is my cluster and who is Primary?"

FGSP → "Which FortiGate knows this session?"

VRRP → "Which device owns the virtual gateway?"

vCluster → "Which FortiGate should own this VDOM?"

Session Pickup → "Can I continue the session after failover?"

Heartbeat → "Are my HA peers alive and synchronized?"

Reserved Management → "How do I reach each physical member individually?"
```

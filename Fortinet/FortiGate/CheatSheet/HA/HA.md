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

One FortiGate actively processes traffic while another unit remains standby.

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
> Active-active does not simply mean that both FortiGates independently operate as separate firewalls. FGCP controls cluster behavior and traffic processing.

---

# 3. HA Prerequisites

Before forming an HA cluster, verify:

* Same FortiGate model/platform where required.
* Compatible FortiOS firmware.
* Compatible hardware configuration.
* HA interfaces are physically connected.
* Heartbeat interfaces are reliable.
* Licenses and registration are completed.
* Network topology is understood.
* Configuration backup exists.

### Before Adding a New FortiGate

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
# Enter the FortiGate HA configuration context.

    set mode a-p
    # Configure the HA operating mode as active-passive.

    set group-id 2
    # Assign the HA group identifier.

    set group-name "FGT-HA"
    # Define the HA cluster name.

    set priority 200
    # Set the device HA priority.

    set override enable
    # Allow the preferred unit to regain the primary role after recovery.

    set hbdev "port3" 50 "port4" 50
    # Configure redundant heartbeat interfaces and their priorities.

end

```

### Device Priority

Range:

```text
0–255
```

Higher priority value means higher preference when priority is used in the election process.

Default:

```text
128
```

---

# 5. HA Election

HA election should be understood as a **tie-breaking process**, not simply "highest priority always wins."

Election behavior depends on HA settings and FortiOS version.

### Important Factors

```text
Monitored interface status
        ↓
Uptime
        ↓
Device priority
        ↓
Serial number
```

The exact ordering can vary by FortiOS implementation and configuration.

---

# 6. HA Override

HA Override determines whether the preferred unit should regain the Primary role after recovery.

Conceptually:

```text
Override disabled
        ↓
Current Primary may remain Primary
after another unit returns

Override enabled
        ↓
Preferred unit can reclaim Primary
according to election criteria
```

CLI:

```bash
config system ha
# Enter the HA configuration context.

    set override enable
    # Enable HA override and allow the preferred unit to reclaim Primary.

end

```

### Important

Higher priority means higher preference.

```text
Priority 200
    >
Priority 100
```

---

# 7. HA Election Example

```text
FGT-1
Priority = 200

FGT-2
Priority = 100
```

With appropriate override configuration:

```text
FGT-1 → Preferred Primary
FGT-2 → Secondary
```

If FGT-1 fails:

```text
FGT-2 → Primary
```

When FGT-1 returns:

```text
Override enabled
    ↓
FGT-1 can reclaim Primary
```

```text
Override disabled
    ↓
FGT-2 may remain Primary
```

---

# 8. HA Election Testing

```bash
diagnose sys ha reset-uptime
# Reset the HA uptime value for election testing.
```

> **Warning:**
> Use election manipulation mainly for troubleshooting and controlled testing.

---

# 9. HA Heartbeat Interfaces

Heartbeat interfaces carry HA control and synchronization information.

They can provide:

* HA election information
* Configuration synchronization
* Session synchronization
* Cluster state
* Health information

Example:

```bash
config system ha
# Enter the HA configuration context.

    set hbdev "port3" 50 "port4" 50
    # Configure two redundant heartbeat interfaces.

end

```

### Recommended Design

```text
FGT-1 port3 <----------> FGT-2 port3
FGT-1 port4 <----------> FGT-2 port4
```

Prefer:

* Direct back-to-back links
* Separate physical paths
* Multiple heartbeat interfaces

---

# 10. Split Brain

Split brain occurs when HA members lose reliable communication and independently believe they should become Primary.

```text
        Heartbeat Failure
              ↓
       +------+------+
       |             |
     FGT-1          FGT-2
     PRIMARY        PRIMARY
```

### Prevention

```text
Multiple heartbeat links
        +
Physically diverse paths
        +
Reliable L2 connectivity
        +
Correct HA monitoring
```

---

# 11. Heartbeat Timing

```bash
config system ha
# Enter the HA configuration context.

    set hb-interval-in-milliseconds 10
    # Define the heartbeat transmission interval.

    set hb-lost-threshold 2
    # Define how many heartbeat losses can trigger failure detection.

end

```

Conceptually:

```text
Heartbeat Interval
        +
Missed Heartbeat Threshold
        =
Failure Detection Behavior
```

> **Production Rule:**
> Do not blindly use aggressive heartbeat timers.

---

# 12. Monitored Interfaces

HA can monitor critical interfaces such as:

```text
WAN
DMZ
LAN
Critical uplinks
Internet-facing interfaces
```

Example:

```bash
config system ha
# Enter the HA configuration context.

    set monitor "wan1" "dmz"
    # Monitor critical interfaces for HA health decisions.

end

```

### Important

A FortiGate can have healthy heartbeat links while its WAN/uplink is unavailable.

Therefore:

```text
Heartbeat Health
        ≠
Service Health
```

---

# 13. Failover Triggers

Common HA failover conditions include:

```text
Device failure
Power loss
Critical interface failure
SSD failure
Memory-based conditions
Other HA health conditions
```

---

# 14. Session Pickup

Session pickup synchronizes session state between HA members.

Without session pickup:

```text
Client
  ↓
FGT-1
  ↓
Session exists on FGT-1

FGT-1 fails
  ↓
FGT-2 receives traffic
  ↓
Session may need to be rebuilt
```

Enable session pickup:

```bash
config system ha
# Enter the HA configuration context.

    set session-pickup enable
    # Enable synchronization of supported session states.

end

```

---

# 15. Session Pickup Types

```bash
config system ha
# Enter the HA configuration context.

    set session-pickup enable
    # Enable session state synchronization.

    set session-pickup-connectionless enable
    # Synchronize supported connectionless sessions such as UDP/ICMP.

    set session-pickup-expectation enable
    # Synchronize supported expectation sessions.

    set session-pickup-nat enable
    # Synchronize supported NAT session information.

end

```

Typical areas:

```text
TCP
UDP
ICMP
NAT
Expectation Sessions
```

Exact support depends on FortiOS version and platform.

---

# 16. Session Pickup vs Configuration Synchronization

These are different mechanisms.

| Function           | Purpose                                             |
| ------------------ | --------------------------------------------------- |
| Configuration Sync | Synchronize configuration                           |
| Session Pickup     | Synchronize active session state                    |
| FGCP               | Native FortiGate clustering                         |
| FGSP               | Session synchronization between FortiGates/clusters |

---

# 17. FGCP

## FortiGate Clustering Protocol

FGCP is Fortinet's native HA clustering mechanism.

It provides mechanisms for:

* HA membership
* Primary/Secondary election
* Configuration synchronization
* Cluster state
* Session synchronization
* HA failover

```text
        FGCP
          |
   +------+------+
   |             |
 FGT-1          FGT-2
 Primary       Secondary
```

---

# 18. Reserved Management Interface

Reserved management interfaces provide individual access to each cluster member.

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

Useful for:

* Individual device management
* SNMP
* Syslog
* FortiAnalyzer
* FortiCloud
* FortiSandbox communication
* NetFlow
* sFlow
* Remote authentication

---

# 19. Reserved Management Configuration

Reserved management interfaces have individual identity.

They:

* Are individually addressable.
* Maintain individual physical MAC identity.
* Are not synchronized like normal cluster interface configuration.
* Should normally use an appropriate management network.

### Important

Reserved management addresses should not automatically be treated as the cluster's normal management identity.

---

# 20. HA Direct

```bash
config system ha
# Enter the HA configuration context.

    set ha-mgmt-status enable
    # Enable reserved HA management interfaces.

    set ha-direct enable
    # Allow supported services to use the reserved management path.

end

```

Example:

```bash
config system ha
# Enter the HA configuration context.

    set ha-direct enable
    # Enable HA-direct functionality.

    set ha-mgmt-status enable
    # Enable reserved management interfaces.

    config ha-mgmt-interfaces
    # Enter reserved management interface configuration.

        edit 1
        # Select the first reserved management interface.

            set interface "mgmt"
            # Assign the physical management interface.

            set gateway 192.168.20.1
            # Define the management gateway.

            set dst 192.168.20.0 255.255.255.0
            # Define the destination network using the management path.

        next
        # Finish the management interface entry.

    end
    # Exit the reserved management configuration.

end

```

---

# 21. SNMP with HA Management

Example:

```bash
config system snmp
# Enter SNMP configuration.

    edit 1
    # Select the SNMP configuration.

        config hosts
        # Enter SNMP host configuration.

            edit 1
            # Select the SNMP manager entry.

                set ha-direct enable
                # Send SNMP communication through the HA management path.

            next
            # Finish the SNMP host entry.

        end
        # Exit SNMP host configuration.

    next
    # Finish the SNMP configuration.

end
# Exit SNMP configuration.
```

---

# 22. Logs in HA

Normal cluster access:

```text
Cluster Management Address
        ↓
Active Processing Context
```

Reserved management:

```text
FGT-1 Management IP
        ↓
FGT-1 Local Perspective

FGT-2 Management IP
        ↓
FGT-2 Local Perspective
```

### Operational Rule

Use individual management addresses when troubleshooting a specific cluster member.

---

# 23. SSD Failover

```bash
config system ha
# Enter the HA configuration context.

    set ssd-failover enable
    # Allow configured SSD conditions to participate in HA failover.

end

```

---

# 24. Memory-Based Failover

Conceptual configuration:

```bash
config system ha
# Enter the HA configuration context.

    set memory-based-failover enable
    # Enable memory-based HA failover monitoring.

    set memory-failover-threshold <value>
    # Define the memory utilization threshold.

    set memory-failover-monitor-period <value>
    # Define how long the condition is monitored.

    set memory-failover-sample-rate <value>
    # Define the memory sampling interval.

    set memory-failover-flip-timeout <value>
    # Define the timeout used by memory-based failover logic.

end

```

Concept:

```text
Memory Usage
     ↓
Threshold
     ↓
Monitoring
     ↓
Qualified Condition
     ↓
HA Failover
```

---

# 25. HA Failover Timing

Failover consists of several stages:

```text
Failure
  ↓
Detection
  ↓
Election
  ↓
Role Transition
  ↓
MAC/ARP Convergence
  ↓
Traffic Forwarding
```

Reducing heartbeat timers is only one part of failover optimization.

---

# 26. Gratuitous ARP — GARP

GARP helps downstream devices update gateway information after failover.

```bash
config system ha
# Enter the HA configuration context.

    set gratuitous-arps enable
    # Enable gratuitous ARP transmission after HA events.

end

```

Concept:

```text
Before Failover:

Gateway IP → FGT-1 MAC

After Failover:

Gateway IP → FGT-2 / HA VMAC

        ↓
       GARP
        ↓
ARP/FDB Convergence
```

---

# 27. ARP Parameters

```bash
config system ha
# Enter the HA configuration context.

    set arps <value>
    # Define the ARP-related HA parameter.

    set arps-interval <value>
    # Define the ARP transmission interval.

    set gratuitous-arps enable
    # Enable gratuitous ARP behavior.

end

```

> Verify exact parameters for the target FortiOS release.

---

# 28. Link-Failed Signaling

Concept:

```text
Interface Failure
       ↓
HA Failure Signal
       ↓
Cluster Decision
       ↓
Traffic Transition
```

Use carefully to avoid unnecessary failovers.

---

# 29. HA Virtual MAC

HA interfaces can use virtual MAC addresses so the active member assumes the cluster identity.

```text
Before:

Client → HA VMAC → FGT-1

After:

Client → HA VMAC → FGT-2
```

This reduces the need for downstream devices to learn a completely different gateway identity.

---

# 30. VMAC Troubleshooting

```bash
diagnose sys ha mac
# Display HA-related MAC information.

diagnose hardware deviceinfo nic <interface>
# Display physical NIC information such as MAC and interface state.
```

Useful information:

```text
Physical MAC
Virtual MAC
Interface State
Speed
Errors
Frames
```

---

# 31. HA VMAC Structure

HA VMAC information is derived from HA-related identifiers such as:

```text
Group ID
Virtual Cluster
Interface Index
```

Conceptual format:

```text
00:09:0f:09:XX:YY
```

> **Exam Tip:**
> Use VMAC structure mainly for troubleshooting and conceptual understanding. Verify the exact FortiOS implementation before relying on a formula.

---

# 32. HA Group ID

The Group ID separates HA clusters.

```text
Cluster A
Group ID = 10

Cluster B
Group ID = 20
```

Example:

```bash
config system ha
# Enter the HA configuration context.

    set group-id 10
    # Assign the HA group identifier.

end

```

### Production Rule

Avoid blindly using the same Group ID for multiple HA clusters sharing the same L2 environment.

---

# 33. HA Management Commands

```bash
show system ha
# Display the HA configuration.

get system ha
# Display the current HA configuration/state.

get system ha status
# Display detailed HA cluster status.

execute ha sync start
# Start HA synchronization.

diagnose sys ha check recalc
# Recalculate/check HA election state.

execute ha manage <index> <admin>
# Access another HA member from the current FortiGate.
```

Example:

```bash
execute ha manage 0 admin
# Access the HA member with index 0 using the admin account.
```

---

# 34. HA Synchronization Troubleshooting

Workflow:

```text
1. Check HA status
2. Check cluster membership
3. Check heartbeat interfaces
4. Check firmware versions
5. Check configuration checksum
6. Check session synchronization
7. Check monitored interfaces
8. Check logs/events
```

Commands:

```bash
get system ha status
# Display cluster membership and HA health.

diagnose sys ha checksum
# Display HA configuration checksum information.

diagnose sys ha checksum autoscale-cluster
# Display checksum information for the autoscale cluster context.
```

---

# 35. What Does Not Synchronize?

Important HA exceptions can include:

```text
Hostname
GUI dashboard state
HA override
Device priority
Virtual cluster priority
Certain HA-specific parameters
Reserved management settings
Individual licensing/registration state
```

### Golden Rule

> **Cluster configuration does not mean identical local device identity.**

---

# 36. Safe HA Deployment Sequence

```text
Backup configuration
        ↓
Verify model + FortiOS
        ↓
Register/license
        ↓
Configure connectivity
        ↓
Configure A-P HA
        ↓
Configure heartbeat
        ↓
Configure monitored interfaces
        ↓
Configure reserved management
        ↓
Verify synchronization
        ↓
Test controlled failover
        ↓
Configure A-A/vCluster if required
```

---

# 37. Adding a New FortiGate

Recommended sequence:

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
Controlled failover test
```

---

# 38. Removing a FortiGate

Recommended sequence:

```text
Remove Secondary
        ↓
Verify Remaining Cluster
        ↓
Remove HA Connections
```

Avoid removing the current Primary first unless the migration procedure specifically requires it.

---

# 39. HA Backup Strategy

Take a configuration backup before:

```text
HA configuration changes
Firmware upgrades
Cluster membership changes
Major VDOM changes
FGSP changes
```

Think of backup as the **rollback mechanism**.

---

# 40. Firmware Upgrade — HA

Always verify:

```text
Firmware Upgrade Path
+
Release Notes
+
HA Compatibility
```

### Uninterruptible Upgrade

```text
FGT-1 upgrades
      ↓
FGT-2 processes traffic
      ↓
FGT-1 returns
      ↓
FGT-2 upgrades
```

### Interrupted Upgrade

```bash
config system ha
# Enter the HA configuration context.

    set uninterruptible-upgrade disable
    # Disable uninterruptible upgrade behavior.

end

```

### Upgrade Checklist

```text
✓ Backup
✓ Verify upgrade path
✓ Verify compatibility
✓ Read release notes
✓ Check HA status
✓ Check synchronization
✓ Perform upgrade
✓ Verify HA after upgrade
```

---

# 41. Hardware Switch and HA Monitoring

Do not assume a hardware switch automatically provides HA-level redundancy.

```text
Physical Failure
      ↓
Switching Layer
      ↓
HA Monitoring
```

HA monitoring must be based on the actual physical and logical failure domain.

---

# 42. VDOM

VDOM = Virtual Domain.

Conceptually similar to VRF in terms of traffic separation, but VDOM provides broader administrative and security isolation.

Enable multi-VDOM:

```bash
config system global
# Enter global system configuration.

    set vdom-mode multi-vdom
    # Enable multi-VDOM operating mode.

end
# Exit global system configuration.
```

---

# 43. VDOM Isolation

Each VDOM can have its own:

```text
Routing
Firewall Policies
VPN
Security Policies
Interfaces/Logical Resources
Administrative Context
```

Global configuration can include:

```text
Physical Interfaces
Firmware
Global System Settings
Global Logging
```

---

# 44. Admin VDOM vs Traffic VDOM

### Management/Admin VDOM

```text
Management
Administration
System Services
```

### Traffic VDOM

```text
Production Traffic
Internet Access
Security Policies
Routing
VPN
```

Keep management logically separated when required by the architecture.

---

# 45. VDOM HA / Virtual Clustering

With VDOMs, HA can distribute active roles at VDOM level.

```text
FGT-1                     FGT-2
------                    ------
VDOM-A → Primary          VDOM-A → Secondary
VDOM-B → Secondary        VDOM-B → Primary
```

This is commonly associated with:

> **vCluster**

---

# 46. vCluster

Conceptual configuration:

```bash
config system ha
# Enter HA configuration.

    set vcluster-status enable
    # Enable virtual clustering.

    config vcluster
    # Enter virtual cluster configuration.

        edit 1
        # Select virtual cluster 1.

            set override enable
            # Enable preferred-role recovery for the virtual cluster.

            set priority <value>
            # Set virtual cluster priority.

            set vdom "root"
            # Associate the VDOM with the virtual cluster.

            set monitor "port3"
            # Monitor the selected interface for virtual cluster health.

        next
        # Finish the virtual cluster entry.

    end
    # Exit virtual cluster configuration.

end
# Exit HA configuration.
```

> Verify syntax and available options for the target FortiOS version.

---

# 47. vCluster Design

Each virtual cluster can have:

```text
Priority
Override
Monitored Interfaces
VDOM Membership
```

Heartbeat interfaces remain part of the global HA mechanism.

---

# 48. vCluster + A-A

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

This allows active traffic processing to be distributed between cluster members.

---

# 49. Inter-VDOM Routing

VDOMs can communicate through VDOM links.

```text
VDOM-A
   |
VDOM Link
   |
VDOM-B
```

Example concept:

```bash
config system vdom-link
# Enter VDOM link configuration.

    edit "vdom-link"
    # Create/select the VDOM link.

end
# Exit VDOM link configuration.
```

> Exact syntax depends on FortiOS release and configuration model.

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

verify hardware-offload behavior for the exact platform.

---

# 50. FGSP

## FortiGate Session Life Support Protocol

FGSP is different from FGCP.

```text
FGCP
=
FortiGate Clustering

FGSP
=
Session Synchronization
```

Typical architecture:

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

FGSP is useful for:

```text
Load Balancers
Asymmetric Routing
Standalone FortiGates
Multiple FortiGate Clusters
Distributed Security Architectures
```

---

# 52. FGSP vs FGCP

| Feature                       | FGCP     | FGSP            |
| ----------------------------- | -------- | --------------- |
| Native HA cluster             | Yes      | No              |
| Primary/Secondary election    | Yes      | No              |
| Configuration synchronization | Yes      | No/Separate     |
| Session synchronization       | Yes      | Yes             |
| Standalone FortiGates         | No       | Yes             |
| Load balancer architecture    | Possible | Strong use case |
| vCluster                      | Yes      | No              |
| Asymmetric routing            | Limited  | Strong use case |

### Remember

```text
FGCP = Cluster

FGSP = Session Synchronization
```

---

# 53. FGSP Session Pickup

Example:

```bash
config system ha
# Enter HA configuration.

    set session-pickup enable
    # Enable session synchronization.

    set session-pickup-expectation enable
    # Synchronize supported expectation sessions.

    set session-pickup-connectionless enable
    # Synchronize supported connectionless sessions.

    set session-pickup-nat enable
    # Synchronize supported NAT session state.

    set sync-packet-balance enable
    # Distribute synchronization packet processing.

end
# Exit HA configuration.
```

---

# 54. FGSP Synchronization Link

Recommended design:

```text
FGT-1
  |
  | Session Synchronization
  |
FGT-2
```

Use a dedicated synchronization path when practical.

The path can use supported L2/L3 designs depending on the FortiOS implementation.

---

# 55. Cluster Sync

Conceptual configuration:

```bash
config system cluster-sync
# Enter cluster synchronization configuration.

    edit 1
    # Select the first synchronization peer.

        set peer-ip 1.2.3.4
        # Define the synchronization peer IP address.

        set peervd "root"
        # Define the peer VDOM.

        set syncvd "root"
        # Define the local synchronization VDOM.

    next
    # Finish the synchronization peer entry.

end
# Exit cluster synchronization configuration.
```

---

# 56. Standalone Cluster

Advanced architecture:

```text
        HA Cluster 1
        FGT-1 + FGT-2
              |
           FGSP/Sync
              |
        HA Cluster 2
        FGT-3 + FGT-4
```

This allows distributed session synchronization.

---

# 57. Standalone Cluster Example

Conceptual configuration:

```bash
config system standalone-cluster
# Enter standalone cluster configuration.

    set standalone-group-id 1
    # Define the standalone synchronization group ID.

    set group-member-id 0
    # Define the local member ID.

    set session-sync-dev "port5"
    # Define the interface used for session synchronization.

end
# Exit standalone cluster configuration.
```

---

# 58. FGSP Encryption

For supported Layer-3 synchronization designs:

```bash
config system standalone-cluster
# Enter standalone cluster configuration.

    set encryption enable
    # Enable synchronization-path encryption.

    set psksecret <secret>
    # Configure the pre-shared secret for synchronization security.

end
# Exit standalone cluster configuration.
```

> Never publish a real PSK in documentation.

---

# 59. FGSP Scale

Do not memorize a universal device limit.

Always verify:

```text
FortiOS Version
+
FortiGate Model
+
FGSP Topology
+
Feature Combination
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

Traffic can enter one FortiGate and later arrive through another.

FGSP synchronizes session state:

```text
Packet → FGT-1
          ↓
     Session Sync
          ↓
Packet → FGT-2
          ↓
   Session Recognized
```

---

# 61. Sync Packet Balance

```bash
config system ha
# Enter HA configuration.

    set sync-packet-balance enable
    # Balance synchronization packet processing.

end
# Exit HA configuration.
```

Jumbo frames may reduce processing overhead when supported end-to-end.

Example:

```text
MTU ≈ 9216
```

> Only use jumbo frames when every device on the synchronization path supports them.

---

# 62. IKE Session Synchronization

Conceptual configuration:

```bash
config system cluster-sync
# Enter cluster synchronization configuration.

    edit 1
    # Select the synchronization peer.

        set peer-ip 1.2.3.4
        # Define the synchronization peer.

        set ike-monitor enable
        # Enable IKE session monitoring.

        set ike-monitor-interval 15
        # Define the IKE monitoring interval.

        set ike-heartbeat-interval <value>
        # Define the IKE heartbeat interval.

        set ike-seqjump-speed 10
        # Define the IKE sequence jump speed.

    next
    # Finish the synchronization peer configuration.

end
# Exit cluster synchronization configuration.
```

---

# 63. FGSP Firmware Compatibility

FGSP session synchronization is sensitive to FortiOS compatibility.

Do not assume arbitrary cross-version synchronization will work.

### Rule

```text
Same/Compatible FortiOS
        +
Compatible Platform
        +
Supported FGSP Design
```

Always verify release notes and upgrade documentation.

---

# 64. PFCP and Session Compatibility

Modern session structures can contain additional state.

Concept:

```text
Newer Session Structure
        ↓
Older FortiOS
        ↓
Unsupported Information
        ↓
Session State May Be Affected
```

---

# 65. SCTP Session Support

SCTP can provide:

```text
Multihoming
Multiple Paths
Telecom Signaling
Path Failover
```

Example:

```bash
config system setting
# Enter system setting configuration.

    set sctp-session-without-init enable
    # Allow supported SCTP sessions without an INIT packet.

end
# Exit system setting configuration.
```

---

# 66. SCTP Security Concepts

SCTP includes mechanisms such as:

```text
Verification Tag
Checksum
Multihoming
Multistreaming
```

These support session integrity and path management.

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

VRRP uses a virtual MAC:

```text
00:00:5e:00:01:<VRID>
```

Example:

```text
VRID = 1

00:00:5e:00:01:01
```

---

# 69. VRRP Configuration

Primary device:

```bash
config system interface
# Enter interface configuration.

    edit "vlan10"
    # Select the interface participating in VRRP.

        set vrrp-virtual-mac enable
        # Enable the VRRP virtual MAC.

        config vrrp
        # Enter VRRP configuration.

            edit 1
            # Create/select VRRP instance 1.

                set vrip 192.168.10.1
                # Define the virtual gateway IP address.

                set priority 255
                # Set the VRRP priority.

            next
            # Finish the VRRP instance.

        end
        # Exit VRRP configuration.

    next
    # Finish interface configuration.

end
# Exit interface configuration.
```

Secondary:

```bash
config system interface
# Enter interface configuration.

    edit "vlan10"
    # Select the VRRP interface.

        config vrrp
        # Enter VRRP configuration.

            edit 1
            # Select VRRP instance 1.

                set vrip 192.168.10.1
                # Use the same virtual gateway IP.

                set priority 200
                # Set a lower VRRP priority.

            next
            # Finish the VRRP instance.

        end
        # Exit VRRP configuration.

    next
    # Finish interface configuration.

end
# Exit interface configuration.
```

---

# 70. VRRP Priority

Higher priority wins the Master role.

```text
FGT-1 = 255
FGT-2 = 200

FGT-1 → MASTER
FGT-2 → BACKUP
```

---

# 71. VRRP Preemption

Preemption allows a higher-priority device to regain Master after recovery.

```bash
config system interface
# Enter interface configuration.

    edit "vlan10"
    # Select the VRRP interface.

        config vrrp
        # Enter VRRP configuration.

            edit 1
            # Select VRRP instance 1.

                set preempt enable
                # Allow the higher-priority device to reclaim Master.

            next
            # Finish the VRRP instance.

        end
        # Exit VRRP configuration.

    next
    # Finish interface configuration.

end
# Exit interface configuration.
```

---

# 72. VRRP Monitoring

Conceptual configuration:

```bash
config system interface
# Enter interface configuration.

    edit "vlan10"
    # Select the VRRP interface.

        config vrrp
        # Enter VRRP configuration.

            edit 1
            # Select VRRP instance 1.

                set vrip 192.168.10.1
                # Define the virtual gateway IP.

                set vrdst 8.8.8.8
                # Define the destination used for VRRP monitoring.

                set vrdst-priority 10
                # Define the priority adjustment related to destination monitoring.

            next
            # Finish the VRRP instance.

        end
        # Exit VRRP configuration.

    next
    # Finish interface configuration.

end
# Exit interface configuration.
```

---

# 73. Multiple VRRP Instances

Example:

```text
VRID 1
VIP = 192.168.100.1
Priority = 255

VRID 2
VIP = 192.168.100.100
Priority = 155
```

Virtual MACs:

```text
VRID 1
→ 00:00:5e:00:01:01

VRID 2
→ 00:00:5e:00:01:02
```

---

# 74. VRRP Troubleshooting

```bash
get router info vrrp
# Display VRRP routing and state information.

get system vrrp
# Display system-level VRRP information.
```

Check:

```text
VRID
VRIP
Priority
State
Master
Backup
Advertisement Interval
Preempt
Virtual MAC
```

---

# 75. VRRP vs FGCP

| Feature             | FGCP HA | VRRP                     |
| ------------------- | ------- | ------------------------ |
| FortiGate Cluster   | Yes     | No                       |
| Configuration Sync  | Yes     | No                       |
| Session Pickup      | Yes     | No native FGCP mechanism |
| Virtual Gateway     | Yes     | Yes                      |
| Primary Election    | Yes     | Yes                      |
| FortiGate Specific  | Yes     | No                       |
| vCluster            | Yes     | No                       |
| Independent Devices | No      | Yes                      |

### Remember

```text
FGCP
= FortiGate Cluster

VRRP
= Virtual Gateway Redundancy

FGSP
= Session Synchronization
```

---

# 76. HA Forced Failover

```bash
execute ha failover set <cluster-id>
# Force the specified HA cluster/virtual cluster failover for testing.
```

Remove the forced condition:

```bash
execute ha failover unset <cluster-id>
# Remove the forced HA failover condition.
```

> Use forced failover for testing and controlled maintenance validation.

---

# 77. HA Status Verification

Start troubleshooting with:

```bash
get system ha status
# Display the complete HA cluster status.
```

Check:

```text
Cluster State
Primary/Secondary Role
Serial Numbers
Priority
Uptime
Heartbeat
Monitored Interfaces
Synchronization
Virtual Cluster State
```

---

# 78. Useful HA Troubleshooting Commands

```bash
get system ha
# Display HA configuration/state.

show system ha
# Display the HA configuration in CLI format.

get system ha status
# Display detailed HA status.

diagnose sys ha check recalc
# Recalculate/check HA election information.

diagnose sys ha mac
# Display HA virtual and physical MAC information.

diagnose sys ha checksum
# Display HA configuration checksum information.

diagnose sys session list
# Display active firewall sessions.

diagnose ips session list
# Display IPS-related session information.

diagnose ips share list
# Display shared IPS information.

execute log display
# Display available local log information.

get test hasync 50
# Display HA synchronization test/statistics information.
```

---

# 79. HA Checksum

HA checksum can help identify configuration synchronization problems.

```bash
diagnose sys ha checksum
# Display configuration checksum information for HA members.
```

Concept:

```text
FGT-1 Checksum
      ≠
FGT-2 Checksum
      ↓
Investigate Synchronization
```

Useful for:

```text
Configuration mismatch
Sync troubleshooting
Cluster verification
```

---

# 80. HA and APIPA

FortiGate HA heartbeat communication can use link-local addressing internally.

These addresses should not be treated as normal management addresses.

```text
Heartbeat Address
=
Cluster Communication
```

---

# 81. HA + NPU + VDOM

Advanced NSE7 design:

```text
HA
+
A-A
+
VDOM
+
NPU
+
Inter-VDOM Traffic
```

can introduce:

```text
Asymmetric Routing
Session Ownership
Traffic Distribution
Hardware Offload Considerations
```

Always verify the exact platform behavior.

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
# Enter global configuration context.

config system vdom-exception
# Enter VDOM exception configuration.

    edit 1
    # Create/select a VDOM exception entry.

        set object <object>
        # Define the configuration object affected by the exception.

        set scope inclusive
        # Define the synchronization scope for the selected object.

        set vdom "test"
        # Apply the exception to the selected VDOM.

    next
    # Finish the VDOM exception entry.

end
# Exit VDOM exception configuration.
```

> Verify supported objects for the target FortiOS release.

---

# 83. Software Switch / Hardware Switch

```bash
show system switch-interface
# Display software switch and switch-interface configuration.
```

### HA Design Warning

Do not automatically assume that a hardware switch provides interface-level HA monitoring.

---

# 84. HA + DHCP

When gateway redundancy is used, DHCP should normally provide the virtual gateway address where appropriate.

```text
DHCP Gateway
      ↓
192.168.10.1
      ↓
Virtual HA/VRRP Gateway
```

Avoid hard-coding one physical FortiGate IP when the design requires a virtual gateway.

---

# 85. HA Design Best Practices

## Heartbeat

```text
✓ Use multiple heartbeat links
✓ Prefer physically diverse paths
✓ Direct links are excellent
✓ Avoid single points of failure
```

## Management

```text
✓ Use reserved/out-of-band management
✓ Use a separate management network
✓ Give each member a unique management IP
```

## Monitoring

```text
✓ Monitor critical WAN links
✓ Monitor important uplinks
✓ Do not monitor every interface blindly
```

## Synchronization

```text
✓ Verify configuration synchronization
✓ Enable session pickup when required
✓ Test failover
```

## Firmware

```text
✓ Backup first
✓ Check upgrade path
✓ Read release notes
✓ Verify HA health
```

---

# 86. HA Failure-Domain Design

### Weak Design

```text
          Same Switch
        +-------------+
        |             |
      FGT-1          FGT-2
        |             |
      HB-1            HB-1
```

### Better Design

```text
FGT-1 HB1 ───────── FGT-2 HB1

FGT-1 HB2 ───────── FGT-2 HB2
     \                /
      Different Physical
          Paths
```

The exact design depends on physical infrastructure.

---

# 87. HA Monitoring Strategy

Do not monitor only heartbeat connectivity.

```text
Heartbeat
    +
WAN
    +
Critical LAN
    +
DMZ
    +
Relevant Health Checks
```

This provides a better representation of actual service availability.

---

# 88. HA vs Link Monitor

### HA Interface Monitoring

Answers:

> "Is this cluster member healthy enough to remain active?"

### Link Monitor

Answers:

> "Is this network path or service reachable?"

They solve different problems.

---

# 89. HA + Link Monitor

Concept:

```text
FortiGate
   |
Link Monitor
   |
Probe Target
   |
Upstream/Internet
```

Failure:

```text
Path Failure
      ↓
Health Failure
      ↓
HA Decision
      ↓
Possible Failover
```

Physical interface state and service reachability are not the same thing.

---

# 90. Production HA Testing Matrix

| Test                       | Expected Result                      |
| -------------------------- | ------------------------------------ |
| Primary power loss         | Secondary takes over                 |
| Primary WAN failure        | Expected failover                    |
| Primary LAN/uplink failure | Expected behavior                    |
| Heartbeat link 1 failure   | Cluster remains healthy              |
| Heartbeat link 2 failure   | Cluster remains healthy              |
| Both heartbeat links fail  | Split-brain behavior verified        |
| Session pickup             | Existing sessions behave as designed |
| Management access          | Both members reachable               |
| SNMP                       | Individual members monitored         |
| Syslog                     | Logs received correctly              |
| FortiAnalyzer              | Connectivity verified                |
| Firmware upgrade           | Supported procedure succeeds         |
| Primary recovery           | Expected role behavior               |
| vCluster failover          | Correct VDOM ownership               |
| FGSP session sync          | Session survives path change         |

---

# 91. Troubleshooting Decision Tree

```text
HA Problem
   |
   +-- Cluster not forming?
   |      |
   |      +-- Model?
   |      +-- Firmware?
   |      +-- Group ID?
   |      +-- Group Name?
   |      +-- Heartbeat?
   |      +-- HA Mode?
   |
   +-- Wrong Primary?
   |      |
   |      +-- Override?
   |      +-- Priority?
   |      +-- Uptime?
   |      +-- Monitored Interfaces?
   |      +-- Serial?
   |
   +-- Failover not happening?
   |      |
   |      +-- Monitor configured?
   |      +-- Interface actually failed?
   |      +-- Heartbeat healthy?
   |      +-- Health-check behavior?
   |
   +-- Sessions dropped?
   |      |
   |      +-- Session Pickup?
   |      +-- Connectionless Pickup?
   |      +-- NAT Pickup?
   |      +-- Expectation Pickup?
   |
   +-- Config mismatch?
          |
          +-- Checksum
          +-- Synchronization
          +-- HA Exceptions
```

---

# 92. NSE Exam Memory Map

## FGCP

```text
Cluster
Election
Primary/Secondary
Configuration Sync
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
show system ha
# Display HA configuration.

get system ha
# Display HA configuration/state.

get system ha status
# Display detailed HA status.

execute ha sync start
# Start HA synchronization.

diagnose sys ha check recalc
# Recalculate/check HA election state.

diagnose sys ha mac
# Display HA MAC information.

diagnose sys ha checksum
# Display HA configuration checksum.

execute ha manage <index> <admin>
# Access another HA member.

diagnose sys session list
# Display active sessions.

diagnose ips share list
# Display shared IPS information.

diagnose ips session list
# Display IPS session information.

get test hasync 50
# Display HA synchronization statistics.

get router info vrrp
# Display VRRP state information.

get system vrrp
# Display system VRRP information.

diagnose hardware deviceinfo nic <interface>
# Display physical NIC information.

show system switch-interface
# Display switch-interface configuration.
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
│   └── Similar concept to preemption
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
├── Session Synchronization
├── Load Balancer
├── Asymmetric Routing
└── Multi-Cluster Designs


VRRP
│
├── Virtual Gateway
├── VRID
├── Priority
├── Preempt
└── Virtual MAC
```

---

# 95. Golden Rules

> **1. FGCP = FortiGate clustering.**

> **2. FGSP = Session synchronization between FortiGates/clusters.**

> **3. VRRP = Virtual gateway redundancy.**

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

> **15. Backup + upgrade path + release notes are mandatory before HA firmware changes.**

---

# 96. Production Golden Architecture

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

Core components:

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

---

# SheynShield — HA Mental Model

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

## Core Distinction

```text
FGCP
→ "Who is my cluster and who is Primary?"

FGSP
→ "Which FortiGate knows this session?"

VRRP
→ "Which device owns the virtual gateway?"

vCluster
→ "Which FortiGate should own this VDOM?"

Session Pickup
→ "Can I continue the session after failover?"

Heartbeat
→ "Are my HA peers alive and synchronized?"

Reserved Management
→ "How do I reach each physical member individually?"
```

---

# SheynShield — Command Annotation Standard

For every CLI command in this  :

```bash
command
# Short English explanation of what the command does.
```

This keeps the document:

```text
Fast to Read
      +
Easy to Search
      +
Easy to Teach
      +
Exam Friendly
      +
GitHub Ready
```

**SheynShield — Engineering Secure Networks**

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

# 🛡️ FortiGate HA Checklist

> **SheynShield | Engineering Secure Networks**
>
> **FortiGate High Availability (HA) — FGCP • FGSP • VRRP • vCluster • VDOM • Session Pickup • HA Operations**
>
> FortiOS Focus: **NSE 4 / NSE 7**
>
> A practical FortiGate HA checklist for deployment, configuration, verification, troubleshooting, failover testing, session synchronization, VDOM HA, FGSP, VRRP and production operations.

---

## 📌 Table of Contents

- [HA Fundamentals](#-1-ha-fundamentals)
- [HA Technologies](#-2-ha-technologies)
- [HA Architecture](#-3-ha-architecture)
- [HA Prerequisites](#-4-ha-prerequisites)
- [HA Configuration](#-5-ha-configuration)
- [HA Election](#-6-ha-election)
- [HA Override](#-7-ha-override)
- [Heartbeat](#-8-heartbeat)
- [Split Brain](#-9-split-brain)
- [Interface Monitoring](#-10-interface-monitoring)
- [Session Pickup](#-11-session-pickup)
- [FGCP](#-12-fgcp)
- [Reserved Management](#-13-reserved-management)
- [VMAC](#-14-ha-virtual-mac)
- [VDOM HA](#-15-vdom-ha-and-vcluster)
- [FGSP](#-16-fgsp)
- [VRRP](#-17-vrrp)
- [Failover](#-18-failover-testing)
- [HA Synchronization](#-19-ha-synchronization)
- [Firmware Upgrade](#-20-ha-firmware-upgrade)
- [Troubleshooting](#-21-ha-troubleshooting)
- [Production Design](#-22-production-design)
- [NSE Exam Checklist](#-23-nse-exam-checklist)
- [Command Reference](#-24-ha-command-reference)
- [Golden Rules](#-25-ha-golden-rules)
- [Final Mental Model](#-26-final-mental-model)
- [SheynShield Resources](#-sheynshield-resources)

---

# 🔥 1. HA Fundamentals

## High Availability

- [ ] Understand the purpose of FortiGate HA.
- [ ] Understand cluster-based redundancy.
- [ ] Understand Primary/Secondary roles.
- [ ] Understand Active/Passive architecture.
- [ ] Understand Active/Active architecture.
- [ ] Understand configuration synchronization.
- [ ] Understand session synchronization.
- [ ] Understand heartbeat communication.
- [ ] Understand HA election.
- [ ] Understand failover behavior.
- [ ] Understand cluster virtual MAC behavior.

### Core Mental Model

```text
                    FortiGate HA
                         │
              ┌──────────┼──────────┐
              │          │          │
             FGCP       FGSP       VRRP
              │          │          │
           Cluster    Session    Gateway
                     Synchronization
````

### Remember

```text
FGCP
→ FortiGate clustering

FGSP
→ Session synchronization

VRRP
→ Virtual gateway redundancy
```

---

# 🧩 2. HA Technologies

| Technology   | Primary Purpose             | Typical Use                                |
| ------------ | --------------------------- | ------------------------------------------ |
| **FGCP**     | Native FortiGate clustering | FortiGate HA                               |
| **FGSP**     | Session synchronization     | Standalone FortiGates / asymmetric designs |
| **VRRP**     | Gateway redundancy          | Independent devices                        |
| **vCluster** | VDOM-level HA distribution  | Active-Active / multi-VDOM                 |

### Technology Checklist

* [ ] Know FGCP.
* [ ] Know FGSP.
* [ ] Know VRRP.
* [ ] Know vCluster.
* [ ] Understand the difference between FGCP and FGSP.
* [ ] Understand the difference between FGCP and VRRP.
* [ ] Understand when FGSP is preferred over FGCP.
* [ ] Understand how vCluster relates to VDOMs.

---

# 🏗️ 3. HA Architecture

## Active-Passive

```text
                Traffic
                   │
             +-----+-----+
             │           │
           FGT-1       FGT-2
          PRIMARY     SECONDARY
             │
          Traffic
```

### Checklist

* [ ] One unit processes traffic actively.
* [ ] Secondary remains available for failover.
* [ ] Configuration synchronization is verified.
* [ ] Session synchronization is evaluated.
* [ ] Failover behavior is tested.
* [ ] Recovery behavior is understood.

---

## Active-Active

```text
                 Traffic
                    │
             +------+------+
             │             │
           FGT-1         FGT-2
           ACTIVE        ACTIVE
```

### Checklist

* [ ] Understand FGCP Active-Active.
* [ ] Understand traffic distribution.
* [ ] Understand session ownership.
* [ ] Understand vCluster interaction.
* [ ] Evaluate asymmetric routing.
* [ ] Evaluate NPU/offload behavior.
* [ ] Verify platform-specific limitations.

> **Important:** Active-Active does not mean two completely independent firewalls. FGCP still controls cluster behavior.

---

# 🧱 4. HA Prerequisites

## Hardware

* [ ] Verify FortiGate models.
* [ ] Verify platform compatibility.
* [ ] Verify hardware configuration.
* [ ] Verify supported HA topology.
* [ ] Verify interface availability.

## FortiOS

* [ ] Verify FortiOS versions.
* [ ] Verify upgrade/downgrade compatibility.
* [ ] Read release notes.
* [ ] Verify feature compatibility.
* [ ] Verify supported HA behavior for the target release.

## Licensing

* [ ] Register devices.
* [ ] Activate required licenses.
* [ ] Verify FortiGuard services where required.
* [ ] Verify support entitlement.

## Network

* [ ] Connect heartbeat interfaces.
* [ ] Provide redundant heartbeat paths.
* [ ] Verify L2 connectivity where required.
* [ ] Verify management connectivity.
* [ ] Verify monitored interfaces.
* [ ] Verify upstream topology.

## Backup

* [ ] Backup the configuration.
* [ ] Store backup outside the FortiGate.
* [ ] Verify backup integrity.
* [ ] Prepare rollback procedure.

---

# ⚙️ 5. HA Configuration

## Basic Configuration

```bash
config system ha
# Enter the FortiGate HA configuration context.

    set mode a-p
    # Configure Active-Passive HA mode.

    set group-id 2
    # Define the HA group identifier.

    set group-name "FGT-HA"
    # Define the HA cluster name.

    set priority 200
    # Configure the device HA priority.

    set override enable
    # Allow the preferred device to reclaim the Primary role.

    set hbdev "port3" 50 "port4" 50
    # Configure redundant heartbeat interfaces and priorities.

end
# Exit HA configuration.
```

### Configuration Checklist

* [ ] Configure HA mode.
* [ ] Configure group ID.
* [ ] Configure group name.
* [ ] Configure device priority.
* [ ] Decide whether override is required.
* [ ] Configure heartbeat interfaces.
* [ ] Configure interface monitoring.
* [ ] Configure session pickup if required.
* [ ] Configure reserved management if required.
* [ ] Verify synchronization.

---

# 🗳️ 6. HA Election

HA election determines which cluster member becomes Primary.

### Important Factors

Depending on FortiOS version and configuration, election behavior can involve factors such as:

```text
HA Health
   ↓
Monitored Interfaces
   ↓
Uptime
   ↓
Priority
   ↓
Serial Number
```

### Checklist

* [ ] Understand HA election logic.
* [ ] Understand device priority.
* [ ] Understand uptime influence.
* [ ] Understand monitored interface influence.
* [ ] Understand serial number tie-breaking.
* [ ] Verify the actual election result.
* [ ] Avoid assuming "highest priority always wins."

### Priority

```text
FGT-1 = 200
FGT-2 = 100

FGT-1
  ↓
Higher Preference
```

> **Exam Tip:** Higher priority means higher preference when priority participates in the election.

---

# 🔄 7. HA Override

Override controls whether a preferred unit can reclaim Primary after recovery.

### Override Enabled

```text
FGT-1
Priority 200
    ↓
PRIMARY

FGT-1 fails
    ↓
FGT-2 becomes PRIMARY

FGT-1 returns
    ↓
FGT-1 may reclaim PRIMARY
```

### Override Disabled

```text
FGT-1 fails
    ↓
FGT-2 becomes PRIMARY
    ↓
FGT-1 returns
    ↓
FGT-2 may remain PRIMARY
```

### Checklist

* [ ] Understand override.
* [ ] Decide whether automatic role reclamation is desirable.
* [ ] Consider maintenance impact.
* [ ] Consider unnecessary role changes.
* [ ] Test recovery behavior.

```bash
config system ha
# Enter HA configuration.

    set override enable
    # Enable HA override behavior.

end
# Exit HA configuration.
```

---

# 💓 8. Heartbeat

Heartbeat interfaces carry HA control and synchronization information.

### Heartbeat Responsibilities

* [ ] HA peer communication.
* [ ] Cluster state.
* [ ] Election information.
* [ ] Configuration synchronization.
* [ ] Session synchronization.
* [ ] Health information.

### Recommended Design

```text
FGT-1 port3 ───────── FGT-2 port3
FGT-1 port4 ───────── FGT-2 port4
```

### Checklist

* [ ] Use multiple heartbeat interfaces.
* [ ] Use physically diverse paths where possible.
* [ ] Prefer direct connections where appropriate.
* [ ] Avoid a single heartbeat failure domain.
* [ ] Verify link speed.
* [ ] Verify errors.
* [ ] Verify interface state.
* [ ] Verify heartbeat health.

### Heartbeat Configuration

```bash
config system ha
# Enter HA configuration.

    set hbdev "port3" 50 "port4" 50
    # Configure redundant heartbeat interfaces.

end
# Exit HA configuration.
```

---

# ⚠️ 9. Split Brain

Split brain occurs when HA members lose reliable peer communication and independently attempt to operate as Primary.

```text
          Heartbeat Failure
                 │
        +--------+--------+
        │                 │
      FGT-1             FGT-2
      PRIMARY           PRIMARY
```

### Prevention Checklist

* [ ] Use redundant heartbeat interfaces.
* [ ] Use physically diverse paths.
* [ ] Avoid a single L2 failure domain.
* [ ] Verify heartbeat reliability.
* [ ] Test heartbeat failure.
* [ ] Understand expected behavior when all heartbeat paths fail.
* [ ] Document recovery procedure.

### Golden Rule

```text
Redundant Heartbeat
        +
Physical Diversity
        +
Correct Monitoring
        ↓
Lower Split-Brain Risk
```

---

# 👁️ 10. Interface Monitoring

Heartbeat health is not the same as service health.

```text
Heartbeat Health
      ≠
Service Health
```

### Monitor Critical Interfaces

* [ ] WAN.
* [ ] Internet uplink.
* [ ] Critical LAN uplink.
* [ ] DMZ.
* [ ] Important service interfaces.
* [ ] Interfaces whose failure should influence HA.

```bash
config system ha
# Enter HA configuration.

    set monitor "wan1" "dmz"
    # Monitor critical interfaces for HA health decisions.

end
# Exit HA configuration.
```

### Checklist

* [ ] Identify critical failure domains.
* [ ] Monitor critical interfaces.
* [ ] Avoid blindly monitoring every interface.
* [ ] Test interface failure.
* [ ] Test service failure separately.
* [ ] Verify expected HA behavior.

---

# 🔁 11. Session Pickup

Session pickup allows supported session state to be synchronized between HA members.

### Without Session Pickup

```text
Client
  ↓
FGT-1
  ↓
Session
  ↓
FGT-1 Failure
  ↓
FGT-2
  ↓
Session May Need Re-establishment
```

### With Session Pickup

```text
Client
  ↓
FGT-1
  ↓
Session State
  ↓
Synchronization
  ↓
FGT-2
  ↓
Failover
  ↓
Session Can Continue
```

### Configuration

```bash
config system ha
# Enter HA configuration.

    set session-pickup enable
    # Enable supported session state synchronization.

end
# Exit HA configuration.
```

### Session Pickup Checklist

* [ ] Enable session pickup when required.
* [ ] Understand supported session types.
* [ ] Evaluate TCP sessions.
* [ ] Evaluate UDP/connectionless sessions.
* [ ] Evaluate NAT sessions.
* [ ] Evaluate expectation sessions.
* [ ] Test real application sessions.
* [ ] Test failover during active traffic.

---

# 🧩 12. FGCP

## FortiGate Clustering Protocol

FGCP is the native FortiGate HA clustering mechanism.

### FGCP Responsibilities

* [ ] Cluster membership.
* [ ] Primary/Secondary election.
* [ ] Configuration synchronization.
* [ ] HA state.
* [ ] Session synchronization.
* [ ] Failover.
* [ ] Virtual MAC behavior.
* [ ] vCluster behavior.

```text
             FGCP
               │
       +-------+-------+
       │               │
     FGT-1           FGT-2
    PRIMARY         SECONDARY
```

### Remember

```text
FGCP = Cluster
```

---

# 🛠️ 13. Reserved Management

Reserved management allows individual access to cluster members.

```text
             Management Network
                    │
          +---------+---------+
          │                   │
        FGT-1               FGT-2
      10.10.10.11         10.10.10.12
```

### Use Cases

* [ ] Individual member administration.
* [ ] SNMP.
* [ ] Syslog.
* [ ] FortiAnalyzer.
* [ ] FortiCloud.
* [ ] NetFlow.
* [ ] sFlow.
* [ ] Remote authentication.
* [ ] Out-of-band troubleshooting.

### Design Checklist

* [ ] Use a separate management network where appropriate.
* [ ] Assign unique management IPs.
* [ ] Verify routing to management network.
* [ ] Verify gateway.
* [ ] Verify administrative access.
* [ ] Verify individual member access.

### HA Direct

```bash
config system ha
# Enter HA configuration.

    set ha-mgmt-status enable
    # Enable reserved HA management interfaces.

    set ha-direct enable
    # Allow supported services to use the HA management path.

end
# Exit HA configuration.
```

---

# 🖥️ 14. HA Virtual MAC

The HA virtual MAC provides the cluster interface identity.

```text
Before Failover:

Client
  ↓
HA VMAC
  ↓
FGT-1

After Failover:

Client
  ↓
HA VMAC
  ↓
FGT-2
```

### Checklist

* [ ] Understand physical MAC.
* [ ] Understand virtual MAC.
* [ ] Understand cluster identity.
* [ ] Understand ARP/FDB convergence.
* [ ] Verify VMAC during failover.
* [ ] Verify downstream switch learning.

### Troubleshooting

```bash
diagnose sys ha mac
# Display HA-related MAC information.

diagnose hardware deviceinfo nic <interface>
# Display physical NIC information and interface details.
```

---

# 🧱 15. VDOM HA and vCluster

## VDOM

VDOM provides logical firewall separation.

```text
Physical FortiGate
       │
   +---+---+---+
   │   │   │
VDOM-A VDOM-B VDOM-C
```

### VDOM Checklist

* [ ] Understand VDOM isolation.
* [ ] Understand VDOM-specific routing.
* [ ] Understand VDOM-specific policies.
* [ ] Understand interface ownership.
* [ ] Understand administrative separation.
* [ ] Understand VDOM links.
* [ ] Understand VDOM HA.

---

## vCluster

vCluster allows HA behavior to be distributed at the VDOM level.

```text
FGT-1                     FGT-2
------                    ------
VDOM-A → PRIMARY          VDOM-A → SECONDARY
VDOM-B → SECONDARY        VDOM-B → PRIMARY
```

### Checklist

* [ ] Understand vCluster.
* [ ] Understand per-VDOM role distribution.
* [ ] Understand virtual cluster priority.
* [ ] Understand VDOM monitoring.
* [ ] Understand override behavior.
* [ ] Evaluate Active-Active requirements.
* [ ] Evaluate NPU/offload implications.

### Concept

```text
VDOM
  +
vCluster
  +
FGCP
  ↓
Per-VDOM HA Role Distribution
```

---

# 🔀 16. FGSP

## FortiGate Session Life Support Protocol

FGSP is not the same as FGCP.

```text
FGCP
→ FortiGate Cluster

FGSP
→ Session Synchronization
```

### Typical Architecture

```text
                 Load Balancer
                /             \
               /               \
            FGT-1             FGT-2
          Standalone         Standalone
               \               /
                \             /
                  FGSP Sync
```

### FGSP Use Cases

* [ ] Load balancer environments.
* [ ] Asymmetric routing.
* [ ] Standalone FortiGates.
* [ ] Multiple FortiGate clusters.
* [ ] Distributed security architectures.
* [ ] Session synchronization between independent devices.

### FGCP vs FGSP

| Feature                       | FGCP     | FGSP            |
| ----------------------------- | -------- | --------------- |
| Native HA cluster             | ✅        | ❌               |
| Primary/Secondary election    | ✅        | ❌               |
| Configuration synchronization | ✅        | ❌               |
| Session synchronization       | ✅        | ✅               |
| Standalone FortiGates         | ❌        | ✅               |
| Load-balancer architecture    | Possible | Common          |
| Asymmetric routing            | Limited  | Strong use case |
| vCluster                      | ✅        | ❌               |

### Golden Rule

```text
FGCP = Cluster

FGSP = Session Synchronization
```

---

# 🌐 17. VRRP

## Virtual Router Redundancy Protocol

VRRP provides virtual gateway redundancy between independent devices.

```text
             Clients
                │
        Virtual Gateway
           192.168.1.1
                │
        +-------+-------+
        │               │
      FGT-1           FGT-2
    Priority 255     Priority 200
      MASTER          BACKUP
```

### VRRP Checklist

* [ ] Understand VRID.
* [ ] Understand virtual IP.
* [ ] Understand priority.
* [ ] Understand Master/Backup.
* [ ] Understand preemption.
* [ ] Understand virtual MAC.
* [ ] Understand VRRP monitoring.
* [ ] Understand that VRRP does not provide FGCP configuration synchronization.

### VRRP Priority

```text
FGT-1 = 255
FGT-2 = 200

FGT-1
  ↓
MASTER
```

### Core Distinction

```text
FGCP
→ FortiGate HA Cluster

VRRP
→ Virtual Gateway Redundancy
```

---

# 🧪 18. Failover Testing

## Controlled Failover

* [ ] Test primary power failure.
* [ ] Test primary reboot.
* [ ] Test monitored WAN failure.
* [ ] Test critical uplink failure.
* [ ] Test heartbeat link 1 failure.
* [ ] Test heartbeat link 2 failure.
* [ ] Test both heartbeat links.
* [ ] Test session pickup.
* [ ] Test management access.
* [ ] Test logging.
* [ ] Test monitoring.
* [ ] Test primary recovery.
* [ ] Test override behavior.

### Failover Flow

```text
Failure
   ↓
Detection
   ↓
Election
   ↓
Role Transition
   ↓
VMAC/ARP Convergence
   ↓
Traffic Forwarding
```

### Important

Do not assume that reducing heartbeat timers alone creates faster application recovery.

---

# 🔄 19. HA Synchronization

## Configuration Synchronization

* [ ] Verify cluster membership.
* [ ] Verify configuration synchronization.
* [ ] Verify checksum.
* [ ] Identify synchronization exceptions.
* [ ] Check HA logs.
* [ ] Check firmware compatibility.
* [ ] Check heartbeat health.

### Checksum

```bash
diagnose sys ha checksum
# Display HA configuration checksum information.
```

### Synchronization

```bash
execute ha sync start
# Start HA synchronization.
```

### Status

```bash
get system ha status
# Display detailed HA cluster status.
```

### Checklist

* [ ] Primary and Secondary are visible.
* [ ] Cluster state is healthy.
* [ ] Heartbeat is healthy.
* [ ] Configuration is synchronized.
* [ ] Checksums are consistent where expected.
* [ ] Session synchronization is healthy.
* [ ] Monitored interfaces are healthy.

---

# 💾 20. HA Firmware Upgrade

## Before Upgrade

* [ ] Backup configuration.
* [ ] Verify FortiOS upgrade path.
* [ ] Read release notes.
* [ ] Verify HA compatibility.
* [ ] Verify FGSP compatibility if used.
* [ ] Verify session synchronization behavior.
* [ ] Verify rollback plan.
* [ ] Schedule maintenance window.

### Uninterruptible Upgrade

Conceptually:

```text
FGT-1 Upgrade
      ↓
FGT-2 Processes Traffic
      ↓
FGT-1 Returns
      ↓
FGT-2 Upgrade
```

### Checklist

* [ ] Confirm upgrade procedure.
* [ ] Confirm supported upgrade path.
* [ ] Confirm HA state before upgrade.
* [ ] Confirm synchronization.
* [ ] Upgrade according to supported procedure.
* [ ] Verify cluster after upgrade.
* [ ] Verify session behavior.
* [ ] Verify logs.
* [ ] Verify management access.

> **Production Rule:** Never treat HA firmware upgrade as simply "upload firmware to both units."

---

# 🔍 21. HA Troubleshooting

## Step 1 — Cluster Not Forming

* [ ] Verify FortiGate model.
* [ ] Verify FortiOS version.
* [ ] Verify HA mode.
* [ ] Verify group ID.
* [ ] Verify group name.
* [ ] Verify heartbeat interfaces.
* [ ] Verify physical connectivity.
* [ ] Verify HA configuration.
* [ ] Verify platform compatibility.

---

## Step 2 — Wrong Primary

* [ ] Check priority.
* [ ] Check override.
* [ ] Check uptime.
* [ ] Check monitored interfaces.
* [ ] Check cluster health.
* [ ] Check serial numbers.
* [ ] Recalculate election state if required.

```bash
diagnose sys ha check recalc
# Recalculate/check HA election state.
```

---

## Step 3 — Failover Does Not Happen

* [ ] Verify monitored interface.
* [ ] Verify actual interface failure.
* [ ] Verify HA health.
* [ ] Verify heartbeat.
* [ ] Verify HA mode.
* [ ] Verify failover conditions.
* [ ] Verify health-check behavior.
* [ ] Check logs.

---

## Step 4 — Sessions Drop

* [ ] Check session pickup.
* [ ] Check connectionless session pickup.
* [ ] Check NAT session pickup.
* [ ] Check expectation session pickup.
* [ ] Check synchronization path.
* [ ] Check session ownership.
* [ ] Check application behavior.
* [ ] Test with real traffic.

---

## Step 5 — Configuration Mismatch

* [ ] Check checksum.
* [ ] Check synchronization status.
* [ ] Check HA heartbeat.
* [ ] Check FortiOS compatibility.
* [ ] Check HA exceptions.
* [ ] Check local-only parameters.

---

# 🧰 22. Production Design

## Heartbeat Design

```text
FGT-1 HB1 ───────── FGT-2 HB1

FGT-1 HB2 ───────── FGT-2 HB2
```

### Checklist

* [ ] Multiple heartbeat paths.
* [ ] Physical path diversity.
* [ ] Reliable connectivity.
* [ ] Dedicated links where practical.
* [ ] Avoid unnecessary congestion.

---

## Management Design

```text
              Management Network
                     │
             +-------+-------+
             │               │
           FGT-1           FGT-2
        Reserved-MGMT   Reserved-MGMT
```

### Checklist

* [ ] Separate management network.
* [ ] Unique member IPs.
* [ ] Out-of-band access where appropriate.
* [ ] HA direct configured when required.
* [ ] SNMP verified.
* [ ] Syslog verified.
* [ ] FortiAnalyzer verified.

---

## Monitoring Design

```text
Heartbeat
    +
WAN
    +
Critical LAN
    +
DMZ
    +
Service Health
```

### Checklist

* [ ] Monitor important WAN interfaces.
* [ ] Monitor critical uplinks.
* [ ] Monitor DMZ where required.
* [ ] Avoid excessive monitoring.
* [ ] Test failure domains independently.

---

# 🧪 23. NSE Exam Checklist

## NSE4 — Must Know

* [ ] HA purpose.
* [ ] FGCP.
* [ ] Active-Passive.
* [ ] Active-Active.
* [ ] Primary/Secondary.
* [ ] HA election.
* [ ] Priority.
* [ ] Override.
* [ ] Heartbeat.
* [ ] Monitored interfaces.
* [ ] Session pickup.
* [ ] Virtual MAC.
* [ ] Reserved management.
* [ ] VDOM.
* [ ] vCluster.
* [ ] Basic troubleshooting.

---

## NSE7 — Advanced Understanding

* [ ] HA election behavior.
* [ ] Failure domains.
* [ ] Split-brain scenarios.
* [ ] Session ownership.
* [ ] Session synchronization.
* [ ] FGSP.
* [ ] Asymmetric routing.
* [ ] Load-balancer architectures.
* [ ] VDOM HA.
* [ ] vCluster.
* [ ] NPU/offload considerations.
* [ ] Inter-VDOM traffic.
* [ ] Reserved management.
* [ ] HA direct.
* [ ] Firmware upgrade behavior.
* [ ] Configuration checksum.
* [ ] Advanced troubleshooting.
* [ ] Production HA design.

---

# 🧠 24. HA Command Reference

## Configuration

```bash
show system ha
# Display the HA configuration.

get system ha
# Display HA configuration/state information.

get system ha status
# Display detailed HA cluster status.
```

## Synchronization

```bash
execute ha sync start
# Start HA synchronization.

diagnose sys ha checksum
# Display HA configuration checksum information.

diagnose sys ha check recalc
# Recalculate/check HA election state.

get test hasync 50
# Display HA synchronization test/statistics information.
```

## Virtual MAC

```bash
diagnose sys ha mac
# Display HA virtual and physical MAC information.

diagnose hardware deviceinfo nic <interface>
# Display physical NIC information.
```

## Session Troubleshooting

```bash
diagnose sys session list
# Display active firewall sessions.

diagnose ips session list
# Display IPS-related session information.

diagnose ips share list
# Display shared IPS information.
```

## HA Member Access

```bash
execute ha manage <index> <admin>
# Access another HA member from the current FortiGate.
```

## VRRP

```bash
get router info vrrp
# Display VRRP routing/state information.

get system vrrp
# Display system-level VRRP information.
```

## Interface / Switch

```bash
show system switch-interface
# Display switch-interface configuration.
```

---

# 🚨 25. HA Golden Rules

* [ ] **FGCP = FortiGate clustering.**
* [ ] **FGSP = Session synchronization.**
* [ ] **VRRP = Virtual gateway redundancy.**
* [ ] **Higher priority means higher preference when priority participates in election.**
* [ ] **Override controls preferred-role recovery.**
* [ ] **Heartbeat health is not the same as service health.**
* [ ] **Monitor critical interfaces.**
* [ ] **Use redundant heartbeat paths.**
* [ ] **Use physical path diversity where possible.**
* [ ] **Reserved management provides individual member identity.**
* [ ] **Session pickup and configuration synchronization are different mechanisms.**
* [ ] **vCluster distributes HA behavior at VDOM level.**
* [ ] **VRRP does not provide FGCP-style configuration synchronization.**
* [ ] **FGSP does not replace FGCP when native clustering is required.**
* [ ] **Always verify commands against the target FortiOS version.**
* [ ] **Always verify model-specific limitations.**
* [ ] **Always backup before major HA changes.**
* [ ] **Always verify the firmware upgrade path.**
* [ ] **Always test failover in a controlled environment.**

---

# ⚡ 26. Final Mental Model

```text
                         FORTIGATE HA
                              │
          ┌───────────────────┼───────────────────┐
          │                   │                   │
         FGCP                FGSP                VRRP
          │                   │                   │
       CLUSTER            SESSION SYNC       GATEWAY
          │                                      │
    ┌─────┼─────┐                           VRID/VMAC
    │     │     │
   A-P   A-A  vCluster
    │     │     │
    └─────┴─────┘
          │
    Session Pickup
          │
    Heartbeat / VMAC
          │
   Reserved Management
```

## Core Distinction

```text
FGCP
→ "Who belongs to my FortiGate cluster?"

HA Election
→ "Who should become Primary?"

Override
→ "Should the preferred member reclaim Primary?"

Heartbeat
→ "Are my HA peers communicating?"

Session Pickup
→ "Can supported sessions survive failover?"

VMAC
→ "What cluster identity should downstream devices use?"

vCluster
→ "Which FortiGate should actively own this VDOM?"

FGSP
→ "Which FortiGate knows this session?"

VRRP
→ "Which independent device owns the virtual gateway?"

Reserved Management
→ "How do I reach each physical member individually?"
```

---

# 📝 HA Deployment Checklist

## Pre-Deployment

* [ ] FortiGate models verified.
* [ ] FortiOS versions verified.
* [ ] Upgrade compatibility verified.
* [ ] Licensing verified.
* [ ] Network topology documented.
* [ ] Failure domains documented.
* [ ] Configuration backup created.
* [ ] Rollback plan prepared.

## HA Configuration

* [ ] HA mode configured.
* [ ] Group ID configured.
* [ ] Group name configured.
* [ ] Device priority configured.
* [ ] Override decision documented.
* [ ] Heartbeat interfaces configured.
* [ ] Interface monitoring configured.
* [ ] Session pickup configured.
* [ ] Reserved management configured.

## Validation

* [ ] Cluster formed.
* [ ] Primary identified.
* [ ] Secondary identified.
* [ ] Configuration synchronized.
* [ ] Checksum verified.
* [ ] Heartbeat verified.
* [ ] Monitored interfaces verified.
* [ ] VMAC verified.
* [ ] Session synchronization verified.
* [ ] Management access verified.

## Failover Testing

* [ ] Power failure tested.
* [ ] WAN failure tested.
* [ ] Critical uplink failure tested.
* [ ] Heartbeat failure tested.
* [ ] Session pickup tested.
* [ ] Logging tested.
* [ ] Monitoring tested.
* [ ] Recovery tested.
* [ ] Override behavior tested.

## Production Sign-Off

* [ ] Documentation completed.
* [ ] HA topology diagram completed.
* [ ] Management IPs documented.
* [ ] Failure scenarios documented.
* [ ] Upgrade procedure documented.
* [ ] Rollback procedure documented.
* [ ] Monitoring configured.
* [ ] Alerting configured.
* [ ] Backup verified.
* [ ] Operational team briefed.

---

# 📊 FGCP vs FGSP vs VRRP — Quick Reference

| Concept                    |         FGCP |            FGSP |     VRRP |
| -------------------------- | -----------: | --------------: | -------: |
| FortiGate HA Cluster       |            ✅ |               ❌ |        ❌ |
| Primary Election           |            ✅ |               ❌ |        ✅ |
| Configuration Sync         |            ✅ |               ❌ |        ❌ |
| Session Sync               |            ✅ |               ✅ |        ❌ |
| Standalone Devices         |            ❌ |               ✅ |        ✅ |
| Virtual Gateway            | Cluster VMAC |               ❌ | VRRP VIP |
| vCluster                   |            ✅ |               ❌ |        ❌ |
| Asymmetric Routing         |      Limited | Strong Use Case | Possible |
| Load Balancer Architecture |     Possible |          Common | Possible |
| Fortinet-Specific          |            ✅ |               ✅ |        ❌ |

---

# 🧭 HA Troubleshooting Decision Tree

```text
                     HA PROBLEM
                         │
          ┌──────────────┼──────────────┐
          │              │              │
      Cluster?       Wrong Primary?   Sessions?
          │              │              │
          ▼              ▼              ▼
      Model?          Priority?       Pickup?
      Firmware?       Override?       NAT?
      Group ID?       Uptime?         UDP?
      Heartbeat?      Monitor?        Expectation?
      HA Mode?        Serial?
          │              │              │
          └──────────────┼──────────────┘
                         │
                  Config Mismatch?
                         │
                    ┌────┴────┐
                    │         │
                 Checksum   Sync
                    │         │
                    └────┬────┘
                         │
                    HA Status
                         │
                  Root Cause Analysis
```

---

# 🏁 One-Minute HA Revision

```text
FGCP
│
├── A-P
│   ├── Primary
│   └── Secondary
│
├── A-A
│   └── Traffic Distribution
│
├── Election
│   ├── Health
│   ├── Monitoring
│   ├── Priority
│   ├── Uptime
│   └── Tie-Breaking
│
├── Heartbeat
│   ├── Control
│   ├── State
│   └── Synchronization
│
├── Session Pickup
│   ├── TCP
│   ├── Connectionless
│   ├── NAT
│   └── Expectations
│
├── VMAC
│   └── Cluster Identity
│
├── vCluster
│   └── VDOM-Level HA
│
└── Reserved Management
    └── Individual Member Access


FGSP
│
├── Standalone FortiGates
├── Session Synchronization
├── Load Balancers
├── Asymmetric Routing
└── Distributed Designs


VRRP
│
├── Virtual Gateway
├── VRID
├── Priority
├── Preemption
└── Virtual MAC
```

---

# 🔎 Keywords

`FortiGate HA`
`FortiGate High Availability`
`FortiOS HA`
`FortiGate FGCP`
`FortiGate FGSP`
`FortiGate VRRP`
`FortiGate Active Passive HA`
`FortiGate Active Active HA`
`FortiGate HA configuration`
`FortiGate HA troubleshooting`
`FortiGate HA failover`
`FortiGate HA election`
`FortiGate HA heartbeat`
`FortiGate session pickup`
`FortiGate HA virtual MAC`
`FortiGate reserved management interface`
`FortiGate HA direct`
`FortiGate vCluster`
`FortiGate VDOM HA`
`FortiGate FGSP session synchronization`
`FortiGate asymmetric routing`
`Fortinet HA troubleshooting`
`Fortinet NSE4 HA`
`Fortinet NSE7 HA`
`FortiGate HA checklist`
`FortiOS HA checklist`
`FortiGate HA best practices`

---

# 🏷️ Topics

```text
fortigate
fortios
fortinet
ha
high-availability
fgcp
fgsp
vrrp
vcluster
vdom
network-security
firewall
cybersecurity
nse4
nse7
fortigate-ha
fortigate-troubleshooting
network-engineering
security-engineering
sheynshield
```

---

# 🔗 SheynShield Resources

### 🎥 Video Learning

* [ ] [YouTube — SheynShield](https://youtube.com/@sheynshield)

  * Fortinet NSE content
  * FortiGate troubleshooting
  * Network Security Engineering

### 📚 Notes & Updates

* [ ] [Telegram — SheynShield](https://t.me/sheynshield)

### 💼 Professional Network

* [ ] [LinkedIn — Shayan-heydarikhah](https://linkedin.com/in/shayan-heydarikhah)

### 🐙 Technical Knowledge Base

* [ ] [SheynShield GitHub](https://github.com/Shayan-heydarikhah/sheynshield)

---

# 🛡️ SheynShield

> **SheynShield — Engineering Secure Networks**

**FortiGate • FortiOS • Network Security • Firewall Architecture • Troubleshooting • NSE4 • NSE7**

---

## ⚠️ Version Disclaimer

> FortiOS CLI syntax, available commands, HA behavior, feature support, platform limitations and synchronization capabilities can vary by FortiOS release and FortiGate model.
>
> Always validate production commands against the documentation for the **exact FortiOS version and FortiGate platform** before deployment.


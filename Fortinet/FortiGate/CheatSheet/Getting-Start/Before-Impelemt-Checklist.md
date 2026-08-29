# FortiGate Pre-Deployment & Interface Design Checklist

> **SheynShield | Engineering Secure Networks**
>
> **FortiGate • FortiOS • Interface Design • LACP • Redundant Interface • VLAN • Zone • VWP • DHCP • DNS • Explicit Proxy • Captive Portal • FSSO • Endpoint Control**
>
> **NSE 4 + NSE 7 Engineering Reference**

---

## 📌 What This Checklist Covers

* [ ] FortiGate pre-deployment validation
* [ ] Hardware and transceiver planning
* [ ] FortiOS and firmware validation
* [ ] Hostname, timezone, and NTP
* [ ] Physical interface design
* [ ] Aggregate / LACP interface
* [ ] Redundant interface
* [ ] Virtual Wire Pair
* [ ] Software Switch
* [ ] Internal Switch Mode
* [ ] VLAN interfaces
* [ ] Inter-VLAN routing
* [ ] Zones
* [ ] DHCP
* [ ] DNS
* [ ] Explicit Proxy
* [ ] Captive Portal
* [ ] FSSO
* [ ] Endpoint visibility and control
* [ ] Transparent Mode
* [ ] Production hardening
* [ ] NSE 4 / NSE 7 troubleshooting

---

# 1. 🚀 FortiGate Pre-Deployment Checklist

Before connecting a FortiGate to production:

### Hardware

* [ ] Confirm FortiGate model.
* [ ] Confirm power requirements.
* [ ] Confirm available physical interfaces.
* [ ] Confirm SFP/SFP+/QSFP requirements.
* [ ] Confirm supported transceivers.
* [ ] Confirm fiber/copper type.
* [ ] Confirm expected interface speed.
* [ ] Confirm required link distance.
* [ ] Confirm rack/cabling requirements.
* [ ] Confirm HA hardware requirements if applicable.

### FortiOS

* [ ] Confirm installed FortiOS version.
* [ ] Confirm target FortiOS version.
* [ ] Review release notes.
* [ ] Review known issues.
* [ ] Review resolved issues.
* [ ] Validate firmware upgrade path.
* [ ] Confirm feature support for the target model.
* [ ] Confirm FortiOS compatibility with surrounding infrastructure.

### Licensing

* [ ] Verify FortiGuard subscriptions.
* [ ] Verify required security services.
* [ ] Verify FortiManager requirements.
* [ ] Verify FortiAnalyzer requirements.
* [ ] Verify FortiClient integration requirements.
* [ ] Verify cloud-management requirements.

### Network Design

* [ ] Prepare management IP.
* [ ] Prepare hostname.
* [ ] Prepare VLAN plan.
* [ ] Prepare IP addressing plan.
* [ ] Prepare routing plan.
* [ ] Prepare WAN/ISP information.
* [ ] Prepare LAN/DMZ topology.
* [ ] Define security zones.
* [ ] Define NAT requirements.
* [ ] Define HA design.
* [ ] Define monitoring/logging architecture.
* [ ] Define rollback plan.

---

# 2. 🗺️ Deployment Design Flow

```text
                    FORTIGATE DEPLOYMENT
                            │
          ┌─────────────────┼─────────────────┐
          ▼                 ▼                 ▼
       Hardware          FortiOS           Design
          │                 │                 │
     ┌────┼────┐       ┌────┼────┐       ┌────┼────┐
     ▼    ▼    ▼       ▼    ▼    ▼       ▼    ▼    ▼
   Ports SFP Power   Version Bugs Path   IP   VLAN Route
                                             │
                                             ▼
                                           HA
                                             │
                                             ▼
                                          Policy
                                             │
                                             ▼
                                         Security
                                             │
                                             ▼
                                         Logging
```

---

# 3. 🔌 Optical Interface Checklist

| Interface Type | Common Speed |
| -------------- | -----------: |
| SFP            |       1 Gb/s |
| SFP+           |      10 Gb/s |
| QSFP           |      40 Gb/s |

> ⚠️ These are common interface/module categories, not universal speed guarantees.

Before deployment:

* [ ] FortiGate model verified.
* [ ] Interface capability verified.
* [ ] Transceiver compatibility verified.
* [ ] Fiber/copper type verified.
* [ ] Speed verified.
* [ ] Distance verified.
* [ ] Switch-side compatibility verified.

### Engineering Rule

```text
FortiGate Model
      +
Interface
      +
Transceiver
      +
Cable
      +
Speed
      +
Distance
```

must be validated as one design.

---

# 4. 🏷️ Hostname Checklist

Configure a meaningful hostname immediately after initial deployment.

Examples:

```text
FGT-HQ-EDGE-01
FGT-DC-FW-01
FGT-BRANCH-01
FGT-DMZ-01
```

Recommended convention:

```text
LOCATION + ROLE + NUMBER
```

Example:

```text
TEH-DC-FW-01
```

Checklist:

* [ ] Location included.
* [ ] Device role included.
* [ ] Unique identifier included.
* [ ] Naming convention documented.
* [ ] Hostname consistent with monitoring systems.

---

# 5. 🕐 Timezone & NTP Checklist

Correct time is critical for:

* [ ] Firewall logs
* [ ] Authentication
* [ ] Certificates
* [ ] VPN
* [ ] FortiAnalyzer correlation
* [ ] FortiManager operations
* [ ] SIEM correlation
* [ ] Incident investigation
* [ ] Troubleshooting
* [ ] Scheduled operations

### Mental Model

```text
NTP
 ↓
Time Synchronization
 ↓
Accurate Timestamps
 ↓
Log Correlation
 ↓
Incident Investigation
```

### Conceptual CLI

```cli
config system global
    set timezone <TIMEZONE_ID>
end
```

### NTP

```cli
config system ntp
    set type custom

    config ntpserver
        edit 1
            set server <NTP_SERVER>
        next
    end
end
```

### Checklist

* [ ] Timezone configured.
* [ ] NTP enabled.
* [ ] Approved NTP server configured.
* [ ] NTP reachability verified.
* [ ] Time synchronization verified.
* [ ] FortiAnalyzer/SIEM time correlation verified.

---

# 6. 🧠 NSE 7 — Why Time Synchronization Matters

Consider:

```text
FortiGate       10:32
Windows AD      10:36
SIEM            10:29
Endpoint        10:34
```

Incident correlation becomes unreliable.

A security investigation should instead have:

```text
FortiGate
FortiAnalyzer
FortiManager
AD
DNS
Proxy
SIEM
Endpoint
      │
      ▼
Common Time Reference
      │
      ▼
Reliable Correlation
```

> **NSE 7 mindset:** NTP is part of security operations, not merely system administration.

---

# 7. 🔗 Configuration Reference / Dependency Checklist

Before modifying or deleting an object:

```text
Object
  │
  ├── Policy
  ├── VIP
  ├── Route
  ├── Zone
  ├── Interface
  └── Other Configuration
```

Checklist:

* [ ] Check object references.
* [ ] Identify dependent objects.
* [ ] Review policies.
* [ ] Review routes.
* [ ] Review VIPs.
* [ ] Review zones.
* [ ] Review related interfaces.
* [ ] Remove or modify dependencies first.
* [ ] Re-check references.
* [ ] Perform the change.

### Reference Mental Model

```text
Ref = Configuration Dependency
```

If:

```text
Ref = 0
```

the object may have no current configuration references.

If:

```text
Ref > 0
```

something is using it.

> **NSE 7:** Think of FortiGate configuration as a dependency graph, not a collection of isolated objects.

---

# 8. 🧩 FortiGate Interface Types

Common interface architectures include:

```text
INTERFACE
   │
   ├── Physical
   ├── VLAN
   ├── Aggregate
   ├── Redundant
   ├── Software Switch
   ├── Virtual Wire Pair
   ├── Loopback
   └── Other Logical Interfaces
```

Before creating an interface, ask:

* [ ] Does it require Layer 3 routing?
* [ ] Does it require Layer 2 forwarding?
* [ ] Is VLAN tagging required?
* [ ] Is link aggregation required?
* [ ] Is link redundancy required?
* [ ] Does it need to be part of a zone?
* [ ] Is it part of an inline security design?

---

# 9. 🏷️ Interface Role Checklist

Common roles:

```text
WAN
LAN
DMZ
Undefined
```

Remember:

```text
Interface Role
      ≠
Security Policy
```

The role primarily affects GUI behavior and defaults.

It does **not** automatically provide Internet access or security permission.

---

# 10. 🌐 WAN Interface Checklist

For WAN interfaces:

* [ ] ISP circuit verified.
* [ ] Physical link verified.
* [ ] IP addressing verified.
* [ ] Gateway verified.
* [ ] DNS verified.
* [ ] Default route verified.
* [ ] NAT requirement verified.
* [ ] Firewall policy verified.
* [ ] SD-WAN requirement evaluated.
* [ ] Health checks evaluated.
* [ ] Failover requirements evaluated.

### Critical Mental Model

```text
WAN Interface
      +
Route
      +
Firewall Policy
      +
NAT where required
      +
Upstream Connectivity
```

=

```text
Working Internet Access
```

---

# 11. 🛣️ DHCP Default Gateway

If a WAN interface receives addressing through DHCP:

```text
ISP DHCP
   │
   ├── IP
   ├── Mask
   └── Gateway
         │
         ▼
      FortiGate
```

A dynamically learned gateway can reduce the need for a manually configured default route in some designs.

### Multi-WAN Warning

For multiple ISPs:

```text
ISP1 ── WAN1 ── Gateway 1
ISP2 ── WAN2 ── Gateway 2
```

Explicitly design:

* [ ] Static routes.
* [ ] SD-WAN.
* [ ] Priority.
* [ ] Health checks.
* [ ] Failover.
* [ ] Return-path behavior.

> Do not blindly depend on automatically learned gateways in a multi-WAN architecture.

---

# 12. ⚡ Aggregate Interface / LACP Checklist

An aggregate interface combines multiple physical links into one logical interface.

```text
              FortiGate
                 │
        ┌────────┼────────┐
        │        │        │
      port1    port2    port3
        │        │        │
        └────────┼────────┘
                 │
              LACP
                 │
                 ▼
               Switch
```

Common standard:

```text
IEEE 802.3ad
```

Modern terminology:

```text
LACP
```

---

# 13. ⚡ LACP Benefits

### Capacity

```text
1G + 1G + 1G
      ↓
Aggregate Interface
      ↓
Higher Aggregate Capacity
```

### Resilience

```text
port1 ── X
port2 ── UP
port3 ── UP
```

Traffic can continue through surviving members.

### Important

Do **not** assume:

```text
2 × 1G
=
one single 2G TCP flow
```

Actual traffic distribution depends on:

* [ ] Hashing.
* [ ] Number of flows.
* [ ] Platform.
* [ ] LACP implementation.
* [ ] Switch configuration.

---

# 14. 🆚 Aggregate vs Redundant Interface

| Characteristic                     | Aggregate             | Redundant  |
| ---------------------------------- | --------------------- | ---------- |
| Multiple interfaces                | ✅                     | ✅          |
| Uses multiple links simultaneously | ✅                     | ❌          |
| Typical purpose                    | Capacity + resilience | Resilience |
| LACP                               | Common                | ❌          |
| Failover                           | ✅                     | ✅          |
| Aggregate throughput               | ✅                     | ❌          |

### Mental Model

```text
AGGREGATE

port1 ─┐
port2 ─┼──► USED TOGETHER
port3 ─┘
```

```text
REDUNDANT

port1 ─► ACTIVE
port2 ─► STANDBY
```

---

# 15. 🚨 LACP Pre-Deployment Checklist

Before creating an aggregate:

* [ ] Physical interfaces identified.
* [ ] Interfaces are in the correct VDOM.
* [ ] Interfaces do not have incompatible existing configuration.
* [ ] No existing IP configuration.
* [ ] No DHCP dependency.
* [ ] No PPPoE dependency.
* [ ] Not already part of another aggregate.
* [ ] Not already part of a redundant interface.
* [ ] Not used as HA heartbeat interfaces.
* [ ] Not referenced by firewall policies.
* [ ] Not referenced by VIPs.
* [ ] Not referenced by IP pools.
* [ ] No conflicting routing dependencies.
* [ ] Switch-side LAG is prepared.
* [ ] VLAN/trunk configuration is correct.

---

# 16. 🔍 LACP Troubleshooting Checklist

If LACP does not form:

```text
FortiGate
   │
   ├── Physical Link?
   ├── Speed?
   ├── Interface State?
   ├── LACP Configuration?
   ├── Correct Members?
   └── Correct Switch LAG?
             │
             ▼
          Switch
```

Check:

* [ ] Physical link.
* [ ] Interface speed.
* [ ] Duplex where applicable.
* [ ] LACP enabled.
* [ ] Correct LACP mode.
* [ ] Correct member interfaces.
* [ ] Correct switch-side LAG.
* [ ] VLAN configuration.
* [ ] Trunk configuration.
* [ ] No port mismatch.
* [ ] LACP state.

---

# 17. 🔄 Redundant Interface Checklist

Use a redundant interface when the primary objective is link resilience rather than simultaneous link utilization.

### Design

```text
port1 ─► ACTIVE
           │
           ▼
       Redundant
           ▲
           │
port2 ─► STANDBY
```

Checklist:

* [ ] Multiple physical links available.
* [ ] Redundancy is the primary requirement.
* [ ] Aggregate throughput is not the requirement.
* [ ] Member interfaces are suitable.
* [ ] Switch-side design supports the topology.
* [ ] Failure behavior is understood.
* [ ] Recovery behavior is tested.

---

# 18. 🔌 Virtual Wire Pair Checklist

A Virtual Wire Pair logically connects two interfaces for inline traffic forwarding.

```text
Network A
   │
   ▼
port3
   │
┌──┴─────────────┐
│ FortiGate VWP  │
└──┬─────────────┘
   │
   ▼
port4
   │
   ▼
Network B
```

Use VWP when FortiGate needs to be inserted inline without using a conventional routed-gateway topology.

---

# 19. 🎯 VWP Use Cases

Possible scenarios:

* [ ] Inline security inspection.
* [ ] Layer 2 insertion.
* [ ] Networks where routed gateway placement is undesirable.
* [ ] Special server/network topologies.
* [ ] Architectures requiring transparent inline security.

### Traffic Model

```text
Ingress
   ↓
VWP Member
   ↓
Firewall Policy
   ↓
Security Inspection
   ↓
VWP Member
   ↓
Egress
```

---

# 20. 🔥 VWP Policy Checklist

Conceptual configuration:

```cli
config system virtual-wire-pair
    edit "VWP-LAN-WEB"
        set member "port3" "port4"
        set wildcard-vlan disable
    next
end
```

Example policy structure:

```cli
config firewall policy
    edit 1
        set name "VWP-LAN-WEB"
        set srcintf "port3"
        set dstintf "port4"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
        set utm-status enable
    next
end
```

Before deployment:

* [ ] VWP members verified.
* [ ] Ingress/egress direction understood.
* [ ] Firewall policy created.
* [ ] Security profiles reviewed.
* [ ] VLAN behavior verified.
* [ ] Traffic path tested.
* [ ] Return path validated.

> Exact syntax and behavior can vary by FortiOS release and topology.

---

# 21. 🧠 VWP — NSE 7 Perspective

Before using VWP, understand:

```text
Where does traffic enter?
        ↓
Which VWP member receives it?
        ↓
Which policy evaluates it?
        ↓
Where is inspection performed?
        ↓
Which interface forwards it?
        ↓
How does return traffic flow?
```

> **Engineering principle:** Design the traffic path first; configure the VWP second.

---

# 22. 🖥️ Software Switch Checklist

A software switch combines interfaces into a logical switching construct.

```text
port1 ─┐
port2 ─┼──► Software Switch
port3 ─┘
             │
             ▼
       Logical Interface
```

Before deployment:

* [ ] Confirm Layer 2 switching requirement.
* [ ] Confirm member interfaces.
* [ ] Confirm subnet design.
* [ ] Confirm performance expectations.
* [ ] Confirm platform capabilities.
* [ ] Verify security policy requirements.

---

# 23. 🆚 Hardware Switch vs Software Switch

| Feature                    | Hardware Switch    | Software Switch               |
| -------------------------- | ------------------ | ----------------------------- |
| Logical interface grouping | ✅                  | ✅                             |
| Switching                  | Hardware-assisted  | Software/logical              |
| ASIC involvement           | Model-dependent    | More software/CPU involvement |
| Typical use                | Simple LAN         | Flexible logical grouping     |
| Performance                | Platform-dependent | Platform-dependent            |

> Exact acceleration depends on the FortiGate platform and configuration.

---

# 24. 🔧 Internal Switch Mode Checklist

Some FortiGate platforms support different internal-switch configurations.

Conceptually:

```text
Switch Mode
     vs
Interface Mode
```

---

## Switch Mode

```text
port1 ─┐
port2 ─┤
port3 ─┼──► internal
port4 ─┘
```

Suitable for:

* [ ] Simple LAN.
* [ ] Single subnet.
* [ ] Basic deployment.

---

## Interface Mode

```text
port1 → 10.1.1.1/24
port2 → 10.2.2.1/24
port3 → 10.3.3.1/24
```

Suitable for:

* [ ] Multiple subnets.
* [ ] DMZ.
* [ ] WAN.
* [ ] Segmentation.
* [ ] Complex topology.

---

# 25. ⚠️ Internal Switch Mode Change

Historical configuration example:

```cli
config system global
    set internal-switch-mode switch
end
```

or:

```cli
config system global
    set internal-switch-mode interface
end
```

Before changing:

* [ ] Take configuration backup.
* [ ] Review interface references.
* [ ] Review policies.
* [ ] Review routes.
* [ ] Review DHCP.
* [ ] Review VLANs.
* [ ] Document current topology.
* [ ] Schedule maintenance.
* [ ] Prepare console/OOB access.
* [ ] Perform the change.
* [ ] Reconfigure affected interfaces.
* [ ] Validate connectivity.

> ⚠️ Treat internal-switch-mode changes as topology-level operations.

---

# 26. 🏷️ VLAN Interface Checklist

A VLAN interface allows FortiGate to terminate a tagged VLAN on a parent interface.

```text
                 Switch
                   │
                 TRUNK
                   │
                   ▼
               FortiGate
                 port4
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
     VLAN 100   VLAN 200   VLAN 300
        │          │          │
      Users      Servers     Guest
```

Before deployment:

* [ ] Parent interface identified.
* [ ] VLAN ID documented.
* [ ] Switch port configured as trunk.
* [ ] Allowed VLAN list verified.
* [ ] IP subnet documented.
* [ ] Gateway address documented.
* [ ] DHCP requirement identified.
* [ ] Firewall policy created.
* [ ] Inter-VLAN requirements documented.

---

# 27. 🔧 VLAN Configuration Example

```cli
config system interface

    edit "VLAN_100"
        set interface "port4"
        set type vlan
        set vlanid 100
        set ip 172.100.1.1 255.255.255.0
        set allowaccess https ping
    next

end
```

Validation:

```text
FortiGate VLAN ID
        =
Switch VLAN ID
```

and:

```text
FortiGate parent interface
        =
Switch trunk connection
```

---

# 28. 🧠 Same VLAN ID ≠ Same Layer 2 Network

Critical concept:

```text
port1
 └── VLAN 300

port2
 └── VLAN 300
```

does **not** automatically mean:

```text
port1.VLAN300
       │
       │ L2 bridge
       X
       │
port2.VLAN300
```

The same VLAN ID on different FortiGate interfaces does not automatically create an internal Layer 2 bridge.

---

# 29. 🔀 Inter-VLAN Routing Checklist

Typical flow:

```text
VLAN 10
   ↓
FortiGate
   ↓
Routing
   ↓
Firewall Policy
   ↓
VLAN 20
```

Before enabling communication:

* [ ] Source interface verified.
* [ ] Destination interface verified.
* [ ] Source subnet verified.
* [ ] Destination subnet verified.
* [ ] Routing table verified.
* [ ] Firewall policy created.
* [ ] Required services defined.
* [ ] NAT requirement evaluated.
* [ ] Logging enabled where required.

### Golden Rule

```text
VLAN Interface
      ≠
Automatic Permission
```

A route does not replace a firewall policy.

---

# 30. 🛡️ Zone Design Checklist

A zone logically groups interfaces.

Example:

```text
USER-ZONE
   │
   ├── VLAN10
   ├── VLAN20
   ├── VLAN30
   └── VLAN40
```

Benefits:

* [ ] Simplify policy design.
* [ ] Reduce repetitive interface combinations.
* [ ] Group interfaces logically.
* [ ] Improve administrative clarity.

But:

```text
ZONE
  ≠
SECURITY POLICY
```

The firewall policy remains the security decision point.

---

# 31. 🔒 Segmentation Checklist

Avoid:

```text
Guest
  ↓
Internal
```

unless explicitly required.

Recommended boundaries:

```text
Users
  X
Servers

Users
  X
Management

Guest
  X
Internal

IoT
  X
Management
```

Checklist:

* [ ] User zone defined.
* [ ] Server zone defined.
* [ ] Guest zone defined.
* [ ] Management zone defined.
* [ ] IoT zone defined where required.
* [ ] Inter-zone policies explicitly designed.
* [ ] Deny-by-default behavior considered.

---

# 32. 🌐 Overlapping Subnet Checklist

Applicable configurations may support overlapping subnets.

Example:

```cli
config system settings
    set allow-subnet-overlap enable
end
```

Potential use cases:

* [ ] Migration.
* [ ] Multi-tenant environments.
* [ ] Overlapping VPN networks.
* [ ] Complex transition architectures.

Before enabling:

* [ ] Confirm business requirement.
* [ ] Understand routing implications.
* [ ] Understand policy implications.
* [ ] Document overlapping networks.
* [ ] Test traffic behavior.
* [ ] Prepare troubleshooting documentation.

> ⚠️ Overlapping addressing should be treated as an architecture decision, not a default setting.

---

# 33. 📡 DHCP Server Checklist

FortiGate can provide DHCP services for LAN/VLAN interfaces.

```text
Client
  │
  │ DHCP
  ▼
FortiGate
  │
  ├── IP Address
  ├── Subnet Mask
  ├── Gateway
  ├── DNS
  └── Lease
```

Checklist:

* [ ] DHCP server enabled.
* [ ] Correct interface selected.
* [ ] Correct address range configured.
* [ ] Gateway configured.
* [ ] DNS configured.
* [ ] Lease time configured.
* [ ] Reserved addresses documented.
* [ ] Excluded addresses documented.
* [ ] DHCP relay requirement evaluated.

---

# 34. 🧷 DHCP Reservation Checklist

Useful for:

* [ ] Printers.
* [ ] Cameras.
* [ ] Servers.
* [ ] Network appliances.
* [ ] IoT devices.
* [ ] Infrastructure devices.

Concept:

```text
MAC Address
     ↓
Reserved IP
     ↓
Predictable Address
```

Before reservation:

* [ ] Confirm client MAC.
* [ ] Confirm reserved IP.
* [ ] Ensure IP is inside/outside the intended DHCP range according to design.
* [ ] Document reservation.
* [ ] Verify lease assignment.

---

# 35. ⏱️ DHCP Lease-Time Checklist

Conceptual CLI:

```cli
config system dhcp server
    edit <SERVER_ID>
        set lease-time <SECONDS>
    next
end
```

### Long Lease

```text
Less DHCP traffic
+
Stable addressing
```

### Short Lease

```text
Faster address recycling
+
More DHCP activity
```

Choose based on:

* [ ] Client behavior.
* [ ] Network size.
* [ ] Address availability.
* [ ] Mobility.
* [ ] IoT/device churn.

---

# 36. 🧹 DHCP Troubleshooting Checklist

Expected DHCP process:

```text
DISCOVER
   ↓
OFFER
   ↓
REQUEST
   ↓
ACK
```

Check:

* [ ] Client connected to correct VLAN.
* [ ] DHCP server enabled.
* [ ] Correct DHCP scope.
* [ ] Address pool available.
* [ ] Exclusions/reservations correct.
* [ ] DHCP relay if applicable.
* [ ] Gateway correct.
* [ ] DNS correct.
* [ ] Lease state.
* [ ] Firewall/security controls.

---

# 37. 🧭 DNS Design Checklist

FortiGate may use DNS for:

* [ ] Name resolution.
* [ ] FQDN objects.
* [ ] Security services.
* [ ] FortiGuard communication.
* [ ] Administrative functions.
* [ ] DNS forwarding.

Enterprise design:

```text
Clients
   │
   ▼
Internal DNS
   │
   ▼
Approved Forwarders
   │
   ▼
External / Security DNS
```

Before deployment:

* [ ] Internal DNS identified.
* [ ] Forwarders identified.
* [ ] DNS resolution tested.
* [ ] Split-DNS requirements evaluated.
* [ ] FQDN dependencies documented.

---

# 38. 🔀 DNS Forwarder Checklist

Concept:

```text
Client
  │
  │ DNS Query
  ▼
FortiGate
  │
  │ Forward
  ▼
Internal DNS
```

Example:

```cli
config system dns-server
    edit "port3"
        set mode forward-only
    next
end
```

Validate:

* [ ] Listening interface.
* [ ] DNS server reachability.
* [ ] Forwarding behavior.
* [ ] Client DNS settings.
* [ ] Internal domain resolution.
* [ ] External domain resolution.

---

# 39. 🌐 DHCP + DNS Architecture

Clean branch design:

```text
Client
   │
   ▼
DHCP
   │
   ├── IP
   ├── Gateway
   └── DNS
          │
          ▼
       FortiGate
          │
          ▼
     Internal DNS
```

Checklist:

* [ ] DHCP scope documented.
* [ ] Gateway documented.
* [ ] DNS servers documented.
* [ ] Internal DNS reachable.
* [ ] External DNS resolution tested.
* [ ] FQDN applications tested.

---

# 40. 🌐 Explicit Web Proxy Checklist

Explicit proxy requires clients to send web traffic through the proxy.

```text
Client
   │
   │ Proxy Request
   ▼
FortiGate Explicit Proxy
   │
   ▼
Internet
```

Checklist:

* [ ] Proxy interface selected.
* [ ] Proxy port configured.
* [ ] Client proxy settings configured.
* [ ] Authentication requirement defined.
* [ ] PAC requirement evaluated.
* [ ] Web filtering evaluated.
* [ ] SSL inspection requirement evaluated.
* [ ] Logging configured.
* [ ] Bypass requirements documented.

---

# 41. 📜 PAC File Checklist

Without PAC:

```text
Admin
  ↓
Manual Client Configuration
```

With PAC:

```text
Client
  ↓
PAC
  ↓
Automatic Proxy Selection
```

Checklist:

* [ ] PAC location defined.
* [ ] Proxy address defined.
* [ ] Proxy port defined.
* [ ] Bypass rules defined.
* [ ] Internal destinations considered.
* [ ] Direct-access exceptions documented.

---

# 42. 🔗 Proxy Chaining

Concept:

```text
Client
   ↓
FortiGate Explicit Proxy
   ↓
Upstream Proxy
   ↓
Internet
```

Evaluate:

* [ ] Upstream proxy address.
* [ ] Authentication.
* [ ] Routing.
* [ ] DNS.
* [ ] Failure behavior.
* [ ] Logging.
* [ ] Security inspection responsibilities.

---

# 43. 🔐 Captive Portal Checklist

Captive portal can require users to authenticate before normal network access.

```text
Guest
  ↓
Network Access
  ↓
Captive Portal
  ↓
Authentication
  ↓
Policy Decision
  ↓
Internet
```

Checklist:

* [ ] Guest interface defined.
* [ ] Guest VLAN defined.
* [ ] Authentication method defined.
* [ ] User/group mapping defined.
* [ ] Portal configuration tested.
* [ ] Firewall policy configured.
* [ ] Internet access policy configured.
* [ ] Internal network isolation configured.
* [ ] Logging enabled.

---

# 44. 👤 FSSO Checklist

**Fortinet Single Sign-On (FSSO)** allows FortiGate to associate users with IP addresses using supported identity information.

Typical architecture:

```text
Windows Users
      │
      ▼
Active Directory
      │
      ▼
FSSO Collector
      │
      ▼
FortiGate
      │
      ▼
Identity-Based Policy
```

Checklist:

* [ ] AD connectivity verified.
* [ ] FSSO architecture identified.
* [ ] Collector/agent requirements verified.
* [ ] FortiGate connectivity verified.
* [ ] User-to-IP mapping verified.
* [ ] Group membership verified.
* [ ] Identity-based policy configured.
* [ ] Logging verified.

---

# 45. 🔌 FSSO Troubleshooting

A commonly encountered FSSO-related port is:

```text
TCP/8002
```

> Port usage depends on the FSSO component and deployment architecture. Verify the exact FortiOS/FSSO release documentation.

When identity is missing:

```text
AD
 ↓
FSSO Collector
 ↓
FortiGate
 ↓
User/IP Mapping
 ↓
Group
 ↓
Firewall Policy
```

Check each layer.

Where applicable:

```cli
execute fsso refresh
```

Then verify:

```text
User
 ↓
IP
 ↓
Group
 ↓
Policy
```

---

# 46. 👥 Identity-Based Firewall Policy

Traditional policy:

```text
Source IP
      ↓
Destination
      ↓
Service
```

Identity-based policy can add:

```text
User
+
Group
+
Source Interface
+
Destination
+
Service
```

Checklist:

* [ ] User identity available.
* [ ] Group mapping correct.
* [ ] Source interface correct.
* [ ] Destination correct.
* [ ] Required services defined.
* [ ] Logging enabled.

---

# 47. 🚨 Captive Portal Security Checklist

Avoid:

```text
Guest
  ↓
ALL
  ↓
Internet
```

Prefer:

```text
Guest
 ↓
Authentication
 ↓
User/Group
 ↓
Firewall Policy
 ↓
Security Profiles
 ↓
Internet
```

Consider:

* [ ] Web Filtering.
* [ ] DNS Filtering.
* [ ] Application Control.
* [ ] Antivirus.
* [ ] IPS.
* [ ] Logging.
* [ ] Rate limiting where appropriate.
* [ ] Internal network isolation.

---

# 48. 🖥️ Endpoint Control Checklist

Endpoint-aware architectures may include:

```text
Device Detection
+
Device Identification
+
FortiClient Integration
+
Endpoint Information
+
Access Control
```

Potential devices:

* [ ] Workstations.
* [ ] Laptops.
* [ ] Mobile devices.
* [ ] Printers.
* [ ] IoT.
* [ ] Servers.
* [ ] Network devices.

---

# 49. 📡 Device Detection Checklist

Possible mechanisms include:

```text
Passive Detection
+
Active Scanning
```

A practical approach:

```text
Passive Visibility
       +
Active Verification
```

Validate:

* [ ] Device detection enabled where required.
* [ ] Appropriate interfaces selected.
* [ ] Active scanning requirements understood.
* [ ] Device identity verified.
* [ ] False-positive considerations evaluated.

---

# 50. 🛡️ FortiClient Integration Checklist

Concept:

```text
Endpoint
   │
   ▼
FortiClient
   │
   ▼
FortiGate
   │
   ├── Device Identity
   ├── Endpoint Information
   └── Access Control
```

Validate:

* [ ] FortiClient deployment.
* [ ] FortiGate integration.
* [ ] Endpoint visibility.
* [ ] Device identity.
* [ ] Security policy integration.
* [ ] Endpoint control requirements.

---

# 51. 🚫 Endpoint Blocking Design

Concept:

```text
Endpoint
   ↓
Threat Detection
   ↓
Risk Decision
   ↓
Allow / Warn / Block
```

Before aggressive blocking:

* [ ] Detection source understood.
* [ ] False-positive risk evaluated.
* [ ] Business impact evaluated.
* [ ] Exception process defined.
* [ ] Logging enabled.
* [ ] Recovery process documented.

---

# 52. ➕ Secondary IP Checklist

Concept:

```text
Interface
 ├── Primary IP
 ├── Secondary IP
 └── Secondary IP
```

Potential use cases:

* [ ] Migration.
* [ ] Legacy applications.
* [ ] Multiple logical subnets.
* [ ] Temporary addressing.
* [ ] Transition architecture.

Avoid using secondary IPs as a substitute for proper VLAN segmentation when VLAN architecture is more appropriate.

---

# 53. 🌉 Transparent Mode Checklist

Transparent Mode can place FortiGate into a Layer 2 security architecture.

Concept:

```text
Network A
    │
    ▼
FortiGate
Layer 2 Security
    │
    ▼
Network B
```

Security functions may still include:

* [ ] Antivirus.
* [ ] IPS.
* [ ] Web Filtering.
* [ ] Anti-Spam where applicable.

Features that depend on routed/NAT architecture may be unavailable or behave differently.

Before deployment:

* [ ] Confirm Layer 2 requirement.
* [ ] Confirm VLAN architecture.
* [ ] Confirm forwarding-domain design.
* [ ] Confirm STP requirements.
* [ ] Confirm management access.
* [ ] Confirm feature limitations.
* [ ] Test failover/traffic behavior.

---

# 54. 🧭 Interface Design Decision Tree

Before creating an interface:

```text
What traffic architecture is required?
                │
       ┌────────┴────────┐
       ▼                 ▼
     Layer 3           Layer 2
       │                 │
       ▼                 ▼
    NAT Mode       Transparent/VWP
       │                 │
   ┌───┼────┐        ┌───┼────┐
   ▼   ▼    ▼        ▼   ▼    ▼
 VLAN Zone Route    VWP STP VLAN
   │
   ├── Aggregate?
   ├── Redundant?
   ├── DHCP?
   └── Secondary IP?
```

---

# 55. 🧠 The 10 Questions Before Creating Any Interface

Ask:

* [ ] Why does this interface exist?
* [ ] What traffic enters?
* [ ] What traffic leaves?
* [ ] Who owns the gateway?
* [ ] Where does routing occur?
* [ ] Where is NAT performed?
* [ ] Where is the security policy enforced?
* [ ] How is redundancy achieved?
* [ ] How is Layer 2 handled?
* [ ] How will failure be detected and troubleshot?

> **NSE 7 Engineering Rule:**
> Design the traffic flow first. Configure the interface second.

---

# 56. 🔎 NSE 7 Troubleshooting Matrix

| Problem                     | First Investigation Areas         |
| --------------------------- | --------------------------------- |
| LACP not forming            | Physical link, LACP, switch LAG   |
| VLAN unavailable            | Trunk, VLAN ID, parent interface  |
| Same VLAN not communicating | Interface architecture, L2 design |
| VWP traffic blocked         | VWP members, firewall policy      |
| Layer 2 loop                | STP, topology                     |
| DHCP unavailable            | VLAN, DHCP scope, relay           |
| DNS unavailable             | DNS server, forwarding            |
| FSSO user missing           | AD, collector, connectivity       |
| Captive portal missing      | Interface, authentication, policy |
| Guest reaches internal LAN  | Zone, policy, segmentation        |
| Endpoint not detected       | Device detection, telemetry       |
| Multiple ISP issue          | Routing, SD-WAN                   |
| Interface change rejected   | Configuration references          |
| Management inaccessible     | Allowaccess, routing, policy      |

---

# 57. 🛡️ Production Hardening Checklist

## Management

* [ ] Change default hostname.
* [ ] Configure timezone.
* [ ] Configure NTP.
* [ ] Restrict management interfaces.
* [ ] Disable unnecessary administrative protocols.
* [ ] Use HTTPS.
* [ ] Use SSH where required.
* [ ] Use trusted certificates.
* [ ] Enable MFA where supported.
* [ ] Restrict administrative source addresses.
* [ ] Verify console/OOB access.

## Interfaces

* [ ] Document every interface.
* [ ] Document interface role.
* [ ] Document IP/subnet.
* [ ] Document VLAN ID.
* [ ] Document switch port.
* [ ] Document upstream device.
* [ ] Document downstream device.
* [ ] Review configuration references.

## WAN

* [ ] ISP information documented.
* [ ] Gateway verified.
* [ ] DNS verified.
* [ ] Default route verified.
* [ ] NAT verified.
* [ ] Firewall policy verified.
* [ ] SD-WAN requirement evaluated.
* [ ] Failover tested.

## LAN

* [ ] VLAN plan documented.
* [ ] DHCP scope documented.
* [ ] DHCP reservations documented.
* [ ] DNS architecture documented.
* [ ] Inter-VLAN policies documented.
* [ ] Segmentation verified.
* [ ] Device detection evaluated.

---

# 58. 🧪 NSE 4 Interface Checklist

For NSE 4 revision:

* [ ] Physical interface
* [ ] Interface role
* [ ] IP addressing
* [ ] Administrative access
* [ ] VLAN
* [ ] Aggregate interface
* [ ] Redundant interface
* [ ] Software switch
* [ ] Zone
* [ ] DHCP
* [ ] DNS
* [ ] NAT
* [ ] Transparent Mode
* [ ] Virtual Wire Pair
* [ ] Basic proxy concepts

---

# 59. 🧠 NSE 7 Engineering Checklist

For NSE 7, go beyond syntax.

* [ ] Understand why an interface type is selected.
* [ ] Understand the forwarding path.
* [ ] Understand Layer 2 behavior.
* [ ] Understand Layer 3 behavior.
* [ ] Understand policy dependencies.
* [ ] Understand routing dependencies.
* [ ] Understand redundancy.
* [ ] Understand failure domains.
* [ ] Understand security boundaries.
* [ ] Understand troubleshooting methodology.
* [ ] Understand interaction with adjacent infrastructure.

---

# 60. ⚡ One-Minute Revision

```text
NEW FORTIGATE
     │
     ▼
HARDWARE
     │
     ▼
FORTIOS
     │
     ▼
LICENSES
     │
     ▼
HOSTNAME
     │
     ▼
TIMEZONE + NTP
     │
     ▼
INTERFACE DESIGN
     │
 ┌───┼─────────────────────────┐
 ▼   ▼      ▼      ▼      ▼    ▼
PHY VLAN  LACP  REDUNDANT ZONE VWP
     │
     ▼
ROUTING
     │
     ▼
DHCP / DNS
     │
     ▼
FIREWALL POLICY
     │
     ▼
SECURITY PROFILES
     │
     ▼
LOGGING
     │
     ▼
MONITORING
     │
     ▼
VALIDATION
```

---

# 61. 🔥 Golden Rules

> ### Rule 1 — Interface ≠ Security
>
> Creating an interface does not automatically authorize traffic.

> ### Rule 2 — Route ≠ Permission
>
> A valid route does not replace a firewall policy.

> ### Rule 3 — VLAN ID ≠ Internal Bridge
>
> The same VLAN ID on different FortiGate interfaces does not automatically connect them at Layer 2.

> ### Rule 4 — Aggregate ≠ Redundant
>
> Aggregate uses multiple links; redundant interfaces focus on failover.

> ### Rule 5 — LACP ≠ One Giant TCP Pipe
>
> Traffic distribution depends on hashing and flow characteristics.

> ### Rule 6 — Zone ≠ Security Policy
>
> A zone groups interfaces; the policy still determines access.

> ### Rule 7 — NTP = Security Visibility
>
> Bad time synchronization can destroy reliable log correlation.

> ### Rule 8 — References Matter
>
> Before changing an object, understand what depends on it.

> ### Rule 9 — Design Before Configuration
>
> Define the traffic path before selecting the interface type.

> ### Rule 10 — Troubleshoot Layer by Layer
>
> ```text
> Physical
>   ↓
> L2
>   ↓
> VLAN
>   ↓
> Interface
>   ↓
> Route
>   ↓
> Policy
>   ↓
> NAT
>   ↓
> Security Inspection
>   ↓
> Logging
> ```

---

# 62. 🏁 Final Engineering Mental Model

A FortiGate interface is **not simply an IP address**.

It is part of a larger forwarding and security architecture:

```text
                     INTERFACE
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
         L2              L3          Logical
          │              │              │
       VLAN/STP        Route           Zone
       MAC/ARP         NAT             VWP
          │            SD-WAN          LACP
          │              │             Switch
          └──────────────┼──────────────┘
                         ▼
                  FIREWALL POLICY
                         │
                         ▼
                 SECURITY INSPECTION
                         │
                         ▼
                      LOGGING
                         │
                         ▼
                  ANALYTICS / SIEM
```

### Final Checklist

* [ ] Hardware validated.
* [ ] FortiOS validated.
* [ ] Firmware path validated.
* [ ] Licensing validated.
* [ ] IP plan completed.
* [ ] VLAN plan completed.
* [ ] Routing plan completed.
* [ ] Interface architecture selected.
* [ ] LACP/redundancy requirements validated.
* [ ] Security zones defined.
* [ ] DHCP/DNS designed.
* [ ] Proxy/captive portal requirements evaluated.
* [ ] Identity architecture evaluated.
* [ ] Endpoint architecture evaluated.
* [ ] Firewall policies designed.
* [ ] Logging designed.
* [ ] Monitoring designed.
* [ ] Failure scenarios tested.
* [ ] Rollback procedure documented.

> **SheynShield Engineering Principle**
>
> **Don't configure interfaces from the GUI outward. Design the traffic flow from the architecture inward.**

---

# 🔖 Keywords

`FortiGate Interface Configuration`
`FortiOS Interface Configuration`
`FortiGate LACP Configuration`
`FortiGate 802.3ad`
`FortiGate Aggregate Interface`
`FortiGate Redundant Interface`
`FortiGate Virtual Wire Pair`
`FortiGate VWP`
`FortiGate VLAN Configuration`
`FortiGate VLAN Interface`
`FortiGate Inter VLAN Routing`
`FortiGate Zone Configuration`
`FortiGate Software Switch`
`FortiGate Hardware Switch`
`FortiGate Internal Switch Mode`
`FortiGate Transparent Mode`
`FortiGate DHCP Server`
`FortiGate DHCP Reservation`
`FortiGate DNS Forwarder`
`FortiGate Explicit Proxy`
`FortiGate PAC File`
`FortiGate Captive Portal`
`FortiGate FSSO`
`FortiGate FortiClient Integration`
`FortiGate Device Detection`
`FortiGate Endpoint Control`
`FortiGate NSE4`
`FortiGate NSE7`
`FortiGate Troubleshooting`
`Fortinet Network Design`
`Fortinet Security Architecture`
`FortiGate Deployment Checklist`
`FortiGate Pre Deployment Checklist`
`FortiGate Interface Design Checklist`

---

# 🔗 SheynShield Resources

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

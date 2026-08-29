# 🔐 FortiGate IPsec Aggregate & Redundancy Checklist

> **SheynShield | FortiGate IPsec Engineering Checklist**
> Practical checklist for **IPsec Aggregate, Round-Robin, Redundant IPsec, IPsec Monitoring, OSPF, HA, ESP Sequence Synchronization, and Dual-ISP VPN designs**.

---

## 📌 What This Checklist Covers

* [ ] IPsec Aggregate
* [ ] IPsec Round-Robin
* [ ] L3 / L4 IPsec load balancing
* [ ] Weighted Round-Robin
* [ ] Redundant IPsec
* [ ] IPsec Monitor
* [ ] Dual-ISP VPN redundancy
* [ ] OSPF over IPsec
* [ ] OSPF path preference
* [ ] FortiGate HA + IPsec
* [ ] ESP sequence synchronization
* [ ] IPsec over SD-WAN
* [ ] Dynamic IPsec interfaces
* [ ] Add Route
* [ ] Device Creation
* [ ] IPsec troubleshooting
* [ ] FortiGate verification commands

---

# 1. 🧠 IPsec Design Selection Checklist

Before configuring the VPN, identify the actual requirement.

### Load Balancing

* [ ] Multiple IPsec tunnels must actively carry traffic
* [ ] Traffic distribution is required
* [ ] Multiple ISP links are available
* [ ] A single logical VPN interface is preferred
* [ ] IPsec Aggregate is selected

### Primary / Backup

* [ ] Only one VPN path should normally be active
* [ ] Second VPN should become active after failure
* [ ] Active/standby behavior is required
* [ ] IPsec Monitor is appropriate
* [ ] Redundant IPsec design is considered

### Dynamic Routing

* [ ] OSPF is required
* [ ] BGP is required
* [ ] Dynamic route convergence is required
* [ ] IPsec interface design supports the routing protocol

### HA

* [ ] FortiGate HA is deployed
* [ ] IPsec sessions must survive or recover correctly after failover
* [ ] ESP sequence synchronization is evaluated
* [ ] Session synchronization requirements are evaluated

---

# 2. 🌐 Dual-ISP Underlay Checklist

## ISP-1

* [ ] ISP-1 interface configured
* [ ] ISP-1 gateway reachable
* [ ] Default route configured
* [ ] Internet reachability verified
* [ ] IPsec peer reachable

Example:

```text
0.0.0.0/0 → ISP-1
Distance = 10
```

## ISP-2

* [ ] ISP-2 interface configured
* [ ] ISP-2 gateway reachable
* [ ] Default route configured
* [ ] Internet reachability verified
* [ ] IPsec peer reachable

Example:

```text
0.0.0.0/0 → ISP-2
Distance = 10 or 11
```

### Aggregation Design

If both underlay paths are expected to participate in an aggregate:

* [ ] Verify default-route selection
* [ ] Verify route distances
* [ ] Verify ECMP/route-selection behavior where applicable
* [ ] Confirm both IPsec peers can establish
* [ ] Do not blindly equalize route distances without validating the resulting routing behavior

---

# 3. 🔐 IPsec Member Tunnel Checklist

For every IPsec aggregate member:

* [ ] Unique tunnel name
* [ ] Correct outgoing interface
* [ ] Correct remote gateway
* [ ] Correct IKE version
* [ ] Correct authentication
* [ ] Correct PSK
* [ ] Correct encryption algorithm
* [ ] Correct authentication/hash algorithm
* [ ] Correct DH group
* [ ] Correct Phase 2 selectors
* [ ] PFS matches peer
* [ ] Auto-negotiate configured as required
* [ ] DPD configured appropriately
* [ ] Peer identity validated
* [ ] NAT-T requirements checked

Example topology:

```text
                 FortiGate
                    |
          +---------+---------+
          |                   |
       ISP-1                ISP-2
          |                   |
       IPsec-1             IPsec-2
          |                   |
          +---------+---------+
                    |
             IPsec Aggregate
```

> ⚠️ **Security note:** Legacy examples may use algorithms such as DES/MD5. For production deployments, use cryptographic algorithms and IKE settings appropriate for the FortiOS version and current security requirements.

---

# 4. 🔗 IPsec Aggregate Checklist

Create the aggregate only after individual IPsec members are correctly established.

* [ ] IPsec member 1 created
* [ ] IPsec member 2 created
* [ ] Both members use compatible VPN parameters
* [ ] Both members can establish independently
* [ ] Aggregate interface created
* [ ] Correct members added
* [ ] Aggregate status verified
* [ ] Aggregate interface addressing configured
* [ ] Firewall policies reference the aggregate where appropriate
* [ ] Routing references the aggregate where appropriate

Example:

```text
IPsec-1 ──┐
          ├──► IPsec Aggregate
IPsec-2 ──┘
```

---

# 5. ⚖️ IPsec Aggregate Algorithm Checklist

Choose the algorithm according to the traffic-distribution requirement.

## Round-Robin

* [ ] Round-robin requirement confirmed
* [ ] Aggregate members have compatible characteristics
* [ ] Underlay routing verified
* [ ] Member health verified

Example:

```bash
config system ipsec-aggregate
    edit "aggregate-links-12"
        set algorithm round-robin
    next
end
```

---

## L3 Load Balancing

Use when Layer-3 traffic distribution is desired.

* [ ] Source IP considered
* [ ] Destination IP considered
* [ ] Flow distribution tested
* [ ] Multiple flows tested

Concept:

```text
Client-1 ──► IPsec-1
Client-2 ──► IPsec-2
Client-3 ──► IPsec-1
Client-4 ──► IPsec-2
```

---

## L4 Load Balancing

* [ ] Source IP considered
* [ ] Destination IP considered
* [ ] Source port considered
* [ ] Destination port considered
* [ ] Protocol considered
* [ ] Multiple application flows tested

Example:

```text
TCP/443 Flow-1 ──► IPsec-1
TCP/443 Flow-2 ──► IPsec-2
UDP Flow-1 ──────► IPsec-1
```

---

## Weighted Round-Robin

Useful when aggregate members have different capacities.

* [ ] Link bandwidth documented
* [ ] Weight assigned according to capacity
* [ ] Traffic distribution tested
* [ ] Congestion monitored

Example:

```text
ISP-1 = Weight 3
ISP-2 = Weight 1
```

---

# 6. 🚨 IPsec Aggregate Route Checklist

When aggregate members unexpectedly go down:

* [ ] Check ISP default routes
* [ ] Check administrative distance
* [ ] Check route selection
* [ ] Check recursive reachability
* [ ] Check remote gateway reachability
* [ ] Check IKE status
* [ ] Check IPsec SA
* [ ] Check aggregate member state

Example:

```text
ISP-1
AD 10

ISP-2
AD 11
```

Do not automatically assume:

```text
AD 10 + AD 11
```

is appropriate for every aggregate design.

Validate the resulting routing table first.

---

# 7. 🧹 IPsec Monitor vs Aggregate Checklist

These are **different redundancy models**.

## IPsec Monitor

```text
Primary VPN
     |
     | monitor
     v
Backup VPN
```

Use when:

* [ ] Primary/backup behavior is required
* [ ] Only one path should normally forward traffic
* [ ] Failure detection should trigger backup activation
* [ ] Active/standby architecture is desired

---

## IPsec Aggregate

```text
IPsec-1 ──┐
          ├──► Aggregate
IPsec-2 ──┘
```

Use when:

* [ ] Multiple members should participate
* [ ] Traffic distribution is required
* [ ] One logical overlay is desired
* [ ] Aggregate algorithms are appropriate

### Migration Checklist

When moving from monitor-based redundancy to aggregation:

* [ ] Review `set monitor`
* [ ] Remove obsolete monitoring relationships
* [ ] Verify aggregate member configuration
* [ ] Verify routing
* [ ] Verify failure behavior

Example:

```bash
config vpn ipsec phase1-interface
    edit "link-2"
        unset monitor
    next
end
```

---

# 8. 🖥️ Aggregate Interface Checklist

Configure the logical aggregate interface.

### FGT-1

```text
Interface:
    aggregate-links-12

IP:
    12.12.12.1/24
```

### FGT-2

```text
Interface:
    aggregate-links-12

IP:
    12.12.12.2/24
```

Validation:

* [ ] IP address assigned correctly
* [ ] Remote IP reachable
* [ ] Ping enabled where required
* [ ] ARP/neighbor behavior verified where applicable
* [ ] Routing table points toward the correct logical interface

---

# 9. 🔥 Firewall Policy Checklist

For routed IPsec traffic:

### LAN → IPsec

* [ ] Source interface = LAN
* [ ] Destination interface = IPsec/aggregate
* [ ] Correct source objects
* [ ] Correct destination objects
* [ ] Required services allowed
* [ ] NAT disabled unless intentionally required
* [ ] Logging enabled

### IPsec → LAN

* [ ] Source interface = IPsec/aggregate
* [ ] Destination interface = LAN
* [ ] Correct source objects
* [ ] Correct destination objects
* [ ] Required services allowed
* [ ] NAT disabled unless intentionally required
* [ ] Logging enabled

### Return Path

* [ ] Remote network has return route
* [ ] Local network has return route
* [ ] Asymmetric routing checked
* [ ] Policy order verified

---

# 10. 🛰️ OSPF over IPsec Aggregate Checklist

## FortiGate 1

* [ ] Router ID configured
* [ ] Area configured
* [ ] IPsec aggregate included in OSPF
* [ ] LAN included in OSPF
* [ ] OSPF neighbor established
* [ ] Routes exchanged
* [ ] Cost validated

Example:

```text
Router-ID:
    1.1.1.1

Area:
    0.0.0.0

Networks:
    12.12.12.0/24
    192.168.101.0/24
```

## FortiGate 2

* [ ] Router ID configured
* [ ] Area configured
* [ ] IPsec aggregate included in OSPF
* [ ] LAN included in OSPF
* [ ] OSPF neighbor established
* [ ] Routes exchanged

Example:

```text
Router-ID:
    2.2.2.2

Area:
    0.0.0.0

Networks:
    12.12.12.0/24
    192.168.102.0/24
```

---

# 11. 🎯 OSPF Path Preference Checklist

If one ISP should be preferred:

* [ ] OSPF cost configured
* [ ] Primary path has lower cost
* [ ] Backup path has higher cost
* [ ] Both paths remain operational if required
* [ ] OSPF database verified
* [ ] Routing table verified

Example:

```text
ISP-1
OSPF Cost = 10
        ↓
     Preferred

ISP-2
OSPF Cost = 11
        ↓
      Backup
```

> OSPF cost influences **routing-path selection**; it is not the same mechanism as IPsec aggregate traffic distribution.

---

# 12. 🔄 Redundant IPsec Checklist

Choose redundant IPsec when the requirement is primarily **active/standby VPN failover**.

### Hub

* [ ] Dialup IPsec configured
* [ ] Device Creation evaluated
* [ ] Add Route evaluated
* [ ] Phase 2 selectors configured
* [ ] No unnecessary interface IP assignment
* [ ] VPN policy configured

### Spoke

* [ ] IPsec-1 created
* [ ] IPsec-2 created
* [ ] Both tunnels can establish
* [ ] Primary tunnel identified
* [ ] Backup tunnel identified
* [ ] Monitoring relationship configured if required
* [ ] Default routes configured
* [ ] Administrative distances validated

Example:

```text
0.0.0.0/0
   |
   +── IPsec-1
   |      AD 10
   |
   +── IPsec-2
          AD 11
```

---

# 13. 📡 IPsec Monitor Checklist

For primary/backup VPN:

```bash
config vpn ipsec phase1-interface
    edit "ipsec-2"
        set monitor "ipsec-1"
    next
end
```

Validate:

* [ ] Primary tunnel UP
* [ ] Backup tunnel available
* [ ] Monitor relationship correct
* [ ] Failure detection tested
* [ ] Backup activation tested
* [ ] Route changes verified
* [ ] Recovery behavior tested

### Failure Test

```text
Primary UP
    ↓
Primary forwards traffic
    ↓
Primary failure
    ↓
Monitor detects failure
    ↓
Backup becomes usable
    ↓
Routing converges
    ↓
Traffic resumes
```

---

# 14. 🧩 Add Route Checklist

Evaluate:

```text
Add Route
```

before enabling it.

* [ ] Dynamic route installation is required
* [ ] Static routing behavior understood
* [ ] Duplicate routes checked
* [ ] Administrative distance checked
* [ ] Routing table verified
* [ ] Failover behavior tested

> `Add Route` changes how VPN-related routes are installed. Do not enable it simply because the tunnel is not forwarding traffic; first determine whether the actual problem is routing, interface creation, ADVPN behavior, policy, or tunnel state.

---

# 15. 🖥️ Device Creation Checklist

Evaluate:

```text
Device Creation
```

when dynamic IPsec tunnels need interface representation.

* [ ] Dynamic tunnel interface required
* [ ] Routing protocol requires interface representation
* [ ] Dynamic tunnel behavior understood
* [ ] Number of generated interfaces considered
* [ ] Routing table impact reviewed

Concept:

```text
IPsec
  |
  +── Device Creation
  |       ↓
  |   Tunnel Interface
  |
  +── Add Route
          ↓
     Routing Table
```

---

# 16. 🏢 HA + IPsec Checklist

For FortiGate HA deployments:

* [ ] HA cluster healthy
* [ ] HA members have compatible configuration
* [ ] IPsec configuration synchronized
* [ ] VPN interfaces correctly synchronized
* [ ] Firewall policies synchronized
* [ ] Routing configuration synchronized
* [ ] Session pickup requirements evaluated
* [ ] IPsec failover tested

Example:

```text
FGT-1
Priority = 129
        ↓
Primary

FGT-2
Priority = 128
        ↓
Secondary
```

---

# 17. 🔐 ESP Sequence Synchronization Checklist

For HA IPsec designs where ESP sequence synchronization is required:

```bash
config vpn ipsec phase1-interface
    edit "ipsec-tun-1"
        set ha-sync-esp-seqno enable
    next
end
```

Validate:

* [ ] Requirement identified
* [ ] Configuration applied consistently
* [ ] HA synchronization verified
* [ ] IPsec traffic generated
* [ ] HA failover performed
* [ ] Existing flows tested
* [ ] New flows tested
* [ ] IPsec SAs verified after failover

Concept:

```text
Primary FortiGate
       |
       | ESP State
       | Sync
       v
Secondary FortiGate
       |
     Failover
       |
       v
New Primary
       |
       v
IPsec forwarding
```

---

# 18. 🆔 IPsec Location ID Checklist

When multiple IPsec VPN locations must be distinguished:

```bash
config system settings
    set location-id 1.1.1.1
end
```

Example:

```text
HUB
location-id = 1.1.1.1

SPOKE-1
location-id = 2.2.2.2

SPOKE-2
location-id = 3.3.3.3
```

* [ ] Unique location ID
* [ ] Location IDs documented
* [ ] Duplicate IDs avoided
* [ ] Multi-VPN design reviewed

---

# 19. ☁️ IPsec over SD-WAN Checklist

Separate the **underlay** from the **overlay**.

```text
             SD-WAN
                |
        +-------+-------+
        |               |
      ISP-1           ISP-2
        |               |
        +-------+-------+
                |
              IPsec
                |
             Overlay
```

### Underlay

* [ ] SD-WAN members healthy
* [ ] ISP health checks configured
* [ ] SLA requirements validated
* [ ] Internet reachability verified

### Overlay

* [ ] IPsec tunnels established
* [ ] Phase 1 verified
* [ ] Phase 2 verified
* [ ] Routing verified
* [ ] Policies verified

### Architecture

* [ ] Underlay and overlay roles clearly documented
* [ ] IPsec interface added to SD-WAN only when the design requires it
* [ ] Routing interaction reviewed

---

# 20. 🧪 IPsec Verification Checklist

## Aggregate

```bash
diagnose sys ipsec-aggregate list
```

Check:

* [ ] Aggregate exists
* [ ] Correct members
* [ ] Member state
* [ ] Aggregate state
* [ ] VRF information

---

## IKE

```bash
diagnose vpn ike gateway list
```

Check:

* [ ] IKE SA established
* [ ] Correct peer
* [ ] Correct interface
* [ ] Correct IKE version
* [ ] No negotiation errors

---

## IPsec

```bash
diagnose vpn tunnel list
```

Check:

* [ ] IPsec SA established
* [ ] Encapsulation counters increase
* [ ] Decapsulation counters increase
* [ ] Correct selectors
* [ ] Correct tunnel state

---

# 21. 🐛 IKE Debug Checklist

Use debugging only when required.

```bash
diagnose debug enable
diagnose debug application ike -1
```

After troubleshooting:

```bash
diagnose debug disable
```

Checklist:

* [ ] Debug enabled intentionally
* [ ] Test traffic generated
* [ ] Relevant negotiation captured
* [ ] Error identified
* [ ] Debug disabled

> ⚠️ Avoid leaving verbose debugging enabled on a production firewall longer than necessary.

---

# 22. 🔍 End-to-End Troubleshooting Checklist

When traffic does not pass:

```text
Underlay
   ↓
IKE
   ↓
IPsec SA
   ↓
Aggregate / Redundant Member
   ↓
Routing
   ↓
OSPF / BGP
   ↓
Firewall Policy
   ↓
NAT
   ↓
Return Path
   ↓
Application
```

Check in this order:

### Underlay

* [ ] ISP interface UP
* [ ] Gateway reachable
* [ ] Default route correct
* [ ] Remote peer reachable

### IKE

* [ ] Phase 1 UP
* [ ] IKE version matches
* [ ] Authentication matches
* [ ] PSK matches
* [ ] Proposal matches
* [ ] DH group matches

### IPsec

* [ ] Phase 2 UP
* [ ] Selectors match
* [ ] PFS matches
* [ ] Auto-negotiate behavior validated
* [ ] SA counters increase

### Aggregate

* [ ] Aggregate exists
* [ ] Members are UP
* [ ] Correct algorithm
* [ ] No unexpected monitor configuration
* [ ] Route selection correct

### Routing

* [ ] Local route exists
* [ ] Remote route exists
* [ ] OSPF/BGP neighbor UP
* [ ] Correct next-hop
* [ ] Correct administrative distance
* [ ] No unexpected static route
* [ ] No routing loop
* [ ] Return route exists

### Firewall

* [ ] LAN → VPN policy exists
* [ ] VPN → LAN policy exists
* [ ] Correct interfaces
* [ ] Correct source
* [ ] Correct destination
* [ ] Correct service
* [ ] NAT behavior correct
* [ ] Policy hit counter increases

---

# 23. 🚨 Common IPsec Aggregate Mistakes

## ❌ Mixing Monitor and Aggregate Designs

```text
Aggregate
   +
Primary/Backup Monitor
```

* [ ] Verify whether both mechanisms are actually required
* [ ] Remove obsolete monitoring configuration

---

## ❌ Incorrect Underlay Routing

```text
ISP-1 AD 10
ISP-2 AD 11
```

while expecting equal participation.

* [ ] Check routing table
* [ ] Check recursive reachability
* [ ] Check aggregate behavior

---

## ❌ Assuming Tunnel UP = Traffic UP

```text
IPsec = UP
```

does not prove:

```text
Routing = Correct
Policy = Correct
NAT = Correct
Return Path = Correct
Application = Working
```

* [ ] Validate the entire forwarding path

---

## ❌ NAT Enabled Accidentally

For normal routed site-to-site VPN traffic:

* [ ] NAT disabled unless intentionally required

---

## ❌ Missing Return Route

Validate both directions:

```text
LAN-A
  ↓
IPsec
  ↓
LAN-B
```

and:

```text
LAN-B
  ↓
IPsec
  ↓
LAN-A
```

---

## ❌ Using Weak Legacy Cryptography in Production

Avoid copying lab values such as:

```text
DES
MD5
```

into production without a security review.

* [ ] Follow the cryptographic recommendations for the deployed FortiOS version
* [ ] Prefer modern encryption/integrity algorithms
* [ ] Verify peer compatibility

---

# 24. 📊 Aggregate vs Redundant IPsec Decision Matrix

| Requirement                     |               IPsec Aggregate |             Redundant IPsec |
| ------------------------------- | ----------------------------: | --------------------------: |
| Multiple IPsec tunnels          |                             ✅ |                           ✅ |
| One logical aggregate interface |                             ✅ |                     Depends |
| Load balancing                  |                             ✅ |                           ❌ |
| Round-Robin                     |                             ✅ |                           ❌ |
| L3 distribution                 |                             ✅ |                           ❌ |
| L4 distribution                 |                             ✅ |                           ❌ |
| Weighted distribution           |                             ✅ |                           ❌ |
| Active/standby                  |                      Possible |                           ✅ |
| IPsec Monitor                   | Usually not primary mechanism |                           ✅ |
| Multiple ISP                    |                             ✅ |                           ✅ |
| OSPF/BGP                        |                             ✅ | Depends on interface design |
| Simple failover                 |                            ⚠️ |                           ✅ |
| Traffic distribution            |                             ✅ |                           ❌ |

---

# 25. 🧭 IPsec Architecture Decision Tree

```text
                    VPN Requirement
                          |
             +------------+------------+
             |                         |
        Load Balance?              Failover?
             |                         |
            YES                       YES
             |                         |
             ▼                         ▼
      IPsec Aggregate          Redundant IPsec
             |                         |
      +------+------+                  |
      |      |      |                  |
    L3     L4    Weighted              |
      |      |      |                  |
      +------+------+                  |
             |                         |
         Round-Robin             IPsec Monitor
```

For dynamic routing:

```text
IPsec
  |
  +── OSPF
  |
  +── BGP
  |
  └── Static
```

For HA:

```text
HA
 |
 +── IPsec
 |
 └── ESP Sequence Sync
```

---

# 26. ⚡ Fast Operational Checklist

Use this during a production change window.

### Before Change

* [ ] Backup FortiGate configuration
* [ ] Document current routes
* [ ] Document current VPN tunnels
* [ ] Document current monitor relationships
* [ ] Document firewall policies
* [ ] Document OSPF/BGP state
* [ ] Define rollback plan

### During Change

* [ ] Establish IPsec members
* [ ] Verify IKE
* [ ] Verify Phase 2
* [ ] Create aggregate/redundant design
* [ ] Configure routing
* [ ] Configure firewall policies
* [ ] Verify route table
* [ ] Test traffic

### After Change

* [ ] Test primary path
* [ ] Test secondary path
* [ ] Test load distribution where applicable
* [ ] Test failure
* [ ] Test recovery
* [ ] Verify OSPF/BGP convergence
* [ ] Check logs
* [ ] Check tunnel counters
* [ ] Save configuration

---

# 27. 🧠 Engineer Mental Model

```text
                 UNDERLAY
        ┌──────────┴──────────┐
        │                     │
      ISP-1                 ISP-2
        │                     │
        ▼                     ▼
     IPsec-1               IPsec-2
        │                     │
        └──────────┬──────────┘
                   │
             VPN DESIGN
                   │
          ┌────────┴────────┐
          │                 │
      Aggregate          Redundant
          │                 │
     Load Balance       Active/Backup
          │                 │
          └────────┬────────┘
                   │
                Routing
                   │
             OSPF / BGP
                   │
                   ▼
               Forwarding
                   │
                   ▼
               Firewall
                   │
                   ▼
                Traffic
```

---

# 28. ⭐ Core Rules

```text
IPsec Aggregate
    ↓
Multiple active VPN members
    ↓
Traffic distribution / redundancy


IPsec Monitor
    ↓
Primary / Backup
    ↓
Failure detection


OSPF
    ↓
Dynamic route exchange
    ↓
Path recalculation


HA
    ↓
Cluster redundancy
    ↓
ESP sequence synchronization when required


SD-WAN
    ↓
Underlay path selection
    ↓
IPsec overlay
```

---

# 29. 📋 Final Production Readiness Checklist

## Architecture

* [ ] Requirement clearly defined
* [ ] Aggregate vs redundant design selected correctly
* [ ] Underlay/overlay separation documented
* [ ] Failure domains documented
* [ ] Capacity requirements documented

## IPsec

* [ ] Phase 1 validated
* [ ] Phase 2 validated
* [ ] Modern cryptography selected
* [ ] DPD configured
* [ ] NAT-T evaluated
* [ ] Tunnel health verified

## Aggregate

* [ ] Correct members
* [ ] Correct algorithm
* [ ] Routing behavior validated
* [ ] Member failure tested
* [ ] Recovery tested

## Redundancy

* [ ] Primary path identified
* [ ] Backup path identified
* [ ] Monitor configured where required
* [ ] Route failover verified
* [ ] Recovery behavior verified

## Routing

* [ ] OSPF/BGP/static routing validated
* [ ] Next-hop verified
* [ ] Administrative distance verified
* [ ] OSPF cost verified where applicable
* [ ] Return path verified
* [ ] No routing loop

## Security

* [ ] Firewall policies reviewed
* [ ] NAT reviewed
* [ ] Least-privilege policy applied
* [ ] Logging enabled
* [ ] Legacy cryptography removed from production

## HA

* [ ] HA cluster healthy
* [ ] Configuration synchronized
* [ ] Session behavior validated
* [ ] ESP sequence synchronization evaluated
* [ ] Failover tested

## Monitoring

* [ ] VPN status monitored
* [ ] ISP health monitored
* [ ] Routing convergence monitored
* [ ] Logs reviewed
* [ ] Alerting configured

---

# 🚀 Quick Reference

| Requirement                                     | FortiGate Feature                   |
| ----------------------------------------------- | ----------------------------------- |
| Multiple IPsec tunnels as one logical interface | IPsec Aggregate                     |
| Load balancing                                  | IPsec Aggregate                     |
| Round-robin distribution                        | `set algorithm round-robin`         |
| Layer-3 distribution                            | L3                                  |
| Layer-4 distribution                            | L4                                  |
| Unequal link capacity                           | Weighted Round-Robin                |
| Primary / Backup VPN                            | IPsec Monitor                       |
| Redundant dialup VPN                            | Redundant IPsec                     |
| Dynamic routing                                 | OSPF / BGP                          |
| OSPF path preference                            | Interface Cost                      |
| HA VPN continuity                               | HA + IPsec                          |
| ESP state synchronization                       | `ha-sync-esp-seqno`                 |
| Aggregate diagnostics                           | `diagnose sys ipsec-aggregate list` |
| IKE diagnostics                                 | `diagnose vpn ike gateway list`     |
| IPsec diagnostics                               | `diagnose vpn tunnel list`          |
| IKE debugging                                   | `diagnose debug application ike -1` |
| Underlay path control                           | SD-WAN                              |
| VPN overlay                                     | IPsec                               |

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

## 🔎 Keywords

`FortiGate IPsec Aggregate` · `FortiGate IPsec Redundancy` · `FortiGate IPsec Failover` · `FortiGate IPsec Monitor` · `FortiGate IPsec Load Balancing` · `FortiGate Round Robin IPsec` · `FortiGate OSPF IPsec` · `FortiGate HA IPsec` · `FortiGate ESP Sequence Synchronization` · `FortiGate Dual ISP IPsec` · `FortiOS IPsec Aggregate` · `FortiGate VPN Troubleshooting`

---

> **SheynShield — Engineering Secure Networks**
>
> **IPsec Aggregate = active traffic distribution**
> **Redundant IPsec = active/standby failover**
> **OSPF/BGP = dynamic routing**
> **HA = device-level redundancy**

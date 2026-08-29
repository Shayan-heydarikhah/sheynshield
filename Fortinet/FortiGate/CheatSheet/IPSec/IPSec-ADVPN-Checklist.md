# FortiGate ADVPN Checklist — Auto Discovery VPN

> **SheynShield | Engineering Secure Networks**
> **Topic:** FortiGate ADVPN (Auto Discovery VPN)
> **Focus:** IPsec, Hub-and-Spoke, Dynamic Routing, Spoke-to-Spoke Shortcuts, BGP, OSPF, RIP, Dual ISP & Troubleshooting

---

## 📌 What This Checklist Covers

* [ ] ADVPN architecture
* [ ] Hub configuration
* [ ] Spoke configuration
* [ ] IPsec Phase 1 / Phase 2
* [ ] Auto-Discovery Sender / Receiver
* [ ] Dynamic tunnel behavior
* [ ] `net-device`
* [ ] `add-route`
* [ ] `device-creation`
* [ ] BGP
* [ ] OSPF
* [ ] RIP
* [ ] RIP split horizon
* [ ] Spoke-to-spoke shortcuts
* [ ] Dual ISP failover
* [ ] Routing verification
* [ ] ADVPN troubleshooting
* [ ] Production validation

---

# 1. 🧠 ADVPN Architecture Checklist

### Core Architecture

* [ ] Hub-and-Spoke topology identified
* [ ] Hub identified as ADVPN discovery point
* [ ] Spokes identified
* [ ] Hub configured as **Auto-Discovery Sender**
* [ ] Spokes configured as **Auto-Discovery Receivers**
* [ ] IPsec used as the transport
* [ ] Dynamic routing protocol selected
* [ ] Spoke-to-spoke shortcut requirement identified
* [ ] Number of spokes estimated
* [ ] Scalability requirements documented

### Mental Model

```text
                  +----------------+
                  |      HUB       |
                  |                |
                  | ADVPN SENDER   |
                  | BGP / OSPF/RIP |
                  +-------+--------+
                          |
              +-----------+-----------+
              |                       |
        +-----+-----+           +-----+-----+
        |  SPOKE-1  |           |  SPOKE-2  |
        | ADVPN RX  |           | ADVPN RX  |
        +-----+-----+           +-----+-----+
              \                       /
               \                     /
                \==== SHORTCUT =====/
```

### Expected Traffic Evolution

```text
Initial:

Spoke-1 ───────> Hub <─────── Spoke-2


After ADVPN discovery:

Spoke-1 ═════════════════════ Spoke-2
              Shortcut
```

* [ ] Initial spoke-to-hub connectivity verified
* [ ] ADVPN discovery verified
* [ ] Dynamic shortcut establishment verified
* [ ] Traffic confirmed to use shortcut when expected

---

# 2. 🏗️ Hub Configuration Checklist

## IPsec Phase 1

* [ ] IPsec interface configured
* [ ] Dialup/dynamic peer model selected
* [ ] Correct WAN interface selected
* [ ] Correct IKE version configured
* [ ] Correct peer type configured
* [ ] Authentication method configured
* [ ] PSK configured
* [ ] Encryption proposal configured
* [ ] Authentication proposal configured
* [ ] DPD configured
* [ ] Auto-negotiation requirement evaluated
* [ ] ADVPN Sender enabled

Example:

```bash
config vpn ipsec phase1-interface
    edit "advpn-hub"
        set type dynamic
        set interface "port9"
        set peertype any
        set auto-discovery-sender enable
        set dpd on-idle
        set psksecret "REDACTED"
        set dpd-retryinterval 5
    next
end
```

> 🔐 Never publish a real PSK in a public GitHub repository.

---

## IPsec Phase 2

* [ ] Phase 2 references correct Phase 1
* [ ] Encryption proposal matches spokes
* [ ] Authentication proposal matches spokes
* [ ] PFS requirement verified
* [ ] Selector design validated
* [ ] All-to-All selector considered where required

Example:

```bash
config vpn ipsec phase2-interface
    edit "advpn-hub"
        set phase1name "advpn-hub"
        set proposal aes128-sha256 aes256-sha256
    next
end
```

---

# 3. 🛰️ Spoke Configuration Checklist

## IPsec Phase 1

* [ ] Spoke uses correct hub public address
* [ ] Correct WAN interface selected
* [ ] Correct IKE version
* [ ] Correct PSK
* [ ] Compatible proposals
* [ ] DPD configured
* [ ] ADVPN Receiver enabled
* [ ] Device creation requirement evaluated
* [ ] Add-route requirement evaluated
* [ ] `net-device` behavior understood

Concept:

```text
HUB
 └── ADVPN Sender

SPOKE
 └── ADVPN Receiver
```

---

## Spoke Phase 2

* [ ] Correct Phase 1 reference
* [ ] Proposal compatible with Hub
* [ ] PFS compatible
* [ ] Selectors validated
* [ ] All-to-All requirement evaluated
* [ ] Auto-negotiation evaluated

---

# 4. 🔧 `net-device` Checklist

`net-device` changes how FortiGate represents dynamic IPsec tunnels.

### Validate

* [ ] Determine whether the design requires dynamic tunnel interfaces
* [ ] Check FortiOS behavior for the target version
* [ ] Check routing protocol requirements
* [ ] Check whether `device-creation` is required
* [ ] Check whether dynamically generated tunnel IDs are expected
* [ ] Validate routing behavior after changing the setting

### Decision

```text
Dynamic ADVPN
      |
      +-- Need conventional/dynamic interfaces?
      |          |
      |          +-- YES → evaluate net-device enable
      |
      +-- Minimal dynamic representation?
                 |
                 +-- evaluate net-device disable
```

> ⚠️ Do not copy a `net-device` value from another ADVPN design without understanding its routing consequences.

---

# 5. 🧩 Device Creation Checklist

* [ ] Determine whether dynamic tunnel interfaces must be created
* [ ] Verify routing protocol behavior
* [ ] Verify route installation behavior
* [ ] Verify number of dynamic interfaces
* [ ] Verify operational visibility
* [ ] Verify spoke-to-spoke behavior

### Typical Troubleshooting Adjustment

```text
Spoke-to-Spoke not working
          |
          v
Evaluate:
    Device Creation
          +
    Add Route
```

Possible troubleshooting configuration:

```text
Device Creation = Enable
Add Route        = Enable
```

> ⚠️ This is a troubleshooting/design decision, not a universal ADVPN default.

---

# 6. 🛣️ Add-Route Checklist

### Before enabling

* [ ] Understand where routes are expected to come from
* [ ] Identify dynamic routing protocol
* [ ] Check existing static routes
* [ ] Check route installation behavior
* [ ] Check routing-table preference
* [ ] Check for duplicate paths

### Verify

```bash
get router info routing-table all
```

Check:

* [ ] Hub routes
* [ ] Spoke routes
* [ ] Shortcut routes
* [ ] Next-hop information
* [ ] Administrative distance
* [ ] Route preference
* [ ] Unexpected static routes

---

# 7. 🔀 Dynamic Routing Selection

| Requirement              | Recommended Direction |
| ------------------------ | --------------------- |
| Large-scale ADVPN        | BGP                   |
| Strong policy control    | BGP                   |
| Fast convergence         | BGP / OSPF            |
| Traditional IGP          | OSPF                  |
| Simple legacy deployment | RIP                   |
| Large number of spokes   | BGP                   |
| Complex route policy     | BGP                   |

### Design Checklist

* [ ] Routing protocol selected
* [ ] AS/area/version documented
* [ ] Router IDs documented
* [ ] Advertised prefixes documented
* [ ] Next-hop behavior documented
* [ ] Failover behavior documented
* [ ] Route summarization considered
* [ ] Static routes minimized

---

# 8. 🚀 ADVPN + BGP Checklist

## Hub

* [ ] BGP enabled
* [ ] Local AS configured
* [ ] Router ID configured
* [ ] Neighbor group configured
* [ ] Neighbor range configured where appropriate
* [ ] Remote AS configured
* [ ] ADVPN tunnel interface selected
* [ ] Hub LAN advertised
* [ ] Route policy evaluated
* [ ] Next-hop behavior verified
* [ ] Link-down failover evaluated

Example:

```bash
config router bgp
    config neighbor-group
        edit "advpn"
            set link-down-failover enable
        next
    end
end
```

---

## BGP Neighbor Range

* [ ] Tunnel subnet documented
* [ ] Neighbor range covers intended spokes
* [ ] Remote AS correct
* [ ] Neighbor group correctly associated

Concept:

```text
12.23.34.0/24
       |
       +-- Spoke-1
       +-- Spoke-2
       +-- Spoke-3
       +-- Spoke-N
```

---

## BGP Spoke

* [ ] Local AS correct
* [ ] Router ID unique
* [ ] Hub neighbor configured
* [ ] Remote AS correct
* [ ] Local LAN advertised
* [ ] Link-down failover configured if required
* [ ] BGP session established
* [ ] Prefixes received
* [ ] Prefixes advertised

Verify:

```bash
get router info bgp summary
```

---

# 9. 🔄 BGP Next-Hop Checklist

For designs using different AS numbers:

```text
Hub      AS 65000
Spoke-1  AS 65001
Spoke-2  AS 65002
```

* [ ] AS design documented
* [ ] eBGP behavior understood
* [ ] Next-hop reachability verified
* [ ] `next-hop-self` evaluated
* [ ] ADVPN discovery verified separately
* [ ] Route recursion verified
* [ ] Static-route dependency minimized

### Mental Model

```text
Different AS
     |
     v
BGP
     |
     +-- Next-Hop Reachability
     |
     +-- ADVPN Discovery
     |
     +-- Dynamic Shortcut
```

---

# 10. 🌐 ADVPN + OSPF Checklist

## Hub

* [ ] OSPF enabled
* [ ] Router ID configured
* [ ] Correct area configured
* [ ] Tunnel network evaluated
* [ ] LAN networks advertised
* [ ] IPsec interface included where required
* [ ] OSPF neighbors verified
* [ ] Route database verified

Example:

```text
Router-ID:
    1.1.1.1

Area:
    0.0.0.0
```

---

## Spoke

* [ ] Unique router ID
* [ ] Correct area
* [ ] Local LAN advertised
* [ ] Tunnel network configured if required
* [ ] OSPF adjacency established
* [ ] Routes received
* [ ] Routes advertised
* [ ] No unexpected route preference

Verify:

```bash
get router info ospf neighbor
```

---

# 11. 🔁 ADVPN + RIP Checklist

* [ ] RIP version selected
* [ ] Networks configured
* [ ] Hub networks advertised
* [ ] Spoke LAN networks advertised
* [ ] Dynamic tunnel behavior understood
* [ ] Device Creation impact evaluated
* [ ] Split Horizon evaluated
* [ ] Routing loops checked

### RIP + Device Creation

```text
RIP
 |
 +-- Device Creation
        |
        +-- Dynamic interfaces
        |
        +-- Route advertisements
        |
        +-- Split Horizon consideration
```

---

# 12. 🛡️ RIP Split-Horizon Checklist

If dynamic interfaces/device creation are used:

* [ ] Split Horizon requirement evaluated
* [ ] Interface added to RIP correctly
* [ ] Route advertisements verified
* [ ] Hub does not incorrectly advertise routes back through the same path
* [ ] Routing loop tested

Example:

```bash
config router rip
    config interface
        edit "ipsec-dial-hs"
            set split-horizon-status enable
        next
    end
end
```

### Verification

```text
Spoke A
   |
   v
 Hub
   |
   v
Spoke B
```

* [ ] Spoke A routes do not create an unintended return advertisement through the same logical path

---

# 13. 🔐 IPsec Validation Checklist

## Phase 1

* [ ] IKE version matches
* [ ] Authentication matches
* [ ] PSK matches
* [ ] Encryption proposal matches
* [ ] Integrity/authentication proposal matches
* [ ] DH groups match
* [ ] Peer configuration correct
* [ ] DPD working
* [ ] NAT-T evaluated
* [ ] Phase 1 established

Verify:

```bash
diagnose vpn ike gateway list
```

---

## Phase 2

* [ ] Phase 2 established
* [ ] Proposal compatible
* [ ] PFS compatible
* [ ] Selectors correct
* [ ] Auto-negotiation behavior understood
* [ ] Rekey behavior validated

Verify:

```bash
diagnose vpn tunnel list
```

---

# 14. 🏠 Firewall Policy Checklist

## Hub

* [ ] LAN → IPsec policy exists
* [ ] IPsec → LAN policy exists
* [ ] Source addresses correct
* [ ] Destination addresses correct
* [ ] Service correct
* [ ] NAT disabled unless explicitly required
* [ ] Logging enabled during troubleshooting

## Spoke

* [ ] LAN → Hub policy
* [ ] Hub → LAN policy
* [ ] LAN → Dynamic Tunnel policy
* [ ] Dynamic Tunnel → LAN policy
* [ ] Shortcut traffic permitted
* [ ] NAT disabled for private-site traffic unless required

---

# 15. 🌍 Spoke-to-Spoke Shortcut Checklist

### Initial State

```text
Spoke-1
   |
   v
 Hub
   |
   v
Spoke-2
```

* [ ] Spoke-1 can reach Hub
* [ ] Spoke-2 can reach Hub
* [ ] Routing information exchanged
* [ ] ADVPN discovery information exchanged

### Shortcut State

```text
Spoke-1
   |
   |==================|
   |  Dynamic Tunnel  |
   |==================|
   |
Spoke-2
```

* [ ] Shortcut requested
* [ ] Shortcut negotiated
* [ ] Dynamic tunnel created
* [ ] Routing updated
* [ ] Traffic uses direct path
* [ ] Hub no longer carries expected data-plane traffic

---

# 16. 🚨 Spoke-to-Spoke Troubleshooting

If:

```text
Spoke-1 → Hub → Spoke-2
```

works but:

```text
Spoke-1 → Spoke-2
```

does not:

* [ ] Verify Phase 1
* [ ] Verify Phase 2
* [ ] Verify ADVPN Sender
* [ ] Verify ADVPN Receiver
* [ ] Verify tunnel IPs
* [ ] Verify routing
* [ ] Verify next-hop
* [ ] Verify dynamic tunnel creation
* [ ] Evaluate `device-creation`
* [ ] Evaluate `add-route`
* [ ] Check firewall policies
* [ ] Check NAT
* [ ] Check static routes
* [ ] Check route preference
* [ ] Check shortcut establishment

### Important Diagnostic Principle

```text
Hub connectivity works
        ≠
ADVPN shortcut works
```

Treat these as separate troubleshooting domains.

---

# 17. 🔎 Routing Verification Checklist

Run:

```bash
get router info routing-table all
```

Check:

* [ ] Local LAN route
* [ ] Hub LAN route
* [ ] Remote spoke LAN route
* [ ] Tunnel next-hop
* [ ] Dynamic route
* [ ] Static route
* [ ] Administrative distance
* [ ] Route preference
* [ ] Route recursion
* [ ] Unexpected default route

---

# 18. 🔍 BGP Verification Checklist

```bash
get router info bgp summary
```

* [ ] Neighbor established
* [ ] Correct remote AS
* [ ] Correct local AS
* [ ] Prefixes received
* [ ] Prefixes advertised
* [ ] Next-hop reachable
* [ ] No route flap
* [ ] No unexpected best-path selection

Useful additional checks:

```bash
get router info bgp network
get router info routing-table bgp
```

---

# 19. 🔍 OSPF Verification Checklist

* [ ] Neighbor state FULL
* [ ] Correct router ID
* [ ] Correct area
* [ ] Correct LSAs
* [ ] Expected routes installed
* [ ] No unexpected path selection

Useful commands:

```bash
get router info ospf neighbor
get router info ospf database
get router info routing-table ospf
```

---

# 20. 🔍 RIP Verification Checklist

* [ ] RIP enabled
* [ ] Correct version
* [ ] Correct networks
* [ ] Correct interfaces
* [ ] Split Horizon evaluated
* [ ] Expected routes received
* [ ] Expected routes advertised
* [ ] No routing loop

---

# 21. 🧪 ADVPN Packet-Flow Troubleshooting

Use a layered approach:

```text
                    Traffic Failure
                          |
                          v
                    Reachability
                          |
              +-----------+-----------+
              |                       |
              v                       v
          IPsec UP?                Routing OK?
              |                       |
              v                       v
          Phase 1/2              Next-Hop OK?
              |                       |
              +-----------+-----------+
                          |
                          v
                  ADVPN Discovery
                          |
                          v
                 Shortcut Created?
                          |
                          v
                    Policy Match?
                          |
                          v
                    NAT Correct?
                          |
                          v
                    Packet Flow
```

---

# 22. 🛠️ Useful Diagnostic Commands

### IPsec

```bash
diagnose vpn ike gateway list
diagnose vpn tunnel list
```

### Routing

```bash
get router info routing-table all
```

### BGP

```bash
get router info bgp summary
get router info routing-table bgp
```

### OSPF

```bash
get router info ospf neighbor
get router info ospf database
```

### Packet Capture

```bash
diagnose sniffer packet any 'host <IP>' 4 0 l
```

### Debug Flow

```bash
diagnose debug reset
diagnose debug flow filter addr <IP>
diagnose debug flow show function-name enable
diagnose debug enable
diagnose debug flow trace start 100
```

Stop:

```bash
diagnose debug disable
diagnose debug reset
```

> ⚠️ Always apply appropriate filters and limits when running debug commands in production.

---

# 23. 🔀 Dual ISP Failover Checklist

### Primary

```text
ISP-1
  |
link-1
  |
Primary IPsec
```

### Backup

```text
ISP-2
  |
link-2
  |
Backup IPsec
```

* [ ] Primary ISP identified
* [ ] Secondary ISP identified
* [ ] Primary tunnel established
* [ ] Secondary tunnel configured
* [ ] Tunnel monitoring configured
* [ ] Routing preference validated
* [ ] Failover tested
* [ ] Failback tested
* [ ] BGP/OSPF/RIP convergence verified
* [ ] ADVPN shortcut behavior after failover verified

Example:

```bash
config vpn ipsec phase1-interface
    edit "link-2"
        set monitor "link-1"
    next
end
```

---

# 24. 🧪 Failover Test Matrix

| Test                    | Expected Result           | Status |
| ----------------------- | ------------------------- | ------ |
| ISP-1 UP                | Primary tunnel active     | [ ]    |
| ISP-1 DOWN              | Backup tunnel active      | [ ]    |
| ISP-1 restored          | Failback behavior correct | [ ]    |
| Hub reachable           | BGP/OSPF/RIP established  | [ ]    |
| Remote spoke reachable  | Route installed           | [ ]    |
| Shortcut active         | Direct spoke path         | [ ]    |
| Shortcut after failover | Re-established correctly  | [ ]    |

---

# 25. 🚫 Static Route Audit

Static routes can interfere with dynamic ADVPN designs.

* [ ] List all static routes
* [ ] Identify overlapping prefixes
* [ ] Check administrative distance
* [ ] Check default route
* [ ] Check recursive routes
* [ ] Remove unnecessary static routes
* [ ] Confirm dynamic route becomes preferred

Concept:

```text
Dynamic ADVPN
      +
Static Route
      |
      v
Unexpected Best Path
```

---

# 26. 🧭 ADVPN Routing Decision Checklist

```text
Remote Spoke Prefix
        |
        v
Routing Table
        |
        v
Best Path
        |
        +---- Hub Path
        |
        +---- Shortcut Path
        |
        +---- Static Path
        |
        +---- Other Dynamic Path
```

* [ ] Best route identified
* [ ] Next-hop identified
* [ ] Shortcut route preferred when expected
* [ ] Static route not overriding dynamic design
* [ ] Routing loop absent
* [ ] Asymmetric routing checked

---

# 27. 🔐 Security Checklist

* [ ] Strong IPsec encryption selected
* [ ] Strong authentication selected
* [ ] Weak/legacy algorithms removed where possible
* [ ] PSK protected
* [ ] Management access restricted
* [ ] Firewall policies least-privilege
* [ ] NAT disabled unless intentionally required
* [ ] Logging enabled
* [ ] VPN events monitored
* [ ] Configuration backups protected
* [ ] Sensitive values removed before GitHub publication

### GitHub Publication Rule

```text
REAL PSK
REAL PUBLIC IP
REAL INTERNAL IP
REAL CUSTOMER NAME
REAL ROUTER-ID
REAL CREDENTIAL
        |
        X
     DO NOT
     PUBLISH
```

Use:

```text
<PSK>
<PUBLIC-IP>
<LAN-SUBNET>
<SPOKE-ID>
```

instead.

---

# 28. 📈 Large-Scale ADVPN Checklist

For large deployments:

* [ ] BGP considered as primary routing protocol
* [ ] Neighbor groups designed
* [ ] Neighbor ranges designed
* [ ] Route policies defined
* [ ] Prefix summarization considered
* [ ] Route filtering implemented
* [ ] Hub redundancy considered
* [ ] Dual ISP architecture considered
* [ ] Failure domains documented
* [ ] Monitoring implemented
* [ ] Configuration automation considered
* [ ] Maximum spoke scale documented
* [ ] Convergence requirements tested

---

# 29. 🏢 Hub Redundancy Checklist

If redundant hubs are used:

* [ ] Primary hub identified
* [ ] Secondary hub identified
* [ ] Routing preference defined
* [ ] ADVPN discovery behavior validated
* [ ] IPsec failover tested
* [ ] BGP/OSPF/RIP failover tested
* [ ] Route withdrawal tested
* [ ] Shortcut recovery tested
* [ ] Asymmetric routing tested

---

# 30. 📊 Production Readiness Checklist

Before production:

### IPsec

* [ ] Phase 1 stable
* [ ] Phase 2 stable
* [ ] DPD validated
* [ ] Rekey tested
* [ ] NAT-T tested where required

### ADVPN

* [ ] Sender/Receiver roles correct
* [ ] Shortcut creation verified
* [ ] Dynamic tunnel behavior understood
* [ ] `net-device` validated
* [ ] `device-creation` validated
* [ ] `add-route` validated

### Routing

* [ ] BGP/OSPF/RIP stable
* [ ] Prefix advertisements correct
* [ ] Next-hop correct
* [ ] Static routes audited
* [ ] Failover tested

### Security

* [ ] Encryption policy approved
* [ ] PSKs protected
* [ ] Firewall policies reviewed
* [ ] Logging enabled
* [ ] Monitoring configured

### Operations

* [ ] Troubleshooting commands documented
* [ ] Backup configuration available
* [ ] Rollback procedure documented
* [ ] Change window approved
* [ ] Failure scenarios tested

---

# 31. 🧠 ADVPN Golden Rules

> ### Rule 01
>
> **ADVPN is not just IPsec.**
> It combines IPsec connectivity, discovery, and routing behavior.

> ### Rule 02
>
> **Hub = Sender. Spoke = Receiver.**

> ### Rule 03
>
> **Hub connectivity and spoke-to-spoke shortcut connectivity are separate troubleshooting problems.**

> ### Rule 04
>
> **BGP is generally the strongest choice for large-scale ADVPN designs.**

> ### Rule 05
>
> **OSPF can be a useful alternative when an IGP design is preferred.**

> ### Rule 06
>
> **RIP requires special attention to Split Horizon when dynamic tunnel/device creation is involved.**

> ### Rule 07
>
> **Do not blindly enable `add-route` or `device-creation`.**

> ### Rule 08
>
> **Static routes can override the intended dynamic routing behavior.**

> ### Rule 09
>
> **Different AS designs require careful BGP next-hop planning.**

> ### Rule 10
>
> **Dual-ISP failover must be tested for both routing and ADVPN shortcut recovery.**

---

# 32. ⚡ 60-Second ADVPN Troubleshooting Checklist

When ADVPN is broken, check in this order:

```text
[ ] 01. WAN connectivity
[ ] 02. Phase 1
[ ] 03. Phase 2
[ ] 04. ADVPN Sender
[ ] 05. ADVPN Receiver
[ ] 06. Tunnel IP
[ ] 07. Routing neighbor
[ ] 08. Route advertisement
[ ] 09. Next-hop
[ ] 10. Dynamic tunnel creation
[ ] 11. Add Route
[ ] 12. Device Creation
[ ] 13. Static route conflicts
[ ] 14. Firewall policy
[ ] 15. NAT
[ ] 16. Shortcut establishment
[ ] 17. Packet flow
```

---

# 33. 🧪 Final ADVPN Validation

```text
                     ┌───────────────┐
                     │ WAN Reachable │
                     └───────┬───────┘
                             │
                             ▼
                     ┌───────────────┐
                     │   Phase 1 UP  │
                     └───────┬───────┘
                             │
                             ▼
                     ┌───────────────┐
                     │   Phase 2 UP  │
                     └───────┬───────┘
                             │
                             ▼
                     ┌───────────────┐
                     │ ADVPN Discovery│
                     └───────┬───────┘
                             │
                             ▼
                     ┌───────────────┐
                     │ Routing Works │
                     └───────┬───────┘
                             │
                             ▼
                     ┌───────────────┐
                     │   Shortcut    │
                     │   Established │
                     └───────┬───────┘
                             │
                             ▼
                     ┌───────────────┐
                     │ Data Path OK  │
                     └───────────────┘
```

* [ ] WAN reachability confirmed
* [ ] Phase 1 confirmed
* [ ] Phase 2 confirmed
* [ ] ADVPN discovery confirmed
* [ ] Routing confirmed
* [ ] Shortcut confirmed
* [ ] Data path confirmed
* [ ] Failover confirmed

---

# 34. 📌 Quick Reference

| Task               | Command / Area                                  |
| ------------------ | ----------------------------------------------- |
| VPN gateway status | `diagnose vpn ike gateway list`                 |
| VPN tunnel status  | `diagnose vpn tunnel list`                      |
| Routing table      | `get router info routing-table all`             |
| BGP summary        | `get router info bgp summary`                   |
| BGP routes         | `get router info routing-table bgp`             |
| OSPF neighbors     | `get router info ospf neighbor`                 |
| OSPF database      | `get router info ospf database`                 |
| OSPF routes        | `get router info routing-table ospf`            |
| Packet capture     | `diagnose sniffer packet any 'host <IP>' 4 0 l` |
| Debug flow         | `diagnose debug flow ...`                       |
| ADVPN Hub          | Auto-Discovery Sender                           |
| ADVPN Spoke        | Auto-Discovery Receiver                         |
| Dynamic routing    | BGP / OSPF / RIP                                |
| Shortcut           | Dynamic Spoke-to-Spoke Tunnel                   |

---

# 35. 🏁 Final Architecture Checklist

```text
                    INTERNET
                       |
              +--------+--------+
              |                 |
           ISP-1              ISP-2
              |                 |
              +--------+--------+
                       |
                 +-----+-----+
                 |    HUB    |
                 | ADVPN TX  |
                 |   BGP     |
                 +-----+-----+
                       |
          +------------+------------+
          |                         |
      +---+---+                 +---+---+
      |Spoke-1|                 |Spoke-2|
      |ADVPN RX                |ADVPN RX|
      +---+---+                 +---+---+
          \                         /
           \                       /
            \===== SHORTCUT =====/
```

### Final Sign-Off

* [ ] Architecture documented
* [ ] Hub configured
* [ ] Spokes configured
* [ ] IPsec validated
* [ ] ADVPN discovery validated
* [ ] Dynamic routing validated
* [ ] Spoke-to-spoke shortcut validated
* [ ] Firewall policies validated
* [ ] Static routes audited
* [ ] Dual ISP tested
* [ ] Failover tested
* [ ] Monitoring configured
* [ ] Security review completed
* [ ] Production rollback plan ready

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

## 🏷️ Keywords

`FortiGate` `ADVPN` `Auto Discovery VPN` `Fortinet ADVPN` `FortiGate ADVPN Configuration` `FortiGate ADVPN Troubleshooting` `FortiGate IPsec` `FortiGate BGP` `FortiGate OSPF` `FortiGate RIP` `Spoke-to-Spoke VPN` `Dynamic VPN` `Hub and Spoke VPN` `Fortinet VPN` `FortiGate Dual ISP` `ADVPN Shortcut` `FortiOS` `Network Security` `Fortinet NSE` `SheynShield`

---

> **⚠️ Version & Platform Note:**
> ADVPN behavior, CLI availability, routing behavior, dynamic tunnel creation, `net-device`, `add-route`, `device-creation`, and supported features can vary by FortiOS release and FortiGate platform. Always validate the configuration against the documentation and behavior of the target FortiOS version before deploying in production.

> **SheynShield Engineering Principle:**
> **Don't troubleshoot ADVPN as one feature. Break it into IPsec → Discovery → Routing → Next-Hop → Dynamic Tunnel → Shortcut → Policy → Data Path.**

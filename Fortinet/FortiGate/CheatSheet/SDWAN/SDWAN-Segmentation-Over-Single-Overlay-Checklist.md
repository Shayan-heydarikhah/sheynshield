# 🔗 SheynShield Resources

# FortiGate SD-WAN Advanced Routing & Segmentation Checklist — Part 1

> **FortiOS | SD-WAN + BGP + ADVPN + VRF + VPNv4 + MPLS + HA + Advanced Routing + Troubleshooting**
>
> **Purpose:** Practical validation checklist for advanced FortiGate SD-WAN architectures involving BGP multipath, ADVPN, multi-VRF, VPNv4, route leaking, SLA steering, HA and MPLS-based service-provider concepts.

---

## 📑 Table of Contents

* [1. Architecture Baseline](#1-architecture-baseline)
* [2. MPLS Support](#2-mpls-support)
* [3. SD-WAN and BGP Multipath](#3-sd-wan-and-bgp-multipath)
* [4. HA + BGP + SD-WAN](#4-ha--bgp--sd-wan)
* [5. HA Cluster and Hardware Switch](#5-ha-cluster-and-hardware-switch)
* [6. SD-WAN Segmentation](#6-sd-wan-segmentation)
* [7. VRF-Based SD-WAN Health Check](#7-vrf-based-sd-wan-health-check)
* [8. Overlay Stickiness](#8-overlay-stickiness)
* [9. ADVPN Shortcut Offers](#9-advpn-shortcut-offers)
* [10. MPLS Fundamentals](#10-mpls-fundamentals)
* [11. VPNv4](#11-vpnv4)
* [12. RD vs RT](#12-rd-vs-rt)
* [13. MPLS Traffic Flow](#13-mpls-traffic-flow)
* [14. Multi-VRF IPsec](#14-multi-vrf-ipsec)
* [15. VRF Roles](#15-vrf-roles)
* [16. VRF Limits](#16-vrf-limits)
* [17. Advanced VRF Configuration](#17-advanced-vrf-configuration)
* [18. Route Leaking](#18-route-leaking)
* [19. Hub BGP Configuration](#19-hub-bgp-configuration)
* [20. BGP Neighbor Groups and Ranges](#20-bgp-neighbor-groups-and-ranges)
* [21. BGP Network Advertisement](#21-bgp-network-advertisement)
* [22. ADVPN + VPNv4 Hub](#22-advpn--vpnv4-hub)
* [23. Spoke BGP Configuration](#23-spoke-bgp-configuration)
* [24. RD/RT Validation](#24-rdrt-validation)
* [25. BGP Additional Path](#25-bgp-additional-path)
* [26. SD-WAN Routing and FIB](#26-sd-wan-routing-and-fib)
* [27. VRF and VPNv4 Troubleshooting](#27-vrf-and-vpnv4-troubleshooting)
* [28. BGP Soft Clear](#28-bgp-soft-clear)
* [29. IPsec and ADVPN Troubleshooting](#29-ipsec-and-advpn-troubleshooting)
* [30. SD-WAN Health Check Troubleshooting](#30-sd-wan-health-check-troubleshooting)
* [31. End-to-End Validation](#31-end-to-end-validation)
* [32. Quick Command Reference](#32-quick-command-reference)
* [33. Final Architecture Checklist](#33-final-architecture-checklist)
* [34. Advanced Mental Model](#34-advanced-mental-model)

---

# 1. Architecture Baseline

Before troubleshooting or deploying advanced SD-WAN, validate the complete forwarding architecture.

### Core Architecture

* [ ] Identify all SD-WAN members.
* [ ] Identify SD-WAN zones.
* [ ] Identify all ISP/WAN connections.
* [ ] Identify overlay interfaces.
* [ ] Identify ADVPN hub and spokes.
* [ ] Identify BGP AS numbers.
* [ ] Identify BGP neighbors.
* [ ] Identify VRFs.
* [ ] Identify RD values.
* [ ] Identify RT import/export values.
* [ ] Identify VPNv4 requirements.
* [ ] Identify SLA/health-check objects.
* [ ] Identify HA mode.
* [ ] Identify routing domains.
* [ ] Identify expected traffic paths.
* [ ] Document expected failover behavior.

### Validate the Decision Chain

```text
Traffic
   ↓
Firewall Policy
   ↓
SD-WAN Rule
   ↓
SLA / Quality
   ↓
RIB / FIB
   ↓
BGP / Static / Connected Route
   ↓
SD-WAN Member
   ↓
Overlay / WAN
   ↓
Destination
```

> [!IMPORTANT]
> SD-WAN does not replace routing. SD-WAN path selection operates within the routing and forwarding context available to the FortiGate.

---

# 2. MPLS Support

### MPLS Capability Checklist

* [ ] Confirm whether the FortiGate is being used as an MPLS forwarding participant.
* [ ] Confirm where MPLS labels are generated.
* [ ] Confirm where labels are switched.
* [ ] Confirm the FortiGate is not expected to generate provider MPLS labels when the design does not support that function.
* [ ] Verify the external MPLS provider architecture.

Conceptually:

```text
FortiGate
   │
   └── MPLS label forwarding
```

> [!NOTE]
> In this architecture, FortiGate can forward MPLS labels but should not be treated as the provider's MPLS label-generation/label-switching router.

---

# 3. SD-WAN and BGP Multipath

When multiple BGP paths are available to SD-WAN, validate BGP multipath and additional-path requirements.

### BGP Configuration

```bash
config router bgp
    set ebgp-multipath enable
    set additional-path enable
    set additional-path-select 4
end
```

### Validation Checklist

* [ ] Confirm eBGP multipath requirement.
* [ ] Confirm iBGP multipath requirement.
* [ ] Confirm additional-path requirement.
* [ ] Confirm VPNv4 additional-path requirement.
* [ ] Confirm the selected number of additional paths.
* [ ] Verify BGP paths are actually received.
* [ ] Verify BGP paths are eligible for installation.
* [ ] Verify SD-WAN rules can evaluate the intended members.
* [ ] Verify SLA state for each candidate member.
* [ ] Verify link-down failover requirements.

Optional:

```bash
set link-down-failover enable
```

### Decision Model

```text
BGP
 │
 ├── Multiple Paths
 │
 ├── Multipath
 │
 └── Additional Path
          │
          ↓
      SD-WAN
          │
          ↓
       SLA/SLA
          │
          ↓
     WAN Selection
```

---

# 4. HA + BGP + SD-WAN

## Active-Passive HA

* [ ] Confirm HA mode is A-P.
* [ ] Confirm only the active unit forwards production traffic.
* [ ] Review whether BGP multipath is actually required for the HA design.
* [ ] Validate BGP behavior after failover.
* [ ] Validate SD-WAN member state after failover.
* [ ] Validate SLA state after failover.
* [ ] Validate route convergence.

## Active-Active HA

* [ ] Confirm HA mode is A-A.
* [ ] Review eBGP multipath requirements.
* [ ] Review iBGP multipath requirements.
* [ ] Validate path utilization.
* [ ] Validate session ownership.
* [ ] Validate routing convergence.

Example:

```bash
config router bgp
    set ebgp-multipath enable
end
```

> [!WARNING]
> Do not enable BGP multipath simply because multiple links exist. Validate the actual BGP path attributes, HA architecture and desired forwarding behavior first.

---

## HA Failure Checklist

If HA does not establish correctly:

* [ ] Verify HA group ID.
* [ ] Verify HA authentication.
* [ ] Verify heartbeat interfaces.
* [ ] Verify heartbeat connectivity.
* [ ] Verify firmware compatibility.
* [ ] Verify hardware/model compatibility.
* [ ] Check whether interfaces use DHCP.
* [ ] Check whether interfaces use PPPoE.
* [ ] Verify interface roles.
* [ ] Verify HA priority.
* [ ] Verify override behavior.
* [ ] Review HA logs.

### New Device Joining Existing Cluster

* [ ] Back up configuration.
* [ ] Verify hardware compatibility.
* [ ] Verify FortiOS version.
* [ ] Remove conflicting configuration.
* [ ] Factory reset only when operationally justified.
* [ ] Configure HA parameters.
* [ ] Connect heartbeat links.
* [ ] Join the cluster.
* [ ] Verify synchronization.

> [!CAUTION]
> Factory reset is a controlled maintenance operation and should be treated as a last-resort remediation step.

---

# 5. HA Cluster and Hardware Switch

Some FortiGate models provide hardware-switch capabilities that can simplify redundant ISP designs.

### Traditional Design

```text
       ISP-1                 ISP-2
         │                     │
         └──────── Switch ─────┘
                    │
             ┌──────┴──────┐
             │             │
           FGT-1         FGT-2
             └──── HA ────┘
```

### Hardware-Switch Design

```text
       ISP-1                 ISP-2
         │                     │
         └─────────┬───────────┘
                   │
            ┌──────┴──────┐
            │             │
          FGT-1         FGT-2
            │             │
            └── HA ───────┘
```

### Hardware-Switch Checklist

* [ ] Confirm the FortiGate model supports hardware switches.
* [ ] Confirm the hardware switch architecture.
* [ ] Verify ISP connectivity.
* [ ] Verify default gateways.
* [ ] Verify SD-WAN members.
* [ ] Verify static routes.
* [ ] Verify SD-WAN rules.
* [ ] Verify SLA health checks.
* [ ] Verify HA failover.
* [ ] Verify ISP failure detection.

### Software-Switch Consideration

* [ ] Verify whether the design uses a software switch.
* [ ] Confirm forwarding behavior on the secondary HA unit.
* [ ] Do not assume software-switch behavior is identical to hardware-switch behavior.

### HA Monitoring

* [ ] Verify whether individual physical ports can be directly monitored.
* [ ] Understand hardware-switch HA-monitor limitations.
* [ ] Use SD-WAN SLA health checks where appropriate.
* [ ] Test actual ISP-path failure rather than relying only on link state.

> [!IMPORTANT]
> In HA + hardware-switch + SD-WAN designs, SLA health checks can provide more meaningful path validation than simple physical link monitoring.

---

# 6. SD-WAN Segmentation

For segmentation over shared infrastructure, evaluate:

* [ ] VRF
* [ ] Layer-3 VPN
* [ ] MPLS
* [ ] VPNv4
* [ ] Multi-VRF IPsec
* [ ] ADVPN
* [ ] SD-WAN zones

### Segmentation Model

```text
Single Physical / Overlay Infrastructure
              │
              ↓
          Segmentation
              │
      ┌───────┼────────┐
      ↓       ↓        ↓
    VRF-10  VRF-20   VRF-30
```

### Validate

* [ ] Traffic isolation.
* [ ] Route isolation.
* [ ] Route import/export.
* [ ] SD-WAN rule matching.
* [ ] SLA context.
* [ ] Firewall policy.
* [ ] VPN tunnel association.
* [ ] Expected inter-VRF connectivity.

---

# 7. VRF-Based SD-WAN Health Check

Example:

```bash
config system sdwan
    config health-check
        edit sla-1
            set vrf 20
            set source 192.168.20.1
        next
    end
end
```

### Checklist

* [ ] Confirm the correct VRF.
* [ ] Confirm the source IP belongs to the expected VRF.
* [ ] Confirm the SLA destination is reachable from that VRF.
* [ ] Confirm the health check is associated with the intended SD-WAN members.
* [ ] Confirm latency/loss/jitter results.
* [ ] Confirm routing inside the VRF.
* [ ] Confirm failure detection.
* [ ] Confirm recovery behavior.

Concept:

```text
VRF 20
   │
   ↓
SLA Health Check
   │
   ↓
Source = 192.168.20.1
   │
   ↓
Path Quality
```

---

# 8. Overlay Stickiness

In multi-overlay environments, validate whether traffic should prefer returning through the same overlay from which it entered.

Useful in:

* [ ] Multi-overlay ADVPN.
* [ ] Multi-VRF designs.
* [ ] Dual-overlay architectures.
* [ ] Segmented SD-WAN.
* [ ] Hub-and-spoke designs.

---

## Zone-Level Tie-Break

```bash
config system sdwan
    config zone
        edit zone-1
            set service-sla-tie-break input-device
        next
    end
end
```

### Checklist

* [ ] Confirm the correct SD-WAN zone.
* [ ] Confirm `input-device` is appropriate.
* [ ] Confirm the incoming overlay is known.
* [ ] Confirm SLA eligibility.
* [ ] Test primary overlay selection.
* [ ] Test fallback behavior.

---

## Service-Level Tie-Break

```bash
config system sdwan
    config service
        edit 1
            set input-zone zone-1
            set tie-break input-device
        next
    end
end
```

### Expected Behavior

```text
Traffic
   ↓
Input Zone
   ↓
Input Device
   ↓
Prefer corresponding overlay
   ↓
SLA validation
   ↓
Fallback when required
```

---

# 9. ADVPN Shortcut Offers

Hub shortcut-offer behavior should be reviewed in large ADVPN deployments.

Example:

```bash
config vpn ipsec phase1-interface
    edit link-1
        set auto-discovery-offer-interval 5
    next
end
```

### Checklist

* [ ] Confirm ADVPN is enabled.
* [ ] Confirm the hub role.
* [ ] Confirm spoke discovery behavior.
* [ ] Confirm shortcut-offer interval.
* [ ] Confirm shortcut establishment.
* [ ] Monitor hub CPU during large-scale operation.
* [ ] Investigate repeated failed shortcut offers.
* [ ] Tune the interval when operationally required.

### Large-Scale Model

```text
                 HUB
                  │
       ┌──────────┼──────────┐
       ↓          ↓          ↓
    Spoke-1    Spoke-2    Spoke-N
       │          │          │
       └──── Shortcut Offers ┘
```

> [!WARNING]
> In large deployments, repeated failed shortcut attempts can create additional processing overhead. Validate actual platform behavior and capacity before tuning timers globally.

---

# 10. MPLS Fundamentals

## MPLS

**MPLS = Multiprotocol Label Switching**

### MPLS Label

* [ ] Understand that an MPLS label is 32 bits.
* [ ] Understand label-based forwarding.
* [ ] Understand label swapping.
* [ ] Understand ingress and egress behavior.

Concept:

```text
L2 Header
    ↓
MPLS Label
    ↓
IP Header
    ↓
Payload
```

---

## Label Switching Router — LSR

* [ ] Identify ingress LSR.
* [ ] Identify transit LSR/P routers.
* [ ] Identify egress LSR.
* [ ] Understand label push.
* [ ] Understand label swap.
* [ ] Understand label pop.

```text
Ingress LSR
    │
    │ Push
    ↓
P Router
    │
    │ Swap
    ↓
P Router
    │
    │ Swap
    ↓
Egress LSR
    │
    │ Pop
    ↓
Destination
```

---

# 11. VPNv4

VPNv4 combines:

```text
RD + IPv4 Prefix
```

### Checklist

* [ ] Understand IPv4 VPN route uniqueness.
* [ ] Configure the correct RD.
* [ ] Verify VPNv4 route advertisement.
* [ ] Verify VPNv4 route reception.
* [ ] Verify RT import/export.
* [ ] Verify VRF association.
* [ ] Verify next-hop reachability.

Example:

```text
IPv4 Prefix
10.1.1.0/24

RD
65001:100

VPNv4 Concept
65001:100:10.1.1.0/24
```

---

# 12. RD vs RT

## Route Distinguisher — RD

Purpose:

* [ ] Make overlapping customer prefixes unique.
* [ ] Identify the VPNv4 route namespace.
* [ ] Assign a unique RD where required.

Example:

```text
RD = 65001:100
```

---

## Route Target — RT

Purpose:

* [ ] Control route export.
* [ ] Control route import.
* [ ] Define VPN membership.
* [ ] Control VRF connectivity.

Example:

```text
RT = 65001:200
```

### RD vs RT Checklist

| Question                   | RD  | RT  |
| -------------------------- | --- | --- |
| Makes routes unique?       | [x] | [ ] |
| Controls route import?     | [ ] | [x] |
| Controls route export?     | [ ] | [x] |
| Controls VPN membership?   | [ ] | [x] |
| Used for VPNv4 uniqueness? | [x] | [ ] |

### Memory Model

```text
RD → Route Distinction
RT → Route Transfer
```

---

# 13. MPLS Traffic Flow

Example:

```text
Site-A
10.1.1.0/24
      │
      ↓
     CE1
      │
      ↓
     PE1
      │
   MPLS Core
      │
      ↓
     PE2
      │
      ↓
     CE2
      │
      ↓
10.1.2.0/24
```

### PE1

Example packet:

```text
[Transport Label][VPN Label][IP Packet]
       200             100
```

### Transit P Router

```text
Label 200
   ↓
Swap
   ↓
Label 300
```

Next router:

```text
Label 300
   ↓
Swap
   ↓
Label 400
```

### PE2

* [ ] Remove/resolve transport label.
* [ ] Process VPN label.
* [ ] Identify destination VRF.
* [ ] Remove VPN encapsulation/label as required.
* [ ] Forward toward CE2.

---

# 14. Multi-VRF IPsec

For multi-VRF behavior across an IPsec overlay, validate VPN-ID/IPIP encapsulation where supported by the target FortiOS/platform.

Example:

```bash
config vpn ipsec phase1-interface
    edit link-1
        set encapsulation vpn-id-ipip
    next
end
```

### Checklist

* [ ] Confirm target FortiOS supports the required encapsulation.
* [ ] Confirm VRF assignment.
* [ ] Confirm IPsec phase 1.
* [ ] Confirm IPsec phase 2.
* [ ] Confirm VPN-ID/IPIP behavior.
* [ ] Confirm remote VRF mapping.
* [ ] Verify traffic isolation.
* [ ] Verify routing.

Concept:

```text
VRF
 │
 ↓
VPN-ID / IPIP
 │
 ↓
IPsec Tunnel
 │
 ↓
Remote VRF
```

---

# 15. VRF Roles

FortiGate BGP VRF environments can use roles such as:

```text
CE
PE
Standalone
```

Example:

```bash
config router bgp
    config vrf
        edit 20
            set role ce
            set rd 1:1
            set export-rt 1:1
            set import-rt 1:1
        next
    end
end
```

### CE Checklist

* [ ] Confirm customer-side VRF.
* [ ] Confirm RD.
* [ ] Confirm RT.
* [ ] Confirm route advertisement.
* [ ] Confirm route reception.

### PE Checklist

* [ ] Confirm provider-edge role.
* [ ] Confirm multiple VRFs where required.
* [ ] Confirm VPNv4.
* [ ] Confirm RT import/export.
* [ ] Confirm route propagation.

### Standalone Checklist

* [ ] Confirm standalone routing requirements.
* [ ] Confirm no unintended PE/CE assumptions.
* [ ] Confirm expected route isolation.

---

# 16. VRF Limits

Validate platform-specific limits before deployment.

Example reference:

```text
Up to 64 VRFs per VDOM
```

### Checklist

* [ ] Confirm FortiGate model.
* [ ] Confirm FortiOS version.
* [ ] Verify current platform limits.
* [ ] Verify VDOM limits.
* [ ] Verify BGP VRF scalability.
* [ ] Verify SD-WAN scalability.
* [ ] Verify IPsec tunnel scalability.
* [ ] Verify expected route scale.

> [!IMPORTANT]
> Always validate scale limits against the exact FortiOS release and FortiGate model used in production.

---

# 17. Advanced VRF Configuration

Example:

```bash
config router bgp
    config vrf
        edit 0
            set role pe
        next

        edit 10
            set role ce
            set rd 1:1
            set export-rt 1:1
            set import-rt 1:1
        next

        edit 20
            set role ce
            set rd 2:1
            set export-rt 2:1
            set import-rt 2:1
        next
    end
end
```

### Validation

* [ ] VRF 0 has expected role.
* [ ] VRF 10 exists.
* [ ] VRF 20 exists.
* [ ] VRF 10 RD is correct.
* [ ] VRF 20 RD is correct.
* [ ] VRF 10 RT is correct.
* [ ] VRF 20 RT is correct.
* [ ] No unintended route import occurs.
* [ ] No unintended route export occurs.

---

# 18. Route Leaking

Route leaking allows selected routes to move between routing domains.

Example:

```bash
config router bgp
    config vrf
        edit 20
            config leak-target
                edit 10
                    set route-map rmp-test
                    set interface <interface>
                next
            end
        next
    end
end
```

### Checklist

* [ ] Identify source VRF.
* [ ] Identify destination VRF.
* [ ] Define exact routes to leak.
* [ ] Create route-map.
* [ ] Match only intended prefixes.
* [ ] Verify next-hop behavior.
* [ ] Verify interface requirements.
* [ ] Verify return path.
* [ ] Verify no accidental route leakage.

Concept:

```text
VRF-10
   │
   │ Selected Routes
   ↓
Route Map
   │
   ↓
VRF-20
```

> [!WARNING]
> Route leaking can intentionally break routing isolation. Use explicit route-map controls and validate the resulting forwarding table.

---

# 19. Hub BGP Configuration

Example:

```bash
config router bgp
    set as 65400
    set router-id 1.1.1.1

    set ebgp-multipath enable
    set ibgp-multipath enable

    set additional-path enable
    set additional-path-vpnv4 enable
    set additional-path-select 4
end
```

### Hub Checklist

* [ ] Confirm local AS.
* [ ] Confirm router ID.
* [ ] Confirm eBGP multipath requirement.
* [ ] Confirm iBGP multipath requirement.
* [ ] Confirm additional-path.
* [ ] Confirm VPNv4 additional-path.
* [ ] Confirm additional-path count.
* [ ] Confirm route-reflector requirements.
* [ ] Confirm next-hop behavior.
* [ ] Confirm neighbor groups.
* [ ] Confirm neighbor ranges.

---

# 20. BGP Neighbor Groups and Ranges

## Neighbor Group

Example:

```bash
config router bgp
    config neighbor-group
        edit "ad-grp"
            set next-hop-self-rr enable
            set soft-reconfiguration enable
            set soft-reconfiguration-vpnv4 enable

            set remote-as 65400

            set additional-path both
            set additional-path-vpnv4 both

            set adv-additional-path 4
            set adv-additional-path-vpnv4 4

            set route-reflector-client enable
            set route-reflector-client-vpnv4 enable
        next
    end
end
```

### Checklist

* [ ] Configure remote AS.
* [ ] Configure route-reflector behavior.
* [ ] Configure VPNv4 route-reflector behavior.
* [ ] Configure additional-path.
* [ ] Configure VPNv4 additional-path.
* [ ] Configure soft reconfiguration if required.
* [ ] Validate next-hop behavior.

---

## Neighbor Range

Example:

```bash
config router bgp
    config neighbor-range
        edit 1
            set prefix 12.23.34.0 255.255.255.0
            set neighbor-group "ad-grp"
        next

        edit 2
            set prefix 10.11.12.0 255.255.255.0
            set neighbor-group "ad-grp"
        next
    end
end
```

### Checklist

* [ ] Confirm neighbor-range prefix.
* [ ] Confirm correct neighbor group.
* [ ] Confirm dynamic neighbor establishment.
* [ ] Confirm expected AS.
* [ ] Confirm BGP capabilities.
* [ ] Confirm route exchange.

---

# 21. BGP Network Advertisement

Example:

```bash
config router bgp
    config network
        edit 1
            set prefix 192.168.101.0
            set mask 255.255.255.0
        next
    end
end
```

### Checklist

* [ ] Confirm prefix exists in the appropriate routing table.
* [ ] Confirm prefix/mask.
* [ ] Confirm BGP network statement.
* [ ] Confirm route advertisement.
* [ ] Confirm neighbor receives the route.
* [ ] Confirm route policy does not block it.

---

# 22. ADVPN + VPNv4 Hub

Example:

```bash
config vpn ipsec phase1-interface
    edit "hs-ad-23"
        set type dynamic
        set interface "port2"
        set peertype any
        set net-device disable

        set proposal des-md5 des-sha1
        set add-route disable

        set dpd on-idle

        set wizard-type hub-fortigate-auto-discovery
        set auto-discovery-sender enable

        set encapsulation vpn-id-ipip

        set psksecret ENC
    next
end
```

### Hub Checklist

* [ ] Confirm dynamic phase 1.
* [ ] Confirm correct WAN interface.
* [ ] Confirm peer type.
* [ ] Confirm `net-device` behavior.
* [ ] Confirm ADVPN configuration.
* [ ] Confirm auto-discovery sender.
* [ ] Confirm encapsulation.
* [ ] Confirm authentication.
* [ ] Confirm IKE proposal.
* [ ] Confirm DPD.
* [ ] Confirm route behavior.

> [!WARNING]
> Cryptographic proposals shown in study examples should not automatically be copied into production. Use current secure algorithms supported by the target FortiOS release.

---

# 23. Spoke BGP Configuration

Example:

```bash
config router bgp
    set ebgp-multipath enable
    set ibgp-multipath enable

    set additional-path enable
    set additional-path-vpnv4 enable

    set recursive-next-hop enable

    set additional-path-select 4
end
```

### Checklist

* [ ] Confirm eBGP multipath.
* [ ] Confirm iBGP multipath.
* [ ] Confirm additional-path.
* [ ] Confirm VPNv4 additional-path.
* [ ] Confirm recursive next-hop requirement.
* [ ] Confirm additional-path count.
* [ ] Verify BGP route installation.

---

## Spoke Neighbor

```bash
config router bgp
    config neighbor
        edit 12.23.34.1
            set capability-dynamic enable

            set soft-reconfiguration enable
            set soft-reconfiguration-vpnv4 enable

            set remote-as 65400

            set additional-path both
            set additional-path-vpnv4 both

            set adv-additional-path 4
            set adv-additional-path-vpnv4 4
        next
    end
end
```

### Checklist

* [ ] Verify neighbor IP.
* [ ] Verify dynamic capability if required.
* [ ] Verify remote AS.
* [ ] Verify additional-path.
* [ ] Verify VPNv4 additional-path.
* [ ] Verify advertised additional-path count.
* [ ] Verify soft reconfiguration.
* [ ] Verify BGP session state.

---

# 24. RD/RT Validation

Example isolated VRFs:

```text
VRF-10
RD = 1:1
RT = 1:1

VRF-20
RD = 2:1
RT = 2:1
```

### Checklist

* [ ] Each required VPN has an appropriate RD.
* [ ] RT import policy is correct.
* [ ] RT export policy is correct.
* [ ] Identical RTs are used intentionally.
* [ ] Different RTs are used where isolation is required.
* [ ] Overlapping IPv4 prefixes are handled correctly.
* [ ] VPNv4 routes appear in the expected VRF.
* [ ] Unwanted VRFs do not receive the route.

> [!IMPORTANT]
> **RD controls route uniqueness. RT controls route distribution.** Matching RDs alone does not create VPN connectivity.

---

# 25. BGP Additional Path

Use additional-path when multiple BGP paths need to remain available/advertised rather than only a single best path.

Example:

```bash
config router bgp
    set additional-path enable
    set additional-path-select 4
end
```

For VPNv4:

```bash
set additional-path-vpnv4 enable
```

### Checklist

* [ ] Confirm multiple paths exist.
* [ ] Confirm paths are eligible.
* [ ] Enable additional-path where required.
* [ ] Configure path selection count.
* [ ] Enable VPNv4 additional-path when required.
* [ ] Verify received paths.
* [ ] Verify advertised paths.
* [ ] Verify interaction with SD-WAN.

---

## Additional Path vs Multipath

| Feature                  | Purpose                                                  |
| ------------------------ | -------------------------------------------------------- |
| `ebgp-multipath`         | Allows multiple eligible eBGP paths to be used/installed |
| `ibgp-multipath`         | Allows multiple eligible iBGP paths to be used/installed |
| `additional-path`        | Allows additional BGP paths to be retained/advertised    |
| `additional-path-select` | Controls number of additional paths selected             |
| `additional-path-vpnv4`  | Applies additional-path behavior to VPNv4                |

### Memory Model

```text
Additional Path
    ↓
Keep / advertise more paths

Multipath
    ↓
Use / install multiple eligible paths
```

---

# 26. SD-WAN Routing and FIB

SD-WAN decisions must be evaluated together with routing.

### Routing Checklist

* [ ] Check connected routes.
* [ ] Check static routes.
* [ ] Check BGP routes.
* [ ] Check recursive next-hop.
* [ ] Check route preference.
* [ ] Check administrative distance.
* [ ] Check longest-prefix match.
* [ ] Check FIB installation.
* [ ] Check SD-WAN member eligibility.
* [ ] Check SLA state.

### Longest Prefix Match

Example:

```text
0.0.0.0/0
   ↓
SD-WAN

192.168.102.0/24
   ↓
port1
```

The `/24` is more specific than `/0`.

```text
192.168.102.0/24
        ↓
Longest Prefix Match
        ↓
Specific Egress
```

### Key Validation

* [ ] Identify the destination prefix.
* [ ] Identify all matching routes.
* [ ] Compare prefix lengths.
* [ ] Determine the winning route.
* [ ] Identify the FIB egress.
* [ ] Determine which SD-WAN members remain candidates.

---

# 27. VRF and VPNv4 Troubleshooting

## Filter by VRF

```bash
diagnose ip router bgp set-filter vrf 20
```

* [ ] Confirm correct VRF.
* [ ] Inspect BGP information.
* [ ] Check route installation.
* [ ] Check route selection.

---

## Filter by Neighbor

```bash
diagnose ip router bgp set-filter neighbor 192.168.101.1
```

* [ ] Confirm neighbor IP.
* [ ] Inspect neighbor-specific routes.
* [ ] Check advertised routes.
* [ ] Check received routes.

---

## Reset BGP Filter

```bash
diagnose ip router bgp set-filter reset
```

* [ ] Reset after targeted troubleshooting.
* [ ] Confirm subsequent diagnostics are not unintentionally filtered.

---

# 28. BGP Soft Clear

## Inbound

```bash
execute router clear bgp vpnv4 unicast soft in
```

## Outbound

```bash
execute router clear bgp vpnv4 unicast soft out
```

### Checklist

* [ ] Confirm policy change requires route reprocessing.
* [ ] Prefer soft clear where appropriate.
* [ ] Confirm session remains established.
* [ ] Confirm routes are re-evaluated.
* [ ] Verify resulting VPNv4 table.

Concept:

```text
BGP Session
     │
     ├── Keep Session
     │
     └── Re-process Routes / Policy
```

---

# 29. IPsec and ADVPN Troubleshooting

## Phase 1

```bash
diagnose vpn ike gateway list
diagnose vpn ike sa list
```

### Check

* [ ] Local IP.
* [ ] Remote IP.
* [ ] IKE version.
* [ ] Encryption.
* [ ] Authentication.
* [ ] Proposal.
* [ ] DPD.
* [ ] SA state.
* [ ] Lifetime.

---

## Phase 1 Established

```bash
diagnose vpn ike sa list
diagnose vpn ike sa filter
```

* [ ] Confirm established state.
* [ ] Confirm expected peer.
* [ ] Confirm cryptographic parameters.
* [ ] Confirm lifetime.
* [ ] Confirm correct interface.

---

## Phase 2

```bash
diagnose vpn ipsec sa list
diagnose vpn ipsec sa filter
```

### Check

* [ ] Local selectors.
* [ ] Remote selectors.
* [ ] Phase-2 proposal.
* [ ] SA establishment.
* [ ] Encryption counters.
* [ ] Decryption counters.

---

## Tunnel Verification

```bash
diagnose vpn ipsec sa list
diagnose vpn tunnel list
diagnose vpn tunnel stats
```

* [ ] IPsec SA exists.
* [ ] Tunnel is operational.
* [ ] Encryption counters increase.
* [ ] Decryption counters increase.
* [ ] Traffic counters increase.

### Important Diagnostic Pattern

```text
Phase 1 = UP
Phase 2 = DOWN
```

Check:

* [ ] Phase-2 configuration.
* [ ] Selectors.
* [ ] Proposal.
* [ ] IPsec interface.
* [ ] Tunnel association.

---

# 30. SD-WAN Health Check Troubleshooting

Check VRF-specific health status:

```bash
diagnose sys sdwan health-check status 20
```

### Checklist

* [ ] Confirm correct VRF.
* [ ] Confirm SLA object.
* [ ] Confirm source IP.
* [ ] Confirm target.
* [ ] Check latency.
* [ ] Check packet loss.
* [ ] Check jitter.
* [ ] Check member status.
* [ ] Check recovery status.
* [ ] Compare SLA state with actual packet forwarding.

---

# 31. End-to-End Validation

Perform validation from the client perspective rather than checking only individual components.

### Traffic Path

```text
Client
  │
  ↓
Firewall Policy
  │
  ↓
SD-WAN Rule
  │
  ├── Source
  ├── Destination
  ├── Application
  ├── DSCP
  └── SLA
  │
  ↓
Routing / FIB
  │
  ↓
SD-WAN Member
  │
  ↓
Overlay / ADVPN
  │
  ↓
BGP / VPNv4
  │
  ↓
VRF
  │
  ↓
Destination
```

### End-to-End Checklist

* [ ] Client has correct source address.
* [ ] Firewall policy matches.
* [ ] Application is identified correctly where applicable.
* [ ] SD-WAN rule matches.
* [ ] SLA is healthy.
* [ ] Correct SD-WAN zone is selected.
* [ ] Correct WAN member is eligible.
* [ ] FIB has the expected route.
* [ ] BGP has the expected prefix.
* [ ] VPNv4 route is present where required.
* [ ] RT import/export is correct.
* [ ] VRF is correct.
* [ ] ADVPN tunnel is established.
* [ ] IPsec SA is established.
* [ ] Return path exists.
* [ ] Session is established.
* [ ] Traffic counters increase.

---

# 32. Quick Command Reference

## BGP

```bash
get router info routing-table bgp
```

```bash
diagnose ip router bgp set-filter vrf 20
```

```bash
diagnose ip router bgp set-filter neighbor 192.168.101.1
```

```bash
diagnose ip router bgp set-filter reset
```

---

## Routing

```bash
get router info routing-table static
```

```bash
get router info filter show
```

```bash
get router info filter vrf all
```

```bash
get router info filter vrf 20
```

---

## SD-WAN

```bash
diagnose sys sdwan health-check status 20
```

---

## Sessions

```bash
diagnose sys session list
```

Look for VRF/VDOM context such as:

```text
vd=0:10
```

---

## IPsec / IKE

```bash
diagnose vpn ike gateway list
```

```bash
diagnose vpn ike sa list
```

```bash
diagnose vpn ike sa filter
```

```bash
diagnose vpn ipsec sa list
```

```bash
diagnose vpn ipsec sa filter
```

```bash
diagnose vpn tunnel list
```

```bash
diagnose vpn tunnel stats
```

---

## IKE Debug

```bash
diagnose debug application ike -1
diagnose debug enable
```

Look for:

* [ ] PSK mismatch.
* [ ] IP mismatch.
* [ ] Proposal mismatch.
* [ ] Phase-1 failure.
* [ ] Phase-2 failure.
* [ ] Selector mismatch.

---

## BGP VPNv4 Soft Clear

```bash
execute router clear bgp vpnv4 unicast soft in
```

```bash
execute router clear bgp vpnv4 unicast soft out
```

---

# 33. Final Architecture Checklist

## BGP

* [ ] Local AS verified.
* [ ] Router ID verified.
* [ ] eBGP multipath reviewed.
* [ ] iBGP multipath reviewed.
* [ ] Additional Path reviewed.
* [ ] VPNv4 Additional Path reviewed.
* [ ] Route reflector configuration verified.
* [ ] Neighbor groups verified.
* [ ] Neighbor ranges verified.
* [ ] Recursive next-hop reviewed.

## SD-WAN

* [ ] SD-WAN members verified.
* [ ] SD-WAN zones verified.
* [ ] SD-WAN rules verified.
* [ ] SLA objects verified.
* [ ] VRF-specific SLA verified.
* [ ] Tie-break behavior verified.
* [ ] Input-device behavior verified.
* [ ] Failover tested.
* [ ] Recovery tested.

## ADVPN

* [ ] Hub configuration verified.
* [ ] Spoke configuration verified.
* [ ] Auto-discovery verified.
* [ ] Shortcut behavior verified.
* [ ] Shortcut offer interval reviewed.
* [ ] Direct spoke-to-spoke connectivity tested.

## VRF

* [ ] VRFs identified.
* [ ] VRF roles verified.
* [ ] RD values verified.
* [ ] RT import values verified.
* [ ] RT export values verified.
* [ ] Route leaking reviewed.
* [ ] Route isolation tested.
* [ ] Inter-VRF connectivity tested.

## VPNv4

* [ ] VPNv4 enabled where required.
* [ ] VPNv4 routes received.
* [ ] VPNv4 routes advertised.
* [ ] RD verified.
* [ ] RT verified.
* [ ] Next-hop resolution verified.

## IPsec

* [ ] Phase 1 established.
* [ ] Phase 2 established.
* [ ] Correct selectors verified.
* [ ] Correct proposals verified.
* [ ] Tunnel counters verified.
* [ ] Encryption verified.
* [ ] Decryption verified.

## HA

* [ ] HA mode verified.
* [ ] Heartbeat links verified.
* [ ] HA synchronization verified.
* [ ] BGP behavior after failover tested.
* [ ] SD-WAN behavior after failover tested.
* [ ] Hardware-switch limitations reviewed.
* [ ] SLA-based failure detection tested.

---

# 34. Advanced Mental Model

```text
                         TRAFFIC
                            │
                            ▼
                   ┌─────────────────┐
                   │ Firewall Policy │
                   │                 │
                   │ Security       │
                   │ Application    │
                   └────────┬────────┘
                            │
                            ▼
                   ┌─────────────────┐
                   │    SD-WAN       │
                   │                 │
                   │ Rules           │
                   │ Zones           │
                   │ SLA             │
                   │ Tie-Break       │
                   └────────┬────────┘
                            │
                    ┌───────┴────────┐
                    ▼                ▼
                  BGP               FIB
                    │                │
                    ├── Multipath     ├── Prefix
                    ├── Additional   ├── LPM
                    │   Path          └── Egress
                    │
                    ▼
                SD-WAN Members
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
        ISP-1     ISP-2     ISP-3
          │         │         │
          ▼         ▼         ▼
       Overlay   Overlay   Overlay
          │         │         │
          └─────────┼─────────┘
                    ▼
                  ADVPN
                    │
                    ▼
                 VPNv4
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
       VRF-10    VRF-20    VRF-30
          │         │         │
          ▼         ▼         ▼
      Customer  Customer  Customer
```

## 🧠 The 10-Second Mental Model

```text
SD-WAN
→ Which path should I use?

BGP
→ Which routes/paths are available?

SLA
→ Is the path healthy?

RIB/FIB
→ Which route/egress wins?

Longest Prefix Match
→ Which destination route is most specific?

VRF
→ Which routing domain does this traffic belong to?

RD
→ How do I make overlapping VPN routes unique?

RT
→ Which VRFs are allowed to exchange routes?

ADVPN
→ Can spokes build dynamic direct tunnels?

VPNv4
→ How are customer IPv4 routes transported across VPN routing?

Additional Path
→ Can multiple BGP paths remain available/advertised?

Multipath
→ Can multiple eligible paths be installed/used?
```

---

# 🔥 Final Takeaways

> [!IMPORTANT]
> **SD-WAN = Path Selection**
>
> **BGP = Route/Path Discovery**
>
> **SLA = Path Quality**
>
> **RIB/FIB = Routing and Forwarding Decision**
>
> **VRF = Routing-Domain Isolation**
>
> **RD = VPN Route Uniqueness**
>
> **RT = VPN Route Distribution**
>
> **ADVPN = Dynamic Overlay Connectivity**
>
> **VPNv4 = IPv4 VPN Routing**
>
> **Additional Path = Multiple BGP Paths**
>
> **Multipath = Multiple Eligible Paths**
>
> **IPsec = Secure Overlay Transport**
>
> **HA = Device-Level Redundancy**

---

# 🔎 Keywords

`FortiGate SD-WAN` · `FortiOS SD-WAN` · `FortiGate BGP` · `BGP Multipath` · `BGP Additional Path` · `FortiGate VRF` · `VPNv4` · `FortiGate ADVPN` · `ADVPN Shortcut` · `MPLS FortiGate` · `Route Distinguisher` · `Route Target` · `RD vs RT` · `Multi-VRF IPsec` · `FortiGate HA` · `SD-WAN SLA` · `SD-WAN Routing` · `FIB` · `Longest Prefix Match` · `FortiGate Troubleshooting` · `Fortinet NSE7` · `Fortinet SD-WAN Troubleshooting`

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

## 🏷️ Tags

```text
fortigate
fortios
sdwan
sd-wan
bgp
bgp-multipath
additional-path
advpn
vrf
vpnv4
mpls
ipsec
ha
network-security
fortinet
nse7
routing
network-engineering
```

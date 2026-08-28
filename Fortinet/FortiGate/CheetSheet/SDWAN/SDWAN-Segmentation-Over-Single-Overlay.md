# FortiGate SD-WAN — Advanced Routing & Segmentation Cheatsheet — Part 1

> **Scope:** SD-WAN + BGP + ADVPN + VRF/VPNv4 + MPLS concepts + HA + Advanced troubleshooting

---

## 📑 Table of Contents

* [MPLS Support](#-mpls-support)
* [SD-WAN + BGP Multipath](#-sd-wan--bgp-multipath)
* [HA + BGP + SD-WAN](#-ha--bgp--sd-wan)
* [HA Cluster + SD-WAN Hardware Switch](#-ha-cluster--sd-wan-hardware-switch)
* [SD-WAN Segmentation over a Single Overlay](#-sd-wan-segmentation-over-a-single-overlay)
* [Overlay Stickiness](#-overlay-stickiness)
* [ADVPN Shortcut Offer](#-advpn-shortcut-offer)
* [MPLS Concepts](#-mpls-concepts)
* [VPNv4 Components](#-vpnv4-components)
* [MPLS Traffic Flow](#-mpls-traffic-flow)
* [Multi-VRF IPsec](#-multi-vrf-ipsec)
* [VRF Roles](#-vrf-roles)
* [VRF + BGP Advanced Configuration](#-vrf--bgp-advanced-configuration)
* [Hub BGP Configuration](#-hub-bgp-configuration)
* [Spoke BGP Configuration](#-spoke-bgp-configuration)
* [VRF / VPNv4 Troubleshooting](#-vrf--vpnv4-troubleshooting)
* [Quick Reference](#-quick-reference)

---

# 🔹 MPLS Support

FortiGate can **forward MPLS labels**, but it does **not generate MPLS labels**.

```text
FortiGate
   │
   ├── Can forward MPLS labels
   │
   └── Cannot generate MPLS labels
```

### Key Point

> FortiGate can participate in an MPLS forwarding scenario, but it is not an MPLS label-generation / label-switching router.

---

# 🔹 SD-WAN + BGP Multipath

When SD-WAN uses multiple BGP paths, especially when negotiating with different AS numbers, consider enabling:

```bash
config router bgp
    set ebgp-multipath enable
    set additional-path enable
    set additional-path-select 4
end
```

### Recommended

```text
SD-WAN
  │
  └── Multiple BGP Paths
          │
          ├── eBGP Multipath
          └── Additional Path
```

Also consider:

```bash
set link-down-failover enable
```

### Why?

```text
Multiple BGP paths
       │
       ├── SD-WAN SLA
       │
       ├── BGP multipath
       │
       └── Additional-path
              │
              └── More available paths for decision making
```

---

# 🔹 HA + BGP + SD-WAN

### Important HA consideration

If the FortiGate is operating as:

```text
A-P HA
```

the behavior of BGP multipath differs from:

```text
A-A HA
```

### Active-Passive

For an ISP-side BGP configuration, `ebgp-multipath` may not always be required in the same way because only the active unit forwards traffic.

### Active-Active

For A-A HA, consider:

```bash
config router bgp
    set ebgp-multipath enable
end
```

This helps FortiGate detect and use multiple paths toward the same AS.

---

## ⚠️ HA Troubleshooting

If HA does not start correctly after configuration:

### Check for DHCP / PPPoE

Make sure FortiGate interfaces used by the HA scenario are not using:

```text
DHCP
PPPoE
```

### New device joining HA

If you need to add a new device to an existing HA cluster and encounter unexplained HA problems:

```text
Factory reset
      ↓
Clean configuration
      ↓
Configure HA
      ↓
Join cluster
```

> Factory reset should be treated as a last-resort / controlled maintenance operation.

---

# 🔹 HA Cluster + SD-WAN Hardware Switch

Some FortiGate models with **hardware switches** can simplify redundant ISP connectivity.

### Traditional topology

```text
        ISP-1                 ISP-2
          │                     │
          │                     │
       ┌──┴─────────────────────┴──┐
       │         Switch Mesh        │
       └───────────┬───────────────┘
                   │
             ┌─────┴─────┐
             │           │
           FGT-1       FGT-2
             └──── HA ──┘
```

### Hardware-switch approach

```text
       ISP-1                 ISP-2
         │                     │
         │                     │
      ┌──┴─────────────────────┴──┐
      │                            │
    FGT-1                        FGT-2
      │                            │
      └──────── HA Heartbeat ──────┘
```

The built-in hardware switch can provide connectivity toward ISP routers without requiring additional external switches.

---

## ⚠️ Hardware Switch Limitations

This design requires FortiGate models that support **hardware switches**.

### Software Switch

Ports belonging to a software switch are **not in a forwarding state on the secondary unit** in an A-P HA cluster.

### HA Monitor

A port belonging to a hardware switch cannot be directly monitored as an individual HA monitor link.

Therefore:

```text
Hardware switch member fails
          │
          ↓
Both cluster members experience the same connectivity impact
```

A dedicated HA link-failure reaction is therefore not necessarily useful in this topology.

### Better monitoring mechanism

Use:

```text
SD-WAN SLA Health Check
```

to monitor each ISP path.

---

## ⚠️ Redesign Requirement

If you introduce:

```text
HA
+
Hardware Switch
+
SD-WAN
```

review:

* Static routes
* SD-WAN rules
* SLA configuration
* Gateway selection
* HA monitoring
* ISP connectivity
* Failover behavior

---

# 🔹 SD-WAN Segmentation over a Single Overlay

For segmentation over a single overlay, use technologies such as:

```text
VRF
Layer-3 VPN
MPLS
VPNv4
```

FortiGate can forward MPLS labels but does not generate them.

---

# 🔹 VRF-based SD-WAN Health Check

A health check can be associated with a specific VRF.

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

### Concept

```text
VRF 20
   │
   └── SLA-1
        │
        └── Source: 192.168.20.1
```

This allows health measurement to be performed in the required VRF context.

---

# 🔹 Overlay Stickiness

When a hub has multiple overlays:

```text
Hub
 ├── Overlay-1
 ├── Overlay-2
 ├── Overlay-3
 └── Overlay-4
```

you may want traffic received on one overlay to leave through the **same overlay**, when possible.

This is particularly useful for:

* Multi-overlay ADVPN
* Multi-VRF environments
* Dual-overlay architectures
* Segmented SD-WAN designs

---

## Zone-Level Tie Break

```bash
config system sdwan
    config zone
        edit zone-1
            set service-sla-tie-break input-device
        next
    end
end
```

### `input-device`

The incoming interface influences the SD-WAN decision.

Conceptually:

```text
Traffic enters
     │
     ↓
Overlay-1
     │
     ├── SLA valid → prefer Overlay-1
     │
     └── SLA invalid → consider another member
```

---

## Service-Level Tie Break

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

### Goal

```text
Input Zone
    │
    ↓
Input Device
    │
    ↓
Prefer same overlay/path
    │
    ↓
Fallback if SLA/path is unavailable
```

---

# 🔹 ADVPN Shortcut Offer

The hub periodically sends shortcut offers to spokes.

Default example:

```bash
config vpn ipsec phase1-interface
    edit link-1
        set auto-discovery-offer-interval 5
    next
end
```

```text
5 seconds = default example
```

### Large-scale deployment consideration

If many spokes exist:

```text
Hub
 │
 ├── Spoke-1
 ├── Spoke-2
 ├── Spoke-3
 ├── ...
 └── Spoke-N
```

Continuous shortcut offers can increase hub CPU/load if many offers repeatedly fail.

Therefore:

```text
Large number of spokes
        +
Frequent failed shortcut offers
        ↓
Potential hub load
        ↓
Tune offer interval
```

---

# 🔹 MPLS Concepts

## MPLS

**MPLS = Multiprotocol Label Switching**

Think of an MPLS label as a small forwarding instruction attached to the packet.

```text
L2 Header
    │
    ↓
MPLS Label
    │
    ↓
IP Header
    │
    ↓
Payload
```

An MPLS label is **32 bits**.

---

## Label Switching Router — LSR

Routers inside the MPLS network are generally called:

```text
LSR = Label Switching Router
```

### Basic flow

```text
Ingress LSR
    │
    │ Add label
    ↓
P Router
    │
    │ Swap label
    ↓
P Router
    │
    │ Swap label
    ↓
Egress LSR
    │
    │ Remove label
    ↓
Destination
```

---

# 🔹 Label Switched Path — LSP

```text
LSP = Label Switched Path
```

Think of an LSP as a predetermined forwarding path:

```text
CE1
 │
 ↓
PE1
 │
 ↓
P1
 │
 ↓
P2
 │
 ↓
PE2
 │
 ↓
CE2
```

---

# 🔹 Forwarding Equivalence Class — FEC

An **FEC** groups packets that should receive the same forwarding treatment.

Examples:

```text
Same destination
Same QoS requirement
Same forwarding behavior
```

Concept:

```text
Packets
 ├── Packet A
 ├── Packet B
 ├── Packet C
 └── Packet D
       │
       ↓
      FEC
       │
       ↓
 Same Label
       │
       ↓
 Same Path
```

---

# 🔹 VPNv4 Components

VPNv4 combines:

```text
RD + IPv4 Prefix
```

---

## Route Distinguisher — RD

The **RD** makes overlapping customer IPv4 routes unique inside the provider network.

Example:

```text
Customer prefix:

10.1.1.0/24
```

With:

```text
RD = 65001:100
```

Conceptually:

```text
65001:100:10.1.1.0/24
```

### RD formats

```text
<ASN>:<number>

Example:
65001:100
```

or:

```text
<IP>:<number>

Example:
192.168.1.1:100
```

---

# 🔹 Route Target — RT

The **Route Target** controls VPN route import/export between VRFs.

Example:

```text
RT = 65001:200
```

Conceptually:

```text
Customer-A
   │
   ├── Export RT 65001:200
   │
   ↓
Provider
   │
   └── Import RT 65001:200
          │
          ↓
      Customer-A VRFs
```

### RT is about:

```text
Route Import
Route Export
VPN membership
VRF connectivity
```

---

# 🔹 RD vs RT

| Feature                | RD                 | RT                         |
| ---------------------- | ------------------ | -------------------------- |
| Purpose                | Make routes unique | Control route distribution |
| Used for               | VPNv4 uniqueness   | Import/export policy       |
| Controls connectivity? | ❌                  | ✅                          |
| Example                | `65001:100`        | `65001:200`                |

### Easy memory trick

```text
RD → Route Distinction
RT → Route Transfer
```

---

# 🔹 MPLS Traffic Flow

Example:

```text
Site-A
10.1.1.0/24

        MPLS Core

Site-B
10.1.2.0/24
```

Traffic:

```text
10.1.1.1
   │
   ↓
CE1
   │
   ↓
PE1
```

PE1 adds:

```text
Transport Label = 200
VPN Label       = 100
```

Packet:

```text
[200][100][10.1.1.1 → 10.1.2.1]
```

---

## MPLS Core

P1 swaps:

```text
200 → 300
```

P2 swaps:

```text
300 → 400
```

So:

```text
PE1
 │
 │ [200][100]
 ↓
P1
 │
 │ [300][100]
 ↓
P2
 │
 │ [400][100]
 ↓
PE2
```

---

## PE2

PE2:

```text
Remove transport label
       ↓
Read VPN label
       ↓
Identify VRF
       ↓
Remove VPN label
       ↓
Forward to CE2
```

Final:

```text
CE2
 ↓
10.1.2.1
```

### Packet model

```text
┌─────────────────┬──────────────┬────────────────────────┐
│ Transport Label │ VPN Label    │ IP Packet               │
│      200        │     100      │ 10.1.1.1 → 10.1.2.1   │
└─────────────────┴──────────────┴────────────────────────┘
```

---

# 🔹 Multi-VRF IPsec

For segmentation where VRF information needs to travel through an IPsec overlay:

```bash
config vpn ipsec phase1-interface
    edit link-1
        set encapsulation vpn-id-ipip
    next
end
```

### Concept

```text
VRF
 │
 ↓
VPN-ID / IPIP encapsulation
 │
 ↓
IPsec Tunnel
 │
 ↓
Remote VRF
```

This enables multi-VRF behavior over an IPsec tunnel.

---

# 🔹 VRF Roles

FortiGate can define VRF roles such as:

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

---

## CE

```text
CE
│
├── Customer-side VRF
├── VPNv4
└── Customer route separation
```

---

## PE

```text
PE
│
├── Provider edge
├── Multiple VRFs
├── VPNv4
└── Route import/export
```

---

## Standalone

Used when the FortiGate operates without the complete PE/CE VPNv4 role model.

---

# 🔹 VRF Limit

Up to:

```text
64 VRFs per VDOM
```

---

# 🔹 Advanced VRF Configuration

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

### Architecture

```text
                    BGP / VPNv4
                         │
                         ▼
                    ┌─────────┐
                    │  VRF 0  │
                    │   PE    │
                    └────┬────┘
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
         ┌─────────┐           ┌─────────┐
         │ VRF 10  │           │ VRF 20  │
         │   CE    │           │   CE    │
         │ RD 1:1  │           │ RD 2:1  │
         │ RT 1:1  │           │ RT 2:1  │
         └─────────┘           └─────────┘
```

---

# 🔹 Route Leaking

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

### Purpose

Allow selected routes to move between routing domains.

```text
VRF-10
   │
   │ Route Leak
   ↓
VRF-20
```

Use route maps to control exactly which routes are leaked.

---

# 🔹 Hub BGP Configuration

Example advanced hub configuration:

```bash
config router bgp
    set as 65400
    set router-id 1.1.1.1

    set ebgp-multipath enable
    set ibgp-multipath enable

    set additional-path enable
    set additional-path-vpnv4 enable
    set additional-path-select 4
```

---

## Neighbor Group

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

---

# 🔹 BGP Neighbor Range

Instead of manually creating every neighbor:

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

### Concept

```text
Neighbor Range
      │
      ↓
Neighbor Group
      │
      ↓
Common BGP Parameters
      │
      ├── Additional Path
      ├── Route Reflector
      ├── Soft Reconfiguration
      └── VPNv4
```

---

# 🔹 BGP Network Advertisement

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

---

# 🔹 Hub VRF Configuration

```bash
config router bgp
    config vrf

        edit "0"
            set role pe
        next

        edit "10"
            set role ce
            set rd "1:1"
            set export-rt "1:1"
            set import-rt "1:1"
        next

        edit "20"
            set role ce
            set rd "2:1"
            set export-rt "2:1"
            set import-rt "2:1"
        next

    end
end
```

### Important

```text
VRF 0
 └── PE / provider context

VRF 10
 ├── RD 1:1
 └── RT 1:1

VRF 20
 ├── RD 2:1
 └── RT 2:1
```

---

# 🔹 ADVPN + VPNv4 Hub

Example phase 1:

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

### Important settings

```text
ADVPN
+
Auto Discovery
+
VPN-ID-IPIP
+
BGP VPNv4
```

---

# 🔹 ADVPN Phase 2

```bash
config vpn ipsec phase2-interface
    edit "hs-ad-23"

        set phase1name "hs-ad-23"

        set proposal des-md5 des-sha1

        set auto-discovery-forwarder enable

    next
end
```

### Purpose

The ADVPN forwarder capability helps support dynamic spoke-to-spoke connectivity.

```text
Spoke-1
   │
   │
   ▼
 Hub
   ▲
   │
   │
Spoke-2

        ↓ Shortcut

Spoke-1 ═════════ Spoke-2
       Direct
```

---

# 🔹 Spoke BGP Configuration

```bash
config router bgp

    set ebgp-multipath enable
    set ibgp-multipath enable

    set additional-path enable
    set additional-path-vpnv4 enable

    set recursive-next-hop enable

    set additional-path-select 4
```

### `recursive-next-hop`

Useful when route resolution requires recursive lookup, particularly around route-reflector / overlay designs.

```text
BGP Route
    │
    ↓
Next-Hop
    │
    ↓
Recursive Lookup
    │
    ↓
Reachable Interface
```

---

# 🔹 Spoke Neighbor

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

---

# 🔹 Spoke VRF

```bash
config router bgp
    config vrf

        edit "0"
            set role pe
        next

        edit "10"
            set role ce
            set rd "1:1"
            set export-rt "1:1"
            set import-rt "1:1"
        next

        edit "20"
            set role ce
            set rd "2:1"
            set export-rt "2:1"
            set import-rt "2:1"
        next

    end
end
```

---

# ⚠️ RD / RT Matching

If two VRFs use the same RD and RT:

```text
VRF-A
 RD = 1:1
 RT = 1:1

VRF-B
 RD = 1:1
 RT = 1:1
```

they are logically placed in the same VPN membership context.

For isolated VRFs:

```text
VRF-10
 RD = 1:1
 RT = 1:1

VRF-20
 RD = 2:1
 RT = 2:1
```

This keeps the VPN routing domains separated.

---

# 🔹 VRF / VPNv4 Troubleshooting

## Filter by VRF

```bash
diagnose ip router bgp set-filter vrf 20
```

---

## Filter by Neighbor

```bash
diagnose ip router bgp set-filter neighbor 192.168.101.1
```

---

## Reset Filter

```bash
diagnose ip router bgp set-filter reset
```

---

# 🔹 Clear VPNv4 BGP Soft

### Inbound

```bash
execute router clear bgp vpnv4 unicast soft in
```

### Outbound

```bash
execute router clear bgp vpnv4 unicast soft out
```

### Why?

Refresh BGP policy without completely tearing down the BGP session.

```text
BGP Session
     │
     ├── Keep session
     │
     └── Re-process routes/policy
```

---

# 🔹 Routing Table / VRF Verification

### General BGP routing table

```bash
get router info routing-table bgp
```

Useful for checking:

* BGP routes
* Recursive lookup
* VRF-related routing
* Next-hop resolution

---

## Filter Information

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

# 🔹 Session + VRF Verification

```bash
diagnose sys session list
```

Look for:

```text
vd=0:10
```

Conceptually:

```text
VDOM 0
  │
  └── VRF 10
```

---

# 🔹 SD-WAN Health Check per VRF

```bash
diagnose sys sdwan health-check status 20
```

### Purpose

Check health-check status for:

```text
VRF 20
```

---

# 🔹 Important ADVPN / IPsec Troubleshooting

## 1. Phase 1 Negotiation

Use:

```bash
diagnose vpn ike gateway list
diagnose vpn ike sa list
```

### Check

```text
✓ Local IP
✓ Remote IP
✓ IKE version
✓ Encryption
✓ Authentication
✓ SA establishment
```

---

# 🔹 Phase 1 Established

```bash
diagnose vpn ike sa list
diagnose vpn ike sa filter
```

Check:

```text
✓ Established state
✓ Lifetime
✓ Encryption algorithm
✓ Authentication algorithm
```

---

# 🔹 Phase 2 Negotiation

```bash
diagnose vpn ipsec sa list
diagnose vpn ipsec sa filter
```

Check:

```text
✓ Local subnet
✓ Remote subnet
✓ Phase-2 proposal
✓ SA establishment
```

---

# 🔹 Phase 2 Established

```bash
diagnose vpn ipsec sa list
diagnose vpn tunnel list
diagnose vpn tunnel stats
```

Check:

```text
✓ IPsec SA
✓ Tunnel state
✓ Encryption counters
✓ Decryption counters
✓ Traffic statistics
```

### Important clue

If:

```text
Phase 1 = UP
Phase 2 = DOWN
```

the Phase-2 name and IPsec interface may not appear as expected in:

```bash
diagnose vpn ipsec sa list
```

---

# 🔹 Tunnel Down

```bash
diagnose vpn ike gateway list
diagnose vpn ike sa list
```

Then:

```bash
diagnose debug application ike -1
diagnose debug enable
```

Look for:

```text
PSK mismatch
IP mismatch
Subnet mismatch
Proposal mismatch
Phase-1 failure
Phase-2 failure
```

---

# 🔹 BGP Additional Path

For environments where multiple valid paths need to remain visible:

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

Concept:

```text
             BGP
              │
       ┌──────┼──────┐
       ↓      ↓      ↓
     Path-1 Path-2 Path-3
       │      │      │
       └──────┼──────┘
              ↓
         SD-WAN / SLA
```

---

# 🔹 BGP Multipath + Additional Path

These features solve different problems.

| Feature                  | Purpose                               |
| ------------------------ | ------------------------------------- |
| `ebgp-multipath`         | Install/use multiple eBGP paths       |
| `ibgp-multipath`         | Install/use multiple iBGP paths       |
| `additional-path`        | Advertise/retain additional BGP paths |
| `additional-path-select` | Number of additional paths selected   |
| `additional-path-vpnv4`  | Additional paths for VPNv4            |

### Common advanced combination

```bash
config router bgp
    set ebgp-multipath enable
    set ibgp-multipath enable
    set additional-path enable
    set additional-path-vpnv4 enable
    set additional-path-select 4
end
```

---

# 🔹 Advanced Architecture

A typical advanced FortiGate design can combine:

```text
                    ┌──────────────────┐
                    │      BGP         │
                    │ Multipath        │
                    │ Additional Path  │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │     SD-WAN       │
                    │                  │
                    │ SLA / Steering   │
                    │ Zones / Rules    │
                    └────────┬─────────┘
                             │
             ┌───────────────┼───────────────┐
             ▼               ▼               ▼
          ISP-1           ISP-2           ISP-3
             │               │               │
             ▼               ▼               ▼
         Overlay-1       Overlay-2       Overlay-3
             │               │               │
             └───────────────┼───────────────┘
                             ▼
                          ADVPN
                             │
                             ▼
                       VPNv4 / VRF
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
           VRF-10         VRF-20         VRF-30
```

---

# 🔹 Quick Reference

## BGP Advanced

```bash
config router bgp
    set ebgp-multipath enable
    set ibgp-multipath enable
    set additional-path enable
    set additional-path-vpnv4 enable
    set additional-path-select 4
    set recursive-next-hop enable
end
```

---

## VRF

```bash
config router bgp
    config vrf
        edit 20
            set role ce
            set rd 2:1
            set export-rt 2:1
            set import-rt 2:1
        next
    end
end
```

---

## Multi-VRF IPsec

```bash
config vpn ipsec phase1-interface
    edit link-1
        set encapsulation vpn-id-ipip
    next
end
```

---

## SD-WAN VRF Health Check

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

---

## Overlay Stickiness

```bash
config system sdwan
    config zone
        edit zone-1
            set service-sla-tie-break input-device
        next
    end
end
```

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

---

## ADVPN Shortcut

```bash
config vpn ipsec phase1-interface
    edit link-1
        set auto-discovery-offer-interval 5
    next
end
```

---

## BGP Soft Clear

```bash
execute router clear bgp vpnv4 unicast soft in
execute router clear bgp vpnv4 unicast soft out
```

---

## VRF Diagnostics

```bash
get router info filter show
get router info filter vrf all
get router info filter vrf 20

get router info routing-table bgp

diagnose ip router bgp set-filter vrf 20
diagnose ip router bgp set-filter neighbor 192.168.101.1
diagnose ip router bgp set-filter reset

diagnose sys session list
diagnose sys sdwan health-check status 20
```

---

## 🔥 Mental Model

```text
                    TRAFFIC
                       │
                       ▼
                 SD-WAN RULE
                       │
             ┌─────────┴─────────┐
             │                   │
          SLA/SLA             BGP/FIB
             │                   │
             └─────────┬─────────┘
                       ▼
                  SD-WAN ZONE
                       │
             ┌─────────┼─────────┐
             ▼         ▼         ▼
           ISP-1     ISP-2     ISP-3
             │         │         │
             ▼         ▼         ▼
          Overlay    Overlay   Overlay
             │         │         │
             └─────────┼─────────┘
                       ▼
                     ADVPN
                       │
                       ▼
                 BGP / VPNv4
                       │
              ┌────────┼────────┐
              ▼        ▼        ▼
            VRF-10   VRF-20   VRF-30
              │        │        │
              ▼        ▼        ▼
           Customer Customer Customer
```

### 🧠 Key distinction

```text
SD-WAN
  → Which path should I use?

BGP
  → Which routes/paths are available?

SLA
  → Is the path healthy?

FIB/RIB
  → Is the destination reachable through this path?

VRF
  → Which routing domain does this traffic belong to?

RD
  → How do I make overlapping routes unique?

RT
  → Which VRFs are allowed to exchange routes?

ADVPN
  → Can spokes build direct dynamic tunnels?

Additional Path
  → Can I retain/advertise multiple BGP paths?

Multipath
  → Can I install/use multiple equal paths?

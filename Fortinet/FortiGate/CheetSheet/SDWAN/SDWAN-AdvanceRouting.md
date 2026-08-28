# FortiGate SD-WAN — Advanced Routing & Overlay Cheat Sheet

> FortiOS SD-WAN advanced routing, BGP, route-tag, ADVPN, SD-WAN neighbor, FEC and troubleshooting reference.

---

## 📑 Table of Contents

- [1. Local Out Routing](#1-local-out-routing)
- [2. SD-WAN with BGP](#2-sd-wan-with-bgp)
- [3. SD-WAN with BGP Route Tags](#3-sd-wan-with-bgp-route-tags)
- [4. SD-WAN with ADVPN](#4-sd-wan-with-advpn)
- [5. Traffic Control with BGP Route Maps](#5-traffic-control-with-bgp-route-maps)
- [6. SD-WAN Neighbor Roles](#6-sd-wan-neighbor-roles)
- [7. Multiple Members per SD-WAN Neighbor](#7-multiple-members-per-sd-wan-neighbor)
- [8. SD-WAN + ADVPN Overlay](#8-sd-wan--advpn-overlay)
- [9. ADVPN Troubleshooting by Phase](#9-advpn-troubleshooting-by-phase)
- [10. Hold-Down Time](#10-hold-down-time)
- [11. Forward Error Correction (FEC)](#11-forward-error-correction-fec)
- [12. FEC Mapping](#12-fec-mapping)
- [13. Useful Troubleshooting Commands](#13-useful-troubleshooting-commands)

---

# 1. Local Out Routing

## Concept

**Local Out Routing** controls how FortiGate-generated traffic reaches external services.

Examples:

- DNS
- FortiGuard
- NTP
- FortiCloud
- Central Management
- Active probes
- Other FortiGate-generated traffic

### GUI

```text
Feature Visibility
└── Local Out Routing
````

---

## Routing Methods

| Method      | Behavior                   |
| ----------- | -------------------------- |
| `auto`      | Best available selection   |
| `sdwan`     | Use SD-WAN rules/services  |
| `interface` | Force a specific interface |

### `auto`

```text
Best available path
```

### `sdwan`

```text
Use SD-WAN service/rule
        |
        └── If no matching SD-WAN rule
                |
                └── Dynamic/Static routing
```

### `interface`

```text
Use a dedicated interface
```

---

## Traceroute with SD-WAN

```bash
execute traceroute-options use-sdwan yes
execute traceroute 8.8.8.8
```

---

## Central Management

```bash
config system central-management
    set interface-select-method specify
    set interface port1
end
```

---

## NTP

```bash
config system ntp
    config ntpserver
        edit 1
            set interface-select-method specify
            set interface port1
        next
    end
end
```

---

## DHCP Proxy

```bash
config system settings
    set dhcp-proxy-interface-select-method specific
    set dhcp-proxy-interface port1
end
```

---

## TLS Active Probe

```bash
config system global
    config tls-active-probe
        set interface-select-method sdwan
    end
end
```

### Local-Out Routing Decision

```text
SD-WAN Rule
     |
     v
Dynamic Routing
     |
     v
Static Routing
```

> Always verify the destination and route-selection sequence before troubleshooting Local-Out behavior.

---

# 2. SD-WAN with BGP

## Topology Concept

```text
                  ISP / WAN
               /            \
          Link-1            Link-2
             |                |
             +------ FGT -----+
                    |
                 SD-WAN
                    |
                 BGP
```

---

## SD-WAN Rules

Create one SLA:

```text
SLA-1
├── Server: 22.22.22.2
└── Server: 2.2.2.2
```

Then create SD-WAN rules.

### Rule 1 — Main

```text
Source       : all
Destination  : 192.168.102.0/24
Outgoing     : zone-main
SLA          : SLA-1
```

### Rule 2 — Backup

```text
Source       : all
Destination  : 192.168.102.0/24
Outgoing     : zone-back
SLA          : SLA-1
```

---

## BGP

Example:

```text
Remote AS  : 65002
Neighbor   : 2.2.2.2

Remote AS  : 65002
Neighbor   : 22.22.22.2
```

### Recommended

Use:

* Interface-based neighbors
* Soft reconfiguration
* BGP multipath when required

---

## ISP Side

Configure:

```text
FGT-X
├── BGP
├── IP addressing
└── SD-WAN
```

### Do not advertise/inject unnecessary routes

The ISP side should provide the required BGP connectivity without injecting unwanted customer routes.

---

## SD-WAN Zones

```text
zone-fgt-1
├── fgt-1-interface-1
└── fgt-1-interface-2

zone-fgt-2
├── fgt-2-interface-1
└── fgt-2-interface-2
```

---

## SLA

### SLA-FGT-1

```text
Servers:
    1.1.1.1
    11.11.11.1

Probe:
    Active

Target:
    Default
```

### SLA-FGT-2

```text
Servers:
    2.2.2.1
    22.22.22.1

Probe:
    Active

Target:
    Default
```

---

## BGP Multipath

For multipath scenarios:

```text
EBGP Multipath
        +
Multi-Hop
        +
Multiple valid BGP paths
```

Enable the required multipath and multihop settings on each hop.

---

# 3. SD-WAN with BGP Route Tags

## Goal

Use BGP route tags to steer traffic into different SD-WAN paths.

```text
BGP Route
   |
   v
Route-Map IN
   |
   v
Route Tag
   |
   v
SD-WAN Rule
   |
   v
Preferred WAN
```

---

## BGP Neighbors

```text
Remote AS 65002
    |
    +── 2.2.2.2
    |
    +── 22.22.22.2
```

Recommended:

```text
Interface-based neighbor
Soft reconfiguration
Route-map-in
```

---

## Route Map

### ACL

```text
ACL-20
├── Prefix: 192.168.20.0/24
├── Exact Match
└── Permit
```

### Route Map

```text
RMP-20
├── Rule 1
│   ├── ACL: ACL-20
│   ├── Tag: 20
│   └── Permit
│
└── Rule 2
    ├── ACL: ACL-ANY
    └── Permit
```

---

## SD-WAN Rule Using Route Tag

```bash
config system sdwan
    config service
        edit 1
            set route-tag 20
        next
    end
end
```

---

## Example Steering

### SD-WAN Rule 1

```text
Source       : all
Destination  : all
Service      : all
Strategy     : Best Quality
Zone         : Main
SLA          : SLA-1
```

### SD-WAN Rule 2

```text
Source       : all
Destination  : all
Service      : all
Strategy     : Best Quality
Zone         : Back
SLA          : SLA-1
```

---

## Refresh BGP

```bash
execute router clear bgp all soft
```

---

## Verification

```bash
get router info bgp network
```

Look for:

```text
Network
Prefix
Next-Hop
Path
Tag
```

---

## Policy Route Debug

```bash
diagnose firewall proute list
```

> `vwl` = Virtual Wire Load Balancing / virtual SD-WAN interfaces.

---

# 4. SD-WAN with ADVPN

## Concept

```text
                HUB
              /     \
          Spoke-1   Spoke-2
              \     /
             ADVPN
                +
             SD-WAN
```

ADVPN allows spokes to dynamically establish direct tunnels with other spokes instead of always forwarding traffic through the hub.

---

## Hub

### ADVPN Main

```text
Role:
    Hub

PSK:
    123456

Tunnel IP:
    12.23.34.1

Remote:
    12.23.34.2/24

Local AS:
    65400

Local Interface:
    port3

Local Subnet:
    192.168.101.0/24
```

### Spokes

```text
12.23.34.2
12.23.34.3
```

---

## Hub ADVPN Settings

```text
Auto-Discovery Sender     : ENABLE
Add Route                 : DISABLE
Device Creation           : DISABLE
```

---

## Spoke ADVPN Settings

```text
Auto-Discovery Receiver   : ENABLE
Device Creation           : ENABLE
Add Route                 : DISABLE
```

---

## Important

If SD-WAN zones/policies are created around ADVPN:

```text
Delete auto-created policies
        |
        v
Create SD-WAN zones
        |
        v
Create SD-WAN rules
        |
        v
Verify static routes
        |
        v
Apply SLA
```

### ADVPN + SD-WAN Warning

```text
DO NOT use:
    Maximize Bandwidth / Load-Balance

with ADVPN members.
```

Prefer:

```text
SLA
Priority
Best Quality
```

---

# 5. Traffic Control with BGP Route Maps

## Goal

Combine:

```text
BGP
+
SD-WAN SLA
+
Route Maps
+
Neighbor Awareness
```

to control traffic based on WAN health.

---

## Route Map Concept

```text
             SLA OK
               |
               v
        Prefer Route Map
               |
               v
        Preferred Path

             SLA FAIL
               |
               v
         Normal Route Map
               |
               v
          Backup Path
```

---

## ACL Example

### ACL-ANY

```text
Prefix: ANY
Exact Match
Permit
```

### ACL-103

```text
Prefix: 192.168.103.0/24
Exact Match
Permit
```

---

## Normal Route Map

```text
RTM-DENY-103

Rule 1:
    Action: DENY
    ACL: ACL-103

Rule 2:
    Action: PERMIT
    ACL: ACL-ANY
```

---

## Preferred Route Map

```text
RTM-BYPASS

Rule 1:
    Action: PERMIT
    ACL: ACL-103

Rule 2:
    Action: PERMIT
    ACL: ACL-ANY
```

---

## BGP Neighbor

```bash
config router bgp
    config neighbor
        edit 12.23.34.1
            set route-map-out rtm-deny-103
            set route-map-out-prefer rtm-bypass
        next
    end
end
```

Refresh:

```bash
execute router clear bgp all soft
```

---

# 6. SD-WAN Neighbor Roles

## Roles

| Role         | Behavior                           |
| ------------ | ---------------------------------- |
| `primary`    | Preferred neighbor when SLA is met |
| `secondary`  | Used when primary fails            |
| `standalone` | Independent operation              |

---

## Primary / Secondary

```text
             SLA OK
               |
               v
           PRIMARY
               |
             FAIL
               |
               v
          SECONDARY
```

---

## SD-WAN Service

### Primary

```bash
config system sdwan
    config service
        edit 1
            set role primary
            set dst all
            set src all
            set priority-members 1
        next
```

### Secondary

```bash
        edit 2
            set role secondary
            set dst all
            set src all
            set priority-members 2
        next
```

### Standalone

```bash
        edit 3
            set role standalone
            set dst all
            set src all
            set priority-members 1 2
        next
    end
end
```

---

## SD-WAN Neighbor

```bash
config system sdwan
    config neighbor

        edit 12.23.34.1
            set member 1
            set role primary
            set health-check SLA_AD
            set sla-id 1
        next

        edit 12.23.34.2
            set member 1
            set role primary
            set health-check SLA_AD
            set sla-id 1
        next

        edit 10.11.12.1
            set member 2
            set role secondary
            set health-check SLA_AD
            set sla-id 1
        next

        edit 10.11.12.2
            set member 2
            set role secondary
            set health-check SLA_AD
            set sla-id 1
        next

    end
end
```

---

## Neighbor Health Check

```bash
config system sdwan
    config health-check
        edit SLA_AD
            set server 12.23.34.1 10.11.12.1

            set members 0

            config sla
                edit 1
                next
            end
        next
    end
end
```

---

## Alternative: Separate SLA Checks

```bash
config system sdwan
    config health-check

        edit sla-1
            set server 10.11.12.2
            set members 1

            config sla
                edit 1
                    set link-cost-factor packet-loss
                    set packetloss-threshold 1
                next
            end
        next

        edit sla-2
            set server 12.23.34.2
            set members 2

            config sla
                edit 1
                    set link-cost-factor packet-loss
                    set packetloss-threshold 1
                next
            end
        next

    end
end
```

---

## Troubleshooting

```bash
diagnose sys sdwan health-check
diagnose sys sdwan neighbor
diagnose sys sdwan service
```

---

# 7. Multiple Members per SD-WAN Neighbor

Useful when a neighbor has multiple links.

```bash
config system sdwan
    config neighbor
        edit 12.23.34.1
            set member 1 2
            set minimum-sla-meet-member 2
        next
    end
end
```

### Meaning

```text
Neighbor
├── Member 1
└── Member 2

Minimum SLA Meet:
    2
```

The preferred route-map is used only when the required number of members meet the SLA.

Otherwise:

```text
Preferred state
      |
    FAIL
      |
      v
Normal route-map
```

---

# 8. SD-WAN + ADVPN Overlay

## ADVPN

ADVPN enables:

```text
Hub-and-Spoke
       |
       v
Dynamic Spoke-to-Spoke Tunnels
       |
       v
Full-Mesh-like Connectivity
```

### Benefits

* Lower latency
* Less hub traffic
* Less provisioning effort
* Better scalability
* Dynamic spoke-to-spoke connectivity

---

## Dual WAN + ADVPN

```text
                 HUB
              /       \
          Main         Backup
           |             |
        Spoke-1       Spoke-2
           \             /
            \___________/
              ADVPN
```

Combined with SD-WAN:

```text
ADVPN
  +
SD-WAN
  |
  +── Load balancing
  +── SLA-based steering
  +── Priority
  +── Performance selection
```

### Important

```text
SD-WAN Load-Balance mode
        +
ADVPN
        =
NOT SUPPORTED
```

Prefer:

```text
SLA
Priority
Best Quality
```

---

## Recommended ADVPN Settings

### Hub

```text
Add Route          : DISABLE
Device Creation    : DISABLE
Auto-Discovery Sender : ENABLE
```

### Spoke

```text
Auto-Discovery Receiver : ENABLE
Device Creation         : ENABLE
Add Route               : DISABLE
```

---

# 9. ADVPN Troubleshooting by Phase

## Phase 1 — Negotiating

### Commands

```bash
diagnose vpn ike gateway list
diagnose vpn ike sa list
```

### Verify

```text
Local IP
Remote IP
IKE version
Encryption
Authentication
PSK
```

---

## Phase 1 — UP

```bash
diagnose vpn ike sa list
diagnose vpn ike sa filter
```

### Verify

```text
IKE SA
Encryption
Authentication
Lifetime
State = ESTABLISHED
```

---

## Phase 2 — Negotiating

```bash
diagnose vpn ipsec sa list
diagnose vpn ipsec sa filter
```

### Verify

```text
Local selector
Remote selector
Phase-2 proposal
Encryption
Authentication
PFS
```

---

## Phase 2 — UP

```bash
diagnose vpn ipsec sa list
diagnose vpn tunnel list
diagnose vpn tunnel stats
```

### Verify

```text
IPsec SA
Tunnel state
Traffic counters
Encryption
Decryption
Lifetime
```

> If Phase 1 is UP but Phase 2 fails, the Phase 2 name and IPsec interface may not appear in `ipsec sa list`.

---

## Tunnel Down

```bash
diagnose vpn ike gateway list
diagnose vpn ike sa list

diagnose debug application ike -1
diagnose debug enable
```

### Check

```text
PSK
Local/Remote IP
IKE version
Phase-1 proposal
Phase-2 proposal
Selectors
DH group
Authentication
```

---

## Useful IPsec Commands

### IKE Gateways

```bash
diagnose vpn ike gateway list
```

Shows:

```text
Gateway
IP addresses
IKE version
Status
```

### IKE SA

```bash
diagnose vpn ike sa list
```

Shows:

```text
SA ID
IPs
Algorithms
Lifetime
```

### IPsec Tunnels

```bash
diagnose vpn tunnel list
```

Shows:

```text
Tunnel
IPs
Algorithms
Status
Traffic statistics
```

### IPsec SA

```bash
diagnose vpn ipsec sa list
```

Shows:

```text
SA ID
IPs
Algorithms
Lifetime
Tunnel interface
```

---

## Restart IPsec

```bash
diagnose vpn ipsec restart
```

Restarts:

```text
IPsec tunnels
IPsec SAs
```

---

## Restart IKE

```bash
diagnose vpn ike restart
```

Restarts:

```text
IKE negotiations
IKE SAs
```

---

## IPsec Packet Debug

```bash
diagnose vpn ipsec packet
```

Useful for:

```text
Packet headers
Encryption status
Decryption status
```

---

# 10. Hold-Down Time

## Problem

WAN link flapping:

```text
UP
 |
DOWN
 |
UP
 |
DOWN
 |
UP
```

can cause repeated SD-WAN path switching.

---

## Solution

Use `hold-down-time`.

```bash
config system sdwan
    config service
        edit 1
            set hold-down-time 100
        next
    end
end
```

### Meaning

```text
100 seconds
```

Wait before switching back to the preferred path after failover.

---

## Behavior

```text
Primary
   |
   v
FAIL
   |
   v
Secondary
   |
   v
Primary RECOVERED
   |
   | hold-down
   v
Primary
```

### Important

```text
0 = no hold-down protection
```

---

## Verification

```bash
diagnose sys sdwan service
```

Look for:

```text
Hold-down timer
Service state
Selected member
```

---

# 11. Forward Error Correction (FEC)

## Concept

FEC adds redundant packets so the receiver can reconstruct lost packets without retransmission.

```text
Base Packets
    +
Redundant Packets
    |
    v
VPN
    |
    v
Packet Loss
    |
    v
Recovery
```

---

## Example

```text
Base      = 10
Redundant = 1
```

For every:

```text
10 base packets
```

send:

```text
1 redundant packet
```

---

## FEC and Packet Loss

Example mapping:

| Packet Loss | FEC                                |
| ----------: | ---------------------------------- |
|     `0–10%` | No redundant packet                |
|    `10–30%` | 1 redundant packet                 |
|      `>30%` | More redundancy / duplicate packet |

> Actual behavior depends on the configured FEC mapping/profile.

---

## FEC Use Cases

Best for:

```text
VoIP
UDP
Real-time traffic
Loss-sensitive applications
```

Avoid unnecessary FEC for:

```text
TCP
Non-loss-sensitive traffic
```

---

## FEC and NPU

```text
FEC
 |
 X
NPU Offload
```

FEC-protected traffic may not be NPU-offloaded.

Therefore:

```text
FEC traffic
    |
    v
CPU processing
```

while non-FEC traffic may remain offloaded.

---

## Phase 1 FEC

Enable on **both sides**:

```bash
config vpn ipsec phase1-interface
    edit link-1
        set npu-offload enable
        set fec-egress enable
        set fec-ingress enable
        set fec-health-check 1
        set fec-mapping-profile map-1
        set fec-base 10
        set fec-redundant 1
    next
end
```

---

# 12. FEC Mapping

FEC mappings associate:

```text
Network SLA
     +
Packet Loss
     +
Bandwidth
     |
     v
Base / Redundant Packets
```

---

## Example

```bash
config vpn ipsec fec
    edit map-1

        config mappings

            edit 1
                set base 8
                set redundant 2
                set packet-loss-threshold 10
            next

            edit 2
                set base 9
                set redundant 3
                set bandwidth-up-threshold 950000
            next

        end
    next
end
```

---

## Mapping Evaluation

Mappings are evaluated:

```text
Top
 |
 v
Mapping 1
 |
 v
Mapping 2
 |
 v
Mapping 3
 |
 v
...
```

Example:

```text
Packet loss > 10%
    |
    v
Base      = 8
Redundant = 2
```

Another condition:

```text
Upload bandwidth > 950 Mbps
    |
    v
Base      = 9
Redundant = 3
```

---

## FEC Firewall Policy

### UDP / VoIP

```bash
config firewall policy
    edit 1
        set srcintf port3
        set dstintf sdwan-1
        set action accept
        set srcaddr 192.168.101.0/24
        set dstaddr all
        set schedule always
        set service ALL_UDP
        set fec enable
    next

    edit 2
        set srcintf any
        set dstintf any
        set action accept
        set srcaddr all
        set dstaddr all
        set schedule always
        set service ALL
    next
end
```

---

## FEC Design

```text
                    Traffic
                       |
             +---------+---------+
             |                   |
            UDP                 TCP
             |                   |
             v                   v
          FEC ON              FEC OFF
             |                   |
             v                   v
        CPU processing       NPU possible
```

---

# 13. Useful Troubleshooting Commands

## SD-WAN

```bash
diagnose sys sdwan service
```

```bash
diagnose sys sdwan health-check
```

```bash
diagnose sys sdwan neighbor
```

```bash
diagnose sys sdwan packet-history
```

---

## SD-WAN + SLA

```bash
diagnose sys sdwan health-check
```

Check:

```text
Member
SLA
Latency
Jitter
Packet loss
Status
```

---

## FEC

```bash
diagnose vpn tunnel fec link-1
```

```bash
diagnose sys sdwan health-check
```

```bash
diagnose sys sdwan packet-history
```

---

## Session / NPU

```bash
diagnose sys session list
```

Example:

```text
npu info:
flag=0x82/0x81
offload=8/8
ips_offload=0/0
epid=249/74
ipid=74/86
```

### Interpretation

```text
0x82 / 0x81
    |
    +── NPU offload indicators
```

For FEC:

```text
TCP + No FEC
    -> NPU offload possible

UDP + FEC
    -> NPU offload unavailable
```

---

# 🧠 SD-WAN Decision Model

```text
                         Packet
                            |
                            v
                    SD-WAN Service
                            |
             +--------------+--------------+
             |              |              |
             v              v              v
          Manual       Best Quality     Lowest Cost
             |              |              |
             |              v              v
             |            SLA             SLA
             |              |              |
             +--------------+--------------+
                            |
                            v
                     Route / FIB Check
                            |
                            v
                    Selected SD-WAN Member
                            |
                            v
                         Gateway
```

---

# 🔀 SD-WAN + BGP + ADVPN

```text
                    ┌──────────────┐
                    │     HUB      │
                    │   BGP/ADVPN  │
                    └──────┬───────┘
                           │
                  ┌────────┴────────┐
                  │                 │
              Main WAN          Backup WAN
                  │                 │
              ┌───┴───┐         ┌───┴───┐
              │Spoke-1│         │Spoke-2│
              └───┬───┘         └───┬───┘
                  │                 │
                  └───── ADVPN ─────┘
                         |
                      SD-WAN
                         |
          +--------------+--------------+
          |              |              |
         SLA          Priority        BGP
          |              |              |
          +--------------+--------------+
                         |
                    Best Forwarder
```

---

# ⚠️ Important Design Notes

| Feature              | Key Point                                          |
| -------------------- | -------------------------------------------------- |
| ADVPN                | Dynamic spoke-to-spoke tunnels                     |
| SD-WAN               | SLA-based path selection                           |
| BGP                  | Dynamic route exchange                             |
| Route Tag            | Route-based steering                               |
| Route Map            | Route filtering / manipulation                     |
| Neighbor Role        | Primary / Secondary / Standalone                   |
| Hold-Down            | Prevents path flapping                             |
| FEC                  | Recovers packet loss with redundancy               |
| FEC + NPU            | FEC traffic may not be NPU-offloaded               |
| ADVPN + Load Balance | Avoid load-balance mode                            |
| ADVPN + SLA          | Preferred                                          |
| BGP Multipath        | Required for multiple equal paths                  |
| Additional Path      | Useful when multiple BGP paths must remain visible |
| Passive SLA          | Requires passive WAN measurement                   |
| Local-Out            | Controls FortiGate-generated traffic               |
| FIB                  | Important in SD-WAN route selection                |

---

# 🚦 Quick Troubleshooting Flow

```text
Traffic Problem
      |
      v
Is SD-WAN rule matching?
      |
   +--+--+
   |     |
  NO    YES
   |     |
   v     v
Check    Is SLA healthy?
policy       |
route     +--+--+
          |     |
         NO    YES
          |     |
          v     v
      Check SLA  Check selected
      server     member
                   |
                   v
              Check FIB/RIB
                   |
                   v
              Check BGP
                   |
                   v
              Check ADVPN
                   |
                   v
              Check IPsec
                   |
                   v
             Phase 1 / Phase 2
                   |
                   v
              Check NPU/FEC
                   |
                   v
             Check Session
```

---

# 📌 One-Liner Reference

```text
SD-WAN Rule
    = Who + Where + What + SLA + Preferred Path

SLA
    = Latency + Jitter + Packet Loss + Cost

BGP
    = Dynamic Route Exchange

BGP Route Tag
    = Route Classification → SD-WAN Steering

Route Map
    = Route Filtering / Attribute Manipulation

ADVPN
    = Dynamic Spoke-to-Spoke IPsec

SD-WAN Neighbor
    = Primary / Secondary / Standalone

Hold-Down
    = Anti-Flapping Protection

FEC
    = Redundancy → Packet Loss Recovery

FIB
    = Forwarding Decision

RIB
    = Routing Information

NPU
    = Hardware Offload
```

---

# 🔧 Core Verification Commands

```bash
# SD-WAN
diagnose sys sdwan service
diagnose sys sdwan health-check
diagnose sys sdwan neighbor
diagnose sys sdwan packet-history

# Routing
get router info routing-table all
get router info routing-table static
get router info routing-table bgp

# BGP
get router info bgp network
execute router clear bgp all soft

# Policy Routing
diagnose firewall proute list

# IPsec / IKE
diagnose vpn ike gateway list
diagnose vpn ike sa list
diagnose vpn ipsec sa list
diagnose vpn tunnel list

# IPsec Debug
diagnose debug application ike -1
diagnose debug enable

# FEC
diagnose vpn tunnel fec link-1

# Sessions
diagnose sys session list
```

---

# 🎯 Design Checklist

* [ ] Define SD-WAN zones
* [ ] Define WAN members
* [ ] Configure accurate gateways
* [ ] Configure estimated bandwidth
* [ ] Configure SLA probes
* [ ] Select active/passive/prefer-passive probing
* [ ] Define SD-WAN service/rules
* [ ] Select steering strategy
* [ ] Validate FIB/RIB behavior
* [ ] Configure BGP if dynamic routing is required
* [ ] Enable BGP multipath when required
* [ ] Use route-tags for route-based steering
* [ ] Use route-maps for route filtering
* [ ] Configure ADVPN if dynamic tunnels are required
* [ ] Avoid SD-WAN load-balance mode with ADVPN
* [ ] Configure SD-WAN neighbor roles when required
* [ ] Configure hold-down for unstable links
* [ ] Use FEC only for loss-sensitive traffic
* [ ] Verify NPU implications
* [ ] Verify firewall policies
* [ ] Verify routing table
* [ ] Verify SD-WAN service selection
* [ ] Verify IPsec Phase 1
* [ ] Verify IPsec Phase 2
* [ ] Verify session/NPU offload

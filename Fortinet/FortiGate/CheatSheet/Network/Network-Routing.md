# FortiGate Routing

## 1. Static Route

### Default Route

- Administrative Distance:
  - Static: `10`
  - DHCP: `5`

### Routing Table Commands

```bash
get router info routing-table all
# Show active and inactive routes

get router info routing-table details
# Detailed routing information

get router info routing-table database
# All received and learned routes

get router info kernel
# Show FIB and routing daemon information
````

### Kernel Routing Table Fields

```text
TAB
254 = Unicast
255 = Multicast
```

```text
VF
= VDOM name/index
= 0 when no VDOM
```

### Route Type

| Value | Type        |
| ----: | ----------- |
|     0 | Unspecified |
|     1 | Unicast     |
|     2 | Local       |
|     3 | Broadcast   |
|     4 | Anycast     |
|     5 | Multicast   |
|     6 | Blackhole   |
|     7 | Unreachable |
|     8 | Prohibited  |

### Route Protocol

| Value | Protocol                           |
| ----: | ---------------------------------- |
|     0 | Unspecified                        |
|     2 | Kernel                             |
|    11 | ZebOS routing module               |
|    14 | FortiOS                            |
|    15 | HA — learned from HA               |
|    16 | Authentication based               |
|    17 | HA1 — learned from heartbeat links |

```text
prio
= Lower value is preferred

pref
= Preferred next-hop
```

---

# 2. Route Lookup

### Route Cache

```bash
diagnose ip rtcache list
# Check route lookup cache
```

Configure route-cache size:

```bash
config system global
    set max-route-cache-size 100
end
```

### Route Lookup Order

```text
1. Policy Route
       ↓
2. Route Cache
       ↓
3. FIB
       ↓
4. Drop if no match
```

### Specific Route Lookup

```bash
get router info routing-table details 4.4.4.4
```

### IP Address List

```bash
diagnose ip address list
# Show all assigned IP addresses
```

> Route lookup contains RIB + route cache information used by sessions.

> Blackhole route priority is normally configured through CLI.

---

# 3. Reverse Path Forwarding — RPF

### Strict Source Check

```bash
config system setting
    set strict-src-check enable
end
```

### Feasible-Path RPF

* Default behavior.
* FortiGate checks whether the source is reachable through **any valid interface/path**.
* Helps prevent IP spoofing.

### Strict RPF

* Checks source and destination interfaces more strictly.
* Can cause problems in asymmetric-routing designs.

### Disable Source Check Per Interface

Useful when asymmetric routing is required:

```bash
config system interface
    edit port8
        set src-check disable
    next
end
```

### Testing Example

```bash
nmap -D 10.10.10.1,192.168.101.1,192.168.101.3 192.168.101.2

nmap 192.168.101.3 192.168.101.2
```

---

# 4. Asymmetric Routing

FortiGate normally expects established TCP sessions to follow the expected state/3-way handshake.

Enable asymmetric routing:

```bash
config system setting
    set asymroute enable
end
```

> ⚠️ Asymmetric routing can reduce security inspection/state tracking.

> Better to keep it disabled unless the design requires it.

---

# 5. Routing Change Without SNAT

Preserve the session route when routing changes:

```bash
config system interface
    edit port4
        set preserve-session-route enable
    next
end
```

### Use Case

Useful when:

* Dynamic routing changes
* HA topology changes
* Route changes occur
* Existing sessions should keep their original route

> By default, routing changes can flush/recalculate session routing.

> May have implications for X-Forwarded-For and related traffic behavior.

---

# 6. Routing Change With SNAT

```bash
config system global
    set snat-route-change enable
end
```

### Behavior

When route changes:

```text
SNAT Session
     ↓
New Route Lookup
     ↓
New Session Path
```

Useful for:

* Dynamic routing
* WAN/link failure
* Continuous forwarding through another link

> Default: `disable`

### Session Troubleshooting

```bash
diagnose sys session filter src 192.168.101.2

diagnose sys session list
# Check session state, stages and order
```

---

# 7. Policy Route

> Policy routes should normally be used together with a valid/default routing path.

---

# 8. ECMP — Equal Cost Multipath

ECMP requires multiple routes with the same destination and equal routing preference/cost.

### ECMP Methods

| Method             | Description                                                    |
| ------------------ | -------------------------------------------------------------- |
| `source-ip-based`  | Sessions from the same source IP use the same path             |
| `weighted-based`   | Sessions distributed according to configured interface weights |
| `usage-based`      | Interface used until bandwidth threshold is reached            |
| `src-dst-ip-based` | Same source + destination pair uses same path                  |
| `volume-based`     | Workload distributed based on packet volume; SD-WAN only       |

### Global ECMP

```bash
config system setting
    set v4-ecmp-mode usage-base
    set ecmp-max-path 4
end
```

### SD-WAN Load Balancing

```bash
config system sdwan
    set status enable
    set load-balance-mode usage-based
end
```

### Troubleshooting

```bash
diagnose sys vd list

get router info routing-table all
```

---

# 9. ECMP + BGP Multipath

```bash
config router bgp
    set as 65000
    set router-id 192.168.254.252
    set ebgp-multipath enable
end
```

> BGP multipath and global ECMP are related but controlled by different mechanisms.

---

# 10. Dual Internet Connections

### Recommended

Use:

```text
SD-WAN
```

Alternative designs:

```text
Link Redundancy
Load Sharing
Combination of Redundancy + Load Sharing
```

---

# 11. Link Redundancy / Health Check

```bash
config system link-monitor
    edit 1
        set status enable
        set addr-mode ipv4
        set srcintf port1
        set server 8.8.8.8
        set protocol ping
        set gateway-ip 192.168.254.2
        set interval 30
        set failtime 5
        set recovertime 5
        set update-static-route enable
        set update-cascade-interface enable
    next
end
```

### Important Options

```text
update-static-route
= Remove/update static routes when link fails

update-cascade-interface
= Cascade failure state to related interfaces
```

### Design Tip

Use different:

```text
Administrative Distance
Priority
```

for primary and backup routes when designing failover.

---

# 12. Load Sharing

With ECMP/load sharing:

```text
WAN1 ─────┐
          ├── FortiGate ── Internet
WAN2 ─────┘
```

If both routes are available:

```text
Traffic → WAN1 + WAN2
```

If one link fails:

```text
Traffic → Remaining Link
```

### Without Link Monitor

If:

```text
WAN interface = UP
Remote gateway = DOWN
```

FortiGate may continue sending traffic toward the primary WAN.

Result:

```text
Traffic interruption
```

If the physical WAN interface itself goes down:

```text
Route removed
     ↓
Secondary WAN becomes active
```

---

# 13. RIP

### RIP Timers

```bash
config router rip
    set timeout-timer 30
    set update-timer 180
    set garbage-timer 30
end
```

> Timers should be consistent across RIP devices.

---

# 14. RIP Authentication / Key Chain

```bash
config router key-chain
    edit 1
        config key
            edit 1
                set key-string 123
                set accept-lifetime 12:20:00 8 2 2025 infinit
                set send-lifetime 12:20:00 8 2 2025 infinit
            next
        end
    next
end
```

Apply to RIP:

```bash
config router rip
    config interface
        edit port1
            set auth-mode md5
            set auth-string 123
            set auth-keychain 1
        next
    end
end
```

### RIP Troubleshooting

```bash
get router info rip

get router info rip interfaces
# Includes split-horizon information

get router info rip database
```

### Source Ping

```bash
execute ping-options source 1.2.3.4

execute ping 8.8.8.8
```

---

# 15. OSPF

### Multiple OSPF Domains

If multiple routing domains/processes are required:

```text
Prefer VRF
```

to separate routing processes where appropriate.

### Basic OSPF

```bash
config router ospf
    set router-id 10.11.101.1

    config area
        edit 0.0.0.0
        next
    end

    config ospf-interface
        edit Router1-Internal-DR
            set interface port1
            set priority 255
            set dead-interval 40
            set hello-interval 10
        next

        edit Router1-External
            set interface port2
            set dead-interval 40
            set hello-interval 10
        next
    end

    config network
        edit 1
            set prefix 10.11.0.0 255.255.0.0
        next

        edit 2
            set prefix 192.168.102.0 255.255.255.0
        next
    end
end
```

---

# 16. OSPF Troubleshooting

### OSPF Configuration

```bash
get router ospf
```

### OSPF Routing Table

```bash
get router info routing-table ospf
```

### OSPF Neighbors

```bash
get router info ospf neighbor

get router info ospf neighbor all

get router info ospf neighbor details
```

### OSPF Database

```bash
get router info ospf database brief

get router info ospf database adv-router 3.3.3.3
```

### OSPF Routes

```bash
get router info ospf route
```

### OSPF Interfaces

```bash
get router info ospf interface
```

### Complete Routing Table

```bash
get router info routing-table all
```

### OSPF Status

```bash
get router info ospf status
```

Shows information such as:

```text
Process uptime
VRF
Area ID
Area information
SPF updates
Data exchanges
```

---

# 17. OSPF Graceful Restart

Enable restart on topology changes:

```bash
config router ospf
    set restart-on-topology-change enable
end
```

### HA + OSPF Graceful Restart

Example:

```text
Helper Gateway
      |
      v
FortiGate HA Cluster
      |
      v
Cisco Gateway
```

Use graceful restart mechanisms on:

```text
Helper Gateway
HA Cluster
```

Example:

```bash
config router ospf
    set router-id 31.1.1.1
    set restart-mode graceful-restart
    set restart-period 180
    set restart-on-topology-change enable

    config area
        edit 0.0.0.0
        next
    end

    config network
        edit 1
            set prefix 172.16.200.0 255.255.255.0
        next

        edit 2
            set prefix 31.1.1.1 255.255.255.255
        next
    end
end
```

### Purpose

```text
HA switch
   ↓
Routing engine temporarily unavailable
   ↓
Forwarding continues
   ↓
OSPF SPF recalculation
   ↓
Routing restored
```

---

# 18. BGP — Basic Configuration

```bash
config router bgp
    set as 64511
    set router-id 1.1.1.1

    config neighbor
        edit 10.100.201.88
            set remote-as 64511
            set update-source toFGTB
        next
    end

    config network
        edit 1
            set prefix 192.168.86.0 255.255.255.0
        next
    end
end
```

### Update Source

Useful when:

* iBGP uses loopbacks
* Different interface/IP domains are used
* BGP peering should use a specific source interface

---

# 19. BGP Troubleshooting

### Best Paths

```bash
get router info bgp paths
```

### All Routing Paths

```bash
get router info routing-table all
```

### BGP Neighbors

```bash
get router info bgp neighbors
```

### BGP Summary

```bash
get router info bgp summary
```

### BGP Networks

```bash
get router info bgp networks

get router info bgp network 12.23.34.2
```

### Soft Reset

```bash
execute router clear bgp all soft
```

### BGP TCP Capture

```bash
diagnose sniffer packet any 'tcp and port 179'
```

---

# 20. BGP Neighbor States

| State         | Meaning / Troubleshooting                                                  |
| ------------- | -------------------------------------------------------------------------- |
| `Idle`        | BGP session not established; check configuration/reachability              |
| `Connect`     | TCP connection process; check routing/TCP connectivity                     |
| `Active`      | BGP is attempting/re-attempting connection; check return path/reachability |
| `Established` | BGP session established                                                    |
| `SIA`         | Investigate response/update problems                                       |

> For `Idle`, `Connect`, or `Active`, check routing, TCP/179, ACL/firewall and peer configuration.

---

# 21. BGP Recursive Next-Hop

Normal BGP scenarios generally do not require recursive next-hop.

For special Layer-3 designs:

```bash
config router bgp
    set recursive-next-hop enable
end
```

### Multipath + Recursive Next-Hop

```bash
config router bgp
    set as 65001
    set router-id 1.1.1.1
    set ebgp-multipath enable
    set ibgp-multipath enable
    set recursive-next-hop enable
end
```

---

# 22. BGP Loopback Peering

Example:

```bash
config router bgp
    config neighbor
        edit 12.12.12.2
            set remote-as 65002
            set update-source lp-1
            set ebgp-enforce-multihop enable
            set ebgp-multihop-ttl 3
            set soft-reconfiguration enable
        next
    end
end
```

### Important

If BGP peering uses loopbacks:

```text
update-source
+
reachability to loopback
+
eBGP multihop when required
```

---

# 23. BGP Network + Backdoor

```bash
config router bgp
    config network
        edit 1
            set prefix 12.12.12.1 255.255.255.255
        next

        edit 2
            set prefix 12.12.12.2 255.255.255.255
            set backdoor enable
        next
    end
end
```

---

# 24. BGP ECMP vs Global ECMP

### Global ECMP

```bash
config system setting
    set v4-ecmp-mode usage-base
    set ecmp-max-path 4
end
```

### BGP Multipath

```bash
set ebgp-multipath enable
set ibgp-multipath enable
```

> These are different mechanisms.

---

# 25. BGP Route Maps

Concept:

```text
ACL / Match Conditions
        ↓
Route Map
        ↓
Action
```

Example:

```bash
config router route-map
    edit exclude1
        config rule
            edit 1
                set action deny
                set match-origin igp
            next
        end
    next
end
```

---

# 26. BGP Conditional Advertisement

### Community List

```bash
config router community-list
    edit 30:5
        config rule
            edit 1
                set action permit
                set match 30:5
            next
        end
    next
end
```

### Route Map

```bash
config router route-map
    edit comm1
        config rule
            edit 1
                set match-community 30:5
                set set-route-tag 15
            next
        end
    next
end
```

### Conditional Advertisement Maps

```bash
config router bgp
    config neighbor
        edit 2.2.2.2
            config conditional-advertise
                edit 2224
                    set condition-routemap 2814 2224 comm1
                    set condition-type non-exist
                next
            end
        next
    end
end
```

IPv6:

```bash
config router bgp
    config neighbor
        edit 2003::2:2:2:2
            config conditional-advertise6
                edit map-222
                    set condition-routemap map-222 map-282
                next
            end

            set route-reflector-client6 enable
        next
    end
end
```

### BGP Router Debug

```bash
diagnose ip router command show-vrf root show running router bgp
```

Useful for:

```text
Router ID
Neighbors
Address families
Advertisement maps
BGP configuration
```

---

# 27. BGP Next-Hop Tag Matching

```bash
config router bgp
    set tag-resolve-mode disable
end
```

### Modes

| Mode        | Behavior                                                    |
| ----------- | ----------------------------------------------------------- |
| `disable`   | Resolve next-hop using best match                           |
| `preferred` | Prefer next-hops with matching tags; fallback to best match |
| `merge`     | Merge best matching tags                                    |

### Recommended Use

```text
preferred → Failover
merge     → Load balancing
```

> In merge mode, tag matching can alter which next-hop tags are considered.

---

# 28. BGP Route Flap

Route flapping is resource intensive.

```text
Route
 ↓
UP
 ↓
DOWN
 ↓
UP
 ↓
DOWN
```

Use:

```text
Hold Timer
Dampening
Graceful Restart
BFD
```

where appropriate.

---

# 29. BGP Hold / Keepalive Timers

```bash
config router bgp
    config neighbor
        edit 1.2.3.4
            set holdtime-timer 60
            set keepalive-timer 60
        next
    end
end
```

---

# 30. BGP Route Dampening

Clear dampening state:

```bash
execute router clear bgp dampening 1.2.3.4/32

execute router clear bgp flap-stat 1.2.3.4/32
```

Configure:

```bash
config router bgp
    set dampening enable
    set dampening-max-suppress-time 60
    set dampening-reachability-half-life 15
    set dampening-reuse 750
    set dampening-route-map rmp-fail-damp
    set dampening-suppress 2000
    set dampening-unreachability-half-life 15
end
```

### Dampening Concepts

| Parameter          | Meaning                                         |
| ------------------ | ----------------------------------------------- |
| Penalty            | Value assigned when route flaps                 |
| Suppress Threshold | Route suppressed when penalty exceeds threshold |
| Reuse Threshold    | Route reused when penalty falls below threshold |
| Half-Life          | Time required for penalty to decay by half      |
| Max Suppress Time  | Maximum suppression duration                    |

### Example

Initial:

```text
Penalty = 0
```

Route flaps 3 times:

```text
Penalty = 1000 × 3
Penalty = 3000
```

Suppress threshold:

```text
2000
```

Therefore:

```text
3000 > 2000
→ Route suppressed
```

After 15 minutes:

```text
3000 / 2 = 1500
```

Still suppressed:

```text
1500 > 750
```

After another 15 minutes:

```text
1500 / 2 = 750
```

Reuse:

```text
750 ≤ 750
→ Route reused
```

If the penalty does not fall below the reuse threshold before the maximum suppression timer:

```text
Max suppress time reached
→ Route reused
```

---

# 31. BGP Graceful Restart

```bash
config router bgp
    set graceful-restart enable
    set graceful-restart-timer 120
    set graceful-stalepath-time 180
    set graceful-update-delay 120

    config neighbor
        edit 1.2.3.4
            set capability-graceful-restart enable
        next
    end
end
```

Restart BGP:

```bash
execute router restart
```

### Concept

```text
BGP Routing Engine Restart
        ↓
Keep forwarding stale routes temporarily
        ↓
BGP reconverges
        ↓
Routes refreshed
```

---

# 32. BFD — Bidirectional Forwarding Detection

### Notes

```text
Echo mode          → Not supported
Authentication     → Not supported
```

BFD provides faster failure detection than normal routing timers.

---

# 33. BFD for BGP

```bash
config router bgp
    set ebgp-multipath enable

    config neighbor
        edit 1.2.3.4
            set bfd enable
            set ebgp-enforce-multihop enable
        next
    end
end
```

---

# 34. Global BFD

Recommended approach:

```bash
config system settings
    set bfd enable
    set bfd-desired-min-tx 250
    set bfd-required-min-rx 250
    set bfd-detect-mult 3
    set bfd-dont-enforce-src-port disable
end
```

### BFD Calculation

```text
Detection Time
≈ Desired TX/RX Interval × Detect Multiplier

250 ms × 3
≈ 750 ms
```

> Check ISP ACL/firewall rules if BFD packets need to cross an external network.

---

# 35. BFD Neighbor

```bash
config router bfd
    config neighbor
        edit 24.24.24.1
            set interface port4
        next

        edit 12.12.12.1
            set interface port1
        next
    end
end
```

Useful for:

```text
Direct links
ISP links
ADVPN
IPsec scenarios
```

---

# 36. BFD Multihop Template

```bash
config router bfd
    config multihop-template
        edit 1
            set src 12.12.12.0/30
            set dst 24.24.24.0/30
        next

        edit 2
            set src 13.13.13.0/30
            set dst 23.23.23.0/30
        next
    end
end
```

> Useful for checking reachability across indirect paths.

---

# 37. BFD Per Interface

```bash
config system interface
    edit port8-9
        set bfd enable
        set bfd-desired-min-tx <ms>
        set bfd-required-min-rx <ms>
        set bfd-detect-mult <multiplier>
    next
end
```

> Interface-level BFD can inherit global settings.

---

# 38. BFD Troubleshooting

```bash
get router info bfd neighbor

get router info bfd requests
```

### State

```text
UP
= BFD session established

DOWN
= BFD session not established / unreachable
```

> For multihop BFD, verify that both sides support/configure the required BFD behavior.

---

# 39. Static Route + BFD

```bash
config router static
    edit 1
        set bfd enable
    next
end
```

Enable on interface:

```bash
config system interface
    edit port8
        set bfd enable
    next
end
```

BFD neighbor:

```bash
config router bfd
    config neighbor
        edit 192.168.15.1
            set interface port8
        next
    end
end
```

Global BFD:

```bash
config system settings
    set bfd enable
end
```

---

# 40. VRF

## VRF — Virtual Routing and Forwarding

```text
Routing segmentation
without creating separate VDOMs
```

### Capacity

```text
Up to 64 VRFs per VDOM
```

Enable visibility:

```text
System → Feature Visibility → Advanced Routing
```

### Important

Changing an interface VRF:

```text
Interface VRF Change
       ↓
RIB disruption
       ↓
Forwarding disruption
```

---

# 41. VRF Routing Table

```bash
get router info routing-table all
```

Example:

```text
Routing table for VRF=0
192.168.101.0/24 is directly connected, port3
192.168.254.0/24 is directly connected, port1

Routing table for VRF=2
12.12.12.0/30 is directly connected, port2
192.168.102.0/24 [20/0] via 12.12.12.2
    recursive is directly connected, port2

Routing table for VRF=3
13.13.13.0/30 is directly connected, port4
192.168.103.0/24 [20/0] via 13.13.13.2
    recursive is directly connected, port4
```

---

# 42. VRF Interconnection with VDOM Link

Create VDOM/VRF links:

```bash
config system vdom-link
    edit v-2-3-
    next
end
```

Allow overlapping subnets:

```bash
config system setting
    set allow-subnet-overlap enable
end
```

Example interfaces:

```text
v-2-3-0
= VRF 2 side

v-2-3-1
= VRF 3 side
```

Example subnet:

```text
10.10.10.0/30
```

---

# 43. VRF Route Leaking

### Step 1 — Create Prefix ACLs

Example:

```text
acl-102-103

permit 192.168.102.0/24 exact-match
permit 12.12.12.0/30 exact-match
permit 10.10.10.0/30 exact-match
```

```text
acl-103-102

permit 192.168.103.0/24 exact-match
permit 13.13.13.0/30 exact-match
permit 10.10.10.0/30 exact-match
```

### Step 2 — Route Maps

```text
rtm-102-103
    permit
    match-ip acl-102-103
```

```text
rtm-103-102
    permit
    match-ip acl-103-102
```

### Step 3 — BGP VRF Leak

```bash
config router bgp

    config vrf
        edit 2

            config leak-target
                edit 3
                    set route-map rtm-102-103
                    set interface v-2-3-0
                next
            end

        next

        edit 3

            config leak-target
                edit 2
                    set route-map rtm-103-102
                    set interface v-2-3-1
                next
            end

        next
    end

end
```

### Result

```text
VRF 2
  ↕
v-2-3-0 / v-2-3-1
  ↕
VRF 3
```

---

# 44. VRF Hub ↔ Spoke Route Leaking

If the hub/main VRF must communicate with VRF 2 and VRF 3:

```bash
config system vdom-link
    edit v-0-2-
    next

    edit v-0-3-
    next
end
```

Allow overlap:

```bash
config system setting
    set allow-subnet-overlap enable
end
```

Example:

```text
v-0-2-0
= VRF 0 side

v-0-2-1
= VRF 2 side
```

```text
v-0-3-0
= VRF 0 side

v-0-3-1
= VRF 3 side
```

---

# 45. VRF 0 ↔ VRF 2 Route Leak

### ACL

```text
acl-101-102

permit 192.168.101.0/24 exact-match
permit 12.12.12.0/30 exact-match
permit 9.9.9.0/30 exact-match
```

```text
acl-102-101

permit 192.168.102.0/24 exact-match
permit 12.12.12.0/30 exact-match
permit 9.9.9.0/30 exact-match
```

### Route Maps

```text
rtm-101-102
    permit
    match-ip acl-101-102
```

```text
rtm-102-101
    permit
    match-ip acl-102-101
```

### BGP Leak

```bash
config router bgp
    config vrf
        edit 0

            config leak-target
                edit 2
                    set route-map rtm-101-102
                    set interface v-0-2-0
                next
            end

        next

        edit 2

            config leak-target
                edit 0
                    set route-map rtm-102-101
                    set interface v-0-2-1
                next
            end

        next
    end
end
```

---

# 46. VRF 0 ↔ VRF 3 Route Leak

### ACL

```text
acl-101-103

permit 192.168.101.0/24 exact-match
permit 13.13.13.0/30 exact-match
permit 8.8.8.0/30 exact-match
```

```text
acl-103-101

permit 192.168.103.0/24 exact-match
permit 13.13.13.0/30 exact-match
permit 8.8.8.0/30 exact-match
```

### Route Maps

```text
rtm-101-103
    permit
    match-ip acl-101-103
```

```text
rtm-103-101
    permit
    match-ip acl-103-101
```

### BGP Leak

```bash
config router bgp
    config vrf
        edit 0

            config leak-target
                edit 3
                    set route-map rtm-101-103
                    set interface v-0-3-0
                next
            end

        next

        edit 3

            config leak-target
                edit 0
                    set route-map rtm-103-101
                    set interface v-0-3-1
                next
            end

        next
    end
end
```

After changes:

```bash
execute router clear bgp all soft
```

> ⚠️ Route leaking requires appropriate firewall policies/interfaces/zones as well.

---

# 47. VRF Route Table Verification

```bash
get router info routing-table all
```

Expected concept:

```text
VRF 0
 ├── Local network
 ├── VRF 2 routes
 └── VRF 3 routes

VRF 2
 ├── Local network
 ├── VRF 0 routes
 └── VRF 3 routes

VRF 3
 ├── Local network
 ├── VRF 0 routes
 └── VRF 2 routes
```

---

# 48. Blackhole Route in VRF

Example:

```text
Destination:
192.168.101.2

Interface:
blackhole

VRF-ID:
3
```

Use case:

```text
Block a specific route
inside a specific VRF
```

---

# 49. LLDP

LLDP provides neighboring/device information.

Useful for:

```text
Device Inventory
REST API
SNMP
Neighbor discovery
Device identification
```

### Per Interface

```bash
config system interface
    edit port1
        set lldp-reception enable
        set lldp-transmission enable
        set device-identification enable
    next
end
```

### Global

```bash
config system global
    set lldp-reception enable
    set lldp-transmission enable
end
```

### VDOM / VRF Context

```bash
config system settings
    set lldp-reception enable
    set lldp-transmission enable
end
```

---

# 50. LLDP Troubleshooting

### Device Information

```bash
diagnose user device list
```

### Neighbor Summary

```bash
diagnose lldprx neighbor summary
```

### Neighbor Details

```bash
diagnose lldprx neighbor details
```

### Clear Neighbors

```bash
diagnose lldprx neighbor clear
```

### Port Details

```bash
diagnose lldprx port details

diagnose lldprx port summary

diagnose lldprx port filter
```

### Port Neighbor Information

```bash
diagnose lldprx port neighbor summary

diagnose lldprx port neighbor details
```

### Netlink Interface Index

```bash
diagnose netlink interface list port2 | grep index
```

---

# 51. Quick Troubleshooting Flow

```text
                    ROUTING ISSUE
                         |
                         v
             get router info routing-table all
                         |
             +-----------+-----------+
             |                       |
         Route exists?           No Route
             |                       |
            Yes                      v
             |                Check RIB/database
             v
       Route Lookup
             |
             v
     get router info routing-table
          details <IP>
             |
             v
      Check Policy Route
             |
             v
       Check Route Cache
             |
             v
       Check FIB / Kernel
             |
             v
       Check Session State
             |
             v
       Check RPF / Asymmetry
             |
             v
        Check BFD/HA
             |
             v
        Check Firewall
           Policy
```

---

# 52. Quick Command Reference

| Task                  | Command                                          |
| --------------------- | ------------------------------------------------ |
| All routes            | `get router info routing-table all`              |
| Route details         | `get router info routing-table details`          |
| Route database        | `get router info routing-table database`         |
| Kernel/FIB            | `get router info kernel`                         |
| Route lookup          | `get router info routing-table details <IP>`     |
| Route cache           | `diagnose ip rtcache list`                       |
| IP addresses          | `diagnose ip address list`                       |
| RIP info              | `get router info rip`                            |
| RIP interfaces        | `get router info rip interfaces`                 |
| RIP database          | `get router info rip database`                   |
| OSPF info             | `get router ospf`                                |
| OSPF neighbors        | `get router info ospf neighbor`                  |
| OSPF neighbor details | `get router info ospf neighbor details`          |
| OSPF database         | `get router info ospf database brief`            |
| OSPF routes           | `get router info ospf route`                     |
| OSPF interfaces       | `get router info ospf interface`                 |
| OSPF status           | `get router info ospf status`                    |
| BGP paths             | `get router info bgp paths`                      |
| BGP neighbors         | `get router info bgp neighbors`                  |
| BGP summary           | `get router info bgp summary`                    |
| BGP networks          | `get router info bgp networks`                   |
| BFD neighbors         | `get router info bfd neighbor`                   |
| BFD requests          | `get router info bfd requests`                   |
| Session list          | `diagnose sys session list`                      |
| BGP soft reset        | `execute router clear bgp all soft`              |
| BGP packet capture    | `diagnose sniffer packet any 'tcp and port 179'` |
| LLDP neighbors        | `diagnose lldprx neighbor summary`               |
| LLDP details          | `diagnose lldprx neighbor details`               |
| LLDP port summary     | `diagnose lldprx port summary`                   |
| Device inventory      | `diagnose user device list`                      |

 
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

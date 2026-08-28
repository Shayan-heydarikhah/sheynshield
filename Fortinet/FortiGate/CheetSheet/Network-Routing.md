# FortiGate Routing & Forwarding Cheat Sheet

## Route Lookup & Core Diagnostics

**Administrative Distances (Default)**
* **Static Route:** 10
* **DHCP:** 5

**Routing Table & Kernel Commands**
```bash
get router info routing-table all       # Active and in-use routes
get router info routing-table details   # Detailed route lookups (e.g., details 4.4.4.4)
get router info routing-table database  # All received and learned routes
get router info kernel                  # FIB and routing daemon information
diagnose ip rtcache list                # Route lookup and cache checking
diagnose ip address list                # Show all assigned IP addresses
```

**Kernel Table Information (`get router info kernel`)**
* **Tables:** `254` (Unicast), `255` (Multicast)
* **vf:** VDOM name/index (0 if no VDOM)
* **prio:** Lower priority is the best path
* **pref:** Preferred next-hop

| Type Value | Description | Proto Value | Description |
| :--- | :--- | :--- | :--- |
| `0` | Unspecific | `0` | Unspecific |
| `1` | Unicast | `2` | Kernel |
| `2` | Local | `11` | ZebOS routing module (dynamic routing) |
| `3` | Broadcast | `14` | FortiOS |
| `4` | Anycast | `15` | HA (Learned from HA) |
| `5` | Multicast | `16` | Authentication based |
| `6` | Blackhole | `17` | HA1 (Learned from heartbeat links) |
| `7` | Unreachable | | |
| `8` | Prohibited (Blocked/TTL omitted) | | |

---

## Route Mechanics & Session Preservation

**Reverse Path Forwarding (RPF)**
Prevents IP spoofing. Feasible-path is default; strict mode checks all source/destination interfaces.
```bash
config system settings
  set strict-src-check enable 
end
```

**Asymmetric Routing & Session Preservation**
```bash
# Allow asymmetric routing (bypass 3-way handshake checks - has security risks)
config system settings
  set asymroute enable
end

# Preserve session route (useful for dynamic routing/HA topology changes)
config system interface
  edit port4
  set preserve-session-route enable 
end

# SNAT route change (Force session to require new route lookup on link fail)
config system global
  set snat-route-change enable
end
```

---

## ECMP & Link Redundancy

**ECMP Load Balancing Algorithms**
* **Source IP (Default):** Divided equally based on source IP.
* **Weighted:** Distributed based on assigned interface weights.
* **Usage (Spillover):** Uses an interface until bandwidth exceeds set thresholds, then spills to the next.
* **Source-Destination IP:** Traffic divided equally; identical source-to-destination sessions use the same path.
* **Volume (SD-WAN only):** Distributed based on packet count.

```bash
# Enable Usage-Based ECMP
config system settings
  set v4-ecmp-mode usage-base
  set ecmp-max-path 4
end
```

**Link Health Monitor (Failover)**
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
    set update-static-route enable     # Switch static routes on fail
    set update-cascade-interface enable # Shutdown cascade interfaces on fail
end
```

---

## Dynamic Routing: RIP & OSPF

### RIP Configurations
Ensure timers match across all devices. Split horizon can be verified via `get router info rip interfaces`.

```bash
config router rip
  set timeout-timer 30
  set update-timer 180
  set garbage-timer 30
end

# RIP Keychain Authentication
config router key-chain
  edit 1
    config key
      edit 1
        set key-string 123
        set accept-lifetime 12:20:00 8 2 2025 infinite
        set send-lifetime 12:20:00 8 2 2025 infinite
    end
end
```

### OSPF Configurations & Diagnostics

**Core Diagnostics**
```bash
get router ospf                          # Comprehensive OSPF setup
get router info routing-table ospf       # Global (VRF 0) received routes
get router info ospf neighbor details    # Neighbor states and statistics
get router info ospf database brief      # LSDB and LSA information
get router info ospf route               # Received and learned routes in RIB
get router info ospf status              # Uptime, VRF binds, SPF algorithm updates
```

**Graceful Restart (Topology Change)**
Provides non-interrupted forwarding between HA switch mechanisms during SPF algorithm runs.
```bash
config router ospf
  set router-id 31.1.1.1
  set restart-mode graceful-restart
  set restart-period 180
  set restart-on-topology-change enable
end
```

---

## Dynamic Routing: BGP

**Basic BGP & Multi-pathing**
```bash
config router bgp
  set as 65001
  set router-id 1.1.1.1
  set ebgp-multipath enable
  set ibgp-multipath enable
  set recursive-next-hop enable      # Resolves recursive lookups avoiding loops
  
  config neighbor
    edit 12.12.12.2
      set remote-as 65002
      set ebgp-enforce-multihop enable
      set ebgp-multihop-ttl 3
      set update-source lp-1         # Required if using loopback interfaces
      set soft-reconfiguration enable
  end
end
```

**BGP Route Flap Dampening**
Prevents resource-intensive route calculations from unstable links.

* **Penalty:** Value added to a route each flap.
* **Suppress Threshold:** Flap penalty value that triggers suppression.
* **Reuse Threshold:** Value where penalty must decay below to advertise the route again.
* **Half-life:** Time for a penalty to decay by 50%.

```bash
config router bgp
  set dampening enable
  set dampening-max-suppress-time 60
  set dampening-reachability-half-life 15
  set dampening-reuse 750
  set dampening-suppress 2000
end
```

**Bidirectional Forwarding Detection (BFD)**
Omits routes from the RIB faster than standard timers. Echo mode and authentication are unsupported.

```bash
# Global Settings
config system settings
  set bfd enable
  set bfd-desired-min-tx 250
  set bfd-required-min-rx 250
  set bfd-detect-mult 3
end

# Enable BFD on BGP Neighbor
config router bgp
  config neighbor
    edit 1.2.3.4
      set bfd enable
  end
end
```

---

## Virtual Routing and Forwarding (VRF)

Provides network segmentation without VDOMs. Supports up to 64 VRFs per VDOM. Modifying VRF attributes on interfaces disrupts forwarding.

**Configure VDOM Links for VRF Overlap**
```bash
config system vdom-link
  edit v-2-3-
end

config system settings
  set allow-subnet-overlap enable
end
```

**Route Leaking Between VRFs (via BGP)**
```bash
config router bgp
  config vrf
    edit 2
      config leak-target
        edit 3
          set route-map rtm-102-103
          set interface v-2-3-0
      end
    next
    edit 3
      config leak-target
        edit 2
          set route-map rtm-103-102
          set interface v-2-3-1
      end
  end
end
```

**Blackhole Routes per VRF**
```bash
config router static
  edit 1
    set dst 192.168.101.2 255.255.255.255
    set blackhole enable
    set vrf 3
end
```

---

## LLDP (Link Layer Discovery Protocol)

Sends client information to the FortiGate during scanning (essential for SNMP/RESTAPI).

**Configuration Tiers**
```bash
# Global
config system global
  set lldp-reception enable
  set lldp-transmit enable
end

# Interface Level
config system interface
  edit port1
    set lldp-reception enable
    set lldp-transmit enable
    set device-identification enable
end
```

**Diagnostics**
```bash
diagnose user device list
diagnose lldprx neighbor summary
diagnose lldprx neighbor details
diagnose lldprx port summary
```

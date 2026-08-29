# 🔥 FortiGate Routing Checklist | Static Route, ECMP, OSPF, BGP, BFD, VRF

> **SheynShield Engineering Secure Networks**
>
> FortiGate Routing Operational Checklist for NSE4 / NSE7 Engineers  
>
> Version: FortiOS 7.x  
>
> Category:
> - Routing
> - High Availability
> - Dynamic Routing
> - Network Troubleshooting

---

# 📌 Table of Contents

- [Static Route Checklist](#static-route-checklist)
- [Route Lookup Checklist](#route-lookup-checklist)
- [RPF Checklist](#rpf-checklist)
- [Asymmetric Routing Checklist](#asymmetric-routing-checklist)
- [Policy Route Checklist](#policy-route-checklist)
- [ECMP Checklist](#ecmp-checklist)
- [SD-WAN Failover Checklist](#sd-wan-failover-checklist)
- [RIP Checklist](#rip-checklist)
- [OSPF Checklist](#ospf-checklist)
- [BGP Checklist](#bgp-checklist)
- [BFD Checklist](#bfd-checklist)
- [VRF Checklist](#vrf-checklist)
- [LLDP Checklist](#lldp-checklist)
- [Routing Troubleshooting Flow](#routing-troubleshooting-flow)


---

# 1. Static Route Checklist

## Deployment Checklist

- [ ] Verify destination network

Example:

```

192.168.10.0/24

````

- [ ] Verify next-hop reachability

```bash
execute ping <gateway-ip>
````

* [ ] Confirm outgoing interface

```bash
show system interface
```

* [ ] Configure administrative distance

Default:

| Route Type | Distance |
| ---------- | -------: |
| Static     |       10 |
| DHCP       |        5 |

---

## Routing Table Validation

Check active routes:

```bash
get router info routing-table all
```

Check detailed route information:

```bash
get router info routing-table details
```

Check all learned routes:

```bash
get router info routing-table database
```

---

# 2. Kernel Routing Table Checklist

Verify FIB information:

```bash
get router info kernel
```

Validate:

* [ ] TAB value

```
254 = Unicast
255 = Multicast
```

* [ ] VRF / VDOM context

```
VF = VDOM index
```

---

# 3. Route Type Validation Checklist

| Value | Type        | Verify |
| ----- | ----------- | ------ |
| 0     | Unspecified | ⬜      |
| 1     | Unicast     | ⬜      |
| 2     | Local       | ⬜      |
| 3     | Broadcast   | ⬜      |
| 4     | Anycast     | ⬜      |
| 5     | Multicast   | ⬜      |
| 6     | Blackhole   | ⬜      |
| 7     | Unreachable | ⬜      |
| 8     | Prohibited  | ⬜      |

---

# 4. Route Lookup Checklist

## Troubleshooting Order

Always check routing decision in this order:

```
1. Policy Route
        |
        v
2. Route Cache
        |
        v
3. FIB
        |
        v
4. Drop
```

---

## Route Cache Validation

```bash
diagnose ip rtcache list
```

Configure cache size:

```bash
config system global
    set max-route-cache-size 100
end
```

---

## Specific Destination Lookup

```bash
get router info routing-table details <destination-ip>
```

Example:

```bash
get router info routing-table details 4.4.4.4
```

---

# 5. Reverse Path Forwarding (RPF) Checklist

## Enable Strict Source Check

```bash
config system setting
    set strict-src-check enable
end
```

---

## Validate RPF Mode

### Feasible Path RPF

* [ ] Source exists in routing table
* [ ] Any valid interface/path accepted
* [ ] Recommended for asymmetric environments

### Strict RPF

* [ ] Source interface must match routing path
* [ ] Validate asymmetric designs before enabling

---

## Disable Source Check Per Interface

Use only when asymmetric routing exists:

```bash
config system interface
    edit port8
        set src-check disable
    next
end
```

---

# 6. Asymmetric Routing Checklist

Before enabling:

* [ ] Confirm asymmetric traffic design
* [ ] Validate security inspection impact
* [ ] Confirm session tracking requirements

Enable:

```bash
config system setting
    set asymroute enable
end
```

⚠️ Impact:

* Reduced session validation
* Possible security inspection limitations

---

# 7. Preserve Session Route Checklist

## Without SNAT

Use when:

* [ ] Dynamic routing changes exist
* [ ] HA failover occurs
* [ ] Existing sessions must maintain path

Configuration:

```bash
config system interface
    edit port4
        set preserve-session-route enable
    next
end
```

---

# 8. SNAT Route Change Checklist

Enable:

```bash
config system global
    set snat-route-change enable
end
```

Expected behavior:

```
Route Change
      |
      v
New Route Lookup
      |
      v
New Session Path
```

---

# Session Troubleshooting

Filter:

```bash
diagnose sys session filter src <ip>
```

Check:

```bash
diagnose sys session list
```

Validate:

* [ ] Source
* [ ] Destination
* [ ] Policy ID
* [ ] NAT
* [ ] Route information

---

# 9. Policy Route Checklist

Before creating policy route:

* [ ] Normal routing exists
* [ ] Default route available
* [ ] Traffic matching criteria defined

Validate:

```bash
diagnose firewall proute list
```

---

# 10. ECMP Checklist

Requirements:

* [ ] Same destination prefix
* [ ] Same administrative distance
* [ ] Same metric/cost

Supported methods:

| Method           | Usage                      |
| ---------------- | -------------------------- |
| source-ip-based  | Same source uses same path |
| weighted-based   | Traffic based on weight    |
| usage-based      | Bandwidth utilization      |
| src-dst-ip-based | Same flow uses same path   |
| volume-based     | Packet volume distribution |

---

## Global ECMP

```bash
config system setting
    set v4-ecmp-mode usage-base
    set ecmp-max-path 4
end
```

Verify:

```bash
get router info routing-table all
```

---

# 11. SD-WAN Load Sharing Checklist

Recommended for:

* [ ] Dual ISP
* [ ] Link balancing
* [ ] Automatic failover

Configuration:

```bash
config system sdwan
    set status enable
    set load-balance-mode usage-based
end
```

Validate:

* [ ] SLA checks
* [ ] Health monitoring
* [ ] Failover behavior



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

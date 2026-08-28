# FortiGate SD-WAN — Speed Test, Packet Duplication & Large-Scale Deployment Cheat Sheet

## 1. SD-WAN Speed Test

### Concept

SD-WAN Speed Test can be used from the **SD-WAN member/VPN panel** to measure link performance.

It can behave similarly to a **site-to-site VPN workflow** and can automatically create:

* IPsec VPN components
* SLA / health-check objects
* SD-WAN rules
* Link-quality measurements

> [!IMPORTANT]
> Speed Test uses **licensed servers / cloud SLA infrastructure** for link-health and performance measurement.

---

## 2. Enable Local FortiOS Speed-Test Server

If the FortiGate itself should operate as a Speed Test server:

```cli
config system global
    set speedtest-server enable
end
```

> [!NOTE]
> The Speed Test server feature must be enabled on the FortiGate that will act as the server.

---

## 3. Speed-Test Schedule

Enable a dynamic server for a Speed Test schedule:

```cli
config system speed-test-schedule
    edit hs-ad-1
        set dynamic-server enable
    next
end
```

Or configure it for a specific interface:

```cli
config system speed-test-schedule
    edit port1
        set dynamic-server enable
    next
end
```

> [!IMPORTANT]
> Speed Test administrative access must be enabled on the ISP/physical interface used for the test.

---

## 4. Custom Speed-Test Server

If `dynamic-server` is disabled, a custom server must be specified.

Typical options include:

* Custom Speed Test server
* `iperf3`
* TCP port `5201`

### iperf3 Server

Run on the test server:

```powershell
.\iperf3.exe -s -p 5201
```

The FortiGate can then perform the test against this server.

---

## 5. Speed-Test Routing Behavior

```cli
config system sdwan
    set speedtest-bypass-routing enable
end
```

### `speedtest-bypass-routing`

| Setting   | Behavior                                                                                    |
| --------- | ------------------------------------------------------------------------------------------- |
| `enable`  | Speed Test follows the manually configured test servers and bypasses normal dynamic routing |
| `disable` | VPN/overlay routing is used by default                                                      |

> [!TIP]
> When troubleshooting Speed Test routing, verify whether the test traffic is using the **underlay** or the **VPN overlay**.

---

# 6. SD-WAN Neighbor Speed-Test Mode

Example:

```cli
config system sdwan
    config neighbor
        edit 12.23.34.1
            set mode speedtest
        next
    end
end
```

This allows the SD-WAN neighbor relationship to participate in Speed Test behavior.

---

# 7. Manually Run Speed Test

### Blocking / Upload Speed Test

```cli
execute speed-test-dynamic hs-ad-1 all n
```

### Non-Blocking Speed Test

```cli
diagnose netlink interface speed-test hs-ad-23
```

---

# 8. Speed-Test Troubleshooting

### Speed-Test daemon

```cli
diagnose debug application speedtest 100
diagnose debug application speedtestd 100
diagnose debug enable
```

### FortiCron / Scheduler

`forticron` is responsible for scheduled jobs on FortiOS.

```cli
diagnose test application forticron 9
diagnose test application forticron 10
diagnose test application forticron 11
diagnose test application forticron 12
diagnose test application forticron 99
```

---

# 9. SD-WAN Speed Test — Hub & Spoke Workflow

Typical large-scale design:

```text
                    HUB
                     |
          +----------+----------+
          |                     |
       Spoke-1               Spoke-2
          |                     |
       Underlay              Underlay
```

### Workflow

1. A recurring Speed Test is configured on the **hub**.
2. The test runs over the **underlay interfaces**.
3. Spokes operate as Speed Test servers.
4. Spokes allow Speed Test traffic on their underlay interfaces.
5. Spokes establish BGP peering with the hub over the VPN overlay.
6. Spokes advertise their loopback/network toward the hub.
7. During the Speed Test:

   * VPN overlay routing can be bypassed.
   * Route maps can filter advertised routes.
   * The hub is forced to reach the spoke through the underlay.
8. After the test:

   * Measured egress bandwidth is dynamically applied to the VPN tunnel.
   * The result is cached.
   * Cached values can be reused after tunnel disconnect/reconnect.

### Routing Logic

```text
Normal operation:

Hub
 |
VPN Overlay
 |
Spoke Network
```

During Speed Test:

```text
Hub
 |
Underlay
 |
Spoke Speed-Test Server
```

> [!IMPORTANT]
> Route-map filtering is important because the Speed Test should not accidentally use the VPN overlay when the goal is to measure the **underlay path**.

---

# 10. Speed Test + Route Maps

During Speed Test:

```text
Hub
 |
 |---- Underlay ----> Spoke
 |
 VPN Overlay
     X
```

The spoke route map can temporarily prevent route advertisement so the hub cannot reach the test destination through the VPN overlay.

After Speed Test:

```text
Spoke route-map
        |
        +--> Advertise spoke network
        |
        +--> Hub restores normal VPN routing
```

---

# 11. Speed-Test Result Handling

After the test completes:

```text
Speed Test
    |
    v
Measured bandwidth
    |
    v
SD-WAN / VPN tunnel
    |
    v
Dynamic bandwidth value
    |
    v
Cached result
```

The measured egress bandwidth can be dynamically applied to the VPN tunnel.

---

# 12. SD-WAN Packet Duplication

Packet duplication creates copies of packets and sends them through multiple SD-WAN members.

```text
                 +--> WAN-1
Client --> SDWAN-+
                 +--> WAN-2
```

The receiving side can perform **packet de-duplication** and discard duplicate packets.

### Main goals

* Improve reliability
* Reduce packet loss impact
* Provide fast recovery
* Protect sensitive traffic
* Improve behavior during unstable WAN links

> [!NOTE]
> Packet duplication can increase bandwidth consumption because the same packet may be transmitted over multiple links.

---

# 13. SD-WAN Duplication Configuration

```cli
config system sdwan
    config duplication
        edit 1
            set srcaddr all
            set dstaddr all
            set srcintf port1
            set dstintf sdwan-1
            set service ALL

            set packet-duplication force
            set packet-de-duplication enable
            set duplication-max-num 2
        next
    end
end
```

---

# 14. Packet-Duplication Modes

| Mode        | Behavior                                             |
| ----------- | ---------------------------------------------------- |
| `disable`   | Packet duplication disabled                          |
| `force`     | Duplicate packets across all eligible SD-WAN members |
| `on-demand` | Duplicate packets only when link quality requires it |

### `disable`

```cli
set packet-duplication disable
```

Default behavior.

No packet duplication is performed.

---

### `force`

```cli
set packet-duplication force
```

Packets are duplicated across all members of the SD-WAN zone.

```text
                +--> Link-1
Packet ---------+
                +--> Link-2
                |
                +--> Link-3
```

If a member's health check fails, that member is removed from the duplication set.

---

### `on-demand`

```cli
set packet-duplication on-demand
```

Duplication occurs only when link quality becomes insufficient.

```text
Good SLA:

Packet ---> Link-1

Poor SLA:

Packet ---> Link-1
       \--> Link-2
```

> [!TIP]
> `on-demand` is generally more bandwidth-efficient than permanently duplicating every packet.

---

# 15. Packet De-Duplication

```cli
set packet-de-duplication enable
```

Default:

```text
disable
```

When enabled, the FortiGate detects duplicate packets and discards the copies.

```text
             +--> Packet A ----+
Sender ------+                 |
             +--> Packet A ----+--> Receiver
                               |
                         Keep one packet
                         Drop duplicate
```

---

# 16. Maximum Duplication Members

```cli
set duplication-max-num 2
```

Controls the maximum number of SD-WAN members that can forward duplicate packets.

Example:

```cli
set duplication-max-num 4
```

With four WAN links:

```text
                 +--> WAN-1
                 +--> WAN-2
Packet ----------+--> WAN-3
                 +--> WAN-4
```

---

# 17. Duplication with SD-WAN Service Rules

Packet duplication can also be associated with a specific SD-WAN service:

```cli
config system sdwan
    config duplication
        edit 1
            set service-id 1
            set packet-duplication force
        next
    end
end
```

This allows duplication to be applied to traffic matching a particular SD-WAN service rule.

---

# 18. Duplication + SLA

### Force

```text
SLA healthy
     |
     +--> Duplicate traffic
     |       |
     |       +--> Member-1
     |       +--> Member-2
     |
     +--> Receiver de-duplicates
```

### On-Demand

```text
SLA healthy
     |
     +--> Normal forwarding
     
SLA degraded
     |
     +--> Duplicate packets
             |
             +--> Member-1
             +--> Member-2
```

---

# 19. `sla-match-service`

When using `on-demand`, packet duplication behavior can depend on SLA matching.

### Without SLA matching

If all relevant SLAs exceed their thresholds:

```text
SLA Map = 0
     |
     v
Packet duplication triggered
```

If SLA conditions remain within thresholds:

```text
SLA Map != 0
     |
     v
No duplication
```

### With SLA matching

Only the SLA health checks and targets associated with the selected SD-WAN service rule are considered.

```text
SD-WAN Service
      |
      v
Service SLA
      |
      v
Threshold exceeded?
      |
   +--+--+
   |     |
  YES    NO
   |     |
 Dup     Normal
```

---

# 20. Packet Duplication with ADVPN

Packet duplication can be useful in **Hub-and-Spoke / ADVPN** scenarios.

Example:

```text
                    HUB
                 /       \
              VPN-1     VPN-2
               /           \
          Spoke-1         Spoke-2
```

During dynamic tunnel establishment or unstable paths, duplication/de-duplication can reduce the impact of packet loss.

> [!NOTE]
> ADVPN can subsequently establish direct spoke-to-spoke tunnels, reducing the need to forward traffic through the hub.

---

# 21. SD-WAN Speed Test vs Packet Duplication

| Feature                      | Speed Test               | Packet Duplication     |
| ---------------------------- | ------------------------ | ---------------------- |
| Main purpose                 | Measure link performance | Improve reliability    |
| Generates additional traffic | Yes                      | Yes                    |
| Uses SLA information         | Yes                      | Yes                    |
| Measures bandwidth           | Yes                      | No                     |
| Sends duplicate packets      | No                       | Yes                    |
| Can consume extra bandwidth  | Yes                      | Yes                    |
| Useful for unstable links    | Measurement              | Resiliency             |
| Common use                   | Bandwidth estimation     | Loss-sensitive traffic |

---

# 22. Large-Scale Deployment

Large-scale Fortinet deployments can combine:

```text
                 SD-WAN
                    |
        +-----------+-----------+
        |           |           |
      ADVPN        BGP        SLA
        |           |           |
        +-----------+-----------+
                    |
             Speed Test
                    |
             Link Measurement
```

Relevant technologies include:

* SD-WAN
* ADVPN
* Dynamic routing
* BGP
* SLA monitoring
* Speed Test
* Packet duplication
* Packet de-duplication
* FEC
* Traffic steering

---

# 23. SD-WAN + ADVPN Mode Configuration

For spokes where Mode Config is required:

```cli
config vpn ipsec phase1-interface
    edit link-1
        set mode-cfg enable
        set mode-cfg-allow-client-selector enable
    next
end
```

### Purpose

```text
Spoke
  |
  +--> Mode Config
          |
          +--> Receive assigned configuration
          |
          +--> Use client selectors
```

> [!TIP]
> This can be useful when dynamically assigning or changing Mode Config parameters on ADVPN spokes.

---

# 24. Practical Troubleshooting Flow

### Speed Test

```text
1. Check Speed-Test server
        |
2. Check admin access
        |
3. Check schedule
        |
4. Check dynamic/custom server
        |
5. Check routing
        |
6. Check underlay path
        |
7. Run manual test
        |
8. Check speedtest debug
        |
9. Verify measured bandwidth
```

### Commands

```cli
diagnose debug application speedtest 100
diagnose debug application speedtestd 100
diagnose test application forticron 9
diagnose test application forticron 10
diagnose test application forticron 11
diagnose test application forticron 12
diagnose test application forticron 99
diagnose debug enable
```

---

# 25. Practical Packet-Duplication Troubleshooting

```text
1. Verify SD-WAN members
        |
2. Verify SLA state
        |
3. Check duplication mode
        |
4. Check duplication-max-num
        |
5. Check packet-de-duplication
        |
6. Verify SD-WAN service matching
        |
7. Check packet captures
        |
8. Confirm duplicate packets
        |
9. Confirm receiver de-duplication
```

---

# 26. Quick Reference

| Task                                 | Command / Setting                           |
| ------------------------------------ | ------------------------------------------- |
| Enable local Speed Test server       | `set speedtest-server enable`               |
| Enable dynamic Speed Test server     | `set dynamic-server enable`                 |
| Bypass SD-WAN routing for Speed Test | `set speedtest-bypass-routing enable`       |
| Speed-Test neighbor mode             | `set mode speedtest`                        |
| Run dynamic Speed Test               | `execute speed-test-dynamic ...`            |
| Run non-blocking Speed Test          | `diagnose netlink interface speed-test ...` |
| Debug Speed Test                     | `diagnose debug application speedtest 100`  |
| Debug Speed-Test daemon              | `diagnose debug application speedtestd 100` |
| Disable duplication                  | `set packet-duplication disable`            |
| Force duplication                    | `set packet-duplication force`              |
| Conditional duplication              | `set packet-duplication on-demand`          |
| Enable de-duplication                | `set packet-de-duplication enable`          |
| Maximum duplicate members            | `set duplication-max-num <N>`               |
| Enable ADVPN Mode Config             | `set mode-cfg enable`                       |
| Allow client selector                | `set mode-cfg-allow-client-selector enable` |

---

# 27. Design Rules — Quick Memory

> **Speed Test = Measure the WAN**

> **SLA = Decide whether the WAN is healthy**

> **SD-WAN Rule = Decide where traffic should go**

> **Packet Duplication = Send copies over multiple paths**

> **Packet De-Duplication = Keep one copy and drop duplicates**

> **ADVPN = Build dynamic spoke-to-spoke tunnels**

> **BGP = Advertise and learn routes**

> **Route Map = Control route advertisement/selection**

> **Mode Config = Dynamically provide VPN client parameters**

> **Underlay = Physical/IP transport path**

> **Overlay = VPN tunnel built over the underlay**

# 🔀 FortiGate SD-WAN Strategies

> **SD-WAN Strategies** determine how FortiGate selects SD-WAN members based on interface priority, SLA results, bandwidth, and routing conditions.

---

## 📌 Strategy Overview

In **SD-WAN Service / Rule**, the outgoing interface selection can use different strategies:

| Strategy             | Main Purpose                          |      SLA | Automatic Selection |
| -------------------- | ------------------------------------- | -------: | ------------------: |
| `Manual`             | Administrator-controlled path         |        ❌ |                   ❌ |
| `Best Quality`       | Select best SLA quality               |        ✅ |                   ✅ |
| `Minimum SLA`        | Select links meeting SLA requirements |        ✅ |                   ✅ |
| `Maximize Bandwidth` | Use available bandwidth efficiently   | Optional |                   ✅ |
| `Load Balance`       | Distribute traffic across members     | Optional |                   ✅ |

---

# 🛠️ Quality-Based Strategy

Example:

```bash
config system sdwan
    config service
        edit 1
            set quality-link 3
        end
    end
end
```

### Selection Logic

The `quality-link` concept can be understood as several levels of path selection:

### 1. Interface Priority

```text
Interface priority
        ↓
Select preferred SD-WAN member
```

* Uses interface/member priority.
* No SLA-based decision is required.

---

### 2. SLA + Route

```text
SLA
 ↓
Route availability
 ↓
Select member
```

* Uses SLA and routing information.
* No fallback lookup mechanism.
* If all suitable SD-WAN members become unavailable, traffic can be dropped.

---

### 3. SLA + Fallback + Link Monitoring

```text
SLA
 ↓
Link monitoring
 ↓
Fallback
 ↓
Another available member
```

* Uses SLA information.
* Uses link monitoring.
* Provides fallback behavior when the preferred member fails.

---

# 🎛️ Manual Strategy

Manual mode provides **administrator-controlled path selection**.

```bash
config system sdwan
    config service
        edit 1
            set mode manual
            set hold-down-time 60
        end
    end
end
```

### `hold-down-time`

```text
Primary link fails
       ↓
Failover
       ↓
Secondary link
       ↓
Primary becomes available
       ↓
Wait 60 seconds
       ↓
Switch back to primary
```

> `hold-down-time` prevents immediate switching back to the primary member after recovery.

### Characteristics

* No automatic health-based selection.
* Administrator determines the preferred path.
* Similar in concept to **policy-based routing**.
* Can work with:

  * Application-aware routing
  * BGP route tags
  * Interface/member priority

---

# ⚠️ Interface Priority vs Zone Priority

When both **Preferred Interface** and **Zone** are configured:

```text
Preferred Interface
        ↓
Zone members
```

The explicitly selected interface has priority over the zone selection.

### Example

```text
Preferred Interface:
    link-2

Zone:
    link-1
    link-2
```

Result:

```text
link-2
  ↓
preferred first
```

> Inside an interface list, the member ordering/priority also affects path selection.

---

# 📊 Bandwidth Measurements

SD-WAN can use:

* `inband` → downstream bandwidth
* `outband` → upstream bandwidth
* `biband` → both directions

### Recommended

Configure estimated bandwidth on ISP interfaces:

```bash
config system interface
    edit <interface>
        set estimated-upstream-bandwidth 100
        set estimated-downstream-bandwidth 200
    end
end
```

> Values are in **Kbps**.

### Example

```text
ISP-1
├── Upstream:   100 Kbps
└── Downstream: 200 Kbps
```

> 💡 A Speed Test can also be used to obtain more realistic bandwidth measurements.

---

# 🟢 Minimum SLA Meet

### Goal

Select a link only when it satisfies the required SLA conditions.

Example topology:

```text
              SD-WAN Zone
             /     |     \
           ISP-1  ISP-2  ISP-3
```

If ISP-1 fails:

```text
ISP-1
  ↓
SLA failure
  ↓
Check ISP-2
  ↓
Check ISP-3
  ↓
Select suitable member
```

The decision is not necessarily:

```text
ISP-1 → ISP-2
```

Instead:

```text
ISP-1 fails
     ↓
Evaluate other members
     ↓
Compare SLA / reachability
     ↓
Choose suitable path
```

This can prevent blindly selecting another member that is also experiencing SLA problems.

---

# 🚀 Maximize Bandwidth

Global SD-WAN load-balancing mode:

```bash
config system sdwan
    set load-balance-mode source-ip-based
end
```

### Service

```bash
config system sdwan
    config service
        edit 1
            set mode load-balance
        end
    end
end
```

---

# 🔀 SD-WAN Hash Modes

When using:

```text
mode = load-balance
```

different hash/selection mechanisms can be used.

| Hash Mode              | Behavior                                       |
| ---------------------- | ---------------------------------------------- |
| `round-robin`          | Distribute traffic equally/cyclically          |
| `source-ip-based`      | Same source IP → same member                   |
| `source-dest-ip-based` | Same source + destination → same member        |
| `inbandwidth`          | Prefer member with more downstream capacity    |
| `outbandwidth`         | Prefer member with more upstream capacity      |
| `bibandwidth`          | Consider both upstream and downstream capacity |

---

## 🔄 Round-Robin

```text
Traffic
  ↓
ISP-1
ISP-2
ISP-3
ISP-1
ISP-2
ISP-3
...
```

* Traffic is distributed in circular order.
* Equal distribution.
* GUI commonly exposes this as the default load-balancing option.

---

## 🧑 Source-IP-Based

```text
Client-A ─────→ ISP-1
Client-B ─────→ ISP-2
Client-C ─────→ ISP-3
```

The source IP is used by the hashing mechanism.

```text
Same Source IP
      ↓
Same SD-WAN member
```

Useful when path consistency is important.

---

## 🎯 Source-Destination-IP-Based

```text
Source + Destination
        ↓
     Hashing
        ↓
SD-WAN member
```

Example:

```text
10.10.10.10 → 8.8.8.8
        ↓
      ISP-1

10.10.10.10 → 1.1.1.1
        ↓
      ISP-2
```

---

# 📥 Inbandwidth

Select the member with the most available **incoming/downstream bandwidth**.

```text
                FortiGate
              ↙    ↓    ↘
           ISP-1 ISP-2 ISP-3
            20    80    40 Mbps
                   ↑
                selected
```

---

# 📤 Outbandwidth

Select the member with the most available **outgoing/upstream bandwidth**.

```text
FortiGate
    ↓
Compare upstream capacity
    ↓
Select member with most available bandwidth
```

---

# ↕️ Bibandwidth

Considers both directions:

```text
Downstream
    +
Upstream
    ↓
Available bandwidth
    ↓
SD-WAN member selection
```

---

## ⚠️ Maximize Bandwidth Notes

> **ADVPN is not supported with this strategy.**

For accurate bandwidth-based decisions:

```text
Speed Test
    +
Estimated Bandwidth
    ↓
Better SD-WAN decision
```

> ⚠️ With maximum-bandwidth strategies, gateway-related commands/behavior may not be available in the expected way. A policy route may be required for specific gateway steering requirements.

---

# 🧭 Policy Route Troubleshooting

Use:

```bash
diagnose ip proute match 3.1.1.34 70:4c:a5:86:de:56 port3 22 6
```

Useful for checking which policy route/path would match specific traffic.

---

# 🎯 QoS / Traffic Shaping Test

## Traffic Shaper

Example:

```text
Name:
    shape-1

Type:
    Shared Shaper

Priority:
    High

Maximum bandwidth:
    200 Kbps
```

---

## Traffic Shaping Policy

```text
Source:
    all

Destination:
    192.168.102.0/24

Service:
    all

Outgoing Interface:
    sdwan-zone-back

Shared Shaper:
    shape-1

Reverse Shaper:
    shape-1
```

Also ensure the required firewall policies allow the traffic.

---

# 🔍 QoS Troubleshooting

### IP Rope

```bash
diagnose firewall iprope list 10015
```

### Sessions

```bash
diagnose sys session list
```

Look for:

```text
shaper
```

### Traffic Shaper Statistics

```bash
diagnose firewall shaper traffic-shaper list
```

### SDN Status

```bash
diagnose sys sdn status
```

### SDN Debug

```bash
diagnose debug application azd -1
```

### SD-WAN Service

```bash
diagnose sys sdwan service
```

---

# 🎯 SD-WAN Steering

SD-WAN can steer traffic based on **application identification**.

```bash
diagnose sys sdwan internet-service-app-ctrl-list 10
```

This helps identify which interface/member is selected for an application.

### General Flow

```text
Client
  ↓
Firewall Policy
  ↓
Application / IPS Identification
  ↓
SD-WAN Rule
  ↓
Strategy
  ↓
SD-WAN Member
```

---

## 🛡️ Application Control + IPS

Use:

* Application Control
* IPS signatures
* Firewall policies

for:

```text
Accept / Deny
Application identification
Access control
```

Then use the SD-WAN rule/service for:

```text
Path selection
      ↓
Preferred SD-WAN member
```

> Application/security policy decides **whether traffic is allowed**; SD-WAN steering decides **which path carries it**.

---

# 🧠 Strategy Selection — Quick Guide

| Requirement                    | Recommended Approach                 |
| ------------------------------ | ------------------------------------ |
| Explicit administrator path    | `Manual`                             |
| Best latency/jitter/loss       | `Best Quality`                       |
| Only use links meeting SLA     | `Minimum SLA`                        |
| Use available bandwidth        | `Maximize Bandwidth`                 |
| Equal traffic distribution     | `Round-Robin`                        |
| Path consistency per client    | `Source-IP-Based`                    |
| Path consistency per flow pair | `Source-Destination-IP-Based`        |
| Prefer downstream capacity     | `Inbandwidth`                        |
| Prefer upstream capacity       | `Outbandwidth`                       |
| Consider both directions       | `Bibandwidth`                        |
| Application-specific path      | SD-WAN Service + Application Control |
| QoS enforcement                | Traffic Shaper + SD-WAN              |

---

# 🧩 SD-WAN Decision Model

```text
                    Traffic
                       │
                       ▼
              ┌─────────────────┐
              │ Firewall Policy │
              └────────┬────────┘
                       │
                       ▼
              Application / IPS
                       │
                       ▼
              ┌─────────────────┐
              │   SD-WAN Rule   │
              └────────┬────────┘
                       │
              ┌────────▼────────┐
              │    Strategy     │
              └────────┬────────┘
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
      Manual       SLA-based     Bandwidth-based
        │              │              │
        ▼              ▼              ▼
     Priority       Quality       in/out/biband
                       │
                       ▼
                Selected Member
                       │
                       ▼
                    Internet
```

---

# ⚡ Quick CLI Reference

```bash
# Global SD-WAN load balancing
config system sdwan
    set load-balance-mode source-ip-based
end
```

```bash
# SD-WAN service
config system sdwan
    config service
        edit 1
            set mode manual
            set hold-down-time 60
        end
    end
end
```

```bash
# Estimated ISP bandwidth
config system interface
    edit <interface>
        set estimated-upstream-bandwidth 100
        set estimated-downstream-bandwidth 200
    end
end
```

```bash
# Check policy route decision
diagnose ip proute match <destination> <mac> <interface> <port> <protocol>
```

```bash
# SD-WAN service status
diagnose sys sdwan service
```

```bash
# Application steering information
diagnose sys sdwan internet-service-app-ctrl-list 10
```

---

# 🚨 Key Troubleshooting Checklist

```text
1. Is the SD-WAN member UP?
        ↓
2. Is the gateway reachable?
        ↓
3. Is the SLA healthy?
        ↓
4. Which strategy is configured?
        ↓
5. What is the member priority?
        ↓
6. Is a zone being used?
        ↓
7. Is a preferred interface configured?
        ↓
8. Which load-balance/hash mode is active?
        ↓
9. Is bandwidth estimation correct?
        ↓
10. Is application steering changing the path?
        ↓
11. Is a firewall policy allowing the traffic?
        ↓
12. Is QoS/traffic shaping affecting the flow?
```

---

# 📌 Important Notes

> **SLA is the measurement mechanism; the SD-WAN strategy determines how those measurements are used.**

> **Manual mode does not provide automatic SLA-based path selection.**

> **For bandwidth-aware decisions, configure accurate estimated bandwidth or use Speed Test.**

> **For application-aware steering, identify the application through security inspection and then apply the SD-WAN steering rule.**

> **Do not confuse global `load-balance-mode` with the SD-WAN Service `mode`.** They control different parts of SD-WAN traffic distribution.

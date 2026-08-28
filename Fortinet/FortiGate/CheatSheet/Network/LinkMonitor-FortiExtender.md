# 🔗 FortiGate Link Monitor & FortiExtender 

> **FortiOS 7.2.0**
> Practical reference for **Link Monitor, Health Checks, SD-WAN, Failover, FortiExtender & LTE**

---

## 🔗 Link Monitor

### 🎯 What Is Link Monitor?

**Link Monitor** performs health checks against a remote server/destination to determine whether a path is reachable and healthy.

It can be used to:

* Monitor link/path availability
* Detect remote server reachability
* Trigger link-down / link-up states
* Remove failed routes from the routing table
* Update static routes
* Update policy routes
* Cascade failures to other interfaces
* Support redundant-link scenarios
* Provide path-health information for **SD-WAN**

### 🧪 Supported Health-Check Protocols

| Protocol      | Purpose                             |
| ------------- | ----------------------------------- |
| `PING / ICMP` | Basic reachability                  |
| `TCP`         | TCP connectivity check              |
| `UDP`         | UDP reachability check              |
| `HTTP`        | Application-level reachability      |
| `TWAMP`       | Two-Way Active Measurement Protocol |
| `TWAMP`       | Latency and jitter measurement      |

> 💡 **TWAMP** can be useful when measuring **latency and jitter**, especially for traffic such as VoIP and real-time applications.

---

## 🧠 Link Monitor Concept

```text
FortiGate
   │
   │ Health Check
   ▼
Remote Server / IP
   │
   ├── Reachable
   │      └── Link = UP
   │
   └── Unreachable
          └── Link = DOWN
                 │
                 ├── Update Routing Table
                 ├── Update Policy Route
                 └── Trigger Interface Failure
```

> ⚠️ The **source interface** should be explicitly defined when you need to control which path is used for the health check.

---

# ⚙️ Basic Link Monitor Configuration

```bash
config system link-monitor
    edit 1
        set srcintf agg1
        set server 5.200.200.200
        set gateway-ip 192.168.254.2
        set route 192.168.102.0/24 192.168.103.0/24
        set failtime 2
        set recoverytime 2
        set probe-count 5
    end
end
```

### 🔍 Important Parameters

| Parameter      | Meaning                                     |
| -------------- | ------------------------------------------- |
| `srcintf`      | Interface used to perform the health check  |
| `server`       | Destination/server being monitored          |
| `gateway-ip`   | Gateway used for the monitored path         |
| `route`        | Routes associated with the monitored link   |
| `failtime`     | Number of failures before declaring failure |
| `recoverytime` | Number of successful checks before recovery |
| `probe-count`  | Number of probes used for detection         |

> 💡 When `route` is configured, the specified route(s) are associated with the monitored path. If the monitor fails, the associated route can be removed from the RIB.

---

# 🔀 Multiple Link Monitors

Example with two independent links:

### WAN / Link 1

```bash
config system link-monitor
    edit mik-1
        set srcintf port4
        set server 8.8.8.8
        set gateway-ip 12.12.12.2
        set failtime 2
        set recoverytime 2
        set probe-count 5
    end
end
```

### WAN / Link 2

```bash
config system link-monitor
    edit mik-2
        set srcintf port5
        set server 8.8.8.8
        set gateway-ip 11.11.11.2
        set failtime 2
        set recoverytime 2
        set probe-count 5
    end
end
```

Check static routes:

```bash
get router info routing-table static
```

---

# 🔗 Update Cascade Interface

### What Does It Do?

`update-cascade-interface` allows a Link Monitor failure to trigger a **link-down event on other interfaces** when combined with interface failure detection.

```text
Link Monitor
     │
     ▼
Health Check Failure
     │
     ▼
Interface considered DOWN
     │
     ▼
Cascade failure
     │
     ▼
Other interface(s) affected
```

### ⚠️ Default Behavior

`update-cascade-interface` is enabled by default.

This is particularly important in **redundant-interface scenarios**.

---

## ❌ Separate Link Monitor Objects

Simply creating separate Link Monitor objects for individual links may not provide the desired cascade behavior.

For redundant-interface scenarios, configure the monitoring logic appropriately under the same Link Monitor object.

---

# 🛡️ Redundant Interface Example

```bash
config system link-monitor
    edit mik-1-red
        set srcintf red-1
        set server 8.8.8.8
        set gateway-ip 21.21.21.2
        set failtime 2
        set recoverytime 2
        set probe-count 5
    end
end
```

With:

```text
update-cascade-interface = enable
```

a failure can cause the monitored redundant interface to be treated as down.

---

## 🔓 Disable Cascade

If you want the remaining link to continue forwarding traffic:

```bash
config system link-monitor
    edit mik-1-red
        set srcintf red-1
        set server 8.8.8.8
        set gateway-ip 21.21.21.2
        set failtime 2
        set recoverytime 2
        set probe-count 5
        set update-cascade-interface disable
    end
end
```

### Behavior

```text
                red-1
             ┌─────────┐
             │ Link A  │ ── ❌ DOWN
             │ Link B  │ ── ✅ UP
             └─────────┘
                   │
                   ▼
       Continue forwarding on Link B
```

> ✅ Useful when you want the remaining redundant member to continue carrying traffic.

---

# 🔄 Other Update Features

These options are enabled by default:

```text
update-static-route
update-policy-route
```

### Effect

| Feature                    | Function                                  |
| -------------------------- | ----------------------------------------- |
| `update-static-route`      | Updates/removes associated static routes  |
| `update-policy-route`      | Updates associated policy routes          |
| `update-cascade-interface` | Cascades link failure to other interfaces |

---

# 🔎 Link Monitor Diagnostics

### Check Link Monitor Status

```bash
diagnose sys link-monitor status
```

### Tunnel Status

```bash
diagnose sys link-monitor tunnel all
```

### Interface Status

```bash
diagnose sys link-monitor interface red-1
```

### Manually Launch a Health Check

```bash
diagnose sys link-monitor launch mik-1-red
```

### Filter by Name

```bash
diagnose sys link-monitor filter name mik
```

---

# 🧩 HA + Link Monitor

In HA environments, Link Monitor status can contain HA-related information.

Example concepts:

```text
local alive
shared dead
```

### Interpretation

| State         | Meaning                                  |
| ------------- | ---------------------------------------- |
| `local alive` | Local monitoring state is alive          |
| `shared dead` | Shared/HA monitoring state is not active |

> 💡 `shared` relates to HA heartbeat/probe behavior.

---

## 🚩 Link Monitor Flags

### Init State

| Flag  | Meaning                             |
| ----- | ----------------------------------- |
| `0x1` | No HA pair                          |
| `0x9` | Failure related to HA or link state |

---

# 📊 Class ID

`class-id` is associated with **SD-WAN** behavior.

```text
Link Monitor
     │
     ▼
Health Check
     │
     ▼
Class ID
     │
     ▼
SD-WAN Path Evaluation
```

---

# 📞 Mean Opinion Score — MOS

**MOS (Mean Opinion Score)** is used for evaluating service quality, particularly for:

* VoIP
* Voice traffic
* Streaming
* Real-time services

When **service detection** is configured, MOS can be tracked as part of service-quality monitoring.

---

# 📡 Probe & Sequence

Diagnostic output may include information such as:

| Field         | Purpose                            |
| ------------- | ---------------------------------- |
| `packet sent` | Probes sent for health checking    |
| `sequence`    | ICMP probe sequence / RTT tracking |

The probe and sequence information can be used to evaluate:

* Link state
* Reachability
* RTT
* Probe behavior

> 💡 When troubleshooting detection problems, compare the probe/sequence behavior and verify that the expected probes are being generated.

---

# 🚨 Link Monitor + BFD

If Link Monitor detection is not behaving as expected, consider combining it with **BFD** for faster and more reliable failure detection.

```text
BFD
 │
 ├── Fast failure detection
 │
 ▼
Link Monitor
 │
 ├── Path / service health
 │
 ▼
Routing / SD-WAN decision
```

---

# 🎯 Server Configuration

Link Monitor can use different server configuration modes.

### Default

```bash
config system link-monitor
    edit mik-1-red
        set server-config default
    end
end
```

The server is defined directly under the Link Monitor configuration.

### Individual

```bash
config system link-monitor
    edit mik-1-red
        set server-config individual

        config server-list
            edit 1
                set dst 8.8.8.8
                set protocol ping
                set weight 30
            next

            edit 2
                set dst 5.200.200.200
                set protocol ping
                set weight 30
            end

        set fail-weight 60
    end
end
```

---

# ⚖️ Server Weight & Fail Weight

When using `server-list`, each server can have a **weight**.

Example:

```text
Server A → weight 30
Server B → weight 30

fail-weight = 60
```

### Example Logic

```text
             Link Monitor
                  │
        ┌─────────┴─────────┐
        ▼                   ▼
   8.8.8.8              5.200.200.200
   Weight 30              Weight 30
        │                   │
        └─────────┬─────────┘
                  ▼
             Total = 60
                  │
          fail-weight = 60
```

### One Server Fails

```text
Server A = 30
Server B = 0

Total failure weight = 30
```

Since:

```text
30 < 60
```

the failure threshold has **not** been reached.

### Both Servers Fail

```text
Server A = 30
Server B = 30

Total failure weight = 60
```

Therefore:

```text
60 >= fail-weight 60
```

→ Link failure is triggered.

---

## 💡 Recommended Redundancy Consideration

When using weighted server checks with redundant interfaces, consider:

```bash
set update-cascade-interface disable
```

This allows the remaining healthy path to continue forwarding when only one monitored path/server fails.

---

# 🚨 When All Servers Fail

Check:

```bash
diagnose sys link-monitor status
```

Pay attention to the initialization/state information and the resulting gateway/link state.

---

# ⏱️ Global Link Monitor Parameters

Example:

```bash
config system link-monitor
    edit 1
        set interval 500
        set probe-timeout 500
        set failtime 2
        set recoverytime 2
        set probe-count 5
        set ha-priority 1
        set service-detection disable
    end
end
```

---

## ⏱️ Interval

```bash
set interval 500
```

Health-check interval:

```text
500 ms
```

---

## ⏳ Probe Timeout

```bash
set probe-timeout 500
```

Maximum time to wait for a probe response:

```text
500 ms
```

---

## ❌ Fail Time

```bash
set failtime 2
```

Defines the failure threshold before the path is considered failed.

```text
Probe
  │
  ├── Failure
  ├── Failure
  │
  ▼
Link Failure
```

---

## ✅ Recovery Time

```bash
set recoverytime 2
```

Defines the recovery threshold before the path is considered healthy again.

```text
Probe
  │
  ├── Success
  ├── Success
  │
  ▼
Link Recovery
```

---

## 📡 Probe Count

```bash
set probe-count 5
```

Controls the number of probes used during monitoring.

> 💡 A practical value such as **5** can be used; avoid unnecessarily aggressive probe counts unless the design requires it.

---

# 🏆 HA Priority

```bash
set ha-priority 1
```

Primarily useful with **Active-Passive HA**.

Example:

```text
Active Device  → HA Priority 1
Passive Device → HA Priority 2
```

This can help control Link Monitor behavior in HA environments and reduce unwanted HA/split-brain scenarios.

---

# 🔍 Service Detection

```bash
set service-detection disable
```

### Disabled

Link Monitor primarily evaluates:

* Link state
* Failure state
* Route/path monitoring

### Enabled

Service-detection monitoring becomes active.

> ⚠️ Service detection is for service-quality monitoring and does not itself represent the same failure-triggering mechanism as basic link monitoring.

---

# 🌐 IPv4 Limitation

> ⚠️ This feature currently supports **IPv4** and **ICMP monitoring** in the referenced configuration context.

---

# 🔐 Link Monitor over IPsec Tunnel

When using Link Monitor over an IPsec tunnel:

```text
net-device = disable
```

Example:

```bash
config system link-monitor
    edit 1
        set srcintf advpn-branch
        set server-type dynamic
    end
end
```

### Dynamic Server Type

```bash
set server-type dynamic
```

Useful for:

* Dial-up VPN users
* Dynamic VPN endpoints
* ADVPN-related scenarios

---

# 🛠️ Link Monitor Troubleshooting Flow

```text
                Link Monitor Failure
                        │
                        ▼
               Check configuration
                        │
                        ▼
               Verify srcintf
                        │
                        ▼
                Verify gateway
                        │
                        ▼
               Verify server IP
                        │
                        ▼
             Check probe parameters
                        │
                        ▼
          diagnose sys link-monitor status
                        │
             ┌──────────┴──────────┐
             ▼                     ▼
         Link Alive             Link Failed
             │                     │
             ▼                     ▼
       Check routing         Check route removal
             │                     │
             ▼                     ▼
       Check SD-WAN          Check cascade
                                   │
                                   ▼
                              Check BFD
```

---

# 📡 FortiExtender

## 🎯 Purpose

**FortiExtender** provides additional WAN/LAN connectivity for FortiGate environments.

Common use cases:

* Backup WAN
* LTE connectivity
* Branch connectivity
* SD-WAN
* Out-of-band management
* LAN extension
* WAN extension
* IPsec over VXLAN
* Remote/branch deployments

---

# 🌐 FortiExtender Connectivity Modes

### WAN

```text
FortiExtender
      │
      ▼
    WAN
      │
      ▼
ISP / LTE
```

Useful for:

* Backup connectivity
* Carrier failover
* SD-WAN members

### LAN

Can be used for LAN extension scenarios, including:

```text
LAN
 │
 ▼
FortiExtender
 │
 ▼
VXLAN over IPsec
 │
 ▼
Headquarters
```

This provides **Layer-2 extension** toward the headquarters.

---

# 🔐 FortiExtender Management

FortiExtender uses **CAPWAP** for communication with FortiGate.

General requirements:

* FortiExtender appliance
* Appropriate license
* FortiGate controller
* CAPWAP connectivity

---

# 📊 Maximum FortiExtender Numbers

| FortiGate Platform                 | WAN |  LAN |
| ---------------------------------- | --: | ---: |
| 40F series + variants / VM01       |   2 |    0 |
| 60F / 70F / 80F + variants / VM02  |   2 |   16 |
| 100E → 200F + variants             |   2 |   16 |
| 400F → 900G + variants / VM04      |   2 |   32 |
| 1000D → 2600F + variants / VM08    |   2 |  256 |
| 3000D and higher / VM16 and higher |   2 | 1024 |

---

# ⚙️ Enable FortiExtender

```bash
config system global
    set fortiextender enable
    set fortiextender-data-port 25246
    set fortiextender-discovery-lockdown disable
    set fortiextender-vlan-mode disable
    set fortiservice-port 8013
    set fortitoken-cloud enable
    set gui-forticare-registration-setup-warning enable
    set gui-fortigate-cloud-sandbox disable
end
```

---

# 🖥️ GUI Feature Visibility

Navigate to:

```text
System
 └── Feature Visibility
      └── FortiExtender
```

Then:

```text
Network
 └── FortiExtender
```

---

# 📡 FortiExtender Controller

Example:

```bash
config extender-controller extender
    edit fex
        set authorized enable

        config modem1
            set ifname eth1
            set redundant-mode enable
            set redundant-intf port1
        end
    end
end
```

---

# 🔄 FortiExtender + SD-WAN

FortiExtender can be integrated into **SD-WAN**.

A common design approach:

```text
                SD-WAN
                   │
       ┌───────────┼───────────┐
       ▼           ▼           ▼
   Internet      MPLS      FortiExtender
                              │
                              ▼
                             LTE
```

FortiExtender can be assigned a higher SD-WAN priority when it is intended to act as a preferred backup/alternate path.

---

# 📶 LTE Modem

Enable LTE:

```bash
config system lte-modem
    set status enable
    set apn rightel
end
```

The WAN interface can then obtain connectivity through DHCP and negotiate with the ISP.

> 💡 FortiGate **30E** includes a built-in LTE modem.

---

# 📱 LTE Configuration Example

```bash
config system lte-modem
    set status enable
    set extra-init port1
    set manual-handover disable
    set force-wireless-profile 0
    set authtype none
    set apn rightel
    set modem-port 255
    set network-type auto
    set auto-connect disable
    set gpsd-enabled disable
    set data-usage-tracking disable
    set gps-port 255
end
```

### `modem-port`

```bash
set modem-port 255
```

`255` indicates automatic modem detection in this configuration.

---

# 📡 Modem / SIM Concepts

### Dual Modem

```text
2 Modems
   │
   ├── SIM 1
   └── SIM 2
```

### Single Modem

```text
1 Modem
   │
   ├── SIM 1
   └── SIM 2
```

The exact available SIM/modem behavior depends on the FortiExtender hardware model.

---

# 🚀 FortiExtender Features

| Feature                           | Use Case                               |
| --------------------------------- | -------------------------------------- |
| **VRRP**                          | HA / redundancy                        |
| **Split Tunneling**               | Selective tunnel traffic               |
| **Carrier Failover**              | Switch between carriers                |
| **Out-of-Band Management**        | Management during primary-path failure |
| **LAN Extension**                 | Extend LAN connectivity                |
| **VXLAN over IPsec**              | Layer-2 extension                      |
| **WAN Extension**                 | Extend WAN connectivity                |
| **Active / Passive Connectivity** | Redundant connectivity                 |

---

# 🏢 Common Deployment

FortiExtender is commonly useful in:

* Branch offices
* POS environments
* Remote locations
* Backup-WAN designs
* LTE failover
* SD-WAN deployments

---

# 📶 LTE Categories

| LTE Category | Approx. Speed |
| ------------ | ------------: |
| **Cat 4**    |      150 Mbps |
| **Cat 6**    |      300 Mbps |
| **Cat 7**    |  100–300 Mbps |
| **Cat 12**   |      600 Mbps |
| **Cat 16**   |        1 Gbps |
| **Cat 20**   |      2–5 Gbps |

> ⚠️ Actual throughput depends on the modem, carrier, spectrum, signal quality, network conditions and deployment.

---

# 🧭 Quick Reference

## Link Monitor

```text
Link Monitor
├── Health Check
│   ├── ICMP
│   ├── TCP
│   ├── UDP
│   ├── HTTP
│   └── TWAMP
│
├── Path Monitoring
│   ├── Server
│   ├── Gateway
│   └── Source Interface
│
├── Failure Actions
│   ├── Update Static Route
│   ├── Update Policy Route
│   └── Cascade Interface
│
├── SD-WAN
│   ├── Class ID
│   └── Service Detection
│
├── HA
│   └── HA Priority
│
└── Diagnostics
    ├── status
    ├── tunnel
    ├── interface
    ├── launch
    └── filter
```

## FortiExtender

```text
FortiExtender
├── WAN
│   ├── LTE
│   ├── Backup WAN
│   └── SD-WAN
│
├── LAN
│   └── LAN Extension
│
├── Tunneling
│   └── VXLAN over IPsec
│
├── HA / Redundancy
│   └── VRRP
│
├── Management
│   └── Out-of-Band
│
└── Cellular
    ├── SIM
    ├── Carrier Failover
    └── LTE Categories
```

---

# 🧪 Essential Commands

```bash
# Link Monitor status
diagnose sys link-monitor status

# Check tunnels
diagnose sys link-monitor tunnel all

# Check interface
diagnose sys link-monitor interface red-1

# Manually trigger health check
diagnose sys link-monitor launch mik-1-red

# Filter Link Monitor objects
diagnose sys link-monitor filter name mik

# Check static routing
get router info routing-table static
```

---

# 🧠 Key Takeaways

> **1.** Always pay attention to the **source interface** used by Link Monitor.

> **2.** `update-static-route` and `update-policy-route` control route-related reactions to Link Monitor state.

> **3.** `update-cascade-interface` can propagate a monitored failure to other interfaces.

> **4.** For redundant interfaces, carefully decide whether cascade behavior should remain enabled.

> **5.** `server-list` with **weights + fail-weight** provides more granular failure evaluation.

> **6.** `ha-priority` is particularly relevant when Link Monitor is used with **Active-Passive HA**.

> **7.** Link Monitor is closely related to **SD-WAN path health evaluation**.

> **8.** Use **BFD + Link Monitor** when faster or complementary failure detection is required.

> **9.** FortiExtender is useful for **LTE backup, SD-WAN, branch connectivity and out-of-band management**.

> **10.** In redundant designs, understand the difference between **link failure, route failure and interface cascade** before enabling failover behavior.

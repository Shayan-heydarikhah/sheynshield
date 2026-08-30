# 🛡️ FortiGate SD-WAN Speed Test & Packet Duplication Checklist

> **FortiOS Focus:** SD-WAN · Speed Test · SLA · Packet Duplication · Packet De-Duplication · ADVPN · BGP · Underlay/Overlay · Large-Scale Deployment
> **Use Case:** Configuration · Verification · Troubleshooting · NSE Exam Review · Production Design
> **Format:** Operational Checklist / CLI Quick Reference

---

## 📌 Table of Contents

* [01 — SD-WAN Speed Test](#01--sd-wan-speed-test)
* [02 — Speed-Test Server](#02--speed-test-server)
* [03 — Speed-Test Schedule](#03--speed-test-schedule)
* [04 — Speed-Test Routing](#04--speed-test-routing)
* [05 — Manual Speed Test](#05--manual-speed-test)
* [06 — Speed-Test Troubleshooting](#06--speed-test-troubleshooting)
* [07 — Hub-and-Spoke Speed-Test Design](#07--hub-and-spoke-speed-test-design)
* [08 — Packet Duplication](#08--packet-duplication)
* [09 — Packet Duplication Modes](#09--packet-duplication-modes)
* [10 — Packet De-Duplication](#10--packet-de-duplication)
* [11 — Duplication with SLA](#11--duplication-with-sla)
* [12 — `sla-match-service`](#12--sla-match-service)
* [13 — Packet Duplication with ADVPN](#13--packet-duplication-with-advpn)
* [14 — Large-Scale SD-WAN Design](#14--large-scale-sd-wan-design)
* [15 — ADVPN Mode Config](#15--advpn-mode-config)
* [16 — Troubleshooting Checklist](#16--troubleshooting-checklist)
* [17 — Quick CLI Reference](#17--quick-cli-reference)
* [18 — Design Rules](#18--design-rules)
* [19 — High-Value Gotchas](#19--high-value-gotchas)
* [20 — Final Validation](#20--final-validation)

---

# 01 — SD-WAN Speed Test

## 🎯 Objective

Use SD-WAN Speed Test to measure WAN-path performance and provide bandwidth information that can be used by SD-WAN/VPN mechanisms.

### Checklist

* [ ] Identify the SD-WAN member/interface to test.
* [ ] Determine whether the test should use the **underlay** or **VPN overlay**.
* [ ] Verify the Speed-Test server.
* [ ] Verify administrative access on the test interface.
* [ ] Verify routing toward the test server.
* [ ] Verify SLA/health-check configuration.
* [ ] Run a manual test before relying on scheduled measurements.
* [ ] Verify the measured bandwidth after the test.
* [ ] Check Speed-Test daemon logs if the test fails.

### Mental Model

```text
WAN Link
   │
   ▼
Speed Test
   │
   ├── Latency
   ├── Jitter
   ├── Packet Loss
   └── Bandwidth
          │
          ▼
      SD-WAN / VPN
```

> [!IMPORTANT]
> Speed Test is a **measurement mechanism**. Do not confuse it with the SD-WAN service rule that ultimately decides traffic steering.

---

# 02 — Speed-Test Server

## Enable Local FortiOS Speed-Test Server

```cli
config system global
    set speedtest-server enable
end
```

### Checklist

* [ ] Enable the Speed-Test server on the FortiGate acting as the server.
* [ ] Verify the interface receiving Speed-Test traffic.
* [ ] Verify administrative access.
* [ ] Verify firewall policy requirements.
* [ ] Verify routing between client and server.

### Architecture

```text
              Speed Test
                  │
                  ▼
        ┌──────────────────┐
        │ FortiGate Server │
        │ speedtest-server │
        └──────────────────┘
```

---

# 03 — Speed-Test Schedule

## Dynamic Server

```cli
config system speed-test-schedule
    edit hs-ad-1
        set dynamic-server enable
    next
end
```

Or:

```cli
config system speed-test-schedule
    edit port1
        set dynamic-server enable
    next
end
```

### Checklist

* [ ] Configure the Speed-Test schedule.
* [ ] Enable `dynamic-server` when using the dynamic server mechanism.
* [ ] Verify the correct interface/member.
* [ ] Verify administrative access on the ISP/physical interface.
* [ ] Verify reachability to the selected test server.

---

# 04 — Speed-Test Routing

## Bypass Normal SD-WAN Routing

```cli
config system sdwan
    set speedtest-bypass-routing enable
end
```

### Behavior

| Setting   | Behavior                                                                             |
| --------- | ------------------------------------------------------------------------------------ |
| `enable`  | Speed Test uses manually configured test servers and bypasses normal dynamic routing |
| `disable` | VPN/overlay routing is used by default                                               |

### Troubleshooting Question

> **Is the Speed-Test traffic using the underlay or the VPN overlay?**

```text
                    Speed Test
                        │
             ┌──────────┴──────────┐
             │                     │
             ▼                     ▼
          Underlay              Overlay
             │                     │
         ISP/WAN              IPsec VPN
```

### Checklist

* [ ] Check `speedtest-bypass-routing`.
* [ ] Check routing table.
* [ ] Check BGP-learned routes.
* [ ] Check IPsec routes.
* [ ] Confirm the actual egress interface.
* [ ] Verify the test destination.

> [!TIP]
> If the objective is measuring the **raw WAN underlay**, make sure the test is not accidentally reaching the destination through the VPN overlay.

---

# 05 — SD-WAN Neighbor Speed-Test Mode

```cli
config system sdwan
    config neighbor
        edit 12.23.34.1
            set mode speedtest
        next
    end
end
```

### Checklist

* [ ] Identify the SD-WAN neighbor.
* [ ] Configure `mode speedtest`.
* [ ] Verify neighbor reachability.
* [ ] Verify the correct underlay interface.
* [ ] Verify Speed-Test server behavior.

---

# 06 — Manual Speed Test

## Blocking Test

```cli
execute speed-test-dynamic hs-ad-1 all n
```

## Non-Blocking Test

```cli
diagnose netlink interface speed-test hs-ad-23
```

### Checklist

* [ ] Run a manual Speed Test.
* [ ] Verify test completion.
* [ ] Record measured bandwidth.
* [ ] Compare results with ISP expectations.
* [ ] Compare results across SD-WAN members.
* [ ] Verify whether the result is being cached/applied.

---

# 07 — Speed-Test Troubleshooting

## Speed-Test Daemon

```cli
diagnose debug application speedtest 100
diagnose debug application speedtestd 100
diagnose debug enable
```

## FortiCron / Scheduler

```cli
diagnose test application forticron 9
diagnose test application forticron 10
diagnose test application forticron 11
diagnose test application forticron 12
diagnose test application forticron 99
```

### Troubleshooting Checklist

* [ ] Speed-Test server enabled.
* [ ] Speed-Test schedule configured.
* [ ] Dynamic/custom server configured.
* [ ] Administrative access enabled.
* [ ] Firewall policy allows required traffic.
* [ ] Routing is correct.
* [ ] Underlay path is reachable.
* [ ] Test server is operational.
* [ ] FortiCron scheduler is functioning.
* [ ] Speed-Test daemon is operational.
* [ ] Debug output shows successful test execution.
* [ ] Measured bandwidth is returned.

---

# 08 — Hub-and-Spoke Speed-Test Design

Typical large-scale architecture:

```text
                         HUB
                          │
              ┌───────────┴───────────┐
              │                       │
           Spoke-1                 Spoke-2
              │                       │
           Underlay                Underlay
```

### Recommended Workflow

* [ ] Configure recurring Speed Test on the hub.
* [ ] Configure spokes as Speed-Test servers.
* [ ] Allow Speed-Test traffic on spoke underlay interfaces.
* [ ] Establish BGP over the VPN overlay.
* [ ] Advertise spoke networks/loopbacks.
* [ ] Prevent the test destination from being reached through the VPN overlay when measuring underlay performance.
* [ ] Run Speed Test.
* [ ] Measure egress bandwidth.
* [ ] Apply/capture the measured bandwidth.
* [ ] Verify cached results after tunnel reconnect.

### Normal Traffic

```text
Hub
 │
 ▼
VPN Overlay
 │
 ▼
Spoke Network
```

### Speed-Test Traffic

```text
Hub
 │
 ▼
Underlay
 │
 ▼
Spoke Speed-Test Server
```

> [!IMPORTANT]
> The key design objective is to ensure that a test intended to measure the underlay does **not accidentally traverse the overlay**.

---

# 09 — Speed Test + Route Maps

### Before Speed Test

```text
Hub
 │
 ├── Underlay
 │
 └── VPN Overlay ──> Spoke Network
```

### During Underlay Measurement

```text
Hub
 │
 ▼
Underlay
 │
 ▼
Spoke Speed-Test Server

VPN Overlay
     X
```

### Checklist

* [ ] Identify the test destination.
* [ ] Check BGP advertisements.
* [ ] Check route-map configuration.
* [ ] Prevent unwanted overlay reachability.
* [ ] Confirm underlay route selection.
* [ ] Run the Speed Test.
* [ ] Restore normal route advertisement behavior.
* [ ] Verify normal VPN routing afterward.

---

# 10 — Speed-Test Result Handling

```text
Speed Test
    │
    ▼
Measured Bandwidth
    │
    ▼
VPN / SD-WAN Tunnel
    │
    ▼
Dynamic Bandwidth
    │
    ▼
Cached Result
```

### Validation Checklist

* [ ] Speed Test completes successfully.
* [ ] Bandwidth result is generated.
* [ ] Result is associated with the expected tunnel/member.
* [ ] Dynamic bandwidth is updated.
* [ ] Cached result survives tunnel disconnect/reconnect as expected.

---

# 11 — Packet Duplication

Packet duplication sends copies of the same packet through multiple eligible SD-WAN members.

```text
                       ┌── WAN-1
                       │
Client ──> SD-WAN ─────┼── WAN-2
                       │
                       └── WAN-3
```

The receiving side can identify duplicate packets and keep one copy.

### Primary Goals

* [ ] Improve resiliency.
* [ ] Reduce the impact of packet loss.
* [ ] Improve recovery over unstable WAN links.
* [ ] Protect loss-sensitive applications.
* [ ] Use multiple paths simultaneously.

> [!WARNING]
> Packet duplication consumes additional bandwidth because one logical packet can generate multiple transmissions.

---

# 12 — Packet Duplication Configuration

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

### Configuration Checklist

* [ ] Define source address.
* [ ] Define destination address.
* [ ] Define source interface.
* [ ] Define destination SD-WAN zone/member.
* [ ] Select applicable service.
* [ ] Select duplication mode.
* [ ] Configure de-duplication when required.
* [ ] Set an appropriate duplication member limit.
* [ ] Validate bandwidth impact.

---

# 13 — Packet Duplication Modes

| Mode        | Behavior                                                  |
| ----------- | --------------------------------------------------------- |
| `disable`   | No packet duplication                                     |
| `force`     | Duplicate traffic across eligible members                 |
| `on-demand` | Duplicate traffic when link-quality conditions require it |

---

## `disable`

```cli
set packet-duplication disable
```

### Behavior

```text
Packet
  │
  ▼
Single Path
```

### Checklist

* [ ] Confirm duplication is intentionally disabled.
* [ ] Verify normal SD-WAN forwarding.

---

## `force`

```cli
set packet-duplication force
```

### Behavior

```text
                ┌── WAN-1
                │
Packet ─────────┼── WAN-2
                │
                └── WAN-3
```

### Checklist

* [ ] Confirm eligible SD-WAN members.
* [ ] Verify `duplication-max-num`.
* [ ] Verify SLA/member state.
* [ ] Monitor WAN bandwidth consumption.
* [ ] Verify duplicate packet handling.

> [!IMPORTANT]
> A failed/unhealthy SD-WAN member can be excluded from the duplication set.

---

## `on-demand`

```cli
set packet-duplication on-demand
```

### Behavior

```text
Healthy SLA
    │
    ▼
Normal Forwarding

Degraded SLA
    │
    ▼
Packet Duplication
    ├── WAN-1
    └── WAN-2
```

### Checklist

* [ ] Verify SLA health checks.
* [ ] Verify SLA thresholds.
* [ ] Verify service-rule matching.
* [ ] Verify `sla-match-service` behavior.
* [ ] Confirm duplication activates only when conditions require it.

> [!TIP]
> `on-demand` can reduce bandwidth consumption compared with permanently duplicating all traffic.

---

# 14 — Packet De-Duplication

Enable:

```cli
set packet-de-duplication enable
```

Default:

```text
disable
```

### Concept

```text
                 ┌── Packet A ──┐
Sender ──────────┤              ├──> Receiver
                 └── Packet A ──┘
                         │
                         ▼
                  Keep one copy
                  Drop duplicate
```

### Checklist

* [ ] Confirm the receiving FortiGate supports the required de-duplication behavior.
* [ ] Enable `packet-de-duplication`.
* [ ] Verify duplicate packets are detected.
* [ ] Capture traffic when troubleshooting.
* [ ] Confirm only one copy reaches the application.

---

# 15 — Maximum Duplication Members

```cli
set duplication-max-num 2
```

Example:

```cli
set duplication-max-num 4
```

### Four-Member Example

```text
Packet
  │
  ├── WAN-1
  ├── WAN-2
  ├── WAN-3
  └── WAN-4
```

### Checklist

* [ ] Determine how many WAN paths should carry duplicates.
* [ ] Configure `duplication-max-num`.
* [ ] Confirm enough healthy SD-WAN members exist.
* [ ] Evaluate bandwidth multiplication.
* [ ] Verify SLA/member health.

---

# 16 — Duplication + SD-WAN Service Rules

Duplication can be associated with a specific SD-WAN service:

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

### Design Principle

```text
Traffic
   │
   ▼
SD-WAN Service Rule
   │
   ▼
Duplication Policy
   │
   ▼
Multiple WAN Paths
```

### Checklist

* [ ] Identify the SD-WAN service ID.
* [ ] Match the correct application/traffic.
* [ ] Associate duplication with the intended service.
* [ ] Verify rule ordering.
* [ ] Validate actual traffic behavior.

---

# 17 — Packet Duplication + SLA

## Force Mode

```text
SLA Healthy
    │
    ▼
Duplicate
 ├── Member-1
 └── Member-2
    │
    ▼
De-Duplicate
```

## On-Demand Mode

```text
SLA Healthy
    │
    ▼
Normal Forwarding

SLA Degraded
    │
    ▼
Duplicate
 ├── Member-1
 └── Member-2
```

### Checklist

* [ ] Verify health-check status.
* [ ] Verify latency threshold.
* [ ] Verify jitter threshold.
* [ ] Verify packet-loss threshold.
* [ ] Confirm duplication activation logic.
* [ ] Confirm service-rule association.

---

# 18 — `sla-match-service`

When using `on-demand`, SLA matching determines which health-check context is evaluated.

### Without Service Matching

```text
Relevant SLA
     │
     ▼
Threshold exceeded?
     │
   ┌─┴─┐
  YES  NO
   │    │
 Dup   Normal
```

### With Service Matching

```text
SD-WAN Service
      │
      ▼
Service SLA
      │
      ▼
Threshold exceeded?
      │
   ┌──┴──┐
  YES   NO
   │     │
 Dup   Normal
```

### Checklist

* [ ] Identify the SD-WAN service.
* [ ] Identify associated SLA health checks.
* [ ] Verify `sla-match-service`.
* [ ] Verify threshold values.
* [ ] Confirm expected duplication trigger.
* [ ] Test healthy and degraded conditions.

---

# 19 — Packet Duplication with ADVPN

Packet duplication can be useful in Hub-and-Spoke and ADVPN environments.

```text
                    HUB
                  /     \
              VPN-1     VPN-2
               /           \
          Spoke-1         Spoke-2
```

### Use Cases

* [ ] Loss-sensitive traffic.
* [ ] Unstable WAN paths.
* [ ] Critical applications.
* [ ] Rapid packet-loss recovery.
* [ ] Temporary path degradation.

### ADVPN Relationship

```text
Hub
 │
 ▼
Spoke
 │
 ▼
Dynamic Tunnel
 │
 ▼
Spoke-to-Spoke Path
```

> [!NOTE]
> ADVPN can establish direct spoke-to-spoke tunnels, reducing the need for traffic to remain hub-routed.

---

# 20 — Speed Test vs Packet Duplication

| Capability                  | Speed Test              | Packet Duplication         |
| --------------------------- | ----------------------- | -------------------------- |
| Primary purpose             | Measure WAN performance | Improve traffic resiliency |
| Additional traffic          | Yes                     | Yes                        |
| Uses SLA information        | Yes                     | Yes                        |
| Measures bandwidth          | Yes                     | No                         |
| Sends packet copies         | No                      | Yes                        |
| Extra bandwidth consumption | Yes                     | Yes                        |
| Main operational role       | Measurement             | Resiliency                 |
| Typical use                 | Bandwidth estimation    | Loss-sensitive traffic     |

### Memory Rule

```text
Speed Test
    =
Measure

SLA
    =
Evaluate

SD-WAN Rule
    =
Steer

Duplication
    =
Replicate

De-Duplication
    =
Eliminate Copies
```

---

# 21 — Large-Scale SD-WAN Design

A scalable Fortinet SD-WAN deployment may combine:

```text
                 SD-WAN
                    │
       ┌────────────┼────────────┐
       │            │            │
      SLA          BGP         ADVPN
       │            │            │
       └────────────┼────────────┘
                    │
              Speed Test
                    │
                    ▼
            Link Measurement
```

### Technology Checklist

* [ ] SD-WAN.
* [ ] ADVPN.
* [ ] BGP.
* [ ] SLA health checks.
* [ ] Speed Test.
* [ ] Packet duplication.
* [ ] Packet de-duplication.
* [ ] FEC where appropriate.
* [ ] Traffic steering.
* [ ] Route maps.
* [ ] Underlay/overlay separation.

---

# 22 — ADVPN Mode Config

For spokes requiring Mode Config:

```cli
config vpn ipsec phase1-interface
    edit link-1
        set mode-cfg enable
        set mode-cfg-allow-client-selector enable
    next
end
```

### Checklist

* [ ] Enable `mode-cfg`.
* [ ] Enable client selector support when required.
* [ ] Verify Phase 1 configuration.
* [ ] Verify hub/spoke relationship.
* [ ] Verify assigned parameters.
* [ ] Verify dynamic tunnel establishment.

### Mental Model

```text
Spoke
  │
  ▼
Mode Config
  │
  ├── Assigned Parameters
  └── Client Selector
```

---

# 23 — Speed-Test Troubleshooting Checklist

## Server

* [ ] Speed-Test server enabled.
* [ ] Correct server selected.
* [ ] Server reachable.
* [ ] Required administrative access enabled.
* [ ] Firewall policy verified.

## Schedule

* [ ] Correct schedule exists.
* [ ] Schedule is enabled.
* [ ] Dynamic/custom server configuration verified.
* [ ] FortiCron operation verified.

## Routing

* [ ] Routing table checked.
* [ ] Underlay route exists.
* [ ] Overlay route checked.
* [ ] Route-map behavior verified.
* [ ] `speedtest-bypass-routing` checked.

## Debug

```cli
diagnose debug application speedtest 100
diagnose debug application speedtestd 100
diagnose debug enable
```

## Scheduler

```cli
diagnose test application forticron 9
diagnose test application forticron 10
diagnose test application forticron 11
diagnose test application forticron 12
diagnose test application forticron 99
```

---

# 24 — Packet-Duplication Troubleshooting Checklist

### SD-WAN

* [ ] SD-WAN members are healthy.
* [ ] Correct SD-WAN zone is selected.
* [ ] Correct service rule is matched.
* [ ] SLA status is healthy/degraded as expected.

### Duplication

* [ ] Duplication policy exists.
* [ ] Source matches.
* [ ] Destination matches.
* [ ] Interface matches.
* [ ] Service matches.
* [ ] `packet-duplication` mode is correct.
* [ ] `duplication-max-num` is correct.

### De-Duplication

* [ ] `packet-de-duplication` enabled where required.
* [ ] Receiving FortiGate is participating.
* [ ] Duplicate packets visible in capture.
* [ ] Receiver keeps one copy.

### Traffic Capture

```text
Sender
  │
  ├── Path-1 ──┐
  │             │
  └── Path-2 ──┼──> Receiver
                │
                ▼
          De-Duplication
```

---

# 25 — Fast CLI Reference

| Task                             | Command / Setting                           |
| -------------------------------- | ------------------------------------------- |
| Enable local Speed-Test server   | `set speedtest-server enable`               |
| Enable dynamic Speed-Test server | `set dynamic-server enable`                 |
| Bypass normal routing            | `set speedtest-bypass-routing enable`       |
| Speed-Test neighbor              | `set mode speedtest`                        |
| Run dynamic Speed Test           | `execute speed-test-dynamic ...`            |
| Non-blocking Speed Test          | `diagnose netlink interface speed-test ...` |
| Debug Speed Test                 | `diagnose debug application speedtest 100`  |
| Debug Speed-Test daemon          | `diagnose debug application speedtestd 100` |
| Disable duplication              | `set packet-duplication disable`            |
| Force duplication                | `set packet-duplication force`              |
| Conditional duplication          | `set packet-duplication on-demand`          |
| Enable de-duplication            | `set packet-de-duplication enable`          |
| Maximum duplicate members        | `set duplication-max-num <N>`               |
| Enable ADVPN Mode Config         | `set mode-cfg enable`                       |
| Allow client selector            | `set mode-cfg-allow-client-selector enable` |

---

# 26 — High-Value Gotchas

> [!WARNING]
>
> ### GOTCHA #1 — Speed Test Is Not the Same as SLA
>
> **Speed Test measures performance.**
> **SLA evaluates link health.**
> **SD-WAN rules steer traffic.**

---

> [!WARNING]
>
> ### GOTCHA #2 — Underlay vs Overlay
>
> A Speed Test intended to measure the ISP path can produce misleading results if traffic reaches the destination through the VPN overlay.

```text
Underlay = Transport

Overlay = VPN Tunnel

Speed Test = Measurement
```

---

> [!WARNING]
>
> ### GOTCHA #3 — Packet Duplication Costs Bandwidth
>
> Sending one packet across two links can approximately double the transmitted packet volume for that traffic.

```text
1 Logical Packet
      │
      ├── WAN-1
      └── WAN-2

≈ 2 Transmissions
```

---

> [!WARNING]
>
> ### GOTCHA #4 — `on-demand` Depends on Link Quality
>
> `on-demand` is not equivalent to permanent duplication. Verify the SLA conditions that trigger replication.

---

> [!WARNING]
>
> ### GOTCHA #5 — De-Duplication Matters
>
> Duplication without appropriate de-duplication can result in duplicate traffic reaching the receiving/application side.

---

> [!WARNING]
>
> ### GOTCHA #6 — Route Maps Can Be Critical
>
> In Hub-and-Spoke Speed-Test designs, route filtering can prevent the hub from accidentally selecting the VPN overlay instead of the underlay.

---

> [!WARNING]
>
> ### GOTCHA #7 — ADVPN Changes the Path Model
>
> Once direct spoke-to-spoke tunnels are established, traffic may no longer follow the original hub-forwarding path.

---

# 27 — NSE / Interview Memory Sheet

| Question                            | Answer                                           |
| ----------------------------------- | ------------------------------------------------ |
| What does Speed Test do?            | Measures WAN/link performance                    |
| What does SLA do?                   | Evaluates link health                            |
| What does SD-WAN rule do?           | Selects/steers traffic path                      |
| What does packet duplication do?    | Sends packet copies over multiple paths          |
| What does de-duplication do?        | Removes duplicate packets                        |
| What does `force` mean?             | Always duplicate across eligible paths           |
| What does `on-demand` mean?         | Duplicate when link conditions require it        |
| Why use `duplication-max-num`?      | Limit number of paths used for duplicate packets |
| Why use route maps with Speed Test? | Prevent unintended overlay routing               |
| What is ADVPN?                      | Dynamic VPN tunnel architecture                  |
| What does BGP provide?              | Dynamic route advertisement/learning             |
| What is the underlay?               | Physical/IP transport network                    |
| What is the overlay?                | VPN/network built over the underlay              |
| What is Mode Config?                | Dynamic VPN client configuration mechanism       |

---

# 28 — Final Validation Checklist

## 🧪 Speed Test

* [ ] Speed-Test server verified.
* [ ] Schedule verified.
* [ ] Interface access verified.
* [ ] Dynamic/custom server verified.
* [ ] Routing verified.
* [ ] Underlay path verified.
* [ ] Overlay path ruled out when necessary.
* [ ] Manual test completed.
* [ ] Speed-Test debug checked.
* [ ] Result verified.

## 🔁 Packet Duplication

* [ ] SD-WAN members verified.
* [ ] SLA verified.
* [ ] Duplication policy verified.
* [ ] Service matching verified.
* [ ] Duplication mode verified.
* [ ] Maximum duplication members verified.
* [ ] De-duplication verified.
* [ ] Packet capture completed where necessary.
* [ ] Bandwidth impact evaluated.

## 🌐 ADVPN

* [ ] Hub configuration verified.
* [ ] Spoke configuration verified.
* [ ] BGP established.
* [ ] Mode Config verified when required.
* [ ] Client selector verified.
* [ ] Dynamic tunnel behavior verified.
* [ ] Route selection verified.

## 🏗️ Production Design

* [ ] WAN capacity evaluated.
* [ ] SLA thresholds defined.
* [ ] Critical applications identified.
* [ ] Duplication limited to appropriate traffic.
* [ ] Bandwidth overhead calculated.
* [ ] Underlay/overlay routing documented.
* [ ] Failure scenarios tested.
* [ ] Monitoring configured.
* [ ] Rollback procedure documented.

---

# 🧠 One-Page SD-WAN Memory Map

```text
                         SD-WAN
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
        ▼                   ▼                   ▼
     SPEED TEST             SLA                RULE
        │                   │                   │
     Measure              Health              Steer
        │                   │                   │
        └───────────────────┼───────────────────┘
                            │
                            ▼
                         ADVPN
                            │
                       Dynamic VPN
                            │
                            ▼
                           BGP
                            │
                       Route Exchange
                            │
                            ▼
                    PACKET DUPLICATION
                            │
                 ┌──────────┴──────────┐
                 │                     │
               Force               On-Demand
                 │                     │
                 ▼                     ▼
              Copies              SLA Trigger
                 │                     │
                 └──────────┬──────────┘
                            ▼
                    PACKET DE-DUPLICATION
                            │
                            ▼
                      Keep One Copy
```

---

# 🏁 Final Rule

```text
SPEED TEST
    ↓
Measure the WAN

SLA
    ↓
Determine link quality

SD-WAN RULE
    ↓
Choose the path

ADVPN
    ↓
Build dynamic VPN paths

BGP
    ↓
Exchange routes

PACKET DUPLICATION
    ↓
Replicate critical traffic

PACKET DE-DUPLICATION
    ↓
Remove duplicate copies
```

> **Don't troubleshoot SD-WAN by looking at only one layer.**
>
> **Measure → Evaluate → Route → Replicate → Verify**

---

## 📌 Version & Lab Note

This checklist is based on the supplied FortiOS training material and should be validated against the exact FortiOS release, hardware/VM platform, topology, and current Fortinet documentation before production deployment.

> [!CAUTION]
> CLI syntax, feature behavior, supported platforms, limits, and defaults can vary between FortiOS releases. Treat lab-specific values as examples rather than universal production values.

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


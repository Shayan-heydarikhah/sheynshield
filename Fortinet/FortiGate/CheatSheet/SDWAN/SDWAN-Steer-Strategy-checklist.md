# 🔀 FortiGate SD-WAN Strategies — Advanced Checklist

> **FortiOS | SD-WAN Strategy Selection, SLA Steering, Manual Routing, Load Balancing, Bandwidth Optimization, Application Steering & Troubleshooting**

[![FortiGate](https://img.shields.io/badge/FortiGate-SD--WAN-red)](https://www.fortinet.com/products/secure-sd-wan)
[![FortiOS](https://img.shields.io/badge/FortiOS-SD--WAN-blue)](https://docs.fortinet.com/)
[![GitHub](https://img.shields.io/badge/GitHub-SheynShield-black)](https://github.com/Shayan-heydarikhah/sheynshield)

---

## 📑 Table of Contents

* [SD-WAN Strategy Fundamentals](#-sd-wan-strategy-fundamentals)
* [Pre-Deployment Checklist](#-pre-deployment-checklist)
* [Manual Strategy](#-manual-strategy)
* [Best Quality Strategy](#-best-quality-strategy)
* [Minimum SLA Strategy](#-minimum-sla-strategy)
* [Maximize Bandwidth Strategy](#-maximize-bandwidth-strategy)
* [Load Balance Strategy](#-load-balance-strategy)
* [SD-WAN Hash Modes](#-sd-wan-hash-modes)
* [Bandwidth Measurement](#-bandwidth-measurement)
* [Interface vs Zone Priority](#-interface-vs-zone-priority)
* [Application-Based Steering](#-application-based-steering)
* [QoS and Traffic Shaping](#-qos-and-traffic-shaping)
* [Policy Route Verification](#-policy-route-verification)
* [SD-WAN Troubleshooting Checklist](#-sd-wan-troubleshooting-checklist)
* [Strategy Selection Matrix](#-strategy-selection-matrix)
* [Advanced Decision Model](#-advanced-decision-model)
* [Quick CLI Reference](#-quick-cli-reference)
* [Key Takeaways](#-key-takeaways)
* [SheynShield Resources](#-sheynshield-resources)

---

# 🔹 SD-WAN Strategy Fundamentals

SD-WAN Service rules determine **how FortiGate selects an eligible SD-WAN member**.

Before selecting a strategy, verify:

* [ ] SD-WAN members are configured.
* [ ] SD-WAN zones are correctly defined.
* [ ] Required gateways are reachable.
* [ ] SLA health checks are configured where required.
* [ ] Source and destination matching is correct.
* [ ] Firewall policies allow the traffic.
* [ ] Routing/FIB provides a valid forwarding path.
* [ ] Application identification is working when application steering is required.
* [ ] Bandwidth values are accurate when bandwidth-based steering is used.

### Core Decision Model

```text
                    TRAFFIC
                       │
                       ▼
              Firewall Policy
                       │
                       ▼
              Application / IPS
                       │
                       ▼
                SD-WAN Rule
                       │
                       ▼
                  Strategy
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
      Manual        SLA-Based     Bandwidth-Based
        │              │              │
        ▼              ▼              ▼
     Priority      Quality/SLA     in/out/biband
                       │
                       ▼
                Selected Member
```

---

# 🛠️ Pre-Deployment Checklist

## SD-WAN Members

* [ ] All WAN interfaces are configured.
* [ ] Each member has the correct gateway.
* [ ] Member status is verified.
* [ ] Member priority is understood.
* [ ] SD-WAN zones are correctly assigned.
* [ ] No unintended interface is included in the zone.

## SLA

* [ ] SLA health checks exist.
* [ ] Probe targets are reachable.
* [ ] Latency thresholds are correct.
* [ ] Packet-loss thresholds are correct.
* [ ] Jitter thresholds are correct where required.
* [ ] SLA status is verified before troubleshooting strategy behavior.

## Routing

* [ ] Routing table contains the expected destination.
* [ ] More-specific routes have been considered.
* [ ] FIB resolution is understood.
* [ ] Policy routes are checked when applicable.
* [ ] Recursive routing is not unexpectedly changing the egress path.

## Firewall Policy

* [ ] Source interface is correct.
* [ ] Destination interface/zone is correct.
* [ ] Source address is correct.
* [ ] Destination address is correct.
* [ ] Required services are allowed.
* [ ] Application Control is correctly configured if needed.
* [ ] Security profiles are not unintentionally affecting application identification.

---

# 🎛️ Manual Strategy

Manual mode provides administrator-controlled SD-WAN path selection.

Example:

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

## Manual Strategy Checklist

* [ ] `mode manual` is intentionally configured.
* [ ] Preferred member is explicitly selected.
* [ ] Member ordering is reviewed.
* [ ] Failover behavior is understood.
* [ ] `hold-down-time` is configured when required.
* [ ] Return-to-primary behavior has been tested.

### Hold-Down Behavior

```text
Primary
   │
   │ Failure
   ▼
Secondary
   │
   │ Primary recovers
   ▼
Hold-down timer
   │
   │ 60 seconds
   ▼
Primary
```

### Verify

* [ ] Primary failure causes the expected failover.
* [ ] Primary recovery does not immediately cause undesirable flapping.
* [ ] Hold-down timer matches operational requirements.

> [!WARNING]
> Manual mode should not be treated as equivalent to SLA-driven automatic path optimization.

---

# 🟢 Best Quality Strategy

Best Quality uses SLA measurements to select the preferred eligible path.

Typical quality metrics include:

```text
Latency
Packet Loss
Jitter
```

### Decision Model

```text
Traffic
   │
   ▼
SD-WAN Rule
   │
   ▼
SLA Evaluation
   │
   ├── Latency
   ├── Packet Loss
   └── Jitter
        │
        ▼
Best Qualified Path
```

## Checklist

* [ ] SLA health check is configured.
* [ ] Correct SLA target is used.
* [ ] Required members participate in the SLA.
* [ ] Quality metric matches the application requirement.
* [ ] Member availability is verified.
* [ ] Failover behavior is tested.
* [ ] Tie-break behavior is understood.

### Example

```text
ISP-1
Latency: 20 ms
Loss:     1%
Jitter:   5 ms

ISP-2
Latency: 40 ms
Loss:     0%
Jitter:   8 ms

ISP-3
Latency: 80 ms
Loss:     3%
Jitter:  20 ms

        ↓

SLA / Quality Evaluation
        ↓
Best Eligible Member
```

---

# 🎯 Minimum SLA Strategy

Minimum SLA focuses on selecting links that satisfy the required SLA conditions.

```text
ISP-1
   │
   ├── SLA FAIL
   │
   ▼
ISP-2
   │
   ├── SLA PASS
   │
   ▼
SELECT ISP-2
```

## Checklist

* [ ] SLA thresholds are defined.
* [ ] All candidate members are monitored.
* [ ] Failed members are correctly excluded.
* [ ] Healthy alternatives are available.
* [ ] SLA recovery has been tested.
* [ ] Failover behavior has been validated.

### Important Concept

Do not assume:

```text
ISP-1 fails → always ISP-2
```

Instead:

```text
Preferred Member
       │
       ▼
SLA Evaluation
       │
       ▼
Other Eligible Members
       │
       ▼
Valid Path Selection
```

This avoids blindly moving traffic to another unhealthy link.

---

# 🚀 Maximize Bandwidth Strategy

Bandwidth-oriented steering attempts to use available link capacity efficiently.

Example global configuration:

```bash
config system sdwan
    set load-balance-mode source-ip-based
end
```

A service can use load-balancing behavior:

```bash
config system sdwan
    config service
        edit 1
            set mode load-balance
        end
    end
end
```

## Checklist

* [ ] Bandwidth-based strategy is actually required.
* [ ] ISP bandwidth values are accurate.
* [ ] Upstream bandwidth is known.
* [ ] Downstream bandwidth is known.
* [ ] Speed Test measurements have been considered.
* [ ] Hash behavior is understood.
* [ ] ADVPN compatibility has been reviewed.
* [ ] Gateway/policy-route requirements have been reviewed.

> [!WARNING]
> Review platform and FortiOS version behavior before using bandwidth-oriented SD-WAN features in production.

---

# 🔀 Load Balance Strategy

Load balancing distributes traffic across eligible SD-WAN members.

Possible mechanisms include:

```text
Round-Robin
Source-IP-Based
Source-Destination-IP-Based
Inbandwidth
Outbandwidth
Bibandwidth
```

## Checklist

* [ ] Load-balance mode is intentionally selected.
* [ ] Hash behavior is understood.
* [ ] Session persistence requirements are considered.
* [ ] Link capacity differences are considered.
* [ ] Application sensitivity is considered.
* [ ] Failure behavior is tested.

---

# 🔄 SD-WAN Hash Modes

| Mode                   | Primary Behavior                     | Use Case                |
| ---------------------- | ------------------------------------ | ----------------------- |
| `round-robin`          | Cyclic distribution                  | General distribution    |
| `source-ip-based`      | Same source tends to same member     | Client/path consistency |
| `source-dest-ip-based` | Source + destination based selection | Flow-pair consistency   |
| `inbandwidth`          | Considers downstream capacity        | Download-heavy traffic  |
| `outbandwidth`         | Considers upstream capacity          | Upload-heavy traffic    |
| `bibandwidth`          | Considers both directions            | Bidirectional traffic   |

---

# 🔄 Round-Robin Checklist

```text
Traffic
   │
   ├── Flow-1 → ISP-1
   ├── Flow-2 → ISP-2
   ├── Flow-3 → ISP-3
   ├── Flow-4 → ISP-1
   └── Flow-5 → ISP-2
```

* [ ] Equal/cyclic distribution is acceptable.
* [ ] Session behavior is understood.
* [ ] Unequal WAN capacity has been considered.
* [ ] Application persistence requirements have been reviewed.

---

# 🧑 Source-IP-Based Checklist

```text
Source IP
    │
    ▼
Hash
    │
    ▼
SD-WAN Member
```

Example:

```text
Client-A → ISP-1
Client-B → ISP-2
Client-C → ISP-3
```

* [ ] Client consistency is required.
* [ ] NAT behavior has been considered.
* [ ] Source IP distribution is reasonably balanced.
* [ ] A single source is not expected to utilize every WAN link simultaneously.

---

# 🎯 Source-Destination-IP-Based Checklist

```text
Source IP
    +
Destination IP
    │
    ▼
Hash
    │
    ▼
SD-WAN Member
```

Example:

```text
10.10.10.10 → 8.8.8.8 → ISP-1

10.10.10.10 → 1.1.1.1 → ISP-2
```

* [ ] Per-destination path consistency is required.
* [ ] Multiple destinations should be distributed.
* [ ] Hash behavior is understood.
* [ ] Return-path requirements are considered.

---

# 📥 Inbandwidth

Inbandwidth focuses on downstream/incoming capacity.

```text
                 FortiGate
                /    |    \
              ISP-1 ISP-2 ISP-3
               20    80    40 Mbps
                     ▲
                     │
                  Preferred
```

Checklist:

* [ ] Downstream bandwidth is measured.
* [ ] Estimated downstream bandwidth is accurate.
* [ ] ISP asymmetry is understood.
* [ ] The application is download-heavy.

---

# 📤 Outbandwidth

Outbandwidth focuses on upstream/outgoing capacity.

```text
FortiGate
    │
    ▼
Compare upstream capacity
    │
    ▼
Select suitable member
```

Checklist:

* [ ] Upstream bandwidth is measured.
* [ ] Estimated upstream bandwidth is accurate.
* [ ] Application upload requirements are understood.

---

# ↕️ Bibandwidth

Bibandwidth considers both directions.

```text
Downstream
     +
Upstream
     │
     ▼
Bandwidth Evaluation
     │
     ▼
SD-WAN Member
```

Checklist:

* [ ] Both directions matter.
* [ ] Upstream values are accurate.
* [ ] Downstream values are accurate.
* [ ] ISP bandwidth asymmetry has been considered.

---

# 📊 Bandwidth Measurement

Configure estimated bandwidth on the WAN interface when required:

```bash
config system interface
    edit <interface>
        set estimated-upstream-bandwidth 100
        set estimated-downstream-bandwidth 200
    end
end
```

Values are specified in **Kbps**.

Example:

```text
ISP-1
├── Upstream:   100 Kbps
└── Downstream: 200 Kbps
```

## Verification Checklist

* [ ] Provider bandwidth has been documented.
* [ ] Actual measured bandwidth has been compared.
* [ ] `estimated-upstream-bandwidth` is correct.
* [ ] `estimated-downstream-bandwidth` is correct.
* [ ] Speed Test results have been considered.
* [ ] Bandwidth changes are reflected in configuration.

---

# ⚠️ Bandwidth Strategy Design Checks

Before deployment:

* [ ] Confirm FortiOS version compatibility.
* [ ] Confirm FortiGate model support.
* [ ] Confirm ADVPN compatibility.
* [ ] Review gateway behavior.
* [ ] Review policy-route requirements.
* [ ] Test asymmetric ISP links.
* [ ] Test link failure.
* [ ] Test link recovery.
* [ ] Test multiple simultaneous flows.

---

# 🧭 Interface vs Zone Priority

When an explicit preferred interface and a zone are both involved, verify which selection mechanism is controlling the decision.

Example:

```text
Preferred Interface:
    link-2

Zone:
    link-1
    link-2
```

Decision:

```text
Preferred Interface
        ↓
link-2
```

## Checklist

* [ ] Preferred interface is intentional.
* [ ] Zone membership is correct.
* [ ] Member ordering is reviewed.
* [ ] No unexpected member is preferred.
* [ ] Failover behavior has been tested.

---

# 🎯 Application-Based Steering

FortiGate can use application identification as part of SD-WAN traffic steering.

General architecture:

```text
Client
   │
   ▼
Firewall Policy
   │
   ▼
Application Identification
   │
   ▼
SD-WAN Rule
   │
   ▼
Strategy
   │
   ▼
SD-WAN Member
```

## Checklist

* [ ] Application Control is enabled where required.
* [ ] Application signatures can identify the traffic.
* [ ] IPS/security inspection is correctly configured where required.
* [ ] Firewall policy allows the traffic.
* [ ] SD-WAN rule matches the intended application.
* [ ] Application steering behavior is verified.

### Important Separation

```text
Firewall Policy
    ↓
Allow / Deny
Application Identification
Security Inspection

SD-WAN Rule
    ↓
Path Selection
```

> [!IMPORTANT]
> **Firewall policy determines whether traffic is allowed. SD-WAN determines which eligible path carries the traffic.**

---

# 🔍 Application Steering Verification

Use:

```bash
diagnose sys sdwan internet-service-app-ctrl-list 10
```

Checklist:

* [ ] Application is detected.
* [ ] Correct SD-WAN rule is matched.
* [ ] Expected member is selected.
* [ ] SLA state is healthy.
* [ ] No higher-priority rule is matching first.

---

# 🛡️ QoS and Traffic Shaping

SD-WAN path selection and traffic shaping are separate functions.

```text
SD-WAN
   ↓
Which path?
   ↓
WAN Member

Traffic Shaper
   ↓
How much bandwidth?
   ↓
Rate / Priority
```

Example:

```text
Name:
    shape-1

Type:
    Shared Shaper

Priority:
    High

Maximum Bandwidth:
    200 Kbps
```

## Traffic Shaping Policy Checklist

* [ ] Source is correct.
* [ ] Destination is correct.
* [ ] Service is correct.
* [ ] Outgoing interface/zone is correct.
* [ ] Shared shaper is configured.
* [ ] Reverse shaper is configured where required.
* [ ] Firewall policy allows the traffic.
* [ ] Shaper statistics are monitored.

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

Search for:

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

## QoS Checklist

* [ ] Session exists.
* [ ] Correct policy is matched.
* [ ] Shaper is visible in session information.
* [ ] Shaper statistics increase.
* [ ] Expected bandwidth limit is observed.
* [ ] Reverse direction is checked.
* [ ] SD-WAN member is correct.

---

# 🧭 Policy Route Verification

Use policy-route matching when forwarding behavior is unclear:

```bash
diagnose ip proute match <destination> <mac> <interface> <port> <protocol>
```

Example:

```bash
diagnose ip proute match 3.1.1.34 70:4c:a5:86:de:56 port3 22 6
```

## Checklist

* [ ] Destination is correct.
* [ ] Source/input interface is correct.
* [ ] MAC address is correct where required.
* [ ] Destination port is correct.
* [ ] IP protocol is correct.
* [ ] Matching policy route is identified.
* [ ] Result is compared with actual packet behavior.

---

# 🚨 SD-WAN Troubleshooting Checklist

## 1. Member Status

* [ ] Is the SD-WAN member up?
* [ ] Is the physical interface up?
* [ ] Is the gateway reachable?
* [ ] Is the member correctly assigned to the intended zone?

---

## 2. SLA

* [ ] Is the SLA health check up?
* [ ] Is the probe target reachable?
* [ ] Is packet loss acceptable?
* [ ] Is latency acceptable?
* [ ] Is jitter acceptable?
* [ ] Is the correct SLA attached to the rule?

---

## 3. Rule Matching

* [ ] Is the expected SD-WAN rule matched?
* [ ] Is another higher-priority rule matching first?
* [ ] Are source/destination objects correct?
* [ ] Is application matching correct?
* [ ] Is DSCP/TOS matching relevant where configured?

---

## 4. Strategy

Identify the active strategy:

```text
Manual
Best Quality
Minimum SLA
Maximize Bandwidth
Load Balance
```

Then verify:

* [ ] Strategy matches the design requirement.
* [ ] Strategy is supported by the intended topology.
* [ ] Strategy behavior is understood.
* [ ] Failover behavior has been tested.

---

## 5. Member Priority

* [ ] Member ordering is correct.
* [ ] Preferred interface is correct.
* [ ] Zone membership is correct.
* [ ] Tie-break behavior is understood.

---

## 6. Load Balancing

* [ ] Global load-balance mode is checked.
* [ ] Service mode is checked.
* [ ] Hash mode is understood.
* [ ] Source-IP behavior is checked.
* [ ] Source-destination behavior is checked.
* [ ] Bandwidth-based selection is checked where applicable.

> [!WARNING]
> Do not confuse the global `load-balance-mode` with the SD-WAN Service `mode`. They affect different parts of the SD-WAN decision process.

---

## 7. Routing

* [ ] Routing table is checked.
* [ ] FIB is checked.
* [ ] More-specific routes are identified.
* [ ] Policy routes are checked.
* [ ] Recursive routing is checked.
* [ ] Unexpected static routes are excluded.

---

## 8. Application Steering

* [ ] Application is detected.
* [ ] Firewall policy allows the flow.
* [ ] Application Control is working.
* [ ] SD-WAN application rule matches.
* [ ] Correct member is selected.

---

## 9. QoS

* [ ] Traffic shaper is configured correctly.
* [ ] Shared shaper is correct.
* [ ] Reverse shaper is correct.
* [ ] Session contains expected shaping information.
* [ ] Shaper statistics are increasing.

---

## 10. Packet-Level Verification

When required:

```text
Client
  ↓
Firewall
  ↓
SD-WAN Rule
  ↓
Selected Member
  ↓
WAN
```

Verify each stage independently.

---

# 🔬 SD-WAN Decision Troubleshooting Flow

```text
                    Traffic
                       │
                       ▼
              Firewall Policy
                       │
                  Allowed?
                  /       \
                NO         YES
                │           │
              STOP          ▼
                    Application Match
                           │
                           ▼
                      SD-WAN Rule
                           │
                           ▼
                       Strategy
                           │
            ┌──────────────┼──────────────┐
            ▼              ▼              ▼
          Manual         SLA          Bandwidth
            │              │              │
            └──────────────┼──────────────┘
                           ▼
                     Routing / FIB
                           │
                           ▼
                    Selected Member
                           │
                           ▼
                      Traffic Flow
```

---

# 📋 Strategy Selection Matrix

| Requirement                   | Recommended Strategy          |
| ----------------------------- | ----------------------------- |
| Administrator-controlled path | `Manual`                      |
| Best latency/jitter/loss      | `Best Quality`                |
| Only SLA-qualified links      | `Minimum SLA`                 |
| Capacity-aware selection      | `Maximize Bandwidth`          |
| General traffic distribution  | `Load Balance`                |
| Cyclic distribution           | `Round-Robin`                 |
| Client path consistency       | `Source-IP-Based`             |
| Flow-pair consistency         | `Source-Destination-IP-Based` |
| Prefer downstream capacity    | `Inbandwidth`                 |
| Prefer upstream capacity      | `Outbandwidth`                |
| Consider both directions      | `Bibandwidth`                 |
| Application-specific path     | Application-aware SD-WAN rule |
| Bandwidth enforcement         | Traffic Shaper                |

---

# 🧠 Strategy Selection Checklist

Before choosing a strategy, answer:

* [ ] Do I need administrator-controlled routing?
* [ ] Do I need SLA-based failover?
* [ ] Do I care about latency?
* [ ] Do I care about packet loss?
* [ ] Do I care about jitter?
* [ ] Do I need minimum SLA compliance?
* [ ] Do I need bandwidth-aware path selection?
* [ ] Are ISP links asymmetric?
* [ ] Do I need client/session stickiness?
* [ ] Do I need source-destination consistency?
* [ ] Do I need application-aware steering?
* [ ] Do I need bandwidth enforcement?
* [ ] Is ADVPN involved?
* [ ] Is policy routing involved?
* [ ] Are there more-specific routes affecting FIB selection?

---

# 🧩 Advanced SD-WAN Decision Model

```text
                         TRAFFIC
                            │
                            ▼
                   ┌─────────────────┐
                   │ Firewall Policy │
                   └────────┬────────┘
                            │
                            ▼
                  Application / Security
                            │
                            ▼
                   ┌─────────────────┐
                   │   SD-WAN Rule   │
                   └────────┬────────┘
                            │
                            ▼
                   ┌─────────────────┐
                   │    Strategy     │
                   └────────┬────────┘
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
          ▼                 ▼                 ▼
       Manual          SLA-Based       Bandwidth-Based
          │                 │                 │
          ▼                 ▼                 ▼
      Priority        Quality/SLA      in/out/biband
          │                 │                 │
          └─────────────────┼─────────────────┘
                            ▼
                     Routing / FIB
                            │
                            ▼
                     SD-WAN Member
                            │
                            ▼
                         Traffic
```

---

# ⚡ Quick CLI Reference

## Global Load Balancing

```bash
config system sdwan
    set load-balance-mode source-ip-based
end
```

## Manual SD-WAN Service

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

## Estimated Bandwidth

```bash
config system interface
    edit <interface>
        set estimated-upstream-bandwidth 100
        set estimated-downstream-bandwidth 200
    end
end
```

## Policy Route Match

```bash
diagnose ip proute match <destination> <mac> <interface> <port> <protocol>
```

## SD-WAN Service Status

```bash
diagnose sys sdwan service
```

## Application Steering

```bash
diagnose sys sdwan internet-service-app-ctrl-list 10
```

## IP Rope

```bash
diagnose firewall iprope list 10015
```

## Session Inspection

```bash
diagnose sys session list
```

## Traffic Shaper

```bash
diagnose firewall shaper traffic-shaper list
```

## SDN Status

```bash
diagnose sys sdn status
```

## SDN Debug

```bash
diagnose debug application azd -1
```

---

# 🚨 Production Validation Checklist

Before deploying an SD-WAN strategy:

### Architecture

* [ ] SD-WAN topology is documented.
* [ ] WAN members are documented.
* [ ] Zones are documented.
* [ ] Routing topology is documented.
* [ ] Application requirements are documented.

### Strategy

* [ ] Correct strategy is selected.
* [ ] Member priorities are correct.
* [ ] SLA requirements are defined.
* [ ] Tie-break behavior is understood.
* [ ] Load-balance behavior is understood.

### Failure Testing

* [ ] Primary WAN failure tested.
* [ ] Secondary WAN failure tested.
* [ ] SLA degradation tested.
* [ ] Packet-loss condition tested.
* [ ] High-latency condition tested.
* [ ] High-jitter condition tested.
* [ ] WAN recovery tested.
* [ ] Session behavior after recovery tested.

### Application Testing

* [ ] Critical applications are identified.
* [ ] Application steering is verified.
* [ ] Correct SD-WAN member is selected.
* [ ] Security policy remains correct.

### QoS

* [ ] Traffic shaper is tested.
* [ ] Bandwidth limit is validated.
* [ ] Reverse shaping is validated.
* [ ] Session statistics are checked.

---

# 🧠 Key Takeaways

> [!IMPORTANT]
> **SLA measures path quality; the SD-WAN strategy determines how that information is used.**

> [!IMPORTANT]
> **Manual = administrator-controlled path selection.**

> [!IMPORTANT]
> **Best Quality = quality-driven path selection.**

> [!IMPORTANT]
> **Minimum SLA = select paths that satisfy the required SLA conditions.**

> [!IMPORTANT]
> **Maximize Bandwidth = capacity-oriented traffic distribution.**

> [!IMPORTANT]
> **Load Balance = distribute traffic according to the configured load-balancing/hash mechanism.**

> [!IMPORTANT]
> **Firewall Policy = security/access control.**

> [!IMPORTANT]
> **SD-WAN Rule = path selection.**

> [!IMPORTANT]
> **Traffic Shaper = bandwidth enforcement.**

---

# 🧠 Final Mental Model

```text
                 ┌───────────────────┐
                 │      TRAFFIC      │
                 └─────────┬─────────┘
                           │
                           ▼
                 ┌───────────────────┐
                 │ Firewall Policy   │
                 │ Allow / Deny     │
                 │ App Identification│
                 └─────────┬─────────┘
                           │
                           ▼
                 ┌───────────────────┐
                 │    SD-WAN Rule    │
                 │                   │
                 │ Source            │
                 │ Destination       │
                 │ Application       │
                 │ DSCP / TOS        │
                 └─────────┬─────────┘
                           │
                           ▼
                 ┌───────────────────┐
                 │     Strategy      │
                 └─────────┬─────────┘
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
       Manual           SLA-Based      Bandwidth-Based
          │                │                │
          ▼                ▼                ▼
      Priority        Quality/SLA      in/out/biband
                           │
                           ▼
                    Routing / FIB
                           │
                           ▼
                    SD-WAN Member
                           │
                           ▼
                     Traffic Shaper
                           │
                           ▼
                    WAN / Internet
```

## 🔑 One-Line Memory

```text
Firewall Policy
    = Can this traffic pass?

SD-WAN Rule
    = Which path should carry it?

SLA
    = How healthy is the path?

Strategy
    = How should FortiGate choose?

Routing / FIB
    = Which forwarding paths are actually available?

Traffic Shaper
    = How much bandwidth can the traffic consume?
```

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

## 🏷️ Topics

`FortiGate` · `FortiOS` · `SD-WAN` · `SD-WAN Strategy` · `Best Quality` · `Minimum SLA` · `Maximize Bandwidth` · `Load Balance` · `SLA` · `Traffic Steering` · `Application Steering` · `QoS` · `Traffic Shaping` · `Network Security` · `Fortinet`

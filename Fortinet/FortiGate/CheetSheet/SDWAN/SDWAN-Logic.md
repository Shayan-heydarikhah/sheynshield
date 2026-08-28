# FortiGate SD-WAN — Cheat Sheet

> **Scope:** SD-WAN, link monitoring, SLA, load balancing, passive health measurement, speed test, routing integration, and troubleshooting
> **Platform:** FortiOS
> **Note:** Command syntax and available options can vary by FortiOS version.

---

## 1. SD-WAN — Core Concept

SD-WAN can:

* Merge multiple WAN links into a single logical SD-WAN interface/zone.
* Build overlays such as:

  * IPsec VPN
  * Other VPN/overlay technologies
* Monitor WAN quality.
* Select the best path based on:

  * Latency
  * Jitter
  * Packet loss
  * Bandwidth
  * Link cost
  * Application/service requirements

```text
                ┌──────────────┐
                │   FortiGate  │
                │    SD-WAN    │
                └──────┬───────┘
                       │
             ┌─────────┴─────────┐
             │                   │
          ISP-1                ISP-2
          Link-1               Link-2
             │                   │
             └─────────┬─────────┘
                       │
                 WAN / Internet
```

---

# 2. SD-WAN Link Quality

## Active Probe

SD-WAN actively generates probe traffic toward configured servers.

```text
FortiGate
   │
   ├── Probe ──> SLA Server 1
   ├── Probe ──> SLA Server 2
   └── Probe ──> SLA Server 3
```

Useful for:

* Continuous link-quality measurement
* Links with little/no user traffic
* Detecting degraded WAN paths

---

## Passive Probe

Passive measurement uses normal client traffic.

```text
Client Traffic
      │
      ▼
 FortiGate
      │
      ├── Measure RTT
      ├── Measure loss
      ├── Measure jitter
      └── Calculate link quality
```

### Important

Passive measurement requires traffic to exist.

If there is no traffic, there may be little/no measurement data.

---

## Prefer Passive

`prefer-passive` behaves primarily like passive measurement but can send probes when there is insufficient new traffic.

```text
ISP-1
  │
  ├── Active traffic
  └── Active measurement

ISP-2
  │
  ├── Standby/passive
  └── Probe when required
```

This can reduce unnecessary probing while still allowing inactive links to be monitored.

---

# 3. WAN Interface Administrative Distance

When WAN interfaces obtain their gateway using DHCP:

```text
ISP-1
0.0.0.0/0 → Gateway
AD 10

ISP-2
0.0.0.0/0 → Gateway
AD 10
```

Using the same AD can be useful when the design requires ECMP/load balancing.

Example:

```text
ISP-1 → AD 10
ISP-2 → AD 10
```

> Avoid arbitrary AD differences when you expect both links to participate equally in SD-WAN/ECMP.

---

# 4. SD-WAN References / Configuration Hygiene

Before building SD-WAN policies:

* Check SD-WAN members.
* Check zones.
* Check health checks.
* Check service rules.
* Check static routes.
* Check gateway reachability.
* Clear/remove unnecessary references from SD-WAN contributor components.

---

# 5. SD-WAN Service Rules

SD-WAN service rules determine **which traffic uses which member**.

Basic structure:

```text
config system sdwan
    config service
        edit 1
            ...
        next
    end
end
```

---

## Preferred Interface

Example concept:

```text
Priority:

1. ISP-1
2. ISP-2
```

This means:

```text
Try ISP-1
   │
   ├── Healthy → use ISP-1
   │
   └── Failed → use ISP-2
```

---

## SD-WAN Zone

Instead of directly binding a rule to one interface:

```text
SD-WAN Zone
   ├── link-1
   ├── link-2
   └── link-3
```

The SD-WAN engine can dynamically select the appropriate member.

---

# 6. Session Load-Balancing

Example:

```cli
config system sdwan
    config service
        edit 1
            set mode load-balance
            set hash-mode round-robin
        next
    end
end
```

### Hash Modes

| Mode                   | Concept                          |
| ---------------------- | -------------------------------- |
| `round-robin`          | Distribute sessions sequentially |
| `source-ip-based`      | Hash based on source IP          |
| `source-dest-ip-based` | Hash source + destination IP     |
| `inbandwidth`          | Consider inbound bandwidth       |
| `outbandwidth`         | Consider outbound bandwidth      |
| `bibandwidth`          | Consider both directions         |

---

# 7. SD-WAN Global Load-Balance Modes

```cli
config system sdwan
    set load-balance-mode source-ip-based
end
```

## Source IP

```text
Client A → ISP-1
Client B → ISP-2
Client C → ISP-1
Client D → ISP-2
```

Traffic is distributed using a source-IP hash.

**Recommended general-purpose mode.**

---

## Session / Weight-Based

```text
ISP-1 → weight 50
ISP-2 → weight 50
```

Traffic is distributed according to session ratios.

Example:

```text
ISP-1 → 50%
ISP-2 → 50%
```

---

## Source-Destination IP

Hash is based on:

```text
Source IP + Destination IP
```

Therefore:

```text
192.168.10.10 → 8.8.8.8 → ISP-1

192.168.10.10 → 1.1.1.1 → ISP-2
```

---

## Volume / Measured Volume

Traffic distribution is based on measured bandwidth/volume.

Useful when:

```text
ISP-1 = 100 Mbps
ISP-2 = 500 Mbps
```

and the goal is to distribute traffic according to capacity.

---

## Spillover / Usage-Based

The preferred link is used until its configured bandwidth threshold is reached.

```text
                 ISP-1
                   │
Traffic ──────────>│
                   │
             Threshold reached
                   │
                   ▼
                 ISP-2
```

Useful when one link should be preferred until it becomes congested.

---

# 8. SD-WAN SLA / Link Monitoring

## Basic SLA

```text
SD-WAN
   │
   ├── ISP-1 ──> SLA Server
   │
   └── ISP-2 ──> SLA Server
```

Typical measurements:

* Latency
* Jitter
* Packet loss

Example:

```text
Latency      < threshold
Jitter       < threshold
Packet Loss  < threshold
```

If the SLA fails:

```text
ISP-1
  │
  └── SLA FAIL
        │
        ▼
      ISP-2
```

---

## SLA Target

Typical behavior:

```text
Latency ↑
Jitter ↑
Packet Loss ↑
       │
       ▼
SLA failure
       │
       ▼
Select another member
```

---

# 9. SLA Health-Check Interval

Important parameters:

```text
Check interval
Fail count
Recovery/pass count
```

Conceptually:

```text
Probe
  │
  ├── Success
  ├── Success
  ├── Failure
  ├── Failure
  └── Failure
          │
          ▼
       Link FAIL
```

After recovery:

```text
Probe
  │
  ├── Success
  ├── Success
  └── Success
          │
          ▼
       Link UP
```

---

# 10. SLA Protocols

## Ping

Simple ICMP-based reachability and latency test.

```text
FortiGate ── ICMP ──> Server
```

---

## TCP Connect

Measures TCP connection establishment.

### Half-Open

Conceptually:

```text
SYN
 │
 ▼
SYN-ACK
```

Measures the time associated with TCP connection establishment.

Default behavior for TCP connect health checking.

### Half-Close

Uses:

```text
FIN
 │
 ▼
FIN-ACK
```

Useful when measuring the close-side interaction.

Example:

```cli
config system sdwan
    config health-check
        edit tcpconnect
            set protocol tcp-connect
            set port 443
            set quality-measured-method half-close
            set server 192.168.20.200
        next
    end
end
```

---

# 11. FTP Health Check

FTP can operate using:

```text
Passive
Port / Active
```

Example:

```cli
config system sdwan
    config health-check
        edit ftp1
            set protocol ftp
            set port 21
            set user root
            set password 123
            set ftp-mode passive
            set ftp-file test_ftp.txt
            set server 192.168.20.200
        next
    end
end
```

---

# 12. DNS Health Check

DNS health checking sends a DNS request and validates the response.

Example:

```cli
config system sdwan
    config health-check
        edit dns1
            set protocol dns
            set dns-request-domain example.com
            set dns-match-ip 1.2.3.4
        next
    end
end
```

> DNS is generally less useful when the goal is pure WAN quality measurement.

---

# 13. TCP Echo / UDP Echo

Example:

```cli
config system sdwan
    config health-check
        edit echo-test
            set protocol udp-echo
            set port 7
            set server 192.168.20.200
        next
    end
end
```

---

# 14. TWAMP / OWAMP

### TWAMP

**Two-Way Active Measurement Protocol**

Measures two-way performance.

```text
FortiGate ───────> TWAMP Server
FortiGate <─────── TWAMP Server
```

### OWAMP

**One-Way Active Measurement Protocol**

Measures one-way characteristics.

---

# 15. HTTP Health Check

HTTP can test:

```text
TCP connection
HTTP GET
Expected response
```

Concept:

```text
FortiGate
   │
   │ HTTP GET /health
   ▼
Web Server
   │
   │ "OK"
   ▼
FortiGate
```

Useful parameters include:

* Port
* HTTP GET URL
* Expected response string

---

# 16. SLA Thresholds

Example structure:

```cli
config system sdwan
    config health-check
        edit 1
            set threshold-warning-packetloss ...
            set threshold-alert-packetloss ...

            set threshold-warning-latency ...
            set threshold-alert-latency ...

            set threshold-warning-jitter ...
            set threshold-alert-jitter ...
        next
    end
end
```

### Measurement Window

Typical interpretation:

```text
Latency / Jitter
→ recent probe calculation

Packet Loss
→ recent probe history
```

Avoid unnecessarily aggressive thresholds because they can create path flapping.

---

# 17. Cascade Interface Update

```cli
config system sdwan
    config health-check
        edit 1
            set update-cascade-interface enable
        next
    end
end
```

Useful when SD-WAN interfaces/members are grouped and health state needs to propagate.

---

# 18. SLA Logging

```cli
config system sdwan
    config health-check
        edit 1
            set sla-fail-log-period 60
            set sla-pass-log-period 60
        next
    end
end
```

Concept:

```text
SLA FAIL
   │
   └── log event

SLA PASS
   │
   └── log recovery
```

---

# 19. Link Cost Factor

When multiple SLA metrics exist, link cost factor determines which measurements are important.

Example:

```cli
config system sdwan
    config health-check
        edit 1
            config sla
                set link-cost-factor latency packet-loss
            end
        next
    end
end
```

Possible factors include:

```text
Latency
Packet Loss
Jitter
```

---

# 20. MOS — Mean Opinion Score

MOS represents perceived voice quality.

| MOS | Quality   |
| --: | --------- |
|   5 | Excellent |
|   4 | Good      |
|   3 | Fair      |
|   2 | Poor      |
|   1 | Bad       |

Approximate codec examples:

| Codec | Approx. MOS |
| ----- | ----------: |
| G.711 |        ~4.2 |
| G.729 |        ~3.9 |
| G.722 |        ~4.5 |

Example:

```cli
config system sdwan
    config health-check
        edit 1
            set mos-codec g711

            config sla
                edit 1
                    set mos-threshold 3.6
                next
            end
        next
    end
end
```

> MOS is particularly useful for VoIP-sensitive traffic.

---

# 21. Passive WAN Health Measurement

Enable measurement on the firewall policy:

```cli
config firewall policy
    edit 1
        set dstintf sdwan
        set passive-wan-health-measurement enable
    next
end
```

Useful diagnostic:

```cli
diagnose sys link-monitor-passive interface port4
```

---

# 22. Passive Measurement + SD-WAN Service

Example:

```cli
config system sdwan
    set status enable

    config service
        edit 1
            set mode priority
            set src all

            set internet-service enable
            set internet-service-name Alibaba-Web Amazon-Web

            set health-check Passive_HC
            set priority-members 1 2

            set passive-measurement enable
        next
    end
end
```

Policy:

```cli
config firewall policy
    edit 1
        set dstintf sdwan
        set passive-wan-health-measurement enable

        set auto-asic-offload disable
    next
end
```

### Important

Passive WAN measurement can require ASIC offloading to be disabled depending on the measurement design/version.

---

# 23. Passive vs Active vs Prefer-Passive

| Mode           | Behavior                                  |
| -------------- | ----------------------------------------- |
| Active         | Continuously sends probes                 |
| Passive        | Uses real traffic measurements            |
| Prefer Passive | Uses traffic first, probes when necessary |

### Quick Decision

```text
Need continuous link testing?
        │
        └── Active

Have enough real traffic?
        │
        └── Passive

Want passive behavior but still
monitor inactive links?
        │
        └── Prefer Passive
```

---

# 24. DSCP in SLA

SD-WAN health-check traffic can use DSCP.

```cli
config system sdwan
    config health-check
        edit 1
            set diffservcode 111110
        next
    end
end
```

Useful for testing QoS treatment across WAN links.

---

# 25. SD-WAN + IPsec

SD-WAN can carry VPN overlays across multiple WAN links.

```text
                 ┌──────── ISP-1 ────────┐
                 │                       │
FortiGate ─ SDWAN│                       │ SDWAN ─ FortiGate
                 │                       │
                 └──────── ISP-2 ────────┘
                          │
                       IPsec
```

Possible architecture:

```text
SD-WAN
  │
  ├── ISP-1
  ├── ISP-2
  └── ISP-3
       │
       ▼
     IPsec
       │
       ▼
    Remote Site
```

> When running IPsec over SD-WAN, the SD-WAN layer handles WAN path selection while IPsec provides encryption.

---

# 26. SD-WAN + Routing

SD-WAN can interact with:

* Static routing
* OSPF
* BGP
* Route tags
* ECMP

Useful concept:

```text
Routing
   │
   ▼
SD-WAN
   │
   ├── Link-1
   ├── Link-2
   └── Link-3
```

---

# 27. Route Tag

Route tags can be used to influence SD-WAN decisions based on routing information.

Useful in:

```text
BGP
OSPF
SD-WAN
```

---

# 28. SD-WAN Default + Gateway

Example:

```cli
config system sdwan
    config service
        edit 1
            set default enable
            set gateway enable
        next
    end
end
```

## `default enable`

Allows matching SD-WAN traffic to use the SD-WAN member without relying on the normal FIB destination lookup behavior in the same way as the default behavior.

## `gateway enable`

Uses the configured gateway of the selected SD-WAN member.

Concept:

```text
Client
  │
  ▼
SD-WAN Rule
  │
  ├── Member-1
  │     └── Gateway-1
  │
  └── Member-2
        └── Gateway-2
```

> SD-WAN members should have valid gateways when using gateway-based forwarding.

---

# 29. FIB / RIB Considerations

FortiGate evaluates routing information when determining whether SD-WAN policy routing can be applied.

Conceptually:

```text
Destination
    │
    ▼
FIB lookup
    │
    ├── SD-WAN member?
    │
    └── Valid route?
```

With appropriate `default` / `gateway` settings, SD-WAN can alter how these checks are handled.

### Troubleshooting checklist

```text
✓ Destination route exists
✓ SD-WAN member is valid
✓ Gateway is reachable
✓ SD-WAN rule matches
✓ Firewall policy matches
✓ SLA is healthy
```

---

# 30. ECMP + SD-WAN

When multiple equal-cost default routes exist:

```text
0.0.0.0/0 → ISP-1
0.0.0.0/0 → ISP-2
```

FortiOS can use ECMP.

Example concept:

```text
              ┌── ISP-1
Traffic ──────┤
              └── ISP-2
```

---

# 31. SD-WAN Selection Modes — Quick Table

| Mode                 | Main Logic                | Typical Use                 |
| -------------------- | ------------------------- | --------------------------- |
| Source IP            | Source hash               | General load balancing      |
| Session              | Session ratio/weight      | Session distribution        |
| Source + Destination | Src/Dst hash              | Stable flow distribution    |
| Spillover            | Bandwidth threshold       | Preferred link + overflow   |
| Volume               | Measured bandwidth/volume | Capacity-based distribution |
| Priority             | Preferred member order    | Primary/backup              |
| SLA                  | Quality threshold         | Quality-sensitive traffic   |

---

# 32. SD-WAN Rule — Negated Source

CLI can be used when a source needs to be excluded.

Concept:

```cli
set src <address>
set src-negate enable
```

Meaning:

```text
Match everything EXCEPT <address>
```

---

# 33. Preferred Interface / Zone

Example:

```text
SD-WAN Zone
 ├── link-1
 └── link-2
```

If:

```text
Preferred = link-2
```

then the rule attempts:

```text
link-2
  │
  ├── Healthy → use link-2
  │
  └── Failed → evaluate another member
```

---

# 34. Link Failover Example

```text
             SLA
              │
              ▼
       ┌───────────────┐
       │    link-1     │
       │    PRIMARY    │
       └───────┬───────┘
               │
          SLA failure
               │
               ▼
       ┌───────────────┐
       │    link-2     │
       │   BACKUP      │
       └───────────────┘
```

---

# 35. SD-WAN Diagnostic Commands

## Health Check

```cli
diagnose sys sdwan health-check
```

---

## SD-WAN Zone

```cli
diagnose sys sdwan zone
```

---

## Route Tags

```cli
diagnose sys sdwan route-tag-list
```

---

## Neighbors

```cli
diagnose sys sdwan neighbor
```

---

## Service

```cli
diagnose sys sdwan service 2
```

---

## SLA Log

```cli
diagnose sys sdwan sla-log slatest 1
```

---

## Interface SLA Log

```cli
diagnose sys sdwan sla-log port1
```

---

# 36. Speed Test

Speed test can measure WAN performance using approved test infrastructure.

Concept:

```text
FortiGate
   │
   │ Speed Test
   ▼
Fortinet / Approved Test Server
   │
   ├── Download
   ├── Upload
   └── Measurements
```

### Requirements

* Appropriate license
* SD-WAN Network Monitor capability
* Speed-test administrative access on WAN interface
* FortiGuard connectivity for approved test servers
* Correct WAN role
* Estimated bandwidth configured

---

# 37. Speed-Test Schedule

Example:

```cli
config system speed-test-schedule
    edit port1
        set schedules 10 14
        set update-inbandwidth enable
        set update-outbandwidth enable

        set update-inbandwidth-maximum 60000
        set update-inbandwidth-minimum 10000

        set update-outbandwidth-maximum 50000
        set update-outbandwidth-minimum 10000
    next
end
```

Schedules:

```cli
config firewall schedule recurring
    edit 10
        set start 10:00
        set end 12:00
        set day monday tuesday wednesday thursday friday
    next

    edit 14
        set start 14:00
        set end 16:00
        set day monday tuesday wednesday thursday friday
    next
end
```

---

# 38. Measured Bandwidth

Example:

```cli
config system interface
    edit isp
        get | grep meas
    end
end
```

Possible output:

```text
measured-upstream-bandwidth:   23691
measured-downstream-bandwidth: 48862
bandwidth-measure-time:        Wed Jan 27 14:00:39 2021
```

Use measured values when designing:

* Volume-based load balancing
* Spillover
* Bandwidth-aware SD-WAN rules

---

# 39. Speed-Test Design

```text
                 FortiGate
                    │
             ┌──────┴──────┐
             │             │
           ISP-1         ISP-2
             │             │
             ▼             ▼
        Speed Test     Speed Test
          Server          Server
```

Recommended:

* Use realistic test servers.
* Avoid testing too frequently.
* Configure bandwidth limits appropriately.
* Consider business-hour impact.

---

# 40. SD-WAN Troubleshooting Workflow

```text
                    START
                      │
                      ▼
             Check interface status
                      │
                      ▼
               Check gateway
                      │
                      ▼
              Check SD-WAN member
                      │
                      ▼
              Check health-check
                      │
                      ▼
                 Check SLA
                      │
                      ▼
               Check SD-WAN rule
                      │
                      ▼
               Check route / FIB
                      │
                      ▼
              Check firewall policy
                      │
                      ▼
              Check session path
                      │
                      ▼
                  Capture
```

---

# 41. SD-WAN Troubleshooting Checklist

## Layer 1/2

```text
[ ] Interface UP
[ ] Link speed correct
[ ] Duplex correct
[ ] Errors/drops checked
```

## Layer 3

```text
[ ] IP address correct
[ ] Gateway reachable
[ ] ARP/neighbor valid
[ ] Default route exists
```

## SD-WAN

```text
[ ] Member enabled
[ ] Member belongs to correct zone
[ ] Health-check configured
[ ] SLA target correct
[ ] SLA server reachable
[ ] Thresholds reasonable
[ ] Rule matches traffic
```

## Firewall

```text
[ ] Correct source
[ ] Correct destination
[ ] Correct outgoing interface
[ ] NAT requirement verified
[ ] Policy order verified
```

## Routing

```text
[ ] RIB route exists
[ ] FIB route exists
[ ] Gateway reachable
[ ] ECMP behavior understood
[ ] Route tag verified
```

---

# 42. SLA Troubleshooting

```cli
diagnose sys sdwan health-check
```

Check:

```text
Member
Status
Latency
Jitter
Packet loss
SLA state
```

If SLA is unexpectedly failing:

```text
1. Ping the SLA server
2. Check routing
3. Check packet loss
4. Check latency
5. Check jitter
6. Verify threshold
7. Verify probe protocol
8. Verify firewall policy
9. Verify server availability
```

---

# 43. Avoid Bad SLA Targets

Do not blindly use:

```text
ISP Gateway
```

as the only SLA target.

Prefer stable, meaningful destinations.

```text
FortiGate
   │
   ▼
ISP
   │
   ▼
Reliable SLA Target
```

Use more than one target where appropriate.

---

# 44. Application / Service-Based SD-WAN

SD-WAN rules can classify traffic by:

```text
Source
Destination
Internet Service
Application
Protocol
Port
Route Tag
SLA
```

Example:

```text
VoIP
  │
  └── SLA / MOS
       │
       └── Best-quality link

Bulk Traffic
  │
  └── Spillover / Volume

General Internet
  │
  └── Load balance
```

---

# 45. Passive Measurement + Internet Service

Concept:

```text
Client
  │
  ▼
Internet Service
  │
  ▼
SD-WAN Rule
  │
  ├── Passive measurement
  └── SLA decision
```

Useful for application-specific path selection.

---

# 46. Security Considerations

## Do not blindly use:

```text
src = all
dst = all
service = all
```

for every SD-WAN policy.

Prefer:

```text
Specific source
Specific destination
Specific application/service
Specific SLA
```

Especially for:

* Internet access
* Remote users
* Sensitive applications
* Business-critical services

---

# 47. SD-WAN + Passive Health Measurement

Important interaction:

```text
Firewall Policy
      │
      └── passive-wan-health-measurement
                    │
                    ▼
              SD-WAN Engine
                    │
                    ▼
              Health Metrics
```

When using passive health measurement, review ASIC offloading requirements for the specific FortiOS release/design.

---

# 48. Quick Configuration Skeleton

```cli
config system sdwan
    set status enable

    config members
        edit 1
            set interface "wan1"
            set gateway 1.1.1.1
        next

        edit 2
            set interface "wan2"
            set gateway 2.2.2.1
        next
    end

    config health-check
        edit "internet-sla"
            set server "8.8.8.8"
            set protocol ping
            set members 1 2
        next
    end

    config service
        edit 1
            set mode priority
            set priority-members 1 2
            set health-check "internet-sla"
        next
    end
end
```

---

# 49. Quick Reference — Most Important Commands

| Purpose              | Command                                                   |
| -------------------- | --------------------------------------------------------- |
| SD-WAN health        | `diagnose sys sdwan health-check`                         |
| SD-WAN zones         | `diagnose sys sdwan zone`                                 |
| SD-WAN neighbors     | `diagnose sys sdwan neighbor`                             |
| Route tags           | `diagnose sys sdwan route-tag-list`                       |
| SD-WAN service       | `diagnose sys sdwan service <id>`                         |
| SLA logs             | `diagnose sys sdwan sla-log slatest <id>`                 |
| Interface SLA logs   | `diagnose sys sdwan sla-log <interface>`                  |
| Passive monitoring   | `diagnose sys link-monitor-passive interface <interface>` |
| SD-WAN configuration | `config system sdwan`                                     |
| Health-check         | `config system sdwan -> config health-check`              |
| SD-WAN service       | `config system sdwan -> config service`                   |

---

# 50. Design Decision Matrix

| Requirement                        | Recommended Approach            |
| ---------------------------------- | ------------------------------- |
| Primary + backup ISP               | Priority                        |
| Equal general traffic distribution | Source-IP based                 |
| Stable Src/Dst flows               | Source-Destination IP           |
| Different ISP capacities           | Volume                          |
| Preferred ISP until congestion     | Spillover                       |
| Application quality                | SLA                             |
| VoIP                               | SLA + MOS                       |
| Links with no traffic              | Active                          |
| Heavy normal traffic               | Passive                         |
| Passive + inactive-link awareness  | Prefer Passive                  |
| Multiple WAN + VPN                 | SD-WAN + IPsec                  |
| Dynamic WAN performance            | SLA + health-check              |
| Bandwidth-aware policy             | Speed Test / measured bandwidth |
| BGP integration                    | Route tags + SD-WAN             |
| OSPF integration                   | Routing + SD-WAN                |
| Critical application               | SLA-based rule                  |

---

# 51. Final Troubleshooting Mental Model

```text
                    TRAFFIC
                       │
                       ▼
                Firewall Policy
                       │
                       ▼
                  SD-WAN Rule
                       │
             ┌─────────┴─────────┐
             │                   │
          Matching            No Match
             │                   │
             ▼                   ▼
       SLA / Strategy         Routing/FIB
             │
       ┌─────┴─────┐
       │           │
    Healthy      Failed
       │           │
       ▼           ▼
   Best Link    Next Link
       │
       ▼
    Gateway
       │
       ▼
       WAN
```

### Golden Rules

```text
1. SD-WAN rule decides WHAT traffic should prefer.
2. SLA decides WHETHER a member is healthy enough.
3. Routing/FIB decides WHETHER the selected path is reachable.
4. Firewall policy decides WHETHER traffic is allowed.
5. Load-balance mode decides HOW traffic is distributed.
6. Active probing generates measurement traffic.
7. Passive measurement relies on real traffic.
8. Spillover is useful when bandwidth thresholds matter.
9. Source-IP hashing provides stable path selection.
10. Always verify the actual forwarding path during troubleshooting.
```

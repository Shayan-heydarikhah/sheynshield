# FortiGate SD-WAN — Cloud On-Ramp & Troubleshooting
> **Focus:** Cloud On-Ramp, DNAT/VIP behavior, SD-WAN diagnostics, interface SLA logs, and Speed Test

---

## ☁️ Cloud On-Ramp

### VIP / Destination NAT + SD-WAN

When creating a **VIP / Destination NAT** rule:

* The firewall policy must use a **specific incoming interface**.
* You **cannot directly select an SD-WAN zone** or SD-WAN interface as the incoming interface for a VIP/DNAT policy in the same way as normal SD-WAN traffic steering.
* For DNAT policies, the incoming interface can be changed from an SD-WAN zone/member to another specific interface where required.
* The SD-WAN interface can still be used appropriately in the policy design for the forwarding side.

### Example Concept

```text
Internet
   |
   v
[ WAN / ISP ]
   |
   v
[ VIP / DNAT ]
   |
   v
[ Firewall Policy ]
   |
   v
[ SD-WAN ]
   |
   +------ WAN-1
   |
   +------ WAN-2
```

### Key Point

```text
VIP/DNAT
   |
   +-- Incoming interface
   |      └── Specific interface
   |
   +-- Forwarding
          └── Can participate in SD-WAN design
```

> ⚠️ **Design note:** DNAT/VIP traffic has different ingress-interface requirements than ordinary outbound SD-WAN traffic. Always verify the actual policy/VIP interface behavior on the FortiOS version in use.

---

# 🔎 SD-WAN Troubleshooting

## 1. SD-WAN Member Status

```bash
diagnose sys sdwan member
```

Useful for checking:

* SD-WAN member state
* Volume
* Volume room
* Overload volume
* Member utilization

### Volume Room

```text
Volume Room
    |
    └── Remaining capacity before reaching the configured/observed
        overload condition
```

### Overload Volume

```text
Overload Volume
    |
    └── Traffic has exceeded the normal/allowed forwarding volume
```

### Quick Interpretation

| Value           | Meaning                                 |
| --------------- | --------------------------------------- |
| Volume          | Current traffic/utilization information |
| Volume Room     | Remaining capacity                      |
| Overload Volume | Traffic beyond the normal threshold     |

---

# 🔗 Layer-2 / Destination MAC Troubleshooting

## Destination MAC

```bash
diagnose netlink dstmac list port1
```

Useful for checking:

* Destination MAC information
* Layer-2 forwarding behavior
* Interface-specific MAC resolution

---

# 📊 Interface SLA / Consumption History

## Interface SLA Log

```bash
diagnose sys sdwan intf-sla-log port1
```

Useful for observing:

* Interface consumption
* Historical interface behavior
* SLA-related interface statistics

> 📌 The command provides approximately the **last 15 minutes** of interface consumption information.

---

# ❤️ SD-WAN SLA Log

## Health-Check / SLA Log

```bash
diagnose sys sdwan sla-log ping 1
```

### Meaning of `1`

The number identifies an SLA / health-check entry.

If the health-check contains multiple components or members:

```text
SD-WAN Health Check
        |
        +-- SLA / Health Check #1
        |       |
        |       +-- Member 1
        |       +-- Member 2
        |       +-- ...
        |
        +-- SLA / Health Check #2
```

So:

```bash
diagnose sys sdwan sla-log ping 1
```

can be used to inspect the first relevant SLA/health-check entity.

---

# 🔌 Link Monitor

```bash
diagnose sys link-monitor interface port1
```

Useful for checking:

* Interface link monitoring
* Link state
* Monitoring status
* Interface health information

---

# 🚀 Speed Test Server

## Download / Update Speed-Test Server Information

```bash
execute speed-test-server download
```

### Important

The speed-test server information is periodically updated.

```text
FortiGate
   |
   +---- Speed-Test Server Database
              |
              └── Updated periodically
```

---

## 📋 List Speed-Test Servers

```bash
execute speed-test-server list
```

Useful for displaying available speed-test servers.

### Notes

```text
IPerf3
  |
  +-- Version 3.6
  |
  +-- SSL aggregation
  |
  +-- Ports 5204 - 5207
```

The speed-test infrastructure can be especially useful in **FortiGate Cloud / SD-WAN performance monitoring** scenarios.

---

# ⚡ Automatic Speed Test

## Basic Automatic Speed Test

```bash
execute speed-test auto
```

## AWS Speed Test

```bash
execute speed-test auto aws
```

Useful when testing connectivity/performance toward supported AWS speed-test infrastructure.

---

# 🧪 Manual Speed-Test Automation

A FortiGate **auto-script** can be used to periodically execute speed-test commands.

## Create Auto Script

```bash
config system auto-script
    edit speedtest
        set interval 86400
        set repeat 0
        set start auto
        set script "
            execute speed-test-server download
            execute speed-test
        "
    end
end
```

### Parameters

| Setting                              |   Value | Meaning                        |
| ------------------------------------ | ------: | ------------------------------ |
| `interval`                           | `86400` | Run every 24 hours             |
| `repeat`                             |     `0` | Continuous/recurring execution |
| `start`                              |  `auto` | Start automatically            |
| `execute speed-test-server download` |       — | Refresh server information     |
| `execute speed-test`                 |       — | Execute speed test             |

### Flow

```text
Auto-Script
    |
    | every 86400 seconds
    v
Download Speed-Test Server DB
    |
    v
Execute Speed Test
    |
    v
Store / Display Result
```

---

# 📑 View Auto-Script Results

```bash
execute auto-script result speedtest
```

Use this command to review the output/results generated by the `speedtest` auto-script.

---

# 🧰 SD-WAN Troubleshooting Quick Reference

| Problem                       | Command                                     |
| ----------------------------- | ------------------------------------------- |
| SD-WAN member utilization     | `diagnose sys sdwan member`                 |
| Destination MAC               | `diagnose netlink dstmac list port1`        |
| Interface SLA history         | `diagnose sys sdwan intf-sla-log port1`     |
| SLA log                       | `diagnose sys sdwan sla-log ping 1`         |
| Interface link monitoring     | `diagnose sys link-monitor interface port1` |
| Download speed-test server DB | `execute speed-test-server download`        |
| List speed-test servers       | `execute speed-test-server list`            |
| Automatic speed test          | `execute speed-test auto`                   |
| AWS speed test                | `execute speed-test auto aws`               |
| Auto-script results           | `execute auto-script result speedtest`      |

---

# 🧭 Practical Troubleshooting Flow

```text
                 SD-WAN Problem
                       |
                       v
             +-------------------+
             | Check SD-WAN      |
             | member status     |
             +---------+---------+
                       |
                       v
          diagnose sys sdwan member
                       |
                       v
             Check utilization
                       |
          +------------+------------+
          |                         |
          v                         v
     Normal volume             Overload?
          |                         |
          |                         v
          |                Check bandwidth
          |                / SLA / shaping
          |                         |
          +------------+------------+
                       |
                       v
             Check Interface SLA
                       |
                       v
      diagnose sys sdwan intf-sla-log
                       |
                       v
             Check SLA history
                       |
                       v
            diagnose sys sdwan
               sla-log ping 1
                       |
                       v
              Check link monitor
                       |
                       v
       diagnose sys link-monitor
             interface port1
                       |
                       v
               Need bandwidth
                  validation?
                       |
                       v
             Speed Test Server
                       |
          +------------+------------+
          |                         |
          v                         v
 execute speed-test-server    execute speed-test
       download
          |
          v
 execute speed-test-server list
```

---

# 🧠 Key Takeaways

* **VIP/DNAT** policies have specific incoming-interface requirements; don't treat them exactly like normal outbound SD-WAN rules.
* `diagnose sys sdwan member` is a good starting point for **member utilization and overload investigation**.
* `diagnose sys sdwan intf-sla-log` helps correlate **interface consumption with SD-WAN behavior**.
* `diagnose sys sdwan sla-log` is useful for investigating **SLA/health-check events**.
* `diagnose sys link-monitor` helps separate **physical/link-monitoring problems** from SD-WAN decision problems.
* Speed-test infrastructure can provide more meaningful **bandwidth measurements** than relying only on manually estimated WAN bandwidth.
* An **auto-script + scheduled speed test** can automate periodic WAN-performance measurements.

> ⚠️ **Version note:** FortiOS CLI syntax and available fields can vary between releases. Verify commands against the FortiOS version running on the device before applying them to production.

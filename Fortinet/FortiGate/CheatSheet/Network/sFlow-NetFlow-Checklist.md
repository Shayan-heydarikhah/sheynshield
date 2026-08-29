# 🌐 SheynShield Checklist
# FortiGate sFlow & NetFlow Configuration, Monitoring & Troubleshooting Checklist

> **FortiGate Traffic Visibility Engineering Checklist**  
> Complete guide for configuring, validating and troubleshooting **sFlow / NetFlow export**, collector integration and traffic analytics.

🔗 **SheynShield Resources**  
Security & Network Engineering Knowledge Base

---

# ✅ 1. sFlow & NetFlow Deployment Checklist

## Architecture Validation

- [ ] Identify monitoring requirement:
  - [ ] Packet visibility → sFlow
  - [ ] Flow/session accounting → NetFlow
  - [ ] Bandwidth analysis
  - [ ] Application visibility
  - [ ] Security monitoring

- [ ] Validate collector availability

Example:

```text
FortiGate
    |
    | UDP Export
    |
    +---- sFlow → UDP/9996
    |
    +---- NetFlow → UDP/2055
    |
    v
Flow Collector
    |
    v
Monitoring Platform (PRTG)
````

---

# ✅ 2. Interface Sampling Configuration

## Enable NetFlow Sampling

```bash
config system interface
    edit port3
        set netflow-sampler both
    next
end
```

## Enable sFlow Sampling

```bash
config system interface
    edit port3
        set sflow-sampler enable
        set sflow-rate 2000
        set sample-direction both
    next
end
```

---

## Configuration Validation

| Parameter               | Validation                       |
| ----------------------- | -------------------------------- |
| `netflow-sampler both`  | NetFlow enabled ingress + egress |
| `sflow-sampler enable`  | sFlow sampling enabled           |
| `sflow-rate`            | Sampling frequency configured    |
| `sample-direction both` | Traffic sampled both directions  |

---

# ✅ 3. NetFlow Configuration Checklist

## Configure Collector

```bash
config system netflow
    set collector-ip 192.168.20.202
    set collector-port 2055
    set active-flow-timeout 1000
    set inactive-flow-timeout 15
    set template-tx-timeout 1000
    set template-tx-counter 20
    set source-ip 192.168.20.254
end
```

---

## NetFlow Timer Review

| Parameter               | Purpose                          |
| ----------------------- | -------------------------------- |
| `active-flow-timeout`   | Export active flows periodically |
| `inactive-flow-timeout` | Export inactive flows            |
| `template-tx-timeout`   | Template refresh interval        |
| `template-tx-counter`   | Template transmission counter    |
| `source-ip`             | Export source address            |

---

## Validation Checklist

* [ ] Collector IP reachable
* [ ] UDP/2055 allowed
* [ ] Source IP exists on FortiGate
* [ ] Collector accepts NetFlow version
* [ ] Templates are received correctly

---

# ✅ 4. sFlow Configuration Checklist

## Configure Collector

```bash
config system sflow
    set collector-ip 192.168.20.202
    set collector-port 9996
end
```

---

## Validation

* [ ] Collector IP configured
* [ ] UDP/9996 reachable
* [ ] Interface sampling enabled
* [ ] sFlow daemon running

---

# ✅ 5. PRTG Integration Checklist

## Add Monitoring Sensors

```
PRTG
 |
 +-- Device
      |
      +-- Sensor
            |
            +-- NetFlow v5
            +-- NetFlow v9
            +-- sFlow
```

---

## Recommended Settings

* [ ] Active Flow Timeout configured
* [ ] Stream data storage enabled
* [ ] Collector receives traffic
* [ ] Top talkers visible
* [ ] Application traffic visible

---

# ✅ 6. Packet Capture Validation Checklist

## Capture sFlow Traffic

```bash
diagnose sniffer packet '192.168.20.202 9996' 6 0 a
```

---

## Sniffer Syntax

```text
diagnose sniffer packet <filter> <verbose> <count> <timestamp>
```

---

## Useful Parameters

| Value | Meaning                     |
| ----- | --------------------------- |
| `1`   | Basic output                |
| `3`   | IP/TCP/UDP information      |
| `6`   | Detailed logging            |
| `0`   | Unlimited packets           |
| `a`   | ASIC/hex output             |
| `x`   | Wireshark compatible output |

---

## Validation Checklist

* [ ] UDP packets leaving FortiGate
* [ ] Destination collector IP correct
* [ ] Correct UDP port
* [ ] No firewall blocking
* [ ] Collector receives packets

---

# ✅ 7. sFlow / NetFlow Diagnostic Checklist

## sFlow Diagnostic Menu

```bash
diagnose test application sflowd 3
```

---

## Diagnostic Options

| Option | Function                        |
| ------ | ------------------------------- |
| `1`    | Show sFlow collector settings   |
| `2`    | Show sFlow statistics           |
| `3`    | Show NetFlow collector settings |
| `4`    | Show NetFlow status             |
| `5`    | Reset NetFlow statistics        |
| `6`    | Send application information    |
| `7`    | Show long-lived session cache   |
| `8`    | Packet dump control             |

---

## Troubleshooting Sequence

```bash
# Collector configuration
diagnose test application sflowd 1
diagnose test application sflowd 3

# Statistics
diagnose test application sflowd 2
diagnose test application sflowd 4

# Packet validation
diagnose sniffer packet '192.168.20.202 9996' 6 0 a

# Reset counters
diagnose test application sflowd 5
```

---

# ✅ 8. sFlow Data Validation Checklist

Verify collected information:

## Packet Information

* [ ] MAC address
* [ ] IPv4 information
* [ ] TCP information

---

## Sampling Information

* [ ] Sampling rate
* [ ] Sampling pool

---

## Interface Information

* [ ] Input interface
* [ ] Output interface
* [ ] Interface statistics

Supported:

```
RFC 1573
RFC 2233
RFC 2358
```

---

## QoS / VLAN Visibility

* [ ] 802.1p priority
* [ ] TOS
* [ ] 802.1Q VLAN

---

## Routing Information

* [ ] Source prefixes
* [ ] Destination prefixes
* [ ] Next-hop information

---

## BGP Visibility

* [ ] Source AS
* [ ] Source peer AS
* [ ] Destination peer AS
* [ ] Communities
* [ ] Local preference

---

## User Information

* [ ] TACACS information
* [ ] RADIUS information
* [ ] Source user ID
* [ ] Destination user ID

---

# ✅ 9. sFlow vs NetFlow Decision Checklist

| Requirement              | Technology |
| ------------------------ | ---------- |
| Packet sampling          | sFlow      |
| Flow accounting          | NetFlow    |
| Top talkers              | Both       |
| Session statistics       | NetFlow    |
| Packet header visibility | sFlow      |
| Traffic analytics        | Both       |
| IP byte accounting       | NetFlow    |

---

# ✅ 10. Troubleshooting Decision Tree

```
Traffic Visibility Problem
            |
            v
Check Interface Sampling
            |
            v
Check Collector Configuration
            |
            v
Check UDP Port
            |
     +------+------+
     |             |
   sFlow        NetFlow
     |             |
 UDP/9996      UDP/2055
     |             |
     +------+------+
            |
            v
Packet Capture
            |
            v
Daemon Statistics
            |
            v
Collector Validation
            |
            v
PRTG Validation
```

---

# ✅ 11. Quick Command Reference

| Task                   | Command                                   |
| ---------------------- | ----------------------------------------- |
| sFlow diagnostics      | `diagnose test application sflowd 3`      |
| Packet capture         | `diagnose sniffer packet 'IP PORT' 6 0 a` |
| Show sFlow settings    | `diagnose test application sflowd 1`      |
| Show sFlow statistics  | `diagnose test application sflowd 2`      |
| Show NetFlow settings  | `diagnose test application sflowd 3`      |
| Show NetFlow status    | `diagnose test application sflowd 4`      |
| Reset NetFlow counters | `diagnose test application sflowd 5`      |

---

# 🧠 12. Engineer Memory Shortcut

```
sFlow
 |
 +-- Sample packets
 +-- Packet headers
 +-- VLAN/QoS
 +-- Interface statistics
 +-- Traffic visibility


NetFlow
 |
 +-- Track flows
 +-- Sessions
 +-- Source/Destination IP
 +-- Bytes
 +-- Accounting
```

---

# 🎯 Final Validation Checklist

## Before Production

* [ ] Collector reachable
* [ ] UDP ports allowed
* [ ] Interface sampling enabled
* [ ] Export source IP configured
* [ ] Templates received
* [ ] Flow records visible
* [ ] PRTG sensors active
* [ ] Packet capture verified
* [ ] Statistics counters increasing

---

# 🔗 SheynShield Resources

## 🎥 Video Learning

* YouTube — SheynShield

  * Fortinet NSE Content
  * FortiGate Troubleshooting
  * Network Security Engineering

## 📚 Notes & Updates

* Telegram — SheynShield

## 💼 Professional Network

* LinkedIn — Shayan-heydarikhah

## 🐙 Technical Knowledge Base

* SheynShield GitHub

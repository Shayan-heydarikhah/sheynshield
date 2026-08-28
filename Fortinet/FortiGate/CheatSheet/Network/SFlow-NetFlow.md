# sFlow & NetFlow 

## 1. Interface Configuration

```bash
config system interface
    edit port3
        set netflow-sampler both
        set sflow-sampler enable
        set sflow-rate 2000
        set sample-direction both
    next
end
```

| Setting                 | Meaning                                    |
| ----------------------- | ------------------------------------------ |
| `netflow-sampler both`  | Enable NetFlow sampling in both directions |
| `sflow-sampler enable`  | Enable sFlow sampling                      |
| `sflow-rate 2000`       | Configure sFlow sampling rate              |
| `sample-direction both` | Sample ingress + egress traffic            |

---

## 2. NetFlow Configuration

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

### Important Timers

| Parameter               |            Value | Purpose                                                                |
| ----------------------- | ---------------: | ---------------------------------------------------------------------- |
| `active-flow-timeout`   |           `1000` | Export a report for active traffic flows after the configured interval |
| `inactive-flow-timeout` |             `15` | Export inactive-flow information after 15 seconds                      |
| `template-tx-timeout`   |           `1000` | Template transmission interval                                         |
| `template-tx-counter`   |             `20` | Template records/counter used for template checking/creation           |
| `source-ip`             | `192.168.20.254` | Source IP used for flow export                                         |

> **Reference:** For NetFlow templates, see the **FortiOS 7.2.0 Administration Guide**, around page **436**.

---

## 3. sFlow Configuration

```bash
config system sflow
    set collector-ip 192.168.20.202
    set collector-port 9996
end
```

### Collector

```text
FortiGate
   |
   | sFlow
   | UDP/9996
   v
192.168.20.202
```

---

# 4. PRTG Configuration

On **PRTG**:

```text
Add Device
    └── Add Sensor
          ├── NetFlow v5
          ├── NetFlow v9
          └── sFlow
```

Suggested monitoring configuration from the notes:

```text
Active Flow Timeout: 30 minutes
Store all stream data
```

---

# 5. Packet Capture — sFlow

```bash
diagnose sniffer packet '192.168.20.202 9996' 6 0 a
```

### Sniffer Parameters

```text
diagnose sniffer packet <interface/filter> <verbose> <count> <timestamp>
```

| Value | Meaning                               |
| ----- | ------------------------------------- |
| `6`   | Full logging/sniffer information      |
| `1`   | Basic information                     |
| `3`   | IP / UDP / TCP information            |
| `0`   | Capture unlimited packets             |
| `a`   | ASIC/hex dump                         |
| `x`   | Wireshark-compatible output (`cflow`) |

Example:

```bash
diagnose sniffer packet '192.168.20.202 9996' 6 0 a
```

---

# 6. sFlow / NetFlow Diagnostics

```bash
diagnose test application sflowd 3
```

### Diagnostic Menu

| Option | Function                                   |
| -----: | ------------------------------------------ |
|    `1` | Show sFlow collector settings              |
|    `2` | Show sFlow statistics                      |
|    `3` | Show NetFlow collector settings            |
|    `4` | Show NetFlow status                        |
|    `5` | Reset NetFlow exported statistics          |
|    `6` | Send application information to collectors |
|    `7` | Show long-lived session cache entries      |
|    `8` | Enable/disable log packet dump             |

> ⚠️ Option `6` may consume additional bandwidth because application information is sent to collectors.

---

# 7. sFlow Datagram Contents

sFlow datagrams can contain information such as:

### Packet Information

```text
MAC
IPv4
TCP
```

### Sampling Information

```text
Sampling rate
Sampling pool
```

### Interface Information

```text
Input port
Output port
Interface statistics
```

Interface statistics include:

```text
RFC 1573
RFC 2233
RFC 2358
```

### QoS / VLAN

```text
802.1p priority
TOS
802.1Q VLAN
```

### Routing Information

```text
Source prefixes
Destination prefixes
Next-hop addresses
```

### BGP Information

```text
Source AS
Source peer AS
Destination peer AS
Communities
Local preference
```

### User Information

```text
TACACS
RADIUS
Source user ID
Destination user ID
```

> sFlow can also work with **NATed traffic** and provide private-IP-related attributes.

---

# 8. sFlow vs NetFlow

| Feature                    | sFlow                         | NetFlow                  |
| -------------------------- | ----------------------------- | ------------------------ |
| Primary purpose            | Packet sampling               | Flow/session tracking    |
| Packet sampling            | ✅                             | —                        |
| Session tracking           | Limited                       | ✅                        |
| Byte counting per IP       | —                             | ✅                        |
| Connection/load visibility | ✅                             | ✅                        |
| Packet headers             | ✅                             | Flow records             |
| Sampling-based             | ✅                             | Depends on configuration |
| Best use                   | Traffic sampling & visibility | Flow/session accounting  |

### Conceptual Difference

```text
sFlow
    ↓
Sample packets
    ↓
Packet/header information
    ↓
Traffic / connection / load visibility


NetFlow
    ↓
Track flows/sessions
    ↓
Flow records
    ↓
IP / bytes / session statistics
```

---

# 9. Quick Troubleshooting Flow

```text
                 ┌──────────────────┐
                 │    FortiGate     │
                 └────────┬─────────┘
                          │
              ┌───────────┴───────────┐
              │                       │
           sFlow                   NetFlow
              │                       │
        UDP / 9996              UDP / 2055
              │                       │
              └───────────┬───────────┘
                          │
                          ▼
                  192.168.20.202
                     Collector
                          │
                          ▼
                         PRTG
```

### Verification Sequence

```bash
# 1. Check collector configuration
diagnose test application sflowd 1
diagnose test application sflowd 3

# 2. Check statistics/status
diagnose test application sflowd 2
diagnose test application sflowd 4

# 3. Capture exported packets
diagnose sniffer packet '192.168.20.202 9996' 6 0 a

# 4. Reset NetFlow statistics if required
diagnose test application sflowd 5
```

---

## 10. Memory Shortcut

```text
sFlow  →  Sample packets
NetFlow → Track flows

sFlow
  ├── Packet headers
  ├── Sampling
  ├── Ports
  ├── VLAN
  ├── QoS
  └── Interface statistics

NetFlow
  ├── Sessions / flows
  ├── IP information
  └── Byte / traffic accounting

# FortiGate Troubleshooting 

> **Scope:** FortiOS system, CPU, memory, sessions, routing, FortiGuard, modem/PPP, packet capture, NIC, ARP, NPU and TAC troubleshooting.

---

## 1. System Status

### Basic System Information

```bash
get system status
```

**Useful for:**

* FortiOS firmware version
* FortiGuard engine versions
* System information
* Hardware/model information

---

## 2. System Performance

```bash
get system performance status
```

Shows:

* CPU usage
* Memory usage
* Average network usage
* Average sessions
* Session setup rate
* Viruses caught
* IPS attacks blocked
* System uptime

### CPU States

Example:

```text
CPU states: 0% user 0% system 0% nice 100% idle 0% iowait 0% irq 0% softirq
```

| Field            | Meaning                             |
| ---------------- | ----------------------------------- |
| `user`           | CPU used by user-space applications |
| `system`         | CPU used by kernel/system processes |
| `nice`           | CPU used by low-priority processes  |
| `idle`           | Unused CPU capacity                 |
| `iowait`         | CPU waiting for I/O operations      |
| `irq` / `hi`     | Hardware interrupt processing       |
| `softirq` / `si` | Software interrupt processing       |
| `steal` / `st`   | CPU time stolen by the hypervisor   |

### Important Interpretation

#### High `user`

May indicate:

* Proxy processing
* Security inspection
* Encryption
* Application processing
* Heavy logging

#### High `system`

May indicate:

* Kernel activity
* Networking load
* NPU/ASIC bypass problems
* Traffic being processed by CPU
* Excessive packet processing

#### High `nice`

Indicates CPU consumption by low-priority/background processes.

> On Linux-based systems, `nice` processes are scheduled with lower priority.

#### High `iowait`

CPU is waiting for I/O.

Possible causes:

* Disk activity
* Logging
* Network I/O
* Storage bottlenecks

#### High `irq`

High hardware interrupt processing.

Possible causes:

* NIC problems
* Hardware activity
* Interrupt storms
* Driver/hardware issues

#### High `softirq`

Can indicate heavy processing related to:

* Networking
* IPS
* VPN
* Firewall
* System calls
* NPU/accelerator interaction
* Timers/scheduling

> **High `softirq` can indicate a processing loop or abnormal workload.**

---

## 3. CPU Emergency Conditions

### Example 1 — No CPU Response

```text
CPU states:
0% user
0% system
0% nice
100% idle
0% iowait
0% irq
0% softirq
```

If the FortiGate becomes completely unresponsive despite apparently idle CPU:

Possible causes:

* Motherboard problem
* CPU/hardware problem
* Kernel panic
* Resource deadlock
* Excessive proxy workload
* System-level failure

> If the system is completely unresponsive, a controlled reboot may not be possible and a forced shutdown may be required.

---

### Example 2 — System CPU Saturation

```text
CPU states:
1% user
98% system
0% nice
1% idle
```

This indicates that the CPU is almost completely consumed by system/kernel processing.

Possible causes:

* Excessive packet processing
* Traffic bypassing SPU/NPU acceleration
* IPS processing
* DoS/DDoS traffic
* Excessive sessions
* `scanunitid` or similar inspection processes
* Malicious traffic

### Possible Response

Reduce unnecessary traffic inspection:

* Block unwanted protocols
* Restrict security policies
* Reduce unnecessary scanning
* Review proxy-based policies
* Investigate abnormal traffic

> A compromised host may also generate excessive traffic, such as spam, scanning, or attack traffic.

---

## 4. Firewall Statistics

```bash
get system performance firewall statistics
```

Useful for reviewing firewall processing statistics.

---

## 5. Top Processes

```bash
get system performance top
```

Shows top FortiGate processes and their CPU consumption.

Look for:

* Processes consuming abnormal CPU
* Unexpected processes
* Sudden CPU spikes
* Persistent high CPU usage

---

## 6. Detailed Process Monitoring

```bash
diagnose sys top
```

```bash
diagnose sys top-all
```

### Interactive Sorting

| Key | Function        |
| --- | --------------- |
| `m` | Sort by memory  |
| `p` | Sort by process |
| `q` | Exit            |

### Example

```text
Run Time: 86 days, 0 hours and 10 minutes

0U, 0N, 0S, 100I, 0WA, 0HI, 0SI, 0ST
3040T, 2437F

bcm.user       93 S < 3.1 0.4
httpsd     18922 S   1.5 0.5
httpsd     19150 S   0.3 0.5
newcli     20195 R   0.1 0.1
cmdbsvr      115 S   0.0 0.8
pyfcgid    20107 S   0.0 0.6
forticron    146 S   0.0 0.5
```

### CPU Fields

| Field | Meaning               |
| ----- | --------------------- |
| `U`   | User-space CPU        |
| `N`   | Nice/low-priority CPU |
| `S`   | System/kernel CPU     |
| `I`   | Idle CPU              |
| `WA`  | I/O wait              |
| `HI`  | Hardware interrupts   |
| `SI`  | Software interrupts   |
| `ST`  | Hypervisor steal time |

### Memory Fields

| Field | Meaning                     |
| ----- | --------------------------- |
| `T`   | Total FortiOS system memory |
| `F`   | Free memory                 |

### Process Fields

| Field        | Meaning               |
| ------------ | --------------------- |
| Process name | Process/thread name   |
| PID          | Process/thread ID     |
| `R`          | Running               |
| `S`          | Sleeping              |
| `Z`          | Zombie                |
| `D`          | Disk sleep / I/O wait |
| CPU %        | CPU consumed          |
| MEM %        | Memory consumed       |

### Filtered Process View

```bash
diagnose sys top 5 20
```

```text
5  = refresh/delay interval
20 = number of processes displayed
```

---

# 7. Session Setup Rate

> Session setup rate is a **global command/statistic**.

In environments with multiple VDOMs and many sessions:

* Session setup rate per VDOM may appear lower.
* Global session processing is shared across VDOMs.

---

# 8. CPU Hardware Information

```bash
diagnose hardware sysinfo cpu
```

Useful for:

* CPU model
* CPU generation
* Microcode
* CPU patches
* Cache
* TLB
* CPU capabilities

### CPU Features

| Flag      | Meaning                                                 |
| --------- | ------------------------------------------------------- |
| `fpu`     | Floating Point Unit                                     |
| `pse36`   | 36-bit physical addressing                              |
| `clflush` | Cache-line flush                                        |
| `pat`     | Page Attribute Table                                    |
| `sse2`    | SIMD processing                                         |
| `sse4a`   | SIMD acceleration                                       |
| `popcnt`  | Population count, useful for networking/CIDR operations |
| `nx`      | No-Execute protection                                   |
| `lm`      | Long Mode / 64-bit support                              |
| `svm`     | AMD virtualization support                              |
| `tsc`     | Time Stamp Counter                                      |
| `cmov`    | Conditional move                                        |
| `fpu`     | Floating-point calculations                             |

### CPU-Intensive Features

Common CPU-heavy operations include:

* High-level VPN encryption
* Full traffic inspection
* Heavy IPS inspection
* Proxy-based security profiles
* Extensive logging
* Packet logging
* Frequently updated dashboards
* Traffic that cannot be hardware-offloaded

---

# 9. Memory Troubleshooting

```bash
get hardware memory
```

```bash
diagnose hardware sysinfo memory
```

### Important Memory Fields

| Field          | Meaning                                 |
| -------------- | --------------------------------------- |
| `MemFree`      | Currently free memory                   |
| `Active`       | Active memory                           |
| `Swap`         | Memory swapped to disk                  |
| `SReclaimable` | Reclaimable kernel memory               |
| `SUnreclaim`   | Non-reclaimable kernel memory           |
| `Shmem`        | Shared memory                           |
| `HugePages`    | Large memory pages                      |
| `Committed_AS` | Memory committed/requested by processes |

### Memory Health

A practical warning point:

```text
MemFree ≈ 20%
```

is generally more comfortable than:

```text
MemFree ≈ 10%
```

Very low free memory may lead to:

* Conservation mode
* Service instability
* Connection loss
* New connections being refused
* Process freezing

### Swap

```text
Swap ≈ 0
```

is generally desirable.

Excessive swap usage can indicate memory pressure.

---

# 10. High Memory Usage

High memory usage can be caused by:

* High traffic volume
* Large numbers of sessions
* Proxy workloads
* Security inspection
* Connection pools
* Logging
* Memory leaks
* Excessive application processing

### Proxy Connection Pool Problem

If a proxy receives a very large number of connections:

```text
Free connection pool
        ↓
       0
        ↓
Proxy cannot accept/process more connections
```

Possible symptoms:

* Services freeze
* Connections are lost
* New connections fail
* Proxy becomes unstable

---

# 11. TCP/UDP Session Timers

```bash
config system global
    set tcp-halfclose-timer 30
    set tcp-halfopen-timer 30
    set tcp-timewait-timer 0
    set udp-idle-timer 60
end
```

### Timer Meaning

| Timer                 | Purpose                       |
| --------------------- | ----------------------------- |
| `tcp-halfclose-timer` | Session after one side closes |
| `tcp-halfopen-timer`  | Half-open TCP/SYN sessions    |
| `tcp-timewait-timer`  | TCP TIME_WAIT sessions        |
| `udp-idle-timer`      | Idle UDP sessions             |

### Typical Considerations

```text
tcp-halfclose-timer = 30
```

Can reduce resource retention after one side closes.

```text
tcp-halfopen-timer = 30
```

Can help reduce resources consumed by half-open/SYN sessions.

```text
tcp-timewait-timer = 0
```

Can reduce TIME_WAIT memory/session retention depending on the environment.

> **Do not blindly tune timers.** Validate application behavior and session requirements first.

---

# 12. Session Status

```bash
get system session status
```

Shows:

* Total sessions
* Session statistics

### Detailed Sessions

```bash
diagnose sys session list
```

Useful for:

* Source/destination
* Ports
* Session state
* Policy
* NAT
* Offload information
* Session flags

---

# 13. Routing Troubleshooting

```bash
get router info routing-table all
```

Shows:

* Routing table
* Route type
* Route source
* Next-hop information
* Routing decisions

### BGP Routes

```bash
get router info routing-table bgp
```

---

# 14. IPS / WebFilter

### IPS Session Information

```bash
get ips session
```

Useful for:

* IPS memory usage
* Maximum IPS memory
* IPS session counters

### FortiGuard WebFilter Statistics

```bash
get webfilter ftgd-statistics
```

Shows FortiGuard-related:

* Requests
* Status
* Errors
* Counters

---

# 15. DNS Troubleshooting

```bash
show system dns
```

Shows configured DNS servers.

---

# 16. NTP Troubleshooting

```bash
diagnose sys ntp status
```

Useful for:

* NTP server status
* Synchronization
* Time source information

---

# 17. VDOM Troubleshooting

If VDOMs are enabled:

```text
Troubleshooting
      │
      ├── Current VDOM
      │
      ├── Other VDOMs
      │
      └── Global/Super_Admin
```

Consider:

* Notify the `super_admin`
* Confirm VDOM permissions
* Check required interfaces
* Coordinate testing with network administrators
* Verify switches/routers/servers involved in the path

---

# 18. TAC Report

```bash
execute tac report
```

> **TAC = Technical Assistance Center**

Generates an extensive system snapshot.

Useful for:

* Fortinet Support
* Comparing system state
* Capturing configuration-related diagnostics
* Troubleshooting complex issues

### Comparison Workflow

```text
TAC Report #1
     ↓
Troubleshooting / Changes
     ↓
TAC Report #2
     ↓
Compare
     ↓
Identify abnormal changes
```

> The TAC report can be very large. Do **not** automatically provide it to support unless requested.

---

# 19. Support Information Checklist

When opening a Fortinet support case, prepare:

* [ ] FortiOS firmware/build version
* [ ] Network topology diagram
* [ ] Recent configuration backup
* [ ] Relevant debug logs
* [ ] Problem description
* [ ] Troubleshooting steps performed
* [ ] Results of each test
* [ ] Time of occurrence
* [ ] Relevant packet captures

### Avoid Sending

```text
execute tac report
```

unless Fortinet Support specifically requests it.

---

# 20. Configuration Backups

> **Best Practice:** Perform periodic configuration backups.

Recommended:

```text
Scheduled Backup
       ↓
Versioned Configuration
       ↓
Change Tracking
       ↓
Fast Recovery
```

---

# 21. SNMP / Monitoring

Periodic monitoring can be used to track:

* CPU
* Memory
* Sessions
* Interface utilization
* System health
* Resource trends

A practical monitoring interval:

```text
300 seconds
```

Use SNMP monitoring to detect trends before they become incidents.

---

# 22. Modem / SIM / PPP Troubleshooting

### Modem Diagnostic Commands

```bash
diagnose sys modem cmd
```

Useful modem operations include:

* Communication
* Detection
* History
* External modem
* Query
* Signaling
* Reset

### Detect Modem

```bash
diagnose sys modem detect
```

Useful for USB/external modem detection.

### Modem Configuration

```bash
get system modem
```

Shows:

* Modem configuration
* Vendor
* Product identification
* Custom modem information

---

## 23. Modem Debugging

Enable debug:

```bash
diagnose debug enable
```

### Modem Daemon

```bash
diagnose debug application modemd
```

Useful for observing communication between:

```text
FortiGate
   ↕
Modem
```

### PPP Negotiation

```bash
diagnose debug application ppp
```

Useful for:

* PPP negotiation
* Authentication
* Link establishment
* PPP failures

### Dial

```bash
execute modem dial
```

Displays modem-related debug information.

---

# 24. Packet Sniffer

Before starting a large packet capture:

> **Save the output to a file whenever possible.**

Large captures can generate huge amounts of output.

### Syntax

```bash
diagnose sniffer packet <interface> <filter> <verbose> <count> <tsformat>
```

### Example

```bash
diagnose sniffer packet port1 'host 1.1.1.1' 4 100 l
```

---

## 25. Sniffer Verbosity

| Level | Description                    |
| ----: | ------------------------------ |
|   `1` | Packet header                  |
|   `2` | Header + IP packet data        |
|   `3` | Header + Ethernet data/MAC     |
|   `4` | Header + interface information |

### Capture All Traffic

If no filter is required:

```bash
diagnose sniffer packet port1 none 4 100 l
```

---

## 26. Sniffer Timestamp

| Value | Meaning    |
| ----- | ---------- |
| `l`   | Local time |
| `a`   | UTC        |

Example:

```bash
diagnose sniffer packet port1 'host 1.1.1.1' 4 100 a
```

---

# 27. NPU and Packet Sniffing

Hardware acceleration can change the packet path.

For NP4/NP6-related troubleshooting, offloading may need to be disabled before capturing traffic.

Conceptually:

```text
Ingress
   ↓
NPU
   ↓
Fast Path
   ↓
Egress
```

instead of:

```text
Ingress
   ↓
CPU
   ↓
Firewall
   ↓
Egress
```

### Disable Fast Path

```bash
diagnose npu <interface-pair> fastpath disable
```

Possible interface pairs:

```text
np4
np6
np4lite
np6lite
```

> Use this only when required for troubleshooting and restore normal acceleration afterward.

---

# 28. Debug Flow Warning

If FortiGate is connected to:

* FortiAnalyzer
* FortiCloud

debug-flow output may be recorded as event logs and forwarded to those systems.

> **Do not run debug flow longer than necessary.**

It can generate a significant amount of data.

---

# 29. NIC / Hardware Troubleshooting

Hardware troubleshooting should separate:

```text
Layer 1
   ↓
Speed
Duplex
Physical link
Cable/transceiver

Layer 2
   ↓
MAC
CRC
Frame errors
Drops
Buffer issues
```

### NIC Information

```bash
diagnose hardware deviceinfo nic port3
```

Useful for:

* NIC state
* Hardware counters
* Errors
* CRC-related problems
* Buffer-related issues

> Unexpected packet drops or CRC errors can indicate a physical/NIC problem.

---

# 30. CMDB Reference Check

To determine which configuration objects reference an interface:

```bash
diagnose sys cmdb refcnt show system.interface.name port1
```

Useful when trying to:

* Delete an interface
* Rename an interface
* Change interface configuration
* Find dependent policies/objects

---

# 31. XAUI

XAUI is used for high-speed internal connectivity between processing components/NPU paths.

### XAUI Hash

```bash
diagnose npu np6 xaui-hash port1 6 1.1.1.1 2.2.2.1 4567 80
```

Useful on supported platforms for examining stream distribution across NPU/XAUI paths.

Examples include:

```text
38xxD
39xxD
34xxE
36xxE
5001E
```

---

# 32. ARP Troubleshooting

### View ARP Table

```bash
get system arp
```

Useful for:

* IP ↔ MAC mapping
* ARP state
* Interface
* Usage/reference information

---

## 33. ARP States

| Hex    | State     | Meaning                             |
| ------ | --------- | ----------------------------------- |
| `0x02` | Reachable | ARP response received               |
| `0x04` | Stale     | No recent ARP response              |
| `0x08` | Delay     | Transition before probing           |
| `0x20` | Failed    | ARP resolution failed               |
| `0x40` | NoARP     | Device does not use ARP, e.g. IPsec |
| `0x80` | Permanent | Static ARP entry                    |

### ARP Reachability

The FortiGate sends ARP when it needs to resolve a new destination.

The actual reachable time is randomized around the configured base value.

Example:

```text
Base reachable time = 30 seconds

Actual value:
15–45 seconds
```

The randomized value is periodically refreshed.

---

# 34. ARP Table Size

```bash
config system global
    set arp-max-entry 10
end
```

Example:

```text
arp-max-entry = 10
```

> Set the value according to the actual environment; do not use a small value in production without understanding the impact.

---

# 35. ARP Diagnostics

### List ARP

```bash
diagnose ip arp list
```

### Add ARP

```bash
diagnose ip arp add
```

### Delete ARP

```bash
diagnose ip arp delete port1 192.168.101.200
```

### Clear ARP Table

```bash
execute clear system arp table
```

---

# 36. Interface ARP Reachable Time

```bash
config system interface
    edit port1
        set reachable-time 30000
    end
end
```

Unit:

```text
milliseconds
```

Maximum:

```text
3600000 ms
```

Example:

```text
30000 ms = 30 seconds
```

---

# 37. Static ARP

```bash
config system arp-table
    edit 1
        set interface port1
        set ip 192.168.101.200
        set mac bc:14:01:e9:77:02
    next
end
```

Use static ARP only when there is a clear operational requirement.

---

# 38. Device / Routing Database

```bash
diagnose sys device list root
```

Useful for understanding:

* Recursive lookup depth
* Routing database
* RIB behavior
* Local routes
* RPDB
* Route table size

### Useful Concepts

```text
RIB
 ↓
Route lookup
 ↓
Recursive lookup
 ↓
Forwarding decision
```

A very deep recursive lookup can indicate complex routing behavior and should be investigated.

---

# 39. FortiGuard Connectivity

FortiGuard connectivity should be verified.

### Ping FortiGuard

```bash
execute ping service.fortiguard.net
```

```bash
execute ping update.fortiguard.net
```

> In HA environments, verify that cluster members have compatible FortiGuard licensing/service levels.

---

# 40. FortiGuard Rating Diagnostics

```bash
diagnose debug rating
```

Useful for:

* FortiGuard server selection
* RTT
* Server status
* Connectivity
* Rating server information

### WebFilter Status

```bash
get webfilter status
```

Can help verify:

* FortiGuard connectivity
* RTT
* Server status
* Service information

---

# 41. FortiGuard Server Flags

| Flag | Meaning                                         |
| ---- | ----------------------------------------------- |
| `d`  | Server discovered through DNS                   |
| `i`  | Server used for the last initialization request |
| `f`  | Server failed/unresponsive                      |
| `t`  | Server currently being timed                    |
| `s`  | Server can receive rating requests              |

### `d` Flag

The server was discovered through DNS.

If DNS returns multiple IP addresses:

```text
Hostname
   ↓
Multiple IPs
   ↓
Servers marked with d
```

These servers are preferred during initialization before fallback.

### `i` Flag

Server used for the latest initialization request.

### `f` Flag

Server did not respond and is considered failed.

### `t` Flag

Server is currently being measured/timed.

### `s` Flag

Server is eligible for rating requests.

---

# 42. FortiGuard Server Selection Concept

FortiGate considers:

* Server weight
* RTT
* Success/failure history
* Availability

Conceptually:

```text
Server
  │
  ├── RTT
  ├── Weight
  ├── Success
  └── Failure
       ↓
Server score
       ↓
Preferred FortiGuard server
```

Successful communication improves server preference, while failures reduce it.

---

# 43. TCP Socket Troubleshooting

```bash
diagnose sys tcpsocks
```

Shows opened TCP/UDP sockets.

Useful when investigating:

* Local services
* Listening ports
* Connection failures
* Local-in behavior
* Socket exhaustion

### Buffer Fields

| Field | Meaning                      |
| ----- | ---------------------------- |
| `RMA` | Read buffer information      |
| `WMA` | Write buffer information     |
| `FMA` | Flags/bit-mask information   |
| `TMA` | Timeout base in milliseconds |

### Socket Type

Common values identify socket protocol types such as:

```text
TCP
UDP
```

### Error Example

```text
err = 111
```

Commonly indicates:

```text
Connection refused / connection failure
```

---

# 44. Quick Troubleshooting Workflow

```text
                 ┌──────────────────┐
                 │  Problem Report  │
                 └────────┬─────────┘
                          ↓
                ┌────────────────────┐
                │ get system status  │
                └─────────┬──────────┘
                          ↓
             ┌──────────────────────────┐
             │ Performance / Resources  │
             └────────────┬─────────────┘
                          ↓
              ┌────────────────────────┐
              │ CPU / Memory / Sessions │
              └────────────┬───────────┘
                           ↓
          ┌────────────────┴────────────────┐
          ↓                                 ↓
     Network issue                    Resource issue
          ↓                                 ↓
      ARP / NIC                         Top processes
      Routing                           Memory
      FortiGuard                        Sessions
      Sniffer                            IPS/Proxy
          ↓                                 ↓
       NPU path                        Policy review
          ↓                                 ↓
    Offloading check                  Optimization
          └──────────────┬──────────────────┘
                         ↓
                  TAC / Support
```

---

# 45. First-Level Command Checklist

```bash
get system status

get system performance status

get system performance firewall statistics

get system performance top

diagnose sys top

diagnose sys top-all

diagnose hardware sysinfo cpu

get hardware memory

diagnose hardware sysinfo memory

get system session status

diagnose sys session list

get router info routing-table all

get ips session

get webfilter ftgd-statistics

show system dns

diagnose sys ntp status

get system arp

diagnose ip arp list

diagnose sys tcpsocks
```

---

# 46. Network-Specific Checklist

```bash
# Routing
get router info routing-table all

# ARP
get system arp
diagnose ip arp list

# Interface/NIC
diagnose hardware deviceinfo nic port3

# Session
diagnose sys session list

# Packet capture
diagnose sniffer packet port1 'host 1.1.1.1' 4 100 l

# NPU
diagnose npu np6 port-list

# FortiGuard
diagnose debug rating
get webfilter status

# DNS
show system dns

# NTP
diagnose sys ntp status
```

---

# 47. Resource Troubleshooting Decision Matrix

| Symptom                     | First Checks                       |
| --------------------------- | ---------------------------------- |
| High CPU                    | `get system performance status`    |
| High system CPU             | `diagnose sys top`                 |
| High user CPU               | Proxy/security processes           |
| High softirq                | Networking/NPU/IPS/VPN             |
| High IRQ                    | NIC/hardware/interrupts            |
| High iowait                 | Disk/logging/I/O                   |
| High memory                 | `diagnose hardware sysinfo memory` |
| Low free memory             | Processes / sessions / proxy       |
| High swap                   | Memory pressure                    |
| New sessions failing        | Session setup rate / memory        |
| Routing problem             | Routing table / ARP                |
| FortiGuard issue            | `diagnose debug rating`            |
| DNS issue                   | `show system dns`                  |
| NTP issue                   | `diagnose sys ntp status`          |
| Physical errors             | NIC diagnostics                    |
| Unknown packet path         | Sniffer + NPU checks               |
| Modem issue                 | `modemd` + `ppp` debug             |
| Local service issue         | `diagnose sys tcpsocks`            |
| Interface cannot be removed | CMDB reference check               |
| Complex unexplained issue   | TAC report                         |

---

# 48. Operational Best Practices

* [ ] Take periodic configuration backups.
* [ ] Monitor CPU and memory trends.
* [ ] Monitor session count and session setup rate.
* [ ] Monitor interface errors and drops.
* [ ] Verify FortiGuard connectivity.
* [ ] Keep accurate network topology documentation.
* [ ] Capture traffic only for the required duration.
* [ ] Disable NPU offloading only when necessary for troubleshooting.
* [ ] Restore acceleration settings after testing.
* [ ] Avoid blindly changing session timers.
* [ ] Avoid running heavy debug commands for long periods.
* [ ] Coordinate troubleshooting in VDOM environments with the appropriate administrator.
* [ ] Keep before/after diagnostic outputs for comparison.
* [ ] Provide Fortinet Support with a concise troubleshooting summary.

---

## 49. Golden Troubleshooting Rule

```text
Observe
   ↓
Measure
   ↓
Identify the processing path
   ↓
Check CPU / Memory / Session state
   ↓
Check Network / ARP / Routing
   ↓
Check NPU / Hardware acceleration
   ↓
Capture packets if required
   ↓
Change ONE variable
   ↓
Measure again
   ↓
Compare results
```

> **Do not troubleshoot by changing multiple parameters simultaneously.**
> Change one variable, measure the result, and keep a record of the before/after state.

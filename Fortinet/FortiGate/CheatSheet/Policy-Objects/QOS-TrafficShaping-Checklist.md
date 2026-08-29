# 🔗 SheynShield Resources

# FortiGate QoS, Traffic Shaping & DSCP Checklist

> **FortiOS Focus:** 7.x  
> **Level:** NSE 4 / NSE 7 / Network Security Engineer  
> **Category:** QoS · Traffic Shaping · DSCP · TOS · Queueing · WRED · Policing · Shaping · NPU Offload

---

# 📌 FortiGate QoS Implementation Checklist

## 1. QoS Fundamentals Checklist

- [ ] Understand QoS objectives:
  - [ ] Bandwidth management
  - [ ] Latency reduction
  - [ ] Jitter control
  - [ ] Packet loss management
  - [ ] Congestion handling

- [ ] Understand QoS lifecycle:

```text
Classification
        ↓
Marking
        ↓
Queueing
        ↓
Scheduling
        ↓
Shaping / Policing
        ↓
Forwarding
````

* [ ] Identify traffic priority:

```text
Voice
 ↓
Interactive Video
 ↓
Business Critical Applications
 ↓
Best Effort
 ↓
Bulk Traffic
```

---

# 2. Bandwidth Design Checklist

## Interface vs Access Rate

* [ ] Identify physical interface speed

Example:

```text
Physical Interface = 1Gbps
```

* [ ] Identify ISP committed bandwidth:

```text
ISP CIR = 100Mbps
```

* [ ] Do not confuse:

| Parameter | Purpose                  |
| --------- | ------------------------ |
| Link Rate | Physical capability      |
| Bandwidth | QoS calculation value    |
| CIR       | Guaranteed provider rate |
| PIR       | Maximum allowed rate     |

---

## WAN Bandwidth Checklist

* [ ] Confirm ISP CIR
* [ ] Confirm PIR
* [ ] Confirm actual usable bandwidth
* [ ] Shape traffic below ISP congestion point

Recommended:

```text
ISP CIR
   ↓
FortiGate Shaping
   ↓
ISP
```

---

# 3. Delay Analysis Checklist

Verify delay components:

* [ ] Serialization delay
* [ ] Propagation delay
* [ ] Processing delay
* [ ] Queueing delay
* [ ] Shaping delay
* [ ] ISP delay
* [ ] Codec delay

---

## Serialization Delay Formula

Verify:

```text
(Packet Size × 8) / Link Speed
```

Example:

```text
1500 byte packet
64Kbps link

=187.5ms
```

Checklist:

* [ ] Slow WAN identified
* [ ] Large packet impact evaluated
* [ ] Voice impact evaluated

---

# 4. Queueing Checklist

Verify:

* [ ] Queue size
* [ ] Queue overflow
* [ ] Tail drop behavior
* [ ] Bufferbloat risk
* [ ] Priority queue configuration

Remember:

```text
Large Queue
      ↓
Less Drop
      ↓
Higher Latency
```

---

# 5. Shaping vs Policing Checklist

## Traffic Shaping

Verify:

* [ ] Excess traffic is queued
* [ ] Packets are delayed
* [ ] Transmission rate is controlled

Model:

```text
Traffic
   ↓
Queue
   ↓
Controlled Transmission
```

---

## Policing

Verify:

* [ ] Rate enforcement
* [ ] Drop behavior
* [ ] Remarking behavior

Model:

```text
Traffic
   ↓
Rate Check
   ↓
Transmit
or
Drop/Remark
```

---

## Decision Checklist

| Requirement            | Solution |
| ---------------------- | -------- |
| Smooth traffic         | Shaping  |
| Enforce hard limit     | Policing |
| Protect WAN queue      | Shaping  |
| Drop excessive traffic | Policing |

---

# 6. Token Bucket Checklist

Understand:

* [ ] CIR
* [ ] PIR
* [ ] BC
* [ ] BE/CBE

Model:

```text
Token Generator
        ↓
 Token Bucket
        ↓
 Packet Transmission
```

Verify:

* [ ] Burst requirements
* [ ] Average rate
* [ ] Peak rate

---

# 7. DSCP / TOS Checklist

Verify packet marking:

* [ ] IPv4 TOS byte
* [ ] DSCP value
* [ ] ECN bits
* [ ] CoS mapping

Structure:

```text
IPv4 TOS Byte

┌───────────┬─────┐
│ DSCP 6bit │ECN  │
└───────────┴─────┘
```

---

# 8. DSCP Classification Checklist

Confirm:

* [ ] DSCP uses 6 bits

```text
2^6 = 64 values
```

* [ ] ECN uses 2 bits

---

## Important DSCP Values

| Traffic     | DSCP |
| ----------- | ---: |
| Best Effort |    0 |
| CS1         |    8 |
| AF21        |   18 |
| AF31        |   26 |
| AF41        |   34 |
| CS5         |   40 |
| EF Voice    |   46 |
| CS6         |   48 |
| CS7         |   56 |

---

# 9. Voice QoS Checklist

Verify:

* [ ] Voice classified correctly
* [ ] EF marking applied
* [ ] Priority queue configured
* [ ] Bandwidth reservation exists
* [ ] Jitter controlled
* [ ] Packet loss monitored

Recommended:

```text
Voice
=
DSCP EF
=
46
=
101110
```

---

# 10. FortiGate Traffic Shaping Checklist

Verify:

* [ ] Traffic shaper created

CLI:

```cli
config firewall shaper traffic-shaper
```

* [ ] Correct bandwidth configured
* [ ] Correct priority configured
* [ ] Correct policy attachment

---

# 11. Shared Shaper Checklist

Use when:

* [ ] Multiple users share one bandwidth pool
* [ ] Aggregate control is required

Example:

```text
Shared Shaper
       |
 ┌─────┼─────┐
User User User

Total = 200Mbps
```

Verify:

* [ ] Maximum bandwidth
* [ ] Priority
* [ ] Sharing mode

---

# 12. Per-IP Shaper Checklist

Use when:

* [ ] User fairness required
* [ ] Guest networks controlled
* [ ] One client must not consume all bandwidth

Example:

```text
Client A = 10Mbps
Client B = 10Mbps
Client C = 10Mbps
```

Verify:

* [ ] Source IP tracking
* [ ] Individual limit
* [ ] User impact

---

# 13. Traffic Direction Checklist

Always verify direction:

## Upload

```text
LAN
 ↓
FortiGate
 ↓
WAN
```

Use:

```text
Outbound shaping
```

---

## Download

```text
WAN
 ↓
FortiGate
 ↓
LAN
```

Use:

```text
Reverse shaping
```

---

# 14. Traffic Shaping Profile Checklist

Verify:

* [ ] Class ID
* [ ] Guaranteed bandwidth
* [ ] Maximum bandwidth
* [ ] Priority
* [ ] Burst
* [ ] C-Burst
* [ ] RED/WRED

Example:

```text
Class 10 = Voice
Class 20 = Business
Class 30 = Default
```

---

# 15. Guaranteed vs Maximum Bandwidth Checklist

Verify:

Example:

```text
WAN = 1Gbps

Voice
Guaranteed = 50%
Maximum    = 100%
```

Check:

* [ ] Total guaranteed bandwidth <= link capacity
* [ ] Maximum bandwidth policy is realistic

---

# 16. Burst Configuration Checklist

Verify:

* [ ] Burst interval
* [ ] C-Burst interval
* [ ] Average rate
* [ ] Temporary traffic spikes

Remember:

```text
Burst
=
Temporary bandwidth increase
```

---

# 17. RED / WRED Checklist

Verify:

## RED

* [ ] Early packet dropping enabled
* [ ] Queue congestion controlled

## WRED

* [ ] Different drop behavior per class
* [ ] DSCP drop precedence considered

Model:

```text
Low Priority
      ↓
Drop Earlier

High Priority
      ↓
Keep Longer
```

---

# 18. DSCP Marking Checklist on FortiGate

Verify firewall policy:

```cli
config firewall policy
edit 1

set diffserv-forward enable
set diffservcode-forward 101110

end
```

Check:

* [ ] Forward marking
* [ ] Reverse marking
* [ ] DSCP preservation

---

# 19. DSCP Matching Checklist

Verify:

DSCP mask:

```text
0xFC
```

Meaning:

```text
11111100

DSCP checked
ECN ignored
```

Example:

```cli
set tos 0xb8
set tos-mask 0xfc
```

Matches:

```text
EF = DSCP 46
```

---

# 20. Multi-Stage DSCP Checklist

Verify:

* [ ] Normal DSCP
* [ ] Exceed DSCP
* [ ] Maximum DSCP

Concept:

```text
Normal Usage
      ↓
Normal DSCP

High Usage
      ↓
Remark DSCP

Extreme Usage
      ↓
Drop Priority
```

---

# 21. NPU Offload Checklist

During QoS troubleshooting:

* [ ] Check ASIC offload
* [ ] Check NPU acceleration
* [ ] Disable temporarily if required

Example:

```cli
config firewall policy

edit 1

set auto-asic-offload disable

next
end
```

---

# 22. FortiGate QoS Verification Checklist

Collect:

```cli
show firewall policy

show firewall shaping-policy

show firewall shaper traffic-shaper

show firewall shaping-profile

show system interface
```

---

# 23. Debug Checklist

## Session

```cli
diagnose sys session list
```

---

## Flow Debug

```cli
diagnose debug flow show function-name enable

diagnose debug flow show iprope enable

diagnose debug flow trace start 100

diagnose debug enable
```

Stop:

```cli
diagnose debug disable

diagnose debug reset
```

---

# 24. Queue Verification Checklist

Check:

```cli
diagnose netlink intf-class list port3

diagnose netlink intf-qdis list port3
```

Verify:

* [ ] Queue assignment
* [ ] Class mapping
* [ ] Packet counters

---

# 25. QoS Troubleshooting Workflow

```text
[ ] Confirm ISP CIR

[ ] Confirm physical speed

[ ] Confirm shaping direction

[ ] Confirm firewall policy match

[ ] Confirm shaping policy match

[ ] Confirm class ID

[ ] Confirm shaper attachment

[ ] Confirm bandwidth values

[ ] Confirm DSCP marking

[ ] Confirm downstream trust

[ ] Check queue statistics

[ ] Check NPU offload

[ ] Capture packets

[ ] Verify DSCP on wire
```

---

# 26. Production QoS Design Checklist

## WAN

* [ ] Know ISP CIR
* [ ] Shape below provider congestion point
* [ ] Identify bottleneck location
* [ ] Monitor utilization

---

## Voice

* [ ] Mark EF
* [ ] Reserve bandwidth
* [ ] Configure priority
* [ ] Control jitter
* [ ] Consider CAC
* [ ] Consider LFI

---

## Users

* [ ] Shared shaper for aggregate limits
* [ ] Per-IP shaper for fairness
* [ ] Prevent bandwidth abuse

---

# 27. NSE Exam Quick Recall

```text
Shaping
=
Queue + Delay


Policing
=
Drop / Remark


DSCP
=
6 Bits


ECN
=
2 Bits


EF
=
46


EF TOS
=
0xB8


DSCP Mask
=
0xFC


CoS
=
3 Bits


IP Precedence
=
3 Bits
```

---

# 28. Golden Rules

* [ ] Shape where you control congestion
* [ ] Policing does not create queues
* [ ] DSCP alone does not create QoS
* [ ] EF is commonly used for voice
* [ ] Large queues increase latency
* [ ] Verify QoS direction carefully
* [ ] Always check hardware acceleration
* [ ] Protect critical applications first

---

# 29. Final QoS Architecture

```text
Traffic
   ↓
Classification
   ↓
Marking
   ↓
Queueing
   ↓
Priority/Fairness
   ↓
Shaping/Policing
   ↓
Controlled Forwarding
```

---

# ⭐ Core Takeaway

A successful FortiGate QoS design answers:

```text
1. What traffic exists?

2. How is it classified?

3. How is it marked?

4. What happens during congestion?

5. Where is the queue controlled?
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

> **SheynShield | Engineering Secure Networks**
> FortiGate · QoS · Traffic Shaping · DSCP · Network Security


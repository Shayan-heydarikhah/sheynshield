# FortiGate Diagnose — Packet Capture & Debug Flow 

> **FortiOS 7.2.0 | Troubleshooting | Packet Sniffer | Debug Flow | Packet Capture | Traffic Analysis**

---

## 🧠 Diagnose Mindset

When troubleshooting FortiGate traffic, first identify **what you actually need to observe**.

```text
                TRAFFIC PROBLEM
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
       Packet Problem       FortiGate Decision
             │                   │
             ▼                   ▼
      Packet Capture         Debug Flow
             │                   │
             ▼                   ▼
     Header / Payload       Route / Policy /
       / Packet Flow         Action / Egress
```

### Quick Rule

> **Packet Sniffer → What is actually on the wire?**

> **Debug Flow → What is FortiOS doing with the packet?**

---

# 📡 Packet Capture / Packet Sniffer

## What Does It Do?

Packet capture allows you to observe packet streams in real time.

It can help you inspect:

* Packet headers
* Packet information
* Protocols
* Source / destination
* Packet direction
* Payload information
* Real-time traffic behavior

Captured traffic can also be stored as:

```text
.pcap
```

and analyzed with packet-analysis tools.

---

# 🔍 Packet Sniffer — Basic Command

```bash
diagnose sniffer packet port3 'tcp' 1 100 a
```

### Command Structure

```text
diagnose sniffer packet <interface> '<filter>' <verbose> <count> <output>
```

Example:

```text
diagnose sniffer packet port3 'tcp' 1 100 a
```

| Parameter | Meaning                  |
| --------- | ------------------------ |
| `port3`   | Interface to capture     |
| `'tcp'`   | Capture filter           |
| `1`       | Verbosity level          |
| `100`     | Number of packets        |
| `a`       | Display/transform option |

---

# 🎯 Packet Sniffer — Mental Model

```text
                    FortiGate
                       │
                       ▼
                 ┌───────────┐
                 │  port3    │
                 └───────────┘
                       │
                       ▼
                Packet Stream
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
          Header              Payload
             │                   │
             └─────────┬─────────┘
                       ▼
                  Troubleshoot
```

The sniffer answers:

> **"What packets are actually being seen on this interface?"**

---

# 🧪 Example — TCP Capture

```bash
diagnose sniffer packet port3 'tcp' 1 100 a
```

Conceptually:

```text
Interface
   │
   ▼
  port3
   │
   ▼
TCP traffic
   │
   ▼
Capture 100 packets
```

---

# 📦 Why PCAP Matters

For deeper analysis, packet captures can be saved as:

```text
.pcap
```

This allows you to inspect traffic at packet level.

Typical analysis:

```text
Ethernet
   ↓
IP
   ↓
TCP / UDP
   ↓
Application Protocol
   ↓
Payload
```

### Useful Questions

When analyzing a capture, ask:

* Did the packet arrive?
* What is the source IP?
* What is the destination IP?
* Which protocol is being used?
* Which TCP/UDP port is involved?
* Did the server respond?
* Is the return packet present?
* Is the packet being retransmitted?
* Is there a TCP handshake?
* Is the expected payload present?

---

# 🧭 Packet Sniffer vs Debug Flow

| Question                             | Tool               |
| ------------------------------------ | ------------------ |
| Did the packet arrive at FortiGate?  | **Packet Sniffer** |
| What does the packet look like?      | **Packet Sniffer** |
| Is TCP/UDP traffic present?          | **Packet Sniffer** |
| Did the server respond?              | **Packet Sniffer** |
| Which firewall policy is selected?   | **Debug Flow**     |
| Which route is selected?             | **Debug Flow**     |
| Which interface is used for egress?  | **Debug Flow**     |
| What action does FortiOS take?       | **Debug Flow**     |
| Why is traffic being denied/dropped? | **Debug Flow**     |

---

# 🔥 Debug Flow

## What Does Debug Flow Do?

Debug Flow traces packet processing through FortiOS.

It helps you understand how FortiGate makes forwarding and security decisions.

```text
Incoming Packet
      │
      ▼
 Packet Processing
      │
      ├── Security Checks
      │
      ├── Routing
      │
      ├── Firewall Policy
      │
      ├── Action
      │
      └── Egress Interface
      │
      ▼
Outgoing / Dropped
```

---

# 🧠 What Can Debug Flow Tell You?

Debug Flow can help identify:

### 🔐 Security

Which security processing path is involved?

```text
Packet
  ↓
Security Processing
```

### 🛡️ Firewall Policy

Which policy is being used?

```text
Packet
  ↓
Policy Lookup
  ↓
Policy ID
```

### 🛣️ Routing

Which route is selected?

```text
Destination IP
      ↓
Routing Lookup
      ↓
Next Hop
```

### 🚪 Egress Interface

Where does FortiGate send the packet?

```text
Routing Decision
      ↓
Egress Interface
```

### ⚡ Packet Action

What happens to the packet?

```text
ACCEPT
DROP
DENY
FORWARD
```

---

# 🔎 Debug Flow — Basic Filter

Example:

```bash
diagnose debug flow filter saddress port3
```

The purpose of a debug-flow filter is to limit troubleshooting to relevant traffic.

> ⚠️ In practice, use packet-specific filters whenever possible so that unrelated traffic does not make the debug output difficult to interpret.

---

# 🧩 Debug Flow Mental Model

```text
                   PACKET
                     │
                     ▼
              ┌──────────────┐
              │   FortiOS    │
              │ Packet Flow  │
              └──────┬───────┘
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       Security    Routing    Policy
          │          │          │
          └──────────┼──────────┘
                     ▼
                 Decision
                     │
              ┌──────┴──────┐
              ▼             ▼
           Forward         Drop
              │
              ▼
        Egress Interface
```

---

# 🚦 Troubleshooting Workflow

When a user says:

> **"I cannot reach the server."**

Don't immediately jump into random commands.

Use a structured workflow.

---

## Step 1 — Verify the Packet Exists

Use Packet Sniffer.

```bash
diagnose sniffer packet port3 'tcp' 1 100 a
```

Question:

```text
Did the packet reach FortiGate?
```

If **NO**:

```text
Client
  │
  X
  │
FortiGate
```

Investigate the client, switching, VLAN, interface or upstream path.

If **YES**:

```text
Client
  │
  ▼
FortiGate
```

Continue.

---

# Step 2 — Trace FortiOS Processing

Use Debug Flow.

```bash
diagnose debug flow filter saddress <source>
```

Now investigate:

```text
Packet
  │
  ├── Security
  ├── Routing
  ├── Policy
  ├── Action
  └── Egress
```

---

# Step 3 — Check the Route

Ask:

> **Where does FortiGate believe the destination should go?**

```text
Destination
     │
     ▼
Routing Table
     │
     ▼
Next Hop
     │
     ▼
Egress Interface
```

---

# Step 4 — Check the Firewall Policy

Ask:

```text
Which policy matched?
```

Then verify:

```text
Source Interface
Destination Interface
Source
Destination
Service
Action
NAT
```

---

# Step 5 — Check the Egress

Debug Flow should help establish where FortiGate attempts to send the packet.

```text
Ingress
   │
   ▼
FortiGate
   │
   ▼
Routing
   │
   ▼
Egress Interface
```

---

# 🔥 The Most Important Difference

## Packet Sniffer

```text
"What is happening to the packet?"
```

Think:

```text
WIRE
 │
 ▼
PACKET
 │
 ├── Header
 ├── Protocol
 ├── Source
 ├── Destination
 └── Payload
```

---

## Debug Flow

```text
"What is FortiOS doing with the packet?"
```

Think:

```text
PACKET
  │
  ▼
FORTIOS
  │
  ├── Security
  ├── Route
  ├── Policy
  ├── Action
  └── Egress
```

---

# 🧠 NSE Memory Trick

```text
SNIFFER
   ↓
SEE THE PACKET

DEBUG FLOW
   ↓
SEE THE DECISION
```

Or simply:

> **Sniffer = Packet Visibility**

> **Debug Flow = FortiOS Decision Visibility**

---

# 🛠️ Practical Troubleshooting Matrix

| Symptom                          | Start With     | Why                       |
| -------------------------------- | -------------- | ------------------------- |
| No traffic visible               | Packet Sniffer | Verify packet arrival     |
| TCP handshake problem            | Packet Sniffer | Inspect SYN/SYN-ACK       |
| Wrong destination path           | Debug Flow     | Inspect routing           |
| Unexpected firewall behavior     | Debug Flow     | Identify policy/action    |
| Unknown egress interface         | Debug Flow     | Trace forwarding decision |
| Server doesn't respond           | Packet Sniffer | Verify return traffic     |
| Packet arrives but is dropped    | Debug Flow     | Identify FortiOS decision |
| Application behavior is strange  | Packet Sniffer | Inspect protocol exchange |
| Need packet-level evidence       | Packet Sniffer | Capture traffic           |
| Need FortiOS processing evidence | Debug Flow     | Trace packet flow         |

---

# ⚠️ Common Troubleshooting Mistake

A common mistake is using only one tool.

```text
             PROBLEM
                │
        ┌───────┴───────┐
        ▼               ▼
    Sniffer          Debug Flow
        │               │
        ▼               ▼
   Packet View     Decision View
```

They answer different questions.

### Better Approach

```text
1. Sniff
   ↓
2. Confirm packet arrival
   ↓
3. Debug Flow
   ↓
4. Check policy + route + action
   ↓
5. Sniff again
   ↓
6. Verify actual forwarding / response
```

---

# 🎯 Expert Troubleshooting Mindset

When troubleshooting FortiGate traffic, divide the problem into **three layers**:

```text
┌───────────────────────────────┐
│ 1. PACKET                     │
│ Did the traffic actually      │
│ arrive / leave?               │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│ 2. DECISION                   │
│ What did FortiOS decide?      │
│ Policy / Route / Action       │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│ 3. RETURN TRAFFIC             │
│ Did the response come back?   │
└───────────────────────────────┘
```

This prevents troubleshooting from becoming guesswork.

---

# 🚀 Quick Reference

## Packet Capture

```bash
diagnose sniffer packet port3 'tcp' 1 100 a
```

**Use when you need to see the traffic itself.**

```text
Packet
 ↓
Header
 ↓
Protocol
 ↓
Payload
```

---

## Debug Flow

```bash
diagnose debug flow filter saddress <source>
```

**Use when you need to understand FortiOS packet processing.**

```text
Packet
 ↓
Security
 ↓
Policy
 ↓
Route
 ↓
Action
 ↓
Egress
```

---

# 🔑 Final Cheat Sheet

```text
                  FORTIGATE TROUBLESHOOTING
                            │
             ┌──────────────┴──────────────┐
             │                             │
             ▼                             ▼
       PACKET SNIFFER                 DEBUG FLOW
             │                             │
             ▼                             ▼
       "What is on the wire?"       "What does FortiOS do?"
             │                             │
       ┌─────┼─────┐               ┌──────┼──────┐
       ▼     ▼     ▼               ▼      ▼      ▼
     Header TCP  Payload         Policy Route  Action
       │     │     │               │      │      │
       └─────┴─────┘               └──────┴──────┘
             │                             │
             ▼                             ▼
       PACKET EVIDENCE               DECISION EVIDENCE
```

> **Remember:** If you don't know whether the packet exists, **sniff first**.
> If you know the packet exists but don't know why FortiGate handles it a certain way, **debug the flow**.

---

## 🔑 Keywords

`FortiGate Diagnose` · `FortiOS Troubleshooting` · `diagnose sniffer packet` · `FortiGate Packet Capture` · `FortiGate Packet Sniffer` · `PCAP` · `FortiGate Debug Flow` · `diagnose debug flow` · `FortiOS Packet Flow` · `FortiGate Firewall Policy Troubleshooting` · `FortiGate Routing Troubleshooting` · `FortiGate Network Troubleshooting` · `FortiOS 7.2.0` · `NSE4` · `NSE7`

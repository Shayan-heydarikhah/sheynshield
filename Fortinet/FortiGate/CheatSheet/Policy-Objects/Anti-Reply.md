# FortiGate Anti-Replay & TCP Sequence Validation  

> **FortiOS | Anti-Replay Protection, TCP Sequence Numbers, Packet Replay & Stateful Inspection Lab**

---

## 📌 Table of Contents

* [1. What Is Anti-Replay?](#1-what-is-anti-replay)
* [2. Why TCP Sequence Numbers Matter](#2-why-tcp-sequence-numbers-matter)
* [3. Lab Topology](#3-lab-topology)
* [4. Lab Address Objects](#4-lab-address-objects)
* [5. Firewall Policy Design](#5-firewall-policy-design)
* [6. Generate Legitimate Traffic](#6-generate-legitimate-traffic)
* [7. Capture Traffic](#7-capture-traffic)
* [8. Replay the Captured Traffic](#8-replay-the-captured-traffic)
* [9. Test Anti-Replay](#9-test-anti-replay)
* [10. Disable Anti-Replay for Comparison](#10-disable-anti-replay-for-comparison)
* [11. Expected Behavior](#11-expected-behavior)
* [12. Why the Replay Is Blocked](#12-why-the-replay-is-blocked)
* [13. Troubleshooting](#13-troubleshooting)
* [14. Security Implications](#14-security-implications)
* [15. NSE High-Value Notes](#15-nse-high-value-notes)
* [16. Quick Reference](#16-quick-reference)

---

# 1. What Is Anti-Replay?

**Anti-Replay** is a security mechanism used to detect and reject packets that appear to be **replayed or inconsistent with the expected state of an existing session**.

For TCP, FortiGate can use TCP state information and sequence numbers when validating traffic.

Conceptually:

```text
Legitimate TCP Session
        │
        ▼
TCP State + Sequence Numbers
        │
        ▼
Expected Packet
        │
        ▼
ALLOW
```

If an old/captured packet is replayed:

```text
Captured Packet
      │
      ▼
Replay
      │
      ▼
FortiGate
      │
      ▼
TCP State / Sequence Validation
      │
      ▼
Unexpected / Replay Traffic
      │
      ▼
DROP
```

> 🧠 **Core idea:** A packet can have a valid TCP structure but still be invalid for the **current session state**.

---

# 2. Why TCP Sequence Numbers Matter

TCP uses sequence numbers to track the position of data inside a connection.

A simplified TCP exchange:

```text
Client                         Server
  │                              │
  │──── SYN ────────────────────>│
  │<─── SYN/ACK ─────────────────│
  │──── ACK ────────────────────>│
  │                              │
  │──── DATA Seq=1000 ──────────>│
  │<─── ACK Seq=... ─────────────│
```

FortiGate also maintains session state.

Therefore:

```text
Packet
  +
TCP Sequence Number
  +
TCP Session State
  ↓
Session Validation
```

A previously captured packet may contain a sequence number that was valid **at the time of capture**, but invalid when replayed later.

---

# 3. Lab Topology

Use an isolated test environment.

```text
                 FORTIGATE
              ┌─────────────┐
              │             │
       LAN    │             │    DMZ
              │             │
              └──────┬──────┘
                     │
             ┌───────┴────────┐
             │                │
          Kali Linux       Normal PC
        192.168.101.3    192.168.101.2
             │
             │
             ▼
        Test Web Server
         192.168.20.200
              IIS
              :80
```

### Lab Hosts

| Host       | IP               | Role               |
| ---------- | ---------------- | ------------------ |
| Normal PC  | `192.168.101.2`  | Legitimate client  |
| Kali Linux | `192.168.101.3`  | Replay test client |
| IIS Server | `192.168.20.200` | Test HTTP server   |

> ⚠️ Perform packet replay only in an authorized, isolated lab. Do not replay captured traffic against systems you do not own or administer.

---

# 4. Lab Address Objects

Create two address objects.

### Normal PC

```text
Name:
NORMAL-PC

IP:
192.168.101.2/24
```

### Kali Linux

```text
Name:
KALI-LINUX

IP:
192.168.101.3/24
```

Conceptually:

```text
Address Objects
│
├── NORMAL-PC
│      └── 192.168.101.2
│
└── KALI-LINUX
       └── 192.168.101.3
```

---

# 5. Firewall Policy Design

The policy order is important because FortiGate evaluates policies according to policy matching/order.

Recommended lab structure:

```text
Policy 1
Kali → DMZ
Action = ACCEPT

Policy 2
Kali → DMZ
Action = DENY

Policy 3
Normal PC → DMZ
Action = ACCEPT
```

Conceptually:

```text
             Incoming Traffic
                    │
                    ▼
              ┌───────────┐
              │ Policy #1 │
              │ Kali      │
              │ ACCEPT    │
              └─────┬─────┘
                    │
              Policy Disabled
                    │
                    ▼
              ┌───────────┐
              │ Policy #2 │
              │ Kali      │
              │ DENY      │
              └───────────┘


Normal PC
   │
   ▼
Policy #3
   │
   ▼
ACCEPT
   │
   ▼
DMZ
```

This makes it possible to compare:

```text
Kali + ACCEPT
       vs
Kali + DENY
```

while the legitimate PC remains allowed.

---

# 6. Generate Legitimate Traffic

From the Normal PC, access the IIS server:

```text
http://192.168.20.200
```

This creates legitimate TCP/HTTP traffic.

From Kali:

```bash
curl http://192.168.20.200
```

Verify that the traffic is working before beginning the replay test.

---

# 7. Capture Traffic

On the Kali test system:

```bash
sudo tcpdump -i eth0 \
  -w legit_traffic.pcap \
  host 192.168.20.200 and port 80
```

This creates:

```text
legit_traffic.pcap
```

Conceptually:

```text
Kali
 │
 │ HTTP/TCP Traffic
 ▼
tcpdump
 │
 ▼
legit_traffic.pcap
```

The PCAP contains packets from a legitimate session.

---

# 8. Replay the Captured Traffic

In the isolated lab, the captured packets can be replayed to reproduce the traffic pattern.

Example:

```bash
sudo tcpreplay \
  -i eth0 \
  --loop=100 \
  --unique-ip \
  legit_traffic.pcap
```

### What is being demonstrated?

```text
Original Traffic
      │
      ▼
Captured PCAP
      │
      ▼
Repeated Transmission
      │
      ▼
FortiGate
      │
      ▼
Session / TCP Validation
```

The important point is that **replaying a packet capture does not recreate the original TCP session state**.

---

# 9. Test Anti-Replay

By default, Anti-Replay protection is enabled in the relevant FortiGate behavior.

The test sequence:

```text
1. Generate legitimate HTTP traffic
2. Capture traffic
3. Replay the PCAP
4. Observe FortiGate behavior
5. Disable Policy #1
6. Allow Policy #2 to match
7. Replay again
8. Compare the results
```

---

## Test #1 — Policy 1 Enabled

```text
Kali
 │
 ▼
Policy #1
 │
 ▼
ACCEPT
 │
 ▼
DMZ
```

Generate normal traffic:

```bash
curl http://192.168.20.200
```

Then perform the controlled replay.

Observe:

```text
Session table
Traffic logs
Packet capture
TCP behavior
```

---

# 10. Disable Anti-Replay for Comparison

The purpose of this experiment is to compare behavior with Anti-Replay enabled versus disabled.

> ⚠️ Only disable security protections temporarily in the isolated lab. Restore the protection immediately after testing.

Repeat the same test procedure after disabling the relevant Anti-Replay protection.

Then compare:

```text
Anti-Replay ENABLED
        vs
Anti-Replay DISABLED
```

The important learning objective is:

```text
Same PCAP
   +
Same Replay
   +
Different Security State
   ↓
Different Packet Handling
```

---

# 11. Expected Behavior

The expected behavior should be analyzed at the **packet/session level**, not merely by looking at whether the firewall policy says `ACCEPT`.

### Anti-Replay Enabled

```text
Replay Packet
     │
     ▼
Policy Match
     │
     ▼
TCP / Session Validation
     │
     ▼
Replay / Invalid Sequence
     │
     ▼
DROP
```

Therefore:

> **Policy ACCEPT does not necessarily mean every packet will be forwarded.**

There are multiple layers of packet processing.

---

## Important Mental Model

```text
             Packet
                │
                ▼
          Policy Matching
                │
                ▼
             ACCEPT
                │
                ▼
       Session / Protocol Checks
                │
                ▼
          Security Validation
                │
          ┌─────┴─────┐
          ▼           ▼
        VALID       INVALID
          │           │
          ▼           ▼
        FORWARD      DROP
```

This distinction is extremely important during troubleshooting.

---

# 12. Why the Replay Is Blocked

Suppose the original packet contained:

```text
TCP Sequence = X
```

At the time of the original connection:

```text
Session State
     +
Seq = X
     ↓
VALID
```

Later, the same packet is replayed:

```text
Old Packet
Seq = X
     │
     ▼
Current Session
     │
     ▼
Sequence / State mismatch
     │
     ▼
Potential Replay Detection
     │
     ▼
DROP
```

The packet is not necessarily malicious because its syntax is wrong.

Instead:

> **Its state is no longer consistent with the current TCP session.**

---

# 13. Troubleshooting

When investigating Anti-Replay behavior, don't start by assuming the firewall policy is the problem.

Use this sequence:

```text
                 Packet Dropped
                       │
                       ▼
                Which policy?
                       │
                       ▼
                Policy = ACCEPT?
                       │
                       ▼
                Session exists?
                       │
                       ▼
             TCP handshake observed?
                       │
                       ▼
              Sequence numbers valid?
                       │
                       ▼
                Replay suspected?
                       │
                       ▼
              Anti-Replay behavior
```

---

## Check the Session

```bash
diagnose sys session list
```

Look for:

* Source IP
* Destination IP
* Source port
* Destination port
* Protocol
* Policy ID
* Session state
* NAT information

---

## Capture the Traffic

Use a controlled packet capture to compare:

```text
Original Packet
       vs
Replayed Packet
```

Pay particular attention to:

```text
TCP SYN
TCP ACK
TCP Sequence Number
TCP Acknowledgment Number
TCP Flags
Window
Timestamp/options
```

---

# 14. Policy Order Matters

A common mistake is assuming that disabling the first policy automatically means traffic will be denied by the second policy.

The actual result depends on:

```text
Policy Order
+
Source
+
Destination
+
Service
+
Schedule
+
Other Matching Criteria
```

Example:

```text
Policy 1
KALI → DMZ
ACCEPT
   │
   │ Disabled
   ▼
Policy 2
KALI → DMZ
DENY
```

Now Kali traffic can reach Policy #2 if it matches the policy criteria.

---

# 15. Anti-Replay vs Firewall Policy

These are different security layers.

| Layer               | Question                                     |
| ------------------- | -------------------------------------------- |
| Policy              | Is this traffic permitted?                   |
| Session             | Does it belong to a valid session?           |
| TCP state           | Is the TCP state expected?                   |
| Sequence validation | Is the sequence consistent?                  |
| Anti-Replay         | Does the traffic appear replayed/unexpected? |

Therefore:

```text
ACCEPT Policy
      ≠
Guaranteed Forwarding
```

---

# 16. Security Implications

Anti-Replay helps protect stateful traffic processing against packets that are:

* Reused from previous traffic
* Out of expected session state
* Inconsistent with the current TCP flow
* Potentially replayed

Disabling such protections can reduce the strictness of state validation.

### Recommended approach

```text
Normal Production
      │
      ▼
Anti-Replay ENABLED
      │
      ▼
Normal Stateful Inspection
```

Only consider changing the behavior when there is a clearly understood compatibility requirement.

---

# 17. Lab Comparison Matrix

| Test         | Policy | Anti-Replay | Expected Focus            |
| ------------ | ------ | ----------- | ------------------------- |
| Normal HTTP  | ACCEPT | Enabled     | Baseline                  |
| Replay PCAP  | ACCEPT | Enabled     | Replay/session validation |
| Normal HTTP  | ACCEPT | Disabled    | Baseline comparison       |
| Replay PCAP  | ACCEPT | Disabled    | Compare relaxed behavior  |
| Kali traffic | DENY   | Enabled     | Policy enforcement        |
| Normal PC    | ACCEPT | Enabled     | Legitimate client         |

---

# 18. Important Distinction: Replay vs Asymmetric Routing

Do not confuse these two scenarios.

### Anti-Replay

```text
Same / old packet
       +
Unexpected TCP state
       ↓
Replay validation
```

### Asymmetric Routing

```text
Request
   ↓
Interface A

Reply
   ↓
Interface B
```

They can both produce confusing TCP behavior, but they are different problems.

---

# 19. NSE High-Value Notes 🧠

### TCP Sequence Numbers

```text
TCP
 ├── Sequence Number
 └── Acknowledgment Number
```

FortiGate can use TCP state/sequence information when processing stateful sessions.

---

### Policy ACCEPT ≠ Packet Guaranteed Forward

```text
Policy
  ↓
ACCEPT
  ↓
Session / Protocol Validation
  ↓
Security Checks
  ↓
Forward or Drop
```

---

### Replay Concept

```text
Valid Packet
     │
     ▼
Captured
     │
     ▼
Replayed Later
     │
     ▼
Session State Changed
     │
     ▼
Packet May Become Invalid
```

---

### Anti-Replay

```text
Anti-Replay
     ↓
Detect unexpected/replayed traffic
     ↓
Protect stateful processing
```

---

# 20. Golden Troubleshooting Workflow 🔥

When a packet appears to be blocked despite an `ACCEPT` policy:

```text
                 PACKET
                    │
                    ▼
             Policy Matching
                    │
                    ▼
                 ACCEPT?
                    │
                    ▼
             Session Matching
                    │
                    ▼
              TCP State Check
                    │
                    ▼
          Sequence / ACK Validation
                    │
                    ▼
             Security Inspection
                    │
                    ▼
                FORWARD?
               /        \
             YES         NO
              │           │
              ▼           ▼
           Success      Investigate
                         Drop Reason
```

---

# 21. Quick Reference

### Generate HTTP traffic

```bash
curl http://192.168.20.200
```

### Capture HTTP traffic

```bash
sudo tcpdump -i eth0 \
  -w legit_traffic.pcap \
  host 192.168.20.200 and port 80
```

### Replay in an isolated lab

```bash
sudo tcpreplay \
  -i eth0 \
  --loop=100 \
  --unique-ip \
  legit_traffic.pcap
```

### Inspect sessions

```bash
diagnose sys session list
```

---

# 🎯 Final Mental Model

```text
                    TCP TRAFFIC
                         │
                         ▼
                   Firewall Policy
                         │
                    ┌────┴────┐
                    │ ACCEPT  │
                    └────┬────┘
                         │
                         ▼
                   Session State
                         │
                         ▼
                TCP Seq / ACK Check
                         │
                         ▼
                    Anti-Replay
                         │
                  ┌──────┴──────┐
                  ▼             ▼
               VALID         REPLAY/
                              INVALID
                  │             │
                  ▼             ▼
               FORWARD         DROP
```

### 🔥 One-Line Memory Hook

```text
ACCEPT Policy
    ≠
Accept Every Packet

TCP Session
    +
Sequence / ACK State
    +
Anti-Replay
    ↓
Final Packet Decision
```

> **Production Best Practice:** Keep Anti-Replay protection enabled unless a specific, validated interoperability or architecture requirement justifies changing it. If a legitimate application is being dropped, first investigate the **session state, TCP sequence/ACK behavior, routing symmetry and packet path** before weakening the security control.
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

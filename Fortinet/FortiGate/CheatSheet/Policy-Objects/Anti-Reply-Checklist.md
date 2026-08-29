# 🔗 SheynShield Resources

# FortiGate Anti-Replay & TCP Sequence Validation Checklist

> **FortiOS Security | Anti-Replay Protection, TCP State Validation, Sequence Numbers, Stateful Inspection & Troubleshooting Lab**

---

# 📌 Table of Contents

- [1. Anti-Replay Concept Checklist](#1-anti-replay-concept-checklist)
- [2. TCP Sequence Validation Checklist](#2-tcp-sequence-validation-checklist)
- [3. Anti-Replay Lab Preparation Checklist](#3-anti-replay-lab-preparation-checklist)
- [4. Address Object Checklist](#4-address-object-checklist)
- [5. Firewall Policy Design Checklist](#5-firewall-policy-design-checklist)
- [6. Legitimate Traffic Generation Checklist](#6-legitimate-traffic-generation-checklist)
- [7. Packet Capture Checklist](#7-packet-capture-checklist)
- [8. Replay Traffic Testing Checklist](#8-replay-traffic-testing-checklist)
- [9. Anti-Replay Validation Checklist](#9-anti-replay-validation-checklist)
- [10. Policy ACCEPT vs Security Validation Checklist](#10-policy-accept-vs-security-validation-checklist)
- [11. Troubleshooting Workflow Checklist](#11-troubleshooting-workflow-checklist)
- [12. Replay vs Asymmetric Routing Checklist](#12-replay-vs-asymmetric-routing-checklist)
- [13. Security Best Practice Checklist](#13-security-best-practice-checklist)
- [14. NSE High-Value Notes](#14-nse-high-value-notes)
- [15. Quick Reference Commands](#15-quick-reference-commands)

---

# 1. Anti-Replay Concept Checklist

## What Is Anti-Replay?

Anti-Replay protects stateful traffic processing by detecting packets that are:

- [ ] Reused from previous sessions
- [ ] Out of expected session state
- [ ] Inconsistent with TCP sequence tracking
- [ ] Potentially replayed

---

## Core Mental Model

```text
Legitimate Packet

        |
        ▼

TCP State Validation

        |
        ▼

Sequence / ACK Check

        |
        ▼

ALLOW
````

Replay scenario:

```text
Captured Packet

        |
        ▼

Replay Attempt

        |
        ▼

FortiGate

        |
        ▼

Session Validation

        |
        ▼

DROP
```

---

## Key Concept

```text
Valid Packet Structure

        ≠

Valid Current Session State
```

Checklist:

* [ ] Understand packet validity
* [ ] Understand session validity
* [ ] Understand TCP state tracking

---

# 2. TCP Sequence Validation Checklist

TCP relies on:

```text
Sequence Number
+
Acknowledgment Number
+
Session State
```

---

## TCP Flow

```text
Client                         Server

 SYN  ----------------------->

      <---------------- SYN/ACK

 ACK  ----------------------->


 DATA Seq=1000 -------------->
```

---

## FortiGate Validation Logic

```text
Packet

 +

TCP Sequence Number

 +

TCP Session State

        ↓

Session Validation
```

---

Checklist:

* [ ] TCP handshake completed
* [ ] Sequence numbers understood
* [ ] ACK numbers validated
* [ ] Session state tracked

---

# 3. Anti-Replay Lab Preparation Checklist

## Lab Topology

```text
                 FORTIGATE

              ┌───────────┐
              │           │
      LAN     │           │     DMZ
              │           │
              └─────┬─────┘
                    |
        -------------------------
        |                       |
        ▼                       ▼

   Kali Linux              IIS Server

192.168.101.3          192.168.20.200
 Replay Client           HTTP Server
```

---

## Lab Requirement

* [ ] Isolated environment
* [ ] Authorized systems only
* [ ] Packet capture enabled
* [ ] Logging enabled
* [ ] Test traffic validated

---

## Lab Hosts

| Device     | IP               | Purpose           |
| ---------- | ---------------- | ----------------- |
| Normal PC  | `192.168.101.2`  | Legitimate client |
| Kali Linux | `192.168.101.3`  | Replay testing    |
| IIS Server | `192.168.20.200` | HTTP server       |

---

# 4. Address Object Checklist

## Normal Client

Create:

```text
Name:

NORMAL-PC


IP:

192.168.101.2/24
```

Checklist:

* [ ] Address object created
* [ ] Correct subnet configured
* [ ] Policy reference verified

---

## Replay Client

Create:

```text
Name:

KALI-LINUX


IP:

192.168.101.3/24
```

Checklist:

* [ ] Kali object created
* [ ] Replay source identified

---

# 5. Firewall Policy Design Checklist

## Policy Order Validation

Example:

```text
Policy 1

Kali → DMZ

ACCEPT


Policy 2

Kali → DMZ

DENY


Policy 3

Normal PC → DMZ

ACCEPT
```

---

Checklist:

* [ ] Policy order reviewed
* [ ] Source matching verified
* [ ] Destination matching verified
* [ ] Service matching verified
* [ ] Schedule verified

---

## Policy Decision Flow

```text
Traffic

  |

  ▼

Policy Match

  |

  ▼

ACCEPT / DENY

  |

  ▼

Session Validation

  |

  ▼

Security Processing
```

---

# 6. Legitimate Traffic Generation Checklist

Before replay testing:

* [ ] Confirm normal traffic works
* [ ] Confirm server availability
* [ ] Confirm TCP session creation

---

Example:

```bash
curl http://192.168.20.200
```

---

Expected:

```text
Client

↓

FortiGate

↓

HTTP Server

↓

Response
```

---

# 7. Packet Capture Checklist

Capture legitimate traffic:

```bash
sudo tcpdump -i eth0 \
-w legit_traffic.pcap \
host 192.168.20.200 and port 80
```

---

Validation:

* [ ] PCAP file created
* [ ] TCP handshake captured
* [ ] HTTP packets captured
* [ ] Sequence numbers visible

---

Captured flow:

```text
Kali

 |

TCP Traffic

 |

tcpdump

 |

legit_traffic.pcap
```

---

# 8. Replay Traffic Testing Checklist

## Replay Requirements

* [ ] Use isolated lab
* [ ] Confirm authorization
* [ ] Understand expected behavior
* [ ] Monitor FortiGate logs

---

Example:

```bash
sudo tcpreplay \
-i eth0 \
--loop=100 \
--unique-ip \
legit_traffic.pcap
```

---

Replay Flow:

```text
Original Traffic

↓

PCAP

↓

Replay

↓

FortiGate

↓

TCP Validation
```

---

# 9. Anti-Replay Validation Checklist

## Test Procedure

### Test 1

Normal traffic:

* [ ] Generate HTTP traffic
* [ ] Confirm session creation
* [ ] Capture packets

---

### Test 2

Replay:

* [ ] Replay captured PCAP
* [ ] Observe session table
* [ ] Review logs
* [ ] Compare packet behavior

---

### Test 3

Security Comparison:

```text
Anti-Replay ENABLED

        VS

Anti-Replay DISABLED
```

---

# 10. Policy ACCEPT vs Security Validation Checklist

Important concept:

```text
Firewall Policy ACCEPT

        ≠

Packet Always Forwarded
```

---

Packet Processing Model:

```text
Packet

 |

 ▼

Policy Matching

 |

 ▼

ACCEPT

 |

 ▼

Session Check

 |

 ▼

TCP Validation

 |

 ▼

Anti-Replay

 |

 ▼

FORWARD / DROP
```

---

Checklist:

* [ ] Policy hit confirmed
* [ ] Session exists
* [ ] TCP state valid
* [ ] Security checks passed

---

# 11. Troubleshooting Workflow Checklist

When packet is dropped:

```text
PACKET DROP

    |

    ▼

Which Policy?

    |

    ▼

Policy ACCEPT?

    |

    ▼

Session Exists?

    |

    ▼

TCP State Valid?

    |

    ▼

Sequence Numbers Valid?

    |

    ▼

Replay Detection?
```

---

## Session Verification

Command:

```bash
diagnose sys session list
```

Check:

* [ ] Source IP
* [ ] Destination IP
* [ ] Source Port
* [ ] Destination Port
* [ ] Protocol
* [ ] Policy ID
* [ ] NAT information
* [ ] Session state

---

## Packet Comparison

Compare:

```text
Original Packet

VS

Replay Packet
```

Review:

* [ ] TCP SYN
* [ ] TCP ACK
* [ ] Sequence Number
* [ ] ACK Number
* [ ] TCP Flags
* [ ] Window Size
* [ ] TCP Options

---

# 12. Replay vs Asymmetric Routing Checklist

Do not confuse:

## Anti-Replay

```text
Old Packet

+

Wrong Current Session State

↓

Replay Detection
```

---

## Asymmetric Routing

```text
Request

↓

Path A


Reply

↓

Path B
```

---

Validation:

* [ ] Routing symmetry checked
* [ ] Session path verified
* [ ] Packet direction analyzed

---

# 13. Security Best Practice Checklist

Production recommendation:

```text
Anti-Replay

        ↓

Enabled
```

---

Checklist:

* [ ] Keep Anti-Replay enabled
* [ ] Change only with valid requirement
* [ ] Test application compatibility
* [ ] Document exceptions
* [ ] Restore protection after testing

---

Avoid:

```text
Application Problem

        ↓

Disable Security Control Immediately
```

---

Prefer:

```text
Application Problem

        ↓

Analyze Session

        ↓

Analyze TCP State

        ↓

Analyze Routing

        ↓

Adjust Carefully
```

---

# 14. NSE High-Value Notes 🧠

## TCP Validation

Remember:

```text
TCP

├── Sequence Number

└── ACK Number
```

---

## Anti-Replay Logic

```text
Captured Packet

↓

Replay Later

↓

Session Changed

↓

Packet Invalid
```

---

## Policy Processing

```text
Policy ACCEPT

↓

Session Inspection

↓

Protocol Validation

↓

Security Decision
```

---

## Golden Rule

```text
ACCEPT Policy

Does Not Mean

Every Packet Is Forwarded
```

---

# 15. Quick Reference Commands

## Generate Traffic

```bash
curl http://192.168.20.200
```

---

## Capture Traffic

```bash
sudo tcpdump -i eth0 \
-w legit_traffic.pcap \
host 192.168.20.200 and port 80
```

---

## Replay Traffic

```bash
sudo tcpreplay \
-i eth0 \
--loop=100 \
--unique-ip \
legit_traffic.pcap
```

---

## Session Debug

```bash
diagnose sys session list
```

---

# 🔥 Final Mental Model

```text
                 TCP PACKET

                      |

                      ▼

              Firewall Policy

                      |

                 ACCEPT?

                      |

                      ▼

              Session Tracking

                      |

                      ▼

            TCP Seq / ACK Validation

                      |

                      ▼

               Anti-Replay Check

              ┌───────────────┐
              ▼               ▼

           VALID           INVALID

              |               |

              ▼               ▼

           FORWARD           DROP
```

---

# ⚡ 30 Second Review

```text
1.
Anti-Replay protects stateful sessions.


2.
TCP Sequence Numbers define packet validity.


3.
A packet can be syntactically correct but invalid for the current session.


4.
Policy ACCEPT is only one step in packet processing.


5.
Troubleshoot:
Packet → Policy → Session → TCP State → Anti-Replay
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

⭐ **SheynShield | Engineering Secure Networks**


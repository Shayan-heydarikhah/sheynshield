# 🔗 SheynShield Resources

# FortiGate Advanced Firewall Policies & Session Handling Checklist

> **FortiOS Focus:** 7.x  
> **Level:** NSE 4 / NSE 7  
> **Category:** Firewall Policy Engineering · Session Management · Capacity Planning  
> **Topics:** Maximum Values · DHCP Capacity · Advanced Policy · Negated Objects · Captive Portal Exemption · Asymmetric Routing · TCP Session Without SYN

---

# 📌 Checklist Index

- [ ] FortiGate Maximum Value Tables
- [ ] DHCP Server Capacity Planning
- [ ] Table Size Verification
- [ ] Bidirectional vs Unidirectional Inspection Understanding
- [ ] Reverse Path Behavior Analysis
- [ ] Advanced Firewall Policy Features
- [ ] Negated Policy Objects
- [ ] Captive Portal Exemption
- [ ] TCP Session Without SYN
- [ ] Asymmetric Routing Troubleshooting
- [ ] Security Impact Evaluation
- [ ] NSE Exam Review Points
- [ ] Production Design Validation

---

# 1. 📊 FortiGate Maximum Value Tables Checklist

## Understanding Platform Limits

- [ ] Identify FortiGate model limitations
- [ ] Verify FortiOS version limitations
- [ ] Review supported maximum values before deployment
- [ ] Validate:

```

Object Capacity
Policy Capacity
Interface Capacity
VDOM Capacity
DHCP Capacity
Session Capacity
Route Capacity

```

Architecture:

```

FortiGate Platform
|
▼
Maximum Value Tables
|
├── Objects
├── Policies
├── VDOMs
├── DHCP Servers
└── Sessions

```

> ⚠️ Never assume maximum values are identical between FortiGate models.

---

# 2. DHCP Server Capacity Checklist

## Capacity Validation

- [ ] Calculate DHCP requirements per VDOM
- [ ] Check available VDOM resources
- [ ] Validate platform limitation

Example:

```

DHCP Server Capacity

Per VDOM:
256 DHCP Servers

```

Calculation:

```

10 VDOMs
|
×
|
256 DHCP Servers

=
2560 Theoretical DHCP Objects

````

Validation:

- [ ] Confirm VDOM license availability
- [ ] Confirm hardware capability
- [ ] Confirm FortiOS support

---

# 3. 🔍 Table Size Verification Checklist

## CLI Verification

Run:

```bash
print tablesize
````

Check:

* [ ] Current object usage
* [ ] Maximum supported values
* [ ] Available capacity margin
* [ ] Future growth requirement

Capacity planning:

```
Requirement
      |
      ▼
Current Usage
      |
      ▼
print tablesize
      |
      ▼
Maximum Capacity
      |
      ▼
Growth Margin
```

---

# 4. 🔄 Bidirectional vs Unidirectional Inspection Checklist

## Stateful Inspection Validation

Verify:

* [ ] Understand protocol behavior
* [ ] Understand session creation
* [ ] Understand reply traffic handling

Normal TCP/HTTP flow:

```
Client
  |
  | Request
  ▼
FortiGate
  |
  ▼
Server

Server
  |
  | Response
  ▼
FortiGate
  |
  ▼
Client
```

Checklist:

* [ ] Session tracking enabled
* [ ] Return traffic belongs to existing session
* [ ] Policy/session matching verified

---

# 5. 🔁 Reverse Path Behavior Checklist

## Asymmetric Routing Validation

Scenario:

```
Request:

Client
 |
 ▼
Interface A
 |
 ▼
FortiGate
 |
 ▼
Server


Reply:

Server
 |
 ▼
Interface B
 |
 ▼
FortiGate
 |
 ▼
Client
```

Validate:

* [ ] Request interface identified
* [ ] Reply interface identified
* [ ] Existing session verified
* [ ] Session tuple checked

Important:

```
Request Interface
        ≠
Reply Interface
```

does not automatically mean:

```
DROP
```

Session state matters.

---

# 6. ⚙️ Advanced Firewall Policy Checklist

## Enable Feature Visibility

Navigate:

```
System
 |
 ▼
Feature Visibility
 |
 ▼
Advanced Policy
```

Verify:

* [ ] Advanced Policy enabled
* [ ] Additional policy options visible
* [ ] Policy behavior documented

---

# 7. 🚫 Negated Policy Objects Checklist

## Normal Matching

Example:

```
Source = TRUSTED_NET
```

Logic:

```
Source belongs to TRUSTED_NET
```

---

## Negated Matching

Example:

```
NOT Source = TRUSTED_NET
```

Logic:

```
Source does NOT belong to TRUSTED_NET
```

Checklist:

* [ ] Understand object inversion
* [ ] Verify source matching logic
* [ ] Verify destination matching logic
* [ ] Test policy order impact

---

# 8. 🔐 Captive Portal Exemption Checklist

Purpose:

Allow selected traffic/users to bypass captive portal processing.

Flow:

```
Client
 |
 ▼
Firewall Policy
 |
 ▼
Captive Portal Exemption
 |
 ▼
Traffic Allowed
```

Validate:

* [ ] Define exact source scope
* [ ] Define exact destination scope
* [ ] Document business reason
* [ ] Review authentication bypass impact

Avoid:

```
Source = ALL

Destination = ALL

Captive Portal Exempt = Enable
```

---

# 9. TCP Session Without SYN Checklist

## Normal TCP Handshake

Expected:

```
Client                 Server

 SYN  ---------------->

      <-------------- SYN/ACK

 ACK  ---------------->

Established Session
```

---

## Feature Purpose

Allows FortiGate to handle TCP traffic when SYN is not observed normally.

Use cases:

* [ ] Asymmetric routing
* [ ] Session path changes
* [ ] Existing connection visibility issues

---

# 10. 🛠️ TCP Session Without SYN Configuration Checklist

## Global Configuration

Enable:

```bash
config system settings
    set tcp-session-without-syn enable
end
```

Checklist:

* [ ] Requirement confirmed
* [ ] Routing reviewed first
* [ ] Security impact approved

---

## Firewall Policy Configuration

Example:

```bash
config firewall policy
    edit 1
        set tcp-session-without-syn all
    next
end
```

Verify:

* [ ] Applied only to required policies
* [ ] Not enabled globally without reason
* [ ] Change documented

---

# 11. ⚠️ TCP Without SYN Security Checklist

Understand impact:

```
Normal TCP Validation

SYN
 |
SYN/ACK
 |
ACK
 |
Session
```

Relaxed mode:

```
Observed TCP Traffic
        |
        ▼
More Flexible Session Handling
```

Security risks:

* [ ] Reduced TCP state validation
* [ ] Increased unexpected traffic acceptance possibility
* [ ] Reduced strict session control
* [ ] Higher troubleshooting complexity

Best practice:

```
Fix Routing
      |
      ▼
Fix Asymmetry
      |
      ▼
Only if required:
Enable TCP Without SYN
```

---

# 12. 🔎 Troubleshooting Workflow Checklist

## Step 1 — Validate Routing

Check:

* [ ] Request path
* [ ] Reply path
* [ ] Asymmetric routing existence

---

## Step 2 — Inspect Session

Command:

```bash
diagnose sys session list
```

Check:

* [ ] Source
* [ ] Destination
* [ ] NAT
* [ ] Interface
* [ ] Policy ID
* [ ] Session state

---

## Step 3 — Validate Policy Match

Confirm:

* [ ] Source address
* [ ] Destination address
* [ ] Service
* [ ] Schedule
* [ ] Action
* [ ] Policy order

---

## Step 4 — Capture Traffic

Command:

```bash
diagnose sniffer packet <interface> ...
```

Analyze:

```
SYN
SYN/ACK
ACK
DATA
```

---

# 13. 🧠 NSE Exam High Value Notes

## Maximum Values

Remember:

```bash
print tablesize
```

Meaning:

```
What can this FortiGate support?
```

---

## Advanced Policy

Location:

```
System
 |
 Feature Visibility
 |
 Advanced Policy
```

---

## Negation

Remember:

```
Object

vs

NOT Object
```

---

## Captive Portal Exemption

Remember:

```
Policy
 |
 Captive Portal Bypass
```

---

## TCP Without SYN

Remember:

```
Missing SYN
+
Asymmetric Routing
+
Relaxed TCP State
```

---

# 14. Production Design Checklist

Before enabling advanced session features:

* [ ] Verify routing design
* [ ] Prefer symmetric routing
* [ ] Avoid unnecessary TCP relaxation
* [ ] Limit policy scope
* [ ] Document exceptions
* [ ] Monitor security impact

---

# 15. Quick Reference Table

| Requirement               | Command / Location                   |
| ------------------------- | ------------------------------------ |
| View capacity limits      | `print tablesize`                    |
| Enable Advanced Policy    | Feature Visibility                   |
| Configure TCP Without SYN | `set tcp-session-without-syn enable` |
| Enable policy exception   | `set tcp-session-without-syn all`    |
| View sessions             | `diagnose sys session list`          |
| Packet capture            | `diagnose sniffer packet`            |

---

# 🎯 Final Mental Model

```
                 FORTIGATE
                     |
     ┌───────────────┼───────────────┐
     |               |               |
     ▼               ▼               ▼

 Capacity       Policy Logic      Sessions

     |               |               |
     ▼               ▼               ▼

tablesize      Advanced Policy   TCP State

                |
        ┌───────┴────────┐
        ▼                ▼

     Negation       Captive Portal


                         |
                         ▼

                 TCP Without SYN

                         |
                         ▼

              Asymmetric Routing
```

---

# 🔥 One Minute Memory Hook

```
print tablesize
        |
        └── Platform capacity


Advanced Policy
        |
        ├── Negated Objects
        └── Captive Portal Exemption


Existing Session
        |
        └── Reply path behavior matters


tcp-session-without-syn
        |
        ├── Asymmetric routing
        ├── Missing SYN handling
        └── Use only when required
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

> **SheynShield Engineering Note**
> Advanced FortiGate engineering requires understanding the relationship between **capacity planning**, **policy logic**, and **session state behavior**. Always solve routing problems before relaxing firewall state validation.

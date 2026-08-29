# FortiGate Advanced Firewall Policies & Session Handling  

> **FortiOS | Policy Advanced Features, Object Capacity, Asymmetric Routing & TCP Sessions Without SYN**

---

## 📌 Table of Contents

* [1. FortiGate Maximum Value Tables](#1-fortigate-maximum-value-tables)
* [2. DHCP Server Capacity](#2-dhcp-server-capacity)
* [3. Checking Table Sizes](#3-checking-table-sizes)
* [4. Bidirectional vs Unidirectional Inspection](#4-bidirectional-vs-unidirectional-inspection)
* [5. FortiGate Reverse-Path Behavior](#5-fortigate-reverse-path-behavior)
* [6. Advanced Firewall Policy Features](#6-advanced-firewall-policy-features)
* [7. Negated Policy Objects](#7-negated-policy-objects)
* [8. Captive Portal Exemption](#8-captive-portal-exemption)
* [9. TCP Sessions Without SYN](#9-tcp-sessions-without-syn)
* [10. Asymmetric Routing Scenario](#10-asymmetric-routing-scenario)
* [11. Configuration](#11-configuration)
* [12. Why TCP Without SYN Is Risky](#12-why-tcp-without-syn-is-risky)
* [13. Troubleshooting Workflow](#13-troubleshooting-workflow)
* [14. NSE High-Value Notes](#14-nse-high-value-notes)
* [15. Quick Reference](#15-quick-reference)

---

# 1. FortiGate Maximum Value Tables

FortiGate has platform- and version-dependent **maximum value tables** that define limits for:

* Objects
* Policies
* Interfaces
* DHCP servers
* VDOMs
* Routes
* Sessions
* Other configuration resources

These limits are important when designing large FortiGate deployments.

Conceptually:

```text
FortiGate Platform
       │
       ▼
Maximum Value Tables
       │
       ├── Object Capacity
       ├── Policy Capacity
       ├── VDOM Capacity
       ├── DHCP Capacity
       └── Other Resources
```

> ⚠️ **Important:** Maximum values are **model, FortiOS-version, and configuration dependent**. Never assume a value from one FortiGate model applies universally.

---

# 2. DHCP Server Capacity

A useful example of a capacity limit is the number of DHCP server objects.

### Per-VDOM

```text
DHCP Server Limit per VDOM
        ↓
       256
```

For example:

```text
VDOM-A → up to 256 DHCP server objects
```

---

## VDOM Context

If a platform has:

```text
10 free VDOMs
```

and the relevant DHCP capacity is:

```text
256 DHCP servers / VDOM
```

the theoretical aggregate is:

```text
256 × 10 = 2560
```

So:

```text
10 VDOMs
   ×
256 DHCP server objects
   =
2560 objects
```

> ⚠️ This is a **capacity calculation**, not necessarily a guarantee that every FortiGate model supports 10 VDOMs or that all resources can simultaneously reach their individual maximums.

---

# 3. Checking Table Sizes

FortiGate CLI provides a way to inspect table sizes and maximum values.

```bash
print tablesize
```

This is useful when you need to investigate:

* Current capacity
* Maximum supported objects
* Platform limits
* Resource planning

### Capacity Planning Flow

```text
Requirement
    │
    ▼
Current Object Count
    │
    ▼
print tablesize
    │
    ▼
Maximum Capacity
    │
    ▼
Capacity Margin
```

---

# 4. Bidirectional vs Unidirectional Inspection

Not every protocol behaves identically from a firewall inspection perspective.

Some traffic requires the firewall to understand and track a **bidirectional exchange**, while other traffic can be handled primarily based on the initial session/request and its corresponding session state.

Conceptually:

```text
Client
  │
  │ Request
  ▼
FortiGate
  │
  ▼
Server
  │
  │ Reply
  ▼
FortiGate
  │
  ▼
Client
```

For stateful protocols, FortiGate tracks session information to determine whether subsequent packets belong to an established connection.

---

## Example: HTTP

A typical web session looks like:

```text
Client ─── HTTP Request ───> Server
Client <── HTTP Response ─── Server
```

FortiGate creates and tracks the session.

The firewall does not simply treat every packet as an independent connection.

---

# 5. FortiGate Reverse-Path Behavior

One important FortiGate behavior is related to reply traffic belonging to an already-allowed session.

FortiOS does **not necessarily perform a traditional reverse-path check on reply traffic solely because the reply arrives through a different interface**, provided the traffic matches the existing allowed session based on the session's IP tuple/state.

Conceptually:

```text
Request Path

Client
  │
  ▼
Interface A
  │
  ▼
FortiGate
  │
  ▼
Server


Reply Path

Server
  │
  ▼
Interface B
  │
  ▼
FortiGate
  │
  ▼
Client
```

This creates an **asymmetric routing** scenario.

---

## ⚠️ Key Concept

```text
Request Interface
       ≠
Reply Interface
```

does not automatically mean that FortiGate will reject the reply simply because the return path differs.

The existing session and packet/session matching behavior are critical.

---

# 6. Advanced Firewall Policy Features

Some advanced policy options are hidden until enabled through **Feature Visibility**.

Navigate to:

```text
System
  ↓
Feature Visibility
  ↓
Advanced Policy
```

Enable the relevant advanced policy features.

Conceptually:

```text
System
   │
   ▼
Feature Visibility
   │
   ▼
Advanced Policy
   │
   ▼
Additional Policy Options
```

These options can expose advanced matching and policy behaviors.

---

# 7. Negated Policy Objects

Advanced policy features can expose options such as **negated source/destination matching**.

Normal matching:

```text
Source = RFC1918
```

means:

```text
Match RFC1918
```

Negated matching conceptually means:

```text
NOT RFC1918
```

So:

```text
Source = NOT <Object>
```

means the policy matches traffic whose source does **not** belong to the selected object.

---

## Example Logic

Normal:

```text
srcaddr = TRUSTED_NET
```

Logic:

```text
Source ∈ TRUSTED_NET
```

Negated:

```text
NOT srcaddr = TRUSTED_NET
```

Logic:

```text
Source ∉ TRUSTED_NET
```

> 🧠 **NSE Tip:** Negation changes the matching logic; it does not mean that the object itself is removed from the configuration.

---

# 8. Captive Portal Exemption

Advanced policy options can also provide **Captive Portal exemption** functionality.

Conceptually:

```text
Client
  │
  ▼
Firewall Policy
  │
  ▼
Captive Portal Exemption
  │
  ▼
Bypass Captive Portal Processing
```

This is useful when a particular traffic flow should not be subjected to the captive portal mechanism.

---

## Security Consideration

Be careful when using captive portal exemptions.

Ask:

```text
Who is allowed to bypass?
What destination can they reach?
What authentication control is being bypassed?
Is the exemption narrowly scoped?
```

Avoid:

```text
Source = all
Destination = all
Captive Portal Exempt = enabled
```

unless there is a very specific architectural reason.

---

# 9. TCP Sessions Without SYN

This is one of the more advanced FortiGate session-handling features.

Normally, a TCP session begins with:

```text
Client                Server
  │                      │
  │──── SYN ────────────>│
  │<─── SYN/ACK ─────────│
  │──── ACK ────────────>│
  │                      │
  ▼
Established Session
```

FortiGate's stateful inspection normally expects TCP sessions to follow the expected state machine.

---

# 10. Asymmetric Routing Scenario

Consider a network with two interfaces and asymmetric routing.

```text
                 ┌──────────────┐
                 │   FortiGate  │
                 │              │
                 │ port1  port2 │
                 └───┬──────┬───┘
                     │      │
                     │      │
               Path A│      │Path B
                     │      │
                   Network
```

Traffic may enter through one interface and return through another.

For example:

```text
Client
   │
   │ Request
   ▼
port1
   │
   ▼
FortiGate
   │
   ▼
Server
   │
   │ Reply
   ▼
port2
   │
   ▼
FortiGate
```

Depending on the traffic flow, routing design and session state, this can create problems for stateful inspection.

---

# 11. Configuration

## Global Enable

```bash
config system settings
    set tcp-session-without-syn enable
end
```

This allows FortiGate to handle TCP sessions where the expected initial SYN is not observed in the normal path.

Conceptually:

```text
Normal:

SYN
 ↓
Session Creation
 ↓
Stateful Inspection
 ↓
Established


Without SYN:

Existing/observed TCP traffic
 ↓
More flexible session handling
```

---

## Enable on Firewall Policy

The feature must also be enabled on the relevant firewall policy.

```bash
config firewall policy
    edit 1
        set tcp-session-without-syn all
    next
end
```

Conceptually:

```text
Global Setting
      +
Policy Setting
      ↓
TCP Session Without SYN
      ↓
Allowed on selected policy
```

---

# 12. Why TCP Without SYN Is Risky

This feature relaxes normal TCP state expectations.

That means the firewall becomes more tolerant of packets that do not follow the normal TCP handshake path.

```text
Normal Stateful Inspection
        │
        ▼
Strict TCP State
        │
        ▼
SYN → SYN/ACK → ACK
```

With TCP without SYN:

```text
TCP Packet
    │
    ▼
Session Without Observed SYN
    │
    ▼
More Flexible Handling
```

### Security Impact

Potentially weaker stateful validation means:

* Unexpected TCP traffic may be accepted
* Session tracking becomes less strict
* Asymmetric designs become easier to accommodate
* Security visibility/control can be reduced

> 🚨 **Best practice:** Do **not** enable `tcp-session-without-syn` unless you have a clearly understood asymmetric-routing or session-path requirement.

---

# 13. Recommended Design

### ❌ Avoid

```text
Enable globally
+
Apply to all policies
+
Ignore routing asymmetry
```

### ✅ Prefer

```text
Fix routing first
       ↓
Achieve symmetric path
       ↓
Validate session behavior
       ↓
Only if required:
Enable TCP without SYN
       ↓
Apply narrowly
```

---

# 14. Troubleshooting Asymmetric Routing

When TCP sessions are being dropped unexpectedly:

### Step 1 — Check Routing

```text
Request Path
     ↓
Interface A

Reply Path
     ↓
Interface B
```

Determine whether the paths are asymmetric.

---

### Step 2 — Check Session

Inspect the FortiGate session table.

```bash
diagnose sys session list
```

Look for:

* Source
* Destination
* Source interface
* Destination interface
* NAT
* Session state
* Policy ID

---

### Step 3 — Check Policy

Confirm the expected firewall policy is matching:

```text
Source
Destination
Service
Schedule
Action
Policy ID
```

---

### Step 4 — Capture Traffic

Use packet-level troubleshooting when required:

```bash
diagnose sniffer packet <interface> ...
```

Compare:

```text
SYN
SYN/ACK
ACK
Data
```

and determine where the expected TCP handshake is missing.

---

### Step 5 — Consider `tcp-session-without-syn`

Only after confirming that the architecture genuinely requires asymmetric/session-without-SYN handling:

```bash
config system settings
    set tcp-session-without-syn enable
end
```

Then restrict it to the required policy:

```bash
config firewall policy
    edit <policy-id>
        set tcp-session-without-syn all
    next
end
```

---

# 15. Advanced Policy Decision Tree

```text
                 TCP Traffic Problem
                         │
                         ▼
                Is TCP handshake
                  visible normally?
                    │          │
                   YES         NO
                    │           │
                    ▼           ▼
              Check policy   Check routing
              /session           │
                    │            ▼
                    │      Asymmetric Path?
                    │        │          │
                    │       NO         YES
                    │        │          │
                    │        ▼          ▼
                    │   Investigate   Fix routing
                    │   other cause       │
                    │                     ▼
                    │               Still required?
                    │                 │        │
                    │                NO       YES
                    │                 │        │
                    │                 ▼        ▼
                    │                Done   Consider
                    │                       TCP without SYN
                    │                              │
                    │                              ▼
                    │                       Apply narrowly
```

---

# 16. Capacity vs Feature Configuration

A useful FortiGate design principle:

```text
Platform Capacity
       +
FortiOS Feature Availability
       +
Current Resource Usage
       +
Configuration
       ↓
Actual Operational Capacity
```

Never design based only on the theoretical maximum.

For example:

```text
Theoretical DHCP Capacity
        ↓
256 / VDOM
        ↓
But actual deployment also depends on:
        ├── VDOM capacity
        ├── Platform model
        ├── FortiOS version
        ├── Other object limits
        └── Overall resource utilization
```

---

# 17. NSE High-Value Notes 🧠

## Maximum Values

```bash
print tablesize
```

Think:

> **What can this FortiGate support?**

---

## Advanced Policy Features

```text
System
  ↓
Feature Visibility
  ↓
Advanced Policy
```

Think:

> **Hidden advanced policy options**

---

## Negated Matching

```text
Normal:
MATCH Object

Negated:
MATCH NOT Object
```

---

## Captive Portal Exemption

```text
Policy
   ↓
Captive Portal Exemption
   ↓
Bypass captive portal processing
```

Use narrowly.

---

## Reverse Path

```text
Request:
Interface A

Reply:
Interface B
```

Do not automatically assume:

```text
Different interface
      =
Immediate drop
```

Session matching and FortiOS forwarding behavior matter.

---

## TCP Without SYN

Global:

```bash
config system settings
    set tcp-session-without-syn enable
end
```

Policy:

```bash
config firewall policy
    edit <id>
        set tcp-session-without-syn all
    next
end
```

Think:

> **Asymmetric routing / missing SYN / relaxed TCP state handling**

---

# 18. Security Comparison

| Feature                    | Normal Behavior           | Advanced/Relaxed Behavior             |
| -------------------------- | ------------------------- | ------------------------------------- |
| TCP state tracking         | Normal handshake expected | Can accommodate missing SYN           |
| Routing                    | Prefer symmetric          | Can tolerate certain asymmetric paths |
| Stateful inspection        | Stronger state validation | More permissive                       |
| Security posture           | Preferred                 | Use only when required                |
| Troubleshooting complexity | Lower                     | Higher                                |
| Recommended default        | ✅                         | ❌                                     |

---

# 19. Quick Reference

| Requirement                      | Command / Location                   |
| -------------------------------- | ------------------------------------ |
| Show capacity tables             | `print tablesize`                    |
| Advanced policy features         | `System → Feature Visibility`        |
| Negated source matching          | Advanced Policy                      |
| Negated destination matching     | Advanced Policy                      |
| Captive Portal exemption         | Advanced Policy                      |
| Enable TCP without SYN globally  | `set tcp-session-without-syn enable` |
| Enable TCP without SYN in policy | `set tcp-session-without-syn all`    |
| Inspect sessions                 | `diagnose sys session list`          |
| Packet capture                   | `diagnose sniffer packet ...`        |

---

# 🎯 Final Mental Model

```text
                       FORTIGATE
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
      Capacity        Policy Features    Sessions
          │                │                │
          ▼                ▼                ▼
   print tablesize     Advanced Policy   TCP State
                           │                │
                    ┌──────┴──────┐         │
                    ▼             ▼         ▼
                 Negation     Captive    Without SYN
                              Portal          │
                                             ▼
                                      Asymmetric Paths
```

### 🔥 One-Line Memory Hook

```text
print tablesize
    → Capacity

Advanced Policy
    → Negation / Captive Portal options

Existing Session
    → Reply path behavior matters

tcp-session-without-syn
    → Relaxed TCP state handling
    → Useful for specific asymmetric scenarios
    → NOT a default best practice
```

> **Production rule:** **Fix the network path first.** If asymmetric routing is the real problem, correcting routing is almost always preferable to globally relaxing TCP stateful inspection.

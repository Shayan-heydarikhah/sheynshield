# FortiGate IPS — Advanced Global Settings, CVE Filtering, Botnet C&C, IEC 61850 & SCTP  

> **FortiGate IPS  ** — Advanced IPS engine tuning, signature submission, anomaly handling, deep application inspection, CVE filtering, Botnet C&C detection, industrial protocol inspection, and SCTP filtering.

---

## 📌 Table of Contents

* [IPS Global Configuration](#-ips-global-configuration)
* [Traffic Submission to FortiGuard](#-traffic-submission-to-fortiguard)
* [Anomaly Mode](#-anomaly-mode)
* [Session TTL Synchronization](#-session-ttl-synchronization)
* [Deep Application Inspection Cache](#-deep-application-inspection-cache)
* [IPS Packet Log Queue](#-ips-packet-log-queue)
* [NGFW Deep Scan Range](#-ngfw-deep-scan-range)
* [CVE-Based IPS Filtering](#-cve-based-ips-filtering)
* [IPS Sensor Attributes](#-ips-sensor-attributes)
* [Botnet C&C Detection](#-botnet-cc-detection)
* [IEC 61850 / MMS / ICCP Inspection](#-iec-61850--mms--iccp-inspection)
* [SCTP Filtering](#-sctp-filtering)
* [SCTP PPID](#-sctp-ppid)
* [SCTP Filter Actions](#-sctp-filter-actions)
* [SCTP Linux Testing](#-sctp-linux-testing)
* [Applying SCTP Filtering to a Firewall Policy](#-applying-sctp-filtering-to-a-firewall-policy)
* [IPS Troubleshooting Checklist](#-ips-troubleshooting-checklist)
* [Quick NSE Exam Notes](#-quick-nse-exam-notes)

---

# 🔥 IPS Global Configuration

FortiGate exposes several global IPS parameters for controlling:

* IPS engine behavior
* signature submission
* anomaly detection
* application inspection caching
* packet-log buffering
* deep inspection limits
* session synchronization
* IPS memory/performance behavior

Basic configuration:

```bash
config ips global
    set traffic-submit enable
    set anomaly-mode continuous
    set sync-session-ttl enable
    set deep-app-insp-timeout 86400
    set deep-app-insp-db-limit 100000
    set packet-log-queue-depth 128
    set ngfw-max-scan-range 4096
end
```

> ⚠️ **Important:** Available options and exact behavior can vary by FortiOS release and FortiGate platform. Always validate the command against the target FortiOS version with `?` or CLI help.

---

# 🚀 Traffic Submission to FortiGuard

```bash
config ips global
    set traffic-submit enable
end
```

### What does it do?

FortiGate can submit a sample of relevant traffic/content to **FortiGuard** to help with threat analysis and signature generation.

Conceptually:

```text
Client
   │
   ▼
FortiGate IPS Engine
   │
   │ suspicious traffic/sample
   ▼
FortiGuard
   │
   │ analysis
   ▼
New / Updated Detection Logic
```

### Key idea

```text
Traffic
   ↓
IPS inspection
   ↓
Threat/sample identification
   ↓
FortiGuard analysis
   ↓
Signature / detection improvement
```

> 🔑 **NSE Note:** Traffic submission is related to FortiGuard threat research and detection improvement; it should not be confused with normal IPS signature matching.

---

# 🔄 Anomaly Mode

```bash
config ips global
    set anomaly-mode continuous
end
```

Depending on the FortiOS implementation/version, anomaly processing can operate in modes such as:

| Mode         | Concept                            |
| ------------ | ---------------------------------- |
| `continuous` | Continuous anomaly tracking        |
| `periodical` | Periodic evaluation/reset behavior |

### Practical distinction

```text
Continuous
    ↓
Keep tracking activity continuously

Periodical
    ↓
Evaluate within defined periods
    ↓
Counters/assessment can reset
```

> 🔑 Choose the mode based on the behavior you want to detect. High-frequency authentication abuse and burst-style attacks should be evaluated differently from periodic traffic patterns.

---

# ⏱️ Session TTL Synchronization

```bash
config ips global
    set sync-session-ttl enable
end
```

FortiGate firewall policies and sessions have timeout values.

Enabling session TTL synchronization allows IPS-related session handling to remain synchronized with the firewall session lifecycle.

### Why it matters

```text
Firewall Session
       │
       ├── TTL
       │
       ▼
IPS Session Tracking
       │
       └── synchronized lifecycle
```

### Recommended when

* IPS is heavily involved in session tracking
* Long-lived sessions exist
* Session timeout consistency matters
* Troubleshooting session-state behavior

---

# 🧠 Deep Application Inspection Cache

## Timeout

```bash
config ips global
    set deep-app-insp-timeout 86400
end
```

`86400` seconds:

```text
86400 sec = 24 hours
```

This controls the timeout associated with deep application inspection information/cache.

---

## Database Limit

```bash
config ips global
    set deep-app-insp-db-limit 100000
end
```

Conceptually:

```text
Maximum cached deep-application inspection entries
        ↓
100000
```

### Performance consideration

Increasing inspection/cache limits may improve reuse of inspection information, but it can also increase resource consumption.

> ⚠️ Do not blindly maximize this value. Tune it according to the FortiGate model, traffic volume, session count, and available memory.

---

# 📦 IPS Packet Log Queue

```bash
config ips global
    set packet-log-queue-depth 128
end
```

Controls the queue depth used for IPS packet-log processing.

Conceptually:

```text
IPS Events
   │
   ▼
Packet Log Queue
   │
   ├── event 1
   ├── event 2
   ├── event 3
   └── ...
```

### Important

A larger queue is **not automatically better**.

Increasing queue depth can help absorb bursts, but excessive buffering can consume additional memory.

---

# 🔬 NGFW Deep Scan Range

```bash
config ips global
    set ngfw-max-scan-range 4096
end
```

The value represents the maximum inspection range used for the relevant deep inspection processing.

Example concept:

```text
Large Content
──────────────────────────────────────────────
0                  4096                  ...
│<──── inspect ────>│
```

For example, with:

```text
ngfw-max-scan-range = 4096
```

the relevant inspection range is limited to approximately:

```text
4096 bytes
```

> ⚠️ Do **not** interpret this as "a 10 GB file is always split into independent 4096-byte files." The actual IPS inspection architecture involves protocol decoding, buffers, streams, and engine-specific processing.

---

# 🆔 CVE-Based IPS Filtering

CVE identifiers allow administrators to associate IPS filtering with known vulnerabilities.

Example:

```bash
config ips sensor
    edit ips-cus-test
        config entries
            edit 1
                set cve 2020
            next
        end
    next
end
```

> ⚠️ The exact syntax/value expected by `set cve` depends on the FortiOS version and CLI schema. Use the version-specific CLI help before deploying.

### CVE workflow

```text
CVE
 │
 ▼
Vulnerability identification
 │
 ▼
FortiGuard / IPS database
 │
 ▼
Matching IPS signatures
 │
 ▼
IPS Sensor
 │
 ▼
Firewall Policy
```

### Useful reference

**NVD — National Vulnerability Database**

```text
nvd.nist.gov
```

---

# 🎯 IPS Sensor Attributes

IPS sensors can filter signatures based on signature attributes instead of manually selecting every individual signature.

Common attributes include:

* Default status
* Default action
* Vulnerability type
* Last modified/update information
* CVE
* Severity
* Protocol/service
* Signature ID

Example:

```bash
config ips sensor
    edit ips-cus-test
        config entries
            edit 1
                set vuln-type 12
                set last-modified ...
            next
        end
    next
end
```

---

## Why Attribute Filtering Matters

Imagine thousands of signatures exist in the IPS database.

Instead of manually selecting:

```text
Signature 1
Signature 2
Signature 3
...
Signature 5000
```

you can filter based on characteristics:

```text
Vulnerability Type
        +
Severity
        +
Protocol
        +
Last Modified
        ↓
Relevant Signatures
```

### Benefit

This can reduce unnecessary processing and avoid irrelevant signature logging.

---

# 🔎 Checking IPS Rule Status

Example diagnostic concept:

```bash
get ips rule status | grep eicar.virus.test.file-a
```

Useful for verifying whether a specific signature/rule exists and what state it is in.

> 🔑 **Troubleshooting tip:** When a signature appears to be configured but does not trigger, verify signature status, policy attachment, inspection mode, protocol/service detection, and traffic visibility before assuming the signature itself is broken.

---

# 🤖 Botnet C&C Detection

FortiGate IPS can use FortiGuard-provided intelligence to identify traffic associated with known **Botnet Command & Control (C&C)** infrastructure.

Conceptually:

```text
Client
  │
  │ outbound connection
  ▼
FortiGate
  │
  ▼
Botnet C&C Intelligence
  │
  ├── Known malicious IP
  ├── Known C&C destination
  └── Threat intelligence
```

### When blocked

FortiGate can use a FortiGuard-provided database containing known malicious infrastructure indicators.

This intelligence can also interact with other security functions depending on the FortiOS feature and configuration.

---

# 🦠 Botnet C&C Detection — Practical Flow

```text
User Device
     │
     │ HTTPS / HTTP / DNS / other traffic
     ▼
FortiGate
     │
     ▼
Threat Intelligence Lookup
     │
     ├── Known C&C → BLOCK
     │
     └── Unknown → Continue inspection
```

> 🔑 **NSE Note:** Botnet C&C detection is primarily an intelligence-driven detection mechanism; it is different from a conventional static IPS exploit signature.

---

# 🏭 IEC 61850 / MMS / ICCP Inspection

FortiGate IPS supports protocol decoding for certain industrial/control-system protocols.

### IEC 61850

**IEC 61850** is widely used in electrical substation automation.

One important communication component is:

```text
MMS
```

### MMS

**Manufacturing Message Specification (MMS)** is used for communication between systems/devices in industrial environments.

FortiGate IPS can use protocol dissectors to identify individual MMS services and messages.

---

# 🔗 ICCP / TASE.2

Another relevant protocol is:

```text
ICCP / TASE.2
```

ICCP is used for communication between control centers and is based on MMS-related transport mechanisms.

Conceptually:

```text
IEC 61850
   │
   └── MMS
        │
        └── Protocol inspection

ICCP / TASE.2
   │
   └── MMS transport
```

---

# 🧩 MMS TCP Segmentation Handling

IPS protocol decoding can handle scenarios such as:

### Multiple MMS PDUs in one TCP payload

```text
TCP Payload
┌────────┬────────┬────────┐
│ MMS #1 │ MMS #2 │ MMS #3 │
└────────┴────────┴────────┘
```

### One MMS message split across TCP segments

```text
TCP Segment 1
┌───────────────┐
│ MMS Message   │
└───────┬───────┘
        │
TCP Segment 2
┌───────▼───────┐
│ continuation  │
└───────────────┘
```

The protocol decoder reconstructs/identifies the relevant protocol structures before applying IPS inspection.

---

# 📡 SCTP Filtering

**SCTP — Stream Control Transmission Protocol**

SCTP provides transport functionality different from TCP/UDP and is commonly encountered in telecom/core-network environments.

FortiGate provides SCTP inspection/filtering capabilities through the SCTP dissector and PPID-based filtering.

Example port commonly associated with telecom signaling:

```text
36412
```

> 🔑 Port numbers are contextual. SCTP inspection should rely on protocol identification and configured filtering rather than assuming every SCTP service uses one fixed port.

---

# 🆔 SCTP PPID

**PPID = Payload Protocol Identifier**

PPID identifies the protocol/application associated with an SCTP data chunk.

Conceptually:

```text
SCTP Packet
│
├── Common Header
│
└── DATA Chunk
     │
     ├── Stream ID
     ├── Stream Sequence
     ├── PPID
     └── Payload
```

FortiGate can use PPID filtering to identify specific SCTP payload types.

---

# 🛡️ SCTP Filter Configuration

Example:

```bash
config sctp-filter profile
    edit sctp-test
        config ppid-filters
            edit 1
                set ppid 112233
                set action reset
            next
        end
    next
end
```

> ⚠️ `112233` is an example PPID value. Use the PPID appropriate to the application/protocol you are actually inspecting.

---

# ⚙️ SCTP Filter Actions

Common conceptual actions include:

| Action    | Meaning                                                        |
| --------- | -------------------------------------------------------------- |
| `pass`    | Allow the SCTP data                                            |
| `reset`   | Terminate/reset the SCTP session                               |
| `replace` | Replace offending data while preserving stream synchronization |

### Why `replace` can matter

Dropping an SCTP data chunk can interfere with the expected sequence/state between endpoints.

Replacing the offending payload with zeros can allow the session structure to remain synchronized while removing the unwanted content.

Conceptually:

```text
Original:

[ Header ][ MALICIOUS DATA ][ Header ]

             ↓ SCTP Filter

Replaced:

[ Header ][ 000000000000 ][ Header ]
```

---

# 📡 SCTP Protocol Examples

SCTP is used by various telecom and signaling protocols, including environments involving:

* SS7-related signaling adaptation
* M3UA
* S1AP
* Diameter-related deployments
* Other telecom signaling stacks

> ⚠️ Do not identify an application solely by its SCTP port. Use protocol/PPID and application context where possible.

---

# 🧪 SCTP Testing on Linux

Install SCTP tooling.

### RHEL/CentOS/Rocky-style systems

```bash
sudo yum install lksctp-tools
```

### Debian/Ubuntu-style systems

```bash
sudo apt install lksctp-tools
```

> Package availability/name can vary slightly by distribution.

---

## SCTP Server Test

Example:

```bash
sctp_test -H 0.0.0.0 -P 5001 -l
```

Concept:

```text
Local IP:   0.0.0.0
Local Port: 5001
Mode:       Listen
```

---

## SCTP Client Test

Example:

```bash
sctp_test \
    -H 192.168.101.5 \
    -P 5002 \
    -h 192.168.20.201 \
    -p 5001 \
    -s
```

### Parameters

```text
-H → local host
-P → local port
-h → remote host
-p → remote port
-s → client/send mode
```

Architecture:

```text
SCTP Client
192.168.101.5:5002
       │
       │ SCTP
       ▼
FortiGate
       │
       ▼
SCTP Server
192.168.20.201:5001
```

---

# 🔥 Applying SCTP Filtering to a Firewall Policy

Example:

```bash
config firewall policy
    edit 1
        set srcintf "lan"
        set dstintf "dmz"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
        set utm-status enable
        set ssl-ssh-profile "custom-deep-inspection"
        set sctp-filter-profile "sctp-test"
        set logtraffic all
    next
end
```

### Traffic flow

```text
LAN
 │
 │ SCTP
 ▼
FortiGate
 │
 ├── SSL/SSH Inspection (where applicable)
 │
 ├── SCTP Dissector
 │
 ├── PPID Filter
 │
 └── Action
      ├── Pass
      ├── Reset
      └── Replace
 │
 ▼
DMZ
```

---

# 🧠 IPS Troubleshooting Checklist

When an IPS signature/filter does not behave as expected:

### 1. Verify the firewall policy

```text
Correct source interface?
Correct destination interface?
Correct policy?
Policy enabled?
```

### 2. Verify inspection mode

```text
Flow / Proxy
Deep Inspection
IPS enabled
```

### 3. Verify IPS sensor

```text
Correct IPS sensor?
Signature enabled?
Correct action?
Correct severity?
Correct protocol/service?
```

### 4. Verify protocol decoding

```text
Is FortiGate identifying the protocol correctly?
```

### 5. Verify signature attributes

```text
CVE
Vulnerability type
Severity
Status
Last modified
Service
Flow
```

### 6. Verify logging

```text
Packet logging
Attack context
IPS event logs
```

### 7. Verify hardware/resource conditions

Check:

```text
CPU
Memory
IPS engine load
Conserve mode
IPS fail-open
Packet-log queue
Socket/buffer configuration
```

---

# ⚡ Quick NSE Exam Notes

| Topic                    | Remember                                                              |
| ------------------------ | --------------------------------------------------------------------- |
| `traffic-submit`         | Submit relevant traffic/sample data for FortiGuard analysis           |
| `anomaly-mode`           | Controls anomaly processing behavior                                  |
| `sync-session-ttl`       | Synchronizes IPS/session TTL handling                                 |
| `deep-app-insp-timeout`  | Deep application inspection cache timeout                             |
| `deep-app-insp-db-limit` | Limit for deep application inspection database/cache                  |
| `packet-log-queue-depth` | IPS packet-log queue depth                                            |
| `ngfw-max-scan-range`    | Limits relevant deep inspection scan range                            |
| CVE                      | Maps/filters IPS protection around known vulnerabilities              |
| Botnet C&C               | Uses threat intelligence to detect known malicious C&C infrastructure |
| IEC 61850                | Industrial/substation automation standard                             |
| MMS                      | Protocol used in IEC 61850 environments                               |
| ICCP/TASE.2              | Control-center communication using MMS-related transport              |
| SCTP                     | Stream Control Transmission Protocol                                  |
| PPID                     | Identifies payload protocol in SCTP DATA chunks                       |
| `pass`                   | Forward/allow                                                         |
| `reset`                  | Terminate/reset SCTP session                                          |
| `replace`                | Replace offending data while preserving session/sequence behavior     |

---

# 🎯 High-Value Mental Model

The advanced FortiGate IPS architecture can be remembered as:

```text
                    ┌──────────────────┐
                    │   FortiGuard     │
                    │ Threat Intel /   │
                    │ Signature Update │
                    └────────┬─────────┘
                             │
                             ▼
Client ──► Firewall Policy ──► IPS Engine
                             │
                ┌────────────┼────────────┐
                │            │            │
                ▼            ▼            ▼
           Protocol      Signature     Anomaly
           Decoder       Matching      Detection
                │            │            │
                └────────────┼────────────┘
                             │
                             ▼
                      Threat Decision
                             │
                ┌────────────┼────────────┐
                ▼            ▼            ▼
              Allow         Log        Block/Reset
```

For specialized traffic:

```text
                    IPS Engine
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
        HTTP          SCTP          MMS
          │             │             │
       Decoder        PPID          IEC 61850
          │           Filter          │
          ▼             ▼             ▼
       Signature      Action        Dissector
```

---

# 🧪 Practical Verification Workflow

```text
1. Identify the traffic
        ↓
2. Confirm firewall policy
        ↓
3. Confirm inspection mode
        ↓
4. Confirm IPS sensor
        ↓
5. Confirm protocol decoder
        ↓
6. Confirm signature/filter attributes
        ↓
7. Generate test traffic
        ↓
8. Check IPS logs
        ↓
9. Check packet/attack context
        ↓
10. Check CPU/memory/IPS engine state
```

---

## 🔑 Final Takeaways

* **IPS is not just signature matching** — protocol decoders, anomaly detection, threat intelligence, and application-aware inspection all contribute to detection.
* **CVE filtering** is useful for narrowing protection around known vulnerabilities.
* **Botnet C&C detection** depends heavily on threat intelligence.
* **Industrial protocols** such as MMS/IEC 61850 require protocol-aware inspection rather than simple port matching.
* **SCTP PPID filtering** provides application-aware control inside SCTP traffic.
* Increasing IPS buffers, caches, or scan limits can affect **CPU and memory consumption**.
* Always validate global IPS tuning against the **FortiGate model, FortiOS release, traffic volume, and available resources**.
* For troubleshooting, think in this order:

```text
Policy
  ↓
Inspection
  ↓
Decoder
  ↓
IPS Sensor
  ↓
Signature / Filter
  ↓
Engine
  ↓
Action
  ↓
Log
```

> **FortiGate IPS mindset:**
> **Decode → Identify → Match → Correlate → Decide → Log/Block**

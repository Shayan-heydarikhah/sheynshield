# 🔗 SheynShield Resources

# FortiGate IPS — Advanced Global Settings, CVE Filtering, Botnet C&C, IEC 61850 & SCTP

> **FortiOS IPS | Advanced IPS Engine Tuning | CVE Filtering | Botnet C&C | IEC 61850/MMS/ICCP | SCTP/PPID**
>
> **Level:** NSE 4 → NSE 7
> **Brand:** SheynShield — Engineering Secure Networks

---

## 📌 Table of Contents

* [1. IPS Global Configuration](#1-ips-global-configuration)
* [2. Traffic Submission to FortiGuard](#2-traffic-submission-to-fortiguard)
* [3. Anomaly Mode](#3-anomaly-mode)
* [4. Session TTL Synchronization](#4-session-ttl-synchronization)
* [5. Deep Application Inspection Cache](#5-deep-application-inspection-cache)
* [6. IPS Packet Log Queue](#6-ips-packet-log-queue)
* [7. NGFW Deep Scan Range](#7-ngfw-deep-scan-range)
* [8. CVE-Based IPS Filtering](#8-cve-based-ips-filtering)
* [9. IPS Sensor Attributes](#9-ips-sensor-attributes)
* [10. IPS Rule Status Verification](#10-ips-rule-status-verification)
* [11. Botnet C&C Detection](#11-botnet-cc-detection)
* [12. IEC 61850 / MMS / ICCP Inspection](#12-iec-61850--mms--iccp-inspection)
* [13. MMS TCP Segmentation](#13-mms-tcp-segmentation)
* [14. SCTP Filtering](#14-sctp-filtering)
* [15. SCTP PPID](#15-sctp-ppid)
* [16. SCTP Filter Actions](#16-sctp-filter-actions)
* [17. SCTP Linux Testing](#17-sctp-linux-testing)
* [18. Applying SCTP Filtering](#18-applying-sctp-filtering)
* [19. IPS Troubleshooting Checklist](#19-ips-troubleshooting-checklist)
* [20. Performance & Resource Checklist](#20-performance--resource-checklist)
* [21. NSE Exam Checklist](#21-nse-exam-checklist)
* [22. Practical Verification Workflow](#22-practical-verification-workflow)
* [23. Advanced IPS Mental Model](#23-advanced-ips-mental-model)
* [24. Final IPS Checklist](#24-final-ips-checklist)

---

# 1. IPS Global Configuration

## Global Configuration Checklist

* [ ] Understand that `config ips global` controls global IPS engine behavior.
* [ ] Identify IPS signature submission settings.
* [ ] Understand anomaly processing behavior.
* [ ] Understand session TTL synchronization.
* [ ] Understand deep application inspection cache settings.
* [ ] Understand IPS packet-log queue behavior.
* [ ] Understand NGFW deep scan range.
* [ ] Validate commands against the target FortiOS release.
* [ ] Check platform-specific limitations before changing global IPS values.

Example:

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

> ⚠️ **Version Check:** Available options and exact behavior can vary by FortiOS release and FortiGate platform.

---

# 2. Traffic Submission to FortiGuard

## Configuration

```bash
config ips global
    set traffic-submit enable
end
```

### Checklist

* [ ] Understand what `traffic-submit` controls.
* [ ] Understand that relevant traffic/sample data may be submitted for FortiGuard analysis.
* [ ] Understand the relationship between IPS detection and FortiGuard threat research.
* [ ] Do not confuse traffic submission with normal IPS signature matching.
* [ ] Review organizational privacy/security requirements before enabling traffic submission.
* [ ] Validate the feature behavior on the target FortiOS release.

### Mental Model

```text
Traffic
   ↓
IPS Inspection
   ↓
Threat / Sample Identification
   ↓
FortiGuard Analysis
   ↓
Signature / Detection Improvement
```

### NSE Memory

* [ ] `traffic-submit` → FortiGuard-related traffic/sample submission.
* [ ] Do not interpret it as the mechanism that performs ordinary local signature matching.

---

# 3. Anomaly Mode

## Configuration

```bash
config ips global
    set anomaly-mode continuous
end
```

### Checklist

* [ ] Understand `continuous` anomaly processing.
* [ ] Understand `periodical` anomaly processing where supported.
* [ ] Understand the difference between continuous and periodic evaluation.
* [ ] Consider authentication abuse and burst traffic when designing anomaly detection.
* [ ] Validate available modes with CLI help on the target FortiOS version.

### Concept

```text
Continuous
    ↓
Continuous activity tracking
```

```text
Periodical
    ↓
Periodic evaluation
    ↓
Counters / assessment may reset
```

### NSE Memory

* [ ] `continuous` → continuous anomaly tracking.
* [ ] `periodical` → periodic evaluation/reset behavior.

---

# 4. Session TTL Synchronization

## Configuration

```bash
config ips global
    set sync-session-ttl enable
end
```

### Checklist

* [ ] Understand firewall session TTL.
* [ ] Understand IPS session tracking.
* [ ] Understand the purpose of synchronizing IPS-related session lifecycle with firewall sessions.
* [ ] Consider this setting when troubleshooting long-lived sessions.
* [ ] Consider session timeout consistency during IPS troubleshooting.
* [ ] Validate behavior on the target FortiOS release.

### Mental Model

```text
Firewall Session
       │
       ├── TTL
       │
       ▼
IPS Session Tracking
       │
       └── Synchronized Lifecycle
```

---

# 5. Deep Application Inspection Cache

## Timeout

```bash
config ips global
    set deep-app-insp-timeout 86400
end
```

### Checklist

* [ ] Understand that `86400` seconds equals 24 hours.
* [ ] Understand the purpose of the deep application inspection timeout.
* [ ] Consider session volume when tuning the timeout.
* [ ] Avoid increasing the value without considering memory usage.

```text
86400 seconds
      ↓
24 hours
```

---

## Database Limit

```bash
config ips global
    set deep-app-insp-db-limit 100000
end
```

### Checklist

* [ ] Understand that this controls the relevant deep application inspection database/cache limit.
* [ ] Consider traffic volume.
* [ ] Consider concurrent sessions.
* [ ] Consider available memory.
* [ ] Avoid blindly maximizing the value.
* [ ] Monitor resource utilization after changes.

### Mental Model

```text
Deep Application Inspection
          ↓
Inspection Information
          ↓
Cache / Database
          ↓
Configured Entry Limit
```

---

# 6. IPS Packet Log Queue

## Configuration

```bash
config ips global
    set packet-log-queue-depth 128
end
```

### Checklist

* [ ] Understand that `packet-log-queue-depth` controls IPS packet-log buffering.
* [ ] Understand that larger queues can absorb bursts.
* [ ] Consider memory consumption.
* [ ] Avoid assuming that a larger queue is always better.
* [ ] Monitor logging behavior during high-volume IPS events.

### Mental Model

```text
IPS Events
    ↓
Packet Log Queue
    ├── Event 1
    ├── Event 2
    ├── Event 3
    └── ...
```

---

# 7. NGFW Deep Scan Range

## Configuration

```bash
config ips global
    set ngfw-max-scan-range 4096
end
```

### Checklist

* [ ] Understand the purpose of the NGFW deep scan range.
* [ ] Understand that the value limits the relevant inspection range.
* [ ] Consider large payloads and deep inspection requirements.
* [ ] Consider CPU and memory implications.
* [ ] Validate the exact behavior on the target FortiOS version.
* [ ] Do not interpret the value as simply splitting a large file into independent chunks.

Example concept:

```text
Large Content
────────────────────────────────────────────
0                  4096                  ...
│<──── Relevant Inspection Range ────>│
```

---

# 8. CVE-Based IPS Filtering

## CVE Concept

* [ ] Understand what a CVE represents.
* [ ] Understand that CVE-based filtering can help target protection around known vulnerabilities.
* [ ] Understand the relationship between CVE → IPS signatures → IPS sensor.
* [ ] Verify the exact `set cve` syntax for the target FortiOS version.
* [ ] Avoid copying CLI syntax between FortiOS releases without validation.

Example concept:

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

> ⚠️ **Important:** The exact syntax/value expected by `set cve` depends on the FortiOS CLI schema.

### CVE Workflow

```text
CVE
 │
 ▼
Vulnerability Identification
 │
 ▼
FortiGuard / IPS Database
 │
 ▼
Matching IPS Signatures
 │
 ▼
IPS Sensor
 │
 ▼
Firewall Policy
```

### Checklist

* [ ] Identify the target CVE.
* [ ] Verify the vulnerability information.
* [ ] Check available IPS coverage.
* [ ] Filter/select relevant signatures.
* [ ] Add the appropriate signatures to the IPS sensor.
* [ ] Attach the IPS sensor to the correct firewall policy.
* [ ] Generate controlled test traffic.
* [ ] Verify IPS logs.

---

# 9. IPS Sensor Attributes

## Attribute Filtering Checklist

* [ ] Understand default status.
* [ ] Understand default action.
* [ ] Understand vulnerability type.
* [ ] Understand last-modified/update information.
* [ ] Understand CVE filtering.
* [ ] Understand severity.
* [ ] Understand protocol/service filtering.
* [ ] Understand signature ID filtering.
* [ ] Use attributes instead of manually selecting thousands of signatures when appropriate.

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

### Why Attribute Filtering Matters

```text
Thousands of IPS Signatures
          ↓
Attribute Filtering
          ├── CVE
          ├── Severity
          ├── Vulnerability Type
          ├── Protocol
          └── Last Modified
          ↓
Relevant Signatures
```

### Checklist

* [ ] Reduce unnecessary signature selection.
* [ ] Filter by vulnerability characteristics where appropriate.
* [ ] Review the resulting signature set.
* [ ] Verify actions and severity.
* [ ] Test before production deployment.

---

# 10. IPS Rule Status Verification

## Diagnostic Checklist

Use the appropriate diagnostic command for the target FortiOS release.

Example:

```bash
get ips rule status | grep eicar.virus.test.file-a
```

### Checklist

* [ ] Verify that the signature exists.
* [ ] Verify that the signature is enabled.
* [ ] Verify the signature status.
* [ ] Verify the configured action.
* [ ] Verify that the IPS sensor contains the signature.
* [ ] Verify that the firewall policy uses the correct IPS sensor.
* [ ] Verify protocol/service identification.
* [ ] Verify traffic visibility.

### Troubleshooting Logic

```text
Signature Configured?
       ↓
Signature Active?
       ↓
IPS Sensor?
       ↓
Firewall Policy?
       ↓
Protocol Detection?
       ↓
Traffic Visible?
       ↓
Signature Trigger?
```

---

# 11. Botnet C&C Detection

## Botnet C&C Checklist

* [ ] Understand Command & Control (C&C).
* [ ] Understand that Botnet C&C detection can use FortiGuard threat intelligence.
* [ ] Understand known malicious IP/destination intelligence.
* [ ] Understand that this is different from a conventional static exploit signature.
* [ ] Verify the relevant FortiGuard services/licensing/configuration.
* [ ] Review resulting security events in logs.

### Detection Flow

```text
User Device
     │
     │ HTTP / HTTPS / DNS / Other Traffic
     ▼
FortiGate
     │
     ▼
Threat Intelligence
     │
     ├── Known C&C → BLOCK / ACTION
     │
     └── Unknown → Continue Inspection
```

### NSE Memory

* [ ] Botnet C&C detection → intelligence-driven detection.
* [ ] Static IPS signature → pattern/protocol-based detection.
* [ ] Do not treat both mechanisms as identical.

---

# 12. IEC 61850 / MMS / ICCP Inspection

## IEC 61850 Checklist

* [ ] Understand that IEC 61850 is widely used in electrical substation automation.
* [ ] Understand the role of MMS in IEC 61850 environments.
* [ ] Understand that protocol-aware inspection is preferable to simple port-based matching.
* [ ] Verify whether the FortiGate model/FortiOS release supports the required industrial protocol inspection.

### MMS

**MMS = Manufacturing Message Specification**

* [ ] Understand MMS as an application-layer protocol used in industrial environments.
* [ ] Understand that IPS can use protocol dissectors to identify MMS structures/services.
* [ ] Understand the difference between raw payload inspection and protocol-aware inspection.

---

# 13. MMS TCP Segmentation

## Multiple MMS PDUs in One TCP Payload

* [ ] Understand that multiple MMS PDUs can exist within a single TCP payload.
* [ ] Verify that the protocol decoder can identify individual structures.

```text
TCP Payload
┌────────┬────────┬────────┐
│ MMS #1 │ MMS #2 │ MMS #3 │
└────────┴────────┴────────┘
```

---

## MMS Message Split Across TCP Segments

* [ ] Understand TCP segmentation.
* [ ] Understand protocol reassembly/decoding.
* [ ] Understand that one logical MMS message can span multiple TCP segments.

```text
TCP Segment 1
┌────────────────┐
│ MMS Message    │
└────────┬───────┘
         │
TCP Segment 2
┌────────▼───────┐
│ Continuation   │
└────────────────┘
```

### NSE Memory

* [ ] Protocol decoder → identifies/reconstructs relevant protocol structures.
* [ ] IPS should not be viewed as simple independent-packet string matching.

---

# 14. SCTP Filtering

**SCTP = Stream Control Transmission Protocol**

### SCTP Checklist

* [ ] Understand SCTP as a transport protocol.
* [ ] Understand that SCTP differs from TCP and UDP.
* [ ] Identify telecom/core-network use cases.
* [ ] Understand SCTP protocol inspection.
* [ ] Understand PPID-based filtering.
* [ ] Do not rely exclusively on port numbers.
* [ ] Verify FortiOS support for SCTP inspection/filtering.

Example contextual port:

```text
36412
```

> ⚠️ Port numbers are contextual. Protocol identification and application context are more reliable than assuming one fixed port.

---

# 15. SCTP PPID

**PPID = Payload Protocol Identifier**

### PPID Checklist

* [ ] Understand that PPID identifies the protocol/application associated with an SCTP DATA chunk.
* [ ] Understand the SCTP DATA chunk structure.
* [ ] Understand how PPID can be used for application-aware filtering.
* [ ] Identify the correct PPID for the target application.

### SCTP Structure

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

### Mental Model

```text
SCTP
 ↓
DATA Chunk
 ↓
PPID
 ↓
Application / Protocol Identification
 ↓
Filtering Decision
```

---

# 16. SCTP Filter Actions

## Example Configuration

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

> ⚠️ `112233` is an example PPID. Replace it with the correct PPID for the application being inspected.

---

## Action Checklist

| Action    | Checklist | Meaning                                                        |
| --------- | --------: | -------------------------------------------------------------- |
| `pass`    |       [ ] | Allow/forward SCTP data                                        |
| `reset`   |       [ ] | Terminate/reset the SCTP session                               |
| `replace` |       [ ] | Replace offending data while preserving stream synchronization |

---

## Replace Behavior

* [ ] Understand why blindly dropping an SCTP DATA chunk may affect expected stream state.
* [ ] Understand the purpose of `replace`.
* [ ] Understand that replacing offending payload data can preserve session/stream synchronization.

Concept:

```text
Original:

[ Header ][ MALICIOUS DATA ][ Header ]

                 ↓

             SCTP Filter

                 ↓

Replaced:

[ Header ][ 000000000000 ][ Header ]
```

---

# 17. SCTP Linux Testing

## Install SCTP Tools

### RHEL / CentOS / Rocky

```bash
sudo yum install lksctp-tools
```

* [ ] Install SCTP userspace tools.
* [ ] Verify the package is available for the distribution.
* [ ] Verify SCTP kernel support.

### Debian / Ubuntu

```bash
sudo apt install lksctp-tools
```

* [ ] Install SCTP tooling.
* [ ] Verify the package is available.
* [ ] Verify SCTP kernel support.

---

## SCTP Server

```bash
sctp_test -H 0.0.0.0 -P 5001 -l
```

### Checklist

* [ ] Confirm local IP.
* [ ] Confirm listening port.
* [ ] Confirm server/listen mode.
* [ ] Verify SCTP connectivity from the client.

```text
Local IP    → 0.0.0.0
Local Port  → 5001
Mode        → Listen
```

---

## SCTP Client

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
-H → Local Host
-P → Local Port
-h → Remote Host
-p → Remote Port
-s → Client / Send Mode
```

### Test Architecture

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

### Testing Checklist

* [ ] SCTP client is reachable.
* [ ] SCTP server is listening.
* [ ] Firewall policy permits the traffic.
* [ ] SCTP filter profile is attached.
* [ ] PPID is correct.
* [ ] Filter action is correct.
* [ ] IPS/security logs are enabled.
* [ ] Expected behavior is observed.

---

# 18. Applying SCTP Filtering

## Firewall Policy Example

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

### Deployment Checklist

* [ ] Correct source interface.
* [ ] Correct destination interface.
* [ ] Correct source addresses.
* [ ] Correct destination addresses.
* [ ] Correct firewall policy.
* [ ] Policy is enabled.
* [ ] `utm-status` is enabled where required.
* [ ] Correct SCTP filter profile is selected.
* [ ] Appropriate inspection profile is configured where applicable.
* [ ] Logging is enabled.
* [ ] Test traffic is generated.
* [ ] SCTP filter events are verified.

### Traffic Flow

```text
LAN
 │
 │ SCTP
 ▼
FortiGate
 │
 ├── Inspection
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

# 19. IPS Troubleshooting Checklist

## A. Firewall Policy

* [ ] Correct source interface?
* [ ] Correct destination interface?
* [ ] Correct source address?
* [ ] Correct destination address?
* [ ] Correct policy?
* [ ] Policy enabled?
* [ ] Correct service?
* [ ] Correct security profile?

---

## B. Inspection Mode

* [ ] Correct inspection mode?
* [ ] Flow-based inspection?
* [ ] Proxy-based inspection?
* [ ] Deep inspection required?
* [ ] SSL inspection required?
* [ ] Encrypted traffic visibility available?

---

## C. IPS Sensor

* [ ] Correct IPS sensor?
* [ ] Signature enabled?
* [ ] Correct action?
* [ ] Correct severity?
* [ ] Correct protocol?
* [ ] Correct service?
* [ ] Correct signature attributes?
* [ ] Correct CVE filter?

---

## D. Protocol Decoder

* [ ] Is FortiGate identifying the protocol correctly?
* [ ] Is the application classified correctly?
* [ ] Is the correct service tree being used?
* [ ] Is protocol reassembly required?
* [ ] Is the protocol supported on the target FortiOS version?

---

## E. Signature / Filter

* [ ] Signature exists?
* [ ] Signature enabled?
* [ ] Pattern correct?
* [ ] Context correct?
* [ ] Flow correct?
* [ ] Range correct?
* [ ] CVE mapping correct?
* [ ] Vulnerability type correct?
* [ ] Severity correct?
* [ ] Attack ID/Vulnerability ID correct where applicable?

---

## F. Logging

* [ ] IPS event logging enabled?
* [ ] Packet logging enabled where required?
* [ ] Attack context available?
* [ ] Correct log source identified?
* [ ] Timestamp correlated with test traffic?
* [ ] Action recorded correctly?

---

## G. Resources

* [ ] CPU utilization checked?
* [ ] Memory utilization checked?
* [ ] IPS engine load checked?
* [ ] Conserve mode checked?
* [ ] IPS fail-open behavior checked?
* [ ] Packet-log queue checked?
* [ ] Socket/buffer conditions checked?

---

# 20. Performance & Resource Checklist

## Global IPS Tuning

* [ ] Avoid changing global IPS values without a defined requirement.
* [ ] Check FortiGate model limitations.
* [ ] Check FortiOS release.
* [ ] Check traffic volume.
* [ ] Check concurrent sessions.
* [ ] Check available memory.
* [ ] Check CPU utilization.
* [ ] Measure before/after changes.
* [ ] Monitor production impact.

---

## Cache / Database

* [ ] Do not blindly maximize `deep-app-insp-db-limit`.
* [ ] Evaluate memory consumption.
* [ ] Evaluate session volume.
* [ ] Evaluate application diversity.

---

## Packet Log Queue

* [ ] Use larger queue depths only when justified.
* [ ] Consider burst logging.
* [ ] Consider memory impact.
* [ ] Monitor packet-log processing.

---

## Scan Range

* [ ] Understand what traffic requires deep scanning.
* [ ] Avoid unnecessary inspection ranges.
* [ ] Consider payload size.
* [ ] Consider CPU/memory impact.

---

# 21. NSE Exam Checklist

## Global IPS

* [ ] `traffic-submit` → FortiGuard traffic/sample submission.
* [ ] `anomaly-mode` → anomaly processing behavior.
* [ ] `sync-session-ttl` → IPS/session TTL synchronization.
* [ ] `deep-app-insp-timeout` → deep application inspection timeout.
* [ ] `deep-app-insp-db-limit` → deep application inspection database/cache limit.
* [ ] `packet-log-queue-depth` → IPS packet-log queue depth.
* [ ] `ngfw-max-scan-range` → relevant deep inspection scan range.

---

## Threat Intelligence

* [ ] Botnet C&C → threat intelligence driven.
* [ ] Known malicious infrastructure → intelligence lookup.
* [ ] C&C detection ≠ ordinary static exploit signature.

---

## CVE

* [ ] CVE identifies a known vulnerability.
* [ ] IPS signatures can provide protection for vulnerabilities.
* [ ] IPS sensor attributes can be used to narrow signature selection.
* [ ] Always verify version-specific CLI syntax.

---

## Industrial Protocols

* [ ] IEC 61850 → electrical/substation automation.
* [ ] MMS → Manufacturing Message Specification.
* [ ] ICCP/TASE.2 → control-center communication.
* [ ] Protocol-aware inspection is preferred over simple port assumptions.
* [ ] TCP segmentation/reassembly matters for protocol inspection.

---

## SCTP

* [ ] SCTP → Stream Control Transmission Protocol.
* [ ] PPID → Payload Protocol Identifier.
* [ ] PPID exists within SCTP DATA chunk information.
* [ ] `pass` → allow/forward.
* [ ] `reset` → terminate/reset session.
* [ ] `replace` → replace offending payload while preserving stream behavior.
* [ ] Do not identify applications only by SCTP port.
* [ ] PPID can provide application-aware filtering.

---

# 22. Practical Verification Workflow

Use this workflow whenever an advanced IPS feature does not behave as expected:

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
7. Generate controlled test traffic
        ↓
8. Check IPS logs
        ↓
9. Check packet / attack context
        ↓
10. Check CPU / memory / IPS engine state
```

## Verification Checklist

* [ ] Traffic identified.
* [ ] Source/destination confirmed.
* [ ] Firewall policy confirmed.
* [ ] Inspection mode confirmed.
* [ ] IPS sensor confirmed.
* [ ] Signature/filter confirmed.
* [ ] Protocol decoder confirmed.
* [ ] Test traffic generated.
* [ ] IPS event generated.
* [ ] Action verified.
* [ ] Logs verified.
* [ ] Resource utilization reviewed.

---

# 23. Advanced IPS Mental Model

## Core IPS Architecture

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

### Architecture Checklist

* [ ] Understand firewall policy as the traffic entry point.
* [ ] Understand protocol decoding.
* [ ] Understand signature matching.
* [ ] Understand anomaly detection.
* [ ] Understand FortiGuard intelligence.
* [ ] Understand final threat decision.
* [ ] Understand logging.
* [ ] Understand blocking/reset behavior.

---

## Specialized Protocol Inspection

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

### Checklist

* [ ] HTTP → application-aware inspection.
* [ ] SCTP → PPID-aware filtering.
* [ ] MMS → industrial protocol decoding.
* [ ] IEC 61850 → industrial/substation context.
* [ ] Avoid relying exclusively on ports.

---

# 24. Final IPS Checklist

## Global Settings

* [ ] `traffic-submit`
* [ ] `anomaly-mode`
* [ ] `sync-session-ttl`
* [ ] `deep-app-insp-timeout`
* [ ] `deep-app-insp-db-limit`
* [ ] `packet-log-queue-depth`
* [ ] `ngfw-max-scan-range`

## Signature / Sensor

* [ ] CVE filtering
* [ ] Vulnerability type
* [ ] Severity
* [ ] Last modified
* [ ] Protocol
* [ ] Service
* [ ] Signature status
* [ ] Signature action

## Threat Intelligence

* [ ] Botnet C&C
* [ ] FortiGuard intelligence
* [ ] Known malicious infrastructure
* [ ] Threat intelligence logging

## Industrial Protocols

* [ ] IEC 61850
* [ ] MMS
* [ ] ICCP/TASE.2
* [ ] TCP segmentation
* [ ] Protocol decoding

## SCTP

* [ ] SCTP inspection
* [ ] PPID identification
* [ ] PPID filtering
* [ ] `pass`
* [ ] `reset`
* [ ] `replace`
* [ ] Linux SCTP testing
* [ ] Firewall policy integration

## Troubleshooting

* [ ] Firewall policy
* [ ] Inspection mode
* [ ] IPS sensor
* [ ] Protocol decoder
* [ ] Signature/filter
* [ ] Logging
* [ ] CPU
* [ ] Memory
* [ ] IPS engine state

---

# 🔥 Golden Rules

> **1. Always validate IPS CLI syntax against the exact FortiOS release.**

> **2. Do not assume that a global IPS setting has the same behavior across every FortiGate platform.**

> **3. Traffic submission to FortiGuard is different from normal IPS signature matching.**

> **4. CVE filtering helps narrow protection around known vulnerabilities.**

> **5. Botnet C&C detection is heavily dependent on threat intelligence.**

> **6. Industrial protocols require protocol-aware inspection rather than simple port assumptions.**

> **7. MMS inspection must account for TCP segmentation and protocol structure.**

> **8. SCTP PPID provides application-aware filtering inside SCTP traffic.**

> **9. `pass`, `reset`, and `replace` have different implications for SCTP session behavior.**

> **10. Increasing cache, queue, or scan limits can increase resource consumption.**

> **11. Always test advanced IPS configuration before applying it globally.**

> **12. Troubleshoot IPS from policy → inspection → decoder → sensor → signature/filter → engine → action → log.**

---

# 🧠 Advanced IPS Memory Map

```text
FORTIGATE IPS
│
├── Global Engine
│   ├── traffic-submit
│   ├── anomaly-mode
│   ├── sync-session-ttl
│   ├── deep-app-insp-timeout
│   ├── deep-app-insp-db-limit
│   ├── packet-log-queue-depth
│   └── ngfw-max-scan-range
│
├── Signature / Sensor
│   ├── CVE
│   ├── Vulnerability Type
│   ├── Severity
│   ├── Protocol
│   ├── Service
│   └── Signature Status
│
├── Threat Intelligence
│   └── Botnet C&C
│
├── Industrial
│   ├── IEC 61850
│   ├── MMS
│   └── ICCP / TASE.2
│
├── SCTP
│   ├── SCTP Decoder
│   ├── PPID
│   ├── pass
│   ├── reset
│   └── replace
│
└── Troubleshooting
    ├── Policy
    ├── Inspection
    ├── Decoder
    ├── Sensor
    ├── Signature / Filter
    ├── Engine
    ├── Action
    └── Logs
```

---

# 🎯 30-Second NSE Interview Answer

**How would you troubleshoot and tune advanced FortiGate IPS?**

> First verify the firewall policy and inspection mode, then confirm that FortiGate correctly identifies the protocol and that the appropriate IPS sensor and signatures are active. For advanced protection, review CVE-based filtering, signature attributes, anomaly detection and FortiGuard threat intelligence such as Botnet C&C detection. For specialized environments, validate protocol-aware inspection such as MMS/IEC 61850 and SCTP PPID filtering. Finally, verify logs and attack context while monitoring CPU, memory and IPS engine resources. Any global tuning should be validated against the exact FortiOS release and FortiGate platform.

---

# 📌 Final Mental Model

```text
                    FORTIGATE IPS
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
   Global Engine     FortiGuard       Protocol
        │            Intelligence      Decoder
        │                │                │
        └────────────────┼────────────────┘
                         ▼
                  IPS Sensor / Rules
                         │
             ┌───────────┼───────────┐
             ▼           ▼           ▼
           CVE        Signatures   Anomaly
             │           │           │
             └───────────┼───────────┘
                         ▼
                  Threat Decision
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
            Allow       Log       Block
```

> **FortiGate IPS Mindset:**
> **Decode → Identify → Filter → Inspect → Correlate → Decide → Log / Block**

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

## 🔖 Tags

```text
fortigate
fortios
fortinet
ips
intrusion-prevention-system
custom-ips
ips-sensor
cve
botnet
botnet-c2
fortiguard
iec-61850
mms
iccp
tase2
sctp
ppid
network-security
cybersecurity
nse4
nse7
fortinet-nse
security-engineering
sheynshield
```

---

## 🏷️  Keywords

```text
FortiGate IPS configuration
FortiGate IPS advanced settings
FortiOS IPS global settings
FortiGate CVE IPS filtering
FortiGate Botnet C&C detection
FortiGate IEC 61850 inspection
FortiGate MMS inspection
FortiGate ICCP inspection
FortiGate SCTP filtering
FortiGate SCTP PPID
FortiGate IPS troubleshooting
FortiGate IPS sensor
FortiOS IPS NSE7
FortiGate industrial protocol security
Fortinet IPS cheat sheet
FortiGate IPS checklist
```

---

**SheynShield — Engineering Secure Networks**
*Practical Network Security • Fortinet • IPS • Secure Network Design*

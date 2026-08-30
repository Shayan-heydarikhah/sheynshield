# 🛡️ FortiGate Application Control — Security Engineering Checklist

> **FortiOS 7.x | Application Identification | IPS Protocol Decoders | Application Signatures | Port Enforcement | QUIC/HTTP3 | SSL Inspection | Quarantine | SCADA/ICS**
>
> **SheynShield Security Engineering Checklist** for NSE preparation, enterprise deployment, troubleshooting, and real-world application visibility.

---

## 📌 Table of Contents

* [1. Application Control Fundamentals](#1-application-control-fundamentals)
* [2. Application Detection](#2-application-detection)
* [3. Application Control Profile](#3-application-control-profile)
* [4. Application Actions](#4-application-actions)
* [5. Application Attributes](#5-application-attributes)
* [6. Other vs Unknown](#6-other-vs-unknown)
* [7. Category vs Application Matching](#7-category-vs-application-matching)
* [8. Application Exclusions](#8-application-exclusions)
* [9. Port Enforcement](#9-port-enforcement)
* [10. Network Protocol Enforcement](#10-network-protocol-enforcement)
* [11. QUIC / HTTP3](#11-quic--http3)
* [12. Application Quarantine](#12-application-quarantine)
* [13. Deep Application Inspection](#13-deep-application-inspection)
* [14. SSL-Based Application Detection](#14-ssl-based-application-detection)
* [15. Sandwich SSL Topology](#15-sandwich-ssl-topology)
* [16. SCADA / ICS Protocol Decoders](#16-scada--ics-protocol-decoders)
* [17. GTP Decoder](#17-gtp-decoder)
* [18. Modbus Decoder](#18-modbus-decoder)
* [19. DNP3 Decoder](#19-dnp3-decoder)
* [20. Multiple Application Parameters](#20-multiple-application-parameters)
* [21. CLI Configuration Checklist](#21-cli-configuration-checklist)
* [22. Troubleshooting Checklist](#22-troubleshooting-checklist)
* [23. Production Deployment Checklist](#23-production-deployment-checklist)
* [24. Performance Checklist](#24-performance-checklist)
* [25. NSE Exam Traps](#25-nse-exam-traps)
* [26. Interview Quick Check](#26-interview-quick-check)
* [27. 60-Second Memory Map](#27-60-second-memory-map)
* [28. Most Important CLI](#28-most-important-cli)

---

# 1. Application Control Fundamentals

## Architecture

```text
                    Client
                       |
                       v
                Firewall Policy
                       |
                       v
              Application Control
                       |
             +---------+---------+
             |                   |
             v                   v
      IPS Protocol Decoder   Application DB
             |                   |
             +---------+---------+
                       |
                       v
                Application ID
                       |
                       v
             Matching / Filtering
                       |
          +------------+------------+
          |            |            |
        PASS         BLOCK      QUARANTINE
```

### Fundamentals Checklist

* [ ] Understand that Application Control identifies applications in network traffic.
* [ ] Understand that Application Control is not simply port-based filtering.
* [ ] Understand the relationship between Application Control and IPS protocol decoders.
* [ ] Understand the role of application signatures.
* [ ] Understand the Application Control database/signature updates.
* [ ] Understand that application visibility depends on the traffic being sufficiently inspectable.
* [ ] Understand that encrypted traffic may require SSL inspection for deeper application visibility.
* [ ] Understand the difference between application identification and application enforcement.

### Core Rule

> **Port Number ≠ Application Identity**

Examples:

```text
TCP/443 ≠ Always HTTPS
TCP/80  ≠ Always HTTP
UDP/443 ≠ Automatically "just QUIC"
```

---

# 2. Application Detection

## Detection Mechanisms

Verify that you understand each detection mechanism:

* [ ] IPS protocol decoder
* [ ] Application signature
* [ ] Application database
* [ ] Protocol characteristics
* [ ] Application metadata
* [ ] Application behavior
* [ ] Application technology
* [ ] Vendor classification
* [ ] Popularity
* [ ] Risk classification

### Detection Flow

```text
Traffic
   |
   v
Protocol Analysis
   |
   +--> IPS Protocol Decoder
   |
   +--> Application Signature
   |
   +--> Application Database
   |
   v
Application Identification
```

### NSE Check

* [ ] Can you explain why an application may be detected on a non-standard port?
* [ ] Can you explain why port `443` alone does not prove HTTPS?
* [ ] Can you distinguish **application detection** from **port enforcement**?

---

# 3. Application Control Profile

Navigate to:

```text
Security Profiles
└── Application Control
```

### Profile Design Checklist

* [ ] Create an Application Control profile.
* [ ] Define default application behavior.
* [ ] Define known application rules.
* [ ] Define category-based rules.
* [ ] Define high-risk application handling.
* [ ] Define unknown application handling.
* [ ] Define other application handling.
* [ ] Define application logging requirements.
* [ ] Define quarantine behavior where required.
* [ ] Review default application port enforcement.
* [ ] Review deep application inspection requirements.
* [ ] Review SSL-related application signatures.
* [ ] Attach the Application Control profile to the intended firewall policy.

### Recommended Starting Model

```text
Default
   |
   +--> Allow / Monitor

High Risk
   |
   +--> Block

Restricted Categories
   |
   +--> Block

Unknown
   |
   +--> Monitor / Block

Other
   |
   +--> Pass / Monitor
```

---

# 4. Application Actions

Verify the intended action for every application rule.

| Action       | Checklist Meaning               |
| ------------ | ------------------------------- |
| `pass`       | [ ] Allow application           |
| `monitor`    | [ ] Allow + log application     |
| `block`      | [ ] Block application           |
| `reset`      | [ ] Reset connection            |
| `quarantine` | [ ] Quarantine violating client |

### Action Checklist

* [ ] Use `pass` where traffic should be explicitly permitted.
* [ ] Use `monitor` during visibility/learning phases.
* [ ] Use `block` for applications that must not be allowed.
* [ ] Use `reset` where immediate session termination is desired.
* [ ] Use `quarantine` when the client itself should be placed into a quarantine state.
* [ ] Verify logging for security-sensitive actions.
* [ ] Test the actual behavior before production enforcement.

### Remember

```text
BLOCK
  ↓
Matching traffic is blocked

QUARANTINE
  ↓
Violating client enters quarantine state
```

---

# 5. Application Attributes

Application Control can classify/match applications using multiple attributes.

## Protocol

* [ ] Understand protocol-based matching.
* [ ] Verify protocol requirements for the application.

```text
protocol
```

---

## Risk

Verify understanding of application risk levels:

| Risk | Level    |
| ---: | -------- |
|  `1` | Low      |
|  `2` | Elevated |
|  `3` | Medium   |
|  `4` | High     |
|  `5` | Critical |

### Checklist

* [ ] Review risk level before creating broad blocking rules.
* [ ] Treat high/critical-risk applications as candidates for stricter controls.
* [ ] Validate business requirements before blocking high-risk applications.

---

## Vendor

* [ ] Understand vendor-based matching.
* [ ] Verify vendor information for applications where required.

```text
vendor
```

---

## Technology

Review technology classification:

| Value | Technology       |
| ----: | ---------------- |
|   `0` | Network Protocol |
|   `1` | Browser-Based    |
|   `2` | Client-Server    |
|   `4` | Peer-to-Peer     |

* [ ] Understand technology-based application filtering.
* [ ] Review P2P applications separately where required.

---

## Behavior

Review behavior classifications:

| Value | Behavior            |
| ----: | ------------------- |
|   `2` | Botnet              |
|   `3` | Evasive             |
|   `5` | Excessive Bandwidth |
|   `6` | Tunneling           |
|   `9` | Cloud               |

### Checklist

* [ ] Review evasive applications.
* [ ] Review tunneling applications.
* [ ] Review botnet-related applications.
* [ ] Review excessive-bandwidth applications.
* [ ] Consider behavior when building security policies.

---

## Popularity

```text
1 → Least Popular
5 → Most Popular
```

* [ ] Understand application popularity classification.
* [ ] Do not confuse popularity with security risk.
* [ ] Avoid assuming that a popular application is automatically safe.

---

# 6. Other vs Unknown

> ⭐ **High-value NSE concept**

## Other Application

* [ ] Understand that the application is known/signature-based.
* [ ] Understand that it does not match the specific configured application rule.
* [ ] Understand `other-application-action`.

```text
Known Signature
      |
      v
Does not match configured rule
      |
      v
OTHER
```

## Unknown Application

* [ ] Understand that FortiGate cannot identify the application using available signatures.
* [ ] Understand `unknown-application-action`.

```text
No Matching Application Identification
                |
                v
             UNKNOWN
```

### Exam Memory

```text
Known but not matched
        ↓
      OTHER

Not identified
        ↓
     UNKNOWN
```

---

# 7. Category vs Application Matching

## Category-Based Rules

* [ ] Use category matching when multiple applications should receive the same action.
* [ ] Review the categories before enforcement.
* [ ] Validate category membership against current FortiGuard/application data.

Example:

```cli
config application list
    edit "app-cont-test"
        config entries
            edit 1
                set category 2 6
                set action block
            next
        end
    next
end
```

## Application-Based Rules

* [ ] Use application IDs when specific applications need explicit control.
* [ ] Verify application IDs before deployment.
* [ ] Document application IDs used in production.

Example:

```cli
config application list
    edit "app-cont-test"
        config entries
            edit 1
                set application 15893 40568
                set action block
            next
        end
    next
end
```

### Comparison

```text
CATEGORY
   ↓
Multiple applications

APPLICATION ID
   ↓
Specific applications
```

---

# 8. Application Exclusions

Use exclusions when a broad application/category rule requires exceptions.

### Checklist

* [ ] Define the broad category/application rule.
* [ ] Identify legitimate exceptions.
* [ ] Add explicit application exclusions.
* [ ] Test excluded applications.
* [ ] Verify excluded applications are not unintentionally blocked.
* [ ] Document business justification for every exclusion.

Example:

```cli
config application list
    edit "app-cont-test"

        set other-application-action pass
        set unknown-application-action pass

        config entries
            edit 1
                set action block
                set category 2 3 5 6 7 8 12 15 17 21 22 23 25 26 28 29 30 31
                set exclusion 15893 40568
            next
        end

    next
end
```

### Mental Model

```text
Broad Category Rule
        |
        +---- Application A → BLOCK
        |
        +---- Application B → EXCLUDED
        |
        +---- Application C → BLOCK
```

---

# 9. Port Enforcement

> ⭐ **Critical NSE distinction**

### Application Detection

```text
"What application is this?"
```

### Port Enforcement

```text
"Is this application using its expected/default port?"
```

Enable:

```cli
config application list
    edit "app-cont-test"
        set enforce-default-app-port enable
    next
end
```

### Checklist

* [ ] Understand application detection independently of port numbers.
* [ ] Enable default application port enforcement where required.
* [ ] Test standard application ports.
* [ ] Test non-standard application ports.
* [ ] Identify legitimate applications using non-standard ports.
* [ ] Document required exceptions.
* [ ] Verify enforcement behavior before production rollout.

### Memory

```text
Without Port Enforcement:

Application
 ├── TCP/443
 ├── TCP/8443
 └── TCP/8080

With Port Enforcement:

Application
 ├── Expected Port → ALLOW
 └── Non-default Port → VIOLATION
```

> **Application Detection ≠ Port Enforcement**

---

# 10. Network Protocol Enforcement

Verify expected network services and ports.

Example:

```cli
config application list
    edit "app-cont-test"

        set enforce-default-app-port enable

        config default-network-services
            edit 2
                set port 53
                set services dns
                set violation-action monitor
            next
        end

    next
end
```

### Checklist

* [ ] Identify expected network services.
* [ ] Identify expected ports.
* [ ] Configure default network services where required.
* [ ] Define violation action.
* [ ] Test legitimate traffic.
* [ ] Test traffic using unexpected ports.
* [ ] Review monitoring logs.
* [ ] Convert monitoring to enforcement only after validation.

### Example

```text
DNS
 |
 +── UDP/53 → Expected
 |
 +── Other Port → Violation
```

---

# 11. QUIC / HTTP3

> ⭐ **Modern web traffic exam + deployment checkpoint**

### Fundamentals

* [ ] Understand that QUIC uses UDP.
* [ ] Understand that HTTP/3 commonly uses UDP/443.
* [ ] Do not assume TCP/443 controls all HTTPS-like traffic.
* [ ] Review UDP/443 behavior in enterprise environments.
* [ ] Define QUIC policy requirements.
* [ ] Test Application Control with QUIC traffic.
* [ ] Test Web Filter behavior with QUIC.
* [ ] Evaluate SSL inspection compatibility.
* [ ] Evaluate HTTP/3 application visibility.

### Traffic Model

```text
HTTP/3
   ↓
QUIC
   ↓
UDP
   ↓
Commonly UDP/443
```

### Enterprise Checklist

```text
Firewall Policy
      |
      +── SSL Inspection
      |
      +── Web Filter
      |
      +── Application Control
      |
      +── QUIC Handling
```

### NSE Trap

```text
TCP/443
   ≠
All HTTPS/Web Traffic
```

---

# 12. Application Quarantine

Quarantine changes the scope of enforcement from simply blocking a matching connection to placing the violating client into a quarantine state.

### Checklist

* [ ] Determine whether quarantine is required.
* [ ] Configure quarantine action.
* [ ] Define quarantine duration.
* [ ] Enable quarantine logging.
* [ ] Test client quarantine behavior.
* [ ] Verify quarantine expiration.
* [ ] Verify monitoring through FortiView.
* [ ] Verify Quarantine Monitor behavior.
* [ ] Document the operational impact of quarantine.

Example:

```cli
set action block
set quarantine attacker
set quarantine-expiry 5
set quarantine-log enable
```

### Concept

```text
Application Violation
        |
        v
Client Identified
        |
        v
Client Quarantined
        |
        v
Traffic Blocked
        |
        v
Timer Expires
```

---

# 13. Deep Application Inspection

Some applications require deeper inspection for reliable identification.

Example:

```cli
config application list
    edit "app-cont-test"
        set deep-app-inspection enable
    next
end
```

### Checklist

* [ ] Identify applications requiring deeper inspection.
* [ ] Enable deep application inspection where justified.
* [ ] Test application identification.
* [ ] Compare identification before/after deep inspection.
* [ ] Monitor CPU impact.
* [ ] Monitor memory impact.
* [ ] Monitor latency.
* [ ] Pay special attention to high-volume traffic.
* [ ] Evaluate impact on QUIC/HTTP3 traffic.

### Concept

```text
Initial Traffic
      |
      v
Basic Identification
      |
      v
Additional Application Data
      |
      v
Deep Application Inspection
      |
      v
Improved Identification
```

> **More inspection depth = potentially more CPU, memory, and latency.**

---

# 14. SSL-Based Application Detection

Encrypted application traffic can limit application visibility.

Relevant configuration:

```cli
set force-inclusion-ssl-di-sigs enable
```

Example:

```cli
config application list
    edit "app-cont-test"
        set force-inclusion-ssl-di-sigs enable
    next
end
```

### Checklist

* [ ] Determine whether traffic is encrypted.
* [ ] Determine whether SSL inspection is available.
* [ ] Deploy appropriate CA certificates to clients where required.
* [ ] Enable Deep Inspection where organizationally appropriate.
* [ ] Verify decrypted application visibility.
* [ ] Review SSL-DI-related application signatures.
* [ ] Understand `require_ssl_di`.
* [ ] Understand `force-inclusion-ssl-di-sigs`.
* [ ] Test application identification after decryption.

### Processing Model

```text
Encrypted Traffic
       |
       v
SSL Inspection
       |
       v
Decrypted Payload
       |
       v
Application Detection
       |
       v
Security Decision
```

### Key Rule

> **No decryption → limited payload visibility → potentially limited application identification.**

---

# 15. Sandwich SSL Topology

A sandwich SSL topology may place FortiGate between decryption and encryption devices.

```text
              SSL Decryption
                    |
                    v
            +---------------+
            |   FortiGate   |
            | Application   |
            |    Control    |
            +---------------+
                    |
                    v
              SSL Encryption
```

### Checklist

* [ ] Confirm the upstream device decrypts traffic.
* [ ] Confirm FortiGate receives clear traffic.
* [ ] Confirm FortiGate can inspect the decrypted application traffic.
* [ ] Enable applicable SSL-DI signatures.
* [ ] Validate application identification.
* [ ] Confirm traffic is re-encrypted after inspection.
* [ ] Test application-control actions.
* [ ] Validate latency and throughput.

Configuration example:

```cli
config application list
    edit "app-cont-test"
        set force-inclusion-ssl-di-sigs enable
    next
end
```

### Flow

```text
Encrypted
   ↓
Decryption Device
   ↓
Clear Traffic
   ↓
FortiGate
   ↓
Application Control
   ↓
Security Decision
   ↓
Encryption Device
   ↓
Destination
```

---

# 16. SCADA / ICS Protocol Decoders

> ⚠️ **High-value security engineering area**

Application Control and IPS protocol decoders can inspect application-specific industrial protocols beyond simple IP/port matching.

### Protocol Checklist

* [ ] Identify industrial protocols in the environment.
* [ ] Identify protocol ports.
* [ ] Confirm decoder availability for the FortiOS release.
* [ ] Review protocol signatures.
* [ ] Review protocol anomalies.
* [ ] Review exploit signatures.
* [ ] Test before enforcement.
* [ ] Establish maintenance/change windows for testing.

### SCADA/ICS Model

```text
SCADA / ICS Traffic
        |
        v
IPS Protocol Decoder
        |
        +── Protocol Validation
        |
        +── Function Analysis
        |
        +── Anomaly Detection
        |
        +── Exploit Detection
        |
        v
Security Decision
```

Supported examples:

```text
GTP
Modbus
DNP3
```

---

# 17. GTP Decoder

## GTP Fundamentals

* [ ] Know that GTP = GPRS Tunneling Protocol.
* [ ] Understand GTP control plane.
* [ ] Understand GTP user plane.
* [ ] Identify GTP-C.
* [ ] Identify GTP-U.
* [ ] Know common ports.

```text
UDP/2123 → GTP-C
UDP/2152 → GTP-U
```

### GTP Security Checklist

* [ ] Validate GTP messages.
* [ ] Review malformed headers.
* [ ] Review tunnel flooding protection.
* [ ] Review fraud detection.
* [ ] Review IMSI-related controls.
* [ ] Review IMEI-related controls.
* [ ] Review protocol anomalies.
* [ ] Review exploit detection.

### IMSI

```text
IMSI
 |
 +── MCC
 |
 +── MNC
 |
 └── MSIN
```

* [ ] Remember IMSI = subscriber identity.

### IMEI

* [ ] Remember IMEI = mobile equipment/device identity.

---

# 18. Modbus Decoder

## Modbus Fundamentals

* [ ] Understand Modbus usage in PLC/SCADA environments.
* [ ] Understand Modbus TCP.
* [ ] Understand Modbus RTU.
* [ ] Know Modbus TCP default port.

```text
TCP/502
```

### Modbus Security Checklist

* [ ] Inspect protocol validity.
* [ ] Review illegal function codes.
* [ ] Review malformed packets.
* [ ] Review request flooding.
* [ ] Review unauthorized write commands.
* [ ] Review protocol anomalies.
* [ ] Validate IPS signatures.
* [ ] Test legitimate PLC traffic before enforcement.

### Security Model

```text
PLC
 |
 | Modbus Write
 v
FortiGate
 |
 +── Function Code Validation
 |
 +── Protocol Validation
 |
 +── IPS Signature
 |
 v
ALLOW / BLOCK
```

> ⚠️ Blocking legitimate industrial write operations can affect physical processes. Always test carefully.

---

# 19. DNP3 Decoder

## DNP3 Fundamentals

* [ ] Know that DNP3 = Distributed Network Protocol 3.
* [ ] Identify common use cases:

  * [ ] Electrical utilities
  * [ ] Water treatment
  * [ ] SCADA
  * [ ] Oil and gas
  * [ ] Industrial control systems
* [ ] Know common TCP port.

```text
TCP/20000
```

### DNP3 Security Checklist

* [ ] Review malformed packets.
* [ ] Review protocol violations.
* [ ] Review function-code behavior.
* [ ] Review read/write operations.
* [ ] Review authentication-related behavior.
* [ ] Review exploit/fuzzing detection.
* [ ] Validate signatures before enforcement.

### Security Model

```text
DNP3 Traffic
      |
      v
DNP3 Decoder
      |
      +── Packet Validation
      |
      +── Function Analysis
      |
      +── Anomaly Detection
      |
      +── Exploit Detection
      |
      v
Security Action
```

---

# 20. Multiple Application Parameters

Some application signatures support multiple parameters.

This can be especially useful for:

* [ ] SCADA
* [ ] ICS
* [ ] Industrial protocols
* [ ] Function-code inspection
* [ ] Protocol-specific inspection

### Concept

```text
Application Signature
       |
       +── Parameter Group 1
       |
       +── Parameter Group 2
       |
       +── Parameter Group 3
```

### Checklist

* [ ] Identify the application signature.
* [ ] Identify available parameters.
* [ ] Understand how parameters affect matching.
* [ ] Validate function-code matching.
* [ ] Test multiple parameter combinations.
* [ ] Verify the resulting action.
* [ ] Review false positives.

### Example

```text
Modbus
   +
Function Code
   +
Protocol Parameter
   ↓
Signature Match
   ↓
Security Action
```

---

# 21. CLI Configuration Checklist

## Basic Application Control

```cli
config application list
    edit "app-cont-test"

        set extended-log enable

        set other-application-action pass
        set other-application-log enable

        set unknown-application-action block
        set unknown-application-log enable

        set enforce-default-app-port enable

        set force-inclusion-ssl-di-sigs disable

        set deep-app-inspection enable

        set options allow-dns

        set control-default-network-services disable

    next
end
```

### Validate

* [ ] `extended-log` requirement reviewed.
* [ ] `other-application-action` reviewed.
* [ ] `other-application-log` reviewed.
* [ ] `unknown-application-action` reviewed.
* [ ] `unknown-application-log` reviewed.
* [ ] Default application port enforcement reviewed.
* [ ] SSL-DI signature inclusion reviewed.
* [ ] Deep application inspection reviewed.
* [ ] DNS handling reviewed.
* [ ] Default network service control reviewed.

---

## Category Rule

```cli
config application list
    edit "app-cont-test"

        config entries
            edit 1
                set category 2 6
                set protocols all
                set vendor all
                set technology all
                set behavior all
                set popularity 1 2 3 4 5
                set action block
                set log enable
                set log-packet disable
                set session-ttl 0
                set quarantine attacker
                set quarantine-expiry 5
                set quarantine-log enable
            next
        end

    next
end
```

### Validate

* [ ] Category selection is correct.
* [ ] Protocol selection is correct.
* [ ] Vendor matching is reviewed.
* [ ] Technology matching is reviewed.
* [ ] Behavior matching is reviewed.
* [ ] Popularity matching is reviewed.
* [ ] Action is correct.
* [ ] Logging is enabled where required.
* [ ] Quarantine behavior is intentional.
* [ ] Quarantine expiry is appropriate.

---

# 22. Troubleshooting Checklist

## FortiGuard

Check:

```cli
get system fortiguard
```

### Checklist

* [ ] FortiGuard connectivity verified.
* [ ] Application signature service available.
* [ ] Subscription/license status verified.
* [ ] FortiGuard services operational.
* [ ] Application signatures current.

---

## WAD

Diagnostic:

```cli
diagnose test app wad 1000
```

### Checklist

* [ ] Check WAD worker/process information.
* [ ] Review proxy-related processing.
* [ ] Review web/application-related processing.
* [ ] Correlate WAD behavior with affected sessions.

---

## WAD Debug

```cli
diagnose wad debug enable level verbose
```

Example category:

```cli
diagnose wad debug enable category video
```

### Checklist

* [ ] Enable debugging only when required.
* [ ] Reproduce the issue.
* [ ] Capture relevant output.
* [ ] Disable debugging after testing.
* [ ] Avoid unnecessary verbose debugging on busy production systems.

---

## Application Detection Failure

When an application is not detected:

* [ ] Verify firewall policy.
* [ ] Verify Application Control profile attachment.
* [ ] Verify FortiGuard connectivity.
* [ ] Verify application signatures.
* [ ] Verify traffic is actually passing through the expected policy.
* [ ] Verify SSL inspection requirements.
* [ ] Verify whether traffic is encrypted.
* [ ] Verify QUIC/HTTP3 behavior.
* [ ] Verify application is not categorized as `unknown`.
* [ ] Verify application is not categorized as `other`.
* [ ] Check non-standard port behavior.
* [ ] Check deep application inspection requirements.
* [ ] Review relevant diagnostics.

---

# 23. Production Deployment Checklist

## Firewall Policy

* [ ] Correct firewall policy identified.
* [ ] Correct source configured.
* [ ] Correct destination configured.
* [ ] Correct service configuration reviewed.
* [ ] Correct inspection mode configured.
* [ ] Application Control profile attached.
* [ ] SSL/SSH inspection configured where required.
* [ ] Application logging configured.
* [ ] Policy order reviewed.

---

## Application Visibility

* [ ] FortiGuard connectivity verified.
* [ ] Application signatures updated.
* [ ] Application categories reviewed.
* [ ] Application risk levels reviewed.
* [ ] Application vendors reviewed where relevant.
* [ ] Technology classification reviewed.
* [ ] Behavior classification reviewed.
* [ ] Popularity classification reviewed.
* [ ] Unknown applications policy defined.
* [ ] Other applications policy defined.

---

## Port Enforcement

* [ ] Default application ports reviewed.
* [ ] Non-standard application ports tested.
* [ ] Default network services reviewed.
* [ ] Violation action tested.
* [ ] Legitimate exceptions documented.

---

## SSL / HTTPS

* [ ] Encrypted traffic identified.
* [ ] SSL inspection requirement evaluated.
* [ ] Deep Inspection configured where appropriate.
* [ ] CA certificate deployed to required clients.
* [ ] SSL-related application signatures reviewed.
* [ ] Decrypted application visibility tested.
* [ ] Application Control behavior tested after decryption.

---

## QUIC / HTTP3

* [ ] UDP/443 identified.
* [ ] QUIC behavior reviewed.
* [ ] HTTP/3 applications tested.
* [ ] QUIC policy defined.
* [ ] Application Control tested against QUIC.
* [ ] Web Filter compatibility tested.
* [ ] SSL inspection impact evaluated.
* [ ] User experience impact evaluated.

---

## Quarantine

* [ ] Quarantine action reviewed.
* [ ] Quarantine duration configured.
* [ ] Quarantine logging enabled.
* [ ] Client quarantine tested.
* [ ] Quarantine expiration tested.
* [ ] FortiView monitoring verified.
* [ ] Operational recovery procedure documented.

---

## SCADA / ICS

* [ ] Industrial protocols identified.
* [ ] Modbus traffic identified.
* [ ] DNP3 traffic identified.
* [ ] GTP requirements evaluated.
* [ ] Industrial signatures reviewed.
* [ ] Protocol decoders verified.
* [ ] False positives tested.
* [ ] Legitimate write operations tested.
* [ ] Maintenance window defined.
* [ ] Enforcement approved by OT/security stakeholders.

---

# 24. Performance Checklist

Application Control can add inspection workload, especially when deeper inspection is enabled.

### Monitor

* [ ] CPU utilization.
* [ ] Memory utilization.
* [ ] Concurrent sessions.
* [ ] Concurrent application sessions.
* [ ] Average file/session size where relevant.
* [ ] High-volume applications.
* [ ] Deep inspection workload.
* [ ] QUIC/HTTP3 workload.
* [ ] Logging volume.
* [ ] FortiGuard lookup/update behavior.
* [ ] Conserve-mode events.
* [ ] Latency.

### Performance Model

```text
More Inspection
      ↓
More Processing
      ↓
CPU / Memory Pressure
      ↓
Potential Latency
      ↓
Potential Conserve Mode
```

### Golden Rule

> **More inspection ≠ automatically more security.**

### Production Checklist

* [ ] Do not enable every advanced inspection feature globally without testing.
* [ ] Establish baseline CPU/memory measurements.
* [ ] Test with realistic traffic volume.
* [ ] Test peak concurrency.
* [ ] Test large-volume applications.
* [ ] Test deep inspection impact.
* [ ] Test failover/recovery behavior.
* [ ] Document sizing assumptions.

---

# 25. NSE Exam Traps

## 🧠 Trap #1 — Port ≠ Application

* [ ] Remember:

```text
TCP/443 ≠ Always HTTPS
TCP/80  ≠ Always HTTP
```

* [ ] Application Control can identify applications using protocol analysis and signatures.
* [ ] Applications may operate on non-standard ports.

---

## 🧠 Trap #2 — Other vs Unknown

* [ ] Remember:

```text
Known application
but does not match configured rule
        ↓
      OTHER
```

* [ ] Remember:

```text
Application cannot be identified
        ↓
     UNKNOWN
```

---

## 🧠 Trap #3 — Category vs Application

* [ ] Category rules can match multiple applications.
* [ ] Application rules target specific application IDs.

```text
Category
   ↓
Multiple Applications

Application ID
   ↓
Specific Application
```

---

## 🧠 Trap #4 — Port Enforcement

* [ ] Do not confuse detection with enforcement.

```text
Application Detection
        ≠
Port Enforcement
```

* [ ] Detection asks:

```text
"What is this?"
```

* [ ] Port enforcement asks:

```text
"Is it using the expected/default port?"
```

---

## 🧠 Trap #5 — QUIC

* [ ] Remember HTTP/3 → QUIC → UDP.
* [ ] Remember UDP/443.
* [ ] Do not assume TCP/443 controls all modern web traffic.

```text
HTTP/3
  ↓
QUIC
  ↓
UDP/443
```

---

## 🧠 Trap #6 — Quarantine

* [ ] `block` blocks matching traffic.
* [ ] `quarantine` places the violating client into quarantine.

```text
BLOCK
→ Traffic enforcement

QUARANTINE
→ Client-level quarantine state
```

---

## 🧠 Trap #7 — SSL Deep Inspection

* [ ] Encrypted payload visibility may be limited without decryption.
* [ ] Deep Inspection can expose decrypted application data to security inspection.

```text
Encrypted
    ↓
SSL Deep Inspection
    ↓
Decrypted Payload
    ↓
Application Control
```

---

## 🧠 Trap #8 — SSL-DI Signatures

* [ ] Understand `require_ssl_di`.
* [ ] Understand `force-inclusion-ssl-di-sigs`.
* [ ] Do not assume they mean exactly the same thing.
* [ ] Understand that predefined SSL-DI-related signatures have specific behavior.

---

## 🧠 Trap #9 — SCADA

* [ ] Do not rely only on IP + port.
* [ ] Understand protocol-specific inspection.

```text
Modbus
→ Function Codes

DNP3
→ Protocol / Function Parameters

GTP
→ Tunnel / Subscriber Parameters
```

---

## 🧠 Trap #10 — Unknown Applications

* [ ] If FortiGate cannot identify an application with available signatures, it may be classified as unknown.
* [ ] Verify `unknown-application-action`.

Example:

```cli
set unknown-application-action block
```

---

# 26. Interview Quick Check

### Q1. What does Application Control do?

* [ ] Identifies applications.
* [ ] Classifies applications.
* [ ] Applies actions to applications.
* [ ] Provides application visibility and control.

### Q2. Does Application Control rely only on ports?

* [ ] **No.**
* [ ] Understand IPS protocol decoders and application signatures.

### Q3. Can an application be detected on a non-standard port?

* [ ] **Potentially yes**, if the applicable decoder/signature can identify it.

### Q4. What is the difference between Other and Unknown?

* [ ] `Other` = known/signature-related application that does not match the configured rule conditions.
* [ ] `Unknown` = application cannot be identified.

### Q5. What is port enforcement?

* [ ] Enforcement of expected/default application ports.

### Q6. What protocol does QUIC use?

* [ ] UDP.

### Q7. What does HTTP/3 commonly use?

* [ ] QUIC over UDP.
* [ ] Commonly UDP/443.

### Q8. What does quarantine do?

* [ ] Places the violating client into a quarantine state according to configured behavior.

### Q9. Why is SSL inspection important?

* [ ] It can provide visibility into encrypted payloads required for deeper application inspection.

### Q10. What are examples of industrial protocol decoders?

* [ ] GTP.
* [ ] Modbus.
* [ ] DNP3.

---

# 27. 60-Second Memory Map

```text
                 APPLICATION CONTROL
                         |
                         v
              "WHAT APPLICATION IS THIS?"
                         |
             +-----------+-----------+
             |           |           |
             v           v           v
          Decoder     Signature   App Database
             |           |           |
             +-----------+-----------+
                         |
                         v
                  APPLICATION ID
                         |
                         v
                 "HOW IS IT CLASSIFIED?"
                         |
       +---------+-------+--------+---------+
       |         |       |        |         |
    Category    Risk   Vendor Technology Behavior
                         |
                         +── Popularity
                         |
                         v
                  "WHAT SHOULD WE DO?"
                         |
          +------+------+------+------+
          |      |      |      |      |
         PASS  MONITOR BLOCK RESET QUARANTINE
                         |
                         v
               "RIGHT DEFAULT PORT?"
                         |
          +--------------+--------------+
          |                             |
          v                             v
     App Port                       Network Service
     Enforcement                    Enforcement
                         |
                         v
                  "IS IT ENCRYPTED?"
                         |
                         v
                   SSL Inspection
                         |
                         v
                Deep Application Inspection
                         |
                         v
                    "MODERN WEB?"
                         |
                         v
                  QUIC / HTTP3 / UDP443
                         |
                         v
                    "INDUSTRIAL?"
                         |
              +----------+----------+
              |          |          |
             GTP      Modbus      DNP3
```

---

# 28. Most Important CLI

## FortiGuard Status

```cli
get system fortiguard
```

* [ ] Verify FortiGuard connectivity.
* [ ] Verify service availability.
* [ ] Verify application-signature dependencies.

---

## Application Control Profile

```cli
config application list
    edit "app-cont-test"
```

* [ ] Verify correct profile.
* [ ] Verify policy attachment.

---

## Unknown Applications

```cli
set unknown-application-action block
```

* [ ] Define explicit unknown-application behavior.

---

## Other Applications

```cli
set other-application-action pass
```

* [ ] Define explicit other-application behavior.

---

## Port Enforcement

```cli
set enforce-default-app-port enable
```

* [ ] Review expected/default ports.
* [ ] Test non-standard ports.

---

## Deep Application Inspection

```cli
set deep-app-inspection enable
```

* [ ] Enable only where justified.
* [ ] Monitor performance.

---

## SSL-DI Signatures

```cli
set force-inclusion-ssl-di-sigs enable
```

* [ ] Review SSL inspection architecture.
* [ ] Verify decrypted traffic.

---

## Application Entries

```cli
config entries
```

* [ ] Configure application/category matching.
* [ ] Configure action.
* [ ] Configure logging.
* [ ] Configure exclusions where required.

---

## WAD Diagnostic

```cli
diagnose test app wad 1000
```

* [ ] Review WAD worker/process information.

---

## WAD Debug

```cli
diagnose wad debug enable level verbose
```

* [ ] Enable only during controlled troubleshooting.
* [ ] Disable after troubleshooting.

---

# 🔥 SheynShield Field Rules

> **Rule #1:**
> `Application Control ≠ Port Filtering`

> **Rule #2:**
> **Port ≠ Application Identity.** Application Control can use protocol decoders and signatures.

> **Rule #3:**
> `Other ≠ Unknown`

```text
Known but not matched → OTHER
Not identified        → UNKNOWN
```

> **Rule #4:**
> **Application Detection ≠ Port Enforcement.**

> **Rule #5:**
> HTTP/3 commonly uses:

```text
HTTP/3
 ↓
QUIC
 ↓
UDP/443
```

> **Rule #6:**
> Encrypted traffic requires appropriate inspection architecture when deeper payload visibility is necessary.

> **Rule #7:**
> Deep Application Inspection can improve identification but can also increase resource consumption.

> **Rule #8:**
> For OT/ICS networks, never blindly block protocol functions without validating the operational impact.

> **Rule #9:**
> Application Control should be combined with appropriate security controls such as SSL inspection, Web Filter, IPS, and other security profiles where required.

> **Rule #10:**
> Always validate behavior against the **exact FortiOS release and FortiGate model** before using a lab configuration as a production standard.

---

# ⚡ Final NSE Revision Card

```text
APPLICATION CONTROL
│
├── IDENTIFICATION
│   ├── IPS Protocol Decoder
│   ├── Application Signature
│   └── Application Database
│
├── CLASSIFICATION
│   ├── Application
│   ├── Category
│   ├── Protocol
│   ├── Risk
│   ├── Vendor
│   ├── Technology
│   ├── Behavior
│   └── Popularity
│
├── ACTION
│   ├── Pass
│   ├── Monitor
│   ├── Block
│   ├── Reset
│   └── Quarantine
│
├── SPECIAL CONTROL
│   ├── Default App Port Enforcement
│   ├── Network Protocol Enforcement
│   ├── Deep App Inspection
│   ├── SSL-DI Signatures
│   └── QUIC
│
├── APPLICATION STATES
│   ├── Known
│   ├── Other
│   └── Unknown
│
└── INDUSTRIAL
    ├── GTP
    ├── Modbus
    └── DNP3
```

---

# 🧠 One-Line NSE Memory Aid

> **Application Control identifies applications through protocol decoders and application signatures, classifies them using attributes such as category, risk, vendor, technology, behavior, and popularity, and then applies actions such as pass, monitor, block, reset, or quarantine—with additional controls for application ports, SSL inspection, QUIC/HTTP3, and industrial protocols.**

---

# 🔎 Keywords

```text
FortiGate Application Control
FortiOS Application Control
FortiGate application control checklist
FortiGate application signatures
FortiGate IPS protocol decoder
FortiGate application identification
FortiGate application quarantine
FortiGate application port enforcement
FortiGate default application port
FortiGate unknown application
FortiGate other application
FortiGate QUIC
FortiGate HTTP/3
FortiGate UDP 443
FortiGate deep application inspection
FortiGate SSL application detection
FortiGate SSL DI signatures
FortiGate SCADA inspection
FortiGate Modbus inspection
FortiGate DNP3 inspection
FortiGate GTP decoder
FortiOS NSE Application Control
Fortinet Application Control
Fortinet NSE Application Control
FortiGate troubleshooting
FortiGate security profile
FortiGate application visibility
```

---

# 🔗 Related SheynShield Topics

* [ ] FortiGate Antivirus & File Inspection
* [ ] FortiGate SSL/SSH Deep Inspection
* [ ] FortiGate IPS
* [ ] FortiGate Web Filter
* [ ] FortiGate DNS Filter
* [ ] FortiGate File Filter
* [ ] FortiGate Security Profiles
* [ ] FortiSandbox
* [ ] FortiGuard
* [ ] FortiClient EMS
* [ ] FortiGate Conserve Mode
* [ ] FortiGate Troubleshooting
* [ ] FortiGate SCADA / ICS Security

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

## 🛡️ SheynShield

**Engineering Secure Networks**

> Practical Network Security • Fortinet • Firewall Engineering • Security Architecture • Troubleshooting • NSE Knowledge

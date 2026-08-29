# 🛡️ FortiOS 7.2.0 Security Fabric Deployment & Troubleshooting Checklist

> **SheynShield | Engineering Secure Networks**  
> Enterprise checklist for **FortiOS 7.2.0 Security Fabric design, synchronization, automation, external connectors, SAML integration, FortiManager, FortiAnalyzer Cloud, EMS, OT/Purdue security, and troubleshooting workflows.**

---

# 📌 Table of Contents

- [Security Fabric Architecture Checklist](#security-fabric-architecture-checklist)
- [Fabric Synchronization Checklist](#fabric-synchronization-checklist)
- [Security Fabric Modes Checklist](#security-fabric-modes-checklist)
- [VDOM Design Checklist](#vdom-design-checklist)
- [IPsec Fabric Connectivity Checklist](#ipsec-fabric-connectivity-checklist)
- [Fabric Diagnostic Commands](#fabric-diagnostic-commands)
- [Automation Checklist](#automation-checklist)
- [Automation Troubleshooting](#automation-troubleshooting)
- [External Connector Checklist](#external-connector-checklist)
- [Threat Feed Checklist](#threat-feed-checklist)
- [SAML Security Fabric Checklist](#saml-security-fabric-checklist)
- [ADFS Integration Checklist](#adfs-integration-checklist)
- [FortiAuthenticator SAML Checklist](#fortiauthenticator-saml-checklist)
- [FortiAnalyzer Cloud Checklist](#fortianalyzer-cloud-checklist)
- [FortiManager Checklist](#fortimanager-checklist)
- [EMS Integration Checklist](#ems-integration-checklist)
- [FortiNAC Integration Checklist](#fortinac-integration-checklist)
- [OT Purdue Security Checklist](#ot-purdue-security-checklist)
- [Security Rating Checklist](#security-rating-checklist)
- [Golden Troubleshooting Workflow](#golden-troubleshooting-workflow)
- [High Value Exam Notes](#high-value-exam-notes)

---

# 🧭 Security Fabric Architecture Checklist

## Root FortiGate Design

Validate:

- [ ] Root FortiGate selected
- [ ] Fabric Coordinator role assigned
- [ ] Downstream devices identified
- [ ] Security Fabric topology documented

Architecture:

```text
                 ROOT FORTIGATE
              Fabric Coordinator

                      |
        --------------------------------

        |              |              |

        ▼              ▼              ▼

  Downstream FGT   FortiSwitch     FortiAP

        |
        |
        ├── EMS
        ├── FortiAnalyzer
        ├── FortiManager
        ├── FortiSandbox
        └── FortiNAC
````

---

# 🔄 Fabric Synchronization Checklist

## Enable Fabric Object Synchronization

Configuration:

```bash
config system csf
    set fabric-object-unification enable
end
```

Validate:

* [ ] Root objects synchronize correctly
* [ ] Downstream devices receive required objects
* [ ] Object ownership documented

---

## Configuration Synchronization

Validate:

```bash
config system csf
    set configuration-sync local
end
```

Check integrations:

* [ ] FortiManager
* [ ] FortiAnalyzer
* [ ] FortiSandbox
* [ ] EMS

---

## Fabric Worker Configuration

Verify:

```bash
config system csf
    set fabric-workers 2
end
```

Checklist:

* [ ] Worker count reviewed
* [ ] Device scale considered
* [ ] Synchronization performance monitored

---

# ⚙️ Security Fabric Mode Checklist

Understand:

| Mode               | Purpose                                 |
| ------------------ | --------------------------------------- |
| Default / Unified  | Root object inheritance                 |
| Local              | Device maintains own values             |
| Configuration Sync | External Fabric service synchronization |

---

# ⚠️ Local Device Trap Checklist

Scenario:

```text
Root FGT

   |

Local Device

   |

Downstream Device
```

Validate:

* [ ] Local device behavior understood
* [ ] Object visibility tested
* [ ] Downstream inheritance verified

Remember:

```text
Object Reference

        ↓

Downstream Advertisement

        ↓

Later root changes

        ≠

Guaranteed downstream modification
```

---

# 🏢 VDOM Design Checklist

Before enabling VDOM:

* [ ] Backup configuration
* [ ] Validate addressing
* [ ] Validate NPU impact
* [ ] Validate Fabric behavior

Important:

* [ ] Fabric synchronization is not supported on multi-mode VDOMs

---

# 🔐 IPsec Security Fabric Checklist

## IPsec Tunnel Preparation

Validate:

* [ ] Site-to-site tunnel created
* [ ] Interface mode enabled
* [ ] Encryption configured
* [ ] Authentication configured
* [ ] Routing completed

Flow:

```text
FortiGate A

     |

   IPsec

     |

FortiGate B
```

---

## After Tunnel Creation

Do not stop at Phase 1/2.

Verify:

* [ ] Tunnel status
* [ ] Firewall policy
* [ ] Routing
* [ ] Interface IP connectivity
* [ ] Administrative access
* [ ] Fabric communication

Test:

```bash
execute ping <REMOTE-FGT-IP>
```

---

# 🔎 Security Fabric Diagnostic Commands

## Fabric Status

```bash
diagnose sys csf
```

---

## Pending Authorization

```bash
diagnose sys csf authorization pending-list
```

---

## Downstream Devices

```bash
diagnose sys csf downstream
```

---

## Upstream Information

```bash
diagnose sys csf upstream
```

---

## Configuration Verification

```bash
show system csf
```

---

# 🤖 Automation Checklist

## Automation Design

Validate:

* [ ] Trigger defined
* [ ] Action defined
* [ ] Stitch created
* [ ] Schedule tested
* [ ] Execution order reviewed

---

## Concurrent Stitch Configuration

Command:

```bash
config system automation-setting
    set max-concurrent-stitches 128
end
```

Range:

```text
32 - 256
```

---

## Execution Strategy

Choose:

### Parallel

Use when:

* [ ] Speed required
* [ ] Commands independent

### Sequential

Use when:

* [ ] Order matters
* [ ] Delay required
* [ ] Recovery actions depend on previous steps

---

# 🧪 Automation Troubleshooting Checklist

## Test Schedule

```bash
diagnose automation test x-schedule
```

---

## Automation Daemon

```bash
diagnose test application autod
```

---

## Debug Automation

```bash
diagnose debug enable
diagnose debug application autod -1
```

Stop:

```bash
diagnose debug disable
```

---

# 🌐 External Connector Checklist

Validate connectors:

* [ ] Identity connectors
* [ ] Endpoint connectors
* [ ] Threat intelligence connectors
* [ ] Third-party integrations

Examples:

```text
FSSO

Symantec Endpoint Protection

RADIUS SSO Agent

Exchange Server

STIX/TAXII Feed
```

---

# 🚨 Threat Feed Checklist

## Threat Feed Architecture

Validate:

```text
External Source

      ↓

Threat Feed

      ↓

Dynamic Object

      ↓

Firewall Policy

      ↓

Enforcement
```

---

## Supported Feed Types

Validate:

* [ ] IPv4
* [ ] IPv6
* [ ] Domains
* [ ] URLs
* [ ] Malware hashes
* [ ] STIX/TAXII

---

## Threat Feed Design Rules

Validate:

* [ ] One entry per line
* [ ] Duplicate entries removed
* [ ] Feed size monitored

Example:

```text
192.168.10.10

192.168.20.20

192.168.30.0/24
```

---

# 📊 Threat Feed Capacity Checklist

Training reference:

```text
Maximum file size:

10 MB


Maximum entries:

131072
```

Validate:

```bash
print tablesize
```

Monitor:

```text
system.external-resource
```

---

# 🔍 Dynamic Object Validation

Command:

```bash
diagnose firewall dynamic list
```

Verify:

* [ ] Connector status
* [ ] Feed entries
* [ ] Dynamic object creation
* [ ] Policy matching

---

# 🔑 SAML Security Fabric Checklist

## Authentication Model

Remember:

```text
Authentication

"What is your identity?"

+

Authorization

"What can you access?"
```

---

## Security Fabric SAML Flow

Architecture:

```text
ROOT FORTIGATE

      |

     IDP

      |

    SAML

      |

DOWNSTREAM FORTIGATE

      |

      SP
```

Rule:

```text
Root = Identity Provider

Downstream = Service Provider
```

---

# 🏢 ADFS Integration Checklist

Validate:

* [ ] Active Directory configured
* [ ] AD CS configured
* [ ] ADFS installed
* [ ] SSL certificate assigned
* [ ] Relying Party Trust created
* [ ] SAML protocol enabled

Flow:

```text
Active Directory

        ↓

AD CS

        ↓

ADFS

        ↓

SAML

        ↓

FortiGate
```

---

# 📜 Certificate Checklist

Validate:

* [ ] Certificate template created
* [ ] Private key export enabled
* [ ] Correct CN configured
* [ ] DNS name included
* [ ] CA certificate exported
* [ ] FortiGate trusts CA

Example:

```text
CN=adfs.example.com
```

---

# 🧱 FortiAuthenticator SAML Checklist

Architecture:

```text
Active Directory

       ↓

FortiAuthenticator

       ↓

SAML

       ↓

FortiGate
```

Validate:

* [ ] LDAP configured
* [ ] User source created
* [ ] SAML IDP configured
* [ ] Attributes mapped
* [ ] Certificate installed

---

# ☁️ FortiAnalyzer Cloud Checklist

Validate:

* [ ] Cloud subscription available
* [ ] Logging enabled
* [ ] Gateway path available
* [ ] Logs received

Configuration:

```bash
config log fortiguard setting
    set status enable
    set upload-option realtime
end
```

---

## Cloud Log Verification

Commands:

```bash
execute log filter device fortianalyzer-cloud
```

```bash
execute log filter category event
```

```bash
execute log display
```

---

# 🏢 FortiManager Checklist

Validate:

* [ ] FortiManager reachable
* [ ] Central management enabled
* [ ] Device registration completed
* [ ] TCP/541 connectivity available

Cloud verification:

```bash
diagnose fdsm central-mgmt-status
```

---

# 🖥️ EMS Integration Checklist

Architecture:

```text
FortiGate

   |

FortiClient EMS

   |

Endpoint Intelligence
```

Validate:

* [ ] EMS reachable
* [ ] Certificate trusted
* [ ] Tags synchronized
* [ ] Vulnerabilities synchronized
* [ ] Malware hash information synchronized

---

## EMS Certificate Troubleshooting

Verify:

```bash
execute fctems verify <EMS_NAME>
```

If trust fails:

* [ ] Export EMS certificate
* [ ] Import certificate
* [ ] Retry connection

---

## EMS Fabric Authentication

Validate:

```bash
config endpoint-control fctems
```

Capabilities:

```text
fabric-auth

silent-approval

websocket

push-ca-cert
```

---

# ⚡ ZTNA Tag Synchronization Checklist

Validate:

* [ ] REST API communication
* [ ] TCP/8013 connectivity
* [ ] WebSocket status
* [ ] Dynamic tag creation

Check:

```bash
diagnose test application fcnacd 2
```

Verify:

```bash
diagnose firewall dynamic list
```

---

# 🏭 OT Purdue Security Checklist

## Purdue Model Validation

| Level | Function          |
| ----- | ----------------- |
| 0     | Physical process  |
| 1     | PLC / Control     |
| 2     | SCADA / HMI       |
| 3     | Manufacturing     |
| DMZ   | Security boundary |
| 4     | Enterprise        |

---

## OT Security Objectives

Validate:

* [ ] Safety
* [ ] Availability
* [ ] Segmentation
* [ ] Visibility
* [ ] Incident response

---

# 🧱 OT Security Fabric Checklist

Enable:

```text
System

 ↓

Feature Visibility

 ↓

OT
```

Validate:

* [ ] Asset Identity Center
* [ ] OT View
* [ ] Device classification

---

# 🔄 Purdue Level Adjustment

Command:

```bash
diagnose user-device-store device memory ot-prudue-set max-ip-level
```

---

# 🛡️ Security Rating Checklist

Disable scheduled execution:

```bash
config system global
    set security-rating-run-on-schedule disable
end
```

Run manually:

```bash
diagnose report-runner trigger
```

Workflow:

```text
Configuration

 ↓

Security Rating

 ↓

Report

 ↓

Remediation
```

---

# 🧭 Golden Security Fabric Troubleshooting Workflow

Follow this order:

```text
1. Interface

↓

2. IP Connectivity

↓

3. Routing

↓

4. Firewall Policy

↓

5. Certificate Trust

↓

6. Authorization

↓

7. Fabric Status

↓

8. Connector Status

↓

9. Process Debug
```

---

# 🧪 Universal FortiOS Debug Pattern

Enable:

```bash
diagnose debug enable

diagnose debug application <PROCESS> -1
```

Examples:

```bash
diagnose debug application autod -1
```

```bash
diagnose debug application kubed -1
```

```bash
diagnose debug application sepmd -1
```

Disable:

```bash
diagnose debug disable
```

---

# 🧠 High Value Exam Memory Checklist

Remember:

* [ ] Root FortiGate = Fabric Coordinator
* [ ] Security Fabric ≠ identical configuration everywhere
* [ ] Local mode changes inheritance behavior
* [ ] VDOM affects synchronization
* [ ] Root FortiGate = SAML IDP
* [ ] Downstream FortiGate = SAML SP
* [ ] Threat Feed = Dynamic object source
* [ ] STIX = Intelligence format
* [ ] TAXII = Intelligence transport
* [ ] EMS = Endpoint intelligence source
* [ ] FortiManager = Central management
* [ ] FortiAnalyzer = Logging and analytics
* [ ] Purdue = OT segmentation model

---

# 🎯 Final Security Fabric Mental Model

```text
                    SECURITY FABRIC

                           |

        --------------------------------------

        |                 |                  |

    Management        Identity            Security

        |                 |                  |

 FortiManager        SAML/EMS          Threat Feed

        |

 FortiAnalyzer

        |

        OT / Purdue

        |

   Segmentation + Visibility + Control
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

**Security Fabric = Connectivity + Trust + Synchronization + Identity + Enforcement**

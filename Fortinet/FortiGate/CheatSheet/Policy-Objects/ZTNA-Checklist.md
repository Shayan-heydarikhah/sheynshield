# 🔐 FortiGate ZTNA Deployment & Operations Checklist — FortiOS 7.2+

> **SheynShield | Engineering Secure Networks**  
> Production-ready checklist for designing, deploying, validating, and troubleshooting **FortiGate ZTNA** environments with **FortiClient EMS, FortiClient, ZTNA Tags, Access Proxy, TCP Forwarding, SSH Proxy, SAML Authentication, and Endpoint Synchronization**.

---

# 📌 Table of Contents

- [Architecture Planning Checklist](#architecture-planning-checklist)
- [Version Compatibility Checklist](#version-compatibility-checklist)
- [FortiClient EMS Preparation](#forticlient-ems-preparation)
- [EMS Identity & Certificate Checklist](#ems-identity--certificate-checklist)
- [FortiGate EMS Integration](#fortigate-ems-integration)
- [ZTNA Tag Design Checklist](#ztna-tag-design-checklist)
- [ZTNA Access Proxy Checklist](#ztna-access-proxy-checklist)
- [ZTNA Authentication Checklist](#ztna-authentication-checklist)
- [SAML Integration Checklist](#saml-integration-checklist)
- [SSH Access Proxy Checklist](#ssh-access-proxy-checklist)
- [TCP Forwarding Checklist](#tcp-forwarding-checklist)
- [Policy Validation Checklist](#policy-validation-checklist)
- [Security Hardening Checklist](#security-hardening-checklist)
- [Performance & Scaling Checklist](#performance--scaling-checklist)
- [Troubleshooting Checklist](#troubleshooting-checklist)
- [Operational Validation Checklist](#operational-validation-checklist)
- [NSE Exam Memory Checklist](#nse-exam-memory-checklist)

---

# 🏗️ Architecture Planning Checklist

## ZTNA Design Decision

- [ ] Define business requirement:

  - [ ] Full network access
  - [ ] Application-specific access
  - [ ] Remote workforce access
  - [ ] BYOD access
  - [ ] Micro-segmentation

- [ ] Decide architecture:

| Requirement | Recommended Solution |
|---|---|
| Access multiple networks/subnets | VPN |
| Access specific applications | ZTNA |
| Application publishing | ZTNA Access Proxy |
| Secure site-to-site | IPsec VPN |
| Device posture enforcement | ZTNA |

---

## ZTNA Security Model Validation

Confirm access decision includes:

- [ ] User identity
- [ ] Device identity
- [ ] Client certificate
- [ ] Endpoint posture
- [ ] ZTNA Tags
- [ ] Application identity
- [ ] Authentication state
- [ ] Network context

Expected model:

```text
WHO?
 +
WHAT DEVICE?
 +
WHAT POSTURE?
 +
WHAT APPLICATION?
 +
WHICH POLICY?
        |
        ▼
   ZTNA Decision
````

---

# 🔄 Version Compatibility Checklist

Validate:

* [ ] FortiGate FortiOS 7.2+
* [ ] FortiClient EMS same release family
* [ ] FortiClient compatible version deployed
* [ ] Windows endpoint requirements verified
* [ ] Platform limitations reviewed

Recommended alignment:

```text
FortiGate 7.2.x

        ↓

FortiClient EMS 7.2.x

        ↓

FortiClient 7.2.x+
```

---

# 🏢 FortiClient EMS Preparation

## EMS Installation

* [ ] Install FortiClient EMS
* [ ] Register EMS with Fortinet account
* [ ] Configure system settings
* [ ] Configure EMS certificate infrastructure
* [ ] Verify EMS connectivity

---

## EMS Certificate Checklist

Validate:

* [ ] EMS certificate exists
* [ ] EMS CA configured
* [ ] FortiClient trusts EMS CA
* [ ] FortiGate trusts EMS certificate
* [ ] Endpoint certificate issuance works

Certificate relationship:

```text
FortiClient

      |

      ▼

FortiClient EMS

      |

      ▼

FortiGate
```

---

# 👤 EMS Authentication Checklist

## LDAP Integration

Validate:

* [ ] LDAP server reachable
* [ ] LDAP service account configured
* [ ] LDAP authentication successful
* [ ] User synchronization verified

Example:

```text
LDAP

IP:
Port:
Bind Account:
```

Security:

* [ ] Never store passwords in GitHub
* [ ] Never publish API tokens
* [ ] Never publish private keys

---

# 🖥️ Endpoint Management Checklist

Verify EMS collects:

* [ ] Device information
* [ ] Operating system
* [ ] Logged-on user
* [ ] Network information
* [ ] Security posture
* [ ] Vulnerability state
* [ ] Endpoint identity

---

# 🏷️ ZTNA Tag Design Checklist

## Create ZTNA Tags

Navigate:

```text
EMS

└── Zero Trust Tags

    └── Rules
```

Validate:

* [ ] Tag naming standard created
* [ ] Tag purpose documented
* [ ] Rules tested
* [ ] Endpoint receives correct tag

---

## Example Tag Logic

```text
Endpoint

 |

 ├── Antivirus Enabled

 ├── OS Version

 ├── Vulnerability State

 ├── Security Posture

        |

        ▼

    ZTNA Tag

        |

        ▼

 FortiGate Policy
```

---

# 🔗 FortiGate EMS Integration Checklist

Navigate:

```text
Security Fabric

└── Fabric Connectors

    └── FortiClient EMS
```

Validate:

* [ ] EMS connector created
* [ ] Certificate trust established
* [ ] Fabric authorization completed
* [ ] EMS connectivity successful
* [ ] Endpoint synchronization active

---

# 🔐 Endpoint Certificate Validation

Verify:

* [ ] FortiClient receives certificate
* [ ] Certificate UID exists
* [ ] Certificate serial number exists
* [ ] FortiGate receives endpoint information

Validation flow:

```text
Client Certificate

        |

        ▼

FortiGate Endpoint Database

        |

        ▼

ZTNA Policy Matching
```

---

# 🚪 ZTNA Access Proxy Checklist

## Access Proxy Preparation

Validate:

* [ ] Public IP available
* [ ] DNS record created
* [ ] SSL certificate installed
* [ ] Virtual host configured
* [ ] Real server reachable

---

## Supported Applications

Validate required services:

* [ ] HTTP
* [ ] HTTPS
* [ ] SSH
* [ ] RDP
* [ ] SMB
* [ ] TCP Applications

---

# 🌐 ZTNA Server Checklist

Verify:

* [ ] ZTNA feature enabled

```text
System

└── Feature Visibility

    └── Zero Trust Network Access
```

Configure:

* [ ] External interface
* [ ] External IP
* [ ] External port
* [ ] Certificate
* [ ] Authentication method

---

# 🧩 Access Proxy Security Checklist

Verify policy evaluates:

* [ ] Client certificate
* [ ] User identity
* [ ] ZTNA Tag
* [ ] Endpoint posture
* [ ] Application

Expected flow:

```text
Client

 |

 ▼

FortiGate Access Proxy

 |

 ├── Certificate Check

 ├── Authentication

 ├── Tag Validation

 └── Policy Match

        |

        ▼

Application Access
```

---

# 🔑 Authentication Checklist

Validate:

* [ ] Local authentication
* [ ] LDAP authentication
* [ ] RADIUS authentication
* [ ] SAML authentication
* [ ] User group mapping

---

# 🔐 SAML + ZTNA Checklist

Validate:

## FortiGate

* [ ] SAML object created
* [ ] SP entity ID configured
* [ ] SSO URL configured
* [ ] SLO URL configured
* [ ] IdP certificate imported

---

## Identity Provider

Validate:

* [ ] FortiGate registered
* [ ] Metadata exchanged
* [ ] Certificate trusted
* [ ] User authentication successful

---

## Cookie Handling

For non-IP authentication:

```bash
set ip-based disable
set web-auth-cookie enable
```

Validate:

* [ ] Authentication session persistence works

---

# 🐧 SSH Access Proxy Checklist

Validate:

* [ ] SSH Access Proxy configured
* [ ] Host key validation enabled
* [ ] SSH certificate configured
* [ ] User authentication works
* [ ] SSH server trusts certificate

---

SSH flow:

```text
FortiClient

↓

FortiGate SSH Proxy

↓

SSH Certificate

↓

SSH Server

↓

Session Established
```

---

# 🔄 TCP Forwarding Checklist

Validate:

* [ ] TCP forwarding required
* [ ] Application supports forwarding
* [ ] Server reachability confirmed
* [ ] Security inspection requirements defined

Remember:

```text
TCP Forwarding ≠ Encryption

```

It does not automatically secure insecure protocols.

---

# 🔥 ZTNA Policy Checklist

Validate:

* [ ] Proxy policy created
* [ ] Source configured
* [ ] Destination configured
* [ ] ZTNA server selected
* [ ] User/group configured
* [ ] ZTNA Tag configured
* [ ] Match logic verified
* [ ] Logging enabled
* [ ] UTM profile attached if required

---

## Tag Matching Logic

Verify:

### AND

```text
Tag A
+
Tag B

=
Both required
```

### OR

```text
Tag A

OR

Tag B

=
One is enough
```

---

# 📊 ZTNA Scaling Checklist

Monitor:

* [ ] Firewall address consumption
* [ ] Dynamic address objects
* [ ] WAD resources
* [ ] EMS synchronization
* [ ] Endpoint count
* [ ] FortiGate model capacity

Important:

```text
1 ZTNA Tag

=

1 IP Address Object

+

1 MAC Address Object
```

---

# ⚡ EMS Fast Convergence Checklist

Validate:

* [ ] Synchronization optimized
* [ ] Websocket capability enabled if required
* [ ] Out-of-sync threshold reviewed
* [ ] Endpoint changes propagate quickly

Critical scenario:

```text
Endpoint Healthy

↓

ZTNA Tag Present

↓

Access Allowed


Endpoint Unhealthy

↓

ZTNA Tag Removed

↓

Access Denied
```

---

# 🧪 Troubleshooting Checklist

## Endpoint

* [ ] FortiClient installed
* [ ] Endpoint registered
* [ ] Certificate exists
* [ ] EMS sees endpoint

Command:

```bash
diagnose endpoint record list
```

---

## EMS Connectivity

Validate:

```bash
diagnose endpoint fctems test-connectivity <EMS>
```

Certificate:

```bash
execute fctems verify <EMS>
```

---

## ZTNA Tags

Check:

```bash
diagnose firewall dynamic list
```

Validate:

* [ ] Tag exists
* [ ] Address created
* [ ] Policy matches

---

## WAD Debugging

Commands:

```bash
diagnose wad user list
```

```bash
diagnose wad worker policy list
```

Endpoint query:

```bash
diagnose wad dev query-by uid <UID>
```

---

## FCNACD Debugging

Commands:

```bash
diagnose test application fcnacd 2
```

```bash
diagnose test application fcnacd 7
```

```bash
diagnose test application fcnacd 8
```

---

# 🧭 Golden Troubleshooting Workflow

Follow this order:

```text
1. FortiClient Registration

        ↓

2. EMS Endpoint Visibility

        ↓

3. Client Certificate

        ↓

4. FortiGate Synchronization

        ↓

5. ZTNA Tag Validation

        ↓

6. Certificate UID/SN Match

        ↓

7. Authentication

        ↓

8. Proxy Policy Match

        ↓

9. Server Reachability

        ↓

10. WAD Session Processing
```

---

# 🛡️ Production Security Checklist

* [ ] Use valid certificates
* [ ] Disable unnecessary empty certificates
* [ ] Enable logging
* [ ] Monitor failed authentication
* [ ] Review ZTNA tags periodically
* [ ] Document access policies
* [ ] Avoid excessive tag creation
* [ ] Review endpoint posture rules
* [ ] Protect EMS credentials

---

# 🧠 NSE Exam Memory Checklist

Remember:

* [ ] EMS = Endpoint identity source
* [ ] FortiClient = Device identity + posture collector
* [ ] FortiGate = ZTNA enforcement point
* [ ] ZTNA Tag = Dynamic security attribute
* [ ] Client Certificate = Device trust
* [ ] SAML = User identity
* [ ] Access Proxy = Application publishing
* [ ] VPN = Network access
* [ ] ZTNA = Application access

---

# 🎯 Final ZTNA Mental Model

```text
FortiClient

(Device Identity + Posture)

          |

          ▼

FortiClient EMS

(Tags + Certificate + Telemetry)

          |

          ▼

FortiGate ZTNA

(Policy Enforcement)

          |

          ▼

Access Proxy

          |

          ▼

Internal Application
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
**ZTNA = Identity + Device Trust + Posture + Application + Policy**


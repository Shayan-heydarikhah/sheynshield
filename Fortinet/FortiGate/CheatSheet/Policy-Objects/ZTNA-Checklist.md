# 🔐 FortiGate ZTNA Deployment & Troubleshooting Checklist
## FortiOS 7.2+ | FortiClient EMS | FortiClient | Access Proxy

> **SheynShield | Engineering Secure Networks**
>
> Practical implementation checklist for FortiGate Zero Trust Network Access (ZTNA).
>
> Covers:
>
> - FortiClient EMS Integration
> - ZTNA Tags
> - Endpoint Identity
> - Access Proxy
> - TCP Forwarding
> - SSH Proxy
> - SAML Authentication
> - EMS Synchronization
> - Troubleshooting Workflow
> - NSE4/NSE7 Exam Preparation

---

# 📌 Table of Contents

- [ZTNA Architecture Checklist](#ztna-architecture-checklist)
- [Version Compatibility Checklist](#version-compatibility-checklist)
- [FortiClient EMS Checklist](#forticlient-ems-checklist)
- [FortiGate EMS Integration Checklist](#fortigate-ems-integration-checklist)
- [ZTNA Tag Checklist](#ztna-tag-checklist)
- [Access Proxy Checklist](#access-proxy-checklist)
- [Authentication Checklist](#authentication-checklist)
- [TCP Forwarding Checklist](#tcp-forwarding-checklist)
- [SSH Access Proxy Checklist](#ssh-access-proxy-checklist)
- [SAML Integration Checklist](#saml-integration-checklist)
- [Troubleshooting Checklist](#troubleshooting-checklist)
- [Production Design Checklist](#production-design-checklist)
- [NSE Memory Map](#nse-memory-map)

---

# 🧠 ZTNA Architecture Checklist

## Core Security Model

ZTNA decision should include:

- [ ] User identity
- [ ] Device identity
- [ ] Client certificate
- [ ] Endpoint posture
- [ ] ZTNA tags
- [ ] Application identity
- [ ] Authentication state
- [ ] Network context

Remember:

```

IP Address ≠ Identity

ZTNA Decision:

User
+
Device
+
Posture
+
Application
+
Policy

=
Access Decision

```

---

# 🔄 Version Compatibility Checklist

## Platform Alignment

- [ ] FortiGate running FortiOS 7.2+
- [ ] FortiClient EMS version aligned with FortiGate
- [ ] FortiClient version aligned with EMS
- [ ] Endpoint OS compatibility verified

Recommended:

```

FortiGate 7.2.x

```
    ↓
```

FortiClient EMS 7.2.x

```
    ↓
```

FortiClient 7.2.x+

```

---

# 🏢 FortiClient EMS Preparation Checklist

## EMS Installation

- [ ] Install FortiClient EMS
- [ ] Register EMS with Fortinet account
- [ ] Verify licensing
- [ ] Configure EMS certificates
- [ ] Configure EMS CA
- [ ] Verify HTTPS communication


## EMS Authentication

Configure:

```

Administrator
|
└── Authentication Server

```

Checklist:

- [ ] LDAP server configured
- [ ] LDAP connectivity verified
- [ ] Service account created
- [ ] Credentials stored securely


Never commit:

- [ ] LDAP password
- [ ] API token
- [ ] Private key
- [ ] Certificate secret

to GitHub.

---

# 🖥 Endpoint Management Checklist

EMS should collect:

- [ ] Device identity
- [ ] Operating system
- [ ] Logged-in user
- [ ] Security posture
- [ ] Vulnerability state
- [ ] Network information


Verify:

```

FortiClient

```
  ↓
```

FortiClient EMS

```
  ↓
```

FortiGate

```

---

# 🏷 ZTNA Tag Checklist

ZTNA Tags are dynamic security attributes.

Verify:

- [ ] ZTNA rules created
- [ ] Endpoint conditions defined
- [ ] Security posture matching configured
- [ ] Tags assigned correctly


Example:

```

Windows Device

*

Antivirus Enabled

```
    ↓
```

ZTNA Tag

```
    ↓
```

FortiGate Policy Match

```


## Tag Capacity Planning

Important:

```

1 ZTNA Tag

=

1 IP Address Object

*

1 MAC Address Object

```


Calculate:

```

Maximum Tags

≈

Firewall Address Limit / 2

```

---

# 🔗 FortiGate EMS Connector Checklist

Navigate:

```

Security Fabric

└── Fabric Connectors

```
  └── FortiClient EMS
```

```


Verify:

- [ ] EMS reachable
- [ ] Certificate trusted
- [ ] Fabric authorization completed
- [ ] Endpoint synchronization active
- [ ] ZTNA tags synchronized


---

# 🚪 ZTNA Access Proxy Checklist

Access Proxy provides:

- [ ] Application publishing
- [ ] Identity validation
- [ ] Device trust validation
- [ ] Posture checking
- [ ] Micro segmentation


Supported:

- [ ] HTTPS
- [ ] HTTP
- [ ] SSH
- [ ] RDP
- [ ] SMB
- [ ] TCP applications


Architecture:

```

Client

↓

FortiGate Access Proxy

↓

Internal Application

```

---

# 🌐 ZTNA Server Checklist

Verify:

- [ ] ZTNA feature enabled
- [ ] External interface configured
- [ ] Public IP configured
- [ ] Certificate assigned
- [ ] Virtual host configured
- [ ] Real server configured


Example:

```

app.company.com

```
    ↓
```

FortiGate Access Proxy

```
    ↓
```

Internal Server

```

---

# 🔥 Proxy Policy Checklist

Verify:

- [ ] Source configured
- [ ] Destination configured
- [ ] Access Proxy selected
- [ ] ZTNA tag configured
- [ ] User group configured
- [ ] Authentication configured
- [ ] Logging enabled


Tag logic:

```

AND

=
All tags required

OR

=
Any tag accepted

```

---

# 🔐 Certificate Authentication Checklist

Verify:

- [ ] EMS issues endpoint certificate
- [ ] FortiGate trusts EMS CA
- [ ] Certificate UID matches endpoint
- [ ] Certificate serial matches endpoint record


Failure:

```

Certificate mismatch

```
    ↓
```

ZTNA authentication failure

```

---

# 🔑 SAML Authentication Checklist

Verify:

- [ ] SAML server configured
- [ ] Identity Provider configured
- [ ] SP entity ID configured
- [ ] SSO URL configured
- [ ] Certificate imported
- [ ] User group mapping completed


Authentication flow:

```

User

↓

FortiGate

↓

SAML IdP

↓

Authentication

↓

ZTNA Policy

↓

Application

```

---

# 🧪 TCP Forwarding Checklist

Use TCP forwarding for:

- [ ] RDP
- [ ] SSH
- [ ] FTPS
- [ ] Other TCP applications


Verify:

- [ ] Real server reachable
- [ ] Correct service configured
- [ ] Security inspection requirements understood


Remember:

```

TCP Forwarding

≠

Encryption

```

---

# 🔐 SSH Access Proxy Checklist

Verify:

- [ ] SSH proxy configured
- [ ] SSH host key validation enabled
- [ ] SSH client certificate configured
- [ ] User identity validated


Security features:

- [ ] Device trust
- [ ] User authentication
- [ ] SSH certificate signing
- [ ] Host key verification

---

# 🔄 EMS Fast Convergence Checklist

For large deployments:

- [ ] Websocket capability enabled
- [ ] Push CA enabled
- [ ] Synchronization timeout optimized
- [ ] Out-of-sync threshold reviewed


Goal:

```

Endpoint Posture Change

```
    ↓
```

EMS Update

```
    ↓
```

FortiGate Sync

```
    ↓
```

ZTNA Policy Re-evaluation

````

---

# 🧪 Troubleshooting Checklist

## Step 1 — Endpoint Registration

Check:

```bash
diagnose endpoint record list
````

Verify:

* [ ] Endpoint exists
* [ ] UID available
* [ ] Certificate available

---

## Step 2 — EMS Connectivity

Test:

```bash
diagnose endpoint fctems test-connectivity <EMS>
```

Verify:

* [ ] FortiGate reaches EMS
* [ ] Certificate validation succeeds

---

## Step 3 — Dynamic Tags

Check:

```bash
diagnose firewall dynamic list
```

Verify:

* [ ] ZTNA tags exist
* [ ] Dynamic addresses created

---

## Step 4 — WAD Troubleshooting

Check:

```bash
diagnose wad user list
```

```bash
diagnose wad worker policy list
```

Verify:

* [ ] User authentication
* [ ] Policy matching
* [ ] Proxy session

---

# 🧰 Golden Troubleshooting Workflow

```
1. FortiClient Registered?

        ↓

2. EMS Knows Endpoint?

        ↓

3. Certificate Exists?

        ↓

4. FortiGate Endpoint Sync?

        ↓

5. ZTNA Tag Available?

        ↓

6. Authentication Successful?

        ↓

7. Proxy Policy Match?

        ↓

8. Internal Server Reachable?

        ↓

9. WAD Session Working?
```

---

# 🏗 Production Design Checklist

## Use ZTNA When:

* [ ] Application-level access required
* [ ] Micro segmentation required
* [ ] Device posture required
* [ ] BYOD control required

## Use VPN When:

* [ ] Full network access required
* [ ] Multiple subnet access required
* [ ] Site connectivity required

Decision:

```
Application Access

        ↓

ZTNA


Network Access

        ↓

VPN
```

---

# 🧠 NSE Memory Map

```
                 ZTNA

                  |

        ---------------------

        |          |        |

       EMS    FortiClient  FortiGate

        |          |        |

       Tags   Certificate  Proxy

       CA     Identity     Policy

       User   Posture      SAML

                  |

                  ↓

            ZTNA Decision

                  |

          Allow / Deny
```

---

# ⭐ Golden Rules

* [ ] ZTNA is application-centric
* [ ] EMS provides endpoint intelligence
* [ ] FortiClient provides device context
* [ ] FortiGate enforces access decisions
* [ ] Certificates represent device identity
* [ ] SAML represents user identity
* [ ] Tags represent dynamic security posture
* [ ] VPN and ZTNA solve different problems
* [ ] Troubleshoot from endpoint → EMS → FortiGate → Policy → Server

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

# 🛡 SheynShield

**Security Engineering Knowledge Base**

Fortinet | Network Security | Firewall | ZTNA | Zero Trust Architecture


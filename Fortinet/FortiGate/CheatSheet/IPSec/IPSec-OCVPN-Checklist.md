# 🔐 FortiGate OCVPN (Overlay Controller VPN) — Configuration & Troubleshooting Checklist

> **SheynShield | FortiGate Network Security Engineering**
>
> A practical FortiGate OCVPN checklist covering **FortiCloud orchestration, Full-Mesh, Hub-and-Spoke, ADVPN, overlays, Inter-Overlay Source NAT, device registration, licensing, and troubleshooting commands**.
>
> **Keywords:** FortiGate OCVPN, Overlay Controller VPN, FortiCloud VPN, FortiGate ADVPN, FortiGate IPsec, FortiCloud orchestration, FortiGate VPN troubleshooting, OCVPN configuration

---

## 📋 Table of Contents

- [OCVPN Fundamentals](#-1-ocvpn-fundamentals)
- [Prerequisites](#-2-prerequisites)
- [OCVPN Roles](#-3-ocvpn-roles)
- [Full-Mesh Deployment](#-4-full-mesh-deployment)
- [Hub-and-Spoke Deployment](#-5-hub-and-spoke-deployment)
- [ADVPN Integration](#-6-advpn-integration)
- [Overlay Configuration](#-7-overlay-configuration)
- [Inter-Overlay Source NAT](#-8-inter-overlay-source-nat)
- [Assign IP / Mode Config](#-9-assign-ip--mode-config)
- [OCVPN Verification](#-10-ocvpn-verification)
- [Troubleshooting Checklist](#-11-troubleshooting-checklist)
- [Common Failure Scenarios](#-12-common-failure-scenarios)
- [Command Reference](#-13-command-reference)
- [Exam & Interview Checklist](#-14-exam--interview-checklist)
- [Mental Model](#-15-mental-model)
- [Quick Reference](#-16-quick-reference)
- [Security & Design Notes](#-17-security--design-notes)
- [SheynShield Resources](#-sheynshield-resources)

---

# 🔐 1. OCVPN Fundamentals

## What is OCVPN?

- [ ] Understand that **OCVPN = Overlay Controller VPN**
- [ ] Understand that OCVPN provides FortiCloud-based VPN orchestration
- [ ] Understand that FortiCloud participates in topology/orchestration
- [ ] Understand that OCVPN automates significant portions of VPN topology management
- [ ] Understand that OCVPN is built around FortiGate IPsec connectivity
- [ ] Understand overlay-based connectivity
- [ ] Understand that configuration is largely wizard-driven

### Core Architecture

```text
                    FortiCloud
                        |
                OCVPN Orchestration
                        |
          +-------------+-------------+
          |                           |
       FortiGate                  FortiGate
          |                           |
       Overlay  =================  Overlay
          |                           |
        LAN                         LAN
````

### OCVPN Mental Model

```text
OCVPN
  |
  +-- FortiCloud
  |
  +-- Overlay
  |
  +-- IPsec
  |
  +-- Topology Automation
  |
  +-- Device Relationships
  |
  +-- ADVPN
```

---

# 🧩 2. Prerequisites

Before deploying OCVPN:

## FortiGate Requirements

* [ ] Verify FortiOS version
* [ ] Confirm the participating devices meet the required FortiOS version for the deployment
* [ ] Verify all devices are associated with the **same FortiCloud account**
* [ ] Verify the deployment uses the **root VDOM**
* [ ] Verify devices can communicate with required FortiCloud services
* [ ] Verify licensing/subscription status
* [ ] Verify WAN connectivity
* [ ] Verify IPsec prerequisites

### Version Compatibility

For the deployment described in this checklist:

```text
FortiOS
   |
   +-- 6.2+
```

> [!WARNING]
> Always verify the exact OCVPN feature matrix against the **FortiOS release and FortiGate model** before production deployment. Feature availability and limits can change between releases.

### FortiCloud

* [ ] FortiGate is registered
* [ ] FortiGate is associated with the correct FortiCloud account
* [ ] All participating devices belong to the same account
* [ ] FortiCloud service is available
* [ ] License/subscription status is valid

---

# 👥 3. OCVPN Roles

OCVPN deployments can use different logical roles.

| Role              | Purpose              |
| ----------------- | -------------------- |
| **Primary Hub**   | Main hub/controller  |
| **Secondary Hub** | Secondary/backup hub |
| **Spoke**         | Remote FortiGate     |

### Role Checklist

* [ ] Identify the Primary Hub
* [ ] Identify the Secondary Hub if required
* [ ] Identify all Spokes
* [ ] Document each device role
* [ ] Verify topology requirements
* [ ] Verify licensing supports the selected topology

---

# 🌐 4. Full-Mesh Deployment

## Full-Mesh Concept

```text
             FortiGate-1
                  |
             +----+----+
             |         |
             |         |
         FortiGate-2---FortiGate-3
```

In a full-mesh topology:

* [ ] Participating FortiGates are members of the OCVPN deployment
* [ ] Overlay relationships are correctly defined
* [ ] Required subnets are associated with the overlay
* [ ] FortiCloud account association is correct
* [ ] Device limits are respected

## Reported Capacity

### Free Service

| Item              |  Limit |
| ----------------- | -----: |
| Full-Mesh Devices |  **3** |
| Overlays          | **10** |
| Subnets / Overlay | **16** |

### Licensed Service

| Item              |  Limit |
| ----------------- | -----: |
| Full-Mesh Devices | **16** |
| Overlays          | **10** |
| Subnets / Overlay | **16** |

> [!IMPORTANT]
> Treat these values as **release/license-dependent limits**. Verify them against the FortiOS/FortiCloud documentation applicable to your deployment.

### Full-Mesh Checklist

* [ ] Maximum device count verified
* [ ] Maximum overlay count verified
* [ ] Maximum subnet count per overlay verified
* [ ] Same FortiCloud account confirmed
* [ ] Root VDOM confirmed
* [ ] FortiOS compatibility confirmed
* [ ] Overlay names verified
* [ ] Required subnets added
* [ ] IPsec connectivity verified

---

# 🏢 5. Hub-and-Spoke Deployment

## Topology

```text
                  Primary Hub
                 /     |     \
                /      |      \
           Spoke-1  Spoke-2  Spoke-3
                         |
                   Secondary Hub
```

### Hub-and-Spoke Checklist

* [ ] Primary Hub configured
* [ ] Secondary Hub configured if required
* [ ] Spokes registered
* [ ] Overlay configuration completed
* [ ] Hub relationships verified
* [ ] Spoke relationships verified
* [ ] ADVPN requirements verified
* [ ] Licensing verified

## Licensed Capacity

| Feature                 | Reported Limit |
| ----------------------- | -------------: |
| Hubs                    |          **2** |
| Spokes                  |       **1024** |
| Overlays                |         **10** |
| Subnets / Overlay       |         **64** |
| Spoke Overlays          |         **10** |
| Spoke Subnets / Overlay |         **16** |

> [!WARNING]
> Capacity values are **version/license dependent**. Confirm current limits before designing a production deployment.

## Free Service

```text
Hub-and-Spoke + ADVPN
          |
          +---- Not available in the described free-service model
```

---

# 🚀 6. ADVPN Integration

OCVPN can be used with ADVPN-style dynamic spoke-to-spoke connectivity.

### Traditional Hub-and-Spoke

```text
Spoke A
   |
   v
  HUB
   |
   v
Spoke B
```

### ADVPN Shortcut

```text
Spoke A
   |
   +====================+
                        |
                        v
                     Spoke B
```

### ADVPN Checklist

* [ ] Hub topology verified
* [ ] Spokes registered
* [ ] Overlay configuration verified
* [ ] ADVPN support/license verified
* [ ] Shortcut requirements verified
* [ ] Routing design verified
* [ ] Spoke-to-spoke connectivity tested
* [ ] NAT requirements reviewed

### Key Architecture

```text
                 FortiCloud
                     |
              +------+------+ 
              |             |
          Primary Hub   Secondary Hub
              |
        +-----+-----+
        |     |     |
      Spoke Spoke Spoke
        \     |     /
         \    |    /
          ADVPN
         Shortcut
```

> **Core idea:** OCVPN provides centralized orchestration while ADVPN can provide dynamic spoke-to-spoke connectivity.

---

# 🏷️ 7. Overlay Configuration

Overlay names are critical for correct topology/selector relationships.

## Overlay Name Checklist

* [ ] Overlay exists
* [ ] Overlay name is documented
* [ ] Local selector uses the correct overlay
* [ ] Remote selector uses the matching overlay
* [ ] Spelling is identical
* [ ] Case/format is verified where applicable

### Correct

```text
FGT-1
Overlay = Branch-Network

FGT-2
Overlay = Branch-Network
```

### Incorrect

```text
FGT-1
Overlay = Branch-Network

FGT-2
Overlay = Branch-Network-01
```

### Verification

```bash
diagnose vpn ocvpn show-overlays
```

* [ ] Run the command
* [ ] Verify expected overlays
* [ ] Verify overlay naming
* [ ] Compare local and remote configuration

> [!WARNING]
> A selector/overlay naming mismatch can prevent the intended VPN relationship from working correctly.

---

# 🔄 8. Inter-Overlay Source NAT

OCVPN can support communication between different overlays using **Source NAT** in the applicable deployment model.

### Concept

```text
Overlay-1
    |
  Spoke-1
    |
   IPsec
    |
   HUB
    |
   IPsec
    |
  Spoke-2
    |
Overlay-2
```

## Design Checklist

* [ ] Identify source overlay
* [ ] Identify destination overlay
* [ ] Verify hub connectivity
* [ ] Verify required routes
* [ ] Verify source NAT requirement
* [ ] Verify security policies
* [ ] Verify return path
* [ ] Disable Auto-Discovery when required by this design

### Important Relationship

```text
Inter-Overlay Source NAT
          |
          +-- Auto-Discovery
                 |
                 +-- Disable
```

> [!IMPORTANT]
> For the described **Inter-Overlay Source NAT** design, **Auto-Discovery should be disabled**.

---

# 📡 9. Assign IP / Mode Config

The **Assign IP** functionality can be conceptually compared with Mode Config in traditional IPsec deployments.

### Concept

```text
Spoke-1 --------\
                 \
                  HUB
                 /
Spoke-2 --------/
```

The hub can assign addresses from the configured address range.

### Checklist

* [ ] Address range defined
* [ ] Address allocation requirements documented
* [ ] Spoke receives expected address
* [ ] Address conflicts checked
* [ ] Routing supports assigned addresses
* [ ] Policies support assigned addresses
* [ ] Return traffic verified

### Mental Model

```text
Traditional IPsec
      |
 Mode Config
      |
 Dynamic Client Parameters


OCVPN
      |
 Assign IP
      |
 Dynamic Overlay Addressing
```

> [!NOTE]
> **Assign IP ≈ Mode Config** is a conceptual comparison, not necessarily an indication that the implementations are identical.

---

# 🔎 10. OCVPN Verification

## Check OCVPN Registration

```bash
diagnose vpn ocvpn list
```

Use this to verify:

* [ ] OCVPN registration
* [ ] Registered devices
* [ ] OCVPN statistics
* [ ] Service state

---

## Check OCVPN Metadata

```bash
diagnose vpn ocvpn show-meta
```

Verify:

* [ ] Service information
* [ ] License information
* [ ] Metadata
* [ ] Service state

---

## Check Overlays

```bash
diagnose vpn ocvpn show-overlays
```

Verify:

* [ ] Overlay names
* [ ] Overlay membership
* [ ] Expected overlays
* [ ] Configuration consistency

---

## Check Members

```bash
diagnose vpn ocvpn show-members
```

Verify:

* [ ] FortiGate members
* [ ] Serial numbers
* [ ] Assigned member numbers
* [ ] Assigned IP addresses
* [ ] Expected topology members

---

## Check IKE

```bash
diagnose vpn ike gateway list
```

Verify:

* [ ] IKE SA
* [ ] Peer address
* [ ] Authentication
* [ ] IKE version
* [ ] Proposal
* [ ] Tunnel state

---

# 🛠️ 11. Troubleshooting Checklist

## Step 1 — FortiCloud

* [ ] FortiGate is registered
* [ ] Correct FortiCloud account is used
* [ ] All devices use the same account
* [ ] Service is available
* [ ] License is valid

---

## Step 2 — FortiOS

* [ ] FortiOS version is supported
* [ ] Device model supports required OCVPN functionality
* [ ] Feature is available in the installed release
* [ ] Release-specific limitations checked

---

## Step 3 — VDOM

* [ ] Root VDOM is being used
* [ ] OCVPN configuration is not accidentally being performed in an unsupported VDOM context

---

## Step 4 — Registration

Run:

```bash
diagnose vpn ocvpn list
```

Check:

* [ ] Device registered
* [ ] OCVPN service operational
* [ ] Expected members visible

---

## Step 5 — Metadata / License

Run:

```bash
diagnose vpn ocvpn show-meta
```

Check:

* [ ] License
* [ ] Service status
* [ ] Metadata
* [ ] Feature availability

---

## Step 6 — Overlay

Run:

```bash
diagnose vpn ocvpn show-overlays
```

Check:

* [ ] Overlay exists
* [ ] Overlay name matches
* [ ] Correct members are associated
* [ ] Expected subnet information exists

---

## Step 7 — Members

Run:

```bash
diagnose vpn ocvpn show-members
```

Check:

* [ ] Expected FortiGate exists
* [ ] Serial number is correct
* [ ] Assigned IP is correct
* [ ] Member information is consistent

---

## Step 8 — IKE

Run:

```bash
diagnose vpn ike gateway list
```

Check:

* [ ] IKE SA established
* [ ] Peer reachable
* [ ] Correct proposal
* [ ] Correct authentication
* [ ] Expected tunnel state

---

## Step 9 — IPsec

Run:

```bash
diagnose vpn tunnel list
```

Check:

* [ ] Phase 2 SA
* [ ] SPI
* [ ] Encryption
* [ ] Authentication
* [ ] Traffic counters
* [ ] Selectors

---

## Step 10 — Routing

* [ ] Local routes correct
* [ ] Remote routes correct
* [ ] Return path correct
* [ ] No overlapping networks
* [ ] SD-WAN rules reviewed where applicable
* [ ] ADVPN routing reviewed where applicable

---

## Step 11 — Firewall Policy

* [ ] Source interface correct
* [ ] Destination interface correct
* [ ] Source address correct
* [ ] Destination address correct
* [ ] Service correct
* [ ] NAT behavior correct
* [ ] Policy order correct
* [ ] Return traffic allowed

---

# 🚨 12. Common Failure Scenarios

| Symptom                     | First Checks                           |
| --------------------------- | -------------------------------------- |
| OCVPN device not registered | FortiCloud account / service / license |
| Overlay missing             | `show-overlays`                        |
| Wrong member information    | `show-members`                         |
| IKE down                    | `diagnose vpn ike gateway list`        |
| Phase 2 down                | `diagnose vpn tunnel list`             |
| Spoke-to-spoke unavailable  | ADVPN / routing / shortcut             |
| Inter-overlay traffic fails | Source NAT / routing / policies        |
| Unexpected topology         | OCVPN role / overlay / registration    |
| Assigned IP unavailable     | Address range / member / configuration |
| Service unavailable         | FortiCloud / license / release support |

---

# 🔬 13. Command Reference

## OCVPN

```bash
# OCVPN registration / status
diagnose vpn ocvpn list
```

```bash
# OCVPN metadata / license
diagnose vpn ocvpn show-meta
```

```bash
# OCVPN overlays
diagnose vpn ocvpn show-overlays
```

```bash
# OCVPN members
diagnose vpn ocvpn show-members
```

## IKE

```bash
diagnose vpn ike gateway list
```

## IPsec

```bash
diagnose vpn tunnel list
```

### Command Matrix

| Command                            | Purpose                   |
| ---------------------------------- | ------------------------- |
| `diagnose vpn ocvpn list`          | OCVPN registration/status |
| `diagnose vpn ocvpn show-meta`     | Metadata/service/license  |
| `diagnose vpn ocvpn show-overlays` | Overlay information       |
| `diagnose vpn ocvpn show-members`  | OCVPN members             |
| `diagnose vpn ike gateway list`    | IKE gateway/SA            |
| `diagnose vpn tunnel list`         | IPsec tunnel/Phase 2      |

---

# 🧠 14. Exam & Interview Checklist

## OCVPN Fundamentals

* [ ] What does OCVPN stand for?
* [ ] Where is OCVPN orchestration hosted?
* [ ] What is an overlay?
* [ ] What FortiCloud relationship is required?
* [ ] Why is the root VDOM important?
* [ ] What FortiOS versions support the described deployment?

## Full-Mesh

* [ ] What is a full-mesh topology?
* [ ] What are the reported free-service device limits?
* [ ] What are the reported licensed limits?
* [ ] How are overlays used?

## Hub-and-Spoke

* [ ] What is the role of the Primary Hub?
* [ ] What is the role of the Secondary Hub?
* [ ] What is the role of a Spoke?
* [ ] What are the reported spoke limits?
* [ ] What is the relationship between OCVPN and ADVPN?

## Overlay

* [ ] Why must overlay names match?
* [ ] How do you verify overlay information?
* [ ] What command displays OCVPN overlays?

## Inter-Overlay NAT

* [ ] What is Inter-Overlay Source NAT?
* [ ] Why would Source NAT be required?
* [ ] What happens to Auto-Discovery in this design?

## Troubleshooting

* [ ] How do you verify OCVPN registration?
* [ ] How do you verify the OCVPN license/service?
* [ ] How do you verify overlay configuration?
* [ ] How do you verify members?
* [ ] How do you verify IKE?
* [ ] How do you verify Phase 2/IPsec?

---

# ⚡ 15. Mental Model

```text
                         FORTICLOUD
                             |
                     OCVPN ORCHESTRATION
                             |
              +--------------+--------------+
              |                             |
          FULL-MESH                    HUB / SPOKE
              |                             |
        +-----+-----+                 +-----+-----+
        |     |     |                 |           |
       FGT   FGT   FGT               HUB         HUB-2
                                          |
                                  +-------+-------+
                                  |       |       |
                                Spoke   Spoke   Spoke
```

### OCVPN Stack

```text
FortiCloud
    ↓
OCVPN
    ↓
Overlay
    ↓
IPsec
    ↓
Topology
    ↓
Routing
    ↓
Application Traffic
```

---

# 🔥 16. Quick Reference

```text
OCVPN
  ↓
Overlay Controller VPN
  ↓
FortiCloud-based orchestration
```

```text
FULL-MESH
  ↓
Device-to-device overlay
```

```text
HUB-AND-SPOKE
  ↓
Primary Hub
  ↓
Secondary Hub
  ↓
Spokes
```

```text
ADVPN
  ↓
Dynamic Spoke-to-Spoke
  ↓
Shortcut
```

```text
INTER-OVERLAY
  ↓
Source NAT
  ↓
Auto-Discovery disabled
```

```text
ASSIGN IP
  ↓
Dynamic IP assignment
  ↓
Conceptually similar to Mode Config
```

---

# 📊 17. Deployment Decision Matrix

| Requirement                    | Recommended OCVPN Design      |
| ------------------------------ | ----------------------------- |
| Small full-mesh deployment     | Full-Mesh                     |
| Multiple branch sites          | Hub-and-Spoke                 |
| Dynamic spoke-to-spoke traffic | Hub-and-Spoke + ADVPN         |
| Centralized orchestration      | OCVPN                         |
| Multiple logical VPN overlays  | Overlay-based design          |
| Communication between overlays | Inter-Overlay Source NAT      |
| Dynamic address assignment     | Assign IP                     |
| Central troubleshooting        | OCVPN + IKE/IPsec diagnostics |

---

# 🛡️ 18. Security & Design Notes

* [ ] Verify FortiOS release compatibility before deployment
* [ ] Verify current OCVPN licensing limits
* [ ] Verify FortiGate model support
* [ ] Use strong IPsec cryptographic settings where manually configurable
* [ ] Avoid legacy cryptographic algorithms
* [ ] Document FortiCloud account ownership
* [ ] Document all OCVPN members
* [ ] Document overlay names
* [ ] Document subnet assignments
* [ ] Document hub/spoke roles
* [ ] Validate routing before production
* [ ] Validate return traffic
* [ ] Review NAT behavior carefully
* [ ] Test spoke-to-spoke connectivity
* [ ] Test hub failure if a secondary hub is deployed
* [ ] Test device registration recovery
* [ ] Test tunnel recovery
* [ ] Keep troubleshooting commands documented

> [!WARNING]
> Do not design a production OCVPN deployment solely from a memorized feature limit. **FortiOS release, FortiGate model, FortiCloud service, and licensing can affect supported functionality.**

---

# 🧪 19. Production Validation Checklist

Before declaring the deployment ready:

### Cloud

* [ ] FortiCloud account verified
* [ ] Devices registered
* [ ] License verified
* [ ] OCVPN service verified

### FortiGate

* [ ] FortiOS compatibility verified
* [ ] Root VDOM verified
* [ ] WAN connectivity verified
* [ ] Required IPsec connectivity verified

### Topology

* [ ] Hub/spoke roles verified
* [ ] Overlay names verified
* [ ] Members verified
* [ ] Subnets verified
* [ ] Device limits verified

### Routing

* [ ] Local routing verified
* [ ] Remote routing verified
* [ ] Return path verified
* [ ] ADVPN routing verified where applicable

### Security

* [ ] Firewall policies verified
* [ ] NAT verified
* [ ] IPsec proposals verified
* [ ] Authentication verified

### Failure Testing

* [ ] Primary path failure tested
* [ ] Secondary path tested
* [ ] Hub failure tested where applicable
* [ ] Spoke failure tested
* [ ] Tunnel recovery tested
* [ ] Spoke-to-spoke recovery tested

---

# 🎯 20. Final OCVPN Checklist

```text
                    OCVPN
                      |
        +-------------+-------------+
        |             |             |
   FortiCloud      Overlay       IPsec
        |             |             |
   Registration   Naming       IKE / SA
        |             |             |
     License       Subnets      Tunnel
        |             |             |
        +-------------+-------------+
                      |
                 Topology
                      |
          +-----------+-----------+
          |                       |
       Full-Mesh            Hub-and-Spoke
                                  |
                                ADVPN
                                  |
                           Spoke-to-Spoke
```

### Final Verification

* [ ] FortiCloud account correct
* [ ] License correct
* [ ] FortiOS support verified
* [ ] Root VDOM verified
* [ ] Device registration verified
* [ ] OCVPN members verified
* [ ] Overlay names verified
* [ ] Overlay subnets verified
* [ ] Hub/spoke roles verified
* [ ] ADVPN verified where required
* [ ] Inter-overlay NAT verified where required
* [ ] Auto-Discovery disabled for the described inter-overlay NAT design
* [ ] IKE SA verified
* [ ] IPsec SA verified
* [ ] Routing verified
* [ ] Firewall policies verified
* [ ] Production failure testing completed

---

# 📌 OCVPN One-Minute Cheat Sheet

| Topic                                  | Remember                                                        |
| -------------------------------------- | --------------------------------------------------------------- |
| **OCVPN**                              | FortiCloud-based Overlay Controller VPN                         |
| **Orchestration**                      | FortiCloud                                                      |
| **Core VPN**                           | IPsec                                                           |
| **Topology**                           | Full-Mesh / Hub-and-Spoke                                       |
| **Primary Hub**                        | Main hub                                                        |
| **Secondary Hub**                      | Secondary hub                                                   |
| **Spoke**                              | Remote FortiGate                                                |
| **ADVPN**                              | Dynamic spoke-to-spoke connectivity                             |
| **Overlay**                            | Logical VPN topology/segmentation                               |
| **Assign IP**                          | Dynamic address assignment; conceptually similar to Mode Config |
| **Inter-Overlay NAT**                  | Source NAT between overlays                                     |
| **Inter-Overlay NAT + Auto-Discovery** | Disable Auto-Discovery for this design                          |
| **Registration**                       | `diagnose vpn ocvpn list`                                       |
| **Metadata**                           | `diagnose vpn ocvpn show-meta`                                  |
| **Overlays**                           | `diagnose vpn ocvpn show-overlays`                              |
| **Members**                            | `diagnose vpn ocvpn show-members`                               |
| **IKE**                                | `diagnose vpn ike gateway list`                                 |
| **IPsec**                              | `diagnose vpn tunnel list`                                      |
| **Root VDOM**                          | Required for the described deployment                           |
| **FortiCloud Account**                 | Participating devices use the same account                      |

---

# 🔗 SheynShield Resources

### 🎥 Video Learning

* [ ] [YouTube — SheynShield](https://youtube.com/@sheynshield)

  * Fortinet NSE content
  * FortiGate troubleshooting
  * Network Security Engineering

### 📚 Notes & Updates

* [ ] [Telegram — SheynShield](https://t.me/sheynshield)

### 💼 Professional Network

* [ ] [LinkedIn — Shayan-heydarikhah](https://linkedin.com/in/shayan-heydarikhah)

### 🐙 Technical Knowledge Base

* [ ] [SheynShield GitHub](https://github.com/Shayan-heydarikhah/sheynshield)

---

## 🏷️ Topics

```text
fortigate
fortinet
ocvpn
overlay-controller-vpn
ipsec
vpn
advpn
forticloud
network-security
cybersecurity
sdwan
network-engineering
fortios
vpn-troubleshooting
fortigate-cheatsheet
fortinet-cheatsheet
```

## 🔎 Keywords

```text
FortiGate OCVPN
FortiGate OCVPN configuration
FortiGate Overlay Controller VPN
FortiGate OCVPN troubleshooting
FortiCloud OCVPN
FortiGate ADVPN
FortiGate Hub and Spoke VPN
FortiGate Full Mesh VPN
FortiGate IPsec troubleshooting
FortiGate OCVPN commands
diagnose vpn ocvpn list
diagnose vpn ocvpn show-overlays
diagnose vpn ocvpn show-members
diagnose vpn ocvpn show-meta
FortiGate VPN cheat sheet
Fortinet VPN checklist
```

---

> **SheynShield | Engineering Secure Networks**
>
> **Learn → Configure → Verify → Troubleshoot → Engineer**


# FortiGate OCVPN (Overlay Controller VPN) 

> **OCVPN = Overlay Controller VPN**
>
> FortiCloud-based overlay VPN orchestration for FortiGate devices.

---

## 1. OCVPN Overview

### Core Concept

- OCVPN services are hosted on **FortiCloud**.
- Most configuration and Phase 1/Phase 2 parameters are created through the **wizard**.
- FortiCloud account status determines access to the OCVPN service.
- OCVPN can automatically manage:
  - IPsec-related configuration
  - Overlay topology
  - Peer relationships
  - Topology changes
  - Self-learning / indexing of topology information

### Overlay Concept

An **overlay** connects multiple attached subnets/networks through the OCVPN VPN topology.

```text
                FortiCloud
                    |
          +---------+---------+
          |                   |
       FortiGate            FortiGate
          |                   |
       Overlay ------------ Overlay
          |                   |
       LAN/Subnet          LAN/Subnet
````

---

# 2. OCVPN Roles

OCVPN supports different device roles:

| Role          | Description                          |
| ------------- | ------------------------------------ |
| Primary Hub   | Main hub/controller in the topology  |
| Secondary Hub | Backup/secondary hub                 |
| Spoke         | Remote FortiGate connected to hub(s) |

---

# 3. Full-Mesh OCVPN

## Free Service

| Feature             |         Limit |
| ------------------- | ------------: |
| Full-mesh devices   | **3 devices** |
| Overlays            |        **10** |
| Subnets per overlay |        **16** |

## Full License

| Feature             |          Limit |
| ------------------- | -------------: |
| Full-mesh devices   | **16 devices** |
| Overlays            |         **10** |
| Subnets per overlay |         **16** |

---

## Requirements

All FortiGates must:

* Run **FortiOS 6.2 or later**
* Be joined to the **same FortiCloud account**
* Use the **root VDOM**

### Important

> FortiOS 6.0 cannot participate with FortiOS 6.2 in this OCVPN scenario.

---

# 4. Full-Mesh Topology

```text
             +----------------+
             |   FortiGate-1  |
             |      HUB       |
             +-------+--------+
                    / \
                   /   \
                  /     \
                 /       \
+---------------+         +---------------+
| FortiGate-2   |---------| FortiGate-3   |
|    Spoke      |         |    Spoke      |
+---------------+         +---------------+
```

### Characteristics

* Each device can establish overlay connectivity with other participating devices.
* OCVPN automatically handles much of the IPsec topology.
* Topology changes can be propagated through FortiCloud.

---

# 5. Overlay Naming

> ⚠️ **Overlay names must match between local and remote selectors.**

Example:

```text
FGT-1
  Overlay: Branch-Network

FGT-2
  Overlay: Branch-Network
```

### Correct

```text
Local selector  = Branch-Network
Remote selector = Branch-Network
```

### Incorrect

```text
Local selector  = Branch-Network
Remote selector = Branch-Network-01
```

---

# 6. Hub-and-Spoke OCVPN

OCVPN can operate in a **Hub-and-Spoke** topology with **ADVPN** behavior.

```text
                 +----------------+
                 |    Primary     |
                 |      Hub       |
                 +-------+--------+
                    /    |    \
                   /     |     \
                  /      |      \
                 /       |       \
             Spoke-1  Spoke-2  Spoke-3
```

---

## Free Service

```text
Hub-and-Spoke + ADVPN
        |
        +---- NOT SUPPORTED
```

---

## Full License

| Feature                 |    Limit |
| ----------------------- | -------: |
| Hubs                    |    **2** |
| Overlays                |   **10** |
| Subnets / overlay       |   **64** |
| Spokes                  | **1024** |
| Spoke overlays          |   **10** |
| Spoke subnets / overlay |   **16** |

### Roles

```text
Primary Hub
     |
     +------ Secondary Hub
     |
     +------ Spoke
     |
     +------ Spoke
     |
     +------ Spoke
```

### Requirements

* FortiOS **6.2 or later**
* All FortiGates joined to the **same FortiCloud account**
* Use **root VDOM**
* FortiOS 6.0 and 6.2 combination is not supported

---

# 7. OCVPN + ADVPN Concept

```text
                 FortiCloud
                     |
              +------+------+
              |             |
         Primary Hub    Secondary Hub
              |
       +------+------+
       |      |      |
    Spoke-1 Spoke-2 Spoke-3
       \      |      /
        \     |     /
         \    |    /
          ADVPN
        Shortcuts
```

### Key Idea

OCVPN provides centralized topology/orchestration while ADVPN can provide dynamic spoke-to-spoke connectivity.

---

# 8. Hub-and-Spoke with Inter-Overlay Source NAT

OCVPN can provide communication between different overlays using **Source NAT**.

### Concept

```text
Overlay-1
   |
 Spoke-1
   |
   | IPsec
   |
  HUB
   |
   | IPsec
   |
 Spoke-2
   |
Overlay-2
```

Communication between different overlays can be handled through **source NAT**.

---

## Assign IP

The **Assign IP** option can work similarly to **Mode Config** in traditional IPsec.

```text
Spoke-1 ----\
             \
              HUB
             /
Spoke-2 ----/
```

Example:

```text
Spoke-1 -> assigned overlay IP
Spoke-2 -> assigned overlay IP
```

If multiple spokes connect to the hub, they can receive addresses from the configured address range.

---

## Important

> ⚠️ When using **Inter-Overlay Source NAT**, disable **Auto-Discovery**.

```text
Auto-Discovery
      |
      +---- DISABLE
```

---

# 9. OCVPN Troubleshooting

## Check OCVPN / IPsec Gateway

```bash
diagnose vpn ike gateway list
```

### Useful for

* Connected clients
* IKE gateway information
* Tunnel status
* Connection statistics

---

## Check OCVPN Registration

```bash
diagnose vpn ocvpn list
```

### Shows

* Registration status
* OCVPN statistics
* Registered devices

---

## Check OCVPN Metadata / License

```bash
diagnose vpn ocvpn show-meta
```

### Useful for

* License information
* OCVPN service information
* Service statistics

---

## Check Overlay Names

```bash
diagnose vpn ocvpn show-overlays
```

### Shows

* Overlay names
* Configured overlays
* Overlay information

---

## Check OCVPN Members

```bash
diagnose vpn ocvpn show-members
```

### Shows

* FortiGate serial numbers
* Assigned numbers
* Assigned IP addresses
* OCVPN members

---

# 10. OCVPN Troubleshooting Flow

```text
                 OCVPN Problem
                       |
                       v
             Check FortiCloud Account
                       |
                       v
               Check License
                       |
                       v
             Check Device Registration
                       |
                       v
          diagnose vpn ocvpn list
                       |
                       v
             Check Overlay Names
                       |
                       v
       diagnose vpn ocvpn show-overlays
                       |
                       v
              Check Members
                       |
                       v
       diagnose vpn ocvpn show-members
                       |
                       v
             Check IKE Gateway
                       |
                       v
        diagnose vpn ike gateway list
```

---

# 11. Quick Reference

| Requirement                           |     Full-Mesh |        Hub-and-Spoke |
| ------------------------------------- | ------------: | -------------------: |
| FortiCloud                            |             ✅ |                    ✅ |
| FortiOS                               |          6.2+ |                 6.2+ |
| Root VDOM                             |             ✅ |                    ✅ |
| Same FortiCloud account               |             ✅ |                    ✅ |
| Free service                          |     3 devices |              ❌ ADVPN |
| Full license                          |    16 devices | 2 hubs / 1024 spokes |
| Overlays                              |            10 |                   10 |
| Primary Hub                           | Optional/role |                    ✅ |
| Secondary Hub                         | Optional/role |                    ✅ |
| Spoke                                 |             ✅ |                    ✅ |
| ADVPN                                 |             — |                    ✅ |
| Inter-overlay Source NAT              |             — |                    ✅ |
| Auto-Discovery with Inter-overlay NAT |             — |                    ❌ |

---

# 12. Important Exam / Troubleshooting Notes

> ### 🧠 Remember

* **OCVPN = FortiCloud-based VPN orchestration**
* OCVPN configuration is largely **wizard-based**.
* Devices must be associated with the **same FortiCloud account**.
* Use **root VDOM**.
* **FortiOS 6.2+** is required for the described OCVPN deployment.
* **Overlay names must match** between corresponding local/remote selectors.
* **Full-mesh free license = 3 devices**.
* **Full-mesh licensed = up to 16 devices**.
* **Hub-and-spoke ADVPN is not available in the free service**.
* Licensed hub-and-spoke supports:

  * **2 hubs**
  * **1024 spokes**
* **Assign IP ≈ Mode Config concept**.
* For **inter-overlay source NAT**, disable **Auto-Discovery**.
* Use `diagnose vpn ocvpn list` to check registration.
* Use `diagnose vpn ocvpn show-overlays` to verify overlay configuration.
* Use `diagnose vpn ocvpn show-members` to verify members.
* Use `diagnose vpn ocvpn show-meta` to check service/license information.
* Use `diagnose vpn ike gateway list` to troubleshoot IKE connectivity.

---

# 13. Command  

```bash
# IKE / VPN Gateway
diagnose vpn ike gateway list

# OCVPN status / registration
diagnose vpn ocvpn list

# OCVPN metadata / license
diagnose vpn ocvpn show-meta

# OCVPN overlays
diagnose vpn ocvpn show-overlays

# OCVPN members
diagnose vpn ocvpn show-members
```

---

# 14. Mental Model

```text
                    FORTICLOUD
                        |
              OCVPN Orchestration
                        |
        +---------------+---------------+
        |                               |
    Full Mesh                     Hub / Spoke
        |                               |
   +----+----+                    +-----+-----+
   |    |    |                    |           |
  FGT  FGT  FGT                  HUB       HUB-2
                                   |
                              +----+----+
                              |    |    |
                            Spoke Spoke Spoke
```

### OCVPN =

```text
FortiCloud
    +
Overlay
    +
IPsec
    +
Topology Automation
    +
ADVPN
    +
Centralized Orchestration
```

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

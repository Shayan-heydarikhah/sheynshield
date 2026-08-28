# FortiOS Dial-up IPsec & OSPF ADVPN Cheat Sheet

A quick-reference guide for configuring Dial-up IPsec, ADVPN, and OSPF over multi-FortiGate topologies (Hub & Spoke with NAT traversal).

---

## 1. Prerequisites
- **Active Directory:** Add AD as an LDAP server on the FortiGate Hub (FGT-1).
- **FSSO:** Configure AD FSSO under `Security Fabric > External Connectors`.

---

## 2. IPsec Phase 1 & 2 Matrix

| Parameter | FGT-1 (Hub / HQ) | FGT-2 (Spoke - Static) | FGT-4 (Spoke - Behind NAT FGT-3) |
| :--- | :--- | :--- | :--- |
| **Gateway Type** | Dialup User | Static IP of FGT-1 | Static IP of FGT-1 |
| **IKE & Mode**| IKEv1, Aggressive | IKEv1, Aggressive | IKEv1, Aggressive |
| **Phase 1 Crypto** | DES, MD5, DH 5 | DES, MD5, DH 5 | DES, MD5, DH 5 |
| **Phase 2 Crypto** | DES, MD5, PFS DH 5, Auto-negotiate | DES, MD5, PFS DH 5, Auto-negotiate | DES, MD5, PFS DH 5, Auto-negotiate |
| **XAuth Setup** | Server: Auto (AD/FSSO users) | Client: `u1` / `1qaz@WSX`| Client: `u2` / `1qaz@WSX`|
| **Peer ID** | Accept Any Peer ID | Specific ID (FGT-1) or Any | Specific ID (FGT-1) or Any |
| **IPsec Advanced** | - Enable: Device Creation<br>- Enable: Auto-Discovery (Sender/Receiver)<br>- Disable: Add Route | - Enable: Device Creation<br>- Enable: Add Route<br>*(Auto-Discovery optional)* | - Enable: Device Creation<br>- Enable: Auto-Discovery (Sender/Receiver)<br>- Disable: Add Route |

---

## 3. Global Firewall Policies (All Nodes)

**Traffic Flow Rules:**
* **Incoming:** `Dialup/IPsec Interface` + `ISP Link` & `LAN`
* **Outgoing:** `Dialup/IPsec Interface` + `ISP Link` & `LAN`
* **Service / Src / Dst:** `ALL`
* **Logging:** `Log all sessions`
* **NAT:** **Disabled** (Required for OSPF routing overlay). 
  > *Exception:* FGT-3 requires NAT enabled from FGT-4 to ISP. Disable NAT from ISP to FGT-4 for troubleshooting.

---

## 4. OSPF Routing & Interfaces

### FGT-1 (Hub)
```text
Interface IP: 12.23.34.1
Remote IP: 12.23.34.2/24 (Allow Ping)

OSPF Router ID: 1.1.1.1
OSPF Area 0.0.0.0:
  - 192.168.101.0/24
  - 12.23.34.0/24
Options: Inject default route always

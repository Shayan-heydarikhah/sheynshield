```markdown
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

```

### FGT-2 (Spoke)

```text
Interface IP: 12.23.34.2
Remote IP: 12.23.34.1/24 (Allow Ping)

OSPF Router ID: 2.2.2.2
OSPF Area 0.0.0.0:
  - 192.168.102.0/24
  - 12.23.34.0/24

```

### FGT-4 (Spoke behind NAT)

```text
Interface IP: 12.23.34.4
Remote IP: 12.23.34.1/24 (Allow Ping)

OSPF Router ID: 4.4.4.4
OSPF Area 0.0.0.0:
  - 192.168.104.0/24
  - 12.23.34.0/24

```

---

## 5. Mode Config (DHCP over IPsec)

To assign IPs dynamically over the Dialup IPsec tunnel:

1. **FGT-1 Network Settings:** In Custom IPsec VPN, enable Mode Config (`Use System DNS`, `Assign IP`).
2. **FGT-1 Interface:**
* Subnet Mask: `255.255.255.0`
* Local IP: `12.23.34.1` | Remote IP: `12.23.34.254/24`
* Enable DHCP Server on the IPsec interface.


3. **Advanced Setup:**
* **FGT-1:** Disable all advanced components except `Device Creation` and `Add Route`.
* **FGT-2 (Client):** Ensure `Mode Config` is enabled.



---

## 6. ADVPN & OSPF Troubleshooting

* **OSPF Stuck in INIT:** Verify **Auto-Discovery Sender and Receiver** are enabled on FGT-1 Phase 1 advanced settings to advertise the OSPF mesh properly.
* **Cisco Interoperability:** If spoke subnets route via public IPs instead of tunnels, change the FortiGate OSPF Interface Network Type to **Point-to-Point** or **Broadcast / Point-to-Multipoint**.
* **Spoke-to-Spoke Traffic:** Enable **Exchange IP Address** on FGT-1 to mimic Cisco DMVPN behavior. *(Dialup user groups alone do not trigger this).*

```

```

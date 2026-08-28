# 🛡️ FortiOS Dial-up IPsec & eBGP Cheat Sheet

A comprehensive reference guide for deploying Dial-up IPsec combined with eBGP routing across a Hub-and-Spoke Fortinet topology. This guide includes configurations for standard spokes and spokes situated behind NAT devices.

---

## 1. 🔑 Identity & Authentication Prerequisites (FGT-1 Hub)

Before establishing the VPN overlay, configure identity integration on the Hub to validate dial-up users.

*   **LDAP Server:** Add the Active Directory server under `User & Authentication > LDAP Servers`.
*   **FSSO Connector:** Configure AD FSSO under `Security Fabric > External Connectors` to map IP addresses dynamically.
*   **User Groups:** Create local FortiGate Firewall User Groups mapped to the AD/FSSO groups to be called in IPsec XAuth.

---

## 2. 🌐 IPsec Phase 1 & 2 Deployment Matrix

| Feature / Setting | **FGT-1 (Hub / HQ)** | **FGT-2 (Standard Spoke)** | **FGT-4 (Spoke Behind NAT FGT-3)** |
| :--- | :--- | :--- | :--- |
| **Gateway Type** | Dialup User | Static IP (Points to FGT-1) | Static IP (Points to FGT-1) |
| **IKE Version & Mode**| IKEv1 / Aggressive Mode | IKEv1 / Aggressive Mode | IKEv1 / Aggressive Mode |
| **Phase 1 Crypto** | DES, MD5, DH 5 | DES, MD5, DH 5 | DES, MD5, DH 5 |
| **Phase 2 Crypto** | DES, MD5, PFS DH 5, Auto-negotiate | DES, MD5, PFS DH 5, Auto-negotiate | DES, MD5, PFS DH 5, Auto-negotiate |
| **XAuth Setup** | Server Type: Auto (Calls AD/FSSO) | Client: `u1` / `1qaz@WSX` | Client: `u2` / `1qaz@WSX` |
| **Peer ID** | Accept Any Peer ID | Specific ID (FGT-1) or Any | Specific ID (FGT-1) or Any |
| **Advanced Routing** | **Enable:** Device Creation<br>**Disable:** Add Route<br>**Enable:** Auto-Discovery | **Enable:** Device Creation<br>**Enable:** Add Route | **Enable:** Device Creation<br>**Disable:** Add Route<br>**Enable:** Auto-Discovery |

> **⚠️ Hub-and-Spoke Scaling Note for FGT-1:** If only *one* device connects via the tunnel, `Add Route` can remain enabled. However, for multiple connections sharing the same tunnel interface, you **must** disable `Add Route` and rely on BGP routing combined with `Device Creation` and `Auto-discovery`.

---

## 3. 🛡️ Global Firewall Policies (All Nodes)

Apply these baseline policies across **all** participating FortiGates (FGT-1, FGT-2, FGT-4).

*   **Incoming Policy:** `Dialup / IPsec Interface` ➔ `ISP Link` & `LAN`
*   **Outgoing Policy:** `Dialup / IPsec Interface` ➔ `ISP Link` & `LAN`
*   **Service / Source / Destination:** `ALL` *(Ensure AD/LDAP services are permitted if branches need to forward requests to HQ).*
*   **NAT:** **STRICTLY DISABLED**. NAT must be off to allow eBGP overlay routing.
*   **Logging:** `Log all sessions`.

---

## 4. 🔀 eBGP Dynamic Routing & Overlay Interfaces

### 🏢 FGT-1 (Hub / HQ)
**Overlay Interface:**
*   **IP / Remote:** `12.23.34.1` / `12.23.34.2/24`
*   **Access:** Allow `ping`

**BGP Configuration:**
```text
Router ID: 1.1.1.1
Local AS: 65001
Network: 192.168.101.0/24 (HQ LAN)

Neighbor:
  - IP: 12.23.34.2
  - Remote-AS: 65002
  - Option: next-hop-self

```

> **🛑 Critical ADVPN / BGP Architecture Note:** On this Dialup model, if `exchange interfaces` is enabled, the setup will fail. Fortinet recommends using traditional Hub-and-Spoke (with iBGP) or Site-to-Site IPsec VPN designs if full ADVPN shortcut capability over BGP is required.

### 🏢 FGT-2 (Standard Spoke)

**Overlay Interface:**

* **IP / Remote:** `12.23.34.2` / `12.23.34.1/24`
* **Access:** Allow `ping`

**BGP Configuration:**

```text
Router ID: 2.2.2.2
Local AS: 65002
Network: 192.168.102.0/24 (Spoke LAN)

Neighbor:
  - IP: 12.23.34.1
  - Remote-AS: 65001
  - Option: next-hop-self

```

### 🏢 FGT-4 (Spoke Behind NAT via FGT-3)

*Topology:* `FGT-1 (Hub) --- ISP Router --- FGT-3 (NAT Router) --- FGT-4 (Spoke)`

**Overlay Interface:**

* **IP / Remote:** `12.23.34.4` / `12.23.34.1/24`
* **Access:** Allow `ping`

**BGP Configuration:**

```text
Router ID: 4.4.4.4
Local AS: 65004
Network: 192.168.104.0/24 (Spoke LAN)

Neighbor:
  - IP: 12.23.34.1
  - Remote-AS: 65001
  - Option: next-hop-self

```

---

## 5. 🛠️ FGT-3 (NAT Router) Specific Configuration

Because FGT-4 is situated behind FGT-3, the NAT handling on FGT-3 must be strictly controlled for UDP 500/4500 packets to traverse properly.

* **FGT-4 ➔ ISP (Outbound Policy):** `Enable NAT` (Allows the spoke to reach the internet and build the tunnel to FGT-1).
* **ISP ➔ FGT-4 (Inbound Policy):** `Disable NAT` (Crucial for testing and isolating phase 1/2 encapsulation issues).

```

```

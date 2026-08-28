```markdown
```
# Comprehensive FortiOS Dial-up IPsec, ADVPN & OSPF Cheat Sheet

An advanced, highly detailed reference guide for deploying Dial-up IPsec, ADVPN (Auto-Discovery VPN), and OSPF dynamic routing over multi-FortiGate Hub-and-Spoke topologies, including NAT traversal scenarios.

---

## 1. Identity & Fabric Prerequisites

- **Active Directory (LDAP):** Define AD as an LDAP server on the FortiGate Hub (FGT-1) under `User & Authentication > LDAP Servers`.
- **FSSO Integration:** Configure AD FSSO under `Security Fabric > External Connectors` to map IP addresses to AD user groups dynamically.
- **User Groups:** Create local Firewall User Groups mapping to the LDAP/FSSO groups to be used in the IPsec XAuth configuration.

---

## 2. IPsec Phase 1 & 2 Matrix (Hub & Spoke)

| Parameter | FGT-1 (Hub / HQ) | FGT-2 (Spoke - Static IP) | FGT-4 (Spoke - Behind NAT FGT-3) |
| :--- | :--- | :--- | :--- |
| **Gateway Type** | Dialup User | Static IP (FGT-1 Public IP) | Static IP (FGT-1 Public IP) |
| **IKE Version/Mode** | IKEv1, Aggressive Mode | IKEv1, Aggressive Mode | IKEv1, Aggressive Mode |
| **Phase 1 Crypto** | DES, MD5, DH 5 (Consider AES/SHA256 for Prod) | DES, MD5, DH 5 | DES, MD5, DH 5 |
| **Phase 2 Crypto** | DES, MD5, PFS DH 5, Auto-negotiate | DES, MD5, PFS DH 5, Auto-negotiate | DES, MD5, PFS DH 5, Auto-negotiate |
| **DPD (Dead Peer)** | On-Idle (Saves Hub resources) | On-Demand | On-Demand |
| **XAuth Setup** | Server: Auto (Validates AD/FSSO) | Client: `u1` / `1qaz@WSX` | Client: `u2` / `1qaz@WSX` |
| **Peer ID** | Accept Any Peer ID | Specific ID (FGT-1) or Any | Specific ID (FGT-1) or Any |
| **ADVPN & Routing** | Enable: Device Creation<br>Enable: Auto-Discovery Sender & Receiver<br>Disable: Add Route | Enable: Device Creation<br>Enable: Add Route<br>*(Auto-Discovery optional)* | Enable: Device Creation<br>Enable: Auto-Discovery Sender & Receiver<br>Disable: Add Route |

> **Pro Tip:** For ADVPN to work over IPsec, ensure `network-id` (e.g., `set network-id 1`) is identically configured on Phase 1 of all participating FortiGates via CLI.

---

## 3. Global Firewall Policies (All Nodes)

**Traffic Flow Rules:**
* **Incoming Policy:** `Dialup/IPsec Interface` + `ISP Link` & `LAN`
* **Outgoing Policy:** `Dialup/IPsec Interface` + `ISP Link` & `LAN`
* **Service / Src / Dst:** `ALL` (Restrict based on least privilege in production).
* **Logging:** `Log all sessions` (Crucial for ADVPN troubleshooting).
* **NAT:** **Strictly Disabled** for the overlay traffic (Required for OSPF routing overlay).
* **TCP MSS:** Adjust TCP MSS (e.g., `1350`) in the policy CLI to prevent MTU fragmentation drops.

> *Exception Rule:* FGT-3 requires NAT enabled from FGT-4 to the ISP. Disable NAT from the ISP to FGT-4 when troubleshooting UDP 500/4500 packets.

---

## 4. OSPF Routing & Overlay Interfaces

**Crucial OSPF Settings:** To allow Hub-and-Spoke and Spoke-to-Spoke routing over IPsec tunnels, the OSPF network type on the IPsec interfaces must be carefully managed.

### FGT-1 (Hub)
```text
Interface IP: 12.23.34.1
Remote IP: 12.23.34.2/24 (Allow Ping, OSPF)

OSPF Router ID: 1.1.1.1
OSPF Area 0.0.0.0:
  - 192.168.101.0/24 (Local LAN)
  - 12.23.34.0/24 (VPN Overlay)
Options: Inject default route always
CLI Tweak: set network-type point-to-multipoint (on IPsec interface)

```

### FGT-2 (Spoke)

```text
Interface IP: 12.23.34.2
Remote IP: 12.23.34.1/24 (Allow Ping, OSPF)

OSPF Router ID: 2.2.2.2
OSPF Area 0.0.0.0:
  - 192.168.102.0/24 (Local LAN)
  - 12.23.34.0/24 (VPN Overlay)
CLI Tweak: set network-type point-to-point

```

### FGT-4 (Spoke behind NAT)

```text
Interface IP: 12.23.34.4
Remote IP: 12.23.34.1/24 (Allow Ping, OSPF)

OSPF Router ID: 4.4.4.4
OSPF Area 0.0.0.0:
  - 192.168.104.0/24 (Local LAN)
  - 12.23.34.0/24 (VPN Overlay)
CLI Tweak: set network-type point-to-point

```

---

## 5. Mode Config (DHCP over IPsec)

To assign IPs dynamically to spokes over the Dialup IPsec tunnel instead of static assignments:

1. **FGT-1 Network Settings (Phase 1):** In Custom IPsec VPN, enable Mode Config (`Use System DNS`, `Assign IP`).
2. **FGT-1 Interface Configuration:**
* Subnet Mask: `255.255.255.0`
* Local IP: `12.23.34.1` | Remote IP: `12.23.34.254/24`
* **Crucial:** Enable the DHCP Server directly on this IPsec interface.


3. **Advanced Setup Adjustments:**
* **FGT-1 (Hub):** Disable all advanced components *except* `Device Creation` and `Add Route`.
* **FGT-2/FGT-4 (Clients):** Ensure `Mode Config` is explicitly enabled in Phase 1 settings to request the IP.



---

## 6. Advanced ADVPN & OSPF Troubleshooting

### OSPF & Routing Issues

* **OSPF Stuck in INIT:** Verify **Auto-Discovery Sender and Receiver** are enabled on FGT-1 Phase 1. If the Hub cannot dynamically create the Spoke interface, OSPF Hellos will fail.
* **Cisco Interoperability / Bad Next-Hops:** If spoke subnets route via public IPs instead of tunnels (a common issue with Cisco DMVPN interop), change the FortiGate OSPF Interface Network Type via CLI:

```bash
config router ospf
  config ospf-interface
    edit "ipsec-tun1"
      set network-type point-to-multipoint
    next
  end
end

```

* **Spoke-to-Spoke Traffic:** Enable `set exchange-ip-addr4` via CLI on FGT-1 Phase 1 to mimic Cisco DMVPN behavior and allow spokes to learn each other's physical IPs for shortcut tunnels. *(Dialup user groups alone do not trigger this).*

### Essential CLI Diagnostic Commands

```bash
# Verify Phase 1 and NAT-T status
diagnose vpn ike gateway list 

# Verify Phase 2 selectors and active Subnets
diagnose vpn tunnel list

# Real-time OSPF neighbor state and LSDB
get router info ospf neighbor
get router info ospf database

# Deep IKE Debugging (Run when tunnels fail to establish)
diagnose vpn ike log filter dst-addr4 <Peer_Public_IP>
diagnose debug application ike -1
diagnose debug enable

```

```

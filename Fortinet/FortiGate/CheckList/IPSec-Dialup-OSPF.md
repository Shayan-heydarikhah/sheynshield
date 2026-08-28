Copy and paste this Markdown checklist directly into your GitHub repository to track the deployment progress.

**Identity & Fabric Prerequisites**

* [ ] Define Active Directory as an LDAP server on the Hub (FGT-1) under `User & Authentication > LDAP Servers`.
* [ ] Configure AD FSSO under `Security Fabric > External Connectors` for dynamic user mapping.
* [ ] Create local Firewall User Groups mapped to LDAP/FSSO groups for IPsec XAuth.

**IPsec Phase 1 & 2 Parameters**

* [ ] Set Gateway Type: Dialup User on Hub (FGT-1) / Static IP on Spokes (FGT-2, FGT-4).
* [ ] Configure IKEv1 with Aggressive Mode across all nodes.
* [ ] Set Phase 1 Crypto: DES, MD5, DH 5.
* [ ] Set Phase 2 Crypto: DES, MD5, PFS DH 5, Auto-negotiate.
* [ ] Configure DPD: `On-Idle` for Hub (FGT-1) and `On-Demand` for Spokes.
* [ ] Set XAuth: `Server: Auto` on Hub / Configure client credentials (`u1`, `u2`) on Spokes.
* [ ] Ensure `network-id` is identically configured via CLI on Phase 1 for all nodes.
* [ ] Enable `Device Creation` on all nodes.
* [ ] Enable `Auto-Discovery Sender & Receiver` on FGT-1 and FGT-4.
* [ ] Disable `Add Route` on FGT-1 and FGT-4; Enable it on FGT-2.

**Global Firewall Policies**

* [ ] Create Incoming Policy: Dialup/IPsec Interface + ISP Link & LAN.
* [ ] Create Outgoing Policy: Dialup/IPsec Interface + ISP Link & LAN.
* [ ] Set Service / Src / Dst to `ALL` (restrict as necessary for production).
* [ ] Enable `Log all sessions` for ADVPN troubleshooting.
* [ ] **Strictly Disable NAT** on overlay policies (Exception: NAT enabled from FGT-4 to ISP on FGT-3).
* [ ] Adjust TCP MSS (e.g., `1350`) in the policy CLI to prevent MTU fragmentation.

**OSPF Routing & Overlay Interfaces**

* [ ] Assign Interface IP and Remote IP/Subnet on IPsec virtual interfaces.
* [ ] Allow `Ping` and `OSPF` administrative access on IPsec interfaces.
* [ ] Assign unique OSPF Router IDs (e.g., `1.1.1.1`, `2.2.2.2`).
* [ ] Announce Local LAN (e.g., `192.168.10x.0/24`) and VPN Overlay (`12.23.34.0/24`) in OSPF Area `0.0.0.0`.
* [ ] Inject default route always on the Hub (FGT-1).
* [ ] CLI Tweak: Set OSPF network-type to `point-to-multipoint` on FGT-1.
* [ ] CLI Tweak: Set OSPF network-type to `point-to-point` on Spokes (FGT-2, FGT-4).

**Mode Config (DHCP over IPsec)**

* [ ] Enable `Mode Config` (`Use System DNS`, `Assign IP`) in FGT-1 Phase 1 settings.
* [ ] Set Subnet Mask, Local IP, and Remote IP range on the FGT-1 IPsec interface.
* [ ] Enable the DHCP Server directly on the FGT-1 IPsec interface.
* [ ] On FGT-1 Phase 1 Advanced Setup: Disable all components except `Device Creation` and `Add Route`.
* [ ] Explicitly enable `Mode Config` in Phase 1 on Spoke clients to request IPs.

**Advanced ADVPN & OSPF Validation**

* [ ] Verify `Auto-Discovery Sender and Receiver` on FGT-1 if OSPF remains in INIT state.
* [ ] Validate OSPF network types via CLI if subnets route via public IPs (bad next-hop).
* [ ] CLI Tweak: Enable `set exchange-ip-addr4` on FGT-1 Phase 1 to permit Spoke-to-Spoke shortcut tunnels.
* [ ] Run `diagnose vpn ike gateway list` to verify Phase 1 and NAT-T status.
* [ ] Run `diagnose vpn tunnel list` to verify active Phase 2 subnets.
* [ ] Run `get router info ospf neighbor` to confirm adjacency.
* [ ] Run `diagnose debug application ike -1` and `diagnose debug enable` for deep IKE troubleshooting.

# 🛡️ Advanced FortiOS Dial-up IPsec & eBGP Architecture Guide

A deep-dive engineering reference for deploying scalable Dial-up IPsec tunnels integrated with eBGP routing in a Hub-and-Spoke topology. This guide covers granular CLI configurations, NAT-Traversal (NAT-T) scenarios, and critical architectural caveats regarding ADVPN limitations.

---

## 1. 🔑 Identity Integration (FGT-1 Hub)

Before tunnel initialization, the Hub must be capable of authenticating dynamic Spoke connections via Active Directory.

### LDAP & FSSO Configuration
1. **LDAP Server:** Navigate to `User & Authentication > LDAP Servers`. Map the AD server IP, bind credentials, and Base DN.
2. **FSSO Connector:** Navigate to `Security Fabric > External Connectors`. Create a Fortinet Single Sign-On (FSSO) agent connection to dynamically map AD groups to IP addresses.
3. **Firewall User Groups:** Create a local User Group (e.g., `Spoke-Routers`) and add the remote LDAP/FSSO server as the remote authentication server.

---

## 2. 🌐 IPsec Overlay Topology & Parameter Matrix

| Parameter | **FGT-1 (Hub / HQ)** | **FGT-2 (Standard Spoke)** | **FGT-4 (Spoke Behind NAT FGT-3)** |
| :--- | :--- | :--- | :--- |
| **Gateway Type** | Dialup User | Static IP (FGT-1 Public IP) | Static IP (FGT-1 Public IP) |
| **Mode & IKE** | Aggressive Mode / IKEv1 | Aggressive Mode / IKEv1 | Aggressive Mode / IKEv1 |
| **NAT-Traversal** | Enable (Forced if needed) | Enable | Enable (Critical for UDP 4500) |
| **Dead Peer Detection** | On-Idle (Saves Hub CPU) | On-Demand | On-Demand |
| **Phase 1 Crypto** | DES / MD5 / DH 5 | DES / MD5 / DH 5 | DES / MD5 / DH 5 |
| **Phase 2 Crypto** | DES / MD5 / PFS DH 5 | DES / MD5 / PFS DH 5 | DES / MD5 / PFS DH 5 |
| **XAuth Setup** | Server Type: Auto | Client: `u1` / `1qaz@WSX` | Client: `u2` / `1qaz@WSX` |
| **Phase 2 Selectors** | `0.0.0.0/0` to `0.0.0.0/0` | `0.0.0.0/0` to `0.0.0.0/0` | `0.0.0.0/0` to `0.0.0.0/0` |

---

## 3. 🏢 FGT-1 (Hub) Configuration

### IPsec Phase 1 & 2 (CLI)
When multiple Spokes connect to a single Dial-up interface, `add-route` **must** be disabled to prevent routing table corruption. Routing will be handled exclusively by eBGP.

```bash
config vpn ipsec phase1-interface
    edit "Hub-Dialup"
        set type dynamic
        set interface "wan1"
        set ike-version 1
        set mode aggressive
        set peertype any
        set net-device enable
        set proposal des-md5
        set dpd on-idle
        set dhgrp 5
        set xauthtype auto
        set authusrgrp "Spoke-Routers"
        set auto-discovery-sender enable
        set auto-discovery-receiver enable
        set add-route disable 
    next
end

config vpn ipsec phase2-interface
    edit "Hub-Dialup-P2"
        set phase1name "Hub-Dialup"
        set proposal des-md5
        set pfs enable
        set auto-negotiate enable
    next
end

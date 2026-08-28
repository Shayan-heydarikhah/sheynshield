```markdown
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

```

### Overlay Interface & eBGP (CLI)

```bash
config system interface
    edit "Hub-Dialup"
        set vdom "root"
        set ip 12.23.34.1 255.255.255.255
        set allowaccess ping
        set type tunnel
        set remote-ip 12.23.34.254 255.255.255.0
    next
end

config router bgp
    set as 65001
    set router-id 1.1.1.1
    config network
        edit 1
            set prefix 192.168.101.0 255.255.255.0
        next
    end
    config neighbor-group
        edit "Spokes"
            set remote-as 65002
            set next-hop-self enable
        next
    end
    config neighbor-range
        edit 1
            set prefix 12.23.34.0 255.255.255.0
            set neighbor-group "Spokes"
        next
    end
end

```

> **🛑 ADVPN Caveat:** Fortinet strictly advises against enabling `exchange-ip-addr4` (ADVPN shortcut creation) when mixing Dial-up topologies with BGP. The exchange interfaces will conflict and disable. For full ADVPN shortcut functionality via BGP, a Hub-and-Spoke (iBGP) or standard Site-to-Site architecture is strictly required.

---

## 4. 🏢 FGT-2 (Standard Spoke) Configuration

```bash
config vpn ipsec phase1-interface
    edit "To-Hub"
        set interface "wan1"
        set ike-version 1
        set mode aggressive
        set remote-gw <FGT-1_Public_IP>
        set net-device enable
        set proposal des-md5
        set add-route enable
        set xauthtype client
        set xauthusr "u1"
        set xauthpwd "1qaz@WSX"
    next
end

config system interface
    edit "To-Hub"
        set ip 12.23.34.2 255.255.255.255
        set remote-ip 12.23.34.1 255.255.255.255
        set allowaccess ping
    next
end

config router bgp
    set as 65002
    set router-id 2.2.2.2
    config network
        edit 1
            set prefix 192.168.102.0 255.255.255.0
        next
    end
    config neighbor
        edit "12.23.34.1"
            set remote-as 65001
            set next-hop-self enable
        next
    end
end

```

---

## 5. 🏢 FGT-4 (Spoke Behind NAT) & FGT-3 Configurations

### FGT-3 (The NAT Router)

FGT-3 sits between FGT-4 and the ISP. IPsec NAT-T (UDP 4500) relies on predictable translation.

* **FGT-4 ➔ ISP (Outbound):** `NAT Enabled`. Overload (PAT) is required for FGT-4 to establish the outbound IKE SA.
* **ISP ➔ FGT-4 (Inbound):** `NAT Disabled`. To ensure clean troubleshooting of Phase 1 encapsulation drops, do not apply inbound Destination NAT (VIP) unless acting as a responder.

### FGT-4 (The Spoke)

Configuration mirrors FGT-2 exactly, but XAuth credentials and BGP AS change:

* **XAuth User:** `u2`
* **XAuth Pass:** `1qaz@WSX`
* **Tunnel IP:** `12.23.34.4`
* **BGP AS:** `65004` (Router ID `4.4.4.4`)

---

## 6. 🛡️ Global Firewall Policies

To form eBGP adjacencies and allow overlay traffic, apply these policies universally. **Crucial:** NAT must be completely disabled on these policies to preserve the original IP headers for BGP TCP/179 routing.

```bash
config firewall policy
    edit 1
        set name "Overlay-In"
        set srcintf "Hub-Dialup" # (Or "To-Hub" on Spokes)
        set dstintf "lan"
        set action accept
        set srcaddr "all"
        set dstaddr "all"
        set schedule "always"
        set service "ALL"
        set logtraffic all
        set nat disable
    next
    edit 2
        set name "Overlay-Out"
        set srcintf "lan"
        set dstintf "Hub-Dialup"
        set action accept
        set srcaddr "all"
        set dstaddr "all"
        set schedule "always"
        set service "ALL"
        set logtraffic all
        set nat disable
    next
end

```

---

## 7. 🛠️ Advanced Diagnostic & Troubleshooting Commands

When Dial-up or BGP states fail to establish, utilize the following CLI commands:

**IPsec Verification:**

```bash
# Check Phase 1 SA and NAT-T port status (Look for UDP 4500)
diagnose vpn ike gateway list

# Check Phase 2 SA, traffic counters, and proxy IDs
diagnose vpn tunnel list

# Deep IKE Debugging (Real-time Phase 1/2 negotiation)
diagnose vpn ike log filter dst-addr4 <Remote_IP>
diagnose debug application ike -1
diagnose debug enable

```

**BGP Overlay Verification:**

```bash
# Verify BGP peering state (Must be 'Established')
get router info bgp summary

# View the learned BGP routing table via the overlay
get router info bgp network

# Verify the FortiOS Routing Information Base (RIB)
get router info routing-table bgp

```

```

```

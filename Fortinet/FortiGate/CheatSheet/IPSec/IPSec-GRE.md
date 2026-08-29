# IPSec GRE Tunnel 

> **FortiGate GRE Tunnel **
>
> GRE provides a logical point-to-point overlay tunnel between two endpoints. GRE itself does **not provide encryption**.
>
> **GRE = IP Protocol 47**
>
> Use **GRE over IPsec** when GRE encapsulation is required together with confidentiality and integrity.

---

## Table of Contents

* [GRE Overview](#gre-overview)
* [GRE Tunnel Configuration](#1-gre-tunnel-configuration)
* [GRE Tunnel Interface](#2-gre-tunnel-interface)
* [Underlay Routing](#3-underlay-routing)
* [Remote Network Routing](#4-remote-network-routing)
* [Firewall Policy](#5-firewall-policy)
* [GRE-to-GRE Topology](#6-gre-to-gre-topology)
* [GRE Over IPsec](#7-gre-over-ipsec)
* [IPsec Phase 1](#8-ipsec-phase-1)
* [IPsec Phase 2 for GRE](#9-ipsec-phase-2-for-gre)
* [GRE over IPsec Interface](#10-gre-over-ipsec-interface)
* [Routing Over GRE over IPsec](#11-routing-over-gre-over-ipsec)
* [Firewall Policies for GRE over IPsec](#12-firewall-policies-for-gre-over-ipsec)
* [Encapsulation](#13-packet-encapsulation)
* [GRE vs GRE over IPsec](#14-gre-vs-gre-over-ipsec)
* [Troubleshooting](#15-troubleshooting)
* [Common Design Mistakes](#16-common-design-mistakes)
* [Deployment Checklist](#17-deployment-checklist)
* [Core Mental Model](#18-core-mental-model)

---

# GRE Overview

GRE creates an overlay tunnel between two endpoints.

```text
                    UNDERLAY
              WAN / Internet / MPLS
                       |
              +--------+--------+
              |                 |
              v                 v
         FortiGate-A       FortiGate-B
          9.9.9.9           8.8.8.8
              |                 |
              +------ GRE ------+
                       |
                 Overlay Network
```

### GRE Characteristics

| Feature                          | GRE             |
| -------------------------------- | --------------- |
| Tunnel type                      | Layer-3 overlay |
| IP protocol                      | **47**          |
| Encryption                       | ❌ No            |
| Integrity                        | ❌ No            |
| Authentication                   | ❌ No            |
| Routing through tunnel           | ✅               |
| Point-to-point connectivity      | ✅               |
| Suitable for encrypted transport | ❌               |
| Can be protected by IPsec        | ✅               |

> **Key point:** GRE provides encapsulation, not security.

---

# 1. GRE Tunnel Configuration

Create the GRE tunnel:

```bash
config system gre-tunnel
    edit "tunn-1"
        set interface "port1"
        set local-gw 9.9.9.9
        set remote-gw 8.8.8.8
    next
end
```

### Parameters

| Parameter   | Description                                     |
| ----------- | ----------------------------------------------- |
| `interface` | Physical/WAN interface used by the GRE underlay |
| `local-gw`  | Local GRE endpoint                              |
| `remote-gw` | Remote GRE endpoint                             |
| `tunn-1`    | GRE tunnel object/interface name                |

### Endpoint Relationship

FortiGate-A:

```text
local-gw  = 9.9.9.9
remote-gw = 8.8.8.8
```

FortiGate-B:

```text
local-gw  = 8.8.8.8
remote-gw = 9.9.9.9
```

---

# 2. GRE Tunnel Interface

Assign IP addressing to the GRE tunnel:

```bash
config system interface
    edit "tunn-1"
        set mode static
        set ip 10.10.10.1 255.255.255.255
        set remote-ip 10.10.10.2 255.255.255.252
        set allowaccess ping
    next
end
```

Verify:

```bash
show system interface
```

### Overlay vs Underlay

```text
                UNDERLAY
                  |
                  v
             port1 / WAN
                  |
                  |
              +-------+
              |  GRE  |
              +-------+
                  |
                  v
            10.10.10.1
            10.10.10.2
                  |
                  v
                LANs
```

The physical WAN interface carries the GRE packet.

The GRE interface provides the **overlay routing path**.

---

# 3. Underlay Routing

The remote GRE endpoint must be reachable through the WAN/underlay.

Example:

```bash
config router static
    edit 85
        set dst 0.0.0.0 0.0.0.0
        set gateway 192.168.10.1
        set device "port1"
    next
end
```

Depending on the topology, the default route may use another upstream gateway:

```bash
config router static
    edit 85
        set dst 0.0.0.0 0.0.0.0
        set gateway 9.9.9.9
        set device "port1"
    next
end
```

### Critical Routing Rule

The route to the **remote GRE endpoint** must use the underlay.

```text
Remote GRE Endpoint
        |
        v
      WAN
        |
    Internet
        |
        v
Remote FortiGate
```

Do **not** create a routing dependency where the remote GRE endpoint itself is reached through the GRE tunnel.

```text
Remote GRE Endpoint
        |
        X
    GRE Tunnel
```

---

# 4. Remote Network Routing

Once the GRE tunnel is available, remote LAN prefixes can be routed through the GRE interface.

```bash
config router static
    edit 7
        set dst 192.168.30.0 255.255.255.0
        set device "tunn-1"
    next

    edit 8
        set dst 172.16.100.0 255.255.255.0
        set device "tunn-1"
    next
end
```

### Example

```text
LAN-A
192.168.10.0/24
      |
      v
FortiGate-A
      |
   tunn-1
      |
     GRE
      |
FortiGate-B
      |
      v
LAN-B
192.168.30.0/24
```

Routing decision:

```text
192.168.30.0/24
       |
       v
    tunn-1
```

### Remote Side

The remote FortiGate must also have a route back toward the local LAN:

```text
Local LAN
    |
    v
   GRE
    |
Remote FortiGate
    |
    v
Remote LAN
```

Without the return route:

```text
Forward path   = OK
Return path    = FAIL
```

---

# 5. Firewall Policy

GRE connectivity does not automatically mean that transit traffic is allowed.

Create a policy from LAN toward the GRE interface:

```bash
config firewall policy
    edit 4
        set name "lan-to-gre"
        set srcintf "zone-lan"
        set dstintf "tunn-1"
        set srcaddr "vlan10" "vlan20"
        set dstaddr "all"
        set service "PING"
        set schedule "always"
        set logtraffic "all"
        set action "accept"
    next
end
```

For production, replace broad objects with the actual:

* Source networks
* Remote destination networks
* Required services

Example:

```text
Source:
10.10.10.0/24

Destination:
192.168.30.0/24

Services:
HTTPS
SSH
DNS
RDP
```

### Reverse Traffic

If traffic must be initiated from the remote side, ensure the reverse direction is permitted according to the firewall policy/session design.

Example:

```bash
config firewall policy
    edit 5
        set name "gre-to-lan"
        set srcintf "tunn-1"
        set dstintf "zone-lan"
        set srcaddr "all"
        set dstaddr "all"
        set service "PING"
        set schedule "always"
        set logtraffic "all"
        set action "accept"
    next
end
```

---

# 6. GRE-to-GRE Topology

Basic FortiGate-to-FortiGate GRE:

```text
                       UNDERLAY
                 Public / WAN Network
                         |
             +-----------+-----------+
             |                       |
             v                       v
        FortiGate-A             FortiGate-B
          9.9.9.9                 8.8.8.8
             |                       |
             +--------- GRE ---------+
                       |
                10.10.10.0/30
                 .1           .2
```

### FortiGate-A

```bash
config system gre-tunnel
    edit "GRE-B"
        set interface "port1"
        set local-gw 9.9.9.9
        set remote-gw 8.8.8.8
    next
end
```

```bash
config system interface
    edit "GRE-B"
        set mode static
        set ip 10.10.10.1 255.255.255.255
        set remote-ip 10.10.10.2 255.255.255.252
        set allowaccess ping
    next
end
```

### FortiGate-B

Use the opposite endpoint relationship:

```text
local-gw  = 8.8.8.8
remote-gw = 9.9.9.9
```

---

# 7. GRE Over IPsec

GRE itself does not encrypt traffic.

When encryption is required:

```text
GRE + IPsec
```

provides:

```text
GRE Encapsulation
        +
IPsec Security
        =
Encrypted GRE Transport
```

### Logical Architecture

```text
LAN
 |
FortiGate-A
 |
 GRE
 |
IPsec
 |
WAN / Internet
 |
IPsec
 |
 GRE
 |
FortiGate-B
 |
LAN
```

---

# 8. IPsec Phase 1

Example based on the original configuration:

```bash
config vpn ipsec phase1-interface
    edit "greipsecif"
        set interface "port3"
        set peertype any
        set proposal des-sha1
        set remote-gw 192.168.10.2
        set psksecret 12345678
    next
end
```

Verify:

```bash
show vpn ipsec phase1-interface
```

### Phase 1 Components

| Parameter   | Purpose                  |
| ----------- | ------------------------ |
| `interface` | IPsec underlay interface |
| `remote-gw` | Remote IPsec peer        |
| `proposal`  | Cryptographic proposal   |
| `psksecret` | Pre-shared key           |
| `peertype`  | Remote peer type         |

> ⚠️ **Security note:** `DES` and weak legacy cryptographic examples should not be used for modern production deployments. Prefer current FortiOS-recommended AES-based encryption, SHA-2 integrity, strong DH groups, and appropriate IKE versions.

---

# 9. IPsec Phase 2 for GRE

GRE uses:

```text
IP Protocol 47
```

Therefore, the IPsec Phase 2 configuration can identify GRE traffic with protocol 47:

```bash
config vpn ipsec phase2-interface
    edit "GREipsecif"
        set phase1name "greipsecif"
        set proposal des-sha1
        set protocol 47
    next
end
```

Verify:

```bash
show vpn ipsec phase2-interface
```

### Protocol Relationship

```text
IPsec Phase 2
      |
      +---- Protocol 47
                 |
                 v
                GRE
```

> The exact Phase 2 selectors and protocol settings should match the remote peer and the intended FortiOS design.

---

# 10. GRE-over-IPsec Interface

Create the IPsec interface:

```bash
config system interface
    edit "greipsecif"
        set ip 10.10.10.1 255.255.255.252
        set remote-ip 10.10.10.2 255.255.255.252
    next
end
```

Create GRE over the IPsec interface:

```bash
config system gre-tunnel
    edit "gre-hq"
        set interface "greipsecif"
        set remote-gw 10.10.10.2
        set local-gw 10.10.10.1
    next
end
```

### Interface Relationship

```text
Physical WAN
     |
     v
 IPsec Interface
  greipsecif
     |
     v
 GRE Tunnel
   gre-hq
     |
     v
Remote Networks
```

---

# 11. Routing Over GRE Over IPsec

Route remote networks through the GRE interface:

```bash
config router static
    edit 2
        set dst 10.1.100.0 255.255.255.0
        set device "gre-hq"
    next
end
```

Another remote network:

```bash
config router static
    edit 3
        set dst 172.16.101.0 255.255.255.0
        set device "gre-hq"
    next
end
```

### Traffic Flow

```text
10.1.10.0/24
      |
      v
FortiGate-A
      |
    gre-hq
      |
     GRE
      |
    IPsec
      |
   Internet
      |
    IPsec
      |
     GRE
      |
FortiGate-B
      |
      v
10.1.100.0/24
```

---

# 12. Firewall Policies for GRE Over IPsec

### LAN → GRE

```bash
config firewall policy
    edit 1
        set name "lan-to-gre-ipsec"
        set srcintf "port2"
        set dstintf "gre-hq"
        set srcaddr "all"
        set dstaddr "all"
        set action "accept"
        set schedule "always"
        set service "ALL"
    next
end
```

### GRE → LAN

```bash
config firewall policy
    edit 2
        set name "gre-ipsec-to-lan"
        set srcintf "gre-hq"
        set dstintf "port2"
        set srcaddr "all"
        set dstaddr "all"
        set action "accept"
        set schedule "always"
        set service "ALL"
    next
end
```

### GRE/IPsec Interface Policy

The original NSE4 notes also included a same-interface policy:

```bash
config firewall policy
    edit 3
        set name "gre-ipsec-self"
        set srcintf "greipsecif"
        set dstintf "greipsecif"
        set srcaddr "all"
        set dstaddr "all"
        set action "accept"
        set schedule "always"
        set service "ALL"
    next
end
```

> Use this only when required by the actual traffic flow/design. Do not blindly add broad `ALL → ALL` policies to production firewalls.

---

# 13. Packet Encapsulation

### Plain GRE

```text
Original Packet
      |
      v
    GRE
      |
      v
   Underlay
      |
      v
Remote Endpoint
```

### GRE Over IPsec

```text
Original Packet
      |
      v
    GRE
      |
      v
   IPsec
      |
      v
Encrypted Packet
      |
      v
   Internet
```

Conceptually:

```text
[ Original IP Packet ]
          ↓
[ GRE Encapsulation ]
          ↓
[ IPsec Protection ]
          ↓
[ WAN / Internet ]
```

At the remote side:

```text
WAN
 ↓
IPsec Decapsulation
 ↓
GRE Decapsulation
 ↓
Original IP Packet
```

---

# 14. GRE vs GRE Over IPsec

| Feature                        | GRE | GRE over IPsec |
| ------------------------------ | :-: | :------------: |
| Overlay tunnel                 |  ✅  |        ✅       |
| IP Protocol 47                 |  ✅  |        ✅       |
| Encryption                     |  ❌  |        ✅       |
| Integrity                      |  ❌  |        ✅       |
| Authentication                 |  ❌  |        ✅       |
| Routing support                |  ✅  |        ✅       |
| Secure Internet transport      |  ❌  |        ✅       |
| Additional overhead            | Low |     Higher     |
| Suitable for sensitive traffic |  ❌  |        ✅       |

### Design Rule

```text
Need GRE only?
        |
        v
       GRE

Need GRE + Encryption?
        |
        v
   GRE over IPsec
```

---

# 15. Troubleshooting

## 15.1 Check Underlay Connectivity

First verify reachability to the remote endpoint:

```bash
execute ping 8.8.8.8
```

If the remote GRE endpoint is unreachable:

```text
Underlay FAIL
     ↓
GRE FAIL
     ↓
Application FAIL
```

---

## 15.2 Check GRE Configuration

```bash
show system gre-tunnel
```

Verify:

```text
interface
local-gw
remote-gw
```

---

## 15.3 Check GRE Interface

```bash
show system interface
```

Verify:

```text
tunn-1
GRE-B
gre-hq
greipsecif
```

---

## 15.4 Test Tunnel IP

```bash
execute ping 10.10.10.2
```

Expected path:

```text
Local FortiGate
      |
    GRE
      |
Remote FortiGate
      |
10.10.10.2
```

---

## 15.5 Verify Routing

```bash
get router info routing-table all
```

For a specific destination:

```bash
get router info routing-table details 192.168.30.0
```

Confirm that the destination uses:

```text
tunn-1
```

or:

```text
gre-hq
```

---

## 15.6 Verify IPsec

For GRE over IPsec:

```bash
diagnose vpn ike gateway list
```

```bash
diagnose vpn tunnel list
```

Verify:

```text
IKE Phase 1 = UP
IPsec Phase 2 = UP
```

---

# 16. Common Design Mistakes

## ❌ 1. No Underlay Reachability

```text
Remote GRE Endpoint
        X
     Underlay
```

GRE cannot work correctly if the endpoint itself is unreachable.

---

## ❌ 2. Routing the GRE Endpoint Through GRE

```text
Remote GRE Public IP
        |
        X
    GRE Tunnel
```

The GRE endpoint must be reached through the underlay.

---

## ❌ 3. Missing Remote LAN Route

```text
GRE = UP
Route = Missing
Traffic = FAIL
```

A working tunnel does not automatically create routes to remote LANs.

---

## ❌ 4. Missing Return Route

```text
Forward path = OK
Return path  = FAIL
```

Both sides need appropriate routing.

---

## ❌ 5. Firewall Policy Denies Traffic

```text
Tunnel = UP
Route  = Correct
Policy = DENY

        ↓

Traffic = FAIL
```

---

## ❌ 6. Expecting GRE to Encrypt

```text
GRE ≠ Encryption
```

For encrypted transport:

```text
GRE + IPsec
```

---

## ❌ 7. Using Legacy Cryptography in Production

Avoid copying old lab values such as:

```text
DES
MD5
Weak DH groups
```

into production.

Use current FortiOS-recommended cryptographic suites instead.

---

# 17. Deployment Checklist

## GRE

```text
[ ] Remote GRE endpoint is reachable through underlay
[ ] local-gw is correct
[ ] remote-gw is correct
[ ] GRE interface is configured
[ ] Tunnel IP addresses are configured
[ ] Remote LAN routes point to GRE
[ ] Return routes exist on remote side
[ ] Firewall policies permit required traffic
[ ] GRE endpoint is NOT routed through GRE
[ ] GRE protocol = 47
```

## GRE Over IPsec

```text
[ ] IPsec Phase 1 configured
[ ] IPsec Phase 1 established
[ ] IPsec Phase 2 configured
[ ] GRE protocol 47 is handled correctly
[ ] IPsec interface exists
[ ] GRE runs over the IPsec interface
[ ] Remote LAN routes point to GRE
[ ] Return routes exist
[ ] Firewall policies permit required traffic
[ ] Strong modern cryptography is used
```

---

# 18. Core Mental Model

## Plain GRE

```text
        UNDERLAY
            |
            v
       FortiGate-A
            |
           GRE
            |
        Internet
            |
           GRE
            |
       FortiGate-B
            |
            v
           LAN
```

### Security

```text
GRE
 |
 +---- Encapsulation
 |
 +---- NO encryption
```

---

## GRE Over IPsec

```text
        UNDERLAY
            |
            v
       FortiGate-A
            |
           GRE
            |
          IPsec
            |
      Encryption
            |
         Internet
            |
       Decryption
            |
          IPsec
            |
           GRE
            |
       FortiGate-B
            |
            v
           LAN
```

### Security

```text
GRE
 |
 +---- Encapsulation
 |
IPsec
 |
 +---- Encryption
 +---- Integrity
 +---- Authentication
```

---

# Quick Reference

| Requirement                         | Recommended Design                       |
| ----------------------------------- | ---------------------------------------- |
| Basic point-to-point overlay        | GRE                                      |
| GRE without encryption              | GRE                                      |
| GRE + encryption                    | GRE over IPsec                           |
| Remote-access VPN                   | IPsec / supported remote-access solution |
| Site-to-site encrypted connectivity | IPsec                                    |
| GRE protocol                        | **47**                                   |
| GRE encryption                      | **None**                                 |
| Remote GRE endpoint routing         | **Underlay/WAN**                         |
| Remote LAN routing                  | **GRE interface**                        |

---

## Key Takeaways

> **1. GRE is an overlay tunnel, not a security protocol.**

> **2. GRE uses IP protocol 47.**

> **3. The remote GRE endpoint must be reachable through the underlay.**

> **4. Remote LAN prefixes must be routed through the GRE interface.**

> **5. GRE over IPsec combines GRE encapsulation with IPsec security.**

> **6. A tunnel being UP does not guarantee application connectivity — verify routing and firewall policy.**

> **7. Do not copy legacy DES/MD5 lab configurations into production.**

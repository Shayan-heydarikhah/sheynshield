# 🔐 FortiGate GRE & GRE over IPsec — Configuration & Troubleshooting Checklist

> **SheynShield | Engineering Secure Networks**
>
> **Topic:** FortiGate GRE Tunnel, GRE over IPsec, Routing & Troubleshooting
> **Focus:** NSE4 / Network Security Engineering / Practical Deployment
> **Protocol:** GRE = **IP Protocol 47**

---

## 📌 Overview

GRE creates a **Layer-3 overlay tunnel** between two endpoints.

> ⚠️ **GRE does NOT provide encryption, integrity, or confidentiality.**

For secure GRE transport:

```text
GRE
 +
IPsec
 =
Encrypted GRE Tunnel
```

### Core Architecture

```text
                UNDERLAY
          WAN / Internet / MPLS
                  |
          +-------+-------+
          |               |
      FortiGate-A     FortiGate-B
       9.9.9.9         8.8.8.8
          |               |
          +----- GRE -----+
                 |
           Overlay Network
```

---

# 🎯 1. GRE Fundamentals Checklist

* [ ] Understand that GRE is a **Layer-3 overlay tunnel**
* [ ] Confirm GRE uses **IP protocol 47**
* [ ] Remember GRE provides encapsulation, not encryption
* [ ] Identify the **underlay** path
* [ ] Identify the **overlay** path
* [ ] Verify the remote GRE endpoint is reachable through the underlay
* [ ] Verify remote LAN prefixes are routed through the GRE interface
* [ ] Verify return routing exists
* [ ] Verify firewall policies permit the required traffic

### GRE Security Model

| Capability            |   GRE  |
| --------------------- | :----: |
| Layer-3 overlay       |    ✅   |
| Encapsulation         |    ✅   |
| Routing support       |    ✅   |
| Encryption            |    ❌   |
| Integrity             |    ❌   |
| Authentication        |    ❌   |
| IP Protocol           | **47** |
| Can run through IPsec |    ✅   |

> **Mental shortcut:**
> **GRE = Tunnel**
> **IPsec = Security**

---

# 🧩 2. Underlay vs Overlay Checklist

### Underlay

* [ ] WAN interface is operational
* [ ] Local GRE endpoint has correct IP
* [ ] Remote GRE endpoint is reachable
* [ ] ISP/upstream routing is correct
* [ ] No routing loop exists

```text
FortiGate-A
    |
   WAN
    |
Internet / MPLS
    |
   WAN
    |
FortiGate-B
```

### Overlay

* [ ] GRE tunnel exists
* [ ] GRE tunnel interface has an IP
* [ ] Remote GRE tunnel IP is reachable
* [ ] Remote LAN routes use the GRE interface

```text
LAN
 |
FortiGate-A
 |
 GRE
 |
FortiGate-B
 |
LAN
```

### Critical Rule

```text
Remote GRE Endpoint
        |
        v
     UNDERLAY
        |
        v
     GRE Tunnel
```

Never create:

```text
Remote GRE Endpoint
        |
        X
     GRE Tunnel
```

> **The tunnel endpoint must be reachable before the tunnel can carry traffic.**

---

# ⚙️ 3. GRE Tunnel Configuration Checklist

Configure the GRE tunnel:

```bash
config system gre-tunnel
    edit "tunn-1"
        set interface "port1"
        set local-gw 9.9.9.9
        set remote-gw 8.8.8.8
    next
end
```

### Verify

* [ ] GRE object exists
* [ ] Tunnel name is correct
* [ ] Underlay interface is correct
* [ ] `local-gw` is correct
* [ ] `remote-gw` is correct
* [ ] Remote endpoint is reachable

### Endpoint Relationship

**FortiGate-A**

```text
local-gw  = 9.9.9.9
remote-gw = 8.8.8.8
```

**FortiGate-B**

```text
local-gw  = 8.8.8.8
remote-gw = 9.9.9.9
```

---

# 🌐 4. GRE Tunnel Interface Checklist

Configure the GRE interface:

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

### Verify

* [ ] GRE interface exists
* [ ] Interface name matches GRE tunnel
* [ ] Local tunnel IP is correct
* [ ] Remote tunnel IP is correct
* [ ] Addressing does not conflict with another interface
* [ ] Required management access is enabled

### Example

```text
FortiGate-A
10.10.10.1
     |
    GRE
     |
10.10.10.2
FortiGate-B
```

---

# 🛣️ 5. Underlay Routing Checklist

The remote GRE endpoint must be reachable via WAN/underlay.

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

### Verify

* [ ] Default route exists when required
* [ ] Specific route to remote GRE endpoint exists when required
* [ ] Route uses the WAN/underlay
* [ ] Route does NOT use the GRE interface
* [ ] Upstream gateway is reachable
* [ ] Remote endpoint responds to reachability tests

### Test

```bash
execute ping 8.8.8.8
```

---

# 🛰️ 6. Remote Network Routing Checklist

After GRE is operational:

* [ ] Add routes to remote LANs
* [ ] Point remote LAN routes to the GRE interface
* [ ] Configure return routes on the remote FortiGate
* [ ] Verify routing table
* [ ] Confirm no more-specific route overrides the GRE route

Example:

```bash
config router static
    edit 7
        set dst 192.168.30.0 255.255.255.0
        set device "tunn-1"
    next
end
```

Additional network:

```bash
config router static
    edit 8
        set dst 172.16.100.0 255.255.255.0
        set device "tunn-1"
    next
end
```

### Routing Model

```text
192.168.30.0/24
       |
       v
     GRE
       |
       v
Remote FortiGate
```

### Return Path

```text
Local LAN
    |
    v
Remote FortiGate
    |
    v
   GRE
    |
    v
Local FortiGate
```

> **Forward route without return route = broken connectivity.**

---

# 🔥 7. GRE Firewall Policy Checklist

GRE tunnel availability does not automatically permit transit traffic.

### LAN → GRE

* [ ] Source interface is correct
* [ ] Destination interface is GRE
* [ ] Source address objects are correct
* [ ] Destination address objects are correct
* [ ] Required services are allowed
* [ ] Logging is enabled where appropriate
* [ ] NAT is disabled unless explicitly required

Example:

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

### Production Principle

Avoid blindly deploying:

```text
ALL → ALL
```

Instead define:

```text
Required Source
+
Required Destination
+
Required Services
```

---

# 🔄 8. Reverse Traffic Checklist

If the remote side initiates traffic:

* [ ] GRE → LAN policy exists when required
* [ ] Source networks are correct
* [ ] Destination networks are correct
* [ ] Required services are allowed
* [ ] Return routing exists
* [ ] NAT behavior is correct

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

# 🔐 9. GRE over IPsec Checklist

Use GRE over IPsec when you need:

```text
GRE Encapsulation
        +
IPsec Security
```

### Verify

* [ ] IPsec Phase 1 exists
* [ ] Phase 1 establishes successfully
* [ ] Phase 2 exists
* [ ] GRE traffic is protected by IPsec
* [ ] IPsec interface exists
* [ ] GRE runs over the IPsec interface
* [ ] Routing uses the GRE interface
* [ ] Firewall policies allow the required traffic

### Architecture

```text
LAN
 |
FortiGate-A
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
LAN
```

---

# 🔑 10. IPsec Phase 1 Checklist

Example lab configuration:

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

### Verify

* [ ] Correct WAN interface selected
* [ ] Remote gateway is correct
* [ ] Peer type is appropriate
* [ ] Proposal matches remote peer
* [ ] Authentication method matches
* [ ] PSK matches
* [ ] IKE version is compatible
* [ ] Phase 1 reaches UP state

### ⚠️ Production Security

Do **not** copy legacy lab values such as:

```text
DES
MD5
Weak DH groups
Weak PSKs
```

into production.

Prefer:

```text
Modern IKE version
+
AES-based encryption
+
SHA-2 integrity
+
Strong DH/ECDH group
+
Strong random PSK or certificate authentication
```

---

# 🧩 11. IPsec Phase 2 for GRE Checklist

GRE uses:

```text
IP Protocol 47
```

Example:

```bash
config vpn ipsec phase2-interface
    edit "GREipsecif"
        set phase1name "greipsecif"
        set proposal des-sha1
        set protocol 47
    next
end
```

### Verify

* [ ] Phase 2 references correct Phase 1
* [ ] Proposal matches peer
* [ ] GRE protocol is handled correctly
* [ ] Selectors match the intended design
* [ ] Phase 2 SA establishes
* [ ] SPI values appear as expected

### Protocol Relationship

```text
IPsec Phase 2
      |
      +---- Protocol 47
                  |
                  v
                 GRE
```

---

# 🌐 12. GRE-over-IPsec Interface Checklist

Create the IPsec interface:

```bash
config system interface
    edit "greipsecif"
        set ip 10.10.10.1 255.255.255.252
        set remote-ip 10.10.10.2 255.255.255.252
    next
end
```

Then create GRE over the IPsec interface:

```bash
config system gre-tunnel
    edit "gre-hq"
        set interface "greipsecif"
        set remote-gw 10.10.10.2
        set local-gw 10.10.10.1
    next
end
```

### Interface Stack

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

Verify:

* [ ] WAN is reachable
* [ ] IPsec interface exists
* [ ] IPsec tunnel is established
* [ ] GRE uses the intended IPsec interface
* [ ] GRE tunnel addressing is correct
* [ ] Remote endpoint addressing is correct

---

# 🛣️ 13. Routing over GRE over IPsec Checklist

Route remote networks through the GRE interface:

```bash
config router static
    edit 2
        set dst 10.1.100.0 255.255.255.0
        set device "gre-hq"
    next
end
```

### Verify

* [ ] Remote LAN route exists
* [ ] Route points to `gre-hq`
* [ ] Return route exists
* [ ] IPsec interface is not accidentally selected
* [ ] WAN is not accidentally selected for remote LAN traffic
* [ ] Routing table shows the expected next hop/interface

---

# 📦 14. Packet Encapsulation Checklist

### Plain GRE

```text
Original Packet
      |
      v
GRE Encapsulation
      |
      v
Underlay
      |
      v
Remote Endpoint
```

### GRE over IPsec

```text
Original Packet
      |
      v
GRE Encapsulation
      |
      v
IPsec Protection
      |
      v
Encrypted Packet
      |
      v
Internet
```

Remote side:

```text
Internet
   ↓
IPsec Decapsulation
   ↓
GRE Decapsulation
   ↓
Original IP Packet
```

### Mental Model

```text
Original IP
    ↓
GRE
    ↓
IPsec
    ↓
WAN
```

---

# 🧪 15. GRE Troubleshooting Checklist

## Step 1 — Underlay

* [ ] Remote GRE endpoint is reachable
* [ ] WAN interface is UP
* [ ] Gateway is reachable
* [ ] Routing table contains a valid underlay path

```bash
execute ping 8.8.8.8
```

---

## Step 2 — GRE Configuration

```bash
show system gre-tunnel
```

Check:

* [ ] Tunnel name
* [ ] Underlay interface
* [ ] Local gateway
* [ ] Remote gateway

---

## Step 3 — GRE Interface

```bash
show system interface
```

Check:

* [ ] GRE interface exists
* [ ] Local IP
* [ ] Remote IP
* [ ] Interface status

---

## Step 4 — Tunnel IP

```bash
execute ping 10.10.10.2
```

Expected:

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

## Step 5 — Routing

```bash
get router info routing-table all
```

Specific destination:

```bash
get router info routing-table details 192.168.30.0
```

Confirm:

```text
Destination
    ↓
GRE Interface
```

---

## Step 6 — Firewall

* [ ] LAN → GRE policy exists
* [ ] GRE → LAN policy exists when required
* [ ] Correct source addresses
* [ ] Correct destination addresses
* [ ] Required services allowed
* [ ] NAT behavior is correct
* [ ] Traffic logs show expected action

---

# 🔎 16. GRE over IPsec Troubleshooting Checklist

## Phase 1

```bash
diagnose vpn ike gateway list
```

Check:

* [ ] IKE SA exists
* [ ] Correct peer
* [ ] Correct proposal
* [ ] Authentication succeeds
* [ ] Phase 1 is UP

## Phase 2

```bash
diagnose vpn tunnel list
```

Check:

* [ ] IPsec SA exists
* [ ] Correct SPI
* [ ] Correct selectors
* [ ] Correct proposal
* [ ] Phase 2 is UP

### Expected Dependency

```text
Underlay
   ↓
IKE Phase 1
   ↓
IPsec Phase 2
   ↓
IPsec Interface
   ↓
GRE
   ↓
Routing
   ↓
Firewall
   ↓
Application
```

---

# 🐛 17. Common GRE Design Mistakes

## ❌ Mistake 1 — No Underlay Reachability

```text
Remote GRE Endpoint
        X
     Underlay
```

Result:

```text
Underlay FAIL
    ↓
GRE FAIL
    ↓
Application FAIL
```

---

## ❌ Mistake 2 — Routing GRE Endpoint Through GRE

```text
Remote GRE Public IP
        |
        X
      GRE
```

### Correct

```text
Remote GRE Public IP
        |
        ↓
      WAN
        |
        ↓
      GRE
```

---

## ❌ Mistake 3 — Missing Remote LAN Route

```text
GRE = UP
Route = Missing
Traffic = FAIL
```

A tunnel being established does **not** automatically mean remote LAN networks are reachable.

---

## ❌ Mistake 4 — Missing Return Route

```text
Forward = OK
Return  = FAIL
```

Always validate both directions.

---

## ❌ Mistake 5 — Firewall Policy Denies Traffic

```text
Tunnel = UP
Route  = Correct
Policy = DENY
```

Result:

```text
Traffic = FAIL
```

---

## ❌ Mistake 6 — Assuming GRE Encrypts

```text
GRE ≠ Encryption
```

For confidentiality:

```text
GRE + IPsec
```

---

## ❌ Mistake 7 — Legacy Cryptography in Production

Do not deploy old lab proposals such as:

```text
DES
MD5
Weak DH
Weak PSK
```

without a specific interoperability requirement and documented risk acceptance.

---

# 📊 18. GRE vs GRE over IPsec

| Feature                        |  GRE  | GRE over IPsec |
| ------------------------------ | :---: | :------------: |
| Layer-3 overlay                |   ✅   |        ✅       |
| Encapsulation                  |   ✅   |        ✅       |
| IP Protocol 47                 |   ✅   |        ✅       |
| Encryption                     |   ❌   |        ✅       |
| Integrity                      |   ❌   |        ✅       |
| Peer authentication            |   ❌   |        ✅       |
| Routing support                |   ✅   |        ✅       |
| Internet-safe by itself        |   ❌   |        ✅       |
| Overhead                       | Lower |     Higher     |
| Suitable for sensitive traffic |   ❌   |        ✅       |

### Design Decision

```text
Need overlay only?
        ↓
       GRE

Need overlay + encryption?
        ↓
   GRE over IPsec
```

---

# 🧠 19. Core Mental Model

## Plain GRE

```text
UNDERLAY
   |
   v
FortiGate-A
   |
  GRE
   |
Internet / WAN
   |
  GRE
   |
FortiGate-B
   |
   v
 LAN
```

Security:

```text
GRE
 |
 +-- Encapsulation
 |
 +-- NO encryption
```

---

## GRE over IPsec

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

Security:

```text
GRE
 |
 +-- Encapsulation
 |
IPsec
 |
 +-- Encryption
 +-- Integrity
 +-- Authentication
```

---

# 🚀 20. Production Deployment Checklist

## Underlay

* [ ] WAN interface is operational
* [ ] Remote endpoint is reachable
* [ ] ISP routing is correct
* [ ] MTU/MSS implications have been evaluated
* [ ] No recursive routing exists

## GRE

* [ ] Local gateway is correct
* [ ] Remote gateway is correct
* [ ] GRE protocol 47 is supported through the path
* [ ] Tunnel interface exists
* [ ] Tunnel IP addressing is correct
* [ ] Remote LAN routes use GRE
* [ ] Return routes exist

## IPsec

* [ ] IKE version is appropriate
* [ ] Strong encryption is configured
* [ ] SHA-2 or stronger integrity is used where supported/appropriate
* [ ] Strong DH/ECDH group is selected
* [ ] Strong PSK/certificate authentication is used
* [ ] Phase 1 is UP
* [ ] Phase 2 is UP
* [ ] GRE traffic is correctly protected

## Firewall

* [ ] LAN → GRE policy exists
* [ ] GRE → LAN policy exists when required
* [ ] Only required services are allowed
* [ ] Source/destination objects are restricted
* [ ] NAT is disabled unless explicitly required
* [ ] Logging is enabled appropriately

## Routing

* [ ] Underlay routes use WAN
* [ ] Overlay routes use GRE
* [ ] Return routes exist
* [ ] No recursive route exists
* [ ] Routing table has expected entries

## Validation

* [ ] Tunnel endpoint is reachable
* [ ] GRE tunnel IP is reachable
* [ ] Remote LAN is reachable
* [ ] Reverse connectivity works
* [ ] Application traffic works
* [ ] Packet captures/logs confirm expected path

---

# 🧪 21. End-to-End Validation Checklist

Run tests in this order:

```text
[ ] 1. Test underlay endpoint
        ↓
[ ] 2. Verify IPsec Phase 1
        ↓
[ ] 3. Verify IPsec Phase 2
        ↓
[ ] 4. Verify IPsec interface
        ↓
[ ] 5. Test GRE tunnel IP
        ↓
[ ] 6. Verify routing table
        ↓
[ ] 7. Verify firewall policy
        ↓
[ ] 8. Test remote LAN
        ↓
[ ] 9. Test return traffic
        ↓
[ ] 10. Test application
```

### Golden Rule

> **Never troubleshoot the application first. Walk the packet path from underlay → IPsec → GRE → routing → firewall → application.**

---

# 📋 22. Quick Reference

| Requirement            | Design                             |
| ---------------------- | ---------------------------------- |
| Point-to-point overlay | GRE                                |
| GRE without encryption | GRE                                |
| GRE + encryption       | GRE over IPsec                     |
| GRE protocol           | **47**                             |
| GRE encryption         | **None**                           |
| GRE endpoint routing   | **Underlay/WAN**                   |
| Remote LAN routing     | **GRE interface**                  |
| Secure GRE transport   | **GRE over IPsec**                 |
| IPsec control plane    | IKE                                |
| IPsec data protection  | ESP/IPsec SA                       |
| Tunnel routing         | Static or dynamic routing over GRE |

---

# 🎯 23. NSE Exam Mental Picture

```text
                    GRE
                     |
             IP Protocol 47
                     |
              +------+------+
              |             |
          Underlay       Overlay
              |             |
             WAN           GRE
              |             |
          Endpoint      Remote LAN
```

For secure deployment:

```text
             GRE
              |
              v
            IPsec
              |
              v
         Encrypted WAN
              |
              v
            IPsec
              |
              v
             GRE
```

### Remember

```text
GRE
= Encapsulation

IPsec
= Security

Routing
= Reachability

Firewall
= Permission

Underlay
= Endpoint transport

Overlay
= GRE path
```

---

# 🏆 Final Key Takeaways

> **1. GRE is an overlay technology, not a security protocol.**

> **2. GRE uses IP protocol 47.**

> **3. The remote GRE endpoint must always be reachable through the underlay.**

> **4. Remote LAN prefixes must be routed through the GRE interface.**

> **5. Every routed tunnel requires a valid return path.**

> **6. GRE over IPsec combines GRE encapsulation with IPsec security.**

> **7. A tunnel being UP does not guarantee application connectivity.**

> **8. Always validate the complete packet path: Underlay → IPsec → GRE → Routing → Firewall → Application.**

> **9. Do not deploy legacy DES/MD5 configurations from old labs in modern production environments.**

> **10. The most important troubleshooting question is: *Where does the packet stop?***

---

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


---

**SheynShield — Engineering Secure Networks**

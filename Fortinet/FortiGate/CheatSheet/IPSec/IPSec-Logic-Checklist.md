# 🔐 IPSec / IKE / VPN Checklist — CCNP & FortiGate

> **SheynShield | Engineering Secure Networks**
>
> A practical **IPSec VPN, IKEv1, IKEv2, FortiGate and Cisco troubleshooting checklist** covering encryption, authentication, Security Associations, NAT-T, DPD, QCD, XAuth, ADVPN, GRE over IPSec, MTU/MSS and VPN troubleshooting.

---

## 📌 Table of Contents

* [CIA Triad](#-1-cia-triad)
* [IPSec Fundamentals](#-2-ipsec-fundamentals)
* [Encryption and Integrity](#-3-encryption-and-integrity)
* [Authentication and Diffie-Hellman](#-4-authentication-and-diffie-hellman)
* [Security Associations](#-5-security-associations-sa)
* [AH vs ESP](#-6-ah-vs-esp)
* [IKEv1 Checklist](#-7-ikev1-checklist)
* [IKEv1 Main Mode](#-8-ikev1-main-mode)
* [IKEv1 Aggressive Mode](#-9-ikev1-aggressive-mode)
* [IKEv1 Phase 2 / Quick Mode](#-10-ikev1-phase-2--quick-mode)
* [IKEv2 Checklist](#-11-ikev2-checklist)
* [IKEv1 vs IKEv2](#-12-ikev1-vs-ikev2)
* [Child SA](#-13-ikev2-child-sa)
* [EAP and Mode Config](#-14-eap-and-mode-config)
* [Site-to-Site VPN](#-15-site-to-site-ipsec-vpn)
* [GRE over IPSec](#-16-gre-over-ipsec)
* [Cisco IPSec Checklist](#-17-cisco-ipsec-configuration-checklist)
* [NAT Traversal](#-18-nat-traversal-nat-t)
* [Dead Peer Detection](#-19-dead-peer-detection-dpd)
* [Quick Crash Detection](#-20-quick-crash-detection-qcd)
* [IKE Fragmentation](#-21-ike-fragmentation)
* [XAuth](#-22-xauth)
* [FortiGate Network ID](#-23-fortigate-network-id)
* [ADVPN](#-24-advpn)
* [UDP Hole Punching](#-25-udp-hole-punching)
* [Mesh and Spoke-to-Spoke](#-26-mesh-and-spoke-to-spoke)
* [DHCP over VPN](#-27-dhcp-over-vpn)
* [FortiGate Local-In Policy](#-28-fortigate-local-in-policy)
* [MTU and MSS](#-29-mtu-and-mss)
* [FortiGate Troubleshooting Commands](#-30-fortigate-troubleshooting-commands)
* [Layered Troubleshooting](#-31-ipsec-troubleshooting-methodology)
* [Failure Mapping](#-32-common-ipsec-failure-mapping)
* [Security Hardening](#-33-ipsec-security-hardening)
* [Exam Quick Review](#-34-ccnp--fortigate-exam-quick-review)
* [Mental Models](#-35-ipsec-mental-models)
* [Final Checklist](#-36-final-ipsec-vpn-checklist)
* [SheynShield Resources](#-sheynshield-resources)

---

# 🔺 1. CIA Triad

* [ ] **Confidentiality** — prevent unauthorized information disclosure
* [ ] **Integrity** — detect unauthorized modification
* [ ] **Availability** — maintain authorized access to services

### Security Mapping

| Security Property | Main Mechanism               |
| ----------------- | ---------------------------- |
| Confidentiality   | Encryption                   |
| Integrity         | Hash / HMAC                  |
| Authentication    | PSK / Certificate            |
| Availability      | Redundancy / HA / Resilience |

### Remember

```text
CIA
 |
 +-- Confidentiality --> Encryption
 +-- Integrity       --> Hash / HMAC
 +-- Availability    --> Reliability / Redundancy
```

---

# 🔐 2. IPSec Fundamentals

* [ ] Understand that IPSec is a **framework**, not a single protocol
* [ ] Understand IKE
* [ ] Understand ESP
* [ ] Understand AH
* [ ] Understand Security Associations
* [ ] Understand SPI
* [ ] Understand anti-replay protection
* [ ] Understand key exchange
* [ ] Understand authentication
* [ ] Understand encryption
* [ ] Understand integrity

### IPSec Architecture

```text
IPSec
 |
 +-- IKE
 +-- AH
 +-- ESP
 +-- Security Associations
 +-- Authentication
 +-- Encryption
 +-- Integrity
 +-- Anti-Replay
 +-- Key Management
```

---

# 🔒 3. Encryption and Integrity

## Encryption

* [ ] Understand that encryption provides confidentiality
* [ ] Know DES
* [ ] Know 3DES
* [ ] Know AES
* [ ] Understand AES-128
* [ ] Understand AES-192
* [ ] Understand AES-256
* [ ] Recognize DES as obsolete
* [ ] Recognize 3DES as legacy
* [ ] Prefer AES for modern deployments

```text
DES      --> Legacy / obsolete
3DES     --> Legacy
AES      --> Modern
AES-128
AES-192
AES-256
```

## Integrity

* [ ] Understand MD5
* [ ] Understand SHA-1
* [ ] Understand SHA-256
* [ ] Understand SHA-512
* [ ] Understand HMAC
* [ ] Recognize MD5 as unsuitable for modern deployments
* [ ] Recognize SHA-1 as unsuitable for new deployments

```text
Encryption --> Confidentiality

Hash/HMAC --> Integrity
```

---

# 🔑 4. Authentication and Diffie-Hellman

## Authentication

* [ ] Understand Pre-Shared Key authentication
* [ ] Understand certificate authentication
* [ ] Understand RSA-based authentication
* [ ] Understand public/private key concepts
* [ ] Understand CA and certificate validation
* [ ] Use strong PSKs when PSK authentication is required
* [ ] Prefer certificates for scalable environments

```text
PSK
 |
 +-- Shared secret
 |
 +-- Both peers must know the same secret
```

## Diffie-Hellman

* [ ] Understand DH as a key-agreement mechanism
* [ ] Understand that the shared secret is not directly transmitted
* [ ] Know DH Group 1
* [ ] Know DH Group 2
* [ ] Know DH Group 5
* [ ] Know DH Group 14
* [ ] Avoid weak legacy DH groups
* [ ] Prefer strong DH groups or elliptic-curve groups where supported

```text
Peer A                     Peer B
  |                          |
  |---- DH public values --->|
  |<--- DH public values ----|
  |                          |
  +---- Shared Secret -------+
```

---

# 🧱 5. Security Associations (SA)

* [ ] Understand what a Security Association represents
* [ ] Understand encryption parameters
* [ ] Understand integrity parameters
* [ ] Understand authentication parameters
* [ ] Understand lifetime
* [ ] Understand SPI
* [ ] Understand traffic selectors
* [ ] Understand IKE SA
* [ ] Understand IPSec / Child SA

```text
Security Association
 |
 +-- Encryption
 +-- Integrity
 +-- Authentication
 +-- Lifetime
 +-- SPI
 +-- Traffic Selectors
```

## SPI

* [ ] Know that SPI = Security Parameter Index
* [ ] Understand SPI as an identifier for the applicable SA
* [ ] Understand INVALID_SPI troubleshooting

---

# 🛡️ 6. AH vs ESP

## AH

* [ ] Know AH = Authentication Header
* [ ] Know AH provides integrity
* [ ] Know AH provides authentication
* [ ] Know AH supports anti-replay protection
* [ ] Know AH does **not** provide encryption
* [ ] Know AH is rarely used in modern deployments
* [ ] Understand AH transport mode
* [ ] Understand AH tunnel mode

## ESP

* [ ] Know ESP = Encapsulating Security Payload
* [ ] Know ESP provides confidentiality
* [ ] Know ESP provides integrity
* [ ] Know ESP supports authentication
* [ ] Know ESP supports anti-replay protection
* [ ] Know ESP is the dominant IPSec protocol
* [ ] Know ESP protocol number = **50**
* [ ] Understand ESP transport mode
* [ ] Understand ESP tunnel mode

### Comparison

| Feature         |   AH |    ESP |
| --------------- | ---: | -----: |
| Encryption      |    ❌ |      ✅ |
| Confidentiality |    ❌ |      ✅ |
| Integrity       |    ✅ |      ✅ |
| Authentication  |    ✅ |      ✅ |
| Anti-Replay     |    ✅ |      ✅ |
| NAT-T           |    ❌ |      ✅ |
| Modern Usage    | Rare | Common |

### Remember

```text
AH  = Integrity + Authentication
ESP = Encryption + Integrity + Authentication
```

---

# 🔄 7. IKEv1 Checklist

* [ ] Understand IKEv1
* [ ] Understand Phase 1
* [ ] Understand Phase 2
* [ ] Understand Main Mode
* [ ] Understand Aggressive Mode
* [ ] Understand Quick Mode
* [ ] Understand IKE/ISAKMP SA
* [ ] Understand IPSec SA

```text
IKEv1
 |
 +-- Phase 1
 |    |
 |    +-- Main Mode
 |    +-- Aggressive Mode
 |
 +-- Phase 2
      |
      +-- Quick Mode
```

---

# 🔐 8. IKEv1 Main Mode

Main Mode uses **6 messages**.

* [ ] MM1
* [ ] MM2
* [ ] MM3
* [ ] MM4
* [ ] MM5
* [ ] MM6

### Message Groups

```text
MM1 + MM2
 |
 +-- Security Policy Negotiation

MM3 + MM4
 |
 +-- DH + Nonce Exchange

MM5 + MM6
 |
 +-- Identity + Authentication
```

## Main Mode Checklist

### MM1

* [ ] Initiator sends SA proposal
* [ ] Understand security policy negotiation

### MM2

* [ ] Responder selects compatible proposal

### MM3

* [ ] Initiator sends DH information
* [ ] Initiator sends nonce

### MM4

* [ ] Responder sends DH information
* [ ] Responder sends nonce

### MM5

* [ ] Initiator sends identity
* [ ] Initiator authenticates

### MM6

* [ ] Responder sends identity
* [ ] Responder authenticates

---

# 🚦 IKEv1 Main Mode States

* [ ] Understand `MM_WAIT_MSG2`
* [ ] Understand `MM_WAIT_MSG3`
* [ ] Understand `MM_WAIT_MSG4`
* [ ] Understand `MM_WAIT_MSG5`
* [ ] Understand `MM_WAIT_MSG6`
* [ ] Understand `MM_ACTIVE`

### Troubleshooting

| State          | Check                           |
| -------------- | ------------------------------- |
| `MM_WAIT_MSG2` | Reachability / proposal / peer  |
| `MM_WAIT_MSG3` | IKE policy / routing / response |
| `MM_WAIT_MSG4` | PSK / identity / tunnel-group   |
| `MM_WAIT_MSG5` | PSK / authentication            |
| `MM_WAIT_MSG6` | Final authentication            |
| `MM_ACTIVE`    | Phase 1 established             |

---

# ⚡ 9. IKEv1 Aggressive Mode

Aggressive Mode uses **3 messages**.

* [ ] Understand AM1
* [ ] Understand AM2
* [ ] Understand AM3
* [ ] Know that identity information is exposed earlier
* [ ] Understand why Main Mode is generally preferred

```text
AM1
 |
 +-- SA
 +-- KE
 +-- Nonce
 +-- ID

AM2
 |
 +-- SA
 +-- KE
 +-- Nonce
 +-- ID
 +-- Authentication

AM3
 |
 +-- Authentication
```

---

# 🔄 10. IKEv1 Phase 2 / Quick Mode

* [ ] Understand Phase 2
* [ ] Understand IPSec SA creation
* [ ] Understand Quick Mode
* [ ] Know Quick Mode normally uses 3 messages
* [ ] Verify ESP encryption
* [ ] Verify ESP integrity
* [ ] Verify traffic selectors
* [ ] Verify lifetime
* [ ] Verify PFS
* [ ] Verify DH group when PFS is enabled

```text
Quick Mode
 |
 +-- QM1
 +-- QM2
 +-- QM3
 |
 +-- IPSec SA
```

### Phase 1 vs Phase 2

|            | Phase 1         | Phase 2           |
| ---------- | --------------- | ----------------- |
| Purpose    | IKE SA          | IPSec SA          |
| Negotiates | IKE parameters  | ESP/AH parameters |
| IKEv1 Mode | Main/Aggressive | Quick Mode        |
| Result     | IKE SA          | IPSec SA          |

---

# 🔐 11. IKEv2 Checklist

* [ ] Understand IKEv2 architecture
* [ ] Understand `IKE_SA_INIT`
* [ ] Understand `IKE_AUTH`
* [ ] Understand Child SA
* [ ] Understand EAP
* [ ] Understand NAT detection
* [ ] Understand rekeying
* [ ] Understand liveness mechanisms
* [ ] Understand multiple Child SAs

## IKE_SA_INIT

* [ ] SA
* [ ] KE
* [ ] Nonce
* [ ] Cryptographic negotiation
* [ ] DH exchange

## IKE_AUTH

* [ ] ID
* [ ] AUTH
* [ ] Certificate where applicable
* [ ] SA
* [ ] Traffic Selectors
* [ ] NAT-related information where applicable

### Message Flow

```text
1. IKE_SA_INIT Request
2. IKE_SA_INIT Response

3. IKE_AUTH Request
4. IKE_AUTH Response
```

### Remember

```text
IKEv2 Initial Exchange = 4 messages
```

---

# 🔀 12. IKEv1 vs IKEv2

| Feature          | IKEv1         | IKEv2   |
| ---------------- | ------------- | ------- |
| Main Mode        | ✅             | ❌       |
| Aggressive Mode  | ✅             | ❌       |
| Quick Mode       | ✅             | ❌       |
| IKE_SA_INIT      | ❌             | ✅       |
| IKE_AUTH         | ❌             | ✅       |
| Initial messages | More          | Fewer   |
| EAP              | Limited       | Native  |
| Child SAs        | Less flexible | Native  |
| Mobility         | Limited       | Better  |
| Scalability      | Lower         | Better  |
| Architecture     | More complex  | Cleaner |

### Check

* [ ] Prefer IKEv2 for new deployments when supported
* [ ] Understand legacy IKEv1 environments
* [ ] Know Main Mode = 6 messages
* [ ] Know Aggressive Mode = 3 messages
* [ ] Know Quick Mode = 3 messages
* [ ] Know IKEv2 initial exchange = 4 messages

---

# 🧩 13. IKEv2 Child SA

* [ ] Understand Child SA
* [ ] Understand Child SA as the IPSec traffic-protection SA
* [ ] Understand multiple Child SAs under one IKE SA
* [ ] Understand ESP protection of user traffic
* [ ] Understand Child SA rekeying

```text
             IKE SA
                |
       +--------+--------+
       |        |        |
    Child SA  Child SA  Child SA
       |        |        |
      VPN1     VPN2     VPN3
```

---

# 👤 14. EAP and Mode Config

## EAP

* [ ] Know EAP = Extensible Authentication Protocol
* [ ] Understand user authentication
* [ ] Understand remote-access use cases
* [ ] Understand integration with authentication infrastructure

```text
IKEv2
 |
 +-- EAP
      |
      +-- User Authentication
      +-- Authentication Server
```

## Mode Config

* [ ] Understand dynamic client parameters
* [ ] Understand dynamic IP assignment
* [ ] Understand DNS assignment
* [ ] Understand route-related parameters
* [ ] Understand remote-access use cases

```text
Mode Config
 |
 +-- IP Address
 +-- DNS
 +-- Routes
 +-- Client Parameters
```

---

# 🌐 15. Site-to-Site IPSec VPN

* [ ] Identify local LAN
* [ ] Identify remote LAN
* [ ] Identify local VPN gateway
* [ ] Identify remote VPN gateway
* [ ] Verify public IP reachability
* [ ] Verify routing
* [ ] Verify IKE
* [ ] Verify IPSec
* [ ] Verify firewall policies
* [ ] Verify NAT exemption
* [ ] Verify traffic selectors

```text
LAN-A
 |
Router / Firewall A
 |
==== Internet ====
 |
Router / Firewall B
 |
LAN-B
```

---

# 🛰️ 16. GRE over IPSec

## GRE

* [ ] Understand GRE tunneling
* [ ] Understand multicast support
* [ ] Understand broadcast support
* [ ] Understand dynamic routing over GRE
* [ ] Understand tunnel interfaces

## IPSec

* [ ] Encrypt GRE traffic
* [ ] Protect GRE tunnel
* [ ] Verify IPSec SA

```text
Routing Protocol
       |
      GRE
       |
     IPSec
       |
   Internet
```

## Cisco GRE Keepalive

```cisco
interface Tunnel0
 keepalive 3 3
```

* [ ] Understand keepalive count
* [ ] Understand keepalive interval
* [ ] Verify tunnel state
* [ ] Verify underlying transport

---

# 🛠️ 17. Cisco IPSec Configuration Checklist

## IKEv1 Phase 1

```cisco
crypto isakmp policy 10
 encryption aes
 hash sha256
 authentication pre-share
 group 14
```

* [ ] Encryption matches peer
* [ ] Hash/integrity matches peer
* [ ] Authentication method matches
* [ ] DH group matches
* [ ] IKE version matches

## Pre-Shared Key

```cisco
crypto isakmp key vpnuser address 10.0.0.2
```

* [ ] Remote peer is correct
* [ ] PSK is correct
* [ ] Identity expectations match

## Transform Set

```cisco
crypto ipsec transform-set MYSET esp-aes esp-sha256-hmac
```

* [ ] ESP encryption matches
* [ ] ESP integrity matches
* [ ] Mode matches where required

## Interesting Traffic

```cisco
access-list 100 permit ip 192.168.101.0 0.0.0.255 192.168.102.0 0.0.0.255
```

* [ ] Local subnet is correct
* [ ] Remote subnet is correct
* [ ] ACL direction is correct
* [ ] Selectors match the peer

## Crypto Map

```cisco
crypto map MYMAP 10 ipsec-isakmp
 set peer 10.0.0.2
 set transform-set MYSET
 match address 100
```

* [ ] Peer configured
* [ ] Transform set configured
* [ ] Interesting traffic configured
* [ ] Crypto map sequence is correct

## Interface

```cisco
interface GigabitEthernet0/0
 crypto map MYMAP
```

* [ ] Crypto map applied to correct external interface

---

# 🧱 18. NAT Traversal (NAT-T)

* [ ] Understand why ESP is problematic through NAT
* [ ] Know ESP = IP protocol 50
* [ ] Know IKE = UDP 500
* [ ] Know NAT-T = UDP 4500
* [ ] Understand NAT detection
* [ ] Understand NAT-D payloads
* [ ] Understand transition to UDP 4500

```text
ESP
 |
 +-- Protocol 50

NAT detected
 |
 v
UDP 4500
 |
 v
ESP encapsulated in UDP
```

### Ports / Protocols

| Protocol |    Value |
| -------- | -------: |
| ESP      |    IP 50 |
| AH       |    IP 51 |
| IKE      |  UDP 500 |
| NAT-T    | UDP 4500 |

## NAT Detection

* [ ] Compare NAT detection values
* [ ] Determine whether NAT exists
* [ ] Verify UDP 500
* [ ] Verify UDP 4500
* [ ] Verify NAT device behavior
* [ ] Check firewall filtering

---

# 💓 19. Dead Peer Detection (DPD)

* [ ] Understand DPD
* [ ] Understand peer liveness
* [ ] Understand request/response behavior
* [ ] Understand retry interval
* [ ] Understand retry count
* [ ] Understand idle behavior
* [ ] Understand on-demand behavior

```text
Peer A
 |
 | Are you alive?
 v
Peer B
 |
 | ACK
 v
Peer A
```

## FortiGate Example

```fortigate
config vpn ipsec phase1-interface
    edit "link-1"
        set dpd-retryinterval 15
        set dpd-retrycount 3
    next
end
```

* [ ] Verify retry interval
* [ ] Verify retry count
* [ ] Avoid unnecessarily aggressive probing
* [ ] Consider scale when deploying thousands of peers

### Deployment Consideration

```text
Dial-up VPN
    --> Consider on-idle

Site-to-site
    --> Consider on-demand
```

> Validate behavior against the FortiOS release and topology.

---

# ⚡ 20. Quick Crash Detection (QCD)

* [ ] Understand QCD
* [ ] Understand fast peer failure detection
* [ ] Understand the relationship between QCD and liveness
* [ ] Understand IKE notifications
* [ ] Understand invalid SPI notifications
* [ ] Understand vendor-specific behavior where applicable

### Useful Notifications

```text
INVALID_IKE_SPI
INVALID_SPI
```

### FortiGate Example

```fortigate
config system settings
    set ike-quick-crash-detect enable
end
```

* [ ] Verify support for the FortiOS release
* [ ] Verify actual behavior before deployment

---

# 🧩 21. IKE Fragmentation

* [ ] Understand large IKE messages
* [ ] Understand certificate-related fragmentation
* [ ] Understand large authentication payloads
* [ ] Understand MTU-related negotiation problems
* [ ] Recognize retransmissions
* [ ] Recognize stuck IKE negotiation
* [ ] Check ISP path
* [ ] Check firewall
* [ ] Check NAT
* [ ] Check MTU
* [ ] Consider IKE fragmentation

## FortiGate IKEv1 Example

```fortigate
config vpn ipsec phase1-interface
    edit "link-1"
        set fragmentation enable
    next
end
```

## FortiGate IKEv2 Example

```fortigate
config vpn ipsec phase1-interface
    edit "link-1"
        set ike-version 2
        set fragmentation enable
        set fragmentation-mtu 500
    next
end
```

* [ ] Validate command syntax for the installed FortiOS release
* [ ] Validate MTU value against the actual path

---

# 👤 22. XAuth

* [ ] Know XAuth = Extended Authentication
* [ ] Know it is commonly associated with IKEv1
* [ ] Understand remote-access VPN
* [ ] Understand user authentication
* [ ] Understand XAuth server
* [ ] Understand XAuth client
* [ ] Understand PAP
* [ ] Understand CHAP
* [ ] Understand Auto
* [ ] Understand local-user authentication
* [ ] Understand LDAP integration
* [ ] Understand RADIUS integration
* [ ] Understand user groups

```text
Remote User
    |
    v
FortiGate
    |
    +-- XAuth
    |
    +-- User Group
    |
    +-- LDAP / RADIUS / Local
```

---

# 🆔 23. FortiGate Network ID

* [ ] Understand Network ID
* [ ] Understand multiple VPN relationships over the same public interface
* [ ] Understand VPN differentiation
* [ ] Understand segmentation use cases

Example:

```fortigate
config vpn ipsec phase1-interface
    edit "link-1"
        set network-id 2
    next
end
```

Concept:

```text
Public Interface
 |
 +-- Network ID 1
 +-- Network ID 2
 +-- Network ID 3
```

---

# 🌎 24. ADVPN

**ADVPN = Auto Discovery VPN**

* [ ] Understand hub-and-spoke topology
* [ ] Understand dynamic shortcuts
* [ ] Understand spoke-to-spoke tunnels
* [ ] Understand dynamic mesh
* [ ] Understand ADVPN with SD-WAN
* [ ] Understand route advertisement
* [ ] Understand shortcut discovery
* [ ] Understand exchange interface IP
* [ ] Understand mesh selectors

### Traditional Hub-and-Spoke

```text
Spoke A --> Hub --> Spoke B
```

### ADVPN Shortcut

```text
Spoke A <===========> Spoke B
        Direct VPN
```

## ADVPN + SD-WAN

* [ ] Understand dynamic path selection
* [ ] Understand dynamic shortcuts
* [ ] Understand route advertisement
* [ ] Understand large-scale VPN fabrics
* [ ] Understand hub-and-spoke optimization

---

# 🕳️ 25. UDP Hole Punching

* [ ] Understand NATed spokes
* [ ] Understand public mapped endpoints
* [ ] Understand UDP endpoint discovery
* [ ] Understand direct spoke-to-spoke connectivity
* [ ] Understand the role of the hub
* [ ] Verify UDP 4500 reachability

```text
Private Spoke
     |
    NAT
     |
     v
Public IP:UDP Port
     |
  Internet
     |
     v
Public IP:UDP Port
     |
    NAT
     |
Private Spoke
```

Example endpoint:

```text
55.1.1.2:64916
```

Interpretation:

```text
Public / NAT-mapped IP
+
UDP source port
```

---

# 🔗 26. Mesh and Spoke-to-Spoke

## Exchange Interface IP

* [ ] Understand hub-assisted endpoint exchange
* [ ] Understand spoke discovery
* [ ] Understand direct tunnel establishment

```text
          HUB
         /   \
   Spoke A   Spoke B
       \       /
        \     /
       Shortcut
```

## Mesh Selector

Example:

```fortigate
config vpn ipsec phase1-interface
    edit "link-1"
        set mesh-selector-type enable
    next
end
```

* [ ] Enable appropriate mesh behavior
* [ ] Verify Phase 2 selectors
* [ ] Verify auto-discovery configuration
* [ ] Verify exchange interface IP
* [ ] Verify route advertisement

---

# 🎯 Phase 2 Selectors

* [ ] Verify local selector
* [ ] Verify remote selector
* [ ] Verify proxy IDs
* [ ] Verify full-mesh requirements
* [ ] Verify ADVPN requirements
* [ ] Avoid unnecessarily restrictive selectors

```text
Restrictive Selector
       |
       v
Limited Connectivity
```

---

# 📡 27. DHCP over VPN

* [ ] Understand DHCP relay
* [ ] Understand DHCP over VPN scenarios
* [ ] Verify DHCP server reachability
* [ ] Verify relay configuration
* [ ] Verify routing
* [ ] Verify firewall policy
* [ ] Verify VPN selectors

```text
Client
  |
DHCP Relay
  |
IPSec VPN
  |
DHCP Server
```

---

# 🔥 28. FortiGate Local-In Policy

* [ ] Understand Local-In Policy
* [ ] Understand traffic destined to the FortiGate itself
* [ ] Distinguish Local-In from normal firewall policies
* [ ] Protect IKE
* [ ] Protect management services
* [ ] Restrict unwanted VPN sources
* [ ] Enable appropriate logging

```text
Internet
   |
   v
FortiGate
   |
   +-- Local-In Policy
          |
          +-- Allow
          +-- Deny
```

## Address Object

Example:

```text
Name: block-2
Type: Subnet
IP: 2.2.2.2/32
Interface: any
```

## Local-In Policy

```fortigate
config firewall local-in-policy
    edit 1
        set interface "port2"
        set srcaddr "all"
        set dstaddr "block-2"
        set service "ALL"
        set schedule "always"
        set action deny
        set status enable
    next
end
```

* [ ] Verify source
* [ ] Verify destination
* [ ] Verify service
* [ ] Verify interface
* [ ] Verify action
* [ ] Verify logging

## Logging

```fortigate
config log setting
    set local-in-allow enable
    set local-in-deny enable
    set local-in-br enable
end
```

---

# 📏 29. MTU and MSS

* [ ] Understand IPSec overhead
* [ ] Understand reduced effective MTU
* [ ] Check path MTU
* [ ] Check fragmentation
* [ ] Check TCP retransmissions
* [ ] Check application failures
* [ ] Consider TCP MSS adjustment

```text
Original MTU
     |
     +-- IPSec Overhead
     |
     v
Effective MTU
```

## FortiGate MSS Example

```fortigate
config firewall policy
    edit 1
        set tcp-mss-sender 1350
        set tcp-mss-receiver 1350
    next
end
```

* [ ] Calculate/test appropriate MSS
* [ ] Consider tunnel overhead
* [ ] Test large packets
* [ ] Test application behavior
* [ ] Avoid blindly choosing MSS values

---

# 🧪 30. FortiGate Troubleshooting Commands

## IKE Gateway

```bash
diagnose vpn ike gateway list
```

* [ ] Check IKE SA
* [ ] Check peer IP
* [ ] Check local IP
* [ ] Check proposal
* [ ] Check authentication
* [ ] Check gateway state

## IPSec Tunnel

```bash
diagnose vpn tunnel list
```

* [ ] Check tunnel state
* [ ] Check SPI
* [ ] Check SA
* [ ] Check proposal
* [ ] Check encapsulation
* [ ] Check packet counters

## Flush IKE State

```bash
diagnose vpn ike gateway flush
```

* [ ] Use when stale IKE state needs to be cleared
* [ ] Force tunnel renegotiation when appropriate

## IKE Debug

```bash
diagnose debug application ike -1
diagnose debug enable
```

Stop debugging:

```bash
diagnose debug disable
diagnose debug reset
```

* [ ] Enable debug
* [ ] Reproduce issue
* [ ] Read negotiation messages
* [ ] Disable debug after troubleshooting

---

# 🔎 31. IPSec Troubleshooting Methodology

Always troubleshoot from the bottom upward.

```text
Physical / ISP
      |
      v
Layer 2
      |
      v
Routing
      |
      v
UDP 500 / UDP 4500
      |
      v
IKE Phase 1
      |
      v
IKE Phase 2
      |
      v
IPSec / ESP
      |
      v
Firewall Policy
      |
      v
Routing / NAT
      |
      v
Application
```

## Layer 1 / ISP

* [ ] WAN interface is up
* [ ] ISP connectivity works
* [ ] Public IP is correct
* [ ] Default route exists
* [ ] Upstream connectivity works

## Layer 3

* [ ] Peer IP is reachable
* [ ] Routing is correct
* [ ] Return routing exists
* [ ] No asymmetric path problem

## UDP

* [ ] UDP 500 is permitted
* [ ] UDP 4500 is permitted
* [ ] NAT device is functioning
* [ ] ISP is not filtering required traffic

## IKE Phase 1

* [ ] IKE version
* [ ] Encryption
* [ ] Integrity
* [ ] DH group
* [ ] Authentication
* [ ] PSK
* [ ] Certificate
* [ ] Identity
* [ ] Lifetime
* [ ] Peer IP
* [ ] Local IP
* [ ] NAT-T

## IKE Phase 2

* [ ] ESP encryption
* [ ] ESP integrity
* [ ] PFS
* [ ] PFS DH group
* [ ] Traffic selectors
* [ ] Proxy IDs
* [ ] Lifetime
* [ ] Crypto map
* [ ] IPSec profile

## Data Plane

* [ ] Routing
* [ ] Firewall policies
* [ ] NAT exemption
* [ ] MTU
* [ ] MSS
* [ ] SPI
* [ ] ESP counters
* [ ] Selectors
* [ ] Symmetric routing

---

# 🚨 32. Common IPSec Failure Mapping

| Symptom                  | Investigation                    |
| ------------------------ | -------------------------------- |
| No IKE packets           | Routing / firewall / ISP         |
| No MM2                   | Peer reachability / UDP 500      |
| `MM_WAIT_MSG3`           | Policy / routing / peer response |
| `MM_WAIT_MSG4`           | PSK / identity                   |
| `MM_WAIT_MSG5`           | Authentication / PSK             |
| `MM_WAIT_MSG6`           | Final authentication             |
| Phase 1 UP, Phase 2 DOWN | Transform / selectors / PFS      |
| Tunnel UP, no traffic    | Routing / policy / selectors     |
| Intermittent tunnel      | DPD / NAT timeout                |
| Large packets fail       | MTU / fragmentation              |
| NAT VPN failure          | NAT-T / UDP 4500                 |
| High CPU with dial-up    | DPD / negotiation load           |
| Spoke-to-spoke failure   | ADVPN / shortcut / NAT           |
| `INVALID_IKE_SPI`        | Stale / mismatched IKE SA        |
| `INVALID_SPI`            | Stale / mismatched IPSec SA      |

---

# 🔐 33. IPSec Security Hardening

## Cryptography

* [ ] Prefer AES
* [ ] Prefer SHA-256 or stronger integrity
* [ ] Prefer strong DH groups
* [ ] Prefer IKEv2
* [ ] Use certificates where scalable authentication is required
* [ ] Use strong PSKs where PSK is required

## Avoid Legacy Cryptography

* [ ] Avoid DES
* [ ] Avoid 3DES for new deployments
* [ ] Avoid MD5
* [ ] Avoid SHA-1 for new deployments
* [ ] Avoid weak DH groups

## Operational Security

* [ ] Restrict IKE sources when possible
* [ ] Use Local-In Policy where appropriate
* [ ] Enable relevant VPN logging
* [ ] Monitor failed authentication
* [ ] Monitor tunnel state
* [ ] Monitor SA lifetime/rekey behavior
* [ ] Monitor DPD behavior
* [ ] Review exposed VPN services

---

# 🧠 34. CCNP & FortiGate Exam Quick Review

## CIA

```text
Confidentiality --> Encryption
Integrity       --> Hash / HMAC
Availability    --> Reliability
```

## IPSec

```text
AH
 |
 +-- Integrity
 +-- Authentication
 +-- Anti-Replay

ESP
 |
 +-- Encryption
 +-- Integrity
 +-- Authentication
 +-- Anti-Replay

IKE
 |
 +-- Negotiation
 +-- Authentication
 +-- Key Management
```

## IKEv1

```text
Phase 1
 |
 +-- Main Mode = 6
 +-- Aggressive = 3
 |
Phase 2
 |
 +-- Quick Mode = 3
```

## IKEv2

```text
IKE_SA_INIT
 |
 +-- SA
 +-- KE
 +-- Nonce

IKE_AUTH
 |
 +-- ID
 +-- AUTH
 +-- SA
 +-- TS
 +-- Certificate
```

## Protocol Numbers / Ports

```text
ESP      = IP 50
AH       = IP 51
IKE      = UDP 500
NAT-T    = UDP 4500
```

---

# 🧩 35. IPSec Mental Models

## Complete IPSec

```text
                 IPSec VPN
                     |
          +----------+----------+
          |                     |
        IKE                    ESP
          |                     |
   +------+-------+             |
   |              |             |
 Phase 1        Phase 2         |
   |              |             |
 IKE SA        Child/IPSec SA   |
   |              |             |
   +--------------+-------------+
                  |
                 SPI
                  |
            Protected Data
```

## IKEv1

```text
IKEv1
 |
 +-- Phase 1
 |    |
 |    +-- Main Mode
 |    +-- Aggressive Mode
 |
 +-- Phase 2
      |
      +-- Quick Mode
      |
      +-- IPSec SA
      |
      +-- ESP
```

## IKEv2

```text
IKEv2
 |
 +-- IKE_SA_INIT
 |    |
 |    +-- Crypto
 |    +-- DH
 |    +-- Nonce
 |
 +-- IKE_AUTH
      |
      +-- Identity
      +-- Authentication
      |
      +-- Child SA
           |
           +-- ESP
           +-- User Traffic
```

## Control Plane vs Data Plane

```text
                 VPN
                  |
          +-------+-------+
          |               |
      Control Plane    Data Plane
          |               |
         IKE             ESP
          |               |
     Negotiate          Protect
     Authenticate       Traffic
     Establish SA       Forward Data
```

---

# 🚀 36. Final IPSec VPN Checklist

## Fundamentals

* [ ] CIA understood
* [ ] Encryption understood
* [ ] Integrity understood
* [ ] Authentication understood
* [ ] DH understood
* [ ] SA understood
* [ ] SPI understood
* [ ] Anti-replay understood

## IPSec

* [ ] AH understood
* [ ] ESP understood
* [ ] Tunnel mode understood
* [ ] Transport mode understood
* [ ] ESP protocol 50 remembered
* [ ] AH protocol 51 remembered

## IKEv1

* [ ] Phase 1 understood
* [ ] Phase 2 understood
* [ ] Main Mode understood
* [ ] Aggressive Mode understood
* [ ] Quick Mode understood
* [ ] Six Main Mode messages remembered
* [ ] Three Aggressive Mode messages remembered
* [ ] Three Quick Mode messages remembered
* [ ] MM states understood

## IKEv2

* [ ] IKE_SA_INIT understood
* [ ] IKE_AUTH understood
* [ ] Child SA understood
* [ ] EAP understood
* [ ] Multiple Child SAs understood
* [ ] Four-message initial exchange remembered

## NAT / Liveness

* [ ] NAT-T understood
* [ ] UDP 500 understood
* [ ] UDP 4500 understood
* [ ] NAT detection understood
* [ ] DPD understood
* [ ] DPD timers understood
* [ ] QCD understood
* [ ] INVALID_IKE_SPI understood
* [ ] INVALID_SPI understood

## FortiGate

* [ ] `diagnose vpn tunnel list`
* [ ] `diagnose vpn ike gateway list`
* [ ] `diagnose vpn ike gateway flush`
* [ ] `diagnose debug application ike -1`
* [ ] `diagnose debug enable`
* [ ] `diagnose debug disable`
* [ ] `diagnose debug reset`
* [ ] Local-In Policy understood
* [ ] Network ID understood
* [ ] IKE fragmentation understood
* [ ] MSS adjustment understood

## Cisco

* [ ] IKE policy understood
* [ ] PSK understood
* [ ] Transform set understood
* [ ] Interesting traffic understood
* [ ] Crypto map understood
* [ ] Crypto map interface application understood
* [ ] GRE over IPSec understood
* [ ] GRE keepalive understood

## ADVPN

* [ ] Hub-and-spoke understood
* [ ] Dynamic shortcut understood
* [ ] Spoke-to-spoke understood
* [ ] Exchange interface IP understood
* [ ] Mesh selectors understood
* [ ] UDP hole punching understood
* [ ] NATed spokes understood
* [ ] ADVPN + SD-WAN understood

## Troubleshooting

* [ ] ISP checked
* [ ] Routing checked
* [ ] UDP 500 checked
* [ ] UDP 4500 checked
* [ ] IKE Phase 1 checked
* [ ] IKE Phase 2 checked
* [ ] Selectors checked
* [ ] PFS checked
* [ ] ESP checked
* [ ] SPI checked
* [ ] Firewall policy checked
* [ ] NAT exemption checked
* [ ] MTU checked
* [ ] MSS checked
* [ ] DPD checked
* [ ] ADVPN shortcut checked

---

# 📊 IPSec Quick Reference

| Topic                  | Key Point                                     |
| ---------------------- | --------------------------------------------- |
| CIA                    | Confidentiality, Integrity, Availability      |
| Encryption             | Confidentiality                               |
| Hash/HMAC              | Integrity / Authentication                    |
| AH                     | Integrity + Authentication                    |
| ESP                    | Encryption + Integrity + Authentication       |
| IKE                    | Negotiation + Authentication + Key Management |
| DH                     | Secure key agreement                          |
| SA                     | Security parameters/state                     |
| SPI                    | Identifies an SA                              |
| IKEv1 Phase 1          | IKE/ISAKMP SA                                 |
| IKEv1 Phase 2          | IPSec SA                                      |
| Main Mode              | 6 messages                                    |
| Aggressive Mode        | 3 messages                                    |
| Quick Mode             | 3 messages                                    |
| IKEv2                  | IKE_SA_INIT + IKE_AUTH                        |
| IKEv2 Initial Exchange | 4 messages                                    |
| Child SA               | IPSec SA under IKEv2                          |
| ESP                    | IP protocol 50                                |
| AH                     | IP protocol 51                                |
| IKE                    | UDP 500                                       |
| NAT-T                  | UDP 4500                                      |
| DPD                    | Peer liveness detection                       |
| QCD                    | Fast crash/failure detection                  |
| XAuth                  | Extended authentication                       |
| Mode Config            | Dynamic client parameters                     |
| ADVPN                  | Dynamic VPN shortcuts                         |
| UDP Hole Punching      | NAT traversal / direct connectivity           |
| MSS                    | Mitigates MTU-related TCP issues              |
| Local-In Policy        | Protects the FortiGate itself                 |
| Network ID             | Differentiates VPN behavior                   |
| Mesh Selector          | Supports dynamic spoke-to-spoke designs       |

---

# 🧠 One-Line Memory Map

```text
CIA
 ↓
IPSec
 ↓
IKE
 ↓
Phase 1
 ↓
IKE SA
 ↓
Phase 2 / Child SA
 ↓
SPI
 ↓
ESP
 ↓
Encrypted + Authenticated Traffic
```

---

# 🚀 ADVPN Mental Model

```text
                  HUB
                   |
        +----------+----------+
        |                     |
     Spoke A               Spoke B
        |                     |
        +---------+-----------+
                  |
              Shortcut
          / UDP Hole Punch
                  |
           Direct Tunnel
```

---

# 🔥 Final Architecture

```text
                       INTERNET
                           |
                 +---------+---------+
                 |                   |
              FortiGate A        FortiGate B
                 |                   |
               LAN-A               LAN-B
                 |                   |
                 +------ IPSec ------+
                         |
                        IKE
                         |
              +----------+----------+
              |                     |
           Phase 1                Phase 2
              |                     |
           IKE SA                Child SA
              |                     |
       +------+------+          +---+---+
       |             |          |       |
   Authentication   DH        ESP      SPI
       |                        |
       +-----------+------------+
                   |
              Protected Data
```

---

# 📚 Related Topics to Study

* [ ] FortiGate IPsec VPN
* [ ] FortiGate IKEv1
* [ ] FortiGate IKEv2
* [ ] FortiGate ADVPN
* [ ] FortiGate SD-WAN
* [ ] FortiGate VPN Troubleshooting
* [ ] Cisco Site-to-Site IPSec
* [ ] Cisco GRE over IPSec
* [ ] NAT Traversal
* [ ] Dead Peer Detection
* [ ] IKE Fragmentation
* [ ] IPSec MTU / MSS
* [ ] IPSec Security Hardening
* [ ] VPN High Availability
* [ ] Dynamic VPN
* [ ] Spoke-to-Spoke VPN
* [ ] Network Security Architecture

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

# 🏷️  Tags

`fortigate` `fortinet` `ipsec` `vpn` `ike` `ikev1` `ikev2` `advpn` `sdwan` `network-security` `cybersecurity` `ccnp` `nse4` `nse7` `networking` `firewall` `nat-t` `dpd` `xauth` `gre` `ipsec-troubleshooting`

---

# 🔎 Keywords
* FortiGate IPSec VPN
* FortiGate IPsec VPN troubleshooting
* IKEv1 vs IKEv2
* IPSec VPN checklist
* FortiGate VPN troubleshooting
* Fortinet IPSec configuration
* ADVPN configuration
* FortiGate ADVPN
* IPSec troubleshooting checklist
* IKEv1 Main Mode
* IKEv1 Aggressive Mode
* IKEv1 Quick Mode
* IKEv2 IKE_SA_INIT
* IKEv2 IKE_AUTH
* NAT-T UDP 4500
* IPSec ESP protocol 50
* FortiGate DPD
* FortiGate QCD
* FortiGate XAuth
* FortiGate IKE fragmentation
* FortiGate MTU MSS
* Cisco IPSec VPN
* GRE over IPSec
* Spoke-to-spoke VPN
* UDP hole punching
* IPSec Phase 1
* IPSec Phase 2
* Child SA
* Security Association
* SPI troubleshooting

---

# ⭐ SheynShield Checklist Status

| Category                  | Status |
| ------------------------- | ------ |
| CIA Triad                 | ⬜      |
| IPSec Fundamentals        | ⬜      |
| AH / ESP                  | ⬜      |
| IKEv1                     | ⬜      |
| IKEv2                     | ⬜      |
| NAT-T                     | ⬜      |
| DPD                       | ⬜      |
| QCD                       | ⬜      |
| XAuth                     | ⬜      |
| IKE Fragmentation         | ⬜      |
| FortiGate Troubleshooting | ⬜      |
| Cisco IPSec               | ⬜      |
| GRE over IPSec            | ⬜      |
| ADVPN                     | ⬜      |
| UDP Hole Punching         | ⬜      |
| Local-In Policy           | ⬜      |
| MTU / MSS                 | ⬜      |
| Security Hardening        | ⬜      |

---

> **SheynShield — Engineering Secure Networks**
>
> Practical Network Security • Fortinet • Cisco • Firewalls • VPN • Network Design

# 🔐 FortiGate IPsec VPN — GFM Cheat Sheet

> **Scope:** IPsec concepts, CIA, IKEv1/v2, Phase 1/2, NAT-T, DPD, QCD, fragmentation, XAuth, ADVPN, mesh, local-in protection, troubleshooting and Cisco examples.

---

# 1. IPsec Security Concept

## CIA Triad

### Confidentiality

- Ensures information is accessible only to authorized parties.
- Protects data:
  - At rest
  - In transit
  - In process
- Example:

```text
X1 --------------------> X2

Only X1 and X2 should be able to understand the information.
Unauthorized users must not be able to read it.
```

### Integrity

- Ensures data is not modified during transmission or processing.
- Receiver should see the same packet/data sent by the sender.
- Provides detection of:
  - Modification
  - Manipulation
  - Corruption

```text
X1 ---- Original Packet ----> X2

X1 == X2
      |
      +-- If different -> Integrity violation
```

### Availability

- Authorized users must have access to information and services when required.
- Important concepts:

  - Acceptable performance
  - Fault tolerance
  - Prevention of data loss
  - Prevention of destruction
  - Reliable backups
  - Redundancy

> **CIA = Base of trust for a secure connection**

---

# 2. IPsec Cryptographic Components

IPsec normally combines several security mechanisms.

```text
Encryption
    +
Integrity / Authentication
    +
Key Exchange
    +
Security Association
    =
Secure IPsec Tunnel
```

---

## 2.1 Encryption

Common encryption algorithms:

- DES
- 3DES
- AES

> Encryption provides confidentiality.

```text
Plain Data
    |
    v
Encryption
    |
    v
Encrypted Data
```

---

## 2.2 Integrity / Hash

Common algorithms:

- MD5
- SHA

SHA family examples:

- SHA-1 / 160-bit
- SHA-224
- SHA-256
- SHA-384
- SHA-512

> Integrity mechanisms are used to detect data manipulation.

Conceptually:

```text
Data
  |
  +----> Hash
  |
  v
Authentication / Integrity Verification
```

> **Note:** Modern deployments should avoid obsolete algorithms such as MD5, SHA-1, DES and 3DES when stronger options are available.

---

## 2.3 Authentication

Authentication verifies the identity of the peer.

Common methods:

### Pre-Shared Key

```text
Peer A
   |
   | Shared Secret
   |
Peer B
```

Both peers know the same secret.

### RSA / Certificates

Uses asymmetric cryptography:

```text
Private Key + Public Key
```

Certificate-based authentication is generally preferred for scalable environments.

---

# 3. Diffie-Hellman

DH is used to establish shared key material without directly sending the final secret across the network.

Examples:

- DH Group 1 → 768-bit
- DH Group 2 → 1024-bit
- DH Group 5 → 1536-bit
- DH Group 14 → 2048-bit
- Higher groups are also available depending on FortiOS/version.

```text
Peer A                         Peer B
  |                              |
  |------ DH Parameters -------->|
  |<----- DH Parameters ---------|
  |                              |
  +---- Shared Secret ----------+
```

> Both peers must use compatible DH groups.

---

# 4. IPsec Protocols

## AH — Authentication Header

AH provides authentication/integrity for IP packets but does **not** provide encryption.

### AH Transport Mode

```text
+----------------+
| IP Header      |
+----------------+
| AH Header      |
+----------------+
| TCP            |
+----------------+
| Data           |
+----------------+
```

### AH Tunnel Mode

```text
+------------------------+
| New IP Header          |
+------------------------+
| AH Header              |
+------------------------+
| Original IP Header     |
+------------------------+
| TCP                    |
+------------------------+
| Data                   |
+------------------------+
```

> AH is rarely used in modern deployments because NAT compatibility and encryption requirements generally favor ESP.

---

# 5. ESP — Encapsulating Security Payload

ESP can provide:

- Confidentiality
- Integrity
- Authentication
- Anti-replay protection

## ESP Transport Mode

```text
+----------------+
| Original IP    |
+----------------+
| ESP Header     |
+----------------+
| TCP*           |
+----------------+
| Data*          |
+----------------+
| ESP Trailer*   |
+----------------+
| ESP Auth       |
+----------------+
```

### ESP Encryption

```text
ESP Trailer + Data
        |
        v
    Encrypted
```

### ESP Authentication

Conceptually covers:

```text
ESP Header + Encrypted Payload
```

### Protocol

```text
ESP = IP Protocol 50
```

---

## ESP Tunnel Mode

Most common mode for site-to-site VPN.

```text
+------------------------+
| New External IP Header |
+------------------------+
| ESP Header             |
+------------------------+
| Original IP Header*    |
+------------------------+
| TCP*                   |
+------------------------+
| Data*                  |
+------------------------+
| ESP Trailer*           |
+------------------------+
| ESP Authentication     |
+------------------------+
```

### ESP Encryption

```text
Original IP Header
      +
TCP
      +
Data
      +
ESP Trailer
      |
      v
Encrypted
```

### ESP Authentication

```text
ESP Header
     +
Encrypted Payload
```

### Protocol

```text
ESP = IP Protocol 50
```

---

# 6. ESP + AH

Possible combination:

```text
ESP + AH
```

However, ESP with strong encryption and integrity is normally sufficient for modern deployments.

---

# 7. IPsec Tunnel Hierarchy

Think about IPsec in layers:

```text
IPsec
 |
 +-- IKE
 |    |
 |    +-- IKE SA
 |         |
 |         +-- Authentication
 |         +-- Key Exchange
 |         +-- Crypto Negotiation
 |
 +-- IPsec SA
      |
      +-- SPI
      +-- Encryption
      +-- Integrity
      +-- Traffic Selectors
```

---

# 8. IKEv1 vs IKEv2

| Feature | IKEv1 | IKEv2 |
|---|---|---|
| Complexity | Higher | Lower |
| Initial exchange | Main/Aggressive | IKE_SA_INIT + IKE_AUTH |
| Main mode | Yes | No |
| Aggressive mode | Yes | No |
| NAT-T | Supported | Supported |
| EAP | Limited/implementation dependent | Native support |
| Exchange efficiency | Lower | Higher |
| Child SAs | Limited compared with IKEv2 | Multiple Child SAs |
| Recommended | Legacy compatibility | Preferred |

---

# 9. IKEv1

## Phase 1

Purpose:

> Build the IKE/ISAKMP control channel.

Negotiates:

- Encryption
- Integrity
- Authentication
- DH group
- Peer identity

```text
Phase 1
   |
   +-- IKE Policy
   +-- Encryption
   +-- Integrity
   +-- DH
   +-- Authentication
   |
   v
IKE / ISAKMP SA
```

---

# 10. IKEv1 Main Mode

Main Mode uses **6 messages**.

```text
Initiator                         Responder

MM1  -------------------------->

      <-------------------------- MM2

MM3  -------------------------->

      <-------------------------- MM4

MM5  -------------------------->

      <-------------------------- MM6
```

---

## MM1

Negotiation of security parameters.

Contains concepts such as:

- SA
- Encryption
- Integrity
- DH
- Vendor ID

---

## MM2

Responder sends the accepted security parameters.

```text
SA
Vendor ID
```

---

## MM3

Key exchange / nonce information.

```text
KE
Nonce
Vendor ID
```

---

## MM4

Responder sends:

```text
KE
Nonce
Vendor ID
```

The shared key material is established.

---

## MM5

Encrypted identity/authentication.

```text
ID
Authentication
Certificate
Certificate Request
```

---

## MM6

Responder authentication.

```text
ID
Authentication
Certificate
```

---

## Main Mode Security Sequence

```text
MM1 + MM2
    |
    +-- Crypto negotiation
    |   Unencrypted / unauthenticated
    |
MM3 + MM4
    |
    +-- DH / Key exchange
    |
MM5 + MM6
    |
    +-- Identity / Authentication
    |   Encrypted
    |
    v
IKE SA Established
```

---

# 11. IKEv1 State Troubleshooting

## mm_wait_msg2

Initiator sent:

```text
Encryption
Hash / Integrity
DH
IKE Policy
```

and is waiting for the responder.

---

## mm_wait_msg3

Responder is waiting for the next message.

Possible issue:

```text
No return route
Incorrect peer configuration
IKE policy mismatch
```

---

## mm_wait_msg4

Initiator sent authentication information and waits for the peer.

Common causes:

```text
Missing PSK
Wrong PSK
Missing tunnel group / identity
```

---

## mm_wait_msg5

Responder has not completed PSK validation.

Possible cause:

```text
PSK mismatch
```

---

## mm_wait_msg6

Initiator is waiting for final authentication confirmation.

---

## mm_active

IKE Phase 1 is established.

---

# 12. IKEv1 Aggressive Mode

Aggressive Mode uses 3 messages.

```text
Initiator                         Responder

AM1  -------------------------->

      <-------------------------- AM2

AM3  -------------------------->
```

### AM1

Contains:

- ISAKMP header
- SA
- Key exchange
- Nonce
- Initiator ID

### AM2

Contains:

- ISAKMP header
- SA
- Key exchange
- Nonce
- Responder ID
- Responder authentication/hash

### AM3

Contains:

- Hash
- Authentication

> Aggressive Mode exposes more identity information during negotiation and is generally less desirable than Main Mode/IKEv2.

---

# 13. IKEv1 Phase 2

Purpose:

> Establish the IPsec data-plane SAs.

Also called:

```text
Quick Mode
```

### Quick Mode

```text
QM1
 |
 +-- Proposal / peer information
 |
QM2
 |
 +-- SA approval
 |
QM3
 |
 +-- Final confirmation
```

Creates unidirectional IPsec SAs.

```text
SA A -> B
SA B -> A
```

---

# 14. IKEv2

IKEv2 is simpler and more efficient than IKEv1.

The initial exchange uses:

```text
IKE_SA_INIT
IKE_AUTH
```

---

# 15. IKEv2 Phase 1 — IKE_SA_INIT

### Initiator

```text
IKE_SA_INIT Request
```

Contains:

- SA
- KE
- Nonce
- Vendor ID

### Responder

```text
IKE_SA_INIT Response
```

Contains:

- SA
- KE
- Nonce
- Vendor ID

This combines:

```text
Crypto negotiation
+
DH key exchange
```

---

# 16. IKEv2 Phase 2 — IKE_AUTH

### Initiator

```text
IKE_AUTH Request
```

Contains:

- ID
- AUTH
- Certificate
- SA
- Traffic Selectors
- NAT detection
- SPI

### Responder

```text
IKE_AUTH Response
```

Contains:

- ID
- AUTH
- Certificate
- SA
- Traffic Selectors
- NAT detection
- SPI

This provides:

```text
Identity
+
Authentication
+
Child SA creation
```

---

# 17. IKEv2 Message Flow

```text
Message 1
IKE_SA_INIT Request
       +
IKE_SA_INIT Response

Message 2
IKE_AUTH Request
       +
IKE_AUTH Response

Message 3+
CREATE_CHILD_SA / INFORMATIONAL
```

Conceptually:

```text
IKEv1:
MM1
MM2
MM3
MM4
MM5
MM6
   |
   v
Quick Mode

IKEv2:
IKE_SA_INIT
   |
IKE_AUTH
   |
Child SA
```

---

# 18. IKEv2 Features

## EAP

Extensible Authentication Protocol.

Useful for authentication using:

- Certificates
- External authentication systems
- Remote-access scenarios

---

## Lower Bandwidth

IKEv2 requires fewer exchanges than IKEv1.

---

## NAT Traversal

Built-in NAT-T support.

---

## Built-in Keepalive / Liveness

IKEv2 has stronger built-in mechanisms for peer liveness.

---

# 19. Child SA

IKEv2 can create multiple Child SAs under one IKE SA.

```text
IKE SA
 |
 +-- Child SA 1
 |
 +-- Child SA 2
 |
 +-- Child SA 3
 |
 +-- Child SA N
```

Useful for:

- Multiple traffic selectors
- Multiple IPsec policies
- Rekeying
- Dynamic VPN architectures

---

# 20. Cisco IPsec — Classic Crypto Map

## Phase 1

```cisco
crypto isakmp policy 10
 encryption aes
 hash sha256
 authentication pre-share
 group 14
```

---

## Pre-Shared Key

```cisco
crypto isakmp key vpnuser address 10.0.0.2
```

---

## Phase 2 Transform Set

```cisco
crypto ipsec transform-set myset esp-aes esp-sha256-hmac
```

---

## Interesting Traffic

```cisco
access-list 100 permit ip \
192.168.101.0 0.0.0.255 \
192.168.102.0 0.0.0.255
```

---

## Crypto Map

```cisco
crypto map mymap 10 ipsec-isakmp
 set peer 10.0.0.2
 set transform-set myset
 match address 100
```

---

## Apply Crypto Map

```cisco
interface GigabitEthernet0/0
 ip address 192.168.102.1 255.255.255.0
 crypto map mymap
```

---

# 21. Cisco Route-Based IPsec Example

## Keyring

```cisco
crypto keyring preshare
 pre-shared-key address 11.12.13.2 key secret
```

## ISAKMP Policy

```cisco
crypto isakmp policy 1
 encr 3des
 authentication pre-share
 group 5
```

## ISAKMP Profile

```cisco
crypto isakmp profile preshare
 keyring preshare
 match identity address 11.12.13.2 255.255.255.255
```

## Transform Set

```cisco
crypto ipsec transform-set aes-sha1 esp-aes esp-sha-hmac
 mode tunnel
```

## IPsec Profile

```cisco
crypto ipsec profile ipsec-preshare
 set transform-set aes-sha1
 set isakmp-profile preshare
```

## Tunnel

```cisco
interface Tunnel111
 ip address 192.168.111.1 255.255.255.0
 tunnel source 11.12.13.1
 tunnel mode ipsec ipv4
 tunnel destination 11.12.13.2
 tunnel protection ipsec profile ipsec-preshare
```

## Route

```cisco
ip route 192.168.102.0 255.255.255.0 Tunnel111
```

---

# 22. FortiGate Phase 1

Important Phase 1 concepts:

- Security Association
- Encryption
- Authentication
- DH
- Auto negotiation
- Negotiation timeout
- NAT Traversal
- DPD
- Mode Config
- XAuth
- Device Creation
- Network ID
- Auto Discovery
- Exchange Interface IP

---

# 23. NAT Traversal — NAT-T

NAT-T encapsulates IPsec traffic inside UDP.

Common ports:

```text
UDP/500
UDP/4500
```

Normally:

```text
IKE
UDP 500
```

When NAT is detected:

```text
IPsec / IKE
UDP 4500
```

---

# 24. NAT Detection

Conceptually:

```text
1. Peer A:500
       |
       v
2. Peer B:500
       |
       v
3. NAT-D payload
       |
       v
4. Hash comparison
       |
       +-- Same hash -> No NAT
       |
       +-- Different -> NAT detected
```

Example:

```text
192.168.1.1:500
        |
        v
      Hash
```

Responder calculates the expected value.

If values differ:

```text
NAT exists
```

---

# 25. NAT-T Modes

## Enable

Preferred in most environments.

```text
NAT-T enabled
```

## Force

Forces NAT-T behavior.

Useful when NAT-T must be used regardless of normal detection.

---

# 26. NAT Keepalive

NAT devices may remove idle UDP mappings.

NAT keepalive sends periodic traffic to keep the mapping alive.

```text
Peer A
  |
  | Keepalive
  v
NAT Device
  |
  v
Peer B
```

---

# 27. Dead Peer Detection — DPD

DPD detects whether the remote peer is alive.

Concept:

```text
Peer A
  |
  | R-U-THERE
  v
Peer B
  |
  | R-U-THERE-ACK
  v
Peer A
```

If the peer does not respond:

```text
IKE / IPsec SA
       |
       v
Considered dead
       |
       v
Tunnel cleanup / renegotiation
```

---

# 28. DPD Modes

## on-idle

Useful for:

```text
Dial-up VPN
Large-scale VPN environments
```

Only checks when the connection is idle.

---

## on-demand

Useful for:

```text
Site-to-site VPN
```

DPD behavior is triggered based on traffic / liveness requirements.

---

# 29. FortiGate DPD Configuration

```bash
config vpn ipsec phase1-interface
    edit "link-1"
        set dpd-retryinterval 15
        set dpd-retrycount 3
    next
end
```

Conceptually:

```text
Retry interval = 15 sec
Retry count    = 3
```

---

# 30. DPD Troubleshooting

```bash
diagnose vpn ike gateway list
```

Useful for checking:

- IKE SA
- HMAC
- Authentication
- Proposals
- Peer address
- Tunnel state

> If DPD ACKs are not received, the peer may be considered unreachable and the IKE/IPsec session may be terminated.

---

# 31. IKEv2 Reauthentication

IKEv2 can periodically reauthenticate the peer.

```bash
config vpn ipsec phase1-interface
    edit "link-1"
        set reauth enable
    next
end
```

The Phase 1 key lifetime controls when reauthentication/rekey behavior occurs depending on configuration.

---

# 32. Quick Crash Detection — QCD

QCD provides faster detection of peer failures.

Conceptually similar to DPD:

```text
Peer A
  |
  | Liveness information
  v
Peer B
```

IKEv2 provides built-in mechanisms for liveness and invalid SPI handling.

---

## Enable QCD

```bash
config system setting
    set ike-quick-crash-detect enable
end
```

> IKEv1 can use vendor-specific QCD extensions; behavior depends on FortiOS/version and configuration.

---

# 33. IKE Fragmentation

IKE packets can become large because of:

- Certificates
- Large proposals
- Authentication payloads
- Multiple attributes

Fragmentation helps prevent problems with:

```text
UDP packet size
ISP MTU
Intermediate network devices
```

---

# 34. IKEv1 Fragmentation

```bash
config vpn ipsec phase1-interface
    edit "link-1"
        set fragmentation enable
    next
end
```

> Default behavior may depend on FortiOS version/configuration.

---

# 35. IKEv2 Fragmentation

```bash
config vpn ipsec phase1-interface
    edit "link-1"
        set ike-version 2
        set fragmentation enable
        set fragmentation-mtu 500
    next
end
```

---

# 36. Embryonic Limit

Protects IPsec/IKE against excessive connection attempts.

Useful against:

```text
DoS
DDoS
IKE connection exhaustion
```

Example:

```bash
config system setting
    set embryonic-limit 50
end
```

> Supported limits depend on FortiGate model and FortiOS version.

---

# 37. Mode Config

Mode Config can assign configuration parameters to dynamic/dial-up clients.

Examples:

- IP address
- DNS
- Routes
- Other client parameters

Useful for:

```text
Dial-up VPN
Remote Access
Dynamic peers
```

---

# 38. Device Creation

For dynamic/dial-up IPsec peers, FortiGate can dynamically create interfaces/objects associated with connected peers.

Conceptually:

```text
Dial-up Peer
     |
     v
IPsec Connection
     |
     v
Dynamic Interface
     |
     +-- IP
     +-- Tunnel ID
     +-- Routing
     +-- Policies
```

---

# 39. XAuth

XAuth provides additional user authentication for IPsec.

Possible modes:

```text
PAP
CHAP
Auto
Client
```

---

## PAP

```text
FortiGate <---- authentication negotiation ----> Client
```

---

## CHAP

Authentication negotiation with the authentication server.

---

## Auto Server

Can combine authentication behavior.

> In the described auto-server configuration, only one user group is assigned directly, while multiple user containers may exist on the authentication backend.

---

# 40. XAuth Client Mode

Useful when FortiGate acts as the client toward a remote VPN gateway.

```text
FortiGate Client
       |
       | XAuth
       v
Remote VPN Gateway
```

---

# 41. Network ID

Network ID can distinguish multiple IPsec behaviors/segments over the same public interface.

```bash
config vpn ipsec phase1-interface
    edit "link-1"
        set network-id 2
    next
end
```

Concept:

```text
Same Physical Interface
        |
        +-- Network-ID 1
        |
        +-- Network-ID 2
        |
        +-- Network-ID 3
```

---

# 42. Auto Discovery VPN / ADVPN

ADVPN is useful for large hub-and-spoke networks.

It can help establish:

```text
Hub <----> Spoke
Hub <----> Spoke
Spoke <----> Spoke
```

Instead of forcing all spoke-to-spoke traffic through the hub.

---

# 43. ADVPN + SD-WAN

Common architecture:

```text
                HUB
                 |
        +--------+--------+
        |                 |
      Spoke1            Spoke2
        |                 |
        +------SD-WAN-----+
```

Useful for:

- Hub-and-spoke
- Dynamic spoke-to-spoke connectivity
- Large-scale VPN deployments
- SD-WAN overlays

---

# 44. Auto Discovery Sender / Receiver

Used when dynamic VPN peers need discovery information.

Useful for:

```text
ADVPN
Mesh
Dynamic routing over IPsec
Spoke-to-spoke shortcuts
```

If routing protocols need to advertise routes/hello information through the IPsec overlay, auto-discovery mechanisms may be required depending on topology.

---

# 45. Exchange Interface IP

Useful in hub-and-spoke scenarios when spokes need to establish direct logical connectivity.

Concept:

```text
Spoke A
   |
   +----------+
              |
             HUB
              |
   +----------+
   |
Spoke B

After shortcut:

Spoke A <--------------> Spoke B
```

---

# 46. Mesh Selector

Can help build full-mesh behavior with dynamic IPsec.

```bash
config vpn ipsec phase1-interface
    edit "link-1"
        set mesh-selector-type enable
    next
end
```

Typically used together with:

```text
Auto Discovery
+
Exchange Interface IP
```

---

# 47. Aggregate IPsec Interfaces

IPsec interfaces can participate in aggregation in supported configurations.

Concept:

```text
IPsec-1 ----\
             +---- Aggregate ---- Network
IPsec-2 ----/
```

Provides redundancy / multiple members where supported.

---

# 48. IPsec Dial-up Address Release

A released dynamic IPsec address may remain reserved for a period.

Example:

```bash
config vpn ipsec phase1-interface
    edit "link-1"
        set ip-delay-interval 200
    next
end
```

To flush IKE gateways:

```bash
diagnose vpn ike gateway flush
```

---

# 49. IPsec MSS / MTU

IPsec adds overhead.

This can cause:

```text
Fragmentation
PMTUD problems
Application issues
TCP retransmissions
```

One approach is to adjust TCP MSS.

```bash
config firewall policy
    edit 1
        set tcp-mss-sender 1350
        set tcp-mss-receiver 1350
    next
end
```

---

# 50. Route Advertisement for Dial-up IPsec

Dynamic/dial-up peers may require additional routes.

Concept:

```text
Dial-up Peer
     |
     v
IPsec Tunnel
     |
     +-- Dynamic Route
     |
     +-- Remote Network
```

Mode Config and dynamic routing can be useful when advertising multiple services/routes across the fabric.

---

# 51. UDP Hole Punching

Useful for spoke-to-spoke connections when peers are behind NAT.

Concept:

```text
Spoke A
  |
 NAT A
  |
  +---------------- Internet ----------------+
                                             |
                                            NAT B
                                             |
                                           Spoke B
```

The hub can help the spokes discover each other's reachable NAT mappings.

---

## Example Debug

```bash
diagnose debug enable
diagnose debug application ike -1
```

Example messages:

```text
SHORTCUT-OFFER
SHORTCUT-QUERY
SHORTCUT-REPLY
NAT hole punching
dynamic tunnel
```

Example:

```text
SHORTCUT-OFFER
10.1.100.11 -> 192.168.4.33

SHORTCUT-QUERY

NAT hole punching to peer
55.1.1.2:64916
```

The external UDP mapping:

```text
55.1.1.2:64916
```

represents the NAT-created reachable UDP endpoint.

---

# 52. Spoke-to-Spoke Shortcut Verification

```bash
diagnose vpn ike gateway list
```

Look for:

```text
addr : 12.23.34.1:4500 -> 22.33.44.1:4500
```

A direct spoke-to-spoke UDP/4500 session indicates the shortcut path.

---

# 53. Local-In Policy — Block IKE / ESP

Local-in policies protect traffic destined **to the FortiGate itself**.

Useful for blocking unwanted:

```text
IKE
ESP
Management
Other local services
```

Architecture:

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

---

# 54. Local-In Policy Example

Create an address object:

```text
Name: block-2
Type: Subnet
IP: 2.2.2.2/32
Interface: any
```

Then:

```bash
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

> Use the actual object names available in your FortiGate configuration.

---

# 55. Local-In Logging

```bash
config log setting
    set local-in-allow enable
    set local-in-deny enable
    set local-in-br enable
end
```

---

# 56. Debug IKE Port 500

```bash
diagnose debug flow filter dport 500
diagnose debug flow trace start 10
diagnose debug enable
```

Useful for checking whether IKE traffic reaches the FortiGate and how it is handled.

---

# 57. Change IKE Port

```bash
config system setting
    set ike-port 500
end
```

> Ensure the selected port is allowed through upstream firewalls/ACLs and compatible with the peer.

---

# 58. IPsec Troubleshooting Commands

## Tunnel Information

```bash
diagnose vpn tunnel list
```

Useful for:

- Tunnel state
- IPsec SA
- SPI
- Proposals
- Source/destination
- Tunnel parameters

---

## IKE Gateway

```bash
diagnose vpn ike gateway list
```

Useful for:

- IKE SA
- Peer address
- Authentication
- Proposal
- HMAC
- Negotiation state

---

## IKE Debug

```bash
diagnose debug enable
diagnose debug application ike -1
```

Stop debugging:

```bash
diagnose debug disable
diagnose debug reset
```

---

# 59. IPsec Troubleshooting Flow

```text
                    IPsec Problem
                         |
                         v
                Is IKE traffic arriving?
                         |
                         +---- NO ----> Routing / ACL / ISP
                         |
                        YES
                         |
                         v
                  Phase 1 established?
                         |
              +----------+----------+
              |                     |
             NO                    YES
              |                     |
              v                     v
      Check Phase 1            Phase 2?
      - Encryption                 |
      - Integrity                  +---- NO
      - DH                         |
      - Authentication             v
      - PSK                    Check:
      - NAT-T                    - Proposal
      - Identity                 - PFS/DH
      - Local-in                 - Selectors
                                 - Routes
                                 - Policy
                                     |
                                     v
                              Traffic passing?
                                     |
                              +------+------+
                              |             |
                             NO            YES
                              |             |
                              v             v
                         MTU/MSS/       Tunnel OK
                         policy/route
                         /NAT/selector
```

---

# 60. Important IPsec Ports / Protocols

| Protocol / Port | Purpose |
|---|---|
| UDP/500 | IKE |
| UDP/4500 | NAT-T / IPsec over UDP |
| ESP / IP 50 | Encapsulating Security Payload |
| AH / IP 51 | Authentication Header |

---

# 61. IPsec Security Association

An SA contains negotiated security parameters.

Conceptually:

```text
IPsec SA
 |
 +-- SPI
 +-- Encryption Algorithm
 +-- Integrity Algorithm
 +-- Keys
 +-- Lifetime
 +-- Traffic Selectors
 +-- Direction
```

Normally there are separate inbound/outbound SAs:

```text
A -> B
B -> A
```

---

# 62. SPI

SPI = Security Parameters Index.

Used to identify the security association.

```text
Packet
 |
 +-- SPI
       |
       v
Find matching IPsec SA
       |
       v
Decrypt / Authenticate
```

---

# 63. Phase 1 vs Phase 2

| | Phase 1 | Phase 2 |
|---|---|---|
| Purpose | Build control channel | Build data-plane SA |
| Main object | IKE SA | IPsec SA / Child SA |
| Negotiates | IKE crypto/auth/DH | IPsec crypto/selectors |
| Authentication | Yes | Depends on established IKE SA |
| Traffic | Control | User/data traffic |

```text
Phase 1
   |
   v
IKE SA
   |
   v
Phase 2
   |
   v
IPsec SA / Child SA
   |
   v
Encrypted User Traffic
```

---

# 64. IPsec Conceptual Full Flow

```text
                IPsec VPN
                    |
        +-----------+-----------+
        |                       |
      Phase 1                 Phase 2
        |                       |
        v                       v
      IKE SA                IPsec SA
        |                       |
        +-----------+-----------+
                    |
                    v
             Secure Data Path
                    |
                    v
             Encrypted Traffic
```

---

# 65. IPsec Packet Flow

## Before IPsec

```text
Original Packet

+-------------+
| IP Header   |
+-------------+
| TCP/UDP     |
+-------------+
| Data        |
+-------------+
```

## ESP Tunnel Mode

```text
+----------------------+
| Outer IP Header      |
+----------------------+
| ESP Header           |
+----------------------+
| Inner IP Header      |
+----------------------+
| TCP/UDP              |
+----------------------+
| Data                 |
+----------------------+
| ESP Trailer          |
+----------------------+
| Authentication Data  |
+----------------------+
```

---

# 66. Practical IPsec Checklist

## Phase 1

```text
[ ] Peer IP
[ ] IKE version
[ ] Encryption
[ ] Integrity
[ ] DH group
[ ] Authentication
[ ] PSK / Certificate
[ ] Local ID
[ ] Peer ID
[ ] NAT-T
[ ] DPD
[ ] Lifetime
[ ] Local-in policy
```

## Phase 2

```text
[ ] Encryption
[ ] Integrity
[ ] PFS
[ ] DH group
[ ] Local selector
[ ] Remote selector
[ ] Lifetime
[ ] Replay protection
```

## Routing

```text
[ ] Route to peer
[ ] Route to remote subnet
[ ] Reverse route
[ ] Policy route / SD-WAN
[ ] NAT exemption where required
```

## Security Policy

```text
[ ] LAN -> IPsec policy
[ ] IPsec -> LAN policy
[ ] NAT disabled where required
[ ] Correct service
[ ] Correct source/destination
```

---

# 67. Common IPsec Failure Domains

```text
                    IPsec Failure
                         |
       +-----------------+-----------------+
       |                 |                 |
    Phase 1           Phase 2          Data Plane
       |                 |                 |
   - PSK              - Proposal        - Route
   - DH               - PFS             - Policy
   - Encryption       - Selector        - NAT
   - Integrity        - Lifetime        - MTU
   - Identity                            - MSS
   - NAT-T                              - Asymmetric path
   - DPD
   - Local-in
```

---

# 68. Quick Reference — FortiGate Commands

```bash
# IKE Gateway
diagnose vpn ike gateway list

# IPsec tunnel
diagnose vpn tunnel list

# Flush IKE gateways
diagnose vpn ike gateway flush

# IKE debug
diagnose debug enable
diagnose debug application ike -1

# Debug flow for IKE
diagnose debug flow filter dport 500
diagnose debug flow trace start 10
diagnose debug enable

# Stop debugging
diagnose debug disable
diagnose debug reset
```

---

# 69. Quick Reference — Important Configuration

## DPD

```bash
config vpn ipsec phase1-interface
    edit "link-1"
        set dpd-retryinterval 15
        set dpd-retrycount 3
    next
end
```

## Reauthentication

```bash
config vpn ipsec phase1-interface
    edit "link-1"
        set reauth enable
    next
end
```

## QCD

```bash
config system setting
    set ike-quick-crash-detect enable
end
```

## IKEv2 Fragmentation

```bash
config vpn ipsec phase1-interface
    edit "link-1"
        set ike-version 2
        set fragmentation enable
        set fragmentation-mtu 500
    next
end
```

## Network ID

```bash
config vpn ipsec phase1-interface
    edit "link-1"
        set network-id 2
    next
end
```

## MSS

```bash
config firewall policy
    edit 1
        set tcp-mss-sender 1350
        set tcp-mss-receiver 1350
    next
end
```

---

# 70. CCNP Mental Model

```text
                 IPsec VPN
                     |
                     v
              +-------------+
              |   Phase 1   |
              |    IKE SA   |
              +-------------+
                     |
          +----------+----------+
          |                     |
       IKEv1                  IKEv2
          |                     |
     Main/Aggressive       IKE_SA_INIT
          |                     |
          |                  IKE_AUTH
          |                     |
          +----------+----------+
                     |
                     v
              +-------------+
              |   Phase 2   |
              | IPsec / Child|
              |     SA      |
              +-------------+
                     |
                     v
             Encrypted Traffic
                     |
                     v
              ESP / IPsec Data
```

---

# 71. One-Line Memory Aids

```text
CIA
Confidentiality = Nobody unauthorized can READ
Integrity       = Nobody unauthorized can MODIFY
Availability    = Authorized users can ACCESS

IKE = Builds the control/security relationship
IPsec SA = Protects the actual data

Phase 1 = IKE SA
Phase 2 = IPsec SA / Child SA

IKEv1 = Main/Aggressive + Quick Mode
IKEv2 = IKE_SA_INIT + IKE_AUTH + Child SA

ESP = Protocol 50
AH  = Protocol 51

IKE = UDP 500
NAT-T = UDP 4500

DPD = Is my peer alive?
NAT-T = Is there a NAT device between us?
QCD = Detect peer failure faster
XAuth = Additional user authentication
Mode Config = Assign client configuration
ADVPN = Dynamic VPN shortcuts
Network-ID = Separate IPsec behavior over the same interface
SPI = Identifies the IPsec SA
```

---

# 72. Final Troubleshooting Rule

```text
If Phase 1 is DOWN
    |
    +--> Check IKE / Authentication / DH / NAT / Local-In

If Phase 1 is UP but Phase 2 is DOWN
    |
    +--> Check IPsec Proposal / PFS / Selectors

If Phase 2 is UP but traffic is DOWN
    |
    +--> Check Routing / Firewall Policy / NAT / MTU / MSS

If Spoke-to-Spoke is DOWN
    |
    +--> Check ADVPN / Auto Discovery / Exchange Interface IP
         / NAT Traversal / UDP Hole Punching
```

> **Golden rule:**  
> **Phase 1 builds trust → Phase 2 builds the secure data path → Routing & Firewall Policy decide whether traffic actually flows.**
````

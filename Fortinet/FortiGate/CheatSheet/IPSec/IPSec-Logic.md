# 🔐 IPSec / IKE / VPN — CCNP & FortiGate

> Comprehensive study notes for IPSec, IKEv1/v2, NAT-T, DPD, QCD, ADVPN, XAuth, FortiGate and Cisco configuration.

---

# 1. 🔺 CIA Triad

The CIA triad is the foundation of information security and secure communications.

| Component | Meaning | Goal |
|---|---|---|
| **C** | Confidentiality | Prevent unauthorized access to information |
| **I** | Integrity | Ensure data is not modified |
| **A** | Availability | Ensure authorized access when required |

## Confidentiality

> Information should be accessible only to authorized entities.

Example:

```text
X1  -------------------->  X2

Only X1 and X2 should understand the information.
Other devices/users should not be able to read it.
````

### Protects against

* Unauthorized access
* Unauthorized use
* Data disclosure
* Data exposure

### Applies to

* Data at rest
* Data in transit
* Data in process

---

## Integrity

> Data received by X2 must be the same data sent by X1.

```text
X1
 |
 | Original Packet
 v
-------------------------
Network
-------------------------
 |
 v
X2
 |
 +--> Packet must not be modified
```

Integrity provides:

* Accuracy
* Completeness
* Detection of modification
* Protection during:

  * Storage
  * Transit
  * Processing

---

## Availability

> Authorized users must be able to access information and services when required.

Important concepts:

* Acceptable level of performance
* Fault tolerance / fault clearance
* Prevention of data loss
* Prevention of destruction
* Reliable backups
* Redundancy
* High availability

---

## CIA = Security Foundation

```text
             SECURITY
                 |
        +--------+--------+
        |        |        |
     Confidentiality  Integrity  Availability
        |        |        |
     secrecy   accuracy  access
```

> Confidentiality + Integrity + Availability form the basic trust model for secure communication.

---

# 2. 🔐 IPSec Overview

IPSec is not a single protocol.

It is a framework that combines multiple protocols and mechanisms.

```text
IPSec
 |
 +-- Encryption
 |
 +-- Integrity
 |
 +-- Authentication
 |
 +-- Key Exchange
 |
 +-- Security Associations
 |
 +-- Anti-Replay
 |
 +-- IKE
 |
 +-- AH / ESP
```

---

# 3. 🔒 Encryption

Encryption provides **Confidentiality**.

### Common algorithms

```text
DES
3DES
AES
```

### DES

* Data Encryption Standard
* Old
* Weak by modern standards
* Should not be used in new deployments

### 3DES

* Triple DES
* More secure than DES
* Legacy
* Slow compared with AES

### AES

* Advanced Encryption Standard
* Modern
* Fast
* Commonly used in IPSec

Common variants:

```text
AES-128
AES-192
AES-256
```

> Encryption alone is not enough. We also need integrity and authentication.

---

# 4. 🧮 Integrity / Hashing

Integrity mechanisms detect modification.

Common historical algorithms:

```text
MD5
SHA-1
SHA-256
SHA-512
```

### SHA

Secure Hash Algorithm.

Examples:

```text
SHA-1     -> 160 bits
SHA-256   -> 256 bits
SHA-512   -> 512 bits
```

> SHA-1 and MD5 are considered obsolete for modern cryptographic authentication.

### Important distinction

```text
Encryption  --> Confidentiality
Hash/HMAC   --> Integrity / Authentication
```

---

# 5. 🔑 Authentication

Authentication verifies the identity of the peer.

Common methods:

```text
Pre-Shared Key (PSK)
RSA / Digital Certificates
```

## Pre-Shared Key

A secret value is configured on both sides.

```text
Router-A
   |
   | shared secret
   |
Router-B
```

Both sides must know the same secret.

---

## RSA / Certificate Authentication

Uses asymmetric cryptography.

```text
Private Key
Public Key
Certificate
CA
```

Conceptually:

```text
Private Key  <---->  Public Key
```

Certificates provide scalable identity validation.

---

# 6. 🔄 Combining Encryption + Integrity + Authentication

A simplified IPSec process:

```text
Original Data
     |
     v
Integrity / Authentication
     |
     v
Encryption
     |
     v
Encrypted Packet
     |
     v
================ NETWORK ================
     |
     v
Decrypt
     |
     v
Verify Integrity / Authentication
     |
     v
Original Data
```

If authentication/integrity verification succeeds:

```text
Trusted + Unmodified
```

If verification fails:

```text
Packet rejected
```

---

# 7. 🔑 Diffie-Hellman

Diffie-Hellman (DH) is used for secure key agreement.

It allows two peers to establish shared secret material without directly sending the secret over the network.

Example historical groups:

```text
DH Group 1  -> 768-bit
DH Group 2  -> 1024-bit
DH Group 5  -> 1536-bit
DH Group 14 -> 2048-bit
```

> Modern deployments should prefer stronger groups such as DH14+ or elliptic-curve groups, depending on platform support and security requirements.

Conceptually:

```text
Peer A                          Peer B
  |                               |
  |------ DH public values ------>|
  |<----- DH public values -------|
  |                               |
  +------ Shared Secret ----------+
```

The shared secret is not directly transmitted.

---

# 8. 🧱 Security Association (SA)

A Security Association defines the security parameters used by IPSec.

Conceptually:

```text
IPSec
  |
  +-- Security Association
        |
        +-- Encryption
        +-- Integrity
        +-- Authentication
        +-- Lifetime
        +-- SPI
        +-- Traffic selectors
```

### SPI

SPI = Security Parameter Index

Used to identify the appropriate Security Association.

---

# 9. 🧩 IPSec Protocols

IPSec mainly uses:

```text
AH
ESP
```

---

# 10. 🛡️ AH — Authentication Header

AH = Authentication Header

AH provides:

* Integrity
* Authentication of packet origin
* Anti-replay protection

AH does **not** provide encryption.

Therefore:

```text
AH != Confidentiality
```

---

## AH Transport Mode

```text
+----------------+
| Original IP    |
+----------------+
| AH Header      |
+----------------+
| TCP / UDP      |
+----------------+
| Data           |
+----------------+
```

---

## AH Tunnel Mode

```text
+------------------------+
| New External IP Header |
+------------------------+
| AH Header               |
+------------------------+
| Original IP Header      |
+------------------------+
| TCP / UDP               |
+------------------------+
| Data                    |
+------------------------+
```

> AH authenticates protected IP information but does not encrypt the payload.

---

# 11. 🔐 ESP — Encapsulating Security Payload

ESP provides:

* Confidentiality
* Integrity
* Authentication
* Anti-replay protection

ESP is the most commonly used IPSec protocol.

### ESP Protocol Number

```text
IP protocol 50
```

---

# 12. ESP Transport Mode

Transport mode keeps the original IP header.

```text
+----------------+
| Original IP    |
+----------------+
| ESP Header     |
+----------------+
| TCP / UDP      | *
+----------------+
| Data           | *
+----------------+
| ESP Trailer    | *
+----------------+
| ESP Auth       |
+----------------+
```

`*` = protected/encrypted portion depending on ESP configuration.

Conceptually:

```text
Original IP Header
        |
        +-- ESP Header
        +-- Encrypted Payload
        +-- ESP Trailer
        +-- Authentication Data
```

---

# 13. ESP Tunnel Mode

Tunnel mode adds a new external IP header.

The original IP header is encapsulated inside ESP.

```text
+------------------------+
| New External IP Header |
+------------------------+
| ESP Header             |
+------------------------+
| Original IP Header   * |
+------------------------+
| TCP / UDP            * |
+------------------------+
| Data                  *|
+------------------------+
| ESP Trailer           *|
+------------------------+
| ESP Auth                |
+------------------------+
```

### Main advantage

The original/internal IP header is hidden from the outside network.

```text
Before:

Original Source ---> Original Destination


After:

Public/External Source ---> Public/External Destination
                              |
                              +-- encrypted internal packet
```

---

# 14. AH vs ESP

| Feature         |   AH |          ESP |
| --------------- | ---: | -----------: |
| Confidentiality |    ❌ |            ✅ |
| Integrity       |    ✅ |            ✅ |
| Authentication  |    ✅ |            ✅ |
| Encryption      |    ❌ |            ✅ |
| Anti-Replay     |    ✅ |            ✅ |
| NAT Friendly    |    ❌ | ✅ with NAT-T |
| Common Today    | Rare |  Very common |

---

# 15. 🔄 ESP + AH

AH and ESP can technically be combined.

However:

```text
ESP is normally preferred.
```

Reasons:

* ESP already supports encryption
* ESP can provide integrity/authentication
* NAT-T works with ESP
* Better interoperability

---

# 16. 🔑 IKE

IKE = Internet Key Exchange

IKE is responsible for:

* Negotiating security parameters
* Authentication
* Diffie-Hellman exchange
* Creating IKE SA
* Creating Child/IPSec SAs
* Rekeying
* Peer liveness mechanisms

Versions:

```text
IKEv1
IKEv2
```

---

# 17. IKEv1

IKEv1 uses:

```text
Phase 1
Phase 2
```

---

# 18. IKEv1 Phase 1

Purpose:

```text
Create the IKE / ISAKMP Security Association
```

Phase 1 establishes the secure control channel used for negotiating IPSec.

### Phase 1 contains

```text
Encryption
Integrity / Hash
Authentication
Diffie-Hellman Group
Lifetime
```

Example:

```text
Encryption: AES
Hash: SHA-256
Authentication: Pre-Shared Key
DH: Group 14
```

---

# 19. IKEv1 Main Mode

Main Mode uses **6 messages**.

```text
MM1
MM2
MM3
MM4
MM5
MM6
```

---

## MM1

Initiator -> Responder

Negotiation of security parameters.

Contains concepts such as:

```text
SA
VID
```

---

## MM2

Responder -> Initiator

Responder selects a matching proposal.

```text
SA
VID
```

---

## MM3

Initiator -> Responder

DH exchange and nonce exchange.

```text
KE
Nonce
VID
```

---

## MM4

Responder -> Initiator

Responder sends its DH/nonce information.

```text
KE
Nonce
VID
```

---

## MM5

Initiator -> Responder

Identity/authentication information.

```text
ID
Authentication
Certificate
Certificate Request
```

---

## MM6

Responder -> Initiator

Identity/authentication information.

```text
ID
Authentication
Certificate
```

---

# 20. IKEv1 Main Mode Security Sequence

Simplified:

```text
MM1 + MM2
    |
    +--> Negotiate crypto parameters
    |
MM3 + MM4
    |
    +--> DH / key exchange
    |
MM5 + MM6
    |
    +--> Identity + authentication
```

### Security state

```text
MM1/MM2
    |
    +--> Negotiation
    |    Unencrypted
    |    Unauthenticated
    |
MM3/MM4
    |
    +--> DH / Key exchange
    |
MM5/MM6
    |
    +--> Identity + Authentication
         Protected by negotiated keys
```

---

# 21. IKEv1 Main Mode States

Common troubleshooting states:

```text
MM_WAIT_MSG2
MM_WAIT_MSG3
MM_WAIT_MSG4
MM_WAIT_MSG5
MM_WAIT_MSG6
MM_ACTIVE
```

---

## MM_WAIT_MSG2

Initiator sent:

```text
Encryption
Hash / Integrity
DH
IKE policy
```

and waits for the responder.

---

## MM_WAIT_MSG3

Responder received the initial proposal.

It checks whether it has a matching IKE policy.

If stuck here, investigate:

* Routing
* Peer reachability
* IKE policy mismatch
* Crypto map
* Interface
* Firewall filtering

---

## MM_WAIT_MSG4

Initiator sends authentication-related material.

Possible causes of getting stuck:

* Missing PSK
* Wrong PSK
* Missing tunnel-group
* Identity mismatch

---

## MM_WAIT_MSG5

Responder is processing peer authentication.

If PSKs do not match, the responder may remain in this state.

---

## MM_WAIT_MSG6

Initiator waits for final authentication.

Possible issue:

```text
PSK mismatch
Authentication mismatch
Identity mismatch
```

---

## MM_ACTIVE

IKEv1 Phase 1 is established.

```text
IKE SA = UP
```

Then Phase 2 can be negotiated.

---

# 22. IKEv1 Aggressive Mode

Aggressive Mode uses **3 messages**.

```text
AM1
AM2
AM3
```

---

## AM1

Initiator sends:

```text
SA
KE
Nonce
ID
```

---

## AM2

Responder sends:

```text
SA
KE
Nonce
ID
Authentication
```

---

## AM3

Initiator sends:

```text
Hash / Authentication
```

---

## Aggressive Mode Summary

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
 +-- Hash
 +-- Authentication
```

> Aggressive Mode is faster but exposes identity information earlier and is generally less desirable than Main Mode.

---

# 23. IKEv1 Phase 2

Purpose:

```text
Create IPSec SAs
```

Phase 2 negotiates:

* IPSec encryption
* IPSec integrity
* Traffic selectors
* Lifetime
* PFS if configured
* IPSec SA parameters

---

# 24. IKEv1 Quick Mode

Quick Mode commonly uses 3 messages:

```text
QM1
QM2
QM3
```

Conceptually:

```text
QM1 --> Proposal / negotiation
QM2 <-- Response / approval
QM3 --> Final confirmation
```

The result is IPSec SA establishment.

---

# 25. IKEv1 Phase 1 vs Phase 2

|             | Phase 1         | Phase 2           |
| ----------- | --------------- | ----------------- |
| Purpose     | IKE SA          | IPSec SA          |
| Plane       | Control plane   | Data plane        |
| Protocol    | IKE/ISAKMP      | IPSec             |
| Negotiates  | IKE parameters  | ESP/AH parameters |
| Common mode | Main/Aggressive | Quick Mode        |

---

# 26. IKEv2

IKEv2 is the newer IKE protocol.

Advantages:

* Fewer messages
* Simpler architecture
* Better mobility support
* Better NAT traversal behavior
* Built-in liveness mechanisms
* EAP support
* Better scalability
* Multiple Child SAs
* More efficient rekeying

---

# 27. IKEv2 Initial Exchange

IKEv2 uses:

```text
IKE_SA_INIT
```

Two messages:

```text
IKE_SA_INIT Request
IKE_SA_INIT Response
```

---

## IKE_SA_INIT Request

Contains concepts such as:

```text
SA
KE
Nonce
Vendor ID
```

---

## IKE_SA_INIT Response

Contains:

```text
SA
KE
Nonce
Vendor ID
```

---

## IKEv2 SA_INIT Purpose

Negotiates:

```text
Cryptographic algorithms
DH exchange
Nonces
```

Simplified:

```text
IKE_SA_INIT
      |
      +--> Crypto negotiation
      |
      +--> DH exchange
      |
      +--> Nonce exchange
```

---

# 28. IKEv2 Authentication Exchange

After SA_INIT:

```text
IKE_AUTH Request
IKE_AUTH Response
```

---

## IKE_AUTH Request

Can contain:

```text
ID
AUTH
Certificate
SA
Traffic Selectors
NAT detection
SPI
```

---

## IKE_AUTH Response

Can contain:

```text
ID
AUTH
Certificate
SA
Traffic Selectors
NAT detection
SPI
```

---

# 29. IKEv2 Message Flow

```text
Message 1
Initiator
    |
    | IKE_SA_INIT Request
    v
Responder
    |
    | IKE_SA_INIT Response
    v
Message 2

        ↓

Message 3
Initiator
    |
    | IKE_AUTH Request
    v
Responder
    |
    | IKE_AUTH Response
    v
Message 4
```

Therefore:

```text
IKEv2 Initial Exchange = 2 messages
IKEv2 Authentication = 2 messages

Total = 4 messages
```

---

# 30. IKEv1 vs IKEv2

| Feature             | IKEv1               | IKEv2              |
| ------------------- | ------------------- | ------------------ |
| Initial negotiation | More messages       | Fewer messages     |
| Main Mode           | 6 messages          | N/A                |
| Aggressive Mode     | 3 messages          | N/A                |
| Initial exchange    | 6-message Main Mode | 4-message exchange |
| EAP                 | Limited             | Built-in           |
| NAT-T               | Supported           | Built-in design    |
| Child SAs           | Less flexible       | Native             |
| Mobility            | Limited             | Better             |
| Scalability         | Lower               | Better             |
| Complexity          | Higher              | Lower              |

---

# 31. IKEv2 Child SA

IKEv2 uses:

```text
Child SA
```

A single IKE SA can protect multiple Child SAs.

Conceptually:

```text
             IKE SA
                |
       +--------+--------+
       |        |        |
    Child SA  Child SA  Child SA
       |        |        |
      VPN1     VPN2     VPN3
```

This is one reason IKEv2 is more flexible for large deployments.

---

# 32. EAP in IKEv2

EAP = Extensible Authentication Protocol

Used to support additional authentication methods.

Example:

```text
IKEv2
  |
  +-- EAP
       |
       +-- Authentication Server
       |
       +-- User Authentication
```

Useful for:

* Remote access
* User authentication
* Certificate-based environments
* Enterprise authentication

---

# 33. IPSec Tunnel Hierarchy

A useful conceptual hierarchy:

```text
IKE
 |
 +-- IKE SA
      |
      +-- Child / IPSec SA
            |
            +-- SPI
            |
            +-- ESP
            |
            +-- Encrypted Traffic
```

For IKEv1:

```text
ISAKMP/IKE Phase 1
        |
        v
IPSec Phase 2
        |
        v
IPSec SA
        |
        v
SPI
```

---

# 34. Site-to-Site VPN vs Remote Access

### Site-to-Site

Usually:

```text
IPSec VPN
```

Example:

```text
LAN-A
 |
Router-A
 |
==== Internet ====
 |
Router-B
 |
LAN-B
```

### Remote Access

Common approaches:

```text
SSL VPN
IKEv2/IPSec
```

> The exact technology depends on platform and deployment requirements.

---

# 35. GRE + IPSec

GRE provides tunneling features such as:

* Multicast
* Dynamic routing protocols
* Broadcast
* Flexible encapsulation

IPSec provides:

* Encryption
* Integrity
* Authentication

Conceptually:

```text
Routing Protocol
       |
      GRE
       |
     IPSec
       |
   Internet
```

Cisco commonly uses:

```text
Tunnel Interface
+
GRE
+
IPSec protection
```

---

# 36. Cisco GRE Keepalive

Example:

```cisco
interface Tunnel0
 keepalive 3 3
```

Conceptually:

```text
keepalive <number-of-probes> <interval>
```

Example:

```text
3 probes
3 seconds interval
```

---

# 37. Cisco IPSec Site-to-Site Configuration

## IKEv1 Phase 1

```cisco
crypto isakmp policy 10
 encryption aes
 hash sha256
 authentication pre-share
 group 14
```

Purpose:

```text
Create IKEv1 Phase 1 policy.
```

---

## Pre-Shared Key

```cisco
crypto isakmp key vpnuser address 10.0.0.2
```

Defines:

```text
PSK
+
Remote Peer
```

---

# 38. Cisco IPSec Phase 2

```cisco
crypto ipsec transform-set MYSET esp-aes esp-sha256-hmac
```

Defines:

```text
ESP Encryption
+
ESP Integrity
```

---

# 39. Interesting Traffic

```cisco
access-list 100 permit ip 192.168.101.0 0.0.0.255 192.168.102.0 0.0.0.255
```

This identifies traffic that should be protected by IPSec.

```text
192.168.101.0/24
        |
       VPN
        |
192.168.102.0/24
```

---

# 40. Cisco Crypto Map

```cisco
crypto map MYMAP 10 ipsec-isakmp
 set peer 10.0.0.2
 set transform-set MYSET
 match address 100
```

---

# 41. Apply Crypto Map

```cisco
interface GigabitEthernet0/0
 ip address 192.168.102.1 255.255.255.0
 crypto map MYMAP
```

The crypto map must be applied to the appropriate external interface.

---

# 42. Cisco Tunnel-Protected IPSec Example

## Keyring

```cisco
crypto keyring PRESHARE
 pre-shared-key address 11.12.13.2 key secret
```

---

## IKE Policy

```cisco
crypto isakmp policy 1
 encr 3des
 authentication pre-share
 group 5
```

> 3DES and DH5 are legacy choices and should normally be replaced with stronger algorithms/groups in new deployments.

---

## IKE Profile

```cisco
crypto isakmp profile PRESHARE
 keyring PRESHARE
 match identity address 11.12.13.2 255.255.255.255
```

---

## IPSec Transform Set

```cisco
crypto ipsec transform-set AES-SHA1 esp-aes esp-sha-hmac
 mode tunnel
```

---

## IPSec Profile

```cisco
crypto ipsec profile IPSEC-PRESHARE
 set transform-set AES-SHA1
 set isakmp-profile PRESHARE
```

---

## Tunnel Interface

```cisco
interface Tunnel111
 ip address 192.168.111.1 255.255.255.0
 tunnel source 11.12.13.1
 tunnel mode ipsec ipv4
 tunnel destination 11.12.13.2
 tunnel protection ipsec profile IPSEC-PRESHARE
```

---

## Route

```cisco
ip route 192.168.102.0 255.255.255.0 Tunnel111
```

---

# 43. 🛰️ Auto Discovery VPN / ADVPN

ADVPN = Auto Discovery VPN

Used to dynamically establish VPN connectivity.

Useful for:

```text
Hub-and-Spoke
Spoke-to-Spoke
Dynamic Mesh
Large-scale VPN
SD-WAN
```

Concept:

```text
             HUB
           /     \
        Spoke1  Spoke2
          \       /
           \     /
        Dynamic
       Spoke-to-Spoke
```

Without dynamic shortcut:

```text
Spoke1 --> Hub --> Spoke2
```

With ADVPN:

```text
Spoke1 ---------> Spoke2
       shortcut
```

---

# 44. ADVPN + SD-WAN

ADVPN can be combined with SD-WAN to dynamically manage:

* Hub-and-spoke connectivity
* Spoke-to-spoke connectivity
* Dynamic shortcuts
* Route advertisement
* Link selection
* Large-scale VPN topology

---

# 45. IPSec Phase 1 Advanced Features

Important Phase 1 features include:

```text
Security Association
Profiles
Automatic negotiation
Negotiation timeout
Mode Config
NAT Traversal
Keepalive
Dead Peer Detection
XAuth
Auto Discovery
Dynamic routes
Network ID
```

---

# 46. Mode Config

Mode Config can provide dynamic parameters to clients.

Examples:

```text
IP Address
DNS
Routes
Other client parameters
```

Useful for:

```text
Dial-up VPN
Remote Access
Dynamic VPN clients
```

---

# 47. NAT Traversal — NAT-T

NAT-T = NAT Traversal

IPSec ESP uses:

```text
IP protocol 50
```

NAT devices cannot normally translate ESP like TCP/UDP.

NAT-T encapsulates IPSec traffic in UDP.

Common ports:

```text
UDP 500
UDP 4500
```

When NAT is detected:

```text
ESP
  ↓
UDP 4500
  ↓
NAT
  ↓
Internet
```

---

# 48. NAT Detection

IKE peers exchange NAT-D payloads.

Simplified process:

```text
Peer A
IP:Port
   |
   | Hash(IP + Port)
   v
Peer B
```

Both peers calculate hashes.

If calculated values differ:

```text
NAT detected
```

Then traffic moves to:

```text
UDP 4500
```

---

# 49. NAT-T IKEv2 Flow

Simplified:

```text
1. Source IP:500
        |
        v
   Destination:500

2. Response
        |
        v
   UDP 500

3. NAT-D payload
        |
        v
   Hash(IP + Port)

4. Compare hashes
        |
        +---- Same ----> No NAT
        |
        +---- Different -> NAT detected
                              |
                              v
                         UDP 4500
```

---

# 50. NAT-T Modes

Common concepts:

```text
Enable
Force
```

### Enable

Allows NAT-T when NAT is detected.

### Force

Forces NAT-T behavior.

> Use the mode appropriate to the actual topology and platform requirements.

---

# 51. Keepalive

NAT devices may remove idle UDP mappings.

Therefore keepalive traffic can maintain the NAT mapping.

```text
Peer A
  |
  | Keepalive
  v
NAT
  |
  v
Peer B
```

Purpose:

```text
Prevent idle NAT mapping expiration
```

---

# 52. Dead Peer Detection — DPD

DPD determines whether the remote peer is alive.

Conceptually similar to:

```text
Are you alive?
     |
     v
R-U-THERE
     |
     v
R-U-THERE-ACK
```

If the peer does not respond:

```text
IKE SA
   |
   +--> Consider peer dead
   |
   +--> Remove / renegotiate VPN
```

---

# 53. DPD Modes

Common modes:

```text
on-idle
on-demand
```

---

## on-idle

DPD is triggered when the VPN is idle.

Useful for:

```text
Dial-up VPN
Large VPN deployments
Resource optimization
```

---

## on-demand

DPD is triggered based on demand/traffic conditions.

Commonly useful for:

```text
Site-to-site VPN
```

> Exact behavior depends on FortiOS version and configuration.

---

# 54. DPD Timers

Example:

```cisco
dpd-retrycount
dpd-retryinterval
```

FortiGate example:

```fortigate
config vpn ipsec phase1-interface
    edit "link-1"
        set dpd-retryinterval 15
        set dpd-retrycount 3
    next
end
```

Conceptually:

```text
Retry interval = 15 seconds
Retry count    = 3
```

---

# 55. DPD Scalability

For large dial-up deployments:

```text
Thousands of peers
       |
       v
Aggressive probing
       |
       v
CPU / bandwidth overhead
```

Therefore optimize DPD behavior.

Useful approach:

```text
Dial-up  --> on-idle
Site-to-site --> on-demand
```

Always validate against the FortiOS version and topology.

---

# 56. FortiGate DPD Verification

```bash
diagnose vpn ike gateway list
```

Useful information can include:

* IKE gateway
* SA state
* Peer address
* Proposal
* Authentication
* Hash/HMAC information
* Negotiated parameters

---

# 57. IKEv2 Reauthentication

IKEv2 supports reauthentication.

Example:

```fortigate
config vpn ipsec phase1-interface
    edit "link-1"
        set reauth enable
    next
end
```

The key lifetime / authentication lifetime can determine when reauthentication occurs.

Conceptually:

```text
IKE SA
  |
  | lifetime
  v
Reauthentication
  |
  v
New authenticated state
```

---

# 58. Quick Crash Detection — QCD

QCD = Quick Crash Detection

Purpose:

```text
Fast detection of peer failure
```

Similar goal to DPD, but designed for faster crash/failure detection.

---

## IKEv1 QCD

Some implementations use vendor-specific extensions such as:

```text
R-U-THERE
R-U-THERE-ACK
```

---

## IKEv2 QCD

IKEv2 provides improved built-in mechanisms.

It can use:

```text
Notify messages
Invalid IKE SPI
Invalid Child SA SPI
```

---

# 59. Invalid SPI

When an IKE message is received for an unknown SPI, a notification can indicate:

```text
INVALID_IKE_SPI
```

Similarly, IPSec can use:

```text
INVALID_SPI
```

These notifications help peers recover from stale or mismatched security associations.

---

# 60. QCD Tokens

QCD tokens can be derived using information associated with the IKE SA/SPI and authentication exchange.

Purpose:

```text
Fast peer-state validation
+
Failure detection
```

---

# 61. FortiGate QCD

Example:

```fortigate
config system settings
    set ike-quick-crash-detect enable
end
```

> Verify exact command syntax for the FortiOS version in use.

---

# 62. IKE Fragmentation

IKE messages can become large.

Examples:

```text
Certificates
Large authentication payloads
Large identity payloads
```

Large UDP packets can cause fragmentation problems.

---

# 63. IKEv1 Fragmentation

Example:

```fortigate
config vpn ipsec phase1-interface
    edit "link-1"
        set fragmentation enable
    next
end
```

> FortiOS defaults and behavior can vary by release; verify before changing the default.

---

# 64. IKEv2 Fragmentation

Example:

```fortigate
config vpn ipsec phase1-interface
    edit "link-1"
        set ike-version 2
        set fragmentation enable
        set fragmentation-mtu 500
    next
end
```

---

# 65. MTU / Fragmentation

IPv4 and IPv6 have different fragmentation considerations.

Large IKE packets may be fragmented by the network.

Symptoms:

```text
IKE negotiation stuck
Retransmissions
MM_WAIT states
IKE_AUTH failure
Certificate authentication failure
```

Troubleshooting:

```text
Check MTU
Check ISP
Check firewall
Check NAT
Enable IKE fragmentation
```

---

# 66. Embryonic Limit

Embryonic limit protects the device against excessive connection attempts.

Concept:

```text
Internet
   |
   | many IKE/IPSec requests
   v
FortiGate
   |
   +--> Embryonic limit
            |
            +--> Limit incomplete negotiations
```

Example:

```fortigate
config system settings
    set embryonic-limit 50
end
```

> Exact platform support and limits depend on FortiOS model/version.

---

# 67. XAuth

XAuth = Extended Authentication

Often used with:

```text
IKEv1
Dial-up VPN
Remote Access
User authentication
```

Authentication can integrate with:

```text
Local users
LDAP
RADIUS
CHAP
PAP
User groups
```

---

# 68. XAuth Modes

Common concepts:

```text
PAP
CHAP
AUTO
```

### PAP

Password Authentication Protocol.

Simple authentication method.

### CHAP

Challenge Handshake Authentication Protocol.

Uses challenge/response rather than sending the password directly.

### Auto Server

Can negotiate based on server capabilities.

> Exact FortiGate behavior depends on the configured authentication server and FortiOS version.

---

# 69. XAuth Client vs Server

### XAuth Server

FortiGate authenticates remote users.

```text
Remote User
    |
    v
FortiGate
    |
    +--> User Group
    |
    +--> Authentication Server
```

### XAuth Client

FortiGate acts as a client and authenticates toward a remote VPN gateway.

---

# 70. Dynamic Routes + Mode Config

Dynamic VPN clients can receive:

```text
IP address
Routes
DNS
Other parameters
```

For large VPN fabrics:

```text
Mode Config
+
Dynamic Routing
+
IPSec
+
ADVPN
```

can provide scalable connectivity.

---

# 71. Network ID

FortiGate supports Network ID to distinguish different VPN behaviors over the same public interface.

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
Same Public Interface
        |
        +-- Network ID 1
        |
        +-- Network ID 2
        |
        +-- Network ID 3
```

Useful for segmentation and distinguishing multiple IPsec relationships.

---

# 72. Auto Discovery Sender / Receiver

Used in dynamic VPN environments such as:

```text
ADVPN
Dynamic Mesh
Large Hub-and-Spoke
SD-WAN
```

Can help advertise:

```text
Routes
Discovery information
Hello messages
Spoke information
```

---

# 73. Exchange Interface IP

In hub-and-spoke designs, spokes may need to know the interface IP of other spokes.

Concept:

```text
Hub
 |
 +---- Spoke A
 |
 +---- Spoke B

Spoke A <--------> Spoke B
```

The hub can help exchange information needed for direct connectivity.

---

# 74. Aggregate Members

Multiple IPSec interfaces can be combined into an aggregate design.

Purpose:

```text
Redundancy
Load sharing
High availability
Multiple links
```

Conceptually:

```text
             Aggregate
             /       \
        IPSec-1     IPSec-2
```

---

# 75. Mesh Selector Network

Mesh selectors can help create dynamic spoke-to-spoke connectivity.

Example:

```fortigate
config vpn ipsec phase1-interface
    edit "link-1"
        set mesh-selector-type enable
    next
end
```

Typically used together with relevant:

```text
Auto-discovery
Exchange interface IP
Phase 2 selectors
```

Concept:

```text
Hub
 |
 +--- Spoke A
 |      \
 |       \
 +--- Spoke B
          \
           Spoke-to-Spoke
```

---

# 76. Phase 2 Selectors and Full Mesh

Phase 2 selectors determine which traffic is protected.

For dynamic full-mesh environments:

```text
Too restrictive selectors
        |
        v
Limited connectivity
```

More flexible selectors can support:

```text
Spoke-to-Spoke
Dynamic routes
ADVPN
Full mesh behavior
```

---

# 77. DHCP over VPN

DHCP relay can be used in VPN designs.

Concept:

```text
Client
  |
  v
DHCP Relay
  |
  v
IPSec VPN
  |
  v
DHCP Server
```

Some VPN architectures can use relay/proxy mechanisms to forward DHCP requests.

---

# 78. 🔥 Local-In Policy for Blocking IKE / IPSec

Local-in policies control traffic destined **to the FortiGate itself**.

Useful for protecting:

```text
IKE
IPSec
Management
Other local services
```

Example scenario:

```text
Internet
   |
   v
FortiGate
   |
   +--> Local-In Policy
           |
           +--> Allow
           +--> Deny
```

---

# 79. Create Address Object

Example:

```text
Name: block-2
Type: Subnet
IP: 2.2.2.2/32
Interface: any
```

---

# 80. FortiGate Local-In Policy

Example:

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

> Adjust object names and services for the actual configuration.

---

# 81. Local-In Policy Logging

Example:

```fortigate
config log setting
    set local-in-allow enable
    set local-in-deny enable
    set local-in-br enable
end
```

Use logging to determine:

```text
Who is attempting IKE?
Which source?
Which interface?
Which destination?
Was it allowed?
Was it denied?
```

---

# 82. Debug IKE / Local-In Traffic

Example:

```bash
diagnose debug flow filter dport 500
diagnose debug flow trace start 10
diagnose debug enable
```

Remember to stop debugging after troubleshooting:

```bash
diagnose debug disable
diagnose debug reset
```

---

# 83. Change IKE Port

FortiGate example:

```fortigate
config system settings
    set ike-port 500
end
```

Common IKE ports:

```text
UDP 500
UDP 4500
```

> Changing the IKE port is a topology-specific design choice and both peers must support the configuration.

---

# 84. FortiGate IPSec Troubleshooting Commands

## Show IPSec tunnels

```bash
diagnose vpn tunnel list
```

Useful for:

* Tunnel state
* SPI
* Proposal
* SA
* Encapsulation
* Counters

---

## Show IKE gateways

```bash
diagnose vpn ike gateway list
```

Useful for:

* IKE SA
* Peer address
* Local address
* Proposal
* Authentication
* Gateway state

---

# 85. Dial-up IP Delay

In some FortiGate dial-up VPN scenarios, released addresses may remain reserved for a period.

Example:

```fortigate
config vpn ipsec phase1-interface
    edit "link-1"
        set ip-delay-interval 200
    next
end
```

This can adjust the delay before an address is reused.

---

# 86. Flush IKE Gateways

To clear existing IKE gateway state:

```bash
diagnose vpn ike gateway flush
```

Useful during troubleshooting when you need to force renegotiation.

---

# 87. TCP MSS / MTU

IPSec adds overhead.

Therefore:

```text
Original MTU
   |
   +-- IPSec overhead
   |
   v
Effective MTU
```

If TCP packets become too large:

```text
Fragmentation
Retransmission
Poor performance
Connection problems
```

---

# 88. TCP MSS Adjustment

Example:

```fortigate
config firewall policy
    edit 1
        set tcp-mss-sender 1350
        set tcp-mss-receiver 1350
    next
end
```

Concept:

```text
MTU problem
    |
    v
Reduce TCP MSS
    |
    v
Smaller TCP segments
    |
    v
Less fragmentation
```

> MSS value should be calculated/tested for the actual tunnel overhead and path MTU.

---

# 89. UDP Hole Punching

UDP hole punching helps establish direct connectivity between peers behind NAT.

Useful in:

```text
ADVPN
Spoke-to-Spoke
NATed Spokes
Dynamic VPN
```

Concept:

```text
Spoke A
   |
  NAT
   |
Internet
   |
  NAT
   |
Spoke B
```

The hub helps the spokes learn the reachable public addresses/ports.

---

# 90. ADVPN UDP Hole Punching

Simplified flow:

```text
Spoke A
   |
   | Shortcut Offer
   v
Hub
   |
   | Shortcut information
   v
Spoke B
   |
   | Shortcut Query
   v
Hub
   |
   | Shortcut Reply
   v
Spoke A
```

Then:

```text
Spoke A <===========> Spoke B
        UDP 4500
```

---

# 91. Example ADVPN Debug

Example:

```text
diagnose debug enable

diagnose debug application ike -1
```

Possible messages:

```text
SHORTCUT-OFFER
SHORTCUT-QUERY
SHORTCUT-REPLY
```

Example conceptual output:

```text
ike 0:toHub1: notify msg received: SHORTCUT-OFFER

ike 0:toHub1:
shortcut-offer
10.1.100.11 -> 192.168.4.33

ike 0:toHub1:
send shortcut-query

ike 0:toHub1:
SHORTCUT-QUERY
12.1.1.2:4500 -> 22.1.1.1:4500

ike 0:toHub1:
jshortcut-reply received
from 55.1.1.2:64916

local-nat=yes
peer-nat=yes

ike 0:toHub1:
NAT hole punching to peer
55.1.1.2:64916

ike 0:toHub1:
created connection
12.1.1.2 -> 55.1.1.2:64916

ike 0:toHub1:
adding new dynamic tunnel
55.1.1.2:64916
```

---

# 92. Understanding NAT Hole Punching Output

Example:

```text
55.1.1.2:64916
```

This represents the public/NAT-mapped UDP endpoint learned for the peer.

Concept:

```text
Private Spoke
10.x.x.x
    |
    v
NAT
    |
    v
Public IP:UDP Port
55.1.1.2:64916
```

The dynamic tunnel can then use this reachable endpoint.

---

# 93. Spoke-to-Spoke Shortcut

Final result:

```text
Before:

Spoke A
   |
   v
 Hub
   |
   v
Spoke B
```

After shortcut:

```text
Spoke A
   |
   +==================+
                      |
                      v
                   Spoke B
```

This removes unnecessary hub forwarding.

---

# 94. Useful FortiGate IPSec Commands

```bash
diagnose vpn tunnel list
```

```bash
diagnose vpn ike gateway list
```

```bash
diagnose vpn ike gateway flush
```

```bash
diagnose debug application ike -1
```

```bash
diagnose debug enable
```

```bash
diagnose debug disable
```

```bash
diagnose debug reset
```

---

# 95. Troubleshooting Methodology

Use a layered approach.

```text
Layer 1
  |
  +--> Physical / ISP
  |
Layer 2
  |
  +--> VLAN / Ethernet
  |
Layer 3
  |
  +--> Routing
  |
Layer 4
  |
  +--> UDP 500 / UDP 4500
  |
IKE Phase 1
  |
  +--> Proposal
  +--> DH
  +--> Authentication
  |
IKE Phase 2
  |
  +--> Transform Set
  +--> Selectors
  +--> PFS
  |
IPSec
  |
  +--> ESP
  +--> SPI
  +--> Counters
  |
Application
```

---

# 96. IKEv1 Troubleshooting States

```text
MM_WAIT_MSG2
       |
       +--> Policy / reachability

MM_WAIT_MSG3
       |
       +--> Routing / peer response

MM_WAIT_MSG4
       |
       +--> PSK / tunnel-group / identity

MM_WAIT_MSG5
       |
       +--> PSK mismatch / authentication

MM_WAIT_MSG6
       |
       +--> Final authentication

MM_ACTIVE
       |
       +--> Phase 1 UP
```

---

# 97. IPSec Troubleshooting Checklist

## Phase 1

Check:

```text
IKE version
Encryption
Integrity
DH group
Authentication method
PSK
Certificates
Peer IP
Local IP
Identity
Lifetime
NAT-T
UDP 500
UDP 4500
```

---

## Phase 2

Check:

```text
ESP encryption
ESP integrity
PFS
DH group
Traffic selectors
Proxy IDs
Lifetime
Crypto map
IPSec profile
```

---

## Data Plane

Check:

```text
Routes
Firewall policies
NAT exemption
MTU
MSS
ESP counters
SPI
Traffic selectors
Routing symmetry
```

---

# 98. Common IPSec Failure Mapping

| Symptom                  | Possible Cause                     |
| ------------------------ | ---------------------------------- |
| No IKE packets           | Routing / firewall / ISP           |
| No MM2                   | Peer unreachable / UDP 500 blocked |
| MM_WAIT_MSG3             | Return path / policy               |
| MM_WAIT_MSG4             | PSK / identity                     |
| MM_WAIT_MSG5             | PSK mismatch                       |
| MM_WAIT_MSG6             | Authentication                     |
| Phase 1 UP, Phase 2 DOWN | Transform/selector/PFS             |
| Tunnel UP, no traffic    | Routing / policy / selectors       |
| Intermittent tunnel      | DPD / NAT timeout                  |
| Large packets fail       | MTU / fragmentation                |
| NAT environment fails    | NAT-T / UDP 4500                   |
| High CPU with dial-up    | DPD / negotiation load             |
| Spoke-to-spoke fails     | ADVPN / shortcut / NAT             |

---

# 99. IPSec Packet Numbers

Important protocol/port values:

```text
ESP       = IP protocol 50
AH        = IP protocol 51
IKE       = UDP 500
NAT-T     = UDP 4500
```

Quick memory:

```text
50 = ESP
51 = AH
500 = IKE
4500 = NAT-T
```

---

# 100. 🔥 Key Concept — ESP vs IKE

Do not confuse:

```text
IKE
```

with:

```text
ESP
```

### IKE

Creates and manages security associations.

```text
IKE
 |
 +-- Authentication
 +-- DH
 +-- Crypto negotiation
 +-- IKE SA
 +-- Child SA
 +-- Rekey
 +-- Liveness
```

### ESP

Carries/protects actual user data.

```text
ESP
 |
 +-- Encryption
 +-- Integrity
 +-- Authentication
 +-- Anti-Replay
```

---

# 101. 🔥 Control Plane vs Data Plane

```text
             VPN
              |
       +------+------+
       |             |
   Control Plane   Data Plane
       |             |
      IKE           ESP
       |             |
   Negotiate       Protect
   Authenticate    User Data
   Establish SA    Forward Traffic
```

---

# 102. 🔥 Complete IPSec Mental Model

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

---

# 103. IKEv1 Mental Model

```text
                 IKEv1
                   |
             +-----+-----+
             |           |
          Phase 1      Phase 2
             |           |
          Main /       Quick
        Aggressive      Mode
             |           |
        IKE/ISAKMP     IPSec SA
             |           |
             +-----+-----+
                   |
                  ESP
                   |
               User Data
```

---

# 104. IKEv2 Mental Model

```text
                  IKEv2
                    |
              +-----+------+
              |            |
         IKE_SA_INIT    IKE_AUTH
              |            |
       Crypto + DH     Identity +
       + Nonce         Authentication
              |            |
              +-----+------+
                    |
                IKE SA
                    |
              +-----+------+
              |            |
          Child SA 1    Child SA 2
              |            |
             ESP          ESP
              |            |
           Traffic      Traffic
```

---

# 105. IKEv1 vs IKEv2 Quick Memory

```text
IKEv1:

Phase 1
  |
  +-- Main Mode = 6 messages
  +-- Aggressive = 3 messages
  |
Phase 2
  |
  +-- Quick Mode = 3 messages


IKEv2:

IKE_SA_INIT
  |
  +-- Request
  +-- Response

IKE_AUTH
  |
  +-- Request
  +-- Response

Total initial exchange = 4 messages
```

---

# 106. 🔥 Important Security Recommendations

For modern deployments:

```text
Prefer AES
Prefer SHA-256 or stronger integrity
Prefer strong DH groups
Prefer IKEv2
Prefer certificates for scalable authentication
Use NAT-T when required
Use DPD appropriately
Use strong PSKs when PSK is required
Avoid legacy DES
Avoid legacy 3DES
Avoid MD5
Avoid SHA-1 for new deployments
Avoid weak DH groups
```

---

# 107. Quick Exam  

```text
CIA
 |
 +-- Confidentiality = Encryption
 +-- Integrity       = Hash/HMAC
 +-- Availability    = Reliability/Redundancy


IPSec
 |
 +-- AH  = Integrity + Authentication
 |
 +-- ESP = Encryption + Integrity + Authentication
 |
 +-- IKE = Negotiation + Authentication + Key Management


IKEv1
 |
 +-- Phase 1
 |    |
 |    +-- Main Mode = 6 messages
 |    +-- Aggressive = 3 messages
 |
 +-- Phase 2
      |
      +-- Quick Mode = 3 messages


IKEv2
 |
 +-- IKE_SA_INIT
 |    |
 |    +-- SA
 |    +-- KE
 |    +-- Nonce
 |
 +-- IKE_AUTH
      |
      +-- ID
      +-- AUTH
      +-- SA
      +-- TS
      +-- Certificate


Ports / Protocols
 |
 +-- ESP = 50
 +-- AH  = 51
 +-- IKE = UDP 500
 +-- NAT-T = UDP 4500
```

---

# 108. 🧠 One-Line Memory Map

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

# 109. 🚀 ADVPN Mental Model

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

# 110. Final IPSec Architecture

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
   Encryption      DH/Auth     ESP     SPI
       |                        |
       +-----------+------------+
                   |
              Protected Data
```

---

# 📌 Final Summary

| Topic             | Key Point                                 |
| ----------------- | ----------------------------------------- |
| CIA               | Confidentiality, Integrity, Availability  |
| Encryption        | Provides confidentiality                  |
| Hash/HMAC         | Provides integrity/authentication         |
| AH                | Authentication/integrity, no encryption   |
| ESP               | Encryption + integrity + authentication   |
| IKE               | Negotiates and manages VPN security       |
| DH                | Secure key agreement                      |
| SA                | Security parameters/state                 |
| SPI               | Identifies an SA                          |
| IKEv1 Phase 1     | IKE/ISAKMP SA                             |
| IKEv1 Phase 2     | IPSec SA                                  |
| Main Mode         | 6 messages                                |
| Aggressive Mode   | 3 messages                                |
| Quick Mode        | 3 messages                                |
| IKEv2             | IKE_SA_INIT + IKE_AUTH                    |
| IKEv2             | 4 initial messages                        |
| Child SA          | IPSec SA in IKEv2                         |
| ESP               | IP protocol 50                            |
| AH                | IP protocol 51                            |
| IKE               | UDP 500                                   |
| NAT-T             | UDP 4500                                  |
| DPD               | Detect dead peer                          |
| QCD               | Faster crash detection                    |
| XAuth             | Extended user authentication              |
| Mode Config       | Dynamic client parameters                 |
| ADVPN             | Dynamic VPN shortcuts                     |
| UDP Hole Punching | Direct NAT traversal                      |
| MSS               | Helps mitigate MTU issues                 |
| Local-In Policy   | Protects FortiGate itself                 |
| Network ID        | Segments/differentiates VPN behavior      |
| Mesh Selector     | Helps dynamic spoke-to-spoke connectivity |

```

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

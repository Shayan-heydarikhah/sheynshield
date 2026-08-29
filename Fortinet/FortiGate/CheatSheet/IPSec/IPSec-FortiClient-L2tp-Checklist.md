# 🔐 FortiGate Dial-Up IPsec VPN Checklist

## FortiClient Remote Access + L2TP over IPsec

> **SheynShield | Engineering Secure Networks**
>
> A practical FortiGate checklist for **Dial-Up IPsec VPN, FortiClient, XAuth, Mode Config, L2TP over IPsec, Windows Native VPN, NAT-T, DPD, firewall policies, and VPN troubleshooting**.

---

## 📌 Quick Navigation

* [Architecture](#-architecture)
* [Dial-Up IPsec + FortiClient](#1--dial-up-ipsec--forticlient)
* [XAuth Authentication](#2--xauth-authentication)
* [Phase 1 Checklist](#3--ipsec-phase-1-checklist)
* [Phase 2 Checklist](#4--ipsec-phase-2-checklist)
* [Mode Config](#5--mode-config)
* [Firewall Policies](#6--firewall-policy)
* [FortiClient Configuration](#7--forticlient-configuration)
* [L2TP over IPsec](#8--l2tp-over-ipsec)
* [Windows Native VPN](#9--windows-native-vpn)
* [NAT-T & DPD](#10--nat-t--dpd)
* [Troubleshooting](#11--troubleshooting)
* [FortiGate Commands](#12--fortigate-troubleshooting-commands)
* [Common Mistakes](#13--common-mistakes)
* [Production Security](#14--production-security-checklist)
* [Final Validation](#15--final-validation)
* [Quick Comparison](#-quick-comparison)

---

# 🗺️ Architecture

## Dial-Up IPsec + FortiClient

```text
                         Internet
                            |
                    +-------+-------+
                    |   FortiGate   |
                    | Dial-Up IPsec |
                    |     Server    |
                    +-------+-------+
                            |
                     IPsec Dial-Up
                            |
                    +-------+-------+
                    |  FortiClient  |
                    |   PC / User   |
                    +---------------+
```

### Core Components

```text
FortiClient
     |
     v
Dial-Up IPsec
     |
     +── IKE
     |
     +── IPsec
     |
     +── XAuth
     |
     +── Mode Config
     |
     +── Firewall Policy
     |
     v
LAN / DMZ
```

---

# 1. 🔐 Dial-Up IPsec + FortiClient

## FortiGate Configuration Checklist

Go to:

```text
VPN
└── IPsec Tunnels
    └── Create New
        └── Custom
```

### Phase 1

* [ ] Remote Gateway = `Dial-Up User`
* [ ] Incoming interface = Internet/WAN
* [ ] IKE version matches the client
* [ ] Authentication method is configured
* [ ] Pre-shared key is configured
* [ ] Peer ID is correctly defined
* [ ] XAuth is enabled when required
* [ ] Mode Config is enabled when required
* [ ] DPD is configured
* [ ] NAT-T is enabled/allowed
* [ ] Phase 1 proposal matches the client
* [ ] DH group matches the client

### Legacy Lab Example

```text
IKE Version     : IKEv1
Mode            : Aggressive
Encryption      : DES
Integrity       : MD5
DH Group        : 5
Authentication  : Pre-shared Key
Peer ID         : Any
XAuth           : Enabled
Mode Config     : Range
Auto-negotiate  : Enabled
```

> ⚠️ **Security note:** DES, MD5, and DH5 are legacy cryptographic settings. Keep them only when reproducing an old lab/interoperability scenario. For production, use modern algorithms supported by the target FortiOS/client versions.

---

# 2. 👤 XAuth Authentication

XAuth provides an additional user-authentication layer for supported dial-up IPsec designs.

## Authentication Backend

```text
                    FortiGate
                       |
                     XAuth
                       |
          +------------+------------+
          |            |            |
         LDAP          AD          FSSO
```

### Checklist

* [ ] User account exists
* [ ] User belongs to the correct group
* [ ] LDAP/AD connectivity is working
* [ ] Authentication server configuration is valid
* [ ] XAuth is enabled
* [ ] Correct user group is associated with the VPN
* [ ] Client credentials are correct
* [ ] Authentication logs are reviewed after failure

### Authentication Flow

```text
FortiClient
     |
     | IKE
     v
FortiGate
     |
     | XAuth
     v
LDAP / Active Directory
     |
     v
User / Group Validation
     |
     v
IPsec Session
```

---

# 3. 🔐 IPsec Phase 1 Checklist

Phase 1 establishes the **IKE Security Association**.

### Verify Both Sides

| Parameter      | FortiGate | FortiClient |
| -------------- | --------- | ----------- |
| IKE Version    | ☐         | ☐           |
| Authentication | ☐         | ☐           |
| PSK            | ☐         | ☐           |
| Peer ID        | ☐         | ☐           |
| Encryption     | ☐         | ☐           |
| Integrity      | ☐         | ☐           |
| DH Group       | ☐         | ☐           |
| XAuth          | ☐         | ☐           |
| DPD            | ☐         | ☐           |
| NAT-T          | ☐         | ☐           |

### Phase 1 Mental Model

```text
Client
  |
  | IKE negotiation
  v
FortiGate
  |
  +── Authentication
  +── Encryption
  +── Integrity
  +── DH Exchange
  +── Peer Identity
  |
  v
IKE SA
```

### If Phase 1 Fails

Check in this order:

* [ ] WAN reachability
* [ ] UDP/500
* [ ] UDP/4500 when NAT-T is used
* [ ] PSK
* [ ] IKE version
* [ ] Proposal
* [ ] Integrity/hash
* [ ] DH group
* [ ] Peer ID
* [ ] Authentication configuration

---

# 4. 🔐 IPsec Phase 2 Checklist

Phase 2 establishes the **IPsec Security Association** used to protect data traffic.

### Checklist

* [ ] Phase 2 proposal matches
* [ ] Encryption matches
* [ ] Integrity/authentication matches
* [ ] PFS configuration matches
* [ ] PFS DH group matches
* [ ] Traffic selectors match
* [ ] Auto-negotiate is configured when required
* [ ] Required source networks are included
* [ ] Required destination networks are included

### Lab Example

```text
Encryption      : DES
Authentication  : MD5 / SHA1
PFS              : Enabled
PFS DH Group     : 5
Auto-negotiate   : Enabled
Selectors        : Required networks
```

> ⚠️ **Production:** prefer modern AES-based encryption and SHA-2-class integrity algorithms with an appropriate DH/ECDH group.

### Phase 2 Mental Model

```text
IKE SA
  |
  v
Phase 2
  |
  +── IPsec SA
  +── ESP
  +── Traffic Selectors
  +── PFS
  |
  v
Encrypted Data Traffic
```

---

# 5. ⚙️ Mode Config

Mode Config allows the FortiGate to provide configuration information to a dial-up client.

### Possible Information

```text
Mode Config
    |
    +── VPN IP
    +── DNS
    +── Address Pool
    +── Network Parameters
    +── Client Configuration
```

## IP Pool Checklist

Example:

```text
10.10.10.10 - 10.10.10.20
```

* [ ] VPN client pool is defined
* [ ] Pool does not overlap with internal networks
* [ ] Correct netmask is configured
* [ ] Address assignment method is correct
* [ ] Client receives an address
* [ ] Assigned address is visible on the client

### Example

```text
FortiGate VPN Pool
        |
        +── 10.10.10.10
        +── 10.10.10.11
        +── 10.10.10.12
        +── ...
        +── 10.10.10.20
```

### /32 Consideration

A `/32` assignment represents a host address rather than a directly connected client subnet.

```text
Client
10.10.10.10/32
      |
      v
FortiGate
```

This can be useful in designs where client-to-client direct subnet behavior is not desired.

---

# 6. 🔥 Firewall Policy

Establishing IPsec does **not** automatically mean the VPN user can access internal resources.

The required firewall policies must exist.

## VPN → LAN / DMZ

* [ ] Incoming interface = IPsec/dial-up interface
* [ ] Outgoing interface = LAN/DMZ
* [ ] Source = VPN users/address range
* [ ] Destination = required internal networks
* [ ] Required services are allowed
* [ ] NAT = Disabled unless explicitly required
* [ ] Logging is enabled according to the security policy

```text
FortiClient
    |
    v
IPsec
    |
    v
FortiGate
    |
    +----> LAN
    |
    +----> DMZ
```

## LAN / DMZ → VPN

* [ ] Incoming interface = LAN/DMZ
* [ ] Outgoing interface = IPsec
* [ ] Source = required internal networks
* [ ] Destination = VPN client pool
* [ ] Required services are allowed
* [ ] NAT = Disabled unless explicitly required
* [ ] Logging is enabled

```text
LAN / DMZ
     |
     v
FortiGate
     |
     v
IPsec
     |
     v
FortiClient
```

---

# 7. 💻 FortiClient Configuration

On the client:

```text
FortiClient
└── VPN
    └── IPsec VPN
```

## Parameter Matching Checklist

* [ ] Server/FortiGate public IP is correct
* [ ] IKE version matches
* [ ] Phase 1 proposal matches
* [ ] Phase 2 proposal matches
* [ ] Encryption matches
* [ ] Integrity/hash matches
* [ ] DH group matches
* [ ] PFS matches
* [ ] Authentication method matches
* [ ] PSK matches
* [ ] XAuth configuration matches
* [ ] Mode Config is enabled when required
* [ ] Client receives an address from the expected pool

### Golden Rule

```text
FortiGate
    ⇅
Matching IPsec Parameters
    ⇅
FortiClient
```

If Phase 1 or Phase 2 parameters do not match, the tunnel may fail to establish.

---

# 8. 🛰️ L2TP over IPsec

## Core Concept

L2TP by itself does not provide the required cryptographic protection for secure VPN traffic.

```text
L2TP
  +
IPsec
  =
Protected L2TP Transport
```

### Architecture

```text
Windows Client
      |
      | L2TP
      |
      v
    IPsec
      |
      v
  FortiGate
      |
      v
  LAN / DMZ
```

---

## 8.1 L2TP Server Checklist

Example CLI:

```bash
config vpn l2tp
    set status enable
    set eip 10.10.10.20
    set sip 10.10.10.10
    set usrgrp test
    set enforce-ipsec enable
end
```

### Verify

* [ ] L2TP status enabled
* [ ] Start IP configured
* [ ] End IP configured
* [ ] User group configured
* [ ] `enforce-ipsec` enabled
* [ ] User exists
* [ ] User belongs to the correct group
* [ ] Address pool does not overlap with internal networks

---

# 9. 🪟 Windows Native VPN

Alternative remote-access design:

```text
VPN
└── IPsec Wizard
    └── Remote Access
        └── Windows Native / L2TP
```

## Windows Configuration

```text
Settings
└── Network & Internet
    └── VPN
        └── Add VPN
```

Configure:

```text
VPN Provider:
    Windows (built-in)

Connection Name:
    FortiGate-L2TP

Server Address:
    <FortiGate Public IP>

VPN Type:
    L2TP/IPsec with pre-shared key

Authentication:
    Username and password
```

### Checklist

* [ ] Correct public IP/FQDN
* [ ] L2TP/IPsec selected
* [ ] PSK matches
* [ ] Username is valid
* [ ] Password is valid
* [ ] User belongs to the configured group
* [ ] Client receives an IP
* [ ] VPN policy allows internal access

---

# 10. 🌐 NAT-T & DPD

Remote-access clients are frequently behind NAT.

## NAT-T

Check:

* [ ] NAT-T enabled/supported
* [ ] UDP/500 reachable
* [ ] UDP/4500 reachable
* [ ] NAT detection succeeds
* [ ] Client is not unexpectedly blocked by upstream firewall

```text
Client
   |
 NAT
   |
Internet
   |
FortiGate
```

When NAT is detected, IPsec commonly transitions to UDP encapsulation using **NAT-T / UDP 4500**.

---

## DPD

Dead Peer Detection helps detect stale/unresponsive VPN peers.

* [ ] DPD enabled when appropriate
* [ ] DPD settings are compatible with the client
* [ ] Tunnel behavior after peer failure is understood
* [ ] Logs are checked when tunnels repeatedly reconnect

---

# 11. 🐛 Troubleshooting

## Step 1 — WAN Connectivity

* [ ] Client can reach FortiGate public IP
* [ ] DNS/FQDN resolves correctly
* [ ] Upstream firewall permits VPN traffic
* [ ] UDP/500 is reachable
* [ ] UDP/4500 is reachable when NAT-T is required

---

## Step 2 — Phase 1

* [ ] IKE SA exists
* [ ] IKE version matches
* [ ] Proposal matches
* [ ] PSK matches
* [ ] DH group matches
* [ ] Peer ID matches
* [ ] Authentication succeeds

Command:

```bash
diagnose vpn ike gateway list
```

---

## Step 3 — Phase 2

* [ ] IPsec SA exists
* [ ] Selectors match
* [ ] Encryption matches
* [ ] Integrity matches
* [ ] PFS matches
* [ ] Traffic counters increase

Command:

```bash
diagnose vpn tunnel list
```

---

## Step 4 — XAuth

* [ ] Username is correct
* [ ] Password is correct
* [ ] User exists
* [ ] User group is correct
* [ ] LDAP/AD is reachable
* [ ] XAuth authentication succeeds

---

## Step 5 — Mode Config

* [ ] Mode Config is enabled
* [ ] Address pool exists
* [ ] Pool is not overlapping
* [ ] Client receives VPN IP
* [ ] DNS information is correct when required

---

## Step 6 — Firewall Policy

* [ ] VPN → LAN policy exists
* [ ] LAN → VPN policy exists when required
* [ ] Correct interfaces are selected
* [ ] Correct source/destination objects are selected
* [ ] Required services are allowed
* [ ] NAT is disabled unless intentionally required
* [ ] Policy logging is enabled as appropriate

---

## Step 7 — Routing

Check:

```bash
get router info routing-table all
```

Verify:

* [ ] Internal destination route exists
* [ ] Return route exists
* [ ] Route points through the correct interface
* [ ] No unexpected NAT is altering the source
* [ ] No more-specific route overrides the VPN path

---

# 12. 🧰 FortiGate Troubleshooting Commands

## IKE Gateway

```bash
diagnose vpn ike gateway list
```

Use it to inspect:

```text
IKE SA
Peer
Source
Destination
Proposal
Authentication
Tunnel state
```

---

## IPsec Tunnel

```bash
diagnose vpn tunnel list
```

Inspect:

```text
IPsec SA
SPI
Phase 2
Proposal
Selectors
Encryption
Authentication
Tunnel state
Traffic counters
```

---

## IKE Debug

Start:

```bash
diagnose debug reset
diagnose debug application ike -1
diagnose debug enable
```

Stop:

```bash
diagnose debug disable
diagnose debug reset
```

> ⚠️ Avoid leaving verbose debugging enabled on production systems.

---

## Flush IKE

```bash
diagnose vpn ike gateway flush
```

Use carefully because this can tear down active IKE sessions.

---

## Packet Capture

### IKE

```bash
diagnose sniffer packet any 'udp port 500 or udp port 4500' 4 0 l
```

### ESP

```bash
diagnose sniffer packet any 'ip proto 50' 4 0 l
```

### L2TP

```bash
diagnose sniffer packet any 'udp port 1701' 4 0 l
```

> Note: when L2TP is enforced through IPsec, inspect the actual encapsulation and platform behavior rather than assuming UDP/1701 will be visible in cleartext on the Internet-facing interface.

---

# 13. ⚠️ Common Mistakes

## ❌ Mistake 1 — Phase 1 Mismatch

Symptoms:

```text
IKE SA does not establish
```

Check:

* [ ] IKE version
* [ ] Encryption
* [ ] Integrity
* [ ] DH
* [ ] PSK
* [ ] Peer ID

---

## ❌ Mistake 2 — Phase 2 Mismatch

Symptoms:

```text
Phase 1 UP
Phase 2 DOWN
```

Check:

* [ ] Phase 2 proposal
* [ ] PFS
* [ ] DH group
* [ ] Traffic selectors
* [ ] Source/destination networks

---

## ❌ Mistake 3 — XAuth Failure

Symptoms:

```text
IPsec negotiation starts
but user authentication fails
```

Check:

* [ ] Username
* [ ] Password
* [ ] User group
* [ ] LDAP/AD
* [ ] XAuth configuration

---

## ❌ Mistake 4 — VPN UP but LAN Access Fails

Check:

```text
IPsec
   ↓
Mode Config
   ↓
VPN IP
   ↓
Firewall Policy
   ↓
Routing
   ↓
Return Route
```

---

## ❌ Mistake 5 — NAT Enabled

For normal VPN-to-LAN traffic:

```text
VPN Client
    |
    v
IPsec
    |
    v
LAN
```

Prefer:

```text
NAT = Disable
```

unless NAT is explicitly part of the design.

---

## ❌ Mistake 6 — Overlapping VPN Pool

Avoid:

```text
VPN Pool:
192.168.1.0/24

LAN:
192.168.1.0/24
```

Prefer a dedicated, non-overlapping VPN address space.

---

## ❌ Mistake 7 — NAT-T Not Considered

If the client is behind NAT:

```text
Client
  |
 NAT
  |
Internet
```

verify:

```text
UDP/500
UDP/4500
NAT-T
```

---

# 14. 🛡️ Production Security Checklist

The lab may use legacy cryptographic parameters, but production VPN deployments should be hardened.

### Cryptography

* [ ] Avoid DES
* [ ] Avoid 3DES where possible
* [ ] Avoid MD5
* [ ] Avoid weak DH groups
* [ ] Use modern AES-based encryption
* [ ] Use SHA-2-class integrity where supported
* [ ] Use an appropriate modern DH/ECDH group
* [ ] Use strong, unique PSKs
* [ ] Prefer certificate-based authentication where practical

### Identity

* [ ] Use dedicated VPN user groups
* [ ] Apply least privilege
* [ ] Integrate with centralized identity where appropriate
* [ ] Avoid shared user accounts
* [ ] Review inactive VPN users regularly

### Network Security

* [ ] Use dedicated VPN address pools
* [ ] Prevent address overlap
* [ ] Restrict VPN-to-LAN access
* [ ] Restrict VPN-to-DMZ access
* [ ] Disable unnecessary services
* [ ] Enable appropriate logging
* [ ] Monitor authentication failures

### Operational Security

* [ ] Document VPN proposals
* [ ] Document address pools
* [ ] Document user groups
* [ ] Document firewall policies
* [ ] Test failover/reconnect behavior
* [ ] Review VPN logs
* [ ] Keep FortiOS/FortiClient versions supported

---

# 15. 🧪 Final Validation Checklist

## IPsec

* [ ] Phase 1 = UP
* [ ] Phase 2 = UP
* [ ] Correct peer identified
* [ ] Correct proposal negotiated
* [ ] Correct selectors installed

## Authentication

* [ ] XAuth succeeds when used
* [ ] User authentication succeeds
* [ ] Correct user group is applied
* [ ] LDAP/AD authentication works when configured

## Mode Config

* [ ] VPN client receives an IP
* [ ] Assigned IP is from expected pool
* [ ] DNS configuration is correct
* [ ] No pool overlap exists

## Firewall

* [ ] VPN → LAN policy exists
* [ ] LAN → VPN policy exists when required
* [ ] NAT is intentionally configured
* [ ] Logging is enabled appropriately

## Routing

* [ ] Destination routes exist
* [ ] Return routes exist
* [ ] VPN traffic uses the expected path
* [ ] No unexpected WAN route wins

## Connectivity

* [ ] VPN client can ping allowed resources
* [ ] DNS works if required
* [ ] Required TCP/UDP services work
* [ ] Unauthorized resources remain inaccessible

---

# 🧠 Troubleshooting Decision Tree

```text
                VPN Connection
                      |
                      v
              Phase 1 Established?
                 /          \
               NO            YES
               |              |
          Check IKE       Phase 2 UP?
          PSK/Proposal      /      \
          ID/DH           NO        YES
                           |          |
                     Check P2      XAuth?
                     Selectors      /   \
                     PFS           NO    YES
                                  |       |
                             Check User   Mode Config?
                                          /      \
                                        NO        YES
                                        |          |
                                   Check Pool     Firewall?
                                                  /     \
                                                NO       YES
                                                |         |
                                           Policy/NAT   Routing
                                                        |
                                                        v
                                                   Connectivity
```

---

# 📊 Quick Comparison

| Feature                 | Dial-Up IPsec + FortiClient                              | L2TP over IPsec                                 |
| ----------------------- | -------------------------------------------------------- | ----------------------------------------------- |
| Remote Access           | ✅                                                        | ✅                                               |
| IPsec                   | ✅                                                        | ✅                                               |
| L2TP                    | ❌                                                        | ✅                                               |
| FortiClient             | ✅                                                        | Not required                                    |
| Native Windows VPN      | ❌                                                        | ✅                                               |
| XAuth                   | Commonly used                                            | User authentication                             |
| Mode Config             | ✅                                                        | Supported by design                             |
| Client IP Pool          | ✅                                                        | ✅                                               |
| NAT-T                   | ✅                                                        | ✅                                               |
| DPD                     | Recommended                                              | Recommended                                     |
| Firewall Policy         | Required                                                 | Required                                        |
| Routing                 | Required                                                 | Required                                        |
| Client Isolation        | Design dependent                                         | Can be influenced by address assignment         |
| Modern Preferred Design | FortiClient/IPsec or current supported remote-access VPN | Use only where required by compatibility/design |

---

# 🔬 Protocol Relationship

```text
                    Remote Access VPN
                           |
              +------------+------------+
              |                         |
       FortiClient IPsec          L2TP over IPsec
              |                         |
             IKE                       L2TP
              |                         |
           IPsec                      IPsec
              |                         |
           XAuth                    Authentication
              |                         |
        Mode Config                IP Assignment
              |                         |
              +------------+------------+
                           |
                      FortiGate
                           |
                      Firewall Policy
                           |
                        Routing
                           |
                       LAN / DMZ
```

---

# 🎯 One-Line Mental Model

```text
Dial-Up IPsec
      ↓
IKE Phase 1
      ↓
IPsec Phase 2
      ↓
XAuth / Authentication
      ↓
Mode Config
      ↓
VPN Client IP
      ↓
Firewall Policy
      ↓
Routing
      ↓
LAN / DMZ Access
```

For L2TP:

```text
Windows Client
      ↓
L2TP
      ↓
IPsec
      ↓
FortiGate
      ↓
Authentication
      ↓
VPN Address
      ↓
Firewall Policy
      ↓
LAN / DMZ
```

---

# 🏆 Key Takeaways

> **Dial-up IPsec is designed for remote peers whose public addresses may be dynamic.**

> **IKE negotiates the security relationship; IPsec protects the data plane.**

> **XAuth can provide user-level authentication in supported dial-up IPsec designs.**

> **Mode Config can dynamically provide VPN client addressing and other configuration information.**

> **NAT-T is especially important when remote VPN clients are behind NAT.**

> **IPsec being UP does not guarantee LAN access; firewall policy and routing still have to permit the traffic.**

> **L2TP should be protected by IPsec rather than treated as an encryption protocol by itself.**

> **For troubleshooting, isolate the failure domain: WAN → Phase 1 → Phase 2 → Authentication → Mode Config → Policy → Routing → Application.**

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

## 🏷️ Keywords

`FortiGate` · `Fortinet` · `Dial-Up IPsec` · `IPsec VPN` · `FortiClient VPN` · `L2TP over IPsec` · `Remote Access VPN` · `XAuth` · `Mode Config` · `NAT-T` · `DPD` · `IKEv1` · `IKEv2` · `IPsec Phase 1` · `IPsec Phase 2` · `FortiGate Troubleshooting` · `FortiClient Troubleshooting` · `Windows Native VPN` · `LDAP` · `Active Directory` · `VPN Firewall Policy` · `FortiGate CLI` · `Network Security` · `Fortinet NSE` · `VPN Security`

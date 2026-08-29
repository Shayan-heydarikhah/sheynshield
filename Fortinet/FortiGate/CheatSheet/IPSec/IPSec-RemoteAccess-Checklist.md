# 🔐 FortiGate Remote Access VPN — Checklist

> **FortiGate Remote Access VPN Cheat Sheet & Troubleshooting Checklist**
>
> Covers **IPsec Dialup, FortiClient VPN, L2TP over IPsec, Split Tunnel, Full Tunnel, Dialup Internet VPN, X.509 Certificate Authentication, XAuth, Mode Config, IKE Debugging and FNBAMD Troubleshooting**.

---

## 📌 Quick Navigation

* [ ] [Remote Access VPN Models](#-remote-access-vpn-models)
* [ ] [FortiClient Dialup IPsec](#1--forticlient-dialup-ipsec)
* [ ] [Mode Config & Split Tunnel](#2--mode-config--split-tunnel)
* [ ] [VPN Firewall Policies](#3--remote-access-vpn-policies)
* [ ] [L2TP over IPsec](#4--l2tp-over-ipsec)
* [ ] [Dialup Internet VPN](#5--dialup-internet-vpn)
* [ ] [Dialup Internet Routing](#6--dialup-internet-routing)
* [ ] [Certificate-Based VPN](#7--certificate-based-ipsec-vpn)
* [ ] [IKE Troubleshooting](#8--ike-troubleshooting)
* [ ] [FNBAMD Authentication Debugging](#9--fnbamd-authentication-troubleshooting)
* [ ] [End-to-End Troubleshooting](#10--remote-access-vpn-troubleshooting-flow)
* [ ] [Security Hardening](#11--security-hardening-checklist)
* [ ] [Exam & Interview Checklist](#12--exam--interview-checklist)
* [ ] [Quick Command Reference](#13--quick-command-reference)

---

# 🎯 Remote Access VPN Models

| VPN Model           | Client            | Primary Use                         |
| ------------------- | ----------------- | ----------------------------------- |
| IPsec Dialup        | FortiClient       | Remote-user VPN                     |
| L2TP over IPsec     | Native OS VPN     | Legacy/native OS VPN                |
| Split Tunnel        | FortiClient       | Corporate resources through VPN     |
| Full Tunnel         | FortiClient       | Corporate + Internet through VPN    |
| Dialup Internet VPN | FortiClient/IPsec | Internet breakout through FortiGate |
| Certificate VPN     | FortiClient       | Certificate-based identity          |
| VIP + WAF           | Web browser       | Published applications              |

---

# 1️⃣ FortiClient Dialup IPsec

## Phase 1 Checklist

* [ ] Create **IPsec Tunnel**
* [ ] Select **Custom**
* [ ] Select **Dialup User**
* [ ] Select correct WAN/interface
* [ ] Configure authentication
* [ ] Configure IKE version
* [ ] Configure proposal
* [ ] Configure DH group
* [ ] Configure XAuth when required
* [ ] Configure user group

### Basic Configuration

```text
VPN Type:
    IPsec

Connection:
    Dialup User

Incoming Interface:
    ISP / WAN

IKE Version:
    IKEv1

Mode:
    Aggressive

Peer ID:
    Any

Authentication:
    Pre-shared Key

XAuth:
    Enable when required
```

### Proposal

```text
Encryption:
    DES

Authentication:
    MD5

DH Group:
    5
    or
    2
```

> ⚠️ **Security note:** DES/MD5/IKEv1/Aggressive Mode are legacy examples. For production deployments, use current FortiOS/FortiClient-supported cryptographic suites and authentication methods appropriate to your security policy.

---

## XAuth Checklist

* [ ] XAuth enabled
* [ ] Authentication method configured
* [ ] User group selected
* [ ] LDAP/RADIUS/AD integration verified
* [ ] User exists
* [ ] User belongs to the expected group
* [ ] Authentication server reachable
* [ ] Authentication logs reviewed

```text
FortiClient
    │
    ▼
IKE Authentication
    │
    ▼
XAuth
    │
    ▼
User Authentication
    │
    ▼
User Group
    │
    ▼
VPN Access
```

---

# 2️⃣ Mode Config & Split Tunnel

## Mode Config Checklist

* [ ] Enable Mode Config
* [ ] Configure client IP assignment
* [ ] Configure DNS when required
* [ ] Configure split tunneling when required
* [ ] Verify client receives expected network parameters

```text
Mode Config
    │
    ├── Client IP
    ├── DNS
    ├── Split Tunnel
    └── Other VPN parameters
```

---

## Split Include

Use **Split Include** when only selected corporate networks should traverse the VPN.

```bash
config vpn ipsec phase1-interface
    edit "dialup-vpn"
        set ipv4-split-include "CORPORATE-NETWORK"
    next
end
```

### Checklist

* [ ] Corporate subnet defined
* [ ] Split-include object configured
* [ ] Client receives the expected route
* [ ] Internet remains on local ISP
* [ ] Corporate traffic enters VPN
* [ ] FortiGate policy allows corporate traffic

```text
Client
   │
   ├── Internet
   │      └── Home ISP
   │
   └── Corporate Network
          └── IPsec VPN
```

---

## Split Exclude

Use **Split Exclude** when most traffic should use the VPN except selected destinations.

```text
Client
   │
   ▼
IPsec VPN
   │
   ├── Corporate Traffic
   │
   └── Excluded Destinations
           │
           ▼
        Local ISP
```

### Checklist

* [ ] Split-exclude destinations identified
* [ ] Excluded networks are correct
* [ ] Client routing table verified
* [ ] Internet behavior tested
* [ ] Corporate traffic tested

---

## Split Include Service

Service-based split tunneling can be used when supported by the FortiOS/FortiClient design.

```bash
set split-include-service
```

Concept:

```text
Client
  │
  ├── TCP/443 ─────► VPN
  ├── TCP/22 ──────► VPN
  └── Other Traffic ─► Local ISP
```

### Checklist

* [ ] Required services identified
* [ ] Service objects verified
* [ ] Client routing behavior tested
* [ ] FortiClient behavior validated

---

# 3️⃣ Remote Access VPN Policies

## VPN → Internal Network

* [ ] Incoming interface = Dialup IPsec
* [ ] Outgoing interface = LAN/DMZ
* [ ] Source = VPN user/group
* [ ] Destination = required internal networks
* [ ] Services restricted
* [ ] NAT disabled unless intentionally required
* [ ] Logging enabled

Recommended model:

```text
Source:
    VPN User Group

Destination:
    Required Internal Networks

Service:
    Required Services

NAT:
    Disable

Logging:
    Enable
```

### Security Principle

> **Do not automatically equate "VPN user" with "trusted internal user."**

Use least privilege:

```text
VPN User
   │
   ▼
User Group
   │
   ▼
Specific Network
   │
   ▼
Specific Service
```

---

## ⚠️ Common Policy Mistake

Avoid unintentionally creating:

```text
Incoming:
    Dialup IPsec

Source:
    ALL

Destination:
    ALL

Service:
    ALL
```

Unless broad access is explicitly required.

### Review

* [ ] Source restricted
* [ ] Destination restricted
* [ ] Services restricted
* [ ] NAT behavior intentional
* [ ] Logging enabled
* [ ] Policy order verified

---

# 4️⃣ L2TP over IPsec

> L2TP over IPsec provides a native OS VPN model, but it should be treated as a legacy/compatibility design where appropriate.

## L2TP Configuration

```bash
config vpn l2tp
    set status enable
    set eip 10.10.10.20
    set sip 10.10.10.10
    set usrgrp "test"
    set enforce-ipsec enable
end
```

### Checklist

* [ ] L2TP enabled
* [ ] Client address range configured
* [ ] User group configured
* [ ] IPsec enforcement enabled
* [ ] Authentication verified
* [ ] Client configuration verified

---

## L2TP Phase 1

```text
Connection:
    Dialup

IKE Version:
    IKEv1

Mode:
    Main Mode

Peer ID:
    Any

Authentication:
    PSK

Encryption:
    DES

Authentication Algorithm:
    MD5 / SHA1

DH Group:
    2
```

### Checklist

* [ ] Dialup connection configured
* [ ] IKE version matches
* [ ] Main Mode configured
* [ ] PSK matches
* [ ] Proposal matches
* [ ] DH group matches
* [ ] User group matches

---

## L2TP Phase 2

```text
Encryption:
    DES

Authentication:
    MD5 / SHA1

PFS:
    Disable

DH:
    Group 2

Selectors:
    Required networks

Auto-negotiate:
    Enable
```

### Checklist

* [ ] Phase 2 proposal matches
* [ ] PFS matches
* [ ] Selectors verified
* [ ] Auto-negotiate configured when required

---

## L2TP Firewall Policy

### L2TP → LAN

* [ ] Incoming = L2TP interface
* [ ] Outgoing = LAN/DMZ
* [ ] Source restricted to L2TP users
* [ ] Destination restricted
* [ ] NAT disabled unless required
* [ ] Logging enabled

### LAN → L2TP

* [ ] Return traffic permitted
* [ ] Correct destination interface
* [ ] Routing verified
* [ ] Policy order verified

---

# 5️⃣ Dialup Internet VPN

## Objective

Force remote-user Internet traffic through the FortiGate:

```text
Remote Client
      │
      ▼
   IPsec VPN
      │
      ▼
   FortiGate
      │
      ▼
     ISP
      │
      ▼
  Internet
```

---

## Hub Phase 1

### Checklist

* [ ] IPsec Custom tunnel created
* [ ] Connection = Dialup
* [ ] WAN interface selected
* [ ] IKE version configured
* [ ] Authentication configured
* [ ] XAuth configured when required
* [ ] User group selected
* [ ] Proposal verified
* [ ] DH group verified

---

## Hub Phase 2

* [ ] Selectors configured
* [ ] Encryption verified
* [ ] Authentication verified
* [ ] PFS verified
* [ ] Auto-negotiate configured when required

---

## Hub VPN → Internet Policy

```text
Incoming:
    dial-ipsec

Outgoing:
    ISP

Source:
    VPN Users / Required Address Objects

Destination:
    ALL

Service:
    Required Services

NAT:
    Enable

Logging:
    Enable
```

### Checklist

* [ ] VPN → WAN policy exists
* [ ] NAT enabled intentionally
* [ ] Correct outgoing interface
* [ ] Internet policy placed correctly
* [ ] DNS works
* [ ] Return traffic works
* [ ] Public IP is the FortiGate WAN IP

---

# 6️⃣ Dialup Internet Routing

## ⚠️ Critical Routing Concept

The most common design mistake is forgetting that **the VPN peer's public IP must remain reachable outside the VPN tunnel**.

Example:

```text
FGT-2
  │
  ├── Hub Public IP
  │       └── Local ISP
  │
  └── 0.0.0.0/0
          └── IPsec Tunnel
```

---

## Spoke Routing Checklist

* [ ] Route to Hub public IP uses local ISP
* [ ] Default route points to IPsec tunnel when full tunnel is required
* [ ] No recursive routing problem
* [ ] Hub public IP remains reachable
* [ ] Spoke LAN routes are correct
* [ ] Return routes exist

### Desired Logic

```text
                 FGT-2
                   │
          ┌────────┴────────┐
          │                 │
    Hub Public IP       Internet
          │                 │
       Local ISP        IPsec VPN
                            │
                            ▼
                           HUB
                            │
                            ▼
                           ISP
                            │
                            ▼
                         Internet
```

---

## Hub Routing Checklist

* [ ] Route to spoke public IP uses ISP
* [ ] Spoke LAN route uses IPsec
* [ ] Return traffic is routable
* [ ] Static route or dynamic routing is configured
* [ ] No asymmetric routing
* [ ] Policy lookup verified

---

## Dynamic Routing Option

For larger deployments:

* [ ] OSPF considered
* [ ] BGP considered
* [ ] Route advertisement verified
* [ ] Return path verified

---

# 7️⃣ Certificate-Based IPsec VPN

Certificate authentication can replace PSK:

```text
FortiClient
    │
    │ X.509 Certificate
    ▼
 FortiGate
    │
    ├── CA Validation
    ├── Subject
    ├── SAN
    └── Certificate Identity
```

## Certificate Checklist

* [ ] CA certificate imported
* [ ] Client certificate issued
* [ ] Private key available to client
* [ ] Certificate is valid
* [ ] Certificate is not expired
* [ ] Certificate chain is trusted
* [ ] Subject/SAN requirements verified
* [ ] Certificate identity mapping verified
* [ ] FortiGate trusts the issuing CA
* [ ] FortiClient has correct certificate selected

---

## Certificate Troubleshooting

Check:

* [ ] Certificate validity
* [ ] Certificate chain
* [ ] CA trust
* [ ] Subject
* [ ] SAN
* [ ] Peer identity
* [ ] Certificate selection
* [ ] IKE authentication logs

---

# 8️⃣ IKE Troubleshooting

Use IKE debugging when Phase 1 fails.

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

## IKE Debug Checklist

### Phase 1

* [ ] IKE version
* [ ] Mode
* [ ] Peer ID
* [ ] Local gateway
* [ ] Remote gateway
* [ ] Authentication
* [ ] PSK
* [ ] Certificate
* [ ] Encryption
* [ ] Integrity
* [ ] DH group
* [ ] NAT-T
* [ ] DPD

### Phase 2

* [ ] Encryption
* [ ] Integrity
* [ ] PFS
* [ ] Proxy IDs/selectors
* [ ] Auto-negotiate
* [ ] Lifetime

---

## IKE Negotiation Model

```text
Client
  │
  ▼
IKE Phase 1
  │
  ├── Proposal
  ├── Authentication
  ├── DH
  └── Identity
        │
        ▼
      IKE SA
        │
        ▼
   IKE Phase 2
        │
        ├── Proposal
        ├── PFS
        └── Selectors
              │
              ▼
           IPsec SA
              │
              ▼
            Traffic
```

---

# 9️⃣ FNBAMD Authentication Troubleshooting

Use FNBAMD debugging for authentication-related problems.

```bash
diagnose debug application fnbamd -1
diagnose debug enable
```

### Useful for

* [ ] XAuth
* [ ] LDAP
* [ ] RADIUS
* [ ] User authentication
* [ ] User groups
* [ ] Authentication servers
* [ ] Authentication failures

```text
VPN Client
    │
    ▼
XAuth
    │
    ▼
FNBAMD
    │
    ├── LDAP
    ├── RADIUS
    └── Local Authentication
```

> **FNBAMD** is the FortiGate authentication daemon used for authentication-related processing.

---

# 🔟 Remote Access VPN Troubleshooting Flow

```text
                VPN FAILURE
                    │
                    ▼
          Internet Reachability
                    │
              ┌─────┴─────┐
              │           │
             FAIL        PASS
              │           │
              ▼           ▼
          Fix WAN      IKE Phase 1
                          │
                          ▼
                    Phase 1 SA?
                          │
                     ┌────┴────┐
                     │         │
                    NO        YES
                     │         │
                     ▼         ▼
                IKE Debug   Phase 2
                               │
                               ▼
                         IPsec SA?
                               │
                          ┌────┴────┐
                          │         │
                         NO        YES
                          │         │
                          ▼         ▼
                     Proposal   XAuth /
                     Selectors  Certificate
                                    │
                                    ▼
                              Mode Config
                                    │
                                    ▼
                                Routing
                                    │
                                    ▼
                               Firewall
                                    │
                                    ▼
                                NAT
                                    │
                                    ▼
                              Destination
```

---

# 🧪 Diagnostic Checklist

## Tunnel Status

```bash
diagnose vpn tunnel list
```

* [ ] Phase 2 SA exists
* [ ] Correct selectors
* [ ] SPI values present
* [ ] Traffic counters increasing
* [ ] Tunnel state correct

---

## IKE Gateway

```bash
diagnose vpn ike gateway list
```

* [ ] IKE SA exists
* [ ] Peer address correct
* [ ] IKE version correct
* [ ] Proposal correct
* [ ] Authentication successful
* [ ] Tunnel established

---

## Authentication

```bash
diagnose debug application fnbamd -1
diagnose debug enable
```

* [ ] User authentication successful
* [ ] LDAP/RADIUS reachable
* [ ] User group correct
* [ ] XAuth successful

---

# 1️⃣1️⃣ Security Hardening Checklist

## Authentication

* [ ] Prefer certificate-based authentication where appropriate
* [ ] Use strong authentication
* [ ] Integrate with centralized identity where required
* [ ] Avoid shared credentials
* [ ] Restrict VPN users to required groups

## Firewall

* [ ] Avoid unnecessary `Source = ALL`
* [ ] Avoid unnecessary `Destination = ALL`
* [ ] Restrict services
* [ ] Restrict destination networks
* [ ] Enable appropriate logging
* [ ] Review policy order

## Cryptography

* [ ] Avoid obsolete cryptographic algorithms in new deployments
* [ ] Prefer modern encryption
* [ ] Prefer strong integrity algorithms
* [ ] Use appropriate DH groups
* [ ] Review IKE version requirements
* [ ] Review vendor/version compatibility

## Routing

* [ ] Verify default route
* [ ] Verify VPN peer reachability
* [ ] Prevent recursive routing
* [ ] Verify return path
* [ ] Check asymmetric routing
* [ ] Verify NAT

---

# 1️⃣2️⃣ Exam & Interview Checklist

> ### 🧠 Know These Concepts

* [ ] **IPsec Dialup** is designed for remote-user VPN access.
* [ ] **Mode Config** can provide client-side VPN parameters.
* [ ] **Split Include** sends selected destinations through VPN.
* [ ] **Split Exclude** sends traffic through VPN except selected destinations.
* [ ] **Full Tunnel** sends Internet and corporate traffic through the VPN.
* [ ] **Dialup Internet VPN** requires correct routing and NAT.
* [ ] The VPN peer's public IP must remain reachable outside the VPN tunnel.
* [ ] **XAuth** provides an additional user-authentication mechanism in applicable IKEv1 designs.
* [ ] **Certificate VPN** uses X.509 identity and CA validation.
* [ ] **L2TP over IPsec** combines L2TP with IPsec protection.
* [ ] **IKE Phase 1** establishes the IKE SA.
* [ ] **IKE Phase 2** establishes IPsec SAs.
* [ ] Firewall policy does not automatically determine client routing.
* [ ] Split tunneling changes which destinations the client sends into the VPN.
* [ ] Full-tunnel VPN requires correct Internet breakout and return routing.
* [ ] `fnbamd` is useful for authentication troubleshooting.
* [ ] IKE debug is useful when Phase 1 negotiation fails.

---

# 1️⃣3️⃣ Quick Command Reference

## IPsec Tunnel

```bash
diagnose vpn tunnel list
```

## IKE Gateway

```bash
diagnose vpn ike gateway list
```

## IKE Debug

```bash
diagnose debug enable
diagnose debug application ike -1
```

## Stop Debug

```bash
diagnose debug disable
diagnose debug reset
```

## FNBAMD Debug

```bash
diagnose debug application fnbamd -1
diagnose debug enable
```

## L2TP Status

```bash
diagnose vpn l2tp status
```

## L2TP Debug

```bash
diagnose debug enable
```

---

# 🧠 Remote Access VPN Mental Model

```text
                   REMOTE USER
                        │
                        ▼
                   FortiClient
                        │
                        ▼
                   Internet
                        │
                        ▼
                ┌───────────────┐
                │   FortiGate   │
                └───────┬───────┘
                        │
              ┌─────────┼─────────┐
              │         │         │
             IKE      XAuth    Certificate
              │         │         │
              └─────────┼─────────┘
                        │
                    IPsec SA
                        │
                 ┌──────┴──────┐
                 │             │
            Split Tunnel    Full Tunnel
                 │             │
                 ▼             ▼
             Corporate      Internet
              Network        via HQ
```

---

# 🔥 Production Validation Checklist

Before declaring a remote-access VPN production-ready:

* [ ] Internet reachability verified
* [ ] Phase 1 verified
* [ ] Phase 2 verified
* [ ] Authentication verified
* [ ] Mode Config verified
* [ ] Client IP assignment verified
* [ ] Split/Full Tunnel behavior verified
* [ ] Firewall policies verified
* [ ] NAT behavior verified
* [ ] DNS resolution verified
* [ ] Internal application access verified
* [ ] Internet access verified if full tunnel
* [ ] Return routing verified
* [ ] VPN peer public-IP route verified
* [ ] NAT-T verified where applicable
* [ ] Logging enabled
* [ ] Troubleshooting commands tested
* [ ] Security policy reviewed
* [ ] Unnecessary VPN access removed

---

# ⚡ 30-Second Troubleshooting Checklist

```text
VPN DOWN?
   │
   ├── Internet reachable?
   │
   ├── Phase 1 UP?
   │
   ├── Phase 2 UP?
   │
   ├── XAuth / Certificate OK?
   │
   ├── Mode Config OK?
   │
   ├── Client route correct?
   │
   ├── Firewall policy matched?
   │
   ├── NAT correct?
   │
   ├── Return route exists?
   │
   └── Destination reachable?
```

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

## 🔎 Keywords

`FortiGate Remote Access VPN` · `FortiClient IPsec VPN` · `FortiGate Dialup VPN` · `FortiGate L2TP over IPsec` · `FortiGate Split Tunnel` · `FortiGate Full Tunnel` · `FortiGate Mode Config` · `FortiGate XAuth` · `FortiGate Certificate VPN` · `FortiGate IKE Troubleshooting` · `FortiGate FNBAMD` · `FortiGate VPN Troubleshooting` · `Fortinet IPsec VPN`

---

> **SheynShield Engineering Note**
>
> Remote-access VPN troubleshooting should never stop at **"the tunnel is UP."**
>
> Always validate the complete chain:
>
> **IKE → IPsec SA → Authentication → Mode Config → Client Routing → Firewall Policy → NAT → Return Path → Application**
>
> A tunnel can be **UP** while the application is still **DOWN**.

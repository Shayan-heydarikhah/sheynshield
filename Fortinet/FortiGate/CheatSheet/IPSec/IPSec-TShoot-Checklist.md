# 🔐 FortiGate IPsec Troubleshooting Checklist

> **FortiGate IPsec VPN Troubleshooting — IKE, Phase 2, NPU/ASIC, NAT-T, MTU, DSCP, Mode Config, FQDN, EAP & Debug Commands**
>
> A practical FortiGate IPsec troubleshooting checklist for network engineers, security engineers, NSE candidates, and FortiGate administrators.

---

## 📑 Table of Contents

- [🎯 Troubleshooting Methodology](#-troubleshooting-methodology)
- [1. IKE Status](#1-ike-status)
- [2. Phase 1 Troubleshooting](#2-phase-1-troubleshooting)
- [3. Phase 2 Troubleshooting](#3-phase-2-troubleshooting)
- [4. Traffic Troubleshooting](#4-traffic-troubleshooting)
- [5. IKE Debug](#5-ike-debug)
- [6. IKE Log Filtering](#6-ike-log-filtering)
- [7. IPsec Counters](#7-ipsec-counters)
- [8. NPU / ASIC Offload](#8-npu--asic-offload)
- [9. IPsec Fragmentation](#9-ipsec-fragmentation)
- [10. DSCP / DiffServ](#10-dscp--diffserv)
- [11. Dialup IPsec / Mode Config](#11-dialup-ipsec--mode-config)
- [12. DNS / FQDN Troubleshooting](#12-dns--fqdn-troubleshooting)
- [13. IKEv2 EAP](#13-ikev2-eap)
- [14. MTU Troubleshooting](#14-mtu-troubleshooting)
- [15. NPU Troubleshooting Flow](#15-npu-troubleshooting-flow)
- [16. Complete Troubleshooting Flow](#16-complete-troubleshooting-flow)
- [17. Phase 1 Checklist](#17-phase-1-checklist)
- [18. Phase 2 Checklist](#18-phase-2-checklist)
- [19. Traffic Checklist](#19-traffic-checklist)
- [20. Final Validation](#20-final-validation)
- [⚡ Command Reference](#-command-reference)
- [🧠 Golden Rules](#-golden-rules)
- [🔗 SheynShield Resources](#-sheynshield-resources)

---

# 🎯 Troubleshooting Methodology

The most important rule in FortiGate IPsec troubleshooting is:

```text
Do NOT troubleshoot everything at once.
````

Troubleshoot from the lowest failing layer upward:

```text
                    IPsec Problem
                         │
                         ▼
                  IKE Daemon Status
                         │
                         ▼
                    Phase 1 / IKE SA
                         │
                         ▼
                  Phase 2 / IPsec SA
                         │
                         ▼
                      Routing
                         │
                         ▼
                   Firewall Policy
                         │
                         ▼
                        NAT
                         │
                         ▼
                      Selectors
                         │
                         ▼
                  Traffic Counters
                         │
                         ▼
                     MTU / MSS
                         │
                         ▼
                    NPU / ASIC
                         │
                         ▼
                       DONE
```

> [!IMPORTANT]
> **Fix the first failing layer before moving to the next layer.**

---

# 1. IKE Status

## ✅ High-Level IKE Check

```bash
diagnose vpn ike status
```

Use this as an initial health check for the IKE subsystem.

### Checklist

* [ ] IKE daemon is operational
* [ ] No obvious IKE subsystem issue
* [ ] Expected IKE processing state exists
* [ ] Continue to gateway-level troubleshooting

---

# 2. Phase 1 Troubleshooting

## 🔎 Check IKE Gateway

```bash
diagnose vpn ike gateway list
```

Check:

* [ ] Local address
* [ ] Remote address
* [ ] Tunnel ID
* [ ] Creation time
* [ ] Establishment time
* [ ] IKE SA
* [ ] IPsec SA
* [ ] IKE version
* [ ] Negotiated proposal
* [ ] Authentication state

---

## Phase 1 Validation Checklist

### 🌐 Connectivity

* [ ] Remote gateway is reachable
* [ ] Correct WAN interface is selected
* [ ] UDP/500 is reachable
* [ ] UDP/4500 is reachable when NAT-T is used
* [ ] Intermediate firewall allows IKE traffic

### 🔐 Authentication

* [ ] Authentication method matches
* [ ] PSK matches
* [ ] Peer ID matches
* [ ] Local ID is correct
* [ ] Remote ID is correct
* [ ] Certificate is valid when certificates are used
* [ ] Certificate identity/SAN is correct

### 🔒 Cryptography

* [ ] IKE version matches
* [ ] Encryption proposal matches
* [ ] Integrity/hash matches
* [ ] DH group matches
* [ ] Authentication method matches

### ⚙️ IKE Behavior

* [ ] DPD configuration is compatible
* [ ] NAT-T behavior is correct
* [ ] Rekey settings are compatible
* [ ] Local/remote gateway configuration is correct

---

# 3. Phase 2 Troubleshooting

## 🔎 Check IPsec Tunnel

```bash
diagnose vpn tunnel list
```

Useful information includes:

* [ ] Phase 2 status
* [ ] Tunnel ID
* [ ] IKE version
* [ ] Local networks
* [ ] Remote networks
* [ ] Proxy IDs / selectors
* [ ] DPD state
* [ ] MTU
* [ ] NPU information
* [ ] SA information

---

## Phase 2 Checklist

* [ ] Phase 1 is UP
* [ ] Phase 2 proposal matches
* [ ] Encryption matches
* [ ] Authentication/integrity matches
* [ ] PFS configuration matches
* [ ] DH group matches
* [ ] Local selector matches
* [ ] Remote selector matches
* [ ] Protocol selector matches
* [ ] Auto-negotiate is configured when required

---

## 🚨 Selector Mismatch

A common failure is:

```text
FortiGate-A

Local:
192.168.10.0/24

Remote:
192.168.20.0/24
```

while the peer expects:

```text
Local:
192.168.20.0/24

Remote:
192.168.10.0/24
```

The selectors must represent the same traffic domains from opposite perspectives.

```text
FGT-A                         FGT-B

192.168.10.0/24  ──────────► 192.168.20.0/24
192.168.20.0/24  ◄────────── 192.168.10.0/24
```

### Checklist

* [ ] Local selector correct
* [ ] Remote selector correct
* [ ] Selector direction understood
* [ ] Protocol/port selectors match when used
* [ ] NAT does not unexpectedly change the expected traffic domain

---

# 4. Traffic Troubleshooting

If:

```text
Phase 1 = UP
Phase 2 = UP
```

but traffic does not pass, move to the **data plane**.

```text
Phase 2 UP
    │
    ▼
Check Route
    │
    ▼
Check Firewall Policy
    │
    ▼
Check NAT
    │
    ▼
Check Selectors
    │
    ▼
Check Counters
    │
    ▼
Check Packet Capture
```

---

## Routing Checklist

* [ ] Local route exists
* [ ] Remote network route exists
* [ ] Route points to the correct IPsec interface
* [ ] Administrative distance is correct
* [ ] Route priority is correct
* [ ] Return route exists
* [ ] No more-specific route overrides the VPN route
* [ ] Policy route/PBR is not redirecting traffic unexpectedly
* [ ] SD-WAN rules are not selecting another path

---

## Firewall Policy Checklist

* [ ] Incoming interface is correct
* [ ] Outgoing interface is correct
* [ ] Source address is correct
* [ ] Destination address is correct
* [ ] Service is correct
* [ ] User/group restrictions are correct
* [ ] Policy order is correct
* [ ] Logging is enabled during troubleshooting
* [ ] NAT is configured intentionally

---

## NAT Checklist

* [ ] NAT is disabled for normal site-to-site traffic when required
* [ ] Source NAT is intentional
* [ ] IP Pool configuration is correct
* [ ] VIP/DNAT is correct when used
* [ ] NAT does not conflict with Phase 2 selectors
* [ ] Return traffic follows the expected path

---

# 5. IKE Debug

## 🔥 Enable IKE Debug

```bash
diagnose debug enable
diagnose debug application ike -1
```

Use this when Phase 1 negotiation fails or behaves unexpectedly.

### Look for

* [ ] IKE version mismatch
* [ ] Proposal mismatch
* [ ] PSK mismatch
* [ ] Peer ID mismatch
* [ ] Authentication failure
* [ ] DH mismatch
* [ ] Certificate failure
* [ ] DPD behavior
* [ ] NAT-T negotiation
* [ ] IKE state machine
* [ ] Rekey events

---

## Stop Debug

```bash
diagnose debug disable
```

Optionally reset debug settings:

```bash
diagnose debug reset
```

> [!WARNING]
> Do not leave verbose debugging enabled longer than necessary, especially on production FortiGates.

---

# 6. IKE Log Filtering

When multiple VPN tunnels exist, reduce unrelated debug output.

Example:

```bash
diagnose vpn ike log-filter src-addr4 12.23.34.1
```

Then:

```bash
diagnose debug enable
diagnose debug application ike -1
```

### Checklist

* [ ] Correct source IP selected
* [ ] Correct tunnel identified
* [ ] IKE debug enabled
* [ ] Negotiation reproduced
* [ ] Failure point identified
* [ ] Debug disabled after testing

---

# 7. IPsec Counters

## 🔎 IPsec Status

```bash
diagnose vpn ipsec status
```

Check:

* [ ] Encrypted packets
* [ ] Decrypted packets
* [ ] Packet counters
* [ ] IPsec processing statistics

### Counter Interpretation

```text
Encrypt counter increasing
        │
        └──► Outbound traffic is entering IPsec
```

```text
Decrypt counter increasing
        │
        └──► Inbound IPsec traffic is being decrypted
```

---

## Useful Comparison

```text
Outbound traffic
      │
      ▼
Encrypt counter
      │
      ▼
Remote peer
      │
      ▼
Decrypt counter
```

If only one side's counters increase:

* [ ] Check return routing
* [ ] Check remote policy
* [ ] Check remote selectors
* [ ] Check NAT
* [ ] Check packet loss
* [ ] Check peer-side counters

---

# 8. NPU / ASIC Offload

## 🧩 Hardware Information

```bash
get hardware status
```

Useful for identifying:

* [ ] FortiGate platform
* [ ] Hardware capabilities
* [ ] Platform-specific acceleration behavior

---

## NP6 Information

```bash
diagnose npu np6 port-list
```

Use when troubleshooting platforms that use NP6 hardware.

---

## Check Phase 1 NPU Configuration

```bash
config vpn ipsec phase1-interface
    edit "link-1"
        get | grep npu
    next
end
```

Look for:

```text
npu-offload
```

---

## Check Tunnel NPU Information

```bash
diagnose vpn tunnel list
```

Look for NPU-related information and flags.

### Interpretation

```text
NPU-related flag present
        │
        └──► Hardware offload may be available
```

```text
No expected NPU indication
        │
        └──► Investigate software processing
```

> [!IMPORTANT]
> NPU behavior depends on the **FortiGate model, FortiOS version, crypto engine, proposal, and enabled IPsec features**. Do not assume that every tunnel or proposal is hardware-offloaded.

---

# 9. IPsec ASIC Statistics

```bash
diagnose vpn ipsec stat
```

Compare software and hardware-related counters.

### Troubleshooting Model

```text
IPsec Traffic
      │
      ▼
diagnose vpn ipsec stat
      │
      ├── Software counters increasing
      │          │
      │          └──► CPU/software processing
      │
      └── Hardware counters increasing
                 │
                 └──► Hardware acceleration
```

---

## Global IPsec ASIC Offload

Configuration may include:

```bash
config system global
    set ipsec-asic-offload disable
    set ipsec-hmac-offload disable
end
```

Normal hardware-acceleration operation generally expects:

```text
ipsec-asic-offload = enable
ipsec-hmac-offload = enable
```

### Checklist

* [ ] IPsec ASIC offload enabled when required
* [ ] HMAC offload enabled when supported
* [ ] NPU compatibility verified
* [ ] Tunnel flags checked
* [ ] Hardware/software counters checked
* [ ] Platform-specific behavior verified

> [!WARNING]
> Do not disable hardware offload as a generic troubleshooting step without understanding the performance impact.

---

# 10. IPsec Proposal vs NPU

Not every proposal or feature is necessarily hardware-offloaded.

Use:

```bash
diagnose vpn tunnel list
```

and:

```bash
diagnose vpn ipsec stat
```

### Investigation Flow

```text
Tunnel UP
   │
   ▼
Check Negotiated Proposal
   │
   ▼
Check Platform Support
   │
   ▼
Check NPU Flags
   │
   ▼
Check Hardware Counters
   │
   ▼
Determine:
Hardware vs Software Processing
```

Consider:

* [ ] Encryption algorithm
* [ ] Integrity/HMAC
* [ ] GCM/AEAD behavior
* [ ] DiffServ
* [ ] Fragmentation
* [ ] Platform support
* [ ] FortiOS version
* [ ] NPU configuration

---

# 11. IPsec Fragmentation

Fragmentation can become important when VPN encapsulation increases packet size.

## Phase 1 Configuration

Example:

```bash
config vpn ipsec phase1-interface
    edit "link-1"
        set ip-fragment pos-encapsulation
    next
end
```

Conceptually:

### Post-Encapsulation

```text
Original Packet
      │
      ▼
IPsec Encapsulation
      │
      ▼
Fragmentation
```

### Pre-Encapsulation

```text
Original Packet
      │
      ▼
Fragmentation
      │
      ▼
IPsec Encapsulation
```

Possible modes include:

```text
pre-encapsulation
post-encapsulation
```

### Checklist

* [ ] Path MTU verified
* [ ] IPsec overhead considered
* [ ] Fragmentation behavior understood
* [ ] Peer supports the selected behavior
* [ ] MTU/MSS tested
* [ ] Large packets tested

---

# 12. DSCP / DiffServ

When DiffServ handling is required:

```bash
config vpn ipsec phase1-interface
    edit "link-1"
        set npu-offload disable
    next
end
```

Phase 2 example:

```bash
config vpn ipsec phase2-interface
    edit "link-1"
        set diffserv enable
        set diffserv 00011
    next
end
```

### Design Concept

```text
DiffServ processing
       │
       ▼
Software processing may be required
       │
       ▼
NPU offload disabled when necessary
```

### Checklist

* [ ] DiffServ requirement identified
* [ ] NPU compatibility checked
* [ ] NPU disabled only when required
* [ ] Phase 2 DiffServ configured
* [ ] Tunnel behavior verified
* [ ] Performance impact considered

---

# 13. Dialup IPsec / Mode Config

Mode Config is commonly used to assign network parameters to remote-access IPsec clients.

Typical flow:

```text
FortiClient
     │
     ▼
IPsec Dialup
     │
     ▼
FortiGate
     │
     ▼
Mode Config
     │
     ├──► Client IP
     ├──► DNS
     ├──► Split Tunnel
     └──► Other Network Parameters
```

---

## DHCP Proxy

Example configuration:

```bash
config system setting
    set dhcp-proxy enable
    set dhcp-server-ip 192.168.20.200
end
```

Traffic flow:

```text
VPN Client
    │
    ▼
Dialup FortiGate
    │
    │ DHCP Proxy
    ▼
DHCP Server
192.168.20.200
```

### Mode Config Checklist

* [ ] Mode Config enabled
* [ ] Client IP range configured
* [ ] Correct subnet mask configured
* [ ] DHCP proxy configured when required
* [ ] DHCP server reachable
* [ ] Phase 1 established
* [ ] Phase 2 established
* [ ] Firewall policy allows traffic
* [ ] DNS settings are correct
* [ ] Split tunnel settings are correct

---

# 14. DNS / FQDN Troubleshooting

## 🔎 DNS Proxy Debug

```bash
diagnose test application dnsproxy 7
```

Useful for investigating:

* [ ] DNS resolution
* [ ] FQDN address objects
* [ ] DDNS behavior
* [ ] IPsec peers using FQDN
* [ ] DNS proxy behavior

---

## Check Specific IPsec Tunnel

```bash
diagnose vpn tunnel list name ddns6
```

Useful when:

* [ ] Remote gateway uses FQDN
* [ ] DDNS is used
* [ ] Multiple IPsec tunnels exist
* [ ] Specific tunnel troubleshooting is required

Check:

* [ ] Tunnel state
* [ ] Remote IP
* [ ] Local IP
* [ ] SA state
* [ ] Negotiation state

---

# 15. IKEv2 EAP

EAP is commonly used in IKEv2 remote-access authentication designs.

## Basic Configuration

```bash
config vpn ipsec phase1-interface
    edit "link-1"
        set eap enable
        set eap-identity send-request
    next
end
```

---

## EAP Remote Access Example

```bash
config vpn ipsec phase1-interface
    edit "VPN1"
        set type dynamic
        set interface "port1"
        set ike-version 2
        set authmethod signature
        set peertype any
        set net-device disable
        set mode-cfg enable

        set ipv4-dns-server1 192.168.1.100

        set proposal aes128-sha256 aes256-sha256
        set localid "vpn.lab.local"

        set dpd on-idle
        set dhgrp 14 5 2

        set eap enable
        set eap-identity send-request

        set certificate "vpn.lab.local"

        set ipv4-start-ip 10.58.58.1
        set ipv4-end-ip 10.58.58.10

        set ipv4-split-include "10/8_net"

        set dpd-retryinterval 60
    next
end
```

### EAP Checklist

* [ ] IKEv2 enabled
* [ ] EAP enabled
* [ ] EAP identity behavior is correct
* [ ] Authentication method is compatible
* [ ] Certificate configured
* [ ] Certificate identity/SAN is correct
* [ ] DH group matches
* [ ] Proposal matches
* [ ] Mode Config enabled when required
* [ ] Client IP pool configured
* [ ] Split tunnel configured when required
* [ ] DNS configured when required
* [ ] DPD configured appropriately

---

# 16. MTU Troubleshooting

If:

```text
Ping works
```

but:

```text
Large packets fail
```

investigate MTU and fragmentation.

### Checklist

* [ ] Physical interface MTU checked
* [ ] WAN MTU checked
* [ ] IPsec overhead considered
* [ ] GRE overhead considered when applicable
* [ ] Fragmentation behavior checked
* [ ] PMTUD behavior checked
* [ ] MSS clamping considered
* [ ] Large packet test performed

### Typical Symptom

```text
Small packet
    │
    └──► SUCCESS

Large packet
    │
    └──► FAILURE
```

This is a strong indication to investigate:

```text
MTU
Fragmentation
PMTUD
MSS
```

---

# 17. NPU Troubleshooting Flow

```text
                IPsec Traffic
                     │
                     ▼
          diagnose vpn tunnel list
                     │
                     ▼
             Check NPU flags
                     │
             ┌───────┴───────┐
             │               │
            YES              NO
             │               │
             ▼               ▼
      Offload possible    Investigate
             │             software path
             │               │
             └───────┬───────┘
                     ▼
          diagnose vpn ipsec stat
                     │
             ┌───────┴────────┐
             │                │
       Hardware counters   Software counters
             │                │
             ▼                ▼
       ASIC/NPU path       CPU path
```

### Verify

* [ ] Platform supports required acceleration
* [ ] Proposal supports offload
* [ ] NPU offload enabled
* [ ] HMAC offload enabled where supported
* [ ] No feature disables acceleration
* [ ] Hardware counters verified
* [ ] Software counters verified

---

# 18. Complete Troubleshooting Flow

```text
                         START
                           │
                           ▼
                diagnose vpn ike status
                           │
                           ▼
             diagnose vpn ike gateway list
                           │
                           ▼
                   Phase 1 UP?
                    /          \
                  NO            YES
                  │              │
                  ▼              ▼
              IKE Debug    diagnose vpn tunnel list
                                 │
                                 ▼
                           Phase 2 UP?
                            /         \
                          NO           YES
                          │             │
                          ▼             ▼
                      Check P2       Check Traffic
                      Proposal           │
                      Selectors          ▼
                      PFS             Check Route
                                        │
                                        ▼
                                  Check Firewall
                                        │
                                        ▼
                                     Check NAT
                                        │
                                        ▼
                                   Check Selectors
                                        │
                                        ▼
                                  Check Counters
                                        │
                                        ▼
                                  Packet Capture
                                        │
                                        ▼
                                    Check MTU
                                        │
                                        ▼
                                   Check NPU
                                        │
                                        ▼
                                       DONE
```

---

# 19. Phase 1 Checklist

## 🌐 Connectivity

* [ ] Remote gateway is correct
* [ ] WAN interface is correct
* [ ] UDP/500 is allowed
* [ ] UDP/4500 is allowed when NAT-T is required
* [ ] Peer is reachable

## 🔐 Authentication

* [ ] Authentication method matches
* [ ] PSK matches
* [ ] Peer ID matches
* [ ] Local ID matches
* [ ] Remote ID matches
* [ ] Certificate is valid
* [ ] Certificate SAN/CN matches expected identity

## 🔒 Proposal

* [ ] IKE version matches
* [ ] Encryption matches
* [ ] Integrity/hash matches
* [ ] DH group matches
* [ ] Authentication method matches

## ⚙️ Operational

* [ ] DPD configuration checked
* [ ] NAT-T checked
* [ ] Rekey behavior checked
* [ ] Correct Phase 1 interface selected

---

# 20. Phase 2 Checklist

* [ ] Phase 1 is UP
* [ ] Phase 2 proposal matches
* [ ] Encryption matches
* [ ] Authentication/integrity matches
* [ ] PFS matches
* [ ] DH group matches
* [ ] Local selector matches
* [ ] Remote selector matches
* [ ] Protocol selector matches
* [ ] Auto-negotiate configured when required
* [ ] GRE protocol 47 configured when required
* [ ] NAT-translated selectors are understood when NAT is used

---

# 21. Traffic Checklist

## Routing

* [ ] Remote network route exists
* [ ] Correct next hop/interface selected
* [ ] Return route exists
* [ ] No conflicting route exists
* [ ] SD-WAN behavior checked
* [ ] PBR behavior checked

## Firewall

* [ ] Correct incoming interface
* [ ] Correct outgoing interface
* [ ] Correct source
* [ ] Correct destination
* [ ] Correct service
* [ ] Correct policy order
* [ ] Logging enabled

## NAT

* [ ] NAT intentionally enabled/disabled
* [ ] IP Pool checked
* [ ] VIP/DNAT checked
* [ ] NAT does not conflict with selectors

## VPN

* [ ] Phase 1 UP
* [ ] Phase 2 UP
* [ ] Encrypt counters increasing
* [ ] Decrypt counters increasing
* [ ] Peer receives expected traffic

---

# 22. Packet Capture

When configuration looks correct but traffic still fails, move to packet-level verification.

Investigate:

```text
Client
  │
  ▼
FortiGate ingress
  │
  ▼
Routing
  │
  ▼
Firewall Policy
  │
  ▼
IPsec Encryption
  │
  ▼
WAN
  │
  ▼
Remote FortiGate
  │
  ▼
Decryption
  │
  ▼
Remote LAN
```

### Packet Capture Questions

* [ ] Does the packet reach FortiGate?
* [ ] Is the expected route selected?
* [ ] Does the packet match the firewall policy?
* [ ] Is it encrypted?
* [ ] Does the encrypted packet leave the WAN?
* [ ] Does the remote peer receive it?
* [ ] Is it decrypted?
* [ ] Does return traffic follow the expected path?

---

# 23. Useful Command Reference

| Command                                | Purpose                             |
| -------------------------------------- | ----------------------------------- |
| `get hardware status`                  | Hardware/platform information       |
| `diagnose npu np6 port-list`           | NP6/NPU information                 |
| `diagnose vpn ike status`              | IKE daemon status                   |
| `diagnose vpn ike gateway list`        | Phase 1 / IKE SA                    |
| `diagnose vpn tunnel list`             | Phase 2 / IPsec SA / tunnel details |
| `diagnose vpn tunnel list name <name>` | Specific tunnel                     |
| `diagnose vpn ipsec status`            | Encryption/decryption counters      |
| `diagnose vpn ipsec stat`              | IPsec hardware/software statistics  |
| `diagnose debug application ike -1`    | IKE debugging                       |
| `diagnose vpn ike log-filter ...`      | Filter IKE debug                    |
| `diagnose test application dnsproxy 7` | DNS proxy debugging                 |

---

# 24. ⚡ Fast IPsec Troubleshooting Commands

## General Status

```bash
diagnose vpn ike status
diagnose vpn ike gateway list
diagnose vpn tunnel list
diagnose vpn ipsec status
```

## IKE Debug

```bash
diagnose debug enable
diagnose debug application ike -1
```

## IKE Filter

```bash
diagnose vpn ike log-filter src-addr4 12.23.34.1
```

## Stop Debug

```bash
diagnose debug disable
diagnose debug reset
```

## NPU

```bash
get hardware status
diagnose npu np6 port-list
diagnose vpn tunnel list
diagnose vpn ipsec stat
```

---

# 25. 🧪 Fast Troubleshooting Sequence

Copy this sequence into your operational runbook:

```bash
# 1. IKE subsystem
diagnose vpn ike status

# 2. Phase 1 / IKE SA
diagnose vpn ike gateway list

# 3. Phase 2 / IPsec SA
diagnose vpn tunnel list

# 4. Traffic counters
diagnose vpn ipsec status

# 5. IKE debug
diagnose debug enable
diagnose debug application ike -1

# 6. Reproduce the problem

# 7. Stop debugging
diagnose debug disable

# 8. Reset debug configuration
diagnose debug reset
```

---

# 26. 🧠 Troubleshooting Decision Tree

```text
IPsec VPN Problem
       │
       ▼
Is Phase 1 UP?
       │
   ┌───┴───┐
   NO      YES
   │        │
   ▼        ▼
 IKE      Is Phase 2 UP?
 Debug       │
          ┌──┴──┐
          NO    YES
          │      │
          ▼      ▼
       P2       Does traffic pass?
       Check       │
               ┌───┴───┐
               NO      YES
               │        │
               ▼        ▼
            Route     DONE
            Policy
            NAT
            Selector
            Counter
            MTU
            NPU
```

---

# 27. 🧩 Control Plane vs Data Plane

A useful way to isolate IPsec failures:

```text
             IPsec Troubleshooting
                      │
             ┌────────┴────────┐
             │                 │
             ▼                 ▼
       Control Plane       Data Plane
             │                 │
             ▼                 ▼
            IKE             Routing
             │              Policy
             ▼              NAT
          Phase 1          Selectors
             │              MTU
             ▼              Counters
          Phase 2          NPU
             │
             ▼
      Authentication
      Proposal
      DH
      PFS
      Certificate
```

---

# 28. 🧠 Golden Rules

> [!IMPORTANT]
> **Phase 1 UP does not mean Phase 2 is UP.**

> [!IMPORTANT]
> **Phase 2 UP does not mean traffic is working.**

> [!IMPORTANT]
> **Working traffic does not automatically mean hardware offload is being used.**

Therefore:

```text
Phase 1 UP
    ≠
Phase 2 UP

Phase 2 UP
    ≠
Traffic Working

Traffic Working
    ≠
Hardware Offload
```

Each layer must be verified independently.

---

# 29. 🚨 Common IPsec Troubleshooting Mistakes

* [ ] Assuming Phase 1 UP means VPN is fully operational
* [ ] Ignoring Phase 2 selectors
* [ ] Forgetting return routing
* [ ] Assuming firewall policy alone controls VPN traffic
* [ ] Ignoring NAT
* [ ] Ignoring NAT-T
* [ ] Ignoring MTU
* [ ] Assuming every IPsec proposal is NPU-offloaded
* [ ] Disabling NPU without checking the reason
* [ ] Running unrestricted IKE debug for too long
* [ ] Troubleshooting performance before establishing basic connectivity
* [ ] Ignoring the peer-side configuration
* [ ] Ignoring certificate SAN/identity
* [ ] Assuming small-packet success means large-packet success

---

# 30. 🏁 Final Validation Checklist

## Phase 1

* [ ] Gateway reachable
* [ ] Correct WAN interface
* [ ] UDP/500 reachable
* [ ] UDP/4500 reachable when required
* [ ] IKE version matches
* [ ] Authentication matches
* [ ] PSK/certificate is correct
* [ ] Peer ID matches
* [ ] Proposal matches
* [ ] DH group matches
* [ ] DPD checked

## Phase 2

* [ ] Phase 2 is established
* [ ] Encryption matches
* [ ] Integrity matches
* [ ] PFS matches
* [ ] DH group matches
* [ ] Selectors match
* [ ] Protocol selector matches
* [ ] Auto-negotiate checked

## Routing

* [ ] Remote route exists
* [ ] Return route exists
* [ ] Correct interface selected
* [ ] No conflicting route
* [ ] SD-WAN/PBR checked

## Firewall

* [ ] Correct ingress interface
* [ ] Correct egress interface
* [ ] Correct source
* [ ] Correct destination
* [ ] Correct service
* [ ] Correct policy order
* [ ] Logging enabled

## NAT

* [ ] NAT behavior verified
* [ ] IP Pool checked
* [ ] VIP/DNAT checked
* [ ] NAT does not conflict with selectors

## Performance

* [ ] MTU checked
* [ ] Fragmentation checked
* [ ] DSCP/DiffServ checked
* [ ] NPU compatibility checked
* [ ] NPU flags checked
* [ ] Hardware/software counters checked

## Verification

* [ ] `diagnose vpn ike status`
* [ ] `diagnose vpn ike gateway list`
* [ ] `diagnose vpn tunnel list`
* [ ] `diagnose vpn ipsec status`
* [ ] IKE debug performed if required
* [ ] Packet capture performed if required
* [ ] Debug disabled after testing

---

# 31. 📌 Minimal Production Troubleshooting Flow

```text
1️⃣ IKE Status
      ↓
2️⃣ Gateway List
      ↓
3️⃣ Phase 1
      ↓
4️⃣ Tunnel List
      ↓
5️⃣ Phase 2
      ↓
6️⃣ Route
      ↓
7️⃣ Firewall Policy
      ↓
8️⃣ NAT
      ↓
9️⃣ Selectors
      ↓
🔟 Counters
      ↓
1️⃣1️⃣ Packet Capture
      ↓
1️⃣2️⃣ MTU
      ↓
1️⃣3️⃣ NPU / ASIC
```

> **Rule:** Troubleshoot from **control plane → forwarding plane → performance plane**.

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

`FortiGate` `Fortinet` `IPsec` `IPsec VPN` `IKE` `IKEv1` `IKEv2` `Phase 1` `Phase 2` `NPU` `ASIC` `IPsec Troubleshooting` `VPN Troubleshooting` `NAT-T` `MTU` `DSCP` `DiffServ` `Mode Config` `EAP` `FQDN` `DNS` `FortiClient` `Network Security` `NSE` `FortiOS`


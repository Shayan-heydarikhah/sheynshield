# FortiGate IPsec Troubleshooting

> **Scope:** IPsec troubleshooting, NPU offload, ASIC acceleration, fragmentation, DSCP, Dialup/Mode-Cfg, DNS/FQDN, EAP and IKE diagnostics.

---

# 1. IPsec + NPU Troubleshooting

## Check Hardware / NPU

```bash
get hardware status
```

```bash
diagnose npu np6 port-list
```

### Check NPU Offload on IPsec Phase 1

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

# 2. Check IPsec NPU Flags

```bash
diagnose vpn tunnel list
```

### Important

The **last line** of the tunnel output contains NPU-related flags.

```text
NPU flags
```

General interpretation:

| Indicator           | Meaning                                 |
| ------------------- | --------------------------------------- |
| NPU flag present    | IPsec processing can use NPU/ASIC       |
| `np` flag = `0`     | Main CPU/software processing            |
| Software processing | Proposal/feature is not being offloaded |

> ⚠️ Some encryption/authentication proposals or advanced IPsec features may prevent NPU offloading.

---

# 3. IPsec ASIC Statistics

```bash
diagnose vpn ipsec stat
```

Look at:

```text
software
```

### Interpretation

```text
software counter increasing
        |
        +--> Main CPU is processing traffic
```

```text
ASIC/NPU counters increasing
        |
        +--> Hardware acceleration is being used
```

---

# 4. IPsec ASIC Offload

### Global IPsec ASIC Offload

```bash
config system global
    set ipsec-asic-offload disable
    set ipsec-hmac-offload disable
end
```

### Important

For normal hardware acceleration:

```text
ipsec-asic-offload = enable
ipsec-hmac-offload = enable
```

If disabled:

```text
IPsec
   |
   v
Main CPU
   |
   v
Software processing
```

If enabled and supported:

```text
IPsec
   |
   v
NPU / ASIC / Crypto Accelerator
   |
   v
Hardware processing
```

> ⚠️ These settings should normally remain **enabled** when hardware offloading is desired.

---

# 5. Automatic ASIC Offloading

Newer FortiGate platforms provide additional counters to help monitor and improve IPsec performance.

Check statistics:

```bash
diagnose vpn ipsec stat
```

### Important

If automatic ASIC/offload functionality is disabled, relevant hardware/software counters may stop being updated.

---

# 6. IPsec Proposal vs NPU

Not every IPsec proposal is necessarily processed by hardware.

General troubleshooting flow:

```text
IPsec Tunnel UP
      |
      v
Check proposal
      |
      v
Check NPU compatibility
      |
      v
diagnose vpn tunnel list
      |
      v
Check NPU flags
      |
      v
diagnose vpn ipsec stat
      |
      +----> Software counters increasing?
      |          |
      |          +--> Main CPU processing
      |
      +----> Hardware counters increasing?
                 |
                 +--> ASIC/NPU processing
```

---

# 7. Suite-B and Hardware Offload

### Suite-B

Suite-B uses AES-based encryption with integrity protection, including GCM-mode implementations.

FortiOS support and hardware acceleration depend on the **FortiGate platform, FortiOS/kernel version, and available crypto/NPU engine**.

### Troubleshooting concept

```text
Suite-B proposal
      |
      v
Check platform support
      |
      +----> Hardware supported
      |          |
      |          +--> Crypto/NPU offload
      |
      +----> Hardware unsupported
                 |
                 +--> Software encryption/decryption
```

> Some platforms support Suite-B hardware offloading through dedicated crypto acceleration, while others process it in software.

---

# 8. Encryption Offload

Common encryption algorithms can have different hardware-offload behavior depending on the FortiGate platform.

Examples to investigate:

```text
NULL encryption
DES
3DES
AES
```

Check actual behavior instead of assuming offload:

```bash
diagnose vpn tunnel list
diagnose vpn ipsec stat
```

> **Best practice:** Verify the actual NPU/ASIC flags and counters on the specific FortiGate model.

---

# 9. Check IKE Proposal / HMAC

```bash
diagnose vpn ike gateway list
```

Check:

```text
Proposal
HMAC
Encryption
DH group
IKE version
```

Useful when determining whether the negotiated proposal matches the expected configuration.

---

# 10. IPsec Fragmentation

### Phase 1 Fragmentation Setting

```bash
config vpn ipsec phase1-interface
    edit "link-1"
        set ip-fragment pos-encapsulation
    next
end
```

### Concept

```text
Original Packet
      |
      v
IPsec Encapsulation
      |
      v
Fragment
```

With pre-encapsulation fragmentation:

```text
Original Packet
      |
      v
Fragment
      |
      v
IPsec Encapsulation
```

Possible modes:

```text
pre-encapsulation
post-encapsulation
```

> Use the appropriate mode according to MTU/path requirements and peer compatibility.

---

# 11. DSCP / DiffServ over IPsec

If DiffServ handling is required:

```bash
config vpn ipsec phase1-interface
    edit "link-1"
        set npu-offload disable
    next
end
```

Then configure Phase 2:

```bash
config vpn ipsec phase2-interface
    edit "link-1"
        set diffserv enable
        set diffserv 00011
    next
end
```

### Why Disable NPU?

Some DiffServ processing scenarios require software processing.

```text
DiffServ processing
       |
       v
NPU offload disabled
       |
       v
Main CPU processing
```

### Verify

```bash
diagnose vpn tunnel list
```

Check for:

```text
DSCP / DiffServ
```

or the relevant tunnel flag.

---

# 12. Dialup IPsec + Mode Config

When using **Mode Config** in Dialup IPsec scenarios, Mode Config must be configured consistently on the required devices.

Typical flow:

```text
FortiClient
     |
     | IPsec Dialup
     v
Dialup FortiGate
     |
     | DHCP Relay / Proxy
     v
DHCP Server
```

---

# 13. DHCP Proxy for Dialup VPN

```bash
config system setting
    set dhcp-proxy enable
    set dhcp-server-ip 192.168.20.200
end
```

### Traffic Flow

```text
Dialup VPN Client
        |
        v
Dialup VPN Server
        |
        | DHCP Proxy / Relay
        v
DHCP Server
192.168.20.200
```

Example topology:

```text
VPN Client
    |
    v
FGT-2
Dialup Server
DHCP Proxy
    |
    v
FGT-1
DHCP Server
```

> The dialup FortiGate can proxy/relay DHCP requests toward the actual DHCP server.

---

# 14. Mode Config Troubleshooting

Check:

```text
[ ] Mode Config enabled
[ ] Client IP range configured
[ ] Correct subnet mask
[ ] DHCP proxy configured if required
[ ] DHCP server reachable
[ ] IPsec Phase 1 established
[ ] Phase 2 established
[ ] Firewall policies allow traffic
```

---

# 15. FQDN / DNS Troubleshooting

### DNS Proxy Debug

```bash
diagnose test application dnsproxy 7
```

Useful for troubleshooting:

```text
DNS resolution
FQDN objects
DDNS
IPsec peers using FQDN
DNS proxy behavior
```

---

# 16. Check IPsec Tunnel by Name

```bash
diagnose vpn tunnel list name ddns6
```

Useful when:

```text
Remote Gateway = FQDN
DDNS is used
Multiple IPsec tunnels exist
```

Check:

```text
Tunnel state
Remote IP
Local IP
SA state
Negotiation
```

---

# 17. EAP over IKEv2

### Basic EAP Configuration

```bash
config vpn ipsec phase1-interface
    edit "link-1"
        set eap enable
        set eap-identity send-request
    next
end
```

---

# 18. Example EAP Remote Access VPN

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

```text
[ ] IKEv2
[ ] EAP enabled
[ ] Correct EAP identity behavior
[ ] Authentication method compatible
[ ] Certificate configured
[ ] Certificate identity/SAN correct
[ ] DH group matches
[ ] Proposal matches
[ ] Mode Config enabled if required
[ ] Client IP pool configured
[ ] Split tunnel configured if required
[ ] DNS configured if required
```

---

# 19. IKE Daemon Status

```bash
diagnose vpn ike status
```

### Shows

```text
IKE daemon summary
IKE processing state
General IKE information
```

Useful as the first high-level IKE check.

---

# 20. IKE Gateway Status

```bash
diagnose vpn ike gateway list
```

### Shows

```text
Phase 1 interface status
IKE SA
Local address
Remote address
IKE version
Proposal
Negotiation state
```

Use when:

```text
Phase 1 is not coming UP
```

---

# 21. IPsec Tunnel Status

```bash
diagnose vpn tunnel list
```

### Shows

```text
Phase 2 status
Tunnel ID
IKE version
Local/remote networks
DPD state
MTU
NPU information
SA information
```

Useful when:

```text
Phase 1 = UP
Phase 2 = DOWN
```

---

# 22. IPsec Encryption / Decryption Counters

```bash
diagnose vpn ipsec status
```

Check:

```text
Encrypted packets
Decrypted packets
Packet counters
IPsec processing statistics
```

### Quick Interpretation

```text
Encrypt counter increasing
        |
        +--> Outbound traffic is entering IPsec

Decrypt counter increasing
        |
        +--> Inbound IPsec traffic is being decrypted
```

---

# 23. IKE Debug

Enable debug:

```bash
diagnose debug enable
```

Enable IKE debug:

```bash
diagnose debug application ike -1
```

Useful for:

```text
Phase 1 negotiation
Authentication
Proposal mismatch
DH mismatch
PSK mismatch
Peer ID mismatch
IKE state machine
DPD
Rekey
```

---

# 24. IKE Log Filter

Example:

```bash
diagnose vpn ike log-filter src-addr4 12.23.34.1
```

Then enable debug:

```bash
diagnose debug enable
diagnose debug application ike -1
```

This helps reduce unrelated IKE debug output when troubleshooting a specific source.

---

# 25. Stop Debug

Always disable debugging after troubleshooting:

```bash
diagnose debug disable
```

Optionally reset debug settings:

```bash
diagnose debug reset
```

---

# 26. Recommended IPsec Troubleshooting Order

```text
                  START
                    |
                    v
          diagnose vpn ike status
                    |
                    v
       diagnose vpn ike gateway list
                    |
                    v
           Phase 1 established?
              /           \
            NO             YES
            |               |
            v               v
       IKE Debug       diagnose vpn
                       tunnel list
                            |
                            v
                     Phase 2 established?
                       /          \
                     NO            YES
                     |              |
                     v              v
                 Check P2        Check traffic
                 proposal        counters
                 selectors           |
                 PFS                 v
                                    |
                            diagnose vpn
                            ipsec status
                                    |
                                    v
                             Traffic works?
                              /       \
                            NO         YES
                            |           |
                            v           v
                         Policy       DONE
                         Routing
                         NAT
                         NPU
```

---

# 27. Phase 1 Failure Checklist

```text
[ ] Remote gateway reachable
[ ] UDP/500 reachable
[ ] UDP/4500 reachable when NAT-T is used
[ ] IKE version matches
[ ] Authentication method matches
[ ] PSK matches
[ ] Peer ID matches
[ ] Encryption proposal matches
[ ] Hash/integrity matches
[ ] DH group matches
[ ] Certificate valid when used
[ ] Local ID correct
[ ] Remote ID correct
[ ] DPD configuration compatible
```

---

# 28. Phase 2 Failure Checklist

```text
[ ] Phase 1 is UP
[ ] Phase 2 proposal matches
[ ] Encryption matches
[ ] Authentication/integrity matches
[ ] PFS matches
[ ] DH group matches
[ ] Local selector matches
[ ] Remote selector matches
[ ] Protocol selector matches
[ ] Auto-negotiate configured if required
```

---

# 29. Traffic Does Not Pass — Checklist

```text
Phase 1 UP?
    |
    +-- NO --> Troubleshoot IKE

Phase 2 UP?
    |
    +-- NO --> Troubleshoot Phase 2

Phase 2 UP
    |
    v
Check route
    |
    v
Check firewall policy
    |
    v
Check NAT
    |
    v
Check selectors
    |
    v
Check counters
    |
    v
Check NPU/offload
    |
    v
Packet capture
```

---

# 30. NPU Troubleshooting Flow

```text
IPsec Traffic
     |
     v
diagnose vpn tunnel list
     |
     +---- NPU flag?
     |       |
     |       +--> YES --> Hardware offload possible
     |       |
     |       +--> NO --> Investigate software processing
     |
     v
diagnose vpn ipsec stat
     |
     +---- software counter increasing?
     |            |
     |            +--> YES --> Main CPU processing
     |
     v
Check:
    - Proposal
    - Encryption
    - HMAC
    - DiffServ
    - Fragmentation
    - Platform support
    - NPU configuration
```

---

# 31. Useful Command Reference

| Command                                | Purpose                             |
| -------------------------------------- | ----------------------------------- |
| `get hardware status`                  | Hardware/platform information       |
| `diagnose npu np6 port-list`           | NP6 port/NPU information            |
| `diagnose vpn ike status`              | IKE daemon summary                  |
| `diagnose vpn ike gateway list`        | Phase 1 / IKE SA                    |
| `diagnose vpn tunnel list`             | Phase 2 / IPsec SA / tunnel details |
| `diagnose vpn tunnel list name <name>` | Specific tunnel                     |
| `diagnose vpn ipsec status`            | Encryption/decryption counters      |
| `diagnose vpn ipsec stat`              | IPsec software/hardware statistics  |
| `diagnose debug application ike -1`    | IKE debug                           |
| `diagnose vpn ike log-filter ...`      | Filter IKE debug                    |
| `diagnose test application dnsproxy 7` | DNS proxy debugging                 |

---

# 32. Fast Command Sequence

### General IPsec

```bash
diagnose vpn ike status
diagnose vpn ike gateway list
diagnose vpn tunnel list
diagnose vpn ipsec status
```

### IKE Debug

```bash
diagnose debug enable
diagnose debug application ike -1
```

### Filter

```bash
diagnose vpn ike log-filter src-addr4 12.23.34.1
```

### Stop

```bash
diagnose debug disable
```

---

# 33. Fast NPU Check

```bash
get hardware status
```

```bash
diagnose npu np6 port-list
```

```bash
diagnose vpn tunnel list
```

```bash
diagnose vpn ipsec stat
```

```bash
diagnose vpn ike gateway list
```

---

# 34. Troubleshooting Mindset

```text
             IPsec Problem
                  |
       +----------+----------+
       |                     |
     Control Plane       Data Plane
       |                     |
       v                     v
      IKE                 Routing
       |                  Policy
       v                  NAT
    Phase 1               Selectors
       |                  Counters
       v                  NPU
    Phase 2               MTU
       |
       v
 Authentication
 Proposal
 DH
 PFS
 Certificate
```

### Golden Rule

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

Each layer must be checked independently.

---

# 35. Minimal IPsec Troubleshooting Flow

```text
1. IKE Status
       ↓
2. Gateway List
       ↓
3. Tunnel List
       ↓
4. IPsec Counters
       ↓
5. Route
       ↓
6. Policy
       ↓
7. NAT
       ↓
8. Selector
       ↓
9. Packet Capture
       ↓
10. NPU / ASIC
```

> **Recommended approach:** Fix the lowest failing layer first. Do not troubleshoot NPU performance while Phase 1/Phase 2 or basic forwarding is still broken.

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

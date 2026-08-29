# FortiGate VXLAN over Dial-up IPsec — Deployment & Troubleshooting Checklist

> **SheynShield | Engineering Secure Networks**
>
> **Scope:** FortiGate VXLAN, VNI, Layer-2 extension, Dial-up IPsec, IPsec interface mode, VLAN bridging, Software Switch, DHCP relay, XAuth, IPsec verification, VXLAN verification, packet flow, and troubleshooting.
>
> **Use Case:** Secure Layer-2 extension between two FortiGate sites across an IP network using VXLAN carried over a Dial-up IPsec tunnel.

---

## Table of Contents

* [Architecture](#architecture)
* [1-pre-deployment-checklist](#1-pre-deployment-checklist)
* [2-vxlan-design-checklist](#2-vxlan-design-checklist)
* [3-fgt-1-vxlan-checklist](#3-fgt-1-vxlan-checklist)
* [4-fgt-2-vxlan-checklist](#4-fgt-2-vxlan-checklist)
* [5-software-switch-and-vlan-checklist](#5-software-switch-and-vlan-checklist)
* [6-dial-up-ipsec-checklist](#6-dial-up-ipsec-checklist)
* [7-ipsec-interface-checklist](#7-ipsec-interface-checklist)
* [8-vxlan-over-ipsec-checklist](#8-vxlan-over-ipsec-checklist)
* [9-dhcp-checklist](#9-dhcp-checklist)
* [10-firewall-policy-checklist](#10-firewall-policy-checklist)
* [11-connectivity-validation](#11-connectivity-validation)
* [12-ipsec-verification](#12-ipsec-verification)
* [13-vxlan-verification](#13-vxlan-verification)
* [14-dhcp-validation](#14-dhcp-validation)
* [15-packet-flow](#15-packet-flow)
* [16-troubleshooting-flow](#16-troubleshooting-flow)
* [17-ipsec-troubleshooting-checklist](#17-ipsec-troubleshooting-checklist)
* [18-vxlan-troubleshooting-checklist](#18-vxlan-troubleshooting-checklist)
* [19-layer-2-troubleshooting](#19-layer-2-troubleshooting)
* [20-performance-and-mtu](#20-performance-and-mtu)
* [21-security-hardening](#21-security-hardening)
* [22-production-readiness](#22-production-readiness)
* [23-quick-command-reference](#23-quick-command-reference)
* [24-final-validation-checklist](#24-final-validation-checklist)

---

# Architecture

## Target Architecture

```text
                         INTERNET
                            |
                 +----------+----------+
                 |                     |
               FGT-1                 FGT-2
                 |                     |
          Dial-up IPsec           IPsec Client
                 |                     |
           12.12.12.1/30        12.12.12.2/30
                 |                     |
                 +----------+----------+
                            |
                       VXLAN VNI 1000
                            |
                 +----------+----------+
                 |                     |
             VLAN 101              VLAN 101
                 |                     |
              LAN Side             LAN Side
```

## Logical Encapsulation

```text
Client Ethernet Frame
        |
        v
     VLAN 101
        |
        v
 Software Switch
        |
        v
 VXLAN VNI 1000
        |
        v
 IPsec Interface
        |
        v
    ESP / NAT-T
        |
        v
     Internet
        |
        v
 IPsec Decryption
        |
        v
 VXLAN VNI 1000
        |
        v
 Software Switch
        |
        v
     VLAN 101
        |
        v
 Remote Client
```

> [!IMPORTANT]
> **VXLAN provides the Layer-2 overlay. IPsec provides the encrypted transport.** They solve different problems and should be troubleshooting independently.

---

# 1. Pre-Deployment Checklist

## FortiGate

* [ ] FortiOS version is confirmed.
* [ ] VXLAN feature is supported on the target platform.
* [ ] IPsec interface mode is supported.
* [ ] Dial-up IPsec is supported.
* [ ] Required interfaces are available.
* [ ] WAN/underlay connectivity is working.
* [ ] System time is correct.
* [ ] DNS is working if FQDNs are used.
* [ ] Configuration backup has been created.
* [ ] Existing VLAN/VXLAN/VPN configuration has been reviewed.

## Network

* [ ] Underlay IP connectivity exists.
* [ ] Remote public IP/FQDN is reachable.
* [ ] UDP/500 is permitted when required.
* [ ] UDP/4500 is permitted when NAT-T is required.
* [ ] No unexpected upstream firewall is blocking IPsec.
* [ ] VXLAN transport path is understood.
* [ ] MTU requirements have been evaluated.

## Design

* [ ] VNI is documented.
* [ ] VLAN ID is documented.
* [ ] Local LAN subnet is documented.
* [ ] Remote LAN subnet is documented.
* [ ] IPsec tunnel addressing is documented.
* [ ] DHCP design is documented.
* [ ] Software Switch design is documented.
* [ ] Failure and rollback procedure is documented.

---

# 2. VXLAN Design Checklist

## VXLAN Parameters

| Parameter           | FGT-1              | FGT-2              |
| ------------------- | ------------------ | ------------------ |
| VXLAN Interface     | `vx101`            | `vx101`            |
| VNI                 | `1000`             | `1000`             |
| VLAN                | `101`              | `101`              |
| VXLAN Remote IP     | `12.12.12.2`       | `12.12.12.1`       |
| Transport Interface | `ipsec-dialup`     | `ipsec-dialup`     |
| LAN IP              | `192.168.101.1/24` | `192.168.101.2/24` |

## VXLAN Validation

* [ ] VNI is identical on both FortiGates.
* [ ] VNI `1000` is documented.
* [ ] VXLAN remote IP points to the IPsec tunnel endpoint.
* [ ] VXLAN transport interface is the intended IPsec interface.
* [ ] VLAN ID `101` is identical on both sites.
* [ ] VXLAN interface exists.
* [ ] Remote VXLAN endpoint is reachable through IPsec.

---

# 3. FGT-1 VXLAN Checklist

## Create VXLAN

```bash
config system vxlan
    edit vx101
        set remote-ip 12.12.12.2
        set interface ipsec-dialup
        set vni 1000
    next
end
```

### Validate

* [ ] VXLAN interface name = `vx101`.
* [ ] VNI = `1000`.
* [ ] Remote IP = `12.12.12.2`.
* [ ] Transport interface = `ipsec-dialup`.
* [ ] VXLAN interface is operational.

---

## VLAN on VXLAN

```text
Interface:
    vx101

VLAN ID:
    101

Name:
    vx-vlan-101
```

Checklist:

* [ ] VLAN interface exists.
* [ ] Parent interface = `vx101`.
* [ ] VLAN ID = `101`.
* [ ] Interface naming is documented.

---

## Local VLAN

```text
Interface:
    port3

VLAN ID:
    101

Name:
    vlan-101
```

Checklist:

* [ ] Physical LAN interface is correct.
* [ ] VLAN ID = `101`.
* [ ] Local switch configuration matches VLAN 101.
* [ ] VLAN tagging/untagging behavior is understood.

---

# 4. FGT-2 VXLAN Checklist

## Create VXLAN

```bash
config system vxlan
    edit vx101
        set remote-ip 12.12.12.1
        set interface ipsec-dialup
        set vni 1000
    next
end
```

Checklist:

* [ ] VXLAN interface name = `vx101`.
* [ ] VNI = `1000`.
* [ ] Remote IP = `12.12.12.1`.
* [ ] Transport interface = `ipsec-dialup`.
* [ ] VXLAN interface is operational.

---

## VLAN on VXLAN

```text
Interface:
    vx101

VLAN ID:
    101

Name:
    vx-vlan-101
```

Checklist:

* [ ] VLAN exists on VXLAN.
* [ ] VLAN ID = `101`.
* [ ] VXLAN parent interface is correct.

---

## Local VLAN

```text
Interface:
    port3

VLAN ID:
    101

Name:
    vlan-101
```

Checklist:

* [ ] VLAN exists on the LAN interface.
* [ ] VLAN ID = `101`.
* [ ] LAN switch is configured correctly.

---

# 5. Software Switch and VLAN Checklist

## FGT-1

Create:

```text
Name:
    ss-vx-vl-101

Type:
    Software Switch

Members:
    vlan-101
    vx-vlan-101

IP:
    192.168.101.1/24

Ping:
    Enable

DHCP:
    Enable
```

Checklist:

* [ ] Software Switch exists.
* [ ] `vlan-101` is a member.
* [ ] `vx-vlan-101` is a member.
* [ ] `192.168.101.1/24` is configured.
* [ ] Ping is enabled if required.
* [ ] DHCP server is configured if FGT-1 provides DHCP.

---

## FGT-2

```text
Name:
    ss-vx-vl-101

Type:
    Software Switch

Members:
    vlan-101
    vx-vlan-101

IP:
    192.168.101.2/24

Ping:
    Enable

DHCP:
    Relay

DHCP Server:
    192.168.101.1
```

Checklist:

* [ ] Software Switch exists.
* [ ] `vlan-101` is a member.
* [ ] `vx-vlan-101` is a member.
* [ ] `192.168.101.2/24` is configured as required.
* [ ] DHCP relay is configured if FGT-2 is relaying DHCP.
* [ ] DHCP server address = `192.168.101.1`.

---

# 6. Dial-up IPsec Checklist

> [!WARNING]
> The example below uses **IKEv1, Aggressive Mode, DES, MD5 and XAuth** because they represent a legacy/lab configuration. Do **not** treat these values as the preferred production security profile.

## FGT-1 — Dial-up Server

```text
Connection:
    Dial-up

Incoming Interface:
    ISP / WAN

Authentication:
    Pre-shared Key

IKE Version:
    IKEv1

Mode:
    Aggressive

Peer ID:
    Any

Encryption:
    DES

Hash:
    MD5

DH Group:
    5

XAuth:
    Auto Server
```

Checklist:

* [ ] Connection type = Dial-up.
* [ ] WAN interface is correct.
* [ ] PSK matches the client.
* [ ] IKE version matches.
* [ ] Authentication parameters match.
* [ ] Peer ID is correct.
* [ ] XAuth server configuration exists.
* [ ] User group is correct.
* [ ] Required user account exists.

---

## FGT-2 — Dial-up Client

```text
Remote Gateway:
    <FGT-1-PUBLIC-IP>

Incoming Interface:
    ISP / WAN

Device Creation:
    Enable

Pre-shared Key:
    <PSK>

IKE Version:
    IKEv1

Mode:
    Aggressive

Peer ID:
    Any

Encryption:
    DES

Hash:
    MD5

DH Group:
    5

XAuth:
    Client
```

Credentials:

```text
Username:
    <VPN-USERNAME>

Password:
    <VPN-PASSWORD>
```

Checklist:

* [ ] Remote gateway is correct.
* [ ] PSK matches.
* [ ] IKE version matches.
* [ ] Mode matches.
* [ ] Proposal matches.
* [ ] DH group matches.
* [ ] XAuth client is configured.
* [ ] Username is correct.
* [ ] Password is correct.

> [!CAUTION]
> Never commit real PSKs, passwords, private keys, or certificates to a public GitHub repository.

---

# 7. IPsec Interface Checklist

## FGT-1

```text
IPsec Interface:
    ipsec-dialup

IP:
    12.12.12.1/30
```

## FGT-2

```text
IPsec Interface:
    ipsec-dialup

IP:
    12.12.12.2/30
```

### Validation

* [ ] FGT-1 = `12.12.12.1/30`.
* [ ] FGT-2 = `12.12.12.2/30`.
* [ ] Both addresses are in the same `/30`.
* [ ] IPsec interface exists.
* [ ] Interface is up.
* [ ] Tunnel endpoint responds to ping where permitted.

Test:

```bash
ping 12.12.12.2
```

From FGT-2:

```bash
ping 12.12.12.1
```

---

# 8. VXLAN over IPsec Checklist

## FGT-1

```bash
config system vxlan
    edit vx101
        set remote-ip 12.12.12.2
        set interface ipsec-dialup
        set vni 1000
    next
end
```

## FGT-2

```bash
config system vxlan
    edit vx101
        set remote-ip 12.12.12.1
        set interface ipsec-dialup
        set vni 1000
    next
end
```

### Validation

* [ ] VXLAN does not use the public WAN interface directly.
* [ ] VXLAN uses the intended IPsec interface.
* [ ] Remote VXLAN IP = remote IPsec endpoint.
* [ ] VNI matches.
* [ ] IPsec is established before VXLAN testing.
* [ ] Underlay connectivity is working.

---

# 9. DHCP Checklist

## FGT-1 DHCP Server

```text
DHCP Server
    |
    v
192.168.101.0/24
```

Checklist:

* [ ] DHCP server enabled.
* [ ] DHCP pool is inside the correct subnet.
* [ ] Default gateway is correct.
* [ ] DNS configuration is correct.
* [ ] DHCP service is bound to the intended interface.

---

## FGT-2 DHCP Relay

```text
DHCP Relay
     |
     v
192.168.101.1
```

Checklist:

* [ ] DHCP relay is enabled.
* [ ] DHCP server = `192.168.101.1`.
* [ ] DHCP requests can traverse the VXLAN path.
* [ ] DHCP responses can return to the client.
* [ ] Firewall policy allows required traffic.

---

# 10. Firewall Policy Checklist

## FGT-1

Required interfaces may include:

```text
LAN / VLAN
Software Switch
VXLAN
IPsec
WAN
```

Checklist:

* [ ] LAN → VXLAN forwarding is permitted.
* [ ] VXLAN → LAN forwarding is permitted.
* [ ] IPsec → VXLAN forwarding is permitted where required.
* [ ] VXLAN → IPsec forwarding is permitted where required.
* [ ] Required traffic is allowed during testing.
* [ ] NAT is disabled where transparent L2/L3 behavior requires it.
* [ ] Logging is enabled during troubleshooting.

---

## FGT-2

Use the same validation model:

```text
LAN
  ↕
Software Switch
  ↕
VXLAN
  ↕
IPsec
```

Checklist:

* [ ] LAN → VXLAN permitted.
* [ ] VXLAN → LAN permitted.
* [ ] IPsec → VXLAN permitted.
* [ ] VXLAN → IPsec permitted.
* [ ] NAT behavior is correct.
* [ ] Logging is enabled during testing.

> [!IMPORTANT]
> Start troubleshooting with narrowly scoped policies in production. Using `ALL / ALL / ALL` can be useful in an isolated lab, but should not be the final production design.

---

# 11. Connectivity Validation

## IPsec Layer

```text
[ ] IKE establishes
[ ] IPsec SA establishes
[ ] 12.12.12.1 reachable from FGT-2
[ ] 12.12.12.2 reachable from FGT-1
```

## VXLAN Layer

```text
[ ] VXLAN interface exists
[ ] VNI = 1000
[ ] Remote VXLAN endpoint is correct
[ ] VXLAN transport uses IPsec
```

## VLAN Layer

```text
[ ] VLAN 101 exists locally
[ ] VLAN 101 exists on VXLAN
[ ] VLAN 101 exists on remote LAN
[ ] Software Switch contains both VLAN members
```

## Client Layer

```text
[ ] Client receives DHCP
[ ] Client receives correct IP
[ ] Client receives correct gateway
[ ] Client can ping gateway
[ ] Client-to-client connectivity works
[ ] Broadcast-dependent applications work if required
```

---

# 12. IPsec Verification

## IKE Gateway

```bash
diagnose vpn ike gateway list
```

Check:

* [ ] IKE SA exists.
* [ ] Remote address is correct.
* [ ] Local address is correct.
* [ ] IKE version is correct.
* [ ] Proposal is correct.
* [ ] Authentication is successful.
* [ ] Tunnel is established.

---

## IPsec Tunnel

```bash
diagnose vpn tunnel list
```

Check:

* [ ] IPsec SA exists.
* [ ] SPI values exist.
* [ ] Encryption is correct.
* [ ] Authentication/integrity is correct.
* [ ] Source is correct.
* [ ] Destination is correct.
* [ ] Tunnel state is UP.

---

# 13. VXLAN Verification

Verify:

```text
Interface:
    vx101

VNI:
    1000

Remote IP:
    12.12.12.2
```

On FGT-2:

```text
Remote IP:
    12.12.12.1
```

Checklist:

* [ ] VXLAN interface exists.
* [ ] VNI is correct.
* [ ] Remote IP is correct.
* [ ] IPsec interface is used as transport.
* [ ] VLAN interface exists over VXLAN.
* [ ] Software Switch contains VXLAN VLAN.

Basic tests:

```bash
ping 12.12.12.2
```

```bash
ping 192.168.101.2
```

---

# 14. DHCP Validation

## Expected Flow

```text
Client
   |
   v
VLAN 101
   |
   v
Software Switch
   |
   v
VXLAN
   |
   v
IPsec
   |
   v
FGT-1
   |
   v
DHCP Server
```

Checklist:

* [ ] DHCP Discover leaves the client.
* [ ] DHCP request reaches the remote site.
* [ ] DHCP server receives the request.
* [ ] DHCP Offer returns.
* [ ] Client receives an address.
* [ ] Client receives the expected subnet.
* [ ] Client receives the expected gateway.
* [ ] DNS settings are correct.

---

# 15. Packet Flow

## Complete Data Path

```text
Ethernet Frame
      |
      v
VLAN 101
      |
      v
Software Switch
      |
      v
VXLAN VNI 1000
      |
      v
IPsec Interface
      |
      v
ESP Encryption
      |
      v
Internet
      |
      v
ESP Decryption
      |
      v
IPsec Interface
      |
      v
VXLAN VNI 1000
      |
      v
Software Switch
      |
      v
VLAN 101
      |
      v
Remote Client
```

## Layer-by-Layer Model

| Layer        | Technology | Purpose               |
| ------------ | ---------- | --------------------- |
| LAN          | Ethernet   | Client connectivity   |
| L2           | VLAN 101   | Local segmentation    |
| L2 Overlay   | VXLAN      | Layer-2 extension     |
| L3 Transport | IPsec      | Secure transport      |
| Security     | ESP        | Encryption/integrity  |
| Underlay     | Internet   | Physical/IP transport |

---

# 16. Troubleshooting Flow

```text
                         START
                           |
                           v
                 Is IPsec established?
                    /             \
                  NO               YES
                  |                 |
                  v                 v
            Troubleshoot        Can IPsec
            Phase 1 / XAuth     interface ping?
            proposals               |
                              +------+------+
                              |             |
                             NO            YES
                              |             |
                              v             v
                         Check IPsec      Can VXLAN
                         interface        communicate?
                                             |
                                       +-----+-----+
                                       |           |
                                      NO          YES
                                       |           |
                                       v           v
                                   Check VNI     Check VLAN
                                   Remote IP     Software Switch
                                   Transport     DHCP
                                                 Policies
```

> **Golden Rule:** Never troubleshoot VXLAN before proving that the IPsec transport is healthy.

---

# 17. IPsec Troubleshooting Checklist

## Phase 1

```text
[ ] Remote gateway is correct
[ ] WAN interface is correct
[ ] UDP/500 is permitted
[ ] UDP/4500 is permitted when NAT-T is used
[ ] IKE version matches
[ ] Authentication method matches
[ ] PSK matches
[ ] Peer ID matches
[ ] Encryption matches
[ ] Hash/integrity matches
[ ] DH group matches
[ ] XAuth mode matches
[ ] Username is correct
[ ] Password is correct
```

## Phase 2

```text
[ ] Phase 1 is UP
[ ] Encryption proposal matches
[ ] Authentication/integrity matches
[ ] PFS configuration matches
[ ] DH group matches
[ ] Selectors are compatible
[ ] Auto-negotiate is configured where required
```

## Debug

```bash
diagnose debug enable
diagnose debug application ike -1
```

After testing:

```bash
diagnose debug disable
diagnose debug reset
```

> [!WARNING]
> Do not leave IKE debug enabled on a busy production firewall. Debug output can become extremely large and impact troubleshooting clarity.

---

# 18. VXLAN Troubleshooting Checklist

```text
[ ] VXLAN interface exists
[ ] VNI matches on both sides
[ ] VNI = 1000
[ ] Remote IP is correct
[ ] Remote IP belongs to the IPsec tunnel
[ ] VXLAN transport interface is correct
[ ] IPsec tunnel is UP
[ ] IPsec endpoint is reachable
[ ] VLAN 101 exists on VXLAN
[ ] VLAN 101 exists on LAN interface
[ ] Software Switch has both members
[ ] Firewall policies permit required traffic
```

### Common VXLAN Failure Pattern

```text
IPsec DOWN
    ↓
VXLAN DOWN / unusable
    ↓
L2 extension fails
```

Therefore:

```text
Fix IPsec first
    ↓
Validate IPsec interface
    ↓
Validate VXLAN
    ↓
Validate VLAN
    ↓
Validate client traffic
```

---

# 19. Layer-2 Troubleshooting

## VLAN

```text
[ ] VLAN ID matches
[ ] Switch trunk is correct
[ ] Access ports are correct
[ ] VLAN 101 is allowed across required links
[ ] No unexpected VLAN filtering exists
```

## Software Switch

```text
[ ] Local VLAN is a member
[ ] VXLAN VLAN is a member
[ ] Interface state is correct
[ ] IP configuration is correct
[ ] DHCP configuration is correct
```

## Broadcast Traffic

```text
[ ] DHCP broadcasts reach the remote side
[ ] ARP resolution works
[ ] Broadcast-dependent applications are tested
[ ] L2 extension requirements are understood
```

> [!IMPORTANT]
> Extending Layer 2 between sites also extends the associated broadcast and failure domain. Use L2 extension only when the application/design actually requires it.

---

# 20. Performance and MTU

VXLAN adds encapsulation overhead, and IPsec adds additional encapsulation.

Conceptually:

```text
Original Ethernet
      +
VXLAN overhead
      +
IPsec overhead
      =
Larger packet
```

Checklist:

* [ ] Underlay MTU has been verified.
* [ ] IPsec MTU has been considered.
* [ ] VXLAN overhead has been considered.
* [ ] Fragmentation behavior is understood.
* [ ] PMTUD behavior has been tested.
* [ ] Large packets have been tested.
* [ ] Application-specific MTU requirements have been checked.

## Symptom

```text
Small ping works
       |
       v
Large packet fails
       |
       v
Investigate MTU / fragmentation
```

---

# 21. Security Hardening

> [!WARNING]
> The original lab configuration uses legacy cryptography. For production deployments, use the strongest mutually supported cryptographic profile available for the specific FortiOS version and peer.

## Replace Legacy Settings

Avoid where possible:

```text
[ ] DES
[ ] 3DES
[ ] MD5
[ ] Weak DH groups
[ ] IKEv1
[ ] Aggressive Mode
```

Prefer where supported:

```text
[ ] IKEv2
[ ] AES-based encryption
[ ] SHA-2 integrity
[ ] Strong DH groups
[ ] PFS
[ ] Strong PSK
[ ] Certificate authentication where appropriate
[ ] NAT-T when required
```

## Credential Security

```text
[ ] No real PSK in GitHub
[ ] No real XAuth password in GitHub
[ ] No private keys in GitHub
[ ] No production certificate material in GitHub
[ ] Lab credentials replaced with placeholders
```

Use:

```text
<PSK>
<VPN-USERNAME>
<VPN-PASSWORD>
<REMOTE-PUBLIC-IP>
```

instead of real secrets.

---

# 22. Production Readiness

Before production:

```text
[ ] FortiOS compatibility verified
[ ] VXLAN support verified on hardware
[ ] IPsec feature support verified
[ ] Dial-up behavior verified
[ ] Cryptographic profile hardened
[ ] MTU tested
[ ] Failover tested
[ ] DHCP failure tested
[ ] IPsec rekey tested
[ ] WAN interruption tested
[ ] FortiGate reboot tested
[ ] Configuration backup created
[ ] Monitoring configured
[ ] Logging configured
[ ] Rollback procedure documented
```

---

# 23. Quick Command Reference

| Command                             | Purpose                    |
| ----------------------------------- | -------------------------- |
| `diagnose vpn ike gateway list`     | IKE / Phase 1 state        |
| `diagnose vpn tunnel list`          | IPsec / Phase 2 / SA state |
| `diagnose vpn ike status`           | IKE daemon status          |
| `diagnose debug enable`             | Enable debugging           |
| `diagnose debug application ike -1` | IKE debug                  |
| `diagnose debug disable`            | Disable debugging          |
| `diagnose debug reset`              | Reset debug settings       |

## Fast IPsec Sequence

```bash
diagnose vpn ike status
diagnose vpn ike gateway list
diagnose vpn tunnel list
```

## IKE Debug

```bash
diagnose debug enable
diagnose debug application ike -1
```

Perform test traffic, then:

```bash
diagnose debug disable
diagnose debug reset
```

---

# 24. Final Validation Checklist

## Architecture

* [ ] VXLAN is being used as the L2 overlay.
* [ ] IPsec is being used as the secure transport.
* [ ] VXLAN remote IP points to the IPsec endpoint.
* [ ] Public Internet is not being used as an unencrypted VXLAN transport.

## IPsec

* [ ] Phase 1 is UP.
* [ ] Phase 2 is UP.
* [ ] XAuth succeeds where configured.
* [ ] IPsec interface is UP.
* [ ] `12.12.12.1/30` is configured on FGT-1.
* [ ] `12.12.12.2/30` is configured on FGT-2.
* [ ] IPsec counters increase during traffic tests.

## VXLAN

* [ ] VXLAN interface = `vx101`.
* [ ] VNI = `1000`.
* [ ] Remote IP is correct.
* [ ] Transport interface = IPsec.
* [ ] VXLAN interface is operational.

## VLAN

* [ ] VLAN ID = `101`.
* [ ] VLAN exists on the local LAN interface.
* [ ] VLAN exists on the VXLAN interface.
* [ ] Software Switch contains both VLAN interfaces.

## DHCP

* [ ] FGT-1 DHCP server works.
* [ ] FGT-2 DHCP relay works if used.
* [ ] DHCP requests cross the VXLAN.
* [ ] DHCP responses return successfully.
* [ ] Client receives the expected address.

## Firewall

* [ ] LAN → VXLAN allowed.
* [ ] VXLAN → LAN allowed.
* [ ] VXLAN → IPsec allowed.
* [ ] IPsec → VXLAN allowed.
* [ ] NAT behavior is correct.
* [ ] Logging is enabled during validation.

## Performance

* [ ] MTU is tested.
* [ ] Large packets work.
* [ ] Fragmentation behavior is understood.
* [ ] CPU utilization is monitored.
* [ ] IPsec performance is acceptable.

## Security

* [ ] Legacy cryptography has been replaced where supported.
* [ ] Strong authentication is used.
* [ ] Production credentials are not stored in GitHub.
* [ ] Firewall policies are least-privilege.
* [ ] Management access is restricted.

---

# Troubleshooting Decision Tree

```text
                    VXLAN OVER IPSEC PROBLEM
                              |
                              v
                       IPsec Phase 1?
                        /          \
                      NO            YES
                      |              |
                      v              v
                 IKE / XAuth     IPsec Phase 2?
                 / Proposal        /       \
                                  NO       YES
                                  |         |
                                  v         v
                             P2 / Selector  IPsec
                                          Interface
                                             |
                                             v
                                      Interface Ping?
                                       /          \
                                     NO            YES
                                     |              |
                                     v              v
                                IPsec Interface   VXLAN?
                                                  /    \
                                                NO      YES
                                                |        |
                                                v        v
                                           VNI/Remote   VLAN?
                                           Transport    /  \
                                                       NO   YES
                                                       |     |
                                                       v     v
                                                   VLAN /   DHCP?
                                                   Switch   /  \
                                                           NO   YES
                                                           |     |
                                                           v     v
                                                       DHCP /  DONE
                                                       Policy
```

---

# Golden Troubleshooting Rules

```text
IPsec UP
  ≠
IPsec Interface Working

IPsec Interface Working
  ≠
VXLAN Working

VXLAN Working
  ≠
VLAN Working

VLAN Working
  ≠
DHCP Working

DHCP Working
  ≠
Application Working
```

Always isolate the failure by layer:

```text
1. Underlay
      ↓
2. IKE
      ↓
3. IPsec SA
      ↓
4. IPsec Interface
      ↓
5. VXLAN
      ↓
6. VLAN
      ↓
7. Software Switch
      ↓
8. DHCP
      ↓
9. Firewall Policy
      ↓
10. MTU / Application
```

---

# One-Minute Validation

```text
[ ] IPsec UP
[ ] IPsec Interface UP
[ ] 12.12.12.1 ↔ 12.12.12.2 reachable
[ ] VXLAN VNI 1000 configured
[ ] VXLAN remote IP correct
[ ] VLAN 101 configured
[ ] Software Switch contains local + VXLAN VLAN
[ ] DHCP works
[ ] Client-to-client traffic works
[ ] IPsec counters increase
[ ] No MTU problems
[ ] No unintended NAT
[ ] No production secrets exposed
```

---

# Quick Reference

```text
VXLAN Interface:
    vx101

VNI:
    1000

VLAN:
    101

FGT-1 IPsec:
    12.12.12.1/30

FGT-2 IPsec:
    12.12.12.2/30

FGT-1 LAN:
    192.168.101.0/24

FGT-2 LAN:
    192.168.101.0/24

FGT-1:
    DHCP Server

FGT-2:
    DHCP Relay

Transport:
    IPsec

Overlay:
    VXLAN

Security:
    IPsec / ESP
```

---

# Key Takeaways

> **VXLAN = Layer-2 Overlay**

> **IPsec = Secure Transport**

> **Software Switch = Local L2 Integration**

> **VLAN = LAN Segmentation**

> **Dial-up IPsec = Dynamic/NATed Peer Connectivity**

> **DHCP Relay = Remote DHCP Service Delivery**

The correct troubleshooting methodology is:

```text
Underlay
   ↓
IPsec
   ↓
IPsec Interface
   ↓
VXLAN
   ↓
VLAN
   ↓
Software Switch
   ↓
DHCP
   ↓
Client Traffic
```

> **SheynShield Engineering Principle:**
> **Do not troubleshoot the overlay until the secure transport underneath it is proven healthy.**

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

## Keywords

`FortiGate VXLAN` · `VXLAN over IPsec` · `FortiGate Dial-up IPsec` · `FortiGate VXLAN troubleshooting` · `FortiGate Layer 2 extension` · `FortiGate VNI` · `FortiGate Software Switch` · `FortiGate VLAN` · `FortiGate IPsec troubleshooting` · `FortiGate DHCP relay` · `VXLAN IPsec tunnel` · `Fortinet VXLAN configuration` · `Fortinet IPsec troubleshooting` · `FortiGate network design` · `Fortinet network security`

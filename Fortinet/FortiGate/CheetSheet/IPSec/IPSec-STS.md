# FortiGate IPsec VPN — Site-to-Site & Advanced

> **Scope:** Site-to-Site IPsec, Overlapping Subnets, GRE over IPsec, GRE over IPsec with Cisco, Dial-up IPsec, Policy-Based IPsec, IPsec Concentrator, and AWS interoperability.

---

## 1. IPsec Site-to-Site — Troubleshooting

### Enable IKE Debug

```bash
diagnose debug enable
diagnose debug application ike -1
```

Useful for identifying:

* IKE negotiation failures
* Tunnel / IKE index
* Exchange state
* VRF
* Pre-shared-key mismatch
* Authentication problems
* Phase 1 negotiation failures

Disable debug after troubleshooting:

```bash
diagnose debug disable
```

---

### Check IKE Gateway

```bash
diagnose vpn ike gateway list
```

Check:

* Source address
* Destination address
* Tunnel ID
* Creation time
* Establishment time
* IKE SA
* IPsec SA

---

### Check IPsec Tunnel

```bash
diagnose vpn tunnel list
```

Useful information:

* IKE version
* Tunnel ID
* Local networks
* Remote networks
* Advertised / reachable networks
* MTU
* DPD state
* NPU ID

---

# 2. Site-to-Site IPsec — Overlapping Subnets

### Topology

```text
        FGT-1
   LAN: 192.168.101.0/24
          |
       IPsec
          |
        FGT-2
   LAN: 192.168.101.0/24
```

Both sides use the same subnet.

To solve this, translate each LAN into a unique subnet before sending traffic through IPsec.

```text
FGT-1 real LAN
192.168.101.0/24

        NAT

FGT-1 translated network
10.10.10.0/24
```

```text
FGT-2 real LAN
192.168.101.0/24

        NAT

FGT-2 translated network
20.20.20.0/24
```

---

# 3. FGT-1 — Overlapping Subnet

## LAN

```text
192.168.101.0/24
```

---

## Address Objects

### Local

```text
192.168.101.0/24
```

### Remote

```text
20.20.20.0/24
```

---

## IP Pool

Create a **Fixed Port Range** IP Pool:

```text
Name: fpr-1

External:
10.10.10.1 - 10.10.10.254

Internal:
192.168.101.1 - 192.168.101.254
```

Enable:

```text
ARP Reply
```

---

## VIP / DNAT

```text
Name: vip-101-over

Interface:
IPsec

Type:
Static

External IP:
10.10.10.1 - 10.10.10.254

Mapped IP:
192.168.101.1
```

---

## IPsec Phase 2

```text
Local:
10.10.10.0/24

Remote:
20.20.20.0/24
```

> [!IMPORTANT]
> The Phase 2 selectors must match the translated networks used by the NAT/DNAT design.

---

## IPsec Interface

```text
Local IP:
12.12.12.1

Remote IP:
12.12.12.2/24
```

Test:

```bash
ping 12.12.12.2
```

---

## Static Routes

Primary route:

```text
20.20.20.0/24
    |
    +-- IPsec
```

Backup blackhole:

```text
20.20.20.0/24
    |
    +-- Blackhole
    +-- Priority: 254
```

---

## Policies

### LAN → IPsec

```text
Name:
nat-101-to-10

Incoming:
LAN

Outgoing:
IPsec

Source:
Local

Destination:
Remote

Service:
ALL

NAT:
Dynamic IP Pool
fpr-1

Logging:
All Sessions
```

---

### IPsec → LAN

```text
Name:
dnat-10-to-101

Incoming:
IPsec

Outgoing:
LAN

Source:
Remote

Destination:
vip-101-over

Service:
ALL

NAT:
Disable

Logging:
All Sessions
```

---

# 4. FGT-2 — Overlapping Subnet

## LAN

```text
192.168.101.0/24
```

---

## Address Objects

### Local

```text
192.168.101.0/24
```

### Remote

```text
10.10.10.0/24
```

---

## IP Pool

```text
Name:
fpr-1

External:
20.20.20.1 - 20.20.20.254

Internal:
192.168.101.1 - 192.168.101.254
```

Enable:

```text
ARP Reply
```

---

## VIP / DNAT

```text
Name:
vip-101-over

Interface:
IPsec

Type:
Static

External IP:
20.20.20.1 - 20.20.20.254

Mapped IP:
192.168.101.1
```

---

## IPsec Phase 2

```text
Local:
20.20.20.0/24

Remote:
10.10.10.0/24
```

---

## IPsec Interface

```text
Local IP:
12.12.12.2

Remote IP:
12.12.12.1/24
```

Test:

```bash
ping 12.12.12.1
```

---

## Static Routes

Primary:

```text
10.10.10.0/24
    |
    +-- IPsec
```

Backup:

```text
10.10.10.0/24
    |
    +-- Blackhole
    +-- Priority: 254
```

---

## Policies

### LAN → IPsec

```text
Name:
nat-101-to-20

Incoming:
LAN

Outgoing:
IPsec

Source:
Local

Destination:
Remote

Service:
ALL

NAT:
Dynamic IP Pool
fpr-1

Logging:
All Sessions
```

---

### IPsec → LAN

```text
Name:
dnat-20-to-101

Incoming:
IPsec

Outgoing:
LAN

Source:
Remote

Destination:
vip-101-over

Service:
ALL

NAT:
Disable

Logging:
All Sessions
```

---

## Connectivity Test

From FGT-1 LAN:

```bash
ping 20.20.20.2
```

Expected destination:

```text
192.168.101.2
```

on the FGT-2 LAN side.

---

# 5. GRE over IPsec

## FGT-1

### IPsec

Create:

```text
Custom IPsec
Site-to-Site
Remote / Peer Gateway
```

Phase 2:

```text
Protocol:
47
```

Protocol `47` = GRE.

---

## Allow Subnet Overlap

```bash
config system settings
    set allow-subnet-overlap enable
end
```

---

## IPsec Interface

Assign:

```text
Local:
12.12.12.1

Remote:
12.12.12.2/24
```

---

## GRE Interface

```bash
config system gre
    edit gre-1
        set interface ipsec-link-1
        set remote-gw 12.12.12.2
        set local-gw 12.12.12.1
    end
end
```

---

## Route

Route remote LAN through GRE:

```text
192.168.102.0/24
    |
    +-- GRE
```

Optional backup:

```text
192.168.102.0/24
    |
    +-- Blackhole
    +-- AD: 254
```

---

## Policies

### Incoming

```text
IPsec-link-1
LAN
GRE-1
```

### Outgoing

```text
IPsec-link-1
LAN
GRE-1
```

Policy settings:

```text
Source:
ALL

Destination:
ALL

Service:
ALL

NAT:
Disable

Logging:
All Sessions
```

---

# 6. GRE over IPsec — FGT-2

## IPsec

Create:

```text
Custom IPsec
Site-to-Site
Remote / Peer Gateway
```

Phase 2:

```text
Protocol:
47
```

---

## Allow Subnet Overlap

```bash
config system settings
    set allow-subnet-overlap enable
end
```

---

## IPsec Interface

```text
Local:
12.12.12.2

Remote:
12.12.12.1/24
```

---

## GRE

```bash
config system gre
    edit gre-1
        set interface ipsec-link-1
        set remote-gw 12.12.12.1
        set local-gw 12.12.12.2
    end
end
```

---

## Route

```text
192.168.101.0/24
    |
    +-- GRE
```

Backup:

```text
192.168.101.0/24
    |
    +-- Blackhole
    +-- AD: 254
```

---

## Policies

```text
Incoming:
IPsec-link-1
LAN
GRE-1

Outgoing:
IPsec-link-1
LAN
GRE-1

Source:
ALL

Destination:
ALL

Service:
ALL

NAT:
Disable

Logging:
All Sessions
```

---

# 7. GRE over IPsec — Remote Device Behind NAT

Use **Dial-up IPsec** when the remote FortiGate is behind NAT or does not have a stable public IP.

### Topology

```text
FGT-1
  |
  | IPsec Dial-up
  |
Internet
  |
FGT-3 / NAT
  |
FGT-4
```

---

# 8. FGT-1 — Dial-up IPsec + GRE

## IPsec

```text
Name:
ipsec-dial-14

Connection:
Dial-up

Incoming:
ISP

Peer ID:
Any

Authentication:
Pre-shared Key

Mode:
Aggressive

Phase 2:
All required subnets

Protocol:
47
```

---

## IPsec Interface

```text
Local:
14.14.14.1

Remote:
14.14.14.4/24
```

---

## GRE

```bash
config system gre
    edit gre-14
        set remote-gw 14.14.14.4
        set local-gw 14.14.14.1
        set interface ipsec-dial-14
    end
end
```

---

## Route

```text
192.168.104.0/24
    |
    +-- GRE
```

Backup:

```text
192.168.104.0/24
    |
    +-- Blackhole
    +-- AD: 254
```

---

## Policies

```text
Incoming:
ipsec-dial-14
LAN
gre-14

Outgoing:
ipsec-dial-14
LAN
gre-14

Source:
ALL

Destination:
ALL

Service:
ALL

NAT:
Disable

Logging:
All Sessions
```

---

# 9. FGT-4 — Dial-up IPsec + GRE

## IPsec

```text
Name:
ipsec-14

Remote:
1.1.1.1

Peer ID:
Any

Pre-shared Key:
123456

Mode:
Aggressive

Phase 2:
All required subnets

Protocol:
47
```

---

## IPsec Interface

```text
Local:
14.14.14.4

Remote:
14.14.14.1/24
```

---

## GRE

```bash
config system gre
    edit gre-14
        set remote-gw 14.14.14.1
        set local-gw 14.14.14.4
        set interface ipsec-14
    end
end
```

---

## Route

```text
192.168.101.0/24
    |
    +-- GRE
```

Backup:

```text
192.168.101.0/24
    |
    +-- Blackhole
    +-- AD: 254
```

---

## Policies

```text
Incoming:
ipsec-14
LAN
gre-14

Outgoing:
ipsec-14
LAN
gre-14

Source:
ALL

Destination:
ALL

Service:
ALL

NAT:
Disable

Logging:
All Sessions
```

---

# 10. GRE over IPsec — Cisco Interoperability

## Cisco GRE

```cisco
interface tunnel1
 ip address 10.10.10.2 255.255.255.0
 tunnel source 20.20.20.2
 tunnel destination 20.20.20.1
```

---

## Cisco Phase 1

```cisco
crypto isakmp policy 10
 encr des
 hash md5
 authentication pre-share
 group 2
```

PSK:

```cisco
crypto isakmp key 123456 address 16.16.16.1
```

---

## Cisco Phase 2

```cisco
crypto ipsec transform-set greipsec_transform esp-des esp-md5-hmac
 mode transport
```

---

## Cisco IPsec Profile

```cisco
crypto ipsec profile greipsec_profile
 set transform-set greipsec_transform
```

---

## Cisco VTI

```cisco
interface tunne10
 ip address 20.20.20.2 255.255.255.0
 tunnel source 16.16.16.2
 tunnel destination 16.16.16.1
 tunnel mode ipsec ipv4
 tunnel protection ipsec profile greipsec_profile
```

---

## Cisco Route

```cisco
ip route 192.168.101.0 255.255.255.0 tunnel1
```

---

# 11. FortiGate — Cisco GRE/IPsec

## Phase 1

```text
Name:
ipsec-cisco-16

Remote:
16.16.16.2

Incoming:
ISP

PSK:
123456

IKE:
Version 1

Mode:
Aggressive

Peer ID:
Any

Encryption:
DES

Authentication:
MD5

DH:
Group 2
```

---

## Phase 2

```text
Networks:
ALL

Encryption:
DES

Authentication:
MD5

PFS:
Disable

Auto-negotiate:
Enable

Protocol:
47
```

---

## IPsec Interface

```text
Local:
20.20.20.1

Remote:
20.20.20.2/24
```

Test:

```bash
ping 20.20.20.2
```

---

## GRE

```bash
config system gre
    edit gre-16
        set interface ipsec-cisco-16
        set remote-gw 20.20.20.2
        set local-gw 20.20.20.1
    end
end
```

GRE interface:

```text
Local:
10.10.10.1

Remote:
10.10.10.2/24
```

---

## Route

```text
192.168.106.0/24
    |
    +-- GRE
```

Backup:

```text
192.168.106.0/24
    |
    +-- Blackhole
    +-- AD: 254
```

---

## Policies

```text
Incoming:
ipsec-cisco-16
LAN
gre-16

Outgoing:
ipsec-cisco-16
LAN
gre-16

Source:
ALL

Destination:
ALL

Service:
ALL

NAT:
Disable

Logging:
All Sessions
```

---

## Transport Mode

Sometimes required for Cisco interoperability:

```bash
config vpn ipsec phase2-interface
    edit ipsec-cisco-16
        set encapsulation transport-mode
    end
end
```

> [!WARNING]
> Depending on the DH group and interoperability scenario, enabling transport mode can cause the tunnel to go down. Test the behavior with the specific Cisco configuration.

---

# 12. Policy-Based IPsec VPN

Useful for:

* Hub-and-spoke
* ADVPN-like designs
* Remote-site initiated traffic
* Scenarios where IPsec interfaces are not desired

---

## Enable Feature

```text
System
  ↓
Feature Visibility
  ↓
Policy-based IPsec VPN
```

---

# 13. Policy-Based IPsec — Hub / FGT-1

Create custom IPsec.

```text
IPsec Interface Mode:
Disable

Remote:
2.2.2.1

Interface:
ISP-1

PSK:
123456

IKE:
Version 1

Mode:
Aggressive

Peer ID:
Any

Encryption:
DES

Authentication:
MD5

DH:
Group 5
```

Phase 2:

```text
DH:
Group 5

Encryption:
DES

Authentication:
MD5

Auto-negotiate:
Enable

Protocols:
ALL

Subnets:
ALL
```

Create another tunnel similarly:

```text
Remote:
3.3.3.1
```

---

## Policy-Based Policy

```text
Incoming:
LAN

Outgoing:
ISP

Action:
IPsec

VPN Tunnel:
IPsec Tunnel X

Allow traffic to be initiated
from the remote site:
Enable

Source:
ALL

Destination:
ALL

Service:
ALL

NAT:
Disable

Logging:
All Sessions
```

Create another policy for the second ISP/IPsec tunnel.

---

# 14. Policy-Based IPsec — Spoke

Example:

```text
FGT-2 / FGT-3
```

Create custom IPsec:

```text
Remote:
1.1.1.1

Interface:
ISP-1

PSK:
123456

IKE:
Version 1

Mode:
Aggressive

Peer ID:
Any

Encryption:
DES

Authentication:
MD5

DH:
Group 5
```

Phase 2:

```text
DH:
Group 5

Encryption:
DES

Authentication:
MD5

Auto-negotiate:
Enable

Protocols:
ALL

Subnets:
ALL
```

---

## Static Routes

Example:

```text
192.168.101.0/24
    |
    +-- IPsec Interface
```

Add a backup blackhole route:

```text
192.168.101.0/24
    |
    +-- Blackhole
    +-- AD: 254
```

Repeat for other spokes.

---

## Spoke Policies

```text
Incoming:
LAN
IPsec-link-1

Outgoing:
LAN
IPsec-link-1

Source:
ALL

Destination:
ALL

Service:
ALL

NAT:
Disable

Logging:
All Sessions
```

---

# 15. Policy-Based IPsec — Important Limitation

Example:

```text
Spoke-2
   |
   X
   |
Spoke-3
```

Traffic between spokes can be blocked because the hub is not automatically acting as an IPsec concentrator for those tunnels.

### Solution

Create an **IPsec Concentrator** on the hub.

Example:

```text
IPsec Concentrator
        |
        +-- IPsec X
        +-- IPsec Y
```

Where:

```text
IPsec X = FGT-2

IPsec Y = FGT-3
```

This allows the hub to provide the required hub-and-spoke / spoke-to-spoke forwarding behavior.

---

# 16. Policy-Based vs Interface-Based IPsec

| Feature                       | Interface-Based |               Policy-Based |
| ----------------------------- | --------------: | -------------------------: |
| IPsec interface               |             Yes |                         No |
| IPsec as routing interface    |             Yes |                         No |
| Action = IPsec                |              No |                        Yes |
| Static routing                |          Common | Limited / different design |
| Dynamic routing               |          Easier |                    Limited |
| GRE over IPsec                |        Suitable |      Not the normal design |
| Hub-and-spoke                 |             Yes |                        Yes |
| ADVPN scenarios               |             Yes |                     Useful |
| Remote-site initiated traffic |  Normal routing |     Requires policy option |

---

# 17. Policy-Based IPsec — Important Option

When creating custom IPsec, the option:

```text
Enable IPsec Interface Mode
```

### Enabled

You get:

```text
IPsec Interface
Routing
Interface-based policies
Advanced IPsec interface features
```

### Disabled

You primarily use:

```text
Policy Action:
IPsec

Add Route
```

---

## Remote-Initiated Traffic

For policy-based IPsec policies, enable:

```text
Allow traffic to be initiated
from the remote site
```

Without this option, traffic initiated from the remote side may not be forwarded as expected.

> [!NOTE]
> If you need to restrict the allowed networks, prefer defining precise Phase 2 selectors or using address objects in firewall policies rather than relying on broad `ALL` selectors.

---

# 18. IPsec with AWS

When interoperating with AWS, use modern cryptographic proposals supported by the AWS configuration.

Typical example:

```text
Encryption:
AES-128

Authentication:
SHA-2

DH:
Group 14
```

Use:

```text
Static Interface IP
```

Pay special attention to:

```text
NAT-T
UDP/500
UDP/4500
```

### Troubleshooting Checklist

```text
[ ] IKE proposal matches
[ ] Phase 1 authentication matches
[ ] DH group matches
[ ] Phase 2 proposal matches
[ ] PFS configuration matches
[ ] Local/remote selectors match
[ ] NAT-T works
[ ] UDP/500 allowed
[ ] UDP/4500 allowed
[ ] Firewall policies allow traffic
[ ] Routes point to IPsec
[ ] AWS tunnel configuration matches FortiGate
```

---

# 19. IPsec Troubleshooting Quick Reference

```bash
# IKE debug
diagnose debug enable
diagnose debug application ike -1

# Disable debug
diagnose debug disable

# IKE gateway state
diagnose vpn ike gateway list

# IPsec tunnel state
diagnose vpn tunnel list
```

---

# 20. Troubleshooting Flow

```text
                    IPsec Down
                        |
                        v
             +----------------------+
             | Check Phase 1        |
             +----------------------+
                        |
                        v
             +----------------------+
             | IKE Debug             |
             +----------------------+
                        |
                        v
             +----------------------+
             | PSK / Peer ID         |
             +----------------------+
                        |
                        v
             +----------------------+
             | Encryption / Hash     |
             +----------------------+
                        |
                        v
             +----------------------+
             | DH Group              |
             +----------------------+
                        |
                        v
             +----------------------+
             | Phase 2 Selectors     |
             +----------------------+
                        |
                        v
             +----------------------+
             | PFS / Proposal        |
             +----------------------+
                        |
                        v
             +----------------------+
             | Routes                |
             +----------------------+
                        |
                        v
             +----------------------+
             | Firewall Policies     |
             +----------------------+
                        |
                        v
             +----------------------+
             | NAT / NAT-T           |
             +----------------------+
```

---

# 21. GRE over IPsec Troubleshooting

Check in this order:

```text
1. Underlay connectivity
2. IKE SA
3. IPsec SA
4. IPsec interface IP
5. GRE interface
6. GRE source/destination
7. GRE protocol 47 in Phase 2
8. Routes
9. Firewall policies
10. MTU
```

Expected logical flow:

```text
LAN
 |
 v
GRE
 |
 v
IPsec
 |
 v
Internet
 |
 v
IPsec
 |
 v
GRE
 |
 v
Remote LAN
```

---

# 22. Security / Design Notes

> [!WARNING]
> The following examples use legacy algorithms such as **DES, MD5, IKEv1, and aggressive mode** because they reflect the lab configurations in this cheatsheet. They should not be considered preferred cryptography for new production deployments.

For production, prefer:

```text
IKEv2
AES
SHA-2
Strong DH groups
PFS
Strong PSK or certificate authentication
NAT-T when required
```

Avoid legacy:

```text
DES
3DES
MD5
Weak DH groups
IKEv1
Aggressive Mode
```

---

# 23. Quick Comparison

| Scenario                         | Recommended Design                          |
| -------------------------------- | ------------------------------------------- |
| Normal branch ↔ HQ               | Site-to-Site IPsec                          |
| Same subnet on both sites        | NAT + DNAT over IPsec                       |
| GRE required                     | GRE over IPsec                              |
| Remote FGT behind NAT            | Dial-up IPsec                               |
| Cisco GRE interoperability       | GRE over IPsec / transport mode as required |
| Hub-and-spoke                    | Interface-based IPsec / ADVPN               |
| Policy-driven VPN                | Policy-based IPsec                          |
| Spoke-to-spoke with policy IPsec | IPsec Concentrator                          |
| AWS Site-to-Site                 | Match AWS proposals + NAT-T                 |
| Dynamic routing over tunnel      | Interface-based IPsec                       |
| OSPF over VPN                    | Route-based/interface IPsec                 |
| BGP over VPN                     | Route-based/interface IPsec                 |

---

# 24. Final Validation Checklist

### Phase 1

```text
[ ] Remote gateway correct
[ ] Incoming interface correct
[ ] IKE version matches
[ ] Mode matches
[ ] Peer ID matches
[ ] PSK matches
[ ] Encryption matches
[ ] Authentication/hash matches
[ ] DH group matches
```

### Phase 2

```text
[ ] Local selector matches
[ ] Remote selector matches
[ ] Encryption matches
[ ] Authentication matches
[ ] PFS matches
[ ] DH group matches
[ ] Auto-negotiate configured if required
[ ] GRE protocol 47 configured when required
```

### Routing

```text
[ ] IPsec interface has correct IP
[ ] Remote network route exists
[ ] Backup blackhole route exists if required
[ ] Administrative distance / priority is correct
[ ] GRE route exists when GRE is used
```

### Firewall

```text
[ ] Incoming interface correct
[ ] Outgoing interface correct
[ ] Source object correct
[ ] Destination object correct
[ ] Service correct
[ ] NAT disabled where routing transparency is required
[ ] Logging enabled
[ ] Remote-initiated traffic enabled where required
```

### NAT / Overlap

```text
[ ] NAT pool correct
[ ] VIP/DNAT correct
[ ] Phase 2 selectors match translated networks
[ ] ARP Reply enabled when required
[ ] Subnet overlap permitted when required
```

### GRE

```text
[ ] GRE protocol 47
[ ] Local GRE gateway correct
[ ] Remote GRE gateway correct
[ ] GRE interface IP correct
[ ] GRE route exists
[ ] Policies include GRE
```

### Verification

```bash
diagnose vpn ike gateway list
diagnose vpn tunnel list

diagnose debug enable
diagnose debug application ike -1

# Perform test traffic

diagnose debug disable
```

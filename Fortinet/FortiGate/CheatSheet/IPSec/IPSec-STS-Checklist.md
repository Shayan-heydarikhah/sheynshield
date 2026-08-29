# 🔐 FortiGate IPsec VPN — Site-to-Site & Advanced Troubleshooting Checklist

> **SheynShield Engineering Checklist**
> **FortiGate IPsec • Site-to-Site VPN • GRE over IPsec • Policy-Based IPsec • AWS VPN • IPsec Concentrator • Overlapping Subnets**

---

## 🎯 What This Checklist Covers

Use this checklist to design, configure, validate, and troubleshoot:

* [ ] FortiGate Site-to-Site IPsec VPN
* [ ] Overlapping subnets over IPsec
* [ ] GRE over IPsec
* [ ] GRE over IPsec with Cisco
* [ ] Dial-up IPsec for remote FortiGate
* [ ] Policy-Based IPsec VPN
* [ ] IPsec Concentrator
* [ ] AWS Site-to-Site VPN interoperability
* [ ] Route-based / Interface-based IPsec
* [ ] IPsec Phase 1 / Phase 2 troubleshooting
* [ ] IKE debugging
* [ ] Routing and firewall validation
* [ ] GRE troubleshooting

---

# 🧭 1. IPsec Site-to-Site — Initial Validation

## Underlay Connectivity

* [ ] Local FortiGate can reach remote peer public IP
* [ ] Correct WAN interface is selected
* [ ] Correct source IP is used
* [ ] Upstream routing is correct
* [ ] No upstream firewall blocks IKE
* [ ] UDP/500 is reachable when required
* [ ] UDP/4500 is reachable when NAT-T is used
* [ ] NAT-T behavior has been verified

## Phase 1

* [ ] Remote gateway is correct
* [ ] Incoming interface is correct
* [ ] IKE version matches
* [ ] Negotiation mode matches
* [ ] Peer ID matches
* [ ] Authentication method matches
* [ ] Pre-shared key matches
* [ ] Encryption proposal matches
* [ ] Authentication/hash proposal matches
* [ ] DH group matches
* [ ] Local/remote identity configuration is correct
* [ ] DPD configuration is compatible

## Phase 2

* [ ] Local selector matches
* [ ] Remote selector matches
* [ ] Encryption proposal matches
* [ ] Authentication/hash matches
* [ ] PFS configuration matches
* [ ] DH group matches when PFS is enabled
* [ ] Auto-negotiate is configured when required
* [ ] Proxy IDs/selectors are correct
* [ ] GRE protocol 47 is configured when GRE is transported

---

# 🔎 2. IPsec Verification Commands

## Check IKE Gateway

```bash
diagnose vpn ike gateway list
```

Verify:

* [ ] Source address
* [ ] Destination address
* [ ] Tunnel ID
* [ ] Creation time
* [ ] Establishment time
* [ ] IKE SA
* [ ] IPsec SA

## Check IPsec Tunnel

```bash
diagnose vpn tunnel list
```

Verify:

* [ ] IKE version
* [ ] Tunnel ID
* [ ] Local networks
* [ ] Remote networks
* [ ] Reachable/advertised networks
* [ ] MTU
* [ ] DPD state
* [ ] NPU information

---

# 🧪 3. IKE Debug Checklist

Start debugging:

```bash
diagnose debug enable
diagnose debug application ike -1
```

Check:

* [ ] IKE version
* [ ] Exchange mode
* [ ] Peer ID
* [ ] Pre-shared key authentication
* [ ] Encryption proposal
* [ ] Authentication/hash
* [ ] DH group
* [ ] Local gateway
* [ ] Remote gateway
* [ ] NAT-T
* [ ] DPD
* [ ] Phase 1 authentication
* [ ] Phase 2 negotiation
* [ ] Proxy IDs/selectors

Stop debugging after testing:

```bash
diagnose debug disable
```

> [!WARNING]
> Do not leave verbose IKE debugging enabled during normal production operation.

---

# 🔀 4. Overlapping Subnets over IPsec

## Problem Identification

Example:

```text
FGT-1 LAN
192.168.101.0/24

        IPsec

FGT-2 LAN
192.168.101.0/24
```

* [ ] Confirm that both sites use overlapping address space
* [ ] Determine which translated networks will represent each site
* [ ] Ensure translated networks do not overlap

Example:

```text
FGT-1

Real:
192.168.101.0/24

Translated:
10.10.10.0/24
```

```text
FGT-2

Real:
192.168.101.0/24

Translated:
20.20.20.0/24
```

---

## FGT-1 — NAT Design

### Address Objects

* [ ] Local network = `192.168.101.0/24`
* [ ] Remote translated network = `20.20.20.0/24`

### IP Pool

```text
Name:
fpr-1

External:
10.10.10.1 - 10.10.10.254

Internal:
192.168.101.1 - 192.168.101.254
```

* [ ] Fixed Port Range IP Pool configured
* [ ] Internal range matches real LAN
* [ ] External range matches translated LAN
* [ ] ARP Reply enabled when required

### VIP / DNAT

```text
Name:
vip-101-over

Interface:
IPsec

Type:
Static

External:
10.10.10.1 - 10.10.10.254

Mapped:
192.168.101.1
```

* [ ] VIP uses correct IPsec interface
* [ ] External translated range is correct
* [ ] Mapped internal network is correct

### Phase 2

```text
Local:
10.10.10.0/24

Remote:
20.20.20.0/24
```

* [ ] Phase 2 uses translated networks
* [ ] Local selector is correct
* [ ] Remote selector is correct

### Routing

* [ ] Route to `20.20.20.0/24` points to IPsec
* [ ] Backup blackhole route exists if required
* [ ] Backup route has appropriate distance/priority

Example:

```text
20.20.20.0/24
    ├── IPsec
    └── Blackhole
         AD/Priority: 254
```

### Firewall Policy

#### LAN → IPsec

* [ ] Incoming interface = LAN
* [ ] Outgoing interface = IPsec
* [ ] Source = local address object
* [ ] Destination = remote translated network
* [ ] Required services are allowed
* [ ] Correct IP pool is selected
* [ ] Logging is enabled

#### IPsec → LAN

* [ ] Incoming interface = IPsec
* [ ] Outgoing interface = LAN
* [ ] Source = remote translated network
* [ ] Destination = VIP/DNAT object
* [ ] NAT is disabled where required
* [ ] Logging is enabled

---

# 🔁 5. Overlapping Subnets — FGT-2 Validation

* [ ] Real LAN = `192.168.101.0/24`
* [ ] Remote translated network = `10.10.10.0/24`
* [ ] IP pool translates local LAN to `20.20.20.0/24`
* [ ] ARP Reply configured when required
* [ ] VIP maps translated addresses back to the real LAN
* [ ] Phase 2 local selector = `20.20.20.0/24`
* [ ] Phase 2 remote selector = `10.10.10.0/24`
* [ ] Route to `10.10.10.0/24` points to IPsec
* [ ] Backup blackhole route is configured if required
* [ ] LAN → IPsec policy uses the correct IP pool
* [ ] IPsec → LAN policy uses the correct VIP/DNAT

### Connectivity Test

From FGT-1 LAN:

```bash
ping 20.20.20.2
```

Expected:

```text
20.20.20.2
    ↓
Translated destination
    ↓
192.168.101.2
```

> [!IMPORTANT]
> With overlapping networks, think in terms of **real addresses vs. translated addresses**. Phase 2 selectors, routing, NAT, and DNAT must all agree on the translated design.

---

# 🛡️ 6. GRE over IPsec

## Design Validation

```text
LAN
 ↓
GRE
 ↓
IPsec
 ↓
Internet
 ↓
IPsec
 ↓
GRE
 ↓
Remote LAN
```

* [ ] IPsec tunnel is established first
* [ ] IPsec interface addressing is correct
* [ ] GRE source is correct
* [ ] GRE destination is correct
* [ ] GRE interface is bound to IPsec
* [ ] GRE tunnel IPs are configured
* [ ] Remote LAN route points to GRE
* [ ] Firewall policies permit GRE/tunnel traffic
* [ ] MTU is considered

## Phase 2

```text
Protocol:
47
```

* [ ] Protocol 47 is configured when required by the design
* [ ] Phase 2 selectors match the GRE/IPsec design

> [!NOTE]
> GRE uses **IP protocol 47**. This is different from TCP/UDP port numbers.

---

# ⚙️ 7. GRE Configuration Validation

Example:

```bash
config system gre
    edit gre-1
        set interface ipsec-link-1
        set remote-gw 12.12.12.2
        set local-gw 12.12.12.1
    end
end
```

Validate:

* [ ] GRE interface exists
* [ ] Correct IPsec interface is selected
* [ ] Local GRE gateway is correct
* [ ] Remote GRE gateway is correct
* [ ] Tunnel IP addressing is correct
* [ ] Remote LAN route points to GRE
* [ ] Backup blackhole route exists when required

Example:

```text
192.168.102.0/24
    ├── GRE
    └── Blackhole
         AD: 254
```

---

# 🌐 8. GRE over IPsec — FortiGate Behind NAT

Use a **Dial-up IPsec** design when the remote FortiGate:

* [ ] Is behind NAT
* [ ] Does not have a stable public IP
* [ ] Cannot accept a traditional static peer design

Concept:

```text
FGT-1
  │
  │ Dial-up IPsec
  ▼
Internet
  │
  ▼
NAT
  │
  ▼
Remote FortiGate
```

### Hub

* [ ] Dial-up IPsec configured
* [ ] Peer ID configured correctly
* [ ] PSK/certificate authentication configured
* [ ] Phase 2 configured
* [ ] GRE protocol 47 handled
* [ ] IPsec interface addressing configured
* [ ] GRE interface created
* [ ] Remote LAN route points to GRE
* [ ] Firewall policies allow required traffic

### Spoke

* [ ] Dial-up/static client behavior configured correctly
* [ ] Hub public IP is reachable
* [ ] Phase 1 matches
* [ ] Phase 2 matches
* [ ] GRE interface is configured
* [ ] Remote LAN route points to GRE
* [ ] NAT behavior is correct

---

# 🔗 9. GRE over IPsec — Cisco Interoperability

## Cisco GRE

```cisco
interface tunnel1
 ip address 10.10.10.2 255.255.255.0
 tunnel source 20.20.20.2
 tunnel destination 20.20.20.1
```

* [ ] Tunnel source is correct
* [ ] Tunnel destination is correct
* [ ] Tunnel IP is correct

## Cisco Phase 1

```cisco
crypto isakmp policy 10
 encr des
 hash md5
 authentication pre-share
 group 2
```

* [ ] Encryption matches
* [ ] Hash matches
* [ ] Authentication method matches
* [ ] DH group matches
* [ ] PSK matches

## Cisco Phase 2

```cisco
crypto ipsec transform-set greipsec_transform esp-des esp-md5-hmac
 mode transport
```

* [ ] Transform set matches FortiGate
* [ ] Transport mode is used when required

## Cisco IPsec Profile

```cisco
crypto ipsec profile greipsec_profile
 set transform-set greipsec_transform
```

## Cisco VTI

```cisco
interface tunne10
 ip address 20.20.20.2 255.255.255.0
 tunnel source 16.16.16.2
 tunnel destination 16.16.16.1
 tunnel mode ipsec ipv4
 tunnel protection ipsec profile greipsec_profile
```

* [ ] VTI addressing matches
* [ ] Peer addresses match
* [ ] IPsec profile is applied

---

# 🧩 10. FortiGate ↔ Cisco GRE/IPsec

## Phase 1

* [ ] Remote gateway = Cisco peer
* [ ] Correct WAN interface
* [ ] PSK matches
* [ ] IKE version matches
* [ ] Mode matches
* [ ] Encryption matches
* [ ] Authentication/hash matches
* [ ] DH group matches

## Phase 2

* [ ] Encryption matches
* [ ] Authentication/hash matches
* [ ] PFS matches
* [ ] Selectors are correct
* [ ] GRE protocol 47 is handled

## IPsec Interface

Example:

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

* [ ] GRE interface exists
* [ ] Correct IPsec interface is used
* [ ] Local gateway is correct
* [ ] Remote gateway is correct
* [ ] GRE tunnel IPs match

---

## Transport Mode

When required by the interoperability design:

```bash
config vpn ipsec phase2-interface
    edit ipsec-cisco-16
        set encapsulation transport-mode
    end
end
```

> [!WARNING]
> Do not enable transport mode blindly. Cisco interoperability depends on the exact GRE/IPsec design and cryptographic parameters. Validate the resulting IKE/IPsec SAs after making the change.

---

# 🔄 11. Policy-Based IPsec VPN

Use policy-based IPsec when:

* [ ] IPsec interface is not required
* [ ] Traffic should trigger the VPN through firewall policy
* [ ] A policy-driven VPN architecture is desired
* [ ] Remote-site initiated traffic must be explicitly permitted

Enable the feature:

```text
System
 → Feature Visibility
 → Policy-based IPsec VPN
```

---

# 🏗️ 12. Policy-Based IPsec — Hub

## Phase 1

* [ ] IPsec Interface Mode = Disabled
* [ ] Correct remote gateway
* [ ] Correct WAN interface
* [ ] PSK matches
* [ ] IKE version matches
* [ ] Mode matches
* [ ] Peer ID matches
* [ ] Encryption matches
* [ ] Authentication/hash matches
* [ ] DH group matches

## Phase 2

* [ ] DH group matches
* [ ] Encryption matches
* [ ] Authentication matches
* [ ] PFS matches
* [ ] Auto-negotiate configured if required
* [ ] Protocols are correct
* [ ] Selectors are correct

## Firewall Policy

* [ ] Incoming interface = LAN
* [ ] Outgoing interface = WAN
* [ ] Action = IPsec
* [ ] Correct VPN tunnel selected
* [ ] Remote-site initiated traffic option enabled when required
* [ ] Source addresses are restricted
* [ ] Destination addresses are restricted
* [ ] NAT is disabled where required
* [ ] Logging is enabled

---

# 🔁 13. Policy-Based IPsec — Spoke

* [ ] Hub public IP is reachable
* [ ] Phase 1 matches hub
* [ ] Phase 2 matches hub
* [ ] Correct WAN interface selected
* [ ] Static routes are correct where required
* [ ] Backup blackhole route is configured where required
* [ ] LAN ↔ IPsec policies are correct
* [ ] NAT is disabled where required
* [ ] Remote-initiated traffic behavior is understood

Example route:

```text
192.168.101.0/24
    ↓
IPsec
```

Backup:

```text
192.168.101.0/24
    ↓
Blackhole
    AD: 254
```

---

# 🚧 14. Policy-Based IPsec — Spoke-to-Spoke Limitation

Example:

```text
        HUB
       /   \
      /     \
   Spoke-2 Spoke-3
```

Validate:

* [ ] Hub can establish both IPsec tunnels
* [ ] Spoke-2 can reach hub
* [ ] Spoke-3 can reach hub
* [ ] Spoke-to-spoke forwarding requirement is identified
* [ ] IPsec Concentrator requirement is evaluated

## IPsec Concentrator

Concept:

```text
IPsec Concentrator
        |
        +── IPsec X
        |
        +── IPsec Y
```

* [ ] IPsec X represents the first spoke
* [ ] IPsec Y represents the second spoke
* [ ] Concentrator is configured when required
* [ ] Hub forwarding behavior is validated

---

# 📊 15. Interface-Based vs Policy-Based IPsec

| Capability                      | Interface-Based |              Policy-Based              |
| ------------------------------- | :-------------: | :------------------------------------: |
| IPsec interface                 |        ✅        |                    ❌                   |
| Routing through IPsec interface |        ✅        |                    ❌                   |
| Policy action = IPsec           |        ❌        |                    ✅                   |
| Static routing                  |        ✅        |            Different design            |
| Dynamic routing                 |        ✅        |                 Limited                |
| GRE over IPsec                  |        ✅        |          Not the normal design         |
| Route-based design              |        ✅        |                    ❌                   |
| Hub-and-spoke                   |        ✅        |                    ✅                   |
| Remote-site initiated traffic   |      Normal     | Explicit policy option may be required |

### Decision Checklist

* [ ] Need dynamic routing → prefer interface-based
* [ ] Need OSPF/BGP → prefer interface-based
* [ ] Need GRE over IPsec → interface-based is the normal design
* [ ] Need traditional policy-triggered VPN → consider policy-based
* [ ] Need IPsec interface features → use interface-based
* [ ] Need spoke-to-spoke forwarding in policy-based design → evaluate IPsec Concentrator

---

# ☁️ 16. FortiGate IPsec with AWS

For AWS Site-to-Site VPN interoperability:

* [ ] AWS tunnel parameters are documented
* [ ] FortiGate Phase 1 proposal matches AWS
* [ ] Authentication method matches
* [ ] DH group matches
* [ ] Phase 2 proposal matches
* [ ] PFS configuration matches
* [ ] Local/remote selectors match
* [ ] NAT-T works
* [ ] UDP/500 is allowed
* [ ] UDP/4500 is allowed when NAT-T is used
* [ ] Firewall policies permit VPN traffic
* [ ] Routing is correct
* [ ] Return path is correct

Example modern-style proposal:

```text
Encryption:
AES-128

Authentication:
SHA-2

DH:
Group 14
```

> [!IMPORTANT]
> AWS interoperability should be built from the **actual AWS tunnel configuration** rather than blindly copying a generic proposal. Cryptographic proposals and tunnel parameters can vary by AWS configuration.

---

# 🧪 17. AWS VPN Verification

### IKE

* [ ] IKE SA is established
* [ ] Correct AWS peer is detected
* [ ] Correct proposal is negotiated
* [ ] NAT-T state is correct

### IPsec

* [ ] IPsec SA is established
* [ ] Correct selectors are installed
* [ ] PFS matches
* [ ] Encryption/authentication matches

### Network

* [ ] AWS route exists
* [ ] FortiGate route exists
* [ ] Return route exists
* [ ] Security controls allow traffic
* [ ] MTU is considered

---

# 🧭 18. IPsec Routing Checklist

## Primary Path

* [ ] Remote network has a route
* [ ] Route points to correct IPsec interface
* [ ] Administrative distance/priority is correct
* [ ] Return route exists

## Backup Path

* [ ] Backup route exists
* [ ] Backup route has higher distance/priority
* [ ] Blackhole route is configured where appropriate
* [ ] Failover behavior has been tested

Example:

```text
Remote Network
      |
      +── IPsec       ← Primary
      |
      +── Blackhole   ← Backup
           AD: 254
```

## Recursive Routing Protection

When the VPN peer's public IP is reached through the VPN:

* [ ] Peer public IP has a route through the local WAN
* [ ] Peer traffic does not recursively enter the VPN
* [ ] Default route does not accidentally capture the peer address
* [ ] Failover routing does not create a recursive dependency

---

# 🔥 19. Firewall Policy Checklist

For each IPsec deployment:

* [ ] Correct incoming interface
* [ ] Correct outgoing interface
* [ ] Correct source address
* [ ] Correct destination address
* [ ] Required services allowed
* [ ] NAT behavior is intentional
* [ ] Logging enabled
* [ ] Security profiles applied where appropriate
* [ ] Remote-initiated traffic permitted when required
* [ ] Return traffic is permitted
* [ ] Policy order is correct

> [!WARNING]
> Avoid using unrestricted `ALL → ALL` policies in production unless the broad access is explicitly required and documented.

---

# 🔐 20. Security Validation

The lab examples may contain legacy cryptography such as:

```text
DES
MD5
IKEv1
Aggressive Mode
Weak DH groups
```

* [ ] Legacy algorithms are used only when interoperability/lab requirements demand them
* [ ] Production deployment uses current FortiOS-supported strong cryptography
* [ ] IKEv2 is preferred where supported
* [ ] AES is preferred
* [ ] SHA-2 is preferred
* [ ] Strong DH groups are selected
* [ ] PFS is enabled where appropriate
* [ ] Strong PSKs are used
* [ ] Certificate authentication is considered where appropriate
* [ ] NAT-T is enabled/used when required

> [!WARNING]
> **DES and MD5 should not be treated as recommended production cryptography.** The examples in this checklist reflect legacy/interoperability lab scenarios.

---

# 🧪 21. GRE over IPsec Troubleshooting

Check in this exact order:

* [ ] Underlay connectivity
* [ ] IKE SA
* [ ] IPsec SA
* [ ] IPsec interface IP
* [ ] GRE interface
* [ ] GRE local gateway
* [ ] GRE remote gateway
* [ ] GRE protocol 47
* [ ] Routes
* [ ] Firewall policies
* [ ] MTU

### Expected Flow

```text
LAN
 ↓
GRE
 ↓
IPsec
 ↓
Internet
 ↓
IPsec
 ↓
GRE
 ↓
Remote LAN
```

If the IPsec SA is UP but GRE is DOWN:

* [ ] Check GRE source
* [ ] Check GRE destination
* [ ] Check GRE interface binding
* [ ] Check tunnel IP addresses
* [ ] Check protocol 47
* [ ] Check routing
* [ ] Check MTU

---

# 🔍 22. IPsec Troubleshooting Decision Tree

```text
                    IPsec DOWN
                        |
                        v
              Underlay Reachability?
                  /             \
                NO              YES
                |                |
        Fix WAN/Routing          v
                           Phase 1?
                           /      \
                         NO       YES
                         |         |
                 IKE Debug         v
                         |      Phase 2?
                         |      /     \
                         |    NO      YES
                         |    |        |
                         |  Selectors   v
                         |  /Proposal  Routing?
                         |             /    \
                         |           NO      YES
                         |           |        |
                         |        Fix Route   v
                         |                 Policy?
                         |                 /    \
                         |               NO      YES
                         |               |        |
                         |            Fix Policy  v
                         |                    NAT / MTU
                         |                       |
                         +-----------------------+
```

---

# 🧰 23. Core FortiGate IPsec Commands

## IKE Debug

```bash
diagnose debug enable
diagnose debug application ike -1
```

## Disable Debug

```bash
diagnose debug disable
```

## IKE Gateway

```bash
diagnose vpn ike gateway list
```

## IPsec Tunnel

```bash
diagnose vpn tunnel list
```

---

# 📝 24. Final Site-to-Site IPsec Checklist

## Phase 1

* [ ] Remote gateway correct
* [ ] WAN interface correct
* [ ] IKE version correct
* [ ] Mode correct
* [ ] Peer ID correct
* [ ] Authentication correct
* [ ] PSK correct
* [ ] Encryption correct
* [ ] Hash/authentication correct
* [ ] DH group correct
* [ ] DPD/NAT-T behavior correct

## Phase 2

* [ ] Local selector correct
* [ ] Remote selector correct
* [ ] Encryption correct
* [ ] Authentication correct
* [ ] PFS correct
* [ ] DH group correct
* [ ] Auto-negotiate correct
* [ ] GRE protocol 47 configured when required

## Routing

* [ ] Remote network route exists
* [ ] IPsec interface is selected
* [ ] Route priority is correct
* [ ] Return route exists
* [ ] Backup route exists if required
* [ ] Blackhole route is correctly configured
* [ ] Peer public IP remains reachable through WAN

## Firewall

* [ ] LAN → IPsec policy exists
* [ ] IPsec → LAN policy exists
* [ ] Source is restricted
* [ ] Destination is restricted
* [ ] Required services are allowed
* [ ] NAT behavior is intentional
* [ ] Logging is enabled
* [ ] Policy order is correct

## Verification

```bash
diagnose vpn ike gateway list
diagnose vpn tunnel list
```

* [ ] IKE SA = UP
* [ ] IPsec SA = UP
* [ ] Correct selectors installed
* [ ] Correct routes installed
* [ ] Traffic counters increase
* [ ] Ping test succeeds
* [ ] Application traffic succeeds

---

# 🚀 25. Advanced IPsec Design Checklist

Before deploying:

* [ ] Is this interface-based or policy-based IPsec?
* [ ] Is dynamic routing required?
* [ ] Is GRE required?
* [ ] Are subnets overlapping?
* [ ] Is NAT required?
* [ ] Is DNAT/VIP required?
* [ ] Is the remote peer behind NAT?
* [ ] Is a dial-up design required?
* [ ] Is Cisco interoperability required?
* [ ] Is AWS interoperability required?
* [ ] Is an IPsec Concentrator required?
* [ ] Is spoke-to-spoke traffic required?
* [ ] Is failover required?
* [ ] Is a backup blackhole route required?
* [ ] Is NAT-T required?
* [ ] Are modern cryptographic algorithms available?
* [ ] Has MTU been validated?
* [ ] Has failover been tested?
* [ ] Has return routing been validated?

---

# 🧠 26. Exam & Interview Quick Notes

> **Remember these concepts:**

* [ ] Site-to-Site IPsec = secure connectivity between fixed peers
* [ ] Phase 1 = IKE negotiation and authentication
* [ ] Phase 2 = IPsec SA and traffic selectors
* [ ] `diagnose vpn ike gateway list` = IKE gateway state
* [ ] `diagnose vpn tunnel list` = IPsec tunnel/SA information
* [ ] GRE = IP protocol `47`
* [ ] Overlapping subnets require a translation strategy
* [ ] Translated networks must be reflected in the VPN design
* [ ] Interface-based IPsec is the normal choice for route-based designs
* [ ] OSPF/BGP generally fit better with interface-based IPsec
* [ ] Policy-based IPsec uses firewall policy action `IPsec`
* [ ] Remote-initiated traffic requires the appropriate policy option in policy-based designs
* [ ] IPsec Concentrator can be required for hub forwarding in policy-based designs
* [ ] AWS VPN parameters must match the AWS tunnel configuration
* [ ] NAT-T commonly uses UDP/4500
* [ ] IKE commonly uses UDP/500
* [ ] GRE troubleshooting starts with IPsec and then moves upward to GRE
* [ ] Routing and firewall policy are just as important as IKE/IPsec negotiation

---

# ⚡ 27. One-Minute IPsec Troubleshooting

```text
1. Can I reach the peer public IP?
        ↓
2. Is IKE SA UP?
        ↓
3. Is IPsec SA UP?
        ↓
4. Are Phase 2 selectors correct?
        ↓
5. Is the route correct?
        ↓
6. Is the firewall policy correct?
        ↓
7. Is NAT intentional?
        ↓
8. Is return routing correct?
        ↓
9. If GRE is used, is protocol 47 working?
        ↓
10. Is MTU causing packet loss?
```

---

# 🏆 Final Mental Model

```text
                 FORTIGATE IPSEC
                       |
       +---------------+---------------+
       |               |               |
   Site-to-Site    Policy-Based      Dial-up
       |               |               |
       |               |               |
   Routing         Firewall         Remote Peer
       |             Action             |
       |             IPsec              |
       |                                |
       +---------------+----------------+
                       |
                    Phase 1
                       |
                  IKE / Auth
                       |
                    Phase 2
                       |
                IPsec SA / Selectors
                       |
                    Routing
                       |
                  Firewall
                       |
                 Application
```

### Golden Rule

> **IPsec being UP does not automatically mean the application will work.**

Always validate the complete path:

```text
Underlay
   ↓
IKE
   ↓
IPsec SA
   ↓
Selectors
   ↓
Routing
   ↓
Firewall Policy
   ↓
NAT
   ↓
Return Path
   ↓
Application
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

## 🏷️ Keywords

`FortiGate` `IPsec VPN` `Site-to-Site VPN` `IPsec Troubleshooting` `GRE over IPsec` `Policy-Based IPsec` `Route-Based IPsec` `IPsec Concentrator` `AWS VPN` `Cisco IPsec` `Overlapping Subnets` `NAT over IPsec` `Fortinet` `FortiOS` `Network Security` `VPN Troubleshooting`

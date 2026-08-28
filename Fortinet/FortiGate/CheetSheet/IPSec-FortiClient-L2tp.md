# FortiGate VPN Cheat Sheet
## Dialup IPsec with FortiClient + L2TP over IPsec

---

# 1. Dialup IPsec with FortiClient

## Topology

```text
                    Internet
                       |
                +------+------+
                |  FortiGate  |
                | Dialup VPN  |
                +------+------+
                       |
              IPsec Dialup Tunnel
                       |
                +------+------+
                | FortiClient |
                |    PC/User  |
                +-------------+
````

---

## 1.1 IPsec Configuration

### Phase 1

Go to:

```text
VPN
└── IPsec Tunnels
    └── Create New
        └── Custom
```

Configure:

| Setting            | Value                       |
| ------------------ | --------------------------- |
| Remote Gateway     | Dialup User                 |
| Incoming Interface | Internet/WAN Interface      |
| Mode Config        | Range                       |
| IP Range           | `10.10.10.10 - 10.10.10.20` |
| Subnet Mask        | `255.255.255.255`           |
| IKE Version        | IKEv1                       |
| Mode               | Aggressive                  |
| Peer ID            | Any                         |
| Authentication     | Pre-shared Key              |
| Encryption         | DES                         |
| Integrity/Hash     | MD5                         |
| DH Group           | 5                           |
| XAuth              | Enabled                     |
| Auto-negotiate     | Enabled                     |

### Mode Config

```text
Mode Config
    └── Range
        ├── Start IP : 10.10.10.10
        ├── End IP   : 10.10.10.20
        └── Netmask  : 255.255.255.255
```

### Advanced Settings

Enable:

```text
Add Route
```

---

## 1.2 XAuth

XAuth can be integrated with the same:

```text
Active Directory
        |
        +── User
        |
        +── User Group
        |
        +── FortiGate XAuth
```

Use the required AD users/groups for authentication.

---

## 1.3 Phase 2

Use:

```text
Phase 2
    ├── Source: All required networks
    ├── Destination: All required networks
    ├── Encryption: DES
    ├── Authentication: MD5 / SHA1
    ├── PFS: Enabled if required
    ├── DH Group: 5
    └── Auto-negotiate: Enabled
```

Example:

```text
Phase 2 Proposal

Encryption      : DES
Authentication  : MD5 / SHA1
PFS             : DH Group 5
Auto-negotiate  : Enable
```

---

# 2. Firewall Policies

## 2.1 Incoming Traffic

```text
Dialup VPN
    |
    +----> LAN
    |
    +----> DMZ
```

Create policy:

```text
Incoming Interface:
    IPsec Dialup

Outgoing Interface:
    LAN / DMZ

Source:
    VPN Users / User Groups

Destination:
    Required LAN / DMZ Networks

Service:
    ALL

NAT:
    DISABLE

Logging:
    ALL SESSIONS
```

---

## 2.2 Outgoing Traffic

```text
LAN
DMZ
 |
 +----> Dialup VPN
```

Create policy:

```text
Incoming Interface:
    LAN / DMZ

Outgoing Interface:
    IPsec Dialup

Source:
    LAN / DMZ Networks

Destination:
    VPN Users / VPN Address Range

Service:
    ALL

NAT:
    DISABLE

Logging:
    ALL SESSIONS
```

---

# 3. FortiClient Configuration

On the client PC:

```text
FortiClient
    |
    └── VPN
        |
        └── IPsec VPN
```

The following parameters must match the FortiGate configuration:

```text
IKE Version
Encryption
Hash / Authentication
DH Group
Authentication Method
Pre-shared Key
Phase 1 Proposal
Phase 2 Proposal
XAuth
Mode Config
```

### Important

```text
FortiGate
    ⇅
All VPN proposals/settings
    ⇅
FortiClient
```

If Phase 1 or Phase 2 proposals do not match, the tunnel will not establish.

---

## 3.1 FortiClient Mode Config

Enable:

```text
Mode Config
```

The FortiClient receives an IP from the configured range:

```text
10.10.10.10
10.10.10.11
10.10.10.12
...
10.10.10.20
```

Example:

```text
FortiGate VPN Pool
    10.10.10.10 - 10.10.10.20
             |
             v
       FortiClient
             |
             +---- 10.10.10.10
```

---

## 3.2 DPD / NAT-T

For remote-access IPsec, verify:

```text
DPD
    └── Enabled

NAT Detection / NAT-T
    └── Enabled
```

NAT-T is especially important when the client is behind a NAT device.

---

# 4. Dialup IPsec Troubleshooting

## Check IKE Gateway

```bash
diagnose vpn ike gateway list
```

Useful for checking:

```text
Peer IP
SA
IKE version
Proposal
Authentication
Tunnel state
```

---

## Check IPsec Tunnel

```bash
diagnose vpn tunnel list
```

Useful for checking:

```text
IPsec SA
SPI
Phase 2
Proposal
Source/Destination
Tunnel state
```

---

## IKE Debug

```bash
diagnose debug application ike -1
diagnose debug enable
```

Stop debug:

```bash
diagnose debug disable
diagnose debug reset
```

---

# 5. L2TP over IPsec

L2TP itself does not provide strong encryption.

Therefore:

```text
L2TP
  +
IPsec
  =
Encrypted L2TP VPN
```

---

# 5.1 L2TP Server Configuration

CLI:

```bash
config vpn l2tp
    set status enable
    set eip 10.10.10.20
    set sip 10.10.10.10
    set usrgrp test
    set enforce-ipsec enable
end
```

### Address Pool

```text
Start IP:
    10.10.10.10

End IP:
    10.10.10.20
```

User authentication:

```text
User Group:
    test
```

IPsec enforcement:

```text
enforce-ipsec:
    enable
```

---

# 6. IPsec Phase 1 for L2TP

Create:

```text
VPN
└── IPsec Tunnels
    └── Custom
```

Configure:

```text
Remote Gateway:
    Dialup User

Incoming Interface:
    WAN / Internet

Authentication:
    Pre-shared Key

IKE Version:
    IKEv1

Mode:
    Main Mode

Peer ID:
    Any

Encryption:
    DES

Authentication / Hash:
    MD5 / SHA1

DH Group:
    2
```

---

# 7. L2TP IPsec Phase 2

Configure:

```text
Phase 2
    ├── Encryption      : DES
    ├── Authentication  : MD5 / SHA1
    ├── PFS              : DH Group 2
    ├── Networks         : All required subnets
    └── Auto-negotiate   : Enable
```

Example:

```text
Encryption:
    DES

Authentication:
    MD5 / SHA1

PFS:
    DH Group 2

Selectors:
    All required networks

Auto-negotiate:
    Enable
```

---

# 8. L2TP Firewall Policies

## Incoming

```text
L2TP Interface
      |
      +----> LAN
      |
      +----> DMZ
```

Policy:

```text
Incoming Interface:
    L2TP

Outgoing Interface:
    LAN / DMZ

Source:
    User Group

Destination:
    LAN / DMZ

Service:
    ALL

NAT:
    DISABLE

Logging:
    ALL SESSIONS
```

---

## Outgoing

```text
LAN
DMZ
 |
 +----> L2TP
```

Policy:

```text
Incoming Interface:
    LAN / DMZ

Outgoing Interface:
    L2TP

Source:
    LAN / DMZ

Destination:
    L2TP Clients

Service:
    ALL

NAT:
    DISABLE

Logging:
    ALL SESSIONS
```

---

# 9. L2TP using FortiGate IPsec Wizard

Alternative method:

```text
VPN
└── IPsec Wizard
    └── Remote Access
        └── Windows Native
```

Select:

```text
Remote Access VPN
    |
    +── Windows Native / L2TP
```

Configure:

```text
Incoming Interface:
    WAN

Authentication:
    Pre-shared Key

User Group:
    Required VPN User Group
```

---

# 10. L2TP Client IP Assignment

Configure the local interface and client IP range for L2TP connections.

Example:

```text
L2TP Client Pool

172.16.0.10
172.16.0.11
172.16.0.12
...
172.16.0.254
```

Recommended example subnet:

```text
172.16.0.0/16
```

---

## /32 Mask

If the client receives:

```text
IP:
    172.16.0.10/32
```

The `/32` mask can provide client isolation behavior because the client has only a host route rather than a directly connected subnet.

```text
Client
172.16.0.10/32
      |
      X
Direct L2 client-to-client subnet behavior
```

---

# 11. Wizard-Based L2TP

When using the FortiGate IPsec Wizard:

```text
IPsec Wizard
      |
      +── Windows Native
              |
              +── L2TP
```

FortiGate can automatically create the required policies.

Therefore:

```text
Wizard
  |
  +── VPN configuration
  +── User authentication
  +── IP assignment
  +── Firewall policies
```

---

# 12. Windows Native VPN Client

On Windows:

```text
Settings
└── Network & Internet
    └── VPN
        └── Add VPN
```

Use:

```text
VPN Provider:
    Windows (built-in)

Connection Name:
    FortiGate-L2TP

Server Address:
    <FortiGate Public IP>

VPN Type:
    L2TP/IPsec with pre-shared key

Pre-shared Key:
    <Configured PSK>

Sign-in Type:
    Username and password
```

Use the credentials configured in the FortiGate user group.

---

# 13. Quick Comparison

| Feature          | Dialup IPsec + FortiClient | L2TP over IPsec           |
| ---------------- | -------------------------- | ------------------------- |
| Client           | FortiClient                | Native Windows VPN        |
| IPsec            | Yes                        | Yes                       |
| L2TP             | No                         | Yes                       |
| Mode Config      | Yes                        | Yes                       |
| XAuth            | Yes                        | User authentication       |
| IKEv1            | Yes                        | Yes                       |
| Main/Aggressive  | Aggressive                 | Main                      |
| NAT-T            | Recommended                | Recommended               |
| DPD              | Recommended                | Recommended               |
| Client IP Pool   | `10.10.10.10-10.10.10.20`  | `10.10.10.10-10.10.10.20` |
| Firewall NAT     | Disable                    | Disable                   |
| Client isolation | Depends on design          | `/32` can isolate clients |
| Native Windows   | No                         | Yes                       |
| FortiClient      | Yes                        | Not required              |

---

# 14. Important Troubleshooting Flow

```text
Client
  |
  v
Internet
  |
  v
FortiGate
  |
  +---- Phase 1
  |       |
  |       +── IKE Version
  |       +── Encryption
  |       +── Hash
  |       +── DH
  |       +── Authentication
  |       +── Peer ID
  |
  +---- Phase 2
  |       |
  |       +── Encryption
  |       +── Authentication
  |       +── PFS / DH
  |       +── Selectors
  |
  +---- XAuth / User Authentication
  |
  +---- Mode Config
  |
  +---- Firewall Policy
  |
  +---- LAN / DMZ
```

### Debug Checklist

```text
[ ] WAN interface reachable
[ ] UDP/500 reachable
[ ] UDP/4500 reachable when NAT-T is used
[ ] PSK matches
[ ] IKE version matches
[ ] Phase 1 proposal matches
[ ] DH group matches
[ ] Peer ID matches
[ ] XAuth credentials are valid
[ ] User exists in correct group
[ ] Phase 2 proposal matches
[ ] Phase 2 selectors match
[ ] Mode Config enabled
[ ] Client receives VPN IP
[ ] Firewall policies exist
[ ] NAT disabled
[ ] DPD enabled
[ ] NAT-T enabled
[ ] LAN/DMZ routes are correct
```

---

# 15. Useful FortiGate Commands

## IKE Gateway

```bash
diagnose vpn ike gateway list
```

## IPsec Tunnel

```bash
diagnose vpn tunnel list
```

## IKE Debug

```bash
diagnose debug application ike -1
diagnose debug enable
```

Stop:

```bash
diagnose debug disable
diagnose debug reset
```

## Flush IKE

```bash
diagnose vpn ike gateway flush
```

---

# 16. Core Concept

```text
                    DIALUP IPSEC
                         |
             +-----------+-----------+
             |                       |
             v                       v
       FortiClient                L2TP
             |                       |
             |                       |
          IPsec                   IPsec
             |                       |
             +-----------+-----------+
                         |
                      FortiGate
                         |
                  Authentication
                         |
                    User / Group
                         |
                  LAN / DMZ Access
```

### Remember

```text
Dialup IPsec + FortiClient
    = FortiClient-based remote access IPsec

L2TP + IPsec
    = Native Windows remote-access VPN

Both require:
    - Correct Phase 1
    - Correct Phase 2
    - Authentication
    - Correct IP assignment
    - Correct firewall policies
    - Correct routing
    - NAT disabled for internal VPN traffic
```

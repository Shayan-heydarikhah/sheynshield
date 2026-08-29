# VXLAN over Dialup IPsec

> **Goal:** Build a Layer-2 VXLAN extension between two FortiGate devices, then protect the VXLAN transport using a **Dialup IPsec tunnel**.

---

# 1. VXLAN Without IPsec

## Topology

```text
        FGT-1                                      FGT-2
   +-------------+                            +-------------+
   |   FortiGate |                            |   FortiGate |
   |             |                            |             |
   |  VXLAN 1000 |============================| VXLAN 1000  |
   |             |        Underlay IP         |             |
   +------+------+                            +------+------+
          |                                           |
       Port3                                        Port3
          |                                           |
       VLAN 101                                     VLAN 101
          |                                           |
       LAN Switch                                  LAN Switch
          |                                           |
       Clients                                     Clients

       192.168.101.0/24  <---- L2 Extension ---->  192.168.101.0/24
```

### VXLAN Parameters

| Parameter | FGT-1 | FGT-2 |
|---|---|---|
| VXLAN Interface | `vx101` | `vx101` |
| VNI | `1000` | `1000` |
| Underlay Interface | `port2` | `port2` |
| Remote IP | `2.2.2.1` | `1.1.1.1` |
| VLAN | `101` | `101` |
| LAN IP | `192.168.101.1/24` | `192.168.101.2/24` |
| Software Switch | `ss-vx-vl-101` | `ss-vx-vl-101` |

---

# 2. FGT-1 — VXLAN

## Create VXLAN Interface

```bash
config system vxlan
    edit vx101
        set remote-ip 2.2.2.1
        set interface port2
        set vni 1000
    next
end
```

---

## Create VLAN on VXLAN Interface

```text
Interface:
    vx101

VLAN ID:
    101

Name:
    vx-vlan-101
```

Concept:

```text
vx101
  |
  +── VLAN 101
       |
       └── vx-vlan-101
```

---

## Create VLAN on Dedicated LAN Interface

```text
Interface:
    port3

VLAN ID:
    101

Name:
    vlan-101
```

Concept:

```text
port3
  |
  +── VLAN 101
       |
       └── vlan-101
```

---

# 3. Software Switch — FGT-1

Create:

```text
Name:
    ss-vx-vl-101

Type:
    Software Switch
```

Add:

```text
Members:
    vlan-101
    vx-vlan-101
```

Assign:

```text
IP:
    192.168.101.1/24

Ping:
    Enable

DHCP:
    Enable
```

Final logical structure:

```text
                 ss-vx-vl-101
                  /          \
                 /            \
          vlan-101           vx-vlan-101
             |                    |
           port3                 vx101
             |                    |
          LAN Switch          VXLAN VNI 1000
```

---

# 4. FGT-2 — VXLAN

## Create VXLAN Interface

```bash
config system vxlan
    edit vx101
        set remote-ip 1.1.1.1
        set interface port2
        set vni 1000
    next
end
```

---

## Create VLAN on VXLAN Interface

```text
Interface:
    vx101

VLAN ID:
    101

Name:
    vx-vlan-101
```

---

## Create VLAN on Dedicated LAN Interface

```text
Interface:
    port3

VLAN ID:
    101

Name:
    vlan-101
```

---

# 5. Software Switch — FGT-2

Create:

```text
Name:
    ss-vx-vl-101

Type:
    Software Switch
```

Members:

```text
vlan-101
vx-vlan-101
```

Assign:

```text
IP:
    192.168.101.2/24

Ping:
    Enable
```

DHCP:

```text
DHCP Mode:
    Relay

DHCP Server:
    192.168.101.1
```

Logical structure:

```text
                 ss-vx-vl-101
                  /          \
                 /            \
          vlan-101           vx-vlan-101
             |                    |
           port3                 vx101
             |                    |
          LAN Switch          VXLAN VNI 1000
```

---

# 6. VXLAN Firewall Policies

## FGT-1

### Incoming

```text
Incoming Interfaces:
    vx101
    ss-vx-vl-101
    ISP Link
```

### Outgoing

```text
Outgoing Interfaces:
    vx101
    ss-vx-vl-101
    ISP Link
```

Policy:

```text
Source:
    ALL

Destination:
    ALL

Service:
    ALL

NAT:
    DISABLE

Logging:
    ALL SESSIONS
```

---

## FGT-2

Use the same policy model:

```text
Incoming:
    vx101
    ss-vx-vl-101
    ISP Link

Outgoing:
    vx101
    ss-vx-vl-101
    ISP Link

Source:
    ALL

Destination:
    ALL

Service:
    ALL

NAT:
    DISABLE

Logging:
    ALL SESSIONS
```

---

# 7. Cisco LAN Switch Configuration

Example:

```cisco
enable
configure terminal

interface GigabitEthernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk

interface range GigabitEthernet0/0-1
 switchport mode access
 switchport access vlan 101

end

write memory
```

Expected LAN:

```text
              Switch
                |
         +------+------+
         |             |
       VLAN 101      VLAN 101
         |             |
       Client        Client
```

---

# 8. VXLAN Test

After configuration:

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
Remote VXLAN
   |
   v
Remote Software Switch
   |
   v
Remote VLAN 101
   |
   v
Remote Client
```

Test:

```text
[ ] Client receives DHCP
[ ] Client receives correct subnet
[ ] FGT-1 can ping 192.168.101.2
[ ] FGT-2 can ping 192.168.101.1
[ ] Client-to-client connectivity works
[ ] DHCP works across VXLAN
[ ] VLAN 101 is extended between sites
```

---

# 9. Security Problem — VXLAN Without IPsec

If VXLAN is running directly over the public/underlay network:

```text
FGT-1
  |
  | VXLAN
  | VNI 1000
  |
  +======================+
                         |
                      Internet
                         |
  +======================+
  |
  | VXLAN
  | VNI 1000
  |
FGT-2
```

The VXLAN transport itself does **not provide encryption**.

A packet capture/sniffer can expose the encapsulated traffic.

Therefore:

```text
VXLAN
  +
IPsec
  =
Encrypted VXLAN Transport
```

---

# 10. VXLAN over Dialup IPsec

## Target Topology

```text
                         INTERNET
                            |
                    +-------+-------+
                    |               |
                  FGT-1           FGT-2
                    |               |
              Dialup IPsec      IPsec Client
                    |               |
             12.12.12.1/30    12.12.12.2/30
                    |               |
                    +-------+-------+
                            |
                       VXLAN VNI 1000
                            |
              +-------------+-------------+
              |                           |
           VLAN 101                    VLAN 101
              |                           |
           LAN Side                    LAN Side
```

---

# 11. FGT-1 — Dialup IPsec

Create:

```text
VPN
└── IPsec Tunnels
    └── Custom
```

Phase 1:

```text
Remote Gateway:
    Dialup User

Incoming Interface:
    ISP / WAN

Pre-shared Key:
    123456

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

User Group:
    Same AD Group
```

---

# 12. FGT-1 — Phase 2

```text
Encryption:
    DES

Authentication:
    MD5

PFS:
    DH Group 5

Selectors:
    All required subnets

Auto-negotiate:
    Enable
```

---

# 13. FGT-1 — IPsec Firewall Policies

### Incoming

```text
Dialup IPsec
      +
VXLAN
      +
ISP Link
```

### Outgoing

```text
Dialup IPsec
      +
VXLAN
      +
ISP Link
```

Policy:

```text
Source:
    ALL

Destination:
    ALL

Service:
    ALL

NAT:
    DISABLE

Logging:
    ALL SESSIONS
```

---

# 14. FGT-1 — IPsec Interface

Assign:

```text
IP:
    12.12.12.1/30
```

Test:

```bash
ping 12.12.12.2
```

Expected:

```text
FGT-1
12.12.12.1/30
     |
     | IPsec
     |
FGT-2
12.12.12.2/30
```

---

# 15. FGT-1 — VXLAN over IPsec

Instead of using the public interface:

```text
VXLAN
    |
    v
IPsec Interface
```

Configure:

```bash
config system vxlan
    edit vx101
        set remote-ip 12.12.12.2
        set interface ipsec-dialup
        set vni 1000
    next
end
```

---

# 16. FGT-1 — VLAN on VXLAN

Create:

```text
Name:
    vx-vlan-101

Type:
    VLAN

Interface:
    vx101

VLAN ID:
    101
```

---

# 17. FGT-1 — Local VLAN

Create:

```text
Name:
    vlan-101

Type:
    VLAN

Interface:
    port3

VLAN ID:
    101
```

---

# 18. FGT-1 — Software Switch

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

Logical flow:

```text
LAN
 |
VLAN 101
 |
vlan-101
 |
Software Switch
 |
vx-vlan-101
 |
VXLAN
 |
IPsec
 |
Internet
```

---

# 19. FGT-2 — Dialup IPsec Client

Create:

```text
VPN
└── IPsec Tunnels
    └── Custom
```

Phase 1:

```text
Remote Gateway:
    Static IP 1.1.1.1

Incoming Interface:
    ISP / WAN

Device Creation:
    Enable

Pre-shared Key:
    123456

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
    u1

Password:
    1qaz@WSX
```

---

# 20. FGT-2 — Phase 2

```text
Encryption:
    DES

Authentication:
    MD5

PFS:
    DH Group 5

Selectors:
    All required subnets

Auto-negotiate:
    Enable
```

---

# 21. FGT-2 — IPsec Firewall Policies

### Incoming

```text
Dialup IPsec
      +
VXLAN
      +
ISP Link
```

### Outgoing

```text
Dialup IPsec
      +
VXLAN
      +
ISP Link
```

Policy:

```text
Source:
    ALL

Destination:
    ALL

Service:
    ALL

NAT:
    DISABLE

Logging:
    ALL SESSIONS
```

---

# 22. FGT-2 — IPsec Interface

Assign:

```text
IP:
    12.12.12.2/30
```

Test:

```bash
ping 12.12.12.1
```

---

# 23. FGT-2 — VXLAN over IPsec

```bash
config system vxlan
    edit vx101
        set remote-ip 12.12.12.1
        set interface ipsec-dialup
        set vni 1000
    next
end
```

---

# 24. FGT-2 — VLAN on VXLAN

```text
Name:
    vx-vlan-101

Type:
    VLAN

Interface:
    vx101

VLAN ID:
    101
```

---

# 25. FGT-2 — Local VLAN

```text
Name:
    vlan-101

Type:
    VLAN

Interface:
    port3

VLAN ID:
    101
```

---

# 26. FGT-2 — Software Switch

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

---

# 27. Final VXLAN over IPsec Flow

```text
                    FGT-1
              +---------------+
              | Software      |
              | Switch        |
              +-------+-------+
                      |
                   VLAN 101
                      |
                  VXLAN VNI
                    1000
                      |
              +-------v-------+
              | VXLAN         |
              +-------+-------+
                      |
                IPsec Interface
                12.12.12.1/30
                      |
                ENCRYPTED IPsec
                      |
             ====================
                    Internet
             ====================
                      |
                IPsec Interface
                12.12.12.2/30
                      |
              +-------v-------+
              | VXLAN         |
              +-------+-------+
                      |
                  VNI 1000
                      |
                   VLAN 101
                      |
              +-------v-------+
              | Software      |
              | Switch        |
              +---------------+
                    FGT-2
```

---

# 28. Encapsulation Concept

Without IPsec:

```text
Original Ethernet Frame
          |
          v
       VXLAN
          |
          v
     UDP / VXLAN
          |
          v
      Underlay
```

With IPsec:

```text
Original Ethernet Frame
          |
          v
       VXLAN
          |
          v
     IPsec ESP
          |
          v
      Underlay
```

Conceptually:

```text
Ethernet
   |
   +── VLAN 101
         |
         +── VXLAN
               |
               +── IPsec
                     |
                     +── Internet
```

---

# 29. Verify IPsec

Check IKE:

```bash
diagnose vpn ike gateway list
```

Check IPsec:

```bash
diagnose vpn tunnel list
```

Look for:

```text
IPsec SA
SPI
Encryption
Authentication
Source
Destination
Tunnel State
```

---

# 30. Verify VXLAN

Check:

```text
VXLAN interface:
    vx101

VNI:
    1000

Remote IP:
    12.12.12.2 / 12.12.12.1

Transport Interface:
    ipsec-dialup
```

Basic tests:

```bash
ping 12.12.12.2
ping 192.168.101.2
```

---

# 31. Verify DHCP

## FGT-1

```text
DHCP Server
    |
    v
192.168.101.0/24
```

## FGT-2

```text
DHCP Relay
    |
    v
192.168.101.1
```

Expected:

```text
Client
   |
   v
VLAN 101
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

---

# 32. Packet Capture Concept

## VXLAN Without IPsec

```text
Client Traffic
      |
      v
   VXLAN
      |
      v
Public / Underlay
      |
      v
Packet Capture
      |
      X
Traffic can be inspected
```

## VXLAN With IPsec

```text
Client Traffic
      |
      v
   VXLAN
      |
      v
    IPsec
      |
      v
    ESP
      |
      v
Public / Underlay
      |
      v
Packet Capture
      |
      X
Payload is encrypted
```

---

# 33. IPsec Encapsulation

With ESP:

```text
Outer IP Header
       |
       v
ESP Header
       |
       v
Encrypted VXLAN Packet
       |
       v
ESP Trailer
       |
       v
ESP Authentication
```

The important concept:

```text
VXLAN
  |
  v
Encrypted by IPsec
  |
  v
ESP
  |
  v
Internet
```

---

# 34. FortiGate IPsec VXLAN Encapsulation

```bash
config vpn ipsec phase1-interface
    edit dialup-vxlan
        set encapsulation vpn-id-ipip
    next
end
```

> **Note:** The exact encapsulation option and supported values depend on the FortiOS version and feature support. Verify the available CLI options on the target FortiGate before applying this configuration.

Concept:

```text
Phase 1
   |
   +── Encapsulation
          |
          +── GRE / VXLAN mode
```

---

# 35. Troubleshooting Flow

```text
                    START
                      |
                      v
             Can IPsec establish?
                      |
              +-------+-------+
              |               |
             NO              YES
              |               |
              v               v
        Check Phase 1     Can IPsec
        Check XAuth       interface ping?
        Check PSK              |
        Check proposals   +----+----+
        Check IKE             |     |
                             NO    YES
                              |     |
                              v     v
                         Check IPsec  Can VXLAN
                         interface    establish?
                                      |
                                +-----+-----+
                                |           |
                               NO          YES
                                |           |
                                v           v
                          Check VNI      Check VLAN
                          Remote IP      Software Switch
                          Interface      DHCP
                                        Connectivity
```

---

# 36. Troubleshooting Checklist

## IPsec

```text
[ ] IKEv1 matches
[ ] Aggressive Mode matches
[ ] PSK matches
[ ] Peer ID is correct
[ ] XAuth is correct
[ ] User credentials are correct
[ ] DES matches
[ ] MD5 matches
[ ] DH Group 5 matches
[ ] Phase 2 matches
[ ] PFS/DH Group 5 matches
[ ] Auto-negotiate enabled
[ ] IPsec interface exists
[ ] 12.12.12.1/30 reachable
[ ] 12.12.12.2/30 reachable
```

## VXLAN

```text
[ ] VNI is identical on both sides
[ ] VNI = 1000
[ ] Remote IP is correct
[ ] VXLAN uses IPsec interface
[ ] VXLAN interface is up
[ ] VLAN ID = 101
[ ] VLAN exists on VXLAN interface
[ ] VLAN exists on LAN interface
[ ] Software Switch has both VLAN members
```

## DHCP

```text
[ ] FGT-1 DHCP server enabled
[ ] FGT-2 DHCP relay enabled
[ ] Relay points to 192.168.101.1
[ ] Client receives IP
[ ] Client receives correct gateway
[ ] Client can ping gateway
```

## Firewall

```text
[ ] Incoming IPsec policy exists
[ ] Outgoing IPsec policy exists
[ ] VXLAN policy exists
[ ] Source = correct
[ ] Destination = correct
[ ] Service = ALL during testing
[ ] NAT disabled
[ ] Logging enabled
```

---

# 37. Important Design Summary

```text
                  VXLAN
                    |
                 VNI 1000
                    |
              +-----+-----+
              |           |
           VLAN 101    VLAN 101
              |           |
            LAN         LAN
              |           |
             FGT-1       FGT-2
                \         /
                 \       /
                  IPsec
                    |
                 ESP / NAT-T
                    |
                 Internet
```

### Layer Separation

| Layer | Technology | Purpose |
|---|---|---|
| L2 | VLAN 101 | Local LAN segmentation |
| L2 Overlay | VXLAN | Extend VLAN/L2 between sites |
| L3 Transport | IPsec Interface | Secure transport |
| Security | IPsec / ESP | Encryption + integrity |
| Authentication | XAuth | User authentication |
| Underlay | Internet | Physical/IP transport |

---

# 38. Final Concept

```text
VLAN
  ↓
Software Switch
  ↓
VXLAN VNI 1000
  ↓
IPsec Interface
  ↓
ESP Encryption
  ↓
Internet
  ↓
ESP Decryption
  ↓
IPsec Interface
  ↓
VXLAN VNI 1000
  ↓
Software Switch
  ↓
VLAN 101
  ↓
Remote Client
```

> **Key idea:** VXLAN provides the **Layer-2 overlay**, while IPsec provides the **secure encrypted transport** underneath it.

---

# Quick Reference

```text
VXLAN:
    Interface     : vx101
    VNI           : 1000
    VLAN           : 101

FGT-1:
    IPsec IP       : 12.12.12.1/30
    VXLAN Remote   : 12.12.12.2
    LAN IP         : 192.168.101.1/24

FGT-2:
    IPsec IP       : 12.12.12.2/30
    VXLAN Remote   : 12.12.12.1
    LAN IP         : 192.168.101.2/24

IPsec:
    IKE            : v1
    Mode           : Aggressive
    Encryption     : DES
    Hash           : MD5
    DH             : 5
    XAuth          : Enabled
    NAT             : Disabled
    Auto-negotiate : Enabled

VXLAN Transport:
    Underlay       : IPsec Interface
    Security       : ESP
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

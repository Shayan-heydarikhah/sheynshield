# FortiGate Interfaces

> **Scope:** Software/Hardware Switch, VLAN, FEC, IPAM, Captive Portal, MTU/MSS, Sniffer, LAG, Redundant Interface, VWP, PRP, EMAC, VXLAN

---

# 1. Interface Types

## Software Switch

* Processing is primarily handled by the **FortiGate CPU**.
* Provides Layer-2 switching functionality in software.
* Flexible and suitable for many general configurations.

```text
Client
  │
  ▼
Software Switch
  │
  ▼
CPU
  │
  ▼
FortiGate Processing
```

---

## Hardware Switch

* Virtual switch implemented at the **ASIC/hardware level**.
* Uses hardware switching resources such as ASICs/co-processors.
* Can provide optimized Layer-2 forwarding.
* Supports **Spanning Tree** with hardware-optimized processing on supported platforms.

```text
Port1 ─┐
Port2 ─┼──► Hardware Switch / ASIC ───► PortX
Port3 ─┘
```

### Quick Comparison

| Feature     | Software Switch      | Hardware Switch         |
| ----------- | -------------------- | ----------------------- |
| Processing  | CPU                  | ASIC / Hardware         |
| Performance | Lower                | Higher                  |
| Flexibility | High                 | Platform dependent      |
| STP         | Software processing  | Hardware optimized      |
| Best use    | General L2 scenarios | High-performance L2/STP |

---

# 2. Enhanced MAC VLAN — EMAC VLAN

**EMAC = Enhanced MAC**

Allows VLAN interfaces on the same physical interface to use **different MAC addresses**.

```text
                Physical Interface
                       │
              ┌────────┴────────┐
              │                 │
           VLAN 10           VLAN 20
              │                 │
          MAC-A             MAC-B
```

### Key Points

* Generates a new MAC address over the parent interface.
* EMAC interfaces are treated similarly to physical interfaces.
* EMAC interfaces can be advertised/synchronized in an HA cluster.
* Avoid using EMAC for:

  * HA heartbeat interfaces
  * Reserved management interfaces
  * Transparent VDOM scenarios

### Important

EMAC requires **NAT mode** for supported operation.

Be careful with DHCP when using EMAC interfaces.

---

## EMAC VLAN ID Rule

When an EMAC interface uses a VLAN ID:

> **VLAN ID + underlying physical interface must form a unique pair.**

This uniqueness applies even across different VDOMs because the underlying physical interface uses the VLAN ID to dispatch traffic.

---

## EMAC on VDOM/NPU Links

EMAC can also be used with:

* VLAN interfaces
* VDOM links
* NPU links

The relevant interface must be configured as an EMAC-type interface where supported.

---

# 3. Interface Administration Options

Under:

```text
System
 └── Network
      └── Interfaces
```

Common administrative access options include:

| Option          | Purpose                                 |
| --------------- | --------------------------------------- |
| HTTPS           | GUI management                          |
| SSH             | CLI management                          |
| PING            | ICMP testing                            |
| SNMP            | Monitoring                              |
| FMG-Access      | FortiManager communication              |
| FTM             | FortiToken Mobile / push authentication |
| Security Fabric | FortiTelemetry / Fabric communication   |
| CAPWAP          | Wireless/AP communication               |
| Speedtest       | Interface/network testing               |

### FMG-Access

Allows FortiManager-related authorization/communication between:

```text
FortiGate ◄────────► FortiManager
```

---

# 4. FEC — Forward Error Correction

Supported on certain high-speed interfaces/transceivers.

Example:

```bash
config system interface
    edit sfp10
        set speed 40000full
        set media-type cr4
        set forward-error-correction cl91-rs-fec
    end
end
```

### Common Speeds

| Speed | FEC Notes                         |
| ----- | --------------------------------- |
| 1G    | FEC not supported                 |
| 10G   | FEC not supported in this context |
| 25G   | CL91-RS-FEC commonly automatic    |
| 40G   | Depends on interface/transceiver  |
| 100G  | CL91-RS-FEC commonly automatic    |

> Always verify the exact FortiGate model, transceiver and FortiOS release before forcing FEC.

---

## Optical Module Naming

### SR4 — Short Range 4

```text
SR = Short Range
4  = 4 lanes
```

* Designed for short-distance MMF.
* Commonly around **100 m**, depending on optics/fiber.
* Uses multiple parallel lanes.
* Common in data-center environments.
* Used with 40G/100G applications.

---

### LR4 — Long Range 4

```text
LR = Long Range
4  = 4 lanes
```

* Designed for longer-distance SMF.
* Typical reach can be around **10 km**, depending on optic.
* Common for 40G/100G applications.

---

### CR4 — Copper 4

```text
CR = Copper
4  = 4 lanes
```

* Short-distance copper/Twinax connection.
* Commonly around **5 m**, depending on DAC.
* Common in data centers/rack-to-rack connections.
* Commonly associated with 40G applications.

---

# 5. IPAM

**IPAM = IP Address Management**

FortiGate can integrate with FortiIPAM for automatic subnet management.

### Enable IPAM

```bash
config system ipam
    set pool-subnet 172.16.0.0/16
    set status enable
end
```

### Enable Interface Management

```bash
config system interface
    set managed-subnetwork-size 512

    edit port8
        set ip-managed-by-fortiipam enable
    end
end
```

---

## IPAM Troubleshooting

### Find Largest Available Subnet

```bash
diagnose sys ipam largest-available-subnet
```

### Check Reservation Status

```bash
diagnose sys ipam reservation-status
```

### Confirm Reservation

```bash
diagnose sys ipam confirm-reserv
```

### Delete Device from IPAM

```bash
diagnose sys ipam delete-device-from-ipam
```

### Show Free Subnets

```bash
diagnose sys ipam dump-ipams-free-subnets
```

### Dump IPAM Entries

```bash
diagnose sys ipam dump-ipams-entreis
```

---

# 6. Captive Portal

Captive Portal is mainly useful for **user authentication at the network edge**.

```text
Client
  │
  ▼
FortiGate
  │
  ├── Authentication
  │
  ▼
Internet
```

### Key Points

* Uses HTTP/HTTPS-based authentication/redirection.
* Useful for local users.
* For larger environments, consider:

  * SSO
  * RADIUS
  * External authentication

Authenticated users can be monitored through:

```text
FortiView
   └── Sources

Firewall Users
```

---

# 7. External Captive Portal Authentication

Example topology:

```text
LAN
192.168.101.0/24
       │
       ▼
     FGT-1
       │
192.168.12.0/30
       │
       ▼
     FGT-2
       │
192.168.254.0/24
       │
       ▼
     Edge
       │
       ▼
    Internet
    8.8.8.8
```

### FGT-1

Configure the external captive portal to point toward:

```text
FGT-2 = 192.168.12.2
```

### FGT-2

On the interface facing FGT-1:

```text
192.168.12.0/30
```

Enable Captive Portal and specify the required user groups.

---

## ⚠️ External Captive Portal Important Points

### Exempt FGT-1

FGT-1 must be able to communicate with FGT-2 for authentication.

Therefore, avoid forcing FGT-1's own authentication traffic through the captive portal.

Example exemption:

```text
FGT-1 = 192.168.12.1
```

### NAT

Avoid unnecessary NAT between:

```text
LAN → FGT-1 → FGT-2
```

Improper NAT can bypass or interfere with captive-portal behavior.

---

# 8. MTU / TCP MSS

### Configure MTU

```bash
config system interface
    edit port8
        set mtu-override enable
        set mtu 1234
        set tcp-mss 1448
    end
end
```

### Concepts

| Setting      | Purpose                     |
| ------------ | --------------------------- |
| MTU          | Maximum Layer-3 packet size |
| TCP MSS      | Maximum TCP payload size    |
| MTU Override | Allows custom interface MTU |

Useful in scenarios involving:

* Tunnels
* VPN
* Encapsulation
* Fragmentation problems
* Path MTU issues

---

# 9. One-Arm Sniffer

A **one-arm sniffer** allows packet capture without placing the FortiGate inline in the traffic path.

### Switch SPAN Example

```bash
conf t

monitor session 1 source interface gig0/0 both

monitor session 1 destination interface gig0/2
```

Then:

```text
SPAN Port
   │
   ▼
FortiGate Sniffer
   │
   ▼
Wireshark
```

---

## FortiGate Sniffer

Configure the FortiGate interface as a one-arm sniffer and start capture from:

```text
Network
 └── Diagnostics
      └── Packet Capture
```

---

## Things That Can Prevent One-Arm Sniffer Activation

Examples include:

* WAN interface configuration
* Firewall policies
* VIP / DNAT dependencies
* Other interface dependencies
* Existing references to the interface

---

## One-Arm Sniffer Warning

One-arm sniffing can have more processing overhead than inline hardware-accelerated inspection because traffic may require additional processing.

Potential effects:

* Higher CPU utilization
* Packet loss
* Kernel buffer pressure

> Use packet capture for a **short troubleshooting window**, not as a permanent traffic-processing design.

---

## Log Commands

```bash
execute log filter category 19
execute log display
```

---

# 10. Physical Interfaces & Transceivers

FortiGate SFP/SFP+ interfaces can be sensitive to:

* Transceiver type
* Vendor compatibility
* Optical characteristics
* Speed
* Media type

### Check Transceiver

```bash
get system interface transceiver
```

---

# 11. VLAN

### 802.1Q

Standard VLAN tagging.

```text
Ethernet Frame
┌──────────────┬─────────┬──────────────┐
│ Ethernet     │ 802.1Q  │ Data         │
└──────────────┴─────────┴──────────────┘
```

* Single VLAN tag
* EtherType: `0x8100`
* 12-bit VLAN ID
* Common enterprise VLAN technology

---

# 12. 802.1ad / QinQ

802.1ad allows **VLAN stacking / double tagging**.

```text
┌────────┬────────┬────────┬────────────┐
│ Outer  │ Inner  │ VLAN   │ Payload    │
│ S-Tag  │ C-Tag  │ Data   │            │
└────────┴────────┴────────┴────────────┘
```

### EtherTypes

```text
Outer S-Tag = 0x88A8
Inner C-Tag = 0x8100
```

### Typical Use Cases

* Service Provider networks
* Metro Ethernet
* Customer VLAN aggregation
* VLAN stacking

---

## 802.1Q vs 802.1ad

| Feature         | 802.1Q            | 802.1ad              |
| --------------- | ----------------- | -------------------- |
| Tagging         | Single            | Double               |
| Outer EtherType | `0x8100`          | `0x88A8`             |
| Inner EtherType | —                 | `0x8100`             |
| Main purpose    | VLAN segmentation | VLAN stacking        |
| Common use      | Enterprise LAN    | ISP / Metro Ethernet |

---

# 13. Virtual VLAN Switch

Supported hardware switch ports can operate as a Layer-2 switch with VLAN assignment.

### Enable

```bash
config system global
    set virtual-switch-vlan enable
end
```

### Create Virtual Switch

```bash
config system virtual-switch
    edit vlan10
        set physical-switch sw0
        set vlan 10

        config port
            edit port10
            next
            edit port11
        end
    end
end
```

### Create Interface

```bash
config system interface
    edit vlan10
        set type hard-switch
        set ip 192.168.100.1/24
        set allow ping http ssh
    end
end
```

---

## When to Use Virtual VLAN Switch

Useful for specific Layer-2 designs where you need:

* Hardware switching
* VLAN-aware switching
* STP
* Multiple VLANs
* Better hardware-level processing

Especially useful when the network has complex:

* STP
* Multiple spanning-tree instances
* VLAN trunking

> Exact support is platform/FortiOS dependent. Verify the target FortiGate model before deployment.

---

# 14. Link Aggregation — LAG

### Switch Side

Example static EtherChannel:

```bash
conf t

interface range gig0/0-1
channel-group 1 mode on

interface range gig1/0-1
channel-group 2 mode on
```

---

## FortiGate

```bash
config system interface
    edit agg-1
        set lacp-mode static
        set lldp-transmission enable
    end
end
```

---

## MikroTik

Typical approach:

```text
Interfaces
   └── Bonding
        └── Mode: 802.3ad
```

Then assign the IP address to the bonding interface.

---

# 15. Enhanced LAG Hashing

FortiGate can use enhanced hashing to select LAG members.

Possible algorithms:

```text
XOR16  → lighter
XOR8
XOR4
CRC16  → more resource intensive
```

Hash inputs can include:

* IP protocol
* Source IP
* Destination IP
* Source port
* Destination port

### Example

```bash
config system npu
    set lag-out-pport-select enable

    config sw-eh-hash
        set computation xor16
        set ip-protocol include
        set source-ip include
        set destination-ip include
        set source-port include
        set destination-port include
        set netmask-length 32
    end
end
```

> Exact available hash fields depend on the FortiOS release/platform.

---

# 16. Redundant Interface

A redundant interface provides interface-level redundancy.

```text
        ┌── Port3 ──► Switch-A
FGT ────┤
        └── Port4 ──► Switch-B
```

### Important

> **Do NOT configure channel-group/LAG on redundant members.**

Common use:

* Full-mesh connectivity
* Dual physical paths
* Link redundancy

---

# 17. Fail Detection — Aggregate / Redundant

Example:

```bash
config system interface
    edit agg-1
        set fail-detect enable
        set fail-alert-method link-down
        set fail-alert-interface port4
    end
end
```

### Meaning

```text
agg-1 failure
     │
     ▼
Failure Detection
     │
     ▼
fail-alert-method
     │
     ▼
Specified interface action
```

---

# 18. 802.1X

802.1X can be supported on specific hardware-switch platforms.

Example:

```bash
config system interface
    edit port8
        set security-mode 802.1x
        set security-group test
    end
end
```

### Troubleshooting

```bash
diagnose sys 802-1x status
```

> 802.1X support is hardware/platform dependent. Verify the FortiGate model and FortiOS version.

---

# 19. Virtual Wire Pair — VWP

Virtual Wire Pair connects two interfaces at **Layer 2** and allows security inspection between them.

```text
Client
  │
  ▼
[ Port3 ]
   │
   │ VWP
   │
[ Port4 ]
  │
  ▼
Server
```

### Main Characteristics

* Layer-2 operation
* No traditional routed interface required
* Security inspection between two interfaces
* Useful for internal segmentation
* Can be used with **Direct Server Return (DSR)** designs
* Can reduce routing complexity in certain designs

---

## Configure VWP

```bash
config system virtual-wire-pair
    edit vwp-1
        set member port3 port4
        set wildcard-vlan disable
    end
end
```

> VWP member interfaces should be unused and should not have conflicting references.

---

## VWP Firewall Policy

Navigate to:

```text
Policy & Objects
 └── Firewall Virtual Wire Pair Policy
```

Traffic direction can be controlled through VWP policies.

Inspection can use:

* Flow-based inspection
* Proxy-based inspection

---

## NAT with VWP

When NAT is required:

```text
VWP
 │
 └── Firewall Policy
       │
       └── IP Pool / SNAT
```

Use an appropriate overload/IP-pool NAT design.

---

# 20. PRP — Parallel Redundancy Protocol

**PRP = Parallel Redundancy Protocol**

Standardized under:

```text
IEC 62439-3
```

Designed for high-availability Ethernet networks.

Common in:

* Industrial networks
* Substation automation
* Critical infrastructure
* Industrial control systems
* High-power systems

---

## PRP Architecture

```text
                 ┌──────── LAN A ────────┐
                 │                       │
Node A ──────────┤                       ├──────── Node B
                 │                       │
                 └──────── LAN B ────────┘
```

The source sends duplicate frames over both networks.

```text
Frame
 ├──► LAN A
 └──► LAN B

Destination:
First frame  → ACCEPT
Duplicate    → DROP
```

### Benefits

* Zero/near-zero recovery time
* No traditional failover convergence
* Layer-2 operation
* Application transparent
* Independent LAN paths

### Trade-off

Requires duplicated network infrastructure.

---

## PRP Configuration

```bash
config system setting
    set prp-trailer-action enable
end
```

### NPU Ports

```bash
config system npu
    set prp-port-in port15
    set prp-port-out port16
end
```

### Virtual Wire Pair

```bash
config system virtual-wire-pair
    edit vwp-1
        set member port15 port16
    end
end
```

---

# 21. VXLAN

**VXLAN = Virtual Extensible LAN**

Provides Layer-2 overlay connectivity over an IP network.

```text
Site A                                Site B

Client                                Client
  │                                     │
VLAN 10                               VLAN 10
  │                                     │
FGT ─────── IP Underlay ───────────── FGT
              VXLAN
             VNI 1000
```

---

## VXLAN Configuration

```bash
config system vxlan
    edit vx10
        set interface port2
        set remote-ip 22.22.22.2
        set vni 1000
    end
end
```

### Important Parameters

| Parameter   | Meaning                  |
| ----------- | ------------------------ |
| `interface` | Underlay interface       |
| `remote-ip` | VXLAN peer               |
| `vni`       | VXLAN Network Identifier |

---

# 22. VLAN over VXLAN

Create a VLAN interface **on top of the VXLAN interface**.

```text
Physical Interface
       │
       ▼
    Underlay
       │
       ▼
     VXLAN
       │
       ▼
    VLAN 10
```

Example concept:

```text
VLAN 10
   │
 VXLAN
   │
 IP Underlay
```

---

## VXLAN + Software Switch

Example:

```text
Client Port
    │
 VLAN 10
    │
Software Switch
    │
 VLAN 10
    │
VXLAN
```

Add:

* Client-side VLAN interface
* VXLAN-side VLAN interface

> Do **not** add the main VXLAN interface itself to the software switch when the design requires the VLAN interfaces.

---

# 23. Software Switch Intra-Switch Traffic

Depending on configuration, traffic inside a software switch can be:

### Implicit

```text
Port/VLAN
   │
Software Switch
   │
Forward
```

Traffic can be forwarded by default without a separate firewall policy.

### Explicit

A dedicated firewall policy is required for traffic between switch members.

```text
VLAN-A
  │
  ▼
Software Switch
  │
Firewall Policy
  │
  ▼
VLAN-B
```

---

# 24. VXLAN + VWP

The VXLAN main interface can also participate in a VWP design where supported.

```text
Client
  │
Physical Interface
  │
  ├──── VWP ──── VXLAN
  │
  ▼
Remote Network
```

Policies then control traffic between the VWP members.

---

# 25. VXLAN + DHCP Design

If DHCP is required around the VXLAN environment, a practical design is to use:

```text
Client Interface
       │
       ▼
Software Switch
       │
       ├── Client-side VLAN
       │
       └── VXLAN-side VLAN
       │
       ▼
     Gateway
       │
      DHCP
```

The IP address and DHCP server can be associated with the appropriate software-switch interface.

---

# 26. Interface Troubleshooting Commands

## Switch Interfaces

```bash
show system switch-interface
```

Shows software-switch information.

---

## Transceiver

```bash
get system interface transceiver
```

Check SFP/SFP+ and transceiver information.

---

## 802.1X

```bash
diagnose sys 802-1x status
```

---

# 27. Design Decision Matrix

| Requirement                          | Recommended Interface/Feature             |
| ------------------------------------ | ----------------------------------------- |
| CPU-based L2 switching               | Software Switch                           |
| ASIC-based L2 switching              | Hardware Switch                           |
| Multiple VLANs over physical port    | VLAN                                      |
| Double VLAN tagging                  | QinQ / 802.1ad                            |
| Different MAC per VLAN               | EMAC VLAN                                 |
| IP address allocation                | IPAM                                      |
| User web authentication              | Captive Portal                            |
| Packet capture                       | One-Arm Sniffer                           |
| Custom packet size                   | MTU Override                              |
| TCP fragmentation control            | TCP MSS                                   |
| Multiple physical links              | LAG                                       |
| Physical path redundancy             | Redundant Interface                       |
| Layer-2 security bridge              | VWP                                       |
| Industrial zero-time redundancy      | PRP                                       |
| L2 overlay across IP                 | VXLAN                                     |
| VLAN over VXLAN                      | VLAN interface on VXLAN                   |
| High-speed optical links             | SFP/SFP+/SFP28/QSFP depending on platform |
| Error correction on high-speed links | FEC                                       |

---

# 🧠 High-Value Exam Notes

```text
Software Switch
    ↓
CPU processing

Hardware Switch
    ↓
ASIC processing

EMAC VLAN
    ↓
Different MAC per VLAN on same parent interface

802.1Q
    ↓
Single VLAN tag
    ↓
0x8100

802.1ad / QinQ
    ↓
Double VLAN tag
    ↓
Outer 0x88A8
    ↓
Inner 0x8100

LAG
    ↓
Multiple links
    ↓
Load balancing + redundancy

Redundant Interface
    ↓
Multiple physical paths
    ↓
NO channel-group

VWP
    ↓
Layer-2 security bridge

PRP
    ↓
Duplicate traffic over LAN-A + LAN-B

VXLAN
    ↓
L2 overlay over IP

IPAM
    ↓
Automatic subnet/address management

One-Arm Sniffer
    ↓
Passive/monitoring capture
    ↓
Short troubleshooting windows
```

---

# ⚠️ Common Design Mistakes

### ❌ Don't confuse LAG and Redundant

```text
LAG
    → bandwidth + redundancy

Redundant
    → path redundancy
    → no channel-group
```

### ❌ Don't treat EMAC as a normal VLAN

```text
Normal VLAN
    → usually inherits/uses parent MAC behavior

EMAC VLAN
    → creates a distinct MAC identity
```

### ❌ Don't forget platform dependencies

Many features depend on:

* FortiGate model
* ASIC generation
* FortiOS version
* Interface type
* Transceiver
* NPU availability

### ❌ Don't use One-Arm Sniffer permanently

```text
Capture
  ↓
CPU / buffer pressure
  ↓
Possible packet loss
```

Use it for focused troubleshooting.

### ❌ Don't add the VXLAN parent unnecessarily

When building VLAN-over-VXLAN:

```text
VXLAN
  └── VLAN 10
```

Usually work with the VLAN interface for the software-switch design rather than blindly adding the VXLAN parent.

---

# 🔥 Fast Revision

| Topic           | Remember                        |
| --------------- | ------------------------------- |
| Software Switch | CPU                             |
| Hardware Switch | ASIC                            |
| EMAC            | Different MAC per VLAN          |
| FEC             | High-speed physical links       |
| SR4             | Short-range, 4 lanes            |
| LR4             | Long-range, 4 lanes             |
| CR4             | Copper, 4 lanes                 |
| IPAM            | IP Address Management           |
| Captive Portal  | User authentication             |
| MTU             | Maximum packet size             |
| MSS             | TCP payload size                |
| SPAN            | Switch-side packet mirroring    |
| LAG             | Aggregate multiple links        |
| XOR16           | Lightweight LAG hashing         |
| Redundant       | Path redundancy, no LAG         |
| 802.1X          | Port-based access control       |
| VWP             | L2 security bridge              |
| PRP             | Duplicate packets over two LANs |
| VXLAN           | L2 over IP                      |
| VNI             | VXLAN Network Identifier        |
| QinQ            | Double VLAN tagging             |
| 802.1Q          | Single VLAN tagging             |
| 802.1ad         | VLAN stacking                   |

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

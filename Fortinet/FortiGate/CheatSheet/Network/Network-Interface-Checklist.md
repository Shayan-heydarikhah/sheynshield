# 🔗 SheynShield Resources  
# FortiGate Interfaces — Expert Checklist

> **Purpose:** FortiGate Interface Design, Deployment Validation, Troubleshooting & NSE4/NSE7 Revision Checklist  
>
> **Scope:** Software Switch, Hardware Switch, VLAN, EMAC VLAN, FEC, IPAM, Captive Portal, MTU/MSS, Sniffer, LAG, Redundant Interface, VWP, PRP, VXLAN, QinQ

---

# ✅ 1. Interface Type Selection Checklist

## Software Switch

### Design Validation

- [ ] Confirm CPU-based Layer-2 forwarding is acceptable.
- [ ] Verify expected traffic volume will not overload CPU.
- [ ] Confirm software switch members are correctly assigned.
- [ ] Validate firewall policy requirements between members.
- [ ] Check whether intra-switch traffic requires inspection.

### Architecture

```text
Interface
    |
    ▼
Software Switch
    |
    ▼
FortiGate CPU
    |
    ▼
Security Processing
````

---

## Hardware Switch

### Design Validation

* [ ] Confirm FortiGate model supports hardware switch.
* [ ] Verify ASIC switching capability.
* [ ] Validate STP requirements.
* [ ] Confirm hardware switch port assignment.
* [ ] Check platform limitations.

### Architecture

```text
Physical Ports
      |
      ▼
ASIC Hardware Switch
      |
      ▼
Layer-2 Forwarding
```

---

## Software Switch vs Hardware Switch

| Check Point | Software Switch | Hardware Switch     |
| ----------- | --------------- | ------------------- |
| Processing  | CPU             | ASIC                |
| Performance | Lower           | Higher              |
| Flexibility | High            | Platform dependent  |
| STP         | Software        | Hardware optimized  |
| Use Case    | General L2      | High-performance L2 |

---

# ✅ 2. EMAC VLAN Checklist

## Enhanced MAC VLAN Validation

* [ ] Confirm requirement for unique MAC per VLAN.
* [ ] Verify parent interface and VLAN ID combination uniqueness.
* [ ] Check EMAC usage across VDOMs.
* [ ] Confirm NAT mode requirement.
* [ ] Validate DHCP behavior.

---

## EMAC Design Rules

Remember:

```text
Same VLAN ID
+
Same Physical Interface
=
Must be Unique
```

---

## Avoid EMAC Usage For:

* [ ] HA heartbeat interfaces
* [ ] Reserved management interfaces
* [ ] Transparent VDOM scenarios

---

# ✅ 3. Interface Administrative Access Checklist

Path:

```text
System
 └── Network
      └── Interfaces
```

Validate:

* [ ] HTTPS enabled if GUI management required.
* [ ] SSH enabled for CLI access.
* [ ] Ping enabled for troubleshooting.
* [ ] SNMP enabled for monitoring.
* [ ] FMG-Access enabled for FortiManager integration.
* [ ] Security Fabric access configured if required.
* [ ] CAPWAP configured for wireless integration.

---

# ✅ 4. FEC (Forward Error Correction) Checklist

## Before Configuration

* [ ] Verify FortiGate model support.
* [ ] Verify transceiver capability.
* [ ] Confirm optic type.
* [ ] Confirm interface speed.
* [ ] Validate FortiOS support.

---

## Common Optical Types

| Optic | Meaning             | Typical Usage     |
| ----- | ------------------- | ----------------- |
| SR4   | Short Range 4 lanes | Data Center MMF   |
| LR4   | Long Range 4 lanes  | Long distance SMF |
| CR4   | Copper 4 lanes      | DAC connections   |

---

## Validation Command

```bash
get system interface transceiver
```

---

# ✅ 5. IPAM Checklist

## Configuration

* [ ] Enable FortiIPAM integration.
* [ ] Configure subnet pool.
* [ ] Enable interface IP management.
* [ ] Validate subnet allocation size.

Example:

```bash
config system ipam
    set pool-subnet 172.16.0.0/16
    set status enable
end
```

---

## Troubleshooting Checklist

Check:

```bash
diagnose sys ipam largest-available-subnet
```

```bash
diagnose sys ipam reservation-status
```

```bash
diagnose sys ipam dump-ipams-free-subnets
```

```bash
diagnose sys ipam dump-ipams-entreis
```

---

# ✅ 6. Captive Portal Checklist

## Design Validation

* [ ] Identify authentication requirement.
* [ ] Select authentication source.
* [ ] Configure user groups.
* [ ] Validate firewall policy flow.
* [ ] Verify user visibility in FortiView.

Supported methods:

* [ ] Local users
* [ ] RADIUS
* [ ] SSO
* [ ] External authentication

---

# ✅ 7. External Captive Portal Checklist

Topology Validation:

```text
Client
 |
FGT-1
 |
FGT-2
 |
Internet
```

---

## Required Checks

* [ ] Configure external captive portal URL.
* [ ] Verify FGT communication path.
* [ ] Exempt authentication traffic.
* [ ] Avoid unnecessary NAT.
* [ ] Validate return traffic.

---

# ✅ 8. MTU / MSS Checklist

## MTU Validation

Check:

* [ ] VPN encapsulation overhead.
* [ ] Tunnel requirements.
* [ ] Fragmentation issues.
* [ ] Path MTU problems.

Example:

```bash
config system interface
 edit port8
    set mtu-override enable
    set mtu 1234
    set tcp-mss 1448
 end
end
```

---

## Remember

| Parameter | Purpose             |
| --------- | ------------------- |
| MTU       | Maximum packet size |
| MSS       | TCP payload size    |

---

# ✅ 9. One-Arm Sniffer Checklist

## Deployment

* [ ] Configure switch SPAN port.
* [ ] Connect mirrored traffic to FortiGate.
* [ ] Start packet capture.
* [ ] Stop capture after troubleshooting.

---

## Avoid Permanent Usage

Check:

* [ ] CPU utilization
* [ ] Buffer pressure
* [ ] Packet loss possibility

---

## Remember

```text
SPAN
 |
 ▼
FortiGate Sniffer
 |
 ▼
Wireshark
```

---

# ✅ 10. VLAN Checklist

## 802.1Q

Validate:

* [ ] VLAN ID configuration.
* [ ] Trunk configuration.
* [ ] Native VLAN behavior.
* [ ] Allowed VLAN list.

Remember:

```text
802.1Q
=
Single VLAN Tag

EtherType:
0x8100
```

---

# ✅ 11. QinQ / 802.1ad Checklist

Validate:

* [ ] Double tagging requirement.
* [ ] Provider network compatibility.
* [ ] Customer VLAN mapping.

Remember:

```text
Outer Tag
0x88A8

Inner Tag
0x8100
```

---

# ✅ 12. Virtual VLAN Switch Checklist

Before Deployment:

* [ ] Verify hardware support.
* [ ] Enable virtual switch VLAN.
* [ ] Configure physical switch mapping.
* [ ] Assign VLAN interfaces.
* [ ] Validate STP requirements.

---

# ✅ 13. LAG Checklist

## Design Validation

* [ ] Confirm multiple physical links.
* [ ] Confirm switch-side configuration.
* [ ] Select LACP/static mode.
* [ ] Validate hashing algorithm.
* [ ] Check member link status.

---

Example:

```bash
config system interface
 edit agg-1
    set lacp-mode static
 end
end
```

---

# ✅ 14. Enhanced LAG Hashing Checklist

Validate:

* [ ] Hash algorithm selection.
* [ ] Source IP inclusion.
* [ ] Destination IP inclusion.
* [ ] Port-based hashing.
* [ ] Platform capability.

Possible algorithms:

```
XOR4
XOR8
XOR16
CRC16
```

---

# ✅ 15. Redundant Interface Checklist

## Important Rules

* [ ] Use for path redundancy.
* [ ] Verify independent physical paths.
* [ ] Do NOT configure channel-group.
* [ ] Validate failover behavior.

---

Remember:

```text
LAG
=
Bandwidth + Redundancy


Redundant Interface
=
Path Redundancy
```

---

# ✅ 16. 802.1X Checklist

Validate:

* [ ] Hardware support.
* [ ] Authentication server.
* [ ] Security group.
* [ ] Port configuration.

Troubleshooting:

```bash
diagnose sys 802-1x status
```

---

# ✅ 17. Virtual Wire Pair Checklist

## Design

* [ ] Confirm Layer-2 security requirement.
* [ ] Select two unused interfaces.
* [ ] Create VWP.
* [ ] Create VWP firewall policy.
* [ ] Enable inspection profile.

---

Architecture:

```text
Port A
 |
VWP
 |
Port B
```

---

# ✅ 18. PRP Checklist

## Industrial HA Validation

* [ ] Confirm IEC 62439-3 requirement.
* [ ] Validate dual LAN design.
* [ ] Configure duplicate paths.
* [ ] Validate zero recovery requirement.

---

Architecture:

```text
Node

 |
 +---- LAN A
 |
 +---- LAN B
```

---

# ✅ 19. VXLAN Checklist

## Design Validation

* [ ] Verify IP underlay connectivity.
* [ ] Configure VNI.
* [ ] Validate remote VTEP.
* [ ] Confirm VLAN mapping.
* [ ] Check MTU requirements.

---

Configuration:

```bash
config system vxlan
 edit vx10
    set interface port2
    set remote-ip 22.22.22.2
    set vni 1000
 end
end
```

---

# ✅ 20. VLAN Over VXLAN Checklist

Validate:

* [ ] Create VXLAN interface.
* [ ] Create VLAN interface over VXLAN.
* [ ] Map VLAN correctly.
* [ ] Validate software switch design.

Correct model:

```text
Physical Interface

      |
      ▼

VXLAN

      |
      ▼

VLAN Interface
```

---

# ✅ 21. VXLAN + Software Switch Checklist

Before Adding:

* [ ] Add VLAN interfaces.
* [ ] Validate DHCP design.
* [ ] Confirm forwarding behavior.
* [ ] Avoid unnecessary VXLAN parent addition.

---

# ✅ 22. Troubleshooting Checklist

## Interface

```bash
show system interface
```

---

## Switch

```bash
show system switch-interface
```

---

## Transceiver

```bash
get system interface transceiver
```

---

## Packet Capture

```text
Network
 └── Diagnostics
      └── Packet Capture
```

---

# 🔥 NSE4/NSE7 Exam Memory Checklist

```text
Software Switch
↓
CPU


Hardware Switch
↓
ASIC


EMAC
↓
Different MAC per VLAN


802.1Q
↓
Single Tag


802.1ad
↓
Double Tag


LAG
↓
Multiple Links


Redundant Interface
↓
No Channel Group


VWP
↓
Layer-2 Security Bridge


PRP
↓
Duplicate Frames


VXLAN
↓
Layer-2 Overlay


IPAM
↓
Automatic Address Management
```

---

# ⚠️ Common Mistakes Checklist

## Interface Design

* [ ] Do not use EMAC without understanding MAC behavior.
* [ ] Do not confuse LAG with redundancy.
* [ ] Do not ignore ASIC limitations.
* [ ] Do not ignore transceiver compatibility.
* [ ] Do not permanently run one-arm sniffing.
* [ ] Do not forget MTU overhead in VXLAN/VPN.
* [ ] Do not configure LAG on redundant members.

---

# 🚀 Deployment Final Validation

Before Production:

* [ ] FortiGate model verified.
* [ ] FortiOS version verified.
* [ ] Interface capability checked.
* [ ] ASIC/NPU dependency checked.
* [ ] Security policy reviewed.
* [ ] HA impact reviewed.
* [ ] Monitoring configured.
* [ ] Backup configuration exported.

---

## 🔗 SheynShield Resources

### 🎥 Video Learning

* YouTube: [https://youtube.com/@sheynshield](https://youtube.com/@sheynshield)

### 📚 Technical Notes

* Telegram: [https://t.me/sheynshield](https://t.me/sheynshield)

### 🐙 Knowledge Base

* GitHub: [https://github.com/Shayan-heydarikhah/sheynshield](https://github.com/Shayan-heydarikhah/sheynshield)

### 💼 Professional Network

* LinkedIn: [https://linkedin.com/in/shayan-heydarikhah](https://linkedin.com/in/shayan-heydarikhah)

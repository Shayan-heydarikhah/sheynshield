```markdown

# FortiGate Architecture & Interface Management Cheat Sheet

This comprehensive reference guide consolidates enterprise FortiGate Layer 2/Layer 3 interface configurations, ASIC hardware optimizations, packet acceleration techniques, and network encapsulation standards.

---

## 📌 Table of Contents
- [Interface Types & Management Options](#-interface-types--management-options)
- [Transceivers & Forward Error Correction (FEC)](#-transceivers--forward-error-correction-fec)
- [IP Address Management (IPAM)](#-ip-address-management-ipam)
- [External Captive Portal Architecture](#-external-captive-portal-architecture)
- [Interface Parameters (MTU & 802.1X)](#-interface-parameters-mtu--8021x)
- [Traffic Sniffing (One-Arm SPAN)](#-traffic-sniffing-one-arm-span)
- [VLANs & Hardware Virtual Switches](#-vlans--hardware-virtual-switches)
- [Link Aggregation (LAG) & Redundancy](#-link-aggregation-lag--redundancy)
- [Virtual Wire Pair (VWP) & PRP](#-virtual-wire-pair-vwp--prp)
- [Enhanced MAC (EMAC) VLAN](#-enhanced-mac-emac-vlan)
- [VXLAN Architecture & Encapsulation](#-vxlan-architecture--encapsulation)

---

## 🛠 Interface Types & Management Options

### Interface Architecture
* **Software Switches:** Traffic is switched in software and processed via the main system CPU.
* **Hardware Switches:** Virtual switches operating at the hardware layer utilizing ASICs and co-processors. Supports optimized Spanning Tree Protocol (STP) processing.
* **Physical Interfaces:** SFP/SFP+ ports are hardware-sensitive and require verified vendor modules. Verify diagnostics via:
  ```bash
  get system interface transceiver

```

* **Migration Limits:** The built-in FortiOS interface migration tool operates exclusively on physical interfaces.

### Administrative Access Protocols

| Access Flag | Purpose & Functionality |
| --- | --- |
| `fmg-access` | Automatically grants FortiManager authorization during device communication exchanges. |
| `ftm` | Enables FortiToken Mobile push authentication handling. |
| `security-fabric` | Enables FortiTelemetry and CAPWAP tunnel management across fabric nodes. |
| `speedtest` | Permits on-demand bandwidth testing requests from FortiManager/CLI. |

---

## ⚡ Transceivers & Forward Error Correction (FEC)

Forward Error Correction (FEC) maintains payload integrity across high-speed data paths (10G, 25G, 40G, 100G).

* **1G / 10G / 40G:** FEC is disabled or unsupported natively on these transceiver speeds.
* **25G / 100G:** Automatically negotiates and applies the `cl91-rs-fec` profile.

### Coding Terminology

* **SR (Short Range):** Reduces information bits to shorten total code length.
* **LR (Long Range):** Lifts the parity-check matrix to expand code length over distance.
* **CR (Copper Range):** Represents code efficiency (ratio of payload information bits to total transmitted bits).

### Transceiver Specification Matrix

| Type | Full Name | Reach / Cable Type | Architectural Design & Use Cases |
| --- | --- | --- | --- |
| **SR4** | Short Range 4-Lane | $\le$ 100m / Multimode Fiber (MMF) | Employs 4 parallel Tx and 4 parallel Rx lanes. Ideal for high-density data center access (40G/100G). |
| **LR4** | Long Range 4-Lane | $\le$ 10km+ / Singlemode Fiber (SMF) | Uses 4 optical lanes optimized for long-haul enterprise backbones (40G/100G). |
| **CR4** | Copper 4-Lane | $\le$ 5m / Twinaxial Copper | Employs 4 twinax copper channels for intra-rack or top-of-rack interconnects (40G). |

### Interface Speed & FEC Configuration

```bash
config system interface
    edit "sfp10"
        set speed 40000full
        set media-type cr4
        set forward-error-correction cl91-rs-fec
    next
end

```

---

## 🏷️ IP Address Management (IPAM)

Automates subnet distribution and centralized tracking for internal interface networks.

```bash
# Global IPAM Pool Setup
config system ipam
    set status enable
    set pool-subnet 172.16.0.0/16
end

# Interface-level IPAM Allocation
config system interface
    edit "port8"
        set ip-managed-by-fortiipam enable
        set managed-subnetwork-size 512
    next
end

```

### IPAM Diagnostics & Management Commands

```bash
# Display the largest available contiguous block
diagnose sys ipam largest-available-subnet

# Display current IP allocation status and entries
diagnose sys ipam reservation-status
diagnose sys ipam dump-ipams-entreis
diagnose sys ipam dump-ipams-free-subnets

# Administrative Operations
diagnose sys ipam confirm-reserv
diagnose sys ipam delete-device-from-ipam

```

---

## 🔐 External Captive Portal Architecture

Captive portals use HTTP redirection for local user authentication. Enterprise environments should prefer 802.1X, SSO, or RADIUS where feasible.

```
[ LAN Clients ] ---> ( LAN: 192.168.101.0/24 )
                          │
                     [ FGT-1 ] ( Forwarder )
                          │ ( Transit: 192.168.12.0/30 )
                          ▼
                     [ FGT-2 ] ( Authentication Server: 192.168.12.2 )
                          │ ( Edge: 192.168.254.0/24 )
                          ▼
                     [ Internet / 8.8.8.8 ]

```

### Deployment Guidelines & Constraints

1. **FGT-1 Configuration:** Configure the ingress LAN interface to point its external captive portal authentication URI to FGT-2 (`192.168.12.2`). Ensure FGT-1's default route points to FGT-2.
2. **FGT-2 Configuration:** Enable captive portal enforcement on the transit interface (`192.168.12.0/30`) and map the required user authorization groups.
3. **Critical Dependencies:**
* **Exempt Forwarder Address:** The upstream interface address of FGT-1 (`192.168.12.1`) **must** be exempted from captive portal enforcement on FGT-2. Blocking this address halts all client authentication flow.
* **Disable NAT on Transit:** Do **not** apply Source NAT on FGT-1 for LAN traffic traversing the `192.168.12.0/30` transit link. NATing obfuscates source IPs, causing authentication bypasses or session mapping failures.



---

## ⚙️ Interface Parameters (MTU & 802.1X)

### Custom MTU and TCP MSS Override

Adjust system MTU and TCP Maximum Segment Size (MSS) to prevent fragmentation over encapsulated links:

```bash
config system interface
    edit "port8"
        set mtu-override enable
        set mtu 1234
        set tcp-mss 1448
    next
end

```

### Port-Based 802.1X Authentication (Hardware Switch)

Hardware-enforced 802.1X is supported on select NP6-accelerated hardware switch platforms (e.g., FortiGate 30xE, 40xE, 110xE).

```bash
config system interface
    edit "port8"
        set security-mode 802.1x
        set security-group "Corporate-802.1X-Group"
    next
end

```

*Verification:*

```bash
diagnose sys 802-1x status

```

---

## 🔍 Traffic Sniffing (One-Arm SPAN)

Used for passive packet monitoring by mirror port ingest.

```
[ Switch Port Gig0/0 ] ──( Monitored Traffic )──► [ Switch Port Gig0/2 ]
                                                          │
                                                    ( SPAN Feed )
                                                          │
                                                          ▼
                                             [ FortiGate One-Arm Interface ]

```

### Switch Mirroring Configuration (Cisco IOS Example)

```text
monitor session 1 source interface gig0/0 both
monitor session 1 destination interface gig0/2

```

### Operational Considerations & Limitations

* **Hardware Offload Bypass:** One-Arm sniffing completely disables ASIC offload (NP/nTurbo/CP engines) for processed packets, shifting handling to CPU software path.
* **Performance Impact:** High packet volumes can cause CPU spikes, packet dropping, and kernel buffer exhaustion. Limit usage to targeted troubleshooting.
* **Configuration Incompatibilities:** One-arm sniffing cannot be enabled on interfaces bound to:
* WAN roles
* Active Firewall Policies
* Virtual IPs (Destination NAT)
* Active Interface Dependencies



### Logging Diagnostic Filters

```bash
execute log filter category 19
execute log display

```

---

## 🔀 VLANs & Hardware Virtual Switches

VLAN processing delivers optimal throughput under NAT operational mode.

### 802.1Q vs. 802.1ad (QinQ) Standard Comparison

| Feature | IEEE 802.1Q (Standard VLAN) | IEEE 802.1ad (QinQ / VLAN Stacking) |
| --- | --- | --- |
| **Tagging Structure** | Single 802.1Q Tag | Double Tagged (Outer S-Tag + Inner C-Tag) |
| **EtherType Field** | `0x8100` | Outer: `0x88A8`, Inner: `0x8100` |
| **VLAN ID Space** | Max 4096 unique IDs | Nested tagging (up to $4096 \times 4096$ theoretical isolation) |
| **Primary Use Cases** | Enterprise VLAN segmentation | Service Provider Metro Ethernet, Carrier Bridges |
| **Class of Service** | Uses Canonical Format Indicator (CFI) | Replaces CFI with Drop Eligibility Indicator (DEI) |

### Virtual VLAN Switch Configuration (FortiOS 7.2)

*Supported Hardware:* FortiGate 60F, 80F, 100E, 100F, 140E, 200F, 300E, 400E, 1100E.

```bash
# Enable Virtual Switch Global Mode
config system global
    set virtual-switch-vlan enable
end

# Define Hardware Virtual Switch Entry
config system virtual-switch
    edit "vlan10"
        set physical-switch "sw0"
        set vlan 10
        config port
            edit "port10"
            next
            edit "port11"
            next
        end
    next
end

# Provision Logical Hard-Switch Interface
config system interface
    edit "vlan10"
        set type hard-switch
        set ip 192.168.100.1 255.255.255.0
        set allowaccess ping http ssh
    next
end

```

---

## 🔗 Link Aggregation (LAG) & Redundancy

### Interface Aggregation vs. Redundant Interfaces

* **Aggregated (LACP):** Combines throughput across physical links using active load balancing.
* **Redundant Interfaces:** Provides active-backup link failover without throughput aggregation. *Never use switch channel-groups for redundant interface configurations.*

### LACP Configuration Example

**Cisco Switch Side:**

```text
interface range gig0/0 - 1
 channel-group 1 mode on
!
interface range gig1/0 - 1
 channel-group 2 mode on

```

**FortiGate Side:**

```bash
config system interface
    edit "agg-1"
        set type aggregate
        set member "port1" "port2"
        set lacp-mode static
        set lldp-transmission enable
    next
end

```

### Enhanced Hardware LAG Hashing Algorithms

Enhances load-balancing entropy by tuning hash calculation methods across NPU links:

* **Computation Engines:** `xor16` (Low CPU overhead; data center default), `xor8`, `xor4`, `crc16` (High distribution precision; resource intensive).

```bash
config system npu
    set lag-out-port-select enable
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

### Link Failure Detection Configuration

Triggers immediate failure isolation actions when member interfaces drop:

```bash
config system interface
    edit "agg-1"
        set fail-detect enable
        set fail-alert-method link-down
        set fail-alert-interface "port4"
    next
end

```

---

## 🌉 Virtual Wire Pair (VWP) & PRP

Virtual Wire Pairs bind two interfaces at Layer 2, allowing transparent pass-through filtering without requiring routing changes or MAC modification.

### Key Rules

* Member interfaces **must** have zero logical dependencies, active IP addresses, or existing references.
* Widely implemented for Internal Segmentation Firewalls (ISFW) and Direct Server Return (DSR) designs.
* NAT within VWPs requires **Overload IP Pools**.

```bash
config system virtual-wire-pair
    edit "vwp-1"
        set member "port3" "port4"
        set wildcard-vlan disable
    next
end

```

> **Policy Enforcement:** Managed separately under `Policy & Objects > Firewall Virtual Wire Pair Policy`. Supports both Flow and Proxy inspection engines.

### Parallel Redundancy Protocol (PRP - IEC 62439-3)

Provides zero-failover-time Layer 2 redundancy for mission-critical industrial networks (SCADA, substations) by transmitting duplicate frames concurrently across two isolated networks (LAN A & LAN B).

```bash
# Enable PRP Trailer Processing
config system settings
    set prp-trailer-action enable
end

# Assign Hardware Acceleration Ports
config system npu
    set prp-port-in "port15"
    set prp-port-out "port16"
end

# Bind PRP Members into Virtual Wire Pair
config system virtual-wire-pair
    edit "vwp-prp"
        set member "port15" "port16"
    next
end

```

---

## 🎭 Enhanced MAC (EMAC) VLAN

EMAC VLANs allow creating logical interfaces with **unique MAC addresses** on top of a single physical parent interface.

```
                 ┌── EMAC 1 (MAC: 00:09:0F:AA:00:01)
[ Physical Port ]┼── EMAC 2 (MAC: 00:09:0F:AA:00:02)
                 └── EMAC 3 (MAC: 00:09:0F:AA:00:03)

```

### Key Considerations & Restrictions

* **Operational Mode:** Must run in **NAT Mode**.
* **DHCP Limitations:** DHCP relay or server configurations may experience anomalous behavior on EMAC interfaces due to MAC binding constraints.
* **Unique Pair Requirement:** A VLAN ID paired with a physical interface must remain unique across the entire chassis, including across different VDOMs.
* **HA Behavior:** Synchronizes and fails over identically to physical interfaces across High Availability clusters.
* **Restrictions:** Do **not** use EMAC on dedicated HA Heartbeat links, global Out-of-Band Management interfaces, or Transparent VDOM links.

---

## 🌐 VXLAN Architecture & Encapsulation

VXLAN encapsulates Layer 2 Ethernet frames inside Layer 3 UDP datagrams (port `4789`) to establish overlay networks over Layer 3 infrastructure.

```bash
# Define Core VXLAN Overlay Tunnel
config system vxlan
    edit "vx10"
        set interface "port2"
        set remote-ip 22.22.22.2
        set vni 1000
    next
end

```

### Implementation Patterns & Best Practices

1. **Dialup / Hub-and-Spoke IPsec Overlays:**
* Disable automatic device creation.
* On spoke nodes, align Phase 2 IPsec subnets with local tunnel interface IPs (e.g., `12.23.34.2`) to establish deterministic reverse routing paths without creating unnecessary dynamic interfaces.


2. **VLAN Tagging over VXLAN:**
* Create standard VLAN sub-interfaces underneath the parent VXLAN tunnel object using unique VLAN IDs to enable multi-tenant frame encapsulation.


3. **L2 Switching Integration:**
* Bind client physical VLAN ports (e.g., `vlan10` on `port3`) and overlay VLAN ports (`vlan10` on `vx10`) into a common **Software Switch**.
* ⚠️ **Never** add the base VXLAN tunnel object directly into a software switch.


4. **Intra-Switch Policy Modes:**
* **Implicit:** Traffic routes automatically between member ports without explicit policy checks.
* **Explicit:** Enforces firewall inspection policies between ports within the software switch.


5. **DHCP Services over VXLAN:**
* Add the base VXLAN interface and local target ingress port into a dedicated software switch, assign an IP address to the switch interface, and attach the system DHCP Server service directly to that switch.



```

```

# FortiGate Pre-Deployment & Interface Design  

> **SheynShield | Engineering Secure Networks**
>
> **FortiGate • FortiOS • Interfaces • LACP • Redundant Interface • Virtual Wire Pair • VLAN • Zone • DHCP • DNS • Explicit Proxy • Captive Portal • FSSO • Endpoint Control**
>
> **NSE 4 + NSE 7 Engineering Reference**

---

# 🎯 1. Pre-Deployment Checklist

Before connecting FortiGate to the production network, validate the hardware, software, topology, addressing, management, licensing, and security requirements.

```text
                    FORTIGATE DEPLOYMENT
                            │
       ┌────────────────────┼────────────────────┐
       ▼                    ▼                    ▼
    Hardware             Software             Design
       │                    │                    │
       ▼                    ▼                    ▼
   Ports/SFP            FortiOS Version      IP Plan
   Power                Firmware             VLANs
   Cables               Release Notes        Routing
   Transceivers         Upgrade Path         HA
                                               │
                                               ▼
                                          Security Policy
```

### Pre-Deployment Checklist

```text
☐ Confirm FortiGate model
☐ Confirm FortiOS version
☐ Verify firmware upgrade path
☐ Read release notes
☐ Check known issues
☐ Check resolved issues
☐ Verify licenses/subscriptions
☐ Verify SFP/SFP+/QSFP requirements
☐ Prepare management IP
☐ Prepare hostname
☐ Configure timezone
☐ Configure NTP
☐ Prepare DNS
☐ Prepare VLAN plan
☐ Prepare IP addressing plan
☐ Prepare routing plan
☐ Prepare security zones
☐ Prepare WAN/ISP information
☐ Prepare HA design if required
☐ Prepare FortiManager integration if required
☐ Prepare FortiAnalyzer integration if required
☐ Backup configuration before major changes
```

---

# 🔌 2. Common Optical Interface Terminology

| Term | Typical Speed |
| ---- | ------------: |
| SFP  |        1 Gb/s |
| SFP+ |       10 Gb/s |
| QSFP |       40 Gb/s |

> ⚠️ These are common module categories, not absolute speed guarantees. Actual speed depends on the transceiver, interface, platform, and supported standards.

### Deployment Rule

Always verify:

```text
FortiGate Model
      +
Interface Type
      +
Transceiver
      +
Fiber/Copper Type
      +
Speed
      +
Distance
```

before deployment.

---

# 🏷️ 3. First Configuration: Hostname

The hostname should be changed immediately after initial deployment.

Example:

```text
FGT-HQ-EDGE-01
FGT-DC-FW-01
FGT-BRANCH-01
FGT-DMZ-01
```

A good naming convention communicates:

```text
Location + Role + Number
```

Example:

```text
TEH-DC-FW-01
```

---

# 🕐 4. Timezone & NTP

Correct time is critical for:

* Logs
* Authentication
* Certificates
* VPN
* FortiAnalyzer correlation
* SIEM correlation
* Incident investigation
* Troubleshooting
* Scheduled tasks

Mental model:

```text
Correct Time
     ↓
Accurate Logs
     ↓
Accurate Correlation
     ↓
Better Troubleshooting
     ↓
Better Incident Response
```

---

## CLI Example

Timezone syntax is version-dependent, but conceptually:

```cli
config system global
    set timezone <TIMEZONE_ID>
end
```

Configure NTP:

```cli
config system ntp
    set type custom

    config ntpserver
        edit 1
            set server <NTP_SERVER>
        next
    end
end
```

### Recommended Architecture

Prefer reliable internal or approved NTP sources:

```text
FortiGate
   │
   ▼
Internal NTP
   │
   ▼
Trusted Time Source
```

rather than randomly assigning public IP addresses.

---

# 🧠 5. Why Time Matters in NSE 7

Suppose an incident occurs at:

```text
10:32:15
```

You investigate:

```text
FortiGate
FortiAnalyzer
FortiManager
Windows AD
DNS
Proxy
SIEM
Endpoint
```

If every device has a different clock:

```text
FortiGate       10:32
Windows         10:36
SIEM             10:29
Endpoint         10:34
```

correlation becomes unreliable.

Therefore:

```text
NTP
 ↓
Time Synchronization
 ↓
Log Correlation
```

is a security requirement, not merely a configuration preference.

---

# 🔗 6. Understanding the `Ref` Column

FortiOS configuration objects can display references indicating where an object is currently being used.

Conceptually:

```text
Object
  │
  ├── Policy
  ├── VIP
  ├── Route
  ├── Zone
  ├── Interface
  └── Other Configuration
```

Before deleting or fundamentally modifying an object:

```text
Check References
       ↓
Identify Dependencies
       ↓
Remove/modify Dependencies
       ↓
Change Object
```

### Important Rule

If the reference count is:

```text
0
```

the object may have no current configuration references.

If:

```text
Ref > 0
```

something is still using it.

### NSE 7 Mental Model

Think in terms of **configuration dependency graphs**:

```text
Interface
   ↓
Zone
   ↓
Policy
   ↓
Security Profile
   ↓
Logging
```

Changing one object can affect multiple downstream components.

---

# 🧩 7. Interface Architecture

FortiGate interfaces can be physical or logical.

```text
INTERFACE
   │
   ├── Physical Interface
   │
   ├── VLAN Interface
   │
   ├── Aggregate Interface
   │
   ├── Redundant Interface
   │
   ├── Software Switch
   │
   ├── Virtual Wire Pair
   │
   ├── Loopback
   │
   └── Other Logical Interfaces
```

---

# 🔗 8. Interface Roles

Common interface roles include:

```text
WAN
LAN
DMZ
Undefined
```

The role is primarily used to influence GUI behavior and available defaults; it should not be confused with a firewall security policy.

---

# 🌐 9. WAN Role

WAN interfaces are intended for Internet/WAN connectivity.

Depending on FortiOS version and configuration, WAN-related behavior can include:

* Internet Service Database integration
* Botnet-related controls
* Default gateway retrieval
* WAN-specific GUI behavior
* Multiple addressing support

### Important

Do not assume:

```text
Role = WAN
```

automatically means:

```text
Internet access = working
```

You still need:

```text
Route
+
Firewall Policy
+
NAT where required
+
Correct upstream connectivity
```

---

# 🛣️ 10. DHCP Default Gateway

If the WAN interface obtains its addressing through DHCP, FortiGate can learn the gateway dynamically.

Conceptually:

```text
ISP DHCP
    │
    ├── IP Address
    ├── Subnet
    └── Default Gateway
             │
             ▼
          FortiGate
```

This can remove the need to manually configure a static default route in some single-WAN designs.

### Multi-WAN Warning

With multiple ISPs:

```text
ISP1
 │
 ├── WAN1
 │
 └── Gateway 1

ISP2
 │
 ├── WAN2
 │
 └── Gateway 2
```

you should explicitly design:

```text
Static Routes
+
SD-WAN
+
Priority
+
Health Checks
+
Failover
```

rather than relying blindly on automatically learned gateways.

---

# 🔗 11. 802.3ad Aggregate Interface / LACP

An aggregate interface combines multiple physical interfaces into one logical interface.

```text
              FortiGate
                 │
        ┌────────┼────────┐
        │        │        │
       Port1    Port2    Port3
        │        │        │
        └────────┼────────┘
                 │
             LACP Bundle
                 │
                 ▼
              Switch
```

Common standard:

```text
IEEE 802.3ad
```

Modern terminology:

```text
LACP
```

---

# ⚡ 12. LACP Benefits

Aggregation provides:

### Increased aggregate capacity

```text
1G + 1G
   ↓
Logical Aggregate
   ↓
Up to aggregate capacity depending on hashing/design
```

### Link redundancy

If one member fails:

```text
Port1 ── X
Port2 ── UP
Port3 ── UP

Traffic continues through surviving members.
```

### Important

Do not interpret LACP as:

```text
2 × 1G = One single 2G TCP flow
```

Actual utilization depends on:

* Hashing
* Traffic flows
* Platform
* LACP configuration
* Switch configuration

---

# 🆚 13. Aggregate vs Redundant Interface

| Feature                       | Aggregate             | Redundant            |
| ----------------------------- | --------------------- | -------------------- |
| Multiple physical interfaces  | Yes                   | Yes                  |
| Uses all links simultaneously | Yes                   | No                   |
| Typical protocol              | LACP                  | FortiGate redundancy |
| Aggregate bandwidth           | Yes                   | No                   |
| Link failover                 | Yes                   | Yes                  |
| Main purpose                  | Capacity + resilience | Resilience           |

### Mental Model

```text
Aggregate:

Port1 ─┐
Port2 ─┼──► USE TOGETHER
Port3 ─┘


Redundant:

Port1 ─► ACTIVE
Port2 ─► STANDBY
```

---

# 🚨 14. LACP Preconditions

Before creating an aggregate interface, member interfaces generally must not already be actively referenced by incompatible configuration objects.

Check:

```text
☐ Physical interfaces
☐ Same VDOM
☐ No existing IP configuration
☐ No DHCP/PPPoE dependency
☐ Not already in another aggregate
☐ Not already in redundant interface
☐ Not HA heartbeat interfaces
☐ Not referenced by policies
☐ Not referenced by VIPs
☐ Not referenced by IP pools
☐ Not referenced by multicast policies
☐ No conflicting routes
```

After aggregation:

```text
Physical Members
       ↓
Hidden/consumed under Aggregate
       ↓
Configure Logical Aggregate Interface
```

---

# 🧠 15. LACP Troubleshooting

If LACP does not come up:

```text
FortiGate
   │
   ├── Physical Link?
   │
   ├── Speed/Duplex?
   │
   ├── LACP mode?
   │
   ├── Correct members?
   │
   └── Same LAG on switch?
             │
             ▼
         Switch Side
```

Check:

```text
☐ LACP enabled
☐ Correct ports
☐ Correct VLAN configuration
☐ Correct trunk configuration
☐ No port mismatch
☐ No speed mismatch
☐ LACP state
```

---

# 🔌 16. Virtual Wire Pair

A Virtual Wire Pair logically connects two interfaces.

Concept:

```text
             FortiGate
          Virtual Wire Pair
              │     │
              │     │
            port3  port4
              │     │
              │     │
            Network A
                 │
            Network B
```

Traffic entering one member can be forwarded toward the other member when permitted by the appropriate firewall policy.

---

# 🎯 17. Virtual Wire Pair Use Cases

Useful when FortiGate needs to be inserted inline without acting as the normal routed gateway.

Example:

```text
LAN
 │
 ▼
FortiGate VWP
 │
 ▼
Web Server
```

The topology remains physically inline:

```text
LAN ── FortiGate ── Server
```

while FortiGate applies security policy.

---

# 🔥 18. Virtual Wire Pair Policy

Conceptual configuration:

```cli
config system virtual-wire-pair
    edit "VWP-LAN-WEB"
        set member "port3" "port4"
        set wildcard-vlan disable
    next
end
```

Then create an appropriate firewall policy.

Example structure:

```cli
config firewall policy
    edit 1
        set name "VWP-LAN-WEB"
        set srcintf "port3"
        set dstintf "port4"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
        set utm-status enable
    next
end
```

> Exact syntax and policy requirements can vary by FortiOS version and VWP topology.

---

# 🧠 19. VWP — NSE 7 Perspective

Virtual Wire Pair is particularly useful when the network has unusual Layer 2 behavior.

Example:

```text
Direct Server Return
       ↓
Unexpected MAC Path
       ↓
Normal Routing Design May Fail
       ↓
VWP Can Provide Inline Security
```

Before deployment, understand:

```text
Ingress Interface
      ↓
VWP Pair
      ↓
Policy
      ↓
Inspection
      ↓
Egress Interface
```

---

# 🖥️ 20. Software Switch

A software switch combines multiple interfaces into a logical switching construct.

Concept:

```text
port1 ─┐
port2 ─┼──► Software Switch
port3 ─┘
             │
             ▼
        Logical Interface
```

It is conceptually similar to a bridge.

---

# 🆚 21. Hardware Switch vs Software Switch

| Feature                    | Hardware Switch   | Software Switch                   |
| -------------------------- | ----------------- | --------------------------------- |
| Switching                  | Hardware-assisted | Software/logical                  |
| ASIC involvement           | Model-dependent   | More CPU/software involvement     |
| Interfaces appear as group | Yes               | Yes                               |
| Use case                   | Simple LAN        | Flexible logical grouping         |
| Performance                | Usually better    | Depends on platform/configuration |

> Exact acceleration behavior depends on FortiGate hardware.

---

# 🔄 22. Internal Switch Mode

Some FortiGate models support different internal switch configurations.

Two important concepts:

```text
Switch Mode
Interface Mode
```

---

## Switch Mode

Multiple physical ports behave as a single logical switch.

```text
port1 ─┐
port2 ─┤
port3 ─┼──► internal
port4 ─┘
```

The ports typically share one Layer 3 interface/subnet.

Suitable for:

```text
Simple LAN
Single Subnet
Basic Deployment
```

---

## Interface Mode

Each physical interface can have its own Layer 3 identity.

```text
port1 → 10.1.1.1/24
port2 → 10.2.2.1/24
port3 → 10.3.3.1/24
```

Suitable for:

```text
Multiple Subnets
DMZ
WAN
Complex Topology
Segmentation
```

---

# 🔧 23. Internal Switch Mode CLI

Historical configuration example:

```cli
config system global
    set internal-switch-mode switch
end
```

or:

```cli
config system global
    set internal-switch-mode interface
end
```

> ⚠️ Changing internal switch mode can have major configuration impact. Treat it as a topology-level change, not a routine interface modification.

Before changing it:

```text
BACKUP
   ↓
Check References
   ↓
Document Current Topology
   ↓
Plan Downtime
   ↓
Change
   ↓
Reconfigure Interfaces
   ↓
Validate
```

---

# 🏷️ 24. VLAN Subinterfaces

FortiGate can create VLAN interfaces on a physical parent interface.

Example:

```text
                  Switch
                    │
                  TRUNK
                    │
                    ▼
                FortiGate
                  port4
                    │
          ┌─────────┼─────────┐
          │         │         │
       VLAN 100   VLAN 200   VLAN 300
          │         │         │
       10.100.1.1 10.200.1.1 10.300.1.1
```

---

# 🔧 25. VLAN Configuration Example

```cli
config system interface

    edit "VLAN_100"
        set interface "port4"
        set type vlan
        set vlanid 100
        set ip 172.100.1.1 255.255.255.0
        set allowaccess https ping
    next

end
```

### Important

The physical interface should normally connect to a switch trunk carrying the required VLANs.

---

# 🧠 26. Same VLAN ID ≠ Internal Connection

This is a critical FortiGate concept.

You can have:

```text
port1
 └── VLAN 300

port2
 └── VLAN 300
```

but:

```text
VLAN 300 on port1
        ≠
VLAN 300 on port2
```

They are separate interfaces.

The same VLAN ID does **not automatically create an internal Layer 2 bridge** between them.

Conceptually:

```text
port1.VLAN300
      │
      X
      │
port2.VLAN300
```

Traffic between them requires the appropriate FortiGate forwarding/policy architecture.

---

# 🔀 27. Inter-VLAN Routing

Typical topology:

```text
                FortiGate
                    │
             ┌──────┼──────┐
             │      │      │
           VLAN10 VLAN20 VLAN30
             │      │      │
           Users   Servers Guest
```

For VLAN-to-VLAN traffic:

```text
VLAN 10
   ↓
Route
   ↓
Firewall Policy
   ↓
VLAN 20
```

### Golden Rule

```text
VLAN Interface
      ≠
Automatic Permission
```

You still need the appropriate firewall policy.

---

# 🛡️ 28. Inter-VLAN Policy

Example:

```text
Source Interface = VLAN10
Destination       = VLAN20
Source Address    = Required subnet
Destination       = Required subnet
Service           = Required services
Action             = ACCEPT
```

For Internet access:

```text
VLAN
 ↓
Policy
 ↓
NAT
 ↓
WAN
```

---

# 📦 29. Zone

A zone is a logical group of interfaces.

Example:

```text
                    INTERNET
                       │
                     WAN
                       │
                 ┌─────┴─────┐
                 │ FortiGate │
                 └─────┬─────┘
                       │
                  SERVER ZONE
                  /          \
              VLAN100       VLAN200
```

Instead of creating separate policies for every interface combination, zones can simplify policy design.

---

# 🎯 30. Why Use Zones?

Without zones:

```text
VLAN10 → WAN
VLAN20 → WAN
VLAN30 → WAN
VLAN40 → WAN
...
```

With a logical zone:

```text
USER-ZONE
   │
   ├── VLAN10
   ├── VLAN20
   ├── VLAN30
   └── VLAN40
```

Policy becomes easier to manage.

### But:

```text
Zone
  ≠
Security Policy
```

The firewall policy still controls the traffic.

---

# 🔒 31. Inter-Zone Traffic

If interfaces are grouped into a zone, determine whether communication between zone members is allowed.

A useful security design principle:

```text
Users
  X
Servers

Users
  X
Management

Guest
  X
Internal
```

Use explicit policies for required communication.

### Segmentation Rule

```text
Group interfaces for administration,
not to bypass security architecture.
```

---

# 🌐 32. Overlapping Subnets

FortiGate can support subnet-overlap behavior in applicable configurations.

Example:

```cli
config system settings
    set allow-subnet-overlap enable
end
```

This should not be enabled casually.

Use cases may include:

* Overlapping networks
* Migrations
* Multi-tenant designs
* Complex VPN scenarios

### Warning

Overlapping addressing can create:

```text
Routing Ambiguity
+
Policy Complexity
+
Troubleshooting Difficulty
```

Treat it as an architecture decision.

---

# 📡 33. DHCP Server

FortiGate can provide DHCP services for LAN/VLAN interfaces.

Typical architecture:

```text
Client
  │
  │ DHCP
  ▼
FortiGate
  │
  ├── IP
  ├── Mask
  ├── Gateway
  ├── DNS
  └── Lease
```

---

# 🧷 34. DHCP Reservation

DHCP reservations bind a client identity, typically MAC address, to a predictable IP address.

Useful for:

```text
Printers
Cameras
Servers
Network Appliances
IoT Devices
Infrastructure
```

Concept:

```text
MAC Address
     ↓
Reserved IP
     ↓
Same Client
     ↓
Predictable Addressing
```

---

# ⏱️ 35. DHCP Lease Time

Lease duration controls how long a DHCP client can use an assigned address before renewal.

Example CLI:

```cli
config system dhcp server
    edit <SERVER_ID>
        set lease-time <SECONDS>
    next
end
```

Long leases:

```text
Less DHCP traffic
+
Stable clients
```

Short leases:

```text
Faster address recycling
+
More DHCP traffic
```

Choose based on network requirements.

---

# 🧹 36. DHCP Lease Troubleshooting

When troubleshooting DHCP:

```text
Client
 ↓
DHCP Discover
 ↓
DHCP Offer
 ↓
DHCP Request
 ↓
DHCP ACK
```

Check:

```text
☐ Interface
☐ DHCP server enabled
☐ Address range
☐ Excluded/reserved addresses
☐ VLAN
☐ DHCP relay if applicable
☐ Gateway
☐ DNS
☐ Lease state
```

---

# 🧭 37. DNS Architecture

FortiGate can use DNS for:

* Name resolution
* FQDN objects
* Security services
* FortiGuard communication
* Administrative functions
* DNS forwarding

Recommended enterprise design:

```text
Clients
   │
   ▼
Internal DNS
   │
   ▼
Approved DNS Forwarders
   │
   ▼
Internet / Security DNS
```

Avoid randomly mixing:

```text
Public DNS
+
Internal DNS
+
FortiGuard DNS
```

without understanding the resolution path.

---

# 🔀 38. DNS Forwarder

FortiGate can act as a DNS forwarding server on an interface.

Concept:

```text
Client
  │
  │ DNS Query
  ▼
FortiGate
  │
  │ Forward
  ▼
Internal DNS
```

Example concept:

```cli
config system dns-server
    edit "port3"
        set mode forward-only
    next
end
```

---

# 🧠 39. DHCP + DNS Design

A clean branch design can be:

```text
Client
   │
   ▼
DHCP
   │
   ├── IP Address
   ├── Gateway
   └── DNS
          │
          ▼
       FortiGate
          │
          ▼
    Internal DNS Server
```

This provides centralized control and predictable resolution.

---

# 🌐 40. Explicit Web Proxy

Explicit proxy requires clients to send web traffic through the FortiGate proxy.

Concept:

```text
Client
   │
   │ Proxy Request
   ▼
FortiGate Explicit Proxy
   │
   ▼
Internet
```

Clients must be configured to use the proxy.

---

# 🧩 41. Explicit Proxy Architecture

```text
                  FortiGate
                     │
             Explicit Proxy
                     │
              ┌──────┴──────┐
              │             │
            HTTP          HTTPS
              │             │
              └──────┬──────┘
                     ▼
                  Internet
```

---

# 🖥️ 42. Explicit Proxy Configuration Areas

Typical GUI areas can include:

```text
Network
  ↓
Explicit Proxy
```

Possible configuration elements:

```text
☐ Listen Interfaces
☐ HTTP Proxy Port
☐ HTTPS Proxy behavior
☐ Proxy FQDN
☐ Authentication
☐ PAC File
☐ Web Cache
☐ Request Limits
☐ Forwarding
☐ Default Policy Action
```

---

# 📜 43. PAC — Proxy Auto Configuration

A PAC file allows clients to automatically determine proxy settings.

Without PAC:

```text
Admin
  ↓
Manually configure every client
```

With PAC:

```text
Client
  ↓
PAC File
  ↓
Automatic Proxy Selection
```

This can simplify enterprise deployment.

---

# 🔗 44. Proxy Chaining

FortiGate can participate in proxy-forwarding architectures where traffic is forwarded to another proxy.

Concept:

```text
Client
   ↓
FortiGate Proxy
   ↓
Upstream Proxy
   ↓
Internet
```

Useful in:

* Enterprise proxy architectures
* Centralized Internet access
* Multi-layer security designs

---

# 🔐 45. Captive Portal

Captive portal forces users to authenticate before receiving normal network access.

Concept:

```text
Guest
  ↓
Network Access
  ↓
Captive Portal
  ↓
Authentication
  ↓
Policy Decision
  ↓
Internet
```

---

# 👤 46. FSSO

**Fortinet Single Sign-On (FSSO)** allows FortiGate to associate users with IP addresses using authentication information obtained from supported identity sources.

Typical environment:

```text
Windows Users
      │
      ▼
Active Directory
      │
      ▼
FSSO Collector / Agent
      │
      ▼
FortiGate
      │
      ▼
Identity-Based Policy
```

---

# 🏢 47. FSSO Architecture

```text
                 Active Directory
                        │
                        ▼
                  FSSO Collector
                        │
                        ▼
                    FortiGate
                        │
                        ▼
                 Firewall Policy
                        │
                        ▼
                   User Group
```

The exact architecture depends on the FortiOS/FSSO deployment model.

---

# 🔌 48. FSSO Port

A commonly encountered FSSO-related communication port is:

```text
TCP/8002
```

> Port usage can vary by component and deployment. Always verify the exact Fortinet version and FSSO architecture.

---

# 🔄 49. FSSO Refresh

When troubleshooting stale identity information, refreshing FSSO information can be useful.

Example:

```cli
execute fsso refresh
```

Then verify:

```text
User
 ↓
IP
 ↓
Group
 ↓
FortiGate
```

---

# 👥 50. Identity-Based Firewall Policy

Instead of only:

```text
Source IP
```

you can build policies around:

```text
User
+
Group
+
Source Interface
+
Destination
+
Service
```

Example:

```text
Guest Group
     ↓
Internet
     ↓
Restricted Services
```

---

# 🚨 51. Captive Portal Security Design

Avoid:

```text
Guest
   ↓
ALL
   ↓
Internet
```

without controls.

Better:

```text
Guest
 ↓
Authentication
 ↓
Restricted Group
 ↓
Firewall Policy
 ↓
Security Profiles
 ↓
Internet
```

Apply:

```text
☐ Web Filtering
☐ DNS Filtering
☐ Application Control
☐ Antivirus
☐ IPS
☐ Logging
☐ Rate Limiting where appropriate
```

---

# 🖥️ 52. Guest Network Architecture

Recommended:

```text
                    INTERNET
                        ▲
                        │
                   Firewall Policy
                        ▲
                        │
                  GUEST VLAN/ZONE
                        │
                        ▲
                 Captive Portal
                        ▲
                        │
                     Guest
```

And explicitly isolate:

```text
Guest
  X
Internal LAN

Guest
  X
Management

Guest
  X
Server Network
```

---

# 🔍 53. Endpoint Control

FortiGate can integrate with endpoint information to improve device visibility and control.

Possible concepts include:

```text
Device Detection
+
Device Identification
+
FortiClient Integration
+
Endpoint Posture
+
Access Control
```

---

# 📡 54. Device Detection

Useful device visibility features can help identify:

```text
PC
Laptop
Phone
Printer
IoT
Server
Network Device
```

Possible mechanisms include:

```text
Passive Detection
+
Active Scanning
```

---

# 🧭 55. Device Discovery

In applicable FortiOS versions, device discovery options can include:

```text
Device Detection
Broadcast Discovery
Active Scanning
```

A good enterprise approach:

```text
Passive Visibility
       +
Active Verification
```

rather than relying on one detection mechanism.

---

# 🛡️ 56. FortiClient Integration

FortiGate can integrate with FortiClient to obtain endpoint-related information.

Concept:

```text
Endpoint
   │
   ▼
FortiClient
   │
   ▼
FortiGate
   │
   ├── Device Identity
   ├── Endpoint Information
   └── Access Control
```

This can support stronger endpoint-aware security architectures.

---

# 🚫 57. Botnet / Endpoint Blocking Concepts

Security architectures may include controls where suspicious or compromised endpoints are restricted.

Concept:

```text
Endpoint
   ↓
Threat Detection
   ↓
Risk Decision
   ↓
Warning / Block
```

Do not enable aggressive blocking without understanding:

```text
Detection Source
+
False Positive Risk
+
Business Impact
+
Exception Process
```

---

# ➕ 58. Secondary IP Addresses

An interface can support secondary IP addressing in applicable configurations.

Concept:

```text
Interface
 ├── Primary IP
 ├── Secondary IP
 └── Secondary IP
```

Possible use cases:

* Migration
* Multiple logical subnets
* Legacy applications
* Temporary addressing
* Network transition

But avoid using secondary IPs as a substitute for proper VLAN segmentation when VLAN architecture is available.

---

# 🌐 59. Transparent Mode Reminder

In Transparent Mode:

```text
FortiGate
   =
Layer 2 Security Bridge
```

It can still provide security inspection such as:

```text
Antivirus
IPS
Web Filtering
Anti-Spam
```

but some routed features are unavailable or architecturally different.

Common limitations include features that depend on Layer 3/NAT functionality, such as:

```text
Traditional NAT
DHCP Server
Some VPN Modes
```

> Exact limitations depend on FortiOS version and feature architecture.

---

# 🧠 60. Deployment Decision Tree

Before creating an interface, ask:

```text
Do I need Layer 3 routing?
        │
     YES│
        ▼
     NAT Mode
        │
        ├── VLAN?
        ├── Zone?
        ├── Aggregate?
        └── Secondary IP?
```

If FortiGate must remain inline at Layer 2:

```text
Need transparent insertion?
        │
       YES
        │
        ▼
Transparent / VWP Design
        │
        ├── Forward Domain
        ├── L2 Forwarding
        ├── STP
        └── VLAN Design
```

---

# 🧪 61. NSE 4 — Interface Design Checklist

```text
☐ Physical Interface
☐ Interface Role
☐ IP Address
☐ Administrative Access
☐ VLAN
☐ Aggregate Interface
☐ Redundant Interface
☐ Software Switch
☐ Zone
☐ DHCP
☐ DNS
☐ NAT
☐ Transparent Mode
☐ Virtual Wire Pair
☐ Basic Proxy
```

---

# 🧠 62. NSE 7 — Engineering Checklist

At advanced level, don't just ask:

> "How do I configure the interface?"

Ask:

```text
WHY this interface type?
        ↓
WHAT traffic enters?
        ↓
WHAT traffic leaves?
        ↓
WHO owns the gateway?
        ↓
WHERE does routing occur?
        ↓
WHERE does security policy occur?
        ↓
HOW is redundancy achieved?
        ↓
HOW is Layer 2 handled?
        ↓
HOW is failure detected?
        ↓
HOW do I troubleshoot it?
```

---

# 🔎 63. NSE 7 Troubleshooting Matrix

| Problem                      | Investigate                         |
| ---------------------------- | ----------------------------------- |
| LACP not forming             | Physical links + LACP + switch      |
| VLAN unavailable             | Trunk + VLAN ID + interface         |
| Same VLAN not communicating  | Interface architecture              |
| VWP traffic blocked          | VWP + policy                        |
| Layer 2 loop                 | STP                                 |
| DHCP unavailable             | VLAN + DHCP + relay                 |
| DNS unavailable              | DNS server + forwarding             |
| FSSO user missing            | AD + collector + connectivity       |
| Captive portal not appearing | Interface + authentication + policy |
| Guest reaches internal LAN   | Zone/policy/segmentation            |
| Endpoint not detected        | Device detection + telemetry        |
| Multiple ISP issue           | Routing + SD-WAN                    |
| Interface change fails       | Configuration references            |
| Management inaccessible      | Allowaccess + routing + ACL         |

---

# 🛡️ 64. Production Hardening Checklist

## Management

```text
☐ Change hostname
☐ Configure timezone
☐ Configure NTP
☐ Restrict management interfaces
☐ Disable unnecessary administrative protocols
☐ Use HTTPS
☐ Use SSH where required
☐ Use trusted certificates
☐ Enable MFA
☐ Restrict admin source addresses
```

## Interfaces

```text
☐ Document every interface
☐ Define interface role
☐ Document IP/subnet
☐ Document VLAN ID
☐ Document switch port
☐ Document upstream/downstream device
☐ Check interface references
```

## WAN

```text
☐ ISP information documented
☐ Gateway verified
☐ DNS verified
☐ Default route verified
☐ NAT verified
☐ Security policy verified
☐ Failover requirement evaluated
```

## LAN

```text
☐ VLAN plan
☐ DHCP scope
☐ DHCP reservations
☐ DNS
☐ Inter-VLAN policies
☐ Segmentation
☐ Device discovery
```

---

# ⚡ 65. Golden Rules

```text
Hostname
    =
Device Identity

NTP
    =
Reliable Time + Reliable Logs

Ref Count
    =
Configuration Dependency

LACP
    =
Capacity + Redundancy

Redundant Interface
    =
Failover

Aggregate
    =
Use Multiple Links Together

Virtual Wire Pair
    =
Inline Layer 2 Security

VLAN
    =
Logical Segmentation

Zone
    =
Interface Grouping

Policy
    =
Actual Security Decision

DHCP
    =
Dynamic Address Assignment

DNS
    =
Name Resolution

FSSO
    =
User Identity

Captive Portal
    =
Interactive Authentication

FortiClient Integration
    =
Endpoint Visibility / Control
```

---

# ⚡ 66. 60-Second Deployment Revision

```text
                     NEW FORTIGATE
                          │
                          ▼
                 HARDWARE VALIDATION
                          │
                          ▼
                 FORTIOS / RELEASE NOTES
                          │
                          ▼
                 LICENSE / SERVICES
                          │
                          ▼
              HOSTNAME + TIMEZONE + NTP
                          │
                          ▼
                     INTERFACES
                          │
          ┌───────────────┼────────────────┐
          ▼               ▼                ▼
       PHYSICAL          VLAN          AGGREGATE
          │               │                │
          │               │                ▼
          │               │              LACP
          │               │
          ├───────────────┼───────────────┐
          ▼               ▼               ▼
        ZONE            DHCP             DNS
          │
          ▼
       POLICY
          │
          ▼
      SECURITY
          │
          ▼
       LOGGING
          │
          ▼
     MONITORING
```

---

# 🏁 Final Mental Model

A FortiGate interface is **not just an IP address**.

It is part of a larger forwarding and security architecture:

```text
                 INTERFACE
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
       L2           L3          Logical
        │            │            │
      VLAN         Route        Zone
      STP          Policy       VWP
      MAC          NAT          LACP
      ARP          SD-WAN       Switch
        │            │            │
        └────────────┼────────────┘
                     ▼
              SECURITY POLICY
                     │
                     ▼
             INSPECTION ENGINE
                     │
                     ▼
                 LOGGING
                     │
                     ▼
               ANALYTICS / SIEM
```

> **Design the interface from the traffic flow backward — not from the GUI forward.**

The correct question is not:

> **"Which interface type should I select?"**

The better engineering question is:

> **"What forwarding behavior, redundancy model, security boundary, and traffic path does this interface need to provide?"**

---

## 🔖 Keywords

`FortiGate Interface Configuration`
`FortiGate Interface  `
`FortiOS Interface Configuration`
`FortiGate LACP`
`FortiGate 802.3ad`
`FortiGate Aggregate Interface`
`FortiGate Redundant Interface`
`FortiGate Virtual Wire Pair`
`FortiGate VWP`
`FortiGate VLAN Configuration`
`FortiGate VLAN Interface`
`FortiGate Inter VLAN Routing`
`FortiGate Zone Configuration`
`FortiGate Software Switch`
`FortiGate Hardware Switch`
`FortiGate Internal Switch Mode`
`FortiGate NAT Mode`
`FortiGate Transparent Mode`
`FortiGate DHCP Server`
`FortiGate DHCP Reservation`
`FortiGate DNS Forwarder`
`FortiGate Explicit Proxy`
`FortiGate PAC File`
`FortiGate Captive Portal`
`FortiGate FSSO`
`FortiGate FortiClient Integration`
`FortiGate Device Detection`
`FortiGate Endpoint Control`
`FortiGate NSE4`
`FortiGate NSE7`
`FortiGate Troubleshooting`
`Fortinet Network Design`
`Fortinet Security Architecture`
`Fortinet  `
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

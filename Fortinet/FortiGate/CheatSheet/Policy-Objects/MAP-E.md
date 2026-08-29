# FortiGate MAP-E, DS-Lite & VNE  

> **FortiOS:** 7.2.x
> **Topic:** IPv4/IPv6 Transition, CGNAT, MAP-E, DS-Lite, VNE
> **Level:** NSE 6/7 — Advanced Networking
> **Brand:** SheynShield | Engineering Secure Networks

---

## 1. What Is VNE?

**VNE — Virtual Network Enabler/Encapsulator** provides mechanisms for IPv4/IPv6 transition and encapsulation.

It is particularly useful when an IPv4 service must operate across an IPv6-only or IPv6-dominant provider network.

### Main VNE modes

| Mode       | Primary purpose                                                       |
| ---------- | --------------------------------------------------------------------- |
| `ds-lite`  | Carry IPv4 traffic over an IPv6 network toward an IPv4/CGNAT function |
| `fixed-ip` | Build an IPv4-over-IPv6 service using provider-assigned parameters    |
| `map-e`    | Map IPv4 addressing/ports into an IPv6 transport domain               |

### Typical use cases

* ISP IPv6-only access networks
* IPv4-as-a-service over IPv6
* Carrier-grade NAT
* Broadband networks
* Mobile/large-scale subscriber networks
* IPv4/IPv6 transition architectures

---

# 2. CGNAT + IPv6 Transition

**CGNAT (Carrier-Grade NAT)** allows many subscribers to share a limited number of public IPv4 addresses.

A common architecture:

```text
Client
192.168.1.10:50000
        |
        v
      CE
        |
        | NAT
        v
100.64.1.10:50000
        |
        v
      CGNAT
        |
        | NAT
        v
Public IPv4
        |
        v
     Internet
```

### Why `100.64.0.0/10`?

`100.64.0.0/10` is the IPv4 Shared Address Space commonly used between subscriber-side equipment and CGNAT infrastructure.

```text
100.64.0.0/10
```

It provides:

```text
100.64.0.0
        |
        +--------------------+
                             |
                        Subscriber
                        addressing
```

### Key idea

A subscriber may already perform NAT on the CE device:

```text
192.168.1.10:50000
        ↓
100.64.1.10:50000
        ↓
CGNAT
        ↓
Public-IP:Translated-Port
```

This is effectively **nested NAT**.

---

# 3. Why CGNAT Needs Port Sharing

A public IPv4 address has a finite TCP/UDP port space.

```text
0 ─────────────────────────────── 65535
```

FortiGate CGNAT implementations reserve part of this space for translation.

> **Important:** Do not treat `60416` as a universal mathematical limit for every FortiGate model/configuration. The usable translation-port capacity depends on FortiOS behavior, reserved ports, configuration, protocol and platform.

The important design principle is:

```text
More subscribers
        ↓
More translated ports
        ↓
More public IPv4 addresses
```

---

# 4. MAP-E — Core Concept

**MAP-E = Mapping of Address and Port using Encapsulation**

MAP-E allows IPv4 traffic to traverse an IPv6 network by encapsulating IPv4 packets inside IPv6.

Conceptually:

```text
IPv4 Client
192.168.1.10
      |
      v
   CE / CPE
      |
      | IPv4-in-IPv6
      v
IPv6 Provider Network
      |
      v
   Border Relay
      |
      v
   IPv4 Internet
```

### Core principle

```text
IPv4 traffic
     ↓
IPv6 encapsulation
     ↓
IPv6 transport network
     ↓
Border Relay
     ↓
IPv4 Internet
```

MAP-E is therefore **not simply another traditional NAT rule**.

It combines:

* IPv4 address/port mapping
* IPv6 transport
* Encapsulation
* Provider-defined mapping parameters

---

# 5. DS-Lite vs MAP-E

| Feature               | DS-Lite                       | MAP-E                                  |
| --------------------- | ----------------------------- | -------------------------------------- |
| Transport             | IPv6                          | IPv6                                   |
| IPv4 traffic          | Encapsulated                  | Encapsulated                           |
| Main mechanism        | IPv4-in-IPv6                  | Address/port mapping + IPv4-in-IPv6    |
| Provider architecture | AFTR-based                    | Border Relay-based                     |
| Subscriber IPv4       | Typically private/shared      | Algorithmically mapped                 |
| Typical deployment    | IPv6-only access              | IPv6-only broadband                    |
| CGNAT role            | AFTR performs centralized NAT | Mapping determines IPv4/port ownership |

### Memory trick

```text
DS-Lite
IPv4 → IPv6 tunnel → AFTR → IPv4 Internet

MAP-E
IPv4 + Port Mapping → IPv6 → BR → IPv4 Internet
```

---

# 6. VNE Lab Topology

A useful test topology:

```text
                         IPv4 / IPv6
                    12.12.12.0/30
                           |
                 2001:db8:12::/64
                           |
                +----------+----------+
                |                     |
             FGT-1                  FGT-2
                |                     |
        192.168.101.0/24       192.168.102.0/24
        2001:db8:101::/64      2001:db8:102::/64
                |
               DMZ
          192.168.20.200
```

> Use documentation/test prefixes such as `2001:db8::/32` in labs instead of real production IPv6 ranges.

---

# 7. DS-Lite — FortiGate Configuration

## FGT-1

```bash
config system vne
    set status enable
    set mode ds-lite
    set interface port2
    set br 2001:db8:12::2
    set ipv4-address 192.168.12.1/30
end
```

The VNE interface can then be inspected/configured as a tunnel interface.

```bash
config system interface
    edit "vne.root"
        set vdom "root"
        set ip 192.168.12.1 255.255.255.252
        set allowaccess ping
        set type tunnel
        set remote-ip 192.168.12.2 255.255.255.252

        config ipv6
            set ip6-address 2001:db8:1212::1/128
            set ip6-allowaccess ping
        end
    next
end
```

---

## FGT-2

```bash
config system vne
    set status enable
    set mode ds-lite
    set interface port2
    set br 2001:db8:12::1
    set ipv4-address 192.168.12.2/30
end
```

Tunnel interface:

```bash
config system interface
    edit "vne.root"
        set vdom "root"
        set ip 192.168.12.2 255.255.255.252
        set allowaccess ping
        set type tunnel
        set remote-ip 192.168.12.1 255.255.255.252

        config ipv6
            set ip6-address 2001:db8:1212::2/128
            set ip6-allowaccess ping
        end
    next
end
```

---

# 8. Routing Through VNE

Example IPv4 route:

```bash
config router static
    edit 1
        set dst 192.168.102.0 255.255.255.0
        set device "vne.root"
    next
end
```

IPv6 route:

```bash
config router static6
    edit 1
        set dst 2001:db8:102::/64
        set device "vne.root"
    next
end
```

---

# 9. Firewall Policy — VNE Traffic

Example outbound policy:

```bash
config firewall policy
    edit 1
        set name "VNE-to-WAN"
        set srcintf "vne.root" "lan"
        set dstintf "wan"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
        set nat enable
        set logtraffic all
    next
end
```

For internal traffic:

```bash
config firewall policy
    edit 2
        set name "VNE-Internal"
        set srcintf "vne.root" "lan" "dmz"
        set dstintf "vne.root" "lan" "dmz"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
        set nat disable
        set logtraffic all
    next
end
```

---

# 10. VNE Diagnostics

Check VNE daemon:

```bash
diagnose test application vned 1
```

Useful troubleshooting workflow:

```text
1. Check VNE status
        ↓
2. Check tunnel interface
        ↓
3. Check IPv4 route
        ↓
4. Check IPv6 route
        ↓
5. Check firewall policy
        ↓
6. Check NAT
        ↓
7. Check sessions
        ↓
8. Run packet/debug flow
```

---

# 11. Fixed-IP Mode

Fixed-IP mode can be used when the transition service receives provider-specific mapping/configuration information.

Example:

```bash
config system vne
    set status enable
    set mode fixed-ip
    set interface port2
    set br 2001:db8:12::1
    set ipv4-address 192.168.12.1/30
    set update-url http://192.168.20.200/map-e-config.xml
end
```

### Important

The `update-url` represents a mechanism for retrieving mapping/service parameters.

Treat this as **provider/service-specific configuration**, not a generic Internet configuration.

---

# 12. MAP-E Mode

Example:

```bash
config system vne
    set status enable
    set mode map-e
    set interface port2
    set ssl-certificate "test.com"
    set bmr-hostname "fgt-1"
end
```

### BMR

**BMR — Basic Mapping Rule**

The BMR defines the mapping relationship used by the MAP-E architecture.

Conceptually:

```text
IPv4 Prefix
     +
Port Set
     +
IPv6 Prefix
     ↓
MAP Mapping
```

The `bmr-hostname` should correspond to the expected provider/DNS configuration in the deployment.

---

# 13. MAP-E Traffic Flow

```text
          IPv4 Client
        192.168.101.x
               |
               v
         +-----------+
         |   FGT-1   |
         |   VNE     |
         +-----------+
               |
               | IPv4 encapsulated
               | in IPv6
               v
      IPv6 Provider Network
               |
               v
        MAP Border Relay
               |
               v
         IPv4 Internet
```

### Packet perspective

```text
Original:

IPv4
[SRC IPv4 | DST IPv4 | TCP/UDP | Payload]

             ↓ MAP-E

IPv6
[IPv6 Header
   +
   Encapsulated IPv4 Packet]
```

---

# 14. DS-Lite Traffic Flow

```text
IPv4 Client
    |
    v
 CPE / CE
    |
    | IPv4-in-IPv6
    v
IPv6 Network
    |
    v
   AFTR
    |
    | CGNAT
    v
IPv4 Internet
```

### Key distinction

In DS-Lite, the centralized IPv4 NAT function is typically associated with the **AFTR**.

In MAP-E, the architecture relies on **mapping rules and Border Relay behavior**.

---

# 15. CGNAT Address Planning

A common architecture:

```text
Subscriber LAN
192.168.x.x
      |
      v
     CE
      |
      v
100.64.0.0/10
      |
      v
    CGNAT
      |
      v
Public IPv4
      |
      v
Internet
```

### Shared Address Space

```text
100.64.0.0/10
```

This range is specifically intended for shared addressing in carrier environments.

Do **not** confuse it with:

```text
RFC1918 private ranges
```

such as:

```text
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
```

---

# 16. CGNAT Port Allocation

Conceptually, one public IPv4 address provides:

```text
0 ─────────────────────────────── 65535
```

But not every port is necessarily available for subscriber translation.

A CGNAT design may divide the available port space into blocks.

Example:

```text
Public IP
192.0.2.10

Port Space
+--------+--------+--------+--------+
| User A | User B | User C | User D |
+--------+--------+--------+--------+
```

Or:

```text
User A → ports 1024–2047
User B → ports 2048–3071
User C → ports 3072–4095
```

The actual allocation depends on the FortiOS CGNAT/IP-pool configuration and platform.

---

# 17. Why Port Block Allocation Matters

Port block allocation is useful when a large subscriber population must share a limited IPv4 pool.

Example:

```text
Public IP
    |
    +---- Block 1 → Subscriber A
    |
    +---- Block 2 → Subscriber B
    |
    +---- Block 3 → Subscriber C
    |
    +---- Block 4 → Subscriber D
```

### Design trade-off

```text
Larger block
    ↓
More ports per subscriber
    ↓
Fewer subscribers per public IP
```

Conversely:

```text
Smaller block
    ↓
Fewer ports per subscriber
    ↓
More subscribers per public IP
```

This is a classic **CGNAT scalability vs application compatibility** trade-off.

---

# 18. When Port Allocation Becomes Important

Pay special attention to applications that require many simultaneous connections:

* SIP / VoIP
* WebRTC
* P2P
* Large-scale web clients
* IoT deployments
* Enterprise applications
* Legacy applications
* High-concurrency subscribers

A subscriber that consumes a large number of source ports can exhaust its allocated block even when the public IP still has free ports.

---

# 19. DS-Lite vs MAP-E — Exam View

```text
                  IPv6 Transport
                        |
             +----------+----------+
             |                     |
          DS-Lite                MAP-E
             |                     |
          IPv4-in-IPv6       Mapping + Encapsulation
             |                     |
            AFTR                   BR
             |                     |
           CGNAT                IPv4 Internet
```

### Remember

> **DS-Lite = IPv4 over IPv6 + centralized AFTR**

> **MAP-E = IPv4/port mapping + IPv4 encapsulation over IPv6**

---

# 20. Troubleshooting Checklist

## Interface

```bash
show system interface
```

Check:

* VNE interface exists
* IPv4 address
* IPv6 address
* Remote IPv4
* Parent interface
* Administrative state

---

## Routing

```bash
get router info routing-table all
```

IPv6:

```bash
get router info6 routing-table
```

Verify:

```text
IPv4 destination → VNE
IPv6 transport → correct next hop/interface
```

---

## Firewall

Check:

```text
Source interface
Destination interface
Source address
Destination address
Service
NAT
Inspection mode
Logging
```

---

## Sessions

```bash
diagnose sys session list
```

For IPv6-related sessions:

```bash
diagnose sys session6 list
```

---

## Flow Debug

A typical flow-debug methodology:

```bash
diagnose debug reset
diagnose debug flow filter addr <IP>
diagnose debug flow show function-name enable
diagnose debug enable
diagnose debug flow trace start 50
```

Stop afterward:

```bash
diagnose debug disable
diagnose debug reset
```

---

# 21. Troubleshooting Decision Tree

```text
Client cannot reach destination
            |
            v
     Does VNE exist?
        /       \
      No         Yes
      |           |
 Create/enable   Check interface
                  |
                  v
            IPv6 reachable?
              /       \
            No         Yes
            |           |
       Fix IPv6       Check route
       transport         |
                          v
                    Correct VNE route?
                      /        \
                    No          Yes
                    |            |
                 Fix route     Check policy
                                  |
                                  v
                            Session created?
                              /       \
                            No         Yes
                            |           |
                       Check policy   Check NAT/
                       and routing    mapping/BR/AFTR
```

---

# 22. Common Design Mistakes

### ❌ Treating MAP-E as ordinary NAT

MAP-E is a transition architecture, not simply:

```text
Private IPv4 → Public IPv4
```

It introduces IPv6 transport and mapping behavior.

---

### ❌ Confusing DS-Lite with MAP-E

Remember:

```text
DS-Lite → AFTR
MAP-E   → Border Relay + Mapping
```

---

### ❌ Using production IPv6 prefixes in a lab

Use documentation prefixes:

```text
2001:db8::/32
```

instead of real public IPv6 allocations.

---

### ❌ Assuming every public IPv4 has exactly the same usable port capacity

Port availability is affected by:

* FortiOS implementation
* Reserved ports
* IP pool configuration
* Protocol
* Platform
* Translation requirements

---

### ❌ Forgetting IPv6 routing

A VNE tunnel can be operational while the actual IPv6 transport route is wrong.

Always verify:

```text
VNE
↓
IPv6 route
↓
Provider/peer
↓
BR/AFTR
```

---

# 23. NSE Memory Map

```text
IPv4/IPv6 Transition
│
├── CGNAT
│   ├── Shared IPv4
│   ├── Port allocation
│   ├── Port blocks
│   └── 100.64.0.0/10
│
├── DS-Lite
│   ├── IPv4-in-IPv6
│   ├── AFTR
│   └── Centralized CGNAT
│
├── MAP-E
│   ├── Mapping
│   ├── Port-set ownership
│   ├── IPv4-in-IPv6
│   └── Border Relay
│
└── FortiGate VNE
    ├── ds-lite
    ├── fixed-ip
    └── map-e
```

---

# 24. Quick Command Reference

| Purpose           | Command                             |
| ----------------- | ----------------------------------- |
| VNE configuration | `config system vne`                 |
| VNE daemon test   | `diagnose test application vned 1`  |
| IPv4 sessions     | `diagnose sys session list`         |
| IPv6 sessions     | `diagnose sys session6 list`        |
| IPv4 routing      | `get router info routing-table all` |
| IPv6 routing      | `get router info6 routing-table`    |
| Flow debug        | `diagnose debug flow ...`           |

---

## 🎯 Interview / Exam Questions

### What problem does DS-Lite solve?

It allows IPv4 connectivity to be delivered across an IPv6 access network by tunneling IPv4 traffic over IPv6 toward an AFTR.

### What is the main architectural component in DS-Lite?

**AFTR — Address Family Transition Router**

### What is MAP-E?

A mechanism that maps IPv4 address/port information into an IPv6 transport architecture and encapsulates IPv4 traffic across IPv6.

### What is the role of a Border Relay?

The Border Relay provides the transition between the MAP IPv6 domain and the IPv4 Internet.

### Why is CGNAT necessary?

Because the available public IPv4 address space is insufficient for assigning a unique public IPv4 address to every subscriber/device.

### Why is port allocation important?

Because multiple subscribers share the same public IPv4 address and must therefore be distinguished using transport-layer ports.

---

## ⚡ One-Minute Revision

```text
CGNAT
→ Many subscribers share public IPv4

100.64.0.0/10
→ Shared Address Space commonly used in carrier networks

DS-Lite
→ IPv4-in-IPv6
→ AFTR
→ Centralized CGNAT

MAP-E
→ IPv4/Port Mapping
→ IPv4 encapsulated in IPv6
→ Border Relay

VNE
→ FortiGate transition framework
→ DS-Lite / Fixed-IP / MAP-E

Port Block Allocation
→ Divide public IPv4 port space among subscribers

Main troubleshooting order
→ VNE
→ Interface
→ IPv6 transport
→ Route
→ Policy
→ NAT/mapping
→ Session
```

---

## 📌 SheynShield Takeaway

> **IPv6 transition is not simply "IPv4 vs IPv6". The real engineering challenge is preserving IPv4 reachability, subscriber identity, port scalability, routing, and security while the transport network moves toward IPv6.**

**SheynShield | Engineering Secure Networks**
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

# ADVPN (Auto Discovery VPN) Cheatsheet

> **Purpose:** Dynamic Hub-and-Spoke connectivity with automatic tunnel discovery and dynamic routing
> **Common use case:** Large-scale branch connectivity
> **Core concept:** Hub advertises/discovers spokes and enables dynamic spoke-to-spoke communication.

---

## 1. ADVPN Overview

| Component    | Role                                                        |
| ------------ | ----------------------------------------------------------- |
| Hub          | Auto-Discovery Sender                                       |
| Spoke        | Auto-Discovery Receiver                                     |
| Routing      | BGP / OSPF / RIP                                            |
| Tunnel       | IPsec Dialup / Dynamic                                      |
| Topology     | Hub-and-Spoke + Dynamic Spoke-to-Spoke                      |
| Main benefit | Automatic tunnel discovery and reduced static configuration |

### Key Concepts

* Useful for **Hub-and-Spoke** environments.
* ADVPN dynamically discovers spoke-to-spoke paths.
* FortiGate can use **BGP as the built-in dynamic-routing mechanism**.
* IPsec tunnels are initially established between **spoke ↔ hub**.
* After discovery, traffic can dynamically use **spoke ↔ spoke** tunnels.
* ADVPN can be configured through the **Hub-and-Spoke / ADVPN wizard** or manually.
* Many ADVPN-related options are controlled from the **IPsec Phase 1 advanced network settings**.

---

# 2. ADVPN with Built-in Hub-and-Spoke Wizard

## Hub — FGT-HUB

### IPsec Phase 1

```bash
config vpn ipsec phase1-interface
    edit "advpn-hub"
        set type dynamic
        set interface "port9"
        set peertype any
        set net-device disable
        set proposal aes128-sha256 aes256-sha256 3des-sha256 aes128-sha1 aes256-sha1 3des-sha1
        set add-route disable
        set dpd on-idle
        set auto-discovery-sender enable
        set psksecret "sample"
        set dpd-retryinterval 5
    next
end
```

### IPsec Phase 2

```bash
config vpn ipsec phase2-interface
    edit "advpn-hub"
        set phase1name "advpn-hub"
        set proposal aes128-sha1 aes256-sha1 3des-sha1 aes128-sha256 aes256-sha256 3des-sha256
    next
end
```

### Hub ADVPN Parameters

```text
Tunnel IP:
    12.23.34.1

Remote/Tunnel IP:
    12.23.34.2/24

Local AS:
    64500

Local Interface:
    LAN

Local Subnet:
    192.168.101.0/24
```

### Spoke Definition

```text
Spoke Type:
    Range
    or
    Individual
```

Example:

```text
Spoke-1:
    Tunnel IP = 12.23.34.2

Spoke-2:
    Tunnel IP = 12.23.34.3
```

> The wizard can generate configuration keys that can be applied to spokes.

---

## Important ADVPN Hub Settings

| Setting                 | Hub              |
| ----------------------- | ---------------- |
| IPsec Type              | Dynamic          |
| Peer Type               | Any              |
| Auto-Discovery Sender   | Enable           |
| Auto-Discovery Receiver | Disable          |
| Device Creation         | Usually disable  |
| Net-Device              | Disable          |
| Add Route               | Disable          |
| DPD                     | On Idle          |
| Routing                 | BGP / OSPF / RIP |

### Why `net-device disable`?

With:

```text
net-device disable
```

FortiGate uses dynamically generated tunnel interfaces/tunnel IDs instead of creating a conventional interface for every dynamic peer.

> **Important:** Dynamic tunnel IDs are used as routing gateways for dynamically created tunnels.

---

# 3. ADVPN Spokes

Spokes use:

```text
Auto-Discovery Receiver = Enable
Device Creation = Enable
```

Conceptually:

```text
                  +-------------+
                  |    HUB      |
                  | ADVPN Sender|
                  +------+------+
                         |
             +-----------+-----------+
             |                       |
        +----+----+             +----+----+
        | Spoke-1 |             | Spoke-2 |
        +---------+             +---------+
             \                       /
              \_____ ADVPN ________/
                 Dynamic Tunnel
```

### Initial Connectivity

```text
Spoke-1 ---> Hub <--- Spoke-2
```

### After ADVPN Discovery

```text
Spoke-1 <----------> Spoke-2
       Dynamic Tunnel
```

---

# 4. ADVPN Routing Behavior

### Hub Networks

Hub networks are normally reachable through the hub tunnel IP.

Example:

```text
Hub LAN:
    192.168.101.0/24

Hub Tunnel:
    12.23.34.1
```

### Spoke-to-Spoke

A notable behavior in some configurations:

```text
Spoke-1
192.168.102.0/24
        |
        v
Spoke-2
192.168.103.0/24
```

The initial routing information may point toward the **spoke public address**, while ADVPN dynamically establishes the direct tunnel.

---

# 5. ADVPN with Different Autonomous Systems

If different AS numbers are required, avoid relying on static routes for the dynamic topology.

Example:

```text
Hub:
    AS 65001

Spoke-1:
    AS 65001

Spoke-2:
    AS 65001
```

### Spoke-1 BGP

```text
Neighbor:
    12.23.34.3
    Remote-AS 65001

Neighbor:
    12.23.34.1
    Remote-AS 65001
```

### Spoke-2 BGP

```text
Neighbor:
    12.23.34.2
    Remote-AS 65001

Neighbor:
    12.23.34.1
    Remote-AS 65001
```

---

# 6. ADVPN with Different AS Numbers

If the Hub and Spokes use different AS numbers:

```text
Hub:
    AS 65000

Spoke-1:
    AS 65001

Spoke-2:
    AS 65002
```

Consider enabling:

```text
Auto-Discovery Sender
Auto-Discovery Receiver
Device Creation
Exchange IP Address
Add Route = Disable
```

And configure:

```text
BGP Next-Hop-Self
```

### Important

```text
Different AS + ADVPN
        |
        +--> Enable ADVPN discovery
        |
        +--> Configure BGP next-hop-self
        |
        +--> Avoid unnecessary static routes
```

> These configurations are generally more useful in **large-scale deployments** rather than simple environments.

---

# 7. Manual ADVPN with Dynamic Routing

This approach does **not** rely on the Hub-and-Spoke wizard.

Supported routing examples:

```text
BGP
OSPF
RIP
```

---

# 8. Manual ADVPN + BGP

## Hub

### Default Route

```text
0.0.0.0/0
    |
    +--> ISP
```

### IPsec

```text
Type:
    Custom

Mode:
    Dialup

Interface:
    ipsec-dial-hs

Advanced Network:
    Auto-Discovery Sender = Enable

Phase 2:
    All-to-All

PFS:
    Disable

Auto-Negotiate:
    Enable
```

### IPsec Interface

```text
Local:
    12.23.34.1

Remote:
    12.23.34.2/24
```

### Policies

```text
Incoming:
    LAN + Dialup

Outgoing:
    LAN + Dialup

Source:
    All

Destination:
    All

Service:
    ALL

NAT:
    Disable

Logging:
    All Sessions
```

---

## Hub BGP

```text
Local-AS:
    65001

Router-ID:
    1.1.1.1
```

### Neighbor Group

```text
Neighbor Group:
    advpn-12

Remote-AS:
    65001

Interface:
    ipsec-dial-hs
```

### Recommended Attributes

```text
Route Reflector
Soft Reconfiguration
Link-Down Failover
```

Example:

```bash
config router bgp
    config neighbor-group
        edit "advpn-12"
            set link-down-failover enable
        next
    end
end
```

### Neighbor Range

```text
Prefix:
    12.23.34.0/24

Neighbor Group:
    advpn-12
```

### Advertised Network

```text
192.168.101.0/24
```

---

# 9. Manual ADVPN + BGP Spoke

## Spoke IPsec

```text
Type:
    Custom

Mode:
    Static

Interface:
    ipsec-dial-hs

Advanced Network:
    Auto-Discovery Receiver = Enable

Phase 2:
    All-to-All

PFS:
    Disable

Auto-Negotiate:
    Enable
```

### Interface

```text
Local:
    12.23.34.x

Remote:
    12.23.34.1/24
```

### Dual ISP Failover

```bash
config vpn ipsec phase1-interface
    edit "link-2"
        set monitor "link-1"
    next
end
```

### Policies

```text
Incoming:
    LAN + link-1 + link-2

Outgoing:
    LAN + link-1 + link-2

Source:
    All

Destination:
    All

Service:
    ALL

NAT:
    Disable

Logging:
    All Sessions
```

### Spoke BGP

```text
Local-AS:
    65001

Router-ID:
    x.x.x.x

Neighbor:
    12.23.34.1

Remote-AS:
    65001

Network:
    192.168.10x.0/24
```

Example:

```bash
config router bgp
    config neighbor
        edit "12.23.34.1"
            set link-down-failover enable
        next
    end
end
```

---

# 10. ADVPN + BGP Troubleshooting

If **spoke-to-spoke connectivity** does not work:

### Try enabling on Spokes

```text
Add Route = Enable
Device Creation = Enable
```

### Why?

```text
Spoke
  |
  +--> Auto-Discovery Receiver
  |
  +--> Learns dynamic tunnel information
  |
  +--> Creates/uses tunnel interfaces
  |
  +--> Spoke-to-Spoke connectivity
```

### Design trade-off

| Configuration       | Behavior                              |
| ------------------- | ------------------------------------- |
| Add Route OFF       | Cleaner dynamic routing               |
| Device Creation OFF | Less interface creation               |
| Add Route ON        | Better visibility of dynamic paths    |
| Device Creation ON  | Easier direct tunnel/routing behavior |

---

# 11. Why Disable Add-Route / Device Creation?

The basic ADVPN discovery process is:

```text
Spoke
   |
   | Discovery
   v
Hub
   |
   | Learn spoke information
   v
Spoke
```

The spoke first joins the Hub's discovery mechanism and retrieves the required tunnel/address information.

### If problems remain

Use:

```text
Add Route = Enable
Device Creation = Enable
```

on the spokes.

---

# 12. ADVPN + OSPF

## Hub

### IPsec

```text
Type:
    Custom

Mode:
    Dialup

Interface:
    ipsec-dial-hs

Advanced Network:
    Auto-Discovery Sender = Enable
    Device Creation = Enable

CLI alternative:
    net-device enable

Phase 2:
    All-to-All

PFS:
    Disable

Auto-Negotiate:
    Enable
```

### Interface

```text
Local:
    12.23.34.1

Remote:
    12.23.34.2/24
```

### OSPF

```text
Router-ID:
    1.1.1.1

Area:
    0.0.0.0

Networks:
    192.168.101.0/24
    12.23.34.0/24
```

### Policies

```text
Incoming:
    LAN + Dialup

Outgoing:
    LAN + Dialup

Source:
    All

Destination:
    All

Service:
    ALL

NAT:
    Disable

Logging:
    All Sessions
```

---

# 13. ADVPN + OSPF Spoke

## IPsec

```text
Type:
    Custom

Mode:
    Static

Interface:
    ipsec-dial-hs

Advanced Network:
    Auto-Discovery Receiver = Enable
    Device Creation = Enable

CLI:
    net-device enable

Phase 2:
    All-to-All

PFS:
    Disable

Auto-Negotiate:
    Enable
```

### Interface

```text
Local:
    12.23.34.x

Remote:
    12.23.34.1/24
```

### OSPF

```text
Router-ID:
    x.x.x.x

Area:
    0.0.0.0

Networks:
    192.168.10x.0/24
    12.23.34.0/24
```

### Dual ISP

```bash
config vpn ipsec phase1-interface
    edit "link-2"
        set monitor "link-1"
    next
end
```

---

# 14. ADVPN + RIP

## Important `net-device` Behavior

The Hub uses:

```text
Auto-Discovery Sender = Enable
Device Creation = Disable
```

### Why?

With:

```text
net-device enable
```

all advertised subnets can become reachable through the generated tunnel IDs.

This can behave more like physical routing and may create:

```text
Suboptimal routing
Routing loops
Unwanted paths
```

---

# 15. RIP Split Horizon

If Device Creation is enabled, use **Split Horizon**.

```bash
config router rip
    config interface
        edit "ipsec-dial-hs"
            set split-horizon-status enable
        next
    end
end
```

### Why?

```text
Spoke A
   |
   v
Hub
   |
   v
Spoke B
```

Without proper split-horizon behavior, RIP advertisements can be re-advertised through the same logical path.

### Rule

```text
RIP + Device Creation
        |
        +--> Enable Split Horizon
```

> The default behavior may already enable split horizon when the interface is explicitly added to RIP.

---

# 16. ADVPN + RIP Hub

### IPsec

```text
Type:
    Custom

Mode:
    Dialup

Interface:
    ipsec-dial-hs

Auto-Discovery Sender:
    Enable

Device Creation:
    Prefer Disable for basic RIP design

Phase 2:
    All-to-All

PFS:
    Disable

Auto-Negotiate:
    Enable
```

### Interface

```text
Local:
    12.23.34.1

Remote:
    12.23.34.2/24
```

### RIP

```text
Version:
    2

Networks:
    192.168.101.0/24
    12.23.34.0/24
```

---

# 17. ADVPN + RIP Spoke

### IPsec

```text
Type:
    Custom

Mode:
    Static

Interface:
    ipsec-dial-hs

Auto-Discovery Receiver:
    Enable

Phase 2:
    All-to-All

PFS:
    Disable

Auto-Negotiate:
    Enable
```

### RIP

```text
Version:
    2

Networks:
    192.168.10x.0/24
    12.23.34.0/24
```

### Split Horizon if Device Creation is Enabled

```bash
config router rip
    config interface
        edit "ipsec-dial-hs"
            set split-horizon-status enable
        next
    end
end
```

---

# 18. ADVPN Routing Decision Matrix

| Scenario        | Hub                 | Spoke                               |
| --------------- | ------------------- | ----------------------------------- |
| ADVPN Sender    | Enable              | Disable                             |
| ADVPN Receiver  | Disable             | Enable                              |
| Device Creation | Usually Disable     | Enable when required                |
| Add Route       | Usually Disable     | Enable when required                |
| BGP             | Recommended         | Recommended                         |
| OSPF            | Supported           | Supported                           |
| RIP             | Supported           | Supported                           |
| Split Horizon   | —                   | Required with RIP + Device Creation |
| Auto-Negotiate  | Enable              | Enable                              |
| PFS             | Disable in this lab | Disable in this lab                 |

---

# 19. BGP vs OSPF vs RIP

| Feature               |       BGP |   OSPF |     RIP |
| --------------------- | --------: | -----: | ------: |
| Large-scale ADVPN     |       ⭐⭐⭐ |     ⭐⭐ |       ⭐ |
| Fast convergence      |       ⭐⭐⭐ |    ⭐⭐⭐ |       ⭐ |
| Scalability           |       ⭐⭐⭐ |     ⭐⭐ |       ⭐ |
| Hub-and-Spoke         | Excellent |   Good |   Basic |
| Policy control        | Excellent |   Good | Limited |
| ADVPN suitability     | Excellent |   Good | Limited |
| Split Horizon concern |       Low | Medium |    High |

> For large-scale ADVPN deployments, **BGP is generally the stronger choice**.

---

# 20. ADVPN + Dual ISP Failover

### Spoke

```text
ISP-1
  |
  +--> link-1
  |
  +--> Primary
       AD = 10

ISP-2
  |
  +--> link-2
  |
  +--> Backup
       AD = 11
```

### IPsec Monitor

```bash
config vpn ipsec phase1-interface
    edit "link-2"
        set monitor "link-1"
    next
end
```

### Result

```text
link-1 UP
    |
    +--> Primary tunnel

link-1 DOWN
    |
    +--> link-2 becomes active
```

---

# 21. ADVPN Packet Flow

## Initial Connection

```text
          IPsec
Spoke-1 ---------> Hub
                    ^
                    |
                   IPsec
                    |
                  Spoke-2
```

## Discovery

```text
Spoke-1
   |
   | ADVPN Discovery
   v
 Hub
   |
   | Spoke information
   v
Spoke-2
```

## Dynamic Shortcut

```text
Spoke-1 ================= Spoke-2
             ADVPN
        Dynamic Tunnel
```

---

# 22. Quick Configuration Checklist

### Hub

```text
[ ] IPsec custom/dialup
[ ] Auto-Discovery Sender
[ ] Correct PSK
[ ] Correct IKE proposal
[ ] Correct Phase 2
[ ] Auto-Negotiate
[ ] Correct tunnel IP
[ ] LAN policies
[ ] NAT disabled
[ ] Dynamic routing configured
[ ] BGP/OSPF/RIP networks advertised
```

### Spoke

```text
[ ] IPsec custom/static
[ ] Auto-Discovery Receiver
[ ] Device Creation if required
[ ] Add Route if required
[ ] Correct PSK
[ ] Correct IKE proposal
[ ] Correct Phase 2
[ ] Auto-Negotiate
[ ] Tunnel IP
[ ] LAN policies
[ ] NAT disabled
[ ] Dynamic routing configured
[ ] Local LAN advertised
```

---

# 23. ADVPN Troubleshooting Checklist

### IPsec

```text
[ ] Phase 1 UP
[ ] Phase 2 UP
[ ] Correct PSK
[ ] Correct IKE version
[ ] Proposal matches
[ ] DH group matches
[ ] PFS matches
[ ] DPD working
[ ] NAT-T working
```

### ADVPN

```text
[ ] Hub = Auto-Discovery Sender
[ ] Spoke = Auto-Discovery Receiver
[ ] Correct tunnel IPs
[ ] Device Creation checked
[ ] Add Route checked if required
[ ] Spoke-to-Spoke discovery working
```

### Routing

```text
[ ] BGP/OSPF/RIP neighbor UP
[ ] Correct router ID
[ ] Correct AS number
[ ] Correct network statements
[ ] Correct next-hop
[ ] No unexpected static routes
[ ] No routing loop
[ ] RIP split horizon configured when required
```

### Policy

```text
[ ] LAN -> IPsec allowed
[ ] IPsec -> LAN allowed
[ ] Correct source
[ ] Correct destination
[ ] Correct service
[ ] NAT disabled
[ ] Logging enabled
```

---

# 24. Mental Model

```text
                    +----------------+
                    |      HUB       |
                    |                |
                    | ADVPN SENDER   |
                    | BGP / OSPF/RIP |
                    +-------+--------+
                            |
              +-------------+-------------+
              |                           |
        +-----+-----+               +-----+-----+
        |  SPOKE-1  |               |  SPOKE-2  |
        |            |               |            |
        | ADVPN RX   |               | ADVPN RX   |
        +-----+------+               +------+-----+
              \                           /
               \                         /
                \==== ADVPN SHORTCUT ===/
```

### Core Rule

```text
HUB
    Auto-Discovery Sender

SPOKE
    Auto-Discovery Receiver

Routing
    BGP > preferred for large-scale
    OSPF > useful alternative
    RIP  > simple/small deployments

Spoke-to-Spoke
    ADVPN dynamically discovers shortcut tunnels
```

---

# 25. Important Caveats

> ⚠️ **Do not blindly enable every advanced IPsec option.**
> `add-route`, `device-creation`, `net-device`, and ADVPN discovery options change how FortiGate represents and routes dynamic tunnels.

> ⚠️ **RIP requires special attention to split horizon** when dynamic tunnel/device creation is used.

> ⚠️ **Static routes can interfere with the intended dynamic-routing design.**

> ⚠️ **Spoke-to-spoke failure does not necessarily mean Phase 1/Phase 2 is broken.** Check ADVPN discovery, routing, next-hop information, and dynamic tunnel creation separately.

> ⚠️ **Always match IPsec proposals between peers.** IKE/Phase 2 parameters such as encryption, authentication, DH/PFS, and negotiation behavior must be compatible.

---

## One-Line Summary

```text
ADVPN = IPsec Dynamic Discovery + Hub/Spoke + Dynamic Routing + Optional Spoke-to-Spoke Shortcut
```

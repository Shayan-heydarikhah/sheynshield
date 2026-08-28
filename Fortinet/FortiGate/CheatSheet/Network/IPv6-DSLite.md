# IPv6 Tunneling & Encapsulation

> **FortiOS 7.2.0 | IPv6 | Dual Stack | 6in4 | 4in6 | DS-Lite | MAP-E | VNE | IPv6 Routing**

---

## 🌐 IPv6 — Think in Encapsulation

When working with IPv6 transition technologies, don't start with the command.

Start with one question:

> **What is the original packet, what infrastructure exists between the endpoints, and what packet is used to carry the original packet?**

The core concept is **encapsulation**.

```text
Original Packet
      │
      ▼
┌───────────────┐
│ IPv6 Packet   │
└───────────────┘
      │
      ▼
   Encapsulate
      │
      ▼
┌───────────────┐
│ IPv4 Packet   │
│   IPv6 Data   │
└───────────────┘
      │
      ▼
 IPv4 Infrastructure
      │
      ▼
   Decapsulate
      │
      ▼
┌───────────────┐
│ IPv6 Packet   │
└───────────────┘
```

### 🧠 Routing Mindset

Always identify:

1. **Original address family**
2. **Transport/infrastructure address family**
3. **Tunnel endpoints**
4. **Tunnel interface**
5. **Routes pointing to the tunnel**
6. **Firewall policies**
7. **NAT requirements**
8. **Return path**

---

# 🔄 Why Dual Stack?

For enterprise networks, a practical IPv6 transition strategy is to support:

```text
IPv4 + IPv6
     │
     ▼
Dual Stack
```

A dual-stack device can process both:

```text
IPv4
IPv6
```

This allows IPv6 traffic to coexist with existing IPv4 infrastructure.

> 💡 **Dual Stack is the foundation. Tunneling is a transition mechanism used when the underlying network cannot directly transport the required address family.**

---

# 🧩 IPv6 Tunnel Models

There are four useful ways to think about where the tunnel exists in the path.

---

## 1️⃣ Network Device → Network Device

Both network devices are dual-stack capable.

The infrastructure between them is IPv4.

```text
IPv6 Network
     │
     ▼
[IPv6 Device]
     │
     │ IPv6 over IPv4 Tunnel
     ▼
========== IPv4 ==========
     │
     ▼
[IPv6 Device]
     │
     ▼
IPv6 Network
```

### Tunnel Scope

The tunnel spans **one segment of the path**.

```text
IPv6 Path:

Device A
   │
   ├── Tunnel ───────────────┐
   │                         │
 IPv4 Infrastructure         │
   │                         │
   └─────────────────────────┘
                             │
                          Device B
```

---

# 2️⃣ Host → Network Device

A dual-stack host tunnels IPv6 packets to an intermediary IPv6/IPv4 network device.

```text
[Dual-Stack Host]
        │
        │ IPv6
        ▼
   IPv6 over IPv4
        │
        ▼
========== IPv4 ==========
        │
        ▼
[Network Device]
        │
        ▼
      IPv6
```

### Tunnel Scope

The tunnel spans the **first segment** of the IPv6 path.

---

# 3️⃣ Host → Host

Both hosts are dual-stack capable and communicate through an IPv4 infrastructure.

```text
[IPv6 Host A]
      │
      │
      ▼
 IPv6 over IPv4
      │
      ▼
=========== IPv4 ===========
      │
      ▼
 IPv6 over IPv4
      │
      ▼
[IPv6 Host B]
```

### Tunnel Scope

The tunnel spans the **entire path** between the hosts.

---

# 4️⃣ Network Device → Host

A dual-stack network device tunnels IPv6 traffic toward the final host.

```text
[Network Device]
       │
       │ IPv6 over IPv4
       ▼
========== IPv4 ==========
       │
       ▼
[IPv6 / IPv4 Host]
```

### Tunnel Scope

The tunnel covers only the **last segment** of the IPv6 path.

---

# 🧠 Tunnel Scope — Quick Table

| Model                           | Tunnel Scope  |
| ------------------------------- | ------------- |
| Network Device → Network Device | One segment   |
| Host → Network Device           | First segment |
| Host → Host                     | Entire path   |
| Network Device → Host           | Last segment  |

---

# 🔵 6in4

## What Is 6in4?

**6in4** carries IPv6 packets through an IPv4 infrastructure.

```text
IPv6
  │
  ▼
[FortiGate 1]
  │
  │ IPv6 over IPv4
  ▼
========= IPv4 =========
  │
  ▼
[FortiGate 2]
  │
  ▼
IPv6
```

Conceptually:

```text
IPv6 Packet
     │
     ▼
Encapsulate
     │
     ▼
IPv4 Transport
     │
     ▼
Decapsulate
     │
     ▼
IPv6 Packet
```

---

# 🔧 6in4 — SIT Tunnel

FortiGate uses a **SIT (Simple Internet Transition)** tunnel for this type of IPv6-over-IPv4 configuration.

### Example Topology

```text
IPv6 LAN
   │
   ▼
FGT-1
   │
   │ 6in4 / SIT
   │
   ▼
IPv4 Infrastructure
   │
   ▼
FGT-2
   │
   ▼
IPv6 LAN
```

---

## ⚙️ Configure 6in4

```bash
config system sit-tunnel
    edit 6in4
        set source 12.12.12.1
        set destination 12.12.12.2
        set ip6 2001:fd:1212::1/64
        set interface port4
    end
end
```

### Parameters

| Parameter     | Purpose                             |
| ------------- | ----------------------------------- |
| `source`      | Local IPv4 tunnel endpoint          |
| `destination` | Remote IPv4 tunnel endpoint         |
| `ip6`         | IPv6 address assigned to the tunnel |
| `interface`   | Underlying interface                |

---

# 🧠 6in4 Addressing

Example:

```text
IPv4 Tunnel Endpoint

FGT-1
12.12.12.1

        IPv4 Infrastructure

FGT-2
12.12.12.2
```

Tunnel IPv6:

```text
FGT-1
2001:fd:1212::1/64
```

Think of it as two address families:

```text
Outer / Transport
        │
        ▼
      IPv4

Inner / Payload
        │
        ▼
      IPv6
```

---

# 🛣️ 6in4 Routing

After creating the tunnel, routes must point IPv6 traffic toward the tunnel interface.

Example:

```text
192.168.102.0/24 → 6in4
2001:fd:102::/64 → 6in4
```

Conceptually:

```text
IPv6 Destination
       │
       ▼
Routing Table
       │
       ▼
    6in4
       │
       ▼
IPv4 Infrastructure
       │
       ▼
Remote FortiGate
```

> ⚠️ Creating the tunnel alone does **not** complete the routing design. The required IPv4/IPv6 routes and firewall policies must also point to the tunnel interface.

---

# 🔐 6in4 Firewall Policies

The tunnel interface must be considered in the firewall policy design.

```text
IPv6 LAN
   │
   ▼
Policy
   │
   ▼
6in4
   │
   ▼
Remote IPv6 Network
```

Update policies whenever the tunnel interface becomes part of the forwarding path.

---

# 🖥️ Client Configuration

Clients participating in the IPv6 network must have appropriate:

```text
IPv4
+
IPv6
```

configuration.

The 6in4 interface can also participate in routing and provide connectivity toward other services depending on the design.

---

# 🟣 4in6

## What Is 4in6?

**4in6** carries IPv4 traffic through an IPv6 infrastructure.

This is the reverse encapsulation mindset:

```text
IPv4 Packet
     │
     ▼
Encapsulate
     │
     ▼
IPv6 Transport
     │
     ▼
IPv6 Infrastructure
     │
     ▼
Decapsulate
     │
     ▼
IPv4 Packet
```

---

# 🔧 4in6 Configuration

```bash
config system ipv6-tunnel
    edit 4in6
        set source 2001:fd8:12::1
        set destination 2001:fd8:12::2
        set interface port4
    end
end
```

---

# 🧠 4in6 Topology

```text
IPv4 Network
     │
     ▼
   FGT-1
     │
     │ IPv4 over IPv6
     ▼
========== IPv6 ==========
     │
     ▼
   FGT-2
     │
     ▼
IPv4 Network
```

---

# 🖥️ 4in6 Tunnel Interface Addressing

An IPv4 address can be assigned to the tunnel interface.

Example:

```text
Local:
192.168.12.1

Remote:
192.168.12.2/30
```

The tunnel itself uses IPv6 transport:

```text
Local IPv6
2001:fd8:12::1

Remote IPv6
2001:fd8:12::2
```

So we have:

```text
Tunnel Transport
        │
        ▼
      IPv6

Tunnel Payload
        │
        ▼
      IPv4
```

---

# 🛣️ 4in6 Routing

Both IPv4 and IPv6 routes can use the tunnel interface where appropriate.

```text
IPv4 Route
    │
    ▼
  4in6
    │
    ▼
IPv6 Infrastructure
```

and:

```text
IPv6 Route
    │
    ▼
  4in6
```

---

# 🔐 4in6 Firewall Policy

The tunnel interface must be included in the appropriate firewall policies.

```text
IPv4 LAN
   │
   ▼
Firewall Policy
   │
   ▼
  4in6
   │
   ▼
Remote IPv4 Network
```

> 💡 Dynamic routing can also be considered where supported by the overall topology.

### NAT Consideration

For tunnel-based inter-site connectivity, avoid unnecessary NAT.

```text
Site A
192.168.x.x
     │
     ▼
   4in6
     │
     ▼
Site B
192.168.y.y
```

The goal is normally to preserve the original addressing between sites unless NAT is explicitly required by the design.

---

# 🟠 DS-Lite

## Dual-Stack Lite

**DS-Lite** allows IPv4 applications to reach IPv4 destinations across an IPv6-only access network.

```text
IPv4 Application
       │
       ▼
FortiGate
       │
       │ IPv4 over IPv6
       ▼
========= IPv6 =========
       │
       ▼
ISP AFTR / BR
       │
       ▼
IPv4 Internet
```

---

# 🧠 Why DS-Lite?

ISPs may use DS-Lite when they do not have enough public IPv4 addresses for customers.

The customer receives IPv6 connectivity while IPv4 applications still need Internet access.

```text
Customer
   │
   ├── IPv6 Internet
   │
   └── IPv4 Applications
             │
             ▼
          DS-Lite
             │
             ▼
         IPv4 Internet
```

---

# 📦 DS-Lite Encapsulation

The key idea:

```text
Original
IPv4 Packet
     │
     ▼
Encapsulate
     │
     ▼
IPv6 Packet
     │
     ▼
ISP IPv6 Network
     │
     ▼
AFTR / Border Relay
     │
     ▼
Decapsulate
     │
     ▼
IPv4 Internet
```

---

# 🌐 ISP Components

A DS-Lite deployment can involve:

* AFTR
* 4over6 Border Relay
* DHCPv6
* Router Solicitation / Router Advertisement
* DNS
* IPv6 connectivity

The client receives IPv6 parameters through the IPv6 access network.

---

# 🧩 MAP-E / VNE Configuration Concept

The FortiGate can use **VNE (Virtual Network Enabler)** functionality for IPv4-over-IPv6 transition services.

The provider can supply information through a configuration service.

Example information includes:

```text
PSID
BR IPv6 Address
IPv6 Prefix
PSID Length
BMR / BR Hostname
```

---

# 🗺️ DS-Lite / MAP-E Logical Flow

```text
                CUSTOMER
                    │
                    ▼
               FortiGate
                    │
             IPv4 Application
                    │
                    ▼
               VNE Tunnel
                    │
                    ▼
              IPv6 Network
                    │
                    ▼
            ISP Border Relay
                    │
                    ▼
              IPv4 Internet
```

---

# 🏢 FortiGate — VNE Example

Example configuration:

```bash
config system vne
    set status enable
    set interface port2
    set ssl-certificate Fortinet_Factory
    set ipv4-address 192.168.12.1 255.255.255.0
    set br 2001:fd8:12::1
    set update-url http://192.168.20.200/map-e-config.xml
    set mode fixed-ip
    set http-username <username>
    set http-password <password>
end
```

> ⚠️ Replace example addresses, credentials and URLs with values from the actual ISP/VNE deployment.

---

# 🔌 VNE Tunnel Interface

The VNE tunnel can have both IPv4 and IPv6 addressing.

Example:

```text
IPv4:

Local
192.168.12.1

Remote
192.168.12.2/30
```

IPv6:

```text
2001:fd8:1212::1/128
```

Conceptually:

```text
VNE Tunnel
├── IPv4 Address
│
└── IPv6 Address
```

---

# 🛣️ VNE Routing

Static routes can point to the VNE tunnel.

Example:

```text
192.168.102.0/24 → VNE

2001:fd8:102::/64 → VNE
```

Conceptually:

```text
IPv4 Destination
       │
       ▼
      VNE
       │
       ▼
IPv6 ISP Infrastructure
```

---

# 🔐 VNE Firewall Policies

Update firewall policies so the VNE interface is included in the required forwarding paths.

```text
LAN
 │
 ▼
Policy
 │
 ▼
VNE
 │
 ▼
ISP
```

---

# 🔄 Changing the Border Relay

If the remote Border Relay IPv6 address changes, the VNE configuration can be updated accordingly.

Example:

```bash
config system vne
    set status enable
    set br 2001:fd8:12::2
end
```

The important concept is:

```text
Customer VNE
      │
      ▼
BR IPv6 Address
      │
      ▼
ISP Border Relay
```

---

# 🌐 Branch VNE Example

Example branch configuration:

```bash
config system vne
    set status enable
    set interface port2
    set ssl-certificate Fortinet_Factory
    set ipv4-address 192.168.12.2 255.255.255.0
    set br map-e-config.test.com
    set update-url http://192.168.20.200/map-e-config.xml
    set mode fixed-ip
    set http-username <username>
    set http-password <password>
end
```

---

# 🧭 Branch Routing to Configuration Server

If the configuration service is hosted at:

```text
192.168.20.200
```

the branch needs appropriate reachability toward the DNS/configuration service.

Example conceptual routes:

```text
192.168.20.200
        │
        ▼
   ISP Gateway
```

and:

```text
0.0.0.0/0
     │
     ▼
    VNE
```

---

# 📄 MAP-E Configuration File

The provider/configuration server can expose a configuration file such as:

```text
map-e-config.xml
```

Example:

```xml
<map-e-config>
    <prefix>2001:fd8:100::/64</prefix>
    <psid-offset>6</psid-offset>
    <psid-length>8</psid-length>
    <br-ipv6>2001:fd8:12::1</br-ipv6>
    <bmr-hostname>map-e-config.test.com</bmr-hostname>
</map-e-config>
```

### Information Retrieved

| Field          | Purpose                   |
| -------------- | ------------------------- |
| `prefix`       | IPv6 prefix               |
| `psid-offset`  | PSID offset               |
| `psid-length`  | PSID length               |
| `br-ipv6`      | Border Relay IPv6 address |
| `bmr-hostname` | Border Relay hostname     |

---

# 🌐 MAP-E BMR Hostname

The FortiGate can use a hostname for the Border Relay.

Example:

```bash
config system vne
    set mode map-e
    set bmr-hostname map-e-config.test.com
end
```

The hostname must resolve to the appropriate IPv6 Border Relay address.

---

# 🧩 DNS AAAA Record

Example DNS record:

```text
map-e-config.test.com
        │
        ▼
AAAA
        │
        ▼
2001:fd8:12::1
```

Conceptually:

```text
FortiGate
   │
   │ DNS Query
   ▼
DNS Server
   │
   ▼
AAAA Record
   │
   ▼
IPv6 BR Address
```

---

# 🖥️ IIS Configuration Server

A Windows IIS server can host the MAP-E configuration file.

Example path:

```text
C:\inetpub\wwwroot\map-e-config.xml
```

The FortiGate retrieves the configuration using the configured update URL.

```text
FortiGate
    │
    │ HTTP Request
    ▼
IIS
    │
    ▼
map-e-config.xml
```

---

# 🔍 VNE Troubleshooting

Use:

```bash
diagnose test application vned 1
```

Check:

* VNE state
* Configuration retrieval
* Border Relay information
* Tunnel status
* Configuration errors

---

# 🧠 IPv6 Transition Technologies — Compare

| Technology  | Payload | Transport |
| ----------- | ------- | --------- |
| **6in4**    | IPv6    | IPv4      |
| **4in6**    | IPv4    | IPv6      |
| **DS-Lite** | IPv4    | IPv6      |
| **MAP-E**   | IPv4    | IPv6      |

### Easy Memory Trick

```text
6in4
6 → inside
4 → outside

4in6
4 → inside
6 → outside
```

Think:

> **First number = payload**
> **Second number = transport**

---

# 🔥 Encapsulation Cheat Sheet

## 6in4

```text
┌─────────────────────────────┐
│ IPv4 Header                 │
│ ┌─────────────────────────┐ │
│ │ IPv6 Packet             │ │
│ └─────────────────────────┘ │
└─────────────────────────────┘
```

```text
IPv6 over IPv4
```

---

## 4in6

```text
┌─────────────────────────────┐
│ IPv6 Header                 │
│ ┌─────────────────────────┐ │
│ │ IPv4 Packet             │ │
│ └─────────────────────────┘ │
└─────────────────────────────┘
```

```text
IPv4 over IPv6
```

---

## DS-Lite

```text
┌─────────────────────────────┐
│ IPv6 Header                 │
│ ┌─────────────────────────┐ │
│ │ IPv4 Packet             │ │
│ └─────────────────────────┘ │
└─────────────────────────────┘
```

```text
IPv4 Application
      ↓
IPv4 Packet
      ↓
IPv6 Encapsulation
      ↓
ISP
      ↓
AFTR
      ↓
IPv4 Internet
```

---

# 🧭 IPv6 Routing Mindset

When troubleshooting an IPv6 tunnel, never check only the tunnel.

Use this sequence:

```text
              IPv6 Tunnel
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
     Endpoint   Routing    Policy
        │          │          │
        ▼          ▼          ▼
     Address    Next-Hop    Firewall
        │          │          │
        └──────────┼──────────┘
                   ▼
              Underlay
                   │
                   ▼
          IPv4 / IPv6 Network
```

---

# 🛠️ Troubleshooting Checklist

## 1. Check Underlay

```text
Can the tunnel endpoints reach each other?
```

For 6in4:

```text
IPv4 Endpoint A
       │
       ▼
IPv4 Infrastructure
       │
       ▼
IPv4 Endpoint B
```

For 4in6:

```text
IPv6 Endpoint A
       │
       ▼
IPv6 Infrastructure
       │
       ▼
IPv6 Endpoint B
```

---

## 2. Check Tunnel Endpoints

Verify:

```text
Source
Destination
Tunnel Interface
```

---

## 3. Check Tunnel Addressing

Verify both sides have compatible tunnel addressing.

```text
Local
Remote
Prefix
```

---

## 4. Check Routing

Ask:

> **Where does the FortiGate send the destination prefix?**

```text
Destination
     │
     ▼
Routing Table
     │
     ▼
Tunnel Interface
```

---

## 5. Check Firewall Policy

Ask:

```text
LAN → Tunnel ?
Tunnel → LAN ?
```

The tunnel interface must be referenced in the appropriate policies.

---

## 6. Check NAT

For site-to-site tunnel designs:

```text
NAT
 │
 └── Usually avoid unless explicitly required
```

---

## 7. Check Return Path

A working outbound tunnel does not guarantee bidirectional connectivity.

```text
Site A
  │
  ▼
Tunnel
  │
  ▼
Site B
  │
  ▼
Return Route
  │
  ▼
Site A
```

---

# ⚡ Quick Reference — Commands & Concepts

### 6in4

```bash
config system sit-tunnel
    edit 6in4
        set source <IPv4>
        set destination <IPv4>
        set ip6 <IPv6>/<prefix>
        set interface <interface>
    end
end
```

### 4in6

```bash
config system ipv6-tunnel
    edit 4in6
        set source <IPv6>
        set destination <IPv6>
        set interface <interface>
    end
end
```

### VNE

```bash
config system vne
    set status enable
    set interface <interface>
    set ipv4-address <IPv4> <mask>
    set br <BR>
    set update-url <URL>
end
```

### VNE Diagnostic

```bash
diagnose test application vned 1
```

---

# 🎯 NSE Memory Map

| If You See...                           | Think...                              |
| --------------------------------------- | ------------------------------------- |
| IPv6 over IPv4                          | **6in4 / SIT**                        |
| IPv4 over IPv6                          | **4in6**                              |
| IPv4 applications over IPv6 ISP         | **DS-Lite**                           |
| IPv4 Internet + IPv6 access network     | **DS-Lite / MAP-E concepts**          |
| Border Relay                            | **BR / AFTR**                         |
| `server-list` / configuration retrieval | **VNE configuration service**         |
| `psid`                                  | **MAP-E**                             |
| `bmr-hostname`                          | **MAP-E Border Relay discovery**      |
| AAAA record                             | **IPv6 hostname resolution**          |
| `update-url`                            | **Retrieve transition configuration** |
| `sit-tunnel`                            | **6in4**                              |
| `ipv6-tunnel`                           | **4in6**                              |

---

# 🔥 Key Takeaways

> **1. IPv6 tunneling is fundamentally an encapsulation problem: identify the payload and the transport.**

> **2. In `6in4`, IPv6 is the payload and IPv4 is the transport.**

> **3. In `4in6`, IPv4 is the payload and IPv6 is the transport.**

> **4. A tunnel interface does not automatically provide end-to-end connectivity; routing and firewall policies still matter.**

> **5. Always verify the tunnel underlay before troubleshooting the overlay.**

> **6. Source and destination tunnel endpoints are critical because they define where encapsulation and decapsulation occur.**

> **7. DS-Lite allows IPv4 applications to operate across an IPv6 access network.**

> **8. In DS-Lite, the ISP uses an AFTR/Border Relay function to process the encapsulated IPv4 traffic.**

> **9. MAP-E relies on parameters such as IPv6 prefix, PSID information and Border Relay information.**

> **10. DNS can be part of the Border Relay discovery process when a hostname is used.**

> **11. For site-to-site tunnel designs, always check the return route.**

> **12. Think about IPv6 as a routing architecture first and a configuration syntax second.**

---

## 🔑 Keywords

`IPv6` · `IPv6 Routing` · `IPv6 Tunneling` · `IPv6 Encapsulation` · `FortiGate IPv6` · `FortiOS 7.2.0` · `Dual Stack` · `6in4` · `SIT Tunnel` · `4in6` · `DS-Lite` · `Dual-Stack Lite` · `MAP-E` · `VNE` · `Virtual Network Enabler` · `AFTR` · `Border Relay` · `IPv4 over IPv6` · `IPv6 over IPv4` · `PSID` · `BMR` · `IPv6 Transition Technologies` · `FortiGate Tunnel` · `IPv6 Firewall Policy` · `IPv6 Static Route`

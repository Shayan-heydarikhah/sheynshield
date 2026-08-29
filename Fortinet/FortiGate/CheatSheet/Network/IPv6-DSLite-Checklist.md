# FortiGate IPv6 Tunneling & Encapsulation Checklist

> **FortiOS 7.2.0 | IPv6 | Dual Stack | 6in4 | 4in6 | DS-Lite | MAP-E | VNE | IPv6 Routing**
>
> A practical **FortiGate IPv6 tunneling and transition technology checklist** for configuration, validation, troubleshooting, and NSE-level review.

---

## 📌 Table of Contents

* [IPv6 Tunnel Design Checklist](#-ipv6-tunnel-design-checklist)
* [Dual-Stack Checklist](#-dual-stack-checklist)
* [Tunnel Model Checklist](#-tunnel-model-checklist)
* [6in4 / SIT Checklist](#-6in4--sit-checklist)
* [4in6 Checklist](#-4in6-checklist)
* [DS-Lite Checklist](#-ds-lite-checklist)
* [MAP-E / VNE Checklist](#-map-e--vne-checklist)
* [IPv6 Routing Checklist](#-ipv6-routing-checklist)
* [Firewall Policy Checklist](#-firewall-policy-checklist)
* [NAT Checklist](#-nat-checklist)
* [Underlay / Overlay Troubleshooting](#-underlay--overlay-troubleshooting)
* [VNE Troubleshooting](#-vne-troubleshooting)
* [Verification Commands](#-verification-commands)
* [NSE Memory Map](#-nse-memory-map)
* [Quick Decision Matrix](#-quick-decision-matrix)
* [Final Validation Checklist](#-final-validation-checklist)

---

# 🌐 IPv6 Tunnel Design Checklist

## 1. Identify the Packet

Before configuring anything, answer:

* [ ] What is the **original packet**?
* [ ] Is the payload IPv4?
* [ ] Is the payload IPv6?
* [ ] What address family exists in the underlay?
* [ ] What address family is used as transport?
* [ ] Where does encapsulation occur?
* [ ] Where does decapsulation occur?

### Core Memory Rule

```text
FIRST number  = payload
SECOND number = transport
```

Therefore:

```text
6in4 = IPv6 inside IPv4
4in6 = IPv4 inside IPv6
```

---

## 2. Identify Tunnel Endpoints

Verify:

* [ ] Local tunnel endpoint identified
* [ ] Remote tunnel endpoint identified
* [ ] Underlay interface identified
* [ ] Tunnel interface identified
* [ ] Tunnel source reachable
* [ ] Tunnel destination reachable
* [ ] Return path available

---

## 3. Identify the Complete Forwarding Path

```text
Client
   │
   ▼
FortiGate
   │
   ▼
Routing Table
   │
   ▼
Tunnel Interface
   │
   ▼
Underlay
   │
   ▼
Remote Tunnel Endpoint
   │
   ▼
Remote Network
```

Validate:

* [ ] Client routing
* [ ] FortiGate routing
* [ ] Tunnel routing
* [ ] Remote routing
* [ ] Return routing
* [ ] Firewall policy
* [ ] NAT behavior

---

# 🔄 Dual-Stack Checklist

Dual Stack allows both address families to operate simultaneously.

```text
IPv4
 +
IPv6
 =
Dual Stack
```

Validate:

* [ ] IPv4 addressing configured
* [ ] IPv6 addressing configured
* [ ] IPv4 routing operational
* [ ] IPv6 routing operational
* [ ] IPv4 DNS behavior verified
* [ ] IPv6 DNS behavior verified
* [ ] Firewall policies support both address families
* [ ] Applications have IPv6 support where required

### Design Principle

> **Dual Stack is an IPv6 transition foundation; tunneling is used when the existing infrastructure cannot directly transport the required protocol.**

---

# 🧩 Tunnel Model Checklist

Identify where the tunnel exists.

| Model                           | Tunnel Scope        | Validation |
| ------------------------------- | ------------------- | ---------- |
| Network Device → Network Device | One network segment | [ ]        |
| Host → Network Device           | First segment       | [ ]        |
| Host → Host                     | Entire path         | [ ]        |
| Network Device → Host           | Last segment        | [ ]        |

---

# 🔵 6in4 / SIT Checklist

## 6in4 Concept

```text
IPv6 Payload
     │
     ▼
IPv4 Encapsulation
     │
     ▼
IPv4 Infrastructure
     │
     ▼
Decapsulation
     │
     ▼
IPv6 Payload
```

Validate:

* [ ] IPv6 is the payload
* [ ] IPv4 is the transport
* [ ] Both endpoints support IPv6
* [ ] IPv4 underlay exists
* [ ] IPv4 endpoint reachability verified
* [ ] SIT tunnel configured
* [ ] IPv6 tunnel address configured
* [ ] IPv6 routes configured
* [ ] Firewall policies configured
* [ ] Return path verified

---

## 6in4 CLI Template

```bash
config system sit-tunnel
    edit 6in4
        set source <LOCAL_IPV4>
        set destination <REMOTE_IPV4>
        set ip6 <TUNNEL_IPV6>/<PREFIX>
        set interface <UNDERLAY_INTERFACE>
    end
end
```

### Parameter Checklist

| Parameter     | Verify                         |
| ------------- | ------------------------------ |
| `source`      | [ ] Local IPv4 endpoint        |
| `destination` | [ ] Remote IPv4 endpoint       |
| `ip6`         | [ ] Tunnel IPv6 address/prefix |
| `interface`   | [ ] Correct IPv4 underlay      |

---

## 6in4 Encapsulation

```text
┌─────────────────────────────┐
│ IPv4 Header                 │
│                             │
│   ┌─────────────────────┐   │
│   │ IPv6 Packet         │   │
│   └─────────────────────┘   │
│                             │
└─────────────────────────────┘
```

Remember:

```text
Outer Header = IPv4
Inner Packet = IPv6
```

---

## 6in4 Routing Checklist

* [ ] Remote IPv6 network has a route
* [ ] Route points to SIT interface
* [ ] Local IPv6 network route is correct
* [ ] Remote FortiGate has return route
* [ ] IPv4 underlay route exists
* [ ] No accidental routing loop
* [ ] Firewall policy references tunnel interface

---

# 🟣 4in6 Checklist

## 4in6 Concept

```text
IPv4 Payload
     │
     ▼
IPv6 Encapsulation
     │
     ▼
IPv6 Infrastructure
     │
     ▼
Decapsulation
     │
     ▼
IPv4 Payload
```

Validate:

* [ ] IPv4 is the payload
* [ ] IPv6 is the transport
* [ ] IPv6 underlay exists
* [ ] IPv6 endpoints are reachable
* [ ] IPv4 tunnel addressing is configured
* [ ] IPv4 routes point to tunnel
* [ ] Firewall policies are configured
* [ ] Return routing is verified

---

## 4in6 CLI Template

```bash
config system ipv6-tunnel
    edit 4in6
        set source <LOCAL_IPV6>
        set destination <REMOTE_IPV6>
        set interface <UNDERLAY_INTERFACE>
    end
end
```

---

## 4in6 Addressing Checklist

### IPv6 Transport

```text
Local:
<LOCAL_IPV6>

Remote:
<REMOTE_IPV6>
```

### IPv4 Payload / Tunnel Network

```text
Local:
<LOCAL_IPV4>

Remote:
<REMOTE_IPV4>/<PREFIX>
```

Verify:

* [ ] IPv6 transport addresses are reachable
* [ ] IPv4 tunnel addresses are compatible
* [ ] Remote IPv4 routes use the tunnel
* [ ] Return route exists

---

## 4in6 NAT Checklist

For site-to-site connectivity:

* [ ] NAT requirement reviewed
* [ ] NAT disabled unless explicitly required
* [ ] Original source addressing preserved where possible
* [ ] Return routing verified

---

# 🟠 DS-Lite Checklist

## DS-Lite Architecture

DS-Lite allows IPv4 applications to operate across an IPv6 access network.

```text
IPv4 Application
       │
       ▼
FortiGate
       │
       │ IPv4 over IPv6
       ▼
================ IPv6 ================
       │
       ▼
ISP AFTR / Border Relay
       │
       ▼
IPv4 Internet
```

Validate:

* [ ] Customer has IPv6 connectivity
* [ ] IPv4 applications require Internet access
* [ ] IPv6-only access infrastructure identified
* [ ] DS-Lite service provided by ISP
* [ ] AFTR identified
* [ ] Border Relay information verified
* [ ] IPv6 reachability to provider infrastructure verified
* [ ] IPv4 application connectivity tested

---

## DS-Lite Encapsulation

```text
┌─────────────────────────────┐
│ IPv6 Header                 │
│                             │
│   ┌─────────────────────┐   │
│   │ IPv4 Packet         │   │
│   └─────────────────────┘   │
│                             │
└─────────────────────────────┘
```

Memory:

```text
IPv4
  ↓
IPv6 encapsulation
  ↓
AFTR
  ↓
IPv4 Internet
```

---

# 🧩 MAP-E / VNE Checklist

## Provider Information

Confirm that the provider supplies the required transition parameters:

* [ ] IPv6 prefix
* [ ] PSID
* [ ] PSID length
* [ ] PSID offset
* [ ] Border Relay IPv6 address
* [ ] BMR information
* [ ] BMR hostname
* [ ] Configuration/update URL

---

## VNE Configuration

```bash
config system vne
    set status enable
    set interface <INTERFACE>
    set ssl-certificate <CERTIFICATE>
    set ipv4-address <IPv4> <MASK>
    set br <BR>
    set update-url <URL>
    set mode <MODE>
    set http-username <USERNAME>
    set http-password <PASSWORD>
end
```

Validate:

* [ ] VNE enabled
* [ ] Correct interface selected
* [ ] IPv4 address configured
* [ ] Border Relay configured
* [ ] Update URL reachable
* [ ] Authentication configured if required
* [ ] Correct VNE mode selected
* [ ] Provider parameters match configuration

> ⚠️ Never commit real credentials, certificates, API tokens, PSKs, or private provider configuration to a public repository.

---

# 🧭 MAP-E Configuration Checklist

### Configuration File

Example:

```text
map-e-config.xml
```

Verify:

* [ ] Configuration server reachable
* [ ] XML/file available
* [ ] Prefix correct
* [ ] PSID offset correct
* [ ] PSID length correct
* [ ] Border Relay address correct
* [ ] BMR hostname correct

Example:

```xml
<map-e-config>
    <prefix>2001:db8:100::/64</prefix>
    <psid-offset>6</psid-offset>
    <psid-length>8</psid-length>
    <br-ipv6>2001:db8:12::1</br-ipv6>
    <bmr-hostname>map-e.example.com</bmr-hostname>
</map-e-config>
```

---

# 🌐 DNS / AAAA Checklist

If Border Relay discovery uses a hostname:

* [ ] BMR hostname configured
* [ ] DNS server reachable
* [ ] AAAA record exists
* [ ] AAAA record returns correct IPv6 address
* [ ] FortiGate can resolve the hostname
* [ ] Returned IPv6 address matches provider information

Example:

```text
map-e.example.com
        │
        ▼
       AAAA
        │
        ▼
2001:db8:12::1
```

---

# 🛣️ IPv6 Routing Checklist

Never troubleshoot only the tunnel.

Check:

```text
Endpoint
   ↓
Addressing
   ↓
Routing
   ↓
Tunnel
   ↓
Underlay
   ↓
Firewall
   ↓
Return Path
```

---

## Routing Validation

* [ ] Destination prefix identified
* [ ] Route exists
* [ ] Correct next hop selected
* [ ] Correct tunnel interface selected
* [ ] No competing route with higher preference
* [ ] Remote side has return route
* [ ] Default route is correct
* [ ] Dynamic routing considered where appropriate

### Core Question

> **Where does FortiGate send the destination prefix?**

---

# 🔐 Firewall Policy Checklist

A functional tunnel does **not** automatically mean traffic is allowed.

Verify:

* [ ] LAN → Tunnel policy exists
* [ ] Tunnel → LAN policy exists
* [ ] Correct source interface
* [ ] Correct destination interface
* [ ] Correct source address
* [ ] Correct destination address
* [ ] Required IPv6 services allowed
* [ ] Logging enabled where required
* [ ] Security profiles reviewed
* [ ] Policy order verified
* [ ] Return traffic permitted

---

## Policy Path

```text
IPv6 LAN
   │
   ▼
Firewall Policy
   │
   ▼
Tunnel Interface
   │
   ▼
Remote IPv6 Network
```

---

# 🔐 NAT Checklist

For most site-to-site tunnel designs:

```text
NAT
 │
 └── Usually NOT required
```

Validate:

* [ ] NAT requirement explicitly identified
* [ ] NAT disabled when unnecessary
* [ ] Source addresses preserved
* [ ] Remote network knows the source prefix
* [ ] Return route exists
* [ ] No asymmetric routing introduced

---

# 🔍 Underlay → Overlay Troubleshooting

## Step 1 — Underlay

### 6in4

```text
IPv4 Endpoint A
       │
       ▼
IPv4 Infrastructure
       │
       ▼
IPv4 Endpoint B
```

Check:

* [ ] Local IPv4 address
* [ ] Remote IPv4 address
* [ ] Route to remote endpoint
* [ ] Reachability
* [ ] No upstream filtering
* [ ] Correct physical/logical interface

### 4in6

```text
IPv6 Endpoint A
       │
       ▼
IPv6 Infrastructure
       │
       ▼
IPv6 Endpoint B
```

Check:

* [ ] Local IPv6 address
* [ ] Remote IPv6 address
* [ ] IPv6 route
* [ ] Reachability
* [ ] Correct interface

---

# 🧪 Tunnel Validation Checklist

## Endpoint

* [ ] Source correct
* [ ] Destination correct
* [ ] Interface correct

## Addressing

* [ ] Tunnel address configured
* [ ] Prefix correct
* [ ] Remote tunnel addressing compatible

## Routing

* [ ] Destination route exists
* [ ] Route points to tunnel
* [ ] Return route exists

## Policy

* [ ] Forward policy exists
* [ ] Return policy exists
* [ ] Policy order correct

## NAT

* [ ] NAT behavior verified

---

# 🛠️ VNE Troubleshooting

Use:

```bash
diagnose test application vned 1
```

Check:

* [ ] VNE process state
* [ ] Configuration retrieval
* [ ] Update URL
* [ ] Border Relay information
* [ ] Tunnel state
* [ ] Provider parameters
* [ ] Configuration errors
* [ ] DNS resolution

---

# 🖥️ Configuration Server Checklist

If the MAP-E configuration is hosted on a web server:

```text
FortiGate
    │
    │ HTTP/HTTPS
    ▼
Configuration Server
    │
    ▼
map-e-config.xml
```

Verify:

* [ ] Server reachable
* [ ] Correct DNS resolution
* [ ] Correct URL
* [ ] File exists
* [ ] File permissions allow retrieval
* [ ] XML format is valid
* [ ] Required MAP-E fields exist

---

# 🔥 Transition Technology Comparison

| Technology  | Payload            | Transport           | Primary Concept                |
| ----------- | ------------------ | ------------------- | ------------------------------ |
| **6in4**    | IPv6               | IPv4                | IPv6 over IPv4                 |
| **4in6**    | IPv4               | IPv6                | IPv4 over IPv6                 |
| **DS-Lite** | IPv4               | IPv6                | IPv4 access over IPv6          |
| **MAP-E**   | IPv4               | IPv6                | IPv4 service over IPv6         |
| **VNE**     | Depends on service | Depends on provider | FortiGate transition mechanism |

---

# 🧠 NSE Memory Map

| If You See...                  | Think...                            |
| ------------------------------ | ----------------------------------- |
| IPv6 over IPv4                 | **6in4 / SIT**                      |
| IPv4 over IPv6                 | **4in6**                            |
| IPv4 application over IPv6 ISP | **DS-Lite**                         |
| AFTR                           | **DS-Lite**                         |
| Border Relay                   | **BR / MAP-E / transition service** |
| PSID                           | **MAP-E**                           |
| PSID length                    | **MAP-E**                           |
| PSID offset                    | **MAP-E**                           |
| BMR hostname                   | **MAP-E discovery**                 |
| AAAA record                    | **IPv6 resolution**                 |
| `update-url`                   | **VNE configuration retrieval**     |
| `sit-tunnel`                   | **6in4**                            |
| `ipv6-tunnel`                  | **4in6**                            |
| `vned`                         | **VNE troubleshooting**             |

---

# ⚡ One-Minute Memory Trick

```text
6in4
│
├── Payload = IPv6
└── Transport = IPv4


4in6
│
├── Payload = IPv4
└── Transport = IPv6


DS-Lite
│
├── IPv4 application
├── IPv6 access network
└── AFTR


MAP-E
│
├── IPv4 service
├── IPv6 infrastructure
├── PSID
└── Border Relay
```

---

# 🧭 IPv6 Tunnel Troubleshooting Flow

```text
                 IPv6 Connectivity Problem
                           │
                           ▼
                  Is the endpoint reachable?
                           │
                    ┌──────┴──────┐
                    │             │
                   NO            YES
                    │             │
                    ▼             ▼
                 Underlay      Tunnel State
                 Problem           │
                                  ▼
                              Addressing
                                  │
                                  ▼
                               Routing
                                  │
                                  ▼
                               Policy
                                  │
                                  ▼
                                 NAT
                                  │
                                  ▼
                              Return Path
                                  │
                                  ▼
                              Application
```

---

# 🧪 Final Validation Checklist

## Architecture

* [ ] IPv6 transition requirement identified
* [ ] Payload address family identified
* [ ] Transport address family identified
* [ ] Tunnel model identified
* [ ] Tunnel endpoints documented

## Underlay

* [ ] IPv4/IPv6 underlay operational
* [ ] Tunnel endpoint reachability verified
* [ ] Correct source interface selected
* [ ] Correct destination configured

## Tunnel

* [ ] Tunnel created
* [ ] Tunnel addressing configured
* [ ] Encapsulation direction verified
* [ ] Decapsulation endpoint verified

## Routing

* [ ] Destination route exists
* [ ] Route points to tunnel
* [ ] Remote route exists
* [ ] Return route exists
* [ ] No routing loop

## Firewall

* [ ] LAN → Tunnel policy exists
* [ ] Tunnel → LAN policy exists
* [ ] Correct addresses configured
* [ ] Required services allowed
* [ ] Policy order checked

## NAT

* [ ] NAT requirement reviewed
* [ ] Unnecessary NAT disabled
* [ ] Source preservation verified
* [ ] Return routing verified

## DS-Lite / MAP-E

* [ ] ISP service parameters verified
* [ ] AFTR/BR information verified
* [ ] PSID information verified
* [ ] BMR information verified
* [ ] Update URL verified
* [ ] DNS AAAA resolution verified
* [ ] VNE state verified

## Testing

* [ ] IPv4 connectivity tested
* [ ] IPv6 connectivity tested
* [ ] Tunnel endpoint tested
* [ ] Remote network tested
* [ ] Return traffic tested
* [ ] Application connectivity tested
* [ ] Logs reviewed
* [ ] Diagnostic commands reviewed

---

# 🎯 Quick Decision Matrix

| Requirement                            | Choose                                   |
| -------------------------------------- | ---------------------------------------- |
| IPv6 through IPv4 infrastructure       | **6in4 / SIT**                           |
| IPv4 through IPv6 infrastructure       | **4in6**                                 |
| IPv4 applications over IPv6 ISP access | **DS-Lite**                              |
| MAP-E provider service                 | **MAP-E / VNE**                          |
| Need IPv6 + IPv4 simultaneously        | **Dual Stack**                           |
| IPv6 hostname discovery                | **AAAA**                                 |
| Border Relay discovery by hostname     | **BMR hostname + DNS**                   |
| Site-to-site tunnel                    | **Check routing + policy + return path** |

---

# 🚨 Common IPv6 Tunnel Mistakes

* [ ] Did you confuse payload and transport?
* [ ] Did you configure the tunnel but forget routing?
* [ ] Did you configure routing but forget firewall policy?
* [ ] Did you verify only the outbound path?
* [ ] Did you forget the return route?
* [ ] Did you accidentally enable NAT?
* [ ] Did you verify the underlay first?
* [ ] Did you use an unreachable tunnel endpoint?
* [ ] Did you forget IPv6 firewall policy requirements?
* [ ] Did you assume tunnel creation automatically provides connectivity?
* [ ] Did you forget DNS/AAAA resolution for hostname-based BR discovery?
* [ ] Did you expose real provider credentials in configuration files?

---

# 🏆 Engineer's Rule

> **Never troubleshoot an IPv6 tunnel from the tunnel interface outward. Troubleshoot from the underlay toward the overlay.**

```text
UNDERLAY
   ↓
ENDPOINT
   ↓
TUNNEL
   ↓
ADDRESSING
   ↓
ROUTING
   ↓
FIREWALL
   ↓
NAT
   ↓
RETURN PATH
   ↓
APPLICATION
```

---

# 🔑 Key Takeaways

1. **6in4 = IPv6 over IPv4.**
2. **4in6 = IPv4 over IPv6.**
3. **DS-Lite carries IPv4 traffic through an IPv6 access network.**
4. **MAP-E uses IPv6 infrastructure to provide IPv4 service.**
5. **The first number identifies the payload address family.**
6. **The second number identifies the transport address family.**
7. **Tunnel creation alone does not guarantee connectivity.**
8. **Routing must explicitly direct traffic toward the tunnel.**
9. **Firewall policies must include the tunnel interface where required.**
10. **Site-to-site designs require a correct return path.**
11. **Underlay reachability should be verified before overlay troubleshooting.**
12. **PSID, PSID length, PSID offset and Border Relay information are important MAP-E concepts.**
13. **AAAA records can participate in IPv6 Border Relay hostname resolution.**
14. **VNE configuration retrieval should be validated independently.**
15. **For public repositories, never publish real credentials or sensitive provider configuration.**

---

# 📚 Related Topics

```text
IPv6
├── Dual Stack
├── IPv6 Routing
├── IPv6 Firewall Policy
├── IPv6 Addressing
│
├── Tunneling
│   ├── 6in4
│   ├── 4in6
│   ├── DS-Lite
│   └── MAP-E
│
└── FortiGate
    ├── SIT Tunnel
    ├── IPv6 Tunnel
    ├── VNE
    ├── Static Routing
    └── Troubleshooting
```

---

# 🔑 Keywords

`FortiGate IPv6` · `FortiOS 7.2.0` · `IPv6 Tunnel` · `IPv6 Tunneling` · `IPv6 Encapsulation` · `FortiGate 6in4` · `6in4 SIT Tunnel` · `FortiGate 4in6` · `IPv4 over IPv6` · `IPv6 over IPv4` · `DS-Lite FortiGate` · `Dual-Stack Lite` · `MAP-E` · `FortiGate VNE` · `Virtual Network Enabler` · `IPv6 Routing` · `IPv6 Static Route` · `IPv6 Firewall Policy` · `Border Relay` · `AFTR` · `PSID` · `BMR` · `AAAA Record` · `IPv6 Transition Technologies` · `FortiGate Troubleshooting` · `NSE IPv6` · `Fortinet IPv6`

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

> **SheynShield | Engineering Secure Networks**
>
> Practical Network Security • Fortinet • IPv6 • Routing • Infrastructure Design

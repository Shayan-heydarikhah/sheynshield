# FortiGate Device Operation & Inspection Mode  

> **SheynShield | Engineering Secure Networks**
>
> **FortiGate • FortiOS • NAT Mode • Transparent Mode • Layer 2 • Layer 3 • Forward Domain • VLAN Forward • Proxy-Based • Flow-Based • SNMP**
>
> **NSE 4 + NSE 7 Technical Reference**

---

## 🎯 1. FortiGate Operation Modes

FortiGate can operate primarily in two major network operation modes:

```text
                    FORTIGATE
                        │
             ┌──────────┴──────────┐
             │                     │
             ▼                     ▼
        NAT MODE             TRANSPARENT MODE
        Layer 3                  Layer 2
             │                     │
          Routing              Bridging
          NAT                  Switching-like
          Firewall             Firewall
```

| Feature                  | NAT Mode         | Transparent Mode                   |
| ------------------------ | ---------------- | ---------------------------------- |
| OSI focus                | Layer 3          | Layer 2                            |
| IP address on interfaces | Yes              | Different management model         |
| Routing                  | Yes              | Limited / different design         |
| NAT                      | Yes              | No traditional NAT                 |
| Deployment               | Routed           | Inline / bridge                    |
| Common use               | Most deployments | Transparent insertion              |
| Management IP            | Interface-based  | Dedicated management configuration |
| Firewall policies        | Yes              | Yes                                |

### Golden Rule

```text
NAT Mode
    =
FortiGate participates in Layer 3 routing

Transparent Mode
    =
FortiGate primarily forwards Layer 2 traffic
```

---

# 🌐 2. NAT Mode

NAT mode is the **default and most commonly used operating mode**.

Typical architecture:

```text
LAN
 │
 ▼
FortiGate
 │
 ├── Routing
 ├── Firewall Policy
 ├── NAT
 └── Security Inspection
 │
 ▼
WAN
 │
 ▼
Internet
```

In NAT mode, FortiGate acts similarly to a Layer 3 router while providing security enforcement.

---

## 🧭 NAT Mode Mental Model

```text
Ingress Packet
      ↓
Interface
      ↓
Route Lookup
      ↓
Firewall Policy
      ↓
NAT
      ↓
Security Inspection
      ↓
Egress Interface
```

Remember:

```text
Routing
   ↓
WHERE should the packet go?

Firewall Policy
   ↓
IS the traffic allowed?

NAT
   ↓
WHAT source/destination address should be translated?
```

---

# 🔀 3. Transparent Mode

Transparent mode is designed for deployments where FortiGate needs to inspect and control traffic **without becoming the Layer 3 gateway for the protected networks**.

Conceptually:

```text
Network A
   │
   ▼
FortiGate
Transparent Mode
   │
   ▼
Network B
```

The device behaves more like a security bridge than a traditional router.

### Typical use cases

* Inline security deployment
* Existing routed infrastructure
* Migration scenarios
* Security inspection without redesigning IP addressing
* Environments where FortiGate should remain transparent to Layer 3 topology

---

# ⚠️ 4. Transparent Mode & ARP

One of the important operational considerations in transparent deployments is **Layer 2 behavior**, especially ARP.

Mental model:

```text
Host A
  │
  │ ARP
  ▼
Layer 2 Network
  │
  ▼
FortiGate
  │
  ▼
Host B
```

Because FortiGate is operating primarily at Layer 2:

```text
ARP
MAC Learning
Broadcast
VLAN
STP
Layer-2 Protocols
```

must be considered carefully during design and troubleshooting.

### NSE 7 Thinking

When troubleshooting Transparent Mode, don't immediately start with routing.

Start with:

```text
Physical
   ↓
Ethernet
   ↓
MAC
   ↓
VLAN
   ↓
ARP
   ↓
L2 Forwarding
   ↓
Firewall Policy
```

---

# 🧩 5. Forward Domain

`forward-domain` can be used in transparent mode to logically separate Layer 2 forwarding domains.

Conceptually:

```text
                 FortiGate
                    │
          ┌─────────┴─────────┐
          │                   │
    Forward Domain 340   Forward Domain 341
          │                   │
       VLAN 340            VLAN 341
```

Traffic belonging to one forwarding domain is isolated from interfaces/VLANs assigned to another forwarding domain.

---

## 🔧 6. Forward Domain Example

```cli
config system interface

    edit "port1"
        set forward-domain 340
    next

    edit "port2"
        set forward-domain 340
    next

    edit "port3"
        set forward-domain 341
    next

    edit "port1-340"
        set interface "port1"
        set vlanid 340
        set forward-domain 340
    next

    edit "port1-341"
        set interface "port1"
        set vlanid 341
        set forward-domain 341
    next

end
```

### Logical Result

```text
port1
 │
 ├── VLAN 340 ── Forward Domain 340
 │                    │
 └────────────────────┤
                      ▼
                  Allowed L2 Group

port1
 │
 └── VLAN 341 ── Forward Domain 341
                       │
                       ▼
                   Separate L2 Group
```

### Important

Forward domains are a **Layer 2 forwarding concept**.

Do not confuse:

```text
Forward Domain
      ≠
VDOM
```

and:

```text
Forward Domain
      ≠
Firewall Policy
```

---

# 🌉 7. Layer 2 Forwarding

By default, FortiGate does not necessarily forward every possible Layer 2 protocol transparently.

If specific Layer 2 protocols must traverse the FortiGate, the relevant forwarding behavior may need to be enabled.

Example:

```cli
config system interface
    edit "port1"
        set l2forward enable
    next
end
```

### Think Before Enabling

Ask:

```text
What protocol?
      ↓
Why does it need Layer 2 forwarding?
      ↓
Which interfaces?
      ↓
What security impact?
      ↓
Could it create a loop?
```

Do not enable broad Layer 2 forwarding simply because a protocol is not working.

---

# 🌲 8. STP Forwarding

Spanning Tree Protocol can become important when FortiGate participates in a Layer 2 topology.

Relevant interface settings can include:

```cli
config system interface
    edit "INTERFACE"
        set l2forward enable
        set stpforward enable
    next
end
```

Conceptual flow:

```text
Layer 2 Network
      │
      ├── Switch A
      │
      ├── FortiGate
      │
      └── Switch B
             │
             ▼
            STP
```

### Why?

STP exists to prevent Layer 2 loops.

```text
Without loop prevention:

Switch A
   ↓
FortiGate
   ↓
Switch B
   ↓
Switch A
   ↓
LOOP
```

---

# 🧠 9. Transparent Mode Troubleshooting

When traffic fails in Transparent Mode:

```text
1. Physical Link
       ↓
2. MAC Learning
       ↓
3. VLAN
       ↓
4. Forward Domain
       ↓
5. L2 Forwarding
       ↓
6. STP
       ↓
7. Firewall Policy
       ↓
8. Security Inspection
```

### Key Questions

```text
☐ Is the interface up?
☐ Is the VLAN correct?
☐ Is the MAC address learned?
☐ Are interfaces in the correct forwarding domain?
☐ Is L2 forwarding enabled where required?
☐ Is STP required?
☐ Is there a Layer 2 loop?
☐ Is the firewall policy matching?
☐ Is security inspection dropping traffic?
```

---

# 🔢 10. Transparent Mode Interface Scale

Older FortiOS documentation and training material may cite specific limits for the number of interfaces/logical interfaces in transparent mode.

These limits are **model- and FortiOS-version-dependent**.

Therefore:

```text
Never memorize:
"FortiGate = X interfaces"
```

Instead:

```text
FortiGate Model
       +
FortiOS Version
       +
VDOM Architecture
       +
Logical Interfaces
       ↓
Actual Scale
```

### NSE 7 Rule

For production design, always validate platform limits in the documentation for the exact:

```text
Model
+
FortiOS Release
```

---

# 🔀 11. VLAN Forwarding in NAT Mode

In NAT mode, FortiGate provides Layer 3 interfaces for VLANs.

Depending on interface configuration, VLAN forwarding behavior can influence how VLAN traffic is handled on a physical interface.

A commonly referenced setting is:

```cli
config system interface
    edit "port1"
        set vlanforward disable
    next
end
```

When VLAN forwarding behavior is changed, understand the resulting Layer 2 traffic path and whether VLAN-to-VLAN forwarding is expected.

---

# 🧠 12. VLAN Forwarding Mental Model

Conceptually:

```text
Physical Interface
        │
        ├── VLAN 100
        │
        ├── VLAN 200
        │
        └── VLAN 300
```

With forwarding behavior enabled:

```text
VLAN Traffic
      ↓
May be forwarded toward other VLAN interfaces
on the same physical interface according to
the configured forwarding behavior.
```

With appropriate forwarding restrictions:

```text
VLAN 100
   │
   X
   │
VLAN 200
```

### Important

Do not use `vlanforward` as a replacement for proper:

```text
Inter-VLAN Firewall Policy
+
Segmentation
+
Routing Design
```

---

# 🖥️ 13. Management IP in Transparent Mode

Transparent mode requires a management address so administrators can manage the device.

Conceptually:

```text
Management Host
      │
      ▼
Management IP
      │
      ▼
FortiGate
```

Historical CLI examples may look like:

```cli
config system settings
    set opmode transparent
    set manageip 192.168.20.250 255.255.255.0
    set gateway 192.168.20.254
end
```

> ⚠️ Syntax and available parameters can vary by FortiOS release. Validate against the target version before deployment.

---

# 🌐 14. NAT Mode Configuration Concept

A simplified historical-style example:

```cli
config system settings
    set opmode nat
    set ip 192.168.20.252 255.255.255.0
    set device "port1"
    set gateway 192.168.20.254
end
```

However, modern FortiOS deployments generally configure IP addressing directly under:

```cli
config system interface
```

Example:

```cli
config system interface
    edit "port1"
        set ip 192.168.20.252 255.255.255.0
    next
end
```

### Version Awareness

Always distinguish:

```text
Historical FortiOS syntax
          vs
Current FortiOS syntax
```

This is particularly important when creating NSE study notes from older material.

---

# 📡 15. NetBIOS Forwarding

FortiGate can provide NetBIOS/WINS-related forwarding functionality in applicable configurations.

Example:

```cli
config system interface
    edit "internal"
        set netbios-forward enable
        set wins-ip 192.168.111.222
    next
end
```

Conceptually:

```text
Windows Client
      │
      ▼
NetBIOS
      │
      ▼
FortiGate
      │
      ▼
WINS Server
```

This is primarily a **legacy Windows networking consideration** and should be used only when required by the environment.

---

# 🔬 16. Inspection Modes

FortiGate security inspection can operate using different inspection architectures.

Two major concepts:

```text
              INSPECTION
                  │
          ┌───────┴───────┐
          ▼               ▼
     FLOW-BASED      PROXY-BASED
```

---

# ⚡ 17. Flow-Based Inspection

Flow-based inspection analyzes traffic as it passes through the FortiGate.

Concept:

```text
Packet
  ↓
Inspect
  ↓
Decision
  ↓
Forward
```

Advantages:

* Lower buffering requirements
* Efficient traffic processing
* Strong performance
* Suitable for many security inspection scenarios

Mental model:

```text
Traffic
  ↓
Inspect in Flow
  ↓
Continue Forwarding
```

---

# 🧪 18. Proxy-Based Inspection

Proxy-based inspection uses an intermediary inspection process.

Conceptually:

```text
Client
  │
  ▼
FortiGate Proxy
  │
  ├── Receive
  ├── Inspect
  ├── Buffer / Process
  └── Forward
  │
  ▼
Server
```

This can provide deeper inspection capabilities for certain security features.

---

# 🆚 19. Flow-Based vs Proxy-Based

| Feature               | Flow-Based                | Proxy-Based                       |
| --------------------- | ------------------------- | --------------------------------- |
| Processing model      | Inline/stream-oriented    | Proxy/intermediary                |
| Buffering             | Lower                     | Higher                            |
| Performance           | Generally high            | Potentially higher resource usage |
| Deep content handling | Feature-dependent         | Stronger for applicable features  |
| Antivirus             | Supported                 | Supported                         |
| Web filtering         | Supported                 | Supported                         |
| DLP                   | Feature/version dependent | Often requires proxy architecture |
| WAF                   | Specialized               | Proxy architecture                |
| Security profiles     | Depends on feature        | Depends on feature                |

> Exact feature availability and behavior are FortiOS-version dependent.

---

# 🛡️ 20. Security Profiles & Inspection Mode

A common exam and operational mistake is assuming:

```text
Security Profile
      =
Any Inspection Mode
```

Instead:

```text
Security Feature
       ↓
Supported Inspection Mode
       ↓
Policy Configuration
       ↓
Actual Inspection
```

Always verify the feature's requirements for the specific FortiOS release.

---

# 🌐 21. Explicit Proxy

FortiGate can provide explicit proxy functionality.

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

Relevant GUI areas and available functionality vary by FortiOS release.

Typical considerations:

```text
☐ Client proxy configuration
☐ Authentication
☐ Web filtering
☐ Logging
☐ SSL inspection
☐ Policy
☐ Performance
```

---

# 💾 22. WAN Optimization & Cache

FortiGate has included WAN optimization and caching capabilities in applicable FortiOS releases and models.

Concept:

```text
Branch A
   │
   ▼
FortiGate
   │
   │ Optimized Traffic
   ▼
WAN
   │
   ▼
FortiGate
   │
   ▼
Branch B
```

Caching can reduce repeated transfers in suitable workloads.

> Availability and behavior depend heavily on FortiOS version and platform.

---

# 🛡️ 23. DLP and Inspection Architecture

Data Loss Prevention requires understanding the relationship between:

```text
Traffic
  ↓
Inspection Mode
  ↓
Content Visibility
  ↓
DLP Engine
  ↓
Policy Decision
```

For NSE 7 troubleshooting, ask:

```text
Can FortiGate actually see the content?
        ↓
Is the traffic encrypted?
        ↓
Is SSL inspection configured?
        ↓
Is the required inspection mode enabled?
        ↓
Is the DLP profile attached?
```

---

# 📊 24. SNMP

SNMP provides monitoring and management visibility.

```text
                    FortiGate
                       │
             ┌─────────┴─────────┐
             │                   │
             ▼                   ▼
          SNMP GET            SNMP TRAP
             │                   │
             ▼                   ▼
       SNMP Manager         SNMP Manager
```

---

# 📥 25. SNMP Query / Polling

The SNMP manager periodically requests information from the FortiGate.

Common SNMP port:

```text
UDP/161
```

Flow:

```text
SNMP Manager
      │
      │ GET
      ▼
FortiGate
      │
      │ RESPONSE
      ▼
SNMP Manager
```

---

# 🚨 26. SNMP Trap

A trap is an event notification sent by the FortiGate toward the SNMP manager.

Common port:

```text
UDP/162
```

Flow:

```text
FortiGate
    │
    │ Event
    ▼
SNMP Trap
    │
    ▼
SNMP Manager
```

### Memory Trick

```text
161 = ASK

162 = ALERT
```

---

# 🔐 27. SNMP Security

Avoid treating SNMP as automatically secure.

Legacy:

```text
SNMPv1
SNMPv2c
```

can rely on community strings and provide limited security.

Prefer stronger SNMP versions where supported:

```text
SNMPv3
```

because it can provide stronger authentication and privacy mechanisms.

---

# 🏢 28. Management Architecture

A secure enterprise design should separate management traffic from production traffic.

```text
                     ADMIN
                       │
                       ▼
                 Management VLAN
                       │
                       ▼
                  FortiGate
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
       SNMP          HTTPS            SSH
        │              │              │
        ▼              ▼              ▼
    Monitoring       Admin          Admin
```

---

# 🧠 29. NSE 4 — Must Know

For NSE 4-level knowledge, understand:

```text
☐ NAT Mode
☐ Transparent Mode
☐ Layer 2 vs Layer 3
☐ Management IP
☐ Default operation mode
☐ Firewall policy behavior
☐ VLAN interfaces
☐ Layer 2 forwarding
☐ Forward domains
☐ STP forwarding
☐ Inspection modes
☐ Flow-based inspection
☐ Proxy-based inspection
☐ SNMP
☐ UDP/161
☐ UDP/162
```

---

# 🧠 30. NSE 7 — Troubleshooting Perspective

At NSE 7 level, don't simply ask:

```text
"Which setting should I enable?"
```

Ask:

```text
What is the traffic architecture?
        ↓
L2 or L3?
        ↓
Which interface receives traffic?
        ↓
How is traffic forwarded?
        ↓
Which policy matches?
        ↓
Which inspection engine processes it?
        ↓
Could ARP / VLAN / STP affect it?
        ↓
Could security inspection drop it?
        ↓
How do I prove the behavior?
```

---

# 🔍 31. Packet Troubleshooting Matrix

| Symptom                     | First Area to Check                 |
| --------------------------- | ----------------------------------- |
| No connectivity in NAT mode | Routing                             |
| Correct route but blocked   | Firewall Policy                     |
| VLAN traffic not working    | VLAN / Interface                    |
| ARP problems                | L2 / VLAN / Transparent Mode        |
| Unexpected Layer 2 traffic  | Forwarding Domain / VLAN Forwarding |
| Layer 2 loop                | STP                                 |
| Web traffic blocked         | Web Filter / SSL Inspection         |
| Content inspection failure  | Inspection Mode                     |
| SNMP monitoring unavailable | SNMP configuration / UDP 161        |
| Traps not received          | SNMP Trap / UDP 162                 |
| High CPU during inspection  | Inspection / Security Profiles      |
| Unexpected proxy behavior   | Explicit Proxy / Policy             |

---

# 🧪 32. Troubleshooting Workflow

Use this order:

```text
PHYSICAL
   ↓
INTERFACE
   ↓
L2 / VLAN
   ↓
ARP / MAC
   ↓
ROUTING
   ↓
FIREWALL POLICY
   ↓
NAT
   ↓
INSPECTION MODE
   ↓
SECURITY PROFILE
   ↓
SESSION
   ↓
RETURN TRAFFIC
```

For Transparent Mode:

```text
PHYSICAL
   ↓
MAC
   ↓
VLAN
   ↓
FORWARD DOMAIN
   ↓
L2 FORWARDING
   ↓
STP
   ↓
POLICY
   ↓
INSPECTION
```

---

# 🛡️ 33. Configuration Checklist

## Operation Mode

```text
☐ Correct operation mode selected
☐ NAT vs Transparent documented
☐ Design matches network topology
☐ Management architecture documented
```

## Transparent Mode

```text
☐ Forward domains reviewed
☐ VLAN behavior verified
☐ L2 forwarding reviewed
☐ STP requirements checked
☐ Layer 2 loops eliminated
☐ ARP behavior tested
```

## NAT Mode

```text
☐ Interface addressing correct
☐ Routing correct
☐ Default route correct
☐ Firewall policy configured
☐ NAT requirement evaluated
☐ Return path verified
```

## Inspection

```text
☐ Flow vs Proxy selected intentionally
☐ Security profile compatible
☐ SSL inspection considered
☐ Performance impact evaluated
☐ Logging enabled where required
```

## SNMP

```text
☐ SNMP version selected
☐ SNMPv3 preferred where supported
☐ Manager IP restricted
☐ UDP/161 verified
☐ UDP/162 verified for traps
☐ Monitoring tested
```

---

# ⚡ 34. Golden Rules

```text
NAT Mode
    =
Layer 3 Security Gateway

Transparent Mode
    =
Layer 2 Security Bridge

Routing
    =
WHERE?

Firewall Policy
    =
ALLOW OR DENY?

Forward Domain
    =
L2 Forwarding Separation

STP
    =
Layer 2 Loop Prevention

Flow-Based
    =
Inspect Traffic Flow

Proxy-Based
    =
Proxy / Buffer / Inspect / Forward

SNMP 161
    =
QUERY

SNMP 162
    =
TRAP

L2 Troubleshooting
    =
MAC + VLAN + ARP + STP

L3 Troubleshooting
    =
IP + Route + Policy + NAT
```

---

# ⚡ 35. 60-Second Revision

```text
                         FORTIGATE
                             │
              ┌──────────────┴──────────────┐
              │                             │
              ▼                             ▼
          NAT MODE                    TRANSPARENT MODE
          Layer 3                       Layer 2
              │                             │
        ┌─────┼─────┐                ┌──────┼──────┐
        ▼     ▼     ▼                ▼      ▼      ▼
      Route Policy NAT             VLAN    MAC     STP
        │     │     │                │      │       │
        └─────┼─────┘                └──────┼───────┘
              │                             │
              └──────────┬──────────────────┘
                         ▼
                   SECURITY POLICY
                         │
                         ▼
                  INSPECTION MODE
                    │         │
                    ▼         ▼
                  FLOW      PROXY
                    │         │
                    └────┬────┘
                         ▼
                      LOGGING
                         │
                         ▼
                       SNMP
                    161 / 162
```

---

# 🏁 Final Mental Model

The most important distinction is:

```text
                 WHERE DOES THE PACKET LIVE?
                           │
                 ┌─────────┴─────────┐
                 ▼                   ▼
               LAYER 2             LAYER 3
                 │                   │
                 ▼                   ▼
           Transparent             NAT
                 │                   │
          MAC / VLAN / STP     Route / Policy / NAT
                 │                   │
                 └─────────┬─────────┘
                           ▼
                    SECURITY INSPECTION
                           │
                    ┌──────┴──────┐
                    ▼             ▼
                  FLOW          PROXY
                    │             │
                    └──────┬──────┘
                           ▼
                       LOGGING
                           │
                           ▼
                         SNMP
```

> **Understand the forwarding plane first. Then understand the security plane. Finally understand the inspection engine.**

That mental model is what separates **FortiGate configuration knowledge** from **FortiGate engineering and troubleshooting knowledge**.

---

## 🔖 Keywords

`FortiGate Operation Mode`
`FortiGate NAT Mode`
`FortiGate Transparent Mode`
`FortiGate Transparent Mode  `
`FortiGate NAT Mode  `
`FortiGate Layer 2 Forwarding`
`FortiGate Layer 3 Routing`
`FortiGate Forward Domain`
`FortiGate VLAN Forwarding`
`FortiGate STP Forwarding`
`FortiGate ARP Troubleshooting`
`FortiGate Flow Based Inspection`
`FortiGate Proxy Based Inspection`
`FortiGate Inspection Mode`
`FortiGate SNMP`
`FortiGate SNMP 161`
`FortiGate SNMP Trap 162`
`FortiOS  `
`Fortinet NSE4`
`Fortinet NSE7`
`FortiGate NSE4`
`FortiGate NSE7`
`FortiGate Troubleshooting`
`FortiGate Layer 2 Troubleshooting`
`FortiGate Layer 3 Troubleshooting`
`Fortinet Firewall  `
`Fortinet Security  `

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

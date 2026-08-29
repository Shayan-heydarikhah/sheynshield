# 🏢 FortiGate VDOM (Virtual Domain)  

> **SheynShield | Engineering Secure Networks**
>
> **FortiGate • VDOM • Multi-Tenancy • Segmentation • Administration • NSE4/NSE7**

---

## 🎯 1. What Is a VDOM?

**VDOM (Virtual Domain)** allows a single physical FortiGate to operate as multiple logically independent firewall instances.

### Mental Model

```text
                    Physical FortiGate
                           │
          ┌────────────────┼────────────────┐
          │                │                │
        VDOM A           VDOM B           VDOM C
          │                │                │
       Customer A       Customer B       Customer C
          │                │                │
       Policies         Policies         Policies
       Routing          Routing          Routing
       VPN              VPN              VPN
       Objects          Objects          Objects
```

Think of VDOMs as:

```text
One Physical FortiGate
        ↓
Multiple Virtual Firewalls
```

This is conceptually similar to:

* Cisco firewall contexts
* VRF-like logical separation
* Multi-tenant firewall architectures

> **Important:** VDOMs provide logical isolation, but they are not identical to Cisco VRF or firewall contexts. Their exact capabilities and resource behavior depend on FortiOS version and platform.

---

# 🧩 2. Why Use VDOM?

VDOMs are useful when one FortiGate must provide separate security environments.

### Common Use Cases

```text
Service Provider
      ↓
Customer A → VDOM-A
Customer B → VDOM-B
Customer C → VDOM-C
```

or:

```text
Enterprise
      ↓
Production → VDOM-Prod
DMZ        → VDOM-DMZ
Guest      → VDOM-Guest
Security   → VDOM-Security
```

### Benefits

* Multi-tenancy
* Configuration separation
* Policy isolation
* Routing-table separation
* Administrative separation
* Separate security policies
* Separate interface ownership
* Service-provider deployments
* Managed security services

---

# 🏗️ 3. VDOM Architecture

A FortiGate with VDOMs can be understood as two major administrative levels:

```text
                 FortiGate
                    │
             ┌──────┴──────┐
             │             │
          Global          VDOM
             │             │
       System-wide     Tenant / Firewall
       configuration     configuration
```

### Global

Controls configuration that belongs to the overall FortiGate.

### VDOM

Contains firewall-instance-specific configuration.

Examples:

```text
Global
 ├── VDOM definitions
 ├── Global administrators
 ├── System-wide settings
 └── VDOM relationships

VDOM
 ├── Interfaces
 ├── Firewall policies
 ├── Routing
 ├── Objects
 ├── VPN
 ├── Security profiles
 └── Logs / configuration
```

---

# 🔐 4. Global vs VDOM Context

This is an important operational concept.

### Global Context

```cli
config global
```

Used for configuration that belongs to the global FortiGate scope.

### VDOM Context

```cli
config vdom
edit root
```

Moves configuration into a specific VDOM.

Example:

```cli
config global
```

Then:

```cli
config vdom
edit root
```

You are now working inside the `root` VDOM.

---

# 🏠 5. Root VDOM

The default FortiGate environment is normally:

```text
root
```

The root VDOM is not simply another physical firewall.

It is the default VDOM and can contain normal FortiGate functions such as:

```text
Interfaces
Routing
Firewall Policies
VPN
Security Profiles
Objects
```

A simplified architecture:

```text
FortiGate
│
├── Global
│
└── root VDOM
     ├── Interfaces
     ├── Routing
     ├── Policies
     └── Security
```

---

# 🔗 6. VDOM Link

A **VDOM Link** provides logical connectivity between VDOMs.

Example:

```text
             VDOM-A
                │
                │
           VDOM Link
                │
                │
             VDOM-B
```

This can be used to build controlled communication between isolated firewall domains.

### Example Architecture

```text
Internet
   │
   ▼
VDOM-EDGE
   │
   │ VDOM Link
   ▼
VDOM-INTERNAL
   │
   ▼
Internal Network
```

The VDOM link behaves as an interface from the perspective of the connected VDOMs.

---

# 🔄 7. VDOM Link Traffic

A useful mental model:

```text
VDOM-A
   │
Firewall Policy
   │
VDOM Link
   │
Firewall Policy
   │
VDOM-B
```

Traffic does **not** automatically become trusted simply because the VDOMs are connected.

You still need appropriate firewall policies.

```text
VDOM Link
    ≠
Automatic Permit
```

---

# 🛠️ 8. VDOM Link Example

VDOM links can be created through:

```text
Network
   ↓
Interfaces
   ↓
VDOM Link
```

The exact GUI workflow can vary by FortiOS release.

Conceptually:

```text
VDOM-A
   │
port/interface
   │
VDOM Link
   │
port/interface
   │
VDOM-B
```

---

# 👥 9. VDOM Administration

VDOMs allow administrative separation.

Example:

```text
                    Global Admin
                         │
             ┌───────────┼───────────┐
             │           │           │
          VDOM-A       VDOM-B      VDOM-C
             │           │           │
          Admin-A      Admin-B     Admin-C
```

A VDOM administrator can be restricted to specific VDOMs instead of receiving full FortiGate control.

---

# 👑 10. Super Administrator vs VDOM Administrator

### Super Administrator

Typically has access to:

```text
Global
+
All VDOMs
```

### VDOM Administrator

Typically has access only to assigned VDOMs.

Mental model:

```text
Super Admin
     ↓
Global + VDOMs

VDOM Admin
     ↓
Assigned VDOM
```

This supports:

* Least privilege
* Administrative separation
* Multi-tenant management
* Operational isolation

---

# ⚙️ 11. Enabling VDOM Administration

A typical configuration begins from the global context.

```cli
config system global
    set vdom-admin enable
end
```

> **Important:** The exact command availability and behavior can vary by FortiOS version. Always verify against the target FortiOS release.

---

# 🧱 12. VDOM Modes

FortiOS has historically used different VDOM operating modes.

Conceptually:

```text
No VDOM
   ↓
Single VDOM

Multi VDOM
   ↓
Multiple independent VDOMs
```

Common terminology encountered in Fortinet documentation/configuration includes:

```text
no-vdom
multi-vdom
split-vdom
```

The exact supported options and command syntax depend on the FortiOS version.

---

# 🧪 13. VDOM Configuration Example

A conceptual workflow:

```cli
config system global
    set vdom-admin enable
end
```

Then:

```cli
config vdom
edit root
```

You can then work with the configuration belonging to that VDOM.

---

# 🌐 14. Routing in VDOMs

One of the most important VDOM concepts:

> **Each VDOM behaves like an independent routing/security domain.**

Conceptually:

```text
VDOM-A
 ├── Routing Table A
 ├── Policies A
 └── Interfaces A

VDOM-B
 ├── Routing Table B
 ├── Policies B
 └── Interfaces B
```

Therefore:

```text
Route in VDOM-A
      ≠
Route in VDOM-B
```

---

# 🚦 15. Firewall Policies Are VDOM-Specific

Policies normally belong to the VDOM where the interfaces exist.

Example:

```text
VDOM-A

LAN
 │
 ▼
Firewall Policy
 │
 ▼
WAN
```

A policy configured in VDOM-A does not automatically apply to VDOM-B.

---

# 🔌 16. Physical Interfaces and VDOMs

An interface is associated with a VDOM.

Conceptually:

```text
Physical Port1
      │
      ▼
   VDOM-A
```

or:

```text
Physical Port2
      │
      ▼
   VDOM-B
```

This allows physical resources to be logically separated between virtual firewalls.

> Interface ownership and resource assignment depend on the FortiGate platform and FortiOS configuration.

---

# 🧠 17. VDOM Link vs Physical Interface

### Physical Interface

```text
Physical Port
      ↓
VDOM
```

### VDOM Link

```text
VDOM-A
   ↓
Logical Link
   ↓
VDOM-B
```

VDOM links are particularly useful when traffic needs to cross administrative or security boundaries between VDOMs.

---

# 🔥 18. Multi-Tenant Firewall Example

A service provider could build:

```text
                  Internet
                     │
                 FortiGate
                     │
       ┌─────────────┼─────────────┐
       │             │             │
    VDOM-A         VDOM-B        VDOM-C
   Customer-A     Customer-B    Customer-C
       │             │             │
    Policies       Policies      Policies
    Routing        Routing       Routing
    VPN            VPN           VPN
```

Each customer can have:

```text
Independent Policies
Independent Routing
Independent Objects
Independent Security Configuration
```

---

# 🔐 19. Security Isolation

VDOMs can help separate:

```text
Tenant A
    ≠
Tenant B
```

However:

> **VDOM separation is a logical security boundary, not a replacement for physical separation when regulatory, architectural, or threat-model requirements demand dedicated hardware.**

Always evaluate:

* Resource contention
* Administrative requirements
* Compliance
* Logging requirements
* HA design
* Upgrade impact
* Management architecture

---

# 🧭 20. VDOM Troubleshooting

When troubleshooting VDOM environments, always ask:

```text
Which VDOM am I in?
        ↓
Which interface belongs to this VDOM?
        ↓
Which routing table is being used?
        ↓
Which firewall policy is being evaluated?
        ↓
Is traffic crossing a VDOM Link?
        ↓
Is the required policy configured?
```

### Critical Question

```text
Am I troubleshooting Global
or
a specific VDOM?
```

---

# 🧪 21. Ping Troubleshooting with VDOM

A common mistake is attempting an operation from the wrong context.

Conceptually:

```cli
config global
execute ping 8.8.8.8
```

may not provide the expected result because the operation needs to be performed from the appropriate VDOM context.

Instead:

```cli
config vdom
edit root
execute ping 8.8.8.8
```

Now the operation is performed from the `root` VDOM context.

### Mental Model

```text
Global
   ↓
System-wide Context

VDOM
   ↓
Firewall / Routing Context
```

---

# 🔍 22. VDOM Troubleshooting Checklist

```text
☐ Confirm VDOM mode
☐ Confirm current VDOM
☐ Confirm interface ownership
☐ Check interface status
☐ Check IP addressing
☐ Check routing table
☐ Check firewall policy
☐ Check NAT
☐ Check security profiles
☐ Check VDOM Link
☐ Check return route
☐ Check administrative permissions
☐ Check resource utilization
```

---

# 📊 23. VDOM Architecture  Table

| Component            | Scope                                  |
| -------------------- | -------------------------------------- |
| Global Configuration | FortiGate-wide                         |
| VDOM                 | Logical firewall instance              |
| Interface            | Assigned to a VDOM                     |
| Firewall Policy      | VDOM-specific                          |
| Routing              | VDOM-specific                          |
| VDOM Link            | Inter-VDOM connectivity                |
| Super Admin          | Global / broad access                  |
| VDOM Admin           | Restricted VDOM access                 |
| Root VDOM            | Default VDOM                           |
| Multi-VDOM           | Multiple logical firewall environments |

---

# 🎯 24. NSE4 Exam Focus

For NSE4-level understanding, know:

```text
VDOM
├── What is it?
├── Why use it?
├── Root VDOM
├── Global vs VDOM
├── VDOM Link
├── Interface ownership
├── Policy isolation
├── Routing isolation
└── Administrator scope
```

### Remember

```text
One FortiGate
      ↓
Multiple VDOMs
      ↓
Multiple Logical Firewalls
```

---

# 🧠 25. NSE7 Design & Troubleshooting Focus

For advanced troubleshooting and architecture, think beyond:

> "How do I create a VDOM?"

Think:

```text
Resource Allocation
       ↓
Interface Ownership
       ↓
Routing Domain
       ↓
Policy Domain
       ↓
VDOM Link
       ↓
Security Boundary
       ↓
Management Boundary
       ↓
HA / Upgrade Impact
       ↓
Troubleshooting Context
```

---

# ⚠️ 26. Common VDOM Mistakes

### ❌ Mistake 1 — Assuming VDOM Link Allows Everything

```text
VDOM Link
   ≠
Automatic Trust
```

Firewall policies may still be required.

---

### ❌ Mistake 2 — Configuring from Global Context

Always determine whether the required setting belongs to:

```text
Global
```

or:

```text
VDOM
```

---

### ❌ Mistake 3 — Confusing VDOM with VRF

VDOMs provide much broader firewall-instance isolation than a simple routing-instance concept.

```text
VRF
→ Routing Separation

VDOM
→ Firewall Instance Separation
```

---

### ❌ Mistake 4 — Ignoring Resource Sharing

Multiple VDOMs still run on the same physical FortiGate.

```text
VDOM-A ─┐
VDOM-B ─┼── Physical Resources
VDOM-C ─┘
```

Therefore capacity planning still matters.

---

# 🚀 27. Real-World Design Pattern

### Managed Security Provider

```text
                         Internet
                            │
                       FortiGate
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
       VDOM-A            VDOM-B            VDOM-C
       Client-A          Client-B          Client-C
          │                 │                 │
       Firewall           Firewall          Firewall
       Policies           Policies          Policies
          │                 │                 │
       Routing            Routing           Routing
```

### Management

```text
                 Global Administrator
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
           VDOM-A      VDOM-B      VDOM-C
           Admin-A    Admin-B     Admin-C
```

This is one of the strongest practical use cases for VDOM architecture.

---

# ⚡ 28. 60-Second Revision

```text
VDOM
│
├── Virtual Domain
│
├── Multiple Logical Firewalls
│
├── Routing Separation
│
├── Policy Separation
│
├── Interface Ownership
│
├── Administrative Separation
│
├── VDOM Link
│
└── Multi-Tenancy
```

### Global vs VDOM

```text
GLOBAL
  ↓
FortiGate-wide configuration

VDOM
  ↓
Virtual Firewall configuration
```

### Connectivity

```text
VDOM-A
   │
   ▼
VDOM Link
   │
   ▼
VDOM-B
```

### Troubleshooting

```text
Check VDOM
   ↓
Check Interface
   ↓
Check Route
   ↓
Check Policy
   ↓
Check VDOM Link
   ↓
Check Return Path
```

---

# 🏁 Final Mental Model

```text
                    PHYSICAL FORTIGATE
                           │
                  ┌────────┴────────┐
                  │                 │
               GLOBAL            VDOMs
                                    │
                   ┌────────────────┼────────────────┐
                   │                │                │
                 VDOM-A           VDOM-B           VDOM-C
                   │                │                │
                Routing          Routing          Routing
                Policy           Policy           Policy
                Security         Security         Security
                   │                │                │
                   └────── VDOM Link / Isolation ───┘
```

> **Master VDOM by remembering one sentence:**
>
> **VDOM turns one physical FortiGate into multiple logically separated firewall environments with independent configuration, routing, policies, and administration.**

---

## 🔖 Keywords

`FortiGate VDOM  `
`FortiOS VDOM`
`Fortinet VDOM`
`FortiGate Virtual Domain`
`FortiGate Multi VDOM`
`FortiGate VDOM Link`
`FortiGate Global vs VDOM`
`FortiGate VDOM Administration`
`FortiGate Multi Tenancy`
`FortiGate VDOM Routing`
`FortiGate VDOM Firewall Policy`
`FortiGate VDOM Troubleshooting`
`Fortinet NSE4 VDOM`
`Fortinet NSE7 VDOM`
`FortiGate Firewall Architecture`
`Fortinet Multi Tenant Firewall`

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

# 🏢 FortiGate VDOM Checklist — Virtual Domains

> **SheynShield | Engineering Secure Networks**
>
> **FortiGate • FortiOS • VDOM • Multi-Tenancy • Segmentation • Routing • Firewall Policy • Administration • NSE 4 • NSE 7**

---

## 🎯 VDOM Quick Definition

* [ ] Understand that **VDOM = Virtual Domain**
* [ ] Understand VDOM as a logical firewall/security domain inside one physical FortiGate
* [ ] Understand that multiple VDOMs can coexist on the same FortiGate
* [ ] Understand that each VDOM can have its own:

  * [ ] Interfaces
  * [ ] Routing
  * [ ] Firewall policies
  * [ ] Firewall objects
  * [ ] VPN configuration
  * [ ] Security configuration
  * [ ] Administrative scope
* [ ] Understand that VDOMs provide **logical isolation**, not physical separation
* [ ] Understand that VDOM is broader than a simple routing-instance concept such as VRF

### Mental Model

```text
                    PHYSICAL FORTIGATE
                           │
          ┌────────────────┼────────────────┐
          │                │                │
        VDOM-A           VDOM-B           VDOM-C
          │                │                │
       Tenant A         Tenant B         Tenant C
          │                │                │
       Routing          Routing          Routing
       Policies         Policies         Policies
       Security         Security         Security
```

> **Golden Rule:**
> `One Physical FortiGate → Multiple Logical Firewall Environments`

---

# 🧩 1. VDOM — Basic Checklist

* [ ] Know what VDOM stands for
* [ ] Understand the purpose of VDOM
* [ ] Understand logical firewall separation
* [ ] Understand multi-tenancy use cases
* [ ] Understand interface ownership
* [ ] Understand routing separation
* [ ] Understand policy separation
* [ ] Understand administrator separation
* [ ] Understand VDOM resource sharing
* [ ] Understand that VDOM behavior depends on FortiOS version and platform

---

# 🏗️ 2. VDOM Use-Case Checklist

### Service Provider / MSSP

* [ ] Identify customer environments that require logical separation
* [ ] Consider one VDOM per customer where appropriate
* [ ] Separate customer policies
* [ ] Separate customer routing
* [ ] Separate customer objects
* [ ] Separate customer administration
* [ ] Evaluate logging requirements
* [ ] Evaluate resource allocation
* [ ] Evaluate HA architecture

```text
                  FORTIGATE
                     │
        ┌────────────┼────────────┐
        │            │            │
     VDOM-A       VDOM-B       VDOM-C
    Client-A     Client-B     Client-C
```

### Enterprise

* [ ] Evaluate VDOMs for strong logical separation between environments
* [ ] Consider Production / DMZ / Guest / Security domains where appropriate
* [ ] Document the security boundary
* [ ] Document administrative ownership
* [ ] Document traffic flows between VDOMs

---

# 🌐 3. VDOM Architecture Checklist

Understand the two major conceptual scopes:

```text
                 FORTIGATE
                    │
             ┌──────┴──────┐
             │             │
          GLOBAL          VDOM
             │             │
       FortiGate-wide   Firewall domain
       configuration    configuration
```

### Global Scope

* [ ] Understand global configuration
* [ ] Identify settings that apply to the FortiGate as a whole
* [ ] Understand global administration
* [ ] Understand VDOM definitions
* [ ] Understand global relationships/settings where applicable

### VDOM Scope

* [ ] Identify VDOM-specific configuration
* [ ] Identify VDOM interfaces
* [ ] Identify VDOM routes
* [ ] Identify VDOM policies
* [ ] Identify VDOM objects
* [ ] Identify VDOM security configuration

---

# 🧭 4. Global vs VDOM Context Checklist

* [ ] Know when configuration must be performed globally
* [ ] Know when configuration must be performed inside a VDOM
* [ ] Verify the current CLI context before troubleshooting
* [ ] Avoid making a VDOM configuration change from the wrong scope

### Global Context

```cli
config global
```

### VDOM Context

```cli
config vdom
edit root
```

### Operational Question

```text
Am I configuring:

☐ Global FortiGate configuration?

or

☐ Specific VDOM configuration?
```

---

# 🏠 5. Root VDOM Checklist

* [ ] Understand the concept of the `root` VDOM
* [ ] Recognize `root` as the default VDOM in typical FortiGate deployments
* [ ] Understand that root is a VDOM, not a physical interface
* [ ] Understand that normal firewall functions can exist inside the root VDOM
* [ ] Do not confuse root VDOM with global configuration

```text
FORTIGATE
│
├── GLOBAL
│
└── ROOT VDOM
     ├── Interfaces
     ├── Routing
     ├── Policies
     ├── Objects
     └── Security
```

---

# 🔗 6. VDOM Link Checklist

A VDOM Link provides logical connectivity between VDOMs.

* [ ] Understand the purpose of a VDOM Link
* [ ] Identify the two VDOMs participating in the link
* [ ] Configure the required logical interfaces
* [ ] Assign appropriate IP addressing where required
* [ ] Configure routing appropriately
* [ ] Configure firewall policies as required
* [ ] Verify return traffic
* [ ] Verify security inspection
* [ ] Verify logging

### Mental Model

```text
VDOM-A
   │
   │
VDOM LINK
   │
   │
VDOM-B
```

### Critical Rule

```text
VDOM Link
    ≠
Automatic Trust
```

* [ ] Never assume that creating a VDOM Link automatically permits traffic
* [ ] Verify the complete traffic path

---

# 🔥 7. Inter-VDOM Traffic Checklist

For traffic crossing VDOM boundaries:

```text
SOURCE
  ↓
SOURCE VDOM
  ↓
ROUTING
  ↓
FIREWALL POLICY
  ↓
VDOM LINK
  ↓
DESTINATION VDOM
  ↓
ROUTING
  ↓
FIREWALL POLICY
  ↓
DESTINATION
```

Verify:

* [ ] Source interface
* [ ] Source IP
* [ ] Destination IP
* [ ] Source VDOM
* [ ] Destination VDOM
* [ ] VDOM Link
* [ ] Route in source VDOM
* [ ] Policy in source VDOM
* [ ] Route in destination VDOM
* [ ] Policy in destination VDOM
* [ ] Return route
* [ ] NAT requirements
* [ ] Security profiles
* [ ] Session state

---

# 🔌 8. Interface Ownership Checklist

* [ ] Identify which VDOM owns each interface
* [ ] Verify physical interface assignment
* [ ] Verify VLAN interface ownership
* [ ] Verify aggregate/interface relationships
* [ ] Verify management interface architecture
* [ ] Verify that the required interface is available to the intended VDOM

```text
Physical Interface
       │
       ▼
    VDOM-A
```

or:

```text
Physical Interface
       │
       ▼
    VDOM-B
```

### Troubleshooting Question

```text
"Does this interface actually belong to the VDOM
where I am troubleshooting?"
```

---

# 🛣️ 9. VDOM Routing Checklist

Treat each VDOM as an independent routing/security domain.

* [ ] Verify routing table for the correct VDOM
* [ ] Verify connected routes
* [ ] Verify static routes
* [ ] Verify dynamic routing where applicable
* [ ] Verify default route
* [ ] Verify route preference
* [ ] Verify next-hop reachability
* [ ] Verify return route
* [ ] Verify asymmetric routing risks
* [ ] Do not assume routes from another VDOM are available

```text
VDOM-A Routing Table
        ≠
VDOM-B Routing Table
```

---

# 🔥 10. VDOM Firewall Policy Checklist

* [ ] Verify that the policy exists in the correct VDOM
* [ ] Verify source interface
* [ ] Verify destination interface
* [ ] Verify source address
* [ ] Verify destination address
* [ ] Verify service
* [ ] Verify schedule
* [ ] Verify action
* [ ] Verify NAT requirement
* [ ] Verify security profiles
* [ ] Verify logging
* [ ] Verify policy order
* [ ] Verify policy hit/session behavior

### Golden Rule

```text
Policy in VDOM-A
    ≠
Policy in VDOM-B
```

---

# 👥 11. VDOM Administration Checklist

* [ ] Understand global administrative scope
* [ ] Understand VDOM administrator scope
* [ ] Identify administrators assigned to each VDOM
* [ ] Apply least privilege
* [ ] Avoid unnecessary super-administrator access
* [ ] Separate customer administration where appropriate
* [ ] Separate operational teams where appropriate
* [ ] Audit administrative access

### Mental Model

```text
                  GLOBAL ADMIN
                       │
          ┌────────────┼────────────┐
          │            │            │
       VDOM-A       VDOM-B       VDOM-C
          │            │            │
       Admin-A      Admin-B      Admin-C
```

---

# 👑 12. Super Admin vs VDOM Admin Checklist

### Super Administrator

* [ ] Understand broad FortiGate-level access
* [ ] Understand global configuration access where applicable
* [ ] Understand access across multiple VDOMs

### VDOM Administrator

* [ ] Understand restricted administrative scope
* [ ] Identify assigned VDOM
* [ ] Verify permissions
* [ ] Verify profile
* [ ] Verify access limitations

### Security Rule

```text
More Privilege
      ↓
More Blast Radius
```

* [ ] Use least privilege
* [ ] Minimize super-admin usage
* [ ] Use individual administrator identities

---

# ⚙️ 13. VDOM Administration Configuration Awareness

A commonly encountered configuration concept is:

```cli
config system global
    set vdom-admin enable
end
```

* [ ] Verify whether the command exists in the target FortiOS version
* [ ] Verify its exact behavior
* [ ] Do not copy commands between FortiOS releases without validation
* [ ] Test administrative impact before production deployment

> **Version Awareness:** FortiOS CLI syntax and feature availability can change between releases.

---

# 🧱 14. VDOM Mode Checklist

Understand the terminology used by the target FortiOS release.

Conceptually:

```text
Single / No VDOM
       ↓
Multi-VDOM
       ↓
Multiple Logical Firewall Domains
```

* [ ] Identify current VDOM mode
* [ ] Understand whether multiple VDOMs are enabled
* [ ] Identify supported VDOM modes for the platform
* [ ] Verify exact CLI syntax for the FortiOS release
* [ ] Evaluate migration impact before changing VDOM architecture

---

# 🏢 15. Multi-Tenant Design Checklist

For service-provider environments:

* [ ] Create logical tenant boundaries
* [ ] Assign interfaces correctly
* [ ] Separate routing
* [ ] Separate policies
* [ ] Separate administrators
* [ ] Define inter-tenant communication rules
* [ ] Define logging ownership
* [ ] Define monitoring
* [ ] Define resource requirements
* [ ] Define HA requirements
* [ ] Define upgrade impact
* [ ] Define backup strategy
* [ ] Define disaster-recovery process

```text
                PHYSICAL FORTIGATE
                       │
       ┌───────────────┼───────────────┐
       │               │               │
    VDOM-A          VDOM-B          VDOM-C
   Tenant A        Tenant B        Tenant C
```

---

# 🔐 16. VDOM Security Isolation Checklist

Understand what VDOM isolation provides:

* [ ] Configuration separation
* [ ] Policy separation
* [ ] Routing separation
* [ ] Administrative separation
* [ ] Interface ownership separation
* [ ] Security-domain separation

Also evaluate:

* [ ] Shared physical resources
* [ ] CPU utilization
* [ ] Memory utilization
* [ ] Session capacity
* [ ] Throughput requirements
* [ ] HA architecture
* [ ] Upgrade dependencies
* [ ] Compliance requirements
* [ ] Management-plane requirements

### Important

```text
VDOM Isolation
      ≠
Physical Firewall Isolation
```

Use dedicated hardware when physical isolation is explicitly required by architecture, compliance, or threat model.

---

# 📊 17. Resource Planning Checklist

Multiple VDOMs still share the same physical FortiGate.

* [ ] Estimate aggregate throughput
* [ ] Estimate concurrent sessions
* [ ] Estimate new sessions/second
* [ ] Estimate VPN requirements
* [ ] Estimate security inspection load
* [ ] Estimate logging load
* [ ] Estimate memory requirements
* [ ] Estimate CPU requirements
* [ ] Review platform-specific limits
* [ ] Review FortiOS-specific limitations

```text
VDOM-A ─┐
VDOM-B ─┼──► Shared Physical Resources
VDOM-C ─┘
```

---

# 🔄 18. VDOM Link Design Checklist

Before implementing an inter-VDOM connection:

* [ ] Define why the VDOMs need communication
* [ ] Define source VDOM
* [ ] Define destination VDOM
* [ ] Define source networks
* [ ] Define destination networks
* [ ] Define required services
* [ ] Define routing
* [ ] Define security policy
* [ ] Define NAT requirements
* [ ] Define logging
* [ ] Define inspection
* [ ] Define return path

### Security Principle

```text
Default
   ↓
No Unnecessary Inter-VDOM Communication
```

---

# 🧪 19. VDOM Troubleshooting Checklist

When troubleshooting a connectivity problem:

### Step 1 — Context

* [ ] Confirm current VDOM
* [ ] Confirm whether operation belongs to Global or VDOM scope

### Step 2 — Interface

* [ ] Confirm interface ownership
* [ ] Confirm interface status
* [ ] Confirm IP configuration
* [ ] Confirm VLAN configuration

### Step 3 — Routing

* [ ] Check routing table
* [ ] Check destination route
* [ ] Check default route
* [ ] Check next hop
* [ ] Check return route

### Step 4 — Policy

* [ ] Identify matching firewall policy
* [ ] Check policy order
* [ ] Check source/destination objects
* [ ] Check service
* [ ] Check NAT
* [ ] Check security profiles

### Step 5 — Inter-VDOM

* [ ] Check VDOM Link
* [ ] Check both VDOMs
* [ ] Check routing on both sides
* [ ] Check policy on both sides
* [ ] Check return traffic

---

# 🔍 20. VDOM Troubleshooting Decision Tree

```text
Traffic Failure
      │
      ▼
Correct VDOM?
      │
 ┌────┴────┐
 NO        YES
 │          │
Fix        ▼
Context   Interface Correct?
             │
        ┌────┴────┐
       NO        YES
       │           │
     Fix           ▼
               Route Correct?
                    │
               ┌────┴────┐
              NO        YES
              │           │
            Fix           ▼
                       Policy?
                          │
                     ┌────┴────┐
                    NO        YES
                    │           │
                  Fix           ▼
                            VDOM Link?
                               │
                          ┌────┴────┐
                         NO        YES
                         │           │
                       Fix           ▼
                              Return Path?
```

---

# 🖥️ 21. VDOM Ping / Diagnostic Checklist

When executing diagnostic commands:

* [ ] Confirm the current VDOM
* [ ] Confirm the source interface
* [ ] Confirm the source IP
* [ ] Confirm the routing context
* [ ] Confirm that the diagnostic operation is being performed in the intended context

Conceptually:

```cli
config vdom
edit root
```

Then perform the appropriate diagnostic command.

### Mental Model

```text
GLOBAL
   ↓
FortiGate-wide Context

VDOM
   ↓
Firewall / Routing Context
```

---

# 🧭 22. VDOM Traffic Troubleshooting Order

Use this order:

```text
1. VDOM CONTEXT
       ↓
2. INTERFACE OWNERSHIP
       ↓
3. INTERFACE STATUS
       ↓
4. IP / VLAN
       ↓
5. ROUTING
       ↓
6. FIREWALL POLICY
       ↓
7. VDOM LINK
       ↓
8. NAT
       ↓
9. SECURITY INSPECTION
       ↓
10. SESSION
       ↓
11. RETURN PATH
```

### NSE 7 Principle

> **Do not troubleshoot the symptom before identifying the forwarding domain.**

---

# ⚠️ 23. Common VDOM Mistakes Checklist

### ❌ Mistake 1 — Wrong VDOM Context

* [ ] Verify current context before changing configuration
* [ ] Verify whether the command belongs to Global or VDOM scope

### ❌ Mistake 2 — Assuming VDOM Link = Trust

```text
VDOM Link
    ≠
Automatic Permit
```

* [ ] Configure required policies
* [ ] Verify routing

### ❌ Mistake 3 — Confusing VDOM and VRF

```text
VRF
→ Primarily Routing Separation

VDOM
→ Logical Firewall / Security Domain
```

* [ ] Understand that VDOM provides broader isolation than simple routing separation

### ❌ Mistake 4 — Ignoring Shared Resources

* [ ] Monitor CPU
* [ ] Monitor memory
* [ ] Monitor sessions
* [ ] Evaluate aggregate capacity

### ❌ Mistake 5 — Ignoring Return Traffic

* [ ] Verify forward path
* [ ] Verify reverse path
* [ ] Check asymmetric routing

### ❌ Mistake 6 — Giving Every Administrator Super-Admin Access

* [ ] Use least privilege
* [ ] Use scoped administrative access
* [ ] Review administrator permissions

---

# 🛡️ 24. VDOM Security Design Checklist

* [ ] Define security boundaries
* [ ] Define administrative boundaries
* [ ] Define routing boundaries
* [ ] Define interface ownership
* [ ] Restrict inter-VDOM traffic
* [ ] Apply least privilege
* [ ] Enable appropriate logging
* [ ] Monitor resource consumption
* [ ] Protect management access
* [ ] Review HA implications
* [ ] Review backup requirements
* [ ] Review upgrade impact
* [ ] Review compliance requirements

---

# 🔄 25. VDOM Change-Management Checklist

Before changing VDOM architecture:

* [ ] Identify affected VDOMs
* [ ] Backup configuration
* [ ] Document current topology
* [ ] Document interface ownership
* [ ] Document routing
* [ ] Document firewall policies
* [ ] Document VDOM Links
* [ ] Document administrators
* [ ] Evaluate service interruption
* [ ] Evaluate HA impact
* [ ] Evaluate rollback plan
* [ ] Test in a controlled environment
* [ ] Schedule maintenance window
* [ ] Validate services after change

---

# 🚨 26. VDOM Incident Response Checklist

If one VDOM is compromised:

* [ ] Identify affected VDOM
* [ ] Identify affected interfaces
* [ ] Identify affected administrators
* [ ] Review configuration changes
* [ ] Review firewall policy changes
* [ ] Review routing changes
* [ ] Review VPN configuration
* [ ] Review VDOM Link traffic
* [ ] Review logs
* [ ] Review management access
* [ ] Evaluate whether other VDOMs are affected
* [ ] Evaluate shared-resource impact
* [ ] Preserve relevant evidence
* [ ] Follow organizational incident-response procedures

### Important

```text
One VDOM Compromised
        ≠
Automatically All VDOMs Compromised

BUT

Shared Physical Platform
        ↓
Requires Broader Impact Assessment
```

---

# 🎯 27. NSE 4 Exam Checklist

You should be able to explain:

* [ ] What is VDOM?
* [ ] Why use VDOM?
* [ ] What is the root VDOM?
* [ ] What is Global context?
* [ ] What is VDOM context?
* [ ] What is a VDOM Link?
* [ ] How are interfaces associated with VDOMs?
* [ ] How is routing separated?
* [ ] How are firewall policies separated?
* [ ] How are administrators separated?
* [ ] What is multi-tenancy?
* [ ] Why does VDOM resource planning matter?

### NSE 4 Memory Model

```text
VDOM
 │
 ├── Virtual Firewall
 ├── Routing
 ├── Policies
 ├── Interfaces
 ├── Security
 └── Administration
```

---

# 🧠 28. NSE 7 Design Checklist

At NSE 7 level, evaluate:

* [ ] Security architecture
* [ ] Multi-tenancy
* [ ] Resource allocation
* [ ] Interface ownership
* [ ] Routing domains
* [ ] Inter-VDOM traffic
* [ ] VDOM Links
* [ ] Policy boundaries
* [ ] Administrative boundaries
* [ ] HA architecture
* [ ] Logging architecture
* [ ] Upgrade impact
* [ ] Disaster recovery
* [ ] Capacity planning
* [ ] Failure domains

### NSE 7 Mental Model

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

# 🏢 29. Real-World MSSP Architecture Checklist

### Customer Separation

* [ ] Customer A → VDOM-A
* [ ] Customer B → VDOM-B
* [ ] Customer C → VDOM-C
* [ ] Define customer-specific interfaces
* [ ] Define customer-specific policies
* [ ] Define customer-specific routing
* [ ] Define customer-specific administrators

```text
                         INTERNET
                            │
                       FORTIGATE
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
       VDOM-A            VDOM-B            VDOM-C
       Client-A          Client-B          Client-C
          │                 │                 │
       Routing            Routing           Routing
       Policy             Policy            Policy
       Security           Security          Security
```

### Administration

* [ ] Global administrator
* [ ] Customer administrator
* [ ] Least privilege
* [ ] Administrative audit trail
* [ ] Management-plane protection

---

# 📊 30. VDOM Architecture Reference

| Component            | Scope / Concept                       |
| -------------------- | ------------------------------------- |
| Global Configuration | FortiGate-wide configuration          |
| VDOM                 | Logical firewall/security domain      |
| Root VDOM            | Default VDOM in typical deployments   |
| Interface            | Owned/associated with a VDOM          |
| Firewall Policy      | VDOM-specific                         |
| Routing              | VDOM-specific                         |
| VDOM Link            | Logical inter-VDOM connectivity       |
| Super Administrator  | Broad/global administrative scope     |
| VDOM Administrator   | Restricted administrative scope       |
| Multi-VDOM           | Multiple logical firewall domains     |
| Physical Resources   | Shared by VDOMs on the same appliance |

---

# 🔬 31. VDOM Design Validation Checklist

Before approving a design:

```text
☐ VDOM purpose documented
☐ VDOM ownership documented
☐ Interface ownership documented
☐ Routing boundaries documented
☐ Firewall policy boundaries documented
☐ VDOM Links documented
☐ Inter-VDOM traffic documented
☐ Administrative boundaries documented
☐ Resource requirements documented
☐ HA design reviewed
☐ Logging design reviewed
☐ Backup strategy reviewed
☐ Upgrade impact reviewed
☐ Disaster recovery reviewed
☐ Capacity planning completed
```

---

# 🧪 32. Production Readiness Checklist

### Architecture

* [ ] VDOM architecture documented
* [ ] Traffic flows documented
* [ ] Security boundaries documented
* [ ] Failure domains documented

### Configuration

* [ ] Interfaces assigned correctly
* [ ] Routing validated
* [ ] Policies validated
* [ ] VDOM Links validated
* [ ] NAT validated
* [ ] Security profiles validated

### Administration

* [ ] Admin scope validated
* [ ] Least privilege applied
* [ ] Management access restricted
* [ ] Audit logging enabled

### Operations

* [ ] Monitoring configured
* [ ] Resource utilization monitored
* [ ] Configuration backup tested
* [ ] Recovery procedure documented
* [ ] Change-management process documented

---

# ⚡ 33. 60-Second VDOM Revision

```text
                    FORTIGATE
                        │
              ┌─────────┴─────────┐
              │                   │
           GLOBAL               VDOMs
                                  │
             ┌────────────────────┼────────────────────┐
             │                    │                    │
           VDOM-A              VDOM-B              VDOM-C
             │                    │                    │
          Routing              Routing              Routing
          Policy               Policy               Policy
          Security             Security             Security
          Admin                Admin                Admin
             │                    │                    │
             └──────────── VDOM LINK ──────────────────┘
```

### Remember

```text
GLOBAL
   =
FortiGate-wide Scope

VDOM
   =
Logical Firewall Scope

VDOM LINK
   =
Inter-VDOM Connectivity

ROUTING
   =
VDOM-specific

POLICY
   =
VDOM-specific

ADMIN
   =
Global or VDOM-scoped
```

---

# 🧠 34. Golden Rules

```text
VDOM
=
Virtual Firewall / Security Domain

ONE FORTIGATE
=
MULTIPLE LOGICAL FIREWALLS

VDOM
≠
VRF

VDOM LINK
≠
AUTOMATIC TRUST

GLOBAL
≠
VDOM

VDOM-A ROUTING
≠
VDOM-B ROUTING

VDOM-A POLICY
≠
VDOM-B POLICY

LOGICAL ISOLATION
≠
PHYSICAL ISOLATION

MULTIPLE VDOMs
=
SHARED PHYSICAL RESOURCES

NSE 7
=
UNDERSTAND THE FORWARDING DOMAIN FIRST
```

---

# 🏁 35. Final Engineer Mental Model

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
                Admin            Admin            Admin
                   │                │                │
                   └─────── VDOM LINK / CONTROL ────┘
```

> **Engineer Rule:**
> **Before troubleshooting a VDOM environment, identify the VDOM, interface ownership, routing domain, policy domain, and inter-VDOM path.**

The difference between basic configuration knowledge and advanced FortiGate engineering is understanding **where the traffic lives, who controls it, how it is forwarded, and which security boundary it crosses**.

---

# 🔖 Keywords

`FortiGate VDOM`
`FortiOS VDOM`
`Fortinet VDOM`
`FortiGate Virtual Domain`
`FortiGate Multi VDOM`
`FortiGate VDOM Link`
`FortiGate Global vs VDOM`
`FortiGate VDOM Administration`
`FortiGate Multi Tenancy`
`FortiGate Multi Tenant Firewall`
`FortiGate VDOM Routing`
`FortiGate VDOM Firewall Policy`
`FortiGate VDOM Troubleshooting`
`FortiGate VDOM Configuration`
`FortiGate VDOM Architecture`
`FortiGate VDOM Security`
`FortiGate VDOM Resource Allocation`
`FortiGate Inter VDOM Communication`
`FortiGate VDOM Link Troubleshooting`
`Fortinet NSE4 VDOM`
`Fortinet NSE7 VDOM`
`FortiGate NSE4 VDOM`
`FortiGate NSE7 VDOM`
`FortiGate Firewall Architecture`
`Fortinet Multi Tenant Firewall`
`FortiGate Firewall Segmentation`
`FortiGate Logical Firewall`
`FortiGate Virtual Firewall`

---

# 🔗 SheynShield Resources

## 🎥 Video Learning

* [ ] [YouTube — SheynShield](https://youtube.com/@sheynshield)

  * Fortinet NSE content
  * FortiGate troubleshooting
  * Network Security Engineering

## 📚 Notes & Updates

* [ ] [Telegram — SheynShield](https://t.me/sheynshield)

## 💼 Professional Network

* [ ] [LinkedIn — Shayan-heydarikhah](https://linkedin.com/in/shayan-heydarikhah)

## 🐙 Technical Knowledge Base

* [ ] [SheynShield GitHub](https://github.com/Shayan-heydarikhah/sheynshield)

---

> **SheynShield | Engineering Secure Networks**
>
> **FortiGate • FortiOS • VDOM • Network Security • Firewall Engineering • NSE 4 • NSE 7**

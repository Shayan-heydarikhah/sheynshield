# FortiGate Device Operation & Inspection Mode Checklist

> **SheynShield | Engineering Secure Networks**
>
> **FortiGate • FortiOS • NAT Mode • Transparent Mode • Layer 2 • Layer 3 • VLAN • Forward Domain • STP • Flow-Based • Proxy-Based • SNMP**
>
> **NSE 4 / NSE 7 Operational & Troubleshooting Checklist**

---

## 🎯 FortiGate Operation Mode Checklist

### Core Architecture

* [ ] Identify whether FortiGate is operating in **NAT mode** or **Transparent mode**
* [ ] Confirm the selected operation mode matches the network design
* [ ] Document the expected traffic flow
* [ ] Identify whether the primary forwarding plane is **Layer 2** or **Layer 3**
* [ ] Document the management path separately from production traffic
* [ ] Verify the FortiOS version before relying on historical CLI syntax
* [ ] Check model-specific limitations before deployment

### Quick Decision

```text
FortiGate
    │
    ├── NAT Mode
    │      └── Layer 3
    │           ├── IP
    │           ├── Routing
    │           ├── Policy
    │           └── NAT
    │
    └── Transparent Mode
           └── Layer 2
                ├── MAC
                ├── VLAN
                ├── ARP
                ├── Forward Domain
                └── STP
```

---

# 🌐 NAT Mode Checklist

## Architecture

* [ ] Confirm NAT mode is required
* [ ] Verify interface IP addressing
* [ ] Verify subnet masks
* [ ] Verify default route
* [ ] Verify routing table
* [ ] Verify firewall policies
* [ ] Determine whether source NAT is required
* [ ] Verify return traffic path
* [ ] Validate asymmetric-routing considerations
* [ ] Confirm security profiles are attached where required

### NAT Mode Mental Model

```text
Ingress
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
Egress
```

### Engineering Questions

* [ ] Where should the packet go?
* [ ] Which route is selected?
* [ ] Which firewall policy matches?
* [ ] Is NAT required?
* [ ] Which security profiles inspect the session?
* [ ] Does return traffic follow the expected path?

> **Golden Rule:**
> **Routing = WHERE?**
> **Firewall Policy = ALLOW or DENY?**
> **NAT = WHAT address translation?**

---

# 🌉 Transparent Mode Checklist

## Architecture

* [ ] Confirm Transparent mode is intentional
* [ ] Verify FortiGate is inline with the Layer 2 topology
* [ ] Identify connected switches
* [ ] Identify VLANs traversing the FortiGate
* [ ] Document Layer 2 forwarding requirements
* [ ] Document management addressing
* [ ] Verify ARP behavior
* [ ] Verify MAC learning
* [ ] Verify VLAN configuration
* [ ] Verify forwarding domains
* [ ] Check STP requirements
* [ ] Check for Layer 2 loops

### Transparent Mode Mental Model

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
Forward Domain
   ↓
L2 Forwarding
   ↓
STP
   ↓
Firewall Policy
   ↓
Security Inspection
```

### Transparent Mode Troubleshooting

* [ ] Interface is physically UP
* [ ] Link speed/duplex is correct
* [ ] Expected MAC addresses are learned
* [ ] VLAN tags are correct
* [ ] Interfaces belong to the expected forwarding domain
* [ ] Required Layer 2 forwarding is enabled
* [ ] STP behavior is understood
* [ ] No Layer 2 loop exists
* [ ] Firewall policy matches
* [ ] Security inspection is not dropping traffic

---

# 🔀 Forward Domain Checklist

`forward-domain` is a **Layer 2 forwarding separation mechanism**.

### Verify

* [ ] Identify the forwarding domain assigned to each interface
* [ ] Confirm interfaces that should communicate share the correct forwarding domain
* [ ] Confirm isolated interfaces belong to different forwarding domains
* [ ] Check VLAN-to-forward-domain relationships
* [ ] Verify forwarding behavior after configuration changes

### Example

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
end
```

### Remember

```text
Forward Domain
      ≠
VDOM

Forward Domain
      ≠
Firewall Policy
```

* [ ] Do not use a forwarding domain as a replacement for security policy
* [ ] Do not confuse Layer 2 forwarding separation with administrative VDOM separation

---

# 🧩 VLAN Forwarding Checklist

### Design

* [ ] Identify physical interfaces carrying VLANs
* [ ] Identify VLAN IDs
* [ ] Verify tagging/untagging behavior
* [ ] Review VLAN forwarding behavior
* [ ] Verify expected VLAN isolation
* [ ] Verify inter-VLAN traffic requirements
* [ ] Confirm firewall policy remains the intended security boundary

### Example

```cli
config system interface
    edit "port1"
        set vlanforward disable
    next
end
```

### Validate

* [ ] Understand the traffic path before changing `vlanforward`
* [ ] Verify whether Layer 2 forwarding between VLAN interfaces is expected
* [ ] Do not use `vlanforward` as a replacement for proper segmentation
* [ ] Validate the configuration against the target FortiOS release

> ⚠️ **Version Warning:** VLAN forwarding behavior and CLI availability can vary between FortiOS releases.

---

# 🌲 STP Forwarding Checklist

Use STP considerations when FortiGate participates in a Layer 2 topology.

### Verify

* [ ] Determine whether STP must traverse the FortiGate
* [ ] Identify all Layer 2 paths
* [ ] Check for redundant links
* [ ] Identify the STP root
* [ ] Verify STP forwarding requirements
* [ ] Check for potential Layer 2 loops
* [ ] Verify the expected topology after enabling forwarding

Example:

```cli
config system interface
    edit "INTERFACE"
        set l2forward enable
        set stpforward enable
    next
end
```

### Loop Prevention Mental Model

```text
Switch A
   │
   ▼
FortiGate
   │
   ▼
Switch B
   │
   └──────────────► Switch A

        ↓
     L2 LOOP
```

* [ ] Never enable Layer 2 protocol forwarding blindly
* [ ] Understand the security and loop implications first

---

# 🔧 Layer 2 Forwarding Checklist

### Before enabling

* [ ] Identify the exact Layer 2 protocol
* [ ] Identify why the protocol must traverse FortiGate
* [ ] Identify affected interfaces
* [ ] Confirm the protocol is supported
* [ ] Evaluate loop risk
* [ ] Evaluate security implications
* [ ] Test in a controlled environment

Example:

```cli
config system interface
    edit "port1"
        set l2forward enable
    next
end
```

### Troubleshooting order

```text
Physical
   ↓
Interface
   ↓
MAC
   ↓
VLAN
   ↓
ARP
   ↓
Forward Domain
   ↓
L2 Forwarding
   ↓
STP
   ↓
Policy
```

---

# 🖥️ Transparent Mode Management Checklist

* [ ] Define the management subnet
* [ ] Define the management IP
* [ ] Define the management gateway
* [ ] Verify administrator reachability
* [ ] Separate management traffic from production traffic where practical
* [ ] Restrict administrative access
* [ ] Verify HTTPS/SSH administrative access requirements

Historical-style example:

```cli
config system settings
    set opmode transparent
    set manageip 192.168.20.250 255.255.255.0
    set gateway 192.168.20.254
end
```

> ⚠️ Treat this as a **version-sensitive example**. Verify the syntax against the FortiOS release deployed in the environment.

---

# 🌐 NAT Mode Interface Checklist

Modern FortiOS deployments generally configure interface addressing under:

```cli
config system interface
    edit "port1"
        set ip 192.168.20.252 255.255.255.0
    next
end
```

### Verify

* [ ] Interface exists
* [ ] Interface is administratively enabled
* [ ] IP address is correct
* [ ] Netmask is correct
* [ ] VLAN configuration is correct
* [ ] Administrative access is intentionally enabled
* [ ] Routing references the correct interface

---

# 📡 NetBIOS / WINS Checklist

Legacy environments may require NetBIOS/WINS forwarding.

Example:

```cli
config system interface
    edit "internal"
        set netbios-forward enable
        set wins-ip 192.168.111.222
    next
end
```

### Validate

* [ ] Confirm the environment actually requires NetBIOS/WINS
* [ ] Identify the WINS server
* [ ] Verify the relevant interface
* [ ] Confirm the feature is supported on the target FortiOS release
* [ ] Avoid enabling legacy functionality without a documented requirement

---

# 🔬 FortiGate Inspection Mode Checklist

FortiGate security inspection commonly involves two major architectures:

```text
Inspection
    │
    ├── Flow-Based
    │
    └── Proxy-Based
```

### Before selecting inspection mode

* [ ] Identify the security feature
* [ ] Check supported inspection mode
* [ ] Check FortiOS version
* [ ] Check model capabilities
* [ ] Evaluate performance requirements
* [ ] Evaluate buffering requirements
* [ ] Consider SSL/TLS visibility
* [ ] Consider logging requirements
* [ ] Validate security profile compatibility

---

# ⚡ Flow-Based Inspection Checklist

### Characteristics

* [ ] Understand stream-oriented processing
* [ ] Evaluate performance requirements
* [ ] Confirm the required security feature supports flow-based inspection
* [ ] Validate security profile compatibility
* [ ] Check SSL inspection requirements
* [ ] Check CPU/resource utilization

### Mental Model

```text
Traffic
   ↓
Inspect
   ↓
Decision
   ↓
Forward
```

> **Memory:** Flow-based inspection emphasizes processing traffic as it passes through the inspection engine.

---

# 🧪 Proxy-Based Inspection Checklist

### Characteristics

* [ ] Confirm proxy-based inspection is supported
* [ ] Understand buffering requirements
* [ ] Evaluate memory/resource impact
* [ ] Validate supported security features
* [ ] Consider SSL inspection requirements
* [ ] Validate application compatibility

### Mental Model

```text
Client
   ↓
FortiGate Proxy
   ↓
Receive
   ↓
Inspect
   ↓
Process / Buffer
   ↓
Forward
   ↓
Server
```

---

# 🆚 Flow-Based vs Proxy-Based Checklist

| Checkpoint                          | Flow-Based | Proxy-Based |
| ----------------------------------- | ---------- | ----------- |
| Stream-oriented processing          | [x]        | [ ]         |
| Proxy/intermediary processing       | [ ]        | [x]         |
| Lower buffering requirement         | [x]        | [ ]         |
| Potentially higher resource usage   | [ ]        | [x]         |
| Feature compatibility verified      | [ ]        | [ ]         |
| SSL inspection requirement reviewed | [ ]        | [ ]         |
| Performance impact evaluated        | [ ]        | [ ]         |
| FortiOS version verified            | [ ]        | [ ]         |

> ⚠️ Exact feature availability and behavior are **FortiOS-version dependent**.

---

# 🌐 Explicit Proxy Checklist

* [ ] Determine whether explicit proxy is required
* [ ] Configure client proxy settings
* [ ] Verify authentication requirements
* [ ] Verify web filtering requirements
* [ ] Verify logging requirements
* [ ] Review SSL inspection
* [ ] Verify policy behavior
* [ ] Evaluate resource impact
* [ ] Test client compatibility

### Traffic Model

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

---

# 💾 WAN Optimization & Cache Checklist

Where supported by the FortiOS release and platform:

* [ ] Identify the WAN optimization requirement
* [ ] Identify repeated-data workloads
* [ ] Check platform support
* [ ] Check FortiOS support
* [ ] Evaluate resource requirements
* [ ] Validate cache behavior
* [ ] Measure performance before/after deployment

```text
Branch A
   ↓
FortiGate
   ↓
Optimized Traffic
   ↓
WAN
   ↓
FortiGate
   ↓
Branch B
```

---

# 🛡️ DLP Inspection Checklist

### Visibility

* [ ] Identify whether traffic is encrypted
* [ ] Determine whether SSL inspection is required
* [ ] Confirm FortiGate can see the relevant content
* [ ] Verify inspection mode
* [ ] Verify DLP profile
* [ ] Confirm DLP profile is attached to the correct policy
* [ ] Check logs for DLP events

### Troubleshooting

```text
Traffic
   ↓
Encrypted?
   │
   ├── YES → SSL Inspection?
   │
   └── NO
        ↓
Inspection Mode
        ↓
Content Visibility
        ↓
DLP Engine
        ↓
Policy Decision
```

---

# 📊 SNMP Checklist

## SNMP Monitoring

* [ ] Identify the SNMP manager
* [ ] Restrict SNMP access to trusted management systems
* [ ] Select the appropriate SNMP version
* [ ] Prefer SNMPv3 where supported and appropriate
* [ ] Configure authentication where required
* [ ] Configure privacy/encryption where required
* [ ] Verify UDP/161 reachability
* [ ] Test polling
* [ ] Validate returned OIDs/information

### SNMP GET

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

**UDP/161 = Query / Polling**

---

# 🚨 SNMP Trap Checklist

* [ ] Identify SNMP trap manager
* [ ] Configure trap destination
* [ ] Verify UDP/162
* [ ] Test event generation
* [ ] Confirm trap reception
* [ ] Validate monitoring-server logs

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

**UDP/162 = Trap / Notification**

### Memory Trick

```text
161 = ASK
162 = ALERT
```

---

# 🔐 SNMP Security Checklist

* [ ] Avoid unnecessary SNMP exposure
* [ ] Restrict manager source IPs
* [ ] Avoid weak/default community strings
* [ ] Prefer SNMPv3 where supported
* [ ] Configure authentication
* [ ] Configure privacy/encryption where supported
* [ ] Place monitoring traffic in the management architecture
* [ ] Monitor unexpected SNMP activity

---

# 🏢 Management Architecture Checklist

A secure enterprise design should logically separate management traffic from production traffic where practical.

```text
                    ADMIN
                      │
                      ▼
                Management VLAN
                      │
                      ▼
                   FortiGate
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
        HTTPS        SSH         SNMP
          │           │           │
          ▼           ▼           ▼
        Admin       Admin      Monitoring
```

### Verify

* [ ] Management VLAN defined
* [ ] Management source networks restricted
* [ ] HTTPS access restricted
* [ ] SSH access restricted
* [ ] SNMP access restricted
* [ ] Monitoring servers documented
* [ ] Administrative access logged

---

# 🔍 FortiGate Hardware / Software Inspection Checklist

Before troubleshooting:

* [ ] Confirm FortiGate model
* [ ] Confirm FortiOS version
* [ ] Confirm VDOM architecture
* [ ] Identify physical interfaces
* [ ] Identify VLAN interfaces
* [ ] Identify aggregate interfaces
* [ ] Identify zones
* [ ] Identify virtual interfaces
* [ ] Identify transparent/routed architecture
* [ ] Identify relevant security profiles
* [ ] Identify management path

### Configuration Tree Discovery

Useful when exploring the available configuration tree:

```bash
tree system interface
```

---

# 🧪 FortiGate Troubleshooting Workflow

## General Workflow

* [ ] Check physical connectivity
* [ ] Check interface state
* [ ] Check Layer 2 behavior
* [ ] Check VLAN
* [ ] Check MAC learning
* [ ] Check ARP
* [ ] Check routing
* [ ] Check firewall policy
* [ ] Check NAT
* [ ] Check session
* [ ] Check inspection mode
* [ ] Check security profile
* [ ] Check return traffic
* [ ] Check logs

```text
PHYSICAL
   ↓
INTERFACE
   ↓
L2 / VLAN
   ↓
MAC / ARP
   ↓
ROUTING
   ↓
FIREWALL POLICY
   ↓
NAT
   ↓
INSPECTION
   ↓
SECURITY PROFILE
   ↓
SESSION
   ↓
RETURN TRAFFIC
   ↓
LOGS
```

---

# 🌉 Transparent Mode Troubleshooting Workflow

Use a different mindset when FortiGate is operating at Layer 2.

* [ ] Physical link checked
* [ ] Interface state checked
* [ ] MAC table checked
* [ ] VLAN checked
* [ ] ARP behavior checked
* [ ] Forward domain checked
* [ ] L2 forwarding checked
* [ ] STP checked
* [ ] Firewall policy checked
* [ ] Security inspection checked

```text
Physical
   ↓
MAC
   ↓
VLAN
   ↓
ARP
   ↓
Forward Domain
   ↓
L2 Forwarding
   ↓
STP
   ↓
Policy
   ↓
Inspection
```

---

# 🔎 Packet Troubleshooting Matrix

| Symptom                           | First Investigation Area         |
| --------------------------------- | -------------------------------- |
| No connectivity in NAT mode       | Routing                          |
| Correct route but traffic blocked | Firewall Policy                  |
| VLAN traffic unavailable          | VLAN / Interface                 |
| ARP failure                       | L2 / VLAN / Transparent Mode     |
| Unexpected L2 forwarding          | Forward Domain / VLAN Forwarding |
| Layer 2 loop                      | STP                              |
| Web traffic unexpectedly blocked  | Web Filter / SSL Inspection      |
| Content inspection failure        | Inspection Mode / Visibility     |
| SNMP polling unavailable          | SNMP / UDP 161                   |
| SNMP traps unavailable            | SNMP / UDP 162                   |
| High CPU during inspection        | Security Profiles / Inspection   |
| Unexpected proxy behavior         | Explicit Proxy / Policy          |

---

# 🧠 NSE 4 Checklist

## Must Know

* [ ] NAT Mode
* [ ] Transparent Mode
* [ ] Layer 2 vs Layer 3
* [ ] Management IP
* [ ] Interface addressing
* [ ] Routing
* [ ] Firewall policy
* [ ] NAT
* [ ] VLAN interfaces
* [ ] Layer 2 forwarding
* [ ] Forward domains
* [ ] STP forwarding
* [ ] Flow-based inspection
* [ ] Proxy-based inspection
* [ ] Explicit proxy basics
* [ ] SNMP
* [ ] UDP/161
* [ ] UDP/162

---

# 🧠 NSE 7 Troubleshooting Checklist

At advanced level, avoid starting with:

```text
"Which command should I run?"
```

Start with:

```text
What is the architecture?
        ↓
L2 or L3?
        ↓
Where does the packet enter?
        ↓
How is the packet forwarded?
        ↓
Which route is selected?
        ↓
Which policy matches?
        ↓
Is NAT involved?
        ↓
Which inspection engine processes it?
        ↓
Could VLAN / MAC / ARP / STP affect it?
        ↓
Could security inspection drop it?
        ↓
How can I prove the behavior?
```

### NSE 7 Engineering Checklist

* [ ] Identify the failure domain
* [ ] Build the expected traffic path
* [ ] Compare expected vs actual behavior
* [ ] Validate Layer 2 dependencies
* [ ] Validate Layer 3 dependencies
* [ ] Validate policy matching
* [ ] Validate NAT
* [ ] Validate inspection architecture
* [ ] Validate security-profile behavior
* [ ] Validate session state
* [ ] Validate return traffic
* [ ] Correlate configuration with logs
* [ ] Prove the root cause instead of guessing

---

# ⚠️ High-Value Exam Traps

### Trap 1 — NAT ≠ Transparent

```text
NAT Mode
    → Layer 3

Transparent Mode
    → Layer 2
```

* [ ] Do not troubleshoot Transparent mode as if it were a normal routed firewall
* [ ] Do not begin with routing when the failure is clearly Layer 2

---

### Trap 2 — Forward Domain ≠ VDOM

```text
Forward Domain
    → Layer 2 forwarding separation

VDOM
    → Virtual administrative / security domain
```

* [ ] Keep the two concepts separate

---

### Trap 3 — STP ≠ Routing

```text
STP
    → Layer 2 loop prevention

Routing
    → Layer 3 path selection
```

* [ ] Determine the OSI layer before choosing the troubleshooting tool

---

### Trap 4 — Inspection Mode ≠ Security Profile

```text
Inspection Mode
      ↓
How traffic is inspected

Security Profile
      ↓
What security control is applied
```

* [ ] Verify compatibility between the security feature and inspection mode

---

### Trap 5 — SNMP 161 vs 162

```text
UDP/161
   → Polling / Query

UDP/162
   → Trap / Notification
```

* [ ] Remember **161 = ASK**
* [ ] Remember **162 = ALERT**

---

### Trap 6 — Internet Connectivity Does Not Prove Application Health

A working network path does not automatically prove that:

```text
Application
Security Profile
Inspection Engine
Proxy
Logging
SNMP
```

are functioning correctly.

* [ ] Validate each layer independently

---

# 🛡️ Production Configuration Checklist

## Operation Mode

* [ ] NAT vs Transparent decision documented
* [ ] Network topology documented
* [ ] Management architecture documented
* [ ] FortiOS version recorded
* [ ] FortiGate model recorded

## Layer 2

* [ ] VLANs documented
* [ ] MAC behavior validated
* [ ] ARP behavior validated
* [ ] Forward domains documented
* [ ] L2 forwarding requirements documented
* [ ] STP requirements documented
* [ ] Loop prevention validated

## Layer 3

* [ ] Interface addressing validated
* [ ] Routing validated
* [ ] Default route validated
* [ ] Firewall policies validated
* [ ] NAT requirements documented
* [ ] Return path validated

## Inspection

* [ ] Flow vs Proxy selected intentionally
* [ ] Security feature compatibility checked
* [ ] SSL inspection requirements reviewed
* [ ] Performance impact evaluated
* [ ] Logging requirements documented
* [ ] Resource utilization monitored

## SNMP

* [ ] SNMP version selected
* [ ] SNMPv3 evaluated/preferred where supported
* [ ] Manager source restricted
* [ ] UDP/161 tested
* [ ] UDP/162 tested where traps are required
* [ ] Monitoring verified

---

# ⚡ Golden Rules

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

NAT
    =
ADDRESS TRANSLATION

Forward Domain
    =
L2 Forwarding Separation

STP
    =
L2 Loop Prevention

Flow-Based
    =
Inspect Traffic in Flow

Proxy-Based
    =
Proxy + Inspect + Forward

SNMP 161
    =
QUERY

SNMP 162
    =
TRAP
```

---

# 🧠 60-Second Revision

```text
                         FORTIGATE
                             │
              ┌──────────────┴──────────────┐
              │                             │
              ▼                             ▼
          NAT MODE                    TRANSPARENT MODE
          Layer 3                       Layer 2
              │                             │
       Route / Policy / NAT          MAC / VLAN / STP
              │                             │
              └──────────────┬──────────────┘
                             ▼
                      SECURITY POLICY
                             │
                             ▼
                    INSPECTION ENGINE
                       │            │
                       ▼            ▼
                     FLOW         PROXY
                       │            │
                       └─────┬──────┘
                             ▼
                          LOGGING
                             │
                             ▼
                            SNMP
                         161 / 162
```

---

# 🏁 Final Engineering Mental Model

The first question should always be:

```text
WHERE DOES THE PACKET LIVE?
```

Then determine:

```text
LAYER 2
   ↓
MAC
VLAN
ARP
Forward Domain
STP
```

or:

```text
LAYER 3
   ↓
IP
Route
Policy
NAT
```

Then:

```text
SECURITY INSPECTION
        ↓
   ┌────┴────┐
   ▼         ▼
 FLOW      PROXY
```

Finally:

```text
LOGGING
   ↓
MONITORING
   ↓
SNMP
```

### The SheynShield Troubleshooting Principle

```text
Architecture
    ↓
Forwarding Plane
    ↓
Security Policy
    ↓
Inspection Engine
    ↓
Session
    ↓
Logs
    ↓
Proof of Root Cause
```

> **Understand the forwarding plane first. Then understand the security plane. Finally understand the inspection engine.**

That is the difference between **FortiGate configuration knowledge** and **FortiGate engineering-level troubleshooting**.

---

# 🔖 Keywords

`FortiGate Operation Mode` · `FortiGate NAT Mode` · `FortiGate Transparent Mode` · `FortiGate Layer 2` · `FortiGate Layer 3` · `FortiGate Forward Domain` · `FortiGate VLAN Forwarding` · `FortiGate STP Forwarding` · `FortiGate ARP Troubleshooting` · `FortiGate Flow Based Inspection` · `FortiGate Proxy Based Inspection` · `FortiGate Inspection Mode` · `FortiGate SNMP` · `FortiGate SNMP 161` · `FortiGate SNMP Trap 162` · `FortiGate Troubleshooting` · `FortiGate Layer 2 Troubleshooting` · `FortiGate Layer 3 Troubleshooting` · `FortiOS` · `Fortinet NSE4` · `Fortinet NSE7` · `FortiGate NSE4` · `FortiGate NSE7`

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

**SheynShield | Engineering Secure Networks**

> **Learn the architecture. Trace the packet. Prove the failure.**

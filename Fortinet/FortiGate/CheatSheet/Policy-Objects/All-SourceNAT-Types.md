# FortiGate Advanced SNAT, IP Pools, Central NAT & CGNAT  

> **FortiOS Networking / NAT Reference**
> Advanced source NAT, IP pool types, deterministic port allocation, Central SNAT, Virtual Wire Pair and CGNAT design.

---

## 1. Source NAT Architecture

Source NAT changes the **source IP/port** of outbound sessions.

```text
Inside Client
192.168.101.10:52341
        |
        | SNAT
        v
FortiGate
        |
        v
Public / Translated
192.168.254.252:5377
```

FortiGate maintains the translation through the **session / xlate / NAT state**.

### Basic Concept

```text
Original:

SRC = 192.168.101.10
SPORT = 52341

        ↓ SNAT

Translated:

SRC = 192.168.254.252
SPORT = 5377
```

The translated tuple must remain unique for simultaneous sessions.

---

# 2. Source Port Preservation

Some applications are sensitive to source-port changes.

Typical examples:

* SIP / VoIP
* Legacy applications
* Applications that track sessions using source IP + source port
* Certain proprietary protocols

Conceptually:

```text
Client:
10.10.10.10:5060

SNAT:

203.0.113.10:5060
```

instead of:

```text
203.0.113.10:49152
```

> ⚠️ Source-port preservation is not universally possible. If the translated tuple conflicts with another active session, FortiGate must maintain uniqueness.

---

# 3. IP Pool Types

FortiGate supports several IP Pool mechanisms:

| Type                      | Main Purpose                              | PAT | Port Allocation |
| ------------------------- | ----------------------------------------- | --: | --------------- |
| **Overload**              | Normal Internet SNAT                      | Yes | Dynamic         |
| **One-to-One**            | Dedicated address mapping                 |  No | 1:1             |
| **Fixed Port Range**      | Deterministic IP/port mapping             | Yes | Fixed           |
| **Port Block Allocation** | CGNAT / scalable deterministic allocation | Yes | Block-based     |

Fortinet documents these four major IPv4 IP Pool types.

---

# 4. Overload IP Pool

## Use Case

The most common SNAT design.

Multiple internal clients share one or more public IP addresses.

```text
10.10.10.10 ─┐
10.10.10.11 ─┤
10.10.10.12 ─┼──> FortiGate ──> 203.0.113.10
10.10.10.13 ─┘
```

### CLI

```bash
config firewall ippool
    edit "SNAT-OVERLOAD"
        set type overload
        set startip 192.168.254.252
        set endip 192.168.254.252
    next
end
```

### Policy

```bash
config firewall policy
    edit 1
        set srcintf "LAN"
        set dstintf "WAN"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
        set nat enable
        set ippool enable
        set poolname "SNAT-OVERLOAD"
    next
end
```

### Mental Model

```text
Many Private IPs
       |
       | PAT
       v
One Public IP
+
Many Source Ports
```

---

# 5. Available Port Capacity

A useful FortiGate NAT planning value is:

```text
60416 usable ports / IPv4 address
```

Do **not** simply assume:

```text
65536 usable NAT ports
```

because FortiGate reserves portions of the port space for NAT allocation.

### Example

One public IP:

```text
203.0.113.10
```

Approximate available NAT capacity:

```text
60416 translations
```

Two public IPs:

```text
60416 × 2
=
120832 available port slots
```

> Capacity is theoretical and actual usable concurrency depends on protocol, timeout, session behavior and the specific FortiOS/platform implementation.

---

# 6. Fixed Port Range

Fixed Port Range provides deterministic mapping between:

```text
Internal IP range
        +
External IP range
        +
Port range
```

Fortinet describes this mode as PAT where both the internal source range and external range can be defined.

### Example

```bash
config firewall ippool
    edit "FPR-POOL"
        set type fixed-port-range
        set startip 192.168.254.252
        set endip 192.168.254.252
        set source-startip 192.168.101.1
        set source-endip 192.168.101.10
    next
end
```

Conceptually:

```text
192.168.101.1  ---> 192.168.254.252 : Port Range A
192.168.101.2  ---> 192.168.254.252 : Port Range B
192.168.101.3  ---> 192.168.254.252 : Port Range C
...
```

This is useful when deterministic source-port allocation is required.

---

# 7. Fixed Port Range — Capacity Calculation

Assume:

```text
Available ports = 60416

Internal IPs = 10
```

Then approximately:

```text
60416 / 10
≈ 6041 ports per internal IP
```

The exact allocation behavior should be verified against the target FortiOS release when designing production CGNAT.

---

# 8. Port Block Allocation (PBA)

Port Block Allocation is especially useful for:

* CGNAT
* Large subscriber populations
* Deterministic port allocation
* Subscriber traceability
* Controlling per-user NAT capacity

Fortinet supports configurable:

```text
Block Size
Blocks Per User
PBA Timeout
```

The documented block-size range is **64–4096**, while blocks-per-user can range from **1–128** on relevant FortiOS versions.

---

## PBA Example

```text
External IP:

203.0.113.10
```

Configuration:

```text
Block Size       = 128
Blocks/User      = 8
```

Maximum ports per user:

```text
128 × 8

= 1024 ports/user
```

Total available blocks:

```text
60416 / 128

≈ 472 blocks
```

Approximate users per public IP:

```text
60416 / 1024

≈ 59 users
```

Therefore:

```text
1 Public IP
│
├── User 1 → 1024 ports
├── User 2 → 1024 ports
├── User 3 → 1024 ports
│
└── ...
    ≈ 59 users
```

Fortinet's own example uses the same:

```text
60416 / 128 = 472 blocks
128 × 8 = 1024 ports/user
60416 / 1024 ≈ 59 users
```

---

# 9. PBA CLI Example

```bash
config firewall ippool
    edit "CGNAT-PBA"
        set type port-block-allocation
        set startip 192.168.254.252
        set endip 192.168.254.252
        set block-size 128
        set num-blocks-per-user 8
    next
end
```

Fortinet documents this configuration pattern for PBA.

---

# 10. PBA Design Formula

### Maximum ports per subscriber

```text
Ports/User =
Block Size × Blocks/User
```

### Number of blocks

```text
Blocks =
Available Ports / Block Size
```

### Approximate subscribers per public IP

```text
Subscribers =
Available Ports / Ports/User
```

### Example

```text
Available Ports = 60416
Block Size      = 256
Blocks/User     = 4
```

Therefore:

```text
Ports/User
= 256 × 4
= 1024
```

and:

```text
Subscribers
≈ 60416 / 1024
≈ 59
```

---

# 11. CGNAT Planning

A common CGNAT architecture:

```text
                     Internet
                         |
                  Public IP Pool
                         |
                    FortiGate
                         |
        +----------------+----------------+
        |                |                |
     User A           User B           User C
 10.10.1.10        10.10.1.11        10.10.1.12
```

For carrier environments, the shared address space:

```text
100.64.0.0/10
```

is commonly used between the subscriber network and CGNAT.

> ⚠️ `100.64.0.0/10` is RFC 6598 Shared Address Space. It is not a public Internet address range.

---

# 12. One-to-One NAT

One-to-One provides dedicated mapping.

```text
Private              Public

10.10.10.10  <----> 203.0.113.10
10.10.10.11  <----> 203.0.113.11
10.10.10.12  <----> 203.0.113.12
```

### CLI

```bash
config firewall ippool
    edit "ONE-TO-ONE"
        set type one-to-one
        set startip 203.0.113.10
        set endip 203.0.113.12
    next
end
```

One-to-One disables PAT; each translated address maps to one internal address.

---

# 13. ARP Reply on IP Pools

IP Pool addresses may need ARP handling when they exist on the directly connected network.

Relevant setting:

```bash
config firewall ippool
    edit "SNAT-POOL"
        set arp-reply enable
        set arp-intf "wan1"
    next
end
```

### Concept

```text
ISP
 |
 | ARP?
 v
FortiGate
 |
 +--> IP Pool address
```

If the upstream network expects the FortiGate to answer ARP for the pool addresses, verify:

```text
arp-reply
arp-intf
associated-interface
```

---

# 14. Central NAT

Central NAT moves source NAT logic out of individual firewall policies.

Enable:

```bash
config system settings
    set central-nat enable
end
```

For a VDOM-specific configuration:

```bash
config vdom
    edit "vd-test"

    config system settings
        set central-nat enable
    end

end
```

Architecture:

```text
Firewall Policy
      |
      | Security decision
      v
Central SNAT
      |
      | Source translation
      v
IP Pool / Interface IP
```

---

# 15. Central SNAT Map

Example:

```bash
config firewall central-snat-map
    edit 1
        set srcintf "LAN"
        set dstintf "WAN"
        set orig-addr "all"
        set dst-addr "all"
        set nat enable
    next
end
```

With an IP Pool:

```bash
config firewall central-snat-map
    edit 1
        set srcintf "LAN"
        set dstintf "WAN"
        set orig-addr "all"
        set dst-addr "all"
        set nat enable
        set nat-ippool "SNAT-OVERLOAD"
    next
end
```

> **Important:** Central SNAT is a separate NAT decision layer from the normal firewall policy. Do not mentally treat `central-snat-map` as a replacement for the security policy itself.

---

# 16. Central SNAT + Source Port Matching

Central SNAT can match source-port conditions.

Example:

```bash
config firewall central-snat-map
    edit 1
        set srcintf "LAN"
        set dstintf "WAN"
        set orig-addr "all"
        set dst-addr "all"
        set origin-port 100-2000
        set nat enable
    next
end
```

Useful for environments such as:

* Hosting
* SIP / VoIP
* IoT
* SCADA
* Legacy applications
* Controlled NAT mappings

### Important

Do not blindly constrain the source port.

```text
Client source port
        ↓
NAT matching
        ↓
Translated session
```

A source-port restriction can unintentionally prevent legitimate traffic.

---

# 17. Central NAT + Virtual Wire Pair

FortiGate supports SNAT with Virtual Wire Pair interfaces.

Example topology:

```text
Internet
   |
MikroTik-1
192.168.254.10
   |
   | VWP
   |
FortiGate
   |
   | VWP
   |
MikroTik-2
192.168.254.11
   |
   |
10.10.10.0/24
```

Fortinet documents SNAT support for IPv4/IPv6 policies using Virtual Wire Pair interfaces, including Central NAT.

---

# 18. Virtual Wire Pair + IP Pool

Create the VWP:

```bash
config system virtual-wire-pair
    edit "VWP-TEST"
        set member "port4" "port5"
    next
end
```

Create IP Pool:

```bash
config firewall ippool
    edit "VWP-SNAT"
        set startip 192.168.254.10
        set endip 192.168.254.10
    next
end
```

### Central NAT

```bash
config firewall central-snat-map
    edit 1
        set srcintf "port4"
        set dstintf "port5"
        set orig-addr "all"
        set dst-addr "all"
        set nat enable
        set nat-ippool "VWP-SNAT"
    next
end
```

Fortinet notes that the IP Pool used with VWP must be in a different subnet from the VWP peers.

---

# 19. Bidirectional VWP SNAT

If both directions require translation:

```text
port4
  |
  | SNAT Pool 1
  v
port5
```

and:

```text
port5
  |
  | SNAT Pool 2
  v
port4
```

Example:

```bash
config firewall central-snat-map

    edit 1
        set srcintf "port4"
        set dstintf "port5"
        set orig-addr "all"
        set dst-addr "all"
        set nat enable
        set nat-ippool "POOL-1"
    next

    edit 2
        set srcintf "port5"
        set dstintf "port4"
        set orig-addr "all"
        set dst-addr "all"
        set nat enable
        set nat-ippool "POOL-2"
    next

end
```

---

# 20. NAT Troubleshooting

### Session table

```bash
diagnose sys session list
```

Alternative:

```bash
get system session list
```

Look for:

```text
original source
original destination
translated source
translated destination
NAT information
```

---

# 21. NAT Debugging Mental Model

When troubleshooting:

```text
1. Does traffic reach FortiGate?
          |
          v
2. Does firewall policy match?
          |
          v
3. Does SNAT decision match?
          |
          v
4. Is IP Pool available?
          |
          v
5. Is translated port available?
          |
          v
6. Is return traffic routed correctly?
          |
          v
7. Is the session established?
```

---

# 22. NAT Exhaustion

A common problem:

```text
Too many clients
       +
Too few public IPs
       +
Too many simultaneous sessions
       =
NAT Port Exhaustion
```

Symptoms may include:

* New outbound connections fail
* Existing connections continue working
* NAT allocation failures
* Large session counts
* High port utilization

### Capacity expansion

```text
Option 1:
Add public IPs

Option 2:
Increase available port allocation

Option 3:
Use PBA

Option 4:
Tune application/session timeouts

Option 5:
Use a dedicated CGNAT architecture
```

---

# 23. Overload vs Fixed Port Range vs PBA

| Feature                  | Overload | Fixed Port Range | PBA           |
| ------------------------ | -------- | ---------------- | ------------- |
| PAT                      | ✅        | ✅                | ✅             |
| Shared public IP         | ✅        | ✅                | ✅             |
| Deterministic allocation | Limited  | ✅                | ✅             |
| Port blocks              | ❌        | ❌                | ✅             |
| CGNAT suitability        | Medium   | High             | **Very High** |
| Subscriber traceability  | Lower    | High             | **High**      |
| Per-user port quota      | Dynamic  | Fixed mapping    | **Explicit**  |
| Configuration complexity | Low      | Medium           | Medium/High   |

---

# 24. NAT Design for SIP / VoIP

SIP environments can be sensitive to:

```text
Source IP
Source Port
NAT timeout
Session tracking
ALG behavior
```

Potential design:

```text
SIP Client
   |
   | UDP 5060
   v
FortiGate
   |
   | Controlled SNAT
   v
SIP Provider
```

Consider:

* Fixed Port Range
* Port preservation requirements
* SIP ALG / helper behavior
* Session TTL
* RTP port ranges
* Return-path symmetry

> Do not select Fixed Port Range merely because the application uses SIP. Validate the actual NAT behavior and application requirements first.

---

# 25. NAT Port Testing

High-volume connection generation can be used in a lab to observe NAT allocation.

Example concept:

```bash
for i in $(seq 1 1000); do
    curl -s http://20.20.20.1 &
done
wait
```

Then inspect:

```bash
diagnose sys session list
```

> ⚠️ Run traffic-generation tests only in an isolated lab or an environment where you are explicitly authorized to generate the traffic.

---

# 26. NAT Session Lifecycle

NAT resources are tied to sessions.

Conceptually:

```text
New Connection
      |
      v
NAT Allocation
      |
      v
Session Created
      |
      v
Traffic
      |
      v
Session Timeout / FIN / RST
      |
      v
NAT Resource Released
```

Therefore, aggressive session timeouts can help reclaim NAT resources, but excessive timeout reduction can break legitimate long-lived applications.

---

# 27. CGNAT with VXLAN over IPv6 Underlay

Example architecture:

```text
             IPv6 Underlay
                  |
        +---------+---------+
        |                   |
      FGT-1               FGT-2
2001:db8::1             2001:db8::2
        |                   |
        +------ VXLAN -------+
                 VNI 1000
                    |
               VLAN / LAN
                    |
             192.168.101.0/24
                    |
                 SNAT
                    |
             100.64.0.0/10
```

### FGT-1

```bash
config system interface
    edit "port4"
        set ip6-address 2001:db8::1/64
    next
end
```

VXLAN:

```bash
config system vxlan
    edit "vx1000"
        set vni 1000
        set interface "port4"
        set ip-version ipv6-unicast
        set remote-ipv6 2001:db8::2
    next
end
```

Create the LAN/VLAN side:

```text
VLAN 10
   |
Software Switch
   |
VXLAN 1000
   |
192.168.101.1/24
```

Then SNAT:

```text
192.168.101.0/24
        |
        v
FortiGate
        |
        v
IP Pool
192.168.254.100-192.168.254.200
```

For CGNAT-style deployments, an RFC 6598 address pool such as:

```text
100.64.0.0/10
```

may be used where appropriate.

---

# 28. FGT-2 VXLAN Concept

```text
IPv6 Underlay:

FGT-2
2001:db8::2
     |
     | VXLAN VNI 1000
     |
     v
FGT-1
2001:db8::1
```

FGT-2 LAN:

```text
VLAN 10
    |
Software Switch
    |
VXLAN 1000
    |
192.168.101.2/24
```

DHCP can be provided locally or relayed depending on the design.

---

# 29. Important NAT Design Rules

### Rule 1 — Do not confuse IP Pool types

```text
Overload
    ≠
One-to-One
    ≠
Fixed Port Range
    ≠
PBA
```

---

### Rule 2 — Plan NAT capacity before deployment

Calculate:

```text
Public IP count
×
Available NAT ports
```

Then compare against:

```text
Expected concurrent sessions
```

---

### Rule 3 — CGNAT requires deterministic thinking

For large subscriber environments:

```text
Subscriber
    ↓
Public IP
    ↓
Port Block
    ↓
Session
```

PBA can simplify subscriber-to-public-IP/port traceability.

---

### Rule 4 — Don't blindly use `ALL`

Especially with:

* Central SNAT
* Port restrictions
* Hosting
* SIP
* IoT
* SCADA

Match only what the architecture actually requires.

---

### Rule 5 — VWP has special requirements

When using an IP Pool with Virtual Wire Pair:

```text
Verify IP Pool subnet
Verify VWP peer subnet
Verify SNAT direction
Verify return path
```

Fortinet specifically documents that the IP Pool must use a different subnet from the VWP peers.

---

# 30. Quick Troubleshooting Checklist

```text
[ ] Firewall policy matches
[ ] Central NAT enabled/disabled as intended
[ ] Correct SNAT map order
[ ] Correct IP Pool selected
[ ] IP Pool has available addresses
[ ] NAT ports are not exhausted
[ ] Return route exists
[ ] Session exists
[ ] Source port requirement verified
[ ] Application ALG/helper checked
[ ] Session timeout checked
[ ] VWP topology verified
[ ] IP Pool ARP behavior verified
[ ] CGNAT port allocation calculated
```

---

# 31. High-Value Commands

### Session table

```bash
diagnose sys session list
```

```bash
get system session list
```

### IP Pool configuration

```bash
show firewall ippool
```

### Central NAT

```bash
show firewall central-snat-map
```

### System NAT/session settings

```bash
show system settings
```

### VWP

```bash
show system virtual-wire-pair
```

---

# 32. Interview / NSE Takeaways

> **Overload** = normal PAT and shared public IPs.

> **One-to-One** = dedicated address mapping without PAT.

> **Fixed Port Range** = deterministic source-IP/source-port allocation.

> **Port Block Allocation** = deterministic blocks of ports assigned to users; highly relevant to CGNAT.

> **Central NAT** = separates source NAT logic from the normal firewall policy.

> **Virtual Wire Pair + SNAT** = supported, but the IP Pool/subnet design has specific requirements.

> **60416** = useful FortiGate NAT planning figure per IPv4 pool address in the documented examples.

> **PBA formula:**

```text
Ports/User =
Block Size × Blocks/User
```

> **Approximate users/public IP:**

```text
60416 / Ports-per-user
```

---

## References

* Fortinet Dynamic SNAT / IP Pool documentation.
* Fortinet IP Pool CLI reference.
* Fortinet SNAT with Virtual Wire Pair documentation.
* Fortinet PBA / Fixed Port Range examples.

---

## SheynShield Rule of Thumb

```text
Normal Enterprise Internet NAT
        ↓
      OVERLOAD

Dedicated Public Mapping
        ↓
    ONE-TO-ONE

Deterministic IP/Port Mapping
        ↓
 FIXED PORT RANGE

Large Subscriber / CGNAT
        ↓
      PBA
```

**Think in three dimensions:**

```text
NAT Design
│
├── Address
│     └── Which public IP?
│
├── Port
│     └── Which source-port range/block?
│
└── Time
      └── How long does the translation exist?
```

That combination is the foundation of scalable FortiGate SNAT/CGNAT design.

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

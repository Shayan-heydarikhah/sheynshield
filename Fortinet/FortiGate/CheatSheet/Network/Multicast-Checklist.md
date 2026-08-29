# 🔥 IPv4 Multicast Security & Networking Checklist  

> **SheynShield | Network Security & Design Knowledge Base**  
> IPv4 Multicast • IGMP • PIM • RP • SSM • MSDP • IGMP Snooping • FortiGate

![GitHub](https://img.shields.io/badge/Category-Network%20Security-blue)
![Topic](https://img.shields.io/badge/Topic-IPv4%20Multicast-orange)
![Platform](https://img.shields.io/badge/Platform-FortiGate%20%7C%20Cisco-red)
![Level](https://img.shields.io/badge/Level-Professional-green)

---

# 📌 IPv4 Multicast Deployment Checklist

## 1. Multicast Design Planning Checklist

- [ ] Define multicast application requirements

  - [ ] IPTV
  - [ ] Video streaming
  - [ ] Financial market data
  - [ ] Video conference
  - [ ] Internal service distribution

- [ ] Identify multicast model

| Requirement | Recommended Model |
|---|---|
| Single source → many receivers | SSM |
| Multiple sources → many receivers | Bidir-PIM |
| Legacy ASM applications | PIM-SM |

---

## 2. Multicast Address Planning

### IPv4 Multicast Range Validation

- [ ] Confirm multicast addresses are inside:

```text
224.0.0.0 - 239.255.255.255
````

---

## Multicast Address Allocation

* [ ] Avoid using:

```text
224.0.0.0/24
```

for applications.

Reason:

```text
Reserved for local control protocols
```

---

## Recommended Enterprise Range

* [ ] Use:

```text
239.0.0.0/8
```

for internal multicast applications.

Example:

```text
239.10.10.10
239.192.20.1
```

---

## SSM Planning

* [ ] Use SSM range when possible:

```text
232.0.0.0/8
```

Architecture:

```text
(Source,Group)

(S,G)
```

---

# 3. Layer 2 Multicast Checklist

## Multicast MAC Validation

* [ ] Verify IPv4 multicast MAC mapping:

```text
01:00:5E:xx:xx:xx
```

* [ ] Understand MAC aliasing:

```text
28-bit multicast IP
        |
        ↓
23-bit MAC mapping
```

Result:

```text
32 multicast IPs
        |
        ↓
same multicast MAC
```

---

# 4. IGMP Configuration Checklist

## Host Membership

Verify:

* [ ] Receiver sends IGMP Join
* [ ] Router receives membership report
* [ ] Correct multicast group is requested

Verification:

Cisco:

```cisco
show ip igmp groups
show ip igmp interface
```

FortiGate:

```cli
get router info multicast igmp
get router info multicast igmp group
```

---

# IGMP Version Selection

## IGMPv1

Use only for legacy environments.

Checklist:

* [ ] No source filtering required
* [ ] No explicit leave required

---

## IGMPv2

Checklist:

* [ ] Need Leave messages
* [ ] Need querier election

Verify:

```text
Query
Report
Leave
```

---

## IGMPv3

Recommended for SSM.

Checklist:

* [ ] Source filtering required
* [ ] SSM deployment
* [ ] INCLUDE/EXCLUDE filtering

Verify:

```text
(S,G)
```

---

# 5. IGMP Querier Checklist

Verify:

* [ ] IGMP querier exists
* [ ] Correct VLAN has multicast router
* [ ] Election result is expected

Remember:

```text
Lowest IP wins
```

Example:

```text
R1 10.1.1.1
R2 10.1.1.2

Winner:
R1
```

---

# 6. IGMP Snooping Checklist

## Switch Configuration

Enable:

* [ ] IGMP Snooping

Reason:

Without snooping:

```text
Multicast
    |
 Switch
    |
 Flood everywhere
```

With snooping:

```text
Multicast
    |
 Switch
    |
 Only interested ports
```

---

Verify:

Cisco:

```cisco
show ip igmp snooping
show ip igmp snooping groups
show ip igmp snooping querier
```

---

Checklist:

* [ ] Multicast router port detected
* [ ] Receiver ports learned
* [ ] No multicast flooding
* [ ] Immediate leave configured correctly

---

# 7. PIM Deployment Checklist

## Select PIM Mode

### PIM Dense Mode

Use only when:

* [ ] Small environment
* [ ] Legacy deployment

Model:

```text
Flood → Prune
```

---

### PIM Sparse Mode

Recommended for ASM.

Checklist:

* [ ] PIM enabled on all routed interfaces
* [ ] RP configured
* [ ] Neighbors established

Verification:

```cisco
show ip pim neighbor
show ip pim interface
```

---

### PIM SSM

Preferred modern design.

Checklist:

* [ ] IGMPv3 enabled
* [ ] SSM range configured
* [ ] RP not required

Flow:

```text
Receiver
   |
IGMPv3 Join
   |
(S,G)
```

---

# 8. PIM Neighbor Checklist

Verify:

* [ ] PIM enabled on interfaces
* [ ] Same subnet
* [ ] No ACL blocking PIM
* [ ] Hello messages received

Commands:

Cisco:

```cisco
show ip pim neighbor
```

FortiGate:

```cli
get router info multicast pim sparse-mode neighbor
```

---

# 9. RP Design Checklist

## Static RP

Checklist:

* [ ] RP reachable
* [ ] RP address configured everywhere

Cisco:

```cisco
ip pim rp-address X.X.X.X
```

FortiGate:

```cli
config router multicast
config rp-address
```

---

## Auto-RP

Verify:

```text
224.0.1.39
224.0.1.40
```

Checklist:

* [ ] Candidate RP exists
* [ ] Mapping agent exists
* [ ] RP discovery works

---

## BSR

Checklist:

* [ ] Candidate BSR configured
* [ ] Candidate RP configured
* [ ] RP-set exchanged

Verification:

```cisco
show ip pim bsr-router
show ip pim rp mapping
```

---

# 10. Anycast RP Checklist

Verify:

* [ ] Multiple RP devices exist
* [ ] Same RP address configured
* [ ] MSDP configured between RPs

Architecture:

```text
        Anycast RP
        10.10.10.10

          /    \

        RP1    RP2
```

---

# 11. MSDP Checklist

Required for:

* [ ] Anycast RP
* [ ] Inter-domain multicast

Verify:

```cisco
show ip msdp peer
show ip msdp sa-cache
```

Checklist:

* [ ] TCP 639 allowed
* [ ] MSDP peers established
* [ ] SA messages exchanged

---

# 12. RPF Validation Checklist

RPF is mandatory.

Check:

```text
Source reachability
        |
        ↓
Unicast Routing Table
        |
        ↓
RPF Interface
```

---

Verify:

Cisco:

```cisco
show ip rpf <source>
```

Checklist:

* [ ] Source route exists
* [ ] Correct incoming interface
* [ ] No asymmetric routing
* [ ] IGP/BGP healthy

---

# 13. Multicast Routing Table Checklist

Verify:

```cisco
show ip mroute
```

Check:

## Shared Tree

```text
(*,G)
```

Means:

```text
Any Source → Group
```

---

## Source Tree

```text
(S,G)
```

Means:

```text
Specific Source → Group
```

---

Validate:

* [ ] Incoming interface
* [ ] Outgoing interface list
* [ ] RPF neighbor
* [ ] State timers

---

# 14. FortiGate Multicast Checklist

## Enable Multicast Routing

Verify:

```cli
config router multicast
    set multicast-routing enable
end
```

---

## PIM Configuration

Checklist:

* [ ] PIM mode configured
* [ ] Hello messages working
* [ ] Neighbor established

Verify:

```cli
get router info multicast pim sparse-mode neighbor
```

---

## IGMP Configuration

Verify:

```cli
get router info multicast igmp group
```

Checklist:

* [ ] Correct IGMP version
* [ ] Correct query interval
* [ ] Receiver groups learned

---

# 15. FortiGate Multicast Policy Checklist

Verify:

* [ ] Multicast policy exists
* [ ] Source allowed
* [ ] Destination multicast group allowed
* [ ] Interface path allowed

Remember:

```text
Unicast firewall policy
≠
Multicast policy
```

---

# 16. FortiGate MFC Checklist

Multicast Forwarding Cache validation:

```cli
diagnose ip multicast mroute
```

Check:

* [ ] Source exists
* [ ] Group exists
* [ ] Incoming VIF correct
* [ ] Outgoing VIF correct

Example:

```text
Source:
192.168.20.200

Group:
239.192.20.1

Incoming:
VIF 3

Outgoing:
VIF 2
```

---

# 17. Multicast Troubleshooting Checklist

## Step 1 — Application

* [ ] Source transmitting?
* [ ] UDP multicast stream exists?

---

## Step 2 — Receiver

Check:

* [ ] Client joined multicast group
* [ ] IGMP report generated

---

## Step 3 — Layer 2

Check:

* [ ] IGMP Snooping
* [ ] VLAN configuration
* [ ] Multicast router port

---

## Step 4 — PIM

Check:

* [ ] Neighbor adjacency
* [ ] Interface mode
* [ ] Hello messages

---

## Step 5 — RP

Check:

* [ ] RP reachable
* [ ] RP mapping correct

---

## Step 6 — RPF

Check:

* [ ] Source route
* [ ] Correct next hop

---

## Step 7 — Forwarding

Check:

* [ ] Mroute exists
* [ ] MFC exists
* [ ] Firewall policy allows traffic

---

# 18. Common Multicast Failure Matrix

| Problem                   | Possible Cause             |
| ------------------------- | -------------------------- |
| No multicast traffic      | IGMP Join missing          |
| Flooding                  | IGMP Snooping disabled     |
| ASM failure               | RP issue                   |
| SSM failure               | IGMPv3 issue               |
| RPF failure               | Wrong unicast route        |
| Duplicate traffic         | PIM Assert issue           |
| Traffic stops after leave | Immediate Leave issue      |
| FortiGate drops traffic   | Multicast policy/MFC issue |

---

# 19. Production Design Checklist

## Recommended Architecture

* [ ] Prefer SSM
* [ ] Use IGMPv3
* [ ] Enable IGMP Snooping
* [ ] Use 239.0.0.0/8 internally
* [ ] Design RP redundancy
* [ ] Monitor multicast state
* [ ] Document multicast groups

---

# 20. Final Multicast Validation Checklist

Before production:

* [ ] Multicast address documented
* [ ] Source documented
* [ ] Receiver groups documented
* [ ] IGMP version selected
* [ ] PIM design approved
* [ ] RP design approved
* [ ] RPF verified
* [ ] IGMP Snooping verified
* [ ] Firewall policy verified
* [ ] FortiGate MFC verified
* [ ] Monitoring configured

---

# 🔥 SheynShield Multicast Mental Model

```text
Receiver
   |
   ↓
IGMP
   |
   ↓
PIM
   |
   ↓
RP / SSM
   |
   ↓
RPF
   |
   ↓
Multicast Routing Table
   |
   ↓
MFC
   |
   ↓
IGMP Snooping
   |
   ↓
Receiver
```

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


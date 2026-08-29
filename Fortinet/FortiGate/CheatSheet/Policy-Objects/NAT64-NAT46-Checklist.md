# 🔗 SheynShield Resources

# FortiGate NAT64 & NAT46 Deployment & Troubleshooting Checklist

> **FortiOS:** 7.2.x  
> **Level:** Advanced / NSE4–NSE7  
> **Topic:** IPv4/IPv6 Translation, NAT64, NAT46, DNS64, VIP, IP Pool, Session Troubleshooting  
> **Brand:** SheynShield | Engineering Secure Networks

---

# 🎯 Objective

Use this checklist to design, configure, validate, and troubleshoot:

- IPv6 → IPv4 translation (**NAT64**)
- IPv4 → IPv6 translation (**NAT46**)
- DNS64 integration
- IPv6/IPv4 VIP translation
- NAT IP Pool behavior
- Return path validation
- FortiGate session troubleshooting

---

# 🧠 Core Memory Map

```text
NAT64
IPv6 Client
      |
      |
      v
FortiGate
      |
      |
      v
IPv4 Server
````

```text
NAT46
IPv4 Client
      |
      |
      v
FortiGate
      |
      |
      v
IPv6 Server
```

---

# 1. Pre-Deployment Checklist

## Feature Validation

* [ ] Confirm FortiOS version compatibility.
* [ ] Confirm IPv6 feature visibility is enabled.
* [ ] Confirm required NAT64/NAT46 features are available.
* [ ] Confirm DNS Database feature visibility if DNS64 is required.
* [ ] Confirm IPv6 routing design.
* [ ] Confirm IPv4 routing design.
* [ ] Document translation direction.

---

# 2. NAT64 vs NAT46 Quick Reference

| Feature | Direction   | Client    | Server    |
| ------- | ----------- | --------- | --------- |
| NAT64   | IPv6 → IPv4 | IPv6 Only | IPv4 Only |
| NAT46   | IPv4 → IPv6 | IPv4 Only | IPv6 Only |

Memory:

```text
64 = IPv6 to IPv4

46 = IPv4 to IPv6
```

---

# 3. NAT64 Design Checklist

## IPv6 Client Requirements

* [ ] IPv6 address configured.
* [ ] IPv6 default gateway reachable.
* [ ] IPv6 DNS configured.
* [ ] IPv6 route validated.
* [ ] Client can reach FortiGate IPv6 interface.

Example:

```bash
ip -6 addr

ip -6 route
```

---

# 4. NAT64 VIP Checklist

Create IPv6 VIP:

Example:

```text
External IPv6:

2001:db8:abcd::1001


Mapped IPv4:

192.168.20.200
```

Validation:

* [ ] VIP created.
* [ ] Correct interface selected.
* [ ] External IPv6 address correct.
* [ ] Mapped IPv4 server reachable.
* [ ] Service port configured correctly.
* [ ] VIP matches incoming IPv6 destination.

Logical flow:

```text
IPv6 Destination
        |
        v
IPv6 VIP
        |
        |
        v
IPv4 Server
```

---

# 5. NAT64 IP Pool Checklist

Validate:

* [ ] IPv4 NAT pool created.
* [ ] NAT64 enabled.
* [ ] Pool address reachable from IPv4 server side.
* [ ] Return routing exists.

Example:

```text
IPv6 Client

2001:db8:abcd::10

        |
        |
        v

NAT64 Pool

192.168.20.201

        |
        |
        v

IPv4 Server

192.168.20.200
```

---

# 6. NAT64 Firewall Policy Checklist

Verify:

* [ ] Source interface is IPv6 client side.
* [ ] Destination is IPv6 VIP.
* [ ] Source object is correct IPv6 object.
* [ ] Service is correct.
* [ ] Action is ACCEPT.
* [ ] Logging enabled.
* [ ] NAT64 pool selected.
* [ ] Policy order is correct.

Logical policy:

```text
IPv6 Client
      |
      v
IPv6 VIP
      |
      v
NAT64 Translation
      |
      v
IPv4 Server
```

---

# 7. DNS64 Checklist

DNS64 is required when IPv6-only clients need IPv4-only services.

Validate:

* [ ] FortiGate DNS64 enabled.
* [ ] Client uses FortiGate DNS.
* [ ] AAAA synthesis works.
* [ ] Synthetic IPv6 address is generated.
* [ ] NAT64 receives translated traffic.

Example:

Enable:

```bash
config system dns64
    set status enable
    set always-synthesize-aaaa-record enable
end
```

---

# 8. DNS64 vs NAT64

Important:

```text
DNS64 ≠ NAT64
```

DNS64:

```text
Creates IPv6 representation
```

NAT64:

```text
Translates packets/session
```

Flow:

```text
Client
 |
 | AAAA Query
 v
DNS64
 |
 | Synthetic AAAA
 v
IPv6 Destination
 |
 v
NAT64
 |
 v
IPv4 Server
```

---

# 9. DNS Database Checklist

For lab/testing:

* [ ] DNS Database enabled.
* [ ] Zone created.
* [ ] AAAA record created.
* [ ] Client points to FortiGate DNS.

Example:

```text
test.co

AAAA:

shayan.test.co

2001:db8:abcd::1001
```

---

# 10. NAT46 Design Checklist

## IPv4 Client Requirements

Verify:

* [ ] IPv4 client reachable.
* [ ] IPv4 routing works.
* [ ] IPv4 policy exists.
* [ ] IPv4 VIP configured.

Topology:

```text
IPv4 Client

192.168.20.10

        |
        v

FortiGate

        |
        v

IPv6 Server

2001:db8:abcd::2
```

---

# 11. NAT46 VIP Checklist

Create IPv4 VIP:

Example:

```text
External IPv4:

192.168.20.202


Mapped IPv6:

2001:db8:abcd::2
```

Verify:

* [ ] IPv4 VIP exists.
* [ ] Destination translation works.
* [ ] IPv6 server is reachable.
* [ ] Correct interface selected.

---

# 12. NAT46 IP Pool Checklist

Verify:

* [ ] IPv6 NAT pool created.
* [ ] NAT46 enabled.
* [ ] IPv6 source address exists.
* [ ] IPv6 return route exists.

Concept:

```text
IPv4 Source

192.168.20.10

       |
       |
       v

IPv6 Source

2001:db8:abcd::3
```

---

# 13. NAT46 Policy Checklist

Verify:

* [ ] IPv4 source interface correct.
* [ ] Destination VIP correct.
* [ ] Service allowed.
* [ ] NAT46 enabled.
* [ ] IPv6 pool selected.
* [ ] Logging enabled.

---

# 14. Routing Checklist

## IPv4 Routing

```bash
get router info routing-table all
```

Verify:

* [ ] IPv4 destination reachable.
* [ ] NAT64 translated source reachable.
* [ ] NAT46 translated source reachable.

---

## IPv6 Routing

```bash
get router info6 routing-table
```

Verify:

* [ ] IPv6 destination reachable.
* [ ] IPv6 return path exists.
* [ ] Neighbor discovery works.

---

# 15. NAT64 Route Checklist

Verify NAT64 route behavior:

```bash
config firewall ippool6
    edit 1
        set add-nat64-route enable
    next
end
```

Checklist:

* [ ] NAT64 route exists.
* [ ] IPv6 traffic follows correct path.
* [ ] Translation route is active.

---

# 16. Session Validation Checklist

IPv6 sessions:

```bash
diagnose sys session6 list
```

IPv4 sessions:

```bash
diagnose sys session list
```

Check:

* [ ] Session created.
* [ ] Policy ID matched.
* [ ] Source translation correct.
* [ ] Destination translation correct.
* [ ] Session state established.

---

# 17. Flow Debug Checklist

Before debugging:

* [ ] Narrow traffic scope.
* [ ] Avoid production-wide debug.
* [ ] Capture only required source/destination.

Example IPv6:

```bash
diagnose debug flow filter6 addr 2001:db8:abcd::10

diagnose debug enable

diagnose debug flow trace start 50
```

Stop:

```bash
diagnose debug disable

diagnose debug reset
```

---

# 18. NAT64 Packet Walk Checklist

Validate each step:

```text
IPv6 Client
      |
      v
DNS64
      |
      v
IPv6 Destination
      |
      v
VIP Matching
      |
      v
Firewall Policy
      |
      v
NAT64 Pool
      |
      v
IPv4 Server
```

---

# 19. NAT46 Packet Walk Checklist

Validate:

```text
IPv4 Client
      |
      v
IPv4 VIP
      |
      v
Firewall Policy
      |
      v
NAT46 Pool
      |
      v
IPv6 Server
```

---

# 20. Return Path Checklist

Critical rule:

```text
Request Path
      +
Return Path
      =
Successful Translation
```

Verify:

* [ ] IPv4 return route.
* [ ] IPv6 return route.
* [ ] NAT pool reachability.
* [ ] Server gateway configuration.
* [ ] Firewall policy.
* [ ] Session state.

Common symptom:

```text
Request works

Reply fails
```

---

# 21. Common Failure Checklist

## DNS Failure

Check:

* [ ] DNS64 enabled.
* [ ] AAAA synthesis.
* [ ] Client DNS configuration.

---

## VIP Failure

Check:

* [ ] Correct IP version.
* [ ] Correct interface.
* [ ] Correct mapped address.
* [ ] Correct service.

---

## Policy Failure

Check:

* [ ] Source.
* [ ] Destination.
* [ ] Service.
* [ ] NAT mode.
* [ ] Policy order.

---

## Application Failure

Check:

* [ ] Protocol compatibility.
* [ ] MTU.
* [ ] Fragmentation.
* [ ] ICMP translation.
* [ ] Return traffic.

---

# 22. Security Checklist

Do not use NAT as a security mechanism.

Verify:

* [ ] Least privilege policy.
* [ ] Specific VIP objects.
* [ ] Specific services.
* [ ] Logging enabled.
* [ ] Monitoring enabled.
* [ ] No unnecessary ALL rules.

Recommended:

```text
Specific Source

+

Specific VIP

+

Specific Service

+

Logging
```

---

# 23. NSE Exam Memory Map

```text
IPv4 / IPv6 Transition

        |
        |
        +── NAT64
        |      |
        |      + IPv6 → IPv4
        |      + DNS64
        |      + IPv6 VIP
        |
        |
        +── NAT46
               |
               + IPv4 → IPv6
               + IPv4 VIP
               + IPv6 Pool
```

---

# 24. Quick Command Reference

| Purpose       | Command                             |
| ------------- | ----------------------------------- |
| IPv6 Sessions | `diagnose sys session6 list`        |
| IPv4 Sessions | `diagnose sys session list`         |
| IPv4 Routing  | `get router info routing-table all` |
| IPv6 Routing  | `get router info6 routing-table`    |
| IPv6 Debug    | `diagnose debug flow filter6`       |
| DNS64 Config  | `config system dns64`               |

---

# 25. One-Minute Troubleshooting Formula

```text
DNS
 ↓
IPv6/IPv4 Destination
 ↓
VIP
 ↓
Policy
 ↓
NAT64/NAT46
 ↓
IP Pool
 ↓
Route
 ↓
Session
 ↓
Return Path
```

---

# 🧠 Engineer's Takeaway

> NAT64/NAT46 troubleshooting is not about checking only the NAT rule. The engineer must follow the complete packet lifecycle: DNS discovery, VIP matching, policy decision, translation, routing, session creation, and return traffic.

---

# 🔥 Interview Questions

## Q1: What is NAT64?

Answer:

```text
IPv6 to IPv4 translation mechanism.
```

---

## Q2: What is NAT46?

Answer:

```text
IPv4 to IPv6 translation mechanism.
```

---

## Q3: What is DNS64?

Answer:

```text
A DNS mechanism that synthesizes IPv6 records for IPv4-only destinations.
```

---

## Q4: Why can DNS work while NAT64 fails?

Because:

```text
DNS64 = name resolution

NAT64 = packet translation
```

---

## Q5: First troubleshooting step?

Follow:

```text
Client
 ↓
DNS
 ↓
VIP
 ↓
Policy
 ↓
NAT
 ↓
Route
 ↓
Session
```

---## 🔗 SheynShield Resources

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

#FortiGate #FortiOS #NAT64 #NAT46 #DNS64 #IPv6 #IPv4 #NetworkSecurity #CyberSecurity #NSE7 #Fortinet


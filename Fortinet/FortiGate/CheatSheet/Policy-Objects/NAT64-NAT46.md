# FortiGate NAT64 & NAT46  

> **FortiOS Focus:** 7.2.x
> **Topic:** IPv4/IPv6 Translation, NAT64, NAT46, DNS64, VIP, IP Pools & Troubleshooting
> **Level:** Advanced / NSE4–NSE7

---

## 1. NAT64 vs NAT46 — Core Concept

| Feature   | Traffic Direction | Source | Destination | Translation |
| --------- | ----------------- | ------ | ----------- | ----------- |
| **NAT64** | IPv6 → IPv4       | IPv6   | IPv4        | IPv6 → IPv4 |
| **NAT46** | IPv4 → IPv6       | IPv4   | IPv6        | IPv4 → IPv6 |

### Mental Model

```text
NAT64
IPv6 Client
    |
    | IPv6
    v
FortiGate
    |
    | IPv4
    v
IPv4 Server
```

```text
NAT46
IPv4 Client
    |
    | IPv4
    v
FortiGate
    |
    | IPv6
    v
IPv6 Server
```

> **Important:** NAT64/NAT46 is not simply "changing the IP version."
> FortiGate must translate the packet/session between IPv6 and IPv4 domains and maintain the corresponding session state.

---

# 2. Feature Visibility

Before configuring IPv6 translation, make sure the required features are visible.

```text
System
 └── Feature Visibility
      ├── IPv6
      └── DNS Database
```

CLI:

```bash
config system settings
    # Enable IPv6-related features as required
end
```

---

# 3. NAT64 — IPv6 Client → IPv4 Server

## Topology

```text
IPv6 Client
2001:db8:abcd::10
        |
        | IPv6
        |
     port3
   FortiGate
     port2
        |
        | IPv4
        |
192.168.20.200
IPv4 Server
```

---

## 4. Create IPv6 VIP

The IPv6 client needs an IPv6 destination that FortiGate can translate toward the IPv4 server.

Example:

```text
VIP:
    Name: vip-test-v6
    Interface: port3
    External IP:
        2001:db8:abcd::1001

    Mapped IP:
        192.168.20.200
```

Conceptually:

```text
2001:db8:abcd::1001
          |
          | NAT64
          v
192.168.20.200
```

---

# 5. NAT64 IP Pool

Create an IPv4 IP pool and enable NAT64.

Concept:

```text
IPv6 source
    |
    v
FortiGate NAT64
    |
    v
IPv4 source address from NAT64 IP Pool
```

Example:

```text
IP Pool:
    ippool-v64

Type:
    IPv4

NAT64:
    enabled

External IP:
    192.168.20.201
```

### Why is an IPv4 pool required?

The translated IPv4 session needs an IPv4 source address.

```text
IPv6 Client
2001:db8:abcd::10

        |
        | NAT64
        v

IPv4 Source
192.168.20.201

        |
        v

IPv4 Server
192.168.20.200
```

---

# 6. NAT64 Firewall Policy

Example logical policy:

```text
Source Interface:
    LAN

Destination Interface:
    DMZ

Source:
    all6

Destination:
    vip-test-v6

Service:
    ALL

Schedule:
    always

Action:
    ACCEPT

NAT:
    NAT64
    IP Pool: ippool-v64

Logging:
    ALL
```

### Important

NAT64 policy matching must use the appropriate IPv6 objects/interfaces on the IPv6 side.

Do **not** treat NAT64 as an ordinary IPv4 policy with an IPv6 address pasted into it.

---

# 7. NAT64 + DNS64

DNS becomes especially important when IPv6-only clients need to access IPv4-only servers.

```text
IPv6 Client
     |
     | DNS AAAA Query
     v
FortiGate DNS64
     |
     | Synthesized AAAA
     v
IPv6 representation of IPv4 destination
     |
     v
NAT64
     |
     v
IPv4 Server
```

---

# 8. FortiGate DNS64

Enable DNS64:

```bash
config system dns64
    set status enable
    set always-synthesize-aaaa-record enable
end
```

### What does DNS64 do?

An IPv4-only destination normally has:

```text
A record
192.168.20.200
```

An IPv6-only client asks:

```text
AAAA?
```

DNS64 can synthesize an IPv6 representation of the IPv4 address.

Conceptually:

```text
IPv4:
192.168.20.200

        |
        | DNS64 synthesis
        v

Synthetic IPv6:
<IPv6-prefix>:192.168.20.200
```

The client then connects using IPv6.

FortiGate recognizes the translated destination and performs NAT64 toward the real IPv4 server.

---

# 9. DNS Database / Shadow DNS

For a controlled lab, you can create a DNS database/zone and define the required records.

Example concept:

```text
Zone:
    test.co

AAAA:
    shayan.test.co
    2001:db8:abcd::1001
```

Client DNS:

```text
DNS Server:
    FortiGate interface IP
```

Example:

```text
IPv6 Client
    |
    | DNS Query
    v
FortiGate DNS
    |
    +--> AAAA
         shayan.test.co
         2001:db8:abcd::1001
```

---

# 10. NAT46 — IPv4 Client → IPv6 Server

NAT46 is the opposite direction.

```text
IPv4 Client
192.168.20.10
       |
       | IPv4
       v
   FortiGate
       |
       | IPv6
       v
IPv6 Server
2001:db8:abcd::2
```

---

# 11. IPv4 VIP for NAT46

Example:

```text
VIP:
    vip-test-v4

External Interface:
    port2

External IP:
    192.168.20.202

Mapped IP:
    2001:db8:abcd::2
```

Concept:

```text
192.168.20.202
       |
       | NAT46
       v
2001:db8:abcd::2
```

The IPv4 client connects to the IPv4 VIP:

```text
192.168.20.202
```

FortiGate translates the destination into the IPv6 server:

```text
2001:db8:abcd::2
```

---

# 12. NAT46 Source IP Pool

For NAT46, the translated source must exist in the IPv6 address space.

Concept:

```text
IPv4 Source
192.168.20.10

        |
        | NAT46
        v

IPv6 Source
2001:db8:abcd::3

        |
        v

IPv6 Server
2001:db8:abcd::2
```

Example:

```text
IP Pool:
    ippool-v46

Type:
    IPv6

NAT46:
    enabled

IPv6 address:
    2001:db8:abcd::3
```

---

# 13. NAT46 Policy

Logical structure:

```text
Source Interface:
    DMZ

Destination Interface:
    LAN

Source:
    all

Destination:
    vip-test-v4

Service:
    ALL

Action:
    ACCEPT

NAT:
    NAT46
    IP Pool: ippool-v46

Logging:
    ALL
```

---

# 14. NAT64 / NAT46 Direction  

```text
NAT64
─────────────────────────────
IPv6 source
     ↓
IPv6 policy/session
     ↓
FortiGate
     ↓
IPv4 source
     ↓
IPv4 server
```

```text
NAT46
─────────────────────────────
IPv4 source
     ↓
IPv4 policy/session
     ↓
FortiGate
     ↓
IPv6 source
     ↓
IPv6 server
```

### Remember

```text
64 = IPv6 → IPv4
46 = IPv4 → IPv6
```

---

# 15. DNS64 vs NAT64

These two features solve different problems.

| Feature     | Purpose                                                          |
| ----------- | ---------------------------------------------------------------- |
| **DNS64**   | Produces/synthesizes IPv6 DNS information for IPv4 destinations  |
| **NAT64**   | Actually translates IPv6 packets/sessions into IPv4              |
| **NAT46**   | Translates IPv4 packets/sessions into IPv6                       |
| **VIP**     | Provides destination translation / published translated endpoint |
| **IP Pool** | Provides translated source addresses                             |

### Critical Concept

```text
DNS64 ≠ NAT64
```

DNS64 helps the client **discover/reach the destination representation**.

NAT64 performs the **actual packet translation**.

---

# 16. DNS64 Always-Synthesize Behavior

```bash
config system dns64
    set status enable
    set always-synthesize-aaaa-record enable
end
```

### `always-synthesize-aaaa-record`

This controls whether FortiGate synthesizes AAAA responses even when the normal DNS response does not contain an AAAA record.

Use carefully in production because DNS behavior can affect applications significantly.

---

# 17. NAT64 Route Injection

FortiOS provides an option to add NAT64-related routes.

Example:

```bash
config firewall ippool6
    edit 1
        set add-nat64-route enable
    next
end
```

Concept:

```text
NAT64 Address Pool
        |
        v
FortiGate Routing Table
        |
        v
NAT64 Translation Path
```

### Why?

The option can help FortiGate install the required NAT64-related route information associated with the IPv6 pool/translation mechanism.

---

# 18. Session Troubleshooting

For IPv6 sessions:

```bash
diagnose sys session6 list
```

Use this when troubleshooting:

* IPv6 session creation
* NAT64/NAT46 translation
* Source/destination translation
* Session state
* Policy matching
* IPv6 traffic behavior

For normal session inspection:

```bash
diagnose sys session list
```

---

# 19. NAT64 Troubleshooting Flow

```text
[1] IPv6 Client
       |
       v
[2] DNS Query
       |
       +--> Is AAAA returned?
       |
       v
[3] IPv6 Destination
       |
       v
[4] VIP Matching
       |
       v
[5] NAT64 Policy
       |
       v
[6] NAT64 IP Pool
       |
       v
[7] IPv4 Server
```

Check each layer independently.

---

# 20. Troubleshooting Checklist

### Layer 1 — IPv6

```bash
get system interface
```

Verify:

* IPv6 address
* Interface status
* Routing
* Neighbor discovery
* Client connectivity

---

### Layer 2 — DNS

Check:

```text
AAAA query
```

Confirm:

```text
Client → FortiGate DNS
```

and verify whether DNS64 synthesis occurs.

---

### Layer 3 — VIP

Verify:

```text
External IPv6 / IPv4 address
        ↓
Mapped IPv4 / IPv6 address
```

Make sure the VIP is associated with the correct interface.

---

### Layer 4 — Firewall Policy

Check:

```text
Source Interface
Destination Interface
Source Address
Destination VIP
Service
NAT Mode
IP Pool
```

---

### Layer 5 — Session

```bash
diagnose sys session6 list
```

Look for:

```text
IPv6 source
IPv6 destination
translated addresses
policy ID
session state
```

---

### Layer 6 — Flow Debug

For difficult cases, use flow debugging with the appropriate IPv4/IPv6 filters.

Example IPv6 troubleshooting:

```bash
diagnose debug flow filter6 addr 2001:db8:abcd::10
diagnose debug enable
diagnose debug flow trace start 50
```

Stop afterward:

```bash
diagnose debug disable
diagnose debug reset
```

> Always narrow the filter before enabling flow debug in production.

---

# 21. Common NAT64 Failure Points

| Symptom                                       | Likely Cause                              |
| --------------------------------------------- | ----------------------------------------- |
| IPv6 client cannot resolve destination        | DNS/DNS64 problem                         |
| AAAA exists but connection fails              | NAT64/policy/routing issue                |
| VIP not matched                               | Wrong interface/address/VIP configuration |
| Policy denied                                 | Wrong source/destination/service          |
| IPv4 server sees unexpected source            | NAT64 IP Pool behavior                    |
| Session created but no reply                  | Return routing problem                    |
| IPv6 connectivity works but application fails | Application/protocol compatibility        |
| NAT64 route missing                           | NAT64 route/pool configuration            |
| ICMP behaves differently                      | IPv4/IPv6 ICMP translation considerations |
| DNS works but TCP fails                       | Translation/policy/session path           |

---

# 22. The Most Important Return-Path Rule

NAT64/NAT46 requires a valid return path.

```text
IPv6 Client
     |
     | IPv6
     v
 FortiGate
     |
     | IPv4
     v
IPv4 Server
     |
     | Reply
     v
 FortiGate
     |
     | IPv6
     v
IPv6 Client
```

If the translated source address is not reachable from the server side:

```text
Request:  works
Reply:    fails
Session:  incomplete
```

### Always verify

```text
IPv4 return route
IPv6 return route
NAT pool reachability
Firewall policy
Session state
```

---

# 23. NAT64 Lab Validation

### IPv6 Client

```bash
ip -6 addr
ip -6 route
```

Test DNS:

```bash
dig AAAA shayan.test.co
```

Test connectivity:

```bash
ping6 2001:db8:abcd::1001
```

Test application:

```bash
curl -6 http://[2001:db8:abcd::1001]
```

---

# 24. Verify IPv4 Server

On the IPv4 server:

```bash
ip addr
ip route
```

Capture traffic:

```bash
tcpdump -ni any host 192.168.20.200
```

You should verify that the server sees the expected translated IPv4 session.

---

# 25. NAT64 Packet-Walk

```text
IPv6 Client
2001:db8:abcd::10
        |
        | IPv6 TCP/80
        v
FortiGate
        |
        | Destination translation
        | Source translation
        v
IPv4 Network
        |
        v
192.168.20.200:80
```

Return:

```text
192.168.20.200:80
        |
        v
FortiGate
        |
        | Reverse translation
        v
2001:db8:abcd::10
```

---

# 26. NAT46 Packet-Walk

```text
IPv4 Client
192.168.20.10
        |
        | IPv4 TCP/443
        v
FortiGate
        |
        | NAT46
        v
IPv6 Network
        |
        v
2001:db8:abcd::2:443
```

Return:

```text
2001:db8:abcd::2
        |
        v
FortiGate
        |
        | Reverse translation
        v
192.168.20.10
```

---

# 27. Advanced Design Notes

### NAT64 is usually associated with:

* IPv6-only client networks
* IPv4-only Internet/services
* IPv6 migration
* IPv6-only data centers
* Dual-stack transition architectures

### NAT46 is useful when:

* IPv4-only clients need IPv6 services
* Legacy IPv4 networks must reach IPv6-only resources
* Migration architectures require protocol translation

---

# 28. Security Considerations

Do **not** treat NAT as a security control.

Always explicitly control:

```text
Source
Destination
Service
Policy
Logging
NAT pool
DNS behavior
```

### Recommended

```text
ALL
  ↓
Specific source
  ↓
Specific VIP
  ↓
Specific service
  ↓
Logging
```

Avoid:

```text
IPv6 ALL
    +
VIP ALL
    +
Service ALL
```

unless this is intentionally a lab or controlled migration environment.

---

# 29. Quick NSE Memory Map

```text
NAT64
IPv6 → IPv4

NAT46
IPv4 → IPv6

DNS64
DNS helper for IPv6-only clients

VIP
Destination translation / published endpoint

IP Pool
Translated source address

DNS Database
Authoritative/local DNS records

add-nat64-route
Installs NAT64-related routing information

session6
IPv6 session troubleshooting
```

---

# 30. One-Minute Troubleshooting Formula

```text
DNS
 ↓
IPv6 Destination
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

If NAT64/NAT46 does not work:

> **Don't start with the NAT configuration. Start from the client and walk the packet path.**

---

## 🔥 Interview / NSE7 Questions

### Q1 — What is the difference between DNS64 and NAT64?

**Answer:**

```text
DNS64 = DNS synthesis
NAT64 = packet/session translation
```

---

### Q2 — Which direction is NAT64?

```text
IPv6 → IPv4
```

---

### Q3 — Which direction is NAT46?

```text
IPv4 → IPv6
```

---

### Q4 — Why can DNS work while NAT64 fails?

Because DNS64 and NAT64 are separate processing stages.

```text
DNS64 ✔
NAT64 ✘
```

The client may successfully receive an IPv6 destination but still fail because of:

* VIP
* policy
* NAT pool
* routing
* session
* return path

---

### Q5 — What is the first thing to check when the server receives no traffic?

Follow the packet:

```text
Client
 → DNS
 → VIP
 → Policy
 → NAT
 → Route
 → Server
```

Do not assume that successful DNS resolution means successful NAT64.

---

## ⚠️ Production Notes

* Validate the exact behavior against the **FortiOS version and platform** before deploying.
* NAT64/NAT46 can introduce application compatibility issues.
* Pay special attention to DNS behavior.
* Verify return routing for translated addresses.
* Use narrow firewall policies instead of broad `ALL` rules.
* Enable appropriate traffic/session logging during troubleshooting.
* Avoid leaving `diagnose debug` enabled after testing.

---

## 🧠 Final Mental Model

```text
                  DNS64
                    │
                    ▼
IPv6 Client ────► FortiGate ────► IPv4 Server
                    │
                    │
                  NAT64
                    │
                    ▼

IPv4 Client ────► FortiGate ────► IPv6 Server
                    │
                    │
                  NAT46
```

### Remember:

```text
DNS64 → "What IPv6 address should I connect to?"

NAT64 → "Translate my IPv6 session to IPv4."

NAT46 → "Translate my IPv4 session to IPv6."

VIP → "Where should the destination go?"

IP Pool → "Which translated source address should be used?"

Session → "Did FortiGate actually build the translation?"
```
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

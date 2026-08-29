# 🔐 FortiGate Dial-up IPsec + BGP Checklist

> **FortiGate Dial-up IPsec VPN + BGP | Hub-and-Spoke | Dynamic Spokes | ADVPN**
>
> A practical deployment, validation, troubleshooting, and hardening checklist for **FortiGate Dial-up IPsec with BGP dynamic routing**.

---

## 📌 What This Checklist Covers

* [ ] Dial-up IPsec Hub configuration
* [ ] Dynamic Spoke connectivity
* [ ] IPsec Phase 1 / Phase 2
* [ ] IPsec Layer-3 interface addressing
* [ ] XAuth authentication
* [ ] BGP over IPsec
* [ ] BGP route advertisement
* [ ] Hub-and-Spoke routing
* [ ] ADVPN spoke-to-spoke shortcuts
* [ ] NAT-T
* [ ] Firewall policy validation
* [ ] IPsec / BGP troubleshooting
* [ ] Packet capture
* [ ] Production hardening

---

# 🗺️ 1. Reference Topology

```text
                         INTERNET / ISP
                              |
                              |
                         +----------+
                         | FGT-HUB  |
                         |          |
                         | AS 65000 |
                         +----+-----+
                              |
              +---------------+---------------+
              |               |               |
           IPsec           IPsec           IPsec
           Dial-up         Dial-up         Dial-up
              |               |               |
         +----+----+      +---+----+      +---+----+
         | SPOKE-1 |      | SPOKE-2 |      | SPOKE-3 |
         | FGT-2   |      | FGT-3   |      | FGT-4   |
         | AS65001 |      | AS65002 |      | AS65003 |
         +---------+      +---------+      +---------+
```

### Example Tunnel Addressing

```text
HUB:
    12.23.34.1

SPOKE-1:
    12.23.34.2

SPOKE-2:
    12.23.34.3

SPOKE-3:
    12.23.34.4
```

---

# 🎯 2. Architecture Validation

Before configuration:

* [ ] Hub has a reachable public IP / FQDN
* [ ] Spokes can reach the Hub over the Internet
* [ ] Spoke public addresses may be dynamic
* [ ] NAT traversal requirement has been identified
* [ ] Tunnel addressing plan is defined
* [ ] LAN subnet plan is defined
* [ ] BGP AS numbers are defined
* [ ] BGP Router-IDs are unique
* [ ] Hub-and-Spoke topology is documented
* [ ] ADVPN requirement has been identified
* [ ] Authentication method is selected
* [ ] IKE version is selected
* [ ] IPsec cryptographic suite is selected

---

# 🔐 3. Dial-up IPsec — Hub Checklist

Navigate to:

```text
VPN
└── IPsec Tunnels
    └── Custom
        └── Dial-up
```

### Phase 1

* [ ] Remote Gateway = Dial-up User
* [ ] Incoming Interface = Internet/WAN
* [ ] IKE version configured
* [ ] Authentication configured
* [ ] PSK configured
* [ ] Peer identity configured
* [ ] NAT-T considered
* [ ] DPD configured
* [ ] Encryption algorithm configured
* [ ] Integrity algorithm configured
* [ ] DH group configured
* [ ] Proposal matches Spokes

### Recommended Production Direction

```text
IKEv2
    +
AES-GCM / strong AES
    +
Strong DH / ECDH
    +
Strong authentication
    +
NAT-T when required
```

Avoid legacy cryptography unless interoperability requires it:

```text
❌ DES
❌ 3DES
❌ MD5
❌ Weak DH groups
```

---

# 🔑 4. XAuth Authentication Checklist

If XAuth is part of the design:

* [ ] XAuth enabled
* [ ] Authentication method selected
* [ ] Local users configured OR
* [ ] LDAP integration configured OR
* [ ] Active Directory integration configured
* [ ] Username/password validation tested
* [ ] Authentication failure logging reviewed
* [ ] XAuth requirement documented

### Authentication Flow

```text
Spoke
   |
   | IKE
   v
Hub
   |
   | XAuth
   v
User / LDAP / AD
   |
   | Authentication
   v
IPsec Tunnel
```

> **Security note:** For new production deployments, evaluate certificate-based authentication where appropriate instead of relying solely on shared secrets and user credentials.

---

# 🔐 5. IPsec Phase 2 Checklist

* [ ] Phase 2 configured
* [ ] Local selector defined
* [ ] Remote selector defined
* [ ] Encryption configured
* [ ] Integrity configured
* [ ] PFS requirement defined
* [ ] PFS DH group configured
* [ ] Auto-negotiate evaluated
* [ ] Phase 2 selectors permit required routed traffic
* [ ] Phase 2 configuration matches Spokes

Example routed VPN selectors:

```text
Local:
    0.0.0.0/0

Remote:
    0.0.0.0/0
```

> Exact selectors depend on the FortiOS release and VPN architecture.

---

# 🧩 6. IPsec Layer-3 Interface Checklist

BGP needs IP reachability to its neighbor.

Therefore:

* [ ] IPsec interface exists
* [ ] Tunnel IP addressing is defined
* [ ] Hub tunnel IP is unique
* [ ] Spoke tunnel IP is unique
* [ ] Hub can reach Spoke tunnel IP
* [ ] Spoke can reach Hub tunnel IP
* [ ] Interface is routable
* [ ] Correct source interface is used for BGP

Example:

```text
HUB
12.23.34.1/24

       |
       | IPsec
       |

SPOKE
12.23.34.2/24
```

---

# 🔥 7. Firewall Policy Checklist

## Hub — IPsec → LAN

* [ ] Incoming interface = IPsec
* [ ] Destination interface = LAN
* [ ] Correct source addresses
* [ ] Correct destination addresses
* [ ] Required services allowed
* [ ] NAT disabled for normal routed VPN traffic
* [ ] Logging enabled where appropriate

## Hub — LAN → IPsec

* [ ] Incoming interface = LAN
* [ ] Destination interface = IPsec
* [ ] Correct source
* [ ] Correct destination
* [ ] Required services allowed
* [ ] NAT disabled unless intentionally required
* [ ] Logging enabled where appropriate

### Important

```text
IPsec UP
    ≠
Application Connectivity
```

The complete forwarding path must work:

```text
LAN
 ↓
Routing
 ↓
Firewall Policy
 ↓
IPsec
 ↓
Remote Firewall
 ↓
Remote LAN
```

---

# 🧭 8. BGP Design Checklist

Before configuring BGP:

* [ ] Local AS defined
* [ ] Remote AS defined
* [ ] Router-ID defined
* [ ] Neighbor IP defined
* [ ] Neighbor reachability verified
* [ ] TCP/179 path verified
* [ ] Update-source requirement identified
* [ ] BGP policy requirements documented
* [ ] Prefix advertisements defined
* [ ] Prefix filtering considered
* [ ] Next-hop behavior understood

### Example

```text
HUB
AS 65000
12.23.34.1

        |
        | TCP/179
        |
        v

SPOKE
AS 65001
12.23.34.2
```

---

# 🏢 9. Hub BGP Checklist

Example:

```bash
config router bgp
    set as 65000
    set router-id 1.1.1.1

    config neighbor
        edit "12.23.34.2"
            set remote-as 65001
        next
    end
end
```

For multiple dynamic spokes:

* [ ] Determine whether static neighbors are appropriate
* [ ] Evaluate dynamic BGP neighbor capabilities supported by the FortiOS release
* [ ] Use peer-groups where appropriate
* [ ] Apply inbound/outbound route policy
* [ ] Verify maximum-prefix requirements
* [ ] Verify next-hop behavior

> **Important:** A real dial-up hub may require a dynamic-neighbor / peer-group design rather than manually defining every spoke as a static neighbor.

---

# 🏢 10. Spoke BGP Checklist

Example:

```bash
config router bgp
    set as 65001
    set router-id 2.2.2.2

    config neighbor
        edit "12.23.34.1"
            set remote-as 65000
        next
    end
end
```

Validate:

* [ ] Local AS correct
* [ ] Remote AS correct
* [ ] Router-ID unique
* [ ] Neighbor IP correct
* [ ] IPsec interface reachable
* [ ] TCP/179 reachable
* [ ] BGP session established

---

# 📢 11. BGP Network Advertisement Checklist

Define every LAN that should be advertised.

### Hub

```text
192.168.101.0/24
```

* [ ] LAN route exists
* [ ] Prefix advertised by BGP
* [ ] Prefix policy permits advertisement

### Spoke-1

```text
192.168.102.0/24
```

### Spoke-2

```text
192.168.103.0/24
```

### Spoke-3

```text
192.168.104.0/24
```

For every prefix:

* [ ] Prefix exists in routing table
* [ ] BGP network statement/policy is correct
* [ ] Prefix appears in BGP table
* [ ] Prefix is advertised to expected neighbor

---

# 🔄 12. BGP Route-Exchange Validation

Expected Hub BGP table:

```text
192.168.102.0/24 → 12.23.34.2
192.168.103.0/24 → 12.23.34.3
192.168.104.0/24 → 12.23.34.4
```

Checklist:

* [ ] Spoke-1 prefix received
* [ ] Spoke-2 prefix received
* [ ] Spoke-3 prefix received
* [ ] Correct next-hop
* [ ] Correct AS_PATH
* [ ] No unexpected route filtering
* [ ] No route recursion problem
* [ ] No routing loop
* [ ] Best path selected correctly

---

# 🌐 13. Hub-and-Spoke Routing Checklist

Basic architecture:

```text
             HUB
              |
       +------+------+ 
       |      |      |
    SPOKE-1 SPOKE-2 SPOKE-3
```

Example:

```text
SPOKE-1
192.168.102.0/24

        |
        v

HUB
12.23.34.1

        |
        v

SPOKE-3
192.168.104.0/24
```

Validate:

* [ ] Hub knows every spoke prefix
* [ ] Spokes know Hub prefixes
* [ ] Spokes receive required remote prefixes
* [ ] Return routes exist
* [ ] Next-hop is reachable
* [ ] No asymmetric routing problem

---

# 🔀 14. Spoke-to-Spoke Connectivity

### Without ADVPN

```text
SPOKE-1
   |
   v
 HUB
   |
   v
SPOKE-3
```

### With ADVPN

```text
SPOKE-1
   |
   | Dynamic Shortcut
   |
   +=================+
                     |
                  SPOKE-3
```

If direct spoke-to-spoke connectivity is required:

* [ ] ADVPN requirement confirmed
* [ ] Auto-Discovery Sender configured on Hub
* [ ] Auto-Discovery Receiver configured on Spokes
* [ ] BGP route advertisement designed for ADVPN
* [ ] Next-hop behavior validated
* [ ] Dynamic shortcut creation validated
* [ ] Spoke-to-spoke traffic tested

### Architecture

```text
IPsec
  +
ADVPN
  +
BGP
  =
Dynamic Secure Routing
```

---

# 🧠 15. ADVPN + BGP Mental Model

```text
IPsec
   |
   +── Secure Transport
   |
ADVPN
   |
   +── Dynamic Shortcut Discovery
   |
BGP
   |
   +── Route Advertisement
```

Think:

```text
IPsec = "How do I securely connect?"

BGP = "Where is the destination network?"

ADVPN = "Can I build a direct spoke-to-spoke path?"
```

---

# 🛡️ 16. NAT Validation

For normal routed VPN traffic:

```text
LAN
 |
 v
IPsec
 |
 v
Remote LAN
```

* [ ] NAT disabled on VPN policy
* [ ] Source IP preserved
* [ ] Return route exists
* [ ] No unintended SNAT
* [ ] NAT-T requirement evaluated separately

> NAT-T concerns the **transport of IPsec through NAT**; it is not the same thing as enabling NAT on a firewall policy.

---

# 🔎 17. IPsec Verification Checklist

Run:

```bash
diagnose vpn tunnel list
```

Verify:

* [ ] Correct tunnel exists
* [ ] Phase 2 is established
* [ ] Correct selectors
* [ ] Correct peer
* [ ] Traffic counters increase
* [ ] No repeated negotiation failures

Run:

```bash
diagnose vpn ike gateway list
```

Verify:

* [ ] IKE SA exists
* [ ] Correct peer
* [ ] Correct IKE version
* [ ] Correct proposal
* [ ] Correct authentication
* [ ] NAT-T status where applicable

---

# 🧭 18. BGP Verification Checklist

### BGP Summary

```bash
get router info bgp summary
```

Expected:

```text
Neighbor        V    AS       State
12.23.34.2      4    65001    Established
12.23.34.3      4    65002    Established
12.23.34.4      4    65003    Established
```

* [ ] State = Established
* [ ] Remote AS correct
* [ ] Message counters increasing
* [ ] Prefix counters correct

---

# 📋 19. BGP Neighbor Deep Check

```bash
get router info bgp neighbors
```

Check:

* [ ] BGP state
* [ ] Local AS
* [ ] Remote AS
* [ ] Router-ID
* [ ] Neighbor address
* [ ] Prefixes received
* [ ] Prefixes advertised
* [ ] Timers
* [ ] Update source
* [ ] Route policy

---

# 🛣️ 20. BGP Routing Table

Run:

```bash
get router info routing-table bgp
```

Expected:

```text
B  192.168.102.0/24
   via 12.23.34.2

B  192.168.103.0/24
   via 12.23.34.3

B  192.168.104.0/24
   via 12.23.34.4
```

Checklist:

* [ ] Expected prefix exists
* [ ] Correct next-hop
* [ ] Correct interface
* [ ] Correct administrative distance
* [ ] Correct best path
* [ ] No competing static route overriding BGP

---

# 🧪 21. Tunnel IP Connectivity Test

From Hub:

```bash
execute ping 12.23.34.2
execute ping 12.23.34.3
execute ping 12.23.34.4
```

Checklist:

* [ ] Hub → Spoke-1 reachable
* [ ] Hub → Spoke-2 reachable
* [ ] Hub → Spoke-3 reachable

Then test LAN prefixes:

```bash
execute ping 192.168.102.1
execute ping 192.168.103.1
execute ping 192.168.104.1
```

---

# 🔥 22. End-to-End Connectivity Checklist

Test:

```text
LAN-1
  |
  v
SPOKE
  |
  v
IPsec
  |
  v
HUB
  |
  v
IPsec
  |
  v
REMOTE SPOKE
  |
  v
LAN
```

Verify:

* [ ] ICMP
* [ ] TCP application traffic
* [ ] DNS if required
* [ ] HTTPS if required
* [ ] Return traffic
* [ ] Session establishment
* [ ] No unexpected NAT

---

# 🧪 23. IKE Debug Checklist

Start:

```bash
diagnose debug reset
diagnose debug application ike -1
diagnose debug enable
```

Stop:

```bash
diagnose debug disable
diagnose debug reset
```

Look for:

* [ ] Proposal mismatch
* [ ] Authentication failure
* [ ] Peer-ID mismatch
* [ ] PSK failure
* [ ] DH mismatch
* [ ] IKE version mismatch
* [ ] NAT-T negotiation
* [ ] DPD events
* [ ] Phase 2 selector mismatch

> Run debugging during a controlled troubleshooting window and stop it after collecting the required information.

---

# 🧪 24. BGP Packet Capture

Capture TCP/179:

```bash
diagnose sniffer packet any 'tcp port 179' 4 0 l
```

Check:

* [ ] SYN leaves source
* [ ] SYN/ACK returns
* [ ] TCP session establishes
* [ ] BGP packets are exchanged

Mental model:

```text
No SYN
   ↓
IP reachability / routing problem

SYN but no SYN/ACK
   ↓
Policy / routing / remote-side problem

TCP established but BGP fails
   ↓
BGP configuration problem
```

---

# 📡 25. IKE Packet Capture

```bash
diagnose sniffer packet any 'udp port 500 or udp port 4500' 4 0 l
```

Validate:

* [ ] UDP/500 visible when applicable
* [ ] UDP/4500 visible when NAT-T is used
* [ ] Bidirectional traffic exists
* [ ] No unexpected packet loss

---

# 🔐 26. ESP Packet Capture

```bash
diagnose sniffer packet any 'ip proto 50' 4 0 l
```

Validate:

* [ ] ESP traffic exists when applicable
* [ ] Traffic is bidirectional
* [ ] Packet counters increase
* [ ] Encrypted traffic leaves expected interface

> When NAT-T encapsulation is used, ESP may be carried inside UDP/4500 rather than appearing as native IP protocol 50.

---

# 🧪 27. FortiGate Flow Debug

Example:

```bash
diagnose debug flow filter addr 192.168.104.10
diagnose debug flow show console enable
diagnose debug flow trace start 20
diagnose debug enable
```

Stop:

```bash
diagnose debug disable
diagnose debug flow trace stop
diagnose debug reset
```

Check:

* [ ] Ingress interface
* [ ] Routing decision
* [ ] Policy ID
* [ ] NAT decision
* [ ] Egress interface
* [ ] Session creation
* [ ] Deny reason
* [ ] Reverse path

---

# 🚨 28. Troubleshooting Decision Tree

```text
                VPN Problem
                    |
                    v
             Is Phase 1 UP?
               /       \
             NO         YES
             |           |
          Check IKE      v
                    Is Phase 2 UP?
                     /       \
                   NO         YES
                   |           |
             Check selectors    v
                          Is BGP UP?
                           /     \
                         NO       YES
                         |         |
                   Check TCP/179    v
                   + routing    Are routes
                                  correct?
                                  /    \
                                NO      YES
                                |        |
                         Check BGP       v
                         advertisements  Test policy
                                         + traffic
```

---

# ❌ 29. IPsec DOWN — Checklist

If Phase 1 is down:

* [ ] WAN interface correct
* [ ] Remote gateway correct
* [ ] IKE version matches
* [ ] PSK matches
* [ ] Peer ID matches
* [ ] Proposal matches
* [ ] Encryption matches
* [ ] Integrity matches
* [ ] DH group matches
* [ ] NAT-T considered
* [ ] UDP/500 permitted
* [ ] UDP/4500 permitted when required
* [ ] DPD behavior checked
* [ ] XAuth configuration checked

---

# ❌ 30. IPsec UP but BGP DOWN

If:

```text
IPsec = UP
BGP = DOWN
```

Check:

* [ ] IPsec interface addressing
* [ ] BGP neighbor IP
* [ ] Neighbor reachability
* [ ] TCP/179
* [ ] Firewall policy
* [ ] Local AS
* [ ] Remote AS
* [ ] Router-ID
* [ ] Update-source
* [ ] Route to neighbor
* [ ] BGP configuration

---

# ❌ 31. BGP Stuck in Active

Investigate:

```text
❌ Neighbor unreachable
❌ TCP/179 blocked
❌ Wrong neighbor IP
❌ Wrong remote AS
❌ Wrong update source
❌ Incorrect routing
❌ IPsec interface problem
❌ Firewall policy problem
```

Workflow:

```text
Ping neighbor
     ↓
Check route
     ↓
Check TCP/179
     ↓
Check BGP configuration
     ↓
Check BGP state
```

---

# ❌ 32. BGP Established but No Routes

If:

```text
BGP = Established
Routes = Missing
```

Check:

* [ ] Network statement
* [ ] Prefix exists in routing table
* [ ] Route-map / policy
* [ ] Prefix-list
* [ ] Inbound filtering
* [ ] Outbound filtering
* [ ] BGP advertisement
* [ ] Next-hop
* [ ] Best-path selection

---

# ❌ 33. Route Exists but Traffic Fails

If:

```text
BGP = Established
Route = Present
IPsec = UP
Traffic = FAIL
```

Check:

* [ ] Firewall policy
* [ ] NAT
* [ ] Return route
* [ ] Phase 2 selectors
* [ ] Remote firewall policy
* [ ] Local host gateway
* [ ] Remote host gateway
* [ ] Asymmetric routing
* [ ] MTU / fragmentation
* [ ] Session state

---

# 🔄 34. Complete Dependency Chain

```text
Internet Reachability
        ↓
IKE / Phase 1
        ↓
IPsec / Phase 2
        ↓
IPsec Interface
        ↓
Tunnel IP Reachability
        ↓
TCP / 179
        ↓
BGP Session
        ↓
BGP Route Advertisement
        ↓
Routing Table
        ↓
Firewall Policy
        ↓
Application Traffic
```

### Golden Rule

> **Never troubleshoot BGP before proving IPsec and IP reachability.**

---

# 🛡️ 35. Production Security Checklist

### Cryptography

* [ ] Prefer IKEv2
* [ ] Use modern encryption
* [ ] Use strong integrity/authentication
* [ ] Use strong DH/ECDH groups
* [ ] Disable weak legacy algorithms
* [ ] Use strong unique PSKs where PSK authentication is required
* [ ] Consider certificate authentication for scalable deployments

### Authentication

* [ ] XAuth requirement reviewed
* [ ] User authentication source secured
* [ ] Least-privilege access applied
* [ ] Credentials protected
* [ ] Authentication logs monitored

### Routing

* [ ] BGP route filtering configured
* [ ] Prefix advertisements limited
* [ ] Maximum-prefix protection considered
* [ ] Unexpected routes rejected
* [ ] AS_PATH policy reviewed
* [ ] Next-hop behavior validated

### Firewall

* [ ] VPN policies restricted
* [ ] Unnecessary services disabled
* [ ] NAT disabled for normal routed VPN traffic
* [ ] Logging enabled where useful
* [ ] Administrative access restricted

---

# 📊 36. Protocol Responsibility Matrix

| Component           | Primary Responsibility                    |
| ------------------- | ----------------------------------------- |
| **IKE**             | Peer negotiation and key management       |
| **IPsec**           | Secure encrypted transport                |
| **Phase 1**         | IKE security association                  |
| **Phase 2**         | IPsec data-plane security association     |
| **IPsec Interface** | Layer-3 routing over VPN                  |
| **XAuth**           | Additional user authentication            |
| **NAT-T**           | IPsec traversal through NAT               |
| **BGP**             | Dynamic route exchange                    |
| **TCP/179**         | BGP transport                             |
| **ADVPN**           | Dynamic spoke-to-spoke shortcut discovery |
| **Firewall Policy** | Traffic authorization                     |
| **Routing Table**   | Final forwarding decision                 |

---

# ⚡ 37. Fast Troubleshooting Matrix

| Symptom                   | First Check                          |
| ------------------------- | ------------------------------------ |
| Phase 1 DOWN              | IKE / PSK / proposal / peer identity |
| Phase 2 DOWN              | Selectors / PFS / proposal           |
| Tunnel UP, BGP DOWN       | IP reachability / TCP 179            |
| BGP Active                | Neighbor reachability / TCP 179      |
| BGP Established, no route | Advertisement / filtering            |
| Route exists, ping fails  | Policy / NAT / return route          |
| Spoke-to-spoke fails      | ADVPN / next-hop / shortcut          |
| Internet NAT issue        | NAT-T / firewall policy              |
| Intermittent VPN          | Underlay / packet loss / DPD         |
| Asymmetric traffic        | Routing / BGP best path              |

---

# 🧠 38. Exam Mental Model

```text
                 DIAL-UP IPSEC
                       |
              +--------+--------+
              |                 |
             IKE              IPsec
              |                 |
         Phase 1             Phase 2
              |                 |
           XAuth          IPsec Interface
                                |
                                v
                              BGP
                                |
                         TCP / 179
                                |
                       Route Exchange
                                |
                    +-----------+-----------+
                    |           |           |
                  LAN-2       LAN-3       LAN-4
```

Remember:

```text
IPsec = Secure Transport

BGP = Dynamic Routing

ADVPN = Dynamic Shortcut

XAuth = Authentication

Firewall Policy = Traffic Authorization
```

---

# 🚀 39. Production Architecture

```text
                         INTERNET
                             |
                             |
                        +---------+
                        | FGT HUB |
                        | AS65000 |
                        +----+----+
                             |
              +--------------+--------------+
              |              |              |
           Dial-up        Dial-up        Dial-up
           IPsec          IPsec          IPsec
              |              |              |
          +---+---+      +---+---+      +---+---+
          |Spoke-1|      |Spoke-2|      |Spoke-3|
          |65001  |      |65002  |      |65003  |
          +---+---+      +---+---+      +---+---+
              |              |              |
            LAN-1          LAN-2          LAN-3
```

### Recommended Control Plane

```text
IKEv2
   +
Strong IPsec Cryptography
   +
IPsec Interface
   +
BGP
   +
Route Filtering
   +
ADVPN when direct spoke-to-spoke traffic is required
```

---

# 🏆 40. Final Deployment Checklist

## Pre-Deployment

* [ ] Topology documented
* [ ] IP addressing documented
* [ ] BGP AS plan documented
* [ ] Authentication design documented
* [ ] Cryptographic policy approved
* [ ] ADVPN requirement confirmed

## IPsec

* [ ] Dial-up Phase 1 configured
* [ ] Phase 2 configured
* [ ] IKE proposals matched
* [ ] IPsec selectors validated
* [ ] Tunnel IPs configured
* [ ] NAT-T tested
* [ ] DPD tested

## BGP

* [ ] Local AS configured
* [ ] Remote AS configured
* [ ] Router-ID configured
* [ ] Neighbor reachability verified
* [ ] TCP/179 verified
* [ ] BGP Established
* [ ] LAN prefixes advertised
* [ ] Routes received
* [ ] Next-hop verified
* [ ] Route filtering applied

## ADVPN

* [ ] ADVPN requirement confirmed
* [ ] Sender configured on Hub
* [ ] Receiver configured on Spokes
* [ ] Shortcut discovery verified
* [ ] Spoke-to-spoke traffic tested

## Firewall

* [ ] LAN → VPN policy exists
* [ ] VPN → LAN policy exists
* [ ] NAT disabled where appropriate
* [ ] Return traffic allowed
* [ ] Logging configured

## Validation

* [ ] Tunnel ping works
* [ ] LAN ping works
* [ ] BGP routes visible
* [ ] Application traffic works
* [ ] Failover behavior tested
* [ ] Debugging procedure documented

## Security

* [ ] Weak cryptography removed
* [ ] Strong authentication used
* [ ] BGP filtering configured
* [ ] Administrative access restricted
* [ ] Logs monitored
* [ ] Configuration backed up

---

# 🔗 SheynShield Resources

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
 

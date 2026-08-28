# 🔐 Dial-up IPsec + BGP

> **Scenario:** Dial-up IPsec VPN + Dynamic Routing using BGP  
> **Topology:** Hub-and-Spoke / Dynamic Spokes

---

## 🗺️ Topology

```text
                         Internet / ISP
                              |
                         Public Network
                              |
                         +----------+
                         |  FGT-1   |
                         |   HUB    |
                         |          |
                         | BGP AS65000
                         +----+-----+
                              |
                  =========================
                  |           |            |
               IPsec       IPsec        IPsec
               Dial-up     Dial-up      Dial-up
                  |           |            |
              +---+---+   +---+---+    +---+---+
              | FGT-2 |   | FGT-3 |    | FGT-4 |
              | SPOKE |   | SPOKE |    | SPOKE |
              |       |   |       |    |       |
              | AS65001   | AS65002    | AS65003
              +-------+   +-------+    +-------+
````

---

# 1. 🎯 Concept

## Dial-up IPsec

Dial-up IPsec is used when:

* Spoke public IP is dynamic
* Spoke is behind NAT
* Hub has a static/public IP
* Multiple remote FortiGates connect to one Hub
* Dynamic routing is required over IPsec

```text
Spoke
  |
  | Dynamic Public IP
  |
  |---- Internet ----|
                     |
                  FGT-1
                   HUB
              Static Public IP
```

---

# 2. 🔄 Control Plane vs Data Plane

```text
                DIAL-UP IPSEC
                     |
          +----------+----------+
          |                     |
       IKE/Phase 1          IPsec/Phase 2
          |                     |
     Authentication        Encrypted Traffic
     Negotiation            Data Plane
          |                     |
          +----------+----------+
                     |
                    BGP
                     |
              Dynamic Routing
```

---

# 3. 🔐 IPsec Phase 1

## Hub — FGT-1

Create:

```text
VPN
 └── IPsec Tunnels
      └── Custom
           └── Dial-up
```

### Basic Parameters

```text
Remote Gateway:
    Dial-up User

Incoming Interface:
    Internet / WAN

IKE Version:
    IKEv1 or IKEv2

Authentication:
    Pre-shared Key

Mode:
    Aggressive   # when using IKEv1 + dynamic peer identity

Encryption:
    AES

Integrity:
    SHA256

DH Group:
    14
```

> Prefer modern cryptographic algorithms in production. Avoid legacy combinations such as DES/3DES/MD5/DH5 unless required for interoperability or lab purposes.

---

# 4. 🔑 XAuth

For dial-up authentication:

```text
FGT-1
 |
 +-- XAuth
      |
      +-- User Authentication
              |
              +-- Local User
              +-- LDAP
              +-- Active Directory
              +-- FSSO
```

Example:

```text
Username:
    u1

Password:
    ********
```

### Authentication Flow

```text
Spoke
  |
  | IKE negotiation
  |
  |---- XAuth credentials ---->
  |
  |                         FGT-1
  |                           |
  |                           +-- LDAP / AD
  |                           |
  |<---- Authentication ------+
  |
  | IPsec Tunnel
```

---

# 5. 🔐 IPsec Phase 2

Example:

```text
Phase 2

Encryption:
    AES

Authentication:
    SHA256

PFS:
    Enable

PFS DH Group:
    14

Auto-negotiate:
    Enable
```

For routing over the tunnel, make sure the Phase 2 selectors allow the required traffic.

Example:

```text
Source:
    0.0.0.0/0

Destination:
    0.0.0.0/0
```

> Exact selectors depend on the VPN design and FortiOS version.

---

# 6. 🧩 IPsec Interface

On the Hub:

```text
Interface:
    IPsec Dial-up Interface

IP:
    12.23.34.1/24
```

Example Spoke:

```text
FGT-2

IP:
    12.23.34.2/24
```

Another Spoke:

```text
FGT-3

IP:
    12.23.34.3/24
```

Another Spoke:

```text
FGT-4

IP:
    12.23.34.4/24
```

Conceptually:

```text
             FGT-1
          12.23.34.1
               |
       +-------+-------+
       |       |       |
       |       |       |
      .2      .3      .4
     FGT-2   FGT-3   FGT-4
```

---

# 7. 🚦 FortiGate Policies

## Hub

### Incoming

```text
Interface:
    IPsec Dial-up

Destination:
    LAN

NAT:
    Disable
```

### Outgoing

```text
Source:
    LAN

Destination:
    IPsec Dial-up

NAT:
    Disable
```

For BGP, remember that the routing protocol itself also needs to be permitted appropriately according to the interface and policy design.

---

# 8. 🧭 BGP Concept

BGP runs **over the IPsec tunnel**.

```text
FGT-2
12.23.34.2
     |
     | TCP/179
     |
     | IPsec
     |
     |
12.23.34.1
FGT-1
```

BGP:

```text
FGT-1
AS 65000
 |
 +---------------- FGT-2
 |                  AS 65001
 |
 +---------------- FGT-3
 |                  AS 65002
 |
 +---------------- FGT-4
                    AS 65003
```

---

# 9. 🏢 Hub BGP — FGT-1

Example:

```text
config router bgp
    set as 65000
    set router-id 1.1.1.1

    config neighbor
        edit "12.23.34.2"
            set remote-as 65001
        next

        edit "12.23.34.3"
            set remote-as 65002
        next

        edit "12.23.34.4"
            set remote-as 65003
        next
    end
end
```

> For a real dial-up hub, static neighbor definitions may not be appropriate when spokes are dynamically created. Use the FortiGate BGP capabilities supported by your FortiOS release for dynamic/dial-up neighbors or peer-groups.

---

# 10. 🏢 Spoke BGP — FGT-2

```text
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

---

# 11. 🏢 Spoke BGP — FGT-3

```text
config router bgp
    set as 65002
    set router-id 3.3.3.3

    config neighbor
        edit "12.23.34.1"
            set remote-as 65000
        next
    end
end
```

---

# 12. 🏢 Spoke BGP — FGT-4

```text
config router bgp
    set as 65003
    set router-id 4.4.4.4

    config neighbor
        edit "12.23.34.1"
            set remote-as 65000
        next
    end
end
```

---

# 13. 📢 Advertise LAN Networks

## FGT-1

Example:

```text
192.168.101.0/24
```

```text
config router bgp
    config network
        edit 1
            set prefix 192.168.101.0 255.255.255.0
        next
    end
end
```

---

## FGT-2

```text
192.168.102.0/24
```

```text
config router bgp
    config network
        edit 1
            set prefix 192.168.102.0 255.255.255.0
        next
    end
end
```

---

## FGT-3

```text
192.168.103.0/24
```

```text
config router bgp
    config network
        edit 1
            set prefix 192.168.103.0 255.255.255.0
        next
    end
end
```

---

## FGT-4

```text
192.168.104.0/24
```

```text
config router bgp
    config network
        edit 1
            set prefix 192.168.104.0 255.255.255.0
        next
    end
end
```

---

# 14. 🔄 Route Advertisement

Example:

```text
FGT-2
192.168.102.0/24
       |
       | BGP
       |
       v
FGT-1
       |
       | BGP
       |
       +----> FGT-3
       |
       +----> FGT-4
```

Hub learns:

```text
BGP Table

192.168.102.0/24 -> 12.23.34.2
192.168.103.0/24 -> 12.23.34.3
192.168.104.0/24 -> 12.23.34.4
```

---

# 15. 🌐 Hub-and-Spoke Routing

```text
                 FGT-1
                  HUB
             12.23.34.1
             AS 65000
             /    |    \
            /     |     \
           /      |      \
       FGT-2    FGT-3    FGT-4
       AS65001  AS65002  AS65003
          |        |        |
     LAN-102   LAN-103   LAN-104
```

Example:

```text
FGT-2 wants:

192.168.104.0/24

Route:

192.168.104.0/24
       |
       v
12.23.34.1
       |
       v
FGT-1
       |
       v
FGT-4
```

---

# 16. 🔀 Spoke-to-Spoke

Basic Hub-and-Spoke:

```text
FGT-2 ---> FGT-1 ---> FGT-4
```

Traffic passes through Hub.

With ADVPN / shortcut mechanisms:

```text
FGT-2 ================= FGT-4
        Direct IPsec
        Shortcut
```

For dynamic spoke-to-spoke connectivity, combine:

```text
IPsec
 +
Auto-Discovery / ADVPN
 +
BGP
```

Concept:

```text
             FGT-1
              HUB
             /   \
            /     \
           /       \
        FGT-2 ===== FGT-4
             |
        Direct Tunnel
```

---

# 17. 🧠 ADVPN + BGP

Typical architecture:

```text
             HUB
              |
       +------+------+
       |             |
     Spoke          Spoke
       \             /
        \           /
         +---------+
         Shortcut
```

Roles:

```text
IPsec:
    Secure transport

ADVPN:
    Dynamic shortcut discovery

BGP:
    Route advertisement

BGP + ADVPN:
    Dynamic routing + dynamic spoke-to-spoke connectivity
```

---

# 18. 🛡️ No NAT for VPN Routing

For normal VPN-to-VPN/LAN routing:

```text
LAN
 |
 v
IPsec
 |
 v
Remote LAN
```

Use:

```text
NAT = Disable
```

Otherwise the original source IP can be hidden and routing behavior may not be what you expect.

---

# 19. 🔎 Troubleshooting — IPsec

### Show IPsec tunnels

```bash
diagnose vpn tunnel list
```

### Show IKE gateways

```bash
diagnose vpn ike gateway list
```

### Debug IKE

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

---

# 20. 🔎 Troubleshooting — BGP

### Show BGP summary

```bash
get router info bgp summary
```

Expected:

```text
Neighbor        V    AS       MsgRcvd
12.23.34.2      4    65001    ...
12.23.34.3      4    65002    ...
12.23.34.4      4    65003    ...
```

State should be:

```text
Established
```

---

# 21. 📋 Show BGP Neighbors

```bash
get router info bgp neighbors
```

Check:

```text
BGP state
Remote AS
Local AS
Remote router-id
Prefixes received
Prefixes advertised
Timers
```

---

# 22. 📋 Show BGP Routes

```bash
get router info routing-table bgp
```

Example:

```text
B    192.168.102.0/24
      via 12.23.34.2

B    192.168.103.0/24
      via 12.23.34.3

B    192.168.104.0/24
      via 12.23.34.4
```

---

# 23. 🧪 Ping Test

From FGT-1:

```bash
execute ping 12.23.34.2
execute ping 12.23.34.3
execute ping 12.23.34.4
```

Test LAN:

```bash
execute ping 192.168.102.1
execute ping 192.168.103.1
execute ping 192.168.104.1
```

---

# 24. 🧪 Routing Test

Check:

```bash
get router info routing-table all
```

Expected:

```text
192.168.102.0/24
        |
        +-- IPsec -> FGT-2

192.168.103.0/24
        |
        +-- IPsec -> FGT-3

192.168.104.0/24
        |
        +-- IPsec -> FGT-4
```

---

# 25. 🔥 Packet Flow

Example:

```text
Host A
192.168.102.10
      |
      v
FGT-2
      |
      | BGP route
      |
      | IPsec encryption
      |
      v
FGT-1
      |
      | Routing
      |
      | IPsec
      |
      v
FGT-4
      |
      v
192.168.104.10
Host B
```

---

# 26. 🧩 Important Dependencies

For BGP over Dial-up IPsec:

```text
1. IPsec Phase 1
        ↓
2. IPsec Phase 2
        ↓
3. IPsec Interface
        ↓
4. IP addressing
        ↓
5. Routing reachability
        ↓
6. TCP/179
        ↓
7. BGP Neighbor
        ↓
8. BGP Established
        ↓
9. Route Advertisement
        ↓
10. End-to-End Connectivity
```

---

# 27. ⚠️ Common Problems

## IPsec is DOWN

Check:

```text
IKE version
PSK
Encryption
Integrity
DH group
Peer ID
XAuth
NAT-T
Local/Remote gateway
```

---

## IPsec UP but BGP DOWN

Check:

```text
IPsec interface IP
BGP neighbor IP
TCP/179
Firewall policy
Routing table
Source IP
BGP AS number
```

---

## BGP stuck in Active

Usually investigate:

```text
❌ No IP reachability
❌ TCP/179 blocked
❌ Wrong neighbor IP
❌ Wrong remote AS
❌ Wrong update-source/source IP
❌ IPsec interface problem
❌ Routing problem
```

---

# 28. 🧪 Useful Packet Capture

Capture BGP:

```bash
diagnose sniffer packet any 'tcp port 179' 4 0 l
```

Capture IKE:

```bash
diagnose sniffer packet any 'udp port 500 or udp port 4500' 4 0 l
```

Capture ESP:

```bash
diagnose sniffer packet any 'ip proto 50' 4 0 l
```

---

# 29. 🧪 Flow Debug

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

---

# 30. 🧠 Mental Model

```text
                BGP
                 |
          Route Advertisement
                 |
             TCP / 179
                 |
           IPsec Interface
                 |
          IPsec Phase 2
                 |
           IPsec Phase 1
                 |
               IKE
                 |
              Internet
```

Think:

```text
BGP = "Where is the network?"

IPsec = "How do I securely reach the peer?"

ADVPN = "Can the spokes build a direct shortcut?"

XAuth = "Who is allowed to connect?"

BGP = "Which routes should I use?"
```

---

# 31. 🏆 Recommended Production Architecture

```text
                    INTERNET
                       |
                       |
                    FGT-HUB
                       |
          +------------+------------+
          |            |            |
        IPsec        IPsec        IPsec
       Dial-up      Dial-up      Dial-up
          |            |            |
        FGT-2        FGT-3        FGT-4
          |            |            |
        LAN-2        LAN-3        LAN-4
```

Control:

```text
IKEv2
   +
AES
   +
SHA-256 / stronger integrity
   +
Strong DH / ECDH
   +
XAuth / certificate authentication
   +
BGP
   +
ADVPN when spoke-to-spoke is required
```

---

# 32. 📌 Final  

| Component           | Purpose                                     |
| ------------------- | ------------------------------------------- |
| **IPsec**           | Secure tunnel                               |
| **IKE**             | Negotiation and key management              |
| **Phase 1**         | IKE/management tunnel                       |
| **Phase 2**         | IPsec/data tunnel                           |
| **XAuth**           | User authentication                         |
| **NAT-T**           | IPsec through NAT                           |
| **IPsec Interface** | Layer-3 interface for routing               |
| **BGP**             | Dynamic route exchange                      |
| **TCP/179**         | BGP transport                               |
| **ADVPN**           | Dynamic spoke-to-spoke shortcut             |
| **Auto Discovery**  | Peer/shortcut discovery                     |
| **BGP + ADVPN**     | Dynamic routing + direct spoke connectivity |

---

# 🎯 Exam Mental Picture

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

> **Key idea:** First establish the IPsec connectivity and a routable IPsec interface. Then establish BGP over that interface. BGP does not replace IPsec; BGP provides routing information through the secure IPsec transport.

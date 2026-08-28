# FortiGate Dial-Up IPsec + XAuth + LDAP + BGP

> **NSE 7 Field Note**
>
> A practical cheat sheet for building and troubleshooting a FortiGate Dial-Up IPsec VPN with XAuth/LDAP authentication and BGP routing.

---

## ⚠️ Lab Note

This lab uses legacy cryptographic parameters:

- IKEv1
- Aggressive Mode
- DES
- MD5
- DH Group 5

These values are included for **lab and NSE 7 learning purposes only**.

Do **not** use them as production security recommendations.

For production deployments, use current FortiOS-supported cryptographic algorithms and authentication methods.

---

# 1. Topology

```text
                         Internet / ISP
                              |
                              |
                           FGT-1
                        HQ / Hub
                        AS 65001
                       12.23.34.1
                         /      \
                        /        \
                   IPsec          IPsec
                      /              \
                     /                \
                 FGT-2              FGT-4
              AS 65002             AS 65004
           12.23.34.2          12.23.34.4
                                      |
                                      |
                                    FGT-3
                                  Transit
````

---

# 2. Authentication Flow

FGT-1 acts as the Dial-Up IPsec server.

Active Directory is configured as an LDAP authentication server.

```text
                    Active Directory
                           |
                          LDAP
                           |
                           v
                         FGT-1
                           |
                         XAuth
                           |
                           v
                    Dial-Up IPsec
                           |
                    +------+------+
                    |             |
                  FGT-2         FGT-4
```

If FSSO is required:

```text
Security Fabric
      |
      +── External Connectors
                |
               FSSO
```

FortiGate can operate as an XAuth server and forward authentication requests to external LDAP/RADIUS authentication servers. ([Fortinet Web][2])

---

# 3. FGT-1 — Configure LDAP

Configure Active Directory as an LDAP server.

Conceptually:

```text
config user ldap
    edit "AD-LDAP"
        set server <AD_SERVER_IP>
        ...
    next
end
```

Then create the required user groups.

Example:

```text
AD Users
    |
    +── VPN-Users
    |
    +── Network-Admins
    |
    +── Branch-Users
```

These groups can later be used for XAuth authentication.

---

# 4. FGT-1 — Dial-Up IPsec

Navigate to:

```text
VPN
└── IPsec Tunnels
```

Create a **Custom IPsec VPN**.

## Phase 1

```text
Type:
    Dial-Up User

Incoming Interface:
    ISP / WAN

Remote Gateway:
    Dial-Up User

Authentication:
    Pre-shared Key

IKE Version:
    IKEv1

Mode:
    Aggressive Mode

Peer ID:
    Any / Specific Peer ID

XAuth:
    Enable

Authentication Server:
    LDAP / FSSO
```

---

# 5. FGT-1 — Lab Phase 1 Parameters

```text
Encryption:
    DES

Authentication:
    MD5

DH Group:
    5

PSK:
    <LAB_PSK>
```

> **Security Warning:** DES, MD5, DH5, and IKEv1 Aggressive Mode are legacy lab parameters.

---

# 6. FGT-1 — XAuth

Configure the valid users or user groups.

Example:

```text
XAuth
  |
  +── User Group
          |
          +── VPN-Users
          |
          +── Branch-Users
```

Authentication flow:

```text
FGT-2 / FGT-4
       |
       | Username + Password
       v
     FGT-1
       |
       | LDAP
       v
Active Directory
```

The FortiGate dial-up server can challenge the client for XAuth credentials and validate them through LDAP/RADIUS. ([Fortinet Web][2])

---

# 7. FGT-1 — Device Creation

In the advanced IPsec configuration:

```text
Device Creation:
    Enable
```

Conceptually:

```text
                  Dial-Up Tunnel
                        |
          +-------------+-------------+
          |             |             |
        Peer-1        Peer-2        Peer-3
          |             |             |
      Dynamic        Dynamic       Dynamic
      Device         Device        Device
```

This becomes important when multiple dynamic peers use the same dial-up configuration.

---

# 8. FGT-1 — Add Route

For a simple single-peer/single-tunnel design:

```text
Add Route:
    Enable
```

For multiple dynamic peers using the same tunnel interface, route installation must be designed carefully.

A Dynamic IPsec + BGP design may use:

```text
Add Route:
    Disable

Auto-Discovery Sender:
    Enable

Auto-Discovery Receiver:
    Enable

Device Creation:
    Enable
```

Fortinet documents `add-route disable` for dynamic VPN designs where multiple dial-up tunnels can reach the same advertised network and routing/BGP is used for dynamic network discovery. ([Fortinet Docs][1])

---

# 9. FGT-1 — Phase 2

Configure the required local and remote networks.

Example:

```text
Local:
    HQ Networks

Remote:
    Branch Networks
```

Lab parameters:

```text
Encryption:
    DES

Authentication:
    MD5

PFS:
    Enable

PFS DH Group:
    5

Auto-negotiate:
    Enable
```

---

# 10. FGT-1 — IPsec Interface

Configure the tunnel interface:

```text
IP Address:
    12.23.34.1/24

Remote IP:
    12.23.34.2
```

Test:

```bash
execute ping 12.23.34.2
```

Expected:

```text
FGT-1
12.23.34.1
     |
     | IPsec
     |
12.23.34.2
FGT-2
```

---

# 11. FGT-1 — Firewall Policies

Create policies in both directions.

## Incoming

```text
Dial-Up / IPsec
        |
        v
       LAN
```

## Outgoing

```text
LAN
 |
 v
Dial-Up / IPsec
```

Example:

```text
Source:
    Dial-Up / IPsec / LAN

Destination:
    LAN / Dial-Up / IPsec

Service:
    Required Services

NAT:
    Disable

Logging:
    Enable
```

If branches need to reach Active Directory services at HQ, make sure the required DNS/LDAP/Kerberos/application traffic is permitted.

---

# 12. FGT-1 — BGP

Configure:

```text
Router ID:
    1.1.1.1

Local AS:
    65001

Network:
    192.168.101.0/24
```

Neighbor:

```text
Neighbor:
    12.23.34.2

Remote AS:
    65002

Next-Hop-Self:
    Enable
```

Logical view:

```text
              BGP
               |
               v
FGT-1 ---------------- FGT-2
AS 65001               AS 65002
     |                      |
     |                      |
192.168.101.0/24      192.168.102.0/24
```

---

# 13. FGT-2 — Dial-Up IPsec Client

Create a custom IPsec VPN.

```text
VPN
└── IPsec Tunnels
```

## Phase 1

```text
Type:
    Custom

Remote Gateway:
    FGT-1 Static IP

Outgoing Interface:
    ISP / WAN

Authentication:
    Pre-shared Key

IKE Version:
    IKEv1

Mode:
    Aggressive Mode

Peer ID:
    Any / Specific Peer ID

XAuth:
    Enable as Client
```

---

# 14. FGT-2 — XAuth Credentials

FGT-2 provides the XAuth username/password.

Example:

```text
Username:
    u1

Password:
    <LAB_PASSWORD>
```

Authentication flow:

```text
FGT-2
  |
  | XAuth
  v
FGT-1
  |
  | LDAP
  v
Active Directory
```

Fortinet documents the FortiGate dial-up client as an XAuth client when the remote peer acts as the XAuth server. ([Fortinet Web][2])

---

# 15. FGT-2 — Phase 2

Lab parameters:

```text
Encryption:
    DES

Authentication:
    MD5

PFS:
    Enable

PFS DH Group:
    5

Auto-negotiate:
    Enable
```

Configure the required local and remote subnets.

---

# 16. FGT-2 — IPsec Interface

```text
IP Address:
    12.23.34.2/24

Remote:
    12.23.34.1
```

Test:

```bash
execute ping 12.23.34.1
```

---

# 17. FGT-2 — Firewall Policies

## Incoming

```text
IPsec
  |
  v
LAN
```

## Outgoing

```text
LAN
 |
 v
IPsec
```

Example:

```text
Source:
    IPsec / LAN

Destination:
    LAN / IPsec

Service:
    Required Services

NAT:
    Disable

Logging:
    Enable
```

---

# 18. FGT-2 — BGP

Configure:

```text
Router ID:
    2.2.2.2

Local AS:
    65002

Network:
    192.168.102.0/24
```

Neighbor:

```text
Neighbor:
    12.23.34.1

Remote AS:
    65001

Next-Hop-Self:
    Enable
```

Expected:

```text
192.168.101.0/24
        |
       BGP
        |
192.168.102.0/24
```

---

# 19. FGT-4 — Branch Behind FGT-3

Topology:

```text
FGT-1
  |
ISP Router
  |
FGT-3
  |
FGT-4
```

FGT-4 establishes the IPsec connection directly to FGT-1 logically, while FGT-3 provides the transit path.

---

# 20. FGT-4 — Phase 1

```text
Remote Gateway:
    FGT-1 Static IP

Outgoing Interface:
    Interface toward FGT-3

Authentication:
    Pre-shared Key

XAuth:
    Enable as Client

Username:
    u2

Password:
    <LAB_PASSWORD>
```

---

# 21. FGT-4 — Phase 2

```text
Encryption:
    DES

Authentication:
    MD5

PFS:
    Enable

PFS DH Group:
    5

Auto-negotiate:
    Enable
```

---

# 22. FGT-4 — IPsec Interface

```text
IP Address:
    12.23.34.4/24

Remote:
    12.23.34.1
```

Test:

```bash
execute ping 12.23.34.1
```

---

# 23. FGT-4 — BGP

Configure:

```text
Router ID:
    4.4.4.4

Local AS:
    65004

Network:
    192.168.104.0/24
```

Neighbor:

```text
Neighbor:
    12.23.34.1

Remote AS:
    65001

Next-Hop-Self:
    Enable
```

Logical topology:

```text
                         AS 65001
                           FGT-1
                          /     \
                         /       \
                    AS 65002    AS 65004
                     FGT-2       FGT-4
                                  |
                                 FGT-3
```

---

# 24. FGT-3 — Transit Policies

Because FGT-4 is behind FGT-3, FGT-3 must correctly forward traffic.

## FGT-4 → ISP

```text
Source:
    FGT-4 / LAN

Destination:
    ISP

NAT:
    Enable
```

## ISP → FGT-4

For lab testing and troubleshooting:

```text
Source:
    ISP

Destination:
    FGT-4

NAT:
    Disable
```

> Production NAT behavior depends on the actual network design.

---

# 25. Important Design Point — Dial-Up IPsec + BGP

Do not automatically treat a dial-up IPsec design as identical to a static site-to-site VPN.

For scalable production deployments, evaluate architectures such as:

```text
              HUB
               |
        +------+------+
        |             |
      SPOKE          SPOKE
        |             |
      BGP            BGP
        \             /
         \           /
          ADVPN / IPsec
```

or:

```text
Site-to-Site IPsec
        +
       BGP
```

Fortinet provides specific Dynamic IPsec + BGP designs where dynamic tunnels, route control, and BGP work together. ([Fortinet Docs][1])

---

# 26. Troubleshooting — IPsec

Check Phase 1:

```bash
diagnose vpn ike gateway list
```

Check Phase 2 and tunnel status:

```bash
diagnose vpn tunnel list
```

Check:

```text
Phase 1
Phase 2
XAuth
PSK
Peer ID
Proposal
DH Group
Selectors
Tunnel Interface
```

---

# 27. Troubleshooting — BGP

Check BGP summary:

```bash
get router info bgp summary
```

Check neighbors:

```bash
get router info bgp neighbors
```

Check routing table:

```bash
get router info routing-table all
```

Verify:

```text
Neighbor State
Remote AS
Prefixes Received
Prefixes Advertised
Next-Hop
Routing Table
```

---

# 28. Troubleshooting — Connectivity

Start with the tunnel IPs:

```bash
execute ping 12.23.34.1
execute ping 12.23.34.2
execute ping 12.23.34.4
```

Then verify the BGP neighbor.

Then verify the advertised networks.

Then verify application traffic.

---

# 29. Troubleshooting Flow

Do not start troubleshooting BGP immediately.

Follow the dependency chain:

```text
                 IPsec DOWN?
                      |
                      v
              Check Phase 1
                      |
                      v
           Check XAuth / PSK / ID
                      |
                      v
              Check Phase 2
                      |
                      v
          Check Tunnel Interface
                      |
                      v
             Check IP Reachability
                      |
                      v
               Check Routing
                      |
                      v
          Check BGP Neighbor Reachability
                      |
                      v
             Check BGP Session
                      |
                      v
             Check Firewall Policy
                      |
                      v
              Check Return Path
```

---

# 30. NSE 7 Key Takeaways

## IPsec UP ≠ BGP UP

A successful IPsec negotiation does not automatically mean that BGP will establish.

```text
IPsec
  ≠
BGP
```

BGP still requires IP connectivity between the configured neighbors.

---

## BGP UP ≠ Application Connectivity

Application connectivity depends on:

```text
IPsec
  +
Tunnel Interface
  +
Routing
  +
BGP
  +
Firewall Policy
  +
Return Path
```

---

## XAuth Authentication

The authentication chain can be:

```text
Dial-Up Client
      |
    XAuth
      |
   FGT-1
      |
    LDAP
      |
Active Directory
```

---

## Add Route

Simple dynamic tunnel:

```text
Add Route:
    Enable
```

Multiple dynamic peers / dynamic routing design:

```text
Add Route:
    Disable

BGP / Dynamic Routing:
    Handle route selection
```

Fortinet documents `add-route` as a dynamic IPsec route-control option and shows it disabled in Dynamic IPsec designs where multiple tunnels may advertise the same network. ([Fortinet Docs][1])

---

## Transit Device

When the remote FortiGate is behind another FortiGate:

```text
FGT-1
  |
Transit
  |
FGT-4
```

The transit device must provide:

```text
Routing
+
Firewall Policy
+
Correct NAT Behavior
+
Return Path
```

---

# 31. Lab Security Warning

These parameters are intentionally included for lab/NSE 7 study:

```text
IKEv1
Aggressive Mode
DES
MD5
DH Group 5
```

Do not copy these cryptographic parameters into production.

Use current Fortinet-supported security settings appropriate for your FortiOS version.

---

# 32. Quick Reference

| Device | Role     | Router ID | Local AS | Tunnel IP  | Advertised Network |
| ------ | -------- | --------- | -------: | ---------- | ------------------ |
| FGT-1  | HQ / Hub | 1.1.1.1   |    65001 | 12.23.34.1 | 192.168.101.0/24   |
| FGT-2  | Branch   | 2.2.2.2   |    65002 | 12.23.34.2 | 192.168.102.0/24   |
| FGT-4  | Branch   | 4.4.4.4   |    65004 | 12.23.34.4 | 192.168.104.0/24   |

---

# 33. Useful Commands

### IPsec

```bash
diagnose vpn ike gateway list
diagnose vpn tunnel list
```

### BGP

```bash
get router info bgp summary
get router info bgp neighbors
```

### Routing

```bash
get router info routing-table all
```

### Ping

```bash
execute ping <destination>
```

---

# 34. Final Mental Model

```text
Authentication
      ↓
IPsec Phase 1
      ↓
IPsec Phase 2
      ↓
Tunnel Interface
      ↓
IP Connectivity
      ↓
Routing
      ↓
BGP
      ↓
Firewall Policy
      ↓
Return Path
      ↓
Application
```

> **If you understand this chain, you can troubleshoot the problem instead of simply checking whether the VPN is green.**

```

این نسخه برای GitHub آماده است و از نظر ساختار هم با سری **NSE7 Field Notes / Cheat Sheets** که داری می‌سازی، خیلی خوب قابل ادامه دادن است.
```

[1]: https://docs.fortinet.com/document/fortigate/7.0.0/administration-guide/523447?utm_source=chatgpt.com "Configure dial-up (dynamic) VPN | FortiGate / FortiOS 7.0.0 | Fortinet Document Library"
[2]: https://fortinetweb.s3.amazonaws.com/docs.fortinet.com/v2/attachments/889f7529-ffdc-11e8-b86b-00505692583a/fortios-handbook-60.pdf?utm_source=chatgpt.com "FortiOS Handbook"

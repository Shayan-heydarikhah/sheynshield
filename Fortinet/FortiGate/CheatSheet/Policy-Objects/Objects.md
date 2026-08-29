# FortiGate Objects & Address  

> **FortiGate Address Objects, Address Groups, MAC-Based Policies, Dynamic Addresses, FSSO, Wildcard FQDN & Internet Service Database (ISDB)**
> FortiOS practical reference for NSE-level configuration, troubleshooting, and policy design.

---

## 📌 Quick Map

```text
Firewall Address Objects
│
├── IPv4 / IPv6 Address
├── Address Group
│   └── Exclude Member
│
├── MAC Address
│
├── Dynamic Address
│   ├── Fabric Device
│   ├── FSSO
│   └── FortiManager / Connector
│
├── FQDN
│   └── Wildcard FQDN
│
├── Internet Service Database (ISDB)
│   ├── Predefined
│   ├── Custom
│   ├── Extension
│   └── Addition
│
└── Address Folder / Array Structure
```

---

# 1. Address Groups

Address Groups allow multiple address objects to be referenced as a single policy object.

```text
Policy
  |
  +── Source
  |     └── Address Group
  |            ├── PC-01
  |            ├── PC-02
  |            └── Server-01
  |
  └── Destination
```

---

## 🚫 Address Group Exclusion

An address group can use an **exclude member** to create a logical exclusion.

Conceptually:

```text
GROUP = ALL
EXCLUDE = 192.168.10.50
```

Result:

```text
GROUP
 ├── 192.168.10.1
 ├── 192.168.10.2
 ├── ...
 ├── 192.168.10.49
 ├── ❌ 192.168.10.50
 └── ...
```

### Important

The exclusion mechanism is intended for address ranges/subnets supported by the address-group configuration.

Think of it as:

```text
Included Objects
       +
Excluded Objects
       ↓
Effective Address Set
```

> 💡 **Design Tip:** Exclusion is useful when a large address set must be reused while a small number of addresses need to be carved out.

---

# 2. MAC Address-Based Policies

FortiGate can use MAC-based address objects for policy matching.

### NAT Mode VDOM

In a NAT-mode VDOM, MAC address objects are used in the:

```text
Source
```

side of policies.

```text
Client MAC
    ↓
Source MAC Address Object
    ↓
Firewall Policy
```

### Transparent Mode VDOM

In transparent mode, MAC address objects can be used on both:

```text
Source
Destination
```

sides.

| VDOM Mode        | Source | Destination |
| ---------------- | :----: | :---------: |
| NAT Mode         |    ✅   |      ❌      |
| Transparent Mode |    ✅   |      ✅      |

---

# 3. Vendor MAC Database

FortiGate maintains a vendor MAC database that can be queried from CLI.

### Show Vendor MAC Information

```bash
diagnose vendor-mac id
```

Example:

```bash
diagnose vendor-mac id 36
```

This can be used to identify the vendor associated with a MAC identifier.

For example, a vendor entry may be associated with VMware virtual NICs.

---

## Match a MAC Address

```bash
diagnose vendor-mac match 00:01:02:03:04:05:06
```

A mask can also be supplied:

```bash
diagnose vendor-mac match 00:01:02:03:04:05:06 48
```

The final value specifies how many bits are considered during the matching operation.

```text
MAC Address
  ↓
48-bit MAC
  ↓
Matching Mask
  ↓
Vendor / Database Match
```

---

# 4. Dynamic Address Objects

Dynamic addresses allow FortiGate policies to reference identities or devices that are learned dynamically instead of manually maintaining IP lists.

Common sources include:

```text
Security Fabric
FSSO
FortiManager
FortiClient EMS
FortiAnalyzer
FortiMail
FortiAP
FortiSwitch
```

---

# 5. `fabric_device`

FortiGate provides a default firewall address object:

```text
fabric_device
```

This object represents devices participating in the Security Fabric.

It can be used in several policy types.

### Supported

```text
Firewall Policy
 ├── Normal firewall policies
 ├── Virtual Wire Pair policies
 ├── NAT46
 └── NAT64

IPv4 Shaping Policy

IPv4 ACL Policy
```

### Not Supported

```text
IPv4 Explicit Proxy Policy
```

---

## `fabric_device` Limitations

The `fabric_device` object cannot be combined with:

```text
Custom Extension on Internet Service
```

or:

```text
Address Group Exclusion
```

---

# 6. Where `fabric_device` Gets Its IPs

The dynamic `fabric_device` object can be populated from Security Fabric components.

Typical sources include:

```text
FortiAnalyzer
FortiManager
FortiMail
FortiClient EMS
FortiAP
FortiSwitch
```

For example:

```text
Security Fabric
      |
      +── FortiManager IP
      +── FortiAnalyzer IP
      +── FortiMail IP
      +── EMS IP
      +── FortiAP IPs
      └── FortiSwitch IPs
                |
                v
        fabric_device
                |
                v
          Firewall Policy
```

---

## 🔎 Verify Security Fabric Addresses

```bash
diagnose firewall sf-address list
```

> `sf-address` = **Security Fabric Address**

---

# 7. FSSO Dynamic Addresses

**Fortinet Single Sign-On (FSSO)** allows FortiGate to associate users with IP addresses.

Instead of creating policies around individual users manually:

```text
User
IP
Policy
```

you can use dynamic FSSO groups.

---

## Typical Workflow

```text
Active Directory
      |
      v
FSSO / Collector
      |
      v
FortiGate
      |
      v
Dynamic FSSO Address
      |
      v
Firewall Policy
```

### Configuration Concept

1. Configure FSSO under the Security Fabric / authentication architecture.
2. Verify connectivity with the directory environment.
3. Create an FSSO address/group object.
4. Select the required AD group.
5. Use that object in firewall policies.

Example:

```text
AD Group
   ↓
FSSO Group
   ↓
Dynamic Address
   ↓
Firewall Policy
```

---

# 8. FSSO Logout Behavior

A dynamic FSSO mapping is not permanent.

When a user logs out and the authentication infrastructure reports the logout event:

```text
User Logout
    ↓
FSSO receives logout information
    ↓
User/IP mapping removed
    ↓
Dynamic address updated
    ↓
Firewall policy no longer matches that user
```

This is important because access is based on the **current identity-to-IP mapping**, not simply the original login event.

---

## 🔎 FSSO Troubleshooting

```bash
diagnose debug authd fsso list
```

Use this to inspect the current FSSO user information.

---

# 9. REST API Admin + Dynamic Object Synchronization

A REST API administrator can be created when an external system needs API-based access to FortiGate.

Navigate to:

```text
System
 └── Administrators
      └── Create New
           └── REST API Admin
```

Generate the API credentials and integrate them with the required management/identity platform.

This mechanism can be used when external systems need to synchronize dynamic information with FortiGate.

---

# 10. Dynamic Address Troubleshooting

Useful command:

```bash
diagnose firewall dynamic list address
```

This helps inspect dynamically populated address information.

---

# 11. FortiManager / Connector-Based Dynamic Objects

Dynamic objects can also be populated through external connectors and Security Fabric integrations.

Example concept:

```text
External Directory / Connector
          |
          v
     FortiManager
          |
          v
     FortiGate
          |
          v
 Dynamic Address Object
```

Example CLI pattern:

```bash
config user adgroup
    edit "obj-test"
        set connector-source itgroup
    next
end
```

After the objects are learned/synchronized, they can be referenced by dynamic address objects and policies.

---

# 12. Wildcard FQDN

Wildcard FQDN objects allow a single address object to represent multiple subdomains.

Example:

```text
*.fortinet.com
```

can conceptually match:

```text
www.fortinet.com
support.fortinet.com
docs.fortinet.com
login.fortinet.com
```

---

## Create a Wildcard FQDN Object

```bash
config firewall address
    edit "block-forti"
        set fqdn "*.fortinet.com"
        set cache-ttl 3600
    next
end
```

---

# 13. Wildcard FQDN Cache

FortiGate maintains DNS-derived information for FQDN objects.

Conceptually:

```text
DNS Query
    ↓
Domain → IP
    ↓
FortiGate FQDN Cache
    ↓
Address Object
    ↓
Firewall Policy
```

The cache has a finite capacity, and DNS-derived entries are maintained according to FortiOS FQDN processing behavior.

> ⚠️ **Important:** Do not confuse the FQDN object's `cache-ttl` with the capacity of the FQDN cache itself.

---

# 14. DNS / FQDN Troubleshooting

Check session-helper configuration:

```bash
config system session-helper
    show
end
```

Verify that the relevant DNS-related helper behavior is present where applicable.

---

## Show FQDN Database

```bash
diagnose firewall fqdn list-all
```

This is useful for viewing learned domain-to-IP mappings.

Query information for a specific domain:

```bash
diagnose firewall fqdn getinfo-ip test.com
```

Conceptually:

```text
FQDN
 ↓
DNS Resolution
 ↓
IP Address
 ↓
FQDN Database
 ↓
Firewall Address Object
```

---

# 15. Address Folder / Array Structure

When many address objects belong to the same logical entity, organizing them into folders/structured groups improves administration.

Example:

```text
Applications/
├── Web/
│   ├── WEB-01
│   ├── WEB-02
│   └── WEB-03
│
├── Database/
│   ├── DB-01
│   └── DB-02
│
└── Monitoring/
    ├── NMS-01
    └── NMS-02
```

### Why?

Without logical organization:

```text
100s of objects
       ↓
Hard to find
       ↓
Policy mistakes
       ↓
Operational risk
```

With structured organization:

```text
Objects
  ↓
Folders / Groups
  ↓
Logical structure
  ↓
Easier administration
```

---

# 16. Internet Service Database — ISDB

**Internet Service Database (ISDB)** is designed to identify known Internet services rather than relying only on manually maintained destination IP addresses.

Instead of:

```text
Destination IP
+
Port
```

FortiGate can identify an Internet service using multiple attributes.

Conceptually:

```text
Internet Service
       |
       +── IP Addresses
       +── Domains
       +── Ports
       +── Protocols
       +── CDN information
       +── SNI / domain information
       └── ASN information
```

This makes ISDB particularly useful for Internet-facing security policies.

---

# 17. Why ISDB?

Internet services frequently use:

* Multiple IP addresses
* CDN infrastructure
* Dynamic IPs
* Multiple domains
* Multiple ASNs
* Multiple ports

A static IP address object becomes difficult to maintain.

```text
Traditional
───────────
Service
  ↓
IP list
  ↓
Manual maintenance

ISDB
────
Service identity
  ↓
FortiGuard-maintained database
  ↓
Dynamic service matching
```

---

# 18. ISDB and FortiGuard

ISDB relies on Fortinet's Internet-service intelligence/database.

Depending on the specific ISDB feature and FortiOS version, updated service intelligence may require appropriate FortiGuard services/licensing.

---

# 19. ISDB Types

Common ISDB customization mechanisms include:

```text
Predefined
Custom
Extension
Addition
```

---

# 20. Predefined ISDB

Predefined Internet services are already known by FortiGate.

Example concept:

```text
Google
Microsoft
Amazon
Facebook
DNS
Cloud Services
```

Use them directly in policies when available.

---

# 21. Custom Internet Service

A custom Internet service allows administrators to define their own matching criteria.

Conceptual configuration:

```bash
config firewall internet-service-custom
    edit "x-isdb"
        set reputation 5

        config entry
            edit 1
                set protocol 6

                config port-range
                    edit 1
                        set start-port 443
                        set end-port 443
                    next
                end
            next
        end
    next
end
```

> **Verify the exact CLI syntax and available fields against the target FortiOS release before deployment.**

---

# 22. ISDB Reputation

Internet services can have reputation scores.

A useful conceptual scale is:

| Score | Meaning                                    |
| ----: | ------------------------------------------ |
|     1 | Known malicious / botnet / phishing        |
|     2 | High-risk services such as Tor, proxy, P2P |
|     3 | Unverified / unknown                       |
|     4 | Reputable services / social platforms      |
|     5 | Known verified safe services               |

If a policy uses:

```text
Minimum reputation = 3
```

the effective accepted range is conceptually:

```text
3 + 4 + 5
```

while lower reputation categories are excluded.

---

# 23. Protocol Matching

Internet-service entries can match protocols.

Examples:

```text
TCP = 6
UDP = 17
ICMP = 1
```

Example:

```bash
set protocol 6
```

means TCP.

Port ranges can then be associated with the entry.

---

# 24. ISDB Extension

An **Internet Service Extension** allows additional matching criteria to be attached to an existing Internet service.

Concept:

```text
Existing ISDB
     +
Custom IP / Port / Protocol
     ↓
Extended Service Definition
```

Example structure:

```bash
config firewall internet-service-extension
    edit 65646
        config entry
            edit 1
                set protocol 6

                config port-range
                    edit 1
                        set start-port 80
                        set end-port 443
                    next
                end
            next
        end
    next
end
```

---

# 25. ISDB Disable / Exclusion Entry

Sometimes an Internet service contains a large IP set but a specific IP must not match.

Use a disable/exclusion entry where supported.

Concept:

```text
Large ISDB
 ├── IP-1
 ├── IP-2
 ├── IP-3
 ├── ❌ Excluded IP
 └── IP-N
```

Example pattern:

```bash
config firewall internet-service-extension
    edit 65646

        config disable-entry
            edit 1
                set protocol 17

                config ip-range
                    edit 1
                        set start-ip 142.250.191.165
                        set end-ip 142.250.191.165
                    next
                end
            next
        end

    next
end
```

---

# 26. ISDB Addition

An addition can append custom matching information to an existing Internet service.

For example, add a custom TCP port range:

```text
Existing ISDB
      +
TCP/8080-8090
      ↓
Extended ISDB behavior
```

Conceptual configuration:

```bash
config firewall internet-service-addition
    edit 65646

        config entry
            edit 1
                set protocol 6

                config port-range
                    edit 1
                        set start-port 8080
                        set end-port 8090
                    next
                end
            next
        end

    next
end
```

---

# 27. ISDB Port Append

Another use case is appending a port to a matching service.

Concept:

```text
Match Port:   80
Append Port:  100
```

This can extend matching behavior without rebuilding the complete Internet-service definition.

---

# 28. Refresh ISDB Changes

After changing Internet-service customization, refresh the Internet-service database where required:

```bash
execute internet-service refresh
```

Depending on the change and FortiOS version, a reboot may also cause the updated configuration to become effective.

---

# 29. ISDB Troubleshooting Commands

### Show ISDB Summary

```bash
diagnose internet-services id-summary 11400
```

### Find Internet Service Owner

```bash
get firewall internet-service-owner 2
```

### Query an Internet Service

```bash
diagnose internet-service info root 6 80 8.8.8.8
```

Conceptually:

```text
VDOM
 ↓
Protocol
 ↓
Port
 ↓
Destination IP
 ↓
ISDB Matching
```

---

# 30. ISDB Reputation Direction

Reputation matching must be considered according to traffic direction.

Example:

```bash
config firewall policy
    edit 1
        set reputation-direction source
    next
end
```

---

## LAN → WAN

For outbound traffic:

```text
LAN → WAN
```

the Internet service is normally evaluated as the **destination** service.

Concept:

```text
Client
  |
  | Request
  v
Internet Service
```

Therefore:

```text
reputation-direction destination
```

is the logical direction to consider for outbound Internet-service reputation matching.

---

## WAN → LAN

For inbound traffic:

```text
WAN → LAN
```

the relevant Internet-service identity can be evaluated against the **source**.

Concept:

```text
Internet Service
  |
  | Request
  v
Internal Client
```

Therefore:

```text
reputation-direction source
```

is appropriate for this direction when the policy design requires source-side reputation matching.

---

# 31. Reputation Direction  Table

| Traffic Direction | Typical Reputation Direction |
| ----------------- | ---------------------------- |
| LAN → WAN         | `destination`                |
| WAN → LAN         | `source`                     |

Example:

```text
LAN ───────────────> WAN
      destination
      reputation
```

```text
WAN ───────────────> LAN
 source reputation
```

---

# 32. ISDB + Custom Groups

When custom Internet services are created, they can be combined into address/service groups depending on the supported policy mechanism.

Concept:

```text
Custom ISDB-1
      +
Custom ISDB-2
      +
Custom ISDB-3
      ↓
Logical Service Group
      ↓
Firewall Policy
```

This is useful when multiple Internet services must be treated as one security category.

---

# 33. GeoIP / IPv6 Caveat

Be careful when moving these address-matching concepts to IPv6.

Some GeoIP-related customization features available for IPv4 are not necessarily available for IPv6.

For example, IPv6 does not provide the same support for:

```text
GeoIP Anycast customization
GeoIP override objects
```

as the corresponding IPv4 functionality.

> ⚠️ **Always verify the exact FortiOS release and IPv6 support matrix before designing a production policy around these features.**

---

# 34. Address Object Decision Tree

```text
Need to match...
       |
       +── IP/Subnet?
       |      └── IPv4/IPv6 Address Object
       |
       +── Multiple IP objects?
       |      └── Address Group
       |
       +── MAC?
       |      └── MAC Address Object
       |
       +── Logged-in user?
       |      └── FSSO Dynamic Address
       |
       +── Security Fabric device?
       |      └── fabric_device
       |
       +── Domain?
       |      ├── FQDN
       |      └── Wildcard FQDN
       |
       +── Internet service?
       |      └── ISDB
       |
       └── Large structured object collection?
              └── Folder / Group organization
```

---

# 35. Address Object vs ISDB

| Feature                     | Address Object           | ISDB                  |
| --------------------------- | ------------------------ | --------------------- |
| IP matching                 | ✅                        | ✅                     |
| Subnet matching             | ✅                        | Depends on definition |
| FQDN                        | ✅                        | ✅                     |
| Dynamic service identity    | ❌                        | ✅                     |
| Domain/service intelligence | Limited                  | ✅                     |
| Port awareness              | Limited                  | ✅                     |
| Protocol awareness          | Policy/service dependent | ✅                     |
| CDN awareness               | ❌                        | ✅                     |
| ASN intelligence            | ❌                        | ✅                     |
| FortiGuard intelligence     | ❌                        | ✅                     |
| Best use                    | Internal/local resources | Internet services     |

---

# 36. Address Object vs Dynamic Address

| Requirement                   | Best Choice             |
| ----------------------------- | ----------------------- |
| Static server IP              | Address Object          |
| Network subnet                | Address Object          |
| Multiple addresses            | Address Group           |
| Exclude an address from group | Address Group + Exclude |
| Physical device by MAC        | MAC Address             |
| AD user/group identity        | FSSO                    |
| Security Fabric devices       | `fabric_device`         |
| Domain-based access           | FQDN                    |
| Multiple subdomains           | Wildcard FQDN           |
| SaaS / CDN / Internet service | ISDB                    |

---

# 37. Troubleshooting Quick Reference

```bash
# Security Fabric dynamic addresses
diagnose firewall sf-address list

# Dynamic address information
diagnose firewall dynamic list address

# FSSO users
diagnose debug authd fsso list

# Vendor MAC database
diagnose vendor-mac id

# MAC matching
diagnose vendor-mac match <MAC>

# FQDN database
diagnose firewall fqdn list-all

# FQDN → IP lookup
diagnose firewall fqdn getinfo-ip <FQDN>

# ISDB summary
diagnose internet-services id-summary <ID>

# ISDB owner
get firewall internet-service-owner <ID>

# ISDB lookup
diagnose internet-service info <VDOM> <PROTOCOL> <PORT> <IP>

# Refresh Internet Service database
execute internet-service refresh
```

---

# 🧠 High-Value NSE Takeaways

> **Address Object = "WHO/WHAT IP?"**
> **FQDN = "WHICH DOMAIN?"**
> **FSSO = "WHICH USER/GROUP?"**
> **fabric_device = "WHICH SECURITY FABRIC DEVICE?"**
> **ISDB = "WHICH INTERNET SERVICE?"**

### Remember the architecture:

```text
                    OBJECT IDENTIFICATION
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
       IP                 MAC              DOMAIN
        │                  │                  │
 Address Object      MAC Address        FQDN/Wildcard
        │                  │                  │
        └──────────────────┼──────────────────┘
                           │
                     FIREWALL POLICY
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
      USER              DEVICE           INTERNET SERVICE
        │                  │                  │
      FSSO          fabric_device           ISDB
```

---

# ⚡ Exam & Troubleshooting Traps

| Trap                                                    | Correct Concept                                |
| ------------------------------------------------------- | ---------------------------------------------- |
| `fabric_device` works everywhere                        | ❌ It has policy-type limitations               |
| FSSO address is static                                  | ❌ It is dynamically maintained                 |
| FQDN object = permanent IP list                         | ❌ It depends on DNS resolution/cache           |
| ISDB = only IP addresses                                | ❌ It can represent service intelligence        |
| `reputation-direction` is always destination            | ❌ Direction depends on traffic flow            |
| MAC policies behave identically in NAT/transparent VDOM | ❌ Destination-side support differs             |
| Wildcard FQDN = DNS wildcard record                     | ❌ It is a FortiGate address-matching mechanism |
| ISDB customization always applies immediately           | ❌ Refresh/effect timing must be considered     |
| IPv6 has all IPv4 GeoIP features                        | ❌ Feature parity is not complete               |

---

## 🔥 One-Minute Revision

```text
Address Object
    ↓
IP / Subnet

Address Group
    ↓
Multiple Address Objects
    ↓
Optional Exclusion

MAC Address
    ↓
Hardware Identity

FSSO
    ↓
User → IP → Dynamic Address

fabric_device
    ↓
Security Fabric Device → Dynamic Address

FQDN
    ↓
Domain → Resolved IP

Wildcard FQDN
    ↓
*.example.com → Multiple Subdomains

ISDB
    ↓
Internet Service Identity
    ├── IP
    ├── Domain
    ├── Port
    ├── Protocol
    ├── CDN
    ├── ASN
    └── Reputation
```

> **FortiGate policy design principle:** use the most meaningful identity available. For internal resources, prefer structured address objects/groups; for users use FSSO; for Fabric devices use dynamic Fabric addresses; and for dynamic Internet services prefer ISDB over manually maintaining large IP lists.

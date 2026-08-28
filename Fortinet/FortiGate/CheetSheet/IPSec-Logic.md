این نسخه بسیار جامع‌تر، با جزئیات عمیق‌تر از مفاهیم تئوری و دستورات عملی FortiOS تنظیم شده است تا تمام نکات ریز فایل شما (از جمله مکانیزم پیام‌های IKE، تنظیمات XAuth، دیباگ‌های ADVPN و Redundancy) را به‌طور کامل پوشش دهد.

محتوای زیر را مستقیماً کپی و در گیت‌هاب استفاده کنید:

```markdown
# Comprehensive IPsec & ADVPN Cheat Sheet (FortiOS NSE 4 & NSE 7)

This document provides an in-depth guide to IPsec VPN architecture, FortiOS CLI configurations, and Advanced Auto-Discovery VPN (ADVPN) deployments, merging NSE 4 foundational concepts with NSE 7 enterprise routing and security features.

---

## 1. Cryptography Core & Trust Foundations (CIA Triad)

To establish a secure IPsec tunnel, the framework relies on the **CIA Triad**:
* **Confidentiality:** Ensures data is readable only by authorized parties. Achieved via symmetric encryption algorithms (DES, 3DES, AES-128/256).
* **Integrity:** Ensures data has not been altered in transit. Achieved via hashing algorithms (MD5, SHA-1, SHA-256/512). The receiver hashes the data and compares it to the sender's hash.
* **Availability:** Ensures reliable access through fault clearance, redundancy, and DoS protection.

**Authentication (Identity Verification):**
* **Pre-Shared Key (PSK):** A symmetrical string configured on both peers.
* **RSA Signatures:** Asymmetrical authentication using public/private key pairs and Digital Certificates.

**Diffie-Hellman (DH):** 
A mathematical algorithm used to securely exchange cryptographic keys over a public channel.
* **Groups:** DH-1 (768-bit), DH-2 (1024-bit), DH-5 (1536-bit), DH-14 (2048-bit). Higher groups provide stronger Perfect Forward Secrecy (PFS) but consume more CPU.

---

## 2. IPsec Protocols & Encapsulation Modes

**Security Protocols:**
* **AH (Authentication Header - Protocol 51):** Protects the IP header and payload for Integrity and Authentication. **No Encryption**. Rarely used in modern networks.
* **ESP (Encapsulating Security Payload - Protocol 50):** Protects the payload for Confidentiality, Integrity, and Authentication.

**Encapsulation Modes:**
* **Transport Mode:** Uses the original IP header. Typically used for host-to-host or when IPsec is combined with GRE (GRE over IPsec).
  * *Structure:* `[Original IP] | [ESP Header] | [TCP/UDP] | [Data] | [ESP Trailer] | [ESP Auth]`
* **Tunnel Mode:** Encapsulates the entire original packet inside a new external IP header. Used for Site-to-Site VPNs.
  * *Structure:* `[New External IP] | [ESP Header] | [Original IP] | [TCP/UDP] | [Data] | [ESP Trailer] | [ESP Auth]`

---

## 3. IKEv1 vs. IKEv2 Mechanics (Control Plane vs. Data Plane)

### IKEv1 (Internet Key Exchange version 1)
**Phase 1 (ISAKMP Tunnel - Control Plane):** Negotiates security policies to build a secure management tunnel.
* **Main Mode (6 Messages - Slower but Secure):**
  * **MM 1 & 2 (Crypto Negotiation):** Initiator and Responder agree on Encryption, Hash, Auth method, and DH group (Unencrypted).
  * **MM 3 & 4 (Key Exchange):** DH public values and Nonce (anti-replay random string) are exchanged (Unencrypted).
  * **MM 5 & 6 (Authentication):** Peers prove their identity using PSK/Certificates (Encrypted but Unauthenticated until verified).
* **Aggressive Mode (3 Messages - Faster but Less Secure):**
  * Consolidates crypto negotiation, key exchange, and identity into 3 packets. The Initiator's ID is sent in the clear (not recommended unless one peer has a dynamic IP).

**Phase 2 (IPsec Tunnel - Data Plane):** 
* **Quick Mode (3 Messages):** Uses the secure Phase 1 tunnel to negotiate IPsec Security Associations (SAs). Creates two unidirectional SPIs (Security Parameter Indices) for actual data transfer.

### IKEv2 (Next-Gen Key Exchange)
IKEv2 is faster, lighter, and condenses Phase 1 and 2 into 4 primary messages, eliminating Main/Aggressive mode distinctions.
* **Message 1 & 2 (`IKE_SA_INIT`):** Negotiates crypto, DH exchange, and nonces.
* **Message 3 & 4 (`IKE_AUTH`):** Authenticates the previous messages, transmits identities, and establishes the first Phase 2 Child SA.
* **Key Advancements:** 
  * Native NAT-Traversal (NAT-T) and Keepalives.
  * Extensible Authentication Protocol (EAP) for certificate/MFA integration.
  * Built-in DoS protection (Cookies).
  * **Child SAs:** Uses ECDSA (Elliptic Curve Digital Signature Algorithm) to derive multiple stable child connections from one parent SA.

---

## 4. Advanced FortiOS Configurations & XAuth (NSE 7)

### Dead Peer Detection (DPD) & Connection Timers
DPD uses probe messages (similar to ICMP) to detect if a peer is down.
* **On-Idle:** Probes are sent only if no traffic is received for a specified time. Optimal for Dial-up/Remote Access to save hub resources.
* **On-Demand:** Probes are sent immediately when traffic is queued but the peer isn't responding. Optimal for Site-to-Site.

```bash
config vpn ipsec phase1-interface
  edit "Link-1"
    set dpd on-idle
    set dpd-retryinterval 15
    set dpd-retrycount 3
    set ip-delay-interval 200 # Flushes dial-up released addresses faster than default 240s
end
```

### XAuth & Dial-up Deployments
XAuth forces users to authenticate with credentials (LDAP/RADIUS/Local) after Phase 1 but before Phase 2.
* **Server Modes:** 
  * `pap`: FortiGate negotiates directly with the client.
  * `chap`: FortiGate negotiates with a backend server.
  * `auto`: The best mode; mixes PAP/CHAP based on client capability. Uses a single user group container.

### NAT-Traversal (NAT-T)
Forces IPsec to shift from UDP port 500 to UDP port 4500 if a NAT device is detected between peers (prevents ESP packets from being dropped by PAT devices).
* **Modes:** `enable` (dynamic detection) or `force` (always use UDP 4500).

### TCP MSS & Fragmentation Optimization
Overhead from IPsec encapsulation causes packet drops on standard 1500 MTU links.
```bash
# Phase 1 IKEv2 Fragmentation
config vpn ipsec phase1-interface
  edit "Link-1"
    set ike-version 2
    set fragmentation enable
    set fragmentation-mtu 500
end

# TCP MSS Adjustment for Data Forwarding
config firewall policy
  edit 1
    set tcp-mss-sender 1350
    set tcp-mss-receiver 1350
end
```

### IPSec Passive Mode
The FortiGate will never initiate the IKE negotiation; it only responds. *Do not enable on both peers.*
```bash
config vpn ipsec phase1-interface
  edit "Link-1"
    set passive-mode enable
    set passive-tunnel-interface enable
end
```

---

## 5. Auto-Discovery VPN (ADVPN) & Mesh Networking

ADVPN allows spokes (branches) to dynamically establish direct VPN tunnels with other spokes, bypassing the Hub to reduce latency and bottlenecking. Requires dynamic routing (BGP or OSPF) across the overlay.

**Hub & Spoke Phase 1 Configuration:**
```bash
config vpn ipsec phase1-interface
  edit "ADVPN-Hub"
    set network-id 2             # Segments IPsec domains over the same public interface
    set mesh-selector-type enable # Allows dynamic Phase 2 selectors
    set auto-discovery-sender enable
    set auto-discovery-receiver enable
end
```

**UDP Hole Punching for Spoke-to-Spoke:**
When Spokes are behind NAT, the Hub facilitates "Hole Punching" to allow direct communication using IKE informational messages:
1. **SHORTCUT-OFFER:** Hub informs Spoke 1 about Spoke 2's details.
2. **SHORTCUT-QUERY:** Spoke 1 asks Hub for Spoke 2's NAT mappings.
3. **SHORTCUT-REPLY:** Hub provides NAT mappings, allowing Spokes to establish direct UDP 4500 connections.

---

## 6. High Availability & IPsec Redundancy

**IPsec Link Monitoring (Failover):**
Configures a primary IPsec tunnel to monitor a secondary. If Link-1 fails, Link-2 takes over.
```bash
config vpn ipsec phase1-interface
  edit "Link-2"
    set monitor "Link-1"
end
```

**IPsec Interface Aggregation:**
FortiOS allows bundling multiple IPsec virtual interfaces into a single Aggregate interface for load balancing and redundancy (requires "Enable IPsec Interface Mode").

---

## 7. DoS Protection & Access Control

**Embryonic Limits:**
Protects against IKE flooding attacks by limiting half-open IPsec connections.
```bash
config system settings
  set embryonic-limit 50 
end
```

**Local-In Policies for IKE (Port 500/4500):**
Block unwanted automated IKE scanning bots before they hit the VPN daemon.
```bash
config firewall local-in-policy
  edit 1
    set interface "port1"
    set srcadd "all"
    set dstadd "Blocked_Subnets"
    set service "IKE"
    set action deny
    set schedule "always"
end
```

**Quick Crash Detection (QCD):**
Sends specialized tokens to peers. If a peer reboots without terminating the tunnel, QCD allows the remaining peer to immediately tear down the SA without waiting for DPD timeouts.
```bash
config system settings
  set ike-quick-crash-detect enable
end
```

---

## 8. CLI Diagnostics & Troubleshooting

```bash
# General Tunnel and Gateway Information
diagnose vpn ike gateway list     # Shows SA, Auth methods, Peer IPs, and Timers
diagnose vpn tunnel list          # Shows Phase 2 SPIs, MTU, and exact Subnet Selectors
diagnose vpn ike gateway flush    # Forcefully drops and clears all IKE SAs

# Advanced IKE Negotiation Debugging (Vital for Phase 1/2 failures)
diagnose debug application ike -1
diagnose debug enable

# Packet Sniffer for ISAKMP and NAT-T Traffic
diagnose debug flow filter dport 500
diagnose debug flow filter dport 4500
diagnose debug flow trace start 10
diagnose debug enable
```

---

## 9. Cisco IOS Interoperability Reference

When terminating FortiGate Site-to-Site VPNs against Cisco IOS routers. Note: Cisco uses Route-Maps and Crypto Maps for legacy IKEv1, while IKEv2 relies heavily on FlexVPN (Virtual Template/Tunnel interfaces).

**Cisco IKEv1 Configuration Example:**
```text
! Phase 1 (ISAKMP Policy)
crypto isakmp policy 10
 encryption aes 256
 hash sha256
 authentication pre-share
 group 14

crypto isakmp key vpnsecret123 address 10.0.0.2

! Phase 2 (IPsec Transform Set)
crypto ipsec transform-set myset esp-aes 256 esp-sha256-hmac

! ACL for Interesting Traffic (Selectors)
access-list 100 permit ip 192.168.101.0 0.0.0.255 192.168.102.0 0.0.0.255

! Crypto Map Binding
crypto map mymap 10 ipsec-isakmp
 set peer 10.0.0.2
 set transform-set myset
 match address 100

interface GigabitEthernet0/0
 crypto map mymap
```

```

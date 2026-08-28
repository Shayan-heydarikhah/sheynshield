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
    set ip-delay-interval 200  # Flushes dial-up released addresses faster than default 240s
end
```

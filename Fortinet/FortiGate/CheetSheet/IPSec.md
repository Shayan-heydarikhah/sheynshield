# IPsec VPN & ADVPN FortiOS Cheat Sheet (NSE 4 & NSE 7)

This cheat sheet consolidates IPsec VPN concepts, FortiOS configurations, and advanced ADVPN deployments, covering both foundational (NSE 4) and enterprise routing/security (NSE 7) objectives.

## 1. Core IPSec Concepts & Trust Foundations

The foundation of IPsec relies on the **CIA Triad**:
* **Confidentiality:** Ensures data is accessible only to authorized entities. Achieved via encryption algorithms (DES, 3DES, AES).
* **Integrity:** Ensures packets are not altered in transit (detection of manipulation). Achieved via authentication/hashing algorithms (MD5, SHA-160/128/256/512).
* **Availability:** Ensures authorized users have reliable access (fault clearance, acceptable performance, redundancy).

**Authentication Methods (Identity Proof):**
* **Pre-Shared Key (PSK):** Symmetrical unique values matching on both sides.
* **RSA Signatures:** Asymmetrical private/public key pairs (Certificates).

**Diffie-Hellman (DH):** Secure key exchange mechanism ensuring keys match on both ends without transmitting the actual key in plain text (e.g., DH-1: 768-bit, DH-2: 1024-bit, DH-5: 1536-bit, DH-14: 2048-bit).

---

## 2. IPSec Protocols & Modes

* **AH (Authentication Header):** Protects the IP header and payload (Integrity/Authentication only). No encryption.
* **ESP (Encapsulating Security Payload - Protocol 50):** Protects the payload (Confidentiality, Integrity, Authentication).
  * **Transport Mode:** Uses the original IP header. (Structure: `Global IP | ESP Header | TCP | Data | ESP Trailer | ESP Auth`).
  * **Tunnel Mode:** Encapsulates the entire original packet inside a new IP header to hide the true destination. (Structure: `New External IP | ESP Header | Internal IP | TCP | Data | ESP Trailer | ESP Auth`).

---

## 3. IKEv1 vs. IKEv2 (Internet Key Exchange)

### IKEv1 Mechanics
* **Phase 1 (ISAKMP Tunnel - Control Plane):** Negotiates encryption, hashing, DH groups, and authentication to build the management tunnel.
  * **Main Mode (6 Messages):**
    * MM 1-2: Negotiate Crypto Settings (Unencrypted/Unauthenticated).
    * MM 3-4: Secret Key Exchange/DH & Nonce for anti-replay (Unencrypted/Unauthenticated).
    * MM 5-6: Prove Identity via ID/Auth (Encrypted/Unauthenticated).
  * **Aggressive Mode (3 Messages):** Consolidates SA, Key Exchange, and Auth. Faster but less secure (exposes identities).
* **Phase 2 (IPsec Tunnel - Data Plane):**
  * **Quick Mode (3 Messages):** Builds the actual unidirectional data tunnels (IPsec SAs) using the secure ISAKMP tunnel.

### IKEv2 Mechanics
Faster, lighter, and more secure natively. Condenses Phase 1 and Phase 2 into 4 primary messages:
* **IKE_SA_INIT (Request/Response):** Merges crypto negotiation and key exchange (equivalent to MM 1-4).
* **IKE_AUTH (Request/Response):** Proves identity and establishes the first Phase 2 Child SA (equivalent to MM 5-6 + Phase 2).
* **Key Features:** Built-in NAT-T, EAP support (certificate-based auth), lower bandwidth consumption, native Keepalive timers, and Child SAs via ECDSA (Elliptic Curve Digital Signature Algorithm).

---

## 4. Advanced FortiOS Configurations (NSE 7)

**Dead Peer Detection (DPD) & Keepalives**
Detects if the remote peer is alive to tear down stale SAs. DPD Phase 1 default is typically 15 seconds.
* **On-Idle:** Checks aliveness only if no traffic is received. Ideal for Dial-up/Remote Access to save resources.
* **On-Demand:** Tears down connections immediately if traffic stops. Ideal for Site-to-Site.

```bash
config vpn ipsec phase1-interface
  edit "Link-1"
    set dpd-retryinterval 15
    set dpd-retrycount 3
end
```

**IPSec Passive Mode**
The device will not initiate the VPN connection, even if traffic dictates it. It strictly waits for incoming IKE requests. (Warning: Do not enable on both sides).

```bash
config vpn ipsec phase1-interface
  edit "Link-1"
    set rekey enable
    set passive-mode enable
    set passive-tunnel-interface enable
end
```

**Quick Crash Detection (QCD)**
Allows faster recovery when a peer reboots without sending delete payloads. Avoids waiting for DPD timeouts. (Uses `invalid_ike_spi` tokens in IKEv2).

```bash
config system settings
  set ike-quick-crash-detect enable
end
```

**Fragmentation & MTU Tuning**
Handles large UDP 500/4500 packets over ISP links.

```bash
config vpn ipsec phase1-interface
  edit "Link-1"
    set fragmentation enable
    set fragmentation-mtu 500  # Highly useful for IKEv2
end

# TCP MSS Adjustment for Data Plane
config firewall policy
  edit 1
    set tcp-mss-sender 1350
    set tcp-mss-receiver 1350
end
```

**Dos Protection (Embryonic Limits)**
Prevents CPU exhaustion from IKE floods.

```bash
config system settings
  set embryonic-limit 50 
end

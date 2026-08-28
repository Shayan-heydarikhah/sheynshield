# IPSec VPN Cheat Sheet (NSE 4 & NSE 7)

## 1. Core Security Concepts (CIA Triad)

* **Confidentiality:** Ensures information is accessible only to authorized entities (Protection from unauthorized access).
* **Integrity:** Safeguards the accuracy and completeness of information. Detects alterations during storage or transit.
* **Availability:** Ensures authorized users have access to information when required (Acceptable performance, fault clearance, redundancy).

### Cryptographic Building Blocks
| Component | Function | Algorithms |
| :--- | :--- | :--- |
| **Encryption** | Provides Confidentiality | DES, 3DES, AES |
| **Authentication/Hash** | Provides Integrity | MD5, SHA-128, SHA-160, SHA-256, SHA-512 |
| **Peer Authentication** | Validates Identity | Pre-Shared Key (PSK), RSA (Public/Private Key) |
| **Diffie-Hellman (DH)** | Secure Key Exchange | DH-1 (768-bit), DH-2 (1024-bit), DH-5 (1536-bit) |

---

## 2. IPSec Protocols & Packet Encapsulation

**AH (Authentication Header - Protocol 51)**
Protects headers only. Does not encrypt data.
* **Transport Mode:** Global IP -> AH Header -> TCP -> Data
* **Tunnel Mode:** New External IP -> AH Header -> Internal IP -> TCP -> Data

**ESP (Encapsulating Security Payload - Protocol 50)**
Protects whole data, port, and payload via encryption and authentication.
* **Transport Mode:** Global IP -> ESP Header -> **[TCP -> Data -> ESP Trailer] (Encrypted)** -> ESP Auth
* **Tunnel Mode:** New External IP -> ESP Header -> **[Internal IP -> TCP -> Data -> ESP Trailer] (Encrypted)** -> ESP Auth

---

## 3. IKE (Internet Key Exchange) Framework

### IKEv1 (6 Messages for Main Mode)
Requires negotiated connection profiles per tunnel.

**Phase 1 (ISAKMP Tunnel / Control Plane)**
* **Main Mode:**
  * **MM1 & MM2 (Unencrypted/Unauthenticated):** Negotiate crypto settings (SA, VID, DH Groups).
  * **MM3 & MM4 (Unencrypted/Unauthenticated):** Secret key exchange (Nonce, KE, Anti-replay).
  * **MM5 & MM6 (Encrypted/Unauthenticated):** Prove identity (ID, Auth, Certs, PSK Hash matching).
* **Aggressive Mode:**
  * Faster, uses 3 messages. Merges SA, Key Exchange, and Authentication.
  * AM1 (Initiator), AM2 (Responder behaves like initiator), AM3 (Hash values/Authentication).

**Phase 2 (IPSec Tunnel / Data Plane)**
* **Quick Mode (QM):** Creates unidirectional tunnels. Mixes Phase 1 approved items to forward data. QM1 (Peer), QM2 (Approve SA), QM3 (Tunnel creation).

### IKEv2 (4 Messages)
Faster, lighter, and more secure than IKEv1. Uses proposal repositories.

**Phase 1 (ISAKMP Tunnel)**
* **Message 1 & 2 (IKE_SA_INIT):** Merges crypto negotiation and secret key exchange (Unencrypted/Unauthenticated). Sends SA, KE, Nonce, VID.

**Phase 2 (IPSec Tunnel)**
* **Message 3 & 4 (IKE_AUTH):** Proves identity (Encrypted but Unauthenticated). Sends ID, Auth, Cert, SA, TS, NAT, SPI.

**IKEv2 Features:**
* **EAP:** Extensible Authentication Protocol (Authentication with certificates).
* Built-in NAT-Traversal and Keepalive timers.
* Lower bandwidth consumption.
* **Child SAs:** Creates multiple stable child connections from one main connection based on ECDSA signatures.

---

## 4. Advanced Phase 1 Settings & VPN Features (NSE 7)

| Feature | Description |
| :--- | :--- |
| **ADVPN** | Auto Discovery VPN. Combined with SD-WAN to manage Hub-and-Spoke or Spoke-to-Spoke shortcuts dynamically. |
| **Mode Config** | Assigns specific features (IP addresses, services) to dial-up clients. |
| **NAT Traversal (NAT-T)** | Forces devices to use UDP 500/4500. Detects if IP/Ports are altered by hashing them in ISAKMP payloads. Modes: Enable (Recommended) or Force. |
| **Keepalive Frequency** | Works with NAT-T to prevent intermediary NAT devices from dropping long idle connections. |
| **FEC** | Forward Error Correction. Duplicates data to correct transmission errors. |
| **Add Route** | Automatically injects specific routes into the RIB for dial-up peers. |
| **Device Creation** | Generates virtual interfaces (shells) dynamically for dial-up users to track connections. |
| **Exchange Interface IP** | Used in Hub-and-Spoke topologies to allow direct branch-to-branch visibility. |
| **XAuth** | Extended Authentication using User Groups, CHAP, or PAP servers (Auto/CHAP/PAP for dial-up, Client mode for static remote gateways). |

### Dead Peer Detection (DPD)
Uses `r-u-there` messages to check peer availability.
* **On-Idle:** Active mode. If a connection is idle, DP checks trigger. Drops tunnel after a timeout (e.g., 10 mins).
* **On-Demand:** Passive mode. Triggers immediately only when there is outbound traffic but no return traffic. Uses `dpd-retrycount` and `dpd-interval`.

### Network-ID (FortiGate Segmentation)
Allows multiple disparate IPSec tunnels to operate over the same public interface while maintaining strict logical separation.
```bash
config vpn ipsec phase1-interface
  edit "VPN_Phase1"
    set network-id 2
end

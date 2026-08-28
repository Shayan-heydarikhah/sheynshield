# FortiOS IPsec VPN & ADVPN Deployment Checklist

### 1. Cryptography & Authentication (Phase 1)
- [ ] Define Pre-Shared Key (PSK) or RSA Certificates for peer authentication.
- [ ] Select appropriate Encryption algorithms (e.g., AES-128/AES-256).
- [ ] Select appropriate Hashing algorithms (e.g., SHA-256).
- [ ] Choose Diffie-Hellman (DH) Group (e.g., DH-14 or higher for PFS).

### 2. Protocol & Encapsulation Details
- [ ] Set Security Protocol to ESP (Protocol 50).
- [ ] Select Encapsulation Mode (Tunnel Mode for Site-to-Site, Transport for Host/GRE).
- [ ] Choose IKE Version (IKEv2 recommended for native NAT-T, Keepalives, and speed).
- [ ] If using IKEv1, define Main Mode (standard) or Aggressive Mode (dynamic IPs).

### 3. Advanced FortiOS Settings & Timers
- [ ] Configure Dead Peer Detection (DPD) timers (On-Idle for Dial-up, On-Demand for Site-to-Site).
- [ ] Enable NAT-Traversal (NAT-T) if traversing intermediate NAT devices.
- [ ] Enable IKE Fragmentation and set MTU (e.g., 500) to prevent UDP 500/4500 drops.
- [ ] Adjust TCP MSS for the Data Plane in Firewall Policies (e.g., 1350).
- [ ] Enable IPsec Passive Mode on the responder if it should never initiate the tunnel.
- [ ] Configure XAuth for Dial-up client authentication (PAP/CHAP/Auto).

### 4. ADVPN (Auto-Discovery VPN) Parameters
- [ ] Set identical `network-id` on Hub and Spokes traversing the same public interface.
- [ ] Enable `mesh-selector-type` to allow dynamic Phase 2 subnets.
- [ ] Enable `auto-discovery-sender` (Hub/Spoke).
- [ ] Enable `auto-discovery-receiver` (Hub/Spoke).
- [ ] Ensure Dynamic Routing (BGP/OSPF) is correctly configured over the IPsec overlay.

### 5. High Availability, Redundancy & DoS
- [ ] Configure Link Monitoring on the secondary tunnel to track the primary tunnel.
- [ ] Set Embryonic Limits to prevent IKE CPU exhaustion/floods.
- [ ] Apply Local-in Policies to drop unauthorized UDP 500/4500 scanning.
- [ ] Enable Quick Crash Detection (QCD) for faster tunnel recovery.

### 6. Validation & Diagnostics (Post-Deployment)
- [ ] Verify Phase 1: `diagnose vpn ike gateway list`
- [ ] Verify Phase 2: `diagnose vpn tunnel list`
- [ ] Test Spoke-to-Spoke dynamic tunnel creation (ADVPN UDP Hole Punching).
- [ ] Run packet sniffers on ports 500/4500 if tunnels fail to establish.
- [ ] Run IKE debugs (`diagnose debug application ike -1`) for precise error identification.

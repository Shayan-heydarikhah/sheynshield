# FortiGate Interface & L2/L3 Deployment Checklist

A comprehensive validation and operational readiness checklist for enterprise FortiGate interface setups, high-speed transceivers, Layer 2 encapsulation, and advanced switching architectures.

---

## 🛠 1. Basic Interface & Administrative Access
- [ ] **Physical Transceiver Compatibility:** Verified SFP/SFP+ optical modules using `get system interface transceiver`.
- [ ] **Interface Migration Boundary:** Ensured migration procedures involve physical interfaces only (logical/virtual interfaces excluded).
- [ ] **Administrative Access Lockdown:** Enabled only required administrative management flags per interface (`fmg-access`, `ftm`, `security-fabric`, `speedtest`).
- [ ] **Processing Mode Verification:** Confirmed CPU-bound interfaces (Software Switches) vs ASIC-accelerated ports (Hardware Switches) based on throughput targets.

---

## ⚡ 2. High-Speed Links & Forward Error Correction (FEC)
- [ ] **Interface Speed & Media Identification:** Validated media-type specifications (e.g., `cr4` for twinax copper, `sr4`/`lr4` for fiber).
- [ ] **FEC Status Check:**
  - [ ] **1G / 10G / 40G:** Verified FEC is disabled or unapplied.
  - [ ] **25G / 100G:** Verified auto-negotiation or explicit binding of `cl91-rs-fec`.
- [ ] **Physical Layer Validation:** Confirmed link stability and error-free transmission on high-density 40G/100G paths.

---

## 🏷️ 3. IPAM (IP Address Management) Integration
- [ ] **Global Pool Provisioning:** Enabled IPAM status and defined main subnet pool range (`config system ipam`).
- [ ] **Interface Allocation Limits:** Set subnetwork size parameters (`managed-subnetwork-size`) per managed interface.
- [ ] **Subnet Space Verification:** Executed `diagnose sys ipam largest-available-subnet` to verify contiguous block availability.
- [ ] **Allocation Status:** Confirmed active reservations via `diagnose sys ipam reservation-status`.

---

## 🔐 4. External Captive Portal Deployment
- [ ] **Redirect Target Alignment:** Set external captive portal IP on FGT-1 (ingress) pointing explicitly to FGT-2 (`192.168.12.2`).
- [ ] **Upstream Forwarder Exemption:** Exempted FGT-1 transit interface IP (`192.168.12.1`) on FGT-2 to prevent authentication loops.
- [ ] **Transit NAT Prevention:** Disabled Source NAT on FGT-1 for transit network traffic (`192.168.12.0/30`) to retain source IP visibility.
- [ ] **Routing Validation:** Verified default gateways and static routes ensure bidirectional client flow between nodes.

---

## ⚙️ 5. MTU Tuning & 802.1X Port Security
- [ ] **MTU Override:** Configured target interface MTU and TCP MSS parameters to accommodate tunneling overhead:
  ```bash
  set mtu-override enable
  set mtu 1234
  set tcp-mss 1448

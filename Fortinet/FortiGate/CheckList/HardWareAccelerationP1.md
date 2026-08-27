# 📋 FortiGate Hardware Acceleration & ASIC Operational Checklist

A step-by-step production verification checklist for validating **ASIC offloading (NP/CP)**, optimizing **nTurbo**, managing **VLAN MAC conflicts**, and troubleshooting **hardware acceleration bypass** on FortiGate firewalls.

---

## 🛠️ Phase 1: Hardware & Architecture Verification

- [ ] **ASIC Identification:** Run `get hardware status` to identify the system architecture (Discrete NP/CP vs. System-on-a-Chip / SoC).
- [ ] **CP Resource Audit:** Confirm CP capability and engine density:
  - [ ] **CP9 (SoC4):** 16 IPsec Engines
  - [ ] **CP9 XLite (SoC4):** 5 IPsec Engines
  - [ ] **CP9 Lite (SoC3):** 1 IPsec Engine
  - [ ] **Entry-Level Models:** Verify if Content Processors (CP) are completely absent.
- [ ] **Port-to-ASIC Mapping:** Inspect interface bindings using the appropriate NPU command:
  - [ ] `get hardware npu np6 port-list`
  - [ ] `get hardware npu np6xlite port-list`
  - [ ] `get hardware npu np6lite port-list`

---

## ⚙️ Phase 2: nTurbo & IPS Acceleration Tuning

- [ ] **nTurbo Mode Verification:** Ensure `set np-accel-mode basic` is enabled under `config ips global` to create direct ingress-to-egress acceleration paths.
- [ ] **Pattern Matching Offload:** Check if `set cp-accel-mode advance` is configured (only supported on devices with $\ge$ 2 CP8/CP9 processors).
- [ ] **Feature Availability:** Confirm that IPS global acceleration settings exist on the device (absence indicates no discrete IPS hardware acceleration).

---

## ⚠️ Phase 3: Offloading Exception Audit (CPU Fallback Check)

Verify that critical traffic flows are not silently falling back to host CPU processing due to misconfigurations:

- [ ] **Policy Level ASIC Offload:** Ensure `set auto-asic-offload enable` is active on high-throughput firewall policies.
- [ ] **Inspection Mode:** Verify traffic intended for ASIC offload uses **Flow-based** security profiles instead of Proxy-based profiles.
- [ ] **Session Helpers Check:** Identify protocols requiring session helpers (e.g., FTP via `ftp-helper`) that bypass NP offloading.
- [ ] **Interface & DoS Policies:** Confirm no Interface Policies or DoS Policies are bound to ingress/egress interfaces if maximum NPU throughput is required.
- [ ] **Tunnel Interfaces:** Acknowledge that traffic traversing tunneled interfaces (**IPsec, SSL-VPN, GRE, CAPWAP, IP-in-IP**) will be processed by CPU / nTurbo exceptions.

---

## 🔀 Phase 4: Virtual Clustering & VLAN MAC Drop Prevention

> [!WARNING]
> **VLAN MAC Mismatch Threat:**
> On NP6/NP7 architectures using Virtual Clustering, traffic drops occur if the VLAN interface MAC address differs from the underlying physical interface MAC address.

- [ ] **MAC Address Parity Check:** Verify if VLAN interfaces share the physical port MAC address.
- [ ] **ARP Resolution Verification:** Ensure upstream/downstream switches properly learn and map FortiGate MAC addresses via ARP.
- [ ] **Workaround Implementation:** If packet drops are observed on mismatched VLAN interfaces, explicitly disable NP offloading on the specific policy:
  ```text
  config firewall policy
      edit <policy_id>
          set auto-asic-offload disable
      next
  end

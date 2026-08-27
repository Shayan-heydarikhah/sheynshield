# 📋 FortiGate High Availability (HA) Deployment & Operational Checklist

A step-by-step production verification checklist for deploying, maintaining, and troubleshooting FortiGate **FGCP** (FortiGate Clustering Protocol) and **FGSP** (FortiGate Session Life Support Protocol) clusters in enterprise environments.

---

## 🛠 Phase 1: Pre-Deployment Hardware & License Validation

- [ ] **Hardware Parity:** Verify both units are identical models (same port counts, internal disk layouts, and process architecture).
- [ ] **Firmware Consistency:** Ensure both FortiGate units are running the **exact same FortiOS build** before joining the cluster.
- [ ] **License Verification:** Confirm entitlements (FortiGuard services, IPsec VPN client pools, VDOM licenses) are matching on both devices.
- [ ] **Direct Heartbeat Cabling:** Ensure heartbeat interfaces are connected using **direct back-to-back cabling** (or dedicated L2 isolated VLANs) with redundant physical links.

---

## ⚙️ Phase 2: High Availability (FGCP) Core Configuration

- [ ] **Matching Group ID:** Verify `set group-id` is identical on both cluster members (range: 0–1023).
- [ ] **Monitored Interfaces:** Explicitly configure monitored links (`set monitor <interface_list>`) on both units for physical link-failure detection.
- [ ] **Heartbeat Timers:** Tune signaling timers appropriately to balance convergence speed and stability:
  - [ ] `set hb-interval` configured.
  - [ ] `set hb-interval-in-milliseconds` set (e.g., `10ms` – `100ms`).
  - [ ] `set hb-lost-threshold` configured (e.g., `10` – `20` missed beats).
- [ ] **Session Pickup:** Enable stateful connection failover via `set session-pickup enable`.
  - [ ] Enable `set session-pickup-expectation enable` for layer-7 protocols (FTP, SIP).
  - [ ] Enable `set session-pickup-connectionless enable` for UDP/ICMP tracking.

---

## ⚠️ Phase 3: Non-Synchronized Configurations Verification

> [!WARNING]
> **Critical Design Gotcha:**
> The following parameters do **NOT** synchronize between nodes and must be configured manually on **EACH** individual unit!

- [ ] **Device Priority & HA Override:**
  - [ ] Set distinct `priority` values (e.g., Master = `200`, Slave = `100`).
  - [ ] Decide on preemption behavior and configure `set override` **consistently on BOTH peers**.
- [ ] **System Hostnames:** Assign distinct hostnames (e.g., `FGT-PRI-01` and `FGT-SEC-02`) for audit visibility.
- [ ] **Out-of-Band Management (Reserved Mgmt Interface):**
  - [ ] Enable `set ha-mgmt-status enable` and `set ha-direct enable`.
  - [ ] Configure unique static management IPs and static gateway routes under `config ha-mgmt-interfaces`.
  - [ ] Configure `ha-direct` settings under SNMP, Syslog, and FortiAnalyzer profiles.

---

## 🔒 Phase 4: Network Integration & Failover Tuning

- [ ] **GARP & VMAC Parameters:** 
  - [ ] Enable `set gratuitous-arps enable` to accelerate switch MAC table refreshes upon failover.
  - [ ] Tune `set arps` repeat count (recommended: `5`) and `set arps-interval`.
- [ ] **Hardware Storage & Memory Protections:**
  - [ ] Enable `set ssd-failover enable` (if using disk offloading/proxy services).
  - [ ] Configure `set memory-based-failover enable` with appropriate `flip-timeout` values to avoid conserve mode flapping.
- [ ] **Upstream Switch Configurations:** Ensure switch ports connected to cluster data interfaces are configured with proper **PortFast/Edge** settings to avoid Spanning Tree topology change delays during failovers.

---

## 🧪 Phase 5: Post-Deployment Verification & Testing

- [ ] **Cluster Synchronization Check:** Run `diagnose sys ha checksum autoscale-cluster` and verify all configuration checksums match across peers.
- [ ] **HA Status Validation:** Run `get system ha status` to confirm primary/secondary roles and cluster health status.
- [ ] **Virtual MAC Verification:** Inspect VMAC mappings using `diagnose sys ha mac`.
- [ ] **Failover Stress Test (Controlled Environment):**
  - [ ] Test link failure by shutting down a monitored interface.
  - [ ] Verify continuous ping/traffic survival during primary failover.
  - [ ] Verify out-of-band management connectivity remains accessible for both individual nodes during failover.

---

## ⚡ Quick Verification Commands Cheat Sheet

| Task | Command |
| :--- | :--- |
| **Check Cluster State Engine** | `get system ha status` |
| **Verify Sync Checksums** | `diagnose sys ha checksum autoscale-cluster` |
| **Inspect VMAC Table** | `diagnose sys ha mac` |
| **Force Test Failover** | `execute ha failover set 1` |
| **Revert Test Failover** | `execute ha failover unset 1` |
| **Access Slave CLI from Master** | `execute ha manage <member_id> <username>` |

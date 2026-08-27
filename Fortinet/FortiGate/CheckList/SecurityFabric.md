# 📋 FortiGate Security Fabric & Automation Operational Checklist

A step-by-step production verification checklist for deploying, hardening, and troubleshooting **Security Fabric**, **Automation Stitches**, **External Threat Feeds**, **OT/Purdue Architectures**, and **SAML SSO** integrations on FortiGate.

---

## 🛠️ Phase 1: Pre-Fabric & Core Interface Prerequisites

- [ ] **Interface Management Access:** Verify `fabric` access is enabled under `config system interface` on all interconnecting ports.
- [ ] **LLDP Protocol Enabled:** Ensure both `lldp-reception` and `lldp-transmission` are active on interconnecting switches and FortiGate links.
- [ ] **NPU / VDOM Considerations:** Confirm that target VDOMs are running in single-mode or supported VDOM topologies (Fabric object synchronization does NOT run on multi-mode VDOMs).
- [ ] **OOB / Dedicated Reachability:** Ensure stable L3 connectivity or IPsec site-to-site connectivity between downstream FortiGates and the Root device.

---

## ⚙️ Phase 2: Security Fabric & Object Unification Configuration

- [ ] **Root Device Authorization:** Check and approve all pending downstream devices on the Root FortiGate (`diagnose sys csf authorization pending-list`).
- [ ] **Configuration Sync Mode:**
  - [ ] Set `fabric-object-unification` (`default` for global inheritance or `local` for independent management).
  - [ ] Set `configuration-sync local` for dedicated appliances (FAZ, FMG, FortiSandbox, EMS).
- [ ] **Fabric Worker Tuning:** Verify `set fabric-workers` is optimized (Default: `2`, Range: `1-4`).
- [ ] **Silent Approval & Certificate Push (EMS Integration):**
  - [ ] Enable `websocket`, `silent-approval`, and `push-ca-cert` capabilities under `config endpoint-control fctems`.
  - [ ] Verify endpoint ZTNA posture tags are dynamically syncing across all cluster nodes.

---

## 🌐 Phase 3: External Connectors & Dynamic Threat Feeds

- [ ] **File & Entry Constraints Checklist:**
  - [ ] Total entries in the feed file do NOT exceed `131,072`.
  - [ ] Total file size is under the model memory limit (Default: `10 MB`).
  - [ ] Max URL categories count is within the `30` limit.
- [ ] **Syntax & Formatting Rules:**
  - [ ] IPv4 addresses use standard CIDR blocks (e.g., `192.168.254.0/24`) without brackets.
  - [ ] IPv6 URLs explicitly use square brackets (e.g., `http://[2001:db8::1]/list.txt`).
  - [ ] Duplicate entries are purged to reduce CPU compilation overhead.
- [ ] **DMZ / Web Host Access:** Confirm local HTTP servers (e.g., HFS) hosting external threat feeds are reachable over target ports (e.g., `8080`).

---

## 🤖 Phase 4: Automation Stitches & Execution Tuning

- [ ] **Concurrent Limit Allocation:** Set `max-concurrent-stitches` within system capacities (Range: `32-256`, Default: `128`).
- [ ] **Execution Mode Check:** Ensure **sequential execution mode** is used for configuration or `.pkg` signature restores instead of parallel execution.
- [ ] **Stitch Testing:** Manually trigger scheduled automation routines using `diagnose automation test <stitch_name>` to verify action pipelines.

---

## 🔐 Phase 5: SAML SSO & Identity Provider (IdP) Setup

> [!WARNING]
> **SAML Architecture Rule:**
> The **Root FortiGate** or external IdP (Okta / ADFS / FortiAuthenticator) acts as the **Identity Provider (IdP)**. Downstream FortiGates MUST be configured as **Service Providers (SP)**.

- [ ] **Certificate Trust Chain:** Export enterprise CA certificate (`certsrv`) in Base64 format and import it into FortiGate trusted CAs.
- [ ] **Service Provider (SP) Mapping:** Copy exact SP entity IDs and assertion URLs from FortiGate into the IdP relying party trust setup.
- [ ] **Attribute Matching:** Ensure `username` and `userprincipalname` SAML attributes are correctly mapped on FortiAuthenticator/ADFS.
- [ ] **Fallback Access:** Enable `normal` login page fallback alongside SSO to prevent administrator lockout if the SAML server becomes unreachable.

---

## 🏭 Phase 6: Operational Technology (OT) & Purdue Model Compliance

- [ ] **OT Feature Visibility:** Enable OT options under `System > Feature Visibility > OT`.
- [ ] **Purdue Level Segmentation:** Ensure physical/logical rules align with IEC 62443 standard levels:
  - [ ] **Level 0 / 1:** Physical devices, sensors, and PLCs isolated from higher networks.
  - [ ] **Level 2 / 3:** SCADA, HMIs, and MES monitored by FortiGate / FortiSwitch security policies.
  - [ ] **DMZ Buffer Zone:** NGFW, IPS, and MFA gateways separating Level 3 from Level 4 Enterprise networks.

---

## 🧪 Phase 7: Post-Deployment Diagnostics & Health Verification

- [ ] **Fabric Link Status:** Run `diagnose sys csf downstream` and `diagnose sys csf upstream` to confirm cluster topology health.
- [ ] **Dynamic Feed Resolution:** Verify dynamically imported dynamic IP lists using `diagnose firewall dynamic list`.
- [ ] **Daemon Process Checks:**
  - [ ] Check `autod` (Automation): `diagnose test application autod 1`
  - [ ] Check `fcnacd` (FortiClient NAC): `diagnose test application fcnacd 2`
  - [ ] Check `kubed` (Kubernetes Connector): `diagnose debug application kubed -1`
- [ ] **Security Rating Verification:** Manually execute `diagnose report-runner trigger` to confirm Fabric security audit compliance.

---

## ⚡ Quick Reference Commands Table

| Objective | Command |
| :--- | :--- |
| **Verify Downstream Devices** | `diagnose sys csf downstream` |
| **Verify Upstream Root Device** | `diagnose sys csf upstream` |
| **Check Pending Fabric Requests** | `diagnose sys csf authorization pending-list` |
| **Test Automation Stitch** | `diagnose automation test <schedule_name>` |
| **List Dynamic Threat Feed IPs** | `diagnose firewall dynamic list` |
| **Test SAML WAD Auto-Discovery** | `diagnose wad user exchange test-auto-discover` |
| **Check Contract & Entitlements** | `diagnose test update info` |
| **Trigger Security Rating Audit** | `diagnose report-runner trigger` |

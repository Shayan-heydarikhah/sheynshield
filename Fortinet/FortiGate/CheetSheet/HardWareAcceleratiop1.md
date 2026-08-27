# ⚡ FortiGate Hardware Acceleration & ASIC Offloading — Operational Cheat Sheet

A production-ready reference covering **FortiGate System-on-a-Chip (SoC)** architectures, **Content Processors (CP)**, **Network Processors (NP)**, **nTurbo acceleration**, and ASIC offloading limitations.

---

## 🏛️ 1. Architecture Overview & Processor Matrix

FortiGate entry-level appliances typically do not feature discrete Content Processors (CP). Instead, they utilize System-on-a-Chip (SoC) architectures that consolidate the CPU, Network Processor (NP), and Content Processor (CP) onto a single die.

```text
               +-----------------------------------+
               |    System-on-a-Chip (SoC)        |
               |                                   |
               |  +---------+  +-----------------+ |
               |  |   CPU   |  |   SPU (ASIC)    | |
               |  +---------+  | +-----+ +-----+ | |
               |               | | NPU | | CP  | | |
               |               | +-----+ +-----+ | |
               |               +-----------------+ |
               +-----------------------------------+

```

### Content Processor (CP) & IPsec Engine Matrix

| CP Version | Integrated SoC Version | IPsec Acceleration Engines |
| --- | --- | --- |
| **CP9** | SoC4 | **16** IPsec Engines |
| **CP9 XLite** | SoC4 | **5** IPsec Engines |
| **CP9 Lite** | SoC3 | **1** IPsec Engine |

```bash
# Verify system hardware architecture and ASIC version
get hardware status

```

---

## 🚀 2. nTurbo & IPS Offloading Mechanisms

**nTurbo** creates a direct fast-path data channel between ingress and egress interfaces for IPS-inspected traffic, offloading flow-based security profiles to ASIC hardware to bypass standard CPU path latency.

```text
config ips global
    set np-accel-mode basic        # Default: basic (Enables nTurbo offloading)
    set cp-accel-mode advance      # Enables pattern matching offload to CPs
end

```

> 📌 **Note:** `set cp-accel-mode advance` optimizes pattern matching performance and is supported on appliances equipped with two or more **CP8** or **CP9** processors. If these options are missing from the CLI, the hardware does not feature discrete IPS ASIC acceleration.

---

## ⛔ 3. nTurbo Offloading Exceptions (CPU Fallback)

Even when explicitly enabled, sessions automatically fall back to the host CPU under any of the following conditions:

* **ASIC Offload Disabled:** `auto-asic-offload` is manually set to `disable` inside the firewall policy.
* **Proxy Security Profiles:** The matching firewall policy utilizes proxy-based security profiles instead of flow-based profiles.
* **Session Helpers Required:** Traffic depends on FortiOS session helpers (e.g., FTP control/data channels managed by `ftp-helper`).
* **Interface / DoS Policies:** Ingress or egress interfaces have active Interface Policies or DoS Policies applied.
* **Tunnel Interfaces:** Traffic originates from or terminates on tunneled interfaces (**IPsec, IP-in-IP, SSL-VPN, GRE, CAPWAP**).

---

## ⚠️ 4. Virtual Clustering, VLAN MAC Mismatch & Drops

On **NP6** and **NP7** architectures operating in Virtual Clustering modes, traffic drops may occur if the MAC address of a VLAN interface differs from the underlying physical interface MAC.

$$\text{VLAN Interface MAC} \neq \text{Physical Interface MAC} \Longrightarrow \text{Potential NP6/NP7 Frame Drop}$$

### Mitigation & Workaround Rules:

* **Offloading Support Conditions:**
* Downstream/upstream devices correctly resolve the FortiGate MAC address via ARP.
* The destination MAC address of reply traffic matches the MAC of the physical host interface.


* **Resolution Action:** If drops persist, disable NP offloading on the specific firewall policy accepting the VLAN traffic:

```text
config firewall policy
    edit <policy_id>
        set auto-asic-offload disable
    next
end

```

---

## 📊 5. NetFlow vs. sFlow Offloading Impact

* **NetFlow:** Fully supported alongside **NP6**, **NP6XLite**, and **NP6Lite** acceleration. State tracking is maintained via the firewall session table without disrupting hardware offloading.
* **sFlow:** **Disables ALL NP6 / NP6XLite / NP6Lite offloading** for all ingress and egress traffic on any interface where sFlow is enabled.

---

## 🔍 6. Hardware Diagnostic & Verification Commands

```bash
# Display NP6 port mapping and ASIC bindings
get hardware npu np6 port-list
diagnose npu np6 port-list

# Display NP6XLite port mapping
get hardware npu np6xlite port-list

# Display NP6Lite port mapping and status
get hardware npu np6lite port-list
diagnose npu np6lite port-list

```

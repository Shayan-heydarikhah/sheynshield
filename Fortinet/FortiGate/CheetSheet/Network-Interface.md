# FortiGate Architecture & Interface Management Cheat Sheet

This comprehensive reference guide consolidates enterprise FortiGate Layer 2/Layer 3 interface configurations, ASIC hardware optimizations, packet acceleration techniques, and network encapsulation standards.

---

## 📌 Table of Contents
- [Interface Types & Management Options](#-interface-types--management-options)
- [Transceivers & Forward Error Correction (FEC)](#-transceivers--forward-error-correction-fec)
- [IP Address Management (IPAM)](#-ip-address-management-ipam)
- [External Captive Portal Architecture](#-external-captive-portal-architecture)
- [Interface Parameters (MTU & 802.1X)](#-interface-parameters-mtu--8021x)
- [Traffic Sniffing (One-Arm SPAN)](#-traffic-sniffing-one-arm-span)
- [VLANs & Hardware Virtual Switches](#-vlans--hardware-virtual-switches)
- [Link Aggregation (LAG) & Redundancy](#-link-aggregation-lag--redundancy)
- [Virtual Wire Pair (VWP) & PRP](#-virtual-wire-pair-vwp--prp)
- [Enhanced MAC (EMAC) VLAN](#-enhanced-mac-emac-vlan)
- [VXLAN Architecture & Encapsulation](#-vxlan-architecture--encapsulation)

---

## 🛠 Interface Types & Management Options

### Interface Architecture
* **Software Switches:** Traffic is switched in software and processed via the main system CPU.
* **Hardware Switches:** Virtual switches operating at the hardware layer utilizing ASICs and co-processors. Supports optimized Spanning Tree Protocol (STP) processing.
* **Physical Interfaces:** SFP/SFP+ ports are hardware-sensitive and require verified vendor modules. Verify diagnostics via:
  ```bash
  get system interface transceiver

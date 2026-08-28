# FortiGate Interface Deployment Checklist

This operational checklist standardizes FortiGate interface configurations, Layer 2/3 optimizations, and hardware-accelerated features for enterprise rollouts.

---

## Base Interface & Transceiver Verification
- [ ] **Determine switch architecture:** Ensure traffic is processed via CPU for software switches, or utilize ASICs/co-processors for hardware switches to achieve optimized spanning tree processing.
- [ ] **Configure administrative access:** Set `fmg-access` for automatic FortiManager authorization, `ftm` for FortiToken Mobile push access, `security-fabric` for FortiTelemetry/CAPWAP, or `speedtest` for bandwidth measurement.
- [ ] **Validate Forward Error Correction (FEC):** Confirm FEC is active (`cl91-rs-fec`) automatically on 25G and 100G interfaces, and note that it is disabled or unsupported on 1000M, 10G, and 40G links.
- [ ] **Match transceiver specifications:** Use SR4 for short-range multimode up to 100m, LR4 for long-range single-mode up to 10km+, and CR4 for twinaxial copper up to 5m.
- [ ] **Adjust MTU parameters:** Apply `mtu-override enable`, set custom MTU values, and define the `tcp-mss` on the interface level if required by the network topology.

## Layer 2 Switching, VLANs & MAC Management
- [ ] **Deploy Virtual VLAN switches:** Use hardware switch ports on supported models (e.g., 60F, 100F, 200F) as Layer 2 switches to assign 802.1Q VLANs directly to trunk ports.
- [ ] **Select VLAN tagging standard:** Use 802.1Q (EtherType 0x8100) for basic single-tag segmentation, or 802.1ad (QinQ) for double-tagging and metro-ethernet tunneling.
- [ ] **Provision Enhanced MAC (EMAC):** Generate distinct MAC addresses for VLANs on the same physical interface, ensuring the firewall operates in NAT mode.
- [ ] **Restrict EMAC deployment:** Avoid configuring EMAC on management interfaces, HA heartbeat links, and transparent VDOMs, as the VLAN ID and underlying interface must remain a unique pair.
- [ ] **Configure VXLAN encapsulation:** Set the remote IP and VNI, create a VLAN under the VXLAN interface, and bind it to a software switch with the physical client port (never add the main VXLAN interface directly).

## Redundancy, LAG & Virtual Wire Pairs
- [ ] **Align Link Aggregation (LAG):** Configure `lacp-mode static` on the FortiGate to establish aggregated links with standard switch channel-groups.
- [ ] **Optimize LAG hashing:** Set the NPU member selection algorithm to `xor16` for lighter data center loads, or `crc16` for resource-intensive, highly granular distribution.
- [ ] **Configure Virtual Wire Pairs (VWP):** Implement VWP for Direct Server Return (DSR) using completely empty member interfaces that possess no existing references.
- [ ] **Apply VWP NAT routing:** Ensure Overload IP Pool NAT is utilized if NAT is required within the Virtual Wire Pair policies.
- [ ] **Enable Parallel Redundancy Protocol (PRP):** Configure dual independent networks (LAN A and LAN B) for critical infrastructure to ensure zero-time recovery through redundant packet transmission.

## Authentication, IPAM & Diagnostics
- [ ] **Allocate IPAM subnets:** Enable FortiIPAM, define pool subnets, and execute `diagnose sys ipam reservation-status` or `diagnose sys ipam largest-available-subnet` to verify allocations.
- [ ] **Setup external captive portals:** Forward authentication traffic to a secondary FortiGate (FGT-2) acting as the external source.
- [ ] **Secure captive portal routing:** Exempt the forwarding firewall's IP address and completely disable NAT on the transit link to prevent the authentication procedures from failing.
- [ ] **Enforce 802.1X security:** Apply port-level security directly on hardware switch interfaces running the NP6 platform.
- [ ] **Monitor via One-Arm Sniffer:** Use one-arm sniffing strictly for short-term packet captures, acknowledging that it disables hardware acceleration (nTurbo/CP), increases CPU utilization, and impacts the kernel buffer size.

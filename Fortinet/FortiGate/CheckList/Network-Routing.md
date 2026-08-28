# FortiGate Routing Configuration & Diagnostic Checklist

## 1. Route Lookup & Core Diagnostics
- [ ] Verify administrative distances (Static: 10, DHCP: 5).
- [ ] Check active and in-use routes: `get router info routing-table all`
- [ ] Inspect specific route lookups: `get router info routing-table details <IP>`
- [ ] Review FIB and routing daemon info: `get router info kernel`
- [ ] Validate route cache: `diagnose ip rtcache list`
- [ ] Confirm assigned IP addresses: `diagnose ip address list`

## 2. Route Mechanics & Session Preservation
- [ ] Verify RPF (Reverse Path Forwarding) settings (Default vs. Strict mode: `set strict-src-check enable`).
- [ ] Enable asymmetric routing ONLY if required and security risks are accepted (`set asymroute enable`).
- [ ] Enable `preserve-session-route` on interfaces for HA and dynamic routing topology changes.
- [ ] Enable `snat-route-change` globally to force session route re-evaluations on link fail.

## 3. ECMP & Link Redundancy
- [ ] Choose appropriate ECMP algorithm (Source IP, Weighted, Usage, Source-Destination IP, Volume).
- [ ] Set `ecmp-max-path` to the correct number of available redundant links.
- [ ] Configure `link-monitor` for critical upstream gateways.
- [ ] Fine-tune failover thresholds (`interval`, `failtime`, `recovertime`) in link-monitor.
- [ ] Enable `update-static-route` in link-monitor to automatically withdraw routes on failure.
- [ ] Enable `update-cascade-interface` to shut down dependent downstream interfaces on failure.

## 4. Dynamic Routing (RIP & OSPF)
- [ ] **RIP:** Ensure timers match exactly across all peer devices (`timeout-timer`, `update-timer`, `garbage-timer`).
- [ ] **RIP:** Configure Keychain authentication for secure routing updates.
- [ ] **OSPF:** Check neighbor states and adjacencies: `get router info ospf neighbor details`
- [ ] **OSPF:** Review LSDB (Link State Database) info: `get router info ospf database brief`
- [ ] **OSPF:** Enable `graceful-restart` to prevent forwarding interruptions during HA failovers or SPF runs.

## 5. Dynamic Routing (BGP)
- [ ] Enable `ebgp-multipath` and `ibgp-multipath` for active-active load sharing.
- [ ] Enable `recursive-next-hop` to safely resolve nested lookups and prevent routing loops.
- [ ] Verify `update-source` is set correctly when using Loopback interfaces.
- [ ] Enable `soft-reconfiguration` for non-disruptive routing policy updates.
- [ ] Configure Route Flap Dampening to protect CPU and RIB from unstable peer links.
- [ ] Enable BFD (Bidirectional Forwarding Detection) globally and on specific BGP neighbors for fast convergence.

## 6. Virtual Routing and Forwarding (VRF)
- [ ] Configure VDOM links (`vdom-link`) if inter-VRF routing is required.
- [ ] Enable `allow-subnet-overlap` if using identical IP schemes across different VRFs.
- [ ] Set up BGP `route-map` configurations for accurate route leaking between VRF instances.
- [ ] Verify VRF IDs are correctly assigned to all relevant interfaces and blackhole routes.

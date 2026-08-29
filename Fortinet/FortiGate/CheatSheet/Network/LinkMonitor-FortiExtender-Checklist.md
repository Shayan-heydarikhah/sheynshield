# 🔗 FortiGate Link Monitor & FortiExtender Checklist

> **FortiOS 7.2.0 | Link Monitor | Health Check | SD-WAN | Failover | BFD | FortiExtender | LTE | WAN Redundancy**

**SheynShield — Fortinet Network Security Engineering Knowledge Base**

---

## 🎯 What This Checklist Covers

Use this checklist to design, configure, validate, and troubleshoot:

* [ ] FortiGate Link Monitor
* [ ] ICMP / Ping health checks
* [ ] TCP / UDP / HTTP monitoring
* [ ] TWAMP monitoring
* [ ] Static-route updates
* [ ] Policy-route updates
* [ ] Interface failure and cascade behavior
* [ ] Redundant interfaces
* [ ] Weighted server monitoring
* [ ] SD-WAN integration
* [ ] Service detection
* [ ] MOS / SLA monitoring
* [ ] BFD integration
* [ ] HA behavior
* [ ] IPsec / ADVPN monitoring
* [ ] FortiExtender
* [ ] LTE backup WAN
* [ ] FortiExtender + SD-WAN
* [ ] LAN / WAN extension
* [ ] VXLAN over IPsec
* [ ] Out-of-band connectivity
* [ ] LTE failover

---

# 🧠 1. Link Monitor — Design Checklist

Before configuring Link Monitor, answer these questions:

* [ ] What path am I monitoring?
* [ ] What destination/server should be tested?
* [ ] Which source interface should generate the probes?
* [ ] Which gateway should be used?
* [ ] What protocol should perform the health check?
* [ ] What constitutes a failure?
* [ ] What constitutes recovery?
* [ ] Should static routes be updated?
* [ ] Should policy routes be updated?
* [ ] Should interface failure be cascaded?
* [ ] Is the monitor used by SD-WAN?
* [ ] Is HA involved?
* [ ] Is BFD required?
* [ ] Is NAT affecting the probe?
* [ ] Is the return path valid?

### Core Mental Model

```text
                 LINK MONITOR
                      │
                      ▼
                Health Check
                      │
             ┌────────┴────────┐
             ▼                 ▼
          SUCCESS           FAILURE
             │                 │
             ▼                 ▼
          Link UP           Link DOWN
                               │
                 ┌─────────────┼─────────────┐
                 ▼             ▼             ▼
              Routing       Policy       Interface
               Update        Route         Cascade
```

> **Expert rule:** Never troubleshoot Link Monitor as an isolated feature. Always analyze **probe → underlay → routing → policy → return path**.

---

# 🧪 2. Health Check Protocol Checklist

Select the protocol according to what you actually need to prove.

| Protocol    | What It Validates              | Checklist |
| ----------- | ------------------------------ | --------- |
| ICMP / Ping | Basic IP reachability          | [ ]       |
| TCP         | TCP connectivity               | [ ]       |
| UDP         | UDP reachability               | [ ]       |
| HTTP        | Application-level reachability | [ ]       |
| TWAMP       | Performance measurement        | [ ]       |

### Ask:

* [ ] Does ICMP prove enough for this application?
* [ ] Should I monitor a TCP service instead?
* [ ] Does the destination actually respond to the selected protocol?
* [ ] Is the probe being filtered?
* [ ] Is the probe traversing the intended WAN?

> 💡 **Reachability is not always service health.** A host can answer ICMP while the actual application is unavailable.

---

# ⚙️ 3. Basic Link Monitor Configuration Checklist

Example:

```bash
config system link-monitor
    edit 1
        set srcintf agg1
        set server 5.200.200.200
        set gateway-ip 192.168.254.2
        set route 192.168.102.0/24 192.168.103.0/24
        set failtime 2
        set recoverytime 2
        set probe-count 5
    end
end
```

### Validate

* [ ] `srcintf` is correct
* [ ] `server` is reachable through the intended path
* [ ] `gateway-ip` is correct
* [ ] Associated routes are correct
* [ ] Failure threshold is appropriate
* [ ] Recovery threshold is appropriate
* [ ] Probe count is appropriate
* [ ] Return traffic is possible

---

# 🔌 4. Source Interface Checklist

The source interface is one of the most important Link Monitor parameters.

```bash
set srcintf <interface>
```

Verify:

* [ ] The interface belongs to the intended WAN/path
* [ ] The interface has valid connectivity
* [ ] The probe is not accidentally leaving through another path
* [ ] Routing toward the monitored server is correct
* [ ] NAT/firewall behavior is understood

### Troubleshooting Question

> **If the WAN fails, from which interface is FortiGate actually sending the health check?**

If you cannot answer this confidently, stop troubleshooting and verify the source interface first.

---

# 🛣️ 5. Route Update Checklist

Link Monitor can interact with routing behavior.

Check:

* [ ] `update-static-route`
* [ ] `update-policy-route`
* [ ] Associated static routes
* [ ] Policy routes
* [ ] Route removal behavior
* [ ] Route recovery behavior

Example:

```bash
get router info routing-table static
```

### Failure Flow

```text
Health Check Failure
        │
        ▼
Link Monitor DOWN
        │
        ▼
Associated Route Updated
        │
        ▼
Routing Decision Changes
```

> ⚠️ A Link Monitor going down does not mean your entire failover design is correct. Verify the resulting routing table.

---

# 🔀 6. Multiple WAN Link Monitor Checklist

For multiple WAN links:

```text
                FortiGate
                   │
          ┌────────┴────────┐
          ▼                 ▼
        WAN-1             WAN-2
          │                 │
          ▼                 ▼
      Health Check      Health Check
          │                 │
          ▼                 ▼
       Server A           Server B
```

Verify:

* [ ] Each WAN has an appropriate source interface
* [ ] Each WAN has the correct gateway
* [ ] Monitoring destinations are reachable
* [ ] Failure thresholds are consistent
* [ ] Recovery thresholds are consistent
* [ ] Routing reacts correctly
* [ ] SD-WAN behavior is understood
* [ ] NAT behavior is understood

Example:

```bash
config system link-monitor
    edit wan1
        set srcintf port4
        set server 8.8.8.8
        set gateway-ip 12.12.12.2
        set failtime 2
        set recoverytime 2
        set probe-count 5
    end
end
```

---

# 🔗 7. Update Cascade Interface Checklist

`update-cascade-interface` controls whether Link Monitor failure can propagate to other interfaces.

Verify:

* [ ] Do I actually want interface cascade?
* [ ] Is this a redundant-interface design?
* [ ] Could cascade unnecessarily disable a healthy member?
* [ ] Have I tested the failure scenario?
* [ ] Have I tested recovery?

### Cascade Enabled

```text
Health Check
     │
     ▼
   FAIL
     │
     ▼
Interface DOWN
     │
     ▼
Cascade Failure
```

### Cascade Disabled

```text
Link A → ❌ DOWN

Link B → ✅ UP
          │
          ▼
   Continue forwarding
```

Disable when appropriate:

```bash
config system link-monitor
    edit mik-1-red
        set update-cascade-interface disable
    end
end
```

> 🔥 **Design question:** Do you want a failed health check to remove only the failed path, or should it propagate failure to the redundant interface?

---

# ⚖️ 8. Weighted Server Monitoring Checklist

For more granular health evaluation:

```bash
config system link-monitor
    edit mik-1-red
        set server-config individual

        config server-list
            edit 1
                set dst 8.8.8.8
                set protocol ping
                set weight 30
            next

            edit 2
                set dst 5.200.200.200
                set protocol ping
                set weight 30
            next
        end

        set fail-weight 60
    end
end
```

### Validate

* [ ] Server 1 has correct destination
* [ ] Server 2 has correct destination
* [ ] Protocol is correct
* [ ] Weight values are intentional
* [ ] `fail-weight` is correct
* [ ] Failure behavior has been tested

### Example

```text
Server A = 30
Server B = 30

fail-weight = 60
```

If only A fails:

```text
Failed weight = 30

30 < 60

→ Link remains UP
```

If A and B fail:

```text
Failed weight = 60

60 >= 60

→ Link becomes DOWN
```

> 🧠 **Exam memory:** `weight` represents the impact of an individual server failure; `fail-weight` determines when the combined failure is sufficient to declare the monitor failed.

---

# ⏱️ 9. Detection Timer Checklist

Example:

```bash
config system link-monitor
    edit 1
        set interval 500
        set probe-timeout 500
        set failtime 2
        set recoverytime 2
        set probe-count 5
    end
end
```

### Validate

* [ ] `interval`
* [ ] `probe-timeout`
* [ ] `failtime`
* [ ] `recoverytime`
* [ ] `probe-count`

### Mental Model

```text
Probe
  │
  ├── Success
  ├── Success
  │
  ▼
Recovery

Probe
  │
  ├── Failure
  ├── Failure
  │
  ▼
Failure
```

> ⚠️ Avoid unnecessarily aggressive timers unless the network and application requirements justify them.

---

# 🏆 10. HA + Link Monitor Checklist

If FortiGate HA is involved:

* [ ] Is the cluster Active-Passive?
* [ ] Is Link Monitor behavior required on both members?
* [ ] Is `ha-priority` configured intentionally?
* [ ] Is failover behavior understood?
* [ ] Have you tested the failure from the active device?
* [ ] Have you tested recovery?
* [ ] Have you verified local/shared monitoring states?

Example:

```bash
set ha-priority 1
```

Conceptually:

```text
Active
  │
  └── HA Priority 1

Passive
  │
  └── HA Priority 2
```

Possible diagnostic concepts:

```text
local alive
shared dead
```

---

# 📊 11. SD-WAN Integration Checklist

Link Monitor can provide path-health information used by SD-WAN.

Check:

* [ ] Health check is associated with the intended path
* [ ] `class-id` behavior is understood
* [ ] Service detection requirements are understood
* [ ] SD-WAN rules use the expected SLA/path state
* [ ] Failover has been tested

```text
Link Monitor
     │
     ▼
Health Check
     │
     ▼
Path Health
     │
     ▼
SD-WAN Decision
     │
     ▼
Preferred Path
```

---

# 📞 12. Service Detection & MOS Checklist

For application/service-quality monitoring:

* [ ] Service detection requirement identified
* [ ] MOS requirement identified
* [ ] Real-time application requirements identified
* [ ] Latency requirements identified
* [ ] Jitter requirements identified
* [ ] Packet-loss requirements identified

Typical use cases:

* [ ] VoIP
* [ ] Voice
* [ ] Real-time applications
* [ ] Streaming

> 💡 **MOS** is a service-quality metric, not simply an indication that an IP endpoint is reachable.

---

# 🚀 13. BFD + Link Monitor Checklist

If faster failure detection is required:

* [ ] Determine whether BFD is appropriate
* [ ] Verify BFD support on the path
* [ ] Verify BFD session state
* [ ] Compare BFD and Link Monitor timers
* [ ] Test failure detection
* [ ] Test recovery

```text
              BFD
               │
       Fast Failure Detection
               │
               ▼
         Path Availability
               │
               ▼
       Routing / SD-WAN
```

> 🧠 Use BFD when the design requires rapid failure detection; use Link Monitor when you need path/server health beyond simple adjacency detection.

---

# 🔐 14. Link Monitor over IPsec / ADVPN Checklist

When monitoring through an IPsec tunnel:

* [ ] Verify tunnel is established
* [ ] Verify `srcintf`
* [ ] Verify monitored destination
* [ ] Verify routing through the tunnel
* [ ] Verify return path
* [ ] Verify `net-device` behavior
* [ ] Determine whether `server-type dynamic` is required

Example:

```bash
config system link-monitor
    edit 1
        set srcintf advpn-branch
        set server-type dynamic
    end
end
```

Useful for scenarios involving:

* [ ] Dial-up VPN
* [ ] Dynamic VPN endpoints
* [ ] ADVPN

---

# 🔍 15. Link Monitor Diagnostics Checklist

Run:

```bash
diagnose sys link-monitor status
```

* [ ] Verify monitor state
* [ ] Verify server state
* [ ] Verify gateway state
* [ ] Verify failure information

Check tunnel information:

```bash
diagnose sys link-monitor tunnel all
```

* [ ] Verify tunnel state

Check interface:

```bash
diagnose sys link-monitor interface red-1
```

* [ ] Verify interface state

Manually launch:

```bash
diagnose sys link-monitor launch mik-1-red
```

* [ ] Generate a health check
* [ ] Observe probe behavior

Filter:

```bash
diagnose sys link-monitor filter name mik
```

* [ ] Narrow troubleshooting to the required monitor

---

# 🧪 16. Link Monitor Troubleshooting Checklist

When Link Monitor reports failure:

### Step 1 — Configuration

* [ ] Verify `srcintf`
* [ ] Verify server
* [ ] Verify gateway
* [ ] Verify protocol
* [ ] Verify timers

### Step 2 — Underlay

* [ ] Can the source interface reach the destination?
* [ ] Is the gateway reachable?
* [ ] Is the ISP path operational?
* [ ] Is there packet loss?

### Step 3 — Probe

* [ ] Are probes being transmitted?
* [ ] Are replies received?
* [ ] Are sequence numbers progressing?
* [ ] Is RTT reasonable?

### Step 4 — Routing

* [ ] Check routing table
* [ ] Check route removal
* [ ] Check route installation
* [ ] Check policy routes

### Step 5 — Policy

* [ ] Verify firewall policy
* [ ] Verify NAT
* [ ] Verify local-out behavior where applicable

### Step 6 — Failover

* [ ] Verify cascade behavior
* [ ] Verify SD-WAN reaction
* [ ] Verify HA reaction
* [ ] Verify BFD state

### Step 7 — Return Path

* [ ] Verify destination can return traffic
* [ ] Verify asymmetric routing is not breaking the test

---

# 📡 17. FortiExtender Design Checklist

Use FortiExtender when you need additional connectivity such as:

* [ ] Backup WAN
* [ ] LTE connectivity
* [ ] Branch connectivity
* [ ] SD-WAN
* [ ] Out-of-band management
* [ ] LAN extension
* [ ] WAN extension
* [ ] VXLAN over IPsec

```text
                 FortiGate
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
      WAN-1        WAN-2      FortiExtender
                                  │
                                  ▼
                                 LTE
```

---

# 🌐 18. FortiExtender WAN Mode Checklist

Use WAN mode when FortiExtender provides WAN connectivity.

* [ ] FortiExtender is reachable
* [ ] FortiGate controller is configured
* [ ] CAPWAP connectivity works
* [ ] LTE modem is operational
* [ ] SIM is recognized
* [ ] APN is correct
* [ ] IP address is obtained
* [ ] Default route is correct
* [ ] SD-WAN membership is configured if required
* [ ] Failover policy is tested

---

# 🏢 19. FortiExtender LAN Mode Checklist

For LAN extension:

* [ ] LAN mode is required
* [ ] FortiExtender is authorized
* [ ] Controller connectivity is established
* [ ] VXLAN requirements are understood
* [ ] IPsec requirements are understood
* [ ] Layer-2 extension design is validated
* [ ] MTU implications are checked

Concept:

```text
Branch LAN
    │
    ▼
FortiExtender
    │
    ▼
VXLAN over IPsec
    │
    ▼
Headquarters
```

---

# 🔐 20. FortiExtender Management Checklist

FortiExtender uses CAPWAP for communication with FortiGate.

Verify:

* [ ] FortiExtender appliance exists
* [ ] Appropriate licensing is available
* [ ] FortiGate controller is configured
* [ ] CAPWAP connectivity exists
* [ ] Device authorization is complete

Example:

```bash
config extender-controller extender
    edit fex
        set authorized enable
    end
end
```

---

# ⚙️ 21. Enable FortiExtender Checklist

Example:

```bash
config system global
    set fortiextender enable
    set fortiextender-data-port 25246
    set fortiextender-discovery-lockdown disable
    set fortiextender-vlan-mode disable
    set fortiservice-port 8013
    set fortitoken-cloud enable
end
```

Validate:

* [ ] FortiExtender feature enabled
* [ ] Discovery behavior understood
* [ ] Required service ports reachable
* [ ] GUI feature visibility enabled if required

GUI:

```text
System
 └── Feature Visibility
      └── FortiExtender
```

Then:

```text
Network
 └── FortiExtender
```

---

# 📶 22. LTE Modem Checklist

Example:

```bash
config system lte-modem
    set status enable
    set apn rightel
end
```

Validate:

* [ ] LTE modem enabled
* [ ] APN is correct
* [ ] SIM is detected
* [ ] Carrier is detected
* [ ] Network type is correct
* [ ] Modem obtains connectivity
* [ ] DHCP works where applicable
* [ ] Default route is correct
* [ ] LTE path is reachable

---

# 📱 23. Advanced LTE Configuration Checklist

Example:

```bash
config system lte-modem
    set status enable
    set extra-init port1
    set manual-handover disable
    set force-wireless-profile 0
    set authtype none
    set apn rightel
    set modem-port 255
    set network-type auto
    set auto-connect disable
    set gpsd-enabled disable
    set data-usage-tracking disable
    set gps-port 255
end
```

### Validate

* [ ] Authentication mode
* [ ] APN
* [ ] Network type
* [ ] Auto-connect behavior
* [ ] Modem port
* [ ] GPS requirements
* [ ] Data usage tracking

> `modem-port 255` can indicate automatic modem selection in the referenced configuration context.

---

# 🔄 24. FortiExtender + SD-WAN Checklist

A common architecture:

```text
                     SD-WAN
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
       Internet        MPLS       FortiExtender
                                      │
                                      ▼
                                     LTE
```

Verify:

* [ ] FortiExtender interface is added to SD-WAN
* [ ] SD-WAN priority is intentional
* [ ] SLA health checks are configured
* [ ] LTE is used as primary or backup as intended
* [ ] Failover is tested
* [ ] Recovery is tested

---

# 🛡️ 25. FortiExtender Redundancy Checklist

Possible technologies/use cases:

* [ ] VRRP
* [ ] Carrier failover
* [ ] Active/Passive connectivity
* [ ] Backup WAN
* [ ] OOB management

Verify:

* [ ] Primary path
* [ ] Secondary path
* [ ] Failure trigger
* [ ] Recovery trigger
* [ ] Route preference
* [ ] SD-WAN preference
* [ ] Monitoring mechanism

---

# 🛰️ 26. Out-of-Band Management Checklist

For OOB design:

* [ ] Primary WAN failure scenario defined
* [ ] LTE/FortiExtender path available
* [ ] Management subnet reachable
* [ ] Management policy allows required access
* [ ] Routing is independent enough from primary WAN
* [ ] Failure has been tested

```text
Primary WAN
     │
     ❌
     │
     ▼
FortiExtender
     │
     ▼
   LTE
     │
     ▼
Management
```

---

# 📊 27. FortiExtender Capacity Checklist

Always validate platform-specific limits before deployment.

| Platform Family                   | WAN |  LAN |
| --------------------------------- | --: | ---: |
| 40F / variants / VM01             |   2 |    0 |
| 60F / 70F / 80F / variants / VM02 |   2 |   16 |
| 100E → 200F / variants            |   2 |   16 |
| 400F → 900G / variants / VM04     |   2 |   32 |
| 1000D → 2600F / variants / VM08   |   2 |  256 |
| 3000D+ / VM16+                    |   2 | 1024 |

* [ ] Platform identified
* [ ] FortiOS version verified
* [ ] WAN limit verified
* [ ] LAN limit verified
* [ ] Deployment capacity validated

> ⚠️ Platform limits can vary by FortiOS release and model. Always verify against the documentation for the exact platform/version before production deployment.

---

# 📶 28. LTE Category Checklist

Approximate theoretical category capabilities:

| LTE Category | Approx. Peak Capability |
| ------------ | ----------------------: |
| Cat 4        |                150 Mbps |
| Cat 6        |                300 Mbps |
| Cat 7        |            100–300 Mbps |
| Cat 12       |                600 Mbps |
| Cat 16       |                  1 Gbps |
| Cat 20       |                2–5 Gbps |

* [ ] Modem category verified
* [ ] Carrier capability verified
* [ ] Spectrum verified
* [ ] Signal quality checked
* [ ] Real-world throughput tested

> ⚠️ LTE category speed is **not the same as guaranteed real-world throughput**.

---

# 🧭 29. Complete WAN Failover Checklist

Before production:

### Primary Path

* [ ] Primary WAN is operational
* [ ] Default route is correct
* [ ] NAT is correct
* [ ] Firewall policies are correct

### Health Monitoring

* [ ] Link Monitor configured
* [ ] Correct source interface
* [ ] Correct destination
* [ ] Correct gateway
* [ ] Correct timers
* [ ] Correct failure threshold

### Secondary Path

* [ ] FortiExtender/LTE available
* [ ] Backup WAN has connectivity
* [ ] Backup route exists
* [ ] SD-WAN membership is correct

### Failure

* [ ] Disconnect primary WAN
* [ ] Verify health-check failure
* [ ] Verify route change
* [ ] Verify SD-WAN decision
* [ ] Verify traffic moves to backup

### Recovery

* [ ] Restore primary WAN
* [ ] Verify health-check recovery
* [ ] Verify route restoration
* [ ] Verify traffic returns as designed

---

# 🚨 30. Golden Troubleshooting Flow

When the path is DOWN:

```text
                 PATH FAILURE
                      │
                      ▼
              Check Link Monitor
                      │
                      ▼
               Check srcintf
                      │
                      ▼
              Check gateway
                      │
                      ▼
              Check server
                      │
                      ▼
             Check probe state
                      │
                      ▼
              Check routing
                      │
                      ▼
              Check firewall
                      │
                      ▼
                Check NAT
                      │
                      ▼
                Check BFD
                      │
                      ▼
              Check SD-WAN
                      │
                      ▼
                Check HA
                      │
                      ▼
             Check return path
```

---

# 🧪 31. Essential CLI Checklist

Run these during troubleshooting:

```bash
# Link Monitor status
diagnose sys link-monitor status

# Link Monitor tunnel information
diagnose sys link-monitor tunnel all

# Interface status
diagnose sys link-monitor interface red-1

# Manually launch health check
diagnose sys link-monitor launch mik-1-red

# Filter Link Monitor objects
diagnose sys link-monitor filter name mik

# Static routing
get router info routing-table static
```

* [ ] Status checked
* [ ] Tunnel checked
* [ ] Interface checked
* [ ] Manual probe tested
* [ ] Filter applied where necessary
* [ ] Routing table verified

---

# 🧠 32. NSE Exam Memory Map

| If You See...              | Think...                         |
| -------------------------- | -------------------------------- |
| `srcintf`                  | Probe source interface           |
| `server`                   | Health-check destination         |
| `gateway-ip`               | Path gateway                     |
| `failtime`                 | Failure threshold                |
| `recoverytime`             | Recovery threshold               |
| `probe-count`              | Probe behavior                   |
| `interval`                 | Probe interval                   |
| `probe-timeout`            | Probe response timeout           |
| `update-static-route`      | Static route reaction            |
| `update-policy-route`      | Policy route reaction            |
| `update-cascade-interface` | Interface cascade                |
| `server-list`              | Individual monitored servers     |
| `weight`                   | Failure weight                   |
| `fail-weight`              | Total failure threshold          |
| `class-id`                 | SD-WAN association               |
| `service-detection`        | Service-quality monitoring       |
| `MOS`                      | Voice/service quality            |
| `ha-priority`              | HA-related Link Monitor behavior |
| `BFD`                      | Fast failure detection           |
| `server-type dynamic`      | Dynamic tunnel endpoint scenario |
| `FortiExtender`            | External WAN/LTE connectivity    |
| `CAPWAP`                   | FortiExtender management         |
| `LTE`                      | Cellular WAN                     |
| `VRRP`                     | Redundancy                       |
| `VXLAN over IPsec`         | LAN extension                    |
| `OOB`                      | Out-of-band management           |

---

# 🔥 33. Interview Questions Checklist

Use these questions to test yourself:

* [ ] What problem does Link Monitor solve?
* [ ] Why is `srcintf` important?
* [ ] What happens when the monitored server becomes unreachable?
* [ ] What is the difference between `failtime` and `recoverytime`?
* [ ] What does `update-static-route` do?
* [ ] What does `update-policy-route` do?
* [ ] What does `update-cascade-interface` do?
* [ ] Why can cascade behavior be dangerous in redundant designs?
* [ ] How does `server-list` work?
* [ ] How do `weight` and `fail-weight` interact?
* [ ] What is `class-id` related to?
* [ ] What is service detection?
* [ ] What is MOS?
* [ ] When would you use BFD?
* [ ] How does Link Monitor behave in HA?
* [ ] How would you troubleshoot a failed health check?
* [ ] How does FortiExtender connect to FortiGate?
* [ ] What is CAPWAP used for?
* [ ] When would you use FortiExtender as a backup WAN?
* [ ] How can FortiExtender participate in SD-WAN?
* [ ] What is the difference between WAN and LAN mode?
* [ ] What is VXLAN over IPsec used for?
* [ ] How can LTE provide backup connectivity?
* [ ] What should you verify before using LTE as a production failover path?

---

# ✅ 34. Production Readiness Checklist

Before declaring the design ready:

* [ ] Monitoring destination is reliable
* [ ] Monitoring source interface is explicit
* [ ] Underlay routing is correct
* [ ] Return path is correct
* [ ] Failure thresholds are reasonable
* [ ] Recovery thresholds are reasonable
* [ ] Static route behavior is tested
* [ ] Policy route behavior is tested
* [ ] Cascade behavior is intentional
* [ ] NAT behavior is understood
* [ ] Firewall policies are correct
* [ ] SD-WAN rules are correct
* [ ] BFD behavior is understood
* [ ] HA behavior is tested
* [ ] FortiExtender is authorized
* [ ] CAPWAP is operational
* [ ] LTE/SIM/APN are validated
* [ ] Backup path is tested
* [ ] Recovery path is tested
* [ ] Monitoring/alerting is configured
* [ ] Failure scenarios are documented

---

# ⚡ One-Minute Revision

```text
LINK MONITOR
│
├── Source Interface
├── Server
├── Gateway
├── Health Check
│
├── Failure
│   ├── Static Route
│   ├── Policy Route
│   └── Interface Cascade
│
├── SD-WAN
│   ├── Class ID
│   └── Service Detection
│
├── HA
│   └── HA Priority
│
└── Troubleshooting
    ├── Status
    ├── Tunnel
    ├── Interface
    ├── Launch
    └── Routing
```

```text
FORTIEXTENDER
│
├── WAN
│   ├── LTE
│   ├── Backup WAN
│   └── SD-WAN
│
├── LAN
│   └── LAN Extension
│
├── Management
│   └── CAPWAP
│
├── Redundancy
│   └── VRRP
│
└── Tunneling
    └── VXLAN over IPsec
```

---

# 🔥 Key Takeaways

> **1.** Link Monitor is fundamentally a **path-health mechanism**, not merely a ping feature.

> **2.** Always control and verify the **source interface** when path-specific monitoring matters.

> **3.** Route changes, policy-route changes and interface cascade are separate design consequences and must be tested independently.

> **4.** `server-list` + `weight` + `fail-weight` enables more granular health evaluation.

> **5.** `update-cascade-interface` can dramatically change redundant-interface behavior.

> **6.** BFD is useful when rapid failure detection is required.

> **7.** Link Monitor can contribute to SD-WAN path-health decisions.

> **8.** Service detection and MOS address service quality rather than simple IP reachability.

> **9.** FortiExtender is a powerful option for LTE backup, branch connectivity, SD-WAN and out-of-band access.

> **10.** In every failover design, test both **failure and recovery**.

> **11.** A successful health check does not automatically mean the application is healthy.

> **12.** A tunnel/path being UP does not guarantee end-to-end connectivity; always validate **routing, policy, NAT and return path**.

---

# 🏷️ Keywords

`FortiGate Link Monitor` · `FortiOS 7.2.0 Link Monitor` · `FortiGate Health Check` · `FortiGate WAN Failover` · `FortiGate SD-WAN` · `FortiGate BFD` · `FortiGate Link Monitor Troubleshooting` · `FortiGate Redundant WAN` · `FortiGate Route Failover` · `FortiGate Service Detection` · `FortiGate MOS` · `FortiExtender` · `FortiExtender LTE` · `FortiExtender SD-WAN` · `FortiGate LTE Backup` · `FortiGate Out-of-Band Management` · `FortiExtender CAPWAP` · `VXLAN over IPsec` · `Fortinet NSE4` · `Fortinet NSE7` · `FortiGate Troubleshooting` · `Network Security Engineering`

---

## 🔗 SheynShield Resources

### 🎥 Video Learning

* [YouTube — SheynShield](https://youtube.com/@sheynshield)

  * Fortinet NSE content
  * FortiGate troubleshooting
  * Network Security Engineering

### 📚 Notes & Updates

* [Telegram — SheynShield](https://t.me/sheynshield)

### 💼 Professional Network

* [LinkedIn — Shayan-heydarikhah](https://linkedin.com/in/shayan-heydarikhah)

### 🐙 Technical Knowledge Base

* [SheynShield GitHub](https://github.com/Shayan-heydarikhah/sheynshield)

---

## 🧭 SheynShield Knowledge Base

**Fortinet → FortiGate → Routing → SD-WAN → WAN Failover → Link Monitor → FortiExtender → LTE → Network Security Engineering**

> **Learn the command. Understand the mechanism. Design the network. Troubleshoot the failure.**

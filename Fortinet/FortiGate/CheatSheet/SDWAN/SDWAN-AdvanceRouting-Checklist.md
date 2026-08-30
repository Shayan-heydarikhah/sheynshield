# FortiGate SD-WAN — Advanced Routing & Overlay Checklist

> **FortiOS | SD-WAN | BGP | Route Tags | Route Maps | ADVPN | Neighbors | FEC | Hold-Down | Troubleshooting**

> [!NOTE]
> This checklist is designed for **FortiGate SD-WAN deployment, validation, troubleshooting, and design review**.
> Always verify syntax and feature behavior against the exact **FortiOS release** before production deployment.

---

## 📑 Table of Contents

* [1. Local-Out Routing Checklist](#1-local-out-routing-checklist)
* [2. SD-WAN + BGP Checklist](#2-sd-wan--bgp-checklist)
* [3. BGP Route-Tag Steering Checklist](#3-bgp-route-tag-steering-checklist)
* [4. SD-WAN + ADVPN Checklist](#4-sd-wan--advpn-checklist)
* [5. BGP Route-Map Traffic Control Checklist](#5-bgp-route-map-traffic-control-checklist)
* [6. SD-WAN Neighbor Checklist](#6-sd-wan-neighbor-checklist)
* [7. Multiple Members per Neighbor Checklist](#7-multiple-members-per-neighbor-checklist)
* [8. SD-WAN + ADVPN Overlay Design Checklist](#8-sd-wan--advpn-overlay-design-checklist)
* [9. ADVPN Troubleshooting Checklist](#9-advpn-troubleshooting-checklist)
* [10. SD-WAN Hold-Down Checklist](#10-sd-wan-hold-down-checklist)
* [11. FEC Checklist](#11-fec-checklist)
* [12. FEC Mapping Checklist](#12-fec-mapping-checklist)
* [13. SD-WAN Verification Checklist](#13-sd-wan-verification-checklist)
* [14. Routing & BGP Verification Checklist](#14-routing--bgp-verification-checklist)
* [15. IPsec Verification Checklist](#15-ipsec-verification-checklist)
* [16. NPU & Session Verification Checklist](#16-npu--session-verification-checklist)
* [17. End-to-End Troubleshooting Checklist](#17-end-to-end-troubleshooting-checklist)
* [18. SD-WAN Design Rules](#18-sd-wan-design-rules)
* [19. Quick Command Reference](#19-quick-command-reference)

---

# 1. Local-Out Routing Checklist

## 🎯 Objective

Use Local-Out Routing to control how **FortiGate-generated traffic** exits the firewall.

Typical traffic includes:

* [ ] DNS
* [ ] FortiGuard
* [ ] NTP
* [ ] FortiCloud
* [ ] Central Management
* [ ] Active probes
* [ ] Other FortiGate-generated traffic

---

## Routing Method

Verify the selected Local-Out method:

| Method      | Purpose                    |
| ----------- | -------------------------- |
| `auto`      | Automatic path selection   |
| `sdwan`     | Use SD-WAN service/rules   |
| `interface` | Force a specific interface |

### Validation

* [ ] Confirm the Local-Out feature is enabled/visible.
* [ ] Identify which FortiGate-generated service is being tested.
* [ ] Identify the intended egress interface.
* [ ] Verify SD-WAN rules if `sdwan` is selected.
* [ ] Verify routing if no SD-WAN rule matches.
* [ ] Verify the selected interface when `interface` is configured.

---

## Traceroute Through SD-WAN

```bash
execute traceroute-options use-sdwan yes
execute traceroute 8.8.8.8
```

* [ ] Enable SD-WAN-aware traceroute when required.
* [ ] Confirm the selected path.
* [ ] Compare SD-WAN path with normal routing behavior.

---

## Central Management

```cli
config system central-management
    set interface-select-method specify
    set interface port1
end
```

* [ ] Confirm Central Management traffic uses the intended interface.
* [ ] Verify routing from the selected interface.
* [ ] Confirm upstream connectivity.

---

## NTP

```cli
config system ntp
    config ntpserver
        edit 1
            set interface-select-method specify
            set interface port1
        next
    end
end
```

* [ ] Confirm NTP interface selection.
* [ ] Verify NTP server reachability.
* [ ] Verify return routing.

---

## DHCP Proxy

```cli
config system settings
    set dhcp-proxy-interface-select-method specific
    set dhcp-proxy-interface port1
end
```

* [ ] Confirm DHCP proxy interface.
* [ ] Verify DHCP server reachability.
* [ ] Verify return traffic.

---

## TLS Active Probe

```cli
config system global
    config tls-active-probe
        set interface-select-method sdwan
    end
end
```

* [ ] Confirm probe interface selection.
* [ ] Verify SD-WAN member state.
* [ ] Verify SLA/health-check behavior.

---

> [!TIP]
> When troubleshooting Local-Out traffic, validate **interface selection → routing → SD-WAN decision → FIB → gateway** instead of assuming normal client traffic and FortiGate-generated traffic behave identically.

---

# 2. SD-WAN + BGP Checklist

## Topology

```text
                ISP / WAN
              /          \
          Link-1        Link-2
             \            /
              +--- FGT ---+
                    |
                 SD-WAN
                    |
                   BGP
```

### Base Validation

* [ ] WAN interfaces are operational.
* [ ] WAN addressing is correct.
* [ ] SD-WAN members are configured.
* [ ] SD-WAN zones are correctly defined.
* [ ] SLA/health checks are operational.
* [ ] BGP neighbors are reachable.
* [ ] BGP sessions are established.
* [ ] Required routes are advertised.
* [ ] Unnecessary customer routes are not advertised.

---

## SD-WAN SLA

Example:

```text
SLA-1
├── Server: 22.22.22.2
└── Server: 2.2.2.2
```

* [ ] Define appropriate SLA targets.
* [ ] Verify latency.
* [ ] Verify jitter.
* [ ] Verify packet loss.
* [ ] Confirm SLA thresholds match the application requirement.

---

## SD-WAN Rules

### Primary Path

```text
Source       : all
Destination  : 192.168.102.0/24
Outgoing     : zone-main
SLA          : SLA-1
```

* [ ] Confirm source matching.
* [ ] Confirm destination matching.
* [ ] Confirm service matching.
* [ ] Confirm SLA assignment.
* [ ] Confirm preferred zone/member.

### Backup Path

```text
Source       : all
Destination  : 192.168.102.0/24
Outgoing     : zone-back
SLA          : SLA-1
```

* [ ] Confirm backup rule exists.
* [ ] Confirm rule ordering.
* [ ] Confirm failover behavior.
* [ ] Confirm backup SLA state.

---

## BGP

Recommended validation:

* [ ] Use interface-based neighbors where appropriate.
* [ ] Configure required multihop.
* [ ] Configure soft reconfiguration where required.
* [ ] Enable BGP multipath when multiple valid paths are required.
* [ ] Verify local AS.
* [ ] Verify remote AS.
* [ ] Verify neighbor IP.
* [ ] Verify route advertisements.
* [ ] Verify received routes.

Example:

```text
Remote AS: 65002

Neighbor:
    2.2.2.2

Neighbor:
    22.22.22.2
```

---

## BGP Multipath

Verify:

* [ ] Multiple valid BGP paths exist.
* [ ] EBGP multipath requirements are satisfied.
* [ ] Multihop requirements are satisfied.
* [ ] Next-hop reachability is valid.
* [ ] Equal-path requirements are satisfied.
* [ ] FIB contains the expected forwarding path.

---

# 3. BGP Route-Tag Steering Checklist

## Objective

Use BGP route tags to classify routes and influence SD-WAN path selection.

```text
BGP Route
    |
    v
Route-Map IN
    |
    v
Route Tag
    |
    v
SD-WAN Rule
    |
    v
Preferred WAN
```

### Validation

* [ ] Identify the route that requires steering.
* [ ] Create a matching prefix-list/ACL.
* [ ] Apply the route-map.
* [ ] Assign the intended route tag.
* [ ] Configure SD-WAN service matching.
* [ ] Verify the tag.
* [ ] Refresh BGP when required.
* [ ] Verify final member selection.

---

## Prefix Match

Example:

```text
ACL-20

Prefix:
192.168.20.0/24

Match:
Exact

Action:
Permit
```

Checklist:

* [ ] Prefix is correct.
* [ ] Exact/less-specific matching is intentional.
* [ ] Permit/deny behavior is correct.
* [ ] Catch-all rule does not override the intended match.

---

## Route Map

```text
RMP-20

Rule 1
├── ACL: ACL-20
├── Tag: 20
└── Permit

Rule 2
├── ACL: ACL-ANY
└── Permit
```

* [ ] Specific route is matched first.
* [ ] Tag is assigned correctly.
* [ ] Catch-all behavior is intentional.
* [ ] Route-map sequence is correct.

---

## SD-WAN Route Tag

```cli
config system sdwan
    config service
        edit 1
            set route-tag 20
        next
    end
end
```

* [ ] Route tag matches the intended BGP classification.
* [ ] SD-WAN service matches the traffic.
* [ ] Correct zone/member is preferred.
* [ ] SLA requirements are satisfied.

---

## BGP Refresh

```bash
execute router clear bgp all soft
```

* [ ] Perform soft refresh when required.
* [ ] Verify BGP routes again.
* [ ] Verify route tag.
* [ ] Verify SD-WAN selected member.

---

## Verification

```bash
get router info bgp network
```

Check:

* [ ] Prefix
* [ ] Next-hop
* [ ] AS path
* [ ] Best path
* [ ] Route tag

---

## Policy Routing

```bash
diagnose firewall proute list
```

* [ ] Verify policy-route entries.
* [ ] Verify SD-WAN-related forwarding decisions.
* [ ] Verify `vwl`/virtual SD-WAN behavior where applicable.

---

# 4. SD-WAN + ADVPN Checklist

## Architecture

```text
                 HUB
               /     \
          Spoke-1   Spoke-2
               \     /
                ADVPN
                  +
                SD-WAN
```

### Validate

* [ ] Hub is correctly configured.
* [ ] Spokes are correctly configured.
* [ ] ADVPN discovery is enabled as required.
* [ ] IPsec tunnels are established.
* [ ] BGP is established where used.
* [ ] SD-WAN members are correctly assigned.
* [ ] SLA checks are operational.
* [ ] Spoke-to-spoke dynamic tunnel behavior is tested.

---

## Hub

Verify:

```text
Role:
    Hub

Auto-Discovery Sender:
    ENABLE

Add Route:
    DISABLE

Device Creation:
    DISABLE
```

* [ ] Hub uses the correct ADVPN role.
* [ ] Auto-Discovery Sender is enabled.
* [ ] Add Route behavior is intentional.
* [ ] Automatic device creation behavior is intentional.

---

## Spoke

Verify:

```text
Auto-Discovery Receiver:
    ENABLE

Device Creation:
    ENABLE

Add Route:
    DISABLE
```

* [ ] Auto-Discovery Receiver is enabled.
* [ ] Device creation is enabled where required.
* [ ] Add Route behavior is intentional.
* [ ] Spoke can establish the required overlay.

---

## ADVPN + SD-WAN Warning

> [!WARNING]
> Avoid using **Maximize Bandwidth / Load-Balance** with ADVPN members where the FortiOS design/version does not support that combination.

Prefer validating:

* [ ] SLA
* [ ] Priority
* [ ] Best Quality

---

## SD-WAN Zone Cleanup

When ADVPN auto-created policies/interfaces are involved:

* [ ] Identify auto-created objects.
* [ ] Remove obsolete policies where appropriate.
* [ ] Create explicit SD-WAN zones.
* [ ] Create explicit SD-WAN services.
* [ ] Verify static/dynamic routing.
* [ ] Verify SLA.
* [ ] Verify forwarding behavior.

---

# 5. BGP Route-Map Traffic Control Checklist

## Objective

Combine:

```text
BGP
+
Route Maps
+
SLA
+
SD-WAN
+
Neighbor Awareness
```

for advanced traffic steering.

---

## Normal Route Map

```text
RTM-DENY-103

Rule 1:
    DENY 192.168.103.0/24

Rule 2:
    PERMIT ANY
```

Checklist:

* [ ] Restricted prefix is explicitly matched.
* [ ] Deny sequence is before permit-any.
* [ ] Backup behavior is understood.
* [ ] Route propagation is verified.

---

## Preferred Route Map

```text
RTM-BYPASS

Rule 1:
    PERMIT 192.168.103.0/24

Rule 2:
    PERMIT ANY
```

* [ ] Preferred route is explicitly permitted.
* [ ] Catch-all rule is intentional.
* [ ] Preferred route behavior is tested.

---

## BGP Neighbor

```cli
config router bgp
    config neighbor
        edit 12.23.34.1
            set route-map-out rtm-deny-103
            set route-map-out-prefer rtm-bypass
        next
    end
end
```

* [ ] Normal route-map is correct.
* [ ] Preferred route-map is correct.
* [ ] Neighbor is correct.
* [ ] BGP session is established.
* [ ] SLA state is verified.
* [ ] BGP soft refresh is performed when required.

```bash
execute router clear bgp all soft
```

---

# 6. SD-WAN Neighbor Checklist

## Neighbor Roles

| Role         | Purpose                                          |
| ------------ | ------------------------------------------------ |
| `primary`    | Preferred neighbor when SLA requirements are met |
| `secondary`  | Backup neighbor                                  |
| `standalone` | Independent operation                            |

---

## Primary / Secondary Logic

```text
SLA OK
  |
  v
PRIMARY
  |
  X
FAIL
  |
  v
SECONDARY
```

Checklist:

* [ ] Primary neighbor is configured.
* [ ] Secondary neighbor is configured.
* [ ] SLA is associated with the neighbor.
* [ ] Priority is correct.
* [ ] Failover has been tested.
* [ ] Recovery behavior has been tested.

---

## Example

```cli
config system sdwan
    config neighbor

        edit 12.23.34.1
            set member 1
            set role primary
            set health-check SLA_AD
            set sla-id 1
        next

        edit 10.11.12.1
            set member 2
            set role secondary
            set health-check SLA_AD
            set sla-id 1
        next

    end
end
```

---

## Neighbor Health Check

```cli
config system sdwan
    config health-check
        edit SLA_AD
            set server 12.23.34.1 10.11.12.1
            set members 0

            config sla
                edit 1
                next
            end
        next
    end
end
```

Verify:

* [ ] Correct health-check name.
* [ ] Correct target IPs.
* [ ] Correct member association.
* [ ] Correct SLA ID.
* [ ] Correct thresholds.

---

## Verification

```bash
diagnose sys sdwan health-check
diagnose sys sdwan neighbor
diagnose sys sdwan service
```

* [ ] Neighbor state is expected.
* [ ] SLA state is expected.
* [ ] Selected service/member is expected.

---

# 7. Multiple Members per Neighbor Checklist

Use multiple SD-WAN members when one neighbor is reachable through multiple links.

```cli
config system sdwan
    config neighbor
        edit 12.23.34.1
            set member 1 2
            set minimum-sla-meet-member 2
        next
    end
end
```

### Validate

* [ ] Neighbor has the expected members.
* [ ] Member 1 is healthy.
* [ ] Member 2 is healthy.
* [ ] `minimum-sla-meet-member` is correctly configured.
* [ ] Preferred state occurs only when required members meet SLA.
* [ ] Failure behavior is tested.

```text
Member-1 ── SLA OK ──┐
                     ├── Preferred
Member-2 ── SLA OK ──┘

One member fails
        |
        v
Preferred condition not met
        |
        v
Normal / Backup behavior
```

---

# 8. SD-WAN + ADVPN Overlay Design Checklist

## ADVPN Objectives

* [ ] Dynamic spoke-to-spoke tunnels are required.
* [ ] Hub bandwidth is considered.
* [ ] Latency requirements are documented.
* [ ] Spoke scalability is considered.
* [ ] Dynamic tunnel behavior is tested.

### Benefits

* [ ] Lower latency
* [ ] Reduced hub traffic
* [ ] Reduced provisioning effort
* [ ] Improved scalability
* [ ] Dynamic spoke-to-spoke connectivity

---

## Dual-WAN ADVPN

```text
                  HUB
               /       \
          Main WAN    Backup WAN
              |          |
           Spoke-1     Spoke-2
              \          /
               \________/
                  ADVPN
```

Combine:

```text
ADVPN
  +
SD-WAN
  |
  +── SLA steering
  +── Priority
  +── Best Quality
  +── Performance selection
```

Checklist:

* [ ] Primary WAN defined.
* [ ] Backup WAN defined.
* [ ] ADVPN tunnels established.
* [ ] SD-WAN members mapped correctly.
* [ ] SLA is operational.
* [ ] Preferred path is deterministic.
* [ ] Failover is tested.
* [ ] Recovery is tested.

---

# 9. ADVPN Troubleshooting Checklist

## Phase 1 — IKE Negotiation

```bash
diagnose vpn ike gateway list
diagnose vpn ike sa list
```

Check:

* [ ] Local IP
* [ ] Remote IP
* [ ] IKE version
* [ ] Encryption
* [ ] Authentication
* [ ] PSK
* [ ] DH group
* [ ] Phase-1 proposal

---

## Phase 1 — Established

```bash
diagnose vpn ike sa list
```

Verify:

* [ ] IKE SA exists.
* [ ] State is established.
* [ ] Encryption is correct.
* [ ] Authentication is correct.
* [ ] Lifetime is expected.

---

## Phase 2 — Negotiation

```bash
diagnose vpn ipsec sa list
diagnose vpn ipsec sa filter
```

Check:

* [ ] Local selector
* [ ] Remote selector
* [ ] Phase-2 proposal
* [ ] Encryption
* [ ] Authentication
* [ ] PFS
* [ ] DH group

---

## Phase 2 — Established

```bash
diagnose vpn ipsec sa list
diagnose vpn tunnel list
diagnose vpn tunnel stats
```

Verify:

* [ ] IPsec SA exists.
* [ ] Tunnel is up.
* [ ] TX counters increase.
* [ ] RX counters increase.
* [ ] Encryption/decryption counters behave as expected.
* [ ] Lifetime is valid.

> [!WARNING]
> If Phase 1 is established but Phase 2 fails, the expected Phase-2/IPsec interface information may not appear in the relevant SA output.

---

## Tunnel Down

```bash
diagnose vpn ike gateway list
diagnose vpn ike sa list

diagnose debug application ike -1
diagnose debug enable
```

Check:

* [ ] PSK
* [ ] Local/remote IP
* [ ] IKE version
* [ ] Phase-1 proposal
* [ ] Phase-2 proposal
* [ ] Selectors
* [ ] DH group
* [ ] Authentication

---

## Restart IPsec

```bash
diagnose vpn ipsec restart
```

* [ ] Restart only when operationally appropriate.
* [ ] Confirm tunnels renegotiate.
* [ ] Verify new SAs.

---

## Restart IKE

```bash
diagnose vpn ike restart
```

* [ ] Confirm IKE renegotiation.
* [ ] Verify IKE SA.
* [ ] Verify IPsec SA afterward.

---

## IPsec Packet Debug

```bash
diagnose vpn ipsec packet
```

Check:

* [ ] Packet headers.
* [ ] Encryption behavior.
* [ ] Decryption behavior.
* [ ] Direction of traffic.

---

# 10. SD-WAN Hold-Down Checklist

## Problem

WAN flapping can cause repeated path switching:

```text
UP
 |
DOWN
 |
UP
 |
DOWN
 |
UP
```

---

## Configure Hold-Down

```cli
config system sdwan
    config service
        edit 1
            set hold-down-time 100
        next
    end
end
```

Checklist:

* [ ] Identify whether path flapping exists.
* [ ] Determine acceptable recovery delay.
* [ ] Configure `hold-down-time`.
* [ ] Verify failover.
* [ ] Verify recovery.
* [ ] Confirm timer behavior.

### Logic

```text
Primary
   |
 FAIL
   |
   v
Secondary
   |
   v
Primary Recovers
   |
Hold-Down Timer
   |
   v
Primary
```

> [!IMPORTANT]
> `0` means no hold-down protection.

---

## Verification

```bash
diagnose sys sdwan service
```

Check:

* [ ] Service state
* [ ] Selected member
* [ ] Hold-down behavior
* [ ] Recovery state

---

# 11. Forward Error Correction (FEC) Checklist

## Objective

FEC adds redundancy so packet loss can potentially be recovered without retransmission.

```text
Base Packets
      +
Redundant Packets
      |
      v
     VPN
      |
 Packet Loss
      |
      v
 Recovery
```

---

## FEC Use Cases

Recommended candidates:

* [ ] VoIP
* [ ] UDP
* [ ] Real-time traffic
* [ ] Loss-sensitive applications

Potentially unnecessary:

* [ ] Normal TCP traffic
* [ ] Loss-insensitive applications
* [ ] High-bandwidth traffic where redundancy cost is unacceptable

---

## FEC Design Review

* [ ] Packet-loss profile is understood.
* [ ] Additional bandwidth consumption is calculated.
* [ ] CPU impact is considered.
* [ ] Both tunnel endpoints are configured correctly.
* [ ] FEC health-check behavior is understood.
* [ ] FEC mapping is appropriate.
* [ ] Application traffic is explicitly selected.

---

## Phase-1 FEC

Example:

```cli
config vpn ipsec phase1-interface
    edit link-1
        set npu-offload enable
        set fec-egress enable
        set fec-ingress enable
        set fec-health-check 1
        set fec-mapping-profile map-1
        set fec-base 10
        set fec-redundant 1
    next
end
```

Validate:

* [ ] FEC egress enabled.
* [ ] FEC ingress enabled.
* [ ] FEC health-check configured.
* [ ] FEC mapping profile exists.
* [ ] Base packet value is appropriate.
* [ ] Redundant packet value is appropriate.
* [ ] Peer-side configuration is compatible.

---

## FEC vs NPU

```text
FEC Traffic
     |
     v
CPU Processing
```

Validate actual platform/FortiOS behavior rather than assuming every FEC flow is offloaded.

* [ ] Check NPU capability for the platform.
* [ ] Check tunnel configuration.
* [ ] Check session output.
* [ ] Confirm actual offload state.

---

# 12. FEC Mapping Checklist

FEC mappings can associate network conditions with redundancy levels.

```text
Packet Loss
     +
Bandwidth
     +
SLA / Health
     |
     v
FEC Mapping
     |
     v
Base + Redundant Packets
```

---

## Mapping Example

```cli
config vpn ipsec fec
    edit map-1

        config mappings

            edit 1
                set base 8
                set redundant 2
                set packet-loss-threshold 10
            next

            edit 2
                set base 9
                set redundant 3
                set bandwidth-up-threshold 950000
            next

        end
    next
end
```

Checklist:

* [ ] Mapping profile exists.
* [ ] Packet-loss threshold is appropriate.
* [ ] Bandwidth threshold is appropriate.
* [ ] Base packet value is appropriate.
* [ ] Redundant packet value is appropriate.
* [ ] Mapping order is intentional.
* [ ] Conditions do not unintentionally overlap.

---

## Mapping Evaluation

```text
Mapping 1
   |
   v
Condition?
   |
  NO
   |
   v
Mapping 2
   |
   v
Condition?
   |
  YES
   |
   v
Apply Base / Redundant
```

---

## FEC Firewall Policy

For selected UDP/VoIP traffic:

```cli
config firewall policy
    edit 1
        set srcintf port3
        set dstintf sdwan-1
        set action accept
        set srcaddr 192.168.101.0/24
        set dstaddr all
        set schedule always
        set service ALL_UDP
        set fec enable
    next
end
```

Checklist:

* [ ] Correct source interface.
* [ ] Correct destination interface.
* [ ] Correct source address.
* [ ] Correct destination address.
* [ ] Correct service.
* [ ] FEC enabled only where required.
* [ ] General traffic is not unintentionally FEC-protected.

---

# 13. SD-WAN Verification Checklist

## Service

```bash
diagnose sys sdwan service
```

Check:

* [ ] Matching service.
* [ ] Selected member.
* [ ] Priority.
* [ ] SLA state.
* [ ] Failover state.
* [ ] Hold-down behavior.

---

## Health Check

```bash
diagnose sys sdwan health-check
```

Check:

* [ ] Member
* [ ] SLA
* [ ] Latency
* [ ] Jitter
* [ ] Packet loss
* [ ] Status
* [ ] Target reachability

---

## Neighbor

```bash
diagnose sys sdwan neighbor
```

Check:

* [ ] Neighbor IP.
* [ ] Member association.
* [ ] Role.
* [ ] Health-check.
* [ ] SLA ID.
* [ ] Current state.

---

## Packet History

```bash
diagnose sys sdwan packet-history
```

Use to investigate:

* [ ] Selected path.
* [ ] Member behavior.
* [ ] Traffic steering.
* [ ] Unexpected forwarding.

---

# 14. Routing & BGP Verification Checklist

## Routing Table

```bash
get router info routing-table all
get router info routing-table static
get router info routing-table bgp
```

Verify:

* [ ] Destination exists.
* [ ] Correct next-hop.
* [ ] Correct outgoing interface.
* [ ] Expected route source.
* [ ] No unexpected more-specific route.

---

## BGP

```bash
get router info bgp network
```

Check:

* [ ] Prefix exists.
* [ ] Next-hop is reachable.
* [ ] Best path is expected.
* [ ] AS path is expected.
* [ ] Route tag is expected.
* [ ] Multiple paths exist when required.

---

## BGP Refresh

```bash
execute router clear bgp all soft
```

* [ ] Use soft reset where appropriate.
* [ ] Re-check learned routes.
* [ ] Re-check advertised routes.
* [ ] Re-check route tags.

---

## Policy Route

```bash
diagnose firewall proute list
```

* [ ] Verify policy routes.
* [ ] Verify matching criteria.
* [ ] Check for unexpected policy-route precedence.

---

# 15. IPsec Verification Checklist

## IKE Gateway

```bash
diagnose vpn ike gateway list
```

Check:

* [ ] Gateway exists.
* [ ] Local IP.
* [ ] Remote IP.
* [ ] IKE version.
* [ ] State.

---

## IKE SA

```bash
diagnose vpn ike sa list
```

Check:

* [ ] SA exists.
* [ ] Authentication.
* [ ] Encryption.
* [ ] Lifetime.
* [ ] Established state.

---

## IPsec SA

```bash
diagnose vpn ipsec sa list
```

Check:

* [ ] SA exists.
* [ ] Local selector.
* [ ] Remote selector.
* [ ] Algorithms.
* [ ] Lifetime.
* [ ] Tunnel association.

---

## Tunnel

```bash
diagnose vpn tunnel list
diagnose vpn tunnel stats
```

Check:

* [ ] Tunnel exists.
* [ ] Tunnel state.
* [ ] TX traffic.
* [ ] RX traffic.
* [ ] Encryption/decryption counters.

---

# 16. NPU & Session Verification Checklist

## Session

```bash
diagnose sys session list
```

Check:

* [ ] Session exists.
* [ ] Correct source/destination.
* [ ] Correct ingress interface.
* [ ] Correct egress interface.
* [ ] NPU information.
* [ ] Offload state.

Example:

```text
npu info:
flag=0x82/0x81
offload=8/8
ips_offload=0/0
epid=249/74
ipid=74/86
```

### Design Validation

```text
Normal eligible traffic
        |
        v
NPU Offload
```

versus:

```text
FEC-protected traffic
        |
        v
Check actual platform/FortiOS offload behavior
```

> [!IMPORTANT]
> Do not infer NPU support solely from a generic rule. Always verify the actual session and platform behavior.

---

# 17. End-to-End Troubleshooting Checklist

Use this sequence when SD-WAN traffic is not following the expected path.

```text
Traffic Problem
      |
      v
SD-WAN Rule Match?
      |
   +--+--+
   |     |
  NO    YES
   |     |
Check    SLA Healthy?
Policy       |
Route     +--+--+
          |     |
         NO    YES
          |     |
       Check    Check
       SLA      Selected
       Target   Member
                   |
                   v
                FIB/RIB
                   |
                   v
                  BGP
                   |
                   v
                 ADVPN
                   |
                   v
                 IPsec
                   |
                   v
             Phase 1 / Phase 2
                   |
                   v
                 FEC/NPU
                   |
                   v
                Session
```

---

## Phase 1 — Policy & SD-WAN

* [ ] Source matches SD-WAN service.
* [ ] Destination matches.
* [ ] Service matches.
* [ ] SD-WAN rule order is correct.
* [ ] Correct SLA is attached.
* [ ] Selected member is expected.

---

## Phase 2 — SLA

* [ ] SLA server is reachable.
* [ ] Probe is active.
* [ ] Latency is acceptable.
* [ ] Jitter is acceptable.
* [ ] Packet loss is acceptable.
* [ ] SLA threshold is not unnecessarily aggressive.

---

## Phase 3 — Routing

* [ ] Destination exists in RIB.
* [ ] Expected route is selected.
* [ ] Next-hop is reachable.
* [ ] FIB contains expected forwarding entry.
* [ ] No policy route overrides the expected decision.

---

## Phase 4 — BGP

* [ ] BGP neighbor is established.
* [ ] Expected route is learned.
* [ ] Expected route is advertised.
* [ ] Best path is correct.
* [ ] Route tag is correct.
* [ ] Route-map behavior is correct.
* [ ] Multipath is configured if required.

---

## Phase 5 — ADVPN

* [ ] Hub discovery is operational.
* [ ] Spoke discovery is operational.
* [ ] Dynamic tunnel is created when expected.
* [ ] Spoke-to-spoke connectivity works.
* [ ] SD-WAN recognizes the correct tunnel/member.

---

## Phase 6 — IPsec

* [ ] Phase 1 is established.
* [ ] Phase 2 is established.
* [ ] Selectors match.
* [ ] Encryption/authentication match.
* [ ] PFS/DH settings match.
* [ ] Traffic counters increase.

---

## Phase 7 — FEC/NPU

* [ ] FEC is required for the traffic.
* [ ] FEC mapping is correct.
* [ ] FEC policy matches.
* [ ] Packet loss is actually present.
* [ ] CPU impact is acceptable.
* [ ] Actual NPU behavior has been verified.

---

# 18. SD-WAN Design Rules

## 🧠 Core Memory

> **SD-WAN Rule = Decide which traffic uses which path**

> **SLA = Measure WAN quality**

> **BGP = Exchange routes dynamically**

> **Route Tag = Classify routes for steering**

> **Route Map = Filter or manipulate routes**

> **ADVPN = Build dynamic spoke-to-spoke VPN connectivity**

> **SD-WAN Neighbor = Define neighbor/path relationship**

> **Primary = Preferred path**

> **Secondary = Backup path**

> **Standalone = Independent path**

> **Hold-Down = Prevent excessive path flapping**

> **FEC = Add redundancy for packet-loss recovery**

> **FIB = Final forwarding decision**

> **RIB = Routing information**

> **NPU = Hardware acceleration/offload**

---

## 🔀 SD-WAN Decision Model

```text
                         Packet
                            |
                            v
                    SD-WAN Service
                            |
             +--------------+--------------+
             |              |              |
             v              v              v
          Manual       Best Quality     Lowest Cost
             |              |              |
             |              v              v
             |             SLA             SLA
             |              |              |
             +--------------+--------------+
                            |
                            v
                      Route / FIB
                            |
                            v
                  Selected SD-WAN Member
                            |
                            v
                         Gateway
```

---

## 🌐 SD-WAN + BGP + ADVPN

```text
                    ┌──────────────┐
                    │     HUB      │
                    │   BGP/ADVPN  │
                    └──────┬───────┘
                           │
                  ┌────────┴────────┐
                  │                 │
              Main WAN          Backup WAN
                  │                 │
              ┌───┴───┐         ┌───┴───┐
              │Spoke-1│         │Spoke-2│
              └───┬───┘         └───┬───┘
                  │                 │
                  └───── ADVPN ─────┘
                         |
                      SD-WAN
                         |
          +--------------+--------------+
          |              |              |
         SLA          Priority        BGP
          |              |              |
          +--------------+--------------+
                         |
                    Best Forwarder
```

---

# 19. Quick Command Reference

## SD-WAN

```bash
diagnose sys sdwan service
diagnose sys sdwan health-check
diagnose sys sdwan neighbor
diagnose sys sdwan packet-history
```

* [ ] Check service.
* [ ] Check SLA.
* [ ] Check neighbor.
* [ ] Check packet history.

---

## Routing

```bash
get router info routing-table all
get router info routing-table static
get router info routing-table bgp
```

* [ ] Verify RIB.
* [ ] Verify static routes.
* [ ] Verify BGP routes.

---

## BGP

```bash
get router info bgp network
execute router clear bgp all soft
```

* [ ] Verify learned prefixes.
* [ ] Verify route attributes.
* [ ] Soft-refresh when required.

---

## Policy Routing

```bash
diagnose firewall proute list
```

* [ ] Verify policy routes.
* [ ] Check route-selection interaction.

---

## IKE / IPsec

```bash
diagnose vpn ike gateway list
diagnose vpn ike sa list
diagnose vpn ipsec sa list
diagnose vpn tunnel list
diagnose vpn tunnel stats
```

* [ ] Verify Phase 1.
* [ ] Verify Phase 2.
* [ ] Verify tunnel.
* [ ] Verify counters.

---

## IPsec Debug

```bash
diagnose debug application ike -1
diagnose debug enable
```

* [ ] Enable only when required.
* [ ] Reproduce the issue.
* [ ] Stop debugging after collection.

---

## FEC

```bash
diagnose vpn tunnel fec link-1
```

```bash
diagnose sys sdwan health-check
diagnose sys sdwan packet-history
```

* [ ] Verify FEC state.
* [ ] Verify SLA.
* [ ] Verify packet behavior.

---

## Session / NPU

```bash
diagnose sys session list
```

* [ ] Find the relevant session.
* [ ] Check NPU flags.
* [ ] Check offload state.
* [ ] Compare FEC vs non-FEC traffic.

---

# ⚠️ Critical Design Review

Before production deployment, verify:

* [ ] SD-WAN service order is intentional.
* [ ] SLA thresholds reflect application requirements.
* [ ] BGP advertisements are minimized to required prefixes.
* [ ] Route maps do not accidentally suppress required routes.
* [ ] Route tags are consistent end-to-end.
* [ ] ADVPN discovery roles are correct.
* [ ] ADVPN tunnels are tested dynamically.
* [ ] Primary/secondary behavior is deterministic.
* [ ] Hold-down is configured where WAN flapping exists.
* [ ] FEC is enabled only for traffic that benefits from it.
* [ ] FEC bandwidth overhead is understood.
* [ ] NPU impact has been tested on the actual platform.
* [ ] FIB/RIB behavior has been validated.
* [ ] Failover has been tested.
* [ ] Recovery has been tested.
* [ ] Packet captures have been collected for unexplained forwarding behavior.

---

# 🚀 Final SD-WAN Deployment Checklist

## Before Deployment

* [ ] Document topology.
* [ ] Document underlay.
* [ ] Document overlay.
* [ ] Document BGP ASNs.
* [ ] Document prefixes.
* [ ] Document SD-WAN members.
* [ ] Document SLA thresholds.
* [ ] Document preferred/backup paths.
* [ ] Document ADVPN roles.
* [ ] Document FEC requirements.

## During Deployment

* [ ] Configure WAN members.
* [ ] Configure SD-WAN zones.
* [ ] Configure SLA.
* [ ] Configure SD-WAN services.
* [ ] Configure BGP.
* [ ] Configure route maps/tags.
* [ ] Configure ADVPN.
* [ ] Configure neighbors.
* [ ] Configure hold-down where required.
* [ ] Configure FEC where required.

## After Deployment

* [ ] Verify SLA.
* [ ] Verify SD-WAN service.
* [ ] Verify BGP.
* [ ] Verify routing table.
* [ ] Verify FIB.
* [ ] Verify ADVPN.
* [ ] Verify IPsec.
* [ ] Verify failover.
* [ ] Verify recovery.
* [ ] Verify packet forwarding.
* [ ] Verify session/NPU behavior.
* [ ] Document final operational state.

---

# 🏁 One-Minute Mental Model

```text
                 APPLICATION
                      |
                      v
                SD-WAN SERVICE
                      |
                      v
                  SLA / TAG
                      |
          +-----------+-----------+
          |                       |
          v                       v
        BGP                    ROUTING
          |                       |
          +-----------+-----------+
                      |
                      v
                     FIB
                      |
                      v
                SD-WAN MEMBER
                      |
                      v
                 IPsec / ADVPN
                      |
          +-----------+-----------+
          |                       |
          v                       v
         FEC                    NPU
          |                       |
          +-----------+-----------+
                      |
                      v
                   SESSION
```

> **Measure → Classify → Route → Select → Forward → Verify**

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

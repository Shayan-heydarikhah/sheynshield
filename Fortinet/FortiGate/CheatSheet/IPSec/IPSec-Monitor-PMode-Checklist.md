# 🔐 FortiGate IPsec Tunnel Monitoring & Passive Mode Checklist

> **SheynShield | FortiGate IPsec VPN Engineering Checklist**
> Practical checklist for **IPsec tunnel monitoring, primary/backup VPN failover, passive mode, routing validation, and FortiGate IPsec troubleshooting**.

---

## 📌 Table of Contents

* [🎯 Scope](#-scope)
* [🏗️ IPsec Tunnel Monitoring Checklist](#️-ipsec-tunnel-monitoring-checklist)
* [🔗 Monitoring Relationship Checklist](#-monitoring-relationship-checklist)
* [🔄 Tunnel Failover Checklist](#-tunnel-failover-checklist)
* [🛣️ Routing Checklist](#️-routing-checklist)
* [🔐 Phase 1 Checklist](#-phase-1-checklist)
* [🔒 Phase 2 Checklist](#-phase-2-checklist)
* [🖥️ Remote FortiGate Checklist](#️-remote-fortigate-checklist)
* [⏸️ Passive Mode Checklist](#️-passive-mode-checklist)
* [⚠️ Passive Mode Failure Scenarios](#️-passive-mode-failure-scenarios)
* [🧪 Verification Commands](#-verification-commands)
* [🚨 Troubleshooting Workflow](#-troubleshooting-workflow)
* [📊 Primary vs Backup Checklist](#-primary-vs-backup-checklist)
* [🧠 Exam & Interview Checklist](#-exam--interview-checklist)
* [⚡ Quick Reference](#-quick-reference)
* [🔗 SheynShield Resources](#-sheynshield-resources)

---

# 🎯 Scope

Use this checklist when designing or troubleshooting:

* [ ] Site-to-site IPsec VPN
* [ ] Primary/backup IPsec tunnels
* [ ] FortiGate IPsec tunnel monitoring
* [ ] IPsec failover
* [ ] Passive IPsec tunnels
* [ ] IKE negotiation problems
* [ ] Phase 1 failures
* [ ] Phase 2 failures
* [ ] Routing-related VPN failures
* [ ] Tunnel state and SA verification

> [!IMPORTANT]
> **IPsec tunnel monitoring and routing are separate mechanisms.** Monitoring can influence tunnel behavior, but routing still determines which usable path carries traffic.

---

# 🏗️ IPsec Tunnel Monitoring Checklist

## Topology

```text
                 ┌───────────────┐
                 │    FGT-1      │
                 └───────┬───────┘
                         │
              ┌──────────┴──────────┐
              │                     │
          Link-1                 Link-2
          PRIMARY                BACKUP
              │                     │
           ISP-1                  ISP-2
              │                     │
              └──────────┬──────────┘
                         │
                    Remote FGT
```

## Design Validation

* [ ] Confirm this is a **site-to-site/static IPsec** design.
* [ ] Identify the primary tunnel.
* [ ] Identify the backup tunnel.
* [ ] Identify the WAN interface used by each tunnel.
* [ ] Confirm both tunnels terminate on the expected remote peer.
* [ ] Confirm routing supports primary/backup behavior.
* [ ] Confirm return traffic has a valid path.

---

# 🔗 Monitoring Relationship Checklist

To make `link-2` monitor `link-1`:

```bash
config vpn ipsec phase1-interface
    edit "link-2"
        set monitor "link-1"
    next
end
```

Verify:

* [ ] `link-1` is the intended primary tunnel.
* [ ] `link-2` is the intended backup tunnel.
* [ ] `link-2` contains `set monitor "link-1"`.
* [ ] The monitored tunnel name is correct.
* [ ] The monitoring relationship is intentionally configured.
* [ ] Do not assume monitoring is automatically bidirectional.

### Mental Model

```text
link-1
PRIMARY
   ▲
   │
monitored by
   │
   │
link-2
BACKUP
```

### Key Point

```text
set monitor "link-1"
```

means:

```text
link-2 → monitors → link-1
```

It does **not** mean:

```text
link-1 ↔ link-2
```

---

# 🔄 Tunnel Failover Checklist

## Normal State

```text
Link-1
  │
  ├── UP
  │
  └── PRIMARY

Link-2
  │
  ├── STANDBY
  │
  └── BACKUP
```

Verify:

* [ ] Primary tunnel is established.
* [ ] Backup tunnel is available according to the intended design.
* [ ] Primary route has better preference.
* [ ] Backup route has worse preference.

## Failure State

```text
Link-1
  │
  └── DOWN
       ↓
    Failover
       ↓
Link-2
  │
  └── ACTIVE
```

Test:

* [ ] Disable/fail the primary WAN path in a controlled test.
* [ ] Verify primary tunnel state changes.
* [ ] Verify backup tunnel becomes usable.
* [ ] Verify routing changes as expected.
* [ ] Verify application traffic uses the backup path.
* [ ] Verify return traffic follows the correct path.
* [ ] Restore the primary path.
* [ ] Verify expected recovery behavior.

> [!WARNING]
> Do not validate failover only by looking at the IPsec SA. **Tunnel state, routing, policy, and return-path behavior must all be checked.**

---

# 🛣️ Routing Checklist

Tunnel monitoring does not replace routing.

## Static Routing

* [ ] Primary route exists.
* [ ] Backup route exists.
* [ ] Primary route has the preferred administrative distance.
* [ ] Backup route has a less-preferred administrative distance.
* [ ] Destination prefixes are correct.
* [ ] Next-hop/interface selection is correct.

Example:

```text
Primary:

192.168.102.0/24
        │
        ▼
      Link-1
      AD 10
```

```text
Backup:

192.168.102.0/24
        │
        ▼
      Link-2
      AD 20
```

Expected:

```text
Link-1 UP
   ↓
Primary route selected

Link-1 DOWN
   ↓
Backup route selected
```

## Advanced Routing

Check whether the design uses:

* [ ] Static routing
* [ ] SD-WAN
* [ ] OSPF
* [ ] BGP
* [ ] Other dynamic routing
* [ ] Policy-based routing

For SD-WAN designs:

* [ ] SD-WAN health checks are correct.
* [ ] SD-WAN rules select the intended tunnel.
* [ ] SLA criteria are correct.
* [ ] Failover behavior is tested.

---

# 🔐 Phase 1 Checklist

Verify both peers:

* [ ] IKE version matches.
* [ ] Encryption proposal matches.
* [ ] Integrity/authentication algorithm matches.
* [ ] DH group matches.
* [ ] Authentication method matches.
* [ ] PSK matches.
* [ ] Peer IP is correct.
* [ ] Local interface is correct.
* [ ] Local/remote identity is correct.
* [ ] IKE mode is compatible.
* [ ] Lifetime is compatible.
* [ ] NAT-T requirements are understood.
* [ ] Local-in policy permits required IKE traffic.

Typical legacy example:

```text
IKEv1
Aggressive Mode
DES
MD5
DH5
PSK
```

> [!WARNING]
> DES, MD5, and weak DH groups are **legacy cryptographic choices** and should not be used for new secure deployments. Prefer modern algorithms and groups supported by the FortiOS release and peer device.

---

# 🔒 Phase 2 Checklist

Verify:

* [ ] ESP encryption matches.
* [ ] ESP integrity matches.
* [ ] PFS configuration matches.
* [ ] PFS DH group matches.
* [ ] Phase 2 lifetime is compatible.
* [ ] Local selectors match.
* [ ] Remote selectors match.
* [ ] Required subnets are included.
* [ ] Auto-negotiate behavior is understood.
* [ ] NAT exemption/design is correct where applicable.
* [ ] Routing points required traffic into the VPN.

### Selector Validation

```text
FGT-1 LAN
192.168.101.0/24
        │
        │ IPsec
        ▼
FGT-2 LAN
192.168.102.0/24
```

Check:

```text
Local selector  = 192.168.101.0/24
Remote selector = 192.168.102.0/24
```

---

# 🖥️ Remote FortiGate Checklist

Verify both endpoints:

* [ ] Phase 1 proposals match.
* [ ] Phase 2 proposals match.
* [ ] PSK matches.
* [ ] DH/PFS settings match.
* [ ] IKE version matches.
* [ ] Peer addresses are correct.
* [ ] IPsec selectors are compatible.
* [ ] Routing is correct.
* [ ] Firewall policies permit required traffic.
* [ ] NAT behavior is correct.
* [ ] WAN connectivity is working.
* [ ] UDP 500 is reachable when required.
* [ ] UDP 4500 is reachable when NAT-T is used.

---

# ⏸️ Passive Mode Checklist

## Understand the Behavior

Passive mode changes the initiation behavior of the FortiGate.

```text
Active:

FortiGate
   │
   ├── Initiates IKE
   ▼
Peer
```

```text
Passive:

FortiGate
   │
   └── Waits for peer
            ▲
            │
        Peer initiates
```

Verify:

* [ ] Passive mode is required by the design.
* [ ] The expected IKE initiator is identified.
* [ ] The peer is capable of initiating negotiation.
* [ ] Rekey behavior is understood.
* [ ] Tunnel-interface behavior is understood.
* [ ] Failover behavior is tested after enabling passive mode.

---

# ⚙️ Passive Mode Configuration Checklist

Example:

```bash
config vpn ipsec phase1-interface
    edit "link-1"
        set rekey enable
        set passive-mode enable
        set passive-tunnel-interface enable
    next
end
```

Validate:

* [ ] `set passive-mode enable` is intentional.
* [ ] `set rekey enable` is configured according to the design.
* [ ] `set passive-tunnel-interface enable` is required for the intended tunnel behavior.
* [ ] The remote peer is configured to initiate when required.
* [ ] The configuration is validated against the specific FortiOS version.

> [!NOTE]
> FortiOS behavior and available options can vary between releases. Always validate the exact CLI syntax and semantics against the FortiOS version deployed.

---

# ⚠️ Passive Mode Failure Scenarios

## Passive + Passive

Avoid blindly configuring both peers as passive.

```text
FGT-1
Passive
   │
   └── waits

FGT-2
Passive
   │
   └── waits
```

Potential result:

```text
No active initiator
       ↓
IKE negotiation does not start
       ↓
Tunnel remains DOWN
```

Checklist:

* [ ] At least one side can initiate when the design requires active negotiation.
* [ ] The intended initiator is documented.
* [ ] Passive mode is not accidentally enabled on both peers.
* [ ] Rekey/reconnect behavior has been tested.
* [ ] Tunnel establishment has been verified after configuration changes.

---

# 📊 Active vs Passive Checklist

| Check                       | Active |          Passive         |
| --------------------------- | :----: | :----------------------: |
| Can initiate IKE            |    ✅   |             ❌            |
| Can respond to IKE          |    ✅   |             ✅            |
| Waits for peer initiation   |    ❌   |             ✅            |
| Can participate in rekey    |    ✅   | Depends on configuration |
| Typical default design      |    ✅   |        Specialized       |
| Requires initiator planning |   Low  |         **High**         |

### Mental Model

```text
ACTIVE

FortiGate
   │
   ├── IKE Initiation
   ▼
Peer
```

```text
PASSIVE

FortiGate
   │
   └── WAIT
        ▲
        │
   Peer initiates
```

---

# 🧪 Verification Commands

## Check IPsec Tunnel State

```bash
diagnose vpn tunnel list
```

Check:

* [ ] Tunnel state
* [ ] Phase 2 SA
* [ ] SPI
* [ ] Encryption
* [ ] Authentication/integrity
* [ ] Traffic counters
* [ ] Local selectors
* [ ] Remote selectors
* [ ] Encapsulation information

---

## Check IKE Gateway

```bash
diagnose vpn ike gateway list
```

Check:

* [ ] IKE SA state
* [ ] Peer address
* [ ] Local address
* [ ] IKE version
* [ ] Authentication
* [ ] Negotiated proposal
* [ ] Gateway state

---

# 🔎 Verification Workflow

Run:

```bash
diagnose vpn ike gateway list
```

Then:

```bash
diagnose vpn tunnel list
```

Determine:

```text
Phase 1 UP?
     │
     ├── NO → Troubleshoot IKE
     │
     └── YES
          │
          ▼
     Phase 2 UP?
          │
          ├── NO → Troubleshoot selectors/PFS/proposals
          │
          └── YES
               │
               ▼
          Traffic working?
               │
               ├── NO → Routing/policy/NAT/MTU
               │
               └── YES → VPN operational
```

---

# 🚨 Troubleshooting Workflow

## Step 1 — Physical / ISP

* [ ] WAN interface is up.
* [ ] Default route exists.
* [ ] ISP connectivity works.
* [ ] Remote peer is reachable.
* [ ] No upstream filtering exists.

---

## Step 2 — IKE Transport

* [ ] UDP 500 is allowed where required.
* [ ] UDP 4500 is allowed when NAT-T is used.
* [ ] Local-in policy is checked.
* [ ] NAT behavior is understood.
* [ ] Correct public IP is used.
* [ ] Correct peer IP is used.

---

## Step 3 — Phase 1

* [ ] IKE version
* [ ] Encryption
* [ ] Integrity
* [ ] DH group
* [ ] Authentication
* [ ] PSK
* [ ] Identity
* [ ] Peer configuration
* [ ] Lifetime
* [ ] NAT-T

---

## Step 4 — Phase 2

* [ ] ESP encryption
* [ ] ESP integrity
* [ ] PFS
* [ ] PFS DH group
* [ ] Local selector
* [ ] Remote selector
* [ ] Lifetime
* [ ] Auto-negotiate
* [ ] IPSec SA

---

## Step 5 — Tunnel Monitoring

* [ ] Monitoring is configured on the intended tunnel.
* [ ] Correct tunnel is being monitored.
* [ ] Monitoring direction is understood.
* [ ] Primary/backup relationship is documented.
* [ ] Failure behavior has been tested.

---

## Step 6 — Routing

* [ ] Primary route exists.
* [ ] Backup route exists.
* [ ] Route preference is correct.
* [ ] SD-WAN behavior is correct if used.
* [ ] Dynamic routing behavior is correct if used.
* [ ] Return route exists.
* [ ] No asymmetric routing problem exists.

---

## Step 7 — Firewall Policy

* [ ] Source is correct.
* [ ] Destination is correct.
* [ ] Service is correct.
* [ ] Policy order is correct.
* [ ] NAT is correct.
* [ ] Inter-zone traffic is permitted.
* [ ] VPN-to-LAN policy exists.
* [ ] LAN-to-VPN policy exists.

---

## Step 8 — MTU / MSS

If the tunnel establishes but applications fail:

* [ ] Check MTU.
* [ ] Check fragmentation.
* [ ] Check TCP MSS.
* [ ] Check PMTUD behavior.
* [ ] Check packet counters.
* [ ] Test small vs large packets.

Concept:

```text
Original MTU
     │
     ├── IPsec overhead
     ▼
Effective MTU
     │
     ▼
Potential fragmentation
```

Example policy adjustment:

```bash
config firewall policy
    edit 1
        set tcp-mss-sender 1350
        set tcp-mss-receiver 1350
    next
end
```

> [!WARNING]
> Do not blindly use `1350`. Determine the correct MSS from the actual path MTU and IPsec encapsulation overhead.

---

# 🧭 Primary vs Backup Checklist

| Item             | Primary | Backup |
| ---------------- | :-----: | :----: |
| IPsec tunnel     |   [ ]   |   [ ]  |
| WAN interface    |   [ ]   |   [ ]  |
| Peer IP          |   [ ]   |   [ ]  |
| Phase 1          |   [ ]   |   [ ]  |
| Phase 2          |   [ ]   |   [ ]  |
| Route            |   [ ]   |   [ ]  |
| Route preference |   [ ]   |   [ ]  |
| Monitoring       |   [ ]   |   [ ]  |
| SD-WAN rule      |   [ ]   |   [ ]  |
| Failover tested  |   [ ]   |   [ ]  |
| Recovery tested  |   [ ]   |   [ ]  |

---

# 🧪 Failover Test Plan

## Before Failure

* [ ] Primary tunnel is UP.
* [ ] Backup tunnel is in the expected state.
* [ ] Primary route is selected.
* [ ] Application traffic is flowing.
* [ ] IPsec counters are increasing.

## Simulate Failure

* [ ] Disconnect primary WAN.
* [ ] Disable primary interface.
* [ ] Simulate primary peer failure.
* [ ] Use a controlled test environment.

## Verify Failover

* [ ] Primary tunnel becomes unavailable.
* [ ] Backup tunnel becomes usable.
* [ ] Routing changes.
* [ ] Traffic moves to backup.
* [ ] Return traffic works.
* [ ] Applications recover.

## Restore

* [ ] Restore primary WAN.
* [ ] Verify primary tunnel re-establishment.
* [ ] Verify route preference.
* [ ] Confirm whether traffic returns to primary as designed.
* [ ] Confirm no unexpected flapping occurs.

---

# 🧠 Failure Mapping Checklist

| Symptom                            | Check                                 |
| ---------------------------------- | ------------------------------------- |
| ❌ No IKE SA                        | Peer reachability / UDP 500 / Phase 1 |
| ❌ Phase 1 fails                    | Proposal / PSK / DH / identity        |
| ❌ Phase 1 UP, Phase 2 DOWN         | Selectors / PFS / Phase 2 proposal    |
| ❌ Tunnel UP, no traffic            | Routing / policy / selectors / NAT    |
| ❌ Backup never becomes usable      | Monitoring / routing / tunnel state   |
| ❌ Passive tunnel never starts      | Initiator / passive configuration     |
| ❌ Both peers wait                  | Passive + passive configuration       |
| ❌ Large packets fail               | MTU / MSS / fragmentation             |
| ❌ NAT environment fails            | NAT-T / UDP 4500                      |
| ❌ Failover works but traffic fails | Routing / policy / return path        |
| ❌ Tunnel flaps                     | DPD / NAT timeout / WAN instability   |

---

# 🛠️ Debug Checklist

When standard verification is insufficient:

```bash
diagnose debug enable
diagnose debug application ike -1
```

Check:

* [ ] IKE negotiation starts.
* [ ] Peer receives IKE packets.
* [ ] Proposal is accepted.
* [ ] Authentication succeeds.
* [ ] Phase 1 establishes.
* [ ] Phase 2 establishes.
* [ ] Rekey behavior is correct.
* [ ] Passive behavior is as expected.

After troubleshooting:

```bash
diagnose debug disable
diagnose debug reset
```

> [!WARNING]
> Always stop debugging after collecting the required information.

---

# 🔐 Security Checklist

For new deployments:

* [ ] Prefer IKEv2 where supported.
* [ ] Prefer modern encryption algorithms.
* [ ] Prefer SHA-2 or stronger integrity mechanisms where supported.
* [ ] Use strong DH groups.
* [ ] Use strong PSKs if PSK authentication is required.
* [ ] Prefer certificates for scalable authentication environments.
* [ ] Avoid DES.
* [ ] Avoid 3DES for new deployments.
* [ ] Avoid MD5.
* [ ] Avoid SHA-1 for new deployments.
* [ ] Avoid weak DH groups.
* [ ] Review NAT-T requirements.
* [ ] Restrict IKE exposure where appropriate.
* [ ] Review FortiGate local-in policies.
* [ ] Log and monitor VPN events.

---

# 📋 Configuration Review Checklist

## Phase 1

* [ ] Interface
* [ ] Remote gateway
* [ ] IKE version
* [ ] Authentication
* [ ] Encryption
* [ ] Integrity
* [ ] DH
* [ ] PSK/certificate
* [ ] Identity
* [ ] NAT-T
* [ ] Passive mode
* [ ] Rekey
* [ ] Monitoring

## Phase 2

* [ ] Encryption
* [ ] Integrity
* [ ] PFS
* [ ] DH
* [ ] Lifetime
* [ ] Selectors
* [ ] Auto-negotiate

## Routing

* [ ] Primary route
* [ ] Backup route
* [ ] Administrative distance
* [ ] SD-WAN
* [ ] Dynamic routing
* [ ] Return path

## Security Policy

* [ ] LAN → VPN
* [ ] VPN → LAN
* [ ] NAT
* [ ] Local-in policy
* [ ] Logging

---

# 🎯 Operational Mental Model

```text
                 IPsec VPN
                     │
          ┌──────────┴──────────┐
          │                     │
       Tunnel-1              Tunnel-2
       PRIMARY                BACKUP
          │                     │
          └──────────┬──────────┘
                     │
                 Monitoring
                     │
                     ▼
               Tunnel State
                     │
                     ▼
                  Routing
                     │
                     ▼
              Selected Path
                     │
                     ▼
                  Traffic
```

### Remember

```text
Tunnel Monitoring
        +
Routing
        +
IPsec SA
        +
Firewall Policy
        +
Return Path
        =
Reliable VPN Failover
```

---

# 🧠 Exam & Interview Checklist

### Tunnel Monitoring

* [ ] What is IPsec tunnel monitoring used for?
* [ ] Is it primarily intended for site-to-site/static IPsec?
* [ ] Which tunnel monitors which tunnel?
* [ ] Is monitoring automatically bidirectional?
* [ ] Does monitoring replace routing?

### Passive Mode

* [ ] What does passive mode change?
* [ ] Which peer initiates IKE?
* [ ] Can a passive peer respond?
* [ ] What happens if both peers are passive?
* [ ] How does passive mode affect tunnel establishment?

### Troubleshooting

* [ ] Can you verify IKE SA state?
* [ ] Can you verify Phase 2 SA?
* [ ] Can you verify SPI?
* [ ] Can you identify a selector mismatch?
* [ ] Can you identify a routing failure?
* [ ] Can you identify a policy/NAT problem?
* [ ] Can you test failover safely?

---

# ⚡ Quick Reference

## Tunnel Monitoring

```bash
config vpn ipsec phase1-interface
    edit "link-2"
        set monitor "link-1"
    next
end
```

Meaning:

```text
link-2 → monitors → link-1
```

---

## Passive Mode

```bash
config vpn ipsec phase1-interface
    edit "link-1"
        set rekey enable
        set passive-mode enable
        set passive-tunnel-interface enable
    next
end
```

Remember:

```text
Passive
   ↓
Does not actively initiate IKE
   ↓
Waits for peer
```

---

## IPsec Tunnel Status

```bash
diagnose vpn tunnel list
```

---

## IKE Status

```bash
diagnose vpn ike gateway list
```

---

## IKE Debug

```bash
diagnose debug enable
diagnose debug application ike -1
```

Stop:

```bash
diagnose debug disable
diagnose debug reset
```

---

# 🚀 One-Minute Troubleshooting Checklist

```text
IPsec DOWN
   │
   ▼
[ ] WAN/ISP
   │
   ▼
[ ] UDP 500 / UDP 4500
   │
   ▼
[ ] IKE Version
   │
   ▼
[ ] Proposal
   │
   ▼
[ ] PSK / Authentication
   │
   ▼
[ ] DH
   │
   ▼
[ ] Phase 2
   │
   ▼
[ ] Selectors
   │
   ▼
[ ] PFS
   │
   ▼
[ ] Monitoring
   │
   ▼
[ ] Routing
   │
   ▼
[ ] Firewall Policy
   │
   ▼
[ ] NAT
   │
   ▼
[ ] MTU / MSS
   │
   ▼
[ ] Logs / Debug
```

---

# 🔥 Golden Rules

> [!IMPORTANT]
> **IPsec tunnel monitoring is not a replacement for routing.**

> [!IMPORTANT]
> **`set monitor "link-1"` means the configured tunnel monitors `link-1`; always verify the direction.**

> [!WARNING]
> **Passive mode requires deliberate initiator/responder planning.**

> [!WARNING]
> **Passive + passive can prevent a tunnel from being actively initiated.**

> [!TIP]
> **When troubleshooting, separate Phase 1, Phase 2, routing, policy, and data-plane problems instead of treating "VPN DOWN" as one problem.**

> [!TIP]
> **Use `diagnose vpn ike gateway list` for IKE state and `diagnose vpn tunnel list` for IPsec/Phase 2 state.**

---

# 📚 Related FortiGate Topics

Use this checklist together with:

* [ ] FortiGate IPsec Phase 1 Checklist
* [ ] FortiGate IPsec Phase 2 Checklist
* [ ] FortiGate IKEv1 / IKEv2 Cheat Sheet
* [ ] FortiGate DPD & NAT-T Checklist
* [ ] FortiGate ADVPN Checklist
* [ ] FortiGate SD-WAN VPN Failover Checklist
* [ ] FortiGate IPsec Troubleshooting Checklist
* [ ] FortiGate Firewall Policy Troubleshooting Checklist
* [ ] FortiGate Routing Troubleshooting Checklist

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

# 🛡️ SheynShield

**FortiGate IPsec | VPN Engineering | Network Security | Troubleshooting**

> Build it. Secure it. Troubleshoot it.
> **SheynShield — Engineering Secure Networks**

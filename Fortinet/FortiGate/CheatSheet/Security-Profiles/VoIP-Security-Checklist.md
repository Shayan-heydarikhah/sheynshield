# 🔐 FortiGate VoIP Security & SIP — Checklist

> **FortiOS | VoIP Security | SIP ALG | HNT | SIP Proxy | RTP/RTCP | SIP Pinholes | MSRP | LLDP-MED | IPS**
>
> Practical **FortiGate VoIP security, SIP troubleshooting, NAT traversal, RTP/RTCP and NSE4/NSE7 reference** for network and security engineers.

---

## 📌 Table of Contents

* [1. VoIP Quick Reference](#1-voip-quick-reference)
* [2. SIP vs SCCP](#2-sip-vs-sccp)
* [3. FortiGate VoIP Architecture](#3-fortigate-voip-architecture)
* [4. VoIP Feature Visibility](#4-voip-feature-visibility)
* [5. VoIP Profile Checklist](#5-voip-profile-checklist)
* [6. SIP ALG Checklist](#6-sip-alg-checklist)
* [7. HNT Checklist](#7-hnt-checklist)
* [8. SIP VIP & NAT Checklist](#8-sip-vip--nat-checklist)
* [9. SIP Firewall Policy Checklist](#9-sip-firewall-policy-checklist)
* [10. SIP + SDP Checklist](#10-sip--sdp-checklist)
* [11. RTP & RTCP Checklist](#11-rtp--rtcp-checklist)
* [12. SIP Pinhole Checklist](#12-sip-pinhole-checklist)
* [13. SIP over TLS Checklist](#13-sip-over-tls-checklist)
* [14. Flow vs Proxy Inspection](#14-flow-vs-proxy-inspection)
* [15. MSRP Inspection Checklist](#15-msrp-inspection-checklist)
* [16. SIP DoS Protection](#16-sip-dos-protection)
* [17. LLDP-MED & Voice VLAN](#17-lldp-med--voice-vlan)
* [18. SIP Troubleshooting Checklist](#18-sip-troubleshooting-checklist)
* [19. No Audio / One-Way Audio](#19-no-audio--one-way-audio)
* [20. Registration Failure](#20-registration-failure)
* [21. Diagnostic Commands](#21-diagnostic-commands)
* [22. Production Security Checklist](#22-production-security-checklist)
* [23. Common VoIP Mistakes](#23-common-voip-mistakes)
* [24. NSE4/NSE7 Exam Traps](#24-nse4nse7-exam-traps)
* [25. One-Minute Revision](#25-one-minute-revision)
* [26. Final VoIP Mental Model](#26-final-voip-mental-model)

---

# 1. VoIP Quick Reference

| Component      | Function                                      |
| -------------- | --------------------------------------------- |
| **SIP**        | Call/session signaling                        |
| **SCCP**       | Cisco Skinny signaling protocol               |
| **SDP**        | Media/session negotiation                     |
| **RTP**        | Real-time audio/video media                   |
| **RTCP**       | RTP control and statistics                    |
| **SIP ALG**    | SIP-aware inspection and NAT/session handling |
| **HNT**        | Hosted NAT Traversal                          |
| **Pinhole**    | Dynamic related-media access                  |
| **MSRP**       | Message Session Relay Protocol                |
| **IPS Engine** | Protocol/security inspection                  |
| **LLDP-MED**   | Endpoint/network policy information           |

### Common SIP Ports

* [ ] UDP/5060 considered where applicable.
* [ ] TCP/5060 considered where applicable.
* [ ] TCP/5061 considered for SIP over TLS.
* [ ] RTP/RTCP ports verified from the actual VoIP platform.
* [ ] Do **not** assume a universal RTP port range.

---

# 2. SIP vs SCCP

## SIP

**SIP — Session Initiation Protocol**

Used to:

* [ ] Establish calls.
* [ ] Modify sessions.
* [ ] Terminate sessions.
* [ ] Exchange signaling.
* [ ] Carry/reference SDP information.

### Typical Flow

```text
REGISTER
   ↓
INVITE
   ↓
SDP Negotiation
   ↓
200 OK
   ↓
ACK
   ↓
RTP / RTCP
   ↓
BYE
```

## SCCP

**SCCP — Skinny Client Control Protocol**

Commonly associated with Cisco environments:

```text
Cisco IP Phone
      ↓
Cisco CUCM
      ↓
VoIP Infrastructure
```

### NSE Memory

* [ ] SIP = signaling.
* [ ] SCCP = signaling.
* [ ] RTP = media.
* [ ] RTCP = media control/statistics.
* [ ] SDP = media/session description.

---

# 3. FortiGate VoIP Architecture

```text
                    SIP Client
                        │
                        ▼
                ┌───────────────┐
                │   FortiGate   │
                │               │
                │ SIP ALG / HNT │
                │       │       │
                │       ▼       │
                │ VoIP Inspect  │
                │       │       │
                │       ▼       │
                │ IPS Engine    │
                └───────┬───────┘
                        │
                        ▼
                    SIP / PBX
                        │
                 ┌──────┴──────┐
                 ▼             ▼
                RTP           RTCP
```

### Architecture Checklist

* [ ] SIP signaling path identified.
* [ ] SIP server/PBX identified.
* [ ] FortiGate inspection point identified.
* [ ] NAT boundary identified.
* [ ] RTP media path identified.
* [ ] RTCP path identified.
* [ ] SIP ALG requirement evaluated.
* [ ] HNT requirement evaluated.
* [ ] IPS inspection requirement evaluated.

---

# 4. VoIP Feature Visibility

If VoIP configuration is not visible:

```text
System
  ↓
Feature Visibility
  ↓
VoIP
```

### Checklist

* [ ] Open **System → Feature Visibility**.
* [ ] Enable **VoIP**.
* [ ] Confirm VoIP-related configuration becomes available.
* [ ] Verify feature support against the target FortiOS version.

---

# 5. VoIP Profile Checklist

VoIP profiles provide VoIP-specific inspection and control.

### Possible Processing Models

```text
VoIP Profile
     │
     ├── Proxy
     │
     └── Flow
```

### Checklist

* [ ] VoIP profile created.
* [ ] SIP inspection configured.
* [ ] SIP ALG requirements evaluated.
* [ ] HNT requirements evaluated.
* [ ] RTP/RTCP behavior reviewed.
* [ ] MSRP requirements reviewed.
* [ ] Inspection mode selected according to the design.
* [ ] Resource impact evaluated.
* [ ] FortiOS version-specific syntax verified.

### Proxy vs Flow

| Mode      | Concept                     |
| --------- | --------------------------- |
| **Proxy** | Proxy-based SIP processing  |
| **Flow**  | IPS-engine-based inspection |

---

# 6. SIP ALG Checklist

**SIP ALG = SIP Application Layer Gateway**

### Functions

* [ ] SIP message inspection.
* [ ] SIP syntax validation.
* [ ] SIP request filtering.
* [ ] SIP request rate restriction.
* [ ] NAT traversal assistance.
* [ ] SIP address/port translation.
* [ ] RTP/RTCP handling.
* [ ] Dynamic related-session handling.

```text
SIP ALG
   │
   ├── Syntax Validation
   ├── Request Filtering
   ├── Rate Restriction
   ├── NAT Assistance
   ├── SIP Translation
   └── RTP/RTCP Handling
```

### Troubleshooting

* [ ] Verify whether SIP ALG is enabled/required.
* [ ] Verify whether ALG modifies SIP headers.
* [ ] Inspect SDP after NAT.
* [ ] Verify negotiated media ports.
* [ ] Check related RTP/RTCP sessions.

---

# 7. HNT Checklist

**HNT = Hosted NAT Traversal**

Typical scenario:

```text
SIP Phone
192.168.20.200
      │
      │ NAT
      ▼
  FortiGate
      │
      ▼
SIP Provider
```

### HNT Checklist

* [ ] SIP endpoint is behind NAT.
* [ ] NAT topology documented.
* [ ] External/public interface identified.
* [ ] HNT requirement evaluated.
* [ ] Source-IP restriction evaluated.
* [ ] SIP/SDP address translation tested.
* [ ] RTP/RTCP media path tested.

### External Interface

Example:

```cli
config system interface
    edit port1
        set external enable
    next
end
```

### HNT Configuration

```cli
config voip profile
    edit voip-test
        config sip
            set hosted-nat-traversal enable
            set hnt-restrict-source-ip enable
        end
    next
end
```

### Verify

* [ ] `hosted-nat-traversal` enabled when required.
* [ ] `hnt-restrict-source-ip` behavior understood.
* [ ] External interface correctly identified.

---

# 8. SIP VIP & NAT Checklist

Typical inbound architecture:

```text
Internet
   │
   ▼
Public SIP Address
   │
   ▼
FortiGate VIP
   │
   ▼
SIP Server / PBX
```

### VIP Checklist

* [ ] External SIP address identified.
* [ ] VIP configured.
* [ ] Internal PBX address verified.
* [ ] SIP service restricted.
* [ ] Source addresses restricted.
* [ ] NAT behavior understood.
* [ ] SIP ALG interaction tested.
* [ ] SDP translation tested.
* [ ] RTP/RTCP path tested.

### Important

There is **no universal "NAT = enable" or "NAT = disable" rule** for SIP.

Always evaluate:

```text
Topology
   ↓
VIP
   ↓
NAT
   ↓
SIP ALG
   ↓
SDP
   ↓
RTP / RTCP
```

---

# 9. SIP Firewall Policy Checklist

A production SIP policy should follow least privilege.

```text
Known SIP Provider IPs
          │
          ▼
       SIP VIP
          │
          ▼
 Restricted Service
          │
          ▼
     VoIP Profile
          │
          ▼
          IPS
```

### Policy Checklist

* [ ] Correct incoming interface.
* [ ] Correct source addresses.
* [ ] Known SIP provider IPs restricted where possible.
* [ ] SIP VIP configured.
* [ ] Required SIP services only.
* [ ] VoIP profile attached.
* [ ] IPS profile attached.
* [ ] SSL inspection evaluated when required.
* [ ] Logging enabled.
* [ ] Excessive Internet exposure avoided.
* [ ] `ALL → ALL` SIP policy avoided.

### ❌ Avoid

```text
Internet
   ↓
0.0.0.0/0
   ↓
ALL SIP
```

### ✅ Prefer

```text
Known SIP Sources
       ↓
    SIP VIP
       ↓
Restricted Service
       ↓
 VoIP Profile
       ↓
      IPS
```

---

# 10. SIP + SDP Checklist

A SIP message can contain SDP.

```text
SIP Message
    │
    ├── Start Line
    ├── Headers
    └── Message Body
             │
             └── SDP
```

### SDP Can Describe

* [ ] Media IP address.
* [ ] Media port.
* [ ] Media type.
* [ ] Codec.
* [ ] RTP parameters.

Example:

```text
m=audio 49170 RTP/AVP 0
a=rtpmap:0 PCMU/8000
```

### Troubleshooting Checklist

If signaling works but media fails:

* [ ] Inspect SDP source IP.
* [ ] Inspect SDP destination/media IP.
* [ ] Inspect media port.
* [ ] Check NAT translation.
* [ ] Check RTP.
* [ ] Check RTCP.
* [ ] Check dynamic expectations/pinholes.

### Critical Mental Model

```text
SIP
 ↓
SDP
 ↓
NAT
 ↓
RTP / RTCP
```

---

# 11. RTP & RTCP Checklist

## RTP

**RTP = Real-time Transport Protocol**

Carries:

* [ ] Voice.
* [ ] Audio.
* [ ] Video.
* [ ] Real-time media.

## RTCP

**RTCP = Real-time Transport Control Protocol**

Provides:

* [ ] Control information.
* [ ] Statistics.
* [ ] Quality-related information.

### Remember

```text
SIP  → Signaling
SDP  → Media Negotiation
RTP  → Media
RTCP → Control / Statistics
```

### Media Troubleshooting

* [ ] RTP packets observed.
* [ ] Correct source/destination IP.
* [ ] Correct UDP port.
* [ ] NAT translation verified.
* [ ] RTCP traffic considered.
* [ ] Firewall sessions verified.
* [ ] Expectations/pinholes verified.

---

# 12. SIP Pinhole Checklist

SIP signaling can dynamically provide information required for media sessions.

```text
SIP
 ↓
SDP
 ↓
Media IP / Port
 ↓
Dynamic Expectation / Pinhole
 ↓
RTP / RTCP
```

### Checklist

* [ ] SIP signaling successfully established.
* [ ] SDP inspected.
* [ ] Media IP identified.
* [ ] Media port identified.
* [ ] Dynamic expectation created where required.
* [ ] RTP session established.
* [ ] RTCP session considered.
* [ ] Pinhole behavior verified.

### NSE Trap

> **SIP pinhole restriction is associated with SIP ALG in proxy mode.**

---

# 13. SIP over TLS Checklist

Common SIP TLS deployment:

```text
SIP over TLS
      ↓
TCP/5061
```

### Checklist

* [ ] SIP TLS requirement identified.
* [ ] TCP/5061 considered where applicable.
* [ ] Certificate requirements reviewed.
* [ ] SSL inspection requirement evaluated.
* [ ] VoIP compatibility tested.
* [ ] CPU/resource impact evaluated.
* [ ] SIP signaling visibility verified.

Example configuration concept:

```cli
config voip profile
    edit voip-test
        set feature-set proxy
        set ssl-mode full
    next
end
```

Verify:

```cli
get | grep ssl
```

### Warning

Do not enable deep SSL inspection on production VoIP traffic without validating:

* [ ] Certificates.
* [ ] Endpoint compatibility.
* [ ] SIP behavior.
* [ ] Latency.
* [ ] CPU usage.
* [ ] Memory usage.

---

# 14. Flow vs Proxy Inspection

## Flow-Based

```text
VoIP Traffic
     ↓
IPS Engine
     ↓
SIP Decoder
     ↓
Protocol Analysis
     ↓
IPS Signatures
```

### Checklist

* [ ] IPS engine available.
* [ ] SIP inspection supported in the deployed version.
* [ ] IPS profile applied.
* [ ] SIP decoder behavior verified.

## Proxy-Based

```text
SIP Client
    ↓
FortiGate SIP Proxy
    ↓
SIP Inspection
    ↓
SIP ALG / HNT
    ↓
SIP Server
```

### Checklist

* [ ] Proxy feature required?
* [ ] SIP ALG required?
* [ ] HNT required?
* [ ] Pinhole restriction required?
* [ ] Resource capacity validated?

### Key Difference

| Inspection | Main Processing        |
| ---------- | ---------------------- |
| **Flow**   | IPS engine             |
| **Proxy**  | Proxy-based processing |

---

# 15. MSRP Inspection Checklist

**MSRP = Message Session Relay Protocol**

Conceptually:

```text
MSRP
 ↓
IPS Engine
 ↓
MSRP Decoder
 ↓
Security Inspection
```

### Checklist

* [ ] MSRP requirement identified.
* [ ] Flow inspection selected where required.
* [ ] MSRP inspection enabled.
* [ ] Violation logging configured.
* [ ] Message size policy defined.
* [ ] Block action reviewed.
* [ ] IPS profile applied.

Example:

```cli
config voip profile
    edit voip-test
        set feature-set flow

        config msrp
            set status enable
            set log-violations enable
            set max-msg-size 500
            set max-msg-size-action block
        end
    next
end
```

### Processing Model

```text
MSRP
 ↓
MSRP Decoder
 ↓
Message Size Check
 ↓
IPS
 ↓
Allow / Log / Block
```

---

# 16. SIP DoS Protection

Potential SIP threats include:

* [ ] SIP flooding.
* [ ] REGISTER flooding.
* [ ] INVITE flooding.
* [ ] SIP scanning.
* [ ] Malformed SIP.
* [ ] Authentication abuse.
* [ ] Resource exhaustion.
* [ ] Spoofed registrations.

### Security Controls

```text
Syntax Validation
       ↓
Request Filtering
       ↓
Rate Restriction
       ↓
Session Handling
       ↓
NAT Traversal
```

### Checklist

* [ ] SIP sources restricted.
* [ ] Unnecessary SIP exposure removed.
* [ ] SIP rate controls evaluated.
* [ ] SIP request filtering evaluated.
* [ ] IPS enabled where appropriate.
* [ ] Logs monitored.
* [ ] Abnormal registration activity monitored.
* [ ] Provider source ranges documented.

---

# 17. LLDP-MED & Voice VLAN

**LLDP-MED** can provide network policy information to media endpoints.

```text
IP Phone
   ↓
Switch
   ↓
Voice VLAN
   ↓
FortiGate
```

### Voice VLAN Checklist

* [ ] Voice VLAN created.
* [ ] Voice subnet configured.
* [ ] DHCP scope configured.
* [ ] Switch voice VLAN configured.
* [ ] LLDP reception enabled where required.
* [ ] LLDP transmission enabled where required.
* [ ] LLDP network policy configured.
* [ ] IP phone receives expected VLAN/IP.
* [ ] VLAN tagging verified.

### Example VLAN

```cli
config system interface
    edit vlan-100
        set vdom root
        set ip 192.168.101.1 255.255.255.0
        set device-identification enable
        set role lan
        set snmp-index 25
        set interface port3
        set vlanid 100
    next
end
```

### LLDP Network Policy

```cli
config system lldp network-policy
    edit 1
        config voice
            set status enable
            set tag dot1q
            set vlan 100
        end
    next
end
```

### Apply to Interface

```cli
config system interface
    edit port3
        set vdom root
        set type physical
        set lldp-reception enable
        set lldp-transmission enable
        set lldp-network-policy 1
    next
end
```

---

# 18. SIP Troubleshooting Checklist

## Phase 1 — Policy

* [ ] Correct firewall policy matched.
* [ ] Source IP correct.
* [ ] Destination VIP correct.
* [ ] SIP service allowed.
* [ ] VoIP profile attached.
* [ ] IPS profile attached.
* [ ] Logging enabled.

## Phase 2 — SIP

* [ ] SIP packet reaches FortiGate.
* [ ] REGISTER observed.
* [ ] INVITE observed.
* [ ] Response codes checked.
* [ ] SIP headers inspected.
* [ ] SIP ALG behavior checked.

## Phase 3 — SDP

* [ ] SDP present.
* [ ] Media IP checked.
* [ ] Media port checked.
* [ ] Codec negotiation checked.
* [ ] NAT translation checked.

## Phase 4 — Media

* [ ] RTP observed.
* [ ] RTCP considered.
* [ ] Correct UDP ports verified.
* [ ] Session table checked.
* [ ] Expectations checked.
* [ ] Pinhole behavior checked.

---

# 19. No Audio / One-Way Audio

This is one of the most important VoIP troubleshooting scenarios.

```text
SIP Works
   ↓
Call Established
   ↓
No Audio / One-Way Audio
   ↓
Inspect SDP
   ↓
Check NAT
   ↓
Check RTP Ports
   ↓
Check RTCP
   ↓
Check Expectations / Pinholes
```

### Checklist

* [ ] SIP signaling works.
* [ ] Call establishment succeeds.
* [ ] SDP inspected.
* [ ] SDP media IP correct.
* [ ] SDP media port correct.
* [ ] NAT translation correct.
* [ ] RTP packets observed.
* [ ] RTP return path verified.
* [ ] RTCP considered.
* [ ] Dynamic expectation exists.
* [ ] SIP ALG behavior verified.
* [ ] HNT behavior verified.
* [ ] VIP configuration verified.

### Fast Diagnostic Rule

```text
REGISTER ✓
INVITE ✓
CALL ✓
AUDIO ✗

       ↓

Do NOT start with SIP registration.

Start with:

SDP
 ↓
NAT
 ↓
RTP
 ↓
RTCP
 ↓
Expectations / Pinholes
```

---

# 20. Registration Failure

### Checklist

* [ ] SIP port reachable.
* [ ] Correct firewall policy.
* [ ] Correct VIP.
* [ ] SIP service allowed.
* [ ] SIP ALG checked.
* [ ] HNT checked.
* [ ] Source-IP restriction checked.
* [ ] NAT checked.
* [ ] SIP headers checked.
* [ ] SIP server reachable.
* [ ] Provider reachability checked.
* [ ] SIP logs reviewed.

### Diagnostic Logic

```text
REGISTER Failure
      ↓
Firewall Policy
      ↓
VIP
      ↓
SIP Service
      ↓
SIP ALG / HNT
      ↓
NAT
      ↓
SIP Server / Provider
```

---

# 21. Diagnostic Commands

## SIP Proxy

```cli
diagnose sys sip-proxy calls
```

Use for:

* [ ] SIP registrations.
* [ ] Active calls.
* [ ] SIP sessions.
* [ ] Call negotiation.
* [ ] SIP proxy troubleshooting.

## Session Table

```cli
diagnose sys session list
```

Check:

* [ ] SIP sessions.
* [ ] RTP sessions.
* [ ] RTCP sessions.
* [ ] Related sessions.

## Session Expectations

```cli
diagnose sys session list expectation
```

Check:

* [ ] Related peers.
* [ ] Dynamic ports.
* [ ] RTP expectations.
* [ ] RTCP expectations.

---

# 22. Production Security Checklist

## Network

* [ ] VoIP topology documented.
* [ ] SIP provider networks documented.
* [ ] PBX addresses documented.
* [ ] Media networks documented.
* [ ] NAT boundaries documented.
* [ ] Voice VLAN documented.

## Firewall

* [ ] Least-privilege policy implemented.
* [ ] SIP sources restricted.
* [ ] VIP restricted.
* [ ] Required services only.
* [ ] VoIP profile attached.
* [ ] IPS profile attached.
* [ ] Logging enabled.

## SIP

* [ ] SIP ALG requirement evaluated.
* [ ] HNT requirement evaluated.
* [ ] SIP pinholes reviewed.
* [ ] SIP TLS evaluated.
* [ ] SIP DoS controls reviewed.

## Media

* [ ] SDP verified.
* [ ] RTP verified.
* [ ] RTCP verified.
* [ ] Media port range documented.
* [ ] Dynamic expectations verified.

## Operations

* [ ] SIP logs monitored.
* [ ] Failed registrations monitored.
* [ ] One-way audio troubleshooting procedure documented.
* [ ] FortiOS version compatibility verified.
* [ ] Configuration backup available.

---

# 23. Common VoIP Mistakes

## ❌ Mistake 1 — Exposing SIP to Everyone

```text
0.0.0.0/0
   ↓
SIP
```

### Better

```text
Known Provider IPs
       ↓
     SIP VIP
       ↓
Restricted Policy
```

---

## ❌ Mistake 2 — Forgetting RTP

```text
SIP ✓
RTP ✗
```

Result:

```text
Call Established
      ↓
No Audio
```

---

## ❌ Mistake 3 — Ignoring SDP

SDP can contain:

```text
Media IP
Media Port
Codec
Media Type
```

---

## ❌ Mistake 4 — Treating SIP as the Entire Call

```text
SIP ≠ Media
```

Remember:

```text
SIP  → Signaling
SDP  → Negotiation
RTP  → Media
RTCP → Control
```

---

## ❌ Mistake 5 — Ignoring NAT

```text
Private SIP/SDP Address
          ↓
        NAT
          ↓
     Public Network
```

Always inspect the actual translated SIP/SDP information.

---

## ❌ Mistake 6 — Enabling SSL Inspection Without Testing

Potential impact:

* [ ] CPU.
* [ ] Memory.
* [ ] Certificates.
* [ ] Latency.
* [ ] Endpoint compatibility.
* [ ] SIP behavior.

---

# 24. NSE4/NSE7 Exam Traps

### 🧠 Trap #1 — SIP vs RTP

```text
SIP → Signaling
RTP → Media
```

---

### 🧠 Trap #2 — SDP

```text
SDP
 ↓
Media IP
Media Port
Codec
Media Type
```

---

### 🧠 Trap #3 — HNT

```text
HNT
 ↓
Hosted NAT Traversal
```

Think **NAT + SIP endpoints**.

---

### 🧠 Trap #4 — Flow Inspection

```text
Flow
 ↓
IPS Engine
```

---

### 🧠 Trap #5 — Proxy Inspection

```text
Proxy
 ↓
Proxy-Based SIP Processing
```

---

### 🧠 Trap #6 — SIP Pinhole Restriction

> SIP pinhole restriction is associated with **SIP ALG in proxy mode**.

---

### 🧠 Trap #7 — RTP/RTCP

```text
RTP
 ↓
Media

RTCP
 ↓
Control / Statistics
```

---

### 🧠 Trap #8 — SIP TLS

```text
SIP over TLS
 ↓
TCP/5061
```

---

### 🧠 Trap #9 — One-Way Audio

Do not stop at SIP.

```text
SIP
 ↓
SDP
 ↓
NAT
 ↓
RTP / RTCP
 ↓
Expectations
```

---

### 🧠 Trap #10 — Voice VLAN

```text
LLDP-MED
 ↓
Network Policy
 ↓
Voice VLAN
```

---

# 25. One-Minute Revision

```text
                    FORTIGATE VOIP
                          │
             ┌────────────┴────────────┐
             ▼                         ▼
         SIGNALING                   MEDIA
             │                         │
            SIP                       RTP
             │                         │
            SDP                      RTCP
             │                         │
             ▼                         ▼
        SIP ALG / HNT             Expectations
             │                         │
             ▼                         ▼
       NAT Traversal               Pinholes
             │                         │
             └────────────┬────────────┘
                          ▼
                    VOIP SECURITY
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
            IPS          DoS        Logging
```

### Quick Memory Table

| Topic      | Remember                    |
| ---------- | --------------------------- |
| SIP        | Signaling                   |
| SCCP       | Cisco Skinny signaling      |
| SDP        | Media negotiation           |
| RTP        | Media                       |
| RTCP       | Control/statistics          |
| SIP ALG    | SIP-aware processing        |
| HNT        | NAT traversal               |
| Pinhole    | Dynamic media access        |
| Flow       | IPS engine                  |
| Proxy      | Proxy-based processing      |
| MSRP       | Session-based messaging     |
| LLDP-MED   | Network policy              |
| Voice VLAN | Voice endpoint segmentation |
| SIP TLS    | Commonly TCP/5061           |
| No Audio   | SDP → NAT → RTP/RTCP        |

---

# 26. Final VoIP Mental Model

```text
                         SIP CALL
                            │
                            ▼
                     SIP Signaling
                            │
                            ▼
                      SIP ALG / HNT
                            │
                            ▼
                         SIP + SDP
                            │
                            ▼
                    Media Negotiation
                            │
                  ┌─────────┴─────────┐
                  ▼                   ▼
                 RTP                 RTCP
               Media             Control/Stats
                  │                   │
                  └─────────┬─────────┘
                            ▼
                    Expectations /
                       Pinholes
                            │
                            ▼
                       Active Call
```

## 🔥 Ultimate Troubleshooting Mental Model

```text
                VOIP FAILURE
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
   SIGNALLING FAILURE      MEDIA FAILURE
          │                     │
          ▼                     ▼
       SIP/VIP              SDP/NAT
          │                     │
          ▼                     ▼
     ALG / HNT              RTP / RTCP
          │                     │
          ▼                     ▼
     SIP Server           Expectations
                                │
                                ▼
                             Pinholes
```

> **SheynShield Engineering Rule:**
> **“SIP is working” does not mean “VoIP is working.”**
>
> For media problems, follow:
>
> **SIP → SDP → NAT → RTP/RTCP → Expectations/Pinholes**

---

# 🎯 30-Second NSE / Interview Summary

```text
SIP
 ↓
Signaling

SDP
 ↓
Media Negotiation

SIP ALG
 ↓
SIP-Aware Inspection
 + NAT Assistance
 + Session Handling

HNT
 ↓
Hosted NAT Traversal

RTP
 ↓
Voice / Video Media

RTCP
 ↓
Control / Statistics

Pinhole
 ↓
Dynamic Media Access

Flow
 ↓
IPS Engine

Proxy
 ↓
Proxy-Based SIP Processing

MSRP
 ↓
Messaging / Session Data

LLDP-MED
 ↓
Voice Network Policy
```

---

## 🏷️ Topics

```text
fortigate
fortios
voip
sip
sip-alg
hnt
sip-proxy
rtp
rtcp
sdp
sip-pinhole
msrp
lldp-med
voice-vlan
ips
network-security
cybersecurity
nse4
nse7
fortinet
```

---

## 🔗 Topics

* SIP ALG
* Hosted NAT Traversal (HNT)
* SIP Proxy
* RTP / RTCP
* SDP
* SIP Pinholes
* VoIP IPS Inspection
* MSRP Inspection
* LLDP-MED
* Voice VLAN
* SIP over TLS
* FortiGate NAT
* VIP
* VoIP Troubleshooting

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

> **SheynShield | Engineering Secure Networks**
>
> **FortiGate VoIP Security • SIP ALG • HNT • RTP/RTCP • SIP Proxy • IPS • Network Security**

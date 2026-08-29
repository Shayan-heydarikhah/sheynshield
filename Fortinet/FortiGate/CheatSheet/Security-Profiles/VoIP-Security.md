# FortiGate VoIP Security & SIP 

> **FortiOS Focus:** VoIP Security, SIP ALG, HNT, RTP/RTCP, SIP Pinholes, SIP Proxy, MSRP, LLDP-MED
> **Audience:** FortiGate / NSE4 / NSE7 / Network & Security Engineers
> **Core Protocols:** SIP, SDP, RTP, RTCP, SCCP, MSRP
> **Security Stack:** SIP ALG + VoIP Profile + IPS Engine + Proxy/Flow Inspection

---

## 📌 Quick Reference

| Component      | Purpose                                                                   |
| -------------- | ------------------------------------------------------------------------- |
| **SIP**        | Signaling protocol used to establish, modify, and terminate VoIP sessions |
| **SCCP**       | Cisco Skinny signaling protocol, commonly associated with Cisco CUCM      |
| **SDP**        | Describes media/session parameters carried within SIP                     |
| **RTP**        | Carries real-time voice/video media                                       |
| **RTCP**       | Provides control, statistics, and quality information for RTP             |
| **SIP ALG**    | Inspects and controls SIP signaling and assists with NAT traversal        |
| **HNT**        | Hosted NAT Traversal support for SIP environments                         |
| **Pinhole**    | Dynamically permits negotiated RTP/RTCP traffic                           |
| **MSRP**       | Message Session Relay Protocol for session-based messaging                |
| **IPS Engine** | Detects malicious payloads and supported VoIP attacks                     |

---

# 1. SIP vs SCCP

## SIP — Session Initiation Protocol

SIP is a signaling protocol used to:

* Establish calls
* Modify sessions
* Terminate sessions
* Exchange signaling information
* Carry or reference SDP media information

Common SIP deployments:

```text
UDP/5060
TCP/5060
TLS/5061
```

### Basic SIP Flow

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

---

## SCCP — Skinny Client Control Protocol

SCCP is a Cisco proprietary signaling protocol commonly associated with:

```text
Cisco IP Phone
      ↓
Cisco CUCM
      ↓
VoIP Infrastructure
```

> **NSE Note:** SIP and SCCP are signaling protocols. RTP is responsible for carrying the actual real-time media.

---

# 2. FortiGate VoIP Security Architecture

Simplified architecture:

```text
                  SIP Client
                      │
                      ▼
              ┌───────────────┐
              │   FortiGate   │
              │               │
              │  SIP ALG/HNT  │
              │       │       │
              │       ▼       │
              │ SIP Inspection│
              │       │       │
              │       ▼       │
              │  IPS Engine   │
              └───────┬───────┘
                      │
                      ▼
                 SIP / PBX
                      │
             ┌────────┴────────┐
             ▼                 ▼
            RTP               RTCP
```

FortiGate can inspect SIP signaling and use information learned from SIP/SDP to handle related RTP/RTCP sessions.

---

# 3. Enable VoIP Feature Visibility

If VoIP configuration options are hidden:

```text
System
  └── Feature Visibility
        └── VoIP
```

Enable:

```text
VoIP
```

---

# 4. VoIP Profiles

VoIP profiles provide VoIP-specific inspection and control.

They can be used for:

* SIP inspection
* SIP ALG
* HNT
* RTP/RTCP handling
* MSRP inspection

Common inspection modes:

```text
Default
Strict
Proxy
Flow
```

### Proxy vs Flow

```text
Proxy
  ↓
Dedicated proxy-based processing
  ↓
Deeper SIP-aware handling
  ↓
Higher resource requirements

Flow
  ↓
IPS Engine
  ↓
Lower additional processing overhead
```

> **Important:** Exact features and behavior depend on the FortiOS release. Always verify the syntax and feature support for the deployed version.

---

# 5. SIP ALG

## What Is SIP ALG?

**SIP Application Layer Gateway (ALG)** provides SIP-aware inspection and handling.

It can provide capabilities such as:

* SIP message inspection
* SIP syntax validation
* SIP request filtering
* SIP request rate restriction
* NAT traversal assistance
* SIP address/port translation
* RTP/RTCP session handling
* Dynamic pinholes/expectations

### SIP ALG Security Functions

```text
SIP ALG
   │
   ├── Syntax Validation
   ├── Request Filtering
   ├── Rate Restriction
   ├── NAT Assistance
   ├── RTP/RTCP Handling
   └── SIP Security Controls
```

---

# 6. Hosted NAT Traversal — HNT

**Hosted NAT Traversal (HNT)** helps FortiGate handle SIP devices located behind NAT.

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
Internet / SIP Provider
```

Some SIP endpoints do not correctly rewrite private addressing information inside SIP/SDP messages when crossing NAT.

HNT allows FortiGate to better handle SIP traffic in NAT traversal scenarios.

---

## 6.1 Mark the External Interface

Example:

```cli
config system interface
    edit port1
        set external enable
    next
end
```

Conceptually:

```text
Internet / Public Side
        │
        ▼
     port1
   external
```

> Marking an interface as external is useful when FortiGate needs to identify the external/public side of the SIP topology.

---

## 6.2 Enable HNT in the VoIP Profile

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

### Important Settings

```text
hosted-nat-traversal enable
```

Enables HNT.

```text
hnt-restrict-source-ip enable
```

Restricts SIP handling based on the source IP information observed by the SIP processing mechanism.

---

# 7. SIP VIP + HNT

Typical inbound SIP architecture:

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
              192.168.20.200
```

Example:

```text
External IP:
192.168.101.101

Mapped IP:
192.168.20.200
```

> The addresses above are documentation examples. A production deployment normally uses an actual public address for the external side.

---

# 8. VoIP Firewall Policy

A controlled SIP policy should combine:

```text
Firewall Policy
      │
      ├── Restricted Source
      ├── SIP VIP
      ├── Required SIP Services
      ├── Proxy / Flow Inspection
      ├── VoIP Profile
      ├── IPS Profile
      └── Logging
```

Example conceptual policy:

```text
Source       : Known SIP sources / required networks
Destination  : SIP VIP
Service      : Required SIP services
Inspection   : Proxy or Flow
VoIP Profile : voip-test
IPS Profile  : Enabled
NAT          : According to topology
```

### Security Recommendation

> **Never expose unrestricted SIP services to the Internet unless there is a specific design requirement.**

Prefer:

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

---

# 9. VIP + NAT + HNT

One of the most important VoIP design questions:

> **Should NAT be enabled when using SIP VIP/HNT?**

There is no universal yes/no answer. It depends on the topology and how the SIP endpoints, PBX, VIP, and media paths are designed.

When NAT/VIP processing is involved, SIP ALG can assist with SIP-related address and port translation and with related RTP/RTCP handling.

Conceptually:

```text
SIP Signaling
      │
      ▼
    SIP ALG
      │
      ├── SIP Translation
      │
      ├── SDP Handling
      │
      ├── RTP Expectations
      │
      └── RTCP Expectations
```

When troubleshooting NATed SIP, check:

```text
[ ] SIP signaling
[ ] SIP headers
[ ] SDP address
[ ] SDP media port
[ ] VIP
[ ] NAT
[ ] HNT
[ ] RTP
[ ] RTCP
[ ] Session expectations
```

---

# 10. RTP vs RTCP

## RTP — Real-time Transport Protocol

RTP carries the actual real-time media:

```text
Voice
Audio
Video
```

```text
SIP  → Signaling
RTP  → Media
```

---

## RTCP — Real-time Transport Control Protocol

RTCP provides control and statistical information associated with RTP sessions.

```text
RTP
 │
 └── Media

RTCP
 │
 └── Control / Statistics / Quality Information
```

### Important

```text
SIP ≠ RTP

SIP  → Establishes/manages the session
RTP  → Carries media
RTCP → Provides RTP control/statistics
```

---

## Example RTP Range

A commonly encountered UDP range is:

```text
UDP/16384-32768
```

> The actual media port range depends on the VoIP platform and its configuration. Do not assume this range is universal.

---

# 11. SIP Pinholes

A VoIP call requires more than the initial SIP signaling connection.

Typical sequence:

```text
SIP Signaling
      │
      ▼
      SDP
      │
      ▼
Media Negotiation
      │
      ├──────────────┐
      ▼              ▼
     RTP            RTCP
```

FortiGate can dynamically create related sessions/expectations for negotiated media traffic.

Conceptually:

```text
SIP
 │
 └── SDP
      │
      ▼
RTP/RTCP Information
      │
      ▼
Dynamic Pinhole / Expectation
      │
      ▼
Media Session
```

### SIP Pinhole Restriction

> **SIP pinhole restriction is supported by the SIP ALG in proxy mode.**

---

# 12. RTP/RTCP NAT Port Range

The `nat-port-range` setting can be used to specify the NAT port range for RTP/RTCP traffic handled by the SIP ALG.

Example:

```cli
config voip profile
    edit voip-test
        config sip
            set nat-port-range 5117-65535
        end
    next
end
```

Conceptually:

```text
Configured NAT Port Range
          │
          ▼
     RTP / RTCP
          │
          ▼
   SIP ALG Handling
```

### RTP / RTCP Port Convention

Traditionally:

```text
RTP  → Odd port
RTCP → Associated even port
```

> Modern implementations may not strictly follow the odd/even convention. Always validate the actual negotiated ports.

---

# 13. SIP over TLS

Encrypted SIP signaling commonly uses:

```text
SIP over TLS
      │
      ▼
TCP/5061
```

When FortiGate needs to inspect encrypted SIP traffic, SSL inspection may be required depending on the topology and FortiOS feature support.

Example:

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

### Resource Consideration

```text
SSL Inspection
      │
      ▼
More Visibility
      │
      ▼
More Processing
      │
      ▼
Potential CPU / Memory Impact
```

> Full SSL inspection of VoIP traffic should be tested carefully for certificate, compatibility, latency, and resource-impact considerations.

---

# 14. SIP Message Anatomy

A SIP message begins with a start line.

Example:

```text
INVITE sip:bob@example.com SIP/2.0
```

---

## SIP Headers

```text
To: Bob <sip:bob@biloxi.com>

From: sip:alice@atlanta.com;tag=4711

Subject: Congratulations!

Content-Length: 177

Content-Type: application/sdp

Call-ID: af1234@pc33.atlanta.com

CSeq: 1 INVITE

Max-Forwards: 70

Contact: sip:alice@pc33.atlanta.com:5066;transport=udp

Via: SIP/2.0/UDP pc33.atlanta.com;branch=z9hG4bK776as
```

Structure:

```text
SIP Message
    │
    ├── Start Line
    │
    ├── Message Headers
    │
    └── Message Body
             │
             └── SDP
```

---

# 15. SDP Inside SIP

SIP messages can contain **SDP — Session Description Protocol** information.

Example:

```text
v=0

o=alice 2345566342 2346553445 IN IP4 pc33.atlanta.com

s=

c=IN IP4 pc33.atlanta.com

t=0 0

m=audio 49170 RTP/AVP 0

a=rtpmap:0 PCMU/8000
```

SDP can communicate:

* Media IP address
* Media port
* Media type
* Codec
* RTP parameters

Simplified:

```text
SIP
 │
 └── SDP
      ├── Media IP
      ├── Media Port
      ├── Media Type
      └── Codec
```

### Troubleshooting Rule

> If SIP signaling succeeds but there is no audio, inspect the **SDP → NAT → RTP/RTCP** path.

---

# 16. SIP Proxy Diagnostics

Check SIP proxy calls:

```cli
diagnose sys sip-proxy calls
```

Useful when investigating:

```text
SIP registrations
Active calls
SIP sessions
Call negotiation
RTP/RTCP handling
```

---

## Session Table

```cli
diagnose sys session list
```

Look for:

```text
SIP sessions
RTP sessions
RTCP sessions
Expectations
```

---

## Session Expectations

```cli
diagnose sys session list expectation
```

Useful for examining:

```text
Peers
Dynamic ports
Related sessions
RTP expectations
RTCP expectations
```

---

# 17. Flow-Based SIP Inspection

In flow-based inspection, SIP inspection can be performed by the **IPS engine**.

Architecture:

```text
VoIP Traffic
      │
      ▼
  IPS Engine
      │
      ├── SIP Decoder
      │
      ├── Protocol Analysis
      │
      └── IPS Signatures
```

### Resource Advantage

When SIP inspection is performed through the IPS engine, flow-based inspection can reduce the additional resource overhead associated with proxy-based processing.

```text
Flow Inspection
      │
      ▼
IPS Engine
      │
      ▼
Protocol Inspection
```

> Exact feature support and behavior are FortiOS-version dependent.

---

# 18. Proxy-Based SIP Inspection

Proxy mode provides proxy-based processing for SIP traffic.

Conceptually:

```text
SIP Client
    │
    ▼
FortiGate SIP Proxy
    │
    ├── SIP Inspection
    ├── SIP ALG
    ├── HNT
    └── Pinhole Handling
    │
    ▼
SIP Server / PBX
```

Proxy mode is particularly important when features requiring proxy-based SIP processing are used.

---

# 19. VoIP Security Profile

Conceptually:

```text
Security Profiles
      │
      └── VoIP
            │
            ├── SIP
            ├── HNT
            ├── MSRP
            └── Protocol-Specific Controls
```

Some profile parameters may use:

```text
0 = Automatic / Default Behavior
```

> Always verify the exact meaning of a value in the FortiOS version being deployed.

---

# 20. Voice VLAN with LLDP-MED

**LLDP-MED** can provide network policy information for voice and other media endpoints.

Conceptual architecture:

```text
IP Phone
    │
    │ LLDP-MED
    ▼
  Switch
    │
    │ Voice VLAN
    ▼
FortiGate
```

LLDP-MED network policies can be used for:

* Voice
* Voice signaling
* Guest
* Guest voice signaling
* Softphone
* Video conferencing
* Streaming video
* Video signaling

---

# 21. Voice VLAN Configuration

## Create VLAN Interface

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

---

## Configure LLDP Network Policy

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

---

## Apply LLDP Policy to Interface

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

# 22. Voice VLAN Verification

### Step 1 — Connect the IP Phone

```text
IP Phone
   ↓
Switch
   ↓
FortiGate
```

### Step 2 — Check Phone IP

The phone should receive an address from the expected voice VLAN.

```text
Phone IP
   ↓
Voice VLAN Subnet
```

### Step 3 — Capture Traffic

Capture traffic on the FortiGate incoming interface and verify:

```text
802.1Q VLAN Tag
       │
       ▼
Voice VLAN ID
       │
       ▼
Expected VLAN
```

---

# 23. MSRP Inspection

**MSRP — Message Session Relay Protocol**

MSRP can transport session-based messaging data.

FortiGate can inspect MSRP traffic using the IPS engine and protocol decoding.

Architecture:

```text
MSRP Traffic
     │
     ▼
 IPS Engine
     │
     ▼
MSRP Decoder
     │
     ▼
IPS Signatures
     │
     ├── Detect
     ├── Log
     └── Block
```

---

# 24. MSRP Flow Inspection

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

### Configuration Logic

```text
MSRP
 │
 ▼
Flow Inspection
 │
 ▼
MSRP Decoder
 │
 ▼
Message Size Check
 │
 ├── ≤ 500 bytes → Allow
 │
 └── > 500 bytes → Block
```

> The exact interpretation of `max-msg-size` and special values such as `0` should be confirmed against the FortiOS version in use.

---

# 25. IPS + VoIP Payload Inspection

For flow-based MSRP inspection:

```text
VoIP Profile
      +
IPS Profile
      +
Flow Inspection
```

Processing model:

```text
MSRP
 │
 ▼
VoIP / MSRP Decoder
 │
 ▼
IPS Engine
 │
 ▼
Signature Matching
 │
 ├── Allow
 ├── Log
 └── Block
```

### Security Concept

> VoIP traffic is not automatically trusted just because it is voice traffic. Supported application payloads can still be inspected by security engines.

---

# 26. SIP DoS Protection

SIP infrastructure can be exposed to attacks such as:

* SIP flooding
* REGISTER flooding
* INVITE flooding
* Malformed SIP messages
* SIP scanning
* Authentication abuse
* Resource exhaustion
* Spoofed registrations

SIP-aware controls can provide mechanisms such as:

```text
Syntax Validation
       │
       ▼
Request Filtering
       │
       ▼
Rate Restriction
       │
       ▼
Session Handling
       │
       ▼
NAT Traversal
```

---

# 27. VoIP Attack Surface

```text
                    VoIP
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
      SIP           RTP           MSRP
       │             │             │
  Signaling        Media        Messaging
       │             │             │
       ▼             ▼             ▼
   Flooding        DoS         Malicious
   Spoofing      Injection      Payload
   Scanning      Hijacking      Exploits
```

---

# 28. Recommended SIP Security Policy

A production-oriented SIP policy should include:

```text
Firewall Policy
│
├── Source Restriction
├── Destination VIP
├── SIP Service Restriction
├── Proxy / Flow Inspection
├── VoIP Profile
├── IPS Profile
├── SSL Inspection if Required
└── Logging
```

### ❌ Avoid

```text
Internet
   │
   ▼
ALL → ALL
   │
   ▼
SIP
```

### ✅ Prefer

```text
Known SIP Provider IPs
        │
        ▼
      SIP VIP
        │
        ▼
 Restricted SIP Service
        │
        ▼
    VoIP Profile
        │
        ▼
       IPS
```

---

# 29. 🔍 Troubleshooting Checklist

## SIP Registration Failure

Check:

```text
[ ] SIP port reachable
[ ] VIP configuration
[ ] Firewall policy
[ ] SIP ALG
[ ] HNT
[ ] Source-IP restriction
[ ] NAT
[ ] SIP headers
[ ] SIP server
[ ] SIP logs
```

---

## Call Establishes but No Audio

Check:

```text
[ ] SIP signaling
[ ] SDP address
[ ] SDP media port
[ ] NAT
[ ] VIP
[ ] RTP
[ ] RTCP
[ ] SIP ALG
[ ] HNT
[ ] Session expectations
[ ] Firewall policy
```

Troubleshooting flow:

```text
SIP Works
   │
   ▼
Call Established
   │
   ▼
No Audio / One-Way Audio
   │
   ▼
Inspect SDP
   │
   ▼
Check NAT
   │
   ▼
Check RTP Ports
   │
   ▼
Check RTCP
   │
   ▼
Check Expectations / Pinholes
```

---

# 30. 🔥 Fast NSE Exam Notes

| Topic                       | Key Point                                     |
| --------------------------- | --------------------------------------------- |
| **SIP**                     | Signaling protocol                            |
| **SCCP**                    | Cisco Skinny signaling protocol               |
| **SDP**                     | Describes media/session parameters            |
| **RTP**                     | Carries real-time media                       |
| **RTCP**                    | RTP control/statistics                        |
| **SIP ALG**                 | SIP-aware inspection and NAT/session handling |
| **HNT**                     | Hosted NAT Traversal                          |
| **Pinhole**                 | Dynamically permits related media traffic     |
| **SIP Pinhole Restriction** | Requires SIP ALG and proxy-based processing   |
| **Flow SIP Inspection**     | Uses IPS engine                               |
| **MSRP Inspection**         | Uses protocol decoding/IPS inspection         |
| **Voice VLAN**              | Can use LLDP/LLDP-MED network policy          |
| **SIP TLS**                 | Commonly TCP/5061                             |
| **RTP**                     | Media traffic                                 |
| **VIP**                     | Common for inbound SIP publishing             |
| **One-Way Audio**           | Check SDP → NAT → RTP/RTCP → Expectations     |

---

# 31. 🧠 Most Important VoIP Mental Model

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
                ┌───────┴───────┐
                ▼               ▼
               RTP             RTCP
             Media         Control/Stats
                │               │
                └───────┬───────┘
                        ▼
               Dynamic Pinhole
                        │
                        ▼
                   Active Call
```

---

# 32. 🎯 Production Design Pattern

For an Internet-facing enterprise SIP deployment:

```text
                         INTERNET
                            │
                            ▼
                    ┌──────────────┐
                    │   FortiGate  │
                    │              │
                    │     VIP      │
                    │   SIP ALG    │
                    │     HNT      │
                    │     IPS      │
                    │ VoIP Profile │
                    └──────┬───────┘
                           │
                           ▼
                       SIP / PBX
                           │
                  ┌────────┴────────┐
                  ▼                 ▼
                 RTP               RTCP
                  │                 │
                  └────────┬────────┘
                           ▼
                       IP Phones
```

### Security Layers

```text
VIP
 ↓
Source Restriction
 ↓
Firewall Policy
 ↓
SIP ALG / HNT
 ↓
VoIP Inspection
 ↓
IPS
 ↓
RTP/RTCP Expectations
 ↓
Logging & Monitoring
```

---

# 33. ⚠️ Common VoIP Design Mistakes

## ❌ Exposing SIP to Everyone

```text
0.0.0.0/0
     ↓
   SIP
```

Prefer restricting SIP traffic to known providers, trusted peers, or required source networks.

---

## ❌ Forgetting RTP/RTCP

A successful SIP registration does **not** guarantee successful media.

```text
SIP  ✓
RTP  ✗
```

Result:

```text
Call Established
      ↓
No Audio / One-Way Audio
```

---

## ❌ Ignoring SDP

SDP contains critical media negotiation information:

```text
IP
Port
Codec
Media Type
```

---

## ❌ Assuming SIP = Entire Call

```text
SIP ≠ Media

SIP  → Signaling
RTP  → Media
RTCP → Control / Statistics
```

---

## ❌ Using Deep SSL Inspection Without Testing

SSL inspection can introduce:

* CPU overhead
* Memory overhead
* Certificate requirements
* Latency
* Compatibility issues

Always validate the complete VoIP call flow after enabling SSL inspection.

---

# 34. 🛠️ VoIP Troubleshooting Command Set

### SIP Proxy

```cli
diagnose sys sip-proxy calls
```

### Session Table

```cli
diagnose sys session list
```

### Session Expectations

```cli
diagnose sys session list expectation
```

### SIP / RTP Troubleshooting Logic

```text
                    SIP Problem
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
        Registration              Call Setup
             │                       │
             ▼                       ▼
       SIP / VIP / NAT         SIP / SDP / ALG
                                     │
                                     ▼
                               Media Problem
                                     │
                         ┌───────────┴───────────┐
                         ▼                       ▼
                        RTP                    RTCP
                         │                       │
                         └───────────┬───────────┘
                                     ▼
                            Expectations /
                               Pinholes
```

---

# 35. 🚀 One-Minute Revision

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
NAT Traversal

RTP
 ↓
Voice / Video Media

RTCP
 ↓
Control / Statistics

Pinhole
 ↓
Dynamic Media Access

Flow Mode
 ↓
IPS Engine

Proxy Mode
 ↓
Proxy-Based SIP Processing

MSRP
 ↓
Messaging / Session Data

LLDP-MED
 ↓
Voice VLAN / Network Policy
```

---

# 🧩 Final Mental Model

```text
                         FORTIGATE VOIP
                                │
                  ┌─────────────┴─────────────┐
                  ▼                           ▼
             SIGNALING                      MEDIA
                  │                           │
                 SIP                         RTP
                  │                           │
                 SDP                        RTCP
                  │                           │
                  ▼                           ▼
              SIP ALG                    Expectations
                  │                           │
                  ▼                           ▼
                 HNT                       Pinholes
                  │                           │
                  └─────────────┬─────────────┘
                                ▼
                          VOIP SECURITY
                                │
                    ┌───────────┼───────────┐
                    ▼           ▼           ▼
                   IPS         DoS       Logging
```

---

# 🎓 NSE / Interview Memory Hook

> **When SIP works but the call has no audio, don't stop at SIP.**

Remember:

```text
SIP
 ↓
SDP
 ↓
NAT
 ↓
RTP / RTCP
 ↓
Expectations / Pinholes
```

This is one of the most useful mental models for troubleshooting FortiGate VoIP deployments.

---

## 🔗 Related Topics

* SIP ALG
* Hosted NAT Traversal (HNT)
* SIP Proxy
* RTP / RTCP
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

> **SheynShield Engineering Note**
>
> In real-world VoIP troubleshooting, **“SIP is working” does not mean “VoIP is working.”**
>
> A successful `REGISTER` or `INVITE` primarily confirms signaling progress. For media problems, follow the complete chain:
>
> **SIP → SDP → NAT → RTP/RTCP → Expectations/Pinholes**
>
> That sequence gives you a much faster path to the root cause of one-way audio, no-audio, and NAT-related VoIP failures.

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

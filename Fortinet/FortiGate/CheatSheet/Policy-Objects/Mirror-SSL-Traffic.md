# FortiGate Decrypted Traffic Mirror — SSL Traffic Mirror  

> **FortiOS:** 7.2.x
> **Topic:** SSL/SSH Inspection, Decrypted Traffic Mirror, Traffic Monitoring
> **Level:** NSE 6/7 — Advanced Security
> **Brand:** SheynShield | Engineering Secure Networks

---

## 1. What Is Decrypted Traffic Mirror?

**Decrypted Traffic Mirror** allows FortiGate to send a copy of traffic **after decryption** to a monitoring interface/client.

This is useful when a security or monitoring team needs to inspect traffic that is normally encrypted with:

* HTTPS
* SSL/TLS
* SSH

### Normal SSL traffic

```text
Client
  |
  | Encrypted HTTPS
  v
FortiGate
  |
  | Decrypt / Inspect
  v
Internet
```

With decrypted traffic mirroring:

```text
                    +--------------------+
                    |                    |
Client ── HTTPS ──> FortiGate ─────────> Internet
                    |
                    | Decrypt
                    v
             Decrypted Traffic
                    |
                    v
             Mirror Interface
                    |
                    v
               Wireshark
```

### Core idea

```text
Encrypted Traffic
       ↓
SSL Inspection
       ↓
Decryption
       ↓
Security Inspection
       ↓
Decrypted Traffic Mirror
       ↓
Monitoring / Analysis
```

---

# 2. When Is It Useful?

Decrypted traffic mirroring can be useful when an organization needs visibility into traffic that would otherwise appear encrypted to an external packet analyzer.

Typical use cases:

* Security troubleshooting
* Application troubleshooting
* IDS/IPS validation
* Packet-level analysis
* SOC investigation
* SSL inspection validation
* Testing security controls
* Investigating suspicious encrypted traffic

> ⚠️ **Security warning:** Mirrored decrypted traffic may contain credentials, tokens, cookies, personal information, API keys, or other sensitive data. Restrict access to the monitoring host and capture infrastructure.

---

# 3. Prerequisites

Before configuring the mirror, verify:

```text
✓ SSL/SSH inspection is enabled
✓ Policy uses proxy-based inspection
✓ Deep inspection profile is applied
✓ Decrypt Traffic Mirror is enabled
✓ Mirror interface is reachable by the analyzer
✓ Wireshark/tcpdump is running on the monitoring host
```

### Important

A normal firewall policy by itself does **not** produce decrypted traffic.

The traffic must pass through the appropriate inspection/decryption path first.

---

# 4. Firewall Policy

Example baseline policy:

```bash
config firewall policy
    edit 1
        set name "LAN-to-WAN-SSL-Inspection"
        set srcintf "lan"
        set dstintf "wan"
        set srcaddr "all"
        set dstaddr "all"
        set schedule "always"
        set service "ALL"
        set action accept
        set nat enable
    next
end
```

The important part is not simply the policy action.

The policy must use the required **proxy-based inspection + SSL inspection profile**.

---

# 5. Proxy-Based Inspection

Decrypted Traffic Mirror is associated with the **proxy-based inspection/decryption path**.

Conceptually:

```text
Flow-Based

Client
  |
  v
FortiGate
  |
  +--> Inline inspection
  |
  v
Server
```

Versus:

```text
Proxy-Based

Client
  |
  v
FortiGate Proxy
  |
  +--> TLS termination/decryption
  |
  +--> Security inspection
  |
  +--> Decrypted Traffic Mirror
  |
  v
Server
```

### Why proxy mode?

Proxy-based inspection allows FortiGate to terminate/process the relevant encrypted session and inspect the decrypted content.

---

# 6. SSL Inspection Profile

Navigate to:

```text
Security Profiles
    ↓
SSL/SSH Inspection
```

Create or modify a profile using:

```text
Deep Inspection
```

Then enable:

```text
Decrypt Traffic Mirror
```

Conceptually:

```text
SSL/SSH Inspection
        |
        +-- Deep Inspection
        |
        +-- Decrypt Traffic Mirror
```

---

# 7. Decrypt Traffic Mirror Parameters

After enabling the feature, additional parameters become available.

Typical parameters include:

| Parameter                | Purpose                                  |
| ------------------------ | ---------------------------------------- |
| Decrypted traffic type   | Select SSL, SSH, or both                 |
| Decrypted traffic source | Client, server, or both                  |
| Interface                | Interface where mirrored traffic is sent |
| Destination MAC          | Monitoring host destination MAC          |

---

# 8. Traffic Type

The mirror can be restricted according to the decrypted protocol.

Conceptually:

```text
Traffic Type
├── SSL
├── SSH
└── Both
```

### Example

```text
SSL
```

Useful when the goal is specifically to analyze:

```text
HTTPS / TLS
```

rather than SSH traffic.

---

# 9. Traffic Direction

The mirror can select which side of the decrypted communication should be copied.

```text
Client
Server
Both
```

### Client

```text
Client → FortiGate
```

### Server

```text
FortiGate → Server
```

### Both

```text
Client ↔ Server
```

For complete packet analysis:

```text
Both
```

is generally the most useful option.

---

# 10. Mirror Interface

The interface determines where FortiGate sends the mirrored traffic.

Example:

```text
FortiGate
    |
    | Mirror
    v
LAN Interface
    |
    v
Monitoring Host
    |
    v
Wireshark
```

### Lab example

```text
FortiGate LAN
      |
      +---------------- Client
      |
      +---------------- Monitoring PC
```

The monitoring host should be connected to the appropriate Layer-2 segment so that it can receive the mirrored packets.

---

# 11. Destination MAC Address

A destination MAC can be used to identify the monitoring host.

Example:

```text
Destination MAC:
aa:bb:cc:dd:ee:ff
```

### Avoid broad mirroring when possible

Your notes use:

```text
ff:ff:ff:ff:ff:ff
```

That represents the Ethernet broadcast destination.

For a controlled monitoring setup, prefer a **specific monitoring host MAC address** whenever the FortiOS implementation/configuration permits it.

```text
Better:

FortiGate
   |
   +----> Monitoring PC MAC
              |
              v
           Wireshark
```

instead of unnecessarily exposing decrypted traffic broadly.

---

# 12. Configuration Verification

CLI:

```bash
config firewall decrypted-traffic-mirror
    get
end
```

Use this to inspect the current decrypted traffic mirror configuration.

---

# 13. Wireshark Validation

Run Wireshark on the monitoring host connected to the configured mirror interface.

Conceptually:

```text
FortiGate
    |
    | decrypted mirror
    v
LAN
    |
    v
Monitoring PC
    |
    v
Wireshark
```

### Capture filter examples

For HTTPS/TLS:

```text
tcp.port == 443
```

For SSH:

```text
tcp.port == 22
```

For a specific host:

```text
ip.addr == 192.168.101.2
```

For TLS:

```text
tls
```

> The exact packets visible depend on the FortiGate version, inspection path, mirror configuration, and where the capture is taken.

---

# 14. What Should You Expect in Wireshark?

Without decryption:

```text
Client
  |
  | TLS encrypted payload
  v
Wireshark

Application Data
Encrypted
```

With decrypted traffic mirroring:

```text
Client
  |
  v
FortiGate
  |
  | TLS decryption
  v
Decrypted traffic
  |
  v
Mirror
  |
  v
Wireshark
```

The monitoring point can therefore provide significantly more visibility than a normal packet capture taken on the encrypted side.

---

# 15. FortiGate SSL Manager Diagnostics

Start SSL manager diagnostics:

```bash
diagnose test application sslmgr
```

This is useful when troubleshooting SSL inspection/decryption behavior.

---

# 16. HTTPSD Debug

Enable HTTPS daemon debugging:

```bash
diagnose debug application httpsd -1
diagnose debug enable
```

Use this carefully in production environments because verbose debugging can generate significant output.

After troubleshooting, stop debugging:

```bash
diagnose debug disable
```

Optionally reset the debug state:

```bash
diagnose debug reset
```

---

# 17. Recommended Troubleshooting Workflow

```text
Client cannot see mirrored traffic
            |
            v
Is SSL inspection enabled?
       /             \
     No               Yes
     |                 |
 Enable it        Proxy-based policy?
                         |
                         v
                 Deep Inspection?
                    /        \
                  No          Yes
                  |            |
             Fix profile    Mirror enabled?
                               |
                               v
                         Correct protocol?
                               |
                               v
                         Correct interface?
                               |
                               v
                       Correct destination MAC?
                               |
                               v
                       Capture with Wireshark
                               |
                               v
                       Check FortiGate debug
```

---

# 18. Common Configuration Mistakes

## ❌ Using Flow-Based Inspection

If the required decrypted traffic mirror functionality depends on proxy-based processing, a flow-based policy will not provide the same decryption workflow.

```text
Flow-based
   ↓
Not the intended mirror/decryption path
```

Use:

```text
Proxy-based
   ↓
SSL Deep Inspection
   ↓
Decrypt Traffic Mirror
```

---

## ❌ Enabling Mirror Without Decryption

Traffic mirroring does not magically decrypt TLS.

The processing chain must be:

```text
TLS
 ↓
SSL Inspection
 ↓
Decryption
 ↓
Mirror
```

---

## ❌ Capturing on the Wrong Interface

If Wireshark is connected to another interface:

```text
FortiGate
    |
    +---- mirror ---> LAN
                         |
                         X
                     Wireshark
```

No mirrored traffic will be observed.

Verify the configured mirror interface.

---

## ❌ Using a Broad Destination MAC

Using:

```text
ff:ff:ff:ff:ff:ff
```

can make the capture unnecessarily broad.

Prefer a dedicated monitoring destination where appropriate.

---

## ❌ Forgetting Certificate Trust

Deep Inspection commonly requires the client to trust the CA used by FortiGate for SSL interception.

Conceptually:

```text
Client
  |
  | trusts FortiGate Inspection CA
  v
FortiGate
  |
  | creates inspected TLS session
  v
Internet Server
```

If the CA is not trusted correctly, clients may receive certificate warnings/errors.

---

# 19. SSL Inspection vs Decrypted Traffic Mirror

| Feature              | SSL Inspection               | Decrypted Traffic Mirror   |
| -------------------- | ---------------------------- | -------------------------- |
| Decrypt TLS          | ✅                            | Depends on SSL inspection  |
| Security inspection  | ✅                            | Not its primary purpose    |
| Send copy elsewhere  | ❌                            | ✅                          |
| Packet analysis      | Limited to FortiGate tools   | External analyzer possible |
| Wireshark visibility | Usually encrypted externally | Decrypted copy available   |
| Typical use          | Security enforcement         | Monitoring/troubleshooting |

### Key distinction

```text
SSL Inspection
    =
FortiGate decrypts traffic for inspection

Decrypted Traffic Mirror
    =
FortiGate copies decrypted traffic
to another monitoring point
```

---

# 20. Security Architecture

A secure monitoring design should look like:

```text
                 Internet
                    |
                    v
              +-----------+
              | FortiGate |
              |           |
              | SSL Proxy |
              +-----------+
                    |
              +-----+------+
              |            |
              v            v
           Server      Mirror Port
                           |
                           v
                    Monitoring PC
                           |
                           v
                       Wireshark
```

### Recommended controls

* Dedicated monitoring host
* Restricted physical/network access
* Limited administrator access
* Short packet-capture retention
* Encryption at rest for captures
* Strict access control
* Avoid capturing unnecessary users
* Mask/redact sensitive data where possible

---

# 21. Why This Feature Is Powerful

Traditional packet capture:

```text
Client ─── TLS ───> Server

Capture:
Encrypted Payload
```

Decrypted traffic mirror:

```text
Client
   |
   | TLS
   v
FortiGate
   |
   | Decrypt
   |
   +------> Security Inspection
   |
   +------> Decrypted Mirror
                  |
                  v
              Wireshark
```

This creates a powerful troubleshooting point between:

```text
Encryption
     ↓
Decryption
     ↓
Security Inspection
     ↓
Application Traffic
```

---

# 22. NSE Exam Memory

### Remember the chain

```text
Policy
  ↓
Proxy Inspection
  ↓
Deep SSL Inspection
  ↓
Decrypt
  ↓
Decrypt Traffic Mirror
  ↓
Monitoring Interface
  ↓
Wireshark
```

### Three important questions

**Q: Does a normal firewall policy decrypt HTTPS?**

> No. Appropriate SSL/SSH inspection must be configured.

**Q: Why use proxy-based inspection?**

> It provides the processing model required for full proxy/decryption-based inspection features.

**Q: What is the purpose of Decrypted Traffic Mirror?**

> To provide a copy of decrypted traffic to an external monitoring/analysis point.

---

# 23. Quick CLI Reference

```bash
# Check decrypted traffic mirror
config firewall decrypted-traffic-mirror
    get
end

# SSL manager diagnostic
diagnose test application sslmgr

# HTTPS daemon debug
diagnose debug application httpsd -1

# Enable debugging
diagnose debug enable

# Stop debugging
diagnose debug disable

# Reset debug
diagnose debug reset
```

---

# 24. Quick Revision

```text
SSL Traffic
     ↓
Proxy-Based Inspection
     ↓
Deep Inspection
     ↓
Decrypt
     ↓
Mirror
     ↓
Monitoring Interface
     ↓
Wireshark
```

### Remember

```text
SSL Inspection
    → decrypt for FortiGate inspection

Decrypted Traffic Mirror
    → copy decrypted traffic for external analysis

Wireshark
    → analyze the mirrored traffic
```

> **⚠️ Production Warning:** Decrypted traffic captures can expose credentials, session cookies, authentication tokens, personal data, and confidential application content. Treat the mirror network and packet captures as highly sensitive security assets.

---

## 🔥 SheynShield Takeaway

> **The real value of Decrypted Traffic Mirror is not simply "seeing HTTPS." It creates an external observation point after FortiGate has performed TLS decryption, allowing security engineers to correlate encrypted sessions, FortiGate inspection behavior, and application-level traffic in packet analysis tools.**

**SheynShield | Engineering Secure Networks**

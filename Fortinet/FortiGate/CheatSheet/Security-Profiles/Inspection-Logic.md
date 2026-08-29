# FortiGate SSL Deep Inspection, Certificate Flow, JA3/JA4 & Traffic Inspection  

> **FortiGate Security  ** — SSL/TLS Inspection, Deep Inspection, Certificate Authentication, PSIPHON Detection, JA3/JA4, Flow vs Proxy Inspection
>
> **Focus:** FortiOS Security Profiles / SSL Inspection / IPS / Application Control / Antivirus
>
> **Difficulty:** NSE 4 → NSE 7
>
> **Use case:** Troubleshooting, security design, classroom reference, certification preparation

---

## Table of Contents

* [1. SSL Inspection in One Minute](#1-ssl-inspection-in-one-minute)
* [2. Why Deep Inspection Is a MITM](#2-why-deep-inspection-is-a-mitm)
* [3. Deep SSL Inspection Packet Flow](#3-deep-ssl-inspection-packet-flow)
* [4. Certificate Logic](#4-certificate-logic)
* [5. Certificate Inspection vs Deep Inspection](#5-certificate-inspection-vs-deep-inspection)
* [6. FortiGate CA Trust Model](#6-fortigate-ca-trust-model)
* [7. Client Certificate Installation](#7-client-certificate-installation)
* [8. HTTPS Packet Flow](#8-https-packet-flow)
* [9. Why Psiphon Can Be Detected](#9-why-psiphon-can-be-detected)
* [10. TLS ClientHello](#10-tls-clienthello)
* [11. JA3 Fingerprinting](#11-ja3-fingerprinting)
* [12. JA4 / JA4+](#12-ja4--ja4)
* [13. Multi-Layer Psiphon Defense](#13-multi-layer-psiphon-defense)
* [14. Security Profile Direction](#14-security-profile-direction)
* [15. Flow-Based Inspection](#15-flow-based-inspection)
* [16. Proxy-Based Inspection](#16-proxy-based-inspection)
* [17. Flow vs Proxy](#17-flow-vs-proxy)
* [18. DFA Pattern Matching](#18-dfa-pattern-matching)
* [19. Oversized File Handling](#19-oversized-file-handling)
* [20. Comfort Client](#20-comfort-client)
* [21. Content Disarm & Reconstruction](#21-content-disarm--reconstruction)
* [22. Sandbox](#22-sandbox)
* [23. Emulator](#23-emulator)
* [24. Infection Quarantine](#24-infection-quarantine)
* [25. Heuristic Detection](#25-heuristic-detection)
* [26. Banned Word Detection](#26-banned-word-detection)
* [27. Practical Troubleshooting Checklist](#27-practical-troubleshooting-checklist)
* [28. NSE Exam Memory Map](#28-nse-exam-memory-map)

---

# 1. SSL Inspection in One Minute

The fundamental problem:

```text
Client
   |
   | HTTPS / TLS
   v
FortiGate
   |
   | Encrypted traffic
   v
Internet Server
```

Without decryption:

```text
Client ---> [Encrypted HTTPS] ---> FortiGate ---> Internet
                                      |
                                      X
                              Cannot inspect payload
```

The FortiGate normally cannot inspect the encrypted payload because it does **not** possess the destination server's private key.

Therefore, when **Deep Inspection** is enabled, FortiGate establishes two separate TLS relationships:

```text
                    TLS Session #1
Client <--------------------------------> FortiGate

                    TLS Session #2
FortiGate <-----------------------------> Internet Server
```

FortiGate becomes a **controlled TLS interception point**.

> **Key idea:** Deep Inspection does not simply "decrypt the original TLS session". It terminates one TLS session and establishes another.

---

# 2. Why Deep Inspection Is a MITM

Deep SSL Inspection is conceptually based on a controlled:

> **Man-in-the-Middle (MITM)**

The important distinction is that this is an **authorized security inspection architecture**, not an uncontrolled attack.

```text
                TLS #1                         TLS #2

Client  <---------------->  FortiGate  <---------------->  Google
         FortiGate CA                  Google Certificate
```

FortiGate:

1. Receives the client's TLS request.
2. Establishes a separate TLS connection to the destination.
3. Validates the destination certificate.
4. Decrypts traffic.
5. Applies security inspection.
6. Re-encrypts the inspected content.
7. Sends it to the client.

---

# 3. Deep SSL Inspection Packet Flow

## Phase 1 — Client Initiates HTTPS

Example:

```text
Client
   |
   | ClientHello
   | SNI = google.com
   v
FortiGate
```

FortiGate intercepts the connection.

---

## Phase 2 — FortiGate Connects to Real Server

```text
FortiGate
   |
   | ClientHello
   | SNI = google.com
   v
Google
```

The real server responds with its actual certificate.

```text
Google
   |
   | Real Google Certificate
   v
FortiGate
```

FortiGate validates the certificate using its trusted CA store.

---

## Phase 3 — FortiGate Decrypts Traffic

```text
Internet Server
       |
       | Encrypted TLS
       v
+--------------------+
|     FortiGate      |
|                    |
| TLS termination    |
|       ↓            |
| Decrypt            |
|       ↓            |
| IPS                |
| Antivirus          |
| Web Filter         |
| DLP                |
| App Control        |
| File Filter        |
|       ↓            |
| Re-encrypt         |
+--------------------+
       |
       | Encrypted TLS
       v
     Client
```

---

# 4. Certificate Logic

This is one of the most important concepts in SSL Deep Inspection.

## Client → FortiGate

The client receives a certificate dynamically generated by FortiGate.

Example:

```text
Requested site:
google.com

Certificate presented to client:
google.com

Certificate issuer:
Fortinet_CA_SSL
```

FortiGate dynamically generates a certificate for the requested hostname and signs it with the configured inspection CA.

---

## FortiGate → Internet

FortiGate connects to the real destination.

```text
FortiGate
    |
    | Real TLS session
    v
google.com

Certificate:
Google's real certificate
```

FortiGate does **not** need Google's private key.

---

## Critical Certificate Rule

```text
FortiGate Trusted Root CAs
        |
        +----> Validate external server certificates
        |
        X----> Not used to impersonate every Internet server
```

The configured **inspection CA** is used to sign dynamically generated certificates presented toward clients.

---

# 5. Certificate Inspection vs Deep Inspection

| Feature                      | Certificate Inspection | Deep Inspection           |
| ---------------------------- | ---------------------- | ------------------------- |
| TLS interception             | No full interception   | Yes                       |
| Payload decrypted            | ❌                      | ✅                         |
| URL/SNI visibility           | ✅                      | ✅                         |
| Certificate information      | ✅                      | ✅                         |
| File inspection              | Limited                | ✅                         |
| Antivirus payload inspection | Limited                | ✅                         |
| HTTP content inspection      | ❌                      | ✅                         |
| DLP payload inspection       | ❌                      | ✅                         |
| Client CA required           | ❌                      | ✅                         |
| CPU impact                   | Lower                  | Higher                    |
| Privacy impact               | Lower                  | Higher                    |
| Detect encrypted tunneling   | Limited                | Stronger                  |
| Certificate warning          | Normally no            | Yes, unless CA is trusted |

---

## Easy Memory Trick

### Certificate Inspection

> **Read the envelope without opening the letter.**

You can inspect things such as:

```text
Destination
Certificate
SNI
TLS metadata
```

But not the encrypted payload.

---

### Deep Inspection

> **Open the envelope, inspect the content, then package it again.**

```text
Encrypted
   ↓
Decrypt
   ↓
Inspect
   ↓
Re-encrypt
   ↓
Forward
```

---

# 6. FortiGate CA Trust Model

There are two different certificate roles that must not be confused.

## A. External Root CA Certificates

Used by FortiGate to validate Internet certificates.

```text
Internet Server Certificate
          |
          v
FortiGate Trusted CA Store
          |
          v
Certificate Validation
```

---

## B. SSL Inspection CA

Used by FortiGate to generate certificates for clients.

```text
Client
   ^
   |
Fake/Dynamic Certificate
   |
Signed by
   |
FortiGate Inspection CA
```

### Exam Memory

```text
Trusted Root CA
    =
Validate external servers

Inspection CA
    =
Sign certificates presented to clients
```

---

# 7. Client Certificate Installation

If the FortiGate inspection CA is not trusted by the client:

```text
Client
   |
   | google.com
   v
FortiGate
   |
   | google.com certificate
   | Issuer = FortiGate Inspection CA
   v
Browser
   |
   X
Certificate Warning
```

The browser may display:

```text
Untrusted Certificate
Certificate Authority Not Trusted
Possible MITM
```

---

## When CA Is Installed

If the organization's inspection CA is installed into the client's trusted root store:

```text
FortiGate
    |
    | Dynamic google.com certificate
    | signed by trusted corporate CA
    v
Client
    |
    ✓ Certificate trusted
    |
    ✓ HTTPS continues
```

In enterprise environments, this is commonly distributed using mechanisms such as:

```text
Active Directory
GPO
MDM
Endpoint Management
Manual Certificate Deployment
```

---

# 8. HTTPS Packet Flow

## Simplified Architecture

```text
                    INTERNET
                       |
                       |
                Real Server Cert
                       |
                       v
              +----------------+
              |    FortiGate   |
              |                |
              | TLS #2         |
              | Decrypt        |
              | Inspect        |
              | Re-encrypt     |
              |                |
              | TLS #1         |
              +----------------+
                       |
                       |
                FortiGate CA
                       |
                       v
                    CLIENT
```

---

## Complete Flow

```text
1. Client → FortiGate
   ClientHello

2. FortiGate → Internet
   New ClientHello

3. Internet → FortiGate
   Real Server Certificate

4. FortiGate
   Validates certificate

5. FortiGate
   Generates client-facing certificate

6. FortiGate → Client
   Dynamic Certificate

7. Client ↔ FortiGate
   TLS Session #1

8. FortiGate ↔ Server
   TLS Session #2

9. FortiGate
   Decrypts traffic

10. FortiGate
    Applies security profiles

11. FortiGate
    Re-encrypts traffic

12. FortiGate → Client
    Inspected content
```

---

# 9. Why Psiphon Can Be Detected

Tools designed to bypass filtering may attempt to make their traffic look like ordinary HTTPS.

Conceptually:

```text
Client
  |
  | TCP/443
  | Looks like HTTPS
  v
FortiGate
  |
  v
Internet
```

Without Deep Inspection:

```text
FortiGate sees:

Port = 443
SNI = apparently legitimate domain
TLS = apparently valid

        ↓

Limited visibility
```

With Deep Inspection:

```text
TLS interception
      ↓
Decrypt
      ↓
Inspect handshake/content
      ↓
Application identification
      ↓
Policy decision
```

---

# 10. TLS ClientHello

The **ClientHello** is the first major TLS handshake message sent by the client.

Simplified structure:

```text
+--------------------------------------------------+
| TLS Record Header                                |
| Content Type = 0x16 (Handshake)                  |
+--------------------------------------------------+
| Handshake Header                                 |
| Type = 0x01 (ClientHello)                       |
+--------------------------------------------------+
| Client Version                                   |
+--------------------------------------------------+
| Random                                           |
+--------------------------------------------------+
| Session ID                                       |
+--------------------------------------------------+
| Cipher Suites                                    |
+--------------------------------------------------+
| Compression Methods                              |
+--------------------------------------------------+
| Extensions                                       |
|                                                  |
| - SNI                                            |
| - Supported Groups                               |
| - ALPN                                           |
| - Key Share (TLS 1.3)                            |
| - Signature Algorithms                           |
| - Other extensions                               |
+--------------------------------------------------+
```

> **Important:** TLS Record Content Type `0x16` means Handshake. `ClientHello` itself has handshake type `0x01`.

---

# 11. JA3 Fingerprinting

**JA3** is a TLS client fingerprinting technique originally introduced by Salesforce researchers.

It uses characteristics from the TLS ClientHello.

Conceptually:

```text
TLS Version
     +
Cipher Suites
     +
Extensions
     +
Elliptic Curves
     +
EC Point Formats
     =
JA3 String
```

Example:

```text
771,4865-4866-4867-49195-49199,0-23-65281-10-11,29-23-24,0
```

The resulting string can be hashed using MD5 to produce a JA3 fingerprint.

Example:

```text
JA3 String
    |
    v
MD5
    |
    v
32-character fingerprint
```

### Important Security Concept

JA3 is a **fingerprint**, not an absolute identity.

The same application can potentially produce different fingerprints across versions/configurations, and different applications can sometimes produce similar fingerprints.

Therefore:

```text
JA3
 +
IP reputation
 +
SNI
 +
Application behavior
 +
Protocol metadata
 +
Other security signals
```

provides stronger detection than JA3 alone.

---

# 12. JA4 / JA4+

JA4 was designed to improve fingerprinting robustness, especially with modern TLS behavior such as TLS 1.3 and GREASE.

Conceptually:

```text
JA3
 |
 +-- Sensitive to ordering / randomization
 |
 +-- MD5 representation
```

Whereas JA4 uses a more structured representation.

Example conceptual format:

```text
t13d151600_8daaf6152771_03fe0828775f
```

JA4+ extends the concept to multiple protocol/application fingerprints.

---

## JA3 vs JA4

| Feature                    | JA3               | JA4                       |
| -------------------------- | ----------------- | ------------------------- |
| TLS fingerprint            | ✅                 | ✅                         |
| Human-readable component   | ❌                 | ✅                         |
| Modern TLS handling        | Limited           | Improved                  |
| GREASE resilience          | Lower             | Better                    |
| Fingerprint representation | MD5               | Structured                |
| Scope                      | Mainly TLS client | JA4+ ecosystem is broader |

---

# 13. Multi-Layer Psiphon Defense

> **Do not rely on SSL Inspection alone.**

A stronger FortiGate architecture uses multiple layers.

```text
                 Internet
                    |
                    v
          +-------------------+
          |   Firewall Policy |
          +-------------------+
                    |
                    v
          +-------------------+
          |   SSL Inspection  |
          +-------------------+
                    |
                    v
          +-------------------+
          | Application Ctrl  |
          +-------------------+
                    |
                    v
          +-------------------+
          | Web Filter / ISDB  |
          +-------------------+
                    |
                    v
          +-------------------+
          | IPS / AV / DLP     |
          +-------------------+
                    |
                    v
                  Client
```

---

## Layer 1 — Deep SSL Inspection

Useful for:

* TLS protocol validation
* Encrypted traffic inspection
* Suspicious TLS behavior
* Application identification
* Content inspection

---

## Layer 2 — Application Control

Application Control can identify applications based on signatures and behavioral characteristics.

Example conceptual configuration:

```text
config application list
    edit "block-vpn-list"
        config entries
            edit 1
                set application <psiphon-signature>
                set action block
            next
        end
    next
end
```

> Application signature IDs can change across FortiOS/FortiGuard versions. Verify the current signature in your environment rather than hard-coding an old ID.

Potential complementary categories/applications include:

```text
Proxy
VPN
Tor
SSH Proxy
Anonymizers
Circumvention Tools
QUIC
```

---

## Layer 3 — Web Filter / ISDB

Useful for blocking:

```text
Proxy Avoidance
Anonymizers
Dynamic DNS
Malicious Websites
Known VPN/Proxy Infrastructure
```

---

## Layer 4 — Port and Protocol Hardening

For controlled environments:

```text
Allow:
TCP/80
TCP/443
DNS

Restrict:
Direct SSH
Unknown outbound ports
Unnecessary UDP protocols
Unapproved tunnels
```

QUIC commonly uses:

```text
UDP/443
```

Blocking or controlling UDP/443 can force some applications toward TCP-based paths, where additional inspection controls may apply.

> Do this carefully: blocking QUIC can affect legitimate HTTP/3 traffic.

---

# 14. Security Profile Direction

Security profiles in a stateful firewall are not simply "outbound only" or "inbound only".

Different engines inspect different parts of a session.

```text
                 OUTBOUND
Client ----------------------------> Server
       Request / Initial Traffic
                    |
                    v
              FortiGate
```

Possible request-side inspection:

```text
Web Filter
DNS Filter
Application Control
IPS
WAF
```

---

## Response Direction

```text
Client <---------------------------- Server
             Response / Data
                    |
                    v
              FortiGate
```

Possible response-side inspection:

```text
Antivirus
File Filter
DLP
IPS
SSL Inspection
```

---

## Simplified Direction Map

| Security Feature    | Typical Visibility              |
| ------------------- | ------------------------------- |
| Web Filter          | Request                         |
| DNS Filter          | DNS query                       |
| Application Control | Session / application behavior  |
| Antivirus           | Files/content                   |
| File Filter         | Files                           |
| DLP                 | Content/data                    |
| IPS                 | Both directions                 |
| SSL Inspection      | TLS/session + decrypted content |

> Exact inspection behavior depends on protocol, inspection mode, security profile, and traffic direction.

---

# 15. Flow-Based Inspection

Flow mode is designed for high-performance inspection.

Conceptually:

```text
Packet
  |
  v
Pattern / Signature Matching
  |
  +---- Match ----> BLOCK
  |
  +---- No Match -> FORWARD
```

The traffic is inspected while it is flowing through the FortiGate rather than being fully reconstructed and stored like a proxy workflow.

---

## Advantages

```text
+ High performance
+ Lower latency
+ Lower resource consumption
+ ASIC acceleration where supported
```

---

## Limitation

Some complex file/content inspection scenarios require proxy-based inspection.

For example:

```text
Very large file
Complex archive
Content reconstruction
Deep file analysis
```

may require buffering/reassembly.

---

# 16. Proxy-Based Inspection

Proxy mode acts more like an intermediary.

Conceptually:

```text
Client
   |
   | File
   v
FortiGate Proxy
   |
   | Buffer / Reassemble
   | Scan
   | Inspect
   v
Security Decision
   |
   +---- Malicious ----> DROP
   |
   +---- Clean --------> Forward
```

For content that must be reconstructed, FortiGate may need to temporarily buffer content.

---

## Resource Consideration

Proxy inspection can consume more:

```text
CPU
Memory
Storage / local disk
Processing resources
```

Therefore:

> **More inspection depth = more resource planning.**

---

# 17. Flow vs Proxy

| Feature                       | Flow                              | Proxy                             |
| ----------------------------- | --------------------------------- | --------------------------------- |
| Architecture                  | Inline inspection                 | Proxy/intermediary                |
| Performance                   | Very high                         | Higher resource cost              |
| Latency                       | Lower                             | Potentially higher                |
| Full file buffering           | Limited                           | Stronger                          |
| Content reconstruction        | Limited                           | Strong                            |
| Large/complex file inspection | More limited                      | Better                            |
| Resource usage                | Lower                             | Higher                            |
| ASIC acceleration             | Available for supported functions | Available for supported functions |
| Typical use                   | Performance-oriented inspection   | Deep content inspection           |

---

# 18. DFA Pattern Matching

**Direct Filter Approach (DFA)** is used for very fast pattern matching.

Conceptually:

```text
Traffic
   |
   v
DFA / Pattern Engine
   |
   +---- Match ----> Security Action
   |
   +---- No Match -> Continue
```

Hardware acceleration can improve pattern-matching performance on supported FortiGate platforms.

---

## File Flow Example

```text
Client
   |
   | Download
   v
FortiGate
   |
   | Pattern detection
   |
   +---- Malicious pattern
             |
             v
           DROP
             |
             v
      Connection terminated
```

Depending on the protocol and inspection engine, FortiGate can terminate the connection so the client cannot complete the download.

---

# 19. Oversized File Handling

FortiGate has finite resources for buffering and scanning files.

Large files such as:

```text
ISO
Video
Large Archive
Large Email Attachment
```

can consume significant memory/storage.

---

## Oversized File Threshold

The oversized file/email threshold determines when content is considered too large for certain Antivirus processing.

Example:

```text
Threshold = 10 MB

File = 5 MB
       |
       v
Normal inspection

File = 50 MB
       |
       v
Oversized handling
```

The exact supported range depends on the FortiGate model and FortiOS version.

---

## Why Oversized Files Matter

```text
Many large files
       |
       v
Memory pressure
       |
       v
Conserve Mode
       |
       v
Performance impact
```

If a FortiGate repeatedly enters conserve mode, reducing oversized-file thresholds may reduce resource pressure.

### Security Trade-off

```text
Lower threshold
      |
      +--> Lower resource consumption
      |
      +--> Potentially less file inspection
      |
      +--> Potentially higher security risk
```

Therefore, monitor logs before changing the threshold.

---

# 20. Comfort Client

Comfort Client controls the user experience while FortiGate processes files.

---

## Comfort Client OFF

Conceptually:

```text
Client requests file
       |
       v
FortiGate receives/buffers file
       |
       v
Scan
       |
       +---- Clean ----> Send file
       |
       +---- Malicious -> Drop
```

The user may see little or no download progress while inspection occurs.

---

## Comfort Client ON

The client can receive progress while FortiGate continues processing the file.

Conceptually:

```text
Client
  |
  | Partial content / progress
  v
FortiGate
  |
  | Continue inspection
  |
  +---- Clean ------> Complete transfer
  |
  +---- Malicious --> Reset / terminate
```

### Operational Trade-off

```text
Comfort Client ON
        |
        +--> Better UX
        |
        +--> Potentially more temporary content handling
```

---

# 21. Content Disarm & Reconstruction

**CDR / Content Disarm and Reconstruction** analyzes supported files and removes potentially dangerous active content.

Conceptually:

```text
Original File
     |
     v
Analyze
     |
     v
Identify active / suspicious content
     |
     v
Remove / sanitize
     |
     v
Reconstruct
     |
     v
Sanitized File
```

The goal is to preserve usable business content while reducing exposure to malicious active components.

---

# 22. Sandbox

Sandboxing provides another analysis layer for suspicious files.

A simplified architecture:

```text
File
 |
 v
FortiGate
 |
 +--> Local Signature Check
 |
 +--> Sandbox Analysis
          |
          v
   Behavioral Analysis
          |
          v
     Verdict
```

Depending on the configured architecture, sandboxing can work with flow- or proxy-based workflows, while some inline workflows have stricter requirements.

---

## Post-Transfer Scanning

Conceptually:

```text
File
 |
 v
Local FortiGate Signature Check
 |
 v
Transfer / Submit to Sandbox
 |
 v
Sandbox Verdict
```

---

## Inline Scanning

Conceptually:

```text
File
 |
 v
Proxy
 |
 v
Hash / Analysis
 |
 v
Sandbox
 |
 v
Verdict
 |
 +---- Clean ----> Forward
 |
 +---- Malicious -> Block
```

> Exact behavior and available sandbox modes depend on FortiOS version, FortiSandbox integration, licensing, and configuration.

---

# 23. Emulator

An emulator provides an isolated environment for analyzing file behavior.

Conceptually:

```text
Suspicious File
      |
      v
+------------------+
| Isolated         |
| Analysis         |
| Environment      |
+------------------+
      |
      v
Behavior Analysis
      |
      v
Verdict
```

This can help identify behavior that traditional static signatures may miss.

---

## Resource Consideration

Emulation can consume:

```text
CPU
Memory
Local Storage
Processing Time
```

Therefore, file-size thresholds and resource planning matter.

---

# 24. Infection Quarantine

Quarantine can retain suspicious/infected content for additional analysis.

Conceptually:

```text
Suspicious File
      |
      v
Detection
      |
      v
Quarantine
      |
      v
Further Investigation
```

Depending on the architecture, quarantine/storage may involve:

```text
Local FortiGate Storage
FortiAnalyzer
FortiManager
FortiSandbox
Other integrated storage/analysis components
```

> Always verify the exact quarantine workflow supported by your FortiOS release.

---

# 25. Heuristic Detection

Heuristic detection does not rely only on a known malware signature.

Instead, it can analyze:

```text
File characteristics
+
Behavior
+
Patterns
+
Anomalies
+
Machine-learning / statistical signals
```

Conceptually:

```text
Known Malware
      |
      +--> Signature Detection

Unknown / Variant Malware
      |
      +--> Heuristic / Behavioral Detection
```

### Key Concept

```text
Signature = "Have I seen this exact pattern?"

Heuristic = "Does this behavior look malicious?"
```

---

# 26. Banned Word Detection

Banned-word filtering can inspect supported email content for configured terms.

Example:

```text
Email
  |
  v
Content Inspection
  |
  v
Banned Word Match?
  |
  +---- YES ---> Policy Action
  |
  +---- NO ----> Continue
```

Depending on protocol and inspection mode, this can be applied in supported:

```text
Flow inspection
Proxy inspection
Email inspection
```

---

# 27. Practical Troubleshooting Checklist

## SSL Deep Inspection

```text
[ ] Is SSL/SSH Inspection enabled?
[ ] Is the correct inspection profile attached?
[ ] Is Deep Inspection enabled?
[ ] Is the inspection CA configured?
[ ] Is the CA trusted by clients?
[ ] Is certificate validation working?
[ ] Is FortiGuard licensing available where required?
[ ] Are security profiles attached to the correct firewall policy?
```

---

## Certificate Problems

```text
[ ] Check client certificate trust store
[ ] Check FortiGate inspection CA
[ ] Check certificate expiration
[ ] Check hostname/SNI
[ ] Check certificate chain
[ ] Check client system time
[ ] Check TLS compatibility
```

---

## Performance Problems

```text
[ ] Check CPU
[ ] Check memory
[ ] Check conserve mode
[ ] Check proxy workload
[ ] Check oversized-file threshold
[ ] Check concurrent sessions
[ ] Check SSL inspection workload
[ ] Check sandbox workload
```

---

## Application Bypass Troubleshooting

```text
[ ] Check Application Control
[ ] Check Web Filter
[ ] Check ISDB
[ ] Check IPS
[ ] Check SSL inspection
[ ] Check QUIC / UDP 443
[ ] Check non-standard outbound ports
[ ] Check DNS behavior
[ ] Check proxy/VPN categories
```

---

# 28. NSE Exam Memory Map

## SSL Inspection

```text
Certificate Inspection
        |
        +--> Metadata
        +--> Certificate
        +--> SNI
        +--> No full payload decryption

Deep Inspection
        |
        +--> TLS termination
        +--> Decryption
        +--> Security inspection
        +--> Re-encryption
        +--> Client CA trust required
```

---

## Certificate Roles

```text
External Trusted CAs
        |
        +--> Validate Internet certificates

Inspection CA
        |
        +--> Sign dynamic certificates
        |
        +--> Presented toward clients
```

---

## Deep Inspection

```text
Client
  |
  | TLS #1
  v
FortiGate
  |
  | Decrypt
  | Inspect
  | Re-encrypt
  |
  | TLS #2
  v
Internet
```

---

## Flow vs Proxy

```text
FLOW
 |
 +--> Fast
 +--> Pattern matching
 +--> Lower resource overhead
 +--> Limited buffering/reconstruction

PROXY
 |
 +--> Buffer
 +--> Reassemble
 +--> Deep file/content inspection
 +--> Higher resource consumption
```

---

## Psiphon / Circumvention Defense

```text
                 Psiphon / VPN / Proxy
                          |
        +-----------------+----------------+
        |                 |                |
        v                 v                v
  SSL Inspection   Application Ctrl   Web Filter
        |                 |                |
        v                 v                v
 TLS/content         Signature          Category/
 analysis             detection         reputation
        |                 |                |
        +-----------------+----------------+
                          |
                          v
                    Firewall Policy
```

---

## JA3 / JA4

```text
TLS ClientHello
      |
      +--> Version
      +--> Cipher Suites
      +--> Extensions
      +--> Supported Groups
      +--> Other TLS metadata
              |
              v
        Fingerprinting
              |
              +--> JA3
              |
              +--> JA4 / JA4+
```

---

# High-Value Exam Traps

> [!IMPORTANT]
> **Do not confuse certificate inspection with deep inspection.** Certificate inspection does not provide full payload visibility.

> [!IMPORTANT]
> **FortiGate does not need Google's private key for Deep Inspection.** It terminates the client-side TLS session and creates a separate TLS session to the real server.

> [!IMPORTANT]
> **The external server's certificate and FortiGate's client-facing certificate are not the same TLS certificate.**

> [!IMPORTANT]
> **FortiGate Trusted CAs validate external certificates; the SSL inspection CA signs certificates presented to clients.**

> [!IMPORTANT]
> **TLS ClientHello is Handshake type `0x01`; the TLS Record Content Type for a handshake record is `0x16`.**

> [!IMPORTANT]
> **JA3/JA4 are fingerprints, not cryptographic proof of application identity.**

> [!IMPORTANT]
> **Blocking UDP/443 can affect legitimate HTTP/3/QUIC traffic. Treat it as a deliberate policy decision, not a universal security rule.**

> [!IMPORTANT]
> **Flow mode is optimized for performance; proxy mode is used when deeper buffering, reconstruction, or content inspection is required.**

---

# One-Page Mental Model

```text
                    HTTPS
                      |
                      v
               +-------------+
               |  FortiGate  |
               +-------------+
                      |
             SSL Inspection?
                /           \
              NO             YES
              |               |
              v               v
        Limited View      TLS MITM
                              |
                    +---------+---------+
                    |                   |
                    v                   v
               Client TLS          Server TLS
                    |                   |
                    +--------+----------+
                             |
                             v
                         DECRYPT
                             |
             +---------------+---------------+
             |               |               |
             v               v               v
            IPS             AV         Application Control
             |               |               |
             +---------------+---------------+
                             |
                             v
                          DLP/Web
                             |
                             v
                          Decision
                         /        \
                       DROP       ALLOW
                                  |
                                  v
                              RE-ENCRYPT
                                  |
                                  v
                                Client
```

---

# Quick Revision

```text
Certificate Inspection
= See certificate/TLS metadata

Deep Inspection
= Decrypt + Inspect + Re-encrypt

Inspection CA
= Signs client-facing dynamic certificates

Trusted Root CA
= Validates external server certificates

Flow
= Fast inline pattern inspection

Proxy
= Buffer/reassemble/deeper content inspection

JA3
= TLS ClientHello fingerprint

JA4
= More structured modern fingerprinting

Application Control
= Application/signature/behavior identification

Web Filter
= URL/category/reputation control

IPS
= Attack/exploit detection

AV
= Malware/file scanning

DLP
= Sensitive-data inspection

Sandbox
= Behavioral analysis of suspicious files

CDR
= Remove active/dangerous content and reconstruct

QUIC
= Commonly UDP/443

Deep Inspection
= Stronger visibility into encrypted traffic
```

---

## Suggested GitHub  Keywords

`FortiGate SSL Inspection` · `FortiGate Deep Inspection` · `FortiOS SSL Inspection  ` · `FortiGate Certificate Inspection` · `FortiGate SSL MITM` · `FortiGate CA Certificate` · `FortiGate JA3` · `FortiGate JA4` · `FortiGate Psiphon Detection` · `FortiGate Application Control` · `FortiGate Flow Inspection` · `FortiGate Proxy Inspection` · `FortiGate Antivirus` · `FortiGate Sandbox` · `FortiGate IPS` · `FortiGate NSE4` · `FortiGate NSE7`

---

**SheynShield | Engineering Secure Networks**

> Learn the packet flow. Understand the inspection engine. Design the security policy.

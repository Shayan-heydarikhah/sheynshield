# FortiGate IPS — Intrusion Prevention System  

> **FortiOS Focus:** IPS Engine, Deep Packet Inspection, Signatures, Protocol Decoders, CVE, Heuristics, Threat Intelligence, Anomaly Detection, Zero-Day Protection
> **Audience:** FortiGate / NSE4–NSE7 / Network Security Engineers
> **Core Technologies:** IPS Signatures, Protocol Decoders, Heuristics, Threat Intelligence, CVE Mapping, Advanced Threat Detection
> **Primary Goal:** Detect and prevent malicious network activity before it reaches protected systems

---

## 📌 Quick Reference

| Component               | Purpose                                                                                        |
| ----------------------- | ---------------------------------------------------------------------------------------------- |
| **IDS**                 | Detects suspicious or malicious activity                                                       |
| **IPS**                 | Detects **and can prevent/block** malicious activity inline                                    |
| **IPS Engine**          | Core inspection engine responsible for protocol decoding, signatures, heuristics and detection |
| **Signature**           | Pattern/rule used to identify known attacks                                                    |
| **Protocol Decoder**    | Understands protocol structure and validates protocol behavior                                 |
| **DPI**                 | Deep Packet Inspection of packet headers and payload                                           |
| **Heuristics**          | Identifies suspicious behavior without relying exclusively on static signatures                |
| **Threat Intelligence** | Uses external/internal intelligence to improve detection                                       |
| **CVE**                 | Standard identifier for publicly known vulnerabilities                                         |
| **NVD**                 | National Vulnerability Database containing vulnerability information                           |
| **SCAP**                | Security Content Automation Protocol used to standardize security content                      |
| **Sandbox**             | Executes suspicious files/objects in an isolated environment                                   |
| **Zero-Day Detection**  | Detection of previously unknown or insufficiently signed threats                               |
| **Aho-Corasick**        | Efficient multi-pattern string matching algorithm                                              |
| **Bloom Filter**        | Probabilistic data structure useful for fast membership checks                                 |

---

# 1. IDS vs IPS

The fundamental difference:

```text
IDS
 │
 ├── Inspect traffic
 ├── Detect suspicious activity
 └── Alert / Log
```

```text
IPS
 │
 ├── Inspect traffic
 ├── Detect suspicious activity
 ├── Log
 └── Block / Prevent
```

### Inline Security Model

```text
                Traffic
                   │
                   ▼
              ┌─────────┐
              │ FortiGate│
              │   IPS    │
              └────┬────┘
                   │
          ┌────────┴────────┐
          ▼                 ▼
       Malicious           Clean
          │                 │
          ▼                 ▼
       BLOCK              ALLOW
```

> **Key concept:** An inline IPS can take an enforcement action against detected traffic; IDS traditionally focuses on detection and alerting.

---

# 2. Evolution of Network Intrusion Detection

Network security inspection evolved significantly as attacks became more sophisticated.

### Early Generation

Early inspection mechanisms primarily focused on:

```text
Source IP
Destination IP
Protocol
Port
Connection State
```

Conceptually:

```text
Packet
 │
 ├── Source
 ├── Destination
 ├── Port
 └── State
```

This was useful for identifying basic network behavior, but it was insufficient against application-layer attacks.

---

# 3. Why Deep Packet Inspection Became Necessary

Attackers started hiding malicious behavior inside legitimate protocols.

Examples:

```text
HTTP
 └── SQL Injection

HTTP
 └── XSS

SMB
 └── Exploitation

SMTP
 └── Malicious Attachment

DNS
 └── Tunneling
```

Therefore, inspection moved deeper:

```text
L2
 ↓
L3
 ↓
L4
 ↓
L7
 ↓
Protocol Structure
 ↓
Payload
```

### Deep Packet Inspection

DPI allows the security engine to inspect:

* Packet headers
* Protocol fields
* Application-layer metadata
* Parameters
* Payloads
* Protocol deviations
* Attack patterns

---

# 4. FortiGate IPS Engine

FortiGate IPS is not simply a collection of signatures.

The IPS engine can use multiple detection technologies:

```text
                 IPS ENGINE
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
   Signatures    Protocol       Heuristics
                  Decoders
       │             │             │
       └─────────────┼─────────────┘
                     ▼
             Threat Intelligence
                     │
                     ▼
             Detection / Action
```

### Major Detection Components

* Signature matching
* Protocol decoders
* Heuristic detection
* Threat intelligence
* Vulnerability/CVE mapping
* Advanced threat detection

---

# 5. IPS Signatures

A signature is essentially a detection rule designed to identify known malicious behavior.

Conceptually:

```text
Traffic
   │
   ▼
Protocol Decoder
   │
   ▼
Signature Matching
   │
   ▼
Match?
 ┌─┴─┐
Yes  No
 │    │
 ▼    ▼
Action Continue
```

A signature may inspect:

```text
Protocol
Direction
Content
Methods
Parameters
Payload
Flags
Patterns
```

---

# 6. Protocol Decoders

A protocol decoder understands how a specific protocol is supposed to work.

For example, HTTP contains methods such as:

```text
GET
POST
PUT
DELETE
HEAD
OPTIONS
PATCH
```

A decoder can understand:

```text
HTTP
 │
 ├── Method
 ├── URI
 ├── Headers
 ├── Parameters
 ├── Content-Type
 └── Payload
```

This provides the foundation for higher-level attack detection.

---

# 7. Protocol Deviation Detection

An IPS engine can compare observed traffic against expected protocol behavior.

Example:

```text
HTTP Request
     │
     ▼
HTTP Decoder
     │
     ├── Method
     ├── URI
     ├── Headers
     └── Parameters
            │
            ▼
      Is behavior valid?
```

Suspicious characteristics may include:

* Invalid protocol syntax
* Unexpected methods
* Malformed headers
* Abnormal parameter values
* Protocol violations
* Suspicious payload structures

---

# 8. SQL Injection Detection Example

Consider:

```http
GET /index.php?id=1' OR '1'='1 HTTP/1.1
```

A simplistic signature could search for suspicious patterns.

Example Snort-style rule:

```text
alert tcp $EXTERNAL_NET any -> $HTTP_SERVERS 80 (
    msg:"SQL Injection Attack Detected";
    flow:to_server,established;
    content:"SELECT"; nocase;
    content:"FROM"; nocase;
    pcre:"/(%27)|(')1(-1-)|(%23)|(#)/i";
    sid:1000001;
    rev:1;
)
```

### Detection Flow

```text
HTTP Packet
    │
    ▼
HTTP Decoder
    │
    ▼
Extract Parameters
    │
    ▼
Pattern Matching
    │
    ▼
Signature / PCRE
    │
    ▼
Detection
```

---

# 9. PCRE-Based Detection

Regular expressions can identify complex attack patterns.

Common SQL-injection-related encodings/patterns include:

```text
%27
'
--
%23
#
```

For example:

```text
pcre:"/(\%27)|(\')|(\-\-)|(\%23)|(#)/i";
```

### Why PCRE?

Instead of checking only one literal string:

```text
SELECT
```

a regular expression can match multiple representations of suspicious input.

```text
Literal Match
     ↓
Simple

Regex / PCRE
     ↓
Flexible Pattern Matching
```

> **Important:** A single regex should not be treated as a complete SQL injection detector. Modern IPS engines combine protocol awareness, normalization, signatures and multiple detection mechanisms.

---

# 10. URL Encoding and Normalization

Attackers may encode malicious characters:

```text
'
```

as:

```text
%27
```

or:

```text
#
```

as:

```text
%23
```

Therefore, effective inspection generally requires normalization before or during signature matching.

Conceptually:

```text
Raw Request
    │
    ▼
Decode / Normalize
    │
    ▼
Protocol Decoder
    │
    ▼
Signature Matching
```

This is one reason protocol decoding is critical to modern IPS engines.

---

# 11. DPI Inspection Model

Deep inspection can be visualized as:

```text
Packet
  │
  ▼
┌─────────────────────┐
│ Packet Headers      │
├─────────────────────┤
│ Protocol Headers    │
├─────────────────────┤
│ Application Fields  │
├─────────────────────┤
│ Parameters          │
├─────────────────────┤
│ Payload              │
└─────────────────────┘
          │
          ▼
      IPS Engine
```

The goal is to understand **what the traffic actually contains**, rather than relying only on IP/port information.

---

# 12. Signature Database vs CVE Database

These concepts are related but not identical.

### Signature Database

Used for:

```text
Attack Detection
Malware Patterns
Protocol Exploits
Exploit Techniques
Known Malicious Traffic
```

### CVE Database

Used to identify vulnerabilities:

```text
CVE-YYYY-NNNN
```

Example:

```text
Vulnerable Software
       │
       ▼
      CVE
       │
       ▼
Related Vulnerability
       │
       ▼
IPS Signature
       │
       ▼
Attack Detection
```

> A CVE identifies a vulnerability; an IPS signature is a detection mechanism. One CVE can be associated with multiple detection signatures or techniques.

---

# 13. NVD — National Vulnerability Database

The **National Vulnerability Database (NVD)** provides structured vulnerability information.

Conceptually:

```text
Vulnerability
      │
      ▼
     CVE
      │
      ▼
     NVD
      │
      ├── Description
      ├── Severity
      ├── References
      └── Vulnerability Metadata
```

---

# 14. SCAP

**Security Content Automation Protocol (SCAP)** is a collection of standards designed to automate security measurement and vulnerability management.

Conceptually:

```text
Security Content
      │
      ▼
     SCAP
      │
      ├── Vulnerability Data
      ├── Configuration Data
      └── Security Automation
```

> **NVD ≠ SCAP.** NVD is a vulnerability database; SCAP is a broader set of standards for automated security configuration and vulnerability management.

---

# 15. Heuristic Detection

Signature-based detection works very well for known threats.

But what about unknown or modified attacks?

```text
Known Attack
    ↓
Signature
    ↓
Easy Detection
```

Modified/unknown attack:

```text
Unknown Attack
      ↓
No Exact Signature
      ↓
Behavior / Structure Analysis
      ↓
Heuristic Detection
```

Heuristics may consider:

* Abnormal protocol behavior
* Suspicious patterns
* Unusual payload structures
* Exploit-like behavior
* Multiple correlated indicators

---

# 16. Threat Intelligence

Threat intelligence adds external or internally generated security knowledge.

Conceptually:

```text
                Threat Intelligence
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
       IPs            URLs          Hashes
        │              │              │
        └──────────────┼──────────────┘
                       ▼
                 IPS Decision
```

Examples:

```text
Malicious IP
Malicious Domain
Malware Hash
Known Exploit
Botnet Infrastructure
```

---

# 17. Advanced Threat Detection

Modern security inspection moved beyond static signatures.

```text
Static Signature
       ↓
Protocol Analysis
       ↓
Behavior Analysis
       ↓
Threat Intelligence
       ↓
Sandbox / Emulation
       ↓
Machine Learning
```

The exact capabilities depend on the FortiOS release, FortiGuard services and licensed security features.

---

# 18. Zero-Day Detection

A zero-day attack may have:

```text
No Existing Signature
        ↓
Unknown Exploit
        ↓
Traditional Signature Detection
        ↓
Potential Miss
```

Therefore, modern security architectures use multiple layers:

```text
             Zero-Day Defense
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼
   Heuristics    Sandbox     Behavioral
                              Analysis
       │            │            │
       └────────────┼────────────┘
                    ▼
              Risk Detection
```

---

# 19. Sandbox and the APT Era

The emergence of sophisticated attacks increased demand for technologies capable of detecting previously unknown malware.

A typical sandbox workflow:

```text
Suspicious File
      │
      ▼
Network Security Device
      │
      ▼
Sandbox / Emulation
      │
      ▼
Execute in Isolation
      │
      ▼
Behavior Analysis
      │
      ▼
Malicious?
   ┌──┴──┐
  Yes    No
   │      │
 Alert   Allow
```

The sandbox approach became especially important for:

* Malicious documents
* Executables
* Unknown malware
* Zero-day behavior
* APT-style attacks

---

# 20. RSA Breach and the APT Concept

The 2011 RSA breach became an important milestone in the industry's understanding of targeted attacks.

A simplified attack chain:

```text
Phishing
   ↓
Malicious Document
   ↓
User Opens Document
   ↓
Malware Execution
   ↓
Initial Compromise
   ↓
Further Attack Activity
```

This highlighted a major limitation of pure signature-based detection:

> What happens when the security product has never seen the malware before?

This drove greater adoption of:

* Sandboxing
* Emulation
* Behavioral analysis
* Threat intelligence
* Advanced malware detection

---

# 21. Hash-Based Malware Detection

A file can be represented by a cryptographic hash:

```text
File
 │
 ▼
Hash Function
 │
 ▼
MD5 / SHA
 │
 ▼
Unique Digest
```

Conceptually:

```text
Known Bad File
      │
      ▼
   SHA-256
      │
      ▼
Known Malicious Hash
```

When a matching hash is observed:

```text
Incoming File
      │
      ▼
Calculate Hash
      │
      ▼
Compare Database
      │
   ┌──┴──┐
 Match  No Match
   │       │
   ▼       ▼
 Block   Continue
```

> Hash matching is extremely fast, but it is primarily useful for **known exact content**. Changing the file changes its cryptographic hash.

---

# 22. Why Hashes Are Not Enough

An attacker can modify a malicious file:

```text
Malware v1
   ↓
Modify
   ↓
Malware v2
   ↓
Different Hash
```

Therefore:

```text
Hash
 +
Signature
 +
Protocol Analysis
 +
Heuristics
 +
Behavior Analysis
 +
Threat Intelligence
 +
Sandbox
```

provides a stronger layered defense.

---

# 23. APT Detection Architecture

A modern security architecture can combine:

```text
Internet
   │
   ▼
FortiGate
   │
   ├── IPS
   │    ├── Signatures
   │    ├── Protocol Decoders
   │    ├── Heuristics
   │    └── Threat Intelligence
   │
   ├── AV
   │
   └── Sandbox
   │
   ▼
Internal Network
```

---

# 24. NGFW Evolution

Next-generation firewalls expanded traditional firewall capabilities.

Traditional firewall:

```text
IP
Port
Protocol
State
```

NGFW:

```text
IP
Port
Protocol
State
+
Application Control
+
IPS
+
Identity
+
Web Filtering
+
Antivirus
+
SSL Inspection
+
Sandbox
+
Threat Intelligence
```

This allowed firewalls to inspect traffic at much deeper levels.

---

# 25. Identity and Context

Modern security systems also need to understand **who** is generating traffic.

Instead of:

```text
192.168.10.20 → Internet
```

the security engine may have context such as:

```text
User
Device
Application
Location
Identity
Destination
Risk
```

Conceptually:

```text
              Security Context
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
       User        Device       App
        │            │            │
        └────────────┼────────────┘
                     ▼
                 Security
                  Policy
```

---

# 26. Machine Learning and Anomaly Detection

As machine learning became more practical, security platforms started moving beyond purely static rules.

Traditional:

```text
IF signature matches
THEN attack
```

Modern:

```text
Observe
  ↓
Learn
  ↓
Model Normal Behavior
  ↓
Detect Anomaly
  ↓
Calculate Risk
  ↓
Take Action
```

Potential indicators:

* Unusual traffic patterns
* Protocol anomalies
* Unexpected application behavior
* Abnormal communication relationships
* Suspicious sequences

---

# 27. EternalBlue — SMB Exploitation

**EternalBlue** is a well-known exploit targeting a vulnerability in **SMBv1** on vulnerable Microsoft Windows systems.

It became particularly significant because it enabled remote exploitation and was later used in major malware campaigns.

Simplified:

```text
Attacker
   │
   ▼
SMBv1
   │
   ▼
Vulnerable Windows Host
   │
   ▼
Remote Code Execution
```

---

# 28. EternalBlue Attack Chain

A simplified conceptual chain:

```text
SMB Discovery / Fingerprinting
          │
          ▼
Check Vulnerable Target
          │
          ▼
Exploit SMBv1 Vulnerability
          │
          ▼
Code Execution
          │
          ▼
Payload / Backdoor
          │
          ▼
Lateral Movement
```

> EternalBlue exploited a vulnerability in SMBv1. It should not be confused with DoublePulsar, which was a separate backdoor/implant associated with later exploitation activity.

---

# 29. DoublePulsar

DoublePulsar was a backdoor associated with exploitation of Windows systems.

A simplified relationship:

```text
EternalBlue
     │
     ▼
Initial Exploitation
     │
     ▼
DoublePulsar
     │
     ▼
Backdoor / Further Access
```

### Important Correction

```text
EternalBlue ≠ DoublePulsar
```

They are separate components with different roles.

---

# 30. EternalBlue and Lateral Movement

One of the major security concerns around SMB exploitation is propagation.

Conceptually:

```text
             Compromised Host
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
        Host A    Host B    Host C
          │         │
          ▼         ▼
        More      More
       Hosts     Hosts
```

This type of propagation can turn a single compromise into a large-scale incident.

---

# 31. Heap Spraying

Heap spraying is a memory exploitation technique.

Conceptually:

```text
Process Heap
┌───────────────────────────────┐
│ Attacker-controlled data      │
│ Attacker-controlled data      │
│ Attacker-controlled data      │
│ Attacker-controlled data      │
│ Attacker-controlled data      │
└───────────────────────────────┘
```

The attacker attempts to increase the probability that useful attacker-controlled data occupies predictable memory locations.

### Kernel Exploitation

Some exploitation techniques manipulate kernel memory allocation behavior to make exploitation more reliable.

> Heap spraying is a general exploitation technique and is not synonymous with EternalBlue itself.

---

# 32. Fast Pattern Matching

IPS engines may need to compare traffic against a large number of patterns.

Naive approach:

```text
Packet
  │
  ├── Signature 1
  ├── Signature 2
  ├── Signature 3
  ├── Signature 4
  ├── ...
  └── Signature N
```

This can become expensive.

Therefore, optimized pattern-matching algorithms can improve efficiency.

---

# 33. Aho-Corasick Algorithm

Aho-Corasick is a multi-pattern string matching algorithm.

Instead of independently scanning for every pattern:

```text
Pattern 1
Pattern 2
Pattern 3
...
Pattern N
```

it builds a structure that can search for many patterns efficiently.

Conceptually:

```text
             Traffic
                │
                ▼
        Aho-Corasick Engine
                │
      ┌─────────┼─────────┐
      ▼         ▼         ▼
   Pattern A Pattern B Pattern C
      │         │         │
      └─────────┼─────────┘
                ▼
             Match
```

This type of optimization is valuable in high-speed security inspection.

---

# 34. Bloom Filters

A Bloom filter is a probabilistic data structure used for fast membership testing.

Example:

```text
Incoming Hash
      │
      ▼
Bloom Filter
      │
 ┌────┴────┐
 ▼         ▼
Maybe     Definitely
Present   Not Present
 │
 ▼
Detailed Check
```

### Key Property

A Bloom filter can produce:

```text
False Positive → Possible
False Negative → Normally no
```

Therefore it can be used as a fast preliminary check before a more expensive lookup.

---

# 35. Fast Packet I/O

High-performance security systems also optimize packet processing.

Conceptually:

```text
Traditional Packet Path
NIC
 ↓
Kernel
 ↓
Network Stack
 ↓
Application
```

High-performance packet frameworks can reduce overhead:

```text
NIC
 ↓
Fast Packet I/O
 ↓
Inspection Engine
 ↓
Decision
```

**Netmap**, for example, is a high-performance packet I/O framework commonly associated with BSD/Linux networking research and high-speed packet processing.

---

# 36. IPS Detection Pipeline

A useful mental model:

```text
                    PACKET
                       │
                       ▼
                Packet Parsing
                       │
                       ▼
                Protocol Decoder
                       │
                       ▼
                  Normalize
                       │
                       ▼
              Extract Parameters
                       │
                       ▼
              Signature Matching
                       │
              ┌────────┴────────┐
              ▼                 ▼
          Heuristics       Threat Intel
              │                 │
              └────────┬────────┘
                       ▼
                  Risk Decision
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
           ALLOW                BLOCK
```

---

# 37. IPS + CVE Mapping

A simplified relationship:

```text
Vulnerability
      │
      ▼
     CVE
      │
      ▼
Vendor Research
      │
      ▼
Detection Logic
      │
      ▼
IPS Signature
      │
      ▼
Network Detection
```

This allows security teams to connect:

```text
Vulnerability
     ↕
CVE
     ↕
Exploit
     ↕
IPS Signature
```

---

# 38. Why IPS Can Detect Without Knowing the Exact CVE

A packet does not necessarily contain:

```text
"CVE-2026-XXXX"
```

Instead, the IPS engine identifies exploit characteristics.

For example:

```text
Protocol
 +
Malformed Field
 +
Specific Payload
 +
Known Exploit Pattern
```

may be sufficient to identify malicious traffic.

Therefore:

```text
CVE
   ≠
Packet Signature
```

A CVE is vulnerability metadata; the IPS signature is the network detection mechanism.

---

# 39. IPS Action Model

When a signature matches:

```text
Signature Match
      │
      ▼
Action
 │
 ├── Pass
 ├── Monitor
 ├── Block
 ├── Reset
 └── Quarantine / Other supported action
```

The exact available actions depend on the FortiOS release and IPS configuration.

---

# 40. IPS vs Application Control

These two technologies are related but solve different problems.

| Feature                 | Primary Goal                                           |
| ----------------------- | ------------------------------------------------------ |
| **Application Control** | Identify and control applications                      |
| **IPS**                 | Detect/prevent exploits and malicious network behavior |
| **Web Filter**          | Control web destinations/categories                    |
| **Antivirus**           | Detect malicious files                                 |
| **DLP**                 | Detect/prevent sensitive data leakage                  |
| **Sandbox**             | Analyze suspicious files/objects                       |
| **Firewall Policy**     | Control network access                                 |

A modern security policy often combines several of them.

---

# 41. Deep Inspection Requirement

When using IPS, the inspection engine needs access to the relevant traffic content.

Conceptually:

```text
Firewall
   │
   ▼
Traffic Inspection
   │
   ▼
IPS Engine
   │
   ▼
Protocol + Payload Analysis
```

For encrypted traffic:

```text
Encrypted Traffic
       │
       ▼
Cannot directly inspect plaintext
       │
       ▼
SSL Inspection
       │
       ▼
Decrypted Content
       │
       ▼
IPS
```

> SSL inspection requirements depend on the traffic, inspection mode and FortiOS configuration.

---

# 42. IPS and HTTPS

HTTPS creates an important visibility challenge.

```text
Client
   │
   │ HTTPS
   ▼
FortiGate
   │
   ▼
Encrypted Payload
```

Without appropriate decryption/inspection:

```text
IPS
 ↓
Limited visibility
```

With SSL inspection where supported and correctly configured:

```text
HTTPS
 ↓
SSL Inspection
 ↓
Decrypted Traffic
 ↓
IPS
```

---

# 43. Common IPS Attack Categories

```text
IPS
 │
 ├── SQL Injection
 ├── XSS
 ├── Buffer Overflow
 ├── Remote Code Execution
 ├── Protocol Exploitation
 ├── SMB Exploitation
 ├── Web Exploitation
 ├── Scanning
 ├── Denial of Service
 └── Malware Communication
```

---

# 44. Zero-Day vs Known Exploit

### Known Exploit

```text
Known Vulnerability
      ↓
Known Exploit
      ↓
Known Pattern
      ↓
IPS Signature
      ↓
Detect / Block
```

### Zero-Day

```text
Unknown Vulnerability
       ↓
Unknown Exploit
       ↓
No Exact Signature
       ↓
Heuristics / Behavior / Sandbox
       ↓
Potential Detection
```

---

# 45. 🧪 Practical IPS Investigation Flow

When investigating a suspected attack:

```text
1. Identify source
        ↓
2. Identify destination
        ↓
3. Identify application/protocol
        ↓
4. Decode protocol
        ↓
5. Inspect parameters
        ↓
6. Check IPS signature
        ↓
7. Map vulnerability/CVE
        ↓
8. Check threat intelligence
        ↓
9. Check logs
        ↓
10. Determine action
```

---

# 46. 🔍 FortiGate IPS Troubleshooting Checklist

```text
[ ] IPS feature enabled
[ ] Correct firewall policy
[ ] IPS security profile attached
[ ] Correct IPS sensor configured
[ ] Inspection mode verified
[ ] SSL inspection considered for HTTPS
[ ] Application identified
[ ] Protocol decoder working
[ ] Signature enabled
[ ] Signature severity reviewed
[ ] Signature action reviewed
[ ] Logs enabled
[ ] Traffic actually reaches the policy
[ ] FortiGuard connectivity verified where required
[ ] Relevant CVE/signature relationship checked
```

---

# 47. 🛡️ Recommended IPS Design

A practical enterprise security policy:

```text
                    INTERNET
                        │
                        ▼
                 ┌─────────────┐
                 │  FortiGate  │
                 │             │
                 │ Firewall    │
                 │     ↓       │
                 │ SSL Inspect │
                 │     ↓       │
                 │    IPS      │
                 │     ↓       │
                 │     AV      │
                 │     ↓       │
                 │ Application │
                 │   Control   │
                 └──────┬──────┘
                        │
                        ▼
                 INTERNAL USERS
```

---

# 48. ⚠️ Common IPS Design Mistakes

### ❌ Relying only on signatures

```text
Signatures Only
      ↓
Known Threats
```

Modern security requires multiple detection layers.

---

### ❌ Forgetting encrypted traffic

```text
HTTPS
 ↓
Encrypted
 ↓
No plaintext visibility
```

---

### ❌ Treating CVE as an IPS signature

```text
CVE ≠ Signature
```

CVE identifies a vulnerability.

Signature identifies traffic characteristics associated with malicious activity.

---

### ❌ Blocking everything suspicious

Aggressive IPS policies can cause:

```text
False Positives
      ↓
Legitimate Traffic Blocked
      ↓
Application Failure
```

Always validate:

* Severity
* Signature
* False positives
* Business impact
* Recommended action

---

### ❌ Ignoring protocol decoders

A signature without proper protocol understanding can be less effective against encoded or obfuscated attacks.

```text
Raw Packet
   ↓
Decode
   ↓
Normalize
   ↓
Inspect
```

---

# 49. 🔥 Fast NSE Exam Notes

| Topic                   | Remember                                               |
| ----------------------- | ------------------------------------------------------ |
| **IDS**                 | Detect + alert                                         |
| **IPS**                 | Detect + prevent/block inline                          |
| **DPI**                 | Inspects deeper packet/application content             |
| **Signature**           | Detects known patterns                                 |
| **Protocol Decoder**    | Understands protocol structure                         |
| **Heuristics**          | Helps detect suspicious/unknown behavior               |
| **Threat Intelligence** | Adds external/internal threat knowledge                |
| **CVE**                 | Vulnerability identifier                               |
| **NVD**                 | Vulnerability database                                 |
| **SCAP**                | Security automation standards                          |
| **Sandbox**             | Isolated execution/analysis                            |
| **Zero-Day**            | Previously unknown or insufficiently signed threat     |
| **Hash**                | Identifies exact content efficiently                   |
| **Aho-Corasick**        | Multi-pattern matching                                 |
| **Bloom Filter**        | Fast probabilistic membership check                    |
| **EternalBlue**         | SMBv1 exploitation                                     |
| **DoublePulsar**        | Backdoor associated with Windows exploitation          |
| **HTTPS**               | Requires appropriate inspection for payload visibility |
| **SQL Injection**       | Application-layer attack                               |
| **XSS**                 | Application-layer web attack                           |

---

# 50. 🧠 Most Important IPS Mental Model

```text
                     IPS ENGINE
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
    Protocol          Signature        Heuristic
    Decoder           Matching         Analysis
        │                │                │
        └────────────────┼────────────────┘
                         ▼
                 Threat Intelligence
                         │
                         ▼
                   Risk / Detection
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
           Benign                Malicious
              │                     │
              ▼                     ▼
            ALLOW                 ACTION
                                    │
                         ┌──────────┼──────────┐
                         ▼          ▼          ▼
                       BLOCK      RESET       LOG
```

---

# 51. 🎯 Final Mental Model — From Packet to Protection

```text
                    NETWORK PACKET
                          │
                          ▼
                  ┌───────────────┐
                  │ Packet Parser │
                  └───────┬───────┘
                          │
                          ▼
                  Protocol Decoder
                          │
                          ▼
                     Normalize
                          │
                          ▼
                  Extract Fields
                          │
              ┌───────────┼───────────┐
              ▼           ▼           ▼
          Signature    Heuristic   Threat Intel
          Matching      Analysis
              │           │           │
              └───────────┼───────────┘
                          ▼
                    IPS Decision
                          │
             ┌────────────┴────────────┐
             ▼                         ▼
          BENIGN                    MALICIOUS
             │                         │
             ▼                         ▼
           ALLOW                  BLOCK / RESET
                                       │
                                       ▼
                                     LOG
```

---

# 52. 🚀 Production Security Stack

For modern enterprise deployments:

```text
                    INTERNET
                       │
                       ▼
                ┌──────────────┐
                │  FortiGate   │
                └──────┬───────┘
                       │
       ┌───────────────┼────────────────┐
       ▼               ▼                ▼
   SSL Inspection     IPS              AV
       │               │                │
       └───────────────┼────────────────┘
                       ▼
                Application Control
                       │
                       ▼
                  Web Filtering
                       │
                       ▼
                     DLP
                       │
                       ▼
                   Sandbox
                       │
                       ▼
                 Internal Network
```

The objective is not to rely on one detection technology:

```text
Firewall
   +
IPS
   +
AV
   +
Application Control
   +
Web Filter
   +
DLP
   +
Sandbox
   +
Threat Intelligence
```

This creates **defense in depth**.

---

# 🧩 Final SheynShield Engineering Mental Model

> **IPS is not simply “a database of signatures.”**

Think about the complete detection pipeline:

```text
                 TRAFFIC
                    │
                    ▼
             Protocol Decode
                    │
                    ▼
               Normalize
                    │
                    ▼
              Deep Inspect
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
       Signature  Heuristic  Threat
       Matching   Analysis   Intel
          │         │         │
          └─────────┼─────────┘
                    ▼
              Detection Engine
                    │
         ┌──────────┴──────────┐
         ▼                     ▼
       Known                 Unknown
       Threat                 Threat
         │                     │
      Signature          Heuristic /
      Detection          Sandbox /
                         Behavior
         │                     │
         └──────────┬──────────┘
                    ▼
                 ACTION
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
        ALLOW      LOG       BLOCK
```

### The core idea

```text
Protocol Understanding
        +
Deep Packet Inspection
        +
Signature Matching
        +
Heuristics
        +
Threat Intelligence
        +
Advanced Threat Detection
        =
Modern IPS
```

---

## 🔗 Related FortiGate Topics

* **Firewall Policy & Inspection Modes**
* **SSL/Deep Inspection**
* **Application Control**
* **Antivirus**
* **Web Filter**
* **DLP**
* **FortiSandbox**
* **Security Fabric**
* **FortiGuard Security Services**
* **IPS Diagnostics & Troubleshooting**

---

> **SheynShield Engineering Note**
>
> When troubleshooting IPS, don't start with the signature alone. Start with the **traffic path**:
>
> **Policy → Inspection Mode → Encryption → Protocol Decoder → Normalization → Signature/Heuristic Detection → Action → Log**
>
> If you understand this pipeline, IPS troubleshooting becomes much more systematic—and you can distinguish a **visibility problem**, a **detection problem**, and an **enforcement problem** instead of simply assuming that “the IPS didn't detect it.”

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

# FortiGate Web Application Firewall (WAF) —  

> **FortiOS Focus:** Web Application Firewall, WAF Policy, VIP, Reverse Proxy, HTTP/HTTPS Protection, Security Fabric
> **Audience:** FortiGate / NSE4–NSE7 / Network & Security Engineers
> **Primary Use Cases:** Web Application Protection, SQL Injection, HTTP Attack Protection, Brute-Force Mitigation
> **Inspection Mode:** Proxy-based

---

## 📌 Quick Reference

| Component            | Purpose                                                                       |
| -------------------- | ----------------------------------------------------------------------------- |
| **WAF**              | Protects web applications against common HTTP/HTTPS application-layer attacks |
| **VIP**              | Publishes an internal web server through a FortiGate address                  |
| **Proxy Inspection** | Required for proxy-based application-layer inspection                         |
| **Deep Inspection**  | Enables inspection of encrypted HTTPS traffic when applicable                 |
| **WAF Profile**      | Defines web application security controls                                     |
| **Security Fabric**  | Integrates FortiGate with other Fabric components and connectors              |

---

# 1. What Is FortiGate WAF?

**Web Application Firewall (WAF)** provides application-layer protection for HTTP/HTTPS services.

Typical deployment:

```text
                    INTERNET
                       │
                       │ HTTP / HTTPS
                       ▼
                ┌───────────────┐
                │   FortiGate   │
                │               │
                │      VIP      │
                │       │       │
                │      WAF      │
                │       │       │
                │ Proxy Inspect │
                └───────┬───────┘
                        │
                        ▼
                 Web Application
                 192.168.20.200
```

The FortiGate acts as a security gateway between external clients and the protected web application.

---

# 2. Enable WAF Feature

If WAF options are not visible in the GUI:

```text
System
  └── Feature Visibility
        └── Web Application Firewall
```

Enable:

```text
Web Application Firewall
```

After enabling the feature, WAF-related configuration becomes available.

---

# 3. Common WAF Use Cases

FortiGate WAF can provide protection against application-layer threats such as:

```text
Web Application
      │
      ├── SQL Injection
      ├── HTTP attacks
      ├── Malicious requests
      ├── Parameter manipulation
      ├── Protocol violations
      └── Application-layer abuse
```

### Example Attack

```text
Attacker
   │
   │ Malicious HTTP Request
   ▼
FortiGate WAF
   │
   ├── Inspect
   ├── Detect
   └── Block
   │
   X
Web Server
```

> WAF should be considered an **application-layer security control**, not a replacement for network-layer security controls such as firewall policies, IPS, authentication, secure application design, and patching.

---

# 4. WAF + VIP Architecture

A common Internet-facing design uses a **Virtual IP (VIP)**.

```text
                    INTERNET
                        │
                        │
                 192.168.101.101
                        │
                        ▼
                  ┌──────────┐
                  │ FortiGate│
                  │          │
                  │   VIP    │
                  └────┬─────┘
                       │
                       │ DNAT
                       ▼
                192.168.20.200
                  Web Server
```

Example:

```text
External IP : 192.168.101.101
Mapped IP   : 192.168.20.200
```

> These addresses are RFC1918 examples for lab/documentation purposes. A production Internet-facing VIP normally uses a publicly routed address.

---

# 5. Configure the VIP

Conceptually:

```text
VIP
├── Interface       : LAN / WAN-facing interface according to topology
├── External IP     : 192.168.101.101
└── Mapped IP       : 192.168.20.200
```

Traffic flow:

```text
Client
  │
  │ HTTP / HTTPS
  ▼
192.168.101.101
  │
  │ VIP / DNAT
  ▼
192.168.20.200
```

The VIP maps the externally reachable address to the internal web server.

---

# 6. WAF Firewall Policy

The WAF profile must be attached to the firewall policy handling the web traffic.

Example architecture:

```text
Source
  │
  ▼
Destination VIP
  │
  ▼
Firewall Policy
  │
  ├── Proxy Inspection
  │
  ├── WAF Profile
  │
  └── Deep Inspection
  │
  ▼
Web Server
```

### Conceptual Policy

```text
Source      : all
Destination : Web Server VIP
Service     : HTTP / HTTPS
Inspection  : Proxy
Security    : WAF
SSL         : Deep Inspection where required
```

---

# 7. Proxy Mode + WAF

WAF operates as an application-layer inspection mechanism and is associated with **proxy-based processing**.

Conceptually:

```text
Client
  │
  ▼
FortiGate
  │
  ▼
Proxy
  │
  ├── HTTP parsing
  ├── HTTPS inspection
  ├── WAF inspection
  └── Security controls
  │
  ▼
Web Server
```

### Key Point

```text
WAF
 ↓
Application Layer
 ↓
HTTP/HTTPS
 ↓
Proxy Inspection
```

> When configuring WAF, verify that the firewall policy uses the inspection mode and security profile combination supported by the FortiOS release and the selected WAF features.

---

# 8. HTTPS + WAF

HTTPS encrypts application traffic:

```text
Client
   │
   │ HTTPS
   │ Encrypted
   ▼
FortiGate
   │
   │ SSL Inspection
   ▼
WAF Inspection
   │
   ▼
Web Server
```

Without visibility into encrypted application data, application-layer inspection is limited.

Therefore, depending on the deployment:

```text
HTTPS
  ↓
SSL Inspection
  ↓
HTTP/Application Visibility
  ↓
WAF
```

### Security Trade-off

More inspection generally means:

```text
More visibility
      ↓
More processing
      ↓
Potentially higher CPU / memory usage
```

SSL inspection should therefore be designed and tested carefully.

---

# 9. WAF Security Stack

A strong Internet-facing web application policy can combine multiple security controls:

```text
                    Client
                       │
                       ▼
                  FortiGate
                       │
                ┌──────┴──────┐
                ▼             ▼
              VIP        SSL Inspection
                │             │
                └──────┬──────┘
                       ▼
                      WAF
                       │
                       ▼
                      IPS
                       │
                       ▼
                Web Application
```

### Defense in Depth

```text
Firewall
   +
VIP
   +
SSL Inspection
   +
WAF
   +
IPS
   +
Logging / Monitoring
```

---

# 10. WAF vs IPS

WAF and IPS protect different layers and should not be treated as identical controls.

| Security Control   | Primary Focus                                                   |
| ------------------ | --------------------------------------------------------------- |
| **Firewall**       | Network/session access control                                  |
| **IPS**            | Known exploits, malicious patterns, network/application attacks |
| **WAF**            | Web application HTTP/HTTPS attack protection                    |
| **SSL Inspection** | Visibility into encrypted traffic                               |
| **Authentication** | Identity and access control                                     |

Simplified:

```text
Firewall
   ↓
Can this connection pass?

IPS
   ↓
Does the traffic match a known malicious pattern?

WAF
   ↓
Is this HTTP/HTTPS request malicious?

Application
   ↓
Is the application itself securely implemented?
```

---

# 11. WAF and SQL Injection

One important WAF use case is protection against **SQL Injection (SQLi)**.

Example:

```text
Normal Request:

GET /product?id=100
```

Potentially malicious request:

```text
GET /product?id=100' OR '1'='1
```

Conceptually:

```text
HTTP Request
      │
      ▼
     WAF
      │
      ├── Parse request
      ├── Inspect parameters
      ├── Match attack patterns
      └── Apply configured action
      │
      ├── Allow
      └── Block
```

> WAF rules should be tuned for the actual application. Poorly tuned signatures can create false positives.

---

# 12. Brute-Force Protection

WAF can also contribute to protecting web applications against application-layer abuse such as repeated login attempts, depending on the configured WAF capabilities and FortiOS version.

Example:

```text
Attacker
   │
   ├── Login Attempt 1
   ├── Login Attempt 2
   ├── Login Attempt 3
   ├── Login Attempt 4
   └── ...
        │
        ▼
       WAF
        │
        ├── Detect
        ├── Rate / rule enforcement
        └── Block / action
```

> For authentication brute-force protection, application-native controls, MFA, account lockout/rate limiting, and dedicated authentication protections should also be considered.

---

# 13. Production Policy Pattern

A typical Internet-facing web application policy:

```text
                    INTERNET
                       │
                       ▼
                 Public Web IP
                       │
                       ▼
                     VIP
                       │
                       ▼
              Firewall Policy
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
        Proxy         WAF        SSL Inspection
          │            │            │
          └────────────┼────────────┘
                       ▼
                      IPS
                       │
                       ▼
                 Web Application
```

---

# 14. Recommended Security Policy

### Source

Avoid unnecessarily broad Internet exposure.

Instead of:

```text
Source: ALL
```

consider:

```text
Trusted Networks
Known Partners
Required Internet Sources
```

when the application's business requirements allow it.

---

### Destination

Use the specific VIP:

```text
Destination:
Web-VIP
```

rather than unrestricted internal destinations.

---

### Services

Allow only required services:

```text
HTTP
HTTPS
```

Avoid:

```text
ALL
```

unless explicitly required.

---

### Inspection

Recommended conceptual stack:

```text
Proxy Inspection
       +
WAF
       +
SSL Inspection
       +
IPS
       +
Logging
```

---

# 15. Security Fabric Integration

In a **Security Fabric** environment, FortiGate can integrate with other Fabric components and connectors.

Conceptually:

```text
                    Security Fabric
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
    FortiGate          Fabric          Security
      WAF              Connector        Services
        │                 │                 │
        └─────────────────┼─────────────────┘
                          ▼
                   Central Visibility
```

Fabric integration can help provide:

* Centralized visibility
* Security telemetry
* Event correlation
* Integration with other security components
* Coordinated security operations

> The exact available Fabric connectors and offloading/integration capabilities depend on the FortiOS version and deployed Fortinet products.

---

# 16. WAF Traffic Flow — Mental Model

The most important concept:

```text
               WEB REQUEST
                    │
                    ▼
             Public VIP
                    │
                    ▼
              FortiGate
                    │
              ┌─────┴─────┐
              ▼           ▼
            Proxy       SSL
              │       Inspection
              └─────┬─────┘
                    ▼
                   WAF
                    │
            ┌───────┴────────┐
            ▼                ▼
          Allow             Block
            │                │
            ▼                X
       Web Server
```

---

# 17. 🔍 WAF Troubleshooting Checklist

## Web Application Not Reachable

```text
[ ] VIP exists
[ ] External IP is correct
[ ] Mapped IP is correct
[ ] Firewall policy matches
[ ] Correct service is allowed
[ ] Proxy inspection enabled
[ ] WAF profile attached
[ ] SSL inspection configured correctly
[ ] Routing is correct
[ ] Return traffic is correct
[ ] Web server is listening
```

---

## False Positive

If legitimate requests are blocked:

```text
Client
  │
  ▼
WAF
  │
  X
Blocked
```

Investigate:

```text
[ ] WAF event/log
[ ] Matched rule
[ ] URL
[ ] HTTP method
[ ] Parameters
[ ] Headers
[ ] Request body
[ ] Application behavior
```

Then determine whether the correct solution is:

```text
Tune rule
   ↓
Create exception
   ↓
Adjust policy
```

Avoid blindly disabling WAF protection.

---

# 18. Web Security Troubleshooting Flow

```text
Client
  │
  ▼
Public IP
  │
  ▼
VIP
  │
  ▼
Firewall Policy
  │
  ▼
Proxy Inspection
  │
  ▼
SSL Inspection
  │
  ▼
WAF
  │
  ▼
IPS
  │
  ▼
Web Server
```

When troubleshooting, identify **the exact layer where traffic stops**.

---

# 19. 🔥 Fast NSE Exam Notes

| Topic                | Key Point                                                  |
| -------------------- | ---------------------------------------------------------- |
| **WAF**              | Application-layer protection for web applications          |
| **VIP**              | Publishes/maps an external address to an internal server   |
| **Proxy Mode**       | Used for proxy-based application inspection                |
| **HTTPS**            | Requires appropriate SSL inspection for content visibility |
| **SQL Injection**    | Major WAF protection use case                              |
| **HTTP/HTTPS**       | Primary WAF traffic                                        |
| **IPS**              | Complements WAF but is not a WAF replacement               |
| **Security Fabric**  | Enables integration and centralized security visibility    |
| **False Positive**   | Investigate logs/rules before disabling protection         |
| **Defense in Depth** | Firewall + SSL inspection + WAF + IPS                      |

---

# 20. 🧠 WAF vs Firewall — Easy Memory Trick

```text
Firewall asks:

"WHO can connect?"

          ↓

WAF asks:

"WHAT is this web request trying to do?"
```

Example:

```text
Internet
   │
   ▼
Firewall
   │
   │ Allowed
   ▼
WAF
   │
   │ Inspect HTTP/HTTPS
   │
   ├── Legitimate Request → Allow
   │
   └── SQLi / Malicious Request → Block
```

---

# 21. ⚠️ Common WAF Design Mistakes

### ❌ Mistake 1 — No WAF on Public Web Applications

```text
Internet
   │
   ▼
VIP
   │
   ▼
Web Server
```

Better:

```text
Internet
   │
   ▼
VIP
   │
   ▼
WAF
   │
   ▼
Web Server
```

---

### ❌ Mistake 2 — HTTPS Without Appropriate Inspection

```text
HTTPS
   │
   ▼
Encrypted Payload
   │
   ▼
Limited Application Visibility
```

For environments requiring content inspection:

```text
HTTPS
   ↓
SSL Inspection
   ↓
WAF
```

---

### ❌ Mistake 3 — Using `ALL` Services

```text
Source: ALL
Destination: Web VIP
Service: ALL
```

Prefer:

```text
Source: Required
Destination: Web VIP
Service: HTTP / HTTPS
```

---

### ❌ Mistake 4 — Treating WAF as a Complete Security Solution

WAF does **not** replace:

```text
Secure Coding
Patch Management
Authentication
MFA
IPS
Firewall
Endpoint Security
Logging
Monitoring
```

---

# 22. 🎯 Production Reference Architecture

```text
                         INTERNET
                            │
                            ▼
                     Public Web IP
                            │
                            ▼
                     ┌────────────┐
                     │ FortiGate  │
                     │            │
                     │    VIP     │
                     │     │      │
                     │   Proxy    │
                     │     │      │
                     │ SSL Inspect│
                     │     │      │
                     │    WAF     │
                     │     │      │
                     │    IPS     │
                     └─────┬──────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │ Web Application  │
                  │ 192.168.20.200  │
                  └─────────────────┘
```

### Security Layers

```text
Public IP
    ↓
VIP
    ↓
Firewall Policy
    ↓
Proxy Inspection
    ↓
SSL Inspection
    ↓
WAF
    ↓
IPS
    ↓
Web Application
    ↓
Logging / Monitoring
```

---

# 23. 🧩 Final Mental Model

```text
                    FORTIGATE WAF
                          │
             ┌────────────┴────────────┐
             ▼                         ▼
         HTTP/HTTPS                 Security
             │                         │
             ▼                         ▼
            VIP                    Firewall
             │                         │
             └──────────┬──────────────┘
                        ▼
                  Proxy Inspection
                        │
                        ▼
                  SSL Inspection
                        │
                        ▼
                       WAF
                        │
              ┌─────────┴─────────┐
              ▼                   ▼
           Allowed              Blocked
              │                   │
              ▼                   X
        Web Application
              │
              ▼
          IPS / Logging
```

---

> **SheynShield Engineering Note**
>
> **WAF is not simply "another firewall rule."** The firewall policy decides whether the connection is permitted, while WAF examines the **web application request itself**.
>
> For an Internet-facing application, think in this order:
>
> **Client → VIP → Firewall Policy → Proxy → SSL Inspection → WAF → IPS → Web Server**
>
> When a web request fails, troubleshoot the chain layer-by-layer instead of immediately disabling WAF.

---

## 🔗 Related FortiGate Topics

* **VIP / DNAT** → Publishing internal services
* **Proxy Inspection** → Application-aware traffic processing
* **SSL Inspection** → HTTPS visibility
* **IPS** → Exploit and malicious traffic detection
* **Security Fabric** → Centralized security integration
* **Web Application Security** → WAF architecture and policy design

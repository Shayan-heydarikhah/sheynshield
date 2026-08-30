# 🔗 SheynShield Resources

# FortiGate WAF Security Checklist

> **FortiOS Focus:** Web Application Firewall (WAF) · VIP · DNAT · Reverse Proxy · HTTP/HTTPS · Proxy Inspection · SSL Inspection · IPS · Security Fabric
> **Audience:** FortiGate · NSE4 · NSE5 · NSE6 · NSE7 · Network & Security Engineers
> **Primary Use Cases:** Web Application Protection · SQL Injection · HTTP Attacks · Brute-Force Mitigation · Internet-Facing Applications
> **Inspection Mode:** Proxy-Based
> **Methodology:** Design → Configure → Secure → Verify → Troubleshoot

---

## 📋 Table of Contents

* [1. WAF Deployment Readiness](#1-waf-deployment-readiness)
* [2. Enable WAF Feature](#2-enable-waf-feature)
* [3. Web Application Architecture](#3-web-application-architecture)
* [4. VIP / DNAT Checklist](#4-vip--dnat-checklist)
* [5. Firewall Policy Checklist](#5-firewall-policy-checklist)
* [6. Proxy Inspection Checklist](#6-proxy-inspection-checklist)
* [7. HTTPS / SSL Inspection Checklist](#7-https--ssl-inspection-checklist)
* [8. WAF Profile Checklist](#8-waf-profile-checklist)
* [9. WAF Security Controls](#9-waf-security-controls)
* [10. WAF vs IPS](#10-waf-vs-ips)
* [11. SQL Injection Protection](#11-sql-injection-protection)
* [12. Brute-Force Protection](#12-brute-force-protection)
* [13. Security Fabric Integration](#13-security-fabric-integration)
* [14. Logging & Monitoring](#14-logging--monitoring)
* [15. Production Hardening](#15-production-hardening)
* [16. WAF Troubleshooting](#16-waf-troubleshooting)
* [17. False Positive Troubleshooting](#17-false-positive-troubleshooting)
* [18. HTTPS Troubleshooting](#18-https-troubleshooting)
* [19. VIP Troubleshooting](#19-vip-troubleshooting)
* [20. WAF Verification Checklist](#20-waf-verification-checklist)
* [21. Common WAF Design Mistakes](#21-common-waf-design-mistakes)
* [22. Production Reference Architecture](#22-production-reference-architecture)
* [23. Fast NSE Exam Checklist](#23-fast-nse-exam-checklist)
* [24. One-Minute Revision](#24-one-minute-revision)
* [25. Final WAF Mental Model](#25-final-waf-mental-model)

---

# 1. WAF Deployment Readiness

Before configuring WAF, verify the application architecture.

### Environment

* [ ] Identify the protected web application
* [ ] Identify the backend web server
* [ ] Identify application IP address
* [ ] Identify application FQDN
* [ ] Identify public IP
* [ ] Identify FortiGate ingress interface
* [ ] Identify FortiGate egress/backend interface
* [ ] Confirm routing
* [ ] Confirm return routing
* [ ] Identify HTTP/HTTPS services
* [ ] Identify application-specific ports
* [ ] Identify application owners
* [ ] Identify expected client sources

### Application Requirements

* [ ] HTTP required?
* [ ] HTTPS required?
* [ ] HTTP → HTTPS redirect required?
* [ ] WebSocket required?
* [ ] Large file uploads required?
* [ ] API endpoints present?
* [ ] Authentication/login portal present?
* [ ] Special HTTP methods required?
* [ ] Custom headers required?
* [ ] Large request bodies required?

> **Production Rule:** Understand the application before enabling aggressive WAF controls. Security policy must protect the application without breaking legitimate business traffic.

---

# 2. Enable WAF Feature

If WAF configuration is not visible in the GUI:

```text
System
   ↓
Feature Visibility
   ↓
Web Application Firewall
```

Checklist:

* [ ] Open Feature Visibility
* [ ] Locate Web Application Firewall
* [ ] Enable WAF
* [ ] Confirm WAF configuration is visible
* [ ] Confirm the deployed FortiOS release supports the required WAF features

> **NSE Note:** Feature availability and exact GUI/CLI behavior can vary between FortiOS releases.

---

# 3. Web Application Architecture

A common Internet-facing WAF design:

```text
                         INTERNET
                            │
                            │ HTTP / HTTPS
                            ▼
                    ┌─────────────────┐
                    │    FortiGate    │
                    │                 │
                    │      VIP        │
                    │       │         │
                    │  Firewall Policy│
                    │       │         │
                    │     Proxy       │
                    │       │         │
                    │ SSL Inspection  │
                    │       │         │
                    │      WAF        │
                    │       │         │
                    │      IPS        │
                    └───────┬─────────┘
                            │
                            ▼
                    Web Application
                      192.168.20.200
```

### Architecture Checklist

* [ ] Internet-facing interface identified
* [ ] Public address identified
* [ ] VIP configured
* [ ] Backend server reachable
* [ ] Firewall policy created
* [ ] Proxy inspection selected
* [ ] WAF profile attached
* [ ] SSL inspection evaluated for HTTPS
* [ ] IPS evaluated
* [ ] Logging enabled
* [ ] Backend return path verified

---

# 4. VIP / DNAT Checklist

The VIP provides the publishing mechanism for the internal application.

### VIP Checklist

* [ ] External IP is correct
* [ ] Mapped IP is correct
* [ ] Correct interface is selected
* [ ] Correct port mapping is configured
* [ ] HTTP mapping verified if required
* [ ] HTTPS mapping verified if required
* [ ] VIP is referenced by the correct firewall policy
* [ ] No conflicting VIP exists
* [ ] VIP matching behavior verified
* [ ] Backend server is listening
* [ ] Return route is correct

Example:

```text
Public IP
192.168.101.101
       │
       │ VIP / DNAT
       ▼
Private Web Server
192.168.20.200
```

> The IP addresses above are documentation/lab examples.

### VIP Security Checklist

* [ ] Do not expose unnecessary ports
* [ ] Do not publish unnecessary backend services
* [ ] Use dedicated VIPs where practical
* [ ] Avoid overlapping/conflicting VIP objects
* [ ] Restrict access at the firewall-policy layer

---

# 5. Firewall Policy Checklist

The firewall policy is the access-control layer.

```text
Client
   │
   ▼
VIP
   │
   ▼
Firewall Policy
   │
   ├── Proxy Inspection
   ├── WAF
   ├── SSL Inspection
   ├── IPS
   └── Logging
   │
   ▼
Web Server
```

### Source

* [ ] Source is restricted where business requirements permit
* [ ] Known partner networks identified
* [ ] Trusted application consumers identified
* [ ] Internet exposure is intentional
* [ ] `ALL` source is avoided when unnecessary

### Destination

* [ ] Destination is the specific Web VIP
* [ ] No unrestricted internal destination
* [ ] VIP is correctly matched

### Services

* [ ] HTTP allowed only if required
* [ ] HTTPS allowed only if required
* [ ] Application-specific services identified
* [ ] `ALL` services avoided unless explicitly required

### Inspection

* [ ] Proxy inspection selected
* [ ] WAF profile attached
* [ ] IPS profile evaluated
* [ ] SSL inspection evaluated
* [ ] Logging enabled

### Policy Security

* [ ] Policy order verified
* [ ] More-specific policies placed appropriately
* [ ] Shadowing policies checked
* [ ] Implicit deny understood
* [ ] Unused policies reviewed

---

# 6. Proxy Inspection Checklist

WAF relies on application-aware processing.

```text
HTTP / HTTPS
     │
     ▼
Proxy Processing
     │
     ├── HTTP Parsing
     ├── Application Inspection
     └── WAF
```

Checklist:

* [ ] Firewall policy uses supported inspection mode
* [ ] Proxy-based inspection verified
* [ ] WAF profile compatible with inspection mode
* [ ] Application traffic successfully reaches proxy
* [ ] HTTP parsing verified
* [ ] HTTPS handling verified
* [ ] Resource utilization monitored

### Proxy Mental Model

```text
Flow
 │
 └── IPS-oriented processing

Proxy
 │
 └── Proxy-based application processing
```

> **Important:** Exact feature support depends on FortiOS version and configuration.

---

# 7. HTTPS / SSL Inspection Checklist

HTTPS encrypts the application payload.

```text
Client
   │
   │ HTTPS
   ▼
FortiGate
   │
   ▼
SSL Inspection
   │
   ▼
Application Visibility
   │
   ▼
WAF
   │
   ▼
Web Server
```

### HTTPS Checklist

* [ ] HTTPS certificate requirements identified
* [ ] SSL inspection requirement evaluated
* [ ] Appropriate SSL inspection profile selected
* [ ] Certificate trust requirements understood
* [ ] Client compatibility tested
* [ ] Backend TLS requirements tested
* [ ] Application functionality tested
* [ ] CPU impact monitored
* [ ] Memory impact monitored
* [ ] Latency impact evaluated

### Security Trade-Off

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
Potential Resource Impact
```

> Do not enable deep inspection blindly in production. Validate certificate behavior, application compatibility, latency, and resource consumption.

---

# 8. WAF Profile Checklist

Create a dedicated WAF profile for the application.

### Profile

* [ ] WAF profile created
* [ ] Profile name is descriptive
* [ ] Profile is attached to the correct firewall policy
* [ ] Appropriate protection categories enabled
* [ ] Logging configured
* [ ] Application-specific exceptions identified
* [ ] False-positive strategy defined

### Policy Model

```text
Firewall Policy
      │
      ├── Proxy Inspection
      │
      ├── WAF Profile
      │
      ├── SSL Inspection
      │
      └── IPS
```

---

# 9. WAF Security Controls

Evaluate the following controls according to the application and FortiOS release:

* [ ] HTTP protocol validation
* [ ] Malformed request detection
* [ ] SQL injection protection
* [ ] HTTP attack protection
* [ ] Parameter inspection
* [ ] Header inspection
* [ ] Request-body inspection
* [ ] URL inspection
* [ ] Request-size controls
* [ ] Rate/abuse controls where supported
* [ ] Brute-force protections where applicable
* [ ] Logging
* [ ] Alerting
* [ ] Exception handling

### WAF Processing Model

```text
HTTP Request
      │
      ▼
HTTP Parsing
      │
      ▼
WAF Inspection
      │
 ┌────┴────┐
 ▼         ▼
Allow     Block
 │         │
 ▼         X
Web       Log
Server
```

---

# 10. WAF vs IPS

Do not treat WAF and IPS as identical security controls.

| Control                  | Primary Question                               |
| ------------------------ | ---------------------------------------------- |
| **Firewall**             | Who can connect?                               |
| **VIP**                  | Where should traffic be translated/published?  |
| **SSL Inspection**       | Can encrypted traffic be inspected?            |
| **IPS**                  | Does traffic match malicious/exploit patterns? |
| **WAF**                  | Is this HTTP/HTTPS request malicious?          |
| **Application Security** | Is the application securely designed?          |

### Mental Model

```text
Firewall
   ↓
Can the connection pass?

SSL Inspection
   ↓
Can encrypted content be inspected?

WAF
   ↓
Is the web request malicious?

IPS
   ↓
Does traffic match supported malicious patterns?

Application
   ↓
Is the application itself secure?
```

### Checklist

* [ ] Firewall used for access control
* [ ] VIP used for publishing
* [ ] SSL inspection used where appropriate
* [ ] WAF used for web-layer protection
* [ ] IPS evaluated as complementary protection
* [ ] Secure coding remains an application responsibility

---

# 11. SQL Injection Protection

SQL Injection is a major WAF use case.

Example:

```http
GET /product?id=100
```

Potentially malicious:

```http
GET /product?id=100' OR '1'='1
```

### SQLi Checklist

* [ ] SQL injection protection enabled where supported
* [ ] Request parameters inspected
* [ ] WAF events logged
* [ ] False positives monitored
* [ ] Application-specific exceptions documented
* [ ] Exceptions scoped as narrowly as possible
* [ ] Application-side parameterized queries implemented
* [ ] Database permissions minimized

### Protection Flow

```text
HTTP Request
      │
      ▼
     WAF
      │
      ├── Parse
      ├── Inspect
      ├── Match
      └── Action
           │
      ┌────┴────┐
      ▼         ▼
    Allow     Block
```

> **Important:** WAF reduces risk but does not replace secure application development.

---

# 12. Brute-Force Protection

Web applications frequently expose authentication endpoints.

Example:

```text
Attacker
   │
   ├── Login Attempt
   ├── Login Attempt
   ├── Login Attempt
   ├── Login Attempt
   └── ...
        │
        ▼
       WAF
        │
        ▼
 Detection / Policy
```

### Checklist

* [ ] Login endpoint identified
* [ ] Repeated authentication attempts monitored
* [ ] Application rate limiting evaluated
* [ ] WAF controls evaluated
* [ ] MFA enabled where appropriate
* [ ] Account lockout/rate limiting evaluated
* [ ] Authentication logs monitored
* [ ] Dedicated authentication protection considered

> WAF can contribute to application-layer abuse protection, but authentication controls should primarily be designed as part of the application/security architecture.

---

# 13. Security Fabric Integration

For Security Fabric deployments:

```text
                  Security Fabric
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
      FortiGate        Fabric       Security
        WAF           Components     Services
          │              │              │
          └──────────────┼──────────────┘
                         ▼
                  Central Visibility
```

Checklist:

* [ ] FortiGate integrated into Security Fabric where required
* [ ] Relevant Fabric components identified
* [ ] Security telemetry evaluated
* [ ] Event visibility verified
* [ ] Centralized monitoring evaluated
* [ ] Integration permissions reviewed
* [ ] FortiOS/product compatibility verified

> Available Fabric integrations vary by FortiOS version and deployed Fortinet products.

---

# 14. Logging & Monitoring

A WAF without visibility is difficult to operate securely.

### Logging Checklist

* [ ] WAF events logged
* [ ] Blocked requests logged
* [ ] Violations logged
* [ ] Security events correlated
* [ ] Firewall policy logs enabled where required
* [ ] SSL inspection events monitored where required
* [ ] IPS events monitored
* [ ] Application errors correlated with WAF events

### Investigate

```text
Timestamp
Source IP
Destination VIP
URL
HTTP Method
HTTP Headers
Parameters
Request Body
Matched Rule
Action
Policy
WAF Profile
```

### Operational Rule

```text
Detect
  ↓
Log
  ↓
Investigate
  ↓
Tune
  ↓
Verify
```

---

# 15. Production Hardening

## Exposure

* [ ] Public exposure is business-required
* [ ] Source restriction implemented where possible
* [ ] Only required services published
* [ ] Unused VIPs removed
* [ ] Unused policies removed

## Firewall

* [ ] Least privilege applied
* [ ] Specific VIP used
* [ ] Specific services used
* [ ] Policy order verified
* [ ] Logging enabled

## WAF

* [ ] WAF enabled
* [ ] Appropriate protections enabled
* [ ] False positives monitored
* [ ] Exceptions minimized
* [ ] Exceptions documented

## SSL

* [ ] HTTPS inspection requirement evaluated
* [ ] Certificate deployment tested
* [ ] Application compatibility tested
* [ ] Resource impact monitored

## IPS

* [ ] IPS profile evaluated
* [ ] Appropriate signatures/policies configured
* [ ] Events monitored

## Application

* [ ] Secure coding practices implemented
* [ ] Patching maintained
* [ ] Authentication hardened
* [ ] MFA evaluated
* [ ] Database security implemented
* [ ] Application logging enabled

---

# 16. WAF Troubleshooting

## Web Application Is Not Reachable

Follow the traffic chain:

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
Proxy
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

### Checklist

* [ ] Client can reach public IP
* [ ] VIP exists
* [ ] External IP is correct
* [ ] Mapped IP is correct
* [ ] VIP matches traffic
* [ ] Firewall policy matches
* [ ] Service is allowed
* [ ] Proxy inspection is correct
* [ ] WAF profile is attached
* [ ] SSL inspection is correct
* [ ] IPS is not blocking
* [ ] Routing is correct
* [ ] Return traffic is correct
* [ ] Web server is listening

---

# 17. False Positive Troubleshooting

If legitimate traffic is blocked:

```text
Client
   │
   ▼
 WAF
   │
   X
Blocked
```

### Investigate

* [ ] WAF event identified
* [ ] Matched rule identified
* [ ] Source identified
* [ ] Destination identified
* [ ] URL identified
* [ ] HTTP method identified
* [ ] Header inspected
* [ ] Parameter inspected
* [ ] Request body inspected
* [ ] Application behavior checked

### Remediation

```text
Identify Rule
     ↓
Understand Why It Matched
     ↓
Validate Application Behavior
     ↓
Tune / Exception
     ↓
Retest
     ↓
Monitor
```

### Avoid

```text
False Positive
      ↓
Disable WAF
```

### Prefer

```text
False Positive
      ↓
Identify Specific Rule
      ↓
Create Narrow Exception
      ↓
Retest
```

---

# 18. HTTPS Troubleshooting

If HTTP works but HTTPS fails:

### Checklist

* [ ] HTTPS VIP exists
* [ ] TCP/443 is allowed
* [ ] Correct certificate configured
* [ ] Client trusts certificate where required
* [ ] SSL inspection policy is correct
* [ ] TLS version compatibility checked
* [ ] Backend TLS checked
* [ ] WAF receives decrypted application traffic where required
* [ ] SSL inspection logs reviewed
* [ ] WAF logs reviewed

### Troubleshooting Flow

```text
HTTPS Failure
     │
     ▼
TCP/443
     │
     ▼
Certificate
     │
     ▼
TLS Handshake
     │
     ▼
SSL Inspection
     │
     ▼
HTTP Visibility
     │
     ▼
WAF
```

---

# 19. VIP Troubleshooting

When traffic never reaches the backend:

```text
Internet
   │
   ▼
Public IP
   │
   ▼
VIP
   │
   ▼
DNAT
   │
   ▼
Backend Server
```

### Checklist

* [ ] Public IP correct
* [ ] VIP enabled
* [ ] VIP interface correct
* [ ] External port correct
* [ ] Mapped port correct
* [ ] Mapped IP correct
* [ ] Firewall policy references VIP
* [ ] Policy order correct
* [ ] Backend listening
* [ ] Backend route correct
* [ ] Return traffic correct

---

# 20. WAF Verification Checklist

After deployment, perform controlled tests.

### Connectivity

* [ ] HTTP test completed
* [ ] HTTPS test completed
* [ ] Correct VIP reached
* [ ] Backend response verified

### WAF

* [ ] WAF profile confirmed
* [ ] Legitimate request allowed
* [ ] Controlled malicious test generated in an authorized environment
* [ ] WAF detection verified
* [ ] Block/log action verified
* [ ] Event appears in logs

### SSL

* [ ] TLS handshake verified
* [ ] Certificate behavior verified
* [ ] Application functionality verified

### Performance

* [ ] CPU monitored
* [ ] Memory monitored
* [ ] Latency monitored
* [ ] Application response time verified

### Operations

* [ ] Logging verified
* [ ] Monitoring verified
* [ ] Alerting verified
* [ ] Exception documentation completed

---

# 21. Common WAF Design Mistakes

## ❌ Mistake 1 — Publishing a Web Server Without WAF

```text
Internet
   │
   ▼
VIP
   │
   ▼
Web Server
```

Prefer:

```text
Internet
   │
   ▼
VIP
   │
   ▼
Firewall Policy
   │
   ▼
Proxy
   │
   ▼
WAF
   │
   ▼
Web Server
```

---

## ❌ Mistake 2 — `ALL` Services

```text
Source      : ALL
Destination : Web VIP
Service     : ALL
```

Prefer:

```text
Source      : Required
Destination : Web VIP
Service     : HTTP / HTTPS
```

---

## ❌ Mistake 3 — Ignoring HTTPS Encryption

```text
HTTPS
  │
  ▼
Encrypted Application Data
  │
  ▼
Limited Visibility
```

For deployments requiring application inspection:

```text
HTTPS
  │
  ▼
SSL Inspection
  │
  ▼
WAF
```

---

## ❌ Mistake 4 — Disabling WAF Because of One False Positive

```text
False Positive
      ↓
Disable WAF
```

Prefer:

```text
False Positive
      ↓
Identify Rule
      ↓
Tune Narrowly
      ↓
Retest
```

---

## ❌ Mistake 5 — Treating WAF as Complete Security

WAF does **not** replace:

* [ ] Secure coding
* [ ] Patch management
* [ ] Authentication security
* [ ] MFA
* [ ] IPS
* [ ] Firewall policy
* [ ] Endpoint security
* [ ] Database security
* [ ] Logging
* [ ] Monitoring

---

# 22. Production Reference Architecture

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
                     │ Firewall   │
                     │   Policy   │
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
                  │ Web Application │
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

# 23. Fast NSE Exam Checklist

| Topic               | Remember                                                    |
| ------------------- | ----------------------------------------------------------- |
| **WAF**             | Application-layer web protection                            |
| **VIP**             | Publishes/maps external address to backend                  |
| **DNAT**            | Translates destination address/port                         |
| **Proxy**           | Proxy-based application processing                          |
| **HTTPS**           | Encrypted application traffic                               |
| **SSL Inspection**  | Provides visibility into encrypted traffic where applicable |
| **SQLi**            | Major WAF use case                                          |
| **IPS**             | Complementary security control                              |
| **Firewall**        | Access control                                              |
| **WAF**             | HTTP/HTTPS request inspection                               |
| **False Positive**  | Investigate and tune instead of disabling WAF               |
| **Security Fabric** | Security integration and visibility                         |
| **Production**      | Least privilege + defense in depth                          |

---

# 24. One-Minute Revision

```text
PUBLIC WEB APPLICATION
        │
        ▼
      PUBLIC IP
        │
        ▼
       VIP
        │
        ▼
 FIREWALL POLICY
        │
        ▼
      PROXY
        │
        ▼
 SSL INSPECTION
        │
        ▼
       WAF
        │
        ▼
       IPS
        │
        ▼
   WEB SERVER
```

Remember:

```text
Firewall
   ↓
WHO can connect?

VIP
   ↓
WHERE does traffic go?

SSL Inspection
   ↓
CAN encrypted traffic be inspected?

WAF
   ↓
WHAT is the web request trying to do?

IPS
   ↓
DOES traffic match malicious patterns?

Application
   ↓
IS the application securely designed?
```

---

# 25. Final WAF Mental Model

```text
                         FORTIGATE WAF
                                │
                                ▼
                           PUBLIC IP
                                │
                                ▼
                               VIP
                                │
                                ▼
                        FIREWALL POLICY
                                │
                                ▼
                         PROXY INSPECTION
                                │
                                ▼
                         SSL INSPECTION
                                │
                                ▼
                               WAF
                                │
                    ┌───────────┴───────────┐
                    ▼                       ▼
                  ALLOW                    BLOCK
                    │                       │
                    ▼                       X
              WEB APPLICATION             LOG
                    │
                    ▼
                   IPS
                    │
                    ▼
             LOGGING / MONITORING
```

## 🧠 Golden Troubleshooting Rule

When a web application fails, **do not immediately blame WAF**.

Follow the chain:

```text
Client
  ↓
Public IP
  ↓
VIP
  ↓
Firewall Policy
  ↓
Proxy
  ↓
SSL Inspection
  ↓
WAF
  ↓
IPS
  ↓
Web Server
  ↓
Return Traffic
```

Find the **first layer where the expected behavior stops**.

---

# 🎯 SheynShield Engineering Rule

> **WAF is not another firewall rule.**

The firewall answers:

```text
"WHO can connect?"
```

The WAF answers:

```text
"WHAT is this HTTP/HTTPS request trying to do?"
```

For production troubleshooting, remember:

```text
CLIENT
  ↓
VIP
  ↓
FIREWALL
  ↓
PROXY
  ↓
SSL INSPECTION
  ↓
WAF
  ↓
IPS
  ↓
WEB SERVER
```

**A successful TCP connection does not prove that the web application is secure.**

**A successful HTTP response does not prove that WAF is correctly protecting the application.**

Always verify:

```text
Connectivity
     ↓
Inspection
     ↓
Detection
     ↓
Blocking
     ↓
Logging
     ↓
Application Functionality
```

---

## 🔗 Topics

* [ ] VIP / DNAT
* [ ] Firewall Policy
* [ ] Proxy Inspection
* [ ] SSL Inspection
* [ ] IPS
* [ ] Security Fabric
* [ ] Web Application Security
* [ ] HTTP/HTTPS Security
* [ ] Application-Layer Protection
* [ ] FortiGate Troubleshooting

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

## 🔎 Keywords

`FortiGate WAF` · `FortiGate Web Application Firewall` · `FortiOS WAF` · `FortiGate WAF configuration` · `FortiGate WAF checklist` · `FortiGate VIP WAF` · `FortiGate reverse proxy` · `FortiGate SQL injection protection` · `FortiGate HTTP security` · `FortiGate HTTPS inspection` · `FortiGate proxy inspection` · `FortiGate IPS WAF` · `FortiGate NSE4 WAF` · `FortiGate NSE7 WAF` · `FortiGate troubleshooting` · `Web Application Firewall checklist`

---

> **SheynShield | Engineering Secure Networks**
>
> **Build it. Secure it. Verify it. Troubleshoot it.**
>
> This checklist is designed as a practical reference for FortiGate WAF deployment, security hardening, NSE preparation, and production troubleshooting.

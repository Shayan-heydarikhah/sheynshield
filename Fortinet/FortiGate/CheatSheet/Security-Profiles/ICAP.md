# FortiGate ICAP — Security & Content Inspection  

> **FortiOS Focus:** ICAP, Content Adaptation, DLP, Antivirus, HTTP/HTTPS Inspection, Proxy-Based Inspection, Request/Response Modification
> **Audience:** FortiGate NSE4 / NSE7, Network Security Engineers, Security Architects
> **Protocol:** Internet Content Adaptation Protocol (ICAP)
> **Default ICAP Port:** TCP/1344
> **Secure ICAP:** TCP/11344
> **Key FortiGate Component:** WAD / Proxy Engine
> **Primary Use Cases:** Content Filtering, Antivirus, DLP, File Inspection, URL Filtering, Content Adaptation

---

## 📌 Quick Reference

| Component            | Purpose                                                                          |
| -------------------- | -------------------------------------------------------------------------------- |
| **ICAP**             | Offloads HTTP/HTTPS content adaptation and inspection to an external ICAP server |
| **REQMOD**           | ICAP request modification/inspection                                             |
| **RESPMOD**          | ICAP response modification/inspection                                            |
| **ICAP Server**      | External server that analyzes or modifies content                                |
| **FortiGate**        | Intercepts proxy traffic and sends selected content to ICAP                      |
| **WAD**              | FortiGate web proxy daemon involved in proxy processing                          |
| **C-ICAP**           | Open-source ICAP server implementation                                           |
| **DLP**              | Detects sensitive information in inspected content                               |
| **Antivirus**        | Scans content for malware                                                        |
| **Streaming Bypass** | Allows selected streaming content to bypass ICAP processing                      |
| **204 Response**     | Indicates that no modification is required                                       |
| **Health Check**     | Verifies ICAP server availability                                                |
| **REQMOD**           | Inspect/modify HTTP requests                                                     |
| **RESPMOD**          | Inspect/modify HTTP responses                                                    |

---

# 1. What Is ICAP?

**ICAP — Internet Content Adaptation Protocol** is a protocol designed to allow a proxy device to send HTTP content to an external content-adaptation server.

Instead of performing every content inspection task locally, FortiGate can forward selected content to an external ICAP server.

```text
Client
  │
  ▼
FortiGate
  │
  │ HTTP/HTTPS Proxy
  ▼
Content Extraction / Decryption
  │
  ▼
ICAP Server
  │
  ├── Antivirus
  ├── DLP
  ├── Content Filtering
  ├── Malware Inspection
  └── Content Modification
  │
  ▼
FortiGate
  │
  ├── Additional Security Profiles
  └── Firewall Processing
  │
  ▼
Client
```

### Core Concept

> **FortiGate acts as the ICAP client, while the external ICAP server performs content adaptation/inspection.**

---

# 2. Why Use ICAP?

ICAP is useful when an organization wants to move specific content inspection workloads to an external security engine.

Typical use cases:

```text
HTTP/HTTPS
     │
     ▼
FortiGate
     │
     ▼
ICAP
     │
     ├── Antivirus
     ├── DLP
     ├── Content Filtering
     ├── Malware Detection
     └── Custom Content Analysis
```

### Advantages

* Externalize content inspection
* Integrate third-party inspection engines
* Centralize specialized content scanning
* Extend FortiGate proxy functionality
* Apply DLP/content filtering outside FortiGate
* Inspect selected content instead of everything

### Trade-off

> **ICAP introduces additional processing, network communication, and infrastructure cost.**

---

# 3. ICAP Architecture

Typical deployment:

```text
                         INTERNET
                            │
                            ▼
                     ┌──────────────┐
                     │   FortiGate  │
                     │              │
                     │ Web Proxy    │
                     │ SSL Inspect  │
                     │ ICAP Client  │
                     └──────┬───────┘
                            │
                            │ TCP/1344
                            ▼
                     ┌──────────────┐
                     │  ICAP Server │
                     │              │
                     │ AV / DLP /   │
                     │ Content Scan │
                     └──────┬───────┘
                            │
                            ▼
                         FortiGate
                            │
                            ▼
                           LAN
                            │
                            ▼
                          Client
```

---

# 4. ICAP Processing Flow

The complete HTTP/HTTPS processing path can look like:

```text
Client
  │
  │ HTTP/HTTPS
  ▼
FortiGate
  │
  ├── Proxy Processing
  │
  ├── SSL Decryption
  │
  ├── Content Extraction
  │
  ▼
ICAP Server
  │
  ├── Inspect
  ├── Scan
  ├── Modify
  └── Allow / Block
  │
  ▼
FortiGate
  │
  ├── IPS
  ├── DLP
  ├── Web Filter
  ├── Application Control
  └── Other Security Profiles
  │
  ▼
Client
```

### Important

ICAP does **not necessarily mean that all traffic is sent to the ICAP server**.

The ICAP profile can be configured to select:

* Requests
* Responses
* HTTP methods
* Hosts
* Response status codes
* Content types
* File-transfer traffic
* Streaming content

---

# 5. ICAP Request vs Response

ICAP primarily provides two important processing models.

## REQMOD

**Request Modification**

```text
Client
  │
  ▼
FortiGate
  │
  ▼
ICAP REQMOD
  │
  ▼
Web Server
```

Used to inspect or modify client requests.

---

## RESPMOD

**Response Modification**

```text
Web Server
     │
     ▼
FortiGate
     │
     ▼
ICAP RESPMOD
     │
     ▼
Client
```

Used to inspect or modify server responses.

### Mental Model

```text
REQMOD  → Request
RESPMOD → Response
```

---

# 6. Enable ICAP in FortiGate

If ICAP options are not visible:

```text
System
  └── Feature Visibility
        └── ICAP
```

Enable **ICAP**.

---

# 7. ICAP and Proxy Mode

ICAP is designed to work with FortiGate's proxy-based traffic processing.

Typical policy:

```text
LAN
 │
 ▼
Firewall Policy
 │
 ├── Proxy Inspection
 ├── Deep SSL Inspection
 └── ICAP Profile
 │
 ▼
Internet
```

### Important

> **ICAP inspection is associated with proxy-based processing because FortiGate needs to process and extract the HTTP content before sending selected content to the ICAP server.**

---

# 8. ICAP Server Configuration

Example:

```cli
config icap server
    edit icap-test
        set ip-version 4
        set ip-address 192.168.20.200
        set port 1344
        set max-connections 200
        set secure enable
        set ssl-cert test.cert
        set healthcheck enable
        set healthcheck-server 192.168.20.200
    next
end
```

### Key Parameters

| Parameter            | Meaning                                           |
| -------------------- | ------------------------------------------------- |
| `ip-address`         | ICAP server IP                                    |
| `port`               | ICAP TCP port                                     |
| `max-connections`    | Maximum configured connections to the ICAP server |
| `secure`             | Enables secure ICAP communication                 |
| `ssl-cert`           | Certificate used for secure ICAP communication    |
| `healthcheck`        | Enables ICAP server health checking               |
| `healthcheck-server` | Server address used for health verification       |

---

# 9. ICAP Ports

Typical deployment:

```text
Standard ICAP
TCP/1344
```

Secure ICAP:

```text
TCP/11344
```

Conceptually:

```text
FortiGate
    │
    ├── TCP/1344   → ICAP
    │
    └── TCP/11344  → Secure ICAP
```

> Always verify the configured port on both FortiGate and the ICAP server.

---

# 10. ICAP Profile

An ICAP profile determines **what traffic is forwarded to the ICAP server and how FortiGate handles ICAP results/failures**.

Example:

```cli
config icap profile
    edit icap-prof-test
        set request enable
        set response enable
        set streaming-content-bypass enable

        set request-server icap-test
        set response-server icap-test

        set request-failure error
        set response-failure error

        set request-path /content-filter/
        set response-path /content-filter/

        set methods delete get head options post put trace other

        set icap-block-log enable
        set 204-response enable
    next
end
```

---

# 11. REQMOD Configuration

Enable request processing:

```cli
config icap profile
    edit icap-prof-test
        set request enable
        set request-server icap-test
        set request-path /content-filter/
        set methods delete get head options post put trace other
    next
end
```

Conceptually:

```text
Client Request
      │
      ▼
FortiGate
      │
      ▼
REQMOD
      │
      ▼
ICAP Server
      │
      ▼
Allow / Modify / Block
```

---

# 12. RESPMOD Configuration

Response inspection can be configured independently.

Example:

```cli
config icap profile
    edit icap-prof-test
        set request disable
        set response enable
        set response-server icap-test
        set response-path ''
        set response-failure error
        set response-req-hdr disable
        set respmod-default-action bypass
    next
end
```

Conceptually:

```text
Web Server
    │
    ▼
HTTP Response
    │
    ▼
FortiGate
    │
    ▼
RESPMOD
    │
    ▼
ICAP Server
    │
    ├── Allow
    ├── Modify
    └── Block
    │
    ▼
Client
```

---

# 13. ICAP Failure Behavior

A critical design decision is:

> **What should FortiGate do when the ICAP server is unavailable?**

Possible behavior depends on the configured failure action.

Conceptually:

```text
FortiGate
    │
    ▼
ICAP Server
    │
    ├── Available → Inspect
    │
    └── Unavailable
            │
            ├── Error
            │
            └── Bypass
```

### Fail-Closed

```text
ICAP unavailable
      ↓
Block / Error
      ↓
Higher security
      ↓
Potential availability impact
```

### Fail-Open

```text
ICAP unavailable
      ↓
Bypass ICAP
      ↓
Higher availability
      ↓
Potential security gap
```

> Choose fail-open vs fail-closed according to the sensitivity of the traffic and business requirements.

---

# 14. Streaming Content Bypass

Streaming content can create unnecessary ICAP processing overhead.

Example:

```cli
config icap profile
    edit icap-prof-test
        set streaming-content-bypass enable
    next
end
```

Conceptually:

```text
Streaming Content
      │
      ├── Bypass ICAP
      │
      └── Continue to Client
```

This can be useful for traffic such as:

```text
Video
Audio
Large streaming objects
```

### Security vs Performance

```text
More ICAP inspection
        ↓
More security visibility
        ↓
More resource consumption

More bypass
        ↓
Better performance
        ↓
Less inspection
```

---

# 15. Selective ICAP Inspection

One of the most important production design principles:

> **Do not automatically send the entire web to ICAP if only specific content requires inspection.**

For example:

```text
varzesh3.ir
      │
      ├── HTML       → Bypass
      ├── CSS        → Bypass
      ├── JavaScript → Bypass
      └── MP4        → ICAP
```

This is much more efficient than:

```text
Entire Website
      │
      ▼
ICAP
```

---

# 16. Response Forwarding Rules

Selective response inspection can be configured using response forwarding rules.

Example:

```cli
config icap profile
    edit icap-prof-test

        set request disable
        set response enable
        set streaming-content-bypass disable
        set preview disable

        set response-server icap-test
        set response-failure error
        set response-path ''
        set response-req-hdr disable
        set respmod-default-action bypass

        config respmod-forward-rules
            edit 1
                set host all
                set action forward
                set http-resp-status-code 200 301 302

                config header-group
                    edit 1
                        set header-name content-type
                        set header image/jpeg
                    next
                end
            next
        end

    next
end
```

---

# 17. `respmod-default-action`

This setting is important when using selective forwarding.

Conceptually:

```text
Default Action
       │
       └── bypass
```

means:

```text
Do not send everything to ICAP
             │
             ▼
Evaluate forwarding rules
             │
             ▼
Matching traffic → ICAP
Non-matching     → Bypass
```

This allows precise control over what content is inspected.

---

# 18. Response Status Code Filtering

ICAP forwarding rules can use HTTP response codes.

Example:

```text
200
301
302
```

Conceptually:

```text
HTTP Response
      │
      ├── 200 → ICAP
      ├── 301 → ICAP
      ├── 302 → ICAP
      └── Other → Default Action
```

This can be useful when the security requirement is focused on specific classes of web responses.

---

# 19. Content-Type Filtering

ICAP forwarding can also be based on the HTTP `Content-Type` header.

Example:

```text
Content-Type: image/jpeg
```

Configuration concept:

```cli
config header-group
    edit 1
        set header-name content-type
        set header image/jpeg
    next
end
```

Example policy:

```text
Content-Type
     │
     ├── image/jpeg → ICAP
     ├── video/mp4  → Bypass
     ├── text/html  → Bypass
     └── application/pdf → ICAP
```

---

# 20. REQMOD Methods

The ICAP profile can specify HTTP methods for request processing.

Example:

```text
DELETE
GET
HEAD
OPTIONS
POST
PUT
TRACE
OTHER
```

CLI:

```cli
set methods delete get head options post put trace other
```

### Security Consideration

Not every HTTP method needs to be exposed or inspected.

Restrict methods according to the application requirements.

---

# 21. ICAP `204` Response

The ICAP server can return:

```text
204 No Modification Needed
```

Meaning:

```text
ICAP inspected content
        │
        ▼
No modification required
        │
        ▼
FortiGate continues processing
```

This can avoid unnecessary content rewriting when the ICAP server determines that no modification is needed.

---

# 22. ICAP HTTP Status Codes

ICAP uses its own response status classes.

## 1xx — Informational

Example:

```text
100 Continue
```

Indicates that processing can continue after preview handling.

---

## 2xx — Success

### 204 — No Modification Needed

```text
204
↓
Content does not require modification
```

### 206 — Partial Content

Used when only part of the content is being handled/filtered.

---

## 4xx — Client Errors

| Code    | Meaning                |
| ------- | ---------------------- |
| **400** | Bad Request            |
| **404** | ICAP Service Not Found |
| **405** | Method Not Allowed     |
| **408** | Request Timeout        |
| **418** | Bad Composition        |

### 405

The requested ICAP method is not supported by the selected service.

Example:

```text
RESPMOD requested
      ↓
Service supports REQMOD only
      ↓
405
```

### 408

The ICAP server timed out while waiting for a request from the ICAP client.

---

## 5xx — Server Errors

| Code    | Meaning                    |
| ------- | -------------------------- |
| **500** | Server Error               |
| **501** | Method Not Implemented     |
| **502** | Bad Gateway                |
| **503** | Service Overloaded         |
| **505** | ICAP Version Not Supported |

### 503

The ICAP server/service has exceeded its supported connection capacity.

```text
Too many connections
        ↓
ICAP overloaded
        ↓
503
```

---

# 23. ICAP Server Capacity

ICAP introduces a second processing system:

```text
FortiGate
    │
    ├── CPU
    ├── Memory
    └── WAD
         │
         ▼
      ICAP Server
         │
         ├── CPU
         ├── Memory
         └── Connection Pool
```

### Important

> The practical maximum connection count is determined by the FortiGate configuration, ICAP implementation, available resources, and deployment architecture. Do not assume a universal fixed limit.

Always size:

```text
FortiGate
+
ICAP Server
+
Network
```

together.

---

# 24. WAD and ICAP

FortiGate web proxy processing is associated with the **WAD** process.

When ICAP is introduced:

```text
Client
  │
  ▼
WAD
  │
  ▼
ICAP
  │
  ▼
WAD
  │
  ▼
Client
```

Additional ICAP processing can affect:

* Proxy connection handling
* WAD resources
* Connection pools
* CPU
* Memory
* Latency

### Production Principle

> **ICAP should be introduced with capacity planning and performance testing.**

---

# 25. ICAP + HTTPS

For HTTPS inspection:

```text
Client
  │
  │ TLS
  ▼
FortiGate
  │
  ├── SSL Inspection
  │
  ▼
HTTP Content
  │
  ▼
ICAP
```

The important concept is:

```text
Encrypted HTTPS
      ↓
FortiGate decrypts
      ↓
HTTP content becomes inspectable
      ↓
Selected content → ICAP
```

Therefore, ICAP content inspection of HTTPS traffic depends on the FortiGate SSL inspection/decryption architecture.

---

# 26. ICAP + Security Profiles

ICAP does not necessarily replace FortiGate security inspection.

A possible chain is:

```text
Client
  │
  ▼
FortiGate
  │
  ├── SSL Inspection
  │
  ├── ICAP
  │
  ├── IPS
  │
  ├── Web Filter
  │
  ├── Application Control
  │
  └── Other Profiles
  │
  ▼
Internet
```

### Key Concept

> **ICAP can be an additional inspection layer, not necessarily the only inspection layer.**

---

# 27. Firewall Policy Design

Typical LAN-to-WAN design:

```text
LAN
 │
 ▼
Firewall Policy
 │
 ├── Proxy Inspection
 ├── Deep/Custom Deep Inspection
 ├── ICAP Profile
 ├── IPS
 ├── Web Filter
 └── Application Control
 │
 ▼
WAN
```

---

# 28. DMZ-to-WAN Design

A DMZ policy may use a simpler inspection architecture depending on the application.

Example concept:

```text
DMZ
 │
 ▼
Firewall Policy
 │
 ├── NAT
 ├── Internet Access
 ├── IPS
 └── DoS / Other Security Controls
 │
 ▼
WAN
```

ICAP should only be attached where the traffic path and inspection requirements justify proxy/content adaptation.

---

# 29. File Transfer Inspection

ICAP can also be used for selected file-transfer inspection scenarios.

Example SSH file transfer:

```cli
config icap profile
    edit icap-prof-test
        set file-transfer ssh
        set file-transfer-server icap-test
        set file-transfer-path ssh_test
    next
end
```

For SSH-based inspection, the relevant SSH deep-inspection capability must be enabled/configured in the applicable inspection profile.

---

# 30. FTP File Transfer Inspection

Example:

```cli
config icap profile
    edit icap-prof-test
        set file-transfer ftp
        set streaming-content-bypass enable
        set file-transfer-server icap-test
        set file-transfer-path ftp_test
    next
end
```

Conceptually:

```text
FTP File
   │
   ▼
FortiGate
   │
   ▼
ICAP
   │
   ├── Malware Scan
   ├── DLP
   └── Content Inspection
   │
   ▼
Destination
```

---

# 31. C-ICAP

**C-ICAP** is an open-source implementation of an ICAP server.

A simplified lab architecture:

```text
FortiGate
    │
    │ TCP/1344
    ▼
Ubuntu / Rocky Linux
    │
    ▼
C-ICAP
    │
    ├── URL Check
    ├── Content Filtering
    └── Antivirus modules
```

---

# 32. C-ICAP — Ubuntu Lab

Example package installation:

```bash
sudo -i

apt update
apt upgrade

apt install git -y
apt install c-icap libc-icap-modules -y
```

Optional module source:

```bash
git clone https://github.com/c-icap/c-icap-modules
```

Additional modules may include:

```bash
apt install libc-icap-mod-contentfiltering
apt install libc-icap-mod-urlcheck
apt install libc-icap-mod-virus-scan
```

> Package names can vary by Ubuntu/Debian release. Verify package availability before using them in automation.

---

# 33. C-ICAP — CentOS / RHEL-Based Systems

Example:

```bash
sudo -i

dnf update
dnf upgrade

dnf install epel-release -y
dnf install c-icap -y
dnf install c-icap-libs -y
dnf install c-icap-modules -y
```

Depending on the distribution/repository, additional packages may be available:

```bash
dnf install c-icap-devel.x86_64 -y
dnf install c-icap-libs.x86_64 -y
dnf install c-icap-modules.x86_64 -y
```

Example configuration locations:

```text
/etc/c-icap/
```

Common files:

```text
/etc/c-icap/c-icap.conf
/etc/c-icap/urlfilter.conf
/etc/c-icap/srv_url_check.conf
```

---

# 34. Verify C-ICAP Modules

Ubuntu-style library location may look like:

```bash
cd /usr/lib/x86_64-linux-gnu/c_icap
```

RHEL/Rocky-style systems may use:

```bash
cd /usr/lib64/c_icap
```

Verify that the expected modules are installed.

---

# 35. Enable C-ICAP Service

Example:

```bash
systemctl start c-icap
systemctl enable c-icap

systemctl status c-icap
```

Check the listening port:

```bash
netstat -tulnp | grep 1344
```

Expected concept:

```text
:::1344
LISTEN
c-icap
```

---

# 36. C-ICAP Basic Configuration

Example:

```bash
nano /etc/c-icap/c-icap.conf
```

Configure:

```text
ServerName 0.0.0.0
Port 1344
```

Restart:

```bash
systemctl restart c-icap
```

Verify:

```bash
systemctl status c-icap
```

---

# 37. C-ICAP URL Filtering Example

Example URL-check configuration:

```bash
nano /etc/c-icap/srv_url_check.conf
```

Example rules:

```text
deny ^https?://([a-z0-9]+\.)*example\.ir(/.*)?$

deny ^https?://([a-z0-9]+\.)*example\.ir:[0-9]+(/.*)?$

allow .*
```

> Replace example domains with your own test domain/policy. Regex syntax should be validated against the exact C-ICAP module/version in use.

---

# 38. Test ICAP Connectivity from FortiGate

From FortiGate:

```cli
execute telnet 192.168.20.200 1344
```

Expected:

```text
Connection successful
```

If it fails, investigate:

```text
[ ] Routing
[ ] Firewall policy
[ ] ICAP server listening
[ ] TCP/1344 allowed
[ ] Server IP
[ ] VDOM routing
[ ] Local-in / network restrictions
[ ] ICAP service status
```

---

# 39. ICAP Health Check

Health checking helps FortiGate determine whether the ICAP server is available.

Conceptually:

```text
FortiGate
    │
    │ Health Check
    ▼
ICAP Server
    │
    ├── UP
    │
    └── DOWN
```

Configuration:

```cli
config icap server
    edit icap-test
        set healthcheck enable
        set healthcheck-server 192.168.20.200
    next
end
```

---

# 40. ICAP Content Selection

A mature deployment should answer:

> **What exactly needs to go to ICAP?**

Possible selectors:

```text
Destination Host
        +
HTTP Method
        +
HTTP Status Code
        +
Content-Type
        +
Request / Response
        +
File Transfer
        +
Streaming Content
```

Example:

```text
                 Web Traffic
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
       Normal                   High Risk
       Content                  Content
          │                       │
       Bypass                   ICAP
                                  │
                     ┌────────────┼────────────┐
                     ▼            ▼            ▼
                    AV           DLP       Content Filter
```

---

# 41. Example: Inspect Only MP4

Requirement:

> Inspect only MP4 content from a specific website.

Conceptual design:

```text
HTTP Response
     │
     ▼
Host Match
     │
     ▼
Content-Type / File Type
     │
     ├── video/mp4 → ICAP
     │
     └── Other     → Bypass
```

This is generally more efficient than forwarding the complete website to ICAP.

---

# 42. Example: Inspect Redirects

If the requirement is to inspect HTTP redirects:

```text
HTTP Response
      │
      ├── 200 → normal content
      ├── 301 → ICAP
      └── 302 → ICAP
```

Response status filtering can be used to selectively forward matching responses.

---

# 43. ICAP Security Model

Think of ICAP as:

```text
                 FORTIGATE
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
      Security              Content
      Controls              Adaptation
          │                     │
          │                    ICAP
          │                     │
          ▼              ┌──────┼──────┐
        IPS              AV     DLP   Filter
        Web Filter
        App Control
```

---

# 44. ICAP vs FortiProxy

FortiProxy is a Fortinet proxy/security product that can provide capabilities such as:

* Web proxy
* SSL inspection
* Web filtering
* Application control
* DLP-related controls
* Antivirus/content inspection
* IPS-related inspection
* CASB-related capabilities
* OCR-related functionality in supported scenarios

ICAP is different:

```text
ICAP
  ↓
Protocol for integrating
external content-adaptation systems
```

Whereas:

```text
FortiProxy
  ↓
Fortinet security/proxy platform
```

### Key Difference

> **ICAP is an integration protocol; FortiProxy is a security product/platform.**

---

# 45. Common ICAP Solutions

Examples of ICAP-capable or ICAP-related products/projects include:

| Solution                       | Role                                                      |
| ------------------------------ | --------------------------------------------------------- |
| **C-ICAP**                     | Open-source ICAP server                                   |
| **Squid**                      | Proxy platform with ICAP integration                      |
| **Symantec Protection Engine** | Security/content scanning integration                     |
| **OPSWAT**                     | Content/file security capabilities with ICAP integrations |
| **Trend Micro**                | Security/content inspection integrations                  |

> Product capabilities and ICAP support vary by version and deployment model.

---

# 46. ICAP Security Architecture

A production-grade design can look like:

```text
                         INTERNET
                            │
                            ▼
                    ┌───────────────┐
                    │   FortiGate   │
                    │               │
                    │ Web Proxy     │
                    │ SSL Inspection│
                    │ ICAP Client   │
                    └───────┬───────┘
                            │
                     ICAP / Secure ICAP
                            │
             ┌──────────────┴──────────────┐
             ▼                             ▼
       ┌───────────┐                 ┌───────────┐
       │ ICAP #1   │                 │ ICAP #2   │
       │ AV / DLP  │                 │ AV / DLP  │
       └───────────┘                 └───────────┘
             │                             │
             └──────────────┬──────────────┘
                            ▼
                       FortiGate
                            │
                            ▼
                           LAN
```

---

# 47. ICAP High Availability

For production environments:

```text
                 FortiGate
                     │
           ┌─────────┴─────────┐
           ▼                   ▼
       ICAP #1              ICAP #2
           │                   │
           └─────────┬─────────┘
                     ▼
                Inspection
```

Consider:

* Multiple ICAP servers
* Load distribution
* Health checks
* Connection limits
* Failure behavior
* Network latency
* Server CPU/memory
* Content scanning capacity

---

# 48. Performance Considerations

ICAP introduces additional processing:

```text
Client
  ↓
FortiGate
  ↓
ICAP
  ↓
FortiGate
  ↓
Client
```

Compared with:

```text
Client
  ↓
FortiGate
  ↓
Client
```

Potential impacts:

* Additional latency
* Additional TCP connections
* Additional CPU
* Additional memory
* ICAP server resource usage
* WAD resource consumption
* Network bandwidth between FortiGate and ICAP

### Optimization Strategy

```text
Selective ICAP
      +
Streaming Bypass
      +
Content-Type Filtering
      +
Response-Code Filtering
      +
Proper Capacity Planning
```

---

# 49. Troubleshooting Checklist

## ICAP Server Unreachable

```text
[ ] ICAP server IP
[ ] Routing
[ ] TCP/1344
[ ] Secure ICAP port
[ ] Firewall policy
[ ] ICAP service running
[ ] Listening socket
[ ] Health check
[ ] VDOM routing
```

---

## Test Connectivity

```cli
execute telnet 192.168.20.200 1344
```

On ICAP server:

```bash
systemctl status c-icap
```

Check port:

```bash
netstat -tulnp | grep 1344
```

---

# 50. HTTP Works but ICAP Is Not Triggered

Check:

```text
[ ] ICAP feature enabled
[ ] ICAP server configured
[ ] ICAP profile attached
[ ] Proxy inspection enabled
[ ] REQMOD enabled
[ ] RESPMOD enabled
[ ] Forwarding rule matches
[ ] Host matches
[ ] Content-Type matches
[ ] HTTP status code matches
[ ] Streaming bypass
[ ] Default action
```

---

# 51. HTTPS Traffic Is Not Inspected

Check:

```text
[ ] SSL inspection enabled
[ ] Correct inspection mode
[ ] Certificate trusted by clients
[ ] HTTPS traffic decrypted
[ ] ICAP profile attached
[ ] Response/request inspection enabled
[ ] Content matches forwarding rules
```

Mental model:

```text
HTTPS
  ↓
SSL Decryption
  ↓
HTTP Content
  ↓
ICAP
```

If decryption does not happen:

```text
Encrypted Content
       ↓
No HTTP payload visibility
       ↓
ICAP content inspection may not occur
```

---

# 52. ICAP Server Returns 503

```text
503 Service Overloaded
```

Investigate:

```text
[ ] ICAP connection limit
[ ] Server CPU
[ ] Server memory
[ ] ICAP worker capacity
[ ] FortiGate connection rate
[ ] Number of clients
[ ] Large file inspection
[ ] Streaming traffic
```

Architecture:

```text
FortiGate
    │
    │ Too many requests
    ▼
ICAP
    │
    ▼
503
```

---

# 53. ICAP Causes Slow Web Browsing

Investigate:

```text
Client
  ↓
FortiGate
  ↓
ICAP
  ↓
FortiGate
  ↓
Internet
```

Measure:

```text
FortiGate CPU
FortiGate Memory
WAD
ICAP CPU
ICAP Memory
ICAP latency
Network latency
Content size
Concurrent connections
```

Optimization:

```text
Selective forwarding
        ↓
Only inspect required content
        ↓
Lower ICAP load
        ↓
Lower latency
```

---

# 54. 🔥 Fast NSE Exam Notes

| Topic                | Remember                                                  |
| -------------------- | --------------------------------------------------------- |
| **ICAP**             | Internet Content Adaptation Protocol                      |
| **FortiGate Role**   | ICAP client                                               |
| **ICAP Server**      | External content adaptation/inspection                    |
| **REQMOD**           | Request processing                                        |
| **RESPMOD**          | Response processing                                       |
| **1344**             | Common ICAP TCP port                                      |
| **11344**            | Secure ICAP port in the described FortiGate configuration |
| **204**              | No modification needed                                    |
| **400**              | Bad request                                               |
| **404**              | Service not found                                         |
| **405**              | Method not allowed                                        |
| **408**              | Request timeout                                           |
| **418**              | Bad composition                                           |
| **500**              | Server error                                              |
| **501**              | Method not implemented                                    |
| **502**              | Bad gateway                                               |
| **503**              | Service overloaded                                        |
| **505**              | ICAP version not supported                                |
| **SSL Inspection**   | Required when encrypted HTTPS content must be inspected   |
| **Proxy Mode**       | Important for ICAP web-content processing                 |
| **Streaming Bypass** | Reduces unnecessary ICAP processing                       |
| **204 Response**     | No content modification required                          |
| **Fail Open**        | Better availability, lower inspection assurance           |
| **Fail Closed**      | Stronger inspection assurance, lower availability         |
| **Content-Type**     | Can be used for selective forwarding                      |
| **HTTP Status Code** | Can be used in response forwarding rules                  |
| **WAD**              | Web proxy processing component                            |
| **C-ICAP**           | Open-source ICAP server implementation                    |

---

# 55. 🧠 ICAP Mental Model

The easiest way to remember ICAP:

```text
               FORTIGATE
                   │
                   ▼
             HTTP / HTTPS
                   │
                   ▼
             SSL Inspection
                   │
                   ▼
            Content Extraction
                   │
                   ▼
                 ICAP
                   │
          ┌────────┼────────┐
          ▼        ▼        ▼
         AV       DLP    Filtering
          │        │        │
          └────────┼────────┘
                   ▼
              ICAP Result
                   │
                   ▼
               FortiGate
                   │
          ┌────────┼────────┐
          ▼        ▼        ▼
         IPS     Web      App
                Filter   Control
                   │
                   ▼
                 Client
```

---

# 56. 🎯 Production Design Pattern

```text
                         INTERNET
                            │
                            ▼
                    ┌───────────────┐
                    │   FortiGate   │
                    │               │
                    │ Proxy         │
                    │ SSL Inspect   │
                    │ ICAP Client   │
                    └───────┬───────┘
                            │
                            │ Selected Content
                            ▼
                    ┌───────────────┐
                    │  ICAP Server  │
                    │               │
                    │ AV            │
                    │ DLP           │
                    │ Filtering     │
                    │ Content Scan  │
                    └───────┬───────┘
                            │
                            ▼
                    ┌───────────────┐
                    │   FortiGate   │
                    │               │
                    │ IPS           │
                    │ Web Filter    │
                    │ App Control   │
                    └───────┬───────┘
                            │
                            ▼
                           LAN
```

---

# 57. ⚠️ Common ICAP Design Mistakes

### ❌ Sending everything to ICAP

```text
All Web Traffic
      ↓
ICAP
```

This can unnecessarily increase:

* CPU
* Memory
* Latency
* ICAP connections
* ICAP server load

### Better

```text
Only Required Content
        ↓
ICAP
```

---

### ❌ Forgetting HTTPS Decryption

```text
HTTPS
  ↓
Still Encrypted
  ↓
No HTTP Content Visibility
```

If the security requirement is content inspection, the decryption architecture must be considered.

---

### ❌ Ignoring ICAP Failure Behavior

Always decide:

```text
ICAP Down
   │
   ├── Bypass?
   │
   └── Block/Error?
```

---

### ❌ Ignoring ICAP Capacity

```text
FortiGate Capacity
        +
ICAP Capacity
        +
Network Capacity
        =
Real Deployment Capacity
```

---

### ❌ Inspecting Streaming Content Unnecessarily

Large streaming objects can consume significant ICAP resources.

Use:

```text
streaming-content-bypass
```

when appropriate.

---

# 58. 🔬 Troubleshooting Decision Tree

```text
                 Web Problem
                     │
                     ▼
              Does HTTP work?
                /          \
              NO            YES
              │              │
              ▼              ▼
        Check Policy      ICAP issue?
        Routing           │
        SSL               ▼
        DNS          Is ICAP reachable?
                         /       \
                       NO         YES
                       │           │
                       ▼           ▼
                    Routing     Does rule
                    Port        match?
                    Service      /    \
                    Health     NO      YES
                               │        │
                               ▼        ▼
                           Forwarding  ICAP
                           Rules       Server
                           Profile     Response
```

---

# 59. 🚀 ICAP Optimization Checklist

```text
[ ] Use only required ICAP inspection
[ ] Avoid unnecessary streaming inspection
[ ] Select REQMOD vs RESPMOD carefully
[ ] Use Content-Type filtering
[ ] Use HTTP response-code filtering
[ ] Configure health checks
[ ] Define failure behavior
[ ] Monitor ICAP connection usage
[ ] Monitor WAD resources
[ ] Monitor ICAP CPU/memory
[ ] Use multiple ICAP servers for critical environments
[ ] Validate HTTPS decryption
[ ] Test large files
[ ] Test concurrent users
[ ] Test ICAP server failure
[ ] Test recovery after ICAP returns
```

---

# 60. 🏆 Final ICAP  

```text
                 ICAP
                  │
      ┌───────────┴───────────┐
      ▼                       ▼
    REQMOD                  RESPMOD
      │                       │
   Request                  Response
      │                       │
      └───────────┬───────────┘
                  ▼
             ICAP Server
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
       AV        DLP      Filter
        │         │         │
        └─────────┼─────────┘
                  ▼
              FortiGate
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
       IPS      WebFilter  AppControl
                  │
                  ▼
                Client
```

---

## ⭐ One-Line Memory Trick

```text
REQMOD = Request
RESPMOD = Response
1344 = ICAP
204 = No Modification
503 = ICAP Overloaded
HTTPS = Decrypt First, Inspect Second
Selective Forwarding = Better Performance
```

---

> **SheynShield Engineering Note**
>
> ICAP should not be viewed simply as *“send web traffic to another server.”* The real design question is **what content should leave FortiGate for external inspection, why it needs inspection, and what happens when the ICAP server is unavailable**.
>
> A strong production design therefore follows:
>
> **Proxy → SSL Decryption → Content Selection → ICAP → Security Profiles → Client**
>
> And for performance:
>
> **Selective ICAP > Inspect Everything**

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

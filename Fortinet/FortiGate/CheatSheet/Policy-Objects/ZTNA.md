# 🔐 FortiGate ZTNA   — FortiOS 7.2+

> **SheynShield | Engineering Secure Networks**
> Practical FortiGate ZTNA reference covering **FortiClient EMS, FortiClient, ZTNA Tags, Access Proxy, TCP Forwarding, SSH Proxy, SAML, EMS synchronization, troubleshooting, and deployment decisions**.

---

## 📌 Table of Contents

* [ZTNA Architecture](#-ztna-architecture)
* [Version Compatibility](#-version-compatibility)
* [FortiClient EMS Preparation](#-forticlient-ems-preparation)
* [FortiGate ↔ EMS Integration](#-fortigate--ems-integration)
* [ZTNA Modes](#-ztna-modes)
* [ZTNA Tags](#-ztna-tags)
* [ZTNA Server](#-ztna-server)
* [ZTNA Access Proxy](#-ztna-access-proxy)
* [Transparent vs Non-Transparent Access Proxy](#-transparent-vs-non-transparent-access-proxy)
* [TCP Forwarding Access Proxy](#-tcp-forwarding-access-proxy)
* [SSH Access Proxy](#-ssh-access-proxy)
* [SAML + ZTNA](#-saml--ztna)
* [ZTNA with FortiAuthenticator](#-ztna-with-fortiauthenticator)
* [ZTNA vs SSL VPN](#-ztna-vs-ssl-vpn)
* [EMS Endpoint Information](#-ems-endpoint-information)
* [FortiGate Endpoint Synchronization](#-fortigate-endpoint-synchronization)
* [ZTNA Tag Scaling](#-ztna-tag-scaling)
* [EMS Fast Convergence](#-ems-fast-convergence)
* [Troubleshooting](#-troubleshooting)
* [Operational Checklist](#-operational-checklist)
* [Key Commands](#-key-commands)
* [Design Rules](#-design-rules)

---

# 🧠 ZTNA Architecture

A typical Fortinet ZTNA deployment contains:

```text
                    ┌─────────────────────┐
                    │ FortiClient EMS     │
                    │                     │
                    │ • Endpoint Identity │
                    │ • ZTNA Tags         │
                    │ • Certificates      │
                    │ • Telemetry         │
                    └──────────┬──────────┘
                               │
                         Endpoint Sync
                               │
                               ▼
                    ┌─────────────────────┐
                    │     FortiGate       │
                    │                     │
                    │ • ZTNA Policy       │
                    │ • Access Proxy      │
                    │ • Identity          │
                    │ • Posture Check     │
                    └──────────┬──────────┘
                               │
                         Access Proxy
                               │
             ┌─────────────────┼─────────────────┐
             ▼                 ▼                 ▼
          HTTP/HTTPS          SSH              RDP/SMB
             │                 │                 │
             └─────────────────┼─────────────────┘
                               ▼
                    ┌─────────────────────┐
                    │ Protected Servers   │
                    └─────────────────────┘
```

### Core ZTNA concept

ZTNA does not simply ask:

> "What is the source IP?"

Instead, access decisions can consider:

* **Device identity**
* **Client certificate**
* **User identity**
* **ZTNA tags**
* **Endpoint security posture**
* **Network location**
* **Authentication state**
* **Application/service**
* **SAML/LDAP/RADIUS identity**

---

# 🔄 Version Compatibility

For a FortiOS ZTNA deployment:

| Component                         | Recommendation                          |
| --------------------------------- | --------------------------------------- |
| FortiGate                         | FortiOS 7.2+                            |
| FortiClient EMS                   | Same major/minor version family         |
| FortiClient                       | Prefer 7.2+                             |
| FortiClient on Windows            | Required for some endpoint capabilities |
| FortiClient Windows SMB detection | FortiClient 7.0.3+                      |

### ⚠️ Version Rule

If using:

```text
FortiGate 7.2.x
        ↓
FortiClient EMS 7.2.x
        ↓
FortiClient 7.2.x+
```

keep the platform versions aligned whenever possible.

---

# 🏢 FortiClient EMS Preparation

After installing FortiClient EMS:

### 1. FortiCare / FortiCloud

The EMS deployment requires the appropriate Fortinet account registration.

```text
FortiCare / FortiCloud
        │
        ▼
FortiClient EMS
        │
        ▼
FortiGate
```

After EMS installation/registration, EMS provides the required registration information/token for integration with the Fortinet cloud environment.

---

## ⚙️ EMS System Settings

Navigate to:

```text
System Settings
└── EMS Settings
```

Important areas:

* FortiClient download location
* EMS certificates
* Certificate Authority
* EMS connectivity information

### Certificates

The EMS certificate infrastructure is important for trust relationships between:

```text
FortiClient
     │
     ▼
FortiClient EMS
     │
     ▼
FortiGate
```

The EMS ZTNA CA can issue endpoint certificates used for device identification.

---

# 👤 EMS Authentication Server

Navigate to:

```text
Administrator
└── Authentication Server
```

Typical LDAP configuration:

```text
LDAP Server
    IP      : <LDAP-IP>
    Port    : 389
    Account : <LDAP-Service-Account>
```

> 🔐 Never commit LDAP passwords, API tokens, private keys, or service-account credentials to GitHub.

---

# 🌐 EMS Endpoint Management

Navigate to:

```text
Endpoint
└── Manage Domains
```

This allows EMS to collect/manage endpoint and user-related information.

EMS can maintain information such as:

* Device information
* Logged-on user
* Operating system
* Network information
* Security posture
* Vulnerability state
* Endpoint identity

---

# 🏷️ ZTNA Tags

Navigate to:

```text
Zero Trust Tags
└── Rules
```

ZTNA Tags are one of the most important mechanisms for dynamic access control.

Example:

```text
Tag:
    AV-Enabled

Rule:
    Windows endpoint
    +
    Windows security posture
```

Possible tag logic:

```text
Endpoint
   │
   ├── Antivirus enabled
   ├── Vulnerability state
   ├── OS
   ├── Security posture
   └── Other endpoint attributes
            │
            ▼
        ZTNA Tag
            │
            ▼
       FortiGate Policy
```

### Example

A policy can require:

```text
ZTNA Tag = AV-Enabled
```

If the endpoint loses the required posture:

```text
Tag removed
     ↓
ZTNA policy no longer matches
     ↓
Access denied / session affected
```

---

# 🔗 EMS Fabric Connector

Navigate on FortiGate:

```text
Security Fabric
└── Fabric Connectors
    └── FortiClient EMS
```

The FortiGate must be able to communicate with EMS.

Verify:

* EMS connectivity
* Certificate trust
* Fabric authorization
* Endpoint synchronization
* ZTNA tags

---

# 🔐 Certificate Trust

The FortiGate and EMS must establish a trusted relationship.

Typical flow:

```text
FortiClient
     │
     │ Client Certificate
     ▼
FortiGate
     │
     │ Endpoint UID / Certificate SN
     ▼
EMS synchronized information
```

If certificate trust fails, ZTNA authentication and device identity validation can fail.

---

# 🌍 ZTNA Modes

## OFF-NET

The endpoint is outside the trusted/local network.

```text
Internet
   │
   ▼
FortiClient
   │
   ▼
FortiGate ZTNA Access Proxy
   │
   ▼
Internal Application
```

Typical remote-access scenario.

---

## ON-NET

The endpoint is inside the organization's network.

ZTNA can evaluate the endpoint's network position and posture.

```text
Corporate LAN
      │
      ▼
FortiClient
      │
      ▼
ZTNA Decision
```

---

# 🚪 ZTNA Access Proxy

One of the most useful FortiGate ZTNA scenarios is publishing internal applications/services through an **Access Proxy**.

It behaves similarly to a reverse proxy:

```text
Remote Client
      │
      │ HTTPS / SSH / TCP
      ▼
FortiGate Access Proxy
      │
      │ Identity + Device + Tag
      ▼
Internal Server
```

Supported use cases can include:

* HTTP
* HTTPS
* SSH
* RDP
* SMB
* FTP
* Other TCP-based applications

---

# 🧩 Access Proxy Decision Model

A typical ZTNA decision can be represented as:

```text
Client
  │
  ├── Client Certificate
  │
  ├── User Identity
  │
  ├── ZTNA Tag
  │
  ├── Endpoint Posture
  │
  └── Network Context
          │
          ▼
      FortiGate
          │
          ▼
   ZTNA Proxy Policy
          │
       ┌──┴──┐
       │     │
     Allow  Deny
       │
       ▼
 Internal Application
```

---

# 🛡️ ZTNA Server

Enable the feature if necessary:

```text
System
└── Feature Visibility
    └── Zero Trust Network Access
```

Then:

```text
Policy & Objects
└── ZTNA
    └── Server
```

Typical configuration:

```text
Name:
    ztna-server

External Interface:
    port3 / WAN

External IP:
    <ZTNA-Public-IP>

External Port:
    443

Certificate:
    <ZTNA-Certificate>

SAML:
    <SAML-Server>
```

---

# 🌐 Virtual Host

A virtual host allows a dedicated hostname/application to be published through the access proxy.

Conceptually:

```text
app.example.com
       │
       ▼
FortiGate Access Proxy
       │
       ▼
Internal Application
```

### Any Host

Can be used when the access proxy should accept requests for any host.

### Specific Host

Use a dedicated:

* Hostname
* IP
* SNI
* Service mapping

for more controlled application publishing.

---

# 🏷️ ZTNA Tags on FortiGate

EMS tags are synchronized to FortiGate and can be used in policy matching.

Useful commands:

```bash
diagnose firewall dynamic address
```

```bash
diagnose firewall dynamic list
```

These help verify dynamically created ZTNA addresses/tags.

---

# 🔥 ZTNA Proxy Policy

Example structure:

```bash
config firewall proxy-policy
    edit 1
        set name "ztna-policy"
        set proxy access-proxy
        set access-proxy "acp-test"

        set srcintf "port3"
        set srcaddr "all"

        set dstaddr "all"

        set ztna-ems-tag "AV-Enabled"
        set ztna-tags-match-logic and

        set action accept
        set schedule "always"

        set logtraffic all

        set utm-status enable
        set ssl-ssh-profile "deep-inspection"
    next
end
```

### Tag Matching

```text
AND
```

means all required tags must match.

```text
OR
```

means at least one matching tag is sufficient.

---

# 🧠 Important: Tag Logic Changes Can Affect Sessions

Example:

```text
Initial:
AV-Enabled OR Low-Risk
```

A user may be allowed.

If the administrator changes the logic to:

```text
AV-Enabled AND Low-Risk
```

the endpoint may no longer satisfy the policy and the existing session can be terminated.

---

# 🔄 Transparent Access Proxy

Example:

```bash
set transparent enable
```

Transparent mode can simplify client-side deployment.

Conceptually:

```text
Client
   │
   ▼
FortiGate
   │
   ▼
Internal Server
```

The client can perceive the connection as direct rather than requiring a traditional explicit-proxy configuration.

---

# 🧪 TCP Forwarding Access Proxy

TCP forwarding can reduce encryption overhead for protocols that already provide their own encryption.

Examples:

* RDP
* SSH
* FTPS

The connection still begins with a TLS handshake, but the proxy can switch to TCP forwarding.

Conceptually:

```text
Client
 │
 │ TLS handshake
 ▼
Access Proxy
 │
 │ TCP forwarding
 ▼
Server
```

### Important

TCP forwarding does **not automatically make an insecure application secure**.

If the underlying protocol is insecure, use appropriate end-to-end encryption.

---

# 🔍 TCP Forwarding + UTM

TCP forwarding access proxy can support security inspection for supported protocols, including scenarios involving:

* HTTP
* HTTPS
* SMTP
* SMTPS
* IMAP
* IMAPS
* POP3
* POP3S
* SMB
* CIFS

---

# 🔐 Access Proxy Client Certificate Authentication

By default, client certificate authentication can be enabled on the access proxy.

Flow:

```text
Client
  │
  │ HTTPS Request
  ▼
FortiGate WAD
  │
  │ Client Certificate Challenge
  ▼
FortiClient
```

---

## Certificate Match — SUCCESS

If the client sends a valid certificate:

```text
Certificate
     │
     ├── Client UID
     └── Certificate Serial Number
              │
              ▼
       FortiGate Endpoint DB
              │
              ▼
            Match
              │
              ▼
      Continue ZTNA Policy
```

---

## Certificate Match — FAILURE

If UID/certificate information does not match the FortiGate endpoint record:

```text
Certificate mismatch
       ↓
ZTNA processing blocked
```

---

# 🚫 Empty Client Certificate

If the client sends an empty certificate:

```bash
config firewall proxy-policy
    edit "ztna-test"
        set empty-cert-action block
    next
end
```

Possible values:

```text
accept
block
```

### Recommended security posture

For certificate-based device trust:

```text
empty-cert-action = block
```

unless there is a deliberate design reason to allow empty certificates.

---

# 🔧 Access Proxy — CLI Skeleton

## VIP

```bash
config firewall vip
    edit "ZTNA-VIP"
        set type access-proxy
        set extip <PUBLIC-IP>
        set extintf "port3"
        set server-type https
        set extport 8443
        set ssl-certificate "ZTNA-CERT"
    next
end
```

> ⚠️ When configuring certain Access Proxy VIP attributes, prefer consistent CLI/GUICLI configuration and verify the resulting configuration with `show/full-configuration`.

---

## Access Proxy

```bash
config firewall access-proxy
    edit "ZTNA-AP"
        set vip "ZTNA-VIP"
        set client-cert enable

        config api-gateway
            edit 1
                set service https

                config realservers
                    edit 1
                        set ip <INTERNAL-SERVER-IP>
                        set port 443
                    next
                end
            next
        end
    next
end
```

---

# 🔑 SAML + ZTNA

SAML allows ZTNA to combine:

```text
Device Identity
       +
User Identity
       +
ZTNA Tags
       +
Application Access
```

Typical flow:

```text
FortiClient
    │
    ▼
FortiGate Access Proxy
    │
    ▼
SAML Authentication
    │
    ▼
Identity Provider
    │
    ▼
User Authentication
    │
    ▼
ZTNA Policy
    │
    ▼
Internal Application
```

---

# 🧱 SAML Configuration Components

Typical objects:

```text
config user saml
```

```text
config authentication scheme
```

```text
config authentication rule
```

```text
config firewall access-proxy
```

```text
config firewall proxy-policy
```

---

## SAML Authentication Example

```bash
config user saml
    edit "saml_ztna"
        set cert "ZTNA-CERT"
        set entity-id "<SP-ENTITY-ID>"
        set single-sign-on-url "<SP-SSO-URL>"
        set single-logout-url "<SP-SLO-URL>"

        set idp-entity-id "<IDP-ENTITY-ID>"
        set idp-single-sign-on-url "<IDP-SSO-URL>"
        set idp-single-logout-url "<IDP-SLO-URL>"
        set idp-cert "<IDP-CERT>"

        set digest-method sha256
    next
end
```

---

# 🍪 Web Authentication Cookie

For SAML authentication rules using non-IP-based authentication:

```bash
set ip-based disable
set web-auth-cookie enable
```

This is important because the authentication state must persist beyond a simple IP-based session model.

---

# 👥 ZTNA Policy + User Group

Example conceptual policy:

```text
Source
   +
User Group
   +
ZTNA Server
   +
ZTNA Tag
   +
Application
   ↓
ALLOW
```

This creates a much stronger control model than IP-only access.

---

# 🏢 SAML + FortiAuthenticator

A common architecture:

```text
                  ┌────────────────────┐
                  │ FortiAuthenticator │
                  │       / IdP        │
                  └─────────┬──────────┘
                            │
                           SAML
                            │
                            ▼
                    ┌───────────────┐
                    │   FortiGate   │
                    │ ZTNA Gateway  │
                    └───────┬───────┘
                            │
                       Access Proxy
                            │
                            ▼
                    Internal Server
```

FortiAuthenticator can provide:

* LDAP integration
* User authentication
* SAML Identity Provider functionality

---

# 🔐 SSH Access Proxy

SSH Access Proxy provides additional security capabilities compared with simple TCP forwarding.

Benefits include:

* Device trust validation
* User identity validation
* SSH deep inspection
* SSH host-key validation
* One-time user authentication
* SSH certificate-based authentication

---

# 🔄 SSH ZTNA Authentication Flow

```text
1. Endpoint registers with EMS
             │
             ▼
2. EMS issues client certificate
             │
             ▼
3. Client connects to SSH Access Proxy
             │
             ▼
4. FortiGate validates device identity
             │
             ▼
5. Client presents EMS certificate
             │
             ▼
6. FortiGate authenticates user
             │
             ▼
7. FortiGate obtains username
             │
             ▼
8. FortiGate signs SSH certificate
             │
             ▼
9. FortiGate connects to SSH server
             │
             ▼
10. SSH server validates certificate
             │
             ▼
11. Username/principal checked
             │
             ▼
12. SSH session established
```

---

# 🔑 SSH Host-Key Validation

The SSH Access Proxy can validate the destination SSH server's host key.

Conceptually:

```text
Client
  │
  ▼
FortiGate SSH Proxy
  │
  │ Validate server host key
  ▼
SSH Server
```

Example:

```bash
config firewall ssh host-key
    edit "ssh-server-key"
        set type rsa
        set usage access-proxy
        set public-key "<SSH-SERVER-PUBLIC-KEY>"
    next
end
```

---

# 🔐 SSH Client Certificate

Example structure:

```bash
config firewall access-proxy-ssh-client-cert
    edit "ssh-access-proxy"
        set source-address enable
        set auth-ca "ZTNA-CERT"
    next
end
```

---

# 🛠️ SSH Access Proxy Configuration Concept

```bash
config firewall access-proxy
    edit "ZTNA-SSH"
        set vip "ZTNA-SSH"
        set client-cert enable

        config api-gateway
            edit 1
                set url-map tcp
                set service tcp-forwarding

                config realservers
                    edit 1
                        set address <SSH-SERVER-IP>
                        set type ssh
                        set ssh-client-cert "ssh-access-proxy"
                        set ssh-host-key-validation enable
                        set ssh-host-key "ssh-server-key"
                    next
                end
            next
        end
    next
end
```

---

# 🧮 ZTNA Tags and Firewall Address Consumption

### Critical Scaling Concept

Each ZTNA tag creates **two firewall address objects per VDOM**:

```text
ZTNA Tag
   │
   ├── IP-based firewall address
   │
   └── MAC-based firewall address
```

Therefore:

```text
ZTNA Tag ≈ 2 firewall addresses
```

The practical maximum number of ZTNA tags is therefore constrained by the FortiGate's:

* Global firewall address limit
* Per-VDOM firewall address limit

Conceptually:

```text
Maximum ZTNA Tags
        ≈
Firewall Address Limit / 2
```

> ⚠️ Always verify the exact maximum values for the specific FortiGate model and FortiOS release.

### Design Rule

When capacity planning ZTNA:

```text
1 ZTNA Tag
   =
1 IP address object
+
1 MAC address object
```

Do **not** calculate tag capacity as if one tag consumed only one firewall address.

---

# 📡 EMS Endpoint Information

When FortiClient registers with EMS, EMS can obtain:

### Device Information

* Operating system
* Model
* Network information
* Device attributes

### User Information

* Logged-on user
* User/domain information

### Security Posture

* On-net / Off-net
* Antivirus status
* Vulnerability status
* Endpoint security information

---

# 📜 Endpoint Certificate

During registration, FortiClient can obtain a client certificate from the EMS ZTNA CA.

```text
FortiClient
     │
     │ Registration
     ▼
FortiClient EMS
     │
     │ Client Certificate
     ▼
FortiClient
```

The client certificate is then used by the FortiGate for endpoint/device identification.

---

# 🔄 FortiGate Endpoint Synchronization

FortiGate maintains a connection to EMS to synchronize endpoint information.

Important information includes:

```text
FortiClient UID
Certificate Serial Number
EMS Serial Number
Device Credentials
User / Domain
IP Address
MAC Address
Network Information
Routing Context
```

---

# ⚡ EMS Fast Convergence

For larger ZTNA deployments, EMS/FortiGate synchronization should be optimized for fast convergence.

Example:

```bash
config endpoint-control fctems
    edit "ems-test"
        set server <EMS-IP>

        set capabilities fabric-auth silent-approval websocket websocket-malware push-ca-certs common-tags-api

        set call-timeout 30
        set out-of-sync-threshold 180

        set preserver-ssl-session disable
        set websocket-override disable
    next
end
```

### Key Parameters

| Parameter               | Purpose                                          |
| ----------------------- | ------------------------------------------------ |
| `capabilities`          | Enables supported EMS integration capabilities   |
| `call-timeout`          | Controls EMS synchronization call timeout        |
| `out-of-sync-threshold` | Defines when EMS state is considered out-of-sync |
| `preserver-ssl-session` | Controls SSL session preservation                |
| `websocket-override`    | Controls WebSocket behavior                      |

---

# 🚀 Why Fast Convergence Matters

ZTNA decisions depend heavily on current endpoint posture.

Example:

```text
Endpoint Healthy
      ↓
ZTNA Tag = AV-Enabled
      ↓
Access Granted
```

Endpoint becomes unhealthy:

```text
Antivirus Disabled
      ↓
ZTNA Tag Removed
      ↓
EMS
      ↓
FortiGate
      ↓
ZTNA Policy Re-evaluated
      ↓
Access Denied
```

The faster the synchronization, the faster the access decision reflects the new endpoint state.

---

# 📈 ZTNA Scalability

FortiGate ZTNA architecture can scale to very large endpoint populations; the supported scale depends on the FortiGate model, FortiOS release, EMS architecture, and configured features.

For high-scale environments:

* Use appropriate EMS architecture
* Enable supported fast-convergence capabilities
* Monitor endpoint synchronization
* Monitor dynamic address consumption
* Monitor WAD resources
* Validate platform-specific maximum values

---

# 🧪 ZTNA Troubleshooting

## 1. Check Endpoint Record

```bash
diagnose endpoint record list
```

Filter by IP when needed:

```bash
diagnose endpoint record list <IP>
```

---

## 2. Query by UID

```bash
diagnose endpoint lls-comm send ztna find-uid <UID>
```

Use this when you know the FortiClient UID.

---

## 3. Query by IP + VDOM

```bash
diagnose endpoint lls-comm send ztna find-ip-vdom <IP> <VDOM>
```

Example:

```bash
diagnose endpoint lls-comm send ztna find-ip-vdom 192.168.1.10 root
```

---

# 🔍 WAD Endpoint Queries

Query by UID:

```bash
diagnose wad dev query-by uid <UID>
```

Query by IPv4:

```bash
diagnose wad dev query-by ipv4 <IP>
```

These are useful for investigating how the WAD process sees the endpoint.

---

# 🧠 FCNACD Diagnostics

Check ZTNA/NAC state:

```bash
diagnose test application fcnacd 7
```

```bash
diagnose test application fcnacd 8
```

Additional EMS connectivity information:

```bash
diagnose test application fcnacd 2
```

---

# 🔗 Test EMS Connectivity

```bash
diagnose endpoint fctems test-connectivity <EMS>
```

Use this to verify FortiGate → EMS connectivity.

---

# 📜 Verify EMS Certificate

```bash
execute fctems verify <EMS>
```

Useful for checking EMS certificate verification.

---

# 🧾 Dynamic ZTNA Addresses

```bash
diagnose firewall dynamic list
```

Use this to inspect:

* EMS ZTNA tags
* Dynamic IP addresses
* Dynamic MAC addresses

---

# 🌐 FQDN / Dynamic Address Diagnostics

```bash
diagnose firewall fqdn getinfo-ip <ADDRESS>
```

```bash
diagnose firewall dynamic address
```

---

# 👤 WAD User Diagnostics

```bash
diagnose wad user list
```

Useful when troubleshooting:

* SAML
* Basic authentication
* User identity
* Access Proxy sessions

---

# 📊 WAD Policy Statistics

```bash
diagnose wad worker policy list
```

Useful for checking Access Proxy policy statistics.

---

# 🐛 WAD Debug

Enable verbose WAD debugging:

```bash
diagnose wad debug enable category all
diagnose wad debug enable level verbose
diagnose debug enable
```

After troubleshooting:

```bash
diagnose debug reset
```

> ⚠️ Avoid leaving verbose debugging enabled in production longer than necessary.

---

# 🔬 TCP Forwarding Debug

When debugging TCP forwarding, WAD output can show:

```text
tls=0
```

This can indicate that the proxy-side TCP forwarding connection is not using an additional TLS encryption layer.

Conceptually:

```text
TLS Handshake
     ↓
HTTP 101 Switching Protocols
     ↓
TCP Forwarding
     ↓
Underlying Protocol
```

---

# 🧪 Explicit Proxy vs Access Proxy

## Access Proxy

Designed for application publishing and ZTNA-style access control.

```text
Client
  │
  ▼
Access Proxy
  │
  ├── Device Identity
  ├── Certificate
  ├── User Authentication
  ├── ZTNA Tags
  └── Application Policy
       │
       ▼
    Server
```

Typical objects:

```text
Firewall VIP
Firewall Access Proxy
Proxy Policy
ZTNA Tags
Authentication
```

---

## Explicit Proxy

Designed around explicit proxy traffic.

Conceptually:

```text
Client
   │
   │ Explicit Proxy Configuration
   ▼
FortiGate Proxy
   │
   ▼
Internet / Application
```

Authentication rules and proxy policies are used differently from ZTNA Access Proxy.

### Important

Do not confuse:

```text
Explicit Proxy
```

with:

```text
ZTNA Access Proxy
```

They solve different access-control problems.

---

# 🛡️ ZTNA vs SSL VPN vs IPsec VPN

| Requirement                           | Recommended         |
| ------------------------------------- | ------------------- |
| Full network access                   | SSL VPN / IPsec VPN |
| Access to network ranges              | VPN                 |
| Publish specific applications         | ZTNA                |
| Remote access to web applications     | ZTNA                |
| Per-application access                | ZTNA                |
| Device posture enforcement            | ZTNA                |
| BYOD application access               | ZTNA                |
| Micro-segmentation                    | ZTNA                |
| Large application-publishing scenario | ZTNA                |
| Secure site-to-site connectivity      | IPsec               |

### Practical Rule

If the requirement is:

> "I need access to many networks and subnets."

Think:

```text
VPN
```

If the requirement is:

> "I need access to these specific applications/services."

Think:

```text
ZTNA
```

---

# 🧭 ZTNA Deployment Decision

```text
                    Remote Access
                         │
             ┌───────────┴───────────┐
             │                       │
      Need network access?     Need application access?
             │                       │
            YES                      YES
             │                       │
             ▼                       ▼
      VPN / IPsec             ZTNA Access Proxy
                                     │
                           ┌─────────┴─────────┐
                           │                   │
                       Web App            TCP/SSH/RDP
                           │                   │
                       HTTP/S             TCP / SSH Proxy
```

---

# 📦 Endpoint Subscription Models

Fortinet licensing can include different endpoint capabilities and deployment models.

Conceptual categories include:

### ZTNA / VPN Agent

Suitable when the primary requirement is:

* ZTNA
* VPN
* Basic endpoint connectivity

---

### EPP / Advanced Protection

Provides broader endpoint security capabilities.

Typically appropriate when the organization also requires stronger endpoint protection.

---

### Managed ZTNA Agent

Cloud-managed endpoint management model.

Useful when the organization wants cloud-based management and future expansion.

---

### Cloud EMS

```text
Cloud EMS
   │
   ├── ZTNA
   ├── EPP
   └── Forensics / Advanced Services
```

---

### On-Premises EMS

```text
On-Prem EMS
   │
   ├── ZTNA
   ├── EPP
   └── Forensics
```

> 💡 Exact SKU names, entitlement codes, and licensing models can change by Fortinet program and release. Verify the current Fortinet licensing catalog before using SKU numbers in production documentation.

---

# 🐧 Linux Consideration

Some FortiClient security capabilities are platform-dependent.

For example, sandbox-related capabilities may not be available on Linux endpoints.

Always validate:

```text
OS
+
FortiClient Version
+
EMS Version
+
Licensed Feature
```

before designing a cross-platform deployment.

---

# 🧠 ZTNA Agent Responsibilities

The endpoint agent provides important security context.

### Endpoint Agent

```text
FortiClient
    │
    ├── Device Identity
    ├── Certificate
    ├── User Identity
    ├── Endpoint Health
    ├── Telemetry
    ├── Network State
    ├── On-Net / Off-Net
    └── Continuous Monitoring
```

It can also support:

* Transparent ZTNA connection steering
* SSO
* SAML
* Device certificate handling
* Endpoint telemetry

---

# 🚪 ZTNA Gateway Responsibilities

The FortiGate acts as the application gateway.

```text
FortiGate
    │
    ├── Application Access
    ├── Micro-Segmentation
    ├── Device Identity Validation
    ├── User Authentication
    ├── Per-Session Posture
    ├── SAML / SSO
    ├── Load Balancing
    └── Security Inspection
```

---

# 🔐 ZTNA Security Model

The strongest design is not:

```text
IP → Allow
```

Instead:

```text
WHO?
  +
WHAT DEVICE?
  +
WHAT POSTURE?
  +
WHAT APPLICATION?
  +
FROM WHERE?
  +
WHICH TAG?
  +
WHICH AUTHENTICATION?
       ↓
   ZTNA DECISION
```

---

# 🧩 ZTNA Order of Security Controls

A modern ZTNA design can combine multiple controls:

```text
CASB
 │
 ├── SaaS/Application Security
 │
 ├── DLP
 │
 ├── Granular Access Control
 │
 ├── Application Control
 │
 └── Data Protection / Encryption
```

The exact available feature set depends on the Fortinet solution stack and licensing.

---

# ✅ ZTNA Deployment Checklist

## EMS

* [ ] Install FortiClient EMS
* [ ] Register EMS with the appropriate Fortinet account
* [ ] Verify EMS certificate configuration
* [ ] Verify EMS CA
* [ ] Configure endpoint domains
* [ ] Configure LDAP if required
* [ ] Create ZTNA Tags
* [ ] Verify endpoint registration
* [ ] Verify FortiClient certificate issuance

---

## FortiGate

* [ ] Enable ZTNA feature visibility
* [ ] Configure EMS Fabric Connector
* [ ] Authorize/trust EMS certificates
* [ ] Verify EMS connectivity
* [ ] Verify endpoint synchronization
* [ ] Verify ZTNA tags
* [ ] Create ZTNA server/access proxy
* [ ] Configure VIP/access proxy
* [ ] Configure certificates
* [ ] Configure real servers
* [ ] Configure proxy policies
* [ ] Configure ZTNA tag matching
* [ ] Configure authentication
* [ ] Configure SAML if required
* [ ] Configure UTM inspection where appropriate

---

## Network Connectivity

* [ ] FortiClient → FortiGate reachable
* [ ] FortiGate → EMS reachable
* [ ] FortiGate → Internal Server reachable
* [ ] Required DNS records exist
* [ ] Required certificates are trusted
* [ ] Firewall policies allow EMS communication
* [ ] Firewall policies allow ZTNA traffic

---

# 🚨 Common ZTNA Failure Points

| Symptom                         | Check                                     |
| ------------------------------- | ----------------------------------------- |
| Client not recognized           | EMS registration / endpoint record        |
| Certificate failure             | EMS CA / FortiGate trust                  |
| Tag missing                     | EMS tag rule / synchronization            |
| User authentication failure     | LDAP / SAML                               |
| Application unreachable         | Access Proxy / VIP / real server          |
| RDP fails                       | TCP forwarding / server reachability      |
| SSH fails                       | SSH certificate / host key                |
| Policy does not match           | ZTNA tag logic                            |
| Session unexpectedly terminates | Tag/posture changed                       |
| EMS shows disconnected          | EMS connectivity / certificate            |
| WAD sees no endpoint            | Endpoint synchronization                  |
| Dynamic address missing         | EMS tag synchronization                   |
| SAML loop                       | SAML URLs / certificates / cookies        |
| BYOD access fails               | Client capability / authentication design |

---

# 🔎 Golden Troubleshooting Workflow

When ZTNA fails, do **not** immediately start debugging everything.

Follow this sequence:

```text
1. Is FortiClient registered?
        │
        ▼
2. Does EMS know the endpoint?
        │
        ▼
3. Does the endpoint have a client certificate?
        │
        ▼
4. Does FortiGate know the endpoint?
        │
        ▼
5. Are ZTNA tags synchronized?
        │
        ▼
6. Does the certificate UID/SN match?
        │
        ▼
7. Does authentication succeed?
        │
        ▼
8. Does the proxy policy match?
        │
        ▼
9. Can FortiGate reach the real server?
        │
        ▼
10. Is WAD forwarding the session correctly?
```

---

# 🧪 Minimal Troubleshooting Command Set

```bash
# Endpoint database
diagnose endpoint record list

# Dynamic ZTNA addresses/tags
diagnose firewall dynamic list

# EMS connectivity
diagnose endpoint fctems test-connectivity <EMS>

# EMS certificate verification
execute fctems verify <EMS>

# FCNACD
diagnose test application fcnacd 2
diagnose test application fcnacd 7
diagnose test application fcnacd 8

# WAD users
diagnose wad user list

# WAD endpoint
diagnose wad dev query-by uid <UID>
diagnose wad dev query-by ipv4 <IP>

# Endpoint lookup
diagnose endpoint lls-comm send ztna find-uid <UID>
diagnose endpoint lls-comm send ztna find-ip-vdom <IP> <VDOM>

# WAD policy statistics
diagnose wad worker policy list
```

---

# 🧠 Design Rules — Memorize These

> **1. ZTNA is application-centric, not simply network-centric.**

> **2. EMS is the source of endpoint identity and posture context.**

> **3. FortiClient provides endpoint identity, telemetry, posture, and certificate context.**

> **4. FortiGate enforces the ZTNA access decision.**

> **5. Client certificates provide device identity; SAML/LDAP can provide user identity.**

> **6. ZTNA Tags are dynamic security attributes, not ordinary static firewall addresses.**

> **7. One ZTNA tag consumes two firewall address objects per VDOM: IP + MAC.**

> **8. `AND` and `OR` tag matching can completely change the access decision.**

> **9. SSH Access Proxy provides stronger SSH-specific controls than generic TCP forwarding.**

> **10. TCP forwarding can reduce proxy encryption overhead, but it does not magically encrypt insecure application protocols.**

> **11. Keep FortiGate, EMS, and FortiClient versions aligned whenever possible.**

> **12. For full network access, VPN is generally the better architectural fit.**

> **13. For application-specific access, ZTNA is generally the better architectural fit.**

> **14. Always troubleshoot ZTNA from endpoint registration → EMS → certificate → synchronization → tag → authentication → policy → server.**

---

# 🧠 NSE Exam Memory Map

```text
                    ZTNA
                     │
        ┌────────────┼────────────┐
        │            │            │
      EMS        FortiClient   FortiGate
        │            │            │
        │            │            ├── Access Proxy
        │            │            ├── Proxy Policy
        │            │            ├── ZTNA Tags
        │            │            ├── SAML
        │            │            └── WAD
        │            │
        ├── Tags     ├── Certificate
        ├── CA       ├── Identity
        ├── Users    ├── Posture
        └── Endpoint └── Telemetry
                     │
                     ▼
              ZTNA DECISION
                     │
              ┌──────┴──────┐
              │             │
            ALLOW          DENY
              │
              ▼
       Internal Application
```

---

## 📚 Quick Reference

| Component          | Primary Responsibility                      |
| ------------------ | ------------------------------------------- |
| FortiClient        | Endpoint agent / identity / posture         |
| FortiClient EMS    | Endpoint management / tags / CA / telemetry |
| FortiGate          | ZTNA enforcement / proxy gateway            |
| WAD                | Web/access-proxy processing                 |
| FCNACD             | FortiClient NAC/endpoint communication      |
| ZTNA Tag           | Dynamic endpoint security attribute         |
| Client Certificate | Device identity                             |
| SAML               | User identity/authentication                |
| LDAP               | User/group directory                        |
| Access Proxy       | Application publishing                      |
| TCP Forwarding     | Generic TCP application proxying            |
| SSH Access Proxy   | SSH-aware ZTNA proxy                        |
| FortiAuthenticator | Authentication / SAML IdP integration       |

---

## 🎯 Final Mental Model

```text
          ┌───────────────┐
          │  FortiClient  │
          └───────┬───────┘
                  │
          Device + User + Posture
                  │
                  ▼
          ┌───────────────┐
          │ FortiClient   │
          │     EMS       │
          └───────┬───────┘
                  │
          Tags + Certificate
                  │
                  ▼
          ┌───────────────┐
          │   FortiGate   │
          │     ZTNA      │
          └───────┬───────┘
                  │
       ┌──────────┼───────────┐
       │          │           │
      SAML      ZTNA       Security
    Identity     Tags       Policy
       │          │           │
       └──────────┼───────────┘
                  ▼
            Access Proxy
                  │
                  ▼
        Internal Application
```

**ZTNA = Identity + Device Trust + Posture + Application + Policy**

That is the core concept to remember.
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

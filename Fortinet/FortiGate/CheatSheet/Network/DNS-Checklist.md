# 🔐 FortiGate DNS Checklist

> **FortiOS 7.2.x | DNS | DNS Proxy | DNS Server | DNS Cache | DDNS | DNS Troubleshooting**
>
> **SheynShield — Engineering Secure Networks**

[![FortiOS](https://img.shields.io/badge/FortiOS-7.2.x-red?style=flat-square)](https://www.fortinet.com/)
[![Fortinet](https://img.shields.io/badge/Vendor-Fortinet-red?style=flat-square)](https://www.fortinet.com/)
[![Topic](https://img.shields.io/badge/Topic-DNS-blue?style=flat-square)](https://www.fortinet.com/)

---

## 📋 Table of Contents

* [1. DNS Architecture](#1-dns-architecture)
* [2. System DNS Configuration](#2-system-dns-configuration)
* [3. DNS Cache](#3-dns-cache)
* [4. Local Domain](#4-local-domain)
* [5. FortiGate DNS Server](#5-fortigate-dns-server)
* [6. DNS Proxy](#6-dns-proxy)
* [7. DNS Proxy Workers](#7-dns-proxy-workers)
* [8. Dynamic DNS DDNS](#8-dynamic-dns-ddns)
* [9. DDNS Configuration](#9-ddns-configuration)
* [10. DNS vs FortiGuard DNS](#10-dns-vs-fortiguard-dns)
* [11. DNS Logging](#11-dns-logging)
* [12. DNS Troubleshooting](#12-dns-troubleshooting)
* [13. DDNS Troubleshooting](#13-ddns-troubleshooting)
* [14. DNS Proxy Troubleshooting](#14-dns-proxy-troubleshooting)
* [15. DNS Security Checklist](#15-dns-security-checklist)
* [16. NSE4 / NSE7 High-Value Notes](#16-nse4--nse7-high-value-notes)
* [17. Quick CLI Reference](#17-quick-cli-reference)
* [18. Production Validation](#18-production-validation)
* [19. DNS Troubleshooting Decision Tree](#19-dns-troubleshooting-decision-tree)
* [20. Final Checklist](#20-final-checklist)

---

# 1. DNS Architecture

Before troubleshooting DNS on FortiGate, identify **which DNS function is actually failing**.

```text
                         ┌──────────────────────┐
                         │       FortiGate      │
                         └──────────┬───────────┘
                                    │
                 ┌──────────────────┼──────────────────┐
                 │                  │                  │
                 ▼                  ▼                  ▼
          System DNS            DNS Proxy          DNS Server
                 │                  │                  │
                 ▼                  ▼                  ▼
          FortiGate itself      DNS Processing      DNS Service
                 │                  │                  │
                 └──────────────────┼──────────────────┘
                                    │
                                    ▼
                              DNS Infrastructure
                                    │
                         ┌──────────┴──────────┐
                         ▼                     ▼
                      Internal               Public
                       DNS                   DNS
```

### Architecture Checklist

* [ ] DNS requirement has been identified.
* [ ] FortiGate system DNS has been identified.
* [ ] DNS Proxy requirement has been identified.
* [ ] DNS Server requirement has been identified.
* [ ] Internal DNS servers are documented.
* [ ] Public DNS servers are documented.
* [ ] DNS traffic path is documented.
* [ ] DNS logging requirements are defined.
* [ ] DNS security requirements are defined.
* [ ] DNS failure behavior is understood.

> **NSE mental model:** Do not treat **System DNS**, **DNS Proxy**, and **DNS Server** as the same feature.

---

# 2. System DNS Configuration

FortiGate uses **System DNS** for DNS resolution required by the FortiGate itself.

### Basic Configuration

```bash
config system dns
    set primary 8.8.8.8
    set secondary 5.200.200.200

    set protocol cleartext
    set ssl-certificate fortinet_factory

    set ip6-primary ::
    set ip6-secondary ::

    set timeout 5
    set retry 2

    set dns-cache-limit 5000
    set dns-cache-ttl 1800
    set cache-notfound-responses disable

    set source-ip 0.0.0.0
    set interface-select-method auto
    set server-select-method least-rtt

    set alt-primary 0.0.0.0
    set alt-secondary 0.0.0.0

    set log all
end
```

### Configuration Checklist

* [ ] Primary DNS server configured.
* [ ] Secondary DNS server configured.
* [ ] DNS protocol reviewed.
* [ ] DNS certificate reviewed where applicable.
* [ ] IPv6 DNS settings reviewed.
* [ ] DNS timeout reviewed.
* [ ] DNS retry count reviewed.
* [ ] DNS cache size reviewed.
* [ ] DNS cache TTL reviewed.
* [ ] Negative-response caching behavior reviewed.
* [ ] DNS source IP reviewed.
* [ ] Interface selection method reviewed.
* [ ] DNS server selection method reviewed.
* [ ] DNS logging reviewed.

### Verify Configuration

```bash
show system dns
```

---

## Important DNS Parameters

| Parameter                  | Purpose                                |
| -------------------------- | -------------------------------------- |
| `primary`                  | Primary DNS server                     |
| `secondary`                | Secondary DNS server                   |
| `protocol`                 | DNS communication protocol             |
| `ssl-certificate`          | Certificate used where required        |
| `timeout`                  | DNS query timeout                      |
| `retry`                    | DNS retry count                        |
| `dns-cache-limit`          | Maximum DNS cache entries              |
| `dns-cache-ttl`            | DNS cache lifetime                     |
| `cache-notfound-responses` | Controls negative DNS response caching |
| `source-ip`                | Source IP used for DNS queries         |
| `interface-select-method`  | Determines interface selection         |
| `server-select-method`     | Determines DNS server selection        |
| `log`                      | DNS logging behavior                   |

---

# 3. DNS Cache

FortiGate can cache DNS responses locally.

### Example

```text
dns-cache-limit = 5000
dns-cache-ttl   = 1800
```

### DNS Cache Checklist

* [ ] DNS cache requirement has been identified.
* [ ] `dns-cache-limit` is appropriate for the appliance.
* [ ] `dns-cache-ttl` is appropriate for the environment.
* [ ] Negative-response caching behavior is understood.
* [ ] DNS cache behavior has been considered during troubleshooting.
* [ ] Cache-related behavior is documented.

### Mental Model

```text
DNS Query
    │
    ▼
FortiGate DNS Processing
    │
    ├── Cache Hit ───────► Return Cached Result
    │
    └── Cache Miss
             │
             ▼
        DNS Server
             │
             ▼
        DNS Response
             │
             ▼
        Local Cache
```

### Key Point

> DNS cache can affect troubleshooting because a DNS query may be answered locally rather than generating a new upstream query.

---

# 4. Local Domain

FortiGate can be configured with a local domain name.

Example:

```text
test.com
```

### Checklist

* [ ] Local domain requirement identified.
* [ ] Domain name documented.
* [ ] DNS design is consistent with the local domain.
* [ ] Internal DNS resolution requirements are documented.
* [ ] Local domain behavior has been tested.

### Example Concept

```text
FortiGate
    │
    ▼
Local Domain
    │
    ▼
test.com
```

---

# 5. FortiGate DNS Server

Do not confuse the FortiGate's **System DNS** with its **DNS Server** functionality.

```text
             FortiGate
                 │
        ┌────────┴────────┐
        │                 │
        ▼                 ▼
    System DNS        DNS Server
        │                 │
        ▼                 ▼
 FortiGate uses       Clients use
 DNS                  FortiGate DNS
```

### DNS Server Checklist

* [ ] DNS Server requirement identified.
* [ ] Listening interface identified.
* [ ] Client DNS traffic path documented.
* [ ] DNS records/zones requirements documented.
* [ ] Internal clients can reach the DNS service.
* [ ] Unauthorized clients cannot use the DNS service.
* [ ] DNS access policy is reviewed.
* [ ] Logging is enabled where required.

---

## Shadow vs Public Mode

### Shadow Mode

```text
Shadow DNS
    │
    ▼
Internal / Inside DNS Resolution
```

### Public Mode

```text
Public DNS
    │
    ▼
External DNS Service
```

### Validation Checklist

* [ ] Intended DNS serving mode is documented.
* [ ] Internal/external exposure is understood.
* [ ] DNS service is not unintentionally exposed to the Internet.
* [ ] Firewall policy protects the DNS service where required.

---

# 6. DNS Proxy

DNS Proxy provides DNS processing/proxy functionality on FortiGate.

### Conceptual Flow

```text
DNS Client
     │
     │ UDP/TCP 53
     ▼
FortiGate DNS Proxy
     │
     ▼
Upstream DNS Server
     │
     ▼
DNS Response
     │
     ▼
DNS Client
```

### DNS Proxy Checklist

* [ ] DNS Proxy requirement identified.
* [ ] DNS client traffic reaches FortiGate.
* [ ] Upstream DNS servers are reachable.
* [ ] DNS Proxy configuration is reviewed.
* [ ] DNS requests are processed correctly.
* [ ] DNS responses return to clients.
* [ ] DNS logging is reviewed where required.
* [ ] DNS Proxy troubleshooting has been tested.

### DNS Proxy Test

```bash
diagnose test application dnsproxy
```

---

# 7. DNS Proxy Workers

The number of DNS Proxy workers can be configured globally.

```bash
config system global
    set dnsproxy-worked-count 1
end
```

### Checklist

* [ ] DNS Proxy worker requirement has been evaluated.
* [ ] Appliance CPU/core resources have been considered.
* [ ] Worker configuration has been reviewed.
* [ ] Performance requirements have been considered.
* [ ] Changes have been validated after modification.

### Mental Model

```text
CPU / Core Resources
        │
        ▼
DNS Proxy Workers
        │
        ▼
DNS Query Processing
        │
        ▼
DNS Responses
```

> **Important:** Validate the exact CLI keyword available on the target FortiOS release before applying configuration.

---

# 8. Dynamic DNS DDNS

**Dynamic DNS (DDNS)** allows a hostname to track the changing IP address of a monitored FortiGate interface.

### DDNS Flow

```text
                  FortiGate
                     │
                     │ monitors
                     ▼
                  port1 IP
                     │
                 IP changes
                     │
                     ▼
               DDNS Provider
                     │
                     ▼
               DNS Record
                  Updated
```

### DDNS Checklist

* [ ] DDNS requirement identified.
* [ ] DDNS provider selected.
* [ ] DDNS account created.
* [ ] Username available.
* [ ] Password/credential available securely.
* [ ] Monitored interface identified.
* [ ] Interface has appropriate connectivity.
* [ ] FortiGate can reach the DDNS provider.
* [ ] DNS/FortiGuard connectivity prerequisites are satisfied.
* [ ] DDNS hostname resolves to the expected address.

### Platform Note

For certain platforms, including **FortiGate 1000 series and FortiGate-VM**, GUI support may differ; verify the target FortiOS/platform documentation and use CLI where required.

---

# 9. DDNS Configuration

### Example

```bash
config system ddns
    edit 1
        set ddns-server tzo.com
        set ddns-username <DDNS-USERNAME>
        set ddns-password <DDNS-PASSWORD>
        set monitor-interface port1
    next
end
```

### Parameter Checklist

| Parameter           | Validation                         |
| ------------------- | ---------------------------------- |
| `ddns-server`       | [ ] Provider is correct            |
| `ddns-username`     | [ ] Account is correct             |
| `ddns-password`     | [ ] Credential is correct          |
| `monitor-interface` | [ ] Correct interface is monitored |

> **⚠️ Security:** Never commit real DDNS credentials, API keys, passwords, or secrets to a public GitHub repository.

### Secure Example

```bash
set ddns-username <REDACTED>
set ddns-password <REDACTED>
```

---

# 10. DNS vs FortiGuard DNS

A common conceptual mistake is assuming the FortiGate DNS Server itself uses FortiGuard as its DNS server.

Keep the functions separate:

```text
             FortiGate
                 │
       ┌─────────┴─────────┐
       │                   │
       ▼                   ▼
   System DNS          DNS Server
       │                   │
       ▼                   ▼
FortiGate DNS          Client DNS
 Resolution            Service
```

### Checklist

* [ ] System DNS has been identified.
* [ ] DNS Server functionality has been identified.
* [ ] DNS Proxy functionality has been identified.
* [ ] FortiGuard services are not confused with ordinary DNS service.
* [ ] Upstream DNS infrastructure is documented.

> **NSE memory:** Always identify **who is asking the DNS question** and **who is providing the DNS answer**.

---

# 11. DNS Logging

DNS activity can be reviewed through FortiGate logging depending on configuration and FortiOS capabilities.

Typical location:

```text
Log & Report
    └── Security Events
        └── DNS Query
```

If FortiAnalyzer is deployed:

```text
FortiGate
    │
    ▼
DNS Events
    │
    ▼
FortiAnalyzer
    │
    ▼
DNS Analysis / Reporting
```

### Logging Checklist

* [ ] DNS logging requirement identified.
* [ ] DNS logging is enabled where required.
* [ ] DNS events are visible.
* [ ] Failed DNS queries can be investigated.
* [ ] Suspicious DNS activity can be investigated.
* [ ] FortiAnalyzer integration is configured where required.
* [ ] Log retention meets organizational requirements.

---

# 12. DNS Troubleshooting

When DNS fails, avoid immediately changing DNS servers.

Use a structured troubleshooting process.

## Step 1 — Identify the DNS Client

* [ ] Is the client FortiGate itself?
* [ ] Is the client an internal endpoint?
* [ ] Is the client a guest endpoint?
* [ ] Is the client using FortiGate as DNS?
* [ ] Is the client using an external DNS server?

---

## Step 2 — Verify Client Configuration

* [ ] Client has valid IP address.
* [ ] DHCP is working.
* [ ] Default gateway is correct.
* [ ] DNS server address is correct.
* [ ] DNS requests are leaving the client.
* [ ] DNS response is returning.

---

## Step 3 — Verify Connectivity

Check:

```text
Client
  │
  ▼
Gateway
  │
  ▼
FortiGate
  │
  ▼
DNS Server
```

Validate:

* [ ] Routing.
* [ ] Interface status.
* [ ] Firewall policy.
* [ ] DNS server reachability.
* [ ] UDP/53.
* [ ] TCP/53 where required.
* [ ] Return traffic.

---

## Step 4 — Verify FortiGate System DNS

```bash
show system dns
```

Check:

* [ ] Primary DNS.
* [ ] Secondary DNS.
* [ ] Timeout.
* [ ] Retry.
* [ ] Source IP.
* [ ] Interface selection.
* [ ] Server selection method.

---

## Step 5 — Check DNS Proxy

```bash
diagnose test application dnsproxy
```

Then use DNS Proxy debugging when necessary:

```bash
diagnose debug application dnsproxy -1 1
diagnose debug enable
```

After troubleshooting:

```bash
diagnose debug disable
```

---

# 13. DDNS Troubleshooting

## DDNS Debug

```bash
diagnose debug application ddnscd -1
diagnose debug enable
```

Disable debugging afterward:

```bash
diagnose debug disable
```

## DDNS Diagnostic Test

```bash
diagnose test application ddnscd 3
```

### DDNS Troubleshooting Checklist

* [ ] DDNS configuration is correct.
* [ ] DDNS provider is reachable.
* [ ] Monitored interface is correct.
* [ ] Interface has the expected IP.
* [ ] FortiGate has Internet connectivity.
* [ ] DDNS credentials are valid.
* [ ] DDNS hostname exists.
* [ ] DNS record updates after IP changes.
* [ ] `ddnscd` diagnostic output has been reviewed.
* [ ] Debugging has been disabled after testing.

### Troubleshooting Flow

```text
DDNS Failure
     │
     ▼
Check DDNS Configuration
     │
     ▼
Check Monitored Interface
     │
     ▼
Check Internet Connectivity
     │
     ▼
Check DDNS Provider
     │
     ▼
ddnscd Debug
     │
     ▼
diagnose test application ddnscd 3
```

---

# 14. DNS Proxy Troubleshooting

### DNS Proxy Debug

```bash
diagnose debug application dnsproxy -1 1
diagnose debug enable
```

### Diagnostic Test

```bash
diagnose test application dnsproxy
```

### Troubleshooting Checklist

* [ ] DNS Proxy is receiving requests.
* [ ] Upstream DNS server is reachable.
* [ ] DNS requests are forwarded.
* [ ] DNS responses are received.
* [ ] DNS responses are returned to clients.
* [ ] Firewall policy is not blocking DNS.
* [ ] Routing is correct.
* [ ] DNS server is responding.
* [ ] DNS Proxy debug output has been reviewed.
* [ ] Debugging has been disabled after testing.

### DNS Proxy Flow

```text
Client
  │
  │ DNS Query
  ▼
FortiGate DNS Proxy
  │
  │ Forward Query
  ▼
Upstream DNS
  │
  │ DNS Response
  ▼
FortiGate DNS Proxy
  │
  │ Response
  ▼
Client
```

---

# 15. DNS Security Checklist

## Network Security

* [ ] DNS servers are reachable only from authorized networks.
* [ ] Guest clients cannot access internal DNS unnecessarily.
* [ ] Management networks are protected.
* [ ] DNS service is not unintentionally exposed to the Internet.
* [ ] Firewall policies restrict unnecessary DNS traffic.
* [ ] Unauthorized DNS servers are investigated where required.

## Authentication & Administration

* [ ] DNS configuration changes require appropriate administrative access.
* [ ] Administrative credentials are protected.
* [ ] Secrets are never stored in public repositories.
* [ ] Configuration backups are protected.

## Monitoring

* [ ] DNS queries are logged where required.
* [ ] DNS failures are monitored.
* [ ] Suspicious DNS behavior can be investigated.
* [ ] Logging infrastructure is operational.
* [ ] FortiAnalyzer integration is validated where applicable.

## Availability

* [ ] Primary DNS server is reachable.
* [ ] Secondary DNS server is reachable.
* [ ] DNS timeout is appropriate.
* [ ] Retry behavior is understood.
* [ ] DNS failure scenarios have been tested.

---

# 16. NSE4 / NSE7 High-Value Notes

## 🧠 System DNS vs DNS Server

```text
System DNS
    │
    └── Used by FortiGate

DNS Server
    │
    └── Used by DNS Clients
```

---

## 🧠 DNS Proxy

```text
Client
  │
  ▼
DNS Proxy
  │
  ▼
Upstream DNS
```

The DNS Proxy is part of the **DNS request processing path**.

---

## 🧠 DDNS

```text
Interface IP
     │
     ▼
FortiGate
     │
     ▼
DDNS Provider
     │
     ▼
Hostname → Current IP
```

DDNS is useful when the public IP address can change.

---

## 🧠 Troubleshooting Separation

```text
DNS Problem
     │
     ├── FortiGate DNS?
     │       └── system dns
     │
     ├── DNS Proxy?
     │       └── dnsproxy
     │
     └── DDNS?
             └── ddnscd
```

---

## 🧠 Debug Lifecycle

Always follow:

```text
Enable Debug
     │
     ▼
Reproduce Problem
     │
     ▼
Collect Output
     │
     ▼
Analyze
     │
     ▼
Disable Debug
```

Never leave unnecessary debugging enabled in production.

---

# 17. Quick CLI Reference

| Task                        | Command                                    |
| --------------------------- | ------------------------------------------ |
| View System DNS             | `show system dns`                          |
| Configure System DNS        | `config system dns`                        |
| Configure DDNS              | `config system ddns`                       |
| DDNS Debug                  | `diagnose debug application ddnscd -1`     |
| DDNS Test                   | `diagnose test application ddnscd 3`       |
| DNS Proxy Test              | `diagnose test application dnsproxy`       |
| DNS Proxy Debug             | `diagnose debug application dnsproxy -1 1` |
| Enable Debug                | `diagnose debug enable`                    |
| Disable Debug               | `diagnose debug disable`                   |
| Configure DNS Proxy Workers | `config system global`                     |

---

## CLI Cheat Sheet

### System DNS

```bash
config system dns
    set primary <PRIMARY-DNS>
    set secondary <SECONDARY-DNS>
end
```

### DDNS

```bash
config system ddns
    edit 1
        set ddns-server <DDNS-PROVIDER>
        set ddns-username <USERNAME>
        set ddns-password <PASSWORD>
        set monitor-interface <INTERFACE>
    next
end
```

### DDNS Debug

```bash
diagnose debug application ddnscd -1
diagnose debug enable
```

### DDNS Test

```bash
diagnose test application ddnscd 3
```

### DNS Proxy Test

```bash
diagnose test application dnsproxy
```

### DNS Proxy Debug

```bash
diagnose debug application dnsproxy -1 1
diagnose debug enable
```

### Stop Debugging

```bash
diagnose debug disable
```

> **⚠️ Version note:** CLI syntax and available options can vary by FortiOS release, FortiGate model, and deployment mode. Validate the command tree on the target device before applying configuration.

---

# 18. Production Validation

## DNS Configuration

* [ ] Primary DNS configured.
* [ ] Secondary DNS configured.
* [ ] DNS protocol validated.
* [ ] DNS timeout reviewed.
* [ ] Retry behavior reviewed.
* [ ] DNS cache reviewed.
* [ ] Source IP reviewed.
* [ ] Interface selection reviewed.
* [ ] DNS server selection method reviewed.

## DNS Service

* [ ] DNS Server requirements documented.
* [ ] DNS Proxy requirements documented.
* [ ] DNS client path verified.
* [ ] Upstream DNS path verified.
* [ ] DNS queries resolve successfully.
* [ ] DNS failures are observable.

## DDNS

* [ ] DDNS provider configured.
* [ ] Credentials validated.
* [ ] Monitored interface validated.
* [ ] Public IP detected correctly.
* [ ] DDNS hostname resolves correctly.
* [ ] IP change behavior tested.

## Logging

* [ ] DNS logging reviewed.
* [ ] Authentication/administrative logging reviewed.
* [ ] FortiAnalyzer integration verified where required.
* [ ] DNS events can be searched and investigated.

---

# 19. DNS Troubleshooting Decision Tree

```text
                         DNS FAILURE
                              │
                              ▼
                    Who is the DNS client?
                              │
             ┌────────────────┼────────────────┐
             │                │                │
             ▼                ▼                ▼
          FortiGate         Client          DDNS
             │                │                │
             ▼                ▼                ▼
       System DNS       DHCP / DNS       ddnscd
             │                │                │
             ▼                ▼                ▼
       show system dns   Connectivity    DDNS Config
             │                │                │
             ▼                ▼                ▼
       DNS Reachability  Policy/Route   Provider
             │                │                │
             ▼                ▼                ▼
          Resolve?        DNS Proxy?      IP Update?
             │                │                │
        ┌────┴────┐      ┌────┴────┐      ┌────┴────┐
        ▼         ▼      ▼         ▼      ▼         ▼
       YES        NO    YES        NO    YES        NO
        │         │      │         │      │         │
        ▼         ▼      ▼         ▼      ▼         ▼
      Done      Debug   Debug    Fix Path Done     Debug
```

---

# 20. Final Checklist

## 🚀 FortiGate DNS Production Readiness

### System DNS

* [ ] Primary DNS configured.
* [ ] Secondary DNS configured.
* [ ] DNS protocol reviewed.
* [ ] Timeout configured appropriately.
* [ ] Retry configured appropriately.
* [ ] Cache settings reviewed.
* [ ] Source IP reviewed.
* [ ] Interface selection reviewed.
* [ ] Server selection method reviewed.

### DNS Server / Proxy

* [ ] DNS Server requirement validated.
* [ ] DNS Proxy requirement validated.
* [ ] DNS client path tested.
* [ ] Upstream DNS path tested.
* [ ] DNS Proxy diagnostics tested.
* [ ] DNS service exposure reviewed.

### DDNS

* [ ] DDNS provider configured.
* [ ] Credentials validated.
* [ ] Monitored interface correct.
* [ ] DDNS hostname resolves correctly.
* [ ] IP change behavior tested.
* [ ] DDNS debug tested where required.

### Troubleshooting

* [ ] Client configuration checked.
* [ ] DHCP checked.
* [ ] DNS server address checked.
* [ ] Routing checked.
* [ ] Firewall policy checked.
* [ ] UDP/53 checked.
* [ ] TCP/53 checked where required.
* [ ] DNS Proxy checked.
* [ ] System DNS checked.
* [ ] DDNS daemon checked.
* [ ] Debugging disabled after testing.

### Security

* [ ] DNS service is not unintentionally Internet-facing.
* [ ] Unauthorized DNS access is restricted.
* [ ] Internal DNS is protected.
* [ ] Guest DNS access is controlled.
* [ ] DNS logging is enabled where required.
* [ ] Credentials are not stored in GitHub.
* [ ] Least-privilege administration is enforced.

---

# 🎯 Final Mental Model

```text
                           DNS
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
          ▼                 ▼                 ▼
      System DNS        DNS Proxy         DNS Server
          │                 │                 │
          │                 │                 │
          ▼                 ▼                 ▼
     FortiGate          DNS Requests       Clients
     Resolution          Processing        DNS Service
          │                 │                 │
          └─────────────────┼─────────────────┘
                            │
                            ▼
                      DNS Infrastructure
                            │
                   ┌────────┴────────┐
                   ▼                 ▼
                Internal           Public
                   DNS               DNS


                         DDNS
                          │
                          ▼
                    FortiGate IP
                          │
                          ▼
                    DDNS Provider
                          │
                          ▼
                   Hostname Record
```

---

# 🔥 SheynShield Golden Rules

> **Rule #1 — System DNS and FortiGate DNS Server are different functions.**

> **Rule #2 — DNS Proxy belongs to the DNS request-processing path; identify it before troubleshooting.**

> **Rule #3 — DDNS maps a changing interface IP to a DNS hostname through a DDNS provider.**

> **Rule #4 — When DNS fails, first identify who is making the DNS request.**

> **Rule #5 — DNS troubleshooting must include connectivity, routing, firewall policy, DNS server reachability and response behavior.**

> **Rule #6 — Use `dnsproxy` diagnostics for DNS Proxy problems and `ddnscd` diagnostics for DDNS problems.**

> **Rule #7 — Always disable debugging after troubleshooting.**

> **Rule #8 — Never publish real DNS, DDNS, LDAP, RADIUS or other authentication credentials in a public repository.**

---

# 📚 Related SheynShield Topics

* [ ] FortiGate Firewall Policy
* [ ] FortiGate DNS Proxy
* [ ] FortiGate DNS Server
* [ ] FortiGate DHCP
* [ ] FortiGate Routing
* [ ] FortiGate SD-WAN
* [ ] FortiGate Guest Network
* [ ] FortiGate Captive Portal
* [ ] FortiGate FortiGuard
* [ ] FortiGate Logging
* [ ] FortiAnalyzer
* [ ] FortiGate Troubleshooting
* [ ] Network Security
* [ ] DNS Security
* [ ] NSE4
* [ ] NSE7

---

# 🔎 Topics

`FortiGate` `FortiOS` `Fortinet` `FortiGate DNS` `FortiOS DNS` `DNS Proxy` `FortiGate DNS Proxy` `FortiGate DNS Server` `FortiGate DDNS` `Dynamic DNS` `DNS Troubleshooting` `FortiGate Troubleshooting` `DNS Cache` `FortiGuard` `Network Security` `Firewall` `NSE4` `NSE7` `Fortinet NSE` `Cyber Security` `DNS Security`

---

# 🔗 SheynShield Resources

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

**SheynShield — Engineering Secure Networks**

> **Learn it. Verify it. Secure it.**

# FortiGate DNS — Part 2

> **Scope:** FortiOS DNS, DNS Proxy, Local Domain, DDNS & DNS Troubleshooting
> **Reference:** FortiOS 7.2.x

---

## 1. DNS Configuration

### Basic DNS Configuration

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

### Important DNS Options

| Option                     | Purpose                                             |
| -------------------------- | --------------------------------------------------- |
| `primary`                  | Primary DNS server                                  |
| `secondary`                | Secondary DNS server                                |
| `protocol`                 | DNS communication protocol                          |
| `ssl-certificate`          | Certificate used when required by the DNS protocol  |
| `timeout`                  | DNS query timeout                                   |
| `retry`                    | Number of DNS query retries                         |
| `dns-cache-limit`          | Maximum DNS cache entries                           |
| `dns-cache-ttl`            | DNS cache lifetime                                  |
| `cache-notfound-responses` | Controls caching/handling of negative DNS responses |
| `source-ip`                | Source IP used for DNS queries                      |
| `interface-select-method`  | Determines the interface used for DNS traffic       |
| `server-select-method`     | DNS server selection method                         |
| `log`                      | DNS logging configuration                           |

### DNS Cache

```text
dns-cache-limit 5000
dns-cache-ttl   1800
```

* `dns-cache-limit` → Maximum number of DNS cache entries.
* `dns-cache-ttl` → Cache lifetime in seconds.
* `cache-notfound-responses` → Controls handling of DNS queries where no cached result is found / negative responses.

---

## 2. Local Domain Name

FortiGate can be configured with a local domain name.

Example:

```text
test.com
```

This can be used as the local DNS domain depending on the DNS/server configuration.

### DNS Management

DNS-related events can be reviewed from:

```text
Log & Report
    └── Security Events
        └── DNS Query
```

> **Note:** If FortiAnalyzer is configured and receiving the relevant logs, DNS queries can be managed and analyzed there as well.

---

# 3. DNS Server — Shadow / Public Mode

FortiOS DNS Server supports different DNS serving behaviors.

### Shadow Mode

```text
Shadow
    ↓
Internal / Inside DNS usage
```

> **Concept:** Shadow DNS is intended for internal/inside DNS resolution scenarios.

### Public Mode

```text
Public DNS
    ↓
External / Public DNS usage
```

---

# 4. Dynamic DNS (DDNS)

Dynamic DNS allows a hostname to dynamically represent the IP address of a monitored FortiGate interface.

### DDNS Support Note

GUI support is not available for:

```text
FortiGate 1000 series
FortiGate-VM
```

For these platforms, use CLI configuration where supported.

---

## DDNS Configuration

> **Prerequisite:** Configure the required DNS/FortiGuard connectivity before configuring DDNS.

```bash
config system ddns
    edit 1
        set ddns-server tzo.com
        set ddns-username shayan
        set ddns-password 123
        set monitor-interface port1
    end
end
```

### Configuration Parameters

| Parameter           | Description                             |
| ------------------- | --------------------------------------- |
| `ddns-server`       | DDNS provider                           |
| `ddns-username`     | DDNS account username                   |
| `ddns-password`     | DDNS account password                   |
| `monitor-interface` | Interface whose IP address is monitored |

### DDNS Flow

```text
FortiGate
   │
   │ monitors
   ▼
port1 IP address
   │
   │ IP changes
   ▼
DDNS Provider
   │
   ▼
DNS record updated
```

---

# 5. DNS Server vs FortiGuard DNS

> **Important:** The FortiGate DNS Server itself does not use FortiGuard as its DNS server.

Separate the concepts:

```text
FortiGate System DNS
        │
        └── DNS resolution used by FortiGate

FortiGate DNS Server
        │
        └── Provides DNS service to clients
```

---

# 6. DNS Troubleshooting

## 6.1 DDNS Debug

Enable DDNS daemon debugging:

```bash
diagnose debug application ddnscd -1
diagnose debug enable
```

Disable debugging after troubleshooting:

```bash
diagnose debug disable
```

---

## 6.2 DDNS Diagnostic Test

```bash
diagnose test application ddnscd 3
```

Useful for testing the DDNS daemon.

---

## 6.3 DNS Proxy Diagnostic

```bash
diagnose test application dnsproxy
```

### DNS Proxy Debug

```bash
diagnose debug application dnsproxy -1 1
diagnose debug enable
```

> Use DNS Proxy debugging when investigating DNS resolution, proxy behavior, DNS requests, and related DNS processing.

---

# 7. DNS Proxy Worker Count

The number of DNS Proxy workers can be configured globally.

```bash
config system global
    set dnsproxy-worked-count 1
end
```

### `dnsproxy-worked-count`

Controls the number of DNS Proxy workers used by FortiOS.

```text
Available resources
        +
CPU/core resources
        ↓
DNS Proxy worker processing
```

> The appropriate worker count depends on the available system resources and CPU/core architecture.

---

# 8. DNS Troubleshooting Quick Reference

| Task                        | Command                                    |
| --------------------------- | ------------------------------------------ |
| View DNS configuration      | `show system dns`                          |
| Configure system DNS        | `config system dns`                        |
| Configure DDNS              | `config system ddns`                       |
| Debug DDNS                  | `diagnose debug application ddnscd -1`     |
| Test DDNS daemon            | `diagnose test application ddnscd 3`       |
| Test DNS Proxy              | `diagnose test application dnsproxy`       |
| Debug DNS Proxy             | `diagnose debug application dnsproxy -1 1` |
| Enable debugging            | `diagnose debug enable`                    |
| Disable debugging           | `diagnose debug disable`                   |
| Configure DNS Proxy workers | `config system global`                     |

---

# 9. DNS Troubleshooting Flow

```text
                DNS Problem
                     │
                     ▼
          Check System DNS Config
                     │
                     ▼
           Check DNS Server Reachability
                     │
                     ▼
             Check DNS Proxy
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
     DDNS Problem          DNS Resolution
          │                     │
          ▼                     ▼
    ddnscd debug           dnsproxy debug
          │                     │
          ▼                     ▼
 diagnose test             diagnose test
 application ddnscd 3      application dnsproxy
```

---

# 10. Key Takeaways

* Configure **primary/secondary DNS** under `config system dns`.
* DNS server configuration and FortiGate's own system DNS are separate concepts.
* Use **DNS cache settings** to control local DNS caching behavior.
* `server-select-method least-rtt` can be used to select the DNS server based on latency.
* Configure **local domain names** when required by the DNS design.
* DNS queries can be reviewed under:

```text
Log & Report → Security Events → DNS Query
```

* **DDNS** dynamically updates DNS records based on a monitored interface.
* Use `ddnscd` diagnostics for DDNS troubleshooting.
* Use `dnsproxy` diagnostics for DNS Proxy troubleshooting.
* `dnsproxy-worked-count` controls DNS Proxy worker processing.
* Always disable debugging after completing troubleshooting:

```bash
diagnose debug disable
```

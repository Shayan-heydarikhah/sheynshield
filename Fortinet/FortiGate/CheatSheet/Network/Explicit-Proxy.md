# FortiGate — Proxy, WAN Optimization & Web Acceleration 

> **Scope:** FortiOS 7.2.x training/reference notes
> **Format:** GitHub Flavored Markdown (GFM)
> **Focus:** Explicit Proxy, Transparent Proxy, FTP Proxy, Proxy Authentication, Upstream Proxy Chaining, WAN Optimization, Web Cache, CDN/VCache, NTLM/Kerberos, mTLS and troubleshooting

---

## Table of Contents

* [1. Proxy Fundamentals](#1-proxy-fundamentals)
* [2. Proxy vs Firewall vs VPN](#2-proxy-vs-firewall-vs-vpn)
* [3. Explicit Proxy](#3-explicit-proxy)
* [4. PAC Files](#4-pac-files)
* [5. Transparent Proxy](#5-transparent-proxy)
* [6. FTP Proxy](#6-ftp-proxy)
* [7. Proxy Policy](#7-proxy-policy)
* [8. Proxy Authentication](#8-proxy-authentication)
* [9. FSSO / LDAP / NTLM](#9-fsso--ldap--ntlm)
* [10. Kerberos & Keytab](#10-kerberos--keytab)
* [11. mTLS Access Proxy](#11-mtls-access-proxy)
* [12. Proxy Headers](#12-proxy-headers)
* [13. SaaS / Microsoft 365 Access Control](#13-saas--microsoft-365-access-control)
* [14. Upstream Proxy Forwarding](#14-upstream-proxy-forwarding)
* [15. Proxy Chaining](#15-proxy-chaining)
* [16. SD-WAN + Upstream Proxy Design](#16-sd-wan--upstream-proxy-design)
* [17. WAN Optimization](#17-wan-optimization)
* [18. WANOpt Architecture](#18-wanopt-architecture)
* [19. WANOpt VM Storage](#19-wanopt-vm-storage)
* [20. WANOpt Peer Configuration](#20-wanopt-peer-configuration)
* [21. WANOpt Profiles](#21-wanopt-profiles)
* [22. HTTPS Optimization](#22-https-optimization)
* [23. WANOpt Cache](#23-wanopt-cache)
* [24. Cache Parameters](#24-cache-parameters)
* [25. Cache Collaboration](#25-cache-collaboration)
* [26. CDN / VCache](#26-cdn--vcache)
* [27. HLS & MPEG-DASH](#27-hls--mpeg-dash)
* [28. WANOpt Statistics](#28-wanopt-statistics)
* [29. WANOpt Troubleshooting](#29-wanopt-troubleshooting)
* [30. Explicit Proxy + FortiGuard/FortiSandbox](#30-explicit-proxy--fortiguardfortisandbox)
* [31. Quick Verification](#31-quick-verification)

---

# 1. Proxy Fundamentals

## Why Use a Proxy?

Common scenarios:

* Access to destinations through an intermediary.
* Caching to reduce resource/bandwidth consumption.
* Restricting access from internal networks.
* Application-layer visibility and control.
* Security inspection of application protocols.
* Indirect access toward external services.

A proxy firewall provides a centralized point for:

* Application protocol inspection.
* Deep packet inspection.
* Attack detection.
* Error detection.
* Validity checks.
* Application-layer access control.

### Trade-off

| Characteristic   | Proxy              |
| ---------------- | ------------------ |
| Processing layer | Layer 7            |
| Resource usage   | Higher             |
| Visibility       | Application-aware  |
| Security control | High               |
| Performance      | Potentially slower |
| Caching          | Supported          |
| Authentication   | Supported          |

---

# 2. Proxy vs Firewall vs VPN

| Feature           | Proxy                                      | Firewall                | VPN                |
| ----------------- | ------------------------------------------ | ----------------------- | ------------------ |
| Primary role      | Intermediary                               | Traffic protection      | Secure tunnel      |
| Main visibility   | Application                                | Network/Application     | Encrypted tunnel   |
| Encryption        | Not inherently required                    | Not inherently required | Core capability    |
| Resource usage    | Lightweight → high depending on inspection | Variable                | Resource intensive |
| Identity hiding   | Yes                                        | No                      | Can provide        |
| Traffic filtering | Yes                                        | Yes                     | Limited by itself  |
| Caching           | Yes                                        | Usually no              | No                 |
| Typical layer     | L7                                         | L3-L7                   | L3/L4              |

### CIA in VPN

VPN primarily provides:

* **Confidentiality**
* **Integrity**
* **Availability / accessibility**

Encryption such as IPsec/AES is a major component.

---

# 3. Explicit Proxy

Explicit proxy requires the client to know that a proxy exists.

Example:

```text
Client
   |
   | HTTP :8080
   v
FortiGate Explicit Proxy
   |
   v
Internet
```

### Client Configuration

```text
Proxy:
    IP: 192.168.101.1
    Port: 8080
```

### FortiGate

Enable:

```text
System
  > Feature Visibility
    > Explicit Proxy
```

Then:

```text
Network
  > Explicit Proxy
```

Typical web proxy settings:

```text
Listen Interface: port3
HTTP Port:        8080
```

### Proxy FQDN

```text
default.fqdn
```

Represents the FortiGate address used by the proxy.

### Default Firewall Action

Options:

```text
deny
accept
```

Recommended design in the notes:

```text
deny
```

---

# 4. PAC Files

PAC = **Proxy Auto-Configuration**

PAC files allow clients to dynamically decide whether traffic should:

* Go directly.
* Use the FortiGate proxy.
* Use a primary proxy.
* Fall back to another proxy.

### Basic PAC

```javascript
function FindProxyForURL(url, host) {

    if (
        isPlainHostName(host) ||
        dnsDomainIs(host, ".local") ||
        isInNet(host, "127.0.0.0", "255.0.0.0") ||
        isInNet(host, "10.0.0.0", "255.0.0.0") ||
        isInNet(host, "172.16.0.0", "255.240.0.0") ||
        isInNet(host, "192.168.0.0", "255.255.0.0")
    ) {
        return "DIRECT";
    }

    return "PROXY <FORTIGATE-FQDN>:8080";
}
```

### Multiple Exceptions

```javascript
function FindProxyForURL(url, host) {

    if (
        isInNet(host, "192.168.0.0", "255.255.0.0") ||
        dnsDomainIs(host, ".internal.company.com")
    ) {
        return "DIRECT";
    }

    if (
        shExpMatch(host, "*.youtube.com") ||
        shExpMatch(host, "*.netflix.com")
    ) {
        return "DIRECT";
    }

    return "PROXY proxy.company.com:8080";
}
```

### Proxy Failover

```javascript
function FindProxyForURL(url, host) {
    return "PROXY primary-fgt:8080; PROXY backup-fgt:8080; DIRECT";
}
```

> **Note:** Consider case sensitivity when designing matching rules.

### `isPlainHostName()`

```text
faz.test.com
```

A hostname considered local/simple by the PAC logic can be processed as:

```text
DIRECT
```

---

# 5. Transparent Proxy

Transparent proxy does **not require client-side proxy configuration**.

```text
Client
   |
   | Normal HTTP
   v
FortiGate
   |
   | Redirect
   v
Transparent Proxy
   |
   v
Internet
```

### Key Difference

| Feature                    | Explicit        | Transparent  |
| -------------------------- | --------------- | ------------ |
| Client proxy configuration | Required        | Not required |
| PAC                        | Optional/useful | Not required |
| Client knows proxy         | Yes             | No           |
| Proxy Policy               | Yes             | Yes          |
| HTTP redirect              | No              | Required     |

### Firewall Policy

```bash
config firewall policy
    edit 1
        set inspection-mode proxy
        set srcintf port3
        set dstintf port1
        set action accept
        set srcaddr all
        set dstaddr all
        set schedule always
        set service ALL
        set logtraffic all
        set nat enable
        set http-policy-redirect enable
    next
end
```

### Important

```text
http-policy-redirect
```

must be used for transparent web proxy forwarding.

The proxy policy direction should match the firewall policy direction.

---

# 6. FTP Proxy

FTP can be handled separately from HTTP proxy.

### Explicit FTP Proxy

```bash
config ftp-proxy explicit
    set status enable
    set incoming-port 21
    set incoming-ip 0.0.0.0
    set sec-default-action deny
    set ssl disable
end
```

### Interface

```bash
config system interface
    edit port3
        set ip 192.168.101.1 255.255.255.0
        set allowaccess ping https http
        set type physical
        set explicit-ftp-proxy enable
    next
end
```

### FTP Proxy Policy

```bash
config firewall proxy-policy
    edit 1
        set proxy ftp
        set dstintf dmz
        set srcaddr all
        set dstaddr all
        set action accept
        set schedule always
        set logtraffic all
    next
end
```

### Client Test

```text
ftp 192.168.101.1
anonymous@192.168.20.200
guest@
```

---

# 7. Proxy Policy

Proxy policies provide application-aware controls beyond the normal firewall policy.

Example:

```bash
config firewall proxy-policy
    edit 1
        set proxy explicit-web
        set dstintf port1
        set srcaddr all
        set dstaddr all
        set service webproxy
        set action accept
        set schedule always
    next
end
```

### Proxy Address Matching

Proxy policies can use:

* Host regex
* URL pattern
* URL category
* HTTP method
* User-Agent
* HTTP headers
* Advanced source matching
* Advanced destination matching

### Design Principle

```text
Main Firewall Policy
        |
        +---- Network access
        |
        +---- NAT
        |
        +---- Security controls

Proxy Policy
        |
        +---- Application access
        |
        +---- URL matching
        |
        +---- Proxy authentication
        |
        +---- Web cache
        |
        +---- Upstream proxy
```

---

# 8. Proxy Authentication

## Authentication Flow

```text
Client
   |
   | HTTP request
   v
FortiGate
   |
   | Authentication Rule
   v
Authentication Schema
   |
   +---- LDAP
   +---- NTLM
   +---- FSSO
   +---- Certificate / mTLS
   |
   v
Proxy Policy
   |
   v
Internet
```

### Authentication Rule

Example:

```text
Incoming Interface: LAN
Source:             all
Protocol:           HTTP
Authentication:     auth-sch
```

---

# 9. FSSO / LDAP / NTLM

## LDAP-based Authentication

Typical workflow:

```text
1. Create LDAP server
2. Create users/groups
3. Create authentication schema
4. Create authentication rule
5. Reference users/groups in proxy policy
```

### NTLM Example

```text
Authentication Schema
    Method: NTLM

Domain Controller:
    IP: 192.168.20.200
    Port: 445
    Domain: test.com
    LDAP Server: ldap-winsrv-2016
```

### Domain Controller

```bash
config user domain-controller
    edit dc-1
        set ad-mode ds
        set hostname <HOSTNAME>
        set username <USERNAME>
        set password <PASSWORD>
    next
end
```

### Authentication Rule

```text
Source:             all
Incoming Interface: LAN
Protocol:           HTTP
Authentication:     auth-sch
```

### Test

Configure the client:

```text
Proxy: 192.168.101.1
Port:  8080
```

Verify:

```bash
diagnose wad user list
diagnose wad webcahe list 10
```

---

# 10. Kerberos & Keytab

### LDAP

```bash
config user ldap
    edit ldap-kerb
        set server 192.168.20.200
        set cnid cn
        set dn DC=test,DC=com
        set type regular
        set username <USERNAME>
        set password <PASSWORD>
    next

    edit ldap-two
        set server 192.168.20.201
        set cnid cn
        set dn OU=publishusers,DC=test1,DC=com
        set type regular
        set username <USERNAME>
        set password <PASSWORD>
    next
end
```

### Kerberos Keytab

```bash
config user krb-keytab
    edit krb-keytab
        set pac-data disable
        set principal http/fgt.example.local@example.local
        set ldap-server ldap-kerberos ldap-two
    next
end
```

### Domain Controller

```bash
config user domain-controller
    edit dc1
        set ip-address 192.168.20.200
        set ldap-server ldap-two ldap-kerberos
    next
end
```

### Web Proxy Global

```bash
config web-proxy global
    set ldap-user-cache disable
    set strict-web-check disable
    set proxy-fqdn default.fqdn
    set webproxy-profile default
    set learn-client-ip enable
    set learn-client-ip-from-header x-forwarded-for
    set learn-client-ip-srcaddr all
end
```

### Client-IP Header Sources

| Header            | Typical use                |
| ----------------- | -------------------------- |
| `X-Real-IP`       | NGINX/reverse proxy        |
| `X-Forwarded-For` | Client behind proxy        |
| `True-Client-IP`  | Client IP behind CDN/proxy |

---

# 11. mTLS Access Proxy

FortiGate can use client certificates for authentication.

```text
Client Certificate
       |
       v
FortiGate Access Proxy
       |
       | Certificate validation
       v
Proxy Policy
       |
       v
Backend Server
```

### VIP

```bash
config firewall vip
    edit "mTLS"
        set type access-proxy
        set extip 192.168.101.200
        set extintf port3
        set server-type https
        set extport 443
        set ssl-certificate Fortinet-CA-SSL
    next
end
```

### Access Proxy

```bash
config firewall access-proxy
    edit mTLS-access-proxy
        set vip mTLS
        set client-cert enable

        config api-gateway
            edit 1
                config realservers
                    edit 1
                        set ip 192.168.20.200
                    next
                end
            next
        end
    next
end
```

### Proxy Policy

```bash
config firewall proxy-policy
    edit 1
        set proxy access-proxy
        set access-proxy mTLS-access-proxy
        set srcintf port3
        set srcaddr all
        set dstaddr all
        set action accept
        set schedule always
        set users single-certificate
        set webproxy-profile mtls
        set utm-status enable
        set ssl-ssh-profile deep-inspection-clone
        set av-profile av
    next
end
```

### Forward Client Certificate

```bash
config web-proxy profile
    edit mtls
        set header-x-forwarded-client-cert add
    next
end
```

---

# 12. Proxy Headers

Dynamic variables can be inserted into HTTP headers.

Common variables:

```text
$client-ip
$user
$domain
$local_grp
$remote_grp
$proxy_name
```

### Example

```bash
config web-proxy profile
    edit 1
        set header-client-ip pass
        set log-header-change enable

        config headers
            edit 1
                set name client-ip
                set content $client-ip
            next

            edit 2
                set name Proxy-Name
                set content $proxy_name
            next

            edit 3
                set name user
                set content $user
            next

            edit 4
                set name domain
                set content $domain
            next

            edit 5
                set name local_grp
                set content $local_grp
            next

            edit 6
                set name remote_grp
                set content $remote_grp
            next
        end
    next
end
```

Attach it:

```bash
config firewall proxy-policy
    edit 1
        set webproxy-profile 1
    next
end
```

### Header Actions

| Action   | Meaning                 |
| -------- | ----------------------- |
| `pass`   | Forward existing header |
| `add`    | Add header              |
| `remove` | Remove/clear header     |

---

# 13. SaaS / Microsoft 365 Access Control

Headers can restrict SaaS tenant access.

### Microsoft 365

Relevant destinations:

```text
login.microsoftonline.com
login.microsoft.com
login.windows.net
login.live.com
```

Headers:

```text
Restrict-Access-To-Tenants
Restrict-Access-Context
Sec-Restrict-Tenant-Access-Policy
```

### Google

```text
X-GoogAppsAllowed-Domains
```

### Dropbox

```text
X-Dropbox-allowedTeam-Ids
```

### FQDN Address

```bash
config firewall address
    edit login.live.com
        set type fqdn
        set fqdn login.live.com
    next
end
```

### URL Filter

```bash
config webfilter urlfilter
    edit 1
        config entries
            edit 1
                set url login.microsoftonline.com
                set action allow
            next

            edit 2
                set url login.microsoft.com
                set action allow
            next

            edit 3
                set url login.windows.net
                set action allow
            next

            edit 4
                set url login.live.com
                set action allow
            next
        end
    next
end
```

### Web Filter Profile

```bash
config webfilter profile
    edit 1
        set feature-set proxy

        config web
            set urlfilter-table 1
        end
    next
end
```

### Header Injection

```bash
config web-proxy profile
    edit 1

        config headers
            edit 1
                set name Restrict-Access-To-Tenants
                set dstaddr login.microsoft.com login.microsoftonline.com login.windows.net
                set action add-to-request
                set base64-encoding disable
                set add-option new
                set protocol https http
                set content <COMPANY_DOMAIN>
            next

            edit 2
                set name Restrict-Access-Context
                set dstaddr login.microsoftonline.com login.microsoft.com login.windows.net
                set action add-to-request
                set base64-encoding disable
                set add-option new
                set protocol https http
                set content <DIRECTORY_ID>
            next

            edit 3
                set name sec-Restrict-Tenant-Access-Policy
                set dstaddr login.live.com
                set action add-to-request
                set base64-encoding disable
                set add-option new
                set protocol https http
                set content restrict-msa
            next
        end
    next
end
```

---

# 14. Upstream Proxy Forwarding

FortiGate can forward proxy traffic to upstream proxy servers.

```text
Client
   |
   v
FortiGate
   |
   +---- Upstream Proxy 1
   |
   +---- Upstream Proxy 2
   |
   +---- Upstream Proxy 3
```

### Forward Servers

```bash
config web-proxy forward-server

    edit server1
        set ip 172.20.120.12
        set port 8080
    next

    edit server2
        set ip 172.20.120.13
        set port 8000
    next

    edit server3
        set ip 172.20.120.14
        set port 8090
    next

end
```

---

# 15. Proxy Chaining

Proxy chaining:

```text
Client
   |
   v
FortiGate Proxy
   |
   v
Upstream Proxy
   |
   v
Internet
```

### Health Monitoring

The notes recommend monitoring an HTTP destination and expecting:

```text
HTTP 200
```

### Forward Server Group

```bash
config web-proxy forward-server-group
    edit prx-fwd-group

        set affinity enable
        set ldb-method weighted
        set group-down-option block

        config server-list
            edit server1
                set weight 10
            next

            edit server2
                set weight 40
            next

            edit server3
                set weight 10
            next
        end

    next
end
```

### Load-Balancing Concepts

| Setting                   | Concept                              |
| ------------------------- | ------------------------------------ |
| `affinity`                | Preserve client/server relationship  |
| `weighted`                | Distribute according to weight       |
| `group-down-option block` | Block when all upstream servers fail |
| Lower weight              | Lower relative preference            |
| Higher weight             | Higher relative share                |

### Proxy Policy

```bash
config firewall proxy-policy
    edit 1
        set proxy explicit-web
        set dstintf port1
        set srcaddr all
        set dstaddr all
        set service webproxy
        set action accept
        set schedule always
        set webproxy-forward-server prx-fwd-group
    next
end
```

> A forwarding server group is treated as a group; individual members are not separately selected in the proxy policy.

---

# 16. SD-WAN + Upstream Proxy Design

Example topology:

```text
                       +----------------+
                       |    Internet    |
                       +--------+-------+
                                |
                       +--------+--------+
                       |   MikroTik Edge |
                       +---+---------+---+
                           |         |
                     +-----+         +-----+
                     |                     |
                +----+----+           +----+----+
                | Proxy 1 |           | Proxy 2 |
                | :9696   |           | :9797   |
                +----+----+           +----+----+
                     |                     |
                     +----------+----------+
                                |
                         +------+------+
                         |  FortiGate  |
                         | SD-WAN      |
                         +-------------+
```

### SD-WAN Rules

Example:

```text
*.ir  -> Proxy 1
*.com -> Proxy 2
```

### SLA

```text
8.8.8.8
1.1.1.1
```

### Static Route

```text
0.0.0.0/0 -> SD-WAN
```

### Important

If NAT is disabled between components:

```text
Return routes are required.
```

---

# 17. WAN Optimization

WAN Optimization aims to:

* Reduce effective latency.
* Reduce bandwidth consumption.
* Improve TCP efficiency.
* Cache repeated content.
* Compress data.
* Optimize application protocols.
* Improve branch/cloud application performance.

### Enterprise Use Cases

* Branch offices.
* SaaS applications.
* Office 365.
* Salesforce.
* Cloud applications.
* Large repeated file transfers.
* HTTP/HTTPS traffic.

---

# 18. WANOpt Architecture

```text
Branch A
   |
   v
FortiGate WANOpt
   |
   | Optimized Tunnel
   |
   v
FortiGate WANOpt
   |
   v
Branch B / Internet
```

WANOpt can use:

* Memory
* Disk
* Byte caching
* Compression
* TCP optimization
* HTTP optimization
* SSL optimization
* Protocol-specific optimization

### Acceleration

```text
Client
   |
   v
Classification
   |
   v
TCP Normalization
   |
   v
Optimization Engine
   |
   v
SSL Proxy Layer
   |
   v
Destination
```

---

# 19. WANOpt VM Storage

## PNETLab / KVM

FortiGate VM example:

```bash
cd /opt/unetlab/tmp
```

Navigate to VM directory:

```bash
cd /opt/unetlab/addons/qemu/fortinet-fgt-v7.2.0-build1157
```

> Backup the VM files before modifying the image/template.

Create disks:

```bash
qemu-img create -f qcow2 virtiob.qcow2 10g
qemu-img create -f qcow2 virtioc.qcow2 20g
```

Verify:

```bash
qemu-img info virtiob.qcow2
qemu-img info virtioc.qcow2
```

Edit template:

```bash
nano /opt/unetlab/html/templates/fortinet-fgt-v7.2.0-build1157.yml
```

Add disks:

```yaml
devices:
  - type: disk
    disk: "virtioa.qcow2"

  - type: disk
    disk: "virtiob.qcow2"

  - type: disk
    disk: "virtioc.qcow2"
```

Check permissions:

```text
root level
0755
```

Restart the VM.

### FortiGate

```bash
diagnose hardware deviceinfo disk
execute disk list
execute disk format 32
```

---

# 20. WANOpt Peer Configuration

### Storage

```bash
config system storage
    edit hdd2
        set status enable
        set usage wanopt
        set wanopt-mode mix
    next
end
```

`mix`:

```text
RAM + Disk
```

### Peer

```bash
config wanopt peer
    edit fgt-2
        set ip 192.168.254.251
    next
end
```

---

# 21. WANOpt Profiles

Enable WANOpt:

```text
System
  > Feature Visibility
    > WANOpt
```

### Profile

```bash
config wanopt profile
    edit wanopt-test-prof

        config http
            set status enable
            set ssl enable
        end

    next
end
```

### Firewall Policy

```bash
config firewall policy
    edit 1
        set srcintf port3
        set dstintf port1
        set action accept
        set srcaddr all
        set dstaddr all
        set schedule always
        set service ALL
        set utm-status enable
        set inspection-mode proxy
        set profile-protocol-options protocol
        set ssl-ssh-profile ssl-test
        set wanopt enable
        set wanopt-profile wanopt-test-prof
        set nat enable
    next
end
```

### Second FortiGate

```bash
config firewall policy
    edit 1
        set srcintf port3
        set dstintf port1
        set action accept
        set srcaddr all
        set dstaddr all
        set schedule always
        set service ALL
        set utm-status enable
        set inspection-mode proxy
        set profile-protocol-options protocol
        set ssl-ssh-profile ssl-test
        set wanopt enable
        set wanopt-detection passive
        set nat enable
    next
end
```

### WANOpt Proxy Policy

```bash
config firewall proxy-policy
    edit 1
        set proxy wanopt
        set dstintf 1
        set srcaddr all
        set dstaddr all
        set service ALL
        set action accept
        set schedule always
        set utm-status enable
        set profile-protocol-options protocol
        set ssl-ssh-profile ssl-test
    next
end
```

---

# 22. HTTPS Optimization

## WANOpt vs SSL Inspection

| Feature                        | WAN Optimization | SSL Inspection |
| ------------------------------ | ---------------- | -------------- |
| Certificate                    | Re-signed        | Re-signed      |
| Full decryption                | No               | Yes            |
| Application payload visibility | Limited          | Full           |
| Header visibility              | Yes              | Yes            |
| Security inspection            | Limited          | Full           |
| Performance optimization       | Yes              | Depends        |

### WANOpt Certificate Swap

FortiGate can:

```text
Client
  |
  | TLS handshake
  v
FortiGate
  |
  | Certificate
  v
Client
```

while maintaining a separate TLS session toward the server.

### FortiGate Can See

* TCP/IP headers
* Source/destination
* Ports
* SNI
* Certificate information

### FortiGate Cannot See Without Decryption

* HTTP GET/POST
* URL path after domain
* Form data
* API payload

### HTTPS Acceleration

The notes identify:

* TLS session resumption
* TLS session tickets
* Connection reuse
* Header compression
* Byte caching
* Limited 0-RTT support

---

# 23. WANOpt Cache

### Main Settings

```text
WANOpt & Cache
    |
    +-- Always Revalidate
    +-- Max Cache Object Size
    +-- Negative Response Duration
    +-- Fresh Factor
    +-- Max TTL
    +-- Min TTL
    +-- Default TTL
    +-- Proxy FQDN
    +-- Max HTTP Request Length
    +-- Max HTTP Message Length
```

---

# 24. Cache Parameters

## Always Revalidate

### Enabled

FortiGate revalidates cached content against the origin.

```text
Cached Object
     |
     v
Origin Validation
     |
     +---- Changed -> HTTP 200
     |
     +---- Unchanged -> HTTP 304
```

### Disabled

Cached content can be served until TTL expiration.

---

## Maximum Cache Object Size

Controls the maximum object size that can be cached.

```text
Large value
    |
    +-- Large files
    +-- Videos
    +-- ISOs
    |
    +-- Higher storage consumption
```

---

## Negative Response Duration

Controls caching of error responses.

Examples:

```text
404
503
```

```text
0 -> Do not cache errors
>0 -> Cache errors for specified duration
```

---

## Fresh Factor

Controls early expiration.

Example:

```text
Origin TTL = 100 minutes
Fresh Factor = 80%

Effective TTL = 80 minutes
```

```bash
config wanopt webcache
    set fresh-factor <VALUE>
end
```

---

## TTL

| Parameter   |  Example | Purpose                       |
| ----------- | -------: | ----------------------------- |
| Max TTL     | 7200 min | Maximum cache lifetime        |
| Min TTL     |    5 min | Minimum cache lifetime        |
| Default TTL | 1440 min | Used when origin gives no TTL |

---

# 25. Cache Collaboration

WANOpt peers can share cached content.

```text
                +-----------+
                | WANOpt-1  |
                | Cache     |
                +-----+-----+
                      |
                Collaboration
                      |
                +-----+-----+
                | WANOpt-2  |
                | Cache     |
                +-----------+
```

### Configuration

```bash
config wanopt cache-service
    set prefer-scenario balance
end
```

### Scenarios

| Scenario   | Concept                     |
| ---------- | --------------------------- |
| `balance`  | Performance/storage balance |
| `speed`    | Performance priority        |
| `accuracy` | Content accuracy priority   |

### Collaboration

```text
Disabled by default
```

Enable collaboration when cache sharing is required.

### Device ID

```text
device-id
```

Unique identifier for the WANOpt device.

### Acceptable Connections

```text
any
peers
local
```

### Peer

```bash
config wanopt cache-service

    config dst-peer
        edit fgt2
            set device-id fgt-2
            set priority 1
        next
    end

end
```

Lower priority value:

```text
Higher priority
```

---

# 26. CDN / VCache

WANOpt can define content-delivery rules for streaming and update content.

### Rule Structure

```bash
config wanopt content-delivery-network-rule
    edit <RULE>
        set status enable
        set category vcache
        ...
    next
end
```

### Match Targets

Possible targets include:

```text
path
parameter
referrer
youtube-map
youtube-id
youku-id
```

### Match Modes

```text
all
any
```

| Mode  | Meaning                            |
| ----- | ---------------------------------- |
| `all` | All matching conditions must match |
| `any` | At least one condition can match   |

---

# 27. HLS & MPEG-DASH

## HLS

HLS = HTTP Live Streaming

Common components:

```text
.m3u8
.ts
.m4a
```

### HLS

```text
.m3u8
   |
   +---- Playlist / Manifest
   |
   +---- Media Segments
```

---

## MPEG-DASH

DASH:

> Dynamic Adaptive Streaming over HTTP

DASH uses:

```text
.mpd
.m4s
.mp4
```

### DASH

```text
MPD Manifest
     |
     +---- Representation
     |
     +---- Segment
     |
     +---- Quality level
```

DASH allows adaptive quality switching during playback.

---

# 28. WANOpt Statistics

### Tunnel Statistics

```bash
diagnose wad stats worker.tunnel
```

Useful values:

```text
comp.n_in_raw_bytes
comp.n_in_comp_bytes
comp.n_out_raw_bytes
comp.n_out_comp_bytes
```

### Interpretation

```text
n_in_raw_bytes
    Original data before compression

n_in_comp_bytes
    Data after compression

n_out_raw_bytes
    Outbound raw data

n_out_comp_bytes
    Outbound compressed data
```

### HTTP Worker

```bash
diagnose wad stats worker.protos.http
```

Important counters:

```text
wan.bytes_in
wan.bytes_out
lan.bytes_in
lan.bytes_out
tunnel.bytes_in
tunnel.bytes_out
```

### Flow

```text
Client
   |
   | Raw request
   v
FortiGate
   |
   v
Origin Server
   |
   | Raw response
   v
FortiGate
   |
   +---- Compress
   +---- Cache
   |
   v
Client
```

Cache hit:

```text
Client
   |
   v
FortiGate
   |
   | Cache Hit
   v
Client
```

---

# 29. WANOpt Troubleshooting

### Tunnel

```bash
diagnose wad stats worker.tunnel
```

### HTTP

```bash
diagnose wad stats worker.protos.http
```

### Event Logs

```text
Log & Report
  > System Events
    > WANOpt
```

### Test

```bash
curl -k https://<TEST-DOMAIN>/<FILE>
```

### Useful Interpretation

```text
wan.bytes_in != 0
```

May indicate unoptimized HTTP traffic according to the notes.

```text
tunnel.bytes_in
tunnel.bytes_out
```

Indicate traffic exchanged through WANOpt peers.

---

# 30. Explicit Proxy + FortiGuard/FortiSandbox

When FortiGate itself reaches FortiGuard/FortiSandbox through an upstream proxy, configure the proxy.

```bash
config system fortiguard
    set proxy-server-ip 172.16.200.44
    set proxy-server-port 3128
    set proxy-username <USERNAME>
    set proxy-password <PASSWORD>
end
```

### Debug

```bash
diagnose debug application forticldd -1
diagnose debug enable
```

---

# 31. Agentless NTLM + Web Proxy

Basic design:

```text
Client
   |
   | Explicit Proxy :8080
   v
FortiGate
   |
   | NTLM
   v
Domain Controller / LDAP
   |
   v
Proxy Policy
   |
   v
Internet
```

### Basic Setup

```text
1. Check default gateway
2. Configure DNS
3. Add LDAP
4. Create groups/users
5. Configure explicit proxy
6. Configure authentication schema
7. Configure authentication rule
8. Configure proxy policy
9. Test client
```

### Authentication Schema

```text
Method: NTLM

DC:
    <DC-IP>

Port:
    445

Domain:
    <DOMAIN>

LDAP:
    <LDAP-SERVER>
```

### Verification

```bash
diagnose wad user list
diagnose wad webcahe list 10
```

---

# 32. HTTPS Authentication / Captive Portal

For HTTPS authentication:

```text
Policy & Objects
    > Authentication Rule
        > Form Based Authentication
```

Authentication settings:

```text
HTTP Redirect: enabled
HTTP + HTTPS: enabled
Certificate: configured
```

### Captive Portal

Example:

```text
FQDN:
login.fortitest.com
```

### CLI

```bash
config authentication setting
    set captive-portal-ssl-port 7831
end
```

### Interface

```bash
config system interface
    edit port3
        set proxy-captive-portal enable
    next
end
```

### DNS Database

```text
DNS Server
    |
    +-- Recursive on port3
    |
    +-- DNS Database
          |
          +-- Shadow
          +-- Primary
          |
          +-- Zone: fortitest.com
                |
                +-- login -> 192.168.101.100
```

---

# 33. Session-Based Basic Authentication

If session-based basic authentication is used:

```text
web-auth-cookie
```

should be considered/enabled.

Check:

```bash
config firewall proxy-policy
    get | grep session
end
```

---

# 34. Proxy Authentication — Common Design

```text
                    +----------------+
                    | Authentication |
                    |     Schema     |
                    +--------+-------+
                             |
              +--------------+--------------+
              |              |              |
             LDAP           FSSO           mTLS
              |              |              |
              +--------------+--------------+
                             |
                             v
                    +--------+--------+
                    | Proxy Policy    |
                    +--------+--------+
                             |
                             v
                         Internet
```

---

# 35. Proxy Policy vs Firewall Policy

### Firewall Policy

Controls:

* Source interface
* Destination interface
* IP addresses
* Services
* NAT
* Security profiles
* Routing-related forwarding

### Proxy Policy

Controls:

* Proxy behavior
* URL
* Host
* HTTP methods
* Proxy authentication
* Proxy users/groups
* Web cache
* Upstream forwarding
* Proxy-specific profiles

### Important

If a destination is not covered by a proxy policy, the normal firewall policy may still control the traffic.

If a proxy policy uses:

```text
destination = all
```

then proxy processing can become the path for all applicable client requests.

---

# 36. Transparent Proxy + Upstream Proxy

```text
Client
   |
   | Normal HTTP
   v
FortiGate
   |
   | Transparent Proxy
   v
Upstream Proxy
   |
   v
Internet
```

Client:

```text
No proxy configuration required
```

FortiGate:

```text
http-policy-redirect
```

Upstream:

```text
web-proxy forward-server
```

---

# 37. Upstream Proxy Health

Recommended test concept:

```text
FortiGate
    |
    v
Upstream Proxy
    |
    v
google.com
    |
    +---- HTTP 200 -> Healthy
    |
    +---- Failure -> Down
```

Server-down behavior:

```text
block
```

can prevent traffic from bypassing the upstream proxy.

---

# 38. Proxy Header Tracking

Useful for:

* Logging
* Authentication
* Access control
* Client identification
* Upstream policy decisions

Example:

```text
Client
   |
   | HTTP
   v
FortiGate
   |
   +-- X-Client-IP
   +-- Proxy-Name
   +-- User
   +-- Domain
   +-- Local Group
   +-- Remote Group
   |
   v
Upstream Proxy
```

---

# 39. Web Proxy Fast Policy Match

```bash
config web-proxy global
    set fast-policy-match enable
end
```

The notes identify this as enabled by default.

Concepts mentioned:

* Trie
* Aho-Corasick-like matching
* Decision trees
* Hash tables
* IP prefix trees

Used for efficient matching of:

* Keywords
* Tags
* ACL-like IP prefixes
* Routing-like lookups
* CDN-related address selection

---

# 40. WANOpt Compression

### Authentication

```text
auth-type

0 -> none
1 -> PSK
2 -> certificate
```

### Compression / Encoding

```text
encode-type

0 -> none
1 -> LZ4
2 -> Deflate
```

| Type    |  CPU | Compression |
| ------- | ---: | ----------: |
| None    |  ~0% |          0% |
| LZ4     |  ~5% |     ~30–50% |
| Deflate | ~15% |     ~50–70% |

> Values above are the figures recorded in the source notes.

---

# 41. WANOpt Tunnel SSL

```text
tunnel-ssl-algorithm
```

### Low

```text
AES-128-GCM
ECDHE-RSA
```

Characteristics recorded in the notes:

* Faster
* General office traffic
* Latency-sensitive applications

### High

```text
AES-256-GCM
ECDHE-ECDSA
```

Characteristics recorded in the notes:

* Stronger profile
* Higher processing cost
* Enterprise-sensitive deployments

---

# 42. Auto Detection Algorithm

```text
auto-detect-algorithm
```

### Simple

```text
Port-based detection
```

Characteristics:

```text
Lower processing cost
Stable environments
```

### Reliable

```text
Deep packet inspection
```

Characteristics:

```text
Higher accuracy
Higher processing cost
Complex environments
```

---

# 43. WANOpt Authentication Group

```bash
config wanopt auth-group
    ...
end
```

Authentication types:

```text
certificate
PSK
digest
```

Peer acceptance:

```text
any
defined peers
```

---

# 44. WANOpt Profile — TCP

```bash
config wanopt profile
    edit wanopt-test-prof

        config tcp
            set status enable
            set secure-tunnel disable
            set byte-caching enable
            set byte-caching-opt mem-only
            set tunnel-sharing private
            set log-traffic enable
            set port 443
            set ssl enable
            set ssl-port 465 993 995
        end

    next
end
```

### Byte Cache Modes

```text
mem-only
mix
disk-only
```

| Mode        | Storage    | Performance |
| ----------- | ---------- | ----------- |
| `mem-only`  | RAM        | Fastest     |
| `mix`       | RAM + Disk | Balanced    |
| `disk-only` | Disk       | Persistent  |

---

# 45. WANOpt Tunnel Sharing

### Private

```text
Dedicated tunnel per peer
```

Useful for:

* Point-to-point
* Dedicated protocol flows
* Aggressive optimization

### Shared

```text
Shared tunnel
```

Advantages:

* Resource conservation

### Express-Shared

The notes associate this with shared live traffic such as:

```text
SNMP
Telnet
```

---

# 46. WANOpt FTP Profile

```bash
config ftp
    set status disable
    set secure-tunnel disable
    set byte-caching enable
    set ssl disable
    set prefer-chunking fix
    set protocol-opt protocol
    set tunnel-sharing private
    set log-traffic enable
end
```

### Chunking

```text
fix
dynamic
disable
```

`fix`:

```text
Fixed block sizes
Predictable transfers
Backup/recovery scenarios
```

`dynamic`:

```text
Adaptive behavior
```

---

# 47. Cache Validation

### `If-Modified-Since`

```text
Client
   |
   | If-Modified-Since
   v
Origin
   |
   +---- 304 -> Cached object valid
   |
   +---- 200 -> New content
```

### HTTP/1.1 Conditionals

Controls whether cached objects are validated or served directly.

### Pragma: no-cache

Legacy HTTP/1.0 cache control mechanism.

### IE Reload

Controls cache behavior for IE reload requests.

---

# 48. Expired Objects

### Cache Expired Objects = Enable

FortiGate may serve stale content when:

* Origin unavailable
* Connection timeout

The notes associate this with:

```text
Warning 110
Response is stale
```

### Disabled

Origin failure can result in:

```text
504 Gateway Timeout
```

---

# 49. Advanced HTTP Settings

Recorded values:

```text
Max HTTP Request Length
    8 KB

Max HTTP Message Length
    32 KB
```

These control HTTP request/header and message sizes used by the proxy processing path.

---

# 50. WANOpt CDN Rule — VCache

Example:

```bash
config wanopt content-delivery-network-rule
    edit vcache://
        set status enable
        set category vcache
        set request-cache-control disable
        set response-cache-control disable
        set response-expires enable
        set updateserver disable

        config rules
            edit 1
                set match-mode all
                set skip-rule-mode all

                config match-entries
                    edit 1
                        set target path
                        set pattern "/*.m3u8"
                    next
                end

                config content-id
                    set target hls-manifest
                    set start-str "/"
                    set start-skip 0
                    set start-direction forward
                    set end-str
                    set end-skip 0
                    set end-direction forward
                    set range-str ''
                end
            next
        end
    next
end
```

---

# 51. WANOpt CDN Rule — Windows Update

```bash
config wanopt content-delivery-network-rule
    edit update://windowsupdate/
        set status enable
        set host-domain-name-suffix download.windowsupdate.com
        set category vcache
        set request-cache-control enable
        set response-cache-control enable
        set response-expires enable
        set updateserver enable
    next
end
```

---

# 52. WANOpt CDN Rule — YouTube

```bash
config wanopt content-delivery-network-rule
    edit cache://youtube/
        set status enable
        set host-domain-name-suffix youtube.com
        set category youtube
        set request-cache-control disable
        set response-cache-control disable
        set response-expires disable
        set updateserver disable
    next
end
```

### Path Matching

Example:

```text
/videoplayback
```

Other patterns recorded:

```text
/stream 204
/ptracking
/get_video_info
```

---

# 53. YouTube Content ID

Example:

```bash
config content-id
    set target youtube-id
    set start-str "v="
    set start-skip 2
    set start-direction forward
    set end-str "&"
    set end-skip 0
    set end-direction forward
    set range-str
end
```

Concept:

```text
URL
 |
 +---- v=<VIDEO-ID>
 |
 +---- &
 |
 v
Extract YouTube ID
```

---

# 54. Content-ID Types

Recorded targets include:

```text
path
parameter
referrer
youtube-map
youtube-id
youku-id
hls-manifest
dash-manifest
hls-fragment
dash-fragment
```

---

# 55. WANOpt Peer Behavior

If requested content exists on a peer:

```text
Client
   |
   v
Local WANOpt
   |
   | Cache miss locally
   v
Peer WANOpt
   |
   | Cache hit
   v
Client
```

If no peer has the content:

```text
WANOpt
   |
   v
Internet / Origin
```

Then the content can be cached for later requests.

---

# 56. WANOpt Traffic Types

### Good Candidates

* Repeated HTTP content
* Repeated downloads
* Large files
* Branch-office traffic
* Cacheable SaaS content
* Update content
* Streaming content where supported

### Bypass Candidates

The notes recommend bypassing WANOpt for:

```text
Real-time traffic
    VoIP
    Video calls

Already compressed content
    ZIP
    RAR

Encrypted streams
```

---

# 57. Active vs Passive WANOpt

| Mode                 | Active               | Passive             |
| -------------------- | -------------------- | ------------------- |
| Traffic modification | Yes                  | No                  |
| Optimization         | Full                 | Monitoring-oriented |
| TCP termination      | Yes                  | No                  |
| Deployment           | Controlled endpoints | Single-ended/pilot  |
| Statistics           | Yes                  | Yes                 |
| Best use             | Dedicated WAN        | Baseline/testing    |

### Active

```text
FortiGate
   |
   | Terminates connection
   v
Optimization
   |
   v
Destination
```

### Passive

```text
Traffic
   |
   v
FortiGate
   |
   +---- Observe
   +---- Measure
   +---- No traffic modification
```

---

# 58. WANOpt Storage Strategy

| Storage    | Latency  | Use                         |
| ---------- | -------- | --------------------------- |
| Memory     | Lowest   | Active/repeated connections |
| Disk       | Higher   | Large cached objects        |
| RAM + Disk | Balanced | General deployment          |

---

# 59. Proxy + WANOpt

A combined design can use:

```text
Client
   |
   v
FortiGate Proxy
   |
   +---- Authentication
   +---- Web Filtering
   +---- SSL handling
   +---- WAN Optimization
   +---- Cache
   |
   v
Internet / Upstream Proxy
```

### Important distinction

WANOpt SSL handling is not equivalent to full SSL inspection.

```text
WANOpt
    |
    +---- Certificate handling
    +---- TLS optimization
    +---- Limited visibility

SSL Inspection
    |
    +---- Full decryption
    +---- Content inspection
    +---- Re-encryption
```

---

# 60. WANOpt HTTPS Flow

```text
Client
   |
   | TLS Handshake
   v
FortiGate
   |
   | FortiGate certificate
   v
Client

FortiGate
   |
   | Separate TLS session
   v
Server
```

Without full SSL inspection:

```text
Application payload remains encrypted.
```

---

# 61. Web Cache Statistics

### `worker.protos.http`

```text
wan.bytes_in
wan.bytes_out

lan.bytes_in
lan.bytes_out

tunnel.bytes_in
tunnel.bytes_out
```

Interpretation:

| Counter            | Meaning                |
| ------------------ | ---------------------- |
| `wan.bytes_in`     | Internet → FortiGate   |
| `wan.bytes_out`    | FortiGate → Internet   |
| `lan.bytes_in`     | Client → FortiGate     |
| `lan.bytes_out`    | FortiGate → Client     |
| `tunnel.bytes_in`  | Peer → Local FortiGate |
| `tunnel.bytes_out` | Local FortiGate → Peer |

---

# 62. Compression Statistics

```text
comp.n_in_raw_bytes
```

Original incoming data.

```text
comp.n_in_comp_bytes
```

Compressed incoming data.

```text
comp.n_out_raw_bytes
```

Outbound raw data.

```text
comp.n_out_comp_bytes
```

Outbound compressed data.

### Compression Ratio

Conceptually:

```text
Compression Ratio
    =
Compressed Size / Raw Size
```

---

# 63. WANOpt Verification Checklist

```text
[ ] WANOpt enabled
[ ] Required storage available
[ ] Storage configured for WANOpt
[ ] Peer configured
[ ] Authentication configured
[ ] WANOpt profile configured
[ ] Firewall policy references WANOpt
[ ] Proxy policy configured if required
[ ] SSL profile configured if required
[ ] Cache behavior reviewed
[ ] WANOpt statistics checked
[ ] Event logs checked
[ ] Test traffic generated
```

---

# 64. Proxy Verification Checklist

```text
[ ] Explicit Proxy enabled
[ ] Listen interface configured
[ ] Proxy port configured
[ ] Client proxy configured
[ ] PAC tested if used
[ ] Proxy policy configured
[ ] Authentication schema configured
[ ] Authentication rule configured
[ ] LDAP/FSSO/NTLM tested
[ ] Security profiles attached
[ ] DNS resolution verified
[ ] Upstream proxy reachable
[ ] Health monitor configured
[ ] Logs checked
```

---

# 65. Troubleshooting Commands

## WAD Users

```bash
diagnose wad user list
```

## WAD Web Cache

```bash
diagnose wad webcahe list 10
```

## WAD HTTP Debug

```bash
diagnose wad debug enable category http
diagnose wad debug enable level info
diagnose debug enable
```

## WAD Full Debug

```bash
diagnose wad debug enable category all
diagnose wad debug enable level verbose
diagnose debug enable
```

---

# 66. FortiGuard / Cloud Proxy Debug

```bash
diagnose debug application forticldd -1
diagnose debug enable
```

Useful when FortiGate must reach FortiGuard/FortiSandbox through an explicit proxy.

---

# 67. Connectivity Tests

### TCP Port Test

```bash
execute telnet 192.168.122.2 9696
```

Useful for validating:

```text
TCP reachability
Open proxy port
Upstream listener
```

### HTTP Test

```bash
curl -k https://<TEST-DOMAIN>/<FILE>
```

---

# 68. Proxy Chaining Troubleshooting

Check in this order:

```text
1. Client
   |
   +-- Correct proxy configuration?
   |
2. FortiGate
   |
   +-- Proxy listening?
   |
3. Proxy Policy
   |
   +-- Matching?
   |
4. Authentication
   |
   +-- User authenticated?
   |
5. Upstream Proxy
   |
   +-- TCP reachable?
   |
6. Health Monitor
   |
   +-- HTTP 200?
   |
7. Destination
   |
   +-- DNS?
   +-- Routing?
   +-- Return traffic?
```

---

# 69. Transparent Proxy Troubleshooting

Check:

```text
[ ] inspection-mode = proxy
[ ] http-policy-redirect enabled
[ ] Firewall policy direction correct
[ ] Proxy policy direction correct
[ ] Proxy policy exists
[ ] Client does not need explicit proxy configuration
[ ] DNS resolution works
[ ] Security profile does not block request
```

---

# 70. Authentication Troubleshooting

### If User Is Not Displayed

```bash
diagnose wad user list
```

Check:

```text
LDAP
FSSO
Authentication Schema
Authentication Rule
Proxy Policy
```

### HTTP Tunnel Authentication

If groups do not appear as expected:

```bash
config firewall proxy-policy
    edit 1
        set http-tunnle-auth enable
    next
end
```

Then use:

```text
Basic Authentication
+
LDAP
```

---

# 71. Proxy Authentication Order

Conceptual flow:

```text
DNS filtering
      |
      v
Firewall policy
      |
      v
Authentication rule
      |
      v
Proxy policy
      |
      v
Web filtering / Security Profiles
      |
      v
Upstream proxy
```

The notes emphasize that DNS filtering occurs before later proxy-policy processing.

---

# 72. FSSO vs LDAP

| Feature              | FSSO            | LDAP                 |
| -------------------- | --------------- | -------------------- |
| User identification  | Transparent     | Authentication-based |
| Client login prompt  | Usually avoided | Can occur            |
| Deployment           | More integrated | Lightweight          |
| Proxy authentication | Supported       | Supported            |
| Management           | Identity-aware  | Directory-based      |

---

# 73. Proxy Security Design

For sensitive environments:

```text
Client
   |
   v
Authentication
   |
   v
Proxy Policy
   |
   +---- Web Filter
   +---- AV
   +---- DLP
   +---- SSL Inspection
   +---- Logging
   |
   v
Upstream Proxy
   |
   v
Internet
```

Recommended controls from the notes:

* Deep inspection where required.
* DLP for sensitive information.
* Header tracking carefully.
* FSSO for identity-based access.
* LDAP for lightweight authentication.
* Restrict client proxy configuration changes.
* Monitor proxy authentication.
* Log header changes when required.

---

# 74. Proxy Credential Protection

Avoid exposing credentials through:

```text
HTTP
Proxy sniffing
Unprotected headers
Debug output
Configuration exports
```

Prefer identity-based mechanisms such as:

```text
FSSO
Kerberos
mTLS
```

where appropriate.

---

# 75. Proxy + Microsoft 365 Flow

```text
Client
   |
   v
FortiGate Explicit Proxy
   |
   | Deep Inspection
   |
   +---- URL Filter
   |
   +---- Header Injection
   |
   +---- Tenant Restriction
   |
   v
Microsoft 365
```

---

# 76. WANOpt + Proxy Flow

```text
Client
   |
   v
Proxy
   |
   +---- Authentication
   +---- URL filtering
   +---- Security inspection
   |
   v
WAN Optimization
   |
   +---- Byte caching
   +---- Compression
   +---- TCP optimization
   +---- Cache
   |
   v
Upstream / Internet
```

---

# 77. Key Commands — Quick Reference

| Purpose                | Command                                   |
| ---------------------- | ----------------------------------------- |
| WAD users              | `diagnose wad user list`                  |
| WAD cache              | `diagnose wad webcahe list 10`            |
| WAD HTTP debug         | `diagnose wad debug enable category http` |
| WAD verbose debug      | `diagnose wad debug enable category all`  |
| WANOpt tunnel stats    | `diagnose wad stats worker.tunnel`        |
| WANOpt HTTP stats      | `diagnose wad stats worker.protos.http`   |
| FortiCloud proxy debug | `diagnose debug application forticldd -1` |
| TCP test               | `execute telnet <IP> <PORT>`              |
| Disk information       | `diagnose hardware deviceinfo disk`       |
| Disk list              | `execute disk list`                       |
| Disk format            | `execute disk format 32`                  |

---

# 78. Quick CLI Templates

## Explicit Proxy

```bash
config system interface
    edit port3
        set ip 192.168.101.1 255.255.255.0
        set allowaccess ping https http
    next
end
```

Client:

```text
Proxy = 192.168.101.1
Port  = 8080
```

---

## Transparent Proxy

```bash
config firewall policy
    edit 1
        set inspection-mode proxy
        set srcintf port3
        set dstintf port1
        set action accept
        set srcaddr all
        set dstaddr all
        set schedule always
        set service ALL
        set logtraffic all
        set nat enable
        set http-policy-redirect enable
    next
end
```

---

## WANOpt Storage

```bash
config system storage
    edit hdd2
        set status enable
        set usage wanopt
        set wanopt-mode mix
    next
end
```

---

## WANOpt Peer

```bash
config wanopt peer
    edit fgt-2
        set ip 192.168.254.251
    next
end
```

---

## WANOpt Profile

```bash
config wanopt profile
    edit wanopt-test-prof

        config http
            set status enable
            set ssl enable
        end

    next
end
```

---

## WANOpt Policy

```bash
config firewall policy
    edit 1
        set srcintf port3
        set dstintf port1
        set action accept
        set srcaddr all
        set dstaddr all
        set schedule always
        set service ALL
        set inspection-mode proxy
        set wanopt enable
        set wanopt-profile wanopt-test-prof
        set nat enable
    next
end
```

---

# 79. Design Decision Matrix

| Requirement                       | Recommended Feature     |
| --------------------------------- | ----------------------- |
| Client explicitly knows proxy     | Explicit Proxy          |
| No client proxy configuration     | Transparent Proxy       |
| Automatic client proxy selection  | PAC                     |
| FTP application control           | FTP Proxy               |
| User authentication               | Proxy Authentication    |
| Directory authentication          | LDAP                    |
| Transparent user identity         | FSSO                    |
| Windows integrated authentication | NTLM/Kerberos           |
| Certificate-based authentication  | mTLS                    |
| Repeated content acceleration     | WANOpt Cache            |
| Repeated byte elimination         | Byte Caching            |
| Bandwidth reduction               | Compression             |
| Multiple WANOpt peers             | Collaboration           |
| Upstream proxy                    | Forward Server          |
| Multiple upstream proxies         | Forward Server Group    |
| SaaS tenant restriction           | HTTP Header Injection   |
| Application content inspection    | SSL Inspection          |
| TLS performance optimization      | WANOpt SSL optimization |

---

# 80. Final Troubleshooting Flow

```text
                     Client Problem
                           |
                           v
                 +-------------------+
                 | DNS Resolution OK?|
                 +---------+---------+
                           |
                          Yes
                           |
                           v
                 +-------------------+
                 | TCP Connectivity? |
                 +---------+---------+
                           |
                          Yes
                           |
                           v
                 +-------------------+
                 | Proxy Listening?  |
                 +---------+---------+
                           |
                          Yes
                           |
                           v
                 +-------------------+
                 | Proxy Policy Hit? |
                 +---------+---------+
                           |
                          Yes
                           |
                           v
                 +-------------------+
                 | Authentication?   |
                 +---------+---------+
                           |
                          Yes
                           |
                           v
                 +-------------------+
                 | Security Profile? |
                 +---------+---------+
                           |
                          Yes
                           |
                           v
                 +-------------------+
                 | WANOpt / Cache?   |
                 +---------+---------+
                           |
                          Yes
                           |
                           v
                 +-------------------+
                 | Upstream Proxy?   |
                 +---------+---------+
                           |
                          Yes
                           |
                           v
                       Internet
```

---

## ⚡ One-Page Memory Map

```text
FORTIGATE PROXY
│
├── Explicit Proxy
│   ├── Client proxy
│   ├── PAC
│   ├── Authentication
│   └── Proxy Policy
│
├── Transparent Proxy
│   ├── No client configuration
│   ├── HTTP redirect
│   └── Proxy Policy
│
├── FTP Proxy
│   ├── Port 21
│   └── FTP Proxy Policy
│
├── Authentication
│   ├── LDAP
│   ├── NTLM
│   ├── FSSO
│   ├── Kerberos
│   └── mTLS
│
├── Proxy Headers
│   ├── Client IP
│   ├── User
│   ├── Domain
│   ├── Groups
│   └── Proxy Name
│
├── SaaS Control
│   ├── Microsoft 365
│   ├── Google
│   └── Dropbox
│
├── Upstream Proxy
│   ├── Forward Server
│   ├── Forward Server Group
│   ├── Health Monitor
│   └── Load Balancing
│
└── WAN OPTIMIZATION
    ├── TCP Optimization
    ├── Byte Caching
    ├── Compression
    ├── HTTP/HTTPS Optimization
    ├── Web Cache
    ├── CDN / VCache
    ├── HLS / DASH
    ├── Peer Collaboration
    └── Statistics / Troubleshooting
```

> **Security note:** Replace all example usernames, passwords, PSKs, UUIDs, certificates, IPs and domain values before committing the cheatsheet to a public repository.

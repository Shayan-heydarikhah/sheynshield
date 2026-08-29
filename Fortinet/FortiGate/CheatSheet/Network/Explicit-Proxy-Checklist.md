# FortiGate Proxy, WAN Optimization & Web Acceleration — Checklist

> **SheynShield | Engineering Secure Networks**
> **Scope:** FortiOS 7.2.x
> **Type:** Deployment • Validation • Security • Troubleshooting Checklist
> **Focus:** Explicit Proxy, Transparent Proxy, FTP Proxy, Proxy Authentication, PAC, NTLM/Kerberos, mTLS, Upstream Proxy, WAN Optimization, Web Cache, CDN/VCache

---

## Table of Contents

* [1. Pre-Deployment Checklist](#1-pre-deployment-checklist)
* [2. Proxy Architecture Checklist](#2-proxy-architecture-checklist)
* [3. Explicit Proxy Checklist](#3-explicit-proxy-checklist)
* [4. PAC File Checklist](#4-pac-file-checklist)
* [5. Transparent Proxy Checklist](#5-transparent-proxy-checklist)
* [6. FTP Proxy Checklist](#6-ftp-proxy-checklist)
* [7. Proxy Policy Checklist](#7-proxy-policy-checklist)
* [8. Proxy Authentication Checklist](#8-proxy-authentication-checklist)
* [9. LDAP / FSSO / NTLM Checklist](#9-ldap--fsso--ntlm-checklist)
* [10. Kerberos / Keytab Checklist](#10-kerberos--keytab-checklist)
* [11. mTLS Access Proxy Checklist](#11-mtls-access-proxy-checklist)
* [12. Proxy Header Checklist](#12-proxy-header-checklist)
* [13. SaaS / Microsoft 365 Checklist](#13-saas--microsoft-365-checklist)
* [14. Upstream Proxy Checklist](#14-upstream-proxy-checklist)
* [15. Proxy Chaining Checklist](#15-proxy-chaining-checklist)
* [16. SD-WAN + Upstream Proxy Checklist](#16-sd-wan--upstream-proxy-checklist)
* [17. WAN Optimization Checklist](#17-wan-optimization-checklist)
* [18. WANOpt Storage Checklist](#18-wanopt-storage-checklist)
* [19. WANOpt Peer Checklist](#19-wanopt-peer-checklist)
* [20. WANOpt Profile Checklist](#20-wanopt-profile-checklist)
* [21. HTTPS Optimization Checklist](#21-https-optimization-checklist)
* [22. WANOpt Cache Checklist](#22-wanopt-cache-checklist)
* [23. Cache Validation Checklist](#23-cache-validation-checklist)
* [24. CDN / VCache Checklist](#24-cdn--vcache-checklist)
* [25. HLS / MPEG-DASH Checklist](#25-hls--mpeg-dash-checklist)
* [26. WANOpt Statistics Checklist](#26-wanopt-statistics-checklist)
* [27. WANOpt Troubleshooting Checklist](#27-wanopt-troubleshooting-checklist)
* [28. Proxy Troubleshooting Checklist](#28-proxy-troubleshooting-checklist)
* [29. Authentication Troubleshooting Checklist](#29-authentication-troubleshooting-checklist)
* [30. Security Hardening Checklist](#30-security-hardening-checklist)
* [31. Performance Optimization Checklist](#31-performance-optimization-checklist)
* [32. Final Validation Checklist](#32-final-validation-checklist)
* [33. Quick CLI Reference](#33-quick-cli-reference)
* [34. One-Page Decision Matrix](#34-one-page-decision-matrix)

---

# 1. Pre-Deployment Checklist

## Architecture

* [ ] Define whether the deployment requires **Explicit Proxy**
* [ ] Define whether the deployment requires **Transparent Proxy**
* [ ] Define whether **PAC** is required
* [ ] Define whether **FTP Proxy** is required
* [ ] Define whether **proxy authentication** is required
* [ ] Define whether an **upstream proxy** is required
* [ ] Define whether **WAN Optimization** is required
* [ ] Define whether **Web Cache** is required
* [ ] Define whether **CDN/VCache** is required
* [ ] Define whether **SSL inspection** is required
* [ ] Define whether **mTLS** is required
* [ ] Identify all client networks
* [ ] Identify all proxy interfaces
* [ ] Identify all upstream proxies
* [ ] Identify authentication servers
* [ ] Identify DNS servers
* [ ] Identify Internet egress interfaces
* [ ] Document expected traffic flows

## Network Prerequisites

* [ ] Client default gateway is correct
* [ ] FortiGate interface addressing is correct
* [ ] DNS resolution works
* [ ] Routing is correct
* [ ] Return routes exist where NAT is disabled
* [ ] Required TCP ports are reachable
* [ ] Upstream proxy listeners are reachable
* [ ] Authentication servers are reachable
* [ ] Certificate trust requirements are documented

---

# 2. Proxy Architecture Checklist

## Basic Flow

```text
Client
   |
   v
FortiGate Proxy
   |
   +-- Authentication
   |
   +-- Proxy Policy
   |
   +-- Web Filtering
   |
   +-- Security Inspection
   |
   +-- Cache
   |
   +-- WAN Optimization
   |
   v
Upstream Proxy / Internet
```

* [ ] Traffic flow is documented
* [ ] Firewall policy direction is correct
* [ ] Proxy policy direction is correct
* [ ] NAT behavior is documented
* [ ] Authentication point is documented
* [ ] SSL inspection point is documented
* [ ] Upstream proxy behavior is documented
* [ ] Failure/bypass behavior is documented
* [ ] Logging requirements are defined

---

# 3. Explicit Proxy Checklist

## Configuration

* [ ] Explicit Proxy feature is enabled
* [ ] Correct listening interface is configured
* [ ] Correct HTTP proxy port is configured
* [ ] Proxy FQDN is defined where required
* [ ] Default proxy action is reviewed
* [ ] Default action is not unintentionally permissive

Example client configuration:

```text
Proxy:
    IP: <FORTIGATE-IP>

Port:
    8080
```

## Client Validation

* [ ] Client proxy configuration is correct
* [ ] Client can resolve proxy FQDN
* [ ] Client can reach proxy TCP port
* [ ] HTTP request reaches FortiGate
* [ ] HTTPS CONNECT request is processed
* [ ] Authentication prompt appears when expected
* [ ] Proxy policy is matched
* [ ] Internet access works

## Security

* [ ] Proxy is not exposed unnecessarily
* [ ] Only required source networks can use the proxy
* [ ] Authentication is enabled where required
* [ ] Logging is enabled
* [ ] Security profiles are attached where required

---

# 4. PAC File Checklist

## PAC Logic

* [ ] PAC file is reachable by clients
* [ ] `FindProxyForURL()` function is valid
* [ ] Internal destinations use `DIRECT` where required
* [ ] Local hostnames are handled
* [ ] RFC1918 networks are handled where required
* [ ] Internal domains are excluded
* [ ] External destinations use the proxy
* [ ] Primary proxy is defined
* [ ] Backup proxy is defined where required
* [ ] `DIRECT` fallback is intentional

Example:

```javascript
function FindProxyForURL(url, host) {

    if (
        isPlainHostName(host) ||
        dnsDomainIs(host, ".local") ||
        isInNet(host, "10.0.0.0", "255.0.0.0") ||
        isInNet(host, "172.16.0.0", "255.240.0.0") ||
        isInNet(host, "192.168.0.0", "255.255.0.0")
    ) {
        return "DIRECT";
    }

    return "PROXY proxy.company.com:8080";
}
```

## PAC Failover

* [ ] Primary proxy is reachable
* [ ] Secondary proxy is reachable
* [ ] Failover behavior is tested
* [ ] `DIRECT` fallback is intentional
* [ ] Domain matching is tested
* [ ] Case sensitivity is considered
* [ ] Wildcard matching is validated

---

# 5. Transparent Proxy Checklist

* [ ] Client-side proxy configuration is not required
* [ ] Firewall policy uses proxy inspection mode
* [ ] `http-policy-redirect` is enabled
* [ ] Firewall policy direction is correct
* [ ] Proxy policy direction matches
* [ ] Proxy policy exists
* [ ] HTTP traffic is redirected
* [ ] HTTPS behavior is understood
* [ ] DNS resolution works
* [ ] NAT behavior is validated

Example:

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

### Critical Validation

* [ ] `inspection-mode = proxy`
* [ ] `http-policy-redirect` is enabled
* [ ] Proxy policy exists
* [ ] Traffic direction is correct

---

# 6. FTP Proxy Checklist

* [ ] FTP Proxy is enabled if required
* [ ] Incoming port is configured
* [ ] Incoming IP is correct
* [ ] Default security action is reviewed
* [ ] FTP proxy interface is configured
* [ ] `explicit-ftp-proxy` is enabled where required
* [ ] FTP proxy policy exists
* [ ] Destination interface is correct
* [ ] FTP authentication requirements are documented
* [ ] FTP client test is successful

Example:

```bash
config ftp-proxy explicit
    set status enable
    set incoming-port 21
    set incoming-ip 0.0.0.0
    set sec-default-action deny
    set ssl disable
end
```

---

# 7. Proxy Policy Checklist

## Matching

* [ ] Proxy type is correct
* [ ] Source interface is correct
* [ ] Destination interface is correct
* [ ] Source address is correct
* [ ] Destination address is correct
* [ ] Service is correct
* [ ] Schedule is correct
* [ ] Policy action is correct

## Application Controls

* [ ] Host matching is reviewed
* [ ] URL matching is reviewed
* [ ] URL category matching is reviewed
* [ ] HTTP method matching is reviewed
* [ ] User-Agent matching is reviewed
* [ ] Header matching is reviewed
* [ ] Authentication is attached
* [ ] Web filtering is attached
* [ ] AV is attached where required
* [ ] DLP is attached where required
* [ ] SSL inspection is attached where required
* [ ] Logging is enabled

## Policy Order

* [ ] More-specific proxy policies appear before broad policies
* [ ] `destination = all` is intentional
* [ ] Unmatched traffic behavior is understood
* [ ] Firewall policy and proxy policy are not conflicting

---

# 8. Proxy Authentication Checklist

## Authentication Architecture

* [ ] Authentication requirement is documented
* [ ] Authentication schema exists
* [ ] Authentication rule exists
* [ ] Authentication protocol is correct
* [ ] Authentication server is reachable
* [ ] User/group mapping is correct
* [ ] Proxy policy references required identity objects

Supported mechanisms may include:

```text
LDAP
FSSO
NTLM
Kerberos
mTLS
```

## Authentication Flow

```text
Client
   |
   v
Authentication Rule
   |
   v
Authentication Schema
   |
   v
Identity Provider
   |
   v
Proxy Policy
   |
   v
Internet
```

* [ ] Authentication challenge is generated when expected
* [ ] User is identified correctly
* [ ] Group membership is correct
* [ ] Authenticated user appears in WAD
* [ ] Access is allowed only by intended policy

---

# 9. LDAP / FSSO / NTLM Checklist

## LDAP

* [ ] LDAP server is configured
* [ ] LDAP IP/FQDN is correct
* [ ] LDAP port is reachable
* [ ] Base DN is correct
* [ ] Bind credentials are valid
* [ ] LDAP users can be resolved
* [ ] LDAP groups can be resolved
* [ ] LDAP authentication test succeeds

## FSSO

* [ ] FSSO collector/identity source is available
* [ ] User identity is received
* [ ] Group mapping is correct
* [ ] Authentication prompt behavior is understood
* [ ] WAD identifies the user correctly

## NTLM

* [ ] Domain is correct
* [ ] Domain controller is reachable
* [ ] LDAP server is correct
* [ ] TCP/445 reachability is verified
* [ ] Authentication schema uses NTLM
* [ ] Authentication rule references correct schema
* [ ] Proxy policy references correct identity
* [ ] Client authentication succeeds

Verification:

```bash
diagnose wad user list
```

---

# 10. Kerberos / Keytab Checklist

* [ ] LDAP server is configured
* [ ] Domain DN is correct
* [ ] Kerberos principal is correct
* [ ] Keytab is configured
* [ ] Keytab principal matches expected FQDN
* [ ] LDAP servers are associated correctly
* [ ] Domain controller is configured
* [ ] DNS forward resolution works
* [ ] DNS reverse resolution works where required
* [ ] Time synchronization is correct
* [ ] Kerberos authentication is tested

Example principal:

```text
HTTP/fgt.example.local@EXAMPLE.LOCAL
```

---

# 11. mTLS Access Proxy Checklist

## Certificate

* [ ] Server certificate is configured
* [ ] Client certificate requirement is enabled
* [ ] CA trust chain is correct
* [ ] Client certificate is valid
* [ ] Certificate expiration is checked
* [ ] Certificate identity mapping is correct

## Access Proxy

* [ ] VIP type is `access-proxy`
* [ ] External IP is correct
* [ ] HTTPS port is correct
* [ ] SSL certificate is correct
* [ ] Client certificate validation is enabled
* [ ] Real server is configured
* [ ] Proxy policy references access proxy

## Header Forwarding

* [ ] Client certificate forwarding is required
* [ ] `X-Forwarded-Client-Cert` behavior is configured
* [ ] Backend application expects the certificate header
* [ ] Header security implications are reviewed

---

# 12. Proxy Header Checklist

## Client Identity

* [ ] Client IP forwarding is required
* [ ] Existing client-IP headers are handled correctly
* [ ] User identity header is required
* [ ] Domain header is required
* [ ] Group headers are required
* [ ] Proxy name header is required

Possible variables:

```text
$client-ip
$user
$domain
$local_grp
$remote_grp
$proxy_name
```

## Security

* [ ] Header injection is intentional
* [ ] Header spoofing is prevented
* [ ] Existing headers are removed/replaced when required
* [ ] Sensitive identity headers are not exposed unnecessarily
* [ ] Header modifications are logged where required

---

# 13. SaaS / Microsoft 365 Checklist

## Microsoft 365

* [ ] Microsoft 365 tenant restriction requirement is documented
* [ ] Required Microsoft authentication destinations are identified
* [ ] `Restrict-Access-To-Tenants` is configured where required
* [ ] `Restrict-Access-Context` is configured where required
* [ ] `Sec-Restrict-Tenant-Access-Policy` is configured where required
* [ ] Tenant/domain value is correct
* [ ] Directory ID is correct
* [ ] HTTPS inspection requirements are understood
* [ ] Header injection is tested

Common destinations:

```text
login.microsoftonline.com
login.microsoft.com
login.windows.net
login.live.com
```

## Google

* [ ] `X-GoogAppsAllowed-Domains` requirement is documented
* [ ] Allowed domain list is correct

## Dropbox

* [ ] `X-Dropbox-allowedTeam-Ids` requirement is documented
* [ ] Allowed Team IDs are correct

---

# 14. Upstream Proxy Checklist

## Forward Server

* [ ] Upstream proxy IP is correct
* [ ] Upstream proxy port is correct
* [ ] TCP reachability is verified
* [ ] Authentication requirements are documented
* [ ] DNS requirements are verified

Example:

```bash
config web-proxy forward-server

    edit server1
        set ip <UPSTREAM-IP>
        set port 8080
    next

end
```

## Multiple Upstream Proxies

* [ ] All upstream proxies are configured
* [ ] Each proxy has a unique identifier
* [ ] Each proxy listener is reachable
* [ ] Load-balancing method is documented
* [ ] Server weights are reviewed
* [ ] Affinity requirement is reviewed
* [ ] Failure behavior is defined

---

# 15. Proxy Chaining Checklist

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

* [ ] Client reaches FortiGate
* [ ] FortiGate proxy is listening
* [ ] Proxy policy matches
* [ ] Authentication succeeds
* [ ] Forward-server group is selected
* [ ] Upstream proxy is reachable
* [ ] Upstream proxy can reach Internet
* [ ] Health monitor is configured
* [ ] HTTP 200 health response is verified
* [ ] Failure behavior is intentional
* [ ] Return routing is correct

## Fail-Safe

* [ ] `group-down-option` is reviewed
* [ ] Bypass to Internet is intentional
* [ ] Traffic blocking during upstream failure is tested

---

# 16. SD-WAN + Upstream Proxy Checklist

* [ ] SD-WAN members are configured
* [ ] SD-WAN health checks are configured
* [ ] SLA targets are reachable
* [ ] SD-WAN rule order is correct
* [ ] Proxy-specific routing is documented
* [ ] Required FQDN categories are identified
* [ ] Default route points to intended SD-WAN path
* [ ] NAT behavior is documented
* [ ] Return routes exist when NAT is disabled

Example SLA targets:

```text
8.8.8.8
1.1.1.1
```

---

# 17. WAN Optimization Checklist

## Enablement

* [ ] WANOpt feature is enabled
* [ ] WANOpt license/platform requirements are verified
* [ ] WANOpt topology is documented
* [ ] Peer FortiGate is identified
* [ ] Optimization traffic is identified

## Optimization Features

* [ ] TCP optimization reviewed
* [ ] Byte caching reviewed
* [ ] Compression reviewed
* [ ] HTTP optimization reviewed
* [ ] HTTPS optimization reviewed
* [ ] Web Cache reviewed
* [ ] SSL optimization reviewed
* [ ] Protocol optimization reviewed

## Traffic Selection

Good candidates:

* [ ] Repeated HTTP content
* [ ] Large files
* [ ] Repeated downloads
* [ ] Update content
* [ ] Branch-office traffic
* [ ] Cacheable SaaS content

Potential bypass candidates:

* [ ] VoIP
* [ ] Real-time video
* [ ] Already compressed files
* [ ] ZIP/RAR content
* [ ] Traffic where optimization provides no benefit

---

# 18. WANOpt Storage Checklist

## Storage

* [ ] Required disk exists
* [ ] Disk is detected by FortiGate
* [ ] Disk is healthy
* [ ] Disk is assigned to WANOpt
* [ ] Storage mode is selected intentionally

Example:

```bash
config system storage
    edit hdd2
        set status enable
        set usage wanopt
        set wanopt-mode mix
    next
end
```

## VM Lab

* [ ] VM is backed up before disk modification
* [ ] Additional QCOW2 disks are created
* [ ] VM template is updated
* [ ] Disk permissions are correct
* [ ] VM restarts successfully
* [ ] FortiGate detects the disks
* [ ] Disk is formatted only after validation

Verification:

```bash
diagnose hardware deviceinfo disk
execute disk list
```

---

# 19. WANOpt Peer Checklist

* [ ] Peer IP is correct
* [ ] Peer is reachable
* [ ] Authentication is configured
* [ ] PSK/certificate requirements are documented
* [ ] Peer identity is unique
* [ ] Device ID is unique
* [ ] Peer priority is documented
* [ ] Peer cache collaboration is required/disabled intentionally

Example:

```bash
config wanopt peer
    edit fgt-2
        set ip <PEER-IP>
    next
end
```

---

# 20. WANOpt Profile Checklist

## HTTP

* [ ] HTTP optimization is enabled
* [ ] SSL optimization requirement is reviewed
* [ ] Cache requirements are reviewed
* [ ] Compression requirements are reviewed

## TCP

* [ ] TCP optimization is enabled where required
* [ ] Byte caching is enabled where beneficial
* [ ] Storage mode is selected
* [ ] Tunnel sharing mode is selected
* [ ] SSL ports are defined where required
* [ ] Logging is enabled where required

Example:

```bash
config wanopt profile
    edit wanopt-profile

        config tcp
            set status enable
            set byte-caching enable
            set byte-caching-opt mem-only
            set tunnel-sharing private
            set log-traffic enable
        end

    next
end
```

---

# 21. HTTPS Optimization Checklist

> **Important:** WANOpt SSL optimization must not be treated as equivalent to full SSL deep inspection.

* [ ] HTTPS optimization requirement is documented
* [ ] TLS behavior is understood
* [ ] Certificate behavior is understood
* [ ] Client-side TLS session is understood
* [ ] Server-side TLS session is understood
* [ ] SNI visibility requirements are understood
* [ ] Application payload visibility requirements are understood
* [ ] Full SSL inspection is enabled when content inspection is required
* [ ] Certificate trust is validated

### Visibility

Without full decryption, do not assume visibility into:

```text
HTTP GET/POST
URL path
Form data
API payload
```

---

# 22. WANOpt Cache Checklist

* [ ] Web Cache is enabled where required
* [ ] Cache storage is available
* [ ] Maximum cache object size is defined
* [ ] Minimum TTL is reviewed
* [ ] Maximum TTL is reviewed
* [ ] Default TTL is reviewed
* [ ] Fresh Factor is reviewed
* [ ] Negative response duration is reviewed
* [ ] Always Revalidate behavior is understood
* [ ] Expired-object behavior is reviewed
* [ ] Cache bypass requirements are documented

---

# 23. Cache Validation Checklist

## Revalidation

* [ ] `Always Revalidate` requirement is understood
* [ ] `If-Modified-Since` behavior is tested
* [ ] HTTP `304 Not Modified` behavior is tested
* [ ] HTTP `200 OK` refresh behavior is tested
* [ ] `Cache-Control` behavior is reviewed
* [ ] `Pragma: no-cache` behavior is reviewed

## Expired Objects

* [ ] Stale-object behavior is understood
* [ ] Origin timeout behavior is tested
* [ ] `Warning 110` behavior is understood
* [ ] `504 Gateway Timeout` behavior is understood

---

# 24. CDN / VCache Checklist

* [ ] CDN/VCache requirement is documented
* [ ] Content category is correct
* [ ] Host matching is correct
* [ ] Path matching is correct
* [ ] Match mode is correct
* [ ] Cache-control behavior is intentional
* [ ] Response expiration behavior is reviewed
* [ ] Update-server behavior is reviewed
* [ ] Content ID extraction is tested

Possible content targets:

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

# 25. HLS / MPEG-DASH Checklist

## HLS

* [ ] HLS traffic is identified
* [ ] `.m3u8` manifests are identified
* [ ] Media segments are identified
* [ ] `.ts` segments are identified
* [ ] `.m4a` segments are identified
* [ ] HLS manifest extraction is validated

## MPEG-DASH

* [ ] DASH traffic is identified
* [ ] `.mpd` manifest is identified
* [ ] `.m4s` segments are identified
* [ ] Representation/quality switching is understood
* [ ] DASH manifest extraction is validated

---

# 26. WANOpt Statistics Checklist

## Tunnel

Run:

```bash
diagnose wad stats worker.tunnel
```

Check:

* [ ] `comp.n_in_raw_bytes`
* [ ] `comp.n_in_comp_bytes`
* [ ] `comp.n_out_raw_bytes`
* [ ] `comp.n_out_comp_bytes`
* [ ] Tunnel traffic is increasing
* [ ] Compression is producing expected results

## HTTP

Run:

```bash
diagnose wad stats worker.protos.http
```

Check:

* [ ] `wan.bytes_in`
* [ ] `wan.bytes_out`
* [ ] `lan.bytes_in`
* [ ] `lan.bytes_out`
* [ ] `tunnel.bytes_in`
* [ ] `tunnel.bytes_out`

---

# 27. WANOpt Troubleshooting Checklist

## Tunnel

* [ ] Peer is reachable
* [ ] WANOpt peer configuration is correct
* [ ] Authentication succeeds
* [ ] Tunnel statistics increase
* [ ] Expected traffic is entering tunnel
* [ ] Expected traffic is leaving tunnel

## HTTP

* [ ] HTTP worker is active
* [ ] LAN counters increase
* [ ] WAN counters increase
* [ ] Tunnel counters increase
* [ ] Cache behavior is observed
* [ ] Compression behavior is observed

## Logs

* [ ] WANOpt event logs checked
* [ ] System events checked
* [ ] Relevant WAD debug enabled
* [ ] Debug output reviewed
* [ ] Debug disabled after troubleshooting

---

# 28. Proxy Troubleshooting Checklist

Follow this order:

```text
Client
  |
  v
DNS
  |
  v
TCP Connectivity
  |
  v
Proxy Listener
  |
  v
Proxy Policy
  |
  v
Authentication
  |
  v
Security Profile
  |
  v
WANOpt / Cache
  |
  v
Upstream Proxy
  |
  v
Internet
```

## Client

* [ ] Proxy IP/FQDN is correct
* [ ] Proxy port is correct
* [ ] PAC is valid
* [ ] PAC selects expected proxy
* [ ] Client can reach proxy

## FortiGate

* [ ] Proxy listener is active
* [ ] Firewall policy matches
* [ ] Proxy policy matches
* [ ] Authentication succeeds
* [ ] Security profile does not block traffic
* [ ] DNS works
* [ ] Routing works

## Upstream

* [ ] Forward server is reachable
* [ ] Correct port is open
* [ ] Health monitor is healthy
* [ ] Upstream proxy can access destination
* [ ] Return traffic works

---

# 29. Authentication Troubleshooting Checklist

If the user is not visible:

```bash
diagnose wad user list
```

Check:

* [ ] LDAP connectivity
* [ ] LDAP credentials
* [ ] LDAP base DN
* [ ] User lookup
* [ ] Group lookup
* [ ] FSSO identity
* [ ] NTLM configuration
* [ ] Kerberos configuration
* [ ] Authentication schema
* [ ] Authentication rule
* [ ] Proxy policy
* [ ] Client authentication method

## HTTP Tunnel Authentication

If required:

```bash
config firewall proxy-policy
    edit 1
        set http-tunnle-auth enable
    next
end
```

Then verify:

* [ ] Basic authentication behavior
* [ ] LDAP authentication
* [ ] Group membership
* [ ] WAD user list

---

# 30. Security Hardening Checklist

## Proxy

* [ ] Default proxy action is reviewed
* [ ] Unauthenticated access is restricted
* [ ] Proxy access is limited to trusted interfaces
* [ ] Proxy port is not unnecessarily exposed
* [ ] Logging is enabled
* [ ] Administrative access is separated from proxy traffic

## Authentication

* [ ] Credentials are protected
* [ ] Debug output does not expose credentials
* [ ] Configuration exports are protected
* [ ] HTTP credential transmission is avoided
* [ ] Kerberos is used where appropriate
* [ ] mTLS is used where appropriate
* [ ] FSSO is used where transparent identity is appropriate

## Headers

* [ ] Header injection is documented
* [ ] Header spoofing is prevented
* [ ] Sensitive headers are restricted
* [ ] Identity headers are not exposed externally
* [ ] Header modification logging is reviewed

## SSL

* [ ] Certificate chain is trusted
* [ ] SSL inspection scope is minimized
* [ ] Deep inspection is enabled only where required
* [ ] Exceptions are documented
* [ ] TLS visibility requirements are understood

---

# 31. Performance Optimization Checklist

## Proxy

* [ ] Policy matching is optimized
* [ ] Broad policies are placed appropriately
* [ ] `fast-policy-match` requirement is reviewed
* [ ] Unnecessary inspection is removed
* [ ] Logging level is appropriate

## WANOpt

* [ ] Appropriate traffic is optimized
* [ ] Real-time traffic is evaluated for bypass
* [ ] Already-compressed traffic is evaluated for bypass
* [ ] Byte caching mode is appropriate
* [ ] Compression algorithm is appropriate
* [ ] Tunnel sharing is appropriate
* [ ] Cache size is sufficient
* [ ] Disk performance is sufficient
* [ ] CPU impact is monitored

## Compression

* [ ] Compression benefit is measured
* [ ] CPU overhead is measured
* [ ] LZ4 vs Deflate is evaluated
* [ ] Compression is not blindly enabled for already-compressed content

---

# 32. Final Validation Checklist

## Connectivity

* [ ] DNS works
* [ ] TCP connectivity works
* [ ] Internet access works
* [ ] Upstream proxy connectivity works
* [ ] Return routing works

## Proxy

* [ ] Explicit Proxy works
* [ ] Transparent Proxy works where required
* [ ] PAC works where required
* [ ] FTP Proxy works where required
* [ ] Proxy Policy matches
* [ ] Authentication works
* [ ] Web filtering works
* [ ] SSL inspection works where required

## WANOpt

* [ ] WANOpt tunnel is operational
* [ ] Peer is reachable
* [ ] Storage is available
* [ ] Byte caching works
* [ ] Compression works
* [ ] Web Cache works
* [ ] CDN/VCache rules work
* [ ] Statistics show expected traffic
* [ ] Logs are clean

## Security

* [ ] Unauthenticated access is restricted
* [ ] Sensitive headers are protected
* [ ] Credentials are protected
* [ ] SSL certificates are trusted
* [ ] Security profiles are active
* [ ] Logging is sufficient

---

# 33. Quick CLI Reference

| Purpose                  | Command                                   |
| ------------------------ | ----------------------------------------- |
| WAD users                | `diagnose wad user list`                  |
| WAD cache                | `diagnose wad webcahe list 10`            |
| WAD HTTP debug           | `diagnose wad debug enable category http` |
| WAD full debug           | `diagnose wad debug enable category all`  |
| WAD verbose debug        | `diagnose wad debug enable level verbose` |
| WANOpt tunnel statistics | `diagnose wad stats worker.tunnel`        |
| WANOpt HTTP statistics   | `diagnose wad stats worker.protos.http`   |
| FortiCloud proxy debug   | `diagnose debug application forticldd -1` |
| Disk information         | `diagnose hardware deviceinfo disk`       |
| Disk list                | `execute disk list`                       |
| Disk format              | `execute disk format 32`                  |
| TCP connectivity         | `execute telnet <IP> <PORT>`              |
| HTTP/HTTPS test          | `curl -k https://<DOMAIN>/<FILE>`         |

> **Note:** Command spelling and availability can vary by FortiOS release and platform. Validate commands against the target FortiOS 7.2.x build before production use.

---

# 34. One-Page Decision Matrix

| Requirement                         | Feature                      |
| ----------------------------------- | ---------------------------- |
| Client explicitly configures proxy  | **Explicit Proxy**           |
| No client proxy configuration       | **Transparent Proxy**        |
| Automatic proxy selection           | **PAC**                      |
| FTP application proxy               | **FTP Proxy**                |
| User authentication                 | **Proxy Authentication**     |
| Directory authentication            | **LDAP**                     |
| Transparent identity                | **FSSO**                     |
| Windows integrated authentication   | **NTLM / Kerberos**          |
| Certificate-based authentication    | **mTLS**                     |
| Multiple upstream proxies           | **Forward Server Group**     |
| Upstream proxy health               | **Health Monitoring**        |
| Repeated content acceleration       | **Web Cache / WANOpt Cache** |
| Repeated byte elimination           | **Byte Caching**             |
| Bandwidth reduction                 | **Compression**              |
| Branch optimization                 | **WAN Optimization**         |
| Streaming acceleration              | **CDN / VCache**             |
| SaaS tenant restriction             | **HTTP Header Injection**    |
| Full application payload inspection | **SSL Deep Inspection**      |
| TLS optimization                    | **WANOpt SSL Optimization**  |

---

# 🔥 SheynShield — Final Mental Model

```text
                         FORTIGATE
                             |
          +------------------+------------------+
          |                  |                  |
       PROXY             AUTHENTICATION       WANOPT
          |                  |                  |
   +------+------+     +-----+------+      +----+-----+
   |             |     |     |      |      |    |     |
Explicit    Transparent LDAP  FSSO  mTLS   TCP Cache Compression
   |             |     |     |      |      |    |     |
   +------+------+     +-----+------+      +----+-----+
          |                  |                  |
          +------------------+------------------+
                             |
                      PROXY POLICY
                             |
             +---------------+---------------+
             |               |               |
          Web Filter       SSL           Headers
             |           Inspection          |
             |               |               |
             +---------------+---------------+
                             |
                      UPSTREAM PROXY
                             |
                        SD-WAN / WAN
                             |
                          Internet
```

---

## 🚨 Production Pre-Commit Check

Before considering the deployment complete:

* [ ] Architecture documented
* [ ] Traffic flows documented
* [ ] Firewall policies reviewed
* [ ] Proxy policies reviewed
* [ ] Authentication tested
* [ ] DNS tested
* [ ] Routing tested
* [ ] NAT behavior verified
* [ ] Upstream proxy tested
* [ ] Health monitoring tested
* [ ] SSL behavior validated
* [ ] WANOpt peer validated
* [ ] Cache behavior validated
* [ ] Statistics reviewed
* [ ] Security profiles validated
* [ ] Logs reviewed
* [ ] Failure scenarios tested
* [ ] Credentials removed from public documentation
* [ ] Example IPs/domains replaced
* [ ] Certificates/PSKs/private data removed
* [ ] Configuration reviewed before production deployment

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

> **Security Note:** Never commit real usernames, passwords, PSKs, private keys, certificates, internal IP addresses, UUIDs, tenant IDs, domain credentials or other sensitive production information to a public repository.

> **SheynShield Philosophy:**
> **Learn → Configure → Validate → Troubleshoot → Secure → Document**

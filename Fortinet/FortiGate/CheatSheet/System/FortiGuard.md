# FortiGate FortiGuard 

> **FortiGuard Services, Updates, Anycast, FortiManager, IoT Detection, Proxy & Air-Gap**
>
> Practical reference for **FortiOS Administration / NSE4 / NSE7**

---

## 1. FortiGuard — Core Concept

**FortiGuard** provides cloud-based security intelligence and update services for FortiGate.

Common FortiGuard services include:

* Antivirus (AV)
* Intrusion Prevention System (IPS)
* Application Control
* AntiSpam
* Web Filtering
* Web Application Firewall (WAF)
* Botnet / threat intelligence
* IoT device identification
* Security rating / reputation services
* Firmware and security database updates

### Basic Requirement

For normal cloud-based FortiGuard operation:

```text
FortiGate
   |
   | HTTPS / DNS / FortiGuard protocols
   v
FortiGuard Distribution Network (FDN)
```

The FortiGate generally needs Internet connectivity to:

1. Validate its FortiGuard license.
2. Reach FortiGuard services.
3. Download security databases.
4. Receive updated security intelligence.

---

# 2. FortiGuard Packages & Signature Validation

AV and IPS packages are cryptographically signed by Fortinet.

During an update, FortiGate validates the package signature before accepting it.

### Manual Package Security Levels

| Level     | Behavior                      |
| --------- | ----------------------------- |
| `level-0` | Accept unsigned package       |
| `level-1` | Warn and request confirmation |
| `level-2` | Reject unsigned package       |

If no level is configured, the effective behavior is generally **Level 1**.

> ⚠️ Security levels are preconfigured on the BIOS on supported platforms.

### Important

```text
FortiGuard Package
       |
       v
Signature Validation
       |
       +---- Valid ----> Accept
       |
       +---- Invalid --> Security-level dependent behavior
```

---

# 3. FortiGuard Automatic Updates

FortiGuard automatic update behavior can operate at intervals determined by the FortiGuard update mechanism and contract state.

Typical automatic update intervals can range from approximately:

```text
10 minutes
      |
      v
...
      |
      v
60 minutes
```

### Configuration

```cli
config system autoupdate schedule
    set status enable
    set frequency automatic
end
```

### Contract-Based Update Frequency

The update interval can be influenced by the percentage of remaining valid contract time.

Conceptually:

```text
More remaining contract
        |
        v
Shorter update interval
        |
        v
More frequent updates
```

Example from the training scenario:

```text
~70% contract remaining
        -> ~10 minute interval

~50% contract remaining
        -> ~20 minute interval
```

> **Exam Tip:** Do not assume that every FortiGate always uses exactly the same update interval. Hardware/platform, service state, FortiOS version and FortiGuard policy can affect behavior.

---

# 4. Immediately Download Updates

On supported hardware/platforms, FortiGate can maintain a persistent secure connection to FortiGuard when the feature is available.

Conceptually:

```text
FortiGate
   |
   | Persistent HTTPS connection
   v
FortiGuard
   |
   | New update notification
   v
fds_notify
   |
   | Download request
   v
FortiGuard Update Server
```

The `fds_notify` daemon waits for update notifications and then initiates the update download.

> This feature availability is platform/version dependent.

---

# 5. Improve IPS Quality

The **Improve IPS Quality** option allows FortiGate to send attack-related information to FortiGuard.

Purpose:

```text
Attack detected
      |
      v
FortiGate
      |
      | Security telemetry
      v
FortiGuard
      |
      v
Threat intelligence / signature improvement
```

The objective is to help FortiGuard improve IPS signatures as attacks evolve.

---

# 6. Antivirus PUP / PUA Detection

PUP/PUA detection allows FortiGate Antivirus functionality to identify:

* Potentially Unwanted Applications
* Grayware
* Other potentially unwanted software

This can provide an additional detection layer beyond conventional malware signatures.

---

# 7. FortiGuard Update Through Proxy

FortiGate can use a proxy/tunnel for certain FortiGuard update operations.

Example:

```cli
config system autoupdate tunneling
    set address 1.2.3.4
    set port 1344
    set username test
    set password 123
    set status enable
end
```

### Important Limitation

Proxy tunneling is supported only for specific services, including:

* Registration
* Antivirus updates
* IPS updates

For FortiGate VMs, proxy tunneling can additionally be used for license validation.

---

# 8. Web Filter / AntiSpam Proxy Limitation

A critical distinction:

```text
AV / IPS Update
       |
       +---- Proxy tunneling supported

Web Filtering / Spam Rating
       |
       +---- UDP-based communication
```

Web filtering and spam-filtering services may use UDP communication such as:

```text
UDP/53
UDP/8888
```

UDP traffic cannot simply be redirected through a traditional proxy.

Therefore, even if FortiOS supports HTTPS-based FortiGuard communication for some services, do not assume **all FortiGuard traffic** can traverse the same proxy.

---

# 9. Closed Network — FortiManager as Local FortiGuard

For environments without direct Internet access, **FortiManager** can provide local update and rating services.

Architecture:

```text
                 Internet
                    |
                    v
              FortiManager
             Local Services
              /          \
             /            \
       Update              Rating
       AV/IPS              Web/Spam
          \                  /
           \                /
              FortiGate
```

This is particularly useful in:

* Closed networks
* Restricted networks
* OT environments
* Data centers
* Air-gapped or partially isolated environments

---

# 10. FortiManager — Update & Rating Servers

Example:

```cli
config system central-management
    set type fortimanager
    set fmg 192.168.254.200

    config server-list
        edit 1
            set server-type update
            set server-address 192.168.254.200
        next

        edit 2
            set server-type rating
            set server-address 192.168.254.200
        next
    end

    set fmg-update-port 443
    set include-default-servers enable
end
```

### Server Types

| Server Type | Typical Purpose                      |
| ----------- | ------------------------------------ |
| `update`    | AV / IPS and update-related services |
| `rating`    | Web filtering / AntiSpam rating      |

### Fallback

```cli
set include-default-servers enable
```

This allows FortiGate to retain default FortiGuard servers as fallback when configured FortiManager connectivity is unavailable.

> Older FortiOS/FortiManager implementations may use different ports or mechanisms. Always verify the target release.

---

# 11. FortiGuard Anycast

FortiGuard can use **Anycast** to improve global connectivity and reduce latency.

### Anycast Concept

```text
                    FortiGuard
                 Global Anycast IP
                         |
          +--------------+--------------+
          |              |              |
        Region A       Region B       Region C
          |              |              |
       FortiGate       FortiGate       FortiGate
```

Instead of manually selecting a distant FortiGuard server, DNS/Anycast routing can direct the FortiGate toward an appropriate service location.

### Common Anycast FQDNs

```text
globalupdate.fortinet.net
globalguardservice.fortinet.net
```

### Non-Anycast Examples

```text
update.fortiguard.net
service.fortiguard.net
securewf.fortiguard.net
```

---

# 12. Regional FortiGuard Servers

### USA

```text
usupdate.fortinet.net
usguardservice.fortinet.net
```

Non-Anycast examples:

```text
usupdate.fortiguard.net
usservice.fortiguard.net
ussecurewf.fortiguard.net
```

### Europe

```text
euupdate.fortinet.net
euguardservice.fortinet.net
```

### Configuration

```cli
config system fortiguard
    set update-server-location automatic
end
```

Possible location behavior can include:

```text
automatic
usa
eu
```

> `automatic` is generally associated with selecting an appropriate FortiGuard location, commonly using Anycast where supported.

---

# 13. FortiGuard Protocol & Ports

Example:

```cli
config system fortiguard
    set protocol https
    set port 443
end
```

Depending on the FortiGuard service and FortiOS release, other ports/protocols may be involved.

Commonly encountered:

```text
HTTPS  443
DNS    53
UDP    8888
```

### Troubleshooting Principle

Do not test FortiGuard connectivity by checking only TCP/443.

A complete troubleshooting process should consider:

```text
DNS
 |
 +--> FortiGuard hostname resolution
 |
Network
 |
 +--> Routing
 |
Firewall
 |
 +--> Required ports
 |
TLS
 |
 +--> Certificate / trust
 |
FortiGuard
 |
 +--> Service/license availability
```

---

# 14. FortiGuard Cache

FortiGate can cache certain FortiGuard results locally.

### Web Filter Cache

The cache can store web filtering results locally for a configurable period.

Concept:

```text
Client
  |
  v
FortiGate
  |
  +---- Cache hit ----> Local decision
  |
  +---- Cache miss ---> FortiGuard
```

Example:

```cli
config system fortiguard
    set webfilter-cache enable
    set webfilter-cache-ttl 3600
    set webfilter-timeout 15
end
```

### AntiSpam Cache

```cli
config system fortiguard
    set antispam-cache enable
    set antispam-cache-ttl 1800
    set antispam-cache-mpercent 2
    set antispam-timeout 7
end
```

> Cache values and supported options are release dependent.

---

# 15. Force-Off Options

FortiGuard services can have force-off controls.

Example:

```cli
config system fortiguard
    set antispam-force-off disable
    set webfilter-force-off disable
end
```

Conceptually:

```text
force-off enable
      |
      v
Service disabled

force-off disable
      |
      v
Service available
```

---

# 16. Outbreak Prevention Cache

Example configuration:

```cli
config system fortiguard
    set outbreak-prevention-force-off disable
    set outbreak-prevention-cache enable
    set outbreak-prevention-cache-ttl 300
    set outbreak-prevention-cache-mpercent 2
    set outbreak-prevention-timeout 7
end
```

The cache helps reduce repeated external lookups.

---

# 17. SDNS / FortiGuard DNS Service

Example:

```cli
config system fortiguard
    set sdns-server-ip 208.91.112.220
    set sdns-server-port 53
end
```

SDNS-related communication can be important for FortiGuard DNS/security services.

---

# 18. FortiGuard Malware Statistics

FortiGate can periodically send encrypted security statistics to FortiGuard.

Potential telemetry includes statistics related to:

* Antivirus
* IPS
* Botnet IP lists
* Application Control

The information can include device-related metadata such as:

* FortiGate IP address
* Serial number
* Country

The purpose is to improve Fortinet threat intelligence and security services.

### Default Reporting Interval

The notes specify:

```text
60 minutes
```

### Configuration

```cli
config system global
    set fds-statistics enable
    set fds-statistics-period 60
end
```

---

# 19. Malware Signature Lifecycle

FortiGuard can use submitted malware statistics to identify active threats.

Conceptually:

```text
New malware activity
        |
        v
FortiGate telemetry
        |
        v
FortiGuard analysis
        |
        v
Signature activity evaluation
        |
        +---- Active ----> Active AV database
        |
        +---- Inactive --> Extended AV database
```

If activity for an inactive threat reappears, its signature can be promoted back into the active database.

---

# 20. FortiGate ↔ FortiGuard TLS Authentication

FortiGate and FortiGuard establish a secure TLS connection before exchanging protected data.

Conceptually:

```text
FortiGate                         FortiGuard
   |                                  |
   |---- TLS ClientHello ------------>|
   |                                  |
   |<--- ServerHello + Certificate ---|
   |                                  |
   |---- Certificate / Key Info ----->|
   |                                  |
   |<--- Certificate Verify ----------|
   |<--- Finished --------------------|
   |                                  |
   |---- Finished ------------------->|
   |                                  |
   |<==== Encrypted Communication ===>|
```

The certificates must belong to a trusted Fortinet certificate chain.

---

# 21. FortiGuard Certificate Validation

During TLS validation, FortiGate evaluates the server identity and certificate status.

Important failure conditions include:

```text
Certificate CN/SAN mismatch
        |
        v
DNS-resolved domain != certificate identity
        |
        v
TLS handshake aborted
```

Other abort conditions can include:

* Invalid OCSP status
* Revoked issuer CA
* Untrusted certificate chain

---

# 22. OCSP & FortiGuard Anycast

FortiGuard connectivity can use certificate status validation such as **OCSP**.

Conceptually:

```text
FortiGate
   |
   | ClientHello
   | + OCSP status request
   v
FortiGuard
   |
   | Certificate + OCSP status
   v
FortiGate
   |
   +---- Valid ----> Continue TLS
   |
   +---- Invalid --> Abort
```

> Exact certificate-validation behavior can vary by FortiOS release and service.

---

# 23. FortiGuard Anycast Source

Example:

```cli
config system fortiguard
    set fortiguard-anycast enable
    set fortiguard-anycast-source fortinet
end
```

Depending on the FortiOS release, alternative source options may be available.

---

# 24. FortiManager vs FortiGuard Anycast

A useful conceptual distinction:

| Feature                         | Direct FortiGuard | Local FortiManager           |
| ------------------------------- | ----------------- | ---------------------------- |
| Anycast                         | Available         | Local architecture           |
| Internet dependency             | Usually yes       | Can reduce direct dependency |
| AV/IPS updates                  | Yes               | Yes                          |
| Rating services                 | Yes               | Yes                          |
| Local control                   | Lower             | Higher                       |
| Closed environment              | Limited           | Useful                       |
| Centralized update distribution | Limited           | Strong                       |

For large environments:

```text
FortiGuard
     |
     v
FortiManager
     |
     +---- FGT-01
     +---- FGT-02
     +---- FGT-03
     +---- FGT-04
```

---

# 25. FortiGuard Communication Troubleshooting

### Service Communication

```cli
diagnose sys service-communication
```

Useful for inspecting FortiGuard/service communication status.

### Autoupdate Debug

```cli
diagnose debug application updated -1
diagnose debug enable
```

Use this when troubleshooting update behavior.

Remember to stop debugging after troubleshooting:

```cli
diagnose debug disable
```

---

# 26. License & FortiGuard Status

Check:

```text
System
  >
FortiGuard
  >
License
```

Also inspect:

```text
Log & Report
  >
System Events
  >
General System Events
```

Cloud/service communication logging can also help identify:

* License problems
* FortiGuard connectivity issues
* Update failures
* Authentication problems

---

# 27. FortiGuard Online Security Tools

FortiGuard Labs provides online lookup resources.

### Web Filter Lookup

Use it to determine:

* URL category
* URL rating
* Classification

It can also be used to request reevaluation of incorrectly categorized URLs.

[FortiGuard Web Filter Lookup](https://www.fortiguard.com/webfilter?utm_source=chatgpt.com)

### Threat Encyclopedia

Useful for researching:

* Viruses
* Botnet C&C
* IPS signatures
* Vulnerabilities
* Mobile malware

[FortiGuard Threat Encyclopedia](https://www.fortiguard.com/encyclopedia?utm_source=chatgpt.com)

### Application Control

Search application signatures and classifications.

[FortiGuard Application Control](https://www.fortiguard.com/appcontrol?utm_source=chatgpt.com)

---

# 28. IoT Detection Service

FortiGuard IoT Detection is a subscription-based service that can help identify devices not completely identified by the local device database.

Architecture:

```text
Unknown Device
      |
      v
FortiGate Device Detection
      |
      v
Local CIDB
      |
      +---- Known ----> Device identified
      |
      +---- Unknown --> FortiGuard
                            |
                            v
                       Device analysis
                            |
                            v
                       Result returned
```

### Requirements

The FortiGate should:

* Be registered with FortiCare.
* Have an appropriate IoT Detection license.
* Be connected to a supported FortiGuard Anycast service.
* Have device detection enabled on the relevant interface.

---

# 29. IoT Detection Data Flow

```text
Device
  |
  | DHCP / MAC / HTTP / traffic characteristics
  v
FortiGate
  |
  | Device information
  v
FortiGuard Collection Server
  |
  v
FortiGuard Query Server
  |
  v
Device identification
  |
  v
FortiGate Device Inventory
```

View results from:

```text
Dashboard
  >
Users & Devices
  >
Device Inventory
```

---

# 30. Force IoT Queries to FortiGuard

For troubleshooting/testing:

```cli
diagnose cid sigs disable
```

This disables the local device signature database so FortiGate can query FortiGuard for device identification.

### IoT Debug

```cli
diagnose debug application iotd -1
diagnose debug enable
```

### Device List

```cli
diagnose user device list
```

---

# 31. FortiAP IoT Device Identification

FortiAP can collect information about connected devices and provide it to FortiGate for device identification.

Potential data sources include:

```text
DHCP
MAC address
HTTP
Device behavior
Traffic characteristics
```

Architecture:

```text
Client Device
      |
      v
    FortiAP
      |
      v
  FortiGate
      |
      v
 FortiGuard IoT
      |
      v
Device intelligence
      |
      v
  FortiGate
```

---

# 32. FortiAP Device Weight

Example:

```cli
config wireless-controller setting
    edit ap-1
        set device-weight 1
        set device-holdoff 5
        set device-idle 1440
    end
end
```

### Device Weight

The notes describe weighting of different device-identification sources.

Conceptually:

```text
FortiGuard intelligence
        +
DHCP information
        +
Behavioral analysis
        |
        v
Confidence score
        |
        v
Device classification
```

> Exact scoring and supported ranges are version/platform dependent. Verify the CLI reference for the target FortiOS release.

### Device Holdoff

Controls how long FortiGate waits before sending device information toward FortiGuard.

Example:

```text
device-holdoff = 5 minutes
```

### Device Idle

Controls how long inactive device information remains in the table.

Example:

```text
device-idle = 1440 minutes
```

---

# 33. FortiGate Cloud / FortiGuard Through Explicit Proxy

FortiGate can use an explicit proxy for certain FortiCloud/FortiGuard communication.

Concept:

```text
FortiGate
    |
    | Proxy authentication
    v
Explicit Proxy
    |
    v
Internet
    |
    v
FortiCloud / FortiGuard
```

> ⚠️ Not every FortiGuard service supports these proxy settings. Web Filter service traffic, in particular, may follow a different communication mechanism.

---

# 34. Example — FortiGate as Explicit Proxy

### Proxy Policy

```cli
config firewall proxy-policy
    edit 1
        set proxy explicit-web
        set dstintf port1
        set srcaddr all
        set dstaddr all
        set service webproxy
        set action accept
        set schedule always
        set logtraffic all
        set users guest1
    next
end
```

### Local User

```cli
config user local
    edit guest1
        set type password
        set passwd <PASSWORD>
    next
end
```

### Authentication Scheme

```cli
config authentication scheme
    edit local-basic
        set method basic
        set user-database local-user-db
    next
end
```

### Authentication Rule

```cli
config authentication rule
    edit local-basic-rule
        set srcaddr all
        set ip-based disable
        set active-auth-method local-basic
    next
end
```

### Firewall Policy

```cli
config firewall policy
    edit 1
        set srcintf port3
        set dstintf port1
        set srcaddr all
        set dstaddr all
        set action accept
        set schedule always
        set service dns
        set fsso disable
        set nat enable
    end
end
```

---

# 35. FortiGate Using the Proxy

On the FortiGate using the proxy:

```cli
config system fortiguard
    set proxy-server-ip 192.168.254.251
    set proxy-server-port 8080
    set proxy-username guest1
    set proxy-password <PASSWORD>
end
```

Test FortiCloud/FortiGuard-related communication as appropriate for the target FortiOS release.

> 🔐 Never place real production credentials inside shared documentation or GitHub repositories.

---

# 36. ISDB & FortiGuard

FortiOS firmware images include Fortinet Internet Service Database (ISDB) information.

Depending on the image/update state, the built-in ISDB can be:

```text
Partial / lightweight ISDB
        |
        v
Core Fortinet services

Full ISDB
        |
        v
Large database with current service objects
```

The lightweight database helps policies and policy routes referencing Fortinet services continue functioning after a firmware upgrade before the full database is retrieved.

---

# 37. Check Internet Service Database

```cli
diagnose firewall internet-service list
```

This displays ISDB entries.

### Example Policy

```cli
config firewall policy
    edit 1
        set srcintf port1
        set dstintf port3
        set srcaddr all
        set dstaddr all
        set internet-service enable
        set internet-service-id 1245187 1245326 1245324 1245325
        set action accept
        set schedule always
        set logtraffic all
        set fsso disable
    end
end
```

> ISDB IDs are release/database dependent. Do not assume an ID has the same meaning across FortiOS versions.

---

# 38. ISDB Automatic Refresh After Firmware Upgrade

After a firmware upgrade/reboot, FortiGate can perform an automatic ISDB update after startup.

Conceptually:

```text
Firmware Upgrade
       |
       v
FortiGate Reboot
       |
       v
Initial built-in ISDB
       |
       v
Automatic update
       |
       v
Current ISDB
```

The notes specify an approximate startup update delay of:

```text
~5 minutes
```

Check:

```cli
diagnose autoupdate versions | grep internet -a 6
```

---

# 39. Air-Gapped / Offline FortiGate Licensing

Air-gapped environments may not permit direct Internet access.

Common examples:

* OT networks
* Industrial environments
* Critical infrastructure
* Isolated data centers

Architecture:

```text
Connected Environment
        |
        | Download license/package
        v
Offline Transfer
        |
        v
Air-Gapped FortiGate
```

FortiGuard packages such as AV and IPS may need to be manually transferred.

Licensing can also be manually transferred on supported hardware appliances.

> According to the supplied training notes, manual licensing for air-gapped environments is supported for supported FortiGate hardware appliances running FortiOS 7.2.0 or later, but not FortiGate VM appliances. Verify current platform support before deployment.

---

# 40. Manual License Import

Example:

```cli
execute restore manual-license ftp a.lic 192.168.20.200
```

The exact transfer method and syntax should be checked against the target FortiOS release.

---

# 41. Manual AV / IPS Package Update

Example:

```cli
execute restore ips tftp nids-720-19.261.pkg 192.168.20.200
```

Then debug update processing:

```cli
diagnose debug application updated -1
diagnose debug enable
```

After troubleshooting:

```cli
diagnose debug disable
```

---

# 42. FortiGuard Air-Gap Example

A FortiGate in a restricted environment might use:

```cli
config system fortiguard
    set protocol https
    set port 443
    set load-balance-servers 1
    set auto-join-forticloud disable
    set update-server-location usa
end
```

The exact configuration depends on the architecture and whether any local update infrastructure exists.

---

# 43. FortiGuard Update Database Options

The following options may appear in `config system fortiguard` depending on FortiOS version:

```cli
set update-ffdb enable
set update-uwdb enable
set update-extdb enable
set update-build-proxy enable
```

### FFDB

Used for FortiGuard/FortiOS database update functionality.

### UWDB

Unified Weight Database.

Can contribute to threat weighting and security intelligence.

### Extended Database

Provides extended security database information.

### Build Proxy

Can be relevant when using an explicit web proxy for supported update workflows.

> **Version Note:** Do not blindly copy all options into production. CLI availability and behavior are FortiOS-release dependent.

---

# 44. FortiGuard Web Filter Cache

Example:

```cli
config system fortiguard
    set webfilter-force-off disable
    set webfilter-cache enable
    set webfilter-cache-ttl 3600
    set webfilter-timeout 15
end
```

Concept:

```text
URL Request
    |
    v
FortiGate
    |
    +---- Cached result ----> Apply rating
    |
    +---- No cache ---------> FortiGuard
                                  |
                                  v
                              Rating result
```

---

# 45. Complete FortiGuard Configuration Reference

A consolidated example:

```cli
config system fortiguard
    set protocol https
    set port 443

    set load-balance-servers 1

    set update-server-location automatic

    set update-ffdb enable
    set update-uwdb enable
    set update-extdb enable
    set update-build-proxy enable

    set antispam-force-off disable
    set antispam-cache enable
    set antispam-cache-ttl 1800
    set antispam-cache-mpercent 2
    set antispam-timeout 7

    set outbreak-prevention-force-off disable
    set outbreak-prevention-cache enable
    set outbreak-prevention-cache-ttl 300
    set outbreak-prevention-cache-mpercent 2
    set outbreak-prevention-timeout 7

    set webfilter-force-off disable
    set webfilter-cache enable
    set webfilter-cache-ttl 3600
    set webfilter-timeout 15

    set sdns-server-ip 208.91.112.220
    set sdns-server-port 53

    set source-ip 0.0.0.0
end
```

> **Important:** Treat this as a reference template, not a universal copy/paste configuration.

---

# 46. FortiGuard Decision Tree

```text
                    FortiGuard Problem
                           |
                           v
                    Is license valid?
                      /          \
                    NO            YES
                    |              |
              Fix licensing        v
                              DNS resolution?
                              /          \
                            NO            YES
                            |              |
                       Fix DNS             v
                                    Routing working?
                                      /       \
                                    NO         YES
                                    |           |
                               Fix routing      v
                                         Required ports?
                                          /          \
                                        NO            YES
                                        |              |
                                   Allow traffic       v
                                                TLS validation?
                                                 /        \
                                               NO          YES
                                               |             |
                                      Check certificates     v
                                                        Service issue?
                                                         /       \
                                                       YES       NO
                                                       |          |
                                              Debug service      Check
                                              communication     FortiGuard
```

---

# 47. High-Value Troubleshooting Commands

| Purpose                          | Command                                   |
| -------------------------------- | ----------------------------------------- |
| FortiGuard/service communication | `diagnose sys service-communication`      |
| Autoupdate debugging             | `diagnose debug application updated -1`   |
| Enable debug                     | `diagnose debug enable`                   |
| Disable debug                    | `diagnose debug disable`                  |
| ISDB entries                     | `diagnose firewall internet-service list` |
| Update versions                  | `diagnose autoupdate versions`            |
| IoT daemon debug                 | `diagnose debug application iotd -1`      |
| Device list                      | `diagnose user device list`               |
| Disable local CIDB signatures    | `diagnose cid sigs disable`               |

---

# 48. FortiGuard Troubleshooting Checklist

### Connectivity

* [ ] Default route exists.
* [ ] DNS works.
* [ ] FortiGate can reach the Internet.
* [ ] Required FortiGuard ports are allowed.
* [ ] Upstream firewall is not blocking FortiGuard.
* [ ] Proxy settings are correct if used.

### License

* [ ] FortiGate is registered.
* [ ] FortiGuard contract is valid.
* [ ] Required subscription exists.
* [ ] System time is correct.

### TLS

* [ ] Certificate chain is trusted.
* [ ] DNS resolves to the expected service.
* [ ] No TLS interception is breaking validation.
* [ ] Required TLS versions are supported.

### Updates

* [ ] Automatic update is enabled.
* [ ] FortiGuard location is appropriate.
* [ ] Update server is reachable.
* [ ] AV/IPS package signature validates.
* [ ] Update debug has no errors.

### FortiManager

* [ ] FortiGate can reach FortiManager.
* [ ] Update server is configured.
* [ ] Rating server is configured.
* [ ] Correct port is reachable.
* [ ] Fallback behavior is understood.

---

# 49. NSE Exam — Must Remember

> ### 🔥 FortiGuard Mental Model

```text
FortiGuard
   |
   +-- AV Updates
   |
   +-- IPS Updates
   |
   +-- Web Rating
   |
   +-- AntiSpam
   |
   +-- Application Intelligence
   |
   +-- IoT Intelligence
   |
   +-- Threat Intelligence
   |
   +-- Security Databases
   |
   +-- Cloud Services
```

### High-Value Facts

| Topic              | Remember                                                                                     |
| ------------------ | -------------------------------------------------------------------------------------------- |
| AV/IPS packages    | Digitally signed by Fortinet CA                                                              |
| Anycast            | Helps reach an appropriate FortiGuard service location                                       |
| Web/Spam           | Can use communication mechanisms that are not proxy-friendly                                 |
| FortiManager       | Can act as a local FortiGuard update/rating source                                           |
| Air-Gap            | Requires offline/manual update/licensing workflows                                           |
| IoT Detection      | Uses FortiGuard intelligence when local identification is insufficient                       |
| ISDB               | Built-in database allows FortiGate policies to continue functioning during update transition |
| Malware statistics | Used to improve Fortinet threat intelligence                                                 |
| TLS                | Certificate/trust validation is part of secure communication                                 |
| Cache              | Reduces repeated FortiGuard lookups                                                          |
| Diagnostics        | `diagnose sys service-communication` is a key starting point                                 |

---

# 50. Quick Command Card

```cli
# FortiGuard configuration
config system fortiguard
    set protocol https
    set port 443
    set update-server-location automatic
end
```

```cli
# Automatic updates
config system autoupdate schedule
    set status enable
    set frequency automatic
end
```

```cli
# Update tunneling
config system autoupdate tunneling
    set address <PROXY-IP>
    set port <PORT>
    set username <USERNAME>
    set password <PASSWORD>
    set status enable
end
```

```cli
# FortiGuard diagnostics
diagnose sys service-communication
```

```cli
# Update debug
diagnose debug application updated -1
diagnose debug enable

# Stop debugging
diagnose debug disable
```

```cli
# IoT diagnostics
diagnose debug application iotd -1
diagnose debug enable

diagnose user device list
```

```cli
# ISDB
diagnose firewall internet-service list

diagnose autoupdate versions | grep internet -a 6
```

---

# 51. Final Architecture

```text
                           ┌─────────────────────┐
                           │    FortiGuard Labs   │
                           │ Threat Intelligence  │
                           └──────────┬──────────┘
                                      |
                              FortiGuard Network
                                      |
                 ┌────────────────────┼────────────────────┐
                 |                    |                    |
              Anycast              Regional           FortiManager
                 |                 Servers              Local FDN
                 |                    |                    |
                 └────────────────────┼────────────────────┘
                                      |
                                  FortiGate
                                      |
       ┌──────────────┬───────────────┼───────────────┬──────────────┐
       |              |               |               |              |
      AV             IPS          Web Filter       AntiSpam         IoT
       |              |               |               |              |
       └──────────────┴───────────────┼───────────────┴──────────────┘
                                      |
                               Security Decision
```

---

## 🎯 One-Line Takeaway

> **FortiGuard is not simply an “update server”; it is a distributed security-intelligence ecosystem that provides signatures, ratings, threat intelligence, device intelligence, and security databases through multiple communication models such as Anycast, direct connectivity, proxy-based access, and FortiManager-local services.**

---

### 🔎 SEO / GitHub Keywords

`FortiGuard` · `FortiGate FortiGuard` · `FortiOS FortiGuard` · `FortiGuard Anycast` · `FortiGuard troubleshooting` · `FortiGuard update` · `FortiManager FortiGuard` · `FortiGate IoT Detection` · `FortiGate ISDB` · `FortiGate air gap` · `FortiGate offline licensing` · `FortiGuard proxy` · `FortiGuard AV IPS update` · `FortiGuard Labs` · `NSE4` · `NSE7` · `FortiGate CLI` · `Fortinet Security`

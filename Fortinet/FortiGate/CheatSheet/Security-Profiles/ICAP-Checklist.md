# 🔗 SheynShield Resources

# FortiGate ICAP — Security & Content Inspection Checklist

> **FortiOS Focus:** ICAP · REQMOD · RESPMOD · Proxy Inspection · SSL Inspection · Content Adaptation · DLP · Antivirus · Content Filtering · Selective Forwarding · WAD · Troubleshooting
> **Audience:** NSE4 / NSE7 · Network Security Engineers · Security Architects
> **Protocol:** Internet Content Adaptation Protocol (ICAP)
> **Common ICAP Port:** TCP/1344
> **Secure ICAP:** Verify configured secure ICAP port for the FortiOS/version and ICAP server
> **FortiGate Component:** WAD / Proxy Engine
> **Primary Principle:** `Selective Inspection > Inspect Everything`

---

## 📋 ICAP Deployment Checklist

### 1. Architecture & Design

* [ ] Define why ICAP is required.
* [ ] Identify the external ICAP inspection engine.
* [ ] Identify whether ICAP will process **requests**, **responses**, or both.
* [ ] Determine whether HTTPS traffic must be inspected.
* [ ] Determine whether SSL inspection/decryption is required.
* [ ] Define which applications require ICAP.
* [ ] Define which hosts require ICAP.
* [ ] Define which HTTP methods require REQMOD.
* [ ] Define which response codes require RESPMOD.
* [ ] Define which `Content-Type` values require inspection.
* [ ] Define whether file-transfer inspection is required.
* [ ] Define whether streaming content should bypass ICAP.
* [ ] Define ICAP failure behavior.
* [ ] Define ICAP server capacity.
* [ ] Define expected concurrent connections.
* [ ] Define latency requirements.
* [ ] Define ICAP server redundancy requirements.
* [ ] Define monitoring and alerting requirements.

### Architecture Mental Model

```text
Client
   │
   ▼
FortiGate
   │
   ├── Proxy Processing
   │
   ├── SSL Inspection
   │
   ├── Content Selection
   │
   ▼
ICAP Server
   │
   ├── Antivirus
   ├── DLP
   ├── Content Filtering
   ├── Malware Inspection
   └── Content Adaptation
   │
   ▼
FortiGate
   │
   ├── IPS
   ├── Web Filter
   ├── Application Control
   └── Other Security Controls
   │
   ▼
Client
```

---

# 2. ICAP Fundamentals

* [ ] Understand that ICAP stands for **Internet Content Adaptation Protocol**.
* [ ] Understand that ICAP allows a proxy to delegate content adaptation/inspection to an external server.
* [ ] Understand that FortiGate acts as the ICAP client in this architecture.
* [ ] Understand that the external ICAP server performs the requested adaptation/inspection.
* [ ] Understand the difference between `REQMOD` and `RESPMOD`.
* [ ] Understand the role of ICAP in proxy-based inspection.
* [ ] Understand that ICAP does not automatically mean all traffic is forwarded.
* [ ] Understand selective ICAP forwarding.
* [ ] Understand ICAP failure behavior.
* [ ] Understand the impact of ICAP on latency and resources.

### Memory

```text
ICAP
  ↓
External Content Adaptation

REQMOD  → Request
RESPMOD → Response
```

---

# 3. ICAP Server Checklist

* [ ] Confirm ICAP server hostname/IP.
* [ ] Confirm IP version.
* [ ] Confirm ICAP listening port.
* [ ] Confirm secure ICAP requirements.
* [ ] Confirm ICAP service is running.
* [ ] Confirm ICAP service is listening.
* [ ] Confirm FortiGate can route to the ICAP server.
* [ ] Confirm intermediate firewalls allow ICAP traffic.
* [ ] Confirm ICAP server capacity.
* [ ] Confirm maximum concurrent connections.
* [ ] Confirm ICAP server CPU capacity.
* [ ] Confirm ICAP server memory capacity.
* [ ] Confirm ICAP inspection engine/module status.
* [ ] Confirm ICAP service supports the required ICAP method.
* [ ] Confirm required ICAP service/path exists.
* [ ] Confirm health-check endpoint/service is available.

### Typical Port

```text
Standard ICAP
TCP/1344
```

> ⚠️ Do not assume a universal secure-ICAP port. Verify the configured port and FortiOS/version-specific documentation.

---

# 4. FortiGate ICAP Feature Visibility

* [ ] Log in to FortiGate.
* [ ] Open **System → Feature Visibility**.
* [ ] Locate ICAP.
* [ ] Enable ICAP if required.
* [ ] Confirm ICAP configuration objects are visible.
* [ ] Confirm the FortiOS version supports the required ICAP features.

---

# 5. ICAP Server Configuration

Example baseline:

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

### Validate

* [ ] `ip-version` is correct.
* [ ] `ip-address` is correct.
* [ ] `port` matches the ICAP server.
* [ ] `max-connections` is appropriate.
* [ ] Secure ICAP is enabled only when required.
* [ ] Required certificate is available.
* [ ] Certificate configuration is valid.
* [ ] Health check is enabled where appropriate.
* [ ] Health-check server is reachable.
* [ ] ICAP server accepts connections from FortiGate.

---

# 6. Network Connectivity

### FortiGate → ICAP

* [ ] Routing table contains a route to the ICAP server.
* [ ] Correct VDOM is being used.
* [ ] Correct source interface/path is available.
* [ ] Intermediate firewall allows ICAP.
* [ ] TCP/1344 is permitted when using standard ICAP.
* [ ] Secure ICAP port is permitted when applicable.
* [ ] ICAP server local firewall permits FortiGate.
* [ ] ICAP server is listening.

### Connectivity Test

```cli
execute telnet 192.168.20.200 1344
```

Expected:

```text
Connection successful
```

### If Connectivity Fails

* [ ] Verify ICAP server IP.
* [ ] Verify routing.
* [ ] Verify VDOM routing.
* [ ] Verify interface status.
* [ ] Verify intermediate firewall rules.
* [ ] Verify server firewall.
* [ ] Verify listening socket.
* [ ] Verify ICAP service.
* [ ] Verify configured port.
* [ ] Verify secure/non-secure ICAP mismatch.

---

# 7. ICAP Profile

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

### Profile Validation

* [ ] ICAP profile exists.
* [ ] Correct ICAP server is selected.
* [ ] Request inspection is enabled if required.
* [ ] Response inspection is enabled if required.
* [ ] Correct REQMOD service/path is configured.
* [ ] Correct RESPMOD service/path is configured.
* [ ] HTTP methods are appropriate.
* [ ] Failure behavior is explicitly defined.
* [ ] Streaming bypass is intentionally configured.
* [ ] Logging behavior is configured.
* [ ] 204 behavior is understood.
* [ ] Forwarding rules are reviewed.

---

# 8. REQMOD Checklist

**REQMOD = Request Modification**

```text
Client
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
   ├── Allow
   ├── Modify
   └── Block
   │
   ▼
Web Server
```

### Validate

* [ ] Request inspection is enabled.
* [ ] Correct request ICAP server is configured.
* [ ] Correct request path/service is configured.
* [ ] Required HTTP methods are selected.
* [ ] Unsupported/unnecessary methods are excluded where appropriate.
* [ ] Request failure behavior is defined.
* [ ] REQMOD service supports REQMOD.
* [ ] Request logging is available for troubleshooting.

### Methods

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

---

# 9. RESPMOD Checklist

**RESPMOD = Response Modification**

```text
Web Server
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

### Validate

* [ ] Response inspection is enabled.
* [ ] Correct response ICAP server is configured.
* [ ] Correct response path/service is configured.
* [ ] Response failure behavior is defined.
* [ ] Response-header handling is understood.
* [ ] `respmod-default-action` is understood.
* [ ] Forwarding rules are reviewed.
* [ ] HTTP status-code matching is validated.
* [ ] Content-Type matching is validated.

---

# 10. Request vs Response Decision

| Requirement                     | Mechanism         |
| ------------------------------- | ----------------- |
| Inspect client request          | `REQMOD`          |
| Modify client request           | `REQMOD`          |
| Inspect server response         | `RESPMOD`         |
| Modify server response          | `RESPMOD`         |
| Request URL/content policy      | Usually `REQMOD`  |
| Download/content inspection     | Usually `RESPMOD` |
| Response Content-Type filtering | `RESPMOD`         |

### Exam Memory

```text
REQMOD  = Request
RESPMOD = Response
```

---

# 11. Firewall Policy Integration

* [ ] Identify the firewall policy carrying the target traffic.
* [ ] Confirm traffic is processed in the expected inspection mode.
* [ ] Confirm proxy-based processing requirements.
* [ ] Attach the appropriate ICAP profile.
* [ ] Confirm SSL inspection profile where required.
* [ ] Confirm Web Filter requirements.
* [ ] Confirm IPS requirements.
* [ ] Confirm Application Control requirements.
* [ ] Confirm policy order does not bypass the intended policy.
* [ ] Test with a controlled client.

### Typical Design

```text
LAN
 │
 ▼
Firewall Policy
 │
 ├── Proxy Inspection
 ├── SSL Inspection
 ├── ICAP
 ├── IPS
 ├── Web Filter
 └── Application Control
 │
 ▼
WAN
```

---

# 12. Proxy Inspection Validation

* [ ] Confirm the traffic uses the intended proxy-based inspection path.
* [ ] Confirm ICAP is associated with the correct traffic flow.
* [ ] Confirm the policy is actually matching.
* [ ] Confirm the selected content is proxy-visible.
* [ ] Confirm required protocol handling is enabled.
* [ ] Confirm ICAP is not bypassed by another configuration.
* [ ] Confirm WAD is processing the traffic.

---

# 13. HTTPS Inspection Checklist

For encrypted traffic:

```text
HTTPS
  ↓
SSL Inspection
  ↓
Decrypted HTTP Content
  ↓
Content Selection
  ↓
ICAP
```

### Validate

* [ ] SSL inspection is enabled.
* [ ] Correct inspection mode is selected.
* [ ] Required CA certificate is trusted by clients.
* [ ] HTTPS traffic is actually decrypted.
* [ ] ICAP profile is attached.
* [ ] REQMOD/RESPMOD is enabled as required.
* [ ] Content matches forwarding criteria.
* [ ] ICAP server receives the selected content.

### Critical Concept

> If FortiGate cannot obtain the required HTTP payload visibility because traffic remains encrypted, content-based ICAP inspection may not occur.

---

# 14. Selective ICAP Inspection

### Do Not Automatically Inspect Everything

Avoid:

```text
All Web Traffic
      │
      ▼
    ICAP
```

Prefer:

```text
Web Traffic
     │
     ▼
Content Selection
     │
 ┌───┴────────────┐
 ▼                ▼
Required         Normal
Content          Content
 │                │
 ▼                ▼
ICAP             Bypass
```

### Selection Criteria

* [ ] Destination host.
* [ ] Request method.
* [ ] Response status code.
* [ ] Content-Type.
* [ ] File type.
* [ ] Request/response direction.
* [ ] Application requirement.
* [ ] Streaming content.
* [ ] Security sensitivity.

---

# 15. `respmod-default-action`

When selective response forwarding is used:

* [ ] Determine the default action.
* [ ] Decide whether unmatched content should bypass ICAP.
* [ ] Create explicit forwarding rules.
* [ ] Verify rule order.
* [ ] Verify matching conditions.
* [ ] Test matching content.
* [ ] Test non-matching content.

Concept:

```text
HTTP Response
      │
      ▼
Forwarding Rules
      │
 ┌────┴─────┐
 ▼          ▼
Match      No Match
 │          │
 ▼          ▼
ICAP      Default Action
```

---

# 16. Content-Type Filtering

Example:

```text
Content-Type: image/jpeg
```

Possible configuration concept:

```cli
config header-group
    edit 1
        set header-name content-type
        set header image/jpeg
    next
end
```

### Validate

* [ ] Confirm exact header name.
* [ ] Confirm expected Content-Type value.
* [ ] Confirm server actually sends the expected header.
* [ ] Test matching content.
* [ ] Test non-matching content.
* [ ] Test content served with unexpected MIME types.
* [ ] Consider file-type/content-sniffing requirements of the external ICAP engine.

Example:

```text
image/jpeg      → ICAP
application/pdf → ICAP
video/mp4       → Bypass
text/html       → Bypass
```

---

# 17. HTTP Response-Code Filtering

Possible selectors:

```text
200
301
302
```

### Validate

* [ ] Identify response codes requiring inspection.
* [ ] Add required status codes to forwarding rules.
* [ ] Test `200`.
* [ ] Test `301`.
* [ ] Test `302`.
* [ ] Test non-matching response codes.
* [ ] Confirm default action for unmatched responses.

---

# 18. Streaming Content

* [ ] Identify streaming applications.
* [ ] Identify large streaming objects.
* [ ] Determine whether inspection is required.
* [ ] Enable streaming bypass when appropriate.
* [ ] Understand the security trade-off.
* [ ] Test video/audio traffic after configuration.

Example:

```cli
config icap profile
    edit icap-prof-test
        set streaming-content-bypass enable
    next
end
```

### Trade-off

```text
More Inspection
      ↓
More Visibility
      ↓
More Resource Usage

More Bypass
      ↓
Better Performance
      ↓
Less Inspection
```

---

# 19. ICAP Failure Behavior

This is a **critical production design decision**.

### Fail-Open

```text
ICAP Down
   ↓
Bypass
   ↓
Traffic Continues
```

* [ ] Confirm business availability requirement.
* [ ] Understand inspection gap.
* [ ] Confirm bypass is acceptable.

### Fail-Closed

```text
ICAP Down
   ↓
Error / Block
   ↓
Traffic Does Not Continue
```

* [ ] Confirm security requirement.
* [ ] Confirm availability impact.
* [ ] Confirm business continuity requirements.

### Decision Checklist

* [ ] Define acceptable security exposure.
* [ ] Define acceptable downtime.
* [ ] Define critical applications.
* [ ] Define incident response procedure.
* [ ] Test ICAP server failure.
* [ ] Test recovery.
* [ ] Verify actual FortiGate behavior.

---

# 20. ICAP 204 Response

Understand:

```text
204 No Modification Needed
```

Conceptually:

```text
FortiGate
   ↓
ICAP
   ↓
Inspection
   ↓
204
   ↓
No Modification Required
   ↓
Continue Processing
```

### Validate

* [ ] Understand the meaning of 204.
* [ ] Confirm 204 handling in the configured FortiOS/ICAP workflow.
* [ ] Test an ICAP response that requires no modification.
* [ ] Confirm traffic continues as expected.

---

# 21. ICAP Status-Code Exam Checklist

|  Code | Meaning                    |
| ----: | -------------------------- |
| `100` | Continue                   |
| `204` | No Modification Needed     |
| `206` | Partial Content            |
| `400` | Bad Request                |
| `404` | ICAP Service Not Found     |
| `405` | Method Not Allowed         |
| `408` | Request Timeout            |
| `418` | Bad Composition            |
| `500` | Server Error               |
| `501` | Method Not Implemented     |
| `502` | Bad Gateway                |
| `503` | Service Overloaded         |
| `505` | ICAP Version Not Supported |

### Memorize

```text
204 → No Modification
405 → Method Not Allowed
408 → Timeout
503 → Service Overloaded
505 → Version Not Supported
```

---

# 22. File Transfer Inspection

If file-transfer inspection is required:

* [ ] Identify required protocol.
* [ ] Confirm FortiOS support.
* [ ] Confirm required inspection profile.
* [ ] Confirm ICAP server supports the required workflow.
* [ ] Configure the required ICAP file-transfer server.
* [ ] Configure the appropriate service/path.
* [ ] Test small files.
* [ ] Test large files.
* [ ] Test malicious/test files in a controlled lab.
* [ ] Test failure behavior.

### SSH Example

```cli
config icap profile
    edit icap-prof-test
        set file-transfer ssh
        set file-transfer-server icap-test
        set file-transfer-path ssh_test
    next
end
```

### FTP Example

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

---

# 23. C-ICAP Lab Checklist

### Installation

* [ ] Prepare Linux VM.
* [ ] Assign static IP.
* [ ] Configure hostname.
* [ ] Configure DNS if required.
* [ ] Configure routing.
* [ ] Install C-ICAP.
* [ ] Install required modules.
* [ ] Verify package versions.

Ubuntu/Debian example:

```bash
apt update
apt install c-icap libc-icap-modules -y
```

Optional modules:

```bash
apt install libc-icap-mod-contentfiltering
apt install libc-icap-mod-urlcheck
apt install libc-icap-mod-virus-scan
```

> Package names vary by distribution/release. Verify availability before automation.

---

# 24. C-ICAP Configuration

Typical configuration location:

```text
/etc/c-icap/
```

Possible files:

```text
/etc/c-icap/c-icap.conf
/etc/c-icap/urlfilter.conf
/etc/c-icap/srv_url_check.conf
```

### Validate

* [ ] `c-icap.conf` exists.
* [ ] Server bind address is correct.
* [ ] Port is correct.
* [ ] Required modules are enabled.
* [ ] Required services are defined.
* [ ] URL filtering configuration is valid.
* [ ] Service starts without errors.

Example concept:

```text
ServerName 0.0.0.0
Port 1344
```

---

# 25. C-ICAP Service Validation

```bash
systemctl start c-icap
systemctl enable c-icap
systemctl status c-icap
```

### Verify Listening Port

```bash
netstat -tulnp | grep 1344
```

or:

```bash
ss -lntp | grep 1344
```

### Validate

* [ ] Service is running.
* [ ] Service starts automatically.
* [ ] TCP/1344 is listening.
* [ ] Correct process owns the port.
* [ ] Linux firewall permits FortiGate.
* [ ] ICAP modules load correctly.

---

# 26. C-ICAP URL Filtering Lab

Example:

```text
deny ^https?://([a-z0-9]+\.)*example\.ir(/.*)?$

deny ^https?://([a-z0-9]+\.)*example\.ir:[0-9]+(/.*)?$

allow .*
```

### Validate

* [ ] Replace test domain with your own lab domain.
* [ ] Validate regular-expression syntax.
* [ ] Test allowed URL.
* [ ] Test denied URL.
* [ ] Test HTTPS URL where applicable.
* [ ] Check C-ICAP logs.
* [ ] Check FortiGate logs.
* [ ] Confirm ICAP response.

> Regex behavior depends on the exact C-ICAP module/version. Validate in the lab before production use.

---

# 27. Health Check

* [ ] Enable ICAP health checking where appropriate.
* [ ] Configure health-check server.
* [ ] Verify ICAP service responds.
* [ ] Stop ICAP service.
* [ ] Confirm FortiGate detects failure.
* [ ] Start ICAP service.
* [ ] Confirm recovery.
* [ ] Verify traffic behavior during failure.
* [ ] Verify traffic behavior after recovery.

Concept:

```text
FortiGate
    │
    │ Health Check
    ▼
ICAP Server
    │
 ┌──┴──┐
 ▼     ▼
 UP    DOWN
```

---

# 28. ICAP High Availability

For critical environments:

* [ ] Deploy multiple ICAP servers.
* [ ] Ensure servers have equivalent inspection capabilities.
* [ ] Validate health checks.
* [ ] Validate connection distribution.
* [ ] Validate server capacity.
* [ ] Test ICAP #1 failure.
* [ ] Test ICAP #2 failure.
* [ ] Test recovery.
* [ ] Monitor connection distribution.
* [ ] Monitor ICAP server CPU/memory.
* [ ] Monitor FortiGate resources.

Architecture:

```text
                  FortiGate
                      │
             ┌────────┴────────┐
             ▼                 ▼
         ICAP #1            ICAP #2
             │                 │
             └────────┬────────┘
                      ▼
                  Inspection
```

---

# 29. Capacity Planning

Evaluate the complete chain:

```text
FortiGate
    +
WAD
    +
Network
    +
ICAP Server
    +
Inspection Engine
    =
Deployment Capacity
```

### FortiGate

* [ ] CPU capacity.
* [ ] Memory capacity.
* [ ] WAD resource usage.
* [ ] Proxy connection count.
* [ ] Concurrent sessions.
* [ ] SSL inspection load.

### ICAP Server

* [ ] CPU.
* [ ] Memory.
* [ ] Worker/process capacity.
* [ ] Maximum connections.
* [ ] Scan throughput.
* [ ] Large-file handling.
* [ ] Concurrent inspection capacity.

### Network

* [ ] Bandwidth.
* [ ] Latency.
* [ ] Packet loss.
* [ ] MTU/path issues.
* [ ] ICAP server location.

---

# 30. Performance Optimization

* [ ] Use selective ICAP inspection.
* [ ] Avoid unnecessary streaming inspection.
* [ ] Select REQMOD only when required.
* [ ] Select RESPMOD only when required.
* [ ] Filter by Content-Type where appropriate.
* [ ] Filter by response status where appropriate.
* [ ] Limit unnecessary hosts.
* [ ] Limit unnecessary methods.
* [ ] Monitor WAD resources.
* [ ] Monitor ICAP CPU.
* [ ] Monitor ICAP memory.
* [ ] Monitor ICAP connection usage.
* [ ] Measure latency before ICAP deployment.
* [ ] Measure latency after deployment.
* [ ] Test large files.
* [ ] Test high concurrency.

### Optimization Pattern

```text
Selective Forwarding
        +
Streaming Bypass
        +
Content-Type Filtering
        +
Response-Code Filtering
        +
Capacity Planning
        ↓
Lower ICAP Load
        ↓
Better Performance
```

---

# 31. ICAP + Security Profiles

ICAP does not necessarily replace native FortiGate security controls.

Validate whether the architecture requires:

* [ ] SSL Inspection.
* [ ] Web Filter.
* [ ] IPS.
* [ ] Application Control.
* [ ] DLP.
* [ ] Antivirus.
* [ ] Other content controls.
* [ ] Logging.
* [ ] Monitoring.

Possible chain:

```text
Client
  ↓
SSL Inspection
  ↓
ICAP
  ↓
IPS
  ↓
Web Filter
  ↓
Application Control
  ↓
Client
```

> The exact processing order and supported combinations depend on the FortiOS version and inspection architecture. Validate the production design against the target release.

---

# 32. ICAP vs FortiProxy

Remember:

```text
ICAP
  ↓
Protocol / Integration Mechanism
```

versus:

```text
FortiProxy
  ↓
Fortinet Security / Proxy Platform
```

### Exam Concept

* [ ] ICAP is understood as a protocol.
* [ ] FortiProxy is understood as a product/platform.
* [ ] External inspection integration is understood.
* [ ] Native Fortinet proxy functionality is distinguished from ICAP.

---

# 33. Common ICAP Ecosystem

Examples include:

* [ ] C-ICAP.
* [ ] Squid ICAP integration.
* [ ] Third-party security/content engines.
* [ ] File-analysis engines with ICAP support.
* [ ] DLP/content-adaptation systems with ICAP support.

> Always verify current vendor/version support before selecting an ICAP engine for production.

---

# 34. Security Validation

* [ ] Use secure ICAP where required.
* [ ] Validate ICAP certificate configuration.
* [ ] Restrict ICAP server access to authorized FortiGate sources.
* [ ] Restrict unnecessary network access to TCP/1344.
* [ ] Monitor ICAP connections.
* [ ] Monitor ICAP errors.
* [ ] Monitor failed inspection events.
* [ ] Log ICAP blocks where appropriate.
* [ ] Protect the ICAP server itself.
* [ ] Patch the ICAP operating system.
* [ ] Patch the ICAP inspection engine.
* [ ] Validate DLP policy.
* [ ] Validate malware scanning policy.
* [ ] Test bypass behavior.
* [ ] Test fail-open/fail-closed behavior.

---

# 35. Troubleshooting — ICAP Not Reachable

### Symptoms

```text
FortiGate cannot connect to ICAP
```

### Checklist

* [ ] Verify ICAP server IP.
* [ ] Verify route.
* [ ] Verify VDOM.
* [ ] Verify interface.
* [ ] Verify TCP port.
* [ ] Verify server firewall.
* [ ] Verify network firewall.
* [ ] Verify C-ICAP service.
* [ ] Verify listening socket.
* [ ] Verify secure/non-secure mismatch.
* [ ] Verify health check.

### Test

```cli
execute telnet 192.168.20.200 1344
```

---

# 36. Troubleshooting — HTTP Works but ICAP Is Not Triggered

* [ ] ICAP feature is enabled.
* [ ] ICAP server is configured.
* [ ] ICAP profile exists.
* [ ] ICAP profile is attached to the correct policy.
* [ ] Proxy inspection is active.
* [ ] REQMOD is enabled if required.
* [ ] RESPMOD is enabled if required.
* [ ] Correct ICAP server is selected.
* [ ] Correct service/path is selected.
* [ ] Forwarding rule matches.
* [ ] Host matches.
* [ ] HTTP method matches.
* [ ] Content-Type matches.
* [ ] Response code matches.
* [ ] Streaming bypass is not unexpectedly excluding traffic.
* [ ] Default action is understood.
* [ ] ICAP health check is healthy.

---

# 37. Troubleshooting — HTTPS Not Inspected

```text
HTTPS
  ↓
SSL Inspection?
  │
  ├── NO → Content remains encrypted
  │
  └── YES
       ↓
   HTTP Content
       ↓
      ICAP
```

### Checklist

* [ ] SSL inspection is enabled.
* [ ] Correct policy matches.
* [ ] Correct inspection mode is selected.
* [ ] Client trusts the inspection CA.
* [ ] HTTPS traffic is actually decrypted.
* [ ] ICAP profile is attached.
* [ ] Correct request/response direction is enabled.
* [ ] Content matches ICAP forwarding criteria.
* [ ] ICAP server is reachable.

---

# 38. Troubleshooting — ICAP Returns 405

```text
405 Method Not Allowed
```

Check:

* [ ] FortiGate is sending the expected ICAP method.
* [ ] ICAP service supports REQMOD/RESPMOD as required.
* [ ] Correct ICAP service/path is configured.
* [ ] ICAP server configuration is correct.
* [ ] ICAP server supports the requested operation.

Mental model:

```text
FortiGate
   │
   ▼
RESPMOD
   │
   ▼
ICAP Service
   │
   └── Supports REQMOD only
          ↓
         405
```

---

# 39. Troubleshooting — ICAP Returns 408

```text
408 Request Timeout
```

Check:

* [ ] ICAP server response time.
* [ ] Network latency.
* [ ] Packet loss.
* [ ] ICAP server CPU.
* [ ] ICAP server load.
* [ ] Large-file processing.
* [ ] Connection handling.
* [ ] Timeout configuration.
* [ ] ICAP service health.

---

# 40. Troubleshooting — ICAP Returns 503

```text
503 Service Overloaded
```

Check:

* [ ] ICAP connection limit.
* [ ] ICAP worker capacity.
* [ ] ICAP CPU.
* [ ] ICAP memory.
* [ ] Number of concurrent clients.
* [ ] FortiGate request rate.
* [ ] Large files.
* [ ] Streaming traffic.
* [ ] Number of FortiGate devices using the server.
* [ ] Load distribution.
* [ ] Additional ICAP servers.

Concept:

```text
Too Many Requests
       ↓
ICAP Capacity Exhausted
       ↓
503
```

---

# 41. Troubleshooting — Slow Web Browsing

Measure:

* [ ] FortiGate CPU.
* [ ] FortiGate memory.
* [ ] WAD resources.
* [ ] SSL inspection load.
* [ ] ICAP CPU.
* [ ] ICAP memory.
* [ ] ICAP connections.
* [ ] ICAP latency.
* [ ] Network latency.
* [ ] Content size.
* [ ] Concurrent users.
* [ ] Large-file processing.
* [ ] Streaming inspection.

### Optimization

```text
Inspect Everything
       ↓
High ICAP Load
       ↓
High Latency
```

Prefer:

```text
Selective Content
       ↓
ICAP
       ↓
Lower Load
       ↓
Lower Latency
```

---

# 42. Troubleshooting Decision Tree

```text
                 Web Problem
                     │
                     ▼
              Does HTTP Work?
                /          \
              NO            YES
              │              │
              ▼              ▼
       Check Policy       ICAP Triggered?
       Routing             /       \
       DNS                NO        YES
       SSL                │          │
                          ▼          ▼
                    Check Profile  ICAP Error?
                    Rules          │
                    Proxy          ▼
                    Matching    Check Server
                                Status
                                Capacity
                                Response
```

---

# 43. Packet-Level Troubleshooting

When required:

* [ ] Capture traffic between FortiGate and ICAP server.
* [ ] Confirm TCP handshake.
* [ ] Confirm correct destination port.
* [ ] Confirm ICAP requests are transmitted.
* [ ] Confirm ICAP responses return.
* [ ] Check retransmissions.
* [ ] Check resets.
* [ ] Check timeouts.
* [ ] Check TLS negotiation for secure ICAP.
* [ ] Correlate packet capture with FortiGate logs.
* [ ] Correlate with ICAP server logs.

Concept:

```text
FortiGate
   │
   │ TCP
   ▼
ICAP Server
   │
   ├── Request
   ├── Response
   └── Error
```

---

# 44. Logging & Monitoring

* [ ] Enable appropriate ICAP block logging.
* [ ] Monitor ICAP errors.
* [ ] Monitor 4xx responses.
* [ ] Monitor 5xx responses.
* [ ] Monitor ICAP health.
* [ ] Monitor ICAP connection utilization.
* [ ] Monitor WAD.
* [ ] Monitor FortiGate CPU.
* [ ] Monitor FortiGate memory.
* [ ] Monitor ICAP server CPU.
* [ ] Monitor ICAP server memory.
* [ ] Monitor inspection latency.
* [ ] Monitor failure events.
* [ ] Monitor fail-open/fail-closed events.

---

# 45. Production Readiness Checklist

## Architecture

* [ ] Business requirement documented.
* [ ] ICAP use case documented.
* [ ] Traffic scope documented.
* [ ] REQMOD/RESPMOD requirement documented.
* [ ] HTTPS requirement documented.
* [ ] File inspection requirement documented.
* [ ] Streaming policy documented.

## Security

* [ ] Secure ICAP requirement evaluated.
* [ ] ICAP server access restricted.
* [ ] Certificate requirements validated.
* [ ] DLP requirements validated.
* [ ] Antivirus requirements validated.
* [ ] Content filtering requirements validated.
* [ ] Logging enabled.

## Availability

* [ ] ICAP health check configured.
* [ ] Failure behavior documented.
* [ ] ICAP redundancy evaluated.
* [ ] Recovery tested.
* [ ] Failure scenario tested.

## Performance

* [ ] FortiGate capacity tested.
* [ ] WAD capacity tested.
* [ ] ICAP capacity tested.
* [ ] Concurrent connections tested.
* [ ] Large files tested.
* [ ] Streaming traffic tested.
* [ ] Latency measured.

---

# 46. Failure Testing

Before production:

* [ ] Stop ICAP service.
* [ ] Verify FortiGate detects ICAP failure.
* [ ] Verify configured failure behavior.
* [ ] Test user traffic.
* [ ] Start ICAP service.
* [ ] Verify recovery.
* [ ] Verify health check returns UP.
* [ ] Verify ICAP processing resumes.
* [ ] Repeat with high traffic.
* [ ] Test one ICAP server failure in a multi-server design.

---

# 47. Validation Test Matrix

| Test                  | Expected Result                | Status |
| --------------------- | ------------------------------ | ------ |
| ICAP server reachable | Connection succeeds            | [ ]    |
| ICAP service running  | Service UP                     | [ ]    |
| Health check          | Server healthy                 | [ ]    |
| REQMOD                | Request reaches ICAP           | [ ]    |
| RESPMOD               | Response reaches ICAP          | [ ]    |
| HTTP inspection       | Content inspected              | [ ]    |
| HTTPS inspection      | Decrypted content inspected    | [ ]    |
| Content-Type match    | Content forwarded              | [ ]    |
| Status-code match     | Response forwarded             | [ ]    |
| Non-match             | Default action applied         | [ ]    |
| 204 response          | No modification workflow works | [ ]    |
| ICAP unavailable      | Failure policy works           | [ ]    |
| Large file            | Expected behavior              | [ ]    |
| Streaming             | Expected bypass/inspection     | [ ]    |
| ICAP recovery         | Processing resumes             | [ ]    |
| Multiple ICAP servers | Redundancy works               | [ ]    |

---

# 48. Common Design Mistakes

### ❌ Inspecting Everything

```text
All Web Traffic
      ↓
ICAP
```

Potential result:

```text
CPU ↑
Memory ↑
Latency ↑
Connections ↑
ICAP Load ↑
```

Better:

```text
Required Content
      ↓
ICAP
```

---

### ❌ Ignoring HTTPS Decryption

```text
HTTPS
  ↓
Encrypted
  ↓
No Payload Visibility
```

Validate the SSL inspection architecture first.

---

### ❌ No ICAP Failure Strategy

Never deploy without deciding:

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
WAD Capacity
      +
Network Capacity
      +
ICAP Capacity
      =
Real Capacity
```

---

### ❌ Unnecessary Streaming Inspection

Large streaming objects can consume significant resources.

Evaluate:

```text
streaming-content-bypass
```

before production.

---

### ❌ No Recovery Test

Do not test only:

```text
ICAP UP
```

Also test:

```text
ICAP DOWN
     ↓
Recovery
     ↓
ICAP UP
```

---

# 49. NSE Fast Recall

| Topic            | Remember                                           |
| ---------------- | -------------------------------------------------- |
| ICAP             | Internet Content Adaptation Protocol               |
| FortiGate        | ICAP Client                                        |
| ICAP Server      | External content adaptation/inspection             |
| REQMOD           | Request                                            |
| RESPMOD          | Response                                           |
| TCP/1344         | Common ICAP port                                   |
| 204              | No Modification Needed                             |
| 405              | Method Not Allowed                                 |
| 408              | Request Timeout                                    |
| 503              | Service Overloaded                                 |
| 505              | ICAP Version Not Supported                         |
| WAD              | Web proxy processing component                     |
| Proxy Mode       | Important for ICAP web-content processing          |
| SSL Inspection   | Needed for decrypted HTTPS content inspection      |
| Streaming Bypass | Reduces unnecessary ICAP processing                |
| Content-Type     | Can be used for selective forwarding               |
| HTTP Status Code | Can be used for response selection                 |
| C-ICAP           | Open-source ICAP implementation                    |
| Fail-Open        | Availability prioritized over inspection assurance |
| Fail-Closed      | Inspection assurance prioritized over availability |

---

# 50. 🧠 One-Minute ICAP Mental Model

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
              Content Selection
                     │
                     ▼
                    ICAP
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
         AV         DLP      Filtering
          │          │          │
          └──────────┼──────────┘
                     ▼
                ICAP Result
                     │
                     ▼
                 FortiGate
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
         IPS      WebFilter  AppControl
                     │
                     ▼
                   Client
```

---

# 51. 🔥 Final Production Checklist

### Design

* [ ] ICAP business requirement defined.
* [ ] Traffic scope defined.
* [ ] REQMOD/RESPMOD requirement defined.
* [ ] HTTPS inspection requirement defined.
* [ ] File-transfer requirement defined.
* [ ] Streaming policy defined.

### Configuration

* [ ] ICAP feature enabled.
* [ ] ICAP server configured.
* [ ] Correct IP configured.
* [ ] Correct port configured.
* [ ] Secure ICAP configuration validated if required.
* [ ] Certificate validated.
* [ ] ICAP profile configured.
* [ ] REQMOD configured where required.
* [ ] RESPMOD configured where required.
* [ ] Failure behavior configured.
* [ ] Health check configured.
* [ ] Forwarding rules configured.

### Policy

* [ ] Correct firewall policy identified.
* [ ] Proxy inspection enabled.
* [ ] SSL inspection configured where required.
* [ ] ICAP profile attached.
* [ ] Security profiles validated.

### Selective Inspection

* [ ] Host filtering validated.
* [ ] Method filtering validated.
* [ ] Content-Type filtering validated.
* [ ] HTTP status filtering validated.
* [ ] Streaming bypass validated.
* [ ] Default action validated.

### Performance

* [ ] WAD monitored.
* [ ] FortiGate CPU monitored.
* [ ] FortiGate memory monitored.
* [ ] ICAP CPU monitored.
* [ ] ICAP memory monitored.
* [ ] Concurrent connections tested.
* [ ] Large files tested.
* [ ] Streaming tested.
* [ ] Latency measured.

### Availability

* [ ] Health check tested.
* [ ] ICAP failure tested.
* [ ] Fail-open/fail-closed behavior tested.
* [ ] Recovery tested.
* [ ] Multiple ICAP servers evaluated.
* [ ] ICAP capacity validated.

### Troubleshooting

* [ ] ICAP connectivity tested.
* [ ] ICAP service status checked.
* [ ] Listening port verified.
* [ ] ICAP logs checked.
* [ ] FortiGate logs checked.
* [ ] WAD checked.
* [ ] Packet capture performed when necessary.
* [ ] ICAP status codes interpreted correctly.

---

# 52. 🏆 Final ICAP Cheat Memory

```text
                  ICAP
                   │
          ┌────────┴────────┐
          ▼                 ▼
       REQMOD            RESPMOD
          │                 │
       Request           Response
          │                 │
          └────────┬────────┘
                   ▼
              ICAP Server
                   │
          ┌────────┼────────┐
          ▼        ▼        ▼
         AV       DLP     Filter
                   │
                   ▼
               FortiGate
                   │
          ┌────────┼────────┐
          ▼        ▼        ▼
         IPS    WebFilter  AppControl
                   │
                   ▼
                 Client
```

## ⭐ One-Line Memory Trick

```text
REQMOD = Request
RESPMOD = Response
1344   = Common ICAP Port
204    = No Modification
405    = Method Not Allowed
408    = Request Timeout
503    = Service Overloaded
HTTPS  = Decrypt First → Inspect Second
WAD    = Web Proxy Processing
Selective Forwarding = Better Performance
```

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

> ## 🛡️ SheynShield Engineering Principle
>
> **ICAP is not simply “send web traffic to another server.”**
>
> A production engineer must answer:
>
> **What content should be inspected?**
>
> **Why should it be inspected?**
>
> **Which ICAP method should be used?**
>
> **What happens if the ICAP server fails?**
>
> **Can the ICAP infrastructure handle the traffic volume?**
>
> The practical architecture is:
>
> ```text
> Proxy
>   ↓
> SSL Decryption
>   ↓
> Content Selection
>   ↓
> ICAP
>   ↓
> Security Profiles
>   ↓
> Client
> ```
>
> And the performance principle is:
>
> ```text
> Selective ICAP
>      >
> Inspect Everything
> ```
>
> **Design the inspection boundary before designing the ICAP server.**

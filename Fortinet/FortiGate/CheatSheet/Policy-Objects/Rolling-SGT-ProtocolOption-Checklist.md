# 🔗 SheynShield Resources

# FortiGate Security Checklist — Policy Rolling Count, Cisco SGT & Protocol Options

> **FortiOS Focus:** 7.x  
> **Level:** NSE 4 / NSE 7 / Network Security Engineer  
> **Topics:** Policy Statistics · Cisco TrustSec SGT · Protocol Options · Oversized Files · Comfort Client · HTTP Chunked Bypass · SMTP Signature

---

# 📌 Quick Navigation

- [Policy Rolling Count Checklist](#-policy-rolling-count-checklist)
- [Cisco SGT Checklist](#-cisco-sgt-checklist)
- [Cisco TrustSec / IOS-XE Checklist](#-cisco-trustsec--ios-xe-checklist)
- [FortiGate SGT Policy Checklist](#-fortigate-sgt-policy-checklist)
- [Protocol Options Checklist](#-protocol-options-checklist)
- [Oversized Files Security Checklist](#-oversized-files-security-checklist)
- [Comfort Client Checklist](#-comfort-client-checklist)
- [HTTP Chunked Bypass Checklist](#-http-chunked-bypass-checklist)
- [SMTP Append Signature Checklist](#-smtp-append-signature-checklist)
- [Troubleshooting Checklist](#-troubleshooting-checklist)
- [NSE Exam Quick Recall](#-nse-exam-quick-recall)

---

# 📊 Policy Rolling Count Checklist

## Purpose

Verify FortiGate policy utilization visibility.

- [ ] Review firewall policy traffic statistics
- [ ] Identify high-volume policies
- [ ] Identify unused policies
- [ ] Identify stale security rules
- [ ] Perform policy cleanup
- [ ] Use statistics for capacity planning

---

## Seven-Day Rolling Count

Verify:

```text
Policy Usage
      ↓
7-Day Traffic History
      ↓
Optimization Decision
````

Checklist:

* [ ] Check historical traffic volume
* [ ] Review policy hit count
* [ ] Validate before deleting rules

---

## CLI Verification

```bash
diagnose firewall iprope list
```

Verify:

* [ ] Policy processing information
* [ ] Policy statistics
* [ ] Rule matching behavior

---

# 🏷️ Cisco Security Group Tag (SGT) Checklist

## SGT Fundamentals

Verify understanding:

* [ ] SGT = Security Group Tag
* [ ] SGT provides security identity
* [ ] SGT is part of Cisco TrustSec
* [ ] Policy decisions can use identity instead of IP

Traditional model:

```text
Source IP
Destination IP
Port
Protocol
```

SGT model:

```text
Security Identity
        ↓
Policy Decision
```

---

## Example SGT Mapping

```text
SGT 20 → Engineering

SGT 30 → Servers

SGT 40 → Contractors
```

Checklist:

* [ ] Define security groups
* [ ] Map users/devices to SGT
* [ ] Verify SGT propagation
* [ ] Verify policy enforcement

---

# 🔄 Cisco TrustSec Architecture Checklist

Verify traffic flow:

```text
Cisco Infrastructure

        ↓

SGT Information

        ↓

FortiGate

        ↓

Policy Matching

        ↓

Security Enforcement
```

Checklist:

* [ ] Cisco TrustSec enabled
* [ ] SGT information available
* [ ] FortiGate can detect SGT
* [ ] Firewall policy uses SGT

---

# ⚙️ FortiGate CTS Configuration Checklist

Verify:

```bash
config system cts
```

Checklist:

* [ ] CTS feature configured
* [ ] Cisco TrustSec integration verified
* [ ] Supported deployment validated

---

# 🔌 SGT Deployment Requirement Checklist

Verify supported design:

* [ ] Virtual Wire Pair deployment
* [ ] Transparent VDOM deployment
* [ ] Cisco TrustSec environment
* [ ] FortiOS version compatibility
* [ ] FortiGate model compatibility

---

# 🧪 SGT Session Verification Checklist

Check session information:

```bash
diagnose sys session list | grep ext
```

Verify:

* [ ] Extended session information
* [ ] SGT context visibility
* [ ] Traffic session association

---

# 🖥️ Cisco IOS-XE SGT Checklist

## Enable SXP

Verify:

```cisco
cts sxp enable
```

Checklist:

* [ ] SXP enabled
* [ ] SXP peer configured
* [ ] Authentication configured

---

## SXP Peer Configuration

Example:

```cisco
cts sxp connection peer 192.168.10.2 source 192.168.10.1 \
password default mode local both
```

Verify:

* [ ] Peer IP correct
* [ ] Source IP correct
* [ ] Mode correct
* [ ] Password configured securely

---

## SGT Mapping Checklist

Verify:

```cisco
cts role-based sgt-map <IP> sgt <TAG>
```

Checklist:

* [ ] IP mapped correctly
* [ ] SGT value correct
* [ ] Mapping advertised

---

# 🔍 Cisco SGT Verification Checklist

Run:

```cisco
show cts sxp connections
```

Check:

* [ ] SXP status
* [ ] Peer state
* [ ] Connection health

---

Run:

```cisco
show cts sxp connections brief
```

Check:

* [ ] Summary information

---

Run:

```cisco
show cts sxp sgt-map brief
```

Check:

* [ ] IP-to-SGT mappings

---

# 🛡️ FortiGate SGT Policy Checklist

Verify firewall policy:

```bash
config firewall policy

edit 1

set sgt-check enable

set sgt 20

next

end
```

Checklist:

* [ ] SGT matching enabled
* [ ] Correct SGT value configured
* [ ] Policy order validated
* [ ] Traffic matches expected rule

---

## SGT Policy Logic

```text
Traffic
   ↓
SGT Detection
   ↓
SGT Match?
   │
   ├── YES → Apply Policy
   │
   └── NO → Continue Evaluation
```

---

# ⚙️ Protocol Options Checklist

Verify:

Location:

```text
Policy & Objects
        ↓
Protocol Options
```

Checklist:

* [ ] Correct protocol option profile assigned
* [ ] File handling reviewed
* [ ] Email inspection reviewed
* [ ] HTTP behavior reviewed

---

# 📦 Oversized Files Security Checklist

## Resource Impact

Large files can consume:

* [ ] Memory
* [ ] CPU
* [ ] File buffering resources
* [ ] Antivirus processing capacity

Examples:

```text
ISO Files

Large Archives

Video Files

Large Email Attachments
```

---

# Threshold Checklist

Verify:

* [ ] Oversized file threshold configured
* [ ] Default value understood
* [ ] Security impact evaluated

Example:

```text
Threshold = 10 MB
```

---

# Oversized File Action Checklist

Verify behavior:

```text
File Size

     ↓

Threshold Check

     ↓

Normal Inspection
OR
Log
OR
Block
OR
Bypass
```

Checklist:

* [ ] Log oversized files
* [ ] Block oversized files
* [ ] Understand bypass behavior

---

# ⚠️ Security vs Performance Checklist

Lower threshold:

```text
↓ Resource Usage

↓ Large File Inspection
```

Higher threshold:

```text
↑ Inspection Coverage

↑ Resource Consumption
```

Verify:

* [ ] Memory usage monitored
* [ ] Conserve mode checked
* [ ] Inspection requirement defined

---

# 💻 Comfort Client Checklist

## Purpose

Verify user experience during inspection.

Comfort Client:

```text
Improves User Experience

NOT

Security Bypass
```

---

# Comfort Client Disabled

Flow:

```text
Client Request

      ↓

FortiGate Buffer

      ↓

Inspection

      ↓

Allow / Block
```

Checklist:

* [ ] Understand buffering behavior
* [ ] Evaluate user experience impact

---

# Comfort Client Enabled

Flow:

```text
Client

 ↓

Progress Data

 ↓

FortiGate Inspection

 ↓

Final Decision
```

Checklist:

* [ ] Enable when user experience requires it
* [ ] Confirm inspection behavior
* [ ] Validate security impact

---

# 🌐 HTTP Chunked Bypass Checklist

## HTTP Chunked Transfer

Verify understanding:

```text
HTTP Server

 ↓

Chunk 1

 ↓

Chunk 2

 ↓

Chunk 3
```

---

## Chunked Bypass Benefits

Checklist:

* [ ] Improve response speed
* [ ] Support dynamic content
* [ ] Reduce initial waiting time

---

## Security Consideration

Verify:

* [ ] Inspection impact understood
* [ ] Security requirements reviewed
* [ ] Bypass enabled intentionally

---

# 📧 SMTP Append Signature Checklist

## Purpose

Verify email disclaimer configuration.

Use cases:

* [ ] Corporate disclaimer
* [ ] Legal notice
* [ ] Security warning
* [ ] Internal policy message

---

## Signature Limitation

Verify:

```text
Maximum Length

1023 Characters
```

Checklist:

* [ ] Message length validated
* [ ] Plain-text format reviewed

---

# 🔍 Troubleshooting Checklist

# SGT Not Matching

```text
SGT Policy Failure

        ↓

CTS Configured?

        ↓

SGT Learned?

        ↓

FortiGate Detecting SGT?

        ↓

Firewall Policy Match?

        ↓

sgt-check + sgt value
```

Checklist:

* [ ] Check CTS configuration
* [ ] Check SXP connection
* [ ] Check SGT mapping
* [ ] Check session information
* [ ] Check firewall policy

---

# Large File Inspection Issue

Checklist:

```text
[ ] Check Protocol Option

[ ] Check Oversized Threshold

[ ] Check AV Profile

[ ] Check Memory Usage

[ ] Check Conserve Mode

[ ] Tune Inspection Strategy
```

---

# 🧰 High Value CLI Checklist

## FortiGate Policy Statistics

```bash
diagnose firewall iprope list
```

---

## FortiGate Sessions

```bash
diagnose sys session list
```

Extended information:

```bash
diagnose sys session list | grep ext
```

---

## CTS Configuration

```bash
config system cts
```

---

## Cisco Verification

```cisco
show cts sxp connections

show cts sxp connections brief

show cts sxp sgt-map brief
```

---

# 🧠 NSE 4 / NSE 7 Exam Quick Recall

```text
Policy Rolling Count
        ↓
7-Day Policy Traffic Visibility


SGT
        ↓
Security Group Identity


CTS
        ↓
Cisco TrustSec


SGT Policy
        ↓
sgt-check + sgt value


Oversized File
        ↓
Security vs Resource Trade-off


Comfort Client
        ↓
Better User Experience


HTTP Chunked Bypass
        ↓
Faster HTTP Response


Append Signature
        ↓
SMTP Disclaimer
```

---

# 🎯 Production Design Checklist

## Identity-Based Security

* [ ] Use SGT when identity is more meaningful than IP
* [ ] Validate Cisco TrustSec integration
* [ ] Verify SGT propagation path
* [ ] Test policy enforcement

---

## File Inspection

* [ ] Balance security and performance
* [ ] Monitor memory usage
* [ ] Tune oversized thresholds carefully
* [ ] Avoid unnecessary bypass behavior

---

## Policy Optimization

* [ ] Review unused policies
* [ ] Review high-volume policies
* [ ] Use traffic statistics before changes

---

# ⭐ Golden Rules

* [ ] SGT is identity-based security context, not an IP replacement
* [ ] Cisco TrustSec provides SGT infrastructure
* [ ] FortiGate must detect SGT before matching policies
* [ ] Oversized file settings affect inspection depth
* [ ] Comfort Client improves UX, not trust level
* [ ] HTTP bypass features require security review
* [ ] Policy statistics help optimize firewall rules

---

# Final Security Architecture

```text
                Traffic

                   ↓

          Identity Detection

                   ↓

              SGT / IP / Port

                   ↓

          Firewall Policy Match

                   ↓

        Security Inspection

                   ↓

          Allow / Block / Log
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

> **SheynShield | Engineering Secure Networks**
> FortiGate · Security Policy · Cisco SGT · TrustSec · Protocol Inspection


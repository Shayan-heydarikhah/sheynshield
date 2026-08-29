# 🔗 SheynShield Resources

# FortiGate Geo-IP Anycast Recognition & Geo-Blocking Checklist

> **FortiOS 7.2.x | Advanced Troubleshooting | Security Engineering**
>
> Geo-IP Filtering, Anycast Detection, Physical vs Registered Location, ISDB Matching Logic, Policy Troubleshooting

---

# 📌 Table of Contents

- [1. Geo-IP Decision Architecture](#1-geo-ip-decision-architecture)
- [2. Anycast Awareness Checklist](#2-anycast-awareness-checklist)
- [3. Geo-IP Investigation Workflow](#3-geo-ip-investigation-workflow)
- [4. IP Location Validation](#4-ip-location-validation)
- [5. Physical vs Registered Location Analysis](#5-physical-vs-registered-location-analysis)
- [6. Geo-IP Policy Design Checklist](#6-geo-ip-policy-design-checklist)
- [7. geoip-anycast Validation](#7-geoip-anycast-validation)
- [8. geoip-match Validation](#8-geoip-match-validation)
- [9. DNS Dependency Checklist](#9-dns-dependency-checklist)
- [10. Policy Order Validation](#10-policy-order-validation)
- [11. Troubleshooting Commands](#11-troubleshooting-commands)
- [12. Production Security Checklist](#12-production-security-checklist)
- [13. NSE Interview Notes](#13-nse-interview-notes)

---

# 1. Geo-IP Decision Architecture

## Core Mental Model

Before creating Geo-IP policies, remember:

```text
Destination IP
       |
       v
Geo-IP Database
       |
       +----------------+
       |                |
 Physical Location   Registered Location
       |                |
       +----------------+
              |
              v
        Anycast Detection
              |
              v
      geoip-anycast Logic
              |
              v
       geoip-match Mode
              |
              v
       Firewall Policy
              |
              v
        ACCEPT / DENY
````

## Engineer Checklist

* [ ] Do not assume Geo-IP equals physical geographic location.
* [ ] Identify whether destination IP is Anycast.
* [ ] Verify physical location.
* [ ] Verify registered location.
* [ ] Understand policy matching mode.
* [ ] Validate final firewall decision.

---

# 2. Anycast Awareness Checklist

## What Is Anycast?

Anycast allows the same IP address to exist in multiple geographic locations.

Example:

```text
          Same IP Address

              8.8.8.8

        /       |       \

     US       EU       ASIA

     DNS      DNS      DNS
    Server   Server   Server
```

## Common Anycast Examples

* [ ] 8.8.8.8
* [ ] 1.1.1.1
* [ ] CDN services
* [ ] Cloud DNS services
* [ ] Global application endpoints

## Before Blocking Countries

Validate:

```text
Destination
     |
     v
Is Anycast?
     |
     +---- YES
     |
     v
Check Geo-IP Behavior
```

---

# 3. Geo-IP Investigation Workflow

## Step 1 — Resolve the FQDN

Example:

```bash
ping example.com
```

Record:

```text
Hostname
     |
     v
Resolved IP
```

Checklist:

* [ ] Record all returned IP addresses.
* [ ] Check DNS load balancing behavior.
* [ ] Check CDN involvement.

---

## Step 2 — Query Geo-IP Database

Command:

```bash
diagnose firewall ipgeo ip2country <IP>
```

Example:

```bash
diagnose firewall ipgeo ip2country 8.8.8.8
```

Validate:

* [ ] Country
* [ ] Registered country
* [ ] Anycast status

---

# 4. IP Location Validation

## Normal IP Example

```bash
diagnose firewall ipgeo ip2country 185.192.112.25
```

Expected:

```text
Country = IR
Registered Country = IR
Anycast = NO
```

Checklist:

* [ ] Physical location matches expectation.
* [ ] Registered location matches expectation.
* [ ] No Anycast behavior.

---

## Anycast Example

```bash
diagnose firewall ipgeo ip2country 8.8.8.8
```

Possible result:

```text
Country = US
Registered Country = US
Anycast = YES
```

Checklist:

* [ ] Confirm Anycast.
* [ ] Confirm Geo-IP country.
* [ ] Confirm policy impact.

---

# 5. Physical vs Registered Location Analysis

## Important Difference

An IP may have:

```text
Physical Location
        !=
Registered Location
```

Example:

```text
IP:

185.143.233.120


Physical Location:

US


Registered Location:

IR


Anycast:

YES
```

## Checklist

* [ ] Identify physical country.
* [ ] Identify registered country.
* [ ] Check Anycast.
* [ ] Select correct matching mode.

---

# 6. Geo-IP Policy Design Checklist

## Example Requirement

```text
Block:

US Destinations
```

Policy:

```text
Source:
all

Destination:
US

Action:
DENY
```

---

## Policy Checklist

* [ ] Create specific exceptions first.
* [ ] Place Geo-IP deny rules after exceptions.
* [ ] Enable logging.
* [ ] Test real traffic.
* [ ] Validate with packet capture.

---

## Recommended Order

```text
Traffic

   |
   v

Specific Exceptions

   |
   v

Anycast Required Services

   |
   v

Geo-IP Restrictions

   |
   v

General Internet Policy
```

---

# 7. geoip-anycast Validation

## Configuration

```bash
set geoip-anycast enable
```

Purpose:

```text
Control Anycast address matching behavior
```

---

## Validation Checklist

* [ ] Check if destination is Anycast.
* [ ] Check geoip-anycast configuration.
* [ ] Confirm expected policy behavior.
* [ ] Test with real traffic.

---

## Important Rule

`geoip-anycast` is NOT:

```text
Allow Anycast IP
```

It controls:

```text
How Geo-IP matching handles Anycast addresses
```

---

# 8. geoip-match Validation

## Physical Location Matching

Default logic may use:

```text
Physical Location
```

Example:

```text
IP

185.143.233.120


Physical:

US


Result:

US Policy Match
```

---

## Registered Location Matching

Configuration:

```bash
config firewall policy

edit 1

set geoip-match registered-location

end
```

Now:

```text
IP

185.143.233.120


Registered:

IR
```

---

## Checklist

* [ ] Verify current geoip-match mode.
* [ ] Confirm business requirement.
* [ ] Test both location modes.
* [ ] Document exceptions.

---

# 9. DNS Dependency Checklist

Geo-IP policies can accidentally affect infrastructure services.

Validate:

## DNS

```text
8.8.8.8
1.1.1.1
```

Checklist:

* [ ] Test DNS resolution.
* [ ] Test FortiGate DNS.
* [ ] Check cloud DNS dependencies.

---

## Other Critical Services

Validate:

* [ ] NTP
* [ ] CDN
* [ ] Cloud APIs
* [ ] Security services
* [ ] Update servers
* [ ] Authentication services

---

# 10. Policy Order Validation

FortiGate uses:

```text
First Matching Policy
```

## Correct Design

```text
Policy 1

Specific Exception

ACCEPT


Policy 2

Geo-IP Block

DENY


Policy 3

General Internet

ACCEPT
```

---

## Checklist

* [ ] Review policy sequence.
* [ ] Check source address.
* [ ] Check destination object.
* [ ] Check service.
* [ ] Check schedule.
* [ ] Confirm logging.

---

# 11. Troubleshooting Commands

## Check Geo-IP

```bash
diagnose firewall ipgeo ip2country <IP>
```

---

## Flow Debug

```bash
diagnose debug flow filter addr <IP>

diagnose debug enable

diagnose debug flow trace start 50
```

Stop:

```bash
diagnose debug disable

diagnose debug flow trace stop
```

---

## Checklist

* [ ] Confirm resolved IP.
* [ ] Confirm Geo-IP database result.
* [ ] Confirm policy match.
* [ ] Confirm deny reason.
* [ ] Confirm Anycast behavior.

---

# 12. Production Security Checklist

## Geo-IP Deployment

* [ ] Document blocked countries.
* [ ] Document business justification.
* [ ] Identify critical Anycast services.
* [ ] Test before enforcement.
* [ ] Enable traffic logging.
* [ ] Monitor false positives.

---

## Before Blocking

Validate:

```text
IP
 |
 +--> Physical Country

 |
 +--> Registered Country

 |
 +--> Anycast Status

 |
 +--> geoip-anycast

 |
 +--> geoip-match

 |
 +--> Policy Order
```

---

# 13. NSE Interview Notes

## Q: How do you check Anycast status?

Answer:

```bash
diagnose firewall ipgeo ip2country <IP>
```

Look for:

```text
is anycast ip.
```

---

## Q: How do you use registered location?

Answer:

```bash
set geoip-match registered-location
```

---

## Q: Why can Geo-IP block legitimate services?

Answer:

Because:

```text
Geo-IP
!=
Real Physical Location
```

Anycast services may have:

```text
Physical Location

different from

Registered Location
```

---

# 🔥 Golden Troubleshooting Workflow

```text
Unexpected Geo-IP Block

        |
        v

Resolve Destination IP

        |
        v

Run ip2country

        |
        v

Check Anycast

        |
        v

Compare Physical / Registered

        |
        v

Check geoip-anycast

        |
        v

Check geoip-match

        |
        v

Review Policy Order

        |
        v

Validate Traffic Logs
```

---

# 🧠 Final Memory Hook

```text
GEO-IP FILTERING

IS NOT ONLY:

IP → Country → Allow/Deny


Think:

Physical Location

+

Registered Location

+

Anycast Detection

+

geoip-anycast

+

geoip-match

+

Policy Order


=

Final Firewall Decision
```

---

# SheynShield Rule

```text
Never troubleshoot Geo-IP only from the country result.

Always verify:

IP
↓
Location Type
↓
Anycast
↓
Match Mode
↓
Policy Logic
```

---

# Tags

```text
#FortiGate
#FortiOS
#Fortinet
#GeoIP
#Anycast
#ISDB
#Firewall
#NetworkSecurity
#CyberSecurity
#NSE7
#NSE4
#FortiGuard
#ZeroTrust
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

**SheynShield | Engineering Secure Networks**


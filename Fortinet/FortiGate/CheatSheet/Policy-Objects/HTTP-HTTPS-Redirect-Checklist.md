# 🔗 SheynShield Resources

# FortiGate HTTP → HTTPS Redirect Checklist

> **FortiOS | Virtual Server | HTTP Redirect | HTTPS Reverse Proxy | TLS Termination | Load Balancing**
>
> **Level:** Advanced / NSE4-NSE7 / Security Engineering  
> **Purpose:** Force HTTP clients to migrate from TCP/80 to HTTPS TCP/443 using FortiGate Virtual Server functionality.

---

# 📌 Table of Contents

- [1. Architecture Overview](#1-architecture-overview)
- [2. Enable Load Balance Feature](#2-enable-load-balance-feature)
- [3. HTTP Redirect Virtual Server Checklist](#3-http-redirect-virtual-server-checklist)
- [4. HTTPS Virtual Server Checklist](#4-https-virtual-server-checklist)
- [5. Firewall Policy Checklist](#5-firewall-policy-checklist)
- [6. SSL Inspection Checklist](#6-ssl-inspection-checklist)
- [7. Traffic Flow Validation](#7-traffic-flow-validation)
- [8. Verification Commands](#8-verification-commands)
- [9. Troubleshooting Checklist](#9-troubleshooting-checklist)
- [10. Common Mistakes](#10-common-mistakes)
- [11. NSE Exam Notes](#11-nse-exam-notes)
- [12. Engineer Final Checklist](#12-engineer-final-checklist)

---

# 1. Architecture Overview

## 🎯 Objective

Convert:

```text
http://192.168.101.100
````

into:

```text
https://192.168.101.100
```

using FortiGate Virtual Server HTTP redirect.

---

## Logical Design

```text
                  FORTIGATE

Client
  |
  |
  | TCP/80 HTTP
  |
  v

VS-HTTP
Port 80

  |
  |
  | HTTP 303 Redirect
  |
  v

Client Creates NEW Connection

  |
  |
  | TCP/443 HTTPS
  |
  v

VS-HTTPS
Port 443

  |
  |
  | Reverse Proxy
  |
  v

Real Server
192.168.20.200:80
```

---

# 2. Enable Load Balance Feature

## GUI Checklist

Navigate:

```text
System
 |
 └── Feature Visibility
        |
        └── Load Balance
```

Enable:

```text
[x] Load Balance
```

---

## Validation Checklist

* [ ] Load Balance feature is enabled
* [ ] Virtual Server menu is visible
* [ ] Firewall VIP objects are available
* [ ] Administrator has required permissions

---

# 3. HTTP Redirect Virtual Server Checklist

## Create HTTP Listener

Path:

```text
Policy & Objects
 |
 └── Virtual Servers
```

Create:

| Parameter           | Value           |
| ------------------- | --------------- |
| Name                | vs-http         |
| Type                | HTTP            |
| Interface           | port3           |
| Virtual IP          | 192.168.101.100 |
| Virtual Port        | 80              |
| Load Balance Method | Static          |
| Real Server         | 192.168.20.200  |
| Real Server Port    | 80              |

---

## HTTP Redirect Configuration

Enable:

```text
HTTP Redirect = Enable
```

CLI:

```bash
config firewall vip
    edit "vs-http"
        set http-redirect enable
    next
end
```

---

## Validation Checklist

* [ ] HTTP Virtual Server created
* [ ] Virtual IP is correct
* [ ] Listening interface is correct
* [ ] TCP/80 is configured
* [ ] HTTP redirect enabled
* [ ] Real server configured
* [ ] Backend connectivity verified

---

# 4. HTTPS Virtual Server Checklist

## Create HTTPS Listener

Configuration:

| Parameter           | Value           |
| ------------------- | --------------- |
| Name                | vs-https        |
| Type                | HTTPS           |
| Interface           | port3           |
| Virtual IP          | 192.168.101.100 |
| Virtual Port        | 443             |
| Load Balance Method | Static          |

---

## Backend Configuration

Example:

```text
Frontend:

Client
192.168.101.100:443


Backend:

Server
192.168.20.200:80
```

---

## Validation Checklist

* [ ] HTTPS Virtual Server exists
* [ ] TCP/443 listener configured
* [ ] Correct certificate selected
* [ ] Backend server reachable
* [ ] Backend port verified
* [ ] Real server status is active

---

# 5. Firewall Policy Checklist

## Required Policy Flow

```text
LAN
 |
 | HTTP/HTTPS
 |
 v
FortiGate Virtual Server
 |
 v
DMZ Web Server
```

---

## Policy Requirements

Example:

```text
Source:
all

Destination:

vs-http
vs-https

Service:

HTTP
HTTPS

Action:

ACCEPT
```

---

## Checklist

* [ ] Correct incoming interface
* [ ] Correct outgoing interface
* [ ] Destination includes Virtual Servers
* [ ] HTTP service allowed
* [ ] HTTPS service allowed
* [ ] Policy order verified
* [ ] No deny policy overrides
* [ ] Logging enabled
* [ ] NAT matches design

---

# 6. SSL Inspection Checklist

## HTTPS Processing

Possible architecture:

```text
Client
 |
 | TLS
 |
 v
FortiGate

TLS Termination

 |
 | HTTP
 |
 v

Backend Server
```

---

## SSL Checklist

* [ ] Certificate imported
* [ ] Certificate matches hostname
* [ ] Client trusts certificate chain
* [ ] SSL inspection profile selected
* [ ] Deep Inspection configured if required
* [ ] Backend protocol verified

---

# 7. Traffic Flow Validation

## HTTP Flow

Expected:

```text
Client
 |
 | GET /
 |
 v
VS-HTTP :80
 |
 |
 | HTTP 303
 |
 v

Location:
https://192.168.101.100
```

---

## HTTPS Flow

Expected:

```text
Client
 |
 | HTTPS :443
 |
 v

VS-HTTPS

 |
 |
 v

Real Server
192.168.20.200:80
```

---

# 8. Verification Commands

## Test HTTP Redirect

```bash
curl -I http://192.168.101.100
```

Expected:

```text
HTTP/1.1 303 See Other

Location:
https://192.168.101.100
```

---

## Follow Redirect

```bash
curl -IL http://192.168.101.100
```

---

## Test HTTPS

```bash
curl -vk https://192.168.101.100
```

---

## Check VIP Configuration

```bash
show firewall vip
```

---

## Check Sessions

```bash
diagnose sys session list
```

---

# 9. Troubleshooting Checklist

## Virtual Server

* [ ] Load Balance enabled
* [ ] VS-HTTP exists
* [ ] VS-HTTP listens on TCP/80
* [ ] HTTP redirect enabled
* [ ] VS-HTTPS exists
* [ ] VS-HTTPS listens on TCP/443
* [ ] Virtual IP correct
* [ ] Real server reachable

---

## Firewall

* [ ] Correct policy matched
* [ ] Policy ID verified
* [ ] Service objects correct
* [ ] Logging enabled
* [ ] No conflicting deny rule

---

## Packet Capture

HTTP:

```bash
diagnose sniffer packet any \
'host 192.168.101.100 and port 80' \
4 0 l
```

HTTPS:

```bash
diagnose sniffer packet any \
'host 192.168.101.100 and port 443' \
4 0 l
```

---

## Flow Debug

```bash
diagnose debug reset

diagnose debug flow filter addr <CLIENT-IP>

diagnose debug flow show function-name enable

diagnose debug enable

diagnose debug flow trace start 50
```

Stop:

```bash
diagnose debug disable

diagnose debug reset
```

---

# 10. Common Mistakes

## ❌ Mistake 1

Only creating HTTPS Virtual Server.

Wrong:

```text
HTTPS VS only
```

Result:

```text
http://site.com
```

will not automatically become:

```text
https://site.com
```

---

## ❌ Mistake 2

Thinking port translation equals redirect.

Incorrect:

```text
80 → 443 NAT
```

Correct:

```text
HTTP Response
      |
      v
303 Redirect
      |
      v
New HTTPS Request
```

---

## ❌ Mistake 3

Using wrong backend port.

Valid design:

```text
Client:

HTTPS :443


FortiGate:


HTTP :80


Server:

HTTP :80
```

---

## ❌ Mistake 4

Ignoring proxy architecture.

Virtual Server processing may require:

```text
Proxy-based inspection
```

especially with:

* TLS termination
* SSL inspection
* HTTP manipulation

---

# 11. NSE Exam Notes 🧠

## Q: Does HTTP Redirect translate TCP ports?

Answer:

```text
No.
```

HTTP redirect is an application-layer response.

---

## Q: What happens after HTTP redirect?

Answer:

```text
Client receives redirect

↓

Creates NEW HTTPS session
```

---

## Q: Can frontend and backend ports differ?

Answer:

```text
Yes.

Example:

Client:

HTTPS 443

Backend:

HTTP 80
```

---

## Q: Main components?

Remember:

```text
VIP
 |
Virtual Server
 |
Frontend Listener
 |
Redirect / TLS Processing
 |
Real Server
```

---

# 12. Engineer Final Checklist 🔥

## Deployment Checklist

* [ ] Requirement confirmed
* [ ] Load Balance enabled
* [ ] HTTP Virtual Server created
* [ ] HTTP redirect enabled
* [ ] HTTPS Virtual Server created
* [ ] Certificate configured
* [ ] Backend server reachable
* [ ] Firewall policy created
* [ ] Inspection mode reviewed
* [ ] SSL inspection validated
* [ ] HTTP redirect tested
* [ ] HTTPS connection tested
* [ ] Logs reviewed
* [ ] Configuration documented

---

# 🧠 Final Mental Model

```text
HTTP REQUEST

Client
 |
 v

VS-HTTP :80

 |
 |
 | 303 Redirect
 |
 v

NEW HTTPS REQUEST

Client
 |
 v

VS-HTTPS :443

 |
 |
 | Reverse Proxy
 |
 v

Backend Server
```

---

# 🔥 One-Line Memory Hook

```text
HTTP Redirect
=
Application Layer Instruction

NOT

Port Translation
```

---

# SheynShield | Engineering Secure Networks

## Related Topics

* FortiGate Virtual Server
* HTTP Redirect
* HTTPS Reverse Proxy
* VIP / DNAT
* Load Balancer
* SSL Termination
* Deep Inspection
* FortiGate Firewall Policy

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


## 🐙 Technical Knowledge Base

* SheynShield GitHub

  [https://github.com/Shayan-heydarikhah/sheynshield](https://github.com/Shayan-heydarikhah/sheynshield)

#FortiGate #Fortinet #FortiOS #VirtualServer #HTTPRedirect #HTTPS #ReverseProxy #Firewall #CyberSecurity #NSE7 #NSE4


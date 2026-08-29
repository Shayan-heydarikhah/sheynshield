# FortiGate HTTP/HTTPS Redirect  

> **FortiGate HTTP → HTTPS Redirect using Virtual Server**
> Redirect client HTTP requests to HTTPS using an HTTP Virtual Server and `http-redirect`.

---

## 🎯 What This Configuration Does

The goal is to force clients requesting:

```text
http://192.168.101.100
```

to be redirected to:

```text
https://192.168.101.100
```

The FortiGate uses an **HTTP Virtual Server** as the redirect listener and an **HTTPS Virtual Server** for the actual HTTPS service.

```text
                    FortiGate
                       |
                       |
Client ── HTTP :80 ──> VS-HTTP
                       |
                       | HTTP 303 Redirect
                       v
Client ─ HTTPS :443 ─> VS-HTTPS
                       |
                       v
                 Real Server
                 192.168.20.200:80
```

---

# 1. Enable Load Balancing / Virtual Server

Navigate to:

```text
System
 └── Feature Visibility
      └── Load Balance
```

Enable **Load Balance**.

This makes the Virtual Server functionality available in the GUI.

---

# 2. Create HTTP Virtual Server

Navigate to:

```text
Policy & Objects
 └── Virtual Servers
```

Create:

| Parameter           | Value             |
| ------------------- | ----------------- |
| Name                | `vs-http`         |
| Type                | HTTP              |
| Interface           | `port3`           |
| Virtual IP          | `192.168.101.100` |
| Virtual Port        | `80`              |
| Load Balance Method | Static            |
| Real Server         | `192.168.20.200`  |
| Real Server Port    | `80`              |
| Max Connections     | `0`               |
| Status              | Active            |

The important requirement is to enable:

```text
HTTP Redirect = Enable
```

Equivalent CLI:

```bash
config firewall vip
    edit "vs-http"
        set http-redirect enable
    next
end
```

### What happens?

The FortiGate receives the HTTP request on TCP/80 and returns an HTTP redirect response.

Conceptually:

```text
Client
  |
  | HTTP GET
  | TCP/80
  v
FortiGate VS-HTTP
  |
  | HTTP 303 Redirect
  v
https://192.168.101.100
```

The client then creates a new HTTPS connection.

> **Important:** HTTP redirect does not magically convert an existing TCP/80 connection into TCP/443. The client receives a redirect response and initiates a new HTTPS connection.

---

# 3. Create HTTPS Virtual Server

Create another Virtual Server:

```text
Name:             vs-https
Type:             HTTPS
Interface:        port3
Virtual IP:       192.168.101.100
Virtual Port:     443
Load Balance:     Static
```

Real Server:

```text
IP:       192.168.20.200
Port:     80
Max Conn: 0
Mode:     Active
```

The resulting flow is:

```text
Client
   |
   | HTTPS :443
   v
VS-HTTPS
   |
   | DNAT / Reverse Proxy
   v
192.168.20.200:80
```

---

# 4. Why Can HTTPS Use Backend Port 80?

The client-facing service and backend service do not have to use the same port.

Example:

```text
Client
192.168.101.x
      |
      | HTTPS TCP/443
      v
FortiGate
VS-HTTPS :443
      |
      | HTTP TCP/80
      v
Web Server
192.168.20.200:80
```

The FortiGate can terminate/process the HTTPS side and communicate with the backend using HTTP.

This is commonly used when:

* HTTPS is required for clients
* The backend application only provides HTTP
* TLS termination is performed on the FortiGate
* Reverse-proxy functionality is required

---

# 5. Firewall Policy

Create a firewall policy allowing traffic to the Virtual Servers.

Example:

```text
Policy 1
--------------------------------
Incoming Interface:  lan
Outgoing Interface: dmz

Source:              all
Destination:
    vs-http
    vs-https

Service:
    HTTP
    HTTPS

Action:              ACCEPT
NAT:                 Disable
Log Traffic:         All

Inspection Mode:     Proxy
SSL Inspection:      Custom Deep Inspection
```

Equivalent conceptual policy:

```text
LAN
 |
 | HTTP/HTTPS
 v
FortiGate Virtual Server
 |
 +---- HTTP :80  → Redirect
 |
 +---- HTTPS :443 → Backend
 |
 v
DMZ Web Server
```

---

# 6. Why Proxy-Based Inspection?

For this architecture, the Virtual Server is operating as a reverse-proxy/load-balancing function.

Therefore, use:

```text
Inspection Mode
        ↓
Proxy-based
```

rather than treating the traffic as a simple flow-based forwarding session.

This is especially important when enabling advanced HTTPS processing such as:

* TLS termination
* SSL inspection
* HTTP manipulation
* Reverse proxy processing
* Application-layer inspection

---

# 7. SSL Inspection

For the HTTPS policy, a custom SSL inspection profile can be applied.

Example:

```text
SSL/SSH Inspection
        ↓
Custom Deep Inspection
```

This allows the FortiGate to decrypt and inspect HTTPS traffic when the required certificate/trust configuration is correctly deployed.

Conceptually:

```text
Client
   |
   | TLS
   v
FortiGate
   |
   | TLS termination / inspection
   |
   | HTTP
   v
Backend Server
```

> **Certificate planning matters:** clients must trust the certificate presented by the FortiGate when the FortiGate performs TLS interception/termination.

---

# 8. Complete Traffic Flow

```text
                    FORTIGATE
              ┌───────────────────┐
              │                   │
Client        │   VS-HTTP :80     │
  │           │        │          │
  │ HTTP      │        │ 303      │
  ├──────────>│        └──────────┤
  │           │                   │
  │ HTTPS     │   VS-HTTPS :443   │
  └──────────>│        │          │
              │        │          │
              │        v          │
              │  SSL Processing   │
              │        │          │
              │        │ HTTP :80  │
              └────────┼──────────┘
                       |
                       v
              192.168.20.200:80
```

---

# 9. Configuration Logic

### HTTP

```text
192.168.101.100:80
        |
        v
     VS-HTTP
        |
        | HTTP Redirect
        v
192.168.101.100:443
```

### HTTPS

```text
192.168.101.100:443
        |
        v
    VS-HTTPS
        |
        v
192.168.20.200:80
```

---

# 10. Key Difference: Redirect vs Reverse Proxy

| Feature               | HTTP Redirect                     | HTTPS Virtual Server             |
| --------------------- | --------------------------------- | -------------------------------- |
| Client connection     | HTTP                              | HTTPS                            |
| Port                  | 80                                | 443                              |
| Main function         | Redirect client                   | Process/forward HTTPS            |
| New client connection | Yes                               | No                               |
| Backend connection    | Depends on configuration          | Yes                              |
| Reverse proxy         | HTTP redirect component           | Yes                              |
| TLS processing        | No                                | Possible                         |
| SSL inspection        | Not applicable to redirect itself | Possible                         |
| Layer                 | HTTP application layer            | L4/L7 depending on configuration |

---

# 11. Common Mistakes

### ❌ Mistake 1 — Creating only HTTPS Virtual Server

If only `vs-https` exists:

```text
http://192.168.101.100
```

does not automatically become HTTPS.

You need an HTTP listener:

```text
VS-HTTP :80
       ↓
HTTP Redirect
       ↓
VS-HTTPS :443
```

---

### ❌ Mistake 2 — Expecting TCP Port Translation to Perform Redirect

A NAT translation such as:

```text
80 → 443
```

is **not equivalent to an HTTP redirect**.

A redirect is an application-layer response telling the client to make another request.

---

### ❌ Mistake 3 — Using the Wrong Firewall Service

The firewall policy must allow the services required by the Virtual Servers:

```text
HTTP
HTTPS
```

Avoid unnecessarily using:

```text
ALL
```

in production unless there is a specific reason.

---

### ❌ Mistake 4 — Forgetting Backend Port

The frontend and backend ports can be different:

```text
Frontend:
192.168.101.100:443

Backend:
192.168.20.200:80
```

This is valid when the Virtual Server is designed to perform the required reverse-proxy/TLS processing.

---

# 12. Troubleshooting Checklist

### Virtual Server

```text
[ ] Load Balance feature enabled
[ ] VS-HTTP exists
[ ] VS-HTTP listens on TCP/80
[ ] HTTP Redirect enabled
[ ] VS-HTTPS exists
[ ] VS-HTTPS listens on TCP/443
[ ] Correct virtual IP configured
[ ] Correct real server configured
[ ] Real server is reachable
```

### Firewall Policy

```text
[ ] Correct incoming interface
[ ] Correct outgoing interface
[ ] Destination includes required Virtual Servers
[ ] HTTP service allowed
[ ] HTTPS service allowed
[ ] Policy is above conflicting deny policies
[ ] NAT configured according to design
[ ] Proxy inspection mode used where required
```

### SSL

```text
[ ] Correct certificate configured
[ ] Certificate matches the hostname
[ ] Client trusts the issuing CA
[ ] SSL inspection profile is appropriate
[ ] Backend HTTP/HTTPS expectation is correct
```

---

# 13. Quick Verification

Test HTTP:

```bash
curl -I http://192.168.101.100
```

Look for a redirect response such as:

```text
HTTP/1.1 303 See Other
Location: https://192.168.101.100
```

Follow redirects automatically:

```bash
curl -IL http://192.168.101.100
```

Test HTTPS:

```bash
curl -vk https://192.168.101.100
```

---

# 14. Useful FortiGate Diagnostics

Inspect sessions:

```bash
diagnose sys session list
```

Filter sessions by client/server IP when troubleshooting.

For packet-level troubleshooting:

```bash
diagnose sniffer packet any 'host 192.168.101.100 and port 80' 4 0 l
```

HTTPS:

```bash
diagnose sniffer packet any 'host 192.168.101.100 and port 443' 4 0 l
```

Flow debugging can also be used when the firewall policy itself is suspected:

```bash
diagnose debug reset
diagnose debug flow filter addr <CLIENT-IP>
diagnose debug flow show function-name enable
diagnose debug enable
diagnose debug flow trace start 50
```

Stop debugging:

```bash
diagnose debug disable
diagnose debug reset
```

---

# 🧠 Engineer's Takeaway

The clean design is:

```text
                    HTTP :80
Client ─────────────────────────> VS-HTTP
                                      |
                                      | 303 Redirect
                                      v
                    HTTPS :443
Client ─────────────────────────> VS-HTTPS
                                      |
                                      | Reverse Proxy
                                      v
                              Web Server :80
```

### Remember these three points

> **1. `http-redirect` is an application-layer redirect, not a simple port translation.**

> **2. The HTTP redirect causes the client to establish a NEW HTTPS connection.**

> **3. Frontend and backend ports can be different because the Virtual Server can proxy/translate the connection.**

---

## 🔑 FortiGate CLI Reference

```bash
config firewall vip
    edit "vs-http"
        set http-redirect enable
    next
end
```

Useful verification:

```bash
show firewall vip
```

---

## 📌 Related FortiGate Topics

* Virtual Server
* HTTP Redirect
* HTTPS Reverse Proxy
* Load Balancing
* SSL/TLS Termination
* SSL Deep Inspection
* Real Server
* VIP / DNAT
* Proxy-based Inspection
* FortiGate Firewall Policy
* FortiGate Load Balancer

**FortiGate mental model:**

```text
VIP
 ↓
Virtual Server
 ↓
Frontend Listener
 ↓
HTTP Redirect / TLS Processing
 ↓
Real Server
```
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

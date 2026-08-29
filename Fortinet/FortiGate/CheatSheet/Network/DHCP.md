# FortiGate DHCP

> **FortiOS Reference:** DHCP Server, DHCP Relay & DHCP Options  

---

## 1. DHCP Server

### Basic DHCP Server Configuration

```bash
config system dhcp server
    edit 1
        set dns-service default
        set default-gateway 192.168.1.2
        set netmask 255.255.255.0
        set interface port3

        config ip-range
            edit 1
                set start-ip 192.168.101.1
                set end-ip 192.168.101.1
            next

            edit 2
                set start-ip 192.168.101.3
                set end-ip 192.168.101.254
            next
        end

        set timezone-option default
        set tftp-server 192.168.20.200
    end
````

### Excluding IP Addresses

Multiple DHCP ranges can be used to exclude specific IP addresses from the DHCP pool.

Example:

```text
192.168.101.1      ← excluded from the second range
192.168.101.2      ← excluded
192.168.101.3-254  ← DHCP range
```

> **Note:** Splitting the DHCP pool into multiple `ip-range` entries can be used to exclude IP addresses.

---

## 2. DHCP Deployment Recommendation

For enterprise environments:

```text
              DHCP Server
               192.168.20.200
                      |
                      |
                +-----+-----+
                | Network   |
                +-----------+
                      |
                  FortiGate
```

> **Recommendation:** Prefer deploying the DHCP server on a dedicated VM rather than running it locally on FortiOS, a switch, router, etc.

---

# 3. DHCP Relay

DHCP Relay allows DHCP requests from a local network to be forwarded to a remote DHCP server.

### Example Topology

```text
                    DHCP Server
                   192.168.20.200
                         |
                         |
                  +------+------+
                  |             |
                  |           FGT-1
                  |             |
                  |       12.12.12.0/30
                  |             |
                  |           FGT-2
                  |             |
                  |       192.168.102.0/24
                  |             |
                  |           LAN-2
                  |
            192.168.101.0/24
                  |
                port3
                  |
                FGT-1
```

---

## 4. DHCP Relay — FGT-1

On **FGT-1**, configure DHCP Relay on the LAN interface.

### GUI Path

```text
Network
 └── Interfaces
      └── port3
           └── DHCP Server
                └── Relay
                     └── 192.168.20.200
```

### Concept

```text
LAN Client
    |
    | DHCP Request
    v
 FGT-1
    |
    | DHCP Relay
    v
192.168.20.200
```

---

# 5. DHCP Relay — FGT-2

FGT-2 must have routes to the DHCP server and the remote LAN.

### Static Routes

```text
192.168.20.0/24
    Gateway → 12.12.12.1

192.168.101.0/24
    Gateway → 12.12.12.1

0.0.0.0/0
    Gateway → 192.168.254.2
```

### DHCP Relay

```text
Network
 └── Interfaces
      └── port3
           └── DHCP Server
                └── Relay
                     └── 192.168.20.200
```

---

## 6. Firewall Policy Requirements

DHCP Relay traffic must be allowed through the required firewall policies.

```text
DHCP Client
     |
     v
FortiGate
     |
     | Firewall Policy
     | NAT: Disabled
     v
DHCP Server
```

### Recommendation

* Ensure firewall policies allow the required DHCP traffic.
* **Better disable NAT** for this communication.

---

# 7. DHCP Relay Request Conditions

For a DHCP message to be forwarded to a relay server, the following characteristics are used:

| Field             | Expected Value                 |
| ----------------- | ------------------------------ |
| DHCP Message Type | `DHCPDISCOVER` or `DHCPINFORM` |
| Client IP Address | `0.0.0.0`                      |
| Server ID         | Null                           |
| Server Address    | `255.255.255.255`              |
| Server Address    | `0.0.0.0`                      |

### DHCP Message Types

#### DHCPDISCOVER

Used when the client is looking for an IP address.

```text
Client → DHCPDISCOVER → Broadcast
```

#### DHCPINFORM

Used when the client is looking for infrastructure-related services.

---

# 8. DHCP Relay Agent Option

Enable the DHCP Relay Agent Option to send additional client information toward the DHCP server.

```bash
config system interface
    edit port3
        set dhcp-relay-agent-option enable
    end
```

### Purpose

```text
DHCP Client
     |
     | Additional information
     v
DHCP Relay
     |
     v
DHCP Server
```

> This provides additional user/client information to the DHCP server.

---

# 9. DHCP Relay — Multiple Servers

If multiple DHCP relay servers are configured, FortiGate can send the request to all relay servers.

```bash
config system interface
    edit port3
        set dhcp-relay-request-all-server enable
    end
```

### Behavior

```text
                 +── DHCP Server 1
                 |
DHCP Client → FGT +── DHCP Server 2
                 |
                 +── DHCP Server 3
                 |
                 +── DHCP Server ...
```

> **Note:** Up to **8 servers** can be used in this scenario.

---

## 10. DHCP Relay + Option 82

When DHCP Relay service is enabled:

```text
DHCP Option 82
      ↓
Automatically used
```

> DHCP Option 82 provides additional relay/client information toward the DHCP server.

---

# 11. DHCP Option 42 — NTP Server

**DHCP Option 42** is used to provide NTP server information to DHCP clients.

### Default Behavior

The default behavior uses the FortiGate NTP server.

### Specify NTP Servers

```bash
config system dhcp server
    edit 2
        set ntp-service specify
        set ntp-server1 192.168.20.200
        set ntp-server2 192.168.20.201
    end
```

### Flow

```text
                    DHCP Server
                         |
                         | DHCP Option 42
                         v
                    DHCP Client
                         |
                         |
              +----------+----------+
              |                     |
        NTP Server 1           NTP Server 2
       192.168.20.200         192.168.20.201
```

---

# 12. DHCP Quick Reference

| Feature                   | Configuration / Value                      |
| ------------------------- | ------------------------------------------ |
| DHCP Server               | `config system dhcp server`                |
| DHCP Relay                | Interface → DHCP Server → Relay            |
| DHCP Relay Agent          | `set dhcp-relay-agent-option enable`       |
| Send to all relay servers | `set dhcp-relay-request-all-server enable` |
| DHCP Option 42            | NTP Server                                 |
| DHCP Option 82            | Relay/client information                   |
| DHCP Server Example       | `192.168.20.200`                           |
| NTP Server 1              | `192.168.20.200`                           |
| NTP Server 2              | `192.168.20.201`                           |
| Recommendation            | Dedicated DHCP VM                          |
| Relay Firewall Policy     | Allow required traffic                     |
| NAT                       | Better disabled                            |

---

# 13. DHCP Troubleshooting Flow

```text
                  DHCP Client
                       |
                       | DHCPDISCOVER
                       v
                  FortiGate
                       |
              +--------+--------+
              |                 |
          DHCP Server       Firewall Policy
              |                 |
              |                 |
              +--------+--------+
                       |
                       v
                  DHCP Response
                       |
                       v
                  DHCP Client
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

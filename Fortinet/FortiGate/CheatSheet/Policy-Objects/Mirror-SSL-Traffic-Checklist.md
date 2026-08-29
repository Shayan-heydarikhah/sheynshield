# 🔗 SheynShield Resources

# FortiGate Decrypted Traffic Mirror Checklist  
## SSL Traffic Mirror — SSL/SSH Inspection Visibility

> **FortiOS:** 7.2.x  
> **Topic:** SSL/SSH Inspection, Decrypted Traffic Mirror, Traffic Monitoring  
> **Level:** NSE 6/7 — Advanced Security Engineering  
> **Brand:** SheynShield | Engineering Secure Networks

---

# 📌 Table of Contents

- [1. Decrypted Traffic Mirror Fundamentals](#1-decrypted-traffic-mirror-fundamentals)
- [2. Use Cases](#2-use-cases)
- [3. Prerequisites Checklist](#3-prerequisites-checklist)
- [4. SSL Inspection Requirements](#4-ssl-inspection-requirements)
- [5. Proxy-Based Inspection](#5-proxy-based-inspection)
- [6. Firewall Policy Validation](#6-firewall-policy-validation)
- [7. Decrypted Traffic Mirror Configuration](#7-decrypted-traffic-mirror-configuration)
- [8. Mirror Traffic Parameters](#8-mirror-traffic-parameters)
- [9. Wireshark Validation](#9-wireshark-validation)
- [10. FortiGate Diagnostics](#10-fortigate-diagnostics)
- [11. Troubleshooting Workflow](#11-troubleshooting-workflow)
- [12. Common Mistakes](#12-common-mistakes)
- [13. Security Design](#13-security-design)
- [14. NSE Exam Notes](#14-nse-exam-notes)
- [15. Quick Reference](#15-quick-reference)

---

# 1. Decrypted Traffic Mirror Fundamentals

## What Is Decrypted Traffic Mirror?

**Decrypted Traffic Mirror** allows FortiGate to send a copy of traffic **after SSL/TLS decryption** to an external monitoring system.

Supported encrypted traffic examples:

- [ ] HTTPS
- [ ] SSL/TLS applications
- [ ] SSH traffic

---

## Normal HTTPS Flow

```text
Client

   |
   | Encrypted HTTPS
   v

FortiGate

   |
   | SSL Inspection
   v

Internet Server
````

---

## With Decrypted Traffic Mirror

```text
                    FortiGate

Client
  |
  | HTTPS
  |
  v
+----------------------+
| SSL Proxy            |
|                      |
| TLS Decryption       |
|                      |
| Security Inspection  |
+----------------------+
          |
          |
          v

 Decrypted Traffic Mirror

          |
          v

 Monitoring Host

          |
          v

       Wireshark
```

---

## Core Processing Chain

```text
Encrypted Traffic

        ↓

SSL/SSH Inspection

        ↓

TLS Decryption

        ↓

Security Inspection

        ↓

Decrypted Traffic Mirror

        ↓

External Analysis
```

---

# 2. Use Cases Checklist

Decrypted Traffic Mirror is useful for:

* [ ] Security investigation
* [ ] SOC packet analysis
* [ ] SSL inspection validation
* [ ] IDS/IPS testing
* [ ] Application troubleshooting
* [ ] Malware investigation
* [ ] Encrypted traffic visibility
* [ ] Packet-level debugging

---

## Security Warning ⚠️

Decrypted captures may contain:

* [ ] Passwords
* [ ] Authentication tokens
* [ ] Cookies
* [ ] API keys
* [ ] Personal information
* [ ] Confidential application data

Production requirements:

* [ ] Restrict capture access
* [ ] Protect packet storage
* [ ] Limit retention time
* [ ] Monitor administrator access

---

# 3. Prerequisites Checklist

Before configuration:

```text
[ ] SSL/SSH Inspection enabled

[ ] Proxy-based inspection enabled

[ ] Deep Inspection profile applied

[ ] Decrypted Traffic Mirror enabled

[ ] Monitoring interface available

[ ] Analyzer host connected

[ ] Wireshark/tcpdump ready
```

---

## Important Concept

A normal firewall policy does NOT create decrypted traffic.

Required path:

```text
Firewall Policy

      ↓

Proxy Inspection

      ↓

SSL Deep Inspection

      ↓

Decryption

      ↓

Mirror
```

---

# 4. SSL Inspection Requirements

Navigate:

```text
Security Profiles

        ↓

SSL/SSH Inspection

        ↓

Deep Inspection
```

Checklist:

* [ ] Deep Inspection profile created
* [ ] Certificate configured
* [ ] Client trusts FortiGate CA
* [ ] SSL inspection applied to policy
* [ ] Decrypt Traffic Mirror enabled

---

## Certificate Flow

```text
Client

  |
  | Trust FortiGate CA
  v

FortiGate SSL Proxy

  |
  | Creates inspected TLS session
  v

Internet Server
```

---

# 5. Proxy-Based Inspection Checklist

Decrypted Traffic Mirror depends on the proxy processing model.

---

## Flow-Based

```text
Client

   |

FortiGate

   |

Server
```

Characteristics:

* Inline inspection
* Limited processing visibility

---

## Proxy-Based

```text
Client

   |

FortiGate Proxy

   |

TLS Termination

   |

Decryption

   |

Security Inspection

   |

Traffic Mirror

   |

Server
```

Checklist:

* [ ] Inspection mode = Proxy
* [ ] SSL inspection profile applied
* [ ] Traffic passes through proxy engine

---

# 6. Firewall Policy Validation Checklist

Example:

```bash
config firewall policy
    edit 1
        set name "LAN-to-WAN-SSL-Inspection"
        set srcintf "lan"
        set dstintf "wan"
        set srcaddr "all"
        set dstaddr "all"
        set service "ALL"
        set action accept
        set nat enable
    next
end
```

Validate:

* [ ] Correct source interface
* [ ] Correct destination interface
* [ ] Policy allows traffic
* [ ] SSL inspection profile applied
* [ ] Proxy mode enabled
* [ ] Logging enabled

---

# 7. Decrypted Traffic Mirror Configuration Checklist

Enable:

```text
SSL/SSH Inspection

        ↓

Deep Inspection

        ↓

Decrypt Traffic Mirror
```

---

## Verify Configuration

CLI:

```bash
config firewall decrypted-traffic-mirror
    get
end
```

Checklist:

* [ ] Mirror enabled
* [ ] Correct protocol selected
* [ ] Correct interface selected
* [ ] Correct destination configured

---

# 8. Mirror Traffic Parameters Checklist

## Traffic Type

Available options:

```text
SSL

SSH

Both
```

Checklist:

* [ ] SSL selected for HTTPS analysis
* [ ] SSH selected for SSH analysis
* [ ] Both selected when full visibility required

---

## Traffic Direction

Options:

```text
Client

Server

Both
```

Meaning:

### Client

```text
Client → FortiGate
```

### Server

```text
FortiGate → Server
```

### Both

```text
Client ↔ Server
```

Recommended:

```text
Both
```

for complete analysis.

---

# 9. Mirror Interface Checklist

Traffic destination:

```text
FortiGate

     |

Mirror Interface

     |

Monitoring Host

     |

Wireshark
```

Validate:

* [ ] Monitoring interface exists
* [ ] Analyzer connected
* [ ] Layer-2 connectivity available
* [ ] Correct VLAN configured

---

# 10. Destination MAC Checklist

Destination MAC identifies the monitoring system.

Example:

```text
aa:bb:cc:dd:ee:ff
```

Recommended:

```text
Specific Monitoring MAC
```

Avoid unnecessary broad mirroring:

```text
ff:ff:ff:ff:ff:ff
```

Preferred:

```text
FortiGate

      |

      +----> Monitoring Host MAC

                    |

                    v

                Wireshark
```

---

# 11. Wireshark Validation Checklist

Monitoring host:

* [ ] Connected to mirror interface
* [ ] Capture started
* [ ] Correct interface selected

---

## Capture Filters

HTTPS:

```text
tcp.port == 443
```

SSH:

```text
tcp.port == 22
```

Specific host:

```text
ip.addr == 192.168.101.2
```

TLS:

```text
tls
```

---

# 12. Expected Wireshark Behavior

## Without Decryption

```text
Client

 |

TLS Encrypted Data

 |

Wireshark
```

Result:

```text
Encrypted Payload
```

---

## With Decrypted Traffic Mirror

```text
Client

 |

FortiGate

 |

TLS Decryption

 |

Decrypted Payload

 |

Mirror

 |

Wireshark
```

Result:

```text
Application Visibility
```

---

# 13. FortiGate Diagnostics Checklist

## SSL Manager

```bash
diagnose test application sslmgr
```

Check:

* [ ] SSL engine status
* [ ] Inspection processing
* [ ] Decryption behavior

---

## HTTPS Debug

Enable:

```bash
diagnose debug application httpsd -1

diagnose debug enable
```

Stop:

```bash
diagnose debug disable

diagnose debug reset
```

---

# 14. Troubleshooting Workflow

```text
No Mirrored Traffic

        |

        v

SSL Inspection Enabled?

        |

   +----+----+

   No       Yes

   |         |

Enable   Proxy Mode?

             |

             v

      Deep Inspection?

             |

             v

      Mirror Enabled?

             |

             v

      Correct Protocol?

             |

             v

      Correct Interface?

             |

             v

      Correct MAC?

             |

             v

        Wireshark Capture
```

---

# 15. Common Mistakes Checklist

## ❌ Flow-Based Inspection

Problem:

```text
Flow-based

↓

No proxy decryption workflow
```

Solution:

```text
Proxy-based

↓

Deep Inspection

↓

Mirror
```

---

## ❌ Mirror Without SSL Decryption

Wrong:

```text
TLS

↓

Mirror
```

Correct:

```text
TLS

↓

SSL Inspection

↓

Decrypt

↓

Mirror
```

---

## ❌ Wrong Capture Interface

Check:

* [ ] Wireshark connected to mirror interface
* [ ] Correct VLAN
* [ ] Correct physical port

---

## ❌ Missing Certificate Trust

Check:

* [ ] Client trusts FortiGate CA
* [ ] Certificate warnings absent
* [ ] TLS interception successful

---

# 16. SSL Inspection vs Decrypted Traffic Mirror

| Feature              | SSL Inspection | Decrypted Traffic Mirror  |
| -------------------- | -------------- | ------------------------- |
| TLS Decryption       | ✅              | Depends on SSL inspection |
| Security Inspection  | ✅              | Not primary purpose       |
| External Copy        | ❌              | ✅                         |
| Wireshark Visibility | Limited        | Decrypted                 |
| SOC Analysis         | Limited        | Excellent                 |

---

## Mental Model

```text
SSL Inspection

=

FortiGate decrypts traffic


Decrypted Mirror

=

FortiGate sends decrypted copy externally
```

---

# 17. Secure Monitoring Architecture

```text
                 Internet

                    |

                    v

              +-----------+

              | FortiGate |

              | SSL Proxy |

              +-----------+

                    |

          +---------+---------+

          |                   |

          v                   v

       Server          Mirror Interface

                              |

                              v

                       Monitoring Host

                              |

                              v

                         Wireshark
```

Security controls:

* [ ] Dedicated monitoring network
* [ ] Restricted access
* [ ] Secure capture storage
* [ ] Short retention
* [ ] Access auditing

---

# 18. NSE High Value Notes 🧠

## Processing Chain

```text
Firewall Policy

↓

Proxy Inspection

↓

Deep SSL Inspection

↓

Decrypt

↓

Decrypt Traffic Mirror

↓

Analyzer
```

---

## Exam Questions

### Does a normal firewall policy decrypt HTTPS?

Answer:

```text
No

SSL/SSH Inspection is required.
```

---

### Why proxy mode?

Answer:

```text
Proxy mode allows TLS termination
and decrypted content inspection.
```

---

### Purpose of Decrypted Traffic Mirror?

Answer:

```text
Provide decrypted traffic visibility
to external monitoring tools.
```

---

# 19. Golden Troubleshooting Workflow 🔥

```text
Policy

↓

Proxy Inspection

↓

SSL Profile

↓

Certificate Trust

↓

Decryption

↓

Mirror Configuration

↓

Interface

↓

MAC

↓

Wireshark

↓

Debug
```

---

# 20. Quick CLI Reference

```bash
# Decrypted mirror configuration

config firewall decrypted-traffic-mirror
    get
end


# SSL diagnostics

diagnose test application sslmgr


# HTTPS debug

diagnose debug application httpsd -1


# Enable debug

diagnose debug enable


# Disable debug

diagnose debug disable


# Reset debug

diagnose debug reset
```

---

# 🎯 Final Engineer Mental Model

```text
Encrypted HTTPS

        ↓

FortiGate Proxy

        ↓

SSL Deep Inspection

        ↓

TLS Decryption

        ↓

Security Inspection

        ↓

Decrypted Traffic Mirror

        ↓

Wireshark / SOC Analysis
```

---

# 🔥 One-Line Memory Hook

```text
SSL Inspection

=
Decrypt for FortiGate


Decrypted Traffic Mirror

=
Export decrypted copy for analysis


Proxy Mode

=
Required processing path
```

---

# ⚠️ Production Security Reminder

Decrypted traffic is highly sensitive.

Protect:

* [ ] Mirror network
* [ ] Capture files
* [ ] Analyzer systems
* [ ] Administrator access

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

#FortiGate #FortiOS #Fortinet #SSLInspection #TLS #DecryptedTrafficMirror #Wireshark #NSE7 #NSE6 #CyberSecurity #NetworkSecurity


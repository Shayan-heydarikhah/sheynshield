# FortiGate Geo-IP Anycast Recognition & Geo-Blocking  

> **FortiOS:** 7.2.x
> **Level:** Advanced / Troubleshooting / Security Engineering
> **Topic:** Geo-IP, ISDB, Anycast, Registered Location, Physical Location

---

## 🎯 What This   Solves

When you create a Geo-IP policy such as:

```text
DENY → Country: US
```

you may unexpectedly block services that use **Anycast IP addresses**.

For example:

```text
8.8.8.8
1.1.1.1
```

An Anycast address may be geographically served from different locations while its Geo-IP database classification can still identify it with a specific country.

FortiGate provides mechanisms to distinguish **Anycast traffic** from normal Geo-IP traffic.

---

# 1. Core Concepts

| Feature                 | Purpose                                                  |
| ----------------------- | -------------------------------------------------------- |
| **Geo-IP**              | Maps IP addresses to geographical locations              |
| **Registered Location** | Country associated with the IP registration              |
| **Physical Location**   | Geographical location represented by the Geo-IP database |
| **ISDB**                | FortiGuard Internet Service Database                     |
| **Anycast**             | Same IP advertised from multiple locations               |
| `geoip-anycast`         | Controls matching behavior for Anycast addresses         |
| `geoip-match`           | Controls whether physical or registered location is used |

---

# 2. Why Anycast Matters

A common security policy is:

```text
LAN → WAN
Destination = US
Action = DENY
```

This can produce unexpected results.

For example:

```text
8.8.8.8
```

may be reported as:

```text
Country: US
Registered country: US
Anycast: YES
```

Therefore:

```text
Client → 8.8.8.8
        ↓
FortiGate Geo-IP
        ↓
US
        ↓
DENY
```

The problem becomes more interesting when an Anycast service has:

```text
Physical Location ≠ Registered Location
```

---

# 3. Verify an IP Before Building the Policy

Use:

```bash
diagnose firewall ipgeo ip2country <IP>
```

### Example — Normal IP

```bash
diagnose firewall ipgeo ip2country 185.192.112.25
```

Example output:

```text
185.192.112.25 is in country: IR,
registered country is IR,
is not anycast ip.
```

Interpretation:

```text
Physical = IR
Registered = IR
Anycast = NO
```

---

### Example — Anycast IP

```bash
diagnose firewall ipgeo ip2country 8.8.8.8
```

Example:

```text
8.8.8.8 is in country: US,
registered country is US,
is anycast ip.
```

Interpretation:

```text
Physical = US
Registered = US
Anycast = YES
```

---

# 4. Interesting Anycast Example

A particularly useful test case is:

```bash
diagnose firewall ipgeo ip2country 185.143.233.120
```

Example:

```text
185.143.233.120 is in country: US,
registered country is IR,
is anycast ip.
```

This creates:

```text
Physical Location   = US
Registered Location = IR
Anycast              = YES
```

This distinction is extremely important when troubleshooting Geo-IP policies.

---

# 5. Build a Geo-IP Blocking Policy

Example:

```text
Policy 1
────────────────────────────
Source      = all
Destination = US
Action      = DENY
Logging     = ALL

Policy 2
────────────────────────────
Source      = all
Destination = all
Action      = ACCEPT
Logging     = ALL
```

CLI concept:

```bash
config firewall policy
    edit 1
        set srcintf "port3"
        set dstintf "port1"
        set srcaddr "all"
        set dstaddr "US"
        set action deny
        set schedule "always"
        set service "ALL"
        set logtraffic all
    next

    edit 2
        set srcintf "port3"
        set dstintf "port1"
        set srcaddr "all"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "ALL"
        set logtraffic all
    next
end
```

---

# 6. The Anycast Exception

Suppose the requirement is:

> Block US destinations, but allow selected Anycast services.

A policy can explicitly enable Anycast matching:

```bash
config firewall policy
    edit 1
        set srcintf "port3"
        set dstintf "port1"
        set action accept
        set srcaddr "all"
        set dstaddr "US"
        set schedule "always"
        set service "ALL"
        set geoip-anycast enable
        set logtraffic all
        set nat enable
    end
```

### Concept

```text
                Destination
                     │
                     ▼
                 Geo-IP = US
                     │
              ┌──────┴──────┐
              │             │
          Anycast?        Normal IP
              │             │
             YES           YES
              │             │
       geoip-anycast      Geo-IP
          matching        matching
```

---

# 7. `geoip-anycast`

The important configuration:

```bash
set geoip-anycast enable
```

This controls whether the policy's Geo-IP matching behavior includes **Anycast addresses**.

### Practical use case

You have:

```text
US = BLOCK
```

but a required service uses:

```text
8.8.8.8
```

which is Anycast.

You can explicitly account for Anycast behavior rather than assuming every Geo-IP result represents a conventional geographically located address.

> ⚠️ **Important:** `geoip-anycast` is not a generic "allow this Anycast IP" switch. It changes how Geo-IP matching treats Anycast destinations. Policy ordering and the selected Geo-IP matching mode still matter.

---

# 8. Registered Location vs Physical Location

FortiGate can distinguish between:

```text
Physical / Geo-IP Location
```

and:

```text
Registered Location
```

Example:

```text
IP: 185.143.233.120

Physical Location   = US
Registered Location = IR
Anycast             = YES
```

Therefore the same IP can produce different policy results depending on the matching mode.

---

# 9. Change Geo-IP Matching Mode

Example:

```bash
config firewall policy
    edit 1
        set geoip-match registered-location
    end
```

The policy now uses the **registered location** for Geo-IP matching.

Conceptually:

```text
                 IP
                  │
          ┌───────┴────────┐
          │                │
      Physical         Registered
      Location          Location
          │                │
          US               IR
```

This can completely change the result of a Geo-IP policy.

---

# 10. Global Geo-IP Behavior

Geo-IP behavior can also be influenced globally:

```bash
config system settings
    set geoip ...
end
```

The exact available values depend on the FortiOS release/model.

### Design principle

Before relying on Geo-IP:

```text
1. Determine which location database/mode is being used.
2. Test the destination IP.
3. Determine whether it is Anycast.
4. Check physical vs registered country.
5. Verify the policy's Geo-IP matching behavior.
6. Test the actual traffic.
```

---

# 11. Recommended Troubleshooting Workflow

## Step 1 — Resolve the hostname

For example:

```bash
ping iran.ir
ping time.ir
```

Record the resolved IP address.

Example:

```text
iran.ir
185.143.233.120

time.ir
185.192.112.25
```

---

## Step 2 — Check Geo-IP

```bash
diagnose firewall ipgeo ip2country 185.143.233.120
```

Then:

```bash
diagnose firewall ipgeo ip2country 185.192.112.25
```

---

## Step 3 — Check Anycast

Look for:

```text
is anycast ip.
```

This is critical.

---

## Step 4 — Compare Locations

Record:

```text
Physical country
Registered country
Anycast status
```

Example:

| IP                | Physical | Registered | Anycast |
| ----------------- | -------- | ---------- | ------- |
| `185.192.112.25`  | IR       | IR         | No      |
| `185.143.233.120` | US       | IR         | Yes     |
| `8.8.8.8`         | US       | US         | Yes     |

---

# 12. Why `iran.ir` Can Be Blocked

Consider:

```text
185.143.233.120
```

with:

```text
Physical   = US
Registered = IR
Anycast    = YES
```

If the policy uses:

```text
Physical Geo-IP
```

then:

```text
185.143.233.120
        ↓
Physical = US
        ↓
US DENY
        ↓
BLOCK
```

Even though the IP is registered in Iran.

---

# 13. Changing to Registered Location

If you configure:

```bash
config firewall policy
    edit 1
        set geoip-match registered-location
    end
```

then:

```text
185.143.233.120
        ↓
Registered = IR
        ↓
NOT US
        ↓
Potentially ALLOWED
```

However, if the policy also enables:

```bash
set geoip-anycast enable
```

the Anycast matching logic must be considered as well.

### Critical rule

Do not troubleshoot:

```text
geoip-match
```

in isolation.

Always evaluate:

```text
geoip-anycast
        +
geoip-match
        +
global Geo-IP behavior
        +
policy order
```

---

# 14. Matching Logic — Mental Model

A useful troubleshooting model is:

```text
                Destination IP
                      │
                      ▼
                Geo-IP Database
                      │
            ┌─────────┴─────────┐
            │                   │
       Anycast?              Normal IP
            │                   │
           YES                  │
            │                   │
   geoip-anycast logic          │
            │                   │
            └─────────┬─────────┘
                      ▼
               Geo-IP Match Mode
                      │
              ┌───────┴────────┐
              │                │
          Physical         Registered
           Location          Location
              │                │
              └───────┬────────┘
                      ▼
                Policy Match
                      │
                      ▼
                 ACCEPT/DENY
```

---

# 15. Test Case: `8.8.8.8`

```bash
diagnose firewall ipgeo ip2country 8.8.8.8
```

Expected concept:

```text
Country           = US
Registered        = US
Anycast           = YES
```

If:

```text
US = DENY
```

then:

```text
Client
  │
  ├── DNS request → 8.8.8.8
  │
  ▼
FortiGate
  │
  ▼
Geo-IP = US
  │
  ▼
DENY
```

This is a classic example of why Geo-IP policies must be tested against **Anycast infrastructure**.

---

# 16. Geo-IP + DNS = Hidden Dependency

If FortiGate itself uses:

```text
8.8.8.8
```

as DNS and the policy blocks US destinations:

```text
FortiGate DNS
     │
     ▼
8.8.8.8
     │
     ▼
Geo-IP = US
     │
     ▼
Potentially BLOCKED
```

The result can be:

```text
DNS failure
      ↓
Name resolution failure
      ↓
Multiple applications appear broken
```

### Troubleshooting rule

When a Geo-IP policy is introduced, test:

```text
DNS
NTP
CDN
Cloud services
API endpoints
Security services
Anycast services
```

---

# 17. Useful Test Matrix

| Test                     | What to Check               |
| ------------------------ | --------------------------- |
| `8.8.8.8`                | Anycast + US                |
| `1.1.1.1`                | Anycast + Cloud DNS         |
| `iran.ir` resolved IP    | Physical vs registered      |
| `time.ir` resolved IP    | Normal Geo-IP comparison    |
| `aparat.com` resolved IP | CDN / Anycast behavior      |
| FortiGate DNS server     | Potential policy dependency |

---

# 18. Policy Ordering Still Matters

Remember:

```text
FortiGate = First Matching Policy
```

Therefore:

```text
Policy 1 → Specific Anycast/exception
Policy 2 → Geo-IP DENY
Policy 3 → General ACCEPT
```

is very different from:

```text
Policy 1 → Geo-IP DENY
Policy 2 → Anycast ACCEPT
```

### Recommended design

```text
                Traffic
                   │
                   ▼
          Specific exceptions
                   │
                   ▼
            Geo-IP controls
                   │
                   ▼
            General policies
```

---

# 19. Security Design Recommendation

For Internet egress:

```text
LAN
 │
 ▼
Specific application exceptions
 │
 ▼
Required Anycast services
 │
 ▼
Geo-IP restrictions
 │
 ▼
General Internet policy
```

For Internet ingress:

```text
WAN
 │
 ▼
Geo-IP restriction
 │
 ▼
VIP / DNAT
 │
 ▼
Security profiles
 │
 ▼
DMZ
```

A common security strategy is to restrict unwanted source countries such as:

```text
Russia
China
```

on Internet-facing policies **when justified by the organization's threat model and business requirements**.

---

# 20. Common Mistakes

### ❌ Mistake 1 — Assuming Geo-IP means physical geography

```text
Geo-IP = physical GPS location
```

Not necessarily.

---

### ❌ Mistake 2 — Ignoring Anycast

```text
8.8.8.8
1.1.1.1
```

are good examples of addresses that require Anycast awareness.

---

### ❌ Mistake 3 — Looking only at registered country

An IP can have:

```text
Physical = US
Registered = IR
```

---

### ❌ Mistake 4 — Changing `geoip-match` without checking Anycast

Always evaluate:

```text
geoip-match
+
geoip-anycast
```

together.

---

### ❌ Mistake 5 — Forgetting policy order

Geo-IP policies still participate in normal FortiGate policy matching and ordering.

---

# 21. Fast Troubleshooting Commands

### IP → Country / Anycast

```bash
diagnose firewall ipgeo ip2country <IP>
```

### Flow debugging

```bash
diagnose debug flow filter addr <IP>
diagnose debug flow filter proto 1

diagnose debug enable
diagnose debug flow trace start 50
```

Stop debugging:

```bash
diagnose debug disable
diagnose debug flow trace stop
```

---

# 22. Production Checklist

* [ ] Identify the destination IP.
* [ ] Resolve the FQDN first.
* [ ] Run `ip2country`.
* [ ] Check **physical country**.
* [ ] Check **registered country**.
* [ ] Check **Anycast status**.
* [ ] Check the policy's `geoip-anycast`.
* [ ] Check `geoip-match`.
* [ ] Check global Geo-IP configuration.
* [ ] Check policy order.
* [ ] Check DNS dependencies.
* [ ] Test both allowed and denied destinations.
* [ ] Review traffic logs.
* [ ] Avoid broad Geo-IP exceptions unless they are justified.
* [ ] Document critical Anycast services.

---

# 🧠 Engineer's Takeaway

> **Geo-IP filtering is not simply "IP → Country → Allow/Deny".**

The real decision path can involve:

```text
IP
 ↓
Geo-IP Database
 ↓
Physical Location
        +
Registered Location
        +
Anycast Detection
 ↓
geoip-anycast
 ↓
geoip-match
 ↓
Policy Matching
 ↓
ACCEPT / DENY
```

When a Geo-IP policy unexpectedly blocks a legitimate service, **check Anycast before assuming the Geo-IP database is wrong.**

---

## 🔥 Interview / NSE Exam Notes

### Q: How do you check whether an IP is Anycast?

```bash
diagnose firewall ipgeo ip2country <IP>
```

Look for:

```text
is anycast ip.
```

### Q: How can you use registered location for policy matching?

```bash
set geoip-match registered-location
```

### Q: What is the key problem with Geo-IP + Anycast?

The same Anycast IP can represent a globally distributed service while Geo-IP classification may associate the address with a particular physical or registered country.

### Q: What should you check when Geo-IP produces an unexpected result?

```text
1. IP
2. Geo-IP database result
3. Physical country
4. Registered country
5. Anycast status
6. geoip-anycast
7. geoip-match
8. Policy order
```

---

## 📌 One-Line Memory Hook

```text
GEO-IP ≠ GEOLOCATION

Always think:

Physical + Registered + Anycast + Match Mode + Policy Order
```

---

**SheynShield | Engineering Secure Networks**

#FortiGate #FortiOS #Fortinet #GeoIP #Anycast #ISDB #NetworkSecurity #Firewall #CyberSecurity #NSE7 #NSE4 #FortiGuard #ZeroTrust

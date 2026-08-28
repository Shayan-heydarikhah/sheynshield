# 🔐 FortiGate IPsec — Tunnel Monitoring & Passive Mode

---

## 1. IPsec Tunnel Monitoring

### 🎯 Purpose

Use **IPsec tunnel monitoring** to provide a simple primary/backup mechanism between two **site-to-site IPsec tunnels**.

> ⚠️ Tunnel monitoring is mainly useful for **site-to-site/static IPsec**.
> It is not intended as a monitoring mechanism for dial-up IPsec tunnels.

### 🗺️ Topology

```text
                 ┌───────────────┐
                 │    FGT-1      │
                 └───────┬───────┘
                         │
              ┌──────────┴──────────┐
              │                     │
          Link-1                 Link-2
       Primary Tunnel          Backup Tunnel
              │                     │
        2.2.2.1                22.22.22.1
              │                     │
              └──────────┬──────────┘
                         │
                    Remote FGT
````

---

## 2. FGT-1 — IPsec Link-1

### Phase 1

```text
IPsec Type       : Custom
Remote Gateway   : Static
Remote IP        : 2.2.2.1
Interface        : ISP / WAN
Pre-shared Key   : 123456

IKE Version      : IKEv1
Mode             : Aggressive
Peer ID          : Any

Encryption       : DES
Integrity        : MD5
DH Group         : 5
```

### Phase 2

```text
Encryption       : DES
Integrity        : MD5
PFS              : DH Group 5
Auto-negotiate   : Enable
Selectors        : All required subnets
```

---

## 3. FGT-1 — IPsec Link-2

### Phase 1

```text
IPsec Type       : Custom
Remote Gateway   : Static
Remote IP        : 22.22.22.1
Interface        : ISP / WAN
Pre-shared Key   : 123456

IKE Version      : IKEv1
Mode             : Aggressive
Peer ID          : Any

Encryption       : DES
Integrity        : MD5
DH Group         : 5
```

### Phase 2

```text
Encryption       : DES
Integrity        : MD5
PFS              : DH Group 5
Auto-negotiate   : Enable
Selectors        : All required subnets
```

---

# 4. Configure Tunnel Monitoring

Configure **Link-2** to monitor **Link-1**:

```bash
config vpn ipsec phase1-interface
    edit "link-2"
        set monitor "link-1"
    next
end
```

### Monitoring Relationship

```text
             Link-1
            PRIMARY
               ▲
               │
            monitored
               │
               │
             Link-2
            BACKUP
```

### Important

```text
set monitor link-1
```

means:

```text
Link-2 monitors Link-1
```

It does **not** automatically create bidirectional monitoring.

> ⚠️ Monitoring is effectively a **one-way relationship** unless you deliberately configure the opposite direction as well.

---

# 5. Tunnel Monitoring Behavior

After monitoring is configured:

```text
                Link-1
               PRIMARY
                  │
                  │ UP
                  ▼
             Link-2 STANDBY
```

If the monitored tunnel fails:

```text
                Link-1
               PRIMARY
                  │
                  │ DOWN
                  ▼
             Link-2 ACTIVE
                  │
                  ▼
             Traffic uses
             backup path
```

### Design Concept

```text
Normal State:

Link-1  →  ACTIVE
Link-2  →  STANDBY


Failure:

Link-1  →  DOWN
Link-2  →  ACTIVE
```

---

# 6. Routing Considerations

Tunnel monitoring alone does **not** replace routing design.

After configuring monitoring:

### Check

```text
✓ Static routes
✓ Route priorities
✓ Administrative distance
✓ SD-WAN rules
✓ Routing protocol behavior
✓ Return path
✓ Tunnel state
```

### Example

```text
Primary:

192.168.102.0/24
        │
        ▼
      Link-1
        │
      AD 10
```

```text
Backup:

192.168.102.0/24
        │
        ▼
      Link-2
        │
      AD 20
```

Therefore:

```text
Link-1 UP
    ↓
Use Link-1

Link-1 DOWN
    ↓
Use Link-2
```

> 💡 Use **SD-WAN or appropriate routing mechanisms** when more advanced path selection and failover logic are required.

---

# 7. Configure the Remote Device

The same IPsec tunnel design must be configured on the remote FortiGate.

```text
FGT-1                         FGT-2
 │                              │
 ├──── Link-1 ──────────────────┤
 │                              │
 └──── Link-2 ──────────────────┘
```

Make sure:

```text
✓ Phase 1 proposals match
✓ Phase 2 proposals match
✓ PSK matches
✓ DH/PFS matches
✓ IPsec selectors match
✓ Routing is correct
```

---

# 8. Passive Mode

## 🎯 Purpose

**Passive mode** makes the FortiGate behave as a responder rather than actively initiating the IPsec negotiation.

```text
Normal:

FGT-1 ─────► Initiates
FGT-2 ─────► Responds
```

With passive mode:

```text
FGT-1
  │
  │ waits
  ▼
FGT-2
  │
  │ waits
  ▼

No side actively initiates
```

---

# 9. Passive Mode Configuration

### FGT-1

```bash
config vpn ipsec phase1-interface
    edit "link-1"
        set rekey enable
        set passive-mode enable
        set passive-tunnel-interface enable
    next
end
```

### FGT-2

```bash
config vpn ipsec phase1-interface
    edit "link-1"
        set rekey enable
        set passive-mode enable
        set passive-tunnel-interface enable
    next
end
```

---

# 10. Passive Mode Parameters

```text
set rekey enable
```

Allows the tunnel to perform the rekey mechanism.

```text
set passive-mode enable
```

Makes the FortiGate passive during IKE negotiation.

```text
set passive-tunnel-interface enable
```

Allows the tunnel interface to operate in passive mode.

---

# 11. ⚠️ Critical Passive Mode Warning

Do **NOT** blindly enable passive mode on both sides.

```text
FGT-1
passive-mode = enable
        │
        ▼
waits for negotiation

FGT-2
passive-mode = enable
        │
        ▼
waits for negotiation
```

Result:

```text
FGT-1  ───── waits ─────┐
                        │
                        │
FGT-2  ───── waits ─────┘

          ❌
      No initiator
```

Therefore:

```text
Passive + Passive
       ↓
No side initiates
       ↓
IKE negotiation may not start
       ↓
Tunnel remains DOWN
```

> ⚠️ Be very careful when configuring `passive-mode` on both peers.

---

# 12. Passive vs Active

| Feature          | Active Mode  | Passive Mode               |
| ---------------- | ------------ | -------------------------- |
| Initiates IKE    | ✅ Yes        | ❌ No                       |
| Responds to IKE  | ✅ Yes        | ✅ Yes                      |
| Waits for peer   | ❌            | ✅                          |
| Rekey            | ✅            | Can be enabled             |
| Auto negotiation | Can initiate | Does not actively initiate |
| Typical use      | Normal IPsec | Responder/passive design   |

---

# 13. Important Difference

### Active

```text
Traffic / auto-negotiate / rekey
            │
            ▼
     FortiGate initiates
            │
            ▼
       IKE negotiation
            │
            ▼
        IPsec SA
```

### Passive

```text
FortiGate waits
      │
      ▼
Peer initiates
      │
      ▼
IKE negotiation
      │
      ▼
IPsec SA
```

---

# 14. Troubleshooting Commands

## Check IPsec tunnels

```bash
diagnose vpn tunnel list
```

Useful for checking:

```text
✓ Tunnel state
✓ Phase 2 SA
✓ SPI
✓ Encryption
✓ Authentication
✓ Traffic counters
✓ Local / remote selectors
```

---

## Check IKE gateways

```bash
diagnose vpn ike gateway list
```

Useful for checking:

```text
✓ IKE SA
✓ Peer address
✓ IKE version
✓ Authentication
✓ Proposal
✓ Tunnel state
```

---

# 15. Quick Troubleshooting Flow

```text
IPsec tunnel DOWN
       │
       ▼
Check Phase 1
       │
       ├── PSK?
       ├── IKE version?
       ├── Encryption?
       ├── Integrity?
       ├── DH group?
       └── Peer IP?
       │
       ▼
Check Phase 2
       │
       ├── Encryption?
       ├── Integrity?
       ├── PFS?
       ├── Selectors?
       └── Auto-negotiate?
       │
       ▼
Check Monitoring
       │
       ├── Correct monitored tunnel?
       └── Correct direction?
       │
       ▼
Check Routing
       │
       ├── Static routes?
       ├── Priority?
       ├── SD-WAN?
       └── Return path?
       │
       ▼
Check Policies
       │
       ├── Source?
       ├── Destination?
       ├── Service?
       └── NAT?
       │
       ▼
Check Logs / Debug
```

---

# 16. Key Takeaways

> [!IMPORTANT]
> **Tunnel monitoring is primarily a site-to-site IPsec failover mechanism.**

> [!WARNING]
> **Do not rely on tunnel monitoring alone. Routing and route preference must also be designed correctly.**

> [!WARNING]
> **If both peers are configured as passive, neither side may initiate IKE negotiation.**

> [!TIP]
> Use `diagnose vpn tunnel list` and `diagnose vpn ike gateway list` as the first checks when troubleshooting tunnel state.

### Mental Model

```text
IPsec Failover
      │
      ├── Tunnel-1
      │      └── Primary
      │
      ├── Tunnel-2
      │      └── Backup
      │
      ├── Monitor
      │      └── Detect tunnel state
      │
      └── Routing
             └── Select usable path
```

---

# 📌 Quick Reference

```text
# Tunnel monitoring

config vpn ipsec phase1-interface
    edit "link-2"
        set monitor "link-1"
    next
end


# Passive mode

config vpn ipsec phase1-interface
    edit "link-1"
        set rekey enable
        set passive-mode enable
        set passive-tunnel-interface enable
    next
end


# Tunnel status

diagnose vpn tunnel list


# IKE status

diagnose vpn ike gateway list
```

---

## 🧠 Exam / Interview Notes

```text
Tunnel Monitoring
    ↓
Site-to-Site IPsec
    ↓
Primary / Backup behavior

set monitor link-1
    ↓
link-2 monitors link-1
    ↓
One-way monitoring relationship

Passive Mode
    ↓
FortiGate does not initiate IKE
    ↓
Waits for peer request

Passive on both sides
    ↓
No active initiator
    ↓
Potential tunnel establishment problem

Failover
    ↓
Tunnel state + Routing
    ↓
Both must be considered
```

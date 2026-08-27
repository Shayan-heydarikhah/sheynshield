# FortiGate High Availability (HA) & Redundancy — Operational Cheat Sheet

A condensed, production-focused engineering reference covering FortiGate Clustering Protocol (FGCP), FortiGate Session Life Support Protocol (FGSP), VRRP, and Advanced Failover Mechanics.

---

## 📌 1. Primary Selection & The Override Trap

### Master Election Hierarchy

FortiOS determines the Primary unit based on the following fallback criteria:

1. **Active Monitored Interfaces** (Highest number of operational monitored links)
2. **HA Uptime** (Unit running longer by $\ge$ 5 minutes) *(Default primary tie-breaker)*
3. **Device Priority** (Higher numerical value wins)
4. **Serial Number** (Alphabetically/numerically higher serial number)

---

### The `override` Trap & Synchronization Warning

Enabling `override` shifts the election criteria order, placing **Device Priority** ahead of **HA Uptime**:

$$\text{Monitored Links} \longrightarrow \mathbf{Priority} \longrightarrow \text{Uptime} \longrightarrow \text{Serial Number}$$

```text
config system ha
    set override enable
    set priority 200
end

```

> ⚠️ **CRITICAL GOTCHA:**
> **`ha override` and `device priority` are NOT synchronized across the cluster.** They belong to FortiOS Non-Synchronized Configurations.
> * **Rules:**
> * You **must** manually set `override` and `priority` on both units.
> * If `override` is enabled on Unit-1 but disabled on Unit-2, role preemption during failover/recovery will become unpredictable.
> * Disabling/enabling `override` or modifying priority dynamically on a active peer requires intentional execution sequence to avoid unplanned failover flapping.
> 
> 
> 
> 

---

## 💓 2. Heartbeat & Timing Parameters

Aggressive signaling reduces failover time but increases susceptibility to **false positives** caused by high CPU spikes or transient switch port delays.

```text
config system ha
    set group-id 2                      # Must match across cluster members
    set hb-interval 2                   # Signal interval frequency multiplier
    set hb-interval-in-milliseconds 10ms # Base unit timer (10ms - 100ms)
    set hb-lost-threshold 2             # Missed packets before triggering failover
end

```

---

## 🛠 3. Reserved Management Interfaces (`ha-mgmt-interfaces`)

Out-of-band management interfaces maintain **unique physical MAC addresses** and static IP identities per physical unit, preventing Virtual MAC routing conflicts during administration.

```text
config system ha
    set ha-direct enable
    set ha-mgmt-status enable
    config ha-mgmt-interfaces
        edit 1
            set interface "mgmt1"
            set gateway 192.168.10.1
        next
    end
end

```

### Services Utilizing `ha-direct`:

* Remote Logging (Syslog, FortiAnalyzer, FortiCloud)
* Remote Authentication & Certificate Verification
* FortiSandbox Communication
* NetFlow / sFlow Samplers
* SNMP Traps and Queries

```text
# Enabling ha-direct for SNMP host communications
config system snmp community
    edit 1
        config hosts
            edit 1
                set ha-direct enable
            next
        end
    end

```

---

## 💾 4. Memory & SSD Failover Controls

Protect cluster integrity against memory leaks or hardware storage crashes:

```text
config system ha
    # SSD Storage Failover (Useful for proxy-base, DLP, and Sandbox offloading)
    set ssd-failover enable

    # Memory Conserve Mode Failover
    set memory-based-failover enable
    set memory-failover-threshold 0        # Triggers when 0% memory remains (Conserve Mode)
    set memory-failover-monitor-period 60  # Check interval window
    set memory-failover-sample-rate 5      # Sampling count per period
    set memory-failover-flip-timeout 600   # Stabilizing delay to avoid flapping
end

```

---

## 🌐 5. Virtual MAC (VMAC) & GARP Mechanics

### VMAC Address Calculation Structure

FGCP derives Virtual MAC addresses based on the Cluster Group ID and Interface Index:

$$\texttt{00-09-0F-09} \ \ \vert{}\ \ \mathbf{Group\ ID\ (Hex)} \ \ \vert{}\ \ \mathbf{vCluster\ ID} \ \ \vert{}\ \ \mathbf{Interface\ Index}$$

```text
Group ID: 0 (Hex: 00)  |  vCluster: 1 (Hex: 00)  |  Interface Index: 3 (port4)
Resulting VMAC = 00-09-0F-09-00-03

```

### Gratuitous ARP (GARP) Tuning

Accelerate upstream switch forwarding-table updates after a role change:

```text
config system ha
    set gratuitous-arps enable
    set arps 5                          # Number of GARP packets sent on failover
    set arps-interval 8                 # Interval between GARP packets (seconds)
    set link-failed-signal disable      # Sends link-down state to connected switches on failover
end

```

---

## ⚙️ 6. Session Synchronization (FGCP vs. FGSP)

### FGCP Stateful Failover Options

```text
config system ha
    set session-pickup enable
    set session-pickup-expectation enable     # Sync L7 helper state (FTP, SIP)
    set session-pickup-connectionless enable  # Sync UDP and ICMP session tables
    set session-pickup-nat enable             # Sync NAT translations
    set sync-packet-balance enable            # Multithread session sync across CPUs
end

```

### FGSP (FortiGate Session Life Support Protocol)

Used when external load balancers handle traffic routing across standalone cluster nodes (2 to 16 nodes):

```text
config system cluster-sync
    edit 1
        set peerip 10.100.10.2
        set peervd "root"
        set syncvd "root"
        set ike-monitor enable
        set ike-monitor-interval 15
    next
end

config system standalone-cluster
    set standalone-group-id 1
    set group-member-id 0
    set session-sync-dev "port5"
    set encryption enable
    set psksecret MyClusterSecretKey123
end

```

---

## 🔀 7. VRRP (Virtual Router Redundancy Protocol)

Used for multi-vendor or dynamic gateway redundancy on shared L2 domains:

```text
config system interface
    edit "vlan10"
        set vrrp-virtual-mac enable     # Employs 00-00-5E-00-01-VRID
        config vrrp
            edit 1
                set vrip 192.168.10.1
                set priority 255
                set adv-interval 1
                set preempt enable
                set vrdst 8.8.8.8       # Tracked IP address (Ping Server)
                set vrdst-priority 10   # Priority drop upon tracking failure
                set vrgrp 10            # VRRP synchronization domain group
            next
        end
    next
end

```

---

## 🔄 8. Non-Synchronized Configurations Checklist

The following items are **never synchronized** between HA nodes and must be managed individually:

* [ ] System Hostname
* [ ] GUI Dashboard Widgets
* [ ] `ha override` setting
* [ ] `device priority` setting
* [ ] Virtual Cluster Priority
* [ ] HA Reserved Management Interface IP & Gateway
* [ ] Ping Server / Gateway Dead Detection parameters
* [ ] Licensing (Entitlements must match, but licenses are applied per-unit)

---

## 🔍 9. Troubleshooting & Inspection CLI Cheat Sheet

```bash
# Check HA Cluster Status & State Engine
get system ha status

# View Cluster Checksums (Verify Sync State across members)
diagnose sys ha checksum autoscale-cluster

# Dump MAC address mapping (Physical vs. Virtual MACs)
diagnose sys ha mac

# Inspect Session Sync statistics
get test hasync 50

# View IPS Sync State & Shared Tables
diagnose ips share list
diagnose ips session list

# Force Manual Failover (Testing purposes only!)
execute ha failover set 1
execute ha failover unset 1

# Manage Cluster Member CLI directly from Primary
execute ha manage <member-index> <admin-username>

# Initiate manual sync sync process
execute ha sync start

# Check VRRP State Tables
get router info vrrp
get system vrrp

```

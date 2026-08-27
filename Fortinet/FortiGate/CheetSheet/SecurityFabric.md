برای اینکه فایل کاملاً آمادهٔ کپی و پیست در گیت‌هاب باشد و **هیچ نیازی به ادیت، تغییر فونت یا اصلاح مارک‌داون نداشته باشی**، کل داکیومنت را داخل یک Block Code یکپارچه قرار داده‌ام.

تمامی استانداردهای پیشرفتهٔ گیت‌هاب (مانند **GitHub Alert Callouts**، **Tables**، **Syntax Highlighting**، و **Diagrams**) کاملاً رعایت شده‌اند. کافی است یک فایل جدید به اسم `fortigate-security-fabric-cheat-sheet.md` در ریپازیتوری ایجاد کنی و کل محتوای داخل باکس زیر را مستقیم در آن کپی کنی:

```markdown
# 🛡️ FortiGate Security Fabric, Automation & External Connectors — Cheat Sheet

A production-grade operational reference for **Fortinet Security Fabric**, **Automation Stitches**, **External Threat Feeds**, **Purdue / OT (ICS) Security Models**, and **SAML SSO / IdP Integration**.

---

## ⚡ 1. Automation Stitches & Process Tuning

Automation stitches can execute commands sequentially or concurrently. For operations involving firmware/license updates or configuration restores, **sequential execution** is strictly required to prevent race conditions.

```text
config system automation setting
    set max-concurrent-stitches 128    # Range: 32-256 (Default: 128)
end

```

> [!TIP]
> **Best Practice:** Use sequential execution mode with `.pkg` signature files when automating configuration or image restores.

### 🔍 Automation Diagnostics & Debugging Commands

```bash
# Test a specific scheduled automation stitch
diagnose automation test <schedule_name>

# Verify automation daemon (autod) health
diagnose test application autod 1

# Enable full verbose debugging for automation events
diagnose debug application autod -1
diagnose debug enable

```

---

## 🌐 2. Threat Feeds & External Connectors

Threat Feeds dynamically import external IP/Domain/URL blocklists via HTTP, HTTPS, or STIX/TAXII protocols.

### 📐 System Limits & Formatting Standards

* **Maximum Entries:** `131,072` per file.
* **Maximum File Size:** `10 MB` (varies by model hardware spec).
* **Maximum Dynamic Categories:** Up to `30` URL categories.
* **Formatting Rules:**
* **IP Networks:** Use standard CIDR notation (`192.168.254.0/24`). Do **NOT** use square brackets for IPv4.
* **IPv6 URLs:** Square brackets **ARE** required (`http://[2001:db8::1]/list.txt`).
* **Wildcards:** Wildcard domains are fully supported (`*.malicious-domain.com`).
* **Optimization:** Avoid duplicate entries to minimize CPU and RAM compilation overhead.



```bash
# Check resource table limits (Global vs. Per-VDOM allocations)
print tablesize

```

> [!NOTE]
> **VDOM Category Identifiers:**
> * `g-cat-192` $\rightarrow$ Global Object
> * `cat-192` $\rightarrow$ Root / VDOM-specific Object
> 
> 

### 🛠️ External Connector Troubleshooting CLI

```bash
# Verify dynamic IP address lists resolved by SDN connectors
diagnose firewall dynamic list

# Debug Kubernetes Connector (kubed process)
diagnose debug application kubed -1
diagnose debug enable

# Debug Symantec Endpoint Protection Manager (sepmd process - Port 8446)
diagnose debug application sepmd -1
diagnose debug enable

# Inspect unauthenticated user tracking state
diagnose firewall auth list

# Troubleshoot WAD / KDC Auto-Discovery (Web Application Database)
diagnose wad debug enable category all
diagnose wad debug enable level verbose
diagnose debug enable
diagnose wad user exchange test-auto-discover

```

---

## 🔗 3. Security Fabric Architecture & Object Synchronization

### 🔌 Core Interface Requirements

To establish a Fabric connection to a Root FortiGate or FortiManager, you **must** enable both LLDP and Fabric Management access on the interconnecting interfaces:

```text
config system interface
    edit "port1"
        set allowaccess ping https ssh fabric
        set lldp-reception enable
        set lldp-transmission enable
    next
end

```

### 🔀 Fabric Object Unification vs. Local Sync

```text
config system csf
    set fabric-object-unification {default | local}
    set configuration-sync local
    set fabric-workers 2                # Synchronization workers (1-4, Default: 2)
end

```

> [!WARNING]
> **Multi-VDOM Limitation:** Security Fabric automatic object synchronization does **NOT** run on multi-mode VDOM environments.

### 📊 Fabric Diagnostics Cheat Sheet

| Action | Command |
| --- | --- |
| **Check Pending Authorizations** | `diagnose sys csf authorization pending-list` |
| **View Downstream Devices** | `diagnose sys csf downstream` |
| **View Upstream Root Device** | `diagnose sys csf upstream` |
| **Check FortiNAC Status** | `diagnose sys csf downstream-devices fortinac` |

---

## 🏭 4. Operational Technology (OT) & Purdue Model (IEC 62443)

The Purdue Enterprise Reference Architecture (PERA / IEC 62443) provides structured network segmentation for Industrial Control Systems (ICS) and Operational Technology (OT).

```text
+-------------------------------------------------------------------+
| Level 4: Enterprise Network (Business Apps, ERP, IT Cloud Access) |
+-------------------------------------------------------------------+
| DMZ: Secure Buffer (NGFW, IPS, SIEM, Proxies, MFA/PAM)            |
+-------------------------------------------------------------------+
| Level 3: Manufacturing Operations (MES, Historians, Process Mgmt) |
+-------------------------------------------------------------------+
| Level 2: Supervisory Control (SCADA, HMIs, Alarm Monitoring)      |
+-------------------------------------------------------------------+
| Level 1: Basic Control (PLCs, RTUs, Automation Controllers)       |
+-------------------------------------------------------------------+
| Level 0: Physical Process (Sensors, Actuators, Solenoids, Motors) |
+-------------------------------------------------------------------+

```

### ⚙️ FortiGate OT & Purdue Configuration

* **OT Visibility:** Enable under `System > Feature Visibility > OT`.
* **Inspection Level:** FortiGate and FortiSwitch inspect traffic primarily at **Level 2** and **Level 3**.

```bash
# Adjust Purdue level IP memory allocation thresholds
diagnose user-device-store device memory ot-purdue-set max ip level

```

---

## 🔐 5. SAML SSO & FortiAuthenticator Integration

Security Fabric relies on **SAML 2.0** for single sign-on authentication across cluster nodes.

* **Root FortiGate:** Acts as the **Identity Provider (IdP)** (or bridges to ADFS/Okta).
* **Downstream FortiGate:** Acts as a **Service Provider (SP)** redirecting auth requests to the Root.

```text
# FortiClient EMS Fabric Integration Configuration
config endpoint-control fctems
    edit "ems1"
        set fortinetone-cloud-authentication disable
        set server "192.168.254.220"
        set https-port 443
        set pull-sysinfo enable
        set pull-vulnerabilities enable
        set pull-avatars enable
        set pull-tags enable
        set pull-malware-hash enable
        set capabilities fabric-auth silent-approval websocket push-ca-cert
        set call-timeout 30
    next
end

```

### 🔍 SAML & Endpoint Debugging

```bash
# Verify Certificate Sync across HA cluster for EMS
diagnose endpoint fctems json deep-inspect-cert-sync

# Check FortiClient Network Access Daemon (fcnacd) WebSocket connections
diagnose test application fcnacd 2

# Verify dynamic ZTNA posture tags on FortiGate
diagnose firewall dynamic list

```

---

## ☁️ 6. FortiCloud & FortiAnalyzer Cloud Management

### 🎛️ VDOM Override Settings for FortiAnalyzer Cloud

```text
config log setting
    set faz-override enable
end

config log fortianalyzer-cloud override-setting
    set status enable
end

config log fortianalyzer-cloud override-filter
    set severity information
    set forward-traffic enable
    set local-traffic disable
    set multicast-traffic disable
    set sniffer-traffic disable
    set anomaly enable
    set voip disable
    set dlp-archive disable
end

```

### 🚀 Sandbox Operations & Log Queries

```text
config system fortiguard
    set sandbox-region 0    # 0: Europe | 1: Global | 2: Japan | 3: US
end

```

```bash
# Manual FortiCloud Sandbox update
execute forticloud-sandbox update

# View FortiAnalyzer Cloud logs from CLI
execute log filter device fortianalyzer-cloud
execute log filter category event
execute log display

```

---

## 📈 7. Security Rating & Maintenance

```text
# Disable automatic scheduled Security Rating execution
config system global
    set security-rating-run-on-schedule disable
end

```

```bash
# Manually trigger Security Rating calculation report
diagnose report-runner trigger

# Check FortiCloud subscription entitlement status
diagnose test update info

```

```

```

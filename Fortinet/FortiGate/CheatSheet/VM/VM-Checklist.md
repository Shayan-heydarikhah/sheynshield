# 🛡️ FortiGate-VM — Deployment, Licensing, Cloud & Performance Checklist

> **SheynShield Technical Checklist**  
> **FortiOS Focus:** FortiGate-VM Operations, Licensing, FortiFlex, VDOM, Terraform, SR-IOV, Metadata Service, FIPS  
>
> **Purpose:** Production-ready validation checklist for FortiGate-VM deployment, troubleshooting, automation, and cloud security.

---

# 📋 FortiGate-VM Pre-Deployment Checklist

## ✅ Platform Preparation

- [ ] Verify supported hypervisor/cloud platform

Supported environments:

- [ ] VMware
- [ ] Hyper-V
- [ ] KVM
- [ ] AWS
- [ ] Azure
- [ ] OCI
- [ ] GCP

---

## ✅ Compute Resource Validation

Verify allocated resources:

```bash
get system status
````

Check:

* [ ] vCPU allocation
* [ ] vRAM allocation
* [ ] VM model capacity
* [ ] License-supported resources

Example:

```text
VM Resources:
1 CPU / 4 allowed
2006 MB RAM
```

---

# 🔐 FortiGate-VM License Checklist

## ✅ License Status Verification

Check:

```bash
diagnose debug vm-print-license
```

Validate license state:

| Status     | Action                        |
| ---------- | ----------------------------- |
| ✅ Valid    | Normal operation              |
| ⚠️ Warning | Check FortiGuard connectivity |
| ❌ Invalid  | License issue required        |
| ⏳ Pending  | Wait for validation           |

---

# 🧪 License Troubleshooting Checklist

## Check VM License Information

```bash
diagnose debug vm-print-license
```

* [ ] Confirm license serial
* [ ] Confirm license status
* [ ] Confirm expiration
* [ ] Confirm duplicate license condition

---

## Check Complete VM Information

```bash
diagnose hardware sysinfo vm full
```

Validate:

* [ ] VM UUID
* [ ] Hypervisor information
* [ ] Resource allocation
* [ ] License information

---

## Debug License Update

Enable debugging:

```bash
diagnose debug application update -1
diagnose debug enable
```

Trigger update:

```bash
execute update-now
```

Disable debugging:

```bash
diagnose debug disable
```

---

# 📦 FortiGate-VM License Types Checklist

## V-Series / Perpetual

* [ ] Verify VM base license
* [ ] Verify FortiCare contract separately
* [ ] Confirm supported vCPU level

---

## S-Series Subscription

Validate:

* [ ] VM subscription
* [ ] FortiCare bundle
* [ ] Service package

Available bundles:

* [ ] FortiCare Only
* [ ] UTM
* [ ] Enterprise
* [ ] ATP

---

## FortiFlex Checklist

Validate:

* [ ] FortiFlex entitlement
* [ ] Daily point consumption
* [ ] Token validity
* [ ] License retrieval

Example:

```bash
execute vm-license <token>
```

---

# 🚨 License Error Code Checklist

## FortiCare Errors

| Code | Meaning              |
| ---- | -------------------- |
| 1    | Runtime/server error |
| 57   | Invalid token        |
| 58   | Token already used   |

---

## FortiGate Errors

| Code | Meaning                  |
| ---- | ------------------------ |
| 60   | Failed FortiCare request |

---

# 📤 Manual License Upload Checklist

Use when FortiGate-VM cannot reach FortiGuard.

GUI:

```text
System
 └── FortiGuard
```

Checklist:

* [ ] Download `.lic` file
* [ ] Upload license
* [ ] Reboot VM
* [ ] Verify FortiGuard validation

---

# 🔄 License Registration Workflow

```text
Purchase VM License
        |
        ▼
Receive Registration Code
        |
        ▼
Register in Fortinet Support Portal
        |
        ▼
Generate Serial Number
        |
        ▼
Download License File
        |
        ▼
Upload to FortiGate-VM
        |
        ▼
Reboot
        |
        ▼
Validate License
```

---

# 🔑 SCP License Upload Checklist

Enable SCP:

```bash
config system global
    set admin-scp enable
end
```

Enable SSH:

```bash
config system interface
edit port1
append allowaccess ssh
end
```

Upload:

```bash
scp license.lic admin@<IP>:vmlicense
```

Validation:

* [ ] License uploaded
* [ ] VM rebooted
* [ ] Status changed to valid

---

# 🧩 VDOM Licensing Checklist

## Verify Current VDOM Capacity

```bash
get system status
```

Check:

```text
Maximum number of virtual domains
```

---

## Add VDOM License

Apply:

```bash
execute upd-vd-license <license-key>
```

Verify:

```bash
get system status
```

---

## VDOM Deployment Validation

* [ ] Base license installed
* [ ] VDOM upgrade license purchased
* [ ] License uploaded successfully
* [ ] Maximum VDOM count verified

---

# ⚙️ Terraform + FortiGate Automation Checklist

## Required Components

* [ ] Terraform installed
* [ ] FortiOS Terraform Provider installed
* [ ] REST API administrator created
* [ ] API token generated
* [ ] Trusted hosts configured

Architecture:

```text
Terraform
     |
     ▼
REST API
     |
     ▼
FortiGate
```

---

# 🔐 REST API Administrator Checklist

Validate:

* [ ] API token created
* [ ] Admin profile permissions correct
* [ ] Trusted hosts configured
* [ ] HTTPS access enabled

---

# 🚨 REST API Troubleshooting

## HTTP 401

Check:

* [ ] API token
* [ ] Authentication
* [ ] Trusted hosts

---

## HTTP 403

Check:

* [ ] Administrator profile
* [ ] API permissions

---

# 🧱 Terraform Certificate Checklist

Error:

```text
CA bundle should be set when insecure is false
```

Validate:

* [ ] CA certificate configured
* [ ] Provider certificate settings checked

Lab example:

```hcl
provider "fortios" {
 insecure = "true"
}
```

⚠️ Avoid disabling certificate verification in production.

---

# 🔍 Terraform Debug Checklist

Enable HTTPS daemon debug:

```bash
diagnose debug enable
diagnose debug application httpsd -1
```

Validate:

```text
Terraform
   ↓
REST API
   ↓
FortiGate
```

---

# 🚀 Performance Acceleration Checklist

## SR-IOV Validation

Understand:

```text
SR-IOV
 |
 +-- PF
 |
 +-- VF
```

---

# 🔌 Physical Function (PF)

Checklist:

* [ ] Dedicated NIC resource assigned
* [ ] PCI passthrough configured
* [ ] Hardware compatibility verified

Advantages:

* [ ] Maximum performance
* [ ] Direct hardware access

Limitations:

* [ ] Less VM sharing
* [ ] Higher resource cost

---

# 🔗 Virtual Function (VF)

Checklist:

* [ ] SR-IOV enabled
* [ ] VF assigned correctly
* [ ] NIC driver validated

Advantages:

* [ ] Multiple VM sharing
* [ ] Better resource utilization

---

# 🖥️ PCI Passthrough Checklist

Validate:

* [ ] NIC passthrough configured
* [ ] Hardware supported
* [ ] Migration limitations understood

Architecture:

```text
FortiGate-VM
      |
      ▼
PCI Passthrough
      |
      ▼
Physical NIC
```

---

# ⚡ DPDK / vSPU Checklist

Validate:

* [ ] Hypervisor support
* [ ] NIC support
* [ ] FortiGate build compatibility

Architecture:

```text
Virtual NIC
      |
      ▼
vSPU
      |
      ▼
Packet Processing
```

---

# 🔍 NIC Troubleshooting Checklist

Check:

```bash
diagnose hardware deviceinfo nic port2
```

Validate:

* [ ] Interface name
* [ ] Driver
* [ ] Driver version

---

# ☁️ Cloud Metadata Service Checklist

Purpose:

Protect metadata access against:

```text
SSRF
Server-Side Request Forgery
```

Architecture:

```text
FortiGate-VM
      |
      ▼
Metadata Service
      |
      ├── Token
      └── Session Enforcement
```

---

# ☁️ OCI Metadata Security Checklist

Validate:

* [ ] Legacy metadata disabled
* [ ] Token-based access enabled

Example:

```json
{
 "areLegacyImdsEndpointsDisabled": true
}
```

---

# 🌥️ Cloud-Init Troubleshooting Checklist

Check:

```bash
diagnose debug cloudinit show
```

Validate:

* [ ] Bootstrap script
* [ ] User-data
* [ ] Initialization process

---

# 🔌 OCI SDN Connector Checklist

Validate:

## Update EIP

```bash
execute update-eip
```

---

## Interface Information

```bash
diagnose test application ocid 4
```

---

## Region Information

```bash
diagnose test application ocid 5
```

---

## Token Information

```bash
diagnose test application ocid 6
```

---

# 🔐 FIPS Mode Checklist

Supported platforms:

* [ ] AWS
* [ ] Azure
* [ ] OCI
* [ ] GCP

---

## Enable FIPS Cipher Mode

```bash
config system fips-cc
set status fips-ciphers
end
```

---

# ⚠️ FIPS Validation

Affected services:

* [ ] SSH
* [ ] HTTPS
* [ ] SSL VPN
* [ ] IPsec

---

# 🚨 Disable FIPS Checklist

Important:

> Disabling FIPS requires factory reset.

Workflow:

```text
FIPS Enabled
      |
      ▼
Factory Reset
      |
      ▼
Reconfigure System
```

---

# 🧪 FortiGate-VM Final Health Checklist

## Licensing

* [ ] License status = Valid
* [ ] FortiGuard reachable
* [ ] No duplicate license
* [ ] Subscription active

---

## Compute

* [ ] vCPU correct
* [ ] vRAM correct
* [ ] VM limits verified

---

## Automation

* [ ] Terraform provider working
* [ ] API token valid
* [ ] HTTPS communication working

---

## Performance

* [ ] SR-IOV validated
* [ ] PF/VF architecture confirmed
* [ ] NIC driver checked

---

## Cloud

* [ ] Metadata security enabled
* [ ] Cloud-init verified
* [ ] SDN connector tested

---

## Security

* [ ] FIPS requirements reviewed
* [ ] Cipher compatibility checked
* [ ] FortiGuard validation completed

---

# 🧠 Golden Troubleshooting Flow

```text
FortiGate-VM Issue
        |
        ▼
License?
        |
        ├── Yes
        │     |
        │     ▼
        │  FortiGuard Connectivity
        |
        ▼
Resource?
        |
        ├── CPU
        ├── RAM
        └── VM Limits
        |
        ▼
API?
        |
        ├── 401 Token
        └── 403 Permission
        |
        ▼
Network?
        |
        ├── NIC
        ├── SR-IOV
        └── PCI
        |
        ▼
Cloud?
        |
        ├── Metadata
        ├── IAM
        └── Cloud-init
        |
        ▼
Security?
        |
        └── FIPS
```

---

# 🏆 FortiGate-VM Interview Cheat Sheet

| Topic           | Key Point                     |
| --------------- | ----------------------------- |
| License Valid   | Full operation                |
| Warning         | Validation unavailable        |
| Invalid         | Restricted functionality      |
| FortiFlex       | Entitlement-based licensing   |
| PF              | Dedicated NIC resource        |
| VF              | Shared NIC via SR-IOV         |
| PCI Passthrough | Direct hardware access        |
| Terraform       | Infrastructure as Code        |
| Metadata Token  | Cloud metadata protection     |
| FIPS            | Restricted cryptographic mode |

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

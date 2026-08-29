# FortiGate VM 

> **Focus:** FortiGate-VM, Licensing, FortiFlex, VDOM, Terraform, SR-IOV, Metadata Service, FIPS

---

## 1. FortiGate VM Resources

### GUI

Navigate to:

```text
System > FortiGuard
````

On FortiGate-VM, the VM resource information includes:

| Resource | Description                        |
| -------- | ---------------------------------- |
| `vCPU`   | Number of allocated virtual CPUs   |
| `vRAM`   | Amount of allocated virtual memory |

---

# 2. FortiGate-VM License Status

FortiGate-VM license status can be:

| Status    | Meaning                                                                                    |
| --------- | ------------------------------------------------------------------------------------------ |
| `Valid`   | VM can connect to FortiManager or FortiGuard and validate the license                      |
| `Warning` | VM cannot validate the license, but has been disconnected for less than 30 continuous days |
| `Invalid` | Warning condition has continued for 30+ days                                               |
| `Pending` | Temporary state while the VM attempts to validate the license                              |

---

## 2.1 Valid License

```text
Valid
```

When the license is valid:

* FortiGate-VM can validate the license.
* FortiGuard connectivity is working.
* All licensed features are available.

---

## 2.2 Warning License

```text
Warning
```

Possible causes:

* FortiGate-VM cannot reach FortiGuard.
* FortiGate-VM cannot validate the license.
* License connectivity has been unavailable for less than 30 continuous days.

> The VM continues operating while the warning period has not exceeded the applicable limit.

---

## 2.3 Invalid License

```text
Invalid
```

Possible causes:

* FortiGuard connectivity is unavailable.
* License validation has failed for 30+ continuous days.
* License is duplicated on another VM.
* Evaluation or term-based license has expired.

Impact:

```text
GUI access       -> Restricted
Firewall policy  -> Does not work
FortiGuard       -> Downloads unavailable
```

---

## 2.4 Pending License

```text
Pending
```

Temporary state while FortiGate-VM attempts license validation.

Common causes of `Warning` / `Invalid`:

* FortiGate-VM cannot reach FortiGuard.
* Evaluation or term-based license has expired.
* The same license has already been validated by another VM.

---

# 3. VM License Diagnostics

### Display VM license

```bash
diagnose debug vm-print-license
```

### Display complete VM information

```bash
diagnose hardware sysinfo vm full
```

---

## 3.1 License Status Values

### `valid`

| Value | Meaning |
| ----: | ------- |
|   `0` | Invalid |
|   `1` | Valid   |

---

## 3.2 License Status Codes

| Status | Meaning                          |
| -----: | -------------------------------- |
|    `0` | Startup                          |
|    `1` | Success                          |
|    `2` | Warning                          |
|    `3` | Error                            |
|    `4` | Invalid copy / duplicate license |
|    `5` | Evaluation expired               |
|    `6` | Grace period                     |

### FortiFlex grace period

FortiFlex provides a temporary grace period after retrieving the license from FortiCare.

---

## 3.3 License Response Codes

|  Code | Meaning                                                     |
| ----: | ----------------------------------------------------------- |
| `200` | Valid                                                       |
| `202` | Accepted                                                    |
| `400` | Expired                                                     |
| `401` | Duplicate                                                   |
| `500` | Warning                                                     |
| `502` | Invalid / cannot connect to FortiGuard distribution servers |
| `6xx` | Evaluation license expired                                  |
| Other | Error                                                       |

> `2xx` and `3xx` generally indicate successful communication.

---

# 4. License Troubleshooting

Enable update debugging:

```bash
diagnose debug application update -1
diagnose debug enable
execute update-now
```

Check system status:

```bash
get system status
```

Example:

```text
License Status: Valid
Evaluation License Expires: Fri Jul 11 19:35:10 2025
VM Resources: 1 CPU/1 allowed, 2007 MB RAM/2048 MB allowed
```

---

# 5. Verify VM Resource Allocation

```bash
get system status | grep resources
```

Example:

```text
VM Resources: 1 CPU/4 allowed, 2006 MB RAM
```

---

## 5.1 Hot-Add vCPU

Some FortiGate-VM models support hot-adding CPUs.

Example:

```bash
execute cpu add 16
```

> A reboot may be required after increasing CPU resources.

---

# 6. FortiFlex Licensing

FortiFlex allows qualified enterprise and MSSP customers to create multiple VM entitlements.

Resource consumption is based on predefined points calculated daily.

### FortiFlex with proxy

```bash
execute vm-license <token> https://<username>:<password>@<proxy-ip>:<proxy-port>
```

---

## 6.1 FortiFlex Error Codes

### FortiCare errors

| Code | Meaning               |
| ---: | --------------------- |
|  `1` | Runtime/server error  |
| `57` | Invalid license token |
| `58` | Token already used    |

### FortiGate-generated errors

| Code | Meaning                             |
| ---: | ----------------------------------- |
| `60` | Failed to request FortiCare license |

Possible message:

```text
Failed to download VM license
```

---

# 7. Manual License Upload

GUI:

```text
System > FortiGuard
```

Manual upload is useful when FortiGate-VM cannot directly communicate with FortiGuard.

---

## 7.1 License Registration Flow

```text
Purchase FortiGate-VM
        |
        v
Receive registration code
        |
        v
Fortinet Customer Service & Support
        |
        v
Register product
        |
        v
Serial number generated
        |
        v
Asset > Manage/View Products
        |
        v
Download .lic
        |
        v
Upload license to FortiGate-VM
        |
        v
Reboot
        |
        v
FortiGuard validation
        |
        v
VM becomes fully functional
```

---

# 8. Upload License Through SCP

Useful when GUI access is unavailable.

### Enable SCP

```bash
config system global
    set admin-scp enable
end
```

### Enable SSH on interface

```bash
config system interface
    edit port1
        append allowaccess ssh
    end
```

### Upload license

From an SCP client:

```bash
scp a.lic admin@192.168.254.252:vmlicense
```

---

# 9. FortiGate-VM License Types

FortiGate-VM licensing includes:

```text
Perpetual
Subscription
FortiFlex
```

---

## 9.1 SKU Families

| Type              | Description                      |
| ----------------- | -------------------------------- |
| Normal / V-Series | Perpetual VM base licensing      |
| S-Series          | Annual subscription              |
| FortiFlex         | Annual entitlement-based program |

VM SKUs are based on vCPU capacity:

```text
1
2
4
8
16
32
Unlimited
```

Examples:

```text
FGTVME...
FGVM...S
```

---

# 10. Licensing and Support

## V-Series / Normal

```text
VM base
   |
   +-- Perpetual
   |
   +-- Support contracted separately
```

---

## S-Series

Single annual SKU containing:

```text
VM Base
+
FortiCare Service Bundle
```

Available service bundles:

```text
FortiCare Only
UTM
Enterprise
ATP
```

---

## FortiFlex

Annual program:

```text
FortiFlex
   |
   +-- VM Base
   |
   +-- FortiCare Bundle
```

Available service bundles:

```text
FortiCare Only
UTM
Enterprise
ATP
```

---

# 11. vCPU Upgrade During Contract

| License           | vCPU Upgrade  |
| ----------------- | ------------- |
| V-Series / Normal | Not supported |
| S-Series          | Supported     |
| FortiFlex         | Supported     |

### S-Series

You can also upgrade the support service bundle.

> Contact a Fortinet sales representative.

### FortiFlex

Different VM entitlement configurations can be applied through the FortiFlex portal.

> API support is not currently available for this operation.

---

# 12. vCPU Downgrade During Contract

| License           | Downgrade     |
| ----------------- | ------------- |
| V-Series / Normal | Not supported |
| S-Series          | Not supported |
| FortiFlex         | Not supported |

---

# 13. VDOM Support on FortiGate-VM

## V-Series / Normal

Each base license provides a default VDOM capability.

V-Series can support additional VDOM licensing depending on the license.

> Refer to the FortiGate-VM datasheet for the exact limits.

---

## S-Series

By default:

```text
Additional VDOMs
        |
        v
Subscription VDOM license required
```

---

# 14. Adding VDOMs to FortiGate-VM V-Series

## Step 1 — Purchase and Register Base License

Purchase the FortiGate-VM base license.

You receive:

```text
License Certification
        |
        v
Registration Code
```

Register it through Fortinet Customer Service & Support.

After registration:

```text
Serial Number
      |
      v
Asset > Manage/View Products
      |
      v
Download License File
```

---

## Step 2 — Upload Base License

GUI:

```text
Dashboard > Status
        |
        v
Virtual Machine Widget
        |
        v
FortiGate VM License
        |
        v
Upload
```

Select the `.lic` file.

> FortiGate-VM reboots after applying the base license.

---

## Step 3 — Verify Base License

Check:

```text
Dashboard > Status
```

The VM license should show a valid/checkmark state.

CLI:

```bash
get system status
```

Example:

```text
Maximum number of virtual domains: 1
```

---

# 15. Add VDOM Upgrade License

Purchase and register the VDOM upgrade license.

Example:

```text
Base license
     +
VDOM upgrade license
     |
     v
Additional VDOM capacity
```

---

## Apply VDOM License Key

```bash
execute upd-vd-license <license-key>
```

Example:

```bash
execute upd-vd-license m6jsd-8ee32-vhijb-n
```

Expected result:

```text
Update VDOM license succeeded
```

---

## Verify

```bash
get system status
```

Example:

```text
Maximum number of virtual domains: 15
```

> When adding VDOMs for the first time on a V-Series instance, FortiOS does not count the default `root` VDOM toward the added VDOM count.

Therefore:

```text
Default root VDOM
+
15 additional VDOMs
```

is displayed as:

```text
Maximum number of virtual domains: 15
```

not:

```text
16
```

---

# 16. Terraform + FortiGate

Terraform can automate FortiGate configuration and infrastructure deployment.

Useful for:

* Infrastructure as Code
* Configuration automation
* Repeatable deployments
* Configuration tracking
* Revision management
* CI/CD
* Testing new FortiOS functionality
* Rapid lab deployment
* Reducing manual configuration errors

---

## 16.1 Minimum Environment

Example:

```text
FortiOS >= 6.0
Terraform
FortiOS Terraform Provider
REST API Administrator
API Token
```

Example versions from the reference environment:

```text
FortiOS: 6.0+
Terraform Provider: 1.0.0
Terraform: 0.11.14
```

> Version compatibility should always be checked against the current provider/FortiOS documentation.

---

# 17. REST API Administrator

Terraform communicates with FortiGate through the REST API.

Create:

```text
REST API Administrator
        |
        +-- API Token
        |
        +-- Trusted Hosts
        |
        +-- Appropriate Admin Profile
```

---

## 17.1 REST API 403

```text
HTTP 403
```

Usually indicates:

```text
Administrator profile
        |
        v
Insufficient permissions
```

---

## 17.2 REST API 401

```text
HTTP 401
```

Check:

* API token
* Trusted hosts
* Authentication configuration

---

## 17.3 Terraform Provider CA Error

Possible error:

```text
Error getting CA bundle,
CA bundle should be set when insecure is false
```

In a lab/test environment:

```hcl
provider "fortios" {
    insecure = "true"
}
```

> Avoid disabling certificate verification in production unless there is a deliberate security reason.

---

# 18. REST API Troubleshooting

Enable HTTPS daemon debugging:

```bash
diagnose debug enable
diagnose debug application httpsd -1
```

Use this when investigating:

```text
Terraform
   |
   v
REST API
   |
   v
FortiGate
```

---

# 19. PF / VF / SR-IOV

FortiGate-VM supports:

```text
PCI Passthrough
SR-IOV
Physical Function (PF)
Virtual Function (VF)
```

---

## 19.1 SR-IOV

```text
SR-IOV
 |
 +-- PF
 |
 +-- VF
```

### Physical Function

A physical NIC function is assigned directly to a VM.

```text
Physical NIC
      |
      v
     PF
      |
      v
FortiGate-VM
```

### Virtual Function

A physical NIC can expose multiple virtual functions.

```text
Physical NIC
      |
      +---- VF ---- VM1
      |
      +---- VF ---- VM2
      |
      +---- VF ---- VM3
```

---

# 20. PF vs VF

| Feature             | PF                     | VF                  |
| ------------------- | ---------------------- | ------------------- |
| Resource allocation | Dedicated NIC function | Shared physical NIC |
| VM sharing          | Low                    | High                |
| Performance         | Usually higher         | High                |
| Resource efficiency | Lower                  | Higher              |
| Cost                | Higher                 | Lower               |
| PCI passthrough     | Yes                    | SR-IOV              |

### PF

Advantages:

* Dedicated NIC resources
* High performance
* Direct PCI access

Disadvantages:

* Requires dedicated NIC resources
* More expensive
* Less flexible for VM sharing

### VF

Advantages:

* Multiple VMs can share one physical NIC
* Efficient resource utilization

---

# 21. PCI Passthrough

PCI passthrough bypasses many virtualization I/O layers.

```text
VM
 |
 +-- PCI Passthrough
       |
       v
Physical NIC
```

Potential benefits:

* Lower virtualization overhead
* Better packet performance
* Direct hardware access

Potential limitations:

* Live migration may be affected.
* Resource sharing may be affected.
* Hardware dependency increases.

---

# 22. DPDK

DPDK provides packet-processing libraries for Linux environments.

Conceptually:

```text
Traditional
NIC -> Kernel -> Virtualization -> VM

DPDK
NIC -> High-performance packet processing -> Application
```

> DPDK support and performance depend on the specific hypervisor, NIC, FortiGate-VM build, and deployment architecture.

---

# 23. VSPU

Virtual SPU support can improve packet processing in supported virtualization environments.

```text
Virtual NIC
     |
     v
vSPU
     |
     v
FortiGate packet processing
```

Some hypervisor combinations may support vSPU, while others are unsupported or unverified.

---

# 24. Verify NIC Driver

Inside FortiGate-VM:

```bash
diagnose hardware deviceinfo nic port2
```

Example:

```text
Name: port2
Driver: i40e
Version: 2.4.10
```

Useful fields:

```text
Name
Driver
Driver Version
```

---

# 25. UEFI

FortiOS 6.2.0 and later removed utilities/software for UEFI 1.x.

Use:

```text
UEFI 2.x
```

for required UEFI tools/software utilities.

---

# 26. Instance Metadata Service

Cloud VM environments provide instance metadata services.

FortiGate-VM can use metadata mechanisms with tokens/session enforcement to reduce risks such as:

```text
SSRF
Server-Side Request Forgery
```

Concept:

```text
FortiGate-VM
     |
     v
Metadata Service
     |
     +-- Token
     |
     +-- Session enforcement
     |
     v
Cloud Instance Metadata
```

---

# 27. OCI Metadata Security

Oracle Cloud Infrastructure can disable legacy IMDS endpoints.

Example:

```json
{
    "areLegacyImdsEndpointsDisabled": true
}
```

Example instance deployment:

```text
OCI Compute Instance
        |
        +-- Availability Domain
        +-- Compartment
        +-- Image
        +-- Subnet
        +-- Shape
        +-- Public IP
        +-- User Data
        +-- SSH Key
        +-- Instance Options
```

Example:

```bash
oci compute instance launch \
    --availability-domain <availability-domain> \
    --compartment-id <compartment-id> \
    --display-name <instance-name> \
    --image-id <image-id> \
    --subnet-id <subnet-id> \
    --shape <shape> \
    --assign-public-ip true \
    --user-data-file <userdata-file> \
    --ssh-authorized-keys-file <public-key> \
    --instance-options file://<metadata-json>
```

---

# 28. Cloud-Init Debugging

```bash
diagnose debug cloudinit show
```

Useful when troubleshooting:

* Bootstrap
* User data
* Cloud-init
* VM initialization

---

# 29. OCI SDN Connector + Metadata IAM

When using an SDN connector with metadata/IAM integration:

```bash
execute update-eip
```

### Interface information

```bash
diagnose test application ocid 4
```

### Region / ID

```bash
diagnose test application ocid 5
```

### Tokens

```bash
diagnose test application ocid 6
```

---

# 30. FIPS Cipher Mode

Supported cloud FortiGate-VM platforms include:

```text
AWS
Azure
OCI
GCP
```

Enable FIPS cipher mode:

```bash
config system fips-cc
    set status fips-ciphers
end
```

---

## 30.1 FIPS Scope

FIPS cipher mode affects supported cryptographic services such as:

```text
SSH
HTTPS
SSL VPN
IPsec
```

Example:

```text
FortiGate-VM
      |
      v
FIPS Cipher Mode
      |
      +-- SSH
      +-- HTTPS
      +-- SSL VPN
      +-- IPsec
```

> Configuration and supported algorithms must be reviewed carefully before enabling FIPS mode.

---

# 31. Disable FIPS

> Disabling FIPS mode requires a factory reset.

Conceptually:

```text
FIPS enabled
     |
     v
Need to disable
     |
     v
Factory reset
     |
     v
Reconfigure system
```

---

# 32. FIPS License / FortiGuard Validation

After enabling FIPS:

```text
FortiCare License
       |
       v
Validation
       |
       v
FortiGuard Databases
       |
       v
FortiGuard Engines
       |
       v
Updated
```

---

# 33. FIPS Unsupported Ciphers

The following cipher suites are not supported in the described FIPS configuration:

```text
DH-RSA-AES128-GCM-SHA256
DH-RSA-AES256-GCM-SHA384
```

---

# 34. Quick VM Troubleshooting

| Problem                  | First Checks                                   |
| ------------------------ | ---------------------------------------------- |
| License invalid          | `diagnose debug vm-print-license`              |
| License validation issue | `diagnose debug application update -1`         |
| VM resources             | `get system status`                            |
| CPU allocation           | `get system status \| grep resources`          |
| VM hardware information  | `diagnose hardware sysinfo vm full`            |
| REST API 401             | Token / trusted hosts                          |
| REST API 403             | Admin profile permissions                      |
| Terraform HTTPS issue    | CA bundle / certificate                        |
| Cloud-init issue         | `diagnose debug cloudinit show`                |
| OCI metadata issue       | `diagnose test application ocid 4/5/6`         |
| NIC issue                | `diagnose hardware deviceinfo nic <interface>` |
| FIPS configuration       | `show system fips-cc`                          |

---

# 35. VM Deployment Mental Model

```text
                       FortiGate-VM
                            |
          +-----------------+-----------------+
          |                 |                 |
       Licensing          Compute          Networking
          |                 |                 |
      FortiCare          vCPU/vRAM       vNIC / PF / VF
      FortiGuard         Hypervisor       SR-IOV
      FortiFlex           vSPU           PCI Passthrough
          |                 |                 |
          +-----------------+-----------------+
                            |
                       Cloud / Hypervisor
                            |
            +---------------+---------------+
            |               |               |
           AWS             Azure            OCI
            |                               |
            +-------------------------------+
                            |
                    Metadata / IAM
                            |
                       SDN Connector
```

---

# 36. Fast CLI Reference

## License

```bash
diagnose debug vm-print-license
diagnose hardware sysinfo vm full
get system status
get system status | grep resources
```

## License Update

```bash
diagnose debug application update -1
diagnose debug enable
execute update-now
```

## FortiFlex

```bash
execute vm-license <token> https://<username>:<password>@<proxy-ip>:<proxy-port>
```

## VDOM License

```bash
execute upd-vd-license <license-key>
```

## VM CPU

```bash
execute cpu add <number>
```

## Cloud-Init

```bash
diagnose debug cloudinit show
```

## NIC

```bash
diagnose hardware deviceinfo nic <interface>
```

## OCI Metadata / SDN

```bash
execute update-eip

diagnose test application ocid 4
diagnose test application ocid 5
diagnose test application ocid 6
```

## FIPS

```bash
config system fips-cc
    set status fips-ciphers
end
```

---

# 37. Troubleshooting Decision Tree

```text
FortiGate-VM Problem
        |
        +-- License?
        |      |
        |      +-- Invalid
        |      |     |
        |      |     +-- Check FortiGuard connectivity
        |      |     +-- Check license expiration
        |      |     +-- Check duplicate VM/license
        |      |
        |      +-- Warning
        |            |
        |            +-- Check FortiGuard connectivity
        |
        +-- Resource?
        |      |
        |      +-- vCPU
        |      +-- vRAM
        |      +-- VM limits
        |
        +-- REST API?
        |      |
        |      +-- 401 -> Token / Trusted Host
        |      +-- 403 -> Administrator permissions
        |
        +-- Networking?
        |      |
        |      +-- vNIC
        |      +-- PF
        |      +-- VF
        |      +-- SR-IOV
        |      +-- PCI passthrough
        |
        +-- Cloud?
        |      |
        |      +-- Metadata
        |      +-- IAM
        |      +-- Cloud-init
        |      +-- SDN connector
        |
        +-- FIPS?
               |
               +-- Cipher compatibility
               +-- FortiCare validation
               +-- FortiGuard updates
```

---

# 38. Key Takeaways

> ### VM Licensing
>
> `Valid` → Fully operational
> `Warning` → License validation unavailable
> `Invalid` → Restricted functionality
> `Pending` → License validation in progress

> ### Performance
>
> `PF` → Dedicated PCI/NIC resource
> `VF` → Shared NIC resource through SR-IOV
> `PCI Passthrough` → Direct hardware access
> `vSPU` → Virtual packet-processing acceleration

> ### Automation
>
> `Terraform + REST API` → Infrastructure as Code + repeatable FortiGate deployment

> ### Cloud
>
> `Metadata Service + Token Enforcement` → Helps protect cloud metadata access against SSRF-style abuse

> ### FIPS
>
> FIPS mode changes the available cryptographic configuration and requires careful planning before deployment.

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

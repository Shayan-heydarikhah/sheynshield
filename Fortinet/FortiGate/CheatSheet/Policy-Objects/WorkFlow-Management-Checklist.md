# 🔗 SheynShield Resources

# FortiGate Workflow Management Checklist

> **FortiOS | Workflow Management · Configuration Change Control · Workspace Mode · Policy Change Summary · Policy Expiration**
>
> **Level:** NSE 4 / NSE 7  
> **Focus:** Enterprise Firewall Governance, Change Management & Policy Lifecycle Control

---

# 📌 Checklist Index

- [ ] [1. Workflow Management Fundamentals](#1-workflow-management-fundamentals)
- [ ] [2. Enable Workflow Management](#2-enable-workflow-management)
- [ ] [3. Verify Workflow Settings](#3-verify-workflow-settings)
- [ ] [4. Configuration Save Modes](#4-configuration-save-modes)
- [ ] [5. Auto Mode Validation](#5-auto-mode-validation)
- [ ] [6. Workspace Mode Validation](#6-workspace-mode-validation)
- [ ] [7. Policy Change Summary](#7-policy-change-summary)
- [ ] [8. Policy Expiration Management](#8-policy-expiration-management)
- [ ] [9. Temporary Access Design](#9-temporary-access-design)
- [ ] [10. Enterprise Change Workflow](#10-enterprise-change-workflow)
- [ ] [11. Security Governance Checklist](#11-security-governance-checklist)
- [ ] [12. Troubleshooting Checklist](#12-troubleshooting-checklist)
- [ ] [13. NSE Exam Notes](#13-nse-exam-notes)
- [ ] [14. Quick Reference](#14-quick-reference)

---

# 1. Workflow Management Fundamentals

## ✅ Understand Workflow Management Purpose

- [ ] Understand that Workflow Management provides configuration change control.
- [ ] Understand that it improves:

  - [ ] Accountability
  - [ ] Audit visibility
  - [ ] Change tracking
  - [ ] Operational governance
  - [ ] Policy lifecycle management

---

## Normal Configuration Model

```text
Administrator
      │
      ▼
Modify Configuration
      │
      ▼
Save Change
````

---

## Workflow Management Model

```text
Administrator
      │
      ▼
Configuration Change
      │
      ▼
Change Description
      │
      ▼
Review Process
      │
      ▼
Commit / Apply
```

---

# 2. Enable Workflow Management

## GUI Path

* [ ] Navigate to:

```text
System
   ↓
Feature Visibility
   ↓
Workflow Management
```

* [ ] Enable:

```text
Workflow Management
```

---

## Validation Concept

```text
Feature Visibility

        ↓

Workflow Management

        ↓

Configuration Change Control
```

---

# 3. Verify Workflow Settings

## GUI Path

* [ ] Navigate to:

```text
System
   ↓
Settings
   ↓
Workflow Management
```

---

## Verify Available Options

* [ ] Config Save Mode
* [ ] Policy Change Summary
* [ ] Policy Expiration
* [ ] Default Expiration Period

---

# 4. Configuration Save Modes

## Supported Modes

| Mode      | Purpose                          |
| --------- | -------------------------------- |
| Auto      | Automatic configuration workflow |
| Workspace | Controlled change workflow       |

---

# 5. Auto Mode Validation

## Auto Mode Behavior

Checklist:

* [ ] Confirm configuration changes are automatically applied.
* [ ] Confirm no additional commit workflow is required.
* [ ] Confirm environment does not require formal approval.

---

## Auto Mode Flow

```text
Admin

 ↓

Modify Configuration

 ↓

Change Applied
```

---

## Recommended For

* [ ] Small environments
* [ ] Simple operational changes
* [ ] Fast troubleshooting scenarios

---

# 6. Workspace Mode Validation

## Workspace Mode Purpose

Checklist:

* [ ] Enable controlled configuration changes.
* [ ] Validate administrator workflow.
* [ ] Validate review process.
* [ ] Validate commit process.

---

## Workspace Flow

```text
Administrator

        ↓

Configuration Change

        ↓

Workspace

        ↓

Review

        ↓

Validate

        ↓

Commit
```

---

## Enterprise Use Cases

* [ ] Multi-admin environments
* [ ] Security operation teams
* [ ] Change-management processes
* [ ] Compliance environments
* [ ] Maintenance windows

---

# 7. Policy Change Summary

## Purpose

Verify that administrators can document:

```text
WHAT?
WHY?
WHO?
SCOPE?
WHEN?
```

---

# Policy Change Summary Modes

## Required Mode

Checklist:

* [ ] Administrator must provide change explanation.
* [ ] Change documentation is mandatory.
* [ ] Audit trail contains business context.

Flow:

```text
Policy Modification

        ↓

Change Summary Required

        ↓

Continue
```

---

## Optional Mode

Checklist:

* [ ] Administrator can provide description.
* [ ] Change can continue without summary.

Flow:

```text
Policy Modification

        ↓

Optional Description
```

---

# Recommended Change Description Template

```text
WHAT:
Allow HTTPS access from vendor network.

WHY:
Temporary troubleshooting requirement.

SCOPE:
Vendor subnet → Application server.

DURATION:
Maintenance window.

REFERENCE:
Change Ticket ID.
```

---

# 8. Policy Expiration Management

## Understand Policy Lifecycle

Checklist:

* [ ] Understand temporary policy risks.
* [ ] Use expiration for temporary access.
* [ ] Review expired policies regularly.

---

## Without Expiration

```text
Temporary Access

        ↓

Firewall Policy

        ↓

Forgotten Forever
```

---

## With Expiration

```text
Temporary Access

        ↓

Firewall Policy

        ↓

Expiration Date

        ↓

Review / Remove
```

---

# Common Policy Expiration Use Cases

## Vendor Access

Checklist:

* [ ] Restrict source.
* [ ] Restrict destination.
* [ ] Restrict service.
* [ ] Define expiration date.

Example:

```text
Vendor

 ↓

Specific Server

 ↓

Required Port

 ↓

30 Days Expiration
```

---

## Troubleshooting Access

Checklist:

* [ ] Create temporary rule.
* [ ] Define maintenance period.
* [ ] Remove after troubleshooting.

---

## Migration Rule

Checklist:

* [ ] Define migration period.
* [ ] Monitor usage.
* [ ] Remove obsolete policy.

---

# 9. Temporary Access Design

## Secure Temporary Policy Pattern

```text
Specific Source

+

Specific Destination

+

Specific Service

+

Specific Schedule

+

Expiration Date
```

---

## Avoid

```text
Source      = ALL

Destination = ALL

Service     = ALL

Expiration  = 30 Days
```

Reason:

> Expiration limits lifetime. It does not make an insecure policy secure.

---

# 10. Enterprise Change Workflow

## Recommended Workflow

```text
Change Request

        ↓

Administrator

        ↓

Modify Policy

        ↓

Document Change

        ↓

Workspace

        ↓

Review

        ↓

Approval

        ↓

Commit

        ↓

Policy Active

        ↓

Expiration Review
```

---

# 11. Security Governance Checklist

## Enterprise Recommendation

* [ ] Enable Workflow Management.
* [ ] Prefer Workspace Mode.
* [ ] Require Policy Change Summary.
* [ ] Enable Policy Expiration.
* [ ] Review temporary rules periodically.
* [ ] Maintain change references.

---

## Governance Model

```text
Identity

+

Change

+

Explanation

+

Review

+

Lifecycle

↓

Controlled Firewall Operations
```

---

# 12. Troubleshooting Checklist

## Feature Not Available

* [ ] Check Feature Visibility.
* [ ] Confirm Workflow Management is enabled.
* [ ] Verify FortiOS version support.

---

## Changes Not Working As Expected

* [ ] Check Config Save Mode.
* [ ] Confirm Auto vs Workspace behavior.
* [ ] Verify commit process.
* [ ] Check administrator permissions.

---

## Policy Summary Problem

* [ ] Check Policy Change Summary mode.
* [ ] Verify Required/Optional setting.
* [ ] Test policy modification workflow.

---

## Policy Expiration Problem

* [ ] Verify expiration feature.
* [ ] Check default expiration value.
* [ ] Check individual policy expiration date.
* [ ] Confirm policy lifecycle status.

---

# 13. NSE Exam Notes 🧠

## Workflow Enable Location

Remember:

```text
System

 ↓

Feature Visibility

 ↓

Workflow Management
```

---

## Workflow Configuration Location

Remember:

```text
System

 ↓

Settings

 ↓

Workflow Management
```

---

# Save Mode Memory

```text
Auto

↓

Automatic Configuration Changes
```

```text
Workspace

↓

Controlled Configuration Changes
```

---

# Policy Change Summary

```text
Required

↓

Administrator Must Explain Change
```

```text
Optional

↓

Administrator May Explain Change
```

---

# Policy Expiration

Remember:

```text
Policy

↓

Expiration Date

↓

Lifecycle Control
```

---

# 14. Quick Reference

| Feature               | Location                    | Purpose                  |
| --------------------- | --------------------------- | ------------------------ |
| Workflow Management   | System → Feature Visibility | Enable workflow feature  |
| Config Save Mode      | System → Settings           | Control save behavior    |
| Auto Mode             | Workflow Management         | Automatic changes        |
| Workspace Mode        | Workflow Management         | Controlled changes       |
| Policy Change Summary | Workflow Management         | Document changes         |
| Required Summary      | Workflow Management         | Mandatory explanation    |
| Optional Summary      | Workflow Management         | Optional explanation     |
| Policy Expiration     | Workflow Management         | Temporary policy control |

---

# 🎯 Final Mental Model

```text
             WORKFLOW MANAGEMENT

                    │

     ┌──────────────┼──────────────┐

     ▼              ▼              ▼

 Save Mode    Change Summary   Expiration

     │              │              │

 Auto /       Required /       Lifecycle

Workspace      Optional        Control

                    │

                    ▼

          Secure Firewall Governance
```

---

# 🔥 One-Minute Revision

```text
Workflow Management

=

Change Control

+

Documentation

+

Review

+

Policy Lifecycle
```

---

# 🚀 Production Best Practice

```text
Workspace Mode

        +

Required Change Summary

        +

Policy Expiration

        ↓

Enterprise Firewall Governance
```

---

> **SheynShield Engineering Note**
>
> A mature firewall deployment is not only about allowing or blocking traffic.
> It is about controlling **who changes security policy, why the change exists, how it is reviewed, and when it should disappear.**

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

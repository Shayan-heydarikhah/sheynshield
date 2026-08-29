# FortiGate Workflow Management  

> **FortiOS | Workflow Management, Configuration Change Control, Policy Change Summary & Policy Expiration**

---

## 📌 Table of Contents

* [1. What Is Workflow Management?](#1-what-is-workflow-management)
* [2. Enable Workflow Management](#2-enable-workflow-management)
* [3. Workflow Management Settings](#3-workflow-management-settings)
* [4. Configuration Save Mode](#4-configuration-save-mode)
* [5. Auto Mode](#5-auto-mode)
* [6. Workspace Mode](#6-workspace-mode)
* [7. Policy Change Summary](#7-policy-change-summary)
* [8. Policy Expiration](#8-policy-expiration)
* [9. Default Expiration](#9-default-expiration)
* [10. Operational Workflow](#10-operational-workflow)
* [11. Security & Governance](#11-security--governance)
* [12. Troubleshooting Checklist](#12-troubleshooting-checklist)
* [13. NSE High-Value Notes](#13-nse-high-value-notes)
* [14. Quick Reference](#14-quick-reference)

---

# 1. What Is Workflow Management?

**Workflow Management** adds an additional layer of **configuration change control** to FortiGate.

Instead of treating every configuration modification as:

```text
Admin
  │
  ▼
Change Configuration
  │
  ▼
Save
```

workflow management can introduce:

```text
Admin
  │
  ▼
Configuration Change
  │
  ▼
Change Summary / Explanation
  │
  ▼
Workflow / Review
  │
  ▼
Commit / Apply
```

This is especially useful in environments where configuration changes need:

* Accountability
* Change documentation
* Review
* Operational control
* Better auditing
* Controlled policy changes

---

# 2. Enable Workflow Management

Navigate to:

```text
System
  ↓
Feature Visibility
  ↓
Workflow Management
```

Enable:

```text
Workflow Management
```

Conceptually:

```text
Feature Visibility
       │
       ▼
Workflow Management
       │
       ▼
Configuration Change Control
```

---

# 3. Workflow Management Settings

After enabling the feature, navigate to:

```text
System
  ↓
Settings
  ↓
Workflow Management
```

The important configuration areas include:

```text
Workflow Management
├── Config Save Mode
│   ├── Auto
│   └── Workspace
│
├── Policy Change Summary
│   ├── Required
│   └── Optional
│
└── Policies Expire By Default
    └── Expire Date
```

---

# 4. Configuration Save Mode

FortiGate provides different approaches for handling configuration changes.

```text
Config Save Mode
       │
       ├── Auto
       │
       └── Workspace
```

The choice affects how administrators work with configuration changes.

---

# 5. Auto Mode

```text
Config Save Mode
    ↓
Auto
```

In **Auto** mode, configuration changes are saved automatically according to the normal FortiGate configuration workflow.

Conceptually:

```text
Admin
  │
  ▼
Modify Configuration
  │
  ▼
Change Applied / Saved
```

This is the simpler operational model.

### Best suited for

* Smaller environments
* Rapid operational changes
* Environments without formal configuration review
* Administrators who need immediate changes

---

# 6. Workspace Mode

```text
Config Save Mode
    ↓
Workspace
```

Workspace mode introduces a more controlled approach to configuration changes.

Conceptually:

```text
Administrator
      │
      ▼
Make Changes
      │
      ▼
Workspace
      │
      ▼
Review / Validate
      │
      ▼
Commit / Apply
```

This is useful when configuration changes require stronger operational control.

### Typical Use Cases

* Enterprise environments
* Multi-administrator environments
* Change-management processes
* Maintenance windows
* Security policy review
* Configuration auditing

---

# 7. Policy Change Summary

Workflow Management can require administrators to provide an explanation/summary when modifying policies.

Navigate to:

```text
System
  ↓
Settings
  ↓
Workflow Management
  ↓
Policy Change Summary
```

Available behavior:

```text
Required
Optional
```

---

## Required

```text
Policy Change
      │
      ▼
Change Summary
      │
      ▼
Required
      │
      ▼
Continue
```

The administrator must provide a summary/explanation for the policy modification.

This improves:

* Change accountability
* Auditability
* Operational visibility
* Change tracking

---

## Optional

```text
Policy Change
      │
      ▼
Change Summary
      │
      ▼
Optional
```

The administrator can provide a description but is not forced to do so.

---

# 8. Policy Expiration

Workflow Management can also introduce an expiration date for policies.

Conceptually:

```text
Firewall Policy
      │
      ▼
Expiration Date
      │
      ▼
Policy Lifecycle
```

This is useful for temporary access.

Examples:

```text
Temporary Vendor Access
Temporary Troubleshooting Rule
Temporary Migration Rule
Temporary Internet Access
Temporary Security Exception
```

Instead of creating:

```text
Temporary Policy
       │
       ▼
Forgotten Forever
```

you can use:

```text
Temporary Policy
       │
       ▼
Expiration Date
       │
       ▼
Automatic Lifecycle Control
```

---

# 9. Default Policy Expiration

The Workflow Management settings include:

```text
Policies Expire By Default
```

with an expiration period.

Example/default value from the configuration context:

```text
Expire Date
    ↓
30 days
```

So conceptually:

```text
New Policy
    │
    ▼
Default Expiration
    │
    ▼
30 Days
```

> ⚠️ **Important:** Treat the expiration period as a configuration/default behavior, not a universal FortiOS constant. Verify the actual value on the target FortiOS release.

---

# 10. Temporary Access Example

Imagine a vendor needs access for troubleshooting.

### Without Policy Expiration

```text
Vendor Access
     │
     ▼
Firewall Policy
     │
     ▼
Access remains enabled
     │
     ▼
Admin must remember to remove it
```

This creates a common security risk:

> **Temporary access becomes permanent access.**

---

### With Expiration

```text
Vendor Access
     │
     ▼
Firewall Policy
     │
     ▼
Expiration = 30 Days
     │
     ▼
Policy Lifecycle
     │
     ▼
Review / Expire
```

This provides a much better operational control.

---

# 11. Operational Workflow

A mature workflow can look like:

```text
                   CHANGE REQUEST
                         │
                         ▼
                  Administrator
                         │
                         ▼
                  Modify Policy
                         │
                         ▼
                Change Summary?
                         │
                  ┌──────┴──────┐
                  ▼             ▼
               Required       Optional
                  │             │
                  └──────┬──────┘
                         ▼
                    Workspace
                         │
                         ▼
                     Review
                         │
                         ▼
                     Validate
                         │
                         ▼
                      Commit
                         │
                         ▼
                   Policy Active
                         │
                         ▼
                 Expiration Timer
                         │
                         ▼
                  Review / Expire
```

---

# 12. Workflow Management vs Normal Configuration

| Capability            | Normal Configuration | Workflow Management |
| --------------------- | -------------------- | ------------------- |
| Configuration changes | ✅                    | ✅                   |
| Change explanation    | Limited              | ✅                   |
| Change accountability | Basic                | Stronger            |
| Workspace concept     | ❌                    | ✅                   |
| Policy lifecycle      | Manual               | Better controlled   |
| Temporary policies    | Manual removal       | Expiration support  |
| Change governance     | Basic                | Stronger            |

---

# 13. Security & Governance

Workflow Management is not just a UI feature.

It can support a broader **security governance model**:

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
Accountable Configuration
```

### Recommended enterprise approach

```text
Workflow Management
        │
        ├── Workspace Mode
        │
        ├── Required Change Summary
        │
        └── Policy Expiration
```

This creates a stronger operational discipline around firewall changes.

---

# 14. Policy Expiration — Security Use Cases

Use expiration dates for policies such as:

### 🔧 Troubleshooting

```text
Source → Engineer
Destination → Server
Service → Temporary troubleshooting ports
Expiration → Maintenance window
```

### 👨‍💻 Vendor Access

```text
Vendor
   ↓
Specific source
   ↓
Specific destination
   ↓
Required service
   ↓
Expiration
```

### 🚀 Migration

```text
Old Network
     ↓
Temporary Policy
     ↓
Migration Period
     ↓
Expire
```

### 🚨 Emergency Access

```text
Emergency Rule
      ↓
Required Duration
      ↓
Expiration
      ↓
Review
```

---

# 15. Least-Privilege Recommendation

Policy expiration should **not** replace proper policy design.

Prefer:

```text
Specific Source
      +
Specific Destination
      +
Specific Service
      +
Specific Schedule
      +
Expiration
```

Avoid:

```text
Source = all
Destination = all
Service = all
Expiration = 30 days
```

Expiration reduces the lifetime of a bad rule; it does **not** make an overly permissive rule secure.

---

# 16. Change Summary — What Should Be Documented?

When Policy Change Summary is required, good descriptions should answer:

```text
WHAT?
WHY?
WHO?
SCOPE?
WHEN?
```

Example:

```text
WHAT:
Allow HTTPS access from vendor subnet.

WHY:
Temporary troubleshooting of application issue.

SCOPE:
Vendor subnet → Application server.

DURATION:
Required during maintenance window.

REFERENCE:
INC-12345
```

This makes the change much more useful during an audit.

---

# 17. Troubleshooting Checklist

If Workflow Management does not behave as expected:

```text
[ ] Check Feature Visibility
[ ] Confirm Workflow Management is enabled
[ ] Check System → Settings → Workflow Management
[ ] Verify Config Save Mode
[ ] Check Auto vs Workspace behavior
[ ] Verify Policy Change Summary setting
[ ] Check whether summary is Required or Optional
[ ] Check policy expiration settings
[ ] Verify default expiration period
[ ] Check the specific policy expiration date
[ ] Review administrator permissions
[ ] Confirm the FortiOS version
```

---

# 18. NSE High-Value Notes 🧠

### Where do I enable it?

```text
System
  ↓
Feature Visibility
  ↓
Workflow Management
```

---

### Where do I configure it?

```text
System
  ↓
Settings
  ↓
Workflow Management
```

---

### Save Modes

```text
Auto
  ↓
Normal automatic configuration workflow

Workspace
  ↓
Controlled configuration workspace
```

---

### Policy Change Summary

```text
Required
  ↓
Administrator must provide summary

Optional
  ↓
Administrator may provide summary
```

---

### Policy Expiration

```text
Policy
  ↓
Expiration Date
  ↓
Lifecycle Control
```

---

### Default Expiration

```text
30 Days
```

> ⚠️ Verify the actual configured/default value on the target FortiOS release.

---

# 19. Golden Design Pattern 🔥

For enterprise firewall change management:

```text
                 ADMIN
                   │
                   ▼
              CHANGE POLICY
                   │
                   ▼
           CHANGE SUMMARY
                   │
                   ▼
              WORKSPACE
                   │
                   ▼
                REVIEW
                   │
                   ▼
               VALIDATE
                   │
                   ▼
                COMMIT
                   │
                   ▼
             POLICY ACTIVE
                   │
                   ▼
             EXPIRATION
                   │
                   ▼
            REVIEW / REMOVE
```

This pattern transforms firewall configuration from:

> **"Admin changes something and forgets about it"**

into:

> **"Every important change has context, lifecycle and accountability."**

---

# 20. Quick Reference

| Setting               | Location                                  | Purpose                            |
| --------------------- | ----------------------------------------- | ---------------------------------- |
| Workflow Management   | `System → Feature Visibility`             | Enable workflow functionality      |
| Config Save Mode      | `System → Settings → Workflow Management` | Select configuration workflow      |
| `Auto`                | Workflow Management                       | Automatic save behavior            |
| `Workspace`           | Workflow Management                       | Controlled workspace-based changes |
| Policy Change Summary | Workflow Management                       | Document policy modifications      |
| `Required`            | Policy Change Summary                     | Summary must be provided           |
| `Optional`            | Policy Change Summary                     | Summary can be provided            |
| Policy Expiration     | Workflow Management                       | Control policy lifecycle           |
| Default Expire Date   | Workflow Management                       | Default expiration period          |
| Example default       | `30 days`                                 | Verify on target FortiOS           |

---

# 🎯 Final Mental Model

```text
                 WORKFLOW MANAGEMENT
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
   Save Mode       Change Summary    Expiration
        │                │                │
   ┌────┴────┐       ┌───┴───┐           │
   ▼         ▼       ▼       ▼           ▼
 Auto     Workspace Required Optional   Lifecycle
                                           │
                                           ▼
                                     Temporary Rules
```

### 🔥 One-Line Memory Hook

```text
Workflow Management
        =
Change Control
+
Change Documentation
+
Controlled Configuration
+
Policy Lifecycle
```

> **Best Practice:** For enterprise deployments, combine **Workspace + Required Policy Change Summary + policy expiration for temporary access** to create a more disciplined firewall change-management process.

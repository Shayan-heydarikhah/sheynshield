# FortiOS CLI  — Special Characters, Grep, Paging & Policy Management

> **SheynShield | Engineering Secure Networks**
> FortiOS CLI quick reference for **NSE4 / NSE7**, troubleshooting, configuration filtering, and operational workflows.

---

## 1. Special Characters in FortiOS CLI

FortiOS CLI has specific rules for entering special characters and values containing spaces.

### `?` — CLI Help

To enter `?` as a character instead of triggering CLI help:

```text
Ctrl + V
```

or:

```text
Ctrl + Shift + -
```

then press:

```text
?
```

> **Tip:** `?` normally invokes CLI context-sensitive help, so escaping it is necessary when it must be entered as data.

---

### `Tab`

To enter a literal `Tab` character:

```text
Ctrl + V
```

then:

```text
Tab
```

---

### Space Inside a String

When a space is part of a string value, use quotation marks:

```text
"Security Administrator"
```

or:

```text
'Security Administrator'
```

Alternatively, escape the space with `\`:

```text
Security\ Administrator
```

### Example

```bash
set description "Security Administrator"
```

or:

```bash
set description Security\ Administrator
```

---

### Literal `"` or `'`

If a quotation mark is part of a string value, escape it.

| Character | Escape |
| --------- | ------ |
| `'`       | `\'`   |
| `"`       | `\"`   |

Example:

```bash
set description "FortiGate \"Production\""
```

---

### Literal Backslash `\`

Use another backslash:

```text
\\
```

Example:

```bash
set description "C:\\FortiGate\\Config"
```

### Quick Reference

| Character | How to enter         |
| --------- | -------------------- |
| `?`       | `Ctrl + V` → `?`     |
| `Tab`     | `Ctrl + V` → `Tab`   |
| Space     | Quote string or `\ ` |
| `'`       | `\'`                 |
| `"`       | `\"`                 |
| `\`       | `\\`                 |

---

# 2. `grep` — Filter FortiOS CLI Output

`grep` is one of the most useful commands for **large FortiOS configurations**.

Basic syntax:

```bash
<command> | grep <option> <pattern>
```

Example:

```bash
show system interface | grep port3
```

---

## `-i` — Ignore Case

Search without considering uppercase/lowercase differences.

```bash
show system interface | grep -i port3
```

Matches:

```text
port3
PORT3
Port3
PoRt3
```

### Use Case

Useful when you don't know the exact capitalization of the target string.

---

# 3. `grep -n` — Show Line Numbers

Displays the matching line together with its line number.

```bash
show system interface | grep -n port3
```

Example:

```text
15: edit "port3"
```

### Why It Matters

Useful for quickly locating a configuration section inside a large CLI output.

---

# 4. `grep -f` — Show Configuration Context

`-f` is extremely useful in FortiOS because it displays the **configuration context surrounding the matching entry**.

Example:

```bash
show system interface | grep -f port3
```

Output:

```text
edit "port3"
set vdom "root"
set ip 192.168.101.1 255.255.255.0
set allowaccess ping
set type physical
set alias "LAN"
set snmp-index 3
next
end
```

### Why `-f` Is Important

Instead of finding only the matching line:

```text
edit "port3"
```

you can retrieve the relevant configuration block.

> ⭐ **NSE Tip:** When troubleshooting FortiOS configuration, `grep -f` is often much more useful than a simple `grep`.

---

# 5. `grep -v` — Invert the Match

Select lines that **do not match** the specified pattern.

```bash
show system interface | grep -v port3
```

This excludes lines containing:

```text
port3
```

### Important Distinction

`grep -v` filters **matching lines**, not necessarily an entire FortiOS configuration object.

Therefore:

```bash
show system interface | grep -v port3
```

does **not** mean:

> "Show every interface except port3 as complete configuration blocks."

It means:

> "Remove output lines containing `port3`."

---

# 6. `grep -c` — Count Matches

Displays only the number of matching lines.

```bash
show system interface | grep -c port3
```

Example:

```text
3
```

### Useful For

* Counting occurrences
* Checking whether a configuration exists
* Quick validation during troubleshooting

Example:

```bash
show firewall policy | grep -c "set action deny"
```

---

# 7. Context Options

The context options are extremely useful when the matching line alone isn't enough.

---

## `-A` — After

Print matching lines plus the specified number of lines **after** the match.

```bash
show system interface | grep -A 2 port3
```

Example:

```text
edit "port3"
set vdom "root"
set ip 192.168.101.1 255.255.255.0
```

Syntax:

```bash
grep -A <number> <pattern>
```

---

## `-B` — Before

Print matching lines plus the specified number of lines **before** the match.

```bash
show system interface | grep -B 6 port3
```

Example:

```text
next
edit "port2"
set vdom "root"
set type physical
set snmp-index 2
next
edit "port3"
```

Syntax:

```bash
grep -B <number> <pattern>
```

---

## `-C` — Context

Print matching lines plus the specified number of lines **before and after** the match.

```bash
show system interface | grep -C 6 port3
```

Example:

```text
next
edit "port2"
set vdom "root"
set type physical
set snmp-index 2
next
edit "port3"
set vdom "root"
set ip 192.168.101.1 255.255.255.0
set allowaccess ping
set type physical
set alias "LAN"
set snmp-index 1
```

Syntax:

```bash
grep -C <number> <pattern>
```

---

# 8. `grep` Options — Exam Quick Reference

| Option | Function                   | Example           |
| ------ | -------------------------- | ----------------- |
| `-i`   | Ignore case                | `grep -i port3`   |
| `-n`   | Show line number           | `grep -n port3`   |
| `-f`   | Show configuration context | `grep -f port3`   |
| `-v`   | Show non-matching lines    | `grep -v port3`   |
| `-c`   | Count matches              | `grep -c port3`   |
| `-A N` | N lines after match        | `grep -A 2 port3` |
| `-B N` | N lines before match       | `grep -B 6 port3` |
| `-C N` | N lines before + after     | `grep -C 6 port3` |

### Memory Trick

```text
-A → After
-B → Before
-C → Context
-v → invert
-c → count
-n → number
-i → ignore case
-f → FortiOS configuration context
```

---

# 9. CLI Screen Paging

By default, FortiOS may pause CLI output when a command produces multiple pages.

Example:

```text
-- more --
```

This prevents long output from immediately scrolling beyond the terminal buffer.

---

## `-- more --` Controls

When the CLI displays:

```text
-- more --
```

you can use:

| Key         | Action                           |
| ----------- | -------------------------------- |
| `Enter`     | Show next line                   |
| `q`         | Stop output and return to prompt |
| Arrow keys  | Navigate output                  |
| `Home`      | Move toward beginning            |
| `End`       | Move toward end                  |
| `Page Up`   | Previous page                    |
| `Page Down` | Next page                        |
| Other key   | Show next page                   |

If there is no interaction for approximately **30 seconds**, the console can truncate the output and return to the command prompt.

---

# 10. Disable CLI Paging

Configure the system console:

```bash
config system console
    set output standard
end
```

This disables the `-- more --` paging behavior.

### Default vs Standard

```text
More mode
    ↓
Output pauses
    ↓
-- more --
```

```text
Standard mode
    ↓
Output continues without paging
```

> ⚠️ **Operational Note:** `standard` output can produce very large terminal output. `more` mode is often safer when working through a terminal with limited scrollback.

---

## Stopping Long Output

When paging is disabled, use:

```text
Ctrl + C
```

to interrupt the output.

---

# 11. Move and Clone Firewall Policies

FortiOS allows firewall policies to be **cloned** and **reordered** directly from the CLI.

Enter the firewall policy configuration:

```bash
config firewall policy
```

---

## Clone a Policy

Clone policy `1` to create policy `12`:

```bash
clone 1 to 12
```

Conceptually:

```text
Policy 1
   │
   └── clone ──► Policy 12
```

The new policy inherits the configuration of the source policy.

---

## Move a Policy

Move policy `12` before policy `1`:

```bash
move 12 before 1
```

Example:

```text
Before:

1   Allow-Web
2   Allow-DNS
12  New-Policy
```

After:

```text
12  New-Policy
1   Allow-Web
2   Allow-DNS
```

Exit the configuration context:

```bash
end
```

---

# 12. Clone + Move Workflow

A common operational workflow:

```bash
config firewall policy

clone 1 to 12

move 12 before 1

end
```

### Logic

```text
Existing Policy 1
       │
       │ clone
       ▼
New Policy 12
       │
       │ move
       ▼
Policy 12 placed before Policy 1
```

> ⚠️ **Critical:** Firewall policy order matters. FortiGate evaluates policies from top to bottom, so moving a policy can change traffic behavior.

---

# 13. High-Value Troubleshooting Patterns

### Find a specific interface

```bash
show system interface | grep -f port3
```

### Search case-insensitively

```bash
show system interface | grep -i port3
```

### Find configuration line number

```bash
show system interface | grep -n port3
```

### Count occurrences

```bash
show firewall policy | grep -c port3
```

### Show surrounding configuration

```bash
show system interface | grep -C 5 port3
```

### Show lines after a match

```bash
show system interface | grep -A 5 port3
```

### Show lines before a match

```bash
show system interface | grep -B 5 port3
```

---

# 14. NSE Exam Fast Recall

```text
grep
│
├── -i  → Ignore case
├── -n  → Line number
├── -f  → Configuration context
├── -v  → Invert match
├── -c  → Count
├── -A  → After
├── -B  → Before
└── -C  → Context
```

```text
CLI Paging
│
├── -- more -- → Paging enabled
├── Enter      → Next line
├── q          → Quit
├── arrows     → Navigate
└── Ctrl+C     → Interrupt output
```

```text
Policy Management
│
├── clone 1 to 12
└── move 12 before 1
```

---

# 15. ⚠️ Common Mistakes

### Mistake 1 — Confusing `grep -v` with object exclusion

```bash
grep -v port3
```

does **not** reliably remove the entire `port3` configuration object.

It removes matching **lines**.

---

### Mistake 2 — Using `grep` when `grep -f` is required

```bash
show system interface | grep port3
```

may return only:

```text
edit "port3"
```

while:

```bash
show system interface | grep -f port3
```

is intended to provide the relevant configuration context.

---

### Mistake 3 — Forgetting policy order

Creating a policy isn't enough.

Its **position in the policy table** determines when it is evaluated.

```text
Top
 ↓
Policy 1
Policy 2
Policy 3
 ↓
Bottom
```

A more specific policy placed below a broad allow/deny rule may never be reached.

---

# SheynShield One-Minute Revision

```text
SPECIAL CHARACTERS
?     → Ctrl+V then ?
Tab   → Ctrl+V then Tab
Space → "value with space" OR \ 
'     → \'
"     → \"
\     → \\

GREP
-i → ignore case
-n → line number
-f → FortiOS context
-v → inverse match
-c → count
-A → after
-B → before
-C → context

PAGING
-- more -- → output paging
Enter      → next line
q          → quit
Ctrl+C     → interrupt

CONSOLE
config system console
    set output standard
end

POLICY
config firewall policy
    clone 1 to 12
    move 12 before 1
end
```

---

## 🎯 Exam Focus

If you are preparing for **NSE4/NSE7**, remember these especially:

> **`grep -f` → configuration context**
> **`grep -v` → inverse matching**
> **`-A / -B / -C` → output context**
> **`set output standard` → disable CLI paging**
> **`clone X to Y` → clone policy**
> **`move X before Y` → change policy order**

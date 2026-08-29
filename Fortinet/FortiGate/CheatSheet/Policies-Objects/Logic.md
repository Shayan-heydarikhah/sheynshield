# FortiGate Policy & Objects  

> **FortiOS Focus:** NSE 4 / NSE 7
> **Scope:** Firewall Policies, Objects, NAT, VIP, IP Pools, Load Balancing, Traffic Shaping, Proxy Policy, DoS Policy, Session Helpers, Policy Lookup
> **Brand:** SheynShield — Engineering Secure Networks

---

# 1. Policy & Objects at a Glance

FortiGate firewall policies determine whether traffic is allowed, denied, inspected, translated, and logged.

A basic policy evaluates:

```text
Incoming Interface
        ↓
Source
        ↓
Destination
        ↓
Service
        ↓
Schedule
        ↓
Action
        ↓
Security Profiles
        ↓
NAT
        ↓
Logging
```

### Core Policy Fields

| Field              | Purpose                       |
| ------------------ | ----------------------------- |
| Incoming Interface | Where traffic enters          |
| Outgoing Interface | Where traffic leaves          |
| Source             | Source IP/network/user/device |
| Destination        | Destination IP/network/VIP    |
| Service            | Protocol/port                 |
| Schedule           | When the policy is active     |
| Action             | Accept / Deny                 |
| NAT                | Source NAT                    |
| Security Profiles  | AV, IPS, Web Filter, etc.     |
| Logging            | Session/event visibility      |

### Important Rule

```text
FortiGate policies are evaluated from top to bottom.
```

The first matching policy is normally selected.

---

# 2. Default Firewall Behavior

FortiGate does not automatically permit arbitrary traffic between interfaces.

Traffic must match an appropriate firewall policy.

Concept:

```text
Traffic
   ↓
Policy Lookup
   ↓
Matching Policy?
   ├── YES → Apply policy action
   └── NO  → Deny
```

### Policy Matching

A policy generally evaluates:

```text
1. Incoming Interface
2. Source
3. Outgoing Interface
4. Destination
5. Service
6. Schedule
```

Additional policy parameters can then control:

```text
NAT
Security Profiles
Traffic Shaping
Logging
Inspection Mode
```

---

# 3. Firewall Policy — Basic Structure

Example:

```bash
config firewall policy
    edit 10
        set name "LAN-to-Internet"
        set srcintf "LAN"
        set dstintf "wan1"
        set srcaddr "LAN-Network"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "HTTP" "HTTPS" "DNS"
        set nat enable
    next
end
```

### Command Notes

```bash
show firewall policy
# Displays configured firewall policies.

get firewall policy
# Displays firewall policy information.

config firewall policy
# Enters firewall policy configuration.

edit 10
# Selects or creates policy ID 10.

set srcintf "LAN"
# Defines the incoming interface.

set dstintf "wan1"
# Defines the outgoing interface.

set srcaddr "LAN-Network"
# Defines allowed source addresses.

set dstaddr "all"
# Allows traffic toward all destination addresses.

set action accept
# Allows traffic matching the policy.

set schedule "always"
# Makes the policy active continuously.

set service "HTTP" "HTTPS" "DNS"
# Allows the selected services.

set nat enable
# Enables source NAT for the policy.

next
# Saves the current policy entry.

end

```

---

# 4. Interface Pair

A firewall policy normally defines an interface pair:

```text
Incoming Interface
        ↓
      Policy
        ↓
Outgoing Interface
```

Example:

```text
LAN → WAN
LAN → DMZ
WAN → DMZ
DMZ → LAN
```

### Important

Interface selection is not simply documentation.

It is part of the policy matching logic.

---

# 5. Policy Objects

FortiGate uses objects to simplify policy configuration.

Common object types include:

```text
Address
Schedule
Service
Virtual IP
IP Pool
Traffic Shaper
Internet Service
```

Concept:

```text
Object
  ↓
Reusable definition
  ↓
Firewall Policy
```

---

# 6. Address Objects

Address objects represent:

```text
IP Address
Subnet
IP Range
FQDN
Geography
Dynamic/Identity-based addresses
```

Example:

```bash
config firewall address
    edit "LAN-Network"
        set subnet 192.168.10.0 255.255.255.0
    next
end
```

### Command Notes

```bash
config firewall address
# Enters address object configuration.

edit "LAN-Network"
# Creates or edits the address object.

set subnet 192.168.10.0 255.255.255.0
# Defines an IPv4 subnet address object.

next
# Saves the address object.

end
# Exits address configuration.
```

---

# 7. Address Object — FQDN

FQDN objects can represent destinations using DNS names.

Example:

```bash
config firewall address
    edit "example.com"
        set type fqdn
        set fqdn "example.com"
    next
end
```

### Command Notes

```bash
set type fqdn
# Changes the address object to FQDN type.

set fqdn "example.com"
# Defines the hostname used by the object.
```

### Important

FQDN-based objects depend on DNS resolution.

Use reliable DNS configuration when deploying FQDN objects.

---

# 8. Address Object Options

Useful address-object options can include:

```text
Static Route Configuration
Show in Address List
```

### Operational Tip

Enable address visibility options when the object should be easily reusable and visible during policy configuration.

---

# 9. Address Groups

Address groups combine multiple address objects.

Example:

```bash
config firewall addrgrp
    edit "Trusted-Networks"
        set member "LAN-Network" "DMZ-Network" "Branch-Network"
    next
end
```

### Command Notes

```bash
config firewall addrgrp
# Enters address-group configuration.

edit "Trusted-Networks"
# Creates or edits the address group.

set member "LAN-Network" "DMZ-Network" "Branch-Network"
# Adds address objects to the group.
```

### Benefit

Instead of:

```text
Policy
 ├── LAN
 ├── DMZ
 └── Branch
```

use:

```text
Policy
 └── Trusted-Networks
```

---

# 10. Schedule Objects

FortiGate supports schedule objects for controlling when policies operate.

Two major types:

```text
Recurring
One-Time
```

### Recurring

Used for:

```text
Business hours
Weekdays
Nightly access
Weekly maintenance
```

### One-Time

Used for:

```text
Temporary access
Maintenance window
Migration
Testing
```

---

# 11. Service Objects

Service objects define protocols and ports.

Examples:

```text
HTTP
HTTPS
DNS
SSH
RDP
ICMP
Custom TCP/UDP
```

Example:

```bash
config firewall service custom
    edit "Web-Access"
        set tcp-portrange 80 443
    next
end
```

### Command Notes

```bash
config firewall service custom
# Enters custom service configuration.

edit "Web-Access"
# Creates or edits the custom service.

set tcp-portrange 80 443
# Defines TCP destination ports 80 and 443.
```

---

# 12. Service Groups

Multiple services can be grouped.

Concept:

```text
Web-Access
 ├── HTTP
 ├── HTTPS
 └── DNS
```

This simplifies policy configuration and administration.

---

# 13. Virtual IP — VIP

VIP is commonly used for **Destination NAT (DNAT)**.

Concept:

```text
Internet
   ↓
Public IP
   ↓
VIP / DNAT
   ↓
Private Server
```

Example:

```text
External IP:
172.20.20.50

Mapped IP:
10.10.10.10

External Port:
80

Mapped Port:
80
```

---

# 14. VIP Configuration

Example:

```bash
config firewall vip
    edit "VIP-Web"
        set extip 172.20.20.50
        set mappedip "10.10.10.10"
        set extintf "wan1"
        set portforward enable
        set extport 80
        set mappedport 80
    next
end
```

### Command Notes

```bash
config firewall vip
# Enters VIP configuration.

edit "VIP-Web"
# Creates or edits the VIP object.

set extip 172.20.20.50
# Defines the external destination IP.

set mappedip "10.10.10.10"
# Defines the internal mapped server IP.

set extintf "wan1"
# Restricts the VIP to the specified external interface.

set portforward enable
# Enables destination port forwarding.

set extport 80
# Defines the external destination port.

set mappedport 80
# Defines the internal server port.
```

---

# 15. VIP Port Forwarding

Without port forwarding:

```text
External IP
      ↓
Internal IP
      ↓
Potentially all ports
```

With port forwarding:

```text
Public-IP:80
      ↓
Private-IP:80
```

This is generally preferred when exposing specific services.

---

# 16. VIP Firewall Policy

Creating a VIP does **not** automatically allow traffic.

A firewall policy is still required.

Example:

```bash
config firewall policy
    edit 20
        set name "WAN-to-Web"
        set srcintf "wan1"
        set dstintf "dmz"
        set srcaddr "all"
        set dstaddr "VIP-Web"
        set action accept
        set schedule "always"
        set service "HTTP" "HTTPS"
        set nat disable
    next
end
```

### Command Notes

```bash
set dstaddr "VIP-Web"
# Uses the VIP as the policy destination.

set nat disable
# Disables source NAT for inbound DNAT traffic.
```

### Important

For inbound DNAT:

```text
Client
 ↓
Public IP
 ↓
VIP / DNAT
 ↓
Firewall Policy
 ↓
Internal Server
```

---

# 17. VIP + Security Profiles

Typical inbound web-server policy:

```text
WAN
 ↓
VIP
 ↓
DMZ Server
```

Security profiles can include:

```text
IPS
Antivirus
Web Filter
Application Control
```

Example concept:

```text
Internet
   ↓
VIP
   ↓
IPS
   ↓
Antivirus
   ↓
Web Server
```

---

# 18. VIP + NAT Consideration

For inbound DNAT policies:

```text
Destination NAT
+
Source NAT
```

should not be enabled blindly.

A common design is:

```text
DNAT = Enabled through VIP
SNAT = Disabled
```

This preserves the original client source IP for the internal server where the topology allows it.

### Operational Benefit

The backend can see:

```text
Real Client IP
```

instead of:

```text
FortiGate IP
```

This is particularly useful for:

```text
Security logs
IPS
Web logs
Forensics
Access control
```

---

# 19. VIP — Lab Example

For a lab environment:

```text
External IP:
172.20.20.50

Mapped IP:
OWASP BWA Server

Port Forward:
80 → 80
```

Concept:

```text
Client
  |
172.20.20.50:80
  |
FortiGate VIP
  |
OWASP BWA:80
```

---

# 20. VIP Best Practice

For Internet-to-DMZ publishing:

```text
WAN
 ↓
VIP
 ↓
Specific Firewall Policy
 ↓
Security Profiles
 ↓
DMZ Server
```

Avoid:

```text
WAN
 ↓
ALL → ALL
 ↓
DMZ
```

---

# 21. Load Balancing

FortiGate can distribute traffic across multiple real servers.

Concept:

```text
                 Virtual Server
                      |
             +--------+--------+
             |        |        |
           Server1  Server2  Server3
```

Enable the required feature visibility if necessary:

```text
System
 └── Feature Visibility
      └── Load Balancing
```

---

# 22. Health Check

Health checks determine whether real servers are available.

Example:

```text
Type:
HTTP

Port:
80

URL:
http://10.10.10.10

Interval:
10 seconds

Timeout:
2 seconds

Retry:
3
```

Concept:

```text
FortiGate
   ↓
Health Check
   ↓
Real Server
   ↓
Healthy / Failed
```

---

# 23. Health Check — Port 0

When the health-check port is configured as:

```text
0
```

the FortiGate can use the port configured on the real server.

This is useful when one health-check object is applied to multiple real servers using different server ports.

---

# 24. Health Check — Matched Content

Matched content validates the actual application response.

Concept:

```text
Health Check
      ↓
HTTP Request
      ↓
Web Server
      ↓
Expected Content?
   ├── YES → Healthy
   └── NO  → Failed
```

This is stronger than checking only whether TCP port 80 is open.

---

# 25. Virtual Server

A virtual server provides the frontend address for load balancing.

Example:

```text
Virtual Server IP:
192.168.10.1

Virtual Server Port:
80

Type:
HTTP
```

Common service ports:

| Protocol | Port |
| -------- | ---: |
| HTTP     |   80 |
| HTTPS    |  443 |
| IMAPS    |  993 |
| POP3S    |  995 |
| SMTPS    |  465 |

---

# 26. Load-Balance Methods

FortiGate can provide multiple load-balancing algorithms.

Common methods:

```text
Source IP Hash
Round Robin
Weighted
First Alive
Least RTT
Least Session
HTTP Host
```

---

# 27. Source IP Hash

Concept:

```text
Client IP
   ↓
Hash
   ↓
Same Real Server
```

Useful when client persistence is important.

### Characteristic

The same source IP tends to be directed toward the same real server.

If the real server becomes unavailable, distribution can change.

---

# 28. Round Robin

Concept:

```text
Request 1 → Server 1
Request 2 → Server 2
Request 3 → Server 3
Request 4 → Server 1
```

All available real servers are treated equally.

Unavailable servers are skipped.

---

# 29. Weighted Load Balancing

Servers receive traffic according to assigned weights.

Example:

```text
Server 1 → Weight 3
Server 2 → Weight 1
```

Server 1 receives a larger proportion of traffic.

Use when servers have different capacities.

---

# 30. First Alive

Traffic is sent to the first responsive server in the configured server order.

Concept:

```text
Server 1
   ↓
Alive?
 ├── YES → Use Server 1
 └── NO
      ↓
   Server 2
```

Useful for active/standby-style designs.

---

# 31. Least RTT

Traffic is directed toward the real server with the lowest round-trip time.

Concept:

```text
Server 1 → 20 ms
Server 2 → 10 ms
Server 3 → 30 ms

Winner → Server 2
```

The RTT value is associated with the health-check mechanism.

---

# 32. Least Session

Traffic is directed toward the server with the lowest current session count.

Example:

```text
Server 1 → 100 sessions
Server 2 → 40 sessions
Server 3 → 70 sessions

Winner → Server 2
```

### Best Fit

Useful when real servers have:

```text
Similar capabilities
Similar performance
Dynamic connection loads
```

---

# 33. HTTP Host

The HTTP Host header can determine the real server selection.

Concept:

```text
HTTP Host:
app1.example.com
       ↓
Server Group A

HTTP Host:
app2.example.com
       ↓
Server Group B
```

Useful for HTTP virtual-host architectures.

---

# 34. Persistence

Persistence keeps a client associated with the same real server.

Available mechanisms can include:

```text
None
HTTP Cookie
SSL Session ID
```

Concept:

```text
Client
  ↓
Server 2
  ↓
Persistence
  ↓
Next Request
  ↓
Server 2
```

---

# 35. HTTP Cookie Persistence

HTTP cookie persistence can maintain backend affinity.

Concept:

```text
Client
  ↓
Load Balancer
  ↓
Server 1
  ↓
Cookie
  ↓
Future requests
  ↓
Server 1
```

This is useful for applications that maintain state locally on a backend server.

---

# 36. HTTP Multiplexing

HTTP multiplexing can reuse a TCP connection between FortiGate and real servers.

Concept:

```text
Clients
  |  |  |
  |  |  |
FortiGate
    |
Single TCP Connection
    |
Real Server
```

This can reduce backend connection overhead.

---

# 37. Preserve Client IP

Preserving the original client IP can be useful for backend logging.

Concept:

```text
HTTP Header
X-Forwarded-For:
Original-Client-IP
```

Without client-IP preservation:

```text
Backend sees:
FortiGate IP
```

With preservation:

```text
Backend can see:
Original Client IP
```

---

# 38. SSL Offloading

FortiGate can terminate SSL/TLS on the frontend.

Concept:

```text
Client
  |
 HTTPS
  |
FortiGate
  |
 HTTP
  |
Real Server
```

Certificate configuration is required.

### Design

```text
TLS Termination
       ↓
Security Inspection
       ↓
Backend Connection
```

Use certificates appropriate for the deployment and supported TLS configuration.

---

# 39. Real Servers

A real server defines the backend destination.

Example:

```text
IP:
10.10.10.10

Port:
80

Max Connections:
Blank

Weight:
1
```

### Max Connection

Blank generally means no explicit maximum connection limit.

---

# 40. Real Server Weight

Weight affects traffic distribution when using a weight-based load-balancing method.

Example:

```text
Server 1 → Weight 1
Server 2 → Weight 3
```

If the algorithm is not weight-based, changing weight does not necessarily change traffic distribution.

---

# 41. Real Server Mode

Real servers can support states such as:

```text
Active
Standby
Disabled
```

Concept:

```text
Active Server
      ↓
Traffic

Standby Server
      ↓
Waits for active failure
```

---

# 42. Load-Balance Policy

Example:

```text
Incoming Interface:
wan1

Destination:
Virtual Server

Outgoing Interface:
DMZ

Service:
HTTP

Schedule:
always

Action:
accept
```

Concept:

```text
Internet
   ↓
Virtual Server
   ↓
Load Balancer
   ↓
DMZ
```

---

# 43. Traffic Shaping

Traffic shaping controls bandwidth usage and traffic priority.

Concept:

```text
Traffic
   ↓
Classification
   ↓
Priority
   ↓
Bandwidth Allocation
```

Typical objectives:

```text
Protect critical applications
Control bandwidth
Prevent congestion
Prioritize VoIP
Limit users
```

---

# 44. Traffic Shaper Types

Common types include:

```text
Shared
Per-IP
```

### Shared

One bandwidth limit is shared by traffic using the shaper.

### Per-IP

The bandwidth limit is applied per source IP.

---

# 45. Apply Shaper — Per Policy

Concept:

```text
Policy 1 → 1 Mbps
Policy 2 → 1 Mbps
Policy 3 → 1 Mbps
```

Each policy receives the configured shaper limit.

---

# 46. Apply Shaper — All Policies

Concept:

```text
Shared Shaper = 1 Mbps

Policy 1
Policy 2
Policy 3
Policy 4
       ↓
Combined = 1 Mbps
```

The configured bandwidth is shared between policies using the same shaper.

---

# 47. Traffic Shaper Priority

Typical priority levels include:

```text
High
Medium
Low
```

High-priority traffic receives preferential treatment when congestion exists.

Example:

```text
VoIP → High
Business Apps → Medium
Bulk Download → Low
```

---

# 48. Guaranteed vs Maximum Bandwidth

### Guaranteed Bandwidth

Defines bandwidth that should be reserved for the traffic.

### Maximum Bandwidth

Defines the upper bandwidth limit.

Concept:

```text
Guaranteed
     ↓
Minimum intended allocation

Maximum
     ↓
Maximum allowed usage
```

---

# 49. DSCP

DSCP:

```text
Differentiated Services Code Point
```

DSCP marks packets for QoS treatment.

Concept:

```text
Application
   ↓
DSCP Marking
   ↓
Network Devices
   ↓
QoS Decision
```

Useful in end-to-end QoS architectures.

---

# 50. Interface Bandwidth Limits

Bandwidth can also be controlled at interface level.

Example:

```bash
config system interface
    edit "port1"
        set inbandwidth 1500
        set outbandwidth 1500
    next
end
```

### Command Notes

```bash
config system interface
# Enters interface configuration.

edit "port1"
# Selects the target interface.

set inbandwidth 1500
# Limits inbound bandwidth to the configured value.

set outbandwidth 1500
# Limits outbound bandwidth to the configured value.

next
# Saves the interface configuration.
```

### Important

A value of:

```text
0
```

generally means no configured bandwidth limit.

---

# 51. Traffic Shaping — Direction

Conceptually:

```text
Shared Shaper
→ Outbound / upload direction

Reverse Shaper
→ Inbound / download direction
```

Always verify the exact behavior and syntax for the FortiOS release in use.

---

# 52. Traffic Shaping Processing

Traffic shaping occurs during traffic processing.

If bandwidth limits are exceeded:

```text
Traffic
   ↓
Bandwidth Check
   ↓
Limit exceeded?
   ├── NO → Continue
   └── YES → Queue/Drop according to configuration
```

### Design Consideration

Applying bandwidth restrictions as early as possible can avoid unnecessary processing of traffic that will eventually be discarded.

---

# 53. Explicit Proxy

FortiGate supports proxy-based traffic handling.

Proxy operates at:

```text
Layer 7
```

Traditional VPN operation is generally associated with:

```text
Layer 3 / Layer 2
```

depending on the VPN technology.

---

# 54. Proxy Policy

Proxy policies are used for proxy-based access control.

Common types include:

```text
Explicit Web
Transparent Web
FTP
```

Concept:

```text
Client
  ↓
Proxy
  ↓
Policy
  ↓
Internet
```

---

# 55. Explicit Proxy Configuration

Typical components:

```text
Proxy Interface
Outgoing Interface
Source
Destination
Schedule
Action
Security Profiles
```

Example concept:

```text
Proxy Interface
      ↓
Users
      ↓
Explicit Proxy
      ↓
WAN
```

---

# 56. Proxy Disclaimer

Disclaimer options can be associated with:

```text
Disabled
Domain
Policy
User
```

A disclaimer can be used to present an authentication/use notification before access.

---

# 57. Proxy Security Profiles

Proxy policies can use security inspection profiles such as:

```text
Web Filter
Antivirus
Application Control
IPS
```

Exact available profiles depend on the proxy type and FortiOS version.

---

# 58. Proxy Policy + IPv4 Policy

In designs where proxy and firewall policies work together, verify both layers.

Concept:

```text
Client
   ↓
Proxy
   ↓
Firewall Policy
   ↓
Internet
```

Do not assume creating a proxy policy automatically replaces every required firewall policy.

---

# 59. Full Proxy Mode

Proxy-based inspection can operate by processing traffic as a full proxy.

Concept:

```text
Client
   ↓
FortiGate Proxy
   ↓
New Backend Connection
   ↓
Server
```

This allows FortiGate to inspect application-layer content more deeply.

---

# 60. Oversized File / Email Handling

Proxy-based processing can apply controls to large content such as:

```text
Large Files
Emails
Web Objects
```

This can be useful for content inspection and resource control.

---

# 61. Policy View

FortiGate policy pages can provide different policy presentation modes.

Common concepts include:

```text
Sequence View
Interface Pair View
```

### Sequence View

Useful for understanding:

```text
Top → Bottom
```

policy evaluation order.

### Interface Pair View

Useful for understanding:

```text
LAN → WAN
LAN → DMZ
WAN → DMZ
```

---

# 62. Policy Ordering

Policy order is critical.

Example:

```text
Policy 1:
LAN → WAN
Source = all
Destination = all
Action = accept

Policy 2:
LAN → WAN
Source = Restricted-Users
Destination = Blocked-Site
Action = deny
```

Policy 2 may never be reached because Policy 1 matches first.

### Golden Rule

```text
More specific policies
        ↓
Above
        ↓
More general policies
```

---

# 63. Insert Policy

Policies can be inserted above or below existing rules.

Concept:

```text
Policy 10
Policy 20
Policy 30
```

Insert above Policy 20:

```text
Policy 10
NEW POLICY
Policy 20
Policy 30
```

---

# 64. Clone / Copy Policy

Cloning an existing policy is useful when creating similar rules.

Typical workflow:

```text
Existing Policy
      ↓
Clone
      ↓
Modify source/destination/service
      ↓
Enable
      ↓
Test
```

### Important

A newly inserted/copied policy should be reviewed carefully before enabling it.

---

# 65. Policy Lookup

Policy Lookup helps determine which policy should process specific traffic.

Concept:

```text
Source
+
Destination
+
Interface
+
Service
       ↓
Policy Lookup
       ↓
Matching Policy
```

Useful for troubleshooting unexpected allow/deny behavior.

---

# 66. Routing + Policy Lookup

When troubleshooting connectivity:

```text
Routing
   +
Policy Lookup
   +
Session
   +
NAT
```

should be checked together.

A correct route does not automatically mean traffic is permitted.

---

# 67. FortiView

FortiView provides operational visibility into traffic.

Useful areas include:

```text
Live Traffic
Applications
Sources
Destinations
Policies
Sessions
Threats
```

Concept:

```text
Traffic
   ↓
FortiView
   ↓
Visibility
   ↓
Troubleshooting
```

---

# 68. Internet Service Database

Internet Service Database provides predefined service/application destinations.

Concept:

```text
Application
   ↓
FortiGuard Internet Service Database
   ↓
IP / Network Identification
   ↓
Policy
```

Useful when application IP addresses change frequently.

Examples:

```text
Cloud Services
CDNs
SaaS Applications
Online Platforms
```

### Important

Availability and functionality can depend on FortiGuard licensing and FortiOS version.

---

# 69. Internet Service in Policy

Instead of:

```text
Destination:
Hundreds of IP addresses
```

you can use:

```text
Destination:
Internet Service
```

This simplifies policies for dynamic cloud services.

---

# 70. Policy Learning / Learning Mode

Policy learning can help observe traffic and recommend policy changes.

Concept:

```text
Traffic
   ↓
Learning
   ↓
Observation
   ↓
Recommendations
```

It is useful for policy analysis and migration.

### Limitation

Learning behavior should not be confused with a normal security enforcement policy.

Security profiles may not be available in the same way as standard firewall policies.

---

# 71. DoS Policy

DoS policies provide protection against common denial-of-service patterns.

Concept:

```text
Incoming Traffic
       ↓
DoS Policy
       ↓
Threshold
       ↓
Allow / Detect / Block
```

Typical matching elements include:

```text
Incoming Interface
Source
Destination
Service
```

---

# 72. DoS Threshold

A DoS policy can use thresholds to detect excessive traffic.

Concept:

```text
Normal Traffic
      ↓
Below Threshold
      ↓
Allowed

Attack Traffic
      ↓
Threshold Exceeded
      ↓
Protection Action
```

Thresholds should be tuned according to the real traffic profile.

---

# 73. Internet Access Policy

Typical LAN-to-Internet policy:

```text
Incoming Interface:
LAN / Zone

Outgoing Interface:
SD-WAN

Source:
VLAN10
VLAN20

Destination:
all

Service:
Web-Access
Ping

Schedule:
always

NAT:
enable
```

Concept:

```text
LAN
 ↓
Firewall Policy
 ↓
SD-WAN
 ↓
Internet
```

---

# 74. LAN-to-DMZ Policy

Example:

```text
Incoming Interface:
LAN

Outgoing Interface:
DMZ

Source:
VLAN10
VLAN20

Destination:
DMZ-Servers

Service:
HTTP
Ping

Schedule:
always

NAT:
disable
```

Concept:

```text
LAN
 ↓
Policy
 ↓
DMZ Server
```

---

# 75. WAN-to-DMZ Policy

Example:

```text
Incoming Interface:
Internet / SD-WAN

Outgoing Interface:
DMZ

Source:
all

Destination:
VIP-Web

Service:
HTTP
HTTPS

NAT:
disable

Security:
IPS
Antivirus

Logging:
All Sessions
```

Concept:

```text
Internet
   ↓
VIP / DNAT
   ↓
Firewall Policy
   ↓
Security Inspection
   ↓
DMZ
```

---

# 76. Restrict Internet Access

Example:

```text
Incoming Interface:
LAN

Outgoing Interface:
Internet

Source:
all
Trusted-Users

Destination:
Site-X
Google-DNS

Service:
all

NAT:
enable

Logging:
All Sessions

Inspection:
Flow-Based
```

### Design Tip

Use specific source, destination, and service objects whenever possible.

Avoid unnecessary:

```text
ALL
```

objects in production policies.

---

# 77. Session Helper

Session helpers assist FortiGate with protocols that require inspection of packet payloads to understand session behavior.

Typical example:

```text
SIP
H.323
PPTP
```

Concept:

```text
Packet Header
     ↓
Session Helper
     ↓
Payload Analysis
     ↓
Related Session Handling
```

---

# 78. Why Session Helpers Exist

Some protocols establish control sessions using one port but create related data sessions using different ports.

Example concept:

```text
Control Channel
      ↓
TCP
      ↓
Dynamic Data Channel
      ↓
UDP / Other Port
```

FortiGate may need additional protocol awareness to correctly handle these sessions.

---

# 79. Session Helper Configuration

View session helpers:

```bash
show system session-helper
# Displays configured session-helper entries.
```

Example:

```bash
config system session-helper
    edit 1
        set name pptp
        set port 1723
        set protocol 6
    next
end
```

### Command Notes

```bash
config system session-helper
# Enters session-helper configuration.

edit 1
# Selects session-helper entry 1.

set name pptp
# Defines the protocol helper name.

set port 1723
# Defines the control port.

set protocol 6
# Defines TCP as the transport protocol.

next
# Saves the helper entry.

end
# Exits session-helper configuration.
```

---

# 80. H.323 Session Helper

Example:

```bash
config system session-helper
    edit 2
        set name h323
        set port 1720
        set protocol 6
    next
end
```

### Command Notes

```bash
set name h323
# Defines the H.323 helper.

set port 1720
# Defines the H.323 control port.

set protocol 6
# Defines TCP as the transport protocol.
```

---

# 81. PMap Session Helper

Example:

```bash
show system session-helper 11
# Displays session-helper entry 11.
```

Possible configuration:

```bash
config system session-helper
    edit 11
        set name pmap
        set port 111
        set protocol 6
    next
end
```

### Command Notes

```bash
set name pmap
# Defines the PMAP helper.

set port 111
# Defines the control port.

set protocol 6
# Defines TCP as the transport protocol.
```

---

# 82. Delete Session Helper

Example:

```bash
config system session-helper
    delete 19
end
```

### Command Notes

```bash
delete 19
# Removes session-helper entry 19.
```

### Important

Do not delete session helpers blindly.

First verify:

```text
Protocol
Application
FortiOS behavior
Current sessions
Production dependency
```

---

# 83. Policy Troubleshooting Flow

```text
Client
  ↓
Interface
  ↓
Routing
  ↓
Policy Lookup
  ↓
Source/Destination Objects
  ↓
Service
  ↓
NAT / VIP
  ↓
Security Profiles
  ↓
Session
  ↓
FortiView / Logs
```

---

# 84. Policy Troubleshooting Checklist

```text
✓ Incoming interface correct
✓ Outgoing interface correct
✓ Source object correct
✓ Destination object correct
✓ VIP correctly configured
✓ Service correct
✓ Schedule active
✓ Policy enabled
✓ Policy order correct
✓ Route exists
✓ NAT behavior correct
✓ Security profile not blocking
✓ Session exists
✓ Logs enabled
```

---

# 85. Policy Design Rules

### Internet Access

```text
LAN
 ↓
SD-WAN/WAN
 ↓
NAT
 ↓
Internet
```

### Internal Access

```text
LAN
 ↓
DMZ
 ↓
Specific Destination
 ↓
No NAT
```

### Published Server

```text
Internet
 ↓
VIP
 ↓
Specific Service
 ↓
Security Profiles
 ↓
DMZ
```

---

# 86. NAT Mental Model

### Source NAT

```text
Private Source IP
       ↓
FortiGate NAT
       ↓
Public Source IP
```

Typical:

```text
LAN → Internet
```

### Destination NAT

```text
Public Destination IP
       ↓
VIP / DNAT
       ↓
Private Server IP
```

Typical:

```text
Internet → DMZ
```

---

# 87. IP Pool

IP pools provide source NAT addresses.

Concept:

```text
Private Client
      ↓
IP Pool
      ↓
Public Source IP
      ↓
Internet
```

IP pools can be used when the FortiGate should use specific public addresses instead of the outgoing interface address.

---

# 88. IP Pool — ARP Reply

ARP Reply can be relevant when IP-pool addresses exist on a directly connected network.

Concept:

```text
Upstream Device
      ↓
ARP Request
      ↓
FortiGate
      ↓
IP Pool Address
```

Enable ARP reply when the network design requires FortiGate to answer ARP for pool addresses.

---

# 89. IP Pool — Overload

Overload allows multiple internal clients to share public addresses using different source ports.

Concept:

```text
10.0.0.10 ─┐
10.0.0.11 ─┼→ Public IP
10.0.0.12 ─┘
```

Typical use:

```text
Many-to-One / PAT
```

---

# 90. IP Pool — One-to-One

One-to-one mapping provides a fixed relationship between internal and external addresses.

Concept:

```text
Public IP 1 ↔ Internal IP 1
Public IP 2 ↔ Internal IP 2
```

Useful when each internal address requires a dedicated public translation.

---

# 91. IP Pool — Fixed Port Range

Fixed-port-range mode maps traffic using defined port ranges.

The internal address range requirements differ from one-to-one designs.

Use when deterministic port allocation is required.

---

# 92. IP Pool — Port Block Allocation

Port-block allocation can provide flexible source-port allocation.

Concept:

```text
Public IP
   ↓
Port Blocks
   ↓
Internal Clients
```

The configured port block size influences how ports are allocated.

---

# 93. Policy + NAT Decision

Ask:

```text
Is traffic leaving toward Internet?
        ↓
       YES
        ↓
      SNAT?
```

Typical answer:

```text
YES
```

For internal routing:

```text
LAN → DMZ
```

Typical answer:

```text
NAT disabled
```

For inbound publishing:

```text
WAN → DMZ
```

Typical design:

```text
VIP / DNAT
+
SNAT disabled
```

---

# 94. Service Selection

A service should represent the actual required protocol.

Examples:

```text
DNS → UDP/TCP 53
HTTP → TCP 80
HTTPS → TCP 443
SSH → TCP 22
RDP → TCP 3389
```

### Security Rule

Prefer:

```text
Specific Service
```

over:

```text
ALL
```

when the application requirement is known.

---

# 95. Schedule Selection

Common:

```text
always
```

Use custom schedules for:

```text
Business Hours
Maintenance
Temporary Access
Scheduled Internet Access
```

### Security Rule

Temporary requirements should not automatically become:

```text
always
```

---

# 96. Policy Logging

Logging is critical for:

```text
Troubleshooting
Auditing
Security Monitoring
Incident Response
Traffic Analysis
```

Common design:

```text
Log Allowed Traffic
Log Security Events
Log Violations
```

For exposed services, detailed logging is particularly useful.

---

# 97. Flow-Based vs Proxy-Based Inspection

### Flow-Based

Concept:

```text
Traffic
 ↓
Inspect Flow
 ↓
Forward
```

Typically lower processing overhead.

### Proxy-Based

Concept:

```text
Client
 ↓
FortiGate Proxy
 ↓
Inspect
 ↓
New Connection
 ↓
Server
```

Provides deeper application-layer processing for supported features.

---

# 98. Policy Object Hierarchy

A useful mental model:

```text
                 POLICY
                   │
       ┌───────────┼───────────┐
       │           │           │
    INTERFACE    OBJECTS     SERVICE
       │           │           │
     IN/OUT    ┌───┼───┐    PORT/PROTO
               │   │   │
            Address VIP Schedule
               │
             NAT
               │
         Security Profiles
               │
             Logging
```

---

# 99. Policy & Objects — Fast Reference

```text
Address
→ Defines IP/network/FQDN identity

Address Group
→ Combines address objects

Schedule
→ Defines when policy is active

Service
→ Defines protocol/port

Service Group
→ Combines services

VIP
→ Destination NAT / DNAT

IP Pool
→ Source NAT address allocation

Firewall Policy
→ Controls traffic

Traffic Shaper
→ Controls bandwidth/QoS

Proxy Policy
→ Layer-7 proxy access control

DoS Policy
→ Threshold-based DoS protection

Session Helper
→ Protocol-aware session handling
```

---

# 100. Most Important Commands

```bash
show firewall policy
# Displays firewall policies.

get firewall policy
# Displays firewall policy information.

show firewall address
# Displays address objects.

show firewall addrgrp
# Displays address groups.

show firewall service custom
# Displays custom services.

show firewall vip
# Displays VIP objects.

show firewall ippool
# Displays IP pool configuration.

show firewall shaper traffic-shaper
# Displays traffic shaper configuration.

show firewall proxy-policy
# Displays proxy policies.

show firewall DoS-policy
# Displays DoS policies.

show system session-helper
# Displays session-helper configuration.

get router info routing-table all
# Displays the routing table.

diagnose sys session list
# Displays active sessions.

execute log display
# Displays available log information.
```

---

# 101. Policy Verification Commands

```bash
diagnose debug enable
# Enables FortiGate debug output.

diagnose debug flow filter addr 10.10.10.10
# Filters flow debugging for a specific IP address.

diagnose debug flow show console enable
# Displays flow-debug output in the CLI.

diagnose debug flow trace start 100
# Captures up to 100 flow-debug traces.

diagnose debug flow trace stop
# Stops flow tracing.

diagnose debug disable
# Disables debug output.
```

> **Production Warning:** Use flow debugging carefully because excessive debug output can affect system resources.

---

# 102. Policy Debug Mental Model

When traffic fails:

```text
Packet
  ↓
Ingress Interface?
  ↓
Route?
  ↓
Policy Match?
  ↓
Destination?
  ↓
Service?
  ↓
NAT?
  ↓
Security Profile?
  ↓
Session?
  ↓
Egress?
```

This prevents random troubleshooting.

---

# 103. Common Policy Mistakes

```text
❌ Wrong incoming interface
❌ Wrong outgoing interface
❌ Wrong source object
❌ Wrong destination object
❌ VIP missing from destination
❌ Wrong service
❌ Schedule inactive
❌ Policy below a broader policy
❌ NAT enabled unnecessarily
❌ NAT disabled for Internet access
❌ Missing route
❌ Security profile blocking traffic
❌ Session helper incorrectly configured
❌ Load-balancer health check failing
```

---

# 104. Policy Ordering — Golden Rule

Bad:

```text
1. LAN → WAN → ALL → ACCEPT
2. VLAN10 → WAN → Block-Site → DENY
```

Better:

```text
1. VLAN10 → WAN → Block-Site → DENY
2. LAN → WAN → ALL → ACCEPT
```

### Rule

```text
Specific
   ↓
General
```

---

# 105. Production Policy Design

Prefer:

```text
Specific Interface
+
Specific Source
+
Specific Destination
+
Specific Service
+
Required Schedule
+
Required Security Profiles
+
Correct NAT
+
Logging
```

Avoid:

```text
ALL
ALL
ALL
ALL
ACCEPT
```

unless there is a deliberate and documented reason.

---

# 106. Policy Security Checklist

```text
[ ] Correct incoming interface
[ ] Correct outgoing interface
[ ] Source restricted
[ ] Destination restricted
[ ] Service restricted
[ ] Schedule reviewed
[ ] NAT reviewed
[ ] Security profiles enabled where required
[ ] Logging enabled
[ ] Policy order reviewed
[ ] Unused objects removed
[ ] Temporary rules have expiration dates
[ ] Policy tested
[ ] Logs verified
```

---

# 107. VIP Security Checklist

```text
[ ] Public IP verified
[ ] External interface verified
[ ] Mapped IP verified
[ ] Port forwarding enabled when required
[ ] Only required ports exposed
[ ] Dedicated firewall policy created
[ ] Source restriction considered
[ ] IPS enabled where appropriate
[ ] Antivirus enabled where appropriate
[ ] Logging enabled
[ ] Backend sees original source IP where required
[ ] Health check validated
```

---

# 108. Load Balancer Checklist

```text
[ ] Virtual server configured
[ ] Virtual server IP verified
[ ] Virtual server port verified
[ ] Real servers configured
[ ] Real server ports verified
[ ] Health check configured
[ ] Health check URL verified
[ ] Interval verified
[ ] Timeout verified
[ ] Retry verified
[ ] Load-balance method selected
[ ] Persistence requirement evaluated
[ ] Client IP preservation evaluated
[ ] SSL offloading evaluated
[ ] Backend connectivity tested
```

---

# 109. Traffic Shaping Checklist

```text
[ ] Application identified
[ ] Bandwidth requirement defined
[ ] Shared vs Per-IP selected
[ ] Guaranteed bandwidth defined
[ ] Maximum bandwidth defined
[ ] Priority defined
[ ] DSCP requirement evaluated
[ ] Upload/download direction verified
[ ] Interface limits reviewed
[ ] Congestion behavior tested
```

---

# 110. NSE Memory Map

## Firewall Policy

```text
Incoming
   ↓
Source
   ↓
Destination
   ↓
Service
   ↓
Schedule
   ↓
Action
   ↓
NAT
   ↓
Security
   ↓
Log
```

## Objects

```text
Address
Schedule
Service
VIP
IP Pool
Shaper
Internet Service
```

## DNAT

```text
Public IP
   ↓
VIP
   ↓
Mapped IP
   ↓
Firewall Policy
```

## SNAT

```text
Private IP
   ↓
IP Pool / Interface IP
   ↓
Public IP
```

## Load Balancing

```text
Virtual Server
      ↓
Health Check
      ↓
Real Servers
      ↓
Algorithm
      ↓
Persistence
```

## Troubleshooting

```text
Interface
   ↓
Route
   ↓
Policy
   ↓
NAT
   ↓
Security
   ↓
Session
   ↓
Logs
```

---

# 111. Golden Rules

> **1. A firewall policy controls traffic; an object defines reusable policy components.**

> **2. Policy order matters.**

> **3. More specific policies should normally be placed above broader policies.**

> **4. VIP provides destination NAT; it does not replace the firewall policy.**

> **5. Source NAT and destination NAT solve different problems.**

> **6. LAN-to-Internet traffic commonly requires SNAT.**

> **7. LAN-to-DMZ traffic normally does not require NAT.**

> **8. Inbound VIP policies commonly use DNAT with source NAT disabled.**

> **9. Use specific services instead of `ALL` whenever possible.**

> **10. Schedule controls when a policy is active.**

> **11. Health checks determine backend availability in load balancing.**

> **12. Least Session is useful when real servers have similar capabilities but different current session loads.**

> **13. Persistence provides backend affinity.**

> **14. Preserve client IP when backend logging or application logic requires the original source IP.**

> **15. Traffic shaping controls bandwidth and priority; it does not replace routing or firewall policy.**

> **16. Policy Lookup and routing should be checked together during troubleshooting.**

> **17. Session helpers provide protocol-aware session handling for supported protocols.**

> **18. DoS policies use thresholds to detect abnormal traffic rates.**

> **19. FQDN objects depend on DNS resolution.**

> **20. Internet Service Database is useful for applications whose IP addresses change frequently.**

> **21. Always verify command syntax against the target FortiOS release.**

---

# 112. Ultra-Fast Policy & Objects

```text
                 POLICY & OBJECTS
                        │
        ┌───────────────┼────────────────┐
        │               │                │
      POLICY          OBJECTS           NAT
        │               │                │
   ┌────┼────┐     ┌────┼────┐       ┌───┴───┐
   │    │    │     │    │    │       │       │
  IN   SRC  DST  Address VIP Service  SNAT   DNAT
   │    │    │     │    │    │       │       │
   └────┼────┘     │ Schedule │      IP Pool VIP
        │          └──────────┘
     Service
        │
    Schedule
        │
      Action
        │
    Security
        │
       NAT
        │
      Logging
```

---

# 113. Final Mental Model

```text
                    FORTIGATE TRAFFIC
                           │
                           ▼
                    Incoming Interface
                           │
                           ▼
                         Route
                           │
                           ▼
                    Policy Lookup
                           │
              ┌────────────┴────────────┐
              │                         │
           MATCH                      NO MATCH
              │                         │
              ▼                         ▼
          Policy Rule                   DENY
              │
      ┌───────┼────────┐
      │       │        │
    Source Destination Service
      │       │        │
      └───────┼────────┘
              │
           Schedule
              │
            Action
              │
      ┌───────┴────────┐
      │                │
     NAT          Security Profiles
      │                │
      └───────┬────────┘
              │
           Session
              │
              ▼
           Logging
              │
              ▼
            Output
```

---

## SheynShield — Policy & Objects Core Memory

```text
POLICY
→ Controls traffic

ADDRESS
→ Defines who/what

SERVICE
→ Defines protocol/port

SCHEDULE
→ Defines when

VIP
→ DNAT / Publish internal service

IP POOL
→ SNAT / Public source addresses

TRAFFIC SHAPER
→ Bandwidth / QoS

LOAD BALANCER
→ Distributes traffic between real servers

PROXY POLICY
→ Layer-7 proxy control

DoS POLICY
→ Threshold-based protection

SESSION HELPER
→ Protocol-aware session handling

POLICY LOOKUP
→ Find the policy that should process traffic

FORTIVIEW
→ See what is actually happening
```

### One-Line NSE Mental Model

```text
Policy = Who + Where + What + When + Action + NAT + Security + Logging
```

**SheynShield — Engineering Secure Networks**

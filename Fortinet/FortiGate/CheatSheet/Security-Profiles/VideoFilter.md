# FortiGate Video Filter

> **FortiGate Video Filter | YouTube Channel Filtering | YouTube API | FortiGuard Video Rating | WAD | QUIC/HTTP3**
>
> Practical reference for **FortiOS 7.x**, NSE-style troubleshooting, configuration, and real-world deployments.

---

## 📚 Table of Contents

* [1. Video Filter Overview](#1-video-filter-overview)
* [2. Prerequisites](#2-prerequisites)
* [3. Video Filter Architecture](#3-video-filter-architecture)
* [4. Video Rating Decision Flow](#4-video-rating-decision-flow)
* [5. Local Video Filter Cache](#5-local-video-filter-cache)
* [6. FortiGuard Video Rating](#6-fortiguard-video-rating)
* [7. YouTube API Fallback](#7-youtube-api-fallback)
* [8. IPS Application Signature Fallback](#8-ips-application-signature-fallback)
* [9. FortiGuard Anycast](#9-fortiguard-anycast)
* [10. YouTube Channel Filtering](#10-youtube-channel-filtering)
* [11. YouTube API Key](#11-youtube-api-key)
* [12. Recommended Firewall Policy](#12-recommended-firewall-policy)
* [13. QUIC / HTTP3 Considerations](#13-quic--http3-considerations)
* [14. FortiGuard License & Updates](#14-fortiguard-license--updates)
* [15. WAD Diagnostics](#15-wad-diagnostics)
* [16. Video Filter Diagnostic Commands](#16-video-filter-diagnostic-commands)
* [17. WAD Video Debugging](#17-wad-video-debugging)
* [18. Troubleshooting Flow](#18-troubleshooting-flow)
* [19. Deployment Checklist](#19-deployment-checklist)
* [20. NSE Exam Traps](#20-nse-exam-traps)
* [21. Quick Revision Card](#21-quick-revision-card)

---

# 1. Video Filter Overview

FortiGate **Video Filter** allows administrators to control video content based on information such as:

* Video ID
* YouTube channel ID
* Video category
* FortiGuard video rating
* YouTube API information

### Main Goal

```text
Client
  |
  | HTTPS / HTTP3
  v
FortiGate
  |
  +--> WAD
  |
  +--> Video Filter
  |
  +--> Video ID / Channel ID
  |
  v
Rating / Classification
  |
  +--> Monitor
  +--> Allow
  +--> Block
```

---

# 2. Prerequisites

Video filtering is designed to work with **proxy-based inspection**.

### Firewall Policy

The policy should include:

```text
Proxy Inspection
      +
SSL Deep Inspection
      +
Video Filter
      +
Application Control
```

### Recommended Security Profile Stack

```text
Firewall Policy
│
├── Proxy-based inspection
│
├── SSL/SSH Inspection
│   └── Deep Inspection
│
├── Web Filter
│
├── Video Filter
│
└── Application Control
```

> **Important:** The exact feature availability and CLI syntax can vary by FortiOS release. Always verify the commands against the target FortiOS version.

---

# 3. Video Filter Architecture

A simplified architecture:

```text
                         CLIENT
                            |
                            | Web Request
                            v
                    +---------------+
                    |   FortiGate   |
                    +---------------+
                            |
                            v
                           WAD
                            |
                            v
                    +---------------+
                    | Video Filter  |
                    +---------------+
                            |
            +---------------+---------------+
            |               |               |
            v               v               v
       Local Cache     FortiGuard       YouTube API
                            |               |
                            |               |
                            +-------+-------+
                                    |
                                    v
                             IPS Signature DB
                                    |
                                    v
                              Final Decision
                                    |
                         +----------+----------+
                         |          |          |
                       Allow      Monitor     Block
```

---

# 4. Video Rating Decision Flow

When WAD receives a video request, FortiGate attempts to identify the video and determine its classification.

### Decision Sequence

```text
1. WAD receives video request
            |
            v
2. Extract Video ID
            |
            v
3. Check Local Video Filter Cache
            |
       +----+----+
       |         |
      HIT       MISS
       |         |
       v         v
    Result   FortiGuard
               Query
                 |
            +----+----+
            |         |
           HIT       FAIL
            |         |
            v         v
         Result    YouTube API
                       |
                  +----+----+
                  |         |
                 HIT       FAIL
                  |         |
                  v         v
               Result   IPS Application
                        Signature DB
                             |
                        +----+----+
                        |         |
                       HIT       FAIL
                        |         |
                        v         v
                     Result   No Match
```

### ⭐ Key Concept

The general fallback logic is:

```text
LOCAL CACHE
     ↓
FORTIGUARD
     ↓
YOUTUBE API
     ↓
IPS APPLICATION SIGNATURE
```

> The fallback sequence and exact implementation can depend on FortiOS version and enabled features.

---

# 5. Local Video Filter Cache

WAD first checks its local video-filter cache.

```text
Video Request
      |
      v
Extract Video ID
      |
      v
Local Cache
      |
 +----+----+
 |         |
HIT       MISS
 |         |
 v         v
Decision  FortiGuard
```

### Why Cache Matters

Local caching can:

* Reduce external queries
* Improve response time
* Reduce dependency on rating services
* Improve scalability

### Diagnostic Commands

```bash
diagnose test app wad 321
```

Display video filter cache statistics.

```bash
diagnose test app wad 323
```

Flush video filter cache entries.

---

# 6. FortiGuard Video Rating

If the video is not found in the local cache, WAD can query the **FortiGuard video rating service**.

```text
Video ID
   |
   v
FortiGuard Video Rating
   |
   +---- Category
   |
   +---- Channel information
   |
   v
Video Filter Decision
```

### Concept

```text
Local Cache
    |
    | Miss
    v
FortiGuard
    |
    +---- Rated
    |      ↓
    |    Decision
    |
    +---- Rating Failure
           ↓
       YouTube API
```

---

# 7. YouTube API Fallback

If FortiGuard cannot provide a video rating, FortiGate can optionally use the **YouTube API**.

### Requirement

A user-provided **YouTube API key** is required.

```text
FortiGuard Rating
       |
       | Failure
       v
YouTube API
       |
       +--> Video Category
       |
       +--> Channel ID
       |
       v
Video Filter
```

### CLI

```bash
config videofilter youtube-key
    edit 1
        set key <YOUTUBE_API_KEY>
    next
end
```

> 🔐 **Security:** Never publish a real YouTube API key in GitHub. Use a placeholder such as `<YOUTUBE_API_KEY>`.

### Important

The YouTube API configuration is **optional**.

```text
FortiGuard
    ↓
If unavailable
    ↓
YouTube API
    ↓
If unavailable
    ↓
IPS Signature Database
```

---

# 8. IPS Application Signature Fallback

If previous classification mechanisms cannot identify the video, WAD can use the IPS/application signature database as another identification mechanism.

```text
Video ID
Channel ID
    |
    v
IPS Application Signature DB
    |
    +---- Match
    |      ↓
    |    Classification
    |
    +---- No Match
           ↓
       No Classification
```

### Why Application Control Matters

Application signatures can help identify applications even when the traffic uses modern protocols such as:

```text
HTTP/2
HTTP/3
QUIC
HTTPS
```

---

# 9. FortiGuard Anycast

FortiGate can use **FortiGuard Anycast** to improve connectivity to FortiGuard services.

### Configuration

```bash
config system fortiguard
    set fortiguard-anycast enable
end
```

### Verify FortiGuard

```bash
get system fortiguard
```

### Concept

```text
FortiGate
    |
    v
FortiGuard Anycast
    |
    +--> Nearest / optimal FortiGuard service
    |
    v
Rating / Security Intelligence
```

### Why Enable Anycast?

Potential benefits include:

* Better service reachability
* Improved response time
* Faster access to FortiGuard services
* More efficient geographic service selection

> **Important:** Anycast does not mean that every query is literally sent to a single fixed FortiGuard IP. It provides an anycast service architecture for FortiGuard connectivity.

---

# 10. YouTube Channel Filtering

Video Filter can be used to filter videos based on their **YouTube channel ID**.

### Configuration

```bash
config videofilter youtube-channel-filter
    edit "test"
        set default-action monitor

        config entries
            edit 1
                set action block
                set channel-id <YOUTUBE_CHANNEL_ID>
            next
        end
    next
end
```

### Logic

```text
YouTube Video
      |
      v
Extract Channel ID
      |
      v
Channel Filter
      |
 +----+----+
 |         |
Match    No Match
 |         |
 v         v
BLOCK    Default Action
```

### Example

```text
Default Action:
MONITOR

Channel:
<YOUTUBE_CHANNEL_ID>

Matched:
BLOCK
```

> **Important:** A YouTube channel's human-readable name and its actual channel ID are not necessarily the same thing. Use the channel ID expected by the FortiGate configuration.

---

# 11. YouTube API Key

The YouTube API key is used when FortiGate needs to query Google's API for video information.

### Configuration

```bash
config videofilter youtube-key
    edit 1
        set key <YOUTUBE_API_KEY>
    next
end
```

### Flow

```text
FortiGate
    |
    v
YouTube API
    |
    +--> Video ID
    |
    +--> Channel ID
    |
    +--> Category
```

### Security Best Practice

Never commit this:

```bash
set key AIxxxxxxxxxxxxxxxxxxxx
```

to a public repository.

Use:

```bash
set key <YOUTUBE_API_KEY>
```

instead.

---

# 12. Recommended Firewall Policy

For a production deployment:

```text
Firewall Policy
│
├── Proxy-based inspection
│
├── SSL Deep Inspection
│
├── Web Filter
│
├── Video Filter
│
└── Application Control
```

### Recommended Concept

```text
Client
  |
  v
Firewall Policy
  |
  +--> Proxy Inspection
  |
  +--> SSL Deep Inspection
  |
  +--> Application Control
  |
  +--> Web Filter
  |
  +--> Video Filter
  |
  v
Internet
```

### Why Application Control?

Application Control can help identify applications independently of simple URL/category matching.

This becomes particularly useful when:

```text
YouTube
  +
HTTPS
  +
QUIC
  +
HTTP/3
```

are involved.

---

# 13. QUIC / HTTP3 Considerations

Modern browsers may use:

```text
HTTP/3
   ↓
QUIC
   ↓
UDP/443
```

instead of:

```text
HTTP/2
   ↓
TCP/443
```

### Potential Problem

Some inspection features can behave differently when applications use QUIC/HTTP3.

```text
YouTube
   |
   +---- TCP/443
   |       ↓
   |    HTTPS
   |
   +---- UDP/443
           ↓
          QUIC
           ↓
        HTTP/3
```

### Recommended Approach

Use **Application Control** together with the required inspection profiles to identify and control YouTube/application traffic.

```text
Application Control
        +
Web Filter
        +
Video Filter
        +
SSL Inspection
```

### Important Design Decision

If the security requirement is to inspect application-layer content that cannot be properly processed through the selected inspection path, you may need to control QUIC/HTTP3 behavior according to your FortiOS version and inspection requirements.

> **Do not blindly block UDP/443.** First determine whether the application and inspection stack actually require QUIC to be restricted.

---

# 14. FortiGuard License & Updates

Check FortiGuard status:

```bash
get system fortiguard
```

### Verify

```text
FortiGuard Connectivity
        ↓
License
        ↓
Web Filtering Service
        ↓
Video Filtering Intelligence
```

### Important

Web and video filtering depend on current security intelligence and service availability.

Before troubleshooting Video Filter, verify:

* FortiGuard connectivity
* Subscription/license status
* DNS resolution
* Routing
* Anycast configuration
* Service reachability

---

# 15. WAD Diagnostics

**WAD (Web Application/Proxy Daemon)** is an important component for proxy-based web processing.

### Basic WAD Test

```bash
diagnose test app wad 1000
```

This can provide information about WAD processes/workers and diagnostic state.

### Conceptual Output

```text
WAD
 |
 +-- Supervisor / Main Process
 |
 +-- Worker Processes
 |
 +-- Web Processing
 |
 +-- Video Filter
 |
 +-- FortiGuard Agent
```

> Diagnostic output varies by FortiOS version and platform.

---

# 16. Video Filter Diagnostic Commands

The following WAD diagnostic functions are useful for Video Filter troubleshooting.

|    ID | Function                                   |
| ----: | ------------------------------------------ |
| `321` | Display Video Filter cache statistics      |
| `322` | Reset Video Filter cache statistics        |
| `323` | Flush Video Filter cache entries           |
| `324` | Display Video Filter module statistics     |
| `325` | Request category list from YouTube API     |
| `326` | Display FortiGuard agent module statistics |
| `327` | Reset FortiGuard agent module statistics   |
| `328` | Toggle Video Filter cache checking         |
| `329` | Toggle Video Filter FortiGuard query       |
| `330` | Toggle Video Filter API checking           |

### Examples

#### Cache Statistics

```bash
diagnose test app wad 321
```

#### Reset Cache Statistics

```bash
diagnose test app wad 322
```

#### Flush Cache

```bash
diagnose test app wad 323
```

#### Video Filter Module Statistics

```bash
diagnose test app wad 324
```

#### YouTube API Category Query

```bash
diagnose test app wad 325
```

#### FortiGuard Agent Statistics

```bash
diagnose test app wad 326
```

#### Reset FortiGuard Agent Statistics

```bash
diagnose test app wad 327
```

---

# 17. WAD Video Debugging

Enable verbose WAD debugging:

```bash
diagnose wad debug enable level verbose
```

Enable Video Filter debugging:

```bash
diagnose wad debug enable category video
```

### Debug Flow

```text
WAD Debug
    |
    +--> Client Request
    |
    +--> Video ID
    |
    +--> Cache Lookup
    |
    +--> FortiGuard Query
    |
    +--> YouTube API
    |
    +--> Application Signature
    |
    v
Final Decision
```

### Recommended Troubleshooting Sequence

```text
1. Verify policy
2. Verify proxy inspection
3. Verify SSL inspection
4. Verify Video Filter
5. Verify Application Control
6. Verify FortiGuard
7. Check local cache
8. Check FortiGuard query
9. Check YouTube API
10. Check IPS/application signatures
11. Enable WAD video debugging
```

---

# 18. Troubleshooting Flow

## Scenario: YouTube Video Is Not Blocked

```text
YouTube Video
      |
      v
Is traffic matching the correct firewall policy?
      |
   +--+--+
   |     |
  NO    YES
   |     |
   v     v
Fix    Proxy Mode?
Policy     |
         +--+--+
         |     |
        NO    YES
         |     |
         v     v
       Fix   Deep Inspection?
                 |
              +--+--+
              |     |
             NO    YES
              |     |
              v     v
            Fix   Video Filter?
                         |
                      +--+--+
                      |     |
                     NO    YES
                      |     |
                      v     v
                    Fix   Application Control?
                                  |
                               +--+--+
                               |     |
                              NO    YES
                               |     |
                               v     v
                             Fix   FortiGuard?
                                         |
                                      +--+--+
                                      |     |
                                     NO    YES
                                      |     |
                                      v     v
                                    Fix   WAD Debug
```

---

## Scenario: FortiGuard Rating Fails

Check:

```bash
get system fortiguard
```

Then inspect:

```bash
diagnose test app wad 326
```

and:

```bash
diagnose wad debug enable category video
```

Then evaluate:

```text
FortiGuard
    |
    X
    |
    v
YouTube API
    |
    X
    |
    v
IPS/Application Signature
```

---

# 19. Deployment Checklist

## Firewall Policy

* [ ] Correct firewall policy matches YouTube traffic.
* [ ] Proxy-based inspection enabled.
* [ ] SSL Deep Inspection configured where required.
* [ ] Video Filter attached.
* [ ] Application Control attached.
* [ ] Web Filter attached where required.

## FortiGuard

* [ ] FortiGuard license verified.
* [ ] FortiGuard connectivity verified.
* [ ] FortiGuard Anycast evaluated/enabled where appropriate.
* [ ] Web filtering service operational.
* [ ] Video rating service reachable.

## Video Filter

* [ ] Video Filter enabled.
* [ ] Default action reviewed.
* [ ] YouTube channel IDs verified.
* [ ] Channel filtering tested.
* [ ] Local cache behavior verified.

## YouTube API

* [ ] API fallback required?
* [ ] YouTube API key configured if required.
* [ ] API quota considered.
* [ ] API key protected.
* [ ] API key excluded from GitHub/public documentation.

## Application Control

* [ ] YouTube application signatures available.
* [ ] Application Control profile applied.
* [ ] QUIC/HTTP3 behavior tested.
* [ ] TCP/443 behavior tested.
* [ ] UDP/443 behavior tested.

## Troubleshooting

* [ ] WAD statistics checked.
* [ ] Video Filter cache checked.
* [ ] FortiGuard agent statistics checked.
* [ ] YouTube API test performed if required.
* [ ] WAD Video debug enabled during testing.

---

# 20. NSE Exam Traps

## 🧠 Trap #1 — Proxy Mode

```text
Video Filter
     ↓
Proxy-based inspection
```

If the question focuses on Video Filter processing requirements, **proxy inspection is a key consideration**.

---

## 🧠 Trap #2 — Local Cache Comes First

The first place WAD attempts to obtain a previously known video classification is the local Video Filter cache.

```text
Video Request
     ↓
Local Cache
```

Only when there is no usable cache result does the external rating process become necessary.

---

## 🧠 Trap #3 — FortiGuard Before YouTube API

Conceptually:

```text
Local Cache
     ↓
FortiGuard
     ↓
YouTube API
```

The YouTube API is a fallback mechanism, not the normal first lookup.

---

## 🧠 Trap #4 — YouTube API Key

The YouTube API fallback requires a user-provided API key.

```text
YouTube API
     +
API Key
```

No API key:

```text
YouTube API fallback
        ↓
Not available
```

---

## 🧠 Trap #5 — Channel Name ≠ Channel ID

Do not confuse:

```text
Channel Name
```

with:

```text
Channel ID
```

Video filtering uses the identifier expected by the FortiGate configuration.

---

## 🧠 Trap #6 — Application Control

Application Control can provide an additional identification/control mechanism for applications such as YouTube.

```text
Video Filter
     +
Application Control
```

This is especially useful when modern transport protocols are involved.

---

## 🧠 Trap #7 — QUIC

Remember:

```text
HTTP/3
   ↓
QUIC
   ↓
UDP/443
```

Not:

```text
HTTP/3
   ↓
TCP/443
```

---

## 🧠 Trap #8 — FortiGuard Anycast

Anycast is related to FortiGuard service connectivity.

```bash
config system fortiguard
    set fortiguard-anycast enable
end
```

It is not itself a Video Filter policy.

---

## 🧠 Trap #9 — Cache vs Cache Statistics

These are different operations:

```bash
diagnose test app wad 321
```

→ Display cache statistics

```bash
diagnose test app wad 323
```

→ Flush cache entries

---

## 🧠 Trap #10 — Debug Category

For Video Filter troubleshooting:

```bash
diagnose wad debug enable category video
```

Do not confuse this with generic WAD debugging.

---

# 21. Quick Revision Card

```text
                    VIDEO FILTER
                         |
                         v
                        WAD
                         |
                         v
                 Extract Video ID
                         |
                         v
                  LOCAL CACHE
                         |
                    +----+----+
                    |         |
                   HIT       MISS
                    |         |
                    v         v
                 RESULT   FORTIGUARD
                              |
                         +----+----+
                         |         |
                        HIT       FAIL
                         |         |
                         v         v
                      RESULT   YOUTUBE API
                                   |
                              +----+----+
                              |         |
                             HIT       FAIL
                              |         |
                              v         v
                           RESULT   IPS SIGNATURE
                                         |
                                    +----+----+
                                    |         |
                                   HIT       FAIL
                                    |         |
                                    v         v
                                 RESULT    NO MATCH
```

---

## 🔥 Core Configuration

### FortiGuard Anycast

```bash
config system fortiguard
    set fortiguard-anycast enable
end
```

### YouTube Channel Filter

```bash
config videofilter youtube-channel-filter
    edit "test"
        set default-action monitor

        config entries
            edit 1
                set action block
                set channel-id <YOUTUBE_CHANNEL_ID>
            next
        end
    next
end
```

### YouTube API Key

```bash
config videofilter youtube-key
    edit 1
        set key <YOUTUBE_API_KEY>
    next
end
```

---

## 🔍 Most Useful Diagnostics

```bash
get system fortiguard

diagnose test app wad 1000

diagnose test app wad 321
diagnose test app wad 322
diagnose test app wad 323
diagnose test app wad 324
diagnose test app wad 325
diagnose test app wad 326
diagnose test app wad 327
diagnose test app wad 328
diagnose test app wad 329
diagnose test app wad 330

diagnose wad debug enable level verbose
diagnose wad debug enable category video
```

---

# 🎯 30-Second Interview / NSE Summary

```text
VIDEO FILTER
│
├── Requires proxy-based inspection
│
├── WAD extracts Video ID
│
├── 1. Local Video Filter Cache
│
├── 2. FortiGuard Video Rating
│
├── 3. YouTube API
│      └── Requires user's API key
│
├── 4. IPS/Application Signature DB
│
├── Channel filtering
│      └── Channel ID
│
├── FortiGuard Anycast
│      └── Better FortiGuard service connectivity
│
└── Troubleshooting
       ├── WAD statistics
       ├── Video cache
       ├── FortiGuard agent
       ├── YouTube API
       └── WAD video debug
```

---

# ⭐ One-Line NSE Memory Aid

> **WAD receives the video request, checks the local cache first, then uses FortiGuard rating, optionally falls back to the YouTube API, and finally can use application signatures to identify the video when earlier classification methods do not provide a match.**

---

## 🏷️ Tags

`fortigate` `fortios` `video-filter` `youtube` `fortiguard` `wad` `application-control` `quic` `http3` `network-security` `cybersecurity` `nse7` `nse4` `-`

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

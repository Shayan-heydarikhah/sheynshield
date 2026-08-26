# SheynShield Cisco Wireless & RF Fundamentals Repository
A deep‑technical, engineering‑grade repository focused on Cisco wireless fundamentals, RF behavior, WLAN architecture, CAPWAP, WLC deployment models, security mechanisms, modulation, spectrum analysis, and antenna/link‑budget design.

This repository is built for wireless engineers, service‑provider designers, enterprise architects, SOC/NOC teams, and CCNP/ENCOR/ENARSI candidates who need real RF understanding, not marketing‑level Wi‑Fi explanations.

All notes are extracted from real deployments, lab‑tested behaviors, and hand‑written RF analysis, covering everything from 802.11 PHY/MAC, antenna gain, EIRP, modulation, WLC split‑MAC, CAPWAP tunnels, WPA2/WPA3 security, and RF interference models.

---

Why This Repository Matters
Modern wireless networks are dense, multi‑channel, multi‑band, and heavily interference‑prone.  
This repo gives you:

- Real RF behavior, not simplified diagrams  
- Accurate 2.4/5 GHz channel planning  
- Deep modulation & spectrum explanations  
- WLC split‑MAC architecture  
- CAPWAP control/data plane internals  
- Security mechanisms (WEP → WPA3)  
- Antenna gain, EIRP, link‑budget math  
- Wireless troubleshooting logic used in SP/Enterprise networks

If you're designing campus WLAN, SP Wi‑Fi offload, metro‑WiFi, or preparing for Cisco ENCOR/ENARSI, this repository is built for you.

---

📡 Wireless Architecture Overview

RF Basics
- Wireless uses RF waves (3 kHz – 300 GHz)  
- Wi‑Fi operates on 2.4 GHz and 5 GHz  
- Half‑duplex medium → airtime contention  
- CSMA/CA governs access  
- Interference affects airtime, throughput, and modulation stability

Service Sets
- BSS — AP + clients  
- ESS — multiple APs forming roaming domain  
- IBSS — ad‑hoc peer‑to‑peer  
- BSSID — MAC identifier of each AP radio  
- SSID — logical WLAN name

AP Modes
- Local  
- Monitor  
- FlexConnect  
- Sniffer  
- Bridge / Mesh  
- Rogue Detector

---

🔧 RF Engineering & Spectrum Concepts

Frequency, Wavelength, Phase
- 2.4 GHz → ~4.92 inch wavelength  
- 5 GHz → ~2.36 inch wavelength  
- Higher frequency = more bandwidth, less coverage

Modulation
- DSSS → 22 MHz channels  
- OFDM → 20 MHz channels  
- Voltage‑based bit encoding  
- Multi‑bit symbol mapping (00, 01, 10, 11)

Channels
- Recommended non‑overlapping: 1 / 6 / 11  
- UNII bands for 5 GHz:  
  - UNII‑1: 5180–5240  
  - UNII‑2: 5260–5320  
  - UNII‑2e: 5500–5700  
  - UNII‑3: 5745–5845

Interference Types
- Co‑channel  
- Adjacent‑channel  
- Noise floor  
- SNR (primary quality metric)

---

📶 Antenna, EIRP & Link Budget

Antenna Gain
- dBi (isotropic reference)  
- Directional vs omni  
- Gain increases direction, not power

EIRP Calculation
`
TX Power (dBm)
– Cable Loss (dB)
+ Antenna Gain (dBi)
= EIRP
`

Link Budget
`
RX Power = TX Power – FSPL – Cable Loss + RX Antenna Gain
`

RSSI Quality
- 0–20 dBm → Excellent  
- 20–50 dBm → Good  
- 50–70 dBm → Weak  
- 70–100 dBm → Poor

---

🔐 Wireless Security

Authentication
- Open  
- WEP (RC4, deprecated)  
- 802.1X / EAP  
- LEAP (deprecated)  
- EAP‑FAST  
- PEAP  
- EAP‑TLS (certificate‑based)

Encryption
- TKIP (deprecated)  
- CCMP (AES) — WPA2  
- GCMP (AES) — WPA3  
- MIC, replay protection, sequence numbers

4‑Way Handshake
- WPA/WPA2 key establishment  
- WPA3 SAE → forward secrecy

---

🖧 Cisco Wireless Architecture

Autonomous AP
- Standalone  
- Local switching  
- Manual channel/power control  
- Suitable for small deployments

Split‑MAC Architecture (Lightweight AP + WLC)
- AP handles RF, WLC handles control/data  
- CAPWAP tunnels:  
  - UDP 5246 → Control  
  - UDP 5247 → Data  
- Centralized authentication, QoS, channel assignment

WLC Features
- DCA (Dynamic Channel Assignment)  
- TPC (Transmit Power Control)  
- CHDM (Coverage Hole Detection)  
- Load balancing  
- RF monitoring
- Rogue detection  
- Multi‑tenant support

Deployment Models
- Unified / Centralized  
- Cloud‑based (vWLC)  
- Embedded WLC (3850/4500/6500)  
- Mobility Express

---

🚀 Vision of This Repository
To build the most accurate, engineer‑friendly, free, and deep Cisco wireless knowledge base — without shortcuts, without shallow explanations, and without vendor marketing fluff.

Continuous updates include:
- New RF troubleshooting cases  
- Advanced WLC designs  
- Multi‑AP roaming behavior  
- Spectrum analysis scenarios  
- Security enhancements  
- Real‑world deployment notes  

---

📬 Connect
- YouTube: @sheynshield  
- LinkedIn: /in/shayan-heydarikhah  
- Telegram: t.me/sheynshield  

---

⭐ Support the Project
If this repository helped you, consider starring ⭐ the repo — it increases visibility and helps more engineers learn real RF fundamentals.

---

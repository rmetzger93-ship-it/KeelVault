---
created: 2025-10-14T17:00:44-04:00
modified: 2025-10-14T17:01:00-04:00
---

# SessionSummary_2025-10-14_GPT-5-Thinking_Pi_v1

# Session Summary • 2025-10-14 • GPT-5-Thinking_Pi
**Instance ID:** GPT-5-Thinking_Pi  
**Validated By:** RJ  
**Change Type:** Addition  
**Summary:** Designed a JB5000 replacement wiring plan to integrate a Navico WP5000 autopilot with the Signal K server using an Elfin EW10 serial-to-Wi-Fi bridge (instead of a USB–NMEA adapter). Produced configuration, wiring map, test procedure, and hardening notes.

---

## 🧠 Overview
Focus was on enabling the WP5000 to communicate with Signal K without the original Navico JB5000 junction box by building a compact “DIY JB5000” and bridging NMEA 0183 over Wi-Fi via an Elfin EW10.

## ✅ Completed
- **Architecture decided:** WP5000 → DIY JB box → Elfin EW10 (TCP Server) → Signal K NMEA 0183 TCP input.
- **Wiring plan (color/function mapping):**
  - Power: **Red = +12 V (10 A fuse)**, **Blue/Black = DC-** (common).
  - NMEA to WP5000 (pilot input): **Yellow = Rx+**, **Green = Rx−** ← connect to EW10 **TX+ / TX−**.
  - NMEA from WP5000 (pilot output): **White = Tx+**, **Brown = Tx−** → connect to EW10 **RX+ / RX−**.
  - Shield: tie to ground **at EW10 side only**.
- **Elfin EW10 target config:**
  - Serial: 4800 bps, 8-N-1, **RS-422**, no flow control.
  - Network: **TCP Server**, port **10110** (or chosen port), join same Wi-Fi as Signal K.
- **Signal K setup guidance:**
  - Add **NMEA 0183 → TCP client** to `EW10_IP:10110`, 4800 bps.
- **Test & bring-up checklist:** staged power-up, polarity swap hint, sentence visibility in SK Data Browser, safe dockside command test.
- **Hardening suggestions:** fused feeds (10 A for pilot, ~1 A for EW10 tap), twisted/shielded pairs, optional DC-DC isolation, static IP for EW10.

## ⚠️ Issues / Failures
- None executed on hardware in this session; plan is ready but unverified on the bench/boat.
- Exact WP5000 variant baud rate assumed **4800**; verify in manual/label before finalizing.

## 📘 Lessons Learned
- JB5000’s role is a **passive distribution hub**; a neat terminal block in a waterproof box reproduces its function for power and NMEA pairs.
- Using **EW10** avoids USB runs to the helm and fits well with RJ’s existing EW10 ecosystem.

## 🧩 Dependencies / Links
- **Manual:** *Navico JB5000 Manual_copy.pdf* (uploaded by RJ; use for pinout confirmation).
- **Living Manual:** Boat computer docs (Signal K, GPS bridge, EW10 usage).
- 🔒 **Private References (Keep):** EW10 Wi-Fi SSID/IP scheme, boat network map, fuse ratings/locations.

## ⏭️ Next Recommended Actions
1. Build the DIY JB box (IP65+ enclosure, terminal strip, labels, strain relief).
2. Bench-wire WP5000 ↔ EW10 and validate serial directions (TX/RX).
3. Configure EW10 (RS-422, 4800, TCP Server @ chosen port) and assign static IP.
4. Add NMEA 0183 TCP connection in Signal K; confirm sentences appear.
5. Dockside control test (rudder commands, route follow) with safety observer.
6. If noise/loop issues appear, add DC-DC isolation for EW10 power and/or opto-isolated NMEA interface module.
7. Document final wiring and screenshots in the Living Manual; archive enclosure photos.

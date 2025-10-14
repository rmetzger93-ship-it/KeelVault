---
created: 2025-10-14T17:00:22-04:00
modified: 2025-10-14T17:00:40-04:00
---

# Handoff_2025-10-14_GPT-5-Thinking_Pi_to_Next_v1

# Handoff • 2025-10-14 • GPT-5-Thinking_Pi → Next Instance
**Updated By:** GPT-5-Thinking_Pi  
**Validated By:** RJ  
**Change Type:** Addition  
**Summary:** Operational handoff to build and validate the WP5000 ↔ EW10 bridge and complete Signal K integration.

---

## 🔗 Links
- **KeelVault (public reference):** https://github.com/rmetzger93-ship-it/KeelVault  
- **Private Docs Loop:** Google Keep → Private Info → Inputs 🔒

## 📘 Key Documents
- *Navico JB5000 Manual_copy.pdf* (uploaded this session) — confirm pinouts & color map.
- Living Manual sections: **Autopilot Integration**, **EW10 Bridges**, **Signal K NMEA 0183 Inputs**.
- Any enclosure/wiring photos to be captured post-build (add paths when created).

## ⚙️ Environment & Mode
- **Sight Mode:** Assisted (no repo read; working from user-provided manual & prior context).  
- **Connectors:** None used this session (no Drive/GitHub read).  
- **Environment:** Offline-capable Pi with Signal K; existing EW10 in system; adding **second** EW10 for autopilot bridge.

## 📊 Summary of Work
- Produced a **complete wiring and configuration plan** to replace the JB5000 using a small terminal-box and an **Elfin EW10** set as a **TCP Server** (4800 bps, RS-422).
- Defined Signal K configuration and end-to-end validation steps with troubleshooting tips (polarity swap, isolation notes).

## 🪄 Next Actions
1. **Build the DIY JB box** (IP65+, terminal strip, 10 A inline fuse for WP5000 feed, ~1 A for EW10 tap).
2. **Wire mapping:**
   - Power: Red → +12 V (fused 10 A); Blue/Black → DC−.
   - Pilot NMEA In (from SK via EW10): Yellow → EW10 TX+; Green → EW10 TX−.
   - Pilot NMEA Out (to SK via EW10): White → EW10 RX+; Brown → EW10 RX−.
   - Shield → ground at EW10 side only.
3. **Configure EW10:**
   - Serial: RS-422, 4800, 8-N-1, no flow.
   - Network: TCP Server, port **10110** (or assigned), static IP reservation.
4. **Configure Signal K:**
   - Add NMEA 0183 TCP client → `EW10_IP:10110`, 4800.
   - Confirm sentences appear in Data Browser; map to autopilot paths.
5. **Run dockside tests:** rudder jog, mode changes, route follow (safe, lines on).
6. **If needed:** add DC-DC isolation for EW10, review grounding; retest.
7. **Document results:** finalize wiring diagram, SK screenshots, EW10 settings; update Living Manual and KeelVault.

## 🧩 Pending / Open Tasks
- [ ] Verify WP5000 variant baud rate (assumed 4800) in the manual/label.
- [ ] Bench test EW10 serial polarity and confirm sentence flow.
- [ ] Assign static IP and record in 🔒 Keep (Network Map).
- [ ] Add SK TCP input and confirm data paths populate.
- [ ] Perform controlled dockside command test.
- [ ] Capture photos/diagram; commit to Living Manual & KeelVault.

✅ **Status:** Open (handoff in progress)

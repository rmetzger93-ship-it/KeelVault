---
created: 2025-10-14T17:06:20-04:00
modified: 2025-10-14T17:09:14-04:00
---

# Handoff_2025-10-14_Eclipse_to_Next_v2

# Handoff • 2025-10-14 • Eclipse → Next Instance  
**Updated By:** Eclipse  
**Validated By:** RJ  
**Change Type:** Addition  
**Summary:** Continue by verifying live NMEA output on :10112, ensure Signal K ingestion, confirm time sync behavior, then begin AIS receiver setup and integration.

---

## 🔗 Links  
- **KeelVault (public reference):** https://github.com/rmetzger93-ship-it/KeelVault  
- **Private Docs Loop:** Google Keep → Private Info → Inputs 🔒

## 📘 Key Documents  
- `/etc/systemd/system/gps-nmea-tcp.service` — GPS→NMEA TCP bridge (authoritative)  
- `/home/pi/.signalk/settings.json` — Signal K sources (ensure UDP/TCP input for localhost:10112)  
- Living Manual — “Raspberry Pi Boat Computer” (status checks & service map)

## ⚙️ Environment & Mode  
- **Platform:** Raspberry Pi (Eclipse)  
- **GPS:** u-blox (USB, `/dev/ttyACM0`) via `gpsd`  
- **Bridge:** `gpspipe -r | socat ... :10112` (localhost)  
- **Consumers:** Signal K (NMEA0183 TCP/UDP input)  
- **Planned:** AIS via RTL-SDR, engine sensors (ESP32), Elfin EW10 links, Victron BMV via BT

## 📊 Summary of Work  
- Pruned duplicate/legacy GPS services & socket activation to avoid port/contention issues.  
- Established single canonical GPS bridge on TCP :10112.  
- Captured a minimal, copy/paste-safe diagnostic flow for services, ports, and NMEA output.

## 🪄 Next Actions  
1. **Verify data on the wire**
   ```bash
   sudo systemctl status gpsd gps-nmea-tcp signalk | cat
   nc 127.0.0.1 10112 | head -10
Expect $GPRMC, $GPGGA, $GPGSV, etc. If empty:


sudo systemctl restart gpsd gps-nmea-tcp
sleep 3
nc 127.0.0.1 10112 | head -10

2. Confirm Signal K ingest

In Signal K Admin → Data Connections: ensure TCP/UDP listener for 127.0.0.1:10112 (NMEA0183).

Check /signalk/v1/api/vessels/self/ for position updates.



3. Time discipline

timedatectl
chronyc sources -v
chronyc tracking

Expect system clock synchronized once GPS has a 2D/3D fix and Chrony is configured to use SHM/NMEA (if applicable).



4. Begin AIS

Install RTL-SDR, test rtl_test, then run rtl_ais (or equivalent) and pipe to Signal K (UDP 10110 or TCP input).

Add a dedicated systemd service once stable.




🧩 Pending / Open Tasks

[ ] Confirm continuous NMEA output on :10112 after cold boot.

[ ] Validate Signal K paths show live GPS data.

[ ] Verify/adjust Chrony configuration for GPS time (if using).

[ ] Implement AIS receiver service and feed into Signal K.

[ ] Update the living manual and trigger a full SD image backup.


✅ Status: Open (handoff in progress)


---

🔐 Security & Privacy Discipline

No credentials, Wi-Fi secrets, or PII included.

Refer to network specifics and device IPs via 🔒 Private References in Google Keep only.

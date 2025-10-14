---
created: 2025-10-14T17:27:16-04:00
modified: 2025-10-14T17:27:26-04:00
---

# Handoff_2025-10-14_EclipsePiGPS_to_Next_v1

# Handoff • 2025-10-14 • EclipsePiGPS → Next Instance
**Updated By:** EclipsePiGPS  
**Validated By:** RJ  
**Change Type:** Addition  
**Summary:** GPS pipeline is online and persistent; next instance should resolve time sync and perform reboot validation.

---

## 🔗 Links
- KeelVault (public reference): https://github.com/rmetzger93-ship-it/KeelVault  
- 🔒 Private Docs Loop: Google Keep → Private Info → Inputs

---

## 📘 Key Documents
- `/etc/systemd/system/gpsd-nmea.service` — gpsd→NMEA bridge (TCP **10120**)
- `/etc/default/gpsd` — `DEVICES="/dev/ttyACM0"`, `GPSD_OPTIONS="-n"`
- `/usr/local/bin/check-boat-status` — health checker (services, port 10120, NMEA stream, time sync)

---

## ⚙️ Environment & Mode
- Raspberry Pi (“Eclipse”), Signal K server active.
- gpsd is **socket-activated** on **127.0.0.1:2947**.
- NMEA bridge exports GPS to **127.0.0.1:10120** for Signal K.
- Sight Mode: Assisted; prefers step-by-step with pauses for outputs.

---

## 📊 Summary of Work
- Removed stale systemd overrides and orphaned gpsd instances.
- Rebuilt gpsd pipeline and confirmed NMEA sentences end-to-end.
- Updated monitoring script to reflect new ports and checks.

---

## 🪄 Next Actions
1. **Time Sync Repair**
   - `sudo systemctl status systemd-timesyncd`
   - If inactive, enable: `sudo systemctl enable --now systemd-timesyncd`
   - Verify: `timedatectl show | grep -E 'NTPSynchronized|SystemClockSynchronized'`
   - Optionally install `chrony` and configure if stricter timing needed.
2. **Reboot Validation**
   - Reboot Pi.
   - Run `check-boat-status` and confirm:
     - gpsd socket active
     - gpsd-nmea service active
     - NMEA data “OK”
     - Signal K shows live position
3. **Optional Enhancements**
   - Add `--verbose` to `check-boat-status` (PIDs, satellite count via `gpspipe -w`).
   - Document latency target (<5s from boot to first fix in Signal K).

---

## 🧩 Pending / Open Tasks
- [ ] Resolve “System clock synchronized: no”
- [ ] Reboot test and record results in KeelVault
- [ ] Add `--verbose` mode to status script

✅ **Status:** Open (handoff in progress)

---

## 🔐 Security & Privacy Discipline
- No passwords, tokens, or PII stored here.  
- Hardware paths (e.g., `/dev/ttyACM0`) referenced; full network details remain 🔒 in Private References.  
- Continue step-by-step workflow: run a command, capture outputs, pause, proceed.

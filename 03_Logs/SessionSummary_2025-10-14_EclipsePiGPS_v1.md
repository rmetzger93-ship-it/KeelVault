---
created: 2025-10-14T17:27:57-04:00
modified: 2025-10-14T17:28:10-04:00
---

# SessionSummary_2025-10-14_EclipsePiGPS_v1

Session Summary • 2025-10-14 • EclipsePiGPS

Instance ID: EclipsePiGPS
Validated By: RJ
Change Type: Addition
Summary: GPS subsystem repaired, stabilized, and verified. Added a persistent gpsd→NMEA bridge on TCP 10120 and updated the check-boat-status monitor script.


---

🧠 Overview

Focused on restoring a broken GPS stream and ensuring stable startup behavior. Implemented a gpsd-driven pipeline with a dedicated NMEA bridge for Signal K. ✅ Completed

Key actions & artifacts

Cleaned up stale gpsd overrides and killed orphan daemons.

Restored socket-activated gpsd on 127.0.0.1:2947.

Created persistent bridge /etc/systemd/system/gpsd-nmea.service that runs:
/usr/bin/gpspipe -r 127.0.0.1:2947 | /usr/bin/socat -u - TCP-LISTEN:10120,reuseaddr,fork

Pointed Signal K NMEA 0183 TCP Client → 127.0.0.1:10120.

Fixed /etc/default/gpsd to use DEVICES="/dev/ttyACM0" and GPSD_OPTIONS="-n".

Rewrote /usr/local/bin/check-boat-status to:

Check core services: gpsd, gpsd-nmea, signalk, pypilot

Confirm listener on 10120

Validate live NMEA $GP/$GN stream

Show time sync status




---

⚠️ Issues / Failures

gpsd.service “dependency failed” due to a stale override (removed).

gpsd.socket initially failed because manual gpsd held port 2947 (fixed).

Time sync still reports System clock synchronized: no (pending).



---

📘 Lessons Learned

Prefer socket-activation (gpsd.socket) and avoid manually launching gpsd.

Validate data at each hop: device → gpsd (2947) → bridge (10120) → Signal K.

For port checks in user context, rely on ss/netstat without requiring sudo.



---

🧩 Dependencies / Links

Bridge service: /etc/systemd/system/gpsd-nmea.service

gpsd config: /etc/default/gpsd

Monitor script: /usr/local/bin/check-boat-status

Signal K input: TCP client → 127.0.0.1:10120

🔒 Private Reference (Keep): u-blox GPS on /dev/ttyACM0



---

⏭️ Next Recommended Actions

1. Fix time sync: check systemd-timesyncd/chrony and integrate GPS time if desired.


2. Reboot test: ensure gpsd socket + bridge autostart and Signal K shows position within seconds.


3. Optional: add `

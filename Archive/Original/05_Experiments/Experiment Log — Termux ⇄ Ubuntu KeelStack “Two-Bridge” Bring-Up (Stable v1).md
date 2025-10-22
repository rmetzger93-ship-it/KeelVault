---
created: 2025-10-15T15:38:27-04:00
modified: 2025-10-15T15:39:01-04:00
---

# Experiment Log — Termux ⇄ Ubuntu KeelStack “Two-Bridge” Bring-Up (Stable v1)

ID: EXP-KEEL-TER-UBU-2BR-0001
Owner: RJ
System: Android (Termux) → proot-distro Ubuntu (shared /tmp)
Goal: Achieve reliable localhost access to Keel services from Termux layer by bridging to Ubuntu daemons via UNIX domain sockets in shared /tmp, with clean startup / shutdown SOPs and web app functional test.
Result: ✅ Success — stable dual bridge with reproducible boot/stop flows.

1) Context & Motivation

Historically, the stack oscillated between:

Processes appearing alive in Ubuntu (ps shows ollama, webchat.py, keel_manager.py)

…but Termux curl probes failing (Expecting value: line 1 column 1), usually due to:

stale Termux listeners still bound to 11434 or 5000

Ubuntu daemons trying to rebind to already-occupied ports

non-shared socket paths (Termux couldn’t see Ubuntu sockets)

bridges pointed to /root/offline-keel/sockets instead of a shared /tmp path

Objective of this run: establish a repeatable boot/stop that guarantees port hygiene and cross-namespace visibility, then validate with positive health checks from both Ubuntu and Termux layers.

2) Environment Snapshot (Ground Truth from Session)

Termux (Android app) — no root, standard limitations on ss/netlink (permission denied).

proot-distro Ubuntu — launched with --shared-tmp (critical).

Ollama: 0.12.5 (binary at /usr/local/bin/ollama)

Python: 3.13.x inside keelenv

Pip: 25.2

Key Python deps: flask, requests, tqdm, gunicorn, psutil

Project root: /root/offline-keel

Services:

ollama serve → 127.0.0.1:11434 (Ubuntu)

webchat.py (Flask) → 127.0.0.1:5000 (Ubuntu)

keel_manager.py (Flask) → 127.0.0.1:5050 (Ubuntu)

Bridging strategy:

Inside Ubuntu: socat UNIX-LISTEN:/tmp/offline-keel-sockets/{ollama,webchat}.sock … TCP:127.0.0.1:{11434,5000}

In Termux: socat TCP-LISTEN:{11434,5000} … UNIX-CONNECT: … /ubuntu/tmp/offline-keel-sockets/{ollama,webchat}.sock

Result: Android apps can talk to 127.0.0.1:{11434,5000} from Termux without directly exposing Ubuntu’s TCP.

3) Conventions Used in This Log

Entrance marker (ENTER UBUNTU):
proot-distro login ubuntu --shared-tmp

Exit marker (EXIT UBUNTU):
exit

Termux vs Ubuntu: I always show a separate block per layer. When a step requires Ubuntu, I include the entrance line before the commands and the exit line if we’re returning to Termux.

Hygiene first: every boot flow begins by killing bridges/daemons and clearing sockets/ports.

4) Final Working Procedure (Golden Path)
A) Full Clean + Boot (from a fresh Termux terminal)

[Termux]

# 🧨 1) Kill absolutely everything that could interfere (host side)
pkill -f "socat" 2>/dev/null
pkill -f "proot" 2>/dev/null
pkill -f "ubuntu" 2>/dev/null
pkill -f "python" 2>/dev/null
pkill -f "ollama" 2>/dev/null
sleep 3

# Optional: verify ports look free (Termux often denies netlink; that's ok)
ss -tuln | grep -E "11434|5000|5050" || echo "✅ All ports clear, ready to rebuild"


ENTER UBUNTU

proot-distro login ubuntu --shared-tmp


[Ubuntu]

# Project root
cd /root/offline-keel || { echo "❌ Missing /root/offline-keel"; exit 1; }

# Activate venv
source keelenv/bin/activate

# Fresh shared sockets (CRITICAL: shared /tmp path)
rm -rf /tmp/offline-keel-sockets
mkdir -p /tmp/offline-keel-sockets

# Start daemons cleanly
nohup /usr/local/bin/ollama serve > ollama.log 2>&1 &  # binds 127.0.0.1:11434
sleep 5
nohup python webchat.py > webchat.log 2>&1 &           # binds 127.0.0.1:5000
nohup python keel_manager.py > manager.log 2>&1 &       # binds 127.0.0.1:5050
sleep 5

# Quick local sanity from inside Ubuntu
curl -s http://127.0.0.1:11434/api/version
curl -s http://127.0.0.1:5000/health
curl -s http://127.0.0.1:5050/status
# Expected:
# {"version":"0.12.5"}
# {"models":["keel:latest","phi3:mini"],"ok":true,"ollama":"http://127.0.0.1:11434","timestamp":...}
# {"ollama":true,"uptime":"...","webchat":true}


EXIT UBUNTU

exit


[Termux]

# Host bridges into the Ubuntu shared sockets
nohup socat TCP-LISTEN:11434,reuseaddr,fork \
  UNIX-CONNECT:/data/data/com.termux/files/usr/var/lib/proot-distro/installed-rootfs/ubuntu/tmp/offline-keel-sockets/ollama.sock \
  >/dev/null 2>&1 &

nohup socat TCP-LISTEN:5000,reuseaddr,fork \
  UNIX-CONNECT:/data/data/com.termux/files/usr/var/lib/proot-distro/installed-rootfs/ubuntu/tmp/offline-keel-sockets/webchat.sock \
  >/dev/null 2>&1 &

sleep 3

# Verify bridges are up
ps aux | grep socat | grep -v grep

# End-to-end health (from Termux)
curl -s http://127.0.0.1:11434/api/version
curl -s http://127.0.0.1:5000/health | python3 -m json.tool


Observed Good Result (from the working run):

{"version":"0.12.5"}

WebChat health with ok:true, models ["keel:latest","phi3:mini"]

Manager status shows {"ollama":true,"webchat":true} from Ubuntu side

5) Failure Modes & Fixes (Root-Cause Library)
Symptom	Likely Cause	Fix
Expecting value: line 1 column 1 (char 0) on curl	Bridge pointed to a dead socket path or service not bound yet	Ensure Ubuntu daemons healthy first; ensure shared /tmp/offline-keel-sockets; recreate Termux bridges.
bind: address already in use in ollama.log / Flask ports	Termux still had listeners on 11434/5000, or Ubuntu daemons double-started	Kill all socat, ollama, Python/Flask (webchat.py, keel_manager.py) on both layers; restart clean; only then start daemons and bridges.
Sockets exist but no responses	Sockets created under /root/offline-keel/sockets (not shared)	Always use /tmp/offline-keel-sockets (shared because of --shared-tmp).
am force-stop unknown	Termux’s built-in am wrapper lacks that subcommand	Use Android shell cmd activity force-stop com.termux (via adb/shell) or kill via Recents/app info. See Quick Boot SOP below.
psutil import error in keel_manager.py	Missing dependency in venv	pip install psutil in the venv.
6) Quick Boot SOP (Daily Use)

Two scripts recommended. You can drop these into ~/bin (Termux) and /root/offline-keel/bin (Ubuntu) or just keep them in KeelVault.

6.1 keel-quickboot (Termux wrapper)

[Termux] (create ~/bin/keel-quickboot, chmod 700 ~/bin/keel-quickboot)

#!/data/data/com.termux/files/usr/bin/bash
set -e

echo "🧨 Killing host bridges & leftovers..."
pkill -f "socat" 2>/dev/null || true
pkill -f "ollama" 2>/dev/null || true
pkill -f "python" 2>/dev/null || true
sleep 2

echo "🧭 Entering Ubuntu..."
proot-distro login ubuntu --shared-tmp <<'UBU'
set -e
cd /root/offline-keel
source keelenv/bin/activate

echo "🔧 Recreating shared sockets..."
rm -rf /tmp/offline-keel-sockets
mkdir -p /tmp/offline-keel-sockets

echo "🚀 Starting daemons..."
nohup /usr/local/bin/ollama serve > ollama.log 2>&1 &
sleep 4
nohup python webchat.py > webchat.log 2>&1 &
nohup python keel_manager.py > manager.log 2>&1 &
sleep 4

echo "🩺 Ubuntu health:"
curl -s http://127.0.0.1:11434/api/version || true
curl -s http://127.0.0.1:5000/health || true
curl -s http://127.0.0.1:5050/status || true
UBU

echo "🌉 Starting Termux bridges..."
nohup socat TCP-LISTEN:11434,reuseaddr,fork \
  UNIX-CONNECT:/data/data/com.termux/files/usr/var/lib/proot-distro/installed-rootfs/ubuntu/tmp/offline-keel-sockets/ollama.sock \
  >/dev/null 2>&1 &

nohup socat TCP-LISTEN:5000,reuseaddr,fork \
  UNIX-CONNECT:/data/data/com.termux/files/usr/var/lib/proot-distro/installed-rootfs/ubuntu/tmp/offline-keel-sockets/webchat.sock \
  >/dev/null 2>&1 &

sleep 3
echo "🧪 Termux test:"
curl -s http://127.0.0.1:11434/api/version || true
curl -s http://127.0.0.1:5000/health | python3 -m json.tool || true

echo "✅ Keel quick boot complete."

6.2 keel-stop (Hard Stop, both layers clean)

[Termux] (create ~/bin/keel-stop, chmod 700 ~/bin/keel-stop)

#!/data/data/com.termux/files/usr/bin/bash
set +e
echo "🛑 Killing Termux bridges..."
pkill -f "socat.*11434"
pkill -f "socat.*5000"
sleep 1

echo "🛑 Killing Ubuntu services..."
proot-distro login ubuntu --shared-tmp <<'UBU'
pkill -f "ollama serve" 2>/dev/null
pkill -f "webchat.py" 2>/dev/null
pkill -f "keel_manager.py" 2>/dev/null
sleep 1
rm -rf /tmp/offline-keel-sockets
echo "✅ Ubuntu services stopped."
UBU

echo "✅ All stopped."


Note on “kill the app every time?”
You do not need to force stop the Termux app if you use keel-stop before exiting and then run keel-quickboot to start fresh.
If Android still resurrects background sockets or something gets wedged, use Android shell:

Via adb shell: cmd activity force-stop com.termux

Or Android Settings → Apps → Termux → Force stop.
The Termux-embedded am wrapper in your build didn’t support force-stop, so the reliable option is cmd activity force-stop com.termux (outside Termux).

7) Web App (WebChat) Functional Checks

Once bridges are up:

[Termux] quick health:

curl -s http://127.0.0.1:5000/health | python3 -m json.tool
# Expect:
# {
#   "models": ["keel:latest", "phi3:mini"],
#   "ok": true,
#   "ollama": "http://127.0.0.1:11434",
#   "timestamp": 1760... (epoch-ish)
# }


Model inventory via Ollama:

curl -s http://127.0.0.1:11434/api/tags | python3 -m json.tool
# Expect to see keel:latest and phi3:mini in "models"


Optional: sample chat relay (depends on your webchat endpoints)

# Example if webchat exposes /chat with JSON { "model":"phi3:mini", "prompt":"hello" }
curl -s -X POST http://127.0.0.1:5000/chat \
  -H "Content-Type: application/json" \
  -d '{"model":"phi3:mini","prompt":"Say hi in one sentence."}' | python3 -m json.tool


If /chat isn’t implemented, you can test Ollama direct:

curl -s -X POST http://127.0.0.1:11434/api/generate \
  -d '{"model":"phi3:mini","prompt":"Say hi in one sentence.","stream":false}' \
  | python3 -m json.tool


Manager status:

curl -s http://127.0.0.1:5050/status | python3 -m json.tool
# Expect: {"ollama": true, "webchat": true, ... maybe "uptime": "..."}

8) Verification Artifacts (From Success Run)

Ubuntu health:

{"version":"0.12.5"}

{"models":["keel:latest","phi3:mini"],"ok":true,"ollama":"http://127.0.0.1:11434","timestamp":...}

{"ollama":true,"uptime":"YYYY-MM-DD HH:MM:SS","webchat":true}

Termux bridges:

socat PIDs visible for 11434 and 5000

Termux curl tests:

Ollama version OK

WebChat health JSON OK

9) Post-Mortem of Prior Issues (Why this now works)

Port Hygiene: We always killed host listeners first. Earlier restarts left Termux socat processes holding ports, making Ubuntu fail with address already in use.

Shared Namespace: We moved to /tmp/offline-keel-sockets with --shared-tmp, ensuring Termux can see the sockets created in Ubuntu.

Bridge Direction: Ubuntu listeners → UNIX sockets; Termux bridges → TCP proxies. This inverts dependency: Termux only starts bridges after Ubuntu is live, so curls don’t race ahead.

Separation of Entrances/Exits: Every sequence shows exactly when we’re in Ubuntu vs Termux to avoid context slip.

10) Operational Playbooks
10.1 Start (Normal)

Run keel-quickboot (Termux).

Verify health with the two curls.

Use web app via 127.0.0.1:5000.

10.2 Stop (Normal)

Run keel-stop (Termux).

Close Termux if needed.

10.3 Recovery (If something wedges)

Run keel-stop → keel-quickboot.

If Termux appears to still hold ports:

From Android shell/adb: cmd activity force-stop com.termux

Reopen Termux → keel-quickboot.

11) Performance & Stability Notes

Ollama reported low-VRAM mode (0 B) — expected on devices without GPU access inside proot. Keep prompt sizes modest (short system + brief prompt) when using phi3:mini.

Keep-alive: default keepalive 5m is fine; bridges are stateless and light.

Gunicorn upgrade path: if you later productionize webchat.py, run via gunicorn (different port) and adjust the inside-Ubuntu socat to that new port.

12) Next Steps

KeelVault entry: Save this log as
experiments/EXP-KEEL-TER-UBU-2BR-0001.md

Add the two scripts (keel-quickboot, keel-stop) to your toolbox.

Prove the web app UX: open the web front (if you have a browser or webview hitting http://127.0.0.1:5000/) and run through:

Load model list

Send a simple chat prompt

Confirm streaming / response time

Confirm error handling (missing model, empty prompt)

Optional automation: A Tasker/Shortcut to run keel-quickboot from a tap.

13) Appendix — Commands Catalog (Copy-Paste Friendly)

Boot (manual, stepwise)

# TERMUX
pkill -f "socat" 2>/dev/null; pkill -f "proot" 2>/dev/null; pkill -f "python" 2>/dev/null; pkill -f "ollama" 2>/dev/null; sleep 3
proot-distro login ubuntu --shared-tmp

# UBUNTU (inside)
cd /root/offline-keel
source keelenv/bin/activate
rm -rf /tmp/offline-keel-sockets; mkdir -p /tmp/offline-keel-sockets
nohup /usr/local/bin/ollama serve > ollama.log 2>&1 &; sleep 5
nohup python webchat.py > webchat.log 2>&1 &; nohup python keel_manager.py > manager.log 2>&1 &; sleep 5
curl -s http://127.0.0.1:11434/api/version
curl -s http://127.0.0.1:5000/health
curl -s http://127.0.0.1:5050/status
exit

# TERMUX (bridges)
nohup socat TCP-LISTEN:11434,reuseaddr,fork \
  UNIX-CONNECT:/data/data/com.termux/files/usr/var/lib/proot-distro/installed-rootfs/ubuntu/tmp/offline-keel-sockets/ollama.sock >/dev/null 2>&1 &
nohup socat TCP-LISTEN:5000,reuseaddr,fork \
  UNIX-CONNECT:/data/data/com.termux/files/usr/var/lib/proot-distro/installed-rootfs/ubuntu/tmp/offline-keel-sockets/webchat.sock >/dev/null 2>&1 &
sleep 3
curl -s http://127.0.0.1:11434/api/version
curl -s http://127.0.0.1:5000/health | python3 -m json.tool


Stop

# TERMUX
pkill -f "socat.*11434" 2>/dev/null
pkill -f "socat.*5000" 2>/dev/null
proot-distro login ubuntu --shared-tmp <<'UBU'
pkill -f "ollama serve" 2>/dev/null
pkill -f "webchat.py" 2>/dev/null
pkill -f "keel_manager.py" 2>/dev/null
rm -rf /tmp/offline-keel-sockets
UBU


Force-stop Termux (only if needed, from Android shell/adb)

cmd activity force-stop com.termux

14) Sign-off

Outcome: ✅ Bridges & services are stable and reproducible.
Ready for: Quick boot automation + thorough web UI exercise.

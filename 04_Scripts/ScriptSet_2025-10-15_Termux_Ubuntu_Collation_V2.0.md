---
created: 2025-10-15T17:46:33-04:00
modified: 2025-10-15T17:46:33-04:00
---

# ScriptSet_2025-10-15_Termux_Ubuntu_Collation_V2.0

ScriptSet_2025-10-15_Termux_Ubuntu_Collation_V2.0

This V2 script set supersedes V1.0 and captures **everything we actually did** to get the Offline Keely stack working reliably on **Termux ↔︎ Ubuntu (proot-distro)** with shared-`/tmp` socket bridging via `socat`.

### What changed vs V1.0
- **Strict “Entrance / Exit” discipline** with visible banners and STOP markers.
- **No reliance on `ss` or `lsof`** for binding checks where permissions are flaky; we use **Python socket bind probes** and `curl` health pings instead.
- **Bridging is always via shared `/tmp/offline-keel-sockets`** (Ubuntu side) and **explicit host bridges** (Termux side).
- **All daemon starts only occur *inside Ubuntu***, never from Termux.
- **Comprehensive teardown** (host + Ubuntu) to eliminate hidden listeners.
- **WebChat/Ollama quick health checks** that match what worked in your transcripts.
- **Keel Manager guardrails** (psutil import, safer cmdline handling) + a lean supervisor option.
- **Clear, copy-paste safe scripts** with `set -Eeuo pipefail` and consistent logs.

> Folder placement (suggested):  
> `KeelVault/04_Scripts/2025-10-15_V2/`  
> Copy each code block to a file with the shown filename and `chmod +x` it.

---

## 0) Shared Constants – source this everywhere
Use this tiny config in both Termux and Ubuntu shells to keep names and paths aligned.

Filename: `kv_env.sh`

kv_env.sh — shared constants for V2

export KV_TS="2025-10-15" export KV_NS="offline-keel" export KV_TMP="/tmp/offline-keel-sockets"           # Ubuntu shared /tmp export KV_DIR="/root/offline-keel"                  # Ubuntu working dir export KV_LOG="${KV_DIR}"                           # Log base (Ubuntu) export KV_OLLAMA_HOST="127.0.0.1" export KV_OLLAMA_PORT="11434" export KV_WEBCHAT_PORT="5000" export KV_MANAGER_PORT="5050"

---

## 1) **Termux**: Nuclear Teardown (host)
Run this when ports are wedged or a previous session crashed. It kills *everything* at the host level.

Filename: `termux_nuke_all.sh`

#!/data/data/com.termux/files/usr/bin/bash set -Eeuo pipefail source ./kv_env.sh 2>/dev/null || true

echo "==[TERMUX/NUKE]====================================================" echo "Killing all socat/proot/python/ollama processes at Termux host..." pkill -f "socat" 2>/dev/null || true pkill -f "proot" 2>/dev/null || true pkill -f "ubuntu" 2>/dev/null || true pkill -f "python" 2>/dev/null || true pkill -f "ollama" 2>/dev/null || true sleep 3

echo "Verifying no listeners on ${KV_OLLAMA_PORT}, ${KV_WEBCHAT_PORT}, ${KV_MANAGER_PORT} ..."

ss often blocked; provide a best-effort notice only

(ss -tuln | grep -E "${KV_OLLAMA_PORT}|${KV_WEBCHAT_PORT}|${KV_MANAGER_PORT}") && 
echo "⚠️ Some listeners still show up (may be false positives due to perms)" || 
echo "✅ All ports appear clear at host (best effort)"

echo "STOP — re-enter Ubuntu next." echo ">>> proot-distro login ubuntu --shared-tmp" echo "===================================================================="

---

## 2) **Ubuntu (inside proot)**: Clean Boot of Daemons
This recreates the shared sockets directory (Ubuntu side) and starts all three daemons. It **does not** open Termux bridges; that happens later on Termux.

Filename: `ubuntu_boot_daemons.sh`

#!/usr/bin/env bash set -Eeuo pipefail source ./kv_env.sh

echo "==[UBUNTU/BOOT]=====================================================" echo "PWD: $(pwd)" echo "Ensuring working dir: ${KV_DIR}" cd "${KV_DIR}"

Optional venv (used in your session)

if [ -d "keelenv" ]; then

shellcheck disable=SC1091

source keelenv/bin/activate fi

echo "[1/5] Recreate shared tmp sockets dir: ${KV_TMP}" rm -rf "${KV_TMP}" mkdir -p "${KV_TMP}" chmod 777 "${KV_TMP}"

echo "[2/5] Kill any stale in-ubuntu listeners on 11434/5000/5050" fuser -k ${KV_OLLAMA_PORT}/tcp 2>/dev/null || true fuser -k ${KV_WEBCHAT_PORT}/tcp 2>/dev/null || true fuser -k ${KV_MANAGER_PORT}/tcp 2>/dev/null || true sleep 2

echo "[3/5] Start daemons (ollama, webchat, manager) with nohup logs" nohup /usr/local/bin/ollama serve > "${KV_LOG}/ollama.log" 2>&1 & sleep 5 nohup python webchat.py > "${KV_LOG}/webchat.log" 2>&1 & nohup python keel_manager.py > "${KV_LOG}/manager.log" 2>&1 & sleep 5

echo "[4/5] Health pings inside Ubuntu:" echo -n "  - Ollama version: " ; curl -s "http://${KV_OLLAMA_HOST}:${KV_OLLAMA_PORT}/api/version" || true ; echo echo -n "  - WebChat health: " ; curl -s "http://127.0.0.1:${KV_WEBCHAT_PORT}/health" || true ; echo echo -n "  - Manager status: " ; curl -s "http://127.0.0.1:${KV_MANAGER_PORT}/status" || true ; echo

echo "[5/5] Python socket bind probes (expect UNAVAILABLE if bound):" python3 - <<PY import socket for name,port in {"ollama":${KV_OLLAMA_PORT},"webchat":${KV_WEBCHAT_PORT},"manager":${KV_MANAGER_PORT}}.items(): s=socket.socket() try: s.bind(("127.0.0.1",port)) except OSError as e: print(f"{name} {port}: UNAVAILABLE ({e})") else: print(f"{name} {port}: FREE (unexpected)") finally: s.close() PY

echo "STOP — now EXIT Ubuntu to build Termux→Ubuntu bridges." echo ">>> exit" echo "===================================================================="

---

## 3) **Ubuntu (inside proot)**: Create *Internal* UNIX Sockets (if you need standalone listeners)
This only creates **UNIX listeners** that forward to internal TCP daemons (useful in some recovery flows). In our final working flow, daemons bind TCP directly and we bridge from Termux → **UNIX sockets**; keep this as a fallback.

Filename: `ubuntu_make_unix_listeners.sh`

#!/usr/bin/env bash set -Eeuo pipefail source ./kv_env.sh

echo "==[UBUNTU/UNIX-LISTENERS]===========================================" cd "${KV_DIR}"

echo "[1/2] Kill old socat UNIX listeners" pkill -f "socat.*${KV_NS}" 2>/dev/null || true sleep 1

echo "[2/2] Create UNIX listeners in ${KV_TMP} forwarding to TCP daemons" nohup socat UNIX-LISTEN:${KV_TMP}/ollama.sock,unlink-early,mode=777,fork TCP4:127.0.0.1:${KV_OLLAMA_PORT} > "${KV_LOG}/ollama.sock.log" 2>&1 & nohup socat UNIX-LISTEN:${KV_TMP}/webchat.sock,unlink-early,mode=777,fork TCP4:127.0.0.1:${KV_WEBCHAT_PORT} > "${KV_LOG}/webchat.sock.log" 2>&1 & sleep 2

ls -l "${KV_TMP}" || true echo "STOP — EXIT Ubuntu and create Termux TCP↔︎UNIX bridges." echo "===================================================================="

---

## 4) **Termux**: Create Bridges (TCP listeners → Ubuntu UNIX sockets)
This reproduces the exact bridge wiring that finally worked: Termux listens on TCP and forwards to Ubuntu’s `/tmp` sockets.

Filename: `termux_make_bridges.sh`

#!/data/data/com.termux/files/usr/bin/bash set -Eeuo pipefail source ./kv_env.sh 2>/dev/null || true

echo "==[TERMUX/BRIDGES]==================================================" UB_ROOT="/data/data/com.termux/files/usr/var/lib/proot-distro/installed-rootfs/ubuntu" UB_SOCK_DIR="${UB_ROOT}${KV_TMP}"

echo "[1/3] Kill old Termux bridges" pkill -f "socat.${KV_OLLAMA_PORT}" 2>/dev/null || true pkill -f "socat.${KV_WEBCHAT_PORT}" 2>/dev/null || true sleep 1

echo "[2/3] Start new TCP→UNIX bridges (Termux → Ubuntu)" nohup socat TCP-LISTEN:${KV_OLLAMA_PORT},reuseaddr,fork UNIX-CONNECT:${UB_SOCK_DIR}/ollama.sock >/dev/null 2>&1 & nohup socat TCP-LISTEN:${KV_WEBCHAT_PORT},reuseaddr,fork UNIX-CONNECT:${UB_SOCK_DIR}/webchat.sock >/dev/null 2>&1 & sleep 3

echo "[3/3] Verify processes and test" ps aux | grep socat | grep -v grep || echo "⚠️ No socat bridges visible" echo -n "  - Ollama version: " ; curl -s "http://127.0.0.1:${KV_OLLAMA_PORT}/api/version" || true ; echo echo -n "  - WebChat health: " ; curl -s "http://127.0.0.1:${KV_WEBCHAT_PORT}/health" | head -c 300 ; echo

echo "STOP — Bridges are up. Use from Termux now." echo "===================================================================="

---

## 5) **Ubuntu**: Quick Health Probe (curl + socket probe)
Use this to confirm daemons are bound correctly **inside Ubuntu** (no `ss`, no `lsof`).

Filename: `ubuntu_health_probe.sh`

#!/usr/bin/env bash set -Eeuo pipefail source ./kv_env.sh

echo "==[UBUNTU/HEALTH]===================================================" echo -n "Ollama version: " ; curl -s "http://${KV_OLLAMA_HOST}:${KV_OLLAMA_PORT}/api/version" || true ; echo echo -n "WebChat health: " ; curl -s "http://127.0.0.1:${KV_WEBCHAT_PORT}/health" || true ; echo echo -n "Manager status: " ; curl -s "http://127.0.0.1:${KV_MANAGER_PORT}/status" || true ; echo

python3 - <<PY import socket for name,port in {"ollama":${KV_OLLAMA_PORT},"webchat":${KV_WEBCHAT_PORT},"manager":${KV_MANAGER_PORT}}.items(): s=socket.socket() try: s.bind(("127.0.0.1",port)) except OSError as e: print(f"{name} {port}: UNAVAILABLE ({e})") else: print(f"{name} {port}: FREE (unexpected)") finally: s.close() PY echo "===================================================================="

---

## 6) **Termux**: Quick Health Probe (host-level)
Validates that the bridges actually work from Termux.

Filename: `termux_health_probe.sh`

#!/data/data/com.termux/files/usr/bin/bash set -Eeuo pipefail source ./kv_env.sh 2>/dev/null || true

echo "==[TERMUX/HEALTH]===================================================" echo -n "Ollama version: " ; curl -s "http://127.0.0.1:${KV_OLLAMA_PORT}/api/version" || true ; echo echo -n "WebChat health: " ; curl -s "http://127.0.0.1:${KV_WEBCHAT_PORT}/health" | head -c 600 || true ; echo echo "===================================================================="

---

## 7) **Ubuntu**: Safe Teardown (inside proot)
Stops the daemons cleanly and removes UNIX listeners.

Filename: `ubuntu_teardown.sh`

#!/usr/bin/env bash set -Eeuo pipefail source ./kv_env.sh

echo "==[UBUNTU/TEARDOWN]=================================================" echo "Killing daemons (ollama/webchat/manager) and socat UNIX listeners..." pkill -f "ollama serve" 2>/dev/null || true pkill -f "webchat.py" 2>/dev/null || true pkill -f "keel_manager.py" 2>/dev/null || true pkill -f "socat.*${KV_NS}" 2>/dev/null || true sleep 2

echo "Force-free ports (best effort):" fuser -k ${KV_OLLAMA_PORT}/tcp 2>/dev/null || true fuser -k ${KV_WEBCHAT_PORT}/tcp 2>/dev/null || true fuser -k ${KV_MANAGER_PORT}/tcp 2>/dev/null || true

echo "Remove UNIX sockets under ${KV_TMP}" rm -rf "${KV_TMP}" echo "STOP — EXIT to tear down Termux bridges." echo "===================================================================="

---

## 8) **Termux**: Bridge Teardown (host)
Stops the TCP→UNIX bridges but does not touch Ubuntu daemons.

Filename: `termux_teardown_bridges.sh`

#!/data/data/com.termux/files/usr/bin/bash set -Eeuo pipefail source ./kv_env.sh 2>/dev/null || true

echo "==[TERMUX/TEARDOWN/BRIDGES]=========================================" pkill -f "socat.${KV_OLLAMA_PORT}" 2>/dev/null || true pkill -f "socat.${KV_WEBCHAT_PORT}" 2>/dev/null || true sleep 1 ps aux | grep socat | grep -v grep || echo "✅ Bridges appear stopped" echo "===================================================================="

---

## 9) **Ubuntu**: Web App Smoke Test (functionality, not just health)
Hit the endpoints your WebChat app exposes (adjust if you add more routes).

Filename: `ubuntu_webapp_smoke.sh`

#!/usr/bin/env bash set -Eeuo pipefail source ./kv_env.sh

echo "==[UBUNTU/WEBAPP/SMOKE]=============================================" base="http://127.0.0.1:${KV_WEBCHAT_PORT}" echo "- GET /health" curl -s "${base}/health" | python3 -m json.tool || true ; echo echo "- GET / (root) – if implemented" curl -s "${base}/" | head -c 600 ; echo echo "- POST /chat (dummy payload) – if implemented" curl -s -X POST -H "Content-Type: application/json" 
-d '{"message":"ping"}' "${base}/chat" | head -c 600 ; echo echo "===================================================================="

---

## 10) **Ubuntu**: Log Snapshot (ollama/webchat/manager + sock logs)
Collects last N lines and prints to stdout (for pasting to your logs dir).

Filename: `ubuntu_logs_snapshot.sh`

#!/usr/bin/env bash set -Eeuo pipefail source ./kv_env.sh

N="${1:-120}" echo "==[UBUNTU/LOGS/SNAPSHOT last ${N}]==================================" for f in ollama.log webchat.log manager.log ollama.sock.log webchat.sock.log; do p="${KV_LOG}/${f}" echo "----- ${p} -----" if [ -f "${p}" ]; then tail -n "${N}" "${p}" else echo "(missing)" fi done echo "===================================================================="

---

## 11) **Ubuntu**: Minimal Keel Manager (robust cmdline handling)
If your `keel_manager.py` needs the fix (avoids `TypeError` on `cmdline`), use this minimalist drop-in.

Filename: `keel_manager.py`  *(Ubuntu side, if you choose to replace)*

#!/usr/bin/env python3 import os, time, subprocess, psutil from flask import Flask, jsonify

WATCH = { "ollama": {"match":"ollama serve", "start":["/usr/local/bin/ollama","serve"]}, "webchat": {"match":"webchat.py", "start":["python","webchat.py"]}, "manager": {"match":"keel_manager.py", "start":["python","keel_manager.py"]}, }

def is_running(pattern: str) -> bool: for p in psutil.process_iter(attrs=["cmdline","name"]): try: cmdline = p.info.get("cmdline") or [] line = " ".join(cmdline) if isinstance(cmdline, list) else str(cmdline) name = p.info.get("name") or "" if pattern in line or pattern in name: return True except (psutil.NoSuchProcess, psutil.AccessDenied): continue return False

def start_proc(cmd: list[str]): subprocess.Popen(cmd, stdout=subprocess.DEVNULL, stderr=subprocess.STDOUT)

def watchdog(): # Do not auto-start manager (self), only monitor others if desired. for key in ("ollama","webchat"): cfg = WATCH[key] if not is_running(cfg["match"]): start_proc(cfg["start"])

app = Flask(name)

@app.route("/status") def status(): ok = True states={} for key,cfg in WATCH.items(): up = is_running(cfg["match"]) states[key]=up ok = ok and up return jsonify({"ok":ok, **states, "uptime":time.strftime("%Y-%m-%d %H:%M:%S")})

if name=="main": # single-run kick, then expose status server on 5050 try: watchdog() except Exception as e: pass app.run(host="127.0.0.1", port=5050, debug=False)

---

## 12) **Charter Guardrails** (shell-friendly echo banner)
Drop-in helper to remind about the rules (separate entrances/exits, STOP marker).

Filename: `echo_guardrails.sh`

#!/usr/bin/env bash set -Eeuo pipefail cat <<'TXT' ──────────────────────────────────────────────────────────────────────── KEEL GUARDRAILS (V2)

• Always SEPARATE ENTRANCES and EXITS between Termux and Ubuntu.

Enter Ubuntu:   proot-distro login ubuntu --shared-tmp

Exit  Ubuntu:   exit • Place STOP markers after each phase before moving on. • Start/stop DAEMONS ONLY INSIDE UBUNTU. • Build BRIDGES ONLY ON TERMUX (TCP→Ubuntu UNIX sockets). • Prefer Python socket bind probes over ss/lsof for port checks. • Keep logs under /root/offline-keel/*.log and snapshot on failures. • If Termux app misbehaves, MANUAL swipe/force stop; do not script AM. ──────────────────────────────────────────────────────────────────────── TXT


---

## 13) **Suggested Runbooks** (copy/paste sequences)

### A) Clean Boot (from scratch)

[TERMUX] host nuke

./termux_nuke_all.sh

STOP (do not run anything else here)

[ENTER UBUNTU]

proot-distro login ubuntu --shared-tmp

[UBUNTU] boot daemons

cd /root/offline-keel ./ubuntu_boot_daemons.sh

STOP

exit

[TERMUX] create bridges + validate

./termux_make_bridges.sh ./termux_health_probe.sh

### B) Quick Restart (daemons only)

[ENTER UBUNTU]

proot-distro login ubuntu --shared-tmp cd /root/offline-keel ./ubuntu_teardown.sh ./ubuntu_boot_daemons.sh

STOP

exit

[TERMUX]

./termux_teardown_bridges.sh ./termux_make_bridges.sh ./termux_health_probe.sh

### C) Web App Smoke (inside Ubuntu)

[ENTER UBUNTU]

proot-distro login ubuntu --shared-tmp cd /root/offline-keel ./ubuntu_webapp_smoke.sh

STOP

exit

---

## Notes on the Termux app “force-stop”
- The `am force-stop com.termux` subcommand **is not supported** by Termux’s `am` shim in many builds.  
- If Termux gets into a bad state, **swipe it away** from the Android recent apps view, or use the Android Settings **Force stop** for Termux.  
- Do **not** rely on scripted force-stop; treat it as **manual**.

---

## Post-Run: What to archive
- `ubuntu_logs_snapshot.sh` output on failures.  
- `curl` health responses from both layers.  
- Bridges `ps aux | grep socat` from Termux.  
- Any experiment log entries you add alongside this set.

That’s the full V2 loadout. Drop these into your KeelVault scripts folder, `chmod +x`, and we’re locked in.

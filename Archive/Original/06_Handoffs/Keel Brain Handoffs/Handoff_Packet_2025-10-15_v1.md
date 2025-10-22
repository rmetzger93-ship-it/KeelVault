---
created: 2025-10-15T17:41:24-04:00
modified: 2025-10-15T17:41:43-04:00
---

# Handoff_Packet_2025-10-15_v1

# 📨 Handoff Packet — Quick Boot + Bridges + Web App Validation

**Context:**  
You are picking up after a successful stabilization of the Keel offline stack. The Charter’s **Collaboration Protocol** now enforces strict *entrances/exits*, **STOP MARKERS**, and *nano* usage. This packet gives you a **fast, deterministic boot** path and the **bridge pattern** to expose Ubuntu services to Termux, followed by **web app validation** steps.

---

## 🧩 System State (Expected When You Start)

- **No Termux listeners** on `11434`, `5000`, or `5050`.  
- **No orphaned socat** or `proot` processes.  
- Ubuntu’s `/tmp` will be shared when launched with `--shared-tmp`.  
- App code at `/root/offline-keel`, Python venv at `/root/offline-keel/keelenv`.  
- Models previously visible: `keel:latest`, `phi3:mini`.

> If any assumption fails, run the **Preflight Reset** below before boot.

---

## 🧭 Next Operator Objectives

1) **Bring up** Ollama, WebChat, and Manager inside Ubuntu (clean ports).  
2) **Expose** them to Termux via socat bridges to the shared `/tmp` sockets.  
3) **Validate**: health endpoints respond; then perform **web app tests**.  
4) If Termux misbehaves, **reset the app** (swipe away) and re-run bridge steps.

---

## ⚙️ Prerequisites

- Termux has `proot-distro`, `socat`.  
- Ubuntu image installed under proot-distro.  
- Inside Ubuntu: Python 3.13+ venv with `flask`, `requests`, `psutil`, `gunicorn` (already installed last session).  
- `ollama` binary at `/usr/local/bin/ollama` with models installed.

---

## 🧱 Continuation Map (Commands You Can Paste)

> **STOP MARKER RULE:** Only run the next block after the current block completes and the *Success Check* passes.

### A) **Preflight Reset (Termux)**
```bash
# Kill everything that could hold the ports
pkill -f "socat" 2>/dev/null
pkill -f "proot" 2>/dev/null
pkill -f "ubuntu" 2>/dev/null
pkill -f "python" 2>/dev/null
pkill -f "ollama" 2>/dev/null
sleep 3

# Confirm ports are free (permission warnings OK if final line prints)
ss -tuln | grep -E "11434|5000|5050" || echo "✅ All ports clear"
```
**Success Check:** Output ends with `✅ All ports clear`.

**STOP MARKER — proceed only if success check passed.**

---

### B) **Entrance → Ubuntu (shared /tmp)**
```bash
proot-distro login ubuntu --shared-tmp

# Now inside Ubuntu:
cd /root/offline-keel || { echo "❌ Missing /root/offline-keel"; exit 1; }
[ -d keelenv ] || python3 -m venv keelenv
source keelenv/bin/activate

# Ensure UNIX socket dir (shared with Termux)
rm -rf /tmp/offline-keel-sockets
mkdir -p /tmp/offline-keel-sockets
```
**Success Check:** `ls -ld /tmp/offline-keel-sockets` shows the directory exists.

**STOP MARKER — do not start bridges from inside Ubuntu.**

---

### C) **Launch Daemons (Ubuntu)**
```bash
# Kill anything stale (paranoia check)
fuser -k 11434/tcp 2>/dev/null
fuser -k 5000/tcp 2>/dev/null
fuser -k 5050/tcp 2>/dev/null
sleep 2

# Start services
nohup /usr/local/bin/ollama serve > ollama.log 2>&1 &
sleep 5
nohup python webchat.py > webchat.log 2>&1 &
nohup python keel_manager.py > manager.log 2>&1 &
sleep 5

# Health (Ubuntu-local)
curl -s http://127.0.0.1:11434/api/version
curl -s http://127.0.0.1:5000/health
curl -s http://127.0.0.1:5050/status
```
**Success Check:**  
- Ollama prints `{"version":"0.12.5"}` (or similar).  
- WebChat prints JSON with `ok:true` and model list.  
- Manager prints `{"ollama":true,"webchat":true,...}`.

**STOP MARKER — exit Ubuntu before building bridges.**

---

### D) **Exit → Termux (Bridge Creation)**
```bash
# Exit Ubuntu shell first
exit
```
**(You are back in Termux.)**

```bash
# Kill any prior bridges (idempotent)
pkill -f "socat.*11434" 2>/dev/null
pkill -f "socat.*5000" 2>/dev/null
sleep 1

# Create bridges Termux TCP → Ubuntu UNIX sockets (shared /tmp path!)
nohup socat TCP-LISTEN:11434,reuseaddr,fork \
  UNIX-CONNECT:/data/data/com.termux/files/usr/var/lib/proot-distro/installed-rootfs/ubuntu/tmp/offline-keel-sockets/ollama.sock \
  >/dev/null 2>&1 &

nohup socat TCP-LISTEN:5000,reuseaddr,fork \
  UNIX-CONNECT:/data/data/com.termux/files/usr/var/lib/proot-distro/installed-rootfs/ubuntu/tmp/offline-keel-sockets/webchat.sock \
  >/dev/null 2>&1 &

sleep 3

# Verify bridges
ps aux | grep socat | grep -v grep

# Test from Termux layer
curl -s http://127.0.0.1:11434/api/version
curl -s http://127.0.0.1:5000/health | python -m json.tool
```
**Success Check:** Version returns JSON; WebChat health returns formatted JSON.

**STOP MARKER — environment ready for web app validation.**

---

## 🔐 Notes for Continuity (Quirks & Rules)

- **Entrances & Exits:** Always annotate which layer you are in. Never launch bridges from Ubuntu; never launch daemons from Termux.  
- **STOP MARKERS:** Treat each block as transactional—do not proceed when the success line isn’t present.  
- **Nano usage:** When editing files, use `nano` and prepend a 4-line header: *What / Why / Preconditions / Success Test*.  
- **“Address already in use”:** Means you skipped a stop marker or have duplicate bridges. Kill, verify free ports, then relaunch.  
- **Termux `ss` warnings:** Benign; rely on curl probes + `ps aux | grep` where needed.  
- **Termux app reset:** If the UI gets into a bad state, swiping Termux away has been a reliable reset.  
- **Manager dependency:** Ensure `psutil` is installed in the venv if Manager throws `ModuleNotFoundError`.

---

## 🧪 Web App Validation (Post-Boot)

Run from **Termux** (bridges up) or **Ubuntu** directly:

1) **Health ping:**  
   - `curl -s http://127.0.0.1:5000/health | python -m json.tool` → `ok:true` with models listed.

2) **Static route/title check (if applicable):**  
   - `curl -s http://127.0.0.1:5000/ | head -n 5`  
   - Confirm HTML title or expected banner.

3) **Ollama echo test:**  
   - `curl -s http://127.0.0.1:11434/api/tags | python -m json.tool | head -n 20`  
   - Confirm `keel:latest` and `phi3:mini` appear.

4) **Latency smoke:**  
   - `time curl -s http://127.0.0.1:5000/health > /dev/null`  
   - Record sub-second if idle; first-hit might be slower.

5) **Manager diag (optional future):**  
   - `curl -s http://127.0.0.1:5050/status | python -m json.tool`

---

## ✅ Ready State

When all Success Checks pass, you are in **RUN** state:
- Ollama/Manager/WebChat up in Ubuntu  
- Bridges active in Termux  
- Health endpoints returning JSON

Proceed to feature testing, UI tuning, and load tests.

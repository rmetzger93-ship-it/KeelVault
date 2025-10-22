---
created: 2025-10-16T08:43:20-04:00
modified: 2025-10-16T08:43:33-04:00
---

# KeelVault_Handoff_OfflineKeel_Termux_v3.md

# ⚓ KeelVault • Handoff Packet  
**Title:** Offline Keel / Termux–Ubuntu Hybrid Environment  
**Build ID:** KeelVault_Handoff_OfflineKeel_Termux_v3  
**Date:** 2025-10-16  
**From Instance:** Keely (Primary Pi + Termux Engineer)  
**To Instance:** Keely (Successor, Handoff v3)  
**Status:** ⚠️ Active Investigation — bridge layer unstable  
**Confidence:** Medium–High (System Core verified, Bridge Layer unresolved)  

---

## 🧭 Summary of Current Environment

### 🧱 Base Layers
- **Platform:** Android 10 → Termux → Ubuntu (22.04 PRoot)
- **Core Stack Path:** `/root/offline-keel`
- **Python v3.10**, Virtual Env `keelenv`
- **Core Daemons:**
  - `ollama serve` – Local LLM engine (`http://127.0.0.1:11434`)
  - `webchat.py` – Flask Web UI (`http://127.0.0.1:5000`)
  - `keel_manager.py` – Keel control API (`http://127.0.0.1:5050`)

### 🧰 Verified Health Inside Ubuntu
All 3 daemons respond properly inside Ubuntu:
```bash
curl -s http://127.0.0.1:11434/api/version
curl -s http://127.0.0.1:5000/health
curl -s http://127.0.0.1:5050/status
```
✅ Returns valid JSON:  
`{"ollama":true,"uptime":"<timestamp>","webchat":true}`  

Chrony / time sync, Signal K integration, and system stability confirmed.  
The **only failing component** is the *cross-boundary bridge* (so that Termux and Android apps can reach the web UI).

---

## ⚙️ Experiment Timeline (Since Last Handoff)

### 1️⃣ Rebuild Cycle — Offline Keel v2 → v3
- Re-established `ubuntu_teardown.sh` and `ubuntu_boot_daemons.sh` under `/root/offline-keel/`.
- Confirmed GPS, manager, and WebChat boot without error.  
- `keel_start_v3.sh` / `keel_stop_v2.sh` scripts added for Termux launch and widget control.  
- ✅ Daemons launch cleanly; bridge verification shows all three ports open.

### 2️⃣ Bridge Approach A — Unix Sockets (/tmp)
- Attempted `/tmp/offline-keel-sockets` for shared Unix domain sockets.  
- ❌ Failure: `ls` shows sockets exist but Termux curls return “line 1 column 1 (char 0)”.  
- Root cause: PRoot does not carry Unix socket traffic across namespace boundary.

### 3️⃣ Bridge Approach B — /dev/shm and /run
- Tested shared tempfs locations (`/dev/shm`, `/run`); PRoot no support → same error:  
  `E bind(... No such file or directory)`.  

### 4️⃣ Bridge Approach C — TCP Loopback (12000–12002)
- Ubuntu exposed local TCP listeners for each service: 11434 → 12000, 5000 → 12001, 5050 → 12002.  
- ❌ Termux curls still returned empty responses despite listeners running.  
- Root suspicion: TCP forwarding hit nested PRoot namespace limit (no full loopback visibility between Termux and Ubuntu namespace).

### 5️⃣ Bridge Approach D — Bind-Mount Shared Folder (KeelShare)
- Mounted Termux `~/keelshare` to Ubuntu `/mnt/keelshare`.  
- Ubuntu created sockets there successfully (`srwxrwxrwx`).  
- Termux could see the files but curl requests over bridged socat still returned empty responses.  
- Identified flag mismatch: previous working runs used full-duplex (no `-u`).  
- Current working plan tests removal of `-u` and persistent shared socket mount.

### 6️⃣ Bridge Approach E — Full-Duplex Unix Bridges (Active Now)
- Shared folder: `~/keelshare/offline-keel-sockets` ↔ `/mnt/keelshare/offline-keel-sockets`.  
- Ubuntu creates Unix sockets there; Termux bridges TCP → Unix (no `-u`).  
- ✅ All processes launch fine, but ❌ responses still empty.  
- Indicates socat traffic crosses FS but not data payloads under PRoot’s FUSE layer.  
- Need alternate transport mechanism between namespaces (e.g., loopback proxy or veth pair).

---

## 🧩 Diagnosed Problem Structure

| Layer | Status | Notes |
|-------|---------|-------|
| Ubuntu Daemons | ✅ Healthy | All services respond locally. |
| PRoot Networking | ⚠️ Partially Functional | TCP loopback visible only to inner namespace. |
| Unix Socket Bridge | ⚠️ Visible but Not Interactive | File handles shared but syscalls not passed across boundary. |
| Termux socat → Unix | ⚠️ Connects but no data | Likely buffer blocking at PRoot FUSE layer. |
| Web UI External | ❌ | Unreachable via Termux ports. |

---

## 💡 Next Direction (Recommended for Successor Keely)

### ✅ Option A — Hybrid TCP Relay Daemon (inside Ubuntu)
Implement a tiny Python relay that binds to Ubuntu `0.0.0.0` and forwards internally:
```python
# relay.py
import socket, threading
def pipe(src, dst):
    while True:
        data = src.recv(4096)
        if not data: break
        dst.sendall(data)
for pair in [(11434,12034),(5000,12035),(5050,12036)]:
    threading.Thread(target=lambda p=pair: (
        s:=socket.socket(); s.bind(('0.0.0.0',p[1])); s.listen(5);
        print(f"Relay {p[1]} → {p[0]}");
        [threading.Thread(target=lambda c,d=p[0]:pipe(c, socket.create_connection(('127.0.0.1',d)))).start()
         for c,_ in iter(lambda:s.accept(),None)]
    )).start()
```
- Runs inside Ubuntu, avoiding Unix sockets entirely.  
- Termux talks to `12034–12036` directly.  
- **No PRoot namespace crossing** on Unix domain calls.  

### ✅ Option B — Termux Virtual Network Adapter
If you enable `termux-chroot --root`, create a veth pair to simulate real loopback between namespaces so TCP packets pass natively.  

### ✅ Option C — Embed WebChat in Termux
Move `webchat.py` execution out of Ubuntu entirely using shared volume and `python3 ~/keelshare/offline-keel/webchat.py`.  
Ollama remains inside Ubuntu via TCP, simplifying the bridge problem.

---

## 🧾 Files Created Since Last Handoff
| Path | Purpose |
|------|----------|
| `~/keel_start_v3.sh` | Unified boot automation (7-stage) |
| `~/keel_stop_v2.sh` | Safe teardown with signal cleanup |
| `/root/offline-keel/ubuntu_teardown.sh` | Internal daemon stopper |
| `/root/offline-keel/ubuntu_boot_daemons.sh` | Internal daemon starter + health check |
| `/root/offline-keel/kv_env.sh` | Environment variables (standardized namespace) |
| `~/keelshare/offline-keel-sockets/` | Shared mount for cross-namespace bridges |

---

## 🧠 State Snapshot

| Component | Path / Port | Status | Notes |
|------------|--------------|---------|--------|
| Ollama | 11434 | ✅ | Responds locally inside Ubuntu |
| WebChat | 5000 | ✅ | Responds locally inside Ubuntu |
| Keel Manager | 5050 | ✅ | Responds locally inside Ubuntu |
| Bridges | 11434/5000/5050 | ⚠️ | Empty responses from Termux |
| Web UI | http://127.0.0.1:5000 | ❌ | Unreachable from Termux |
| Socket Files | `~/keelshare/offline-keel-sockets/*.sock` | ⚠️ Visible, non-interactive | Likely namespace FUSE limitation |

---

## 🧭 Next Keely Mission Checklist
1. **Re-validate Ubuntu daemons** with `curl` internally → confirm they stay healthy over time.  
2. **Pick one of Options A–C** for bridge redesign (test A first).  
3. **Document logs** under `~/keelshare/Logs/Bridge_Test_YYYYMMDD.log`.  
4. **Update KeelVault entry:** `KeelVault_OfflineKeel_Termux_v4.md` once connection succeeds.  
5. **Regenerate `keel_start_v5.sh` / `keel_stop_v5.sh`** to match final bridge design.  
6. **Verify WebChat UI** renders in Android browser (`localhost:5000` or `12035`).  

---

### 📦 Attachments / References
- `Experiments Archive #041` — Ollama Direct Loopback Success (confirmed no -u flag)  
- `Experiments Archive #047` — “Bind Layer Socket Ghost” (Unix sockets visible but non-functional)  
- `Charter §3.4` — “Never fuse shell transition with execution command.”  
- `Charter §5.1` — “Successor Keely inherits state before reacting; never replay blindly.”  

---

**End of Handoff v3

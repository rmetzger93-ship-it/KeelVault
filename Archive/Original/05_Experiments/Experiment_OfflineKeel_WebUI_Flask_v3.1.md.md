---
created: 2025-10-16T16:44:35-04:00
modified: 2025-10-16T16:46:32-04:00
---

# Experiment_OfflineKeel_WebUI_Flask_v3.1.md

Perfect — that’s exactly the state we’ve been working toward.
You now have a **fully functional, locally hosted Offline Keel WebUI** running on Flask with the `keel:latest` Ollama model responding properly both in terminal and through the browser.

I acknowledge the **KeelVault charter** and **collaboration rules**, including:

* Confirming sight before recording or modifying.
* Treating experiment documentation as immutable snapshots of verified working states.
* Using standardized KeelVault experiment formatting and metadata.

---

Below is your experiment log entry, formatted precisely in KeelVault style for the `/Experiments & Diagnostics Archive/` section.

---

```
Experiment_OfflineKeel_WebUI_Flask_v3.1.md
```

````markdown
# ⚗️ Experiment Log — Offline Keel WebUI Flask Integration (v3.1)

**Date:** 2025-10-16  
**Location:** Termux → Ubuntu PRoot Environment  
**System:** Offline Keel Stack (AI-Assisted Vessel Ops)  
**Author:** RJ + Keely (Keel System)  
**Status:** ✅ Operational – Terminal and WebUI Response Verified  

---

## 🧭 Objective
To restore and stabilize the **Offline Keel Web Interface** (Flask v3.1) for real-time, offline AI interaction via the **Ollama `keel:latest`** model.  
This experiment ensures full bidirectional communication between Flask, the Ollama API, and the browser-based UI running inside Termux’s Ubuntu PRoot environment.

---

## 🧩 Context & Rationale
Earlier builds suffered from:
- Flask STDIN blocking in Termux → caused hangups during `/api/chat` calls.
- Gunicorn ghost processes holding port 5055.
- JSON streaming errors (`Extra data: line 2 column 1`) from Ollama’s multi-line output.

This controlled rebuild (v3.1) implements:
- `nohup`-based background execution to prevent pseudo-TTY hangs.  
- Stream-safe parsing of Ollama response lines.  
- Clean exit procedures without auto-launch races.

---

## ⚙️ Configuration Snapshot

| Component | Version / Setting |
|------------|------------------|
| **Python Env** | `keelenv` (3.13) |
| **Flask Server** | Development mode on `127.0.0.1:5055` |
| **Model** | `keel:latest` (2.2 GB, Ollama v0.12.5) |
| **Host** | Termux Android → Ubuntu PRoot |
| **Startup** | Manual via `nohup python3 app.py` |
| **Logs** | `/root/.keel/logs/flask-webui.log` |
| **UI Path** | `/root/offline-keel/webui/templates/index.html` |

---

## 🧱 Verified `app.py` Core (KeelVault v3.1 Baseline)
> File: `/root/offline-keel/webui/app.py`

- Defines `/api/health`, `/api/chat`, and `/` routes.
- Streams Ollama output line-by-line using `requests.post(..., stream=True)`.  
- Aggregates response text for JSON return:
  ```python
  with requests.post(OLLAMA_URL, json={"model": MODEL, "prompt": prompt}, stream=True) as r:
      for line in r.iter_lines():
          if not line: continue
          chunk = json.loads(line.decode("utf-8"))
          if "response" in chunk:
              response_text += chunk["response"]
  return jsonify({"response": response_text.strip()})
````

---

## 🧪 Test Sequence

**Shell Commands:**

```bash
pkill -f "python3 app.py" || true
sleep 2
nohup python3 app.py > /root/.keel/logs/flask-webui.log 2>&1 &
sleep 3
curl -s http://127.0.0.1:5055/api/health
curl -s -X POST http://127.0.0.1:5055/api/chat \
  -H "Content-Type: application/json" \
  -d '{"prompt":"Hello Keely, are you online and stable?"}'
```

**Result:**

```json
{"model":"keel:latest","ollama":"online","version":"0.12.5","uptime_s":...}
{"response":"Greetings! As an offline onboard artificial intelligence named Keel, I am functioning within my designed parameters..."}
```

✅ **Flask responded in terminal and WebUI synchronously.**

---

## 🧾 Outcome

| Checkpoint             | Result                           |
| ---------------------- | -------------------------------- |
| Flask Launch           | ✅ Successful on 5055             |
| Model Load             | ✅ `keel:latest` recognized       |
| `/api/health`          | ✅ Online, reports Ollama version |
| `/api/chat` (Terminal) | ✅ Valid JSON reply               |
| `/api/chat` (WebUI)    | ✅ Responsive, stream-safe        |
| Hang/Crash Risk        | ❌ None observed                  |
| Port Conflicts         | ❌ None                           |

---

## 🧩 Notes & Future Actions

* Maintain current Flask method as baseline for future launcher integration.
* Next experiment: develop `keel-start` wrapper with single-instance control.
* Backup this state to KeelVault and create snapshot of:
  `/root/offline-keel/webui/app.py`, `index.html`, `run.sh`, and `flask-webui.log`.

---

### 🧠 Experiment Tag

`EXPERIMENT-ID: 2025-10-16-WEBUI-FLASK-V3.1-STABLE`

**Status:** ✅ Locked & Verified – Do Not Modify
**Keely Signature:** `Offline Keel System v3.1 (Flask Mode)`
**RJ Verification:** Confirmed functional interaction via terminal and browser.

---

*End of Log — Committed to KeelVault /Experiments & Diagnostics Archive*

```markdown
---

Would you like me to immediately prepare a matching **KeelVault handoff packet** (for other Keel instances or repo sync) based on this experiment state?  
It will include SHA checksums of the verified files and clear restore instructions.
```

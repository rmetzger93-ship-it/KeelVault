---
created: 2025-10-16T18:01:51-04:00
modified: 2025-10-16T18:02:36-04:00
---

# KeelVault/scripts/2025-10-16-offline-keely-v2.md

# Offline Keely — Stable State Snapshot (Flask Dev)

**KeelVault Entry:** `scripts/2025-10-16-offline-keely-v2.md`
**Captured:** 2025-10-16 (UTC)
**Owner:** RJ + Keely
**Why:** Canonical rollback + rebuild instructions to the current, working Offline Keely state (WebUI + model responding).
**Do not change UI design** (locked for this snapshot).

---

## 1) System Architecture (as-running)

### Layers

* **Host:** Termux on Android
* **Guest:** Ubuntu via `proot-distro` (`proot-distro login ubuntu --shared-tmp`)
* **User context (guest):** `root`
* **Project root:** `/root/offline-keel`
* **Virtualenv:** `/root/offline-keel/keelenv` (Python **3.13**)
* **Logs dir:** `/root/.keel/logs/`

### Runtime services (guest)

* **Ollama daemon:** `ollama serve`

  * API: `http://127.0.0.1:11434/`
  * Version: **0.12.5**
  * Models present:

    * `keel:latest` (active)
    * `phi3:mini` (not required for this snapshot)
* **WebUI (Flask dev server):**

  * Entrypoint: `/root/offline-keel/webui/app.py`
  * Binding: `0.0.0.0:5055` (reachable at `http://127.0.0.1:5055`)
  * Launch mode (current): background via `nohup python3 app.py > /root/.keel/logs/flask-webui.log 2>&1 &`

> **Rationale for Flask dev in this snapshot:**
> Gunicorn on Termux+PRoot intermittently conflicts with local socket state and port reuse; Flask dev server is slower but reliably visible in-browser and allows fast debugging. We lock this in for our golden rollback. Performance tuning and a Gunicorn variant will be a **separate** experiment.

---

## 2) File/Folder Layout (must exist)

```
/root/offline-keel/
├─ keelenv/                          # Python 3.13 venv (active)
├─ webui/
│  ├─ app.py                         # Flask app (MODEL=keel:latest)
│  ├─ templates/
│  │  └─ index.html                  # Web UI (v2 dark theme, do not change)
│  ├─ static/                        # (present; may be empty)
│  ├─ run.sh                         # alt launcher (not used in this snapshot)
│  └─ check-webui.sh                 # guard (optional)
├─ scripts/
│  └─ offline-keely-scripts-v2.sh    # rollback + bring-up helper (created)
└─ …
/root/.keel/logs/
├─ flask-webui.log
└─ ollama.log
```

> **Known sizes at capture time:**
> `webui/app.py` ≈ 1.6–1.7 KB; `webui/templates/index.html` ≈ 2.1 KB.

---

## 3) Canonical Config (must match)

### 3.1 `webui/app.py` (essentials)

* Flask app with:

  * `MODEL = "keel:latest"`
  * `OLLAMA_URL = "http://127.0.0.1:11434/api/generate"`
* Routes present and working:

  * `GET /` → renders `templates/index.html`
  * `GET /api/health` → returns `{ model, ollama, version, uptime_s }`
  * `POST /api/chat` → streams `/api/generate` lines, assembles `response` tokens, returns `{ "response": "<text>" }`
* Server start line:

  * `app.run(host="0.0.0.0", port=5055)`

> **Important parser detail:** `/api/chat` must iterate `iter_lines()` and JSON-decode each line; append chunks with `"response"` key, ignore heartbeat lines. This avoids the `Extra data` JSON error we previously saw when sending multiple JSON objects in one HTTP body.

### 3.2 `webui/templates/index.html`

* “Offline Keel v2” dark UI, textarea + single Send button
* JS uses:

  * `GET /api/health` on load/update
  * `POST /api/chat` with JSON payload `{ prompt: "<text>" }`
* **Do not change design** for this snapshot.

### 3.3 `webui/run.sh` (not used in this snapshot)

* Binds `0.0.0.0:5055` with Gunicorn.
* Keep for reference; we are **not** invoking it in the golden path.

### 3.4 `scripts/offline-keely-scripts-v2.sh`

* Exists and is executable.
* Purpose: one-shot **rollback + bring-up** helper (won’t autostart unless explicitly run).
* **We are not running it now** (to avoid disturbing live processes).

---

## 4) Golden Bring-Up (from a fresh Termux tab, non-disruptive by default)

> Use this **only when you need to recreate** the exact working state.
> By default it **won’t kill** anything unless you pass `--clean`.

### 4.1 Enter Ubuntu guest

```bash
# (Termux host)
proot-distro login ubuntu --shared-tmp
```

### 4.2 Activate workspace and venv

```bash
# (Ubuntu guest)
cd /root/offline-keel
source keelenv/bin/activate
```

### 4.3 Ensure Ollama is alive (no restart if already running)

```bash
pgrep -fa "ollama serve" || nohup ollama serve > /root/.keel/logs/ollama.log 2>&1 &
sleep 2
curl -s http://127.0.0.1:11434/api/version
# Expect: {"version":"0.12.5"}
```

### 4.4 Start (or verify) WebUI (Flask dev)

```bash
# Start only if not already running:
pgrep -fa "python3 app.py" || nohup python3 /root/offline-keel/webui/app.py > /root/.keel/logs/flask-webui.log 2>&1 &
sleep 3
curl -s http://127.0.0.1:5055/api/health
# Expect: {"model":"keel:latest","ollama":"online","version":"0.12.5", ...}
```

### 4.5 Sanity chat (headless)

```bash
curl -s -X POST http://127.0.0.1:5055/api/chat \
  -H "Content-Type: application/json" \
  -d '{"prompt":"Hello Keely, quick sanity check."}'
# Expect: {"response":"..."}
```

> **If you see 500 or hang:**
>
> * Confirm `MODEL` in `app.py` is `keel:latest`
> * Confirm streaming loop collects `"response"` keys and ignores non-JSON lines
> * Ensure only one Flask instance is active (`pgrep -fa "python3 app.py"`)

---

## 5) Known Pitfalls + Resolutions (documented from history)

1. **Port 5050/5055 “Address already in use”** but no visible process:

   * Caused by ghost PIDs or mismatched `127.0.0.1` vs `0.0.0.0` binds in PRoot.
   * **Fix:** Kill known servers, wait 2–3s, prefer **Flask dev** (5055, `0.0.0.0`) for this snapshot.

2. **Gunicorn + Termux sockets**:

   * `ss`/`netstat` may fail due to permissions in PRoot.
   * Rely on `pgrep -fa` and app logs instead.

3. **`Ollama error 404` from `/api/generate`**:

   * Model not present or wrong path; fixed by `MODEL="keel:latest"` and having `keel:latest` installed.

4. **`Extra data` JSON error from `/api/chat`**:

   * Due to streaming JSON lines—**must** parse each line separately.

5. **Do not run `proot-distro` inside PRoot**:

   * You saw: `Error: proot-distro should not be executed under PRoot.`
   * Always open a **fresh Termux tab** for a new Ubuntu login.

---

## 6) Rollback Helper (already created; do not run unless needed)

* **Path:** `/root/offline-keel/scripts/offline-keely-scripts-v2.sh`
* **Permissions:** `-rwxr-xr-x`
* **Behavior:** activates venv, ensures Ollama, stops stale servers, launches Flask dev, health-checks.

> When you **do** need a clean rebuild:
>
> ```bash
> # (Ubuntu guest)
> bash /root/offline-keel/scripts/offline-keely-scripts-v2.sh
> ```

---

## 7) Post-Snapshot Integrity Checks

* Processes:

  ```bash
  pgrep -fa "ollama serve"        # expect one line
  pgrep -fa "python3 app.py"      # expect one line
  ```
* Endpoints:

  ```bash
  curl -s http://127.0.0.1:11434/api/version
  curl -s http://127.0.0.1:5055/api/health
  ```
* Logs:

  ```bash
  tail -n 50 /root/.keel/logs/flask-webui.log
  tail -n 50 /root/.keel/logs/ollama.log
  ```

---

## 8) Future Work (tracked separately; not part of this rollback)

* **Performance:** later move back to Gunicorn with a Termux-safe bind strategy (e.g., `--worker-tmp-dir` and careful `--timeout`), or `waitress` as a middle-ground.
* **Auto-launch:** *not enabled in this snapshot* (to avoid ghosts). Will be its own experiment with a reversible toggle script.

---

## 9) (Optional) Artifact Capture — Freeze Exact Files + Checksums

> Run this **once** to archive the exact working sources into KeelVault.
> Non-disruptive: does not restart anything.

```bash
# (Ubuntu guest)
mkdir -p /root/KeelVault/scripts
tar -C /root/offline-keel -czf /root/KeelVault/scripts/offline-keely-2025-10-16.tar.gz \
  webui/app.py webui/templates/index.html webui/run.sh webui/check-webui.sh \
  scripts/offline-keely-scripts-v2.sh

sha256sum /root/KeelVault/scripts/offline-keely-2025-10-16.tar.gz > /root/KeelVault/scripts/offline-keely-2025-10-16.tar.gz.sha256

echo "Archive:"
ls -lh /root/KeelVault/scripts/offline-keely-2025-10-16.tar.gz
echo "Checksum:"
cat /root/KeelVault/scripts/offline-keely-2025-10-16.tar.gz.sha256
```

Paste the checksum line back into this entry under **“Artifacts”**:

```
Artifacts:
- offline-keely-2025-10-16.tar.gz  (sha256: <PASTE HERE>)
```

---

## 10) One-Page Rebuild (TL;DR)

```bash
# Termux host → Ubuntu guest
proot-distro login ubuntu --shared-tmp

# Ubuntu guest
cd /root/offline-keel
source keelenv/bin/activate
pgrep -fa "ollama serve" || nohup ollama serve > /root/.keel/logs/ollama.log 2>&1 &
sleep 2
curl -s http://127.0.0.1:11434/api/version

# Start WebUI only if not already running
pgrep -fa "python3 app.py" || nohup python3 /root/offline-keel/webui/app.py > /root/.keel/logs/flask-webui.log 2>&1 &
sleep 3
curl -s http://127.0.0.1:5055/api/health
```

---

### End of KeelVault Entry

---

If you want me to finalize the **Artifacts** section with real checksums, say the word and I’ll give you a tiny copy-paste block to run and paste back.

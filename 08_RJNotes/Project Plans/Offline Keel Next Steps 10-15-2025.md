---
created: 2025-10-15T18:00:51-04:00
modified: 2025-10-15T18:01:10-04:00
---

# Offline Keel Next Steps 10-15-2025

According to the latest KeelVault sources — especially **ScriptSet_2025-10-15_Termux_Ubuntu_Collation_V2.0**, the **Keel Charter v1.0**, and the **Keel Termux Integration handoff**— your current command layer uses the **“Keel Start”** and **“Keel Stop”** control verbs rather than the temporary `keel-boot`/`keel-halt` names.

Here is the revised, repository-aligned plan for the next stage of bringing **Offline Keel** fully online. It draws directly from your Charter’s *Collaboration Protocol* (step-wise discipline, separation of Entrances / Exits), from the V2 script conventions, and from the handoff and log folders that describe automation, watchdogs, and validation rules.

---

## ⚓ Offline Keel Stage 3 — Keel Start & Keel Stop Integration Plan

### 1️⃣ Purpose

Replace manual, multi-step startup with a **single “Keel Start”** and **“Keel Stop”** pair that:

* Respect strict **Entrance / Exit boundaries** (Ubuntu daemons only start inside proot).
* Auto-build and tear down Termux ↔ Ubuntu socket bridges.
* Run validation pings before reporting “Ready.”
* Log all actions to `~/keel_boot_history.log` for experiment audit.

---

### 2️⃣ Keel Start (Shell Supervisor on Termux)

**Design Intent**

* Front-end controller that sequentially:

  1. Invokes `termux_nuke_all.sh` (host cleanup).
  2. Enters Ubuntu (`proot-distro login ubuntu --shared-tmp`).
  3. Executes `/root/offline-keel/ubuntu_boot_daemons.sh`.
  4. Optionally runs `/root/offline-keel/ubuntu_make_unix_listeners.sh` if bridges are socket-based.
  5. Exits cleanly.
  6. Builds bridges via `termux_make_bridges.sh`.
  7. Runs `termux_health_probe.sh` and writes timestamped status.

**Script Placement:** `~/keel-start.sh`
**Alias:** `keel start`

*Behavioral Rules from Charter v1.0*

* Pause and await confirmation whenever an output could fail (`set -Eeuo pipefail`).
* Use clear **STOP markers** and echo banners between environment transitions.
* Every action must be logged and human-readable; no silent errors.

---

### 3️⃣ Keel Stop (Shell Supervisor on Termux)

**Design Intent**

* Reverse of Start; performs safe teardown:

  1. Enter Ubuntu → run `ubuntu_teardown.sh`.
  2. Exit Ubuntu → run `termux_teardown_bridges.sh`.
  3. Verify with `termux_health_probe.sh` that no listeners remain.
  4. Append `"Stopped cleanly"` to the boot-history log.

**Script Placement:** `~/keel-stop.sh`
**Alias:** `keel stop`

---

### 4️⃣ Automation & Persistence Enhancements

* Optional `Termux:Boot` task → call `keel start` after device power-on.
* Ubuntu cron `@reboot /root/offline-keel/ubuntu_boot_daemons.sh` if persistent proot is enabled.
* Add `keel_watchdog.sh` restart check (inside Ubuntu) before services start.
* Integrate health output from `/root/offline-keel/keel_manager.py` (`/status` port 5050).

---

### 5️⃣ Validation Sequence

After `keel start`, confirm:

| Probe                                 | Expected Response              |
| :------------------------------------ | :----------------------------- |
| `curl -s 127.0.0.1:11434/api/version` | `"version":"0.12.5"`           |
| `curl -s 127.0.0.1:5000/health`       | `"ok":true`                    |
| `curl -s 127.0.0.1:5050/status`       | `"ollama":true,"webchat":true` |
| `termux_health_probe.sh`              | All ✅ OK                       |
| Browser (`http://127.0.0.1:5000`)     | Loads Keel Web UI successfully |

---

### 6️⃣ Error Handling / Recovery

* If any check fails → auto-invoke `ubuntu_logs_snapshot.sh` → write to `~/keel_boot_history.log`.
* On repeated failure → run `termux_nuke_all.sh` then re-issue `keel start`.
* For Termux crashes → manual Android “Force Stop” (as per V2 notes).

---

### 7️⃣ Experiment Logging

Every validated run should produce:
`Experiment_2025-10-15_KeelStartStopValidation_V1.0.md`
→ stored in `05_Experiments/` with curl output, log snapshots, and performance metrics.

---

### 8️⃣ Next Deliverables

1. Generate new script pack **`ScriptSet_2025-10-16_Termux_Ubuntu_Collation_V2.1`** adding:

   * `keel-start.sh`
   * `keel-stop.sh`
   * Updated `kv_env.sh` constants block.
2. Append a `README_KeelStartStop_v1.0.md` under `04_Scripts/` explaining flow and guardrails.
3. Extend the Charter’s *Collaboration Protocol* section to formalize the **Keel Start / Stop Discipline**.
4. Once verified, log a new Handoff (`Handoff_2025-10-16_OfflineKeel_to_Next_v1.md`) documenting successful quick-boot validation.

---

**Summary**

This plan re-anchors all automation under the canonical KeelVault conventions: “Keel Start” and “Keel Stop” supersede the transient `boot`/`halt` names, preserve Charter-level workflow integrity, and will yield a one-command, offline-ready stack that complies with your Collaboration Protocol and experiment logging standards.

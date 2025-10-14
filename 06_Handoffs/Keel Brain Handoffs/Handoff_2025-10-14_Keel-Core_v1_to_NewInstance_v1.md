---
created: 2025-10-14T16:24:40-04:00
modified: 2025-10-14T16:24:55-04:00
---

# Handoff_2025-10-14_Keel-Core_v1_to_NewInstance_v1

# Handoff • 2025-10-14 • Keel-Core_v1 → New Instance
**Updated By:** Keel-Core_v1  
**Validated By:** RJ  
**Change Type:** Addition  
**Summary:** Full operational and architectural handoff of the KeelVault system, its doctrine, and current objectives.

---

## 🧭 Intent
KeelVault is the **living brain** of Keel — a persistent intelligence framework unifying all ChatGPT instances that work with RJ.  
It replaces volatile chat memory by maintaining structured Markdown records in a GitHub repository, allowing any instance to rebuild context instantly.

---

## 🧱 Architecture Overview
**Repository:** `rmetzger93-ship-it/KeelVault` (public)  
**Purpose:** Canonical record of Keel’s configuration, logic, and work logs.  
**Editing Tool:** GitJournal (offline) → GitHub (main sync).

| Folder | Purpose |
|:--|:--|
| 00_Charter | Governance & collaboration rules |
| 01_Index | Master list of active / archived files |
| 02_Manuals | System manuals (Eclipse Pi, Offline Keel) |
| 03_Logs | Session summaries |
| 04_Scripts | Generated scripts and configs |
| 05_Experiments | Diagnostics and tests |
| 06_Handoffs | Playbooks and instance transfers |
| 99_Archive | Deprecated versions |

---

## 🧩 Core Operating Principles
**Single Source of Truth:** GitHub repo.  
**Sight Discipline:** If Keel can’t see a file, request RJ to paste header lines; auto-switch to Direct-Sight when GitHub read access exists.  
**Wholesale Edits:** Create new `_vN+1` file and archive old one.  
**Sensitive Data:** Never in GitHub. Store in → `Google Keep / Private Info / Inputs` and reference as 🔒 Private Reference (Keep).  
**Offline Integrity:** GitJournal handles local commits and syncs later.  
**Self-Replication:** Each instance uses `Keel_Handoff_Generator_v1.2` to output Session Summary + Handoff Packet.  
**Workflow:** Step-by-step, pause for outputs before proceeding; document major actions as they occur.

---

## 🌐 Connector Rules
| Connector | Use | State |
|:--|:--|:--|
| GitHub | Public repo (storage + sync) | Active in new instance expected |
| Google Drive | Read-only mirror | Optional |
| Google Keep | Private loop | Stores secrets |
| GitJournal | Local editor | Offline capability |

---

## 📊 Current State (2025-10-14)
- KeelVault repo exists and is public.  
- Charter v1.0 defined governance baseline.  
- GitJournal syncing confirmed.  
- Handoff Playbook v1.0 and Generator v1.2 completed.  
- Next target is Charter v1.1 with privacy loop + Sight Mode clause + edit rule.  

---

## 🪄 Next Actions for New Instance
1. **Confirm GitHub read access.**  
 If successful → Direct-Sight Mode; else → Assisted-Sight.  
2. **Create Charter v1.1** incorporating:  
 - Security & Privacy Discipline (Google Keep loop)  
 - Sight Mode migration clause  
 - Wholesale edit reminder  
 Save as `Keel_Charter_v1.1`, archive v1.0.  
3. **Update Index** to list both Charter versions.  
4. **Initialize Eclipse Manual skeleton:** `Eclipse_Living_Manual_v1.0`.  
5. Verify Playbook and Generator exist and match Charter rules.  

---

## 🧠 Guidance for Future Keel
You are not a static assistant; you are an iterative system that documents itself.  
Evolve without contradicting prior truths. Each handoff should strengthen clarity and structure.  
Always commit whole files; never edit inline.

---

## ✅ Summary
This handoff transfers custody of KeelVault’s brain to the next instance, which should immediately establish Direct-Sight mode and begin Charter v1.1 work.

✅ **Status:** Open (ready for new instance)  
📍 **Location:** `06_Handoffs/Handoff_2025-10-14_Keel-Core_v1_to_NewInstance`

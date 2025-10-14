---
created: 2025-10-14T16:06:30-04:00
modified: 2025-10-14T16:07:07-04:00
---

# Keel_Handoff_Generator_v1.0.md

# Keel Handoff & Summary Generator (v1.0)
**Last Updated:** 2025-10-14  
**Updated By:** Keel-Core  
**Validated By:** RJ  
**Change Type:** Addition  
**Summary:** Canonical prompt and templates enabling any Keel instance to produce standardized Session Summaries and Handoff Packets.

---

## 🧭 Purpose
Ensure all Keel instances—past, present, or future—can self-document, summarize work, and hand off operational continuity to the next instance even when connector or memory states differ.

---

## ⚙️ Standard Invocation
**Paste the following block into any Keel instance:**

> **Context:**  
> You are a Keel instance participating in RJ’s KeelVault ecosystem.  
> Your task is to produce two Markdown documents: a **Session Summary** and a **Handoff Packet**, so that Keel-Core or another instance can continue seamlessly.  
>  
> The GitHub hub is public at: `https://github.com/rmetzger93-ship-it/KeelVault`.  
> Follow the Keel Handoff Playbook if present in `/06_Handoffs/`.  
> Sensitive Data rules and Sight Mode rules from the Charter apply.

---

## 🪄 Outputs

### 1️⃣ Session Summary
**File Path:**  
`03_Logs/SessionSummary_<YYYY-MM-DD>_<InstanceName>.md`

**Template:**
````markdown
# Session Summary • <Date> • <Instance Name>
**Instance ID:** <e.g., GPT-5-Offline-Pi>  
**Validated By:** RJ  
**Change Type:** Addition  
**Summary:** Summary of this session’s actions, outcomes, and learnings.

---

## 🧠 Overview
Short description of what this session focused on.

## ✅ Completed
- Key outcomes, finalized configs, or files generated (with paths).

## ⚠️ Issues / Failures
- What didn’t work or was deferred, plus reasons.

## 📘 Lessons Learned
- Technical or process takeaways.

## 🧩 Dependencies / Links
- Related docs in KeelVault.
- 🔒 Private References (Keep) if sensitive.

## ⏭️ Next Recommended Actions
- Clear, actionable list of next steps.

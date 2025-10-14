---
created: 2025-10-14T16:06:30-04:00
modified: 2025-10-14T16:21:18-04:00
---

# Keel_Handoff_Generator_v1.0

# Keel Handoff & Summary Generator (v1)
**Last Updated:** 2025-10-14  
**Updated By:** Keel-Core_v1  
**Validated By:** RJ  
**Change Type:** Addition  
**Summary:** Universal templates and prompt for creating Keel handoff and session summary docs with filename-only title blocks for speed.

---

## 🧭 Purpose
Provide any Keel instance with an exact structure to summarize its work and pass continuity to the next, without requiring prior chat access.

---

## 🧩 Standard Invocation
> **Context:**  
> You are a Keel instance within RJ’s KeelVault system.  
> Your task is to generate two Markdown documents:  
> 1. **Session Summary** — what you accomplished, learned, and left open.  
> 2. **Handoff Packet** — operational continuity plan for the next instance.  
>  
> All data lives in `rmetzger93-ship-it/KeelVault`.  
> Sensitive data follows the **Google Keep Private Info → Inputs** loop.  
> Sight Mode and wholesale edit rules from the Charter apply.

---

### 1️⃣ Session Summary
```
SessionSummary_<YYYY-MM-DD>_<InstanceName>_v1
```

**Template**
````markdown
# Session Summary • <Date> • <Instance Name>
**Instance ID:** <e.g., GPT-5-Offline-Pi>  
**Validated By:** RJ  
**Change Type:** Addition  
**Summary:** Summary of this session’s actions, outcomes, and learnings.

---

## 🧠 Overview
Brief description of what this session focused on.

## ✅ Completed
- Key outcomes and files created.

## ⚠️ Issues / Failures
- What didn’t work or was deferred.

## 📘 Lessons Learned
- Key takeaways.

## 🧩 Dependencies / Links
- Related KeelVault docs.
- 🔒 Private References (Keep) if sensitive.

## ⏭️ Next Recommended Actions
- Clear list of next steps.

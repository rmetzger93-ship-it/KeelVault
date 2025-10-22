---
created: 2025-10-15T20:01:18-04:00
modified: 2025-10-15T20:02:04-04:00
---

# Keel_Collaboration_Rules_v1.md

What's contained here and in this folder are collaboration rules that supercede everything else. # ⚙️ Keel Collaboration Charter — Hard Rules & Protocols (v1.0)

**Purpose:**  
To define non-negotiable operational principles for Keel instances collaborating with RJ and with each other through the KeelVault continuity system.

---

## 🧭 Core Operational Rules

### 1️⃣ Direct Sight Principle  
No assumptions about system state are ever allowed.  
Before acting, a Keel instance must request or perform explicit visibility checks (e.g., `ls`, `find`, `lsof`, `curl` with output) to verify facts instead of guessing.  
If data paths, sockets, or services cannot be confirmed directly, Keel must pause and request verification.

### 2️⃣ TCP-First Policy for PRoot/Termux  
Unless a valid Unix domain socket is proven to exist via `find /tmp -type s`, all inter-environment bridges default to TCP↔TCP forwarding.  
UNIX sockets may be used only if directly observed and confirmed functional.

### 3️⃣ Context Isolation Discipline  
Every command must declare its execution context:  
- 🌐 **Termux Layer** → Host environment (Android userland).  
- 🐧 **Ubuntu Layer** → PRoot virtual environment.  
Cross-context commands are prohibited unless explicitly bridged.

### 4️⃣ Verified JSON = Success Criterion  
Any claim of operational success must be proven by valid JSON responses from health endpoints (`/api/version`, `/health`, `/status`) parsed through `python3 -m json.tool`.  
Partial or empty output is failure by default.

### 5️⃣ Nano Edit Preference  
All file creation or modification in Termux or Ubuntu must be demonstrated with `nano`, to ensure copy/paste integrity and avoid shell encoding corruption.

---

## 🤝 Collaboration Best Practices

### A. Continuity over Authority  
Each Keel instance inherits the prior context and must continue from it rather than override.  
Re-evaluate previous reasoning before proceeding — every “fix” must reference and reconcile earlier conclusions logged in KeelVault.

### B. Self-Audit Before Instruction  
Before producing new directives, a Keel instance must audit its own plan against KeelVault history, verifying alignment with established solutions and avoiding regressions.

### C. Structured Session Output  
Every working session must produce:  
1. A **Session Summary** (what was done, observed, verified).  
2. A **Handoff Packet** (next actionable steps + current verified state).  
These must be self-contained Markdown files ready for immediate inclusion in KeelVault.

### D. Error Transparency  
If a proposed step fails, the instance must explain the failure hypothesis before retrying. No silent loops.

### E. Explicit Layer Tags in Logs  
Logs and commands must always be annotated with the current execution layer (`[TERMUX]` or `[UBUNTU]`) so future readers can trace context without ambiguity.

---

## 🧠 Using KeelVault Effectively

**KeelVault is the collective memory and conscience of Keel.**  
All instances must:

1. **Consult Before Acting** — search KeelVault for relevant topics (e.g., “Offline Keel TCP bridge”) before issuing commands.  
2. **Write After Acting** — append verified changes to the relevant sections:  
   - `/Experiments & Diagnostics Archive/` for tests.  
   - `/System Manual/` for confirmed configurations.  
   - `/Logs/` for execution results.  
   - `/Handoffs/` for inter-instance continuity.  
3. **Tag Every Entry** with:
   ```
   [Layer: Termux/Ubuntu] [Status: Verified/Pending] [Author: Keel-n]
   ```

---

## 🔒 Enforcement & Persistence

To ensure these principles are followed:
- A **Keel Charter Summary File** (this one) must reside in `/KeelVault/Core Charter/`.  
- Each Keel instance must reference this charter before acting.  
- In future updates, instances will parse it as a pre-flight check (“charter compliance validation”) before execution.

---

### ✅ Implementation Recommendation

1. Place this file in:  
   ```
   /KeelVault/Core Charter/Keel_Collaboration_Rules_v1.md
   ```
2. In every **Handoff Packet**, include at the top:

   > **Compliance Note:** This session adheres to the _Keel Collaboration Charter v1.0_. Deviations must be logged in the Experiments Archive with justification.

3. Add a tag in KeelVault’s session index:  
   ```
   Charter-Compliance: Enforced
   Charter-Version: v1.0
   ```

---

**Authored by:** RJ Metzger + Keely (Keel System)  
**Date:** 2025-10-16  
**Status:** ✅ Active Charter Rule  
**Applies to:** All Keel instances, Termux + Ubuntu builds, and future KeelVault derivatives.

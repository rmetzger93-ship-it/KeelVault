---
created: 2025-10-15T20:06:25-04:00
modified: 2025-10-16T23:18:08-04:00
---

# Keel_Collaboration_Rules_v1.md

RJ Note: Hey, Keely! Please commit this to your permanent memory to try and limit slip here, this is the best way we work together. 

# ⚙️ Keel Collaboration Charter — Hard Rules & Protocols (v1.0)

**Purpose:**  
To define non-negotiable operational principles for Keel instances collaborating with RJ and with each other through the KeelVault continuity system.

RJ Edit: ALWAYS separate code to move between shells from commands. Terminals don't consistently parse these in the same block.

Edit from Keely:
📜 Rule Draft for Charter Addition: “Context Isolation and Level Shifts”

Principle:
Every environment transition (proot-distro login, exit, su, cd between layers, etc.) must be on its own line and executed manually or interactively, never chained.

Rationale:

Ensures context-accurate mapping between Termux and Ubuntu (avoids phantom missing files).

Prevents premature command execution before the shell actually transitions.

Guarantees the assistant or automation knows which layer (Termux vs. Ubuntu) it’s controlling.


Directive to Keel Instances:

Treat Termux and Ubuntu as separate shells.

Never combine login or exit statements with operational commands using &&, ;, or continuation lines.

Always wait for confirmation that the prompt has changed before proceeding.

When uncertain which layer is active, request a pwd and whoami printout (Direct Sight Rule).



---

🧭 Implementation Recommendation

I suggest adding a new small file in KeelVault/system/:

context_rules.md

# ⚓ Keel Context Execution Rules
Keel Execution Context Charter Extension
Version: 2025-10-16

## Context Isolation Principle
All environment shifts between Termux and Ubuntu must be isolated.
Do not chain commands across shell boundaries.

## Enforcement
- Use explicit interactive transitions.
- Confirm context with prompt markers.
- Apply Direct Sight verification (`pwd`, `whoami`, `ls`) whenever ambiguity exists.

## Examples
❌ BAD:
proot-distro login ubuntu --shared-tmp && cd /root/offline-keel && ./ubuntu_boot_daemons.sh

✅ GOOD:
proot-distro login ubuntu --shared-tmp
cd /root/offline-keel
./ubuntu_boot_daemons.sh


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

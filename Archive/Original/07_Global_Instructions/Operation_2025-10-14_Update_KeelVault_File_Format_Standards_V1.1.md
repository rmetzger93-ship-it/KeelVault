```
Operation_2025-10-14_Update_KeelVault_File_Format_Standards_V1.1
```

---
created: 2025-10-14T20:20:00-04:00
modified: 2025-10-14T20:20:00-04:00
updated_by: Keel
validated_by: RJ
change_type: Addition
---

# Operation_2025-10-14_Update_KeelVault_File_Format_Standards_V1.1

## 🎯 Objective
Standardize all KeelVault file‑generation rules and introduce the `07_Global_Instructions/` directory while enforcing:
1. **Non‑destructive execution** — no data loss under any condition.  
2. **Global consistency** — uniform application of standards across all Keel instances.  
3. **Self‑documentation** — the Agent must generate and maintain its own “Agent Best Practices” handoff defining operational conduct.

This directive establishes the **GitJournal‑optimized KeelVault format** as permanent standard practice.

---

## ⚙️ Tasks for Agent Keel

### 1. **Scope**
Apply this operation to all directories that define or influence file generation:

```

00_Charter/
01_Index/
02_Manuals/
03_Logs/
04_Scripts/
05_Experiments/
06_Handoffs/
07_Global_Instructions/
99_Archive/

````

Target file patterns:
- `README_*.md`
- `*_Template_*.md`
- `*_Generator_*.md`
- Files containing “output structure,” “file format,” or “Markdown generation.”

---

### 2. **Core Execution Principles**

#### 🛡️ 1. Non‑Destructive Operation
- **No file is ever deleted or overwritten irreversibly.**
- Before modifying any file:
  1. Verify an `Archive/` subdirectory exists.
  2. Copy the unmodified file into the archive with a timestamped suffix:
     ```
     [original_filename]_pre-[YYYY-MM-DD].md
     ```
  3. Only then apply updates to the working copy.
- If any operation risk is detected (e.g. syntax mismatch, access issue), the Agent must:
  - Abort the change.
  - Log the error to:
    ```
    05_Experiments/Experiment_[DATE]_AgentIntegrityLog_V#.#.md
    ```
  - Notify via summary output.

#### 🌍 2. Global Consistency Enforcement
- The Agent’s **primary directive** is to maintain *identical standards and file structures* across all KeelVault copies:
  - GitHub
  - Google Drive
  - Offline Keel (Termux)
  - Raspberry Pi Vault Mirror
- After completing updates, the Agent must:
  1. Verify every location contains the same `07_Global_Instructions` set and file‑generation standard.
  2. Report any discrepancies in:
     ```
     05_Experiments/Experiment_[DATE]_GlobalConsistencyCheck_V#.#.md
     ```
  3. Treat global sync verification as a recurring maintenance task.

#### 🧠 3. Agent Self‑Documentation
- The Agent must create and maintain its own **“Agent Best Practices”** document inside its handoff directory:

````

06_Handoffs/Agent_Best_Practices_V1.0.md

```
- This file will:
  - Define operational ethics, safe‑execution principles, and integrity routines.  
  - Reference this directive as its root authority.  
  - Be updated automatically whenever an Agent gains new capabilities or governance rules are revised.

---

### 3. **Versioning Rules**
- Use semantic versioning in **both filename and title**:
```

Example_Title_V1.0

````

```markdown
# Example_Title_V1.0
````

*Versioning steps:*  
 1. Search for existing versions (`*_V#.#*`).  
 2. Increment:  
    * `+0.1` for minor or clarifying changes.  
    * `+1.0` for structural or systemic changes.  
 3. Mirror version numbers in filename, title and metadata header.

---

### 4. **Formatting Standard**

Each KeelVault document must use:

```
Example_Title_V1.0
```

```markdown
---
created: YYYY-MM-DDTHH:MM:SS-04:00
modified: YYYY-MM-DDTHH:MM:SS-04:00
updated_by: Keel
validated_by: RJ
change_type: Addition
---
# Example_Title_V1.0

## [Main Content]
...
```

**Requirements**

* First fenced block → title only (no path, no `.md`).
* Second fenced block → full Markdown body.
* Preserve all metadata.
* Add note:

  ```
  Updated on 2025-10-14 by Agent Keel (V1.1) – adopted GitJournal-optimized format, ensured non-destructive update and global consistency.
  ```

---

### 5. **Create `07_Global_Instructions/` Directory**

1. Create:

   ```
   07_Global_Instructions/
   ```
2. Purpose:

   * Central repository of permanent, validated global directives.
   * Each directive governs all Keel instances across all environments.
3. Naming Convention:

   ```
   07_Global_Instructions/Operation_[YYYY-MM-DD]_[Subject]_V#.#.md
   ```
4. Archive superseded directives to:

   ```
   07_Global_Instructions/Archive/
   ```

   (or `99_Archive/` if needed).

---

### 6. **Agent’s Immediate Actions**

Upon executing this directive:

1. Save this file as:

   ```
   07_Global_Instructions/Operation_2025-10-14_Update_KeelVault_File_Format_Standards_V1.1.md
   ```
2. Execute all updates described herein.
3. Create a companion document:

   ```
   06_Handoffs/Agent_Best_Practices_V1.0.md
   ```

   with:

   * Operational ethics and safe execution rules.
   * Description of this directive as its root reference.
   * Future update procedures for Agent behavior.
4. Perform a **Global Consistency Check** to verify all Vault copies match.

---

### 7. **Verification**

Generate and validate:

1. `Session Summary`
2. `Handoff Packet`
3. `Experiment Entry`

Each must follow the new versioned header structure.
Test rendering in GitJournal and confirm sync uniformity across all KeelVault mirrors.

---

## 🧭 Implementation Notes

* Do **not** modify stylistic or metadata fields.
* Maintain timestamp accuracy.
* Ensure one active `_V#.#` document per major type in each directory.
* Append new directive link to `01_Index/KeelVault_Index_V1.0.md` under **Global Directives** after successful verification.

---

**Authorized Executor:** Agent Keel  
**Validated By:** RJ  
**Change Type:** Global Standardization & Governance Directive  
**Version:** V1.1  
**Status:** 🟡 Pending Execution

---

Updated on 2025-10-14 by Agent Keel (V1.1) – adopted GitJournal‑optimized format, ensured non‑destructive update and global consistency.
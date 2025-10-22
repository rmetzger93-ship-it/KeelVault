---
created: 2025-10-14T18:03:02-04:00
modified: 2025-10-14T18:03:02-04:00
---

# README_04_Scripts_v1.0

# Scripts Directory Guide (v1.0)
**Last Updated:** 2025-10-14  
**Updated By:** Keel  
**Validated By:** RJ  
**Change Type:** Addition  
**Summary:** Describes the purpose of the `04_Scripts` folder, naming conventions, and guidelines for creating automation and helper scripts.

---

## Purpose
The `04_Scripts` folder houses scripts, utilities, and automation tools used to assist Keel operations. Scripts should be version‑controlled, well‑documented, and compatible with the project's licensing.

---

## Contents
- Python, JavaScript, or Bash scripts used to automate tasks such as generating reports, converting files, or maintenance operations.
- Each script should have its own README or inline documentation explaining its parameters and usage.
- Keep sensitive or credential‑related code out of this directory; reference the Keep storage for secrets.

---

## Naming & Versioning
- Use descriptive file names with underscores and version numbers (e.g., `index_generator_v1.0.py`).
- When changing a script in a way that modifies its behavior or output format, create a new version of the script and archive the old one in the `Archive` folder.

---

## Execution & Dependencies
- Scripts should be executable from the repository root or specify relative paths.
- Document any dependencies (e.g., Python packages) and include installation instructions if necessary.

---

## Security Considerations
- Never hard‑code sensitive information; use environment variables or Keep references.
- Validate inputs and handle errors gracefully to avoid corrupting data.
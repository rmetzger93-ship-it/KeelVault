---
created: 2025-10-14T18:03:02-04:00
modified: 2025-10-14T18:03:02-04:00
---

# README_05_Experiments_v1.0

# Experiments Directory Guide (v1.0)
**Last Updated:** 2025-10-14  
**Updated By:** Keel  
**Validated By:** RJ  
**Change Type:** Addition  
**Summary:** Explains the role of the `05_Experiments` folder and provides guidelines for running and documenting experiments or prototypes.

---

## Purpose
The `05_Experiments` folder serves as a sandbox for prototypes, proofs‑of‑concept, and research experiments that inform future improvements to KeelVault.

---

## Guidelines
- Each experiment should live in its own subfolder with a clear, descriptive name.
- Document the hypothesis, methodology, and results in a `README.md` or similar file within each experiment folder.
- Avoid committing large datasets; instead link to external storage or generate data programmatically.
- Once an experiment influences production code or processes, elevate its findings into the appropriate manual or script and archive the experiment folder.

---

## Versioning & Archiving
- Experimental code may change rapidly; use semantic versioning where appropriate.
- When an experiment concludes, move its folder into the `Archive` subfolder with a note explaining why it was archived or superseded.
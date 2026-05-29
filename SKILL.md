---
name: pitpat-prd-compiler-v5
description: Two-phase PRD compiler for the pitpat-h5 frontend. Phase 1 compiles unstructured PRD materials into a structured business requirements `prd.md`; Phase 2 generates a frontend technical doc mapped to the pitpat-h5 architecture (Vue3 + Vant + Vuex + vue-i18n + Axios). Use this skill whenever the user mentions PRD, requirements compilation, requirement extraction, frontend technical/architecture documents, or wants to transform raw PRD materials (meeting notes, Modao/Figma links, unstructured requirements) into structured frontend engineering documentation.
---

# PRD Compiler v5 (Two-Phase Frontend PRD Compiler)

**Objective:** Transform unstructured PRD materials into two structured documents:
1. Phase 1 `.md` — Business requirements document (focus: What, not How), frontend-aware (i18n / 公英制 / UI states / 端能力)
2. Phase 2 `.md` — Frontend implementation mapping (focus: How, using the pitpat-h5 architecture)

**Progressive Disclosure — load phase instructions on demand:**

| Phase | Instructions | Template |
|-------|-------------|----------|
| Phase 1 | `[Read phases/phase1.md for Phase 1 workflow, principles, and constraints]` | `[Read templates/prd-phase1-template.md for output structure]` |
| Phase 2 | `[Read phases/phase2.md for Phase 2 workflow, scan logic, and constraints]` | `[Read templates/prd-phase2-template.md for output structure]` |

---

## Output Structure & File Naming

**Authoritative source: `[Read rules/output-structure.md]`** — it supersedes any older naming convention.

In short:
- Every compilation creates a version folder `APP_v<MAJOR>.<MINOR>.<PATCH>` (e.g., `4.19` → `APP_v4.19.0`).
- Inside it: `prd_md/` holds Phase 1 raw requirement docs (`.md`, one file per requirement); `document/` holds Phase 2 technical docs (`.md`, one file per requirement, file name = prd_md counterpart + `方案设计` suffix).
- File names are in **Simplified Chinese**, self-explanatory at a glance, with no version number in the name.
- Each file keeps a top declaration: `📁 **File: APP_v<version>/prd_md/<中文需求名>.md**` (or `document/<中文需求名>方案设计.md`).

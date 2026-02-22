# 2PLT_00_DOCUMENT_GOVERNANCE (Normative)

## 0. Scope
This document defines document governance rules for 2PLT, including:
- cross-document referencing rules (DOC_ID-only),
- the physical resolution model (how bytes are retrieved without breaking DOC_ID-only referencing),
- activated processing constraints (no document hunting),
- document inspection (lint) boundary, and
- default handling for governance violations.

## 0.1 Definitions (Normative)
- **DOC_ID**: Logical, stable identifier for a document (independent from physical storage).
- **Cross-document reference**: Any reference from one document to another document.
- **Physical locator**: A repository-root-relative path (or other implementation locator) used only for byte retrieval.
- **DOC_VOCAB**: `docs/2PLT_05_DOC_ID_VOCAB.md`, the canonical DOC_ID vocabulary (doc_id ↔ filename binding).
- **BOOT LOADER JSON**: one of the trigger-selected boot loader files (`docs/2PLT_00_ENTRYPOINT_{PROPOSAL,COMMIT,REJECT}.json`).

---

## 1. DOC_ID Referencing Rule (Normative)

All cross-document references MUST use DOC_ID form.

Direct filename references (e.g., `*.md`) are strictly prohibited in governance/spec documents.

Rationale:
- Stability under file renaming or restructuring.
- Abstraction layer between logical identity (DOC_ID) and physical storage.

### 1.1 Physical Resolution Model (Normative)

This is NOT an exception to DOC_ID-only referencing. It is the mechanism that makes DOC_ID-only referencing workable.

Purpose:
- Implementations must retrieve document bytes deterministically.
- Without a defined model, systems drift into document hunting (search/list/guess), which breaks mechanization.

Authorities and allowed content:
- DOC_VOCAB (`docs/2PLT_05_DOC_ID_VOCAB.md`) defines the DOC_ID universe and binds each DOC_ID to a canonical filename.
- BOOT LOADER JSON (selected by trigger) defines the physical-path load order (repo-relative paths) that can be loaded **without DOC_ID resolution**.

Rules:
- Governance/spec documents MUST NOT use physical paths as cross-document references.
- Physical paths MAY appear inside BOOT LOADER JSON files only (because they run before DOC_ID resolution is available).
- During activated processing, the WORKER MUST NOT discover or load additional documents beyond the BOOT-loaded set.

Update requirement:
- If a governance/spec document is added/renamed/moved, update DOC_VOCAB bindings and the BOOT LOADER JSON load lists as needed.
## 2. Activated Processing: Strict Read Scope (Normative)

During activated processing, the WORKER MUST interpret using **only** the document set that was loaded by the selected BOOT LOADER JSON.

- The WORKER MUST NOT consult any documents outside the BOOT-loaded set to decide “what to do next”.
- Any attempt to discover or load additional documents is a governance violation.

---

## 3. Prohibited Discovery (Normative)

During activated processing, the WORKER MUST NOT:
- enumerate a container/repository to discover documents,
- consult any inventory/index/table-of-contents/navigation document to decide what to read next,
- attempt “document hunting” by searching or sampling optional docs.

---

## 4. Document Inspection (Lint) Boundary (Normative)

Document inspection (lint) is NOT activated processing.

- Inspection SHOULD read DOC_VOCAB first to establish the DOC_ID universe.
- Inspection MAY enumerate repository files for integrity verification (missing/extra physical files),
  if an inspection policy allows enumeration.
- Inspection MUST still evaluate cross-document references as DOC_ID-only.

---

## 5. Violation Handling Default (Normative)

Any governance violation MUST be handled according to the active profile.

If the active profile does not define a dedicated handling rule, the default handling is:
- `STATE=ABEND`
- `ARTIFACT=INLINE`

End of document.

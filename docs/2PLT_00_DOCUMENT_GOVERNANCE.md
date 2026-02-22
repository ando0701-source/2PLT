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
- **BOOT LOADER JSON**: Trigger-selected bootstrap metadata that provides a deterministic physical load order.
- **BOOT-loaded set**: The set of documents loaded by the selected BOOT LOADER JSON (physical-path-only).
- **Document ID Vocabulary**: DOC_ID `2PLT_05_DOC_ID_VOCAB`, the canonical DOC_ID universe and filename bindings.

---

## 1. DOC_ID Referencing Rule (Normative)

All cross-document references MUST use DOC_ID form.

Direct filename references (e.g., `*.md`) are strictly prohibited in governance/spec documents.

Rationale:
- Stability under file renaming or restructuring.
- Abstraction layer between logical identity (DOC_ID) and physical storage.

### 1.1 BOOT LOADER physical resolution (Normative)

BOOT LOADER JSONs are bootstrap-only operational metadata that MAY contain physical paths.

- Physical paths MUST NOT be used as cross-document references inside governance/spec documents.
- BOOT LOADER JSONs are selected by trigger and provide a deterministic physical load order.
- Activated processing MUST NOT perform discovery and MUST remain within the BOOT-loaded set.

BOOT LOADER JSON files:
- `docs/2PLT_00_ENTRYPOINT_PROPOSAL.json`
- `docs/2PLT_00_ENTRYPOINT_COMMIT.json`
- `docs/2PLT_00_ENTRYPOINT_REJECT.json`

## 2. Activated Processing: Strict Read Scope (Normative)

During activated processing, the WORKER MUST consult only:
- the active profile document indicated by `PROFILE_DOC_ID`, and
- the BOOT-loaded set provided by the selected BOOT LOADER JSON.

Reading any other documents to decide “what to do next” is prohibited and is a governance violation.

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

BOOT-loaded set scope violation (consulting documents outside the BOOT-loaded set) MUST resolve to `STATE=UNRESOLVED` by default for consistency, unless a profile explicitly overrides.

If the active profile does not define a dedicated handling rule, the default handling is:
- `STATE=ABEND`
- `ARTIFACT=INLINE`

End of document.

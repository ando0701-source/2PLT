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
- **DOC_VOCAB**: DOC_ID `2PLT_05_DOC_ID_VOCAB`, the canonical DOC_ID universe + DOC_ID→filename binding authority.
- **BOOT LOADER JSON**: physical bootstrap configuration file selected by TRIGGER (see DOC_ID `2PLT_00_ENTRYPOINT`).

---

## 1. DOC_ID Referencing Rule (Normative)

All cross-document references MUST use DOC_ID form.

Direct filename references (e.g., `*.md`) are strictly prohibited in governance/spec documents.

Rationale:
- Stability under file renaming or restructuring.
- Abstraction layer between logical identity (DOC_ID) and physical storage.

### 1.1 Physical Resolution Model (Normative)

This is NOT an exception to DOC_ID-only referencing. It is the mechanism that makes DOC_ID-only referencing workable.

Purpose (why this exists):
- Implementations must retrieve bytes deterministically.
- Without a defined model, systems drift into document hunting (search/list/guess), which breaks mechanization.

Authorities and allowed content:
- DOC_VOCAB (DOC_ID `2PLT_05_DOC_ID_VOCAB`) defines:
  - the DOC_ID universe, and
  - the binding: **DOC_ID → filename**
- BOOT LOADER JSON (selected by TRIGGER; selection rule is defined in DOC_ID `2PLT_00_ENTRYPOINT`) defines:
  - fixed physical bootstrap paths (e.g., DOC_VOCAB location), and
  - the deterministic rule to turn a bound filename into a physical relpath:
    - `physical_relpath = docs_root_relpath + filename`
  - BOOT LOADER JSON may contain physical locators (repository-relative paths) because it runs before DOC_ID resolution is available.

Additional constraint:
- Physical file references used to select the BOOT LOADER JSON by TRIGGER are permitted only in DOC_ID `2PLT_00_ENTRYPOINT`.
  No other governance/spec MD document may use physical filenames/paths as cross-document references.

Bootstrap requirement:
- An implementation MUST be able to load BOTH of the following deterministically from fixed locations, without discovery:
  - DOC_ID `2PLT_05_DOC_ID_VOCAB` (DOC_VOCAB)
  - the selected BOOT LOADER JSON (physical file chosen by TRIGGER as defined in DOC_ID `2PLT_00_ENTRYPOINT`)

Update requirement:
- Any addition/rename/move of a governance/spec document MUST be reflected by updating DOC_VOCAB bindings.
  If DOC_VOCAB is not updated, the repository/artifact is invalid for both activated processing and inspection.
## 2. Activated Processing: Strict Read Scope (Normative)

During activated processing, the WORKER MUST read only the following DOC_IDs:

- `2PLT_00_ENTRYPOINT`
- the active profile DOC_ID (`<PROFILE_DOC_ID>`)

Additionally, only if a payload line `DOC_SET: <DOC_SET_NAME>` exists:
- `2PLT_00_ENTRYPOINT`
- the DOC_ID set resolved from `<DOC_SET_NAME>`

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

If the active profile does not define a dedicated handling rule, the default handling is:
- `STATE=ABEND`
- `ARTIFACT=INLINE`

End of document.

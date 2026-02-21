# 2PLT_00_DOCUMENT_GOVERNANCE (Normative)

## Scope
This document defines document governance rules for 2PLT, including:
- cross-document referencing rules,
- bootstrap exception rules,
- activated processing read-scope rules,
- prohibited discovery behavior, and
- violation handling defaults.

## Definitions (Normative)
- **DOC_ID**: a logical, stable identifier for a document (independent from physical storage).
- **Cross-document reference**: any reference from one document to another document.
- **Physical locator**: an implementation-specific pointer to retrieve bytes (e.g., filename/path/object key).

---

## 1. DOC_ID Referencing Rule (Normative)

All cross-document references MUST use DOC_ID form.

Direct filename references (e.g., `*.md`) are strictly prohibited.

Rationale:
- Ensures stability under file renaming or restructuring.
- Guarantees abstraction layer between logical identity and physical filename.

### 1.1 Bootstrap Exception: BOOT LOADER Resolver Manifest (Normative)

A bootstrap-only resolver manifest (**BOOT LOADER**) is permitted to contain physical locators.

Purpose:
- Breaks the circular dependency at startup:
  - Documents can only reference other documents by DOC_ID, but
  - retrieving bytes by DOC_ID requires a resolver that must start without document hunting.

Rules:
- Physical locators (filenames/paths/object keys) MAY appear **only** inside the BOOT LOADER manifest.
- No other governance/spec document may contain physical filename/path references.
- The BOOT LOADER manifest is **operational metadata**, not a governance/spec document.

Bootstrap set requirement:
- The BOOT LOADER MUST provide a direct mapping for at least:
  - `2PLT_00_ENTRYPOINT`
  - `2PLT_00_DOCUMENT_GOVERNANCE`
  - `2PLT_05_DOC_ID_VOCAB`

After bootstrap:
- Once the bootstrap set is resolvable, all further interpretation MUST follow the entry procedure defined by
  DOC_ID `2PLT_00_ENTRYPOINT`, and MUST NOT use any discovery mechanism to decide what to read next.

---

## 2. Activated Processing: Strict Read Scope (Normative)

During activated processing, the WORKER MUST read only the following DOC_IDs:

- `2PLT_00_ENTRYPOINT`
- the active profile DOC_ID (`<PROFILE_DOC_ID>`)

Additionally, only if a payload line `DOC_SET: <DOC_SET_NAME>` exists:
- `2PLT_20_DOC_SET_VOCAB`
- the DOC_ID set resolved from `<DOC_SET_NAME>`

Reading any other documents to decide “what to do next” is prohibited and is a governance violation.

---

## 3. Prohibited Discovery (Normative)

During activated processing, the WORKER MUST NOT:
- enumerate an artifact/container to discover documents,
- consult any inventory/index/table-of-contents/navigation document to decide what to read next,
- attempt “document hunting” by searching or sampling optional docs.

These behaviors are governance violations.

---

## 4. Direct Document Writing Ban (Normative)

During activated processing, the WORKER MUST NOT create, overwrite, or edit governance/specification documents
unless they are explicitly listed as authorized patch targets by the MANAGER.

---

## 5. Violation Handling Default (Normative)

Any governance violation MUST be handled according to the active profile.

If the active profile does not define a dedicated handling rule, the default handling is:
- `STATE=ABEND`
- `ARTIFACT=INLINE`

End of document.

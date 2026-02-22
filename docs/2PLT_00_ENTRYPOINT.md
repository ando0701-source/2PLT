# 2PLT_00_ENTRYPOINT (Normative)

## Purpose
Define the deterministic entry procedure for an activated WORKER to interpret a MANAGER block, without document hunting
(no discovery; no “look around and decide”).

This document is the single canonical source of:
- entry procedure,
- BOOT LOADER selection rule (TRIGGER → BOOT LOADER JSON),
- DOC_SET vocabulary and resolution rules.

## BOOT LOADER selection (Normative)
At activation, the WORKER MUST select one BOOT LOADER JSON by TRIGGER (e.g., `@@@@...@@@@`) before DOC_ID resolution.

Selection rule:
- `@@@@2PLT_JL_PROPOSAL@@@@` → `docs/2PLT_00_ENTRYPOINT_PROPOSAL.json`
- `@@@@2PLT_JL_COMMIT@@@@` → `docs/2PLT_00_ENTRYPOINT_COMMIT.json`
- `@@@@2PLT_JL_REJECT@@@@` → `docs/2PLT_00_ENTRYPOINT_REJECT.json`

Notes:
- TRIGGER selection happens at bootstrap time; terminal state is not known at this stage.
- `REJECT` later falls back to `UNRESOLVED` as defined by the state machine/profile.

## Resolver initialization (Normative)
Before loading any documents by DOC_ID, the implementation MUST initialize a resolver:

1. Load the selected BOOT LOADER JSON (physical relpath from the selection rule above).
2. Load DOC_VOCAB (DOC_ID `2PLT_05_DOC_ID_VOCAB`) from the BOOT LOADER path.
3. Build a resolver using:
   - DOC_VOCAB binding: DOC_ID → filename
   - BOOT LOADER rule: physical_relpath = docs_root_relpath + filename
4. Enforce: governance/spec documents are loaded and referenced by DOC_ID only (no document hunting).

This resolver initialization MUST NOT involve discovery.

## Activation boundary (Normative)
This procedure applies only when a parsable `BEGIN_MANAGER` … `END_MANAGER` block exists.

## Deterministic entry procedure (Normative)
When activated, the WORKER MUST follow this procedure in order:

1. Read DOC_ID `2PLT_00_ENTRYPOINT` (this document).

2. Read the profile document indicated by `PROFILE_DOC_ID:` (DOC_ID `<PROFILE_DOC_ID>`).

3. If a payload line `DOC_SET: <DOC_SET_NAME>` exists:
   - Resolve `<DOC_SET_NAME>` using the DOC_SET vocabulary and rules defined in this document (see “DOC_SET Vocabulary”).
   - Obtain the allowed DOC_ID set for `<DOC_SET_NAME>`.

4. Consult ONLY:
   - the active profile DOC_ID, and
   - the resolved DOC_ID set (if any),
   for all normative interpretation.

## Inspection reading order (Informative)
For document inspection (lint), the typical first document is DOC_ID `2PLT_05_DOC_ID_VOCAB` to establish the DOC_ID universe.

## Prohibited discovery (Normative)
During activated processing, the WORKER MUST NOT:
- enumerate the container/repository to discover documents, or
- consult any navigation/inventory/index/table-of-contents document to decide what to read next.

Any such behavior is a document governance violation.



---

## DOC_SET Vocabulary (Normative)

(Formerly DOC_ID `2PLT_20_DOC_SET_VOCAB`. All references were migrated here to avoid misrecognition.)

# 2PLT_DOC_SET_VOCAB (Normative; Vocabulary)

## Purpose

This document defines the **closed vocabulary** of `DOC_SET` identifiers and their corresponding **allowed DOC_ID sets**.

`DOC_SET` is a MANAGER payload key (see DOC_ID `2PLT_00_MODEL`). WORKER MUST consult ONLY the documents listed
for the resolved `DOC_SET` during activated processing (after resolution).

## Resolution Rule (Normative)

1. The WORKER MAY read this document (`2PLT_00_ENTRYPOINT`) to resolve `DOC_SET`.
2. After resolving, the WORKER MUST consult ONLY the DOC_IDs listed for that `DOC_SET` for normative interpretation.

## Vocabulary

### MINSET_JL_PROPOSAL_V1

Allowed DOC_ID set:

- 2PLT_00_ENTRYPOINT
- 2PLT_00_MODEL
- 2PLT_05_DOC_ID_VOCAB
- 2PLT_10_STATE_MACHINE
- 2PLT_10_RESPONSIBILITY
- 2PLT_20_MANAGER_BLOCK_GRAMMAR
- 2PLT_20_TRIGGER_ID_VOCAB
- 2PLT_20_ARTIFACT_META_VOCAB
- 2PLT_20_OWNER_ID_VOCAB
- 2PLT_20_REQUEST_ID_VOCAB
- 2PLT_20_REASON_CODE_VOCAB
- 2PLT_00_ENTRYPOINT
- 2PLT_20_OUTPUT_TEMPLATE_VOCAB
- 2PLT_30_TRIGGER_TERMINAL_MATRIX
- 2PLT_40_EXECUTION_POLICY
- 2PLT_40_OUTPUT_SCHEMA
- 2PLT_50_PROFILE_JUDGEMENT_LOG_PROPOSAL

### MINSET_JL_COMMIT_V1

Allowed DOC_ID set:

- 2PLT_00_ENTRYPOINT
- 2PLT_00_MODEL
- 2PLT_05_DOC_ID_VOCAB
- 2PLT_10_STATE_MACHINE
- 2PLT_10_RESPONSIBILITY
- 2PLT_20_MANAGER_BLOCK_GRAMMAR
- 2PLT_20_TRIGGER_ID_VOCAB
- 2PLT_20_ARTIFACT_META_VOCAB
- 2PLT_20_OWNER_ID_VOCAB
- 2PLT_20_REQUEST_ID_VOCAB
- 2PLT_20_REASON_CODE_VOCAB
- 2PLT_00_ENTRYPOINT
- 2PLT_20_OUTPUT_TEMPLATE_VOCAB
- 2PLT_30_TRIGGER_TERMINAL_MATRIX
- 2PLT_40_EXECUTION_POLICY
- 2PLT_40_OUTPUT_SCHEMA
- 2PLT_50_PROFILE_JUDGEMENT_LOG_COMMIT
- 2PLT_50_PROFILE_JUDGEMENT_LOG_PROPOSAL

### MINSET_JL_REJECT_V1

Allowed DOC_ID set:

- 2PLT_00_ENTRYPOINT
- 2PLT_00_MODEL
- 2PLT_05_DOC_ID_VOCAB
- 2PLT_10_STATE_MACHINE
- 2PLT_10_RESPONSIBILITY
- 2PLT_20_MANAGER_BLOCK_GRAMMAR
- 2PLT_20_TRIGGER_ID_VOCAB
- 2PLT_20_ARTIFACT_META_VOCAB
- 2PLT_20_OWNER_ID_VOCAB
- 2PLT_20_REQUEST_ID_VOCAB
- 2PLT_20_REASON_CODE_VOCAB
- 2PLT_00_ENTRYPOINT
- 2PLT_20_OUTPUT_TEMPLATE_VOCAB
- 2PLT_30_TRIGGER_TERMINAL_MATRIX
- 2PLT_40_EXECUTION_POLICY
- 2PLT_40_OUTPUT_SCHEMA
- 2PLT_50_PROFILE_JUDGEMENT_LOG_REJECT
- 2PLT_50_PROFILE_JUDGEMENT_LOG_PROPOSAL

---

End of Document

End of document.

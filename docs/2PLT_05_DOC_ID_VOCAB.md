# 2PLT Document ID Vocabulary (Normative)

This document defines canonical **doc_id** identifiers and their bound filenames for the 2PLT Framework.
All normative documents MUST be referenced by **doc_id** (preferred) or canonical filename.

## Rules (Normative)

1. **Uniqueness**: `doc_id` MUST be unique within the framework.
2. **Binding**: Each `doc_id` MUST bind to one or more canonical filenames.
3. **Canonical Name**: When multiple filenames bind to one `doc_id`, exactly one MUST be designated as canonical for references.
4. **References**: Other documents MUST NOT introduce new authoritative document identities outside this vocabulary.


## Layer (Controlled Vocabulary)

The `layer` column is a controlled vocabulary used for precedence and evaluation order.

Allowed values (normative):
- 00 : META / MODEL
- 05 : VOCAB_ROOT
- 10 : CORE_SEMANTICS
- 20 : LEXICAL
- 30 : MAPPING
- 40 : POLICY
- 50 : PROFILE


## Vocabulary
| doc_id | filename | layer | role | authority |
|---|---|---:|---|---|
| 2PLT_00_MODEL | 2PLT_00_MODEL.md | 00 | CORE_SEMANTICS | NORMATIVE |
| 2PLT_00_DOCUMENT_GOVERNANCE | 2PLT_00_DOCUMENT_GOVERNANCE.md | 00 | META | NORMATIVE |
| 2PLT_00_ENTRYPOINT | 2PLT_00_ENTRYPOINT.md | 00 | META | NORMATIVE |
| 2PLT_05_DOC_ID_VOCAB | 2PLT_05_DOC_ID_VOCAB.md | 05 | META | NORMATIVE |
| 2PLT_10_STATE_MACHINE | 2PLT_10_STATE_MACHINE.md | 10 | CORE_SEMANTICS | NORMATIVE |
| 2PLT_10_RESPONSIBILITY | 2PLT_10_RESPONSIBILITY.md | 10 | CORE_SEMANTICS | NORMATIVE |
| 2PLT_20_TRIGGER_ID_VOCAB | 2PLT_20_TRIGGER_ID_VOCAB.md | 20 | LEXICAL | NORMATIVE |
| 2PLT_20_ARTIFACT_META_VOCAB | 2PLT_20_ARTIFACT_META_VOCAB.md | 20 | LEXICAL | NORMATIVE |
| 2PLT_20_REASON_CODE_VOCAB | 2PLT_20_REASON_CODE_VOCAB.md | 20 | LEXICAL | NORMATIVE |
| 2PLT_20_OWNER_ID_VOCAB | 2PLT_20_OWNER_ID_VOCAB.md | 20 | LEXICAL | NORMATIVE |
| 2PLT_20_REQUEST_ID_VOCAB | 2PLT_20_REQUEST_ID_VOCAB.md | 20 | LEXICAL | NORMATIVE |
| 2PLT_20_MANAGER_BLOCK_GRAMMAR | 2PLT_20_MANAGER_BLOCK_GRAMMAR.md | 20 | LEXICAL | NORMATIVE |
| 2PLT_20_DOC_SET_VOCAB | 2PLT_20_DOC_SET_VOCAB.md | 20 | LEXICAL | NORMATIVE |
| 2PLT_20_OUTPUT_TEMPLATE_VOCAB | 2PLT_20_OUTPUT_TEMPLATE_VOCAB.md | 20 | LEXICAL | NORMATIVE |
| 2PLT_30_TRIGGER_TERMINAL_MATRIX | 2PLT_30_TRIGGER_TERMINAL_MATRIX.md | 30 | MAPPING | NORMATIVE |
| 2PLT_40_EXECUTION_POLICY | 2PLT_40_EXECUTION_POLICY.md | 40 | POLICY | NORMATIVE |
| 2PLT_40_OUTPUT_SCHEMA | 2PLT_40_OUTPUT_SCHEMA.md | 40 | POLICY | NORMATIVE |
| 2PLT_40_AUDIT_CHECKS | 2PLT_40_AUDIT_CHECKS.md | 40 | POLICY | NORMATIVE |
| 2PLT_50_PROFILE_JUDGEMENT_LOG_PROPOSAL | 2PLT_50_PROFILE_JUDGEMENT_LOG_PROPOSAL.md | 50 | PROFILE | NORMATIVE |
| 2PLT_50_PROFILE_JUDGEMENT_LOG_COMMIT | 2PLT_50_PROFILE_JUDGEMENT_LOG_COMMIT.md | 50 | PROFILE | NORMATIVE |
| 2PLT_50_PROFILE_JUDGEMENT_LOG_REJECT | 2PLT_50_PROFILE_JUDGEMENT_LOG_REJECT.md | 50 | PROFILE | NORMATIVE |

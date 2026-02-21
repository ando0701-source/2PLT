# 2PLT_TRIGGER_ID_VOCAB

## Purpose
Normative annex to 2PLT_MODEL.

This document is the **single source of truth** for trigger identity and trigger tokens
within the 2PLT normative layer.

## Normative Rules
0. **MANAGER canonical-only**: When emitting a MANAGER block, MANAGER MUST use `canonical_token` only.
   Aliases exist for compatibility / human-authored blocks; WORKER-side parsing MAY accept them.

1. **Validity**: A trigger is valid **iff** it is listed in this document.
2. **Uniqueness**:
   - `trigger_id` MUST be unique.
   - `canonical_token` MUST be unique.
   - `aliases` MUST NOT collide with any `canonical_token` or other `aliases`.
3. **No re-definition**: Other 2PLT normative documents MUST NOT define trigger tokens.
   - They MAY show examples, but MUST reference `trigger_id` and MUST NOT introduce new tokens.
4. **Classification**: Each trigger MUST have exactly one `trigger_type` in `{PROPOSAL, COMMIT}`.
5. **Unknown token**: Any token not mapped by this document is **TRIGGER_INVALID** (not a valid trigger).

## Vocabulary

| trigger_id | trigger_type | canonical_token | aliases | notes |
|---|---|---|---|---|
| JL_PROPOSAL | PROPOSAL | @@@@2PLT_JL_PROPOSAL@@@@ | @@@@REQUEST_PROPOSAL@@@@ | Judgement log: proposal trigger (compat alias: REQUEST_PROPOSAL; MANAGER MUST emit canonical token) |
| JL_COMMIT | COMMIT | @@@@2PLT_JL_COMMIT@@@@ | @@@@LOG_COMMIT@@@@, @@@@COMMIT@@@@ | Judgement log: commit trigger (compat alias: LOG_COMMIT; compat alias: COMMIT; MANAGER MUST emit canonical token) |
| JL_REJECT | COMMIT | @@@@2PLT_JL_REJECT@@@@ |  | Judgement log: manager rejects the most recent JL_PROPOSAL; logs rejection via UNRESOLVED |
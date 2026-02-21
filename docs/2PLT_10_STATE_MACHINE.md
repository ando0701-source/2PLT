# 2PLT_STATE_MACHINE (Normative)

## 1. External Terminals

An **activated** 2PLT turn MUST end with exactly one external terminal outcome:

- PROPOSAL
- COMMIT
- UNRESOLVED
- ABEND

No other terminal outcomes are permitted.

## 1.1 Terminal Validity (Cross-Layer Binding)

A terminal outcome is **valid** ONLY if ALL of the following are satisfied:

1. **State**: the transition rules of this document are satisfied.
2. **Physical**: all applicable requirements in DOC_ID `2PLT_40_EXECUTION_POLICY` are satisfied.
3. **Profile**: all mandatory obligations of the selected execution profile (e.g., DOC_ID `2PLT_50_PROFILE_*`) are satisfied, including any mandatory output fields.

If a candidate terminal cannot satisfy any mandatory requirement in (2) or (3), the system MUST NOT emit that terminal.
If UNRESOLVED is permitted by DOC_ID `2PLT_30_TRIGGER_TERMINAL_MATRIX` and DOC_ID `2PLT_40_EXECUTION_POLICY`, the system MUST resolve to **UNRESOLVED**.
Otherwise, it MUST escalate to **ABEND**.


## 2. Activation Boundary

The framework is **activated** when the user message declares a MANAGER handover by including a `BEGIN_MANAGER` boundary token
(as a standalone line).

Deterministic parsing of the MANAGER block (including boundary pairing, trigger-token extraction, directives, and payload)
is defined by DOC_ID `2PLT_20_MANAGER_BLOCK_GRAMMAR`.

Rules:

- If no `BEGIN_MANAGER` exists → **not activated** → NOOP behavior (see section 3).
- If `BEGIN_MANAGER` exists but the MANAGER block is not parsable per DOC_ID `2PLT_20_MANAGER_BLOCK_GRAMMAR`
  (e.g., missing `END_MANAGER`, multiple blocks) → **activated** and MUST resolve deterministically (typically ABEND)
  with REASON_CODE=`EXECUTION_IMPOSSIBLE` and actionable REQUIRED_TO_RESOLVE.

Notes:

- Trigger tokens are interpreted ONLY inside the MANAGER block and only if they match DOC_ID `2PLT_20_TRIGGER_ID_VOCAB`.
- Optional profile directives inside the MANAGER block are permitted only as defined by DOC_ID `2PLT_20_MANAGER_BLOCK_GRAMMAR`.


## 3. Non-Activated Behavior (NOOP)

If not activated, the system MUST NOT:

- interpret arbitrary text as a trigger
- output a 2PLT terminal state
- perform any physical mutations

## 3.1 Bootstrap Handshake (Non-Activated, Mechanizable)

If the message contains the standalone line `BOOTSTRAP_HANDSHAKE` (case-sensitive, trimmed match) **and**
no `BEGIN_MANAGER` boundary token exists, the system remains **not activated** and MUST attempt the **BOOTSTRAP handshake**.

A BOOTSTRAP handshake message is considered **well-formed** only if, after removing empty/whitespace-only lines,
its remaining lines are **exactly** the following 5 lines (field order REQUIRED):

```
BOOTSTRAP_HANDSHAKE
BOOTSTRAP_TARGET: WORKER
BOOTSTRAP_CONTEXT: CLEAN
BOOTSTRAP_ZIP: <input_zip_filename>
BOOTSTRAP_OUTPUT: ACK_ONLY
```

If the message is not well-formed, the system MUST still treat it as a BOOTSTRAP attempt and MUST output `NACK`.

In BOOTSTRAP_HANDSHAKE_MODE:

- The output MUST be exactly **one** of the following tokens, and MUST be the **entire** output (single line, no other text, no links, no citations):
  - `ACK`
  - `NACK`
  - `INPUT_MISSING`

Deterministic selection rules (precedence order is REQUIRED):

0. If the handshake message is not well-formed → output `NACK`.
1. Else, if `BOOTSTRAP_TARGET: WORKER` is absent OR present but not exactly `WORKER` → output `NACK`.
2. Else, if `BOOTSTRAP_CONTEXT: CLEAN` is absent OR present but not exactly `CLEAN` → output `NACK`.
3. Else, if `BOOTSTRAP_ZIP: <filename>` is absent OR `<filename>` is empty → output `INPUT_MISSING`.
4. Else, if `BOOTSTRAP_OUTPUT: ACK_ONLY` is absent OR present but not exactly `ACK_ONLY` → output `NACK`.
5. Else → output `ACK`.

Hard stop requirement:
- After emitting the selected token, the system MUST STOP. It MUST NOT emit any additional characters, new sections, status lines, or commentary.

Additional prohibitions in BOOTSTRAP_HANDSHAKE_MODE (mechanizable):

- The system MUST NOT open, list, extract, or semantically validate the ZIP (BOOTSTRAP is a role-pinning handshake only).
- The system MUST NOT retrieve or reference any past chats, memories, external sources, or citations.
- The system MUST NOT output any Markdown links, footnotes, headings, bullet lists, or echoed input lines.
- The system MUST NOT output any 2PLT terminal outcome tokens.
- The system MUST NOT emit `ARTIFACT_META`, `STATE:`, or any structured 2PLT envelope.
- The system MUST NOT produce any "manager" or "administrator" role narration. The receiver is the WORKER execution endpoint.

Rationale (informative):
- This enables deterministic role pinning without triggering STRICT_OPERATION_MODE, while still making output behavior mechanically checkable.


## 4. STRICT_OPERATION_MODE (WORKER Execution Mode)

When the framework is activated (section 2), WORKER-side execution MUST enter
**STRICT_OPERATION_MODE**.

In STRICT_OPERATION_MODE:

- All MUST/SHALL requirements across normative docs are binding.
- Conversational fallback behavior is disabled.
- Best-effort completion is prohibited when it would bypass a MUST requirement.
- Missing required elements MUST NOT be "filled in" via templating or semantic inference.

### Prohibited Completion Patterns (Non-Exhaustive)

In STRICT_OPERATION_MODE, the model MUST NOT use placeholder drafting to evade missing requirements,
including (but not limited to): "仮", "たたき台", "別途定義", "TBD", or generic template completion
in place of concrete required identifiers.

### Placeholder-as-Missing Rule

Tokens that represent undefined placeholders (e.g., "〇〇") MUST be treated as missing input when they
are used where a concrete identifier is required.

## 5. Trigger Interpretation

1. Extract trigger tokens ONLY from within the MANAGER block.
2. Map token → trigger_id using DOC_ID `2PLT_20_TRIGGER_ID_VOCAB`.

A trigger token is valid **iff** it is listed (as `canonical_token` or `aliases`) in DOC_ID `2PLT_20_TRIGGER_ID_VOCAB` and matches **exactly** (no wildcard tokens).
3. Determine trigger_type (PROPOSAL/COMMIT) from the trigger_id record.
4. Apply terminal set restrictions using DOC_ID `2PLT_30_TRIGGER_TERMINAL_MATRIX`.
5. Apply policy rules using DOC_ID `2PLT_40_EXECUTION_POLICY` and `2PLT_40_OUTPUT_SCHEMA`.

Unknown tokens MUST be treated as TRIGGER_INVALID.

Trigger selection MUST be deterministic:
- If the MANAGER block contains exactly one valid trigger token, proceed with that trigger.
- If the MANAGER block contains zero valid trigger tokens, the turn MUST terminate with terminal = ABEND (REASON_CODE=TRIGGER_INVALID).
- If the MANAGER block contains more than one valid trigger token, the turn MUST terminate with terminal = ABEND (REASON_CODE=TRIGGER_INVALID)
   and MUST list REQUIRED_TO_RESOLVE (e.g., "exactly one trigger token is required").

## 6. Deterministic Termination Requirement

For an activated turn, the system MUST select exactly one terminal outcome from the allowed set.

If obligations cannot be fulfilled within the allowed set, the system MUST escalate deterministically:

- First attempt: select a success terminal (PROPOSAL/COMMIT) only when all MUST obligations for that terminal are satisfiable given the provided inputs and required artifacts
- Otherwise: for COMMIT-type triggers, select UNRESOLVED only when explicitly permitted by DOC_ID `2PLT_40_EXECUTION_POLICY`; in all other cases select ABEND as the final fail-safe

UNRESOLVED is NOT a free choice; it is permitted only under conditions defined in policy
(DOC_ID `2PLT_40_EXECUTION_POLICY`).

## 7. Physical Responsibility Enforcement

If terminal = COMMIT, physical responsibility MUST be executed in accordance with DOC_ID `2PLT_10_RESPONSIBILITY`.

Failure to execute required physical responsibility constitutes policy violation and MUST escalate per
DOC_ID `2PLT_40_EXECUTION_POLICY`.

## 8. Proposal Artifact Requirement (Framework-Level Default)

In this framework, PROPOSAL is **not** a conversational draft. PROPOSAL is a **commit-capable artifact proposal**.

Therefore, a PROPOSAL MUST:

- Identify the `input_zip` (the authoritative input snapshot) being proposed against.
- Identify a concrete `patch_target` (file path(s) to be changed within the input_zip or its working copy).
- Provide a mechanically-applicable `diff/patch` (preferred: unified diff).

A PROPOSAL MUST return `ARTIFACT: INLINE`.

If any required element cannot be determined from the provided inputs, the model MUST return `ABEND`
with `REASON_CODE: INPUT_MISSING`, and MUST describe the missing items in free-text
(REQUIRED_TO_RESOLVE-style text is RECOMMENDED).

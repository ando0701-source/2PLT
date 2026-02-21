# 2PLT — Two-Phase Layered Trigger

2PLT (Two-Phase Layered Trigger) is a layered trigger framework with a two-phase flow (**PROPOSAL → COMMIT**) plus explicit terminal states (**UNRESOLVED / ABEND**).

Start here: [`docs/2PLT_00_ENTRYPOINT.md`](docs/2PLT_00_ENTRYPOINT.md)  
日本語: [`README.ja.md`](README.ja.md)

## Goals
- Predictable execution via explicit triggers
- Auditable, deterministic changes through a two-phase process
- Clear terminal states for exceptions and human intervention

## Core concepts
- **Phase**: PROPOSAL → COMMIT
- **Terminal states**: UNRESOLVED / ABEND
- **Trigger-driven**: e.g., `@@@@2PLT_...@@@@`

## Reading route
- `docs/2PLT_00_ENTRYPOINT.md` — deterministic entry procedure (no ZIP-wide scanning)
- `docs/INDEX.md` — curated "read next" route

## Status
This repository is currently used for experiments and verification.

## Note on "REJECT"
Some repository descriptions may mention `REJECT`. In the current normative documents in this repository, the terminal states are `UNRESOLVED` and `ABEND`. If `REJECT` is introduced later, it must be defined by an explicit profile/doc-set.

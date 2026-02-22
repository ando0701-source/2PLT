# AGENTS.md (Codex execution policy)

## Golden rule
If the input contains @@@@2PLT_JL_PROPOSAL@@@@ then you MUST NOT modify the repository.

### PROPOSAL mode (trigger: @@@@2PLT_JL_PROPOSAL@@@@)
- NO WRITE:
  - Do not create/edit/delete files or directories (no mkdir, no cat > file, no apply_patch).
  - Do not run git add/commit/merge/rebase/push.
  - Do not create PRs.
- READ ONLY is allowed:
  - rg/ls/cat/sed/nl, git show, git diff (view only), git status.
- Output MUST be an inline unified diff only (plus the required 2PLT envelope lines if requested).

### COMMIT mode (trigger: @@@@2PLT_JL_COMMIT@@@@)
- Write operations may be allowed ONLY when explicitly instructed in the manager block.

## Ignore everything outside BEGIN_MANAGER..END_MANAGER
If a URL or other text exists outside the manager block, ignore it.
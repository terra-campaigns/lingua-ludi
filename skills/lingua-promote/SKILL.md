---
name: lingua-promote
description: "Sanitise a closed Lingua Ludi session and stage a cross-language Tier 1 canon proposal."
license: CC-BY-4.0
argument-hint: "[campaign-slug] [session-id]"
---

# lingua-promote

Sanitise a closed session and stage a Tier 1 canon proposal for human review.

## Procedure

Execute the following steps in strict order.

### 1. Kit Root Discovery & Workspace Check

Resolve `$LINGUA_LUDI_ROOT` (or the `$HOME/.local/share/lingua-ludi` fallback) and verify `skills/`, `agents/`, `languages/`, and `templates/` exist (D5).
If `_gm-materials/lingua-ludi/<campaign>/` is missing, abort immediately with `Workspace not found; run lingua-init first.`
Resolve the target `<session_id>`: when supplied explicitly, verify `sessions/<session_id>.md` exists with non-empty `ended_at` (otherwise abort cleanly with `Session <session_id> not found or not yet closed.` and zero files modified); when omitted, default to the lexicographic maximum `session_id` among `sessions/*.md` files with non-empty `ended_at`, or abort cleanly with `No closed sessions found; finish and close a session with lingua-play first.` when none exist.

### 2. Pre-Flight Safety Checks

If `.active_session` exists, refuse staging and ask the user to close the active session cleanly via `lingua-play` first.
If `state.md` shows non-empty `pending_coach_turns`, warn the user (Coach updates are unaffected since the Promoter never reads `ledger.md`) and offer to proceed or drain first via `lingua-play`.

### 3. Sanitised Input Assembly (No `ledger.md`)

Read `sessions/<session_id>.md`, strip the session YAML front matter completely, drop every line starting with `> [OOC]`, and extract only the `### Turn <N>` narrative body.
Extract only the last `3` H2 `## Session <session_id>:` sections from `log.md`.
Determine `canonical_language` from `.agents/AGENTS.md` using the exact scan precedence, prose-to-BCP-47 mapping table, BCP-47 passthrough (`^[a-z]{2}(-[A-Z]{2})?$`), and single-consistent-tag ambiguity rule from `lingua-init` step 5, falling back to `campaign.yml` only if `.agents/AGENTS.md` is absent, silent, or ambiguous.
Read `.agents/lingua-ludi/glossary.<iso>.yml` if present.
If `.agents/context/catalog.json` exists, filter `entries` to `tier == 1` only (entry shape: `{"path": "directory/PlaceA.md", "title": "...", "summary": null, "sha": "a40893ba", "tier": 1}`), explicitly ignoring `tier: 2` `rolling_glossary.md` as an update target; otherwise mark catalog mode as standalone (`observed_source_sha: n/a (no catalog.json)`).
Never open or pass `ledger.md`, `ledger.md.corrupt-*`, or Coach recaps to the Promoter subagent.

### 4. Fresh `lingua-promoter` Invocation

If `_gm-materials/lingua-ludi/<campaign>/exports/<session_id>/tier1_proposal.md` already exists, prompt the user interactively to confirm overwriting before regenerating, aborting without modifying files if declined.
Otherwise spawn a fresh `lingua-promoter` subagent (`agents/lingua-promoter.md`) with the sanitised inputs and `templates/tier1_proposal.md` to write `_gm-materials/lingua-ludi/<campaign>/exports/<session_id>/tier1_proposal.md`.

### 5. Shell Meta-Leak Scan

When shell execution and either `rg` or `grep` are available, run the single fallback command below (preferring `rg -n`; the `grep -nE` branch uses GNU word boundaries and needs GNU-compatible grep).
Under Spec §3.6 rule 4, the only permitted writes to `exports/<session_id>/tier1_proposal.md` after promotion are rewriting `meta_leak_scan` and prepending the `> [!WARNING]` callout.
Immediately rewrite `meta_leak_scan: "pending"` in `tier1_proposal.md` to `pass` (zero matches) or `warn` (any match, prepending a `> [!WARNING]` block containing the verbatim phrase `WARNING: potential coaching meta-leak` and listing the matched lines while keeping the file).
When no shell execution tool exists or neither `rg` nor `grep` is available, record `meta_leak_scan: n/a (no shell)`.

```sh
if command -v rg >/dev/null 2>&1; then
  rg -n '(ledger\.md|\bCEFR\b|\bclean_uses\b|\bnext_due\b|\bpending_coach_turns\b|\berr-[a-z0-9-]+)' "<proposal_path>"
elif command -v grep >/dev/null 2>&1; then
  grep -nE '(ledger\.md|\bCEFR\b|\bclean_uses\b|\bnext_due\b|\bpending_coach_turns\b|\berr-[a-z0-9-]+)' "<proposal_path>"
else
  echo "n/a (no shell)"
fi
```

### 6. Read-Only Audit & Summary

Confirm zero modifications under `campaigns/`, `directory/`, `systems/`, `.agents/context/catalog.json`, and `.gitignore`.
Output the staged `tier1_proposal.md` path plus any meta-leak or glossary warnings.

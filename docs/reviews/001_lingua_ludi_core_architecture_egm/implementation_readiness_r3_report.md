# Stage 9 Implementation Readiness Report — Round 3 (`omp` / `mistral/mistral-large-latest` `high`)

**Target Plan:** [001_lingua_ludi_core_architecture_plan.md](../../plans/001_lingua_ludi_core_architecture_plan.md) (lines `1–356`, full file)
**Implements:** [001_lingua_ludi_core_architecture.md](../../designs/001_lingua_ludi_core_architecture.md) (Spec `0.2.2`, `main` at `d151e30`)
**Execution Mode:** 3 isolated zero-context `omp` (`v18.22.0`) processes using `mistral/mistral-large-latest` (`--thinking=high`, `--no-session --no-context-files --no-rules --no-skills --no-extensions --no-tools --no-lsp --no-pty --no-title`)
**Date:** 2026-11-27

## Reviewer Status

| Lens | Working Output (pruned per `AGENTS.md` §2) | Blockers | Majors | Nits |
| :--- | :--- | :---: | :---: | :---: |
| `egm-implementation-readiness` | `stage9_r3_implementation_readiness.md` | 0 | 5 | 11 |
| `egm-codebase-details` | `stage9_r3_codebase_details.md` | 0 | 0 | 3 |
| `egm-style` | `stage9_r3_style.md` | 0 | 5 | 8 |

## Overall Verdict

Zero Blockers were found by any Round 3 reviewer, and `egm-codebase-details` reported **0 Blockers and 0 Majors** (`Satisfies Standard` with `3` polish Nits).
All Round 2 Blocker (`current_turn` counter space), `wingman.policy` intake, Tier 2 Step 3 recovery, `tier1_proposal.md` item schema, and full issue-URL fixes verified closed.
However, `egm-implementation-readiness` and `egm-style` surfaced **9 deduplicated Majors** (`S9R3-M1`–`S9R3-M9`, mostly tight contract edge cases introduced or exposed by the deeper Round 2 pins) plus **18 Nits** (`S9R3-N1`–`S9R3-N18`), so resolving those `9` Majors before final plan sign-off is recommended.

## Deduplicated Action Items

### Blockers (`0`)

None.

### Majors (`9`)

1.  **S9R3-M1 (`ir` M1) — `exports/` write-ownership carve-out not stated explicitly:**
    Target File 14 step 5 instructs the `lingua-promote` skill to rewrite `meta_leak_scan` and prepend `> [!WARNING]` inside `tier1_proposal.md` per Spec §3.6 rule 4, while Spec §3.3 and Invariant 4 state only `lingua-promoter` writes `exports/`.
    Pin one explicit sentence in `D4`, `D13`, and Target File 14 step 5 noting that `lingua-promote`'s sole permitted post-promotion edits under `exports/<session_id>/tier1_proposal.md` are the `meta_leak_scan` header rewrite and the prepended `> [!WARNING]` callout per Spec §3.6 rule 4, while all other proposal content is written solely by `lingua-promoter`.

2.  **S9R3-M2 (`ir` M2, `style` #3) — Harness capability detection signals `(a)`/`(b)`/`(c)` lack recognition rules and uncertainty fallback:**
    Target File 13 step 2 instructs inspecting "active session tool declarations" for `(a)`/`(b)`/`(c)` without saying how the GM evaluates its own tool list or what happens if `(a)` or `(b)` detection is uncertain.
    Pin that the GM evaluates its own harness-provided tool list against observable functional roles (subagent creation/delegation tool for `(a)`, message/follow-up tool addressed to a persistent worker ID for `(b)`, async completion event or task-status/wait tool for `(c)`), and pin that uncertainty on `(a)` or `(b)` treats that signal as absent and aborts cleanly with the `https://github.com/terra-campaigns/lingua-ludi/issues/3` refusal.

3.  **S9R3-M3 (`ir` M3) — Crash-recovery fresh `ledger.md` token substitution source undefined:**
    Target File 10 mode 3 instructs `lingua-coach` to write a fresh header-only `ledger.md` from `templates/ledger.md` when rebuild fails, without specifying where `lingua-coach` reads `{{CAMPAIGN_ID}}` and `{{TARGET_LANGUAGE}}` or what `last_updated_turn` records after replay vs reset.
    Pin that `lingua-coach` reads `campaign.yml` (permitted per Spec §3.3) to substitute `{{CAMPAIGN_ID}}` and `{{TARGET_LANGUAGE}}`, sets `last_updated_turn` to the highest replayed monotonic turn on successful rebuild, and sets `last_updated_turn` to `0` (or `state.md` `current_turn`) when initialising a fresh header-only `ledger.md` on unrecoverable corruption.

4.  **S9R3-M4 (`ir` M4) — Standalone no-`catalog.json` note-search fallback omitted from `lingua-play`:**
    Target File 13 step 1 loads `.agents/context/catalog.json` "if present", without specifying Spec §2.5 / CUJ 5's fallback for reading local campaign Markdown notes directly when `catalog.json` is absent.
    Pin in Target File 13 step 1 that when `.agents/context/catalog.json` is absent, the GM locates scene and NPC Markdown notes by searching `campaigns/`, `directory/`, and `systems/` directly.

5.  **S9R3-M5 (`ir` M5) — Provenance trailer sources and `validate_provenance.py` invocation undefined:**
    The Phases preamble references `validate_provenance.py` and `Model-Author` / `Agent-Role` / `Co-authored-by` without pinning how `validate_provenance.py` is invoked or which runtime fields supply each trailer.
    Pin the exact `python3 "$ARTIFICE_ROOT/scripts/validate_provenance.py"` command (checking the prepared commit message file or commit range per `AGENTS.md` §4) and clarify how each trailer is sourced (`Model-Author` from active runtime model identity or literal `unavailable`, `Agent-Role` from the active persona/executor role, `Co-authored-by` from `git config` / known contributor identity).

6.  **S9R3-M6 (`style` #1, `ir` N6) — Undefined `canonical_language` user-override path and matching regex specificity:**
    Target File 12 step 5 references "a user-supplied override" that triggers a mismatch warning, yet step 2 defines no override argument or prompt (`argument-hint` has three positional arguments); meanwhile, exact keyword case sensitivity and key/heading matching regexes for categories `(1)`–`(3)` are prose-described.
    Remove the phantom override mismatch clause from Target File 12 step 5 (matching Spec §2.1 CUJ 1, where `.agents/AGENTS.md` is authoritative and step 5 prompts interactively for fallback `canonical_language` only when absent, silent, or ambiguous), and tighten the case-insensitive key/heading matchers for categories `(1)`–`(3)`.

7.  **S9R3-M7 (`style` #2) — Cross-category conflict semantics in `lingua-init` step 5 contradict fall-through:**
    Target File 12 step 5 says "letting the first category that produces a single consistent tag win", which implies falling through on conflict, while the next sentence treats "conflicting tags within that winning category" as overall ambiguity triggering fallback.
    Pin unambiguously that categories `(1)` → `(2)` → `(3)` are checked sequentially, a category yielding zero tags advances to the next category, and conflicting tags within the first non-empty category immediately halt matching as ambiguous and trigger the interactive fallback prompt.

8.  **S9R3-M8 (`style` #4) — Validation failure abort behaviour omitted for `campaign_id` and `cefr_level`:**
    Target File 12 steps 2–3 state validation of `campaign_id` (`[a-z0-9_-]`) and `cefr_level` (`cefr_levels` array membership) without pinning clean abort on failure.
    Pin that failing either check aborts immediately with an explicit validation error message (listing valid `cefr_levels` for step 3) and zero files written.

9.  **S9R3-M9 (`style` #5) — Undefined abort when explicitly supplied `<session_id>` is missing or unclosed:**
    Target File 14 step 1 defines omitted-session lookup and empty-set abort (`No closed sessions found...`), but omits failure behaviour when an explicit `<session_id>` argument has no matching `sessions/<session_id>.md` file or still has empty `ended_at`.
    Pin an explicit abort with `Session <session_id> not found or not yet closed.` and zero files modified.

### Nits (`18`)

1.  **S9R3-N1 (`ir` N1):** Pin in Target File 10 that subsequent clean uses at `interval = 16` retain `interval = 16` (`next_due = current_turn + 16`).
2.  **S9R3-N2 (`ir` N2):** Pin slug normalisation from `surface_form` in Target File 10 (lowercase ASCII transliteration where applicable, non-alphanumeric runs collapsed to single hyphens `-`, leading/trailing hyphens trimmed).
3.  **S9R3-N3 (`ir` N3, `cd` Nit 1):** Combine D13 / Target File 14 step 5's `sh` snippet into a single `command -v rg >/dev/null 2>&1 && rg -n ... || grep -nE ...` probe and guard so `command -v` checks shell availability for `n/a (no shell)`.
4.  **S9R3-N4 (`ir` N4):** Specify in Target File 14 step 3 that `log.md` last-3 extraction splits on `## Session <session_id>:` headings.
5.  **S9R3-N5 (`ir` N7):** Pin in Target File 13 step 1 prompting interactively for `[campaign-slug]` if omitted, aborting cleanly if `state.md` is missing/malformed, and validating `max_active_npcs` (`1..2`), `max_retrieval_targets` (`0..5`), and `cefr_level` membership against the pack.
6.  **S9R3-N6 (`ir` N8):** Close the D16 US-spelling identifier exclusion note to `catalog.json` and inline/fenced code spans only.
7.  **S9R3-N7 (`ir` N9):** Clarify in Phase 3 exit gate that `mktemp -d` dry-run checks execute the three skills inside the harness under test (or record harness-simulated walkthrough steps with exact file diffs).
8.  **S9R3-N8 (`ir` N10):** Pin mid-session `lingua-coach` `status: error` ack handling in Target File 13 turn loop (log warning locally and record the turn in `pending_coach_turns` on close rather than blocking play).
9.  **S9R3-N9 (`cd` Nit 2):** Pin verbatim error/warning strings in Target Files 12–13 (`missing pack` list, `non-Git` warning, `crashed-session` takeover prompt) for exact `V3`/`V4` assertions.
10. **S9R3-N10 (`cd` Nit 3, `ir` N11, `ir` N5):** Explicitly note that Target File 6 commented glossary examples cover all four Spec §4.3 fields (`canonical_term`, `target_rendering`, `policy`, `related_catalog_path`) and that Dutch prose examples, template HTML comments, and `README.md` registration wording are intentional content authoring latitude within `AGENTS.md` §3.
11. **S9R3-N11 (`style` #6):** Recalibrate line 31 wording once `S9R3-M1`–`S9R3-M9` are resolved.
12. **S9R3-N12 (`style` #7):** Align `wingman.name` / `wingman.role` default phrasing (`Companion A` / `Local fixer and colleague` vs starter literal defaults `"Companion"` / `"Local colleague and fixer"`).
13. **S9R3-N13 (`style` #8):** Add the `[ ... ]` (`^\[[^\[\]]*\]$`) OOC test case explicitly to `V4(c)`.
14. **S9R3-N14 (`style` #9):** Pin simple key/list shapes for `updated_items` (comma-separated modified `item_id` list) and `due_targets_preview` (next up to `4` due `item_id: surface_form` items) in `D12` and Target File 10.
15. **S9R3-N15 (`style` #10):** Scope Phase 3 exit gate to dry-run tests `V3`–`V7` plus repo-level `V2` verification separately so `mktemp -d` wording stays exact.
16. **S9R3-N16 (`style` #11):** Specify that fixture `languages/xx/pack.md` uses D14's minimal front-matter keys and H2 headings (`## CEFR Profiles`, `## Error Taxonomy Notes`, `## Wingman Scaffolding Cues`).
17. **S9R3-N17 (`style` #12):** Frame `.gitignore` baseline content in line 24 (`__pycache__/`, `.venv/`) explicitly as verified on `main` at commit `d151e30` on the authoring host.
18. **S9R3-N18 (`style` `L304`):** Close the parenthesis properly in Target File 14 step 2 (`warn the user (Coach updates are unaffected since Promoter never reads \`ledger.md\`) and offer to proceed or drain first via \`lingua-play\``).

## Traceability Matrix

| Raw ID | Lens File | Severity (Raw) | Deduplicated ID | Disposition |
| :--- | :--- | :---: | :--- | :--- |
| `ir3` M1 | `stage9_r3_implementation_readiness.md` | Major | `S9R3-M1` | Unique Major kept |
| `ir3` M2 | `stage9_r3_implementation_readiness.md` | Major | `S9R3-M2` | Merged with `style3` #3 |
| `ir3` M3 | `stage9_r3_implementation_readiness.md` | Major | `S9R3-M3` | Unique Major kept |
| `ir3` M4 | `stage9_r3_implementation_readiness.md` | Major | `S9R3-M4` | Unique Major kept |
| `ir3` M5 | `stage9_r3_implementation_readiness.md` | Major | `S9R3-M5` | Unique Major kept |
| `ir3` N1 | `stage9_r3_implementation_readiness.md` | Nit | `S9R3-N1` | Kept as Nit |
| `ir3` N2 | `stage9_r3_implementation_readiness.md` | Nit | `S9R3-N2` | Kept as Nit |
| `ir3` N3 | `stage9_r3_implementation_readiness.md` | Nit | `S9R3-N3` | Merged with `cd3` Nit 1 |
| `ir3` N4 | `stage9_r3_implementation_readiness.md` | Nit | `S9R3-N4` | Kept as Nit |
| `ir3` N5 | `stage9_r3_implementation_readiness.md` | Nit | `S9R3-N10` | Grouped with prose-latitude Nits |
| `ir3` N6 | `stage9_r3_implementation_readiness.md` | Nit | `S9R3-M6` | Folded into `S9R3-M6` |
| `ir3` N7 | `stage9_r3_implementation_readiness.md` | Nit | `S9R3-N5` | Kept as Nit |
| `ir3` N8 | `stage9_r3_implementation_readiness.md` | Nit | `S9R3-N6` | Kept as Nit |
| `ir3` N9 | `stage9_r3_implementation_readiness.md` | Nit | `S9R3-N7` | Kept as Nit |
| `ir3` N10 | `stage9_r3_implementation_readiness.md` | Nit | `S9R3-N8` | Kept as Nit |
| `ir3` N11 | `stage9_r3_implementation_readiness.md` | Nit | `S9R3-N10` | Grouped with prose-latitude Nits |
| `cd3` Nit 1 | `stage9_r3_codebase_details.md` | Nit | `S9R3-N3` | Merged with `ir3` N3 |
| `cd3` Nit 2 | `stage9_r3_codebase_details.md` | Nit | `S9R3-N9` | Kept as Nit |
| `cd3` Nit 3 | `stage9_r3_codebase_details.md` | Nit | `S9R3-N10` | Grouped with prose-latitude Nits |
| `style3` #1 | `stage9_r3_style.md` | Major | `S9R3-M6` | Merged with `ir3` N6 |
| `style3` #2 | `stage9_r3_style.md` | Major | `S9R3-M7` | Unique Major kept |
| `style3` #3 | `stage9_r3_style.md` | Major | `S9R3-M2` | Merged with `ir3` M2 |
| `style3` #4 | `stage9_r3_style.md` | Major | `S9R3-M8` | Unique Major kept |
| `style3` #5 | `stage9_r3_style.md` | Major | `S9R3-M9` | Unique Major kept |
| `style3` #6 | `stage9_r3_style.md` | Nit | `S9R3-N11` | Kept as Nit |
| `style3` #7 | `stage9_r3_style.md` | Nit | `S9R3-N12` | Kept as Nit |
| `style3` #8 | `stage9_r3_style.md` | Nit | `S9R3-N13` | Kept as Nit |
| `style3` #9 | `stage9_r3_style.md` | Nit | `S9R3-N14` | Kept as Nit |
| `style3` #10 | `stage9_r3_style.md` | Nit | `S9R3-N15` | Kept as Nit |
| `style3` #11 | `stage9_r3_style.md` | Nit | `S9R3-N16` | Kept as Nit |
| `style3` #12 | `stage9_r3_style.md` | Nit | `S9R3-N17` | Kept as Nit |
| `style3` `L304` | `stage9_r3_style.md` | Nit | `S9R3-N18` | Kept as Nit |

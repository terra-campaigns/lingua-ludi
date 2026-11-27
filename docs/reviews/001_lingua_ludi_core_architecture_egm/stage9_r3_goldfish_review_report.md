# Goldfish Annotated Review Report

**Target:** `docs/plans/001_lingua_ludi_core_architecture_plan.md` (`lines 1–356`, full file)
**Source Report:** [implementation_readiness_r3_report.md](implementation_readiness_r3_report.md)
**Status:** resolved

## Prioritized Findings & AI Suggestions

### [Issue 1 — S9R3-M1] [`egm-implementation-readiness`] - Severity: Major

*   **Affected Scope:** [Line 38](../../plans/001_lingua_ludi_core_architecture_plan.md#L38), [Line 47](../../plans/001_lingua_ludi_core_architecture_plan.md#L47), and [Line 313](../../plans/001_lingua_ludi_core_architecture_plan.md#L313) (`D4`, `D13`, Target File 14 step 5)
*   **Discrepancy Description:** Target File 14 step 5 instructs the `lingua-promote` skill to rewrite `meta_leak_scan` and prepend `> [!WARNING]` inside `tier1_proposal.md` per Spec §3.6 rule 4, while Spec §3.3 and Invariant 4 state that only `lingua-promoter` writes `exports/`.
*   **Rationale:** Without explicitly recording the Spec §3.6 rule 4 carve-out in the plan, an implementer applying line 11's stop-and-raise rule could treat `lingua-promote` step 5 as an ownership contradiction.
*   ***AI Suggestion:*** PROPOSE FIX
*   ***Justification:*** Stating the narrow Spec §3.6 rule 4 post-promotion exception across `D4`, `D13`, and Target File 14 step 5 aligns the plan cleanly with the approved specification.
*   ***Proposed Edit:***
    *   Location: `D4` ([Line 38](../../plans/001_lingua_ludi_core_architecture_plan.md#L38)), `D13` ([Line 47](../../plans/001_lingua_ludi_core_architecture_plan.md#L47)), and Target File 14 step 5 ([Line 313](../../plans/001_lingua_ludi_core_architecture_plan.md#L313))
    *   Change: Pin one explicit sentence noting that `lingua-promote`'s sole permitted post-promotion edits under `exports/<session_id>/tier1_proposal.md` are rewriting `meta_leak_scan` and prepending the `> [!WARNING]` callout per Spec §3.6 rule 4, while all other proposal content is written solely by `lingua-promoter`.

### [Issue 2 — S9R3-M2] [`egm-implementation-readiness`, `egm-style`] - Severity: Major

*   **Affected Scope:** [Lines 264–266](../../plans/001_lingua_ludi_core_architecture_plan.md#L264-L266) (Target File 13 step 2)
*   **Discrepancy Description:** Target File 13 step 2 instructs inspecting "active session tool declarations" for `(a)`, `(b)`, and `(c)` without stating how the GM evaluates its own tool list or what happens when uncertainty arises on `(a)` or `(b)`.
*   **Rationale:** Only signal `(c)` specifies an uncertainty fallback, leaving `(a)` or `(b)` ambiguity undecided between refusal and proceeding without isolated subagents.
*   ***AI Suggestion:*** PROPOSE FIX
*   ***Justification:*** Pinning functional tool-role recognition criteria on the GM's active tool list and classifying uncertainty on `(a)` or `(b)` as absent keeps tier detection harness-neutral (`AGENTS.md` §3) and fails safe.
*   ***Proposed Edit:***
    *   Location: Target File 13 step 2 ([Lines 264–266](../../plans/001_lingua_ludi_core_architecture_plan.md#L264-L266))
    *   Change: Pin that the GM evaluates its own active harness tool list against observable functional roles (`(a)` subagent creation or delegation tool, `(b)` follow-up messaging tool addressed to a persistent worker or conversation ID, `(c)` asynchronous completion notification or background task status/await tool), and pin that uncertainty on `(a)` or `(b)` treats that signal as absent and triggers the clean refusal citing `https://github.com/terra-campaigns/lingua-ludi/issues/3`.

### [Issue 3 — S9R3-M3] [`egm-implementation-readiness`] - Severity: Major

*   **Affected Scope:** [Line 217](../../plans/001_lingua_ludi_core_architecture_plan.md#L217) (Target File 10 mode 3)
*   **Discrepancy Description:** Target File 10 mode 3 tells `lingua-coach` to write a fresh header-only `ledger.md` from `templates/ledger.md` when crash recovery fails, without specifying where `lingua-coach` reads `{{CAMPAIGN_ID}}` and `{{TARGET_LANGUAGE}}` or what `last_updated_turn` records after replay versus reset.
*   **Rationale:** `templates/ledger.md` uses D6 placeholders, and D6 forbids leaving unresolved `{{...}}` tokens in instantiated files.
*   ***AI Suggestion:*** PROPOSE FIX
*   ***Justification:*** Spec §3.3 grants `lingua-coach` read access to `campaign.yml`, so referencing `campaign.yml` for token substitution and pinning `last_updated_turn` closes both gaps without widening permissions.
*   ***Proposed Edit:***
    *   Location: Target File 10 mode 3 ([Line 217](../../plans/001_lingua_ludi_core_architecture_plan.md#L217))
    *   Change: Pin that `lingua-coach` reads `campaign.yml` (permitted per Spec §3.3) to substitute `{{CAMPAIGN_ID}}` and `{{TARGET_LANGUAGE}}` when instantiating a fresh `ledger.md` from `templates/ledger.md`, sets `last_updated_turn` to the highest replayed monotonic turn on successful rebuild, and sets `last_updated_turn: 0` when writing a fresh header-only `ledger.md` after unrecoverable corruption.

### [Issue 4 — S9R3-M4] [`egm-implementation-readiness`] - Severity: Major

*   **Affected Scope:** [Line 263](../../plans/001_lingua_ludi_core_architecture_plan.md#L263) (Target File 13 step 1)
*   **Discrepancy Description:** Target File 13 step 1 loads `.agents/context/catalog.json` only when present, without specifying Spec §2.5 / CUJ 5's standalone fallback for reading local campaign Markdown notes when `catalog.json` is absent.
*   **Rationale:** Implementers following only Target File 13 could leave standalone non-`oude-mechanismen` vaults without scene or NPC note lookup.
*   ***AI Suggestion:*** PROPOSE FIX
*   ***Justification:*** One sentence directly encodes CUJ 5's existing standalone vault rule into Target File 13 step 1.
*   ***Proposed Edit:***
    *   Location: Target File 13 step 1 ([Line 263](../../plans/001_lingua_ludi_core_architecture_plan.md#L263))
    *   Change: Pin in Target File 13 step 1 that when `.agents/context/catalog.json` is absent, the GM locates relevant scene and NPC Markdown notes by searching `campaigns/`, `directory/`, and `systems/` directly.

### [Issue 5 — S9R3-M5] [`egm-implementation-readiness`] - Severity: Major

*   **Affected Scope:** [Lines 79–80](../../plans/001_lingua_ludi_core_architecture_plan.md#L79-L80) (Phases preamble)
*   **Discrepancy Description:** The Phases preamble mandates `validate_provenance.py` and three trailers without specifying how `ARTIFICE_ROOT` is resolved, which CLI flags `validate_provenance.py` takes, or which runtime fields supply each trailer.
*   **Rationale:** `AGENTS.md` §4 names the checker and trailers without showing the invocation contract (`--message-file`), which risks commit-gate trial and error.
*   ***AI Suggestion:*** PROPOSE FIX
*   ***Justification:*** Recording the exact `validate_provenance.py --message-file` command and workspace provenance mapping eliminates commit-time guesswork.
*   ***Proposed Edit:***
    *   Location: Phases preamble ([Lines 79–80](../../plans/001_lingua_ludi_core_architecture_plan.md#L79-L80))
    *   Change: Pin resolving `ARTIFICE_ROOT` (`$ARTIFICE_ROOT`, else `$HOME/.local/share/artifice`, else `$HOME/Repos/artifice`), writing a commit message file with `Model-Author` (actual runtime model identifier or literal `unavailable`), `Agent-Role` (actual executing role), and `Co-authored-by` (known contributor identity), running pre-commit `python3 "$ARTIFICE_ROOT/scripts/validate_provenance.py" --message-file <msg_file>`, and committing via `python3 "$ARTIFICE_ROOT/scripts/github_app_auth.py" exec -- git commit -F <msg_file>`.

### [Issue 6 — S9R3-M6] [`egm-style`, `egm-implementation-readiness`] - Severity: Major

*   **Affected Scope:** [Lines 247–249](../../plans/001_lingua_ludi_core_architecture_plan.md#L247-L249) (Target File 12 step 5)
*   **Discrepancy Description:** Target File 12 step 5 references "a user-supplied override" triggering a mismatch warning, yet step 2 defines no override argument or prompt; meanwhile, category `(1)`–`(3)` matching syntax (case sensitivity and YAML/heading forms) stays prose-described.
*   **Rationale:** Referencing a non-existent input path forces implementers either to invent an unlisted flag or leave dead logic, while imprecise category patterns risk scan drift.
*   ***AI Suggestion:*** PROPOSE FIX
*   ***Justification:*** Removing the phantom user-override clause aligns Target File 12 strictly with Spec §2.1 CUJ 1, and tightening case-insensitive matchers makes categories `(1)`–`(3)` explicit.
*   ***Proposed Edit:***
    *   Location: Target File 12 step 5 ([Lines 247–249](../../plans/001_lingua_ludi_core_architecture_plan.md#L247-L249))
    *   Change: Remove the `"if both .agents/AGENTS.md and a user-supplied override disagree..."` clause so `.agents/AGENTS.md` is authoritative and step 5 prompts interactively for fallback `canonical_language` only when absent, silent, or ambiguous (matching Spec §2.1 CUJ 1), and pin case-insensitive matching across categories `(1)` (`canonical_language:` YAML/key or `canonical language` heading), `(2)` (writing-style language rule or heading matching `British English`, `UK English`, `American English`, `US English`, or `write in`/`use <Language>`), and `(3)` (chapter or session output language heading/line).

### [Issue 7 — S9R3-M7] [`egm-style`] - Severity: Major

*   **Affected Scope:** [Lines 248–249](../../plans/001_lingua_ludi_core_architecture_plan.md#L248-L249) (Target File 12 step 5)
*   **Discrepancy Description:** Target File 12 step 5 states "letting the first category that produces a single consistent tag win", which implies falling through on conflict, while the next sentence treats "conflicting tags within that winning category" as overall ambiguity.
*   **Rationale:** Contradictory conflict-handling sentences force the implementer to choose between fall-through and fail-to-prompt on contradictory headings.
*   ***AI Suggestion:*** PROPOSE FIX
*   ***Justification:*** Pinning sequential zero-match fall-through and immediate ambiguity halt on multiple conflicting tags inside the first non-empty category removes the contradiction.
*   ***Proposed Edit:***
    *   Location: Target File 12 step 5 ([Lines 248–249](../../plans/001_lingua_ludi_core_architecture_plan.md#L248-L249))
    *   Change: Pin unambiguously that categories `(1)` → `(2)` → `(3)` are evaluated sequentially: a category yielding zero recognised tags advances to the next category, and the first category yielding one or more recognised tags stops scanning (a single consistent tag wins, whereas two or more distinct tags inside that category immediately mark `.agents/AGENTS.md` ambiguous and trigger the interactive fallback prompt).

### [Issue 8 — S9R3-M8] [`egm-style`] - Severity: Major

*   **Affected Scope:** [Lines 241–243](../../plans/001_lingua_ludi_core_architecture_plan.md#L241-L243) (Target File 12 steps 2–3)
*   **Discrepancy Description:** Target File 12 steps 2–3 state validating `campaign_id` against `[a-z0-9_-]` and `cefr_level` against `cefr_levels` without specifying abort behaviour on validation failure.
*   **Rationale:** Contrast step 3's explicit abort on a missing pack file; unpinned validation failures leave retry versus hard-abort unspecified.
*   ***AI Suggestion:*** PROPOSE FIX
*   ***Justification:*** Failing closed with zero files written keeps workspace initialisation consistent across all validation checks.
*   ***Proposed Edit:***
    *   Location: Target File 12 steps 2–3 ([Lines 241–L243](../../plans/001_lingua_ludi_core_architecture_plan.md#L241-L243))
    *   Change: Pin explicit clean aborts with zero files written when `campaign_id` fails `^[a-z0-9_-]+$` (printing the allowed pattern) or when requested `cefr_level` is absent from the pack's `cefr_levels` list (printing the pack's supported levels).

### [Issue 9 — S9R3-M9] [`egm-style`] - Severity: Major

*   **Affected Scope:** [Line 302](../../plans/001_lingua_ludi_core_architecture_plan.md#L302) (Target File 14 step 1)
*   **Discrepancy Description:** Target File 14 step 1 defines omitted-argument session selection and empty-set abort, without defining failure behaviour when an explicitly supplied `<session_id>` is missing or still unclosed.
*   **Rationale:** Explicit invalid `<session_id>` arguments would otherwise fall into ad-hoc error handling or attempt to promote an open session.
*   ***AI Suggestion:*** PROPOSE FIX
*   ***Justification:*** One abort clause handles both missing files and empty `ended_at` cleanly.
*   ***Proposed Edit:***
    *   Location: Target File 14 step 1 ([Line 302](../../plans/001_lingua_ludi_core_architecture_plan.md#L302))
    *   Change: Pin that when an explicit `<session_id>` has no matching `sessions/<session_id>.md` file or its front-matter `ended_at` is empty, `lingua-promote` aborts cleanly with `Session <session_id> not found or not yet closed.` and modifies zero files.

### [Issue 10 — S9R3-N1] [`egm-implementation-readiness`] - Severity: Nit

*   **Affected Scope:** [Line 218](../../plans/001_lingua_ludi_core_architecture_plan.md#L218) (Target File 10)
*   **Discrepancy Description:** Target File 10 describes interval progression `1 → 2 → 4 → 8 → 16` without explicitly stating that subsequent clean uses at `16` stay capped at `16`.
*   **Rationale:** One clause removes any off-ladder overflow question at the top interval step.
*   ***AI Suggestion:*** PROPOSE FIX
*   ***Justification:*** Minimal one-phrase precision matching Spec §3.5.
*   ***Proposed Edit:***
    *   Location: Target File 10 ([Line 218](../../plans/001_lingua_ludi_core_architecture_plan.md#L218))
    *   Change: Add `, keeping subsequent clean uses capped at interval = 16 (next_due = current_turn + 16)`.

### [Issue 11 — S9R3-N2] [`egm-implementation-readiness`] - Severity: Nit

*   **Affected Scope:** [Line 218](../../plans/001_lingua_ludi_core_architecture_plan.md#L218) (Target File 10)
*   **Discrepancy Description:** Target File 10 specifies lowercase alphanumeric/hyphen slugs without defining how `surface_form` text is normalised into `<slug>`.
*   **Rationale:** A brief normalisation rule prevents duplicate `item_id` rows from minor punctuation or case variations.
*   ***AI Suggestion:*** PROPOSE FIX
*   ***Justification:*** Deterministic ASCII-lowercase hyphenation keeps `item_id` slugs uniform.
*   ***Proposed Edit:***
    *   Location: Target File 10 ([Line 218](../../plans/001_lingua_ludi_core_architecture_plan.md#L218))
    *   Change: Pin slug derivation from `surface_form`: ASCII-lowercase transliteration where applicable, replacing non-alphanumeric runs with single `-` hyphens and trimming leading/trailing hyphens.

### [Issue 12 — S9R3-N3] [`egm-implementation-readiness`, `egm-codebase-details`] - Severity: Nit

*   **Affected Scope:** [Line 47](../../plans/001_lingua_ludi_core_architecture_plan.md#L47) and [Lines 313–318](../../plans/001_lingua_ludi_core_architecture_plan.md#L313-L318) (`D13`, Target File 14 step 5)
*   **Discrepancy Description:** Target File 14 step 5's `sh` block places `rg -n` and `grep -nE` on two standalone lines instead of a single conditional fallback, and omits the shell/tool availability check driving `n/a (no shell)`.
*   **Rationale:** Running both standalone lines executes the scan twice; a single `command -v rg ... || grep ...` pipeline avoids redundant execution and pins tool detection.
*   ***AI Suggestion:*** PROPOSE FIX
*   ***Justification:*** Standard POSIX fallback command handles both tools and `n/a (no shell)` cleanly.
*   ***Proposed Edit:***
    *   Location: `D13` ([Line 47](../../plans/001_lingua_ludi_core_architecture_plan.md#L47)) and Target File 14 step 5 ([Lines 313–318](../../plans/001_lingua_ludi_core_architecture_plan.md#L313-L318))
    *   Change: Combine the fenced `sh` block into `if command -v rg >/dev/null 2>&1; then rg -n '<pattern>' "<proposal_path>"; elif command -v grep >/dev/null 2>&1; then grep -nE '<pattern>' "<proposal_path>"; else echo "n/a (no shell)"; fi`, and state that missing shell tool capability records `meta_leak_scan: n/a (no shell)`.

### [Issue 13 — S9R3-N4] [`egm-implementation-readiness`] - Severity: Nit

*   **Affected Scope:** [Line 307](../../plans/001_lingua_ludi_core_architecture_plan.md#L307) (Target File 14 step 3)
*   **Discrepancy Description:** Target File 14 step 3 requires extracting the last `3` session entries from `log.md` without specifying the heading delimiter used to split entries.
*   **Rationale:** Naming `## Session <session_id>:` matches Target File 3 (`templates/log.md`) directly.
*   ***AI Suggestion:*** PROPOSE FIX
*   ***Justification:*** One phrase eliminates delimiter guesswork during `log.md` slicing.
*   ***Proposed Edit:***
    *   Location: Target File 14 step 3 ([Line 307](../../plans/001_lingua_ludi_core_architecture_plan.md#L307))
    *   Change: Clarify extracting the last `3` H2 `## Session <session_id>:` sections from `log.md`.

### [Issue 14 — S9R3-N5] [`egm-implementation-readiness`] - Severity: Nit

*   **Affected Scope:** [Lines 262–263](../../plans/001_lingua_ludi_core_architecture_plan.md#L262-L263) (Target File 13 step 1)
*   **Discrepancy Description:** Target File 13 step 1 omits omitted-slug prompting for `[campaign-slug]`, clean abort wording if `state.md` is missing or malformed, and load-time config bounds validation.
*   **Rationale:** Checking `state.md` and clamping/validating `max_active_npcs` (`1..2`), `max_retrieval_targets` (`0..5`), and `cefr_level` on load prevents subtle runtime misconfiguration.
*   ***AI Suggestion:*** PROPOSE FIX
*   ***Justification:*** Short defensive load checks keep pre-flight comprehensive without adding design choices.
*   ***Proposed Edit:***
    *   Location: Target File 13 step 1 ([Lines 262–263](../../plans/001_lingua_ludi_core_architecture_plan.md#L262-L263))
    *   Change: Pin prompting interactively for `campaign_id` when `[campaign-slug]` is omitted, aborting cleanly with zero files modified if `state.md` is missing or malformed, and validating `max_active_npcs` (`1..2`), `max_retrieval_targets` (`0..5`), and `cefr_level` pack membership on load.

### [Issue 15 — S9R3-N6] [`egm-implementation-readiness`] - Severity: Nit

*   **Affected Scope:** [Line 50](../../plans/001_lingua_ludi_core_architecture_plan.md#L50) (`D16`)
*   **Discrepancy Description:** `D16` says `exact identifiers such as catalog.json`, leaving the exclusion set phrased as an open-ended example.
*   **Rationale:** Closing the exclusion list strictly to inline/fenced code spans and the exact filename `catalog.json` keeps verification deterministic.
*   ***AI Suggestion:*** PROPOSE FIX
*   ***Justification:*** Tightens `D16` and `V1` wording in one phrase.
*   ***Proposed Edit:***
    *   Location: `D16` ([Line 50](../../plans/001_lingua_ludi_core_architecture_plan.md#L50))
    *   Change: Replace `outside inline code spans, fenced blocks, and exact identifiers such as catalog.json` with `outside inline code spans, fenced blocks, and the exact filename catalog.json`.

### [Issue 16 — S9R3-N7] [`egm-implementation-readiness`] - Severity: Nit

*   **Affected Scope:** [Line 102](../../plans/001_lingua_ludi_core_architecture_plan.md#L102) (Phase 3 exit gate)
*   **Discrepancy Description:** Phase 3 exit gate states executing the dry-run matrix without noting whether it runs inside the active harness against the disposable `mktemp -d` workspace and kit copy.
*   **Rationale:** Specifying live skill execution inside the active harness against the temporary directories makes Phase 3 verification reproducible.
*   ***AI Suggestion:*** PROPOSE FIX
*   ***Justification:*** Simple procedural clarification matching Spec §8.3.
*   ***Proposed Edit:***
    *   Location: Phase 3 exit gate ([Line 102](../../plans/001_lingua_ludi_core_architecture_plan.md#L102))
    *   Change: Clarify that dry runs invoke the authored skills directly inside the active subagent-capable harness against the `mktemp -d` workspace and kit copy, recording observed files and outputs as PR evidence.

### [Issue 17 — S9R3-N8] [`egm-implementation-readiness`] - Severity: Nit

*   **Affected Scope:** [Line 288](../../plans/001_lingua_ludi_core_architecture_plan.md#L288) (Target File 13 turn loop)
*   **Discrepancy Description:** Only startup Step 3 pins `status: error` handling for `lingua-coach`, leaving mid-session per-turn `status: error` ack behaviour unstated.
*   **Rationale:** Queuing failed per-turn updates into `pending_coach_turns` on close keeps play smooth without dropping learner turns.
*   ***AI Suggestion:*** PROPOSE FIX
*   ***Justification:*** Consistent with Spec §3.2 bounded close-drain resilience.
*   ***Proposed Edit:***
    *   Location: Target File 13 turn loop ([Line 288](../../plans/001_lingua_ludi_core_architecture_plan.md#L288))
    *   Change: Pin that if `lingua-coach` returns `status: error` mid-session, the GM logs a non-blocking warning internally without interrupting fiction and appends that turn into `pending_coach_turns` at session close for startup Step 3 recovery/drain.

### [Issue 18 — S9R3-N9] [`egm-codebase-details`] - Severity: Nit

*   **Affected Scope:** [Lines 243–246](../../plans/001_lingua_ludi_core_architecture_plan.md#L243-L246) and [Line 269](../../plans/001_lingua_ludi_core_architecture_plan.md#L269) (Target Files 12 & 13)
*   **Discrepancy Description:** Routine informational prompts and warnings (missing-pack list phrasing, non-Git warning text, crashed-session takeover prompt) specify required content by purpose rather than verbatim wording.
*   **Rationale:** Minor copy differences do not alter behaviour or fail automated assertions unless exact prose is enforced.
*   ***AI Suggestion:*** DECLINE
*   ***Declination Rationale:*** All behavioural invariants and required display data (available `<iso>` packs, unprotected `_gm-materials/` notice, `.active_session` `session_id` and `started_at` confirmation) are already pinned; mandating verbatim copy for routine UX prompts adds verbosity without resolving any implementation ambiguity.

### [Issue 19 — S9R3-N10] [`egm-codebase-details`, `egm-implementation-readiness`] - Severity: Nit

*   **Affected Scope:** [Line 172](../../plans/001_lingua_ludi_core_architecture_plan.md#L172) (Target File 6 `templates/glossary.yml`)
*   **Discrepancy Description:** Target File 6 names placeholder tokens (`Term A`, `Place A`, `directory/PlaceA.md`) for commented examples without enumerating all four Spec §4.3 term keys (`canonical_term`, `target_rendering`, `policy`, `related_catalog_path`), while remaining prose choices stay intentionally descriptive.
*   **Rationale:** Listing the four YAML keys aligns `templates/glossary.yml` commented examples directly with Spec §4.3 while preserving legitimate content-authoring latitude elsewhere.
*   ***AI Suggestion:*** PROPOSE FIX
*   ***Justification:*** Explicitly citing Spec §4.3's four term keys prevents omission of `related_catalog_path` in `glossary.yml` examples.
*   ***Proposed Edit:***
    *   Location: Target File 6 ([Line 172](../../plans/001_lingua_ludi_core_architecture_plan.md#L172))
    *   Change: Explicitly state that each commented neutral example entry includes all four Spec §4.3 keys (`canonical_term`, `target_rendering`, `policy`, `related_catalog_path`), retaining content-authoring latitude for Dutch pack examples, template HTML notes, and `README.md` registration prose.

### [Issue 20 — S9R3-N11] [`egm-style`] - Severity: Nit

*   **Affected Scope:** [Line 31](../../plans/001_lingua_ludi_core_architecture_plan.md#L31) (Decisions preamble)
*   **Discrepancy Description:** Style reviewer flagged recalibrating line 31 (`so the implementer makes zero design choices during authoring`) if the identified Majors remained open.
*   **Rationale:** Conditional calibration note tied directly to unresolved Majors.
*   ***AI Suggestion:*** DECLINE
*   ***Declination Rationale:*** Once accepted fixes `S9R3-M1`–`S9R3-M9` are applied, the plan leaves zero design choices to the implementer, so line 31 remains accurate as written.

### [Issue 21 — S9R3-N12] [`egm-style`] - Severity: Nit

*   **Affected Scope:** [Line 240](../../plans/001_lingua_ludi_core_architecture_plan.md#L240) (Target File 12 step 2)
*   **Discrepancy Description:** Target File 12 step 2 defaults `wingman.name` to `"Companion"` and `wingman.role` to `"Local colleague and fixer"`, diverging slightly from Spec §4.2's placeholder strings (`Companion A`, `Local fixer and colleague`).
*   **Rationale:** Matching Spec §4.2 verbatim avoids any mismatch when inspecting initialised `campaign.yml` against Spec §4.2.
*   ***AI Suggestion:*** PROPOSE FIX
*   ***Justification:*** Two-word adjustment yields 100% Spec §4.2 fidelity.
*   ***Proposed Edit:***
    *   Location: Target File 12 step 2 ([Line 240](../../plans/001_lingua_ludi_core_architecture_plan.md#L240))
    *   Change: Align defaults with Spec §4.2: `wingman.name: "Companion A"` and `wingman.role: "Local fixer and colleague"`.

### [Issue 22 — S9R3-N13] [`egm-style`] - Severity: Nit

*   **Affected Scope:** [Line 352](../../plans/001_lingua_ludi_core_architecture_plan.md#L352) (`V4(c)`)
*   **Discrepancy Description:** `V4(c)` tests `(How do I say ...?)` and `OOC: ...` without testing the third pinned OOC regex form `^\[[^\[\]]*\]$`.
*   **Rationale:** Adding one short `[ ... ]` test input ensures full coverage of all three OOC trigger patterns.
*   ***AI Suggestion:*** PROPOSE FIX
*   ***Justification:*** Minimal one-example addition closes the `V4(c)` test matrix.
*   ***Proposed Edit:***
    *   Location: `V4(c)` ([Line 352](../../plans/001_lingua_ludi_core_architecture_plan.md#L352))
    *   Change: Add `[How do I say ...?]` alongside `(How do I say ...?)` and `OOC: ...` in `V4(c)`.

### [Issue 23 — S9R3-N14] [`egm-style`] - Severity: Nit

*   **Affected Scope:** [Line 46](../../plans/001_lingua_ludi_core_architecture_plan.md#L46) and [Line 219](../../plans/001_lingua_ludi_core_architecture_plan.md#L219) (`D12`, Target File 10)
*   **Discrepancy Description:** `D12` and Target File 10 name `updated_items` and `due_targets_preview` without specifying their value formatting, even though Target File 13 consumes them on session close.
*   **Rationale:** Pinning simple key-value inline formats prevents mismatched parsing between `lingua-coach` and `lingua-play`.
*   ***AI Suggestion:*** PROPOSE FIX
*   ***Justification:*** Short comma/semicolon list format keeps Coach replies human-readable and unambiguous.
*   ***Proposed Edit:***
    *   Location: `D12` ([Line 46](../../plans/001_lingua_ludi_core_architecture_plan.md#L46)) and Target File 10 ([Line 219](../../plans/001_lingua_ludi_core_architecture_plan.md#L219))
    *   Change: Pin value shapes for D12 ack fields: `updated_items` as a comma-separated list of modified `item_id`s (or `none`) and `due_targets_preview` as up to `4` semicolon-separated `<item_id>: <surface_form>` entries (or `none`).

### [Issue 24 — S9R3-N15] [`egm-style`] - Severity: Nit

*   **Affected Scope:** [Line 102](../../plans/001_lingua_ludi_core_architecture_plan.md#L102) (Phase 3 exit gate)
*   **Discrepancy Description:** Phase 3 exit gate groups `V2`–`V7` together as the `mktemp -d` dry-run matrix, whereas `V2` (zero-Python scan and checkout symlink discovery) runs partly at repository level.
*   **Rationale:** Distinguishing `V2` from `V3`–`V7` keeps Phase 3 test scope exact.
*   ***AI Suggestion:*** PROPOSE FIX
*   ***Justification:*** Trivial one-phrase wording fix in Phase 3 exit gate.
*   ***Proposed Edit:***
    *   Location: Phase 3 exit gate ([Line 102](../../plans/001_lingua_ludi_core_architecture_plan.md#L102))
    *   Change: Phrase Phase 3 exit gate as running repo/discovery check `V2` alongside executing the neutral dry-run matrix (`V3`–`V7`) inside a disposable `mktemp -d` workspace and kit copy.

### [Issue 25 — S9R3-N16] [`egm-style`] - Severity: Nit

*   **Affected Scope:** [Line 102](../../plans/001_lingua_ludi_core_architecture_plan.md#L102) (Phase 3 exit gate)
*   **Discrepancy Description:** Throwaway dry-run fixture `languages/xx/pack.md` specifies `Target Language X`, `B1`, and two error tags without naming its front-matter keys and H2 headings per D14.
*   **Rationale:** Naming the minimal D14 skeleton guarantees consistent throwaway fixture setup during Phase 3.
*   ***AI Suggestion:*** PROPOSE FIX
*   ***Justification:*** One parenthetical note matches `D14` cleanly.
*   ***Proposed Edit:***
    *   Location: Phase 3 exit gate ([Line 102](../../plans/001_lingua_ludi_core_architecture_plan.md#L102))
    *   Change: Specify that `languages/xx/pack.md` uses D14's front-matter keys (`language_code: "xx"`, `display_name: "Target Language X"`, `cefr_levels: ["B1"]`, `error_taxonomy: ["word_order", "preposition"]`) and minimal H2 sections (`## CEFR Profiles`, `## Error Taxonomy Notes`, `## Wingman Scaffolding Cues`).

### [Issue 26 — S9R3-N17] [`egm-style`] - Severity: Nit

*   **Affected Scope:** [Line 24](../../plans/001_lingua_ludi_core_architecture_plan.md#L24) (Starting Baseline)
*   **Discrepancy Description:** Style reviewer noted `.gitignore` contents (`__pycache__/`, `.venv/`) were absent from the zero-context review packet.
*   **Rationale:** Packet-content observation rather than inaccuracy in the plan.
*   ***AI Suggestion:*** DECLINE
*   ***Declination Rationale:*** Line 19 (`These facts were measured on 2026-11-27 on main at commit d151e30 before this plan was written:`) already grounds line 24 accurately as a pre-plan measurement of `main`; omitting `.gitignore` text from the review packet is not a plan defect.

### [Issue 27 — S9R3-N18] [`egm-style`] - Severity: Nit

*   **Affected Scope:** [Line 304](../../plans/001_lingua_ludi_core_architecture_plan.md#L304) (Target File 14 step 2)
*   **Discrepancy Description:** Target File 14 step 2 leaves a parenthetical open across `and offer` (`warn the user (Coach updates are unaffected since Promoter never reads \`ledger.md\`, and offer to proceed or drain first via \`lingua-play\`).`).
*   **Rationale:** Closing the parenthesis right after `ledger.md` restores grammatical structure cleanly.
*   ***AI Suggestion:*** PROPOSE FIX
*   ***Justification:*** Trivial punctuation fix in Target File 14 step 2.
*   ***Proposed Edit:***
    *   Location: Target File 14 step 2 ([Line 304](../../plans/001_lingua_ludi_core_architecture_plan.md#L304))
    *   Change: Replace `warn the user (Coach updates are unaffected since Promoter never reads \`ledger.md\`, and offer to proceed or drain first via \`lingua-play\`).` with `warn the user (Coach updates are unaffected since Promoter never reads \`ledger.md\`) and offer to proceed or drain first via \`lingua-play\`.`.

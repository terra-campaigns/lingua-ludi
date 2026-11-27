# Consolidated Review Report — `001_lingua_ludi_core_architecture.md`

## Review Metadata

| Field | Value |
| :--- | :--- |
| Target | `docs/designs/001_lingua_ludi_core_architecture.md` (Spec v0.2.0, lines 1–414, whole document) |
| Preset | uninterrupted (consolidated report, all rubrics) |
| FeatureRange | lines 1–414 |
| DetailedRange | N/A — the document contains no `## Detailed Implementation Plan`, `## Implementation Plan`, `## Work Breakdown`, or `## Implementation Steps` section, so Stage 9 (Implementation Readiness Test) is NOT RUN |
| GoldfishContext | no local linked files; the only Markdown link in the target is the external TOME URL at line 25, excluded in this headless run |
| Baseline | `AGENTS.md` (British English, blank line after every heading, one sentence per line in prose) |
| Synthesizer | egm-synthesizer, 2026-11-25 |

## Reviewer Status

| Stage | Report file | Lens | Status |
| :--- | :--- | :--- | :--- |
| 5 | `zoom_out_report.md` | egm-zoom-out | completed |
| 5 | `inventive_catalyst_report.md` | egm-inventive-catalyst | completed |
| 6 | `comprehension_report.md` | egm-comprehension | completed |
| 7 | `critic_report_design.md` | egm-design | completed |
| 7 | `critic_report_structural.md` | egm-structural | completed |
| 7 | `critic_report_style.md` | egm-style | completed |
| 8 | `intermediate_zoom_out_report.md` | egm-zoom-out | completed |
| 9 | — | egm-implementation-readiness | not run (no implementation-plan section exists in the target; DetailedRange N/A) |

## Overall Verdict

No stage rated any finding as a 🔴 Blocker.
The architecture's direction holds at both zoom-out altitudes: the root problems (lines 34–41) are genuine, each goal (lines 69–76) maps to one of them, the zero-Python stance is applied consistently with `AGENTS.md` §1/§3, and the reused `play-uvg`/`dutch-tutor` patterns were independently verified to exist in the sibling repos by Stages 5 and 8.
The document is nonetheless not sign-off ready.
Twelve consolidated Majors remain, led by the async Shadow Coach reliability cluster (M1), which Stage 8 calls a blocker in disguise because invariant 5 (line 347) is asserted rather than engineered and §7 (line 382) falsely claims no open questions directly above the §9 sign-off gate (lines 406–414).
The remaining Majors (M2–M12) are one-line contract decisions, and the nine Nits are contract-hygiene items.
Recommended gate: resolve M1–M8 before human sign-off; M9–M12 and the Nits can land in the same editing pass.

## Consolidated Action Items

### 🔴 Blockers

None.
No reviewer rated any finding as a Blocker.
Stage 8 assesses M1 together with M7 as the closest thing to a blocker in disguise, because the accepted prompt-only trade-off (alternatives row 371) silently covers the `ledger.md` race and the meta-leak isolation while §7 claims nothing is open.

### 🟡 Major Action Items

#### M1 — Async Shadow Coach: unnamed await primitive, scene-boundary `ledger.md` race, no drain, backlog cap, or crash recovery

* **Severity:** 🟡 Major (Stage 8: blocker-in-disguise).
* **Affected lines:** 100, 112, 118, 198–201, 219, 240–252, 256, 347, 356.
* **Sources (deduplicated):** `inventive_catalyst_report.md` F1 + F2; `critic_report_design.md` F1; `intermediate_zoom_out_report.md` F1.
* **Discrepancy:** The design dispatches every target-language utterance to `lingua-coach` fire-and-forget (lines 100, 198) and claims invariant 5 (line 347) plus the failure-mode row (line 356) make `ledger.md` races impossible.
  That claim assumes an await/ack harness primitive that is never named or verified — CUJ 5 (line 118) covers harnesses with no subagent tools, but not the middle case (subagents exist, no await), so learner data is silently dropped at session close.
  The GM's scene-boundary reads of `ledger.md` (line 256, permitted by lines 219/356) race the previous turn's in-flight coach write, because the only guaranteed wait is before session close (line 201).
  There is also no backlog cap, timeout, or coalescing rule, so a slow coach turns session close (line 112) into an unbounded wait, and a coach dying mid-table-write corrupts `ledger.md` (lines 240–252) with no recovery path anywhere in the document.
* **Recommendation:** Name the required versus optional harness capabilities explicitly.
  Define a drain protocol (queue-depth counter plus completion callback) with a bounded timeout at session close.
  Make scene-boundary reads wait for coach acknowledgement, or drop them.
  Add a "rebuild `ledger.md` from `sessions/`" recovery rule.
  All of this is expressible in Markdown within the zero-Python bound (line 344).

#### M2 — Fallback mode leaves the `ledger.md` writer undefined, breaking invariant 4

* **Severity:** 🟡 Major.
* **Affected lines:** 118, 219, 346, 363.
* **Sources (deduplicated):** `critic_report_design.md` F2; `intermediate_zoom_out_report.md` F6 (partial).
* **Discrepancy:** The write-ownership table (line 219) and invariant 4 (line 346) make `lingua-coach` the sole writer of `ledger.md`, but CUJ 5 (line 118) and the failure-mode row (line 363) defer coaching "to a separate session-end pass" without stating who writes `ledger.md` in that pass.
  Either the GM writes it (violating invariant 4) or nobody does (silently dropping the spaced-retrieval loop, the product's core pedagogical feature, lines 75, 238–257).
* **Recommendation:** Specify the fallback writer explicitly (for example, the GM writes `ledger.md` only in fallback mode, with the ownership table amended to "coach-only when subagents are available"), or state that fallback mode disables retrieval tracking and record that in `campaign.yml`.

#### M3 — Concurrent GM sessions race on the sole-writer files; no one-active-session rule

* **Severity:** 🟡 Major.
* **Affected lines:** 156, 216–218, 354–363.
* **Sources (deduplicated):** `critic_report_design.md` F3; `intermediate_zoom_out_report.md` F2.
* **Discrepancy:** `state.md`, `log.md`, and `sessions/<session_id>.md` are "GM only" writers (lines 216–218), but "GM" is a role, not a session: nothing prevents two parallel chats running `lingua-play` in the same `_gm-materials/lingua-ludi/<campaign>/` workspace, so two GMs concurrently append to the same files.
  The failure-mode table (lines 354–363) covers only the coach race; there is no locking, session marker, or "one active session per campaign" rule.
* **Recommendation:** Add an invariant that `lingua-play` refuses to start (or takes over with a warning) when a session marker exists, and add this race to the failure-mode table.

#### M4 — Unbounded context growth in the persistent personas and the Promoter; no token budgets

* **Severity:** 🟡 Major.
* **Affected lines:** 106, 112, 188, 195, 198, 203, 269.
* **Sources (deduplicated):** `critic_report_design.md` F4; `intermediate_zoom_out_report.md` F3.
* **Discrepancy:** The Wingman accumulates the full session transcript (line 188), Spotlight NPCs keep full scene history alive (line 195), the Coach receives every target-language turn (line 198), and the Promoter reads the entire append-only `log.md` plus the session's in-character lines (lines 112, 203), so promoter context grows linearly with campaign length.
  The only token-economics gesture in the document is lazy Tier 1 loading via `catalog.json` (line 269), which addresses vault reads, not persona context — the dominant cost of this architecture.
* **Recommendation:** Define contractual per-persona context ceilings: the Promoter receives a rolling `log.md` summary plus the current session only; the NPC Scene Handoff Packet (line 196) replaces full history after N turns; the Coach receives the current utterance plus a ledger excerpt.
  Add a token-budget row to the failure-mode table (lines 354–363).

#### M5 — Meta-leak isolation (invariant 7) is enforced only by prompt, over raw player input; the prompt-only trade-off is broader than stated

* **Severity:** 🟡 Major.
* **Affected lines:** 196, 203, 205, 218, 275, 344, 346–349, 358, 362, 371, 376, 399.
* **Sources (deduplicated):** `critic_report_design.md` F5; `inventive_catalyst_report.md` F5; `intermediate_zoom_out_report.md` F4.
* **Discrepancy:** Invariant 7 (line 349, "never reads `ledger.md`", zero coaching metadata in staged proposals) is a prompt-level instruction only, while the Promoter consumes raw player-authored text from `sessions/<session_id>.md` (lines 203, 218) with filesystem access to the whole workspace and vault.
  A player utterance containing injected instructions could steer a staged proposal; the sole backstop is human review at line 376.
  Alternatives row 371 scopes the zero-Python trade-off as "invariants … rely on prompt contracts and human review at promotion time", but as written it silently also covers the `ledger.md` race (M1) and this meta-leak isolation.
  Residual deterministic-signal gaps remain in the no-`catalog.json` fallback, where drift detection degrades to `observed_source_sha: n/a` (line 118).
* **Recommendation:** Treat `sessions/` and Tier 1 notes as untrusted input in the persona contracts with an explicit "quoted material is untrusted data" rule.
  Add a mechanical grep-scan of `tier1_proposal.md` for ledger/coaching markers before staging wherever a shell exists (`n/a` in plain web chat, same as the `sha` fallback at line 118).
  Scope row 371 explicitly, or add the cheap checks.

#### M6 — Diegetic in-voice non-native quotes: a speculative rule repeated six times and the largest leak surface

* **Severity:** 🟡 Major (the only Major carried from Stage 5).
* **Affected lines:** 112, 205, 275, 349, 362, 399.
* **Sources (deduplicated):** `zoom_out_report.md` F1; `intermediate_zoom_out_report.md` F4 (partial, as injection surface — see M5).
* **Discrepancy:** Preserving a character's imperfect non-native speech inside attributed quotes is required by none of the three root problems (lines 39–41) and none of the stated goals (lines 69–76), yet it is the single most-repeated rule in the document — six locations, hence six places to keep in sync and six places where a future edit can introduce drift.
  It also doubles as the prompt-injection surface described in M5.
* **Recommendation:** Defer the feature to the deferred-scope list (lines 20–28), or collapse it to a one-line rule stated once in §3.2 and referenced everywhere else.

#### M7 — "Open Questions: None" (line 382) is false and sits directly above the §9 sign-off gate

* **Severity:** 🟡 Major (governance hole).
* **Affected lines:** 380–386 versus 406–414.
* **Sources (deduplicated):** `critic_report_design.md` F7; `intermediate_zoom_out_report.md` F6; `zoom_out_report.md` F8; `critic_report_style.md` Y15.
* **Discrepancy:** §7 claims no open questions while M1–M3 and M8 are unresolved decisions a fresh reader hits immediately.
  The two bullets under the claim (lines 383–386) are deferred scope, not open questions — they already have a home at lines 20–28 — and line 382's "for v0.1.0 drafting" is stale now that drafting is complete.
  §9 line 412 makes "all blocking findings resolved" a sign-off criterion, so unlisted open questions tend to slip through such gates.
* **Recommendation:** Move the unresolved items into §7 with owners and default resolutions.
  Reword "None open." and move or reword the two bullets so they cannot be read as unresolved.

#### M8 — `.gitignore` privacy guarantee: "already excluded" contradicts verify-at-init, and the failure behaviour is undefined

* **Severity:** 🟡 Major.
* **Affected lines:** 74, 94, 145, 361, 375.
* **Sources (deduplicated):** `critic_report_design.md` F6; `critic_report_style.md` Y1; `intermediate_zoom_out_report.md` F7.
* **Discrepancy:** Lines 74 and 145 assert that `oude-mechanismen` vaults "already" exclude `_gm-materials/` from Git, Jekyll, `catalog.json`, `data_extractor.rb`, and `lint_wiki.py`, while CUJ 1 (line 94), the failure-mode row (line 361), and the alternatives row (line 375) all treat the ignore rule as something `lingua-init` verifies — a direct internal contradiction.
  Stage 8 re-verified the "already" claim against `oude-mechanismen` source (`data_extractor.rb:42–43`, `_config.yml.template:33`, `lint_wiki.py:55`), so the claim is true for template-built vaults today, but for any other host the guarantee reduces to the init check at line 94, whose failure behaviour is undefined: abort, warn-and-proceed, or auto-append to the host `.gitignore` (itself an unsanctioned write to a host-repo root file).
  If init proceeds on failure, failure mode "personal coaching data pushed to public campaign repo" (line 361) is the default outcome.
* **Recommendation:** Narrow the "already" claim to vaults running current `oude-mechanismen` and present the `lingua-init` check as the guarantee for all other hosts.
  Define hard-abort with instructions as the only failure behaviour, and state that `lingua-init` never edits the host's `.gitignore` itself.

#### M9 — Core retrieval-loop contract stated two incompatible ways: "3–5" prose versus `max_retrieval_targets: 4  # 0..5`

* **Severity:** 🟡 Major (upgraded from Nit by Stage 8, confirmed by Stage 6 and Stage 7-design).
* **Affected lines:** 75, 98, 187, 256, 398 versus 318.
* **Sources (deduplicated):** `zoom_out_report.md` F2; `inventive_catalyst_report.md` F4; `comprehension_report.md` P6; `critic_report_design.md` F8; `intermediate_zoom_out_report.md` F5.
* **Discrepancy:** Five places promise "3–5 due targets" while `campaign.yml` sets `max_retrieval_targets: 4` with range `0..5` (line 318): the floor of 3 is unenforced (0 is permitted) and the default cap of 4 makes 5 unreachable.
  This is the product's core pedagogical loop (goal line 75), so implementers will code the config and testers will check the prose — different behaviours.
* **Recommendation:** State the rule once in §3.5, either "up to `max_retrieval_targets` (default 4)" or a hardcoded 3–5 with the knob removed.

#### M10 — `canonical_language` has two sources of truth, and `feedback_language` versus `canonical_language` is never explained

* **Severity:** 🟡 Major.
* **Affected lines:** 56, 93, 112, 142, 203–204, 308–310.
* **Sources (deduplicated):** `zoom_out_report.md` F6; `inventive_catalyst_report.md` F3; `critic_report_design.md` F9; `comprehension_report.md` P5; `intermediate_zoom_out_report.md` F9.
* **Discrepancy:** `campaign.yml` declares `canonical_language: "yy"  # Overrides or mirrors host .agents/AGENTS.md` (line 310), while the rest of the document treats `.agents/AGENTS.md` as the authoritative canonical language (lines 56, 142, 203–204).
  "Overrides or mirrors" leaves precedence ambiguous: if the two disagree, `lingua-promote` could stage Tier 1 proposals in a language diverging from the vault's canonical language, detectable only if a human notices at review.
  Separately, `campaign.yml` carries both `feedback_language` and `canonical_language` (lines 308–309) but CUJ 1 (line 93) and CUJ 4 (line 112) use "Canonical Language Y" for both, so when the two keys differ is never stated.
* **Recommendation:** State one precedence rule — host `.agents/AGENTS.md` wins, `campaign.yml` is a fallback when the host file is absent (mirroring the CUJ 5 fallback at line 118) — and add one sentence distinguishing the two keys.

#### M11 — Undefined external references and undefined harness concepts; alternatives rejections are not auditable

* **Severity:** 🟡 Major.
* **Affected lines:** 17, 37, 45, 49–52, 51, 74, 90–118, 129–137, 142, 144–145, 186, 192, 196, 224, 233, 261, 269, 274, 289, 302, 311, 371–376, 402.
* **Sources (deduplicated):** `comprehension_report.md` P1, P2, P3, P4, P7; `critic_report_design.md` F12; `intermediate_zoom_out_report.md` F10.
* **Discrepancy:** `oude-mechanismen` is the load-bearing external system yet is never defined or linked, and its claimed exclusions of `_gm-materials/`, `.agents/`, `catalog.json`, `data_extractor.rb`, and `lint_wiki.py` (lines 74, 144–145, 261, 269) are asserted without a checkable reference.
  The prior experiments `play-uvg` (repo `charged`) and `dutch-tutor` (repo `hostile`) are treated as normative patterns (lines 17, 37, 224, 233, 274, 402) without links, and §6's rejections (lines 371–376) cite them with no issue, discussion, or PR reference.
  The core mechanism — what a "skill" (`SKILL.md`) or "subagent" is, how `lingua-init` is invoked, what "persistent", "scene-scoped", "fire-and-forget", and "async dispatch" mean mechanically — is never defined (lines 45, 49–52, 90–118, 129–137), which is also what M1 needs named.
  CEFR, the design's central axis (lines 51, 93, 100, 136, 139, 186, 192, 196, 289, 302, 311), is never expanded or linked.
  Stage 8 independently verified the external claims are accurate today, so these are self-containment gaps, not accuracy errors.
* **Recommendation:** Add a short external-dependencies note with links (or an explicit sibling-repo path convention) for `oude-mechanismen`, `play-uvg`/`charged`, `dutch-tutor`/`hostile`, CEFR, and the harness skill/subagent interface the design assumes.
  Expand CEFR on first use, and link the experiment repos or the superseded 0.1.0 draft in §6 so the rejection rationale is auditable.

#### M12 — Editorial Majors: overlong CUJ `Then:` sentences and an unsupported "proved" claim

* **Severity:** 🟡 Major (editorial, per the style lens).
* **Affected lines:** 37, 100, 106, 112.
* **Sources:** `critic_report_style.md` Y2, Y3.
* **Discrepancy:** The CUJ 2/3/4 `Then:` steps are single sentences of 6–8 chained clauses (line 112 runs about 90 words), which technically meets one-sentence-per-line but defeats its purpose: a fresh engineer must re-parse several unrelated behaviours per line.
  Line 37 claims two experiments "proved" a general claim about turn friction; the evidence supports "showed" or "demonstrated", and "turn overhead" is plainer than "turn friction".
* **Recommendation:** Split each `Then:` into one clause per line, and calibrate the verb at line 37 to match the evidence.

### 🟢 Nit Action Items

#### N1 — Speculative config knobs in v0.1: `wingman.policy: active`, `max_active_npcs`, `max_retrieval_targets`

* **Affected lines:** 191, 194, 316–318.
* **Sources (deduplicated):** `zoom_out_report.md` F3; `intermediate_zoom_out_report.md` F8.
* **Discrepancy:** The caps are sound cost controls (hardcoded defaults restated at lines 75, 187, 194, 348, 357), but making them configurable in v0.1 is speculative generality — no CUJ requires varying them per campaign.
* **Recommendation:** Hardcode `on_request`, `2`, and the retrieval bound for v0.1 and let real play justify the knobs later, applying the document's own Python-deferral discipline (lines 24, 374). This also collapses part of M9.

#### N2 — `catalog.json` schema restated in full instead of referenced

* **Affected lines:** 261–264 (rules at 266–275).
* **Sources (deduplicated):** `zoom_out_report.md` F5; `intermediate_zoom_out_report.md` F10.
* **Discrepancy:** The schema is owned by `oude-mechanismen`'s `jekyll/_plugins/data_extractor.rb`; restating all top-level and per-entry keys here creates a second source of truth that silently goes stale when the extractor changes (Stage 8 verified it matches today, so the risk is future drift).
* **Recommendation:** Reference the extractor (or its docs) for the shape and keep only the three lingua-ludi-relevant rules (lines 266–275).

#### N3 — Dual installation-discovery mechanisms, an "e.g." hedge, and an undefined `LINGUA_LUDI_ROOT`

* **Affected line:** 128.
* **Sources (deduplicated):** `zoom_out_report.md` F7; `comprehension_report.md` P-n10; `intermediate_zoom_out_report.md` F11.
* **Discrepancy:** "installed read-only at e.g. `~/.local/share/lingua-ludi/` or discovered via `LINGUA_LUDI_ROOT`" names two mechanisms with an "e.g." that makes the contract untestable, and the env var's expected value (path to what, exactly) is never specified; §8 has no check that installation discovery works.
* **Recommendation:** Pick the env var as the one discovery mechanism, specify its value (absolute path to the kit root), and keep the path only as a labelled example.

#### N4 — Fallback persona labels inconsistent between CUJ 5 and the failure-mode table

* **Affected lines:** 118 versus 363.
* **Sources (deduplicated):** `zoom_out_report.md` F4; `intermediate_zoom_out_report.md` F11.
* **Discrepancy:** CUJ 5 falls back to `[GM]`, `[Wingman]`, `[Actor A]`; the failure-mode row uses `[GM]`, `[Wingman]`, `[NPC]`.
* **Recommendation:** Pick one canonical label set for the no-subagent fallback.

#### N5 — OOC-pause detection syntax is ambiguous for target-language play

* **Affected lines:** 106, 233–236.
* **Source:** `critic_report_design.md` F10.
* **Discrepancy:** Any message "wrapped in parentheses, brackets, or prefixed with `OOC:`" pauses fiction (line 233), but a player writing a target-language aside like `(laughs) hello` has no specified behaviour — a false positive silently freezes the scene, a false negative leaks coaching into fiction.
* **Recommendation:** Define the rule precisely (pause only if the message starts with `OOC:`, `(`, or `[`) and add a dry-run case for embedded parentheses in §8 (line 397).

#### N6 — The 8-char `sha` prefix is the only drift detector, and its limit is undocumented

* **Affected lines:** 264, 272, 359.
* **Source:** `critic_report_design.md` F11.
* **Discrepancy:** Drift detection relies on 8-hex-char SHA-256 prefixes (32 bits) from `catalog.json` (line 264); a changed Tier 1 file keeping the same prefix (probability about 2⁻³² per change) would pass undetected.
* **Recommendation:** Acceptable at vault scale, but document the limitation next to the "Stale parent `catalog.json`" failure mode (line 359).

#### N7 — Structural nits: risks heading naming, mixed subsection numbering, misplaced deferred scope, and an unlinked review record

* **Affected lines:** 13, 20–28, 78, 124–259, 283–321, 339, 352, 408.
* **Sources (deduplicated):** `critic_report_structural.md` S1–S4; `intermediate_zoom_out_report.md` F11.
* **Discrepancy:** The risks/failure-mode content exists but only under `## 5. Invariants & Failure Modes` (line 339) and `### Failure Modes & Mitigations` (line 352), so a reader scanning for the standard's literal "Risks" vocabulary will not find it.
  Only sections 3 and 4 use numbered H3s, breaking cross-reference symmetry.
  The deferred-scope block (lines 20–28) sits under `## Changelog` (line 13) and duplicates `### Non-Goals` (line 78).
  Line 408 references the EGM review record as a code span, not a Markdown link, against the repo convention in `AGENTS.md` §2.
* **Recommendation:** Rename the H3 at line 352 to include "Risks" or add a cross-reference; number all H3s or none; move lines 20–28 under `### Non-Goals` or promote them to their own H2; convert line 408 to a relative Markdown link when the review record lands.

#### N8 — Undefined tokens, terminology, and missing file contracts for `log.md` and the session log

* **Affected lines:** 10, 36, 41, 51–52, 55, 63, 65–66, 75, 92–106, 136, 138–139, 141, 143, 224, 236, 279–335, 287, 321, 375, 386, 402, 410.
* **Sources:** `comprehension_report.md` P-n1 through P-n9.
* **Discrepancy:** BCP-47, `<iso>`, and `nl` are never tied together (lines 51, 136, 138–139, 143, 287, 321, 402); there is no link to the 24XX SRD (line 224); VBT and `gem-instructions.md` are undefined (line 386); `session_id` has no defined format, generator, or uniqueness rule (lines 65–66, 94, 100, 112, 218, 220, 270); "Setting Cartridge" (line 141) and "the current brief prompt" (line 236) are used once and never defined; "vault" versus "host campaign repository" is never stated to be the same thing (lines 36, 41, 55, 75, 375); the neutral-placeholder and EGM conventions live only in `AGENTS.md`, not in the document (lines 10, 92–106, 410); and §4 details only `pack.md`, `campaign.yml`, and `glossary.<iso>.yml`, giving `log.md` no contract beyond "append-only narrative session summaries" (line 63) and never specifying the `sessions/<session_id>.md` turn-log format that the Promoter strips `OOC:` lines from (lines 52, 137, 279–335).
* **Recommendation:** Add a short "Terminology & Conventions" paragraph (placeholders, vault = host campaign repository, `session_id` format and generator) and expand or link BCP-47, the 24XX SRD, and VBT on first use.
  Add `log.md` and `sessions/<session_id>.md` contracts to §4.

#### N9 — Style nit tail: uncalibrated claims, filler words, and ASCII arrows in prose

* **Affected lines:** 36, 39, 41, 100, 105, 144, 195, 224, 236, 240, 255, 272, 274, 371–372, 398.
* **Sources:** `critic_report_style.md` Y4–Y14.
* **Discrepancy:** Uncalibrated superlatives and absolutes ("works best" line 36, "never" line 39, "clutter" line 41, an unevidenced external-behaviour claim at line 144); filler adverbs and adjectives (lines 100, 195, 236, 240, 224, 272, 274); subjective quality judgments in the alternatives rationale (lines 371–372); and ASCII `->` arrows in prose (lines 105, 255, 398) where the diagram uses `→`.
* **Recommendation:** Calibrate or delete each item as itemised in `critic_report_style.md`; use `→` in prose for typographic consistency.

## Deduplicated Findings

The following raw findings were merged because they describe the same underlying issue:

1. Async-coach race/ack/backlog/crash gap: `inventive_catalyst_report.md` F1 + F2, `critic_report_design.md` F1, `intermediate_zoom_out_report.md` F1 → **M1**.
2. "3–5" versus `max_retrieval_targets: 4`: `zoom_out_report.md` F2, `inventive_catalyst_report.md` F4, `comprehension_report.md` P6, `critic_report_design.md` F8, `intermediate_zoom_out_report.md` F5 → **M9**.
3. `canonical_language` dual source of truth plus the unexplained `feedback_language` distinction: `zoom_out_report.md` F6, `inventive_catalyst_report.md` F3, `critic_report_design.md` F9, `comprehension_report.md` P5, `intermediate_zoom_out_report.md` F9 → **M10**.
4. Diegetic-quotes repetition (six locations, speculative, injection surface): `zoom_out_report.md` F1, `intermediate_zoom_out_report.md` F4 → **M6** (injection aspect in **M5**).
5. `.gitignore`-check failure behaviour and the "already excluded" contradiction: `critic_report_design.md` F6, `critic_report_style.md` Y1, `intermediate_zoom_out_report.md` F7 → **M8**.
6. Concurrent-session race on GM-owned files: `critic_report_design.md` F3, `intermediate_zoom_out_report.md` F2 → **M3**.
7. Undefined external systems (`oude-mechanismen`, `play-uvg`/`dutch-tutor` and their repos, harness skill/subagent interface, CEFR) and unauditable §6 rejections: `comprehension_report.md` P1, P2, P3, P4, P7, `critic_report_design.md` F12, `intermediate_zoom_out_report.md` F10 → **M11**.
8. Prompt-only enforcement gaps (meta-leak isolation, `n/a` sha fallback): `inventive_catalyst_report.md` F5, `critic_report_design.md` F5, `intermediate_zoom_out_report.md` F4 → **M5**.
9. False "Open Questions: None" above the §9 gate, including the misplaced deferred bullets and the stale "for v0.1.0 drafting": `critic_report_design.md` F7, `intermediate_zoom_out_report.md` F6, `zoom_out_report.md` F8, `critic_report_style.md` Y15 → **M7**.
10. Fallback-mode `ledger.md` writer undefined: `critic_report_design.md` F2, `intermediate_zoom_out_report.md` F6 → **M2**.
11. Unbounded persona/Promoter context growth: `critic_report_design.md` F4, `intermediate_zoom_out_report.md` F3 → **M4**.
12. Speculative config knobs: `zoom_out_report.md` F3, `intermediate_zoom_out_report.md` F8 → **N1** (also collapses part of M9).
13. Restated `catalog.json` schema as a second source of truth: `zoom_out_report.md` F5, `intermediate_zoom_out_report.md` F10 → **N2**.
14. Dual install discovery and undefined `LINGUA_LUDI_ROOT`: `zoom_out_report.md` F7, `comprehension_report.md` P-n10, `intermediate_zoom_out_report.md` F11 → **N3**.
15. Fallback label mismatch `[Actor A]`/`[NPC]`, deferred-scope placement, and the line 408 code span: `zoom_out_report.md` F4, `critic_report_structural.md` S3–S4, `intermediate_zoom_out_report.md` F11 → **N4** and **N7**.

## Traceability Matrix

| Raw finding | Stage report | Reported severity | Consolidated item |
| :--- | :--- | :--- | :--- |
| Z-F1 | zoom_out_report.md (Stage 5) | Major | M6 |
| Z-F2 | zoom_out_report.md (Stage 5) | Nit | M9 |
| Z-F3 | zoom_out_report.md (Stage 5) | Nit | N1 |
| Z-F4 | zoom_out_report.md (Stage 5) | Nit | N4 |
| Z-F5 | zoom_out_report.md (Stage 5) | Nit | N2 |
| Z-F6 | zoom_out_report.md (Stage 5) | Nit | M10 |
| Z-F7 | zoom_out_report.md (Stage 5) | Nit | N3 |
| Z-F8 | zoom_out_report.md (Stage 5) | Nit | M7 |
| C-F1 | inventive_catalyst_report.md (Stage 5) | Major | M1 |
| C-F2 | inventive_catalyst_report.md (Stage 5) | Major | M1 |
| C-F3 | inventive_catalyst_report.md (Stage 5) | Major | M10 |
| C-F4 | inventive_catalyst_report.md (Stage 5) | Nit | M9 |
| C-F5 | inventive_catalyst_report.md (Stage 5) | Nit | M5 |
| P-1 | comprehension_report.md (Stage 6) | Major | M11 |
| P-2 | comprehension_report.md (Stage 6) | Major | M11 |
| P-3 | comprehension_report.md (Stage 6) | Major | M11 |
| P-4 | comprehension_report.md (Stage 6) | Major | M11 |
| P-5 | comprehension_report.md (Stage 6) | Major | M10 |
| P-6 | comprehension_report.md (Stage 6) | Major | M9 |
| P-7 | comprehension_report.md (Stage 6) | Major | M11 |
| P-n1 | comprehension_report.md (Stage 6) | Nit | N8 |
| P-n2 | comprehension_report.md (Stage 6) | Nit | N8 |
| P-n3 | comprehension_report.md (Stage 6) | Nit | N8 |
| P-n4 | comprehension_report.md (Stage 6) | Nit | N8 |
| P-n5 | comprehension_report.md (Stage 6) | Nit | N8 |
| P-n6 | comprehension_report.md (Stage 6) | Nit | N8 |
| P-n7 | comprehension_report.md (Stage 6) | Nit | N8 |
| P-n8 | comprehension_report.md (Stage 6) | Nit | N8 |
| P-n9 | comprehension_report.md (Stage 6) | Nit | N8 |
| P-n10 | comprehension_report.md (Stage 6) | Nit | N3 |
| D-F1 | critic_report_design.md (Stage 7) | Major | M1 |
| D-F2 | critic_report_design.md (Stage 7) | Major | M2 |
| D-F3 | critic_report_design.md (Stage 7) | Major | M3 |
| D-F4 | critic_report_design.md (Stage 7) | Major | M4 |
| D-F5 | critic_report_design.md (Stage 7) | Major | M5 |
| D-F6 | critic_report_design.md (Stage 7) | Major | M8 |
| D-F7 | critic_report_design.md (Stage 7) | Major | M7 |
| D-F8 | critic_report_design.md (Stage 7) | Nit | M9 |
| D-F9 | critic_report_design.md (Stage 7) | Nit | M10 |
| D-F10 | critic_report_design.md (Stage 7) | Nit | N5 |
| D-F11 | critic_report_design.md (Stage 7) | Nit | N6 |
| D-F12 | critic_report_design.md (Stage 7) | Nit | M11 |
| S-F1 | critic_report_structural.md (Stage 7) | Nit | N7 |
| S-F2 | critic_report_structural.md (Stage 7) | Nit | N7 |
| S-F3 | critic_report_structural.md (Stage 7) | Nit | N7 |
| S-F4 | critic_report_structural.md (Stage 7) | Nit | N7 |
| Y-1 | critic_report_style.md (Stage 7) | Major | M8 |
| Y-2 | critic_report_style.md (Stage 7) | Major | M12 |
| Y-3 | critic_report_style.md (Stage 7) | Major | M12 |
| Y-4 | critic_report_style.md (Stage 7) | Nit | N9 |
| Y-5 | critic_report_style.md (Stage 7) | Nit | N9 |
| Y-6 | critic_report_style.md (Stage 7) | Nit | N9 |
| Y-7 | critic_report_style.md (Stage 7) | Nit | N9 |
| Y-8 | critic_report_style.md (Stage 7) | Nit | N9 |
| Y-9 | critic_report_style.md (Stage 7) | Nit | N9 |
| Y-10 | critic_report_style.md (Stage 7) | Nit | N9 |
| Y-11 | critic_report_style.md (Stage 7) | Nit | N9 |
| Y-12 | critic_report_style.md (Stage 7) | Nit | N9 |
| Y-13 | critic_report_style.md (Stage 7) | Nit | N9 |
| Y-14 | critic_report_style.md (Stage 7) | Nit | N9 |
| Y-15 | critic_report_style.md (Stage 7) | Nit | M7 |
| E-F1 | intermediate_zoom_out_report.md (Stage 8) | Major | M1 |
| E-F2 | intermediate_zoom_out_report.md (Stage 8) | Major | M3 |
| E-F3 | intermediate_zoom_out_report.md (Stage 8) | Major | M4 |
| E-F4 | intermediate_zoom_out_report.md (Stage 8) | Major | M5 / M6 |
| E-F5 | intermediate_zoom_out_report.md (Stage 8) | Major | M9 |
| E-F6 | intermediate_zoom_out_report.md (Stage 8) | Major | M7 (and M2) |
| E-F7 | intermediate_zoom_out_report.md (Stage 8) | Major | M8 |
| E-F8 | intermediate_zoom_out_report.md (Stage 8) | Nit | N1 |
| E-F9 | intermediate_zoom_out_report.md (Stage 8) | Nit | M10 |
| E-F10 | intermediate_zoom_out_report.md (Stage 8) | Nit | M11 / N2 |
| E-F11 | intermediate_zoom_out_report.md (Stage 8) | Nit | N3 / N4 / N7 |

## Per-Stage Summaries

### Stage 5 — `zoom_out_report.md` (egm-zoom-out)

The first altitude pass found no Blockers and confirmed context sufficiency, genuine root-problem focus, and correct reuse of the verified `play-uvg`/`dutch-tutor` patterns.
It carried one Major — the six-times-repeated diegetic-quotes rule (M6) — and seven Nits, chiefly the "3–5" versus `max_retrieval_targets` conflict (M9), the `canonical_language` dual source (M10), and speculative config knobs (N1).
It praised the async coach as justified without checking its reliability property, a gap Stage 8 later exploited.

### Stage 5 — `inventive_catalyst_report.md` (egm-inventive-catalyst)

The TRIZ contradiction audit found no Blocker-level contradiction and confirmed the staged `exports/` intermediary cleanly resolves the canon-safety tension.
It contributed the first rigorous statement of the async-coach race (F1), the unbounded session-close backlog (F2), both merged into M1, plus the `canonical_language` precedence contradiction (F3 → M10), the retrieval-count inconsistency (F4 → M9), and prompt-enforcement gaps (F5 → M5).

### Stage 6 — `comprehension_report.md` (egm-comprehension)

The naive-reader self-containment test paraphrased every section and found no Blockers, confirming internal coherence.
Its seven Majors are all undefined-reference gaps — `oude-mechanismen`, `play-uvg`/`dutch-tutor` and their repos, the skill/subagent harness interface, CEFR, the `feedback_language`/`canonical_language` distinction, the retrieval-count ambiguity, and assumed external verification at lines 261/269 — consolidated into M11, M10, and M9.
Its ten Nits (BCP-47, 24XX SRD link, VBT, `session_id`, "Setting Cartridge", "brief prompt", vault terminology, §4 file contracts, placeholder conventions, `LINGUA_LUDI_ROOT`) roll up into N8 and N3.

### Stage 7 — `critic_report_design.md` (egm-design)

The hostile design critic produced the densest report: seven Majors covering the async-coach ack/drain/crash gap (F1 → M1), the fallback `ledger.md` writer (F2 → M2), the concurrent-session race (F3 → M3), missing token budgets (F4 → M4), prompt-injection into the Promoter (F5 → M5), the undefined `.gitignore` failure behaviour (F6 → M8), and the false "Open Questions: None" (F7 → M7).
Its five Nits map to M9, M10, N5, N6, and M11.

### Stage 7 — `critic_report_structural.md` (egm-structural)

The zero-context layout audit found no Blockers or Majors: heading hierarchy, blank lines, code fences, tables, separators, British English, and one-sentence-per-line all verified compliant programmatically.
Its four Nits (risks-heading naming, mixed H3 numbering, deferred scope under `## Changelog`, the line 408 code span instead of a link) are consolidated into N7.

### Stage 7 — `critic_report_style.md` (egm-style)

The style lens confirmed the repo writing conventions pass and found three Majors: the "already gitignored" contradiction (Y1 → M8), overlong CUJ `Then:` sentences (Y2 → M12), and the unsupported "proved" claim at line 37 (Y3 → M12).
Its twelve Nits are claim-calibration and filler-word items (Y4–Y14 → N9) plus the stale "for v0.1.0 drafting" at line 382 (Y15 → M7).

### Stage 8 — `intermediate_zoom_out_report.md` (egm-zoom-out)

The second altitude pass independently re-verified the external claims against `oude-mechanismen` source and the sibling skill repos — all accurate today — and confirmed the architecture still passes the zoom-out standard at altitude.
It upgraded the async-coach hole to the top Major (E-F1 → M1), named it a blocker in disguise together with the false §7 (E-F6 → M7), upgraded the retrieval-count conflict to Major (E-F5 → M9), and consolidated the concurrent-session race (E-F2 → M3), token budgets (E-F3 → M4), the prompt-only trade-off's true scope (E-F4 → M5/M6), and the `.gitignore` failure behaviour (E-F7 → M8).
Its Nits roll into N1, M10, M11/N2, and N3/N4/N7.

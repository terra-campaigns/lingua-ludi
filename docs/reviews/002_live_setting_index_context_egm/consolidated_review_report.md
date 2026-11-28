# Consolidated Review Report — `docs/designs/002_live_setting_index_context.md`

**Date:** 2026-11-28.

**Target:** `docs/designs/002_live_setting_index_context.md` (FeatureRange lines 1–221; DetailedRange lines 222–286), read against baseline `docs/designs/001_lingua_ludi_core_architecture.md` v0.2.2.

**Sources consolidated:** `zoom_out_report.md` (Stage 5, 15 findings), `comprehension_report.md` (Stage 6, 26 findings), `critic_report.md` (Stage 7, 16 findings), `intermediate_zoom_out_report.md` (Stage 8, 9 findings), `implementation_readiness_report.md` (Stage 9, 12 findings) — 78 raw findings in total.

**Method:** overlapping findings across stages were deduplicated into single actionable items; every raw finding is mapped to exactly one consolidated item in the traceability matrix at the end.

## Severity Summary

| Severity | Count | Item IDs |
| :--- | :--- | :--- |
| 🔴 Blocker | 1 | B-01 |
| 🟡 Major | 15 | M-01 – M-15 |
| 🟢 Nit | 11 | N-01 – N-11 |

No stage reported a blocker-class safety or invariant violation; the single blocker below is a contract gap that makes the spec's own verification test unpassable as written.

## 🔴 Blockers

### B-01 — Malformed-index behaviour is required by the verification plan but specified nowhere

§8 line 284 demands a degradation test for malformed `catalog.json` and `references.json`, yet §3.6 (lines 144–154) defines only absent-input rows and §5.1 item 1 (line 229) never mentions malformation, leaving abort-with-zero-writes versus skip-subsection-with-warning undecided — opposite behaviours with different blast radius on a read-only load.

* **Stage reports:** `implementation_readiness_report.md` (Blocker); related: `critic_report.md` N5 (matrix header overpromises "malformed").
* **Spec scope:** §3.6 lines 144–154; §5.1 line 229; §8 line 284.
* **Action:** add a §3.6 row ("malformed index JSON → skip that subsection, print one warning naming the file, never abort play") and a matching clause in §5.1 item 1.

## 🟡 Majors

### M-01 — The cap unit contradicts itself: "token-capped" versus the zero-Python character-cap rule

Lines 33, 68, 104 and 229 say the continuity window is "token-capped", while lines 125 and 158 pin line/character caps because a zero-Python kit cannot measure tokens, so an implementer cannot tell which cap the §3.2 contract uses.

* **Stage reports:** `zoom_out_report.md` M1; `comprehension_report.md` finding 1; `critic_report.md` M2; `implementation_readiness_report.md` Major ("token-capped" summaries).
* **Spec scope:** lines 33, 68, 104, 125, 158, 229.
* **Action:** replace every "token-capped" with "character-capped" and pin the number in §3.7.

### M-02 — The continuity window's unit, grouping key and sort key are all undefined

"Last-3 summaries" is ambiguous against the verified layout (four files per session on both hosts, plus the `2026-10-23_ep_012_*` outlier on `charged`): last-3 files splits a session and drops the glossary; last-3 sessions loads ≈48 KB pre-cap; §7 line 273 leaves the sort/grouping rule unpinned while §5.1 line 229 implements "last-3" as if decided.

* **Stage reports:** `zoom_out_report.md` M3; `intermediate_zoom_out_report.md` M3; `implementation_readiness_report.md` Major (grouping/sort key).
* **Spec scope:** lines 33, 57, 68, 125, 229, 273.
* **Action:** pin group key = leading `YYYY-MM-DD` filename prefix, cap = last 3 groups, per-group character cap with a printed truncation warning.

### M-03 — The headline budget figures are not reproducible from the spec's own numbers

The 5–12k headline (lines 35, 40) contradicts the doc's own component estimates (≈2.7k + 0.9k + 1.5k + 12.2k ≈ 17.6k with the continuity window on `charged`), §7 line 276 admits the ceiling "is not yet a number", and the 27–32k figure is called "the untrimmed literal rendering" at line 35 but "the measured literal cost of the trimmed live read" at line 182.

* **Stage reports:** `zoom_out_report.md` M2; `comprehension_report.md` finding 2.
* **Spec scope:** lines 35, 40, 182, 276.
* **Action:** pin the continuity character cap first, then restate the range from the pinned caps and name the 27–32k rendering precisely.

### M-04 — AGENTS.md is specified as both "wholesale" and "tone", and the Tone subsection has no cap

Line 31 and line 33 say `.agents/AGENTS.md` is read wholesale, while lines 68, 101 and 229 say only a "tone subsection"/"tone text" with no pinned extraction rule; §3.7 line 158 claims every component has a cap, which is false for Tone, so the bounded-fixed-cost goal rests on a measurement, not a cap.

* **Stage reports:** `zoom_out_report.md` M4; `intermediate_zoom_out_report.md` M1; `implementation_readiness_report.md` Major (wholesale versus tone).
* **Spec scope:** lines 31, 33, 68, 101, 158, 229, 276.
* **Action:** pick one rule (wholesale is the boring implementable choice) and give the Tone subsection a character cap with a printed warning; state whether `aesthetics.yml` is intentionally out of the read list.

### M-05 — The catalog pre-mark is simultaneously pinned, optional and an open question

§3.2 line 115 lists the pre-mark as a fixed subsection and line 158 pins its cap at 8, line 33 calls it "optional", §5.1 line 229 builds it with cap 8, and §7 line 275 asks whether it stays or is dropped; the path/title matching heuristic itself is defined nowhere and §4.3 line 188 already concedes it can misfire.

* **Stage reports:** `critic_report.md` M3; `zoom_out_report.md` M8; `intermediate_zoom_out_report.md` N4; `implementation_readiness_report.md` Major (cap 8 versus open question).
* **Spec scope:** lines 33, 69, 105, 115, 158, 188, 229, 275.
* **Action:** decide before approval — reviewers recommend dropping the pre-mark outright; if it stays, pin the matcher and an "omitted when…" rule in §3.2.

### M-06 — The §5.1 edit plan collides with the real SKILL.md structure and mixes two "step 1"s

Line 229 moves the pinned sequence to §4 but `skills/lingua-play/SKILL.md` already has `### 4. Turn Loop` and `### 5. Session Close Protocol`, so a literal edit yields duplicate headings; the same bullet references the pre-flight §1 catalog line (SKILL.md line 22), pinned step 1, and the malformed list marker at SKILL.md line 33 at once.

* **Stage reports:** `intermediate_zoom_out_report.md` M2; `implementation_readiness_report.md` Majors (renumbering; "step-1 catalog line"); `zoom_out_report.md` M5.
* **Spec scope:** line 229 versus line 32; SKILL.md lines 22, 33, 43, 59.
* **Action:** rewrite item 1 to renumber Turn Loop/Session Close to §5/§6 explicitly, name the §1 catalog line by its quoted text, and keep the marker fix as a separate instruction with step text verbatim.

### M-07 — The gaps register is promised at every play start, implemented only at init, and dormant on live hosts

CUJ 3 (line 83) says the register is re-derived at every `lingua-play` start, but §5.1 item 2 (line 235) adds it only to `lingua-init` step 7; on live hosts `digest_exists: false` is 0/100 and 0/54, so the register either prints ~49/16 null-summary "gaps" (noise) or nothing, and the third leg (categories with no digest) has no defined coverage rule.

* **Stage reports:** `zoom_out_report.md` M6, M7; `critic_report.md` N6; `intermediate_zoom_out_report.md` N5.
* **Spec scope:** lines 83, 153, 235.
* **Action:** narrow CUJ 3 to init-only, restrict the register to `digest_exists: false` plus empty digest categories, print a count only, and accept that both legs are currently untestable on live data.

### M-08 — The anti-forgery claim and the injection test do not cover the two wholesale subsections

§3.8 lines 163–164 claims flat-bullet rendering means injected text cannot forge block structure, but that rule is pinned only for catalog and reference index lines; Tone and Rulings load `AGENTS.md` and `decisions.md` wholesale, whose live content contains real `#`/`##` headings, and §8 line 285 tests injection only in a reference heading and a catalog summary.

* **Stage reports:** `critic_report.md` M1; `intermediate_zoom_out_report.md` M4.
* **Spec scope:** lines 33, 102, 115–117, 163–164, 285.
* **Action:** either strip heading sigils in Tone/Rulings too, or scope the §3.8 claim to index lines and extend the §8 injection test to AGENTS.md/decisions.md prose.

### M-09 — Canon tier 2 preps have no input path

§1.5 line 56 defines canon tier 2 as decisions.md plus session summaries **and preps**, and `session-preps/` exists on both hosts, but the read list (line 33), CUJ 1 (line 68) and the new ownership row (line 135) cover only `session-summaries/` and `digests/`.

* **Stage reports:** `critic_report.md` M4.
* **Spec scope:** lines 27, 33, 56, 68, 135.
* **Action:** either add preps to the continuity window or narrow the §1.5 definition and say why preps are excluded.

### M-10 — "Freshness by construction" overclaims: the catalog is a regenerated local cache

Line 41 claims freshness with no drift check, but `catalog.json` is an untracked, gitignored cache regenerated by `jekyll build` / `oude doctor --fix` (or on git commit on `hostile`), so notes edited without a rebuild are silently invisible to grep-on-demand and the pre-mark at play start.

* **Stage reports:** `critic_report.md` M6.
* **Spec scope:** lines 41, 105, 138–142, 149.
* **Action:** add a risk-table row for cache staleness and state explicitly that §3.6's direct-search fallback is the mitigation (promoter SHA citations catch drift only at promotion time).

### M-11 — Tier terminology collides and the any-tier branch is dead on live data

Inline `[T1]`/`[T2]` catalog labels share one token with canon tier 1/2 (§1.5 line 56), bare "Tier 1" is ambiguous between harness tier and canon tier, and the verified 114/114 and 72/72 `tier == 1` catalogs mean the `[T2]` label and the tier-2 filter can never fire on either live host.

* **Stage reports:** `comprehension_report.md` finding 6; `critic_report.md` M7; `zoom_out_report.md` N6; `intermediate_zoom_out_report.md` N2.
* **Spec scope:** lines 34, 49, 56, 76, 121–122, 129, 202, 257.
* **Action:** rename the inline labels (e.g. `[cat-T2]`), always write "canon tier N" outside §1.5, and note in §4.6 that both branches are untestable against live catalogs.

### M-12 — Core vocabulary is undefined and key meanings are deferred to unavailable documents

A fresh reader cannot learn what "pinned", "kit", "slug", "OOC", "EGM"/"Goldfish", "coach drain", "promoter", `max_retrieval_targets`, "chapter", "engine"/gem/Jekyll plugin, "host", the workspace file roles, or the pinned five steps are, because spec 001 sections and GitHub issues #6–#9 are referenced but opaque.

* **Stage reports:** `comprehension_report.md` findings 3, 4, 5, 7–13, 16, 17, 18.
* **Spec scope:** lines 11, 17, 25–27, 50–51, 57, 66, 70, 91–93, 130, 149, 154, 158, 188, 214, 218, 256, 263, 266.
* **Action:** define each term once at first use (or attach the full path to spec 001 at each reference) and add a one-line-per-step list of the pinned five steps or a pointer to SKILL.md.

### M-13 — Section references collide with spec 001 and with SKILL.md

"§3.2 per-persona ceilings" (lines 159, 276) means spec 001 §3.2, not this spec's §3.2; §-references in §3.1 and §5.1 (lines 91, 98–110, 229) point at SKILL.md sections; "the meaning of 'directly' is pinned in spec 001" (line 149) overstates what spec 001 pins.

* **Stage reports:** `comprehension_report.md` findings 14, 15; `zoom_out_report.md` N1, N2; `critic_report.md` N1, N2.
* **Spec scope:** lines 91, 98–110, 149, 159, 229, 276.
* **Action:** always write "spec 001 §x.y" or "SKILL.md §x" explicitly.

### M-14 — The §7 archived-marker premise is factually wrong and inline retracted rulings survive the stop

Line 274 claims host conventions differ, but both hosts use the identical `## Archived or Formalised Rulings` HTML-comment section, so the `/archived/i` stop already works on both; `charged` additionally keeps an inline `[Retracted …]` ruling above the Archived heading that the stop does not remove, so the loaded "confirmed rulings" include a retracted one on the motivating host.

* **Stage reports:** `critic_report.md` M5; `zoom_out_report.md` N3; `implementation_readiness_report.md` Nit (archived marker).
* **Spec scope:** lines 33, 102, 274.
* **Action:** close the open question with the verified convention and either strip `[Retracted…]` spans or state that inline-retracted rulings remain visible with their marks.

### M-15 — The document breaks spec 001's structural conventions: no risk table and the plan is embedded

Spec 001 §5.2 consolidates risks in a table and spec 001 §9 gate criterion 3 requires implementation plans in `docs/plans/NNN_*.md`, but spec 002 scatters its failure modes across §3.5/§3.6/§7 and embeds the Detailed Implementation Plan as §5 of the design doc.

* **Stage reports:** `critic_report.md` S1, S2.
* **Spec scope:** §3.5–§3.6, §5 lines 222–286, §7.
* **Action:** add a risk/failure-mode table (trigger + mitigation) alongside §6 and move §5 to `docs/plans/002_live_setting_index_context_plan.md`.

## 🟢 Nits

### N-01 — `## Scene Notes` is a bullet, not a heading, in `state.md`

`templates/state.md` line 24 (and the live `charged` workspace) has `- **Scene Notes:**` under `## Active Scene`, so the spec's H2 references will not grep.

* **Stage reports:** `zoom_out_report.md` N4; `critic_report.md` N4; `implementation_readiness_report.md` Nit.
* **Spec scope:** lines 42, 77, 83, 229, 268.

### N-02 — The read list is restated in four places and line 229 ends mid-bullet with a literal "…"

The same five-input list appears in §1.2, CUJ 1, the §3.1 diagram and §5.1, and the unfinished "…" suggests a dropped enumeration.

* **Stage reports:** `zoom_out_report.md` N5.
* **Spec scope:** lines 31–34, 68–69, 101–106, 229.

### N-03 — `references.json`'s top-level shape is under-described and the integrity check is available today

§3.5 lists per-source fields but never names the `sources` wrapper, hostile rows carry extra fields that would trip a naive unknown-field warning, and the verified `total_sources`/`generated` keys already allow a cheap count cross-check without an upstream `schema_version`.

* **Stage reports:** `zoom_out_report.md` N7; `implementation_readiness_report.md` Nit (top-level shape).
* **Spec scope:** lines 141–142, 277.

### N-04 — §5.1 item 4 labels the spec file "(Create)" though it already exists

`docs/designs/002_live_setting_index_context.md` is the current Draft; the label is stale.

* **Stage reports:** `implementation_readiness_report.md` Nit.
* **Spec scope:** lines 244–247.

### N-05 — The README conditional and the unnumbered ceiling are self-resolving; record the outcome

README.md does not enumerate the startup sequence, so §5.1 item 5's condition resolves to no change, and §8's budget test only needs the pinned line/char caps.

* **Stage reports:** `implementation_readiness_report.md` Nit.
* **Spec scope:** lines 249–252, 276, 282.

### N-06 — Vacuous ordering rationale at line 92

The load "completes before the step-4 write so the block and the retrieval targets never share a section", but the block is prompt-only and can never enter `state.md` regardless of order.

* **Stage reports:** `intermediate_zoom_out_report.md` N1.
* **Spec scope:** line 92 (versus line 116).

### N-07 — `sections[].heading` is consumed but never rendered, and the lazy-read wording is imprecise

Headings already live in `references.json`, so only digest bodies are actually lazy; §4.4 B says "lazy-read headings".

* **Stage reports:** `intermediate_zoom_out_report.md` N3.
* **Spec scope:** line 141 versus lines 123, 193.

### N-08 — Line 26 conflates startup phases: ledger rows load at pinned step 4, not "step 1"

`campaign.yml`/`state.md` load in pre-flight §1; the `max_retrieval_targets` rows load at pinned step 4 (SKILL.md line 40).

* **Stage reports:** `critic_report.md` N3.
* **Spec scope:** line 26.

### N-09 — The degradation matrix header says "Absent or malformed" but only absence is defined, and warning policies differ per row

Malformed handling exists only for the index files via §8 synthetic fixtures, and the `decisions.md`-absent row prints no warning unlike the AGENTS.md row.

* **Stage reports:** `critic_report.md` N5 (resolved together with B-01).
* **Spec scope:** lines 144–154 versus line 284.

### N-10 — Assorted style and jargon nits: "pre-mark", `<iso>`/`<cefr>`, digest/prep, mislabelled §4.3/§4.4 reason lines, "round 1/2", dangling file paths, undefined phrases, "flat turns"

Each needs one clause at first use; §4.3's catalog-line description also omits the `[T1]` label that the pinned format at line 121 includes.

* **Stage reports:** `comprehension_report.md` findings 19–26.
* **Spec scope:** lines 17, 81, 121, 175, 182, 186, 214, 229, 246.

### N-11 — Alternatives are prose subsections, not the spec 001 table, and no review-gate section names the intended EGM gate

Acceptable for a Draft, but one line naming the gate (mirroring spec 001 §9) makes the approval path explicit.

* **Stage reports:** `critic_report.md` N7.
* **Spec scope:** §4 lines 168–218; whole document.

## Traceability Matrix

| Stage report | Raw finding | Consolidated item |
| :--- | :--- | :--- |
| zoom_out_report.md | M1 | M-01 |
| zoom_out_report.md | M2 | M-03 |
| zoom_out_report.md | M3 | M-02 |
| zoom_out_report.md | M4 | M-04 |
| zoom_out_report.md | M5 | M-06 |
| zoom_out_report.md | M6 | M-07 |
| zoom_out_report.md | M7 | M-07 |
| zoom_out_report.md | M8 | M-05 |
| zoom_out_report.md | N1 | M-13 |
| zoom_out_report.md | N2 | M-13 |
| zoom_out_report.md | N3 | M-14 |
| zoom_out_report.md | N4 | N-01 |
| zoom_out_report.md | N5 | N-02 |
| zoom_out_report.md | N6 | M-11 |
| zoom_out_report.md | N7 | N-03 |
| comprehension_report.md | 1 (Blocker, cap mechanism) | M-01 |
| comprehension_report.md | 2 (27–32k double-booked) | M-03 |
| comprehension_report.md | 3 (5 steps never enumerated) | M-12 |
| comprehension_report.md | 4 ("pinned" undefined) | M-12 |
| comprehension_report.md | 5 ("kit" undefined) | M-12 |
| comprehension_report.md | 6 (harness vs canon tier) | M-11 |
| comprehension_report.md | 7 ("chapter" deferred) | M-12 |
| comprehension_report.md | 8 ("slug" undefined) | M-12 |
| comprehension_report.md | 9 (roles / coach drain undefined) | M-12 |
| comprehension_report.md | 10 (`max_retrieval_targets` undefined) | M-12 |
| comprehension_report.md | 11 (workspace file roles assumed) | M-12 |
| comprehension_report.md | 12 (OOC undefined) | M-12 |
| comprehension_report.md | 13 (EGM / Goldfish undefined) | M-12 |
| comprehension_report.md | 14 (section-number collisions) | M-13 |
| comprehension_report.md | 15 (§ refs point at SKILL.md) | M-13 |
| comprehension_report.md | 16 (meanings deferred to opaque docs) | M-12 |
| comprehension_report.md | 17 (engine / gem / plugin unclear) | M-12 |
| comprehension_report.md | 18 (host terms, "Full" undefined) | M-12 |
| comprehension_report.md | 19 (pre-mark jargon) | N-10 |
| comprehension_report.md | 20 (`<iso>`/`<cefr>` unexplained) | N-10 |
| comprehension_report.md | 21 (digest / prep undefined) | N-10 |
| comprehension_report.md | 22 (§4.3/§4.4 labels mislabeled) | N-10 |
| comprehension_report.md | 23 ("round 1/2" undefined) | N-10 |
| comprehension_report.md | 24 (dangling file references) | N-10 |
| comprehension_report.md | 25 (assorted undefined phrases) | N-10 |
| comprehension_report.md | 26 ("flat turns" jargon) | N-10 |
| critic_report.md | M1 (security claim vs wholesale subsections) | M-08 |
| critic_report.md | M2 (cap unit / no pinned number) | M-01 |
| critic_report.md | M3 (pre-mark pinned and undecided) | M-05 |
| critic_report.md | M4 (tier 2 preps gap) | M-09 |
| critic_report.md | M5 (archived-marker premise wrong) | M-14 |
| critic_report.md | M6 (freshness overclaim) | M-10 |
| critic_report.md | M7 ([T1]/[T2] label collision) | M-11 |
| critic_report.md | S1 (no risk table) | M-15 |
| critic_report.md | S2 (plan embedded in design doc) | M-15 |
| critic_report.md | N1 (§3.2 ceilings self-reference) | M-13 |
| critic_report.md | N2 ("directly" misattributed) | M-13 |
| critic_report.md | N3 ("step 1 loads ledger rows") | N-08 |
| critic_report.md | N4 (Scene Notes bullet) | N-01 |
| critic_report.md | N5 (matrix header overpromises) | N-09 |
| critic_report.md | N6 (gaps-register legs dormant) | M-07 |
| critic_report.md | N7 (alternatives table / gate section) | N-11 |
| intermediate_zoom_out_report.md | M1 (Tone subsection uncapped) | M-04 |
| intermediate_zoom_out_report.md | M2 (renumbering collision) | M-06 |
| intermediate_zoom_out_report.md | M3 (continuity window shape) | M-02 |
| intermediate_zoom_out_report.md | M4 (injection claim / test gap) | M-08 |
| intermediate_zoom_out_report.md | N1 (vacuous ordering rationale) | N-06 |
| intermediate_zoom_out_report.md | N2 (any-tier rule dead on live data) | M-11 |
| intermediate_zoom_out_report.md | N3 (consumed-but-unused heading field) | N-07 |
| intermediate_zoom_out_report.md | N4 (pre-mark weakest component) | M-05 |
| intermediate_zoom_out_report.md | N5 (register headline dormant) | M-07 |
| implementation_readiness_report.md | Blocker (malformed index unspecified) | B-01 |
| implementation_readiness_report.md | Major ("token-capped" in §5.1) | M-01 |
| implementation_readiness_report.md | Major (renumbering incomplete) | M-06 |
| implementation_readiness_report.md | Major (continuity grouping open) | M-02 |
| implementation_readiness_report.md | Major (pre-mark cap 8 vs §7) | M-05 |
| implementation_readiness_report.md | Major (AGENTS.md wholesale vs tone) | M-04 |
| implementation_readiness_report.md | Nit (Scene Notes H2 vs bullet) | N-01 |
| implementation_readiness_report.md | Nit ("(Create)" label stale) | N-04 |
| implementation_readiness_report.md | Nit ("step-1 catalog line" is §1) | M-06 |
| implementation_readiness_report.md | Nit (references.json shape) | N-03 |
| implementation_readiness_report.md | Nit (archived marker resolvable) | M-14 |
| implementation_readiness_report.md | Nit (README conditional / ceiling) | N-05 |

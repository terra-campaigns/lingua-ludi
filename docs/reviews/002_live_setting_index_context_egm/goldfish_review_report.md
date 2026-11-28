# Consolidated Goldfish Review — Resolution Record — `docs/designs/002_live_setting_index_context.md`

**Date:** 2026-11-28.
**Input:** `consolidated_review_report.md` (27 items deduplicated from 78 raw findings across `zoom_out_report.md`, `comprehension_report.md`, `critic_report.md`, `intermediate_zoom_out_report.md`, `implementation_readiness_report.md`).
**Method:** each item classified by the author (Elephant) as PROPOSE FIX or CONTEST; fixes applied to spec 002 v0.1.1 in the same pass.

| Item | Severity | Classification | Resolution |
| :--- | :--- | :--- | :--- |
| B-01 | 🔴 | PROPOSE FIX | §3.6 gains a malformed-index row (skip subsection, one warning naming the file, never abort); §5.1 of the plan carries the matching clause. |
| M-01 | 🟡 | PROPOSE FIX | All "token-capped" replaced with character caps; numbers pinned in §3.7. |
| M-02 | 🟡 | PROPOSE FIX | Continuity unit pinned: groups by leading `YYYY-MM-DD` filename prefix, last 3 groups, 4,000 chars per group, printed truncation warning. |
| M-03 | 🟡 | PROPOSE FIX | Budget restated from pinned caps (≈ 8–9k tokens measured estimate on `charged`); the 27–32k figure named precisely as the untrimmed literal rendering. |
| M-04 | 🟡 | PROPOSE FIX | AGENTS.md read wholesale (boring implementable rule); Tone subsection character-capped at 12,000 chars with printed warning; `aesthetics.yml` recorded as intentionally out of the read list. |
| M-05 | 🟡 | PROPOSE FIX | Catalog pre-mark dropped outright (YAGNI); catalog is grep-on-demand only. |
| M-06 | 🟡 | PROPOSE FIX | Plan item 1 rewritten: renumber `### 4. Turn Loop` → §5 and `### 5. Session Close Protocol` → §6 explicitly; catalog line named by quoted text; marker fix kept as a separate instruction. |
| M-07 | 🟡 | PROPOSE FIX | Gaps register narrowed to init-only, restricted to `digest_exists: false` plus empty digest categories, count printed only. |
| M-08 | 🟡 | PROPOSE FIX | §3.8 claim scoped to index lines; Tone/Rulings render inside a fenced data block under the untrusted banner; §9 injection test extended to AGENTS.md/decisions.md prose. |
| M-09 | 🟡 | PROPOSE FIX | `session-preps/` recorded as deliberately excluded from the continuity window (pre-session scratch, not continuity). |
| M-10 | 🟡 | PROPOSE FIX | Risk-table row added: `catalog.json` is a regenerated local cache, so notes edited without a rebuild are invisible to grep-on-demand; mitigation is the direct-search fallback plus promoter SHA checks at promotion time. |
| M-11 | 🟡 | PROPOSE FIX | Inline labels renamed `[cat-T1]`/`[cat-T2]`; "canon tier N" used outside §1.5; §4.6 notes both tier branches are untestable on live catalogs. |
| M-12 | 🟡 | PROPOSE FIX | §1.5 extended with one-line definitions for kit, pinned, slug, OOC, EGM/Goldfish, coach drain, promoter, `max_retrieval_targets`, chapter, host, and the workspace file roles. |
| M-13 | 🟡 | PROPOSE FIX | All cross-references written as "spec 001 §x.y" or "SKILL.md §x". |
| M-14 | 🟡 | PROPOSE FIX | Open question closed: both hosts use `## Archived or Formalised Rulings`, so the /archived/i stop works on both; inline `[Retracted …]` spans remain visible with their marks, stated explicitly. |
| M-15 | 🟡 | PROPOSE FIX | Risk/failure-mode table added as §6; the implementation plan moved to `docs/plans/002_live_setting_index_context_plan.md`, with §5 reduced to a pointer. |
| N-01 | 🟢 | PROPOSE FIX | References changed to the `Scene Notes` bullet under `## Active Scene` (it is a bullet in `templates/state.md`, not an H2). |
| N-02 | 🟢 | PROPOSE FIX | Read list stated once in §1.2 and referenced, not restated; the truncated "…" bullet finished in the plan file. |
| N-03 | 🟢 | PROPOSE FIX | `sources` wrapper named in §3.5; extra per-source fields ignored silently (not warned); `total_sources`/`generated` used for a cheap count cross-check instead of requesting an upstream `schema_version`. |
| N-04 | 🟢 | PROPOSE FIX | Plan labels the spec file as existing Draft, not Create. |
| N-05 | 🟢 | PROPOSE FIX | Recorded: README does not enumerate the startup sequence, so no README change; budget test uses the pinned char caps. |
| N-06 | 🟢 | PROPOSE FIX | Vacuous ordering rationale removed; only the non-blocking-of-coach-drain rule remains. |
| N-07 | 🟢 | PROPOSE FIX | Wording fixed: headings live in `references.json` and are consulted lazily per source; only digest bodies are lazy-read. |
| N-08 | 🟢 | PROPOSE FIX | §1.1 reworded: `campaign.yml`/`state.md` load in pre-flight, ledger retrieval rows load at pinned step 4. |
| N-09 | 🟢 | PROPOSE FIX | Resolved with B-01; warning policy unified — every missing, malformed, or skipped input prints one warning naming the file. |
| N-10 | 🟢 | PROPOSE FIX | One-clause definitions added at first use; §4.3 notes the `[cat-T1]` label. |
| N-11 | 🟢 | PROPOSE FIX | §10 names the review/approval gate mirroring spec 001 §9. |

No items were contested. Spec 002 bumped to v0.1.1 with a changelog row.

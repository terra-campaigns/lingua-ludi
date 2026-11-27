# EGM Review Summary: Lingua Ludi Core Architecture (`001`)

**Target Specification:** [001_lingua_ludi_core_architecture.md](../designs/001_lingua_ludi_core_architecture.md)
**Review Record Directory:** `docs/reviews/001_lingua_ludi_core_architecture_egm/`
**Status:** Approved (`Spec 0.2.2` signed off by `efsalvarenga` on `2026-11-27` after two independent zero-context Goldfish EGM review rounds; `0` Blockers)

---

## Round 1 — External Harness Goldfish Review (Spec `0.2.0` → `0.2.1`)

* **Date:** 2026-11-25
* **Scope:** Stages 5–8 (`egm-zoom-out`, `egm-inventive-catalyst`, `egm-comprehension`, `egm-design`, `egm-structural`, `egm-style`, intermediate `egm-zoom-out`).
* **Summary Record:** [consolidated_review_report.md](001_lingua_ludi_core_architecture_egm/consolidated_review_report.md) and resolved [goldfish_review_report.md](001_lingua_ludi_core_architecture_egm/goldfish_review_report.md).
* **Findings:** `0` Blockers, `12` Majors (`M1–M12`), `9` Nits (`N1–N9`).
* **Disposition:**
  * **Accepted & Resolved (`19`):** `M1–M5`, `M7–M12`, `N2–N9` applied in Spec `0.2.1` (named harness capabilities, race-free `ledger.md` reads, bounded close drain and crash recovery, `.active_session` concurrency guard, per-persona context ceilings, untrusted-data framing, shell meta-leak scan, `.gitignore` hard-abort behaviour, consistent retrieval count, `canonical_language` precedence, external dependency links, terminology, `session_id` and `log.md`/`sessions/` contracts, CUJ readability, and style calibration).
  * **Deferred (`1`):** `M6` (in-fiction non-native character quotes) deferred to `v0.2.0` in https://github.com/terra-campaigns/lingua-ludi/issues/2.
  * **Declined (`1`):** `N1` (declarative `campaign.yml` defaults `wingman.policy`, `max_active_npcs`, `max_retrieval_targets` retained by author agreement).

---

## Round 2 — Zero-Context `omp` Mistral Large (`high`) Re-Check (Spec `0.2.1` → `0.2.2`)

* **Date:** 2026-11-26
* **Execution Mode:** 5 isolated zero-context `omp` (`v18.22.0`) processes using `mistral/mistral-large-latest` (`--thinking=high`, `--no-session --no-context-files --no-rules --no-skills --no-extensions --no-tools`) across `egm-zoom-out`, `egm-comprehension`, `egm-design`, `egm-structural`, and `egm-style`.
* **Summary Record:** [round2_consolidated_report.md](001_lingua_ludi_core_architecture_egm/round2_consolidated_report.md).
* **Findings:** `0` Blockers, `6` deduplicated Majors (`R2-M1` through `R2-M6`), plus contract-precision Nits.
* **Disposition:**
  * **Accepted & Resolved (`5` Majors + Nits):**
    * `R2-M1`: Pinned startup ordering so `ledger.md` crash recovery (with `ledger.md.corrupt-<session_id>` archive fallback) and `pending_coach_turns` drain complete before startup step 4 reads due targets.
    * `R2-M2`: Mandated `git check-ignore -q _gm-materials/` semantics inside Git vaults and defined explicit unprotected-directory warning behaviour for non-Git standalone folders.
    * `R2-M3`: Corrected `catalog.json` consumed per-entry field count from six to five.
    * `R2-M4`: Aligned §3.2 `canonical_language` fallback condition with §4.2 ("absent or specifies no canonical language").
    * `R2-M5`: Removed maintainer-attribution prefixes in §6 Alternatives Considered and added rows for external SRS (Anki) and portable prompt bundles.
    * Nits: Unified `pending_coach_turns` and `observed_source_sha`, narrowed meta-leak scan markers to structural tokens with warn-and-annotate behaviour, added missing Language Pack hard-abort, pinned OOC whole-message regex, added Wingman untrusted-data symmetry, distinguished `rolling_glossary.md` from `glossary.<iso>.yml`, documented `.active_session` takeover prompt and `published: false` lifecycle, and aligned §9.1 review links.
  * **Deferred (`1` Major):** `R2-M6` (Tier 3 no-subagent single-context fallback and portable prompt-bundle compilation) deferred to `v0.2.0` in https://github.com/terra-campaigns/lingua-ludi/issues/3, making `lingua-coach` unconditionally the sole writer of `ledger.md` in `v0.1.0`.
  * **Note on `egm-zoom-out` `[UNVERIFIED]` check:** External repo claims (`oude-mechanismen`, `charged`, `hostile`) could not be checked by the `--no-tools` Round 2 agent and remain grounded in Round 1 Stage 8's direct source verification.

---

## Record Hygiene (`AGENTS.md` §2)

Intermediate per-lens working reports under `docs/reviews/001_lingua_ludi_core_architecture_egm/` were pruned before commit in accordance with [AGENTS.md](../../AGENTS.md), retaining only the linked summary records (`consolidated_review_report.md`, `goldfish_review_report.md`, and `round2_consolidated_report.md`).

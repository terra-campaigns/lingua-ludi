# Round 2 Goldfish Re-Check Report (`omp` / `mistral/mistral-large-latest` `high`)

**Target:** `docs/designs/001_lingua_ludi_core_architecture.md` (Spec `0.2.1`, lines 1–470)
**Execution Mode:** 5 isolated zero-context `omp` (`v18.22.0`) processes, `--model=mistral/mistral-large-latest --thinking=high --no-session --no-context-files --no-rules --no-skills --no-extensions --no-tools`
**Date:** 2026-11-26

## Reviewer Status

| Lens | Working Output (pruned per `AGENTS.md` §2) | Blockers | Majors | Nits |
| :--- | :--- | :---: | :---: | :---: |
| `egm-zoom-out` | `round2_zoom_out.md` | 0 | 2* | 7 |
| `egm-comprehension` | `round2_comprehension.md` | 0 | 2 | 7 |
| `egm-design` | `round2_design.md` | 0 | 4 | 7 |
| `egm-structural` | `round2_structural.md` | 0 | 0 | 2 |
| `egm-style` | `round2_style.md` | 0 | 2 | 8 |

*\*Note: `egm-zoom-out` Major #2 marked external repo claims `[UNVERIFIED]` solely because `--no-tools` prevented filesystem reads in zero-context mode; all external repo claims were independently verified against `oude-mechanismen`, `charged`, and `hostile` in Round 1 Stage 8 (summarised in [consolidated_review_report.md](consolidated_review_report.md)).*

## Overall Verdict

Zero Blockers across all 5 lenses.
The architecture, structural layout, writing conventions, and core write-ownership model passed every lens.
Six deduplicated Majors remain (all narrow contract/ordering fixes plus one scope decision on Tier 3 fallback), alongside a small set of contract-precision Nits.

## Deduplicated Majors (Round 2)

1. **R2-M1 (`egm-design` #4) — Startup ordering conflict between crash recovery / `pending_coach_turns` drain and session-start `ledger.md` read:**
   §3.2/§3.5 states the GM reads `ledger.md` *before spawning `lingua-coach`*, while §3.5 requires `lingua-coach` to rebuild a missing/malformed `ledger.md` and drain `pending_coach_turns`.
   *Fix:* Pin startup order: inspect `ledger.md` and `pending_coach_turns` → if malformed or pending turns exist, spawn `lingua-coach` first to rebuild/drain and wait for acknowledgement → GM reads due targets (or skips retrieval injection if unreadable) → begin turns.

2. **R2-M2 (`egm-design` #3) — `.gitignore` check semantics and non-git standalone directory handling:**
   Text matching `.gitignore` misses negation rules and patterns; non-git standalone directories (supported by CUJ 5) have no `.gitignore` at all.
   *Fix:* Specify `git check-ignore -q _gm-materials/` semantics inside Git repos; explicitly define behaviour for non-git standalone directories (warn that Git ignore protection is inactive and proceed, OR abort — author decision).

3. **R2-M3 (`egm-comprehension` #1, `egm-style` #2, `egm-design` #5) — `catalog.json` field count mismatch (§3.6 line 304):**
   Text states "six per-entry fields" then lists five (`path`, `title`, nullable `summary`, 8-char `sha`, `tier`).
   *Fix:* Correct "six" to "five".

4. **R2-M4 (`egm-comprehension` #2) — `canonical_language` fallback divergence (§3.2 line 243 vs §4.2 lines 364–365):**
   §3.2 says fallback only when `.agents/AGENTS.md` is "absent"; §4.2 says "absent or specifies no canonical language".
   *Fix:* Align §3.2 to "absent or specifies no canonical language".

5. **R2-M5 (`egm-design` #1 & #2, `egm-style` #1, `egm-structural` #2) — §6 Alternatives Rule attribution form and two missing alternatives:**
   "Rejected by author:" without linked records trips the standard's maintainer-attribution rule; external SRS (e.g. Anki) and portable prompt-bundle alternatives are not listed in §6.
   *Fix:* Rephrase to impersonal technical rationale ("Rejected because …", retaining commit `46f21cf` in row 1) and add concise rows for external SRS and portable prompt-bundle compilation.

6. **R2-M6 (`egm-zoom-out` #1) — Tier 3 no-subagent fallback vs §1.4 deferred scope:**
   §1.4 defers portable prompt-bundle compilation for mobile/web chat clients, while CUJ 5 / Tier 3 specifies a full single-context fallback mode with dual `ledger.md` ownership.
   *Decision needed:* Either (a) keep Tier 3 fallback explicitly for single-context harnesses and clarify §1.4 as deferring only *automated prompt-bundle compilation*, or (b) defer Tier 3 to v0.2.0 and require a subagent-capable harness in v0.1.0 (retaining Tier 2 session-end Coach degradation).

## High-Value Nits Worth Folding Into the Same Pass

* Unify field names: `pending_coach_turns` (lines 238 vs 265) and `observed_source_sha` (lines 145 vs 312).
* Tighten meta-leak scan (§3.6 rule 4): scan structural markers (`ledger.md`, `CEFR`, `clean_uses`, `next_due`, `pending_coach_turns`, `err-`) rather than ordinary English words (`register`, `idiom`), and specify **warn-and-annotate** behaviour.
* Define `lingua-init` missing Language Pack hard-abort listing available `languages/<iso>/` packs.
* Pin OOC whole-message regex (`^\([^()]*\)$` or `^\[[^\[\]]*\]$`), `max_active_npcs` range (`1..2`) + diagram sync, glossary `canonical_language` as descriptive metadata, Wingman untrusted-data 1-line symmetry, `rolling_glossary.md` vs `glossary.<iso>.yml` distinction, `published: false` lifecycle, `.active_session` takeover prompt, and §9.1 review link / calibration wording.

# Consolidated Re-Audit Report — `docs/designs/002_live_setting_index_context.md` v0.1.1 → v0.1.2

**Date:** 2026-11-28.
**Input:** the single allowed re-audit of the fixed spec, reports `reaudit_zoom_out_report.md`, `reaudit_comprehension_report.md`, `reaudit_critic_report.md`, `reaudit_intermediate_zoom_out_report.md`, `reaudit_implementation_readiness_report.md`.
**Result:** all 27 round-1 items re-verified against the live kit and both hosts — 24 fully resolved, 3 partially (M-11, M-12, M-13 residuals). New findings: 1 blocker, 4 majors, ~14 nits; all fixed in v0.1.2.

## Round-2 consolidated items

| Item | Severity | Finding | Fix in v0.1.2 |
| :--- | :--- | :--- | :--- |
| R2-B1 | 🔴 | The single fenced `## Live Setting Index` block is broken by real ``` lines inside both hosts' `AGENTS.md`, falsifying §3.8's anti-forgery claim; the §9 injection test had no fence-breaker case. | §3.2 fence-escape rule (strip fence lines and `#` sigils from Tone/Rulings); §9 injection test extended with a fence-breaker case. |
| R2-M1 | 🟡 | The `aesthetics.yml` exclusion was recorded only in the resolution record, not in the spec. | §1.2 states the exclusion explicitly. |
| R2-M2 | 🟡 | Continuity is empty on standalone hosts even though the campaign's own `log.md` has a last-3 window. | §3.2/§3.6 fallback to the last-3 `## Session` sections of `log.md`. |
| R2-M3 | 🟡 | Spec 001 §5.2 blanket `tier == 1` risk row contradicts the new any-tier play rule. | Plan item 1.3 now includes the §5.2 risk-row update. |
| R2-M4 | 🟡 | Plan link from `docs/designs/` was dead (`002_..._plan.md` resolved inside `docs/designs/`). | §5 link fixed to `../plans/002_live_setting_index_context_plan.md`. |
| R2-N1..N10 | 🟢 | Harness-tier ref pointed at spec 001 §1.5 instead of §3.2; within-group truncation non-deterministic; "last-3-files" unit wording in the 27–32k baseline; `<iso>`/`<cefr>` and the five pinned steps undefined; freshness goal unqualified; mtime fallback only in the risk table; "category with no digest" undefined. | All pinned: §1.5 definitions, §3.1 step enumeration, §3.2 truncation and mtime rules, §3.6 definition, §1.3 freshness qualified by the §6 risk row. |

Residual accepted trade-offs: zero tier-2 catalog records and `digest_exists: false` remain untestable on live data (covered by §9 synthetic fixtures); inline `[Retracted …]` rulings stay visible with their marks.

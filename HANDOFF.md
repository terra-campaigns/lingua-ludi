# Project Continuation & Handoff Context

This file captures the decisions, repository state, and next steps from the Elephant design sessions so any fresh conversation opened inside `~/Repos/lingua-ludi` can continue immediately without losing context.
Remove this file once v0.1.0 is shipped and normal issue tracking takes over.

## 1. Locked Project Decisions

* **Repository:** https://github.com/terra-campaigns/lingua-ludi (`PUBLIC`, trunk-based protected `main`, `deleteBranchOnMerge: true`).
* **Branch Protection Ruleset (`protect-main`):** https://github.com/terra-campaigns/lingua-ludi/rules/28094084 (PR required, conversation resolution required, stale reviews dismissed on push, force pushes and deletions blocked; `required_approving_review_count` is currently `0` until the GitHub App bot is live).
* **Dual Licence:** MIT (software code, if any is added later) + CC BY 4.0 (game mechanics, skills, personas, language packs, documentation), with attribution to Jason Tocci's 24XX SRD (*"24XX rules are CC BY 4.0 Jason Tocci"*).
* **Zero Python in v0.1.0:** Pure Markdown skills, subagent persona definitions, reusable Language Packs, and YAML/Markdown templates—matching the lightweight feel of `play-uvg` (`charged`) and `dutch-tutor` (`hostile`) without a Python runtime or CLI state machine.
* **Installed Globally, Executed Inside Host Campaign Repos:** Mirrors `oude-mechanismen` (`~/.local/share/lingua-ludi/` or `LINGUA_LUDI_ROOT`); players invoke skills from inside their campaign vault (e.g. `~/Repos/hostile` or `~/Repos/charged`), never inside `lingua-ludi`.
* **Campaign Workspace & Canon Safety:** Active play state (`campaign.yml`, `state.md`, `log.md`, `ledger.md`, `sessions/`, `exports/`) lives under the host vault's gitignored `_gm-materials/lingua-ludi/<campaign>/` (excluded from Jekyll, `catalog.json`, and `lint_wiki.py`); per-language setting glossaries live at `.agents/lingua-ludi/glossary.<iso>.yml`.
  Tier 1 vault directories (`campaigns/`, `directory/`, `systems/`) remain strictly read-only to all `lingua-ludi` agents.
* **Primary Gameplay Loop (v0.1.0):** Zoomed-in 24XX micro-scenes with player-owned dice rolls (`state.md` / `log.md`), `OOC:` pause/resume, and a reserved `## World Events` Markdown section in `state.md` for manual or future [TOME](https://github.com/efsalvarenga/tome) hooks.
* **Multi-Persona Subagent Topology (v0.1.0):**
  * **GM (Main Session):** Runs scenes, environment, stakes, 24XX risk statements, and owns `state.md`, `log.md`, and `sessions/`.
  * **Wingman (`lingua-wingman`):** Persistent companion/friendly tutor buddy (`on_request` default or `active`, dialogue-only, handles `OOC:` help).
  * **Spotlight NPCs (`lingua-npc`):** Selective scene-scoped subagents (max `2` per scene, briefed from Tier 1 notes, dialogue-only).
  * **Shadow Coach (`lingua-coach`):** Single persistent asynchronous background subagent (degrades to session-end synchronous pass in Tier 2 harnesses lacking completion tracking; unconditionally the sole writer of `ledger.md`, pinned startup recovery/drain before GM reads due targets, bounded close drain, crash recovery from `sessions/*.md`, surfaces up to `max_retrieval_targets` [default `4`] due targets at session start).
  * **Promoter (`lingua-promoter`):** Fresh session-end subagent (never reads `ledger.md`, reads current `sessions/<session_id>.md` with `[OOC]` stripped plus last `3` `log.md` summaries, filters `catalog.json` to `tier == 1`, verifies glossary rules and `observed_source_sha`s, runs structural shell meta-leak scan [warn-and-annotate] where available, and stages proposals under `_gm-materials/lingua-ludi/<campaign>/exports/<session_id>/`).
  * **Deferred to `v0.2.0`:** In-fiction non-native character quotes (https://github.com/terra-campaigns/lingua-ludi/issues/2) and single-context no-subagent fallback / portable prompt bundle (https://github.com/terra-campaigns/lingua-ludi/issues/3; `v0.1.0` requires a subagent-capable harness).
* **First Concrete Testbed:** Dutch (`languages/nl/`) Language Pack played inside `terra-campaigns/hostile` (`~/Repos/hostile`) with `.agents/lingua-ludi/glossary.nl.yml`, staging British English (`en-GB`) Tier 1 proposals.

## 2. Related Ecosystem References

* **Prior Lightweight Play Skills:**
  * `~/Repos/charged/.agents/skills/play-uvg/SKILL.md` + `campaigns/UVG-solo/{state.md,log.md}` — lightweight Markdown state/log and player-owned dice.
  * `~/Repos/hostile/.agents/skills/dutch-tutor/SKILL.md` — `OOC:` pause rule and unpublished-transcript-to-published-narrative promotion pattern.
* **Reference Campaign Engine:** `~/Repos/oude-mechanismen` (https://github.com/terra-campaigns/oude-mechanismen) — Tier 1 `catalog.json` (`data_extractor.rb`, including `nav_order`, nullable `summary`, and `tier: 2` `rolling_glossary.md`), `_gm-materials/` gitignore convention, and skill harness distribution.
* **First Host Campaign Testbed:** `~/Repos/hostile` (https://github.com/terra-campaigns/hostile) — `.agents/context/catalog.json` and `.agents/AGENTS.md`.
* **Agent Bot Identity Issue:** https://github.com/efsalvarenga/artifice/issues/63 — tracks creating a dedicated GitHub App (`<app-slug>[bot]`) for agent commits and PRs.

## 3. Current State

* **Active Branch:** `docs/phase2-core-spec` (PR #1 open: https://github.com/terra-campaigns/lingua-ludi/pull/1).
* **Files Updated on Branch (awaiting review/commit):**
  * `LICENSE` — MIT + CC BY 4.0 dual licence with 24XX SRD attribution.
  * `README.md` — zero-Python multi-persona skill kit overview.
  * `AGENTS.md` — repository boundaries, EGM gates, Markdown/YAML v0.1 conventions, provenance, and human-only merge rules.
  * `docs/designs/001_lingua_ludi_core_architecture.md` — Spec `0.2.2`, `Status: Approved` (`2026-11-27`, human sign-off by `efsalvarenga` after two zero-context Goldfish EGM review rounds; `N1` declined, `M6` deferred to https://github.com/terra-campaigns/lingua-ludi/issues/2, `R2-M6` deferred to https://github.com/terra-campaigns/lingua-ludi/issues/3).
  * `docs/reviews/001_lingua_ludi_core_architecture_review.md` — top-level EGM review summary linking the Round 1 and Round 2 summary records under `docs/reviews/001_lingua_ludi_core_architecture_egm/` (intermediate per-lens working reports pruned per `AGENTS.md` §2).
  * `HANDOFF.md` — this continuation file.

## 4. Exact Next Steps (Pick Up Here)

1. **Human Merge of PR #1 (`docs/phase2-core-spec`):** Review and merge https://github.com/terra-campaigns/lingua-ludi/pull/1 into `main` (human-only).
2. **Optional — Bump Review Count on `protect-main`:** Once `artifice-agent[bot]` PR workflow is confirmed on https://github.com/terra-campaigns/lingua-ludi/pull/1, raise `required_approving_review_count` on https://github.com/terra-campaigns/lingua-ludi/rules/28094084 from `0` to `1`.
3. **Draft & Review Implementation Plan:**
   * Create `docs/plans/001_lingua_ludi_core_architecture_plan.md` covering `skills/`, `agents/`, `templates/`, `languages/nl/`, and installation/playtest verification.
   * Run `egm-implementation-readiness` review before authoring files.
4. **Author v0.1.0 Kit & Playtest:**
   * Author `skills/` (`lingua-init`, `lingua-play`, `lingua-promote`), `agents/` (`lingua-wingman`, `lingua-npc`, `lingua-coach`, `lingua-promoter`), `templates/`, and `languages/nl/pack.md`.
   * Run dry-run checks and playtest inside `~/Repos/hostile`.

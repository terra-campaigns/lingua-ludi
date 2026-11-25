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
  * **Shadow Coach (`lingua-coach`):** Single persistent asynchronous background subagent (zero turn wait latency, sole writer of `ledger.md`, maintains `3–5` due spaced-retrieval targets).
  * **Promoter (`lingua-promoter`):** Fresh session-end subagent (never reads `ledger.md`, reads `log.md` and in-character session lines, filters `catalog.json` to `tier == 1`, verifies glossary rules and observed `sha`s, selectively weaves usable non-native player phrasing strictly into attributed in-voice character quotes in narrative drafts, and stages proposals under `_gm-materials/lingua-ludi/<campaign>/exports/<session_id>/`).
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
  * `docs/designs/001_lingua_ludi_core_architecture.md` — Spec `0.2.0`, `Status: Draft` (Elephant phase complete; ready for external Goldfish review).
  * `HANDOFF.md` — this continuation file.

## 4. Exact Next Steps (Pick Up Here)

1. **Review & Commit Spec `0.2.0` on `docs/phase2-core-spec`:** Review the rewritten [001_lingua_ludi_core_architecture.md](docs/designs/001_lingua_ludi_core_architecture.md), commit with validated provenance trailers, and push to https://github.com/terra-campaigns/lingua-ludi/pull/1.
2. **External Goldfish EGM Review (Other Harness):**
   * Run zero-context Goldfish EGM reviews on `docs/designs/001_lingua_ludi_core_architecture.md` in your review harness and record consolidated findings in `docs/reviews/001_lingua_ludi_core_architecture_review.md`.
   * Return to the Elephant session to resolve blocking findings, obtain explicit human developer sign-off, and set `Status: Approved`.
3. **Optional — Attach GitHub App Bot (from https://github.com/efsalvarenga/artifice/issues/63):**
   * Install the bot App on `terra-campaigns/lingua-ludi`.
   * Bump `required_approving_review_count` on `protect-main` from `0` to `1`.
4. **Draft & Review Implementation Plan:**
   * Create `docs/plans/001_lingua_ludi_core_architecture_plan.md` covering `skills/`, `agents/`, `templates/`, `languages/nl/`, and installation/playtest verification.
   * Run `egm-implementation-readiness` review before authoring files.
5. **Author v0.1.0 Kit & Playtest:**
   * Author `skills/` (`lingua-init`, `lingua-play`, `lingua-promote`), `agents/` (`lingua-wingman`, `lingua-npc`, `lingua-coach`, `lingua-promoter`), `templates/`, and `languages/nl/pack.md`.
   * Run dry-run checks and playtest inside `~/Repos/hostile`.

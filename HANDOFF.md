# Project Continuation & Handoff Context

This file captures the decisions, repository state, and next steps from the initial design session so a fresh conversation opened inside `~/Repos/lingua-ludi` can continue immediately without losing context.
Remove this file once Phase 3 (tooling bootstrap) is complete and normal issue tracking takes over.

## 1. Locked Project Decisions

* **Repository:** https://github.com/terra-campaigns/lingua-ludi (`PUBLIC`, trunk-based protected `main`, `deleteBranchOnMerge: true`).
* **Branch Protection Ruleset (`protect-main`):** https://github.com/terra-campaigns/lingua-ludi/rules/28094084 (PR required, conversation resolution required, stale reviews dismissed on push, force pushes and deletions blocked; `required_approving_review_count` is currently `0` until the GitHub App bot is live).
* **Dual Licence:** MIT (software code) + CC BY 4.0 (game mechanics, language packs, prompt contracts, documentation), with attribution to Jason Tocci's 24XX SRD (*"24XX rules are CC BY 4.0 Jason Tocci"*).
* **Primary Gameplay Loop (v0.1):** Zoomed-in 24XX micro-scenes (fast conversational turns, practical fluency, micro-lore promotes to Tier 1), keeping a `world_events` schema hook so a macro faction engine (e.g. [TOME](https://github.com/efsalvarenga/tome)) can plug in later without breaking v0.1 contracts.
* **Agent Topology (v0.1):** Human PC + AI Referee (in-character, target language, CEFR-bounded) + Silent Shadow Coach (out-of-character post-turn pass) + optional toggleable Companion (`on_request` default) + session-end Promoter (cross-language Tier 1 export).
* **Pedagogical Model (v0.1):** Configurable CEFR profile per campaign (`A2`–`C1`), charitable Referee interpretation, silent Coach error/vocabulary ledger feeding `3`–`5` due spaced-retrieval targets back into future Referee scenes.
* **Three Orthogonal Layers:**
  1. Generic Engine (`src/lingua_ludi/`, pure Python 3.11+, `uv`, `pydantic` v2, `pytest`, `ruff`, zero LLM/setting/language hardcoding).
  2. Reusable Language Packs (`languages/<iso>/`; first pack: Dutch `nl`).
  3. Read-only Setting Cartridges (`cartridges/<setting>/`; ingesting `oude-mechanismen` `.agents/context/catalog.json` Tier 1 format + per-language `glossary.<iso>.yml`).
* **First Concrete Testbed:** Dutch (`nl`) language pack + `terra-campaigns/hostile` (`~/Repos/hostile`) setting cartridge, exporting Tier 1 proposals back to British English (`en-GB`) in `exports/` (never mutating `~/Repos/hostile`).

## 2. Related Ecosystem References

* **Reference Engine Pattern:** `~/Repos/tome` (https://github.com/efsalvarenga/tome) — pure-Python state transformer, EGM specification/plan workflow, `uv` + `./scripts/lint.sh` gate, dual licence.
* **Reference Campaign Engine:** `~/Repos/oude-mechanismen` (https://github.com/terra-campaigns/oude-mechanismen) — Tier 1 `catalog.json` schema and skill harness conventions.
* **First Setting Cartridge Source:** `~/Repos/hostile` (https://github.com/terra-campaigns/hostile) — Tier 1 index at `~/Repos/hostile/.agents/context/catalog.json` and setting calibration at `~/Repos/hostile/.agents/AGENTS.md`.
* **Agent Bot Identity Issue:** https://github.com/efsalvarenga/artifice/issues/63 — tracks creating a dedicated GitHub App (`<app-slug>[bot]`) for agent commits and PRs so `efsalvarenga` can review, approve, and merge bot PRs cleanly.

## 3. Current State

* **Active Branch:** `docs/phase2-core-spec` (PR opened against `main`; awaiting human review and merge).
* **Files Authored on Branch:**
  * `LICENSE` — MIT + CC BY 4.0 dual licence with 24XX SRD attribution.
  * `README.md` — architecture overview, three-pass loop, and licence badges.
  * `AGENTS.md` — repository boundaries, EGM gates, tech stack, provenance, and human-only merge rules.
  * `docs/designs/001_lingua_ludi_core_architecture.md` — Spec `0.1.0` (Interface Contract `0.1.0`), `Status: Draft`.
  * `HANDOFF.md` — this continuation file.

## 4. Exact Next Steps (Pick Up Here)

1. **Merge or Review PR #1 (`docs/phase2-core-spec`):** Human reviews and merges the bootstrap docs and draft spec into `main` (or iterates directly on the PR branch).
2. **Run EGM Review Loop on `docs/designs/001_lingua_ludi_core_architecture.md`:**
   * Launch zero-context EGM Goldfish reviewers (`egm-synthesizer` / `egm-design` / `egm-comprehension` / `egm-zoom-out` / `egm-structural` / `egm-style`).
   * Write consolidated review to `docs/reviews/001_lingua_ludi_core_architecture_review.md` and resolve all blocking findings.
   * Obtain explicit human developer sign-off and update `Status: Approved` in `docs/designs/001_lingua_ludi_core_architecture.md`.
3. **Optional — Attach GitHub App Bot (from https://github.com/efsalvarenga/artifice/issues/63):**
   * Install the bot App on `terra-campaigns/lingua-ludi`.
   * Bump `required_approving_review_count` on `protect-main` from `0` to `1`.
4. **Draft & Review Implementation Plan:**
   * Create `docs/plans/001_lingua_ludi_core_architecture_plan.md` implementing the approved spec.
   * Run `egm-implementation-readiness` review before coding.
5. **Bootstrap Python Tooling & CI (`validate` gate):**
   * Add `pyproject.toml`, `.python-version` (`3.11`), `uv.lock`, `scripts/lint.sh`, and `.github/workflows/ci.yml` (`validate` job).
   * Add `validate` as a required status check on `protect-main` (https://github.com/terra-campaigns/lingua-ludi/rules/28094084).
6. **Execute TDD Implementation (v0.1.0):**
   * Build `src/lingua_ludi/` strictly via `code-author` TDD PRs + `code-reviewer` 4-lens review + human merge.
   * Add the Dutch (`nl`) Language Pack, the `hostile` Setting Cartridge fixture (`glossary.nl.yml`), and agent harness skills (`skills/`).

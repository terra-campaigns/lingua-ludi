# AGENTS.md — Lingua Ludi Agent Conventions

This document defines repository boundaries, governance gates, and engineering standards for developing **Lingua Ludi** (`terra-campaigns/lingua-ludi`).

## 1. Project Overview & Architectural Boundaries

* **Repository Identity:** `lingua-ludi` is a generic, setting-agnostic, language-agnostic tabletop RPG harness for acquiring European languages through structured gameplay, with staged Tier 1 lore export back to parent campaign vaults.
* **Three Orthogonal Layers:** Keep the **Engine** (core mechanics, state schemas, validators, prompt contracts), **Language Packs** (CEFR rubrics, register rules, Coach error taxonomy), and **Setting Cartridges** (read-only parent `oude-mechanismen` Tier 1 catalogues plus per-language setting glossaries) strictly decoupled.
* **Setting-Agnostic & Language-Agnostic Core:** The core library (`src/lingua_ludi/`) must never hardcode any specific setting, campaign lore, or target language.
  Core documentation and core unit test fixtures use neutral placeholders (`Setting A`, `Target Language X`, `Canonical Language Y`, `Actor A`, `Place A`, `Term A`).
  Concrete validation packs (such as the initial Dutch `nl` pack and `terra-campaigns/hostile` cartridge adapter fixture) live strictly outside the core engine package.
* **Authoritative Design:** [`docs/designs/001_lingua_ludi_core_architecture.md`](docs/designs/001_lingua_ludi_core_architecture.md) defines the mechanics, schemas, and interface contract.
  Where this file and the specification differ, the specification wins.
* **Core Philosophy:**
  1. *Deterministic Mechanical Core:* Pure Python handles 24XX dice/risk resolution, state transitions, learner-ledger scoring, spaced-retrieval selection, and export validation with zero network or LLM dependencies; a supplied seed reproduces every engine-controlled draw.
  2. *Strict Role Separation:* Referee (in-character, target language, CEFR-bounded), optional Companion (gated scaffolding), Shadow Coach (out-of-character, silent post-turn analysis), and Promoter (session-end distillation and cross-language translation) are isolated passes that never share a single blurred prompt.
  3. *Read-Only Parent Vault Boundary:* The harness may read a parent `oude-mechanismen` repository's `.agents/context/catalog.json` and notes, but must never write to or mutate any parent repository directly.
     All Tier 1 lore proposals are written to a local staging directory (`exports/`) for human review.
  4. *Transport-Neutral Boundary:* The caller (a CLI wrapper, coding agent skill, or external runner) owns LLM provider selection, credentials, transport, and interactive presentation.
  5. *Future Macro Extension Boundary:* v0.1 implements 24XX micro-scenes while reserving clean schema hooks (`world_events`) so a macro faction engine (such as [TOME](https://github.com/efsalvarenga/tome)) can plug in without breaking v0.1 contracts.
* **Out of Scope for v0.1:** Web UI, hosted deployment, authentication, direct parent-repo Git push/PR automation, audio transcription/TTS pipelines, and macro TOME faction execution.

## 2. Specification & Governance Protocols

* All major architectural features, schema changes, and engine stages must follow the **Elephant-Goldfish Model (EGM)**:
  * Specifications live in `docs/designs/<NNN>_<feature_slug>.md` using the EGM design template.
  * Zero-context adversarial critiques live in `docs/reviews/<NNN>_<feature_slug>_review.md`.
  * Before production code implementation commences, a specification must pass an EGM review (using the upstream `egm-review` or `egm-review-loop` workflow) and record `Status: Approved` with the human developer's explicit sign-off.
  * Implementation plans live in `docs/plans/<NNN>_<feature_slug>_plan.md` and implement an approved specification without restating it.
  * EGM workflow records live under `docs/reviews/<NNN>_<feature_slug>_egm/`.
    Commit only the summary records linked by a specification or plan; remove intermediate working records before merge and repoint references at a permanent GitHub commit URL.
* **Strict No-Code Rule during Elephant Drafting:** Design specifications focus on contracts, state schemas, invariants, failure modes, and verification criteria before production code is written.
* **Writing Conventions:**
  * Use British English across all documentation, prompts, comments, and error messages.
  * Follow every Markdown heading with a blank line.
  * In Markdown prose, write one sentence per line for clean diffs; headings, tables, and code blocks are exempt.

## 3. Technology Stack & Standards

* **Language & Runtime:** Python 3.11+ (pinned in `.python-version` once tooling is bootstrapped).
* **Package & Environment Management:** `uv` (`pyproject.toml` and committed `uv.lock`).
* **Verification Gate:** `./scripts/lint.sh` (running `ruff check`, `ruff format --check`, and `pytest` via locked `uv`) will serve as the green-baseline CI check (`validate`).
* **Core Libraries:**
  * `pydantic` (v2) for strict closed-schema runtime validation and JSON/YAML serialisation.
  * `pytest` for deterministic contract, mechanics, retrieval, and export tests.
  * `ruff` for linting and formatting.
* **LLM Integration:** Provider-agnostic; LLM calls live strictly in caller-supplied adapters or agent skills, never inside `src/lingua_ludi/`.

## 4. Branching, Provenance & Pull Request Protocols

* `main` is protected by repository ruleset `protect-main` (`https://github.com/terra-campaigns/lingua-ludi/rules/28094084`): pull request required, conversation resolution required, stale reviews dismissed on push, force pushes blocked, branch deletions blocked, and head branches auto-deleted on merge.
* Development branches use semantic prefixes: `feat/`, `fix/`, `docs/`, `chore/`.
* Every agent-generated commit must include validated provenance trailers (`Model-Author`, `Agent-Role`, `Co-authored-by`) checked with `validate_provenance.py` from `ARTIFICE_ROOT/scripts/`.
* **Autonomous Merge Prohibition [HUMAN-ONLY]:** Under no circumstances may any autonomous agent call `gh pr merge`, invoke GitHub merge APIs, or push directly to `main`.
  Agents must push topic branches, open Pull Requests with verification evidence, report the full PR URL to the user, and stop.

## 5. Attribution & Licensing

* **Code:** MIT Licence.
* **Game Mechanics, Language Packs, Prompt Contracts & Documentation:** Creative Commons Attribution 4.0 International (CC BY 4.0).
* Preserve explicit attribution to Jason Tocci's 24XX SRD (*"24XX rules are CC BY 4.0 Jason Tocci"*) in `LICENSE` and `README.md`.
* Do not copy commercial 2400 text or imply affiliation with the commercial 2400 series.

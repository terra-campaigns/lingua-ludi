# AGENTS.md — Lingua Ludi Agent Conventions

This document defines repository boundaries, governance gates, and authoring standards for developing **Lingua Ludi** (`terra-campaigns/lingua-ludi`).

## 1. Project Overview & Architectural Boundaries

* **Repository Identity:** `lingua-ludi` is a zero-Python (v0.1), setting-agnostic, language-agnostic tabletop RPG skill and subagent persona kit for acquiring European languages through structured gameplay, executed from inside host campaign repositories with staged Tier 1 lore export.
* **Three Orthogonal Layers:** Keep the **Generic Harness Kit** (`skills/`, `agents/`, `templates/`), **Language Packs** (`languages/<iso>/`: CEFR rubrics, register rules, Coach error taxonomy), and **Host Campaign Setting Data** (read-only `oude-mechanismen` Tier 1 catalogues, `.agents/AGENTS.md`, and campaign-hosted `.agents/lingua-ludi/glossary.<iso>.yml`) strictly decoupled.
* **Setting-Agnostic & Language-Agnostic Core:** Core skills (`skills/`), subagent persona definitions (`agents/`), and templates (`templates/`) must never hardcode any specific setting, campaign lore, or target language.
  Core documentation and dry-run fixtures use neutral placeholders (`Setting A`, `Target Language X`, `Canonical Language Y`, `Actor A`, `Place A`, `Term A`).
  Concrete Language Packs (such as Dutch `languages/nl/`) live strictly under `languages/<iso>/`, and setting-specific glossaries live inside their host campaign repositories rather than inside `lingua-ludi`.
* **Authoritative Design:** [`docs/designs/001_lingua_ludi_core_architecture.md`](docs/designs/001_lingua_ludi_core_architecture.md) defines the architecture, persona contracts, file formats, and invariants.
  Where this file and the specification differ, the specification wins.
* **Core Philosophy:**
  1. *Lightweight Markdown-First Play:* Gameplay uses plain human-readable Markdown files (`state.md`, `log.md`, `ledger.md`) and player-owned 24XX dice rolls with zero Python runtime dependencies in v0.1.
  2. *Multi-Persona Subagent Table:* The GM runs in the main session while delegating to a persistent Wingman (`lingua-wingman`), selective Spotlight NPCs (`lingua-npc`, capped per scene), an asynchronous background Shadow Coach (`lingua-coach`), and a fresh session-end Promoter (`lingua-promoter`).
  3. *Campaign-Repo Execution & Strict Write Ownership:* `lingua-ludi` is installed globally (mirroring `oude-mechanismen`) and invoked from inside a host campaign repository.
     Active play artefacts and staged exports live under the host vault's gitignored `_gm-materials/lingua-ludi/<campaign>/` directory; no `lingua-ludi` skill or subagent ever edits Tier 1 vault files (`campaigns/`, `directory/`, `systems/`).
  4. *Future Macro Extension Boundary:* v0.1 implements 24XX micro-scenes while reserving a `## World Events` section in `state.md` so manual hooks or a future macro engine (such as [TOME](https://github.com/efsalvarenga/tome)) can feed world shifts into play without breaking v0.1 templates.
* **Out of Scope for v0.1:** Python engine or CLI, web UI, hosted deployment, authentication, direct Tier 1 canon writes or PR automation, audio transcription/TTS pipelines, and macro TOME faction execution.

## 2. Specification & Governance Protocols

* All major architectural features, skill contracts, and schema changes must follow the **Elephant-Goldfish Model (EGM)**:
  * Specifications live in `docs/designs/<NNN>_<feature_slug>.md` using the EGM design template.
  * Zero-context adversarial critiques live in `docs/reviews/<NNN>_<feature_slug>_review.md`.
  * Before production skill, agent, or pack authoring commences, a specification must pass an EGM review and record `Status: Approved` with the human developer's explicit sign-off.
  * Implementation plans live in `docs/plans/<NNN>_<feature_slug>_plan.md` and implement an approved specification without restating it.
  * EGM workflow records live under `docs/reviews/<NNN>_<feature_slug>_egm/`.
    Commit only the summary records linked by a specification or plan; remove intermediate working records before merge and repoint references at a permanent GitHub commit URL.
* **Strict No-Code Rule during Elephant Drafting:** Design specifications focus on contracts, persona boundaries, file formats, invariants, failure modes, and verification criteria before production artefacts are written.
* **Writing Conventions:**
  * Use British English across all documentation, skill prompts, persona definitions, and templates.
  * Follow every Markdown heading with a blank line.
  * In Markdown prose, write one sentence per line for clean diffs; headings, tables, and code blocks are exempt.

## 3. Technology Stack & Standards

* **Core Formats (v0.1):** Markdown (`SKILL.md`, persona definitions, `state.md`, `log.md`, `ledger.md`, staged export proposals) and YAML (`campaign.yml`, `glossary.<iso>.yml`, front matter).
* **Zero Python in v0.1:** Do not introduce Python packages, `pyproject.toml`, or runtime scripts unless an approved specification explicitly justifies a deterministic tooling need.
* **Model & Harness Neutrality:** Skills and persona files must remain provider-agnostic and avoid embedding API keys, hardcoded model names, or harness-locked proprietary syntax outside documented fallback notes.

## 4. Branching, Provenance & Pull Request Protocols

* `main` is protected by repository ruleset `protect-main` (`https://github.com/terra-campaigns/lingua-ludi/rules/28094084`): pull request required, conversation resolution required, stale reviews dismissed on push, force pushes blocked, branch deletions blocked, and head branches auto-deleted on merge.
* Development branches use semantic prefixes: `feat/`, `fix/`, `docs/`, `chore/`.
* Every agent-generated commit must include validated provenance trailers (`Model-Author`, `Agent-Role`, `Co-authored-by`) checked with `validate_provenance.py` from `ARTIFICE_ROOT/scripts/`.
* **Autonomous Merge Prohibition [HUMAN-ONLY]:** Under no circumstances may any autonomous agent call `gh pr merge`, invoke GitHub merge APIs, or push directly to `main`.
  Agents must push topic branches, open Pull Requests with verification evidence, report the full PR URL to the user, and stop.

## 5. Attribution & Licensing

* **Code (when applicable):** MIT Licence.
* **Game Mechanics, Skills, Personas, Language Packs & Documentation:** Creative Commons Attribution 4.0 International (CC BY 4.0).
* Preserve explicit attribution to Jason Tocci's 24XX SRD (*"24XX rules are CC BY 4.0 Jason Tocci"*) in `LICENSE` and `README.md`.
* Do not copy commercial 2400 text or imply affiliation with the commercial 2400 series.

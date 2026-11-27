# Lingua Ludi

A zero-Python, setting-agnostic, language-agnostic tabletop RPG skill and subagent persona kit for acquiring European languages through structured gameplay, run inside host campaign repositories with staged Tier 1 lore export.

[![Code Licence: MIT](https://img.shields.io/badge/Code_Licence-MIT-yellow.svg)](LICENSE)
[![Content Licence: CC BY 4.0](https://img.shields.io/badge/Content_Licence-CC_BY_4.0-lightgrey.svg)](LICENSE)

> **Status:** `v0.1.0` implements approved Spec `0.2.2` (`2026-11-27`).
> Core architecture is defined in [docs/designs/001_lingua_ludi_core_architecture.md](docs/designs/001_lingua_ludi_core_architecture.md) and implemented per the approved [docs/plans/001_lingua_ludi_core_architecture_plan.md](docs/plans/001_lingua_ludi_core_architecture_plan.md) (EGM reviews: [docs/reviews/001_lingua_ludi_core_architecture_review.md](docs/reviews/001_lingua_ludi_core_architecture_review.md)).
> The kit ships `skills/`, `agents/`, `templates/`, and the Dutch pack `languages/nl/`.

---

## Overview

Lingua Ludi pairs lightweight, Markdown-first solo tabletop play (player-rolled 24XX micro-scenes with readable `state.md` and `log.md` files) with [Oude Mechanismen](https://github.com/terra-campaigns/oude-mechanismen)'s tiered campaign canon architecture.

Rather than running as a monolithic GM-plus-tutor chat prompt or a heavy software runtime, Lingua Ludi is installed alongside `oude-mechanismen` and invoked directly inside any host campaign repository.
Distinct subagent personas provide a genuine table feel: the GM runs the scene in the main session, a persistent **Wingman** offers friendly companion scaffolding and `OOC:` help, selective **Spotlight NPCs** speak with independent voices grounded in Tier 1 notes, an asynchronous **Shadow Coach** silently updates a Markdown spaced-retrieval ledger (`ledger.md`), and a fresh session-end **Promoter** stages clean canonical-language Tier 1 lore proposals without ever modifying published campaign notes.

---

## Three Orthogonal Layers

```text
┌──────────────────────────────────────────────────────────────────────┐
│  1. Lingua Ludi Kit (Generic, Zero Setting or Language Hardcoding)   │
│     • Skills: lingua-init, lingua-play, lingua-promote               │
│     • Subagent Personas: Wingman, Spotlight NPC, Coach, Promoter     │
│     • Plain Markdown & YAML campaign/export templates                │
├─────────────────────────────────┬────────────────────────────────────┤
│  2. Language Packs              │  3. Host Campaign Repo (Vault)     │
│     • ISO / BCP-47 code         │     • Read-only Tier 1 canon       │
│     • CEFR profiles (A2–C1)     │       (.agents/context/catalog.json│
│     • Coach error taxonomy      │       & .agents/AGENTS.md)         │
│     • Register & scaffolding    │     • .agents/lingua-ludi/glossary │
│       (first pack: Dutch `nl`)  │     • _gm-materials/lingua-ludi/   │
└─────────────────────────────────┴────────────────────────────────────┘
```

---

## Multi-Persona Turn & Session Loop

```text
  player turn (target language, player-rolled 24XX dice on risky actions)
        │
        ├──► [GM — Main Session] (in-character, target language, CEFR-bounded)
        │         • handles scene narration, stakes, state.md & log.md
        │         • invokes persistent [Wingman] on OOC:/request (or active beats)
        │         • invokes [Spotlight NPC] (max 1–2/scene) when engaging a named NPC
        │
        └──► [Shadow Coach — Async Background Subagent] (silent, non-blocking)
                  • analyses player's target-language utterance only
                  • sole writer of _gm-materials/lingua-ludi/<campaign>/ledger.md
                  • feeds up to max_retrieval_targets (default 4) due spaced-retrieval targets into future scenes

  session end
        └──► [Promoter — Fresh Subagent] (distil + translate + glossary/SHA check)
                  • reads narrative records only (never ledger.md or coaching notes)
                  • translates canon-altering outcomes into vault canonical language
                  • stages spoiler-safe proposal under _gm-materials/.../exports/
```

---

## Installation & Quick Start

`v0.1.0` is docs-only: clone this repository and point your harness at it, with zero install scripts.

1. Clone the kit and register its root: set the `LINGUA_LUDI_ROOT` environment variable to the checkout, or symlink `$HOME/.local/share/lingua-ludi` to the checkout so the `$HOME/.local/share/lingua-ludi` fallback resolves it.
2. Register `skills/` and `agents/` in a subagent-capable harness (`agy`, `omp`, Claude Code, Codex), keeping registration harness-neutral in your own configuration.
   `v0.1.0` requires a subagent-capable harness; single-context clients are unsupported (tracked in <https://github.com/terra-campaigns/lingua-ludi/issues/3>).
3. From inside a host campaign repository, confirm `_gm-materials/` is listed in that repository's `.gitignore` (`lingua-init` hard-aborts otherwise and never edits `.gitignore` itself).
4. Run `/lingua-init <campaign-slug> <target-iso> <cefr>`, then `/lingua-play <campaign-slug>` for sessions, and `/lingua-promote <campaign-slug> <session-id>` at session end to stage a Tier 1 proposal under `_gm-materials/lingua-ludi/<campaign>/exports/<session_id>/`.

---

## Repository Structure

- `skills/` — `lingua-init`, `lingua-play`, and `lingua-promote` Markdown skills.
- `agents/` — `lingua-wingman`, `lingua-npc`, `lingua-coach`, and `lingua-promoter` subagent persona definitions.
- `languages/nl/` — the first concrete Language Pack (Dutch, CEFR `A2`–`C1`, seven-tag error taxonomy).
- `templates/` — campaign workspace and Tier 1 export templates (`campaign.yml`, `state.md`, `log.md`, `ledger.md`, `session.md`, `glossary.yml`, `tier1_proposal.md`).
- `docs/` — the approved specification, implementation plan, and EGM review records.

---


## Project Governance & Development

Development follows the **Elephant-Goldfish Model (EGM)**.
See [AGENTS.md](AGENTS.md) for architectural boundaries, specification gates, and branch protection rules.

---

## Attribution & Licences

Lingua Ludi uses a dual-licence structure; see [LICENSE](LICENSE) for full terms:

- **Source Code & Software (if added later):** MIT Licence.
- **Game Mechanics, Skills, Personas, Language Packs & Documentation:** Creative Commons Attribution 4.0 International (CC BY 4.0).

24XX rules are CC BY 4.0 Jason Tocci (<https://jasontocci.itch.io/24xx>).

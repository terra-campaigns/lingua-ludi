# Lingua Ludi

A zero-Python, setting-agnostic, language-agnostic tabletop RPG skill and subagent persona kit for acquiring European languages through structured gameplay, run inside host campaign repositories with staged Tier 1 lore export.

[![Code Licence: MIT](https://img.shields.io/badge/Code_Licence-MIT-yellow.svg)](LICENSE)
[![Content Licence: CC BY 4.0](https://img.shields.io/badge/Content_Licence-CC_BY_4.0-lightgrey.svg)](LICENSE)

> **Status:** Specification Draft (`Spec 0.2.0`).
> Core architecture is defined in [docs/designs/001_lingua_ludi_core_architecture.md](docs/designs/001_lingua_ludi_core_architecture.md) and awaits Elephant-Goldfish Model (EGM) review and human sign-off before skills, personas, and language packs are authored.

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
                  • feeds 3–5 due spaced-retrieval targets into future scenes

  session end
        └──► [Promoter — Fresh Subagent] (distil + translate + glossary/SHA check)
                  • reads narrative records only (never ledger.md or coaching notes)
                  • translates canon-altering outcomes into vault canonical language
                  • stages spoiler-safe proposal under _gm-materials/.../exports/
```

---

## Project Governance & Development

Development follows the **Elephant-Goldfish Model (EGM)**.
See [AGENTS.md](AGENTS.md) for architectural boundaries, specification gates, and branch protection rules.
See [HANDOFF.md](HANDOFF.md) for current milestone context and continuation steps.

---

## Attribution & Licences

Lingua Ludi uses a dual-licence structure; see [LICENSE](LICENSE) for full terms:

- **Source Code & Software (if added later):** MIT Licence.
- **Game Mechanics, Skills, Personas, Language Packs & Documentation:** Creative Commons Attribution 4.0 International (CC BY 4.0).

24XX rules are CC BY 4.0 Jason Tocci (<https://jasontocci.itch.io/24xx>).

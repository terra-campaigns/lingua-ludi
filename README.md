# Lingua Ludi

A setting-agnostic, language-agnostic tabletop RPG harness for acquiring European languages through structured gameplay, with staged Tier 1 lore export back to parent campaign universes.

[![Code Licence: MIT](https://img.shields.io/badge/Code_Licence-MIT-yellow.svg)](LICENSE)
[![Content Licence: CC BY 4.0](https://img.shields.io/badge/Content_Licence-CC_BY_4.0-lightgrey.svg)](LICENSE)
[![Python 3.11+](https://img.shields.io/badge/python-3.11+-blue.svg)](https://www.python.org/downloads/)

> **Status:** Specification Draft (`Spec 0.1.0` / `Interface Contract 0.1.0`).
> Core architecture is defined in [docs/designs/001_lingua_ludi_core_architecture.md](docs/designs/001_lingua_ludi_core_architecture.md) and awaits Elephant-Goldfish Model (EGM) review and human sign-off before production code implementation begins.

---

## Overview

Lingua Ludi combines the deterministic, transport-neutral engine discipline of [TOME](https://github.com/efsalvarenga/tome) with the tiered canon architecture of [Oude Mechanismen](https://github.com/terra-campaigns/oude-mechanismen).

It is a plain-text, agent-driven harness rather than a deployed web application.
Players interact through zoomed-in 24XX-style micro-scenes in a target European language while a silent post-turn Shadow Coach logs errors and feeds spaced-retrieval targets back into future scenes.
At the end of a session, canon-altering outcomes can be distilled, translated into the parent universe's canonical language, checked against a setting glossary, and staged for human-reviewed Tier 1 promotion without ever mutating the parent repository directly.

---

## Three Orthogonal Layers

```text
┌──────────────────────────────────────────────────────────────────────┐
│  1. Engine / Harness (Generic, Zero Setting or Language Hardcoding)  │
│     • 24XX risk-resolution state machine (seeded RNG, pure Python)   │
│     • Three-pass agent contracts (Referee, Shadow Coach, Promoter)   │
│     • Spaced-retrieval selector & Tier 1 staging validator           │
├─────────────────────────────────┬────────────────────────────────────┤
│  2. Language Pack               │  3. Setting Cartridge (Read-Only)  │
│     • ISO language code         │     • Parent vault catalog.json    │
│     • CEFR profiles (A2–C1)     │       (oude-mechanismen Tier 1)    │
│     • Coach error taxonomy      │     • Canonical language metadata  │
│     • Register & scaffolding    │     • Per-language term glossary   │
└─────────────────────────────────┴────────────────────────────────────┘
```

---

## Three-Pass Turn & Session Loop

```text
  player turn (target language)
        │
        ├──► [Pass 1: Referee] (in-character, target language, CEFR-bounded)
        │         • optional Companion only when toggled on or addressed
        │         • deterministic 24XX roll when action carries risk (1–2 / 3–4 / 5+)
        │         • emits scene outcome + state delta
        │
        └──► [Pass 2: Shadow Coach] (out-of-character, silent post-turn)
                  • analyses learner utterance only (never breaks fiction)
                  • records structured corrections, idioms, and error tags
                  • updates spaced-retrieval ledger for future Referee prompts

  session end
        └──► [Pass 3: Promoter] (distil + translate + glossary check)
                  • converts canon-altering outcomes into parent canonical language
                  • links affected Tier 1 catalog paths and SHAs
                  • writes spoiler-safe proposal bundle to exports/ (staging only)
```

---

## Project Governance & Development

Development follows the **Elephant-Goldfish Model (EGM)** and strict Test-Driven Development (TDD).
See [AGENTS.md](AGENTS.md) for architectural boundaries, specification gates, and branch protection rules.
See [HANDOFF.md](HANDOFF.md) for current milestone context and continuation steps.

---

## Attribution & Licences

Lingua Ludi uses a dual-licence structure; see [LICENSE](LICENSE) for full terms:

- **Source Code & Software:** MIT Licence.
- **Game Mechanics, Language Packs, Prompt Contracts & Documentation:** Creative Commons Attribution 4.0 International (CC BY 4.0).

24XX rules are CC BY 4.0 Jason Tocci (<https://jasontocci.itch.io/24xx>).

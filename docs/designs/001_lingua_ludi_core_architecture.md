# Design Specification: Lingua Ludi Core Architecture

**Document ID:** `docs/designs/001_lingua_ludi_core_architecture.md`

| Metadata | Value |
| :--- | :--- |
| Author | `efsalvarenga` |
| Date | 2026-11-27 |
| Spec Version | 0.2.2 |
| Status | Approved (Human Sign-Off: `efsalvarenga`, 2026-11-27) |
| Target Product Milestone | `v0.1.0` (Zero-Python Multi-Persona Skill Kit, Language Packs & Campaign-Repo Staging) |

## Changelog

| Spec Version | Summary |
| :--- | :--- |
| 0.2.2 | Approved 2026-11-27 by `efsalvarenga` after resolving Round 2 zero-context Goldfish EGM review (`omp` / `mistral/mistral-large-latest`, `high`): pinned session-start crash-recovery and `pending_coach_turns` drain ordering (`R2-M1`); mandated `git check-ignore -q _gm-materials/` semantics and defined non-Git standalone warning behaviour (`R2-M2`); corrected `catalog.json` consumed field count from six to five (`R2-M3`); aligned `canonical_language` fallback condition between §3.2 and §4.2 (`R2-M4`); removed maintainer-attribution prefixes in §6 and added external SRS and portable prompt-bundle rows (`R2-M5`); deferred Tier 3 no-subagent fallback to `v0.2.0` in https://github.com/terra-campaigns/lingua-ludi/issues/3 so `lingua-coach` is unconditionally the sole writer of `ledger.md` (`R2-M6`); unified `pending_coach_turns` and `observed_source_sha`, narrowed meta-leak scan markers to structural tokens with warn-and-annotate behaviour, added missing Language Pack abort, pinned OOC regex, and aligned §9.1 review links. |
| 0.2.1 | Resolved Round 1 external Goldfish EGM review findings (`M1–M12`, `N2–N9`; `N1` declined by author, `M6` deferred to `v0.2.0` in https://github.com/terra-campaigns/lingua-ludi/issues/2): defined subagent harness tiers, race-free `ledger.md` reads, bounded close drain and crash recovery, `.active_session` concurrency guard, per-persona context ceilings, untrusted-data framing, shell meta-leak scan, `.gitignore` hard-abort behaviour, consistent retrieval count, `canonical_language` precedence, and `log.md`/`sessions/` file contracts. |
| 0.2.0 | Replaced the Python state-machine draft with a zero-Python, Markdown-and-skills harness installed alongside `oude-mechanismen` and executed inside host campaign repositories, using player-owned 24XX mechanics, multi-persona subagents, and strict write ownership under `_gm-materials/lingua-ludi/<campaign>/`. |
| 0.1.0 | Initial Elephant draft (commit `46f21cf`) proposing a pure-Python (`pydantic` v2 / `uv`) state-transformer engine, deterministic seeded RNG, and 7-command CLI run from the `lingua-ludi` repository. Superseded before Goldfish review due to unnecessary turn-loop overhead and single-voice play limitations. |

---

## 1. Overview & Problem Statement

### 1.1 The Problem

Practising a European language through solo tabletop roleplaying requires fast turn cycles and direct access to an existing campaign repository.
Prior single-skill experiments — `play-uvg` in [`terra-campaigns/charged`](https://github.com/terra-campaigns/charged) and `dutch-tutor` in [`terra-campaigns/hostile`](https://github.com/terra-campaigns/hostile) — showed that plain Markdown state files (`state.md`, `log.md`), player-rolled dice, and one prompt per turn avoid engine round-trip overhead, while exposing three recurring limitations:

1. **Single-voice play:** A single monolithic GM-plus-tutor prompt speaks in one voice, so the player does not experience distinct table companions or independent NPC perspectives.
2. **Tight coupling to one setting and one language:** Each experiment hardcoded a single campaign universe and a single target language inside its skill file.
3. **Manual canon and pedagogy hygiene:** Inline grammar corrections interrupt transcripts and risk leaking out-of-character coaching metadata into published campaign logs.
   Separately, promoting cross-language lore back to a host campaign vault lacks a structured glossary and drift-check staging workflow.

### 1.2 Overview

Lingua Ludi addresses these limitations as a **zero-Python, setting-agnostic, language-agnostic skill and subagent persona kit** installed once (mirroring [`oude-mechanismen`](https://github.com/terra-campaigns/oude-mechanismen)) and executed directly inside any host campaign repository using a subagent-capable agent harness.

```text
  [lingua-ludi kit (installed read-only at $LINGUA_LUDI_ROOT)]
     ├── skills/      (lingua-init, lingua-play, lingua-promote)
     ├── agents/      (lingua-wingman, lingua-npc, lingua-coach, lingua-promoter)
     ├── languages/   (reusable CEFR + error-taxonomy Language Packs, e.g. nl)
     └── templates/   (campaign.yml, state.md, log.md, ledger.md, glossary, export)
                            │
                            ▼  invoked inside a host campaign repo (vault)
  [host campaign repo (e.g. an oude-mechanismen vault)]
     ├── .agents/AGENTS.md                        (read-only setting tone & canon language)
     ├── .agents/context/catalog.json             (read-only Tier 1 index + SHAs)
     ├── .agents/lingua-ludi/glossary.<iso>.yml   (tracked per-language proper-noun rules)
     ├── campaigns/, directory/, systems/         (read-only Tier 1 canon; human-promoted only)
     └── _gm-materials/lingua-ludi/<campaign>/    (gitignored active play workspace)
            ├── .active_session                   (transient single-session concurrency lock)
            ├── campaign.yml                      (language, CEFR, companion & NPC settings)
            ├── state.md                          (24XX character sheet, scene, World Events)
            ├── log.md                            (append-only narrative session summaries)
            ├── ledger.md                         (spaced-retrieval error & vocab table)
            ├── sessions/<session_id>.md          (unpublished turn transcript)
            └── exports/<session_id>/             (staged canonical-language Tier 1 proposals)
```

### 1.3 Goals

* **Zero-Python Gameplay:** Keep play in chat and human-readable Markdown files (`state.md`, `log.md`, `ledger.md`), with player-owned [24XX](https://jasontocci.itch.io/24xx) dice rolls and zero runtime build or CLI overhead.
* **Distinct Table Personas via Subagents:** Run the GM as the main session while delegating to a persistent **Wingman** subagent (friendly tutor buddy), selective **Spotlight NPC** subagents (independent voice and perspective), an asynchronous background **Shadow Coach** subagent, and a fresh session-end **Promoter** subagent.
* **Strict Three-Layer Separation:** Keep generic skills and personas (`lingua-ludi`), reusable Language Packs (`languages/<iso>/`), and host campaign setting canon and glossaries decoupled so any language pack pairs with any host campaign repository.
* **Campaign-Repo Execution Without Vault Pollution:** Store all mutable session files, personal learner ledgers, and staged exports under `_gm-materials/lingua-ludi/<campaign>/` (excluded by default in template-built `oude-mechanismen` vaults and enforced via `git check-ignore` during `lingua-init`).
* **Closed-Loop Spaced Retrieval in Plain Markdown:** Track errors and vocabulary in `ledger.md` via the Shadow Coach and surface up to `max_retrieval_targets` due targets (default `4`, range `0..5`) at session start without breaking fiction.
* **Safe Cross-Language Tier 1 Promotion Staging:** Distil canon-altering outcomes into the host vault's canonical language, check glossary rules and Tier 1 `catalog.json` paths/SHAs, strip `[OOC]` blocks, exclude `ledger.md`, and run a shell meta-leak scan before staging Markdown proposals for human review without mutating published Tier 1 notes.

### 1.4 Non-Goals & Deferred Scope

The following items are out of scope for `v0.1.0`:

* Python runtime engine, Pydantic schemas, seeded RNG code, gameplay CLI, or standalone validator scripts (deferred until playtesting demonstrates a mechanical validation failure that warrants adding a Python toolchain).
* Playing campaigns inside the `lingua-ludi` repository itself.
* Hardcoding any specific European language or any specific campaign setting inside core skills, persona definitions, or templates.
* Automated direct edits, commits, or pull requests against Tier 1 vault directories (`campaigns/`, `directory/`, `systems/`).
* Mechanical penalties for grammatical mistakes during play (the GM interprets learner intent charitably; the Shadow Coach logs errors silently).
* In-fiction (diegetic) non-native character quotes in staged narrative exports (deferred to `v0.2.0` and tracked in https://github.com/terra-campaigns/lingua-ludi/issues/2).
* Single-context no-subagent fallback mode and portable prompt-bundle compilation for plain web or mobile chat clients (deferred to `v0.2.0` and tracked in https://github.com/terra-campaigns/lingua-ludi/issues/3; `v0.1.0` requires a subagent-capable harness).
* Macro faction turn execution (such as integrating [TOME](https://github.com/efsalvarenga/tome) as a background geopolitical engine); `v0.1.0` reserves a plain Markdown `## World Events` section in `state.md` so manual or future macro outputs can be pasted in without changing templates.
* Audio input/output (local Whisper STT or TTS pipelines), web UI, hosted deployment, authentication, and multi-player networking.

### 1.5 Terminology, External Dependencies & Harness Interface

* **Vault / Host Campaign Repository:** Used interchangeably for the user's campaign repository where Lingua Ludi skills are invoked (for example an `oude-mechanismen` Obsidian/Jekyll vault or a standalone Markdown campaign directory).
* **Tier 1 Canon:** Published campaign notes residing under `campaigns/`, `directory/`, and `systems/` (indexed with `tier: 1` in `.agents/context/catalog.json` when using `oude-mechanismen`).
* **24XX:** Micro-scene tabletop ruleset from the [24XX SRD](https://jasontocci.itch.io/24xx) (CC BY 4.0 Jason Tocci), detailed in §3.4.
* **CEFR:** [Common European Framework of Reference for Languages](https://www.coe.int/en/web/common-european-framework-reference-languages) (`A2`, `B1`, `B2`, `C1`), used to bound vocabulary ceiling, sentence complexity, and register guidance in Language Packs.
* **BCP-47 & `<iso>`:** [BCP-47](https://www.rfc-editor.org/info/bcp47) language tags (such as `nl` for Dutch or `en-GB` for British English); the `<iso>` path segment in `languages/<iso>/` and `glossary.<iso>.yml` is the lowercased primary language subtag (e.g. `nl`).
* **Neutral Placeholders & EGM:** Core sections use neutral placeholders (`Setting A`, `Target Language X`, `Canonical Language Y`, `Actor A`, `Place A`, `Term A`, `campaign-a`) to stay setting-agnostic and language-agnostic; design reviews follow the Elephant-Goldfish Model (EGM), where zero-context reviewer subagents ("Goldfishes") audit specifications drafted by the persistent author context ("Elephant").
* **External Dependencies:**
  * [`oude-mechanismen`](https://github.com/terra-campaigns/oude-mechanismen) — Jekyll/Obsidian campaign-vault engine that defines Tier 1 canon (`campaigns/`, `directory/`, `systems/`), `.agents/AGENTS.md` (setting tone and canonical language), `.agents/context/catalog.json` (generated by `jekyll/_plugins/data_extractor.rb`), and default `_gm-materials/` exclusions (verified in `data_extractor.rb`, `scripts/lint_wiki.py`, and `templates/jekyll/_config.yml.template`).
  * [`terra-campaigns/charged`](https://github.com/terra-campaigns/charged) (`play-uvg` skill) — prior single-skill experiment contributing the player-owned dice and plain Markdown `state.md`/`log.md` pattern.
  * [`terra-campaigns/hostile`](https://github.com/terra-campaigns/hostile) (`dutch-tutor` skill) — prior single-skill experiment contributing the `OOC:` pause/resume rule and unpublished-transcript-to-published-narrative promotion pattern.
* **Harness Skill & Subagent Interface:**
  * A **skill** is a Markdown procedure (`SKILL.md`) registered in an agent harness and invoked by name (e.g. `/lingua-play` or `lingua-play`).
  * A **subagent** is an isolated worker context spawned by the host harness from an agent definition file (`agents/*.md`).
  * **Persistent** means the host harness assigns the subagent a stable conversation ID so the main session can send follow-up messages across turns without re-spawning it; **scene-scoped** means the subagent stays persistent for one scene and is retired when the scene changes.
  * **Async dispatch with completion tracking** means the host harness can deliver a message to `lingua-coach` without blocking the main session's reply and later notify or report whether that worker has finished processing pending messages.

---

## 2. Critical User Journeys (CUJs)

### 2.1 CUJ 1: Initialise a Language-Learning Campaign Inside a Host Vault

* **Given:** A host campaign repository (with or without `oude-mechanismen` `.agents/context/catalog.json` and `.agents/AGENTS.md`) and an installed `lingua-ludi` kit containing the requested Language Pack (`Target Language X`).
* **When:** The user invokes `lingua-init` inside the host campaign repository, supplying a campaign slug (`campaign-a`), `target_language` (`Target Language X`), `feedback_language` (e.g. `Canonical Language Y`), CEFR level (`B1`), and Wingman policy (`on_request` or `active`).
* **Then:** `lingua-init` first checks that the requested Language Pack exists under `$LINGUA_LUDI_ROOT/languages/<iso>/pack.md`; if missing, it aborts without writing any files and lists available packs.
  Next, if inside a Git repository (`git rev-parse --is-inside-work-tree`), `lingua-init` runs `git check-ignore -q _gm-materials/`; if `_gm-materials/` is not ignored, it aborts without creating any play files, prints `_gm-materials/` for the user to add manually, and never edits `.gitignore` itself.
  If the directory is not a Git repository, `lingua-init` prints an explicit warning that no Git ignore protection is active and proceeds.
  `lingua-init` derives `canonical_language` from `.agents/AGENTS.md` (prompting for a fallback only when `.agents/AGENTS.md` is absent or specifies no canonical language).
  It creates `_gm-materials/lingua-ludi/campaign-a/` populated from templates (`campaign.yml`, `state.md`, `log.md`, `ledger.md`, `sessions/`, `exports/`), creates `.agents/lingua-ludi/glossary.<iso>.yml` if absent, and leaves all Tier 1 vault files untouched.

### 2.2 CUJ 2: Routine Play Turn with Background Shadow Coaching

* **Given:** An active `lingua-play` session in `campaign-a` on a Tier 1 subagent harness, with `state.md`, up to `max_retrieval_targets` (default `4`) due targets loaded from `ledger.md` at session start, and a persistent `lingua-coach` subagent running.
* **When:** The player submits a routine action in `Target Language X` (containing a subtle grammatical error) that does not address a Spotlight NPC or invoke the Wingman.
* **Then:** The GM interprets the player's intent charitably and narrates the scene outcome directly in `Target Language X` within the active CEFR profile, weaving in a loaded retrieval target.
  The GM updates `state.md` if scene state changed and appends the turn to `sessions/<session_id>.md`.
  The GM dispatches the player's utterance and preceding scene prompt asynchronously to `lingua-coach`, returning to the player without waiting while `lingua-coach` updates `ledger.md` in the background.

### 2.3 CUJ 3: Spotlight NPC Interaction, Wingman OOC Help, and Player-Rolled 24XX Risk

* **Given:** An active scene in `Place A` featuring named Tier 1 NPC `Actor A`.
* **When:** The player asks `(How do I say "pressure seal"?)` (`OOC:` pause), then addresses `Actor A` in `Target Language X` to attempt a risky negotiation, rolls their 24XX skill die (`d8` → `3`, setback), and reports the roll.
* **Then:** On the `OOC:` line, fiction pauses (`state.md` stays unchanged and `lingua-coach` is not triggered).
  The persistent `lingua-wingman` answers concisely in `feedback_language` with a phrasing cue, and the GM reprints the current scene prompt.
  On the negotiation turn, the GM states the risk and accepts the player's `setback` result.
  The GM messages the scene's `lingua-npc` subagent (briefed from `Actor A`'s Tier 1 note and private agenda) to obtain `Actor A`'s in-character dialogue and visible reaction.
  The GM integrates the complication into `state.md`, appends the turn to `sessions/<session_id>.md`, and dispatches the player's utterance asynchronously to `lingua-coach`.

### 2.4 CUJ 4: Session End and Cross-Language Tier 1 Lore Staging

* **Given:** A completed play session in `campaign-a` where a canon-altering event occurred involving `Actor A` and `Place A`, and `ledger.md` contains personal learner error records.
* **When:** The user ends the session and invokes `lingua-promote`.
* **Then:** The GM executes the bounded drain protocol for any pending `lingua-coach` turn, appends a narrative summary to `log.md`, and removes `.active_session`.
  `lingua-promote` launches a fresh `lingua-promoter` subagent with only the current `sessions/<session_id>.md` (with `[OOC]` blocks stripped), the most recent `3` summaries from `log.md`, `.agents/AGENTS.md`, `.agents/lingua-ludi/glossary.<iso>.yml`, and Tier 1 entries (`tier == 1`) from `.agents/context/catalog.json` — never `ledger.md` or coaching recaps.
  `lingua-promoter` writes `_gm-materials/lingua-ludi/campaign-a/exports/<session_id>/tier1_proposal.md` in `Canonical Language Y`, citing exact `path` and `observed_source_sha` values and preserving `keep_untranslated` glossary terms.
  `lingua-promote` runs a structural meta-leak scan (where shell tools exist) before confirming staging and leaves all files in `campaigns/`, `directory/`, and `systems/` untouched.

### 2.5 CUJ 5: Standalone Directory, Tier 2 Harness, or Unsupported No-Subagent Harness

* **Given:** A user running in a standalone campaign directory without `oude-mechanismen` (`catalog.json` absent, possibly non-Git), in a Tier 2 harness whose subagents lack completion tracking, or in a harness with no subagent tools at all.
* **When:** The user invokes `lingua-init`, `lingua-play`, and `lingua-promote`.
* **Then:** Without `catalog.json`, the GM reads local campaign Markdown notes directly and `lingua-promote` records `observed_source_sha: n/a (no catalog.json)`.
  In a non-Git directory, `lingua-init` prints a warning that Git ignore protection is inactive and proceeds.
  In a Tier 2 harness (subagents exist without completion tracking), `lingua-play` runs `lingua-wingman`, `lingua-npc`, and `lingua-promoter` as subagents while degrading `lingua-coach` from background async to a synchronous session-end subagent pass so no turns are dropped.
  In a harness with no subagent tools at all, `lingua-play` refuses cleanly and states that `v0.1.0` requires a subagent-capable harness (single-context play is deferred to `v0.2.0` in https://github.com/terra-campaigns/lingua-ludi/issues/3).

---

## 3. Technical Architecture

### 3.1 Three Orthogonal Layers & Repository Topology

Lingua Ludi separates generic harness procedures, reusable Language Packs, and host campaign setting data:

1. **Lingua Ludi Kit (`terra-campaigns/lingua-ludi`):**
   * **Discovery precedence:** Resolved from the environment variable `LINGUA_LUDI_ROOT` (absolute path to the read-only kit root directory containing `skills/`, `agents/`, `languages/`, and `templates/`), falling back to `$HOME/.local/share/lingua-ludi` when `LINGUA_LUDI_ROOT` is unset.
   * `skills/lingua-init/SKILL.md` — verifies Language Pack existence and `.gitignore` exclusion, then scaffolds a campaign workspace inside the host campaign repository.
   * `skills/lingua-play/SKILL.md` — GM session loop, `.active_session` guard, startup recovery/drain sequence, subagent orchestration protocol, 24XX risk handling, `OOC:` pause handling, and session closing.
   * `skills/lingua-promote/SKILL.md` — session-end cross-language canonicalisation, shell meta-leak scan, and Tier 1 staging workflow.
   * `agents/lingua-wingman.md` — persistent companion/tutor buddy persona contract.
   * `agents/lingua-npc.md` — on-demand Spotlight NPC persona contract.
   * `agents/lingua-coach.md` — Shadow Coach persona contract (sole writer of `ledger.md`).
   * `agents/lingua-promoter.md` — fresh session-end Promoter persona contract.
   * `languages/<iso>/pack.md` — reusable Language Packs (YAML front matter + CEFR profiles `A2`–`C1`, register guidance, error taxonomy, scaffolding cues).
   * `templates/` — starter templates for `campaign.yml`, `state.md`, `log.md`, `ledger.md`, `glossary.yml`, and `tier1_proposal.md`.
2. **Language Pack Layer (`languages/<iso>/`):**
   * Declares BCP-47 `language_code` (e.g. `nl`), CEFR ceiling profiles (`A2`, `B1`, `B2`, `C1`), register guidance, closed Coach error-category tags (e.g. `word_order`, `gender_article`, `verb_conjugation`, `preposition`, `lexical_choice`, `register`, `idiom`, `spelling`), and optional Wingman scaffolding phrases.
   * Contains zero campaign lore or setting proper nouns.
   * If `lingua-init` is called with an `<iso>` that has no matching `languages/<iso>/pack.md`, it aborts without writing files and prints the list of installed packs.
3. **Host Campaign Repository (Vault & Local Play Workspace):**
   * **Read-only canon sources:** `.agents/AGENTS.md` (authoritative setting tone, voice, and canonical language), `.agents/context/catalog.json` (when `oude-mechanismen` is present), and lazy-loaded Tier 1 notes (`campaigns/`, `directory/`, `systems/`).
   * **Tracked setting-language mapping:** `.agents/lingua-ludi/glossary.<iso>.yml` — per-language proper nouns and jargon (`keep_untranslated` vs `preferred_translation`).
     Inspection of `oude-mechanismen`'s `jekyll/_plugins/data_extractor.rb` confirms `.agents/` is excluded from Jekyll builds and `catalog.json`, so this file stays tracked in Git without entering published pages.
   * **Gitignored local play workspace:** `_gm-materials/lingua-ludi/<campaign>/` — excluded by default in template-built `oude-mechanismen` vaults (verified in `data_extractor.rb`, `lint_wiki.py`, and `_config.yml.template`) and enforced across Git host repositories by `lingua-init`'s `git check-ignore -q _gm-materials/` hard-abort check.

### 3.2 Multi-Persona Subagent Topology, Turn Cadence & Context Budgets

```text
  Player Turn (Target Language X)
       │
       ▼
  ┌─────────────────────────────────────────────────────────────────────┐
  │  GM (Main Session — skills/lingua-play/SKILL.md)                    │
  │  • Owns scene narration, environment, minor extras, stakes, 24XX    │
  │  • Sole writer of: .active_session, state.md, log.md, sessions/     │
  └───────┬──────────────────────────┬──────────────────────────┬───────┘
          │                          │                          │
          │ on OOC / request /       │ only when PC engages     │ async dispatch
          │ active stall beat        │ named Spotlight NPC      │ (or session-end in
          ▼                          ▼ (max max_active_npcs,    │  Tier 2 harness)
  ┌──────────────────┐      ┌──────────default 2)───────┐       ▼
  │ lingua-wingman   │      │ lingua-npc                │ ┌──────────────────────┐
  │ (persistent)     │      │ (scene-scoped)            │ │ lingua-coach         │
  │ • dialogue only  │      │ • dialogue only           │ │ (persistent)         │
  │ • zero writes    │      │ • private agenda          │ │ • current turn +     │
  │ • CEFR-bounded   │      │ • zero writes             │ │   ledger.md only     │
  └──────────────────┘      └───────────────────────────┘ │ • sole writer of     │
                                                          │   ledger.md          │
                                                          └──────────────────────┘

  Session End (skills/lingua-promote/SKILL.md)
       │
       ▼  current session ([OOC] stripped) + last 3 log.md summaries (NEVER ledger.md)
  ┌─────────────────────────────────────────────────────────────────────┐
  │  lingua-promoter (fresh subagent)                                   │
  │  • Reads: sessions/<session_id>.md (no OOC), last 3 log.md entries, │
  │           catalog.json (tier == 1), .agents/AGENTS.md,              │
  │           .agents/lingua-ludi/glossary.<iso>.yml                    │
  │  • Sole writer of: exports/<session_id>/tier1_proposal.md           │
  └─────────────────────────────────────────────────────────────────────┘
```

#### Role Contracts, Harness Capability Tiers & Context Ceilings

* **Supported Harness Capability Tiers (`v0.1.0`):**
  1. *Tier 1 — Full Subagent Harness* (subagent spawn + persistent messaging + completion notification/status check): runs persistent `lingua-wingman`, scene-scoped `lingua-npc`, async background `lingua-coach`, and fresh `lingua-promoter`.
  2. *Tier 2 — Subagents Without Completion Tracking* (spawn/messaging available, no await/status check): runs `lingua-wingman`, `lingua-npc`, and `lingua-promoter` as subagents, and runs `lingua-coach` as a synchronous session-end subagent pass so no turns are dropped at session close.
  * *Unsupported (No Subagent Tools):* `lingua-play` refuses cleanly and informs the user that `v0.1.0` requires a subagent-capable harness (deferred to `v0.2.0` in https://github.com/terra-campaigns/lingua-ludi/issues/3).
* **Session-Start Ordering (Pinned):** On `lingua-play` start, the GM executes steps in this exact order:
  1. Verify `.active_session` is absent (if present, halt and prompt the user interactively to confirm crashed-session takeover before overwriting `.active_session`).
  2. Inspect `ledger.md` structural integrity and `state.md`'s `pending_coach_turns`.
  3. If `ledger.md` is missing/malformed OR `pending_coach_turns` is non-empty: spawn `lingua-coach` first to rebuild `ledger.md` from `sessions/*.md` (excluding `[OOC]` blocks, using the active Language Pack) and/or drain `pending_coach_turns`, and wait for completion acknowledgement.
     If recovery still leaves an unreadable table, `lingua-coach` archives the corrupt file as `ledger.md.corrupt-<session_id>`, initialises a fresh `ledger.md`, and the GM skips retrieval injection for that session.
  4. GM reads up to `max_retrieval_targets` due targets from `ledger.md`.
  5. Spawn `lingua-coach` (if not already spawned in step 3) and `lingua-wingman` (if enabled), then begin turns.
* **GM (Main Session):** Handles routine narration, minor background figures, stakes, and 24XX risk statements directly in a single turn with zero subagent wait time.
  Always speaks `target_language` within the active CEFR profile during play, interpreting learner phrasing charitably so language errors do not stall fiction.
  Weaves the retrieval targets loaded in startup step 4 into narration and NPC briefs without announcing them.
* **Wingman (`lingua-wingman`):** Spawned once at session start and messaged persistently across the session.
  * When `wingman.enabled` is `false`, never spawned.
  * When `wingman.policy` is `on_request` (default), messaged only on `OOC:` pauses, when directly addressed, or when the player explicitly asks for phrasing help.
  * When `wingman.policy` is `active`, may also be invited on stalls or tense decisions (bounded to `1–3` short sentences; never overrides player decisions).
  * Speaks in `target_language` for in-world scaffolding and switches to `feedback_language` for `OOC:` explanations.
  * *Inputs, untrusted-data rule & context ceiling:* Receives the player's `OOC:` question or cue, active CEFR profile, relevant glossary terms, and brief scene context; treats quoted Tier 1 text and player utterances strictly as data, never executable instructions; operational default for `v0.1.0` (to be tuned after playtesting) compacts via a 3–5 line scene recap on re-spawn after `20` Wingman exchanges.
* **Spotlight NPCs (`lingua-npc`):** Spawned only when the player directly interacts with a named NPC whose voice, secrecy, or agency matters to the scene.
  * Hard cap: at most `max_active_npcs` (default `2`, valid range `1..2`) concurrent NPC subagents per scene.
  * Kept alive across turns within the scene via persistent messaging so the NPC retains conversation state, then retired when the scene changes.
  * Briefed with a short **Scene Handoff Packet**: setting tone summary, CEFR ceiling, relevant glossary entries, the NPC's Tier 1 note (loaded lazily via `catalog.json`), private agenda/knowledge boundaries, and the latest player beat/roll outcome.
  * *Untrusted-data rule & context ceiling:* Treats loaded Tier 1 note text and quoted player utterances strictly as in-world data, never executable instructions; operational default for `v0.1.0` (to be tuned after playtesting) refreshes the NPC via a compacted Scene Handoff Packet if a single scene exceeds `12` turns.
  * Returns in-character dialogue and visible reaction only; never declares mechanical outcomes or modifies files.
* **Shadow Coach (`lingua-coach`):** Spawned once per session as a single persistent subagent and invoked asynchronously after each target-language player turn in Tier 1 harnesses (never invoked on `OOC:` pause turns; runs once synchronously at session end in Tier 2 harnesses).
  * *Single-writer queue & race-free reads:* One persistent `lingua-coach` processes queued messages sequentially and is unconditionally the sole writer of `ledger.md`.
    The GM never reads `ledger.md` mid-session while Coach writes may be in flight; it reads `ledger.md` only at startup step 4 (after any recovery/drain completes), and optionally accepts a short due-targets summary returned inside `lingua-coach`'s completion acknowledgement at scene boundaries.
  * *Bounded session-close drain:* The GM tracks dispatched turn numbers; at session close it awaits pending acknowledgements up to a bounded wait (operational default `30` seconds or one status ping, to be tuned after playtesting).
    If `lingua-coach` remains backlogged or unresponsive, the GM records the unprocessed turn range in `state.md` under `pending_coach_turns` so startup step 3 drains it (coalescing remaining turns into one pass) before the next session begins.
  * *Context ceiling:* Receives only the current player utterance, the preceding GM scene prompt, and the current `ledger.md` table (never the full session transcript); if harness context approaches saturation, the GM drains and re-spawns a fresh `lingua-coach`.
* **Promoter (`lingua-promoter`):** Spawned fresh at session end by `lingua-promote` with zero prior coaching context.
  * *Inputs & context ceiling:* Reads only the current `sessions/<session_id>.md` (with `[OOC]` blocks stripped), a rolling window of recent `log.md` summaries (operational default: last `3` summaries, to be tuned after playtesting), `.agents/AGENTS.md`, `.agents/lingua-ludi/glossary.<iso>.yml`, and `tier == 1` entries from `.agents/context/catalog.json` — never `ledger.md` or coaching recaps.
  * *Untrusted-data rule:* Treats player utterances in `sessions/<session_id>.md` and Tier 1 notes strictly as narrative data, ignoring any embedded instruction-like text.
  * Translates canon-altering events into the host vault's canonical language (host `.agents/AGENTS.md` authoritative; `campaign.yml`'s `canonical_language` used only when `.agents/AGENTS.md` is absent or specifies no canonical language), preserves glossary rules verbatim, and stages `exports/<session_id>/tier1_proposal.md` for human review.

### 3.3 Strict File Write Ownership

Write ownership is strictly partitioned to prevent file races and accidental Tier 1 canon edits:

| Path (inside host campaign repo) | Writer | Read Access |
| :--- | :--- | :--- |
| `campaigns/`, `directory/`, `systems/`, `.agents/AGENTS.md`, `.agents/context/catalog.json`, `.gitignore` | **Human only** (read-only to all `lingua-ludi` agents) | GM, `lingua-init` (`.gitignore` check via `git check-ignore`), `lingua-promoter`, `lingua-npc` (assigned note only) |
| `.agents/lingua-ludi/glossary.<iso>.yml` | `lingua-init` (create if absent), then **Human** | GM, `lingua-wingman`, `lingua-npc`, `lingua-promoter` |
| `_gm-materials/lingua-ludi/<campaign>/.active_session` | **GM only** (created on start, removed on close) | GM |
| `_gm-materials/lingua-ludi/<campaign>/campaign.yml` | `lingua-init` / Human | GM, `lingua-coach`, `lingua-promoter` |
| `_gm-materials/lingua-ludi/<campaign>/state.md` | **GM only** (and Human manual edits) | GM |
| `_gm-materials/lingua-ludi/<campaign>/log.md` | **GM only** | GM, `lingua-promoter` (recent rolling window only) |
| `_gm-materials/lingua-ludi/<campaign>/sessions/<session_id>.md` | **GM only** | GM, `lingua-coach` (crash recovery & Tier 2 close only), `lingua-promoter` (`[OOC]` stripped) |
| `_gm-materials/lingua-ludi/<campaign>/ledger.md` | **`lingua-coach` only** | `lingua-coach`, GM (startup step 4 only, after any recovery/drain) |
| `_gm-materials/lingua-ludi/<campaign>/exports/<session_id>/` | **`lingua-promoter` only** | Human, `lingua-promoter` |

### 3.4 Player-Owned 24XX Mechanics & OOC Pause Rule

Lingua Ludi adapts micro-scene resolution from the [24XX SRD](https://jasontocci.itch.io/24xx) (CC BY 4.0 Jason Tocci), following `play-uvg`'s division of labour (player owns dice and mechanical state; GM narrates fiction):

* **Character Sheet in `state.md`:** Plain Markdown listing `Name`, `Concept`, `Skills` (`d6` untrained/baseline, `d8`/`d10`/`d12` skilled, `d4` hindered), `Inventory`, `Hindrances`, `Current Location`, `Open Stakes`, `pending_coach_turns` (empty when drained), and `World Events`.
* **Player-Owned Rolls:** Routine actions succeed without a roll.
  When an action carries risk, the GM states the risk in `target_language` (what goes wrong on failure) and suggests the skill die and mode (`normal`, `hindered` = `d4`, `advantage` = roll 2 keep highest, `disadvantage` = roll 2 keep lowest).
  The player rolls physical or external dice and reports the kept result (or explicitly asks the GM to roll):
  * `1–2` (**Disaster**): action fails and the stated risk occurs.
  * `3–4` (**Setback**): action succeeds with a complication, cost, or partial result.
  * `5+` (**Success**): action succeeds cleanly.
* **OOC Pause Rule (adapted from `dutch-tutor`):** An `OOC:` pause triggers **only** when the trimmed player message matches one of three exact rules: starts with `OOC:` (case-insensitive), matches `^\([^()]*\)$`, or matches `^\[[^\[\]]*\]$`.
  Anything else — including embedded parentheticals such as `(lacht) Ik ga naar binnen` or paired segments like `(a) (b)` — stays in-character and does not pause the scene.
  On a valid OOC pause:
  1. Do not advance the scene, resolve actions, mutate `state.md`, or trigger `lingua-coach`.
  2. Route the language/vocabulary question to `lingua-wingman` (or answer directly as GM if Wingman is disabled) concisely in `feedback_language`.
  3. Record the exchange as an `> [OOC]` block in `sessions/<session_id>.md` (if logging verbatim) and reprint the current scene prompt so the player can resume.

### 3.5 Spaced Retrieval & Crash Recovery (`ledger.md`)

`ledger.md` stores a Markdown table maintained exclusively by `lingua-coach`:

| Column | Meaning |
| :--- | :--- |
| `item_id` | Stable slug (e.g. `vocab-term-a` or `err-word-order-01`) |
| `surface_form` | Target-language word, phrase, or pattern |
| `category` | Error tag from the active Language Pack taxonomy (or `vocab`) |
| `errors` | Integer count of observed errors |
| `clean_uses` | Integer count of confirmed clean learner uses |
| `last_turn` | Session/turn marker (e.g. `2026-11-26-001:T04`) |
| `interval` | Current turn interval (`1`, `2`, `4`, `8`, or `16`) |
| `next_due` | Turn number at which the item becomes due again |
| `note` | Concise `feedback_language` hint or corrected form |

* **Interval rule:** New error sets `interval = 1`, `next_due = current_turn + 1`.
  Each subsequent clean learner use advances `interval` along `1 → 2 → 4 → 8 → 16`; a repeated error resets `interval` to `1` and increments `errors`.
* **Selection rule (canonical):** At startup step 4 (after any recovery or `pending_coach_turns` drain has completed), the GM reads `ledger.md` and selects up to `max_retrieval_targets` rows (default `4`, valid range `0..5`; `0` disables retrieval injection) where `next_due <= current_turn`, prioritising most overdue first, then highest `errors`.
  Zero-error turns are valid and still advance `clean_uses` for tracked items used correctly.
* **Crash recovery:** Executed at startup step 3 when `ledger.md` is missing, truncated, or structurally malformed: `lingua-coach` rebuilds the table by replaying `sessions/*.md` (excluding `[OOC]` blocks) against the active Language Pack taxonomy; if recovery fails to produce a valid table, `lingua-coach` archives the corrupt file as `ledger.md.corrupt-<session_id>`, initialises a fresh `ledger.md`, and the GM skips retrieval injection for that session.

### 3.6 `oude-mechanismen` `catalog.json` Compatibility & Tier 1 Export Staging

The full schema of `.agents/context/catalog.json` is owned by [`oude-mechanismen/jekyll/_plugins/data_extractor.rb`](https://github.com/terra-campaigns/oude-mechanismen/blob/main/jekyll/_plugins/data_extractor.rb).
Lingua Ludi relies only on the `entries` array and five per-entry fields: `path`, `title`, nullable `summary`, 8-character hex `sha`, and `tier` (noting that `data_extractor.rb` indexes `.agents/context/rolling_glossary.md` with `tier: 2` alongside `tier: 1` canon pages).
`rolling_glossary.md` is `oude-mechanismen`'s single-language campaign canon glossary and is distinct from `.agents/lingua-ludi/glossary.<iso>.yml`, which defines cross-language `keep_untranslated` vs `preferred_translation` policy; Lingua Ludi treats `rolling_glossary.md` as read-only context if consulted and excludes it from Tier 1 update proposals.

`lingua-play` and `lingua-promote` enforce four rules when reading `catalog.json` and staging exports:

1. **Tier 1 Filtering:** Filter `entries` to `tier == 1` for canon grounding and Tier 1 export targets; ignore `tier != 1` records (such as `rolling_glossary.md`) for `update_existing_entry` proposals.
2. **Lazy Note Loading:** Use `catalog.json` strictly as a navigation index (matching `oude-mechanismen`'s lazy-read convention); open only the specific Tier 1 Markdown files needed for the active scene or Spotlight NPC.
3. **Staging Proposal Contract (`exports/<session_id>/tier1_proposal.md`):**
   * Classifies each canon-altering outcome as `update_existing_entry`, `propose_new_entry`, or `session_chronicle_note` (empty sessions explicitly state that no Tier 1 changes are proposed).
   * Every `update_existing_entry` item cites the exact `path` and `observed_source_sha` from `catalog.json` (or `observed_source_sha: n/a (no catalog.json)` in standalone vaults) so the human reviewer can spot upstream drift before editing the Tier 1 file.
   * Lists every setting glossary term encountered and verifies that `keep_untranslated` terms retain their exact canonical spelling and `preferred_translation` terms map back to their canonical form.
   * Optionally includes a publishable chapter draft (stripping all `[OOC]` exchanges, state blocks, and coaching data, adapted from `dutch-tutor`'s novelised play-report convention) ready for manual promotion into `campaigns/`.
4. **Structural Meta-Leak Scan (Warn-and-Annotate):** Before reporting staging complete, `lingua-promote` runs a shell text scan (`rg` or `grep`, where shell tools are available; marked `n/a (no shell)` otherwise) on `exports/<session_id>/tier1_proposal.md` for structural coaching markers (`ledger.md`, `CEFR`, `clean_uses`, `next_due`, `pending_coach_turns`, and `err-` item slugs).
   Any match does not delete the staged file; `lingua-promote` prepends a visible `WARNING: potential coaching meta-leak` callout listing matching lines for human inspection before promotion.

---

## 4. File Contracts & Templates

All structured configuration and play files use plain YAML or Markdown with YAML front matter so humans can edit them directly in Obsidian or any text editor.

### 4.1 Language Pack (`languages/<iso>/pack.md`)

```yaml
---
language_code: "xx"           # BCP-47 tag (e.g. nl)
display_name: "Language X"
cefr_levels: ["A2", "B1", "B2", "C1"]
error_taxonomy:
  - word_order
  - gender_article
  - verb_conjugation
  - preposition
  - lexical_choice
  - register
  - idiom
  - spelling
---
```

Body sections (Markdown): `## CEFR Profiles` (`A2`–`C1` vocabulary ceiling, sentence-length guidance, register targets, grammar focus), `## Error Taxonomy Notes`, and `## Wingman Scaffolding Cues`.

### 4.2 Campaign Config (`_gm-materials/lingua-ludi/<campaign>/campaign.yml`)

```yaml
campaign_id: "campaign-a"
target_language: "xx"
feedback_language: "yy"       # Learner study language for Wingman OOC & Coach ledger notes
canonical_language: "zz"      # Fallback ONLY when host .agents/AGENTS.md is absent or silent
cefr_level: "B1"
wingman:
  enabled: true
  name: "Companion A"
  role: "Local fixer and colleague"
  policy: "on_request"        # on_request | active
max_active_npcs: 2            # 1..2
max_retrieval_targets: 4      # 0..5 (0 disables retrieval injection)
```

* **`feedback_language` vs `canonical_language`:** `feedback_language` is the learner's preferred explanation language for Wingman `OOC:` replies and Shadow Coach notes in `ledger.md`.
  `canonical_language` is the host vault's published canon language used by `lingua-promoter` in `exports/`.
  The two are often identical (e.g. both `en-GB`), but differ when a learner studies `Target Language X` with explanations in `Language Y` inside a vault published in `Language Z`.
* **Precedence rule:** Host `.agents/AGENTS.md` is authoritative for `canonical_language`.
  `campaign.yml`'s `canonical_language` is used solely as a fallback when `.agents/AGENTS.md` is absent or specifies no canonical language (`lingua-init` warns if both are present and disagree).

### 4.3 Setting Glossary (`.agents/lingua-ludi/glossary.<iso>.yml`)

```yaml
target_language: "xx"
canonical_language: "zz"      # Descriptive metadata matching the vault's resolved canonical_language
terms:
  - canonical_term: "Term A"
    target_rendering: "Term A"
    policy: "keep_untranslated"        # keep_untranslated | preferred_translation
    related_catalog_path: "directory/PlaceA.md"
  - canonical_term: "Place A"
    target_rendering: "Plaats A"
    policy: "preferred_translation"
    related_catalog_path: "directory/PlaceA.md"
```

`glossary.<iso>.yml`'s `canonical_language` is descriptive metadata only and does not override `.agents/AGENTS.md` or `campaign.yml` (if it disagrees with the vault's resolved `canonical_language`, `lingua-promote` notes the mismatch for human review).

### 4.4 Session Identifier, `.active_session`, `log.md`, and `sessions/<session_id>.md`

* **`session_id` Format & Generation:** Generated by the GM at `lingua-play` start as `YYYY-MM-DD-NNN` (ISO date plus zero-padded per-day sequence number, e.g. `2026-11-26-001`), unique within `_gm-materials/lingua-ludi/<campaign>/sessions/`.
* **`.active_session` Contract:** Small YAML file (`session_id`, `started_at`) created by the GM at `lingua-play` start and deleted on clean session close.
  If `.active_session` already exists on start, `lingua-play` displays its contents, refuses to start automatically, and asks the user via an explicit interactive confirmation prompt whether to abort or take over from a crashed session.
* **`log.md` Contract:** Append-only Markdown containing one `## Session <session_id>: <Title>` heading per completed session, followed by a concise narrative summary (locations visited, NPCs engaged, stakes resolved, canon-relevant outcomes) with zero coaching metadata.
* **`sessions/<session_id>.md` Contract:** Markdown turn log with YAML front matter (`session_id`, `campaign_id`, `published: false`, `started_at`, `ended_at`) followed by sequential `### Turn N` sections recording player input, any 24XX roll outcome, GM narration, and labelled subagent lines (`[Wingman]`, `[NPC: <Name>]`).
  `published: false` remains `false` permanently on the gitignored transcript inside `_gm-materials/`; when a human manually promotes a staged chapter draft from `exports/` into `campaigns/`, the new Tier 1 chapter file sets `published: true`.
  All `OOC:` exchanges are prefixed with `> [OOC]` so `lingua-promoter` and `lingua-coach` crash recovery can strip them reliably.

---

## 5. Invariants, Risks & Failure Modes

### 5.1 Invariants

1. **Zero Setting/Language Hardcoding in Core:** `skills/`, `agents/`, and `templates/` contain no specific setting names, campaign lore, or target-language word lists.
2. **Zero Python Runtime Dependency in `v0.1.0`:** All `v0.1.0` workflows execute purely through Markdown skills, subagent persona definitions, shell utilities where available, and plain YAML/Markdown workspace files.
3. **Strict Tier 1 & Host-Root Read-Only Boundary:** No `lingua-ludi` skill or subagent ever writes to `campaigns/`, `directory/`, `systems/`, `.agents/context/catalog.json`, or the host `.gitignore`.
4. **Strict Write Ownership:** Only the GM writes `.active_session`, `state.md`, `log.md`, and `sessions/`; `lingua-coach` is unconditionally the sole writer of `ledger.md`; `lingua-wingman` and `lingua-npc` write nothing; only `lingua-promoter` writes `exports/`.
5. **One Active Session per Campaign Workspace:** `lingua-play` creates `.active_session` on start, removes it on clean close, and refuses to start if `.active_session` already exists unless the user explicitly confirms crashed-session takeover.
6. **Single Coach Writer Queue & Race-Free Startup Sequence:** At most one persistent `lingua-coach` runs per session; startup step 3 completes any `ledger.md` recovery or `pending_coach_turns` drain before startup step 4 reads due targets; the GM never reads `ledger.md` mid-session while Coach writes may be in flight; Tier 2 harnesses lacking completion tracking run `lingua-coach` synchronously at session end.
7. **Bounded Subagent Fan-Out & Context Ceilings:** Routine turns invoke zero synchronous subagents; Spotlight NPCs cap at `max_active_npcs` (default `2`, range `1..2`); persistent subagents and `lingua-promoter` enforce the operational context ceilings defined in §3.2 rather than accumulating unbounded history.
8. **No Coaching Meta-Leak & Untrusted-Data Framing:** `lingua-promoter` never reads `ledger.md` or coaching recaps; `lingua-wingman`, `lingua-npc`, and `lingua-promoter` treat quoted session utterances and Tier 1 notes strictly as untrusted data; staged proposals are scanned for structural coaching markers (warn-and-annotate) where shell tools exist.
9. **Charitable Fiction & Explicit OOC Separation:** Grammatical errors do not trigger in-world mechanical failure, and valid `OOC:` pauses do not mutate `state.md` or trigger `lingua-coach`.

### 5.2 Risks, Failure Modes & Mitigations

| Risk / Failure Mode | Trigger | Mitigation |
| :--- | :--- | :--- |
| Concurrent edit race or unbounded drain on `ledger.md` | Async `lingua-coach` writes while GM reads, harness lacks completion tracking, or Coach stalls at close | Single persistent `lingua-coach`; pinned startup order completes recovery/drain before GM reads `ledger.md`; zero mid-session GM reads; Tier 2 harnesses run Coach at session end; bounded close drain records unprocessed turns in `state.md` (`pending_coach_turns`). |
| Corrupted `ledger.md` after Coach crash | `lingua-coach` dies mid-table write | Startup step 3 rebuilds `ledger.md` from GM-owned `sessions/*.md` (excluding `[OOC]` blocks) using the active Language Pack; if rebuild fails, archives corrupt file as `ledger.md.corrupt-<session_id>`, initialises fresh `ledger.md`, and skips retrieval injection for that session. |
| Concurrent GM sessions racing on workspace files | Two parallel chats run `lingua-play` in the same campaign | `.active_session` lock file blocks a second `lingua-play` start until the user explicitly confirms crashed-session takeover via interactive prompt. |
| Context & token budget growth | Long sessions or multi-month campaigns grow persistent subagent or `log.md` context | Operational ceilings (§3.2): `lingua-coach` gets current turn + `ledger.md` only; `lingua-npc` refreshes after `12` turns; `lingua-wingman` compacts after `20` exchanges; `lingua-promoter` reads current session + last `3` `log.md` summaries. |
| Prompt injection or coaching meta-leak into `exports/` | Raw player utterances or Tier 1 notes contain instruction-like text, or Coach data slips into staging | Untrusted-data framing across `lingua-wingman`, `lingua-npc`, and `lingua-promoter`; `lingua-promoter` barred from `ledger.md`; shell scan flags structural coaching markers (warn-and-annotate); mandatory human review before Tier 1 promotion. |
| Slow or chatty turns | Spawning Wingman and NPCs on every routine line | Routine actions stay in the GM main session; `wingman.policy` defaults to `on_request`; Spotlight NPCs spawn only on direct engagement and cap at `max_active_npcs` (default `2`, range `1..2`). |
| NPC canon contradiction or state hijacking | Spotlight NPC invents world outcomes or breaks Tier 1 lore | NPC subagent receives only its Tier 1 note + Scene Handoff Packet, returns dialogue/reactions only, and has zero write authority over `state.md`. |
| Stale parent `catalog.json` | Host vault notes changed after play began | `lingua-promoter` records the exact `observed_source_sha` from `catalog.json` on every `update_existing_entry` item (note: `catalog.json` uses 8-char SHA-256 prefixes, a 32-bit drift signal with ~2⁻³² collision probability per change, acceptable at campaign-vault scale). |
| Tier 2 `rolling_glossary.md` misclassified as Tier 1 | `oude-mechanismen` indexes `rolling_glossary.md` with `tier: 2` | Skills filter `catalog.json` to `tier == 1` and forbid `update_existing_entry` against non-Tier-1 paths. |
| Personal coaching data pushed to public campaign repo | Host Git repo does not ignore `_gm-materials/`, or directory is non-Git | Inside Git repos, `lingua-init` runs `git check-ignore -q _gm-materials/` and hard-aborts with manual instructions if ignored check fails (never editing `.gitignore` itself); in non-Git directories, prints an explicit unprotected-directory warning. |
| Missing Language Pack at init | User requests an `<iso>` with no `languages/<iso>/pack.md` | `lingua-init` hard-aborts without writing files and lists available packs under `$LINGUA_LUDI_ROOT/languages/`. |
| Harness lacks subagent tools | User invokes `lingua-play` in a single-context client | `lingua-play` refuses cleanly and informs the user that `v0.1.0` requires a subagent-capable harness (tracked for `v0.2.0` in https://github.com/terra-campaigns/lingua-ludi/issues/3). |

---

## 6. Alternatives Considered

| Selected approach | Alternative | Reason for `v0.1.0` | Cost or limitation accepted |
| :--- | :--- | :--- | :--- |
| Zero-Python Markdown skills + subagent personas | Pure-Python (`pydantic` v2 / `uv`) state-transformer engine + 7-command CLI (superseded Spec `0.1.0`, commit `46f21cf`) | Rejected because maintaining a Python package, serialised turn schemas, and CLI commit choreography adds implementation and turn-loop overhead for a workflow that [`play-uvg`](https://github.com/terra-campaigns/charged) and [`dutch-tutor`](https://github.com/terra-campaigns/hostile) ran directly in Markdown. | Write ownership, meta-leak isolation, glossary spelling, `observed_source_sha` citation, and `ledger.md` hygiene rely on prompt contracts, shell text scans where available, and mandatory human review at promotion time rather than compiled runtime guards. |
| GM main session + persistent Wingman + selective Spotlight NPCs + async Coach | Single monolithic GM+Tutor prompt ([`dutch-tutor`](https://github.com/terra-campaigns/hostile) pattern) | Rejected because a single prompt merges GM narration, companion help, NPC voices, and grammar correction into one context window with no perspective isolation. | Requires a subagent-capable harness (Tier 1 or Tier 2) in `v0.1.0`. |
| Subagent-capable harness required in `v0.1.0` | Single-context fallback mode and portable prompt-bundle compiler for plain web/mobile chat | Deferred to `v0.2.0` (https://github.com/terra-campaigns/lingua-ludi/issues/3) because supporting no-subagent clients requires dual `ledger.md` write-ownership exceptions and single-window persona switching before the core multi-persona flow is playtested. | Plain web chat and mobile clients without subagent tools are unsupported in `v0.1.0`. |
| Selective Spotlight NPCs + `on_request` Wingman + async Coach | Full-table parallel fan-out on every turn | Rejected because fanning out every turn adds wait latency and token cost on routine beats and causes companions and NPCs to speak over mundane player actions. | Wingman and NPCs stay quiet on routine beats unless addressed or configured `active`. |
| Embedded Markdown `ledger.md` spaced-retrieval table | External SRS backend (e.g. Anki) | Rejected because external SRS tools require Python connectors or sync scripts, violating the zero-Python offline workspace boundary. | No cross-device mobile flashcard sync in `v0.1.0`; review occurs inside play sessions or by reading `ledger.md`. |
| Zero Python in `v0.1.0` (no standalone validator script yet) | Add a standalone `scripts/lint_lingua.py` validator right away | Rejected because introducing a Python toolchain before playtesting risks building checks for drift patterns that do not occur in practice. | Export and pack validation relies on checklists and shell grep scans during `v0.1.0`; a thin linter can be added in `v0.2.0` without changing file formats. |
| Installed globally (`$LINGUA_LUDI_ROOT`) and run inside host campaign repos | Playing inside the `lingua-ludi` repository and referencing host vaults externally | Rejected because global installation matches [`oude-mechanismen`](https://github.com/terra-campaigns/oude-mechanismen) and [TOME](https://github.com/efsalvarenga/tome) workflows and co-locates play workspaces under the host vault's `_gm-materials/`. | Git host vaults without `_gm-materials/` ignored trigger a `lingua-init` hard abort until the user adds the ignore rule manually. |
| Staged proposals under `_gm-materials/lingua-ludi/<campaign>/exports/` | Direct edits to Tier 1 `campaigns/` or `directory/` notes | Rejected because direct writes risk allowing playful language-learning detours or coaching artefacts to corrupt published Tier 1 campaign canon. | Requires human review and a manual edit/copy step to promote staged proposals into Tier 1 canon. |

---

## 7. Open Questions

None open.

---

## 8. Verification Strategy

Because `v0.1.0` contains no Python runtime code, verification validates structural contracts, skill procedures, and playtest scenarios before tagging `v0.1.0`:

1. **Repository Neutrality & Hygiene Check:** Verify `skills/`, `agents/`, and `templates/` contain zero hardcoded setting names, campaign lore, or target-language vocabulary lists; verify every Markdown file uses British English, blank lines after headings, and valid YAML front matter.
2. **Installation Discovery Check:** Verify skills resolve kit files via `LINGUA_LUDI_ROOT` and via the `$HOME/.local/share/lingua-ludi` fallback when `LINGUA_LUDI_ROOT` is unset.
3. **Playtest Dry-Run Matrix (using neutral fixture `Setting A` / `Target Language X`):**
   * **Init, Language Pack & `.gitignore` safety test:** Run `lingua-init` with an unknown `<iso>` (verify hard-abort listing installed packs); run inside a Git vault without `_gm-materials/` ignored (verify `git check-ignore` hard-abort with zero writes, including zero `.gitignore` edits); run inside a non-Git standalone folder (verify unprotected warning and clean scaffold); then add `_gm-materials/` to the Git vault's `.gitignore`, re-run `lingua-init`, and verify `_gm-materials/lingua-ludi/` layout and zero Tier 1 edits.
   * **Concurrency & Turn/OOC test:** Verify `.active_session` blocks a duplicate `lingua-play` start and prompts for crashed-session takeover; run a routine turn, a valid `OOC:` pause (`state.md` unchanged, `lingua-coach` skipped), an embedded parenthetical `(laughs) ...` and paired `(a) (b)` (both stay in-character), a 24XX setback roll, and a Spotlight NPC turn (`max_active_npcs <= 2`, zero file writes by NPC/Wingman).
   * **Startup Recovery, Async Coach & Drain test:** Verify `ledger.md` updates sequentially without mid-session GM reads, advances clean-use intervals (`1 → 2 → 4 → 8 → 16`), resets repeated errors to `+1`, surfaces up to `max_retrieval_targets` (default `4`) due targets at startup step 4, records `pending_coach_turns` on simulated close timeout, drains `pending_coach_turns` in startup step 3 before step 4 reads targets, and rebuilds (or archives to `ledger.md.corrupt-<session_id>`) a truncated `ledger.md` from `sessions/*.md`.
   * **Harness Tier 2 & Refusal test:** Verify a Tier 2 harness (no completion tracking) runs `lingua-coach` synchronously at session end without dropping turns, and a harness with no subagent tools refuses cleanly citing `v0.1.0` subagent requirement.
   * **Promotion & Canon-Safety test:** Run `lingua-promote` on a session containing a Tier 1 update and personal coaching errors in `ledger.md`; verify `tier == 2` `rolling_glossary.md` is ignored, `observed_source_sha` matches `catalog.json`, `keep_untranslated` terms are preserved, the structural shell meta-leak scan passes (and warns-and-annotates when a synthetic `CEFR` or `err-` marker is injected), and zero Tier 1 files are modified.
4. **First Concrete Testbed Validation (Outside Core Skills):**
   * Author `languages/nl/pack.md` (Dutch `A2`–`C1` profiles and Dutch error taxonomy: `de_het`, `v2_word_order`, `separable_verb`, `er_construction`, `preposition`, `lexical_choice`, `register`).
   * Playtest inside [`terra-campaigns/hostile`](https://github.com/terra-campaigns/hostile) with `.agents/lingua-ludi/glossary.nl.yml`, verifying British English (`en-GB`) staged Tier 1 export against `hostile`'s real `.agents/context/catalog.json` and `.agents/AGENTS.md`.

---

## 9. Goldfish Review Gates

### 9.1 Approved Specification: Spec Version 0.2.2

* **Status:** Approved (`2026-11-27`, human sign-off by `efsalvarenga`) — two zero-context Goldfish EGM review rounds completed with all blocking and accepted findings resolved or explicitly deferred; consolidated summary recorded in [001_lingua_ludi_core_architecture_review.md](../reviews/001_lingua_ludi_core_architecture_review.md).
  Ready to draft and review `docs/plans/001_lingua_ludi_core_architecture_plan.md`.
* **Sign-Off Criteria:**
  1. Zero-context Goldfish EGM reviews converged with all blocking and accepted Major findings resolved or explicitly deferred ([001_lingua_ludi_core_architecture_review.md](../reviews/001_lingua_ludi_core_architecture_review.md)) — **Satisfied**.
  2. Human developer (`efsalvarenga`) explicitly approves the specification and updates `Status: Approved` — **Satisfied (`2026-11-27`)**.
  3. Implementation plan (`docs/plans/001_lingua_ludi_core_architecture_plan.md`) approved before production skill, persona, template, and Dutch pack files are authored — **Next Gate**.

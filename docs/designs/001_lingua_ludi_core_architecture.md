# Design Specification: Lingua Ludi Core Architecture

**Document ID:** `docs/designs/001_lingua_ludi_core_architecture.md`

| Metadata | Value |
| :--- | :--- |
| Author | `efsalvarenga` |
| Date | 2026-11-25 |
| Spec Version | 0.2.0 |
| Status | Draft (Awaiting External Goldfish EGM Review & Human Sign-Off) |
| Target Milestone | v0.1.0 (Zero-Python Multi-Persona Skill Kit, Language Packs & Campaign-Repo Staging) |

## Changelog

| Spec Version | Summary |
| :--- | :--- |
| 0.2.0 | Simplified architecture from a Python state-machine engine to a zero-Python, Markdown-and-skills harness installed alongside `oude-mechanismen` and executed inside host campaign repositories. Adopted player-owned 24XX mechanics (`play-uvg` pattern), multi-persona subagent orchestration (GM main session, persistent Wingman, selective Spotlight NPCs, async background Shadow Coach, session-end Promoter), strict file write ownership under `_gm-materials/lingua-ludi/<campaign>/`, and verified compatibility with `oude-mechanismen`'s `.agents/context/catalog.json` (including `nav_order`, nullable `summary`, and Tier 2 `rolling_glossary.md` filtering). |
| 0.1.0 | Initial Elephant draft proposing a pure-Python (`pydantic` v2 / `uv`) state-transformer engine, deterministic seeded RNG, and 7-command CLI run from the `lingua-ludi` repository. Superseded before Goldfish review due to unnecessary turn-loop complexity and single-voice play limitations. |

### Deferred Future Scope

The following capabilities are intentionally excluded from v0.1.0:

* Python runtime engine, CLI, or automated validator scripts (deferred until real play demonstrates a mechanical validation failure that warrants introducing a Python toolchain).
* Macro faction turn execution (such as integrating [TOME](https://github.com/efsalvarenga/tome) as a background geopolitical engine); v0.1.0 reserves a plain Markdown `## World Events` section in `state.md` so manual or future macro outputs can be pasted in without changing templates.
* Audio input/output (local Whisper STT or TTS pipelines).
* Web UI, hosted deployment, authentication, and multi-player networking.
* Direct writes, Git commits, or pull requests applied automatically to Tier 1 campaign canon (`campaigns/`, `directory/`, `systems/`).

---

## 1. Overview & Problem Statement

### The Problem

Practising a European language through solo tabletop roleplaying works best when gameplay stays fast, light, and grounded in a living campaign vault.
Previous single-skill experiments (`play-uvg` in `charged` and `dutch-tutor` in `hostile`) proved that plain Markdown state files (`state.md`, `log.md`), player-rolled dice, and short skill instructions keep turn friction low, while exposing three recurring limitations:

1. **Flat single-voice play:** A single monolithic GM-plus-tutor prompt speaks in one voice, so the player never experiences distinct table companions or independent NPC perspectives.
2. **Tight coupling to one setting and one language:** Each experiment hardcoded a single campaign universe and a single target language inside the skill file.
3. **Manual canon and pedagogy hygiene:** Inline grammar corrections clutter transcripts and risk leaking out-of-character coaching metadata into published campaign logs, while cross-language lore promotion back to a parent campaign vault lacks a structured glossary and drift-check staging workflow.

### Overview

Lingua Ludi solves this as a **zero-Python, setting-agnostic, language-agnostic skill and subagent persona kit** installed once (mirroring `oude-mechanismen`) and run directly from inside any host campaign repository.

```text
  [lingua-ludi kit (installed globally, read-only during play)]
     ├── skills/      (lingua-init, lingua-play, lingua-promote)
     ├── agents/      (lingua-wingman, lingua-npc, lingua-coach, lingua-promoter)
     ├── languages/   (reusable CEFR + error-taxonomy Language Packs, e.g. nl)
     └── templates/   (campaign.yml, state.md, log.md, ledger.md, glossary, export)
                            │
                            ▼  invoked inside a host campaign repo
  [host campaign repo (e.g. an oude-mechanismen vault)]
     ├── .agents/AGENTS.md                        (read-only setting tone & canon language)
     ├── .agents/context/catalog.json             (read-only Tier 1 index + SHAs)
     ├── .agents/lingua-ludi/glossary.<iso>.yml   (tracked per-language proper-noun rules)
     ├── campaigns/, directory/, systems/         (read-only Tier 1 canon; human-promoted only)
     └── _gm-materials/lingua-ludi/<campaign>/    (gitignored active play workspace)
            ├── campaign.yml                      (language, CEFR, companion & NPC settings)
            ├── state.md                          (24XX character sheet, scene, World Events)
            ├── log.md                            (append-only narrative session summaries)
            ├── ledger.md                         (spaced-retrieval error & vocab table)
            ├── sessions/<session_id>.md          (unpublished turn log)
            └── exports/<session_id>/             (staged canonical-language Tier 1 proposals)
```

### Goals

* **Lightweight Zero-Python Gameplay:** Keep play strictly in chat and human-readable Markdown files (`state.md`, `log.md`, `ledger.md`), with player-owned 24XX dice rolls and zero runtime build or CLI overhead.
* **Distinct Table Personas via Subagents:** Run the GM as the main session while delegating to a persistent **Wingman** subagent (friendly tutor buddy), selective **Spotlight NPC** subagents (independent voice and perspective), an asynchronous background **Shadow Coach** subagent, and a fresh session-end **Promoter** subagent.
* **Strict Three-Layer Separation:** Keep generic skills/personas (`lingua-ludi`), reusable Language Packs (`languages/<iso>/`), and host campaign setting canon/glossaries decoupled so any language pack pairs cleanly with any campaign vault.
* **Campaign-Repo Execution Without Vault Pollution:** Store all mutable session files, personal learner ledgers, and staged exports under `_gm-materials/lingua-ludi/<campaign>/`, which `oude-mechanismen` vaults already exclude from Git, Jekyll builds, `catalog.json`, and `lint_wiki.py`.
* **Closed-Loop Spaced Retrieval in Plain Markdown:** Track errors and vocabulary in `ledger.md` via the async Shadow Coach and surface `3–5` due targets into the next scene or session without breaking fiction.
* **Safe Cross-Language Tier 1 Promotion Staging:** Distil canon-altering outcomes into the host vault's canonical language, check glossary rules and Tier 1 `catalog.json` paths/SHAs, and stage spoiler-safe Markdown proposals for human review without mutating published Tier 1 notes.

### Non-Goals

* Python engine library, Pydantic schemas, seeded RNG code, or gameplay CLI in v0.1.0.
* Playing campaigns inside the `lingua-ludi` repository itself.
* Hardcoding any specific European language or any specific campaign setting inside core skills, persona definitions, or templates.
* Automated direct edits, commits, or pull requests against Tier 1 vault directories (`campaigns/`, `directory/`, `systems/`).
* Mechanical penalties for grammatical mistakes during play (the GM interprets learner intent charitably; the Shadow Coach logs errors silently).

---

## 2. Critical User Journeys (CUJs)

### CUJ 1: Initialise a Language-Learning Campaign Inside a Host Vault

* **Given:** A host campaign repository (with or without `oude-mechanismen` `.agents/context/catalog.json` and `.agents/AGENTS.md`) and an installed `lingua-ludi` kit containing a target Language Pack (e.g. `Target Language X`).
* **When:** The user invokes `lingua-init` inside the host campaign repository, supplying a campaign slug (`campaign-a`), target language (`Target Language X`), feedback language (`Canonical Language Y`), CEFR level (`B1`), and Wingman policy (`on_request` or `active`).
* **Then:** The skill creates `_gm-materials/lingua-ludi/campaign-a/` populated from templates (`campaign.yml`, `state.md`, `log.md`, `ledger.md`, `sessions/`, `exports/`), creates `.agents/lingua-ludi/glossary.<iso>.yml` if absent, verifies `.gitignore` ignores `_gm-materials/`, and leaves all Tier 1 vault files untouched.

### CUJ 2: Routine Play Turn with Background Shadow Coaching

* **Given:** An active `lingua-play` session in `campaign-a` with `state.md`, `ledger.md` (containing `3–5` due targets loaded at session start), and a persistent `lingua-coach` subagent running.
* **When:** The player submits a routine action in `Target Language X` (containing a subtle grammatical error) that does not address a Spotlight NPC or invoke the Wingman.
* **Then:** The GM interprets the player's intent charitably, narrates the scene outcome directly in `Target Language X` within the active CEFR profile (weaving in a due retrieval target naturally), updates `state.md` if needed, appends the turn to `sessions/<session_id>.md`, and dispatches the player's raw utterance asynchronously to `lingua-coach` so the player experiences zero waiting latency while `lingua-coach` updates `ledger.md` in the background.

### CUJ 3: Spotlight NPC Interaction, Wingman OOC Help, and Player-Rolled 24XX Risk

* **Given:** An active scene in `Place A` featuring named Tier 1 NPC `Actor A`.
* **When:** The player asks `(How do I say "pressure seal"?)` (`OOC:` pause), then addresses `Actor A` in `Target Language X` to attempt a risky negotiation, rolls their 24XX skill die (`d8` -> `3`, setback), and reports the roll.
* **Then:** On the `OOC:` line, fiction pauses cleanly (`state.md` unchanged, `lingua-coach` skipped), the persistent `lingua-wingman` answers concisely in `Canonical Language Y` and offers a natural phrasing cue, and the prior prompt remains active; on the negotiation turn, the GM states the risk, takes the player's `setback` result, messages the scene's `lingua-npc` subagent (briefed from `Actor A`'s Tier 1 note and private agenda) for `Actor A`'s in-character dialogue, integrates the complication into `state.md`, and dispatches the player's utterance to `lingua-coach`.

### CUJ 4: Session End and Cross-Language Tier 1 Lore Staging

* **Given:** A completed play session in `campaign-a` where a canon-altering event occurred involving `Actor A` and `Place A`, and `ledger.md` contains personal learner error records.
* **When:** The user ends the session and invokes `lingua-promote`.
* **Then:** The GM waits for any pending `lingua-coach` update to finish, appends a narrative summary to `log.md`, and launches a fresh `lingua-promoter` subagent with ONLY narrative inputs (`log.md` and in-character lines from `sessions/<session_id>.md`, excluding `OOC:` pauses and never reading `ledger.md`), `.agents/AGENTS.md`, `.agents/lingua-ludi/glossary.<iso>.yml`, and Tier 1 entries (`tier == 1`) from `.agents/context/catalog.json`; `lingua-promoter` writes `_gm-materials/lingua-ludi/campaign-a/exports/<session_id>/tier1_proposal.md` in `Canonical Language Y` citing exact `path` and `sha` values, preserving `keep_untranslated` glossary terms, and optionally weaving usable non-native player phrasing strictly into attributed in-voice character quotes (never narrator prose or factual `directory/` updates), without modifying any file in `campaigns/`, `directory/`, or `systems/`.

### CUJ 5: Standalone Folder or Non-Subagent Harness Fallback

* **Given:** A user running in a campaign directory without `oude-mechanismen` (`catalog.json` absent), or in a chat harness that lacks subagent tools.
* **When:** The user starts a session using `lingua-play` and later runs `lingua-promote`.
* **Then:** Without `catalog.json`, the GM reads local campaign Markdown notes directly and `lingua-promote` emits proposals with `observed_source_sha: n/a (no catalog.json)`; without subagent tools, the skill explicitly notifies the user that subagent isolation is unavailable, falls back to clearly labelled single-context personas (`[GM]`, `[Wingman]`, `[Actor A]`), and defers coaching to a separate session-end pass rather than mixing live grammar corrections into dialogue.

---

## 3. Technical Architecture

### 3.1 Three Orthogonal Layers & Repository Topology

Lingua Ludi separates generic harness procedures, reusable language packs, and host campaign setting data:

1. **Lingua Ludi Kit (`terra-campaigns/lingua-ludi`, installed read-only at e.g. `~/.local/share/lingua-ludi/` or discovered via `LINGUA_LUDI_ROOT`):**
   * `skills/lingua-init/SKILL.md` — bootstraps a campaign workspace inside the current host campaign repository.
   * `skills/lingua-play/SKILL.md` — GM session loop, subagent orchestration protocol, 24XX risk handling, `OOC:` pause handling, and session closing.
   * `skills/lingua-promote/SKILL.md` — session-end cross-language canonicalisation and Tier 1 staging workflow.
   * `agents/lingua-wingman.md` — persistent companion/tutor buddy persona contract.
   * `agents/lingua-npc.md` — on-demand Spotlight NPC persona contract.
   * `agents/lingua-coach.md` — asynchronous background Shadow Coach persona contract.
   * `agents/lingua-promoter.md` — fresh session-end Promoter persona contract.
   * `languages/<iso>/pack.md` — reusable Language Packs (YAML front matter + CEFR profiles `A2`–`C1`, register guidance, error taxonomy, scaffolding cues).
   * `templates/` — starter templates for `campaign.yml`, `state.md`, `log.md`, `ledger.md`, `glossary.yml`, and `tier1_proposal.md`.
2. **Language Pack Layer (`languages/<iso>/`):**
   * Declares BCP-47 `language_code` (e.g. `Target Language X`), CEFR ceiling profiles (`A2`, `B1`, `B2`, `C1`), register guidance, closed Coach error-category tags (e.g. `word_order`, `gender_article`, `verb_conjugation`, `preposition`, `lexical_choice`, `register`, `idiom`, `spelling`), and optional Wingman scaffolding phrases.
   * Contains zero campaign lore or setting proper nouns.
3. **Host Campaign Repository (Setting Cartridge + Local Play Workspace):**
   * **Read-only canon sources:** `.agents/AGENTS.md` (setting tone, voice, canonical language), `.agents/context/catalog.json` (when `oude-mechanismen` is present), and lazy-loaded Tier 1 notes (`campaigns/`, `directory/`, `systems/`).
   * **Tracked setting-language mapping:** `.agents/lingua-ludi/glossary.<iso>.yml` — per-language proper nouns and jargon (`keep_untranslated` vs `preferred_translation`).
     Because `.agents/` is excluded by `oude-mechanismen`'s `data_extractor.rb`, this file stays tracked in Git without entering Jekyll builds or `catalog.json`.
   * **Gitignored local play workspace:** `_gm-materials/lingua-ludi/<campaign>/` — already gitignored in `oude-mechanismen` vaults and skipped by Jekyll, `data_extractor.rb`, and `lint_wiki.py`.

### 3.2 Multi-Persona Subagent Topology & Turn Cadence

```text
  Player Turn (Target Language X)
       │
       ▼
  ┌─────────────────────────────────────────────────────────────────────┐
  │  GM (Main Session — skills/lingua-play/SKILL.md)                    │
  │  • Owns scene narration, environment, minor extras, stakes, 24XX    │
  │  • Sole writer of: state.md, log.md, sessions/<session_id>.md       │
  └───────┬──────────────────────────┬──────────────────────────┬───────┘
          │                          │                          │
          │ on OOC / request /       │ only when PC engages     │ async fire-and-forget
          │ active stall beat        │ named Spotlight NPC      │ after target-language turn
          ▼                          ▼ (max 1–2 per scene)      ▼
  ┌──────────────────┐      ┌──────────────────┐       ┌──────────────────────┐
  │ lingua-wingman   │      │ lingua-npc       │       │ lingua-coach         │
  │ (persistent)     │      │ (scene-scoped)   │       │ (persistent, async)  │
  │ • dialogue only  │      │ • dialogue only  │       │ • analyses player    │
  │ • zero writes    │      │ • private agenda │       │   utterance only     │
  │ • CEFR-bounded   │      │ • zero writes    │       │ • sole writer of     │
  └──────────────────┘      └──────────────────┘       │   ledger.md          │
                                                       └──────────────────────┘

  Session End (skills/lingua-promote/SKILL.md)
       │
       ▼  narrative inputs ONLY (never ledger.md or coaching notes)
  ┌─────────────────────────────────────────────────────────────────────┐
  │  lingua-promoter (fresh subagent)                                   │
  │  • Reads: log.md, in-character session lines (no OOC/ledger.md),    │
  │           catalog.json (tier == 1), .agents/AGENTS.md,              │
  │           .agents/lingua-ludi/glossary.<iso>.yml                    │
  │  • Sole writer of: exports/<session_id>/tier1_proposal.md           │
  └─────────────────────────────────────────────────────────────────────┘
```

#### Role Contracts & Latency Controls

* **GM (Main Session):** Handles routine narration, minor background figures, stakes, and 24XX risk statements directly in a single turn with zero subagent wait time.
  Always speaks `target_language` within the active CEFR profile during play, interpreting learner phrasing charitably so language errors never stall fiction.
  Weaves the `3–5` due retrieval targets loaded at session start (or scene transitions) subtly into narration and NPC briefs without announcing them.
* **Wingman (`lingua-wingman`):** Spawned once at session start and messaged persistently across the session.
  * When `wingman.enabled` is `false`, never spawned.
  * When `wingman.policy` is `on_request` (default), messaged only when the player uses an `OOC:` / `(question)` pause, directly addresses the Wingman, or explicitly asks for phrasing help.
  * When `wingman.policy` is `active`, may also be invited on moments of stall, ambiguity, or tense decisions (bounded to `1–3` short sentences; never overrides player decisions).
  * Speaks in `target_language` for in-world banter/scaffolding and switches to `feedback_language` for direct `OOC:` explanations.
* **Spotlight NPCs (`lingua-npc`):** Spawned only when the player directly interacts with a named NPC whose voice, secrecy, or agency matters to the scene.
  * Hard cap: at most `max_active_npcs` (default `2`) concurrent NPC subagents per scene.
  * Kept alive across turns within the scene via persistent messaging so the NPC remembers the conversation naturally, then retired when the scene changes.
  * Briefed with a short **Scene Handoff Packet**: setting tone summary, CEFR ceiling, relevant glossary entries, the NPC's Tier 1 note (loaded lazily via `catalog.json`), private agenda/knowledge boundaries, and the latest player beat/roll outcome.
  * Returns in-character dialogue and visible reaction only; never declares mechanical outcomes or modifies files.
* **Shadow Coach (`lingua-coach`):** Spawned once at session start as a single persistent subagent and invoked **asynchronously (fire-and-forget)** after each target-language player turn (never invoked on pure `OOC:` pause turns).
  * Because a single persistent `lingua-coach` processes queued messages sequentially, there are never two concurrent writers to `ledger.md`.
  * Analyses only the player's utterance against the active Language Pack's error taxonomy and CEFR level, updating `ledger.md` silently without emitting chat output during play.
  * Before closing a session, the GM waits for any in-flight `lingua-coach` message to complete and prints an optional end-of-session coaching recap from `ledger.md` if the player requests it.
* **Promoter (`lingua-promoter`):** Spawned fresh at session end by `lingua-promote` with zero prior coaching context.
  * Receives `log.md`, raw in-character lines from `sessions/<session_id>.md` (excluding `OOC:` pauses; never reading `ledger.md` or coaching recaps), `.agents/AGENTS.md`, `.agents/lingua-ludi/glossary.<iso>.yml`, and `tier == 1` entries from `.agents/context/catalog.json`.
  * Translates canon-altering events into the host vault's canonical language, preserves glossary rules verbatim, and stages `exports/<session_id>/tier1_proposal.md` for human review.
  * Where raw in-character player utterances contain usable non-native phrasing, malapropisms, or awkward calques that fit the setting (e.g. outsiders, immigrants, or rough speakers who do not speak the language fluently), may selectively preserve them **strictly inside attributed in-voice character quotes** in narrative chronicle or chapter drafts—never in the chapter narrator's prose, factual `directory/` canon entries, or `keep_untranslated` proper nouns.

### 3.3 Strict File Write Ownership

To prevent race conditions with the async Shadow Coach and prevent accidental Tier 1 canon corruption, write ownership is strictly partitioned:

| Path (inside host campaign repo) | Owner | Read Access |
| :--- | :--- | :--- |
| `campaigns/`, `directory/`, `systems/`, `.agents/AGENTS.md`, `.agents/context/catalog.json` | **Human only** (read-only to all `lingua-ludi` agents) | GM, `lingua-promoter`, `lingua-npc` (assigned note only) |
| `.agents/lingua-ludi/glossary.<iso>.yml` | `lingua-init` (create if absent), then **Human** | GM, `lingua-wingman`, `lingua-npc`, `lingua-promoter` |
| `_gm-materials/lingua-ludi/<campaign>/campaign.yml` | `lingua-init` / Human | GM, `lingua-coach`, `lingua-promoter` |
| `_gm-materials/lingua-ludi/<campaign>/state.md` | **GM only** (and Human manual edits) | GM |
| `_gm-materials/lingua-ludi/<campaign>/log.md` | **GM only** | GM, `lingua-promoter` |
| `_gm-materials/lingua-ludi/<campaign>/sessions/<session_id>.md` | **GM only** | GM, `lingua-promoter` (in-character lines only; `OOC:` stripped) |
| `_gm-materials/lingua-ludi/<campaign>/ledger.md` | **`lingua-coach` only** | `lingua-coach`, GM (read at session start / scene boundary only) |
| `_gm-materials/lingua-ludi/<campaign>/exports/<session_id>/` | **`lingua-promoter` only** | Human, `lingua-promoter` |

### 3.4 Player-Owned 24XX Mechanics & OOC Pause Rule

Lingua Ludi adopts 24XX micro-scene resolution (CC BY 4.0 Jason Tocci) following `play-uvg`'s lightweight division of labour:

* **Character Sheet in `state.md`:** Plain Markdown listing `Name`, `Concept`, `Skills` (`d6` untrained/baseline, `d8`/`d10`/`d12` skilled, `d4` hindered), `Inventory`, `Hindrances`, `Current Location`, `Open Stakes`, and `World Events`.
* **Player-Owned Rolls:** Routine actions succeed without a roll.
  When an action carries risk, the GM states the risk clearly in `target_language` (what goes wrong on failure) and suggests the skill die and mode (`normal`, `hindered` = `d4`, `advantage` = roll 2 keep highest, `disadvantage` = roll 2 keep lowest).
  The player rolls physical or external dice and reports the kept result (or explicitly asks the GM to roll):
  * `1–2` (**Disaster**): action fails and the stated risk occurs.
  * `3–4` (**Setback**): action succeeds with a complication, cost, or partial result.
  * `5+` (**Success**): action succeeds cleanly.
* **OOC Pause Rule (from `dutch-tutor`):** Any player message wrapped in parentheses `(...)`, brackets `[...]`, or prefixed with `OOC:` pauses the fiction:
  1. Do not advance the scene, resolve actions, mutate `state.md`, or trigger `lingua-coach`.
  2. Route the language/vocabulary question to `lingua-wingman` (or answer directly as GM if Wingman is disabled) concisely in `feedback_language`.
  3. Reprint the current brief prompt so the player resumes without friction.

### 3.5 Spaced Retrieval in Plain Markdown (`ledger.md`)

`ledger.md` stores a simple Markdown table maintained exclusively by `lingua-coach`:

| Column | Meaning |
| :--- | :--- |
| `item_id` | Stable slug (e.g. `vocab-term-a` or `err-word-order-01`) |
| `surface_form` | Target-language word, phrase, or pattern |
| `category` | Error tag from the active Language Pack taxonomy (or `vocab`) |
| `errors` | Integer count of observed errors |
| `clean_uses` | Integer count of confirmed clean learner uses |
| `last_turn` | Session/turn marker (e.g. `S03-T04`) |
| `interval` | Current turn interval (`1`, `2`, `4`, `8`, or `16`) |
| `next_due` | Turn number at which the item becomes due again |
| `note` | Concise `feedback_language` hint or corrected form |

* **Interval rule:** New error sets `interval = 1`, `next_due = current_turn + 1`.
  Each subsequent clean learner use advances `interval` along `1 -> 2 -> 4 -> 8 -> 16`; a repeated error resets `interval` to `1` and increments `errors`.
* **Selection rule:** At session start (and optionally at scene transitions, never mid-turn while `lingua-coach` may be writing), the GM reads `ledger.md` and picks `3–5` rows where `next_due <= current_turn`, prioritising most overdue first, then highest `errors`.
  Zero-error turns are valid and still advance `clean_uses` for tracked items used correctly.

### 3.6 `oude-mechanismen` `catalog.json` Compatibility & Tier 1 Export Staging

Inspecting `oude-mechanismen`'s `jekyll/_plugins/data_extractor.rb` confirms the exact `.agents/context/catalog.json` shape:

* Top-level keys: `generated`, `engine_version`, `tier`, `total_entries`, `entries`.
* Each entry in `entries` contains: `path`, `title`, `type`, `parent`, `role`, `status`, `region`, `nav_order` (`str | null`), `summary` (`str | null`), `links_out`, `links_in`, `sha` (8-char SHA-256 prefix), and `tier` (`1` for Tier 1 canon notes; `2` for `.agents/context/rolling_glossary.md`).

Accordingly, `lingua-play` and `lingua-promote` enforce three concrete rules when reading `catalog.json`:

1. **Tier 1 Filtering:** Filter `entries` to `tier == 1` for canon grounding and Tier 1 export targets; ignore `tier != 1` records (such as `rolling_glossary.md`) for `update_existing_entry` proposals.
2. **Lazy Note Loading:** Use `catalog.json` only as a navigation index (matching `oude-mechanismen`'s Context & Token Optimisation Policy); open only the specific Tier 1 Markdown files needed for the active scene or Spotlight NPC.
3. **Staging Proposal Contract (`exports/<session_id>/tier1_proposal.md`):**
   * Classifies each canon-altering outcome as `update_existing_entry`, `propose_new_entry`, or `session_chronicle_note` (empty sessions explicitly state that no Tier 1 changes are proposed).
   * Every `update_existing_entry` item cites the exact `path` and observed `sha` from `catalog.json` so the human reviewer immediately spots upstream drift before editing the Tier 1 file.
   * Lists every setting glossary term encountered and verifies that `keep_untranslated` terms retain their exact canonical spelling and `preferred_translation` terms map back to their canonical form.
   * Optionally includes a clean, meta-free, publishable chapter draft (stripping all `OOC:` exchanges, state blocks, and coaching data, following `dutch-tutor`'s novelised play-report convention) ready for manual promotion into `campaigns/`.
   * **Diegetic In-Voice Quotes:** In `session_chronicle_note` items and chapter drafts, `lingua-promoter` may selectively reuse usable miswritten or non-native player phrasing strictly inside clearly attributed character dialogue quotes so it reads unmistakably as a character's imperfect speech (an outsider, immigrant, or rough speaker) rather than an error by the chapter narrator; narrator prose, factual `update_existing_entry`/`propose_new_entry` entries, and `keep_untranslated` glossary proper nouns remain strictly canonical.

---

## 4. File Contracts & Templates

All structured configuration files use plain YAML or Markdown with YAML front matter so humans can edit them directly in Obsidian or any text editor.

### 4.1 Language Pack (`languages/<iso>/pack.md`)

```yaml
---
language_code: "xx"           # BCP-47 tag (e.g. Target Language X)
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

Body sections (Markdown): `## CEFR Profiles` (`A2`–`C1` vocabulary ceiling, sentence length guidance, register targets, grammar focus), `## Error Taxonomy Notes`, and `## Wingman Scaffolding Cues`.

### 4.2 Campaign Config (`_gm-materials/lingua-ludi/<campaign>/campaign.yml`)

```yaml
campaign_id: "campaign-a"
target_language: "xx"
feedback_language: "yy"
canonical_language: "yy"      # Overrides or mirrors host .agents/AGENTS.md
cefr_level: "B1"
wingman:
  enabled: true
  name: "Companion A"
  role: "Local fixer and colleague"
  policy: "on_request"        # on_request | active
max_active_npcs: 2
max_retrieval_targets: 4      # 0..5
```

### 4.3 Setting Glossary (`.agents/lingua-ludi/glossary.<iso>.yml`)

```yaml
target_language: "xx"
canonical_language: "yy"
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

---

## 5. Invariants & Failure Modes

### Invariants

1. **Zero Setting/Language Hardcoding in Core:** `skills/`, `agents/`, and `templates/` contain no specific setting names, campaign lore, or target-language word lists.
2. **Zero Python Runtime Dependency in v0.1:** All v0.1 workflows execute purely through Markdown skills, subagent persona definitions, and plain YAML/Markdown workspace files.
3. **Strict Tier 1 Read-Only Boundary:** No `lingua-ludi` skill or subagent ever writes to `campaigns/`, `directory/`, `systems/`, or `.agents/context/catalog.json`.
4. **Strict Write Ownership:** Only the GM writes `state.md`, `log.md`, and `sessions/`; only `lingua-coach` writes `ledger.md`; `lingua-wingman` and `lingua-npc` write nothing; only `lingua-promoter` writes `exports/`.
5. **Single Coach Writer Queue:** At most one persistent `lingua-coach` subagent runs per session so background updates to `ledger.md` execute sequentially without write races.
6. **Bounded Subagent Fan-Out:** Routine turns invoke zero synchronous subagents (GM replies directly, async Coach runs in background); Spotlight NPCs are capped at `max_active_npcs` (default `2`) per scene.
7. **No Coaching Meta-Leak & Attributed Diegetic Voice:** `lingua-promoter` never reads `ledger.md` or coaching recaps; staged proposals contain zero CEFR ratings, error counts, or learner corrections, and any preserved non-native phrasing appears strictly inside attributed in-voice character quotes in narrative drafts rather than narrator prose or factual `directory/` entries.
8. **Charitable Fiction & Explicit OOC Separation:** Grammatical errors never trigger in-world mechanical failure, and `OOC:` questions pause the scene without mutating `state.md`.

### Failure Modes & Mitigations

| Failure Mode | Trigger | Mitigation |
| :--- | :--- | :--- |
| Concurrent edit race on `ledger.md` | GM reads or writes `ledger.md` while async `lingua-coach` is updating it | Single persistent `lingua-coach` is the sole writer; GM reads `ledger.md` only at session start / scene boundaries and waits for in-flight Coach completion before session close. |
| Slow or chatty turns | Spawning Wingman and NPCs on every routine line | Routine actions stay in the GM main session; `wingman.policy` defaults to `on_request`; Spotlight NPCs spawn only on direct engagement and cap at `2` per scene. |
| NPC canon contradiction or state hijacking | Spotlight NPC invents world outcomes or breaks Tier 1 lore | NPC subagent receives only its Tier 1 note + scene handoff packet, returns dialogue/reactions only, and has zero write authority over `state.md`. |
| Stale parent `catalog.json` | Host vault notes changed after play began | `lingua-promoter` records the exact observed `sha` from `catalog.json` on every `update_existing_entry` item so the human catches drift before applying edits. |
| Tier 2 `rolling_glossary.md` misclassified as Tier 1 | `oude-mechanismen` indexes `rolling_glossary.md` with `tier: 2` | Skills explicitly filter `catalog.json` to `tier == 1` and forbid `update_existing_entry` against non-Tier-1 paths. |
| Personal coaching data pushed to public campaign repo | Storing `ledger.md` or raw transcripts under `campaigns/` | All play workspaces default to `_gm-materials/lingua-ludi/<campaign>/`, which `lingua-init` checks against `.gitignore`. |
| Learner mistakes corrupting narrator prose or factual canon | `lingua-promoter` copying unattributed non-native phrasing into narrator text or `directory/` updates | Preserved non-native phrasing is restricted strictly to attributed in-voice character dialogue quotes in narrative drafts, leaving narrator prose, factual canon updates, and glossary proper nouns canonical, with human review before promotion. |
| Harness lacks subagent tools | Running in plain web chat or a single-context client | Skill notifies user explicitly, falls back to labelled single-context personas (`[GM]`, `[Wingman]`, `[NPC]`), and defers coaching to a separate session-end pass. |

---

## 6. Alternatives Considered

| Selected approach | Alternative | Reason for v0.1.0 | Cost or limitation accepted |
| :--- | :--- | :--- | :--- |
| Zero-Python Markdown skills + subagent personas | Pure-Python (`pydantic` v2 / `uv`) state-transformer engine + 7-command CLI (Spec `0.1.0`) | Rejected by author: adds heavy serialisation and orchestration plumbing around conversational play that `play-uvg` and `dutch-tutor` handled cleanly in Markdown. | Invariants (glossary spelling, `sha` presence, `ledger.md` table hygiene) rely on prompt contracts and human review at promotion time rather than deterministic code. |
| GM main session + persistent Wingman + selective Spotlight NPCs + async Coach | Single monolithic GM+Tutor prompt (`dutch-tutor` pattern) | Rejected by author: single-voice play lacked the feeling of distinct table personas and mixed tutor feedback with narrative voice. | Requires a subagent-capable harness for full persona isolation; single-chat clients fall back to labelled persona switching. |
| Selective Spotlight NPCs + `on_request` Wingman + async Coach | Full-table parallel fan-out on every turn | Rejected by author: adds wait latency and token cost on routine turns and causes companions/NPCs to talk over mundane player actions. | Wingman and NPCs stay quiet on routine beats unless addressed or configured `active`. |
| Zero Python in v0.1.0 (no standalone validator script yet) | Add a standalone `scripts/lint_lingua.py` validator right away | Rejected by author: omit Python unless real play demonstrates a clear need; avoids maintaining a Python toolchain before the skill kit is playtested. | Export and pack validation is checklist-driven during v0.1.0; a thin linter can be added later without changing file formats. |
| Installed globally and run inside host campaign repos (mirroring `oude-mechanismen`) | Playing inside the `lingua-ludi` repository and referencing parent vaults externally | Rejected by author: matches established `oude-mechanismen` and `tome` workflows and keeps campaign state co-located with the campaign vault's `_gm-materials/`. | Host vaults without `_gm-materials/` in `.gitignore` require `lingua-init` to verify ignore rules before creating local play files. |
| Staged proposals under `_gm-materials/lingua-ludi/<campaign>/exports/` | Direct edits to Tier 1 `campaigns/` or `directory/` notes | Rejected by author: prevents playful language-learning detours or coaching artefacts from corrupting published Tier 1 campaign canon. | Requires a brief human review and manual copy/edit step to promote staged proposals into Tier 1 canon. |

---

## 7. Open Questions

None open for v0.1.0 drafting.
The following items are consciously held for post-v0.1.0 evaluation after playtesting:

* Whether real play surfaces enough `ledger.md` or `exports/` formatting drift to justify adding a standalone Python linter script in v0.2.0.
* Whether a small prompt-bundle compiler (like VBT's `gem-instructions.md`) is needed for mobile/web chat play outside coding-agent harnesses.

---

## 8. Verification Strategy

Because v0.1.0 contains no Python runtime code, verification validates structural contracts, skill procedures, and real-world playtest scenarios before tagging v0.1.0:

1. **Repository Neutrality & Hygiene Check:** Verify `skills/`, `agents/`, and `templates/` contain zero hardcoded setting names, campaign lore, or target-language vocabulary lists; verify every Markdown file uses British English, blank lines after headings, and valid YAML front matter.
2. **Playtest Dry-Run Matrix (using neutral fixture `Setting A` / `Target Language X`):**
   * **Init test:** Run `lingua-init` in a sample vault and verify `_gm-materials/lingua-ludi/` layout, `.gitignore` check, and zero Tier 1 edits.
   * **Turn & OOC test:** Run routine turn, `OOC:` pause (verifying `state.md` unchanged and `lingua-coach` skipped), 24XX setback roll, and Spotlight NPC turn (verifying NPC cap `<= 2` and zero file writes by NPC/Wingman).
   * **Async Coach & Ledger test:** Verify `ledger.md` updates sequentially without race conditions, advances clean-use intervals (`1 -> 2 -> 4 -> 8 -> 16`), resets repeated errors to `+1`, and surfaces `3–5` due targets at the next session start.
   * **Promotion & Canon-Safety test:** Run `lingua-promote` on a session containing a Tier 1 update, personal coaching errors in `ledger.md`, and a usable non-native in-character player utterance; verify `tier == 2` `rolling_glossary.md` is ignored, observed `sha` matches `catalog.json`, `keep_untranslated` terms are preserved, any retained non-native phrasing appears strictly inside attributed in-voice character quotes (leaving narrator prose and factual `directory/` updates canonical), zero coaching metadata appears in `tier1_proposal.md`, and zero Tier 1 files are modified.
3. **First Concrete Testbed Validation (Outside Core Skills):**
   * Author `languages/nl/pack.md` (Dutch `A2`–`C1` profiles and Dutch error taxonomy: `de_het`, `v2_word_order`, `separable_verb`, `er_construction`, `preposition`, `lexical_choice`, `register`).
   * Playtest inside `terra-campaigns/hostile` with `.agents/lingua-ludi/glossary.nl.yml`, verifying British English (`en-GB`) staged Tier 1 export against `hostile`'s real `.agents/context/catalog.json` and `.agents/AGENTS.md`.

---

## 9. Goldfish Review Gates

### Current Draft: Spec Version 0.2.0

* **Status:** Draft — ready for external zero-context Goldfish EGM review (recorded in `docs/reviews/001_lingua_ludi_core_architecture_review.md`) and explicit human sign-off before drafting `docs/plans/001_lingua_ludi_core_architecture_plan.md` or authoring production skills and packs.
* **Sign-Off Criteria:**
  1. External EGM Goldfish review converged with all blocking findings resolved.
  2. Human developer (`efsalvarenga`) explicitly approves the specification and updates `Status: Approved`.
  3. Implementation plan approved before skill, persona, template, and Dutch pack files are authored.

# Design Specification: Lingua Ludi Core Architecture

**Document ID:** `docs/designs/001_lingua_ludi_core_architecture.md`

| Metadata | Value |
| :--- | :--- |
| Spec Version | 0.1.0 |
| Interface Contract Version | 0.1.0 |
| Status | Draft (Awaiting EGM Review & Human Sign-Off) |
| Author | Elephant Architect & Developer |
| Target Milestone | v0.1.0 (Core Harness, 24XX Micro-Scene Engine, Shadow Coach Loop & Staged Tier 1 Export) |

## Changelog

| Spec Version | Summary |
| :--- | :--- |
| 0.1.0 | Initial Elephant draft defining the three-layer architecture (Engine, Language Pack, Setting Cartridge), deterministic 24XX micro-scene mechanics, isolated three-pass agent contracts (Referee with optional Companion, Shadow Coach, Promoter), closed-loop spaced retrieval, and cross-language Tier 1 export staging compatible with `oude-mechanismen` Tier 1 catalogues. |

### Deferred Future Scope

The following capabilities are explicitly deferred past v0.1.0 and are not requirements of Interface Contract 0.1.0:

* Macro faction turn execution (such as integrating [TOME](https://github.com/efsalvarenga/tome) as a background geopolitical clock).
  Interface Contract 0.1.0 reserves an optional `world_events` hook on campaign state so a future macro layer can inject hooks without breaking v0.1 schemas.
* Audio input/output (local Whisper STT or TTS pipelines).
* Web UI, hosted deployment, authentication, and multi-player networking.
* Direct Git commits or pull requests written automatically into a parent campaign repository.

---

## 1. Overview & Objectives

Lingua Ludi is a deterministic, transport-neutral tabletop RPG harness for acquiring European languages through interactive play, while preserving canon discipline with external campaign universes.

Rather than prompting a single language model to juggle game rules, setting canon, pedagogical difficulty, and grammar correction simultaneously, Lingua Ludi separates deterministic mechanics from three strictly isolated agent passes:

$$\mathcal{T}(\text{CampaignState}_{t}, \text{PlayerTurn}) \longrightarrow (\text{CampaignState}_{t+1}, \text{TurnRecord}_{t+1}, \text{CoachEntry}_{t+1})$$

$$\mathcal{E}(\text{CampaignState}_{0..n}, \text{SettingCartridge}) \longrightarrow \text{Tier1ExportBundle}$$

### Goals

* **Strict Three-Layer Separation:** Decouple the generic **Engine**, reusable **Language Packs** (CEFR rubrics, register targets, error taxonomy), and read-only **Setting Cartridges** (parent Tier 1 catalogue + per-language term glossary).
* **Deterministic 24XX Mechanical Core:** Pure Python resolves 24XX risk rolls (`d4`–`d12` skill die, standard/advantage/disadvantage die selection, `1–2` disaster, `3–4` setback/cost, `5+` success) using an explicit seeded random stream, with zero network or LLM dependencies.
* **Three Isolated Agent Passes:** Keep **Referee** (in-character, target language, CEFR-bounded, optional Companion), **Shadow Coach** (out-of-character, silent post-turn analysis), and **Promoter** (session-end distillation and cross-language translation) cleanly separated so meta-feedback never leaks into fiction.
* **Closed-Loop Spaced Retrieval:** Score learner error tags and vocabulary gaps deterministically from the Shadow Coach ledger and inject a bounded set (`3`–`5`) of due retrieval targets into subsequent Referee context packs.
* **Safe Cross-Language Tier 1 Export:** Distil canon-altering outcomes from target-language play, translate them into the parent universe's canonical language, validate proper-noun consistency against the setting glossary and parent `catalog.json` paths/SHAs, and stage a spoiler-safe Markdown + JSON export bundle without mutating the parent repository.
* **Transport-Neutral Harness Boundary:** Provide versioned Pydantic schemas, pure-Python engine entry points, and a thin CLI/skill contract so any coding agent or caller can supply the LLM execution passes.

### Non-Goals

* Web application, HTTP API server, database persistence, or hosted cloud deployment.
* Embedding specific LLM SDKs, API keys, prompt-retry heuristics, or model selection inside `src/lingua_ludi/`.
* Hardcoding any specific European language or any specific campaign setting inside the core engine package.
* Automated direct writes or commits to a parent campaign repository.
* Enforcing linguistic grading as an in-world mechanical failure penalty in v0.1 (the Referee interprets learner intent charitably; the Shadow Coach records language errors silently).

### Alternatives Considered

| Selected approach | Alternative | Reason for v0.1.0 | Cost or limitation accepted |
| :--- | :--- | :--- | :--- |
| 24XX micro-scenes only in v0.1 (with `world_events` hook) | Hybrid 24XX + TOME faction clock from day one | Avoids maintaining two state machines, two turn cadences, and two prompt contracts in the first milestone; focuses directly on conversational fluency. | Macro world shifts must be injected manually or via `world_events` until a v0.2 macro adapter is specified. |
| Three separate orthogonal layers (Engine / Language Pack / Setting Cartridge) | Embedding language glossaries and CEFR rules inside setting cartridges | Allows pairing any European language pack with any setting cartridge without duplicating CEFR or coaching logic. | Requires a small per-language glossary mapping file (`glossary.<iso>.yml`) when translating setting proper nouns. |
| Isolated sequential Referee and Shadow Coach passes | Single prompt handling fiction, rules, and grammar correction | Prevents prompt dilution, voice drift, and out-of-character grammar commentary leaking into the scene. | Requires two caller invocations per player turn (one Referee, one Shadow Coach). |
| Charitable Referee + silent Shadow Coach ledger | In-world misunderstandings triggered automatically by grammar errors | Keeps A2–B1 play fluid and encouraging while still capturing complete diagnostic precision for spaced retrieval. | Learners wanting strict linguistic friction at B2–C1 must wait for an optional strictness mode in a later version. |
| Read-only `oude-mechanismen` `catalog.json` ingestion + local `exports/` staging | Direct edits or PRs against the parent vault | Eliminates any risk of playful language-learning detours corrupting Tier 1 campaign canon. | Requires a human review and manual copy/PR step to merge staged proposals into the parent vault. |

---

## 2. Domain Framing & Engine Execution Contract

### 2.0 Execution Boundary

The engine package (`src/lingua_ludi/`) is a pure-Python state transformer and validator.
The caller (a thin CLI wrapper, an interactive coding agent using repository skills, or a host script) owns file I/O, LLM provider calls, credentials, and user presentation.

The caller supplies trusted executable adapters (or invokes CLI stages between agent calls) for three distinct roles:

1. **Referee Adapter:** Receives a compiled `RefereeContextPack` and returns a structured `RefereeResponse` (scene narration, optional Companion line if permitted, risk-roll request if an action is risky, and proposed state updates).
2. **Shadow Coach Adapter:** Receives the learner's raw utterance, the Referee's preceding scene prompt, the active `LanguagePack`, and recent ledger history, and returns a structured `CoachEntry`.
3. **Promoter Adapter:** Receives the completed session turn log, the `SettingCartridge`, and the setting glossary, and returns a structured `Tier1ExportProposal` written in the parent universe's canonical language.

The engine validates every adapter payload against closed Pydantic v2 schemas, executes all deterministic mechanics, checks invariants, and fails atomically without mutating caller inputs.

### 2.1 Three Orthogonal Layers

1. **Engine (`src/lingua_ludi/`):** Contains only setting-agnostic, language-agnostic schemas, 24XX mechanics, spaced-retrieval selection, context-pack assembly, and export validation.
2. **Language Pack (`LanguagePack`):** Declares an ISO 639-1 language code (e.g. `Target Language X`), CEFR profile definitions (`A2`, `B1`, `B2`, `C1`) specifying vocabulary ceiling, sentence-complexity guidance, and register targets, plus a closed set of Coach error-category tags (e.g. `word_order`, `gender_article`, `verb_conjugation`, `preposition`, `lexical_choice`, `register`, `idiom`, `spelling`).
3. **Setting Cartridge (`SettingCartridge`):** Points to a read-only parent universe snapshot—specifically supporting the `oude-mechanismen` Tier 1 `.agents/context/catalog.json` format (entries with `path`, `title`, `type`, `parent`, `role`, `status`, `region`, `summary`, `links_out`, `links_in`, `sha`, `tier`)—paired with canonical-language metadata (e.g. `en-GB`) and an optional per-language `SettingGlossary` mapping canonical terms to target-language renderings or explicit `keep_untranslated` flags.

### 2.2 24XX Micro-Scene Mechanics

Lingua Ludi adopts core 24XX micro-scene resolution (CC BY 4.0 Jason Tocci):

* **Character Profile:** A player character holds a `name`, `concept`, a map of `skills` to die sizes (`d6`, `d8`, `d10`, `d12`; untrained baseline is `d6`, hindered baseline is `d4`), an `inventory` list of narrative items, a `hindrances` list of narrative conditions, and an optional `companion` configuration (`enabled: bool`, `name`, `role`, `intervention_policy`).
* **Risk Trigger:** Routine actions succeed without a roll.
  When the Referee declares an action risky, it specifies the `risk_statement` (what goes wrong on failure), `skill_used` (optional), and `roll_mode` (`normal`, `advantage`, `disadvantage`, or `hindered`).
* **Deterministic Die Selection & Roll:**
  * `normal`: roll 1 die of the character's skill size (default `d6` if untrained).
  * `hindered`: roll 1 `d4` regardless of skill die.
  * `advantage`: roll 2 dice of the skill size (or `d6` if untrained) and keep the highest.
  * `disadvantage`: roll 2 dice of the skill size (or `d6` if untrained) and keep the lowest.
* **Outcome Bands (based on kept die value):**

| Kept Result | Outcome Band | Meaning |
| :---: | :--- | :--- |
| `1–2` | `disaster` | Action fails and the stated risk occurs. |
| `3–4` | `setback` | Action succeeds with a complication, cost, or partial result. |
| `5+` | `success` | Action succeeds cleanly without triggering the stated risk. |

* **Seeded RNG:** Each campaign turn stream uses an explicit integer seed and deterministic draw counter so identical inputs and seeds reproduce identical rolls across runs on the same Python runtime.

### 2.3 Three-Pass Agent Separation & Optional Companion Policy

* **Pass 1 — Referee (In-Character, Target Language Only):**
  * Speaks strictly in the campaign's `target_language` within the active `cefr_level` ceiling.
  * Interprets learner phrasing charitably so grammatical errors do not stall play.
  * Subtly weaves the engine-selected `due_retrieval_targets` (3–5 items) into NPC dialogue, environment cues, or prompts without announcing them.
  * **Optional Companion Gate:** When `companion.enabled` is `false`, `RefereeResponse.companion_utterance` must be `null`.
    When `companion.enabled` is `true` and `intervention_policy` is `on_request` (the default), the companion speaks only when the learner explicitly flags `ask_companion: true` on the turn request (e.g. asking in-character how to say something).
    When `intervention_policy` is `active`, the companion may contribute at most one brief scaffolding utterance per turn (bounded by `max_companion_sentences`, default `2`), never taking over the PC's decision.
* **Pass 2 — Shadow Coach (Out-of-Character, Silent Post-Turn):**
  * Runs strictly after the player turn is recorded and never injects feedback into the in-character transcript.
  * Evaluates only the learner's utterance against the `LanguagePack` error taxonomy and active `cefr_level`.
  * Outputs structured `CorrectionItem` entries (original span, suggested target-language form, brief explanation in the learner's preferred feedback language, error tag, and `vocab_items` encountered or missed).
  * Zero-error turns are valid and record an empty `corrections` list plus optional `praise_notes` or `idiomatic_upgrades`.
* **Pass 3 — Promoter (Session-End Cross-Language Canonicalisation):**
  * Runs at session end (or arc boundary) over the append-only turn log.
  * Identifies only genuinely canon-altering events (e.g. an NPC condition change, a resolved local conflict, a newly established location or faction debt).
  * Translates each candidate into the parent setting's `canonical_language` (e.g. British English), preserving glossary terms verbatim.
  * Classifies each item as `update_existing_entry` (citing the exact `catalog.json` `path` and `sha`), `propose_new_entry`, or `session_chronicle_note`.

### 2.4 Closed-Loop Spaced Retrieval

The engine computes retrieval priority deterministically from the `LearnerLedger` without calling an LLM:

* Each tracked item (`VocabTarget` or recurring `ErrorPattern`) stores `item_id`, `surface_form`, `category`, `first_seen_turn`, `last_seen_turn`, `error_count`, `clean_use_count`, and `next_due_turn`.
* **Deterministic Interval Schedule:** New errors schedule `next_due_turn = current_turn + 1`.
  Each subsequent clean learner use confirmed by the Shadow Coach advances the interval along a fixed Fibonacci-like turn schedule (`1, 2, 4, 8, 16` turns); a repeated error resets the interval to `1` and increments `error_count`.
* **Selection Rule:** Before each Referee turn, `select_retrieval_targets(ledger, current_turn, limit)` selects up to `limit` (default `4`, valid range `0..5`) items where `next_due_turn <= current_turn`, ordered deterministically by `(overdue_turns DESC, error_count DESC, item_id ASC)`.

### 2.5 Cross-Language Tier 1 Export Pipeline

The export stage enforces a strict one-way staging barrier:

1. **Read-Only Parent Verification:** Load the parent `SettingCartridge` snapshot (`catalog.json` entries and `SettingGlossary`).
2. **Glossary Compliance Gate:** Every `SettingGlossary` term with `policy: keep_untranslated` or a canonical spelling must match its canonical form in the Promoter's canonical-language output; mismatches fail validation rather than silently drifting.
3. **Reference Integrity Gate:** Every `update_existing_entry` item must reference an existing `path` in the loaded `catalog.json` and record its observed `source_sha` so the human reviewer can detect upstream drift before applying the patch.
4. **Spoiler-Safe Staging Output:** Emit both `exports/<session_id>/tier1_proposal.json` (machine-validated structured bundle) and `exports/<session_id>/tier1_proposal.md` (human-readable Markdown diff/patch summary suitable for manual review or PR authoring into the parent vault).
   No file inside the parent repository path is ever opened for writing.

### 2.6 Macro / TOME Extension Boundary

To allow a future macro faction engine (such as TOME) to drive high-level world shifts without modifying v0.1 schemas:

* `CampaignState` includes an optional append-only list `world_events: list[WorldEvent]`, where each event carries `event_id`, `source` (e.g. `"manual"` or `"tome"`), `summary_canonical`, `summary_target` (optional), `related_catalog_paths`, and `injected_at_turn`.
* In v0.1, `world_events` defaults to empty and may be populated manually by the caller; the Referee context pack includes active unresolved `world_events` as background hooks.

### 2.7 Privacy, Budgets & Trusted-Adapter Boundaries

* Learner transcripts and coaching logs are personal study artefacts and reside strictly in local session storage excluded from core repository fixtures.
* Adapter callables are trusted caller-selected integrations; the engine isolates its own working state via deep-copied immutable snapshots but does not sandbox external network calls made by caller adapters.
* Context packs enforce explicit configurable ceilings (`max_recent_turns`, default `6`; `max_catalog_entries`, default `12`; `max_retrieval_targets`, default `4`) to bound token usage deterministically.

---

## 3. State Schema & Data Models

### 3.0 Value Rules & Text Normalisation

* All models use Pydantic v2 with `extra="forbid"` (closed schemas) and `strict=False` only for JSON deserialisation of enums/UUIDs where explicitly typed, rejecting unknown fields and null overrides of required containers.
* String identifiers (`campaign_id`, `session_id`, `language_code`, `item_id`, `term_id`) must be non-empty, stripped, and free of control characters.
* `language_code` follows BCP-47 primary/region tags (e.g. `nl`, `en-GB`, `fr`, `de`, `es`).
* Text comparison for glossary matching and deduplication uses Unicode NFC normalisation, case-folding, and collapsed ASCII whitespace, while preserving original display text verbatim in persisted records.

### 3.1 Language Pack Schema (`LanguagePack`)

| Field | Type | Constraints |
| :--- | :--- | :--- |
| `contract_version` | `Literal["0.1.0"]` | Must equal `"0.1.0"`. |
| `language_code` | `str` | Valid BCP-47 tag (e.g. `"nl"`). |
| `display_name` | `str` | Non-empty human label. |
| `cefr_profiles` | `dict[CefrLevel, CefrProfile]` | Must include at least one of `A2`, `B1`, `B2`, `C1`. |
| `error_taxonomy` | `list[ErrorTagDefinition]` | Non-empty unique list of allowed Coach error tags. |
| `scaffolding_phrases` | `dict[str, str]` | Optional reusable companion/clarification cues in target language. |

Each `CefrProfile` specifies:

* `level`: `Literal["A2", "B1", "B2", "C1"]`
* `max_sentence_words`: `int` (positive upper guideline for Referee prose)
* `register_guidance`: `str`
* `vocabulary_ceiling_notes`: `str`
* `grammar_focus`: `list[str]`

### 3.2 Setting Cartridge Schema (`SettingCartridge`)

| Field | Type | Constraints |
| :--- | :--- | :--- |
| `contract_version` | `Literal["0.1.0"]` | Must equal `"0.1.0"`. |
| `setting_id` | `str` | Non-empty slug (e.g. `"setting-a"`). |
| `canonical_language` | `str` | BCP-47 tag of parent vault canon (e.g. `"en-GB"`). |
| `source_engine_version` | `str \| None` | Copied from `catalog.json`'s `engine_version` when loaded from an `oude-mechanismen` vault. |
| `tone_summary` | `str` | Compact setting tone calibration (non-empty). |
| `entries` | `list[CatalogEntry]` | Non-empty read-only Tier 1 entity index. |
| `glossary` | `SettingGlossary` | Per-target-language proper-noun and jargon mapping. |

Each `CatalogEntry` mirrors `oude-mechanismen`'s `.agents/context/catalog.json` Tier 1 record:

* `path: str` (relative vault path, unique within `entries`)
* `title: str`
* `type: str`
* `parent: str | None`
* `role: str | None`
* `status: str | None`
* `region: str | None`
* `summary: str`
* `links_out: list[str]`
* `links_in: list[str]`
* `sha: str`
* `tier: Literal[1]`

Each `SettingGlossaryEntry` specifies:

* `term_id: str`
* `canonical_term: str`
* `target_language: str`
* `target_rendering: str`
* `policy: Literal["keep_untranslated", "preferred_translation"]`
* `related_catalog_path: str | None`

### 3.3 Campaign, Character & Scene State (`CampaignState`)

| Field | Type | Constraints |
| :--- | :--- | :--- |
| `contract_version` | `Literal["0.1.0"]` | Must equal `"0.1.0"`. |
| `campaign_id` | `str` | Non-empty identifier. |
| `target_language` | `str` | Must match loaded `LanguagePack.language_code`. |
| `feedback_language` | `str` | Language for Shadow Coach explanations (e.g. `"en-GB"`). |
| `cefr_level` | `Literal["A2", "B1", "B2", "C1"]` | Must exist in `LanguagePack.cefr_profiles`. |
| `setting_id` | `str` | Must match loaded `SettingCartridge.setting_id`. |
| `turn_number` | `int` | Monotonically non-decreasing integer starting at `0`. |
| `rng_seed` | `int` | Seed for deterministic 24XX rolls. |
| `rng_draws` | `int` | Count of rolls already consumed from the seeded stream (starts at `0`). |
| `character` | `CharacterSheet` | Skills (`d6`–`d12`), inventory, hindrances, optional `CompanionConfig`. |
| `active_scene` | `SceneState` | Location (`catalog_path` optional), present NPCs, open stakes, scene summary. |
| `world_events` | `list[WorldEvent]` | Append-only background hooks (reserved macro/TOME extension point). |
| `turn_history` | `list[TurnRecord]` | Append-only chronological record of resolved turns. |
| `learner_ledger` | `LearnerLedger` | Persistent vocabulary and error tracking state. |

### 3.4 Turn Request, Context Pack, Turn Record & Coach Entry

* **`PlayerTurnInput`:**
  * `utterance: str` (non-empty learner input in `target_language`)
  * `ask_companion: bool = False`
  * `focus_catalog_paths: list[str] = []` (optional explicit Tier 1 entities to include in Referee context)
* **`RefereeContextPack`:**
  * Built deterministically by the engine from `CampaignState`, `LanguagePack`, `SettingCartridge`, and `select_retrieval_targets(...)`.
  * Enforces `max_recent_turns`, `max_catalog_entries`, and `max_retrieval_targets` budgets.
* **`RiskAssessment` & `RollRecord`:**
  * `RiskAssessment`: `risk_statement: str`, `skill_used: str | None`, `roll_mode: Literal["normal", "advantage", "disadvantage", "hindered"]`.
  * `RollRecord`: `die_size: Literal[4, 6, 8, 10, 12]`, `raw_rolls: list[int]`, `kept_roll: int`, `outcome_band: Literal["disaster", "setback", "success"]`, `draw_index: int`.
* **`RefereeResponse`:**
  * `narration: str` (target language, non-empty)
  * `companion_utterance: str | None` (must be `None` when companion is disabled or unrequested under `on_request`)
  * `risk_assessment: RiskAssessment | None`
  * `scene_update: ScenePatch` (updated stakes, location, present NPCs, added/removed hindrances or items)
  * `canon_candidate_flag: bool` (lightweight hint for session-end Promoter; never mutates parent canon directly)
* **`CoachEntry`:**
  * `turn_number: int`
  * `corrections: list[CorrectionItem]` (each tag must exist in `LanguagePack.error_taxonomy`)
  * `confirmed_clean_item_ids: list[str]` (tracked ledger items used correctly in this turn)
  * `new_vocab_items: list[VocabCue]`
  * `idiomatic_upgrades: list[str]`
  * `summary_feedback: str` (in `feedback_language`)

### 3.5 Tier 1 Export Proposal & Bundle (`Tier1ExportBundle`)

* **`Tier1ProposalItem`:**
  * `proposal_id: str`
  * `kind: Literal["update_existing_entry", "propose_new_entry", "session_chronicle_note"]`
  * `target_catalog_path: str | None` (required for `update_existing_entry`; must match an entry in `SettingCartridge.entries`)
  * `observed_source_sha: str | None` (required for `update_existing_entry`; must equal the loaded entry's `sha`)
  * `title_canonical: str` (in `SettingCartridge.canonical_language`)
  * `summary_canonical: str` (spoiler-safe, neutral canon text in `canonical_language`)
  * `source_turn_numbers: list[int]` (non-empty list of valid turns in `turn_history`)
  * `glossary_terms_checked: list[str]`
* **`Tier1ExportBundle`:**
  * `contract_version: Literal["0.1.0"]`
  * `campaign_id: str`
  * `session_id: str`
  * `setting_id: str`
  * `source_language: str`
  * `canonical_language: str`
  * `generated_from_turn_range: tuple[int, int]`
  * `proposals: list[Tier1ProposalItem]`
  * `rendered_markdown: str` (deterministic Markdown rendering for `exports/<session_id>/tier1_proposal.md`)

### 3.6 Runner Entry Points & Harness CLI Boundary

Core pure-Python library API (`src/lingua_ludi/engine.py`):

* `load_oude_catalog(catalog_payload: dict, setting_id: str, canonical_language: str, tone_summary: str, glossary: SettingGlossary) -> SettingCartridge`
* `build_referee_context(state: CampaignState, language_pack: LanguagePack, cartridge: SettingCartridge, turn_input: PlayerTurnInput, budgets: ContextBudgets | None = None) -> RefereeContextPack`
* `resolve_risk_roll(state: CampaignState, assessment: RiskAssessment) -> tuple[RollRecord, int]` (returns roll record and updated `rng_draws`; never mutates `state`)
* `execute_turn(state: CampaignState, language_pack: LanguagePack, cartridge: SettingCartridge, turn_input: PlayerTurnInput, referee_adapter, coach_adapter, budgets: ContextBudgets | None = None) -> TurnResult`
* `stage_tier1_export(state: CampaignState, cartridge: SettingCartridge, session_id: str, promoter_adapter) -> Tier1ExportBundle`

A thin CLI (`src/lingua_ludi/cli.py`, invoked via `uv run lingua-ludi ...`) wraps file persistence (JSON/YAML campaign files and `exports/` staging) and exposes discrete deterministic subcommands (`init`, `context`, `roll`, `apply-turn`, `apply-coach`, `stage-export`, `validate`) so coding agents (`agy`, `vibe`, `codex`) can run each pass step-by-step without needing a Python host script.

---

## 4. Invariants & Rules

1. **Core Agnosticism:** `src/lingua_ludi/` contains zero hardcoded setting names, lore strings, or target-language word lists.
2. **Input Immutability:** Every engine function treats supplied `CampaignState`, `LanguagePack`, and `SettingCartridge` instances as immutable and returns fresh validated models.
3. **Deterministic Seeded Rolls:** Given identical `rng_seed`, `rng_draws`, `CharacterSheet.skills`, and `RiskAssessment`, `resolve_risk_roll` returns the exact same `RollRecord` and advances `rng_draws` by exactly `1` (normal/hindered) or `2` (advantage/disadvantage) individual die draws.
4. **Closed-Schema Validation:** Unknown fields, wrong contract versions, out-of-range die sizes, or missing required fields raise a typed `LinguaLudiError` before any state transition is published.
5. **Companion Gate Enforcement:** If `character.companion.enabled` is `false`, or if `intervention_policy == "on_request"` and `turn_input.ask_companion` is `false`, any non-null `RefereeResponse.companion_utterance` is rejected with `RoleBoundaryError`.
6. **Taxonomy Grounding:** Every `CorrectionItem.error_tag` emitted by the Shadow Coach must exist in `LanguagePack.error_taxonomy`; unknown tags raise `CoachValidationError`.
7. **Deterministic Spaced Retrieval:** `select_retrieval_targets` is pure and total; ties break deterministically by `(overdue_turns DESC, error_count DESC, item_id ASC)`, never exceeding `budgets.max_retrieval_targets`.
8. **Ledger Consistency:** A turn's `CoachEntry` may not mark the same `item_id` as both erred and cleanly used in the same turn; conflicting assertions raise `CoachValidationError`.
9. **Turn Atomicity:** `execute_turn` publishes either both the resolved `TurnRecord` and its paired `CoachEntry` (advancing `turn_number` by `1`) or raises a typed error with zero partial mutation.
10. **Strict Read-Only Parent Boundary:** No engine function or CLI subcommand accepts a write target inside the parent setting vault; `stage_tier1_export` returns an in-memory `Tier1ExportBundle` and the CLI writes exclusively under the workspace's `exports/<session_id>/` directory.
11. **Export Reference & Glossary Integrity:** Every `update_existing_entry` proposal must cite an existing `CatalogEntry.path` and matching `sha`, every `source_turn_numbers` entry must exist in `CampaignState.turn_history`, and every `keep_untranslated` glossary term referenced by a proposal must appear in its canonical form.
12. **No Meta-Leak in Export:** Exported `summary_canonical` text must not contain learner coaching metrics, CEFR ratings, or error logs (enforced via deterministic schema separation: `Promoter` receives narrative turn records and canon metadata, never `LearnerLedger` or `CoachEntry` payloads).

---

## 5. Execution & State Mutation Logic

### Inputs & Triggers

* **Turn Trigger:** Caller submits `(CampaignState_t, LanguagePack, SettingCartridge, PlayerTurnInput)` plus `referee_adapter` and `coach_adapter` (or invokes the equivalent discrete CLI commands).
* **Export Trigger:** Caller submits `(CampaignState, SettingCartridge, session_id)` plus `promoter_adapter` at session end.

### Turn Execution (`execute_turn`)

1. **Pre-Flight Validation:** Verify `contract_version`, matching `target_language` and `setting_id`, valid `cefr_level`, and non-empty `turn_input.utterance`.
2. **Context Assembly:** Run `select_retrieval_targets(state.learner_ledger, state.turn_number + 1, budgets.max_retrieval_targets)` and construct `RefereeContextPack` containing active scene, character sheet, bounded recent turns, relevant `CatalogEntry` summaries, unresolved `world_events`, active `CefrProfile`, and due retrieval targets.
3. **Pass 1A — Risk Assessment & Narration:** Invoke `referee_adapter(context_pack, roll_record=None)`.
   Validate `RefereeResponse` schema and Companion Gate (Invariant 5).
4. **Deterministic 24XX Resolution (if risky):** If `response.risk_assessment` is present, compute `(roll_record, next_rng_draws) = resolve_risk_roll(state, response.risk_assessment)` and invoke `referee_adapter(context_pack, roll_record=roll_record)` to obtain the final outcome narration and `scene_update` grounded in `roll_record.outcome_band`.
   Re-validate `RefereeResponse` and Companion Gate.
5. **Pass 2 — Shadow Coach:** Invoke `coach_adapter(coach_input)` with only the learner utterance, preceding Referee prompt, `LanguagePack`, and relevant tracked ledger items.
   Validate `CoachEntry` against `LanguagePack.error_taxonomy` and Invariant 8.
6. **Atomic State Transition:** Construct `CampaignState_{t+1}` with `turn_number = t + 1`, updated `rng_draws`, applied `scene_update`, appended `TurnRecord`, and deterministically updated `LearnerLedger`.
   Return `TurnResult(state=CampaignState_{t+1}, turn_record=..., coach_entry=...)`.

### Session-End Promotion (`stage_tier1_export`)

1. **Sanitised Input Assembly:** Build `PromoterInput` from `CampaignState.turn_history` (narrative fields only; excluding `LearnerLedger` and `CoachEntry`), `SettingCartridge.entries`, and `SettingCartridge.glossary`.
2. **Pass 3 — Promoter:** Invoke `promoter_adapter(promoter_input)` to obtain candidate `Tier1ProposalItem` records in `SettingCartridge.canonical_language`.
3. **Deterministic Validation Gates:** Verify `source_turn_numbers`, `target_catalog_path` + `observed_source_sha` matches, and `keep_untranslated` glossary compliance (Invariant 11).
4. **Deterministic Staging Render:** Render `rendered_markdown` and return `Tier1ExportBundle`.

### Outputs & Events

* Per turn: immutable `TurnResult` (`CampaignState_{t+1}`, `TurnRecord`, `CoachEntry`).
* Per export: immutable `Tier1ExportBundle` (`proposals` + `rendered_markdown`), ready to be written to `exports/<session_id>/tier1_proposal.json` and `exports/<session_id>/tier1_proposal.md`.

---

## 6. Edge Cases & Failure Modes

All engine errors inherit from `LinguaLudiError` and provide a serialisable `to_failure()` payload with a stable error code:

| Error Subclass | Trigger Condition | Handling |
| :--- | :--- | :--- |
| `SchemaValidationError` | Invalid Pydantic shape, unknown extra field, unsupported `contract_version`, or invalid die size. | Reject immediately; zero state mutation. |
| `ConfigurationMismatchError` | `CampaignState.target_language != LanguagePack.language_code`, `cefr_level` missing from pack, or `setting_id` mismatch. | Reject before invoking any adapter. |
| `RoleBoundaryError` | Companion utterance returned when companion disabled or unrequested under `on_request`, or Companion exceeds `max_companion_sentences`. | Reject Referee response; zero state mutation. |
| `CoachValidationError` | Unknown `error_tag` outside `LanguagePack.error_taxonomy`, or conflicting error + clean-use claim on the same `item_id` in one turn. | Reject Coach entry; fail turn atomically. |
| `CartridgeIntegrityError` | Duplicate `CatalogEntry.path`, non-Tier-1 entry in `catalog.json`, or malformed `SettingGlossary`. | Reject cartridge load before play or export. |
| `ExportValidationError` | Proposal cites unknown `target_catalog_path`, mismatched `observed_source_sha`, out-of-range `source_turn_numbers`, or alters a `keep_untranslated` glossary term. | Reject export; write no staging files. |
| `AdapterExecutionError` | Caller-supplied adapter raises an unexpected exception or returns a non-conforming type. | Wrap cleanly; publish no partial state. |

### Specific Edge Cases

* **Zero-Error Learner Turn:** Valid; `CoachEntry.corrections` is empty, `confirmed_clean_item_ids` still advances spaced-retrieval intervals, and `turn_number` increments normally.
* **Empty Export Session (No Canon-Altering Events):** Valid; `Tier1ExportBundle.proposals` is `[]` and `rendered_markdown` explicitly records that no Tier 1 canon changes were proposed.
* **Untrained or Hindered 24XX Skill:** Missing skill key in `CharacterSheet.skills` defaults deterministically to `d6`; `roll_mode == "hindered"` always uses `d4` regardless of skill die.
* **Stale Parent `catalog.json`:** If the parent vault changes after play begins, `observed_source_sha` preserves the exact SHA seen during play so the human reviewer immediately spots drift before applying the staged patch.
* **Empty `world_events` or Empty `SettingGlossary`:** Both default cleanly to empty lists without blocking play or export.

---

## 7. Verification & Automated Test Strategy

Verification runs through `./scripts/lint.sh` (`ruff check .`, `ruff format --check .`, and `pytest` via locked `uv`).

### 7.1 Unit & Contract Tests (`tests/`)

* **Schema & Value Tests (`test_contracts.py`):** Verify closed schemas reject unknown fields, enforce `contract_version == "0.1.0"`, validate BCP-47 tags, reject non-Tier-1 catalogue entries, and enforce `ConfigurationMismatchError`.
* **24XX Mechanics & Golden Vectors (`test_mechanics.py`):** Assert exact golden roll sequences across fixed seeds for `d4`, `d6`, `d8`, `d10`, `d12` under `normal`, `hindered`, `advantage`, and `disadvantage`; verify outcome band boundaries (`1–2`, `3–4`, `5+`) and exact `rng_draws` increments.
* **Role Separation & Turn Atomicity (`test_turn_loop.py`):** Verify Companion gate rejection (`RoleBoundaryError`), two-stage Referee call when a risk roll triggers vs. single call on routine action, Coach taxonomy checks, conflicting error/clean-use rejection, and zero mutation of caller input state on both success and failure.
* **Spaced-Retrieval Selector (`test_retrieval.py`):** Verify Fibonacci-like interval advancement on clean use, reset to `+1` on repeated error, deterministic tie-breaking `(overdue DESC, error_count DESC, item_id ASC)`, and hard `max_retrieval_targets` ceiling.
* **Cartridge Ingestion & Tier 1 Export Staging (`test_export.py`):** Verify `oude-mechanismen` `catalog.json` parsing, `PromoterInput` redaction of all `LearnerLedger` and `CoachEntry` data, `target_catalog_path` + `observed_source_sha` verification, `keep_untranslated` glossary enforcement, and zero filesystem writes outside staging.

### 7.2 First Concrete Testbed Validation (Outside Core Package)

Once core tests pass using neutral placeholders, an integration fixture suite validates the real-world target pairing without coupling core code to either pack:

* **Language Pack `nl` (Dutch):** Complete `A2`–`C1` CEFR profiles and Dutch error taxonomy (`de_het`, `v2_word_order`, `separable_verb`, `er_construction`, `preposition`, `lexical_choice`, `register`).
* **Setting Cartridge Fixture (`hostile`):** Read-only fixture snapshot shaped identically to `terra-campaigns/hostile`'s `.agents/context/catalog.json` with a Dutch setting glossary (`glossary.nl.yml`) testing cross-language export back into British English (`en-GB`).

---

## 8. Goldfish Review Gates

### Current Draft: Spec Version 0.1.0

* **Status:** Draft — awaiting zero-context EGM review (`docs/reviews/001_lingua_ludi_core_architecture_review.md`) and explicit human developer sign-off before creating `docs/plans/001_lingua_ludi_core_architecture_plan.md` or writing production code.
* **Sign-Off Criteria:**
  1. EGM review loop converged with all blocking findings resolved.
  2. Human developer (`efsalvarenga`) explicitly approves the specification and records `Status: Approved`.
  3. Implementation plan approved before TDD code implementation starts.

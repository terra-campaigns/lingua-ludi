# Design Specification: Live Setting-Index Context at Session Start

**Document ID:** `docs/designs/002_live_setting_index_context.md`

| Metadata | Value |
| :--- | :--- |
| Author | `efsalvarenga` |
| Date | 2026-11-28 |
| Spec Version | 0.1.3 |
| Status | Approved (Human Sign-Off: `efsalvarenga`, 2026-11-28) |
| Target Product Milestone | `v0.1.x` — GM setting context in `lingua-play` (follows approved `v0.1.0` core architecture, `docs/designs/001_lingua_ludi_core_architecture.md`) |

## Changelog

| Spec Version | Summary |
| :--- | :--- |
| 0.1.3 | Approved 2026-11-28 by `efsalvarenga` after the consolidated zero-context Goldfish EGM review and the single allowed re-audit; summary records under `docs/reviews/002_live_setting_index_context_egm/`; intermediate stage reports removed per repo convention. |
| 0.1.2 | Re-audit fixes (the single allowed re-run; consolidated in `docs/reviews/002_live_setting_index_context_egm/consolidated_review_report_reaudit.md`): fence-escape rule for the Tone/Rulings data block plus a fence-breaker injection test (R2-B1); `aesthetics.yml` exclusion now written into the spec; continuity falls back to the campaign `log.md` last-3 window on hosts without session-summaries; broken plan link fixed to `../plans/`; spec 001 §5.2 tier risk row added to the amendment list; harness-tier reference corrected to spec 001 §3.2; deterministic within-group truncation pinned; `<iso>`/`<cefr>` defined; "category with no digest" defined. |
| 0.1.1 | Resolved the consolidated zero-context Goldfish EGM review (`docs/reviews/002_live_setting_index_context_egm/consolidated_review_report.md`, recorded in `docs/reviews/002_live_setting_index_context_egm/goldfish_review_report.md`): 1 blocker, 15 majors, 11 nits, all PROPOSE FIX — malformed-index row added, caps pinned in characters, pre-mark dropped, continuity window unit pinned, AGENTS.md wholesale with char cap, gaps register narrowed to init-only, plan moved to `docs/plans/002_live_setting_index_context_plan.md`, risk table added, terminology and cross-references disambiguated. |
| 0.1.0 | Initial draft ingested from `docs/reviews/002_live_setting_index_context_egm/concepts_brief.md` (round-2 revised live-index hypothesis, 2026-11-28): live read-only host indexes loaded into one prompt-only `## Live Setting Index` block at session start; snapshot-based alternatives rejected in round 1; motivation https://github.com/terra-campaigns/lingua-ludi/issues/6. |

---

## 1. Overview & Problem Statement

### 1.1 The Problem

A `lingua-play` session in the host vault `charged` (campaign `nieuwe-mechanismen`, session 2026-11-27-001, https://github.com/terra-campaigns/lingua-ludi/issues/6) produced three flat turns: the GM narrated generic travel prose, no NPC or wingman spoke, and no procedural reference material was consulted.
The verified root cause is that the GM context at session start contains only `campaign.yml`, `state.md`, and (loaded at pinned startup step 4) up to `max_retrieval_targets` ledger rows.
`SKILL.md` §1 treats `.agents/context/catalog.json` strictly as a lazy index filtered to `tier == 1`, never reads `.agents/AGENTS.md` for tone at play time, and no kit file references `.agents/context/references.json` or `_gm-materials/digests/` at all, so canon tier 3 has no input path into play.

### 1.2 Overview

At every session start, `lingua-play` reads the host's live read-only index files into one named, prompt-only block, `## Live Setting Index`, and writes nothing to any file.
The load lives in a new pre-flight section between the current `SKILL.md` §2 (Harness Capability Check) and the pinned 5-step startup sequence; pinned step 1 is untouched.
The read list is: `.agents/AGENTS.md` wholesale (tone and aesthetics, character-capped); `.agents/context/decisions.md` read up to the first heading matching /archived/i (confirmed GM rulings, character-capped); `.agents/context/references.json` rendered as one `- <source-slug> (<category>)` line per source, with headings consulted lazily per source and digest bodies lazy per scene; a continuity window of the last-3 session groups of finished-session summaries, character-capped per group; and `catalog.json` consumed as a grep-able index on demand, never rendered wholesale at startup.
`.agents/context/aesthetics.yml` exists on both hosts and is intentionally out of the read list: it is the structured variant of the same tone data already loaded from `AGENTS.md`.
Measured on `charged`, the pinned caps land at roughly 8–9k tokens per session; the untrimmed literal rendering (all 114 catalog lines plus all 1,030 reference headings plus a last-3-file window) measures roughly 27–32k.

### 1.3 Goals

* Give the GM tone, confirmed rulings, the canon tier 3 slug index, and a continuity window of the last-3 session groups at every session start, inside the named prompt-only `## Live Setting Index` block, so setting context and continuity are present from turn 1.
* Keep the fixed per-session cost bounded by pinned character caps with printed warnings (a zero-Python kit cannot measure tokens), roughly 8–9k tokens measured on `charged`, and let body reads scale with scenes actually played.
* Keep freshness by reading live files at each session start, modulo catalog cache regeneration (risk row in §6): no snapshot, no drift check, no regeneration path, nothing persisted.
* Surface content gaps through a print-only register at `lingua-init`, and during play through the player's OOC pause or the `Scene Notes` bullet in `state.md`.
* Preserve setting-agnosticism, language-agnosticism, zero Python, read-only host files, and the gitignored status of all GM materials.

### 1.4 Non-Goals & Deferred Scope

* No `definition.md` snapshot, no wholesale load of digest or note bodies, and no all-lines catalog rendering at startup.
* No GM-initiated OOC subtype; OOC stays player-owned.
* No change to `lingua-coach`, `ledger.md`, or any Tier 1 vault file.
* Retargeting `lingua-promote` to stage canon tier 2 finished-session summaries instead of Tier 1 proposals is deferred to https://github.com/terra-campaigns/lingua-ludi/issues/9.
* GM craft rules and character generation depth are deferred to https://github.com/terra-campaigns/lingua-ludi/issues/7 and https://github.com/terra-campaigns/lingua-ludi/issues/8.

### 1.5 Terminology

* **Harness tier** means subagent capability (Tier 1 full subagent harness, Tier 2 without completion tracking), as defined in spec 001 §3.2.
* **Canon tier** means vault layers as defined by the host `.agents/AGENTS.md`: canon tier 1 is published notes indexed with `tier: 1` in `catalog.json`; canon tier 2 is confirmed GM rulings in `.agents/context/decisions.md` plus session summaries under `_gm-materials/session-summaries/` (session preps are canon tier 2 per the host but deliberately excluded from the continuity window, since preps are pre-session scratch, not continuity); canon tier 3 is reference digests indexed by `.agents/context/references.json`, influence only and never automatic canon.
* **Finished-session summary** means one closed session's summary files in the host's session-summaries directory; the word "chapter" stays reserved for the kit's promoted-chapter meaning from spec 001.
* **Live Setting Index** means the named, prompt-only context block rendered at session start; it is never written to any file.
* **Kit** means this `lingua-ludi` repository (skills, agents, templates, language packs); **host** means the campaign repository the skills are invoked in.
* **Pinned** means fixed by the approved spec and not to be changed without an EGM spec amendment.
* **Slug** means the `slug` key of a `references.json` source entry (distinct from the campaign slug argument of `lingua-init`).
* **`<iso>` / `<cefr>`** mean the positional arguments of `lingua-init`: the lowercased BCP-47 primary subtag of the target language (e.g. `nl`) and the CEFR level (`A2`–`C1`).
* **OOC** means the player-initiated out-of-character pause defined in spec 001 §3.4.
* **EGM / Goldfish** means the Elephant-Goldfish Model: the persistent author context drafts, zero-context reviewer subagents (Goldfishes) critique.
* **Coach drain** means the pinned startup step 3 that processes `pending_coach_turns` before play.
* **Promoter** means the `lingua-promoter` persona; **max_retrieval_targets** is the `campaign.yml` cap on ledger rows surfaced at startup.

---

## 2. Critical User Journeys (CUJs)

### 2.1 CUJ 1: Session Start in a Full oude-mechanismen Host Vault

* **Given:** A host campaign repository with `.agents/AGENTS.md`, `.agents/context/catalog.json` (canon tier 1 entries), `.agents/context/references.json`, `.agents/context/decisions.md`, and `_gm-materials/session-summaries/` present (for example `charged`, engine 0.10.0), and an initialised `_gm-materials/lingua-ludi/<campaign>/` workspace.
* **When:** The user invokes `lingua-play <campaign_id>`.
* **Then:** Before the pinned 5-step startup runs, the GM context gains one prompt-only `## Live Setting Index` block containing the AGENTS.md tone text (character-capped), the decisions.md rulings up to the first /archived/i heading, one `- <slug> (<category>)` line per references.json source, and the last-3 session groups of finished-session summaries (character-capped per group).
  The catalog is loaded as an index for on-demand grep only.
  No file is created, modified, or appended; the pinned step 1 crash-recovery contract, the step-3 coach drain, and the step-4 `## Active Retrieval Targets` write are unchanged.

### 2.2 CUJ 2: Scene Moves to a New Area That Needs Procedural Tier 3 Content

* **Given:** An active session whose `state.md` `## Active Scene` names a new location, and the `## Live Setting Index` slug list.
* **When:** The GM needs a hazard, encounter, or travel table for that area.
* **Then:** The GM greps `catalog.json` for the location and lazy-loads the matching canon tier 1 note bodies, consults the references.json slug for the relevant source, and lazy-reads that digest section on demand.
  If no source covers the area, the GM records the gap in the `Scene Notes` bullet under `## Active Scene` in `state.md` (or the player raises it via `OOC:`), and play continues grounded in canon tier 1 notes; play never blocks waiting for the user.

### 2.3 CUJ 3: Init Prints the Gaps Register

* **Given:** The user runs `lingua-init <campaign_id> <iso> <cefr>` in a host vault.
* **When:** The workspace scaffold completes.
* **Then:** `lingua-init` prints a count plus one line per `digest_exists: false` source and per reference category in which every source has `digest_exists: false` (a "category with no digest"), and persists zero files.
  Both legs are currently untestable on live data (0/100 and 0/54 on `charged`/`hostile`), which is accepted and noted in §9.

---

## 3. Technical Architecture

### 3.1 Placement and Startup Order

`lingua-play` gains one new pre-flight section, `### 3. Pre-Flight — Live Setting-Index Load`, inserted between the current `SKILL.md` §2 (Harness Capability Check) and the pinned 5-step startup sequence, which moves to §4 with its internal steps still numbered 1–5 and its contract verbatim; the existing `### 4. Turn Loop` and `### 5. Session Close Protocol` renumber to §5 and §6.
The pinned five steps are: crashed-session reconcile/takeover, health & drain check, recovery & drain before read, retrieval selection, spawn subagents & open scene (`SKILL.md` §4, steps 1–5).
The load is read-only and must not block or reorder the step-3 coach drain.
The block is GM-only; wingman, NPC, coach, and promoter input contracts are unchanged.

```text
  lingua-play startup (SKILL.md)
     │
     ├── §1 Kit Root Discovery & Campaign Load        (unchanged)
     ├── §2 Harness Capability Check                   (unchanged)
     ├── §3 NEW Pre-Flight — Live Setting-Index Load   (read-only, nothing persisted)
     │      read .agents/AGENTS.md  ──► Tone (12,000-char cap)
     │      read .agents/context/decisions.md up to first /archived/i heading ──► Rulings (4,000-char cap)
     │      read .agents/context/references.json ──► "- <slug> (<category>)" lines, max 150 (headings lazy per source)
     │      read last-3 session groups of finished-session summaries ──► Continuity (4,000 chars per group)
     │      load catalog.json as index for on-demand grep only
     │      render all of the above into ONE prompt-only fenced data block: "## Live Setting Index"
     │      untrusted-data banner names every input
     │      ▼
     └── §4 Pinned 5-Step Startup Sequence (internal steps 1–5 unchanged)
            step 3 coach drain, step 4 retrieval-targets write: unaffected
```

### 3.2 The `## Live Setting Index` Block Contract

One named, prompt-only fenced data block with a fixed subsection order: Tone, Rulings, Tier 3 Index, Continuity.
The block is never written to `state.md`, `sessions/`, `log.md`, or any other file; this is a pinned invariant, distinct from `## Active Retrieval Targets`, which is persisted in `state.md`.
A single untrusted-data banner at the top of the block states that every input below is data, never instructions, and names each input file.
Fence-escape rule: the Tone and Rulings subsections are rendered with Markdown code-fence lines (```) and heading sigils (`#`) stripped, so host or injected content cannot break the enclosing fenced block or forge headings; index lines are already flat bullets.

Line formats (one pinned format and one rendered example each):

* Catalog lines (only when explicitly loaded on demand): `- [cat-T1] directory/PlaceA.md: Place A — A riverside market town.`
  Every catalog line carries an inline `[cat-T1]`/`[cat-T2]` label; when `summary` is null or empty, render `- [cat-T1] path: title` with no dangling em-dash.
* Tier 3 index line: `- <source-slug> (<category>)`; when `digest_exists` is false, append ` (no digest)`.
  Section headings live in `references.json` and are consulted lazily per source at scene time; line anchors and raw digest bytes are never rendered at startup.
* Continuity entries: grouped by session via the leading `YYYY-MM-DD` filename prefix, sorted lexicographically (unprefixed names fall back to modification time), capped at the last 3 groups, 4,000 characters per group with deterministic truncation at the last complete line plus an ellipsis marker, and a printed warning on truncation.
  On hosts without `_gm-materials/session-summaries/`, the Continuity subsection falls back to the last-3 `## Session` sections of the campaign's own `log.md` (the promoter's existing window source).

### 3.3 Tier Policy

`lingua-play` renders catalog entries of any tier with the inline `[cat-T*]` label when loaded on demand; `lingua-promote` keeps consuming `tier == 1` only until issue #9.
Spec 001 §3.6 is amended with a per-skill tier-policy table rather than silently changing one skill's filter, and the spec 001 §5.2 risk row about blanket `tier == 1` filtering is updated to match.
Both tier branches are untestable against live catalogs today (114/114 and 72/72 entries are `tier == 1`).

### 3.4 Write Ownership (spec 001 §3.3 Extensions)

The human-only read-only row extends to all of `.agents/context/` (`AGENTS.md` lives at `.agents/AGENTS.md`; plus `catalog.json`, `references.json`, `decisions.md`, `rolling_glossary.md` when present).
A new row classifies `_gm-materials/session-summaries/` and `_gm-materials/digests/` as host/engine-written, GM read-only, gitignored.
No new writable artefact exists in this design.

### 3.5 Engine Skew and Provenance

`catalog.json` is generated by the host's Jekyll plugin; `references.json` is compiled by the oude-mechanismen gem's `generate_digests.py` run inside the host; hosts run engine versions 0.9.1 (`hostile`) and 0.10.0 (`charged`).
The spec pins a consumed-field subset for each index: catalog consumes `path`, `title`, `summary`, `sha`, `tier`; references consumes the top-level `sources` wrapper (with `total_sources` and `generated` used for a cheap count cross-check) and per-source `slug`, `title`, `category`, `digest_path`, `digest_exists`, `sections[].heading`.
Unknown fields are ignored silently (hostile rows carry extra fields such as `entity_type` and `pages` that must not trip warnings); no upstream `schema_version` is requested.
Row-level degradation applies with a printed warning count on missing fields across engine 0.9.1 and 0.10.0.

### 3.6 Input-Presence Degradation Matrix

Every missing, malformed, or skipped input prints exactly one warning naming the file; play never aborts over host indexes.

| Input | Absent or malformed behaviour |
| :--- | :--- |
| `.agents/AGENTS.md` | Skip the Tone subsection; print one warning. |
| `catalog.json` (absent) | Fall back to spec 001 §2.5 CUJ 5: search `campaigns/`, `directory/`, `systems/` directly, as spec 001 already defines. |
| `catalog.json` (malformed JSON) | Skip catalog loading entirely; print one warning; use the direct-search fallback above. |
| `references.json` (absent or malformed) | Skip the Tier 3 Index subsection; print one warning; never abort play over canon tier 3. |
| `decisions.md` | Skip the Rulings subsection; print one warning. |
| `_gm-materials/session-summaries/` | Use the `log.md` last-3 fallback from §3.2; print one warning if `log.md` is also empty. |
| `digest_exists: false` | Flag `(no digest)` on the slug line; feeds the printed gaps register. |
| Non-Git standalone host | Same as absent indexes, plus the standalone warning from spec 001 §2.5 CUJ 5; no block subsection is emitted for a missing input. |

### 3.7 Scale Rule and GM Startup Ceiling

Each component has a pinned character or line cap with a printed warning and deterministic truncation (a zero-Python kit cannot measure tokens): Tone 12,000 chars; Rulings 4,000 chars; Tier 3 Index 150 lines; Continuity last 3 groups at 4,000 chars per group.
The GM startup ceiling is the sum of these caps (about 34,000 chars, roughly 8–9k tokens measured on `charged`), stated against the spec 001 §3.2 per-persona ceilings, with the promoter's last-3 `log.md` window as the precedent for the continuity window.

### 3.8 Security: Untrusted-Data Coverage

The data-never-instructions clause names every new input explicitly.
Catalog and reference index lines render as flat bullets without Markdown heading sigils.
The Tone and Rulings subsections render wholesale inside the fenced data block under the same banner, with fence lines and heading sigils stripped per the §3.2 fence-escape rule, so injected content cannot break the block or forge structure; line anchors and digest bytes are never rendered at startup.

---

## 4. Alternatives Considered

### 4.1 Content Snapshot Plus SHA Drift Check at Each Play Start

* **Mechanism:** `lingua-init` writes a curated `definition.md` snapshot; `lingua-play` compares pinned SHAs against the live catalog at every start and regenerates on mismatch.
* **Pros:** staleness becomes detectable; one curated artefact to review.
* **Cons:** keeps the full fixed per-session token tax; needs a size cap, a new spec 001 §3.3 ownership row, a regeneration write path with crash semantics, a backfill path for existing workspaces such as `nieuwe-mechanismen`, and relies on 32-bit SHA prefixes as a drift signal.
* **Reason for rejection:** rejected by author after round-1 review — it fixes staleness but not the tax, and adds operational burden the live-read design avoids by construction.

### 4.2 Wholesale All-Tiers `definition.md` Loaded Every Session

* **Mechanism:** generate one file with tone, canon tier 1 summaries, canon tier 3 excerpts, and gaps; load it wholesale at every session start.
* **Pros:** maximum context density in one place.
* **Cons:** roughly 40k-token fixed tax per session, silent staleness against a human-edited vault, lost-in-the-middle attention dilution, enlarged untrusted-data surface, unbounded gaps growth inside a reloaded file.
* **Reason for rejection:** rejected in round 1 by the Goldfish review; the measured untrimmed literal cost of the live read (roughly 27–32k on `charged`) already showed the wholesale variant is worse.

### 4.3 Full Catalog Index Lines at Startup vs Grep-On-Demand

* **Mechanism A:** render all 114 `[cat-T1] path: title — summary` lines at startup (roughly 5–8k tokens per session on `charged`).
* **Mechanism B:** load `catalog.json` as an index and grep on demand; render a line only when the GM explicitly loads an entry.
* **Reason for choosing B:** rejected by author — issue #6's flatness came from missing tone, rulings, and continuity, not from a missing index; B costs near-zero fixed tokens. An optional active-scene pre-mark was considered and dropped as YAGNI (matching heuristic undefined and able to misfire).

### 4.4 References Headings at Startup vs Lazy Per Source

* **Mechanism A:** render all section headings at startup (1,030 headings on `charged`, roughly 10.7k tokens; level-2-only still roughly 6–8k).
* **Mechanism B:** render `- <slug> (<category>)` lines only (roughly 1.5k tokens) and consult headings plus digest sections lazily per source when a scene needs them.
* **Reason for choosing B:** rejected by author — headings are a discovery aid, not scene context; B keeps the discovery path at one lazy read per source.

### 4.5 Continuity Window Last-3 Groups vs Last-1 vs None

* **Reason for choosing last-3 groups (character-capped):** mirrors the promoter's existing last-3 `log.md` window; continuity was part of issue #6. Last-1 group is the pinned fallback if the budget tightens; none is rejected because continuity context is part of the motivation.

### 4.6 Tier-2-Specific Catalog Rendering Rule vs Generic Any-Tier-Labelled Rule

* **Reason for choosing the generic rule:** both live hosts are 100% `tier == 1` in `catalog.json` (114/114 on `charged`, 72/72 on `hostile`), so a tier-2-specific rule is untestable against live data; the generic `[cat-T*]`-labelled rule covers any tier.

### 4.7 `decisions.md` Wholesale vs Capped Pointer Lines

* **Reason for choosing wholesale with a stop at the first /archived/i heading:** the file is small (roughly 0.9k tokens on `charged`) and rulings change play from turn 1; both hosts use an `## Archived or Formalised Rulings` section, so the stop works on both. Inline `[Retracted …]` spans on `charged` remain visible with their marks; that is stated, not hidden. Capped newest-first pointer lines are the pinned fallback if the file grows, with the caveat that the startup view is then not the complete rulings set.

### 4.8 Pointer-Only `definition.md` with SHAs

* **Reason for rejection:** subsumed — the chosen design already loads pointers live, so a separate pointer file adds a second artefact without value.

### 4.9 Hard Size Cap with Deterministic Eviction on a Snapshot

* **Reason for rejection:** rejected in round 1 — an eviction policy in a zero-Python, prose-only kit is the fuzzy curation the EGM review flagged as drift-prone; caps belong on the live-rendered components instead (§3.7).

### 4.10 GM-Craft-Only Fix With No New Context

* **Reason for deferral, not rejection:** deferred to https://github.com/terra-campaigns/lingua-ludi/issues/7; context loading and scene craft are separable features.

---

## 5. Detailed Implementation Plan

The implementation plan lives in [`docs/plans/002_live_setting_index_context_plan.md`](../plans/002_live_setting_index_context_plan.md), per the spec 001 §9 gate convention that plans reside in `docs/plans/NNN_*.md`.

## 6. Risk & Failure-Mode Table

| Risk | Trigger | Mitigation |
| :--- | :--- | :--- |
| Catalog cache staleness | Tier 1 notes edited without a Jekyll rebuild are invisible to grep-on-demand at play start | Spec 001 §2.5 direct-search fallback; promoter SHA citations catch drift at promotion time; risk accepted and documented. |
| Malformed host index JSON | `catalog.json` or `references.json` fails to parse | §3.6 malformed rows: skip the subsection, one warning, never abort play. |
| Injection via wholesale Tone/Rulings | AGENTS.md or decisions.md contains instruction-like prose with fences or heading sigils | §3.2 fence-escape rule (strip fences and `#` sigils); §9 injection test includes a fence-breaker case. |
| Continuity window mis-sorted | Inconsistent session-summary filenames (e.g. the `ep_`-named outlier on `charged`) | Group by leading `YYYY-MM-DD` prefix, sort lexicographically, fall back to modification time for unprefixed names. |
| Retracted rulings loaded as active | Inline `[Retracted …]` spans above the Archived heading | Stop at /archived/i; retracted spans remain visible with their marks (stated, §4.7). |
| Untestable contract rows on live data | Zero tier-2 catalog records; `digest_exists: false` never occurs | §9 synthetic fixtures cover both; noted as accepted. |

## 7. Invariants (Deltas to Spec 001 §5)

* The `## Live Setting Index` block is prompt-only: the load writes zero files, and no `state.md` section is added for it.
* The pinned 5-step startup contract is unchanged; the new pre-flight load runs before it and never blocks the step-3 coach drain.
* Live-read only: no snapshot, no drift check, no regeneration, no backfill path.
* OOC remains player-owned; in-play gaps go to the `Scene Notes` bullet in `state.md` or the player's OOC pause, with `> [OOC]` stripping unchanged.
* All host inputs are read-only and named in the data-never-instructions clause.

## 8. Open Questions

None remaining; the window sort key, archived marker, pre-mark, ceiling, and `references.json` versioning questions were resolved during the round-1 review, and the re-audit residuals during the re-audit (see `docs/reviews/002_live_setting_index_context_egm/goldfish_review_report.md` and `docs/reviews/002_live_setting_index_context_egm/consolidated_review_report_reaudit.md`).

## 9. Verification Plan

* **Dry-run matrix:** run `lingua-play` startup on `charged` / `nieuwe-mechanismen` (114 catalog entries, 100 reference sources, engine 0.10.0) and on `hostile` (72 entries, 54 sources, engine 0.9.1); assert the block contains tone, rulings, slug index, and the continuity window, and that digest bodies are absent until a scene needs them.
* **Budget:** measure harness-reported token usage at startup against the pinned character caps; print warnings on truncation.
* **No-write check:** snapshot `git status` and the workspace directory before and after the load; assert zero modifications.
* **Degradation tests:** synthetic fixtures for each §3.6 row, including malformed `catalog.json` and `references.json`, zero tier-2 catalog records, `digest_exists: false`, and a non-Git standalone directory.
* **Injection test:** embed a synthetic instruction in a reference heading, a catalog summary, an AGENTS.md paragraph, a decisions.md ruling, and an AGENTS.md paragraph containing a ``` fence line; assert all render as data under the banner and the enclosing block stays intact.
* **Regression:** the pinned 5-step startup, coach drain, retrieval-targets write, and close protocol behave exactly as in spec 001.

## 10. Review & Approval Gate

This specification passed one consolidated zero-context Goldfish EGM review and the single allowed re-audit; summary records: [consolidated_review_report.md](../reviews/002_live_setting_index_context_egm/consolidated_review_report.md) and [consolidated_review_report_reaudit.md](../reviews/002_live_setting_index_context_egm/consolidated_review_report_reaudit.md), resolutions in [goldfish_review_report.md](../reviews/002_live_setting_index_context_egm/goldfish_review_report.md), ideation in [concepts_brief.md](../reviews/002_live_setting_index_context_egm/concepts_brief.md).
Approved (Human Sign-Off: `efsalvarenga`, 2026-11-28), mirroring spec 001 §9; production edits happen only under `docs/plans/002_live_setting_index_context_plan.md`.

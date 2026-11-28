# Live Setting-Index Context at Session Start (round-2 revised hypothesis)

* **Author:** efsalvarenga
* **Date:** 2026-11-28

## Overview

The original hypothesis (a session-0 `definition.md` snapshot in `_gm-materials/lingua-ludi/<campaign>/`, loaded wholesale at every session start) was rejected by the round-1 Goldfish review, and that rejection is settled.
The revised hypothesis: at every session start, `lingua-play` reads the host's live read-only index files into one named prompt-only block, `## Live Setting Index`, and nothing is ever written to any file.
Structurally, the load lives in a new pre-flight section between the current §2 (Harness Capability Check) and the pinned 5-step startup sequence; the pinned step 1 itself is not touched.
The trimmed read list is: `.agents/AGENTS.md` wholesale; `.agents/context/decisions.md` read up to `## Archived`; `.agents/context/references.json` rendered as `slug (category)` lines only, with headings lazy per source (level-2-only if ever rendered at startup); a rolling window pinned at the last-3 finished-session summaries, token-capped; and `catalog.json` consumed as a grep-able index on demand rather than rendered line-by-line at startup, with an optional capped pre-mark of entries matching the active scene.
Measured on `charged`, this fixed tax lands at roughly 5–12k tokens per session.
Terminology: "harness tier" means subagent capability, "canon tier" means vault layers as defined per host AGENTS.md (canon tier 2 = `decisions.md` rulings plus session-summaries/session-preps; canon tier 3 = `references.json` digests), and "chapter" stays reserved for the kit's promoted-chapter meaning, so the window is described as finished-session summaries.
Finished play scenes are canon tier 2 conceptually but are not `catalog.json` entries; they live only in the host's session-summaries directory and enter play through the rolling window.
Retargeting `lingua-promote` from Tier 1 proposals to canon tier 2 finished-session summaries is follow-up issue #9.
Full Tier 1 note bodies and `_gm-materials/digests/` sections remain lazy per scene, as today.
Motivation: https://github.com/terra-campaigns/lingua-ludi/issues/6 (flat three-turn session in `charged` / `nieuwe-mechanismen`); follow-ups are #7 (GM craft), #8 (chargen), and #9 (promote retarget).

## Analysis

The round-2 critique comes first: the earlier claim of an 11–14k fixed load does not survive measurement.
On `charged`, the full literal rendering costs roughly 27–32k tokens per session: `.agents/AGENTS.md` ≈ 2.7k, the 114 catalog summary lines ≈ 5–8k, `references.json` slugs plus all 1,030 section headings ≈ 10.7k, `decisions.md` ≈ 0.9k, and the last-3 scene window ≈ 12.2k.
The 11–14k figure only holds for the trimmed read list stated above, so the trimmed list is the revised hypothesis, and the budget must be pinned in countable units (line/char caps) because a zero-Python, prose-only kit cannot measure tokens.
A second correction: the tier-2 catalog rule targets records no current engine emits — both live hosts are 114/114 and 72/72 tier 1, with zero tier-2 entries and no `rolling_glossary.md` on disk — so it is replaced by a generic "render any tier, labelled" rule.
The verified diagnosis stands: no kit file references `references.json` or `_gm-materials/digests/` at all, so canon tier 3 had no input path into play, and `lingua-play` loaded no setting context beyond `campaign.yml`, `state.md`, and ledger retrieval targets.
The live-read design kills the round-1 staleness and operational objections by construction: because no copy of host data is ever created, there is nothing to drift, every session reads the files as they exist at that moment, and the weak 8-hex catalog SHA prefixes stay confined to the `lingua-promote` boundary where they already live.
Residual risks that remain: the injection surface grows at the front of the GM context, so all new inputs must be named in the data-never-instructions clause and `references.json` headings must render as flat bullets without markdown heading sigils; `references.json` carries no version stamp and `digest_exists: false` behaviour is unspecified; `decisions.md` keeps retracted rulings inline, hence the stop at `## Archived`; the hosts run different engine versions (0.10.0 in `charged`, 0.9.1 in `hostile`) and entry keys differ (`id` exists only in 0.10.0), though today this skew is cosmetic; session-summary filenames are not a stable ordering key across hosts, so the window needs a grouping and sort rule; and there is still no area-to-procedural-table mapping in `references.json`, so "this area has no hazard table" is not derivable from the index and gaps surface only through the existing OOC or Scene Notes channels.
Governance: the spec delta is small — no new file, no snapshot, no regeneration path — but §3.3 ownership rows must extend to all of `.agents/context/` and `_gm-materials/session-summaries/`, a per-skill tier-policy table or §3.6 amendment is required, and any `skills/` edit still requires an EGM-approved `docs/designs/002` with human sign-off.

## Goals vs. Non-Goals

Goals:

* Give the GM tone, confirmed rulings, the canon tier 3 slug index, and a rolling window of the last-3 finished-session summaries at every session start, inside the named prompt-only `## Live Setting Index` block, so setting context and continuity are present from turn 1.
* Keep the fixed per-session cost bounded in countable units (line and character caps, not token estimates), roughly 5–12k tokens measured on `charged`, and let body reads scale with scenes actually played.
* Keep freshness by reading live files; no snapshot, no drift check, no regeneration path, nothing persisted.
* Surface content gaps: a print-only gaps register at `lingua-init`, re-derived at every start, and in-play gaps via the player's OOC pause or `state.md` Scene Notes.
* Preserve setting-agnosticism, language-agnosticism, zero Python, read-only Tier 1 and host indexes, and the gitignored status of GM materials.

Non-goals:

* No `definition.md` snapshot, no wholesale load of digest or note bodies, and no all-lines catalog rendering at startup.
* No GM-initiated OOC subtype; OOC stays player-owned.
* No change to the coach, `ledger.md`, or any Tier 1 vault file.
* No retargeting of `lingua-promote` to stage canon tier 2 finished-session summaries instead of Tier 1 proposals; issue #9 owns that.
* Not a fix for GM craft or character generation; issues #7 and #8 own those.

## Alternatives Weighed

Settled round-1 rejections (not re-litigated):

* **Content snapshot plus SHA drift check at each play start (option D):** rejected — it keeps the fixed per-session token tax and requires a size cap, a new §3.3 ownership row, a regeneration write path with crash semantics, a backfill path for existing workspaces such as `nieuwe-mechanismen`, and carries the 32-bit SHA-collision caveat.
* **Wholesale all-tiers `definition.md` loaded every session (original hypothesis):** rejected — ~40k-token fixed tax, silent staleness against a human-edited vault, lost-in-the-middle attention dilution, enlarged untrusted-data surface, and unbounded `## Open Gaps` growth inside a reload-every-session file.
* **Pointer-only `definition.md` with SHAs:** subsumed — the revised design already loads pointers live, so a separate pointer file adds nothing.
* **Hard size cap with deterministic eviction on a snapshot:** rejected — an eviction policy in a zero-Python, prose-only kit is exactly the fuzzy curation the EGM review flagged as drift-prone.
* **Per-scene digest loading at scene change:** kept as a possible future refinement; currently blocked on the missing area-to-digest mapping in `references.json`.
* **GM-craft-level fix with no new context:** deferred to issue #7, not discarded.

Round-2 alternatives:

* **Full catalog index lines at startup vs grep-on-demand:** grep-on-demand wins — 114 `path: title — summary` lines cost ≈ 5–8k tokens every session to save one grep per scene, and issue #6's flatness came from missing tone, rulings, and continuity, not from a missing index; an optional capped pre-mark of active-scene entries preserves discoverability at near-zero fixed cost.
* **References headings at startup vs lazy per source:** lazy wins — all 1,030 headings cost ≈ 10.7k tokens on `charged`, and even level-2-only costs ≈ 6–8k; slugs plus categories alone cost ≈ 1.5k and keep the discovery path through per-scene lazy reads.
* **Rolling window last-3 vs last-1 vs none:** last-3 token-capped wins, mirroring the promoter's existing last-3 `log.md` window; last-1 (≈ 2.3k with session_notes plus tactical_facts only) is the fallback if the budget tightens, and none is rejected because continuity was part of issue #6.
* **Tier-2-specific catalog rendering rule vs a generic any-tier-labelled rule:** the generic rule wins because zero tier-2 entries exist in either live host, so a tier-2-specific rule is untestable against live data today.
* **`decisions.md` wholesale vs capped newest-first pointer lines:** wholesale with a stop at `## Archived` wins for now because the file is small (≈ 0.9k on `charged`) and rulings change how the GM plays from turn 1; pointer lines are the fallback if the file grows.

## Key Decisions & Staging for Next Steps

An eventual `docs/designs/002` specification must pin down, before any `skills/` edit, consolidated and deduplicated from the round-2 readiness, structural, catalyst, and comprehension reports:

* **Block and placement:** the named prompt-only `## Live Setting Index` block, never persisted to any file and never written into `state.md`; a new pre-flight section in `lingua-play` between current §2 and the pinned 5-step startup, leaving pinned step 1 untouched; the load order relative to ledger recovery (it must not block the step-3 coach drain) and to the step-4 `## Active Retrieval Targets` write; and which personas receive the block (GM-only by default, subagent inputs unchanged).
* **Read list and line formats:** one pinned format and one rendered example per input, with an inline tier label on every catalog line, e.g. `- [T1] directory/PlaceA.md: Place A — A riverside market town`; the null-summary and empty-summary rendering (`- [T1] path: title` with no dangling em-dash); references lines as `- <source-slug> (<category>)` with headings lazy per source, a pinned level filter if headings are ever rendered at startup, and never line anchors or raw digest bytes; `decisions.md` read wholesale stopping at `## Archived`; and the rolling window pinned at the last-3 finished-session summaries, token-capped, grouped by session slug and sorted lexicographically by filename date.
* **Catalog handling:** consumed as a grep-able index on demand, with a pinned decision on whether to pre-mark entries matching the active scene and at what cap.
* **Tier policy:** a per-skill tier-policy table or a §3.6 amendment — play renders any catalog tier labelled, while promote consumes tier 1 only until issue #9.
* **Ownership:** §3.3 rows extended to all of `.agents/context/` (AGENTS.md, catalog.json, references.json, decisions.md, rolling_glossary.md) and to `_gm-materials/session-summaries/` (host-written, GM read-only, gitignored).
* **Terminology:** reserve "harness tier" for subagent capability and "canon tier" for vault layers, with canon tiers defined per host AGENTS.md, and say "finished-session summary" rather than "chapter".
* **Degradation matrix:** one input-presence row per input — absent `.agents/AGENTS.md`; absent or malformed `catalog.json` (the existing CUJ 5 fallback searches `campaigns/`, `directory/`, `systems/` directly, and "directly" must be pinned); absent, malformed, or unknown-schema `references.json` (never abort play over canon tier 3); absent `decisions.md`; absent or empty session-summaries; `digest_exists: false` rendering; zero tier-2 catalog records; and a non-Git standalone host.
* **Engine skew and provenance:** a consumed-field subset for `references.json` and `catalog.json` with row-level degradation across engine 0.9.1 and 0.10.0 (for example `id` exists only in 0.10.0 and `hostile`'s `references.json` carries extra keys), plus correct attribution — `catalog.json` is Jekyll-generated while `references.json` is compiled by the oude-mechanismen gem's `generate_digests.py` run inside the host.
* **Untrusted-data coverage:** the data-never-instructions clause names every new input explicitly, headings render as flat bullets without markdown heading sigils, line anchors and raw digest bytes are never rendered at startup, and the exact banner wording is pinned.
* **Gaps register:** print-only at `lingua-init`, re-derived at every start, zero files persisted; in-play gaps via the player's OOC pause or `state.md` Scene Notes, with `> [OOC]` stripping unchanged so the coach and promoter contracts are unaffected.
* **Scale rule:** line/character caps per component with a printed warning and deterministic truncation, plus a stated GM startup context ceiling against the §3.2 per-persona ceilings.
* **Verification criteria:** a dry-run matrix on `charged` / `nieuwe-mechanismen` (114 catalog entries, 100 sources, engine 0.10.0) and `hostile` (72 entries, 54 sources, engine 0.9.1), showing the block contains AGENTS.md tone, the decisions.md rulings, the slug index, and the window, with digest bodies absent until a scene needs them; harness-reported token usage measured against the pinned budget rather than estimates; degradation tests for each absent or malformed input; and an injection test with a synthetic instruction embedded in a heading.
* **Staging:** this brief feeds EGM review and a `docs/designs/002_<slug>.md` that must reach `Status: Approved` with human sign-off before any production skill, template, or persona edit; issues #7, #8, and #9 remain deferred follow-ups outside this feature.

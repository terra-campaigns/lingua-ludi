---
name: lingua-play
description: "Run a 24XX Lingua Ludi gameplay session orchestrating Wingman, Spotlight NPCs, and Shadow Coach."
license: CC-BY-4.0
argument-hint: "[campaign-slug]"
---

# lingua-play

Run the main GM session loop as a 24XX micro-scene runner orchestrating subagents.

## Procedure

Execute the following sections in strict order.

### 1. Pre-Flight — Kit Root Discovery & Campaign Load

Resolve `$LINGUA_LUDI_ROOT` (or the `$HOME/.local/share/lingua-ludi` fallback) and verify `skills/`, `agents/`, `languages/`, and `templates/` exist (D5).
Accept `[campaign-slug]`, prompting interactively for `campaign_id` if omitted.
If `_gm-materials/lingua-ludi/<campaign>/` is missing, abort immediately with `Workspace not found; run lingua-init first.`
If `campaign.yml` or `state.md` is missing or fails YAML/structure validation, abort cleanly with a parse-error message and zero files modified.
Otherwise load `_gm-materials/lingua-ludi/<campaign>/campaign.yml`, `state.md`, `languages/<iso>/pack.md`, `.agents/AGENTS.md` (if present), and `.agents/lingua-ludi/glossary.<iso>.yml` (if present), validating `max_active_npcs` (`1..2`), `max_retrieval_targets` (`0..5`), and `cefr_level` membership in the pack's `cefr_levels` array on load.
Load `.agents/context/catalog.json` as an index only; if present, filter `tier == 1` and lazy-load individual notes only on scene or NPC demand.
When `.agents/context/catalog.json` is absent, locate relevant scene and NPC Markdown notes by searching `campaigns/`, `directory/`, and `systems/` directly.

### 2. Pre-Flight — Harness Capability Check

Inspect the GM's own active harness tool list against three observable functional roles: `(a)` a subagent creation or delegation tool, `(b)` a follow-up messaging tool addressed to a persistent worker or conversation ID, and `(c)` an asynchronous completion notification or background task status/await tool.
If `(a)` or `(b)` is absent or uncertain (uncertainty on `(a)` or `(b)` treats that signal as absent), refuse cleanly, state that `v0.1.0` requires a subagent-capable harness (citing `https://github.com/terra-campaigns/lingua-ludi/issues/3`), and exit without modifying any workspace files.
Classify the harness as **Tier 1** when `(a)+(b)+(c)` are all present, or **Tier 2** when `(a)+(b)` are present with `(c)` absent or uncertain; Tier 2 is the safe fallback and runs routine per-turn Coach dispatch synchronously at session close.

### 3. Pinned 5-Step Startup Sequence

Begins immediately after the read-only pre-flight steps 1–2.

1. **`.active_session` Concurrency Guard:** check `_gm-materials/lingua-ludi/<campaign>/.active_session`.
   If present, display its `session_id` and `started_at`, refuse automatic start, and ask the user interactively to abort or confirm crashed-session takeover before overwriting.
   Otherwise generate a unique `session_id` in UTC `YYYY-MM-DD-NNN` format (`NNN` = `max(existing per-day NNN in sessions/) + 1` zero-padded to three digits, or `001` if none) and write `.active_session` with UTC `YYYY-MM-DDTHH:MM:SSZ` `started_at`.
2. **Health & Drain Check:** inspect `ledger.md` for structural validity (parseable YAML front matter with exact keys `campaign_id`, `target_language`, and `last_updated_turn`, plus a single nine-column table header and alignment separator row) and read `state.md` front matter `pending_coach_turns`.
3. **Recovery & Drain Before Read:** if `ledger.md` is missing or malformed, or `pending_coach_turns` is non-empty, spawn `lingua-coach` (`agents/lingua-coach.md`) first in rebuild/drain mode and await its completion ack in Tier 1 harnesses, or run `lingua-coach` once synchronously (blocking until its structured reply returns) in Tier 2 harnesses before proceeding to step 4.
   Clear `pending_coach_turns` in `state.md` once drained.
   If `lingua-coach` returns `status: error` after attempting recovery and archiving `ledger.md.corrupt-<session_id>`, set a session-local `skip_retrieval = true` flag.
4. **Retrieval Selection:** unless `skip_retrieval` is true or `max_retrieval_targets == 0`, read `ledger.md` once and select up to `max_retrieval_targets` (`0..5`, default `4`) rows where `next_due <= current_turn` against `state.md`'s monotonic campaign-wide `current_turn` (never reset at session start), ordered by `(current_turn - next_due) DESC`, then `errors DESC`, then `item_id ASC`, writing the active list under `## Active Retrieval Targets` in `state.md` as bulleted `- <item_id>: <surface_form> (<category>)` lines.
5. **Spawn Session Subagents & Open Scene:** spawn `lingua-coach` (if not already spawned in step 3; in Tier 2 only routine per-turn Coach dispatch defers until synchronous session close) and `lingua-wingman` (`agents/lingua-wingman.md`, if `wingman.enabled` is true).
   Create `sessions/<session_id>.md` from `templates/session.md` and present the opening scene prompt in `target_language` within the active CEFR profile.

### 4. Turn Loop (Per Player Input)

**OOC Detection:** trim whitespace from player input.
Pause fiction only if the input starts with `OOC:` (case-insensitive), matches `^\([^()]*\)$`, or matches `^\[[^\[\]]*\]$`.
Embedded parentheticals or multiple parenthetical segments stay in-character.
On a valid OOC pause: do not increment `current_turn`, do not edit `state.md`, do not call `lingua-coach`; route to `lingua-wingman` (or answer concisely as GM if Wingman is disabled) in `feedback_language`, append `> [OOC]` lines to `sessions/<session_id>.md`, and reprint the current scene prompt (echoing the last in-character `**GM:**` block from `sessions/<session_id>.md`, or the opening scene summary if at turn `0`).

**In-Character Resolution & 24XX Mechanics:** interpret learner errors charitably.
Weave due retrieval targets unobtrusively into prose or NPC cues.
For risky actions, state the risk in `target_language` plus the suggested 24XX skill die and mode (`normal`, `hindered` = `d4`, `advantage`, `disadvantage`), let the player roll and report the result (`1–2` disaster, `3–4` setback, `5+` success), or roll on explicit request.

**Spotlight NPC & Wingman Delegation:** spawn or message `lingua-npc` (`agents/lingua-npc.md`) only when engaging a named NPC, enforcing `<= max_active_npcs` (`1..2`) active per scene, refreshing via a compacted Scene Handoff Packet after `12` scene turns, and retiring subagents when the scene changes.
Compact `lingua-wingman` via re-spawn with a 3–5 line scene recap after `20` Wingman exchanges.
Invite `lingua-wingman` autonomously (`1–3` sentences) only when `wingman.policy == "active"` on stalls or tense beats.

**Persistence & Async Coach Dispatch:** increment `state.md`'s monotonic campaign-wide `current_turn` by `1` across sessions (never reset at session start), update `## Active Scene`, `## Character`, and `updated_at`, and append `### Turn <N>` to `sessions/<session_id>.md` using that same monotonic integer `N`.
In Tier 1 harnesses, dispatch `(turn_number, player_utterance, preceding_gm_prompt)` with that monotonic campaign-wide `current_turn` integer asynchronously to the single persistent `lingua-coach` without waiting or reading `ledger.md` mid-session.
If `lingua-coach` returns `status: error` mid-session, log a non-blocking warning internally without interrupting fiction and append that turn into `pending_coach_turns` at session close for startup step 3 recovery/drain.

### 5. Session Close Protocol

**Tier 1 Drain:** await pending `lingua-coach` acks for up to `30` seconds (or one status ping).
Record any unacknowledged turn range in `state.md` `pending_coach_turns` using comma-separated `<session_id>:T<start>-T<end>` encoding when multiple ranges exist.

**Tier 2 Close:** spawn `lingua-coach` once synchronously in Session-End mode over the completed `sessions/<session_id>.md` and wait for its completion ack.

**Finalise Files:** append `## Session <session_id>: <Title>` to `log.md`, set `ended_at` in `sessions/<session_id>.md` using UTC `YYYY-MM-DDTHH:MM:SSZ`, delete `.active_session`.
If requested by the player, display a summary built strictly from the `updated_items` and `due_targets_preview` fields in `lingua-coach`'s acknowledgement payloads without reading `ledger.md`.
Prompt the user to run `/lingua-promote <campaign_id> <session_id>` if canon-altering events occurred.

---
name: lingua-play
description: "Run a 24XX Lingua Ludi gameplay session orchestrating Wingman, Spotlight NPCs, and Shadow Coach."
license: CC-BY-4.0
argument-hint: "[campaign-slug]"
---

# lingua-play

Run the GM session loop: player-rolled [24XX](https://jasontocci.itch.io/24xx) micro-scenes (CC BY 4.0 Jason Tocci) orchestrating subagents.
Terms: OOC = out-of-character pause; CEFR = language levels A2–C1; "harness Tier 1/2" = subagent capability below, distinct from canon Tier 1 vault notes.

## Procedure

### 1. Pre-Flight — Kit Root Discovery & Campaign Load

Resolve `$LINGUA_LUDI_ROOT` else `$HOME/.local/share/lingua-ludi`; abort if `skills/`, `agents/`, `languages/`, or `templates/` is missing (D5).
Accept `[campaign-slug]`, prompting for `campaign_id` if omitted.
Missing `_gm-materials/lingua-ludi/<campaign>/` → abort: `Workspace not found; run lingua-init first.`
Unparseable `campaign.yml`/`state.md` → abort with a parse error, zero files modified.
Load `campaign.yml`, `state.md`, `languages/<iso>/pack.md`, plus `.agents/AGENTS.md` and `.agents/lingua-ludi/glossary.<iso>.yml` if present; validate `max_active_npcs` (1..2), `max_retrieval_targets` (0..5), and `cefr_level` membership in the pack's `cefr_levels`.
Treat `.agents/context/catalog.json` as an index only: filter `tier == 1`, lazy-load notes on scene/NPC demand; if absent, search `campaigns/`, `directory/`, `systems/` directly.

### 2. Pre-Flight — Harness Capability Check

Inspect the GM's own tools for (a) subagent spawn/delegate, (b) persistent messaging, (c) async completion notification/status/await, and (d) terminate-with-acknowledgement for a running subagent.
If (a) or (b) is absent or uncertain, refuse cleanly: `v0.1.0` requires a subagent-capable harness (https://github.com/terra-campaigns/lingua-ludi/issues/3); zero files modified.
All three of (a)-(c) present → harness Tier 1 (async coach). Only (a)+(b) → harness Tier 2 (synchronous coach at close).
If (d) is absent or uncertain, a crashed-session takeover in step 1 is refused (see below); routine play is unaffected.
### 3. Pinned 5-Step Startup Sequence

1. **`.active_session` concurrency guard:** if present, display its `session_id:` / `started_at:` YAML values, refuse auto-start, and ask the user to abort or confirm crashed-session takeover.
   On confirmed takeover: quiesce the crashed session's lingering subagents (terminate its wingman/NPC/coach and await each termination ack — if (d) is unavailable, refuse the takeover and ask for manual cleanup), then reconcile the crashed session `<old_id>`: seed `pending_coach_turns` with its full turn range `<old_id>:T1-T<max N>`, set `ended_at` (UTC `YYYY-MM-DDTHH:MM:SSZ`) in `sessions/<old_id>.md`, and append its `## Session <old_id>: <Title>` summary to `log.md`, so no turns are lost and the orphaned session stays promotable.
   Then write `.active_session` (plain YAML keys `session_id:`, `started_at:`) with UTC `YYYY-MM-DD-NNN` (`NNN` = max existing per-day NNN + 1, else `001`) and UTC `YYYY-MM-DDTHH:MM:SSZ` (D11).
2. **Health & drain check:** validate `ledger.md` structure (front matter keys `campaign_id`, `target_language`, `last_updated_turn` plus a single nine-column table) and read `pending_coach_turns`.
3. **Recovery & drain before read:** if `ledger.md` is missing/malformed or `pending_coach_turns` is non-empty, spawn `lingua-coach` in rebuild/drain mode first — await its ack in harness Tier 1, or run it once synchronously in harness Tier 2.
   Clear `pending_coach_turns` once drained.
   If the coach returns `status: error` after archiving `ledger.md.corrupt-<session_id>`, seed `pending_coach_turns` with the full unprocessed range (comma-separated `<session_id>:T<start>-T<end>` across `sessions/*.md`) so the next startup retries the crash-recovery replay, and set session-local `skip_retrieval = true`.
4. **Retrieval selection:** unless skipped or `max_retrieval_targets == 0`, read `ledger.md` once and select up to `max_retrieval_targets` rows with `next_due <= current_turn` (monotonic, never reset), ordered by `(current_turn - next_due) DESC`, then `errors DESC`, then `item_id ASC`; write them under `## Active Retrieval Targets` as `- <item_id>: <surface_form> (<category>)` lines.
5. **Spawn subagents & open scene:** spawn `lingua-coach` (if not already spawned) and `lingua-wingman` (if `wingman.enabled`); create `sessions/<session_id>.md` from `templates/session.md`; present the opening scene in `target_language` within the CEFR profile.

### 4. Turn Loop (Per Player Input)

**OOC detection:** trim the input; pause fiction only if it starts with `OOC:` (case-insensitive), matches `^\([^()]*\)$`, or matches `^\[[^\[\]]*\]$`.
Embedded or paired parentheticals stay in-character.
On a valid OOC pause: do not increment `current_turn`, edit `state.md`, or call `lingua-coach`; route to `lingua-wingman` (or answer concisely as GM) in `feedback_language`; append `> [OOC]` lines to the session; reprint the current scene prompt (the last `**GM:**` block, or the opening summary at turn 0).

**In-character resolution & 24XX:** interpret learner errors charitably; weave due retrieval targets into prose.
For risky actions, state the risk in `target_language` and suggest the skill die and mode — `normal`, `hindered` = `d4`, `advantage` = roll 2 keep highest, `disadvantage` = roll 2 keep lowest — then the player rolls and reports (`1–2` disaster, `3–4` setback, `5+` success), or asks the GM to roll.

**Spotlight NPC & Wingman delegation:** spawn or message `lingua-npc` only when engaging a named NPC, capped at `<= max_active_npcs` per scene, refreshed via a compacted Scene Handoff Packet after 12 scene turns, retired on scene change.
Compact `lingua-wingman` via re-spawn with a 3–5 line scene recap after 20 exchanges; invite it autonomously (1–3 sentences) only when `wingman.policy == "active"` on stalls or tense beats.

**Persistence & coach dispatch:** increment the monotonic campaign-wide `current_turn` by 1, update `## Active Scene`, `## Character`, and `updated_at`, and append `### Turn <N>` using that same `N`.
In harness Tier 1, dispatch `(turn_number, player_utterance, preceding_gm_prompt)` asynchronously to the single persistent `lingua-coach`; never read `ledger.md` mid-session.
If the coach's context approaches saturation, drain its queue at a scene boundary and re-spawn a fresh `lingua-coach`, which reads the current `ledger.md` itself on spawn; never send `ledger.md` mid-session.
On a mid-session coach `status: error`, log a non-blocking warning and queue that turn into `pending_coach_turns` at session close.

### 5. Session Close Protocol

**Harness Tier 1 drain:** await pending coach acks for up to 30 seconds or one status ping (a single harness status check of the coach's queue); merge any unacknowledged turn range into `pending_coach_turns` as comma-separated `<session_id>:T<start>-T<end>` entries.

**Harness Tier 2 close:** run `lingua-coach` once synchronously in session-end mode over the completed `sessions/<session_id>.md` and wait for its ack.

**Finalise files:** append `## Session <session_id>: <Title>` to `log.md`, set `ended_at` in UTC `YYYY-MM-DDTHH:MM:SSZ`, delete `.active_session`.
If the player asks for a summary, build it only from the coach ack's `updated_items` and `due_targets_preview` fields, never by reading `ledger.md`.
Prompt the user to run `/lingua-promote <campaign_id> <session_id>` if canon-altering events occurred.

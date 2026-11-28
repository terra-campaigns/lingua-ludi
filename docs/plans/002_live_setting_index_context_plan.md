# Implementation Plan: Live Setting-Index Context at Session Start

**Document ID:** `docs/plans/002_live_setting_index_context_plan.md`

| Metadata | Value |
| :--- | :--- |
| Author | `efsalvarenga` |
| Date | 2026-11-28 |
| Plan Version | 0.1.2 |
| Status | Approved (Human Sign-Off: `efsalvarenga`, 2026-11-28) |
| Implements | [Spec 002, version 0.1.3](../designs/002_live_setting_index_context.md), approved 2026-11-28 |
| Review Record | [consolidated_review_report.md](../reviews/002_live_setting_index_context_egm/consolidated_review_report.md) and [consolidated_review_report_reaudit.md](../reviews/002_live_setting_index_context_egm/consolidated_review_report_reaudit.md), resolved in [goldfish_review_report.md](../reviews/002_live_setting_index_context_egm/goldfish_review_report.md) |
| Spec | `docs/designs/002_live_setting_index_context.md` |

## 1. Target Files to Create/Modify

### 1.1 `skills/lingua-play/SKILL.md` (Modify)

* **Scope:** the one skill whose startup gains the load.
* **Proposed changes:**
  1. Insert a new `### 3. Pre-Flight — Live Setting-Index Load` section between the current §2 (Harness Capability Check) and the pinned startup sequence; renumber the pinned sequence `### 3.` → `### 4.`, and renumber the existing `### 4. Turn Loop (Per Player Input)` → `### 5.` and `### 5. Session Close Protocol` → `### 6.`, keeping the internal steps 1–5 of the pinned sequence verbatim.
  2. The new section renders the `## Live Setting Index` block per spec 002 §3.2: Tone (`.agents/AGENTS.md` wholesale, 12,000-char cap), Rulings (`.agents/context/decisions.md` up to the first heading matching /archived/i, 4,000-char cap), Tier 3 Index (one `- <source-slug> (<category>)` line per `sources[]` entry, max 150 lines), Continuity (last 3 session groups by leading `YYYY-MM-DD` filename prefix, 4,000 chars per group), inside a fenced data block under one untrusted-data banner naming every input.
  3. Replace the §1 catalog line (the line beginning "Treat `.agents/context/catalog.json` as an index only") with the grep-on-demand rule: catalog is an index for on-demand grep, entries rendered with inline `[cat-T1]`/`[cat-T2]` labels only when explicitly loaded, never rendered wholesale at startup.
  4. Extend the untrusted-data line (line 12) to name `.agents/AGENTS.md`, `decisions.md`, `references.json`, session summaries, and any catalog lines loaded on demand.
  5. Fix the malformed list marker at the start of pinned step 1 (raw line 33 renders without a visible `1.` marker) as a separate, verbatim step text.
  6. Malformed or missing index JSON: skip that block subsection, print one warning naming the file, never abort play.
  7. Add one turn-loop line: on moving to a new area, if no `references.json` source plausibly covers it, record the gap in the `Scene Notes` bullet under `## Active Scene` in `state.md` and continue; the player may also raise it via `OOC:`.
* **Why:** this is the only behaviour change; everything else is contract documentation.

### 1.2 `skills/lingua-init/SKILL.md` (Modify)

* **Scope:** print-only gaps register.
* **Proposed changes:** extend the post-init check (current step 7) to print a count plus lines for `digest_exists: false` sources and reference categories with no digest; zero files written, nothing persisted.
* **Why:** gives the user the escalation channel decided in the brainstorm without a new artefact or ownership row.

### 1.3 `docs/designs/001_lingua_ludi_core_architecture.md` (Modify, targeted amendments)

* **Scope:** the approved core spec.
* **Proposed changes:** extend the §3.3 write-ownership table to all of `.agents/context/` and to `_gm-materials/session-summaries/` and `_gm-materials/digests/` (host-written, kit read-only); amend §3.6 with the per-skill tier-policy table (play renders any catalog tier with a `[cat-T*]` label; promote consumes `tier == 1` only until issue #9) and the grep-on-demand catalog rule; update the §5.2 risk row about blanket `tier == 1` filtering to match; extend the §1.5 terminology with harness tier vs canon tier; add a changelog row pointing at spec 002.
* **Why:** the new read set and tier policy must be contractual, not implied; amendments go through the EGM gate like any spec change.

### 1.4 `docs/designs/002_live_setting_index_context.md` (Exists as Draft v0.1.2; update on approval)

* **Scope:** the specification itself.
* **Why:** EGM requires an approved spec before production skill edits.

### 1.5 `README.md` (No change)

* **Scope:** user-facing docs.
* **Outcome recorded during review:** README does not enumerate the `lingua-play` startup sequence, so no README edit is required.

## 2. Files Explicitly Not Changed

* `agents/lingua-wingman.md`, `agents/lingua-npc.md`, `agents/lingua-coach.md`, `agents/lingua-promoter.md` — subagent input contracts are unchanged (GM-only block).
* `templates/` — no new template; nothing is persisted, so `state.md`, `log.md`, `ledger.md`, `campaign.yml`, `session.md`, `glossary.yml`, `tier1_proposal.md` are untouched.
* `skills/lingua-promote/SKILL.md` — retargeting to canon tier 2 is issue #9.
* `languages/` — no pack change in this feature.

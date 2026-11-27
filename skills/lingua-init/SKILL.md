---
name: lingua-init
description: "Initialise a Lingua Ludi campaign workspace and host setting glossary inside a campaign repository."
license: CC-BY-4.0
argument-hint: "[campaign-slug] [target-iso] [cefr]"
---

# lingua-init

Run inside a host campaign repository to scaffold a gitignored Lingua Ludi workspace.

## Procedure

Execute the following steps in strict order.

### 1. Kit Root Discovery

Resolve `$LINGUA_LUDI_ROOT` if set, otherwise `$HOME/.local/share/lingua-ludi`.
Verify readable `skills/`, `agents/`, `languages/`, and `templates/` subdirectories exist under the resolved root.
Abort with an explicit missing-path error if any check fails.

### 2. Intake & Parameters

Read positional arguments `campaign_id` (`[campaign-slug]`), `target_language` (`[target-iso]`), and `cefr_level` (`[cefr]`, a CEFR level `A2`–`C1` per the [Common European Framework of Reference](https://www.coe.int/en/web/common-european-framework-reference-languages)), prompting interactively if omitted.
Prompt interactively for `feedback_language`, `wingman.policy` (`on_request` or `active`, defaulting to `on_request` if left blank and validating against those two values), `character_name`, `character_concept`, and `start_location`.
Apply defaults `wingman.enabled: true`, `wingman.name: "Companion A"`, `wingman.role: "Local fixer and colleague"`, `max_active_npcs: 2`, and `max_retrieval_targets: 4`.
Validate `campaign_id` against `^[a-z0-9_-]+$`.
If invalid, abort immediately printing the allowed slug pattern with zero files written.

### 3. Language Pack Verification

Check that `$LINGUA_LUDI_ROOT/languages/<iso>/pack.md` exists and verify the requested `cefr_level` is a member of its YAML front-matter `cefr_levels` array.
If the pack file is missing, abort immediately with zero files created and list the available `<iso>` directories under `$LINGUA_LUDI_ROOT/languages/`.
If `cefr_level` is absent from `cefr_levels`, abort immediately printing the pack's declared `cefr_levels` with zero files written.

### 4. Git & `.gitignore` Safety Gate

Run `git rev-parse --is-inside-work-tree`.
If inside a Git repository, run `git check-ignore -q _gm-materials/`.
If the exit code is non-zero, abort immediately with zero files written, print `_gm-materials/` with manual `.gitignore` instructions, and never edit `.gitignore` automatically.
If not inside a Git repository, print an explicit warning that Git ignore protection is inactive and continue.

### 5. Canonical Language Resolution

Read `.agents/AGENTS.md` if present and evaluate categories `(1)` → `(2)` → `(3)` sequentially using case-insensitive matching.
Category `(1)`: an explicit `canonical_language:` key or `canonical language` heading.
Category `(2)`: a writing-style language rule or heading matching `British English`, `UK English`, `American English`, `US English`, or `write in` / `use <Language>`.
Category `(3)`: a chapter or session output language heading or line.
Map recognised prose names (`British English` or `UK English` → `en-GB`, `American English` or `US English` → `en-US`, `English` → `en`, `German` → `de`, `French` → `fr`, `Spanish` → `es`, `Portuguese` → `pt`, `Italian` → `it`), or preserve an explicit BCP-47 tag matching `^[a-z]{2}(-[A-Z]{2})?$` case-sensitively — the passthrough resolves tags such as `nl` directly, since this kit forbids naming concrete target languages in `skills/`.
Advance when a category yields zero recognised tags and stop at the first category yielding one or more recognised tags.
A single consistent tag wins; two or more distinct tags inside that category immediately mark `.agents/AGENTS.md` ambiguous.
If `.agents/AGENTS.md` is absent, silent across all three categories, or ambiguous within the first non-empty category, prompt interactively here for the fallback `canonical_language` and record the resolved tag in `campaign.yml`.

### 6. Scaffold Workspace & Glossary

If `_gm-materials/lingua-ludi/<campaign_id>/` already exists, abort immediately with an existing-workspace message and overwrite zero files.
Otherwise create `_gm-materials/lingua-ludi/<campaign_id>/` along with `sessions/` and `exports/` subdirectories.
Instantiate `campaign.yml`, `state.md`, `log.md`, and `ledger.md` from `templates/`, resolving all `{{...}}` tokens cleanly so zero unresolved tokens remain.
Create `.agents/lingua-ludi/` parent directories if missing and create `.agents/lingua-ludi/glossary.<iso>.yml` from `templates/glossary.yml` only if it does not already exist, never overwriting an existing glossary.

### 7. Post-Init Check

Confirm zero Tier 1 files (`campaigns/`, `directory/`, `systems/`) or `.gitignore` were touched.
Print the workspace path and the next command (`/lingua-play <campaign_id>`).

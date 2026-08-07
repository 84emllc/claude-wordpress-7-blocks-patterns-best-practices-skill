# AGENTS.md

Instructions for agents and human contributors working in this repository.

## What this repo is

A single agent skill (`SKILL.md`) covering the blocks-vs-patterns-vs-template-parts decision on WordPress 7.0+ block themes, backed by an empirical test of pattern-reference flattening.

## Hard rules

1. **Never let a client name, project name, or business-specific implementation detail into this file.** This skill was extracted and genericized from project-specific rule files that named a real client, their theme's custom namespace, and their internal CLI commands. Any future example must use a generic or invented illustration (a made-up block name, a generic slug), never a real client's project internals.

2. **Keep the "Empirical test" section's claim falsifiable and current.** It asserts a specific, testable fact about WP 7.0 core behavior (pattern references flatten on first editor save). If a future core version changes this, update the section rather than leaving a stale claim, and note the version tested against.

3. **The four-mechanism table (Template Part / Synced Pattern / Pattern / custom block) and the Decision Model table must stay consistent with each other.** They describe the same taxonomy from two angles (official definitions vs. practical decision-making); a change to one that isn't reflected in the other creates a contradiction a reader won't notice.

4. **The core-first row stays first in the Decision Model, and Step 0 stays ahead of the model.** The table is first-match-wins, and the ordering is the point: an earlier version routed "hero image, card, FAQ, CTA" straight to a custom block with no core-first check, and a real theme reimplemented blocks core already shipped as a result. Any future edit that reorders the table, or that adds a row above the core-first one, is reintroducing that bug.

5. **Do not let the `role: "content"` guidance drift back into a general answer for editable text.** It exists for attributes a custom block legitimately owns. Inner blocks are preferred wherever they will do the job, because copy in an attribute is copy the operator edits in the Inspector.

## Conventions

- Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`). Semantic Versioning. Keep a Changelog.
- GitHub Actions pinned to commit SHAs, not moving tags.
- Do not commit generated junk.

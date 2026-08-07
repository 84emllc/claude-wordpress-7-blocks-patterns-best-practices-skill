# Changelog

All notable changes to this project are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- "Step 0: does core already ship this block?", with the `ls wp-includes/blocks/` check, the signals that an agent skipped it, a worked example (an FAQ panel that is `core/accordion`) and a counter-example (a carousel, which core does not ship).
- "Registering a block variation": the `get_block_type_variations` PHP filter with a worked example, the JS `registerBlockVariation` alternative, the server-definition-wins behaviour, the `innerBlocks` triplet shape, and when `isActive` is and is not needed.
- "Content on the canvas, not in the Inspector": the prohibition on copy, HTML and JSON in Inspector fields and on `ServerSideRender` standing in for editable content, plus inner blocks and custom inline formats as the two mechanisms that remove the temptation.
- "Use core's own attributes first": prefer a core block attribute over CSS, why core sometimes defends its own attribute hooks with `!important`, and why equal columns carry no `width` attribute at all.
- "Generating valid core markup for seeded content": build the tree with `createBlock`, save through the editor, export with WP-CLI, rather than hand-writing core's save markup.
- A block-attribute layer in the styling cascade, for per-instance values that no theme.json layer should own.
- Sources for Block Variations and Rich Text formats.

### Changed

- **The decision model now leads with a core-first row, and is explicitly ordered first-match-wins.** The previous table routed "hero image, card, FAQ, CTA" to a custom block with no core-first check, which is how a theme ends up reimplementing blocks core already ships.
- TL;DR rewritten and expanded from four items to seven, leading with the core-first check.
- "Rules for future agents" expanded from six to nine, leading with reading `wp-includes/blocks/`.
- `role: "content"` is now framed as damage control for attributes a custom block legitimately owns, not as the general answer for editable text; inner blocks are preferred where they will do.
- The four-mechanism section notes block variations as a fifth mechanism alongside them.
- Skill description updated to cover the core-first check and variation registration.

## [0.1.0] - 2026-07-11

### Added

- Initial `wordpress-7-blocks-patterns-best-practices` agent skill (`SKILL.md`), genericized from a client project's internal rule file: the official blocks/patterns/template-parts reuse model, WordPress 7.0 content-only editing changes, an empirical test of pattern-reference flattening, a section-mechanism decision table, and a styling-cascade decision model.

[Unreleased]: https://github.com/84emllc/claude-wordpress-7-blocks-patterns-best-practices-skill/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/84emllc/claude-wordpress-7-blocks-patterns-best-practices-skill/releases/tag/v0.1.0

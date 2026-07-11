---
name: wordpress-7-blocks-patterns-best-practices
description: Use when deciding whether a WordPress block-theme section should be a custom block, a pattern, a synced pattern, or a template part -- especially on WordPress 7.0+, where content-only editing changed how patterns and template parts behave. Covers the official reuse-mechanism model, the WP 7.0 content-only editing changes, an empirical test of pattern-reference flattening, and a styling-cascade decision model (theme.json vs block style variation vs class vs inline).
---

# WordPress 7.0 blocks vs patterns: best practices

**Status:** authoritative reference. Read before deciding whether a section should be a block, a pattern, a synced pattern, or a template part.
**Established:** 2026-05-28, from official WordPress sources plus an empirical test on a WordPress 7.0 install.
**Targets:** PHP 8.4+, WordPress 7.0, theme.json v3.

## TL;DR

1. **Editor-managed content must live as real blocks in the page, not as bare `wp:pattern` references.** A `wp:pattern` reference flattens into inline blocks the moment the page is saved in the editor -- verified on WP 7.0 core (see "Empirical test" below). Content-only editing does NOT keep the reference live.
2. **Mark content attributes `"role": "content"`** on every custom block so they stay editable in WP 7.0 content-only mode.
3. Per-section mechanism: a functional/managed component (hero, card, FAQ, CTA) -> custom block; a plain layout of core blocks whose content varies per page -> expanded core blocks in the page; content identical everywhere -> synced pattern; site structure (header/footer/nav) -> template part. Cross-cutting visual style -> theme.json / CSS.
4. **Never frame a bespoke section by putting a class or inline styles on a core block.** It is not rebuild-proof: when someone deletes the block or the whole section and rebuilds it, the class/inline style is gone and the formatting silently reverts. Specially-formatted, managed-media, or behavioral content is a **custom block** (which owns its own render + stylesheet); use a block **variation** for "same block, preset attributes," and a block **style variation** only for a pure CSS-class tweak after other options are exhausted.

## Official best practices (with sources)

### Patterns are insertion starting points
>
> "Block Patterns are predefined block layouts available from the patterns tab of the block inserter. Once inserted into content, the blocks are ready for additional or modified content and configuration." -- Block Editor Handbook, [Patterns](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-patterns/)

The lifecycle is: insert -> the pattern's blocks are **copied into the page** -> edit them there. The pattern is a template/starting point, not a live link. The Theme Handbook recommends the file-based `/patterns` folder for registration ([Patterns -- Theme Handbook](https://developer.wordpress.org/themes/patterns/)).

### The four reuse mechanisms have distinct jobs

From [Comparing Patterns, Template Parts, and Synced Patterns](https://wordpress.org/documentation/article/comparing-patterns-template-parts-and-reusable-blocks/):

| Mechanism | Type | Syncing | Example |
|---|---|---|---|
| Template Part | Site structure | Synced | Header, Footer |
| Synced Pattern (reusable block) | User content | Synced (one source, updates everywhere) | Business hours |
| Pattern (unsynced) | User content | Un-synced (per-instance copies) | Call to action |

> "patterns are best for content that you expect to change across your site... When it's less about content and more about wanting to repeat a design or layout, patterns are best."

Custom **blocks** are for functionality / managed components (custom rendering, structured data, media handling, interactivity) -- not for static arrangements of core blocks (those are patterns).

### WordPress 7.0 changes

From [Pattern Editing in WordPress 7.0](https://make.wordpress.org/core/2026/03/15/pattern-editing-in-wordpress-7-0/) and the [Call for Testing](https://make.wordpress.org/test/2026/02/27/call-for-testing-pattern-editing-and-content-only-interactivity-in-wordpress-7-0/):

- **Content-only editing is the default for unsynced patterns and template parts:** "Unsynced patterns and template parts inserted into the editor now default to content-only mode, prioritizing the editing of text and media without exposing the deeper block structure or style controls." Editors change text/media; structure stays locked until they click "Edit pattern".
- **Custom blocks need `"role": "content"`** on any attribute that should be editable in content-only mode. Example: `message: { type: 'string', default: '...', role: 'content' }`.
- **Pattern Overrides now work for any block** (including custom blocks) via Block Bindings -- no longer limited to a hardcoded set of core blocks ([Pattern Overrides in WP 7.0](https://make.wordpress.org/core/2026/03/16/pattern-overrides-in-wp-7-0-support-for-custom-blocks/)). Overrides are a SYNCED-pattern feature: the structure stays synced from one source while each instance overrides marked content, stored as metadata on the instance.

## Empirical test (WP 7.0 core, 2026-05-28)

The one thing the official posts do not state is whether an unsynced, file-based `wp:pattern` reference survives content-only editing without flattening. Tested directly on a WP 7.0 install:

1. Created a page whose entire content was a single `wp:pattern` reference block pointing at a registered pattern (47 chars).
2. Opened it in the 7.0 block editor (it loaded with the pattern's top-level block already expanded, i.e. `core/group`), and saved via `wp.data.dispatch('core/editor').savePost()`.
3. Re-read `post_content`: **over 2,000 chars of expanded inline blocks, zero `wp:pattern` references.** The top block was a `core/group` carrying `metadata.patternName` for provenance, but every child block was fully expanded inline.

**Conclusion:** a `wp:pattern` reference flattens to inline blocks on the first editor save, even in WP 7.0. The reference is not a stable, editable representation. (7.0 does stamp `metadata.patternName` on the expanded group as provenance, but the content is still inlined.) Therefore: do not rely on bare pattern references for content a human will edit.

## Decision model -- which mechanism for a section

| Section kind | Mechanism | Why |
|---|---|---|
| Functional/managed component (hero image, card, FAQ, CTA) | **Custom block** with content attrs marked `role:"content"`; media via an `imageId` attribute | Block instance persists in content; content editable; structure owned by the block and deploys with the theme |
| Plain layout of core blocks, content varies per page | **Expanded core blocks in the page content** (core/image, core/cover for media); the pattern stays as an inserter starting point only | Core blocks are inherently editable; content-only editing protects structure |
| Identical content reused site-wide (e.g. legal footnote) | **Synced pattern**, optionally with pattern overrides | One source updates everywhere |
| Site structure (header/footer/nav) | **Template part** (+ core/navigation for the menu) | Carries semantic meaning in block themes |
| Cross-cutting visual style (color, spacing) | **theme.json / CSS** | Applies regardless of how content is composed |

## Rules for future agents

1. **Page content is real blocks, not bare `wp:pattern` references**, for anything a human will edit. References flatten on first save (tested above) and are not editor-stable.
2. **Mark `"role": "content"`** on custom-block attributes that hold editable text or media (titles, descriptions, button text/URLs, image IDs/alt). Do NOT mark pure settings (toggles, speeds, overlap px, layout switches).
3. **Images that should be editor-swappable** go through the media library: store an `imageId` attribute on the block and render with `wp_get_attachment_image()` (a theme-file fallback is fine until one is chosen). If content is provisioned programmatically (a seed script, an import), import assets into the media library rather than hardcoding a theme-file URL, so the result stays editor-swappable.
4. **A hardcoded theme-file image URL inside a pattern is a smell** -- the content editor cannot manage it. Make it a block attribute or a media-library attachment instead.
5. **Patterns remain useful** as inserter starting points and as the dev-time composition source, but the page's saved content is the editable source of truth.
6. **Don't reach for a pattern when the thing carries per-page editor-set media or copy** -- that is a block (or expanded core blocks). Reserve patterns for fixed structural layout reused as-is.

## Styling cascade: where a style belongs

Block-theme styling is a defined cascade, most-global to most-specific (Theme Handbook / Block Editor Handbook). Put each value at the RIGHT layer; the layer is decided by SCOPE, not preference.

1. **theme.json root `styles`** -- global defaults that "trickle down through the design and are used unless a more specific element or block style overrides them."
2. **theme.json `styles.elements.<element>`** (e.g. `button` -> `.wp-element-button`) -- the site-wide default for that element type.
3. **theme.json `styles.blocks.<block>`** -- default for a block type.
4. **Block style variations** (`.is-style-{name}`, registered via `register_block_style()` or a `/styles` partial) -- named, selectable variants. The Theme Handbook RECOMMENDS these over custom CSS "because they appear in the Appearance > Editor > Styles interface, and your theme users can make their own changes."
5. **Custom CSS class / stylesheet** -- only for what theme.json / variations cannot express.
6. **Inline `style=` literal values** -- avoid where a design-token layer above could express the same value. A preset-referencing inline value (e.g. `padding:var(--wp--preset--spacing--lg)`) is fine; a literal value (e.g. `border-radius:20px`) usually belongs at a layer above instead.

**Decision rule:** is the value the default for EVERY instance of an element/block? -> global layer (theme.json elements/blocks, set once). Is it a VARIANT (only some instances)? -> a block style variation (preferred, editor-visible/user-editable) or a custom class. Is it already the global default? -> don't restate it.

WordPress 7.0 also supports pseudo-class selectors (`:hover`, `:focus`, `:active`) directly in theme.json on blocks and their variations, so interactive states can avoid custom CSS too.

## Sources

- Block Editor Handbook -- Patterns: <https://developer.wordpress.org/block-editor/reference-guides/block-api/block-patterns/>
- Theme Handbook -- Patterns: <https://developer.wordpress.org/themes/patterns/>
- Comparing Patterns, Template Parts, and Synced Patterns: <https://wordpress.org/documentation/article/comparing-patterns-template-parts-and-reusable-blocks/>
- Pattern Editing in WordPress 7.0 (Make/Core): <https://make.wordpress.org/core/2026/03/15/pattern-editing-in-wordpress-7-0/>
- Pattern Overrides in WP 7.0: Support for Custom Blocks (Make/Core): <https://make.wordpress.org/core/2026/03/16/pattern-overrides-in-wp-7-0-support-for-custom-blocks/>
- Call for Testing: Pattern editing and content-only interactivity in WP 7.0 (Make/Test): <https://make.wordpress.org/test/2026/02/27/call-for-testing-pattern-editing-and-content-only-interactivity-in-wordpress-7-0/>
- An introduction to overrides in Synced Patterns (Developer Blog): <https://developer.wordpress.org/news/2024/06/an-introduction-to-overrides-in-synced-patterns/>
- Applying Styles (Theme Handbook): <https://developer.wordpress.org/themes/global-settings-and-styles/styles/applying-styles/>
- Block Style Variations (Theme Handbook): <https://developer.wordpress.org/themes/features/block-style-variations/>
- Global Settings & Styles / theme.json (Block Editor Handbook): <https://developer.wordpress.org/block-editor/how-to-guides/themes/global-settings-and-styles/>

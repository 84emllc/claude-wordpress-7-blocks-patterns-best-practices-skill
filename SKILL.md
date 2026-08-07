---
name: wordpress-7-blocks-patterns-best-practices
description: Use when deciding whether a WordPress block-theme section should be a core block, a block variation, a custom block, a pattern, a synced pattern, or a template part -- especially on WordPress 7.0+, where content-only editing changed how patterns and template parts behave. Covers the core-first check, block variation registration, the official reuse-mechanism model, the WP 7.0 content-only editing changes, an empirical test of pattern-reference flattening, and a styling-cascade decision model (block attributes vs theme.json vs block style variation vs class vs inline).
---

# WordPress 7.0 blocks vs patterns: best practices

**Status:** authoritative reference. Read before deciding whether a section should be a core block, a block variation, a custom block, a pattern, a synced pattern, or a template part.
**Established:** 2026-05-28, from official WordPress sources plus an empirical test on a WordPress 7.0 install.
**Revised:** 2026-08-07, after a block theme was audited and ten of its twelve custom blocks were found to duplicate core blocks or to edit through Inspector fields. The core-first check, the variation-registration mechanics, the canvas-not-Inspector rule and the block-attribute layer of the styling cascade all come from that audit.
**Targets:** PHP 8.4+, WordPress 7.0, theme.json v3.

## TL;DR

1. **Check whether core already ships the block, before anything else.** See "Step 0" below. Core's block library grows every release, and a design brief that reads like a bespoke component (an accordion, a details disclosure, tabs, a cover, a media-and-text split) is frequently a core block plus styling. Building a custom block for something core ships is the most expensive mistake in this document.
2. **Editor-managed content must live as real blocks in the page, not as bare `wp:pattern` references.** A `wp:pattern` reference flattens into inline blocks the moment the page is saved in the editor -- verified on WP 7.0 core (see "Empirical test" below). Content-only editing does NOT keep the reference live.
3. **Content is edited on the canvas, not in the Inspector.** Copy belongs in real blocks the operator types into. An Inspector full of `TextControl`s, a textarea holding HTML or JSON, and a `ServerSideRender` preview standing in for editable content are all the same anti-pattern. See "Content on the canvas".
4. **Prefer a core block attribute over CSS** when core exposes a control for the thing. Column widths, vertical alignment, block gap, minimum height and background colour are attributes; reaching for CSS instead fights hooks core deliberately provides, and sometimes protects with `!important`. See "Use core's own attributes first".
5. Per-section mechanism: core ships it -> **core block, plus a variation and styles**; core does not ship it and it needs bespoke rendering, managed data or real interactivity -> **custom block**; a plain layout of core blocks whose content varies per page -> expanded core blocks in the page; content identical everywhere -> synced pattern; site structure (header/footer/nav) -> template part. Cross-cutting visual style -> theme.json / CSS.
6. **Mark content attributes `"role": "content"`** on the custom blocks that survive the core-first check, so they stay editable in WP 7.0 content-only mode. This is damage control for attributes a block legitimately owns; it is not licence to keep copy in attributes when the copy could be an inner block.
7. **Never frame a bespoke section by putting a class or inline styles on a core block.** It is not rebuild-proof: when someone deletes the block or the whole section and rebuilds it, the class/inline style is gone and the formatting silently reverts. Reach in this order: a core **block attribute**; then **theme.json**; then a registered **block style** (`is-style-*`), which is rebuild-proof because the operator can reapply it by name from the Styles panel; then a custom block that owns its own render and stylesheet.

## Step 0: does core already ship this block?

Run this before opening the decision model. It is the cheapest check in the process and the one most often skipped.

```bash
ls wp-includes/blocks/
```

Read the directory in the WordPress version the project actually runs, not a remembered list. Core's library has grown steadily, and a block added two releases ago will not be in an agent's training data. Check the `@since` line in the block's `render.php` or its `block.json` to confirm it exists in the target version.

If core ships it, the section is a core block. The design is then expressed as:

- a **block variation** for the starting shape (see "Registering a block variation"), and
- **theme.json styles** and/or a registered **block style** for the appearance.

Signals that an agent skipped this step, all of which have appeared in real code:

- A custom block whose `render.php` emits `<details>`/`<summary>`, a tab list, a quote with a citation, or an image beside text.
- A custom block named for a generic UI component (`accordion`, `tabs`, `quote`, `details`, `cover`) rather than for a domain concept.
- A theme with a large `blocks/` directory where every block is a static arrangement of headings, paragraphs, lists and buttons.

A custom block is justified by **behaviour or managed data core has no block for**. It is never justified by styling, and never by preferring a different editing UI.

**Worked example.** A design shows a panel of expandable question-and-answer rows. That reads like a bespoke FAQ component, and an earlier version of this document routed it to a custom block. Core has shipped `core/accordion`, `core/accordion-item`, `core/accordion-heading` and `core/accordion-panel` since 6.9 (`@since 6.9.0` in each block's server file). The correct build is a variation of `core/accordion` plus styles, and the custom block is dead weight that also has to reimplement keyboard operation and ARIA that core already ships.

**Counter-example.** A design shows a slider that cycles between photographic panels. Core ships no carousel. That is a real custom block, but only for the slider shell: the region and slide roles, the controls and the live region. Each slide should still be a `core/group` of ordinary blocks, rendered by iterating the block's inner blocks so each can be wrapped in the slide chrome. Split the behaviour core lacks from the content it does not.

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

Custom **blocks** are for functionality / managed components (custom rendering, structured data, media handling, interactivity) -- not for static arrangements of core blocks (those are patterns), and not for anything core already ships a block for (Step 0). A fifth mechanism sits alongside these four and is easy to forget: a **block variation** of a core block, which gives a named starting point in the inserter without any of the ownership cost of a custom block.

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

Work down this table in order. The first row that matches wins, which is why the core-first row is first.

| Section kind | Mechanism | Why |
|---|---|---|
| **Core already ships a block for it** (check `wp-includes/blocks/` first) | **Core block**, plus a **block variation** for the starting shape and **theme.json / a block style** for appearance | Core owns the markup, the accessibility and the editing experience, and maintains all three for free |
| Behaviour or managed data core has no block for (a carousel, a third-party embed with credentials, a structured record) | **Custom block**, scoped to the part core lacks; content attrs marked `role:"content"`, media via an `imageId` attribute, and inner blocks for anything that is really just content | Block instance persists in content; structure owned by the block and deploys with the theme |
| Plain layout of core blocks, content varies per page | **Expanded core blocks in the page content** (core/image, core/cover for media); the pattern stays as an inserter starting point only | Core blocks are inherently editable; content-only editing protects structure |
| Identical content reused site-wide (e.g. legal footnote) | **Synced pattern**, optionally with pattern overrides | One source updates everywhere |
| Site structure (header/footer/nav) | **Template part** (+ core/navigation for the menu) | Carries semantic meaning in block themes |
| Cross-cutting visual style (color, spacing) | **theme.json / CSS** | Applies regardless of how content is composed |

## Registering a block variation

A variation is how a theme ships a design as a core block. Two registration routes exist, both official.

**PHP, the `get_block_type_variations` filter.** No bundle, no build step. The filter runs in `WP_Block_Type::get_variations()`; the result reaches the editor through `get_block_editor_server_block_settings()`, which is inlined as a call to `wp.blocks.unstable__bootstrapServerSideBlockDefinitions`.

```php
add_filter(
    'get_block_type_variations',
    function ( array $variations, WP_Block_Type $block_type ): array {
        if ( 'core/accordion' !== $block_type->name ) {
            return $variations;
        }

        $item = array(
            'core/accordion-item',
            array(),
            array(
                array( 'core/accordion-heading' ),
                array( 'core/accordion-panel', array(), array( array( 'core/paragraph' ) ) ),
            ),
        );

        $variations[] = array(
            'name'        => 'faq',
            'title'       => __( 'FAQ', 'example-theme' ),
            'scope'       => array( 'inserter' ),
            'attributes'  => array( 'iconPosition' => 'right' ),
            'innerBlocks' => array( $item, $item, $item ),
        );

        return $variations;
    },
    10,
    2
);
```

**JavaScript, `wp.blocks.registerBlockVariation( name, variation )`.** Needed when the variation wants a live `isActive` callback or anything else that only exists client side.

Things worth knowing before writing either:

- **A server definition wins over the client one.** The bootstrapped-block-types reducer returns early when a name is already present, so a PHP variation registered against a core block survives the block library's own client-side registration.
- **`innerBlocks` is an array of `[ blockName, attributes, innerBlocks ]` triplets**, nested. In PHP an empty `array()` for attributes is fine: it JSON-encodes to `[]` and spreads to `{}` client side.
- **`scope`** is `inserter`, `block` or `transform`. An inserter-only starting point needs nothing else.
- **`isActive`** (a string array of attribute paths, or a function) is what makes an inserted block report as the variation in the UI. Omit it when the variation sets no attribute that distinguishes it, and do not invent a marker attribute purely to satisfy it.

Variation fields, per the Block Editor Handbook: `name` (required), `title`, `description`, `category`, `keywords`, `icon`, `attributes`, `innerBlocks`, `example`, `scope`, `isDefault`, `isActive`.

## Content on the canvas, not in the Inspector

The operator edits real blocks in the content area, the way core works. The following are all the same mistake wearing different clothes, and all of them have shipped in real themes:

- Rich text or copy in a `TextControl` or `TextareaControl` in the Inspector.
- HTML typed into a field, usually to let part of a heading carry a different colour or weight.
- **JSON in any field, ever.** A repeating structure is repeating blocks (inner blocks, `core/list`, `core/accordion-item`), not an array attribute edited as text.
- Splitting one heading across several fields because the design colours part of it.
- `ServerSideRender` standing in for editable content, which gives the operator a picture of their page instead of their page.

Inspector controls are for **settings**: a tone, an alignment, an icon position, a count. Not for copy.

Two mechanisms remove most of the temptation:

- **Inner blocks** for anything repeating or composed. A custom block can own the shell and still let every child be an ordinary block.
- **A custom inline format** (`wp.richText.registerFormatType`) for a run of differently-styled text inside a heading or quotation. That is the supported way to let an operator mark up part of a string from the toolbar, and it replaces the "type a span into this field" pattern outright. Register one format and let each section decide its size and weight in CSS.

Converting a JSON textarea into a nicer repeater UI is not the fix. It is a better Inspector for something that should not be in the Inspector.

## Use core's own attributes first

Before writing CSS for a core block, check whether core exposes an attribute for it. Common ones that get reimplemented in CSS by mistake: `core/columns` `verticalAlignment`, per-column `width`, `spacing.blockGap`, `dimensions.minHeight`, `backgroundColor` and `textColor`, `style.spacing.padding`.

Two reasons this matters beyond tidiness.

**Core sometimes defends its own hooks with `!important`.** `core/columns` ships `.wp-block-columns{align-items:normal!important}` precisely so that the per-column `is-vertically-aligned-*` classes its `verticalAlignment` attribute emits can win via `align-self`. A theme that sets `align-items: center` on the columns block gets silently ignored, and the instinct to answer with a louder `!important` is the wrong lesson. The attribute is the mechanism.

**Attributes stay in the operator's sidebar.** A value expressed as an attribute is visible and changeable; the same value in a stylesheet is not. Where the design has one answer for every instance, theme.json is right. Where it is per-instance, the attribute is right. CSS is for what neither can express: breakpoints, pseudo-elements, and selectors that reach inside a block's markup.

**Equal columns carry no `width` at all.** Omitting the attribute is core's own default for an even split, which is more robust than writing `50%` and much more robust than fractional percentages derived from a design's pixel geometry, which have to be recomputed by hand whenever a column's contents change.

## Generating valid core markup for seeded content

Themes that provision page content as code (a seed file, a fixture, an importer) have to produce core's exact save markup, or the editor reports invalid blocks on first load. Hand-writing that markup for `core/columns`, `core/quote`, `core/image` or the accordion family is error-prone: the block comment's attributes and the serialised HTML must agree exactly, including details like a percentage losing its trailing zero between the comment (`"51.70%"`) and the inline style (`flex-basis:51.7%`).

A reliable way to produce it, rather than hand-write it:

1. Build the block tree in the live editor with `wp.blocks.createBlock`.
2. Apply it with `wp.data.dispatch( 'core/block-editor' ).resetBlocks( blocks )`.
3. Save with `wp.data.dispatch( 'core/editor' ).savePost()`.
4. Export the stored content back into the seed file with `wp post get <id> --field=post_content`, and re-tokenise any attachment ids and URLs.

That produces core's exact output by construction. Verify by re-seeding from the file and confirming the editor reports zero invalid blocks, so what is committed is what was checked rather than the editor session that produced it.

## Rules for future agents

1. **Read `wp-includes/blocks/` before building anything.** Core-first is the first question, not a later optimisation. A custom block that duplicates a core block is the most expensive outcome in this document, because it also reimplements the accessibility and keyboard behaviour core maintains.
2. **Page content is real blocks, not bare `wp:pattern` references**, for anything a human will edit. References flatten on first save (tested above) and are not editor-stable.
3. **Copy is never an Inspector field.** No rich text in a `TextControl`, no HTML in a textarea, no JSON anywhere, no `ServerSideRender` standing in for editable content. Use inner blocks for structure and a registered inline format for a differently-styled run inside a string.
4. **Reach for a core block attribute before CSS.** Core sometimes protects its own attribute hooks with `!important`; a stylesheet that loses to core is usually a sign the attribute was the intended mechanism.
5. **Mark `"role": "content"`** on custom-block attributes that hold editable text or media (titles, descriptions, button text/URLs, image IDs/alt). Do NOT mark pure settings (toggles, speeds, overlap px, layout switches). Marking an attribute `role: "content"` does not make it the right home for copy; prefer an inner block where one will do.
6. **Images that should be editor-swappable** go through the media library: store an `imageId` attribute on the block and render with `wp_get_attachment_image()` (a theme-file fallback is fine until one is chosen). If content is provisioned programmatically (a seed script, an import), import assets into the media library rather than hardcoding a theme-file URL, so the result stays editor-swappable.
7. **A hardcoded theme-file image URL inside a pattern is a smell** -- the content editor cannot manage it. Make it a block attribute or a media-library attachment instead.
8. **Patterns remain useful** as inserter starting points and as the dev-time composition source, but the page's saved content is the editable source of truth.
9. **Don't reach for a pattern when the thing carries per-page editor-set media or copy** -- that is a block (or expanded core blocks). Reserve patterns for fixed structural layout reused as-is.

## Styling cascade: where a style belongs

Block-theme styling is a defined cascade, most-global to most-specific (Theme Handbook / Block Editor Handbook). Put each value at the RIGHT layer; the layer is decided by SCOPE, not preference.

1. **theme.json root `styles`** -- global defaults that "trickle down through the design and are used unless a more specific element or block style overrides them."
2. **theme.json `styles.elements.<element>`** (e.g. `button` -> `.wp-element-button`) -- the site-wide default for that element type.
3. **theme.json `styles.blocks.<block>`** -- default for a block type.
4. **Block style variations** (`.is-style-{name}`, registered via `register_block_style()` or a `/styles` partial) -- named, selectable variants. The Theme Handbook RECOMMENDS these over custom CSS "because they appear in the Appearance > Editor > Styles interface, and your theme users can make their own changes."
5. **Custom CSS class / stylesheet** -- only for what theme.json / variations cannot express: breakpoints, pseudo-elements, and selectors reaching inside a block's markup.
6. **Inline `style=` literal values** -- avoid where a design-token layer above could express the same value. A preset-referencing inline value (e.g. `padding:var(--wp--preset--spacing--lg)`) is fine; a literal value (e.g. `border-radius:20px`) usually belongs at a layer above instead.

**A per-instance value sits outside this cascade entirely: it is a block attribute.** The cascade above answers "which layer owns this value for a class of blocks". When the value differs per instance and core exposes a control for it, none of these layers is the answer; the block's own attribute is, because it is the only option the operator can see and change. Column widths, vertical alignment, block gap, minimum height and tone are attributes, not stylesheet entries. Note that a block style variation can then style the framing while the attribute carries the instance value, and the two do not conflict.

**Decision rule:** does the value differ per instance, and does core expose a control for it? -> **block attribute**. Is it the default for EVERY instance of an element/block? -> global layer (theme.json elements/blocks, set once). Is it a VARIANT (only some instances) that no attribute expresses? -> a block style variation (preferred, editor-visible/user-editable). Does it need a breakpoint or a pseudo-element? -> a stylesheet, attached to a registered block style so it stays reapplicable. Is it already the global default? -> don't restate it.

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
- Block Variations (Block Editor Handbook): <https://developer.wordpress.org/block-editor/reference-guides/block-api/block-variations/>
- Rich Text formats, `registerFormatType` (Block Editor Handbook): <https://developer.wordpress.org/block-editor/reference-guides/richtext/>
- Global Settings & Styles / theme.json (Block Editor Handbook): <https://developer.wordpress.org/block-editor/how-to-guides/themes/global-settings-and-styles/>

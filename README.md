# wordpress-7-blocks-patterns-best-practices

[![CI](https://github.com/84emllc/claude-wordpress-7-blocks-patterns-best-practices-skill/actions/workflows/ci.yml/badge.svg)](https://github.com/84emllc/claude-wordpress-7-blocks-patterns-best-practices-skill/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](./LICENSE)
[![Agent Skill](https://img.shields.io/badge/type-agent%20skill-8250df.svg)](https://agentskills.io/specification)

An agent skill for deciding whether a WordPress block-theme section should be a custom block, a pattern, a synced pattern, or a template part -- especially on WordPress 7.0+, where content-only editing changed how patterns and template parts behave. Includes an empirical test (not just the official docs) confirming that a bare `wp:pattern` reference flattens into inline blocks on first editor save, and a styling-cascade decision model for where a given CSS value belongs (theme.json vs block style variation vs class vs inline).

Built by [84EM](https://84em.com) for use with Claude Code and other agent harnesses that load [Agent Skills](https://agentskills.io/specification).

## What is in here

| Path | What it is | License |
|------|------------|---------|
| `SKILL.md` | The skill: the official reuse-mechanism model, WP 7.0 content-only editing changes, the empirical pattern-flattening test, a decision table, and the styling cascade | MIT |

## Install

**Claude Code (personal skill):** clone and symlink into your skills directory.

```bash
git clone https://github.com/84emllc/claude-wordpress-7-blocks-patterns-best-practices-skill.git
ln -sfn "$(pwd)/claude-wordpress-7-blocks-patterns-best-practices-skill" ~/.claude/skills/wordpress-7-blocks-patterns-best-practices
```

The skill is then discoverable by its `name` and `description` frontmatter. No build step is required.

## Usage

Ask the agent to decide how to build a section of a WordPress 7.0+ block theme, or to review whether a theme is using blocks, patterns, and template parts correctly. The skill gives a decision table (functional component -> custom block; per-page-varying layout -> expanded core blocks; identical-everywhere content -> synced pattern; site structure -> template part) plus the empirical finding that bare pattern references are not editor-stable, so a human-editable section should never rely on one.

## Licensing

MIT. See [LICENSE](./LICENSE).

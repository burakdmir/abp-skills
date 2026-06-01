# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A **content-only** repository: AI agent skill files for ABP Framework v10.4, targeting two tools — Claude Code and OpenCode. There is no application code, build system, test suite, or lint step. Every change is editing Markdown.

The deliverable is 46 `SKILL.md` files (23 topics × 2 tools) plus repo docs (README, CONTRIBUTING, etc.). Skill content is written in **Turkish prose** with English code/technical terms. Targets ABP v10.4 on **.NET 10**.

## Architecture

Two parallel skill trees, one directory per topic:

```
claude/abp-{topic}/SKILL.md      # detailed version  (~210–850 lines)
opencode/abp-{topic}/SKILL.md    # compact version   (~55–230 lines)
```

The **same 23 topics exist in both trees** with identical directory names (`abp-framework`, `abp-ddd`, `abp-efcore`, ...). The two versions of a topic cover the same material at different depth:

- **`claude/`** — detailed, comprehensive: full explanations, multiple patterns, deep scenarios.
- **`opencode/`** — compact quick-reference: condensed concepts and copy-paste snippets.

The README "Skill Matrix" table is the canonical index of all 23 topics and links to both versions of each.

## The Core Invariant: Keep Both Trees In Sync

When adding or substantively changing a topic, **edit both `claude/` and `opencode/` versions**. Adding a topic to only one tree breaks the matrix and the repo's contract (CONTRIBUTING: "If adding a new topic, create both Claude and OpenCode versions").

After adding a new topic, also update:
- `README.md` Skill Matrix table (add a numbered row with links to both files)
- `README.md` Stats section (file count, line count)

## SKILL.md File Structure

Every `SKILL.md` follows this section order (per CONTRIBUTING):

0. **YAML frontmatter** — **required** first bytes of the file: `name` (matches directory) + `description` (what + when, drives agent auto-activation). Both Claude Code and OpenCode skill formats require this.
1. `# ABP {Topic} ...` title + one-line Turkish summary
2. `## Trigger` — bullet list of activation phrases/keywords (mostly Turkish, plus literal commands like `abp new`)
3. Core concepts (Turkish prose, tables for option matrices)
4. Code examples — production-ready, copy-paste ready (C# / ASP.NET Core, ABP conventions)
5. Best practices
6. `## Related` — cross-links to sibling skills and official ABP docs

Match the existing style of the topic's tree when editing: detailed for `claude/`, terse for `opencode/`.

## Conventions

- All content must be grounded in official ABP Framework v10.4 docs (https://abp.io/docs/latest). Don't invent APIs.
- Code examples follow ABP conventions (DDD layering, `ITransientDependency`, `[ExposeServices]`, etc.) — see `abp-ddd` and `abp-dependency-injection` skills for the canonical patterns.
- Naming: `abp-{topic}/SKILL.md` only.
- Commit style: Conventional Commits (`feat:`, `docs:`), matching git history.

## Auxiliary Content

- `community-article/article.md` — an ABP Community article draft about the project.
- `.github/` — issue/PR templates, funding, social preview assets.

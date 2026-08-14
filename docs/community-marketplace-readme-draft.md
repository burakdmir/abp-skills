# Post-Acceptance README Draft — Community Marketplace

> Use this **only after** `abp-sensei` is accepted into `anthropics/claude-plugins-community`.
> It replaces the current `### Install` block in `README.md` (the one under
> `## Claude Code Plugin — **ABP Sensei**`). Nothing else in the README changes.
> Keep the existing `### Updating` section as-is — it still applies.

---

## Replacement for the `### Install` section

```markdown
### Install

**From the community marketplace (recommended)** — auto-update is on by default,
and the plugin shows up in Claude Code's **Discover** tab so users can install it
from the UI without typing a marketplace command:

\`\`\`bash
# 1. Add Anthropic's community marketplace (one-time)
/plugin marketplace add anthropics/claude-plugins-community

# 2. Install ABP Sensei
/plugin install abp-sensei@claude-plugins-community
\`\`\`

**Direct from this repo** — always tracks the latest `main`:

\`\`\`bash
/plugin marketplace add burakdmir/abp-skills
/plugin install abp-sensei@abp-skills
\`\`\`

That's it — skills, subagent, and commands are immediately available. No manual file copying.
```

---

## Apply checklist

1. Confirm acceptance email / PR landed in `anthropics/claude-plugins-community`.
2. Verify the exact install id: `abp-sensei@claude-plugins-community`
   (the marketplace name after `@` is whatever the community repo's
   `marketplace.json` declares — confirm before publishing).
3. Swap the block above into `README.md`.
4. Add a badge near the top (optional):
   `[![Community Marketplace](https://img.shields.io/badge/Claude%20Code-Community%20Marketplace-blueviolet)](https://github.com/anthropics/claude-plugins-community)`
5. Ship via branch → PR → squash merge (main is protected).

## Notes

- If a user installs from **both** marketplaces, Claude Code treats them as the
  same plugin by `name` (`abp-sensei`); keep the manifest `name` identical so
  there's no duplicate listing.
- Community marketplace = auto-update **on by default**; the repo-direct path
  still needs the per-user `autoUpdate` opt-in documented in `### Updating`.
- Do **not** point users at `anthropics/claude-plugins-official` — that's
  invitation-only, no open submission.

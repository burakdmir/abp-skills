# Contributing to ABP Framework Skills

Thank you for your interest in contributing! This repository contains skill files for Claude Code and OpenCode that help AI agents work with ABP Framework v10.4.

## How to Contribute

### 1. Fork and Clone

```bash
git clone https://github.com/YOUR_USERNAME/abp-skills.git
cd abp-skills
```

### 2. Create a Branch

```bash
git checkout -b feat/add-new-skill
```

### 3. Make Your Changes

- Each skill lives in its own directory under `claude/` or `opencode/`
- Claude skill files should be detailed (200-800+ lines)
- OpenCode skill files should be compact (50-200 lines)
- Follow the existing naming convention: `abp-{topic}/SKILL.md`
- Each SKILL.md must have a `## Trigger` section

### 4. Commit and Push

```bash
git add -A
git commit -m "feat: add abp-{topic} skill"
git push origin feat/add-new-skill
```

### 5. Open a Pull Request

- Describe what you added or changed
- Reference any ABP documentation URLs you used
- Mention which AI tool the skill targets (Claude, OpenCode, or both)

## Skill File Structure

```
{tool}/abp-{topic}/SKILL.md
```

### Required Sections

1. **Title** — `# ABP {Topic} Skill`
2. **Trigger** — Keywords that activate the skill
3. **Core Concepts** — Brief overview
4. **Code Examples** — Practical, copy-paste ready examples
5. **Best Practices** — Actionable recommendations
6. **Related** — Links to other skills or external docs

## Guidelines

- **Accuracy**: All content must be based on official ABP Framework v10.4 documentation
- **Code Quality**: Examples should be production-ready and follow ABP conventions
- **No Fluff**: Keep explanations concise and actionable
- **Cross-reference**: Link to related skills when relevant
- **Both versions**: If adding a new topic, create both Claude and OpenCode versions

## Reporting Issues

- Use GitHub Issues for bugs, missing topics, or outdated information
- Include the ABP version and the specific skill file affected

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

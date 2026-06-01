# ABP Framework Skills

> Comprehensive AI agent skill files for **Claude Code** and **OpenCode** covering ABP Framework v10.4.

<p align="center">
  <a href="#available-skills"><strong>23 Skills</strong></a> ·
  <a href="#quick-start"><strong>Quick Start</strong></a> ·
  <a href="#skill-matrix"><strong>Skill Matrix</strong></a> ·
  <a href="#contributing"><strong>Contributing</strong></a>
</p>

<p align="center">
  <a href="https://github.com/burakdmir/abp-skills/stargazers"><img src="https://img.shields.io/github/stars/burakdmir/abp-skills?style=for-the-badge&logo=github" alt="Stars"></a>
  <a href="https://github.com/burakdmir/abp-skills/blob/main/LICENSE"><img src="https://img.shields.io/badge/License-MIT-green.svg?style=for-the-badge" alt="License"></a>
  <a href="https://abp.io/docs/latest"><img src="https://img.shields.io/badge/ABP-v10.4-6b21a8?style=for-the-badge&logo=data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCIgZmlsbD0id2hpdGUiPjxwYXRoIGQ9Ik0xMiAyTDIgN2wxMCA1IDEwLTUtMTAtNXpNMiAxN2wxMCA1IDEwLTVNMiAxMmwxMCA1IDEwLTUiLz48L3N2Zz4=" alt="ABP v10.4"></a>
  <a href="https://github.com/burakdmir/abp-skills/pulse"><img src="https://img.shields.io/github/last-commit/burakdmir/abp-skills?style=for-the-badge&logo=git" alt="Last Commit"></a>
</p>

---

## What is This?

This repository contains **46 skill files** (23 Claude Code + 23 OpenCode) that teach AI coding agents how to work effectively with **ABP Framework v10.4** (.NET 10). Each skill covers a specific ABP topic with:

- **YAML frontmatter** — `name` + `description` for agent auto-activation (Claude Code / OpenCode skill format)
- **Trigger keywords** — when the AI should activate the skill
- **Core concepts** — concise explanations of ABP patterns
- **Code examples** — production-ready, copy-paste ready snippets
- **Best practices** — actionable recommendations from real-world usage
- **Cross-references** — links to related skills and official docs

### Claude vs OpenCode Versions

| | Claude Code | OpenCode |
|---|---|---|
| **Style** | Detailed, comprehensive | Compact, quick-reference |
| **Lines** | ~210–850 per file | ~55–230 per file |
| **Best for** | Deep understanding, complex scenarios | Fast lookup, inline reference |
| **Format** | Full explanations + patterns | Quick reference + snippets |

---

## Quick Start

### Claude Code

```bash
# Clone into your Claude skills directory
git clone https://github.com/burakdmir/abp-skills.git ~/.claude/skills/abp-skills
```

Or reference individual skills in your `.claude/CLAUDE.md`:

```markdown
## ABP Framework Skills
See: ~/.claude/skills/abp-skills/claude/
```

### OpenCode

```bash
# Clone into your OpenCode skills directory
git clone https://github.com/burakdmir/abp-skills.git ~/.config/opencode/skills/abp-skills
```

### Manual Installation

Download individual skill files and place them in your AI agent's skills directory:

```
your-project/
├── .claude/
│   └── skills/
│       └── abp-{topic}/
│           └── SKILL.md
└── .opencode/
    └── skills/
        └── abp-{topic}/
            └── SKILL.md
```

---

## Skill Matrix

| # | Skill | Claude | OpenCode | Coverage |
|---|---|---|---|---|
| 1 | **Framework** | [SKILL.md](claude/abp-framework/SKILL.md) | [SKILL.md](opencode/abp-framework/SKILL.md) | Getting started, solution templates, project structure |
| 2 | **DDD** | [SKILL.md](claude/abp-ddd/SKILL.md) | [SKILL.md](opencode/abp-ddd/SKILL.md) | Entities, aggregates, repositories, app services, DTOs, UOW, specifications |
| 3 | **Modularity** | [SKILL.md](claude/abp-modularity/SKILL.md) | [SKILL.md](opencode/abp-modularity/SKILL.md) | Module system, DependsOn, lifecycle, plugin modules, modular monolith |
| 4 | **EF Core** | [SKILL.md](claude/abp-efcore/SKILL.md) | [SKILL.md](opencode/abp-efcore/SKILL.md) | DbContext, repositories, migrations, PostgreSQL/MySQL/SQLite/Oracle |
| 5 | **MongoDB** | [SKILL.md](claude/abp-mongodb/SKILL.md) | [SKILL.md](opencode/abp-mongodb/SKILL.md) | MongoDbContext, collections, indexes, transactions, replica set |
| 6 | **Multi-Tenancy** | [SKILL.md](claude/abp-multitenancy/SKILL.md) | [SKILL.md](opencode/abp-multitenancy/SKILL.md) | Tenant resolvers, ICurrentTenant, IMultiTenant, database isolation |
| 7 | **CLI** | [SKILL.md](claude/abp-cli/SKILL.md) | [SKILL.md](opencode/abp-cli/SKILL.md) | ABP CLI commands, new, add-package, build, new-package |
| 8 | **UI** | [SKILL.md](claude/abp-ui/SKILL.md) | [SKILL.md](opencode/abp-ui/SKILL.md) | MVC/Razor Pages, Blazor, Angular, React, UI components |
| 9 | **Infrastructure** | [SKILL.md](claude/abp-infrastructure/SKILL.md) | [SKILL.md](opencode/abp-infrastructure/SKILL.md) | Event Bus, Background Jobs, Caching, Redis, BLOB, Emailing, SignalR |
| 10 | **API** | [SKILL.md](claude/abp-api/SKILL.md) | [SKILL.md](opencode/abp-api/SKILL.md) | API controllers, dynamic proxies, Swagger, versioning, dynamic clients |
| 11 | **Authorization** | [SKILL.md](claude/abp-authorization/SKILL.md) | [SKILL.md](opencode/abp-authorization/SKILL.md) | Permission system, permission groups, resource-based auth, policies |
| 12 | **Validation** | [SKILL.md](claude/abp-validation/SKILL.md) | [SKILL.md](opencode/abp-validation/SKILL.md) | DTO validation, Data Annotations, FluentValidation, IValidatableObject |
| 13 | **Exception Handling** | [SKILL.md](claude/abp-exception-handling/SKILL.md) | [SKILL.md](opencode/abp-exception-handling/SKILL.md) | Error handling, RemoteServiceErrorResponse, HTTP status mapping |
| 14 | **Localization** | [SKILL.md](claude/abp-localization/SKILL.md) | [SKILL.md](opencode/abp-localization/SKILL.md) | Localization resources, JSON files, culture fallback, L[] helper |
| 15 | **Settings & Features** | [SKILL.md](claude/abp-settings-features/SKILL.md) | [SKILL.md](opencode/abp-settings-features/SKILL.md) | ISettingProvider, ISettingManager, IFeatureChecker, feature toggles |
| 16 | **Audit Logging** | [SKILL.md](claude/abp-audit-logging/SKILL.md) | [SKILL.md](opencode/abp-audit-logging/SKILL.md) | Audit logging, entity history, AbpAuditingOptions, IAuditingStore |
| 17 | **Dependency Injection** | [SKILL.md](claude/abp-dependency-injection/SKILL.md) | [SKILL.md](opencode/abp-dependency-injection/SKILL.md) | DI, ITransientDependency, [Dependency], [ExposeServices], LazyServiceProvider, Autofac |
| 18 | **Testing** | [SKILL.md](claude/abp-testing/SKILL.md) | [SKILL.md](opencode/abp-testing/SKILL.md) | Integration tests, *TestBase, SQLite in-memory, Shouldly, NSubstitute, data seeding |
| 19 | **Microservices** | [SKILL.md](claude/abp-microservices/SKILL.md) | [SKILL.md](opencode/abp-microservices/SKILL.md) | Solution structure, Integration Services, distributed events (RabbitMQ), YARP gateway, OpenIddict |
| 20 | **Object Mapping** | [SKILL.md](claude/abp-object-mapping/SKILL.md) | [SKILL.md](opencode/abp-object-mapping/SKILL.md) | IObjectMapper, Mapperly (default), AutoMapper profiles, entity↔DTO mapping |
| 21 | **Development Flow** | [SKILL.md](claude/abp-development-flow/SKILL.md) | [SKILL.md](opencode/abp-development-flow/SKILL.md) | End-to-end entity flow: domain → migration → contracts → service → permission → test |
| 22 | **Dependency Rules** | [SKILL.md](claude/abp-dependency-rules/SKILL.md) | [SKILL.md](opencode/abp-dependency-rules/SKILL.md) | Layer dependency direction, project reference matrix, architecture anti-patterns |
| 23 | **Deployment** | [SKILL.md](claude/abp-deployment/SKILL.md) | [SKILL.md](opencode/abp-deployment/SKILL.md) | Clustered/stateless, distributed cache/lock, forwarded headers, SSL, OpenIddict prod, Docker/Helm |

---

## Stats

- **46 SKILL.md files** (23 Claude + 23 OpenCode)
- **~9,960 total lines** of skill content
- **ABP Framework v10.4** (.NET 10) documentation based — cross-checked against the official `ai-rules` and `docs`
- **YAML frontmatter** on every skill for agent auto-activation
- **All examples** production-ready

---

## Contributing

Contributions are welcome! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

### Quick Contribution

1. Fork the repository
2. Create a branch: `git checkout -b feat/add-abp-{topic}`
3. Add your skill file(s) under `claude/` and/or `opencode/`
4. Commit and push: `git commit -m "feat: add abp-{topic} skill"`
5. Open a Pull Request

### Request a New Skill

Open an issue using the [Skill Request template](.github/ISSUE_TEMPLATE/skill-request.md).

---

## Related Resources

- [ABP Framework Official Docs](https://abp.io/docs/latest)
- [ABP Community](https://community.abp.io/)
- [ABP GitHub Repository](https://github.com/abpframework/abp)
- [Claude Code Docs](https://docs.anthropic.com/en/docs/claude-code/overview)
- [OpenCode Docs](https://opencode.ai)

---

## License

[MIT](LICENSE) — Free to use, modify, and distribute.

---

<p align="center">
  Made with ❤️ for the ABP Framework community
</p>

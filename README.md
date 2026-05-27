# ABP Framework Skills

Comprehensive skill files for **Claude Code** and **OpenCode** covering ABP Framework v10.4.

## Overview

| Directory | Description |
|---|---|
| [`claude/`](claude/) | Detailed skill files for Claude Code (200-840 lines each) |
| [`opencode/`](opencode/) | Compact skill files for OpenCode (53-587 lines each) |

## Available Skills

| Skill | Claude | OpenCode | Coverage |
|---|---|---|---|
| **abp-framework** | [SKILL.md](claude/abp-framework/SKILL.md) | [SKILL.md](opencode/abp-framework/SKILL.md) | Getting started, solution templates, project structure |
| **abp-ddd** | [SKILL.md](claude/abp-ddd/SKILL.md) | [SKILL.md](opencode/abp-ddd/SKILL.md) | Entities, aggregates, repositories, application services, DTOs, UOW, specifications |
| **abp-modularity** | [SKILL.md](claude/abp-modularity/SKILL.md) | [SKILL.md](opencode/abp-modularity/SKILL.md) | Module system, DependsOn, lifecycle, plugin modules, modular monolith |
| **abp-efcore** | [SKILL.md](claude/abp-efcore/SKILL.md) | [SKILL.md](opencode/abp-efcore/SKILL.md) | DbContext, repositories, migrations, PostgreSQL/MySQL/SQLite/Oracle |
| **abp-mongodb** | [SKILL.md](claude/abp-mongodb/SKILL.md) | [SKILL.md](opencode/abp-mongodb/SKILL.md) | MongoDbContext, collections, indexes, transactions, replica set |
| **abp-multitenancy** | [SKILL.md](claude/abp-multitenancy/SKILL.md) | [SKILL.md](opencode/abp-multitenancy/SKILL.md) | Tenant resolvers, ICurrentTenant, IMultiTenant, database isolation |
| **abp-cli** | [SKILL.md](claude/abp-cli/SKILL.md) | [SKILL.md](opencode/abp-cli/SKILL.md) | ABP CLI commands, new, add-package, build, new-package |
| **abp-ui** | [SKILL.md](claude/abp-ui/SKILL.md) | [SKILL.md](opencode/abp-ui/SKILL.md) | MVC/Razor Pages, Blazor, Angular, React, UI components |
| **abp-infrastructure** | [SKILL.md](claude/abp-infrastructure/SKILL.md) | [SKILL.md](opencode/abp-infrastructure/SKILL.md) | Event Bus, Background Jobs, Caching, Redis, BLOB, Emailing, SignalR, Data Filtering |
| **abp-api** | [SKILL.md](claude/abp-api/SKILL.md) | [SKILL.md](opencode/abp-api/SKILL.md) | API controllers, dynamic proxies, Swagger, versioning, dynamic clients |
| **abp-authorization** | [SKILL.md](claude/abp-authorization/SKILL.md) | [SKILL.md](opencode/abp-authorization/SKILL.md) | Permission system, permission groups, resource-based auth, policies |
| **abp-validation** | [SKILL.md](claude/abp-validation/SKILL.md) | [SKILL.md](opencode/abp-validation/SKILL.md) | DTO validation, Data Annotations, FluentValidation, IValidatableObject |
| **abp-exception-handling** | [SKILL.md](claude/abp-exception-handling/SKILL.md) | [SKILL.md](opencode/abp-exception-handling/SKILL.md) | Error handling, RemoteServiceErrorResponse, HTTP status mapping, business exceptions |
| **abp-localization** | [SKILL.md](claude/abp-localization/SKILL.md) | [SKILL.md](opencode/abp-localization/SKILL.md) | Localization resources, JSON files, culture fallback, L[] helper |
| **abp-settings-features** | [SKILL.md](claude/abp-settings-features/SKILL.md) | [SKILL.md](opencode/abp-settings-features/SKILL.md) | ISettingProvider, ISettingManager, IFeatureChecker, feature toggles |
| **abp-audit-logging** | [SKILL.md](claude/abp-audit-logging/SKILL.md) | [SKILL.md](opencode/abp-audit-logging/SKILL.md) | Audit logging, entity history, AbpAuditingOptions, IAuditingStore |
| **abp-dependency-injection** | [SKILL.md](claude/abp-dependency-injection/SKILL.md) | [SKILL.md](opencode/abp-dependency-injection/SKILL.md) | DI, ITransientDependency, [Dependency], [ExposeServices], Autofac |

## Usage

### Claude Code
Place skill directories under your project's `.claude/skills/` directory or reference them in your Claude Code configuration.

### OpenCode
Place skill directories under your project's `.opencode/skills/` directory or reference them in your OpenCode configuration.

## Stats

- **34 SKILL.md files** (17 Claude + 17 OpenCode)
- **~8,551 total lines** of skill content
- **ABP Framework v10.4** documentation based

## License

MIT

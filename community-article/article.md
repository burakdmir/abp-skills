# Empowering AI Agents with ABP Framework: A Comprehensive Skill Collection

> How I built 17 AI agent skills covering every aspect of ABP Framework v10.4 — and why it matters for the future of .NET development.

---

## The Problem

AI coding assistants like Claude Code and OpenCode are transforming how we write software. But they have a fundamental limitation: **they don't know your framework**.

When you ask an AI agent to "create a new application service in ABP," it might generate something that looks reasonable but misses critical ABP patterns:

- No `ApplicationService` base class inheritance
- Missing `[Authorize]` permission checks
- Direct `DbContext` access instead of `IRepository`
- No UOW attribute
- Manual DTO mapping instead of using the configured object mapper
- Ignoring localization, validation, and exception handling conventions

The result? Code that compiles but doesn't follow ABP best practices — and requires significant manual correction.

## The Solution: AI Agent Skills

AI agents support **skills** — structured knowledge files that teach them specific frameworks, patterns, and conventions. When a skill is active, the AI's responses are grounded in real framework documentation instead of generic patterns.

I built **abp-skills** — a collection of 34 skill files (17 topics × 2 AI tools) covering every major aspect of ABP Framework v10.4.

**GitHub Repository:** [github.com/burakdmir/abp-skills](https://github.com/burakdmir/abp-skills)

## What's Included

The repository covers 17 ABP topics, each with two versions:

| # | Topic | What It Covers |
|---|---|---|
| 1 | **Framework** | Getting started, solution templates, project structure |
| 2 | **DDD** | Entities, aggregates, repositories, app services, DTOs, UOW, specifications |
| 3 | **Modularity** | Module system, DependsOn, lifecycle, plugin modules, modular monolith |
| 4 | **EF Core** | DbContext, repositories, migrations, PostgreSQL/MySQL/SQLite/Oracle |
| 5 | **MongoDB** | MongoDbContext, collections, indexes, transactions, replica set |
| 6 | **Multi-Tenancy** | Tenant resolvers, ICurrentTenant, IMultiTenant, database isolation |
| 7 | **CLI** | ABP CLI commands, new, add-package, build, new-package |
| 8 | **UI** | MVC/Razor Pages, Blazor, Angular, React, UI components |
| 9 | **Infrastructure** | Event Bus, Background Jobs, Caching, Redis, BLOB, Emailing, SignalR |
| 10 | **API** | API controllers, dynamic proxies, Swagger, versioning, dynamic clients |
| 11 | **Authorization** | Permission system, permission groups, resource-based auth, policies |
| 12 | **Validation** | DTO validation, Data Annotations, FluentValidation, IValidatableObject |
| 13 | **Exception Handling** | Error handling, RemoteServiceErrorResponse, HTTP status mapping |
| 14 | **Localization** | Localization resources, JSON files, culture fallback, L[] helper |
| 15 | **Settings & Features** | ISettingProvider, ISettingManager, IFeatureChecker, feature toggles |
| 16 | **Audit Logging** | Audit logging, entity history, AbpAuditingOptions, IAuditingStore |
| 17 | **Dependency Injection** | DI, ITransientDependency, [Dependency], [ExposeServices], Autofac |

Each topic has:
- **Claude Code version** — Detailed (200-840 lines) with full explanations, patterns, and best practices
- **OpenCode version** — Compact (53-587 lines) as quick-reference for inline use

## How It Works

### Skill Activation

Each skill file defines a **Trigger** section — keywords that tell the AI when to activate it:

```markdown
## Trigger
- "ABP authorization"
- "ABP permission"
- "ABP role"
- "ABP policy"
```

When you mention any of these terms, the AI loads the full skill content and grounds its response in ABP-specific patterns.

### Example: Before vs After

**Without the skill:**
```csharp
// AI generates generic code
public class BookService
{
    private readonly AppDbContext _db;
    public async Task<Book> Create(Book book)
    {
        _db.Books.Add(book);
        await _db.SaveChangesAsync();
        return book;
    }
}
```

**With the ABP Authorization skill active:**
```csharp
// AI generates ABP-conformant code
public class BookAppService : ApplicationService, IBookAppService
{
    private readonly IRepository<Book, Guid> _bookRepository;

    public BookAppService(IRepository<Book, Guid> bookRepository)
    {
        _bookRepository = bookRepository;
    }

    [Authorize("BookStore_Books_Create")]
    public async Task<BookDto> CreateAsync(CreateBookDto input)
    {
        var book = ObjectMapper.Map<CreateBookDto, Book>(input);
        return ObjectMapper.Map<Book, BookDto>(
            await _bookRepository.InsertAsync(book, autoSave: true)
        );
    }
}
```

The difference is night and day: proper base class, repository abstraction, permission check, object mapping, and UOW handling — all automatically.

## Installation

### Claude Code

```bash
git clone https://github.com/burakdmir/abp-skills.git ~/.claude/skills/abp-skills
```

### OpenCode

```bash
git clone https://github.com/burakdmir/abp-skills.git ~/.config/opencode/skills/abp-skills
```

### Per-Project

Drop individual skill directories into your project's `.claude/skills/` or `.opencode/skills/` folder.

## Real-World Impact

Here's what changes when AI agents have ABP skills:

### 1. Faster Development Cycles
No more explaining ABP conventions to the AI. It already knows that:
- Application services inherit from `ApplicationService`
- Repositories use `IRepository<TEntity, TKey>`
- Permissions are defined via `PermissionDefinitionProvider`
- Settings use `ISettingProvider` with a fallback chain

### 2. Consistent Code Quality
Every generated snippet follows ABP patterns:
- Proper DDD layer separation
- Correct dependency injection conventions
- Standard exception handling and validation
- Localization-ready code structure

### 3. Reduced Review Time
Code generated with skills active requires less manual correction because:
- It uses the right base classes and interfaces
- It includes authorization checks
- It follows the options pattern for configuration
- It respects multi-tenancy boundaries

### 4. Onboarding Acceleration
New team members can use AI agents as ABP tutors:
- "How do I create a new module in ABP?"
- "What's the difference between ISettingProvider and ISettingManager?"
- "How does the distributed event bus work?"

The AI answers with ABP-specific examples, not generic .NET patterns.

## Behind the Scenes

### Methodology

Every skill file is built from **official ABP Framework v10.4 documentation**. I reviewed:

- 36 module documentation files
- 8 framework subdirectories (api-development, architecture, data, fundamentals, infrastructure, real-time, ui)
- 7 tutorials (book-store, todo, modular-crm, microservice, mobile)
- 6 solution templates (single-layer, layered, microservice, MAUI, WPF, console, empty)
- 10 get-started guides

### Design Decisions

**Two versions per topic:** Claude Code benefits from detailed explanations (the model has a large context window), while OpenCode's architecture prefers compact quick-reference files.

**Trigger-based activation:** Skills should activate naturally when the user mentions relevant topics — no manual loading required.

**Production-ready examples:** Every code snippet is copy-paste ready and follows ABP conventions exactly.

**Cross-references:** Skills link to each other and to official ABP docs for deeper exploration.

## Contributing

The repository is open source under the MIT License. Contributions are welcome:

- **New skills:** Missing a topic? Open a [skill request issue](https://github.com/burakdmir/abp-skills/issues/new?template=skill-request.md)
- **Updates:** Found outdated information? Submit a PR
- **Bug reports:** Incorrect code examples? File a [bug report](https://github.com/burakdmir/abp-skills/issues/new?template=bug-report.md)

See [CONTRIBUTING.md](https://github.com/burakdmir/abp-skills/blob/main/CONTRIBUTING.md) for guidelines.

## What's Next

The ABP ecosystem is evolving rapidly. Planned additions:

- **ABP Studio integration** skills for AI-assisted development workflows
- **LeptonX theme** customization skills
- **Testing** skills (unit, integration, E2E with ABP patterns)
- **Deployment** skills (Docker, Kubernetes, Azure, AWS)
- **Module-specific** skills for Identity, Saas, CMS Kit, and other pre-built modules

## Conclusion

AI coding agents are only as good as the context they have. By providing structured, framework-specific knowledge through skill files, we can transform generic AI output into production-quality ABP code.

This repository is a starting point — a proof that framework-specific AI skills work, and a foundation for the community to build upon.

**Star the repo, contribute, and let's make AI-assisted ABP development the norm, not the exception.**

🔗 **GitHub:** [github.com/burakdmir/abp-skills](https://github.com/burakdmir/abp-skills)

---

*Burak Demir — .NET Developer & ABP Framework Enthusiast*

---
name: abp-dependency-rules
description: "ABP Framework v10.4 layer dependency rules: layer direction (Domain.Shared→Domain→Application.Contracts→Application→HttpApi→Host), project reference matrix, anti-patterns (no DbContext in Application, don't expose IQueryable, don't return entity as DTO). Use when you need project/layer architecture or dependency rules in ABP."
---

# ABP Framework — Dependency Rules

ABP Framework v10.4 layer dependency rules and project-structure guardrails. Correct dependency direction, use of abstractions, and common violations.

## Trigger

- "ABP layer architecture"
- "ABP dependency rules"
- "ABP layer dependency"
- "ABP project references"
- "ABP where is DbContext"
- "ABP architecture guardrail"

## Core Principles (All Templates)

1. **Domain logic never depends on infrastructure** (no DbContext in Domain/Application)
2. **Use abstractions (interfaces) for dependencies**
3. **An upper layer depends on a lower layer**, never the reverse
4. **Data access goes through the repository**, not directly through DbContext

## Layered Template Structure

```
Domain.Shared    → Constants, enums, localization keys
       ↑
    Domain       → Entity, repository interface, domain service
       ↑
Application.Contracts → App service interface, DTO
       ↑
  Application    → App service implementation
       ↑
   HttpApi       → REST controller (optional)
       ↑
     Host        → Final application with DI + middleware
```

### Reference Matrix

| Project | Can reference | Referenced by |
|---|---|---|
| Domain.Shared | (none) | All |
| Domain | Domain.Shared | Application, Data layer |
| Application.Contracts | Domain.Shared | Application, HttpApi, Clients |
| Application | Domain, Contracts | Host |
| EntityFrameworkCore / MongoDB | Domain | Host only |
| HttpApi | Contracts only | Host |

## ❌ Never Do

```csharp
// DbContext directly in the Application layer
public class BookAppService : ApplicationService
{
    private readonly MyDbContext _dbContext; // ❌ WRONG — use a repository
}

// Domain depending on application
public class BookManager : DomainService
{
    private readonly IBookAppService _appService; // ❌ WRONG
}

// HttpApi depending on the Application implementation
public class BookController : AbpController
{
    private readonly BookAppService _bookAppService; // ❌ WRONG — use the interface
}
```

## ✅ Always Do

```csharp
public class BookAppService : ApplicationService
{
    private readonly IBookRepository _bookRepository; // ✅ repository abstraction
}

public class BookController : AbpController
{
    private readonly IBookAppService _bookAppService; // ✅ contract (interface)
}
```

## Repository Location

```csharp
// Interface → Domain project
public interface IBookRepository : IRepository<Book, Guid>
{
    Task<Book> FindByNameAsync(string name);
}

// Implementation → EntityFrameworkCore project
public class BookRepository : EfCoreRepository<MyDbContext, Book, Guid>, IBookRepository { }

// or MongoDB project
public class BookRepository : MongoDbRepository<MyDbContext, Book, Guid>, IBookRepository { }
```

## Multiple Applications Scenario (Admin + Public)

```
MyProject.Admin.Application    — Admin-specific services
MyProject.Public.Application   — Public-specific services
MyProject.Domain               — Shared domain (both reference it)
```
- The Admin and Public application layers **DO NOT reference each other**
- Domain logic is shared, application logic is not
- Each vertical can have its own DTO (even if similar)

## Common Violations

| Violation | Impact | Solution |
|---|---|---|
| DbContext in Application | Breaks DB independence | Use a repository |
| Returning entity as DTO | Exposes internal structure | Map to a DTO |
| `IQueryable` in interface | Breaks the abstraction | Return a concrete type |
| App service call across modules | Tight coupling | Use an event or domain |

## Best Practices

1. **Stick to the dependency direction** — top to bottom, never the reverse
2. **No DbContext in Domain/Application** — only the repository abstraction
3. **Repository interface in Domain, implementation in the Data layer**
4. **Don't expose `IQueryable`** — return a concrete type
5. **Don't let entities cross the boundary** — always a DTO
6. **Use events/domain for cross-module communication** — not a direct app service call

## Related

- [Framework Core](../abp-framework/SKILL.md) — layer/template overview
- [DDD](../abp-ddd/SKILL.md) — domain/application design
- [EF Core](../abp-efcore/SKILL.md) — repository implementation
- [Modularity](../abp-modularity/SKILL.md) — module dependencies
- [Development Flow](../abp-development-flow/SKILL.md) — which code in which layer
- ABP Docs: https://abp.io/docs/latest/framework/architecture/best-practices

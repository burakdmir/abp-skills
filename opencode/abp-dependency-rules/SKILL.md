---
name: abp-dependency-rules
description: "ABP Framework v10.4 layer dependency rules quick reference: layer direction, project reference matrix, anti-patterns (no DbContext in Application, don't expose IQueryable, don't return entity as DTO). Use when you need layer architecture/dependency rules in ABP."
---

# ABP Framework — Dependency Rules

ABP v10.4 layer dependency guardrails.

## Trigger

"ABP layer architecture", "dependency rules", "layer dependency", "project references", "where is DbContext".

## Principles

1. Domain logic does not depend on infrastructure (no DbContext in Domain/Application)
2. Use abstractions (interfaces)
3. Upper layer → lower layer (never the reverse)
4. Data access through the repository

## Layers & References

```
Domain.Shared → Domain → Application.Contracts → Application → HttpApi → Host
                  ↑ EntityFrameworkCore/MongoDB (only Host references it)
```

| Project | References |
|---|---|
| Domain.Shared | — |
| Domain | Domain.Shared |
| Application.Contracts | Domain.Shared |
| Application | Domain, Contracts |
| EF Core/MongoDB | Domain |
| HttpApi | Contracts |

## ❌ / ✅

```csharp
// ❌ DbContext in Application / ❌ concrete AppService in HttpApi / ❌ Domain → AppService
private readonly MyDbContext _db;            // ❌
private readonly BookAppService _svc;        // ❌ use the interface

// ✅
private readonly IBookRepository _repo;      // App/Domain
private readonly IBookAppService _svc;       // Controller
```

Repository: interface → Domain, impl → `EfCoreRepository<...>` / `MongoDbRepository<...>` (Data layer).

## Common Violations

| Violation | Solution |
|---|---|
| DbContext in Application | Repository |
| Returning entity as DTO | Map to a DTO |
| `IQueryable` in interface | Return a concrete type |
| App service call across modules | Event/domain |

## Related

- [Framework](../abp-framework/SKILL.md) · [DDD](../abp-ddd/SKILL.md) · [EF Core](../abp-efcore/SKILL.md) · [Modularity](../abp-modularity/SKILL.md) · [Development Flow](../abp-development-flow/SKILL.md)
- ABP Docs: https://abp.io/docs/latest/framework/architecture/best-practices

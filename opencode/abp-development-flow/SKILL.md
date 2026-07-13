---
name: abp-development-flow
description: "ABP Framework v10.x (10.4/10.5) development flow quick reference: adding an entity end-to-end (Domain→Shared→repo→EF config→migration→Contracts/DTO→mapping→app service→localization→permission→test). Use when you need the flow for adding a new feature in ABP."
---

# ABP Framework — Development Flow

Adding a new entity end-to-end in the ABP v10.x layered template.

## Trigger

"ABP add new entity/feature", "end-to-end CRUD", "development flow", "how do I start".

## Flow

1. **Domain/Entities** — `Book : AggregateRoot<Guid>` (private setter, constructor invariant, `Check.NotNullOrWhiteSpace`).
2. **Domain.Shared** — `BookConsts`, enum.
3. **Domain** (opt.) — custom `IBookRepository : IRepository<Book,Guid>` only if a custom query is needed.
4. **EntityFrameworkCore** — `DbSet<Book>` + `OnModelCreating`:
   ```csharp
   builder.Entity<Book>(b => { b.ToTable(Prefix+"Books"); b.ConfigureByConvention(); b.Property(x=>x.Name).IsRequired().HasMaxLength(128); });
   ```
5. **Migration**:
   ```bash
   cd src/MyProject.EntityFrameworkCore
   dotnet ef migrations add Added_Book
   dotnet run --project ../MyProject.DbMigrator   # apply + seed
   ```
6. **Application.Contracts** — `BookDto : EntityDto<Guid>`, `CreateBookDto` ([Required]/[Range]), `IBookAppService : IApplicationService`.
7. **Mapping** — Mapperly `[Mapper] partial class BookMapper : MapperBase<Book, BookDto>`.
8. **Application** — `BookAppService : ApplicationService, IBookAppService`; `[Authorize(...)]`, `GuidGenerator.Create()`, `ObjectMapper.Map<>()`.
9. **Localization** — `*.Domain.Shared/Localization/*/en.json`.
10. **Permission** — `MyProjectPermissions.Books.Create` (Application.Contracts).
11. **Test** — `*ApplicationTestBase`, `GetRequiredService<IBookAppService>()`.

## Commands

```bash
dotnet build
dotnet ef migrations add Name        # in the EntityFrameworkCore project
dotnet run --project ../MyProject.DbMigrator
abp generate-proxy -t ng
```

## Checklist

Entity → Shared → (repo) → EF config + ConfigureByConvention → migration/DbMigrator → DTO+interface → Mapperly → service+authorize → localization → permission → test.

## Related

- [DDD](../abp-ddd/SKILL.md) · [EF Core](../abp-efcore/SKILL.md) · [Object Mapping](../abp-object-mapping/SKILL.md) · [Authorization](../abp-authorization/SKILL.md) · [Testing](../abp-testing/SKILL.md) · [Dependency Rules](../abp-dependency-rules/SKILL.md)
- ABP Docs: https://abp.io/docs/latest/tutorials

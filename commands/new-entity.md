---
description: Scaffold a complete ABP DDD entity end-to-end (domain → contracts → application service → EF Core mapping + migration → permissions → integration test) for ABP v10.4 / .NET 10.
---

# Scaffold an ABP entity end-to-end

Target entity and details: **$ARGUMENTS**

Generate a full, ABP v10.4-idiomatic vertical slice for this entity. Follow the `abp-development-flow`, `abp-ddd`, `abp-efcore`, and `abp-authorization` skills. Before writing anything, inspect the current solution to detect its structure (Layered / Modular Monolith / Microservice), ORM (EF Core vs MongoDB), and naming conventions, then match them exactly.

Produce, in order:

1. **Domain** — the aggregate root / entity in the `*.Domain` project: private setters, constructor + behavior methods that enforce invariants, `GuidGenerator`-friendly key, domain events if state changes warrant them. Add a repository interface if the entity needs custom queries.
2. **Domain.Shared** — enums / consts / error codes.
3. **Application.Contracts** — DTOs (`...Dto`, `CreateUpdate...Dto`), the app service interface, and permission constants in the `*Permissions` class + `*PermissionDefinitionProvider`.
4. **Application** — the app service (extend `CrudAppService`/`ApplicationService` as appropriate), object mapping (Mapperly by default — see `abp-object-mapping`), `[Authorize]` with the new permissions.
5. **EntityFrameworkCore** (or MongoDB) — `DbSet`, `ConfigureByConvention()` + property mapping in `OnModelCreating`, then the EF Core migration command to run.
6. **Test** — an integration test (`*TestBase`, SQLite in-memory, Shouldly) covering create + a key invariant.

Do not invent APIs. If a step requires a decision (e.g. aggregate boundary, soft-delete, multi-tenancy), state the assumption and proceed with the ABP-default choice. End with the exact `abp`/`dotnet ef` commands to run and how to execute the test.

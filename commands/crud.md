---
description: Generate an ABP CrudAppService (or ICrudAppService implementation) with DTOs and permissions for an existing aggregate, ABP v10.4 style.
---

# Generate a CRUD application service

Aggregate / requirements: **$ARGUMENTS**

For the given aggregate root, generate a complete CRUD application layer following the `abp-ddd`, `abp-api`, and `abp-object-mapping` skills. First locate the entity and confirm its key type and properties.

Produce:

1. **DTOs** in `Application.Contracts`: `EntityDto`, `CreateUpdateEntityDto` (with Data Annotations), and a paged/filtered `GetListInput` (extend `PagedAndSortedResultRequestDto`) if listing needs filters.
2. **Service interface** extending `ICrudAppService<...>` (or `IApplicationService` with explicit methods when the CRUD surface is partial).
3. **Implementation** extending `CrudAppService<TEntity, TDto, TKey, TGetListInput, TCreateUpdateInput>` with `Get`/`Create`/`Update`/`Delete` permission policies wired to the entity's `*Permissions` constants. Override `CreateFilteredQueryAsync` if a list filter was requested.
4. **Object mapping** — Mapperly mapper (default) or AutoMapper profile, matching whatever the solution already uses.
5. **Permissions** — ensure the CRUD permissions exist in the `*Permissions` class and `*PermissionDefinitionProvider`; add any that are missing.

Auto-generated dynamic API controllers expose this service over HTTP — note the resulting route. Don't invent APIs; match the existing solution's conventions and key type.

---
description: Review code or the current diff against ABP Framework v10.4 best practices — layering, dependency rules, DDD, async, security, and deprecated APIs.
---

# ABP best-practices review

Scope (optional — file paths, a glob, or empty for the working diff): **$ARGUMENTS**

If no scope is given, review the current working changes (`git diff` and staged changes). Otherwise review the named files/paths.

Audit strictly against ABP v10.4 conventions (lean on `abp-dependency-rules`, `abp-ddd`, `abp-efcore`, `abp-authorization`, `abp-validation`). Check for:

- **Layer violations** — Domain/Application referencing EF Core; repositories exposing `IQueryable` across boundaries; controllers holding business logic.
- **Anemic domain** — public setters, logic that belongs in the aggregate leaking into app services; missing invariant enforcement.
- **Deprecated / wrong APIs** — `ConfigureIdentityServer` (use `ConfigureOpenIddict`), `DateTime.Now` (use `IClock`), `.Result`/`.Wait()` (async all the way), manual `new Guid()` (use `GuidGenerator`).
- **Authorization gaps** — app service methods without `[Authorize]` / permission checks; permissions not defined in a `PermissionDefinitionProvider`.
- **DTO/validation** — entities returned directly instead of DTOs; missing Data Annotations / FluentValidation; missing `ConfigureByConvention()` in mappings.
- **Multi-tenancy** — `IMultiTenant` entities missing `TenantId`; cross-tenant queries without `ICurrentTenant.Change` or `IDataFilter`.

Output one finding per line as `path:line — <severity> — <problem>. <fix>.` (severity: 🔴 high / 🟠 medium / 🟡 low). No praise, no restating correct code. End with a one-line verdict. If clean, say so.

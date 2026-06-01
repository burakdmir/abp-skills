---
name: abp-framework
description: "ABP Framework v10.4 core quick reference: solution templates, layered architecture, module system, base classes, Clock/GuidGenerator/CurrentUser/LazyServiceProvider, .NET 10. Use when creating an ABP project, or when architecture or core conventions are needed."
---

# ABP Framework — Core Skill

ABP Framework v10.4 (.NET 10) core reference. Opinionated, DDD-based, modular ASP.NET Core.

## Trigger

"create ABP project", "ABP solution/template", "ABP module", "ABP architecture", "abp new", ABP best practices.

## Solution Templates

| Template | Description |
|---|---|
| `app` | Layered (default) |
| `app-nolayers` | Single-layer |
| `microservice` | Microservice (Business+ license) |
| `empty` | Empty |

**Modern Monolith / Modular Monolith** can also be selected as a separate path in the ABP Studio wizard.

**UI:** `mvc`, `angular`, `blazor-webapp`, `blazor` (WASM), `blazor-server`, `react` (`--modern`), `no-ui`
**DB:** `ef` (default), `mongodb`

## CLI (summary)

```bash
dotnet tool install -g Volo.Abp.Studio.Cli
abp new Acme.BookStore --template app                 # classic
abp new Acme.BookStore --template app --modern        # React-first modern
abp new Acme.BookStore --template app --modern --modular  # modular monolith
abp add-package Volo.Abp.EntityFrameworkCore
abp generate-proxy -t ng        # ng | csharp | js
abp update
```
> For details see [CLI skill](../abp-cli/SKILL.md).

## Layer Structure (Layered)

```
Domain.Shared → Domain → Application.Contracts → Application → HttpApi → Web/Host
                  ↑ EntityFrameworkCore/MongoDB (only Host references it)
```
> For dependency rules see [Dependency Rules skill](../abp-dependency-rules/SKILL.md).

## Module Class

```csharp
[DependsOn(typeof(AbpAspNetCoreMvcModule), typeof(AbpEntityFrameworkCoreModule), typeof(AbpAutofacModule))]
public class BookStoreModule : AbpModule
{
    public override void ConfigureServices(ServiceConfigurationContext context) { }   // DI + config
    public override void OnApplicationInitialization(ApplicationInitializationContext context) { } // middleware (host only)
}
```
Lifecycle: `PreConfigureServices` → `ConfigureServices` → `PostConfigureServices` → `OnApplicationInitialization` (+ `*Async` versions).

## DI (summary)

```csharp
public class TaxCalculator : ITransientDependency { }   // ISingletonDependency | IScopedDependency
[Dependency(ServiceLifetime.Transient, ReplaceServices = true)] public class X { }
[ExposeServices(typeof(ITaxCalculator))] public class TaxCalculator : ITaxCalculator, ITransientDependency { }
```

## Base Class Pre-injected Services

`ApplicationService` / `DomainService` / `AbpController` provide these as properties — don't inject them:

| Property | Note |
|---|---|
| `GuidGenerator` | instead of `Guid.NewGuid()` |
| `Clock` | instead of `DateTime.Now` (test/timezone) |
| `CurrentUser` | user info |
| `CurrentTenant` | tenant context |
| `L` | localization (ApplicationService/AbpController) |
| `AuthorizationService` / `FeatureChecker` | permission/feature check |
| `LazyServiceProvider` | lazy resolution via `LazyGetRequiredService<T>()` |
| `UnitOfWorkManager` | UOW |

**Async all-the-way** — don't use `.Result`/`.Wait()`; methods should end with `Async`.

## Pre-built Modules

`Volo.Abp.Account`, `.Identity`, `.TenantManagement`, `.SettingManagement`, `.PermissionManagement`, `.FeatureManagement`, `.AuditLogging`, `.BackgroundJobs`, `.CmsKit`, `.OpenIddict` (auth server, replacing IdentityServer from v6.0+).

## Best Practices

1. Define module dependencies with `[DependsOn]`
2. `IGuidGenerator.Create()` (sequential GUID), `Clock` (time)
3. Don't expose entities — use DTOs
4. Mapperly (default in v10.4) object mapping
5. Inject `IRepository<TEntity, TKey>`, rely on UOW conventions

## Related

- [DDD](../abp-ddd/SKILL.md) · [Modularity](../abp-modularity/SKILL.md) · [Dependency Injection](../abp-dependency-injection/SKILL.md) · [CLI](../abp-cli/SKILL.md) · [Dependency Rules](../abp-dependency-rules/SKILL.md) · [Microservices](../abp-microservices/SKILL.md)
- ABP Docs: https://abp.io/docs/latest

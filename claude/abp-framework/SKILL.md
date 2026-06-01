---
name: abp-framework
description: "ABP Framework v10.4 core guide: solution templates (app, app-nolayers, microservice), layered architecture, module system, base classes (ApplicationService/DomainService), Clock/GuidGenerator/CurrentUser/LazyServiceProvider, .NET 10. Use when creating an ABP project, or when architecture or core conventions are needed."
---

# ABP Framework — Core Skill

Core development skill for ABP Framework v10.4. A guide to opinionated, DDD-based, modular ASP.NET Core application development.

## Trigger

- "create a project with ABP"
- "ABP solution/template"
- "ABP module"
- "ABP best practices"
- "ABP architecture"
- "abp new"
- working on an ABP project

## What Is ABP

ABP is a modular framework built on .NET and ASP.NET Core that offers an **opinionated architecture** based on DDD principles. It automates repetitive work and provides production-ready startup templates, pre-built application modules, and tooling.

## Solution Templates

| Template | Description |
|---|---|
| `app` | Layered web application (default) |
| `app-nolayers` | Single-layer web application |
| `microservice` | Microservice solution (Business+ license) |
| `empty` | Empty solution |

### UI Framework Options

- `mvc` — ASP.NET Core MVC / Razor Pages
- `angular` — Angular SPA
- `blazor-webapp` — Blazor Web App
- `blazor` — Blazor WASM
- `blazor-server` — Blazor Server
- `react` — React SPA (with the modern template)
- `no-ui` — Without a frontend

### Database Providers

- `ef` — Entity Framework Core (default)
- `mongodb` — MongoDB

## ABP CLI Commands

```bash
# Installation
dotnet tool install -g Volo.Abp.Studio.Cli

# Create a new solution
abp new Acme.BookStore --template app
abp new Acme.BookStore --template app --modern          # React-first modern template
abp new Acme.BookStore --template app-nolayers --modern # Single-layer modern
abp new Acme.BookStore --template microservice --modern # Microservice modern

# Create a module
abp new-module Acme.BookStore.Orders -t module:ddd
abp new-module Acme.BookStore.Orders --modern

# Add a package
abp add-package Volo.Abp.EntityFrameworkCore

# Update packages
abp update

# Generate proxy (Angular/C#/JS)
abp generate-proxy -t ng
abp generate-proxy -t csharp

# Download source code
abp get-source Volo.Blogging
abp add-source-code Volo.Chat
```

### Modern Templates (`--modern` flag)

Modern templates are React-first and use a different template source shipped with ABP Studio.

```bash
abp new Acme.BookStore --template app --modern
abp new Acme.BookStore --template app-nolayers --modern --modular  # Modular monolith
abp new Acme.BookStore --template microservice --modern --services Ordering,Shipping
abp new Acme.BookStore --template microservice --modern --shadcn-theme blue
```

**Shadcn theme values:** `slate` (default), `pink`, `blue`, `turquoise`, `orange`, `purple`

## Project Structure (Layered Template)

```
Acme.BookStore/
├── src/
│   ├── Acme.BookStore.Domain.Shared    # Constants, enums, localization
│   ├── Acme.BookStore.Domain           # Entities, aggregates, domain services, repositories (interface)
│   ├── Acme.BookStore.Application.Contracts  # DTOs, application service interfaces
│   ├── Acme.BookStore.Application      # Application services, object mapping
│   ├── Acme.BookStore.EntityFrameworkCore  # DbContext, repository implementations, migrations
│   ├── Acme.BookStore.HttpApi          # API controllers
│   ├── Acme.BookStore.HttpApi.Client   # Dynamic C# HTTP clients
│   └── Acme.BookStore.Web              # MVC/Razor Pages UI
└── test/
    ├── Acme.BookStore.TestBase
    ├── Acme.BookStore.Domain.Tests
    ├── Acme.BookStore.Application.Tests
    └── Acme.BookStore.Web.Tests
```

## Module Class Structure

Each module defines a derivative of `AbpModule`:

```csharp
[DependsOn(
    typeof(AbpAspNetCoreMvcModule),
    typeof(AbpEntityFrameworkCoreModule),
    typeof(AbpAutofacModule)
)]
public class BookStoreModule : AbpModule
{
    public override void PreConfigureServices(ServiceConfigurationContext context) { }
    
    public override void ConfigureServices(ServiceConfigurationContext context)
    {
        // DI registration, module configuration
    }
    
    public override void OnApplicationInitialization(ApplicationInitializationContext context)
    {
        // Middleware pipeline, startup logic
    }
    
    public override void OnApplicationShutdown(ApplicationShutdownContext context) { }
}
```

### Lifecycle Methods

| Method | Description | Async Version |
|---|---|---|
| `PreConfigureServices` | Runs before all ConfigureServices | `PreConfigureServicesAsync` |
| `ConfigureServices` | DI registration and module configuration | `ConfigureServicesAsync` |
| `PostConfigureServices` | Runs after all ConfigureServices | `PostConfigureServicesAsync` |
| `OnPreApplicationInitialization` | Before init | `OnPreApplicationInitializationAsync` |
| `OnApplicationInitialization` | Middleware setup | `OnApplicationInitializationAsync` |
| `OnPostApplicationInitialization` | After init | `OnPostApplicationInitializationAsync` |
| `OnApplicationShutdown` | Shutdown logic | `OnApplicationShutdownAsync` |

## Dependency Injection

### Automatic Registration Methods

```csharp
// 1. Interface implementation
public class TaxCalculator : ITransientDependency { }       // Transient
public class CacheService : ISingletonDependency { }        // Singleton
public class ScopedService : IScopedDependency { }          // Scoped

// 2. With an attribute
[Dependency(ServiceLifetime.Transient, ReplaceServices = true)]
public class TaxCalculator { }

// 3. Use ExposeServices to control which interfaces are exposed
[ExposeServices(typeof(ITaxCalculator))]
public class TaxCalculator : ICalculator, ITaxCalculator, ITransientDependency { }

// 4. Keyed Services
[ExposeKeyedService<ITaxCalculator>("taxCalculator")]
public class TaxCalculator : ITransientDependency { }
```

### Manual Registration

```csharp
public override void ConfigureServices(ServiceConfigurationContext context)
{
    context.Services.AddSingleton<TaxCalculator>(new TaxCalculator(0.18));
    context.Services.AddScoped<ITaxCalculator>(sp => sp.GetRequiredService<TaxCalculator>());
    
    // Replacing a service
    context.Services.Replace(ServiceDescriptor.Transient<IConnectionStringResolver, MyResolver>());
}
```

## Base Classes and Pre-injected Services

ABP base classes provide frequently used services ready as properties. **Before injecting a service, check whether the base class already has it.**

| Base Class | Purpose |
|---|---|
| `Entity<TKey>` / `AggregateRoot<TKey>` | Basic entity / DDD aggregate root |
| `DomainService` | Domain business logic |
| `ApplicationService` | Use-case orchestration |
| `AbpController` | REST API controller |

| Property | Available in | Description |
|---|---|---|
| `GuidGenerator` | All base classes | Generate sequential GUID (instead of `Guid.NewGuid()`) |
| `Clock` | All base classes | Current time (instead of `DateTime.Now`) |
| `CurrentUser` | All base classes | Authenticated user info |
| `CurrentTenant` | All base classes | Multi-tenancy context |
| `L` (StringLocalizer) | ApplicationService, AbpController | Localization |
| `AuthorizationService` | ApplicationService, AbpController | Permission check |
| `FeatureChecker` | ApplicationService, AbpController | Feature check |
| `LazyServiceProvider` | All base classes | Lazy service resolution |
| `UnitOfWorkManager` | ApplicationService, DomainService | UOW management |
| `Logger` | All base classes | Logging |

```csharp
public class BookAppService : ApplicationService
{
    public async Task DoAsync()
    {
        var now = Clock.Now;              // ✅ DO NOT use DateTime.Now (untestable, ignores timezone)
        var id = GuidGenerator.Create();  // ✅ DO NOT use Guid.NewGuid()
        var userId = CurrentUser.Id;
        await CheckPolicyAsync("BookStore.Books.Create");  // base class method
    }
}
```

For non-base-class services, inject the relevant abstraction: `IClock`, `ICurrentUser`, `IGuidGenerator`. For lazy resolution in a base class use `LazyServiceProvider.LazyGetRequiredService<T>()`; if it is to be injected, `ITransientCachedServiceProvider` is preferred (`IAbpLazyServiceProvider` for backward-compat).

### Async Convention

- **Async all-the-way** — **do not use** `.Result` / `.Wait()` (deadlock risk + untestable).
- Async methods should end with the `Async` suffix.
- ABP manages the `CancellationToken` automatically in most cases (e.g. `HttpContext.RequestAborted`); pass it explicitly only for custom cancellation logic.

### Time Management (IClock)

Do not use `DateTime.Now` / `DateTime.UtcNow` directly. Inject the `Clock` property (base class) or `IClock` — for testability and UTC/timezone settings. The `Kind` (Utc/Local/Unspecified) is configured via `AbpClockOptions`.

## Important Configuration Patterns

```csharp
// Options pattern
Configure<AbpDbConnectionOptions>(options =>
{
    options.ConnectionStrings.Default = "...";
});

// Module configuration
Configure<AbpMultiTenancyOptions>(options =>
{
    options.IsEnabled = true;
});
```

## Pre-built Application Modules

| Module | Description |
|---|---|
| `Volo.Abp.Account` | Account management |
| `Volo.Abp.Identity` | Identity & user management |
| `Volo.Abp.TenantManagement` | Tenant management (multi-tenancy) |
| `Volo.Abp.SettingManagement` | Setting management |
| `Volo.Abp.PermissionManagement` | Permission management |
| `Volo.Abp.FeatureManagement` | Feature management |
| `Volo.Abp.AuditLogging` | Audit logging |
| `Volo.Abp.BackgroundJobs` | Background job system |
| `Volo.Abp.CmsKit` | CMS kit (content management) |
| `Volo.Abp.Saas` | SaaS module (PRO) |
| `Volo.Abp.OpenIddict` | OpenIddict integration |

## Best Practices

1. **Always define module dependencies with `AbpModule`** — use the `[DependsOn]` attribute
2. **Use a GUID primary key** — generate a sequential GUID with `IGuidGenerator.Create()`
3. **Use DTOs** — do not expose entities to the presentation layer
4. **Use Mapperly** — for object-to-object mapping (the default in ABP 10.4)
5. **Use the repository pattern** — inject `IRepository<TEntity, TKey>`
6. **Rely on Unit of Work conventions** — no need for manual UOW management
7. **Prefer async methods** — write scalable code with `async/await`
8. **Keep the domain layer isolated from the database provider** — use `IAsyncQueryableExecuter`

## Resources

- ABP Docs: https://abp.io/docs/latest
- ABP CLI: `abp help`
- ABP GitHub: https://github.com/abpframework/abp
- ABP Community: https://abp.io/community/

## Related

- [DDD](../abp-ddd/SKILL.md) — entity, aggregate, repository, application service
- [Modularity](../abp-modularity/SKILL.md) — module system, [DependsOn]
- [Dependency Injection](../abp-dependency-injection/SKILL.md) — DI, ITransientDependency, LazyServiceProvider
- [Dependency Rules](../abp-dependency-rules/SKILL.md) — layer dependency rules
- [CLI](../abp-cli/SKILL.md) — abp new, ABP Studio, --modern
- [Development Flow](../abp-development-flow/SKILL.md) — flow for adding a new feature
- [Microservices](../abp-microservices/SKILL.md) — microservice solution template

---
name: abp-dependency-injection
description: "ABP Framework v10.4 dependency injection: ITransientDependency/IScopedDependency/ISingletonDependency, [Dependency], [ExposeServices], LazyServiceProvider, property injection, Autofac. Use when you need service registration, DI, or automatic registration in ABP."
---

# ABP Dependency Injection Skill

## Trigger
User asks about dependency injection, DI, ITransientDependency, ISingletonDependency, IScopedDependency, [Dependency] attribute, [ExposeServices] attribute, auto service registration, or Autofac in ABP Framework.

---

## Core Concepts

ABP's DI system is built on Microsoft's `Microsoft.Extensions.DependencyInjection` with:
- **Conventional (automatic) registration** — Classes registered by convention
- **Dependency interfaces** — `ITransientDependency`, `ISingletonDependency`, `IScopedDependency`
- **Dependency attribute** — `[Dependency]` for fine-grained control
- **ExposeServices attribute** — `[ExposeServices]` to control exposed interfaces
- **Autofac integration** — Required for dynamic proxying (included in startup templates)

---

## Conventional Registration

ABP automatically registers all services in your assembly. No manual registration needed.

```csharp
// This class is auto-registered as transient
public class TaxCalculator : ITransientDependency
{
    public decimal Calculate(TaxInput input) { ... }
}
```

### Disable Auto Registration

```csharp
public class BlogModule : AbpModule
{
    public BlogModule()
    {
        SkipAutoServiceRegistration = true;
    }

    public override void ConfigureServices(ServiceConfigurationContext context)
    {
        // Manual registration
        context.Services.AddAssemblyOf<BlogModule>();
    }
}
```

---

## Dependency Interfaces

Implement these interfaces for automatic registration:

| Interface | Lifetime | Use Case |
|---|---|---|
| `ITransientDependency` | Transient | Default for most services |
| `ISingletonDependency` | Singleton | Shared state, caches |
| `IScopedDependency` | Scoped | Per-request state |

```csharp
public class TaxCalculator : ITransientDependency { }
public class CacheService : ISingletonDependency { }
public class RequestTracker : IScopedDependency { }
```

---

## Dependency Attribute

More control over registration:

```csharp
[Dependency(ServiceLifetime.Transient, ReplaceServices = true)]
public class TaxCalculator
{
}
```

### Properties

| Property | Type | Purpose |
|---|---|---|
| `Lifetime` | ServiceLifetime | Transient, Singleton, or Scoped |
| `TryRegister` | bool | Register only if not already registered |
| `ReplaceServices` | bool | Replace existing registration |

```csharp
[Dependency(ServiceLifetime.Singleton, TryRegister = true)]
public class MyCacheService { }

[Dependency(ServiceLifetime.Transient, ReplaceServices = true)]
public class OverridingService { }
```

> `[Dependency]` has higher priority than dependency interfaces if `Lifetime` is defined.

---

## ExposeServices Attribute

Control which interfaces a class exposes:

```csharp
[ExposeServices(typeof(ITaxCalculator))]
public class TaxCalculator : ICalculator, ITaxCalculator, ICanCalculate, ITransientDependency
{
}
```

- Only `ITaxCalculator` can be injected
- `TaxCalculator`, `ICalculator`, `ICanCalculate` are NOT injectable

### Expose Multiple Services

```csharp
[ExposeServices(typeof(ITaxCalculator), typeof(ICalculator))]
public class TaxCalculator : ICalculator, ITaxCalculator, ITransientDependency
{
}
```

### Expose All (default behavior)

Without `[ExposeServices]`, all implemented interfaces are exposed:

```csharp
public class TaxCalculator : ICalculator, ITaxCalculator, ITransientDependency
{
}
// Can inject: TaxCalculator, ICalculator, ITaxCalculator
```

---

## Combining Attributes and Interfaces

```csharp
[Dependency(ReplaceServices = true)]
[ExposeServices(typeof(ITaxCalculator))]
public class TaxCalculator : ITaxCalculator, ITransientDependency
{
}
```

- `ReplaceServices = true` → replaces existing `ITaxCalculator` registration
- `ExposeServices` → only `ITaxCalculator` can be injected
- `ITransientDependency` → transient lifetime (overridden by `[Dependency]` if specified)

---

## Inherently Registered Types

These types are automatically registered by ABP:
- `AbpModule` implementations
- Controllers, PageModels
- Domain services, Application services
- Repositories (via EF Core / MongoDB modules)

---

## Autofac Integration

ABP requires a DI provider that supports **dynamic proxying** for:
- Unit of Work interception
- Validation interception
- Authorization interception
- Auditing interception
- Feature checking

Startup templates come with Autofac pre-installed.

### Manual Autofac Setup

```bash
dotnet add package Volo.Abp.Autofac
```

```csharp
[DependsOn(typeof(AbpAutofacModule))]
public class MyModule : AbpModule { }
```

In `Program.cs`:
```csharp
builder.Host.UseAutofac();
```

---

## Property Injection

ABP supports property injection via Autofac:

```csharp
public class MyService : ITransientDependency
{
    public IEmailSender EmailSender { get; set; } // Property injected

    private readonly IRepository<MyEntity> _repository; // Constructor injected

    public MyService(IRepository<MyEntity> repository)
    {
        _repository = repository;
    }
}
```

> Constructor injection is preferred. Property injection is useful for optional dependencies.

---

## Resolving Services

### Constructor Injection (Recommended)

```csharp
public class MyService : ITransientDependency
{
    private readonly IEmailSender _emailSender;

    public MyService(IEmailSender emailSender)
    {
        _emailSender = emailSender;
    }
}
```

### Property Injection

```csharp
public class MyService : ITransientDependency
{
    public IEmailSender EmailSender { get; set; }
}
```

### Manual Resolution (Avoid When Possible)

```csharp
public class MyService : ITransientDependency
{
    private readonly IServiceProvider _serviceProvider;

    public MyService(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }

    public void DoWork()
    {
        var emailSender = _serviceProvider.GetRequiredService<IEmailSender>();
    }
}
```

### Lazy Service Resolution (LazyServiceProvider)

Instead of manual resolution via `IServiceProvider`, ABP's lazy service resolution is preferred. ABP base classes (`ApplicationService`, `DomainService`, `AbpController`) come with a `LazyServiceProvider` property out of the box — it prevents constructor bloat when you have optional or numerous dependencies:

```csharp
public class BookAppService : ApplicationService
{
    // No need to inject — comes from the base class
    private IEmailSender EmailSender => LazyServiceProvider.LazyGetRequiredService<IEmailSender>();

    public async Task NotifyAsync()
    {
        await EmailSender.SendAsync("to@x.com", "Subject", "Body");
    }
}
```

In non-base-class services, inject `ITransientCachedServiceProvider` (the old `IAbpLazyServiceProvider` is for backward compatibility and may be removed in the future):

```csharp
public class MyService : ITransientDependency
{
    private readonly ITransientCachedServiceProvider _serviceProvider;
    public MyService(ITransientCachedServiceProvider serviceProvider)
        => _serviceProvider = serviceProvider;

    private IEmailSender EmailSender => _serviceProvider.GetRequiredService<IEmailSender>();
}
```

---

## Best Practices

1. **Use conventional registration** — implement `ITransientDependency` (default)
2. **Prefer constructor injection** — explicit dependencies, easier to test
3. **Use `[Dependency(ReplaceServices = true)]`** to override framework services
4. **Use `[ExposeServices]`** to limit exposed interfaces
5. **Use `IScopedDependency`** for per-request state (HTTP context, etc.)
6. **Use `ISingletonDependency`** only for stateless services or caches
7. **Don't use service locator pattern** — avoid `IServiceProvider` resolution
8. **Keep services focused** — single responsibility, small interfaces
9. **Use Autofac** — required for dynamic proxying (interceptors, UOW, validation)
10. **Skip auto-registration** only when you need full control over DI setup

---

## Common Patterns

### Repository Injection

```csharp
public class BookAppService : ApplicationService, IBookAppService
{
    private readonly IRepository<Book, Guid> _bookRepository;

    public BookAppService(IRepository<Book, Guid> bookRepository)
    {
        _bookRepository = bookRepository;
    }
}
```

### Domain Service

```csharp
public class BookManager : DomainService, ITransientDependency
{
    private readonly IRepository<Book, Guid> _bookRepository;

    public BookManager(IRepository<Book, Guid> bookRepository)
    {
        _bookRepository = bookRepository;
    }

    public async Task<Book> CreateAsync(CreateBookDto input)
    {
        var book = new Book { Name = input.Name };
        return await _bookRepository.InsertAsync(book);
    }
}
```

### Override Framework Service

```csharp
[Dependency(ReplaceServices = true)]
public class MyCustomEmailSender : IEmailSender, ITransientDependency
{
    public async Task SendAsync(string to, string subject, string body)
    {
        // Custom email logic
    }
}
```

---

## Related

- [Framework Core](../abp-framework/SKILL.md) — base classes, module system, configuration
- [Modularity](../abp-modularity/SKILL.md) — module-based DI registration, [DependsOn]
- [DDD](../abp-ddd/SKILL.md) — application/domain service registration
- [Autofac](https://autofac.org/) — DI container with dynamic proxying support
- ABP Docs: https://abp.io/docs/latest/framework/fundamentals/dependency-injection

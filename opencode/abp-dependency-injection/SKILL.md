---
name: abp-dependency-injection
description: "ABP Framework v10.4 dependency injection: ITransientDependency/IScopedDependency/ISingletonDependency, [Dependency], [ExposeServices], LazyServiceProvider, property injection, Autofac. Use when you need service registration, DI, or automatic registration in ABP."
---

# ABP Dependency Injection Skill

## Trigger
Dependency injection, DI, ITransientDependency, ISingletonDependency, IScopedDependency, [Dependency], [ExposeServices], auto registration, Autofac.

---

## Quick Reference

### Conventional Registration (auto)
```csharp
public class TaxCalculator : ITransientDependency { }  // Transient
public class CacheService : ISingletonDependency { }   // Singleton
public class RequestTracker : IScopedDependency { }    // Scoped
```

### [Dependency] Attribute
```csharp
[Dependency(ServiceLifetime.Transient, ReplaceServices = true)]
public class TaxCalculator { }

[Dependency(ServiceLifetime.Singleton, TryRegister = true)]
public class MyCacheService { }
```
Properties: `Lifetime`, `TryRegister`, `ReplaceServices`
Higher priority than dependency interfaces.

### [ExposeServices] Attribute
```csharp
[ExposeServices(typeof(ITaxCalculator))]
public class TaxCalculator : ICalculator, ITaxCalculator, ITransientDependency { }
```
Only `ITaxCalculator` injectable. Without attribute, all interfaces exposed.

### Disable Auto Registration
```csharp
public class BlogModule : AbpModule
{
    public BlogModule() { SkipAutoServiceRegistration = true; }
    public override void ConfigureServices(ServiceConfigurationContext context)
    {
        context.Services.AddAssemblyOf<BlogModule>();
    }
}
```

### Autofac (required for dynamic proxying)
```bash
dotnet add package Volo.Abp.Autofac
```
```csharp
[DependsOn(typeof(AbpAutofacModule))]
public class MyModule : AbpModule { }
```
`Program.cs`: `builder.Host.UseAutofac();`

### Property Injection
```csharp
public class MyService : ITransientDependency
{
    public IEmailSender EmailSender { get; set; } // Optional dependency
}
```

### Override Framework Service
```csharp
[Dependency(ReplaceServices = true)]
public class MyCustomEmailSender : IEmailSender, ITransientDependency { }
```

### Best Practices
- ITransientDependency for most services
- Constructor injection preferred
- [Dependency(ReplaceServices=true)] to override framework services
- [ExposeServices] to limit exposed interfaces
- ISingletonDependency only for stateless/caches
- Don't use service locator pattern
- Autofac required for interceptors (UOW, validation, auth, audit)

## Related

[Framework](../abp-framework/SKILL.md) · [Modularity](../abp-modularity/SKILL.md) · [DDD](../abp-ddd/SKILL.md) · Docs: https://abp.io/docs/latest/framework/fundamentals/dependency-injection

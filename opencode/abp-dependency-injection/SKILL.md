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

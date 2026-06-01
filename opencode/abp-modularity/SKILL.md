---
name: abp-modularity
description: "ABP Framework v10.4 modularity: AbpModule, [DependsOn], module lifecycle, plugin modules, modular monolith. Use when creating a module, defining module dependencies, or building a modular architecture in ABP."
---

# ABP Framework — Modularity

A guide to modular application development in ABP Framework v10.4. The module system, dependency management, plugin modules, and best practices.

## Trigger

- "ABP modularity"
- "create an ABP module"
- "ABP DependsOn"
- "ABP plugin module"
- "ABP modular monolith"
- "ABP module dependency"

## What Is Modularity

ABP supports building fully modular applications and systems. Each module can contain its own entities, services, database integration, APIs, and UI components.

## Module Class

```csharp
[DependsOn(
    typeof(AbpAspNetCoreMvcModule),
    typeof(AbpEntityFrameworkCoreModule),
    typeof(AbpAutofacModule)
)]
public class BlogModule : AbpModule
{
    public override void PreConfigureServices(ServiceConfigurationContext context) { }
    
    public override void ConfigureServices(ServiceConfigurationContext context)
    {
        Configure<AbpDbConnectionOptions>(options =>
        {
            options.ConnectionStrings.Default = "...";
        });
    }
    
    public override void OnApplicationInitialization(ApplicationInitializationContext context)
    {
        var app = context.GetApplicationBuilder();
        var env = context.GetEnvironment();
        
        if (env.IsDevelopment())
            app.UseDeveloperExceptionPage();
        
        app.UseMvcWithDefaultRoute();
    }
    
    public override void OnApplicationShutdown(ApplicationShutdownContext context) { }
}
```

### Lifecycle Methods

| Method | When It Runs | Usage |
|---|---|---|
| `PreConfigureServices` | Before all `ConfigureServices` | Early configuration |
| `ConfigureServices` | Service registration | DI registration, module settings |
| `PostConfigureServices` | After all `ConfigureServices` | Late configuration |
| `OnPreApplicationInitialization` | Before init | Pre-init logic |
| `OnApplicationInitialization` | Application startup | Middleware pipeline |
| `OnPostApplicationInitialization` | After init | Post-init logic |
| `OnApplicationShutdown` | Application shutdown | Cleanup logic |

## Module Dependencies

```csharp
[DependsOn(typeof(AbpAspNetCoreMvcModule), typeof(AbpAutofacModule))]
public class BlogModule : AbpModule { }
```

At startup, ABP inspects the dependency graph and starts/shuts down modules in the correct order.

## Creating a Module with the CLI

```bash
abp new-module Acme.Blog -t module:ddd
abp new-module Acme.Blog --modern
abp new-module Acme.Blog -t module:ddd -ts Acme.Crm.sln
abp new-module Acme.Blog -t module:ddd -d ef -u mvc
```

## Installing a Module

```bash
abp install-module Volo.Blogging
abp install-local-module ../Acme.Blogging
abp add-package Volo.Abp.Blogging
```

## Module Extending

```csharp
ObjectExtensionManager.Instance
    .MapEfCoreProperty<IdentityUser, string>(
        "Title",
        (entityBuilder, propertyBuilder) =>
        {
            propertyBuilder.HasMaxLength(64);
        }
    );
```

## Modular Monolith

```bash
abp new Acme.Crm --template app-nolayers --modern --modular
```

## Related

[Framework](../abp-framework/SKILL.md) · [Dependency Injection](../abp-dependency-injection/SKILL.md) · [Dependency Rules](../abp-dependency-rules/SKILL.md) · [Microservices](../abp-microservices/SKILL.md) · Docs: https://abp.io/docs/latest/framework/architecture/modularity/basics

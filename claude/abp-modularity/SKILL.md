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
        // DI registration, module configuration
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

An `Async` version of each method is also available.

## Module Dependencies

```csharp
// Multiple within a single DependsOn
[DependsOn(typeof(AbpAspNetCoreMvcModule), typeof(AbpAutofacModule))]
public class BlogModule : AbpModule { }

// Multiple attributes
[DependsOn(typeof(AbpAspNetCoreMvcModule))]
[DependsOn(typeof(AbpAutofacModule))]
public class BlogModule : AbpModule { }
```

At startup, ABP inspects the dependency graph and starts/shuts down modules in the correct order.

## Additional Assembly

In rare cases, if your module consists of more than one assembly:

```csharp
[DependsOn(...)]
[AdditionalAssembly(typeof(BlogService))]  // A type from the target assembly
public class BlogModule : AbpModule { }
```

> **Warning:** Use `AdditionalAssembly` only when truly needed. Normally `DependsOn` should be preferred.

## Framework vs Application Modules

| Type | Description | Example |
|---|---|---|
| **Framework Module** | Infrastructure, integration, abstraction | Caching, EF Core, Validation, Logging |
| **Application Module** | Functional/business features | Blogging, Identity, Tenant Management |

## Plugin Modules

Modules that can be loaded dynamically at runtime:

```csharp
[DependsOn(typeof(AbpKernelModule))]
public class MyPluginModule : AbpModule { }
```

Plugin modules:
- Are not referenced at compile-time
- Are loaded at runtime from a specified directory
- Are used for hot-plug-like scenarios

## Module Development Best Practices

1. **Package according to DDD layers:**
   ```
   Acme.Blog/
   ├── Acme.Blog.Domain.Shared     # Constants, enums, localization
   ├── Acme.Blog.Domain            # Entities, repositories (interface)
   ├── Acme.Blog.Application.Contracts  # DTOs, service interfaces
   ├── Acme.Blog.Application       # Application services
   ├── Acme.Blog.EntityFrameworkCore  # DbContext, migrations
   ├── Acme.Blog.HttpApi           # API controllers
   └── Acme.Blog.HttpApi.Client    # Dynamic C# clients
   ```

2. **Design independently of the database provider** — don't make the Domain/Application layers depend on EF Core

3. **Each module can define its own connection string:**
   ```csharp
   Configure<AbpDbConnectionOptions>(options =>
   {
       options.ConnectionStrings["Blog"] = "...";
   });
   ```

4. **Use the Options pattern for module configuration:**
   ```csharp
   public class BlogOptions
   {
       public int MaxPostLength { get; set; } = 5000;
   }
   
   // Configure it in the Module
   Configure<BlogOptions>(options => options.MaxPostLength = 10000);
   ```

5. **Use `IRepository` for reusable modules** — don't use `DbContext` directly

## Creating a Module with the CLI

```bash
# DDD module
abp new-module Acme.Blog -t module:ddd

# Modern module
abp new-module Acme.Blog --modern

# Add to a specific solution
abp new-module Acme.Blog -t module:ddd -ts Acme.Crm.sln

# With EF + MVC support
abp new-module Acme.Blog -t module:ddd -d ef -u mvc
```

## Installing a Module

```bash
# Install a NuGet module
abp install-module Volo.Blogging

# Install a local module
abp install-local-module ../Acme.Blogging

# Add a package
abp add-package Volo.Abp.Blogging
```

## Module Extending

Extending pre-built modules:

```csharp
// Extending an entity
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

A modular monolith offers the advantages of microservices within a single process:

- Each module defines its own bounded context
- Inter-module communication happens via interfaces
- Deployment is as a single application
- It can later be converted into a microservice

```bash
abp new Acme.Crm --template app-nolayers --modern --modular
```

---

## Related

- [Framework Core](../abp-framework/SKILL.md) — AbpModule, lifecycle, [DependsOn]
- [Dependency Injection](../abp-dependency-injection/SKILL.md) — module-based service registration
- [Dependency Rules](../abp-dependency-rules/SKILL.md) — inter-module dependency rules
- [Microservices](../abp-microservices/SKILL.md) — from modular monolith to microservice
- ABP Docs: https://abp.io/docs/latest/framework/architecture/modularity/basics

---
name: abp-audit-logging
description: "ABP Framework v10.4 audit logging: AbpAuditingOptions, entity history, IAuditingStore, audit log storage and filtering. Use when configuring audit trails, audit logs, or entity history in ABP."
---

# ABP Audit Logging Skill

## Trigger
User asks about audit logging, audit trails, entity history, AbpAuditingOptions, IAuditingStore, audit log contributors, or tracking changes in ABP Framework.

---

## Core Concepts

ABP's audit logging system:
- **Automatically logs** application service method calls
- **Tracks entity changes** (create, update, delete) when configured
- **Records** HTTP request details, parameters, execution time, exceptions
- **Stores** audit logs in database via `IAuditingStore`
- **Extensible** via contributors and options

---

## Configuration

### AbpAuditingOptions

```csharp
Configure<AbpAuditingOptions>(options =>
{
    options.IsEnabled = true;                    // Root switch (default: true)
    options.HideErrors = true;                   // Hide audit save errors (default: true)
    options.IsEnabledForAnonymousUsers = true;   // Log anonymous users (default: true)
    options.AlwaysLogOnException = true;         // Always log on exception (default: true)
    options.IsEnabledForIntegrationService = false; // Enable for integration services
    options.IsEnabledForGetRequests = false;     // Log GET requests (default: false)
    options.DisableLogActionInfo = false;        // Don't log AuditLogActionInfo
    options.ApplicationName = "MyApp";           // Distinguish multi-app logs
    options.SaveEntityHistoryWhenNavigationChanges = true;
});
```

**Key options:**
- `IsEnabled` — Master switch; if false, all other options ignored
- `HideErrors` — If true, audit save errors logged silently; if false, throws
- `AlwaysLogOnException` — Force audit log on exception regardless of other settings
- `IsEnabledForGetRequests` — GET requests normally not logged (shouldn't change data)
- `ApplicationName` — Critical when multiple apps share audit log database

---

## Entity History

### Enabling Entity History

**Important:** Entity history is **disabled by default** for all entities. You must explicitly enable it.

#### Enable for All Entities

```csharp
Configure<AbpAuditingOptions>(options =>
{
    options.EntityHistorySelectors.AddAllEntities();
});
```

#### Enable with Custom Selector

```csharp
Configure<AbpAuditingOptions>(options =>
{
    options.EntityHistorySelectors.Add(
        new NamedTypeSelector(
            "MySelectorName",
            type => typeof(IEntity).IsAssignableFrom(type)
        )
    );
});
```

#### Enable per Entity with Attribute

```csharp
[Audited]
public class MyEntity : Entity<Guid>
{
    // ...
}
```

#### Disable per Entity

```csharp
[DisableAuditing]
public class MyEntity : Entity<Guid>
{
    // ...
}
```

#### Disable Specific Properties

```csharp
[Audited]
public class MyUser : Entity<Guid>
{
    public string Name { get; set; }

    [DisableAuditing] // Don't log password changes
    public string Password { get; set; }
}
```

#### Enable Only Specific Properties

```csharp
[DisableAuditing]
public class MyUser : Entity<Guid>
{
    [Audited] // Only log Name changes
    public string Name { get; set; }

    public string Email { get; set; }
    public string Password { get; set; }
}
```

#### Ignore Update Audit Properties

```csharp
// Skip audit properties on update and skip publishing entity updated event
entity.IgnoreAuditPropertiesOnUpdate();
```

### Entity Rules

An entity is **ignored** in entity change audit logging if:
- Added to `AbpAuditingOptions.IgnoredTypes`
- Does not implement `IEntity`
- Entity type is not public

---

## Enabling/Disabling Audit Logging

### Controllers & Actions

```csharp
[DisableAuditing]
public class MyController : AbpController
{
    [DisableAuditing]
    public IActionResult MyAction() { }
}
```

### Application Services & Methods

```csharp
[DisableAuditing]
public class MyAppService : ApplicationService
{
    [DisableAuditing]
    public async Task DoSomethingAsync() { }
}
```

### Other Services

```csharp
[Audited]
public class MyService : ITransientDependency
{
    public virtual async Task DoWorkAsync() { }
}
```

> Service must be DI-registered and method must be `virtual` for interception.

---

## IAuditingStore

Default implementation saves to database. Custom implementation:

```csharp
public class MyAuditingStore : IAuditingStore, ITransientDependency
{
    public async Task SaveAsync(AuditLogInfo auditInfo)
    {
        // Save to custom location (file, external service, etc.)
    }
}
```

### Audit Log Object

```csharp
public class AuditLogInfo
{
    public string ApplicationName { get; set; }
    public Guid? UserId { get; set; }
    public string UserName { get; set; }
    public Guid? TenantId { get; set; }
    public string TenantName { get; set; }
    public string ImpersonatorUserName { get; set; }
    public Guid? ImpersonatorUserId { get; set; }
    public DateTime ExecutionTime { get; set; }
    public int ExecutionDuration { get; set; } // milliseconds
    public string ClientIpAddress { get; set; }
    public string ClientName { get; set; }
    public string BrowserInfo { get; set; }
    public string HttpMethod { get; set; }
    public string Url { get; set; }
    public int? HttpStatusCode { get; set; }
    public string ExceptionMessage { get; set; }
    public string Exception { get; set; }
    public Dictionary<string, object> ExtraProperties { get; set; }
    public List<AuditLogActionInfo> Actions { get; set; }
    public List<EntityChangeInfo> EntityChanges { get; set; }
    public List<EntityPropertyChangeInfo> EntityPropertyChanges { get; set; }
}
```

---

## Audit Log Contributors

Extend audit log with custom data:

```csharp
public class MyAuditLogContributor : AuditLogContributor
{
    public override Task PreContributeAsync(AuditLogContributionContext context)
    {
        // Before action execution
        context.AuditLog.ExtraProperties["CustomKey"] = "CustomValue";
        return Task.CompletedTask;
    }

    public override Task PostContributeAsync(AuditLogContributionContext context)
    {
        // After action execution
        return Task.CompletedTask;
    }
}

// Register
Configure<AbpAuditingOptions>(options =>
{
    options.Contributors.Add<MyAuditLogContributor>();
});
```

---

## AlwaysLog Selectors

Force audit logging for specific services regardless of other settings:

```csharp
Configure<AbpAuditingOptions>(options =>
{
    options.AlwaysLogSelectors.Add(
        new NamedTypeSelector(
            "CriticalServices",
            type => typeof(ICriticalService).IsAssignableFrom(type)
        )
    );
});
```

---

## Database Provider Support

Audit logging module supports:
- **Entity Framework Core** — `AbpAuditLoggingDbContext`
- **MongoDB** — `AbpAuditLoggingMongoDbContext`

Both providers store audit logs in `AbpAuditLogs` collection/table.

### UseAuditing()

Enable ASP.NET Core auditing middleware:

```csharp
app.UseAuditing();
```

Usually called automatically by `AbpAspNetCoreAuditingModule`.

---

## AbpAspNetCoreAuditingOptions

ASP.NET Core specific auditing options:

```csharp
Configure<AbpAspNetCoreAuditingOptions>(options =>
{
    // Configure URL filtering
    options.UrlControllers.Add("HealthCheck"); // Exclude health check endpoints
});

Configure<AbpAspNetCoreAuditingUrlOptions>(options =>
{
    // URL-based filtering
    options.IgnoredUrls.Add("/health");
    options.IgnoredUrls.Add("/api/health");
});
```

---

## Blazor Server Limitation

> **Entity history in Blazor Server:** Not guaranteed to be complete for every UI interaction. Blazor Server uses SignalR-based event handling, and under some flows the audit scope/action tracking may not align with `DbContext.SaveChanges`, causing missing or partial entity change records. See [GitHub #11682](https://github.com/abpframework/abp/issues/11682).

---

## Best Practices

1. **Enable entity history explicitly** — it's off by default
2. **Use `AddAllEntities()`** only if you need full audit trail (storage intensive)
3. **Use `[Audited]` / `[DisableAuditing]`** for fine-grained control
4. **Disable auditing on sensitive properties** (passwords, tokens, PII)
5. **Set `ApplicationName`** when multiple apps share audit database
6. **Use contributors** to add custom context (correlation IDs, session data)
7. **Don't log GET requests** unless debugging (they shouldn't change data)
8. **Monitor audit log storage** — it grows fast with entity history enabled
9. **Use `AlwaysLogSelectors`** for critical services that must always be logged
10. **Test Blazor Server entity history** — known limitations exist

---

## Audit Logging Module

The `Volo.Abp.SettingManagement` module includes `IAuditingStore` implementation:
- Persists audit logs to database
- Provides EF Core and MongoDB providers
- Includes domain layer (aggregates, repositories)

### Installation

```bash
abp add-package Volo.Abp.AuditLogging.EntityFrameworkCore
```

---

## Related

- [Exception Handling](../fundamentals/exception-handling.md) — Exceptions logged in audit trail
- [Settings](../infrastructure/settings.md) — Audit log settings
- [Multi-Tenancy](../architecture/multi-tenancy/index.md) — Tenant-scoped audit logs

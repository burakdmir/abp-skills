---
name: abp-audit-logging
description: "ABP Framework v10.4 audit logging: AbpAuditingOptions, entity history, IAuditingStore, audit log storage and filtering. Use when configuring audit trails, audit logs, or entity history in ABP."
---

# ABP Audit Logging Skill

## Trigger
Audit logging, audit trails, entity history, AbpAuditingOptions, IAuditingStore, tracking changes.

---

## Quick Reference

### Configuration
```csharp
Configure<AbpAuditingOptions>(options =>
{
    options.IsEnabled = true;
    options.IsEnabledForGetRequests = false;
    options.ApplicationName = "MyApp";
    options.EntityHistorySelectors.AddAllEntities();
});
```

### Entity History (OFF by default — must enable explicitly)
```csharp
// All entities
options.EntityHistorySelectors.AddAllEntities();

// Specific entities
options.EntityHistorySelectors.Add(new NamedTypeSelector("MySelector",
    type => typeof(IEntity).IsAssignableFrom(type)));

// Per entity
[Audited] public class MyEntity : Entity<Guid> { }
[DisableAuditing] public class SecretEntity : Entity<Guid> { }

// Specific properties
[DisableAuditing] public string Password { get; set; }
[Audited] public string Name { get; set; }  // Only log this
```

### Disable Auditing
```csharp
[DisableAuditing] public class MyController : AbpController { }
[DisableAuditing] public async Task DoWorkAsync() { }
```

### IAuditingStore (custom implementation)
```csharp
public class MyStore : IAuditingStore, ITransientDependency
{
    public async Task SaveAsync(AuditLogInfo auditInfo) { /* custom storage */ }
}
```

### Contributors
```csharp
public class MyContributor : AuditLogContributor
{
    public override Task PreContributeAsync(AuditLogContributionContext ctx)
    {
        ctx.AuditLog.ExtraProperties["CorrelationId"] = CorrelationId;
        return Task.CompletedTask;
    }
}
Configure<AbpAuditingOptions>(o => o.Contributors.Add<MyContributor>());
```

### AlwaysLog Selectors
```csharp
options.AlwaysLogSelectors.Add(new NamedTypeSelector("Critical",
    type => typeof(ICriticalService).IsAssignableFrom(type)));
```

### ASP.NET Core Options
```csharp
Configure<AbpAspNetCoreAuditingUrlOptions>(o =>
{
    o.IgnoredUrls.Add("/health");
});
```

### AuditLogInfo Key Fields
- UserId, TenantId, ExecutionTime, ExecutionDuration
- ClientIpAddress, BrowserInfo, HttpMethod, Url
- ExceptionMessage, EntityChanges, EntityPropertyChanges

### Best Practices
- Entity history OFF by default — enable explicitly
- Use AddAllEntities() only if needed (storage intensive)
- Disable auditing on sensitive properties (passwords, tokens)
- Set ApplicationName for multi-app audit databases
- Don't log GET requests unless debugging
- Monitor storage growth with entity history
- Blazor Server entity history has known limitations (#11682)

## Related

[Infrastructure](../abp-infrastructure/SKILL.md) · [DDD](../abp-ddd/SKILL.md) · [Multi-Tenancy](../abp-multitenancy/SKILL.md) · Docs: https://abp.io/docs/latest/framework/infrastructure/audit-logging

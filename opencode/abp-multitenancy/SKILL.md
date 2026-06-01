---
name: abp-multitenancy
description: "ABP Framework v10.4 multi-tenancy: tenant resolver, ICurrentTenant, IMultiTenant, database isolation, tenant-based data filtering. Use when working with SaaS, multi-tenancy, or tenant management in ABP."
---

# ABP Framework — Multi-Tenancy

ABP Framework v10.4 multi-tenancy (SaaS) guide. Tenant resolver, IMultiTenant, ICurrentTenant.

## Trigger

- "ABP multi-tenancy"
- "ABP tenant"
- "ABP SaaS"
- "ABP IMultiTenant"
- "ABP ICurrentTenant"
- "ABP tenant resolver"

## Configuration

```csharp
Configure<AbpMultiTenancyOptions>(options => options.IsEnabled = true);
```

## IMultiTenant Entity

```csharp
public class Product : AggregateRoot<Guid>, IMultiTenant
{
    public Guid? TenantId { get; set; }
    public string Name { get; set; }
}
```

- `TenantId` is nullable — `null` = belongs to the Host
- ABP automatically applies data filtering

## ICurrentTenant

```csharp
CurrentTenant.Id           // Guid?
CurrentTenant.Name         // string
CurrentTenant.IsAvailable  // bool

// Changing the tenant (scoped)
using (CurrentTenant.Change(tenantId))
{
    var count = await _productRepository.GetCountAsync();
}

// Host context
using (CurrentTenant.Change(null)) { }
```

## Data Filter Disable

```csharp
using (_dataFilter.Disable<IMultiTenant>())
{
    return await _productRepository.GetCountAsync();  // All tenants
}
```

## Tenant Resolvers

Default order: CurrentUser → QueryString → Route → Header → Cookie

### Subdomain Resolver

```csharp
Configure<AbpTenantResolveOptions>(options =>
{
    options.AddDomainTenantResolver("{0}.mydomain.com");
});
```

### Fallback Tenant

```csharp
Configure<AbpTenantResolveOptions>(options =>
{
    options.FallbackTenant = "acme";
});
```

## Best Practices

1. Implement `IMultiTenant` for tenant-specific entities
2. Use `CurrentTenant.Change` with `using`
3. `CurrentUserTenantResolveContributor` must always be first
4. Use the Tenant Management module

## Related

[EF Core](../abp-efcore/SKILL.md) · [MongoDB](../abp-mongodb/SKILL.md) · [Authorization](../abp-authorization/SKILL.md) · [Settings & Features](../abp-settings-features/SKILL.md) · Docs: https://abp.io/docs/latest/framework/architecture/multi-tenancy

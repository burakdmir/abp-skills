---
name: abp-multitenancy
description: "ABP Framework v10.4 multi-tenancy: tenant resolver, ICurrentTenant, IMultiTenant, database isolation, tenant-based data filtering. Use when working with SaaS, multi-tenancy, or tenant management in ABP."
---

# ABP Framework — Multi-Tenancy

ABP Framework v10.4 multi-tenancy (SaaS) implementation guide. Tenant resolver, IMultiTenant, ICurrentTenant, database isolation.

## Trigger

- "ABP multi-tenancy"
- "ABP tenant"
- "ABP SaaS"
- "ABP IMultiTenant"
- "ABP ICurrentTenant"
- "ABP tenant resolver"
- "ABP subdomain tenant"
- "ABP database per tenant"

## Terminology

- **Tenant:** A customer of the SaaS application
- **Host:** The company that manages the application
- **Multi-Tenancy:** A single instance, multiple customers

## Configuration

```csharp
Configure<AbpMultiTenancyOptions>(options =>
{
    options.IsEnabled = true;
});
```

> In the startup templates this is controlled from a single point via the `MultiTenancyConsts` class.

## Database Architectures

| Approach | Description |
|---|---|
| **Single Database** | All tenants in the same DB, separated by `TenantId` |
| **Database per Tenant** | Each tenant has its own DB |
| **Hybrid** | Some tenants share a DB, others have a separate DB |

## IMultiTenant Interface

```csharp
public class Product : AggregateRoot<Guid>, IMultiTenant
{
    public Guid? TenantId { get; set; }  // From the IMultiTenant interface
    public string Name { get; set; }
    public float Price { get; set; }
}
```

**Important:**
- `TenantId` is nullable — if `null`, the entity belongs to the **Host**
- ABP automatically applies **data filtering** for the current tenant
- `TenantId` is set automatically (from `ICurrentTenant.Id`)

## ICurrentTenant

```csharp
// Properties
CurrentTenant.Id        // Guid? — Current tenant ID
CurrentTenant.Name      // string — Current tenant name
CurrentTenant.IsAvailable  // bool — true if ID is not null

// Changing the tenant (scoped)
using (CurrentTenant.Change(tenantId))
{
    // Within this scope, operations are performed on behalf of the specified tenant
    var count = await _productRepository.GetCountAsync();
}

// Switch to the host context
using (CurrentTenant.Change(null))
{
    // Operation in the host context
}
```

## Data Filtering — Disabling the Multi-Tenancy Filter

```csharp
public class ProductManager : DomainService
{
    private readonly IRepository<Product, Guid> _productRepository;
    private readonly IDataFilter _dataFilter;

    public ProductManager(IRepository<Product, Guid> productRepository, IDataFilter dataFilter)
    {
        _productRepository = productRepository;
        _dataFilter = dataFilter;
    }

    public async Task<long> GetAllProductCountAsync()
    {
        using (_dataFilter.Disable<IMultiTenant>())
        {
            return await _productRepository.GetCountAsync();
        }
    }
}
```

## Tenant Resolvers

### Default Resolvers (In Order)

1. **CurrentUserTenantResolveContributor** — From user claims (must always be first)
2. **QueryStringTenantResolveContributor** — `?__tenant=xxx`
3. **RouteTenantResolveContributor** — From the URL path
4. **HeaderTenantResolveContributor** — From the HTTP header (`__tenant`)
5. **CookieTenantResolveContributor** — From the cookie (`__tenant`)

### Changing the Tenant Key

```csharp
Configure<AbpAspNetCoreMultiTenancyOptions>(options =>
{
    options.TenantKey = "MyTenantKey";
});
```

### Subdomain/Domain Tenant Resolver

```csharp
// Subdomain: mytenant.mydomain.com
Configure<AbpTenantResolveOptions>(options =>
{
    options.AddDomainTenantResolver("{0}.mydomain.com");
});

// OpenIddict wildcard domain (if a separate Auth Server is used)
PreConfigure<AbpOpenIddictWildcardDomainOptions>(options =>
{
    options.EnableWildcardDomainSupport = true;
    options.WildcardDomainsFormat.Add("https://{0}.mydomain.com");
});
```

### Custom Tenant Resolver

```csharp
public class MyCustomTenantResolveContributor : TenantResolveContributorBase
{
    public override string Name => "Custom";

    public override Task ResolveAsync(ITenantResolveContext context)
    {
        // Set context.TenantIdOrName
        // Use DI via context.ServiceProvider
        return Task.CompletedTask;
    }
}

// Registration
Configure<AbpTenantResolveOptions>(options =>
{
    options.TenantResolvers.Add(new MyCustomTenantResolveContributor());
});
```

### Fallback Tenant

```csharp
Configure<AbpTenantResolveOptions>(options =>
{
    options.FallbackTenant = "acme";  // Used when no tenant is found
});
```

## Multi-Tenancy Middleware

```csharp
app.UseAuthentication();
app.UseMultiTenancy();  // Immediately after authentication
```

> Already configured in the startup templates.

## Tenant Store

### Tenant Management Module (Recommended)

Included in the startup templates. The `ITenantStore` implementation fetches tenant information from the DB.

### Configuration Data Store (Alternative)

```json
// appsettings.json
"Tenants": [
    {
        "Id": "446a5211-3d72-4339-9adc-845151f8ada0",
        "Name": "tenant1",
        "NormalizedName": "TENANT1"
    },
    {
        "Id": "25388015-ef1c-4355-9c18-f6b6ddbaf89d",
        "Name": "tenant2",
        "NormalizedName": "TENANT2",
        "ConnectionStrings": {
            "Default": "...tenant2's connection string..."
        }
    }
]
```

## Host vs Tenant DbContext

```csharp
[IgnoreMultiTenancy]  // Always uses the host DB
public class TenantManagementDbContext : AbpDbContext<TenantManagementDbContext> { }
```

## Other Multi-Tenancy Infrastructure

In ABP, the following services are designed to be multi-tenancy-aware:
- BLOB Storing
- Caching
- Data Filtering
- Data Seeding
- Authorization
- Settings

## Best Practices

1. **Always implement `IMultiTenant`** — For tenant-specific entities
2. **Use `CurrentTenant.Change` with `using`** — The previous value is restored outside the scope
3. **Tenant resolver order matters** — `CurrentUserTenantResolveContributor` must always be first
4. **Use `CurrentTenant.Change(null)` for host-side operations**
5. **Use the Tenant Management module** — The `appsettings.json` approach is only for simple scenarios
6. **In the separate DB approach, cross-tenant queries must be implemented yourself**

---

## Related

- [EF Core](../abp-efcore/SKILL.md) — tenant connection string, IgnoreMultiTenancy
- [MongoDB](../abp-mongodb/SKILL.md) — tenant isolation with MongoDB
- [Authorization](../abp-authorization/SKILL.md) — permissions with MultiTenancySides
- [Settings & Features](../abp-settings-features/SKILL.md) — tenant-based feature/setting
- ABP Docs: https://abp.io/docs/latest/framework/architecture/multi-tenancy

# ABP Framework — Multi-Tenancy

ABP Framework v10.4 multi-tenancy (SaaS) rehberi. Tenant resolver, IMultiTenant, ICurrentTenant.

## Trigger

- "ABP multi-tenancy"
- "ABP tenant"
- "ABP SaaS"
- "ABP IMultiTenant"
- "ABP ICurrentTenant"
- "ABP tenant resolver"

## Konfigürasyon

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

- `TenantId` nullable — `null` = Host'a ait
- ABP otomatik data filtering uygular

## ICurrentTenant

```csharp
CurrentTenant.Id           // Guid?
CurrentTenant.Name         // string
CurrentTenant.IsAvailable  // bool

// Tenant değiştirme (scoped)
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
    return await _productRepository.GetCountAsync();  // Tüm tenant'lar
}
```

## Tenant Resolver'lar

Varsayılan sıra: CurrentUser → QueryString → Route → Header → Cookie

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

1. `IMultiTenant` implement et tenant-specific entity'ler için
2. `CurrentTenant.Change` ile `using` kullan
3. `CurrentUserTenantResolveContributor` her zaman ilk olmalı
4. Tenant Management module kullan

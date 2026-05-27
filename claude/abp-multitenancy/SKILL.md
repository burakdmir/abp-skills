# ABP Framework — Multi-Tenancy

ABP Framework v10.4 multi-tenancy (SaaS) uygulama rehberi. Tenant resolver, IMultiTenant, ICurrentTenant, database isolation.

## Trigger

- "ABP multi-tenancy"
- "ABP tenant"
- "ABP SaaS"
- "ABP IMultiTenant"
- "ABP ICurrentTenant"
- "ABP tenant resolver"
- "ABP subdomain tenant"
- "ABP database per tenant"

## Terminoloji

- **Tenant:** SaaS uygulamasının müşterisi
- **Host:** Uygulamayı yöneten şirket
- **Multi-Tenancy:** Tek instance, birden fazla müşteri

## Konfigürasyon

```csharp
Configure<AbpMultiTenancyOptions>(options =>
{
    options.IsEnabled = true;
});
```

> Startup template'lerde `MultiTenancyConsts` class'ı ile tek noktadan kontrol edilir.

## Database Mimarileri

| Yaklaşım | Açıklama |
|---|---|
| **Single Database** | Tüm tenant'lar aynı DB'de, `TenantId` ile ayrılır |
| **Database per Tenant** | Her tenant'ın kendi DB'si |
| **Hybrid** | Bazı tenant'lar paylaşır, bazıları ayrı DB'ye sahip |

## IMultiTenant Interface

```csharp
public class Product : AggregateRoot<Guid>, IMultiTenant
{
    public Guid? TenantId { get; set; }  // IMultiTenant interface'den
    public string Name { get; set; }
    public float Price { get; set; }
}
```

**Önemli:**
- `TenantId` nullable — `null` ise entity **Host**'a ait
- ABP otomatik olarak current tenant için **data filtering** uygular
- `TenantId` otomatik set edilir (`ICurrentTenant.Id`'den)

## ICurrentTenant

```csharp
// Properties
CurrentTenant.Id        // Guid? — Mevcut tenant ID
CurrentTenant.Name      // string — Mevcut tenant adı
CurrentTenant.IsAvailable  // bool — ID null değilse true

// Tenant değiştirme (scoped)
using (CurrentTenant.Change(tenantId))
{
    // Bu scope'ta işlemler belirtilen tenant adına yapılır
    var count = await _productRepository.GetCountAsync();
}

// Host context'e geç
using (CurrentTenant.Change(null))
{
    // Host contextinde işlem
}
```

## Data Filtering — Multi-Tenancy Filter'ı Devre Dışı Bırakma

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

## Tenant Resolver'lar

### Varsayılan Resolver'lar (Sırayla)

1. **CurrentUserTenantResolveContributor** — User claim'lerinden (her zaman ilk olmalı)
2. **QueryStringTenantResolveContributor** — `?__tenant=xxx`
3. **RouteTenantResolveContributor** — URL path'ten
4. **HeaderTenantResolveContributor** — HTTP header'dan (`__tenant`)
5. **CookieTenantResolveContributor** — Cookie'den (`__tenant`)

### Tenant Key Değiştirme

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

// OpenIddict wildcard domain (ayrı Auth Server kullanılıyorsa)
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
        // context.TenantIdOrName set et
        // context.ServiceProvider ile DI kullan
        return Task.CompletedTask;
    }
}

// Kayıt
Configure<AbpTenantResolveOptions>(options =>
{
    options.TenantResolvers.Add(new MyCustomTenantResolveContributor());
});
```

### Fallback Tenant

```csharp
Configure<AbpTenantResolveOptions>(options =>
{
    options.FallbackTenant = "acme";  // Tenant bulunamazsa bu kullanılır
});
```

## Multi-Tenancy Middleware

```csharp
app.UseAuthentication();
app.UseMultiTenancy();  // Auth'dan hemen sonra
```

> Startup template'lerde zaten yapılandırılmıştır.

## Tenant Store

### Tenant Management Module (Önerilen)

Startup template'lerde dahil. `ITenantStore` implementasyonu DB'den tenant bilgilerini çeker.

### Configuration Data Store (Alternatif)

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
[IgnoreMultiTenancy]  // Her zaman host DB kullanır
public class TenantManagementDbContext : AbpDbContext<TenantManagementDbContext> { }
```

## Diğer Multi-Tenancy Infrastructure

ABP'de aşağıdaki servisler multi-tenancy-aware tasarlanmıştır:
- BLOB Storing
- Caching
- Data Filtering
- Data Seeding
- Authorization
- Settings

## Best Practices

1. **Her zaman `IMultiTenant` implement et** — Tenant-specific entity'ler için
2. **`CurrentTenant.Change` ile `using` kullan** — Scope dışında eski değer geri yüklenir
3. **Tenant resolver sırası önemli** — `CurrentUserTenantResolveContributor` her zaman ilk olmalı
4. **Host-side işlemler için `CurrentTenant.Change(null)` kullan**
5. **Tenant Management module kullan** — `appsettings.json` yaklaşımı sadece basit senaryolar için
6. **Separate DB yaklaşımında cross-tenant query kendi başına implement edilmeli**

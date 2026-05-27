# ABP Framework — Core Skill

ABP Framework v10.4 için temel geliştirme skill'i. Opinionated, DDD-tabanlı, modüler ASP.NET Core uygulama geliştirme rehberi.

## Trigger

- "ABP ile proje oluştur"
- "ABP solution/template"
- "ABP module"
- "ABP best practices"
- "ABP architecture"
- "abp new"
- ABP projesinde çalışma

## ABP Nedir

ABP, .NET ve ASP.NET Core üzerinde **opinionated architecture** sunan, DDD prensiplerine dayalı, modüler bir framework'tür. Tekrarlayan işleri otomatize eder, production-ready startup template'ler, pre-built application modüller ve tooling sağlar.

## Solution Template'leri

| Template | Açıklama |
|---|---|
| `app` | Layered web application (varsayılan) |
| `app-nolayers` | Single-layer web application |
| `microservice` | Microservice solution (Business+ lisans) |
| `empty` | Empty solution |

### UI Framework Seçenekleri

- `mvc` — ASP.NET Core MVC / Razor Pages
- `angular` — Angular SPA
- `blazor-webapp` — Blazor Web App
- `blazor` — Blazor WASM
- `blazor-server` — Blazor Server
- `react` — React SPA (modern template ile)
- `no-ui` — Frontend olmadan

### Database Provider'ları

- `ef` — Entity Framework Core (varsayılan)
- `mongodb` — MongoDB

## ABP CLI Komutları

```bash
# Kurulum
dotnet tool install -g Volo.Abp.Studio.Cli

# Yeni solution oluştur
abp new Acme.BookStore --template app
abp new Acme.BookStore --template app --modern          # React-first modern template
abp new Acme.BookStore --template app-nolayers --modern # Single-layer modern
abp new Acme.BookStore --template microservice --modern # Microservice modern

# Module oluştur
abp new-module Acme.BookStore.Orders -t module:ddd
abp new-module Acme.BookStore.Orders --modern

# Package ekle
abp add-package Volo.Abp.EntityFrameworkCore

# Paketleri güncelle
abp update

# Proxy generate (Angular/C#/JS)
abp generate-proxy -t ng
abp generate-proxy -t csharp

# Kaynak kodu indir
abp get-source Volo.Blogging
abp add-source-code Volo.Chat
```

### Modern Template'ler (`--modern` flag)

Modern template'ler React-first'tir ve ABP Studio ile gönderilen farklı template kaynağını kullanır.

```bash
abp new Acme.BookStore --template app --modern
abp new Acme.BookStore --template app-nolayers --modern --modular  # Modular monolith
abp new Acme.BookStore --template microservice --modern --services Ordering,Shipping
abp new Acme.BookStore --template microservice --modern --shadcn-theme blue
```

**Shadcn theme değerleri:** `slate` (default), `pink`, `blue`, `turquoise`, `orange`, `purple`

## Proje Yapısı (Layered Template)

```
Acme.BookStore/
├── src/
│   ├── Acme.BookStore.Domain.Shared    # Sabitler, enum'lar, localization
│   ├── Acme.BookStore.Domain           # Entities, aggregates, domain services, repositories (interface)
│   ├── Acme.BookStore.Application.Contracts  # DTOs, application service interfaces
│   ├── Acme.BookStore.Application      # Application services, object mapping
│   ├── Acme.BookStore.EntityFrameworkCore  # DbContext, repository implementations, migrations
│   ├── Acme.BookStore.HttpApi          # API controllers
│   ├── Acme.BookStore.HttpApi.Client   # Dynamic C# HTTP clients
│   └── Acme.BookStore.Web              # MVC/Razor Pages UI
└── test/
    ├── Acme.BookStore.TestBase
    ├── Acme.BookStore.Domain.Tests
    ├── Acme.BookStore.Application.Tests
    └── Acme.BookStore.Web.Tests
```

## Module Class Yapısı

Her modül bir `AbpModule` türevi tanımlar:

```csharp
[DependsOn(
    typeof(AbpAspNetCoreMvcModule),
    typeof(AbpEntityFrameworkCoreModule),
    typeof(AbpAutofacModule)
)]
public class BookStoreModule : AbpModule
{
    public override void PreConfigureServices(ServiceConfigurationContext context) { }
    
    public override void ConfigureServices(ServiceConfigurationContext context)
    {
        // DI kayıt, modül konfigürasyonu
    }
    
    public override void OnApplicationInitialization(ApplicationInitializationContext context)
    {
        // Middleware pipeline, startup logic
    }
    
    public override void OnApplicationShutdown(ApplicationShutdownContext context) { }
}
```

### Lifecycle Metodları

| Metot | Açıklama | Async Versiyon |
|---|---|---|
| `PreConfigureServices` | Tüm ConfigureServices'den önce çalışır | `PreConfigureServicesAsync` |
| `ConfigureServices` | DI kayıt ve modül konfigürasyonu | `ConfigureServicesAsync` |
| `PostConfigureServices` | Tüm ConfigureServices'den sonra çalışır | `PostConfigureServicesAsync` |
| `OnPreApplicationInitialization` | Init öncesi | `OnPreApplicationInitializationAsync` |
| `OnApplicationInitialization` | Middleware kurulumu | `OnApplicationInitializationAsync` |
| `OnPostApplicationInitialization` | Init sonrası | `OnPostApplicationInitializationAsync` |
| `OnApplicationShutdown` | Kapanış logic'i | `OnApplicationShutdownAsync` |

## Dependency Injection

### Otomatik Kayıt Yöntemleri

```csharp
// 1. Interface implementasyonu
public class TaxCalculator : ITransientDependency { }       // Transient
public class CacheService : ISingletonDependency { }        // Singleton
public class ScopedService : IScopedDependency { }          // Scoped

// 2. Attribute ile
[Dependency(ServiceLifetime.Transient, ReplaceServices = true)]
public class TaxCalculator { }

// 3. ExposeServices ile hangi interface'lerin expose edileceğini kontrol et
[ExposeServices(typeof(ITaxCalculator))]
public class TaxCalculator : ICalculator, ITaxCalculator, ITransientDependency { }

// 4. Keyed Services
[ExposeKeyedService<ITaxCalculator>("taxCalculator")]
public class TaxCalculator : ITransientDependency { }
```

### Manuel Kayıt

```csharp
public override void ConfigureServices(ServiceConfigurationContext context)
{
    context.Services.AddSingleton<TaxCalculator>(new TaxCalculator(0.18));
    context.Services.AddScoped<ITaxCalculator>(sp => sp.GetRequiredService<TaxCalculator>());
    
    // Service değiştirme
    context.Services.Replace(ServiceDescriptor.Transient<IConnectionStringResolver, MyResolver>());
}
```

## Önemli Konfigürasyon Pattern'leri

```csharp
// Options pattern
Configure<AbpDbConnectionOptions>(options =>
{
    options.ConnectionStrings.Default = "...";
});

// Modül konfigürasyonu
Configure<AbpMultiTenancyOptions>(options =>
{
    options.IsEnabled = true;
});
```

## Pre-built Application Modüller

| Modül | Açıklama |
|---|---|
| `Volo.Abp.Account` | Account management |
| `Volo.Abp.Identity` | Identity & user management |
| `Volo.Abp.TenantManagement` | Tenant management (multi-tenancy) |
| `Volo.Abp.SettingManagement` | Setting management |
| `Volo.Abp.PermissionManagement` | Permission management |
| `Volo.Abp.FeatureManagement` | Feature management |
| `Volo.Abp.AuditLogging` | Audit logging |
| `Volo.Abp.BackgroundJobs` | Background job system |
| `Volo.Abp.CmsKit` | CMS kit (content management) |
| `Volo.Abp.Saas` | SaaS module (PRO) |
| `Volo.Abp.OpenIddict` | OpenIddict integration |

## Best Practices

1. **Her zaman `AbpModule` ile modül bağımlılıklarını tanımla** — `[DependsOn]` attribute kullan
2. **GUID primary key kullan** — `IGuidGenerator.Create()` ile sequential GUID üret
3. **DTO kullan** — Entity'leri presentation layer'a expose etme
4. **Mapperly kullan** — Object-to-object mapping için (ABP 10.4'te varsayılan)
5. **Repository pattern kullan** — `IRepository<TEntity, TKey>` inject et
6. **Unit of Work convention'larına güven** — Manuel UOW yönetimine gerek yok
7. **Async metodları tercih et** — `async/await` ile scalable kod yaz
8. **Domain layer'ı database provider'dan izole tut** — `IAsyncQueryableExecuter` kullan

## Kaynaklar

- ABP Docs: `/Users/burakdemir/Code/abp-upstream/docs/en/`
- ABP CLI: `abp help`
- ABP GitHub: https://github.com/abpframework/abp
- ABP Community: https://abp.io/community/

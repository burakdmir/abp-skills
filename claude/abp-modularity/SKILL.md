# ABP Framework — Modularity

ABP Framework v10.4 modüler uygulama geliştirme rehberi. Module sistemi, dependency management, plugin modüller ve best practices.

## Trigger

- "ABP modularity"
- "ABP module oluştur"
- "ABP DependsOn"
- "ABP plugin module"
- "ABP modular monolith"
- "ABP module dependency"
- "ABP modül bağımlılığı"

## Modularity Nedir

ABP tam modüler uygulamalar ve sistemler oluşturmayı destekler. Her modül kendi entity'lerini, servislerini, database entegrasyonunu, API'lerini ve UI component'lerini içerebilir.

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
        // DI kayıt, modül konfigürasyonu
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

### Lifecycle Metodları

| Metot | Ne Zaman Çalışır | Kullanım |
|---|---|---|
| `PreConfigureServices` | Tüm `ConfigureServices`'lerden önce | Erken konfigürasyon |
| `ConfigureServices` | Service registration | DI kayıt, modül ayarları |
| `PostConfigureServices` | Tüm `ConfigureServices`'lerden sonra | Geç konfigürasyon |
| `OnPreApplicationInitialization` | Init öncesi | Pre-init logic |
| `OnApplicationInitialization` | Uygulama başlatma | Middleware pipeline |
| `OnPostApplicationInitialization` | Init sonrası | Post-init logic |
| `OnApplicationShutdown` | Uygulama kapanışı | Cleanup logic |

Her metodun `Async` versiyonu da mevcuttur.

## Module Dependencies

```csharp
// Tek DependsOn ile birden fazla
[DependsOn(typeof(AbpAspNetCoreMvcModule), typeof(AbpAutofacModule))]
public class BlogModule : AbpModule { }

// Birden fazla attribute
[DependsOn(typeof(AbpAspNetCoreMvcModule))]
[DependsOn(typeof(AbpAutofacModule))]
public class BlogModule : AbpModule { }
```

ABP startup'ta dependency graph'i inceler ve modülleri doğru sırayla başlatır/kapatır.

## Additional Assembly

Nadir durumlarda, modülünüz birden fazla assembly'den oluşuyorsa:

```csharp
[DependsOn(...)]
[AdditionalAssembly(typeof(BlogService))]  // Hedef assembly'den bir type
public class BlogModule : AbpModule { }
```

> **Uyarı:** `AdditionalAssembly` sadece gerçekten gerektiğinde kullanın. Normalde `DependsOn` tercih edilmelidir.

## Framework vs Application Modules

| Tip | Açıklama | Örnek |
|---|---|---|
| **Framework Module** | Altyapı, entegrasyon, abstraction | Caching, EF Core, Validation, Logging |
| **Application Module** | İşlevsel/business özellikler | Blogging, Identity, Tenant Management |

## Plugin Modules

Runtime'da dinamik olarak yüklenebilen modüller:

```csharp
[DependsOn(typeof(AbpKernelModule))]
public class MyPluginModule : AbpModule { }
```

Plugin modüller:
- Compile-time'da referans edilmez
- Runtime'da belirtilen dizinden yüklenir
- Hot-plug benzeri senaryolar için kullanılır

## Module Geliştirme Best Practices

1. **DDD katmanlarına uygun paketleme:**
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

2. **Database provider'dan bağımsız tasarla** — Domain/Application layer'ları EF Core'a bağımlı yapma

3. **Her modül kendi connection string'ini tanımlayabilir:**
   ```csharp
   Configure<AbpDbConnectionOptions>(options =>
   {
       options.ConnectionStrings["Blog"] = "...";
   });
   ```

4. **Modül konfigürasyonu için Options pattern kullan:**
   ```csharp
   public class BlogOptions
   {
       public int MaxPostLength { get; set; } = 5000;
   }
   
   // Module'da configure et
   Configure<BlogOptions>(options => options.MaxPostLength = 10000);
   ```

5. **Reusable modüller için `IRepository` kullan** — Direkt `DbContext` kullanma

## CLI ile Module Oluşturma

```bash
# DDD modülü
abp new-module Acme.Blog -t module:ddd

# Modern modül
abp new-module Acme.Blog --modern

# Belirli solution'a ekle
abp new-module Acme.Blog -t module:ddd -ts Acme.Crm.sln

# EF + MVC destekli
abp new-module Acme.Blog -t module:ddd -d ef -u mvc
```

## Modül Yükleme

```bash
# NuGet modülü yükle
abp install-module Volo.Blogging

# Local modül yükle
abp install-local-module ../Acme.Blogging

# Package ekle
abp add-package Volo.Abp.Blogging
```

## Module Extending

Pre-built modülleri genişletme:

```csharp
// Entity genişletme
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

Modüler monolith, mikro servislerin avantajlarını tek process'te sunar:

- Her modül kendi bounded context'ini tanımlar
- Modüller arası iletişim interface üzerinden
- Deployment tek bir uygulama olarak
- İleride mikro servise dönüştürülebilir

```bash
abp new Acme.Crm --template app-nolayers --modern --modular
```

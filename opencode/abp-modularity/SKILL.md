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

## Module Dependencies

```csharp
[DependsOn(typeof(AbpAspNetCoreMvcModule), typeof(AbpAutofacModule))]
public class BlogModule : AbpModule { }
```

ABP startup'ta dependency graph'i inceler ve modülleri doğru sırayla başlatır/kapatır.

## CLI ile Module Oluşturma

```bash
abp new-module Acme.Blog -t module:ddd
abp new-module Acme.Blog --modern
abp new-module Acme.Blog -t module:ddd -ts Acme.Crm.sln
abp new-module Acme.Blog -t module:ddd -d ef -u mvc
```

## Modül Yükleme

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

# ABP Framework — API Development

ABP Framework v10.4 API geliştirme rehberi. Auto API Controllers, Dynamic C# Clients, Static C# Clients, Swagger, API Versioning, Integration Services.

## Trigger

- "ABP API controller"
- "ABP auto controller"
- "ABP dynamic client"
- "ABP static client"
- "ABP swagger"
- "ABP API versioning"
- "ABP integration service"
- "ABP REST API"
- "ABP HTTP client"

---

## Auto API Controllers

ABP application service'leri otomatik olarak REST API endpoint'lerine dönüştürür.

### Konfigürasyon

```csharp
[DependsOn(BookStoreApplicationModule)]
public class BookStoreWebModule : AbpModule
{
    public override void PreConfigureServices(ServiceConfigurationContext context)
    {
        PreConfigure<AbpAspNetCoreMvcOptions>(options =>
        {
            options.ConventionalControllers.Create(typeof(BookStoreApplicationModule).Assembly);
        });
    }
}
```

### HTTP Method Mapping (Convention)

| Method Prefix | HTTP Method |
|---|---|
| `GetList`, `GetAll`, `Get` | GET |
| `Put`, `Update` | PUT |
| `Delete`, `Remove` | DELETE |
| `Create`, `Add`, `Insert`, `Post` | POST |
| `Patch` | PATCH |
| Diğer | POST (default) |

### Route Hesaplama

| Service Method | HTTP Method | Route |
|---|---|---|
| `GetAsync(Guid id)` | GET | `/api/app/book/{id}` |
| `GetListAsync()` | GET | `/api/app/book` |
| `CreateAsync(CreateBookDto input)` | POST | `/api/app/book` |
| `UpdateAsync(Guid id, UpdateBookDto input)` | PUT | `/api/app/book/{id}` |
| `DeleteAsync(Guid id)` | DELETE | `/api/app/book/{id}` |
| `GetEditorsAsync(Guid id)` | GET | `/api/app/book/{id}/editors` |

**Route kuralları:**
- Her zaman `/api` ile başlar
- Varsayılan root path: `/app`
- Controller adı normalize edilir: `BookAppService` → `book` (kebab-case, suffix'ler kaldırılır)
- `id` parametresi route'a eklenir: `/{id}`
- Action adı eklenir (HTTP prefix ve `Async` suffix kaldırılır)

### Root Path Değiştirme

```csharp
PreConfigure<AbpAspNetCoreMvcOptions>(options =>
{
    options.ConventionalControllers.Create(typeof(BookStoreApplicationModule).Assembly, opts =>
    {
        opts.RootPath = "volosoft/book-store";
    });
});
// Route: /api/volosoft/book-store/book/{id}
```

### RemoteService Attribute

```csharp
// API controller olarak expose etme
[RemoteService(IsEnabled = false)]
public class PersonAppService : ApplicationService { }

// Sadece belirli metodları disable et
[RemoteService(IsEnabled = false)]
public override Task DeleteAsync(Guid id) { }
```

### IRemoteService Interface

```csharp
// Application service olmayan class'ları API controller yap
public class MyCustomService : IRemoteService, ITransientDependency
{
    public Task<string> GetDataAsync() => Task.FromResult("data");
}
```

---

## Dynamic C# API Client Proxies

Runtime'da otomatik C# client proxy oluşturur.

### Kurulum

```bash
abp add-package Volo.Abp.Http.Client
```

### Konfigürasyon

```csharp
[DependsOn(
    typeof(AbpHttpClientModule),
    typeof(BookStoreApplicationContractsModule)
)]
public class MyClientModule : AbpModule
{
    public override void ConfigureServices(ServiceConfigurationContext context)
    {
        context.Services.AddHttpClientProxies(
            typeof(BookStoreApplicationContractsModule).Assembly
        );
    }
}
```

### Endpoint Konfigürasyonu

```json
{
  "RemoteServices": {
    "Default": {
      "BaseUrl": "http://localhost:53929/"
    }
  }
}
```

### Kullanım

```csharp
public class MyClientService : ITransientDependency
{
    private readonly IBookAppService _bookAppService;
    public MyClientService(IBookAppService bookAppService) => _bookAppService = bookAppService;

    public async Task<List<BookDto>> GetBooksAsync()
    {
        return await _bookAppService.GetListAsync();
    }
}
```

**Otomatik yönetilen:**
- HTTP method, route, query string mapping
- Authentication (access token header'a eklenir)
- JSON serialization/deserialization
- API versioning
- Correlation ID, tenant ID, culture header'ları
- Error handling (proper exception'lar fırlatılır)

---

## Static C# API Client Proxies

Development-time'da proxy kodu generate edilir (daha iyi performans).

### CLI ile Generate

```bash
abp generate-proxy -t csharp
abp generate-proxy -t csharp --without-contracts  # Sadece client proxy, contracts olmadan
abp generate-proxy -t csharp --folder MyProxies   # Özel klasör
```

### Manuel Generate

```csharp
// HttpApi.Client projesinde
[DependsOn(typeof(AbpHttpClientModule))]
public class MyClientModule : AbpModule
{
    public override void ConfigureServices(ServiceConfigurationContext context)
    {
        context.Services.AddStaticHttpClientProxies(
            typeof(BookStoreApplicationContractsModule).Assembly
        );
    }
}
```

### Dynamic vs Static

| Özellik | Dynamic | Static |
|---|---|---|
| Performans | Runtime generate | Compile-time |
| Geliştirme | Kolay, otomatik | Re-generate gerekli |
| API değişikliği | Otomatik algılar | Manuel re-generate |

---

## Swagger

### Konfigürasyon

```csharp
[DependsOn(typeof(AbpAspNetCoreMvcModule))]
public class MyHttpApiModule : AbpModule
{
    public override void ConfigureServices(ServiceConfigurationContext context)
    {
        context.Services.AddAbpSwaggerGen(options =>
        {
            options.SwaggerDoc("v1", new OpenApiInfo
            {
                Title = "My API",
                Version = "v1",
                Description = "My API Description"
            });
            options.DocInclusionPredicate((docName, description) => true);
            options.CustomSchemaIds(type => type.FullName);
        });
    }

    public override void OnApplicationInitialization(ApplicationInitializationContext context)
    {
        var app = context.GetApplicationBuilder();
        app.UseSwagger();
        app.UseAbpSwaggerUI(options =>
        {
            options.SwaggerEndpoint("/swagger/v1/swagger.json", "My API v1");
        });
    }
}
```

### JWT Authentication ile Swagger

```csharp
options.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
{
    Description = "JWT Authorization header using the Bearer scheme.",
    Name = "Authorization",
    In = ParameterLocation.Header,
    Type = SecuritySchemeType.ApiKey,
    Scheme = "Bearer"
});

options.AddSecurityRequirement(new OpenApiSecurityRequirement
{
    {
        new OpenApiSecurityScheme
        {
            Reference = new OpenApiReference
            {
                Type = ReferenceType.SecurityScheme,
                Id = "Bearer"
            }
        },
        Array.Empty<string>()
    }
});
```

---

## API Versioning

```csharp
// Versioned application service
[RemoteService]
[ApiVersion("1.0")]
[ApiVersion("2.0")]
public class BookAppService : ApplicationService, IBookAppService
{
    [Obsolete("Use the new version instead")]
    public virtual Task<BookDto> GetAsync(Guid id) { }
}

// Versioned controller
[Area("app")]
[Route("api/v{version:apiVersion}/app/book")]
[ApiVersion("1.0")]
[ApiVersion("2.0")]
public class BookController : AbpController { }
```

### Swagger ile Versioning

```csharp
context.Services.AddAbpSwaggerGenWithVersioning(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo { Title = "My API", Version = "v1" });
    options.SwaggerDoc("v2", new OpenApiInfo { Title = "My API", Version = "v2" });
});
```

---

## Integration Services

Integration service'ler modüller arası doğrudan iletişim için kullanılır (HTTP yerine).

```csharp
// Integration service interface (Application.Contracts'ta)
public interface IProductIntegrationService : IIntegrationService
{
    Task<ProductDto> GetProductAsync(Guid id);
    Task CreateProductAsync(CreateProductDto input);
}

// Implementation (Application'da)
[ExposeServices(typeof(IProductIntegrationService))]
public class ProductIntegrationService : ApplicationService, IProductIntegrationService
{
    private readonly IRepository<Product, Guid> _productRepository;
    public ProductIntegrationService(IRepository<Product, Guid> productRepository) =>
        _productRepository = productRepository;

    public async Task<ProductDto> GetProductAsync(Guid id)
    {
        var product = await _productRepository.GetAsync(id);
        return ObjectMapper.Map<Product, ProductDto>(product);
    }

    public async Task CreateProductAsync(CreateProductDto input)
    {
        var product = ObjectMapper.Map<CreateProductDto, Product>(input);
        await _productRepository.InsertAsync(product);
    }
}
```

**IIntegrationService** marker interface'i ile ABP bu servisleri otomatik olarak:
- Local event bus üzerinden çağırır (modular monolith)
- Distributed event bus üzerinden çağırır (microservice, provider yapılandırılmışsa)

---

## Best Practices

1. **Auto API Controllers kullan** — Manuel controller yazma, convention yeterli
2. **Dynamic client proxy kullan** — `HttpClient` manuel kullanma
3. **Static proxy'yi production'da tercih et** — Runtime overhead yok
4. **Swagger'da JWT authentication ekle** — API test kolaylığı
5. **API versioning ile backward compatibility sağla** — `[Obsolete]` ile eski versiyonları işaretle
6. **Integration services ile modüller arası iletişim kur** — HTTP yerine doğrudan çağrı
7. **RemoteService attribute ile API exposure kontrol et** — Gereksiz endpoint'leri kapat
8. **Root path'i modül bazlı ayarla** — `/api/app` yerine `/api/my-module`

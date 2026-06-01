---
name: abp-api
description: "ABP Framework v10.4 API development: Auto API Controllers, dynamic and static C#/JS client proxies, Swagger, API versioning, Integration Services. Use when you need a REST API, controller, dynamic proxy or client generation in ABP."
---

# ABP Framework — API Development

Guide to ABP Framework v10.4 API development. Auto API Controllers, Dynamic C# Clients, Static C# Clients, Swagger, API Versioning, Integration Services.

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

ABP automatically converts application services into REST API endpoints.

### Configuration

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
| Other | POST (default) |

### Route Computation

| Service Method | HTTP Method | Route |
|---|---|---|
| `GetAsync(Guid id)` | GET | `/api/app/book/{id}` |
| `GetListAsync()` | GET | `/api/app/book` |
| `CreateAsync(CreateBookDto input)` | POST | `/api/app/book` |
| `UpdateAsync(Guid id, UpdateBookDto input)` | PUT | `/api/app/book/{id}` |
| `DeleteAsync(Guid id)` | DELETE | `/api/app/book/{id}` |
| `GetEditorsAsync(Guid id)` | GET | `/api/app/book/{id}/editors` |

**Route rules:**
- Always starts with `/api`
- Default root path: `/app`
- The controller name is normalized: `BookAppService` → `book` (kebab-case, suffixes removed)
- The `id` parameter is added to the route: `/{id}`
- The action name is added (HTTP prefix and `Async` suffix removed)

### Changing the Root Path

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
// Don't expose as an API controller
[RemoteService(IsEnabled = false)]
public class PersonAppService : ApplicationService { }

// Disable only specific methods
[RemoteService(IsEnabled = false)]
public override Task DeleteAsync(Guid id) { }
```

### IRemoteService Interface

```csharp
// Turn non-application-service classes into API controllers
public class MyCustomService : IRemoteService, ITransientDependency
{
    public Task<string> GetDataAsync() => Task.FromResult("data");
}
```

---

## Dynamic C# API Client Proxies

Automatically creates a C# client proxy at runtime.

### Installation

```bash
abp add-package Volo.Abp.Http.Client
```

### Configuration

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

### Endpoint Configuration

```json
{
  "RemoteServices": {
    "Default": {
      "BaseUrl": "http://localhost:53929/"
    }
  }
}
```

### Usage

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

**Handled automatically:**
- HTTP method, route, query string mapping
- Authentication (the access token is added to the header)
- JSON serialization/deserialization
- API versioning
- Correlation ID, tenant ID, culture headers
- Error handling (proper exceptions are thrown)

---

## Static C# API Client Proxies

Proxy code is generated at development time (better performance).

### Generate with the CLI

```bash
abp generate-proxy -t csharp
abp generate-proxy -t csharp --without-contracts  # Only the client proxy, without contracts
abp generate-proxy -t csharp --folder MyProxies   # Custom folder
```

### Manual Generation

```csharp
// In the HttpApi.Client project
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

| Feature | Dynamic | Static |
|---|---|---|
| Performance | Runtime generation | Compile-time |
| Development | Easy, automatic | Re-generation required |
| API change | Detected automatically | Manual re-generation |

---

## Swagger

### Configuration

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

### Swagger with JWT Authentication

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

### Versioning with Swagger

```csharp
context.Services.AddAbpSwaggerGenWithVersioning(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo { Title = "My API", Version = "v1" });
    options.SwaggerDoc("v2", new OpenApiInfo { Title = "My API", Version = "v2" });
});
```

---

## Integration Services

Integration services are used for direct communication between modules (instead of HTTP).

```csharp
// Integration service interface (in Application.Contracts)
public interface IProductIntegrationService : IIntegrationService
{
    Task<ProductDto> GetProductAsync(Guid id);
    Task CreateProductAsync(CreateProductDto input);
}

// Implementation (in Application)
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

With the **IIntegrationService** marker interface, ABP automatically calls these services:
- Over the local event bus (modular monolith)
- Over the distributed event bus (microservice, if a provider is configured)

---

## Best Practices

1. **Use Auto API Controllers** — Don't write controllers manually, the convention is enough
2. **Use dynamic client proxies** — Don't use `HttpClient` manually
3. **Prefer static proxies in production** — No runtime overhead
4. **Add JWT authentication in Swagger** — Easier API testing
5. **Ensure backward compatibility with API versioning** — Mark old versions with `[Obsolete]`
6. **Communicate between modules with integration services** — Direct calls instead of HTTP
7. **Control API exposure with the RemoteService attribute** — Close off unnecessary endpoints
8. **Set the root path per module** — `/api/my-module` instead of `/api/app`

---

## Related

- [DDD](../abp-ddd/SKILL.md) — application service (the source of Auto API Controllers)
- [UI](../abp-ui/SKILL.md) — proxy consumption (Angular/Blazor/React)
- [Microservices](../abp-microservices/SKILL.md) — Integration Services, static HTTP client proxy
- [Authorization](../abp-authorization/SKILL.md) — endpoint authorization
- ABP Docs: https://abp.io/docs/latest/framework/api-development

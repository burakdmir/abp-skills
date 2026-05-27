# ABP Framework — API Development

ABP Framework v10.4 API geliştirme. Auto API Controllers, Dynamic/Static C# Clients, Swagger, API Versioning.

## Trigger

- "ABP API controller"
- "ABP auto controller"
- "ABP dynamic client"
- "ABP swagger"
- "ABP API versioning"
- "ABP REST API"

## Auto API Controllers

```csharp
PreConfigure<AbpAspNetCoreMvcOptions>(options =>
{
    options.ConventionalControllers.Create(typeof(BookStoreApplicationModule).Assembly);
});
```

### HTTP Method Mapping

| Prefix | HTTP |
|---|---|
| `GetList`, `GetAll`, `Get` | GET |
| `Put`, `Update` | PUT |
| `Delete`, `Remove` | DELETE |
| `Create`, `Add`, `Insert`, `Post` | POST |
| `Patch` | PATCH |

### Route Örnekleri

| Method | Route |
|---|---|
| `GetAsync(Guid id)` | `/api/app/book/{id}` |
| `GetListAsync()` | `/api/app/book` |
| `CreateAsync(CreateBookDto input)` | `/api/app/book` |
| `UpdateAsync(Guid id, UpdateBookDto input)` | `/api/app/book/{id}` |
| `DeleteAsync(Guid id)` | `/api/app/book/{id}` |

### RemoteService

```csharp
[RemoteService(IsEnabled = false)]  // API olarak expose etme
public class PersonAppService : ApplicationService { }
```

## Dynamic C# Client Proxies

```bash
abp add-package Volo.Abp.Http.Client
```

```csharp
context.Services.AddHttpClientProxies(typeof(BookStoreApplicationContractsModule).Assembly);
```

```json
{ "RemoteServices": { "Default": { "BaseUrl": "http://localhost:53929/" } } }
```

```csharp
// Kullanım — interface inject et
var books = await _bookAppService.GetListAsync();
```

## Static C# Client Proxies

```bash
abp generate-proxy -t csharp
abp generate-proxy -t csharp --without-contracts
```

## Swagger

```csharp
context.Services.AddAbpSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo { Title = "My API", Version = "v1" });
});

// Init
app.UseSwagger();
app.UseAbpSwaggerUI(options => options.SwaggerEndpoint("/swagger/v1/swagger.json", "v1"));
```

## API Versioning

```csharp
[ApiVersion("1.0")]
[ApiVersion("2.0")]
public class BookAppService : ApplicationService, IBookAppService { }
```

## Best Practices

1. Auto API Controllers kullan — manuel controller yazma
2. Dynamic client proxy kullan — HttpClient manuel kullanma
3. Static proxy production'da tercih et
4. Swagger'da JWT auth ekle
5. API versioning ile backward compatibility sağla

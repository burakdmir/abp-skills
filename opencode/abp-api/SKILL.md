---
name: abp-api
description: "ABP Framework v10.4 API development: Auto API Controllers, dynamic and static C#/JS client proxies, Swagger, API versioning, Integration Services. Use when you need a REST API, controller, dynamic proxy or client generation in ABP."
---

# ABP Framework — API Development

ABP Framework v10.4 API development. Auto API Controllers, Dynamic/Static C# Clients, Swagger, API Versioning.

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

### Route Examples

| Method | Route |
|---|---|
| `GetAsync(Guid id)` | `/api/app/book/{id}` |
| `GetListAsync()` | `/api/app/book` |
| `CreateAsync(CreateBookDto input)` | `/api/app/book` |
| `UpdateAsync(Guid id, UpdateBookDto input)` | `/api/app/book/{id}` |
| `DeleteAsync(Guid id)` | `/api/app/book/{id}` |

### RemoteService

```csharp
[RemoteService(IsEnabled = false)]  // Don't expose as API
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
// Usage — inject the interface
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

1. Use Auto API Controllers — don't write controllers manually
2. Use dynamic client proxies — don't use HttpClient manually
3. Prefer static proxies in production
4. Add JWT auth in Swagger
5. Ensure backward compatibility with API versioning

## Related

[DDD](../abp-ddd/SKILL.md) · [UI](../abp-ui/SKILL.md) · [Microservices](../abp-microservices/SKILL.md) · [Authorization](../abp-authorization/SKILL.md) · Docs: https://abp.io/docs/latest/framework/api-development

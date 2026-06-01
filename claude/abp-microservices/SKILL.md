---
name: abp-microservices
description: "ABP Framework v10.4 microservice solution: structure (apps/gateways/services), Integration Services [IntegrationService], distributed events (RabbitMQ Outbox/Inbox), YARP gateway, OpenIddict auth server, Entity Cache, database-per-service. Use when you need microservices or inter-service communication in ABP."
---

# ABP Framework — Microservices

ABP Framework v10.4 microservice solution template (Business+ license). A distributed system with synchronous (Integration Services) and asynchronous (distributed events) inter-service communication, a YARP gateway, and an OpenIddict auth server.

## Trigger

- "ABP microservice"
- "ABP microservices"
- "ABP integration service"
- "ABP inter-service communication"
- "ABP distributed event"
- "ABP YARP gateway"
- "ABP auth server"

## Solution Structure

```
MyMicroservice/
├── apps/                      # UI applications
│   ├── web/                   # Web application
│   ├── public-web/            # Public site
│   └── auth-server/           # Authentication server (OpenIddict)
├── gateways/                  # BFF — one gateway per UI
│   └── web-gateway/           # YARP reverse proxy
├── services/                  # Microservices
│   ├── administration/        # Permission, setting, feature
│   ├── identity/              # User, role
│   └── [business-services]/   # Your own business services
└── etc/
    ├── docker/                # docker compose for local infra
    └── helm/                  # Kubernetes deployment
```

## Microservice Structure (NOT Layered!)

Each microservice has a simplified single-project structure:

```
services/ordering/
├── OrderingService/                # Main project
│   ├── Entities/
│   ├── Services/
│   ├── IntegrationServices/        # For inter-service communication
│   ├── Data/                       # DbContext (IHasEventInbox, IHasEventOutbox)
│   └── OrderingServiceModule.cs
├── OrderingService.Contracts/      # Interface, DTO, ETO (shared)
└── OrderingService.Tests/
```

## Synchronous Communication — Integration Services

For synchronous inter-service calls, use an **Integration Service, not a regular application service**.

### 1. Provider — define the Integration Service

```csharp
// In the CatalogService.Contracts project
[IntegrationService]
public interface IProductIntegrationService : IApplicationService
{
    Task<List<ProductDto>> GetProductsByIdsAsync(List<Guid> ids);
}

// In the CatalogService project
[IntegrationService]
public class ProductIntegrationService : ApplicationService, IProductIntegrationService
{
    public async Task<List<ProductDto>> GetProductsByIdsAsync(List<Guid> ids)
    {
        var products = await _productRepository.GetListAsync(p => ids.Contains(p.Id));
        return ObjectMapper.Map<List<Product>, List<ProductDto>>(products);
    }
}
```

### 2. Provider — expose the Integration Services

```csharp
// CatalogServiceModule.cs
Configure<AbpAspNetCoreMvcOptions>(options =>
{
    options.ExposeIntegrationServices = true;
});
```

### 3-4. Consumer — reference Contracts + generate proxy

```bash
abp generate-proxy -t csharp -u http://localhost:44361 -m catalog --without-contracts
```

### 5. Consumer — register the HTTP client proxy

```csharp
[DependsOn(typeof(CatalogServiceContractsModule))]
public class OrderingServiceModule : AbpModule
{
    public override void ConfigureServices(ServiceConfigurationContext context)
    {
        context.Services.AddStaticHttpClientProxies(
            typeof(CatalogServiceContractsModule).Assembly, "CatalogService");
    }
}
```

### 6. Consumer — Remote service URL

```json
// appsettings.json
"RemoteServices": {
    "CatalogService": { "BaseUrl": "http://localhost:44361" }
}
```

### 7. Use it

```csharp
public class OrderAppService : ApplicationService
{
    private readonly IProductIntegrationService _productIntegrationService;

    public async Task<List<OrderDto>> GetListAsync()
    {
        var orders = await _orderRepository.GetListAsync();
        var productIds = orders.Select(o => o.ProductId).Distinct().ToList();
        var products = await _productIntegrationService.GetProductsByIdsAsync(productIds);
        // ...
    }
}
```

> **Why an Integration Service?** Application services are for the UI (with different authorization/validation/optimization needs). Integration services are designed for service-to-service communication.
>
> **When:** Immediate response + data needed to complete the current operation (e.g. product details to show in an order list).

## Asynchronous Communication — Distributed Events (RabbitMQ)

For loosely coupled state notifications.

```csharp
// ETO — in the Contracts project
[EventName("Product.StockChanged")]
public class StockCountChangedEto { public Guid ProductId { get; set; } public int NewCount { get; set; } }

// Publish
await _distributedEventBus.PublishAsync(new StockCountChangedEto { ... });

// Subscribe (in another service)
public class StockChangedHandler : IDistributedEventHandler<StockCountChangedEto>, ITransientDependency
{
    public async Task HandleEventAsync(StockCountChangedEto eventData) { }
}
```

> The DbContext must implement `IHasEventInbox` and `IHasEventOutbox` for the Outbox/Inbox pattern.
>
> **When:** State change notifications (order placed, stock updated), operations that don't require an immediate response and where services should remain independent.

## Performance — Entity Cache

```csharp
// Registration
context.Services.AddEntityCache<Product, ProductDto, Guid>();

// Usage (automatically invalidated when the entity changes)
private readonly IEntityCache<ProductDto, Guid> _productCache;
public Task<ProductDto> GetProductAsync(Guid id) => _productCache.GetAsync(id);
```

## Built-in Infrastructure

- **RabbitMQ** — Distributed events (Outbox/Inbox)
- **Redis** — Distributed cache + locking
- **YARP** — API Gateway
- **OpenIddict** — Auth server

## Best Practices

1. **Choose the right communication** — synchronous: queries needing immediate data; asynchronous: notification/state change
2. **Use Integration Services** — not application services for inter-service calls
3. **Cache remote data** — Entity Cache / IDistributedCache
4. **Share only Contracts** — never share the implementation
5. **Idempotent handlers** — events can be delivered multiple times
6. **Database-per-service** — each service owns its own database

## Related

- [Framework Core](../abp-framework/SKILL.md) — solution templates
- [API](../abp-api/SKILL.md) — Integration Services, dynamic proxy
- [Infrastructure](../abp-infrastructure/SKILL.md) — distributed event bus, Redis cache
- [Authorization](../abp-authorization/SKILL.md) — OpenIddict, permission
- [Deployment](../abp-deployment/SKILL.md) — Docker, Kubernetes/Helm, gateway
- ABP Docs: https://abp.io/docs/latest/solution-templates/microservice

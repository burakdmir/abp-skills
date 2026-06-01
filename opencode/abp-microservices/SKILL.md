---
name: abp-microservices
description: "ABP Framework v10.4 microservice quick reference: Integration Services [IntegrationService], distributed events (RabbitMQ Outbox/Inbox), YARP gateway, OpenIddict, Entity Cache, database-per-service. Use when you need microservices or inter-service communication in ABP."
---

# ABP Framework — Microservices

ABP v10.4 microservice template (Business+). Synchronous = Integration Services, asynchronous = distributed events. YARP gateway + OpenIddict auth server.

## Trigger

"ABP microservices", "integration service", "inter-service communication", "distributed event", "YARP gateway", "auth server".

## Structure

```
apps/{web,public-web,auth-server}  gateways/web-gateway(YARP)
services/{administration,identity,...}  etc/{docker,helm}
```
Services are NOT layered: a single project + `*.Contracts` (interface/DTO/ETO) + `*.Tests`. DbContext: `IHasEventInbox`, `IHasEventOutbox`.

## Synchronous — Integration Service

```csharp
// Contracts
[IntegrationService]
public interface IProductIntegrationService : IApplicationService
{ Task<List<ProductDto>> GetProductsByIdsAsync(List<Guid> ids); }

// Service
[IntegrationService]
public class ProductIntegrationService : ApplicationService, IProductIntegrationService { /* ... */ }

// Provider module: options.ExposeIntegrationServices = true;
// Consumer module: context.Services.AddStaticHttpClientProxies(typeof(CatalogServiceContractsModule).Assembly, "CatalogService");
```
```bash
abp generate-proxy -t csharp -u http://localhost:44361 -m catalog --without-contracts
```
```json
"RemoteServices": { "CatalogService": { "BaseUrl": "http://localhost:44361" } }
```
> Use an Integration Service, not an application service. When: immediate response + data needed for the operation.

## Asynchronous — Distributed Event (RabbitMQ)

```csharp
[EventName("Product.StockChanged")]
public class StockCountChangedEto { public Guid ProductId { get; set; } public int NewCount { get; set; } }

await _distributedEventBus.PublishAsync(new StockCountChangedEto { ... });

public class Handler : IDistributedEventHandler<StockCountChangedEto>, ITransientDependency
{ public async Task HandleEventAsync(StockCountChangedEto e) { } }
```
> When: state change notifications, loose coupling. Handlers must be idempotent.

## Entity Cache

```csharp
context.Services.AddEntityCache<Product, ProductDto, Guid>();
private readonly IEntityCache<ProductDto, Guid> _cache; // _cache.GetAsync(id)
```

Built-in infrastructure: RabbitMQ, Redis, YARP, OpenIddict.

## Best Practices

1. Choose the right communication (synchronous query / asynchronous notification)
2. Inter-service → Integration Service
3. Share only Contracts, idempotent handlers
4. Database-per-service, cache remote data

## Related

- [Framework](../abp-framework/SKILL.md) · [API](../abp-api/SKILL.md) · [Infrastructure](../abp-infrastructure/SKILL.md) · [Deployment](../abp-deployment/SKILL.md)
- ABP Docs: https://abp.io/docs/latest/solution-templates/microservice

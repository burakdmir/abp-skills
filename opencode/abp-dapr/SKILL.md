---
name: abp-dapr
description: "ABP Framework v10.x (10.4–10.6) Dapr quick reference: Volo.Abp.Dapr packages, C# client proxies over Dapr service invocation, distributed event bus via Dapr pub/sub, AbpDaprOptions, API token security. Use when you need Dapr sidecar communication, Dapr pub/sub, or Dapr service invocation in ABP."
---

# ABP Framework — Dapr Integration

ABP v10.x + Dapr. Integration packages cover the intersection points: service invocation, pub/sub event bus, distributed lock. Everything else → plain Dapr API.

## Trigger

"ABP Dapr", "Dapr service invocation", "Dapr pub/sub", "Dapr event bus", "DaprClient", "Dapr sidecar".

## Packages

- `Volo.Abp.Dapr` — core (`AbpDaprModule`); others depend on it
- `Volo.Abp.Http.Client.Dapr` — dynamic/static C# client proxies over service invocation (`AbpHttpClientDaprModule`)
- `Volo.Abp.EventBus.Dapr` — event bus, **publish only** (`AbpEventBusDaprModule`)
- `Volo.Abp.AspNetCore.Mvc.Dapr.EventBus` — publish + receive; references the previous one (`AbpAspNetCoreMvcDaprEventBusModule`)
- `Volo.Abp.DistributedLocking.Dapr` — distributed lock (Alpha — ABP recommends the DistributedLock library instead)

Install: `abp add-package <PackageName>` (or NuGet + `[DependsOn(...)]`).

## Configuration

```csharp
Configure<AbpDaprOptions>(o => { /* HttpEndpoint, GrpcEndpoint, DaprApiToken, AppApiToken — all optional */ });
Configure<AbpDaprEventBusOptions>(o => { o.PubSubName = "pubsub"; }); // default "pubsub"
```
Or `appsettings.json`: `"Dapr": { "HttpEndpoint": "http://localhost:3500/" }`.

`IAbpDaprClientFactory` (recommended, uses AbpDaprOptions):
```csharp
DaprClient daprClient = await _daprClientFactory.CreateAsync();
HttpClient httpClient = await _daprClientFactory.CreateHttpClientAsync("target-app-id");
```

## Service Invocation — Client Proxies

Install `Volo.Abp.Http.Client.Dapr` on the client side; set `BaseUrl` to the Dapr **app-id**:

```json
"RemoteServices": { "Default": { "BaseUrl": "http://dapr-httpapi/" } }
```
Remote service name must match `AddHttpClientProxies` / `AddStaticHttpClientProxies`. Proxy calls then route through Dapr automatically — no code change.

## Distributed Event Bus — Dapr Pub/Sub

ASP.NET Core send+receive → `Volo.Abp.AspNetCore.Mvc.Dapr.EventBus`. Non-ASP.NET Core → `Volo.Abp.EventBus.Dapr` (publish only).

ABP endpoints: `dapr/subscribe` (subscription list) and `api/abp/dapr/event` (unified receiver). ABP calls `MapSubscribeHandler` internally — don't call it yourself. `app.UseCloudEvents()` optional for CloudEvents.

```csharp
// ABP way — unchanged code, outbox/inbox preserved
await _distributedEventBus.PublishAsync(new StockCountChangedEto { ProductCode = "AT837234", NewStockCount = 42 });

public class MyHandler : IDistributedEventHandler<StockCountChangedEto>, ITransientDependency
{ public async Task HandleEventAsync(StockCountChangedEto e) { } }

// Dapr API directly (bypasses outbox/inbox)
await _daprClient.PublishEventAsync("pubsub", "StockChanged", eto);

[HttpPost("/stock-changed")] [Topic("pubsub", "StockChanged")]
public async Task<IActionResult> OnStockChanged([FromBody] StockCountChangedEto model)
{ HttpContext.ValidateDaprAppApiToken(); return Ok(); }
```

## Security

- **Dapr API token** (app → Dapr): `AbpDaprOptions.DaprApiToken`, default from `DAPR_API_TOKEN` env var; read via `IDaprApiTokenProvider`.
- **App API token** (Dapr → app): `AbpDaprOptions.AppApiToken`, default from `APP_API_TOKEN` env var. Validate with `HttpContext.ValidateDaprAppApiToken()` (throws `AbpAuthorizationException` on bad/missing `dapr-api-token` header) or `IDaprAppApiTokenValidator`. Strongly recommended — protects the event endpoints.

## Best Practices

1. Send+receive events → only `Volo.Abp.AspNetCore.Mvc.Dapr.EventBus` (brings the publish package)
2. Prefer `IDistributedEventBus` over raw `DaprClient` publishing — keeps outbox/inbox
3. `RemoteServices:BaseUrl` = Dapr app-id, not a real host
4. Validate the App API token on Dapr-called endpoints; never call `MapSubscribeHandler` manually
5. Skip Dapr distributed lock (Alpha) — use the DistributedLock library

## Related

- [Microservices](../abp-microservices/SKILL.md) · [Infrastructure](../abp-infrastructure/SKILL.md) · [API](../abp-api/SKILL.md)
- ABP Docs: https://abp.io/docs/latest/framework/dapr

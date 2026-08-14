---
name: abp-dapr
description: "ABP Framework v10.x (10.4–10.6) Dapr integration: Volo.Abp.Dapr packages, service invocation via C# API client proxies (Volo.Abp.Http.Client.Dapr), distributed event bus over Dapr pub/sub (Volo.Abp.EventBus.Dapr + Volo.Abp.AspNetCore.Mvc.Dapr.EventBus), distributed lock, AbpDaprOptions, API token security. Use when you need Dapr sidecar communication, Dapr pub/sub, or Dapr service invocation in ABP."
---

# ABP Framework — Dapr Integration

ABP Framework v10.x (10.4–10.6) integration with [Dapr](https://dapr.io/) (Distributed Application Runtime). ABP and Dapr intersect on service-to-service communication, distributed message bus, and distributed locking — ABP provides integration packages exactly at these intersection points. Other Dapr building blocks are used directly via Dapr's own documentation.

## Trigger

- "ABP Dapr"
- "Dapr integration"
- "Dapr service invocation"
- "Dapr pub/sub"
- "Dapr event bus"
- "Dapr sidecar"
- "DaprClient"

## ABP Dapr Integration Packages

| Package | Purpose |
|---|---|
| `Volo.Abp.Dapr` | Main integration package — all others depend on it (`AbpDaprModule`) |
| `Volo.Abp.Http.Client.Dapr` | Dynamic/static C# API client proxies over Dapr service invocation (`AbpHttpClientDaprModule`) |
| `Volo.Abp.EventBus.Dapr` | Distributed event bus via Dapr pub/sub — **publish only** (`AbpEventBusDaprModule`) |
| `Volo.Abp.AspNetCore.Mvc.Dapr.EventBus` | Subscription endpoints to **receive** events — publish + subscribe (`AbpAspNetCoreMvcDaprEventBusModule`) |
| `Volo.Abp.DistributedLocking.Dapr` | ABP distributed locking over Dapr's distributed lock block (`AbpDistributedLockingDaprModule`) |

Installation (any of them):

```bash
abp add-package Volo.Abp.Dapr
```

Or manually: install the NuGet package + add `[DependsOn(typeof(AbpDaprModule))]` to your module class.

## Configuration — AbpDaprOptions

All settings are optional; you mostly don't need to configure them.

```csharp
Configure<AbpDaprOptions>(options =>
{
    // options.HttpEndpoint  — HTTP endpoint used when creating DaprClient
    // options.GrpcEndpoint  — gRPC endpoint used when creating DaprClient
    // options.DaprApiToken  — app → Dapr requests (default: DAPR_API_TOKEN env var)
    // options.AppApiToken   — validates Dapr → app requests (default: APP_API_TOKEN env var)
});
```

Or via `appsettings.json`:

```json
"Dapr": {
  "HttpEndpoint": "http://localhost:3500/"
}
```

## IAbpDaprClientFactory

Creates `DaprClient` or `HttpClient` objects using `AbpDaprOptions` — central configuration. Recommended, but not required (you can use the Dapr API directly).

```csharp
public class MyService : ITransientDependency
{
    private readonly IAbpDaprClientFactory _daprClientFactory;

    public MyService(IAbpDaprClientFactory daprClientFactory)
    {
        _daprClientFactory = daprClientFactory;
    }

    public async Task DoItAsync()
    {
        // DaprClient with default options
        DaprClient daprClient = await _daprClientFactory.CreateAsync();

        // DaprClient with DaprClientBuilder configuration
        DaprClient daprClient2 = await _daprClientFactory
            .CreateAsync(builder =>
            {
                builder.UseDaprApiToken("...");
            });

        // HttpClient targeting a Dapr app-id
        HttpClient httpClient = await _daprClientFactory.CreateHttpClientAsync("target-app-id");
    }
}
```

`CreateHttpClientAsync` also accepts optional `daprEndpoint` and `daprApiToken` parameters.

## Service Invocation — C# API Client Proxies over Dapr

ABP's [dynamic](https://abp.io/docs/latest/framework/api-development/dynamic-csharp-clients) and [static](https://abp.io/docs/latest/framework/api-development/static-csharp-clients) C# client proxies can route their HTTP calls through Dapr's service invocation building block. Install `Volo.Abp.Http.Client.Dapr` on the **client side** — then the only remaining step is remote service configuration.

```json
{
  "RemoteServices": {
    "Default": {
      "BaseUrl": "http://dapr-httpapi/"
    }
  }
}
```

- `dapr-httpapi` is the **application id** of the server application in your Dapr configuration — not a real host name.
- The remote service name (`Default` here) must match the name given in `AddHttpClientProxies` (dynamic) or `AddStaticHttpClientProxies` (static). Multiple servers → multiple keys under `RemoteServices`.
- After that, your existing client proxy code works unchanged: calls automatically go through Dapr.

## Distributed Event Bus over Dapr Pub/Sub

Two packages implement ABP's distributed event bus with Dapr's publish & subscribe building block:

- **ASP.NET Core app, send + receive** → install `Volo.Abp.AspNetCore.Mvc.Dapr.EventBus` (it already references `Volo.Abp.EventBus.Dapr` — don't install both).
- **Non-ASP.NET Core app (e.g. Console)** → install `Volo.Abp.EventBus.Dapr`; you can publish but **cannot receive** events via ABP's packages.

### Configuration

```csharp
Configure<AbpDaprEventBusOptions>(options =>
{
    options.PubSubName = "pubsub"; // pubsubName for DaprClient.PublishEventAsync; default: "pubsub"
});
```

### The ABP Subscription Endpoints

ABP automatically exposes:

- `dapr/subscribe` — Dapr calls this to get the subscription list. ABP returns all subscriptions for your distributed event handler classes and controller actions with the `Topic` attribute.
- `api/abp/dapr/event` — unified endpoint receiving all events from Dapr; ABP dispatches to handlers by topic name.

> ABP calls `MapSubscribeHandler` internally — **do not call it manually**. Add `app.UseCloudEvents()` middleware if you want CloudEvents standard support.

### Usage — The ABP Way

No application code changes: the standard `IDistributedEventBus` / `IDistributedEventHandler` pattern works over Dapr automatically.

```csharp
// Publish
public class MyService : ITransientDependency
{
    private readonly IDistributedEventBus _distributedEventBus;

    public MyService(IDistributedEventBus distributedEventBus)
    {
        _distributedEventBus = distributedEventBus;
    }

    public async Task DoItAsync()
    {
        await _distributedEventBus.PublishAsync(new StockCountChangedEto
        {
            ProductCode = "AT837234",
            NewStockCount = 42
        });
    }
}

// Subscribe — ABP auto-registers this with Dapr via dapr/subscribe
public class MyHandler :
    IDistributedEventHandler<StockCountChangedEto>,
    ITransientDependency
{
    public async Task HandleEventAsync(StockCountChangedEto eventData)
    {
        var productCode = eventData.ProductCode;
        // ...
    }
}
```

### Usage — The Dapr API Directly

You can also publish with `DaprClient` and subscribe with a `Topic`-attributed controller. Note: direct Dapr API publishing bypasses ABP's event bus features like the outbox/inbox pattern.

```csharp
// Publish with DaprClient
await _daprClient.PublishEventAsync(
    "pubsub",        // pubsub name
    "StockChanged",  // topic name
    new StockCountChangedEto { ProductCode = "AT837234", NewStockCount = 42 }
);

// Subscribe with a controller
public class MyController : AbpController
{
    [HttpPost("/stock-changed")]
    [Topic("pubsub", "StockChanged")]
    public async Task<IActionResult> TestRouteAsync([FromBody] StockCountChangedEto model)
    {
        HttpContext.ValidateDaprAppApiToken();

        // Do something with the event
        return Ok();
    }
}
```

## Distributed Lock over Dapr

> Dapr's distributed lock block is in **Alpha** — ABP recommends the [DistributedLock](https://github.com/madelson/DistributedLock) library instead for now.

Install `Volo.Abp.DistributedLocking.Dapr`, then:

```csharp
Configure<AbpDistributedLockDaprOptions>(options =>
{
    options.StoreName = "mystore"; // required — lock keys are scoped per store
    // options.Owner                     — DaprClient.Lock owner; default: random value
    // options.DefaultExpirationTimeout  — default: 2 minutes
});
```

Usage is the standard `IAbpDistributedLock.TryAcquireAsync("MyLockName")`. Dapr differences: the `timeout` parameter is ignored (Dapr doesn't support waiting for a lock), and locks auto-expire after `DefaultExpirationTimeout`.

## Security — API Tokens

Dapr uses two tokens to secure app ↔ Dapr communication:

- **Dapr API Token** (`AbpDaprOptions.DaprApiToken`) — sent by your app on requests **to** Dapr. Auto-filled from the `DAPR_API_TOKEN` environment variable; used by `IAbpDaprClientFactory`. Read it via `IDaprApiTokenProvider.GetDaprApiToken()`.
- **App API Token** (`AbpDaprOptions.AppApiToken`) — validates requests coming **from** Dapr. Auto-filled from the `APP_API_TOKEN` environment variable. Validate with `HttpContext.ValidateDaprAppApiToken()` (throws `AbpAuthorizationException` if the `dapr-api-token` header is missing/wrong; no-op if not configured) or inject `IDaprAppApiTokenValidator` in any service.

```csharp
Configure<AbpDaprOptions>(options =>
{
    options.AppApiToken = "..."; // or "Dapr": { "AppApiToken": "..." } in appsettings.json
});
```

> Enabling App API token validation is **strongly recommended** — otherwise any client can call your event subscription endpoint directly and fake an event.

## Best Practices

1. **One package per need** — event send+receive: only `Volo.Abp.AspNetCore.Mvc.Dapr.EventBus` (it brings `Volo.Abp.EventBus.Dapr` transitively)
2. **Prefer the ABP way for events** — `IDistributedEventBus` keeps outbox/inbox support; direct `DaprClient.PublishEventAsync` loses it
3. **Use `IAbpDaprClientFactory`** — central `AbpDaprOptions` configuration instead of hand-built clients
4. **`RemoteServices:BaseUrl` = Dapr app-id** — service invocation routes through the sidecar, not a host address
5. **Validate the App API token** — `ValidateDaprAppApiToken()` on every Dapr-called endpoint
6. **Never call `MapSubscribeHandler` manually** — ABP already does it
7. **Avoid Dapr distributed lock in production** — Alpha stage; use the DistributedLock library per ABP's recommendation

## Related

- [Microservices](../abp-microservices/SKILL.md) — inter-service communication, distributed events
- [Infrastructure](../abp-infrastructure/SKILL.md) — distributed event bus, distributed locking
- [API](../abp-api/SKILL.md) — dynamic/static C# client proxies
- ABP Docs: https://abp.io/docs/latest/framework/dapr

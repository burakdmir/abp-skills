---
name: abp-infrastructure
description: "ABP Framework v10.4 infrastructure: Distributed Event Bus, Background Jobs/Workers, Caching (Redis), BLOB Storing, Emailing, SignalR, IClock, Distributed Locking, Entity Cache. Use when you need an event bus, background job, cache, blob or email in ABP."
---

# ABP Framework — Infrastructure

ABP Framework v10.4 infrastructure components. Event Bus, Background Jobs, Caching, BLOB Storing, Emailing, Data Filtering, Data Seeding, Settings, Features, Virtual File System, Entity Cache, Distributed Locking.

## Trigger

- "ABP event bus"
- "ABP background job"
- "ABP cache"
- "ABP Redis"
- "ABP BLOB"
- "ABP email"
- "ABP data filter"
- "ABP data seeding"
- "ABP settings"
- "ABP features"
- "ABP virtual file"
- "ABP entity cache"
- "ABP distributed lock"
- "ABP current user"
- "ABP infrastructure"

## Event Bus

| Type | Interface | Usage |
|---|---|---|
| Local | `ILocalEventBus`, `ILocalEventHandler<TEvent>` | Same process |
| Distributed | `IDistributedEventBus`, `IDistributedEventHandler<TEvent>` | Across processes |

```csharp
// Event
public class StockCountChangedEvent { public Guid ProductId { get; set; } public int NewCount { get; set; } }

// Publish (service)
await _localEventBus.PublishAsync(new StockCountChangedEvent { ProductId = id, NewCount = count });

// Publish (entity)
public class Product : AggregateRoot<Guid>
{
    public void ChangeStock(int count) { StockCount = count; AddLocalEvent(new StockCountChangedEvent { ... }); }
}

// Handler
public class Handler : ILocalEventHandler<StockCountChangedEvent>, ITransientDependency
{
    [UnitOfWork]
    public virtual async Task HandleEventAsync(StockCountChangedEvent e) { /* logic */ }
}
```

**Distributed Providers:** Local (default), RabbitMQ, Kafka, Azure Service Bus, Rebus

**Rule:** Microservice → Distributed, Modular Monolith → Distributed (inter-module), Monolith → Local

## Background Jobs

```csharp
public class EmailSendingArgs { public string Email { get; set; } public string Subject { get; set; } public string Body { get; set; } }

public class EmailSendingJob : AsyncBackgroundJob<EmailSendingArgs>, ITransientDependency
{
    private readonly IEmailSender _emailSender;
    public EmailSendingJob(IEmailSender emailSender) => _emailSender = emailSender;
    public override async Task ExecuteAsync(EmailSendingArgs args) =>
        await _emailSender.SendAsync(args.Email, args.Subject, args.Body);
}

// Enqueue
await _backgroundJobManager.EnqueueAsync(new EmailSendingArgs { ... }, priority: BackgroundJobPriority.Normal, delay: TimeSpan.FromMinutes(5));
```

**Providers:** Default (in-memory/DB), Hangfire, Quartz, RabbitMQ

## Caching

```csharp
[CacheName("Books")]
public class BookCacheItem { public string Name { get; set; } public float Price { get; set; } }

public class BookService : ITransientDependency
{
    private readonly IDistributedCache<BookCacheItem, Guid> _cache;
    public BookService(IDistributedCache<BookCacheItem, Guid> cache) => _cache = cache;

    public async Task<BookCacheItem> GetAsync(Guid id) =>
        await _cache.GetOrAddAsync(id, async () => await GetFromDbAsync(id),
            () => new DistributedCacheEntryOptions { AbsoluteExpiration = DateTimeOffset.Now.AddHours(1) });
}
```

**Redis:** `abp add-package Volo.Abp.Caching.StackExchangeRedis`
```json
"Redis": { "IsEnabled": "true", "Configuration": "127.0.0.1" }
```
**Why the ABP Redis package?** Has `SetManyAsync`/`GetManyAsync` (not in Microsoft's), simple configuration.

**Batch:** `GetManyAsync`, `SetManyAsync`, `GetOrAddManyAsync`, `RemoveManyAsync`

**UOW-level:** `await _cache.SetAsync(key, value, considerUow: true)`

## BLOB Storing

```csharp
[BlobContainerName("product-images")]
public class ProductImageBlobContainer : AbpBlobContainer { }

// Usage
await _blobContainer.SaveAsync(productId.ToString(), imageBytes, true);
var bytes = await _blobContainer.GetAllAsync(productId.ToString());
```

**Providers:** FileSystem, Database, AWS S3, Azure, MinIO, Google, Alibaba, Bunny, Memory

## Emailing

```csharp
await _emailSender.SendAsync(to: email, subject: "Welcome!", body: "...", isBodyHtml: true);
```

## Data Filtering

```csharp
using (_dataFilter.Disable<ISoftDelete>()) { return await _repository.GetListAsync(); }
using (_dataFilter.Disable<IMultiTenant>()) { return await _repository.GetCountAsync(); }
```

## Data Seeding

```csharp
public class MyDataSeedContributor : IDataSeedContributor, ITransientDependency
{
    public async Task SeedAsync(DataSeedContext context)
    {
        if (await _roleRepository.FindAsync(x => x.Name == "Admin") == null)
            await _roleRepository.InsertAsync(new IdentityRole(GuidGenerator.Create(), "Admin"));
    }
}
```

## Settings

```csharp
public class MyAppSettings : SettingDefinitionProvider
{
    public override void Define(ISettingDefinitionContext context) =>
        context.Add(new SettingDefinition("MyApp.MaxPrice", defaultValue: "1000", isVisibleToClients: true));
}

// Usage
var max = await _settingProvider.GetOrNullAsync<decimal>("MyApp.MaxPrice");
await SettingManager.SetAsync("MyApp.MaxPrice", "2000");
```

## Features

```csharp
public class MyAppFeatures : FeatureDefinitionProvider
{
    public override void Define(IFeatureDefinitionContext context) =>
        context.Add(new FeatureDefinition("MyApp.Premium", defaultValue: "false"));
}

// Usage
var enabled = await _featureChecker.IsEnabledAsync("MyApp.Premium");
```

## Virtual File System

```csharp
Configure<AbpVirtualFileSystemOptions>(options => options.FileSets.AddEmbedded<MyModule>());
```

## Current User

```csharp
var userId = CurrentUser.Id;
var userName = CurrentUser.UserName;
var tenantId = CurrentUser.TenantId;
var roles = CurrentUser.Roles;
var isAuthenticated = CurrentUser.IsAuthenticated;
```

## Distributed Locking

```csharp
await using var handle = await _distributedLock.TryAcquireAsync("my-lock-key");
if (handle != null) { /* critical operation */ }
```

## Best Practices

1. Publish events from the entity, handle them in the service
2. Move long-running work to a background job
3. Use `GetOrAddAsync`, set expiration
4. Use the BLOB provider abstraction
5. Disable the data filter with `using`
6. Settings for runtime-changeable values
7. Features for per-tenant toggles

## Related

[DDD](../abp-ddd/SKILL.md) · [Microservices](../abp-microservices/SKILL.md) · [Deployment](../abp-deployment/SKILL.md) · [Settings & Features](../abp-settings-features/SKILL.md) · Docs: https://abp.io/docs/latest/framework/infrastructure

# ABP Framework — Infrastructure

ABP Framework v10.4 altyapı bileşenleri. Event Bus, Background Jobs, Caching, BLOB Storing, Emailing, Data Filtering, Data Seeding, Settings, Features, Virtual File System, Entity Cache, Distributed Locking.

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

| Tip | Interface | Kullanım |
|---|---|---|
| Local | `ILocalEventBus`, `ILocalEventHandler<TEvent>` | Aynı process |
| Distributed | `IDistributedEventBus`, `IDistributedEventHandler<TEvent>` | Process'ler arası |

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

**Distributed Provider'lar:** Local (default), RabbitMQ, Kafka, Azure Service Bus, Rebus

**Kural:** Microservice → Distributed, Modular Monolith → Distributed (inter-module), Monolith → Local

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

**Provider'lar:** Default (in-memory/DB), Hangfire, Quartz, RabbitMQ

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
**Neden ABP Redis paketi?** `SetManyAsync`/`GetManyAsync` var (Microsoft'ta yok), konfigürasyon basit.

**Batch:** `GetManyAsync`, `SetManyAsync`, `GetOrAddManyAsync`, `RemoveManyAsync`

**UOW-level:** `await _cache.SetAsync(key, value, considerUow: true)`

## BLOB Storing

```csharp
[BlobContainerName("product-images")]
public class ProductImageBlobContainer : AbpBlobContainer { }

// Kullanım
await _blobContainer.SaveAsync(productId.ToString(), imageBytes, true);
var bytes = await _blobContainer.GetAllAsync(productId.ToString());
```

**Provider'lar:** FileSystem, Database, AWS S3, Azure, MinIO, Google, Alibaba, Bunny, Memory

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

// Kullanım
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

// Kullanım
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
if (handle != null) { /* kritik işlem */ }
```

## Best Practices

1. Entity'den event publish et, service'den handle et
2. Uzun işleri background job'a al
3. `GetOrAddAsync` kullan, expiration ayarla
4. BLOB provider abstraction kullan
5. Data filter'ı `using` ile disable et
6. Settings runtime-değiştirilebilir ayarlar için
7. Features tenant bazlı toggle için

---
name: abp-infrastructure
description: "ABP Framework v10.4 infrastructure: Distributed Event Bus, Background Jobs/Workers, Caching (Redis), BLOB Storing, Emailing, SignalR, IClock, Distributed Locking, Entity Cache. Use when you need an event bus, background job, cache, blob or email in ABP."
---

# ABP Framework — Infrastructure

Guide to ABP Framework v10.4 infrastructure components. Event Bus, Background Jobs, Caching, BLOB Storing, Emailing, Data Filtering, Data Seeding, Settings, Features, Virtual File System, Entity Cache, Distributed Locking, Audit Logging, Current User.

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
- "ABP audit log"
- "ABP current user"
- "ABP infrastructure"

---

## Event Bus

ABP provides two types of event bus:

| Type | Usage | Interface |
|---|---|---|
| **Local Event Bus** | Within the same process | `ILocalEventBus`, `ILocalEventHandler<TEvent>` |
| **Distributed Event Bus** | Across processes (microservice) | `IDistributedEventBus`, `IDistributedEventHandler<TEvent>` |

### Local Event Bus

```csharp
// Event class
public class StockCountChangedEvent
{
    public Guid ProductId { get; set; }
    public int NewCount { get; set; }
}

// Publish (from a service)
public class MyService : ITransientDependency
{
    private readonly ILocalEventBus _localEventBus;
    public MyService(ILocalEventBus localEventBus) => _localEventBus = localEventBus;

    public async Task ChangeStockAsync(Guid productId, int newCount)
    {
        await _localEventBus.PublishAsync(new StockCountChangedEvent
        {
            ProductId = productId,
            NewCount = newCount
        });
    }
}

// Publish (from an Entity/Aggregate Root)
public class Product : AggregateRoot<Guid>
{
    public void ChangeStockCount(int newCount)
    {
        StockCount = newCount;
        AddLocalEvent(new StockCountChangedEvent { ProductId = Id, NewCount = newCount });
    }
}

// Handler
public class StockChangeHandler : ILocalEventHandler<StockCountChangedEvent>, ITransientDependency
{
    [UnitOfWork]
    public virtual async Task HandleEventAsync(StockCountChangedEvent eventData)
    {
        // Event handling logic
    }
}
```

**Features:**
- Event handlers run within the same UOW/transaction
- If a handler throws an exception, the transaction is rolled back
- Control execution order with `LocalEventHandlerOrder`
- Events published from an entity are triggered on SaveChanges (EF Core)

### Distributed Event Bus

```csharp
// ETO (Event Transfer Object) — must be serializable
[EventName("MyApp.Product.StockChange")]
public class StockCountChangedEto
{
    public Guid ProductId { get; set; }
    public int NewCount { get; set; }
}

// Publish
public class MyService : ITransientDependency
{
    private readonly IDistributedEventBus _distributedEventBus;
    public MyService(IDistributedEventBus distributedEventBus) => _distributedEventBus = distributedEventBus;

    public async Task ChangeStockAsync(Guid productId, int newCount)
    {
        await _distributedEventBus.PublishAsync(new StockCountChangedEto
        {
            ProductId = productId,
            NewCount = newCount
        });
    }
}

// Publish from an entity
public class Product : AggregateRoot<Guid>
{
    public void ChangeStockCount(int newCount)
    {
        StockCount = newCount;
        AddDistributedEvent(new StockCountChangedEto { ProductId = Id, NewCount = newCount });
    }
}

// Handler
public class StockChangeHandler : IDistributedEventHandler<StockCountChangedEto>, ITransientDependency
{
    [UnitOfWork]
    public virtual async Task HandleEventAsync(StockCountChangedEto eventData)
    {
        // Event handling logic
    }
}
```

**Distributed Event Bus Providers:**

| Provider | Package | Description |
|---|---|---|
| `LocalDistributedEventBus` | Default | In-process (for monolith) |
| `RabbitMqDistributedEventBus` | Volo.Abp.EventBus.RabbitMQ | RabbitMQ |
| `KafkaDistributedEventBus` | Volo.Abp.EventBus.Kafka | Apache Kafka |
| `AzureDistributedEventBus` | Volo.Abp.EventBus.AzureServiceBus | Azure Service Bus |
| `RebusDistributedEventBus` | Volo.Abp.EventBus.Rebus | Rebus |

**Which Event Bus Should You Use?**

| Scenario | Recommendation |
|---|---|
| Monolith (non-modular) | Local Event Bus |
| Modular Monolith | Distributed Event Bus (inter-module), Local (intra-module) |
| Microservice | Distributed Event Bus (inter-service), Local (intra-service) |

### Inbox/Outbox Pattern (Distributed Event Bus)

Inbox/outbox pattern for data consistency with the distributed event bus:

```csharp
Configure<AbpDistributedEventBusOptions>(options =>
{
    options.InboxDatabaseName = "MyApp";
    options.OutboxDatabaseName = "MyApp";
});
```

---

## Background Jobs

Background jobs are used to queue long-running operations and execute them in the background.

### Defining a Job

```csharp
// Args class
public class EmailSendingArgs
{
    public string EmailAddress { get; set; }
    public string Subject { get; set; }
    public string Body { get; set; }
}

// Job class
public class EmailSendingJob : AsyncBackgroundJob<EmailSendingArgs>, ITransientDependency
{
    private readonly IEmailSender _emailSender;
    public EmailSendingJob(IEmailSender emailSender) => _emailSender = emailSender;

    public override async Task ExecuteAsync(EmailSendingArgs args)
    {
        await _emailSender.SendAsync(args.EmailAddress, args.Subject, args.Body);
    }
}
```

### Enqueuing a Job

```csharp
public class RegistrationService : ApplicationService
{
    private readonly IBackgroundJobManager _backgroundJobManager;
    public RegistrationService(IBackgroundJobManager backgroundJobManager) => _backgroundJobManager = backgroundJobManager;

    public async Task RegisterAsync(string email)
    {
        await _backgroundJobManager.EnqueueAsync(
            new EmailSendingArgs { EmailAddress = email, Subject = "Welcome!", Body = "..." },
            priority: BackgroundJobPriority.Normal,
            delay: TimeSpan.FromMinutes(5)  // Optional delay
        );
    }
}
```

### Job Naming

```csharp
[BackgroundJobName("emails")]
public class EmailSendingArgs { }
```

### Background Job Providers

| Provider | Package | Description |
|---|---|---|
| Default | Volo.Abp.BackgroundJobs | In-memory, persistent (DB) |
| Hangfire | Volo.Abp.BackgroundJobs.Hangfire | Hangfire dashboard, retry |
| Quartz | Volo.Abp.BackgroundJobs.Quartz | Cron scheduling |
| RabbitMQ | Volo.Abp.BackgroundJobs.RabbitMQ | RabbitMQ queue |

### Disabling Job Execution

```csharp
Configure<AbpBackgroundJobOptions>(options =>
{
    options.IsJobExecutionEnabled = false;  // Don't run jobs, only enqueue them
});
```

---

## Caching

### IDistributedCache<TCacheItem>

```csharp
// Cache Item
[CacheName("Books")]
public class BookCacheItem
{
    public string Name { get; set; }
    public float Price { get; set; }
}

// Usage
public class BookService : ITransientDependency
{
    private readonly IDistributedCache<BookCacheItem, Guid> _cache;
    public BookService(IDistributedCache<BookCacheItem, Guid> cache) => _cache = cache;

    public async Task<BookCacheItem> GetAsync(Guid bookId)
    {
        return await _cache.GetOrAddAsync(
            bookId,
            async () => await GetBookFromDatabaseAsync(bookId),
            () => new DistributedCacheEntryOptions
            {
                AbsoluteExpiration = DateTimeOffset.Now.AddHours(1)
            }
        );
    }
}
```

### Configuration

```csharp
Configure<AbpDistributedCacheOptions>(options =>
{
    options.KeyPrefix = "MyApp1";  // Prefix on a shared cache server
    options.GlobalCacheEntryOptions = new DistributedCacheEntryOptions
    {
        SlidingExpiration = TimeSpan.FromMinutes(20)
    };
});
```

### Batch Operations

```csharp
await _cache.GetManyAsync(keys);
await _cache.SetManyAsync(items);
await _cache.GetOrAddManyAsync(keys, async missingKeys => { ... });
await _cache.RemoveManyAsync(keys);
```

### UOW-Level Cache

```csharp
// Not written to the cache until the UOW succeeds
await _cache.SetAsync(key, value, considerUow: true);
```

### Redis Cache

```bash
abp add-package Volo.Abp.Caching.StackExchangeRedis
```

```json
// appsettings.json
"Redis": { 
    "IsEnabled": "true",
    "Configuration": "127.0.0.1"
}
```

Or via code:

```csharp
Configure<RedisCacheOptions>(options => { /* ... */ });
```

**Why use `Volo.Abp.Caching.StackExchangeRedis`?**
1. `SetManyAsync` and `GetManyAsync` implementations (the Microsoft package lacks them)
2. Simplified Redis configuration
3. Built on top of `Microsoft.Extensions.Caching.StackExchangeRedis`

### Entity Cache

```csharp
public class BookEntityCache : EntityCache<Book, BookCacheItem>, ITransientDependency
{
    public BookEntityCache(ICache<Book> cache) : base(cache) { }
}
```

Entity cache is read-only and is automatically invalidated on entity update/delete.

---

## BLOB Storing

An abstraction for file storage. Various providers are supported:

| Provider | Package |
|---|---|
| File System | Volo.Abp.BlobStoring.FileSystem |
| Database | Volo.Abp.BlobStoring.Database |
| AWS S3 | Volo.Abp.BlobStoring.Aws |
| Azure | Volo.Abp.BlobStoring.Azure |
| MinIO | Volo.Abp.BlobStoring.Minio |
| Google Cloud | Volo.Abp.BlobStoring.Google |
| Alibaba Cloud | Volo.Abp.BlobStoring.Aliyun |
| Bunny CDN | Volo.Abp.BlobStoring.Bunny |
| Memory | Volo.Abp.BlobStoring.Memory |

### Usage

```csharp
[BlobContainerName("product-images")]
public class ProductImageBlobContainer : AbpBlobContainer { }

// Configuration
Configure<AbpBlobStoringOptions>(options =>
{
    options.Containers.ConfigureDefault(configuring =>
    {
        configuring.UseFileSystem(fileSystem =>
        {
            fileSystem.BasePath = "C:\\blobs";
        });
    });
});

// Usage
public class ProductImageService : ITransientDependency
{
    private readonly IBlobContainer<ProductImageBlobContainer> _blobContainer;
    
    public ProductImageService(IBlobContainer<ProductImageBlobContainer> blobContainer) =>
        _blobContainer = blobContainer;

    public async Task SaveAsync(Guid productId, byte[] imageBytes)
    {
        await _blobContainer.SaveAsync(productId.ToString(), imageBytes, true);
    }

    public async Task<byte[]> GetAsync(Guid productId)
    {
        return await _blobContainer.GetAllAsync(productId.ToString());
    }
}
```

### Custom Provider

```csharp
public class MyBlobProvider : IBlobProvider, ITransientDependency
{
    public async Task SaveAsync(BlobProviderArgs args) { }
    public async Task<byte[]> GetAsync(BlobProviderArgs args) { }
    public async Task<bool> ExistsAsync(BlobProviderArgs args) { }
    public async Task DeleteAsync(BlobProviderArgs args) { }
}
```

---

## Emailing (MailKit)

```bash
abp add-package Volo.Abp.MailKit
```

```csharp
public class MyService : ITransientDependency
{
    private readonly IEmailSender _emailSender;
    public MyService(IEmailSender emailSender) => _emailSender = emailSender;

    public async Task SendWelcomeEmailAsync(string email)
    {
        await _emailSender.SendAsync(
            to: email,
            subject: "Welcome!",
            body: "Welcome to our platform...",
            isBodyHtml: true
        );
    }
}
```

### Queueing Emails (Background Jobs)

```csharp
await _emailSender.QueueAsync(
    to: email,
    subject: "Welcome!",
    body: "Welcome...",
    isBodyHtml: true
);
```
Emails sent via background job queue — tolerates errors with retry mechanism.

### Configuration

```json
"Settings": {
    "Abp.Mailing.Smtp.Host": "smtp.gmail.com",
    "Abp.Mailing.Smtp.Port": "587",
    "Abp.Mailing.Smtp.UserName": "user@gmail.com",
    "Abp.Mailing.Smtp.Password": "password",
    "Abp.Mailing.Smtp.EnableSsl": "true",
    "Abp.Mailing.DefaultFromAddress": "noreply@myapp.com",
    "Abp.Mailing.DefaultFromDisplayName": "My App"
}
```

### ISmtpEmailSenderConfiguration

Custom configuration source (instead of settings system):

```csharp
public class MySmtpConfiguration : ISmtpEmailSenderConfiguration, ITransientDependency
{
    public string Host => "smtp.mycompany.com";
    public int Port => 587;
    public string UserName => "app@mycompany.com";
    public string Password => "encrypted-password";
    public bool EnableSsl => true;
    public string DefaultFromAddress => "noreply@mycompany.com";
    public string DefaultFromDisplayName => "My App";
}
```

### Text Template Integration

```csharp
public class MyService : ITransientDependency
{
    private readonly IEmailSender _emailSender;
    private readonly ITextTemplateRenderer _textTemplateRenderer;

    public async Task SendTemplateEmailAsync(string email)
    {
        var body = await _textTemplateRenderer.RenderAsync(
            "WelcomeEmailTemplate",
            new Dictionary<string, object>
            {
                { "userName", "John" },
                { "activationLink", "https://myapp.com/activate/123" }
            }
        );

        await _emailSender.SendAsync(email, "Welcome!", body, isBodyHtml: true);
    }
}
```

### Encrypt SMTP Password

```csharp
public class MyEncryptionService : ISettingEncryptionService, ITransientDependency
{
    public string Decrypt(string encryptedValue) { /* ... */ }
    public string Encrypt(string plainValue) { /* ... */ }
}
```

---

## SMS (Twilio)

```bash
abp add-package Volo.Abp.Sms.Twilio
```

```csharp
public class MyService : ITransientDependency
{
    private readonly ITwilioSmsSender _smsSender;
    public MyService(ITwilioSmsSender smsSender) => _smsSender = smsSender;

    public async Task SendSmsAsync(string phone, string message)
    {
        await _smsSender.SendAsync(phone, message);
    }
}
```

---

## Data Filtering

```csharp
public class MyService : ITransientDependency
{
    private readonly IDataFilter _dataFilter;
    private readonly IRepository<Product, Guid> _productRepository;

    public MyService(IDataFilter dataFilter, IRepository<Product, Guid> productRepository)
    {
        _dataFilter = dataFilter;
        _productRepository = productRepository;
    }

    // Disable the soft-delete filter
    public async Task<List<Product>> GetAllIncludingDeletedAsync()
    {
        using (_dataFilter.Disable<ISoftDelete>())
        {
            return await _productRepository.GetListAsync();
        }
    }

    // Disable the multi-tenancy filter
    public async Task<long> GetAllTenantProductCountAsync()
    {
        using (_dataFilter.Disable<IMultiTenant>())
        {
            return await _productRepository.GetCountAsync();
        }
    }
}
```

---

## Data Seeding

```csharp
public class MyDataSeedContributor : IDataSeedContributor, ITransientDependency
{
    private readonly IRepository<IdentityRole, Guid> _roleRepository;

    public MyDataSeedContributor(IRepository<IdentityRole, Guid> roleRepository) =>
        _roleRepository = roleRepository;

    public async Task SeedAsync(DataSeedContext context)
    {
        if (await _roleRepository.FindAsync(x => x.Name == "Admin") == null)
        {
            await _roleRepository.InsertAsync(new IdentityRole(GuidGenerator.Create(), "Admin"));
        }
    }
}
```

---

## Settings

```csharp
// Settings definition
public class MyAppSettings : SettingDefinitionProvider
{
    public override void Define(ISettingDefinitionContext context)
    {
        context.Add(
            new SettingDefinition(
                name: "MyApp.MaxProductPrice",
                defaultValue: "1000",
                displayName: "Maximum Product Price",
                description: "Maximum allowed product price",
                isVisibleToClients: true
            )
        );
    }
}

// Usage
public class MyService : ITransientDependency
{
    private readonly ISettingProvider _settingProvider;
    public MyService(ISettingProvider settingProvider) => _settingProvider = settingProvider;

    public async Task<decimal> GetMaxPriceAsync()
    {
        return await _settingProvider.GetOrNullAsync<decimal>("MyApp.MaxProductPrice");
    }
}

// Changing
await SettingManager.SetAsync("MyApp.MaxProductPrice", "2000");
```

---

## Features

```csharp
// Feature definition
public class MyAppFeatures : FeatureDefinitionProvider
{
    public override void Define(IFeatureDefinitionContext context)
    {
        var myFeature = context.Add(
            new FeatureDefinition(
                name: "MyApp.PremiumFeature",
                defaultValue: "false",
                displayName: "Premium Feature",
                description: "Enables premium features"
            )
        );
    }
}

// Usage
public class MyService : ITransientDependency
{
    private readonly IFeatureChecker _featureChecker;
    public MyService(IFeatureChecker featureChecker) => _featureChecker = featureChecker;

    public async Task<bool> IsPremiumEnabledAsync()
    {
        return await _featureChecker.IsEnabledAsync("MyApp.PremiumFeature");
    }
}
```

---

## Virtual File System

```csharp
// Add embedded files to the virtual file system
Configure<AbpVirtualFileSystemOptions>(options =>
{
    options.FileSets.AddEmbedded<MyModule>();
});

// Usage (in a Razor view)
<link href="/MyModule/styles.css" rel="stylesheet" />

// Or with IVirtualFileProvider
public class MyService : ITransientDependency
{
    private readonly IVirtualFileProvider _virtualFileProvider;
    public MyService(IVirtualFileProvider virtualFileProvider) => _virtualFileProvider = virtualFileProvider;

    public async Task<string> GetFileContentAsync(string path)
    {
        var fileInfo = _virtualFileProvider.GetFileInfo(path);
        using var reader = new StreamReader(fileInfo.CreateReadStream());
        return await reader.ReadToEndAsync();
    }
}
```

---

## Current User

```csharp
public class MyService : ApplicationService
{
    // Pre-injected in ApplicationService/DomainService
    public async Task DoSomethingAsync()
    {
        var userId = CurrentUser.Id;
        var userName = CurrentUser.UserName;
        var tenantId = CurrentUser.TenantId;
        var roles = CurrentUser.Roles;
        var email = CurrentUser.FindClaimValue("email");
        
        var isAuthenticated = CurrentUser.IsAuthenticated;
        var isAdmin = CurrentUser.IsInRole("admin");
    }
}
```

---

## Distributed Locking

```csharp
public class MyService : ITransientDependency
{
    private readonly IDistributedLockProvider _distributedLock;

    public MyService(IDistributedLockProvider distributedLock) => _distributedLock = distributedLock;

    public async Task DoSomethingAsync()
    {
        await using var handle = await _distributedLock.TryAcquireAsync("my-lock-key");
        if (handle != null)
        {
            // Lock acquired, critical operation
        }
    }
}
```

---

## SignalR Integration

```bash
abp add-package Volo.Abp.AspNetCore.SignalR
```

### Server Side

```csharp
[DependsOn(typeof(AbpAspNetCoreSignalRModule))]
public class MyModule : AbpModule { }
```

### Hub Definition

```csharp
public class MessagingHub : Hub
{
    public async Task SendMessage(string message)
    {
        await Clients.All.SendAsync("receiveMessage", message);
    }
}
```

- ABP auto-registers hubs to DI (transient)
- Auto-maps hub endpoint at `/signalr-hubs/messaging` (kebab-case, without `Hub` suffix)
- Custom route: `[HubRoute("/my-messaging-hub")]`

### AbpHub Base Classes

```csharp
public class MyHub : AbpHub
{
    // Pre-injected: ICurrentTenant, ICurrentPrincipalAccessor, etc.
}
```

### Client Side (MVC/Razor Pages)

```bash
abp add-package @abp/signalr
abp install-libs
```

```xml
@using Volo.Abp.AspNetCore.Mvc.UI.Packages.SignalR
@section scripts {
    <abp-script type="typeof(SignalRBrowserScriptContributor)" />
}
```

### SignalR Options

```csharp
Configure<AbpSignalROptions>(options =>
{
    options.Hubs.AddOrUpdate(
        typeof(MessagingHub),
        "/my-messaging/route",
        hubOptions =>
        {
            hubOptions.LongPolling.PollTimeout = TimeSpan.FromSeconds(30);
        }
    );
});
```

---

## Timing & Timezone

```csharp
public class MyService : ITransientDependency
{
    private readonly IClock _clock;
    public MyService(IClock clock) => _clock = clock;

    public void DoSomething()
    {
        var now = _clock.Now;        // UTC (default)
        var localNow = _clock.Now.ToLocalTime();
    }
}

// Timezone configuration
Configure<AbpClockOptions>(options =>
{
    options.Kind = DateTimeKind.Utc;  // or DateTimeKind.Local
});
```

---

## Best Practices

1. **Event Bus:** Publish events from the entity, handle them in the service
2. **Background Jobs:** Move long-running operations to a job, don't keep the user waiting
3. **Caching:** Use `GetOrAddAsync`, set expiration durations
4. **BLOB Storing:** Use the provider abstraction, don't depend directly on the storage API
5. **Data Filtering:** Disable inside a `using` block; it is automatically restored outside the scope
6. **Settings:** Use for runtime-changeable settings
7. **Features:** Use for per-tenant feature toggles
8. **Distributed Lock:** Use for operations requiring concurrent execution control

---

## Related

- [DDD](../abp-ddd/SKILL.md) — domain event publish (AddLocalEvent/AddDistributedEvent)
- [Microservices](../abp-microservices/SKILL.md) — distributed event bus (RabbitMQ), Entity Cache
- [Deployment](../abp-deployment/SKILL.md) — clustered cache/lock, Redis backplane
- [Settings & Features](../abp-settings-features/SKILL.md) — settings/feature infrastructure
- ABP Docs: https://abp.io/docs/latest/framework/infrastructure

# ABP Framework — Infrastructure

ABP Framework v10.4 altyapı bileşenleri rehberi. Event Bus, Background Jobs, Caching, BLOB Storing, Emailing, Data Filtering, Data Seeding, Settings, Features, Virtual File System, Entity Cache, Distributed Locking, Audit Logging, Current User.

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

ABP iki tür event bus sağlar:

| Tip | Kullanım | Interface |
|---|---|---|
| **Local Event Bus** | Aynı process içinde | `ILocalEventBus`, `ILocalEventHandler<TEvent>` |
| **Distributed Event Bus** | Process'ler arası (microservice) | `IDistributedEventBus`, `IDistributedEventHandler<TEvent>` |

### Local Event Bus

```csharp
// Event class
public class StockCountChangedEvent
{
    public Guid ProductId { get; set; }
    public int NewCount { get; set; }
}

// Publish (service içinden)
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

// Publish (Entity/Aggregate Root içinden)
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
        // Event işleme logic'i
    }
}
```

**Özellikler:**
- Event handler'lar aynı UOW/transaction içinde çalışır
- Handler exception atarsa transaction rollback olur
- `LocalEventHandlerOrder` ile execution order kontrolü
- Entity'den publish edilen event'ler SaveChanges'ta tetiklenir (EF Core)

### Distributed Event Bus

```csharp
// ETO (Event Transfer Object) — serialize edilebilir olmalı
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

// Entity'den publish
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
        // Event işleme logic'i
    }
}
```

**Distributed Event Bus Provider'ları:**

| Provider | Package | Açıklama |
|---|---|---|
| `LocalDistributedEventBus` | Varsayılan | In-process (monolith için) |
| `RabbitMqDistributedEventBus` | Volo.Abp.EventBus.RabbitMQ | RabbitMQ |
| `KafkaDistributedEventBus` | Volo.Abp.EventBus.Kafka | Apache Kafka |
| `AzureDistributedEventBus` | Volo.Abp.EventBus.AzureServiceBus | Azure Service Bus |
| `RebusDistributedEventBus` | Volo.Abp.EventBus.Rebus | Rebus |

**Hangi Event Bus Kullanılmalı?**

| Senaryo | Öneri |
|---|---|
| Monolith (non-modular) | Local Event Bus |
| Modular Monolith | Distributed Event Bus (inter-module), Local (intra-module) |
| Microservice | Distributed Event Bus (inter-service), Local (intra-service) |

### Inbox/Outbox Pattern (Distributed Event Bus)

Distributed event bus ile data consistency için inbox/outbox pattern:

```csharp
Configure<AbpDistributedEventBusOptions>(options =>
{
    options.InboxDatabaseName = "MyApp";
    options.OutboxDatabaseName = "MyApp";
});
```

---

## Background Jobs

Background job'lar uzun süren işlemleri kuyruğa alıp arka planda çalıştırmak için kullanılır.

### Job Tanımlama

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

### Job Kuyruğa Ekleme

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
            delay: TimeSpan.FromMinutes(5)  // Opsiyonel gecikme
        );
    }
}
```

### Job İsimlendirme

```csharp
[BackgroundJobName("emails")]
public class EmailSendingArgs { }
```

### Background Job Provider'ları

| Provider | Package | Açıklama |
|---|---|---|
| Default | Volo.Abp.BackgroundJobs | In-memory, persistent (DB) |
| Hangfire | Volo.Abp.BackgroundJobs.Hangfire | Hangfire dashboard, retry |
| Quartz | Volo.Abp.BackgroundJobs.Quartz | Cron scheduling |
| RabbitMQ | Volo.Abp.BackgroundJobs.RabbitMQ | RabbitMQ queue |

### Job Execution Disable

```csharp
Configure<AbpBackgroundJobOptions>(options =>
{
    options.IsJobExecutionEnabled = false;  // Job'ları çalıştırma, sadece enqueue et
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

// Kullanım
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

### Konfigürasyon

```csharp
Configure<AbpDistributedCacheOptions>(options =>
{
    options.KeyPrefix = "MyApp1";  // Paylaşılan cache server'da prefix
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
// UOW başarılı olana kadar cache'e yazılmaz
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

Veya kod ile:

```csharp
Configure<RedisCacheOptions>(options => { /* ... */ });
```

**Neden `Volo.Abp.Caching.StackExchangeRedis` kullanmalı?**
1. `SetManyAsync` ve `GetManyAsync` implementasyonu (Microsoft paketi yok)
2. Redis konfigürasyonu basitleştirilmiş
3. `Microsoft.Extensions.Caching.StackExchangeRedis` üzerine inşa edilmiş

### Entity Cache

```csharp
public class BookEntityCache : EntityCache<Book, BookCacheItem>, ITransientDependency
{
    public BookEntityCache(ICache<Book> cache) : base(cache) { }
}
```

Entity cache read-only'dir ve entity update/delete'de otomatik invalidate olur.

---

## BLOB Storing

Dosya depolama için soyutlama. Çeşitli provider'lar desteklenir:

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

### Kullanım

```csharp
[BlobContainerName("product-images")]
public class ProductImageBlobContainer : AbpBlobContainer { }

// Konfigürasyon
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

// Kullanım
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

### Konfigürasyon

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

    // Soft-delete filter'ı devre dışı bırak
    public async Task<List<Product>> GetAllIncludingDeletedAsync()
    {
        using (_dataFilter.Disable<ISoftDelete>())
        {
            return await _productRepository.GetListAsync();
        }
    }

    // Multi-tenancy filter'ı devre dışı bırak
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

// Kullanım
public class MyService : ITransientDependency
{
    private readonly ISettingProvider _settingProvider;
    public MyService(ISettingProvider settingProvider) => _settingProvider = settingProvider;

    public async Task<decimal> GetMaxPriceAsync()
    {
        return await _settingProvider.GetOrNullAsync<decimal>("MyApp.MaxProductPrice");
    }
}

// Değiştirme
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

// Kullanım
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
// Embedded dosyaları virtual file system'e ekle
Configure<AbpVirtualFileSystemOptions>(options =>
{
    options.FileSets.AddEmbedded<MyModule>();
});

// Kullanım (Razor view'da)
<link href="/MyModule/styles.css" rel="stylesheet" />

// Veya IVirtualFileProvider ile
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
    // ApplicationService/DomainService'de pre-injected
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
            // Lock alındı, kritik işlem
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
        var now = _clock.Now;        // UTC (varsayılan)
        var localNow = _clock.Now.ToLocalTime();
    }
}

// Timezone konfigürasyonu
Configure<AbpClockOptions>(options =>
{
    options.Kind = DateTimeKind.Utc;  // veya DateTimeKind.Local
});
```

---

## Best Practices

1. **Event Bus:** Entity'den event publish et, service'den handle et
2. **Background Jobs:** Uzun süren işlemleri job'a al, kullanıcıyı bekletme
3. **Caching:** `GetOrAddAsync` kullan, expiration sürelerini ayarla
4. **BLOB Storing:** Provider abstraction kullan, direkt storage API'sine bağımlı olma
5. **Data Filtering:** `using` bloğu içinde disable et, scope dışında otomatik geri yüklenir
6. **Settings:** Runtime'da değiştirilebilir ayarlar için kullan
7. **Features:** Tenant bazlı feature toggle için kullan
8. **Distributed Lock:** Concurrent execution gerektiren işlemlerde kullan

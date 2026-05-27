# ABP Framework — Domain Driven Design (DDD)

ABP Framework v10.4'te DDD uygulama rehberi. Entity, Aggregate Root, Repository, Domain Service, Application Service ve DTO tasarım pattern'leri.

## Trigger

- "ABP entity oluştur"
- "ABP aggregate root"
- "ABP repository"
- "ABP application service"
- "ABP DTO"
- "ABP domain service"
- "ABP unit of work"
- "ABP CRUD app service"
- "ABP DDD"

## DDD Katmanları

ABP dört katmanlı mimari uygular:

```
┌─────────────────────────────────────────┐
│         Presentation Layer              │  ← MVC, Blazor, Angular, React
├─────────────────────────────────────────┤
│         Application Layer               │  ← Application Services, DTOs, UOW
├─────────────────────────────────────────┤
│         Domain Layer                    │  ← Entities, Aggregates, Domain Services, Repositories (interface)
├─────────────────────────────────────────┤
│         Infrastructure Layer            │  ← EF Core, MongoDB, 3rd-party libraries
└─────────────────────────────────────────┘
```

DDD esas olarak **Domain** ve **Application** katmanlarını ilgilendirir.

---

## Domain Layer

### Entity Tasarımı

```csharp
public class Book : Entity<Guid>
{
    public string Name { get; set; }
    public float Price { get; set; }

    protected Book() { }  // ORM deserialization için

    public Book(Guid id) : base(id) { }
}
```

**GUID Key Best Practices:**
- Constructor'da `id` parametresi al ve base'e geçir
- `Guid.NewGuid()` **kullanma** — `IGuidGenerator.Create()` kullan (sequential GUID)
- Protected/private empty constructor ekle (ORM deserialization)

### Aggregate Root

Aggregate Root, bir aggregate'in tek giriş noktasıdır ve tutarlılıktan sorumludur.

```csharp
public class Order : AggregateRoot<Guid>
{
    public virtual string ReferenceNo { get; protected set; }
    public virtual int TotalItemCount { get; protected set; }
    public virtual DateTime CreationTime { get; protected set; }
    public virtual List<OrderLine> OrderLines { get; protected set; }

    protected Order() { }

    public Order(Guid id, string referenceNo)
    {
        Check.NotNull(referenceNo, nameof(referenceNo));
        Id = id;
        ReferenceNo = referenceNo;
        OrderLines = new List<OrderLine>();
    }

    public void AddProduct(Guid productId, int count)
    {
        if (count <= 0)
            throw new ArgumentException("Count must be positive", nameof(count));

        var existingLine = OrderLines.FirstOrDefault(ol => ol.ProductId == productId);
        if (existingLine == null)
            OrderLines.Add(new OrderLine(Id, productId, count));
        else
            existingLine.ChangeCount(existingLine.Count + count);

        TotalItemCount += count;
    }
}

public class OrderLine : Entity
{
    public virtual Guid OrderId { get; protected set; }
    public virtual Guid ProductId { get; protected set; }
    public virtual int Count { get; protected set; }

    protected OrderLine() { }

    internal OrderLine(Guid orderId, Guid productId, int count)
    {
        OrderId = orderId;
        ProductId = productId;
        Count = count;
    }

    internal void ChangeCount(int newCount) => Count = newCount;

    public override object[] GetKeys() => new object[] { OrderId, ProductId };
}
```

**Aggregate Root Kuralları:**
- Aggregate root kendi ve alt entity'lerinin geçerliliğinden sorumludur
- `Id` ile referans ver, navigation property ile değil
- Tek bir birim olarak retrieve ve update edilir (transaction boundary)
- Alt entity'ler aggregate root üzerinden değiştirilir

### Audit Properties

```csharp
// En yaygın kullanım
public class Product : FullAuditedAggregateRoot<Guid>
{
    // Otomatik: CreationTime, CreatorId, LastModificationTime, 
    //           LastModifierId, IsDeleted, DeletionTime, DeleterId
    public string Name { get; set; }
}
```

| Base Class | Sağladığı Özellikler |
|---|---|
| `CreationAuditedAggregateRoot<TKey>` | CreationTime, CreatorId |
| `AuditedAggregateRoot<TKey>` | + LastModificationTime, LastModifierId |
| `FullAuditedAggregateRoot<TKey>` | + IsDeleted, DeletionTime, DeleterId (soft-delete) |

### Value Objects

```csharp
public class Money : ValueObject
{
    public decimal Amount { get; private set; }
    public string Currency { get; private set; }

    private Money() { }

    public Money(decimal amount, string currency)
    {
        Amount = amount;
        Currency = currency;
    }

    protected override IEnumerable<object> GetAtomicValues()
    {
        yield return Amount;
        yield return Currency;
    }
}
```

### Domain Services

```csharp
public class OrderManager : DomainService
{
    private readonly IRepository<Order, Guid> _orderRepository;

    public OrderManager(IRepository<Order, Guid> orderRepository)
    {
        _orderRepository = orderRepository;
    }

    public async Task<Order> CreateOrderAsync(Guid customerId, List<OrderItem> items)
    {
        var order = new Order(GuidGenerator.Create(), $"ORD-{DateTime.UtcNow:yyyyMMdd}");
        // domain logic
        await _orderRepository.InsertAsync(order);
        return order;
    }
}
```

Domain service kullan:
- İş kuralı tek bir entity'ye sığmıyorsa
- Birden fazla aggregate ile çalışılması gerekiyorsa

---

## Application Layer

### DTO Tasarımı

```csharp
public class BookDto : AuditedEntityDto<Guid>
{
    public string Name { get; set; }
    public BookType Type { get; set; }
    public float Price { get; set; }
}

public class CreateUpdateBookDto
{
    [Required]
    [StringLength(128)]
    public string Name { get; set; }

    [Required]
    public BookType Type { get; set; } = BookType.Undefined;

    [Required]
    public float Price { get; set; }
}
```

**Kural:** Entity'leri presentation layer'a asla expose etme. Her zaman DTO kullan.

### Object Mapping (Mapperly)

ABP 10.4'te varsayılan mapping provider Mapperly'dir.

```csharp
[Mapper]
public partial class BookToBookDtoMapper : MapperBase<Book, BookDto>
{
    public override partial BookDto Map(Book source);
    public override partial void Map(Book source, BookDto destination);
}

[Mapper]
public partial class CreateUpdateBookDtoToBookMapper : MapperBase<CreateUpdateBookDto, Book>
{
    public override partial Book Map(CreateUpdateBookDto source);
    public override partial void Map(CreateUpdateBookDto source, Book destination);
}
```

Module konfigürasyonu:

```csharp
[DependsOn(typeof(AbpMapperlyModule))]
public class MyModule : AbpModule
{
    public override void ConfigureServices(ServiceConfigurationContext context)
    {
        context.Services.AddMapperlyObjectMapper<MyModule>();
    }
}
```

Kullanım:

```csharp
var bookDto = ObjectMapper.Map<Book, BookDto>(book);
var book = ObjectMapper.Map<CreateUpdateBookDto, Book>(input);
```

### Application Service

```csharp
public interface IBookAppService : IApplicationService
{
    Task<BookDto> GetAsync(Guid id);
    Task<PagedResultDto<BookDto>> GetListAsync(PagedAndSortedResultRequestDto input);
    Task<BookDto> CreateAsync(CreateUpdateBookDto input);
    Task<BookDto> UpdateAsync(Guid id, CreateUpdateBookDto input);
    Task DeleteAsync(Guid id);
}

public class BookAppService : ApplicationService, IBookAppService
{
    private readonly IRepository<Book, Guid> _bookRepository;

    public BookAppService(IRepository<Book, Guid> bookRepository)
    {
        _bookRepository = bookRepository;
    }

    public async Task<BookDto> GetAsync(Guid id)
    {
        var book = await _bookRepository.GetAsync(id);
        return ObjectMapper.Map<Book, BookDto>(book);
    }

    public async Task<BookDto> CreateAsync(CreateUpdateBookDto input)
    {
        var book = new Book(GuidGenerator.Create(), input.Name, input.Type, input.Price);
        await _bookRepository.InsertAsync(book);
        return ObjectMapper.Map<Book, BookDto>(book);
    }
}
```

### CRUD Application Service (Base Class)

```csharp
public interface IBookAppService : ICrudAppService<
    BookDto,                         // TEntityDto
    Guid,                            // TKey
    PagedAndSortedResultRequestDto,  // TGetListInput
    CreateUpdateBookDto,             // TCreateInput
    CreateUpdateBookDto>             // TUpdateInput
{
}

public class BookAppService : CrudAppService<
    Book, BookDto, Guid,
    PagedAndSortedResultRequestDto,
    CreateUpdateBookDto, CreateUpdateBookDto>, IBookAppService
{
    public BookAppService(IRepository<Book, Guid> repository) : base(repository) { }
}
```

`ICrudAppService` otomatik sağlar: `GetAsync`, `GetListAsync`, `CreateAsync`, `UpdateAsync`, `DeleteAsync`

### AbstractKeyCrudAppService (Composite Key)

```csharp
public class DistrictKey
{
    public Guid CityId { get; set; }
    public string Name { get; set; }
}

public class DistrictAppService : AbstractKeyCrudAppService<District, DistrictDto, DistrictKey>
{
    public DistrictAppService(IRepository<District> repository) : base(repository) { }

    protected override async Task DeleteByIdAsync(DistrictKey id)
    {
        await Repository.DeleteAsync(d => d.CityId == id.CityId && d.Name == id.Name);
    }

    protected override async Task<District> GetEntityByIdAsync(DistrictKey id)
    {
        var queryable = await Repository.GetQueryableAsync();
        return await AsyncQueryableExecuter.FirstOrDefaultAsync(
            queryable.Where(d => d.CityId == id.CityId && d.Name == id.Name)
        );
    }
}
```

### Authorization (CRUD)

```csharp
public class BookAppService : CrudAppService<Book, BookDto, Guid, PagedAndSortedResultRequestDto, CreateUpdateBookDto, CreateUpdateBookDto>
{
    public BookAppService(IRepository<Book, Guid> repository) : base(repository)
    {
        GetPolicyName = "BookStore.Books";
        GetListPolicyName = "BookStore.Books";
        CreatePolicyName = "BookStore.Books.Create";
        UpdatePolicyName = "BookStore.Books.Update";
        DeletePolicyName = "BookStore.Books.Delete";
    }
}
```

---

## Repositories

### Generic Repository Kullanımı

```csharp
public class PersonAppService : ApplicationService
{
    private readonly IRepository<Person, Guid> _personRepository;

    public PersonAppService(IRepository<Person, Guid> personRepository)
    {
        _personRepository = personRepository;
    }

    // Standart metodlar:
    // GetAsync, FindAsync, InsertAsync, UpdateAsync, DeleteAsync
    // GetListAsync, GetPagedListAsync, GetCountAsync
    // InsertManyAsync, UpdateManyAsync, DeleteManyAsync

    public async Task<List<Person>> SearchAsync(string filter)
    {
        var queryable = await _personRepository.GetQueryableAsync();
        return await AsyncExecuter.ToListAsync(
            queryable.Where(p => p.Name.Contains(filter))
        );
    }
}
```

### Custom Repository

```csharp
// Domain layer — interface
public interface IPersonRepository : IRepository<Person, Guid>
{
    Task<Person> FindByNameAsync(string name);
}

// EF Core layer — implementation
public class PersonRepository : EfCoreRepository<MyDbContext, Person, Guid>, IPersonRepository
{
    public PersonRepository(IDbContextProvider<MyDbContext> dbContextProvider)
        : base(dbContextProvider) { }

    public async Task<Person> FindByNameAsync(string name)
    {
        var dbContext = await GetDbContextAsync();
        return await dbContext.Set<Person>()
            .FirstOrDefaultAsync(p => p.Name == name);
    }
}
```

### WithDetails (Eager Loading)

```csharp
var queryable = await _orderRepository.WithDetailsAsync(x => x.OrderLines);
var orders = await AsyncExecuter.ToListAsync(queryable);
```

### Change Tracking Kontrolü

```csharp
// Disable tracking (read-only sorgular için performans artışı)
using (_personRepository.DisableTracking())
{
    var list = await _personRepository.GetPagedListAsync(0, 100, "Name ASC");
}

// Attribute ile
[DisableEntityChangeTracking]
public virtual async Task<List<PersonDto>> GetListAsync() { ... }
```

---

## Unit of Work

### Conventions (Otomatik)

ABP otomatik UOW başlatır:
- ASP.NET Core MVC Controller Actions
- Razor Page Handlers
- Application Service metodları
- Repository metodları

**HTTP GET** istekleri transactional UOW başlatmaz (sadece read-only).

### Manuel UOW

```csharp
public class MyService : ITransientDependency
{
    private readonly IUnitOfWorkManager _unitOfWorkManager;

    public MyService(IUnitOfWorkManager unitOfWorkManager)
    {
        _unitOfWorkManager = unitOfWorkManager;
    }

    public async Task FooAsync()
    {
        using (var uow = _unitOfWorkManager.Begin(requiresNew: true, isTransactional: false))
        {
            // operations
            await uow.CompleteAsync();
        }
    }
}
```

### UnitOfWork Attribute

```csharp
[UnitOfWork]
public virtual async Task FooAsync() { }

[UnitOfWork(IsDisabled = true)]
public virtual async Task BarAsync() { }

[UnitOfWork(IsTransactional = false)]
public virtual async Task BazAsync() { }
```

### SaveChanges

```csharp
// autoSave parametresi (tercih edilen)
await _repository.InsertAsync(entity, autoSave: true);

// Manuel
await UnitOfWorkManager.Current.SaveChangesAsync();
// veya
await CurrentUnitOfWork.SaveChangesAsync();
```

---

## Validation

Application service input'ları otomatik validate edilir:

```csharp
public class CreateBookDto
{
    [Required]
    [StringLength(128)]
    public string Name { get; set; }
}
```

Fluent Validation desteği de mevcuttur:

```csharp
public class CreateBookDtoValidator : AbstractValidator<CreateBookDto>
{
    public CreateBookDtoValidator()
    {
        RuleFor(x => x.Name).NotEmpty().MaximumLength(128);
        RuleFor(x => x.Price).GreaterThan(0);
    }
}
```

---

## Specifications

```csharp
public class ProductsByCategorySpec : Specification<Product>
{
    private readonly Guid _categoryId;
    public ProductsByCategorySpec(Guid categoryId) => _categoryId = categoryId;

    public override Expression<Func<Product, bool>> ToExpression()
    {
        return p => p.CategoryId == _categoryId && !p.IsDeleted;
    }
}

// Kullanım
var spec = new ProductsByCategorySpec(categoryId);
var products = await _productRepository.GetListAsync(spec);
var count = await _productRepository.CountAsync(spec);
```

---

## Extra Properties

Entity'lere dinamik property ekleme (özellikle modül genişletme için):

```csharp
// Set
user.SetProperty("Title", "Dr.");

// Get
var title = user.GetProperty<string>("Title");

// Extension method ile
public static class IdentityUserExtensions
{
    private const string TitlePropertyName = "Title";
    public static void SetTitle(this IdentityUser user, string title) =>
        user.SetProperty(TitlePropertyName, title);
    public static string GetTitle(this IdentityUser user) =>
        user.GetProperty<string>(TitlePropertyName);
}
```

---

## Working with Streams (IRemoteStreamContent)

```csharp
// Interface
public interface ITestAppService : IApplicationService
{
    Task Upload(Guid id, IRemoteStreamContent streamContent);
    Task<IRemoteStreamContent> Download(Guid id);
}

// Implementation
public class TestAppService : ApplicationService, ITestAppService
{
    public async Task Upload(Guid id, IRemoteStreamContent streamContent)
    {
        using var fs = new FileStream($"C:\\Temp\\{id}.blob", FileMode.Create);
        await streamContent.GetStream().CopyToAsync(fs);
    }

    public Task<IRemoteStreamContent> Download(Guid id)
    {
        var fs = new FileStream($"C:\\Temp\\{id}.blob", FileMode.Open);
        return Task.FromResult<IRemoteStreamContent>(
            new RemoteStreamContent(fs) { ContentType = "application/octet-stream" }
        );
    }
}
```

DTO'da stream kullanırken `AbpAspNetCoreMvcOptions` konfigürasyonu:

```csharp
Configure<AbpAspNetCoreMvcOptions>(options =>
{
    options.ConventionalControllers.FormBodyBindingIgnoredTypes.Add(typeof(CreateFileInput));
});
```

---

## IUnitOfWorkEnabled Interface

Convention dışındaki class'larda UOW enable etmek için:

```csharp
public class MyService : ITransientDependency, IUnitOfWorkEnabled
{
    // Tüm metodlar UOW scope'unda çalışır
    public virtual async Task FooAsync() { }  // virtual olmalı (interface ile inject edilmiyorsa)
}
```

**Kurallar:**
- Interface ile inject edilmiyorsa metodlar `virtual` olmalı
- Sadece `async` metodlar (Task/Task<T> dönen) intercept edilir
- Sync metodlar UOW başlatamaz

---

## Best Practices

1. **Aggregate Root tasarla** — Entity'leri doğrudanAggregateRoot yerine AggregateRoot kullan
2. **Protected setter kullan** — Entity state'ini koru
3. **Constructor validation** — Entity constructor'ında validasyon yap
4. **DTO kullan** — Entity'leri asla presentation'a expose etme
5. **Mapperly mapper'ları partial class olarak tanımla** — `[Mapper]` attribute ile
6. **IAsyncQueryableExecuter kullan** — Domain layer'ı EF Core'dan izole tut
7. **Sequential GUID kullan** — `IGuidGenerator.Create()` ile
8. **Soft-delete için FullAuditedAggregateRoot kullan** — `ISoftDelete` implementasyonu otomatik
9. **CrudAppService base class kullan** — CRUD işlemleri için boilerplate azaltır
10. **UOW convention'larına güven** — Manuel UOW sadece özel durumlarda gerekli

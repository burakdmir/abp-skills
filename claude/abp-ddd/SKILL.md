---
name: abp-ddd
description: "ABP Framework v10.4 Domain Driven Design: Entity, AggregateRoot, repository, domain service, application service, DTO, domain events (AddLocalEvent/AddDistributedEvent), specification, value object, UOW. Use when designing the domain layer, entities, aggregates, or repositories in ABP."
---

# ABP Framework — Domain Driven Design (DDD)

A guide to applying DDD in ABP Framework v10.4. Entity, Aggregate Root, Repository, Domain Service, Application Service, and DTO design patterns.

## Trigger

- "create an ABP entity"
- "ABP aggregate root"
- "ABP repository"
- "ABP application service"
- "ABP DTO"
- "ABP domain service"
- "ABP unit of work"
- "ABP CRUD app service"
- "ABP DDD"

## DDD Layers

ABP applies a four-layer architecture:

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

DDD primarily concerns the **Domain** and **Application** layers.

---

## Domain Layer

### Entity Design

```csharp
public class Book : Entity<Guid>
{
    public string Name { get; set; }
    public float Price { get; set; }

    protected Book() { }  // for ORM deserialization

    public Book(Guid id) : base(id) { }
}
```

**GUID Key Best Practices:**
- Take an `id` parameter in the constructor and pass it to base
- **Do not use** `Guid.NewGuid()` — use `IGuidGenerator.Create()` (sequential GUID)
- Add a protected/private empty constructor (ORM deserialization)

### Aggregate Root

The Aggregate Root is the single entry point of an aggregate and is responsible for consistency.

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

**Aggregate Root Rules:**
- The aggregate root is responsible for the validity of itself and its child entities
- Reference by `Id`, not via a navigation property
- It is retrieved and updated as a single unit (transaction boundary)
- Child entities are modified through the aggregate root

### Audit Properties

```csharp
// Most common usage
public class Product : FullAuditedAggregateRoot<Guid>
{
    // Automatic: CreationTime, CreatorId, LastModificationTime, 
    //            LastModifierId, IsDeleted, DeletionTime, DeleterId
    public string Name { get; set; }
}
```

| Base Class | Properties Provided |
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

Use a domain service when:
- The business rule doesn't fit into a single entity
- Multiple aggregates need to be worked with

**Domain Service Best Practices (ai-rules):**
- Use the `*Manager` suffix (e.g. `OrderManager`)
- Don't define an interface by default (add one if needed)
- Take/return domain objects, not DTOs
- Don't depend on the authenticated user — take values as parameters from the application layer
- Don't inject `IGuidGenerator`/`IClock` — use the base class properties (`GuidGenerator`, `Clock`)

### Domain Events

Aggregate roots publish domain events to trigger side effects. There are two kinds:

```csharp
public class Order : AggregateRoot<Guid>
{
    public OrderStatus Status { get; private set; }

    public void Complete()
    {
        if (Status != OrderStatus.Created)
            throw new BusinessException("Orders:CannotCompleteOrder");

        Status = OrderStatus.Completed;

        // Processed synchronously within the same transaction — can access the entire entity
        AddLocalEvent(new OrderCompletedEvent(Id));

        // Asynchronous, across modules/microservices — use an ETO (Event Transfer Object)
        AddDistributedEvent(new OrderCompletedEto { OrderId = Id });
    }
}
```

**Local Event** — synchronous, within the same UOW/transaction:

```csharp
public class OrderCompletedEventHandler
    : ILocalEventHandler<OrderCompletedEvent>, ITransientDependency
{
    public async Task HandleEventAsync(OrderCompletedEvent eventData)
    {
        // Runs within the same transaction
    }
}
```

**Distributed Event (ETO)** — asynchronous, loosely coupled. Define the ETO in `*.Domain.Shared`:

```csharp
[EventName("Orders.OrderCompleted")]
public class OrderCompletedEto
{
    public Guid OrderId { get; set; }
    public string OrderNumber { get; set; }
}
```

> Events are added inside the aggregate root with `AddLocalEvent`/`AddDistributedEvent`; ABP publishes them automatically when the entity is saved. Handlers are registered automatically via `ITransientDependency`. For distributed event publish/subscribe details see the [Infrastructure skill](../abp-infrastructure/SKILL.md).

---

## Application Layer

### DTO Design

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

**Rule:** Never expose entities to the presentation layer. Always use DTOs.

### Object Mapping (Mapperly)

In ABP 10.4 the default mapping provider is Mapperly.

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

Module configuration:

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

Usage:

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

`ICrudAppService` automatically provides: `GetAsync`, `GetListAsync`, `CreateAsync`, `UpdateAsync`, `DeleteAsync`

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

### Using the Generic Repository

```csharp
public class PersonAppService : ApplicationService
{
    private readonly IRepository<Person, Guid> _personRepository;

    public PersonAppService(IRepository<Person, Guid> personRepository)
    {
        _personRepository = personRepository;
    }

    // Standard methods:
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

### Change Tracking Control

```csharp
// Disable tracking (performance boost for read-only queries)
using (_personRepository.DisableTracking())
{
    var list = await _personRepository.GetPagedListAsync(0, 100, "Name ASC");
}

// With an attribute
[DisableEntityChangeTracking]
public virtual async Task<List<PersonDto>> GetListAsync() { ... }
```

---

## Unit of Work

### Conventions (Automatic)

ABP starts a UOW automatically for:
- ASP.NET Core MVC Controller Actions
- Razor Page Handlers
- Application Service methods
- Repository methods

**HTTP GET** requests do not start a transactional UOW (read-only only).

### Manual UOW

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
// autoSave parameter (preferred)
await _repository.InsertAsync(entity, autoSave: true);

// Manual
await UnitOfWorkManager.Current.SaveChangesAsync();
// or
await CurrentUnitOfWork.SaveChangesAsync();
```

---

## Validation

Application service inputs are validated automatically:

```csharp
public class CreateBookDto
{
    [Required]
    [StringLength(128)]
    public string Name { get; set; }
}
```

Fluent Validation support is also available:

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

// Usage
var spec = new ProductsByCategorySpec(categoryId);
var products = await _productRepository.GetListAsync(spec);
var count = await _productRepository.CountAsync(spec);
```

---

## Extra Properties

Adding dynamic properties to entities (especially for module extension):

```csharp
// Set
user.SetProperty("Title", "Dr.");

// Get
var title = user.GetProperty<string>("Title");

// With an extension method
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

When using a stream in a DTO, the `AbpAspNetCoreMvcOptions` configuration:

```csharp
Configure<AbpAspNetCoreMvcOptions>(options =>
{
    options.ConventionalControllers.FormBodyBindingIgnoredTypes.Add(typeof(CreateFileInput));
});
```

---

## IUnitOfWorkEnabled Interface

To enable UOW in classes outside the convention:

```csharp
public class MyService : ITransientDependency, IUnitOfWorkEnabled
{
    // All methods run within a UOW scope
    public virtual async Task FooAsync() { }  // must be virtual (if not injected via an interface)
}
```

**Rules:**
- Methods must be `virtual` if not injected via an interface
- Only `async` methods (returning Task/Task<T>) are intercepted
- Sync methods cannot start a UOW

---

## Best Practices

1. **Design Aggregate Roots** — use AggregateRoot instead of plain Entity directly
2. **Use protected setters** — protect entity state
3. **Constructor validation** — validate in the entity constructor
4. **Use DTOs** — never expose entities to the presentation layer
5. **Define Mapperly mappers as partial classes** — with the `[Mapper]` attribute
6. **Use IAsyncQueryableExecuter** — keep the domain layer isolated from EF Core
7. **Use sequential GUIDs** — with `IGuidGenerator.Create()`
8. **Use FullAuditedAggregateRoot for soft-delete** — `ISoftDelete` implementation is automatic
9. **Use the CrudAppService base class** — reduces boilerplate for CRUD operations
10. **Rely on UOW conventions** — manual UOW is only needed in special cases

---

## Related

- [EF Core](../abp-efcore/SKILL.md) — repository implementation, ConfigureByConvention, migration
- [MongoDB](../abp-mongodb/SKILL.md) — MongoDB repository implementation
- [Object Mapping](../abp-object-mapping/SKILL.md) — entity↔DTO conversion (Mapperly/AutoMapper)
- [Dependency Rules](../abp-dependency-rules/SKILL.md) — layer dependency rules
- [Development Flow](../abp-development-flow/SKILL.md) — end-to-end flow for adding a new entity
- [Validation](../abp-validation/SKILL.md) — DTO validation
- [Infrastructure](../abp-infrastructure/SKILL.md) — domain event publish/subscribe
- ABP Docs: https://abp.io/docs/latest/framework/architecture/domain-driven-design

---
name: abp-ddd
description: "ABP Framework v10.4 DDD quick reference: Entity, AggregateRoot, repository, domain service, application service, DTO, domain events, specification, UOW. Use when designing the domain layer, entities, aggregates, or repositories in ABP."
---

# ABP Framework — Domain Driven Design (DDD)

ABP Framework v10.4 DDD quick reference. Entity, Aggregate Root, Repository, Domain/Application Service, DTO.

## Trigger

"ABP entity/aggregate root", "ABP repository", "ABP application service", "ABP DTO", "ABP domain service", "ABP unit of work", "ABP DDD".

## Layers

`Presentation → Application (Services, DTO, UOW) → Domain (Entity, Aggregate, Domain Service, Repo interface) → Infrastructure (EF Core/MongoDB)`. DDD primarily concerns Domain + Application.

## Entity & Aggregate Root

```csharp
public class Order : AggregateRoot<Guid>
{
    public string ReferenceNo { get; private set; }
    public ICollection<OrderLine> Lines { get; private set; }

    protected Order() { }                                  // for ORM
    public Order(Guid id, string referenceNo) : base(id)   // id from outside (IGuidGenerator)
    {
        ReferenceNo = Check.NotNullOrWhiteSpace(referenceNo, nameof(referenceNo));
        Lines = new List<OrderLine>();
    }

    public void AddProduct(Guid productId, int count)      // behavior in the entity, private setter
    {
        if (count <= 0) throw new BusinessException("Orders:InvalidCount");
        Lines.Add(new OrderLine(Id, productId, count));
    }
}
```

**Rules:** rich model (private setter + method), `GuidGenerator.Create()` not `Guid.NewGuid()`, reference by `Id` (no cross-aggregate navigation), **one** repository per aggregate root (don't open a repo for a child entity).

### Audit base classes

| Base | Provides |
|---|---|
| `CreationAuditedAggregateRoot<TKey>` | CreationTime, CreatorId |
| `AuditedAggregateRoot<TKey>` | + LastModification* |
| `FullAuditedAggregateRoot<TKey>` | + IsDeleted/Deletion* (soft-delete) |

### Value Object

```csharp
public class Money : ValueObject
{
    public decimal Amount { get; private set; }
    public string Currency { get; private set; }
    protected override IEnumerable<object> GetAtomicValues() { yield return Amount; yield return Currency; }
}
```

## Domain Service

```csharp
public class OrderManager : DomainService   // *Manager suffix
{
    private readonly IOrderRepository _orderRepository;
    public OrderManager(IOrderRepository orderRepository) => _orderRepository = orderRepository;

    public async Task<Order> CreateAsync(string referenceNo)
    {
        if (await _orderRepository.FindByReferenceAsync(referenceNo) != null)
            throw new BusinessException("Orders:ReferenceAlreadyExists");
        return new Order(GuidGenerator.Create(), referenceNo);   // base class GuidGenerator
    }
}
```
Use: when a rule doesn't fit a single entity / when multiple aggregates are needed. Take/return domain objects, not DTOs; don't depend on the authenticated user.

## Domain Events

```csharp
public void Complete()
{
    Status = OrderStatus.Completed;
    AddLocalEvent(new OrderCompletedEvent(Id));            // same transaction, synchronous
    AddDistributedEvent(new OrderCompletedEto { OrderId = Id }); // asynchronous, ETO
}

public class OrderCompletedHandler : ILocalEventHandler<OrderCompletedEvent>, ITransientDependency
{
    public async Task HandleEventAsync(OrderCompletedEvent e) { }
}

// ETO → *.Domain.Shared
[EventName("Orders.OrderCompleted")]
public class OrderCompletedEto { public Guid OrderId { get; set; } }
```

## Application Layer

```csharp
public class BookDto : AuditedEntityDto<Guid> { public string Name { get; set; } public float Price { get; set; } }

public class CreateUpdateBookDto
{
    [Required, StringLength(128)] public string Name { get; set; }
    [Required] public float Price { get; set; }
}
```
**Rule:** never expose entities, always DTOs.

### Object Mapping (Mapperly — default in v10.4)

```csharp
[Mapper]
public partial class BookMapper : MapperBase<Book, BookDto>
{
    public override partial BookDto Map(Book source);
    public override partial void Map(Book source, BookDto destination);
}
// Module: [DependsOn(typeof(AbpMapperlyModule))] + context.Services.AddMapperlyObjectMapper<MyModule>();
var dto = ObjectMapper.Map<Book, BookDto>(book);
```
> For Mapperly/AutoMapper details see the [Object Mapping skill](../abp-object-mapping/SKILL.md).

### Application Service

```csharp
public class BookAppService : ApplicationService, IBookAppService
{
    private readonly IRepository<Book, Guid> _repo;
    public BookAppService(IRepository<Book, Guid> repo) => _repo = repo;

    [Authorize(BookStorePermissions.Books.Create)]
    public async Task<BookDto> CreateAsync(CreateUpdateBookDto input)
    {
        var book = new Book(GuidGenerator.Create(), input.Name, input.Price);
        await _repo.InsertAsync(book);
        return ObjectMapper.Map<Book, BookDto>(book);
    }
}
```

### CrudAppService (reduces boilerplate)

```csharp
public class BookAppService
    : CrudAppService<Book, BookDto, Guid, PagedAndSortedResultRequestDto, CreateUpdateBookDto, CreateUpdateBookDto>, IBookAppService
{
    public BookAppService(IRepository<Book, Guid> repository) : base(repository)
    {
        CreatePolicyName = "BookStore.Books.Create";   // GetPolicyName/UpdatePolicyName/DeletePolicyName
    }
}
```

## Repository

```csharp
// Generic (simple CRUD): IRepository<Book, Guid>  → Get/Find/Insert/Update/Delete/GetListAsync/GetPagedListAsync...

// Custom (interface in Domain, impl in EF Core layer)
public interface IBookRepository : IRepository<Book, Guid> { Task<Book> FindByNameAsync(string name); }
public class BookRepository : EfCoreRepository<MyDbContext, Book, Guid>, IBookRepository { /* ... */ }

// Eager loading
var q = await _orderRepository.WithDetailsAsync(x => x.Lines);
var list = await AsyncExecuter.ToListAsync(q);
```
> Don't expose `IQueryable`; don't return a projection class. Details: [EF Core](../abp-efcore/SKILL.md).

## Unit of Work

UOW is **automatic** in app service / controller / repository methods. HTTP GET is not transactional.

```csharp
await _repo.InsertAsync(entity, autoSave: true);            // preferred
[UnitOfWork(IsTransactional = false)] public virtual async Task FooAsync() { }
using (var uow = _uowManager.Begin(requiresNew: true)) { /* ... */ await uow.CompleteAsync(); }
```

## Specification

```csharp
public class ProductsByCategorySpec : Specification<Product>
{
    private readonly Guid _categoryId;
    public ProductsByCategorySpec(Guid id) => _categoryId = id;
    public override Expression<Func<Product, bool>> ToExpression() => p => p.CategoryId == _categoryId && !p.IsDeleted;
}
var products = await _productRepository.GetListAsync(new ProductsByCategorySpec(categoryId));
```

## Extra Properties

```csharp
user.SetProperty("Title", "Dr.");
var title = user.GetProperty<string>("Title");
```

## Best Practices

1. Aggregate Root + protected setter + constructor validation
2. Use DTOs, don't expose entities
3. Mapperly `[Mapper]` partial class
4. `IGuidGenerator.Create()` sequential GUID, `Clock` for time
5. Soft-delete → `FullAuditedAggregateRoot`
6. Reduce boilerplate with CrudAppService, rely on UOW conventions

## Related

- [EF Core](../abp-efcore/SKILL.md) · [MongoDB](../abp-mongodb/SKILL.md) · [Object Mapping](../abp-object-mapping/SKILL.md) · [Validation](../abp-validation/SKILL.md) · [Dependency Rules](../abp-dependency-rules/SKILL.md) · [Development Flow](../abp-development-flow/SKILL.md)
- ABP Docs: https://abp.io/docs/latest/framework/architecture/domain-driven-design

---
name: abp-mongodb
description: "ABP Framework v10.4 MongoDB: AbpMongoDbContext, collection mapping, index, transaction, replica set, repository. Use when working with MongoDB, document databases, or MongoDB repositories in ABP."
---

# ABP Framework — MongoDB

ABP Framework v10.4 MongoDB integration guide. MongoDbContext, collection mapping, repository, indexes, transactions.

## Trigger

- "ABP MongoDB"
- "ABP MongoDbContext"
- "ABP Mongo repository"
- "ABP Mongo collection"
- "ABP Mongo index"
- "ABP Mongo transaction"

## Installation

```bash
abp add-package Volo.Abp.MongoDB
```

```csharp
[DependsOn(typeof(AbpMongoDbModule))]
public class MyModule : AbpModule { }
```

## MongoDbContext

```csharp
public class MyDbContext : AbpMongoDbContext
{
    public IMongoCollection<Question> Questions => Collection<Question>();
    public IMongoCollection<Category> Categories => Collection<Category>();

    protected override void CreateModel(IMongoModelBuilder modelBuilder)
    {
        base.CreateModel(modelBuilder);
        // Collection configuration
    }
}
```

### Collection Mapping

```csharp
protected override void CreateModel(IMongoModelBuilder modelBuilder)
{
    base.CreateModel(modelBuilder);

    modelBuilder.Entity<Question>(b =>
    {
        b.CollectionName = "MyQuestions";
        b.BsonMap.UnmapProperty(x => x.MyProperty);  // Ignore property
    });
}
```

Or via attribute:

```csharp
[MongoCollection("MyQuestions")]
public IMongoCollection<Question> Questions => Collection<Question>();
```

### Index Configuration

```csharp
protected override void CreateModel(IMongoModelBuilder modelBuilder)
{
    base.CreateModel(modelBuilder);

    modelBuilder.Entity<Question>(b =>
    {
        b.CreateCollectionOptions.Collation = new Collation(locale: "en_US", strength: CollationStrength.Secondary);
        b.ConfigureIndexes(indexes =>
        {
            indexes.CreateOne(
                new CreateIndexModel<BsonDocument>(
                    Builders<BsonDocument>.IndexKeys.Ascending("MyProperty"),
                    new CreateIndexOptions { Unique = true }
                )
            );
        });
    });
}
```

## DbContext Registration

```csharp
context.Services.AddMongoDbContext<MyDbContext>(options =>
{
    options.AddDefaultRepositories();
    // options.AddDefaultRepositories(includeAllEntities: true);
});
```

## Using the Default Repository

```csharp
public class BookManager : DomainService
{
    private readonly IRepository<Book, Guid> _bookRepository;

    public BookManager(IRepository<Book, Guid> bookRepository) => _bookRepository = bookRepository;

    public async Task<Book> CreateBookAsync(string name, BookType type)
    {
        var book = new Book(GuidGenerator.Create(), name, type);
        await _bookRepository.InsertAsync(book);
        return book;
    }
}
```

## Custom Repository

```csharp
// Interface (Domain layer)
public interface IBookRepository : IRepository<Book, Guid>
{
    Task DeleteBooksByTypeAsync(BookType type, CancellationToken cancellationToken = default);
}

// Implementation (MongoDB layer)
public class BookRepository : MongoDbRepository<MyMongoDbContext, Book, Guid>, IBookRepository
{
    public BookRepository(IMongoDbContextProvider<MyMongoDbContext> dbContextProvider)
        : base(dbContextProvider) { }

    public async Task DeleteBooksByTypeAsync(BookType type, CancellationToken cancellationToken = default)
    {
        var collection = await GetCollectionAsync(cancellationToken);
        await collection.DeleteManyAsync(
            Builders<Book>.Filter.Eq(b => b.Type, type),
            cancellationToken
        );
    }
}
```

### Default Repository Override

```csharp
context.Services.AddMongoDbContext<MyMongoDbContext>(options =>
{
    options.AddDefaultRepositories();
    options.AddRepository<Book, BookRepository>();
});
```

## MongoDB API Access

```csharp
public class BookService
{
    private readonly IRepository<Book, Guid> _bookRepository;
    public BookService(IRepository<Book, Guid> bookRepository) => _bookRepository = bookRepository;

    public async Task FooAsync()
    {
        IMongoDatabase database = await _bookRepository.GetDatabaseAsync();
        IMongoCollection<Book> books = await _bookRepository.GetCollectionAsync();
        IAggregateFluent<Book> aggregate = await _bookRepository.GetAggregateAsync();
    }
}
```

> A reference to the `Volo.Abp.MongoDB` package is required.

## Transactions

MongoDB 4.0+ supports multi-document transactions. In the startup templates, transactions are **disabled by default**.

### Enabling Transactions

```csharp
// REMOVE the following code from the YourProjectMongoDbModule class:
// context.Services.AddAlwaysDisableUnitOfWorkTransaction();
// Configure<AbpUnitOfWorkDefaultOptions>(options =>
// {
//     options.TransactionBehavior = UnitOfWorkTransactionBehavior.Disabled;
// });
```

### Docker Replica Set (for Transactions)

```yaml
version: "3.8"
services:
  mongo:
    image: mongo:8.0
    command: ["--replSet", "rs0", "--bind_ip_all", "--port", "27017"]
    ports:
      - 27017:27017
    healthcheck:
      test: echo "try { rs.status() } catch (err) { rs.initiate({_id:'rs0',members:[{_id:0,host:'127.0.0.1:27017'}]}) }" | mongosh --port 27017 --quiet
      interval: 5s
      timeout: 30s
      start_period: 0s
      start_interval: 1s
      retries: 30
```

Connection string: `mongodb://localhost:27017/YourProjectName?replicaSet=rs0`

## Connection String Selection

```csharp
[ConnectionStringName("MySecondConnString")]
public class MyDbContext : AbpMongoDbContext { }
```

## Multi-Tenancy

```csharp
[IgnoreMultiTenancy]  // Always use the host connection string
public class TenantManagementMongoDbContext : AbpMongoDbContext { }
```

## Default Repository Base Class

```csharp
public class MyRepositoryBase<TEntity> : MongoDbRepository<MyMongoDbContext, TEntity>
    where TEntity : class, IEntity
{
    public MyRepositoryBase(IMongoDbContextProvider<MyMongoDbContext> dbContextProvider)
        : base(dbContextProvider) { }
}

context.Services.AddMongoDbContext<MyMongoDbContext>(options =>
{
    options.SetDefaultRepositoryClasses(typeof(MyRepositoryBase<,>), typeof(MyRepositoryBase<>));
});
```

## ReplaceDbContext Pattern

```csharp
public interface IBookStoreMongoDbContext : IAbpMongoDbContext
{
    IMongoCollection<Book> Books { get; }
}

[ReplaceDbContext(typeof(IBookStoreMongoDbContext))]
public class OtherMongoDbContext : AbpMongoDbContext, IBookStoreMongoDbContext { }

// or
context.Services.AddMongoDbContext<OtherMongoDbContext>(options =>
{
    options.ReplaceDbContext<IBookStoreMongoDbContext>();
});
```

## Bulk Operations Customization

```csharp
public class MyCustomMongoDbBulkOperationProvider : IMongoDbBulkOperationProvider, ITransientDependency
{
    public async Task InsertManyAsync<TEntity>(
        IMongoDbRepository<TEntity> repository,
        IEnumerable<TEntity> entities,
        IClientSessionHandle sessionHandle,
        bool autoSave,
        CancellationToken cancellationToken)
    {
        // Custom bulk insert logic
    }

    public async Task UpdateManyAsync<TEntity>(...) { }
    public async Task DeleteManyAsync<TEntity>(...) { }
}
```

## EF Core vs MongoDB Comparison

| Feature | EF Core | MongoDB |
|---|---|---|
| DbContext | `AbpDbContext<T>` | `AbpMongoDbContext` |
| Collection | `DbSet<T>` | `IMongoCollection<T>` |
| Mapping | Fluent API / Data Annotations | `IMongoModelBuilder` |
| Repository Base | `EfCoreRepository` | `MongoDbRepository` |
| Query | `IQueryable` (LINQ) | MongoDB Filter/Aggregate |
| Transaction | Enabled by default | Disabled by default |
| Change Tracking | Yes | No |

## Best Practices

1. **Use `AbpMongoDbContext`** — Instead of directly using `IMongoDatabase`
2. **Define collection names via attribute** — `[MongoCollection("name")]`
3. **Define indexes in `CreateModel`** — A migration-like approach
4. **Set up a replica set if you need transactions** — A single node does not support transactions
5. **Define custom repositories in the MongoDB layer** — Only the interface in the domain layer
6. **Use `IAsyncQueryableExecuter`** — Keep the domain layer isolated from MongoDB
7. **Extra Properties are natively supported in MongoDB** — Stored as a separate element instead of a JSON field

---

## Related

- [DDD](../abp-ddd/SKILL.md) — entity, aggregate, repository pattern
- [EF Core](../abp-efcore/SKILL.md) — relational database alternative
- [Dependency Rules](../abp-dependency-rules/SKILL.md) — repository interface/impl placement
- [Multi-Tenancy](../abp-multitenancy/SKILL.md) — tenant data isolation
- ABP Docs: https://abp.io/docs/latest/framework/data/mongodb

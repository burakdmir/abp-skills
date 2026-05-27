# ABP Framework — MongoDB

ABP Framework v10.4 MongoDB entegrasyon rehberi. MongoDbContext, collection mapping, repository, indexes, transactions.

## Trigger

- "ABP MongoDB"
- "ABP MongoDbContext"
- "ABP Mongo repository"
- "ABP Mongo collection"
- "ABP Mongo index"
- "ABP Mongo transaction"

## Kurulum

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
        // Collection konfigürasyonu
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
        b.BsonMap.UnmapProperty(x => x.MyProperty);  // Property ignore
    });
}
```

Veya attribute ile:

```csharp
[MongoCollection("MyQuestions")]
public IMongoCollection<Question> Questions => Collection<Question>();
```

### Index Konfigürasyonu

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

## DbContext Kaydı

```csharp
context.Services.AddMongoDbContext<MyDbContext>(options =>
{
    options.AddDefaultRepositories();
    // options.AddDefaultRepositories(includeAllEntities: true);
});
```

## Default Repository Kullanımı

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

## MongoDB API Erişimi

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

> `Volo.Abp.MongoDB` package referansı gerekli.

## Transactions

MongoDB 4.0+ multi-document transaction destekler. Startup template'lerde transaction **varsayılan olarak kapalıdır**.

### Transaction Enable Etme

```csharp
// YourProjectMongoDbModule class'ında şu kodları KALDIR:
// context.Services.AddAlwaysDisableUnitOfWorkTransaction();
// Configure<AbpUnitOfWorkDefaultOptions>(options =>
// {
//     options.TransactionBehavior = UnitOfWorkTransactionBehavior.Disabled;
// });
```

### Docker Replica Set (Transaction için)

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

## Connection String Seçimi

```csharp
[ConnectionStringName("MySecondConnString")]
public class MyDbContext : AbpMongoDbContext { }
```

## Multi-Tenancy

```csharp
[IgnoreMultiTenancy]  // Her zaman host connection string kullan
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

// veya
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

## EF Core vs MongoDB Karşılaştırma

| Özellik | EF Core | MongoDB |
|---|---|---|
| DbContext | `AbpDbContext<T>` | `AbpMongoDbContext` |
| Collection | `DbSet<T>` | `IMongoCollection<T>` |
| Mapping | Fluent API / Data Annotations | `IMongoModelBuilder` |
| Repository Base | `EfCoreRepository` | `MongoDbRepository` |
| Query | `IQueryable` (LINQ) | MongoDB Filter/Aggregate |
| Transaction | Varsayılan açık | Varsayılan kapalı |
| Change Tracking | Var | Yok |

## Best Practices

1. **`AbpMongoDbContext` kullan** — Direkt `IMongoDatabase` yerine
2. **Collection name'leri attribute ile tanımla** — `[MongoCollection("name")]`
3. **Index'leri `CreateModel`'da tanımla** — Migration benzeri yaklaşım
4. **Transaction gerekiyorsa replica set kur** — Single node transaction desteklemez
5. **Custom repository'leri MongoDB layer'da tanımla** — Domain layer'da sadece interface
6. **`IAsyncQueryableExecuter` kullan** — Domain layer'ı MongoDB'den izole tut
7. **Extra Properties MongoDB'de native desteklenir** — JSON field yerine ayrı element olarak saklanır

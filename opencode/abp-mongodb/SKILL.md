# ABP Framework — MongoDB

ABP Framework v10.4 MongoDB entegrasyonu. MongoDbContext, collection mapping, repository, indexes, transactions.

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
        modelBuilder.Entity<Question>(b => { b.CollectionName = "MyQuestions"; b.BsonMap.UnmapProperty(x => x.MyProperty); });
    }
}
```

Veya attribute: `[MongoCollection("MyQuestions")]`

### Index

```csharp
modelBuilder.Entity<Question>(b =>
{
    b.CreateCollectionOptions.Collation = new Collation(locale: "en_US", strength: CollationStrength.Secondary);
    b.ConfigureIndexes(indexes => indexes.CreateOne(new CreateIndexModel<BsonDocument>(Builders<BsonDocument>.IndexKeys.Ascending("MyProperty"), new CreateIndexOptions { Unique = true })));
});
```

## DbContext Kaydı

```csharp
context.Services.AddMongoDbContext<MyDbContext>(options => options.AddDefaultRepositories());
```

## Custom Repository

```csharp
public interface IBookRepository : IRepository<Book, Guid> { Task DeleteBooksByTypeAsync(BookType type); }

public class BookRepository : MongoDbRepository<MyMongoDbContext, Book, Guid>, IBookRepository
{
    public BookRepository(IMongoDbContextProvider<MyMongoDbContext> dbContextProvider) : base(dbContextProvider) { }
    public async Task DeleteBooksByTypeAsync(BookType type)
    {
        var collection = await GetCollectionAsync();
        await collection.DeleteManyAsync(Builders<Book>.Filter.Eq(b => b.Type, type));
    }
}
```

## MongoDB API Erişimi

```csharp
IMongoDatabase db = await _repository.GetDatabaseAsync();
IMongoCollection<Book> coll = await _repository.GetCollectionAsync();
IAggregateFluent<Book> agg = await _repository.GetAggregateAsync();
```

## Transactions

MongoDB 4.0+ transaction destekler. Startup template'lerde **varsayılan kapalı**.

**Enable:** `YourProjectMongoDbModule`'da `AddAlwaysDisableUnitOfWorkTransaction()` ve `TransactionBehavior = Disabled` kodlarını kaldır.

**Docker Replica Set:**
```yaml
services:
  mongo:
    image: mongo:8.0
    command: ["--replSet", "rs0", "--bind_ip_all", "--port", "27017"]
    ports: [27017:27017]
```
Connection: `mongodb://localhost:27017/YourProjectName?replicaSet=rs0`

## EF Core vs MongoDB

| Özellik | EF Core | MongoDB |
|---|---|---|
| DbContext | `AbpDbContext<T>` | `AbpMongoDbContext` |
| Query | `IQueryable` (LINQ) | Filter/Aggregate |
| Transaction | Varsayılan açık | Varsayılan kapalı |
| Change Tracking | Var | Yok |
| Extra Properties | JSON field | Native element |

## Best Practices

1. `AbpMongoDbContext` kullan
2. Collection name'leri `[MongoCollection]` ile tanımla
3. Index'leri `CreateModel`'da tanımla
4. Transaction için replica set kur
5. Custom repository'leri MongoDB layer'da tanımla

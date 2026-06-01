---
name: abp-mongodb
description: "ABP Framework v10.4 MongoDB: AbpMongoDbContext, collection mapping, index, transaction, replica set, repository. Use when working with MongoDB, document databases, or MongoDB repositories in ABP."
---

# ABP Framework — MongoDB

ABP Framework v10.4 MongoDB integration. MongoDbContext, collection mapping, repository, indexes, transactions.

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
        modelBuilder.Entity<Question>(b => { b.CollectionName = "MyQuestions"; b.BsonMap.UnmapProperty(x => x.MyProperty); });
    }
}
```

Or via attribute: `[MongoCollection("MyQuestions")]`

### Index

```csharp
modelBuilder.Entity<Question>(b =>
{
    b.CreateCollectionOptions.Collation = new Collation(locale: "en_US", strength: CollationStrength.Secondary);
    b.ConfigureIndexes(indexes => indexes.CreateOne(new CreateIndexModel<BsonDocument>(Builders<BsonDocument>.IndexKeys.Ascending("MyProperty"), new CreateIndexOptions { Unique = true })));
});
```

## DbContext Registration

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

## MongoDB API Access

```csharp
IMongoDatabase db = await _repository.GetDatabaseAsync();
IMongoCollection<Book> coll = await _repository.GetCollectionAsync();
IAggregateFluent<Book> agg = await _repository.GetAggregateAsync();
```

## Transactions

MongoDB 4.0+ supports transactions. In the startup templates they are **disabled by default**.

**Enable:** In `YourProjectMongoDbModule`, remove the `AddAlwaysDisableUnitOfWorkTransaction()` and `TransactionBehavior = Disabled` code.

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

| Feature | EF Core | MongoDB |
|---|---|---|
| DbContext | `AbpDbContext<T>` | `AbpMongoDbContext` |
| Query | `IQueryable` (LINQ) | Filter/Aggregate |
| Transaction | Enabled by default | Disabled by default |
| Change Tracking | Yes | No |
| Extra Properties | JSON field | Native element |

## Best Practices

1. Use `AbpMongoDbContext`
2. Define collection names with `[MongoCollection]`
3. Define indexes in `CreateModel`
4. Set up a replica set for transactions
5. Define custom repositories in the MongoDB layer

## Related

[DDD](../abp-ddd/SKILL.md) · [EF Core](../abp-efcore/SKILL.md) · [Dependency Rules](../abp-dependency-rules/SKILL.md) · [Multi-Tenancy](../abp-multitenancy/SKILL.md) · Docs: https://abp.io/docs/latest/framework/data/mongodb

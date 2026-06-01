---
name: abp-efcore
description: "ABP Framework v10.4 Entity Framework Core: AbpDbContext, ConfigureByConvention, AddAbpDbContext, repository (EfCoreRepository), migration, PostgreSQL/MySQL/SQLite/Oracle. Use when working with EF Core, DbContext, migrations, or repository implementation in ABP."
---

# ABP Framework — Entity Framework Core

A guide to EF Core integration in ABP Framework v10.4. DbContext, repository, migration, eager/lazy loading.

## Trigger

- "ABP EF Core"
- "ABP DbContext"
- "ABP migration"
- "ABP repository EF"
- "ABP ConfigureByConvention"
- "ABP WithDetails"
- "ABP DbSet"

## DbContext

```csharp
public class MyDbContext : AbpDbContext<MyDbContext>
{
    public DbSet<Book> Books { get; set; }
    public MyDbContext(DbContextOptions<MyDbContext> options) : base(options) { }
}
```

## Entity Mapping

```csharp
protected override void OnModelCreating(ModelBuilder builder)
{
    base.OnModelCreating(builder);
    builder.Entity<Book>(b =>
    {
        b.ToTable("Books");
        b.ConfigureByConvention();  // REQUIRED
        b.Property(x => x.Name).IsRequired().HasMaxLength(128);
    });
}
```

## DbContext Registration

```csharp
context.Services.AddAbpDbContext<MyDbContext>(options =>
{
    options.AddDefaultRepositories();
});
```

## DBMS Configuration

```csharp
Configure<AbpDbContextOptions>(options =>
{
    options.UseSqlServer();
    options.Configure<MyOtherDbContext>(opts => opts.UseMySQL());
});
```

## Custom Repository

```csharp
public interface IBookRepository : IRepository<Book, Guid>
{
    Task DeleteBooksByType(BookType type);
}

public class BookRepository : EfCoreRepository<BookStoreDbContext, Book, Guid>, IBookRepository
{
    public BookRepository(IDbContextProvider<BookStoreDbContext> dbContextProvider)
        : base(dbContextProvider) { }

    public async Task DeleteBooksByType(BookType type)
    {
        var dbContext = await GetDbContextAsync();
        await dbContext.Database.ExecuteSqlRawAsync($"DELETE FROM Books WHERE Type = {(int)type}");
    }
}
```

## Eager Loading

```csharp
var queryable = await _orderRepository.WithDetailsAsync(x => x.Lines);
var orders = await AsyncExecuter.ToListAsync(queryable);
```

## Object Extension Manager

```csharp
ObjectExtensionManager.Instance
    .MapEfCoreProperty<IdentityRole, string>("Title",
        (entityBuilder, propertyBuilder) => propertyBuilder.HasMaxLength(64));
```

## Choosing a DBMS

```bash
abp new Acme.BookStore -dbms PostgreSQL
abp new Acme.BookStore -dbms MySQL
abp new Acme.BookStore -dbms SQLite
abp new Acme.BookStore -dbms Oracle
```

### PostgreSQL

```csharp
// Add the Volo.Abp.EntityFrameworkCore.PostgreSql package
[DependsOn(typeof(AbpEntityFrameworkCorePostgreSqlModule))]
Configure<AbpDbContextOptions>(options => options.UseNpgsql());
AppContext.SetSwitch("Npgsql.EnableLegacyTimestampBehavior", true);
```

### MySQL

```csharp
Configure<AbpDbContextOptions>(options =>
{
    options.Configure(ctx =>
    {
        ctx.DbContextOptions.UseMySql(ctx.ExistingConnection ?? ctx.ConnectionString);
    });
});
// Module setting: builder.ConfigureOpenIddict(o => o.DatabaseProvider = EfCoreDatabaseProvider.MySql);
```

## Migrations

```bash
dotnet ef migrations add InitialCreate --project Acme.BookStore.EntityFrameworkCore --startup-project Acme.BookStore.DbMigrator
dotnet ef database update --project Acme.BookStore.EntityFrameworkCore --startup-project Acme.BookStore.DbMigrator
```

## ReplaceDbContext

```csharp
public interface IBookStoreDbContext : IEfCoreDbContext { DbSet<Book> Books { get; } }

context.Services.AddAbpDbContext<BookStoreDbContext>(options => options.AddDefaultRepositories<IBookStoreDbContext>());

[ReplaceDbContext(typeof(IBookStoreDbContext))]
public class UnifiedDbContext : AbpDbContext<UnifiedDbContext>, IBookStoreDbContext { }
```

## Best Practices

1. Always call `ConfigureByConvention()`
2. Keep the domain layer isolated from EF Core
3. Use `IReadOnlyRepository` for read-only queries
4. Do eager loading with `WithDetailsAsync`

## Related

[DDD](../abp-ddd/SKILL.md) · [MongoDB](../abp-mongodb/SKILL.md) · [Dependency Rules](../abp-dependency-rules/SKILL.md) · [Development Flow](../abp-development-flow/SKILL.md) · Docs: https://abp.io/docs/latest/framework/data/entity-framework-core

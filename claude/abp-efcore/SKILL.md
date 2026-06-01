---
name: abp-efcore
description: "ABP Framework v10.4 Entity Framework Core: AbpDbContext, ConfigureByConvention, AddAbpDbContext, repository (EfCoreRepository), migration, PostgreSQL/MySQL/SQLite/Oracle. Use when working with EF Core, DbContext, migrations, or repository implementation in ABP."
---

# ABP Framework — Entity Framework Core

A guide to EF Core integration in ABP Framework v10.4. DbContext, repository, migration, eager/lazy loading, and advanced topics.

## Trigger

- "ABP EF Core"
- "ABP DbContext"
- "ABP migration"
- "ABP repository EF"
- "ABP ConfigureByConvention"
- "ABP WithDetails"
- "ABP DbSet"
- "ABP entity mapping"

## Installation

```bash
abp add-package Volo.Abp.EntityFrameworkCore
```

## Creating a DbContext

```csharp
using Microsoft.EntityFrameworkCore;
using Volo.Abp.EntityFrameworkCore;

namespace MyCompany.MyProject
{
    public class MyDbContext : AbpDbContext<MyDbContext>
    {
        public DbSet<Book> Books { get; set; }
        public DbSet<Author> Authors { get; set; }

        public MyDbContext(DbContextOptions<MyDbContext> options)
            : base(options) { }
    }
}
```

## Entity Mapping (Fluent API)

```csharp
protected override void OnModelCreating(ModelBuilder builder)
{
    base.OnModelCreating(builder);

    builder.Entity<Book>(b =>
    {
        b.ToTable("Books");
        b.ConfigureByConvention();  // REQUIRED for base properties
        b.Property(x => x.Name).IsRequired().HasMaxLength(128);
        b.HasIndex(x => x.Name);
    });
}
```

**`ConfigureByConvention()` must always be called** — it automatically configures base class properties (Id, CreationTime, etc.).

## DbContext Registration

```csharp
[DependsOn(typeof(AbpEntityFrameworkCoreModule))]
public class MyModule : AbpModule
{
    public override void ConfigureServices(ServiceConfigurationContext context)
    {
        context.Services.AddAbpDbContext<MyDbContext>(options =>
        {
            options.AddDefaultRepositories();  // Automatic repository for AggregateRoots
            // options.AddDefaultRepositories(includeAllEntities: true);  // For all entities
        });
    }
}
```

## DBMS Configuration

### Choosing a DBMS with the CLI

```bash
abp new Acme.BookStore -dbms PostgreSQL
abp new Acme.BookStore -dbms MySQL
abp new Acme.BookStore -dbms SQLite
abp new Acme.BookStore -dbms Oracle
```

### Manually Changing the DBMS

**PostgreSQL:**
```bash
# 1. Remove the Volo.Abp.EntityFrameworkCore.SqlServer package
# 2. Add the Volo.Abp.EntityFrameworkCore.PostgreSql package
```

```csharp
// Change the module dependency
[DependsOn(typeof(AbpEntityFrameworkCorePostgreSqlModule))]  // instead of SqlServer

// Use UseNpgsql()
Configure<AbpDbContextOptions>(options => options.UseNpgsql());

// Also use UseNpgsql() in the DbContextFactory

// Enable legacy timestamp behavior (Npgsql 6.0+)
AppContext.SetSwitch("Npgsql.EnableLegacyTimestampBehavior", true);
```

**MySQL (Pomelo):**
```csharp
Configure<AbpDbContextOptions>(options =>
{
    options.Configure(ctx =>
    {
        if (ctx.ExistingConnection != null)
            ctx.DbContextOptions.UseMySql(ctx.ExistingConnection);
        else
            ctx.DbContextOptions.UseMySql(ctx.ConnectionString);
    });
});

// Set the DBMS provider for modules like OpenIddict (default auth server in ABP v6.0+: OpenIddict)
builder.ConfigureOpenIddict(options =>
{
    options.DatabaseProvider = EfCoreDatabaseProvider.MySql;
});
```

**SQLite:**
```csharp
Configure<AbpDbContextOptions>(options => options.UseSqlite());
```

### Supported DBMSs

| DBMS | Package |
|---|---|
| SQL Server | `Volo.Abp.EntityFrameworkCore.SqlServer` |
| PostgreSQL | `Volo.Abp.EntityFrameworkCore.PostgreSql` |
| MySQL | `Volo.Abp.EntityFrameworkCore.MySQL` |
| SQLite | `Volo.Abp.EntityFrameworkCore.Sqlite` |
| Oracle | `Volo.Abp.EntityFrameworkCore.Oracle` |

## Choosing a Connection String

```csharp
[ConnectionStringName("MySecondConnString")]
public class MyDbContext : AbpDbContext<MyDbContext> { }
```

If not specified, the `Default` connection string is used.

## Using the Default Repository

```csharp
public class BookManager : DomainService
{
    private readonly IRepository<Book, Guid> _bookRepository;

    public BookManager(IRepository<Book, Guid> bookRepository)
    {
        _bookRepository = bookRepository;
    }

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
    Task DeleteBooksByType(BookType type);
}

// Implementation (EF Core layer)
public class BookRepository : EfCoreRepository<BookStoreDbContext, Book, Guid>, IBookRepository
{
    public BookRepository(IDbContextProvider<BookStoreDbContext> dbContextProvider)
        : base(dbContextProvider) { }

    public async Task DeleteBooksByType(BookType type)
    {
        var dbContext = await GetDbContextAsync();
        await dbContext.Database.ExecuteSqlRawAsync(
            $"DELETE FROM Books WHERE Type = {(int)type}"
        );
    }
}
```

### Overriding the Default Repository

```csharp
context.Services.AddAbpDbContext<BookStoreDbContext>(options =>
{
    options.AddDefaultRepositories();
    options.AddRepository<Book, BookRepository>();  // BookRepository is used instead of IRepository<Book, Guid>
});
```

## Eager Loading (WithDetails)

```csharp
// Repository.WithDetailsAsync
var queryable = await _orderRepository.WithDetailsAsync(x => x.Lines);
var orders = await AsyncExecuter.ToListAsync(queryable);

// DefaultWithDetailsFunc configuration
Configure<AbpEntityOptions>(options =>
{
    options.Entity<Order>(orderOptions =>
    {
        orderOptions.DefaultWithDetailsFunc = query => query.Include(o => o.Lines);
    });
});

// It can then be used without a parameter
var queryable = await _orderRepository.WithDetailsAsync();
```

### includeDetails on Get/Find Methods

```csharp
var order = await _orderRepository.GetAsync(id);  // includeDetails: true (default)
var order = await _orderRepository.GetAsync(id, includeDetails: false);
var orders = await _orderRepository.GetListAsync(includeDetails: true);
```

## Explicit Loading

```csharp
var order = await _orderRepository.GetAsync(id, includeDetails: false);
await _orderRepository.EnsureCollectionLoadedAsync(order, x => x.Lines);
// order.Lines is now populated
```

## Lazy Loading

```csharp
// 1. Install the Microsoft.EntityFrameworkCore.Proxies package
// 2. Configuration
Configure<AbpDbContextOptions>(options =>
{
    options.PreConfigure<MyDbContext>(opts =>
    {
        opts.DbContextOptions.UseLazyLoadingProxies();
    });
    options.UseSqlServer();
});

// 3. Make navigation properties virtual
public virtual ICollection<OrderLine> Lines { get; set; }
public virtual Order Order { get; set; }
```

## Read-Only Repositories

```csharp
// No-Tracking is applied automatically
public class MyService : ApplicationService
{
    private readonly IReadOnlyRepository<Book, Guid> _bookRepository;
    
    // When tracking is needed
    var query = (await _bookRepository.GetQueryableAsync()).AsTracking();
}
```

## Object Extension Manager (Extra Properties)

```csharp
ObjectExtensionManager.Instance
    .MapEfCoreProperty<IdentityRole, string>(
        "Title",
        (entityBuilder, propertyBuilder) =>
        {
            propertyBuilder.HasMaxLength(64);
        }
    );
```

> `MapEfCoreProperty` must be called before the DbContext is used. In startup templates, the `EfCoreEntityExtensionMappings` class is the safe spot.

## Split Queries

```csharp
Configure<AbpDbContextOptions>(options =>
{
    options.UseSqlServer(optionsBuilder =>
    {
        optionsBuilder.UseQuerySplittingBehavior(QuerySplittingBehavior.SingleQuery);
    });
});
```

## EF Core with Multi-Tenancy

```csharp
[IgnoreMultiTenancy]  // Always use the host connection string
public class TenantManagementDbContext : AbpDbContext<TenantManagementDbContext> { }
```

## Default Repository Base Class

```csharp
public class MyRepositoryBase<TEntity> : EfCoreRepository<BookStoreDbContext, TEntity>
    where TEntity : class, IEntity
{
    public MyRepositoryBase(IDbContextProvider<BookStoreDbContext> dbContextProvider)
        : base(dbContextProvider) { }
}

// Registration
context.Services.AddAbpDbContext<BookStoreDbContext>(options =>
{
    options.SetDefaultRepositoryClasses(
        typeof(MyRepositoryBase<,>),
        typeof(MyRepositoryBase<>)
    );
});
```

## Best Practices

1. **Always call `ConfigureByConvention()`** — for base property mapping
2. **Prefer the Fluent API** — over data annotations
3. **Keep the domain layer isolated from EF Core** — use `IAsyncQueryableExecuter`
4. **Do eager loading with `WithDetailsAsync`** — to avoid the N+1 problem
5. **Use `IReadOnlyRepository` for read-only queries** — No-Tracking is automatic
6. **Define custom repositories in the EF Core layer** — only the interface in the domain layer
7. **Manage migrations in the EF Core project** — `Add-Migration`, `Update-Database`

---

## Migrations

```bash
# Create a migration
dotnet ef migrations add InitialCreate --project Acme.BookStore.EntityFrameworkCore --startup-project Acme.BookStore.DbMigrator

# Apply the migration
dotnet ef database update --project Acme.BookStore.EntityFrameworkCore --startup-project Acme.BookStore.DbMigrator

# or run the DbMigrator
dotnet run --project Acme.BookStore.DbMigrator
```

### Multiple DbContext Migrations

```csharp
// A separate migration folder for each DbContext
builder.Entity<Book>(b => { ... });  // BookStoreDbContext

// Migration design
public class BookStoreDbContextModelSnapshot : ModelSnapshot { }
```

### Seed Data via Migration

```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.InsertData(
        table: "Books",
        columns: new[] { "Id", "Name", "Price" },
        values: new object[] { Guid.NewGuid(), "1984", 29.99m }
    );
}
```

---

## ReplaceDbContext Pattern

Combining multiple DbContexts into a single DbContext:

```csharp
// Define an interface
public interface IBookStoreDbContext : IEfCoreDbContext
{
    DbSet<Book> Books { get; }
}

// Have the DbContext implement the interface
public class BookStoreDbContext : AbpDbContext<BookStoreDbContext>, IBookStoreDbContext
{
    public DbSet<Book> Books { get; set; }
}

// Register the default repositories with the interface
context.Services.AddAbpDbContext<BookStoreDbContext>(options =>
{
    options.AddDefaultRepositories<IBookStoreDbContext>();
});

// Another DbContext can replace this interface
[ReplaceDbContext(typeof(IBookStoreDbContext))]
public class UnifiedDbContext : AbpDbContext<UnifiedDbContext>, IBookStoreDbContext
{
    public DbSet<Book> Books { get; set; }
}
```

---

## Bulk Operations Customization

```csharp
public class MyCustomEfCoreBulkOperationProvider : IEfCoreBulkOperationProvider, ITransientDependency
{
    public async Task InsertManyAsync<TDbContext, TEntity>(
        IEfCoreRepository<TEntity> repository,
        IEnumerable<TEntity> entities,
        bool autoSave,
        CancellationToken cancellationToken)
    {
        // Custom bulk insert logic (e.g. EFCore.BulkExtensions)
    }

    public async Task UpdateManyAsync<TDbContext, TEntity>(...) { }
    public async Task DeleteManyAsync<TDbContext, TEntity>(...) { }
}
```

---

## Related

- [DDD](../abp-ddd/SKILL.md) — entity, aggregate, repository pattern
- [MongoDB](../abp-mongodb/SKILL.md) — MongoDB alternative
- [Dependency Rules](../abp-dependency-rules/SKILL.md) — repository interface in Domain, impl in the Data layer
- [Multi-Tenancy](../abp-multitenancy/SKILL.md) — IgnoreMultiTenancy, tenant connection string
- [Development Flow](../abp-development-flow/SKILL.md) — migration and DbMigrator flow
- ABP Docs: https://abp.io/docs/latest/framework/data/entity-framework-core

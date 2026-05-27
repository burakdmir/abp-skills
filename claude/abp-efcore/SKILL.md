# ABP Framework — Entity Framework Core

ABP Framework v10.4 EF Core entegrasyon rehberi. DbContext, repository, migration, eager/lazy loading ve ileri konular.

## Trigger

- "ABP EF Core"
- "ABP DbContext"
- "ABP migration"
- "ABP repository EF"
- "ABP ConfigureByConvention"
- "ABP WithDetails"
- "ABP DbSet"
- "ABP entity mapping"

## Kurulum

```bash
abp add-package Volo.Abp.EntityFrameworkCore
```

## DbContext Oluşturma

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
        b.ConfigureByConvention();  // Base properties için ZORUNLU
        b.Property(x => x.Name).IsRequired().HasMaxLength(128);
        b.HasIndex(x => x.Name);
    });
}
```

**`ConfigureByConvention()` her zaman çağrılmalı** — Base class property'lerini (Id, CreationTime, vb.) otomatik konfigüre eder.

## DbContext Kaydı

```csharp
[DependsOn(typeof(AbpEntityFrameworkCoreModule))]
public class MyModule : AbpModule
{
    public override void ConfigureServices(ServiceConfigurationContext context)
    {
        context.Services.AddAbpDbContext<MyDbContext>(options =>
        {
            options.AddDefaultRepositories();  // AggregateRoot'lar için otomatik repository
            // options.AddDefaultRepositories(includeAllEntities: true);  // Tüm entity'ler için
        });
    }
}
```

## DBMS Konfigürasyonu

### CLI ile DBMS Seçimi

```bash
abp new Acme.BookStore -dbms PostgreSQL
abp new Acme.BookStore -dbms MySQL
abp new Acme.BookStore -dbms SQLite
abp new Acme.BookStore -dbms Oracle
```

### Manuel DBMS Değiştirme

**PostgreSQL:**
```bash
# 1. Volo.Abp.EntityFrameworkCore.SqlServer paketini kaldır
# 2. Volo.Abp.EntityFrameworkCore.PostgreSql paketini ekle
```

```csharp
// Module dependency değiştir
[DependsOn(typeof(AbpEntityFrameworkCorePostgreSqlModule))]  // SqlServer yerine

// UseNpgsql() kullan
Configure<AbpDbContextOptions>(options => options.UseNpgsql());

// DbContextFactory'da da UseNpgsql() kullan

// Legacy timestamp behavior enable et (Npgsql 6.0+)
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

// IdentityServer gibi modüller için DBMS provider ayarla
builder.ConfigureIdentityServer(options =>
{
    options.DatabaseProvider = EfCoreDatabaseProvider.MySql;
});
```

**SQLite:**
```csharp
Configure<AbpDbContextOptions>(options => options.UseSqlite());
```

### Desteklenen DBMS'ler

| DBMS | Package |
|---|---|
| SQL Server | `Volo.Abp.EntityFrameworkCore.SqlServer` |
| PostgreSQL | `Volo.Abp.EntityFrameworkCore.PostgreSql` |
| MySQL | `Volo.Abp.EntityFrameworkCore.MySQL` |
| SQLite | `Volo.Abp.EntityFrameworkCore.Sqlite` |
| Oracle | `Volo.Abp.EntityFrameworkCore.Oracle` |

## Connection String Seçimi

```csharp
[ConnectionStringName("MySecondConnString")]
public class MyDbContext : AbpDbContext<MyDbContext> { }
```

Belirtilmezse `Default` connection string kullanılır.

## Default Repository Kullanımı

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

### Default Repository'yi Override Etme

```csharp
context.Services.AddAbpDbContext<BookStoreDbContext>(options =>
{
    options.AddDefaultRepositories();
    options.AddRepository<Book, BookRepository>();  // IRepository<Book, Guid> yerine BookRepository kullanılır
});
```

## Eager Loading (WithDetails)

```csharp
// Repository.WithDetailsAsync
var queryable = await _orderRepository.WithDetailsAsync(x => x.Lines);
var orders = await AsyncExecuter.ToListAsync(queryable);

// DefaultWithDetailsFunc konfigürasyonu
Configure<AbpEntityOptions>(options =>
{
    options.Entity<Order>(orderOptions =>
    {
        orderOptions.DefaultWithDetailsFunc = query => query.Include(o => o.Lines);
    });
});

// Sonra parametresiz kullanılabilir
var queryable = await _orderRepository.WithDetailsAsync();
```

### Get/Find Metodlarında includeDetails

```csharp
var order = await _orderRepository.GetAsync(id);  // includeDetails: true (varsayılan)
var order = await _orderRepository.GetAsync(id, includeDetails: false);
var orders = await _orderRepository.GetListAsync(includeDetails: true);
```

## Explicit Loading

```csharp
var order = await _orderRepository.GetAsync(id, includeDetails: false);
await _orderRepository.EnsureCollectionLoadedAsync(order, x => x.Lines);
// order.Lines artık dolu
```

## Lazy Loading

```csharp
// 1. Microsoft.EntityFrameworkCore.Proxies paketini yükle
// 2. Konfigürasyon
Configure<AbpDbContextOptions>(options =>
{
    options.PreConfigure<MyDbContext>(opts =>
    {
        opts.DbContextOptions.UseLazyLoadingProxies();
    });
    options.UseSqlServer();
});

// 3. Navigation property'leri virtual yap
public virtual ICollection<OrderLine> Lines { get; set; }
public virtual Order Order { get; set; }
```

## Read-Only Repositories

```csharp
// No-Tracking otomatik uygulanır
public class MyService : ApplicationService
{
    private readonly IReadOnlyRepository<Book, Guid> _bookRepository;
    
    // Tracking gerektiğinde
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

> `MapEfCoreProperty` DbContext kullanılmadan önce çağrılmalı. Startup template'lerde `EfCoreEntityExtensionMappings` class'ı güvenli noktadır.

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

## Multi-Tenancy ile EF Core

```csharp
[IgnoreMultiTenancy]  // Her zaman host connection string kullan
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

// Kayıt
context.Services.AddAbpDbContext<BookStoreDbContext>(options =>
{
    options.SetDefaultRepositoryClasses(
        typeof(MyRepositoryBase<,>),
        typeof(MyRepositoryBase<>)
    );
});
```

## Best Practices

1. **Her zaman `ConfigureByConvention()` çağır** — Base property mapping için
2. **Fluent API tercih et** — Data annotation yerine
3. **Domain layer'ı EF Core'dan izole tut** — `IAsyncQueryableExecuter` kullan
4. **`WithDetailsAsync` ile eager loading yap** — N+1 sorununu önle
5. **Read-only sorgular için `IReadOnlyRepository` kullan** — No-Tracking otomatik
6. **Custom repository'leri EF Core layer'da tanımla** — Domain layer'da sadece interface
7. **Migration'ları EF Core project'ında yönet** — `Add-Migration`, `Update-Database`

---

## Migrations

```bash
# Migration oluştur
dotnet ef migrations add InitialCreate --project Acme.BookStore.EntityFrameworkCore --startup-project Acme.BookStore.DbMigrator

# Migration uygula
dotnet ef database update --project Acme.BookStore.EntityFrameworkCore --startup-project Acme.BookStore.DbMigrator

# veya DbMigrator'ı çalıştır
dotnet run --project Acme.BookStore.DbMigrator
```

### Multiple DbContext Migrations

```csharp
// Her DbContext için ayrı migration klasörü
builder.Entity<Book>(b => { ... });  // BookStoreDbContext

// Migration tasarımı
public class BookStoreDbContextModelSnapshot : ModelSnapshot { }
```

### Migration ile Seed Data

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

Birden fazla DbContext'i tek DbContext'te birleştirme:

```csharp
// Interface tanımla
public interface IBookStoreDbContext : IEfCoreDbContext
{
    DbSet<Book> Books { get; }
}

// DbContext interface'i implement et
public class BookStoreDbContext : AbpDbContext<BookStoreDbContext>, IBookStoreDbContext
{
    public DbSet<Book> Books { get; set; }
}

// Default repository'leri interface ile kaydet
context.Services.AddAbpDbContext<BookStoreDbContext>(options =>
{
    options.AddDefaultRepositories<IBookStoreDbContext>();
});

// Başka bir DbContext bu interface'i replace edebilir
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
        // Custom bulk insert logic (örn. EFCore.BulkExtensions)
    }

    public async Task UpdateManyAsync<TDbContext, TEntity>(...) { }
    public async Task DeleteManyAsync<TDbContext, TEntity>(...) { }
}
```

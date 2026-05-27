# ABP Framework — Entity Framework Core

ABP Framework v10.4 EF Core entegrasyon rehberi. DbContext, repository, migration, eager/lazy loading.

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
        b.ConfigureByConvention();  // ZORUNLU
        b.Property(x => x.Name).IsRequired().HasMaxLength(128);
    });
}
```

## DbContext Kaydı

```csharp
context.Services.AddAbpDbContext<MyDbContext>(options =>
{
    options.AddDefaultRepositories();
});
```

## DBMS Konfigürasyonu

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

## DBMS Seçimi

```bash
abp new Acme.BookStore -dbms PostgreSQL
abp new Acme.BookStore -dbms MySQL
abp new Acme.BookStore -dbms SQLite
abp new Acme.BookStore -dbms Oracle
```

### PostgreSQL

```csharp
// Volo.Abp.EntityFrameworkCore.PostgreSql paketini ekle
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
// Modül ayarı: builder.ConfigureIdentityServer(o => o.DatabaseProvider = EfCoreDatabaseProvider.MySql);
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

1. `ConfigureByConvention()` her zaman çağır
2. Domain layer'ı EF Core'dan izole tut
3. Read-only sorgular için `IReadOnlyRepository` kullan
4. `WithDetailsAsync` ile eager loading yap

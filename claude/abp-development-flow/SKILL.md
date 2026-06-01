---
name: abp-development-flow
description: "ABP Framework v10.4 development flow: adding a new entity end-to-end (Domain → Domain.Shared → repository → EF Core config → migration → Contracts/DTO → object mapping → app service → localization → permission → test). Use when you need the flow for adding a new feature in ABP."
---

# ABP Framework — Development Flow

The flow for adding a new entity/feature **end-to-end** in ABP Framework v10.4 (layered template). Proceed across layers in the correct order.

## Trigger

- "ABP add new entity"
- "ABP add feature"
- "ABP end-to-end CRUD"
- "ABP development flow"
- "ABP how do I start"

## Step by Step (Layered Template)

### 1. Domain — Entity (`*.Domain/Entities/`)

```csharp
public class Book : AggregateRoot<Guid>
{
    public string Name { get; private set; }
    public decimal Price { get; private set; }
    public Guid AuthorId { get; private set; }

    protected Book() { }
    public Book(Guid id, string name, decimal price, Guid authorId) : base(id)
    {
        Name = Check.NotNullOrWhiteSpace(name, nameof(name));
        SetPrice(price);
        AuthorId = authorId;
    }

    public void SetPrice(decimal price) => Price = Check.Range(price, nameof(price), 0, 9999);
}
```

### 2. Domain.Shared — Constant/Enum (`*.Domain.Shared/`)

```csharp
public static class BookConsts { public const int MaxNameLength = 128; }
public enum BookType { Novel, Science, Biography }
```

### 3. Repository Interface (optional — only if a custom query is needed, `*.Domain/`)

```csharp
public interface IBookRepository : IRepository<Book, Guid>
{
    Task<Book> FindByNameAsync(string name);
}
```
For simple CRUD, `IRepository<Book, Guid>` is sufficient.

### 4. EF Core Configuration (`*.EntityFrameworkCore/`)

```csharp
// DbContext
public DbSet<Book> Books { get; set; }

// OnModelCreating
builder.Entity<Book>(b =>
{
    b.ToTable(MyProjectConsts.DbTablePrefix + "Books", MyProjectConsts.DbSchema);
    b.ConfigureByConvention();
    b.Property(x => x.Name).IsRequired().HasMaxLength(BookConsts.MaxNameLength);
    b.HasIndex(x => x.Name);
});
```

### 5. Migration

```bash
cd src/MyProject.EntityFrameworkCore
dotnet ef migrations add Added_Book
dotnet run --project ../MyProject.DbMigrator   # Recommended — also runs seed data
```

### 6. Application.Contracts — DTO + interface

```csharp
public class BookDto : EntityDto<Guid> { public string Name { get; set; } public decimal Price { get; set; } }

public class CreateBookDto
{
    [Required, StringLength(BookConsts.MaxNameLength)] public string Name { get; set; }
    [Range(0, 9999)] public decimal Price { get; set; }
    [Required] public Guid AuthorId { get; set; }
}

public interface IBookAppService : IApplicationService
{
    Task<BookDto> GetAsync(Guid id);
    Task<PagedResultDto<BookDto>> GetListAsync(PagedAndSortedResultRequestDto input);
    Task<BookDto> CreateAsync(CreateBookDto input);
}
```

### 7. Object Mapping (Mapperly — v10.4 default)

```csharp
[Mapper]
public partial class BookMapper : MapperBase<Book, BookDto>
{
    public override partial BookDto Map(Book source);
    public override partial void Map(Book source, BookDto destination);
}
```
Detail: [Object Mapping skill](../abp-object-mapping/SKILL.md).

### 8. Application — Service

```csharp
public class BookAppService : ApplicationService, IBookAppService
{
    private readonly IRepository<Book, Guid> _bookRepository; // or IBookRepository

    public BookAppService(IRepository<Book, Guid> bookRepository) => _bookRepository = bookRepository;

    public async Task<BookDto> GetAsync(Guid id)
        => ObjectMapper.Map<Book, BookDto>(await _bookRepository.GetAsync(id));

    [Authorize(MyProjectPermissions.Books.Create)]
    public async Task<BookDto> CreateAsync(CreateBookDto input)
    {
        var book = new Book(GuidGenerator.Create(), input.Name, input.Price, input.AuthorId);
        await _bookRepository.InsertAsync(book);
        return ObjectMapper.Map<Book, BookDto>(book);
    }
}
```

### 9. Localization (`*.Domain.Shared/Localization/*/en.json`)

```json
{ "Book": "Book", "Books": "Books", "BookName": "Name", "BookPrice": "Price" }
```

### 10. Permission (if needed, `*.Application.Contracts/`)

```csharp
public static class MyProjectPermissions
{
    public static class Books
    {
        public const string Default = "MyProject.Books";
        public const string Create = Default + ".Create";
    }
}
```

### 11. Test

```csharp
public class BookAppService_Tests : MyProjectApplicationTestBase
{
    private readonly IBookAppService _bookAppService;
    public BookAppService_Tests() => _bookAppService = GetRequiredService<IBookAppService>();

    [Fact]
    public async Task Should_Create_Book()
    {
        var result = await _bookAppService.CreateAsync(new CreateBookDto { Name = "Test", Price = 19.99m });
        result.Id.ShouldNotBe(Guid.Empty);
    }
}
```
Detail: [Testing skill](../abp-testing/SKILL.md).

## Quick Commands

```bash
dotnet build
cd src/MyProject.EntityFrameworkCore && dotnet ef migrations add MigrationName
dotnet run --project ../MyProject.DbMigrator   # apply migration + seed
abp generate-proxy -t ng                       # Angular proxy
```

## Checklist (New Feature)

- [ ] Entity (constructor + invariant)
- [ ] Domain.Shared constant/enum
- [ ] Custom repo interface (only if needed)
- [ ] EF Core config + `ConfigureByConvention()`
- [ ] Migration + DbMigrator
- [ ] DTO + service interface (Application.Contracts)
- [ ] Mapperly mapper
- [ ] Service implementation + authorization
- [ ] Localization keys
- [ ] Permission (if needed)
- [ ] Test

## Related

- [DDD](../abp-ddd/SKILL.md) — entity/aggregate/service design
- [EF Core](../abp-efcore/SKILL.md) — DbContext, migration
- [Object Mapping](../abp-object-mapping/SKILL.md) · [Authorization](../abp-authorization/SKILL.md) · [Localization](../abp-localization/SKILL.md) · [Testing](../abp-testing/SKILL.md)
- [Dependency Rules](../abp-dependency-rules/SKILL.md) — which code in which layer
- ABP Docs: https://abp.io/docs/latest/tutorials

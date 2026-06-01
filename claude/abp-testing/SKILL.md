---
name: abp-testing
description: "ABP Framework v10.4 testing: integration tests, *TestBase classes (Domain/Application/EntityFrameworkCore), SQLite in-memory, Shouldly, NSubstitute, data seeding, CurrentUser/CurrentTenant.Change, AddAlwaysAllowAuthorization. Use when you need to write unit/integration tests in ABP."
---

# ABP Framework — Testing

ABP Framework v10.4 testing guide. ABP prefers **integration tests** over unit tests: they run with real services + a real (SQLite in-memory) database, and internal services are not mocked.

## Trigger

- "write ABP test"
- "ABP integration test"
- "ABP unit test"
- "ABP TestBase"
- "ABP test data seed"
- "ABP authorization test"
- "ABP Shouldly / NSubstitute"

## Test Projects and Base Classes

| Project | Scope | Base Class |
|---|---|---|
| `*.Domain.Tests` | Domain logic, entity, domain service | `*DomainTestBase` |
| `*.Application.Tests` | Application services | `*ApplicationTestBase` |
| `*.EntityFrameworkCore.Tests` | Repository implementations | `*EntityFrameworkCoreTestBase` |

Each test gets a fresh database instance. Services are resolved via `GetRequiredService<T>()`.

## Application Service Test

```csharp
public class BookAppService_Tests : MyProjectApplicationTestBase
{
    private readonly IBookAppService _bookAppService;

    public BookAppService_Tests()
    {
        _bookAppService = GetRequiredService<IBookAppService>();
    }

    [Fact]
    public async Task Should_Create_Book()
    {
        // Arrange
        var input = new CreateBookDto { Name = "New Book", Price = 19.99m };

        // Act
        var result = await _bookAppService.CreateAsync(input);

        // Assert
        result.Id.ShouldNotBe(Guid.Empty);
        result.Name.ShouldBe("New Book");
    }

    [Fact]
    public async Task Should_Not_Create_Book_With_Invalid_Name()
    {
        var input = new CreateBookDto { Name = "", Price = 10m };
        await Should.ThrowAsync<AbpValidationException>(async () =>
        {
            await _bookAppService.CreateAsync(input);
        });
    }
}
```

## Domain Service Test

```csharp
public class BookManager_Tests : MyProjectDomainTestBase
{
    private readonly BookManager _bookManager;

    public BookManager_Tests()
    {
        _bookManager = GetRequiredService<BookManager>();
    }

    [Fact]
    public async Task Should_Not_Allow_Duplicate_Book_Name()
    {
        await _bookManager.CreateAsync("Existing Book", 10m);

        var exception = await Should.ThrowAsync<BusinessException>(async () =>
        {
            await _bookManager.CreateAsync("Existing Book", 20m);
        });

        exception.Code.ShouldBe("MyProject:BookNameAlreadyExists");
    }
}
```

## Naming & AAA

```csharp
// Pattern: Should_ExpectedBehavior_When_Condition
public async Task Should_Throw_BusinessException_When_Name_Already_Exists() { }

[Fact]
public async Task Should_Update_Book_Price()
{
    // Arrange
    var bookId = await CreateTestBookAsync();
    // Act
    var result = await _bookAppService.UpdateAsync(bookId, new UpdateBookDto { Price = 39.99m });
    // Assert
    result.Price.ShouldBe(39.99m);
}
```

## Assertions (Shouldly)

ABP uses the Shouldly library:

```csharp
result.ShouldNotBeNull();
result.Name.ShouldBe("Expected");
result.Price.ShouldBeGreaterThan(0);
result.Items.ShouldContain(x => x.Id == expectedId);
result.Items.ShouldBeEmpty();

// Exception
var ex = await Should.ThrowAsync<BusinessException>(async () => await _service.DoAsync());
ex.Code.ShouldBe("MyProject:ErrorCode");
```

## Test Data Seeding

```csharp
public class MyProjectTestDataSeedContributor : IDataSeedContributor, ITransientDependency
{
    public static readonly Guid TestBookId = Guid.Parse("....");
    private readonly IBookRepository _bookRepository;

    public MyProjectTestDataSeedContributor(IBookRepository bookRepository)
        => _bookRepository = bookRepository;

    public async Task SeedAsync(DataSeedContext context)
    {
        await _bookRepository.InsertAsync(
            new Book(TestBookId, "Test Book", 19.99m, Guid.Empty), autoSave: true);
    }
}
```

## Disabling Authorization in Tests

```csharp
public override void ConfigureServices(ServiceConfigurationContext context)
{
    context.Services.AddAlwaysAllowAuthorization();
}
```

## Mocking External Services (NSubstitute)

Don't mock internal ABP services — only external dependencies:

```csharp
public override void ConfigureServices(ServiceConfigurationContext context)
{
    var emailSender = Substitute.For<IEmailSender>();
    emailSender.SendAsync(Arg.Any<string>(), Arg.Any<string>(), Arg.Any<string>())
        .Returns(Task.CompletedTask);
    context.Services.AddSingleton(emailSender);
}
```

## Testing with a Specific User / Tenant

```csharp
// User
using (CurrentUser.Change(TestData.UserId))
{
    var result = await _bookAppService.GetMyBooksAsync();
    result.Items.ShouldAllBe(b => b.CreatorId == TestData.UserId);
}

// Tenant
using (CurrentTenant.Change(TestData.TenantId))
{
    var result = await _bookAppService.GetListAsync(new GetBookListDto());
    // Results are filtered by tenant
}
```

## Best Practices

1. **Prefer integration tests** — real services + SQLite in-memory, don't mock internal services
2. **Keep each test independent** — don't share state between tests
3. **Meaningful test data** — use a seed contributor for shared data
4. **Test edge cases and error conditions** — verify the exception `Code`
5. **Focus on a single behavior** — `Should_X_When_Y` naming
6. **Don't test the framework internals**

## Related

- [DDD](../abp-ddd/SKILL.md) — entity/domain/application service design
- [Authorization](../abp-authorization/SKILL.md) — AddAlwaysAllowAuthorization, CurrentUser
- [Multi-Tenancy](../abp-multitenancy/SKILL.md) — CurrentTenant.Change
- [Development Flow](../abp-development-flow/SKILL.md) — the final step of the feature flow: testing
- ABP Docs: https://abp.io/docs/latest/testing

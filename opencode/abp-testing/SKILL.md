---
name: abp-testing
description: "ABP Framework v10.4 testing quick reference: integration tests, *TestBase, SQLite in-memory, Shouldly, NSubstitute, data seeding, CurrentUser/CurrentTenant.Change. Use when you need to write tests in ABP."
---

# ABP Framework — Testing

ABP v10.4 testing. **Integration tests** are preferred: real services + SQLite in-memory DB, internal services not mocked.

## Trigger

"ABP test", "integration test", "TestBase", "test data seed", "Shouldly", "NSubstitute".

## Base Classes

| Project | Base Class |
|---|---|
| `*.Domain.Tests` | `*DomainTestBase` |
| `*.Application.Tests` | `*ApplicationTestBase` |
| `*.EntityFrameworkCore.Tests` | `*EntityFrameworkCoreTestBase` |

## Test (AAA + Shouldly)

```csharp
public class BookAppService_Tests : MyProjectApplicationTestBase
{
    private readonly IBookAppService _svc;
    public BookAppService_Tests() => _svc = GetRequiredService<IBookAppService>();

    [Fact]
    public async Task Should_Create_Book()
    {
        var result = await _svc.CreateAsync(new CreateBookDto { Name = "New", Price = 19.99m });
        result.Id.ShouldNotBe(Guid.Empty);
        result.Name.ShouldBe("New");
    }

    [Fact]
    public async Task Should_Throw_When_Invalid()
        => await Should.ThrowAsync<AbpValidationException>(async () =>
               await _svc.CreateAsync(new CreateBookDto { Name = "" }));
}
```

Naming: `Should_ExpectedBehavior_When_Condition`. Exception: `ex.Code.ShouldBe("MyProject:ErrorCode")`.

## Data Seed

```csharp
public class TestDataSeedContributor : IDataSeedContributor, ITransientDependency
{
    public async Task SeedAsync(DataSeedContext context)
        => await _bookRepository.InsertAsync(new Book(TestBookId, "Test", 19.99m), autoSave: true);
}
```

## Disable Authorization / Mock

```csharp
context.Services.AddAlwaysAllowAuthorization();           // disable permissions

var emailSender = Substitute.For<IEmailSender>();          // NSubstitute (external only)
context.Services.AddSingleton(emailSender);
```

## User / Tenant

```csharp
using (CurrentUser.Change(TestData.UserId)) { /* ... */ }
using (CurrentTenant.Change(TestData.TenantId)) { /* ... */ }
```

## Best Practices

1. Integration tests + SQLite in-memory, don't mock internal services
2. Keep each test independent, use meaningful seed data
3. Cover edge cases + verify exception `Code`

## Related

- [DDD](../abp-ddd/SKILL.md) · [Authorization](../abp-authorization/SKILL.md) · [Multi-Tenancy](../abp-multitenancy/SKILL.md) · [Development Flow](../abp-development-flow/SKILL.md)
- ABP Docs: https://abp.io/docs/latest/testing

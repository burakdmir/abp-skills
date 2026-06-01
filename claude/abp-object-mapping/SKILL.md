---
name: abp-object-mapping
description: "ABP Framework v10.4 object mapping: IObjectMapper, Mapperly (v10.4 default, MapperBase), AutoMapper profiles, entity↔DTO conversion, AutoMap attributes. Use when you need object mapping or DTO mapping in ABP."
---

# ABP Framework — Object Mapping

ABP Framework v10.4 object mapping guide. Entity↔DTO conversion via the `IObjectMapper` abstraction. In v10.4, **Mapperly** (compile-time, source-generated) is the default provider; **AutoMapper** is also supported. Stick with whichever one the solution already uses.

## Trigger

- "ABP object mapping"
- "ABP entity to DTO mapping"
- "ABP DTO mapping"
- "ABP Mapperly"
- "ABP AutoMapper"
- "ABP IObjectMapper"

## The IObjectMapper Abstraction

ABP base classes (`ApplicationService`, etc.) come with an `ObjectMapper` property out of the box:

```csharp
public class BookAppService : ApplicationService
{
    public async Task<BookDto> GetAsync(Guid id)
    {
        var book = await _bookRepository.GetAsync(id);
        return ObjectMapper.Map<Book, BookDto>(book);            // single object
    }

    public async Task<List<BookDto>> GetAllAsync()
    {
        var books = await _bookRepository.GetListAsync();
        return ObjectMapper.Map<List<Book>, List<BookDto>>(books); // list
    }

    // Map onto an existing object (source, destination)
    public void Update(Book book, UpdateBookDto input)
        => ObjectMapper.Map(input, book);
}
```

In places without a base class, inject `IObjectMapper`.

## Mapperly (v10.4 Default)

A compile-time source generator — no reflection, fast. ABP integration uses `MapperBase<TSource, TDestination>`:

```csharp
[Mapper]
public partial class BookToBookDtoMapper : MapperBase<Book, BookDto>
{
    public override partial BookDto Map(Book source);
    public override partial void Map(Book source, BookDto destination);
}

[Mapper]
public partial class CreateUpdateBookDtoToBookMapper : MapperBase<CreateUpdateBookDto, Book>
{
    public override partial Book Map(CreateUpdateBookDto source);
    public override partial void Map(CreateUpdateBookDto source, Book destination);
}
```

Module configuration:

```csharp
[DependsOn(typeof(AbpMapperlyModule))]
public class MyApplicationModule : AbpModule
{
    public override void ConfigureServices(ServiceConfigurationContext context)
    {
        context.Services.AddMapperlyObjectMapper<MyApplicationModule>();
    }
}
```

Since the mappers derive from `MapperBase`, an `ObjectMapper.Map<Book, BookDto>(...)` call uses them automatically.

### Custom Field Mapping (Mapperly)

```csharp
[Mapper]
public partial class BookMapper : MapperBase<Book, BookDto>
{
    [MapProperty(nameof(Book.Name), nameof(BookDto.Title))]   // different name
    public override partial BookDto Map(Book source);
    public override partial void Map(Book source, BookDto destination);
}
```

## AutoMapper (Alternative)

If the solution uses AutoMapper, define a profile:

```csharp
public class MyApplicationAutoMapperProfile : Profile
{
    public MyApplicationAutoMapperProfile()
    {
        CreateMap<Book, BookDto>();
        CreateMap<CreateUpdateBookDto, Book>();
    }
}
```

Module configuration:

```csharp
[DependsOn(typeof(AbpAutoMapperModule))]
public class MyApplicationModule : AbpModule
{
    public override void ConfigureServices(ServiceConfigurationContext context)
    {
        context.Services.AddAutoMapperObjectMapper<MyApplicationModule>();
        Configure<AbpAutoMapperOptions>(options =>
        {
            options.AddMaps<MyApplicationModule>(validate: true);
        });
    }
}
```

### Attribute-Based Mapping (AutoMapper)

On the DTO:

```csharp
[AutoMapFrom(typeof(Book))]      // Book → BookDto
public class BookDto : EntityDto<Guid>
{
    public string Name { get; set; }
}

[AutoMapTo(typeof(Book))]        // CreateBookDto → Book
public class CreateBookDto { public string Name { get; set; } }
```

## Best Practices

1. **Use the `ObjectMapper` abstraction** — don't depend directly on the provider (Mapperly/AutoMapper)
2. **Stick with the solution's provider** — for new projects, prefer Mapperly (the v10.4 default)
3. **Keep mappings in the Application layer** — entity↔DTO conversion is an application responsibility
4. **Use `validate: true` with AutoMapper** — catch unmapped fields early
5. **Map entity to DTO, do the reverse carefully** — on create/update, only map allowed fields

## Related

- [DDD](../abp-ddd/SKILL.md) — DTO design, application service
- [API](../abp-api/SKILL.md) — how DTOs are reflected in proxies
- [Development Flow](../abp-development-flow/SKILL.md) — where the mapper fits in the end-to-end flow
- ABP Docs: https://abp.io/docs/latest/framework/infrastructure/object-to-object-mapping

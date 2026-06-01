---
name: abp-object-mapping
description: "ABP Framework v10.4 object mapping quick reference: IObjectMapper, Mapperly (v10.4 default), AutoMapper profile/attribute, entity↔DTO. Use when you need object mapping in ABP."
---

# ABP Framework — Object Mapping

ABP v10.4 object mapping. The `IObjectMapper` abstraction; **Mapperly** is the default (compile-time), AutoMapper is also supported.

## Trigger

"ABP object mapping", "DTO mapping", "Mapperly", "AutoMapper", "IObjectMapper".

## Usage (ObjectMapper)

```csharp
ObjectMapper.Map<Book, BookDto>(book);
ObjectMapper.Map<List<Book>, List<BookDto>>(books);
ObjectMapper.Map(input, book);    // onto an existing destination
```
`ObjectMapper` is available in base classes; elsewhere, inject `IObjectMapper`.

## Mapperly (default)

```csharp
[Mapper]
public partial class BookMapper : MapperBase<Book, BookDto>
{
    public override partial BookDto Map(Book source);
    public override partial void Map(Book source, BookDto destination);
}
// Module: [DependsOn(typeof(AbpMapperlyModule))] + context.Services.AddMapperlyObjectMapper<MyModule>();
// Custom field: [MapProperty(nameof(Book.Name), nameof(BookDto.Title))]
```

## AutoMapper (alternative)

```csharp
public class MyProfile : Profile
{
    public MyProfile() { CreateMap<Book, BookDto>(); CreateMap<CreateUpdateBookDto, Book>(); }
}
// Module: [DependsOn(typeof(AbpAutoMapperModule))]
// context.Services.AddAutoMapperObjectMapper<MyModule>();
// Configure<AbpAutoMapperOptions>(o => o.AddMaps<MyModule>(validate: true));

// Attribute:
[AutoMapFrom(typeof(Book))] public class BookDto : EntityDto<Guid> { }
[AutoMapTo(typeof(Book))] public class CreateBookDto { }
```

## Best Practices

1. Use the `ObjectMapper` abstraction (don't depend on the provider)
2. Mapperly for new projects (the v10.4 default)
3. Keep mappings in the Application layer; use `validate: true` with AutoMapper
4. On create/update, only map allowed fields

## Related

- [DDD](../abp-ddd/SKILL.md) · [API](../abp-api/SKILL.md) · [Development Flow](../abp-development-flow/SKILL.md)
- ABP Docs: https://abp.io/docs/latest/framework/infrastructure/object-to-object-mapping

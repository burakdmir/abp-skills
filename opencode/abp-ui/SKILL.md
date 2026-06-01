---
name: abp-ui
description: "ABP Framework v10.4 UI: MVC/Razor Pages (AbpPageModel), Blazor (AbpComponentBase), Angular, React (--modern), theming (LeptonX), menu contributor, dynamic proxy. Use when developing frontend, UI, pages, or components in ABP."
---

# ABP Framework — UI & Frontend

ABP Framework v10.4 UI framework integrations. MVC, Blazor, Angular, React, theming.

## Trigger

- "ABP UI"
- "ABP MVC"
- "ABP Blazor"
- "ABP Angular"
- "ABP React"
- "ABP theme"

## UI Frameworks

| Framework | Type | Note |
|---|---|---|
| MVC/Razor Pages | Server-side | Default |
| Blazor Web App | Server-side (.NET 10) | Modern Blazor |
| Blazor WASM | Client-side | SPA |
| Angular | Client-side | TypeScript |
| React | Client-side | Modern template |

## MVC Controller

```csharp
[Area("App")]
[Route("api/app/book")]
public class BookController : AbpController
{
    private readonly IBookAppService _bookAppService;
    public BookController(IBookAppService bookAppService) => _bookAppService = bookAppService;
    
    [HttpGet]
    public Task<PagedResultDto<BookDto>> GetListAsync(PagedAndSortedResultRequestDto input) =>
        _bookAppService.GetListAsync(input);
}
```

## Razor Page

```csharp
public class IndexModel : AbpPageModel
{
    public List<BookDto> Books { get; set; }
    public IndexModel(IBookAppService bookAppService) => _bookAppService = bookAppService;
    
    public async Task OnGetAsync()
    {
        var result = await _bookAppService.GetListAsync(new());
        Books = result.Items;
    }
}
```

## Blazor Component

```razor
@page "/books"
@inject IBookAppService BookAppService

@if (books == null) { <p>Loading...</p> }
else {
    @foreach (var book in books.Items)
    {
        <div>@book.Name - @book.Price</div>
    }
}

@code {
    private PagedResultDto<BookDto> books;
    protected override async Task OnInitializedAsync() =>
        books = await BookAppService.GetListAsync(new());
}
```

## React (Modern)

```bash
abp new Acme.BookStore --template app --modern
abp new Acme.BookStore --modern --shadcn-theme blue
```

Theme: `slate`, `pink`, `blue`, `turquoise`, `orange`, `purple`

## Theming

```bash
abp new Acme.BookStore --theme leptonx-lite
abp new Acme.BookStore --theme basic
```

## Permission

```csharp
public static class BookStorePermissions
{
    public const string GroupName = "BookStore";
    public const string Books = GroupName + ".Books";
    public const string BooksCreate = Books + ".Create";
}
```

## Best Practices

1. MVC/Razor Pages as the default choice
2. Use Blazor Web App (.NET 10)
3. Use the `--modern` flag for React
4. Define permissions in a constant class

## Related

[API](../abp-api/SKILL.md) · [Authorization](../abp-authorization/SKILL.md) · [Localization](../abp-localization/SKILL.md) · [Framework](../abp-framework/SKILL.md) · Docs: https://abp.io/docs/latest/framework/ui

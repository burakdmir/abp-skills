# ABP Framework — UI & Frontend

ABP Framework v10.4 UI framework entegrasyonları. MVC, Blazor, Angular, React, theming.

## Trigger

- "ABP UI"
- "ABP MVC"
- "ABP Blazor"
- "ABP Angular"
- "ABP React"
- "ABP theme"

## UI Framework'leri

| Framework | Tip | Not |
|---|---|---|
| MVC/Razor Pages | Server-side | Varsayılan |
| Blazor Web App | Server-side (.NET 8+) | Modern Blazor |
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

1. MVC/Razor Pages varsayılan tercih
2. Blazor Web App (.NET 8+) kullan
3. React için `--modern` flag
4. Permission'ları sabit class'ta tanımla

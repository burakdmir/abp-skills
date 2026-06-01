---
name: abp-ui
description: "ABP Framework v10.4 UI: MVC/Razor Pages (AbpPageModel), Blazor (AbpComponentBase), Angular, React (--modern), theming (LeptonX), menu contributor, dynamic proxy. Use when developing frontend, UI, pages, or components in ABP."
---

# ABP Framework — UI & Frontend

ABP Framework v10.4 UI framework integrations. MVC/Razor Pages, Blazor, Angular, React, and UI theming.

## Trigger

- "ABP UI"
- "ABP MVC"
- "ABP Blazor"
- "ABP Angular"
- "ABP React"
- "ABP theme"
- "ABP Razor Pages"
- "ABP frontend"

## UI Framework Options

| Framework | Type | Note |
|---|---|---|
| **MVC/Razor Pages** | Server-side | Default, most mature |
| **Blazor Web App** | Server-side (.NET 10) | Modern Blazor |
| **Blazor WASM** | Client-side | SPA style |
| **Blazor Server** | Server-side | With SignalR |
| **Angular** | Client-side | TypeScript SPA |
| **React** | Client-side | With the modern template |
| **MAUI Blazor** | Cross-platform mobile | Team+ license |

---

## MVC / Razor Pages

### Module Registration

```csharp
[DependsOn(typeof(AbpAspNetCoreMvcModule))]
public class MyWebModule : AbpModule
{
    public override void ConfigureServices(ServiceConfigurationContext context)
    {
        context.Services.AddRazorPages();
    }
    
    public override void OnApplicationInitialization(ApplicationInitializationContext context)
    {
        var app = context.GetApplicationBuilder();
        app.UseStaticFiles();
        app.UseRouting();
        app.UseAuthentication();
        app.UseAuthorization();
        app.UseConfiguredEndpoints(endpoints => endpoints.MapRazorPages());
    }
}
```

### ABP Controller

```csharp
[Area("App")]
[Route("api/app/book")]
public class BookController : AbpController
{
    private readonly IBookAppService _bookAppService;
    
    public BookController(IBookAppService bookAppService)
    {
        _bookAppService = bookAppService;
    }
    
    [HttpGet]
    public Task<PagedResultDto<BookDto>> GetListAsync(PagedAndSortedResultRequestDto input)
    {
        return _bookAppService.GetListAsync(input);
    }
}
```

### Razor Page

```csharp
public class IndexModel : AbpPageModel
{
    private readonly IBookAppService _bookAppService;
    
    public List<BookDto> Books { get; set; }
    
    public IndexModel(IBookAppService bookAppService)
    {
        _bookAppService = bookAppService;
    }
    
    public async Task OnGetAsync()
    {
        var result = await _bookAppService.GetListAsync(new PagedAndSortedResultRequestDto());
        Books = result.Items;
    }
}
```

---

## Blazor

### Blazor Web App Module

```csharp
[DependsOn(
    typeof(AbpAspNetCoreComponentsWebAssemblyModule),
    typeof(AbpAutoMapperModule)
)]
public class MyBlazorModule : AbpModule { }
```

### Blazor Component

```razor
@page "/books"
@inject IBookAppService BookAppService

<h3>Books</h3>

@if (books == null)
{
    <p>Loading...</p>
}
else
{
    <table class="table">
        <thead>
            <tr><th>Name</th><th>Type</th><th>Price</th></tr>
        </thead>
        <tbody>
            @foreach (var book in books.Items)
            {
                <tr>
                    <td>@book.Name</td>
                    <td>@book.Type</td>
                    <td>@book.Price</td>
                </tr>
            }
        </tbody>
    </table>
}

@code {
    private PagedResultDto<BookDto> books;

    protected override async Task OnInitializedAsync()
    {
        books = await BookAppService.GetListAsync(new PagedAndSortedResultRequestDto());
    }
}
```

### Blazor Service Proxy

```csharp
// The dynamic C# HTTP proxy is generated automatically
// Defined in the HttpApi.Client project
```

---

## Angular

### Service Proxy

```bash
abp generate-proxy -t ng
```

The automatically generated services are created in the `src/app/proxy/` directory.

### Using a Component

```typescript
import { BookService } from '@proxy/books';

@Component({...})
export class BookListComponent {
  books: BookDto[] = [];
  
  constructor(private bookService: BookService) {}
  
  ngOnInit(): void {
    this.bookService.getList({}).subscribe(result => {
      this.books = result.items;
    });
  }
}
```

### Multi-Tenancy (Angular)

```typescript
// app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideAbpCore(
      withOptions({
        tenantKey: "__tenant",  // Default
      })
    ),
  ],
};
```

---

## React (Modern Template)

A React SPA is created with the modern template:

```bash
abp new Acme.BookStore --template app --modern
```

### Project Structure

```
Acme.BookStore/
├── react/                    # React SPA (user interface)
├── src/
│   └── ...                   # Backend projects
```

In the microservice template:

```
Acme.BookStore/
├── apps/react/               # React SPA
├── apps/react-admin-console/ # Admin Console (served by the backend)
├── apps/auth-server/         # OpenIddict Auth Server
├── gateways/web/             # YARP reverse proxy
└── gateways/mobile/          # Mobile gateway
```

### OpenIddict Client Seeding

The modern template seeds automatically:
- `MyProjectName_App` — React SPA
- `MyProjectName_AdminConsole` — Admin Console

### shadcn/ui Theme

```bash
abp new Acme.BookStore --modern --shadcn-theme blue
```

Theme values: `slate`, `pink`, `blue`, `turquoise`, `orange`, `purple`

---

## UI Theming

With the ABP UI theming system, you can use pre-built themes or custom themes.

### Pre-built Themes

| Theme | Description | License |
|---|---|---|
| `leptonx` | LeptonX (full) | Team+ |
| `leptonx-lite` | LeptonX Lite | Free |
| `basic` | Basic Theme | Free |

### Theme Selection

```bash
abp new Acme.BookStore --theme leptonx-lite
abp new Acme.BookStore --theme basic
```

### Basic Theme Installation (MVC)

```csharp
[DependsOn(
    typeof(AbpAspNetCoreMvcUiBasicThemeModule)
)]
public class MyWebModule : AbpModule { }
```

---

## Authorization UI

### Defining Permissions

```csharp
public static class BookStorePermissions
{
    public const string GroupName = "BookStore";
    
    public const string Books = GroupName + ".Books";
    public const string BooksCreate = Books + ".Create";
    public const string BooksUpdate = Books + ".Update";
    public const string BooksDelete = Books + ".Delete";
}
```

### Permission Provider

```csharp
public class BookStorePermissionDefinitionProvider : PermissionDefinitionProvider
{
    public override void Define(IPermissionDefinitionContext context)
    {
        var bookStoreGroup = context.AddGroup(BookStorePermissions.GroupName);
        
        var booksPermission = bookStoreGroup.AddPermission(BookStorePermissions.Books);
        booksPermission.AddChild(BookStorePermissions.BooksCreate);
        booksPermission.AddChild(BookStorePermissions.BooksUpdate);
        booksPermission.AddChild(BookStorePermissions.BooksDelete);
    }
}
```

### Checking Permissions in the UI

```razor
@* MVC/Razor *@
@if (await AuthorizationService.IsGrantedAsync(BookStorePermissions.BooksCreate))
{
    <button>Create Book</button>
}
```

```razor
@* Blazor *@
<Authorize Policy="BookStore.Books.Create">
    <button>Create Book</button>
</Authorize>
```

---

## Menu Contribution

```csharp
public class BookStoreMenuContributor : IMenuContributor
{
    public async Task ConfigureMenuAsync(MenuConfigurationContext context)
    {
        if (context.Menu.Name == StandardMenus.Main)
        {
            await ConfigureMainMenuAsync(context);
        }
    }

    private Task ConfigureMainMenuAsync(MenuConfigurationContext context)
    {
        context.Menu.AddItem(
            new ApplicationMenuItem(
                "BookStore",
                l["Menu:BookStore"],
                icon: "fa fa-book"
            ).AddItem(
                new ApplicationMenuItem(
                    "BookStore.Books",
                    l["Menu:Books"],
                    url: "/Books"
                )
            )
        );
        return Task.CompletedTask;
    }
}
```

---

## Best Practices

1. **MVC/Razor Pages as the default choice** — The most mature and best integrated
2. **Prefer Blazor Web App (.NET 10)** — Instead of Blazor Server/WASM
3. **Use the modern template for React** — With the `--modern` flag
4. **Use the dynamic C# proxy** — Avoid writing an HTTP client manually
5. **Define permissions in a constant class** — Don't use string literals
6. **Use the menu contributor** — Add menu items in a modular way
7. **Start with LeptonX-Lite** — Free, with sufficient features

---

## Related

- [API](../abp-api/SKILL.md) — dynamic proxy, Auto API Controllers
- [Authorization](../abp-authorization/SKILL.md) — checking permissions in the UI
- [Localization](../abp-localization/SKILL.md) — UI texts, L[] helper
- [Framework Core](../abp-framework/SKILL.md) — solution template UI options
- ABP Docs: https://abp.io/docs/latest/framework/ui

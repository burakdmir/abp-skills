# ABP Framework — UI & Frontend

ABP Framework v10.4 UI framework entegrasyonları. MVC/Razor Pages, Blazor, Angular, React ve UI theming.

## Trigger

- "ABP UI"
- "ABP MVC"
- "ABP Blazor"
- "ABP Angular"
- "ABP React"
- "ABP theme"
- "ABP Razor Pages"
- "ABP frontend"

## UI Framework Seçenekleri

| Framework | Tip | Not |
|---|---|---|
| **MVC/Razor Pages** | Server-side | Varsayılan, en olgun |
| **Blazor Web App** | Server-side (.NET 8+) | Modern Blazor |
| **Blazor WASM** | Client-side | SPA tarzı |
| **Blazor Server** | Server-side | SignalR ile |
| **Angular** | Client-side | TypeScript SPA |
| **React** | Client-side | Modern template ile |
| **MAUI Blazor** | Cross-platform mobile | Team+ lisans |

---

## MVC / Razor Pages

### Module Kaydı

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
// Dynamic C# HTTP proxy otomatik generate edilir
// HttpApi.Client projesinde tanımlanır
```

---

## Angular

### Service Proxy

```bash
abp generate-proxy -t ng
```

Otomatik generate edilen servisler `src/app/proxy/` dizininde oluşturulur.

### Component Kullanımı

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
        tenantKey: "__tenant",  // Varsayılan
      })
    ),
  ],
};
```

---

## React (Modern Template)

Modern template ile React SPA oluşturulur:

```bash
abp new Acme.BookStore --template app --modern
```

### Proje Yapısı

```
Acme.BookStore/
├── react/                    # React SPA (kullanıcı arayüzü)
├── src/
│   └── ...                   # Backend projeleri
```

Microservice template'te:

```
Acme.BookStore/
├── apps/react/               # React SPA
├── apps/react-admin-console/ # Admin Console (backend tarafından serve edilir)
├── apps/auth-server/         # OpenIddict Auth Server
├── gateways/web/             # YARP reverse proxy
└── gateways/mobile/          # Mobile gateway
```

### OpenIddict Client Seeding

Modern template otomatik seed eder:
- `MyProjectName_App` — React SPA
- `MyProjectName_AdminConsole` — Admin Console

### shadcn/ui Theme

```bash
abp new Acme.BookStore --modern --shadcn-theme blue
```

Theme değerleri: `slate`, `pink`, `blue`, `turquoise`, `orange`, `purple`

---

## UI Theming

ABP UI theming sistemi ile pre-built temalar veya custom temalar kullanılabilir.

### Pre-built Temalar

| Theme | Açıklama | Lisans |
|---|---|---|
| `leptonx` | LeptonX (full) | Team+ |
| `leptonx-lite` | LeptonX Lite | Ücretsiz |
| `basic` | Basic Theme | Ücretsiz |

### Theme Seçimi

```bash
abp new Acme.BookStore --theme leptonx-lite
abp new Acme.BookStore --theme basic
```

### Basic Theme Kurulumu (MVC)

```csharp
[DependsOn(
    typeof(AbpAspNetCoreMvcUiBasicThemeModule)
)]
public class MyWebModule : AbpModule { }
```

---

## Authorization UI

### Permission Tanımlama

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

### UI'da Permission Kontrolü

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

1. **MVC/Razor Pages varsayılan tercih** — En olgun ve en iyi entegre
2. **Blazor Web App (.NET 8+) tercih et** — Blazor Server/WASM yerine
3. **React için modern template kullan** — `--modern` flag ile
4. **Dynamic C# proxy kullan** — Manuel HTTP client yazma
5. **Permission'ları sabit class'ta tanımla** — String literal kullanma
6. **Menu contributor kullan** — Menü öğelerini modüler şekilde ekle
7. **LeptonX-Lite başla** — Ücretsiz, yeterli özellikler

# ABP Authorization Skill

## Trigger
User asks about permissions, authorization, roles, policies, permission groups, resource-based authorization, or access control in ABP Framework.

---

## Core Concepts

ABP extends ASP.NET Core Authorization with a **Permission System** that auto-registers permissions as policies. Two permission types exist:

1. **Standard (Global) Permissions** — Apply globally (e.g., "can create documents")
2. **Resource-Based Permissions** — Target specific instances (e.g., "can edit Document #123")

---

## Permission System

### Defining Permissions

Create a class inheriting `PermissionDefinitionProvider`:

```csharp
using Volo.Abp.Authorization.Permissions;

namespace Acme.BookStore.Permissions
{
    public class BookStorePermissionDefinitionProvider : PermissionDefinitionProvider
    {
        public override void Define(IPermissionDefinitionContext context)
        {
            var myGroup = context.AddGroup(
                "BookStore",
                LocalizableString.Create<BookStoreResource>("BookStore")
            );

            myGroup.AddPermission(
                "BookStore_Author_Create",
                LocalizableString.Create<BookStoreResource>("Permission:BookStore_Author_Create")
            );
        }
    }
}
```

- ABP auto-discovers this class — no registration needed
- Typically placed in `Application.Contracts` project
- Permission name becomes usable as an ASP.NET Core **policy** name

### Permission Groups

- `context.AddGroup("GroupName")` creates a new group
- Groups appear as tabs in the UI permission dialog
- Use `LocalizableString.Create<TResource>("Key")` for localized display names

### Permission Options

```csharp
myGroup.AddPermission(
    "BookStore_Books_Manage",
    LocalizableString.Create<BookStoreResource>("Permission:ManageBooks"),
    multiTenancySide: MultiTenancySides.Tenant  // Host, Tenant, or Both (default)
);
```

**Multi-tenancy side options:**
- `MultiTenancySides.Host` — Available only on host side
- `MultiTenancySides.Tenant` — Available only on tenant side
- `MultiTenancySides.Both` (default) — Available on both

### Child Permissions

Permissions can have parent-child relationships for hierarchical UI display:

```csharp
var booksPermission = myGroup.AddPermission("BookStore_Books_Manage");
booksPermission.AddChild("BookStore_Books_Create");
booksPermission.AddChild("BookStore_Books_Edit");
booksPermission.AddChild("BookStore_Books_Delete");
```

### Enable/Disable Permissions

```csharp
myGroup.AddPermission("BookStore_Feature", isEnabled: false); // Disabled by default
```

### Permission Depending on a Condition

```csharp
// Depending on Features
myGroup.AddPermission("BookStore_Advanced")
    .WithFeatureDependency("AdvancedFeature");

// Depending on Global Features
myGroup.AddPermission("BookStore_Preview")
    .WithGlobalFeatureDependency("PreviewModule");

// Custom condition
myGroup.AddPermission("BookStore_Special")
    .WithStateChecker(() => MyStaticChecker.IsSpecialEnabled);
```

### Overriding Permissions with Custom Policies

```csharp
myGroup.AddPermission("BookStore_Special")
    .WithPolicyDependency(new MyCustomPolicyRequirement());
```

### Changing Permission Definitions of Dependent Modules

```csharp
[DependsOn(typeof(AbpIdentityModule))]
public class MyModule : AbpModule
{
    public override void PreConfigureServices(ServiceConfigurationContext context)
    {
        PreConfigure<PermissionDefinitionContext>(options =>
        {
            // Modify identity module permissions before they're finalized
        });
    }
}
```

---

## Using Permissions

### In Controllers / Page Models

```csharp
[Authorize("BookStore_Author_Create")]
public async Task<IActionResult> CreateAsync(CreateAuthorDto input)
{
    // ...
}
```

### In Application Services

```csharp
public class AuthorAppService : ApplicationService, IAuthorAppService
{
    public async Task<AuthorDto> CreateAsync(CreateAuthorDto input)
    {
        await AuthorizationService.CheckAsync("BookStore_Author_Create");
        // or
        if (!await AuthorizationService.IsGrantedAsync("BookStore_Author_Create"))
        {
            throw new AbpAuthorizationException("...");
        }
        // ...
    }
}
```

### Using IAuthorizationService

```csharp
public class MyService : ITransientDependency
{
    private readonly IAuthorizationService _authorizationService;

    public MyService(IAuthorizationService authorizationService)
    {
        _authorizationService = authorizationService;
    }

    public async Task DoWorkAsync()
    {
        await _authorizationService.CheckAsync("BookStore_Author_Create");
    }
}
```

---

## Resource-Based Authorization

For fine-grained, per-instance permissions:

```csharp
// Define a resource permission
var booksGroup = context.AddGroup("BookStore_Books");
var bookPermission = booksGroup.AddResourcePermission(
    "BookStore_Books_Edit",
    typeof(Book),
    LocalizableString.Create<BookStoreResource>("Permission:EditBook")
);
```

Resource-based permissions are managed through the **Resource Permission Management Dialog** on individual resource instances (not the global permissions dialog).

See: [Resource-Based Authorization](resource-based-authorization.md)

---

## Multi-Tenancy Integration

- Permissions can be scoped to Host, Tenant, or Both
- When setting permissions for users/roles, use `ISettingManager` or the Identity module UI
- Tenant-specific permissions are stored per-tenant in the database

---

## UI Integration

- Permissions dialog available with Identity module pre-installed
- Permission groups shown as tabs
- Standard permissions shown in global dialog
- Resource-based permissions managed per-resource instance

---

## Best Practices

1. **Naming convention:** `ModuleName_Entity_Action` (e.g., `BookStore_Books_Create`)
2. **Always localize** permission display names using `LocalizableString.Create<TResource>()`
3. **Group logically** — one group per module or feature area
4. **Use child permissions** for hierarchical UI (Manage > Create/Edit/Delete)
5. **Set multi-tenancy side** explicitly if your app is multi-tenant
6. **Use resource-based permissions** for instance-level access control
7. **Check permissions early** in application service methods to fail fast

---

## Common Patterns

### CRUD Permissions

```csharp
var books = myGroup.AddPermission("BookStore_Books_Manage");
books.AddChild("BookStore_Books_Create");
books.AddChild("BookStore_Books_Edit");
books.AddChild("BookStore_Books_Delete");
books.AddChild("BookStore_Books_ViewList");
```

### Permission Check in App Service Base Class

```csharp
public abstract class BookStoreAppService : ApplicationService
{
    protected BookStoreAppService()
    {
        LocalizationResource = typeof(BookStoreResource);
    }

    protected virtual async Task CheckPolicyAsync(string policyName)
    {
        await AuthorizationService.CheckAsync(policyName);
    }
}
```

---

## Related Modules

- **Identity Module** — User and role management, permission UI
- **Permission Management Module** — Resource permission management dialog
- **Setting Management Module** — Feature-based permission toggling

# ABP Authorization Skill

## Trigger
Permissions, authorization, roles, policies, permission groups, resource-based auth, access control.

---

## Quick Reference

### Define Permissions
```csharp
public class BookStorePermissionDefinitionProvider : PermissionDefinitionProvider
{
    public override void Define(IPermissionDefinitionContext context)
    {
        var group = context.AddGroup("BookStore", LocalizableString.Create<BookStoreResource>("BookStore"));
        group.AddPermission("BookStore_Author_Create", LocalizableString.Create<BookStoreResource>("Permission:Create"));
    }
}
```
- Auto-discovered, place in `Application.Contracts`
- Permission name = ASP.NET Core policy name

### Use Permissions
```csharp
[Authorize("BookStore_Author_Create")]  // Controller/Page
await AuthorizationService.CheckAsync("BookStore_Author_Create");  // App Service
```

### Options
- `multiTenancySide`: Host | Tenant | Both (default)
- Child permissions: `parent.AddChild("Child_Name")`
- Feature dependency: `.WithFeatureDependency("FeatureName")`
- Disabled by default: `isEnabled: false`

### Resource-Based Auth
```csharp
group.AddResourcePermission("BookStore_Books_Edit", typeof(Book), ...);
```
Managed per-instance via Resource Permission Management Dialog.

### IAuthorizationService
```csharp
await _authService.CheckAsync("PermissionName");
await _authService.IsGrantedAsync("PermissionName");
```

### Best Practices
- Naming: `Module_Entity_Action`
- Always localize display names
- Group by module/feature
- Use child perms for CRUD hierarchy
- Set multi-tenancy side explicitly

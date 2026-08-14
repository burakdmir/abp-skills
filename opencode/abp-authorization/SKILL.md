---
name: abp-authorization
description: "ABP Framework v10.x (10.4–10.6) authorization: defining permissions (PermissionDefinitionProvider), [Authorize], CheckPolicyAsync/IsGrantedAsync, CurrentUser, IPermissionManager, resource-based auth, multi-tenancy permissions. Use when you need permission, role or access checks in ABP."
---

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

## v10.5+

- Identity token providers are single-active per user/purpose (`AbpDefaultTokenProvider`, 10-min default lifetime via `AbpDefaultTokenProviderOptions`/`AbpLinkUserTokenProviderOptions`). Re-test 2FA, password change, and link-user flows after upgrading.
- OpenIddict default-scope fallback (opt-in): `AbpOpenIddictAspNetCoreOptions.UseDefaultScopesForClientCredentials/Password/TokenExchange = true`.

## v10.6+

- Antiforgery user-id claim issuer normalization is on by default (`AbpAntiForgeryOptions.NormalizeUserIdClaimIssuer`) — re-test mixed SPA + MVC flows.
- `HttpContextAbpAccessTokenProvider` forwards the incoming access token for any authenticated request (incl. `client_credentials`).

## Dynamic Claims

- Overrides token/cookie claims with fresh values on each request (e.g. a revoked role takes effect without re-login). Enable: `AbpClaimsPrincipalFactoryOptions.IsDynamicClaimsEnabled = true` + `app.UseDynamicClaims()` before `UseAuthorization` (default-on in templates since v8.0).
- Tiered UI app: also set `options.RemoteRefreshUrl = authServerUrl + options.RemoteRefreshUrl`.
- Custom contributor: implement `IAbpDynamicClaimsPrincipalContributor` + register in DI (`ContributeAsync` runs every request — cache it). Built-ins: Identity (auth server), Remote (tiered UI), WebRemote (microservices, opt-in).

## Related

[Settings & Features](../abp-settings-features/SKILL.md) · [Multi-Tenancy](../abp-multitenancy/SKILL.md) · [UI](../abp-ui/SKILL.md) · [Exception Handling](../abp-exception-handling/SKILL.md) · Docs: https://abp.io/docs/latest/framework/fundamentals/authorization

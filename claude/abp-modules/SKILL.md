---
name: abp-modules
description: "ABP Framework v10.x (10.4–10.6) pre-built application modules: Identity, Account, OpenIddict, Tenant/Permission/Setting/Feature Management, Audit Logging, CMS Kit, SaaS, Payment and more — which module solves what, how to install it, free vs Pro. Use when choosing, installing, or extending a pre-built ABP module (abp add-module) in an ABP application."
---

# ABP Framework — Pre-Built Application Modules

A "which module solves what" reference for the pre-built application modules of ABP Framework v10.x (10.4–10.6). Covers the free open-source modules, the commercial (Pro) modules, installation with the ABP CLI, and how to extend or replace a module.

## Trigger

- "ABP modules"
- "ABP Identity module" / "ABP Account module"
- "abp add-module"
- "ABP CMS Kit" / "ABP SaaS module"
- "ABP free vs Pro modules"
- "install a pre-built ABP module"
- "extend an ABP module entity"

## Application Modules vs Framework Modules

ABP has two types of modules. They have no structural difference — both are `AbpModule`-based NuGet/NPM packages — but they differ in purpose:

| Type | What It Provides | Examples |
|---|---|---|
| **Framework module** | Infrastructure, integration, abstraction — no business functionality | Caching, emailing, theming, security, validation, EF Core / MongoDB integration |
| **Application module** | Complete application/business features with their own entities, services, APIs, and UI | Identity, Tenant Management, Blogging, CMS Kit, SaaS |

This skill covers **application modules**. For framework modules and the module system itself, see [Modularity](../abp-modularity/SKILL.md).

## Module Table — Free (Open Source, MIT)

| Module | Package / Module Name | Purpose |
|---|---|---|
| **Account** | `Volo.Abp.Account` (e.g. `Volo.Abp.Account.Web.OpenIddict`) | Login, register, forgot password, account management UI |
| **Identity** | `Volo.Abp.Identity` (e.g. `Volo.Abp.Identity.EntityFrameworkCore`) | Users, roles, organization units, and their permissions — based on Microsoft Identity |
| **OpenIddict** | OpenIddict module (integration packages: `Volo.Abp.Account.Web.OpenIddict`, `Volo.Abp.PermissionManagement.Domain.OpenIddict`) | SSO, single log-out, API access control; persists OpenIddict applications and scopes |
| **Permission Management** | `Volo.Abp.PermissionManagement` | Implements `IPermissionStore` — persists and manages permission values in the database |
| **Setting Management** | `Volo.Abp.SettingManagement` | Implements `ISettingStore` + `ISettingManager` — persists setting values |
| **Feature Management** | `Volo.Abp.FeatureManagement` | Implements `IFeatureStore` — persists feature values, provides management UI |
| **Tenant Management** | Tenant Management module | Implements `ITenantStore` — manages tenants for multi-tenant apps |
| **Audit Logging** | Audit Logging module (connection string name: `AbpAuditLogging`) | Implements `IAuditingStore` — persists audit logs to the database |
| **Background Jobs** | Background Jobs module (connection string name: `AbpBackgroundJobs`) | Implements `IBackgroundJobStore` for the default background job manager |
| **CMS Kit** | `Volo.CmsKit` | Reusable CMS building blocks and sub-systems for content-enabled websites |
| **Docs** | `Volo.Docs` | Technical documentation website (sources: GitHub or local file system) |
| **Blogging** | `Volo.Blogging` | Blogs, posts, tags, comments, member profiles — MVC/Razor Pages UI |

Also free: **IdentityServer** module (IdentityServer4 integration, legacy) and **Virtual File Explorer** (simple UI over the virtual file system).

## Module Table — Pro (Commercial License Required)

All Pro modules require an **ABP Team or higher license**. Mark of the Pro edition: richer UI, extra features on top of the free counterpart, or entirely commercial-only functionality.

| Module | Package / Module Name | Purpose |
|---|---|---|
| **SaaS (Pro)** | `Volo.Saas` | Tenants **and editions**: assign features to editions/tenants, per-tenant connection strings — the Pro replacement for Tenant Management |
| **File Management (Pro)** | `Volo.FileManagement` | Upload/download/organize files in hierarchical folders; built on BLOB Storing; per-tenant size limits |
| **Payment (Pro)** | `Volo.Payment` | Payment gateway integration: Stripe, PayPal, 2Checkout, PayU, Iyzico, Alipay; one-time payments (all) and subscriptions (Stripe) |
| **Chat (Pro)** | `Volo.Chat` (SignalR-based, `Volo.Chat.SignalR`) | Real-time messaging between application users |
| **GDPR (Pro)** | `Volo.Gdpr` (events via `Volo.Abp.Gdpr.Abstractions`) | Users request download or deletion of their personal data |
| **Language Management (Pro)** | `Volo.LanguageManagement` | Add/remove/enable languages and translate UI texts on the fly |
| **Text Template Management (Pro)** | `Volo.TextTemplateManagement` | UI for editing text-templating system templates (e.g. Account module emails) |
| **Operation Rate Limiting (Pro)** | `Volo.Abp.OperationRateLimiting`, `Volo.Abp.OperationRateLimiting.AspNetCore` | Application/domain-level rate limiting (SMS codes, login attempts, expensive reports) |
| **Forms (Pro)** | Forms module | Create forms and surveys |
| **Twilio SMS (Pro)** | Twilio SMS module | Send SMS messages via the Twilio cloud service |

Pro editions of free modules: **Account (Pro)** (social logins, email activation), **Identity (Pro)** (claims + advanced user/role management), **Audit Logging (Pro)** (reporting UI, entity history), **OpenIddict (Pro)** (UI to manage applications/scopes), **CMS Kit (Pro)** (extra CMS building blocks), **Identity Server (Pro)**.

## Which Module Solves What

| Need | Module |
|---|---|
| Users can log in / register / reset password | Account (free); social logins → Account Pro |
| Manage users, roles, permissions | Identity + Permission Management |
| OAuth2 / OpenID Connect server, SSO | OpenIddict |
| Multi-tenancy: manage tenants | Tenant Management (free) or SaaS Pro (adds editions + feature assignment) |
| Persist per-tenant/per-user settings | Setting Management |
| Feature flags per edition/tenant | Feature Management (+ SaaS Pro for edition-based) |
| Who changed what, when | Audit Logging; entity-history reporting UI → Audit Logging Pro |
| Reliable queued jobs | Background Jobs |
| Pages, comments, tags, ratings for a website | CMS Kit |
| Product docs site | Docs |
| Company blog | Blogging (free) or CMS Kit blogging features |
| File upload/browse UI | File Management (Pro) |
| Take payments / subscriptions | Payment (Pro) |
| In-app user-to-user messaging | Chat (Pro) |
| GDPR data export/erasure requests | GDPR (Pro) |
| Editable email/SMS templates | Text Template Management (Pro) |
| Throttle specific operations | Operation Rate Limiting (Pro) |

## Installing a Module

### Pre-installed modules

Identity, Account, OpenIddict, Permission/Setting/Feature/Tenant Management, Audit Logging, and Background Jobs **come pre-installed** (as NuGet/NPM packages) when you create a solution from a startup template. Keep them as packages to get updates easily, or pull their source into your solution:

```bash
# Include a module's source code in your solution for deep customization
abp get-source Volo.Abp.Identity
```

### Adding a module with the CLI

`abp add-module` installs **all packages of a module** to the correct projects of a layered solution (Domain, Application, HttpApi, EF Core, UI…) and wires up module dependencies and DB integration:

```bash
# Free modules
abp add-module Volo.Blogging
abp add-module Volo.Docs
abp add-module Volo.CmsKit --skip-db-migrations

# Pro modules (require ABP Team or higher license)
abp add-module Volo.FileManagement
abp add-module Volo.Payment
abp add-module Volo.Gdpr
abp add-module Volo.TextTemplateManagement
```

Use `--skip-db-migrations` when you want to add the EF Core migration yourself (CMS Kit's docs show this pattern — it requires enabling its global feature first).

### add-module vs add-package vs install-module

```bash
abp add-module Volo.Blogging          # whole module -> all solution layers
abp add-package Volo.Abp.Blogging     # single package -> current project only
abp install-module Volo.Blogging      # install a NuGet module
abp install-local-module ../Acme.Blogging  # install a module from local source
```

- **`add-module`** — the normal way to add a pre-built application module to a solution.
- **`add-package`** — adds one specific package to one project; use when you only need a single layer (e.g. only the `Domain.Shared` contract package).
- After adding a module with EF Core, add a migration and update the database (`dotnet ef migrations add Added_ModuleName` + run the `DbMigrator`), unless the CLI already did it.

## Module Notes

### Identity & Account

- Identity is based on the Microsoft Identity library; manages roles, users, organization units, and their permissions.
- Account provides the login/register/forgot-password UI and integrates with OpenIddict through `Volo.Abp.Account.Web.OpenIddict` (installed by the application startup template).

### Permission / Setting / Feature Management

These are the persistence backends of framework infrastructure: they implement `IPermissionStore`, `ISettingStore`, and `IFeatureStore` respectively. The framework defines the abstractions; these modules store the values in your database and provide management UIs. Blazor component namespaces exist per UI flavor (e.g. `Volo.Abp.PermissionManagement.Blazor.Components`, MudBlazor variants under `...Blazor.MudBlazor.Components`).

### Tenant Management vs SaaS (Pro)

- Tenant Management (free) implements `ITenantStore` and basic tenant CRUD.
- SaaS (Pro) replaces it in commercial templates: tenants **plus editions**, feature-to-edition assignment, tenant activation states, and tenant-based connection string management.

### Audit Logging & Background Jobs

- Both use the `Abp` table prefix by default; change via `AbpAuditLoggingDbProperties` / `AbpBackgroundJobsDbProperties`.
- Both use their own connection string name (`AbpAuditLogging`, `AbpBackgroundJobs`) with fallback to `Default` — this is what lets you move their tables to a separate database.

### CMS Kit

Provides core building blocks (comments, tags, ratings, menus, pages, blogging…) as individually enable-able sub-systems via the global feature system. Split into `Volo.CmsKit.Admin` (management side) and `Volo.CmsKit.Public` (public website side). Install with `--skip-db-migrations` and enable the sub-systems you need before creating the migration.

### Docs & Blogging

- Docs supports **Entity Framework Core and MongoDB**; it loads documentation from **GitHub or the local file system**, and custom document sources can be added. ABP's own documentation site runs on this module.
- Blogging provides an MVC / Razor Pages UI with EF Core and MongoDB integrations. Its upload service accepts JPEG, PNG, GIF, and BMP images, 5 MiB max by default; `BloggingWebConsts.FileUploading.MaxFileSize` is a process-wide static — set it once at startup, before the app accepts uploads.

### SaaS (Pro) details

- A tenant can have **one edition**; features are assigned to editions and tenants.
- Supports default and module-specific tenant **connection strings** (tenant-based connection string management).
- Commercial startup templates seed standard editions via `IEditionDataSeeder.CreateStandardEditionsAsync()` — layered apps run it from the `.DbMigrator`, no-layers apps from the host's migration service.

### Payment (Pro) details

- Gateways: Stripe, PayPal, 2Checkout, PayU, Iyzico, Alipay.
- All gateways support one-time payments; **only Stripe supports subscriptions/recurring payments**.
- Install with `abp add-module Volo.Payment`; per-gateway UI packages exist (e.g. `Volo.Payment.Iyzico.Blazor.Server`, `Volo.Payment.PayPal.Blazor.Server`).

### GDPR (Pro) details

- Users request a **download** of their personal data or **deletion** of data and account.
- Works over distributed events from the `Volo.Abp.Gdpr.Abstractions` package: participating modules collect their own data and publish prepared-data events; the GDPR module stores payloads and returns them as a ZIP archive.
- With Identity Pro installed, its built-in subscriber anonymizes personal fields, deactivates the user, and deletes the identity-user record; other modules stay responsible for their own data.

### Text Template Management & Operation Rate Limiting (Pro)

- Text Template Management edits templates of ABP's text templating system — for example, the Account module's password-reset email templates. Its `TextManagement.Enable` feature is on by default and gates the module's permissions, services, and menu items.
- Operation Rate Limiting throttles specific operations in application/domain code: SMS verification codes per phone number, resource-heavy reports per user, login attempts per IP. Packages: `Volo.Abp.OperationRateLimiting` and `Volo.Abp.OperationRateLimiting.AspNetCore`.

### File Management & Chat (Pro)

- File Management is built on the **BLOB Storing** system, so any storage provider works for file contents; it is multi-tenancy compatible with per-tenant total size limits. Install: `abp add-module Volo.FileManagement`.
- Chat implements real-time messaging over SignalR (`Volo.Chat.SignalR`); source can be downloaded with `abp get-source Volo.Chat` (ABP Suite also supports it).

## Module Package Anatomy

Every pre-built module ships as a set of layered NuGet packages following the DDD packaging pattern — the CLI's `add-module` maps each to the matching project of your solution:

```
Volo.Abp.Identity.Domain.Shared        -> *.Domain.Shared        (constants, enums, localization)
Volo.Abp.Identity.Domain               -> *.Domain               (entities, repository interfaces)
Volo.Abp.Identity.Application.Contracts-> *.Application.Contracts (DTOs, service interfaces)
Volo.Abp.Identity.Application          -> *.Application          (application services)
Volo.Abp.Identity.EntityFrameworkCore  -> *.EntityFrameworkCore  (DbContext integration)
Volo.Abp.Identity.MongoDB              -> *.MongoDB              (MongoDB integration)
Volo.Abp.Identity.HttpApi              -> *.HttpApi              (API controllers)
Volo.Abp.Identity.HttpApi.Client       -> *.HttpApi.Client       (dynamic C# client proxies)
```

UI packages come per flavor (MVC, Blazor, Blazor.MudBlazor, Angular NPM packages such as `@abp/ng.tenant-management`). Depend on `*.HttpApi.Client` when a service needs to call a module's API remotely.

## Module Databases

- Module tables/collections use the `Abp` prefix by default; override via each module's `*DbProperties` class (e.g. `AbpAuditLoggingDbProperties`, `AbpBackgroundJobsDbProperties`).
- Each module resolves its own connection string name (e.g. `AbpAuditLogging`, `AbpBackgroundJobs`) and falls back to `Default` — define the named connection string to move a module to its own database.

## Replacing / Extending a Module

Pre-built modules are designed to be used **as packages** — don't fork them for small changes:

1. **Extend entities without touching module source** (module entity extensions):

```csharp
ObjectExtensionManager.Instance
    .MapEfCoreProperty<IdentityUser, string>(
        "SocialSecurityNumber",
        (entityBuilder, propertyBuilder) =>
        {
            propertyBuilder.HasMaxLength(32);
        }
    );
```

2. **Replace a service** — register your own implementation over the module's service (standard ABP DI replacement, `[ExposeServices]` / `Dependency(ReplaceServices = true)`).
3. **Override a UI page/component** — startup templates support overriding module pages, components, and localization resources.
4. **Take the source** — `abp get-source <ModuleName>` includes the module's MIT-licensed source in your solution when package-level extension isn't enough (free modules only; Pro modules' source access depends on license).

See [Modularity](../abp-modularity/SKILL.md) for `ObjectExtensionManager` details and module development practices.

## Best Practices

1. **Prefer a pre-built module over writing your own** — Identity, tenant, settings, audit, and background-job persistence are solved problems in ABP.
2. **Keep modules as NuGet packages** — only `get-source` when you truly need to modify internals; packages upgrade with a version bump.
3. **Use `abp add-module`, not manual package references** — it targets every layer and wires DB integration correctly.
4. **Check free vs Pro before designing** — e.g. plan Tenant Management (free) vs SaaS editions (Pro) early; Pro modules need an ABP Team or higher license.
5. **Extend entities via `ObjectExtensionManager`**, not by forking the module.
6. **Separate module databases when needed** — use the module-specific connection string names (e.g. `AbpAuditLogging`) instead of everything on `Default`.
7. **Run migrations after adding a module** — every EF Core module brings its own tables; use the `DbMigrator` (layered) or host migration flow (no-layers).
8. **Don't confuse application modules with framework modules** — caching/validation/EF Core integration are framework modules and are documented separately.

---

## Related

- [Modularity](../abp-modularity/SKILL.md) — module system, creating/installing modules, entity extensions
- [Framework Core](../abp-framework/SKILL.md) — AbpModule, lifecycle, [DependsOn]
- [ABP CLI](../abp-cli/SKILL.md) — add-module, add-package, get-source commands
- ABP Docs: https://abp.io/docs/latest/modules

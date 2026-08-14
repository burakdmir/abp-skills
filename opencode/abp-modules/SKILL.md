---
name: abp-modules
description: "ABP Framework v10.x (10.4–10.6) pre-built application modules: Identity, Account, OpenIddict, Tenant/Permission/Setting/Feature Management, Audit Logging, CMS Kit, SaaS, Payment and more — which module solves what, how to install it, free vs Pro. Use when choosing, installing, or extending a pre-built ABP module (abp add-module) in an ABP application."
---

# ABP Framework — Pre-Built Application Modules

Pre-built application modules of ABP Framework v10.x (10.4–10.6): which module solves what, installation, free vs Pro.

## Trigger

- "ABP modules"
- "ABP Identity module" / "ABP Account module"
- "abp add-module"
- "ABP CMS Kit" / "ABP SaaS module"
- "ABP free vs Pro modules"

## Application vs Framework Modules

- **Framework modules**: infrastructure (caching, validation, EF Core integration) — no business features.
- **Application modules** (this skill): full features with entities, services, APIs, UI.

## Module Table

| Module | Package / Name | Purpose | License |
|---|---|---|---|
| Account | `Volo.Abp.Account` | Login, register, forgot password | Free (Pro: social logins) |
| Identity | `Volo.Abp.Identity` | Users, roles, OUs, permissions (Microsoft Identity) | Free (Pro: claims, advanced UI) |
| OpenIddict | OpenIddict module (`Volo.Abp.Account.Web.OpenIddict`) | SSO, OAuth2/OIDC server | Free (Pro: management UI) |
| Permission Management | `Volo.Abp.PermissionManagement` | Persists permissions (`IPermissionStore`) | Free |
| Setting Management | `Volo.Abp.SettingManagement` | Persists settings (`ISettingStore`/`ISettingManager`) | Free |
| Feature Management | `Volo.Abp.FeatureManagement` | Persists feature values (`IFeatureStore`) | Free |
| Tenant Management | Tenant Management module | Tenant CRUD (`ITenantStore`) | Free |
| Audit Logging | conn. string `AbpAuditLogging` | Persists audit logs (`IAuditingStore`) | Free (Pro: reporting UI) |
| Background Jobs | conn. string `AbpBackgroundJobs` | Persists jobs (`IBackgroundJobStore`) | Free |
| CMS Kit | `Volo.CmsKit` | CMS building blocks (pages, comments, tags…) | Free (Pro: extra blocks) |
| Docs | `Volo.Docs` | Documentation website (GitHub / local FS source) | Free |
| Blogging | `Volo.Blogging` | Blogs, posts, tags, comments | Free |
| SaaS | `Volo.Saas` | Tenants + editions, per-tenant conn strings | **Pro** |
| File Management | `Volo.FileManagement` | Files in hierarchical folders (BLOB Storing) | **Pro** |
| Payment | `Volo.Payment` | Stripe, PayPal, 2Checkout, PayU, Iyzico, Alipay | **Pro** |
| Chat | `Volo.Chat` | Real-time user messaging (SignalR) | **Pro** |
| GDPR | `Volo.Gdpr` | Personal data download/deletion requests | **Pro** |
| Language Management | `Volo.LanguageManagement` | Manage languages, translate UI on the fly | **Pro** |
| Text Template Management | `Volo.TextTemplateManagement` | Edit text templates (e.g. account emails) | **Pro** |
| Operation Rate Limiting | `Volo.Abp.OperationRateLimiting` | App/domain-level rate limiting | **Pro** |
| Forms / Twilio SMS | Forms, Twilio SMS modules | Surveys / SMS sending | **Pro** |

Pro modules require an ABP Team or higher license.

## Installing a Module

Identity, Account, OpenIddict, Permission/Setting/Feature/Tenant Management, Audit Logging, Background Jobs come **pre-installed** with startup templates.

```bash
# Add a whole module to all solution layers
abp add-module Volo.Blogging
abp add-module Volo.Docs
abp add-module Volo.CmsKit --skip-db-migrations
abp add-module Volo.FileManagement    # Pro

# Single package to current project only
abp add-package Volo.Abp.Blogging

# Pull a module's source into the solution
abp get-source Volo.Abp.Identity
```

After adding an EF Core module: add a migration and run the `DbMigrator`.

## Extending a Module

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

Prefer entity extensions, service replacement, and UI overrides over forking; `abp get-source` only when internals must change.

## Best Practices

1. Prefer a pre-built module over custom code — identity, tenants, settings, audit are solved problems.
2. Keep modules as NuGet packages; use `abp add-module`, not manual references.
3. Check free vs Pro early (Tenant Management free vs SaaS Pro editions).
4. Extend entities via `ObjectExtensionManager`, not by forking.
5. Use module connection string names (`AbpAuditLogging`, `AbpBackgroundJobs`) to split databases.

## Related

[Modularity](../abp-modularity/SKILL.md) · [Framework](../abp-framework/SKILL.md) · [ABP CLI](../abp-cli/SKILL.md) · Docs: https://abp.io/docs/latest/modules

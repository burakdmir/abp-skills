---
description: Audit an ABP solution for readiness against the latest stable ABP (v10.5) / .NET 10 — deprecated APIs, outdated packages, auth migration, and configuration drift.
---

# ABP upgrade audit (dynamic target)

Optional focus (path, area, or explicit target version like `10.4`; empty = whole solution, latest stable): **$ARGUMENTS**

First detect the solution's **current** ABP version (`Volo.Abp.*` package version in `Directory.Packages.props` / `*.csproj`). The **target** is the version given in the arguments, or the latest stable (**v10.5**) if none is given. Scan the solution and report what stands between the current state and a clean ABP **target version on .NET 10** state. Use Grep/Glob across the repo; do not change code unless asked.

Check and report:

1. **Target framework** — `*.csproj` `<TargetFramework>` values vs `net10.0`; `global.json` SDK pin.
2. **ABP package versions** — `Volo.Abp.*` package versions vs the target (`10.5.x` by default); mismatched/mixed versions across projects.
3. **Auth migration** — any `ConfigureIdentityServer` / IdentityServer packages still present (must move to OpenIddict / `ConfigureOpenIddict`).
4. **Object mapping** — AutoMapper-only setups that could adopt Mapperly (default since v10.4); missing mapping registrations.
5. **Convention drift** — `DateTime.Now` instead of `IClock`; `.Result`/`.Wait()` blocking calls; repositories leaking `IQueryable`; missing `ConfigureByConvention()`.
6. **Module wiring** — `DependsOn` gaps, missing `[DependsOn]` for packages that are referenced.
7. **v10.5-specific checks** (when crossing 10.4 → 10.5):
   - Identity token providers became **single-active** per user/purpose (`AbpDefaultTokenProvider` replaces the default `DataProtectorTokenProvider`; default lifetime 10 min via `AbpDefaultTokenProviderOptions`/`AbpLinkUserTokenProviderOptions`). Flag flows that keep multiple tokens valid (2FA, password reset/change, link-user) for re-test.
   - Custom logic on `IDynamicBackgroundWorkerManager` must check the new capability markers (`ISupportsRuntimeRegistration`, `ISupportsCronScheduling`) — the in-memory manager now rejects cron expressions.
   - MySQL + Permission Management: `ResourcePermissionGrant` column max lengths were shortened for MySQL — regenerate/review fresh migrations.
   - Direct pins on Blazorise (→ 2.2.1), MongoDB.Driver (→ 3.9.0), or CodeMirror (→ 6.0.2) that would conflict with ABP's versions.

Output a prioritized checklist (🔴/🟠/🟡) as `area — finding — recommended action`, each with the file path(s). State the detected current and target versions up front. Finish with a short "migration order" suggestion. Reference the official ABP migration guides (e.g. `abp-10-5.md`) and the relevant `abp-*` skills, and flag anything that needs manual verification rather than guessing.

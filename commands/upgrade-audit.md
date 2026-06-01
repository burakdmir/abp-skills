---
description: Audit an ABP solution for v10.4 / .NET 10 readiness — deprecated APIs, outdated packages, auth migration, and configuration drift.
---

# ABP v10.4 / .NET 10 upgrade audit

Optional focus (path or area; empty = whole solution): **$ARGUMENTS**

Scan the solution and report what stands between it and a clean ABP **v10.4 on .NET 10** state. Use Grep/Glob across the repo; do not change code unless asked.

Check and report:

1. **Target framework** — `*.csproj` `<TargetFramework>` values vs `net10.0`; `global.json` SDK pin.
2. **ABP package versions** — `Volo.Abp.*` package versions vs `10.4.x`; mismatched/mixed versions across projects.
3. **Auth migration** — any `ConfigureIdentityServer` / IdentityServer packages still present (must move to OpenIddict / `ConfigureOpenIddict`).
4. **Object mapping** — AutoMapper-only setups that could adopt Mapperly (the v10.4 default); missing mapping registrations.
5. **Convention drift** — `DateTime.Now` instead of `IClock`; `.Result`/`.Wait()` blocking calls; repositories leaking `IQueryable`; missing `ConfigureByConvention()`.
6. **Module wiring** — `DependsOn` gaps, missing `[DependsOn]` for packages that are referenced.

Output a prioritized checklist (🔴/🟠/🟡) as `area — finding — recommended action`, each with the file path(s). Finish with a short "migration order" suggestion. Reference the official ABP migration guides and the relevant `abp-*` skills, and flag anything that needs manual verification rather than guessing.

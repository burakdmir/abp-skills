---
name: abp-deployment
description: "ABP Framework v10.4 deployment quick reference: clustered/stateless, distributed cache (Redis), BLOB provider, distributed lock, SignalR backplane, DataProtection, ForwardedHeaders, SSL, OpenIddict prod certificates, Docker/Helm. Use when you need to deploy an ABP application to production."
---

# ABP Framework — Deployment

ABP v10.4 deployment. Standard .NET deployment + ABP-specific clustered/proxy/OpenIddict notes.

## Trigger

"ABP deploy", "production", "Docker/Kubernetes", "clustered/multiple instance", "reverse proxy/forwarded headers", "SSL".

## Clustered (design stateless)

- **Distributed cache (Redis):** `AbpCachingStackExchangeRedisModule` + `"Redis":{"Configuration":"..."}`. In-memory cache is not enough in a cluster.
- **BLOB:** Database BLOB (ready in the template) or a cloud provider instead of the File System provider.
- **Distributed lock:** configure a provider so the background job runs on a single instance (default is in-process).
- **SignalR backplane:** Redis backplane for multiple instances.
- **DataProtection:** point keys to a shared store (Redis) (cookie/token encryption).

## Forwarded Headers (reverse proxy)

```csharp
context.Services.Configure<ForwardedHeadersOptions>(o =>
    o.ForwardedHeaders = ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto);
// at the start of the pipeline:
app.UseForwardedHeaders();
```

## OpenIddict / Production

- Persistent signing/encryption certificates in production (not the automatic dev certificate).
- Secrets (connection string, Redis, cert) → env var / secret store, **do not write to the repo**.
- `ASPNETCORE_ENVIRONMENT=Production` + `appsettings.Production.json`.
- Migration: `dotnet run --project src/MyProject.DbMigrator`.
- HTTPS mandatory.

## Docker / Kubernetes

Microservice template: `etc/docker` (compose) + `etc/helm` (K8s). Monolith → standard .NET image; clustered principles still apply.

## Best Practices

1. Design stateless; distributed cache/lock/SignalR backplane (Redis)
2. Non-File-System provider for BLOB
3. Configure ForwardedHeaders
4. OpenIddict prod certificates + secrets in a secret store
5. Migrations with DbMigrator, HTTPS mandatory

## Related

- [Infrastructure](../abp-infrastructure/SKILL.md) · [Microservices](../abp-microservices/SKILL.md) · [Authorization](../abp-authorization/SKILL.md)
- ABP Docs: https://abp.io/docs/latest/deployment

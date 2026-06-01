---
name: abp-deployment
description: "ABP Framework v10.4 deployment: production config, clustered/stateless design, distributed cache (Redis), BLOB provider selection, distributed lock, SignalR backplane, ForwardedHeaders (reverse proxy), SSL/HTTPS, OpenIddict production certificates, Docker/Kubernetes (Helm). Use when you need to deploy an ABP application to production or containerize it."
---

# ABP Framework — Deployment

ABP Framework v10.4 deployment guide. An ABP application is deployed like any .NET/ASP.NET Core application (Azure/AWS/GCP/on-prem/IIS). However, there are ABP-specific points to watch for clustered environments, reverse proxy, OpenIddict, and production configuration.

## Trigger

- "ABP deploy"
- "ABP production"
- "ABP Docker / Kubernetes"
- "ABP clustered / multiple instance"
- "ABP reverse proxy / forwarded headers"
- "ABP SSL / HTTPS"

## Clustered / Multi-Instance Environment

When running multiple instances (cluster, container, cloud), **design the application to be stateless** — state kept in memory is lost because the next request may be handled by another instance.

### 1. Distributed Cache (Redis)

In-memory cache is per-instance. Use a **distributed cache** in a cluster. ABP Distributed Cache extends the ASP.NET Core distributed cache; the default is in-memory — configure a real provider (Redis) in production:

```csharp
// Volo.Abp.Caching.StackExchangeRedis package
[DependsOn(typeof(AbpCachingStackExchangeRedisModule))]
```
```json
"Redis": { "Configuration": "localhost:6379" }
```
> In solutions created by selecting Tiered + MVC, Redis usually comes ready out of the box.

### 2. BLOB Storage Provider

The File System BLOB provider uses the local disk → not suitable in a cluster. Use the **Database BLOB provider** (ready in startup templates) or a cloud provider (Azure/AWS S3).

### 3. Distributed Lock

The default ABP background job manager uses a distributed lock to ensure jobs run on a single instance. In a cluster, configure a **distributed lock provider** (the DistributedLock library; e.g. Redis-based). The default is in-process — meaning it is not actually distributed unless a provider is configured.

### 4. SignalR Backplane

Configure a backplane (e.g. Redis) for SignalR across multiple instances.

### 5. DataProtection

For anti-forgery, cookie, and token encryption, point DataProtection keys to a shared store that all instances can access (e.g. Redis) — otherwise instances cannot decrypt each other's tokens.

## Reverse Proxy — Forwarded Headers

Behind a reverse proxy/load balancer, the original client IP, host, and protocol (`X-Forwarded-For`, `X-Forwarded-Proto`, `X-Forwarded-Host`) must be read correctly:

```csharp
// ConfigureServices
context.Services.Configure<ForwardedHeadersOptions>(options =>
{
    options.ForwardedHeaders =
        ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto;
});

// OnApplicationInitialization — at the start of the pipeline
app.UseForwardedHeaders();
```
If not done correctly, URLs are generated with the wrong protocol (http/https), and IP-based logging/auth breaks.

## SSL / HTTPS

HTTPS is mandatory in production. Configure the SSL certificate at the reverse proxy (Nginx/YARP) or application level. Detail: ABP "Configuring SSL" guide.

## OpenIddict — Production

The automatic certificates used in development are not suitable for production. In production, configure **persistent signing/encryption certificates** (e.g. `.pfx` + password, key store). Do not embed certificate/secret values in code — use environment variables / a secret store. Detail: ABP "Configuring OpenIddict" guide.

## Production Configuration

- `ASPNETCORE_ENVIRONMENT=Production` and `appsettings.Production.json`
- Pass **secrets** such as the connection string, Redis, and OpenIddict certificate via a secret store / environment variable (do not write them to the repo)
- Apply migrations with DbMigrator (e.g. a CI/CD step): `dotnet run --project src/MyProject.DbMigrator`

## Docker & Kubernetes

The microservice solution template includes ready infrastructure:

```
etc/
├── docker/     # docker compose for local infrastructure (Redis, RabbitMQ, DB...)
└── helm/       # Kubernetes deployment (Helm chart)
```
Monolith applications are also packaged as a standard .NET Docker image. In a container, the clustered principles above (distributed cache/lock, BLOB, DataProtection) apply.

## Best Practices

1. **Design stateless** — for cluster/container/cloud
2. **Configure distributed cache + lock + SignalR backplane** (Redis)
3. **Use a non-File-System provider for BLOB**
4. **Configure ForwardedHeaders** — behind a reverse proxy
5. **OpenIddict production certificates** + keep secrets in a secret store
6. **Apply migrations with DbMigrator** in the pipeline
7. **HTTPS mandatory**, `ASPNETCORE_ENVIRONMENT=Production`

## Related

- [Infrastructure](../abp-infrastructure/SKILL.md) — distributed cache, BLOB, background jobs, distributed lock
- [Microservices](../abp-microservices/SKILL.md) — gateway, docker/helm, distributed deployment
- [Authorization](../abp-authorization/SKILL.md) — OpenIddict
- ABP Docs: https://abp.io/docs/latest/deployment

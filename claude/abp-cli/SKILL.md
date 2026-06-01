---
name: abp-cli
description: "ABP Framework v10.4 CLI and tooling: abp new, add-package, generate-proxy, ABP Studio, ABP Suite, --modern flag. Use when working with ABP CLI commands, creating projects, adding packages, or generating proxies."
---

# ABP Framework — CLI & Tooling

ABP Framework v10.4 CLI commands, ABP Studio, ABP Suite, and development tooling guide.

## Trigger

- "ABP CLI"
- "ABP command"
- "abp new"
- "abp update"
- "ABP Suite"
- "ABP Studio"
- "ABP generate-proxy"
- "ABP package management"

## ABP CLI Installation

```bash
# Installation
dotnet tool install -g Volo.Abp.Studio.Cli

# Update
dotnet tool update -g Volo.Abp.Studio.Cli

# Global options
--skip-cli-version-check (-scvc)   # Skip the version check
--skip-extension-version-check (-sevc)  # Skip the extension check
--old  # Use the old CLI (Volo.Abp.Cli)
--help (-h)  # Help
```

## Creating a Solution

```bash
# Layered application (default)
abp new Acme.BookStore --template app

# Single-layer
abp new Acme.BookStore --template app-nolayers

# Microservice (Business+ license)
abp new Acme.BookStore --template microservice

# Modern template (React-first)
abp new Acme.BookStore --template app --modern
abp new Acme.BookStore --template app-nolayers --modern --modular
abp new Acme.BookStore --template microservice --modern

# DBMS selection
abp new Acme.BookStore --database-management-system PostgreSQL
abp new Acme.BookStore --database-management-system MySQL

# Connection string
abp new Acme.BookStore --connection-string "Server=localhost;Database=BookStore;Trusted_Connection=True"

# Multi-tenancy disabled
abp new Acme.BookStore --no-multi-tenancy

# Without test projects
abp new Acme.BookStore --no-tests

# Theme selection
abp new Acme.BookStore --theme leptonx-lite
abp new Acme.BookStore --theme basic

# Shadcn theme (modern)
abp new Acme.BookStore --modern --shadcn-theme blue
```

### Template + UI Combinations

| Template | MVC | Angular | Blazor | React | No-UI |
|---|---|---|---|---|---|
| `app` | ✓ | ✓ | ✓ | --modern | ✓ |
| `app-nolayers` | ✓ | ✓ | ✓ | --modern | ✓ |
| `microservice` | ✓ | ✓ | ✓ | --modern | ✓ |

### Modern Template Options

| Option | Description |
|---|---|
| `--shadcn-theme <theme>` | slate, pink, blue, turquoise, orange, purple |
| `--admin-password <password>` | Initial admin password |
| `--modular` | Modular monolith variant (`app-nolayers --modern`) |
| `--services <list>` | Additional microservice names (comma-separated) |

## Creating a Module

```bash
# DDD module
abp new-module Acme.BookStore.Orders -t module:ddd

# Modern module
abp new-module Acme.BookStore.Orders --modern

# Add to the target solution
abp new-module Acme.BookStore.Orders -t module:ddd -ts Acme.BookStore.sln

# EF + MVC
abp new-module Acme.BookStore.Orders -d ef -u mvc
```

## Package Management

```bash
# Add a package (NuGet + DependsOn automatically)
abp add-package Volo.Abp.EntityFrameworkCore
abp add-package Volo.Abp.AspNetCore.Mvc.UI.Theme.Basic

# Add with source code
abp add-package Volo.Abp.AspNetCore.Mvc.UI.Theme.Basic --with-source-code

# Update all ABP packages
abp update
abp update --version 8.0.0       # Specific version
abp update --npm                 # NPM only
abp update --nuget               # NuGet only

# Switch between Preview/Stable/Nightly
abp switch-to-preview
abp switch-to-stable
abp switch-to-nightly

# NuGet → Local project reference
abp switch-to-local --paths "D:\Github\abp"
```

## Generate Proxy

```bash
# Angular proxy
abp generate-proxy -t ng
abp generate-proxy -t ng -url https://localhost:44302/
abp generate-proxy -t ng -a default -m identity

# C# proxy
abp generate-proxy -t csharp
abp generate-proxy -t csharp --without-contracts

# JavaScript proxy
abp generate-proxy -t js

# Remove proxy
abp remove-proxy -t ng
abp remove-proxy -t csharp
```

## Module Source Management

```bash
# List modules
abp list-modules

# Download source code
abp get-source Volo.Blogging
abp get-source Volo.Blogging --local-framework-ref --abp-path D:\GitHub\abp

# Add source code to the project (NuGet → project reference)
abp add-source-code Volo.Chat --add-to-solution-file

# Install a module
abp install-module Volo.Blogging
abp install-local-module ../Acme.Blogging

# Remote module source management
abp list-module-sources
abp add-module-source -n "Custom" -p "https://example.com/modules.json"
abp delete-module-source -n "Custom"
```

## Listing Templates

```bash
abp list-templates
```

## Cleaning the Project

```bash
abp clean  # Deletes all BIN/OBJ folders
```

## ABP Studio Integration

```bash
# Create the Studio config for a solution
abp init-solution --name Acme.BookStore

# Kubernetes connection (Business+ license)
abp kube-connect
abp kube-connect -p Default.abpk8s.json
abp kube-connect -c docker-desktop -ns mycrm-local

# Kubernetes service intercept (Business+ license)
abp kube-intercept mycrm-product-service -ns mycrm-local
abp kube-intercept mycrm-product-service -ns mycrm-local -a MyCrm.ProductService.HttpApi.Host.csproj
```

## ABP Suite

ABP Suite is a tool that automatically generates CRUD pages:

1. Define the entity and its properties
2. Specify the relationships
3. Choose the UI framework (MVC/Blazor/Angular)
4. CRUD pages are generated automatically

```bash
# No CLI command for Suite — it runs through ABP Studio
```

## Localization Translation

```bash
# Create a unified translation file
abp translate -c de                    # German
abp translate -c de -r en              # Reference: English
abp translate -c de -o my-trans.json   # Output file
abp translate -c de --all-values       # All keys (including already translated ones)

# Apply translations
abp translate --apply
abp translate -a
```

## Upgrade (Open Source → PRO)

```bash
abp upgrade -t app
abp upgrade -t app --language-management --gdpr --audit-logging-ui
abp upgrade -t app-nolayers --audit-logging-ui
```

## MCP Studio (AI Tools)

```bash
abp mcp-studio  # ABP Studio MCP bridge (for AI tools, ABP Studio must be running)
```

## Other Commands

```bash
# Install NPM packages (MVC/Blazor)
abp install-libs

# Blazor script/style bundle
abp bundle

# Clear the template download cache
abp clear-download-cache

# Extension version check
abp check-extensions

# Install the old CLI
abp install-old-cli

# Generate a Razor page
abp generate-razor-page

# Generate JWKS (OpenIddict private_key_jwt)
abp generate-jwks

# Login
abp login
abp login-info
abp logout

# Build
abp build

# Create a package
abp new-package --name Acme.BookStore.Domain --template lib.domain

# Add a package reference
abp add-package-ref Acme.BookStore.Domain
abp add-package-ref "Acme.BookStore.Domain Acme.BookStore.Domain.Shared" -t Acme.BookStore.Web
```

### new-package Templates

| Template | Description |
|---|---|
| `lib.class-library` | Class library |
| `lib.domain-shared` | Domain shared |
| `lib.domain` | Domain |
| `lib.application-contracts` | Application contracts |
| `lib.application` | Application |
| `lib.ef` | EF Core |
| `lib.mongodb` | MongoDB |
| `lib.http-api` | HTTP API |
| `lib.http-api-client` | HTTP API Client |
| `lib.mvc` | MVC |
| `lib.blazor` | Blazor |
| `host.http-api` | HTTP API Host |
| `host.mvc` | MVC Host |
| `host.blazor-server` | Blazor Server Host |

## Best Practices

1. **Always use the latest CLI version** — `dotnet tool update -g Volo.Abp.Studio.Cli`
2. **Prefer modern templates** — React-first, more up-to-date stack
3. **Keep packages up to date with `abp update`** — Use `abp switch-to-stable` for the Preview→Stable transition
4. **The server must be running to generate proxies** — `abp generate-proxy`
5. **Use `abp new-module` for module development** — Instead of creating manually
6. **Use `abp add-source-code` when you need the source code** — For debugging and customization

---

## Related

- [Framework Core](../abp-framework/SKILL.md) — solution templates, `abp new`
- [Modularity](../abp-modularity/SKILL.md) — `abp new-module`, modular monolith
- [Microservices](../abp-microservices/SKILL.md) — `generate-proxy`, adding services
- [Development Flow](../abp-development-flow/SKILL.md) — migration, proxy generation steps
- ABP Docs: https://abp.io/docs/latest/cli

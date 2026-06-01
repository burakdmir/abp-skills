---
name: abp-cli
description: "ABP Framework v10.4 CLI and tooling: abp new, add-package, generate-proxy, ABP Studio, ABP Suite, --modern flag. Use when working with ABP CLI commands, creating projects, adding packages, or generating proxies."
---

# ABP Framework — CLI & Tooling

ABP Framework v10.4 CLI commands and tooling guide.

## Trigger

- "ABP CLI"
- "abp new"
- "abp update"
- "ABP generate-proxy"
- "ABP package"

## Installation

```bash
dotnet tool install -g Volo.Abp.Studio.Cli
dotnet tool update -g Volo.Abp.Studio.Cli
```

## Creating a Solution

```bash
abp new Acme.BookStore --template app
abp new Acme.BookStore --template app --modern
abp new Acme.BookStore --template app-nolayers --modern --modular
abp new Acme.BookStore --template microservice --modern
abp new Acme.BookStore --database-management-system PostgreSQL
abp new Acme.BookStore --no-multi-tenancy
abp new Acme.BookStore --modern --shadcn-theme blue
```

## Module

```bash
abp new-module Acme.Orders -t module:ddd
abp new-module Acme.Orders --modern
```

## Package

```bash
abp add-package Volo.Abp.EntityFrameworkCore
abp update
abp switch-to-stable
abp switch-to-preview
abp switch-to-local --paths "D:\Github\abp"
```

## Proxy

```bash
abp generate-proxy -t ng
abp generate-proxy -t csharp
abp remove-proxy -t ng
```

## Module Source

```bash
abp list-modules
abp get-source Volo.Blogging
abp add-source-code Volo.Chat --add-to-solution-file
abp install-module Volo.Blogging
```

## Other

```bash
abp clean
abp list-templates
abp init-solution --name Acme.BookStore
abp translate -c de
abp translate --apply
abp upgrade -t app
abp install-libs
abp bundle
abp build
abp new-package --name Acme.BookStore.Domain --template lib.domain
abp add-package-ref Acme.BookStore.Domain
```

## Best Practices

1. Use the latest CLI version
2. Prefer modern templates
3. Keep packages up to date with `abp update`
4. The server must be running to generate proxies

## Related

[Framework](../abp-framework/SKILL.md) · [Modularity](../abp-modularity/SKILL.md) · [Microservices](../abp-microservices/SKILL.md) · [Development Flow](../abp-development-flow/SKILL.md) · Docs: https://abp.io/docs/latest/cli

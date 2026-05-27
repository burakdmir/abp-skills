# ABP Framework — CLI & Tooling

ABP Framework v10.4 CLI komutları ve tooling rehberi.

## Trigger

- "ABP CLI"
- "abp new"
- "abp update"
- "ABP generate-proxy"
- "ABP package"

## Kurulum

```bash
dotnet tool install -g Volo.Abp.Studio.Cli
dotnet tool update -g Volo.Abp.Studio.Cli
```

## Solution Oluşturma

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

## Diğer

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

1. En son CLI versiyonunu kullan
2. Modern template'leri tercih et
3. `abp update` ile paketleri güncel tut
4. Proxy generate için server çalışır olmalı

# ABP Framework — CLI & Tooling

ABP Framework v10.4 CLI komutları, ABP Studio, ABP Suite ve geliştirme tooling rehberi.

## Trigger

- "ABP CLI"
- "ABP komut"
- "abp new"
- "abp update"
- "ABP Suite"
- "ABP Studio"
- "ABP generate-proxy"
- "ABP package yönetimi"

## ABP CLI Kurulum

```bash
# Kurulum
dotnet tool install -g Volo.Abp.Studio.Cli

# Güncelleme
dotnet tool update -g Volo.Abp.Studio.Cli

# Global options
--skip-cli-version-check (-scvc)   # Versiyon kontrolünü atla
--skip-extension-version-check (-sevc)  # Extension kontrolünü atla
--old  # Eski CLI (Volo.Abp.Cli) kullan
--help (-h)  # Yardım
```

## Solution Oluşturma

```bash
# Layered application (varsayılan)
abp new Acme.BookStore --template app

# Single-layer
abp new Acme.BookStore --template app-nolayers

# Microservice (Business+ lisans)
abp new Acme.BookStore --template microservice

# Modern template (React-first)
abp new Acme.BookStore --template app --modern
abp new Acme.BookStore --template app-nolayers --modern --modular
abp new Acme.BookStore --template microservice --modern

# DBMS seçimi
abp new Acme.BookStore --database-management-system PostgreSQL
abp new Acme.BookStore --database-management-system MySQL

# Connection string
abp new Acme.BookStore --connection-string "Server=localhost;Database=BookStore;Trusted_Connection=True"

# Multi-tenancy kapalı
abp new Acme.BookStore --no-multi-tenancy

# Test projeleri olmadan
abp new Acme.BookStore --no-tests

# Theme seçimi
abp new Acme.BookStore --theme leptonx-lite
abp new Acme.BookStore --theme basic

# Shadcn theme (modern)
abp new Acme.BookStore --modern --shadcn-theme blue
```

### Template + UI Kombinasyonları

| Template | MVC | Angular | Blazor | React | No-UI |
|---|---|---|---|---|---|
| `app` | ✓ | ✓ | ✓ | --modern | ✓ |
| `app-nolayers` | ✓ | ✓ | ✓ | --modern | ✓ |
| `microservice` | ✓ | ✓ | ✓ | --modern | ✓ |

### Modern Template Seçenekleri

| Option | Açıklama |
|---|---|
| `--shadcn-theme <theme>` | slate, pink, blue, turquoise, orange, purple |
| `--admin-password <password>` | Initial admin şifresi |
| `--modular` | Modular monolith variant (`app-nolayers --modern`) |
| `--services <list>` | Ek microservice isimleri (virgülle ayrılmış) |

## Module Oluşturma

```bash
# DDD modülü
abp new-module Acme.BookStore.Orders -t module:ddd

# Modern modül
abp new-module Acme.BookStore.Orders --modern

# Hedef solution'a ekle
abp new-module Acme.BookStore.Orders -t module:ddd -ts Acme.BookStore.sln

# EF + MVC
abp new-module Acme.BookStore.Orders -d ef -u mvc
```

## Package Yönetimi

```bash
# Package ekle (NuGet + DependsOn otomatik)
abp add-package Volo.Abp.EntityFrameworkCore
abp add-package Volo.Abp.AspNetCore.Mvc.UI.Theme.Basic

# Kaynak kodu ile ekle
abp add-package Volo.Abp.AspNetCore.Mvc.UI.Theme.Basic --with-source-code

# Tüm ABP paketlerini güncelle
abp update
abp update --version 8.0.0       # Belirli versiyon
abp update --npm                 # Sadece NPM
abp update --nuget               # Sadece NuGet

# Preview/Stable/Nightly arası geçiş
abp switch-to-preview
abp switch-to-stable
abp switch-to-nightly

# NuGet → Local project reference
abp switch-to-local --paths "D:\Github\abp"
```

## Proxy Generate

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

# Proxy kaldır
abp remove-proxy -t ng
abp remove-proxy -t csharp
```

## Module Kaynak Yönetimi

```bash
# Modül listele
abp list-modules

# Kaynak kodu indir
abp get-source Volo.Blogging
abp get-source Volo.Blogging --local-framework-ref --abp-path D:\GitHub\abp

# Kaynak kodu projeye ekle (NuGet → project reference)
abp add-source-code Volo.Chat --add-to-solution-file

# Modül yükle
abp install-module Volo.Blogging
abp install-local-module ../Acme.Blogging

# Remote module source yönetimi
abp list-module-sources
abp add-module-source -n "Custom" -p "https://example.com/modules.json"
abp delete-module-source -n "Custom"
```

## Template Listeleme

```bash
abp list-templates
```

## Proje Temizleme

```bash
abp clean  # Tüm BIN/OBJ klasörlerini siler
```

## ABP Studio Entegrasyonu

```bash
# Solution için Studio config oluştur
abp init-solution --name Acme.BookStore

# Kubernetes bağlantısı (Business+ lisans)
abp kube-connect
abp kube-connect -p Default.abpk8s.json
abp kube-connect -c docker-desktop -ns mycrm-local

# Kubernetes service intercept (Business+ lisans)
abp kube-intercept mycrm-product-service -ns mycrm-local
abp kube-intercept mycrm-product-service -ns mycrm-local -a MyCrm.ProductService.HttpApi.Host.csproj
```

## ABP Suite

ABP Suite, CRUD sayfa otomatik oluşturma aracıdır:

1. Entity ve property'lerini tanımla
2. İlişkileri belirle
3. UI framework seç (MVC/Blazor/Angular)
4. CRUD sayfaları otomatik generate edilir

```bash
# Suite ile ilgili CLI komutu yok — ABP Studio üzerinden çalışır
```

## Localization Çeviri

```bash
# Unified translation dosyası oluştur
abp translate -c de                    # Almanca
abp translate -c de -r en              # Referans: İngilizce
abp translate -c de -o my-trans.json   # Çıktı dosyası
abp translate -c de --all-values       # Tüm keys (zaten çevrilmiş olanlar dahil)

# Çevirileri uygula
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
abp mcp-studio  # ABP Studio MCP bridge (AI tools için, ABP Studio çalışıyor olmalı)
```

## Diğer Komutlar

```bash
# NPM paketlerini yükle (MVC/Blazor)
abp install-libs

# Blazor script/style bundle
abp bundle

# Template download cache temizle
abp clear-download-cache

# Extension versiyon kontrolü
abp check-extensions

# Eski CLI kurulumu
abp install-old-cli

# Razor page generate
abp generate-razor-page

# JWKS generate (OpenIddict private_key_jwt)
abp generate-jwks

# Login
abp login
abp login-info
abp logout

# Build
abp build

# Package oluştur
abp new-package --name Acme.BookStore.Domain --template lib.domain

# Package referansı ekle
abp add-package-ref Acme.BookStore.Domain
abp add-package-ref "Acme.BookStore.Domain Acme.BookStore.Domain.Shared" -t Acme.BookStore.Web
```

### new-package Template'leri

| Template | Açıklama |
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

1. **Her zaman en son CLI versiyonunu kullan** — `dotnet tool update -g Volo.Abp.Studio.Cli`
2. **Modern template'leri tercih et** — React-first, daha güncel stack
3. **`abp update` ile paketleri güncel tut** — Preview→Stable geçiş için `abp switch-to-stable`
4. **Proxy generate için server çalışır durumda olmalı** — `abp generate-proxy`
5. **Module development için `abp new-module` kullan** — Manuel oluşturmak yerine
6. **Kaynak kodu gerektiğinde `abp add-source-code` kullan** — Debug ve customizasyon için

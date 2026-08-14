# Changelog

All notable changes to the **abp-sensei** plugin and its skill trees are documented here.

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) · Versioning: [Semantic Versioning](https://semver.org/).

## [1.2.0] - 2026-08-14

### Added

- **3 new skills** (both trees, 23 → 26 topics): `abp-ai` (Microsoft.Extensions.AI, Semantic Kernel, Agent Framework, AI Management/RAG), `abp-modules` (pre-built module reference, free vs Pro), `abp-dapr` (Dapr packages, service invocation, event bus)
- New sections: Concurrency Check (`abp-ddd`), Dynamic Claims (`abp-authorization`), Cancellation Token Provider (`abp-infrastructure`)
- **ABP v10.6 support** (released 2026-07-27) across all 46 skills, the `abp-expert` agent, and slash commands:
  - Background jobs: dedicated workers, parallel execution, successful-job retention — `abp-infrastructure`
  - API definition `ContentTypes` / `IsRemoteStream` + multipart `FormData` upload proxies — `abp-api`
  - Angular 22 upgrade guidance + locale-loading fallback — `abp-ui`
  - Antiforgery user-id claim issuer normalization, OpenIddict cookie `client_id` fix, access-token forwarding for authenticated client requests — `abp-authorization`
  - MongoDB.Driver 3.10.0 — `abp-mongodb`
  - New **"v10.6-specific checks"** section in `/abp-sensei:upgrade-audit`
- `argument-hint` frontmatter on all slash commands
- Official `claude plugin validate --strict` step in CI
- This changelog

### Changed

- Latest-stable target: v10.5 → **v10.6** (agent, `explain`, `upgrade-audit`, framework skills)
- Dependency pin guidance: MongoDB.Driver → 3.10.0, Swashbuckle.AspNetCore → 10.2.3, Microsoft.Data.SqlClient → 7.0.2, `Microsoft.*`/`System.*` → 10.0.9

## [1.1.0] - 2026-07-13

### Added

- **ABP v10.5 support** with dynamic per-solution version detection; version-specific features marked `v10.5+`
- v10.5 topics: S3-compatible blob storage, single-active identity token providers, OpenIddict default-scope fallback, dynamic background worker capability markers, MySQL `ResourcePermissionGrant` index fix

## [1.0.0] - 2026-06-01

### Added

- Initial release: 23 Claude Code + 23 OpenCode skills, `abp-expert` subagent, 5 slash commands (`new-entity`, `crud`, `review`, `upgrade-audit`, `explain`)
- Plugin marketplace packaging (`/plugin marketplace add burakdmir/abp-skills`)
- CI validation for skill frontmatter, tree parity, broken links, and plugin manifests

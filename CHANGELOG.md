# Changelog

All notable changes to **bitrix24-mcp** are documented here.

This file tracks the public-facing release timeline for the `.mcpb` bundle published to <https://rcs.kz/bitrix24-mcp> and the `@rcs-kz/bitrix24-mcp` npm package. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and version numbers follow [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Initial public-front repository (README, install guide, threat-model summary, security-disclosure policy).

## [1.0.0] — pending GA

First general-availability release. Planned scope:

### Added
- 45 tools across CRM (contacts, companies, deals, leads, products, activities), Tasks, Users, IM, and Calendar.
- Type-safe argument validation via Zod against strict JSON schemas.
- `destructive: true` declaration on all `*_delete` and mass-selector `*_update` tools — Claude Desktop surfaces a confirmation dialog before invocation.
- Per-tenant / per-user / per-tool rate limiter with backpressure queuing.
- Local audit log at `~/.bitrix24-mcp/audit.log` (JSON-lines, mode `0600`).
- Webhook URL / OAuth token storage in the OS keychain (Keychain / Credential Manager / libsecret via `keytar`).
- ECDSA P-256 license verification with tenant-scoped bundle hash.
- 14-day free trial flow with no credit card and no feature gating.
- Bilingual install guide (RU + EN).

### Security
- 10-attack harness (sentinel-zero, sentinel-all-zero, bytecode flip, loader comment, rebake, env skip, fs hook, license redirect, bytecode decompile, baseline). Summary published in [`docs/threat-model.md`](./docs/threat-model.md).
- Zero telemetry. Two outbound hosts only: customer's Bitrix24 tenant and `license.rcs.kz`.

[Unreleased]: https://github.com/rcs-kz/bitrix24-mcp/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/rcs-kz/bitrix24-mcp/releases/tag/v1.0.0

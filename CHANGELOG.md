# Changelog

All notable changes to **bitrix24-mcp** are documented here.

This file tracks the public-facing release timeline for the `.mcpb` bundle published to <https://rcs.kz/bitrix24-mcp> and the `@rcs-kz/bitrix24-mcp` npm package. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and version numbers follow [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

Nothing yet.

## [2.1.0] — 2026-08-24

Driven by the first paid evaluation, which needed manager-performance
reporting and found the toolset could prove a lead existed but not what
anyone did about it.

### Added
- Telephony call records — duration, direction, missed-call codes, recording links.
- CRM activities — calls, open-line sessions, emails and meetings bound to a lead and a responsible user, sliceable by channel and period.
- Open-line access — the portal's configured channels and full client conversation history, not only internal chats.
- Chat messaging — a real message to a user or group chat, so a report can land somewhere a human can reply. Previously only bell notifications were possible.
- Timeline comment reading — the read half of a previously write-only surface.
- Total tool count: 55.

### Fixed
- Protected builds are now emitted with a CommonJS marker beside them. Without it Node parsed the bundle as ESM and every protected build died on its first `require` — the likely cause of the withdrawn 2.0.0.
- The version reported to MCP clients is real. Every prior build announced itself as `0.0.0-dev` because the esbuild define never matched the expression the source read.
- Obfuscation is retried until it emits parseable JavaScript. Measured across three runs, five of nine attempts produced invalid output; a single-shot build had close to even odds of shipping a dead artifact.

### Distribution
- Ships as a one-click `.mcpb` bundle for Claude Desktop. The webhook URL and licence key are collected by a form instead of hand-edited into `claude_desktop_config.json`, which is where the first evaluation actually broke.
- Self-contained: one file, no dependencies to install, same artifact on macOS, Windows and Linux. Verified on Windows 10 x64 against a clean Node 20.

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

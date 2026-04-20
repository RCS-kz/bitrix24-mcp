# About this repository

This is the **public front** of [bitrix24-mcp](https://rcs.kz/bitrix24-mcp), a commercial MCP server that connects Claude Desktop to Bitrix24 Cloud.

## What lives here

- [`README.md`](./README.md) — product overview (RU + EN)
- [`CHANGELOG.md`](./CHANGELOG.md) — release notes for the published `.mcpb` bundle and the `@rcs-kz/bitrix24-mcp` npm package
- [`SECURITY.md`](./SECURITY.md) — how to report security findings, PGP key, SLA
- [`docs/install/claude-desktop.md`](./docs/install/claude-desktop.md) — step-by-step install guide for Claude Desktop
- [`docs/threat-model.md`](./docs/threat-model.md) — threat model and 10-attack harness summary (synchronised with the in-source attack suite each release)

## What does NOT live here

The source code — MCP server implementation, Bitrix24 REST adapter, license-verification logic, license server, trial flow, and the V8-bytecode build pipeline — is **proprietary** and not published here.

The product is distributed through these channels only:

| Channel | Artifact | Link |
|---|---|---|
| Landing | `.mcpb` bundle + docs | <https://rcs.kz/bitrix24-mcp> |
| npm | `@rcs-kz/bitrix24-mcp` | <https://www.npmjs.com/package/@rcs-kz/bitrix24-mcp> |
| MCP Registry | Metadata entry | `io.github.rcs-kz/bitrix24-mcp` |
| Bitrix24.Market | Vendor listing | pending review |

See the [Commercial EULA](https://rcs.kz/bitrix24-mcp/eula) for license terms that apply to the binary.

## Why README-only?

Three reasons:

1. **Enterprise procurement & SSO reviewers expect a GitHub presence.** They want to see the changelog, the threat model, and a security contact before approving an install. This repo gives them that without leaking the source.
2. **MCP Registry requires a reverse-DNS namespace** of the form `io.github.<org>/<name>`. The namespace is keyed to an existing GitHub org, so this repo is load-bearing even though it doesn't host code.
3. **Our licensing model is tenant-scoped with ECDSA-signed bundles.** Publishing the implementation would hand an attacker most of the machinery they'd need to mint licenses; it would also invite the fork-and-drift anti-pattern we're actively trying to avoid in the Bitrix24 integration space.

## How to file an issue

- **Bug report** or **feature request** — use the [issue templates](./.github/ISSUE_TEMPLATE).
- **Security finding** — do NOT file a public issue. Read [`SECURITY.md`](./SECURITY.md) for the coordinated-disclosure channel.
- **Commercial / licensing questions** — email `support@rcs.kz`.

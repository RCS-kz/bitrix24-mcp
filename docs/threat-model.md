# Threat model & attack harness

This document is the **public summary** of the threat model that gates every release of bitrix24-mcp. The detailed per-attack report lives in-tree in the proprietary build, under `tests/phase4-7/ATTACK-REPORT.md`; we keep it in sync with the summary below at each release.

## Assumptions

We explicitly design against the following operator-realistic threat profile:

1. **The user's workstation is not trusted to be malware-free.** The user installs the product, but commodity malware (clipboard hijackers, Keychain-scraping info-stealers) may be running concurrently. We protect against leaking *new* secrets via our own logs or config files, but we assume that a malware-rooted box is already lost.
2. **The `.mcpb` / npm package is a public artifact.** Any customer, competitor, or adversary can download it. In-process anti-tamper only needs to raise the cost of re-bake attacks, not prevent them outright. The license server is the authoritative enforcement layer.
3. **Network egress is observable.** Customers' Ops teams should be able to see exactly where we talk to (customer's Bitrix24 tenant + `license.rcs.kz` and nowhere else) and verify that in their firewall logs.
4. **Bitrix24 tenant credentials are high-value.** We never cache CRM data locally. We store only the webhook URL / OAuth tokens, in the OS keychain, with no sync to cloud drives.
5. **The Claude Desktop client is trusted to honour the `destructive: true` manifest flag.** We ship with destructive tools clearly marked; a client that ignores the flag is a client-side issue we escalate upstream rather than defend against locally.

## Attack harness

We run a 10-attack regression suite in CI for every release. The harness lives in the proprietary source tree (`tests/phase4-7/`) and is **summarised here**:

| # | Vector | In-process defence | Out-of-process defence | Outcome |
|---|---|---|---|---|
| 00 | Baseline (no modification) | — | — | PASS |
| 01 | Zero a single integrity sentinel | ❌ Detected — `tag_inconsistent` | — | **caught** |
| 02 | Zero all integrity sentinels (consistent zeros) | ⚠️ Treated as dev-placeholder | License server rejects `bundle_hash=null` | **caught by server** |
| 03 | Flip one byte in the V8 bytecode | ❌ Detected — `hash_mismatch` | — | **caught** |
| 04 | Mutate the loader outside sentinels (e.g. add a comment) | ❌ Detected — `hash_mismatch` | — | **caught** |
| 05 | Re-bake after any mutation (hash recomputed + re-embedded) | — (mathematically bypasses in-proc) | ECDSA signature fails + bundle-hash whitelist rejects | **caught** |
| 06 | `BITRIX24_SKIP_INTEGRITY=1` env flag | ⚠️ Dev-override honoured | License server call-home still happens | **caught** |
| 07 | `fs.readFile` patch via `--require` preload | — (tamper hidden from verifier) | License bundle-hash whitelist rejects modified build | **caught** |
| 08 | DNS redirect of `license.rcs.kz` | — | ECDSA verify rejects foreign-signed responses | **caught** |
| 09 | Decompile V8 bytecode | — | Strings recoverable; control flow is not trivial to restore | **partial leak (accepted)** |

### What "caught" means

A build that has been tampered with in any of attacks 01–08 will, by the time Claude actually invokes a Bitrix24 tool, fail the license check and refuse to proceed. In attacks that bypass the in-process check (05, 06, 07), the license-server's `bundle_hash` whitelist is the authoritative gate — the license server never issued a license for the modified bundle's hash, so verification returns `revoked`.

### Attack 02 — our known limit

Zeroing all integrity sentinels to a consistent value is treated as a "dev-placeholder" build in the current release. Defence-in-depth catches it (the license server rejects a null `bundle_hash`), but a future minor release will tighten the in-process check as well: dev-placeholder will require an explicit `BITRIX24_DEV_BUILD=1` opt-in rather than being inferred from all-zero sentinels. Tracked; non-blocking for 1.0.

### Attack 09 — accepted partial leak

V8 bytecode is not a security boundary against string extraction — UTF-8 literals (error messages, tool names, JSON-schema keys) are recoverable with any bytecode inspection tool. We accept this: the literals are the same ones a customer could read from tool definitions in any MCP client. What is *not* recoverable from bytecode alone is the source-level control flow, which is the actual differentiator.

## What we don't claim to defend against

- **Full root on the customer's machine.** An attacker with root can hook `openat()`, patch libcrypto, or simply invoke Bitrix24's REST API directly with the customer's token. We're the wrong layer of defence for this. Ops teams should manage endpoint security separately.
- **Social engineering of the customer into running a malicious "patched" build.** We publish checksums and signatures at <https://rcs.kz/bitrix24-mcp>; customers who install from other sources assume that risk.
- **Denial of service against the customer's Bitrix24 tenant.** Our rate limiter prevents *us* from DDoS-ing the tenant, but a customer deliberately misusing Claude prompts to trigger excessive load is a customer-side policy matter.
- **Exfiltration of CRM data via Claude's context.** Claude sees only what the tool returns. If a customer's prompt causes Claude to surface sensitive CRM data in chat, that is a DLP question about how Claude Desktop / Claude enterprise policies are configured — not a bitrix24-mcp issue.

## Reporting findings

If you believe you have a finding that our harness misses, please follow [`../SECURITY.md`](../SECURITY.md). Do **not** open a public issue.

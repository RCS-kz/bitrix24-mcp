# Security Policy

We take security findings seriously. Our goal is to acknowledge reports within **24 hours** and ship a fix within **7 calendar days** for confirmed high-severity issues.

## Supported versions

| Version | Status | Security fixes |
|---|---|---|
| 1.x | active | yes |
| < 1.0 | pre-release | no |

Once 1.0 ships, we will support the most recent two minor releases (e.g., 1.4 and 1.5) with security backports. Earlier minors will be told to upgrade.

## Reporting a vulnerability

**Do NOT** open a public GitHub issue or pull request for security findings. Instead:

1. **Email** `security@rcs.kz` with subject prefix `[SECURITY]`.
2. **Encrypt** the body with our PGP key (fingerprint below).
3. Include — at minimum:
   - Affected version(s) and platform(s).
   - Reproducer (proof-of-concept code, exact tool invocation, or trace).
   - Your assessment of impact (confidentiality, integrity, availability).
   - Whether you intend to disclose publicly, and on what timeline.

We will acknowledge within **24 hours** (Mon–Fri, Asia/Almaty UTC+5; weekends best-effort) and provide a tracking ID.

### PGP key

A copy of our PGP public key is available at:

- <https://rcs.kz/.well-known/security.pgp> (HTTPS, served from the same origin as our landing)
- The `keys.openpgp.org` keyserver, indexed under `security@rcs.kz`

Key fingerprint will be published here once the production landing is live (Sprint S3). Until then, please request the key in your initial email and we will reply with it inline before you send the reproducer.

### Coordinated disclosure timeline

| Day | Milestone |
|---|---|
| 0 | Initial report received |
| 0–1 | Acknowledgement + tracking ID issued |
| 1–3 | Triage + severity classification (CVSS v3.1) |
| 3–7 | Fix developed + tested against the 10-attack harness |
| 7 | Patch release published; affected customers notified via `support@rcs.kz` distribution list |
| 7+30 | Public advisory published if scope warrants (CVSS ≥ 7.0 by default) |

We coordinate disclosure with the reporter. If you have a hard public-disclosure deadline, tell us at first contact and we will confirm whether 7 days is achievable; if not, we will negotiate.

### Recognition

By default we credit reporters by name (or pseudonym) in the advisory and the changelog. Tell us if you'd prefer to remain anonymous.

We do not currently run a paid bounty programme. We do send physical thank-you items (laptop sticker pack + Almaty-themed coffee) to reporters of confirmed valid findings — opt-in by sending us a shipping address after the fix ships.

## In-scope

- The `@rcs-kz/bitrix24-mcp` npm package and the `.mcpb` bundle distributed via <https://rcs.kz/bitrix24-mcp>.
- The license verification path against `license.rcs.kz`.
- The trial-issuance flow at <https://rcs.kz/bitrix24-mcp> (including any signed-token URLs and email links).
- Any documentation in this repository that, if relied upon, could induce insecure configuration.

## Out of scope

- Self-inflicted misconfigurations (e.g., publishing your Bitrix24 webhook URL on a public site).
- Findings that require physical access to an unlocked, attended workstation.
- Issues in third-party software we depend on (Node.js, Claude Desktop, Bitrix24 itself) — please report those upstream. We are happy to relay if the upstream channel is unclear.
- Social-engineering attacks against RCS staff.
- Brute-force or rate-limit-exhaustion attempts against our public endpoints. Our rate-limit response IS our security control here; demonstrating that you can hit a 429 is not a finding.

## Threat model

A summary of our threat model and the 10-attack harness used in CI for every release is published in [`docs/threat-model.md`](./docs/threat-model.md). Reading it before submitting a finding will save us both time.

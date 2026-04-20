---
name: Bug report
about: Report something that doesn't work the way the docs say it should.
title: "[BUG] "
labels: bug
assignees: ''
---

<!--
  SECURITY FINDINGS: do NOT open a public issue. Read SECURITY.md and email
  security@rcs.kz instead.
-->

## What happened

A one-paragraph description of the behaviour you observed.

## What you expected

What should have happened if the docs and the behaviour agreed.

## Reproducer

- Exact tool invocation or Claude prompt that triggered it
- Any relevant input data (redacted; do NOT include real customer data)
- Screenshots / transcript snippets if UI-side

## Environment

- bitrix24-mcp version (`bitrix24-mcp --version` or the `.mcpb` filename):
- Install path (landing `.mcpb` / `npm install -g` / MCP Registry):
- Claude Desktop version:
- OS + version (macOS 14.4, Windows 11 23H2, Ubuntu 22.04, …):
- Bitrix24 tenant region (`.kz` / `.ru` / `.com`):

## Logs

Paste the relevant lines from `~/.bitrix24-mcp/audit.log`. If the issue is about secrets or PII leaking into logs — stop, read [`SECURITY.md`](../../SECURITY.md), and email `security@rcs.kz` instead.

```
<paste here>
```

## Anything else

Things you've already tried, things you suspect, links to related issues.

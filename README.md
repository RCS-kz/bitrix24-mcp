# bitrix24-mcp

> **Production-grade MCP server for Bitrix24 Cloud — 45 tools, safe by default.**
>
> Профессиональный MCP-сервер для Bitrix24 — 45 инструментов, безопасно по умолчанию.

[Русский](#-русский)  ·  [English](#-english)  ·  [Install](#install)  ·  [Docs](https://rcs.kz/bitrix24-mcp/docs)  ·  [Buy](https://rcs.kz/bitrix24-mcp/buy)

> **This repository is the public front of a commercial product.** It contains the README you're reading, install guides, changelogs, the published threat model, and our security-disclosure policy. **Source code is proprietary** and ships as a signed V8-bytecode bundle through the channels listed under [Install](#install). See [`NOTICE.md`](./NOTICE.md) for the rationale.

---

## 🇷🇺 Русский

### Что это

**bitrix24-mcp** — сервер Model Context Protocol, подключающий Claude Desktop к вашему облачному Bitrix24. Один раз устанавливаете, авторизуете через входящий вебхук Bitrix24 (или OAuth-приложение Bitrix24.Market) — Claude получает **45 инструментов** по CRM (контакты, компании, сделки, лиды, товары), Задачам, Пользователям, Мессенджеру и Календарю.

Дальше Claude делает то, что умеет лучше всего — рассуждает над неструктурированными запросами вроде «покажи все сделки больше 50 млн ₸ с просроченной датой закрытия в этом квартале, самые крупные три — отправь email менеджерам и поставь задачу на follow-up» — и вызывает REST API Bitrix24 от вашего имени, с вашим подтверждением.

### Почему это существует

Типовые «ИИ-интеграции с CRM» оптимизируют под демо. Мы оптимизируем под тот вечер, когда вы показали это CEO и он сказал «отлично, теперь запусти на реальном tenant'е». Именно здесь rate-лимиты, аудит-логи, обработка ошибок записи и гигиена токенов решают, останется продукт в вашем стеке или нет.

### Что внутри

**45 инструментов — каждый:**

- **Типобезопасен** — аргументы валидируются через Zod против строгой JSON-схемы до отправки REST-запроса. Некорректный ввод падает локально с понятной ошибкой, а не мусорным ответом API.
- **Confirm-gated по умолчанию** — деструктивные инструменты (`*_delete`, `*_update` с массовыми селекторами) декларируют `destructive: true` в манифесте. Claude Desktop показывает диалог подтверждения.
- **Ограничен по частоте** — per-tenant, per-user, per-tool. Массовые операции становятся в очередь с backpressure, а не DDoS-ят ваш Bitrix24.
- **Записывается в аудит-лог** — каждый вызов — JSON-строка в `~/.bitrix24-mcp/audit.log` (timestamp, инструмент, пользователь, rate-limit remaining, длительность, результат). Локально, права `-rw-------`.

**Покрытие:** CRM (контакты / компании / сделки / лиды / товары / активности + cross-type поиск), Задачи (create / assign / stage / comment / attach / time-log), Пользователи (list / get / by-department), IM (chat / DM / recent), Календарь (list / CRUD / respond). Полный справочник — `docs/TOOLS.md` на landing'е после установки.

### Безопасность

- **Webhook / OAuth токены — в keychain ОС** (macOS Keychain, Windows Credential Manager, Linux libsecret через `keytar`). Никогда в конфиг-файле, env или логах.
- **Zero telemetry.** Не собираем метрики использования, stack traces, call patterns. Отключать нечего — ничего и не собирается.
- **Декларированный egress.** Два хоста видны firewall: `*.bitrix24.kz` / `*.bitrix24.ru` / `*.bitrix24.com` (ваш tenant) и `license.rcs.kz` (валидация лицензии раз в 24 ч, fail-open после 14-дневного grace).
- **Целостность бандла.** Продакшен-бандл — V8-bytecode с SHA-256 fingerprint'ом, проверяемым из 5 независимых check-points. Попытка модификации даёт явный fail.
- **Лицензии на ECDSA P-256.** Подпись над tenant-scoped bundle hash + expiry. Украденная лицензия не переносится на другой tenant, срок без приватного ключа RCS не продлевается.

Модель угроз и 10-attack harness — [`docs/threat-model.md`](./docs/threat-model.md) (synced с `tests/phase4-7/ATTACK-REPORT.md` каждый релиз).

### Для кого

- **Клиенты Bitrix24** на Claude Pro / Team / Enterprise, которым надоело переключаться между чатом Claude и CRM.
- **Клиенты RCS** (1С:франчайзинг Казахстан) — единый поставщик автоматизаций Bitrix24 поверх уже работающих интеграций 1С.
- **IT / Ops команды**, которым нужно согласовать AI-CRM интеграцию и получить внятные ответы про хранение токенов, аудит и rate-лимиты.

### Чем это НЕ является

- Не чат-виджет в Bitrix24. Claude живёт в Claude Desktop (или любом MCP-совместимом клиенте).
- Не no-code автоматизация. Для визуальных workflow — встроенные автоматизации Bitrix24.
- Не поддерживает on-premise (коробочный) Bitrix24. Только облако. On-prem — в roadmap, без ETA.
- Не бесплатный продукт. Мы берём деньги, потому что обновляемся, ведём поддержку и реагируем на изменения REST API Bitrix24 в течение недели. Заброшенный бесплатный инструмент хуже, чем никакой.

### Цена

- **299 USD / tenant Bitrix24 / год.** Оплата USD, KZT или RUB (ЮKassa / CloudPayments по курсу дня).
- **14 дней trial**, без карты, без урезанных функций.
- Per-tenant — неограниченное число пользователей внутри tenant'а.
- Включает обновления в рамках текущей мажорной версии (1.x).
- Поддержка: `support@rcs.kz`, SLA 1 рабочий день (Almaty, UTC+5).

### Требования

- Claude Desktop 0.8.0+ (или любой MCP-совместимый клиент)
- macOS 13+, Windows 10+, либо Linux с `libsecret`
- Bitrix24 Cloud tenant с правами администратора (для создания входящего вебхука)
- Node.js 20 LTS или 22 LTS (только для ручной npm-установки, путь для продвинутых)

### Установка

**Рекомендуется:** скачать `.mcpb`-бандл с [`rcs.kz/bitrix24-mcp`](https://rcs.kz/bitrix24-mcp) и установить в Claude Desktop одним кликом. Пошаговая инструкция — [`docs/install/claude-desktop.md`](./docs/install/claude-desktop.md).

**Для продвинутых:**

```bash
npm install -g @rcs-kz/bitrix24-mcp
```

Конфиг в `claude_desktop_config.json` описан в install-гайде.

### Поддержка и SLA

- Ответ в течение 1 рабочего дня (Пн–Пт, 09:00–18:00 Asia/Almaty, UTC+5).
- Патч-релизы в течение 5 рабочих дней после воспроизведения бага.
- Безопасность: `security@rcs.kz` + PGP-ключ опубликован в [`SECURITY.md`](./SECURITY.md). Ответ в течение 24 часов, фикс в 7 дней, кредит репортеру (если не просил иное).

### Вендор

**RCS** (1С:франчайзинг Казахстан) — алматинский партнёр 1С с 10+ годами работы по enterprise ERP-интеграциям в Казахстане, России, Узбекистане.

- Сайт: <https://rcs.kz>
- Email: `support@rcs.kz`
- GitHub: <https://github.com/rcs-kz/bitrix24-mcp> (публичный README + docs; исходники проприетарны)

---

## 🇬🇧 English

### What it is

**bitrix24-mcp** is a Model Context Protocol server that connects Claude Desktop to a Bitrix24 Cloud tenant. Install once, authorise against a Bitrix24 inbound webhook (or Bitrix24.Market OAuth app) — Claude gains **45 tools** spanning CRM (contacts, companies, deals, leads, products), Tasks, Users, Instant Messaging, and Calendar.

Claude then does what Claude does best — reasoning over unstructured requests like *"Summarise every deal over 50 M KZT that slipped its close date this quarter, email the three biggest to the account owner, and create a follow-up task for each"* — and calls the Bitrix24 REST API on your behalf, with your approval.

### Why it exists

Generic "connect AI to CRM" tools optimise for demo-ability. We optimise for the evening after the demo, when rate limits, audit trails, failed-write recovery, and auth-token hygiene decide whether the product stays in your stack.

### What's in the box

**45 tools, all of them:**

- **Type-safe** — every argument validated with Zod against a strict JSON schema before the REST call leaves the machine.
- **Confirm-gated by default** — destructive tools (`*_delete`, `*_update` with mass-selectors) declare `destructive: true` in the manifest. Compliant clients surface a confirmation dialog.
- **Rate-limited** — per-tenant, per-user, per-tool. Bulk imports queue with backpressure instead of DDoS-ing your own Bitrix24.
- **Audit-logged** — every call writes a JSON line to `~/.bitrix24-mcp/audit.log` with timestamp, tool, caller, rate-limit remaining, duration, result. User-readable only.

**Coverage:** CRM (contacts, companies, deals, leads, products, activities + cross-type search), Tasks (create, assign, move stages, comment, attach files, log time), Users (list, get, by-department), IM (chat, DM, recent), Calendar (list, CRUD, respond). Exhaustive reference: `docs/TOOLS.md` on the landing page after install.

### Security posture

- **Webhook URL / OAuth tokens in the OS keychain** (Keychain / Credential Manager / libsecret via `keytar`). Never in a config file, never in env vars, never echoed in logs.
- **No telemetry.** We do not collect usage metrics, crash reports, or tool-call patterns.
- **Declared network egress.** Two hosts, both visible to your firewall: `*.bitrix24.{kz,ru,com}` (your tenant) and `license.rcs.kz` (24 h license check, fail-open after a 14-day grace period).
- **Bundle integrity.** Production bundle ships as V8 bytecode with SHA-256 fingerprint verified at load time from five independent check-sites.
- **ECDSA-signed licenses.** P-256 signature over tenant-scoped bundle hash + expiry. Leaked licenses cannot be retargeted to another tenant.

Full threat model and 10-attack harness: [`docs/threat-model.md`](./docs/threat-model.md).

### Who it's for

- Bitrix24 customers on Claude Pro / Team / Enterprise who want to stop context-switching between the chat window and the CRM.
- RCS clients (existing 1С franchise portfolio in Kazakhstan) who want a single trusted vendor for Bitrix24 automation on top of their existing 1С integrations.
- Ops / IT teams who need to approve an AI-CRM integration and will ask pointed questions about token storage, audit, and rate limiting.

### What it is *not*

- Not a chat widget embedded inside Bitrix24. Claude lives in Claude Desktop (or any MCP-compatible client).
- Not a no-code automation tool.
- Not an on-premise Bitrix24 connector. Cloud tenants only.
- Not a free tool.

### Pricing

- **299 USD / Bitrix24 tenant / year.**
- **14-day free trial**, no credit card, no features gated.
- Per-tenant — unlimited users in that tenant.
- Includes updates for the licensed major version (1.x).
- Support via `support@rcs.kz`, 1 business day SLA (Almaty time, UTC+5).

### Requirements

- Claude Desktop 0.8.0+ (or any MCP-compliant client)
- macOS 13+, Windows 10+, or Linux with `libsecret`
- Bitrix24 Cloud tenant with admin rights to create an inbound webhook

### Install

Recommended: download the `.mcpb` bundle from [`rcs.kz/bitrix24-mcp`](https://rcs.kz/bitrix24-mcp) — one-click install in Claude Desktop. Step-by-step: [`docs/install/claude-desktop.md`](./docs/install/claude-desktop.md).

Advanced:

```bash
npm install -g @rcs-kz/bitrix24-mcp
```

### Support & SLA

- Response within 1 business day (Mon–Fri, 09:00–18:00 Asia/Almaty, UTC+5).
- Patch releases within 5 business days of confirmed reproduction.
- Security: `security@rcs.kz` + PGP key fingerprint in [`SECURITY.md`](./SECURITY.md). 24 h ack, 7-day fix, reporter credited unless they ask otherwise.

### Vendor

Built and maintained by **RCS** (1С:франчайзинг Казахстан) — Almaty-based 1С franchise with 10+ years of enterprise ERP integration work across Kazakhstan, Russia, and Uzbekistan.

- Website: <https://rcs.kz>
- Email: `support@rcs.kz`
- GitHub: <https://github.com/rcs-kz/bitrix24-mcp> (public README + docs; source proprietary)

---

## Links

- **Documentation:** <https://rcs.kz/bitrix24-mcp/docs>
- **Landing / install:** <https://rcs.kz/bitrix24-mcp>
- **Changelog:** [`CHANGELOG.md`](./CHANGELOG.md)
- **Threat model:** [`docs/threat-model.md`](./docs/threat-model.md)
- **Privacy & EULA:** <https://rcs.kz/bitrix24-mcp/privacy> · <https://rcs.kz/bitrix24-mcp/eula>
- **npm:** <https://www.npmjs.com/package/@rcs-kz/bitrix24-mcp>
- **MCP Registry:** `io.github.rcs-kz/bitrix24-mcp`

## License

This repository is licensed under [CC BY-ND 4.0](./LICENSE) (documentation) — see [`LICENSE`](./LICENSE). The **software itself** (the npm package, `.mcpb` bundle, and all related artifacts) is governed by the [Commercial EULA](https://rcs.kz/bitrix24-mcp/eula); source is proprietary.

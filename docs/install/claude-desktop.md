# Установка bitrix24-mcp в Claude Desktop

**Для кого:** пользователи Bitrix24 Cloud, которые хотят управлять CRM, задачами и коммуникациями из Claude Desktop (macOS / Windows).

**Что получите:** 45 tools поверх вашего Bitrix24 Cloud — Claude сможет читать сделки, создавать контакты, выставлять задачи, писать в IM и смотреть календарь. Все необратимые действия закрыты confirm-гейтом, санитизация пользовательского ввода, rate-limiter с поддержкой X-RateLimit-Retry-Tables.

**Стоимость:** $299 / tenant / год. 14 дней бесплатный trial — без банковской карты. Купить и получить ключ: [rcs.kz/bitrix24-mcp/buy](https://rcs.kz/bitrix24-mcp/buy).

---

## Что нужно до установки

1. **Claude Desktop** установлен (macOS 12+ или Windows 10+). Скачать: [claude.ai/download](https://claude.ai/download).
2. **Bitrix24 Cloud tenant** с правами администратора (нужно создать входящий вебхук).
3. **License key** от `rcs.kz/bitrix24-mcp/buy` — придёт на email сразу после оформления. На trial — ключ активен 14 дней, без call-home ограничений.
4. **Bitrix24 webhook URL**. Получить так: в своём портале Bitrix24 — `Разработчикам` → `Другое` → `Входящий вебхук` → `Создать вебхук` с правами **`crm, task, im, calendar, user, department`**. Скопировать полный URL вида `https://your-portal.bitrix24.kz/rest/1/abcXYZ123xyz/`.

---

## Путь 1 — установка через `.mcpb` (рекомендуется)

Это одно-кликовая установка через формат MCP Bundle. Подходит для 95% пользователей.

1. Скачайте `bitrix24-mcp-1.0.0.mcpb` с [rcs.kz/bitrix24-mcp/download](https://rcs.kz/bitrix24-mcp/download).
2. Откройте Claude Desktop → `Settings` → `Extensions` → `Install Extension` → укажите скачанный `.mcpb`. На macOS файл можно открыть двойным кликом — Claude Desktop подхватит его автоматически.
3. В диалоге конфигурации введите:
   - **Bitrix24 webhook URL** — из шага 4 выше (поле помечено как sensitive, хранится в OS keychain).
   - **Log level** — оставьте `info`. Менять на `debug` только если нужен детальный трейс для поддержки.
   - **License key** — вставьте ключ из письма от `rcs.kz`.
4. Нажмите `Install`. Claude перезапустит MCP-процесс.
5. Проверьте, что tools появились: в чате наберите `/` — должны появиться `bitrix24_app_info`, `bitrix24_deal_list` и ещё 43.

**Проверка что установка живая:** в чате спросите Claude: «Покажи 5 последних сделок из Bitrix24». Claude должен позвать tool `bitrix24_deal_list` и вернуть таблицу.

---

## Путь 2 — ручная конфигурация через `claude_desktop_config.json`

Для продвинутых пользователей, которые предпочитают управлять конфигом вручную (например, для нескольких tenants). Нужен установленный **Node.js 20+**.

1. Установите пакет глобально:
   ```
   npm install -g @rcs-kz/bitrix24-mcp
   ```
2. Откройте конфиг-файл Claude Desktop:
   - **macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
   - **Windows:** `%APPDATA%\Claude\claude_desktop_config.json`
3. Добавьте в `mcpServers`:
   ```json
   {
     "mcpServers": {
       "bitrix24": {
         "command": "npx",
         "args": ["-y", "@rcs-kz/bitrix24-mcp"],
         "env": {
           "BITRIX24_AUTH_MODE": "webhook",
           "BITRIX24_WEBHOOK_URL": "https://your-portal.bitrix24.kz/rest/1/abcXYZ123xyz/",
           "BITRIX24_LICENSE_KEY": "rcs_live_...",
           "BITRIX24_LOG_LEVEL": "info"
         }
       }
     }
   }
   ```
   Подставьте свой webhook URL из шага 4 раздела «Что нужно до установки» и license key из письма.
4. **Полностью закройте Claude Desktop** (`Cmd+Q` на macOS или выход из трея на Windows — перезагрузка окна недостаточна, нужно завершить процесс).
5. Запустите Claude Desktop заново. Tools появятся автоматически.

**Если не появились:** откройте `Settings` → `Developer` → `Open logs folder`, найдите `mcp-server-bitrix24.log` — он покажет причину. Чаще всего: неверный формат webhook URL (не должен содержать пробелов или кавычек), истёк license key, или `node` не в `PATH`.

---

## Путь 3 — через «Add custom connector» в claude.ai (для web)

**Это НЕ основной путь.** В Claude Desktop работают пути 1 и 2. «Add custom connector» в web-версии claude.ai принимает только remote-MCP (HTTP/SSE), а bitrix24-mcp — stdio-local. Если вам нужен hosted-вариант, напишите на `support@rcs.kz` — это roadmap-feature, не GA.

---

## Типичные проблемы

- **«401 Unauthorized» при первом вызове.** Webhook URL неверен или истёк. Перевыпустите вебхук в Bitrix24 с правами `crm, task, im, calendar, user, department`.
- **«License expired».** Обновите подписку на `rcs.kz/bitrix24-mcp/billing`. В offline-режиме grace period 14 дней до блокировки tools.
- **Tools не появляются после установки `.mcpb`.** Проверьте, что Claude Desktop именно **перезапущен** (`Cmd+Q`, не окно закрыто). На Windows — убедитесь, что `Claude.exe` вышел из трея.
- **Rate limit hit.** Мы соблюдаем Bitrix24 ограничения автоматически через `X-RateLimit-Retry-Tables`. Если видите ошибку — откройте в логах `mcp-server-bitrix24.log` секцию `rate-limiter` — бывает, что ваш tenant на `free` плане имеет жёсткий лимит 2 qps.

---

## Безопасность

- **Webhook URL хранится в OS keychain.** На macOS — Keychain Access, на Windows — Credential Manager. Не передаётся в логах, не попадает в историю Claude.
- **Confirm-гейт на необратимых tools.** `bitrix24_deal_delete`, `bitrix24_task_delete`, `bitrix24_rest_raw` — всегда требуют `confirm: true` в arguments. Без него Claude получит понятную ошибку и попросит подтверждение у вас.
- **Санитизация пользовательского ввода.** Всё, что Claude получает из Bitrix24 (комментарии в deals, описания задач), пропускается через `wrapUserContent` перед попаданием в ответ — защита от prompt-injection.
- **Zero-telemetry.** Единственное network-взаимодействие кроме Bitrix24 API — онлайн-валидация license key раз в 24 часа к `license.rcs.kz` (offline grace period 14 дней). Если нужен air-gapped режим — напишите на `support@rcs.kz`.

---

## Поддержка

- Email: `support@rcs.kz` (SLA: ответ в течение 1 рабочего дня; P1 — 2 часа)
- Docs: [rcs.kz/bitrix24-mcp/docs](https://rcs.kz/bitrix24-mcp/docs)
- Пакет в npm: [@rcs-kz/bitrix24-mcp](https://www.npmjs.com/package/@rcs-kz/bitrix24-mcp) (публичный метадата-листинг; install с license key).
- MCP Registry: `io.github.rcs-kz/bitrix24-mcp` на [modelcontextprotocol.io](https://modelcontextprotocol.io).

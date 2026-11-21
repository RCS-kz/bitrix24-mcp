# bitrix24-mcp — MCP-сервер для Битрикс24

[Русский](#русский) · [English](#english) · [rcs.kz/product/mcp-bitrix24](https://rcs.kz/product/mcp-bitrix24) · [Облачный коннектор](https://rcs.kz/product/oblachnyj-mcp)

> Это публичная витрина коммерческого продукта: README, инструкции, журнал изменений, модель угроз. Исходный код закрыт; сборки выдаём по заявке.

## Русский

**Что это.** bitrix24-mcp подключает Claude к облачному Битрикс24: ИИ работает с лидами, сделками, задачами, чатами, открытыми линиями и статистикой звонков, а удаление и массовые изменения делает только после вашего подтверждения.

### Два способа подключения

| | Облачный коннектор | Локальная версия |
|---|---|---|
| Где работает | claude.ai в браузере, приложение Claude на телефоне, Claude Desktop | Claude Desktop на одном компьютере |
| Что поставить | ничего: коннектор добавляется в Claude по адресу | файл `.mcpb` |
| Доступ к Битрикс24 | каждый сотрудник входит своим логином Битрикс24 и видит только то, что ему разрешено в портале | один входящий вебхук администратора |
| Обновления | мы, на своём сервере | вы |
| Цена | та же, за облако доплаты нет | |

**Облако — 3 шага:**
1. Оставляете [заявку](https://rcs.kz/product/oblachnyj-mcp) — мы заводим ваш портал и присылаем инструкцию.
2. Администратор Битрикс24 ставит приложение в портале и вставляет его ключи по одноразовой ссылке.
3. Коннектор один раз добавляется в Claude по выданному адресу вида `https://b24-mcp.rcs.kz/t/<код-портала>/mcp` (в Claude Team это делает владелец организации — сразу для всех); каждый сотрудник нажимает «Подключить» и входит своим логином Битрикс24.

Claude Code: `claude mcp add --transport http bitrix24 https://b24-mcp.rcs.kz/t/<код-портала>/mcp`

**Локально — 3 шага:**
1. Оставляете [заявку](https://rcs.kz/product/mcp-bitrix24) — присылаем файл `.mcpb` и ключ на пробный период.
2. В Битрикс24 под администратором создаёте входящий вебхук с правами `crm, task, calendar, im, imopenlines, telephony, user, department`.
3. Открываете `.mcpb` в Claude Desktop — форма спросит адрес вебхука и ключ. Node.js ставить не нужно.

### Что можно поручить

| Кто | Запрос |
|---|---|
| Руководитель отдела продаж | «Сколько звонков сделал и пропустил каждый менеджер на этой неделе? У кого больше всего пропущенных?» |
| РОП | «Покажи переписку в открытых линиях по сделке … и сколько клиент ждал первого ответа» |
| РОП | «Почему зависли сделки на стадии "КП отправлено" больше 14 дней? Что менеджеры писали в таймлайне?» |
| Директор | «Откуда на самом деле приходят продажи? Разбери лиды с 1 августа по каналам» |
| Менеджер | «Создай лид с email …, свяжи с компанией …, поставь задачу ответственному на завтра» — Claude попросит подтверждение |
| Руководитель | «Собери сводку по воронке за неделю и отправь её в чат отдела продаж» |

### Что внутри

- **CRM:** лиды, сделки, контакты, компании, смарт-процессы, дела, комментарии таймлайна, воронки и стадии.
- **Звонки и переписка:** статистика телефонии, дела-звонки и расшифровки, открытые линии (WhatsApp, Telegram, чат на сайте — что подключено в портале), чаты и уведомления.
- **Задачи, календарь, сотрудники и отделы.**
- Локальная версия 2.3.0 — 61 инструмент. В облаке — 64 инструмента Битрикс24 (плюс Диск, рабочие группы и Новости — только чтение); пакетный вызов и произвольные методы REST в облаке выключены.
- В облаке администратор может дать сотруднику узкий профиль инструментов (CRM, задачи, связь, только реклама) — меньше лишнего в контексте модели. Права в Битрикс24 это не расширяет.

### Безопасность

- Удаление и массовые изменения — только с подтверждением.
- Облако: сотрудник работает со своими правами Битрикс24; отключили его в портале — доступ из Claude пропадает на следующем запросе. Содержимое CRM на диск нашего сервера не записывается: запросы идут к вашему порталу от имени сотрудника.
- Локально: вебхук и ключ вводятся в форму Claude Desktop (поля помечены как секретные); лицензия проверяется на license.rcs.kz.
- Работает только с облачным Битрикс24 (коробочная версия не поддерживается).
- Модель угроз — [`docs/threat-model.md`](./docs/threat-model.md), уязвимости — [`SECURITY.md`](./SECURITY.md).

### Чем отличаемся (честно)

**Официальный MCP Битрикс24.** У вендора есть свой MCP-сервер для управления порталом из внешних ИИ-систем (ChatGPT, Yandex AI Studio, Cursor, n8n и др.): действия по команде сотрудника и с учётом его прав. По справке Битрикс24 (обновлена 12.10.2026) внешние интеграции по MCP доступны по подписке «BitrixGPT + Маркетплейс»; внутри портала агент BitrixGPT работает с инструментами бесплатно в рамках базового лимита. Отдельный b24-dev-mcp отдаёт только документацию REST API для разработчиков и к данным портала доступа не имеет.
Если у вас уже есть эта подписка и нужны CRM и задачи — начните с официального сервера: его поддерживает сам вендор.

Наш коннектор выбирают, когда нужно:
- **контроль отдела продаж**: звонки, расшифровки, открытые линии, время первого ответа, сводка в чат — в одном диалоге;
- **связка с рекламой и 1С**: Яндекс Директ, VK Реклама и Roistat в том же облачном коннекторе ([rcs-ads-mcp](https://rcs.kz/product/rcs-ads-mcp)), учёт — через [mcp-1c](https://github.com/RCS-kz/mcp-1c);
- **внедрение и поддержка партнёром** в Казахстане, договор и оплата в тенге.

Состав инструментов официального сервера построчно мы не сравнивали — перед выбором сверьте его список с нашим.

### Цены

| Пакет | Казахстан, с НДС 16 % | РФ, по инвойсу |
|---|---|---|
| **РОП-пакет (Битрикс24)** | 29 900 ₸/мес | 9 900 ₽/мес |
| **Всё вместе** (Битрикс24 + реклама + 1С) | 49 900 ₸/мес | 16 900 ₽/мес |

Цена облачной и локальной версии одинаковая. Бесплатного тарифа нет, пробный период — по заявке. Для юрлица РФ покупатель — налоговый агент по НДС 22 %. Подробно — [rcs.kz/product/mcp-bitrix24](https://rcs.kz/product/mcp-bitrix24).

### Поддержка
go@rcs.kz — ответ в течение рабочего дня (Алматы, UTC+5). RCS (ИП КВАНТ, Астана) — партнёр фирмы «1С» в Казахстане с 2009 года.

---

## English

**What it is.** bitrix24-mcp connects Claude to Bitrix24 Cloud: the AI works with leads, deals, tasks, chats, open lines and call statistics; deletes and bulk changes run only after you confirm.

### Two ways to connect

- **Hosted connector** — works in claude.ai (browser), the Claude mobile app and Claude Desktop; nothing to install. Each employee signs in with their own Bitrix24 login (OAuth) and sees only what the portal allows them. Steps: request access at [rcs.kz/product/oblachnyj-mcp](https://rcs.kz/product/oblachnyj-mcp) → your Bitrix24 admin installs the app → add the connector URL `https://b24-mcp.rcs.kz/t/<portal-code>/mcp` to Claude once (in Claude Team the owner does it for everyone) → each employee clicks Connect.
- **Local `.mcpb`** for Claude Desktop — request the file and a trial key, create an inbound webhook (`crm, task, calendar, im, imopenlines, telephony, user, department`), open the `.mcpb` in Claude Desktop.

Claude Code [verify]: `claude mcp add --transport http bitrix24 https://b24-mcp.rcs.kz/t/<portal-code>/mcp`

### What you can ask

- "How many calls did each sales rep make and miss this week?"
- "Show the open-line conversation for deal #… and the first-response time."
- "Which deals have been stuck in 'Proposal sent' for 14+ days, and what did reps write in the timeline?"
- "Create a lead with email …, link it to company …, assign a follow-up task for tomorrow." — asks for confirmation.
- "Summarise this week's pipeline and post it to the sales team chat."

Local 2.3.0: 61 tools; hosted: 64 Bitrix24 tools (Drive, workgroups and Feed read-only; batch and raw REST disabled). Bitrix24 Cloud only.

### How it compares

Bitrix24 has its own official MCP server for external AI systems; per Bitrix24 help (updated 12 Oct 2026) external MCP integrations require the "BitrixGPT + Marketplace" subscription. If you have it and need CRM and tasks, start there. Choose ours for sales-team control (calls, transcripts, open lines, first-response time, reports to chat), for one connector with Yandex Direct / VK Ads / Roistat and 1C, and for partner support in Kazakhstan. We have not compared the official tool list line by line.

### Pricing

Sales-manager package (Bitrix24): 29 900 KZT/month incl. 16% VAT (9 900 RUB by invoice). Everything (Bitrix24 + ads + 1C): 49 900 KZT (16 900 RUB). Same price for hosted and local. No free tier; trial on request. Support: go@rcs.kz.

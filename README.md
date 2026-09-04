# 🔐 Synology Manual ACME

Автоматизированное получение и продление wildcard-сертификатов Let's Encrypt для Synology DSM **без использования DNS API**.

Проект построен вокруг официального Docker-образа `acme.sh` и автоматизирует весь процесс, кроме единственного ручного шага — замены TXT-записей `_acme-challenge` у DNS-провайдера.

> 🚀 Текущая версия: **v1.3.1**

---

## ✨ Возможности

- 🔄 Автоматическая проверка срока действия сертификатов
- 🌐 Wildcard-сертификаты для:
  - `example.com`
  - `*.example.com`
- 🧩 Поддержка нескольких доменов в одном проекте
- 👤 Отдельные DSM-пользователи для каждого домена
- 🗂 Поддержка нескольких ACME-аккаунтов через `ACME_ACCOUNT`
- 📝 Автоматический вывод необходимых TXT-записей
- 🔎 Проверка распространения DNS через публичные resolver
- ⏳ Дополнительная DNS-стабилизация перед проверкой Let's Encrypt
- 🖥 Автоматический deploy сертификата в Synology DSM
- 🔁 Повтор только deploy, если сертификат уже выпущен, но DSM его не принял
- 📲 Telegram-уведомления
- 🤖 Telegram-команды
- 💾 Резервные копии сертификатов перед deploy
- 🧹 Автоматическая очистка старых логов и backup
- 🔍 Опциональная проверка сертификата, который реально отдаёт сервис
- 🛡 Безопасное обновление версии без удаления данных и секретов

---

## 🧠 Как это работает

Основной жизненный цикл:

```text
OK
↓
EXPIRING
↓
DSM preflight
↓
WAITING_DNS
↓
Вы меняете TXT-записи
↓
Перезапуск контейнера
↓
Проверка DNS
↓
DNS stabilization
↓
Let's Encrypt validation
↓
CERT_READY
↓
DSM deploy
↓
OK
```

Если сертификат уже получен, но deploy в DSM завершился ошибкой:

```text
CERT_READY
↓
DSM_DEPLOY_FAILED
↓
Повтор только deploy
```

✅ Новый ACME order при этом **не создаётся**.

---

# 📁 Структура проекта

```text
synology-acme-manual/
├── compose.yaml
├── .env
├── upgrade.sh
├── migrate-v1.2-to-v1.3.sh
├── README.md
├── CHANGELOG.md
├── QUICKSTART-RU.txt
│
├── app/
│   ├── entrypoint.sh
│   └── telegram-worker.sh
│
├── config/
│   ├── domain.example.env
│   └── domains.d/
│
├── data/
├── state/
├── logs/
└── backups/
```

---

# 🚀 Быстрый старт

## 1️⃣ Создайте папку проекта

Например:

```text
/volume1/docker/synology-acme-manual
```

Распакуйте архив проекта в эту папку.

---

## 2️⃣ Создайте конфиг домена

Скопируйте:

```text
config/domain.example.env
```

в:

```text
config/domains.d/01-example.env
```

Пример конфигурации:

```env
DOMAIN='example.com'
EXTRA_DOMAINS=''

ACME_ACCOUNT='default'

SYNO_CERTIFICATE='Wildcard example.com'
SYNO_USERNAME='acme-example'
SYNO_PASSWORD='CHANGE_ME'

SYNO_HOSTNAME='192.168.1.100'
SYNO_SCHEME='auto'
SYNO_PORT='auto'

SYNO_CREATE=''

VERIFY_HOST=''
VERIFY_PORT='443'

ENABLED='1'
```

---

# 👤 Пользователь Synology DSM

Для автоматического импорта сертификата рекомендуется создать отдельного пользователя DSM.

Например:

```text
acme-example
```

Рекомендации:

- ✅ отдельный пользователь для автоматизации;
- ✅ длинный уникальный пароль;
- ✅ член группы `administrators`, если этого требует `synology_dsm`;
- ✅ без лишнего доступа к общим папкам;
- ✅ не использовать личный admin-аккаунт;
- ✅ по возможности ограничить доступ только локальной сетью.

---

# 🪪 Существующий или новый сертификат

Если сертификат **уже существует в DSM**:

```env
SYNO_CREATE=''
```

Если разрешено создать новый:

```env
SYNO_CREATE='1'
```

> ⚠️ Не используйте `SYNO_CREATE='0'`.

Для существующего сертификата имя:

```env
SYNO_CERTIFICATE='Wildcard example.com'
```

должно точно совпадать с названием сертификата в:

```text
Панель управления → Безопасность → Сертификат
```

---

# 🐳 Установка через Container Manager

Откройте:

```text
Container Manager → Проект → Создать
```

Укажите папку проекта и используйте:

```text
compose.yaml
```

Будут созданы два контейнера:

```text
acme-synology
acme-telegram
```

### `acme-synology`

Отвечает за:

- сертификаты;
- DNS challenge;
- проверку срока;
- Let's Encrypt;
- deploy в DSM.

### `acme-telegram`

Отвечает за:

- Telegram-команды;
- ответы бота;
- работу независимо от ACME-процесса.

---

# 🌐 Manual DNS workflow

Когда сертификат пора продлевать, контейнер покажет:

```text
WAITING_DNS: example.com

TXT  _acme-challenge.example.com  =  TOKEN_VALUE_1
TXT  _acme-challenge.example.com  =  TOKEN_VALUE_2
```

Если сертификат включает:

```text
example.com
*.example.com
```

могут потребоваться **две TXT-записи с одинаковым именем**.

Обе должны существовать одновременно.

После изменения DNS:

```text
перезапустите контейнер acme-synology
```

Дальше контейнер сам:

1. 🔎 проверит TXT;
2. ⏳ дождётся DNS;
3. 🕒 выдержит дополнительную паузу стабилизации;
4. 🔐 завершит Let's Encrypt validation;
5. 📥 получит сертификат;
6. 🖥 загрузит его в DSM.

---

# ⏳ DNS stabilization

Настройки находятся в `.env`:

```env
DNS_CHECK_INTERVAL_SECONDS=30
DNS_CHECK_TIMEOUT_SECONDS=1800
DNS_STABILIZATION_SECONDS=600
```

### Что это значит

`DNS_CHECK_INTERVAL_SECONDS=30`

Проверять DNS каждые 30 секунд.

`DNS_CHECK_TIMEOUT_SECONDS=1800`

Ждать распространения DNS максимум 30 минут.

`DNS_STABILIZATION_SECONDS=600`

После того как нужные TXT уже видны, подождать ещё 10 минут перед Let's Encrypt validation.

Это снижает риск ошибки:

```text
Incorrect TXT record
```

у DNS-провайдеров с медленным распространением записей.

---

# 🌍 Несколько доменов

Для каждого домена создайте отдельный файл:

```text
config/domains.d/01-domain-a.env
config/domains.d/02-domain-b.env
config/domains.d/03-domain-c.env
```

У каждого домена могут быть свои:

- DSM user;
- DSM password;
- имя сертификата;
- DSM host;
- ACME account.

---

# 🗂 Несколько ACME-аккаунтов

В каждом конфиге можно указать:

```env
ACME_ACCOUNT='default'
```

Домены с одинаковым значением используют один ACME account.

Например:

```env
ACME_ACCOUNT='account-a'
```

и:

```env
ACME_ACCOUNT='account-b'
```

создадут независимые каталоги:

```text
data/accounts/account-a/account.conf
data/accounts/account-b/account.conf
```

Сертификаты также будут разделены:

```text
data/accounts/account-a/example.com_ecc/
data/accounts/account-b/example.net_ecc/
```

---

# 🧩 Дополнительные SAN и wildcard-зоны

По умолчанию запрашиваются:

```text
example.com
*.example.com
```

Можно добавить:

```env
EXTRA_DOMAINS='*.office.example.com vpn.example.com *.lab.example.com'
```

> ⚠️ Для дополнительных wildcard-зон могут потребоваться дополнительные TXT challenge.

---

# 📲 Telegram

Telegram полностью опционален.

В `.env`:

```env
TELEGRAM_BOT_TOKEN=
TELEGRAM_CHAT_ID=
TELEGRAM_SILENT=0
TELEGRAM_COMMANDS=1
TELEGRAM_POLL_SECONDS=5
```

Если `TELEGRAM_BOT_TOKEN` и `TELEGRAM_CHAT_ID` пустые — Telegram-интеграция просто не используется.

---

## 🔔 Какие уведомления приходят

Бот может сообщить:

- 🔐 нужно изменить TXT;
- ⏳ DNS ещё не распространился;
- ❌ DSM недоступен;
- ❌ ошибка авторизации DSM;
- ✅ сертификат Let's Encrypt получен;
- ⚠️ deploy в DSM не удался;
- ✅ сертификат успешно установлен.

---

# 🤖 Telegram-команды

Команды принимаются только из указанного:

```env
TELEGRAM_CHAT_ID=
```

Доступные команды:

```text
/ping
/version
/status
/domains
/cert <domain>
/dns <domain>
/deploy <domain>
/check
/renew <domain>
/help
```

### Примеры

```text
/status
```

Показывает состояние всех доменов.

```text
/domains
```

Показывает список настроенных доменов.

```text
/cert example.com
```

Показывает дату окончания сертификата.

```text
/dns example.com
```

Повторно показывает текущие TXT challenge.

```text
/deploy example.com
```

Повторяет deploy уже готового сертификата.

```text
/check
```

Запрашивает внеплановую проверку.

```text
/renew example.com
```

Подготавливает новый manual DNS challenge.

> ⚠️ `/renew` не отменяет необходимость ручной замены TXT.

---

# 🧹 Telegram webhook

Telegram worker использует:

```text
getUpdates
```

При запуске контейнер автоматически вызывает:

```text
deleteWebhook
```

чтобы старый webhook не мешал работе команд.

Проверка webhook:

```text
https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getWebhookInfo
```

Удаление:

```text
https://api.telegram.org/bot<YOUR_BOT_TOKEN>/deleteWebhook
```

Удаление вместе со старыми pending updates:

```text
https://api.telegram.org/bot<YOUR_BOT_TOKEN>/deleteWebhook?drop_pending_updates=true
```

После этого поле:

```json
"url": ""
```

должно быть пустым.

---

# 🖥 DSM endpoint auto-detection

Рекомендуемые настройки:

```env
SYNO_SCHEME='auto'
SYNO_PORT='auto'
```

Контейнер проверяет:

```text
HTTPS : 5001
HTTP  : 5000
```

При этом HTTPS используется только если TLS-проверка подходит и для реального deploy-hook.

Если DSM на `5001` доступен только с self-signed или несовпадающим сертификатом, контейнер может использовать локальный fallback:

```text
HTTP : 5000
```

Это помогает избежать ошибок вида:

```text
curl error 60
```

---

# 🔍 Проверка реально установленного сертификата

Опционально:

```env
VERIFY_HOST='nas.example.com'
VERIFY_PORT='443'
```

После успешного deploy контейнер прочитает сертификат, который реально отдаёт указанный сервис, и выведет:

- subject;
- issuer;
- дату окончания.

Если проверка не нужна:

```env
VERIFY_HOST=''
```

---

# 💾 Где хранятся данные

## ACME

```text
data/
```

Содержит:

- `account.conf`;
- private keys;
- сертификаты;
- ACME state.

## Состояние

```text
state/
```

Содержит:

- `WAITING_DNS`;
- `CERT_READY`;
- Telegram offset;
- pending операции.

## Конфигурация доменов

```text
config/domains.d/
```

Содержит:

- домены;
- DSM usernames;
- DSM passwords;
- ACME account mappings.

---

# 🛟 Backup

Перед deploy сертификат копируется в:

```text
backups/<domain>/<timestamp>/
```

Настройки хранения:

```env
LOG_RETENTION_DAYS=90
BACKUP_RETENTION_DAYS=180
```

---

# 🔄 Обновление без потери данных

Рекомендуется всегда использовать одну постоянную рабочую папку:

```text
/volume1/docker/synology-acme-manual
```

Новую версию распаковывать рядом:

```text
/volume1/docker/synology-acme-manual-new
```

Затем выполнить:

```sh
sh /volume1/docker/synology-acme-manual-new/upgrade.sh   /volume1/docker/synology-acme-manual-new   /volume1/docker/synology-acme-manual
```

---

## ✅ Что `upgrade.sh` НЕ удаляет

```text
data/
state/
logs/
backups/
config/domains.d/*.env
.env
```

Также существующие значения `.env` **не перезаписываются**.

Новые параметры добавляются только если их ещё нет.

---

## 🗃 Backup перед обновлением

Перед заменой программных файлов создаётся:

```text
upgrade-backup-YYYYMMDD-HHMMSS/
```

В него копируются старые:

- `compose.yaml`;
- `entrypoint.sh`;
- README;
- шаблоны.

---

# 🔁 Миграция старой структуры data

Старые версии могли хранить:

```text
data/account.conf
data/example.com_ecc/
```

Multi-account версии используют:

```text
data/accounts/default/account.conf
data/accounts/default/example.com_ecc/
```

Для миграции используйте:

```sh
sh migrate-v1.2-to-v1.3.sh /volume1/docker/synology-acme-manual
```

---

# 🛡 Безопасность

Файлы, которые могут содержать секреты:

```text
.env
config/domains.d/*.env
data/
```

Рекомендации:

- 🔒 ограничьте ACL папки проекта;
- 🚫 не публикуйте `.env`;
- 🚫 не публикуйте `config/domains.d/*.env`;
- 🚫 не публикуйте `data/`;
- 🔑 используйте отдельные DSM service accounts;
- 🔐 используйте длинные уникальные пароли;
- 🤖 Telegram bot token считайте секретом.

`.gitignore` уже исключает чувствительные данные.

---

# ⚠️ Ограничение manual DNS-01

Проект намеренно не использует DNS API.

Поэтому при каждом новом выпуске или продлении сертификата значение:

```text
_acme-challenge
```

будет новым.

То есть полностью автоматизировать DNS-01 без механизма автоматического изменения DNS нельзя.

В этом проекте ручным остаётся только один шаг:

```text
получить TXT → изменить TXT у DNS-провайдера
```

Всё остальное выполняется автоматически.

---

# 🧪 Проверка после установки

После запуска:

```text
/ping
```

должен вернуть:

```text
pong ✅
```

Команда:

```text
/version
```

должна показать:

```text
Synology Manual ACME v1.3.1
```

Команда:

```text
/status
```

покажет текущие состояния всех доменов.

---

# 📌 Типичные статусы

| Статус | Значение |
|---|---|
| ✅ `OK` | Сертификат свежий |
| ⚠️ `EXPIRING` | Пора продлевать |
| 🌐 `WAITING_DNS` | Нужно изменить TXT и перезапустить контейнер |
| 🔐 `CERT_READY` | Сертификат уже выпущен, нужен только deploy |
| ❌ `DSM_DEPLOY_FAILED` | Ошибка deploy, новый сертификат не запрашивается |

---

# ❤️ Назначение проекта

Этот проект подходит для пользователей Synology NAS, которые:

- используют wildcard-сертификаты;
- не хотят использовать DNS API;
- готовы вручную менять только TXT challenge;
- хотят убрать ручные команды `acme.sh`;
- хотят автоматический импорт сертификатов в DSM;
- хотят контролировать процесс через Telegram.

---

## 📄 Лицензия

Лицензия в проект по умолчанию не добавлена.

Перед публикацией репозитория рекомендуется выбрать подходящую лицензию, например MIT.

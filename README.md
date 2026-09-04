# 🔐 Synology Manual ACME

Автоматизированное получение и продление wildcard-сертификатов Let's Encrypt для Synology DSM **без DNS API**.

Ручным остаётся только один шаг: заменить TXT-записи `_acme-challenge`. Всё остальное контейнер делает сам.

> Версия: **v1.3.1**

## ✨ Возможности

- wildcard `example.com` + `*.example.com`;
- несколько доменов;
- отдельный DSM-пользователь для каждого домена;
- несколько ACME-аккаунтов через `ACME_ACCOUNT`;
- автоматическая проверка DNS;
- дополнительная DNS-стабилизация;
- автоматический deploy сертификата в DSM;
- повтор только deploy, если сертификат уже получен;
- Telegram-уведомления и команды;
- безопасное обновление без удаления данных.

## 🚀 Установка

Распакуйте проект, например в:

```text
/volume1/docker/synology-acme-manual
```

Скопируйте:

```text
config/domain.example.env
```

в:

```text
config/domains.d/01-example.env
```

Заполните:

```env
DOMAIN='example.com'
ACME_ACCOUNT='default'

SYNO_CERTIFICATE='Wildcard example.com'
SYNO_USERNAME='acme-example'
SYNO_PASSWORD='CHANGE_ME'

SYNO_HOSTNAME='192.168.1.100'
SYNO_SCHEME='auto'
SYNO_PORT='auto'

SYNO_CREATE=''
ENABLED='1'
```

Для существующего сертификата оставьте:

```env
SYNO_CREATE=''
```

Создайте проект через:

```text
Container Manager → Проект → Создать
```

и используйте `compose.yaml`.

## 🌐 Продление сертификата

Когда сертификат пора обновлять, контейнер покажет:

```text
TXT _acme-challenge.example.com = TOKEN_1
TXT _acme-challenge.example.com = TOKEN_2
```

Если значений два — оба должны существовать одновременно.

После изменения DNS перезапустите `acme-synology`.

Дальше контейнер сам:

```text
проверит DNS
→ подождёт стабилизацию
→ получит сертификат
→ установит его в DSM
```

## 📲 Telegram

В `.env`:

```env
TELEGRAM_BOT_TOKEN=
TELEGRAM_CHAT_ID=
TELEGRAM_COMMANDS=1
```

Команды:

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

## 🗂 Несколько ACME-аккаунтов

В каждом домене можно указать:

```env
ACME_ACCOUNT='account-a'
```

Данные будут храниться отдельно:

```text
data/accounts/account-a/
data/accounts/account-b/
```

## 🔄 Обновление

Новую версию распакуйте рядом и выполните:

```sh
sh /path/to/new-version/upgrade.sh   /path/to/new-version   /volume1/docker/synology-acme-manual
```

Не удаляются:

```text
data/
state/
logs/
backups/
config/domains.d/*.env
.env
```

## ⚠️ Важно

Manual DNS-01 требует новый TXT challenge при каждом новом выпуске или продлении сертификата.

То есть полностью автоматическое продление без DNS API невозможно.

## 🛡 Безопасность

Не публикуйте:

```text
.env
config/domains.d/*.env
data/
```

Они могут содержать Telegram token, DSM-пароли, ACME account и private keys.

---

Если проект вам полезен — можете добавить ⭐ на GitHub.

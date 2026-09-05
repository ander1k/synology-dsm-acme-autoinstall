<div align="center">

# 🔐 SynoCert Flow

### Automated Let's Encrypt certificate management for Synology DSM

**DNS API • Manual DNS fallback • DSM deploy • Telegram control • Multi-domain**

![Release](https://img.shields.io/badge/Release-v1.5.0-7C3AED?style=for-the-badge&logo=github&logoColor=white)
![POSIX Shell](https://img.shields.io/badge/POSIX-Shell-F4B400?style=for-the-badge&logo=gnubash&logoColor=111827)
![Docker Compose](https://img.shields.io/badge/Docker-Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)

![Let's Encrypt](https://img.shields.io/badge/Let's%20Encrypt-ACME-003A70?style=for-the-badge&logo=letsencrypt&logoColor=white)
![Synology DSM](https://img.shields.io/badge/Synology-DSM-B5B5B6?style=for-the-badge&logo=synology&logoColor=111827)
![Telegram](https://img.shields.io/badge/Telegram-Control-26A5E4?style=for-the-badge&logo=telegram&logoColor=white)

**🟢 Automatic DNS API** &nbsp; **🟡 Manual TXT fallback** &nbsp; **🔵 DSM deploy** &nbsp; **🟣 Telegram**

</div>

---

**SynoCert Flow** — лёгкий Docker-проект для автоматического выпуска, продления и установки wildcard-сертификатов Let's Encrypt в **Synology DSM**.

Проект написан на **POSIX Shell**, работает поверх официального образа **acme.sh**, запускается через **Docker Compose** и не требует собственной базы данных или отдельного backend.

## ✨ Возможности

- 🔄 автоматическое продление сертификатов через DNS API;
- 🌐 поддержка **REG.RU** и **Spaceship**;
- 📝 manual DNS-01 workflow, если API не используется;
- 🛟 автоматический API → manual fallback;
- 🔐 deploy готового сертификата прямо в Synology DSM;
- 🧩 несколько доменов, DSM-пользователей и ACME accounts;
- 🧪 production / Let's Encrypt staging;
- 🧠 renewal scheduling с учётом `Le_NextRenewTime` / ARI;
- 🤖 Telegram-команды, уведомления и диагностика;
- ❤️ Docker healthchecks, логи и backups;
- 🛡 защита от случайных повторных `--force` выпусков.

## 🧱 Как это работает

```text
SynoCert Flow
    │
    ├── acme.sh
    │     └── Let's Encrypt
    │
    ├── DNS
    │     ├── REG.RU API
    │     ├── Spaceship API
    │     └── Manual TXT
    │
    ├── Synology DSM
    │     └── automatic certificate deploy
    │
    └── Telegram
          ├── status
          ├── test
          ├── renew
          └── notifications
```

## 🌍 DNS-провайдеры

SynoCert Flow использует DNS hooks из `acme.sh`, поэтому проект не привязан только к REG.RU и Spaceship.

Для другого поддерживаемого провайдера достаточно создать профиль:

```text
config/dns-providers.d/<profile>.env
```

указать в нём нужный `DNS_API` и переменные credentials из соответствующего `acme.sh` hook, а в домене выбрать:

```env
DNS_PROVIDER='<profile>'
```

> Полный provider-specific `/test` в v1.5.0 реализован для REG.RU и Spaceship. Для остальных провайдеров выпуск/renewal может работать без изменения версии, но расширенная проверка credentials и доступа к зоне потребует отдельного test-adapter.

## 🚀 Быстрый старт

```bash
cp .env.example .env
cp config/domain.example.env config/domains.d/example.com.env
docker compose up -d
```

Минимальный конфиг домена:

```env
CONFIG_VERSION='4'

DOMAIN='example.com'
ACME_ACCOUNT='default'
ACME_ENV='production'

DNS_PROVIDER=''

SYNO_CERTIFICATE='Wildcard example.com'
SYNO_USERNAME='acme-example'
SYNO_PASSWORD='CHANGE_ME'
SYNO_HOSTNAME='192.168.1.100'

ENABLED='1'
```

### DNS API

Для REG.RU:

```env
DNS_PROVIDER='regru-main'
```

Профиль `config/dns-providers.d/regru-main.env`:

```env
DNS_API='dns_regru'

REGRU_API_Username='LOGIN'
REGRU_API_Password='PASSWORD'
```

Для Spaceship используется тот же принцип с `dns_spaceship`.

> Реальные `.env`, DNS API credentials, ACME data и private keys не должны попадать в Git.

## 🤖 Telegram

После настройки `TELEGRAM_BOT_TOKEN` и `TELEGRAM_CHAT_ID`:

```text
/help
/status
/domains
/cert <domain>
/test <domain>
/health
/renew <domain>
/renew-force <domain>
/deploy <domain>
/debug <domain>
```

`/test <domain>` выполняет безопасный preflight без изменения DNS и без выпуска сертификата:

```text
DSM API
DNS profile
DNS credentials
Registrar authentication
Domain / DNS zone access
Let's Encrypt CA
ACME account
Renewal readiness
```

Успешный итог:

```text
Automatic renewal: ✅ READY
```

## 🛡 Безопасное продление

Обычный цикл **не создаёт новый ACME order**, пока сертификат не вошёл в окно продления.

Если `acme.sh` или ARI сообщает, что renewal ещё не требуется:

```text
SKIPPED / NOT DUE
```

это считается нормальным состоянием — без ошибки и без manual fallback.

Принудительный выпуск доступен только через:

```text
/renew-force <domain>
```

## 🔄 Обновление с v1.4.x

```bash
sh /volume1/docker/synocert-flow-v1.5.0/upgrade.sh \
  /volume1/docker/synocert-flow-v1.5.0 \
  /volume1/docker/synology-acme-manual
```

После обновления пересоздайте Project в Synology Container Manager.

Сохраняются:

```text
.env
data/
state/
logs/
backups/
config/domains.d/
config/dns-providers.d/
```

## 📁 Структура

```text
app/                      workers и healthcheck
config/domains.d/         конфиги доменов
config/dns-providers.d/   DNS API profiles
data/                     ACME accounts, keys, certificates
state/                    runtime state
logs/                     журналы
backups/                  резервные копии сертификатов
compose.yaml              Docker Compose
```

## ⚙️ Stack

**POSIX Shell · Docker Compose · acme.sh · Let's Encrypt · Synology DSM API · Telegram Bot API**

---

<div align="center">

### SynoCert Flow

**Set it once. Let certificates renew themselves.**

</div>

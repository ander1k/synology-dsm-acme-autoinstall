<div align="center">

# 🔐 SynoCert Flow

### Automated Let's Encrypt certificate management for Synology DSM

**DNS API • Manual DNS fallback • DSM deploy • Telegram control • Multi-domain**

![Version](https://img.shields.io/badge/version-1.5.0-111827?style=for-the-badge)
![Shell](https://img.shields.io/badge/POSIX-Shell-111827?style=for-the-badge&logo=gnu-bash&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Compose-111827?style=for-the-badge&logo=docker&logoColor=white)
![ACME](https://img.shields.io/badge/acme.sh-Let's_Encrypt-111827?style=for-the-badge&logo=letsencrypt&logoColor=white)
![Synology](https://img.shields.io/badge/Synology-DSM-111827?style=for-the-badge)
![Telegram](https://img.shields.io/badge/Telegram-Control-111827?style=for-the-badge&logo=telegram&logoColor=white)

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

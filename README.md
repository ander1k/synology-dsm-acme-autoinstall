<div align="center">

# 🔐 SynoCert Flow

### Automated Let's Encrypt certificate management for Synology DSM

**DNS API • Manual TXT fallback • DSM deploy • Telegram control**

![Release](https://img.shields.io/badge/Release-v1.5.0-7C3AED?style=for-the-badge&logo=github&logoColor=white)
![POSIX Shell](https://img.shields.io/badge/POSIX-Shell-F4B400?style=for-the-badge&logo=gnubash&logoColor=111827)
![Docker Compose](https://img.shields.io/badge/Docker-Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)

![Let's Encrypt](https://img.shields.io/badge/Let's%20Encrypt-ACME-003A70?style=for-the-badge&logo=letsencrypt&logoColor=white)
![Synology DSM](https://img.shields.io/badge/Synology-DSM-B5B5B6?style=for-the-badge&logo=synology&logoColor=111827)
![Telegram](https://img.shields.io/badge/Telegram-Control-26A5E4?style=for-the-badge&logo=telegram&logoColor=white)

**🟢 Automatic DNS API** &nbsp; **🟡 Manual TXT** &nbsp; **🔵 DSM deploy** &nbsp; **🟣 Telegram**

</div>

---

**SynoCert Flow** — лёгкий Docker-проект для выпуска, продления и автоматической установки wildcard-сертификатов Let's Encrypt в **Synology DSM**.

Написан на **POSIX Shell**, работает поверх **acme.sh**, запускается через **Docker Compose** и не требует базы данных или отдельного backend.

> 🚀 **Полностью автоматический цикл:** SynoCert Flow создаёт DNS challenge через API регистратора, проверяет появление TXT-записей в публичном DNS, выдерживает период стабилизации **600 секунд** (по умолчанию), завершает проверку Let's Encrypt и **сам устанавливает готовый сертификат в Synology DSM**. Ручной deploy не требуется.

```text
DNS API → TXT verification → 600s stabilization → Let's Encrypt → Synology DSM deploy
```

## ✨ Возможности

- 🔄 **Автопродление** — автоматический renewal через DNS API;
- 🌍 **DNS API** — REG.RU и Spaceship из коробки;
- 📝 **Manual DNS-01** — ручной TXT workflow, если API не используется;
- 🛟 **Fallback** — автоматический переход API → manual при реальной ошибке;
- 🔐 **DSM Deploy** — установка готового сертификата прямо в Synology DSM;
- 🧩 **Multi-domain** — несколько доменов, DSM-пользователей и ACME accounts;
- 🧪 **Production / Staging** — безопасное тестирование через Let's Encrypt staging;
- 🧠 **ARI-aware renewal** — учёт `Le_NextRenewTime` и рекомендованного окна продления;
- 🤖 **Telegram Control** — команды, уведомления, диагностика и статусы;
- ❤️ **Health & Logs** — healthcheck, журналы и резервные копии;
- 🛡 **Rate-limit safety** — защита от случайных повторных `--force` выпусков.

## 🚀 Установка в Synology Container Manager

### 1. Распакуйте проект

Например:

```text
/volume1/docker/synocert-flow
```

### 2. Создайте `.env`

```bash
cp .env.example .env
```

Укажите минимум:

```env
TELEGRAM_BOT_TOKEN=''
TELEGRAM_CHAT_ID=''
```

Telegram можно оставить пустым, если он не нужен.

### 3. Создайте конфиг домена

```bash
cp config/domain.example.env \
   config/domains.d/example.com.env
```

Минимальный пример:

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

### 4. Создайте Project в Container Manager

Откройте:

```text
Container Manager → Проект → Создать
```

Укажите:

```text
Имя проекта: synocert-flow
Путь: /volume1/docker/synocert-flow
Источник: compose.yaml
```

Нажмите **Далее → Готово** и запустите проект.

После старта должны появиться контейнеры:

```text
acme-synology
acme-telegram
```

### 5. Проверьте работу

В Telegram:

```text
/version
/health
/status
/test example.com
```

Без Telegram смотрите журнал контейнера `acme-synology`.

## 🌐 DNS API

Для REG.RU:

```env
DNS_PROVIDER='regru-main'
```

Создайте:

```text
config/dns-providers.d/regru-main.env
```

```env
DNS_API='dns_regru'

REGRU_API_Username='LOGIN'
REGRU_API_Password='PASSWORD'
```

Для Spaceship используется тот же принцип с `dns_spaceship`.

Другие DNS-провайдеры можно подключать через DNS hooks, поддерживаемые `acme.sh`, без изменения ядра проекта.

> Реальные `.env`, DNS credentials, ACME data и private keys не публикуйте в Git.

## 🤖 Telegram

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

`/test <domain>` проверяет DSM, DNS API, доступ к домену, Let's Encrypt CA и готовность к автоматическому продлению — без изменения DNS и без выпуска сертификата.

Успешный итог:

```text
Automatic renewal: ✅ READY
```

## 🛡 Renewal safety

Обычный цикл не создаёт новый ACME order, пока сертификат не вошёл в окно продления.

`SKIPPED / NOT DUE` считается нормальным состоянием и не запускает manual fallback.

Принудительный выпуск:

```text
/renew-force <domain>
```


---

<div align="center">

### SynoCert Flow

**Set it once. Let certificates renew themselves.**

</div>

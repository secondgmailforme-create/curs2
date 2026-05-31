# curs2

Курсовой проект: веб-приложение для службы поддержки пользователей. Внутри — Node.js backend на Express, PostgreSQL, Redis, Socket.IO и статический frontend на HTML/CSS/JS. Проект рассчитан на запуск через Docker Compose, чтобы не ставить локально Node.js, PostgreSQL и Redis.

Приложение работает как helpdesk: клиент создаёт обращение или пишет в чат, AI-помощник может ответить автоматически, оператор берёт заявку в работу, эксперт подключается к сложным обращениям, администратор управляет пользователями, заявками, категориями и статистикой.

## Что нужно

Для обычного запуска нужен только Docker.

Установить:

- Docker Desktop / Docker Engine;
- Docker Compose;
- Git, если проект скачивается через `git clone`.

Порты по умолчанию:

| Порт | Назначение |
|------|------------|
| `3001` | backend + frontend |
| `5432` | PostgreSQL |
| `6379` | Redis |
| `1025` | SMTP Mailpit |
| `8025` | Web-интерфейс Mailpit |

Если порт занят, его можно поменять в `.env`.

## Быстрый запуск

Скачать проект:

```bash
git clone https://github.com/secondgmailforme-create/curs2.git
cd curs2
```

Запустить через Docker Compose:

```bash
docker compose up --build -d
```

Открыть приложение:

```text
http://localhost:3001/frontend/main-module.html
```

Или открыть корень:

```text
http://localhost:3001
```

Посмотреть логи backend:

```bash
docker compose logs -f backend
```

## Запуск через управляющие скрипты

В проекте есть готовые bat/sh-скрипты для управления Docker-стеком. Они лежат в папке:

```text
curs5-scripts/
```

Это удобный вариант, чтобы не писать Docker-команды вручную.

### Windows

Главный скрипт-меню:

```text
curs5-scripts/curs5.bat
```

Его можно открыть двойным кликом или через терминал:

```bat
cd curs5-scripts
curs5.bat
```

В меню есть пункты:

| Пункт | Что делает |
|------|------------|
| `1` | запуск с пересборкой: `up --build -d` |
| `2` | запуск без пересборки |
| `3` | остановка контейнеров |
| `4` | пересоздание backend после изменения `.env` |
| `5` | полная пересборка backend без кэша |
| `6` | логи backend |
| `7` | логи всех сервисов |
| `8` | статус контейнеров |
| `9` | открыть приложение в браузере |
| `M` | открыть Mailpit |
| `P` | открыть `psql` внутри postgres-контейнера |
| `B` | открыть shell внутри backend-контейнера |
| `R` | полная очистка с удалением volume'ов |
| `0` | выход |

Также есть короткие bat-скрипты:

| Файл | Назначение |
|------|------------|
| `curs5-scripts/up.bat` | быстрый запуск с пересборкой и открытием приложения |
| `curs5-scripts/start.bat` | запуск с build + up, оставлен как альтернативный старт |
| `curs5-scripts/down.bat` | остановка контейнеров |
| `curs5-scripts/down.bat --purge` | остановка и удаление volume'ов |
| `curs5-scripts/stop.bat` | остановка контейнеров с паузой в конце |
| `curs5-scripts/stop.bat --purge` | остановка и удаление данных |
| `curs5-scripts/restart-backend.bat` | пересоздание только backend |
| `curs5-scripts/logs.bat` | просмотр логов backend |

Обычный сценарий на Windows:

```bat
cd curs5-scripts
curs5.bat
```

Или быстро:

```bat
cd curs5-scripts
up.bat
```

### Linux / macOS

Главное меню:

```bash
cd curs5-scripts
chmod +x curs5.sh start.sh stop.sh
./curs5.sh
```

Быстрый запуск:

```bash
cd curs5-scripts
chmod +x start.sh
./start.sh
```

Остановка:

```bash
cd curs5-scripts
./stop.sh
```

Остановка с удалением данных:

```bash
cd curs5-scripts
./stop.sh --purge
```

## Ручное управление через Docker Compose

Если скрипты не нужны, можно использовать команды напрямую из корня проекта.

Запуск с пересборкой:

```bash
docker compose up --build -d
```

Запуск без пересборки:

```bash
docker compose up -d
```

Статус контейнеров:

```bash
docker compose ps
```

Логи backend:

```bash
docker compose logs -f backend
```

Логи всех сервисов:

```bash
docker compose logs -f
```

Остановка:

```bash
docker compose down
```

Остановка с удалением данных:

```bash
docker compose down -v
```

Пересоздать только backend:

```bash
docker compose up -d --force-recreate backend
```

Пересобрать backend без кэша:

```bash
docker compose build --no-cache backend
docker compose up -d backend
```

## Тестовый вход

В начальных данных есть тестовый администратор:

```text
Email: testadmin2@mail.com
Password: 12345678
```

Он создаётся при первичной инициализации базы из файла:

```text
backend/base_data/shema.sql
```

Если база уже была создана раньше, SQL-файл повторно автоматически не применяется. Для пересоздания базы нужно удалить volume'ы:

```bash
docker compose down -v
docker compose up --build -d
```

## Что поднимается в Docker

В `docker-compose.yml` описаны четыре сервиса.

### `postgres`

PostgreSQL 16. Хранит пользователей, заявки, сообщения, уведомления, рейтинги, логи и остальные данные.

Контейнер:

```text
curs5-postgres
```

Схема БД:

```text
backend/base_data/shema.sql
```

### `redis`

Redis 7. Используется для сессий, rate-limit и вспомогательных данных.

Контейнер:

```text
curs5-redis
```

### `backend`

Node.js приложение. Запускает Express API, Socket.IO и отдаёт frontend как статику.

Контейнер:

```text
curs5-backend
```

Основной файл:

```text
backend/server.js
```

### `mailpit`

Тестовый SMTP-сервер. Нужен для локальной проверки писем.

Контейнер:

```text
curs5-mailpit
```

Web-интерфейс:

```text
http://localhost:8025
```

## `.env`

Настройки проекта хранятся в `.env` в корне проекта.

Основные переменные:

```env
# Порты
BACKEND_PORT_HOST=3001
DB_PORT_HOST=5432
REDIS_PORT_HOST=6379
MAILPIT_SMTP_PORT_HOST=1025
MAILPIT_UI_PORT_HOST=8025

# PostgreSQL
DB_NAME=helpdesk
DB_USER=helpdesk
DB_PASSWORD=helpdesk_secret

# Приложение
NODE_ENV=production
PORT=3001
FRONTEND_URL=http://localhost:3001

# JWT / сессии
JWT_SECRET=change_me
JWT_EXPIRES_IN=7d
SESSION_SECRET=change_me

# SMTP
SMTP_HOST=
SMTP_PORT=
SMTP_SECURE=false
SMTP_USER=
SMTP_PASS=
SMTP_FROM=
EMAIL_USER=

# AI
AI_API_URL=
AI_MODEL_NAME=
CLASSIFIER_URL=
```

Внутри Docker-сети backend подключается к PostgreSQL и Redis по именам сервисов:

```text
postgres
redis
```

Поэтому `DB_HOST` и `REDIS_URL` задаются в `docker-compose.yml`.

После изменения `.env` обычно нужно пересоздать backend:

```bash
docker compose up -d --force-recreate backend
```

Или через меню:

```text
curs5.bat -> пункт 4
```

## SMTP и письма

Если реальный SMTP не настроен, письма удобно смотреть через Mailpit:

```text
http://localhost:8025
```

Если используется Gmail, нужен App Password, а не обычный пароль от аккаунта.

Переменные для SMTP:

```env
SMTP_HOST=smtp.gmail.com
SMTP_PORT=465
SMTP_SECURE=true
SMTP_USER=your_email@gmail.com
SMTP_PASS=your_app_password
SMTP_FROM=your_email@gmail.com
EMAIL_USER=your_email@gmail.com
```

## AI

AI-подключение задаётся переменными:

```env
AI_API_URL=
AI_MODEL_NAME=
CLASSIFIER_URL=
```

Если LM Studio запущен на той же машине, где Docker Desktop:

```env
AI_API_URL=http://host.docker.internal:1234/v1
```

Если AI-сервис на другой машине, нужно указать её IP и порт.

## Структура проекта

```text
curs2/
├── backend/
│   ├── base_data/
│   │   └── shema.sql
│   ├── constants/
│   ├── controllers/
│   ├── middlewares/
│   ├── queries/
│   ├── repositories/
│   ├── routes/
│   ├── schemas/
│   ├── services/
│   ├── utils/
│   ├── Dockerfile
│   ├── package.json
│   └── server.js
├── frontend/
│   ├── css/
│   ├── files/
│   ├── htmls/
│   ├── scripts-js/
│   └── main-module.html
├── curs5-scripts/
│   ├── curs5.bat
│   ├── curs5.sh
│   ├── up.bat
│   ├── start.bat
│   ├── down.bat
│   ├── stop.bat
│   ├── restart-backend.bat
│   ├── logs.bat
│   ├── start.sh
│   └── stop.sh
├── docker-compose.yml
└── README.md
```

## Backend

Backend находится в папке:

```text
backend/
```

Основные части:

| Папка | Назначение |
|------|------------|
| `controllers/` | обработка HTTP-запросов |
| `services/` | бизнес-логика |
| `repositories/` | работа с базой данных |
| `routes/` | API-маршруты |
| `middlewares/` | авторизация, роли, загрузка файлов, ошибки |
| `schemas/` | Joi-валидация входных данных |
| `queries/` | SQL-запросы |
| `constants/` | роли, статусы и другие константы |
| `base_data/` | SQL-схема базы данных |
| `utils/` | вспомогательные функции |

## Frontend

Frontend находится в папке:

```text
frontend/
```

Это статические HTML, CSS и JS файлы. Backend отдаёт их через Express.

Основные части:

| Папка | Назначение |
|------|------------|
| `htmls/auth/` | вход, регистрация, восстановление пароля |
| `htmls/client/` | клиентский кабинет и чат |
| `htmls/operator/` | интерфейс оператора |
| `htmls/expert/` | интерфейс эксперта |
| `htmls/admin/` | административная панель |
| `css/` | стили |
| `scripts-js/` | frontend-логика |
| `files/` | изображения и статические файлы |

## Основные API-разделы

| API | Назначение |
|-----|------------|
| `/api/auth` | регистрация, вход, выход, восстановление пароля |
| `/api/tickets` | заявки, комментарии, вложения, история |
| `/api/admin` | административная панель |
| `/api/operator` | действия оператора |
| `/api/expert` | действия эксперта |
| `/api/notifications` | уведомления |
| `/api/ratings` | оценки операторов и экспертов |
| `/api/profile` | профиль пользователя |
| `/api/translate` | перевод текста |

## Роли

| ID | Роль | Назначение |
|----|------|------------|
| `1` | `client` | клиент |
| `2` | `operator` | оператор |
| `3` | `expert` | эксперт |
| `4` | `admin` | администратор |

Роли описаны в файле:

```text
backend/constants/roles.js
```

## База данных

Подключиться к PostgreSQL внутри контейнера:

```bash
docker compose exec postgres psql -U helpdesk -d helpdesk
```

Или через меню Windows:

```text
curs5.bat -> пункт P
```

Полезные команды внутри `psql`:

```sql
\dt
SELECT * FROM users;
SELECT * FROM tickets;
SELECT * FROM statuses;
```

Основные таблицы:

| Таблица | Назначение |
|---------|------------|
| `roles` | роли пользователей |
| `users` | пользователи |
| `user_tokens` | refresh-токены |
| `verification_codes` | email-коды подтверждения |
| `password_resets` | токены сброса пароля |
| `statuses` | статусы заявок |
| `categories` | категории обращений |
| `tickets` | заявки |
| `chat_messages` | сообщения чата |
| `comments` | комментарии |
| `ticket_ratings` | оценки |
| `attachments` | вложения |
| `ticket_history` | история изменений |
| `logs` | журнал действий |
| `notifications` | уведомления |
| `ai_training_data` | данные для AI |

## Логи

Логи backend:

```bash
docker compose logs -f backend
```

Логи всех сервисов:

```bash
docker compose logs -f
```

Через Windows-меню:

```text
curs5.bat -> пункт 6  # backend
curs5.bat -> пункт 7  # все сервисы
```

## Тесты

Для backend добавлены unit-тесты на Jest.

Запуск:

```bash
cd backend
npm install
npm test
```

С покрытием:

```bash
npm run test:coverage
```

Тесты лежат здесь:

```text
backend/__tests__/
```

## Частые ситуации

### Порт занят

Поменять порт в `.env`, например:

```env
BACKEND_PORT_HOST=3002
DB_PORT_HOST=5433
REDIS_PORT_HOST=6380
```

После этого перезапустить:

```bash
docker compose up -d
```

### Нужно полностью пересоздать базу

```bash
docker compose down -v
docker compose up --build -d
```

Или через меню:

```text
curs5.bat -> пункт R
```

### Нужно посмотреть письма

Открыть:

```text
http://localhost:8025
```

Или:

```text
curs5.bat -> пункт M
```

### Нужно зайти внутрь backend-контейнера

```bash
docker compose exec backend sh
```

Или:

```text
curs5.bat -> пункт B
```

## Короткая шпаргалка

```bash
# запуск
docker compose up --build -d

# статус
docker compose ps

# логи backend
docker compose logs -f backend

# остановка
docker compose down

# полная очистка
docker compose down -v

# psql
docker compose exec postgres psql -U helpdesk -d helpdesk

# shell backend
docker compose exec backend sh
```

На Windows удобнее всего пользоваться:

```text
curs5-scripts/curs5.bat
```

# emarket-api-mock
Создание отдельного Mock API на базе WireMock для имитации HTTP API платформы emarket.

# emarket-api-mock — Stage 1

## 1. Назначение проекта

`emarket-api-mock` — отдельный проект, который имитирует работу `emarket-api-gateway` и возвращает заранее подготовленные HTTP responses для frontend-приложений.

На Stage 1 у нас два клиента:

- `emarket-web` — React + TypeScript + Vite.
- `emarket-mobile` — мобильное приложение.

В качестве основного решения предлагаю использовать WireMock.

## Stage 1 — Architecture

emarket-web

React + TypeScript + Vite

emarket-mobile

Mobile application

emarket-api-mock

WireMock

HTTP request matching · Response simulation

Mock responses

JSON · HTTP status · Headers · Delays · Errors

### Основные задачи

1. Имитировать REST API будущего `emarket-api-gateway`.
2. Предоставлять независимые mock responses для Web и Mobile.
3. Поддерживать стандартные HTTP status codes.
4. Имитировать ошибки backend: 400, 401, 403, 404, 409, 500, 503.
5. Поддерживать разные тестовые сценарии.
6. Позволять frontend-разработке продолжаться без готового backend.
7. Не требовать изменения frontend-кода после подключения настоящего API Gateway.

Важное архитектурное решение: `emarket-api-mock` должен имитировать внешний HTTP-контракт API Gateway, а не внутреннюю реализацию BFF или микросервисов.

## 2. Структура проекта

Предлагаю следующую структуру:

```
emarket-api-mock/
│
├── wiremock/
│   │
│   ├── mappings/
│   │   │
│   │   ├── common/
│   │   │   ├── health.json
│   │   │   └── errors.json
│   │   │
│   │   ├── web/
│   │   │   ├── auth/
│   │   │   │   ├── register.json
│   │   │   │   ├── login.json
│   │   │   │   └── logout.json
│   │   │   │
│   │   │   ├── users/
│   │   │   │   └── get-current-user.json
│   │   │   │
│   │   │   └── products/
│   │   │       └── get-products.json
│   │   │
│   │   └── mobile/
│   │       ├── auth/
│   │       │   ├── register.json
│   │       │   ├── login.json
│   │       │   └── logout.json
│   │       │
│   │       ├── users/
│   │       │   └── get-current-user.json
│   │       │
│   │       └── products/
│   │           └── get-products.json
│   │
│   ├── __files/
│   │   │
│   │   ├── web/
│   │   │   ├── auth/
│   │   │   ├── users/
│   │   │   └── products/
│   │   │
│   │   └── mobile/
│   │       ├── auth/
│   │       ├── users/
│   │       └── products/
│   │
│   └── extensions/
│
├── tests/
│   ├── web/
│   ├── mobile/
│   └── common/
│
├── docs/
│   ├── architecture.md
│   ├── api-contracts.md
│   └── scenarios.md
│
├── config/
│   └── wiremock.properties
│
├── docker-compose.yml
├── .env.example
├── .gitignore
└── README.md
```

## 3. Описание папок и файлов

| Папка / файл         | Назначение                                           |
| -------------------- | ---------------------------------------------------- |
| `wiremock/`          | Основная конфигурация WireMock                       |
| `mappings/`          | Правила сопоставления HTTP requests и responses      |
| `mappings/common/`   | Общие endpoints и ошибки                             |
| `mappings/web/`      | Mapping-файлы для Web                                |
| `mappings/mobile/`   | Mapping-файлы для Mobile                             |
| `__files/`           | JSON-файлы с телами HTTP responses                   |
| `__files/web/`       | Response payloads для Web                            |
| `__files/mobile/`    | Response payloads для Mobile                         |
| `extensions/`        | Дополнительные расширения WireMock, если понадобятся |
| `tests/`             | Автоматические тесты mock API                        |
| `tests/web/`         | Проверка Web responses                               |
| `tests/mobile/`      | Проверка Mobile responses                            |
| `tests/common/`      | Проверка общих сценариев                             |
| `docs/`              | Документация проекта                                 |
| `config/`            | Конфигурация WireMock                                |
| `docker-compose.yml` | Запуск WireMock через Docker                         |
| `.env.example`       | Пример переменных окружения                          |
| `.gitignore`         | Исключения Git                                       |
| `README.md`          | Описание и инструкции запуска                        |

### Назначение основных файлов

Mapping-файлы

Например, `mappings/web/auth/login.json`:

- HTTP method: POST.
- URL: `/api/v1/web/auth/login`.
- Request matching: JSON body, headers.
- Response status: 200.
- Response body: ссылка на JSON в `__files`.

Response-файлы

Например:

`__files/web/auth/login-success.json`

Содержат готовый JSON, который WireMock возвращает frontend-приложению.

Такое разделение позволяет изменять сценарии HTTP matching отдельно от содержимого responses.

## 4. API contracts — Stage 1

Предлагаю на первом этапе реализовать следующие группы endpoints.

| Module   | Method | Endpoint                    | Успешный response |
| -------- | ------ | --------------------------- | ----------------- |
| Auth     | POST   | `/api/v1/web/auth/register` | 201               |
| Auth     | POST   | `/api/v1/web/auth/login`    | 200               |
| Auth     | POST   | `/api/v1/web/auth/logout`   | 204               |
| Users    | GET    | `/api/v1/web/users/me`      | 200               |
| Products | GET    | `/api/v1/web/products`      | 200               |
| Health   | GET    | `/actuator/health`          | 200               |

Для Mobile используется аналогичный набор:

`/api/v1/mobile/...`

Это позволяет сразу отделить API-контракты двух клиентов.

### 4.1. Authentication

POST `/api/v1/web/auth/register`

Request:

```
{
  "email": "alex@example.com",
  "password": "Password123!",
  "firstName": "Alex",
  "lastName": "Green"
}
```

Response — HTTP 201:

```
{
  "id": "user-001",
  "email": "alex@example.com",
  "firstName": "Alex",
  "lastName": "Green",
  "createdAt": "2026-11-05T10:00:00Z"
}
```

POST `/api/v1/web/auth/login`

Request:

```
{
  "email": "alex@example.com",
  "password": "Password123!"
}
```

Response — HTTP 200:

```
{
  "accessToken": "mock-access-token",
  "refreshToken": "mock-refresh-token",
  "tokenType": "Bearer",
  "expiresIn": 3600,
  "user": {
    "id": "user-001",
    "email": "alex@example.com",
    "firstName": "Alex",
    "lastName": "Green",
    "roles": [
      "USER"
    ]
  }
}
```

POST `/api/v1/web/auth/logout`

Request:

```
Authorization: Bearer mock-access-token
```

Response:

```
HTTP/1.1 204 No Content
```

### 4.2. User profile

GET `/api/v1/web/users/me`

Request:

```
GET /api/v1/web/users/me
Authorization: Bearer mock-access-token
```

Response — HTTP 200:

```
{
  "id": "user-001",
  "email": "alex@example.com",
  "firstName": "Alex",
  "lastName": "Green",
  "roles": [
    "USER"
  ],
  "addresses": [
    {
      "id": "address-001",
      "city": "Chisinau",
      "country": "Moldova",
      "street": "Example Street",
      "postalCode": "2001"
    }
  ]
}
```

### 4.3. Products

GET `/api/v1/web/products`

Request:

```
GET /api/v1/web/products?page=0&size=20&sort=name,asc
```

Response — HTTP 200:

```
{
  "content": [
    {
      "id": "product-001",
      "name": "Laptop Lenovo",
      "description": "Business laptop",
      "price": 1200.00,
      "currency": "EUR",
      "imageUrl": "/images/laptop-001.jpg",
      "available": true
    },
    {
      "id": "product-002",
      "name": "Wireless Mouse",
      "description": "Bluetooth mouse",
      "price": 35.00,
      "currency": "EUR",
      "imageUrl": "/images/mouse-001.jpg",
      "available": true
    }
  ],
  "page": 0,
  "size": 20,
  "totalElements": 2,
  "totalPages": 1
}
```

## 5. Поддержка разных HTTP responses для Web и Mobile

Да, WireMock поддерживает это

Это одна из основных причин, почему WireMock подходит для данного проекта.

Есть несколько способов разделять responses.

### Вариант A. Разные URL

Например:

```
POST /api/v1/web/auth/login
POST /api/v1/mobile/auth/login
```

WireMock выбирает соответствующий mapping по URL.

Преимущества:

- Простая конфигурация.
- Легко поддерживать.
- Нет необходимости определять тип клиента по User-Agent.

Недостаток: необходимо поддерживать два набора URL.

### Вариант B. Один URL + HTTP Header

Я рекомендую этот вариант, если в будущем Web и Mobile будут использовать единый API Gateway contract.

Оба клиента обращаются к одному endpoint:

```
POST /api/v1/auth/login
```

Но передают разные headers.

Web:

```
X-Client-Platform: WEB
Content-Type: application/json
```

Mobile:

```
X-Client-Platform: MOBILE
Content-Type: application/json
```

Request body может быть одинаковым.

WireMock определяет клиента по `X-Client-Platform` и возвращает соответствующий response.

Пример поведения:

Web

HTTP 200

X-Client-Platform: WEB

```
{
  "accessToken": "web-token",
  "user": {
    "id": "user-001",
    "email": "alex@example.com"
  }
}
```

Mobile

HTTP 200

X-Client-Platform: MOBILE

```
{
  "accessToken": "mobile-token",
  "refreshToken": "mobile-refresh-token",
  "expiresIn": 3600,
  "user": {
    "id": "user-001",
    "email": "alex@example.com"
  }
}
```

### Пример WireMock mapping

`mappings/web/auth/login.json`:

```
{
  "request": {
    "method": "POST",
    "urlPath": "/api/v1/auth/login",
    "headers": {
      "X-Client-Platform": {
        "equalTo": "WEB"
      }
    }
  },
  "response": {
    "status": 200,
    "headers": {
      "Content-Type": "application/json"
    },
    "bodyFileName": "web/auth/login-success.json"
  }
}
```

`mappings/mobile/auth/login.json`:

```
{
  "request": {
    "method": "POST",
    "urlPath": "/api/v1/auth/login",
    "headers": {
      "X-Client-Platform": {
        "equalTo": "MOBILE"
      }
    }
  },
  "response": {
    "status": 200,
    "headers": {
      "Content-Type": "application/json"
    },
    "bodyFileName": "mobile/auth/login-success.json"
  }
}
```

Оба mapping используют один URL, но возвращают разные JSON.

### 5.1. Можно ли возвращать разные HTTP status codes?

Да. Например:

| Scenario                  | Web | Mobile |
| ------------------------- | --- | ------ |
| Успешная авторизация      | 200 | 200    |
| Неверный пароль           | 401 | 401    |
| Пользователь заблокирован | 403 | 403    |
| Ошибка валидации          | 400 | 422    |
| Backend unavailable       | 503 | 503    |
| Internal server error     | 500 | 500    |

WireMock позволяет задавать разные status codes, headers, body и задержки для каждого mapping.

Однако рекомендую не создавать искусственные различия без необходимости: если бизнес-семантика операции одинаковая, лучше сохранять одинаковые HTTP status codes.

## 6. Обработка ошибок

Предлагаю создать общие error response contracts.

Например, HTTP 400:

```
{
  "timestamp": "2026-11-05T10:00:00Z",
  "status": 400,
  "error": "Bad Request",
  "message": "Validation failed",
  "path": "/api/v1/web/auth/register",
  "details": [
    {
      "field": "email",
      "message": "Invalid email format"
    }
  ]
}
```

Другие сценарии:

| HTTP status | Сценарий                  |
| ----------- | ------------------------- |
| 400         | Invalid request           |
| 401         | Unauthorized              |
| 403         | Forbidden                 |
| 404         | Resource not found        |
| 409         | Email already registered  |
| 422         | Business validation error |
| 429         | Too many requests         |
| 500         | Internal server error     |
| 503         | Service unavailable       |

Для каждого endpoint желательно предусмотреть:

- Success response.
- Validation error.
- Unauthorized.
- Business error.
- Server error.

Необязательно создавать все сценарии сразу. Их можно добавлять по мере реализации frontend.

## 7. Рекомендуемая конфигурация Docker

`docker-compose.yml`:

```
services:
  emarket-api-mock:
    image: wiremock/wiremock:3.13.1
    container_name: emarket-api-mock

    ports:
      - "8080:8080"

    volumes:
      - ./wiremock:/home/wiremock

    command:
      - "--global-response-templating"
      - "--verbose"

    restart: unless-stopped
```

Запуск:

```
docker compose up -d
```

WireMock будет доступен по адресу:

```
http://localhost:8080
```

Примеры:

```
http://localhost:8080/api/v1/web/auth/login
http://localhost:8080/api/v1/mobile/auth/login
http://localhost:8080/api/v1/web/products
```

Для Web и Mobile можно использовать разные base URLs в конфигурациях frontend.

## 8. Какой подход я рекомендую для emarket

Учитывая архитектуру Stage 1, я бы выбрал следующий вариант:

| Критерий              | Решение               |
| --------------------- | --------------------- |
| Mock server           | WireMock              |
| Deployment            | Docker                |
| Configuration         | JSON mappings         |
| Response payloads     | JSON files            |
| Web mappings          | Отдельная папка       |
| Mobile mappings       | Отдельная папка       |
| Common mappings       | Общая папка           |
| API versioning        | `/api/v1/`            |
| Client identification | Разные URL namespaces |
| Error responses       | Общий формат          |
| Automated tests       | JUnit + WireMock      |

Важное уточнение по идентификации клиента.

В твоём проекте уже предусмотрено разделение:

```
API Gateway
    |
    ├── /api/v1/web/**    → emarket-bff-web
    |
    └── /api/v1/mobile/** → emarket-bff-mobile
```

Поэтому для Stage 1 лучше сохранить эти namespaces и в mock API.

`X-Client-Platform` можно дополнительно использовать для тестирования, но не стоит делать его единственным механизмом идентификации клиента или безопасности.

Также рекомендую сохранять одинаковые бизнес-контракты там, где операции совпадают. Различия между Web и Mobile должны появляться только тогда, когда действительно нужны разные представления данных или сценарии взаимодействия.

## 9. Итоговая архитектура Stage 1

## emarket-api-mock

Recommended architecture

emarket-web

React + TypeScript + Vite

emarket-mobile

WireMock

`/api/v1/web/**`

mappings/web + \_\_files/web

`/api/v1/mobile/**`

mappings/mobile + \_\_files/mobile

`/actuator/health`

Common mappings

Главное преимущество: когда завтра появится третий клиент, например `emarket-admin`, не потребуется менять архитектуру mock server. Достаточно добавить:

```
mappings/admin/
__files/admin/
tests/admin/
```

И новые API scenarios.

Таким образом, `emarket-api-mock` будет независимым инфраструктурным проектом, который можно использовать для разработки, интеграционного тестирования и демонстрации всех frontend-клиентов ещё до готовности настоящего backend.

Следующим шагом можно подготовить полноценный `emarket-api-mock` repository: все директории, Docker Compose, WireMock mappings, JSON responses и автоматические тесты для Stage 1.


# Dev и Deployment режимы

Предлагаю реализовать `emarket-api-mock` на базе **WireMock 3.x** с поддержкой двух режимов запуска:

1. **Dev mode** — WireMock запускается непосредственно на host-машине, без Docker.
2. **Deployment mode** — WireMock запускается внутри Docker-контейнера.

> **Главное архитектурное решение:** оба режима используют одни и те же WireMock mappings и JSON responses. Не нужно создавать отдельные конфигурации API для локальной разработки и Docker.

---

## 1. Архитектура двух режимов

### Dev mode — Host machine (без Docker)

```text
emarket-web (Vite · localhost:5173)
       │
       │ HTTP · localhost:9090
       v
emarket-api-mock (WireMock Standalone · Java 21 · Port 9090)
```
*Mappings и responses читаются непосредственно с файловой системы host-машины.*

### Deployment mode — Docker

```text
emarket-web (Browser / Frontend)
       │
       │ HTTP · localhost:9090
       v
Docker Container: emarket-api-mock (WireMock · Container port 8080)
```
*Mounted `mappings` и `__files`.*

### Сетевые порты

Предлагаю использовать следующие порты:

| Компонент | Host port |
| :--- | :--- |
| `emarket-web` (Vite) | `5173` |
| `emarket-api-gateway` | `8080` |
| `emarket-api-mock` | `9090` |

**Почему 9090?**  
Потому что в Stage 1 уже предусмотрен `emarket-api-gateway` на порту `8080`. Mock API не должен конфликтовать с ним.

---

## 2. Структура проекта

Одна структура для обоих режимов:

```text
emarket-api-mock/
│
├── wiremock/
│   │
│   ├── mappings/
│   │   │
│   │   ├── common/
│   │   │   └── health.json
│   │   │
│   │   ├── web/
│   │   │   ├── auth/
│   │   │   │   ├── register.json
│   │   │   │   ├── login.json
│   │   │   │   ├── logout.json
│   │   │   │   ├── refresh.json
│   │   │   │   ├── me.json
│   │   │   │   ├── forgot-password.json
│   │   │   │   └── reset-password.json
│   │   │   │
│   │   │   ├── users/
│   │   │   │   ├── get-profile.json
│   │   │   │   └── update-profile.json
│   │   │   │
│   │   │   ├── addresses/
│   │   │   └── preferences/
│   │   │
│   │   └── mobile/
│   │       ├── auth/
│   │       ├── users/
│   │       ├── addresses/
│   │       └── preferences/
│   │
│   └── __files/
│       ├── common/
│       │   └── errors/
│       │
│       ├── web/
│       │   ├── auth/
│       │   ├── users/
│       │   ├── addresses/
│       │   └── preferences/
│       │
│       └── mobile/
│           ├── auth/
│           ├── users/
│           ├── addresses/
│           └── preferences/
│
├── scripts/
│   ├── start-host.sh
│   └── stop-host.sh
│
├── config/
│   └── wiremock-host.properties
│
├── docker-compose.yml
├── .env.example
├── .gitignore
├── README.md
└── LICENSE
```

**Описание элементов:**
- `mappings` — правила обработки HTTP requests.
- `__files` — JSON responses.
- `common` — общие ответы, например health check и ошибки.
- `web` — сценарии для Web.
- `mobile` — сценарии для Mobile.
- `scripts` — управление host-запуском.
- `config` — настройки host-режима.
- `docker-compose.yml` — конфигурация Docker-запуска.

*Структура `mappings` и `responses` не зависит от режима запуска.*

---

## 3. Общая конфигурация WireMock

Для обоих режимов необходимо зафиксировать одинаковые настройки:

| Параметр | Значение |
| :--- | :--- |
| **WireMock version** | 3.x, фиксированная версия |
| **HTTP port (host mode)** | `9090` |
| **HTTP port (Docker)** | `8080` внутри контейнера |
| **Host directory** | `./wiremock` |
| **Container directory** | `/home/wiremock` |
| **Mappings** | `wiremock/mappings` |
| **Response files** | `wiremock/__files` |
| **Response templating** | Enabled |
| **Request journal** | Enabled |
| **Verbose logging** | Development only |

Рекомендую использовать официальный WireMock Standalone JAR и официальный Docker image одной и той же версии. Например, зафиксировать версию **3.13.1** во всех конфигурациях, а не использовать `latest`.

> **Важно:** `wiremock-host.properties` не должен быть единственным источником настроек. Все параметры запуска должны явно передаваться в команду запуска или скрипт, чтобы host и Docker работали одинаково.

---

## 4. Dev mode — запуск на host-машине

В этом режиме Docker не нужен. WireMock запускается как обычное Java-приложение.

### 4.1. Предварительные требования

На host-машине должны быть установлены:
- Java 21
- Bash
- WireMock Standalone JAR

Проверка Java:
```bash
java -version
```
*Ожидаемый результат — Java 21.*

### 4.2. Загрузка WireMock

Создай директорию:
```bash
mkdir -p tools
```

Загрузи WireMock Standalone:
```bash
wget -O tools/wiremock-standalone-3.13.1.jar \
  https://repo1.maven.org/maven2/org/wiremock/wiremock-standalone/3.13.1/wiremock-standalone-3.13.1.jar
```

*Рекомендую не включать JAR в Git repository. Версию зафиксировать в документации, а загрузку выполнять отдельно.*

### 4.3. Скрипт запуска

Файл `scripts/start-host.sh`:

```bash
#!/usr/bin/env bash

set -euo pipefail

PROJECT_ROOT="$(cd "$(dirname "${BASH_SOURCE[0]}")/.." && pwd)"

WIREMOCK_VERSION="3.13.1"
WIREMOCK_JAR="$PROJECT_ROOT/tools/wiremock-standalone-${WIREMOCK_VERSION}.jar"

MOCK_API_PORT="${MOCK_API_PORT:-9090}"

if ! command -v java >/dev/null 2>&1; then
    echo "ERROR: Java is not installed."
    exit 1
fi

if [[ ! -f "$WIREMOCK_JAR" ]]; then
    echo "ERROR: WireMock JAR not found: $WIREMOCK_JAR"
    exit 1
fi

echo "Starting emarket-api-mock in DEV mode..."
echo "Port: $MOCK_API_PORT"

exec java -jar "$WIREMOCK_JAR" \
    --port "$MOCK_API_PORT" \
    --root-dir "$PROJECT_ROOT/wiremock" \
    --global-response-templating \
    --verbose
```

Сделать исполняемым:
```bash
chmod +x scripts/start-host.sh
```

Запуск:
```bash
./scripts/start-host.sh
```

WireMock запустится на: `http://localhost:9090`  
Остановка: `Ctrl + C`

### 4.4. Проверка работы

Проверить доступность WireMock:
```bash
curl http://localhost:9090/__admin
```

Проверить health endpoint:
```bash
curl -i http://localhost:9090/actuator/health
```
> **Здесь важно:** `/actuator/health` — это mock endpoint, а не настоящий Spring Boot Actuator. WireMock не предоставляет его автоматически.

Для проверки самого WireMock также доступен:
```bash
curl http://localhost:9090/__admin/health
```

Просмотр зарегистрированных mappings:
```bash
curl http://localhost:9090/__admin/mappings
```

---

## 5. Deployment mode — запуск через Docker

В этом режиме Java на host-машине не требуется. Нужен только Docker Engine и Docker Compose.

### 5.1. Docker Compose

Файл `docker-compose.yml`:

```yaml
services:
  emarket-api-mock:
    image: wiremock/wiremock:${WIREMOCK_VERSION:-3.13.1}
    container_name: emarket-api-mock
    ports:
      - "${MOCK_API_PORT:-9090}:8080"
    volumes:
      - ./wiremock:/home/wiremock:ro
    command:
      - "--global-response-templating"
    restart: unless-stopped
```

**Объяснение настроек:**

| Настройка | Назначение |
| :--- | :--- |
| `image` | Официальный WireMock image |
| `container_name` | Имя контейнера |
| `ports` | Публикация порта host → container |
| `volumes` | Подключение mappings и responses |
| `:ro` | Read-only доступ к mock-файлам |
| `command` | Включение response templating |
| `restart` | Автоматический перезапуск контейнера |

> **Обрати внимание:**
> ```yaml
> ports:
>   - "9090:8080"
> ```
> WireMock слушает порт `8080` внутри контейнера, но с host-машины доступен через `9090`.

### 5.2. Environment variables

Файл `.env.example`:
```env
WIREMOCK_VERSION=3.13.1
MOCK_API_PORT=9090
```

Создать локальный `.env`:
```bash
cp .env.example .env
```
*(Добавить `.env` в `.gitignore`)*

### 5.3. Запуск Docker

Из корневой директории проекта:
```bash
docker compose up -d
```

Проверить контейнер:
```bash
docker compose ps
```

Посмотреть логи:
```bash
docker compose logs -f emarket-api-mock
```

Проверить API:
```bash
curl http://localhost:9090/__admin
```

Остановка:
```bash
docker compose down
```

Перезапуск после изменения Docker-конфигурации:
```bash
docker compose up -d --force-recreate
```

### 5.4. Обновление mappings без перезапуска

Поскольку директория `wiremock` подключена как volume, изменения файлов доступны контейнеру. Однако важно учитывать, что WireMock не гарантирует автоматическую перезагрузку всех изменённых mappings.

Для явной перезагрузки:
```bash
curl -X POST http://localhost:9090/__admin/mappings/reset
```
*Эта команда удалит mappings, загруженные из файлов, и зарегистрирует их заново.*

Если нужно обновить mappings вместе с изменёнными файлами, используй:
```bash
curl -X POST http://localhost:9090/__admin/mappings/reload
```
*Это позволяет менять mock responses без пересоздания контейнера.*

> **Примечание по безопасности:** Admin API WireMock не следует открывать наружу в production. В deployment-профиле для тестовой среды ограничь доступ к нему сетью или отключи административный интерфейс.

---

## 6. Настройка emarket-web для двух режимов

В документации `emarket-web` уже предусмотрено разделение Base URL по окружениям:
- **Development:** `http://localhost:8080/api/v1`
- **Production:** `http://localhost:80/api/v1`

Я рекомендую не менять существующую настройку Development, а добавить отдельный Vite mode для Mock API.

### 6.1. Файлы окружений

В `emarket-web`:

```text
emarket-web/
│
├── .env.development
├── .env.mock
├── .env.production
└── package.json
```

`.env.development` — настоящий API Gateway:
```env
VITE_API_BASE_URL=http://localhost:8080/api/v1
```

`.env.mock` — Mock API:
```env
VITE_API_BASE_URL=http://localhost:9090/api/v1
```

`.env.production` — настоящий deployment API:
```env
VITE_API_BASE_URL=http://localhost:80/api/v1
```

### 6.2. package.json

Добавить команды в `scripts`:

```json
{
  "scripts": {
    "dev": "vite",
    "dev:mock": "vite --mode mock",
    "build": "tsc -b && vite build",
    "build:mock": "tsc -b && vite build --mode mock",
    "build:prod": "tsc -b && vite build --mode production"
  }
}
```

### 6.3. Запуск frontend с Mock API

Сначала запускаем WireMock на host:
```bash
cd emarket-api-mock
./scripts/start-host.sh
```

Во втором терминале:
```bash
cd emarket-web
npm run dev:mock
```

**Схема взаимодействия:**
```text
Browser
   │
   v
emarket-web (localhost:5173)
   │
   v
emarket-api-mock (localhost:9090)
```

Для перехода на настоящий API Gateway достаточно выполнить:
```bash
npm run dev
```
*И frontend снова использует `.env.development`.*

> **Важно:** для браузерных запросов с `localhost:5173` на `localhost:9090` необходимо настроить CORS в mock responses, включая preflight `OPTIONS`. Альтернативно можно настроить Vite proxy, чтобы браузер обращался к своему origin.

---

## 7. CORS — обязательная настройка

Поскольку Web работает на `localhost:5173`, а Mock API на `localhost:9090`, браузер рассматривает их как разные origins. Поэтому нужно предусмотреть CORS.

Для успешных responses:
```json
{
  "response": {
    "status": 200,
    "headers": {
      "Content-Type": "application/json",
      "Access-Control-Allow-Origin": "http://localhost:5173",
      "Access-Control-Allow-Methods": "GET, POST, PUT, PATCH, DELETE, OPTIONS",
      "Access-Control-Allow-Headers": "Content-Type, Authorization, X-Client-Platform"
    },
    "bodyFileName": "web/auth/login-success.json"
  }
}
```

Также нужен mapping для `OPTIONS` (preflight):
```json
{
  "request": {
    "method": "OPTIONS",
    "urlPattern": "/api/v1/.*"
  },
  "response": {
    "status": 204,
    "headers": {
      "Access-Control-Allow-Origin": "http://localhost:5173",
      "Access-Control-Allow-Methods": "GET, POST, PUT, PATCH, DELETE, OPTIONS",
      "Access-Control-Allow-Headers": "Content-Type, Authorization, X-Client-Platform",
      "Access-Control-Max-Age": "3600"
    }
  }
}
```

В deployment-среде разрешённый origin нужно изменить на адрес frontend deployment.

> **Важно:** Не используй `Access-Control-Allow-Origin: *`, если тестируешь cookie-based authentication с credentials.

---

## 8. Настройка emarket-mobile

Для Mobile необходимо предусмотреть отдельные настройки Base URL:

```text
emarket-mobile/
│
├── .env.development
├── .env.mock
└── .env.production
```

| Environment | API Base URL |
| :--- | :--- |
| **Development** | `http://localhost:8080/api/v1` |
| **Mock** | `http://localhost:9090/api/v1` |
| **Production** | `http://localhost:80/api/v1` |

- Если используется **Android Emulator**, для обращения к host-машине обычно потребуется: `http://10.0.2.2:9090/api/v1`
- Для **физического устройства** — IP-адрес компьютера в локальной сети, например: `http://192.168.1.100:9090/api/v1`

*(Это примеры адресов, а не фиксированные настройки. При этом необходимо разрешить доступ к порту 9090 через локальный firewall).*

---

## 9. Запуск всего проекта в разных режимах

### Сценарий A — Web + Mock API (Dev mode)
*Рекомендую для ежедневной разработки*

**Терминал №1:**
```bash
cd emarket-api-mock
./scripts/start-host.sh
```

**Терминал №2:**
```bash
cd emarket-web
npm run dev:mock
```

**Результат:**
| Компонент | Адрес |
| :--- | :--- |
| Web | `http://localhost:5173` |
| Mock API | `http://localhost:9090` |

---

### Сценарий B — Web + Mock API (Deployment mode)

**Терминал №1:**
```bash
cd emarket-api-mock
docker compose up -d
```

**Терминал №2:**
```bash
cd emarket-web
npm run dev:mock
```

**Результат:**
| Компонент | Адрес |
| :--- | :--- |
| Web | `http://localhost:5173` |
| Mock API container | `http://localhost:9090` |

*Для frontend разницы между этими режимами нет.*

---

### Сценарий C — Web + Mobile + Mock API

```text
                emarket-api-mock
                 localhost:9090
                       │
              ┌────────┴────────┐
              │                 │
              v                 v
         emarket-web       emarket-mobile
         localhost:5173    Emulator / Device
```

*Оба клиента используют один Mock API, но получают responses согласно своим mappings.*

---

### Сценарий D — Docker deployment frontend + mock API

Если в дальнейшем `emarket-web` также запускается в Docker:

```text
Docker Network: emarket-network

┌────────────────────────────────────┐
│                                    │
│   ┌────────────────────────────┐   │
│   │       emarket-web          │   │
│   │       Nginx                │   │
│   └────────────────────────────┘   │
│                 │                  │
│                 v                  │
│   ┌────────────────────────────┐   │
│   │       emarket-api-mock     │   │
│   │       WireMock:8080        │   │
│   └────────────────────────────┘   │
│                                    │
└────────────────────────────────────┘
```

Внутри Docker network адрес Mock API: `http://emarket-api-mock:8080`

> **Но если запрос отправляет браузер пользователя**, а не серверный компонент, использовать Docker service name нельзя. Браузеру нужен доступный ему host/domain URL, например `http://localhost:9090` в локальной среде.

---

## 10. Сравнение двух режимов

| Критерий | Dev mode | Deployment mode |
| :--- | :--- | :--- |
| **Запуск** | Java JAR | Docker |
| **Требуется Java на host** | Да | Нет |
| **Требуется Docker** | Нет | Да |
| **WireMock version** | 3.13.1 | 3.13.1 |
| **Port внутри процесса** | 9090 | 8080 |
| **Host port** | 9090 | 9090 |
| **Mappings** | Общие | Общие |
| **JSON responses** | Общие | Общие |
| **Изменение JSON** | Без перезапуска | Через volume |
| **CI/CD** | Необязательно | Удобно |
| **Изоляция окружения** | Нет | Да |

---

## 11. Рекомендуемые команды проекта

Предлагаю зафиксировать следующие команды в `README.md`:

| Команда | Назначение |
| :--- | :--- |
| `./scripts/start-host.sh` | Запустить Dev mode |
| `docker compose up -d` | Запустить Deployment mode |
| `docker compose down` | Остановить Docker |
| `docker compose logs -f` | Просмотр логов |
| `curl localhost:9090/__admin` | Проверить WireMock |
| `curl -X POST localhost:9090/__admin/mappings/reset` | Перезагрузить mappings из файлов |

> **Важно:** два режима нельзя запускать одновременно на одном host-порту `9090`. Для параллельного запуска потребуется изменить `MOCK_API_PORT` одного из режимов.

---

## 12. Итоговые архитектурные решения

Я рекомендую зафиксировать следующие правила для `emarket-api-mock`:

1. **WireMock** — единственный Mock API engine на Stage 1.
2. Поддерживаются два режима: **Dev** и **Deployment**.
3. **Dev mode** запускается через Java Standalone JAR.
4. **Deployment mode** запускается через Docker Compose.
5. **WireMock version** фиксируется и одинакова в обоих режимах.
6. **Mappings** и **JSON responses** общие для обоих режимов.
7. `web` и `mobile` имеют независимые mock scenarios.
8. Общие endpoints и error responses располагаются в `common`.
9. **Host port** — `9090`, **container port** — `8080`.
10. Frontend выбирает Mock API через environment configuration.
11. Mock API не содержит бизнес-логики и не зависит от Spring Boot.
12. Docker deployment не должен требовать изменения mock mappings.

---

### Моя рекомендация по организации репозиториев:

```text
emarket/
│
├── emarket-web/
├── emarket-mobile/
├── emarket-api-mock/
│
├── emarket-api-gateway/
├── emarket-bff-web/
├── emarket-identity-service/
├── emarket-user-service/
│
└── emarket-docs/
```

Так `emarket-api-mock` будет самостоятельным инфраструктурным репозиторием, который можно использовать как для локальной разработки, так и для CI/CD и deployment тестового окружения.

При появлении настоящего `emarket-api-gateway` frontend сможет переключиться с Mock API на Gateway только изменением Base URL, сохраняя API-контракты.

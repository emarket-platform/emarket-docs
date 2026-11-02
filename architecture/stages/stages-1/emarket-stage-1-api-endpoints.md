# emarket --- Stage 1: Complete API Endpoint Specification

Предлагаемая спецификация API первого этапа для пяти приложений:

- emarket-web --- React + TypeScript.
- emarket-api-gateway --- Spring Cloud Gateway.
- emarket-bff-web --- Web Backend for web.
- emarket-bff-mobile --- Web Backend for mobile app.
- emarket-identity-service --- Authentication & Authorizatio

Архитектурное правило: **Frontend работает только с API Gateway. BFF
предоставляет публичный Web API. Identity и User Service предоставляют
внутренние API, недоступные непосредственно из интернета.**

------------------------------------------------------------------------

## 1. Общая схема взаимодействия

``` mermaid
flowchart TB
    WEB["emarket-web<br/>React + TypeScript"]
    GW["emarket-api-gateway<br/>Public API"]
    BFF["emarket-web-bff<br/>Web API"]
    ID["emarket-identity-service<br/>Authentication / Authorization"]
    US["emarket-user-service<br/>Profile / Addresses / Preferences"]
    IDDB[("Identity DB")]
    USDB[("User DB")]
    REDIS[("Redis")]
    KAFKA[("Kafka")]

    WEB --> GW
    GW --> BFF
    BFF --> ID
    BFF --> US
    BFF <--> REDIS
    ID --> IDDB
    US --> USDB
    ID --> KAFKA
    KAFKA --> US
```

### Публичный URL сервиса emarket-api-gateway

два разных base URL одного и того же сервиса emarket-api-gateway в разных окружениях.

| Environment | API Gateway Base URL |
|        :--- | :--- |
| Development | `http://localhost:8080/api/v1` |
| Production | `https://api.emarket.com/api/v1` |

Например, endpoint регистрации:

Development:
```html
POST http://localhost:8080/api/v1/web/auth/register
```
Production:
```html
POST https://api.emarket.com/api/v1/web/auth/register
```

При этом:
- localhost:8080 — локальный API Gateway.
- api.emarket.com — публичный домен API Gateway в production.
- /api/v1 — версия публичного API.
- /web — маршрутизация запросов в emarket-web-bff.
- /auth/register — endpoint, реализованный в Web BFF.


Рекомендация: в React-приложении emarket-web храни base URL через environment variables Vite:

.env.development
```config
VITE_API_BASE_URL=http://localhost:8080/api/v1
```

.env.production
```config
VITE_API_BASE_URL=https://api.emarket.com/api/v1
```
Так frontend не зависит от конкретного адреса API Gateway.

### Правила маршрутизации

  Public API prefix     Destination
  --------------------- ---------------------------------------
  `/api/v1/web/**`      `emarket-web-bff`
  `/api/v1/mobile/**`   `emarket-mobile-bff` 

Gateway не должен знать о внутренних endpoints Identity и User Service.
Их знает BFF.

------------------------------------------------------------------------

## На emarket-api-gateway нет бизнес-endpoint-ов
У API Gateway нет собственных бизнес-контроллеров для регистрации, авторизации, получения профиля и других операций.

Его основные обязанности:
- Routing / Forwarding — перенаправление запросов.
- Authentication — проверка токена или сессии, если это предусмотрено архитектурой.
- Rate Limiting — ограничение количества запросов.
- CORS — настройка междоменных запросов.
- Logging / Tracing — логирование и трассировка.
- Load Balancing — распределение запросов между экземплярами сервисов.


При этом у Gateway могут быть технические endpoints, например Spring Boot Actuator /actuator/health и /actuator/metrics. Это не бизнес-endpoint-ы.

##  Маршруты Gateway прописаны в application.yml
Например POST /api/v1/web/auth/register

### 1. emarket-api-gateway
В API Gateway не реализует бизнес-эндпоинты.
- Его основные обязанности:
- Принимать входящие HTTP-запросы.
- Маршрутизировать запросы в соответствующие BFF.
- Rate Limiting.
- CORS.
- Централизованные технические фильтры.
- Обработка некоторых инфраструктурных ошибок.

### 2. Маршрутизация в application.yml
Например:
```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: emarket-web-bff
          uri: http://localhost:8081
          predicates:
            - Path=/api/v1/web/**

        - id: emarket-mobile-bff
          uri: http://localhost:8083
          predicates:
            - Path=/api/v1/mobile/**
```

Gateway просто определяет, куда направить запрос.
### 3. emarket-web-bff
Именно здесь реализуется endpoint.

Пример регистрации пользователя:
```html
POST /api/v1/web/auth/register
```

```java
@RestController
@RequestMapping("/api/v1/web/auth")
@RequiredArgsConstructor
public class AuthController {

    private final AuthService authService;

    @PostMapping("/register")
    public ResponseEntity<RegistrationResponse> register(
            @Valid @RequestBody RegistrationRequest request) {

        return ResponseEntity
                .status(HttpStatus.CREATED)
                .body(authService.register(request));
    }
}
```

### Итоговая архитектура 

        emarket-web
        React + TypeScript
    POST /api/v1/web/auth/register
            |
            V
        emarket-api-gateway
        Routing / Rate Limiting / CORS
    Path=/api/v1/web/**
            |
            V
        emarket-web-bff
        AuthController → AuthService
    POST /api/v1/web/auth/register
            |
            V
    emarket-identity-service
    Identity / Credentials / Authentication
    Создание пользователя

Важное уточнение: 

Gateway всё-таки имеет технические HTTP endpoints, например /actuator/health, если включён Spring Boot Actuator. Но бизнес-endpoint-ы вроде /auth/register, /auth/login и /users/me в нём не реализуются.

Итого: Gateway маршрутизирует, BFF реализует публичные API endpoints, микросервисы реализуют бизнес-логику.

| Компонент | Что делает |
| :--- | :--- |
| `emarket-web` | Отправляет запрос |
| `emarket-api-gateway` | Принимает и маршрутизирует запрос |
| `emarket-web-bff` | Реализует endpoint регистрации |
| `emarket-identity-service` | Создаёт пользователя и управляет его учётными данными |

## 2. Frontend --- emarket-web

Frontend не предоставляет backend REST endpoints. Он предоставляет
пользовательские страницы (React Router routes) и вызывает публичный
API.

### 2.1. Frontend routes

| HTTP | Frontend route | Назначение | Доступ |
| :--- | :--- | :--- | :--- |
| GET | `/` | Home page | Public |
| GET | `/login` | Login page | Public |
| GET | `/register` | Registration page | Public |
| GET | `/forgot-password` | Password recovery | Public |
| GET | `/reset-password` | Password reset | Public |
| GET | `/verify-email` | Email verification | Public |
| GET | `/profile` | User profile | Authenticated |
| GET | `/profile/addresses` | User addresses | Authenticated |
| GET | `/profile/preferences` | User preferences | Authenticated |

В BFF-архитектуре OAuth callback лучше обрабатывать на backend. Поэтому
отдельный React route `/auth/callback` не нужен, если callback
обрабатывает BFF.

### 2.2. Frontend API client

Рекомендуемая структура:

``` text
src/
├── app/
│   ├── router/
│   └── providers/
├── shared/
│   └── api/
│       ├── api-client.ts
│       ├── api-error.ts
│       └── api-types.ts
├── features/
│   ├── auth/
│   │   ├── api/
│   │   ├── components/
│   │   ├── hooks/
│   │   └── types/
│   └── user/
│       ├── profile/
│       ├── addresses/
│       └── preferences/
└── pages/
```

Base URL:

``` typescript
const API_BASE_URL = '/api/v1/web';
```

Для Vite development используется proxy:

``` text
React :5173
   |
   v
/api/v1/web/**
   |
   v
Gateway :8080
```

Настройки HTTP client:

``` typescript
fetch(url, {
  credentials: 'include',
  headers: {
    'Content-Type': 'application/json',
  },
});
```

Для изменяющих состояние запросов также передаётся CSRF token в
согласованном с BFF header, например `X-CSRF-TOKEN`.

Frontend никогда не должен самостоятельно добавлять JWT в
`Authorization`.

------------------------------------------------------------------------

## 3. API Gateway --- emarket-api-gateway

Gateway --- единственная публичная точка входа.

### 3.1. Публичный routing contract

| Method | Public endpoint | Destination |
| :--- | :--- | :--- |
| GET | `/api/v1/web/auth/csrf` | Web BFF |
| GET | `/api/v1/web/auth/login` | Web BFF |
| POST | `/api/v1/web/auth/register` | Web BFF |
| GET | `/api/v1/web/auth/callback` | Web BFF |
| POST | `/api/v1/web/auth/logout` | Web BFF |
| GET | `/api/v1/web/auth/session` | Web BFF |
| GET | `/api/v1/web/users/me` | Web BFF |
| PATCH | `/api/v1/web/users/me` | Web BFF |
| GET | `/api/v1/web/users/me/addresses` | Web BFF |
| POST | `/api/v1/web/users/me/addresses` | Web BFF |
| GET | `/api/v1/web/users/me/addresses/{addressId}` | Web BFF |
| PATCH | `/api/v1/web/users/me/addresses/{addressId}` | Web BFF |
| DELETE | `/api/v1/web/users/me/addresses/{addressId}` | Web BFF |
| GET | `/api/v1/web/users/me/preferences` | Web BFF |
| PUT | `/api/v1/web/users/me/preferences` | Web BFF |

Дополнительные endpoints Authentication, описанные ниже, также
маршрутизируются в BFF.

### 3.2. Обязанности Gateway

-   HTTPS.
-   Routing.
-   Rate Limiting.
-   CORS.
-   Request Size Limits.
-   Request ID / Correlation ID.
-   Timeout.
-   Circuit Breaker.
-   Security headers.
-   Access logging без чувствительных данных.

Gateway не должен реализовывать регистрацию, проверку пароля, управление
профилем или хранение сессий.

### 3.3. Management endpoints

Примеры:

  Endpoint                 Назначение
  ------------------------ -------------------------
  `/actuator/health`       Health check
  `/actuator/prometheus`   Metrics
  `/actuator/info`         Application information

Они доступны только из доверенной инфраструктуры или через защищённый
management interface.

------------------------------------------------------------------------

## 4. Web BFF --- emarket-web-bff

BFF --- единственный backend API, с которым взаимодействует React.

Публичный base path:

``` text
/api/v1/web
```

Внутренний base path:

``` text
/internal/v1
```

Группы endpoints:

1.  Authentication.
2.  Session Management.
3.  User Profile.
4.  User Addresses & Preferences.

### 4.1. Authentication endpoints

  Method   Endpoint           Назначение                   Auth
  -------- ------------------ ---------------------------- --------------------------
  GET      `/auth/login`      Начать процесс авторизации   Public
  POST     `/auth/register`   Регистрация пользователя     Public
  GET      `/auth/callback`   OAuth2 callback              Public, state validation
  POST     `/auth/logout`     Завершить сессию             Session
  GET      `/auth/session`    Получить состояние сессии    Session
  GET      `/auth/csrf`       Получить CSRF token          Public/session

#### POST `/auth/register`

Request:

``` json
{
  "email": "user@example.com",
  "password": "StrongPassword123!",
  "firstName": "Alex",
  "lastName": "Green"
}
```

Response: `202 Accepted`.

``` json
{
  "registrationId": "uuid",
  "status": "PENDING",
  "message": "Registration accepted"
}
```

BFF вызывает Identity Service. После успешной регистрации Identity
Service публикует `UserRegistered`, а User Service создаёт профиль.

Email verification можно выполнять отдельным процессом.

Возможные ошибки:

-   `400` --- validation error.
-   `409` --- email already registered, если выбран такой публичный
    контракт.
-   `429` --- rate limit exceeded.
-   `503` --- Identity Service unavailable.

Для защиты от user enumeration можно возвращать одинаковый ответ для
существующего и нового email.

#### GET `/auth/login`

Начинает OAuth2 Authorization Code Flow.

Пример:

``` http
GET /api/v1/web/auth/login?returnUrl=/profile
```

BFF:

1.  Генерирует `state`.
2.  Генерирует PKCE verifier/challenge.
3.  Сохраняет временный authorization context.
4.  Формирует redirect на Identity Service.
5.  Отправляет браузеру redirect.

Ответ:

``` http
HTTP/1.1 302 Found
Location: https://identity.example.com/oauth2/authorize?...
```

`returnUrl` должен проверяться по allowlist, чтобы исключить open
redirect.

#### GET `/auth/callback`

``` http
GET /api/v1/web/auth/callback?code=...&state=...
```

BFF:

1.  Проверяет `state`.
2.  Получает PKCE verifier.
3.  Обменивает authorization code на токены.
4.  Проверяет результат авторизации.
5.  Создаёт серверную сессию в Redis.
6.  Устанавливает session cookie.
7.  Перенаправляет пользователя на разрешённый frontend URL.

Ответ:

``` http
HTTP/1.1 302 Found
Set-Cookie: EMARKET_SESSION=opaque-id; HttpOnly; Secure; SameSite=Lax; Path=/
Location: /profile
```

#### POST `/auth/logout`

Request:

``` http
POST /api/v1/web/auth/logout
X-CSRF-TOKEN: ...
Cookie: EMARKET_SESSION=...
```

Response: `204 No Content`.

BFF:

-   Инвалидирует Redis session.
-   Отзывает Refresh Token через Identity Service.
-   Очищает session cookie.

#### GET `/auth/session`

Response: `200 OK`.

``` json
{
  "authenticated": true,
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "roles": ["CUSTOMER"]
  }
}
```

Если пользователь не авторизован:

``` json
{
  "authenticated": false,
  "user": null
}
```

Для этого endpoint можно использовать `200` в обоих случаях: он
возвращает состояние сессии, а не защищённый ресурс.

#### GET `/auth/csrf`

Response:

``` json
{
  "token": "csrf-token-value",
  "headerName": "X-CSRF-TOKEN"
}
```

CSRF token не является Access Token. BFF устанавливает или возвращает
его в соответствии с выбранной реализацией Spring Security.

### 4.2. User Profile endpoints

  Method   Endpoint      Назначение
  -------- ------------- ------------------
  GET      `/users/me`   Получить профиль
  PATCH    `/users/me`   Обновить профиль

#### GET `/users/me`

Request:

``` http
GET /api/v1/web/users/me
Cookie: EMARKET_SESSION=...
```

Response: `200 OK`.

``` json
{
  "id": "uuid",
  "email": "user@example.com",
  "firstName": "Alex",
  "lastName": "Green",
  "phone": "+37360000000",
  "status": "ACTIVE",
  "emailVerified": true,
  "createdAt": "2026-11-01T12:00:00Z",
  "updatedAt": "2026-11-01T12:00:00Z"
}
```

BFF получает User ID из проверенной сессии и обращается к User Service.

#### PATCH `/users/me`

Обновляет только переданные поля.

Request:

``` json
{
  "firstName": "Alexei",
  "lastName": "Green",
  "phone": "+37360000000"
}
```

Response: `200 OK`.

``` json
{
  "id": "uuid",
  "email": "user@example.com",
  "firstName": "Alexei",
  "lastName": "Green",
  "phone": "+37360000000",
  "status": "ACTIVE",
  "updatedAt": "2026-11-01T12:30:00Z"
}
```

Email и статус аккаунта нельзя менять этим endpoint. Изменение email
требует отдельного процесса подтверждения и синхронизации с Identity
Service.

### 4.3. User Addresses endpoints

  Method   Endpoint                            Назначение
  -------- ----------------------------------- ---------------------------
  GET      `/users/me/addresses`               Получить все адреса
  POST     `/users/me/addresses`               Добавить адрес
  GET      `/users/me/addresses/{addressId}`   Получить конкретный адрес
  PATCH    `/users/me/addresses/{addressId}`   Обновить адрес
  DELETE   `/users/me/addresses/{addressId}`   Удалить адрес

#### POST `/users/me/addresses`

Request:

``` json
{
  "addressType": "HOME",
  "country": "Moldova",
  "city": "Chisinau",
  "street": "Stefan cel Mare",
  "building": "100",
  "apartment": "25",
  "postalCode": "MD-2001",
  "isDefault": true
}
```

Response: `201 Created`.

``` json
{
  "id": "address-uuid",
  "addressType": "HOME",
  "country": "Moldova",
  "city": "Chisinau",
  "street": "Stefan cel Mare",
  "building": "100",
  "apartment": "25",
  "postalCode": "MD-2001",
  "isDefault": true,
  "createdAt": "2026-11-01T12:00:00Z"
}
```

#### GET `/users/me/addresses`

Response: `200 OK`.

``` json
{
  "items": [
    {
      "id": "address-uuid",
      "addressType": "HOME",
      "city": "Chisinau",
      "street": "Stefan cel Mare",
      "building": "100",
      "isDefault": true
    }
  ],
  "totalElements": 1
}
```

#### PATCH `/users/me/addresses/{addressId}`

Request:

``` json
{
  "street": "Dacia",
  "building": "10",
  "isDefault": true
}
```

Response: `200 OK`.

#### DELETE `/users/me/addresses/{addressId}`

Response: `204 No Content`.

BFF не принимает `userId` от frontend. User ID извлекается из сессии.

### 4.4. User Preferences endpoints

  Method   Endpoint                  Назначение
  -------- ------------------------- --------------------
  GET      `/users/me/preferences`   Получить настройки
  PUT      `/users/me/preferences`   Обновить настройки

#### GET `/users/me/preferences`

Response:

``` json
{
  "language": "en",
  "currency": "EUR",
  "timezone": "Europe/Chisinau",
  "theme": "LIGHT"
}
```

#### PUT `/users/me/preferences`

Request:

``` json
{
  "language": "en",
  "currency": "EUR",
  "timezone": "Europe/Chisinau",
  "theme": "DARK"
}
```

Response: `200 OK`.

``` json
{
  "language": "en",
  "currency": "EUR",
  "timezone": "Europe/Chisinau",
  "theme": "DARK",
  "updatedAt": "2026-11-01T12:30:00Z"
}
```

`PUT` означает полную замену представления настроек. Для частичного
обновления можно использовать `PATCH`.

------------------------------------------------------------------------

## 5. Identity Service --- emarket-identity-service

Identity Service владеет:

-   Identity accounts.
-   Credentials.
-   Passwords.
-   Authentication.
-   OAuth2/OIDC.
-   Access Tokens.
-   Refresh Tokens.
-   Roles.
-   Account authentication status.

Base path внутренних API:

``` text
/internal/v1
```

Кроме внутренних API, Identity Service предоставляет стандартные
OAuth2/OIDC protocol endpoints.

### 5.1. Internal Identity API

  --------------------------------------------------------------------------------------------------------
  Method                  Endpoint                                                 Назначение
  ----------------------- -------------------------------------------------------- -----------------------
  POST                    `/internal/v1/identities`                                Создать identity

  GET                     `/internal/v1/identities/{identityId}`                   Получить identity
                                                                                   metadata

  GET                     `/internal/v1/identities/by-user-id/{userId}`            Найти identity по User
                                                                                   ID

  POST                    `/internal/v1/identities/{identityId}/password/change`   Изменить пароль

  POST                    `/internal/v1/identities/{identityId}/password/reset`    Завершить сброс пароля

  POST                    `/internal/v1/identities/{identityId}/email/verify`      Подтвердить email

  POST                    `/internal/v1/identities/{identityId}/email/change`      Инициировать изменение
                                                                                   email

  POST                    `/internal/v1/identities/{identityId}/lock`              Заблокировать аккаунт

  POST                    `/internal/v1/identities/{identityId}/unlock`            Разблокировать аккаунт

  GET                     `/internal/v1/identities/{identityId}/roles`             Получить роли

  PUT                     `/internal/v1/identities/{identityId}/roles`             Изменить роли
  --------------------------------------------------------------------------------------------------------

Административные операции доступны только привилегированным внутренним
callers и не вызываются обычной пользовательской сессией.

#### POST `/internal/v1/identities`

Request:

``` json
{
  "email": "user@example.com",
  "password": "StrongPassword123!",
  "firstName": "Alex",
  "lastName": "Green"
}
```

Response: `201 Created`.

``` json
{
  "identityId": "identity-uuid",
  "userId": "user-uuid",
  "email": "user@example.com",
  "status": "PENDING_VERIFICATION",
  "createdAt": "2026-11-01T12:00:00Z"
}
```

Identity Service:

1.  Валидирует email и пароль.
2.  Хеширует пароль.
3.  Создаёт identity.
4.  Назначает начальную роль `CUSTOMER`.
5.  Создаёт Outbox event.
6.  Возвращает идентификаторы.

`userId` создаётся Identity Service и используется как идентификатор
профиля в User Service.

Пароль никогда не возвращается в response.

#### GET `/internal/v1/identities/{identityId}`

Response:

``` json
{
  "identityId": "identity-uuid",
  "userId": "user-uuid",
  "email": "user@example.com",
  "status": "ACTIVE",
  "emailVerified": true
}
```

Не возвращать password hash и секреты.

#### POST `/internal/v1/identities/{identityId}/password/change`

Request:

``` json
{
  "currentPassword": "OldPassword123!",
  "newPassword": "NewPassword123!"
}
```

Response: `204 No Content`.

Операция должна также отзывать остальные активные refresh token sessions
согласно принятой security policy.

#### POST `/internal/v1/identities/{identityId}/password/reset`

Request:

``` json
{
  "resetToken": "single-use-reset-token",
  "newPassword": "NewPassword123!"
}
```

Response: `204 No Content`.

Reset token должен иметь ограниченный срок действия и быть одноразовым.

#### POST `/internal/v1/identities/{identityId}/email/verify`

Request:

``` json
{
  "verificationToken": "email-verification-token"
}
```

Response: `204 No Content`.

#### POST `/internal/v1/identities/{identityId}/email/change`

Request:

``` json
{
  "newEmail": "new@example.com",
  "currentPassword": "CurrentPassword123!"
}
```

Response: `202 Accepted`.

Фактическое изменение email выполняется после подтверждения нового
адреса.

### 5.2. OAuth2 / OpenID Connect endpoints

  -------------------------------------------------------------------------------------------
  Method                  Endpoint                                    Назначение
  ----------------------- ------------------------------------------- -----------------------
  GET                     `/.well-known/openid-configuration`         OIDC Discovery

  GET                     `/.well-known/oauth-authorization-server`   Authorization Server
                                                                      Metadata

  GET                     `/oauth2/authorize`                         Authorization endpoint

  POST                    `/oauth2/token`                             Token endpoint

  GET                     `/oauth2/jwks`                              Public signing keys

  GET                     `/userinfo`                                 OIDC UserInfo

  POST                    `/oauth2/revoke`                            Token revocation

  POST                    `/oauth2/introspect`                        Token introspection

  GET                     `/login`                                    Identity login page

  POST                    `/login`                                    Submit login
                                                                      credentials
  -------------------------------------------------------------------------------------------

`/oauth2/introspect` нужен, если выбранный способ проверки токенов
требует introspection. Для локально проверяемых JWT User Service обычно
достаточно JWKS.

#### GET `/oauth2/authorize`

Пример параметров:

``` text
response_type=code
client_id=emarket-web-bff
redirect_uri=https://api.emarket.com/api/v1/web/auth/callback
scope=openid profile email
state=random-state
code_challenge=...
code_challenge_method=S256
```

Identity Service аутентифицирует пользователя и возвращает authorization
code через redirect.

#### POST `/oauth2/token`

BFF отправляет:

``` text
grant_type=authorization_code
client_id=emarket-web-bff
code=authorization-code
redirect_uri=...
code_verifier=...
```

Response: `200 OK`.

``` json
{
  "access_token": "access-token",
  "token_type": "Bearer",
  "expires_in": 600,
  "refresh_token": "refresh-token",
  "scope": "openid profile email",
  "id_token": "id-token"
}
```

#### POST `/oauth2/revoke`

BFF отзывает Refresh Token при logout.

#### GET `/oauth2/jwks`

Возвращает публичные ключи, которыми downstream сервисы проверяют JWT.

### 5.3. Login architecture

Не нужно создавать собственный публичный `POST /login` с передачей
пароля через React → BFF.

В рекомендуемом OIDC flow:

-   React начинает авторизацию через BFF.
-   Identity Service показывает login page.
-   Пользователь проходит аутентификацию у Identity Service.
-   BFF получает токены через authorization code.
-   React получает только session cookie.

Это позволяет не использовать OAuth2 Password Grant.

------------------------------------------------------------------------

## 6. User Service --- emarket-user-service

User Service отвечает исключительно за пользовательские бизнес-данные.

Base path:

``` text
/internal/v1
```

Внешние запросы через Gateway к этому сервису запрещены.

### 6.1. Profile API

  --------------------------------------------------------------------------------------
  Method                  Internal endpoint                      Назначение
  ----------------------- -------------------------------------- -----------------------
  POST                    `/internal/v1/users`                   Создать профиль

  GET                     `/internal/v1/users/{userId}`          Получить профиль

  PATCH                   `/internal/v1/users/{userId}`          Обновить профиль

  GET                     `/internal/v1/users/{userId}/status`   Получить статус профиля

  DELETE                  `/internal/v1/users/{userId}`          Удалить или
                                                                 деактивировать профиль
  --------------------------------------------------------------------------------------

#### POST `/internal/v1/users`

Вызывается Kafka consumer при обработке `UserRegistered` либо доверенным
provisioning-компонентом.

Request:

``` json
{
  "userId": "user-uuid",
  "email": "user@example.com",
  "firstName": "Alex",
  "lastName": "Green"
}
```

Response: `201 Created`.

``` json
{
  "id": "user-uuid",
  "email": "user@example.com",
  "firstName": "Alex",
  "lastName": "Green",
  "status": "ACTIVE",
  "createdAt": "2026-11-01T12:00:00Z"
}
```

При повторном событии с тем же `userId` профиль не должен создаваться
повторно.

### GET `/internal/v1/users/{userId}`

Response:

``` json
{
  "id": "user-uuid",
  "email": "user@example.com",
  "firstName": "Alex",
  "lastName": "Green",
  "phone": "+37360000000",
  "status": "ACTIVE",
  "createdAt": "2026-11-01T12:00:00Z",
  "updatedAt": "2026-11-01T12:00:00Z"
}
```

### PATCH `/internal/v1/users/{userId}`

Request:

``` json
{
  "firstName": "Alexei",
  "lastName": "Green",
  "phone": "+37360000000"
}
```

Response: `200 OK`.

BFF сначала проверяет пользовательскую сессию, затем вызывает этот
endpoint с проверяемым security context. User Service самостоятельно
проверяет, что caller имеет право изменять указанный профиль.

### 6.2. Addresses API

  -----------------------------------------------------------------------------------------------------
  Method                  Internal endpoint                                     Назначение
  ----------------------- ----------------------------------------------------- -----------------------
  GET                     `/internal/v1/users/{userId}/addresses`               Получить адреса

  POST                    `/internal/v1/users/{userId}/addresses`               Создать адрес

  GET                     `/internal/v1/users/{userId}/addresses/{addressId}`   Получить адрес

  PATCH                   `/internal/v1/users/{userId}/addresses/{addressId}`   Изменить адрес

  DELETE                  `/internal/v1/users/{userId}/addresses/{addressId}`   Удалить адрес
  -----------------------------------------------------------------------------------------------------

#### POST `/internal/v1/users/{userId}/addresses`

Request:

``` json
{
  "addressType": "HOME",
  "country": "Moldova",
  "city": "Chisinau",
  "street": "Stefan cel Mare",
  "building": "100",
  "apartment": "25",
  "postalCode": "MD-2001",
  "isDefault": true
}
```

Response: `201 Created`.

``` json
{
  "id": "address-uuid",
  "userId": "user-uuid",
  "addressType": "HOME",
  "country": "Moldova",
  "city": "Chisinau",
  "street": "Stefan cel Mare",
  "building": "100",
  "apartment": "25",
  "postalCode": "MD-2001",
  "isDefault": true
}
```

#### GET `/internal/v1/users/{userId}/addresses`

Response:

``` json
{
  "items": [
    {
      "id": "address-uuid",
      "addressType": "HOME",
      "country": "Moldova",
      "city": "Chisinau",
      "street": "Stefan cel Mare",
      "building": "100",
      "isDefault": true
    }
  ],
  "totalElements": 1
}
```

#### PATCH `/internal/v1/users/{userId}/addresses/{addressId}`

Request:

``` json
{
  "street": "Dacia",
  "building": "10",
  "isDefault": true
}
```

Response: `200 OK`.

#### DELETE `/internal/v1/users/{userId}/addresses/{addressId}`

Response: `204 No Content`.

Обязательно проверять принадлежность адреса пользователю.

### 6.3. Preferences API

  -------------------------------------------------------------------------------------------
  Method                  Internal endpoint                           Назначение
  ----------------------- ------------------------------------------- -----------------------
  GET                     `/internal/v1/users/{userId}/preferences`   Получить настройки

  PUT                     `/internal/v1/users/{userId}/preferences`   Полностью обновить
                                                                      настройки
  -------------------------------------------------------------------------------------------

#### GET `/internal/v1/users/{userId}/preferences`

Response:

``` json
{
  "userId": "user-uuid",
  "language": "en",
  "currency": "EUR",
  "timezone": "Europe/Chisinau",
  "theme": "LIGHT"
}
```

#### PUT `/internal/v1/users/{userId}/preferences`

Request:

``` json
{
  "language": "en",
  "currency": "EUR",
  "timezone": "Europe/Chisinau",
  "theme": "DARK"
}
```

Response: `200 OK`.

``` json
{
  "userId": "user-uuid",
  "language": "en",
  "currency": "EUR",
  "timezone": "Europe/Chisinau",
  "theme": "DARK",
  "updatedAt": "2026-11-01T12:30:00Z"
}
```

### 6.4. Взаимодействие BFF с User Service

Frontend вызывает:

``` http
GET /api/v1/web/users/me/addresses
```

BFF определяет `userId` из сессии и вызывает:

``` http
GET /internal/v1/users/{userId}/addresses
```

Публичный API и внутренний API имеют разные URL намеренно: BFF
предоставляет интерфейс для frontend, User Service --- внутренний
ресурсно-ориентированный API.

------------------------------------------------------------------------

## 7. Дополнительные Authentication endpoints

Для production-ready Identity API следует предусмотреть восстановление
пароля и подтверждение email.

### 7.1. Дополнения в Web BFF

  ------------------------------------------------------------------------------------
  Method                  Public endpoint                      Назначение
  ----------------------- ------------------------------------ -----------------------
  POST                    `/auth/password/forgot`              Запросить
                                                               восстановление пароля

  POST                    `/auth/password/reset`               Установить новый пароль

  POST                    `/auth/email/verification/resend`    Повторно отправить
                                                               verification email

  POST                    `/auth/email/verification/confirm`   Подтвердить email

  POST                    `/auth/password/change`              Изменить пароль
                                                               авторизованного
                                                               пользователя
  ------------------------------------------------------------------------------------

#### POST `/auth/password/forgot`

Request:

``` json
{
  "email": "user@example.com"
}
```

Response: `202 Accepted`.

``` json
{
  "message": "If the account exists, instructions will be sent."
}
```

#### POST `/auth/password/reset`

Request:

``` json
{
  "token": "password-reset-token",
  "newPassword": "NewPassword123!"
}
```

Response: `204 No Content`.

#### POST `/auth/email/verification/confirm`

Request:

``` json
{
  "token": "email-verification-token"
}
```

Response: `204 No Content`.

#### POST `/auth/password/change`

Request:

``` json
{
  "currentPassword": "CurrentPassword123!",
  "newPassword": "NewPassword123!"
}
```

Response: `204 No Content`.

Все эти операции маршрутизируются через BFF в Identity Service. Токены
восстановления и подтверждения должны быть случайными, одноразовыми, с
ограниченным сроком действия и храниться в Identity Service в безопасном
виде.

------------------------------------------------------------------------

## 8. Полная карта endpoints по сервисам

### 8.1. Frontend --- emarket-web

  Тип    Route
  ------ ------------------------
  Page   `/`
  Page   `/login`
  Page   `/register`
  Page   `/forgot-password`
  Page   `/reset-password`
  Page   `/verify-email`
  Page   `/profile`
  Page   `/profile/addresses`
  Page   `/profile/preferences`

### 8.2. API Gateway --- emarket-api-gateway

  Method   Route                    Target
  -------- ------------------------ ---------
  GET      `/api/v1/web/auth/**`    Web BFF
  POST     `/api/v1/web/auth/**`    Web BFF
  GET      `/api/v1/web/users/**`   Web BFF
  POST     `/api/v1/web/users/**`   Web BFF
  PATCH    `/api/v1/web/users/**`   Web BFF
  PUT      `/api/v1/web/users/**`   Web BFF
  DELETE   `/api/v1/web/users/**`   Web BFF

`**` --- логическая группировка маршрутов. В production лучше явно
определить разрешённые методы и пути, а не использовать безусловный
wildcard.

### 8.3. Web BFF --- emarket-web-bff

  Method   Endpoint
  -------- -----------------------------------------------
  GET      `/api/v1/web/auth/login`
  POST     `/api/v1/web/auth/register`
  GET      `/api/v1/web/auth/callback`
  POST     `/api/v1/web/auth/logout`
  GET      `/api/v1/web/auth/session`
  GET      `/api/v1/web/auth/csrf`
  POST     `/api/v1/web/auth/password/forgot`
  POST     `/api/v1/web/auth/password/reset`
  POST     `/api/v1/web/auth/password/change`
  POST     `/api/v1/web/auth/email/verification/resend`
  POST     `/api/v1/web/auth/email/verification/confirm`
  GET      `/api/v1/web/users/me`
  PATCH    `/api/v1/web/users/me`
  GET      `/api/v1/web/users/me/addresses`
  POST     `/api/v1/web/users/me/addresses`
  GET      `/api/v1/web/users/me/addresses/{addressId}`
  PATCH    `/api/v1/web/users/me/addresses/{addressId}`
  DELETE   `/api/v1/web/users/me/addresses/{addressId}`
  GET      `/api/v1/web/users/me/preferences`
  PUT      `/api/v1/web/users/me/preferences`

### 8.4. Identity Service --- emarket-identity-service

  Method   Endpoint
  -------- --------------------------------------------------------
  POST     `/internal/v1/identities`
  GET      `/internal/v1/identities/{identityId}`
  GET      `/internal/v1/identities/by-user-id/{userId}`
  POST     `/internal/v1/identities/{identityId}/password/change`
  POST     `/internal/v1/identities/{identityId}/password/reset`
  POST     `/internal/v1/identities/{identityId}/email/verify`
  POST     `/internal/v1/identities/{identityId}/email/change`
  POST     `/internal/v1/identities/{identityId}/lock`
  POST     `/internal/v1/identities/{identityId}/unlock`
  GET      `/internal/v1/identities/{identityId}/roles`
  PUT      `/internal/v1/identities/{identityId}/roles`
  GET      `/.well-known/openid-configuration`
  GET      `/.well-known/oauth-authorization-server`
  GET      `/oauth2/authorize`
  POST     `/oauth2/token`
  GET      `/oauth2/jwks`
  GET      `/userinfo`
  POST     `/oauth2/revoke`
  POST     `/oauth2/introspect`
  GET      `/login`
  POST     `/login`

### 8.5. User Service --- emarket-user-service

  Method   Endpoint
  -------- -----------------------------------------------------
  POST     `/internal/v1/users`
  GET      `/internal/v1/users/{userId}`
  PATCH    `/internal/v1/users/{userId}`
  GET      `/internal/v1/users/{userId}/status`
  DELETE   `/internal/v1/users/{userId}`
  GET      `/internal/v1/users/{userId}/addresses`
  POST     `/internal/v1/users/{userId}/addresses`
  GET      `/internal/v1/users/{userId}/addresses/{addressId}`
  PATCH    `/internal/v1/users/{userId}/addresses/{addressId}`
  DELETE   `/internal/v1/users/{userId}/addresses/{addressId}`
  GET      `/internal/v1/users/{userId}/preferences`
  PUT      `/internal/v1/users/{userId}/preferences`

------------------------------------------------------------------------

## 9. Kafka Events --- межсервисное взаимодействие

На первом этапе предлагаются следующие события:

  ------------------------------------------------------------------------------
  Event              Producer          Consumer          Назначение
  ------------------ ----------------- ----------------- -----------------------
  `UserRegistered`   Identity          User              Создание профиля

  `EmailVerified`    Identity          User              Обновление статуса
                                                         профиля, если нужно

  `UserBlocked`      Identity          User              Блокировка профиля

  `UserUnblocked`    Identity          User              Разблокировка профиля

  `UserDeleted`      Identity          User              Удаление/анонимизация
                                                         профиля
  ------------------------------------------------------------------------------

### Kafka topic naming

``` text
emarket.identity.user-registered.v1
emarket.identity.email-verified.v1
emarket.identity.user-blocked.v1
emarket.identity.user-unblocked.v1
emarket.identity.user-deleted.v1
```

### Пример события UserRegistered

``` json
{
  "eventId": "event-uuid",
  "eventType": "UserRegistered",
  "eventVersion": 1,
  "occurredAt": "2026-11-01T12:00:00Z",
  "producer": "emarket-identity-service",
  "payload": {
    "userId": "user-uuid",
    "email": "user@example.com",
    "firstName": "Alex",
    "lastName": "Green"
  }
}
```

Kafka consumer находится внутри `emarket-user-service`:

``` text
emarket-user-service
│
├── profile/
│   ├── UserProfileService.java
│   └── UserRegisteredEventHandler.java
│
├── messaging/
│   ├── UserEventConsumer.java
│   └── EventDeduplicationService.java
│
└── infrastructure/
    └── kafka/
```

Consumer вызывает `UserProfileService`, а не делает HTTP-запрос к
собственному REST API.

Поэтому `POST /internal/v1/users` не обязателен для Kafka. Его можно
оставить только для доверенного административного provisioning, если
такая потребность появится.

Production requirements:

-   Transactional Outbox на стороне Identity.
-   Idempotent Consumer на стороне User.
-   Повторная доставка событий.
-   Retry с backoff.
-   Dead Letter Topic.
-   Контроль порядка событий по `userId`.
-   Версионирование event schema.

------------------------------------------------------------------------

## 10. Общий формат ошибок API

Рекомендуется RFC 9457 --- Problem Details.

Пример:

``` http
HTTP/1.1 400 Bad Request
Content-Type: application/problem+json
```

``` json
{
  "type": "https://api.emarket.com/problems/validation-error",
  "title": "Validation failed",
  "status": 400,
  "detail": "One or more fields are invalid",
  "instance": "/api/v1/web/users/me",
  "code": "VALIDATION_ERROR",
  "traceId": "trace-uuid",
  "errors": [
    {
      "field": "firstName",
      "message": "First name must not be blank"
    }
  ]
}
```

### Рекомендуемые HTTP status codes

  Status                        Использование
  ----------------------------- --------------------------------------
  `200 OK`                      Успешное получение/изменение ресурса
  `201 Created`                 Ресурс создан
  `202 Accepted`                Асинхронная операция принята
  `204 No Content`              Успешная операция без response body
  `400 Bad Request`             Некорректный запрос
  `401 Unauthorized`            Нет действительной аутентификации
  `403 Forbidden`               Нет необходимых прав
  `404 Not Found`               Ресурс не найден
  `409 Conflict`                Конфликт состояния
  `422 Unprocessable Content`   Семантическая ошибка данных
  `429 Too Many Requests`       Rate limit
  `500 Internal Server Error`   Внутренняя ошибка
  `502 Bad Gateway`             Некорректный ответ upstream
  `503 Service Unavailable`     Сервис временно недоступен
  `504 Gateway Timeout`         Timeout upstream

Публичный API не должен возвращать внутренние stack traces, SQL errors
или подробности инфраструктуры.

------------------------------------------------------------------------

## 11. Security matrix

  Endpoint group                Anonymous   CUSTOMER   ADMIN   Internal service
  ---------------------------- ----------- ---------- ------- ------------------
  Registration                      ✓          ✓         ✓    
  Login                             ✓          ✓         ✓    
  Logout                                       ✓         ✓    
  Get own profile                              ✓         ✓    
  Update own profile                           ✓         ✓    
  Manage own addresses                         ✓         ✓    
  Manage own preferences                       ✓         ✓    
  Change user roles                                      ✓    
  Create identity internally                                          ✓
  Read identity metadata                                              ✓

ADMIN не получает автоматический доступ к любым персональным данным
только из-за своей роли. Для административного доступа нужны отдельные
permissions и аудит.

Для внутренних вызовов используем service authentication: OAuth2 Client
Credentials с ограниченными scopes и/или mTLS. Для пользовательских
операций также передаём проверяемый user security context.

------------------------------------------------------------------------

## 12. Итоговые Request Flows

### Scenario A --- Register User

1.  React вызывает `POST /api/v1/web/auth/register`.
2.  Gateway маршрутизирует запрос в Web BFF.
3.  BFF вызывает Identity Service.
4.  Identity создаёт аккаунт и Outbox event.
5.  Kafka доставляет `UserRegistered` в User Service.
6.  User Service создаёт профиль.

### Scenario B --- Login User

1.  React выполняет browser navigation на `/api/v1/web/auth/login`.
2.  Gateway перенаправляет запрос в BFF.
3.  BFF начинает OAuth2 Authorization Code Flow.
4.  Identity аутентифицирует пользователя.
5.  BFF получает токены и сохраняет их в Redis.
6.  BFF устанавливает HttpOnly Session Cookie.

### Scenario C --- Get User Data

1.  React вызывает `GET /api/v1/web/users/me`.
2.  Gateway маршрутизирует запрос в BFF.
3.  BFF проверяет Session Cookie.
4.  BFF определяет User ID.
5.  BFF вызывает User Service.
6.  User Service проверяет authorization и загружает профиль.
7.  BFF возвращает DTO в React.

------------------------------------------------------------------------

## 13. Production decisions

  Вопрос                                  Архитектурное решение
  --------------------------------------- --------------------------------------
  Где реализован Login?                   Identity Service
  Где реализован Registration?            Identity Service
  Где реализован Get User Data?           User Service
  Где находятся публичные endpoints?      Web BFF
  Где хранится JWT?                       Backend, в BFF session storage
  Где хранится Session?                   Redis
  Кто создаёт User Profile?               User Service через Kafka event
  Кто создаёт User ID?                    Identity Service
  Где хранятся пароли?                    Identity PostgreSQL
  Где хранятся пользовательские данные?   User PostgreSQL
  Кто проверяет JWT?                      Каждый защищённый downstream service
  Кто выполняет routing?                  API Gateway

Дополнительные решения:

1.  Не создавать Mobile BFF на первом этапе --- оставить возможность
    подключения через `/api/v1/mobile/**`.
2.  Не объединять Identity и User Service.
3.  Не хранить JWT в браузере --- использовать BFF Session Pattern.
4.  Не использовать общую PostgreSQL database для всех микросервисов.
5.  Не вводить синхронные вызовы между всеми сервисами без
    необходимости.
6.  Реализовать Transactional Outbox для событий Identity → User.
7.  Не усложнять Stage 1 лишними бизнес-сервисами --- сначала получить
    работающие Registration, Login и User Profile.
8.  Сохранять независимость CI/CD и deployment каждого микросервиса.

------------------------------------------------------------------------

## 14. Рекомендуемый MVP implementation scope

Не нужно сразу реализовывать все endpoints. Для первого работающего
вертикального среза достаточно:

  -----------------------------------------------------------------------
  Сервис                              Endpoints / capabilities
  ----------------------------------- -----------------------------------
  API Gateway                         Routing `/api/v1/web/**`

  Web BFF                             Register, Login initiation,
                                      Callback, Logout, Session

  Identity                            Registration, OAuth2 Authorization,
                                      Token, JWKS, Revocation

  User                                Get Profile, Update Profile

  Kafka                               `UserRegistered`

  React                               Login, Register, Profile
  -----------------------------------------------------------------------

Затем добавляются:

-   Email verification.
-   Password recovery.
-   Addresses.
-   Preferences.
-   RBAC administration.
-   Расширенный monitoring и нагрузочное тестирование.

### Рекомендуемая структура документации

``` text
emarket-docs/
└── api/
    ├── public-api.md
    ├── web-bff-api.md
    ├── identity-service-api.md
    ├── user-service-api.md
    └── kafka-events.md
```

OpenAPI спецификацию (`openapi.yaml`) хранить непосредственно в каждом
backend repository, чтобы API-контракт развивался вместе с
соответствующим микросервисом.

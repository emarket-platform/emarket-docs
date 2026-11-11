# emarket --- Web и Mobile HTTP Requests

Документ описывает, как выглядят HTTP-запросы от React Web и нативного
Mobile-клиента, как API Gateway выбирает BFF и как отличаются модели
аутентификации.

## 1. Общая схема

``` text
React Web
  |
  | https://api.emarket.com/api/v1/web/...
  v
emarket-api-gateway
  |
  v
emarket-bff-web
  |
  +--> emarket-identity-service
  +--> emarket-user-service
  +--> emarket-product-service


Mobile App
  |
  | https://api.emarket.com/api/v1/mobile/...
  v
emarket-api-gateway
  |
  v
emarket-bff-mobile
  |
  +--> emarket-identity-service
  +--> emarket-product-service
  +--> emarket-order-service
```

Gateway выбирает маршрут по URL path, а не по `User-Agent` или
`X-Client-Type`.

  Клиент                Публичный URL prefix              Назначение
  --------------------- --------------------------------- --------------
  
  React Web             `/api/v1/web/**`                  Web BFF

  Mobile                `/api/v1/mobile/**`               Mobile BFF

## Причина создания emarket-bff-web и emarket-bff-mobile
Возвращать ответы заточенные для React клиента и мобильного приложения.
Это одна из главных причин использования BFF (Backend for Frontend).
emarket-bff-web и emarket-bff-mobile позволяют предоставлять разные API и форматы ответов, оптимизированные под конкретных клиентов.

Но есть и другие важные причины.
#### 1. Разные клиенты — разные потребности
Представим, что пользователь открывает страницу своего профиля.

React Web (emarket-bff-web)

#### Web-приложению нужны:
- User profile.
- Email.
- Addresses.
- Preferences.
- Avatar URL.
  
Пример ответа:
```json
{
  "id": "user-123",
  "firstName": "Alex",
  "lastName": "Smith",
  "email": "alex@example.com",
  "avatarUrl": "https://cdn.emarket.com/avatars/123.jpg",
  "addresses": [
    {
      "id": "address-1",
      "city": "Chisinau",
      "street": "Stefan cel Mare"
    }
  ],
  "preferences": {
    "language": "en",
    "theme": "dark"
  }
}
```

#### Mobile (emarket-bff-mobile)

Мобильному приложению может быть нужен другой формат:

```json
{
  "user": {
    "id": "user-123",
    "name": "Alex Smith",
    "avatar": "https://cdn.emarket.com/avatars/123.jpg"
  },
  "defaultAddress": {
    "city": "Chisinau",
    "street": "Stefan cel Mare"
  }
}
```
Например, мобильному приложению не обязательно загружать все адреса и настройки при каждом открытии профиля.

### 2. Другие преимущества BFF
| Преимущество | Описание |
| :--- | :--- |
| **Different API contracts** | Разные DTO и JSON responses для Web и Mobile |
| **API aggregation** | BFF объединяет ответы нескольких микросервисов |
| **Performance** | Уменьшение количества запросов от клиента |
| **Payload optimization** | Передача только необходимых данных |
| **Client-specific logic** | Логика, специфичная для Web или Mobile |
| **Independent evolution** | Web и Mobile API можно развивать независимо |
| **Security** | Контроль доступа и управление сессиями для конкретного клиента |

### 3. Нужно ли дублировать бизнес-логику в BFF?
Нет.

Например, расчёт стоимости заказа, проверка остатков товара и применение скидок должны находиться в соответствующих бизнес-микросервисах.

BFF отвечает за:
- адаптацию API под клиента;
- агрегацию данных;
- преобразование DTO;
- client-specific orchestration.
  
Он не должен становиться ещё одним местом реализации бизнес-правил marketplace.

Именно поэтому BFF — это не просто дополнительный proxy. Это отдельный backend-слой, адаптирующий API под конкретный frontend-клиент.

## Запросы от React Web

В выбранной True BFF модели браузер не получает OAuth access token или
refresh token. Web BFF хранит токены на серверной стороне, а браузер
использует защищённую session cookie.

###  Получить профиль пользователя

React отправляет запрос:

``` http
GET /api/v1/web/users/me HTTP/1.1
Host: api.emarket.com
Accept: application/json
Cookie: __Host-emarket-session=<session-id>
X-Request-ID: 6f7b9a20-4f21-4f51-9d91-2a4d9b6f4e11
```

Маршрут:

``` text
Browser
  -> API Gateway: /api/v1/web/users/me
  -> emarket-bff-web
  -> emarket-user-service
```

### 2.2. Обновить профиль (пример)

``` http
PATCH /api/v1/web/users/me HTTP/1.1
Host: api.emarket.com
Content-Type: application/json
Accept: application/json
Cookie: __Host-emarket-session=<session-id>
X-CSRF-TOKEN: <csrf-token>
```

``` json
{
  "firstName": "Alex",
  "lastName": "Green",
  "locale": "en",
  "theme": "dark"
}
```


Поскольку браузер использует cookie-based authentication, изменяющие
состояние запросы должны быть защищены от CSRF. Способ получения и
передачи CSRF-токена должен быть согласован с Web BFF.


Пример ответа, адаптированного под Mobile:

``` json
{
  "id": "usr_123",
  "displayName": "Alex Smith",
  "avatarUrl": "https://cdn.emarket.com/avatars/usr_123.jpg"
}
```

## Архитектурное решение

Для emarket фиксируем:

-   Web: React → API Gateway → Web BFF → Business Services.
-   Mobile: Mobile App → API Gateway → Mobile BFF → Business Services.
-   Gateway маршрутизирует по URL prefix.
-   Каждый backend Resource Server независимо проверяет токен и права
    доступа.


# 2. Frontend — emarket-web

Frontend не предоставляет backend REST endpoints. Он предоставляет пользовательские страницы (React Router routes) и вызывает публичный API.

## Frontend endpoints

Предлагаю следующую структуру URL:

| HTTP | Frontend route | Назначение | Доступ |
| :--- | :--- | :--- | :--- |
| `GET` | `/` | Home page | Public |
| `GET` | `/login` | Login page | Public |
| `GET` | `/register` | Registration page | Public |
| `GET` | `/forgot-password` | Password recovery | Public |
| `GET` | `/reset-password` | Password reset | Public |
| `GET` | `/verify-email` | Email verification | Public |
| `GET` | `/profile` | User profile | Authenticated |
| `GET` | `/profile/addresses` | User addresses | Authenticated |
| `GET` | `/profile/preferences` | User preferences | Authenticated |

# API Documentation

Все запросы направляются в `emarket-api-gateway`, который маршрутизирует `/api/v1/web/**` в `emarket-web-bff`.

---

## 2. Authentication & Registration

| REST Method | Name | Route | Description | Destination URL (Development) |
| :--- | :--- | :--- | :--- | :--- |
| `GET` | Start Login | `/api/v1/web/auth/login` | Начать процесс авторизации. Redirect на Identity Provider | `http://localhost:8080/api/v1/web/auth/login` |
| `POST` | Register User | `/api/v1/web/auth/register` | Регистрация нового пользователя | `http://localhost:8080/api/v1/web/auth/register` |
| `GET` | OAuth Callback | `/api/v1/web/auth/callback` | Обработка callback после OAuth2/OIDC авторизации | `http://localhost:8080/api/v1/web/auth/callback` |
| `POST` | Logout User | `/api/v1/web/auth/logout` | Завершение пользовательской сессии | `http://localhost:8080/api/v1/web/auth/logout` |
| `GET` | Get Auth Session | `/api/v1/web/auth/session` | Проверка текущей сессии пользователя | `http://localhost:8080/api/v1/web/auth/session` |
| `GET` | Get CSRF Token | `/api/v1/web/auth/csrf` | Получение CSRF-токена для защиты запросов | `http://localhost:8080/api/v1/web/auth/csrf` |

---

## 3. Password Management

| REST Method | Name | Route | Description | Destination URL (Development) |
| :--- | :--- | :--- | :--- | :--- |
| `POST` | Forgot Password | `/api/v1/web/auth/password/forgot` | Запросить восстановление пароля | `http://localhost:8080/api/v1/web/auth/password/forgot` |
| `POST` | Reset Password | `/api/v1/web/auth/password/reset` | Установить новый пароль по recovery token | `http://localhost:8080/api/v1/web/auth/password/reset` |
| `POST` | Change Password | `/api/v1/web/auth/password/change` | Изменить пароль авторизованного пользователя | `http://localhost:8080/api/v1/web/auth/password/change` |

---

## 4. Email Verification

| REST Method | Name | Route | Description | Destination URL (Development) |
| :--- | :--- | :--- | :--- | :--- |
| `POST` | Resend Verification Email | `/api/v1/web/auth/email/verification/resend` | Повторно отправить письмо подтверждения email | `http://localhost:8080/api/v1/web/auth/email/verification/resend` |
| `POST` | Confirm Email | `/api/v1/web/auth/email/verification/confirm` | Подтвердить email с помощью verification token | `http://localhost:8080/api/v1/web/auth/email/verification/confirm` |

---

## 5. User Profile

| REST Method | Name | Route | Description | Destination URL (Development) |
| :--- | :--- | :--- | :--- | :--- |
| `GET` | Get My Profile | `/api/v1/web/users/me` | Получить профиль текущего пользователя | `http://localhost:8080/api/v1/web/users/me` |
| `PATCH` | Update My Profile | `/api/v1/web/users/me` | Обновить данные профиля текущего пользователя | `http://localhost:8080/api/v1/web/users/me` |

---

## 6. User Addresses

| REST Method | Name | Route | Description | Destination URL (Development) |
| :--- | :--- | :--- | :--- | :--- |
| `GET` | Get My Addresses | `/api/v1/web/users/me/addresses` | Получить список адресов пользователя | `http://localhost:8080/api/v1/web/users/me/addresses` |
| `POST` | Create Address | `/api/v1/web/users/me/addresses` | Добавить новый адрес | `http://localhost:8080/api/v1/web/users/me/addresses` |
| `GET` | Get Address | `/api/v1/web/users/me/addresses/{addressId}` | Получить конкретный адрес | `http://localhost:8080/api/v1/web/users/me/addresses/{addressId}` |
| `PATCH` | Update Address | `/api/v1/web/users/me/addresses/{addressId}` | Изменить адрес | `http://localhost:8080/api/v1/web/users/me/addresses/{addressId}` |
| `DELETE` | Delete Address | `/api/v1/web/users/me/addresses/{addressId}` | Удалить адрес | `http://localhost:8080/api/v1/web/users/me/addresses/{addressId}` |

---

## 7. User Preferences

| REST Method | Name | Route | Description | Destination URL (Development) |
| :--- | :--- | :--- | :--- | :--- |
| `GET` | Get My Preferences | `/api/v1/web/users/me/preferences` | Получить настройки пользователя | `http://localhost:8080/api/v1/web/users/me/preferences` |
| `PUT` | Update My Preferences | `/api/v1/web/users/me/preferences` | Полностью обновить настройки пользователя | `http://localhost:8080/api/v1/web/users/me/preferences` |

---

## 8. Production URLs

В Production меняется только **Base URL**. Остальные маршруты остаются прежними.

### Примеры:

| Name | Development | Production |
| :--- | :--- | :--- |
| Register User | `http://localhost:8080/api/v1/web/auth/register` | `https://api.emarket.com/api/v1/web/auth/register` |
| Get My Profile | `http://localhost:8080/api/v1/web/users/me` | `https://api.emarket.com/api/v1/web/users/me` |
| Get My Addresses | `http://localhost:8080/api/v1/web/users/me/addresses` | `https://api.emarket.com/api/v1/web/users/me/addresses` |
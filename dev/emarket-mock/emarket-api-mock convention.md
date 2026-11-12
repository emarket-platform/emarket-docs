# Конвенция именования для `emarket-api-mock`

Для `emarket-api-mock` рекомендуется использовать **resource-oriented naming convention**, основанную на HTTP method, ресурсе и бизнес-сценарии.

> **Главный принцип:** имя JSON-файла должно позволять понять, какой endpoint он имитирует, без необходимости открывать файл.

---

## 1. Рекомендуемая конвенция

**Формат:**
```text
{http-method}-{resource}-{scenario}.json
```

### Правила
* Только `kebab-case`.
* HTTP method — в `lowercase`.
* Один файл — один WireMock mapping.
* Сценарий указывается, если есть несколько вариантов ответа.
* Не использовать версии API в имени файла, если они уже отражены в структуре каталогов.
* Не использовать случайные имена вроде `test1.json`, `mock-user.json`, `response.json`.

### Примеры

| Endpoint | Имя файла |
| :--- | :--- |
| `GET /users/me` | `get-current-user.json` |
| `GET /products` | `get-products.json` |
| `GET /products/{id}` | `get-product-by-id.json` |
| `POST /auth/login` | `post-login.json` |
| `POST /auth/logout` | `post-logout.json` |
| `POST /users` | `post-create-user.json` |

---

## 2. Структура каталогов

Рекомендуемая структура проекта:

```text
emarket-api-mock/
├── mappings/
│   └── web/
│       ├── auth/
│       │   ├── post-login.json
│       │   ├── post-login-invalid-credentials.json
│       │   ├── post-logout.json
│       │   └── post-refresh-token.json
│       │
│       ├── users/
│       │   ├── get-current-user.json
│       │   ├── get-user-by-id.json
│       │   ├── post-create-user.json
│       │   ├── patch-update-user.json
│       │   └── delete-user-by-id.json
│       │
│       ├── products/
│       │   ├── get-products.json
│       │   ├── get-products-empty.json
│       │   ├── get-product-by-id.json
│       │   ├── post-create-product.json
│       │   ├── put-update-product.json
│       │   └── delete-product-by-id.json
│       │
│       └── categories/
│           ├── get-categories.json
│           └── get-category-by-id.json
│
├── __files/
│   └── web/
│       ├── auth/
│       ├── users/
│       ├── products/
│       └── categories/
│
├── openapi/
│   └── openapi.yaml
│
├── scripts/
│   └── list-endpoints.sh
│
├── Dockerfile
└── docker-compose.yml
```

---

## 3. Конвенция именования сценариев

Для одного endpoint могут существовать разные сценарии. Рекомендуется использовать следующие суффиксы:

| Сценарий | Суффикс |
| :--- | :--- |
| Успешный ответ | `success` |
| Пустой результат | `empty` |
| Ресурс не найден | `not-found` |
| Некорректный запрос | `bad-request` |
| Не авторизован | `unauthorized` |
| Нет прав доступа | `forbidden` |
| Конфликт | `conflict` |
| Внутренняя ошибка | `server-error` |

### Пример организации сценариев

```text
products/
├── get-products.json
├── get-products-empty.json
├── get-product-by-id.json
├── get-product-by-id-not-found.json
└── post-create-product-conflict.json
```

> **Примечание:** Не нужно создавать отдельный файл с суффиксом `success`, если успешный сценарий является единственным для endpoint.

---

## 4. Конвенция для response body файлов

В директории `__files` рекомендуется использовать аналогичные понятные имена:

```text
__files/web/products/
├── products-list.json
├── products-empty.json
├── product-details.json
└── product-not-found.json
```

### Пример файла mapping

```json
{
  "request": {
    "method": "GET",
    "url": "/api/v1/web/products"
  },
  "response": {
    "status": 200,
    "headers": {
      "Content-Type": "application/json"
    },
    "bodyFileName": "web/products/products-list.json"
  }
}
```

---

## 5. Дополнительные правила

| Элемент | Конвенция |
| :--- | :--- |
| **Каталоги** | `kebab-case` |
| **Mapping files** | `kebab-case.json` |
| **Response files** | `kebab-case.json` |
| **HTTP methods** | `lowercase` |
| **API versions** | В URL и структуре каталогов, если нужны |
| **Один mapping** | Один файл |
| **Один response body** | Один файл |

---

## 6. Итоговая рекомендация

> **Recommended:** Выбрать формат `{http-method}-{resource}-{scenario}.json`

Для `emarket-platform` это даст несколько преимуществ:
1. Быстрый поиск mappings по HTTP method и ресурсу.
2. Простое добавление новых API endpoints.
3. Удобное разделение сценариев для frontend-разработки.
4. Понятная структура при code review.
5. Возможность автоматической проверки naming convention в GitHub Actions.

**Важное правило:** имя mapping-файла должно соответствовать HTTP method и endpoint, а не имени React-компонента или frontend-страницы. Так `emarket-api-mock` останется независимым от реализации `emarket-web` и сможет использоваться другими клиентами платформы.

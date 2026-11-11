# Конвенция наименования GitHub Issues для `emarket-platform`

Для `emarket-platform` рекомендуется использовать единый формат названий GitHub Issues для всех репозиториев с обязательным префиксом сервиса и типом задачи.

Это особенно важно, поскольку все Issues из семи репозиториев будут отображаться в одном **GitHub Project**. По названию должно быть понятно, к какому микросервису относится задача, даже без просмотра поля *Repository*.

---

## 1. Единый формат названия

### Рекомендуемая конвенция
```text
[EM-REPOSITORY][TYPE] Description
```

*Например:*
```text
[EM-WEB][TASK] Configure FSD architecture
```

### Структура названия

| Часть | Назначение |
| :--- | :--- |
| `EM` | Emarket platform |
| `REPOSITORY` | Идентификатор сервиса |
| `TYPE` | Тип задачи |
| `Description` | Краткое описание задачи |

---

## 2. Конвенция идентификаторов репозиториев

Предлагается зафиксировать следующие коды:

| GitHub Repository | Repository Code |
| :--- | :--- |
| `emarket-web` | `EM-WEB` |
| `emarket-api-gateway` | `EM-GATEWAY` |
| `emarket-bff-web` | `EM-BFF-WEB` |
| `emarket-bff-mobile` | `EM-BFF-MOBILE` |
| `emarket-identity-service` | `EM-IDENTITY` |
| `emarket-user-service` | `EM-USER` |
| `emarket-docs` | `EM-DOCS` |

**Преимущества:**
* Короткие идентификаторы.
* Однозначная идентификация сервиса.
* Удобная фильтрация в GitHub Project.
* Независимость от GitHub Issue Number.

---

## 3. Конвенция типов задач

Используйте следующие префиксы:

| Issue Type | Title Prefix | Назначение |
| :--- | :--- | :--- |
| **Feature** | `[FEATURE]` | Новая функциональность |
| **Bug** | `[BUG]` | Исправление ошибки |
| **Task** | `[TASK]` | Техническая задача |
| **Refactoring** | `[REFACTOR]` | Рефакторинг |
| **Documentation** | `[DOCS]` | Документация |

> **Важно:** префикс в названии не заменяет встроенное поле *GitHub Issue Type*. Оба значения должны соответствовать друг другу.

---

## 4. Примеры для `emarket` (`emarket-web`)

| Issue Type | GitHub Issue Title |
| :--- | :--- |
| **Feature** | `[EM-WEB][FEATURE] Implement user registration page` |
| **Bug** | `[EM-WEB][BUG] Fix login form validation` |
| **Task** | `[EM-WEB][TASK] Configure Vitest testing environment` |
| **Refactoring** | `[EM-WEB][REFACTOR] Align project structure with FSD` |
| **Documentation** | `[EM-WEB][DOCS] Document frontend architecture` |
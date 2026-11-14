# emarket — Production-ready Pull Request Naming Convention

Для проекта **emarket** рекомендуется использовать следующую комбинацию стандартов и инструментов:

- **Conventional Commits** — для commit messages.
- **Conventional Pull Requests** — для названий Pull Requests.
- **GitHub Pull Request Template** — для единой структуры описания PR.
- **GitHub Actions** — для автоматической проверки названий.
- **Branch protection rules** — для обязательного прохождения CI и Code Review.

Учитывая, что каждый микросервис `emarket` находится в отдельном GitHub-репозитории, конвенция должна быть одинаковой во всех репозиториях.

---

## 1. Формат названия Pull Request

Используется следующий формат:

```text
<type>(<scope>): <description>
```

| Элемент | Назначение |
| :--- | :--- |
| `type` | Тип изменений |
| `scope` | Компонент или область изменений (опционально) |
| `description` | Краткое описание изменений |

### Примеры

- `feat(auth): implement login functionality`
- `fix(users): resolve user registration validation`
- `refactor(security): simplify JWT token validation`
- `test(auth): add integration tests for login`
- `docs(api): update authentication API documentation`
- `ci(github): add pull request validation workflow`
- `build(gradle): update Spring Boot dependencies`

> **Правило:** название PR пишется на английском языке, в `lowercase`, в повелительном наклонении (imperative style), без точки в конце.

---

## 2. Допустимые PR types

Используется фиксированный список типов:

| Type | Когда использовать | Пример |
| :--- | :--- | :--- |
| `feat` | Новая функциональность | `feat(auth): implement login` |
| `fix` | Исправление ошибки | `fix(users): handle duplicate email` |
| `refactor` | Рефакторинг без изменения поведения | `refactor(auth): simplify token validation` |
| `perf` | Оптимизация производительности | `perf(products): optimize product search` |
| `test` | Добавление или изменение тестов | `test(users): add service unit tests` |
| `docs` | Документация | `docs(api): document user endpoints` |
| `build` | Изменение зависимостей и сборки | `build(gradle): update dependencies` |
| `ci` | CI/CD и GitHub Actions | `ci(github): add build workflow` |
| `chore` | Техническое обслуживание проекта | `chore(project): update gitignore` |
| `style` | Изменения форматирования без изменения логики | `style(java): apply spotless formatting` |
| `revert` | Отмена предыдущих изменений | `revert(auth): revert login changes` |

> **Примечание:** Не рекомендуется добавлять `hotfix`, `release`, `merge` как самостоятельные типы PR. Для срочных исправлений достаточно `fix`.

---

## 3. Правила scope

`scope` определяет область приложения, в которой выполняются изменения.

### Рекомендуемые значения для emarket

- **Backend:** `api`, `auth`, `security`, `users`, `products`, `orders`, `payments`, `notifications`, `database`, `messaging`
- **Frontend:** `ui`, `routing`, `forms`, `auth`, `users`, `products`, `state`, `api`, `i18n`
- **Infrastructure:** `docker`, `github`, `gradle`, `config`, `monitoring`, `logging`
- **Documentation:** `architecture`, `api`, `deployment`, `development`

### Правила оформления scope

1. Указывается в единственном числе, если это естественно для компонента.
2. Используется `lowercase`.
3. Без пробелов.
4. Необходимо выбирать наиболее конкретную область изменений.
5. Если PR затрагивает несколько независимых областей, можно использовать общий scope, например `api`, `security` или `project`.

### Примеры

- `feat(auth): implement refresh token rotation`
- `fix(database): resolve migration execution order`
- `ci(github): enforce pull request title convention`

---

## 4. Связь PR с Git Branch

Единая система именования веток:

```text
<type>/<issue-id>-<short-description>
```

### Примеры веток

- `feat/EM-123-user-registration`
- `fix/EM-124-login-validation`
- `refactor/EM-125-jwt-service`
- `test/EM-126-user-service`
- `docs/EM-127-api-documentation`

### Соответствие Branch и Pull Request

| Branch | Pull Request |
| :--- | :--- |
| `feat/EM-123-user-registration` | `feat(auth): implement user registration` |
| `fix/EM-124-login-validation` | `fix(auth): resolve login validation` |
| `refactor/EM-125-jwt-service` | `refactor(security): simplify JWT service` |

> **Обратите внимание:** ID задачи не обязателен в PR title, но должен присутствовать в названии ветки, если используется система задач. Для небольшого проекта без отдельного issue tracker допустимы ветки без ID.

---

## 5. Правила написания description

- Английский язык.
- `lowercase`.
- Imperative mood (`implement`, `add`, `update`, `remove`, `fix`).
- Желательная длина — максимум 72–80 символов.
- Без точки в конце.
- Запрещено использовать неопределённые описания (`changes`, `updates`, `improvements`).

### Примеры

**Плохо:**
- `feat(auth): authentication changes`
- `fix(users): some bug fixes`
- `update products`
- `changes in security`

**Хорошо:**
- `feat(auth): implement JWT authentication`
- `fix(users): prevent duplicate account creation`
- `refactor(products): extract product validation logic`

---

## 6. GitHub Pull Request Template

В каждом репозитории необходимо создать файл `.github/PULL_REQUEST_TEMPLATE.md`:

```markdown
## Description

<!-- Describe what this PR changes and why. -->

## Changes

- 
- 

## Related Issue

Closes EM-123

## Type of Change

- [ ] feat - New functionality
- [ ] fix - Bug fix
- [ ] refactor - Code refactoring
- [ ] perf - Performance improvement
- [ ] test - Tests
- [ ] docs - Documentation
- [ ] build - Build and dependencies
- [ ] ci - CI/CD
- [ ] chore - Maintenance

## Testing

- [ ] Unit tests
- [ ] Integration tests
- [ ] Manual testing
- [ ] All existing tests pass

## Quality Checks

- [ ] Code follows project conventions
- [ ] No unnecessary dependencies added
- [ ] No secrets or credentials committed
- [ ] Documentation updated if necessary
- [ ] CI pipeline passes

## Screenshots

<!-- Add screenshots if UI changes are included. -->
```

---

## 7. GitHub Actions — автоматическая проверка PR title

Для автоматической валидации названия PR создайте файл `.github/workflows/pr-title.yml`:

```yaml
name: PR Title Validation

on:
  pull_request_target:
    types:
      - opened
      - edited
      - synchronize
      - reopened

permissions:
  pull-requests: read

jobs:
  validate:
    name: Validate PR title
    runs-on: ubuntu-latest

    steps:
      - name: Validate PR title
        env:
          PR_TITLE: ${{ github.event.pull_request.title }}
        run: |
          pattern='^(feat|fix|refactor|perf|test|docs|build|ci|chore|style|revert)(\([a-z0-9-]+\))?: [a-z0-9].{0,79}$'

          if [[ ! "$PR_TITLE" =~ $pattern ]]; then
            echo "::error::Invalid PR title: $PR_TITLE"
            echo "Expected format: <type>(<scope>): <description>"
            exit 1
          fi

          if [[ "$PR_TITLE" =~ [.]$ ]]; then
            echo "::error::PR title must not end with a period"
            exit 1
          fi

          echo "PR title is valid: $PR_TITLE"
```

> **Безопасность:** `pull_request_target` используется исключительно для проверки метаданных PR, а не для выполнения кода из недоверенной ветки. Не добавляйте в этот workflow `checkout` или выполнение скриптов из PR без отдельной проверки безопасности.

### Примеры валидации

| PR title | Результат |
| :--- | :--- |
| `feat(auth): implement login` | **PASS** |
| `fix(users): resolve validation error` | **PASS** |
| `feat: add authentication` | **PASS** |
| `Feature(auth): implement login` | **FAIL** |
| `feat(auth): Implement login` | **FAIL** |
| `feat(auth): implement login.` | **FAIL** |
| `new login functionality` | **FAIL** |

> *Примечание: данный workflow проверяет формат заголовка, но не сопоставляет `type` PR с `type` ветки и не проверяет существование задачи в трекере.*

---

## 8. GitHub Repository Settings

Обязательные настройки защиты веток (Branch Protection Rules) для каждого репозитория `emarket`:

1. Обязательная проверка `Validate PR title`.
2. Обязательная проверка `Build and Test`.
3. Обязательная проверка форматирования (например, Spotless для Java).
4. Запрет прямого `push` в `main`.
5. Запрет `force push`.
6. Обязательное прохождение CI перед merge.

> При работе единственного разработчика нет необходимости искусственно требовать второго reviewer — допустим self-review перед merge, однако PR остаётся обязательным способом интеграции изменений в `main`.

---

## 9. Merge strategy

Для проекта `emarket` используется стратегия **Squash and merge**.

### Преимущества

- Каждый PR превращается в один логический commit.
- История `main` остаётся чистой и линейной.
- Удобно анализировать изменения через `git log`.
- PR title автоматически становится названием итогового commit.
- В истории Git строго сохраняется формат Conventional Commits.

### Настройки GitHub

| Настройка | Решение |
| :--- | :--- |
| Allow merge commits | Disabled |
| Allow squash merging | Enabled |
| Allow rebase merging | Disabled |
| Default commit message | Pull request title |
| Delete head branches | Enabled |

### Пример истории коммитов в `main`

```text
main
 │
 ├── feat(auth): implement user registration
 ├── feat(auth): implement JWT authentication
 ├── fix(users): prevent duplicate email
 ├── test(users): add integration tests
 └── docs(api): document authentication endpoints
```

---

## 10. Итоговая сводка конвенции emarket

### Обязательный стандарт для всех репозиториев

| Правило | Стандарт |
| :--- | :--- |
| PR title | Conventional PR |
| Формат | `<type>(<scope>): <description>` |
| Язык | English |
| Регистр (Case) | `lowercase` |
| Description | Imperative mood |
| Scope | Optional, `lowercase` |
| Issue ID | В названии Branch |
| PR template | Обязателен |
| Title validation | GitHub Actions |
| Merge strategy | Squash and merge |
| Direct push to main | Запрещён |
| Code review | Self-review допустим |

### Примеры PR для разных репозиториев

| Repository | Pull Request |
| :--- | :--- |
| `emarket-web` | `feat(auth): implement login page` |
| `emarket-api-gateway` | `feat(gateway): configure authentication routes` |
| `emarket-bff-web` | `feat(users): add user profile endpoint` |
| `emarket-bff-mobile` | `feat(users): add mobile profile endpoint` |
| `emarket-identity-service` | `feat(auth): implement JWT token rotation` |
| `emarket-user-service` | `feat(users): implement user registration` |
| `emarket-docs` | `docs(architecture): add authentication diagram` |

---

## 11. Структура файлов в каждом репозитории

```text
repository/
├── .github/
│   ├── workflows/
│   │   ├── ci.yml
│   │   └── pr-title.yml
│   └── PULL_REQUEST_TEMPLATE.md
├── ...
└── README.md
```

### Архитектурная рекомендация

Поскольку каждый микросервис `emarket` имеет отдельный GitHub-репозиторий, необходимо поддерживать идентичные PR conventions, GitHub Actions и шаблоны во всех репозиториях. В дальнейшем их можно вынести в отдельный репозиторий с общими инженерными стандартами (Reusable Workflows и `.github` organization repository).

Для текущего масштаба проекта оптимально отказаться от сложного GitFlow в пользу простого и надёжного процесса: **`main` + `feature/fix` ветки + Pull Requests + Squash and merge**.
# Git Branching Convention & Workflow — emarket

Данный документ определяет единую политику именования веток (Git Branching Convention) и правила работы с Git workflow для всех GitHub-репозиториев экосистемы **emarket**:

- `emarket-web`
- `emarket-api-gateway`
- `emarket-bff-web`
- `emarket-bff-mobile`
- `emarket-identity-service`
- `emarket-user-service`
- `emarket-docs`

Проверка соблюдения правил выполняется автоматически через GitHub Actions при каждом `push` и создании `Pull Request`.

---

## 1. Git Branch Naming Convention

За основу взята упрощённая модель **GitFlow + Conventional Branch Names**: она обеспечивает высокую строгий подход для production, не усложняя ежедневную разработку.

### 1.1. Основные типы веток

| Тип | Назначение | Пример |
| :--- | :--- | :--- |
| `main` | Production | `main` |
| `develop` | Интеграционная ветка | `develop` |
| `feature/` | Новая функциональность | `feature/EM-123-add-product` |
| `bugfix/` | Исправление ошибок | `bugfix/EM-124-fix-login` |
| `hotfix/` | Срочное исправление production | `hotfix/EM-125-fix-payment` |
| `release/` | Подготовка релиза | `release/v1.2.0` |
| `refactor/` | Рефакторинг кода | `refactor/EM-126-simplify-auth` |
| `test/` | Добавление или изменение тестов | `test/EM-127-add-user-tests` |
| `docs/` | Обновление документации | `docs/EM-128-update-readme` |
| `chore/` | Технические задачи и обслуживание | `chore/EM-129-update-dependencies` |
| `ci/` | Изменения CI/CD пайплайнов | `ci/EM-130-update-github-actions` |
| `perf/` | Оптимизация производительности | `perf/EM-131-optimize-query` |

### 1.2. Правила именования

Обязательные требования к названию веток:

1. Все символы в нижнем регистре (**lowercase**), за исключением идентификатора задачи.
2. Разделитель слов — дефис (`-`, **kebab-case**).
3. Наличие идентификатора задачи после указания типа ветки.
4. Формат идентификатора задачи: `EM-123`.
5. Запрещено использовать пробелы, символы подчёркивания (`_`) и спецсимволы.
6. Название должно кратко и точно описывать задачу.
7. Максимальная длина имени ветки — **100 символов**.

#### Шаблон
```text
<type>/<ticket-id>-<description>
```

#### Примеры корректных имен
- `feature/EM-101-add-product-search`
- `feature/EM-102-add-user-profile`
- `bugfix/EM-103-fix-authentication`
- `refactor/EM-104-refactor-user-service`
- `test/EM-105-add-integration-tests`
- `ci/EM-106-configure-github-actions`

#### Формат веток релизов
- `release/v1.0.0`
- `release/v1.1.0`
- `release/v2.0.0`

### 1.3. Git Workflow

Схема жизненного цикла веток:

```text
[ Production ]       main
                       ▲
                       │  (Release / Hotfix)
                       │
[ Integration ]     develop ◀─── feature/*, bugfix/*, refactor/*, test/*
```

- **Рабочие ветки** (`feature/*`, `bugfix/*`, `refactor/*` и др.) создаются от `develop` и сливаются обратно в `develop` исключительно через Pull Request.
- **Релизные ветки** (`release/v1.0.0`) создаются от `develop` для финальной стабилизации перед мепджем в `main` и `develop`.
- **Ветки экстренных правок** (`hotfix/EM-125-fix-payment`) создаются от `main` и после проверки ответвляются/сливаются в `main` и `develop`.

---

## 2. GitHub Actions — Проверка названий веток

Автоматическая валидация реализуется отдельным workflow, размещаемым в каждом репозитории:

```text
.github/
└── workflows/
    ├── branch-name-validation.yml
    └── ci.yml
```

### 2.1. Конфигурация `branch-name-validation.yml`

```yaml
name: Branch Name Validation

on:
  push:
    branches:
      - '**'

  pull_request:
    types:
      - opened
      - synchronize
      - reopened
      - edited

permissions:
  contents: read

jobs:
  validate-branch-name:
    name: Validate Branch Name
    runs-on: ubuntu-latest

    steps:
      - name: Validate branch name
        shell: bash
        env:
          BRANCH_NAME: ${{ github.event_name == 'pull_request' && github.head_ref || github.ref_name }}
        run: |
          set -euo pipefail

          BRANCH="$BRANCH_NAME"

          echo "Validating branch: $BRANCH"

          # Maximum branch name length
          if [[ ${#BRANCH} -gt 100 ]]; then
            echo "::error::Branch name exceeds 100 characters."
            exit 1
          fi

          # Protected branch names
          if [[ "$BRANCH" == "main" || "$BRANCH" == "develop" ]]; then
            echo "Valid protected branch: $BRANCH"
            exit 0
          fi

          # Release branches
          if [[ "$BRANCH" =~ ^release/v[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
            echo "Valid release branch: $BRANCH"
            exit 0
          fi

          # Feature / bugfix / hotfix / refactor / test / docs / chore / ci / perf branches
          PATTERN='^(feature|bugfix|hotfix|refactor|test|docs|chore|ci|perf)/EM-[0-9]+-[a-z0-9]+(-[a-z0-9]+)*$'

          if [[ "$BRANCH" =~ $PATTERN ]]; then
            echo "Valid branch name: $BRANCH"
            exit 0
          fi

          echo "::error::Invalid branch name: $BRANCH"
          echo ""
          echo "Expected formats:"
          echo "  feature/EM-123-add-product-search"
          echo "  bugfix/EM-124-fix-login"
          echo "  hotfix/EM-125-fix-payment"
          echo "  refactor/EM-126-refactor-auth"
          echo "  test/EM-127-add-tests"
          echo "  docs/EM-128-update-readme"
          echo "  chore/EM-129-update-dependencies"
          echo "  ci/EM-130-update-workflow"
          echo "  perf/EM-131-optimize-query"
          echo "  release/v1.2.0"

          exit 1
```

### 2.2. Примеры результатов валидации

| Branch name | Result | Причина |
| :--- | :--- | :--- |
| `main` | **PASS** | Защищенная ветка |
| `develop` | **PASS** | Защищенная ветка |
| `feature/EM-123-add-product` | **PASS** | Соблюден формат |
| `bugfix/EM-124-fix-login` | **PASS** | Соблюден формат |
| `release/v1.2.0` | **PASS** | Корректная версия SemVer |
| `feature/add-product` | **FAIL** | Отсутствует тикет `EM-XXX` |
| `Feature/EM-123-add-product` | **FAIL** | Префикс с заглавной буквы |
| `feature/EM-123-Add-Product` | **FAIL** | Заглавные буквы в описании |
| `feature/EM-123_add-product` | **FAIL** | Использован `_` вместо `-` |
| `feature/EM-123` | **FAIL** | Отсутствует описание |
| `random-branch` | **FAIL** | Произвольное имя |

---

## 3. Защита GitHub-репозитория

Для блокировки невалидных веток одного лишь CI недостаточно. Обязательно настраиваются **Branch Protection Rules** или **Repository Rulesets**.

### 3.1. Защита ветки `main`

**Рекомендуемые параметры:**
- Require a pull request before merging
- Require approvals (минимум 1)
- Require status checks to pass before merging
- Require branches to be up to date before merging
- Block force pushes
- Block branch deletion
- Require conversation resolution

### 3.2. Защита ветки `develop`

**Рекомендуемые параметры:**
- Require a pull request before merging
- Require status checks to pass before merging
- Block force pushes
- Block branch deletion

### 3.3. Required status checks

В настройках репозитория (**Settings → Rules → Rulesets** или **Settings → Branches**) необходимо добавить в список обязательных проверок:

- `Validate Branch Name`
- `Build`
- `Unit Tests`
- `Integration Tests`
- `Code Quality`

> **Обратите внимание:** GitHub Actions сам по себе не запрещает исходный `git push`. Он делает проверку обязательной для получения статуса `SUCCESS` и выполнения `Merge`. Для полного запрета push невалидных веток на сервер следует настроить *Repository Rulesets* (если доступно в рамках вашего тарифа GitHub).

---

## 4. Дополнительные Production-рекомендации

Стандарты для обеспечения стабильности и единого порядка в проекте:

| Практика | Рекомендация |
| :--- | :--- |
| **Commit messages** | Conventional Commits (`feat: ...`, `fix: ...`) |
| **Branch names** | Предложенная convention |
| **PR title** | Conventional Commits |
| **Merge strategy** | Squash and merge |
| **Default branch** | `develop` |
| **Production branch** | `main` |
| **CI/CD** | GitHub Actions |
| **Release versioning** | Semantic Versioning (`vX.Y.Z`) |

### Примеры оформлений PR Title

- `feat(user): add profile management`
- `fix(auth): resolve token expiration issue`
- `refactor(product): simplify product filtering`
- `test(user): add integration tests`
- `docs(api): update swagger documentation`

---

## 5. Итоговая структура документации и CI

Рекомендуется зафиксировать эти стандарты в виде единой базовой структуры во всех сервисах:

```text
emarket/
├── .github/
│   ├── workflows/
│   │   ├── branch-name-validation.yml
│   │   └── ci.yml
│   └── PULL_REQUEST_TEMPLATE.md
│
└── docs/
    └── development/
        ├── git-workflow.md
        ├── branch-naming.md
        └── commit-convention.md
```

### Дальнейшее развитие (Централизация workflow)
Поскольку сервисы распределены по отдельным репозиториям, со временем workflow `branch-name-validation.yml` стоит вынести в общий репозиторий `emarket-github-workflows` и подключать его через **Reusable Workflows** (`workflow_call`). Это позволит обновлять правила именования синхронно для всех сервисов в одном месте.

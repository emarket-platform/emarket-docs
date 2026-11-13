# Production-ready Git branching strategy для emarket-platform

## 1. Рекомендованная стратегия

Для `emarket-platform`, где каждый микросервис находится в отдельном
GitHub-репозитории, рекомендуется **Trunk-Based Development с
короткоживущими feature branches**.

В каждом репозитории:

-   `main` --- единственная постоянная ветка, содержащая проверенный
    код, готовый к релизу.
-   `feature/*` --- разработка новой функциональности.
-   `bugfix/*` --- исправление обычных ошибок.
-   `hotfix/*` --- срочное исправление production.
-   `chore/*` --- технические задачи и обновления зависимостей.
-   `refactor/*` --- рефакторинг.
-   `docs/*` --- документация.
-   `test/*` --- добавление или изменение тестов.

Не нужно создавать постоянные `develop`, `release` и `staging` ветки в
каждом микросервисе.

Окружения `development`, `staging` и `production` --- это deployment
environments, а не обязательные Git-ветки.

## 2. Почему не GitFlow

Классический GitFlow использует постоянные `main` и `develop`, а также
временные `feature/*`, `release/*` и `hotfix/*`.

GitFlow полезен, когда у продукта длинные циклы релизов, несколько
одновременно поддерживаемых версий и отдельные release-стадии.

Для независимых микросервисов GitFlow часто добавляет лишнюю сложность:

1.  Каждый сервис имеет собственный жизненный цикл и релизы.
2.  `develop` создаёт дополнительную точку интеграции.
3.  `release` и `staging` не нужны, если релизы создаются из
    проверенного `main`.
4.  Исправления приходится дополнительно переносить между ветками.
5.  Увеличивается объём Git-операций и обслуживания веток.

Отсутствие `develop` не означает отсутствие development environment.

## 3. Workflow разработки

### Шаг 1. Создать GitHub Issue

Например:

`EM-123 — Implement user registration`

### Шаг 2. Создать feature-ветку от актуальной main

``` bash
git switch main
git pull origin main

git switch -c feature/EM-123-user-registration
```

### Шаг 3. Разрабатывать и коммитить

Примеры Conventional Commits:

``` text
feat: implement user registration
test: add registration integration tests
refactor: extract user validation
```

### Шаг 4. Отправить ветку и создать Pull Request

``` bash
git push -u origin feature/EM-123-user-registration
```

Pull Request направляется из `feature/EM-123-user-registration` в
`main`.

### Шаг 5. Выполнить CI-проверки

Для Java/Gradle:

``` bash
./gradlew clean build
```

Для React/TypeScript/Vite (пример набора проверок):

``` bash
npm ci
npm run lint
npm run typecheck
npm run test
npm run build
```

В обязательные проверки следует включать применимые к репозиторию:

-   build;
-   unit tests;
-   integration tests;
-   Spotless check (Java);
-   lint и typecheck (frontend);
-   security checks;
-   дополнительные quality checks, если они настроены.

Имена required status checks в Ruleset должны точно совпадать с именами
реальных CI jobs.

### Шаг 6. Выполнить merge

После успешных проверок и необходимых approvals выполнить **Squash and
merge**.

### Шаг 7. Удалить временную ветку

После merge feature/bugfix/hotfix-ветка удаляется.

### Workflow в целом

``` text
main
  |
  +--> feature/EM-123-user-registration
           |
           +--> Development
           |
           +--> Push + Pull Request
                    |
                    +--> GitHub Actions CI
                    |
                    +--> Code Review
                    |
                    +--> Squash and merge
                              |
                              +--> main
                                      |
                                      +--> Deployment
```

## 4. Bugfix и hotfix

### Обычный bugfix

``` bash
git switch main
git pull origin main

git switch -c bugfix/EM-124-fix-login
```

После исправления создаётся Pull Request в `main`. Далее выполняются CI,
review (если требуется), merge и удаление ветки.

### Срочный hotfix

``` bash
git switch main
git pull origin main

git switch -c hotfix/EM-125-fix-token-validation
```

Далее:

1.  Исправить production-проблему.
2.  Добавить тест, воспроизводящий ошибку.
3.  Создать Pull Request в `main`.
4.  Выполнить обязательные CI-проверки.
5.  Провести review согласно принятой политике.
6.  Выполнить merge.
7.  Запустить deployment.

В этой стратегии hotfix вливается только в `main`: переносить его в
`develop` не требуется.

## 5. Релизы и версии

Для каждого микросервиса использовать Semantic Versioning:

`MAJOR.MINOR.PATCH`

Примеры:

``` text
v1.0.0
v1.1.0
v1.1.1
v1.2.0
v2.0.0
```

Релиз создаётся Git tag на соответствующем commit в `main`:

``` bash
git switch main
git pull origin main

git tag -a v1.2.0 -m "Release v1.2.0"
git push origin v1.2.0
```

Каждый микросервис имеет собственную версию и собственные релизы.

Например:

| Сервис | Версия |
| :--- | :--- |
| `emarket-web` | v1.3.0 |
| `emarket-api-gateway` | v1.1.0 |
| `emarket-user-service` | v1.5.2 |
| `emarket-identity-service` | v1.2.1 |

Это позволяет выпускать и развёртывать сервисы независимо друг от друга.

## 6. Как должны выглядеть настройки всех твоих репозиториев

Рекомендую использовать одинаковую политику:

| Repository | Default branch | Protection |
| :--- | :--- | :--- |
| `emarket-web` | `main` | `protect-main` |
| `emarket-api-gateway` | `main` | `protect-main` |
| `emarket-bff-web` | `main` | `protect-main` |
| `emarket-bff-mobile` | `main` | `protect-main` |
| `emarket-identity-service` | `main` | `protect-main` |
| `emarket-user-service` | `main` | `protect-main` |

Один и тот же принцип разработки, одинаковые требования к Pull Request и CI.

При этом каждый репозиторий самостоятельно отвечает за:
- собственные feature branches;
- собственные CI pipelines;
- собственные версии;
- собственные релизы;
- собственный deployment.

## 7. Моя окончательная рекомендация

Для твоего проекта я бы зафиксировал следующую архитектуру:

| Элемент | Решение |
| :--- | :--- |
| **Branching strategy** | Trunk-Based Development |
| **Permanent branches** | Только `main` |
| **Feature development** | `feature/*` |
| **Bug fixes** | `bugfix/*` |
| **Production fixes** | `hotfix/*` |
| **Pull Requests** | Обязательны |
| **Merge strategy** | Squash and merge |
| **CI** | GitHub Actions |
| **Main protection** | Repository Rulesets |
| **Code review** | 1 approval при наличии второго разработчика |
| **Release versioning** | Semantic Versioning |
| **Release mechanism** | Git Tags |
| **Deployment** | Независимый для каждого сервиса |

### Почему именно это решение?

Ты строишь систему независимых микросервисов, а не один продукт с единым циклом релизов. Каждый сервис должен иметь возможность независимо разрабатываться, тестироваться, выпускаться и развёртываться.

GitFlow здесь добавит организационную сложность, но не даст сопоставимой практической пользы.

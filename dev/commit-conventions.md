###  Conventional Commits

Для production-ready проекта я рекомендую использовать Conventional Commits.

```text
<type>(<scope>): <description>
```

scope не обязателен.
В Conventional Commits допустимы оба варианта:
```text
feat: add user profile
```

и

```text
feat(user): add user profile
```
### Основные type
| Type       | Когда использовать                        | Пример                                    |
|------------|--------------------------------------------|-------------------------------------------|
| `feat`     | Новая функциональность                     | `feat(user): add profile endpoint`        |
| `fix`      | Исправление бага                           | `fix(auth): fix expired token handling`   |
| `refactor` | Изменение кода без изменения поведения    | `refactor(user): extract profile mapper`  |
| `test`     | Добавление/изменение тестов                | `test(user): add profile service tests`   |
| `docs`     | Документация                               | `docs(api): document user endpoints`      |
| `chore`    | Технические изменения                      | `chore(build): update Gradle wrapper`     |
| `build`    | Изменения build/dependencies               | `build(deps): upgrade Spring Boot`        |
| `ci`       | CI/CD                                      | `ci(github): add build workflow`          |
| `perf`     | Улучшение производительности               | `perf(search): add product cache`         |
| `style`    | Форматирование, без изменения логики      | `style(user): format service classes`     |
| `revert`   | Откат commit                               | `revert: revert user profile changes`     |

Для emarket я бы рекомендовал использовать scope, потому что много отдельных сервисов:
```text
feat(identity): add login endpoint
feat(user): add profile endpoint
feat(bff-web): add product catalog endpoint
feat(gateway): configure web bff routes
feat(web): add login page
test(identity): add authentication tests
docs(api): document public endpoints
ci(user): add GitHub Actions workflow
```

Это делает историю Git намного понятнее.

Но если изменение не относится к конкретному компоненту, можно без scope:
```text
chore: update project documentation
docs: update architecture diagram
ci: update GitHub Actions
```

Например, создание emarket-user-service:
```text
feat(user): initialize user service
```
Добавление endpoint:
```text
feat(user): add get current user endpoint
```
Добавление тестов:
```text
test(user): add current user service tests
```

GitHub Actions:
```text
ci(user): add build and test workflow
```
Документация
```text
docs(user): document REST API
```
Главное правило: commit должен описывать одно логическое изменение, а не, например, update project или changes.

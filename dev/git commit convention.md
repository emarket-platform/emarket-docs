###  Conventional Commits + GitHub Branch Protection

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
### Conventional Commits - основные типы
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

## Настройка Commit conventions
Для emarket я рекомендую настроить единый стандарт Conventional Commits для всех 7 GitHub-репозиториев.
Поскольку каждый сервис находится в отдельном репозитории, важно сделать проверку централизованной по правилам, но автоматической в каждом репозитории.

Моя рекомендация — использовать три уровня защиты:
1. Commitlint — проверяет соответствие commit message стандарту Conventional Commits.
2. Husky — проверяет сообщение локально перед созданием commit.
3. GitHub Actions — повторно проверяет commits на GitHub. Это серверная проверка, которую нельзя обойти простым отключением локального hook.

Для production-ready процесса третий уровень особенно важен.

Рекомендую использовать одинаковые правила во всех репозиториях.

Примеры:
```text
feat(web): add product catalog page
feat(gateway): configure web bff routing
feat(bff-web): add product catalog endpoint
feat(bff-mobile): add mobile catalog endpoint
feat(identity): implement JWT authentication
feat(user): add profile endpoint
docs: update system architecture
```
В emarket-docs scope можно не использовать, поскольку репозиторий целиком посвящён документации.

### Архитектура проверки
    
            Developer
        Создаёт commit
             |
             V
        Husky + Commitlint
        Valid commit
    Локальная проверка commit message
             |
             V
    GitHub Push / Pull Request
             |    
             V
    GitHub Actions + Commitlint
        Check passed
    Серверная проверка всех commits в PR
             |   
             V
      Branch Protection / Rulesets
    Merge разрешён только при успешных проверках


### Связка Husky + Commitlint + Node.js
Это индустриальный стандарт (industry standard) как для Frontend (emarket-web), так и для Backend-разработки.
Она универсальна, поддерживается на любых ОС и избавляет от необходимости писать и поддерживать кастомные скрипты проверки 
валидации вручную.

#### Зачем нужен Node.js в данном контексте?
Node.js нужен исключительно как инструментальная среда (runtime) для запуска утилит автоматизации разработки (Husky и Commitlint).

### Настройка Commitlint
Будем использовать:
- Node.js 22 LTS
- npm
- @commitlint/cli
- @commitlint/config-conventional
- Husky

Commitlint официально поддерживает Conventional Commits и интеграцию с GitHub Actions.

#### Шаг 1. Установить зависимости
В каждом репозитории:
```bash
npm install --save-dev \
  @commitlint/cli \
  @commitlint/config-conventional \
  husky
```
В Java-репозиториях package.json и package-lock.json нужны для Git tooling (Commitlint + Husky), но не являются частью Java-приложения.

- package.json  - Декларация dev-зависимостей Node.js (husky, commitlint)
- package-lock.json - Lock-файл версий npm-пакетов

В .gitignore нужно добавить только:
```test
# Node.js
node_modules/
npm-debug.log*
```

Для Java-репозиториев можно создать небольшой package.json исключительно для Git tooling:
```bassh
npm init -y
```
Он не будет влиять на Java-приложение или Gradle build.
#### Шаг 2. Создать конфигурацию Commitlint
В корне каждого репозитория создать файл:

commitlint.config.mjs - Правила проверки коммитов (extends @commitlint/config-conventional)

Содержимое:
```javascript
export default {
  extends: ['@commitlint/config-conventional'],
};
```

### Шаг 3. Настроить Husky
Выполнить:
```Bash
npx husky init
```
- .husky/ - Конфигурация Git-хуков (Husky)

Убедиться, что в package.json присутствует:
```json
{
  "scripts": {
    "prepare": "husky"
  }
}
```

Создать файл:
.husky/commit-msg

Содержимое:
```bash
npx --no -- commitlint --edit "$1"
```
Теперь при создании commit Husky будет запускать Commitlint.
#### Шаг 4. Проверить локальную работу
Для java репозиториев файл .husky/pre-commit нужно удалить.

Попробуй выполнить:
```bash
git commit -m "add user profile"
```
Commit не должен создаться, поскольку отсутствует корректный Conventional Commit type.

#### 5. Настройка GitHub Actions
Это наиболее важная часть: проверка на стороне GitHub.
В каждом репозитории создать директорию:
```text
.github/
└── workflows/
    └── commitlint.yml
```
Файл .github/workflows/commitlint.yml:
```yml
name: Commitlint

on:
  pull_request:
    branches:
      - main
    types:
      - opened
      - synchronize
      - reopened
      - edited

permissions:
  contents: read

jobs:
  commitlint:
    name: commitlint
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Install dependencies
        run: npm ci

      - name: Validate commit messages
        run: |
          npx commitlint \
            --from "${{ github.event.pull_request.base.sha }}" \
            --to "${{ github.event.pull_request.head.sha }}" \
            --verbose
```

Что происходит:
1. Developer создаёт Pull Request в main.
2. GitHub Actions получает все commits из PR.
3. Commitlint проверяет каждый commit.
4. Если хотя бы один commit неправильный — workflow завершается с ошибкой.
5. Pull Request не сможет пройти обязательную проверку.

Именно такой способ проверки диапазона commits рекомендует документация Commitlint.

Примечание: здесь предполагается, что в каждом репозитории есть package-lock.json. После установки зависимостей его нужно закоммитить.

#### Настройка GitHub Branch Protection
Без этого шага разработчик потенциально сможет проигнорировать результат GitHub Actions.

Для каждого репозитория:
1. Открыть GitHub repository.
2. Перейти в Settings.
3. Выбрать Rules → Rulesets.
4. Нажать New ruleset → New branch ruleset.
5. Название:
   main-branch-protection
6. Target branches:
   - Выберите шаблон для ветки: Target by inclusion
   - В появившемся меню выберите Include by pattern (Включить по шаблону).
   - Укажите имя ветки main
7. Включить следующие настройки:
   - Require a pull request before merging
   - Require status checks to pass	
   - Require branches to be up to date
   - Block force pushes	
   - Restrict deletions	
8. В Required status checks добавить:
```text
commitlint
```
9. Сохранить Ruleset.

После этого GitHub не позволит выполнить merge PR, пока проверка Commitlint не пройдёт успешно.

#### Нужно ли повторять настройку для каждого репозитория?
Да, при первоначальной настройке — но не обязательно выполнять всё вручную.
Есть три варианта.
1. Копирование конфигурации
   Одинаковые файлы в каждом репозитории
2. Shared GitHub Workflow
   Один общий workflow, вызываемый всеми репозиториями
3. Organization Rulesets
   Общие правила защиты для нескольких репозиториев организации

#### Что необходимо добавить в каждый репозиторий
Файл
- package.json
- package-lock.json
- commitlint.config.mjs
- .husky/commit-msg
- .github/workflows/commitlint.yml
- Branch Protection / Ruleset

#### Production-ready дополнительные рекомендации
Я бы добавил ещё несколько правил.
1. Запрет прямого push в main
   Все изменения только через Pull Request.
2. Использовать Conventional Commits для PR commits
   Например:
```text
feat(user): add profile endpoint
fix(identity): fix refresh token validation
test(user): add profile integration tests
```
3. Не использовать scope как обязательное поле
```text
feat(user): add profile endpoint
docs: update architecture documentation
```
#### Итоговая структура каждого репозитория
Например, emarket-user-service:
   
      emarket-user-service/
      │
      ├── .github/
      │   └── workflows/
      │       └── commitlint.yml
      │
      ├── .husky/
      │   └── commit-msg
      │
      ├── src/
      │   ├── main/
      │   └── test/
      │
      ├── gradle/
      │
      ├── build.gradle.kts
      ├── settings.gradle.kts
      │
      ├── package.json
      ├── package-lock.json
      ├── commitlint.config.mjs
      │
      ├── Dockerfile
      └── README.md

### Можно ли этот процесс автоматизировать?
Да. Для emarket я рекомендую автоматизировать этот процесс с помощью одного Bash-скрипта, который настроит все 7 локальных Git-репозиториев.
Тогда вместо повторения одних и тех же действий вручную достаточно будет выполнить одну команду.

#### 1. Предлагаемое решение
Создать отдельный скрипт  отдельном репозитории emarket-setup:
```text
emarket-setup/
└── scripts/
    └── setup-conventional-commits.sh
```

Он будет автоматически:
1. Проверять наличие Node.js и npm.
2. Проверять наличие Git-репозитория.
3. Создавать package.json, если его нет.
4. Устанавливать Commitlint и Husky.
5. Создавать commitlint.config.mjs.
6. Создавать .husky/commit-msg.
7. Создавать .github/workflows/commitlint.yml.
8. Добавлять node_modules/ в .gitignore.
9. Сохранять существующие настройки package.json и .gitignore.

Важно: скрипт не должен удалять существующие npm scripts, Git hooks или другие настройки проектов.

2. Как будет выглядеть использование
   Предположим, у тебя все репозитории находятся в одной директории:
```text
~/projects/emarket/
│
├── emarket-web/
├── emarket-api-gateway/
├── emarket-bff-web/
├── emarket-bff-mobile/
├── emarket-identity-service/
├── emarket-user-service/
├── emarket-docs/
│
└── emarket-setup/
    └── scripts/
        └── setup-conventional-commits.sh
```
Запуск
```bash
chmod +x scripts/setup-conventional-commits.sh

./scripts/setup-conventional-commits.sh
```
Скрипт последовательно обработает все семь репозиториев.
3. Что останется сделать вручную?
   Есть две операции, которые нужно отделить от локальной настройки.

- GitHub Branch Protection / Rulesets

Это настройки GitHub. Их можно:
- Настроить вручную.
- Автоматизировать через GitHub CLI (gh) и GitHub API.
- Настроить одним Organization Ruleset для нескольких репозиториев, если они находятся в GitHub Organization и доступна соответствующая функциональность.


- Git commit и push
  После выполнения скрипта необходимо добавить новые файлы в каждый репозиторий:
```bash
git add .
git commit -m "chore: configure conventional commits"
git push
- ```
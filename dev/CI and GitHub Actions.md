# Настройка CI на GitHub Actions для `emarket-api-gateway`

Для `emarket-api-gateway` рекомендуется настроить **production-ready CI pipeline**, который автоматически запускается при каждом Pull Request и Push в основные ветки.

### Используемый стек:
- **Java:** 21
- **Spring Boot:** 4.1.2
- **Build Tool:** Gradle Kotlin DSL + Gradle Wrapper
- **CI/CD Platform:** GitHub Actions
- **Formatting:** Spotless — проверка форматирования Java-кода
- **Testing:** JUnit 5 — unit и integration тесты
- **Coverage:** JaCoCo — измерение покрытия кода тестами
- **Dependency Security:** Gradle dependency verification (при необходимости)

> **Основная задача CI:** не позволить объединить в `main` код, который не компилируется, не проходит тесты или не соответствует стандартам форматирования.

---

## 1. Архитектура CI Pipeline

```text
[Developer]
    │ (git push / Pull Request)
    ▼
[GitHub Actions] ────────► .github/workflows/ci.yml
                            │
                            ├── 1. Checkout (Get source code)
                            ├── 2. Setup Java (JDK 21)
                            ├── 3. Spotless (Check formatting)
                            ├── 4. Tests (JUnit 5)
                            ├── 5. JaCoCo (Test coverage)
                            └── 6. Build (Boot JAR)
                            │
                            ▼
                     [CI Passed] ──► Pull Request can be merged
```

---

## 2. Создание GitHub Actions workflow

В корне репозитория `emarket-api-gateway` создайте следующую структуру файлов:

```text
emarket-api-gateway/
├── .github/
│   └── workflows/
│       └── ci.yml
├── gradle/
├── src/
├── build.gradle.kts
├── settings.gradle.kts
├── gradlew
└── gradlew.bat
```

### Файл `.github/workflows/ci.yml`:

```yaml
name: Java CI

on:
  push:
    branches:
      - main
      - develop

  pull_request:
    branches:
      - main
      - develop

  workflow_dispatch:

permissions:
  contents: read

concurrency:
  group: java-ci-${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  build:
    name: Build and Test

    runs-on: ubuntu-latest
    timeout-minutes: 15

    steps:
      - name: Checkout repository
        uses: actions/checkout@v7

      - name: Set up JDK 21
        uses: actions/setup-java@v6
        with:
          distribution: temurin
          java-version: '21'

      - name: Set up Gradle
        uses: gradle/actions/setup-gradle@v6
        with:
          cache-provider: basic

      - name: Build and run checks
        run: ./gradlew --no-daemon clean build
```

*Примечание: Здесь используются официальные GitHub Actions и Gradle Actions. Gradle Setup автоматически кэширует зависимости и проверяет Gradle Wrapper.*

---

## 3. Что именно выполняет `./gradlew clean build`

`./gradlew build` последовательно запускает жизненный цикл Gradle tasks:

| Task | Назначение |
| :--- | :--- |
| `clean` | Удаление предыдущих build artifacts |
| `compileJava` | Компиляция Java-кода |
| `processResources` | Обработка ресурсов |
| `classes` | Подготовка классов |
| `test` | Запуск unit-тестов |
| `check` | Проверки качества и тестов |
| `assemble` | Сборка артефактов |
| `build` | Полная сборка |

Если Spotless подключён к Gradle `check`, то `spotlessCheck` будет выполнен автоматически.

### Важное отличие:
- **`spotlessApply`** — автоматически исправляет форматирование локально.
- **`spotlessCheck`** — проверяет форматирование и завершает сборку с ошибкой при нарушениях.

```bash
# Локально перед коммитом:
./gradlew spotlessApply

# В GitHub Actions:
./gradlew clean build
```

---

## 4. Проверка и отчётность JaCoCo

Если JaCoCo подключён в `build.gradle.kts`, вы можете дополнить шаг сборки:

```yaml
- name: Build, test and generate coverage
  run: ./gradlew clean build jacocoTestReport --no-daemon
```

### Пример конфигурации `build.gradle.kts` (Kotlin DSL):

```kotlin
plugins {
    jacoco
}

tasks.test {
    useJUnitPlatform()
    finalizedBy(tasks.jacocoTestReport)
}

tasks.jacocoTestReport {
    dependsOn(tasks.test)

    reports {
        xml.required.set(true)
        html.required.set(true)
    }
}
```

HTML-отчёт формируется в директории:
`build/reports/jacoco/test/html/index.html`

> **Совет:** На следующем этапе можно добавить *JaCoCo coverage verification* (например, минимальное покрытие 80%), чтобы CI блокировал сборку при несоблюдении порога.

---

## 5. Запуск CI

Выполните команды в консоли для отправки workflow в репозиторий:

```bash
git add .github/workflows/ci.yml
git commit -m "ci: add GitHub Actions workflow"
git push origin develop
```

### Проверка выполнения:
1. Откройте GitHub и перейдите в репозиторий `emarket-api-gateway`.
2. Перейдите во вкладку **Actions**.
3. Найдите workflow **CI**.
4. Откройте выполнение **Build and Test**.

- **Зелёная галочка:** Сборка и тесты успешны.
- **Статус Failed:** Ошибка в тестах или нарушения форматирования.

---

## 6. Настройка Branch Protection

Настройка правил защиты веток — обязательная часть production-ready процесса.

**Маршрут:** `GitHub` → `Repository` → `Settings` → `Rules` → `Rulesets`

Создайте ruleset для ветки `main`:

| Настройка | Значение |
| :--- | :--- |
| **Require a pull request** | `Enabled` |
| **Require status checks to pass** | `Enabled` |
| **Required status check** | `Build and Test` |
| **Require branches to be up to date** | `Enabled` |
| **Block force pushes** | `Enabled` |
| **Restrict deletions** | `Enabled` |

---

## 7. Дополнительные Production-Ready рекомендации

1. **Проверять Gradle Wrapper:** Убедитесь, что скрипт исполняемый:
   ```bash
   chmod +x gradlew
   git add gradlew
   git commit -m "build: make Gradle wrapper executable"
   ```
2. **Проверять CI локально перед Push:**
   ```bash
   ./gradlew clean spotlessCheck test build
   ```
3. **Безопасность (Credentials):** Не храните секреты в репозитории. Используйте:
   `GitHub` → `Settings` → `Secrets and variables` → `Actions`.
4. **Разделение CI и CD:**

| Pipeline | Ответственность |
| :--- | :--- |
| **CI** | Compile, formatting, tests, coverage, build |
| **CD** | Docker image, registry, deployment |

---

## 8. Целевая структура CI/CD для микросервисов emarket

Для каждого отдельного микросервиса рекомендуется придерживаться единого подхода к файлам workflow:

```text
emarket-api-gateway/
└── .github/
    └── workflows/
        ├── ci.yml
        ├── docker-build.yml
        └── cd.yml

emarket-user-service/
└── .github/
    └── workflows/
        ├── ci.yml
        ├── docker-build.yml
        └── cd.yml

emarket-identity-service/
└── .github/
    └── workflows/
        ├── ci.yml
        ├── docker-build.yml
        └── cd.yml
```

> **Рекомендация:** Начните с базового `ci.yml`, добейтесь его стабильного выполнения и настройте Branch Protection. После этого переходите к автоматической сборке и публикации Docker Image.

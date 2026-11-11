# Форматирование Java-проекта

**Стек:** Spring Boot, Java 21, Gradle Kotlin DSL, GitHub.

## 1. Выбор инструментов

Рекомендуемый набор:

-   **Spotless** --- автоматическое форматирование Java-кода.
-   **Google Java Format** --- единый стиль форматирования Java.
-   **Checkstyle** --- контроль соглашений и качества оформления кода.
-   **Gradle** --- интеграция проверок в build lifecycle.
-   **Git Hooks** --- локальная проверка перед commit.
-   **GitHub Actions** --- обязательная проверка в CI.

Главный принцип production-ready решения:

1.  Форматирование должно выполняться автоматически.
2.  Разработчик должен видеть ошибки до создания commit.
3.  CI должен запрещать merge, если форматирование не соответствует
    стандарту.
4.  Не должно быть зависимости от настроек конкретной IDE.
5.  Все разработчики должны получать одинаковый результат независимо от
    IDE и ОС.

### Почему Spotless + Google Java Format?

Google Java Format обеспечивает единый стиль, а Spotless интегрирует его
в Gradle и предоставляет задачи `spotlessApply` и `spotlessCheck`.

Не рекомендуется использовать одновременно несколько Java-форматтеров:
это может приводить к конфликтам и нестабильному результату.

## 2. Подключение Spotless к Gradle

В корневом `build.gradle.kts`:

``` kotlin
plugins {
    id("java")
    id("org.springframework.boot") version "3.5.7"
    id("io.spring.dependency-management") version "1.1.7"

    id("com.diffplug.spotless") version "8.11.0"
}

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
}

spotless {
    java {
        target("src/**/*.java")

        // Expand wildcard imports into explicit imports.
        expandWildcardImports()
        //forbidWildcardImports()

        // Remove unused imports.
        removeUnusedImports()

        // Format Java code.
        googleJavaFormat("2.0.0").reorderImports(true)

        // Format annotations.
        formatAnnotations()
    }
}
```

Версии Spring Boot и Dependency Management приведены как пример. В
существующем проекте следует сохранить согласованные версии
зависимостей.

Важно явно закрепить версию Google Java Format, чтобы обновление
форматтера было контролируемым изменением.

### Что обеспечивает Spotless

-   единый стиль отступов;
-   автоматическое форматирование методов и классов;
-   единый порядок импортов;
-   удаление неиспользуемых импортов;
-   удаление пробелов в конце строк;
-   наличие пустой строки в конце файла.

## 3. Настройка Gradle Tasks

Spotless предоставляет две основные задачи:

  Команда           Назначение
  ----------------- ----------------------------------------
  `spotlessApply`   Исправить форматирование
  `spotlessCheck`   Проверить форматирование без изменений

Запуск:

``` bash
# Автоматическое форматирование
./gradlew spotlessApply

# Проверка форматирования
./gradlew spotlessCheck

# Сборка и проверки
./gradlew clean build
```


## 5. GitHub Actions --- обязательная проверка в CI

Создай файл `.github/workflows/java-quality.yml`:

``` yaml
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

permissions:
  contents: read

concurrency:
  group: java-ci-${{ github.ref }}
  cancel-in-progress: true

jobs:
  formatting:
    name: Java Formatting Check

    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '21'

      - name: Set up Gradle
        uses: gradle/actions/setup-gradle@v4

      - name: Check Java formatting
        run: ./gradlew --no-daemon spotlessCheck

      - name: Run Gradle checks
        run: ./gradlew --no-daemon check
```

Здесь:

-   `check` --- форматирование, Checkstyle и другие проверки,
    подключённые к Gradle lifecycle;
-   `bootJar` --- создание Spring Boot JAR.

Для production CI рекомендуется закреплять GitHub Actions по полному
commit SHA, а не только по тегу `v4`. При обновлении зависимостей
обновляй SHA контролируемо.

## 6. Git Hooks --- проверка перед commit

CI обнаружит проблему, но лучше обнаруживать её раньше --- на компьютере
разработчика.

Создай `.husky/pre-commit`:

``` bash
#!/usr/bin/env sh

set -eu

echo "Running Spotless formatting..."

./gradlew spotlessApply

echo "Staging formatted Java files..."

git add -- '*.java'

echo "Spotless formatting completed successfully."
```

Сделай файл исполняемым:

``` bash
chmod +x .husky/pre-commit
```

Теперь при попытке создать commit Git сначала сделает форматирование.

## 8. Настройка `.gitignore`

Убедись, что временные файлы и результаты сборки не попадают в Git:

``` gitignore
# Compiled class file
*.class

# Log file
*.log

# BlueJ files
*.ctxt

# Mobile Tools for Java (J2ME)
.mtj.tmp/

# Package Files #
*.jar
*.war
*.nar
*.ear
*.zip
*.tar.gz
*.rar

# virtual machine crash logs, see http://www.java.com/en/download/help/error_hotspot.xml
hs_err_pid*
replay_pid*

# Ignore Gradle project-specific cache directory
.gradle

# Ignore Gradle build output directory
build

# Ignore IntelliJ IDEA project files and folders
.idea/
*.iml
*.iws
*.ipr

```

При этом обязательно коммить:

``` text
gradlew
gradlew.bat
gradle/wrapper/
```

Gradle Wrapper гарантирует использование согласованной версии Gradle всеми разработчиками и CI.

## 9. Защита main branch в GitHub

Открой:

`GitHub → Repository → Settings → Rules → Rulesets`

Настрой правила для `main`:

| Правило | Значение |
| :--- | :--- |
| Require a pull request | Enabled |
| Require status checks | Enabled |
| Required check | `Code Quality` |
| Require branches to be up to date | Enabled |
| Block force pushes | Enabled |
| Restrict deletions | Enabled |

Результат: нельзя отправить в `main` код, который не прошёл обязательную
проверку.

Дополнительно рекомендуется запретить обход обязательных проверок для
обычных участников команды.

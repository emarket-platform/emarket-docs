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

        googleJavaFormat("1.36.1")

        removeUnusedImports()
        trimTrailingWhitespace()
        endWithNewline()
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

Подключи проверку к стандартному Gradle lifecycle:

``` kotlin
tasks.named("check") {
    dependsOn("spotlessCheck")
}
```

Теперь `./gradlew check` будет проверять форматирование. Команда
`./gradlew build` также выполнит `check` и не позволит собрать проект с
нарушением правил форматирования.

## 4. Настройка Checkstyle

Spotless отвечает за внешний вид кода. Checkstyle --- за соблюдение
соглашений.

Например:

-   длина методов;
-   длина строк;
-   наличие Javadoc;
-   соглашения об именовании;
-   запрещённые конструкции;
-   количество параметров метода.

Создай структуру:

``` text
project/
├── config/
│   └── checkstyle/
│       └── checkstyle.xml
├── src/
├── build.gradle.kts
└── settings.gradle.kts
```

Добавь в `build.gradle.kts`:

``` kotlin
plugins {
    checkstyle
}

checkstyle {
    toolVersion = "10.24.0"
    configFile = file("config/checkstyle/checkstyle.xml")
    isIgnoreFailures = false
    maxWarnings = 0
}
```

Пример `config/checkstyle/checkstyle.xml`:

``` xml
<?xml version="1.0"?>
<!DOCTYPE module PUBLIC
    "-//Checkstyle//DTD Checkstyle Configuration 1.3//EN"
    "https://checkstyle.org/dtds/configuration_1_3.dtd">

<module name="Checker">

    <property name="charset" value="UTF-8"/>

    <module name="LineLength">
        <property name="max" value="120"/>
    </module>

    <module name="TreeWalker">

        <module name="AvoidStarImport"/>

        <module name="UnusedImports"/>

        <module name="OneStatementPerLine"/>

        <module name="NeedBraces"/>

        <module name="VisibilityModifier"/>

        <module name="MethodLength">
            <property name="max" value="50"/>
        </module>

        <module name="ParameterNumber">
            <property name="max" value="7"/>
        </module>

    </module>

</module>
```

Запуск:

``` bash
./gradlew checkstyleMain
./gradlew checkstyleTest
```

Gradle автоматически включает задачи Checkstyle в `check`.

**Рекомендация:** начни со Spotless, затем постепенно добавляй
Checkstyle. Не стоит сразу вводить десятки строгих правил, которые будут
мешать разработке.

## 5. GitHub Actions --- обязательная проверка в CI

Создай файл `.github/workflows/java-quality.yml`:

``` yaml
name: Java Quality

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

jobs:
  quality:
    name: Code Quality
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '21'

      - name: Set up Gradle
        uses: gradle/actions/setup-gradle@v4

      - name: Verify
        run: ./gradlew clean check bootJar
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

Создай `.git/hooks/pre-commit`:

``` bash
#!/bin/sh

echo "Running Java formatting check..."

./gradlew spotlessCheck

if [ $? -ne 0 ]; then
    echo "Formatting check failed!"
    echo "Run: ./gradlew spotlessApply"

    exit 1
fi

echo "Formatting check passed!"
```

Сделай файл исполняемым:

``` bash
chmod +x .git/hooks/pre-commit
```

Теперь при попытке создать commit Git сначала проверит форматирование.
Если оно нарушено, commit будет отклонён.

**Почему не стоит автоматически форматировать файлы в pre-commit?**

Автоматическое изменение файлов во время создания commit может привести
к неожиданным изменениям staged/unstaged состояния. Лучше явно запускать
`spotlessApply`, а hook использовать для проверки.

Для командной разработки можно использовать Husky или другой менеджер
Git Hooks, но это необязательно: Gradle и обычные Git Hooks уже решают
задачу.

## 7. Настройка IntelliJ IDEA

Для IntelliJ IDEA:

1.  Открой Settings.
2.  Перейди в Editor → Code Style → Java.
3.  Выбери Google Java Format, если установлен соответствующий плагин.
4.  Настрой UTF-8.
5.  Используй JDK 21.
6.  Включи оптимизацию импортов при необходимости.

Полезные действия:

  Действие           Shortcut
  ------------------ ------------------
  Format Code        `Ctrl + Alt + L`
  Optimize Imports   `Ctrl + Alt + O`
  Reformat File      `Ctrl + Alt + L`

Принципиальная разница:

-   **IDE** --- удобство разработчика.
-   **Spotless** --- единый источник истины.
-   **GitHub Actions** --- контроль качества перед merge.

Даже если разработчик использует VS Code, Eclipse или другую IDE, CI
должен выдавать одинаковый результат.

## 8. Настройка `.gitignore`

Убедись, что временные файлы и результаты сборки не попадают в Git:

``` gitignore
# Gradle
.gradle/
build/
**/build/

# IntelliJ IDEA
.idea/
*.iml

# Eclipse
.project
.classpath
.settings/

# VS Code
.vscode/

# OS
.DS_Store
Thumbs.db

# Logs
*.log
logs/
```

При этом обязательно коммить:

``` text
gradlew
gradlew.bat
gradle/wrapper/
```

Gradle Wrapper гарантирует использование согласованной версии Gradle
всеми разработчиками и CI.

## 9. Защита main branch в GitHub

Открой:

`GitHub → Repository → Settings → Rules → Rulesets`

Настрой правила для `main`:

  Правило                             Значение
  ----------------------------------- ----------------
  Require a pull request              Enabled
  Require status checks               Enabled
  Required check                      `Code Quality`
  Require branches to be up to date   Enabled
  Block force pushes                  Enabled
  Restrict deletions                  Enabled

Результат: нельзя отправить в `main` код, который не прошёл обязательную
проверку.

Дополнительно рекомендуется запретить обход обязательных проверок для
обычных участников команды.

## 10. Если проект содержит несколько Gradle-модулей

Например:

``` text
aws-dbs/
├── api-gateway/
├── document-service/
├── notification-service/
├── common/
├── config/
│   └── checkstyle/
├── build.gradle.kts
└── settings.gradle.kts
```

В этом случае важно не ограничивать Spotless только исходниками
корневого модуля.

Для общего форматирования можно использовать в корневом
`build.gradle.kts`:

``` kotlin
subprojects {
    apply(plugin = "com.diffplug.spotless")

    configure<com.diffplug.gradle.spotless.SpotlessExtension> {
        java {
            target("src/**/*.java")

            googleJavaFormat("1.36.1")

            removeUnusedImports()
            trimTrailingWhitespace()
            endWithNewline()
        }
    }
}
```

Checkstyle также лучше централизованно настроить через convention
plugin, если проект большой.

В результате все модули будут использовать одинаковые правила.

## 11. Рекомендуемая структура production-ready проекта

``` text
project/
│
├── .github/
│   └── workflows/
│       ├── java-quality.yml
│       └── build.yml
│
├── .git/
│   └── hooks/
│       └── pre-commit
│
├── config/
│   └── checkstyle/
│       └── checkstyle.xml
│
├── gradle/
│   └── wrapper/
│
├── src/
│   ├── main/
│   │   └── java/
│   └── test/
│       └── java/
│
├── .editorconfig
├── .gitignore
├── build.gradle.kts
├── settings.gradle.kts
├── gradle.properties
└── gradlew
```

Рекомендуется добавить `.editorconfig`:

``` ini
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true

[*.java]
indent_style = space
indent_size = 4

[*.{yml,yaml,json,xml}]
indent_style = space
indent_size = 2

[*.md]
trim_trailing_whitespace = false
```

Это помогает унифицировать базовые настройки редакторов.

## 12. Итоговый workflow разработчика

1.  **Разработка** --- разработчик пишет Java-код в IntelliJ IDEA.

2.  **Автоматическое форматирование:**

    ``` bash
    ./gradlew spotlessApply
    ```

3.  **Локальная проверка:**

    ``` bash
    ./gradlew clean check
    ```

4.  **Git commit:**

    ``` bash
    git add .
    git commit -m "feat: add document storage"
    ```

    Pre-commit проверяет форматирование.

5.  **Pull Request** --- разработчик отправляет изменения в GitHub.

6.  **GitHub Actions** --- выполняются форматирование, Checkstyle, тесты
    и сборка.

7.  **Merge** --- разрешён только после успешного прохождения
    обязательных проверок.

## 13. Финальная рекомендация

  Инструмент                           Обязательность
  ------------------------------------ ----------------
  Spotless                             Обязательно
  Google Java Format                   Обязательно
  Gradle Wrapper                       Обязательно
  GitHub Actions                       Обязательно
  Branch protection                    Обязательно
  Checkstyle                           Рекомендуется
  Git Hooks                            Рекомендуется
  EditorConfig                         Рекомендуется
  IntelliJ Google Java Format plugin   Опционально

**Важное различие:** Spotless отвечает за форматирование, Checkstyle ---
за соглашения и статические проверки оформления. Ни один из них не
заменяет полноценный static analysis вроде SonarQube.

Полезная официальная документация:

-   [Spotless Gradle Plugin](https://github.com/diffplug/spotless)
-   [Google Java Format](https://github.com/google/google-java-format)
-   [Gradle Checkstyle
    Plugin](https://docs.gradle.org/current/userguide/checkstyle_plugin.html)
-   [Gradle
    Plugins](https://docs.gradle.org/current/userguide/plugins.html)

**Итоговый рекомендуемый набор для Java 21 / Spring Boot:** Spotless +
Google Java Format + Checkstyle + GitHub Actions + Branch Protection.
Это хорошая основа, которую можно применять как в небольшом Spring Boot
сервисе, так и в большом multi-module или microservices проекте.

# Автоформатирование React проекта (emarket-web)

**Стек:** React, TypeScript, Vite, GitHub Actions, Husky

## 1. Рекомендуемый набор инструментов

  -----------------------------------------------------------------------
  Инструмент                          Ответственность
  ----------------------------------- -----------------------------------
  Prettier                            Единое форматирование кода

  ESLint                              Статический анализ и проверка
                                      качества

  `eslint-plugin-unused-imports`      Обнаружение и автоматическое
                                      удаление неиспользуемых imports

  lint-staged                         Запуск проверок только для
                                      staged-файлов

  Husky                               Git hooks

  Commitlint                          Проверка Conventional Commits

  GitHub Actions                      CI quality gate

  GitHub Branch Protection            Защита от merge при неуспешных
                                      проверках
  -----------------------------------------------------------------------

**Принцип разделения ответственности:** Prettier отвечает за
форматирование, ESLint --- за качество кода. Не следует дублировать
правила форматирования в ESLint.

## 2. Целевая структура проекта

``` text
emarket-web/
├── .github/
│   └── workflows/
│       └── ci.yml
├── .husky/
│   ├── pre-commit
│   └── commit-msg
├── .prettierignore
├── .prettierrc.json
├── eslint.config.js
├── lint-staged.config.js
├── tsconfig.json
├── package.json
└── src/
```

Существующие настройки Commitlint следует сохранить. Добавляются только
необходимые настройки форматирования и linting.

## 3. Установка зависимостей

Установка devDependencies:

``` bash
npm install -D \
  prettier \
  eslint \
  @eslint/js \
  typescript-eslint \
  eslint-config-prettier \
  eslint-plugin-unused-imports \
  eslint-plugin-react-hooks \
  eslint-plugin-react-refresh \
  globals \
  lint-staged
```

Если Husky ещё не установлен:

``` bash
npm install -D husky
```

Если Husky уже установлен и настроен, повторно устанавливать его не
нужно.

Используется современный ESLint Flat Config (`eslint.config.js`).

## 4. Конфигурация Prettier

### `.prettierrc.json`

``` json
{
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false,
  "semi": true,
  "singleQuote": true,
  "trailingComma": "all",
  "bracketSpacing": true,
  "arrowParens": "always",
  "endOfLine": "lf"
}
```

### `.prettierignore`

``` gitignore
node_modules/
dist/
coverage/
.vite/
*.tsbuildinfo

package-lock.json
```

Эта конфигурация задаёт единый стиль для поддерживаемых Prettier файлов:
TypeScript, TSX, JSON, CSS/SCSS, Markdown и YAML.

## 5. Конфигурация ESLint

### `eslint.config.js`

``` javascript
import js from '@eslint/js';
import prettier from 'eslint-config-prettier';
import reactHooks from 'eslint-plugin-react-hooks';
import reactRefresh from 'eslint-plugin-react-refresh';
import unusedImports from 'eslint-plugin-unused-imports';
import globals from 'globals';
import tseslint from 'typescript-eslint';

export default tseslint.config(
  {
    ignores: [
      'dist/**',
      'coverage/**',
      'node_modules/**',
      '*.config.js',
      '*.config.ts',
    ],
  },

  js.configs.recommended,

  ...tseslint.configs.recommended,

  {
    files: ['**/*.{ts,tsx}'],

    languageOptions: {
      globals: globals.browser,
    },

    plugins: {
      'react-hooks': reactHooks,
      'react-refresh': reactRefresh,
      'unused-imports': unusedImports,
    },

    rules: {
      // React Hooks
      'react-hooks/rules-of-hooks': 'error',
      'react-hooks/exhaustive-deps': 'warn',

      // React Fast Refresh
      'react-refresh/only-export-components': [
        'warn',
        {
          allowConstantExport: true,
        },
      ],

      // Automatically remove unused imports
      'unused-imports/no-unused-imports': 'error',

      // Detect unused variables
      '@typescript-eslint/no-unused-vars': 'off',
      'no-unused-vars': 'off',

      'unused-imports/no-unused-vars': [
        'warn',
        {
          vars: 'all',
          varsIgnorePattern: '^_',
          args: 'after-used',
          argsIgnorePattern: '^_',
          ignoreRestSiblings: true,
        },
      ],
    },
  },

  // Prettier takes care of formatting
  prettier,
);
```

### Как работает удаление imports

До запуска ESLint:

``` tsx
import { useState, useEffect } from 'react';
import axios from 'axios';
import { Button } from '@/shared/ui';

export function Example() {
  const [value, setValue] = useState(0);

  return <Button onClick={() => setValue(value + 1)} />;
}
```

После автоматического исправления:

``` tsx
import { useState } from 'react';

export function Example() {
  const [value, setValue] = useState(0);

  return <Button onClick={() => setValue(value + 1)} />;
}
```

Неиспользуемые `useEffect` и `axios` удаляются. Неиспользуемые обычные
переменные обнаруживаются как предупреждения.

## 6. Конфигурация lint-staged

### `lint-staged.config.js`

``` javascript
export default {
  '*.{js,jsx,ts,tsx}': [
    'eslint --fix',
    'prettier --write',
  ],

  '*.{json,md,css,scss,html,yml,yaml}': [
    'prettier --write',
  ],
};
```

При коммите обрабатываются только файлы, добавленные в Git staging area.

Порядок выполнения:

1.  ESLint удаляет неиспользуемые imports и исправляет доступные
    проблемы.
2.  Prettier форматирует итоговый код.

## 7. Скрипты package.json

Добавить следующие scripts, сохранив остальные существующие scripts и
зависимости:

``` json
{
  "scripts": {
    "prepare": "husky",

    "format": "prettier --write .",
    "format:check": "prettier --check .",

    "lint": "eslint . --max-warnings=0",
    "lint:fix": "eslint . --fix",

    "typecheck": "tsc -b",

    "test:run": "vitest run",

    "check": "npm run format:check && npm run lint && npm run typecheck && npm run test:run"
  }
}
```

`tsc -b` подходит для стандартной Vite-структуры с TypeScript project
references. Если конфигурация TypeScript в проекте отличается, команду
следует адаптировать.

`test:run` предполагает, что Vitest уже установлен и настроен.

## 8. Настройка Husky

Существующую интеграцию Commitlint необходимо сохранить.

### `.husky/pre-commit`

``` bash
npx lint-staged
```

### `.husky/commit-msg`

Если Commitlint уже настроен, оставляем существующую команду. Например:

``` bash
npx --no -- commitlint --edit "$1"
```

Ответственность hooks:

-   `pre-commit` --- форматирование и ESLint для staged-файлов.
-   `commit-msg` --- проверка Conventional Commits.

Если Husky уже настроен через `prepare`, не нужно повторно запускать
`husky init` и перезаписывать существующие hooks.

## 9. GitHub Actions

Создать workflow `.github/workflows/ci.yml`:

``` yaml
name: Frontend CI

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
  group: frontend-ci-${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  quality:
    name: Code Quality

    runs-on: ubuntu-latest
    timeout-minutes: 15

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Check formatting
        run: npm run format:check

      - name: ESLint
        run: npm run lint

      - name: TypeScript
        run: npm run typecheck

      - name: Unit tests
        run: npm run test:run

      - name: Production build
        run: npm run build
```

CI не исправляет файлы автоматически. Он проверяет, что разработчик
привёл изменения в соответствие с правилами.

Для production CI Pull Request должен завершаться ошибкой, если
форматирование, ESLint, TypeScript, тесты или сборка не проходят.

**Примечание:** версии GitHub Actions (`actions/checkout`,
`actions/setup-node`) следует регулярно обновлять и при необходимости
фиксировать на проверенные commit SHA согласно политике безопасности
репозитория.

## 10. Локальные команды разработчика

  Команда                  Назначение
  ------------------------ -----------------------------------
  `npm run format`         Форматирование всего проекта
  `npm run format:check`   Проверка форматирования
  `npm run lint`           Проверка ESLint
  `npm run lint:fix`       Автоматическое исправление ESLint
  `npm run typecheck`      Проверка TypeScript
  `npm run test:run`       Unit-тесты
  `npm run check`          Полная локальная проверка

Первоначальная настройка:

``` bash
npm install
npm run format
npm run lint:fix
npm run check
```

## 11. Git workflow

``` text
Developer
    |
    v
Code changes
    |
    v
git add
    |
    v
Husky: pre-commit
    |
    +--> lint-staged
    |       |
    |       +--> ESLint --fix
    |       |       |
    |       |       +--> Remove unused imports
    |       |
    |       +--> Prettier --write
    |
    v
Husky: commit-msg
    |
    +--> Commitlint
    |
    v
GitHub Pull Request
    |
    v
GitHub Actions
    |
    +--> Prettier check
    +--> ESLint
    +--> TypeScript
    +--> Unit tests
    +--> Production build
    |
    v
Required checks passed
    |
    v
Code Review
    |
    v
Merge
```

## 12. GitHub Branch Protection

Для веток `main` и `develop` рекомендуется включить:

-   Require a pull request before merging.
-   Require approvals.
-   Require status checks to pass before merging.
-   Required status check: `Code Quality`.
-   Require conversation resolution.
-   Block force pushes.

Название required check должно соответствовать фактическому имени job в
GitHub Actions.

## 13. Итоговые архитектурные решения

  Инструмент                       Ответственность
  -------------------------------- -----------------------------
  Prettier                         Code formatting
  ESLint                           Static code analysis
  `eslint-plugin-unused-imports`   Remove unused imports
  lint-staged                      Проверка staged files
  Husky                            Git hooks
  Commitlint                       Commit message validation
  GitHub Actions                   CI quality gate
  Branch Protection                Защита от невалидного merge

### Рекомендации

1.  Не запускать Prettier и ESLint для всего проекта в `pre-commit`.
    Использовать `lint-staged`.
2.  Полную проверку выполнять в GitHub Actions.
3.  Не допускать автоматического исправления файлов непосредственно в
    CI.
4.  Сохранять единые правила форматирования во всех ветках.
5.  Добавить `Code Quality` как обязательную проверку в GitHub Branch
    Protection.
6.  Не перезаписывать существующие настройки Husky и Commitlint.

Такое решение подходит для Stage 1 `emarket-web` и масштабируется при
добавлении новых функциональных модулей.

## 14. Официальная документация

-   [ESLint ---
    Configuration](https://eslint.org/docs/latest/use/configure/)
-   [typescript-eslint --- Getting
    Started](https://typescript-eslint.io/getting-started/)
-   [Prettier --- Installation](https://prettier.io/docs/install)
-   [Prettier --- Integrating with
    Linters](https://prettier.io/docs/integrating-with-linters)
-   [Husky](https://typicode.github.io/husky/)
-   [lint-staged](https://github.com/lint-staged/lint-staged)
-   [GitHub Actions](https://docs.github.com/en/actions)
-   [GitHub Branch
    Protection](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches)

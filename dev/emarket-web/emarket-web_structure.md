# Production-ready структура проекта `emarket-web`

**Технологии:** React + TypeScript + Vite\
**Архитектура:** Feature-Sliced Design (FSD) с практическими адаптациями
под e-commerce.

`emarket-web` отвечает за пользовательский интерфейс интернет-магазина и
взаимодействует только с `emarket-api-gateway`. Frontend не должен знать
адреса или внутреннее устройство BFF и микросервисов.

## 1. Рекомендуемая структура проекта

``` text
emarket-web/
├── public/
│   ├── favicon.svg
│   ├── logo.svg
│   └── locales/
│       ├── en/
│       └── bg/
├── src/
│   ├── app/
│   │   ├── providers/
│   │   │   ├── AppProviders.tsx
│   │   │   ├── RouterProvider.tsx
│   │   │   ├── QueryProvider.tsx
│   │   │   └── ThemeProvider.tsx
│   │   ├── router/
│   │   │   ├── index.tsx
│   │   │   ├── routes.ts
│   │   │   └── guards/
│   │   ├── styles/
│   │   │   ├── globals.css
│   │   │   ├── variables.css
│   │   │   └── themes/
│   │   ├── App.tsx
│   │   └── main.tsx
│   ├── pages/
│   │   ├── home/
│   │   ├── catalog/
│   │   ├── product-details/
│   │   ├── cart/
│   │   ├── checkout/
│   │   ├── login/
│   │   ├── register/
│   │   ├── profile/
│   │   ├── orders/
│   │   ├── admin/
│   │   └── not-found/
│   ├── widgets/
│   │   ├── header/
│   │   ├── footer/
│   │   ├── navigation/
│   │   ├── sidebar/
│   │   ├── product-grid/
│   │   ├── product-filters/
│   │   ├── shopping-cart/
│   │   └── admin-sidebar/
│   ├── features/
│   │   ├── auth/
│   │   │   ├── login/
│   │   │   ├── register/
│   │   │   └── logout/
│   │   ├── product/
│   │   │   ├── search/
│   │   │   ├── filter/
│   │   │   └── sort/
│   │   ├── cart/
│   │   │   ├── add-to-cart/
│   │   │   ├── remove-from-cart/
│   │   │   └── update-quantity/
│   │   ├── wishlist/
│   │   ├── checkout/
│   │   ├── reviews/
│   │   └── theme-switch/
│   ├── entities/
│   │   ├── user/
│   │   ├── product/
│   │   ├── category/
│   │   ├── cart/
│   │   ├── order/
│   │   ├── payment/
│   │   └── review/
│   ├── shared/
│   │   ├── api/
│   │   │   ├── client/
│   │   │   ├── generated/
│   │   │   ├── interceptors/
│   │   │   └── types/
│   │   ├── ui/
│   │   │   ├── button/
│   │   │   ├── input/
│   │   │   ├── modal/
│   │   │   ├── dropdown/
│   │   │   ├── loader/
│   │   │   ├── skeleton/
│   │   │   └── typography/
│   │   ├── hooks/
│   │   ├── lib/
│   │   ├── config/
│   │   ├── constants/
│   │   ├── types/
│   │   ├── utils/
│   │   └── assets/
│   │       ├── images/
│   │       ├── icons/
│   │       └── fonts/
│   └── test/
│       ├── setup/
│       ├── mocks/
│       ├── fixtures/
│       └── utils/
├── e2e/
│   ├── tests/
│   ├── fixtures/
│   └── playwright.config.ts
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── code-quality.yml
├── .husky/
├── .vscode/
├── .env.example
├── .env.development
├── .env.production
├── .gitignore
├── eslint.config.js
├── prettier.config.js
├── tsconfig.json
├── vite.config.ts
├── vitest.config.ts
├── package.json
├── package-lock.json
└── README.md
```

Это целевая структура, а не требование создать все директории сразу.
Добавляй папки по мере появления функциональности. Не оставляй множество
пустых директорий «на будущее».

## 2. Назначение папок

### `public/` --- статические файлы

Файлы, которые Vite копирует в итоговую сборку без обработки модульным
графом.

  Путь                Назначение
  ------------------- --------------------------------------------------
  `public/`           Статические ресурсы, доступные по публичному URL
  `public/locales/`   Файлы переводов, если i18n загружает их напрямую
  `favicon.svg`       Иконка браузера
  `logo.svg`          Статический логотип

Не размещай здесь React-компоненты и TypeScript-код. Ресурсы, которые
нужно импортировать из компонентов и обрабатывать Vite, лучше хранить в
`src/shared/assets/`.

### `src/app/` --- инициализация приложения

Корневой слой конфигурации React-приложения.

  Путь                   Назначение
  ---------------------- ------------------------------------------------
  `providers/`           Глобальные React Providers
  `AppProviders.tsx`     Композиция провайдеров приложения
  `RouterProvider.tsx`   Подключение маршрутизации
  `QueryProvider.tsx`    Настройка TanStack Query
  `ThemeProvider.tsx`    Управление темой интерфейса
  `router/`              Конфигурация маршрутов
  `router/guards/`       Защита маршрутов: авторизация и проверка ролей
  `styles/`              Глобальные CSS-стили, переменные и темы
  `App.tsx`              Корневой компонент приложения
  `main.tsx`             Точка входа React-приложения

Пример `main.tsx`:

``` tsx
import React from 'react';
import ReactDOM from 'react-dom/client';

import { App } from './App';
import './styles/globals.css';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
);
```

### `src/pages/` --- страницы приложения

Каждая папка представляет страницу или маршрут. Страница в основном
компонуется из widgets, features и entities, а не содержит всю
бизнес-логику.

  Папка                Назначение
  -------------------- ---------------------------
  `home/`              Главная страница
  `catalog/`           Каталог товаров
  `product-details/`   Детальная страница товара
  `cart/`              Страница корзины
  `checkout/`          Оформление заказа
  `login/`             Страница входа
  `register/`          Страница регистрации
  `profile/`           Профиль пользователя
  `orders/`            История заказов
  `admin/`             Административная часть
  `not-found/`         Страница 404

Пример:

``` text
pages/
└── catalog/
    ├── ui/
    │   └── CatalogPage.tsx
    └── index.ts
```

### `src/widgets/` --- крупные UI-блоки

Самостоятельные композиционные блоки интерфейса, которые могут
использоваться на нескольких страницах.

  Папка                Назначение
  -------------------- ----------------------------------
  `header/`            Шапка сайта
  `footer/`            Подвал сайта
  `navigation/`        Основная навигация
  `sidebar/`           Боковая панель
  `product-grid/`      Сетка товаров
  `product-filters/`   Панель фильтрации
  `shopping-cart/`     Композиционный виджет корзины
  `admin-sidebar/`     Навигация административной части

Например, `Header` может объединять логотип, поиск, навигацию, кнопку
входа и индикатор корзины. Это готовый UI-блок, а не набор базовых
элементов вроде кнопок и полей ввода.

### `src/features/` --- пользовательские действия

Здесь реализуются действия пользователя, имеющие конкретный сценарий или
бизнес-смысл.

  Папка                      Назначение
  -------------------------- -----------------------------------
  `auth/login/`              Вход пользователя
  `auth/register/`           Регистрация
  `auth/logout/`             Выход
  `product/search/`          Поиск товаров
  `product/filter/`          Фильтрация товаров
  `product/sort/`            Сортировка товаров
  `cart/add-to-cart/`        Добавление товара в корзину
  `cart/remove-from-cart/`   Удаление товара из корзины
  `cart/update-quantity/`    Изменение количества
  `wishlist/`                Управление избранным
  `checkout/`                Сценарии оформления заказа
  `reviews/`                 Создание и редактирование отзывов
  `theme-switch/`            Переключение темы

Пример структуры feature:

``` text
features/
└── cart/
    └── add-to-cart/
        ├── ui/
        │   └── AddToCartButton.tsx
        ├── model/
        │   └── useAddToCart.ts
        ├── api/
        │   └── addToCart.ts
        └── index.ts
```

`ui/` отвечает за отображение, `model/` --- за состояние и логику
сценария, `api/` --- за вызов backend API. Не обязательно создавать все
эти папки, если модуль небольшой.

### `src/entities/` --- бизнес-сущности

Слой основных предметных объектов приложения.

  Папка         Назначение
  ------------- ------------------
  `user/`       Пользователь
  `product/`    Товар
  `category/`   Категория товара
  `cart/`       Корзина
  `order/`      Заказ
  `payment/`    Платёж
  `review/`     Отзыв

Пример:

``` text
entities/
└── product/
    ├── api/
    │   └── getProduct.ts
    ├── model/
    │   ├── types.ts
    │   └── queries.ts
    ├── ui/
    │   ├── ProductCard.tsx
    │   └── ProductPrice.tsx
    └── index.ts
```

Разделение ответственности:

-   `entities/product` --- модель товара, его типы, отображение и
    получение данных.
-   `features/product/search` --- пользовательский сценарий поиска.
-   `widgets/product-grid` --- отображение группы товаров.
-   `pages/catalog` --- композиция страницы каталога.

### `src/shared/` --- общая инфраструктура

Переиспользуемые элементы, не принадлежащие конкретной бизнес-сущности
или пользовательскому сценарию.

  Папка                 Назначение
  --------------------- --------------------------------------------
  `api/`                Общая HTTP-инфраструктура
  `api/client/`         Настройка HTTP-клиента
  `api/generated/`      Клиенты и типы, сгенерированные из OpenAPI
  `api/interceptors/`   HTTP interceptors
  `api/types/`          Общие типы API
  `ui/`                 Переиспользуемые UI-компоненты
  `hooks/`              Общие React hooks
  `lib/`                Адаптеры и настройки сторонних библиотек
  `config/`             Общие настройки приложения
  `constants/`          Общие константы
  `types/`              Общие TypeScript-типы
  `utils/`              Вспомогательные функции
  `assets/`             Изображения, иконки и шрифты

Примеры:

-   `shared/ui/button` --- универсальная кнопка.
-   `shared/ui/modal` --- универсальное модальное окно.
-   `shared/hooks/useDebounce` --- общий debounce hook.
-   `shared/utils/formatCurrency` --- форматирование цены.
-   `shared/api/client` --- единый HTTP-клиент для API Gateway.

### `src/test/` --- общие тестовые инструменты

  Папка         Назначение
  ------------- ---------------------------------------
  `setup/`      Настройка тестового окружения
  `mocks/`      Общие моки
  `fixtures/`   Повторно используемые тестовые данные
  `utils/`      Тестовые утилиты и render helpers

Тесты конкретных компонентов, features и entities удобно размещать рядом
с исходным кодом.

### `e2e/` --- End-to-End тестирование

Для E2E-тестов рекомендуется Playwright.

Примеры пользовательских сценариев:

-   Регистрация пользователя.
-   Авторизация.
-   Поиск товара.
-   Добавление товара в корзину.
-   Оформление заказа.
-   Работа административной панели.

### `.github/workflows/` --- CI/CD

GitHub Actions workflows:

-   `ci.yml` --- установка зависимостей, lint, typecheck, unit tests и
    build.
-   `code-quality.yml` --- дополнительные проверки качества кода.

## 3. Архитектурные зависимости между слоями

В FSD зависимости направляются от верхних слоёв к нижним:

``` text
App
 ↓
Pages
 ↓
Widgets
 ↓
Features
 ↓
Entities
 ↓
Shared
```

Правила:

-   `pages/catalog` может использовать `widgets/product-grid`.
-   `widgets/product-grid` может использовать `entities/product`.
-   `features/cart/add-to-cart` может использовать `entities/product`.
-   `entities/product` может использовать `shared/api`.
-   `shared` не должен импортировать `entities`, `features`, `widgets`
    или `pages`.
-   Не допускай циклических зависимостей между модулями.

Схема показывает общее направление зависимостей, а не обязательную
цепочку: например, feature может использовать entity напрямую, не
проходя через widget.

## 4. Взаимодействие с `emarket-api-gateway`

Frontend взаимодействует только с Gateway:

``` text
emarket-web
    |
    | HTTP / HTTPS
    v
emarket-api-gateway
    |
    v
emarket-bff-web
    |
    v
Backend services
```

Централизуй HTTP-коммуникацию в `shared/api/`. Не создавай отдельный
HTTP-клиент для каждой feature.

Рекомендуемая структура:

``` text
shared/
└── api/
    ├── client/
    │   ├── apiClient.ts
    │   └── apiConfig.ts
    ├── interceptors/
    │   ├── requestInterceptor.ts
    │   └── responseInterceptor.ts
    ├── generated/
    └── types/
```

Пример конфигурации:

``` typescript
// shared/api/client/apiConfig.ts

export const API_CONFIG = {
  baseURL: import.meta.env.VITE_API_BASE_URL,
  timeout: 15_000,
} as const;
```

`.env.development`:

``` dotenv
VITE_API_BASE_URL=http://localhost:8080/api/v1
```

`.env.production`:

``` dotenv
VITE_API_BASE_URL=https://api.emarket.com/api/v1
```

Для локальной разработки также можно использовать Vite proxy и
относительный URL `/api/v1`, направляя запросы на локальный Gateway.

Все запросы должны проходить через единый HTTP-клиент. Не отправляй
запросы напрямую из React-компонентов.

## 5. Стандартная структура модуля

Для крупных модулей используй единый шаблон:

``` text
module-name/
├── api/
│   └── ...
├── model/
│   ├── types.ts
│   ├── queries.ts
│   └── ...
├── ui/
│   └── ...
├── lib/
│   └── ...
└── index.ts
```

  Папка        Ответственность
  ------------ -----------------------------------
  `api/`       HTTP-запросы и API-операции
  `model/`     Типы, состояние и бизнес-логика
  `ui/`        React-компоненты
  `lib/`       Локальные вспомогательные функции
  `index.ts`   Публичный API модуля

Не создавай все пять папок автоматически: добавляй только те, которые
действительно нужны.

## 6. Рекомендуемый технологический стек

  Область        Рекомендация
  -------------- --------------------------------
  Архитектура    Feature-Sliced Design
  Language       TypeScript Strict Mode
  UI             React
  Build          Vite
  Routing        React Router
  Server State   TanStack Query
  Client State   Zustand
  Forms          React Hook Form
  Validation     Zod
  HTTP           Axios
  Styling        CSS Modules
  i18n           i18next + react-i18next
  Unit Testing   Vitest + React Testing Library
  E2E Testing    Playwright
  API Types      OpenAPI TypeScript generation
  Code Quality   ESLint + Prettier

### Важные правила

1.  Не создавай глобальный Zustand store для всех данных. Серверное
    состояние преимущественно управляй через TanStack Query.
2.  Не размещай бизнес-логику в `pages`.
3.  Не превращай `shared/ui` в хранилище всех компонентов приложения.
4.  Не допускай прямых HTTP-запросов из React-компонентов.
5.  Используй `index.ts` как публичную точку экспорта модулей.
6.  Загружай страницы лениво через `React.lazy` или механизм lazy
    loading выбранного роутера.
7.  Не храни секреты в переменных с префиксом `VITE_`: они доступны
    клиентскому коду.
8.  Не смешивай административные и пользовательские сценарии без
    архитектурной необходимости.

## 7. Что реализовать на первом этапе

Не нужно сразу создавать все директории целевой структуры. Начни с
минимального архитектурного каркаса:

``` text
src/
├── app/
│   ├── providers/
│   ├── router/
│   ├── styles/
│   ├── App.tsx
│   └── main.tsx
├── pages/
│   ├── home/
│   └── not-found/
├── widgets/
│   ├── header/
│   └── footer/
├── features/
│   └── auth/
├── entities/
│   └── user/
├── shared/
│   ├── api/
│   ├── ui/
│   ├── hooks/
│   ├── config/
│   ├── types/
│   └── utils/
└── test/
    ├── setup/
    ├── mocks/
    └── utils/
```

Дальнейшее расширение:

1.  Authentication & Authorization.
2.  Product Catalog.
3.  Shopping Cart & Wishlist.
4.  Checkout & Orders.
5.  Admin Panel.
6.  E2E Testing & Performance.

## 8. Итоговая рекомендация

Используй FSD как архитектурную основу `emarket-web`, но не внедряй её
чрезмерно формально. Для проекта это сбалансированный подход к
поддерживаемости и масштабированию.

Главные архитектурные границы:

-   `app` --- инициализация приложения.
-   `pages` --- страницы и маршруты.
-   `widgets` --- крупные UI-блоки.
-   `features` --- пользовательские действия.
-   `entities` --- бизнес-сущности.
-   `shared` --- общая инфраструктура.
-   `test` и `e2e` --- тестовая инфраструктура и сквозные сценарии.

**Ключевое правило интеграции:** `emarket-web` обращается исключительно
к `emarket-api-gateway`. Внутренние backend-сервисы и BFF остаются
скрытыми от frontend.

# Архитектура и Технологический Стек `emarket-web`

Настоящий документ содержит описание технологического стека, архитектурных решений и стандартов разработки для клиентского веб-приложения **`emarket-web`** (Stage 1).

Стек ориентирован на создание современного production-ready e-commerce приложения с масштабированием от базового этапа до полноценной платформы с каталогом, корзиной, заказами, оплатой и административной панелью.

---

## 1. Рекомендуемый технологический стек

| Категория | Технология | Назначение |
| :--- | :--- | :--- |
| **Framework** | React 19 | Отрисовка UI и декларативные компоненты |
| **Language** | TypeScript | Статическая типизация |
| **Build tool** | Vite | Сборка для Development / Production |
| **Routing** | React Router 7 | Client-side навигация |
| **Server state** | TanStack Query 5 | Запросы к API, кэширование, синхронизация |
| **Client state** | Zustand | Локальное и UI состояние |
| **HTTP Client** | Axios | HTTP-запросы, interceptors |
| **Validation** | Zod | Runtime-валидация схем и данных |
| **Forms** | React Hook Form | Управление состоянием форм |
| **UI Components** | shadcn/ui | Компонентная система |
| **UI Primitives** | Radix UI | Доступность (Accessibility) |
| **CSS** | Tailwind CSS 4 | Стилизация |
| **Icons** | Lucide React | Набор иконок |
| **i18n** | i18next + react-i18next | Интернационализация |
| **API Contracts** | OpenAPI + openapi-typescript | Автогенерация типов TypeScript |
| **Unit Testing** | Vitest | Юнит-тестирование |
| **Component Testing** | React Testing Library | Тестирование React-компонентов |
| **API Mocking** | MSW (Mock Service Worker) | Имитация ответов бэкенда |
| **E2E Testing** | Playwright | End-to-end тестирование |
| **Code Quality** | ESLint + Prettier | Линтинг и форматирование кода |
| **Git Hooks** | Husky + lint-staged + Commitlint | Проверки перед коммитом |
| **Error Monitoring**| Sentry | Мониторинг ошибок на клиенте |
| **Performance** | Web Vitals | Отслеживание метрик производительности |

---

## 2. Архитектура Frontend

Для проекта рекомендуется использовать **Featured-Sliced Design (FSD)**.

```text
   src/
   ├── app/                  # Инициализация приложения
   │
   ├── pages/                # Страницы приложения (маршруты)
   │
   ├── widgets/              # Крупные композитные блоки интерфейса
   │
   ├── features/             # Пользовательские сценарии (интерактивные действия)
   │
   ├── entities/             # Бизнес-сущности
   │
   └── shared/               # Переиспользуемый код без бизнес-логики
```

### Преимущества подхода
Каждая бизнес-фича (`feature`) изолирована и включает собственные:
* API-методы
* React-компоненты
* Хуки
* Типы TypeScript
* Схемы валидации

Модуль `features/auth` не зависит напрямую от реализации `features/profile`. При масштабировании проекта (добавление Cart, Orders, Payments) архитектура сохранит свою устойчивость.

---

## 3. Взаимодействие с Backend (API Gateway)

### Ключевое архитектурное правило
Frontend взаимодействует **исключительно** с `emarket-api-gateway`. Взаимодействие с бэкенд-сервисами напрямую не допускается.

```text
                 Browser
                    |
                    v
               emarket-web
                    |
                    | HTTP / HTTPS
                    v
           emarket-api-gateway
             /api/v1/web/**
                    |
                    v
              emarket-bff-web
                /         \
               v           v
       identity-service  user-service
```

Frontend абстрагирован от:
* Прямого URL `emarket-bff-web`
* Внутренних адресов `identity-service`, `user-service` и т.д.
* Деталей топологии микросервисов

### Конфигурация API

`.env.development`
```env
VITE_API_BASE_URL=http://localhost:8080/api/v1
```

`.env.production`
```env
VITE_API_BASE_URL=/api/v1
```

> **Примечание:** В production рекомендуется use-origin конфигурация, когда маршрутизатор направляет запрос `/api/v1` на API Gateway. Если Gateway находится на отдельном домене, указывается абсолютный HTTPS URL.

### Конфигурация Axios

```typescript
import axios from 'axios';

export const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  timeout: 15000,
  withCredentials: true,
  headers: {
    'Content-Type': 'application/json',
  },
});
```

### Пример использования API в фиче

```typescript
// features/auth/api/auth.api.ts
import { apiClient } from '@/shared/api/client';
import type { LoginRequest } from '../types';

export const authApi = {
  login: (data: LoginRequest) =>
    apiClient.post('/web/auth/login', data),

  logout: () =>
    apiClient.post('/web/auth/logout'),

  getCurrentUser: () =>
    apiClient.get('/web/auth/me'),
};
```

### Управление состоянием (Server State vs Client State)

* **TanStack Query 5** используется для работы с Server State (получение данных, кэширование, инвалидация, повторные запросы):
  ```typescript
  export const useCurrentUser = () =>
    useQuery({
      queryKey: ['auth', 'me'],
      queryFn: authApi.getCurrentUser,
    });
  ```
* **Zustand** применяется **только** для Client State (состояние UI, модальные окна, локальные переключатели). Дублировать данные с сервера в Zustand не рекомендуется.

---

## 4. UI / Design System

Применяется стек на базе **shadcn/ui** и **Tailwind CSS 4**:

* **shadcn/ui** — переиспользуемые UI-компоненты (`Dialog`, `Button`, `Dropdown`, `Select`, `Form`, `Tabs`).
* **Tailwind CSS 4** — стилизация, дизайн-токеновые переменные, адаптивные сетки.
* **Radix UI** — стандарт доступности (accessibility primitives) и навигация с клавиатуры.

### Требования к UI:
* Стиль: **Soft Modern design**
* Подход: **Desktop-first responsive layout**
* Поддержка двух тем: **Light / Dark**
* Соответствие стандарту доступности: **WCAG 2.2 AA**

---

## 5. Формы, Валидация и API Contracts

| Технология | Ответственность |
| :--- | :--- |
| **React Hook Form** | Управление состоянием и жизненным циклом форм |
| **Zod** | Runtime-валидация входящих и исходящих данных |
| **OpenAPI** | Строгий контракт между Frontend и Backend |

### Пример валидации со Zod

```typescript
import { z } from 'zod';

export const loginSchema = z.object({
  email: z.string().email({ message: 'Некорректный адрес электронной почты' }),
  password: z.string().min(8, { message: 'Пароль должен содержать минимум 8 символов' }),
});

export type LoginRequest = z.infer<typeof loginSchema>;
```

### Автоматизация генерации типов
1. `emarket-bff-web` публикует актуальную OpenAPI-спецификацию.
2. Frontend загружает или запрашивает спецификацию в рамках сборки.
3. Пакет `openapi-typescript` генерирует строгие TypeScript-типы.
4. Сгенерированные типы импортируются в сервисный слой frontend.

---

## 6. Безопасность и Авторизация (Authentication & Security)

Используется подход **BFF-managed Session**:

| Компонент | Решение |
| :--- | :--- |
| **Session** | Управляется полностью на стороне BFF |
| **Access token** | **Не хранится** в `localStorage` / `sessionStorage` |
| **Refresh token** | Недоступен для JavaScript |
| **Session cookie** | Флаг `HttpOnly` |
| **Production cookie** | Флаг `Secure` |
| **SameSite** | `Lax` (или `Strict` в зависимости от доменной модели) |
| **CSRF** | Защита запросов, изменяющих состояние (POST, PUT, DELETE) |
| **CORS** | Белый список только разрешённых доменов |
| **HTTPS** | Обязателен для всех окружений кроме local dev |

> **Важно:** Флаг `withCredentials: true` в Axios отправляет cookie, но не отменяет необходимости обработки CSRF-токена в координации с BFF.

---

## 7. Тестирование

```text
features/
  auth/
    api/
    components/
    hooks/
    schemas/
    __tests__/
```

| Инструмент | Назначение |
| :--- | :--- |
| **Vitest** | Быстрый запуск Unit-тестов |
| **React Testing Library** | Интеграционное тестирование UI-компонентов |
| **MSW (Mock Service Worker)** | Перехват и имитация ответов API Gateway |
| **Playwright** | E2E сценарии в реальном браузере |
| **axe-core** | Автоматизированная проверка accessibility |

---

## 8. Качество кода и CI/CD

### Локальная разработка
1. **ESLint** — линтинг кода.
2. **Prettier** — форматирование.
3. **TypeScript** — статическая проверка типов.
4. **Husky + lint-staged** — автоматический запуск проверок перед коммитом.
5. **Commitlint** — проверка соответствия сообщений коммитов стандарту Conventional Commits.

### Пайплайн GitHub Actions
1. Установка зависимостей (`npm ci`)
2. Запуск ESLint & Prettier
3. Проверка типов (`tsc --noEmit`)
4. Прогон Unit/Component тестов (`Vitest`)
5. Запуск E2E тестов (`Playwright`)
6. Сборка приложения (`Vite build`)
7. Проверка безопасности зависимостей (`audit`)

### Мониторинг в Production
* **Sentry** — сбор JS-ошибок и unhandled exceptions.
* **Web Vitals** — отслеживание метрик LCP, INP, CLS.
* **Correlation ID** — сквозная проброска ID запроса с клиенте на бэкенд для упрощения отладки.

---

## 9. Деплойment и Инфраструктура

Деплоймент frontend-приложения отделён от бэкенд-инфраструктуры.

### Варианты размещения static-файлов:
* **Option A:** Docker + Nginx
* **Option B (Рекомендуемый для AWS):** AWS S3 + CloudFront (CDN)

### Чек-лист настройки окружения:
- [x] Настроен SPA Fallback на `index.html`
- [x] Настроено заголовки `Cache-Control` для статических ресурсов
- [x] Подключены Security Headers (HSTS, X-Frame-Options, X-Content-Type-Options)
- [x] Настроена политика Content Security Policy (CSP)
- [x] Принудительный редирект на HTTPS
- [x] Отсутствие секретных данных в переменных `VITE_*`

---

## 10. Ограничения Stage 1 (Что НЕ рекомендуется добавлять)

| Технология | Причина откланения на Stage 1 |
| :--- | :--- |
| **Redux Toolkit** | Избыточен; связки Zustand + TanStack Query более чем достаточно |
| **Next.js** | На текущем этапе Vite SPA покрывает все требования (пересмотр возможен при необходимости SEO для каталога) |
| **GraphQL / Apollo** | REST API через BFF полностью закрывает текущие потребности |
| **Material UI / AntD** | Конфликтует с выбранной базой на Tailwind + shadcn/ui |
| **WebSockets** | Отсутствуют бизнес-сценарии реального времени на Stage 1 |
| **Microfrontends / MF** | Преждевременная усложнённость для текущего масштаба |
| **SSR** | Избыточно для закрытой/начальной части приложения на Stage 1 |

---

## Итоговый вывод

Выбранный стек (**React 19 + TypeScript + Vite + TanStack Query + Tailwind CSS + shadcn/ui**) обеспечивает баланс между скоростью разработки, производительностью приложения и простотой поддержки.

Следование архитектурному правилу единой точки входа через **`emarket-api-gateway`** обеспечивает независимость клиентской части и позволяет эволюционировать бэкенд-микросервисам без переписывания кода frontend.

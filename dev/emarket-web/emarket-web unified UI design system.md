# Unified UI Design System для `emarket-web` на базе Feature-Sliced Design (FSD)

Для `emarket-web` рекомендуется внедрить единый **UI Design System** на уровне `shared` согласно методологии Feature-Sliced Design (FSD).

Вместо простого выноса повторяющегося CSS в SCSS-миксины, production-ready решение должно обеспечивать единый внешний вид всех UI-компонентов и централизованное управление цветами, типографикой, размерами и состояниями:

* Все `Link` используют одинаковый цвет, размер шрифта и состояние `hover`.
* Все `Button` имеют одинаковые базовые размеры, цвета и состояния.
* Все `Input` используют одинаковые стили `border`, `focus` и `disabled`.
* Все `Label` имеют единую типографику.
* Все компоненты опираются на общие **Design Tokens**.
* Переключение **Light/Dark themes** управляется централизованно.

---

## 1. Рекомендуемая FSD-структура

```text
src/
├── app/
│   ├── providers/
│   ├── styles/
│   │   ├── _reset.scss
│   │   ├── _globals.scss
│   │   └── index.scss
│   └── App.tsx
│
├── pages/
│
├── widgets/
│
├── features/
│   ├── auth-by-email/
│   │   └── ui/
│   │       └── login-form.module.scss
│   │
│   └── request-password-reset/
│       └── ui/
│           └── forgot-password-form.module.scss
│
└── shared/
    ├── ui/
    │   ├── button/
    │   │   ├── ui/
    │   │   │   ├── button.tsx
    │   │   │   └── button.module.scss
    │   │   └── index.ts
    │   │
    │   ├── link/
    │   │   ├── ui/
    │   │   │   ├── link.tsx
    │   │   │   └── link.module.scss
    │   │   └── index.ts
    │   │
    │   ├── input/
    │   │   ├── ui/
    │   │   │   ├── input.tsx
    │   │   │   └── input.module.scss
    │   │   └── index.ts
    │   │
    │   └── label/
    │       ├── ui/
    │       │   ├── label.tsx
    │       │   └── label.module.scss
    │       └── index.ts
    │
    └── styles/
        ├── tokens/
        │   ├── _colors.scss
        │   ├── _typography.scss
        │   ├── _spacing.scss
        │   └── _index.scss
        │
        ├── themes/
        │   ├── _light.scss
        │   └── _dark.scss
        │
        ├── _mixins.scss
        └── index.scss
```

### Основные принципы разделения:
* **`shared/styles`** — общие design tokens, темы и SCSS-инструменты.
* **`shared/ui`** — переиспользуемые атомарные UI-компоненты.
* **`features`** — бизнес-функциональность, использующая готовые UI-компоненты.
* **`app/styles`** — глобальные стили приложения и их точка входа.

---

## 2. Design Tokens — единый источник стилей

Использование захардкоженных значений вроде `#737373` или `#2563eb` внутри каждого SCSS-модуля не рекомендуется. Вместо этого создаются централизованные CSS-переменные.

### `shared/styles/tokens/_colors.scss`
```scss
:root {
  // Text
  --color-text-primary: #171717;
  --color-text-secondary: #737373;
  --color-text-muted: #a7a7a7;
  --color-text-inverse: #ffffff;

  // Brand
  --color-primary: #2563eb;
  --color-primary-hover: #1855c4;

  // Borders
  --color-border: #d5d5d0;
  --color-border-focus: #2563eb;

  // Background
  --color-background: #ffffff;
  --color-background-disabled: #f5f5f5;

  // Status
  --color-error: #dc2626;
  --color-error-background: #fef2f2;
  --color-success: #16a34a;
}
```

### `shared/styles/tokens/_typography.scss`
```scss
:root {
  // Font family
  --font-family-base: Inter, Arial, sans-serif;

  // Font sizes
  --font-size-xs: 12px;
  --font-size-sm: 14px;
  --font-size-md: 16px;
  --font-size-lg: 18px;
  --font-size-xl: 20px;

  // Font weights
  --font-weight-regular: 400;
  --font-weight-medium: 500;
  --font-weight-semibold: 600;
  --font-weight-bold: 700;

  // Line heights
  --line-height-normal: 1.5;
  --line-height-relaxed: 1.6;
}
```

### `shared/styles/tokens/_spacing.scss`
```scss
:root {
  --spacing-1: 4px;
  --spacing-2: 8px;
  --spacing-3: 12px;
  --spacing-4: 16px;
  --spacing-5: 20px;
  --spacing-6: 24px;
  --spacing-8: 32px;

  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-lg: 10px;
}
```

### `shared/styles/tokens/_index.scss`
```scss
@forward 'colors';
@forward 'typography';
@forward 'spacing';
```

> Теперь изменение основного цвета или отступов всего приложения выполняется в одном месте.

---

## 3. Общий компонент `Link`

Вместо дублирования стилей ссылок внутри каждой фичи:
```scss
// ❌ Плохо: дублирование в каждом модуле
a {
  color: #737373;
  font-size: 14px;
  text-decoration: underline;

  &:hover {
    color: #2563eb;
  }
}
```

Создаётся переиспользуемый компонент в слое `shared`:

### `shared/ui/link/ui/link.module.scss`
```scss
.link {
  color: var(--color-text-secondary);
  font-size: var(--font-size-sm);
  font-weight: var(--font-weight-regular);
  text-decoration: underline;

  transition: color 150ms ease;

  &:hover {
    color: var(--color-primary-hover);
  }

  &:focus-visible {
    outline: 2px solid var(--color-primary);
    outline-offset: 3px;
  }
}
```

### `shared/ui/link/ui/link.tsx`
```tsx
import type { AnchorHTMLAttributes } from 'react';

import styles from './link.module.scss';

type LinkProps = AnchorHTMLAttributes<HTMLAnchorElement>;

export function Link({ className, ...props }: LinkProps) {
  return (
    <a
      {...props}
      className={[styles.link, className].filter(Boolean).join(' ')}
    />
  );
}
```

### `shared/ui/link/index.ts`
```ts
export { Link } from './ui/link';
```

> **Примечание:** Для приложения с React Router рекомендуется дополнительно поддержать `RouterLink` на основе `Link` из `react-router-dom`, чтобы не терять SPA-навигацию.

### Использование в формах:
```tsx
// Вместо <a href="/forgot-password">Forgot password?</a>
<Link href="/forgot-password">
  Forgot password?
</Link>

// Для React Router:
<RouterLink to="/forgot-password">
  Forgot password?
</RouterLink>
```

---

## 4. Единый `Button`

### `shared/ui/button/ui/button.module.scss`
```scss
.button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--spacing-2);

  width: 100%;
  min-height: 48px;
  padding: 0 var(--spacing-4);

  color: var(--color-text-inverse);
  font-family: var(--font-family-base);
  font-size: var(--font-size-md);
  font-weight: var(--font-weight-semibold);

  background: var(--color-primary);
  border: none;
  border-radius: var(--radius-md);

  cursor: pointer;

  transition:
    background 150ms ease,
    opacity 150ms ease;

  &:hover:not(:disabled) {
    background: var(--color-primary-hover);
  }

  &:focus-visible {
    outline: 2px solid var(--color-primary);
    outline-offset: 3px;
  }

  &:disabled {
    opacity: 0.65;
    cursor: not-allowed;
  }
}
```

### `shared/ui/button/ui/button.tsx`
```tsx
import type { ButtonHTMLAttributes } from 'react';

import styles from './button.module.scss';

type ButtonProps = ButtonHTMLAttributes<HTMLButtonElement>;

export function Button({ className, ...props }: ButtonProps) {
  return (
    <button
      {...props}
      className={[styles.button, className].filter(Boolean).join(' ')}
    />
  );
}
```

### Использование и масштабирование:
```tsx
<Button type="submit" disabled={isSubmitting}>
  Sign in
</Button>
```
В будущем компонент легко расширяется через пропсы:
* **`variant`**: `primary`, `secondary`, `outline`, `ghost`, `danger`
* **`size`**: `sm`, `md`, `lg`

---

## 5. Единые `Input` и `Label`

### `shared/ui/input/ui/input.module.scss`
```scss
.input {
  width: 100%;
  min-height: 46px;
  padding: 0 var(--spacing-4);

  color: var(--color-text-primary);
  font-family: var(--font-family-base);
  font-size: var(--font-size-md);

  background: var(--color-background);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-md);
  outline: none;

  transition:
    border-color 150ms ease,
    box-shadow 150ms ease;

  &::placeholder {
    color: var(--color-text-muted);
  }

  &:focus {
    border-color: var(--color-border-focus);
    box-shadow: 0 0 0 3px rgb(37 99 235 / 12%);
  }

  &:disabled {
    background: var(--color-background-disabled);
    cursor: not-allowed;
  }

  &[aria-invalid='true'] {
    border-color: var(--color-error);
  }
}
```

### `shared/ui/label/ui/label.module.scss`
```scss
.label {
  color: var(--color-text-primary);
  font-family: var(--font-family-base);
  font-size: var(--font-size-md);
  font-weight: var(--font-weight-semibold);
}
```

---

## 6. Работа с типографикой (Font)

Не стоит создавать отдельный React-компонент для каждого размера текста. Достаточно базовых стилей, токенов типографики и, при необходимости, универсальных компонентов `Heading` и `Text`.

### `shared/styles/_typography.scss`
```scss
body {
  font-family: var(--font-family-base);
  font-size: var(--font-size-md);
  font-weight: var(--font-weight-regular);
  line-height: var(--line-height-normal);
  color: var(--color-text-primary);
}
```

Для централизованного управления визуальной иерархией в будущем можно добавить:
```text
shared/ui/
└── typography/
    ├── ui/
    │   ├── heading.tsx
    │   ├── text.tsx
    │   └── typography.module.scss
    └── index.ts
```

---

## 7. Поддержка Light / Dark Theme

Использование CSS Custom Properties позволяет переключать темы без переписывания стилей самих компонентов.

### `shared/styles/themes/_light.scss`
```scss
[data-theme='light'] {
  --color-text-primary: #171717;
  --color-text-secondary: #737373;
  --color-background: #ffffff;
  --color-border: #d5d5d0;
}
```

### `shared/styles/themes/_dark.scss`
```scss
[data-theme='dark'] {
  --color-text-primary: #f5f5f5;
  --color-text-secondary: #a3a3a3;
  --color-background: #171717;
  --color-border: #404040;
}
```

---

## 8. Результат: рефакторинг Feature-модулей

### Пример 1: `src/features/auth-by-email`

**До:**
```scss
// login-form.module.scss
.passwordLabel {
  display: flex;
  align-items: center;
  justify-content: space-between;

  a {
    color: #737373;
    font-size: 14px;
    text-decoration: underline;

    &:hover {
      color: #2563eb;
    }
  }
}
```

**После:**
```scss
// login-form.module.scss
.passwordLabel {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
```
```tsx
<div className={styles.passwordLabel}>
  <Label htmlFor="password">
    Password
  </Label>

  <Link href="/forgot-password">
    Forgot password?
  </Link>
</div>
```

### Пример 2: `src/features/request-password-reset`

**До:**
```scss
// forgot-password-form.module.scss
.backLink {
  display: flex;
  align-items: center;
  justify-content: center;

  a {
    color: #737373;
    font-size: 14px;
    text-decoration: underline;

    &:hover {
      color: #2563eb;
    }
  }
}
```

**После:**
```scss
// forgot-password-form.module.scss
.backLink {
  display: flex;
  align-items: center;
  justify-content: center;
}
```
```tsx
<div className={styles.backLink}>
  <Link href="/login">
    Back to login
  </Link>
</div>
```

---

## 9. Итоговая архитектура

```text
[ Design Tokens (Colors · Typography · Spacing · Radius) ]
                           │
                           ▼
        [ shared/ui (Button, Link, Input, Label) ]
                           │
             ┌─────────────┴─────────────┐
             ▼                           ▼
     [ auth-by-email ]       [ request-password-reset ]
       (Login Form)            (Forgot Password Form)
```

| Уровень | Зона ответственности |
| :--- | :--- |
| **`shared/styles/tokens`** | Единые значения цветов, размеров, шрифтов и радиусов |
| **`shared/styles/themes`** | Переопределение токенов для Light / Dark тем |
| **`shared/ui`** | Переиспользуемые UI-компоненты без бизнес-логики |
| **`features`** | Бизнес-логика и композиция UI-компонентов |
| **`app/styles`** | Глобальные стили приложения и сброс (reset) |

---

## 10. Архитектурные правила для production-ready `emarket-web`

1. **Запретить hardcoded colors** внутри Feature SCSS Modules — использовать только Design Tokens.
2. Все стандартные `Button`, `Link`, `Input`, `Label` импортировать исключительно из `shared/ui`.
3. **Не дублировать** общие CSS-стили между различными Feature.
4. **Не использовать глобальные стили** для индивидуального оформления конкретных Feature.
5. Для визуальных вариаций UI использовать пропсы `variant` и `size` в общих компонентах.
6. Для темизации использовать только **CSS Custom Properties**.
7. Для нестандартных элементов разрешать локальные стили в модулях фичи, если они действительно специфичны только для неё.
8. **Не создавать отдельный компонент** для каждого незначительного CSS-элемента (соблюдать баланс).

> **Важный нюанс:** `shared/ui` не должен превращаться в свалку всех мыслимых компонентов. Добавляйте туда только действительно переиспользуемые и независимые от бизнес-логики элементы.

### Резюме выбора
**Стек:** Design Tokens + Shared UI Components + CSS Modules + CSS Custom Properties.  
Это обеспечит единый визуальный язык приложения, упростит поддержку и позволит менять дизайн всего `emarket-web` централизованно.

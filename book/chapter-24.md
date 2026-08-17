# Глава 24. Создание собственной UI-библиотеки

> К моменту завершения книги читатель уже познакомился практически со всеми ключевыми возможностями современного CSS. Теперь задача состоит не в изучении отдельных свойств, а в объединении их в единую компонентную архитектуру.

Современная UI-библиотека представляет собой не набор готовых кнопок и карточек, а систему независимых компонентов, построенных на единых принципах проектирования. Каждый компонент должен обладать предсказуемым API, минимальным количеством внешних зависимостей, встроенной адаптивностью и поддержкой доступности.

Именно эти принципы позволяют создавать библиотеки компонентов, пригодные как для небольших проектов, так и для масштабных дизайн-систем.

---

## 24.1 Принципы проектирования компонентов

### Четыре обязательных свойства

Любой компонент современной библиотеки должен обладать несколькими обязательными свойствами.

**1. Независимость**

Компонент не должен зависеть от окружающей страницы.

```css
/* ❌ Плохо — зависит от контекста */
.card .title { ... }
.page .button { ... }

/* ✅ Хорошо — независим */
.button { ... }
.title { ... }
```

Вместо этого он использует:

```text
Container Queries
  ↓
Адаптация к контейнеру, а не к странице
  ↓
Собственные токены
  ↓
--component-color: var(--color-primary)
  ↓
Локальные состояния
  ↓
:hover, :focus-visible, data-state
```

**Такой компонент можно свободно переносить между проектами без изменения CSS.**

**2. Минимальный публичный API**

Компонент должен предоставлять наружу только то, что действительно необходимо.

```text
CSS Custom Properties
  ↓
--button-color: var(--color-primary)
  ↓
data-атрибуты
  ↓
[data-variant="primary"]
  ↓
Состояния
  ↓
:hover, :focus-visible, [data-state]
  ↓
CSS Parts (для Web Components)
  ↓
::part(button)
  ↓
Документированные слоты
  ↓
<slot>Действие</slot>
```

**Чем меньше внешний API, тем проще поддерживать библиотеку.**

**3. Отделение структуры от оформления**

```text
HTML
  ↓
Определяет структуру и семантику
  ↓
CSS
  ↓
Определяет внешний вид
  ↓
JavaScript
  ↓
Управляет поведением
```

**Современный CSS позволяет значительно сократить объём логики, необходимой для реализации компонента.**

**4. Доступность по умолчанию**

```text
Семантический HTML
  ↓
Правильные ARIA-атрибуты
  ↓
Клавиатурная навигация
  ↓
Фокус
  ↓
Контрастность
  ↓
prefers-reduced-motion
```

---

## 24.2 Универсальная архитектура компонента

### Схема компонента

Практически любой компонент современной библиотеки можно представить одинаковой схемой.

```text
Component
  ↓
├── Tokens (Custom Properties)
├── States (:hover, :focus, :disabled)
├── Layout (Grid, Flexbox)
├── Variants (data-* атрибуты)
├── Accessibility (ARIA, focus-visible)
└── Animations (transitions, @keyframes)
```

**Каждый уровень отвечает только за одну задачу.**

### Пример: универсальный компонент Button

```css
@layer components {
  .button {
    /* === 1. Design Tokens === */
    --button-bg: var(--color-primary);
    --button-color: var(--color-text-inverse);
    --button-padding: var(--space-md) var(--space-lg);
    --button-radius: var(--radius-md);
    --button-gap: var(--space-xs);
    
    /* === 2. Базовая структура (Layout) === */
    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: var(--button-gap);
    padding: var(--button-padding);
    background: var(--button-bg);
    color: var(--button-color);
    border: 1px solid transparent;
    border-radius: var(--button-radius);
    font-family: inherit;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.2s ease;
    
    /* === 3. Состояния (States) === */
    &:hover {
      background: color-mix(in oklch, var(--button-bg), white 15%);
      transform: translateY(-1px);
    }
    
    &:active {
      transform: scale(0.98);
    }
    
    &:focus-visible {
      outline: 2px solid var(--color-primary);
      outline-offset: 2px;
    }
    
    &:disabled {
      opacity: 0.5;
      cursor: not-allowed;
      pointer-events: none;
    }
    
    /* === 4. Варианты (Variants) === */
    &[data-variant="primary"] {
      --button-bg: var(--color-primary);
    }
    
    &[data-variant="danger"] {
      --button-bg: var(--color-danger);
    }
    
    &[data-variant="outline"] {
      background: transparent;
      border-color: var(--color-primary);
      color: var(--color-primary);
      
      &:hover {
        background: var(--color-primary);
        color: white;
      }
    }
    
    /* === 5. Размеры === */
    &[data-size="sm"] {
      --button-padding: var(--space-sm) var(--space-md);
      font-size: var(--font-size-sm);
    }
    
    &[data-size="lg"] {
      --button-padding: var(--space-lg) var(--space-xl);
      font-size: var(--font-size-lg);
    }
    
    /* === 6. Состояние загрузки === */
    &[data-state="loading"] {
      opacity: 0.7;
      cursor: wait;
      pointer-events: none;
    }
    
    /* === 7. Адаптивность (Container Query) === */
    @container (max-width: 400px) {
      width: 100%;
      justify-content: center;
    }
    
    /* === 8. Анимация === */
    &[data-state="loading"] .spinner {
      animation: spin 0.8s linear infinite;
    }
    
    /* === 9. Доступность (prefers-reduced-motion) === */
    @media (prefers-reduced-motion: reduce) {
      transition: none;
      
      &:hover {
        transform: none;
      }
    }
  }
  
  @keyframes spin {
    from { transform: rotate(0deg); }
    to { transform: rotate(360deg); }
  }
}
```

---

## 24.3 Базовый набор компонентов

### Фундаментальные компоненты

Практически любая дизайн-система начинается с небольшого числа фундаментальных компонентов.

```text
Формы
  ↓
Button, Input, Select, Checkbox, Radio, Switch
  ↓
Контейнеры
  ↓
Card, Dialog, Popover, Menu
  ↓
Навигация
  ↓
Tabs, Accordion, Breadcrumbs
  ↓
Обратная связь
  ↓
Tooltip, Toast, Progress
```

**Именно они становятся строительными блоками остальных интерфейсов.**

### Карточка (Card)

```css
@layer components {
  .card {
    /* Tokens */
    --card-padding: var(--space-md);
    --card-radius: var(--radius-lg);
    --card-bg: var(--color-surface);
    --card-border: var(--color-border);
    
    /* Layout */
    display: grid;
    gap: var(--space-sm);
    padding: var(--card-padding);
    background: var(--card-bg);
    border: 1px solid var(--card-border);
    border-radius: var(--card-radius);
    
    /* Состояния */
    &:hover {
      box-shadow: var(--shadow-md);
    }
    
    /* Адаптивность */
    container-type: inline-size;
    container-name: card;
    
    @container card (max-width: 400px) {
      padding: var(--space-sm);
      
      & .title {
        font-size: var(--font-size-md);
      }
    }
    
    /* Варианты */
    &[data-elevated] {
      box-shadow: var(--shadow-md);
      border-color: transparent;
    }
    
    &[data-interactive] {
      cursor: pointer;
      
      &:hover {
        transform: translateY(-2px);
        box-shadow: var(--shadow-lg);
      }
    }
  }
}
```

### Диалог (Dialog)

> **Важное уточнение:** нативный элемент `<dialog>`, открытый методом `showModal()`, уже поставляется с собственным оверлеем — псевдоэлементом `::backdrop`, который рендерится браузером в top layer позади диалога и автоматически перекрывает всё остальное содержимое страницы, включая элементы с `position: fixed` и высоким `z-index`. Стилизовать этот оверлей вручную через `inset: 0` и `background` на самом `.dialog` — не ошибка сама по себе, но избыточное дублирование того, что браузер уже делает бесплатно, и это лишает интерфейс части встроенных гарантий (например, `::backdrop` всегда выше любых `position: fixed`-элементов независимо от `z-index`, а самодельный оверлей на `.dialog` подчиняется обычным правилам стекового контекста). Правильный современный подход — стилизовать именно `::backdrop`:

```css
@layer components {
  .dialog {
    /* Tokens */
    --dialog-padding: var(--space-lg);
    --dialog-radius: var(--radius-lg);
    --dialog-bg: var(--color-surface);
    
    /* Стилизация самого <dialog> — только контент, без ручного оверлея */
    max-width: 500px;
    width: 100%;
    padding: var(--dialog-padding);
    background: var(--dialog-bg);
    border: none;
    border-radius: var(--dialog-radius);
    box-shadow: var(--shadow-xl);
    
    /* Нативный оверлей — рендерится браузером в top layer автоматически,
       появляется только при открытии через showModal() */
    &::backdrop {
      background: color-mix(in oklch, var(--color-text), transparent 50%);
    }
    
    &[open] {
      animation: fade-in 0.3s ease;
    }
    
    /* Анимация */
    @keyframes fade-in {
      from {
        opacity: 0;
        transform: scale(0.95);
      }
      to {
        opacity: 1;
        transform: scale(1);
      }
    }
    
    /* Доступность */
    @media (prefers-reduced-motion: reduce) {
      &[open] {
        animation: none;
      }
    }
  }
}
```

```javascript
// Открытие именно через showModal() — только тогда появляется ::backdrop,
// работает захват фокуса и закрытие по Esc.
// dialog.show() открывает немодальный диалог без оверлея и без этих гарантий.
document.querySelector('.dialog').showModal();
```

Такой вариант даёт то же самое визуально, но опирается на встроенные гарантии платформы — автоматический top layer, захват фокуса, закрытие по Esc — вместо их ручного воссоздания, что полностью соответствует принципу Progressive Enhancement, о котором книга говорит в главах 1 и 4.

### Единые принципы для всех компонентов

Каждый компонент должен использовать:

```text
Одинаковые токены
  ↓
--color-primary, --space-md, --radius-md
  ↓
Одинаковые состояния
  ↓
:hover, :focus-visible, :disabled
  ↓
Одинаковую систему вариантов
  ↓
[data-variant], [data-size], [data-state]
  ↓
Единые правила доступности
  ↓
:focus-visible, prefers-reduced-motion
```

---

## 24.4 Адаптивность компонентов

### От адаптивности страницы к адаптивности компонента

Современная библиотека практически полностью отказывается от идеи "адаптивности страницы". Теперь адаптируется сам компонент.

```text
Media Queries
  ↓
Адаптация к окну браузера
  ↓
Проблема: компонент не знает свой контекст
  ↓
Container Queries
  ↓
Адаптация к контейнеру
  ↓
Компонент знает свой контекст
```

### Инструменты адаптивности компонента

```text
Container Queries
  ↓
Адаптация к размеру контейнера
  ↓
Subgrid
  ↓
Наследование структуры сетки
  ↓
Логические свойства
  ↓
margin-inline, padding-block
  ↓
Относительные единицы
  ↓
cqi, cqb, clamp(), min(), max()
```

### Пример: адаптивный компонент

```css
@layer components {
  .product-card {
    container-type: inline-size;
    container-name: product;
    
    display: grid;
    gap: var(--space-md);
    
    /* Разные layout'ы в зависимости от контейнера */
    @container product (max-width: 400px) {
      grid-template-columns: 1fr;
      
      & .image {
        aspect-ratio: 16/9;
      }
      
      & .title {
        font-size: var(--font-size-md);
      }
    }
    
    @container product (min-width: 401px) and (max-width: 700px) {
      grid-template-columns: 1fr 2fr;
      
      & .image {
        aspect-ratio: 1/1;
      }
    }
    
    @container product (min-width: 701px) {
      grid-template-columns: 1fr 1fr;
      
      & .image {
        aspect-ratio: 4/3;
      }
    }
  }
}
```

### Результат

```text
Один и тот же компонент
  ↓
Работает в карточке (узкий контейнер)
  ↓
Работает в боковой панели (средний)
  ↓
Работает в основном контенте (широкий)
  ↓
Работает внутри модального окна (произвольный)
  ↓
Без дополнительных медиазапросов
```

---

## 24.5 Компоненты как система

### Правила взаимодействия

По-настоящему масштабируемая библиотека строится не вокруг отдельных элементов, а вокруг правил их взаимодействия.

```text
Cascade Layers
  ↓
reset, tokens, base, components, utilities, overrides
  ↓
Design Tokens
  ↓
Единый источник истины
  ↓
Container Queries
  ↓
Автономность компонентов
  ↓
@scope
  ↓
Локализация влияния правил
  ↓
Custom Properties
  ↓
Публичный API компонентов
  ↓
View Transitions
  ↓
Плавность взаимодействия
```

### Полная архитектура библиотеки

```css
/* 1. Reset — нормализация браузерных стилей */
@layer reset {
  * { margin: 0; padding: 0; box-sizing: border-box; }
}

/* 2. Tokens — дизайн-токены */
@layer tokens {
  :root {
    --color-primary: oklch(65% 0.22 255);
    --space-md: 1rem;
    --radius-md: 8px;
  }
}

/* 3. Base — базовые стили HTML-элементов */
@layer base {
  body { font-family: system-ui; }
  h1 { font-size: 2rem; }
}

/* 4. Components — UI-компоненты */
@layer components {
  .button { ... }
  .card { ... }
  .dialog { ... }
}

/* 5. Utilities — атомарные классы */
@layer utilities {
  .flex { display: flex; }
  .hidden { display: none; }
}

/* 6. Overrides — исключения */
@layer overrides {
  .legacy-fix { ... }
}
```

---

## 24.6 Практический чек-лист

### Перед публикацией компонента

Перед публикацией компонента полезно задать несколько вопросов.

**Дизайн-токены:**
- [ ] Использует ли компонент дизайн-токены?
- [ ] Нет ли магических значений (16px, #2563eb)?

**Независимость:**
- [ ] Независим ли он от размеров окна браузера?
- [ ] Работает ли он внутри любого контейнера?

**Доступность:**
- [ ] Поддерживает ли клавиатурную навигацию?
- [ ] Есть ли правильные ARIA-атрибуты?
- [ ] Есть ли поддержка `prefers-reduced-motion`?
- [ ] Достаточный ли контраст?

**API:**
- [ ] Минимально ли количество публичных CSS API?
- [ ] Документированы ли Custom Properties?
- [ ] Документированы ли data-атрибуты?

**Структура:**
- [ ] Нет ли жёсткой привязки к конкретной HTML-структуре?
- [ ] Используются ли семантические элементы?

**Современность:**
- [ ] Используются ли современные возможности CSS вместо JavaScript там, где это возможно?
- [ ] Используются ли Container Queries для адаптивности?
- [ ] Используются ли `:has()` и другие современные селекторы?

### Если на все вопросы можно ответить положительно

```text
Компонент с высокой вероятностью
  ↓
Окажется пригодным для повторного использования
  ↓
Будет работать в разных проектах
  ↓
Будет легко поддерживать
  ↓
Будет доступным
  ↓
Будет производительным
```

---

## 24.7 Итоги главы

1. **Четыре принципа компонентов:** независимость, минимальный API, отделение структуры, доступность по умолчанию

2. **Универсальная архитектура:** Tokens → States → Layout → Variants → Accessibility → Animations

3. **Базовый набор компонентов:** Button, Input, Card, Dialog, Popover, Menu, Tabs, Accordion

4. **Адаптивность компонентов:** Container Queries вместо Media Queries

5. **Компоненты как система:** @layer, Tokens, @scope, Container Queries, Custom Properties

6. **Практический чек-лист:** токены, независимость, доступность, API, структура, современность

---

**Главная мысль:** Создание современной UI-библиотеки — это прежде всего проектирование системы, а не отдельных компонентов. Современный CSS предоставляет достаточный набор возможностей, чтобы строить автономные, адаптивные и доступные компоненты без чрезмерной зависимости от JavaScript или сложных соглашений по именованию классов.

Все технологии, рассмотренные в книге, работают наиболее эффективно именно совместно: дизайн-токены обеспечивают единый язык оформления, каскадные слои делают архитектуру предсказуемой, контейнерные запросы освобождают компоненты от зависимости от размеров окна, а современные селекторы и псевдоклассы позволяют описывать сложное поведение декларативно. Именно такая комбинация возможностей определяет подход к разработке UI-библиотек в экосистеме Modern CSS 2026.


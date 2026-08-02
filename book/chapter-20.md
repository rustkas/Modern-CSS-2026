# Глава 20. Modern CSS Architecture: построение дизайн-систем средствами платформы

> За последние несколько лет CSS перестал быть языком оформления страниц и превратился в полноценную платформу проектирования пользовательских интерфейсов. Современная дизайн-система больше не строится вокруг препроцессоров, методологий именования классов или тяжёлых JavaScript-фреймворков. Большинство фундаментальных задач теперь решается средствами самой платформы: каскадными слоями, пользовательскими свойствами, контейнерными запросами, современными цветовыми моделями и компонентной архитектурой.

Эта глава завершает книгу, объединяя рассмотренные ранее возможности в единую архитектурную модель. Её задача — показать не отдельные свойства CSS, а то, как они взаимодействуют между собой при построении масштабируемых интерфейсов.

---

## 20.1 Современная архитектура CSS

### Независимые уровни ответственности

Архитектура современного CSS строится вокруг нескольких независимых уровней ответственности. Каждый из них отвечает за собственную задачу.

| Уровень | Инструменты | Задача |
|---------|-------------|--------|
| **Дизайн** | Design Tokens, OKLCH, `color-mix()` | Единый язык интерфейса |
| **Каскад** | `@layer`, `@scope` | Управление приоритетами |
| **Компоненты** | Custom Properties, Shadow DOM, `:has()` | Изоляция и API |
| **Адаптивность** | Container Queries, Grid, Flexbox | Реакция на контекст |
| **Динамика** | Scroll-driven Animations, View Transitions | Поведение интерфейса |
| **Производительность** | `content-visibility`, `contain` | Оптимизация рендеринга |

**Такое разделение позволяет каждому механизму выполнять собственную функцию, не вмешиваясь в остальные.**

```text
Дизайн → что
  ↓
Каскад → где и с каким приоритетом
  ↓
Компоненты → как изолированно
  ↓
Адаптивность → когда и как меняется
  ↓
Динамика → как движется
  ↓
Производительность → как быстро
```

---

## 20.2 Design Tokens как фундамент системы

### От конкретных значений к абстракциям

Практически все современные дизайн-системы строятся вокруг токенов. Токены представляют собой абстракцию над конкретными значениями.

```css
:root {
  /* Цвета */
  --color-primary: oklch(65% 0.22 255);
  --color-surface: light-dark(white, #1a1a1a);
  --color-text: light-dark(#1a1a1a, white);
  
  /* Пространство */
  --space-xs: 0.25rem;
  --space-sm: 0.5rem;
  --space-md: 1rem;
  --space-lg: 1.5rem;
  --space-xl: 2rem;
  
  /* Радиусы */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-xl: 16px;
  
  /* Типографика */
  --font-family: system-ui, -apple-system, sans-serif;
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.125rem;
  --font-size-xl: 1.25rem;
  --font-size-2xl: 1.5rem;
}
```

### Почему компоненты используют токены

Компоненты никогда не используют конкретные значения напрямую. Вместо этого они обращаются исключительно к токенам.

```css
/* ❌ Плохо — конкретное значение */
.button {
  background: #2563eb;
  padding: 16px;
  border-radius: 8px;
}

/* ✅ Хорошо — токен */
.button {
  background: var(--color-primary);
  padding: var(--space-md);
  border-radius: var(--radius-md);
}
```

**Это позволяет:**

- менять тему приложения;
- автоматически синхронизироваться с Figma;
- централизованно изменять дизайн;
- использовать алгоритмические палитры.

---

## 20.3 Современный компонент

### Структура компонента

Современный компонент представляет собой небольшой самостоятельный модуль. Он имеет несколько уровней API.

```text
Публичный API (Custom Properties)
  ↓
:root { --button-color: blue; }
  ↓
Состояния (псевдоклассы)
  ↓
:hover, :focus-visible, [data-state]
  ↓
Адаптивность (Container Queries)
  ↓
@container (max-width: 400px)
  ↓
Изоляция (@scope, @layer)
  ↓
@scope (.button) { ... }
  ↓
Анимация (View Transitions, Scroll Timeline)
  ↓
view-transition-name: button
```

### Полный пример современного компонента

```css
@layer components {
  @scope (.button) {
    /* 1. Базовые стили через токены */
    .button {
      --button-bg: var(--color-primary);
      --button-color: var(--color-text-inverse);
      --button-padding: var(--space-md) var(--space-lg);
      --button-radius: var(--radius-md);
      
      display: inline-flex;
      align-items: center;
      justify-content: center;
      gap: var(--space-xs);
      padding: var(--button-padding);
      background: var(--button-bg);
      color: var(--button-color);
      border: 1px solid transparent;
      border-radius: var(--button-radius);
      font-family: inherit;
      font-weight: 500;
      cursor: pointer;
      transition: all 0.2s ease;
      
      /* 2. Состояния */
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
      
      &[data-state="loading"] {
        opacity: 0.7;
        cursor: wait;
      }
      
      &[data-state="disabled"] {
        opacity: 0.5;
        cursor: not-allowed;
        pointer-events: none;
      }
      
      /* 3. Варианты через data-атрибуты */
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
      
      /* 4. Размеры */
      &[data-size="sm"] {
        --button-padding: var(--space-sm) var(--space-md);
        font-size: var(--font-size-sm);
      }
      
      &[data-size="lg"] {
        --button-padding: var(--space-lg) var(--space-xl);
        font-size: var(--font-size-lg);
      }
      
      /* 5. Адаптивность через Container Queries */
      @container (max-width: 400px) {
        width: 100%;
        justify-content: center;
      }
      
      /* 6. Анимация через Scroll Timeline */
      animation: button-reveal linear;
      animation-timeline: view();
      animation-range: entry 0% entry 50%;
    }
    
    @keyframes button-reveal {
      from {
        opacity: 0;
        transform: translateY(20px);
      }
      to {
        opacity: 1;
        transform: translateY(0);
      }
    }
  }
}
```

---

## 20.4 Архитектура дизайн-системы

### Уровни системы

Современная дизайн-система обычно состоит из нескольких уровней.

```text
Reset
  ↓
Браузерная нормализация
  ↓
Tokens
  ↓
Дизайн-токены (цвета, размеры, типографика)
  ↓
Foundations
  ↓
Базовые стили элементов (h1, p, a)
  ↓
Layout
  ↓
Структурные шаблоны (Stack, Grid, Sidebar)
  ↓
Components
  ↓
UI-компоненты (Button, Card, Dialog)
  ↓
Patterns
  ↓
Сочетания компонентов (Page, Section)
  ↓
Utilities
  ↓
Атомарные классы (flex, hidden, text-center)
  ↓
Application
  ↓
Специфические стили приложения
```

### Зависимости между уровнями

```text
Application
  ↓
использует
  ↓
Utilities
  ↓
использует
  ↓
Patterns
  ↓
использует
  ↓
Components
  ↓
использует
  ↓
Layout
  ↓
использует
  ↓
Foundations
  ↓
использует
  ↓
Tokens
  ↓
использует
  ↓
Reset
```

**Каждый уровень зависит только от предыдущего. Это делает систему предсказуемой и легко расширяемой.**

---

## 20.5 Взаимодействие современных возможностей CSS

### Единая экосистема

Главная идея Modern CSS состоит не в отдельных новых свойствах, а в их совместной работе.

Например, карточка может одновременно использовать:

```text
OKLCH
  ↓
Перцептивный цвет
  ↓
color-mix()
  ↓
Вычисление состояний
  ↓
Design Tokens
  ↓
Единый язык дизайна
  ↓
Container Queries
  ↓
Адаптация к контейнеру
  ↓
@scope
  ↓
Локальный каскад
  ↓
@layer
  ↓
Архитектурный приоритет
  ↓
View Transition
  ↓
Плавная смена состояний
  ↓
Scroll Timeline
  ↓
Анимация при скролле
  ↓
Anchor Positioning
  ↓
Позиционирование оверлеев
```

### Полный пример

```css
@layer reset, tokens, base, components, utilities;

@layer tokens {
  :root {
    --color-primary: oklch(65% 0.22 255);
    --color-surface: light-dark(white, #1a1a1a);
    --space-md: 1rem;
    --radius-md: 8px;
  }
}

@layer components {
  @scope (.product-card) {
    .product-card {
      container-type: inline-size;
      container-name: product;
      display: grid;
      gap: var(--space-md);
      padding: var(--space-md);
      background: var(--color-surface);
      border-radius: var(--radius-md);
      
      /* OKLCH + color-mix() */
      --product-accent: var(--color-primary);
      
      &:hover {
        border-color: color-mix(in oklch, var(--product-accent), white 20%);
      }
      
      /* View Transition */
      view-transition-name: product-{id};
      
      /* Scroll Timeline */
      animation: reveal linear;
      animation-timeline: view();
      animation-range: entry 0% entry 50%;
      
      /* Container Query */
      @container product (max-width: 400px) {
        grid-template-columns: 1fr;
        gap: var(--space-sm);
      }
      
      @container product (min-width: 401px) {
        grid-template-columns: 1fr 2fr;
      }
      
      /* :has() логика */
      &:has(.badge-sale) {
        border-color: var(--color-danger);
      }
      
      &:has(.badge-new) {
        border-color: var(--color-success);
      }
    }
    
    @keyframes reveal {
      from {
        opacity: 0;
        transform: translateY(30px) scale(0.95);
      }
      to {
        opacity: 1;
        transform: translateY(0) scale(1);
      }
    }
  }
}

@layer utilities {
  .product-card-compact {
    --space-md: 0.5rem;
  }
}
```

**Каждая технология отвечает лишь за собственную область ответственности.**

```text
OKLCH → цвет
  ↓
color-mix() → вычисление состояний
  ↓
Design Tokens → единые значения
  ↓
Container Queries → адаптивность
  ↓
@scope → локальный каскад
  ↓
@layer → архитектурный приоритет
  ↓
View Transition → смена состояний
  ↓
Scroll Timeline → анимация при скролле
  ↓
:has() → логика на основе структуры
```

**Именно такое разделение делает архитектуру устойчивой.**

---

## 20.6 Прогрессивное улучшение как принцип архитектуры

### Три уровня поддержки

Одной из важнейших особенностей современного CSS остаётся прогрессивное улучшение.

При проектировании компонентов удобно разделить возможности платформы на три группы.

**1. Базовый уровень — используется без ограничений**

```text
Grid
  ↓
Flexbox
  ↓
Subgrid
  ↓
Container Queries
  ↓
Cascade Layers
  ↓
CSS Nesting
  ↓
:has()
  ↓
OKLCH
  ↓
color-mix()
```

**2. Расширенный уровень — используется при наличии поддержки**

```text
Style Queries
  ↓
:open (popover)
  ↓
contrast-color()
  ↓
Anchor Positioning
  ↓
View Transitions
  ↓
Scroll-driven Animations
```

**3. Экспериментальный уровень — подключается через @supports**

```text
corner-shape
  ↓
Gap Decorations
  ↓
animation-trigger
  ↓
Masonry Layout
  ↓
Cross-document View Transitions
```

### Стратегия внедрения

```css
/* 1. Базовый уровень — всегда работает */
.card {
  display: grid;
  gap: var(--space-md);
  background: var(--color-surface);
  border-radius: var(--radius-md);
}

/* 2. Расширенный уровень — проверка поддержки */
@supports (container-type: inline-size) {
  .card-container {
    container-type: inline-size;
  }
}

@supports (view-transition: 1) {
  .card {
    view-transition-name: card;
  }
}

/* 3. Экспериментальный уровень — только через @supports */
@supports (grid-template-rows: masonry) {
  .card-grid {
    grid-template-rows: masonry;
  }
}
```

**Такой подход позволяет развивать дизайн-систему без риска для существующих пользователей.**

---

## 20.7 Архитектура Modern CSS 2026

### Единая система платформы

Если посмотреть на книгу целиком, можно увидеть, что современные возможности CSS складываются в единую систему.

```text
Design Tokens
  ↓
Единый язык интерфейса
  ↓
Modern Colors (OKLCH, color-mix)
  ↓
Перцептивные цвета, вычисляемые палитры
  ↓
Cascade Layers (@layer)
  ↓
Архитектурное управление приоритетами
  ↓
@scope
  ↓
Локальный каскад, изоляция компонентов
  ↓
Container Queries
  ↓
Адаптация к контейнеру, а не к вьюпорту
  ↓
Components (Custom Properties, :has())
  ↓
Самодостаточные компоненты с публичным API
  ↓
Motion (Scroll-driven, View Transitions)
  ↓
Декларативные анимации без JavaScript
  ↓
Производительность (content-visibility, contain)
  ↓
Оптимизация рендеринга
```

**Это уже не набор независимых спецификаций. Это единая архитектура платформы.**

### Связь между главами

```text
Часть I. Новый CSS
  ↓
Каскад, слои, компонентное мышление
  ↓
Часть II. Новый Layout
  ↓
Grid, Flexbox, Container Queries, Anchor Positioning
  ↓
Часть III. Современные селекторы
  ↓
:has(), :is(), :where(), CSS Nesting
  ↓
Часть IV. Цвет как система
  ↓
OKLCH, color-mix(), алгоритмические палитры
  ↓
Часть V. Анимация без JavaScript
  ↓
View Transitions, Scroll-driven Animations
  ↓
Часть VI. Архитектура CSS
  ↓
@layer, изоляция, дизайн-системы
  ↓
Часть VII. Производительность
  ↓
Rendering Pipeline, content-visibility, contain
  ↓
Часть VIII. Практика
  ↓
Создание UI-библиотеки, дизайн-системы, полный проект
```

---

## 20.8 Итоги главы

1. **Современная архитектура CSS** — независимые уровни: дизайн, каскад, компоненты, адаптивность, динамика, производительность

2. **Design Tokens** — фундамент системы: абстракции над конкретными значениями

3. **Современный компонент** — публичный API, состояния, адаптивность, изоляция, анимация

4. **Архитектура дизайн-системы** — Reset → Tokens → Foundations → Layout → Components → Patterns → Utilities → Application

5. **Взаимодействие технологий** — каждая технология отвечает за свою область ответственности

6. **Прогрессивное улучшение** — базовый, расширенный и экспериментальный уровни

7. **Единая архитектура** — все возможности CSS складываются в единую систему

---

**Главная мысль:** Современный CSS превратился в полноценную платформу разработки пользовательских интерфейсов. Большинство архитектурных задач, которые ещё несколько лет назад требовали препроцессоров, CSS-in-JS, сложных соглашений об именовании или большого количества JavaScript, теперь решаются средствами самой веб-платформы. Cascade Layers управляют каскадом, `@scope` ограничивает область действия правил, Container Queries делают компоненты независимыми от размеров окна, Design Tokens обеспечивают единый источник истины, а современные цветовые пространства и анимации позволяют строить адаптивные интерфейсы без вспомогательных библиотек.

Эволюция CSS — это переход от языка описания внешнего вида к декларативной системе проектирования интерфейсов. Именно эта идея лежит в основе Modern CSS 2026: использовать возможности платформы как единое целое, выбирая зрелые стандарты в качестве основы архитектуры и внедряя новые спецификации по мере их готовности через прогрессивное улучшение.

# Глава 26. Полностью современный проект: архитектура Modern CSS 2026

> После знакомства с современными возможностями CSS закономерно возникает практический вопрос: **как объединить все эти технологии в единую архитектуру реального проекта?**

За последние годы CSS перестал быть просто языком оформления страниц. Современный проект строится вокруг каскадных слоёв, дизайн-токенов, компонентной архитектуры и нативных возможностей браузера. Большая часть задач, которые ещё недавно решались при помощи Sass, Bootstrap, jQuery-плагинов или большого количества JavaScript, сегодня реализуется средствами самой платформы.

Эта глава показывает, как выглядит типичный проект уровня **Modern CSS 2026** — от структуры каталогов до принципов выбора технологий.

---

## 26.1 Архитектура проекта

### Структура каталогов

Современный CSS-проект строится вокруг нескольких независимых слоёв ответственности.

Типичная структура может выглядеть следующим образом:

```
styles/
├── reset.css          # Браузерная нормализация
├── tokens.css         # Дизайн-токены
├── themes.css         # Цветовые схемы
├── base.css           # Базовые HTML-элементы
├── layout.css         # Структурные шаблоны
├── components/        # UI-компоненты
│   ├── button.css
│   ├── card.css
│   ├── dialog.css
│   └── ...
├── utilities.css      # Атомарные классы
└── overrides.css      # Исключения
```

### Архитектурный фундамент

```text
@layer
  ↓
Управление приоритетами
  ↓
Design Tokens
  ↓
Единый язык дизайна
  ↓
Custom Properties
  ↓
Динамические значения
  ↓
CUBE CSS
  ↓
Composition, Utility, Block, Exception
  ↓
Component-first подход
  ↓
Автономные компоненты
```

**Именно они определяют организацию проекта, а не выбранный фреймворк.**

### Полная структура слоёв

```css
@layer reset, tokens, themes, base, layout, components, utilities, overrides;

/* 1. Reset — нормализация браузерных стилей */
@layer reset { ... }

/* 2. Tokens — дизайн-токены */
@layer tokens { ... }

/* 3. Themes — цветовые схемы */
@layer themes { ... }

/* 4. Base — базовые HTML-элементы */
@layer base { ... }

/* 5. Layout — структурные шаблоны */
@layer layout { ... }

/* 6. Components — UI-компоненты */
@layer components { ... }

/* 7. Utilities — атомарные классы */
@layer utilities { ... }

/* 8. Overrides — исключения */
@layer overrides { ... }
```

---

## 26.2 Каскад как архитектурный механизм

### От случайности к управляемости

В современных проектах каскад перестаёт быть источником случайностей и становится управляемой системой.

```text
Раньше:
  ↓
Специфичность → войны селекторов
  ↓
!important → цепная реакция
  ↓
Сейчас:
  ↓
@layer → архитектурные уровни
  ↓
@scope → локальный каскад
  ↓
:where() → нулевая специфичность
```

### Практическая реализация

Практически любая крупная кодовая база строится вокруг заранее объявленных каскадных слоёв:

```css
@layer reset, tokens, themes, base, layout, components, utilities, overrides;

@layer tokens {
  :root {
    --color-primary: oklch(65% 0.22 255);
    --space-md: 1rem;
  }
}

@layer components {
  .button {
    background: var(--color-primary);
    padding: var(--space-md);
  }
}

@layer utilities {
  .button-danger {
    background: var(--color-danger);
  }
}

@layer overrides {
  .legacy-button {
    background: #e74c3c;
  }
}
```

**Такой подход практически устраняет необходимость постоянно повышать специфичность селекторов и значительно упрощает сопровождение больших проектов.**

### Глобальные решения

Здесь же располагаются глобальные дизайн-токены, цветовые схемы и базовая типографика (см. главы 3, 4, 18).

```css
@layer tokens {
  :root {
    /* Цветовая система */
    --color-primary: oklch(65% 0.22 255);
    --color-surface: light-dark(white, #1a1a1a);
    --color-text: light-dark(#1a1a1a, white);
    
    /* Пространство */
    --space-xs: 0.25rem;
    --space-sm: 0.5rem;
    --space-md: 1rem;
    --space-lg: 1.5rem;
    --space-xl: 2rem;
    
    /* Типографика */
    --font-family: system-ui, -apple-system, sans-serif;
    --font-size-base: 1rem;
    --font-size-lg: 1.125rem;
    --font-size-xl: 1.25rem;
  }
}
```

---

## 26.3 Компоненты как единица архитектуры

### Принципы компонентов

Основной единицей современной разработки становится компонент.

Каждый компонент должен быть максимально автономным:

```text
Использовать собственный API
  ↓
Custom Properties, data-атрибуты
  ↓
Не зависеть от размеров окна браузера
  ↓
Container Queries вместо Media Queries
  ↓
Иметь минимальное количество внешних зависимостей
  ↓
Только токены и базовые стили
  ↓
Переиспользоваться в различных контекстах
  ↓
Работать в любом контейнере
```

### Инструменты изоляции

Для этого используются:

```text
Container Queries
  ↓
Адаптация к контейнеру
  ↓
@scope
  ↓
Локальный каскад
  ↓
CSS Modules / Shadow DOM
  ↓
Изоляция имён / полная инкапсуляция
  ↓
Design Tokens
  ↓
Единый язык дизайна
  ↓
data-атрибуты
  ↓
Публичный API компонентов
```

### Пример: современный компонент

```css
@layer components {
  @scope (.product-card) {
    .product-card {
      /* Tokens */
      --card-padding: var(--space-md);
      --card-radius: var(--radius-lg);
      
      /* Layout */
      container-type: inline-size;
      container-name: product;
      display: grid;
      gap: var(--space-md);
      padding: var(--card-padding);
      background: var(--color-surface);
      border-radius: var(--card-radius);
      
      /* Структура */
      & .image {
        aspect-ratio: 16/9;
        object-fit: cover;
      }
      
      & .title {
        font-size: var(--font-size-xl);
        font-weight: 700;
      }
      
      /* Состояния */
      &:hover {
        box-shadow: var(--shadow-lg);
      }
      
      /* Варианты */
      &[data-featured] {
        border: 2px solid var(--color-primary);
      }
      
      /* Адаптивность к контейнеру */
      @container product (max-width: 400px) {
        grid-template-columns: 1fr;
        --card-padding: var(--space-sm);
        
        & .title {
          font-size: var(--font-size-md);
        }
      }
      
      @container product (min-width: 401px) {
        grid-template-columns: 1fr 2fr;
      }
      
      /* Логика через :has() */
      &:has(.badge-sale) {
        border-color: var(--color-danger);
      }
      
      /* Анимация при скролле */
      animation: reveal linear;
      animation-timeline: view();
      animation-range: entry 0% entry 50%;
    }
  }
}
```

**Современный компонент адаптируется к своему контейнеру, а не к устройству пользователя** (см. главы 7, 19, 24).

---

## 26.4 Современная цветовая система

### Математическая модель цвета

Цветовая архитектура проекта строится вокруг математических моделей.

Вместо хранения десятков заранее подготовленных оттенков используются:

```text
OKLCH
  ↓
Перцептивное цветовое пространство
  ↓
Relative Color Syntax
  ↓
Изменение отдельных каналов
  ↓
color-mix()
  ↓
Смешивание цветов
  ↓
light-dark()
  ↓
Автоматическая адаптация к теме
```

### Практическая реализация

```css
@layer tokens {
  :root {
    /* Базовые цвета в OKLCH */
    --color-primary: oklch(65% 0.22 255);
    --color-secondary: oklch(55% 0.20 120);
    --color-danger: oklch(55% 0.25 30);
    --color-success: oklch(60% 0.20 150);
    
    /* Вычисляемые состояния */
    --color-primary-hover: color-mix(in oklch, var(--color-primary), white 15%);
    --color-primary-active: color-mix(in oklch, var(--color-primary), black 15%);
    --color-primary-subtle: color-mix(in oklch, var(--color-primary), transparent 85%);
    
    /* Адаптивные поверхности */
    --color-surface: light-dark(
      oklch(98% 0.01 100),
      oklch(18% 0.01 100)
    );
    --color-text: light-dark(
      oklch(20% 0.01 100),
      oklch(90% 0.01 100)
    );
  }
}
```

**Это позволяет получать производные цвета непосредственно в CSS, значительно сокращая объём дизайн-токенов и облегчая сопровождение палитры.**

### Учёт поддержки браузеров

При использовании новых функций (`contrast-color()` и Style Queries) следует учитывать их статус Baseline и предусматривать резервные варианты (см. главы 12–14).

```css
/* Проверка поддержки contrast-color() */
@supports (color: contrast-color(white)) {
  .text-on-accent {
    color: contrast-color(var(--color-primary));
  }
}

@supports not (color: contrast-color(white)) {
  .text-on-accent {
    color: white;
  }
}
```

---

## 26.5 Интерактивность без JavaScript

### Перемещение логики в CSS

Одной из главных тенденций последних лет стало постепенное перемещение интерактивной логики из JavaScript непосредственно в CSS.

Современные проекты используют:

```text
:has()
  ↓
Родительский селектор, логика на основе структуры
  ↓
:focus-visible
  ↓
Фокус без mouse-эффектов
  ↓
:user-valid / :user-invalid
  ↓
Валидация после взаимодействия
  ↓
:open
  ↓
Состояние открытых элементов (dialog, popover, details)
  ↓
View Transitions
  ↓
Плавные переходы между состояниями
  ↓
Scroll-driven Animations
  ↓
Анимация при скролле
  ↓
Anchor Positioning
  ↓
Позиционирование оверлеев
```

### Пример: полностью декларативный компонент

```html
<!-- Без JavaScript -->
<details class="accordion">
  <summary>Заголовок</summary>
  <div class="content">Содержимое</div>
</details>

<div class="card">
  <img src="image.jpg">
  <div class="card-content">
    <h3>Заголовок</h3>
    <p>Описание</p>
  </div>
</div>

<button popovertarget="menu" class="trigger">Меню</button>
<div id="menu" popover class="dropdown">...</div>
```

```css
/* Вся логика в CSS */
.accordion[open] .content {
  animation: slide-down 0.3s ease;
}

.card:has(img) {
  padding: 0;
}

/* Anchor Positioning требует пары объявлений:
   имя якоря на элементе-триггере... */
.trigger {
  anchor-name: --trigger;
}

/* ...и ссылку на это имя на позиционируемом элементе */
.dropdown {
  position: absolute;
  position-anchor: --trigger;
  top: anchor(bottom);
  left: anchor(left);
  position-try-fallbacks: flip-block;
}

/* Обязательный сброс: браузер по умолчанию
   центрирует popover-элементы через margin: auto,
   что конфликтует с anchor-позиционированием (см. главу 8) */
[popover] {
  margin: unset;
}
```

> **Напоминание:** этот фрагмент — сокращённая иллюстрация архитектурного принципа «логика в CSS, а не в JavaScript», а не самостоятельный production-ready сниппет. Полный разбор Anchor Positioning с обработкой крайних случаев (Gotchas: `@position-try`, `popover="hint"`, повторяющиеся компоненты и `anchor-scope`) — в главе 8, раздел 8.5.

**Во многих сценариях это позволяет отказаться от вспомогательных скриптов, оставляя JavaScript только для бизнес-логики приложения** (см. главы 8–17).

### Статус технологий

При этом важно различать зрелые и экспериментальные технологии:

```text
✅ Same-document View Transitions → Baseline
  ↓
Можно использовать везде
  ↓
⚠️ Cross-document View Transitions → Новые
  ↓
Проверять поддержку
  ↓
⚠️ Anchor Positioning → Недавно
  ↓
Проверять поддержку
  ↓
⚠️ animation-trigger → Экспериментально
  ↓
Только через @supports
```

---

## 26.6 Производительность как часть архитектуры

### Встроенная оптимизация

Производительность современного проекта закладывается на этапе проектирования.

Практически каждая архитектурная рекомендация книги одновременно является рекомендацией по производительности:

```text
contain
  ↓
Изоляция области пересчёта
  ↓
content-visibility
  ↓
Пропуск рендеринга вне viewport
  ↓
Ограничение области Layout
  ↓
Анимация через transform и opacity
  ↓
Только Composite
  ↓
Минимизация дорогостоящих Paint-операций
  ↓
Избегать blur, box-shadow при анимации
  ↓
Профилирование через DevTools
  ↓
Измерение вместо предположений
```

### Практическая реализация

```css
/* Изоляция компонента */
.card {
  contain: content;
}

/* Длинные списки */
.long-list-item {
  content-visibility: auto;
  contain-intrinsic-size: 200px;
}

/* Производительная анимация */
.element {
  will-change: transform; /* Только перед анимацией */
  transition: transform 0.3s ease;
}

.element:hover {
  transform: scale(1.1); /* Только Composite */
}
```

**Оптимизация перестаёт быть отдельным этапом разработки и становится частью архитектуры компонентов** (см. главы 21–23).

---

## 26.7 Modern CSS без лишних зависимостей

### От фреймворков к платформе

Современный проект всё реже требует тяжёлых CSS-фреймворков.

Большинство возможностей, ради которых раньше использовали Bootstrap или Sass, сегодня входят непосредственно в стандарт CSS:

| Раньше | Сегодня |
|--------|---------|
| Sass variables | CSS Custom Properties |
| Sass Nesting | CSS Nesting |
| Bootstrap Grid | CSS Grid + Subgrid |
| JS Popper | Anchor Positioning |
| JS Scroll libraries | Scroll-driven Animations |
| CSS-in-JS | Design Tokens + Cascade Layers + Native CSS |
| BEM для изоляции | `@scope`, CSS Modules, Shadow DOM |

### Что остаётся

```text
Сторонние инструменты
  ↓
Становятся дополнением к платформе
  ↓
А не её обязательной частью
  ↓
Lightning CSS → обработка
  ↓
PostCSS → совместимость
  ↓
Stylelint → качество
  ↓
Storybook → документация
```

**Это не означает, что сторонние инструменты исчезают, но теперь они становятся дополнением к платформе, а не её обязательной частью.**

### Сравнение подходов

```text
Legacy CSS (2015-2020)
  ↓
Sass → компиляция
  ↓
Bootstrap → готовая сетка
  ↓
jQuery-плагины → интерактивность
  ↓
CSS-in-JS → изоляция
  ↓
Modern CSS (2026)
  ↓
Нативный CSS → без компиляции
  ↓
CSS Grid → нативная сетка
  ↓
CSS → нативная интерактивность
  ↓
@scope + @layer → нативная изоляция
```

---

## 26.8 Итоги главы

### Что мы построили

Современный CSS-проект 2026 года выглядит так:

```text
1. Архитектура
  ↓
@layer, Design Tokens, компоненты
  ↓
2. Каскад
  ↓
Управляемый, предсказуемый, без !important
  ↓
3. Компоненты
  ↓
Автономные, адаптивные, с чётким API
  ↓
4. Цвет
  ↓
OKLCH, вычисляемые палитры, алгоритмические
  ↓
5. Интерактивность
  ↓
Декларативная, без JavaScript где возможно
  ↓
6. Производительность
  ↓
Встроенная в архитектуру
  ↓
7. Зависимости
  ↓
Минимальные, только инструменты
```

### Ключевые принципы

```text
Используйте нативные возможности платформы
  ↓
Проектируйте каскад как архитектуру
  ↓
Стройте компоненты вокруг контейнера
  ↓
Вычисляйте цвета, а не храните их
  ↓
Перемещайте логику в CSS
  ↓
Закладывайте производительность в архитектуру
  ↓
Минимизируйте зависимости
```

### Главный вывод

Современный CSS перестал быть языком отдельных правил оформления и превратился в полноценную инженерную платформу. Каскадные слои, дизайн-токены, контейнерные запросы, математические цветовые модели, новые селекторы и встроенные механизмы производительности позволяют строить масштабируемые интерфейсы, опираясь преимущественно на возможности самой веб-платформы.

**Главный вывод книги** заключается не в том, что следует использовать каждую новую возможность сразу после её появления, а в умении осознанно выбирать технологии в зависимости от их зрелости и задач проекта. Именно сочетание нативных стандартов, архитектурной дисциплины и понимания статуса Baseline делает Modern CSS 2026 практическим инженерным подходом, а не просто набором новых возможностей языка.

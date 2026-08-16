# Глава 11. CSS Nesting: современная организация стилей

> Нативная вложенность (**CSS Nesting**) стала одной из последних возможностей, окончательно устранивших необходимость использовать препроцессоры исключительно ради удобного синтаксиса. Однако её значение значительно шире простой замены Sass.

В современной архитектуре CSS вложенность является способом организации компонентов, локализации состояний и объединения связанных правил в единый модуль. Вместе с `@layer`, `@scope`, Container Queries и CSS Custom Properties она формирует новый стиль написания CSS, в котором структура таблицы стилей повторяляет структуру интерфейса.

---

## 11.1 От препроцессоров к языку платформы *(уточнена дата и статус)*

### Эволюция вложенности

На протяжении почти двадцати лет вложенность ассоциировалась исключительно с Sass.

```text
Sass (2006)
  ↓
Препроцессорная вложенность
  ↓
Текстовая трансформация
  ↓
CSS Nesting (Newly available — август 2023, Widely available — июнь 2026)
  ↓
Нативная вложенность
  ↓
Анализ в CSSOM
```

> **Статус зрелости:** нативный CSS Nesting достиг Baseline Newly available в августе 2023 года, а в июне 2026 года — уже статуса Widely available (30-месячный период с момента межбраузерной поддержки истёк). Это означает, что вложенность сегодня можно использовать в продакшене без каких-либо оговорок и фолбэков, что и отражено во всех примерах этой главы.

### Как работает нативный Nesting

Sass выполнял лишь текстовую трансформацию исходного кода:

```scss
// Sass — текстовая трансформация
.card {
  .title { color: blue; }
}
// Компилируется в:
.card .title { color: blue; }
```

Нативный CSS Nesting работает иначе. Браузер анализирует структуру селекторов непосредственно в CSSOM, поэтому вложенность становится частью языка, а не этапом сборки проекта.

```css
/* CSS — анализ в CSSOM */
.card {
  & .title { color: blue; }
}
/* Браузер понимает вложенность на уровне движка */
```

### Важные следствия

```text
Препроцессорная вложенность
  ↓
Генерация длинных селекторов
  ↓
Увеличение специфичности
  ↓
Сложность переопределения
  ↓
Нативная вложенность
  ↓
Отсутствие генерации
  ↓
Браузер вычисляет специфичность
  ↓
Интеграция с каскадом
```

**Преимущества нативной вложенности:**

1. **Отсутствует генерация огромных списков селекторов** — браузер не создаёт длинные цепочки
2. **Браузер самостоятельно вычисляет специфичность** — через механизм `:is()`
3. **Вложенность интегрируется с каскадом** — работает с `@layer`, `@scope`, `@container`
4. **Правила участвуют в работе DevTools** — без промежуточной компиляции

Таким образом Nesting перестаёт быть синтаксическим сахаром и становится элементом архитектуры CSS.

---

## 11.2 Вложенность как структура компонента

### Объединение логики компонента

Главная задача Nesting — объединение всей логики компонента в одном месте.

Современный компонент обычно включает:

```text
Базовые стили
  ↓
Интерактивные состояния
  ↓
Варианты оформления
  ↓
Локальные media-запросы
  ↓
Container Queries
  ↓
Анимации
  ↓
Состояния доступности
```

### Пример: полный компонент

```css
.card {
  /* === Базовые стили === */
  display: grid;
  gap: var(--space-md);
  padding: var(--space-md);
  background: var(--color-surface);
  border-radius: var(--radius-lg);
  transition: all 0.3s ease;
  
  /* === Структура === */
  & .header {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  
  & .title {
    font-size: var(--font-size-xl);
    font-weight: var(--font-weight-bold);
  }
  
  & .content {
    margin-top: var(--space-sm);
    color: var(--color-text-secondary);
  }
  
  /* === Состояния === */
  &:hover {
    transform: translateY(-4px);
    box-shadow: var(--shadow-lg);
  }
  
  &:focus-visible {
    outline: 2px solid var(--color-primary);
    outline-offset: 2px;
  }
  
  /* === Варианты === */
  &[data-featured] {
    border-color: var(--color-primary);
    background: var(--color-featured-bg);
  }
  
  &[data-compact] {
    padding: var(--space-sm);
    gap: var(--space-xs);
    
    & .title {
      font-size: var(--font-size-md);
    }
  }
  
  /* === Адаптивность === */
  @container (width > 40rem) {
    grid-template-columns: 1fr auto;
    gap: var(--space-lg);
  }
  
  @media (max-width: 600px) {
    padding: var(--space-sm);
    
    & .title {
      font-size: var(--font-size-md);
    }
  }
  
  /* === Анимации === */
  & .icon {
    transition: transform 0.3s ease;
  }
  
  &:hover .icon {
    transform: rotate(90deg);
  }
}
```

### Результат

```text
Весь жизненный цикл компонента
  ↓
описан внутри одного логического блока
  ↓
CSS отражает структуру интерфейса
  ↓
Легко находить и изменять
  ↓
Не нужно искать по файлам
```

---

## 11.3 Nesting и современный каскад

### Интеграция с архитектурными инструментами

В современной архитектуре Nesting редко используется самостоятельно. Обычно он сочетается с другими возможностями CSS.

### Nesting + @layer

```css
@layer reset, tokens, base, components, utilities;

@layer components {
  .button {
    display: inline-flex;
    padding: 0.5rem 1rem;
    border: 1px solid transparent;
    border-radius: var(--radius-md);
    cursor: pointer;
    
    /* Вложенные состояния принадлежат тому же слою */
    &:hover {
      background: color-mix(in oklch, var(--color-primary), black 10%);
    }
    
    &:focus-visible {
      outline: 2px solid var(--color-primary);
      outline-offset: 2px;
    }
    
    &:disabled {
      opacity: 0.5;
      cursor: not-allowed;
    }
    
    /* Варианты внутри компонента */
    &[data-variant="primary"] {
      background: var(--color-primary);
      color: white;
    }
    
    &[data-variant="danger"] {
      background: var(--color-danger);
      color: white;
    }
    
    /* Размеры */
    &[data-size="lg"] {
      padding: 0.75rem 1.5rem;
      font-size: var(--font-size-lg);
    }
    
    &[data-size="sm"] {
      padding: 0.25rem 0.75rem;
      font-size: var(--font-size-sm);
    }
  }
}
```

Все состояния и варианты автоматически принадлежат тому же слою `components`.

### Nesting + @scope

```css
@scope (.dialog) {
  .button {
    padding: 0.5rem 1rem;
    
    &:hover {
      background: var(--color-primary);
    }
  }
  
  .close-button {
    position: absolute;
    top: var(--space-sm);
    right: var(--space-sm);
    
    &:hover {
      transform: rotate(90deg);
    }
  }
}
```

Вложенность делает область действия компонента максимально очевидной.

### Nesting + Container Queries

```css
.card {
  container-type: inline-size;
  container-name: card;
  
  padding: var(--space-md);
  
  @container card (width > 50rem) {
    /* Контекст компонента остаётся целостным */
    display: flex;
    gap: var(--space-lg);
    
    & .image {
      flex: 0 0 40%;
    }
    
    & .content {
      flex: 1;
    }
  }
}
```

### Nesting + Style Queries

```css
.card {
  @container style(--theme: dark) {
    background: var(--color-dark-surface);
    color: var(--color-dark-text);
    
    & .title {
      color: var(--color-dark-title);
    }
    
    &:hover {
      box-shadow: var(--shadow-dark-lg);
    }
  }
  
  @container style(--theme: light) {
    background: var(--color-light-surface);
    color: var(--color-light-text);
  }
}
```

### Nesting + @media

```css
.card {
  padding: var(--space-md);
  
  @media (prefers-color-scheme: dark) {
    background: var(--color-dark-surface);
    color: var(--color-dark-text);
  }
  
  @media (prefers-reduced-motion: reduce) {
    transition: none;
  }
  
  @media (hover: hover) {
    &:hover {
      transform: translateY(-2px);
    }
  }
}
```

---

## 11.4 Специфичность: почему Nesting работает через `:is()`

### Как вычисляется специфичность

Одна из самых необычных особенностей нативной вложенности состоит в том, что она вычисляет специфичность не так, как Sass.

Рабочая группа CSSWG сознательно построила Nesting на механизме `:is()`.

**Ключевое правило:** специфичность определяется наиболее специфичным вариантом родительского селектора.

### Пример

```css
#a,
.card {
  & span {
    color: red;
  }
}
```

Даже если совпадение произошло по `.card`, специфичность будет соответствовать наиболее сильному варианту (`#a`).

```text
Родительский селектор: #a (1-0-0) + .card (0-1-0)
  ↓
Наиболее специфичный: #a (1-0-0)
  ↓
Вложенный селектор: span (0-0-1)
  ↓
Итоговая специфичность: 1-0-1
```

### Почему так сделано

```text
Если бы Nesting работал как Sass:
  ↓
#a span { ... }
.card span { ... }
  ↓
Два селектора — экспоненциальный рост
  ↓
Если Nesting работает через :is():
  ↓
:is(#a, .card) span { ... }
  ↓
Один селектор — эффективно
```

Такое поведение может показаться неожиданным, однако именно оно позволило избежать экспоненциального роста количества селекторов при компиляции.

### Практическая рекомендация

> **Не объединяйте в одном списке селекторы с резко различающейся специфичностью.**

```css
/* ❌ Плохо — #id и .card в одном списке */
#header,
.card {
  & .title { ... }
}
/* Специфичность: 1-1-0 (из-за #header) */

/* ✅ Хорошо — сепаратные правила */
#header .title { ... }
.card .title { ... }
```

### Сравнение с Sass

```scss
// Sass — текстовая трансформация
.card {
  .title { color: blue; }
  &:hover { color: red; }
}
// Компилируется в:
.card .title { color: blue; }
.card:hover { color: red; }
```

```css
/* CSS — через :is() */
.card {
  & .title { color: blue; }
  &:hover { color: red; }
}
/* Разворачивается в: */
:is(.card) .title { color: blue; }
:is(.card):hover { color: red; }
```

---

## 11.5 Современные рекомендации по вложенности

### Принципы использования

После появления нативного Nesting лучшие практики существенно изменились.

**1. Используйте вложенность только внутри компонентов**

```css
/* ✅ Хорошо — вложенность внутри компонента */
.card {
  & .title { ... }
  & .content { ... }
}

/* ❌ Плохо — вложенность вне компонента */
.page {
  & .header { ... }
  & .content { ... }
}
```

**2. Ограничивайте глубину двумя-тремя уровнями**

```css
/* ✅ Хорошо — 2 уровня */
.card {
  & .header {
    & .title { ... }
  }
}

/* ❌ Плохо — 5+ уровней */
.card {
  & .header {
    & .content {
      & .item {
        & .link { ... }
      }
    }
  }
}
```

**3. Группируйте состояния рядом с базовыми стилями**

```css
/* ✅ Хорошо — всё в одном месте */
.button {
  /* Базовые стили */
  padding: 0.5rem 1rem;
  
  &:hover { ... }
  &:focus-visible { ... }
  &:disabled { ... }
  &[data-variant="primary"] { ... }
}

/* ❌ Плохо — состояния разбросаны по файлу */
.button { ... }
.button:hover { ... }
.button:focus { ... }
.button[data-variant="primary"] { ... }
```

### Практический пример

```css
/* ✅ Рекомендуемый подход */
@layer components {
  .button {
    /* 1. Базовые стили */
    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: var(--space-xs);
    padding: 0.5rem 1rem;
    border: 1px solid transparent;
    border-radius: var(--radius-md);
    font-family: inherit;
    font-weight: var(--font-weight-medium);
    cursor: pointer;
    transition: all 0.2s ease;
    
    /* 2. Состояния */
    &:hover {
      transform: translateY(-1px);
    }
    
    &:focus-visible {
      outline: 2px solid var(--color-primary);
      outline-offset: 2px;
    }
    
    &:active {
      transform: scale(0.98);
    }
    
    &:disabled {
      opacity: 0.5;
      cursor: not-allowed;
    }
    
    /* 3. Варианты */
    &[data-variant="primary"] {
      background: var(--color-primary);
      color: white;
      
      &:hover {
        background: color-mix(in oklch, var(--color-primary), black 10%);
      }
    }
    
    &[data-variant="danger"] {
      background: var(--color-danger);
      color: white;
      
      &:hover {
        background: color-mix(in oklch, var(--color-danger), black 10%);
      }
    }
    
    /* 4. Размеры */
    &[data-size="lg"] {
      padding: 0.75rem 1.5rem;
      font-size: var(--font-size-lg);
    }
    
    &[data-size="sm"] {
      padding: 0.25rem 0.75rem;
      font-size: var(--font-size-sm);
    }
    
    /* 5. Адаптивность */
    @container (max-width: 400px) {
      width: 100%;
    }
  }
}
```

---

## 11.6 Что больше не стоит делать

### Антипаттерны при переходе с Sass

Переход с Sass на нативный CSS требует отказаться от ряда старых привычек.

**1. Глубокая вложенность**

```css
/* ❌ Плохо — копирование DOM-структуры */
.page {
  .layout {
    .sidebar {
      .menu {
        li {
          a {
            color: blue;
          }
        }
      }
    }
  }
}

/* ✅ Хорошо — плоская структура */
.page .menu-link {
  color: blue;
}
```

**2. Копирование HTML-структуры в CSS**

```css
/* ❌ Плохо — CSS повторяет HTML */
.card {
  .header {
    .title {
      .link {
        color: blue;
      }
    }
  }
}

/* ✅ Хорошо — CSS описывает компоненты */
.card-title-link {
  color: blue;
}
```

**3. Вложенность вместо компонентной архитектуры**

```css
/* ❌ Плохо — вложенность для управления специфичностью */
.component {
  .wrapper {
    .container {
      .item {
        .link {
          color: blue;
        }
      }
    }
  }
}

/* ✅ Хорошо — простые селекторы + @layer */
@layer components {
  .component-link {
    color: blue;
  }
}
```

**4. Sass-конкатенация (&__element)**

```scss
/* ❌ Sass-специфичная конкатенация */
.card {
  &__title { ... }
  &__content { ... }
}
/* Не поддерживается в нативном CSS */
```

```css
/* ✅ Нативный подход */
.card-title { ... }
.card-content { ... }
```

### Почему это больше не нужно

```text
Sass-подход:
  ↓
Генерация новых имён классов
  ↓
(&__title → .card__title)
  ↓
Нативный CSS:
  ↓
Описание отношений между существующими селекторами
  ↓
& .title → .card .title
```

Современный CSS ориентирован не на генерацию новых имён классов, а на описание отношений между существующими селекторами.

---

## 11.7 Nesting и будущее CSS

### Точка интеграции

Сегодня вложенность становится точкой интеграции практически всех новых возможностей платформы.

```text
Cascade Layers
  ↓
@scope
  ↓
Container Queries
  ↓
Style Queries
  ↓
Custom Properties
  ↓
View Transitions
  ↓
Anchor Positioning
  ↓
Анимации
  ↓
Логические селекторы
  ↓
Nesting
```

### Полный пример современного компонента

```css
@layer components {
  .product-card {
    /* Базовые стили */
    container-type: inline-size;
    container-name: product;
    display: grid;
    gap: var(--space-md);
    padding: var(--space-md);
    background: var(--color-surface);
    border-radius: var(--radius-lg);
    transition: all 0.3s ease;
    
    /* Структура */
    & .image {
      aspect-ratio: 16/9;
      object-fit: cover;
      border-radius: var(--radius-md);
      overflow: hidden;
    }
    
    & .title {
      font-size: var(--font-size-xl);
      font-weight: var(--font-weight-bold);
    }
    
    & .price {
      font-size: var(--font-size-lg);
      color: var(--color-primary);
    }
    
    /* Состояния через :has() */
    &:has(.badge-sale) {
      border-color: var(--color-danger);
    }
    
    &:has(.badge-new) {
      border-color: var(--color-success);
    }
    
    /* Интерактивные состояния */
    &:hover {
      transform: translateY(-4px);
      box-shadow: var(--shadow-lg);
    }
    
    /* Варианты */
    &[data-featured] {
      grid-column: span 2;
      padding: var(--space-lg);
      border-color: var(--color-primary);
    }
    
    /* Темизация через Style Queries */
    @container style(--theme: dark) {
      background: var(--color-dark-surface);
      color: var(--color-dark-text);
      
      & .title {
        color: var(--color-dark-title);
      }
    }
    
    /* Адаптивность через Container Queries */
    @container product (width > 40rem) {
      grid-template-columns: 1fr 2fr;
      
      & .image {
        height: 100%;
      }
    }
    
    @container product (width <= 30rem) {
      gap: var(--space-sm);
      padding: var(--space-sm);
      
      & .title {
        font-size: var(--font-size-md);
      }
    }
    
    /* Медиа-запросы */
    @media (prefers-color-scheme: dark) {
      background: var(--color-dark-bg);
      border-color: var(--color-dark-border);
    }
    
    @media (prefers-reduced-motion: reduce) {
      transition: none;
    }
    
    /* Анимации */
    & .image {
      transition: transform 0.5s ease;
    }
    
    &:hover .image {
      transform: scale(1.02);
    }
  }
}
```

---

## 11.8 Итоги главы

1. **Нативный Nesting — часть языка CSS**, а не этап сборки проекта

2. **Браузер анализирует вложенность в CSSOM** — без генерации длинных селекторов

3. **Nesting работает через `:is()`** — специфичность определяется наиболее специфичным вариантом

4. **Главная задача Nesting** — объединение всей логики компонента в одном месте

5. **Nesting интегрируется с `@layer`, `@scope`, `@container`** — все состояния принадлежат тому же контексту

6. **Ограничение глубины** — рекомендуется 2-3 уровня вложенности

7. **Группировка состояний** — рядом с базовыми стилями компонента

8. **Что больше не нужно:** глубокая вложенность, копирование DOM-структуры, Sass-конкатенация

9. **Nesting — точка интеграции** всех современных возможностей CSS

10. **Современные таблицы стилей** — описание компонентов, а не набор правил

---

**Главная мысль:** Нативный CSS Nesting завершил многолетнюю эволюцию языка CSS от плоских таблиц стилей к компонентной организации кода. В современной практике вложенность используется не для сокращения количества символов, а для локализации всей логики компонента — состояний, адаптивности, анимаций и взаимодействия с каскадом. В сочетании с `@layer`, `@scope` и контейнерными запросами Nesting становится одним из ключевых инструментов построения масштабируемой архитектуры **Modern CSS 2026**.


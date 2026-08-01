# Глава 2. Современный каскад

## Почему каскад стал архитектурной системой управления приложением

---

## 2.0 Почему каскад является сердцем CSS

На протяжении многих лет каскад воспринимался как одна из самых сложных и непредсказуемых частей CSS. Разработчики пытались «победить» его при помощи соглашений об именовании классов, увеличения специфичности селекторов и повсеместного использования `!important`.

Однако современный CSS предлагает иной подход. Вместо борьбы с каскадом разработчик получает инструменты архитектурного управления им. Появление `@layer`, `@scope`, типизированных пользовательских свойств (`@property`) и новых механизмов наследования превратило каскад из источника случайных конфликтов в основу масштабируемой системы стилизации.

**Почему понимание каскада важнее знания отдельных свойств:**

```text
DOM
  ↓
CSS Rules
  ↓
Cascade
  ↓
Computed Style
  ↓
Layout
  ↓
Paint
```

Практически все современные возможности CSS — Container Queries, View Transition, Anchor Positioning, `:has()`, `@scope`, Layers — в итоге проходят через каскад. Именно поэтому понимание каскада сегодня важнее знания отдельных CSS-свойств. Каскад — это операционная система CSS.

---

## 2.1 Алгоритм каскада: пять этапов

Современный каскад состоит из пяти последовательных этапов. Каждый этап фильтрует и уточняет правила, прежде чем передать управление следующему.

```text
Origin
  ↓
Layer
  ↓
Specificity
  ↓
Scope
  ↓
Order
```

### Почему порядок именно такой

Каждый этап решает свою архитектурную задачу:

```text
Origin
  ↓
защищает пользователя
  ↓
Layer
  ↓
управляет архитектурой проекта
  ↓
Specificity
  ↓
учитывает локальность правила
  ↓
Scope
  ↓
учитывает локальность компонента
  ↓
Order
  ↓
последний шанс разрешить конфликт
```

**1. Origin (Источник)**

Определяет, кто написал правило:

```text
User Agent (браузер)
  ↓
User (пользователь)
  ↓
Author (разработчик)
```

!important меняет этот порядок на противоположный, что делает его мощным, но опасным инструментом.

**2. Layer (Слой)**

Управляет архитектурой проекта. Разработчик может явно задать иерархию:

```css
@layer reset, tokens, base, components, utilities, overrides;
```

Правила из более поздних слоёв имеют приоритет над правилами из более ранних.

**3. Specificity (Специфичность)**

Учитывает вес селектора. Более специфичный селектор перевешивает менее специфичный:

```text
ID          → 1-0-0
Class       → 0-1-0
Element     → 0-0-1
```

Важно: !important и @layer влияют на специфичность.

**4. Scope (Область видимости)**

Учитывает, насколько локально правило определено. `@scope` позволяет ограничить действие правил определённой частью DOM.

**5. Order (Порядок)**

Последний этап — порядок объявления правил в коде. Если все предыдущие этапы не дали результата, побеждает последнее объявленное правило.

---

## 2.2 Эволюция каскада: от CSS2 к CSS 2026

История каскада — это история попыток управлять сложностью.

```text
CSS2
  ↓
Specificity (ID > Class > Element)
  ↓
!important (последнее средство)
  ↓
BEM (соглашение об именовании)
  ↓
CSS Modules (изоляция)
  ↓
@layer (архитектурное управление)
```

**Ключевой сдвиг в мышлении:** сегодня увеличение специфичности считается архитектурным запахом. Современный CSS рекомендует решать большинство конфликтов через Layers, а не через повышение веса селекторов.

### Влияние современных селекторов на специфичность

**`:is()`** — принимает вес самого специфичного селектора внутри:

```css
:is(#id, .class, element) {
  /* Вес: 1-0-0 (как у ID) */
}
```

**`:where()`** — всегда имеет вес 0-0-0 (революция!):

```css
:where(#id, .class, element) {
  /* Вес: 0-0-0 */
}
```

Это позволяет создавать базовые стили, которые легко переопределять.

**`:not()`** — принимает вес самого специфичного селектора внутри:

```css
:not(#id, .class) {
  /* Вес: 1-0-0 (как у ID) */
}
```

**`:has()`** — принимает вес самого специфичного селектора внутри:

```css
.card:has(.badge) {
  /* Вес: 0-2-0 (класс + класс) */
}
```

**Практическое применение `:where()`:**

```css
/* Базовый стиль — легко переопределить */
:where(.button) {
  background: #3498db;
  color: white;
  padding: 0.5rem 1rem;
}

/* Переопределение без повышения специфичности */
.button-primary {
  background: #2ecc71;
}
```

---

## 2.3 Custom Properties и @property

### Обычные CSS-переменные

```css
:root {
  --color: #3498db;
  --size: 16px;
  --duration: 300ms;
}
```

**Особенности:**
- Значения — строки
- Без проверки типов
- Без анимации
- Могут быть пустыми
- Наследуются по умолчанию

### Типизированные свойства (@property)

```css
@property --color {
  syntax: '<color>';
  inherits: true;
  initial-value: #3498db;
}

@property --size {
  syntax: '<length>';
  inherits: false;
  initial-value: 16px;
}

@property --duration {
  syntax: '<time>';
  inherits: false;
  initial-value: 300ms;
}
```

**Преимущества перед обычными переменными:**

| Без @property | С @property |
|---------------|-------------|
| строка | тип |
| нет проверки | есть проверка |
| нет анимации | есть анимация |
| нет initial-value | есть initial-value |
| нет inherits | есть inherits |

**Пример: анимация типизированного свойства**

```css
@property --rotation {
  syntax: '<angle>';
  inherits: false;
  initial-value: 0deg;
}

.element {
  --rotation: 0deg;
  transform: rotate(var(--rotation));
  transition: --rotation 2s ease;
}

.element:hover {
  --rotation: 360deg;
}
```

### CSS Typed OM

Раньше CSS работал со строками:

```javascript
// Старый подход — строки
element.style.setProperty('--size', '16px');
const size = element.style.getPropertyValue('--size');
// '16px'
```

Теперь можно работать с типизированными объектами:

```javascript
// Новый подход — Typed OM
element.attributeStyleMap.set('--size', CSS.px(16));
const size = element.attributeStyleMap.get('--size');
// CSSUnitValue { value: 16, unit: 'px' }
```

```text
CSS
  ↓
строки
  ↓
Typed OM
  ↓
объекты
```

**Преимущества:**
- Безопасность типов
- Автоматическая конвертация
- Лучшая производительность
- Интеграция с JavaScript

---

## 2.4 @scope и Shadow DOM

Современный CSS предлагает два подхода к изоляции стилей:

### @scope

```css
@scope (.card) {
  /* Стили применяются только внутри .card */
  .title {
    font-size: 1.5rem;
    color: #333;
  }

  /* Доступ к родителю через :scope */
  :scope {
    border: 1px solid #ddd;
    border-radius: 8px;
  }

  /* Пограничный случай: выход из области */
  .content :scope {
    /* :scope здесь — .card */
  }
}
```

**Преимущества:**
- Только CSS (не требует HTML)
- Легче, чем Shadow DOM
- Работает с существующим DOM
- Поддерживает вложенность

### Shadow DOM

```html
<my-component>
  #shadow-root
    <style>
      /* Полная изоляция */
      .title {
        font-size: 1.5rem;
        color: #333;
      }
    </style>
    <div class="title">Hello</div>
</my-component>
```

**Преимущества:**
- Полная инкапсуляция HTML + CSS
- Слоты для контента
- Истинная изоляция компонентов

### Сравнение подходов

| @scope | Shadow DOM |
|--------|------------|
| Только CSS | HTML + CSS |
| Нет нового DOM | Создаёт новый DOM |
| Легче | Тяжелее |
| Работает в любом DOM | Требует Custom Elements |
| Менее строгая изоляция | Полная изоляция |

**Когда что использовать:**

- `@scope` — для организации стилей в рамках компонента, для группировки правил
- Shadow DOM — для создания переиспользуемых веб-компонентов с полной изоляцией
- Комбинация — Shadow DOM для изоляции, `@scope` для организации внутри компонента

---

## 2.5 Современная модель наследования

CSS предоставляет пять ключевых слов для управления наследованием:

```text
inherit
  ↓
initial
  ↓
unset
  ↓
revert
  ↓
revert-layer
```

### inherit

Принудительно наследует значение от родителя:

```css
.element {
  color: inherit; /* Берёт цвет у родителя */
}
```

### initial

Возвращает начальное значение (определённое спецификацией):

```css
.element {
  display: initial; /* block для div, inline для span */
}
```

### unset

Сбрасывает к значению по умолчанию: если свойство наследуется — `inherit`, если нет — `initial`:

```css
.element {
  color: unset; /* inherit (color наследуется) */
  display: unset; /* initial (display не наследуется) */
}
```

### revert

Возвращает значение, которое было до применения стилей разработчика:

```css
.element {
  font-weight: revert; /* Возвращает стандартный вес шрифта */
}
```

### revert-layer

Возвращает значение, которое было до применения текущего слоя (`@layer`):

```css
@layer base {
  .button {
    background: #3498db;
    color: white;
  }
}

@layer override {
  .button {
    background: #e74c3c;
    color: revert-layer; /* Возвращает white (из base) */
  }
}
```

**Практическое применение:**

```css
@layer components {
  .card {
    /* Базовые стили карточки */
    padding: 1rem;
    background: white;
    border-radius: 8px;
  }
}

@layer overrides {
  .card-dark {
    /* Отменяем только фон, остальное оставляем */
    background: #1a1a1a;
    color: revert-layer; /* Возвращает цвет из components */
  }
}
```

---

## 2.6 Cascade Layers как архитектура приложения

`@layer` — самый важный архитектурный инструмент современного CSS.

### Базовая структура

```css
@layer reset, tokens, base, components, utilities, overrides;
```

### Полная архитектура приложения

```css
@layer reset {
  /* Сброс браузерных стилей */
  /* Normalize, Sanitize, или кастомный reset */
}

@layer tokens {
  /* Design Tokens */
  /* Цвета, размеры, отступы, радиусы, шрифты */
  :root {
    --color-primary: #3498db;
    --color-secondary: #2ecc71;
    --space-base: 1rem;
    --radius-base: 8px;
  }
}

@layer base {
  /* Базовые стили элементов */
  body {
    font-family: system-ui, sans-serif;
    line-height: 1.5;
    color: #333;
    background: #fff;
  }
  
  h1, h2, h3 {
    font-weight: 700;
    line-height: 1.2;
  }
}

@layer components {
  /* Стили компонентов */
  .button {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    padding: 0.5rem 1rem;
    border-radius: var(--radius-base);
    background: var(--color-primary);
    color: white;
    cursor: pointer;
    transition: all 0.2s ease;
  }
  
  .card {
    padding: var(--space-base);
    background: white;
    border-radius: var(--radius-base);
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  }
}

@layer utilities {
  /* Утилиты — переопределяют компоненты */
  .button-sm {
    padding: 0.25rem 0.75rem;
    font-size: 0.875rem;
  }
  
  .button-lg {
    padding: 0.75rem 1.5rem;
    font-size: 1.25rem;
  }
}

@layer overrides {
  /* Последнее слово — для исправлений */
  .button-danger {
    background: #e74c3c;
  }
}
```

### Визуализация архитектуры

```text
Reset
  ↓
Tokens
  ↓
Base
  ↓
Components
  ↓
Utilities
  ↓
Overrides
```

Каждый слой знает только о слоях выше. Это создаёт предсказуемую иерархию.

### Преимущества слоёв

1. **Предсказуемость** — всегда знаешь, что перевесит
2. **Масштабируемость** — легко добавлять новые слои
3. **Изоляция** — конфликты остаются внутри слоёв
4. **Отказ от !important** — слои делают его ненужным
5. **Документация** — слои документируют архитектуру

---

## 2.7 Современная архитектура Design System

Design System строится вокруг каскада:

```text
Tokens
  ↓
Theme
  ↓
Components
  ↓
Utilities
```

### Пример: система тем

```css
@layer tokens {
  :root {
    /* Базовые токены */
    --color-bg: #ffffff;
    --color-text: #1a1a1a;
    --color-primary: #3498db;
    --space-base: 1rem;
  }
}

@layer theme {
  /* Тёмная тема */
  @media (prefers-color-scheme: dark) {
    :root {
      --color-bg: #1a1a1a;
      --color-text: #f0f0f0;
    }
  }
  
  /* Контрастная тема */
  @media (prefers-contrast: high) {
    :root {
      --color-primary: #0055cc;
    }
  }
}

@layer components {
  .card {
    background: var(--color-bg);
    color: var(--color-text);
    padding: var(--space-base);
    border-radius: 8px;
  }
}
```

### Преимущества такого подхода

1. **Темы не ломают компоненты** — компоненты используют переменные
2. **Простое добавление тем** — достаточно обновить токены
3. **Семантические имена** — `--color-bg` вместо `--white`
4. **Гибкость** — кастомизация на любом уровне

---

## 2.8 Каскад и Web Components

Web Components добавляют новый уровень в каскад: Shadow DOM.

### Структура каскада в Web Components

```text
External Styles
  ↓
Shadow DOM Styles
  ↓
::part() / ::theme()
  ↓
Custom Properties
```

### Практический пример

```html
<my-button>
  #shadow-root
    <style>
      /* Изолированные стили */
      .button {
        background: var(--button-bg, #3498db);
        color: var(--button-text, white);
        padding: 0.5rem 1rem;
        border: none;
        border-radius: 8px;
        cursor: pointer;
      }
      
      /* Части для стилизации извне */
      .button::part(icon) {
        margin-right: 0.5rem;
      }
    </style>
    <button class="button" part="button">
      <span part="icon">★</span>
      <slot>Button</slot>
    </button>
</my-button>
```

```css
/* Внешние стили */
my-button {
  /* Переменные проходят в Shadow DOM */
  --button-bg: #e74c3c;
  --button-text: #ffffff;
}

/* Стилизация частей */
my-button::part(button) {
  border-radius: 4px;
  font-weight: bold;
}

my-button::part(icon) {
  color: gold;
}
```

### Почему переменные проходят через Shadow DOM

```text
:root
  ↓
Custom Properties (наследуются)
  ↓
Shadow DOM (получает наследование)
  ↓
Компонент
```

### Почему @layer работает с Web Components

```css
@layer base {
  my-button {
    --button-bg: #3498db;
  }
}

@layer components {
  my-button.primary {
    --button-bg: #2ecc71;
  }
}
```

Слои работают на уровне документа и применяются ко всем элементам, включая Web Components.

---

## 2.9 Best Practices

### Практические рекомендации

**1. Проектируйте архитектуру вокруг @layer, а не вокруг высокой специфичности**

```css
/* ❌ Плохо */
.card {
  background: white;
}
.card .header {
  background: #f0f0f0;
}
.card .header .title {
  font-size: 1.5rem;
}

/* ✅ Хорошо */
@layer components {
  .card {
    background: white;
  }
  .card-header {
    background: #f0f0f0;
  }
  .card-title {
    font-size: 1.5rem;
  }
}
```

**2. Используйте :where() для снижения специфичности базовых селекторов**

```css
/* Базовые стили с нулевой специфичностью */
:where(.button) {
  padding: 0.5rem 1rem;
  border-radius: 8px;
}

/* Легко переопределить */
.button-primary {
  background: #3498db;
  color: white;
}
```

**3. Рассматривайте @scope как средство локализации стилей, а не как замену Shadow DOM**

```css
@scope (.product-card) {
  /* Стили только внутри карточки товара */
  .title {
    font-size: 1.25rem;
  }
  .price {
    color: #2ecc71;
  }
}
```

**4. Регистрируйте часто используемые пользовательские свойства через @property**

```css
/* Для цветов */
@property --primary {
  syntax: '<color>';
  inherits: true;
  initial-value: #3498db;
}

/* Для размеров */
@property --space-base {
  syntax: '<length>';
  inherits: false;
  initial-value: 1rem;
}

/* Для анимаций */
@property --rotation {
  syntax: '<angle>';
  inherits: false;
  initial-value: 0deg;
}
```

**5. Избегайте !important**

```css
/* ❌ Плохо */
.button {
  background: #3498db !important;
}

/* ✅ Хорошо — используйте слои */
@layer components {
  .button {
    background: #3498db;
  }
}

@layer overrides {
  .button-danger {
    background: #e74c3c;
  }
}
```

**6. Применяйте revert-layer для точечного отката изменений**

```css
@layer base {
  .text {
    color: #333;
    font-size: 1rem;
    line-height: 1.5;
  }
}

@layer overrides {
  .text-highlight {
    color: #3498db;
    font-size: revert-layer; /* Возвращает размер из base */
    line-height: revert-layer; /* Возвращает высоту строки из base */
  }
}
```

**7. Стройте дизайн-системы вокруг каскада, а не вопреки ему**

```text
Tokens
  ↓
Theme
  ↓
Components
  ↓
Utilities
```

Используйте каскад как архитектурный инструмент, а не как помеху.

---

## 2.10 Итоги главы

1. **Каскад — сердце CSS:** все современные возможности проходят через него

2. **Пять этапов каскада:** Origin → Layer → Specificity → Scope → Order

3. **Каждый этап решает свою задачу:** от защиты пользователя до последнего шанса

4. **Специфичность больше не главное:** современный CSS предлагает `@layer` как архитектурный инструмент

5. **`:where()` — революция:** нулевая специфичность для базовых стилей

6. **`@property` добавляет типизацию:** проверка, анимация, наследование

7. **Typed OM — безопасная работа с CSS из JavaScript:** объекты вместо строк

8. **`@scope` vs Shadow DOM:** два подхода к изоляции с разными применениями

9. **Пять ключевых слов наследования:** `inherit`, `initial`, `unset`, `revert`, `revert-layer`

10. **`@layer` — архитектура приложения:** Reset → Tokens → Base → Components → Utilities → Overrides

11. **Design System строится на каскаде:** Tokens → Theme → Components → Utilities

12. **Каскад работает с Web Components:** переменные проходят через Shadow DOM

13. **Best Practices:** слои вместо специфичности, :where() для базовых стилей, отказ от !important

---

**Главная мысль:** Каскад больше не является историческим наследием CSS, с которым приходится мириться. В современном CSS он превратился в центральный механизм организации интерфейсов. Слои (`@layer`), области видимости (`@scope`), типизированные пользовательские свойства (`@property`) и новые правила наследования позволяют проектировать масштабируемые системы стилей без чрезмерной специфичности и без борьбы с `!important`.

Именно каскад связывает воедино практически все современные возможности платформы — от контейнерных запросов и пользовательских свойств до Web Components и дизайн-систем. Понимание современной модели каскада означает понимание архитектуры CSS в целом.


# Глава 10. Архитектура современных селекторов

> К моменту написания этой книги CSS-селекторы перестали быть лишь способом поиска элементов в DOM. Современные спецификации превратили их в полноценный язык декларативной логики, позволяющий описывать структуру интерфейса, управлять специфичностью, ограничивать область действия правил и строить компонентные системы практически без вспомогательных классов и JavaScript.

Если ранние поколения CSS были сосредоточены на том, **какой элемент выбрать**, то современный CSS отвечает на значительно более широкий круг вопросов:

```text
Где правило должно действовать?
  ↓
Насколько легко его можно переопределить?
  ↓
При каких условиях оно применяется?
  ↓
Как оно взаимодействует с другими правилами каскада?
```

Именно поэтому селекторы сегодня являются частью архитектуры CSS, а не просто синтаксисом поиска элементов.

---

## 10.1 Управление специфичностью вместо борьбы со специфичностью

### Историческая проблема

Одним из важнейших изменений последних лет стало переосмысление самой идеи специфичности.

Исторически разработчики пытались выигрывать «войны селекторов»:

```css
/* Типичный селектор из legacy-проекта */
#app .sidebar ul li a.button.active {
  color: blue;
  font-weight: bold;
  padding: 0.5rem 1rem;
}
```

Сегодня подобный код считается **архитектурным запахом**.

### Три современных механизма управления приоритетами

Современный CSS предлагает три независимых механизма управления приоритетами:

```text
@layer
  ↓
управляет приоритетом правил
  ↓
@scope
  ↓
управляет областью действия
  ↓
Логические псевдоклассы
  ↓
управляют специфичностью
```

### Новая философия

Благодаря этим механизмам селекторы вновь становятся максимально простыми:

```css
.button { ... }
.title { ... }
.card { ... }
.nav-link { ... }
```

**Сложность современной архитектуры переносится с селекторов на устройство каскада.**

```text
Старый подход:
  ↓
Сложные селекторы для управления приоритетами
  ↓
Новый подход:
  ↓
Простые селекторы
  ↓
Приоритеты управляются через @layer
  ↓
Специфичность управляется через :where()
  ↓
Область действия управляется через @scope
```

---

## 10.2 Логические селекторы как язык условий

### Эволюция селекторов

Selectors Level 4 ввёл семейство функциональных псевдоклассов, позволяющих описывать логические выражения непосредственно в CSS.

```text
CSS 2.1
  ↓
Простые селекторы (элемент, класс, ID)
  ↓
CSS 3
  ↓
Псевдоклассы (:nth-child, :not)
  ↓
Selectors Level 4
  ↓
Логические селекторы (:is, :where, :has)
  ↓
CSS 2026
  ↓
Логическая модель + архитектурные инструменты
```

### Три ключевых логических селектора

**`:is()` — логическое «ИЛИ»**

```css
/* Раньше — повторение */
article h1,
section h1,
aside h1 {
  font-size: 2rem;
}

/* Теперь — группировка */
:is(article, section, aside) h1 {
  font-size: 2rem;
}
```

**`:where()` — логическое «ИЛИ» с нулевой специфичностью**

```css
/* Базовые стили — легко переопределить */
:where(h1, h2, h3, h4) {
  text-wrap: balance;
  line-height: 1.2;
}

/* Любой класс переопределяет без повышения специфичности */
.title-large {
  font-size: 3rem;
}
```

**`:not()` — отрицание**

```css
/* Все кроме первого */
.card:not(:first-child) {
  margin-top: var(--space-md);
}

/* Все кроме последнего */
.card:not(:last-child) {
  border-bottom: 1px solid var(--border-color);
}
```

### Селекторы как логические выражения

```text
:is(article, section, aside) → OR
  ↓
:not(.disabled) → NOT
  ↓
.card:has(img) → EXISTS
  ↓
:where(h1, h2, h3) → OR с нулевой специфичностью
```

Эти селекторы превращают CSS в декларативный язык условий.

---

## 10.3 `:where()` как инструмент архитектуры

### Почему `:where()` важнее `:is()`

На практике наиболее важным селектором оказывается вовсе не `:is()`, а `:where()`. Его особенность — **нулевая специфичность**.

```text
:is(article, section, aside) → специфичность = вес самого тяжёлого селектора
  ↓
:where(article, section, aside) → специфичность = 0-0-0
```

### Применение `:where()` в архитектуре

**1. Reset-стили**

```css
/* Нулевая специфичность — легко переопределить */
:where(*, *::before, *::after) {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
```

**2. Базовые стили элементов**

```css
/* Базовые стили не мешают переопределению */
:where(h1, h2, h3, h4, h5, h6) {
  font-family: var(--font-heading);
  font-weight: var(--font-weight-bold);
  text-wrap: balance;
}

:where(p, li, blockquote) {
  text-wrap: pretty;
  max-width: 70ch;
}

:where(a) {
  color: var(--color-primary);
  text-decoration: underline;
}
```

**3. Дизайн-системы**

```css
/* Компоненты с нулевой специфичностью */
:where(.button) {
  display: inline-flex;
  padding: 0.5rem 1rem;
  border: 1px solid transparent;
  border-radius: var(--radius-md);
  cursor: pointer;
  transition: all 0.2s ease;
}

:where(.card) {
  padding: var(--space-md);
  background: var(--color-surface);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-sm);
}
```

**4. Библиотеки компонентов**

```css
/* Библиотека не создаёт проблем с переопределением */
:where(.ui-button) {
  /* Базовые стили */
}

:where(.ui-card) {
  /* Базовые стили */
}
```

### Почему современные CSS-фреймворки переходят на `:where()`

```text
Sass/SCSS
  ↓
Генерируют классы с высокой специфичностью
  ↓
Сложно переопределять
  ↓
Tailwind / Open Props
  ↓
Используют :where() для базовых стилей
  ↓
Легко переопределять
  ↓
Пользовательские стили всегда имеют приоритет
```

---

## 10.4 CSS Nesting и новая композиция селекторов *(добавлена секция про ловушку специфичности)*

...

### Важные особенности

**1. Специфичность не увеличивается — но только для простого родительского селектора**

```css
.card {
  & .title { /* специфичность: 0-2-0 (как .card .title) */ }
  &:hover { /* специфичность: 0-2-0 (как .card:hover) */ }
}
```

В отличие от Sass, где вложенность создаёт цепочки селекторов, нативный CSS Nesting разворачивается аналогично использованию `:is()`:

```css
.card {
  & .title {}
}
/* Разворачивается в */
:is(.card) .title {}
```

Когда родитель — одиночный простой селектор (как `.card` выше), этого можно почти не замечать: специфичность `:is(.card)` совпадает со специфичностью самого `.card`, и итоговый результат ничем не отличается от «наивного» разворачивания в `.card .title`.

> **Важная ловушка:** если родительский селектор — это **список через запятую**, а не одиночный класс, ситуация меняется. Браузер оборачивает весь список в `:is()`, а специфичность `:is()` всегда равна специфичности **самого тяжёлого** селектора внутри скобок — независимо от того, какая именно ветка списка реально совпала с элементом.
>
> ```css
> /* Родитель — список из класса и ID */
> .card, #hero {
>   & .title {
>     color: blue;
>   }
> }
>
> /* Разворачивается в */
> :is(.card, #hero) .title {
>   color: blue;
> }
> /* Специфичность — 1-0-1 (как у ID!), 
>    даже если реально совпал .card, а не #hero */
> ```
>
> Из-за этого правило, которое выглядит как обычный селектор класса, на деле получает специфичность на уровне ID и неожиданно перебивает более поздние «простые» переопределения — именно тот эффект, с которым `@layer` и `:where()` призваны бороться, но который сама вложенность может незаметно свести на нет. Проблема усиливается тем, что `:is()`-обёртки вкладываются друг в друга на каждом уровне вложенности, поэтому в глубоко вложенных блоках источник неожиданно высокой специфичности бывает трудно найти.
>
> **Практическое правило:** если родительский селектор — список через запятую (`.card, #hero`, `.a, .b.c`), либо не используйте вложенность на этом уровне, либо следите, чтобы все элементы списка имели одинаковую специфичность, либо оборачивайте базовые reset-подобные списки в `:where()` заранее, чтобы избежать эффекта `:is()` вообще.

**2. `&` всегда ссылается на родителя**

```css
.card {
  & .title { /* .card .title */ }
  &:hover { /* .card:hover */ }
  & > img { /* .card > img */ }
}
```
---

## 10.5 Селекторы как интерфейс компонентов

### Контракт между HTML и CSS

В компонентной архитектуре селекторы становятся **контрактом** между HTML и CSS. Компонент описывает своё состояние декларативно, а CSS реагирует на него.

### Современные подходы к API компонентов

**Состояния через data-атрибуты**

```html
<button class="button" data-state="loading">Загрузка...</button>
<button class="button" data-state="disabled">Отключено</button>
<button class="button" data-state="active">Активно</button>
```

```css
.button[data-state="loading"] {
  opacity: 0.7;
  cursor: wait;
}

.button[data-state="disabled"] {
  opacity: 0.5;
  cursor: not-allowed;
  pointer-events: none;
}

.button[data-state="active"] {
  background: var(--color-primary);
  color: white;
}
```

**Варианты через data-атрибуты**

```html
<button class="button" data-variant="primary">Основная</button>
<button class="button" data-variant="danger">Опасная</button>
<button class="button" data-variant="outlined">Обводная</button>
```

```css
.button[data-variant="primary"] {
  background: var(--color-primary);
  color: white;
}

.button[data-variant="danger"] {
  background: var(--color-danger);
  color: white;
}

.button[data-variant="outlined"] {
  background: transparent;
  border: 2px solid var(--color-primary);
  color: var(--color-primary);
}
```

**Размеры через data-атрибуты**

```html
<button class="button" data-size="sm">Маленькая</button>
<button class="button" data-size="lg">Большая</button>
```

```css
.button[data-size="sm"] {
  padding: 0.25rem 0.75rem;
  font-size: var(--font-size-sm);
}

.button[data-size="lg"] {
  padding: 0.75rem 1.5rem;
  font-size: var(--font-size-lg);
}
```

**Темы через data-атрибуты**

```html
<div class="card" data-theme="dark">
  <!-- Карточка в тёмной теме -->
</div>
```

```css
.card[data-theme="dark"] {
  --card-bg: var(--color-dark);
  --card-text: var(--color-light);
  --card-border: var(--color-dark-border);
}

.card[data-theme="light"] {
  --card-bg: var(--color-light);
  --card-text: var(--color-dark);
  --card-border: var(--color-light-border);
}
```

### Преимущества подхода

```text
Множество классов
  ↓
.button--primary
.button--danger
.button--outlined
.button--small
.button--large
.button--loading
.button--disabled
  ↓
Сложно поддерживать
  ↓
data-атрибуты
  ↓
[data-variant="primary"]
[data-variant="danger"]
[data-size="sm"]
[data-state="loading"]
  ↓
Просто и предсказуемо
```

**Преимущества:**

1. **Единый источник истины** — состояние компонента в одном месте
2. **Легко читать** — семантические имена
3. **Легко расширять** — добавлять новые варианты
4. **Совместимость с фреймворками** — React, Vue, Angular, Svelte
5. **Accessibility** — ARIA-атрибуты можно комбинировать

### Комбинация с `:has()`

```css
/* Компонент сам определяет свои состояния */
.card:has(.button[data-state="loading"]) {
  opacity: 0.5;
}

.card:has([data-featured]) {
  border: 2px solid var(--color-primary);
}

.card:has([data-theme="dark"]) {
  background: var(--color-dark);
  color: var(--color-light);
}
```

---

## 10.6 Интеграция с современной архитектурой

### Селекторы + @layer

```css
@layer reset, tokens, base, components, utilities, overrides;

@layer base {
  /* Базовые стили с :where() — нулевая специфичность */
  :where(h1, h2, h3) {
    font-family: var(--font-heading);
    text-wrap: balance;
  }
}

@layer components {
  /* Компоненты с простыми селекторами */
  .button {
    padding: 0.5rem 1rem;
    border-radius: var(--radius-md);
  }
  
  .button[data-variant="primary"] {
    background: var(--color-primary);
    color: white;
  }
}

@layer overrides {
  /* Переопределение с простым селектором */
  .button {
    border: 2px solid var(--color-primary);
  }
}
```

### Селекторы + @scope

```css
@scope (.card) {
  /* Стили только внутри .card */
  .title {
    font-size: var(--font-size-xl);
  }
  
  .content {
    padding: var(--space-md);
  }
  
  /* Вложенность внутри @scope */
  &:hover {
    transform: translateY(-2px);
  }
}
```

### Селекторы + Container Queries

```css
.card-container {
  container-type: inline-size;
  container-name: card;
}

@container card (max-width: 400px) {
  .card {
    padding: var(--space-sm);
  }
  
  .card .title {
    font-size: var(--font-size-md);
  }
}
```

### Полная архитектура компонента

```css
@layer components {
  .card {
    /* Базовые стили */
    container-type: inline-size;
    container-name: card;
    padding: var(--space-md);
    background: var(--color-surface);
    border-radius: var(--radius-lg);
    
    /* Структура */
    & .header {
      margin-bottom: var(--space-sm);
    }
    
    & .title {
      font-size: var(--font-size-xl);
      font-weight: var(--font-weight-bold);
    }
    
    & .content {
      margin-top: var(--space-sm);
    }
    
    /* Состояния */
    &[data-theme="dark"] {
      --color-surface: var(--color-dark-surface);
      --color-text: var(--color-dark-text);
    }
    
    &:hover {
      transform: translateY(-2px);
      box-shadow: var(--shadow-lg);
    }
    
    /* Вложенность с логическими селекторами */
    &:has(img) {
      padding: 0;
      overflow: hidden;
    }
    
    &:has(img) .content {
      padding: var(--space-md);
    }
    
    /* Container Query */
    @container card (max-width: 400px) {
      padding: var(--space-sm);
      
      & .title {
        font-size: var(--font-size-md);
      }
    }
  }
}
```

---

## 10.7 Современные рекомендации

### Принципы хорошего стиля

Сегодня хорошим стилем считается:

| Принцип | Пример |
|---------|--------|
| ✔ Использовать простые селекторы | `.button`, `.card`, `.nav-link` |
| ✔ Управлять приоритетами через `@layer` | `@layer components { .button { ... } }` |
| ✔ Снижать специфичность через `:where()` | `:where(.button) { ... }` |
| ✔ Использовать `:has()` для реляционной логики | `.card:has(img) { ... }` |
| ✔ Строить API через data-атрибуты | `[data-variant="primary"]` |
| ✔ Ограничивать область через `@scope` | `@scope (.card) { ... }` |
| ✔ Избегать длинных цепочек потомков | `.card .title` вместо `.card .header .title` |
| ✔ Не использовать ID в CSS | ❌ `#app`, ❌ `#header` |

### Чего следует избегать

```css
/* ❌ Плохо — длинная цепочка потомков */
#app .sidebar ul li a.button.active {
  color: blue;
}

/* ❌ Плохо — ID в селекторах */
#header { ... }
#main { ... }

/* ❌ Плохо — !important */
.button {
  background: blue !important;
}

/* ❌ Плохо — чрезмерная специфичность */
.card .content .title .link {
  color: blue;
}
```

### Современные альтернативы

```css
/* ✅ Хорошо — простой селектор + @layer */
@layer components {
  .nav-link {
    color: blue;
  }
}

/* ✅ Хорошо — :where() для базовых стилей */
:where(.button) {
  background: blue;
}

/* ✅ Хорошо — @layer для управления приоритетами */
@layer overrides {
  .button {
    background: red;
  }
}

/* ✅ Хорошо — плоская структура */
.card .title-link {
  color: blue;
}
```

---

## 10.8 Будущее селекторов

### Эволюция CSS-селекторов

```text
CSS 2.1
  ↓
Выбор по структуре документа
  ↓
CSS 3
  ↓
Выбор по состоянию (:nth-child, :not)
  ↓
Selectors Level 4
  ↓
Логические селекторы (:is, :where, :has)
  ↓
CSS 2026
  ↓
Реляционная модель + архитектурные инструменты
  ↓
Selectors Level 5 (будущее)
  ↓
Более глубокая интеграция с компонентной моделью
```

### Что уже возможно

Современные селекторы уже позволяют:

- **Учитывать структуру документа** — `:has()`, `:nth-child(of)`
- **Анализировать состояние потомков** — `:has(:invalid)`, `:has(:focus-visible)`
- **Управлять специфичностью** — `:where()`, `:is()`
- **Группировать сложные выражения** — `:is()`, `:where()`
- **Работать с областями компонентов** — `@scope`
- **Интегрироваться с каскадными слоями** — `@layer`
- **Естественно сочетаться с нативной вложенностью** — CSS Nesting

### Вероятные направления развития

**Selectors Level 5** вероятнее всего будет связан не столько с появлением новых типов селекторов, сколько с:

- Более глубокой интеграцией с компонентной моделью веб-платформы
- Работой с контейнерными запросами
- Декларативными возможностями браузера
- Расширением реляционной модели

---

## 10.9 Итоги главы

1. **Современные селекторы — часть архитектуры CSS**, а не просто синтаксис поиска элементов

2. **Три механизма управления приоритетами:** `@layer` (приоритет), `@scope` (область), логические псевдоклассы (специфичность)

3. **Сложность переносится с селекторов на каскад** — селекторы становятся простыми

4. **Логические селекторы превращают CSS в язык условий:** `:is()` (OR), `:not()` (NOT), `:has()` (EXISTS)

5. **`:where()` — инструмент архитектуры** с нулевой специфичностью для reset-стилей, дизайн-систем и библиотек

6. **CSS Nesting — часть стандарта** — вложенность разворачивается аналогично `:is()`, специфичность не увеличивается

7. **Селекторы как контракт компонентов** — data-атрибуты становятся API компонентов

8. **Интеграция с `@layer`, `@scope`, Container Queries** — селекторы работают в экосистеме

9. **Лучшие практики:** простые селекторы, `:where()` для базовых стилей, `@layer` для приоритетов

10. **Будущее:** дальнейшая интеграция с компонентной моделью веб-платформы

---

**Главная мысль:** Современные селекторы перестали быть лишь механизмом поиска элементов в DOM. Они стали одним из ключевых инструментов архитектуры CSS, определяя область действия правил, уровень специфичности, структуру компонентов и декларативную логику интерфейсов. В экосистеме **Modern CSS 2026** грамотное использование селекторов означает не умение писать самые сложные выражения, а способность проектировать простые, предсказуемые и масштабируемые системы стилей, где приоритеты определяются архитектурой каскада, а не «войнами специфичности».

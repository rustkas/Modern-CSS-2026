Отлично! Полностью перерабатываю Главу 1 с учётом ваших замечаний и глубоких архитектурных идей.

---

# Глава 1. Рождение CSS-платформы

**Почему современный CSS уже нельзя изучать как набор свойств**

---

## 1.1 История CSS как история постоянного переноса логики в браузер

CSS начал свой путь в 1996 году как простой язык стилизации документов. За три десятилетия он прошёл путь от раскрашивания текста до полноценной платформы, способной принимать решения.

```text
1996    →    раскрашиваем текст
2005    →    строим layout
2015    →    Flex/Grid
2020    →    адаптивность
2023    →    логика
2026    →    платформа
```

**Главная мысль:** каждые несколько лет браузер забирал ещё один класс задач у JavaScript. То, что раньше требовало императивного кода, теперь решается декларативно.

**Эволюция на примере центрирования:**
```css
/* 1996 — таблицы */
<table><tr><td align="center">...</td></tr></table>

/* 2005 — margin-хаки */
.center {
  margin: 0 auto;
  width: 50%;
}

/* 2015 — Flexbox */
.center {
  display: flex;
  justify-content: center;
  align-items: center;
}

/* 2026 — Grid */
.center {
  display: grid;
  place-items: center;
}
```

Каждый шаг — это удаление хаков и приближение к декларативной логике.

---

## 1.2 Почему CSS больше не является "языком стилей"

Современный CSS умеет самостоятельно анализировать и принимать решения на основе множества факторов:

**CSS анализирует DOM:**
```css
.card:has(img) {
  border-color: gold;
  padding: 2rem;
}
```
Карточка с изображением выглядит иначе, чем без него.

**CSS анализирует Layout:**
```css
@container (min-width: 400px) {
  .card {
    grid-template-columns: 1fr 1fr;
  }
}
```
Компонент адаптируется к доступному пространству.

**CSS анализирует устройство:**
```css
@media (prefers-color-scheme: dark) {
  :root {
    --bg: #1a1a1a;
    --text: #f0f0f0;
  }
}
```
Стили зависят от настроек пользователя.

**CSS анализирует стили:**
```css
@container style(--theme: dark) {
  .card {
    background: #2a2a2a;
  }
}
```
Стили зависят от значений CSS-переменных.

**CSS анализирует прокрутку:**
```css
@keyframes reveal {
  from { opacity: 0; transform: translateY(50px); }
  to { opacity: 1; transform: translateY(0); }
}

.reveal {
  animation: reveal linear;
  animation-timeline: view();
}
```
Анимация запускается автоматически при прокрутке.

**CSS анализирует положение элементов:**
```css
.tooltip {
  position: fixed;
  position-anchor: --target;
  top: anchor(bottom);
  left: anchor(left);
}
```
Элемент позиционируется относительно другого без JavaScript.

---

**Общая схема работы современного CSS:**

```text
CSS
  ↓
получает состояние
  ↓
принимает решение
  ↓
изменяет интерфейс
```

Это уже не язык оформления. Это язык **декларативной логики пользовательского интерфейса.**

---

## 1.3 Новая архитектура браузеров

Чтобы понять мощь современного CSS, важно увидеть, как браузер обрабатывает страницу. CSS влияет на каждый этап Rendering Pipeline:

```text
DOM
  ↓
Style        ← CSS Rules, Specificity, Cascade, Variables
  ↓
Layout       ← Grid, Flexbox, Container Queries, Subgrid
  ↓
Paint        ← Houdini Paint API, Colors, Gradients, Shadows
  ↓
Composite    ← Transform, Opacity, View Transitions, Will-change
```

**Что это значит на практике:**

| Этап | CSS-технологии | Что происходит |
|------|---------------|----------------|
| **Style** | `@layer`, Custom Properties, `@scope` | Браузер вычисляет, какие правила применены к каждому элементу |
| **Layout** | Grid, Flexbox, Container Queries, Subgrid | Браузер рассчитывает позиции и размеры элементов |
| **Paint** | OKLCH, `color-mix()`, Houdini Paint | Браузер рисует пиксели: цвета, фоны, границы |
| **Composite** | View Transitions, `will-change`, `transform` | Браузер накладывает слои друг на друга |

**Ключевой вывод:** CSS перестал быть "файлом со стилями". Он стал частью Rendering Engine, влияя на все этапы создания страницы.

---

## 1.4 Почему JavaScript постепенно теряет обязанности

Каждый год несколько JavaScript-задач переходят в CSS. Это не случайность — это архитектурное решение Web Platform.

| Раньше (требовался JavaScript) | Сегодня (нативный CSS) |
|-------------------------------|----------------------|
| Модальное окно | `<dialog>` + `::backdrop` |
| Всплывающая подсказка | Popover API |
| Аккордеон | `<details>` + `<summary>` |
| Masonry-сетка | CSS Grid + Subgrid |
| Резиновый компонент | Container Queries |
| Анимация при скролле | Scroll-driven Animations |
| Смена темы | CSS Variables + `@media (prefers-color-scheme)` |
| Стилизация родителя | `:has()` |
| Расчёт цветов | `color-mix()`, Relative Colors |
| Позиционирование оверлеев | Anchor Positioning |

**Тенденция очевидна:** браузер стремится обеспечить декларативные альтернативы императивному коду.

---

## 1.5 Современный CSS — декларативная state machine

Традиционный подход к состояниям интерфейса был императивным:

```javascript
// JavaScript — императивно
button.addEventListener('mouseenter', () => {
  button.classList.add('hover');
});
button.addEventListener('mouseleave', () => {
  button.classList.remove('hover');
});
```

Современный CSS описывает состояния декларативно:

```css
/* CSS — декларативно */
button:hover { ... }
button:focus { ... }
button:active { ... }
button:disabled { ... }
button:user-invalid { ... }
dialog:open { ... }
:popover-open { ... }
details:open { ... }
```

**Почему это важно:**

- CSS описывает **что** должно произойти, не описывая **как**
- Браузер сам управляет состояниями
- Нет race conditions
- Нет утечек памяти
- Автоматическая работа с фокусом и доступностью

**Пример сложного состояния:**
```css
/* Форма, где поле стало невалидным после взаимодействия */
input:user-invalid {
  border-color: red;
  box-shadow: 0 0 0 3px red;
}

/* Только если поле обязательно */
input[required]:user-invalid::after {
  content: '⚠️ Обязательное поле';
}

/* Карточка с изображением в тёмной теме */
.card:has(img) {
  background: color-mix(in oklch, var(--bg), var(--accent) 10%);
}
```

CSS больше не просто раскрашивает элементы — он управляет логикой их отображения.

---

## 1.6 Эволюция каскада

Каскад всегда был фундаментальной концепцией CSS, но его роль менялась:

```text
CSS2
  ↓
Specificity (вес селекторов)
  ↓
CSS Variables (динамические значения)
  ↓
Cascade Layers (@layer) — архитектурная организация
  ↓
Scoped Styles (@scope) — ограничение области действия
```

**Что изменилось:**

| Эпоха | Проблема | Решение |
|-------|----------|---------|
| CSS2 | Конфликты стилей | Specificity |
| CSS3 | Повторяющиеся значения | Переменные (только препроцессоры) |
| 2016 | Динамические значения | Custom Properties |
| 2022 | Архитектурный хаос | `@layer` |
| 2023 | Изоляция компонентов | `@scope` |

**Современный подход:**
```css
@layer reset, tokens, base, components, utilities, overrides;

@layer components {
  .button {
    background: var(--button-bg);
    border: var(--button-border);
    border-radius: var(--radius);
    padding: var(--space-2) var(--space-4);
  }
}

@layer overrides {
  .button-danger {
    --button-bg: var(--danger);
  }
}
```

Каскад стал **архитектурой приложения**, а не просто правилом разрешения конфликтов.

---

## 1.7 CSS становится типизированным языком

Одно из самых недооценённых изменений — появление типов в CSS.

**Раньше:** все значения были строками.
```css
:root {
  --size: 16px;      /* строка */
  --color: #3498db;  /* строка */
  --duration: 300ms; /* строка */
}
```

**Сегодня:**
```css
@property --size {
  syntax: '<length>';
  inherits: false;
  initial-value: 16px;
}

@property --color {
  syntax: '<color>';
  inherits: true;
  initial-value: #3498db;
}

@property --duration {
  syntax: '<time>';
  inherits: false;
  initial-value: 300ms;
}
```

**Что даёт типизация:**

| Тип | Использование |
|-----|--------------|
| `<length>` | Пиксели, em, rem, % |
| `<color>` | Все цветовые форматы |
| `<angle>` | 45deg, 0.25turn |
| `<number>` | 1, 2, 3 |
| `<integer>` | 1, 2, 3 |
| `<percentage>` | 50% |
| `<time>` | 300ms, 2s |
| `<transform-function>` | rotate(), scale() |

**Преимущества:**
- **Валидация:** браузер проверяет типы
- **Анимация:** интерполяция работает корректно
- **Инструменты:** лучшее автодополнение, проверка ошибок
- **Typed OM:** JavaScript может работать с типами безопасно

```javascript
// Typed OM — работа с типами в JavaScript
const size = element.attributeStyleMap.get('--size');
console.log(size.value);      // 16
console.log(size.unit);       // 'px'
```

---

## 1.8 CSS становится расширяемым

Самый философский сдвиг в CSS — возможность расширять язык без участия браузеров.

**Houdini — набор API для расширения CSS:**

```text
Houdini
  ↓
Paint API      →  новые свойства background
  ↓
Layout API     →  новые модели layout
  ↓
Animation API  →  новые типы анимаций
  ↓
Properties API →  новые CSS-свойства
```

**Пример: кастомный фон через Paint API:**
```javascript
// JavaScript
registerPaint('dots', class {
  paint(ctx, size) {
    ctx.fillStyle = '#3498db';
    for (let y = 0; y < size.height; y += 20) {
      for (let x = 0; x < size.width; x += 20) {
        ctx.beginPath();
        ctx.arc(x, y, 5, 0, Math.PI * 2);
        ctx.fill();
      }
    }
  }
});
```

```css
/* CSS */
.element {
  background: paint(dots);
}
```

**Пример: кастомное свойство с типом:**
```css
@property --my-gradient {
  syntax: '<color>#<color>';
  inherits: false;
  initial-value: #3498db#ecf0f1;
}
```

**Философский сдвиг:**
- Раньше: разработчик использует то, что создали браузеры
- Теперь: разработчик создаёт новые возможности CSS

**Это огромный сдвиг в мышлении:** CSS перестал быть закрытым языком, заданным спецификацией. Он стал открытой платформой.

---

## 1.9 Почему Sass перестал быть обязательным

Sass был создан для того, чего не хватало CSS. Сейчас большинство этих недостатков устранены.

**Сравнение подходов:**

```text
Sass
  ↓
компилятор → статический CSS
```

```text
CSS
  ↓
runtime → динамический интерфейс
```

**Что Sass давал и что CSS даёт сейчас:**

| Возможность Sass | Современный CSS | Преимущество CSS |
|-----------------|-----------------|------------------|
| Переменные | Custom Properties | Динамические, обновляются в рантайме |
| Вложенность | CSS Nesting | Браузерная поддержка |
| Миксины | Функции + Custom Properties | Работают в рантайме |
| Цветовые функции | `color-mix()`, Relative Colors | Динамическое вычисление |
| Управление каскадом | `@layer` | Декларативное управление |
| Импорты | `@import` + HTTP/2 | Параллельная загрузка |

**Пример: динамическая тема**
```scss
// Sass — статическая
$bg-light: #fff;
$bg-dark: #1a1a1a;

.theme-dark {
  background: $bg-dark;
}
```

```css
/* CSS — динамическая */
:root {
  --bg: #fff;
}

@media (prefers-color-scheme: dark) {
  :root {
    --bg: #1a1a1a;
  }
}

/* Автоматическое обновление */
.component {
  background: var(--bg);
}
```

**Когда Sass всё ещё нужен:**
- Сложные циклы и условия (редко)
- Сторонние библиотеки
- Исторический код

**Главная причина перехода:** современный CSS выигрывает тем, что всё происходит во время выполнения, а не на этапе сборки. Это даёт динамичность, адаптивность и сокращает время разработки.

---

## 1.10 Baseline и новая модель совместимости

Раньше поддержка браузеров выглядела как хаос:

```text
IE 6-8 → можно
Chrome → почти всегда
Firefox → почти всегда
Safari → иногда
Opera → что-то работает
```

Сегодня существует новая модель — **Baseline.**

**Что такое Baseline:**
- Функция считается доступной, когда она реализована в трёх основных браузерах (Chrome, Firefox, Safari)
- Каждый год публикуется новый Baseline (Baseline 2023, 2024, 2025, 2026)

**Как это выглядит:**
```text
Baseline 2026
  ↓
доступно в:
  • Chrome 120+
  • Firefox 120+
  • Safari 17.4+
```

**Преимущества для разработчика:**
- Один порог поддержки вместо таблицы
- Можно использовать все функции текущего Baseline
- Автоматические обновления

**Стратегия работы с новыми функциями:**

```css
/* 1. Проверка поддержки через @supports */
@supports (container-type: inline-size) {
  /* Используем Container Queries */
}

@supports (view-transition: 1) {
  /* Используем View Transitions */
}

/* 2. Progressive Enhancement */
.card {
  /* Базовые стили — работают всегда */
  display: flex;
  flex-direction: column;
}

@supports (display: grid) {
  .card {
    display: grid;
    /* Улучшенный layout, если поддерживается */
  }
}

@supports (container-type: inline-size) {
  .card {
    container-type: inline-size;
    /* Ещё более адаптивный layout */
  }
}
```

**Новая философия:** вместо борьбы с браузерами — использование того, что уже есть в платформе.

---

## 1.11 Progressive Enhancement становится главным принципом CSS

Progressive Enhancement — это архитектурная стратегия, которая стала фундаментом современной Web Platform.

```text
HTML
  ↓
работает всегда
  ↓
CSS
  ↓
улучшает интерфейс
  ↓
JavaScript
  ↓
добавляет сложную логику
```

**Принцип в действии:**

```html
<!-- 1. HTML — структура, работает всегда -->
<details>
  <summary>Показать детали</summary>
  <div class="content">
    Содержимое доступно всегда, даже без CSS.
  </div>
</details>
```

```css
/* 2. CSS — улучшает внешний вид и поведение */
details {
  border: 1px solid var(--border);
  border-radius: var(--radius);
  padding: var(--space-2);
}

summary {
  cursor: pointer;
  font-weight: bold;
  color: var(--link-color);
  &:hover {
    color: var(--link-hover);
  }
}

/* 3. Анимация — если CSS умеет */
details[open] .content {
  animation: slideDown 0.3s ease;
}

/* 4. Дополнительная логика — если нужен JS */
/* Подгрузка данных, валидация, сложные взаимодействия */
```

**Почему это важно:**
- **Доступность:** базовый слой работает для всех
- **Производительность:** не нужно загружать JS для базовой функциональности
- **Надёжность:** отказ JS не ломает интерфейс
- **SEO:** контент доступен поисковикам
- **Будущее:** работает даже в новых браузерах

**Современный подход к Progressive Enhancement:**
- 80% задач решается HTML + CSS
- 15% требует минимального JavaScript
- 5% — сложные сценарии

---

## 1.12 Практическая трансформация проекта

Лучший способ понять эволюцию CSS — увидеть, как выглядит трансформация реального проекта.

**Исходный проект: Bootstrap 3 (2013)**

```html
<div class="container">
  <div class="row">
    <div class="col-md-4">
      <div class="panel panel-default">
        <div class="panel-heading">
          <h3 class="panel-title">Карточка товара</h3>
        </div>
        <div class="panel-body">
          <img src="product.jpg" class="img-responsive">
          <p>Описание товара</p>
          <button class="btn btn-primary">Купить</button>
        </div>
      </div>
    </div>
  </div>
</div>
```

```css
/* Требовался Clearfix */
.row::before,
.row::after {
  content: " ";
  display: table;
}
.row::after {
  clear: both;
}

/* Медиа-запросы по вьюпорту */
@media (max-width: 992px) {
  .col-md-4 {
    float: left;
    width: 33.333333%;
  }
}

/* !important в компонентах */
.btn-primary {
  background: #337ab7 !important;
  border: none !important;
}
```

**Трансформация: CSS 2026**

```css
/* 1. Больше не нужен Clearfix */
/* 2. Используем Grid вместо Float */
/* 3. Container Queries вместо Media Queries */
/* 4. @layer вместо !important */
/* 5. Custom Properties вместо переменных Sass */
/* 6. :has() для адаптивных карточек */

@layer reset, tokens, base, components, utilities;

@layer tokens {
  :root {
    --spacing: 1rem;
    --radius: 8px;
    --primary: #3498db;
    --bg: #ffffff;
    --text: #2c3e50;
  }
}

@layer components {
  .product-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
    gap: var(--spacing);
    container-type: inline-size;
    container-name: grid;
  }

  .product-card {
    display: grid;
    grid-template-rows: auto 1fr auto;
    gap: var(--spacing);
    background: var(--bg);
    border-radius: var(--radius);
    padding: var(--spacing);
    transition: transform 0.2s;

    &:hover {
      transform: translateY(-4px);
    }

    /* Стилизация по наличию изображения */
    &:has(img) {
      border: 2px solid var(--primary);
    }
  }

  .product-card img {
    width: 100%;
    aspect-ratio: 16/9;
    object-fit: cover;
    border-radius: calc(var(--radius) / 2);
  }

  .product-card button {
    background: var(--primary);
    color: white;
    border: none;
    border-radius: var(--radius);
    padding: 0.75rem 1.5rem;
    cursor: pointer;
    font-weight: bold;

    &:hover {
      background: color-mix(in oklch, var(--primary), black 15%);
    }
  }
}

/* Адаптивность через Container Query */
@container grid (max-width: 600px) {
  .product-grid {
    grid-template-columns: 1fr;
  }

  .product-card {
    grid-template-rows: auto 1fr auto;
  }
}

/* Контрастная тема */
@media (prefers-color-scheme: dark) {
  @layer tokens {
    :root {
      --bg: #1a1a1a;
      --text: #f0f0f0;
      --primary: #4a9eff;
    }
  }
}
```

**Что исчезло из проекта:**
- ❌ clearfix-хаки
- ❌ !important
- ❌ медиа-запросы по вьюпорту для компонентов
- ❌ float-сетки
- ❌ отдельные файлы для тем
- ❌ зависимость от CSS-фреймворка

**Что появилось:**
- ✅ Декларативная логика
- ✅ Архитектурная организация
- ✅ Динамические темы
- ✅ Компонентная адаптивность
- ✅ Родительские селекторы
- ✅ Вложенность

---

## 1.13 Главная идея главы

CSS прошёл путь от языка оформления документов до полноценной декларативной платформы пользовательского интерфейса.

**Раньше CSS лишь описывал внешний вид элементов.** Сегодня он:

- **участвует в принятии решений** — `:has()`, `@container`
- **управляет состояниями** — `:user-invalid`, `:popover-open`
- **адаптируется к окружению** — Container Queries, Media Queries
- **взаимодействует с механизмами рендеринга** — Rendering Pipeline
- **постепенно берёт задачи JavaScript** — View Transitions, Popover

Современный фронтенд строится не вокруг борьбы между HTML, CSS и JavaScript, а вокруг их сотрудничества:

```text
HTML
  ↓
описывает структуру и семантику
  ↓
CSS
  ↓
отвечает за визуальную логику и поведение интерфейса
  ↓
JavaScript
  ↓
реализует сложные сценарии, которые невозможно выразить декларативно
```

Именно этот переход от императивного программирования к возможностям веб-платформы и является главной темой книги **Modern CSS 2026.**

**Три фундаментальных изменения:**

1. **От статики к динамике** — CSS теперь реагирует на состояние
2. **От оформления к логике** — CSS принимает решения
3. **От языка к платформе** — CSS расширяется и развивается

---

## 1.14 Итоги главы

1. **CSS эволюционирует:** каждые несколько лет браузер забирает у JavaScript новые задачи

2. **CSS — это логика, а не просто стили:** он анализирует DOM, Layout, устройство, стили, прокрутку и позиционирование

3. **CSS влияет на каждый этап рендеринга:** Style, Layout, Paint, Composite

4. **JavaScript теряет задачи:** модальные окна, tooltip, аккордеоны, анимации, позиционирование — теперь в CSS

5. **CSS — декларативная state machine:** состояния описываются, а не программируются

6. **Каскад стал архитектурой:** `@layer`, `@scope`, Custom Properties управляют организацией кода

7. **CSS стал типизированным:** `@property` добавляет типы, валидацию и интерполяцию

8. **CSS стал расширяемым:** Houdini позволяет разработчикам создавать новые возможности

9. **Sass больше не обязателен:** всё, что давал Sass, теперь есть в нативном CSS

10. **Baseline меняет подход к совместимости:** один порог поддержки вместо таблицы

11. **Progressive Enhancement — главный принцип:** HTML → CSS → JS

12. **CSS 2026 — это платформа:** язык принятия решений, а не просто набор свойств

---


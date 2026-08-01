# Глава 4. CSS как язык компонентов

> **Цель главы:** показать, как современный CSS превратился в основу компонентной архитектуры веб-приложений. После этой главы читатель должен понимать, что компонент — это уже не особенность React, Angular или Vue, а нативная возможность современной веб-платформы.

---

## 4.1 От страницы к компонентам

### Эволюция мышления в CSS

За последние двадцать лет архитектура веб-интерфейсов прошла путь от отдельных страниц к системам компонентов:

```text
Page (1990-е)
  ↓
Blocks (2000-е)
  ↓
Widgets (2010-е)
  ↓
Components (2015-2020)
  ↓
Design System (2020-2026)
```

**Что изменилось:**

| Эпоха          | Мышление               | CSS-подход                          |
| -------------- | ---------------------- | ----------------------------------- |
| Страницы       | Документы              | Глобальный CSS                      |
| Блоки          | Секции                 | Каскад + селекторы                  |
| Виджеты        | Переиспользуемые части | BEM, SMACSS, OOCSS                  |
| Компоненты     | Независимые единицы    | CSS Modules, CSS-in-JS              |
| Дизайн-системы | Экосистема             | `@layer`, Tokens, Container Queries |

**Ключевой вывод:** современный интерфейс состоит из небольших независимых компонентов. CSS постепенно перестаёт быть глобальным.

### Почему компонентное мышление изменило CSS

```text
Глобальный CSS
  ↓
Стили влияют на всё
  ↓
Селекторы становятся длиннее
  ↓
Специфичность растёт
  ↓
!important появляется
  ↓
Сложность растёт экспоненциально
```

Против:

```text
Компонентный CSS
  ↓
Стили ограничены компонентом
  ↓
Селекторы короткие
  ↓
Специфичность низкая
  ↓
!important не нужен
  ↓
Сложность растёт линейно
```

---

## 4.2 Что такое компонент в современной веб-платформе

### Архитектура современного компонента

```text
HTML
  ↓
Структура и семантика
  ↓
CSS
  ↓
Внешний вид и адаптивность
  ↓
State
  ↓
Состояния и вариации
  ↓
Behavior
  ↓
Интерактивность и логика
```

### Что принадлежит каждой части

**HTML:**

- Структура компонента
- Семантические элементы
- ARIA-атрибуты
- Атрибуты состояния (`data-*`, `aria-*`, стандартные)

**CSS:**

- Внешний вид (цвета, размеры, отступы)
- Адаптивность (Container Queries, Media Queries)
- Состояния (`:hover`, `:focus`, `:disabled`, `:user-invalid`)
- Анимации (View Transitions, Scroll-driven Animations)
- Позиционирование (Anchor Positioning)

**State (управляется браузером и CSS):**

- `:hover`, `:focus`, `:active`
- `:checked`, `:disabled`
- `:valid`, `:invalid`, `:user-invalid`
- `:popover-open`, `:open`
- `prefers-color-scheme`, `prefers-reduced-motion`

**Behavior (браузерные API):**

- `<dialog>` — модальные окна
- Popover API — выпадающие элементы
- `<details>` — аккордеоны
- View Transitions — навигация
- Scroll-driven Animations — анимация прокрутки

### Почему современная платформа постепенно берёт часть ответственности на себя

```text
JavaScript
  ↓
Браузерные API
  ↓
Декларативный HTML/CSS
  ↓
Меньше JavaScript
```

**Пример: аккордеон**

```html
<!-- JavaScript: сложный код -->
<div class="accordion">
  <button onclick="toggleAccordion()">Заголовок</button>
  <div id="content">Содержимое</div>
</div>
```

```html
<!-- HTML + CSS: декларативно -->
<details>
  <summary>Заголовок</summary>
  <div>Содержимое</div>
</details>
```

---

## 4.3 Design Tokens как фундамент дизайн-систем

### Что такое Design Tokens

Design Tokens — это атомарные единицы дизайна, выраженные через переменные. Они создают единый язык интерфейса.

```text
Brand
  ↓
Design Tokens
  ↓
Components
  ↓
Pages
```

### Пример токенов

```css
/* Цветовые токены */
--color-primary: #3498db;
--color-secondary: #2ecc71;
--color-danger: #e74c3c;
--color-warning: #f39c12;
--color-bg: #ffffff;
--color-text: #1a1a1a;

/* Пространственные токены */
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
--font-family-body: system-ui, -apple-system, sans-serif;
--font-family-heading: 'Inter', system-ui, sans-serif;
--font-size-sm: 0.875rem;
--font-size-base: 1rem;
--font-size-lg: 1.125rem;
--font-size-xl: 1.25rem;
--font-size-2xl: 1.5rem;
--font-weight-normal: 400;
--font-weight-medium: 500;
--font-weight-bold: 700;
```

### Почему Design Tokens стали промышленным стандартом

1. **Единый язык** — дизайнеры, разработчики, менеджеры говорят на одном языке
2. **Масштабируемость** — добавление новых токенов не ломает существующие
3. **Темизация** — одна система, множество тем
4. **Инструменты** — Figma, Style Dictionary, Open Props

### Интеграция с инструментами

**Figma + Style Dictionary:**

```text
Figma (Design)
  ↓
Style Dictionary (Transform)
  ↓
CSS Variables + JSON + SCSS
  ↓
Приложение
```

**Open Props — готовые токены:**

```css
@import 'https://unpkg.com/open-props' layer(tokens);

/* Используем готовые токены */
.card {
  background: var(--gray-0);
  padding: var(--size-4);
  border-radius: var(--radius-3);
  box-shadow: var(--shadow-4);
}
```

**Design Tokens Community Group:**

- [W3C Design Tokens Community Group](https://www.w3.org/community/design-tokens/)
- Стандартизация формата токенов
- Инструменты для генерации кода

---

## 4.4 Custom Properties и `@property`

### Почему CSS-переменные стали динамическими

```css
/* Статические переменные (Sass) */
$color-primary: #3498db;
$space-base: 1rem;

/* Динамические переменные (CSS) */
:root {
  --color-primary: #3498db;
  --space-base: 1rem;
}
```

**Динамические свойства:**

- Изменяются в рантайме
- Реагируют на темы
- Передаются через Shadow DOM
- Могут быть анимированы

### Зарегистрированные свойства (`@property`)

```css
/* Без @property — строка */
:root {
  --rotation: 0deg;
  --color: #3498db;
  --size: 16px;
}

/* С @property — типизированное значение */
@property --rotation {
  syntax: '<angle>';
  inherits: false;
  initial-value: 0deg;
}

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
```

### Почему современные компоненты используют зарегистрированные свойства

```css
/* Анимация угла поворота */
@property --angle {
  syntax: '<angle>';
  inherits: false;
  initial-value: 0deg;
}

.element {
  --angle: 0deg;
  transform: rotate(var(--angle));
  transition: --angle 2s ease;
}

.element:hover {
  --angle: 360deg; /* Анимируется плавно благодаря @property */
}
```

**Преимущества `@property` в компонентах:**

1. **Типизация** — браузер проверяет типы
2. **Анимация** — свойства анимируются корректно
3. **Наследование** — явное управление наследованием
4. **Документация** — типы документируют API

---

## 4.5 Изоляция компонентов

### История проблемы

```text
Global CSS
  ↓
Все стили конфликтуют
  ↓
BEM (Блок-Элемент-Модификатор)
  ↓
Соглашение об именовании
  ↓
CSS Modules
  ↓
Изоляция на уровне файлов
  ↓
Shadow DOM
  ↓
Истинная изоляция
  ↓
@scope
  ↓
Изоляция на уровне CSS
```

### Когда использовать разные подходы

| Подход          | Когда использовать              | Преимущества            | Недостатки                  |
| --------------- | ------------------------------- | ----------------------- | --------------------------- |
| **BEM**         | Проекты без инструментов сборки | Простота, понятность    | Длинные классы, ручной труд |
| **CSS Modules** | Проекты с Webpack/Vite          | Локальная изоляция      | Зависит от сборки           |
| **Shadow DOM**  | Web Components                  | Полная изоляция         | Сложность, ограничения      |
| **@scope**      | Любые проекты                   | CSS-изоляция без HTML   | Требует поддержки браузера  |
| **@layer**      | Архитектура проекта             | Управление приоритетами | Только для приоритетов      |

### Практическое сравнение

**BEM:**

```css
/* Глобальные имена с префиксами */
.block {
}
.block__element {
}
.block--modifier {
}
```

**CSS Modules:**

```css
/* Локальные имена в сборке */
.component {
}
```

**Shadow DOM:**

```html
<my-component>
  #shadow-root
  <!-- Полная изоляция HTML + CSS -->
</my-component>
```

**@scope:**

```css
@scope (.component) {
  /* Стили только внутри .component */
  .title { ... }
}
```

---

## 4.6 Container Queries

### Одно из главных изменений последних лет

```text
Viewport (Media Queries)
  ↓
Вся страница
  ↓
Container (Container Queries)
  ↓
Отдельный компонент
```

### Почему адаптивность должна зависеть от компонента

**Проблема Media Queries:**

```css
/* Реагирует на размер окна браузера */
@media (max-width: 768px) {
  .card {
    flex-direction: column;
  }
}
```

Если карточка находится в узкой колонке на широком экране, она не адаптируется.

**Решение Container Queries:**

```css
/* Реагирует на размер родительского контейнера */
.product-card {
  container-type: inline-size;
}

@container (max-width: 400px) {
  .product-card {
    flex-direction: column;
  }
}
```

### Практический пример

```css
/* Определяем контейнер */
.sidebar {
  container-type: inline-size;
  container-name: sidebar;
}

/* Компонент внутри контейнера */
@container sidebar (max-width: 300px) {
  .card {
    padding: 0.5rem;
    gap: 0.5rem;
  }
  .card-title {
    font-size: 1rem;
  }
  .card-image {
    display: none;
  }
}

@container sidebar (min-width: 301px) and (max-width: 600px) {
  .card {
    padding: 1rem;
    display: grid;
    grid-template-columns: 1fr 2fr;
  }
}

@container sidebar (min-width: 601px) {
  .card {
    padding: 1.5rem;
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
  }
}
```

### Почему Grid и Container Queries идеально работают вместе

```css
.product-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 1rem;
  container-type: inline-size;
  container-name: grid;
}

@container grid (max-width: 500px) {
  .product-grid {
    grid-template-columns: 1fr;
  }
}

@container grid (min-width: 501px) and (max-width: 800px) {
  .product-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@container grid (min-width: 801px) {
  .product-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}
```

---

## 4.7 Style Queries

### Совершенно новый уровень CSS

Style Queries позволяют компоненту реагировать на стиль родителя.

```css
/* Родитель определяет тему */
.sidebar {
  --theme: dark;
}

/* Компонент реагирует на тему */
@container style(--theme: dark) {
  .card {
    background: #1a1a1a;
    color: #f0f0f0;
    border-color: #333;
  }
}

@container style(--theme: light) {
  .card {
    background: #ffffff;
    color: #1a1a1a;
    border-color: #ddd;
  }
}
```

### Декларативная тема компонентов

```css
/* Определяем вариации компонента */
.card {
  container-type: inline-size;
  container-name: card;
}

/* Реагируем на тему родителя */
@container card style(--theme: dark) {
  .card {
    background: var(--dark-bg);
    color: var(--dark-text);
  }
  .card-title {
    color: var(--dark-title);
  }
}

/* Реагируем на размер */
@container card style(--size: large) {
  .card {
    padding: 2rem;
    border-radius: 16px;
  }
  .card-title {
    font-size: 2rem;
  }
}

@container card style(--size: small) {
  .card {
    padding: 0.5rem;
    border-radius: 4px;
  }
  .card-title {
    font-size: 0.875rem;
  }
}
```

### Использование Style Queries на практике

```html
<!-- Родитель задаёт тему -->
<div class="sidebar" style="--theme: dark; --size: large">
  <!-- Компонент автоматически адаптируется -->
  <div class="card">
    <h3 class="card-title">Заголовок</h3>
    <p>Содержимое</p>
  </div>
</div>
```

---

## 4.8 Контракты компонентов

### Как компонент сообщает о своих состояниях

Компонент должен иметь чёткий контракт — публичное API, через которое его можно настраивать.

```text
Attributes
  ↓
Data Attributes
  ↓
ARIA
  ↓
CSS States
```

### Атрибуты как публичное API

```html
<!-- Базовый компонент -->
<button class="btn">Кнопка</button>

<!-- Вариации через атрибуты -->
<button class="btn" data-variant="primary">Основная</button>
<button class="btn" data-variant="danger">Опасная</button>
<button class="btn" data-size="large">Большая</button>
<button class="btn" data-size="small">Маленькая</button>

<!-- Состояния -->
<button class="btn" disabled>Отключена</button>
<button class="btn" data-loading>Загрузка</button>
```

```css
.btn {
  padding: 0.5rem 1rem;
  border-radius: var(--radius-md);
  border: none;
  font-weight: var(--font-weight-medium);
  cursor: pointer;
  transition: all 0.2s ease;
}

.btn[data-variant='primary'] {
  background: var(--color-primary);
  color: white;
}

.btn[data-variant='danger'] {
  background: var(--color-danger);
  color: white;
}

.btn[data-size='large'] {
  padding: 0.75rem 1.5rem;
  font-size: var(--font-size-lg);
}

.btn[data-size='small'] {
  padding: 0.25rem 0.75rem;
  font-size: var(--font-size-sm);
}

.btn[disabled] {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn[data-loading] {
  opacity: 0.7;
  cursor: wait;
}
```

### ARIA и Accessibility

```html
<!-- ARIA-атрибуты как часть контракта -->
<button
  class="btn"
  aria-label="Закрыть"
  aria-expanded="false"
  aria-controls="panel"
>
  ✕
</button>

<!-- Использование в CSS -->
.btn[aria-expanded="true"] { background: var(--color-primary); }
```

### Почему атрибуты становятся публичным API компонента

1. **Документация** — атрибуты документируют возможности
2. **Тестирование** — легко проверять состояния
3. **Доступность** — интеграция с ARIA
4. **Фреймворки** — работают со всеми атрибутами

---

## 4.9 Компоненты и современные селекторы

### Как селекторы упрощают компонентный CSS

**`:is()` — группировка:**

```css
/* Вместо повторения */
.card .title,
.card .subtitle,
.card .content {
  color: var(--text-color);
}

/* Используем :is() */
.card :is(.title, .subtitle, .content) {
  color: var(--text-color);
}
```

**`:where()` — нулевая специфичность:**

```css
/* Базовые стили — легко переопределить */
:where(.btn) {
  padding: 0.5rem 1rem;
  border-radius: var(--radius-md);
}

/* Любой класс переопределяет без повышения специфичности */
.btn-primary {
  background: var(--color-primary);
  color: white;
}
```

**`:has()` — родительский селектор:**

```css
/* Карточка с изображением */
.card:has(img) {
  padding: 0;
  overflow: hidden;
}

.card:has(img) .card-content {
  padding: 1rem;
}

/* Карточка с кнопкой */
.card:has(.btn) {
  border: 2px solid var(--color-primary);
}

/* Форма с ошибкой */
.form-group:has(input:user-invalid) {
  border-color: var(--color-danger);
}
```

**`:not()` — исключение:**

```css
/* Все кроме первого */
.card:not(:first-child) {
  margin-top: 1rem;
}

/* Все кроме последнего */
.card:not(:last-child) {
  border-bottom: 1px solid var(--border-color);
}
```

### Сложный пример

```css
/* Компонент карточки с полной логикой */
.card {
  background: var(--card-bg);
  border-radius: var(--radius-lg);
  padding: var(--space-md);
  box-shadow: var(--shadow-sm);
  transition: all 0.3s ease;
}

/* Карточка с изображением */
.card:has(img) {
  padding: 0;
  overflow: hidden;
}

/* Карточка с изображением и контентом */
.card:has(img) .card-content {
  padding: var(--space-md);
}

/* Карточка с кнопкой */
.card:has(.btn) {
  border: 2px solid var(--color-primary);
}

/* Карточка в фокусе */
.card:has(:focus-visible) {
  box-shadow: var(--shadow-lg);
}

/* Карточка в тёмной теме */
.card:where(.card-dark) {
  --card-bg: var(--dark-bg);
  color: var(--dark-text);
}

/* Карточка в активном состоянии */
.card:active {
  transform: scale(0.98);
}
```

---

## 4.10 Современная композиция компонентов

### Архитектура композиции

```text
Design Tokens
  ↓
Primitive Components (атомы)
  ↓
Composite Components (молекулы)
  ↓
Application (организмы)
```

### Пример композиции

**Primitive Components (Атомы):**

```css
/* Базовые элементы */
.btn {
  /* Стили кнопки */
}

.input {
  /* Стили поля ввода */
}

.label {
  /* Стили метки */
}

.card {
  /* Стили карточки */
}
```

**Composite Components (Молекулы):**

```css
/* Форма входа — из атомов */
.login-form {
  display: flex;
  flex-direction: column;
  gap: var(--space-md);
}

.login-form .form-group {
  display: flex;
  flex-direction: column;
  gap: var(--space-xs);
}

.login-form .form-actions {
  display: flex;
  gap: var(--space-sm);
  justify-content: flex-end;
}
```

**Application (Организмы):**

```css
/* Страница авторизации */
.login-page {
  display: grid;
  place-items: center;
  min-height: 100vh;
  background: var(--color-bg);
}

.login-page .login-wrapper {
  max-width: 400px;
  width: 100%;
  padding: var(--space-xl);
  background: white;
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-lg);
}
```

### Почему CSS постепенно становится языком композиции

```css
/* Композиция через Container Queries */
@container (max-width: 600px) {
  .login-form {
    flex-direction: column;
  }
}

/* Композиция через Style Queries */
@container style(--theme: dark) {
  .login-page {
    background: var(--dark-bg);
  }
  .login-wrapper {
    background: var(--dark-card);
  }
}

/* Композиция через :has() */
.login-form:has(.input:user-invalid) {
  border-color: var(--color-danger);
}
```

---

## 4.11 Компоненты и Web Components

### Как взаимодействуют CSS и Web Components

```text
CSS
  ↓
Shadow DOM (изоляция)
  ↓
Custom Elements (повторное использование)
  ↓
Declarative Shadow DOM (SSR)
```

### Пример современного Web Component

```html
<!-- Определение компонента -->
<my-button variant="primary" size="large"> Нажми меня </my-button>
```

```javascript
// Custom Element
class MyButton extends HTMLElement {
  constructor() {
    super();
    const shadow = this.attachShadow({ mode: 'open' });
    shadow.innerHTML = `
      <style>
        /* Стили изолированы внутри Shadow DOM */
        @layer reset, components;
        
        @layer components {
          :host {
            display: inline-block;
          }
          
          .btn {
            padding: 0.5rem 1rem;
            border: none;
            border-radius: var(--radius-md, 8px);
            background: var(--btn-bg, #3498db);
            color: var(--btn-text, white);
            cursor: pointer;
            font-weight: 500;
            transition: all 0.2s ease;
          }
          
          /* Вариации через атрибуты */
          :host([variant="primary"]) .btn {
            --btn-bg: var(--color-primary, #3498db);
          }
          
          :host([variant="danger"]) .btn {
            --btn-bg: var(--color-danger, #e74c3c);
          }
          
          :host([size="large"]) .btn {
            padding: 0.75rem 1.5rem;
            font-size: 1.125rem;
          }
          
          :host([size="small"]) .btn {
            padding: 0.25rem 0.75rem;
            font-size: 0.875rem;
          }
          
          .btn:hover {
            transform: translateY(-2px);
            box-shadow: var(--shadow-md, 0 4px 12px rgba(0,0,0,0.1));
          }
          
          .btn:active {
            transform: scale(0.98);
          }
        }
      </style>
      <button class="btn">
        <slot>Кнопка</slot>
      </button>
    `;
  }
}

customElements.define('my-button', MyButton);
```

### Использование вне фреймворка

```html
<!-- Работает везде -->
<my-button variant="primary">Купить</my-button>
<my-button variant="danger" size="large">Удалить</my-button>

<!-- Можно стилизовать извне через Custom Properties -->
<my-button style="--btn-bg: #9b59b6;">Фиолетовая</my-button>
```

### Что принадлежит каждой части

| Часть          | Что делает                          |
| -------------- | ----------------------------------- |
| **HTML**       | Структура, атрибуты, слоты          |
| **CSS**        | Внешний вид, анимации, адаптивность |
| **JavaScript** | Поведение, события, логика          |

### Почему Web Components становятся частью современной платформы

1. **Нативная поддержка браузеров** — не нужен фреймворк
2. **Изоляция** — Shadow DOM защищает стили
3. **Стандартизация** — работают везде одинаково
4. **Производительность** — браузерная оптимизация
5. **Переносимость** — работают во всех фреймворках

---

## 4.12 CSS и современные фреймворки

### Что происходит после появления новых возможностей CSS

| Фреймворк   | Подход к стилям                           | Изменения в 2026                                       |
| ----------- | ----------------------------------------- | ------------------------------------------------------ |
| **React**   | CSS Modules, CSS-in-JS, Styled Components | Переход на нативный CSS + `@layer` + Container Queries |
| **Vue**     | Scoped CSS, CSS Modules                   | Использование `@scope` и Container Queries             |
| **Angular** | Shadow DOM, View Encapsulation            | Интеграция с `@layer` и Custom Properties              |
| **Svelte**  | Scoped CSS                                | Нативная изоляция через CSS                            |
| **Astro**   | Global CSS, Scoped CSS                    | `@layer` + Container Queries                           |
| **Qwik**    | Inline Styles, CSS-in-JS                  | Нативный CSS + Resumability                            |

### Почему CSS вытесняет альтернативы

**CSS-in-JS (Emotion, Styled Components):**

```javascript
const Button = styled.button`
  background: blue;
  color: white;
  padding: 0.5rem 1rem;
`;
```

**Нативный CSS:**

```css
/* Всё тоже самое, но без runtime-издержек */
.btn {
  background: blue;
  color: white;
  padding: 0.5rem 1rem;
}
```

**Преимущества нативного CSS:**

1. **Производительность** — нет runtime-вычислений
2. **Меньше кода** — CSS компилируется в CSS
3. **Инструменты** — DevTools, Stylelint, PostCSS
4. **Server-Side Rendering** — работает из коробки
5. **Container Queries** — поддерживаются только в нативном CSS

### Пример: React-компонент с нативным CSS

```jsx
// Button.jsx
import './Button.css';

function Button({ variant, size, children }) {
  return (
    <button className="btn" data-variant={variant} data-size={size}>
      {children}
    </button>
  );
}
```

```css
/* Button.css */
@layer components {
  .btn {
    padding: 0.5rem 1rem;
    border: none;
    border-radius: var(--radius-md);
    cursor: pointer;
    transition: all 0.2s ease;
  }

  .btn[data-variant='primary'] {
    background: var(--color-primary);
    color: white;
  }

  .btn[data-size='large'] {
    padding: 0.75rem 1.5rem;
    font-size: var(--font-size-lg);
  }
}
```

---

## 4.13 Progressive Enhancement компонентной архитектуры

### Как проектировать компоненты, работающие без JavaScript

```text
HTML (работает всегда)
  ↓
CSS (улучшает внешний вид)
  ↓
JavaScript (добавляет сложную логику)
```

### Пример: интерактивный компонент

```html
<!-- 1. HTML — работает всегда -->
<details class="accordion">
  <summary class="accordion-header">
    <h3>Заголовок</h3>
    <span class="accordion-icon">▼</span>
  </summary>
  <div class="accordion-content">
    <p>Содержимое доступно всегда.</p>
    <button class="btn" data-action="expand">Подробнее</button>
  </div>
</details>
```

```css
/* 2. CSS — улучшает внешний вид и поведение */
.accordion {
  border: 1px solid var(--border-color);
  border-radius: var(--radius-md);
  overflow: hidden;
}

.accordion-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: var(--space-md);
  cursor: pointer;
  transition: background 0.2s ease;
}

.accordion-header:hover {
  background: var(--hover-bg);
}

.accordion-icon {
  transition: transform 0.3s ease;
}

details[open] .accordion-icon {
  transform: rotate(180deg);
}

.accordion-content {
  padding: var(--space-md);
  border-top: 1px solid var(--border-color);
}

/* Анимация через Scroll Timeline */
.accordion-content {
  view-timeline-name: --accordion-content;
}
```

```javascript
// 3. JavaScript — добавляет сложную логику
// (загрузка данных, анимации, взаимодействие с сервером)
document.querySelectorAll('[data-action="expand"]').forEach((btn) => {
  btn.addEventListener('click', async () => {
    const content = await loadMoreData();
    // ...
  });
});
```

### Почему именно так строится современная платформа

1. **Доступность** — интерфейс работает для всех
2. **Производительность** — базовый функционал без JS
3. **SEO** — контент доступен поисковикам
4. **Надёжность** — отказ JS не ломает интерфейс
5. **Будущее** — работает в новых браузерах

---

## 4.14 Архитектурные рекомендации (Best Practices)

### Практические правила проектирования компонентов

**1. Начинайте с семантического HTML, а не со стилей**

```html
<!-- ✅ Хорошо -->
<article class="card">
  <h2>Заголовок</h2>
  <p>Описание</p>
  <button>Действие</button>
</article>

<!-- ❌ Плохо -->
<div class="card">
  <div class="card-title">Заголовок</div>
  <div class="card-text">Описание</div>
  <div class="card-action">Действие</div>
</div>
```

**2. Используйте Design Tokens вместо «магических» значений**

```css
/* ❌ Плохо */
.card {
  padding: 16px;
  border-radius: 8px;
  margin-bottom: 20px;
}

/* ✅ Хорошо */
.card {
  padding: var(--space-md);
  border-radius: var(--radius-md);
  margin-bottom: var(--space-lg);
}
```

**3. Применяйте Custom Properties для настройки компонентов**

```css
.card {
  --card-bg: var(--color-bg);
  --card-padding: var(--space-md);
  --card-radius: var(--radius-md);

  background: var(--card-bg);
  padding: var(--card-padding);
  border-radius: var(--card-radius);
}
```

**4. Регистрируйте анимируемые свойства через `@property`**

```css
@property --card-rotation {
  syntax: '<angle>';
  inherits: false;
  initial-value: 0deg;
}

.card {
  --card-rotation: 0deg;
  transform: rotate(var(--card-rotation));
  transition: --card-rotation 0.5s ease;
}

.card:hover {
  --card-rotation: 2deg;
}
```

**5. Адаптируйте компоненты с помощью Container Queries**

```css
.card-container {
  container-type: inline-size;
  container-name: card;
}

@container card (max-width: 400px) {
  .card {
    flex-direction: column;
  }
}
```

**6. Ограничивайте область действия стилей через `@scope` или Shadow DOM**

```css
@scope (.card) {
  .title {
    font-size: 1.5rem;
  }
  .content {
    padding: var(--space-md);
  }
}
```

**7. Используйте `@layer` для разделения архитектурных уровней**

```css
@layer tokens, base, components, utilities;

@layer components {
  .card {
    /* Стили компонента */
  }
}
```

**8. Проектируйте публичный API компонента через атрибуты**

```html
<button class="btn" data-variant="primary" data-size="large" disabled>
  Кнопка
</button>
```

**9. Минимизируйте специфичность селекторов**

```css
/* ❌ Плохо — высокая специфичность */
#app .sidebar .menu .item .link {
  color: blue;
}

/* ✅ Хорошо — низкая специфичность */
.menu-link {
  color: blue;
}
```

**10. Придерживайтесь Progressive Enhancement**

```text
HTML → работает всегда
  ↓
CSS → улучшает интерфейс
  ↓
JavaScript → добавляет сложную логику
```

---

## 4.15 Итоги главы

1. **CSS прошёл путь от стилей страницы к компонентам и дизайн-системам**

2. **Компонент в современной веб-платформе** — это сочетание HTML, CSS, состояния и поведения

3. **Design Tokens** — единый язык интерфейса через переменные

4. **`@property`** — типизированные CSS-переменные с анимацией и наследованием

5. **Изоляция компонентов** — BEM → CSS Modules → Shadow DOM → `@scope`

6. **Container Queries** — адаптивность к контейнеру, а не к вьюпорту

7

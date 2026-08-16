## 4.7 Style Queries

### Совершенно новый уровень CSS

Style Queries позволяют компоненту реагировать на стиль родителя.

> **Статус поддержки:** на момент выхода книги Style Queries остаются **Limited availability**. Возможность реализована в Chromium-браузерах (Chrome, Edge) и в Safari, но Firefox пока её не поддерживает — работа над реализацией ведётся в рамках инициативы Interop, но сроки окончательного релиза не зафиксированы. В отличие от размерных Container Queries (`@container (min-width: ...)`), которые давно достигли Baseline Widely available, Style Queries пока нельзя использовать как единственный механизм — только как прогрессивное улучшение поверх базового поведения.

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

Поскольку поддержка неполная, компонент обязательно должен иметь работающий вид по умолчанию — Style Queries используются как дополнительный уровень темизации, а не как единственный способ передать тему.

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

> **Рекомендация для продакшена:** до тех пор, пока Firefox не добавит поддержку, критичную тематизацию (например, переключение светлой/тёмной темы всего приложения) надёжнее реализовывать через `@media (prefers-color-scheme)` или через классы/атрибуты на корневом элементе в сочетании с обычными Custom Properties — а Style Queries применять там, где деградация до отсутствия темизации в Firefox приемлема.

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

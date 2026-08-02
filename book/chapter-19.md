# Глава 19. Изоляция стилей и компонентная архитектура

> Если в начале развития веба CSS воспринимался как единая глобальная таблица стилей, то современная платформа предоставляет полноценный набор инструментов для построения независимых компонентов. Изоляция больше не ограничивается соглашениями об именовании классов или возможностями отдельных фреймворков. Современный CSS включает собственные механизмы ограничения области действия правил, управления каскадом и организации публичного API компонентов.

Сегодня разработчик может выбирать между несколькими уровнями изоляции — от локального каскада до полностью инкапсулированного Shadow DOM — в зависимости от архитектуры проекта.

> Эта глава объединяет архитектурные возможности современного CSS. Она опирается на каскадные слои (глава 18) и показывает, как строить масштабируемые компонентные системы без чрезмерной зависимости от инструментов сборки.

---

## 19.1 Уровни изоляции в современном CSS

### Четыре уровня изоляции

Современная платформа предлагает несколько принципиально различных способов ограничения области действия стилей.

Можно выделить четыре уровня:

```text
1. Локальный каскад (@scope)
  ↓
Изоляция через CSS (браузерный)
  ↓
2. Локальная область имён (CSS Modules)
  ↓
Изоляция через сборку (инструменты)
  ↓
3. Инкапсуляция браузером (Shadow DOM)
  ↓
Полная изоляция (браузерный)
  ↓
4. Архитектурная изоляция (Cascade Layers + Tokens)
  ↓
Изоляция через архитектуру
```

### Сравнение уровней

| Уровень | Механизм | Браузер | Сборка | Изоляция |
|---------|----------|---------|--------|----------|
| **@scope** | CSS | ✅ | ❌ | Селекторы |
| **CSS Modules** | Переименование | ❌ | ✅ | Имена классов |
| **Shadow DOM** | DOM-дерево | ✅ | ❌ | Всё |
| **Архитектурная** | Каскад + токены | ✅ | ❌ | Ответственность |

---

## 19.2 `@scope` — локальный каскад нового поколения

### Принцип работы

Появление `@scope` стало одной из важнейших возможностей CSS последних лет.

В отличие от CSS Modules, правило не переименовывает классы. В отличие от Shadow DOM, оно не создаёт отдельное дерево. Оно лишь ограничивает действие обычного CSS определённой частью документа.

```css
@scope (.card) {
  h2 {
    color: var(--accent);
  }
  
  p {
    line-height: 1.6;
  }
}
```

### Особенности

```text
Отсутствует компиляция
  ↓
Браузер понимает @scope напрямую
  ↓
Сохраняется каскад
  ↓
Вложенные стили работают естественно
  ↓
Все современные селекторы
  ↓
:is(), :where(), :has() работают внутри
  ↓
Cascade Layers
  ↓
@scope работает внутри @layer
  ↓
Container Queries
  ↓
@scope можно комбинировать с @container
```

### Доступ к корню

```css
@scope (.card) {
  /* :scope ссылается на .card */
  :scope {
    border: 1px solid var(--border);
    border-radius: var(--radius-lg);
  }
  
  /* Выход за пределы области (пограничный случай) */
  .content :scope {
    /* :scope здесь — .card, не .content */
    background: var(--surface-raised);
  }
}
```

### Вложенность и @scope

```css
@scope (.card) {
  & .header {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  
  &:hover {
    transform: translateY(-2px);
  }
}
```

### Когда использовать @scope

`@scope` особенно хорошо подходит для:

```text
Библиотек компонентов
  ↓
Независимые компоненты без сборки
  ↓
Документации
  ↓
Стили только внутри компонента
  ↓
CMS и сайтов без сложной сборки
  ↓
Простота и надёжность
```

---

## 19.3 CSS Modules и современные инструменты сборки

### Как работают CSS Modules

Несмотря на появление `@scope`, CSS Modules остаются одним из наиболее распространённых решений.

```css
/* Button.module.css */
.button {
  padding: 0.5rem 1rem;
  background: var(--color-primary);
  color: white;
  border-radius: var(--radius-md);
}

.button-primary {
  background: var(--color-primary);
}
```

```javascript
// Button.jsx
import styles from './Button.module.css';

function Button({ variant, children }) {
  return (
    <button className={`${styles.button} ${styles[variant]}`}>
      {children}
    </button>
  );
}
```

### Почему CSS Modules остаются популярны

```text
Работает во всех сборщиках
  ↓
Vite, Webpack, Turbopack, Parcel
  ↓
Совместимость с React
  ↓
Работает с существующей экосистемой
  ↓
Простота
  ↓
Не требует изучения нового синтаксиса
```

### Ограничения CSS Modules

Важно понимать, что CSS Modules решают только одну задачу — уникальность имён.

```text
CSS Modules
  ↓
Решает: уникальность имён классов
  ↓
Не решает: управление каскадом
  ↓
Не решает: управление специфичностью
  ↓
Не решает: архитектуру проекта
```

### Комбинация с современным CSS

Именно поэтому CSS Modules всё чаще комбинируют с:

```text
Cascade Layers
  ↓
Управление приоритетами
  ↓
Design Tokens
  ↓
Единый язык дизайна
  ↓
@scope
  ↓
Локальный каскад
  ↓
CSS Custom Properties
  ↓
Динамические значения
```

```css
/* Component.module.css */
@layer components {
  .button {
    padding: var(--space-md);
    background: var(--color-primary);
    border-radius: var(--radius-md);
    
    &:hover {
      background: var(--color-primary-hover);
    }
  }
}
```

---

## 19.4 Shadow DOM и Web Components

### Полная инкапсуляция

Shadow DOM остаётся единственным механизмом, обеспечивающим настоящую инкапсуляцию.

```html
<my-component>
  #shadow-root
    <style>
      /* Изолированные стили */
      .title {
        font-size: 1.5rem;
        font-weight: bold;
      }
      
      .content {
        padding: 1rem;
      }
    </style>
    <div class="title">Заголовок</div>
    <div class="content">
      <slot>Содержимое</slot>
    </div>
</my-component>
```

### Что изолируется

```text
Внутри Shadow DOM:
  ↓
Селекторы изолированы
  ↓
Структура изолирована
  ↓
Псевдоэлементы изолированы
  ↓
Большинство внешних правил не действуют
```

### Каскад внутри Shadow DOM

Внутри теневого дерева:

```text
Работают собственные стили
  ↓
Существует собственный каскад
  ↓
Собственные Cascade Layers
  ↓
Собственные области @scope
```

```css
/* Внутри Shadow DOM */
@layer reset, components;

@layer components {
  :host {
    display: block;
    padding: var(--space-md);
  }
  
  .title {
    font-size: var(--font-size-xl);
  }
}
```

### Внешнее API

Внешний документ взаимодействует с компонентом только через официальный API.

Основными средствами настройки являются:

```text
CSS Custom Properties
  ↓
:root { --component-color: blue; }
  ↓
::part()
  ↓
my-component::part(button) { ... }
  ↓
:host
  ↓
:host { display: block; }
  ↓
:host-context()
  ↓
:host-context(.dark-theme) { ... }
  ↓
::slotted()
  ↓
::slotted(.icon) { ... }
```

### Пример Web Component

```javascript
class MyButton extends HTMLElement {
  constructor() {
    super();
    const shadow = this.attachShadow({ mode: 'open' });
    shadow.innerHTML = `
      <style>
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
          }
          
          .btn:hover {
            transform: translateY(-2px);
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

```html
<!-- Использование -->
<my-button style="--btn-bg: #e74c3c;">Удалить</my-button>
```

---

## 19.5 Компонентная архитектура в современных фреймворках

### React

Современные проекты обычно используют:

```text
CSS Modules
  ↓
Локальная изоляция имён
  ↓
Tailwind CSS
  ↓
Utility-first подход
  ↓
vanilla-extract
  ↓
Типизированный CSS-in-JS
  ↓
Panda CSS
  ↓
CSS-in-JS с нулевым runtime
  ↓
StyleX
  ↓
Meta-решение для больших проектов
```

**Тренд:** количество runtime CSS-in-JS постепенно уменьшается вследствие распространения React Server Components.

```jsx
// React + CSS Modules
import styles from './Button.module.css';

function Button({ variant, children }) {
  return (
    <button className={`${styles.button} ${styles[variant]}`}>
      {children}
    </button>
  );
}
```

### Vue

Использует Single File Components. Локализация достигается атрибутом `scoped`:

```vue
<template>
  <button class="btn">{{ text }}</button>
</template>

<style scoped>
.btn {
  padding: 0.5rem 1rem;
  background: var(--color-primary);
  color: white;
  border-radius: var(--radius-md);
}
</style>
```

### Svelte

Использует автоматическое ограничение области действия стилей во время компиляции:

```svelte
<script>
  export let text = 'Кнопка';
</script>

<button class="btn">{text}</button>

<style>
  .btn {
    padding: 0.5rem 1rem;
    background: var(--color-primary);
    color: white;
    border-radius: var(--radius-md);
  }
</style>
```

### Angular

Поддерживает три режима View Encapsulation:

```typescript
@Component({
  selector: 'app-button',
  template: `<button class="btn">{{ text }}</button>`,
  styles: [`
    .btn {
      padding: 0.5rem 1rem;
      background: var(--color-primary);
      color: white;
    }
  `],
  encapsulation: ViewEncapsulation.Emulated
})
```

| Режим | Описание |
|-------|----------|
| **Emulated** | Эмуляция Shadow DOM (по умолчанию) |
| **ShadowDom** | Нативный Shadow DOM |
| **None** | Глобальные стили |

---

## 19.6 Публичный API компонентов

### Компонент как модуль

Современный компонент следует рассматривать как модуль с собственным API.

```text
Компонент
  ↓
Публичный API
  ↓
├── Design Tokens
├── CSS Custom Properties
├── CSS Parts
└── Состояния (data-* / ARIA)
```

### Design Tokens

```css
/* Компонент использует токены */
.button {
  --button-color: var(--color-primary);
  --button-radius: var(--radius-md);
  --button-gap: var(--space-xs);
  
  background: var(--button-color);
  border-radius: var(--button-radius);
  gap: var(--button-gap);
}
```

### CSS Custom Properties

Позволяют изменять внешний вид без вмешательства во внутреннюю реализацию:

```css
/* Пользователь может настроить */
.custom-button {
  --button-color: #e74c3c;
  --button-radius: 4px;
}
```

### CSS Parts (Shadow DOM)

Используются в Shadow DOM для стилизации внутренних элементов:

```css
/* Внутри Shadow DOM */
<button part="button">Кнопка</button>
<span part="icon">★</span>
```

```css
/* Снаружи */
my-button::part(button) {
  font-weight: bold;
}

my-button::part(icon) {
  color: gold;
}
```

### Состояния

Компонент публикует состояния через:

```text
Атрибуты
  ↓
<button disabled>...</button>
  ↓
data-атрибуты
  ↓
<button data-state="loading">...</button>
  ↓
ARIA
  ↓
<button aria-expanded="true">...</button>
  ↓
Псевдоклассы
  ↓
:hover, :focus, :active
```

```css
/* Реакция на состояния */
.button[data-state="loading"] {
  opacity: 0.7;
  cursor: wait;
}

.button[data-state="disabled"] {
  opacity: 0.5;
  cursor: not-allowed;
}

.button[aria-expanded="true"] {
  background: var(--color-primary);
}
```

### Контракт компонента

Полный контракт компонента:

```css
/* Компонент ожидает */
@layer components {
  .button {
    /* Токены */
    --button-bg: var(--color-primary);
    --button-color: white;
    --button-padding: var(--space-md);
    --button-radius: var(--radius-md);
    
    /* Базовые стили */
    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: var(--space-xs);
    padding: var(--button-padding);
    background: var(--button-bg);
    color: var(--button-color);
    border: none;
    border-radius: var(--button-radius);
    cursor: pointer;
    
    /* Состояния */
    &:hover {
      background: color-mix(in oklch, var(--button-bg), white 15%);
    }
    
    &:active {
      transform: scale(0.98);
    }
    
    &[data-state="loading"] {
      opacity: 0.7;
      cursor: wait;
    }
    
    &[data-state="disabled"] {
      opacity: 0.5;
      cursor: not-allowed;
    }
    
    /* Варианты */
    &[data-variant="primary"] {
      --button-bg: var(--color-primary);
    }
    
    &[data-variant="danger"] {
      --button-bg: var(--color-danger);
    }
  }
}
```

---

## 19.7 Практические рекомендации

### Выбор уровня изоляции

| Сценарий | Рекомендация |
|----------|--------------|
| Простой сайт без сборки | `@scope` + `@layer` |
| React-проект | CSS Modules + `@layer` + Tokens |
| Vue-проект | Scoped CSS + `@layer` |
| Библиотека компонентов | Shadow DOM + `@layer` + Parts |
| Дизайн-система | `@layer` + Tokens + `@scope` |

### Архитектурные правила

**1. Начинайте с архитектурной изоляции**

```css
/* Базовый уровень — всегда есть */
@layer tokens, base, components, utilities;
```

**2. Используйте @scope для локального каскада**

```css
@scope (.card) {
  /* Стили только внутри карточки */
}
```

**3. Используйте CSS Modules для уникальности имён**

```css
/* В сборке — для уникальности */
.button { ... }
```

**4. Используйте Shadow DOM для полной инкапсуляции**

```javascript
// Для Web Components
this.attachShadow({ mode: 'open' });
```

**5. Проектируйте публичный API компонентов**

```css
/* Через Custom Properties */
--button-bg: var(--color-primary);

/* Через data-атрибуты */
[data-variant="primary"]

/* Через ::part() */
::part(button)
```

**6. Комбинируйте уровни**

```css
/* @layer + @scope + CSS Modules */
@layer components {
  @scope (.card) {
    .button { ... }
  }
}
```

---

## 19.8 Итоги главы

1. **Четыре уровня изоляции:** `@scope` (CSS), CSS Modules (сборка), Shadow DOM (браузер), архитектурная (каскад + токены)

2. **`@scope` — локальный каскад** — ограничивает действие правил без сборки и DOM-изоляции

3. **CSS Modules** — уникальность имён через сборку, но не управляет каскадом

4. **Shadow DOM** — полная инкапсуляция HTML + CSS, собственный каскад

5. **Архитектурная изоляция** — через `@layer`, Custom Properties, низкую специфичность

6. **Публичный API компонентов** — Design Tokens, Custom Properties, `::part()`, состояния

7. **Фреймворки** — React (CSS Modules), Vue (scoped), Svelte (автоматическая), Angular (3 режима)

8. **Комбинация подходов** — `@layer` + `@scope` + CSS Modules + Shadow DOM

---

**Главная мысль:** Современная компонентная архитектура CSS больше не сводится к выбору между CSS Modules и Shadow DOM. Платформа предлагает несколько уровней изоляции, каждый из которых решает собственный класс задач. `@scope` локализует каскад, Cascade Layers управляют архитектурой приоритетов, CSS Modules обеспечивают уникальность имён, а Shadow DOM предоставляет полную инкапсуляцию. Вместе они образуют современную модель проектирования интерфейсов, в которой компоненты обладают чёткими границами ответственности, предсказуемым каскадом и открытым API для настройки без нарушения внутренней реализации.

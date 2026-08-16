# Глава 3. Cascade Layers (`@layer`): архитектура современного каскада

> **Цель главы:** показать, что `@layer` — не просто новая директива CSS, а фундаментальный механизм, изменивший архитектуру каскада. После этой главы читатель должен перестать воспринимать специфичность как основной инструмент управления стилями.

---

## 3.1 Почему появился `@layer`

### История проблемы: как выглядел CSS до появления слоёв

До 2022 года разработчики десятилетиями пытались бороться с самим каскадом:

```text
CSS
  ↓
Specificity
  ↓
!important
  ↓
BEM
  ↓
CSS Modules
  ↓
CSS-in-JS
```

**Почему это происходило:**

1. **Каскад был непредсказуем** — нельзя было гарантировать, что стиль компонента не перебьют стили другого компонента
2. **Специфичность была единственным механизмом управления приоритетом**
3. **Чем сложнее проект, тем сложнее селекторы**
4. **!important использовался как последнее средство, создавая цепную реакцию**

**Реальные проблемы:**

```css
/* Проблема 1: «война специфичности» */
#app .sidebar .menu .item {
  color: blue;
}

.menu-item {
  color: red; /* Не работает, нужно добавлять специфичности */
}

/* Проблема 2: зависимость от структуры DOM */
.nav > ul > li > a {
  color: blue;
}

/* Изменили структуру — сломались стили */
.nav > li > a {
  /* Не работает */
}

/* Проблема 3: цепная реакция !important */
.button {
  background: blue !important;
}

.button-primary {
  background: red !important; /* Уже два !important */
}
```

**Ключевой вывод:** проблема была не в CSS, а в отсутствии архитектурного механизма управления приоритетами. Специфичность — это инструмент **разрешения конфликтов**, а не **архитектурного проектирования**.

### Рождение `@layer`

В 2022 году спецификация CSS Cascading and Inheritance Level 5 представила `@layer` — первый архитектурный механизм управления каскадом.

**Что изменилось:**

```text
До @layer:
Specificity → Order

После @layer:
Layer → Specificity → Order
```

Layer теперь стоит **выше специфичности**. Это полностью меняет подход к организации CSS.

---

## 3.2 Как работает современный каскад

### Полный алгоритм каскада в CSS 2026

```text
1. Origin (Источник)
   ↓
2. Importance (!important)
   ↓
3. Layer (Слой)
   ↓
4. Specificity (Специфичность)
   ↓
5. Scope (Область видимости)
   ↓
6. Order (Порядок)
```

### Детальное объяснение каждого этапа

**1. Origin (Источник)**

Определяет, кто автор стиля:

```
User Agent (браузер)
  ↓
User (пользователь)
  ↓
Author (разработчик)
```

С `!important` порядок инвертируется.

**2. Importance (!important)**

Правила с `!important` имеют приоритет над обычными.

**3. Layer (Слой)**

Определяет архитектурный слой стиля:

```text
Слой 1 (reset)
  ↓
Слой 2 (base)
  ↓
Слой 3 (components)
  ↓
Слой 4 (utilities)
```

Более поздние слои имеют приоритет.

**Важно:** Layer теперь стоит **выше Specificity. Именно это является революцией.**

```css
/* Это правило в слое components */
@layer components {
  .button {
    background: blue;
  }
}

/* Это правило без слоя (unlayered) */
#app .nav .menu .button {
  background: red;
}

/* Побеждает .button из layers.components, 
   потому что Layer имеет приоритет над Specificity */
```

**4. Specificity (Специфичность)**

Вес селектора:

```text
ID → 1-0-0
Class → 0-1-0
Element → 0-0-1
```

**5. Scope (Область видимости)**

`@scope` ограничивает применение правил.

**6. Order (Порядок)**

Последнее объявленное правило.

---

## 3.3 Архитектура Cascade Layers

### Что такое слой

Слой — это **архитектурный уровень** стилей, а не просто группа правил.

```text
Слой ≠ файл
Слой ≠ модуль
Слой = архитектурный уровень
```

**Аналогия:** если CSS-файлы — это комнаты в доме, то слои — это этажи. На первом этаже — фундамент (reset), на втором — базовые стили (base), на третьем — комнаты (components).

### Структура в браузере

```text
Author Styles
  ↓
├── reset (слой 1)
├── base (слой 2)
├── components (слой 3)
├── utilities (слой 4)
  ↓
User Agent Styles
```

### Почему слой не равен файлу

```css
/* Один файл может содержать несколько слоёв */
@layer reset {
  /* Reset стили */
}

@layer base {
  /* Базовые стили */
}

/* Несколько файлов могут принадлежать одному слою */
/* reset.css */
@layer reset {
  /* ... */
}

/* normalize.css */
@layer reset {
  /* ... */
}

/* Один слой может быть распределён по множеству файлов */
```

### Анонимные слои

```css
/* Слой без имени — анонимный слой */
@layer {
  /* Эти стили имеют приоритет над именованными слоями */
  .special {
    background: gold;
  }
}
```

Анонимные слои полезны для быстрых переопределений, но их сложно поддерживать в больших проектах.

---

## 3.4 Специфичность перестаёт быть главным инструментом

### Исторический контекст

До появления `@layer` разработчики были вынуждены создавать сложные селекторы:

```css
/* Типичный селектор в legacy-проекте */
#app .menu li a.active {
  color: blue;
  font-weight: bold;
  padding: 0.5rem 1rem;
}

/* Чтобы переопределить, нужно было ещё больше специфичности */
#app .menu li a.active.primary {
  color: red;
}
```

### Новая философия

С появлением `@layer` сложные селекторы больше не нужны:

```css
@layer components {
  /* Простой селектор */
  .menu-link {
    color: blue;
    font-weight: bold;
    padding: 0.5rem 1rem;
  }

  .menu-link-primary {
    color: red;
  }
}

@layer overrides {
  /* Простой селектор побеждает сложный из другого слоя */
  .menu-link {
    color: green;
  }
}
```

**Важный вывод:** после появления `@layer` простой селектор `.button` может победить сложный `#application nav ul li a.active` без единого `!important`.

### Сравнение подходов

| До @layer                | После @layer              |
| ------------------------ | ------------------------- |
| Увеличение специфичности | Использование слоёв       |
| Сложные селекторы        | Простые селекторы         |
| !important как решение   | Перемещение в нужный слой |
| Борьба с каскадом        | Проектирование каскада    |
| Хаотичная архитектура    | Предсказуемая иерархия    |

---

## 3.5 Unlayered Styles

### Что такое стили вне слоя

Стили, объявленные без `@layer`, считаются **unlayered** (без слоя).

```css
/* Unlayered style */
.button {
  background: blue;
}

@layer components {
  /* Layered style */
  .button {
    background: red;
  }
}

/* Unlayered побеждает, потому что Layer стоит выше Specificity */
```

### Почему это сделано специально

Спецификация CSS определяет, что стили вне слоёв имеют более высокий приоритет, чем стили внутри слоёв:

```text
Layer 1
  ↓
Layer 2
  ↓
Layer N
  ↓
Unlayered ← самый высокий приоритет
```

**Причины:**

1. **Обратная совместимость** — старый код работает как раньше
2. **Миграция** — можно постепенно переносить стили в слои
3. **Переопределение** — unlayered стили служат «последним словом»

### Использование при миграции проектов

```css
/* Шаг 1: существующий код остаётся unlayered */
.header {
  background: #333;
  color: white;
}

/* Шаг 2: новые стили добавляются в слои */
@layer components {
  .card {
    background: white;
    border-radius: 8px;
  }
}

/* Шаг 3: постепенный перенос старого кода в слои */
@layer base {
  .header {
    background: #333;
    color: white;
  }
}

/* Шаг 4: все стили в слоях, unlayered только для переопределений */
@layer overrides {
  /* Специальные случаи */
}
```

### Unlayered vs Anonymous Layer

```css
/* Unlayered — самый высокий приоритет */
.button {
  background: blue;
}

/* Anonymous Layer — имеет приоритет над именованными слоями */
@layer {
  .button {
    background: red;
  }
}

@layer components {
  .button {
    background: green; /* Самый низкий приоритет */
  }
}
```

---

## 3.6 `!important` внутри Cascade Layers

### Самая неожиданная особенность

Внутри слоёв `!important` ведёт себя неожиданно:

```text
Обычные правила
  ↓
последний слой выигрывает
```

Но:

```text
!important
  ↓
первый слой выигрывает
```

### Почему спецификация инвертирует порядок

**Обычные правила:** последний слой побеждает (естественный порядок).

**!important:** первый слой побеждает (инвертированный порядок).

**Причина:** `!important` предназначен для **критических стилей**, которые не должны быть переопределены. Если бы последний слой выигрывал с `!important`, это создало бы цепную реакцию `!important` в каждом слое.

### Пример

```css
@layer reset {
  .button {
    background: blue !important; /* Побеждает */
  }
}

@layer components {
  .button {
    background: red !important;
  }
}

@layer overrides {
  .button {
    background: green !important;
  }
}

/* Побеждает .button из reset (самый первый слой) */
```

### Практические правила

1. **Избегайте `!important` в слоях** — это нарушает предсказуемость
2. Если `!important` неизбежен, помещайте его в самый ранний слой (например, `reset`)
3. В большинстве случаев `!important` можно заменить слоями

---

## 3.7 `revert-layer`

### Современное ключевое слово

`revert-layer` — одно из самых недооценённых нововведений CSS.

### Сравнение всех ключевых слов

| Ключевое слово | Действие                                   |
| -------------- | ------------------------------------------ |
| `inherit`      | Наследует от родителя                      |
| `initial`      | Возвращает начальное значение              |
| `unset`        | Inherit или initial                        |
| `revert`       | Возвращает значение до стилей разработчика |
| `revert-layer` | Возвращает значение до текущего слоя       |

### Практический пример

```css
@layer base {
  .text {
    color: #333;
    font-size: 16px;
    line-height: 1.5;
  }
}

@layer components {
  .text-highlight {
    color: blue;
    font-size: revert-layer; /* Возвращает 16px из base */
    line-height: revert-layer; /* Возвращает 1.5 из base */
  }
}

@layer overrides {
  .text-highlight-important {
    color: red;
    font-size: revert-layer; /* Возвращает 16px из base */
    line-height: revert-layer; /* Возвращает 1.5 из base */
  }
}
```

### Когда использовать `revert-layer`

1. **Частичное переопределение** — хотим изменить только одно свойство
2. **Откат внутри слоя** — возврат к значению из предыдущего слоя
3. **Сохранение архитектуры** — не нарушаем иерархию слоёв

### Почему `revert-layer` невозможно заменить

```css
/* ❌ Не работает: unset сбрасывает всё */
.text-highlight {
  color: blue;
  font-size: unset; /* Сбрасывает, а не возвращает */
  line-height: unset;
}

/* ✅ Работает: revert-layer */
.text-highlight {
  color: blue;
  font-size: revert-layer;
  line-height: revert-layer;
}
```

---

## 3.8 Импорт библиотек через Layer

### Одно из лучших применений

```css
/* Импорт библиотек в собственные слои */
@import url('normalize.css') layer(reset);
@import url('bootstrap.css') layer(framework);
@import url('tailwind.css') layer(utilities);
```

### Почему это гениально

До появления `@layer`:

```css
/* Bootstrap переопределял кастомные стили */
@import 'bootstrap.css';

/* Приходилось повышать специфичность */
.custom .button {
  /* сложный селектор */
}
```

После появления `@layer`:

```css
/* Все библиотеки в своих слоях */
@import 'bootstrap.css' layer(framework);
@import 'tailwind.css' layer(utilities);

/* Кастомные стили в собственном слое */
@layer components {
  .button {
    /* Простой селектор побеждает Bootstrap */
  }
}
```

### Порядок импорта

```css
@import 'reset.css' layer(reset); /* 1 */
@import 'tokens.css' layer(tokens); /* 2 */
@import 'base.css' layer(base); /* 3 */
@import 'bootstrap.css' layer(framework); /* 4 */
@import 'components.css' layer(components); /* 5 */
@import 'utilities.css' layer(utilities); /* 6 */
@import 'overrides.css'; /* Unlayered — переопределяет всё */
```

### Работа с вендорскими библиотеками

```css
/* Библиотека может быть импортирована в слой */
@import 'some-library.css' layer(vendor);

/* Её стили не загрязняют проект */
@layer components {
  .custom-button {
    background: blue;
    /* Просто переопределяет стили из vendor */
  }
}
```

---

## 3.9 Архитектура современного CSS-проекта

### Рекомендуемая структура слоёв

```text
reset
  ↓
tokens
  ↓
base
  ↓
theme
  ↓
layout
  ↓
components
  ↓
utilities
  ↓
overrides
```

### Детальное описание каждого слоя

```css
/* 1. RESET — нормализация браузерных стилей */
@layer reset {
  /* Normalize.css, Sanitize.css или кастомный reset */
  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }
}

/* 2. TOKENS — дизайн-токены */
@layer tokens {
  :root {
    --color-primary: #3498db;
    --color-secondary: #2ecc71;
    --space-base: 1rem;
    --radius-base: 8px;
  }
}

/* 3. BASE — базовые стили элементов */
@layer base {
  body {
    font-family: system-ui, sans-serif;
    line-height: 1.5;
    color: var(--color-text);
    background: var(--color-bg);
  }
  h1,
  h2,
  h3 {
    font-weight: 700;
    line-height: 1.2;
  }
}

/* 4. THEME — темы приложения */
@layer theme {
  @media (prefers-color-scheme: dark) {
    :root {
      --color-bg: #1a1a1a;
      --color-text: #f0f0f0;
    }
  }
}

/* 5. LAYOUT — структура страниц */
@layer layout {
  .container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 var(--space-base);
  }
  .grid {
    display: grid;
    gap: var(--space-base);
  }
}

/* 6. COMPONENTS — переиспользуемые компоненты */
@layer components {
  .button {
    display: inline-flex;
    padding: 0.5rem 1rem;
    border-radius: var(--radius-base);
    background: var(--color-primary);
    color: white;
  }
  .card {
    padding: var(--space-base);
    background: white;
    border-radius: var(--radius-base);
  }
}

/* 7. UTILITIES — утилиты */
@layer utilities {
  .text-center {
    text-align: center;
  }
  .mt-2 {
    margin-top: 2rem;
  }
  .flex {
    display: flex;
  }
}

/* 8. OVERRIDES — последнее слово */
@layer overrides {
  .button-danger {
    background: #e74c3c;
  }
}
```

### Почему именно такой порядок

```text
Reset → убираем браузерные стили
  ↓
Tokens → определяем переменные
  ↓
Base → базовые стили элементов
  ↓
Theme → темы (используют токены)
  ↓
Layout → структура (использует base, theme)
  ↓
Components → компоненты (используют всё выше)
  ↓
Utilities → утилиты (переопределяют компоненты)
  ↓
Overrides → последнее слово
```

### Как используют слои популярные решения

**Open Props:**

```css
@import 'https://unpkg.com/open-props' layer(open-props);
```

**Bootstrap:**

```css
@import 'bootstrap.css' layer(bootstrap);
```

**Tailwind:**

```css
@import 'tailwind.css' layer(tailwind);
```

**Material Design:**

```css
@import 'material-components.css' layer(material);
```

---

## 3.10 `@layer` и современные фреймворки

### Angular

Angular использует Shadow DOM и CSS Modules, но `@layer` может использоваться для глобальной архитектуры:

```css
/* styles.css — глобальные стили */
@layer reset, base, angular-components;

@layer angular-components {
  /* Стили для Angular-компонентов */
}
```

### React

```css
/* global.css */
@layer reset, base, react-components, utilities;

@layer react-components {
  /* Стили для React-компонентов */
}
```

### Vue

```css
/* global.css */
@layer reset, base, vue-components;

@layer vue-components {
  /* Стили для Vue-компонентов */
}
```

### Svelte

```css
/* global.css */
@layer reset, base, svelte-components;

@layer svelte-components {
  /* Стили для Svelte-компонентов */
}
```

### Web Components

```css
/* Импорт стилей в Shadow DOM */
@layer reset, components;

@layer components {
  :host {
    display: block;
  }
}
```

### CSS Modules

```css
/* В CSS-модулях слой определяет архитектуру модуля */
@layer module-base, module-components;

@layer module-base {
  .module {
    /* Базовые стили модуля */
  }
}
```

### CSS-in-JS

```javascript
// styled-components, Emotion
const Button = styled.button`
  /* @layer автоматически применяется */
  background: blue;
`;
```

---

## 3.11 `@layer` и Design Tokens

### Архитектура Design System с Layers

```text
Design Tokens (@property, Custom Properties)
  ↓
Theme Layer (темные/светлые темы)
  ↓
Components Layer (компоненты)
  ↓
Utilities Layer (утилиты)
```

### Совместная работа @property, Custom Properties и @layer

```css
/* 1. Токены с типизацией */
@property --color-primary {
  syntax: '<color>';
  inherits: true;
  initial-value: #3498db;
}

@property --space-base {
  syntax: '<length>';
  inherits: false;
  initial-value: 1rem;
}

/* 2. Токены в слое */
@layer tokens {
  :root {
    --color-primary: #3498db;
    --space-base: 1rem;
    --radius-base: 8px;
  }
}

/* 3. Тема */
@layer theme {
  @media (prefers-color-scheme: dark) {
    :root {
      --color-primary: #4a9eff;
      --color-bg: #1a1a1a;
    }
  }
}

/* 4. Компоненты используют токены */
@layer components {
  .button {
    background: var(--color-primary);
    padding: var(--space-base);
    border-radius: var(--radius-base);
  }
}
```

### Преимущества

1. **Разделение ответственности** — токены в одном слое, компоненты в другом
2. **Типизация** — `@property` добавляет безопасность
3. **Темизация** — темы переопределяют токены
4. **Масштабируемость** — легко добавлять новые компоненты

---

## 3.12 Progressive Enhancement

### Что делать, если Layer не поддерживается

На момент 2026 года `@layer` поддерживается во всех современных браузерах:

```text
Chrome: 99+
Firefox: 97+
Safari: 15.4+
Edge: 99+
```

Поддержка практически универсальна, и на практике фиче-детект для `@layer` сегодня почти никогда не требуется.

### Использование `@supports`

Важный технический нюанс: конструкция `@supports (свойство: значение)` проверяет поддержку пары «свойство/значение», а не поддержку самого at-rule. Чтобы корректно проверить поддержку именно at-rule — например, `@layer`, `@property` или `@starting-style`, — используется отдельная функция `at-rule()` из CSS Conditional Rules Module:

```css
/* Правильная проверка поддержки самого @layer */
@supports at-rule(@layer) {
  @layer components {
    .button {
      /* Современные стили */
    }
  }
}

@supports not at-rule(@layer) {
  .button {
    /* Fallback-стили для браузеров без поддержки @layer */
  }
}
```

Форма `@supports (layer)`, которую иногда можно встретить в старых статьях, не является корректной проверкой поддержки at-rule и может вести себя непредсказуемо в разных браузерах — её стоит избегать.

На практике, поскольку `@layer` уже входит в Baseline Widely available, фолбэк через `@supports not at-rule(@layer)` нужен лишь в проектах с жёсткими требованиями к устаревшим браузерам. Для большинства современных проектов достаточно писать unlayered-стили как естественный fallback: браузеры без поддержки `@layer` просто проигнорируют это правило и продолжат применять остальные стили.

### Философия Progressive Enhancement

```text
HTML → работает всегда
  ↓
CSS (базовые стили) → работает всегда
  ↓
CSS (@layer) → улучшает архитектуру
  ↓
JavaScript → добавляет интерактивность
```

### Почему Layer хорошо вписывается в Progressive Enhancement

1. **Базовые стили не зависят от `@layer`** — структура сохраняется
2. **`@layer` только улучшает организацию** — не влияет на внешний вид
3. **Браузеры без поддержки игнорируют неизвестный at-rule** — стили внутри блока `@layer` при этом браузером, не понимающим `@layer`, не применяются, поэтому осознанный fallback (unlayered-версия или `@supports not at-rule(@layer)`) обязателен, если нужно поддерживать действительно старые браузеры
4. **Простое падение** — unlayered-стили служат естественным fallback для актуальных браузеров, где `@layer` уже входит в Baseline
---

## 3.13 Лучшие практики

### Практические рекомендации

**1. Объявляйте все слои в начале проекта**

```css
/* Список всех слоёв проекта */
@layer reset, tokens, base, theme, layout, components, utilities, overrides;
```

**2. Используйте слой reset только для базовой нормализации**

```css
@layer reset {
  /* Только reset-стили */
  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }
  /* Без кастомных стилей! */
}
```

**3. Храните дизайн-токены отдельно от компонентов**

```css
@layer tokens {
  :root {
    /* Только переменные */
  }
}

@layer components {
  /* Используют токены, не определяют их */
}
```

**4. Избегайте !important — используйте слои**

```css
/* ❌ Плохо */
.button {
  background: blue !important;
}

/* ✅ Хорошо */
@layer overrides {
  .button {
    background: blue;
  }
}
```

**5. Импортируйте сторонние библиотеки в собственные слои**

```css
@import 'bootstrap.css' layer(bootstrap);
@import 'tailwind.css' layer(tailwind);
```

**6. Используйте простые селекторы — архитектура слоёв важнее высокой специфичности**

```css
/* ❌ Плохо */
#app .nav .menu .item .link {
  color: blue;
}

/* ✅ Хорошо */
.nav-link {
  color: blue;
}
```

**7. Оставляйте стили вне слоёв только для осознанных переопределений или при постепенной миграции legacy-кода**

```css
/* Unlayered — только для особых случаев */
.special-case {
  background: gold;
}
```

### Антипаттерны

**1. Слишком много слоёв**

```css
/* ❌ Плохо */
@layer a, b, c, d, e, f, g, h, i, j, k, l, m, n, o, p;

/* ✅ Хорошо */
@layer reset, tokens, base, components, utilities, overrides;
```

**2. Сложные селекторы внутри слоёв**

```css
/* ❌ Плохо */
@layer components {
  #app .sidebar .menu li a {
    color: blue;
  }
}

/* ✅ Хорошо */
@layer components {
  .nav-link {
    color: blue;
  }
}
```

**3. !important внутри слоёв**

```css
/* ❌ Плохо */
@layer components {
  .button {
    background: blue !important;
  }
}

/* ✅ Хорошо */
@layer overrides {
  .button {
    background: blue;
  }
}
```

---

## 3.14 Итоги главы

1. **`@layer` появился в 2022 году как архитектурный механизм каскада**

2. **Современный каскад:** Origin → Importance → Layer → Specificity → Scope → Order

3. **Layer стоит выше специфичности** — это ключевая революция

4. **Слой — это архитектурный уровень, а не файл или модуль**

5. **Специфичность перестала быть главным инструментом** — теперь используются слои

6. **Unlayered стили имеют самый высокий приоритет** — для обратной совместимости и миграции

7. **`!important` внутри слоёв работает инвертированно** — первый слой побеждает

8. **`revert-layer` возвращает значение до текущего слоя** — незаменимый инструмент

9. **Импорт библиотек через `layer()`** — библиотеки перестают загрязнять проект

10. **Архитектура:** Reset → Tokens → Base → Theme → Layout → Components → Utilities → Overrides

11. **Все фреймворки поддерживают `@layer`** — Angular, React, Vue, Svelte

12. **Design Tokens + `@property` + `@layer`** — идеальная архитектура

13. **Progressive Enhancement** — `@layer` отлично вписывается в философию

14. **Best Practices** — объявляйте слои в начале, используйте простые селекторы, избегайте !important

---

**Главная мысль:** До появления `@layer` управление приоритетами было побочным эффектом специфичности селекторов. Чем сложнее становился проект, тем больше разработчики были вынуждены усложнять селекторы, использовать `!important` и внедрять архитектурные соглашения вроде BEM, CSS Modules или CSS-in-JS для борьбы с конфликтами стилей.

Cascade Layers изменили саму модель каскада. Впервые разработчик получил возможность декларативно определять архитектурные уровни стилей независимо от специфичности селекторов. Приоритет перестал быть следствием сложности селектора и стал частью структуры приложения.

В сочетании с `@scope`, Custom Properties, `@property`, современными селекторами и системой Baseline каскад превратился из исторического ограничения CSS в один из его главных архитектурных инструментов. Современный CSS больше не требует борьбы с каскадом — он предоставляет средства для его осознанного проектирования. Именно поэтому `@layer` следует рассматривать не как очередную новую директиву языка, а как один из ключевых шагов в превращении CSS в полноценную платформу разработки пользовательских интерфейсов.

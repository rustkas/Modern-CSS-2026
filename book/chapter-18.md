# Глава 18. Cascade Layers и архитектура современных дизайн-систем

> Каскадные слои (`@layer`) стали одним из самых значимых архитектурных изменений CSS за последние годы. Если раньше приоритет правил определялся сложной комбинацией происхождения стилей, специфичности и порядка объявления, то теперь разработчик может проектировать каскад как осознанную систему уровней ответственности.

Однако сами по себе слои не являются архитектурой. Они становятся по-настоящему ценными только в сочетании с современными подходами к организации CSS: Design Tokens, компонентной моделью, CUBE CSS, ITCSS, CSS Nesting и Custom Properties.

В этой главе рассматривается не механизм работы `@layer` (он подробно разобран в главе 3), а способы построения масштабируемой архитектуры стилей для современных приложений.

---

## 18.1 Cascade Layers как уровень архитектуры

### От конфликтов к архитектуре

Главное изменение, которое принесли слои, заключается в переносе контроля над каскадом с отдельных селекторов на целые категории стилей.

```text
До @layer:
  ↓
Управление через специфичность
  ↓
"Какой селектор сильнее?"
  ↓
Войны специфичности
  ↓
После @layer:
  ↓
Управление через архитектуру
  ↓
"Какой уровень имеет больший приоритет?"
  ↓
Предсказуемая иерархия
```

**Теперь разработчик управляет не тем, какой селектор сильнее, а тем, какой уровень системы имеет больший приоритет.**

**Это меняет сам подход к проектированию CSS. Каскад превращается из побочного эффекта языка в архитектурный инструмент.**

```text
Каскад как побочный эффект:
  ↓
Стили конфликтуют
  ↓
Нужно управлять специфичностью
  ↓
Сложно, непредсказуемо
  ↓
Каскад как архитектура:
  ↓
Стили организованы по уровням
  ↓
Приоритет определён заранее
  ↓
Просто, предсказуемо
```

---

## 18.2 Проектирование слоёв

### Стандартная структура

Практика последних лет привела к достаточно устойчивой структуре современных проектов.

```css
@layer reset, tokens, base, patterns, components, utilities, overrides;
```

### Каждый слой имеет собственную ответственность

**1. Reset — нормализация браузерных стилей**

Здесь располагаются:

- Modern CSS Reset
- Normalize
- Правила доступности
- Базовые гарантии платформы

```css
@layer reset {
  *,
  *::before,
  *::after {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }
  
  :where(img, picture, video, canvas, svg) {
    display: block;
    max-width: 100%;
  }
}
```

**2. Tokens — единый источник проектных значений**

Обычно здесь объявляются:

- Цвета
- Размеры
- Интервалы
- Типографика
- Радиусы
- Длительности анимаций

Практически весь слой состоит из Custom Properties.

```css
@layer tokens {
  :root {
    /* Цвета */
    --color-primary: oklch(65% 0.22 255);
    --color-surface: light-dark(white, #1a1a1a);
    --color-text: light-dark(#1a1a1a, white);
    
    /* Размеры */
    --space-xs: 0.25rem;
    --space-sm: 0.5rem;
    --space-md: 1rem;
    --space-lg: 1.5rem;
    --space-xl: 2rem;
    
    /* Радиусы */
    --radius-sm: 4px;
    --radius-md: 8px;
    --radius-lg: 12px;
    
    /* Типографика */
    --font-family: system-ui, -apple-system, sans-serif;
    --font-size-sm: 0.875rem;
    --font-size-base: 1rem;
    --font-size-lg: 1.125rem;
    --font-size-xl: 1.25rem;
  }
}
```

**3. Base — оформление HTML как языка документа**

Например:

- Заголовки
- Ссылки
- Списки
- Таблицы
- Формы

Этот слой отвечает за внешний вид документа ещё до появления компонентов.

```css
@layer base {
  body {
    font-family: var(--font-family);
    font-size: var(--font-size-base);
    line-height: 1.5;
    color: var(--color-text);
    background: var(--color-surface);
  }
  
  h1, h2, h3, h4, h5, h6 {
    font-weight: 700;
    line-height: 1.2;
    text-wrap: balance;
  }
  
  a {
    color: var(--color-primary);
    text-decoration: underline;
    text-underline-offset: 2px;
  }
  
  a:hover {
    text-decoration: none;
  }
}
```

**4. Patterns — повторяемые шаблоны**

Современные проекты всё чаще выделяют промежуточный уровень между Base и Components.

Здесь находятся повторяемые шаблоны из CUBE CSS:

- Stack (вертикальный стек)
- Cluster (горизонтальный кластер)
- Grid (сетка)
- Sidebar (сайдбар)
- Center (центрирование)
- Switcher (переключатель)

Подход пришёл из CUBE CSS и существенно уменьшает количество компонентного кода.

```css
@layer patterns {
  .stack {
    display: flex;
    flex-direction: column;
    gap: var(--space-md);
  }
  
  .cluster {
    display: flex;
    flex-wrap: wrap;
    gap: var(--space-md);
    align-items: center;
  }
  
  .grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: var(--space-md);
  }
  
  .sidebar {
    display: grid;
    grid-template-columns: 1fr 2fr;
    gap: var(--space-lg);
  }
  
  .center {
    display: grid;
    place-items: center;
    min-height: 100vh;
  }
}
```

**5. Components — основной слой пользовательского интерфейса**

Здесь располагаются:

- Button
- Card
- Dialog
- Navigation
- Tabs
- Menu
- Accordion

Каждый компонент представляет собой законченную систему.

```css
@layer components {
  .button {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: var(--space-xs);
    padding: 0.5rem 1rem;
    border: 1px solid transparent;
    border-radius: var(--radius-md);
    font-family: inherit;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.2s ease;
    
    &:hover {
      transform: translateY(-1px);
    }
    
    &:active {
      transform: scale(0.98);
    }
    
    &[data-variant="primary"] {
      background: var(--color-primary);
      color: white;
      
      &:hover {
        background: color-mix(in oklch, var(--color-primary), white 15%);
      }
    }
  }
  
  .card {
    padding: var(--space-md);
    background: var(--color-surface);
    border-radius: var(--radius-lg);
    box-shadow: 0 2px 8px rgba(0,0,0,0.05);
    
    & .card-title {
      font-size: var(--font-size-xl);
      font-weight: 700;
      margin-bottom: var(--space-sm);
    }
    
    & .card-content {
      color: var(--color-text-secondary);
    }
  }
}
```

**6. Utilities — атомарные классы**

Например:

- `.hidden`
- `.sr-only`
- `.mt-auto`
- `.flex`
- `.text-center`

Благодаря расположению в конце они практически всегда выигрывают каскад без повышения специфичности.

```css
@layer utilities {
  .hidden { display: none; }
  .sr-only { position: absolute; width: 1px; height: 1px; overflow: hidden; }
  .flex { display: flex; }
  .grid { display: grid; }
  .text-center { text-align: center; }
  .mt-auto { margin-top: auto; }
  .gap-sm { gap: var(--space-sm); }
  .gap-md { gap: var(--space-md); }
  .gap-lg { gap: var(--space-lg); }
}
```

**7. Overrides — минимальный слой исключений**

Его задача — локализовать временные решения и облегчить их последующее удаление. Хорошо спроектированная система использует этот слой редко.

```css
@layer overrides {
  /* Исправление бага в конкретном компоненте */
  .legacy-component .button {
    background: var(--color-danger);
  }
  
  /* Временное решение до рефакторинга */
  .temp-fix {
    position: relative;
    z-index: 10;
  }
}
```

---

## 18.3 Cascade Layers и современные методологии

### Универсальный механизм реализации

Cascade Layers не заменяют существующие методологии организации CSS. Наоборот, они становятся механизмом их реализации.

| Методология | Как реализуется через `@layer` |
|-------------|-------------------------------|
| **ITCSS** | Каждый уровень пирамиды соответствует отдельному слою |
| **CUBE CSS** | Composition, Utilities и Blocks естественно распределяются по слоям |
| **Design Systems** | Tokens, Components и Utilities становятся независимыми уровнями |
| **Utility-first** | Utilities получают максимальный приоритет без роста специфичности |

### Пример: ITCSS через `@layer`

```text
ITCSS (Inverted Triangle CSS)
  ↓
Settings → Tokens
  ↓
Tools → (вычисляемые значения)
  ↓
Generic → Reset + Base
  ↓
Elements → Base HTML
  ↓
Objects → Patterns
  ↓
Components → Components
  ↓
Utilities → Utilities
```

### Пример: CUBE CSS через `@layer`

```text
CUBE CSS
  ↓
Composition → Patterns
  ↓
Utilities → Utilities
  ↓
Blocks → Components
  ↓
Exceptions → Overrides
```

**Таким образом, слои становятся универсальным механизмом реализации различных архитектурных подходов.**

---

## 18.4 Интеграция с современным CSS

### Комбинация технологий

К 2026 году `@layer` редко используется изолированно. Наиболее типичная комбинация выглядит следующим образом:

```text
Layer (приоритет)
  ↓
Design Tokens (значения)
  ↓
Custom Properties (динамика)
  ↓
CSS Nesting (структура)
  ↓
Container Queries (адаптивность)
  ↓
Components (интерфейс)
```

### Полный пример

```css
@layer reset, tokens, base, patterns, components, utilities, overrides;

@layer tokens {
  :root {
    --color-primary: oklch(65% 0.22 255);
    --space-md: 1rem;
    --radius-md: 8px;
  }
}

@layer components {
  .card {
    /* Nesting */
    & .title {
      font-size: var(--font-size-xl);
    }
    
    /* Container Query */
    @container (max-width: 400px) {
      padding: var(--space-sm);
      
      & .title {
        font-size: var(--font-size-md);
      }
    }
    
    /* Custom Properties */
    --card-padding: var(--space-md);
    padding: var(--card-padding);
    background: var(--color-surface);
    border-radius: var(--radius-md);
  }
}

@layer utilities {
  .card-compact {
    --card-padding: var(--space-sm);
  }
}
```

**Каждый уровень решает собственную задачу:**

```text
Слой → определяет приоритет
  ↓
Токены → определяют значения
  ↓
Nesting → организует структуру
  ↓
Container Queries → делают компонент адаптивным
  ↓
Компонент → объединяет всё в единое целое
```

**Именно такая комбинация стала характерной для современных дизайн-систем.**

---

## 18.5 Работа со сторонними библиотеками

### Контроль над сторонним CSS

Одним из главных преимуществ Cascade Layers стала возможность полностью контролировать сторонний CSS.

```css
/* Импорт библиотек в собственные слои */
@import url("normalize.css") layer(reset);
@import url("bootstrap.css") layer(vendor);
@import url("tailwind.css") layer(utilities);
```

После этого любой слой проекта может предсказуемо переопределять библиотеку.

```css
@layer components {
  .button {
    /* Простой селектор побеждает Bootstrap */
    background: var(--color-primary);
    color: white;
  }
}
```

**Отпадает необходимость:**

- повышать специфичность
- использовать длинные селекторы
- применять `!important`

### Пример с Tailwind

> **Актуально для Tailwind CSS v4** (текущая мажорная версия на 2026 год). Начиная с этой версии Tailwind сам построен на нативных Cascade Layers и подключается одной строкой, а не через отдельные директивы `@tailwind base/components/utilities` из версии 3:

```css
/* Tailwind v4 сам объявляет и заполняет свои слои: 
   theme, base, components, utilities */
@import "tailwindcss";

/* Наши собственные компоненты — в отдельном слое проекта,
   объявленном после слоёв Tailwind, чтобы предсказуемо их переопределять */
@layer components {
  .btn {
    @apply px-4 py-2 rounded font-medium;
    /* Переопределяем цвет */
    background: var(--color-primary);
    color: white;
  }
}

@layer utilities {
  .btn-lg {
    @apply px-6 py-3 text-lg;
  }
}
```

Если нужен более тонкий контроль — например, встроить Tailwind в проект с собственной многослойной архитектурой из раздела 18.2 — Tailwind v4 позволяет импортировать свои части по отдельности, явно указывая слой для каждой:

```css
@layer reset, tailwind-theme, tokens, base, tailwind-utilities, components, overrides;

@import "tailwindcss/theme.css" layer(tailwind-theme);
@import "tailwindcss/utilities.css" layer(tailwind-utilities);

@layer tokens {
  :root {
    --color-primary: oklch(65% 0.22 255);
  }
}

@layer components {
  .btn {
    background: var(--color-primary);
    color: white;
  }
}
```

Такой подход раскрывает главное преимущество нативных слоёв: поскольку с версии 4 Tailwind сам генерирует настоящие CSS `@layer`-блоки (а не собственную, изолированную от остального каскада систему приоритетов, как в Tailwind v3), его слои становятся полноправной частью общей архитектуры проекта — их можно переставлять, комбинировать с `tokens`, `components` и `overrides` проекта и предсказуемо переопределять простыми селекторами, без `!important` и без борьбы со специфичностью.

---

## 18.6 `!important` в современной архитектуре

### Инвертированный порядок

Особенность Cascade Layers состоит в том, что при использовании `!important` порядок слоёв инвертируется.

```text
Обычные правила:
  ↓
Последний слой выигрывает
  ↓
!important:
  ↓
Первый слой выигрывает
```

### Когда использовать `!important`

На практике рекомендуется использовать `!important` исключительно для:

```text
Правил доступности
  ↓
.sr-only { position: absolute !important; }
  ↓
Служебных утилит
  ↓
.hidden { display: none !important; }
  ↓
Аварийных исправлений
  ↓
.critical-fix { color: red !important; }
```

### Почему `!important` почти не нужен

```text
До @layer:
  ↓
!important — единственный способ переопределить высокую специфичность
  ↓
Цепная реакция !important
  ↓
После @layer:
  ↓
Перемещение в нужный слой
  ↓
Простой селектор побеждает сложный
  ↓
!important не нужен
```

**Во всех остальных случаях архитектура слоёв должна полностью устранять необходимость его применения.**

---

## 18.7 Практические рекомендации

### Архитектурные правила

Современная практика показывает несколько устойчивых правил.

**1. Не создавайте большое количество слоёв**

```css
/* ❌ Плохо — слишком много слоёв */
@layer a, b, c, d, e, f, g, h, i, j, k, l, m, n, o, p;

/* ✅ Хорошо — 5-8 слоёв */
@layer reset, tokens, base, components, utilities, overrides;
```

**2. Используйте слои для уровней ответственности, а не для отдельных файлов**

```css
/* ❌ Плохо — слой = файл */
@layer header { ... }
@layer footer { ... }
@layer sidebar { ... }

/* ✅ Хорошо — слой = уровень ответственности */
@layer components {
  .header { ... }
  .footer { ... }
  .sidebar { ... }
}
```

**3. Не смешивайте компоненты и утилиты**

```css
/* ❌ Плохо — утилита внутри компонентов */
@layer components {
  .button { ... }
  .text-center { ... } /* Утилита здесь не место */
}

/* ✅ Хорошо — разделение по слоям */
@layer components {
  .button { ... }
}
@layer utilities {
  .text-center { ... }
}
```

**4. Не помещайте Design Tokens внутрь компонентов**

```css
/* ❌ Плохо — токены в компоненте */
@layer components {
  .button {
    --color-primary: #2563eb; /* Токен не здесь */
    background: var(--color-primary);
  }
}

/* ✅ Хорошо — токены отдельно */
@layer tokens {
  :root {
    --color-primary: #2563eb;
  }
}
@layer components {
  .button {
    background: var(--color-primary);
  }
}
```

**5. Не используйте неслойные стили без веской причины**

```css
/* ❌ Плохо — стили вне слоёв */
.button {
  background: blue; /* Непредсказуемый приоритет */
}

/* ✅ Хорошо — стили в слоях */
@layer components {
  .button {
    background: blue;
  }
}
```

**6. Делайте слой Overrides минимальным**

```css
/* ✅ Хорошо — минимальный оверрайд */
@layer overrides {
  .legacy-fix {
    padding: 0;
  }
}

/* ❌ Плохо — слишком много оверрайдов */
@layer overrides {
  .button { ... }
  .card { ... }
  .nav { ... }
  /* Весь проект в оверрайдах */
}
```

**7. Рассматривайте слои как публичную архитектуру проекта**

```css
/* Слои документируют архитектуру */
@layer 
  reset,      /* Браузерная нормализация */
  tokens,     /* Дизайн-токены */
  base,       /* Базовые HTML-элементы */
  patterns,   /* Повторяемые шаблоны */
  components, /* UI-компоненты */
  utilities,  /* Утилиты */
  overrides;  /* Исключения */
```

---

## 18.8 Cascade Layers и будущее CSS

### Фундамент платформы

Cascade Layers стали фундаментом большинства современных возможностей платформы.

```text
Design Tokens
  ↓
CSS Nesting
  ↓
@scope
  ↓
Container Queries
  ↓
Custom Properties
  ↓
Shadow DOM
  ↓
View Transitions
  ↓
Cascade Layers
```

### Эволюция каскада

```text
CSS 2.1
  ↓
Специфичность
  ↓
CSS 3
  ↓
!important
  ↓
CSS 2022
  ↓
@layer
  ↓
CSS 2026
  ↓
@layer + @scope + Custom Properties
  ↓
Каскад как архитектура
```

**В результате каскад превращается из механизма разрешения конфликтов в средство проектирования архитектуры.**

---

## 18.9 Итоги главы

1. **Cascade Layers — уровень архитектуры** — управление не селекторами, а уровнями системы

2. **Стандартная структура:** Reset → Tokens → Base → Patterns → Components → Utilities → Overrides

3. **Reset** — нормализация браузерных стилей

4. **Tokens** — дизайн-токены (все значения через Custom Properties)

5. **Base** — стили HTML-элементов (документ как язык)

6. **Patterns** — повторяемые шаблоны (Stack, Cluster, Grid, Sidebar)

7. **Components** — UI-компоненты (Button, Card, Dialog)

8. **Utilities** — атомарные классы (с высоким приоритетом)

9. **Overrides** — минимальный слой исключений

10. **Методологии** — ITCSS, CUBE CSS, Design Systems реализуются через `@layer`

11. **Интеграция** — работает с Nesting, Container Queries, Custom Properties

12. **Сторонние библиотеки** — импорт в слои для полного контроля

13. **`!important`** — только для доступности, утилит и критических исправлений

14. **Практические правила:** не более 8 слоёв, разделение ответственности, оверрайды минимальны

---

**Главная мысль:** Cascade Layers стали одним из ключевых элементов современной архитектуры CSS. Они позволяют проектировать каскад как систему уровней ответственности, а не как результат конкуренции селекторов. В сочетании с Design Tokens, компонентным подходом, CSS Nesting и Container Queries слои формируют основу масштабируемых дизайн-систем, в которых приоритет определяется архитектурой, а не случайным порядком правил или специфичностью.

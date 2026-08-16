# Глава 6. Flexbox: алгоритм одномерной композиции

> Несмотря на появление CSS Grid, Flexbox не потерял своей актуальности. Напротив, современная веб-платформа окончательно разделила ответственность между двумя алгоритмами. Grid управляет архитектурой страницы и двумерными макетами, тогда как Flexbox отвечает за композицию компонентов, распределение пространства и адаптацию содержимого вдоль одной оси. Вместе они образуют основу современного декларативного Layout.

---

## 6.1 Почему Flexbox не заменён Grid

### Самое распространённое заблуждение последних лет

Многие разработчики считают, что Grid — это "улучшенный Flexbox" и что после появления Grid Flexbox стал не нужен.

```text
❌ Grid > Flexbox
```

На самом деле:

```text
Grid
  ↓
отвечает за двумерную архитектуру страницы
  ↓
Flexbox
  ↓
отвечает за одномерную композицию компонентов
```

### Разные задачи — разные алгоритмы

| Аспект | Grid | Flexbox |
|--------|------|---------|
| **Измерение** | Двумерное (строки + колонки) | Одномерное (одна ось) |
| **Уровень** | Архитектура страницы | Композиция компонентов |
| **Управление** | Сетка в целом | Отдельные элементы |
| **Распределение** | Треки и области | Свободное пространство |
| **Адаптация** | Автоматическое размещение | Распределение и выравнивание |

### Почему они не конкурируют

```text
Страница
  ↓
Grid (архитектура)
  ↓
  ├── Header
  ├── Sidebar
  ├── Main Content
  │     ├── Flex (композиция карточек)
  │     ├── Flex (тулбар)
  │     ├── Flex (форма)
  │     └── Flex (навигация)
  └── Footer
```

**Grid** отвечает за то, **где** находятся секции.  
**Flexbox** отвечает за то, **как** элементы внутри секций распределены.

### Современный интерфейс использует оба алгоритма одновременно

```css
/* Grid — архитектура страницы */
.page {
  display: grid;
  grid-template-columns: 240px 1fr 280px;
  grid-template-areas: 
    "sidebar header header"
    "sidebar main widget"
    "sidebar footer footer";
  min-height: 100vh;
}

/* Flexbox — композиция внутри секций */
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: var(--space-md);
}

.toolbar {
  display: flex;
  gap: var(--space-sm);
  align-items: center;
}

.card-list {
  display: flex;
  flex-wrap: wrap;
  gap: var(--space-md);
}
```

---

## 6.2 Flex Formatting Context

### Flexbox — это отдельный алгоритм браузера

Flexbox — это не просто набор CSS-свойств. Это отдельный **форматирующий контекст** браузера.

```text
Flex Container
  ↓
Flex Items
  ↓
Main Axis
  ↓
Cross Axis
  ↓
Free Space Distribution
```

### Внутренняя модель Flexbox

**1. Flex Container (гибкий контейнер)**

Родительский элемент с `display: flex`:

```css
.container {
  display: flex; /* Создаёт Flex Formatting Context */
}
```

**2. Flex Items (гибкие элементы)**

Непосредственные дети контейнера:

```html
<div class="container">
  <div>Item 1</div>   <!-- Flex Item -->
  <div>Item 2</div>   <!-- Flex Item -->
  <div>Item 3</div>   <!-- Flex Item -->
</div>
```

**3. Main Axis (главная ось)**

Направление flex-элементов:

```css
.container {
  flex-direction: row;    /* Горизонтальная ось (по умолчанию) */
  flex-direction: column; /* Вертикальная ось */
  flex-direction: row-reverse;
  flex-direction: column-reverse;
}
```

**4. Cross Axis (поперечная ось)**

Перпендикулярное направление:

```text
Main Axis → горизонтальная
Cross Axis → вертикальная
```

**5. Free Space (свободное пространство)**

Пространство, которое нужно распределить между элементами:

```text
Размер контейнера
  ↓
- Сумма размеров элементов
  ↓
= Свободное пространство
  ↓
Распределяется через flex-grow / flex-shrink
```

### Как браузер вычисляет размеры

```text
1. Контейнер определяет размеры
   ↓
2. Основной размер контента (flex-basis)
   ↓
3. Распределение свободного пространства
   ↓
4. Выравнивание на осях
   ↓
5. Финальные размеры
```

### Почему Flexbox нельзя понимать только через свойства

```css
/* Свойства — только интерфейс */
.container {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.item {
  flex: 1;
  min-width: 0;
}
```

**За этими свойствами стоит сложный алгоритм:**

1. **flex: 1** → grow:1, shrink:1, basis:0
2. Браузер рассчитывает базовые размеры
3. Вычисляет свободное пространство
4. Распределяет его согласно flex-grow
5. Проверяет минимальные размеры (min-width)
6. Корректирует при переполнении

---

## 6.3 Алгоритм распределения пространства

### Три ключевых свойства

```css
.item {
  flex-grow: 0;    /* Способность увеличиваться */
  flex-shrink: 1;  /* Способность уменьшаться */
  flex-basis: auto; /* Базовый размер */
}

/* Сокращённая запись */
.item {
  flex: 0 1 auto; /* grow shrink basis */
}

/* Часто используемые значения */
.item {
  flex: 1;    /* grow:1, shrink:1, basis:0 */
  flex: auto; /* grow:1, shrink:1, basis:auto */
  flex: none; /* grow:0, shrink:0, basis:auto */
}
```

### Что происходит внутри браузера

**Шаг 1: Определение базового размера**

```css
.item {
  flex-basis: 100px;  /* Чёткий размер */
  flex-basis: auto;   /* Размер по содержимому */
  flex-basis: 0;      /* Размер игнорируется */
  flex-basis: 25%;    /* Процент от контейнера */
}
```

**Шаг 2: Вычисление свободного пространства**

```text
Свободное пространство = 
  Размер контейнера - 
  Сумма базовых размеров элементов
```

**Шаг 3: Распределение свободного пространства**

Если пространство **положительное** → используем `flex-grow`:

```css
.container {
  display: flex;
  width: 600px;
}

.item1 { flex-grow: 1; }
.item2 { flex-grow: 2; }
.item3 { flex-grow: 1; }

/* item1 получает 1/4 пространства */
/* item2 получает 2/4 пространства */
/* item3 получает 1/4 пространства */
```

Если пространство **отрицательное** → используем `flex-shrink`:

```css
.container {
  display: flex;
  width: 300px;
}

.item {
  flex-basis: 150px;
  flex-shrink: 1; /* Все сжимаются одинаково */
}
```

### Почему `flex: 1` не означает `width: 100%`

```css
/* ❌ Неправильное понимание */
.item {
  flex: 1; /* "Занимает всё доступное пространство" */
  width: 100%; /* Не нужно! */
}

/* ✅ Правильное понимание */
.item {
  flex: 1; /* Получает пропорцию свободного пространства */
  /* Ширина = базовый размер + доля свободного пространства */
}
```

**Реальный пример:**

```css
.container {
  display: flex;
  gap: 10px;
}

.item1 {
  flex: 1; /* 33.33% + базовый размер */
  background: red;
}

.item2 {
  flex: 2; /* 66.66% + базовый размер */
  background: blue;
}

/* Итоговая ширина зависит от контента и свободного пространства */
```

### flex-grow vs flex-shrink

| Свойство | Когда работает | Что делает |
|----------|---------------|------------|
| `flex-grow` | Есть свободное пространство | Увеличивает элементы |
| `flex-shrink` | Не хватает пространства | Уменьшает элементы |

**Важное правило:**

```text
flex-grow работает только при положительном свободном пространстве
flex-shrink работает только при отрицательном свободном пространстве
```

---

## 6.4 Intrinsic Sizing и Flexbox

### Современный Flexbox построен вокруг содержимого

```text
Фиксированные размеры
  ↓
min-width, max-width, width
  ↓
Intrinsic Sizing
  ↓
min-content, max-content, fit-content()
```

### Ключевые ключевые слова

**min-content** — минимальный размер по содержимому:

```css
.item {
  width: min-content;
  /* Элемент становится минимально возможной ширины */
  /* Для текста — ширина самого длинного слова */
}
```

**max-content** — максимальный размер по содержимому:

```css
.item {
  width: max-content;
  /* Элемент становится шириной по содержимому */
  /* Без переносов текста */
}
```

**fit-content()** — адаптация с ограничением:

```css
.item {
  width: fit-content(300px);
  /* Элемент по содержимому, но не больше 300px */
}
```

### Практическое применение

```css
/* Адаптивная кнопка */
.button {
  width: fit-content;
  padding: 0.5rem 1rem;
  /* Кнопка — по содержимому, а не растянута */
}

/* Заголовок с минимальной шириной */
.title {
  width: min-content;
  /* Не даёт заголовку слишком сильно растягиваться */
}

/* Контейнер, который не может расширяться */
.sidebar {
  width: minmax(200px, max-content);
  /* От 200px до содержимого */
}
```

### Почему контент определяет размеры

```css
/* Без Intrinsic Sizing */
.container {
  display: flex;
  gap: 10px;
}

.item {
  flex: 0 0 200px; /* Жёсткий размер */
  /* Контент внутри может вылезать */
}

/* С Intrinsic Sizing */
.container {
  display: flex;
  gap: 10px;
  align-items: flex-start;
}

.item {
  flex: 0 0 auto; /* Адаптация к контенту */
  width: min-content; /* Максимально компактно */
  max-width: 300px; /* Но не больше 300px */
}
```

### Почему современный CSS отказался от фиксированных размеров

```text
Фиксированные размеры
  ↓
"Элемент должен быть 300px"
  ↓
Проблема: контент не влезает
  ↓
Intrinsic Sizing
  ↓
"Элемент должен быть между 
 содержимым и 300px"
  ↓
Браузер сам выбирает оптимальный размер
```

---

## 6.5 Выравнивание нового поколения

### Flexbox — универсальная система выравнивания

```css
.container {
  display: flex;
  
  /* Выравнивание по главной оси */
  justify-content: flex-start | flex-end | center | 
                    space-between | space-around | space-evenly;
  
  /* Выравнивание по поперечной оси */
  align-items: stretch | flex-start | flex-end | center | baseline;
  
  /* Выравнивание отдельных элементов */
  align-self: auto | stretch | flex-start | flex-end | center | baseline;
  
  /* Выравнивание групп строк */
  align-content: stretch | flex-start | flex-end | center |
                 space-between | space-around | space-evenly;
  
  /* Промежутки */
  gap: 10px;
}
```

### justify-content — распределение на главной оси

```text
flex-start
┌─────────────────────┐
│[A][B][C]             │
└─────────────────────┘

center
┌─────────────────────┐
│      [A][B][C]       │
└─────────────────────┘

space-between
┌─────────────────────┐
│[A]        [B]       │[C]│
└─────────────────────┘

space-around
┌─────────────────────┐
│  [A]    [B]    [C]   │
└─────────────────────┘

space-evenly
┌─────────────────────┐
│   [A]   [B]   [C]    │
└─────────────────────┘
```

### align-items — выравнивание на поперечной оси

```text
stretch (по умолчанию)
┌─────────────────────┐
│┌──────┐┌──────┐┌──────┐│
││A     ││B     ││C     ││
││      ││      ││      ││
│└──────┘└──────┘└──────┘│
└─────────────────────┘

center
┌─────────────────────┐
│      ┌──┐            │
│      │A │  ┌──┐      │
│┌──┐  └──┘  │B │ ┌──┐│
││A │       └──┘ │C ││
│└──┘            └──┘│
└─────────────────────┘
```

### gap — замена margin-хаков

**Раньше:**
```css
.item {
  margin-right: 10px;
  margin-bottom: 10px;
}

.item:last-child {
  margin-right: 0;
}

.item:nth-last-child(-n+3) {
  margin-bottom: 0;
}
```

**Теперь:**
```css
.container {
  display: flex;
  gap: 10px;
  /* Просто и предсказуемо */
}
```

### Почему современный CSS отказался от margin-хаков

```text
margin-right: 10px
  ↓
У каждого элемента отступ
  ↓
Но последнему не нужен
  ↓
:last-child { margin-right: 0; }
  ↓
Сложно и хрупко
  ↓
gap: 10px
  ↓
Единый интервал между элементами
  ↓
Просто и надёжно
```

---

## 6.6 Современные паттерны Flexbox

### Navigation Bar

```css
.navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: var(--space-md);
  background: var(--color-surface);
  border-bottom: 1px solid var(--border-color);
}

.navbar-logo {
  display: flex;
  align-items: center;
  gap: var(--space-sm);
}

.navbar-menu {
  display: flex;
  gap: var(--space-md);
  list-style: none;
}

.navbar-actions {
  display: flex;
  gap: var(--space-sm);
  align-items: center;
}

/* Адаптивность */
@container (max-width: 600px) {
  .navbar-menu {
    display: none; /* Меню-бургер */
  }
}
```

### Toolbar

```css
.toolbar {
  display: flex;
  gap: var(--space-xs);
  padding: var(--space-sm);
  background: var(--color-surface);
  border-radius: var(--radius-md);
  align-items: center;
  flex-wrap: wrap;
}

.toolbar-group {
  display: flex;
  gap: var(--space-xs);
  align-items: center;
  padding-right: var(--space-sm);
  border-right: 1px solid var(--border-color);
}

.toolbar-group:last-child {
  border-right: none;
  padding-right: 0;
}
```

### Card Header

```css
.card-header {
  display: flex;
  align-items: center;
  gap: var(--space-md);
  padding: var(--space-md);
  border-bottom: 1px solid var(--border-color);
}

.card-header-avatar {
  flex-shrink: 0;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: var(--color-primary);
}

.card-header-content {
  flex: 1;
  min-width: 0; /* Для обрезки текста */
}

.card-header-content h3 {
  font-size: var(--font-size-md);
  font-weight: var(--font-weight-bold);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.card-header-actions {
  display: flex;
  gap: var(--space-xs);
  flex-shrink: 0;
}
```

### Media Object

```css
.media {
  display: flex;
  gap: var(--space-md);
  align-items: flex-start;
}

.media-image {
  flex-shrink: 0;
  width: 80px;
  height: 80px;
  border-radius: var(--radius-md);
  overflow: hidden;
}

.media-image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.media-content {
  flex: 1;
  min-width: 0;
}

.media-title {
  font-weight: var(--font-weight-bold);
}

.media-description {
  color: var(--color-text-secondary);
  margin-top: var(--space-xs);
}
```

### Tags (flex-wrap)

```css
.tags {
  display: flex;
  flex-wrap: wrap;
  gap: var(--space-xs);
  padding: var(--space-sm);
}

.tag {
  display: flex;
  align-items: center;
  gap: var(--space-xs);
  padding: 0.25rem 0.75rem;
  background: var(--color-surface);
  border-radius: var(--radius-full);
  font-size: var(--font-size-sm);
  white-space: nowrap;
}

.tag-remove {
  cursor: pointer;
  opacity: 0.6;
  transition: opacity 0.2s ease;
}

.tag-remove:hover {
  opacity: 1;
}
```

### Breadcrumbs

```css
.breadcrumbs {
  display: flex;
  gap: var(--space-xs);
  align-items: center;
  padding: var(--space-sm);
  font-size: var(--font-size-sm);
}

.breadcrumb {
  display: flex;
  align-items: center;
  gap: var(--space-xs);
}

.breadcrumb:not(:last-child)::after {
  content: '/';
  opacity: 0.4;
}

.breadcrumb a {
  text-decoration: none;
  color: var(--color-primary);
}

.breadcrumb a:hover {
  text-decoration: underline;
}
```

### Search Bar

```css
.search-bar {
  display: flex;
  gap: 0;
  align-items: center;
  background: var(--color-surface);
  border: 1px solid var(--border-color);
  border-radius: var(--radius-md);
  overflow: hidden;
  transition: border-color 0.2s ease;
}

.search-bar:focus-within {
  border-color: var(--color-primary);
}

.search-bar input {
  flex: 1;
  padding: 0.5rem 1rem;
  border: none;
  outline: none;
  background: transparent;
  font-size: var(--font-size-base);
  min-width: 100px;
}

.search-bar button {
  padding: 0.5rem 1rem;
  background: transparent;
  border: none;
  cursor: pointer;
  color: var(--color-text-secondary);
}

.search-bar button:hover {
  color: var(--color-text);
}
```

### Split Button

```css
.split-button {
  display: flex;
  gap: 0;
  align-items: stretch;
}

.split-button-main {
  padding: 0.5rem 1.5rem;
  background: var(--color-primary);
  color: white;
  border: none;
  border-radius: var(--radius-md) 0 0 var(--radius-md);
  cursor: pointer;
}

.split-button-dropdown {
  padding: 0.5rem 0.75rem;
  background: var(--color-primary);
  color: white;
  border: none;
  border-left: 1px solid rgba(255,255,255,0.2);
  border-radius: 0 var(--radius-md) var(--radius-md) 0;
  cursor: pointer;
}
```

### Почему большинство UI-компонентов строится именно на Flexbox

```text
UI-компонент
  ↓
Обычно одномерный
  ↓
Список элементов
  ↓
Распределение вдоль одной оси
  ↓
Flexbox — идеальное решение
```

**Grid используется для:**
- Расположения секций страницы
- Сложных двумерных макетов
- Когда важны обе оси одновременно

**Flexbox используется для:**
- Компонентов (кнопки, карточки, навигация)
- Распределения элементов в строке/столбце
- Адаптации содержимого

---

## 6.7 Flexbox и современные возможности CSS

### Container Queries

```css
.card-container {
  container-type: inline-size;
  container-name: card;
}

@container card (max-width: 400px) {
  .card {
    flex-direction: column;
  }
  
  .card-content {
    flex: 1;
  }
}

@container card (min-width: 401px) {
  .card {
    flex-direction: row;
  }
}
```

### Aspect Ratio

```css
.card-image {
  flex-shrink: 0;
  aspect-ratio: 16 / 9;
  width: 100px;
  background: var(--color-surface);
  border-radius: var(--radius-md);
  overflow: hidden;
}
```

### Logical Properties

```css
/* Раньше — физические свойства */
.container {
  margin-left: 1rem;
  padding-right: 1rem;
  border-left: 1px solid;
}

/* Теперь — логические свойства */
.container {
  margin-inline-start: 1rem;   /* LTR: left, RTL: right */
  padding-inline-end: 1rem;    /* LTR: right, RTL: left */
  border-inline-start: 1px solid;
}
```

### Writing Modes

```css
/* Один компонент работает во всех режимах */
.component {
  display: flex;
  gap: var(--space-md);
  flex-direction: row; /* Становится column в vertical writing */
}

/* Явное указание направления */
.component {
  flex-direction: row;
  writing-mode: horizontal-tb; /* Горизонтальный */
}

.component-vertical {
  flex-direction: column;
  writing-mode: vertical-rl; /* Вертикальный */
}
```

### Почему один и тот же компонент автоматически работает во всех режимах

```css
/* Компонент */
.card {
  display: flex;
  gap: var(--space-md);
  padding: var(--space-md);
}

.card-title {
  flex: 1;
  min-width: 0;
}

.card-actions {
  display: flex;
  gap: var(--space-sm);
  flex-shrink: 0;
}
```

**Благодаря логическим свойствам и Writing Modes:**

```text
LTR (English)
  ↓
Icon → Title → Actions
  ↓
RTL (Arabic)
  ↓
Actions ← Title ← Icon
  ↓
Vertical (Japanese)
  ↓
    Icon
    Title
    Actions
```

---

## 6.8 Частые ошибки

### min-width: auto

**Проблема:** элементы не хотят сжиматься.

```css
.container {
  display: flex;
  width: 300px;
}

.item {
  flex: 1;
  /* min-width: auto (по умолчанию) */
  /* Элемент не сжимается меньше содержимого */
}
```

**Причина:** у flex-элементов по умолчанию `min-width: auto`, что предотвращает сжатие меньше размера содержимого.

**Решение:** `min-width: 0`

```css
.item {
  flex: 1;
  min-width: 0; /* Позволяет сжиматься */
}
```

### Переполнение контейнера

**Проблема:** содержимое вылезает за пределы.

```css
.container {
  display: flex;
  width: 300px;
}

.item {
  flex: 0 0 auto; /* Не сжимается */
}
```

**Решение:** `flex-shrink` или `overflow: hidden`

```css
.item {
  flex: 0 1 auto; /* Может сжиматься */
  overflow: hidden; /* Обрезает содержимое */
  text-overflow: ellipsis;
  white-space: nowrap;
}
```

### flex: 1 vs flex: auto

```css
/* flex: 1 — basis: 0 */
.item {
  flex: 1;
  /* Все элементы одинаковые, независимо от контента */
}

/* flex: auto — basis: auto */
.item {
  flex: auto;
  /* Элементы учитывают содержимое */
}
```

### 100% vs flex: 1

```css
/* ❌ Неправильно */
.item {
  width: 100%; /* Может не работать в flex */
}

/* ✅ Правильно */
.item {
  flex: 1; /* Доля свободного пространства */
}
```

---

## 6.9 Flexbox и производительность

### Что происходит внутри Layout Engine

```text
Flex Container
  ↓
Вычисление размеров элементов
  ↓
Определение свободного пространства
  ↓
Распределение (flex-grow / flex-shrink)
  ↓
Выравнивание
  ↓
Финальный Layout
```

### Почему Flexbox быстрее JavaScript Layout

```text
JavaScript Layout
  ↓
Загрузка JS
  ↓
Выполнение скрипта
  ↓
Расчёт размеров в JS
  ↓
Обновление DOM
  ↓
Пересчёт Layout
  ↓
Медленно (особенно при рефлоу)
  ↓
Flexbox Layout
  ↓
Нативный алгоритм в C++
  ↓
Оптимизирован для производительности
  ↓
Быстро
```

### Когда Grid дешевле, когда Flexbox

| Сценарий | Grid | Flexbox |
|----------|------|---------|
| Сложная сетка | Быстрее | Медленнее (много обёрток) |
| Простая композиция | Медленнее | Быстрее |
| Много элементов | Быстрее | Медленнее |
| Адаптивные размеры | Быстрее | Медленнее |
| Одномерный список | Медленнее | Быстрее |

### Почему браузер выполняет все вычисления самостоятельно

```text
Разработчик
  ↓
описывает правила
  ↓
Браузер
  ↓
вычисляет геометрию
  ↓
Пользователь
  ↓
получает быстрый интерфейс
```

---

## 6.10 Grid + Flexbox

### Современная архитектура

```text
Grid
  ↓
Страница
  ↓
Секции
  ↓
Компоненты
  ↓
Flexbox
  ↓
Содержимое
```

### Реальный пример

```html
<!-- Grid: архитектура страницы -->
<div class="dashboard">
  
  <!-- Flexbox: навигация -->
  <nav class="sidebar">
    <div class="logo">Лого</div>
    <ul class="menu">
      <li>Главная</li>
      <li>Продукты</li>
      <li>Настройки</li>
    </ul>
    <div class="user">Пользователь</div>
  </nav>
  
  <!-- Flexbox: хедер -->
  <header class="header">
    <div class="search">Поиск</div>
    <div class="actions">Действия</div>
  </header>
  
  <!-- Grid + Flexbox: контент -->
  <main class="main">
    <!-- Grid: карточки -->
    <div class="card-grid">
      <!-- Flexbox: карточка -->
      <div class="card">
        <img src="..." class="card-image">
        <div class="card-content">
          <h3>Заголовок</h3>
          <p>Описание</p>
          <div class="card-actions">
            <button>Действие</button>
          </div>
        </div>
      </div>
      <!-- Ещё карточки -->
    </div>
  </main>
  
  <!-- Flexbox: виджеты -->
  <aside class="widgets">
    <div class="widget">Виджет 1</div>
    <div class="widget">Виджет 2</div>
  </aside>
  
  <!-- Flexbox: футер -->
  <footer class="footer">
    <span>© 2026</span>
    <div class="links">Ссылки</div>
  </footer>
  
</div>
```

```css
/* Grid: архитектура страницы */
.dashboard {
  display: grid;
  grid-template-columns: 240px 1fr 280px;
  grid-template-rows: auto 1fr auto;
  grid-template-areas: 
    "sidebar header header"
    "sidebar main widgets"
    "sidebar footer footer";
  min-height: 100vh;
}

/* Flexbox: компоненты */
.sidebar {
  display: flex;
  flex-direction: column;
  gap: var(--space-lg);
  padding: var(--space-md);
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: var(--space-md);
}

.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: var(--space-md);
  padding: var(--space-md);
}

.card {
  display: flex;
  flex-direction: column;
  gap: var(--space-sm);
  padding: var(--space-md);
  background: var(--color-surface);
  border-radius: var(--radius-md);
}

.card-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: var(--space-sm);
}

.card-actions {
  display: flex;
  gap: var(--space-sm);
  justify-content: flex-end;
}
```

### Почему именно так строятся современные интерфейсы

```text
Grid — где находятся секции
  ↓
Flexbox — как распределены элементы внутри секций
  ↓
Container Queries — как компонент адаптируется
  ↓
Custom Properties — настройка компонентов
```

---

## 6.11 Progressive Enhancement *(добавлено уточнение про gap)*

### Flexbox — часть Baseline

```text
Flexbox поддерживается во всех современных браузерах
  ↓
Chrome: 29+
Firefox: 28+
Safari: 9+
Edge: 12+
```

> **Уточнение:** сам Flexbox — одна из самых давних и стабильных частей современного CSS, полностью в статусе Baseline Widely available. Однако свойство `gap` внутри flex-контейнеров пришло в браузеры позже, чем сам Flexbox: в Grid `gap` поддерживался с первых реализаций, а во Flexbox добавлен заметно позднее (последним его реализовал Safari). Сегодня `gap` во Flexbox тоже входит в Baseline Widely available, поэтому фича-детект через `@supports (gap: 10px)`, показанный ниже, на практике уже не обязателен для большинства проектов — но он остаётся хорошим примером паттерна прогрессивного улучшения и полезен для проектов с расширенной поддержкой старых браузеров.

### Проверка современных возможностей

```css
/* Проверка gap */
@supports (gap: 10px) {
  .container {
    display: flex;
    gap: var(--space-md);
  }
}

@supports not (gap: 10px) {
  .container {
    display: flex;
  }
  .container > * {
    margin-right: var(--space-md);
  }
  .container > *:last-child {
    margin-right: 0;
  }
}

/* Проверка логических свойств */
@supports (margin-inline-start: 1rem) {
  .container {
    margin-inline-start: var(--space-md);
  }
}

/* Проверка Container Queries */
@supports (container-type: inline-size) {
  .card-container {
    container-type: inline-size;
  }
}
```

---

## 6.12 Практические архитектурные шаблоны

### Dashboard

```css
.dashboard {
  display: grid;
  grid-template-columns: 200px 1fr;
  grid-template-rows: auto 1fr auto;
  grid-template-areas: 
    "sidebar header"
    "sidebar main"
    "sidebar footer";
  min-height: 100vh;
}

.sidebar {
  display: flex;
  flex-direction: column;
  gap: var(--space-md);
  padding: var(--space-md);
  background: var(--color-surface);
  border-right: 1px solid var(--border-color);
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: var(--space-md);
  border-bottom: 1px solid var(--border-color);
}

.main {
  padding: var(--space-md);
  container-type: inline-size;
}

@container (max-width: 800px) {
  .dashboard {
    grid-template-columns: 1fr;
    grid-template-areas: 
      "header"
      "sidebar"
      "main"
      "footer";
  }
}
```

### Admin Panel

```css
.admin-panel {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.admin-toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: var(--space-md);
  background: var(--color-surface);
  border-bottom: 1px solid var(--border-color);
}

.admin-content {
  flex: 1;
  padding: var(--space-md);
}

.admin-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: var(--space-md);
  border-top: 1px solid var(--border-color);
}
```

### Settings Page

```css
.settings {
  display: flex;
  flex-direction: column;
  gap: var(--space-lg);
  max-width: 800px;
  margin: 0 auto;
  padding: var(--space-lg);
}

.settings-section {
  display: flex;
  flex-direction: column;
  gap: var(--space-md);
  padding: var(--space-md);
  background: var(--color-surface);
  border-radius: var(--radius-md);
}

.settings-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: var(--space-md);
  padding: var(--space-sm) 0;
  border-bottom: 1px solid var(--border-color);
}

.settings-row:last-child {
  border-bottom: none;
}

@media (max-width: 600px) {
  .settings-row {
    flex-direction: column;
    align-items: flex-start;
    gap: var(--space-sm);
  }
}
```

### Chat Interface

```css
.chat {
  display: grid;
  grid-template-columns: 280px 1fr;
  grid-template-rows: auto 1fr auto;
  grid-template-areas: 
    "sidebar

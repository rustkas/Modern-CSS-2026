# Глава 5. Современный CSS Grid: декларативная архитектура макетов

> CSS Grid давно перестал быть просто заменой `float` или Flexbox. Сегодня это фундаментальная система декларативного описания двумерных интерфейсов, в которой браузер самостоятельно вычисляет геометрию страницы. Вместо ручного позиционирования элементов разработчик описывает правила, ограничения и отношения между компонентами, а движок браузера выполняет всю вычислительную работу. Именно поэтому современный Grid следует рассматривать не как инструмент вёрстки, а как язык проектирования интерфейсов.

---

## 5.1 От координат к ограничениям (Constraints)

### Эволюция подхода к позиционированию

Раньше разработчик явно задавал координаты каждого элемента:

```css
/* Координатный подход */
.element {
  position: absolute;
  left: 50px;
  top: 100px;
  width: 300px;
  height: 200px;
}

/* Float-подход */
.column {
  float: left;
  width: 33.333%;
}
```

**Проблемы координатного подхода:**

- Жёсткие размеры не адаптируются к контенту
- Изменение одного элемента ломает всю страницу
- Медиа-запросы требуют пересчёта всех координат
- Содержимое не определяет размеры

### Декларативная модель ограничений (Constraint-based Layout)

Современный Grid описывает **ограничения и правила**, а не координаты:

```css
/* Ограничения, а не координаты */
.grid {
  display: grid;
  grid-template-columns: 
    repeat(auto-fill, minmax(250px, 1fr));
  gap: var(--space-lg);
}
```

**Что описывает этот код:**
- Повторяющиеся колонки (repeat)
- Заполняют доступное пространство (auto-fill)
- Минимальная ширина — 250px (minmax)
- Максимальная ширина — пропорционально доступному месту (1fr)
- Отступы между элементами (gap)

```text
Координатный подход
  ↓
"Элемент на 50px от левого края"
  ↓
Grid подход
  ↓
"Элемент должен быть в колонке 
 шириной от 250px до 1fr"
```

### Язык ограничений Grid

```css
/* Базовые ограничения */
repeat()        → повторение паттернов
minmax()        → диапазон размеров
fit-content()   → адаптация к контенту
fr              → доля пространства
auto            → автоматический размер
```

**Почему это работает:**

```text
Разработчик
  ↓
описывает ограничения
  ↓
Браузер
  ↓
вычисляет оптимальную геометрию
  ↓
Пользователь
  ↓
получает адаптивный интерфейс
```

---

## 5.2 Алгоритм размещения Grid

### Как браузер строит сетку

```text
1. Explicit Grid (явная сетка)
   ↓
2. Implicit Grid (неявная сетка)
   ↓
3. Auto-placement (автоматическое размещение)
   ↓
4. Track sizing (вычисление размеров)
   ↓
5. Alignment (выравнивание)
```

### Подробное объяснение каждого этапа

**1. Explicit Grid (явная сетка)**

Определяется разработчиком через `grid-template-columns` и `grid-template-rows`:

```css
.grid {
  /* Явные колонки и строки */
  grid-template-columns: 200px 1fr 200px;
  grid-template-rows: 100px 1fr 100px;
}
```

**2. Implicit Grid (неявная сетка)**

Браузер создаёт дополнительные строки/колонки для элементов, не поместившихся в явную сетку:

```css
.grid {
  grid-template-columns: repeat(3, 1fr);
  grid-auto-rows: 200px; /* Неявные строки */
  grid-auto-flow: row; /* Направление потока */
}
```

**3. Auto-placement (автоматическое размещение)**

Браузер автоматически размещает элементы, не имеющие явной позиции:

```css
.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  /* Элементы автоматически заполняют ячейки */
}
```

**4. Track sizing (вычисление размеров)**

Браузер вычисляет размеры каждой строки и колонки на основе ограничений:

```css
.grid {
  grid-template-columns: 
    minmax(100px, 200px)   /* 100-200px */
    1fr                     /* доля оставшегося */
    auto;                   /* по содержимому */
}
```

**5. Alignment (выравнивание)**

Выравнивание элементов в ячейках и самой сетки в контейнере:

```css
.grid {
  /* Выравнивание внутри ячеек */
  justify-items: center;  /* по горизонтали */
  align-items: center;    /* по вертикали */
  
  /* Выравнивание самой сетки */
  justify-content: center;  /* по горизонтали */
  align-content: center;    /* по вертикали */
}
```

### Почему порядок вычислений важен

```text
Знание отдельных свойств
  ↓
Понимание алгоритма
  ↓
Предсказуемые макеты
```

Без понимания алгоритма невозможно предсказать поведение сетки.

---

## 5.3 Современные единицы измерения Grid

### fr (fractional unit)

Доля доступного пространства:

```css
.grid {
  grid-template-columns: 1fr 2fr 1fr;
  /* 1/4, 2/4, 1/4 доступного пространства */
}
```

### min-content

Минимальный размер по содержимому:

```css
.grid {
  grid-template-columns: min-content 1fr;
  /* Первая колонка — по самому длинному слову */
  /* Вторая — оставшееся пространство */
}
```

### max-content

Максимальный размер по содержимому:

```css
.grid {
  grid-template-columns: max-content 1fr;
  /* Первая колонка — по самому длинному содержимому */
  /* Вторая — оставшееся пространство */
}
```

### fit-content()

Адаптация к содержимому, но с ограничением:

```css
.grid {
  grid-template-columns: 
    fit-content(300px) 1fr;
  /* Первая колонка — по содержимому, но не больше 300px */
  /* Вторая — оставшееся пространство */
}
```

### minmax()

Диапазон размеров:

```css
.grid {
  grid-template-columns: 
    minmax(200px, 1fr) minmax(100px, 300px);
  /* Первая: от 200px до 1fr */
  /* Вторая: от 100px до 300px */
}
```

### auto

Автоматический размер:

```css
.grid {
  grid-template-columns: auto 1fr auto;
  /* Первая и третья — по содержимому */
  /* Средняя — оставшееся пространство */
}
```

### Почему исчезают магические размеры

```css
/* ❌ Магические размеры */
.grid {
  grid-template-columns: 250px 320px 280px;
  /* Почему именно эти числа? */
}

/* ✅ Декларативные ограничения */
.grid {
  grid-template-columns: 
    minmax(200px, 1fr) 
    minmax(250px, 2fr) 
    minmax(200px, 1fr);
  /* Логика размещения очевидна */
}
```

---

## 5.4 Intrinsic Layout

### Интринсик-дизайн

**Intrinsic Web Design** — подход, при котором размеры определяются содержимым, а не внешними ограничениями.

```text
Extrinsic Layout
  ↓
"Элемент должен быть шириной 300px"
  ↓
Intrinsic Layout
  ↓
"Элемент должен быть 
 не меньше содержимого 
 и не больше доступного пространства"
```

### Инструменты Intrinsic Layout

```css
/* 1. min-content — по содержимому */
.sidebar {
  width: min-content;
}

/* 2. max-content — максимально по содержимому */
.header {
  width: max-content;
}

/* 3. fit-content() — адаптация с ограничением */
.content {
  width: fit-content(800px);
}

/* 4. aspect-ratio — соотношение сторон */
.image {
  aspect-ratio: 16 / 9;
  width: 100%;
  object-fit: cover;
}

/* 5. object-fit — управление контентом внутри */
.image {
  object-fit: cover;
  object-position: center;
}
```

### Почему браузер самостоятельно рассчитывает размеры

```css
/* Раньше — жёсткие размеры */
.image-container {
  width: 300px;
  height: 200px;
  overflow: hidden;
}

.image-container img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

/* Теперь — адаптивные размеры */
.image-container {
  width: 100%;
  max-width: 600px;
  aspect-ratio: 3 / 2;
}

.image-container img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
```

**Преимущества Intrinsic Layout:**

1. Адаптация к содержимому
2. Меньше медиа-запросов
3. Браузер сам выбирает оптимальные размеры
4. Работает с любым контентом

---

## 5.5 Auto Layout

### Автоматическое размещение

Современный Grid практически полностью автоматизирует размещение элементов.

```css
/* Автоматическая сетка */
.grid {
  display: grid;
  grid-template-columns: 
    repeat(auto-fill, minmax(250px, 1fr));
  gap: var(--space-md);
}
```

**Что делает этот код:**

1. `auto-fill` — добавляет столько колонок, сколько помещается
2. `minmax(250px, 1fr)` — каждая колонка от 250px до 1fr
3. Браузер вычисляет оптимальное количество колонок
4. Сетка адаптируется к любому количеству элементов

### auto-fill vs auto-fit

**auto-fill — добавляет пустые колонки:**

```css
.grid {
  grid-template-columns: 
    repeat(auto-fill, minmax(250px, 1fr));
}
/* Если остаётся место, создаются пустые колонки */
```

**auto-fit — схлопывает пустые колонки:**

```css
.grid {
  grid-template-columns: 
    repeat(auto-fit, minmax(250px, 1fr));
}
/* Пустые колонки схлопываются, элементы растягиваются */
```

### Почему исчезают десятки медиа-запросов

**Раньше:**
```css
.grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
}

@media (max-width: 1200px) {
  .grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

@media (max-width: 900px) {
  .grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 600px) {
  .grid {
    grid-template-columns: 1fr;
  }
}
```

**Теперь:**
```css
/* Одна строка заменяет десятки медиа-запросов */
.grid {
  grid-template-columns: 
    repeat(auto-fit, minmax(250px, 1fr));
}
```

---

## 5.6 Grid Areas как декларативная схема интерфейса

### Визуальное проектирование через Areas

```css
.page {
  display: grid;
  grid-template-columns: 200px 1fr 200px;
  grid-template-rows: auto 1fr auto;
  grid-template-areas: 
    "header  header  header"
    "sidebar main    aside"
    "footer  footer  footer";
  gap: var(--space-md);
  min-height: 100vh;
}
```

**Что это описывает:**

```text
"header  header  header"   → шапка на всю ширину
"sidebar main    aside"    → три колонки: меню, контент, виджеты
"footer  footer  footer"   → подвал на всю ширину
```

### Использование Areas

```html
<div class="page">
  <header class="header">Шапка</header>
  <nav class="sidebar">Меню</nav>
  <main class="main">Контент</main>
  <aside class="aside">Виджеты</aside>
  <footer class="footer">Подвал</footer>
</div>
```

```css
.header { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main { grid-area: main; }
.aside { grid-area: aside; }
.footer { grid-area: footer; }
```

### Когда Areas лучше линий

**Areas лучше для:**
- Структурных макетов (страницы, разделы)
- Когда схема макета важна
- Быстрого прототипирования
- Документации дизайна

**Линии Grid лучше для:**
- Сложных сеток
- Пересечений элементов
- Точного контроля
- Динамического позиционирования

### Адаптация Areas

```css
/* Планшет */
@media (max-width: 1024px) {
  .page {
    grid-template-areas: 
      "header  header"
      "sidebar main"
      "aside   aside"
      "footer  footer";
    grid-template-columns: 200px 1fr;
  }
}

/* Телефон */
@media (max-width: 768px) {
  .page {
    grid-template-areas: 
      "header"
      "sidebar"
      "main"
      "aside"
      "footer";
    grid-template-columns: 1fr;
  }
}
```

---

## 5.7 Subgrid

### Проблема первой версии Grid

**Проблема:** вложенные сетки не наследовали структуру родителя.

```css
/* Родительская сетка */
.grid {
  display: grid;
  grid-template-columns: 200px 1fr 200px;
}

/* Дочерний элемент создавал свою сетку */
.card {
  display: grid;
  grid-template-columns: 1fr 1fr; /* Не соответствует родителю */
}
```

### Subgrid в действии

```css
/* Родительская сетка */
.product-grid {
  display: grid;
  grid-template-columns: 200px 1fr 200px;
  gap: var(--space-md);
}

/* Дочерний элемент наследует структуру */
.card {
  display: grid;
  grid-template-columns: subgrid;
  grid-column: span 3; /* Занимает все колонки */
}

.card-title {
  grid-column: 1 / 2; /* Использует колонки родителя */
}

.card-description {
  grid-column: 2 / 3;
}

.card-price {
  grid-column: 3 / 4;
}
```

### Почему Subgrid важен

```text
Без Subgrid
  ↓
Каждый компонент — отдельная сетка
  ↓
Нет согласованности между компонентами
  ↓
Сложно выровнять элементы в разных карточках
  ↓
С Subgrid
  ↓
Компоненты наследуют сетку родителя
  ↓
Единая структура для всех карточек
  ↓
Гарантированное выравнивание
```

### Практический пример

```css
/* Галерея продуктов */
.products {
  display: grid;
  grid-template-columns: 
    repeat(auto-fill, minmax(250px, 1fr));
  gap: var(--space-lg);
}

/* Каждая карточка наследует сетку родителя */
.product-card {
  display: grid;
  grid-template-rows: subgrid; /* Наследуем строки */
  grid-row: span 3; /* Занимаем 3 строки */
  gap: var(--space-sm);
}

/* Все элементы выровнены идеально */
.product-card img {
  grid-row: 1 / 2; /* Все изображения на одной высоте */
}

.product-card .title {
  grid-row: 2 / 3; /* Все заголовки выровнены */
}

.product-card .price {
  grid-row: 3 / 4; /* Все цены выровнены */
}
```

---

## 5.8 Grid и Container Queries

### Революция в адаптивности

```text
Media Queries
  ↓
Адаптация к окну браузера
  ↓
Container Queries
  ↓
Адаптация к контейнеру
  ↓
Grid + Container Queries
  ↓
Полностью автономные компоненты
```

### Практический пример

```css
/* 1. Контейнер для карточки */
.product-wrapper {
  container-type: inline-size;
  container-name: product;
}

/* 2. Grid внутри контейнера */
.product-card {
  display: grid;
  grid-template-columns: 1fr 2fr;
  gap: var(--space-md);
  padding: var(--space-md);
}

/* 3. Адаптация к контейнеру */
@container product (max-width: 400px) {
  .product-card {
    grid-template-columns: 1fr;
    gap: var(--space-sm);
  }
  
  .product-card img {
    aspect-ratio: 16/9;
  }
}

@container product (min-width: 401px) and (max-width: 700px) {
  .product-card {
    grid-template-columns: 1fr 1fr;
  }
}

@container product (min-width: 701px) {
  .product-card {
    grid-template-columns: 1fr 2fr;
  }
}
```

### Почему это работает

```text
Media Queries
  ↓
зависят от размера окна
  ↓
Container Queries
  ↓
зависят от размера контейнера
  ↓
Grid внутри Container Queries
  ↓
компонент адаптируется к месту
```

---

## 5.9 Grid и современные компоненты

### Интеграция со всей экосистемой

```css
/* 1. @layer — архитектура */
@layer components {
  /* 2. Design Tokens — единый язык */
  .grid {
    display: grid;
    gap: var(--space-lg);
    padding: var(--space-md);
    background: var(--color-bg);
    border-radius: var(--radius-lg);
    
    /* 3. Container Queries — адаптивность */
    container-type: inline-size;
    container-name: grid;
    
    /* 4. Grid — структура */
    grid-template-columns: 
      repeat(auto-fit, minmax(250px, 1fr));
  }
  
  /* 5. @scope — изоляция */
  @scope (.grid) {
    .card {
      /* 6. Subgrid — наследование структуры */
      display: grid;
      grid-template-rows: subgrid;
      grid-row: span 2;
      
      /* 7. Custom Properties — настройка */
      --card-bg: var(--color-surface);
      --card-radius: var(--radius-md);
      
      background: var(--card-bg);
      border-radius: var(--card-radius);
      
      /* 8. :has() — логика */
      &:has(img) {
        padding: 0;
        overflow: hidden;
      }
    }
  }
}
```

### Полная архитектура

```text
Design Tokens
  ↓
@layer (архитектура)
  ↓
@scope (изоляция)
  ↓
Grid (структура)
  ↓
Subgrid (наследование)
  ↓
Container Queries (адаптивность)
  ↓
Custom Properties (настройка)
  ↓
:has() (логика)
```

---

## 5.10 Производительность Grid

### Что происходит внутри браузера

```text
Grid Container
  ↓
Разбор свойств
  ↓
Вычисление треков
  ↓
Размещение элементов
  ↓
Вычисление размеров
  ↓
Layout
  ↓
Paint
  ↓
Composite
```

### Почему Grid быстрее сложного Flexbox

| Аспект | Flexbox | Grid |
|--------|---------|------|
| Расчёт | Однонаправленный | Двунаправленный |
| Сложность | Увеличивается с вложенностью | Остаётся постоянной |
| Вложенность | Требует много контейнеров | Один контейнер |
| Алгоритм | Простой, но медленный | Оптимизированный |

### Почему Grid почти всегда лучше JavaScript Layout

```text
JavaScript Layout
  ↓
Загрузка JS
  ↓
Парсинг
  ↓
Выполнение
  ↓
Обновление DOM
  ↓
Пересчёт Layout
  ↓
Медленно
  ↓
Grid Layout
  ↓
Парсинг CSS
  ↓
Layout Engine (нативная оптимизация)
  ↓
Быстро
```

**Сравнение производительности:**

```javascript
// ❌ Медленно — JavaScript Layout
const grid = document.querySelector('.grid');
for (let i = 0; i < 100; i++) {
  const item = document.createElement('div');
  item.style.width = `${100 / 4}%`;
  grid.appendChild(item);
}
```

```css
/* ✅ Быстро — CSS Grid */
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
}
```

---

## 5.11 Экспериментальные возможности Grid

### Gap Decorations

Украшения промежутков между колонками.

> **Статус поддержки:** Gap Decorations — на данный момент **Chromium-only** возможность (Chrome и Edge, начиная с версии 149, июнь 2026 года). Ни Firefox, ни Safari её пока не реализуют. Спецификация расширяет уже существующее свойство `column-rule` (ранее работавшее только в многоколоночной вёрстке) и добавляет парное свойство `row-rule` для горизонтальных промежутков.

```css
.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
  column-rule: 2px dashed var(--border-color);
}
```

Поскольку в браузерах без поддержки декорации гэпов просто не отображаются (макет при этом не ломается — пустые промежутки остаются пустыми), эту возможность можно применять как чистое прогрессивное улучшение, без обязательного `@supports`-фолбэка.

### Grid Lanes

Именованные полосы для более сложных макетов:

```css
.grid {
  display: grid;
  grid-template-columns: 
    [main-start] 1fr 
    [content-start] 2fr 
    [content-end] 1fr 
    [main-end];
}
```

### Masonry *(нативный Masonry-layout)*

> **Статус поддержки:** нативный Masonry-режим Grid остаётся фрагментированным между браузерами и синтаксисами. Safari реализовал его первым — под названием **Grid Lanes** (`display: grid-lanes` в новой ревизии спецификации). Chrome и Firefox поддержку пока не выпустили в стабильных версиях: обсуждение в CSS Working Group долго шло между синтаксисом `grid-template-rows: masonry` (за него исторически выступали Firefox и Safari) и отдельным `display: masonry` с `masonry-template-tracks` (позиция команды Chrome). Итоговый синтаксис на момент написания книги ещё не полностью устоялся между движками, поэтому при использовании в продакшене нужно закладывать существенный разброс поведения между браузерами, а не просто отсутствие поддержки.

```css
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  grid-template-rows: masonry; /* Экспериментально; синтаксис ещё не финализирован во всех движках */
}
```

### Использование через Progressive Enhancement

```css
/* Базовый макет без Masonry */
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
}

/* Улучшение для браузеров с поддержкой */
@supports (grid-template-rows: masonry) {
  .grid {
    grid-template-rows: masonry;
  }
}
```

Учитывая, что финальный синтаксис Masonry ещё может измениться, для продакшен-проектов в 2026 году разумнее относиться к этой возможности как к возможности для экспериментов и постепенного тестирования, а не как к готовому к внедрению инструменту — в отличие, например, от Subgrid, который уже полностью стабилен (см. ниже).

## 5.13 Progressive Enhancement *(уточнён статус Subgrid)*

### Как писать Grid сегодня

**Стратегия Baseline:**

```text
Grid Level 1 — Baseline Widely available, поддерживают все браузеры
  ↓
Subgrid (Grid Level 2) — Baseline Widely available с марта 2026 года
  ↓
Container Queries (размерные) — Baseline Widely available
  ↓
Style Queries — Limited availability (пока без Firefox)
  ↓
Gap Decorations — Limited availability (только Chromium)
  ↓
Masonry / Grid Lanes — экспериментально, синтаксис не финализирован
```

Subgrid стоит особняком в этом списке: он был реализован во всех основных браузерных движках ещё в 2023 году (Firefox — даже раньше, в 2019-м), и в марте 2026 года официально достиг статуса Baseline Widely available. Это означает, что откладывать использование Subgrid «до полной поддержки» больше нет причин — он готов к использованию в продакшене наравне с самим Grid Level 1.

### Использование @supports

```css
/* Базовый макет — работает везде */
.grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: var(--space-md);
}

/* Subgrid — можно применять без фича-детекта, но @supports не помешает при поддержке очень старых браузеров */
@supports (grid-template-columns: subgrid) {
  .card {
    grid-template-columns: subgrid;
  }
}

/* Masonry — обязательно требует @supports, синтаксис ещё не устоялся */
@supports (grid-template-rows: masonry) {
  .grid {
    grid-template-rows: masonry;
  }
}

@supports (grid-template-columns: repeat(auto-fit, minmax(250px, 1fr))) {
  .grid {
    grid-template-columns: 
      repeat(auto-fit, minmax(250px, 1fr));
  }
}
```
---

## 5.14 Итоги главы

После изучения главы читатель должен понимать, что:

1. **Grid — это алгоритм, а не набор свойств** — понимание алгоритма важнее знания синтаксиса

2. **Браузер самостоятельно рассчитывает геометрию страницы** — разработчик описывает ограничения, а не координаты

3. **Современный Layout строится вокруг ограничений (constraints)** — `fr`, `minmax()`, `fit-content()`, `auto` описывают правила

4. **Intrinsic Layout постепенно вытесняет классический Responsive Design** — размеры определяются содержимым

5. **Subgrid делает компонентные системы по-настоящему согласованными** — вложенные сетки наследуют структуру родителя

6. **Container Queries превращают Grid в основу адаптивных компонентов** — адаптация к контейнеру, а не к вьюпорту

7. **Экспериментальные возможности (Gap Decorations, Grid Lanes, Masonry) следует использовать через Progressive Enhancement**

8. **Современный CSS Layout — это декларативная система** — разработчик описывает *что* должно получиться, а браузер решает *как* этого добиться

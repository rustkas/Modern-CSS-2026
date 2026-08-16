# Глава 7. Container Queries: новая эра адаптивных интерфейсов

> Долгое время адаптивность веб-интерфейсов замыкалась исключительно на габаритах клиентского окна браузера. Однако расцвет компонентно-ориентированной архитектуры потребовал принципиально иных инструментов — таких, которые позволяют изолированному элементу реагировать на собственное непосредственное окружение, а не на абстрактный размер экрана. Контейнерные запросы (*Container Queries*) стали недостающим фундаментальным звеном, превратившим обычные блоки в автономные, интеллектуальные единицы интерфейса.

---

## 7.1 Пределы традиционных Media Queries

### От вьюпорта к контейнеру

Классические медиа-запросы ориентируются исключительно на *viewport* (окно просмотра), что порождает ряд системных архитектурных проблем:

```text
Media Queries
  ↓
ориентируются на окно браузера
  ↓
не видят изменения внутри страницы
  ↓
компоненты не могут адаптироваться к контексту
```

### Три архитектурные проблемы Media Queries

**1. Зависимость от глобального макета**

Если в структуре страницы динамически скрывается или появляется боковая панель, ширина области основного контента меняется, но глобальный `media query` этого «не замечает», поскольку физический размер окна остался прежним. Это приводит к визуальным коллизиям и нарушению целостности сетки.

```css
/* Media Query не видит изменения внутри страницы */
@media (max-width: 768px) {
  .sidebar {
    width: 100%; /* Но sidebar может быть уже 300px */
  }
}
```

**2. Трудности переиспользования**

Чтобы один и тот же компонент (например, универсальная карточка товара) корректно выглядел и в узком сайдбаре, и на широком экране, разработчики были вынуждены нагромождать сложные контекстные модификаторы.

```css
/* Без Container Queries */
.card--compact { /* для сайдбара */ }
.card--featured { /* для широкой области */ }
.card--default { /* для обычного состояния */ }

/* С Container Queries — одна карточка работает везде */
```

**3. Проблема сайдбара (*sidebar problem*)**

Компоненты часто преждевременно схлопываются в мобильное представление под воздействием общего размера экрана, хотя внутри их локального контейнера было предостаточно свободного пространства для полноценного отображения.

```text
Широкий экран (1200px)
  ↓
Media Query: всё хорошо
  ↓
Сайдбар шириной 300px
  ↓
Карточка внутри сайдбара должна быть компактной
  ↓
Но Media Query не видит ширину сайдбара
  ↓
Карточка остаётся широкой
```

### Container Queries решают эти проблемы

```text
Container Queries
  ↓
ориентируются на контейнер компонента
  ↓
видят изменения внутри страницы
  ↓
компоненты адаптируются к своему контексту
```

### Эволюция адаптивности

```text
1990-е: фиксированные макеты
  ↓
2000-е: адаптация к устройству (User Agent)
  ↓
2010-е: Media Queries (viewport)
  ↓
2020-е: Container Queries (контейнер)
  ↓
2026: Size + Style + Scroll-state Queries
```

---

## 7.2 Создание контекста контейнера

### container-type

Чтобы дочерний элемент получил способность гибко реагировать на размеры или свойства окружения, его родительский блок должен быть явно задекларирован в качестве контейнера с помощью свойства `container-type`.

```css
/* Базовое объявление контейнера */
.card-container {
  container-type: inline-size;
}
```

### Доступные режимы контейнеризации

| Режим | Отслеживает | Особенности |
|-------|-------------|-------------|
| **`inline-size`** | Ширину (инлайновую ось) | Наиболее востребованный. Сохраняет естественную высоту по содержимому. |
| **`size`** | Ширину и высоту | Требует явной высоты контейнера. |
| **`normal`** | Ничего (по умолчанию) | Не является размерным контейнером. |

**Важно:** `inline-size` позволяет сохранять естественную, вычисляемую по содержимому высоту родителя, полностью предотвращая его неожиданное схлопывание. `size` же требует явного задания размеров.

```css
/* inline-size — высота по содержимому */
.container-inline {
  container-type: inline-size;
  /* height: auto (по умолчанию) */
}

/* size — нужна явная высота */
.container-size {
  container-type: size;
  height: 400px; /* Без этого схлопнется */
}
```

### container-name

Для удобства адресации контейнерам можно присваивать уникальные имена:

```css
.card-container {
  container-type: inline-size;
  container-name: card;
}

/* Или краткая запись */
.card-container {
  container: card / inline-size;
}
```

### Важное архитектурное правило

**Сам элемент не может запрашивать параметры собственного контейнера** — запрос всегда направляется к ближайшему подходящему предку в DOM-дереве.

```html
<div class="card-container"> <!-- Контейнер -->
  <div class="card"> <!-- Запрашивает размеры контейнера -->
    <div class="card-content"> <!-- Тоже может запрашивать -->
    </div>
  </div>
</div>
```

### Особенность Style Queries

**Важное уточнение:** для стилевых запросов (Style Queries) в общем случае вообще не требуется явно объявлять `container-type` — по спецификации практически любой непустой элемент может выступать контейнером стилевого запроса.

```css
/* Для Style Queries container-type не обязателен */
.parent {
  --theme: dark; /* Достаточно переменной */
}

/* Дочерний элемент может запрашивать стиль родителя */
@container style(--theme: dark) {
  .child {
    background: #1a1a1a;
  }
}
```

`container-type: normal` — это не «включение» такой возможности, а лишь явное подтверждение того, что элемент *не* является ещё и размерным контейнером. Эта тонкость часто ускользает от внимания при первом знакомстве с темой.

---

## 7.3 Size Queries (запросы размера)

### Базовый синтаксис

Запросы размера (*Size Queries*) позволяют динамически изменять визуальное оформление потомков в зависимости от габаритов родительского контейнера. Синтаксически они близки к медиа-запросам, но вместо привычной директивы `@media` используется `@container`:

```css
/* Определяем контейнер */
.card-container {
  container-type: inline-size;
  container-name: card;
}

/* Запрашиваем размер контейнера */
@container card (min-width: 480px) {
  .card {
    grid-template-columns: 1fr 2fr;
  }
}

@container card (max-width: 479px) {
  .card {
    grid-template-columns: 1fr;
  }
}
```

### Диапазоны и условия

```css
@container card (min-width: 400px) and (max-width: 600px) {
  .card {
    padding: var(--space-md);
  }
}

@container card (width > 600px) {
  .card {
    padding: var(--space-xl);
  }
}

@container card (width <= 400px) {
  .card {
    padding: var(--space-sm);
  }
}
```

### Вложенные запросы

```css
/* Контейнер для карточек */
.product-grid {
  container-type: inline-size;
  container-name: grid;
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: var(--space-md);
}

/* Изменяем сетку в зависимости от ширины контейнера */
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

### Принцип интринсического дизайна

```text
Media Queries
  ↓
"На смартфоне — так, на планшете — так"
  ↓
Container Queries
  ↓
"В узком контейнере — так, 
 в широком контейнере — так"
```

Это знаменует окончательный переход к интринсическому (*внутреннему*) дизайну: каждый компонент самостоятельно определяет оптимальную схему расположения в зависимости от выделенного ему пространственного лимита, игнорируя то, запущен интерфейс на смартфоне или на сверхшироком мониторе.

### Статус зрелости

**Size Queries** — самая зрелая часть этой главы: они находятся в статусе **Baseline Widely available** с 2023 года и безопасны для продакшна.

---

## 7.4 Style Queries (запросы стилей)

### Декларативная темизация

Запросы стилей (*Style Queries*) расширяют возможности контейнерных запросов, позволяя проверять актуальные значения CSS-переменных (*Custom Properties*) у родительского контейнера:

```css
/* Родитель задаёт тему */
.sidebar {
  --theme: dark;
  --size: compact;
}

/* Дочерний компонент реагирует */
@container style(--theme: dark) {
  .card {
    background: var(--surface-dark);
    color: var(--text-light);
    border-color: var(--border-dark);
  }
}

@container style(--theme: light) {
  .card {
    background: var(--surface-light);
    color: var(--text-dark);
    border-color: var(--border-light);
  }
}
```

### Контекстные модификаторы без классов

Устраняется необходимость раздувать разметку вспомогательными модификаторами при размещении блоков в специфических зонах интерфейса:

```html
<!-- Без Style Queries — нужно добавлять классы -->
<div class="sidebar">
  <div class="card card--sidebar">...</div>
</div>

<!-- С Style Queries — классы не нужны -->
<div class="sidebar" style="--context: sidebar">
  <div class="card">...</div>
</div>
```

```css
@container style(--context: sidebar) {
  .card {
    padding: var(--space-sm);
    font-size: var(--font-size-sm);
  }
}

@container style(--context: featured) {
  .card {
    padding: var(--space-xl);
    font-size: var(--font-size-xl);
  }
}
```

### Логические цепочки

```css
/* Несколько условий */
@container style(--theme: dark) and style(--size: large) {
  .card {
    background: var(--surface-dark);
    padding: var(--space-xl);
    border-radius: var(--radius-lg);
  }
}

@container style(--theme: dark) or style(--size: compact) {
  .card {
    box-shadow: var(--shadow-sm);
  }
}
```

### Важная практическая оговорка

**Сравнение значений в Style Queries** идёт по вычисленному (*computed*) значению переменной как строки, а не по семантически эквивалентному значению.

```css
/* ❌ Не сработает */
.parent {
  --color: #0000ff; /* Синий в hex */
}

@container style(--color: blue) {
  .child {
    color: white; /* Не применится — 'blue' !== '#0000ff' */
  }
}
```

**Решение:** использовать `@property` для регистрации типа (см. главу 2, раздел 2.3):

```css
@property --color {
  syntax: '<color>';
  inherits: true;
  initial-value: #0000ff;
}

.parent {
  --color: blue; /* Теперь 'blue' и '#0000ff' эквивалентны */
}

@container style(--color: blue) {
  .child {
    color: white; /* Работает! */
  }
}
```

**Это частая причина, почему стилевой запрос «не срабатывает», хотя визуально цвет совпадает.**

### Статус зрелости

**Style Queries** — заметно более свежая возможность, чем Size Queries: статус **Baseline Newly available** достигнут 19 мая 2026 года, когда поддержку добавил Firefox 151 (Chrome и Safari поддерживали Style Queries раньше). Важно не путать это со статусом **Widely available**: «Newly» означает лишь то, что все четыре core-браузера поддерживают функцию в актуальных стабильных версиях, но 30-месячный период, после которого функция станет по-настоящему безопасной без оговорок, ещё не прошёл (и истечёт не раньше ноября 2028 года).

На практике это означает:

* пользователи с устаревшими версиями браузеров (например, Firefox младше 151 или корпоративные сборки на Firefox ESR, которые могут отставать от актуального релиза на многие месяцы) Style Queries не увидят;
* для продакшн-проектов с широкой аудиторией стоит закладывать `@supports`-проверку и разумный fallback ещё как минимум несколько лет — до тех пор, пока функция не перейдёт в статус Widely available.

```css
@supports (container-type: style) {
  /* Style Queries можно использовать */
}
```

---

## 7.5 Scroll-state Queries (запросы состояния скролла)

### Новое измерение контейнерных запросов

Помимо размера и стиля, контейнер теперь может сообщать своим потомкам о собственном *состоянии прокрутки* — это отдельный, третий вид условий `@container`, который стоит знать наравне с первыми двумя.

```css
/* Определяем контейнер с отслеживанием скролла */
.scroll-container {
  container-type: scroll-state;
  container-name: panel;
  scroll-snap-type: y mandatory;
  overflow-y: auto;
  height: 400px;
}

/* Реагируем на состояние скролла */
@container panel scroll-state(snapped: block) {
  .indicator {
    opacity: 1;
    transform: scale(1);
  }
}

@container panel scroll-state(scrollable: block) {
  .scroll-hint {
    display: block;
  }
}
```

### Доступные состояния

| Состояние | Описание |
|-----------|----------|
| `snapped: block` | Контейнер зафиксирован на снап-позиции (block-ось) |
| `snapped: inline` | Контейнер зафиксирован на снап-позиции (inline-ось) |
| `snapped: both` | Контейнер зафиксирован на снап-позиции по обеим осям |
| `scrollable: block` | Контейнер прокручиваем по block-оси |
| `scrollable: inline` | Контейнер прокручиваем по inline-оси |
| `scrollable: both` | Контейнер прокручиваем по обеим осям |

### Практический пример: карусель

```css
.carousel {
  container-type: scroll-state;
  container-name: carousel;
  display: flex;
  overflow-x: auto;
  scroll-snap-type: x mandatory;
  gap: var(--space-md);
}

.carousel-item {
  flex: 0 0 300px;
  scroll-snap-align: start;
  transition: transform 0.3s ease;
}

/* Активный слайд увеличивается */
@container carousel scroll-state(snapped: inline) {
  .carousel-item {
    transform: scale(1.05);
  }
}

/* Индикатор прогресса */
@container carousel scroll-state(snapped: inline) {
  .progress {
    width: 100%;
    transition: width 0.3s ease;
  }
}
```

### Преимущества перед JavaScript

```text
Раньше:
  ↓
JavaScript + IntersectionObserver
  ↓
Подписка на события скролла
  ↓
Вычисление позиции
  ↓
Обновление DOM
  ↓
Медленно, сложно

Теперь:
  ↓
CSS + Scroll-state Queries
  ↓
Декларативно
  ↓
Браузер сам отслеживает состояние
  ↓
Быстро, просто
```

### Статус зрелости

**Scroll-state Queries** — относительно недавняя часть спецификации `@container`. Как и Style Queries, это относительно свежая возможность — перед использованием в продакшне стоит свериться с актуальным статусом на [webstatus.dev](https://webstatus.dev).

---

## 7.6 Контейнерные единицы измерения (Container Units)

### Новые относительные единицы

Вместе с запросами спецификация привнесла набор новых относительных единиц измерения, привязанных строго к физическим размерам родительского контейнера, а не окна браузера:

| Единица | Относительно |
|---------|-------------|
| **`cqw`** | 1% от ширины контейнера |
| **`cqh`** | 1% от высоты контейнера |
| **`cqi`** | 1% от инлайнового размера контейнера |
| **`cqb`** | 1% от блочного размера контейнера |
| **`cqmin`** | Меньшее из `cqi` и `cqb` |
| **`cqmax`** | Большее из `cqi` и `cqb` |

### Практическое применение

```css
/* Адаптивная типографика */
.card {
  container-type: inline-size;
}

.card-title {
  font-size: clamp(1rem, 3cqi, 2.5rem);
  /* От 1rem до 2.5rem, зависит от ширины контейнера */
}

.card-image {
  width: 100%;
  height: 40cqw; /* 40% от ширины контейнера */
  object-fit: cover;
}

.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(30cqi, 1fr));
  gap: 2cqi;
}
```

### Fluid Typography с Container Units

```css
.heading {
  container-type: inline-size;
}

.heading h1 {
  font-size: calc(1.5rem + 2cqi);
  /* Растёт вместе с контейнером */
}

.heading p {
  font-size: calc(1rem + 0.5cqi);
  line-height: calc(1.5 + 0.5cqi);
}
```

### Важное ограничение

**Контейнерные единицы работают только применительно к элементам, у которых `container-type` установлен в `size` или `inline-size`.** Если подходящего размерного контейнера-родителя в иерархии нет, движок браузера корректно возвращается к дефолтным единицам малой видимой области (`sv*`, small viewport units), а не к обычным `vw`/`vh`.

```css
/* ❌ Не сработает — нет контейнера */
.element {
  font-size: 2cqi; /* Вернётся к svw */
}

/* ✅ Сработает — есть контейнер */
.container {
  container-type: inline-size;
}

.container .element {
  font-size: 2cqi; /* Относится к контейнеру */
}
```

---

## 7.7 Практические архитектурные паттерны

### Интеллектуальные карточки контента

Элементы ленты новостей или товаров автоматически переключаются из компактного вертикального режима в развёрнутую горизонтальную «фичерную» статью, как только сеточный контейнер выделяет им расширенную область.

```css
.card-container {
  container-type: inline-size;
  container-name: card;
}

/* Компактный режим (узкий контейнер) */
@container card (max-width: 400px) {
  .card {
    display: flex;
    flex-direction: column;
    gap: var(--space-sm);
    padding: var(--space-sm);
  }
  .card-image {
    width: 100%;
    aspect-ratio: 16/9;
  }
  .card-title {
    font-size: var(--font-size-md);
  }
}

/* Горизонтальный режим (средний контейнер) */
@container card (min-width: 401px) and (max-width: 700px) {
  .card {
    display: grid;
    grid-template-columns: 1fr 2fr;
    gap: var(--space-md);
    padding: var(--space-md);
  }
  .card-image {
    height: 100%;
    aspect-ratio: 1/1;
  }
}

/* Фичерный режим (широкий контейнер) */
@container card (min-width: 701px) {
  .card {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: var(--space-xl);
    padding: var(--space-xl);
  }
  .card-title {
    font-size: var(--font-size-2xl);
  }
}
```

### Адаптивные дашборды

Пользовательские панели управления, где изменение размеров колонок виджетов пользователем на лету вызывает перестройку внутренних графиков и таблиц с сокрытием или раскрытием детализации.

```css
.dashboard {
  container-type: inline-size;
  container-name: dashboard;
}

@container dashboard (max-width: 600px) {
  .widget {
    grid-column: span 2;
  }
  .widget-chart {
    height: 200px;
  }
  .widget-details {
    display: none;
  }
}

@container dashboard (min-width: 601px) and (max-width: 900px) {
  .widget {
    grid-column: span 1;
  }
  .widget-chart {
    height: 300px;
  }
  .widget-details {
    display: block;
    font-size: var(--font-size-sm);
  }
}

@container dashboard (min-width: 901px) {
  .widget {
    grid-column: span 1;
  }
  .widget-chart {
    height: 400px;
  }
  .widget-details {
    display: grid;
    grid-template-columns: 1fr 1fr;
  }
}
```

### Интерфейсная навигация

Боковые панели управления, способные при программном сжатии ширины контейнера бесшовно трансформировать текстовые пункты меню в аккуратные интерактивные иконки силами одного лишь CSS.

```css
.sidebar {
  container-type: inline-size;
  container-name: sidebar;
}

@container sidebar (max-width: 200px) {
  .nav-item {
    display: flex;
    justify-content: center;
    padding: var(--space-sm);
  }
  .nav-item .label {
    display: none;
  }
  .nav-item .icon {
    font-size: var(--font-size-xl);
  }
}

@container sidebar (min-width: 201px) {
  .nav-item {
    display: flex;
    align-items: center;
    gap: var(--space-md);
    padding: var(--space-md);
  }
  .nav-item .label {
    display: inline;
  }
}
```

### Гибкие формы ввода

Поля ввода данных и группы элементов управления, переходящие из вертикального стека в многоколоночные ряды в зависимости от ширины формы, а не размеров экрана телефона.

```css
.form {
  container-type: inline-size;
  container-name: form;
}

@container form (max-width: 400px) {
  .form-row {
    display: flex;
    flex-direction: column;
    gap: var(--space-xs);
  }
  .form-actions {
    flex-direction: column;
  }
}

@container form (min-width: 401px) {
  .form-row {
    display: grid;
    grid-template-columns: 120px 1fr;
    gap: var(--space-md);
  }
  .form-actions {
    display: flex;
    gap: var(--space-sm);
    justify-content: flex-end;
  }
}
```

### Индикаторы скролла и снап-карусели

Комбинация Scroll-state Queries с CSS Scroll Snap — панель может подсвечивать текущий активный элемент без единой строчки JavaScript.

```css
.gallery {
  container-type: scroll-state;
  container-name: gallery;
  display: flex;
  overflow-x: auto;
  scroll-snap-type: x mandatory;
  gap: var(--space-md);
}

.gallery-item {
  flex: 0 0 300px;
  scroll-snap-align: start;
  transition: all 0.3s ease;
}

/* Активный слайд */
@container gallery scroll-state(snapped: inline) {
  .gallery-item {
    transform: scale(1.05);
    box-shadow: var(--shadow-lg);
  }
}

/* Индикатор прокрутки */
@container gallery scroll-state(scrollable: inline) {
  .scroll-indicator {
    opacity: 1;
  }
}
```

---

## 7.8 Progressive Enhancement

### Стратегия внедрения

```css
/* 1. Базовые стили — работают всегда */
.card {
  display: flex;
  flex-direction: column;
  padding: var(--space-md);
}

/* 2. Проверка поддержки Container Queries */
@supports (container-type: inline-size) {
  .card-container {
    container-type: inline-size;
  }
  
  @container (min-width: 400px) {
    .card {
      flex-direction: row;
    }
  }
}

/* 3. Проверка Style Queries */
@supports (container-type: style) {
  .parent {
    --theme: dark;
  }
  
  @container style(--theme: dark) {
    .card {
      background: var(--surface-dark);
    }
  }
}

/* 4. Проверка Scroll-state Queries */
@supports (container-type: scroll-state) {
  .carousel {
    container-type: scroll-state;
  }
}
```

### Проверка поддержки в JavaScript

```javascript
// Проверка поддержки Container Queries
const supportsContainerQueries = CSS.supports('container-type', 'inline-size');

// Проверка поддержки Style Queries
const supportsStyleQueries = CSS.supports('container-type', 'style');

// Проверка поддержки Scroll-state Queries
const supportsScrollState = CSS.supports('container-type', 'scroll-state');
```

### Философия Progressive Enhancement

```text
Базовый HTML
  ↓
работает всегда
  ↓
Базовый CSS
  ↓
работает всегда
  ↓
Container Queries
  ↓
улучшают адаптивность
  ↓
Style Queries
  ↓
улучшают темизацию
  ↓
Scroll-state Queries
  ↓
улучшают интерактивность
```

---

## 7.9 Итоги главы

После изучения главы читатель должен понимать, что:

1. **Media Queries больше не достаточны** — компонентная архитектура требует адаптации к контейнеру, а не к вьюпорту

2. **Существует три типа Container Queries:**
   - **Size Queries** — реагируют на размер контейнера (Baseline Widely available)
   - **Style Queries** — реагируют на стили контейнера (Baseline Newly available)
   - **Scroll-state Queries** — реагируют на состояние прокрутки (Новейшая возможность)

3. **Container Queries — фундаментальный сдвиг** — от адаптации к устройству к адаптации к контексту

4. **Container Units** — `cqw`, `cqh`, `cqi`, `cqb`, `cqmin`, `cqmax` — привязывают размеры к контейнеру

5. **Style Queries требуют внимания к типам** — сравнение значений зависит от регистрации через `@property`

6. **Разные типы имеют разную зрелость** — Size Queries — надёжный стандарт, Style и Scroll-state — развивающийся фронтир

7. **Container Queries интегрируются с Grid и Flexbox** — создавая полностью автономные компоненты

8. **Progressive Enhancement — ключевая стратегия** — проверка через `@supports` и разумные fallback

---

**Главная мысль:** Container Queries фундаментально трансформируют инженерный подход к вёрстке. Мы больше не проектируем статичные страницы под фиксированные разрешения устройств — мы создаём суверенные, независимые компоненты, способные элегантно адаптироваться к любой программной и пространственной среде. При этом стоит различать зрелость частей спецификации: Size Queries — надёжный сегодняшний стандарт, тогда как Style Queries и особенно Scroll-state Queries — быстро развивающийся, но всё ещё относительно свежий фронтир, который имеет смысл использовать с проверкой поддержки.


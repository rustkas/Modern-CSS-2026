# Глава 21. Rendering Pipeline и производительность современного CSS

> Современный CSS невозможно эффективно использовать без понимания того, как браузер преобразует HTML и CSS в изображение на экране. Практически каждое CSS-свойство оказывает влияние на один или несколько этапов рендеринга. Одни изменения требуют полного пересчёта документа, другие затрагивают только отдельный слой композиции и практически не влияют на производительность.

Современный разработчик уже не оптимизирует отдельные селекторы «на глаз». Вместо этого он проектирует интерфейс таким образом, чтобы изменения как можно реже проходили дорогостоящие стадии Rendering Pipeline.

Главная идея главы проста:

> **Чем позже в Rendering Pipeline произошло изменение, тем дешевле оно обходится браузеру.**

---

## 21.1 Rendering Pipeline современного браузера

### Последовательность стадий

Любое изменение документа проходит несколько последовательных стадий.

```text
DOM
  │
  ▼
Style (пересчёт стилей)
  │
  ▼
Layout (геометрия и позиционирование)
  │
  ▼
Paint (отрисовка пикселей)
  │
  ▼
Composite (композиция слоёв)
```

### Зависимости между стадиями

Каждая последующая стадия зависит от предыдущей.

```text
Изменение Style
  ↓
Style → Layout → Paint → Composite
```

```text
Изменение Layout
  ↓
Layout → Paint → Composite
```

```text
Изменение Paint
  ↓
Paint → Composite
```

```text
Изменение Composite
  ↓
Composite
```

**Если произошло изменение Layout — автоматически выполняются Layout, Paint и Composite.**

**Если изменение затронуло только Paint — выполняются Paint и Composite.**

**Если изменился только Composite — никакие предыдущие этапы уже не запускаются.**

### Стоимость стадий

```text
Style ★☆☆☆☆
  ↓
Layout ★★★★★ (самая дорогая)
  ↓
Paint ★★★☆☆
  ↓
Composite ★☆☆☆☆ (самая дешёвая)
```

**Именно поэтому анимации `transform` и `opacity` остаются самым производительным вариантом практически для любых интерфейсов.**

---

## 21.2 Style Recalculation

### Что происходит на этой стадии

Первая стадия — вычисление окончательных стилей элемента. На этом этапе браузер:

```text
Разрешает Cascade
  ↓
Вычисляет специфичность
  ↓
Применяет Layers (@layer)
  ↓
Вычисляет Custom Properties
  ↓
Разрешает наследование
  ↓
Применяет Container Queries
  ↓
Вычисляет значения функций (calc(), clamp(), color-mix() и др.)
```

### Практические рекомендации

Для большинства современных проектов именно эта стадия стала важнее, чем традиционные разговоры о "быстрых селекторах".

**1. Избегайте чрезмерно сложных селекторов**

```css
/* ❌ Плохо — сложный селектор */
#app .sidebar .menu li a.active {
  color: blue;
}

/* ✅ Хорошо — простой селектор */
.nav-link.active {
  color: blue;
}
```

**2. Используйте `@layer` для управления приоритетами**

```css
@layer components {
  .button { ... }
}
```

**3. Применяйте `:where()` для снижения специфичности**

```css
/* Нулевая специфичность — быстрое вычисление */
:where(.button) {
  padding: 0.5rem 1rem;
}
```

**4. Проектируйте небольшие независимые компоненты**

```css
/* Каждый компонент изолирован */
@scope (.card) {
  .title { ... }
  .content { ... }
}
```

**5. Осторожно используйте сложные конструкции внутри `:has()`**

```css
/* ✅ Хорошо — простой селектор внутри :has() */
.card:has(> img) { ... }

/* ❌ Плохо — сложный поиск */
.card:has(.content .nested .element) { ... }
```

---

## 21.3 Layout: самая дорогая стадия

### Что вызывает Layout

Layout вычисляет размеры и положение каждого элемента. Именно здесь работают:

```text
Flexbox
  ↓
Grid
  ↓
Subgrid
  ↓
Container Queries
  ↓
Anchor Positioning
  ↓
Writing Modes
  ↓
Intrinsic Sizing
```

### Свойства, вызывающие Layout

Изменения таких свойств обычно приводят к полному пересчёту Layout:

```text
width / height
  ↓
inset / top / left / right / bottom
  ↓
margin / padding
  ↓
border-width
  ↓
display
  ↓
position
  ↓
float
  ↓
flex-* (в некоторых случаях)
  ↓
grid-* (в некоторых случаях)
```

**На больших документах именно Layout остаётся самым дорогим этапом Rendering Pipeline.**

### Практические рекомендации

```css
/* ❌ Плохо — анимация Layout */
.element {
  transition: width 0.3s ease;
  width: 100px;
}
.element:hover {
  width: 200px; /* Вызывает Layout на каждом кадре */
}

/* ✅ Хорошо — анимация Composite */
.element {
  transition: transform 0.3s ease;
  transform: scaleX(1);
}
.element:hover {
  transform: scaleX(2); /* Только Composite */
}
```

---

## 21.4 Ограничение области пересчёта

### contain — изоляция элемента

Современный CSS позволяет значительно уменьшить объём работы браузера через свойство `contain`.

```css
.element {
  contain: layout;   /* Изолирует Layout */
  contain: paint;    /* Изолирует Paint */
  contain: size;     /* Изолирует размеры */
  contain: content;  /* layout + paint + style */
}
```

**Браузеру больше не приходится пересчитывать всю страницу.**

```css
/* Изолируем карточку */
.card {
  contain: content;
  /* Изменения внутри .card не влияют на остальную страницу */
}
```

### content-visibility — пропуск рендеринга

Позволяет полностью пропустить вычисление элементов вне области просмотра.

```css
.long-list-item {
  content-visibility: auto;
  /* Элемент рендерится только когда появляется в viewport */
}
```

**Совместно с `contain-intrinsic-size` это стало одним из самых мощных инструментов ускорения длинных страниц.**

```css
.long-list-item {
  content-visibility: auto;
  contain-intrinsic-size: 200px; /* Резервируем место */
  /* Исчезают скачки Layout (CLS) */
}
```

### contain-intrinsic-size — резервирование места

Позволяет зарезервировать место ещё до вычисления содержимого.

```css
/* Без contain-intrinsic-size — CLS */
.long-list-item {
  content-visibility: auto;
}

/* С contain-intrinsic-size — нет CLS */
.long-list-item {
  content-visibility: auto;
  contain-intrinsic-size: 200px 100px; /* width height */
}
```

---

## 21.5 Paint

### Что происходит на стадии Paint

После вычисления геометрии начинается отрисовка. Paint включает:

```text
Фон
  ↓
Текст
  ↓
Изображения
  ↓
Границы
  ↓
Тени
  ↓
Градиенты
  ↓
SVG
  ↓
Маски
  ↓
Фильтры
```

### Свойства, вызывающие Paint

```text
background / background-color
  ↓
color
  ↓
border / border-color
  ↓
box-shadow
  ↓
filter
  ↓
clip-path
  ↓
border-radius
  ↓
text-shadow
```

**Изменение этих свойств обычно приводит именно к Paint без повторного Layout.**

### Дорогие эффекты

Следует помнить, что некоторые эффекты могут быть значительно дороже обычной отрисовки.

```text
box-shadow с большим blur
  ↓
filter: blur()
  ↓
сложные градиенты
  ↓
множественные фоны
  ↓
SVG-фильтры
```

```css
/* ❌ Плохо — дорогой эффект */
.element {
  box-shadow: 0 0 50px rgba(0,0,0,0.5);
  filter: blur(10px);
}

/* ✅ Хорошо — бюджетный эффект */
.element {
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  filter: blur(2px);
}
```

---

## 21.6 Composite

### Что происходит на стадии Composite

Последний этап Rendering Pipeline объединяет готовые слои. Именно здесь работают:

```text
transform
  ↓
opacity
  ↓
filter (некоторые)
  ↓
clip-path (в отдельных случаях)
```

### Свойства, работающие только на Composite

```text
transform
  ↓
translate, rotate, scale, skew, matrix
  ↓
opacity
  ↓
filter: blur() (в некоторых браузерах)
  ↓
will-change
```

**Если изменение можно выполнить исключительно на стадии Composite, браузер не выполняет ни Layout, ни Paint.**

### Почему transform и opacity — самые производительные

```text
Анимация transform
  ↓
Не влияет на Layout (геометрия не меняется)
  ↓
Не влияет на Paint (содержимое не меняется)
  ↓
Только Composite
  ↓
Быстро (GPU)
  ↓
Анимация opacity
  ↓
Не влияет на Layout
  ↓
Не влияет на Paint (содержимое не перерисовывается)
  ↓
Только Composite
  ↓
Быстро (GPU)
```

---

## 21.7 Композитные слои

### Как браузер создаёт слои

Браузер автоматически создаёт отдельные compositing layers для:

```text
Элементов с анимацией transform/opacity
  ↓
Элементов с will-change: transform/opacity
  ↓
Элементов с 3D-трансформациями
  ↓
Элементов с video, canvas, WebGL
  ↓
Элементов с z-index и position: fixed
  ↓
Элементов с overflow: hidden + clip
```

### will-change — подсказка браузеру

Иногда разработчик может подсказать браузеру заранее.

```css
.element {
  will-change: transform;
  /* Браузер создаст слой до начала анимации */
}

.element {
  will-change: opacity;
}
```

### Осторожно: стоимость слоёв

Однако злоупотреблять этим механизмом нельзя. Каждый дополнительный слой:

```text
Потребляет видеопамять
  ↓
Увеличивает стоимость композиции
  ↓
Может ухудшить производительность
```

**`will-change` следует применять только непосредственно перед ожидаемой анимацией.**

```css
/* ❌ Плохо — всегда активен */
* {
  will-change: transform;
}

/* ✅ Хорошо — только для анимируемых элементов */
.animated-element {
  will-change: transform;
}

/* ✅ Лучше — динамическое добавление */
.element:hover {
  will-change: transform;
}
```

---

## 21.8 Производительные анимации

### Главное правило

**Не заставляйте браузер выполнять Layout на каждом кадре.**

### Что анимировать

| Приоритет | Свойства | Стадия |
|-----------|----------|--------|
| ✅ Лучше всего | `transform`, `opacity` | Composite |
| ⚠️ С осторожностью | `filter`, `clip-path` | Paint |
| ❌ Избегать | `width`, `height`, `left`, `top`, `margin` | Layout |

### Пример: производительная анимация

```css
/* ✅ Хорошо — только Composite */
@keyframes slide-in {
  from {
    transform: translateX(-100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

.element {
  animation: slide-in 0.5s ease;
}
```

```css
/* ❌ Плохо — Layout на каждом кадре */
@keyframes slide-in-bad {
  from {
    left: -100%;
    opacity: 0;
  }
  to {
    left: 0;
    opacity: 1;
  }
}

.element {
  animation: slide-in-bad 0.5s ease;
  position: relative;
}
```

### Scroll-driven Animations

Особенно это важно для Scroll-driven Animations и View Transitions, рассмотренных в предыдущих главах.

```css
/* ✅ Хорошо — производительная Scroll-анимация */
@keyframes reveal {
  from {
    transform: translateY(50px);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}

.card {
  animation: reveal linear;
  animation-timeline: view();
  /* Только Composite */
}
```

---

## 21.9 DevTools как инструмент анализа

### Почему профилирование важно

Современная оптимизация невозможна без профилирования. Вместо прежнего подхода «оптимизировать заранее» рекомендуется:

```text
Измерить проблему
  ↓
Локализовать её
  ↓
Изменить CSS
  ↓
Проверить результат
```

### Что показывают DevTools

Chrome DevTools позволяют увидеть:

```text
Style Recalculation
  ↓
Layout
  ↓
Paint
  ↓
Composite
  ↓
Количество слоёв
  ↓
FPS
  ↓
Layout Shift (CLS)
  ↓
Interaction to Next Paint (INP)
  ↓
Largest Contentful Paint (LCP)
```

### Основные вкладки

**Performance** — профилирование рендеринга:

```text
Запись производительности
  ↓
Показывает все стадии
  ↓
Позволяет найти узкие места
```

**Layers** — анализ композитных слоёв:

```text
Показывает созданные слои
  ↓
Позволяет увидеть, какие элементы на GPU
  ↓
Помогает оптимизировать композицию
```

**Rendering** — визуализация перерисовок:

```text
Paint flashing
  ↓
Layout Shift
  ↓
FPS Meter
```

---

## 21.10 Практические рекомендации

### Архитектурные правила

При проектировании компонентов полезно придерживаться следующих правил:

**1. Минимизируйте количество изменений Layout**

```css
/* ❌ Плохо — анимация Layout */
.element {
  transition: width 0.3s ease;
}

/* ✅ Хорошо — анимация Composite */
.element {
  transition: transform 0.3s ease;
}
```

**2. Используйте `content-visibility` для больших страниц**

```css
.long-list-item {
  content-visibility: auto;
  contain-intrinsic-size: 200px;
}
```

**3. Применяйте `contain` для независимых блоков**

```css
.card {
  contain: content;
}
```

**4. Анимируйте преимущественно `transform` и `opacity`**

```css
/* ✅ Рекомендуется */
transform: translateX(100px);
opacity: 0.5;

/* ❌ Не рекомендуется */
left: 100px;
width: 200px;
```

**5. Используйте `will-change` только перед началом анимации**

```css
.element {
  will-change: transform;
}

/* Или динамически */
.element:hover {
  will-change: transform;
}
```

**6. Проверяйте производительность через DevTools, а не по субъективным ощущениям**

```text
Запись Performance
  ↓
Анализ Layout/Paint/Composite
  ↓
Оптимизация
  ↓
Повторная проверка
```

---

## 21.11 Итоги главы

1. **Rendering Pipeline:** Style → Layout → Paint → Composite

2. **Чем позже стадия, тем дешевле изменение:** Composite (дешевле всего) → Paint → Layout (дороже всего)

3. **Style Recalculation** — разрешение каскада, вычисление значений

4. **Layout** — самая дорогая стадия (width, height, top, left, margin)

5. **Paint** — отрисовка пикселей (background, color, border, shadow)

6. **Composite** — объединение слоёв (transform, opacity) — самая дешёвая

7. **contain** — изоляция области пересчёта

8. **content-visibility** — пропуск рендеринга вне viewport

9. **contain-intrinsic-size** — резервирование места (борьба с CLS)

10. **will-change** — подсказка браузеру, но с осторожностью

11. **Производительные анимации** — transform и opacity

12. **DevTools** — основной инструмент анализа производительности

---

**Главная мысль:** Rendering Pipeline — это фундамент, на котором строится вся современная производительность CSS. Понимание последовательности **Style → Layout → Paint → Composite** позволяет осознанно выбирать свойства, проектировать компоненты и создавать интерфейсы, которые остаются плавными даже при высокой сложности приложения.

Для большинства проектов 2026 года ключевыми инструментами оптимизации являются `contain`, `content-visibility`, `contain-intrinsic-size`, каскадные слои, производительные анимации на основе `transform` и `opacity`, а также регулярное профилирование в DevTools. Именно эти механизмы позволяют превратить производительность из набора эмпирических приёмов в системную инженерную дисциплину.

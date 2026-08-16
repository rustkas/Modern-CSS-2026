# Глава 8. CSS Anchor Positioning: нативная геометрия плавающих интерфейсов

> Модель **CSS Anchor Positioning API** стала одним из ключевых прорывов в веб-платформе, навсегда изменив подход к созданию плавающих интерфейсных элементов. Исторически реализация всплывающих подсказок (*tooltips*), выпадающих списков (*dropdowns*), контекстных меню и плашек требовала подключения громоздких JavaScript-библиотек для динамического вычисления координат и отслеживания прокрутки. Теперь нативный браузерный движок берёт эту сложнейшую алгоритмическую задачу на себя, обеспечивая эталонную производительность и абсолютную декларативность кода.

> **Статус на момент написания книги:** технология достигла статуса **Baseline Newly available** в январе 2026 года, когда её поддержка появилась в Firefox 147 без флагов (13 января 2026 года). Chrome и Edge поддерживают Anchor Positioning с версии 125 (май 2024 года), а вот Safari — заметно позже и под другой схемой нумерации: полная поддержка появилась только в **Safari 26** (сентябрь 2025 года, после того как Apple перешла на нумерацию версий Safari по году релиза ОС). Версии линейки Safari 18.x, включая 18.2–18.4, Anchor Positioning не поддерживают вообще. Это значит, что технология уже готова к использованию в продакшне для широкой аудитории, но проект с требованиями к поддержке Safari младше 26-й версии всё ещё нуждается в запасном варианте — подробности в разделе 8.5.

---

## 8.1 Архитектура якорного позиционирования

### Декларативная связь между элементами

В основе новой модели лежит жёсткая декларативная связь между двумя сущностями: **якорем** (*anchor*) и **позиционируемым элементом** (*anchored element*).

```text
Якорь (anchor)
  ↓
элемент, к которому привязываемся
  ↓
Позиционируемый элемент
  ↓
элемент, который позиционируем
```

### Три шага для создания якорной связи

**1. Регистрация якоря**

Любой элемент DOM-дерева может быть назначен якорем с помощью свойства `anchor-name`, значение которого обязательно должно начинаться с двух дефисов — это так называемый *dashed-ident*, тот же формат, что используется для пользовательских свойств:

```css
.trigger-button {
  anchor-name: --menu-trigger;
}
```

**2. Установка связи**

Позиционируемый элемент (обязательно находящийся вне нормального потока — с `position: absolute` или `position: fixed`) привязывается к указанному якорю через свойство `position-anchor`:

```css
.dropdown {
  position: absolute;
  position-anchor: --menu-trigger;
}
```

**3. Позиционирование через функцию `anchor()`**

Главный инструмент соотнесения геометрии. Функция `anchor()` позволяет переносить координаты одной стороны якоря на соответствующую сторону плавающего элемента:

```css
.dropdown {
  position: absolute;
  position-anchor: --menu-trigger;
  top: anchor(bottom);   /* подсказка под кнопкой */
  left: anchor(left);    /* выравнивание по левому краю */
}
```

### Почему это работает

```text
Раньше (JavaScript):
  ↓
getBoundingClientRect()
  ↓
расчёт координат
  ↓
обновление стилей
  ↓
отслеживание изменений
  ↓
Сейчас (CSS):
  ↓
браузер сам вычисляет координаты
  ↓
во время Layout
  ↓
без JavaScript
```

**Ключевое преимущество:** два элемента не обязаны быть siblings, иметь общего родителя или какую-либо DOM-связь. Браузер разрешает привязку во время Layout, используя только имя якоря.

---

## 8.2 Функция `anchor()` и `anchor-size()`

### Доступные значения `anchor()`

Функция `anchor()` позволяет получить координаты краёв якоря:

```css
.element {
  /* Края якоря */
  top: anchor(top);
  bottom: anchor(bottom);
  left: anchor(left);
  right: anchor(right);
  
  /* Центры */
  left: anchor(center);    /* центр по горизонтали */
  top: anchor(center);     /* центр по вертикали */
  
  /* Проценты */
  left: anchor(25%);       /* 25% от ширины якоря */
  top: anchor(75%);        /* 75% от высоты якоря */
}
```

### `anchor-size()` — размеры якоря

Функция `anchor-size()` позволяет позиционируемому элементу считывать габариты якоря:

```css
.dropdown {
  position: absolute;
  position-anchor: --menu-trigger;
  
  /* Ширина как у кнопки */
  width: anchor-size(width);
  
  /* Минимальная ширина как у кнопки */
  min-width: anchor-size(width);
  
  /* Высота как у кнопки */
  height: anchor-size(height);
}
```

### Использование в `calc()`

```css
.tooltip {
  position: absolute;
  position-anchor: --target;
  
  /* 10px отступа от низа якоря */
  top: calc(anchor(bottom) + 10px);
  
  /* Центрирование с учётом половины ширины */
  left: calc(anchor(center) - 50%);
  
  /* Ширина на 20px меньше ширины якоря */
  width: calc(anchor-size(width) - 20px);
}
```

### Идеальное центрирование через `anchor-center`

Специальное значение `anchor-center` для свойств выравнивания позволяет мгновенно центрировать плавающий элемент относительно центральной оси якоря:

```css
.tooltip {
  position: absolute;
  position-anchor: --target;
  
  /* Центрирование по горизонтали */
  justify-self: anchor-center;
  
  /* Центрирование по вертикали */
  align-self: anchor-center;
  
  /* Или оба сразу */
  place-self: anchor-center;
}
```

---

## 8.3 Интеллектуальная адаптивность и `@position-try`

### Проблема краёв экрана

Когда плавающий элемент упирается в границы экрана, он должен автоматически менять позицию:

```text
Кнопка внизу экрана
  ↓
Меню открывается вниз
  ↓
Не помещается → переворачивается вверх
```

### `position-try-fallbacks`

Разработчик может задекларировать набор альтернативных вариантов размещения:

```css
.dropdown {
  position: absolute;
  position-anchor: --menu-trigger;
  top: anchor(bottom);
  left: anchor(left);
  
  /* Автоматический переворот */
  position-try-fallbacks: flip-block, flip-inline;
}
```

**Доступные ключевые слова:**

| Ключевое слово | Действие |
|----------------|----------|
| `flip-block` | Переворот по блочной оси (верх ↔ низ) |
| `flip-inline` | Переворот по инлайновой оси (лево ↔ право) |
| `flip-block flip-inline` | Переворот по обеим осям |

### Кастомные fallback-позиции

```css
/* Определяем кастомную позицию */
@position-try --above-right {
  top: anchor(top);
  bottom: anchor(bottom);
  left: anchor(right);
  margin-left: 10px;
}

@position-try --below-left {
  top: anchor(bottom);
  left: anchor(left);
  margin-top: 10px;
}

/* Используем кастомные fallback */
.tooltip {
  position-try-fallbacks: --above-right, --below-left, flip-block;
}
```

### `position-try-order`

Управляет порядком выбора fallback-позиций:

```css
.dropdown {
  position-try-order: most-block-size;
  /* Выбирает позицию с наибольшим доступным пространством */
}
```

### Важное поведение в разных браузерах

**Современное поведение (Chrome, Firefox, Safari 26+):** если элемент перешёл на fallback-позицию, он остаётся в ней, пока она не переполнится.

**Более ранние версии Safari (18.x и ниже):** Anchor Positioning не поддерживается вообще — ни базовое размещение, ни `@position-try`. Разница в поведении, о которой стоит помнить, актуальна между Safari 26 и последующими минорными обновлениями внутри 26-й линейки, а не между версиями 18.x.


---

## 8.4 Синергия с Popover API

### Две независимые технологии

Anchor Positioning создано для совместной работы с *Popover API* и нативными диалогами, образуя единую экосистему интерактивных оверлеев:

```text
Popover API
  ↓
отвечает за показ/скрытие
  ↓
обработку Escape
  ↓
закрытие по клику вне
  ↓
рендеринг в Top Layer
  ↓
Anchor Positioning
  ↓
отвечает за геометрию
  ↓
где именно появится элемент
```

### Полный пример

```html
<!-- Кнопка-триггер -->
<button popovertarget="menu" class="menu-btn">
  Меню ▼
</button>

<!-- Выпадающее меню -->
<ul id="menu" popover class="dropdown">
  <li><a href="/about">О нас</a></li>
  <li><a href="/blog">Блог</a></li>
  <li><a href="/contact">Контакты</a></li>
</ul>
```

```css
.menu-btn {
  anchor-name: --menu-anchor;
}

.dropdown {
  position: absolute;
  position-anchor: --menu-anchor;
  
  top: anchor(bottom);
  left: anchor(left);
  
  /* Ширина как у кнопки */
  width: anchor-size(width);
  min-width: 140px;
  
  margin-top: 4px;
  
  /* Авто-переворот */
  position-try-fallbacks: flip-block;
}

/* Сброс margin у popover */
[popover] {
  margin: unset;
}
```

### Почему нужен `margin: unset`

Браузер по умолчанию применяет к popover-элементам `margin: auto` для центрирования. При использовании Anchor Positioning это мешает:

```css
/* Без этого правила popover будет смещён */
[popover] {
  margin: unset;
}
```

**Будущее:** CSSWG работает над решением, которое автоматически отключает `margin: auto` при использовании `position-area` или anchor-позиционирования.

### Автоматическое определение якоря

Когда popover открывается через `popovertarget`, браузер устанавливает неявный якорь на popover, указывающий на кнопку:

```css
/* Можно опустить явный position-anchor */
.dropdown {
  position: absolute;
  top: anchor(bottom);
  left: anchor(left);
}
```

**Рекомендация:** писать `position-anchor` явно — это сохраняет читаемость и избегает сюрпризов.

---

## 8.5 Практические подводные камни (Gotchas)

Три ловушки, с которыми чаще всего сталкиваются на практике при внедрении Anchor Positioning, стоит знать заранее.

### Gotcha 1: базовое размещение без `@position-try` в ранних сборках Safari 26

**Проблема:** в первых сборках Safari 26 базовое размещение (`anchor-name`, `position-anchor`, `anchor()`) уже работало, но `@position-try` в самых ранних минорных версиях 26-й линейки поддерживался не полностью — поведение стоит перепроверять по актуальной таблице на webstatus.dev или caniuse, поскольку Apple обновляет минорные версии Safari 26.x достаточно часто.

**Решение:** всегда явно задавайте безопасную базовую позицию элемента вне блока `@position-try` — это защищает от любых пробелов в реализации `@position-try`, независимо от конкретной минорной версии:

```css
/* ✅ Безопасно для любых версий с поддержкой Anchor Positioning */
.tooltip {
  position: absolute;
  position-anchor: --btn;
  
  /* Базовая позиция — работает везде, где есть базовая поддержка */
  top: anchor(bottom);
  left: anchor(center);
  transform: translateX(-50%);
  margin-top: 6px;
  
  /* Игнорируется там, где нет @position-try, применяется там, где есть */
  position-try-fallbacks: flip-block;
}

/* ❌ Сломается там, где @position-try не поддерживается */
@position-try --default {
  top: anchor(bottom);
  left: anchor(center);
}

.tooltip {
  position-try-fallbacks: --default;
  /* Базовой позиции нет! */
}
```


### Gotcha 2: `popover="hint"` — Chrome-only

**Проблема:** `popover="hint"` (режим для подсказок при наведении) в середине 2026 года поддерживается только в Chrome. Firefox и Safari его не поддерживают.

```html
<!-- ❌ popover="hint" — только Chrome -->
<div id="tip" popover="hint">Подсказка</div>

<!-- ✅ popover="auto" — кросс-браузерно для кликов -->
<button popovertarget="menu">Открыть</button>
<ul id="menu" popover>...</ul>
```

**Решение для hover-тултипов:**

```css
/* ✅ CSS :hover — работает везде */
.trigger {
  position: relative;
}

.tooltip {
  position: absolute;
  visibility: hidden;
  opacity: 0;
  transition: opacity 0.2s ease;
}

.trigger:hover .tooltip,
.trigger:focus-visible .tooltip {
  visibility: visible;
  opacity: 1;
}
```

### Gotcha 3: Повторяющиеся компоненты и `anchor-scope`

**Проблема:** если один и тот же якорь используется в нескольких повторяющихся компонентах (например, карточки в списке), по умолчанию все они разрешаются в последний элемент DOM с этим именем.

```html
<!-- Все три tooltip будут указывать на последнюю карточку -->
<div class="card">
  <button class="trigger">?</button>
  <div class="tooltip">Подсказка 1</div>
</div>
<div class="card">
  <button class="trigger">?</button>
  <div class="tooltip">Подсказка 2</div>
</div>
<div class="card">
  <button class="trigger">?</button>
  <div class="tooltip">Подсказка 3</div>
</div>
```

```css
/* ❌ Все используют один якорь */
.trigger {
  anchor-name: --info-tooltip;
}
.tooltip {
  position-anchor: --info-tooltip;
  /* Все указывают на последний .trigger */
}
```

**Решение:** `anchor-scope` ограничивает область действия имени якоря:

```css
/* ✅ Каждый экземпляр изолирован */
.card {
  anchor-scope: all;
  /* --info-tooltip работает только внутри этого .card */
}

.card .trigger {
  anchor-name: --info-tooltip;
}

.card .tooltip {
  position-anchor: --info-tooltip;
}
```

### Gotcha 4: Popover по умолчанию центрируется

**Проблема:** браузер применяет к popover-элементам `margin: auto` для центрирования в Top Layer.

**Решение:** сбрасываем margin:

```css
[popover] {
  margin: unset;
}
```

---

## 8.6 Проверка поддержки и Progressive Enhancement

### Проверка через `@supports`

```css
@supports (anchor-name: --test) {
  .tooltip {
    position: absolute;
    position-anchor: --my-anchor;
    top: anchor(bottom);
    left: anchor(left);
    position-try-fallbacks: flip-block;
  }
}

/* Fallback для браузеров без поддержки */
@supports not (anchor-name: --test) {
  .tooltip {
    display: none;
  }
  /* Или использование JS-библиотеки */
}
```

### Проверка в JavaScript

```javascript
const supportsAnchorPositioning = CSS.supports('anchor-name', '--test');

if (!supportsAnchorPositioning) {
  // Подгружаем полифилл или JS-библиотеку
  const script = document.createElement('script');
  script.src = 'https://unpkg.com/@oddbird/css-anchor-positioning';
  document.head.appendChild(script);
}
```

### Стратегия Progressive Enhancement

```text
Базовый HTML
  ↓
работает всегда
  ↓
Базовый CSS (fallback)
  ↓
работает всегда
  ↓
Anchor Positioning (улучшение)
  ↓
работает в современных браузерах
  ↓
Полифилл (при необходимости)
  ↓
для старых браузеров
```

### Полифилл

Полифилл от OddBird (`@oddbird/css-anchor-positioning`, ~8KB gzipped) добавляет полную поддержку Anchor Positioning для старых браузеров:

```html
<script>
  if (!CSS.supports('anchor-name: --a')) {
    const s = document.createElement('script');
    s.src = 'https://unpkg.com/@oddbird/css-anchor-positioning';
    document.head.appendChild(s);
  }
</script>
```

**Преимущество:** современные браузеры ничего не загружают; старые браузеры получают 8KB полифилл — это лучше, чем грузить 12KB Floating UI всем пользователям.

---

## 8.7 Сравнение с JavaScript-библиотеками

### Когда использовать CSS Anchor Positioning

**CSS Anchor Positioning покрывает ~90% случаев использования Floating UI:**

| Сценарий | Лучшее решение | Почему |
|----------|---------------|--------|
| Базовая подсказка (клик/фокус) | CSS Anchor + popover | 0 JS, нативная доступность, Top Layer |
| Выпадающее меню | CSS Anchor + popover | `anchor-size()` — ширина как у кнопки, `flip-block` — авто-переворот |
| Hover-тултип | CSS `:hover` + `:focus-visible` | `popover="hint"` пока Chrome-only |
| Контекстное меню | CSS Anchor + popover | Дешево, быстро, декларативно |

### Когда всё ещё нужен JavaScript (Floating UI)

| Сценарий | Почему нужен JS |
|----------|-----------------|
| Виртуальные списки | Якорь может быть размонтирован |
| Cross-Shadow-DOM | `anchor-name` не пересекает границы Shadow DOM |
| Многоуровневые динамические меню | Контент подгружается по требованию |
| IE11 / очень старые Safari | Полифилл добавляет 8KB |

### Сравнение производительности

```text
Floating UI
  ↓
getBoundingClientRect() (вызывает reflow)
  ↓
расчёт в JavaScript
  ↓
обновление DOM
  ↓
пересчёт Layout
  ↓
~8ms на устройстве среднего класса
  ↓
CSS Anchor Positioning
  ↓
расчёт во время Layout (нативный C++)
  ↓
без JavaScript
  ↓
<1ms на устройстве среднего класса
```

**Реальный бенчмарк:** на странице с 20 тултипами замена Floating UI на CSS Anchor Positioning сократила bundle на 11.4KB и снизила задержку отрисовки с 8ms до <1ms на среднестатистическом телефоне.

---

## 8.8 Практические паттерны

### Tooltip

```css
/* HTML: <button class="trigger" data-tooltip="Подсказка">Hover</button> */

.trigger {
  anchor-name: --tooltip-anchor;
  position: relative;
}

.trigger::after {
  content: attr(data-tooltip);
  position: absolute;
  position-anchor: --tooltip-anchor;
  
  top: anchor(bottom);
  left: anchor(center);
  transform: translateX(-50%);
  
  padding: 4px 8px;
  background: #333;
  color: white;
  border-radius: 4px;
  font-size: 12px;
  white-space: nowrap;
  
  /* По умолчанию скрыт */
  opacity: 0;
  visibility: hidden;
  transition: opacity 0.2s ease;
  
  position-try-fallbacks: flip-block;
}

.trigger:hover::after,
.trigger:focus-visible::after {
  opacity: 1;
  visibility: visible;
}
```

### Dropdown

```css
/* HTML:
<button popovertarget="menu" class="menu-btn">Меню ▼</button>
<ul id="menu" popover class="dropdown">...</ul>
*/

.menu-btn {
  anchor-name: --menu-anchor;
}

.dropdown {
  position: absolute;
  position-anchor: --menu-anchor;
  
  top: anchor(bottom);
  left: anchor(left);
  
  width: anchor-size(width);
  min-width: 140px;
  
  margin-top: 4px;
  padding: 8px 0;
  background: white;
  border: 1px solid #ddd;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
  
  position-try-fallbacks: flip-block;
}

[popover] {
  margin: unset;
}
```

### Context Menu

```css
/* HTML: <div popover id="menu" class="context-menu">...</div> */

.context-menu {
  position: absolute;
  position-anchor: --context-trigger;
  
  top: anchor(bottom);
  left: anchor(left);
  
  padding: 8px 0;
  background: white;
  border: 1px solid #ddd;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
  min-width: 180px;
  
  position-try-fallbacks: flip-block, flip-inline;
}

[popover] {
  margin: unset;
}
```

### Popover с Anchor Positioning

```css
/* HTML:
<button popovertarget="panel" class="trigger">Открыть</button>
<div id="panel" popover class="panel">...</div>
*/

.trigger {
  anchor-name: --panel-anchor;
}

.panel {
  position: absolute;
  position-anchor: --panel-anchor;
  
  top: anchor(bottom);
  left: anchor(center);
  transform: translateX(-50%);
  
  padding: 16px;
  background: white;
  border: 1px solid #ddd;
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0,0,0,0.15);
  min-width: 300px;
  max-width: 500px;
  
  position-try-fallbacks: flip-block;
}

[popover] {
  margin: unset;
}
```

---

## 8.9 Итоги главы

После изучения главы читатель должен понимать, что:

1. **Anchor Positioning — нативная геометрия браузера** — без JavaScript, без `getBoundingClientRect()`, без `ResizeObserver`

2. **Три шага:** `anchor-name` (якорь) → `position-anchor` (связь) → `anchor()` (позиционирование)

3. **`anchor-size()`** — позволяет копировать размеры якоря (например, ширина выпадающего списка равна ширине кнопки)

4. **`anchor-center`** — мгновенное центрирование относительно якоря

5. **`@position-try` и `position-try-fallbacks`** — авто-переворот при нехватке места (заменяет `flip()` middleware из Floating UI)

6. **Popover + Anchor Positioning** — полная система оверлеев без JavaScript (показ/скрытие + позиционирование)

7. **Три главных gotcha:**
   - Ранние минорные версии Safari 26.x могут не полностью поддерживать `@position-try` → пишите базовую позицию
   - `popover="hint"` — Chrome-only → используйте CSS `:hover`
   - Повторяющиеся компоненты → используйте `anchor-scope`

8. **Anchor Positioning — Baseline Newly available с января 2026 года** — Chrome/Edge с версии 125, Firefox с версии 147, Safari с версии 26 (обратите внимание: Safari перешёл на новую схему нумерации версий, синхронизированную с годом релиза ОС, поэтому «Safari 26» — это актуальная, а не будущая версия)

9. **Progressive Enhancement** — проверка через `@supports` + полифилл при необходимости

10. **Когда всё ещё нужен JavaScript:** виртуальные списки, cross-Shadow-DOM, динамические многоуровневые меню

---

**Главная мысль:** CSS Anchor Positioning API закрывает многолетнюю потребность веб-разработки в нативных плавающих интерфейсах. Переложив расчёты геометрии, перенос координат и обработку краевых эффектов на движок браузера, инженеры получили инструмент для создания сложнейших интерактивных компонентов с минимальными накладными расходами на JavaScript.

**Практическое правило на 2026 год:** начинайте с CSS Anchor Positioning как решения по умолчанию для всех новых тултипов и выпадающих списков. Это ничего не стоит, ничего не грузит, а браузерный движок обрабатывает геометрию эффективнее любой JavaScript-библиотеки. Когда вы столкнётесь с одним из трёх краевых случаев — виртуальные списки, cross-Shadow-DOM или динамические вложенные меню — переключитесь на Floating UI для этого конкретного компонента и оставьте всё остальное на CSS.

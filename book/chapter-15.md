# Глава 15. Motion Design в современном CSS

> Современный Motion Design выходит далеко за пределы традиционного понимания анимации как набора переходов между двумя состояниями. Сегодня браузер предоставляет полноценную систему описания движения, объединяющую переходы, временные шкалы, навигацию, изменение размеров компонентов и адаптацию к пользовательским предпочтениям. Большинство сценариев, которые ещё несколько лет назад реализовывались исключительно средствами JavaScript, теперь описываются декларативно средствами CSS и связанных Web API.

Главная задача современной анимации — не украшение интерфейса, а повышение его понятности, пространственной связности и отзывчивости.

---

## 15.1 Motion как часть архитектуры интерфейса

### От декорации к архитектуре

В современной разработке движение рассматривается как такой же архитектурный уровень, как типографика, цвет или сетка.

```text
Motion Design
  ↓
Объясняет изменения состояния
  ↓
Показывает причинно-следственные связи
  ↓
Помогает сохранять пространственную ориентацию
  ↓
Делает взаимодействие предсказуемым
```

### Функции движения в интерфейсе

**1. Объяснение изменений**

```text
Элемент исчезает
  ↓
Анимация показывает, куда он ушёл
  ↓
Пользователь понимает, что произошло
```

**2. Связь между состояниями**

```text
Кнопка → Диалог
  ↓
Анимация показывает, что диалог появился из кнопки
  ↓
Пространственная связь сохраняется
```

**3. Обратная связь**

```text
Нажатие → Эффект
  ↓
Анимация подтверждает действие
  ↓
Пользователь знает, что нажатие зарегистрировано
```

**4. Навигация**

```text
Страница A → Страница B
  ↓
Анимация показывает переход
  ↓
Пользователь сохраняет ориентацию в пространстве
```

### Motion в дизайн-системах

Большинство современных дизайн-систем рассматривают анимацию как обязательный компонент интерфейса:

```text
Material Design 3
  ↓
Motion — часть системы
  ↓
Fluent Design
  ↓
Движение как фундаментальный принцип
  ↓
Human Interface Guidelines
  ↓
Анимация для связи и контекста
```

---

## 15.2 Новое поколение CSS Transitions

### Традиционные ограничения

Долгое время CSS Transitions не могли анимировать переходы к intrinsic-размерам (размерам, определяемым содержимым):

```css
/* ❌ Не работало */
.panel {
  height: 0;
  overflow: hidden;
  transition: height 0.3s ease;
}

.panel.open {
  height: auto; /* Не анимируется */
}
```

### `interpolate-size`: анимация intrinsic-размеров

Наиболее заметной возможностью стало появление свойства `interpolate-size`, которое позволяет анимировать переходы между фиксированными размерами и внутренними (*intrinsic*) размерами элемента.

```css
:root {
  interpolate-size: allow-keywords;
}

.panel {
  height: 0;
  overflow: hidden;
  transition: height 0.3s ease;
}

.panel.open {
  height: auto; /* Теперь анимируется! */
}
```

**Поддержка:** на момент написания книги свойство `interpolate-size` является экспериментальным. Рекомендуется проверять его поддержку через `@supports`:

```css
@supports (interpolate-size: allow-keywords) {
  :root {
    interpolate-size: allow-keywords;
  }
  
  .panel.open {
    height: auto;
  }
}

@supports not (interpolate-size: allow-keywords) {
  /* Fallback: анимация через max-height */
  .panel {
    max-height: 0;
    transition: max-height 0.3s ease;
  }
  .panel.open {
    max-height: 1000px;
  }
}
```

### `calc-size()`: вычисление размеров

Вместе с `interpolate-size` появляется функция `calc-size()`, которая позволяет выполнять вычисления с intrinsic-размерами:

```css
.element {
  height: calc-size(auto, size + 20px);
  /* Высота = auto + 20px */
  
  width: calc-size(min-content, size * 2);
  /* Ширина = min-content * 2 */
}
```

**Это позволяет создавать:**

- аккордеоны с автоматической высотой
- раскрывающиеся меню
- панели с анимированным содержимым
- карточки с динамическими размерами
- боковые панели

**Без измерения размеров через JavaScript.**

---

## 15.3 Scroll-driven Animations

### От времени к положению

Ранее прогресс анимации определялся исключительно временем. Теперь источником времени может стать сам документ.

```text
Time-driven Animation
  ↓
Прогресс зависит от времени
  ↓
0% → 100% за N секунд
  ↓
Scroll-driven Animation
  ↓
Прогресс зависит от скролла
  ↓
0% → 100% от начала до конца прокрутки
```

### Типы временных шкал

**1. Scroll Timeline — привязана к скроллу контейнера**

```css
@keyframes progress {
  from { width: 0%; }
  to { width: 100%; }
}

.progress-bar {
  animation: progress linear;
  animation-timeline: scroll();
  /* Прогресс = положение скролла */
}
```

**2. View Timeline — привязана к видимости элемента**

```css
@keyframes reveal {
  from { opacity: 0; transform: translateY(50px); }
  to { opacity: 1; transform: translateY(0); }
}

.card {
  animation: reveal linear;
  animation-timeline: view();
  /* Прогресс = видимость элемента */
}
```

### Практические применения

**Индикатор чтения:**

```css
@keyframes reading-progress {
  from { transform: scaleX(0); }
  to { transform: scaleX(1); }
}

.reading-indicator {
  animation: reading-progress linear;
  animation-timeline: scroll();
  transform-origin: left;
}
```

**Появление карточек при скролле:**

```css
@keyframes card-reveal {
  from {
    opacity: 0;
    transform: translateY(50px) scale(0.95);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

.card {
  animation: card-reveal linear;
  animation-timeline: view();
  animation-range: entry 0% entry 50%;
}
```

**Параллакс:**

```css
@keyframes parallax {
  from { transform: translateY(0); }
  to { transform: translateY(-50px); }
}

.hero-image {
  animation: parallax linear;
  animation-timeline: scroll();
}
```

### Ключевое преимущество

**Прокрутка назад автоматически перематывает анимацию.**

```text
Пользователь скроллит вниз
  ↓
Анимация идёт вперёд
  ↓
Пользователь скроллит вверх
  ↓
Анимация идёт назад
  ↓
Никакого JavaScript
```

---

## 15.4 View Transitions

### Революция в навигации

Одним из крупнейших изменений платформы стало появление View Transitions. Ранее смена экранов требовала сложной координации JavaScript.

```text
Раньше:
  ↓
JavaScript управляет переходом
  ↓
Координация между состояниями
  ↓
Сложно, много кода
  ↓
Сейчас:
  ↓
Браузер управляет переходом
  ↓
Автоматическая интерполяция
  ↓
Просто, декларативно
```

### Как это работает

Браузер самостоятельно:

1. Делает снимки состояний
2. Сопоставляет элементы
3. Интерполирует их положение
4. Выполняет плавный переход

```css
/* Базовая настройка */
@view-transition {
  navigation: auto;
}

/* Кастомный переход */
::view-transition-old(root) {
  animation: slide-out 0.5s ease;
}

::view-transition-new(root) {
  animation: slide-in 0.5s ease;
}

@keyframes slide-out {
  from { transform: translateX(0); }
  to { transform: translateX(-100%); }
}

@keyframes slide-in {
  from { transform: translateX(100%); }
  to { transform: translateX(0); }
}
```

### Shared Elements

View Transitions позволяют сопоставлять элементы между состояниями:

```html
<!-- Состояние 1: список -->
<div class="gallery">
  <img src="thumb-1.jpg" style="view-transition-name: hero">
  <img src="thumb-2.jpg">
</div>

<!-- Состояние 2: детальный просмотр -->
<div class="detail">
  <img src="full-1.jpg" style="view-transition-name: hero">
</div>
```

```css
/* Браузер автоматически анимирует переход */
img[style*="view-transition-name: hero"] {
  view-transition-name: hero;
}
```

### Где работают View Transitions

```text
SPA (Single Page Applications)
  ↓
Переходы между компонентами
  ↓
MPA (Multi Page Applications)
  ↓
Переходы между страницами
  ↓
Изменение состояния интерфейса
  ↓
Списки → Детали
  ↓
Галереи → Просмотр
```

**Благодаря этому современные сайты начинают вести себя подобно нативным приложениям.**

> **Важное уточнение статуса поддержки:** View Transitions делятся на два независимых сценария с разной зрелостью.
>
> * **Same-document (SPA)** переходы через `document.startViewTransition()` в JavaScript — поддерживаются уже во всех core-браузерах: Chrome/Edge 111+, Safari 18+, Firefox 133+/144+. Это Baseline-готовая возможность.
> * **Cross-document (MPA)** переходы через декларативный at-rule `@view-transition { navigation: auto; }`, показанный в примерах этого раздела, — остаются **Limited availability**: поддерживаются в Chrome/Edge 126+ и Safari 18.2+, но Firefox их вообще не реализует (at-rule просто игнорируется, и навигация происходит как обычно, без анимации).
>
> На практике это значит: пример с `@view-transition` в начале раздела — это прогрессивное улучшение по своей природе, а не универсально работающий код. В браузерах без поддержки cross-document переходов страница просто переключается мгновенно, без анимации и без ошибок — деградация полностью безопасна и не требует `@supports`-обёртки. Но при планировании MPA-навигации важно закладывать этот сценарий как ожидаемый для пользователей Firefox, а не как редкое исключение.

---

## 15.5 Программируемые анимации

### `@property` — анимация пользовательских свойств

CSS постепенно превращается в язык описания поведения. Ключевую роль здесь играет регистрация пользовательских свойств.

```css
@property --progress {
  syntax: '<number>';
  inherits: false;
  initial-value: 0;
}

@property --rotation {
  syntax: '<angle>';
  inherits: false;
  initial-value: 0deg;
}
```

После регистрации браузер начинает понимать тип значения. Это открывает возможность анимировать:

- пользовательские переменные
- градиенты
- фильтры
- сложные вычисления
- компоненты дизайн-систем

### Примеры программируемых анимаций

**Анимация градиента:**

```css
@property --gradient-position {
  syntax: '<percentage>';
  inherits: false;
  initial-value: 0%;
}

.element {
  --gradient-position: 0%;
  background: linear-gradient(
    to right,
    var(--color-primary),
    var(--color-secondary)
  );
  background-position: var(--gradient-position);
  transition: --gradient-position 1s ease;
}

.element:hover {
  --gradient-position: 100%;
}
```

**Анимация фильтра:**

```css
@property --blur {
  syntax: '<length>';
  inherits: false;
  initial-value: 0px;
}

.element {
  --blur: 0px;
  filter: blur(var(--blur));
  transition: --blur 0.5s ease;
}

.element:hover {
  --blur: 5px;
}
```

### `animation-composition` — композиция анимаций

Несколько независимых анимаций могут одновременно воздействовать на одно свойство без взаимного конфликта.

```css
.element {
  animation: 
    rotate 2s linear infinite,
    pulse 1s ease-in-out infinite;
  animation-composition: 
    add,  /* Складываем значения */
    add;  /* Складываем значения */
}

@keyframes rotate {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

@keyframes pulse {
  from { transform: scale(1); }
  to { transform: scale(1.2); }
}

/* Результат: rotate + pulse одновременно */
```

### Typed OM / Houdini

Typed OM позволяет работать с CSS-значениями как с типизированными объектами в JavaScript:

```javascript
// Без Typed OM — строки
element.style.setProperty('--rotation', '45deg');
const rotation = element.style.getPropertyValue('--rotation'); // '45deg'

// С Typed OM — объекты
element.attributeStyleMap.set('--rotation', CSS.deg(45));
const rotation = element.attributeStyleMap.get('--rotation');
// CSSUnitValue { value: 45, unit: 'deg' }
```

Houdini позволяет создавать кастомные анимации, работающие на уровне браузерного движка.

---

## 15.6 Новые функции плавности

### `linear()` — произвольные кривые

Функция `linear()` значительно расширила возможности управления скоростью движения.

```css
.element {
  animation: move 1s linear(
    0,         /* 0% — начало */
    0.1 20%,   /* 20% — почти ноль */
    0.5 40%,   /* 40% — половина */
    0.9 80%,   /* 80% — почти конец */
    1          /* 100% — конец */
  );
}
```

В отличие от `cubic-bezier()`, `linear()` позволяет описывать произвольную кривую через множество контрольных точек.

**Это делает возможным моделирование:**

- инерции
- замедления
- пружины
- bounce
- сложных физических эффектов

**Без JavaScript.**

### Пример: эффект пружины

```css
@keyframes spring {
  0% { transform: scale(0); }
  30% { transform: scale(1.2); }
  50% { transform: scale(0.9); }
  70% { transform: scale(1.05); }
  100% { transform: scale(1); }
}

.element {
  animation: spring 0.8s ease;
}
```

### `steps()` — дискретные анимации

Функция `steps()` позволяет создавать пошаговые анимации:

```css
.element {
  animation: walk 1s steps(4);
}

@keyframes walk {
  from { background-position: 0; }
  to { background-position: -200px; }
}
```

---

## 15.7 Производительность Motion

### Rendering Pipeline

Современная анимация должна учитывать устройство рендеринга браузера.

```text
Style
  ↓
Layout
  ↓
Paint
  ↓
Composite
```

### Что анимировать

**Рекомендуется анимировать:**

```text
transform
  ↓
Работает на GPU
  ↓
Не вызывает Layout
  ↓
Быстро
  ↓
opacity
  ↓
Работает на GPU
  ↓
Не вызывает Paint
  ↓
Быстро
  ↓
Зарегистрированные @property
  ↓
Оптимизированы браузером
  ↓
Быстро
```

**Следует избегать постоянной анимации:**

```text
width
  ↓
Вызывает Layout
  ↓
Медленно
  ↓
height
  ↓
Вызывает Layout
  ↓
Медленно
  ↓
top / left
  ↓
Вызывает Layout
  ↓
Медленно
  ↓
margin
  ↓
Вызывает Layout
  ↓
Медленно
```

### Оптимизация через CSS

```css
/* Подсказка браузеру */
.element {
  will-change: transform, opacity;
  /* Только если анимация будет происходить */
}

/* Изоляция элемента */
.element {
  contain: layout style paint;
  /* Указывает, что элемент самодостаточен */
}

/* Отложенная отрисовка */
.element {
  content-visibility: auto;
  /* Невидимые элементы не рендерятся */
}
```

### Современные браузеры

Современные браузеры самостоятельно оптимизируют значительную часть подобных сценариев, однако архитектура анимации по-прежнему оказывает решающее влияние на частоту кадров.

---

## 15.8 Доступность Motion

### `prefers-reduced-motion`

Любая современная анимация должна учитывать пользовательские настройки операционной системы.

```css
/* Базовые анимации */
.element {
  animation: slide-in 0.5s ease;
}

/* Учитываем настройки пользователя */
@media (prefers-reduced-motion: reduce) {
  .element {
    animation: none;
    /* Или упрощённый вариант */
    opacity: 1;
    transform: none;
  }
}
```

### Принципы доступной анимации

```text
При уменьшении движения:
  ↓
Декоративные эффекты отключаются
  ↓
Сохраняются только функционально необходимые переходы
  ↓
Навигация остаётся понятной без динамики
```

### Практические рекомендации

```css
/* ✅ Хорошо — учитываем настройки */
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.001ms !important;
    transition-duration: 0.001ms !important;
  }
}

/* ✅ Хорошо — только функциональные анимации */
@media (prefers-reduced-motion: reduce) {
  .dialog {
    animation: fade-in 0.001ms;
    /* Минимальная анимация для доступности */
  }
}

/* ❌ Плохо — игнорируем настройки */
.element {
  animation: flash 0.5s infinite;
  /* Пользователь не может отключить */
}
```

**Motion не должен становиться препятствием для использования приложения.**

---

## 15.9 Практические рекомендации

### Архитектурные правила

Современная практика разработки постепенно сформировала несколько устойчивых правил.

**1. Используйте CSS как основной инструмент анимации**

```css
/* ✅ Рекомендуется */
.element {
  transition: transform 0.3s ease;
}

/* ❌ Устаревает */
// JavaScript requestAnimationFrame
```

**2. Применяйте JavaScript только для управления состояниями**

```javascript
// ✅ Хорошо — управление состоянием
element.classList.toggle('open');

// ❌ Плохо — управление анимацией
element.style.transform = `translateX(${x}px)`;
```

**3. Предпочитайте Scroll-driven Animations вместо ручной обработки scroll**

```css
/* ✅ Рекомендуется */
animation-timeline: scroll();

/* ❌ Устаревает */
// JavaScript: window.addEventListener('scroll', ...)
```

**4. Используйте View Transitions для смены экранов**

```css
/* ✅ Рекомендуется */
@view-transition { navigation: auto; }

/* ❌ Устаревает */
// JavaScript-роутер с ручными анимациями
```

**5. Регистрируйте пользовательские свойства через `@property`**

```css
/* ✅ Рекомендуется */
@property --progress {
  syntax: '<number>';
  inherits: false;
  initial-value: 0;
}
```

**6. Анимируйте `transform` и `opacity` вместо геометрических свойств**

```css
/* ✅ Рекомендуется */
transform: scale(1.2);
opacity: 0.5;

/* ❌ Устаревает */
width: 200px;
height: 150px;
```

**7. Всегда поддерживайте `prefers-reduced-motion`**

```css
/* ✅ Рекомендуется */
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.001ms !important;
  }
}
```

**8. Проектируйте Motion как часть дизайн-системы**

```css
/* Motion токены */
:root {
  --motion-duration-fast: 0.15s;
  --motion-duration-medium: 0.3s;
  --motion-duration-slow: 0.5s;
  --motion-easing-standard: ease;
  --motion-easing-spring: cubic-bezier(0.34, 1.56, 0.64, 1);
}
```

---

## 15.10 Итоги главы

1. **Motion как архитектура** — движение — такой же фундаментальный уровень, как типографика или цвет

2. **`interpolate-size` и `calc-size()`** — анимация intrinsic-размеров (аккордеоны, панели)

3. **Scroll-driven Animations** — анимация, привязанная к скроллу и видимости элементов

4. **View Transitions** — плавные переходы между страницами и состояниями

5. **Программируемые анимации** — через `@property`, Typed OM, Houdini

6. **`animation-composition`** — композиция нескольких анимаций на одном свойстве

7. **`linear()`** — произвольные кривые плавности для сложных эффектов

8. **Производительность** — анимируйте `transform` и `opacity`, используйте `will-change`

9. **Доступность** — всегда поддерживайте `prefers-reduced-motion`

10. **Motion в дизайн-системах** — единые токены для всего интерфейса

---

**Главная мысль:** Современный CSS перестал быть лишь языком оформления и превратился в полноценную платформу для проектирования поведения интерфейсов. Переходы, временные шкалы, анимация пользовательских свойств, View Transitions и Scroll-driven Animations образуют единую систему Motion Design, позволяющую реализовывать сложные сценарии взаимодействия без постоянного обращения к JavaScript. Для разработчика 2026 года важнейшей задачей становится уже не создание отдельных эффектов, а проектирование целостной архитектуры движения, в которой анимация повышает понятность интерфейса, остаётся производительной и учитывает требования доступности.

# Глава 17. Декларативные таймлайны: Scroll-driven Animations и Timeline Architecture

> После появления Scroll-driven Animations CSS перестал воспринимать время как единственный источник движения.

До появления этой спецификации все анимации строились вокруг часов: браузер увеличивал значение времени, а CSS вычислял очередной кадр.

Современный CSS использует значительно более общую модель.

Теперь источником прогресса анимации могут быть:

- течение времени;
- положение полосы прокрутки;
- положение элемента во viewport;
- пользовательские именованные таймлайны;
- переходы состояний документа (View Transition API).

В результате таймлайн превращается в самостоятельную архитектурную сущность, а не в скрытый механизм работы `animation`.

---

## 17.1 Новая модель анимации: Timeline как источник прогресса

### Традиционная модель

Традиционная модель CSS основывалась исключительно на времени.

```css
.element {
  animation: fade 500ms linear;
}
```

Здесь прогресс вычисляется исключительно по часам:

```text
Время
  ↓
0ms → 500ms
  ↓
0% → 100%
```

### Современная модель

Современная модель разделяет два понятия:

```text
Что анимируется (@keyframes)
  ↓
Описание изменения свойств
  ↓
Откуда берётся прогресс (animation-timeline)
  ↓
Источник прогресса
```

**Именно это разделение стало одним из крупнейших изменений CSS Animations за последние годы.**

Теперь одна и та же анимация может работать:

- по времени;
- по скроллу;
- по положению элемента;
- по пользовательскому таймлайну.

```css
/* Одна и та же анимация */
@keyframes fade {
  from { opacity: 0; }
  to { opacity: 1; }
}

/* По времени */
.element-time {
  animation: fade 500ms linear;
}

/* По скроллу */
.element-scroll {
  animation: fade linear;
  animation-timeline: scroll();
}

/* По видимости */
.element-view {
  animation: fade linear;
  animation-timeline: view();
}
```

**Это делает CSS значительно ближе к декларативным системам анимации профессиональных графических движков.**

---

## 17.2 Scroll Progress Timeline

### Привязка к прокрутке

Первый тип современных таймлайнов отслеживает положение полосы прокрутки.

```css
.element {
  animation: progress linear;
  animation-timeline: scroll();
}
```

Прогресс анимации изменяется вместе со скроллом документа.

```text
Начало скролла
  ↓
0%
  ↓
Середина скролла
  ↓
50%
  ↓
Конец скролла
  ↓
100%
```

### Применение

**Reading Progress (индикатор чтения):**

```css
@keyframes reading {
  from { transform: scaleX(0); }
  to { transform: scaleX(1); }
}

.progress-bar {
  animation: reading linear;
  animation-timeline: scroll();
  transform-origin: left;
}
```

**Параллакс:**

```css
@keyframes parallax {
  from { transform: translateY(0); }
  to { transform: translateY(-100px); }
}

.hero-image {
  animation: parallax linear;
  animation-timeline: scroll();
}
```

**Прогресс-бар:**

```css
@keyframes fill {
  from { width: 0%; }
  to { width: 100%; }
}

.progress {
  animation: fill linear;
  animation-timeline: scroll();
}
```

### Ключевое свойство: синхронизация

**При прокрутке назад анимация также движется назад. Она полностью синхронизирована со скроллом.**

```text
Пользователь скроллит вниз
  ↓
Анимация идёт вперёд
  ↓
Пользователь скроллит вверх
  ↓
Анимация идёт назад
  ↓
Полная синхронизация
```

---

## 17.3 View Progress Timeline

### Привязка к видимости элемента

Второй механизм ориентирован уже не на документ, а на конкретный элемент.

```css
.element {
  animation: reveal linear;
  animation-timeline: view();
}
```

Теперь прогресс зависит от положения элемента относительно viewport.

```text
Элемент вне viewport (снизу)
  ↓
0%
  ↓
Элемент входит в viewport
  ↓
Начало анимации
  ↓
Элемент в центре viewport
  ↓
Середина анимации
  ↓
Элемент выходит из viewport
  ↓
Конец анимации
```

### Диапазоны (animation-range)

Можно отдельно анимировать:

- появление (`entry`)
- нахождение внутри экрана (`contain`)
- полное покрытие (`cover`)
- исчезновение (`exit`)

```css
/* Появление при входе */
.element {
  animation: reveal linear;
  animation-timeline: view();
  animation-range: entry 0% entry 50%;
}

/* Исчезновение при выходе */
.element {
  animation: fade-out linear;
  animation-timeline: view();
  animation-range: exit 0% exit 50%;
}

/* Полный цикл: появление → показ → исчезновение */
.element {
  animation: reveal linear;
  animation-timeline: view();
  animation-range: entry 0% exit 100%;
}
```

### Практические примеры

**Появление карточек:**

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
  animation-range: entry 0% entry 60%;
}
```

**Стикки-анимация:**

```css
@keyframes sticky-header {
  from { 
    background: transparent;
    box-shadow: none;
  }
  to {
    background: white;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  }
}

.header {
  animation: sticky-header linear;
  animation-timeline: scroll();
  animation-range: 0px 100px;
}
```

**Инфографика:**

```css
@keyframes chart-fill {
  from { height: 0%; }
  to { height: var(--target); }
}

.chart-bar {
  animation: chart-fill linear;
  animation-timeline: view();
  animation-range: entry 0% entry 80%;
}
```

---

## 17.4 Именованные Timeline

### Создание собственных таймлайнов

Настоящая сила спецификации проявляется при использовании собственных таймлайнов.

```css
/* Создаём таймлайн на основе скролла секции */
.section {
  scroll-timeline: --gallery;
}

/* Любой другой элемент может использовать этот таймлайн */
.card {
  animation: reveal linear;
  animation-timeline: --gallery;
}
```

### Синхронизация компонентов

Благодаря этому появляется возможность строить сложные композиции:

```css
/* Секция-контейнер с таймлайном */
.gallery-section {
  scroll-timeline: --gallery;
  overflow-y: scroll;
  height: 100vh;
}

/* Несколько элементов синхронизированы */
.gallery-title {
  animation: fade-in linear;
  animation-timeline: --gallery;
  animation-range: entry 0% entry 50%;
}

.gallery-image {
  animation: zoom linear;
  animation-timeline: --gallery;
  animation-range: entry 0% entry 100%;
}

.gallery-caption {
  animation: slide-up linear;
  animation-timeline: --gallery;
  animation-range: entry 50% entry 100%;
}
```

### View Timeline (именованный)

Для создания таймлайна на основе видимости элемента:

```css
/* Создаём таймлайн на основе видимости */
.hero {
  view-timeline: --hero;
}

/* Другие элементы используют этот таймлайн */
.hero-text {
  animation: reveal linear;
  animation-timeline: --hero;
  animation-range: entry 0% entry 50%;
}

.hero-image {
  animation: scale linear;
  animation-timeline: --hero;
  animation-range: entry 20% entry 80%;
}
```

### timeline-scope

Для управления областью видимости используется `timeline-scope`, который позволяет экспортировать таймлайн выше по DOM.

```css
/* Экспортируем таймлайн на уровень выше */
.parent {
  timeline-scope: --global-timeline;
}

/* Таймлайн создаётся внутри, но доступен снаружи */
.section {
  scroll-timeline: --global-timeline;
}

/* Элемент вне секции использует таймлайн */
.anywhere {
  animation: progress linear;
  animation-timeline: --global-timeline;
}
```

---

## 17.5 Timeline Composition

### Несколько источников движения

Современный CSS позволяет использовать несколько независимых источников движения.

Один компонент может одновременно иметь:

- собственный таймлайн;
- несколько анимаций;
- разные режимы композиции.

```css
.element {
  /* Две анимации с разными таймлайнами */
  animation: 
    rotate linear,
    pulse linear;
  animation-timeline: 
    scroll(),  /* Синхронизация со скроллом */
    view();    /* Синхронизация с видимостью */
  animation-composition: 
    add,       /* Складываем значения */
    add;       /* Складываем значения */
}

@keyframes rotate {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

@keyframes pulse {
  from { transform: scale(1); }
  to { transform: scale(1.2); }
}
```

### Режимы композиции

```css
.element {
  animation-composition: add;       /* Складываем */
  animation-composition: accumulate; /* Накопление */
  animation-composition: replace;   /* Замена (по умолчанию) */
}
```

### Сложная композиция

```css
@keyframes progress {
  from { --progress: 0; }
  to { --progress: 1; }
}

@keyframes color-shift {
  from { --hue: 0; }
  to { --hue: 360; }
}

.element {
  animation: 
    progress linear,
    color-shift linear;
  animation-timeline: 
    scroll(),
    view();
  animation-composition: 
    add,
    add;
  
  background: oklch(
    from var(--color) 
    calc(50% + var(--progress) * 20%) 
    calc(0.1 + var(--progress) * 0.2) 
    calc(200 + var(--hue))
  );
}
```

---

## 17.6 Scroll-driven Motion Design

### Полноценный язык Motion Design

Scroll Timeline — это не просто способ сделать параллакс. Сегодня он используется как полноценный язык Motion Design.

**Типичные паттерны:**

```text
Reading Progress
  ↓
Индикатор чтения статьи
  ↓
Reveal Animation
  ↓
Появление элементов при скролле
  ↓
Sticky-анимации
  ↓
Фиксация и трансформация при скролле
  ↓
Parallax
  ↓
Разная скорость движения слоёв
  ↓
Инфографика
  ↓
Анимированные диаграммы и графики
  ↓
Storytelling
  ↓
Рассказ через скролл
```

### Пример: Storytelling-страница

```css
/* Глава 1: появление */
.chapter-1 {
  animation: reveal linear;
  animation-timeline: scroll();
  animation-range: entry 0% entry 50%;
}

/* Глава 2: масштабирование */
.chapter-2 {
  animation: scale-up linear;
  animation-timeline: scroll();
  animation-range: entry 20% entry 80%;
}

/* Глава 3: смена цвета */
.chapter-3 {
  animation: color-shift linear;
  animation-timeline: scroll();
  animation-range: entry 40% entry 100%;
}

@keyframes reveal {
  from { opacity: 0; transform: translateY(50px); }
  to { opacity: 1; transform: translateY(0); }
}

@keyframes scale-up {
  from { transform: scale(0.8); opacity: 0.5; }
  to { transform: scale(1); opacity: 1; }
}

@keyframes color-shift {
  from { background: var(--surface); }
  to { background: var(--accent-surface); }
}
```

**Все эти сценарии больше не требуют постоянного JavaScript.**

---

## 17.7 Производительность

### Преимущества декларативного подхода

Главное преимущество Scroll-driven Animations состоит в том, что вычисление прогресса происходит внутри браузерного движка.

```text
JavaScript-подход:
  ↓
Слушаем scroll
  ↓
Вызываем getBoundingClientRect()
  ↓
Синхронизируем requestAnimationFrame()
  ↓
Нагрузка на основной поток
  ↓
Scroll-driven Animations:
  ↓
Вычисление в браузерном движке
  ↓
Без вызова getBoundingClientRect()
  ↓
Без requestAnimationFrame()
  ↓
Минимальная нагрузка
```

### Рекомендации

Наиболее эффективными остаются:

```text
transform
  ↓
Работает на GPU
  ↓
Быстро
  ↓
opacity
  ↓
Работает на GPU
  ↓
Быстро
  ↓
Зарегистрированные @property
  ↓
Оптимизированы браузером
  ↓
Быстро
```

Следует избегать анимации свойств, вызывающих перерасчёт макета или повторную растеризацию больших областей:

```text
width / height
  ↓
Вызывают Layout
  ↓
Медленно
  ↓
top / left
  ↓
Вызывают Layout
  ↓
Медленно
  ↓
margin / padding
  ↓
Вызывают Layout
  ↓
Медленно
```

---

## 17.8 Доступность

### prefers-reduced-motion

Как и любые современные анимации, Scroll-driven Animations должны учитывать пользовательские предпочтения.

```css
/* Базовые анимации */
.element {
  animation: reveal linear;
  animation-timeline: view();
}

/* Учитываем настройки пользователя */
@media (prefers-reduced-motion: reduce) {
  .element {
    animation: none;
    opacity: 1; /* Всегда видим */
    transform: none;
  }
}
```

### Принципы

При включённом `prefers-reduced-motion` следует:

```text
Полностью отключать сложные движения
  ↓
Декоративные эффекты удаляются
  ↓
Сохраняются только функционально необходимые переходы
  ↓
Навигация остаётся понятной без динамики
```

**Motion Design должен улучшать восприятие интерфейса, а не создавать препятствия для пользователей.**

---

## 17.9 Интеграция с современным CSS

### Часть единой экосистемы

Scroll-driven Animations являются частью более широкой архитектуры платформы.

```text
Scroll-driven Animations
  ↓
Интегрируются с:
  ↓
CSS Nesting
  ↓
Container Queries
  ↓
View Transition API
  ↓
@property
  ↓
Custom Properties
  ↓
linear()
  ↓
interpolate-size
  ↓
content-visibility
  ↓
contain
```

### Полный пример

```css
@layer components {
  .card-container {
    container-type: inline-size;
    container-name: cards;
  }
  
  .card {
    /* Базовые стили */
    padding: var(--space-md);
    background: var(--surface);
    border-radius: var(--radius-lg);
    
    /* Nesting */
    & .title {
      font-size: var(--font-size-xl);
      font-weight: var(--font-weight-bold);
    }
    
    /* Scroll-driven Animation */
    animation: reveal linear;
    animation-timeline: view();
    animation-range: entry 0% entry 50%;
    
    /* Container Query */
    @container cards (max-width: 600px) {
      padding: var(--space-sm);
      
      & .title {
        font-size: var(--font-size-md);
      }
    }
    
    /* prefers-reduced-motion */
    @media (prefers-reduced-motion: reduce) {
      animation: none;
    }
  }
  
  @keyframes reveal {
    from {
      opacity: 0;
      transform: translateY(30px) scale(0.95);
    }
    to {
      opacity: 1;
      transform: translateY(0) scale(1);
    }
  }
}
```

**Именно совместное использование этих технологий формирует современную систему декларативной анимации CSS.**

---

## 17.10 Практические рекомендации

### Архитектурные правила

**1. Используйте Scroll-driven Animations вместо JavaScript-обработчиков**

```css
/* ✅ Рекомендуется */
.element {
  animation: reveal linear;
  animation-timeline: view();
}

/* ❌ Устаревает */
// window.addEventListener('scroll', () => { ... });
```

**2. Выбирайте правильный тип таймлайна**

```text
scroll()
  ↓
Привязан к прокрутке документа
  ↓
view()
  ↓
Привязан к видимости элемента
  ↓
Именованный
  ↓
Привязан к конкретному элементу
```

**3. Используйте animation-range для точного контроля**

```css
.element {
  animation-range: entry 0% entry 50%;
  /* Анимируется только при входе */
}
```

**4. Комбинируйте несколько анимаций**

```css
.element {
  animation: rotate, pulse;
  animation-timeline: scroll(), view();
}
```

**5. Всегда учитывайте prefers-reduced-motion**

```css
@media (prefers-reduced-motion: reduce) {
  * {
    animation: none !important;
  }
}
```

**6. Анимируйте производительные свойства**

```css
/* ✅ Рекомендуется */
transform: translateY(50px);
opacity: 0;

/* ❌ Не рекомендуется */
width: 100%;
height: 100%;
```

---

## 17.11 Итоги главы

1. **Новая модель анимации** — разделение на `@keyframes` (что) и `animation-timeline` (откуда)

2. **Scroll Progress Timeline** — привязка к скроллу через `scroll()`

3. **View Progress Timeline** — привязка к видимости через `view()`

4. **Диапазоны** — `entry`, `contain`, `cover`, `exit` через `animation-range`

5. **Именованные таймлайны** — `scroll-timeline`, `view-timeline`, `timeline-scope`

6. **Timeline Composition** — несколько анимаций с разными таймлайнами

7. **Scroll-driven Motion Design** — параллакс, reveal, sticky-анимации, storytelling

8. **Производительность** — вычисление в браузерном движке, анимация `transform` и `opacity`

9. **Доступность** — всегда учитывайте `prefers-reduced-motion`

10. **Интеграция** — работает с Nesting, Container Queries, View Transitions, `@property`

---

**Главная мысль:** Появление Scroll-driven Animations изменило саму модель CSS-анимации. Вместо единственного временного таймлайна браузер получил универсальную систему источников прогресса, где движение может определяться временем, прокруткой, положением элемента или пользовательскими таймлайнами. В сочетании с View Transition API, `@property` и современными функциями плавности это превращает CSS в полноценную платформу Motion Design, способную реализовывать сложные интерактивные сценарии без постоянного участия JavaScript.

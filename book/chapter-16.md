# Глава 16. View Transitions и архитектура современной навигации

> Плавные переходы между состояниями интерфейса долгое время оставались одной из наиболее сложных задач фронтенд-разработки. Для их реализации приходилось одновременно удерживать несколько состояний DOM, синхронизировать JavaScript-анимации и вручную управлять жизненным циклом компонентов.

View Transition API изменил саму модель работы браузера. Вместо того чтобы заставлять разработчика анимировать элементы самостоятельно, браузер берёт на себя построение промежуточного визуального состояния, автоматически связывает старый и новый интерфейс и выполняет переход средствами собственного движка рендеринга.

Главная идея технологии заключается в том, что анимируется не DOM, а визуальное представление документа.

---

## 16.1 Новая модель переходов

### От DOM-анимации к визуальной

До появления View Transitions смена состояния выглядела следующим образом:

```text
Старый DOM
  ↓
Изменение DOM (JavaScript)
  ↓
Новый DOM
```

Изменение происходило мгновенно. Любая анимация требовала JavaScript для координации состояний.

Современный браузер использует другую схему:

```text
Старое состояние
  ↓
Снимок (Snapshot)
  ↓
Изменение DOM
  ↓
Снимок (Snapshot)
  ↓
Интерполяция
  ↓
Новый экран
```

**Разработчик описывает только изменения состояния. Промежуточную фазу полностью строит браузер.**

### Три этапа перехода

**1. Снимок старого состояния**

Браузер фиксирует визуальное представление текущего документа.

**2. Обновление DOM**

Разработчик изменяет состояние документа (добавляет, удаляет, изменяет элементы).

**3. Снимок нового состояния**

Браузер фиксирует визуальное представление обновлённого документа.

**4. Интерполяция**

Браузер создаёт анимацию между двумя снимками.

### Почему это работает

```text
Традиционный подход:
  ↓
Разработчик управляет DOM и анимацией
  ↓
Нужно синхронизировать состояния
  ↓
Сложно, много кода
  ↓
View Transitions:
  ↓
Разработчик управляет только DOM
  ↓
Браузер управляет анимацией
  ↓
Просто, декларативно
```

---

## 16.2 Same-document и Cross-document

### Две ветви развития

Современный View Transition API развивается в двух направлениях:

```text
Same-document
  ↓
Внутри одной страницы
  ↓
SPA, смена вкладок
  ↓
document.startViewTransition()
  ↓
Cross-document
  ↓
Между страницами
  ↓
MPA, навигация
  ↓
@view-transition { navigation: auto; }
```

### Same-document Transitions

Используется внутри одной страницы для переходов между состояниями:

```javascript
// Запуск перехода
document.startViewTransition(() => {
  // Изменяем DOM
  updateUI();
});
```

**Применение:**

- Смена вкладок
- Фильтрация списка
- Открытие панели
- Изменение карточек
- SPA-навигация

**Пример:**

```javascript
// Фильтрация списка с переходом
function filterList(category) {
  document.startViewTransition(() => {
    items.forEach(item => {
      item.hidden = item.category !== category;
    });
  });
}
```

### Cross-document Transitions

Вторая ветвь технологии распространяет переходы на обычную навигацию между страницами:

```css
/* Активация в CSS */
@view-transition {
  navigation: auto;
}
```

При поддержке браузером переход автоматически применяется между страницами одного происхождения (*same origin*).

При отсутствии поддержки навигация продолжает работать без каких-либо изменений.

**Именно поэтому технология хорошо вписывается в концепцию Progressive Enhancement.**

```html
<!-- Страница A -->
<!DOCTYPE html>
<html>
<head>
  <style>
    @view-transition { navigation: auto; }
  </style>
</head>
<body>
  <h1 style="view-transition-name: title">Страница A</h1>
  <a href="/page-b">Перейти на страницу B</a>
</body>
</html>
```

```html
<!-- Страница B -->
<!DOCTYPE html>
<html>
<head>
  <style>
    @view-transition { navigation: auto; }
  </style>
</head>
<body>
  <h1 style="view-transition-name: title">Страница B</h1>
  <a href="/page-a">Вернуться на страницу A</a>
</body>
</html>
```

---

## 16.3 Общие элементы (Shared Elements)

### Связь между состояниями

Одной из сильнейших возможностей View Transitions является автоматическое сопоставление одинаковых элементов между двумя состояниями.

```css
/* Одинаковое имя связывает элементы */
.element {
  view-transition-name: hero-image;
}
```

### Пример: карточка → детальный просмотр

```html
<!-- Состояние 1: список карточек -->
<div class="gallery">
  <div class="card" style="view-transition-name: product-1">
    <img src="thumb-1.jpg">
    <h3>Товар 1</h3>
    <p>Цена: 1000 ₽</p>
  </div>
  <div class="card" style="view-transition-name: product-2">
    <img src="thumb-2.jpg">
    <h3>Товар 2</h3>
    <p>Цена: 2000 ₽</p>
  </div>
</div>
```

```html
<!-- Состояние 2: детальный просмотр -->
<div class="detail">
  <div style="view-transition-name: product-1">
    <img src="full-1.jpg">
    <h3>Товар 1</h3>
    <p>Цена: 1000 ₽</p>
    <p>Описание: ...</p>
  </div>
</div>
```

**Браузер автоматически:**
1. Находит элемент с `view-transition-name: product-1` в старом состоянии
2. Находит элемент с `view-transition-name: product-1` в новом состоянии
3. Интерполирует его положение, размер и содержимое
4. Создаёт плавный переход

### Динамические имена

Для списков можно генерировать имена динамически:

```javascript
// Генерация уникальных имён
function getTransitionName(id) {
  return `item-${id}`;
}

// При рендеринге
item.style.viewTransitionName = getTransitionName(item.id);
```

### Визуальная непрерывность

```text
Изображение, заголовок и цена
  ↓
Визуально продолжают существовать
  ↓
Хотя фактически браузер уже построил новый DOM
  ↓
Создаётся ощущение непрерывности интерфейса
```

---

## 16.4 Временное дерево перехода

### Структура псевдоэлементов

Во время анимации браузер создаёт собственное внутреннее дерево. Разработчик получает доступ к нему через специальные псевдоэлементы.

```text
::view-transition
  ↓
::view-transition-group(name)
  ↓
  ├── ::view-transition-image-pair(name)
  │     ├── ::view-transition-old(name)
  │     └── ::view-transition-new(name)
  └── ::view-transition-group(другой)
```

### Управление отдельными элементами

```css
/* Группа — контейнер для пары old/new */
::view-transition-group(hero) {
  animation-duration: 0.5s;
}

/* Старое состояние — уходящий элемент */
::view-transition-old(hero) {
  animation: slide-out 0.5s ease;
}

/* Новое состояние — появляющийся элемент */
::view-transition-new(hero) {
  animation: slide-in 0.5s ease;
}
```

### Кастомизация по умолчанию

```css
/* Полностью кастомный переход */
::view-transition-group(hero) {
  animation-duration: 0.4s;
  animation-timing-function: ease-in-out;
}

::view-transition-old(hero) {
  animation: zoom-out 0.4s ease;
}

::view-transition-new(hero) {
  animation: zoom-in 0.4s ease;
}

@keyframes zoom-out {
  from { transform: scale(1); opacity: 1; }
  to { transform: scale(0.8); opacity: 0; }
}

@keyframes zoom-in {
  from { transform: scale(1.2); opacity: 0; }
  to { transform: scale(1); opacity: 1; }
}
```

---

## 16.5 Настройка переходов

### Базовые возможности

View Transitions не ограничиваются стандартным cross-fade. Можно изменять:

- длительность
- easing
- масштаб
- прозрачность
- фильтры
- трансформации

```css
::view-transition-old(root),
::view-transition-new(root) {
  animation-duration: 0.5s;
  animation-timing-function: cubic-bezier(0.34, 1.56, 0.64, 1);
}
```

### Сложные переходы

```css
/* Слайд-эффект */
::view-transition-old(root) {
  animation: slide-out 0.4s ease;
}

::view-transition-new(root) {
  animation: slide-in 0.4s ease;
}

@keyframes slide-out {
  from { transform: translateX(0); opacity: 1; }
  to { transform: translateX(-100%); opacity: 0; }
}

@keyframes slide-in {
  from { transform: translateX(100%); opacity: 0; }
  to { transform: translateX(0); opacity: 1; }
}
```

### Использование `linear()`

```css
::view-transition-old(hero),
::view-transition-new(hero) {
  animation-duration: 0.35s;
  animation-timing-function: linear(
    0,        /* 0% */
    0.1 20%,  /* 20% */
    0.5 40%,  /* 40% */
    0.9 80%,  /* 80% */
    1         /* 100% */
  );
}
```

**Здесь используются обычные CSS Animations, рассмотренные в предыдущей главе. Таким образом View Transitions не заменяют существующую систему анимации, а используют её.**

---

## 16.6 Архитектурные ограничения

### Уникальные имена

Каждый `view-transition-name` должен быть уникален в пределах документа.

```html
<!-- ❌ Плохо — два элемента с одинаковым именем -->
<div view-transition-name="hero">...</div>
<div view-transition-name="hero">...</div>

<!-- ✅ Хорошо — уникальные имена -->
<div view-transition-name="hero-1">...</div>
<div view-transition-name="hero-2">...</div>
```

Если несколько элементов используют одно имя одновременно, браузер не сможет корректно сопоставить состояния.

```css
/* Решение для списков */
.item:nth-child(1) { view-transition-name: item-1; }
.item:nth-child(2) { view-transition-name: item-2; }
/* ... */
```

### Минимальные изменения DOM

Метод `startViewTransition()` не должен содержать длительных синхронных вычислений.

```javascript
// ❌ Плохо — тяжёлые вычисления внутри
document.startViewTransition(() => {
  // Тяжёлые вычисления на 300ms
  heavyComputation();
  updateDOM();
});

// ✅ Хорошо — вычисления до перехода
const data = prepareData(); // До перехода
document.startViewTransition(() => {
  updateDOM(data); // Быстрое обновление
});
```

**Чем быстрее браузер получит новое состояние документа, тем плавнее будет переход.**

### Progressive Enhancement

Cross-document переходы всё ещё имеют неодинаковую поддержку браузеров.

```javascript
// Проверка поддержки
if (document.startViewTransition) {
  document.startViewTransition(() => {
    updateUI();
  });
} else {
  // Fallback — без анимации
  updateUI();
}
```

```css
/* Проверка поддержки cross-document */
@supports (view-transition: 1) {
  @view-transition {
    navigation: auto;
  }
}
```

**Архитектура приложения должна работать независимо от наличия View Transition API. Анимация рассматривается как улучшение пользовательского опыта, а не как обязательная часть навигации.**

---

## 16.7 Motion Design уровня приложения

### От компонента к приложению

Наиболее важное изменение последних лет заключается не в самой технологии View Transitions. Изменилась философия проектирования интерфейсов.

Теперь переходы рассматриваются как часть общей системы Motion Design.

```text
Компонент
  ↓
  CSS Transitions, Animations
  ↓
Экран
  ↓
  View Transitions (same-document)
  ↓
Страница
  ↓
  View Transitions (cross-document)
  ↓
Навигация
  ↓
  Scroll-driven Animations
  ↓
Всё приложение
  ↓
  Единая система Motion Design
```

### Объединение технологий

View Transitions объединяются с:

- CSS Animations
- Scroll-driven Animations
- `@property`
- `linear()`
- `prefers-reduced-motion`

```css
/* Единая система Motion Design */
:root {
  --motion-duration-fast: 0.15s;
  --motion-duration-medium: 0.3s;
  --motion-duration-slow: 0.5s;
  --motion-easing-standard: ease;
  --motion-easing-spring: cubic-bezier(0.34, 1.56, 0.64, 1);
}

/* Компоненты */
.element {
  transition: transform var(--motion-duration-fast) var(--motion-easing-spring);
}

/* View Transitions */
::view-transition-old(root) {
  animation-duration: var(--motion-duration-slow);
  animation-timing-function: var(--motion-easing-standard);
}

/* Scroll-driven */
.card {
  animation: reveal var(--motion-duration-medium) var(--motion-easing-spring);
  animation-timeline: view();
}
```

### Результат

```text
Движение становится непрерывным
  ↓
На всех уровнях приложения
  ↓
Снижает когнитивную нагрузку
  ↓
Создаёт ощущение единого пространства
  ↓
Пользователь понимает структуру интерфейса
```

---

## 16.8 Практические рекомендации

### Архитектурные правила

При использовании View Transition API стоит придерживаться нескольких правил.

**1. Используйте технологию как прогрессивное улучшение**

```javascript
// ✅ Рекомендуется
if (document.startViewTransition) {
  document.startViewTransition(updateUI);
} else {
  updateUI(); // Без анимации
}
```

**2. Присваивайте `view-transition-name` только значимым элементам**

```css
/* ✅ Рекомендуется — только ключевые элементы */
.hero-image { view-transition-name: hero; }
.product-title { view-transition-name: title; }

/* ❌ Не рекомендуется — каждому элементу */
* { view-transition-name: all; }
```

**3. Не выполняйте тяжёлые вычисления внутри `startViewTransition()`**

```javascript
// ✅ Рекомендуется
const data = await fetchData();
document.startViewTransition(() => {
  render(data);
});

// ❌ Не рекомендуется
document.startViewTransition(async () => {
  const data = await fetchData(); // Задержка
  render(data);
});
```

**4. Настраивайте анимации через CSS, а не JavaScript**

```css
/* ✅ Рекомендуется */
::view-transition-old(root) {
  animation: fade-out 0.3s ease;
}

/* ❌ Не рекомендуется */
// JavaScript: старомодный подход
```

**5. Всегда учитывайте `prefers-reduced-motion`**

```css
/* ✅ Рекомендуется */
@media (prefers-reduced-motion: reduce) {
  ::view-transition-old(root),
  ::view-transition-new(root) {
    animation-duration: 0.001ms;
  }
}
```

**6. Используйте View Transitions для навигации, а не как замену всем существующим анимациям компонентов**

```text
Компоненты → CSS Transitions / Animations
  ↓
Экраны → View Transitions (same-document)
  ↓
Страницы → View Transitions (cross-document)
```

---

## 16.9 Итоги главы

1. **Новая модель переходов** — анимируется не DOM, а визуальное представление документа

2. **Same-document** — переходы внутри страницы через `document.startViewTransition()`

3. **Cross-document** — переходы между страницами через `@view-transition { navigation: auto; }`

4. **Shared Elements** — автоматическое сопоставление элементов через `view-transition-name`

5. **Временное дерево** — доступ к `::view-transition`, `::view-transition-group()`, `::view-transition-old()`, `::view-transition-new()`

6. **Настройка через CSS Animations** — полный контроль через стандартные CSS-анимации

7. **Архитектурные ограничения** — уникальные имена, минимальные изменения DOM, Progressive Enhancement

8. **Motion Design уровня приложения** — единая система от компонентов до навигации

9. **Практические правила:** progressive enhancement, осмысленные имена, быстрые переходы, предпочтение CSS, `prefers-reduced-motion`

---

**Главная мысль:** View Transition API знаменует переход веб-платформы от анимации отдельных элементов к анимации состояний интерфейса. Браузер самостоятельно управляет снимками документа, сопоставляет элементы между экранами и выполняет переходы средствами собственного движка рендеринга. В сочетании с современными возможностями CSS — Scroll-driven Animations, зарегистрированными пользовательскими свойствами и системой Motion Design — View Transitions становятся важнейшим элементом архитектуры современных веб-приложений, позволяя создавать непрерывный пользовательский опыт без сложной инфраструктуры JavaScript.

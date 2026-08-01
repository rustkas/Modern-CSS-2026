# Modern CSS 2026

## Архитектура, производительность и платформа современных интерфейсов

> Книга о современном CSS как инженерной дисциплине.  
> Не просто набор правил оформления, а полноценный язык проектирования интерфейсов.

---

## 📖 О книге

За последние пять лет CSS перестал быть просто языком оформления и превратился в полноценную платформу разработки интерфейсов. Сегодня CSS — это:

- мощная система компоновки (Grid, Flexbox, Container Queries);
- архитектурные инструменты (@layer, каскад, специфичность);
- анимации без JavaScript (View Transitions, Scroll-driven Animations);
- цвет как система (OKLCH, color-mix());
- полноценный язык компонентов (селекторы, nesting, изоляция).

Современные фреймворки:

- Angular
- React
- Vue
- Svelte
- Astro
- Qwik

строятся **поверх CSS**, используя его как основу для стилизации и компоновки интерфейсов.

Эта книга показывает CSS как полноценную инженерную дисциплину, а не просто набор правил оформления.

---

# 🎯 Цель книги

После изучения книги читатель должен понимать:

- как мыслить при разработке интерфейсов;
- как проектировать CSS для приложений со 100–200 компонентами;
- как избегать «войны специфичности» без !important;
- как строить дизайн-системы на основе @layer, Design Tokens и Container Queries;
- как использовать новые возможности CSS вместо JavaScript там, где это стало возможным;
- как организовать код так, чтобы он оставался понятным через несколько лет;
- как измерять и оптимизировать производительность CSS.

---

# 👥 Для кого эта книга

Книга предназначена для:

- Frontend-разработчиков;
- Angular / React / Vue / Svelte разработчиков;
- архитекторов интерфейсов;
- разработчиков дизайн-систем;
- специалистов по Web Performance;
- всех, кто хочет понимать современную веб-платформу.

---

# 🧠 Главная идея

Современный frontend строится на трёх фундаментальных технологиях:

```
HTML
 |
 |-- структура и смысл

CSS
 |
 |-- внешний вид, layout и анимация

JavaScript / TypeScript
 |
 |-- логика и поведение
```

CSS больше не просто оформление:

```
CSS 2026
 |
 |-- Layout (Grid, Flexbox, Container Queries)
 |
 |-- Архитектура (@layer, каскад, специфичность)
 |
 |-- Анимация (View Transitions, Scroll-driven Animations)
 |
 |-- Цвет как система (OKLCH, color-mix())
 |
 |-- Компонентный подход (селекторы, nesting)
 |
 |-- Производительность (rendering pipeline, GPU)
```

Фреймворки являются дополнительным уровнем:

```
Angular
React
Vue
Svelte

        ↓

HTML + CSS + JavaScript

        ↓

Browser Web Platform
```

---

# 📚 Содержание

# Предисловие

## CSS больше не язык оформления

- Почему CSS изменился
- От CSS2 к платформе CSS
- Почему Sass уже не обязателен
- Эволюция браузеров
- Baseline и совместимость

* [📖 Читать главу](./book/preface.md)
* [📚 Литература](./references/preface.md)
* [💻 Примеры](./examples/preface.md)
* [🧪 Практика](./exercises/preface.md)

---

# Часть I. Новый CSS

## Глава 1. Рождение CSS-платформы

Темы:

- Что умел CSS2 и чего ему не хватало
- Эволюция Layout: от Float к Grid
- Как CSS стал платформой: хронология прорывов
- Эволюция браузеров
- Революция в селекторах
- CSS vs JavaScript: границы смещаются
- Препроцессоры теряют актуальность
- Совместимость и Baseline

* [📖 Читать главу](./book/chapter-01.md)
* [📚 Литература](./references/chapter-01.md)
* [💻 Примеры](./examples/chapter-01.md)
* [🧪 Практика](./exercises/chapter-01.md)

---

## Глава 2. Современный каскад

Темы:

- Повторение каскада
- Специфичность
- Порядок применения правил
- Custom Properties
- @scope
- Современная модель наследования

* [📖 Читать главу](./book/chapter-02.md)
* [📚 Литература](./references/chapter-02.md)
* [💻 Примеры](./examples/chapter-02.md)
* [🧪 Практика](./exercises/chapter-02.md)

---

## Глава 3. Cascade Layers (`@layer`): архитектура современного каскада

Темы:

- Почему появился @layer
- Иерархия слоёв
- Архитектура проекта
- Замена !important
- Стратегии организации больших проектов

* [📖 Читать главу](./book/chapter-03.md)
* [📚 Литература](./references/chapter-03.md)
* [💻 Примеры](./examples/chapter-03.md)
* [🧪 Практика](./exercises/chapter-03.md)

---

## Глава 4. CSS как язык компонентов

Темы:

- Компонентное мышление
- Изоляция
- Design Tokens
- Переиспользование
- Контракты компонентов

* [📖 Читать главу](./book/chapter-04.md)
* [📚 Литература](./references/chapter-04.md)
* [💻 Примеры](./examples/chapter-04.md)
* [🧪 Практика](./exercises/chapter-04.md)

---

# Часть II. Новый Layout

## Глава 5. Современный CSS Grid: декларативная архитектура макетов

Темы:

- Современный Grid
- Subgrid
- Masonry (если станет стандартом)
- Практические паттерны

* [📖 Читать главу](./book/chapter-05.md)
* [📚 Литература](./references/chapter-05.md)
* [💻 Примеры](./examples/chapter-05.md)
* [🧪 Практика](./exercises/chapter-05.md)

---

## Глава 6. Flexbox: алгоритм одномерной композиции

Темы:

- Когда использовать Flexbox
- Когда использовать Grid
- Типичные ошибки
- Производительность

* [📖 Читать главу](./book/chapter-06.md)
* [📚 Литература](./references/chapter-06.md)
* [💻 Примеры](./examples/chapter-06.md)
* [🧪 Практика](./exercises/chapter-06.md)

---

## Глава 7. Container Queries

Темы:

- Почему Media Queries перестали быть достаточными
- Контейнеры
- Size Queries
- Style Queries
- Container Units
- Практические примеры

* [📖 Читать главу](./book/chapter-07.md)
* [📚 Литература](./references/chapter-07.md)
* [💻 Примеры](./examples/chapter-07.md)
* [🧪 Практика](./exercises/chapter-07.md)

---

## Глава 8. Anchor Positioning

Темы:

- Новая модель позиционирования
- Tooltip
- Dropdown
- Popover
- Контекстные меню
- Overlay без JavaScript

* [📖 Читать главу](./book/chapter-08.md)
* [📚 Литература](./references/chapter-08.md)
* [💻 Примеры](./examples/chapter-08.md)
* [🧪 Практика](./exercises/chapter-08.md)

---

# Часть III. Современные селекторы

## Глава 9. :has() — долгожданный родительский селектор

Темы:

- Принцип работы
- Реальные сценарии
- Формы
- Навигация
- Карточки
- Производительность

* [📖 Читать главу](./book/chapter-09.md)
* [📚 Литература](./references/chapter-09.md)
* [💻 Примеры](./examples/chapter-09.md)
* [🧪 Практика](./exercises/chapter-09.md)

---

## Глава 10. Новое поколение селекторов

Темы:

- :is()
- :where()
- :not()
- :nth-child()
- Комбинирование селекторов

* [📖 Читать главу](./book/chapter-10.md)
* [📚 Литература](./references/chapter-10.md)
* [💻 Примеры](./examples/chapter-10.md)
* [🧪 Практика](./exercises/chapter-10.md)

---

## Глава 11. CSS Nesting

Темы:

- Почему Nesting появился только сейчас
- Отличия от Sass
- Лучшие практики
- Ограничения
- Организация файлов

* [📖 Читать главу](./book/chapter-11.md)
* [📚 Литература](./references/chapter-11.md)
* [💻 Примеры](./examples/chapter-11.md)
* [🧪 Практика](./exercises/chapter-11.md)

---

# Часть IV. Цвет как система

## Глава 12. Современные цветовые пространства

Темы:

- RGB
- HSL
- Lab
- LCH
- OKLab
- OKLCH

* [📖 Читать главу](./book/chapter-12.md)
* [📚 Литература](./references/chapter-12.md)
* [💻 Примеры](./examples/chapter-12.md)
* [🧪 Практика](./exercises/chapter-12.md)

---

## Глава 13. Работа с цветом

Темы:

- color-mix()
- Relative Colors
- Цветовые функции
- Контрастность
- Темизация

* [📖 Читать главу](./book/chapter-13.md)
* [📚 Литература](./references/chapter-13.md)
* [💻 Примеры](./examples/chapter-13.md)
* [🧪 Практика](./exercises/chapter-13.md)

---

## Глава 14. Design Tokens

Темы:

- Цветовые токены
- Типографика
- Размеры
- Радиусы
- Пространства
- Интеграция с Figma

* [📖 Читать главу](./book/chapter-14.md)
* [📚 Литература](./references/chapter-14.md)
* [💻 Примеры](./examples/chapter-14.md)
* [🧪 Практика](./exercises/chapter-14.md)

---

# Часть V. Анимация без JavaScript

## Глава 15. Современные анимации

Темы:

- Animation
- Transition
- Motion Design
- Производительность

* [📖 Читать главу](./book/chapter-15.md)
* [📚 Литература](./references/chapter-15.md)
* [💻 Примеры](./examples/chapter-15.md)
* [🧪 Практика](./exercises/chapter-15.md)

---

## Глава 16. View Transition API

Темы:

- Переходы между страницами
- SPA
- MPA
- Современная навигация
- Shared Elements

* [📖 Читать главу](./book/chapter-16.md)
* [📚 Литература](./references/chapter-16.md)
* [💻 Примеры](./examples/chapter-16.md)
* [🧪 Практика](./exercises/chapter-16.md)

---

## Глава 17. Scroll-driven Animations

Темы:

- Scroll Timeline
- View Timeline
- Parallax
- Reveal Effects
- Sticky-анимации
- Без IntersectionObserver

* [📖 Читать главу](./book/chapter-17.md)
* [📚 Литература](./references/chapter-17.md)
* [💻 Примеры](./examples/chapter-17.md)
* [🧪 Практика](./exercises/chapter-17.md)

---

# Часть VI. Архитектура CSS

## Глава 18. Cascade Layers как архитектурный инструмент

Темы:

- Слои приложения
- Reset
- Tokens
- Base
- Components
- Utilities
- Overrides

* [📖 Читать главу](./book/chapter-18.md)
* [📚 Литература](./references/chapter-18.md)
* [💻 Примеры](./examples/chapter-18.md)
* [🧪 Практика](./exercises/chapter-18.md)

---

## Глава 19. CSS-модули

Темы:

- CSS Modules
- Shadow DOM
- Angular
- React
- Vue
- Svelte

* [📖 Читать главу](./book/chapter-19.md)
* [📚 Литература](./references/chapter-19.md)
* [💻 Примеры](./examples/chapter-19.md)
* [🧪 Практика](./exercises/chapter-19.md)

---

## Глава 20. Design System

Темы:

- Построение библиотеки компонентов
- API компонентов
- Переиспользование
- Масштабирование

* [📖 Читать главу](./book/chapter-20.md)
* [📚 Литература](./references/chapter-20.md)
* [💻 Примеры](./examples/chapter-20.md)
* [🧪 Практика](./exercises/chapter-20.md)

---

# Часть VII. Производительность

## Глава 21. Rendering Pipeline

Темы:

- Style
- Layout
- Paint
- Composite

* [📖 Читать главу](./book/chapter-21.md)
* [📚 Литература](./references/chapter-21.md)
* [💻 Примеры](./examples/chapter-21.md)
* [🧪 Практика](./exercises/chapter-21.md)

---

## Глава 22. Производительность CSS

Темы:

- contain
- content-visibility
- will-change
- Layout Shift
- GPU

* [📖 Читать главу](./book/chapter-22.md)
* [📚 Литература](./references/chapter-22.md)
* [💻 Примеры](./examples/chapter-22.md)
* [🧪 Практика](./exercises/chapter-22.md)

---

## Глава 23. Современные инструменты

Темы:

- Lightning CSS
- PostCSS
- Stylelint
- DevTools
- Анализ производительности

* [📖 Читать главу](./book/chapter-23.md)
* [📚 Литература](./references/chapter-23.md)
* [💻 Примеры](./examples/chapter-23.md)
* [🧪 Практика](./exercises/chapter-23.md)

---

# Часть VIII. Практика

## Глава 24. Создание собственной UI-библиотеки

Темы:

- Кнопки
- Формы
- Диалоги
- Карточки
- Навигация

* [📖 Читать главу](./book/chapter-24.md)
* [📚 Литература](./references/chapter-24.md)
* [💻 Примеры](./examples/chapter-24.md)
* [🧪 Практика](./exercises/chapter-24.md)

---

## Глава 25. Построение дизайн-системы

Темы:

- Tokens
- Темы
- Цветовые схемы
- Компоненты
- Документация

* [📖 Читать главу](./book/chapter-25.md)
* [📚 Литература](./references/chapter-25.md)
* [💻 Примеры](./examples/chapter-25.md)
* [🧪 Практика](./exercises/chapter-25.md)

---

## Глава 26. Полностью современный проект

Создание полноценного приложения без Sass и Bootstrap с использованием:

- CSS Variables
- @layer
- Container Queries
- :has()
- CSS Nesting
- Grid
- Flexbox
- View Transition API
- Scroll Timeline
- Anchor Positioning
- color-mix()
- OKLCH
- Design Tokens
- Dark Theme

* [📖 Читать главу](./book/chapter-26.md)
* [📚 Литература](./references/chapter-26.md)
* [💻 Примеры](./examples/chapter-26.md)
* [🧪 Практика](./exercises/chapter-26.md)

---

# Приложения

## A. Browser Baseline 2026

* [📖 Читать приложение](./appendices/appendix-a.md)

## B. Совместимость браузеров

* [📖 Читать приложение](./appendices/appendix-b.md)

## C. Каталог новых CSS API

* [📖 Читать приложение](./appendices/appendix-c.md)

## D. Каталог функций CSS

* [📖 Читать приложение](./appendices/appendix-d.md)

## E. Шпаргалка по современной архитектуре CSS

* [📖 Читать приложение](./appendices/appendix-e.md)

---

# 📚 Источники

Основные источники:

- [CSS Working Group](https://www.w3.org/Style/CSS/)
- [MDN Web Docs](https://developer.mozilla.org/)
- [web.dev](https://web.dev/)
- [Chrome Developers](https://developer.chrome.com/)
- [Mozilla Hacks](https://hacks.mozilla.org/)
- [CSS Tricks](https://css-tricks.com/)

---

# 🧪 Практический подход

Каждая глава содержит:

- объяснение концепций;
- ссылки на спецификации;
- практические примеры;
- рекомендации;
- типичные ошибки;
- задания;
- ссылки для дальнейшего изучения.

---

# 📌 Статус проекта

🚧 В разработке

Версия: 0.1

Цель:
создать современный учебник по CSS, ориентированный на разработчиков 2026 года.

---

# Лицензия

MIT

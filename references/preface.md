# CSS больше не язык оформления: рождение декларативной платформы

Современный CSS прошел путь от простого набора правил для стилизации текста до полноценной **декларативной программной платформы**. Сегодня это мощная среда, которая берет на себя логику интерфейса, анимации и сложные расчеты, ранее требовавшие JavaScript или тяжелых препроцессоров.

### Почему CSS изменился

Традиционных инструментов стилизации стало недостаточно для создания современных отзывчивых интерфейсов. Основные причины трансформации:

- **Потребность в системности:** Разработчики перешли от хаотичных стилей к масштабируемым дизайн-системам (таким как [CUBE CSS](https://piccalil.li/blog/cube-css/)).
- **Производительность:** Нативные решения для анимаций (например, [Scroll-driven animations](https://www.joshwcomeau.com/css/scroll-driven-animations/)) работают в отдельных потоках браузера, не нагружая основной поток JavaScript.
- **Интерактивность:** Грань между состояниями CSS (pseudo-classes) и событиями JavaScript стирается. Современный CSS умеет «слушать» изменения окружения и реагировать на них декларативно.

### От CSS2 к платформе CSS

Эпоха монолитных версий (CSS1, CSS2) осталась в прошлом.

1.  **Модульная архитектура:** После CSS 2.1 спецификация развивается отдельными модулями разной степени зрелости. Официально «CSS4» не существует — вместо версий используются ежегодные «снимки» ([CSS Snapshots](https://www.w3.org/TR/css/)), определяющие стабильное состояние языка на текущий момент.
2.  **Houdini — «JS для CSS»:** Это набор API, открывающий доступ к внутренним механизмам рендеринга браузера. Разработчики могут создавать свои свойства с проверкой типов через `@property` и писать собственные алгоритмы отрисовки (Paint API) или раскладки (Layout API).

### Почему Sass уже не обязателен

Препроцессоры вроде Sass годами компенсировали недостатки CSS, но сегодня язык ассимилировал их лучшие функции, сделав их динамическими:

- **Нативная вложенность (Nesting):** Поддержка вложенности стилей ([CSS Nesting](https://www.w3.org/TR/css-nesting-1/)) теперь встроена в браузеры, что делает код чище без этапа компиляции.
- **«Живые» переменные:** В отличие от переменных Sass, которые исчезают после сборки, [Custom Properties](https://web.dev/learn/css/custom-properties) доступны в браузере в реальном времени, наследуются и могут изменяться через JS.
- **Функции и миксины:** В разработке находятся спецификации для нативных функций и миксинов прямо внутри `.css` файлов.

### Эволюция браузеров

Браузеры перестали быть препятствием для инноваций благодаря новым подходам:

- **«Evergreen» модель:** Основные браузеры обновляются автоматически и часто, мгновенно внедряя новые функции.
- **Проекты Interop:** Инициативы [Interop 2025](https://hacks.mozilla.org/2025/02/interop-2025/) и [Interop 2026](https://css-tricks.com/interop-2026/) объединяют Google, Apple, Mozilla и Microsoft для обеспечения единой работы CSS-фич во всех движках.
- **Инструменты разработчика:** DevTools теперь нативно поддерживают отладку [Grid](https://developer.chrome.com/docs/css-ui/css-grid-tooling), [Cascade Layers](https://css-tricks.com/css-cascade-layers/) и [Container Queries](https://developer.mozilla.org/docs/Web/CSS/Reference/At-rules/@container).

### Baseline и совместимость

Для упрощения жизни разработчиков была введена концепция [**Baseline**](https://web.dev/baseline). Она дает четкий сигнал о готовности технологии:

- **Newly available (Новинка):** Функция поддерживается всеми основными браузерами и готова к внедрению.
- **Widely available (Широкая доступность):** С момента достижения полной совместимости прошло 30 месяцев. Такие возможности безопасны для использования без глубоких проверок поддержки.

Например, такие мощные инструменты, как субсетки ([Subgrid](https://web.dev/articles/css-subgrid)) и типизированные свойства ([@property](https://web.dev/blog/at-property-baseline)), уже стали частью Baseline, изменив подход к разработке.

Далее, представляю подборку авторитетных источников по каждому разделу — это не «пруф» дословного цитирования, а актуальные первоисточники и авторитетные разборы, на которые можно опираться:

## Предисловие / CSS больше не язык оформления

- MDN — CSS Houdini (общий обзор трансформации CSS в платформу): https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Properties_and_values_API/Houdini
- web.dev — Baseline (общий контекст «зрелости» современного CSS): https://web.dev/baseline

## Почему CSS изменился

- MDN — Houdini APIs (обзор всех API, разгружающих JS): https://developer.mozilla.org/en-US/docs/Web/API/Houdini_APIs
- MDN — CSS Painting API (перенос вычислений на движок/воркеры): https://developer.mozilla.org/en-US/docs/Web/API/CSS_Painting_API
- W3C — CSS Paint API Explainer (обоснование производительности): https://github.com/w3c/css-houdini-drafts/blob/main/css-paint-api/EXPLAINER.md

## От CSS2 к платформе CSS

- W3C — CSS Snapshot 2026 (официальный документ, «снимок» вместо версий): https://www.w3.org/TR/css-2026/
- W3C — новость о публикации CSS Snapshot 2026: https://www.w3.org/news/2026/css-snapshot-2026-published-as-a-group-note
- W3C — история публикаций CSS Snapshot: https://www.w3.org/standards/history/css-2026/
- W3C — список всех спецификаций CSS (модульная архитектура): https://www.w3.org/Style/CSS/specs.en.html
- MDN — CSS Typed Object Model API: https://developer.mozilla.org/en-US/docs/Web/API/Houdini_APIs (раздел Typed OM)

## Почему Sass уже не обязателен

- MDN — CSS Custom Properties (`--var`), «живые» переменные: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_cascading_variables
- MDN — Houdini `@property` (типизация кастомных свойств): https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Properties_and_values_API/Houdini
- Статья о Typed OM и работе со значениями CSS как с объектами (в противовес строкам Sass): https://www.mo4tech.com/css-houdini-properties-values-and-the-paint-api.html

## Эволюция браузеров

- web.dev — Interop 2026 (совместная работа Google, Apple, Mozilla, Microsoft, Igalia): https://web.dev/blog/interop-2026
- WebKit (Apple) — Announcing Interop 2026: https://webkit.org/blog/17818/announcing-interop-2026/
- Mozilla Hacks — Launching Interop 2026: https://hacks.mozilla.org/2026/02/launching-interop-2026/
- Microsoft Edge Blog — Interop 2026: https://blogs.windows.com/msedgedev/2026/02/12/microsoft-edge-and-interop-2026/
- GitHub web-platform-tests — Interop 2026 README (техническая база проекта): https://github.com/web-platform-tests/interop/blob/main/2026/README.md

## Baseline и совместимость

- web.dev — Baseline (определение Newly/Widely available): https://web.dev/baseline
- web.dev — How to use Baseline: https://web.dev/how-to-use-baseline
- web.dev — How to choose your Baseline target (детали про 30 месяцев и container queries как пример): https://web.dev/articles/how-to-choose-your-baseline-target
- web.dev — Baseline monthly digest, март 2026 (актуальный пример перехода фич в Widely available): https://web.dev/blog/baseline-digest-mar-2026

Хочешь, я вставлю эти ссылки прямо в текст статьи как сноски/список литературы в конце файла?

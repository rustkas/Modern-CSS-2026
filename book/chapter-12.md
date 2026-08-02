# Глава 12. Современная цветовая система CSS

За последние несколько лет CSS пережил настоящую революцию в области работы с цветом. Если раньше цвет представлял собой фиксированное значение (`#1976d2`, `rgb()`, `hsl()`), то современная платформа рассматривает его как вычисляемую сущность.

Сегодня CSS умеет:

- работать в перцептивных цветовых пространствах;
- автоматически адаптироваться к светлой и тёмной теме;
- программно изменять цвет без препроцессоров;
- смешивать цвета;
- использовать широкий цветовой охват современных дисплеев;
- строить полноценные дизайн-системы исключительно средствами браузера.

Именно поэтому современный CSS всё чаще рассматривает цвет не как оформление, а как отдельную вычислительную подсистему интерфейса.

---

## 12.1 От фиксированных цветов к вычисляемым

### Исторический подход

На протяжении многих лет большинство проектов использовали одинаковую модель работы:

```css
:root {
  --primary: #2563eb;
  --primary-hover: #1d4ed8;
  --primary-active: #1e40af;
  --primary-light: #3b82f6;
  --primary-dark: #1e3a8a;
}
```

Каждое состояние приходилось создавать вручную. Это приводило к:

- разрастанию цветовых переменных;
- ошибкам при ручном вычислении оттенков;
- сложности поддержки нескольких тем;
- необходимости использовать препроцессоры.

### Современный подход

Современный CSS предлагает другой подход. В дизайн-системе хранится один исходный цвет, а остальные вычисляются автоматически.

```css
:root {
  /* Единственный исходный цвет */
  --primary: oklch(65% 0.22 255);
}

/* Все состояния вычисляются */
.button {
  background: var(--primary);

  &:hover {
    background: color-mix(in oklch, var(--primary), white 15%);
  }

  &:active {
    background: color-mix(in oklch, var(--primary), black 15%);
  }

  &:disabled {
    background: color-mix(in oklch, var(--primary), gray 50%);
    opacity: 0.5;
  }
}
```

**Цвет становится функцией, а не константой.**

```text
Раньше:
  ↓
Хранили каждый оттенок
  ↓
Ручное управление палитрой
  ↓
Сложность поддержки
  ↓
Теперь:
  ↓
Храним базовый цвет
  ↓
Вычисляем остальные
  ↓
Единая математическая модель
```

---

## 12.2 Перцептивные цветовые пространства

### Классические пространства

**sRGB** — исторический стандарт веба:

```css
/* RGB */
color: rgb(37, 99, 235);
/* HEX */
color: #2563eb;
/* HSL */
color: hsl(224, 76%, 53%);
```

**Проблема:** отсутствие перцептивной равномерности. Изменение Lightness в HSL не означает одинакового изменения воспринимаемой человеком яркости.

```text
HSL Lightness 50% → 60%
  ↓
Визуальное изменение зависит от оттенка
  ↓
Неравномерное восприятие
```

### Современные перцептивные пространства

**Lab и LCH** — перцептивно-равномерные пространства:

```css
/* Lab */
color: lab(50% 20 -10);
/* LCH */
color: lch(50% 25 330);
```

**OKLab и OKLCH** — улучшенная версия, оптимизированная для веба:

```css
/* OKLab */
color: oklab(65% -0.08 0.15);
/* OKLCH */
color: oklch(65% 0.22 255);
```

### Почему OKLCH важен

**Преимущества OKLCH:**

1. **Одинаковое визуальное изменение светлоты** — изменение `l` на 10% выглядит как изменение яркости на 10%
2. **Естественное изменение насыщенности** — изменение `c` даёт предсказуемый результат
3. **Предсказуемая генерация палитр** — можно математически создавать гармоничные палитры
4. **Корректное смешивание цветов** — `color-mix()` даёт визуально правильный результат

```text
HSL:
  ↓
Светлота ≠ воспринимаемая яркость
  ↓
OKLCH:
  ↓
Светлота = воспринимаемая яркость
  ↓
Предсказуемые результаты
```

### Сравнение

```css
/* ❌ Старый подход — не перцептивный */
:root {
  --primary: #2563eb;
  --primary-light: #3b82f6; /* Угадываем */
  --primary-dark: #1d4ed8; /* Угадываем */
}

/* ✅ Современный подход — перцептивный */
:root {
  --primary: oklch(65% 0.22 255);
  --primary-light: oklch(75% 0.22 255); /* Предсказуемо */
  --primary-dark: oklch(55% 0.22 255); /* Предсказуемо */
}
```

---

## 12.3 Wide Gamut и Display-P3

### Проблема sRGB

Исторически веб строился вокруг пространства sRGB. Однако современные устройства способны отображать значительно больше цветов.

```text
sRGB
  ↓
Ограниченный цветовой охват
  ↓
~35% видимых цветов
  ↓
Display-P3
  ↓
Широкий цветовой охват
  ↓
~50% видимых цветов
```

### Использование Display-P3

CSS позволяет использовать широкий цветовой охват через пространство `display-p3`:

```css
/* Базовый цвет в sRGB (будет работать везде) */
background: oklch(65% 0.22 255);

/* Расширенный цвет в Display-P3 (для современных устройств) */
@supports (color: color(display-p3 0 0 0)) {
  background: color(display-p3 0.18 0.78 0.36);
}
```

### Когда использовать Wide Gamut

Использование широкого цветового охвата особенно актуально для:

```text
Медиа-приложений
  ↓
Фотографии, видео, редактирование
  ↓
Интернет-магазинов
  ↓
Точная передача цветов товаров
  ↓
Брендированных интерфейсов
  ↓
Точное соответствие брендбуку
  ↓
Профессиональных графических систем
  ↓
Дизайн-инструменты, редакторы
```

### Практический подход

```css
:root {
  /* Базовый цвет — работает везде */
  --brand: oklch(65% 0.22 255);

  /* Расширенный цвет — для современных устройств */
  @supports (color: color(display-p3 0 0 0)) {
    --brand: color(display-p3 0.18 0.78 0.36);
  }
}

.button {
  background: var(--brand);
}
```

---

## 12.4 Программирование цвета

### `color-mix()` — смешивание цветов

Позволяет смешивать цвета в любом цветовом пространстве:

```css
/* Базовое смешивание */
background: color-mix(in oklch, var(--primary), white 15%);

/* Смешивание в разных пространствах */
background: color-mix(in oklch, red 50%, blue 50%);
background: color-mix(in hsl, red 50%, blue 50%);
background: color-mix(in srgb, red 50%, blue 50%);
```

**Применение:**

```css
.button {
  --color: oklch(65% 0.22 255);

  background: var(--color);

  &:hover {
    background: color-mix(in oklch, var(--color), white 15%);
  }

  &:active {
    background: color-mix(in oklch, var(--color), black 15%);
  }

  &:disabled {
    background: color-mix(in oklch, var(--color), gray 40%);
  }
}

/* Overlay */
.overlay {
  background: color-mix(in oklch, var(--color), transparent 60%);
}

/* Shadow */
.shadow {
  box-shadow: 0 4px 12px color-mix(in oklch, var(--color), black 30%);
}
```

### Relative Color Syntax — изменение каналов

Позволяет изменить отдельный канал существующего цвета:

```css
/* Изменение светлоты */
background: oklch(from var(--primary) calc(l - 0.08) c h);

/* Изменение насыщенности */
background: oklch(from var(--primary) l calc(c - 0.05) h);

/* Изменение оттенка */
background: oklch(from var(--primary) l c calc(h + 30));
```

**Важное отличие от `color-mix()`:**

```text
color-mix()
  ↓
Смешивает два цвета
  ↓
Результат — смесь
  ↓
Relative Color Syntax
  ↓
Изменяет один параметр
  ↓
Результат — тот же цвет, но с изменённым каналом
```

### Когда использовать что

| Задача                | Инструмент      | Пример                                               |
| --------------------- | --------------- | ---------------------------------------------------- |
| Смешать два цвета     | `color-mix()`   | `color-mix(in oklch, red 50%, blue 50%)`             |
| Изменить светлоту     | Relative Colors | `oklch(from var(--color) calc(l - 0.08) c h)`        |
| Изменить насыщенность | Relative Colors | `oklch(from var(--color) l calc(c - 0.05) h)`        |
| Изменить тон          | Relative Colors | `oklch(from var(--color) l c calc(h + 30))`          |
| Создать прозрачность  | `color-mix()`   | `color-mix(in oklch, var(--color), transparent 40%)` |

### Практический пример

```css
:root {
  /* Базовый цвет */
  --accent: oklch(65% 0.22 255);

  /* Варианты через Relative Colors */
  --accent-hover: oklch(from var(--accent) calc(l + 0.08) c h);
  --accent-active: oklch(from var(--accent) calc(l - 0.08) c h);
  --accent-subtle: oklch(from var(--accent) calc(l + 0.2) calc(c / 2) h);
  --accent-muted: oklch(from var(--accent) l calc(c / 3) h);
}

.button {
  background: var(--accent);

  &:hover {
    background: var(--accent-hover);
  }

  &:active {
    background: var(--accent-active);
  }
}

.subtle-bg {
  background: var(--accent-subtle);
}
```

---

## 12.5 Цвет как часть дизайн-системы

### Дизайн-токены

Современные проекты редко работают с отдельными цветовыми значениями. Используются дизайн-токены:

```css
:root {
  /* Поверхности */
  --surface: oklch(98% 0.01 100);
  --surface-elevated: oklch(96% 0.01 100);
  --surface-inverse: oklch(20% 0.01 100);

  /* Текст */
  --text-primary: oklch(20% 0.01 100);
  --text-secondary: oklch(40% 0.01 100);
  --text-inverse: oklch(98% 0.01 100);

  /* Акценты */
  --accent: oklch(65% 0.22 255);
  --danger: oklch(55% 0.25 30);
  --success: oklch(60% 0.2 150);
  --warning: oklch(70% 0.2 80);
}
```

### Вычисляемые токены

Конкретное значение каждого токена может вычисляться:

```css
:root {
  --accent: oklch(65% 0.22 255);

  /* Вычисляемые состояния */
  --accent-hover: color-mix(in oklch, var(--accent), white 15%);
  --accent-active: color-mix(in oklch, var(--accent), black 15%);
  --accent-subtle: color-mix(in oklch, var(--accent), transparent 80%);

  /* Адаптивные токены */
  --text-on-accent: light-dark(white, black);
}
```

### Полная система токенов

```css
@layer tokens {
  :root {
    /* === Цветовая система === */
    /* Базовые цвета */
    --color-primary: oklch(65% 0.22 255);
    --color-secondary: oklch(55% 0.2 120);
    --color-danger: oklch(55% 0.25 30);
    --color-success: oklch(60% 0.2 150);
    --color-warning: oklch(70% 0.2 80);
    --color-info: oklch(60% 0.18 210);

    /* Вычисляемые состояния */
    --color-primary-hover: color-mix(in oklch, var(--color-primary), white 15%);
    --color-primary-active: color-mix(
      in oklch,
      var(--color-primary),
      black 15%
    );
    --color-primary-subtle: color-mix(
      in oklch,
      var(--color-primary),
      transparent 85%
    );

    /* Поверхности */
    --color-surface: light-dark(oklch(98% 0.01 100), oklch(18% 0.01 100));
    --color-surface-raised: light-dark(
      oklch(96% 0.01 100),
      oklch(22% 0.01 100)
    );
    --color-surface-overlay: light-dark(
      oklch(94% 0.01 100),
      oklch(26% 0.01 100)
    );

    /* Текст */
    --color-text: light-dark(oklch(20% 0.01 100), oklch(90% 0.01 100));
    --color-text-secondary: light-dark(
      oklch(40% 0.01 100),
      oklch(70% 0.01 100)
    );
    --color-text-inverse: light-dark(oklch(98% 0.01 100), oklch(15% 0.01 100));

    /* Границы */
    --color-border: light-dark(oklch(90% 0.01 100), oklch(30% 0.01 100));
    --color-border-strong: light-dark(oklch(80% 0.01 100), oklch(40% 0.01 100));

    /* === Тени === */
    --shadow-sm: 0 1px 2px
      color-mix(in oklch, var(--color-text), transparent 95%);
    --shadow-md: 0 4px 12px
      color-mix(in oklch, var(--color-text), transparent 92%);
    --shadow-lg: 0 8px 24px
      color-mix(in oklch, var(--color-text), transparent 88%);

    /* === Скругления === */
    --radius-sm: 4px;
    --radius-md: 8px;
    --radius-lg: 12px;
    --radius-xl: 16px;
    --radius-full: 9999px;

    /* === Отступы === */
    --space-xs: 0.25rem;
    --space-sm: 0.5rem;
    --space-md: 1rem;
    --space-lg: 1.5rem;
    --space-xl: 2rem;
    --space-2xl: 3rem;
  }
}
```

---

## 12.6 Адаптивные цветовые системы

### `light-dark()` — адаптация к теме

Позволяет определить оба варианта одновременно:

```css
:root {
  --surface: light-dark(white, /* Светлая тема */ #1b1b1b /* Тёмная тема */);

  --text: light-dark(#1a1a1a, #f0f0f0);

  --accent: light-dark(
    oklch(65% 0.22 255),
    oklch(70% 0.22 255) /* Автоматически адаптирован */
  );
}
```

Браузер автоматически выберет нужный цвет на основе `prefers-color-scheme`.

### `contrast-color()` — автоматический контраст

Одной из самых новых возможностей CSS Color Level 5 является функция `contrast-color()`, которая автоматически подбирает контрастный цвет текста относительно заданного фона.

```css
.text-on-accent {
  color: contrast-color(var(--accent));
}

/* Всплывающая подсказка с автоматическим контрастом */
.tooltip {
  background: var(--accent);
  color: contrast-color(var(--accent), white, black);
}
```

**На момент выхода книги:** эта возможность ещё не относится к **Baseline Widely available**, поэтому для массового продакшена требует проверки поддержки через `@supports` и резервного решения.

```css
@supports (color: contrast-color(white)) {
  .text-on-accent {
    color: contrast-color(var(--accent));
  }
}

@supports not (color: contrast-color(white)) {
  .text-on-accent {
    color: white; /* Fallback */
    text-shadow: 0 1px 2px rgba(0, 0, 0, 0.3);
  }
}
```

### Полная адаптивная система

```css
@layer tokens {
  :root {
    /* Базовые цвета — адаптируются автоматически */
    --surface: light-dark(oklch(98% 0.01 100), oklch(18% 0.01 100));

    --text: light-dark(oklch(20% 0.01 100), oklch(90% 0.01 100));

    --accent: oklch(65% 0.22 255);

    /* Вычисляемые состояния — адаптируются */
    --accent-hover: color-mix(
      in oklch,
      var(--accent),
      light-dark(white, black) 15%
    );

    --text-on-accent: contrast-color(var(--accent));
  }
}

/* Компонент использует адаптивную систему */
.card {
  background: var(--surface);
  color: var(--text);
  border-color: var(--border);
}

.card .accent-text {
  background: var(--accent);
  color: var(--text-on-accent);
}
```

---

## 12.7 Практические рекомендации

### Архитектурные правила

Современная архитектура цвета постепенно выработала несколько устойчивых правил.

**1. Используйте OKLCH как основное рабочее пространство**

```css
/* ✅ Рекомендуется */
:root {
  --primary: oklch(65% 0.22 255);
}

/* ❌ Устаревает */
:root {
  --primary: #2563eb;
}
```

**2. Храните цвета в виде дизайн-токенов**

```css
/* ✅ Рекомендуется */
:root {
  --color-primary: oklch(65% 0.22 255);
  --color-surface: light-dark(white, #1a1a1a);
}

/* ❌ Устаревает */
.button {
  background: #2563eb;
}
```

**3. Не создавайте вручную состояния hover и active — вычисляйте их**

```css
/* ✅ Рекомендуется */
:root {
  --primary: oklch(65% 0.22 255);
}

.button {
  background: var(--primary);

  &:hover {
    background: color-mix(in oklch, var(--primary), white 15%);
  }
}

/* ❌ Устаревает */
:root {
  --primary: #2563eb;
  --primary-hover: #1d4ed8;
  --primary-active: #1e40af;
}
```

**4. Для смешивания используйте `color-mix()`**

```css
/* ✅ Рекомендуется */
background: color-mix(in oklch, var(--primary), white 15%);

/* ❌ Устаревает */
background: rgba(37, 99, 235, 0.85);
```

**5. Для изменения каналов — Relative Color Syntax**

```css
/* ✅ Рекомендуется */
background: oklch(from var(--primary) calc(l - 0.08) c h);

/* ❌ Устаревает */
background: oklch(55% 0.22 255); /* Дублирование */
```

**6. Используйте `display-p3`, когда насыщенность действительно важна**

```css
/* ✅ Рекомендуется для ключевых брендовых цветов */
@supports (color: color(display-p3 0 0 0)) {
  --brand: color(display-p3 0.18 0.78 0.36);
}

/* ❌ Не используйте для всех цветов */
/* Это увеличивает сложность без необходимости */
```

**7. Проверяйте экспериментальные функции перед использованием в продакшене**

```css
/* ✅ С проверкой поддержки */
@supports (color: contrast-color(white)) {
  color: contrast-color(var(--accent));
}
@supports not (color: contrast-color(white)) {
  color: white;
}
```

### Проверка поддержки

```javascript
// Проверка поддержки OKLCH
const supportsOKLCH = CSS.supports('color', 'oklch(50% 0.1 100)');

// Проверка поддержки color-mix()
const supportsColorMix = CSS.supports(
  'color',
  'color-mix(in oklch, red, blue)',
);

// Проверка поддержки Relative Color Syntax
const supportsRelative = CSS.supports('color', 'oklch(from red l c h)');

// Проверка поддержки light-dark()
const supportsLightDark = CSS.supports('color', 'light-dark(black, white)');

// Проверка поддержки contrast-color()
const supportsContrast = CSS.supports('color', 'contrast-color(black)');

// Проверка поддержки Display-P3
const supportsP3 = CSS.supports('color', 'color(display-p3 0 0 0)');
```

---

## 12.8 Итоги главы

1. **CSS стал языком программируемого цвета** — цвет вычисляется, а не задаётся константами

2. **OKLCH — основное рабочее пространство** — перцептивная равномерность, предсказуемость

3. **Цвет становится функцией** — один базовый цвет → все состояния вычисляются

4. **`color-mix()`** — смешивание цветов в любом пространстве

5. **Relative Color Syntax** — изменение отдельных каналов

6. **Wide Gamut и Display-P3** — использование возможностей современных дисплеев

7. **`light-dark()`** — автоматическая адаптация к теме

8. **`contrast-color()`** — автоматический подбор контрастного текста (экспериментально)

9. **Дизайн-токены** — единая математическая модель цвета

10. **Практические правила:** OKLCH, токены, вычисляемые состояния, проверка поддержки

---

**Главная мысль:** Современный CSS окончательно превратил цвет из набора фиксированных констант в вычисляемую систему. Перцептивные пространства (`OKLCH`), функции `color-mix()` и Relative Color Syntax, поддержка широкого цветового охвата и адаптивные функции вроде `light-dark()` позволяют строить дизайн-системы, в которых большинство оттенков больше не хранится явно, а вычисляется браузером по заданным правилам. Именно такой подход сегодня становится основой масштабируемых интерфейсов: вместо ручного управления десятками оттенков разработчик описывает математическую модель цвета, а платформа автоматически адаптирует её к различным устройствам, темам оформления и сценариям использования.

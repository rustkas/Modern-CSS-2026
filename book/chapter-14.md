# Глава 14. Design Tokens 2026: архитектура современного CSS

> Современные дизайн-токены давно перестали быть простым набором CSS-переменных. Сегодня они представляют собой полноценный архитектурный слой, связывающий дизайн, код, компоненты и процессы сборки в единую систему. Именно токены становятся единым источником истины (*Single Source of Truth*), позволяя проектировать масштабируемые интерфейсы, независимые от конкретного фреймворка или библиотеки компонентов.

В этой главе дизайн-токены рассматриваются не как инструмент хранения цветов, а как язык описания интерфейса, опирающийся на современные возможности CSS.

---

## 14.1 От CSS Custom Properties к Design Tokens

### Эволюция переменных

Исторически CSS Custom Properties использовались как механизм повторного использования значений. Современные дизайн-токены идут значительно дальше.

```text
CSS Custom Properties (2016)
  ↓
Повторное использование значений
  ↓
Дизайн-токены (2020-2026)
  ↓
Архитектурный слой
  ↓
Единый источник истины
  ↓
Контракт между дизайном и кодом
```

### Три уровня абстракции

Они разделяют визуальную систему на несколько уровней абстракции:

```text
Primitive Tokens (фундаментальные значения)
  ↓
--blue-500, --space-4, --radius-md
  ↓
Semantic Tokens (назначение)
  ↓
--color-primary, --surface-background, --text-muted
  ↓
Component Tokens (настройки компонентов)
  ↓
--button-background, --dialog-shadow, --card-padding
```

**Такое разделение позволяет менять внешний вид интерфейса без изменения компонентов и значительно упрощает поддержку больших проектов.**

### Пример: система токенов

```css
@layer tokens {
  :root {
    /* === Primitive Tokens === */
    /* Цвета */
    --blue-500: oklch(65% 0.22 255);
    --blue-600: oklch(55% 0.22 255);
    --gray-100: oklch(98% 0.01 100);
    --gray-900: oklch(20% 0.01 100);
    
    /* Пространство */
    --space-1: 0.25rem;
    --space-2: 0.5rem;
    --space-3: 1rem;
    --space-4: 1.5rem;
    --space-5: 2rem;
    
    /* Радиусы */
    --radius-sm: 4px;
    --radius-md: 8px;
    --radius-lg: 12px;
    
    /* === Semantic Tokens === */
    --color-primary: var(--blue-500);
    --color-primary-hover: var(--blue-600);
    --color-surface: var(--gray-100);
    --color-text: var(--gray-900);
    --color-border: var(--gray-300);
    
    --space-md: var(--space-3);
    --space-lg: var(--space-4);
    
    --radius-card: var(--radius-md);
    --radius-button: var(--radius-sm);
    
    /* === Component Tokens === */
    --button-background: var(--color-primary);
    --button-background-hover: var(--color-primary-hover);
    --button-padding: var(--space-md) var(--space-lg);
    --button-radius: var(--radius-button);
    
    --card-padding: var(--space-lg);
    --card-radius: var(--radius-card);
    --card-shadow: 0 2px 8px rgba(0,0,0,0.08);
    
    --dialog-padding: var(--space-xl);
    --dialog-radius: var(--radius-lg);
    --dialog-shadow: 0 16px 48px rgba(0,0,0,0.16);
  }
}
```

---

## 14.2 Цветовые токены нового поколения

### От статики к вычислениям

Современный CSS превратил цветовые токены из статических значений в вычисляемую систему.

Вместо хранения большого количества заранее подготовленных оттенков используются:

```text
OKLCH
  ↓
Перцептивное цветовое пространство
  ↓
Relative Color Syntax
  ↓
Изменение каналов
  ↓
color-mix()
  ↓
Смешивание цветов
  ↓
light-dark()
  ↓
Адаптация к теме
```

### Пример: вычисляемые цветовые токены

```css
@layer tokens {
  :root {
    /* Базовый цвет — один источник */
    --color-primary-base: oklch(65% 0.22 255);
    
    /* Все состояния вычисляются */
    --color-primary: var(--color-primary-base);
    --color-primary-hover: color-mix(in oklch, var(--color-primary-base), white 15%);
    --color-primary-active: color-mix(in oklch, var(--color-primary-base), black 15%);
    --color-primary-subtle: color-mix(in oklch, var(--color-primary-base), transparent 85%);
    
    /* Адаптивные поверхности */
    --color-surface: light-dark(
      oklch(98% 0.01 100),
      oklch(18% 0.01 100)
    );
    
    --color-surface-raised: light-dark(
      oklch(96% 0.01 100),
      oklch(22% 0.01 100)
    );
    
    --color-text: light-dark(
      oklch(20% 0.01 100),
      oklch(90% 0.01 100)
    );
  }
}
```

### Преимущества

```text
Из одного базового токена
  ↓
Автоматически вычисляются состояния
  ↓
hover, active, disabled, selected, outline
  ↓
При этом сами токены остаются компактными
  ↓
Большая часть палитры вычисляется непосредственно браузером
```

Подробное описание цветовых функций приведено в главе 13.

---

## 14.3 Пространство, типографика и размеры

### Полноценная система

Современные токены описывают не только цвета. К полноценной системе относятся:

```text
Размеры
  ↓
Ширина, высота, максимальные/минимальные значения
  ↓
Интервалы
  ↓
Отступы, поля, зазоры
  ↓
Радиусы
  ↓
Скругления углов
  ↓
Типографика
  ↓
Шрифты, размеры, высота строки, трекинг
  ↓
Анимации
  ↓
Длительность, функции плавности (easing)
  ↓
Тени
  ↓
Уровни elevation
  ↓
Прозрачность
  ↓
Уровни opacity
```

### Адаптивные размеры

Размерные токены обычно строятся вокруг:

```text
clamp()
  ↓
Гибкие диапазоны
  ↓
lh / rlh
  ↓
Единицы высоты строки
  ↓
Контейнерные единицы
  ↓
cqi, cqb, cqw
```

### Пример: система размеров

```css
@layer tokens {
  :root {
    /* === Интервалы (масштабируемые) === */
    --space-xs: clamp(0.25rem, 0.5vw, 0.5rem);
    --space-sm: clamp(0.5rem, 1vw, 0.75rem);
    --space-md: clamp(1rem, 2vw, 1.5rem);
    --space-lg: clamp(1.5rem, 3vw, 2.5rem);
    --space-xl: clamp(2rem, 4vw, 3.5rem);
    
    /* === Радиусы === */
    --radius-sm: 4px;
    --radius-md: 8px;
    --radius-lg: 12px;
    --radius-xl: 16px;
    --radius-full: 9999px;
    
    /* === Типографика === */
    --font-family: system-ui, -apple-system, sans-serif;
    --font-size-sm: clamp(0.75rem, 1vw, 0.875rem);
    --font-size-base: clamp(1rem, 1.25vw, 1.125rem);
    --font-size-lg: clamp(1.125rem, 1.5vw, 1.25rem);
    --font-size-xl: clamp(1.25rem, 2vw, 1.5rem);
    --font-size-2xl: clamp(1.5rem, 2.5vw, 2rem);
    
    --line-height-tight: 1.2;
    --line-height-normal: 1.5;
    --line-height-loose: 1.8;
    
    /* === Анимации === */
    --duration-fast: 0.15s;
    --duration-medium: 0.3s;
    --duration-slow: 0.5s;
    
    --easing-standard: ease;
    --easing-spring: cubic-bezier(0.34, 1.56, 0.64, 1);
    --easing-smooth: cubic-bezier(0.4, 0, 0.2, 1);
    
    /* === Тени === */
    --shadow-sm: 0 1px 2px rgba(0,0,0,0.05);
    --shadow-md: 0 4px 12px rgba(0,0,0,0.08);
    --shadow-lg: 0 8px 24px rgba(0,0,0,0.12);
    --shadow-xl: 0 16px 48px rgba(0,0,0,0.16);
    
    /* === Elevation === */
    --elevation-1: var(--shadow-sm);
    --elevation-2: var(--shadow-md);
    --elevation-3: var(--shadow-lg);
    --elevation-4: var(--shadow-xl);
  }
}
```

**Благодаря этому интерфейс становится интринсик-системой, адаптирующейся к доступному пространству без большого количества медиазапросов.**

---

## 14.4 Типизированные токены через `@property`

### Регистрация пользовательских свойств

Одним из важнейших изменений современного CSS стало появление возможности регистрировать пользовательские свойства.

Использование `@property` позволяет определить:

```text
Тип данных
  ↓
<color>, <length>, <number>, <angle>, <time>
  ↓
Начальное значение
  ↓
initial-value
  ↓
Наследование
  ↓
inherits: true / false
```

### Пример: типизированные токены

```css
/* Типизированные цветовые токены */
@property --color-primary {
  syntax: '<color>';
  inherits: true;
  initial-value: oklch(65% 0.22 255);
}

@property --color-surface {
  syntax: '<color>';
  inherits: true;
  initial-value: oklch(98% 0.01 100);
}

/* Типизированные размерные токены */
@property --space-md {
  syntax: '<length>';
  inherits: false;
  initial-value: 1rem;
}

@property --radius-md {
  syntax: '<length>';
  inherits: false;
  initial-value: 8px;
}

/* Типизированные анимационные токены */
@property --duration-medium {
  syntax: '<time>';
  inherits: false;
  initial-value: 0.3s;
}

@property --rotation {
  syntax: '<angle>';
  inherits: false;
  initial-value: 0deg;
}
```

### Преимущества типизации

```text
Безопасная интерполяция
  ↓
Браузер знает тип → корректная анимация
  ↓
Корректная анимация
  ↓
Анимируются только поддерживаемые типы
  ↓
Автоматическая проверка типов
  ↓
Браузер проверяет значения
  ↓
Более предсказуемое поведение
  ↓
Меньше ошибок при использовании
```

Для масштабных дизайн-систем регистрация ключевых токенов становится частью архитектуры, а не исключительно средством создания анимаций (см. главу 2).

---

## 14.5 Токены компонентов

### Двухступенчатая схема

Современная библиотека компонентов практически никогда не использует примитивные токены напрямую.

Обычно применяется двухступенчатая схема:

```text
Primitive Token (фундаментальное значение)
  ↓
--blue-500
  ↓
Semantic Token (назначение)
  ↓
--color-primary
  ↓
Component Token (настройка компонента)
  ↓
--button-background
  ↓
CSS компонента
  ↓
background: var(--button-background)
```

### Пример: компонентные токены

```css
@layer tokens {
  :root {
    /* === Primitive === */
    --blue-500: oklch(65% 0.22 255);
    --blue-600: color-mix(in oklch, var(--blue-500), black 15%);
    
    /* === Semantic === */
    --color-primary: var(--blue-500);
    --color-primary-hover: var(--blue-600);
    
    /* === Component: Button === */
    --button-background: var(--color-primary);
    --button-background-hover: var(--color-primary-hover);
    --button-text: var(--color-text-inverse);
    --button-padding: var(--space-md) var(--space-lg);
    --button-radius: var(--radius-md);
    
    /* === Component: Card === */
    --card-background: var(--color-surface);
    --card-padding: var(--space-lg);
    --card-radius: var(--radius-lg);
    --card-shadow: var(--shadow-md);
    
    /* === Component: Dialog === */
    --dialog-background: var(--color-surface-raised);
    --dialog-padding: var(--space-xl);
    --dialog-radius: var(--radius-xl);
    --dialog-shadow: var(--shadow-xl);
  }
}

@layer components {
  .button {
    background: var(--button-background);
    color: var(--button-text);
    padding: var(--button-padding);
    border-radius: var(--button-radius);
    
    &:hover {
      background: var(--button-background-hover);
    }
  }
  
  .card {
    background: var(--card-background);
    padding: var(--card-padding);
    border-radius: var(--card-radius);
    box-shadow: var(--card-shadow);
  }
  
  .dialog {
    background: var(--dialog-background);
    padding: var(--dialog-padding);
    border-radius: var(--dialog-radius);
    box-shadow: var(--dialog-shadow);
  }
}
```

**Такой подход позволяет полностью заменить цветовую схему продукта без изменения компонентов. Именно компонентные токены становятся публичным API дизайн-системы.**

---

## 14.6 Автоматизация и единый источник истины

### Мультиплатформенность

Современные токены существуют одновременно в нескольких средах:

```text
Figma
  ↓
Среда дизайна
  ↓
Design Tokens Community Group
  ↓
Стандарт W3C
  ↓
JSON
  ↓
Универсальный формат
  ↓
Style Dictionary
  ↓
Генератор кода
  ↓
CSS, Android, iOS, Flutter, ...
```

### Типичный конвейер

```text
Figma (дизайн)
  ↓
Экспорт токенов в JSON
  ↓
Style Dictionary (трансформация)
  ↓
├── CSS Custom Properties (веб)
├── Android XML (мобильные)
├── iOS Swift (мобильные)
├── Flutter (кросс-платформа)
└── JavaScript (приложения)
```

**Благодаря этому одна и та же система токенов используется всеми платформами без ручной синхронизации.**

### Инструменты

```bash
# Style Dictionary — генерация токенов
npm install -D style-dictionary

# Конфигурация
{
  "source": ["tokens/**/*.json"],
  "platforms": {
    "css": {
      "transformGroup": "css",
      "buildPath": "build/css/",
      "files": [{
        "destination": "tokens.css",
        "format": "css/variables"
      }]
    }
  }
}

# Генерация
npx style-dictionary build
```

---

## 14.7 Экспериментальные возможности

### Технологии будущего

Не все современные возможности CSS одинаково зрелы.

**corner-shape и superellipse()**

Эти технологии открывают новые способы описания геометрии компонентов, однако пока не входят в Baseline и требуют использования `@supports` и безопасной деградации.

```css
/* Экспериментально */
@supports (corner-shape: superellipse) {
  .card {
    corner-shape: superellipse(1.2);
    border-radius: var(--radius-lg);
  }
}

/* Безопасная деградация */
.card {
  border-radius: var(--radius-lg);
}
```

### Стратегия внедрения

```text
Экспериментальные возможности
  ↓
Использовать только с @supports
  ↓
Предусматривать безопасную деградацию
  ↓
Токены формы отдельно от токенов радиуса
  ↓
При отсутствии поддержки → обычный border-radius
```

**Поэтому токены формы желательно проектировать отдельно от токенов радиуса, чтобы при отсутствии поддержки компонент автоматически возвращался к обычному `border-radius`.**

---

## 14.8 Итоги главы

1. **Три уровня токенов:** Primitive → Semantic → Component

2. **Цветовые токены нового поколения:** OKLCH, Relative Color Syntax, `color-mix()`, `light-dark()`

3. **Размерные токены:** `clamp()`, `lh`, контейнерные единицы (`cqi`, `cqb`)

4. **Типизированные токены:** `@property` для типов, начальных значений и наследования

5. **Компонентные токены:** публичный API дизайн-системы

6. **Автоматизация:** Figma → Design Tokens → Style Dictionary → CSS

7. **Экспериментальные возможности:** `corner-shape` с безопасной деградацией

---

**Главная мысль:** Современные дизайн-токены перестали быть просто способом избежать дублирования CSS-переменных. Они стали архитектурным контрактом между дизайном, компонентами и платформой, позволяя описывать интерфейс на уровне смыслов, а не конкретных значений. Цвета, размеры, типографика, пространства, анимации и геометрия компонентов формируют единую систему, которая автоматически адаптируется к различным темам, устройствам и контекстам использования.

При этом зрелая архитектура дизайн-токенов строится не только на использовании новых возможностей CSS, но и на понимании их статуса поддержки: большинство механизмов (Custom Properties, `color-mix()`, `light-dark()`, `clamp()`, контейнерные единицы) уже стали надёжной частью платформы, тогда как такие возможности, как `corner-shape`, пока следует внедрять как прогрессивное улучшение с безопасной деградацией.

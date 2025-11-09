# Подготовка к собеседованию: Strong Middle Frontend React Developer (Часть 5)

## CSS углубленно

---

### 16. CSS Специфичность и Каскад

**Как работает:**

CSS специфичность определяет, какой стиль будет применен при конфликте правил.

**Вес селекторов (от меньшего к большему):**

```css
/* 0,0,0,1 - элемент */
div { color: blue; }

/* 0,0,1,0 - класс */
.button { color: red; }

/* 0,0,1,1 - класс + элемент */
div.button { color: green; }

/* 0,1,0,0 - ID */
#header { color: purple; }

/* 1,0,0,0 - inline стили */
<div style="color: orange;">Text</div>

/* ∞ - !important (избегайте!) */
.text { color: black !important; }
```

**Расчет специфичности:**

```css
/* Специфичность: (IDs, Classes/Attributes/Pseudo-classes, Elements/Pseudo-elements) */

/* 0,0,1 */
.class { }

/* 0,1,0 */
#id { }

/* 0,0,2 */
.class1.class2 { }

/* 0,1,1 */
#id .class { }

/* 0,0,3 */
ul li a { }

/* 0,1,3 */
#nav ul li a { }

/* 0,2,1 */
.header .nav a { }

/* 0,0,1,1 - атрибут селектор = класс */
a[href*="example"] { }

/* 0,0,1,1 - pseudo-class = класс */
a:hover { }

/* 0,0,0,2 - pseudo-element */
p::before { }
p::after { }
```

**Примеры конфликтов:**

```html
<div id="container" class="box active">
  <p class="text">Hello</p>
</div>

<style>
/* Какой цвет победит? */

/* 0,0,1 */
.text { color: blue; }

/* 0,0,2 */
.box .text { color: red; }

/* 0,1,1 */
#container .text { color: green; } /* ← Победитель (самая высокая специфичность) */

/* 0,0,3 */
.active .box .text { color: yellow; }

/* Результат: green */
</style>
```

**Каскад (порядок применения стилей):**

```css
/* 1. Origin и Importance */
/*    a) user agent !important */
/*    b) user !important */
/*    c) author !important */
/*    d) CSS animations */
/*    e) author normal */
/*    f) user normal */
/*    g) user agent normal */

/* 2. Специфичность */
/* 3. Порядок появления (последний побеждает) */

.button { color: red; }
.button { color: blue; } /* ← Победитель (при равной специфичности) */
```

**Правила специфичности:**

```css
/* :not() не имеет специфичности, но его содержимое имеет */
div:not(.excluded) { } /* 0,0,1,1 (div + class) */

/* :is() и :where() */
:is(.class1, .class2) { } /* 0,0,1 - берет максимальную специфичность */
:where(.class1, .class2) { } /* 0,0,0 - всегда 0! */

/* :has() */
div:has(.child) { } /* 0,0,1,1 */

/* Universal selector (*) не добавляет специфичность */
* { margin: 0; } /* 0,0,0 */
```

**Best Practices:**

```css
/* ✅ Используйте классы для стилизации */
.button { }
.button--primary { }
.button--large { }

/* ❌ Избегайте ID селекторов для стилей */
#button { } /* Слишком высокая специфичность */

/* ❌ Избегайте !important */
.text { color: red !important; } /* Сложно переопределить */

/* ✅ Используйте BEM для избежания конфликтов */
.block__element--modifier { }

/* ✅ Используйте :where() для обнуления специфичности */
:where(.button) { /* 0,0,0 */ }
.button--primary { /* 0,0,1 - легко переопределит */ }

/* ✅ Структурируйте CSS от общего к частному */
/* Базовые стили */
button { }

/* Компонентные стили */
.button { }

/* Модификаторы */
.button--primary { }

/* Состояния */
.button:hover { }
.button:disabled { }

/* Контекстные стили (с осторожностью) */
.header .button { }
```

**CSS Cascade Layers (новая возможность):**

```css
/* Определение слоев (меньший приоритет → больший) */
@layer reset, base, components, utilities;

@layer reset {
  * { margin: 0; padding: 0; }
}

@layer base {
  body { font-family: Arial; }
}

@layer components {
  .button { background: blue; }
}

@layer utilities {
  .mt-4 { margin-top: 1rem; }
}

/* Стили без слоя имеют наивысший приоритет */
.button { background: red; } /* Переопределит .button из @layer components */

/* Преимущества:
   - Явный контроль порядка
   - Снижение необходимости в !important
   - Лучшая организация кода
*/
```

**Для чего:**
- Понимание как браузер применяет стили
- Избежание конфликтов стилей
- Правильная архитектура CSS

**Плюсы понимания специфичности:**
- Меньше !important
- Предсказуемые стили
- Легче отлаживать

**Минусы высокой специфичности:**
- Сложно переопределить
- Хрупкий код
- Конфликты стилей

---

### 17. CSS Grid vs Flexbox - когда что использовать

**Как работает:**

Flexbox - одномерный layout (row или column).
Grid - двумерный layout (rows и columns одновременно).

**Flexbox:**

```css
/* Одномерный контейнер */
.container {
  display: flex;
  
  /* Направление */
  flex-direction: row; /* row | row-reverse | column | column-reverse */
  
  /* Перенос */
  flex-wrap: wrap; /* nowrap | wrap | wrap-reverse */
  
  /* Короткая запись */
  flex-flow: row wrap; /* flex-direction + flex-wrap */
  
  /* Выравнивание по главной оси */
  justify-content: space-between; /* flex-start | flex-end | center | space-between | space-around | space-evenly */
  
  /* Выравнивание по поперечной оси */
  align-items: center; /* flex-start | flex-end | center | baseline | stretch */
  
  /* Выравнивание строк при wrap */
  align-content: flex-start; /* flex-start | flex-end | center | space-between | space-around | stretch */
  
  /* Расстояние между элементами */
  gap: 1rem; /* row-gap + column-gap */
}

.item {
  /* Рост при наличии свободного места */
  flex-grow: 1; /* 0 (default) | positive number */
  
  /* Сжатие при нехватке места */
  flex-shrink: 1; /* 1 (default) | 0 | positive number */
  
  /* Базовый размер */
  flex-basis: auto; /* auto | length | % */
  
  /* Короткая запись */
  flex: 1 1 auto; /* flex-grow + flex-shrink + flex-basis */
  
  /* Порядок */
  order: 0; /* integer */
  
  /* Индивидуальное выравнивание */
  align-self: center; /* auto | flex-start | flex-end | center | baseline | stretch */
}
```

**Типичные Flexbox паттерны:**

```css
/* 1. Навигационное меню */
.nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

/* 2. Карточки равной высоты */
.cards {
  display: flex;
  gap: 1rem;
}
.card {
  flex: 1; /* Все карточки равной ширины */
}

/* 3. Центрирование */
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
}

/* 4. Sticky footer */
.page {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}
.content {
  flex: 1; /* Растягивается на доступное место */
}

/* 5. Media object */
.media {
  display: flex;
  gap: 1rem;
}
.media__image {
  flex-shrink: 0; /* Изображение не сжимается */
}
.media__content {
  flex: 1; /* Контент занимает оставшееся место */
}

/* 6. Responsive layout без media queries */
.items {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}
.item {
  flex: 1 1 300px; /* Минимум 300px, потом wrap */
}
```

**CSS Grid:**

```css
/* Двумерная сетка */
.container {
  display: grid;
  
  /* Определение колонок */
  grid-template-columns: 200px 1fr 200px; /* fixed | fr | auto | % */
  grid-template-columns: repeat(3, 1fr); /* 3 равные колонки */
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); /* Адаптивная сетка */
  
  /* Определение рядов */
  grid-template-rows: 100px auto 100px;
  grid-template-rows: repeat(3, minmax(100px, auto));
  
  /* Именованные области */
  grid-template-areas:
    "header header header"
    "sidebar content content"
    "footer footer footer";
  
  /* Расстояние между ячейками */
  gap: 1rem; /* row-gap + column-gap */
  column-gap: 2rem;
  row-gap: 1rem;
  
  /* Выравнивание ячеек */
  justify-items: start; /* start | end | center | stretch */
  align-items: start; /* start | end | center | stretch */
  
  /* Выравнивание всей сетки */
  justify-content: center; /* start | end | center | space-between | space-around | space-evenly */
  align-content: center;
  
  /* Автоматическое размещение */
  grid-auto-flow: row; /* row | column | dense */
  grid-auto-rows: 100px; /* Размер автоматически созданных рядов */
  grid-auto-columns: 100px;
}

.item {
  /* Позиционирование по колонкам */
  grid-column: 1 / 3; /* От линии 1 до линии 3 */
  grid-column: 1 / span 2; /* Начать с 1, занять 2 колонки */
  grid-column: span 2; /* Занять 2 колонки */
  
  /* Позиционирование по рядам */
  grid-row: 1 / 3;
  grid-row: 1 / span 2;
  
  /* Короткая запись */
  grid-area: 1 / 1 / 3 / 3; /* row-start / col-start / row-end / col-end */
  
  /* Именованная область */
  grid-area: header;
  
  /* Индивидуальное выравнивание */
  justify-self: center; /* start | end | center | stretch */
  align-self: center;
}
```

**Типичные Grid паттерны:**

```css
/* 1. Holy Grail Layout */
.page {
  display: grid;
  grid-template-areas:
    "header header header"
    "nav content aside"
    "footer footer footer";
  grid-template-columns: 200px 1fr 200px;
  grid-template-rows: auto 1fr auto;
  min-height: 100vh;
}
.header { grid-area: header; }
.nav { grid-area: nav; }
.content { grid-area: content; }
.aside { grid-area: aside; }
.footer { grid-area: footer; }

/* 2. Адаптивная сетка карточек */
.cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
}

/* 3. Centered layout */
.container {
  display: grid;
  place-items: center; /* justify-items: center + align-items: center */
  min-height: 100vh;
}

/* 4. Responsive без media queries */
.layout {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
}

/* 5. Masonry-style (Pinterest layout) */
.masonry {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  grid-auto-rows: 10px; /* Маленький шаг */
}
.masonry-item {
  grid-row: span 20; /* Базовая высота */
}
.masonry-item--tall {
  grid-row: span 40;
}

/* 6. Overlay */
.overlay-container {
  display: grid;
}
.background,
.foreground {
  grid-area: 1 / 1; /* Оба в одной ячейке */
}

/* 7. 12-колоночная сетка */
.grid-12 {
  display: grid;
  grid-template-columns: repeat(12, 1fr);
  gap: 1rem;
}
.col-6 { grid-column: span 6; }
.col-4 { grid-column: span 4; }
.col-3 { grid-column: span 3; }
```

**Когда использовать Flexbox:**

```css
/* ✅ Одномерные layouts */
/* Навигация */
.nav { display: flex; }

/* Списки элементов в ряд/колонку */
.list { display: flex; }

/* Выравнивание контента в одном направлении */
.centered { display: flex; justify-content: center; }

/* Равномерное распределение элементов */
.items { display: flex; gap: 1rem; }

/* Когда порядок элементов может меняться */
.sortable { display: flex; }
.item { order: 1; }

/* Компоненты с неизвестным количеством элементов */
.tags { display: flex; flex-wrap: wrap; }
```

**Когда использовать Grid:**

```css
/* ✅ Двумерные layouts */
/* Основная структура страницы */
.page { display: grid; }

/* Карточки/галереи одинакового размера */
.gallery {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
}

/* Сложные layouts с областями */
.dashboard { display: grid; grid-template-areas: "..."; }

/* Когда нужно точное позиционирование */
.calendar { display: grid; }
.event { grid-column: 2 / 4; grid-row: 3 / 5; }

/* Overlapping элементы */
.hero { display: grid; }
.hero-image,
.hero-text {
  grid-area: 1 / 1; /* Наложение */
}
```

**Комбинирование Flexbox и Grid:**

```css
/* Grid для общего layout */
.page {
  display: grid;
  grid-template-areas:
    "header"
    "main"
    "footer";
}

/* Flexbox для компонентов внутри */
.header {
  grid-area: header;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.nav {
  display: flex;
  gap: 1rem;
}

/* Grid для карточек */
.cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 2rem;
}

/* Flexbox внутри карточки */
.card {
  display: flex;
  flex-direction: column;
}
.card__content {
  flex: 1;
}
```

**Для чего:**
- Flexbox: одномерные layouts, компоненты, навигация
- Grid: двумерные layouts, страничная структура, галереи

**Плюсы Flexbox:**
- Проще для одномерных layouts
- Лучше для контента неизвестного размера
- Хорошая поддержка браузеров

**Плюсы Grid:**
- Мощный для сложных layouts
- Меньше кода для двумерных структур
- Точное позиционирование
- Responsive без media queries

**Правило большого пальца:**

```
Flexbox = Content first (контент определяет layout)
Grid = Layout first (layout определяет расположение контента)

Используйте Flexbox когда:
- Одномерный layout (row ИЛИ column)
- Размеры элементов зависят от контента
- Нужна гибкость в распределении пространства

Используйте Grid когда:
- Двумерный layout (rows И columns)
- Нужна точная структура
- Элементы должны выравниваться в обоих направлениях
```

---

### 18. CSS-in-JS подходы и сравнение

**Как работает:**

CSS-in-JS - подход, где стили пишутся в JavaScript и генерируются динамически.

**Основные библиотеки:**

**1. Styled Components:**

```javascript
import styled from 'styled-components';

// Создание компонента со стилями
const Button = styled.button`
  background: ${props => props.primary ? 'blue' : 'gray'};
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  
  /* Вложенные стили */
  &:hover {
    opacity: 0.8;
  }
  
  /* Media queries */
  @media (max-width: 768px) {
    padding: 8px 16px;
  }
`;

// Наследование стилей
const PrimaryButton = styled(Button)`
  background: blue;
  font-weight: bold;
`;

// Использование
function App() {
  return (
    <>
      <Button>Normal</Button>
      <Button primary>Primary</Button>
      <PrimaryButton>Primary Bold</PrimaryButton>
    </>
  );
}

// Стили на основе props
const Container = styled.div`
  display: ${props => props.flex ? 'flex' : 'block'};
  padding: ${props => props.padding || '1rem'};
  background: ${props => {
    switch(props.variant) {
      case 'primary': return 'blue';
      case 'secondary': return 'gray';
      default: return 'white';
    }
  }};
`;

// Темизация
import { ThemeProvider } from 'styled-components';

const theme = {
  colors: {
    primary: '#007bff',
    secondary: '#6c757d',
  },
  spacing: {
    small: '0.5rem',
    medium: '1rem',
    large: '2rem',
  }
};

const ThemedButton = styled.button`
  background: ${props => props.theme.colors.primary};
  padding: ${props => props.theme.spacing.medium};
`;

function App() {
  return (
    <ThemeProvider theme={theme}>
      <ThemedButton>Themed Button</ThemedButton>
    </ThemeProvider>
  );
}

// Global styles
import { createGlobalStyle } from 'styled-components';

const GlobalStyle = createGlobalStyle`
  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }
  
  body {
    font-family: Arial, sans-serif;
    background: ${props => props.theme.colors.background};
  }
`;

function App() {
  return (
    <>
      <GlobalStyle />
      {/* rest of app */}
    </>
  );
}
```

**2. Emotion:**

```javascript
/** @jsxImportSource @emotion/react */
import { css } from '@emotion/react';
import styled from '@emotion/styled';

// Подход 1: css prop
function Button({ primary, children }) {
  return (
    <button
      css={css`
        background: ${primary ? 'blue' : 'gray'};
        color: white;
        padding: 10px 20px;
        border: none;
        border-radius: 4px;
        
        &:hover {
          opacity: 0.8;
        }
      `}
    >
      {children}
    </button>
  );
}

// Подход 2: styled components (как styled-components)
const Button = styled.button`
  background: ${props => props.primary ? 'blue' : 'gray'};
  color: white;
  padding: 10px 20px;
`;

// Подход 3: css функция для переиспользования
const buttonStyles = (primary) => css`
  background: ${primary ? 'blue' : 'gray'};
  color: white;
  padding: 10px 20px;
`;

function Button({ primary, children }) {
  return (
    <button css={buttonStyles(primary)}>
      {children}
    </button>
  );
}

// Композиция стилей
const baseStyles = css`
  padding: 10px;
  border-radius: 4px;
`;

const primaryStyles = css`
  background: blue;
  color: white;
`;

function Button() {
  return (
    <button css={[baseStyles, primaryStyles]}>
      Button
    </button>
  );
}
```

**3. CSS Modules:**

```css
/* Button.module.css */
.button {
  background: blue;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
}

.button:hover {
  opacity: 0.8;
}

.primary {
  background: darkblue;
  font-weight: bold;
}

/* Композиция */
.largeButton {
  composes: button;
  padding: 15px 30px;
  font-size: 1.2em;
}
```

```javascript
// Button.jsx
import styles from './Button.module.css';

function Button({ primary, large, children }) {
  const className = [
    styles.button,
    primary && styles.primary,
    large && styles.largeButton
  ].filter(Boolean).join(' ');
  
  return (
    <button className={className}>
      {children}
    </button>
  );
}

// Или с библиотекой classnames
import classNames from 'classnames';

function Button({ primary, large, children }) {
  return (
    <button
      className={classNames(styles.button, {
        [styles.primary]: primary,
        [styles.largeButton]: large
      })}
    >
      {children}
    </button>
  );
}
```

**4. Tailwind CSS:**

```javascript
// Utility-first подход
function Button({ primary, large, children }) {
  const baseClasses = "px-4 py-2 rounded border-none cursor-pointer";
  const variantClasses = primary
    ? "bg-blue-500 text-white hover:bg-blue-600"
    : "bg-gray-500 text-white hover:bg-gray-600";
  const sizeClasses = large ? "px-6 py-3 text-lg" : "";
  
  return (
    <button className={`${baseClasses} ${variantClasses} ${sizeClasses}`}>
      {children}
    </button>
  );
}

// С библиотекой clsx
import clsx from 'clsx';

function Button({ primary, large, disabled, children }) {
  return (
    <button
      className={clsx(
        "px-4 py-2 rounded border-none cursor-pointer transition-colors",
        {
          "bg-blue-500 text-white hover:bg-blue-600": primary,
          "bg-gray-500 text-white hover:bg-gray-600": !primary,
          "px-6 py-3 text-lg": large,
          "opacity-50 cursor-not-allowed": disabled
        }
      )}
      disabled={disabled}
    >
      {children}
    </button>
  );
}

// Кастомные компоненты с Tailwind
const Button = ({ variant = 'primary', size = 'md', children, ...props }) => {
  const variants = {
    primary: 'bg-blue-500 hover:bg-blue-600 text-white',
    secondary: 'bg-gray-500 hover:bg-gray-600 text-white',
    outline: 'border-2 border-blue-500 text-blue-500 hover:bg-blue-50'
  };
  
  const sizes = {
    sm: 'px-3 py-1 text-sm',
    md: 'px-4 py-2',
    lg: 'px-6 py-3 text-lg'
  };
  
  return (
    <button
      className={clsx(
        'rounded font-semibold transition-colors',
        variants[variant],
        sizes[size]
      )}
      {...props}
    >
      {children}
    </button>
  );
};
```

**5. Vanilla Extract:**

```typescript
// button.css.ts
import { style } from '@vanilla-extract/css';

export const button = style({
  background: 'blue',
  color: 'white',
  padding: '10px 20px',
  border: 'none',
  borderRadius: '4px',
  ':hover': {
    opacity: 0.8
  }
});

export const primary = style({
  background: 'darkblue',
  fontWeight: 'bold'
});

// Theme
import { createTheme } from '@vanilla-extract/css';

export const [themeClass, vars] = createTheme({
  color: {
    primary: '#007bff',
    secondary: '#6c757d'
  },
  spacing: {
    small: '0.5rem',
    medium: '1rem',
    large: '2rem'
  }
});

export const themedButton = style({
  background: vars.color.primary,
  padding: vars.spacing.medium
});
```

```javascript
// Button.jsx
import { button, primary, themedButton } from './button.css';

function Button({ isPrimary, children }) {
  return (
    <button className={isPrimary ? `${button} ${primary}` : button}>
      {children}
    </button>
  );
}
```

**Сравнение подходов:**

| Характеристика | Styled Components | Emotion | CSS Modules | Tailwind | Vanilla Extract |
|---------------|-------------------|---------|-------------|----------|-----------------|
| Runtime | Да | Да | Нет | Нет | Нет |
| Type Safety | Нет | Частично | Нет | С plugin | Да |
| Bundle Size | Средний | Средний | Маленький | Большой* | Маленький |
| Performance | Средняя | Средняя | Высокая | Высокая | Высокая |
| Learning Curve | Низкая | Низкая | Низкая | Средняя | Средняя |
| Динамические стили | Легко | Легко | Сложно | Сложно | Средне |
| SSR | Да | Да | Да | Да | Да |
| DevEx | Отлично | Отлично | Хорошо | Отлично | Отлично |

*С PurgeCSS становится маленьким

**Performance сравнение:**

```javascript
// Runtime CSS-in-JS (Styled Components, Emotion)
// ❌ Минусы:
// - Стили генерируются в runtime
// - Увеличивает bundle size
// - Может влиять на First Paint

// ✅ Плюсы:
// - Полностью динамические стили
// - Props-based styling
// - Автоматический critical CSS

// Zero-runtime CSS-in-JS (CSS Modules, Vanilla Extract)
// ✅ Плюсы:
// - Стили извлекаются в build time
// - Меньший bundle size
// - Быстрый First Paint

// ❌ Минусы:
// - Меньше динамичности
// - Нужно больше кода для условных стилей

// Utility-first (Tailwind)
// ✅ Плюсы:
// - Очень быстрая разработка
// - Консистентный design system
// - С PurgeCSS - маленький размер

// ❌ Минусы:
// - "Грязная" разметка
// - Сложно кастомизировать
// - Крутая кривая обучения
```

**Когда использовать что:**

```javascript
// Styled Components / Emotion
// ✅ Когда нужны:
// - Полностью динамические стили на основе props
// - Тесная интеграция с React
// - Компонентный подход
// - Runtime theming

const Button = styled.button`
  background: ${props => props.theme.colors[props.variant]};
  padding: ${props => props.theme.spacing[props.size]};
`;

// CSS Modules
// ✅ Когда нужны:
// - Максимальная производительность
// - Традиционный CSS workflow
// - Нет динамических стилей
// - SSR оптимизация

// Tailwind
// ✅ Когда нужны:
// - Быстрая разработка
// - Консистентный design system из коробки
// - Responsive design
// - Прототипирование

// Vanilla Extract
// ✅ Когда нужны:
// - Type safety
// - Zero-runtime
// - TypeScript integration
// - Лучшее из обоих миров
```

**Best Practices:**

```javascript
// 1. Избегайте inline стилей в production
// ❌ Плохо
<div style={{ color: 'red', padding: '10px' }}>Text</div>

// ✅ Хорошо
const Text = styled.div`
  color: red;
  padding: 10px;
`;

// 2. Используйте темизацию
const theme = {
  colors: { primary: '#007bff' },
  spacing: { md: '1rem' }
};

// 3. Оптимизируйте для production
// Styled Components
import { StyleSheetManager } from 'styled-components';

<StyleSheetManager disableVendorPrefixes>
  <App />
</StyleSheetManager>

// Emotion
/** @jsxImportSource @emotion/react */
import { CacheProvider } from '@emotion/react';
import createCache from '@emotion/cache';

const cache = createCache({ key: 'css' });

<CacheProvider value={cache}>
  <App />
</CacheProvider>

// 4. Code splitting для стилей
// Используйте React.lazy для компонентов со стилями
const HeavyComponent = lazy(() => import('./HeavyComponent'));

// 5. Извлекайте переиспользуемые стили
// ❌ Плохо - дублирование
const Button = styled.button`
  padding: 10px;
  border-radius: 4px;
`;

const Input = styled.input`
  padding: 10px;
  border-radius: 4px;
`;

// ✅ Хорошо - переиспользование
const baseStyles = css`
  padding: 10px;
  border-radius: 4px;
`;

const Button = styled.button`
  ${baseStyles}
`;

const Input = styled.input`
  ${baseStyles}
`;
```

---

## Build Tools

### 19. Webpack vs Vite vs Rollup - различия и когда что использовать

**Webpack:**

**Как работает:**
- Bundler + Dev Server
- Использует CommonJS/ES Modules
- Горячая замена модулей (HMR)
- Плагины и loaders

```javascript
// webpack.config.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
  mode: 'production', // 'development' | 'production'
  
  entry: './src/index.js',
  
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash].js',
    clean: true
  },
  
  module: {
    rules: [
      {
        test: /\.jsx?$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-react']
          }
        }
      },
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader, 'css-loader']
      },
      {
        test: /\.(png|jpg|gif|svg)$/,
        type: 'asset/resource'
      }
    ]
  },
  
  plugins: [
    new HtmlWebpackPlugin({
      template: './src/index.html'
    }),
    new MiniCssExtractPlugin({
      filename: '[name].[contenthash].css'
    })
  ],
  
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          priority: 10
        }
      }
    }
  },
  
  devServer: {
    port: 3000,
    hot: true,
    open: true
  }
};
```

**Плюсы Webpack:**
- Мощная экосистема плагинов
- Зрелый и стабильный
- Поддержка legacy браузеров
- Asset management из коробки
- Code splitting и lazy loading

**Минусы Webpack:**
- Сложная конфигурация
- Медленный dev server (особенно для больших проектов)
- Долгий cold start
- Большая кривая обучения

**Vite:**

**Как работает:**
- Использует ESM в dev mode (нет bundling)
- Rollup для production
- Мгновенный HMR
- Быстрый cold start

```javascript
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  
  // Dev server
  server: {
    port: 3000,
    open: true,
    cors: true
  },
  
  // Build
  build: {
    outDir: 'dist',
    sourcemap: true,
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          utils: ['lodash', 'axios']
        }
      }
    }
  },
  
  // CSS
  css: {
    modules: {
      localsConvention: 'camelCase'
    },
    preprocessorOptions: {
      scss: {
        additionalData: '@import "./src/styles/variables.scss";'
      }
    }
  },
  
  // Alias
  resolve: {
    alias: {
      '@': '/src',
      '@components': '/src/components'
    }
  },
  
  // Environment variables
  define: {
    'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV)
  }
});
```

**Плюсы Vite:**
- Очень быстрый dev server
- Мгновенный HMR
- Простая конфигурация
- Из коробки поддержка TypeScript, JSX, CSS
- Оптимизированный build (Rollup)

**Минусы Vite:**
- Молодой проект (меньше плагинов)
- Некоторые пакеты не совместимы с ESM
- Различия между dev и production

**Rollup:**

**Как работает:**
- Изначально для библиотек
- Tree shaking из коробки
- Создает ES modules
- Более простой чем Webpack

```javascript
// rollup.config.js
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import babel from '@rollup/plugin-babel';
import terser from '@rollup/plugin-terser';
import postcss from 'rollup-plugin-postcss';

export default {
  input: 'src/index.js',
  
  output: [
    {
      file: 'dist/bundle.cjs.js',
      format: 'cjs', // CommonJS
      sourcemap: true
    },
    {
      file: 'dist/bundle.esm.js',
      format: 'esm', // ES Modules
      sourcemap: true
    },
    {
      file: 'dist/bundle.umd.js',
      format: 'umd', // UMD для браузеров
      name: 'MyLibrary',
      sourcemap: true
    }
  ],
  
  plugins: [
    resolve({
      extensions: ['.js', '.jsx']
    }),
    commonjs(),
    babel({
      babelHelpers: 'bundled',
      exclude: 'node_modules/**',
      presets: ['@babel/preset-react']
    }),
    postcss({
      extract: true,
      minimize: true
    }),
    terser()
  ],
  
  external: ['react', 'react-dom'] // Не включать в bundle
};
```

**Плюсы Rollup:**
- Лучший tree shaking
- Чище output
- Идеален для библиотек
- Множественные форматы (CJS, ESM, UMD)

**Минусы Rollup:**
- Меньше подходит для приложений
- Нет встроенного dev server
- Меньше плагинов чем у Webpack

**Сравнение:**

| Характеристика | Webpack | Vite | Rollup |
|---------------|---------|------|--------|
| Dev Server | Медленный | Очень быстрый | Нет |
| HMR | Да | Мгновенный | Нет |
| Build Speed | Медленный | Быстрый | Быстрый |
| Tree Shaking | Да | Да | Лучший |
| Config | Сложный | Простой | Средний |
| Use Case | Приложения | Приложения | Библиотеки |
| Ecosystem | Огромный | Растущий | Средний |
| Learning Curve | Высокая | Низкая | Средняя |

**Когда использовать:**

```javascript
// Webpack
// ✅ Когда:
// - Большое enterprise приложение
// - Нужна поддержка legacy браузеров
// - Много custom настроек
// - Существующий проект на Webpack
// - Нужны специфичные webpack плагины

// Vite
// ✅ Когда:
// - Новый проект
// - Современные браузеры
// - Нужна скорость разработки
// - React/Vue/Svelte приложение
// - TypeScript из коробки

// Rollup
// ✅ Когда:
// - Создаете библиотеку
// - Нужен минимальный bundle size
// - Множественные форматы output
// - Tree shaking критичен
```

**Migration от Webpack к Vite:**

```javascript
// 1. Установка
npm install -D vite @vitejs/plugin-react

// 2. package.json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  }
}

// 3. index.html в root (не в public)
<!DOCTYPE html>
<html>
  <head>
    <title>App</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>

// 4. Замена process.env
// Webpack
const API_URL = process.env.REACT_APP_API_URL;

// Vite
const API_URL = import.meta.env.VITE_API_URL;

// 5. .env файл
// VITE_API_URL=https://api.example.com

// 6. Замена require на import
// ❌ Webpack
const logo = require('./logo.png');

// ✅ Vite
import logo from './logo.png';

// 7. Dynamic imports остаются те же
const Component = lazy(() => import('./Component'));

// 8. vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': '/src'
    }
  }
});
```

**Для чего:**
- Webpack: комплексные приложения с кастомными требованиями
- Vite: быстрая разработка современных приложений
- Rollup: создание библиотек и оптимизированных бандлов

---

Это пятая часть. Продолжить с шестой частью (Testing, Git, Service Workers)?


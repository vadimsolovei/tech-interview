# Подготовка к собеседованию: Strong Middle Frontend React Developer (Часть 3)

## Производительность и оптимизация

---

### 9. Critical Rendering Path (Критический путь рендеринга)

**Как работает:**

Critical Rendering Path - последовательность шагов, которые браузер выполняет для преобразования HTML, CSS и JavaScript в пиксели на экране.

**Этапы CRP:**

```
1. DOM Construction (HTML → DOM)
2. CSSOM Construction (CSS → CSSOM)
3. Render Tree (DOM + CSSOM → Render Tree)
4. Layout (вычисление геометрии)
5. Paint (отрисовка пикселей)
6. Composite (композиция слоев)
```

**1. DOM Construction:**

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Page</title>
    <link rel="stylesheet" href="style.css"> <!-- Блокирует рендеринг -->
  </head>
  <body>
    <div class="container">
      <h1>Hello</h1>
      <p>World</p>
    </div>
    <script src="app.js"></script> <!-- Блокирует парсинг -->
  </body>
</html>
```

**Процесс построения DOM:**
```
HTML → Bytes → Characters → Tokens → Nodes → DOM Tree

<html>
  └─ <head>
      ├─ <title>
      └─ <link>
  └─ <body>
      └─ <div class="container">
          ├─ <h1>
          └─ <p>
```

**2. CSSOM Construction:**

```css
/* style.css */
body { font-size: 16px; }
.container { max-width: 1200px; margin: 0 auto; }
h1 { color: blue; font-size: 32px; }
p { color: gray; }
```

**CSSOM Tree:**
```
body
  ├─ font-size: 16px
  └─ .container
      ├─ max-width: 1200px
      ├─ margin: 0 auto
      ├─ font-size: 16px (наследуется)
      └─ h1
          ├─ color: blue
          ├─ font-size: 32px
          └─ font-size: 16px (перезаписано)
```

**3. Render Tree:**

```javascript
// Render Tree = DOM + CSSOM - невидимые элементы

DOM Tree:
- html
  - head
    - title (не в Render Tree - не визуальный)
    - link (не в Render Tree)
  - body
    - div.container
      - h1 (в Render Tree)
      - p (в Render Tree)
    - script (не в Render Tree)

Render Tree:
- body
  - div.container (max-width: 1200px, margin: 0 auto)
    - h1 (color: blue, font-size: 32px)
    - p (color: gray, font-size: 16px)
```

**Элементы не попадают в Render Tree:**
```css
/* Эти элементы не будут в Render Tree */
display: none;  /* Полностью удаляется из Render Tree */
<head>, <script>, <meta>  /* Не визуальные элементы */

/* Эти элементы БУДУТ в Render Tree */
visibility: hidden;  /* Занимает место, но невидим */
opacity: 0;  /* Занимает место, но прозрачен */
```

**4. Layout (Reflow):**

Вычисление точной позиции и размера каждого элемента.

```javascript
// Layout вычисляет:
// - Позицию (x, y)
// - Размеры (width, height)
// - Отступы (margin, padding)

// Пример вычислений для div.container:
const viewportWidth = window.innerWidth; // 1920px
const maxWidth = 1200; // из CSS
const computedWidth = Math.min(viewportWidth, maxWidth); // 1200px
const marginLeft = (viewportWidth - computedWidth) / 2; // 360px
const marginRight = marginLeft; // 360px

// Позиция h1 относительно родителя:
const h1Top = 0;
const h1Left = 0;
const h1Width = 1200; // ширина родителя
const h1Height = 32; // font-size + line-height
```

**Что вызывает Reflow (дорогая операция):**
```javascript
// ❌ Вызывают reflow (избегайте в циклах!)
element.offsetWidth, element.offsetHeight
element.clientWidth, element.clientHeight
element.scrollWidth, element.scrollHeight
element.getBoundingClientRect()
window.getComputedStyle(element)

// Изменения, вызывающие reflow:
element.style.width = '100px';
element.style.margin = '10px';
element.style.padding = '5px';
element.style.border = '1px solid';
element.style.fontSize = '16px';
element.style.position = 'absolute';
element.classList.add('large');
element.innerHTML = 'new content';

// ✅ Оптимизация - batch reads and writes
// ❌ Плохо - чередование read/write
for (let i = 0; i < elements.length; i++) {
  const width = elements[i].offsetWidth; // Read (reflow)
  elements[i].style.width = width + 10 + 'px'; // Write (reflow)
  // Reflow × elements.length
}

// ✅ Хорошо - сначала все reads, потом все writes
const widths = elements.map(el => el.offsetWidth); // Все reads
widths.forEach((width, i) => {
  elements[i].style.width = width + 10 + 'px'; // Все writes
});
// Reflow × 2 вместо × elements.length
```

**5. Paint:**

Заполнение пикселей (цвета, изображения, тени, границы).

```javascript
// Что вызывает repaint (дешевле reflow):
element.style.color = 'red';
element.style.backgroundColor = 'blue';
element.style.visibility = 'hidden';
element.style.outline = '1px solid red';
element.style.boxShadow = '0 0 10px rgba(0,0,0,0.5)';

// CSS свойства по стоимости:
// 1. Самые дешевые (только composite):
//    - transform, opacity, filter
// 2. Средние (repaint):
//    - color, background-color, box-shadow
// 3. Самые дорогие (reflow + repaint):
//    - width, height, margin, padding, border, font-size, position
```

**6. Composite:**

Отрисовка слоев друг над другом в правильном порядке (z-index).

```css
/* Создание нового композитного слоя (GPU ускорение): */
.element {
  /* Явное создание слоя: */
  will-change: transform;
  
  /* Или автоматически при использовании: */
  transform: translateZ(0); /* или translate3d(0,0,0) */
  transform: scale(1.1);
  opacity: 0.9;
  filter: blur(10px);
  
  /* НЕ создают слой: */
  position: relative;
  z-index: 1;
}
```

**Оптимизация CRP:**

**1. Оптимизация CSS:**
```html
<!-- ❌ Плохо - блокирует рендеринг -->
<link rel="stylesheet" href="styles.css">

<!-- ✅ Хорошо - критичный CSS inline -->
<style>
  /* Только стили для above-the-fold контента */
  body { margin: 0; font-family: Arial; }
  .header { height: 60px; background: #333; }
</style>

<!-- ✅ Некритичный CSS загружать асинхронно -->
<link rel="preload" href="styles.css" as="style" onload="this.onload=null;this.rel='stylesheet'">
<noscript><link rel="stylesheet" href="styles.css"></noscript>

<!-- ✅ Media queries для условной загрузки -->
<link rel="stylesheet" href="print.css" media="print">
<link rel="stylesheet" href="mobile.css" media="(max-width: 768px)">
```

**2. Оптимизация JavaScript:**
```html
<!-- ❌ Плохо - блокирует парсинг HTML -->
<script src="app.js"></script>

<!-- ✅ Хорошо - defer (выполняется после парсинга HTML) -->
<script src="app.js" defer></script>

<!-- ✅ async (выполняется асинхронно, как только загрузится) -->
<script src="analytics.js" async></script>

<!-- Разница между defer и async: -->
<!-- defer: скрипты выполняются по порядку, после DOMContentLoaded -->
<!-- async: скрипты выполняются сразу после загрузки, порядок не гарантирован -->

<!-- ✅ Динамическая загрузка скриптов -->
<script>
  // Загрузка после основного контента
  window.addEventListener('load', () => {
    const script = document.createElement('script');
    script.src = 'non-critical.js';
    document.body.appendChild(script);
  });
</script>

<!-- ✅ Module scripts (defer by default) -->
<script type="module" src="app.js"></script>
```

**3. Resource Hints:**
```html
<!-- DNS prefetch - резолв DNS заранее -->
<link rel="dns-prefetch" href="https://api.example.com">

<!-- Preconnect - установка соединения заранее -->
<link rel="preconnect" href="https://fonts.googleapis.com" crossorigin>

<!-- Prefetch - загрузка ресурса для следующей навигации -->
<link rel="prefetch" href="next-page.js">

<!-- Preload - загрузка критичного ресурса с высоким приоритетом -->
<link rel="preload" href="hero-image.jpg" as="image">
<link rel="preload" href="font.woff2" as="font" type="font/woff2" crossorigin>

<!-- Prerender (осторожно! - загружает всю страницу) -->
<link rel="prerender" href="next-page.html">
```

**4. Измерение CRP:**
```javascript
// Performance API
window.addEventListener('load', () => {
  const perfData = performance.timing;
  
  // Время до интерактивности
  const domInteractive = perfData.domInteractive - perfData.navigationStart;
  console.log('DOM Interactive:', domInteractive + 'ms');
  
  // Время полной загрузки
  const loadComplete = perfData.loadEventEnd - perfData.navigationStart;
  console.log('Load Complete:', loadComplete + 'ms');
  
  // DOMContentLoaded
  const domContentLoaded = perfData.domContentLoadedEventEnd - perfData.navigationStart;
  console.log('DOMContentLoaded:', domContentLoaded + 'ms');
  
  // First Paint
  const paintEntries = performance.getEntriesByType('paint');
  paintEntries.forEach(entry => {
    console.log(`${entry.name}: ${entry.startTime}ms`);
  });
});

// PerformanceObserver для более современного API
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log(`${entry.name}: ${entry.startTime}ms`);
  }
});

observer.observe({ 
  entryTypes: ['paint', 'navigation', 'resource', 'measure'] 
});
```

**Для чего:**
- Понимание как работает браузер
- Оптимизация времени загрузки
- Улучшение perceived performance
- Достижение хороших Web Vitals метрик

**Best Practices:**
```javascript
// Checklist оптимизации CRP:

// ✅ Минимизируйте количество критичных ресурсов
// ✅ Минимизируйте размер критичных ресурсов (minify, gzip)
// ✅ Используйте inline CSS для above-the-fold контента
// ✅ defer/async для некритичного JavaScript
// ✅ Оптимизируйте порядок загрузки ресурсов
// ✅ Используйте resource hints (preload, preconnect)
// ✅ Избегайте @import в CSS
// ✅ Минимизируйте количество reflows/repaints
// ✅ Используйте CSS animations вместо JavaScript
// ✅ Используйте transform и opacity для анимаций
```

---

### 10. Web Vitals (Core Web Vitals)

**Как работает:**

Web Vitals - набор метрик, измеряющих реальный пользовательский опыт загрузки страницы.

**Core Web Vitals (2023):**

**1. LCP (Largest Contentful Paint):**

Время отрисовки самого большого контентного элемента в видимой области.

```javascript
// Измерение LCP
new PerformanceObserver((entryList) => {
  const entries = entryList.getEntries();
  const lastEntry = entries[entries.length - 1];
  
  console.log('LCP:', lastEntry.renderTime || lastEntry.loadTime);
  console.log('LCP Element:', lastEntry.element);
  console.log('LCP Size:', lastEntry.size);
}).observe({ type: 'largest-contentful-paint', buffered: true });

// LCP учитывает:
// - <img> элементы
// - <image> внутри <svg>
// - <video> с poster
// - background-image через CSS
// - Блоковые элементы с текстом

// Целевые значения:
// Good: < 2.5s
// Needs Improvement: 2.5s - 4s
// Poor: > 4s
```

**Оптимизация LCP:**
```html
<!-- 1. Оптимизация изображений -->
<!-- ❌ Плохо -->
<img src="hero-large.jpg" alt="Hero">

<!-- ✅ Хорошо -->
<img 
  src="hero-small.jpg"
  srcset="hero-small.jpg 400w, hero-medium.jpg 800w, hero-large.jpg 1200w"
  sizes="(max-width: 640px) 400px, (max-width: 1024px) 800px, 1200px"
  alt="Hero"
  loading="eager"
  fetchpriority="high"
>

<!-- 2. Preload критичных изображений -->
<link rel="preload" as="image" href="hero.jpg" fetchpriority="high">

<!-- 3. Используйте современные форматы -->
<picture>
  <source srcset="hero.avif" type="image/avif">
  <source srcset="hero.webp" type="image/webp">
  <img src="hero.jpg" alt="Hero">
</picture>

<!-- 4. CDN для быстрой доставки -->
<img src="https://cdn.example.com/hero.jpg" alt="Hero">
```

```css
/* 5. Оптимизация CSS для LCP элемента */
.hero {
  /* ❌ Плохо - background-image загружается после CSS */
  background-image: url('hero.jpg');
  
  /* ✅ Лучше использовать <img> с preload */
}

/* 6. Избегайте CSS блокировки */
/* Критичный CSS должен быть inline */
```

```javascript
// 7. Минимизация JavaScript выполнения
// ❌ Плохо - тяжелые вычисления блокируют main thread
for (let i = 0; i < 1000000; i++) {
  // heavy computation
}

// ✅ Хорошо - разбить на chunks
function heavyTask() {
  let i = 0;
  function processChunk() {
    const end = Math.min(i + 1000, 1000000);
    for (; i < end; i++) {
      // process
    }
    if (i < 1000000) {
      requestIdleCallback(processChunk);
    }
  }
  processChunk();
}

// ✅ Или использовать Web Worker
const worker = new Worker('heavy-task.js');
worker.postMessage({ task: 'compute' });
```

**2. INP (Interaction to Next Paint):**

Заменил FID в 2024. Измеряет задержку между взаимодействием пользователя и визуальным откликом.

```javascript
// Измерение INP
new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    const duration = entry.processingEnd - entry.startTime;
    console.log('Interaction:', entry.name);
    console.log('Duration:', duration);
    console.log('Target:', entry.target);
  }
}).observe({ type: 'event', buffered: true, durationThreshold: 16 });

// INP учитывает все взаимодействия:
// - Click
// - Tap
// - Keyboard input

// Целевые значения:
// Good: < 200ms
// Needs Improvement: 200ms - 500ms
// Poor: > 500ms
```

**Оптимизация INP:**
```javascript
// 1. Debounce и throttle для частых событий
// ❌ Плохо - выполняется на каждое событие
input.addEventListener('input', (e) => {
  heavySearch(e.target.value);
});

// ✅ Хорошо - debounce
const debouncedSearch = debounce((value) => {
  heavySearch(value);
}, 300);

input.addEventListener('input', (e) => {
  debouncedSearch(e.target.value);
});

function debounce(func, wait) {
  let timeout;
  return function executedFunction(...args) {
    const later = () => {
      clearTimeout(timeout);
      func(...args);
    };
    clearTimeout(timeout);
    timeout = setTimeout(later, wait);
  };
}

// 2. Оптимизация event handlers
// ❌ Плохо - тяжелые вычисления в обработчике
button.addEventListener('click', () => {
  for (let i = 0; i < 1000000; i++) {
    // heavy computation
  }
  updateUI();
});

// ✅ Хорошо - разделение на frames
button.addEventListener('click', async () => {
  // Показываем loading сразу
  showLoading();
  
  // Ждем следующий frame
  await new Promise(resolve => requestAnimationFrame(resolve));
  
  // Выполняем тяжелую работу
  const result = await heavyComputation();
  
  // Обновляем UI
  updateUI(result);
  hideLoading();
});

// 3. Passive event listeners для scroll/touch
// ✅ Не блокирует прокрутку
window.addEventListener('scroll', handleScroll, { passive: true });
window.addEventListener('touchstart', handleTouch, { passive: true });

// 4. Виртуализация для больших списков
// React
import { FixedSizeList } from 'react-window';

function VirtualList({ items }) {
  return (
    <FixedSizeList
      height={600}
      itemCount={items.length}
      itemSize={50}
      width="100%"
    >
      {({ index, style }) => (
        <div style={style}>{items[index]}</div>
      )}
    </FixedSizeList>
  );
}

// 5. Code splitting для уменьшения JS bundle
// ❌ Плохо - весь код загружается сразу
import HeavyComponent from './HeavyComponent';

// ✅ Хорошо - lazy loading
const HeavyComponent = React.lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <HeavyComponent />
    </Suspense>
  );
}

// 6. Избегание layout thrashing
// ❌ Плохо
elements.forEach(el => {
  const height = el.offsetHeight; // Read
  el.style.height = height + 10 + 'px'; // Write
  // Force reflow на каждой итерации
});

// ✅ Хорошо
const heights = elements.map(el => el.offsetHeight); // All reads
elements.forEach((el, i) => {
  el.style.height = heights[i] + 10 + 'px'; // All writes
});
```

**3. CLS (Cumulative Layout Shift):**

Сумма всех неожиданных сдвигов макета во время загрузки страницы.

```javascript
// Измерение CLS
let clsValue = 0;
let clsEntries = [];

new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    // Только unexpected shifts (не от user interaction)
    if (!entry.hadRecentInput) {
      clsValue += entry.value;
      clsEntries.push(entry);
      console.log('CLS:', clsValue);
      console.log('Shifted elements:', entry.sources);
    }
  }
}).observe({ type: 'layout-shift', buffered: true });

// Целевые значения:
// Good: < 0.1
// Needs Improvement: 0.1 - 0.25
// Poor: > 0.25
```

**Оптимизация CLS:**
```html
<!-- 1. Всегда указывайте размеры изображений -->
<!-- ❌ Плохо - изображение "прыгает" после загрузки -->
<img src="image.jpg" alt="Image">

<!-- ✅ Хорошо - резервируется место -->
<img src="image.jpg" alt="Image" width="800" height="600">

<!-- ✅ Или через CSS aspect ratio -->
<style>
  .image-container {
    aspect-ratio: 16 / 9;
  }
  .image-container img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
</style>
<div class="image-container">
  <img src="image.jpg" alt="Image">
</div>

<!-- 2. Резервируйте место для динамического контента -->
<!-- ❌ Плохо - баннер появляется и сдвигает контент -->
<div id="banner"></div>
<main>Content</main>

<script>
  fetch('/api/banner')
    .then(res => res.text())
    .then(html => {
      document.getElementById('banner').innerHTML = html;
      // CLS! Контент сдвигается вниз
    });
</script>

<!-- ✅ Хорошо - место зарезервировано -->
<div id="banner" style="min-height: 200px;">
  <div class="skeleton"></div>
</div>
<main>Content</main>

<!-- 3. Избегайте вставки контента выше существующего -->
<!-- ❌ Плохо -->
<div id="notifications"></div>
<main>Content</main>

<script>
  // Новое уведомление сдвигает контент вниз
  const notification = document.createElement('div');
  notification.textContent = 'New notification!';
  document.getElementById('notifications').appendChild(notification);
</script>

<!-- ✅ Хорошо - уведомления в fixed/absolute позиции -->
<style>
  .notifications {
    position: fixed;
    top: 20px;
    right: 20px;
    z-index: 1000;
  }
</style>
<div class="notifications"></div>
<main>Content</main>

<!-- 4. Избегайте web fonts без fallback -->
<!-- ❌ Плохо - FOUT (Flash of Unstyled Text) -->
<style>
  @font-face {
    font-family: 'CustomFont';
    src: url('font.woff2');
  }
  body {
    font-family: 'CustomFont';
  }
</style>

<!-- ✅ Хорошо - font-display и fallback -->
<style>
  @font-face {
    font-family: 'CustomFont';
    src: url('font.woff2');
    font-display: swap; /* или optional */
  }
  body {
    font-family: 'CustomFont', Arial, sans-serif;
  }
</style>

<!-- ✅ Еще лучше - preload font -->
<link rel="preload" href="font.woff2" as="font" type="font/woff2" crossorigin>
```

```css
/* 5. Используйте CSS transform вместо изменения top/left */
/* ❌ Плохо - вызывает layout shift */
.element {
  animation: slideIn 0.3s;
}

@keyframes slideIn {
  from {
    top: -100px;
  }
  to {
    top: 0;
  }
}

/* ✅ Хорошо - не вызывает layout shift */
.element {
  animation: slideIn 0.3s;
}

@keyframes slideIn {
  from {
    transform: translateY(-100px);
  }
  to {
    transform: translateY(0);
  }
}

/* 6. Резервируйте место для ads/embeds */
.ad-container {
  min-height: 250px; /* Типичный размер рекламы */
  background: #f0f0f0; /* Placeholder */
}
```

**Другие важные Web Vitals:**

**FCP (First Contentful Paint):**
```javascript
// Время первой отрисовки контента
new PerformanceObserver((entryList) => {
  const entries = entryList.getEntries();
  const fcp = entries[0];
  console.log('FCP:', fcp.startTime);
}).observe({ type: 'paint', buffered: true });

// Good: < 1.8s
// Needs Improvement: 1.8s - 3s
// Poor: > 3s
```

**TTFB (Time to First Byte):**
```javascript
// Время до получения первого байта от сервера
const ttfb = performance.timing.responseStart - performance.timing.requestStart;
console.log('TTFB:', ttfb);

// Good: < 800ms
// Needs Improvement: 800ms - 1800ms
// Poor: > 1800ms

// Оптимизация TTFB:
// - CDN
// - Server-side caching (Redis)
// - Optimize database queries
// - Use HTTP/2 or HTTP/3
// - Reduce server processing time
```

**TBT (Total Blocking Time):**
```javascript
// Сумма времени блокировки main thread между FCP и TTI

// Оптимизация TBT:
// - Code splitting
// - Уменьшение JavaScript bundle
// - Defer non-critical JavaScript
// - Optimize third-party scripts
// - Use Web Workers
```

**Инструменты для измерения Web Vitals:**

```javascript
// 1. web-vitals библиотека
import { onCLS, onFID, onLCP, onINP, onFCP, onTTFB } from 'web-vitals';

function sendToAnalytics(metric) {
  // Отправка в Google Analytics, Sentry и т.д.
  console.log(metric.name, metric.value, metric.rating);
  
  // Example: Google Analytics
  gtag('event', metric.name, {
    value: Math.round(metric.name === 'CLS' ? metric.value * 1000 : metric.value),
    event_category: 'Web Vitals',
    event_label: metric.id,
    non_interaction: true,
  });
}

onCLS(sendToAnalytics);
onINP(sendToAnalytics);
onLCP(sendToAnalytics);
onFCP(sendToAnalytics);
onTTFB(sendToAnalytics);

// 2. Chrome DevTools
// - Performance panel
// - Lighthouse
// - Performance Insights

// 3. PageSpeed Insights
// https://pagespeed.web.dev/

// 4. Chrome User Experience Report (CrUX)
// Реальные данные от пользователей Chrome

// 5. Real User Monitoring (RUM)
// - Google Analytics
// - Sentry Performance
// - New Relic
// - Datadog
```

**React оптимизации для Web Vitals:**

```javascript
// 1. Code Splitting для LCP
const HeroSection = React.lazy(() => import('./HeroSection'));

function App() {
  return (
    <Suspense fallback={<HeroSkeleton />}>
      <HeroSection />
    </Suspense>
  );
}

// 2. Мемоизация для INP
const ExpensiveComponent = React.memo(({ data }) => {
  // Тяжелые вычисления
  const processed = useMemo(() => {
    return processData(data);
  }, [data]);
  
  return <div>{processed}</div>;
});

// 3. Избегание CLS с placeholder
function ImageWithPlaceholder({ src, alt, width, height }) {
  const [loaded, setLoaded] = useState(false);
  
  return (
    <div style={{ aspectRatio: `${width} / ${height}` }}>
      {!loaded && <div className="skeleton" />}
      <img
        src={src}
        alt={alt}
        width={width}
        height={height}
        onLoad={() => setLoaded(true)}
        style={{ opacity: loaded ? 1 : 0 }}
      />
    </div>
  );
}

// 4. Виртуализация для больших списков (INP)
import { FixedSizeList } from 'react-window';

function LargeList({ items }) {
  return (
    <FixedSizeList
      height={600}
      itemCount={items.length}
      itemSize={50}
      width="100%"
    >
      {({ index, style }) => (
        <div style={style}>{items[index].name}</div>
      )}
    </FixedSizeList>
  );
}

// 5. Debounce для поисковой строки (INP)
function SearchInput() {
  const [query, setQuery] = useState('');
  
  const debouncedSearch = useMemo(
    () => debounce((value) => {
      // Тяжелый поиск
      performSearch(value);
    }, 300),
    []
  );
  
  const handleChange = (e) => {
    const value = e.target.value;
    setQuery(value);
    debouncedSearch(value);
  };
  
  return <input value={query} onChange={handleChange} />;
}

// 6. Intersection Observer для lazy loading (LCP)
function LazyImage({ src, alt }) {
  const [imageSrc, setImageSrc] = useState(null);
  const imgRef = useRef();
  
  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setImageSrc(src);
          observer.disconnect();
        }
      },
      { rootMargin: '100px' }
    );
    
    if (imgRef.current) {
      observer.observe(imgRef.current);
    }
    
    return () => observer.disconnect();
  }, [src]);
  
  return (
    <img 
      ref={imgRef}
      src={imageSrc || 'placeholder.jpg'}
      alt={alt}
    />
  );
}
```

**Для чего:**
- SEO (Google использует Web Vitals в ранжировании)
- Улучшение пользовательского опыта
- Увеличение конверсии
- Снижение bounce rate

**Best Practices:**
```javascript
// Checklist Web Vitals:

// LCP оптимизация:
// ✅ Preload hero images
// ✅ Используйте CDN
// ✅ Оптимизируйте изображения (WebP, AVIF)
// ✅ Inline критичный CSS
// ✅ Минимизируйте render-blocking ресурсы

// INP оптимизация:
// ✅ Debounce/throttle частые события
// ✅ Code splitting
// ✅ Виртуализация больших списков
// ✅ Используйте Web Workers
// ✅ Passive event listeners

// CLS оптимизация:
// ✅ Указывайте размеры изображений
// ✅ Резервируйте место для ads/dynamic content
// ✅ font-display: swap для web fonts
// ✅ Избегайте вставки контента выше существующего
// ✅ Используйте transform для анимаций
```

---

### 11. RAIL модель производительности

**Как работает:**

RAIL - модель производительности, ориентированная на пользователя.

**R - Response (Отклик):**

Отклик на действия пользователя должен быть < 100ms.

```javascript
// ✅ Цель: < 100ms от input до visual response

// Event handlers должны завершаться быстро
button.addEventListener('click', (e) => {
  // ✅ Быстрая обратная связь
  showLoadingSpinner(); // < 100ms
  
  // Тяжелая работа асинхронно
  requestIdleCallback(() => {
    processHeavyTask();
  });
});

// Optimistic UI updates
async function likePost(postId) {
  // ✅ Обновляем UI сразу (< 100ms)
  updateLikeCountLocally(postId);
  
  try {
    // Отправляем запрос в фоне
    await api.likePost(postId);
  } catch (error) {
    // Откатываем при ошибке
    revertLikeCountLocally(postId);
    showError('Failed to like post');
  }
}

// Debounce для частых событий
const debouncedSearch = debounce((query) => {
  searchAPI(query);
}, 150);

input.addEventListener('input', (e) => {
  // ✅ UI обновляется сразу
  updateSearchPreview(e.target.value); // < 100ms
  
  // API вызов с задержкой
  debouncedSearch(e.target.value);
});
```

**A - Animation (Анимация):**

Анимации должны работать при 60 FPS (16ms на frame).

```javascript
// ✅ Цель: 60 FPS = 16ms на frame

// ❌ Плохо - JavaScript animation (может быть laggy)
let position = 0;
setInterval(() => {
  position += 1;
  element.style.left = position + 'px'; // Layout, Paint, Composite
}, 16);

// ✅ Хорошо - CSS animation (GPU accelerated)
element.style.animation = 'slide 0.3s ease-out';

@keyframes slide {
  from { transform: translateX(0); }
  to { transform: translateX(100px); }
}

// ✅ requestAnimationFrame для JS animations
function animate() {
  // Update в следующем frame
  position += velocity;
  element.style.transform = `translateX(${position}px)`;
  
  if (position < target) {
    requestAnimationFrame(animate);
  }
}
requestAnimationFrame(animate);

// ✅ Используйте только transform и opacity
// Эти свойства не вызывают reflow/repaint
.element {
  /* ✅ Дешевые свойства */
  transform: translateX(100px);
  opacity: 0.5;
  
  /* ❌ Дорогие свойства */
  left: 100px;
  width: 200px;
}

// ✅ will-change для оптимизации
.element {
  will-change: transform;
}

// ⚠️ Не злоупотребляйте will-change!
// Используйте только для анимируемых элементов

// Измерение FPS
let lastTime = performance.now();
let frames = 0;

function measureFPS() {
  frames++;
  const currentTime = performance.now();
  
  if (currentTime >= lastTime + 1000) {
    const fps = Math.round((frames * 1000) / (currentTime - lastTime));
    console.log('FPS:', fps);
    
    frames = 0;
    lastTime = currentTime;
  }
  
  requestAnimationFrame(measureFPS);
}
measureFPS();

// React - avoid animations during render
function AnimatedComponent() {
  // ❌ Плохо - анимация в render
  const [position, setPosition] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setPosition(p => p + 1); // Re-render на каждом frame!
    }, 16);
    return () => clearInterval(interval);
  }, []);
  
  return <div style={{ left: position }}></div>;
  
  // ✅ Хорошо - CSS animation
  return <div className="animated"></div>;
}
```

**I - Idle (Простой):**

Используйте idle time для неприоритетных задач.

```javascript
// ✅ Цель: выполнять неприоритетные задачи в idle time

// requestIdleCallback для отложенных задач
function sendAnalytics(data) {
  requestIdleCallback((deadline) => {
    // deadline.timeRemaining() - оставшееся время до конца idle
    if (deadline.timeRemaining() > 0) {
      fetch('/analytics', {
        method: 'POST',
        body: JSON.stringify(data)
      });
    } else {
      // Недостаточно времени, попробуем позже
      requestIdleCallback(() => sendAnalytics(data));
    }
  });
}

// Chunking для тяжелых задач
function processLargeDataset(data) {
  let index = 0;
  const chunkSize = 100;
  
  function processChunk(deadline) {
    // Обрабатываем пока есть время
    while (index < data.length && deadline.timeRemaining() > 0) {
      processItem(data[index]);
      index++;
    }
    
    if (index < data.length) {
      // Еще есть данные - продолжим в следующий idle
      requestIdleCallback(processChunk);
    } else {
      onComplete();
    }
  }
  
  requestIdleCallback(processChunk);
}

// React: разделение работы
function HeavyList({ items }) {
  const [visibleItems, setVisibleItems] = useState([]);
  
  useEffect(() => {
    // Рендерим по частям в idle time
    let index = 0;
    const batchSize = 10;
    
    function renderBatch() {
      requestIdleCallback((deadline) => {
        while (index < items.length && deadline.timeRemaining() > 1) {
          setVisibleItems(prev => [...prev, items[index]]);
          index++;
          
          if (index % batchSize === 0) {
            // Даем UI обновиться
            break;
          }
        }
        
        if (index < items.length) {
          renderBatch();
        }
      });
    }
    
    renderBatch();
  }, [items]);
  
  return (
    <ul>
      {visibleItems.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}

// Priorities для задач
const tasks = [
  { fn: sendAnalytics, priority: 'low' },
  { fn: preloadImages, priority: 'low' },
  { fn: updateUI, priority: 'high' }
];

tasks.forEach(task => {
  if (task.priority === 'high') {
    task.fn();
  } else {
    requestIdleCallback(() => task.fn());
  }
});
```

**L - Load (Загрузка):**

Страница должна быть интерактивной < 5s (на медленных устройствах).

```javascript
// ✅ Цель: Time to Interactive < 5s

// 1. Code Splitting
// ❌ Плохо - весь код загружается сразу
import Dashboard from './Dashboard';
import Settings from './Settings';
import Profile from './Profile';

// ✅ Хорошо - route-based splitting
const Dashboard = lazy(() => import('./Dashboard'));
const Settings = lazy(() => import('./Settings'));
const Profile = lazy(() => import('./Profile'));

function App() {
  return (
    <Router>
      <Suspense fallback={<Loading />}>
        <Routes>
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/settings" element={<Settings />} />
          <Route path="/profile" element={<Profile />} />
        </Routes>
      </Suspense>
    </Router>
  );
}

// 2. Defer non-critical JavaScript
<script src="critical.js"></script>
<script src="analytics.js" defer></script>
<script src="chat-widget.js" async></script>

// 3. Progressive Loading
function App() {
  const [loaded, setLoaded] = useState({
    critical: false,
    important: false,
    nice: false
  });
  
  useEffect(() => {
    // Phase 1: Critical content
    loadCriticalContent().then(() => {
      setLoaded(prev => ({ ...prev, critical: true }));
      
      // Phase 2: Important content
      requestIdleCallback(() => {
        loadImportantContent().then(() => {
          setLoaded(prev => ({ ...prev, important: true }));
          
          // Phase 3: Nice-to-have content
          requestIdleCallback(() => {
            loadNiceToHaveContent().then(() => {
              setLoaded(prev => ({ ...prev, nice: true }));
            });
          });
        });
      });
    });
  }, []);
  
  return (
    <>
      {loaded.critical && <CriticalContent />}
      {loaded.important && <ImportantContent />}
      {loaded.nice && <NiceToHaveContent />}
    </>
  );
}

// 4. Optimize bundle size
// package.json
{
  "scripts": {
    "analyze": "webpack-bundle-analyzer build/bundle-stats.json"
  }
}

// webpack.config.js
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendors: {
          test: /[\\/]node_modules[\\/]/,
          priority: -10
        },
        common: {
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true
        }
      }
    }
  }
};

// 5. Tree shaking
// ❌ Плохо
import _ from 'lodash';
_.debounce(fn, 300);

// ✅ Хорошо
import debounce from 'lodash/debounce';
debounce(fn, 300);

// 6. Lazy load images
<img 
  src="placeholder.jpg" 
  data-src="real-image.jpg"
  loading="lazy"
  alt="Image"
>

// Or с Intersection Observer
function LazyImage({ src, alt }) {
  const [imageSrc, setImageSrc] = useState('placeholder.jpg');
  const imgRef = useRef();
  
  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setImageSrc(src);
          observer.disconnect();
        }
      },
      { rootMargin: '50px' }
    );
    
    if (imgRef.current) {
      observer.observe(imgRef.current);
    }
    
    return () => observer.disconnect();
  }, [src]);
  
  return <img ref={imgRef} src={imageSrc} alt={alt} />;
}

// 7. Service Worker для caching
// sw.js
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open('v1').then((cache) => {
      return cache.addAll([
        '/',
        '/styles.css',
        '/script.js',
        '/logo.png'
      ]);
    })
  );
});

self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then((response) => {
      return response || fetch(event.request);
    })
  );
});
```

**Измерение RAIL метрик:**

```javascript
// Performance monitoring
class RAILMonitor {
  constructor() {
    this.metrics = {
      response: [],
      animation: [],
      idle: [],
      load: {}
    };
  }
  
  // R - Response
  measureResponse(eventName, callback) {
    const start = performance.now();
    
    callback();
    
    const duration = performance.now() - start;
    this.metrics.response.push({ eventName, duration });
    
    if (duration > 100) {
      console.warn(`Response too slow: ${eventName} took ${duration}ms`);
    }
  }
  
  // A - Animation
  startAnimationMonitoring() {
    let lastFrame = performance.now();
    let frames = [];
    
    const checkFrame = () => {
      const now = performance.now();
      const frameDuration = now - lastFrame;
      
      frames.push(frameDuration);
      
      // Check FPS every second
      if (frames.length >= 60) {
        const avgFrameTime = frames.reduce((a, b) => a + b) / frames.length;
        const fps = 1000 / avgFrameTime;
        
        if (fps < 55) {
          console.warn(`Low FPS detected: ${fps.toFixed(1)}`);
        }
        
        frames = [];
      }
      
      lastFrame = now;
      requestAnimationFrame(checkFrame);
    };
    
    requestAnimationFrame(checkFrame);
  }
  
  // L - Load
  measureLoad() {
    window.addEventListener('load', () => {
      const navTiming = performance.timing;
      
      this.metrics.load = {
        domContentLoaded: navTiming.domContentLoadedEventEnd - navTiming.navigationStart,
        loadComplete: navTiming.loadEventEnd - navTiming.navigationStart,
        domInteractive: navTiming.domInteractive - navTiming.navigationStart
      };
      
      console.log('Load metrics:', this.metrics.load);
      
      if (this.metrics.load.domInteractive > 5000) {
        console.warn('Time to Interactive > 5s');
      }
    });
  }
  
  report() {
    // Отправка метрик в аналитику
    navigator.sendBeacon('/analytics', JSON.stringify(this.metrics));
  }
}

// Использование
const monitor = new RAILMonitor();
monitor.startAnimationMonitoring();
monitor.measureLoad();

button.addEventListener('click', () => {
  monitor.measureResponse('button-click', () => {
    handleClick();
  });
});

window.addEventListener('beforeunload', () => {
  monitor.report();
});
```

**React оптимизации по RAIL:**

```javascript
// R - Response
function QuickResponseButton() {
  const handleClick = () => {
    // ✅ Сразу показываем feedback
    setLoading(true);
    
    // Тяжелая работа отложенно
    startTransition(() => {
      processHeavyTask();
    });
  };
  
  return <button onClick={handleClick}>Click</button>;
}

// A - Animation
function SmoothAnimation() {
  // ✅ Используем CSS вместо JS
  return (
    <motion.div
      initial={{ opacity: 0 }}
      animate={{ opacity: 1 }}
      transition={{ duration: 0.3 }}
    >
      Content
    </motion.div>
  );
}

// I - Idle
function IdleTaskComponent() {
  useEffect(() => {
    // ✅ Неприоритетные задачи в idle
    const handle = requestIdleCallback(() => {
      preloadImages();
      warmupCache();
    });
    
    return () => cancelIdleCallback(handle);
  }, []);
}

// L - Load
function LazyApp() {
  // ✅ Code splitting
  const HeavyComponent = lazy(() => import('./HeavyComponent'));
  
  return (
    <Suspense fallback={<Skeleton />}>
      <HeavyComponent />
    </Suspense>
  );
}
```

**Для чего:**
- Фреймворк для оптимизации производительности
- Понимание приоритетов оптимизации
- Измеримые цели для производительности

**Best Practices:**
```javascript
// RAIL Checklist:

// Response (< 100ms)
// ✅ Optimistic UI updates
// ✅ Debounce частые события
// ✅ Показывайте loading состояния сразу

// Animation (60 FPS)
// ✅ Используйте transform и opacity
// ✅ CSS animations вместо JS
// ✅ requestAnimationFrame для JS animations
// ✅ will-change для оптимизации

// Idle
// ✅ requestIdleCallback для неприоритетных задач
// ✅ Chunking для тяжелых вычислений
// ✅ Prefetch/preload в idle time

// Load (< 5s)
// ✅ Code splitting
// ✅ Tree shaking
// ✅ Lazy loading
// ✅ Service Worker caching
// ✅ CDN
// ✅ Optimize images
```

---

Это третья часть. Хотите, чтобы я продолжил с Browser APIs, CSS, Build Tools и остальными темами?


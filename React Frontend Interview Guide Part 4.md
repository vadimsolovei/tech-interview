# Подготовка к собеседованию: Strong Middle Frontend React Developer (Часть 4)

## Browser APIs

---

### 12. Intersection Observer API

**Как работает:**

Intersection Observer API позволяет асинхронно отслеживать пересечение элемента с viewport или другим элементом.

**Основное использование:**

```javascript
// Создание observer
const observer = new IntersectionObserver(
  (entries, observer) => {
    entries.forEach(entry => {
      // entry.isIntersecting - элемент виден
      // entry.intersectionRatio - процент видимости (0-1)
      // entry.target - наблюдаемый элемент
      // entry.boundingClientRect - размеры элемента
      // entry.intersectionRect - видимая часть
      // entry.rootBounds - размеры root элемента
      
      if (entry.isIntersecting) {
        console.log('Element is visible:', entry.target);
        
        // Можно прекратить наблюдение
        observer.unobserve(entry.target);
      }
    });
  },
  {
    root: null, // null = viewport, или конкретный элемент
    rootMargin: '0px', // отступ от границ root
    threshold: 0 // 0 = хоть 1 пиксель, 1 = полностью виден, [0, 0.5, 1] = несколько порогов
  }
);

// Начать наблюдение
const element = document.querySelector('.target');
observer.observe(element);

// Прекратить наблюдение
observer.unobserve(element);

// Прекратить все наблюдения
observer.disconnect();
```

**1. Lazy Loading изображений:**

```javascript
// HTML
<img 
  class="lazy" 
  data-src="image.jpg" 
  src="placeholder.jpg" 
  alt="Image"
>

// JavaScript
const lazyImages = document.querySelectorAll('img.lazy');

const imageObserver = new IntersectionObserver(
  (entries, observer) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        const img = entry.target;
        
        // Загружаем реальное изображение
        img.src = img.dataset.src;
        
        img.onload = () => {
          img.classList.remove('lazy');
          img.classList.add('loaded');
        };
        
        // Прекращаем наблюдение
        observer.unobserve(img);
      }
    });
  },
  {
    rootMargin: '50px' // Начинаем загрузку за 50px до появления в viewport
  }
);

lazyImages.forEach(img => imageObserver.observe(img));

// React версия
function LazyImage({ src, alt, placeholder = 'placeholder.jpg' }) {
  const [imageSrc, setImageSrc] = useState(placeholder);
  const [isLoaded, setIsLoaded] = useState(false);
  const imgRef = useRef(null);
  
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
  
  return (
    <img
      ref={imgRef}
      src={imageSrc}
      alt={alt}
      onLoad={() => setIsLoaded(true)}
      className={isLoaded ? 'loaded' : 'loading'}
    />
  );
}
```

**2. Infinite Scroll:**

```javascript
function InfiniteScroll({ items, loadMore, hasMore }) {
  const [displayedItems, setDisplayedItems] = useState(items);
  const [loading, setLoading] = useState(false);
  const loaderRef = useRef(null);
  
  useEffect(() => {
    const observer = new IntersectionObserver(
      async ([entry]) => {
        if (entry.isIntersecting && hasMore && !loading) {
          setLoading(true);
          
          try {
            const newItems = await loadMore();
            setDisplayedItems(prev => [...prev, ...newItems]);
          } finally {
            setLoading(false);
          }
        }
      },
      {
        rootMargin: '100px' // Загружаем за 100px до конца
      }
    );
    
    if (loaderRef.current) {
      observer.observe(loaderRef.current);
    }
    
    return () => observer.disconnect();
  }, [hasMore, loading, loadMore]);
  
  return (
    <div>
      {displayedItems.map(item => (
        <div key={item.id}>{item.name}</div>
      ))}
      
      {hasMore && (
        <div ref={loaderRef}>
          {loading ? <Spinner /> : <div>Load more...</div>}
        </div>
      )}
    </div>
  );
}
```

**3. Отслеживание видимости элементов (Analytics):**

```javascript
// Отслеживание когда элемент стал виден
function trackVisibility(element, callback) {
  const observer = new IntersectionObserver(
    ([entry]) => {
      if (entry.isIntersecting) {
        const visibleTime = Date.now();
        
        // Отслеживаем сколько времени элемент виден
        const exitObserver = new IntersectionObserver(
          ([exitEntry]) => {
            if (!exitEntry.isIntersecting) {
              const duration = Date.now() - visibleTime;
              callback({
                element: entry.target,
                duration,
                visibleRatio: entry.intersectionRatio
              });
              exitObserver.disconnect();
            }
          }
        );
        
        exitObserver.observe(element);
        observer.unobserve(element);
      }
    },
    {
      threshold: 0.5 // Считаем видимым если > 50%
    }
  );
  
  observer.observe(element);
  return observer;
}

// Использование
trackVisibility(
  document.querySelector('.ad-banner'),
  ({ element, duration, visibleRatio }) => {
    // Отправка в аналитику
    gtag('event', 'ad_view', {
      ad_id: element.dataset.adId,
      view_duration: duration,
      visible_ratio: visibleRatio
    });
  }
);

// React hook
function useViewTracking(elementRef, onView) {
  useEffect(() => {
    const element = elementRef.current;
    if (!element) return;
    
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          onView({
            element: entry.target,
            ratio: entry.intersectionRatio
          });
        }
      },
      { threshold: 0.5 }
    );
    
    observer.observe(element);
    return () => observer.disconnect();
  }, [elementRef, onView]);
}

function AdComponent({ adId }) {
  const adRef = useRef(null);
  
  useViewTracking(adRef, ({ ratio }) => {
    console.log(`Ad ${adId} viewed with ratio ${ratio}`);
  });
  
  return <div ref={adRef} data-ad-id={adId}>Ad Content</div>;
}
```

**4. Анимация при появлении (Scroll Animations):**

```javascript
// CSS
.animate-on-scroll {
  opacity: 0;
  transform: translateY(50px);
  transition: opacity 0.6s, transform 0.6s;
}

.animate-on-scroll.visible {
  opacity: 1;
  transform: translateY(0);
}

// JavaScript
const animateElements = document.querySelectorAll('.animate-on-scroll');

const animationObserver = new IntersectionObserver(
  (entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        entry.target.classList.add('visible');
        // Если анимация одноразовая:
        animationObserver.unobserve(entry.target);
      }
    });
  },
  {
    threshold: 0.1
  }
);

animateElements.forEach(el => animationObserver.observe(el));

// React версия с Framer Motion
function AnimateOnScroll({ children }) {
  const [isVisible, setIsVisible] = useState(false);
  const ref = useRef(null);
  
  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setIsVisible(true);
          observer.disconnect();
        }
      },
      { threshold: 0.1 }
    );
    
    if (ref.current) {
      observer.observe(ref.current);
    }
    
    return () => observer.disconnect();
  }, []);
  
  return (
    <motion.div
      ref={ref}
      initial={{ opacity: 0, y: 50 }}
      animate={isVisible ? { opacity: 1, y: 0 } : {}}
      transition={{ duration: 0.6 }}
    >
      {children}
    </motion.div>
  );
}
```

**5. Sticky Header при скролле:**

```javascript
function StickyHeader() {
  const [isSticky, setIsSticky] = useState(false);
  const headerRef = useRef(null);
  const sentinelRef = useRef(null);
  
  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        // Когда sentinel НЕ виден, header должен быть sticky
        setIsSticky(!entry.isIntersecting);
      },
      {
        threshold: 0,
        rootMargin: '-1px 0px 0px 0px' // Небольшой offset
      }
    );
    
    if (sentinelRef.current) {
      observer.observe(sentinelRef.current);
    }
    
    return () => observer.disconnect();
  }, []);
  
  return (
    <>
      {/* Sentinel element */}
      <div ref={sentinelRef} style={{ height: '1px' }} />
      
      <header
        ref={headerRef}
        className={isSticky ? 'sticky' : ''}
        style={{
          position: isSticky ? 'fixed' : 'relative',
          top: 0,
          width: '100%',
          transition: 'all 0.3s'
        }}
      >
        Header Content
      </header>
    </>
  );
}
```

**6. Паузе видео когда не в viewport:**

```javascript
function AutoPauseVideo({ src }) {
  const videoRef = useRef(null);
  
  useEffect(() => {
    const video = videoRef.current;
    if (!video) return;
    
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          video.play();
        } else {
          video.pause();
        }
      },
      {
        threshold: 0.5 // Играем если > 50% видно
      }
    );
    
    observer.observe(video);
    return () => observer.disconnect();
  }, []);
  
  return (
    <video
      ref={videoRef}
      src={src}
      loop
      muted
      playsInline
    />
  );
}
```

**Для чего:**
- Lazy loading изображений и контента
- Infinite scroll
- Анимации при скролле
- Tracking видимости для аналитики
- Оптимизация производительности (загрузка только видимого)

**Плюсы:**
- Асинхронный (не блокирует main thread)
- Производительный (лучше чем scroll events)
- Простой API
- Поддержка threshold для точного контроля

**Минусы:**
- Не поддерживается в старых браузерах (нужен polyfill)
- Не подходит для точного отслеживания позиции scroll

**Аналоги:**
- Scroll events (устаревший, менее производительный подход)
- getBoundingClientRect + scroll events
- Библиотеки: react-intersection-observer, react-lazyload

---

### 13. Mutation Observer API

**Как работает:**

Mutation Observer отслеживает изменения в DOM дереве.

**Основное использование:**

```javascript
// Создание observer
const observer = new MutationObserver((mutations, observer) => {
  mutations.forEach(mutation => {
    console.log('Type:', mutation.type);
    console.log('Target:', mutation.target);
    
    switch (mutation.type) {
      case 'childList':
        console.log('Added nodes:', mutation.addedNodes);
        console.log('Removed nodes:', mutation.removedNodes);
        break;
        
      case 'attributes':
        console.log('Attribute:', mutation.attributeName);
        console.log('Old value:', mutation.oldValue);
        break;
        
      case 'characterData':
        console.log('Old data:', mutation.oldValue);
        break;
    }
  });
});

// Начать наблюдение
observer.observe(targetElement, {
  childList: true,        // Наблюдать за добавлением/удалением детей
  attributes: true,       // Наблюдать за изменением атрибутов
  characterData: true,    // Наблюдать за изменением текста
  subtree: true,          // Наблюдать за всеми потомками
  attributeOldValue: true,// Сохранять старое значение атрибута
  characterDataOldValue: true, // Сохранять старый текст
  attributeFilter: ['class', 'style'] // Наблюдать только за этими атрибутами
});

// Прекратить наблюдение
observer.disconnect();

// Получить накопленные, но необработанные мутации
const mutations = observer.takeRecords();
```

**1. Отслеживание изменений в DOM (для библиотек):**

```javascript
// Пример: автоматическая инициализация компонентов
class ComponentRegistry {
  constructor() {
    this.components = new Map();
    this.observer = new MutationObserver(this.handleMutations.bind(this));
  }
  
  start() {
    this.observer.observe(document.body, {
      childList: true,
      subtree: true
    });
    
    // Инициализируем существующие компоненты
    this.initializeComponents(document.body);
  }
  
  handleMutations(mutations) {
    mutations.forEach(mutation => {
      // Обрабатываем добавленные узлы
      mutation.addedNodes.forEach(node => {
        if (node.nodeType === Node.ELEMENT_NODE) {
          this.initializeComponents(node);
        }
      });
      
      // Обрабатываем удаленные узлы
      mutation.removedNodes.forEach(node => {
        if (node.nodeType === Node.ELEMENT_NODE) {
          this.cleanupComponents(node);
        }
      });
    });
  }
  
  initializeComponents(root) {
    const elements = root.querySelectorAll('[data-component]');
    
    elements.forEach(element => {
      const componentName = element.dataset.component;
      const ComponentClass = this.getComponent(componentName);
      
      if (ComponentClass && !this.components.has(element)) {
        const instance = new ComponentClass(element);
        this.components.set(element, instance);
      }
    });
  }
  
  cleanupComponents(root) {
    const elements = root.querySelectorAll('[data-component]');
    
    elements.forEach(element => {
      const instance = this.components.get(element);
      if (instance && instance.destroy) {
        instance.destroy();
      }
      this.components.delete(element);
    });
  }
  
  stop() {
    this.observer.disconnect();
  }
}

// Использование
const registry = new ComponentRegistry();
registry.start();
```

**2. Отслеживание изменений class для анимаций:**

```javascript
function AnimationMonitor(element, callback) {
  const observer = new MutationObserver(mutations => {
    mutations.forEach(mutation => {
      if (mutation.type === 'attributes' && mutation.attributeName === 'class') {
        const oldClasses = mutation.oldValue ? mutation.oldValue.split(' ') : [];
        const newClasses = mutation.target.className.split(' ');
        
        const added = newClasses.filter(c => !oldClasses.includes(c));
        const removed = oldClasses.filter(c => !newClasses.includes(c));
        
        callback({ added, removed, target: mutation.target });
      }
    });
  });
  
  observer.observe(element, {
    attributes: true,
    attributeOldValue: true,
    attributeFilter: ['class']
  });
  
  return observer;
}

// Использование
const monitor = AnimationMonitor(
  document.querySelector('.animated'),
  ({ added, removed, target }) => {
    console.log('Classes added:', added);
    console.log('Classes removed:', removed);
    
    // Запускаем анимацию при добавлении класса
    if (added.includes('show')) {
      target.addEventListener('animationend', () => {
        console.log('Animation completed');
      }, { once: true });
    }
  }
);
```

**3. Отслеживание добавления элементов для validation:**

```javascript
function FormValidator() {
  const observer = new MutationObserver(mutations => {
    mutations.forEach(mutation => {
      mutation.addedNodes.forEach(node => {
        if (node.nodeType === Node.ELEMENT_NODE) {
          // Ищем input элементы
          const inputs = node.matches('input, textarea, select')
            ? [node]
            : node.querySelectorAll('input, textarea, select');
          
          inputs.forEach(input => {
            // Добавляем validation
            attachValidation(input);
          });
        }
      });
    });
  });
  
  observer.observe(document.body, {
    childList: true,
    subtree: true
  });
  
  return observer;
}

function attachValidation(input) {
  if (input.dataset.validated) return;
  input.dataset.validated = 'true';
  
  input.addEventListener('blur', () => {
    validateField(input);
  });
}

function validateField(input) {
  const value = input.value;
  const errorElement = input.nextElementSibling;
  
  if (input.required && !value) {
    input.classList.add('error');
    if (errorElement?.classList.contains('error-message')) {
      errorElement.textContent = 'This field is required';
    }
  } else {
    input.classList.remove('error');
    if (errorElement?.classList.contains('error-message')) {
      errorElement.textContent = '';
    }
  }
}

// Запуск
const validator = FormValidator();
```

**4. React: отслеживание внешних изменений:**

```javascript
// Custom hook для отслеживания DOM мутаций
function useMutationObserver(
  ref,
  callback,
  options = {}
) {
  useEffect(() => {
    if (!ref.current) return;
    
    const observer = new MutationObserver(callback);
    
    observer.observe(ref.current, {
      childList: true,
      subtree: true,
      attributes: true,
      characterData: true,
      ...options
    });
    
    return () => observer.disconnect();
  }, [ref, callback, options]);
}

// Использование: отслеживание когда сторонняя библиотека меняет DOM
function ThirdPartyWidget() {
  const widgetRef = useRef(null);
  
  useMutationObserver(
    widgetRef,
    (mutations) => {
      mutations.forEach(mutation => {
        if (mutation.type === 'childList') {
          console.log('Widget DOM changed');
          // Можем адаптировать стили React компонента
          adjustLayout();
        }
      });
    },
    {
      childList: true,
      subtree: true
    }
  );
  
  useEffect(() => {
    // Инициализация сторонней библиотеки
    if (widgetRef.current) {
      window.ThirdPartyLib.init(widgetRef.current);
    }
  }, []);
  
  return <div ref={widgetRef} className="third-party-widget" />;
}
```

**5. Отслеживание изменений в contenteditable:**

```javascript
function ContentEditableTracker({ initialContent, onChange }) {
  const editorRef = useRef(null);
  
  useEffect(() => {
    const editor = editorRef.current;
    if (!editor) return;
    
    const observer = new MutationObserver((mutations) => {
      // Объединяем все изменения и вызываем onChange один раз
      const content = editor.innerHTML;
      onChange(content);
    });
    
    observer.observe(editor, {
      characterData: true,
      childList: true,
      subtree: true
    });
    
    return () => observer.disconnect();
  }, [onChange]);
  
  return (
    <div
      ref={editorRef}
      contentEditable
      dangerouslySetInnerHTML={{ __html: initialContent }}
    />
  );
}
```

**6. Обнаружение добавления скриптов (Security):**

```javascript
// Защита от XSS через мониторинг добавления скриптов
function ScriptMonitor() {
  const observer = new MutationObserver(mutations => {
    mutations.forEach(mutation => {
      mutation.addedNodes.forEach(node => {
        if (node.nodeName === 'SCRIPT') {
          console.warn('Script tag added:', node.src || 'inline');
          
          // Проверяем whitelist
          const allowedDomains = ['cdn.example.com', 'trusted-cdn.com'];
          const src = node.src;
          
          if (src) {
            const url = new URL(src);
            if (!allowedDomains.some(domain => url.hostname.includes(domain))) {
              console.error('Unauthorized script blocked:', src);
              node.remove();
              
              // Отправка в security monitoring
              reportSecurityIncident({
                type: 'unauthorized_script',
                src: src,
                timestamp: Date.now()
              });
            }
          }
        }
      });
    });
  });
  
  observer.observe(document.documentElement, {
    childList: true,
    subtree: true
  });
  
  return observer;
}

// Запуск мониторинга
if (process.env.NODE_ENV === 'production') {
  ScriptMonitor();
}
```

**Для чего:**
- Отслеживание изменений DOM для интеграции с legacy кодом
- Автоматическая инициализация компонентов
- Мониторинг изменений от сторонних библиотек
- Security monitoring (обнаружение XSS)
- Синхронизация с внешними изменениями

**Плюсы:**
- Асинхронный (не блокирует)
- Более производительный чем DOM events
- Детальная информация о изменениях
- Может наблюдать за subtree

**Минусы:**
- Может быть сложно обрабатывать большое количество мутаций
- Не все изменения генерируют события
- Нужно быть осторожным с бесконечными циклами (observer меняет DOM → новые мутации)

**Best Practices:**

```javascript
// ✅ Отписываться при unmount
useEffect(() => {
  const observer = new MutationObserver(callback);
  observer.observe(element, options);
  
  return () => observer.disconnect();
}, []);

// ✅ Debounce для частых изменений
const debouncedCallback = debounce((mutations) => {
  handleMutations(mutations);
}, 100);

const observer = new MutationObserver(debouncedCallback);

// ✅ Фильтровать нужные мутации
const observer = new MutationObserver(mutations => {
  mutations.forEach(mutation => {
    // Игнорируем свои собственные изменения
    if (mutation.target.dataset.ignore) return;
    
    // Обрабатываем только нужные
    if (mutation.type === 'childList') {
      handleChildListMutation(mutation);
    }
  });
});

// ❌ Избегайте бесконечных циклов
const observer = new MutationObserver(mutations => {
  mutations.forEach(mutation => {
    // ❌ Плохо - изменяем тот же элемент
    mutation.target.classList.add('processed'); // Вызовет новую мутацию!
  });
});

// ✅ Хорошо - временно отключаем observer
const observer = new MutationObserver(mutations => {
  observer.disconnect(); // Отключаем
  
  mutations.forEach(mutation => {
    mutation.target.classList.add('processed'); // Безопасно
  });
  
  observer.observe(element, options); // Включаем обратно
});
```

---

### 14. Resize Observer API

**Как работает:**

Resize Observer отслеживает изменения размеров элементов.

**Основное использование:**

```javascript
const observer = new ResizeObserver((entries) => {
  entries.forEach(entry => {
    console.log('Element:', entry.target);
    console.log('Content rect:', entry.contentRect);
    console.log('Border box size:', entry.borderBoxSize);
    console.log('Content box size:', entry.contentBoxSize);
    
    const width = entry.contentRect.width;
    const height = entry.contentRect.height;
    
    console.log(`Size: ${width}x${height}`);
  });
});

// Начать наблюдение
observer.observe(element);

// Прекратить наблюдение
observer.unobserve(element);
observer.disconnect();
```

**1. Responsive компоненты без media queries:**

```javascript
function ResponsiveCard() {
  const cardRef = useRef(null);
  const [size, setSize] = useState('large');
  
  useEffect(() => {
    const observer = new ResizeObserver(([entry]) => {
      const width = entry.contentRect.width;
      
      if (width < 300) {
        setSize('small');
      } else if (width < 600) {
        setSize('medium');
      } else {
        setSize('large');
      }
    });
    
    if (cardRef.current) {
      observer.observe(cardRef.current);
    }
    
    return () => observer.disconnect();
  }, []);
  
  return (
    <div ref={cardRef} className={`card card--${size}`}>
      {size === 'small' && <SmallLayout />}
      {size === 'medium' && <MediumLayout />}
      {size === 'large' && <LargeLayout />}
    </div>
  );
}

// Container Queries в CSS (современный подход)
.card {
  container-type: inline-size;
}

@container (max-width: 300px) {
  .card__content {
    flex-direction: column;
  }
}
```

**2. Автоматический responsive текст:**

```javascript
function AutoScalingText({ children }) {
  const containerRef = useRef(null);
  const textRef = useRef(null);
  
  useEffect(() => {
    const container = containerRef.current;
    const text = textRef.current;
    if (!container || !text) return;
    
    const observer = new ResizeObserver(() => {
      const containerWidth = container.offsetWidth;
      const textWidth = text.scrollWidth;
      
      // Уменьшаем размер шрифта если текст не помещается
      if (textWidth > containerWidth) {
        const scale = containerWidth / textWidth;
        text.style.transform = `scale(${scale})`;
        text.style.transformOrigin = 'left center';
      } else {
        text.style.transform = 'scale(1)';
      }
    });
    
    observer.observe(container);
    
    return () => observer.disconnect();
  }, [children]);
  
  return (
    <div ref={containerRef} style={{ overflow: 'hidden' }}>
      <div ref={textRef} style={{ whiteSpace: 'nowrap' }}>
        {children}
      </div>
    </div>
  );
}
```

**3. Canvas resize с правильным масштабированием:**

```javascript
function ResponsiveCanvas() {
  const canvasRef = useRef(null);
  const containerRef = useRef(null);
  
  useEffect(() => {
    const canvas = canvasRef.current;
    const container = containerRef.current;
    if (!canvas || !container) return;
    
    const ctx = canvas.getContext('2d');
    
    const observer = new ResizeObserver(([entry]) => {
      const { width, height } = entry.contentRect;
      
      // Учитываем device pixel ratio для четкости
      const dpr = window.devicePixelRatio || 1;
      
      canvas.width = width * dpr;
      canvas.height = height * dpr;
      
      canvas.style.width = `${width}px`;
      canvas.style.height = `${height}px`;
      
      ctx.scale(dpr, dpr);
      
      // Перерисовываем
      draw(ctx, width, height);
    });
    
    observer.observe(container);
    
    return () => observer.disconnect();
  }, []);
  
  return (
    <div ref={containerRef} style={{ width: '100%', height: '400px' }}>
      <canvas ref={canvasRef} />
    </div>
  );
}
```

**4. Виртуализация с динамическими высотами:**

```javascript
function DynamicVirtualList({ items }) {
  const [heights, setHeights] = useState({});
  const itemRefs = useRef({});
  
  useEffect(() => {
    const observers = new Map();
    
    Object.entries(itemRefs.current).forEach(([id, element]) => {
      if (!element) return;
      
      const observer = new ResizeObserver(([entry]) => {
        const height = entry.contentRect.height;
        
        setHeights(prev => ({
          ...prev,
          [id]: height
        }));
      });
      
      observer.observe(element);
      observers.set(id, observer);
    });
    
    return () => {
      observers.forEach(observer => observer.disconnect());
    };
  }, [items]);
  
  // Вычисляем позиции на основе реальных высот
  const positions = useMemo(() => {
    let offset = 0;
    return items.map(item => {
      const height = heights[item.id] || 100; // default height
      const position = { offset, height };
      offset += height;
      return position;
    });
  }, [items, heights]);
  
  return (
    <div style={{ height: '600px', overflow: 'auto' }}>
      {items.map((item, index) => (
        <div
          key={item.id}
          ref={el => itemRefs.current[item.id] = el}
          style={{
            position: 'absolute',
            top: positions[index].offset,
            width: '100%'
          }}
        >
          <ItemContent item={item} />
        </div>
      ))}
    </div>
  );
}
```

**5. Отслеживание breakpoints:**

```javascript
// Custom hook для breakpoints
function useBreakpoint() {
  const [breakpoint, setBreakpoint] = useState('xl');
  const elementRef = useRef(null);
  
  useEffect(() => {
    if (!elementRef.current) {
      elementRef.current = document.body;
    }
    
    const observer = new ResizeObserver(([entry]) => {
      const width = entry.contentRect.width;
      
      if (width < 640) setBreakpoint('sm');
      else if (width < 768) setBreakpoint('md');
      else if (width < 1024) setBreakpoint('lg');
      else if (width < 1280) setBreakpoint('xl');
      else setBreakpoint('2xl');
    });
    
    observer.observe(elementRef.current);
    
    return () => observer.disconnect();
  }, []);
  
  return breakpoint;
}

// Использование
function ResponsiveComponent() {
  const breakpoint = useBreakpoint();
  
  return (
    <div>
      <p>Current breakpoint: {breakpoint}</p>
      
      {breakpoint === 'sm' && <MobileView />}
      {breakpoint === 'md' && <TabletView />}
      {['lg', 'xl', '2xl'].includes(breakpoint) && <DesktopView />}
    </div>
  );
}
```

**6. Sticky sidebar с правильной высотой:**

```javascript
function StickySidebar({ children }) {
  const sidebarRef = useRef(null);
  const [maxHeight, setMaxHeight] = useState('auto');
  
  useEffect(() => {
    const observer = new ResizeObserver(() => {
      if (!sidebarRef.current) return;
      
      const viewportHeight = window.innerHeight;
      const rect = sidebarRef.current.getBoundingClientRect();
      const availableHeight = viewportHeight - rect.top - 20; // 20px padding
      
      setMaxHeight(`${availableHeight}px`);
    });
    
    observer.observe(document.body);
    
    // Также слушаем scroll для обновления позиции
    const handleScroll = () => {
      if (!sidebarRef.current) return;
      
      const viewportHeight = window.innerHeight;
      const rect = sidebarRef.current.getBoundingClientRect();
      const availableHeight = viewportHeight - Math.max(rect.top, 0) - 20;
      
      setMaxHeight(`${availableHeight}px`);
    };
    
    window.addEventListener('scroll', handleScroll, { passive: true });
    
    return () => {
      observer.disconnect();
      window.removeEventListener('scroll', handleScroll);
    };
  }, []);
  
  return (
    <aside
      ref={sidebarRef}
      style={{
        position: 'sticky',
        top: '20px',
        maxHeight,
        overflowY: 'auto'
      }}
    >
      {children}
    </aside>
  );
}
```

**Для чего:**
- Responsive компоненты без media queries
- Адаптация под размер контейнера (не viewport)
- Canvas и WebGL приложения
- Виртуализация с динамическими размерами
- Container Queries полифиллы

**Плюсы:**
- Работает на уровне элементов (не только viewport)
- Асинхронный, не блокирует
- Более производительный чем window resize events
- Поддержка в современных браузерах

**Минусы:**
- Нет в старых браузерах (нужен polyfill)
- Может вызываться часто при анимациях

**Best Practices:**

```javascript
// ✅ Debounce для частых изменений
function useResizeObserver(ref, callback) {
  useEffect(() => {
    if (!ref.current) return;
    
    const debouncedCallback = debounce(callback, 100);
    
    const observer = new ResizeObserver(entries => {
      debouncedCallback(entries);
    });
    
    observer.observe(ref.current);
    
    return () => observer.disconnect();
  }, [ref, callback]);
}

// ✅ Используйте requestAnimationFrame для плавности
const observer = new ResizeObserver(entries => {
  requestAnimationFrame(() => {
    entries.forEach(entry => {
      updateLayout(entry);
    });
  });
});

// ✅ Избегайте изменения размера в callback (бесконечный цикл)
const observer = new ResizeObserver(([entry]) => {
  // ❌ Плохо
  entry.target.style.width = '100px'; // Вызовет новый resize!
  
  // ✅ Хорошо - используйте флаг
  if (!entry.target.dataset.resizing) {
    entry.target.dataset.resizing = 'true';
    entry.target.style.width = '100px';
    
    setTimeout(() => {
      entry.target.dataset.resizing = '';
    }, 0);
  }
});
```

---

### 15. Web Workers

**Как работает:**

Web Workers позволяют выполнять JavaScript в фоновом потоке, не блокируя main thread.

**Типы Workers:**
- Dedicated Workers (для одной страницы)
- Shared Workers (разделяются между вкладками)
- Service Workers (для PWA и кэширования)

**Dedicated Worker:**

```javascript
// main.js
const worker = new Worker('worker.js');

// Отправка данных в worker
worker.postMessage({ type: 'start', data: [1, 2, 3, 4, 5] });

// Получение данных от worker
worker.addEventListener('message', (event) => {
  console.log('Result from worker:', event.data);
});

// Обработка ошибок
worker.addEventListener('error', (error) => {
  console.error('Worker error:', error.message, error.filename, error.lineno);
});

// Завершение worker
worker.terminate();

// worker.js
self.addEventListener('message', (event) => {
  const { type, data } = event.data;
  
  if (type === 'start') {
    // Тяжелые вычисления
    const result = heavyComputation(data);
    
    // Отправка результата обратно
    self.postMessage({ type: 'result', result });
  }
});

function heavyComputation(data) {
  // Сложные вычисления, которые заняли бы main thread
  let sum = 0;
  for (let i = 0; i < 1000000000; i++) {
    sum += data[i % data.length];
  }
  return sum;
}
```

**1. Обработка больших данных:**

```javascript
// main.js
function processLargeDataset(data) {
  return new Promise((resolve, reject) => {
    const worker = new Worker('data-processor.js');
    
    worker.postMessage({ data });
    
    worker.addEventListener('message', (event) => {
      resolve(event.data);
      worker.terminate();
    });
    
    worker.addEventListener('error', (error) => {
      reject(error);
      worker.terminate();
    });
    
    // Timeout
    setTimeout(() => {
      reject(new Error('Worker timeout'));
      worker.terminate();
    }, 30000);
  });
}

// Использование
const largeData = Array.from({ length: 1000000 }, (_, i) => i);

processLargeDataset(largeData)
  .then(result => {
    console.log('Processed:', result);
  })
  .catch(error => {
    console.error('Error:', error);
  });

// data-processor.js
self.addEventListener('message', (event) => {
  const { data } = event.data;
  
  // Сложная обработка
  const processed = data.map(item => {
    // Тяжелые вычисления
    return Math.sqrt(item) * Math.PI;
  });
  
  // Прогресс можно отправлять периодически
  const batchSize = 10000;
  for (let i = 0; i < processed.length; i += batchSize) {
    if (i % (batchSize * 10) === 0) {
      self.postMessage({
        type: 'progress',
        progress: (i / processed.length) * 100
      });
    }
  }
  
  self.postMessage({
    type: 'complete',
    result: processed
  });
});
```

**2. React hook для Web Workers:**

```javascript
// useWebWorker.js
import { useEffect, useRef, useState, useCallback } from 'react';

function useWebWorker(workerFunction) {
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(false);
  const workerRef = useRef(null);
  
  useEffect(() => {
    // Создаем worker из функции
    const blob = new Blob(
      [`(${workerFunction.toString()})()`],
      { type: 'application/javascript' }
    );
    const workerUrl = URL.createObjectURL(blob);
    const worker = new Worker(workerUrl);
    
    workerRef.current = worker;
    
    worker.addEventListener('message', (event) => {
      setData(event.data);
      setLoading(false);
    });
    
    worker.addEventListener('error', (err) => {
      setError(err);
      setLoading(false);
    });
    
    return () => {
      worker.terminate();
      URL.revokeObjectURL(workerUrl);
    };
  }, [workerFunction]);
  
  const postMessage = useCallback((message) => {
    setLoading(true);
    setError(null);
    workerRef.current?.postMessage(message);
  }, []);
  
  return { data, error, loading, postMessage };
}

// Использование
function App() {
  const workerFunction = () => {
    self.addEventListener('message', (e) => {
      const { numbers } = e.data;
      
      // Тяжелые вычисления
      const result = numbers.reduce((sum, n) => sum + Math.sqrt(n), 0);
      
      self.postMessage({ result });
    });
  };
  
  const { data, loading, postMessage } = useWebWorker(workerFunction);
  
  const handleCalculate = () => {
    const numbers = Array.from({ length: 1000000 }, (_, i) => i);
    postMessage({ numbers });
  };
  
  return (
    <div>
      <button onClick={handleCalculate} disabled={loading}>
        Calculate
      </button>
      {loading && <p>Loading...</p>}
      {data && <p>Result: {data.result}</p>}
    </div>
  );
}
```

**3. Worker Pool для параллельных задач:**

```javascript
class WorkerPool {
  constructor(workerScript, poolSize = 4) {
    this.workerScript = workerScript;
    this.poolSize = poolSize;
    this.workers = [];
    this.taskQueue = [];
    this.activeWorkers = new Set();
    
    // Создаем workers
    for (let i = 0; i < poolSize; i++) {
      this.createWorker();
    }
  }
  
  createWorker() {
    const worker = new Worker(this.workerScript);
    
    worker.addEventListener('message', (event) => {
      const { taskId, result } = event.data;
      
      // Разрешаем promise задачи
      const task = this.workers.find(w => w.worker === worker)?.currentTask;
      if (task && task.id === taskId) {
        task.resolve(result);
      }
      
      // Освобождаем worker
      this.activeWorkers.delete(worker);
      this.workers.find(w => w.worker === worker).currentTask = null;
      
      // Запускаем следующую задачу из очереди
      this.processQueue();
    });
    
    worker.addEventListener('error', (error) => {
      const task = this.workers.find(w => w.worker === worker)?.currentTask;
      if (task) {
        task.reject(error);
      }
      
      this.activeWorkers.delete(worker);
      this.workers.find(w => w.worker === worker).currentTask = null;
      this.processQueue();
    });
    
    this.workers.push({ worker, currentTask: null });
  }
  
  exec(data) {
    return new Promise((resolve, reject) => {
      const taskId = Math.random().toString(36);
      const task = { id: taskId, data, resolve, reject };
      
      // Ищем свободного worker
      const availableWorker = this.workers.find(
        w => !this.activeWorkers.has(w.worker)
      );
      
      if (availableWorker) {
        this.executeTask(availableWorker, task);
      } else {
        // Добавляем в очередь
        this.taskQueue.push(task);
      }
    });
  }
  
  executeTask(workerObj, task) {
    this.activeWorkers.add(workerObj.worker);
    workerObj.currentTask = task;
    workerObj.worker.postMessage({ taskId: task.id, data: task.data });
  }
  
  processQueue() {
    if (this.taskQueue.length === 0) return;
    
    const availableWorker = this.workers.find(
      w => !this.activeWorkers.has(w.worker)
    );
    
    if (availableWorker) {
      const task = this.taskQueue.shift();
      this.executeTask(availableWorker, task);
    }
  }
  
  terminate() {
    this.workers.forEach(({ worker }) => worker.terminate());
    this.workers = [];
    this.activeWorkers.clear();
    this.taskQueue = [];
  }
}

// Использование
const pool = new WorkerPool('heavy-worker.js', 4);

// Параллельная обработка нескольких задач
const tasks = Array.from({ length: 10 }, (_, i) => ({
  id: i,
  data: Array.from({ length: 100000 }, (_, j) => j)
}));

Promise.all(
  tasks.map(task => pool.exec(task))
).then(results => {
  console.log('All tasks completed:', results);
  pool.terminate();
});
```

**4. Comlink для простой работы с Workers:**

```javascript
// Comlink делает работу с workers проще (RPC-style)
import * as Comlink from 'comlink';

// worker.js
import * as Comlink from 'comlink';

const api = {
  async processData(data) {
    // Тяжелые вычисления
    return data.map(x => x * 2);
  },
  
  async sortLargeArray(array) {
    return array.sort((a, b) => a - b);
  }
};

Comlink.expose(api);

// main.js
import * as Comlink from 'comlink';

const worker = new Worker('worker.js');
const api = Comlink.wrap(worker);

// Использование как обычные функции!
async function processData() {
  const data = [1, 2, 3, 4, 5];
  const result = await api.processData(data);
  console.log(result); // [2, 4, 6, 8, 10]
  
  const array = [5, 2, 8, 1, 9];
  const sorted = await api.sortLargeArray(array);
  console.log(sorted); // [1, 2, 5, 8, 9]
}
```

**5. Передача данных между Worker и Main:**

```javascript
// Structured Clone (копирование данных)
const worker = new Worker('worker.js');
const largeArray = new Array(1000000).fill(0);

// ❌ Плохо - копирует весь массив (медленно)
worker.postMessage({ data: largeArray });

// ✅ Хорошо - Transferable Objects (передача владения)
const buffer = new ArrayBuffer(1000000);
worker.postMessage({ data: buffer }, [buffer]);
// buffer теперь недоступен в main thread

// Transferable types:
// - ArrayBuffer
// - MessagePort
// - ImageBitmap
// - OffscreenCanvas

// worker.js
self.addEventListener('message', (event) => {
  const { data } = event.data;
  
  // Обработка ArrayBuffer
  const view = new Uint8Array(data);
  // ... обработка
  
  // Возвращаем обратно (transfer)
  self.postMessage({ result: data }, [data]);
});
```

**6. Shared Worker (разделяемый между вкладками):**

```javascript
// shared-worker.js
const connections = [];

self.addEventListener('connect', (event) => {
  const port = event.ports[0];
  connections.push(port);
  
  port.addEventListener('message', (event) => {
    const { type, data } = event.data;
    
    if (type === 'broadcast') {
      // Отправляем всем подключенным вкладкам
      connections.forEach(p => {
        if (p !== port) {
          p.postMessage({ type: 'update', data });
        }
      });
    }
  });
  
  port.start();
});

// main.js (вкладка 1 и вкладка 2)
const worker = new SharedWorker('shared-worker.js');

worker.port.addEventListener('message', (event) => {
  console.log('Update from another tab:', event.data);
});

worker.port.start();

// Отправка сообщения всем вкладкам
worker.port.postMessage({
  type: 'broadcast',
  data: { message: 'Hello from this tab' }
});
```

**Ограничения Web Workers:**

```javascript
// ❌ Нет доступа к:
// - DOM (document, window)
// - parent window объекты
// - Некоторым API (alert, confirm)

// ✅ Доступно:
// - XMLHttpRequest / fetch
// - WebSockets
// - IndexedDB
// - Cache API
// - setTimeout / setInterval
// - console
// - navigator
// - location (read-only)
// - Worker (nested workers)

// worker.js
// ✅ Работает
self.addEventListener('message', async (event) => {
  // Fetch API работает
  const response = await fetch('https://api.example.com/data');
  const data = await response.json();
  
  // IndexedDB работает
  const db = await openDB('mydb', 1);
  await db.put('store', data);
  
  // WebSocket работает
  const ws = new WebSocket('wss://example.com/socket');
  
  self.postMessage({ result: data });
});

// ❌ Не работает
// document.getElementById('div'); // Error!
// window.alert('Hello'); // Error!
// localStorage.setItem('key', 'value'); // Error!
```

**Для чего:**
- Тяжелые вычисления (парсинг, сортировка, шифрование)
- Обработка больших данных
- Image/Video processing
- Background sync
- Офлайн кэширование (Service Workers)

**Плюсы:**
- Не блокирует UI
- Параллельные вычисления
- Улучшение производительности

**Минусы:**
- Нет доступа к DOM
- Overhead на создание и коммуникацию
- Сложность отладки
- Нужно управлять жизненным циклом

**Best Practices:**

```javascript
// ✅ Переиспользуйте workers
const workerCache = new Map();

function getWorker(name) {
  if (!workerCache.has(name)) {
    workerCache.set(name, new Worker(`${name}.js`));
  }
  return workerCache.get(name);
}

// ✅ Terminate workers когда не нужны
useEffect(() => {
  const worker = new Worker('worker.js');
  
  return () => {
    worker.terminate();
  };
}, []);

// ✅ Используйте Worker Pool для множества задач
// ✅ Используйте Transferable Objects для больших данных
// ✅ Обрабатывайте ошибки
// ✅ Используйте timeout для долгих операций
```

---

Это четвертая часть с Browser APIs. Продолжить с пятой частью (CSS, CSS-in-JS, Build Tools)?


# Подготовка к собеседованию: Strong Middle Frontend React Developer (Часть 7 - Финальная)

## Accessibility (A11y)

---

### 23. Accessibility - WCAG, ARIA, лучшие практики

**Как работает:**

Accessibility (A11y) - доступность веб-приложений для людей с ограниченными возможностями.

**WCAG уровни:**
- **A** - базовый уровень (минимум)
- **AA** - рекомендуемый уровень (стандарт)
- **AAA** - максимальный уровень

**Четыре принципа WCAG (POUR):**

**1. Perceivable (Воспринимаемость):**

```jsx
// Альтернативный текст для изображений
// ❌ Плохо
<img src="logo.png" />

// ✅ Хорошо
<img src="logo.png" alt="Company Logo" />

// Декоративные изображения
<img src="decoration.png" alt="" /> // Пустой alt для screen readers

// Сложные изображения
<img 
  src="chart.png" 
  alt="Sales chart showing 50% increase in Q4 2024"
  aria-describedby="chart-details"
/>
<div id="chart-details">
  Detailed description: Sales increased from $100k in Q3 to $150k in Q4...
</div>

// Видео с субтитрами
<video controls>
  <source src="video.mp4" type="video/mp4" />
  <track 
    kind="captions" 
    src="captions.vtt" 
    srclang="en" 
    label="English"
  />
</video>

// Контрастность цветов
// ❌ Плохо - низкий контраст
.text {
  color: #777; /* серый */
  background: #fff; /* белый */
  /* Контраст ratio: 4.5:1 минимум для текста */
}

// ✅ Хорошо
.text {
  color: #333; /* темно-серый */
  background: #fff; /* белый */
}

// Проверка контраста:
// - Normal text: минимум 4.5:1
// - Large text (18px+): минимум 3:1
// - UI components: минимум 3:1
```

**2. Operable (Управляемость):**

```jsx
// Keyboard navigation
// ✅ Все интерактивные элементы доступны с клавиатуры

// ❌ Плохо - div без keyboard support
<div onClick={handleClick}>Click me</div>

// ✅ Хорошо - button по умолчанию keyboard accessible
<button onClick={handleClick}>Click me</button>

// Или добавить keyboard support к div
<div
  role="button"
  tabIndex={0}
  onClick={handleClick}
  onKeyDown={(e) => {
    if (e.key === 'Enter' || e.key === ' ') {
      handleClick();
    }
  }}
>
  Click me
</div>

// Skip links для быстрой навигации
function App() {
  return (
    <>
      <a href="#main-content" className="skip-link">
        Skip to main content
      </a>
      <Header />
      <main id="main-content">
        <Content />
      </main>
    </>
  );
}

// CSS для skip link
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  background: #000;
  color: #fff;
  padding: 8px;
  z-index: 100;
}

.skip-link:focus {
  top: 0;
}

// Focus management
function Modal({ isOpen, onClose, children }) {
  const modalRef = useRef(null);
  const previousFocusRef = useRef(null);
  
  useEffect(() => {
    if (isOpen) {
      // Сохраняем текущий focus
      previousFocusRef.current = document.activeElement;
      
      // Фокусируемся на модалке
      modalRef.current?.focus();
      
      // Trap focus внутри модалки
      const focusableElements = modalRef.current.querySelectorAll(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      );
      const firstElement = focusableElements[0];
      const lastElement = focusableElements[focusableElements.length - 1];
      
      const handleTab = (e) => {
        if (e.key === 'Tab') {
          if (e.shiftKey && document.activeElement === firstElement) {
            e.preventDefault();
            lastElement.focus();
          } else if (!e.shiftKey && document.activeElement === lastElement) {
            e.preventDefault();
            firstElement.focus();
          }
        }
        
        if (e.key === 'Escape') {
          onClose();
        }
      };
      
      document.addEventListener('keydown', handleTab);
      
      return () => {
        document.removeEventListener('keydown', handleTab);
        // Возвращаем focus обратно
        previousFocusRef.current?.focus();
      };
    }
  }, [isOpen, onClose]);
  
  if (!isOpen) return null;
  
  return (
    <div
      ref={modalRef}
      role="dialog"
      aria-modal="true"
      aria-labelledby="modal-title"
      tabIndex={-1}
    >
      <h2 id="modal-title">Modal Title</h2>
      {children}
      <button onClick={onClose}>Close</button>
    </div>
  );
}

// Focus indicators
// ✅ Не убирайте outline без альтернативы
button {
  outline: 2px solid blue;
  outline-offset: 2px;
}

button:focus-visible {
  outline: 2px solid blue;
  outline-offset: 2px;
}

// :focus-visible показывает outline только при keyboard навигации
```

**3. Understandable (Понятность):**

```jsx
// Языковая атрибуция
<html lang="en">
  <head>
    <title>My App</title>
  </head>
  <body>
    <p>English text</p>
    <p lang="ru">Русский текст</p>
    <p lang="fr">Texte français</p>
  </body>
</html>

// Labels для форм
// ❌ Плохо
<input type="text" placeholder="Name" />

// ✅ Хорошо
<label htmlFor="name">Name</label>
<input type="text" id="name" />

// Или implicit label
<label>
  Name
  <input type="text" />
</label>

// Группировка форм
<fieldset>
  <legend>Personal Information</legend>
  
  <label htmlFor="firstName">First Name</label>
  <input type="text" id="firstName" />
  
  <label htmlFor="lastName">Last Name</label>
  <input type="text" id="lastName" />
</fieldset>

// Ошибки валидации
function Form() {
  const [errors, setErrors] = useState({});
  
  return (
    <form aria-labelledby="form-title">
      <h2 id="form-title">Registration Form</h2>
      
      {Object.keys(errors).length > 0 && (
        <div role="alert" aria-live="polite">
          <h3>Please fix the following errors:</h3>
          <ul>
            {Object.entries(errors).map(([field, error]) => (
              <li key={field}>{error}</li>
            ))}
          </ul>
        </div>
      )}
      
      <div>
        <label htmlFor="email">Email</label>
        <input
          type="email"
          id="email"
          aria-invalid={!!errors.email}
          aria-describedby={errors.email ? 'email-error' : undefined}
        />
        {errors.email && (
          <span id="email-error" role="alert">
            {errors.email}
          </span>
        )}
      </div>
    </form>
  );
}

// Предсказуемость
// ✅ Не меняйте контекст при focus
// ❌ Плохо - автоматический submit при focus
<input onFocus={handleSubmit} />

// ✅ Хорошо - явное действие для submit
<input onBlur={handleValidation} />
<button onClick={handleSubmit}>Submit</button>
```

**4. Robust (Надежность):**

```jsx
// Валидный HTML
// ✅ Правильное использование семантических тегов
<header>
  <nav>
    <ul>
      <li><a href="/">Home</a></li>
    </ul>
  </nav>
</header>

<main>
  <article>
    <h1>Title</h1>
    <p>Content</p>
  </article>
</main>

<footer>
  <p>&copy; 2024</p>
</footer>

// Совместимость с assistive technologies
// ARIA roles, states, properties
```

**ARIA (Accessible Rich Internet Applications):**

```jsx
// ARIA Roles
<div role="navigation">Navigation</div>
<div role="main">Main content</div>
<div role="complementary">Sidebar</div>
<div role="contentinfo">Footer</div>
<div role="search">Search form</div>
<div role="button" tabIndex={0}>Custom button</div>
<div role="alert">Error message</div>

// ARIA States
<button aria-pressed={isPressed}>Toggle</button>
<button aria-expanded={isExpanded}>Menu</button>
<input aria-invalid={hasError} />
<input aria-required={true} />
<div aria-hidden={!isVisible}>Content</div>
<button aria-disabled={isDisabled}>Submit</button>

// ARIA Properties
<button aria-label="Close dialog">×</button>
<input aria-labelledby="label-id" />
<input aria-describedby="description-id" />
<div aria-live="polite">Dynamic content</div>
<div aria-atomic="true">Updated together</div>

// Dropdown Menu
function Dropdown() {
  const [isOpen, setIsOpen] = useState(false);
  
  return (
    <div>
      <button
        aria-expanded={isOpen}
        aria-haspopup="true"
        aria-controls="menu"
        onClick={() => setIsOpen(!isOpen)}
      >
        Menu
      </button>
      
      {isOpen && (
        <ul
          id="menu"
          role="menu"
          aria-labelledby="menu-button"
        >
          <li role="menuitem">
            <a href="/profile">Profile</a>
          </li>
          <li role="menuitem">
            <a href="/settings">Settings</a>
          </li>
          <li role="menuitem">
            <a href="/logout">Logout</a>
          </li>
        </ul>
      )}
    </div>
  );
}

// Tabs
function Tabs({ tabs }) {
  const [activeTab, setActiveTab] = useState(0);
  
  return (
    <div>
      <div role="tablist" aria-label="Content tabs">
        {tabs.map((tab, index) => (
          <button
            key={index}
            role="tab"
            aria-selected={activeTab === index}
            aria-controls={`panel-${index}`}
            id={`tab-${index}`}
            tabIndex={activeTab === index ? 0 : -1}
            onClick={() => setActiveTab(index)}
          >
            {tab.label}
          </button>
        ))}
      </div>
      
      {tabs.map((tab, index) => (
        <div
          key={index}
          role="tabpanel"
          id={`panel-${index}`}
          aria-labelledby={`tab-${index}`}
          hidden={activeTab !== index}
        >
          {tab.content}
        </div>
      ))}
    </div>
  );
}

// Live Regions (динамический контент)
function Notifications() {
  const [message, setMessage] = useState('');
  
  return (
    <>
      <button onClick={() => setMessage('Item added to cart')}>
        Add to cart
      </button>
      
      {/* Polite - не прерывает screen reader */}
      <div
        role="status"
        aria-live="polite"
        aria-atomic="true"
      >
        {message}
      </div>
      
      {/* Assertive - прерывает screen reader (для важных сообщений) */}
      <div
        role="alert"
        aria-live="assertive"
        aria-atomic="true"
      >
        {message}
      </div>
    </>
  );
}
```

**Accessibility Testing:**

```javascript
// Jest + Testing Library
import { render, screen } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

test('button is accessible', async () => {
  const { container } = render(<button>Click me</button>);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});

// React Testing Library - accessibility queries
test('form is accessible', () => {
  render(<Form />);
  
  // getByRole - предпочтительный способ
  const button = screen.getByRole('button', { name: /submit/i });
  const textbox = screen.getByRole('textbox', { name: /email/i });
  
  // getByLabelText
  const input = screen.getByLabelText(/password/i);
  
  // getByAltText для изображений
  const img = screen.getByAltText(/company logo/i);
});

// Lighthouse CI
// lighthouse.config.js
module.exports = {
  ci: {
    collect: {
      numberOfRuns: 3,
      url: ['http://localhost:3000/'],
    },
    assert: {
      assertions: {
        'categories:accessibility': ['error', { minScore: 0.9 }],
        'categories:performance': ['error', { minScore: 0.8 }],
      },
    },
  },
};
```

**Keyboard Navigation:**

```jsx
// Комбинации клавиш
function useKeyboardShortcuts() {
  useEffect(() => {
    const handleKeyDown = (e) => {
      // Ctrl/Cmd + K - открыть поиск
      if ((e.ctrlKey || e.metaKey) && e.key === 'k') {
        e.preventDefault();
        openSearch();
      }
      
      // Escape - закрыть модалку
      if (e.key === 'Escape') {
        closeModal();
      }
      
      // Arrow keys - навигация
      if (e.key === 'ArrowDown') {
        e.preventDefault();
        navigateDown();
      }
    };
    
    document.addEventListener('keydown', handleKeyDown);
    return () => document.removeEventListener('keydown', handleKeyDown);
  }, []);
}

// Roving tabindex для списков
function List({ items }) {
  const [focusedIndex, setFocusedIndex] = useState(0);
  
  const handleKeyDown = (e, index) => {
    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault();
        setFocusedIndex((focusedIndex + 1) % items.length);
        break;
      case 'ArrowUp':
        e.preventDefault();
        setFocusedIndex((focusedIndex - 1 + items.length) % items.length);
        break;
      case 'Home':
        e.preventDefault();
        setFocusedIndex(0);
        break;
      case 'End':
        e.preventDefault();
        setFocusedIndex(items.length - 1);
        break;
    }
  };
  
  return (
    <ul role="listbox">
      {items.map((item, index) => (
        <li
          key={item.id}
          role="option"
          tabIndex={focusedIndex === index ? 0 : -1}
          onKeyDown={(e) => handleKeyDown(e, index)}
          onFocus={() => setFocusedIndex(index)}
          aria-selected={focusedIndex === index}
        >
          {item.name}
        </li>
      ))}
    </ul>
  );
}
```

**Для чего:**
- Доступность для людей с ограниченными возможностями
- Улучшение UX для всех пользователей
- SEO (семантический HTML)
- Legal compliance (законодательные требования)

**Best Practices:**

```jsx
// ✅ Используйте семантический HTML
<button> вместо <div onClick>
<nav>, <main>, <aside>, <footer>
<h1>-<h6> в правильном порядке

// ✅ Всегда предоставляйте альтернативный текст
<img alt="Description" />

// ✅ Обеспечьте keyboard navigation
// Все интерактивные элементы доступны с клавиатуры

// ✅ Достаточная контрастность цветов
// Минимум 4.5:1 для обычного текста

// ✅ Не полагайтесь только на цвет
// ❌ "Click the red button"
// ✅ "Click the 'Submit' button"

// ✅ Labels для всех inputs
<label htmlFor="input">Label</label>
<input id="input" />

// ✅ Focus indicators
// Не убирайте outline без замены

// ✅ ARIA когда семантического HTML недостаточно
// Но предпочитайте семантический HTML

// ✅ Тестируйте с screen readers
// - NVDA (Windows, free)
// - JAWS (Windows)
// - VoiceOver (macOS, iOS)
// - TalkBack (Android)

// ✅ Используйте accessibility linters
// - eslint-plugin-jsx-a11y
// - axe DevTools
```

---

### 24. Error Handling и Error Boundaries

**Как работает:**

Error handling - правильная обработка и отображение ошибок.

**Error Boundaries (React):**

```jsx
// ErrorBoundary.jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null, errorInfo: null };
  }
  
  static getDerivedStateFromError(error) {
    // Обновляем state чтобы показать fallback UI
    return { hasError: true };
  }
  
  componentDidCatch(error, errorInfo) {
    // Логируем ошибку в error reporting service
    console.error('Error caught by boundary:', error, errorInfo);
    
    this.setState({
      error,
      errorInfo
    });
    
    // Отправка в Sentry/LogRocket
    logErrorToService(error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return (
        <div className="error-container">
          <h2>Something went wrong</h2>
          <details style={{ whiteSpace: 'pre-wrap' }}>
            {this.state.error && this.state.error.toString()}
            <br />
            {this.state.errorInfo.componentStack}
          </details>
          <button onClick={() => window.location.reload()}>
            Reload page
          </button>
        </div>
      );
    }
    
    return this.props.children;
  }
}

// Использование
function App() {
  return (
    <ErrorBoundary>
      <Header />
      <ErrorBoundary>
        <MainContent />
      </ErrorBoundary>
      <ErrorBoundary>
        <Sidebar />
      </ErrorBoundary>
    </ErrorBoundary>
  );
}

// Error Boundary с reset
class ErrorBoundaryWithReset extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }
  
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }
  
  resetError = () => {
    this.setState({ hasError: false });
  }
  
  render() {
    if (this.state.hasError) {
      return this.props.fallback({ resetError: this.resetError });
    }
    
    return this.props.children;
  }
}

// Использование
<ErrorBoundaryWithReset
  fallback={({ resetError }) => (
    <div>
      <h2>Error occurred</h2>
      <button onClick={resetError}>Try again</button>
    </div>
  )}
>
  <Component />
</ErrorBoundaryWithReset>
```

**React 19 - use() hook для Error Boundaries:**

```jsx
// React 19+ (будущее)
function ErrorBoundary({ children, fallback }) {
  const [error, resetError] = use(ErrorBoundary);
  
  if (error) {
    return fallback({ error, resetError });
  }
  
  return children;
}
```

**Try-Catch в компонентах:**

```jsx
// ❌ Плохо - ошибка упадет весь компонент
function Component() {
  const data = JSON.parse(invalidJson); // Error!
  return <div>{data}</div>;
}

// ✅ Хорошо - обработка ошибки
function Component() {
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    try {
      const parsed = JSON.parse(jsonString);
      setData(parsed);
    } catch (err) {
      setError(err.message);
      console.error('Failed to parse JSON:', err);
    }
  }, [jsonString]);
  
  if (error) {
    return <div>Error: {error}</div>;
  }
  
  return <div>{data}</div>;
}
```

**Async Error Handling:**

```jsx
// Promises
function DataFetcher() {
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetch('/api/data')
      .then(response => {
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        return response.json();
      })
      .then(data => {
        setData(data);
        setLoading(false);
      })
      .catch(error => {
        setError(error.message);
        setLoading(false);
        
        // Логирование
        console.error('Fetch error:', error);
      });
  }, []);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  
  return <div>{JSON.stringify(data)}</div>;
}

// Async/Await
async function fetchData() {
  try {
    const response = await fetch('/api/data');
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const data = await response.json();
    return data;
  } catch (error) {
    if (error.name === 'TypeError') {
      // Network error
      throw new Error('Network error. Please check your connection.');
    } else if (error.message.includes('404')) {
      throw new Error('Resource not found.');
    } else {
      throw error;
    }
  }
}

// React Query - автоматический error handling
import { useQuery } from '@tanstack/react-query';

function Component() {
  const { data, error, isLoading } = useQuery({
    queryKey: ['data'],
    queryFn: fetchData,
    retry: 3,
    retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000)
  });
  
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return <div>{data}</div>;
}
```

**Custom Error Classes:**

```javascript
// Кастомные ошибки
class APIError extends Error {
  constructor(message, statusCode, response) {
    super(message);
    this.name = 'APIError';
    this.statusCode = statusCode;
    this.response = response;
  }
}

class ValidationError extends Error {
  constructor(message, errors) {
    super(message);
    this.name = 'ValidationError';
    this.errors = errors;
  }
}

class NetworkError extends Error {
  constructor(message) {
    super(message);
    this.name = 'NetworkError';
  }
}

// Использование
async function apiRequest(url, options) {
  try {
    const response = await fetch(url, options);
    
    if (!response.ok) {
      const errorData = await response.json().catch(() => ({}));
      throw new APIError(
        errorData.message || 'API request failed',
        response.status,
        errorData
      );
    }
    
    return await response.json();
  } catch (error) {
    if (error instanceof APIError) {
      throw error;
    }
    
    if (error.name === 'TypeError') {
      throw new NetworkError('Network connection failed');
    }
    
    throw error;
  }
}

// Обработка разных типов ошибок
try {
  await apiRequest('/api/users');
} catch (error) {
  if (error instanceof ValidationError) {
    displayValidationErrors(error.errors);
  } else if (error instanceof APIError) {
    if (error.statusCode === 401) {
      redirectToLogin();
    } else if (error.statusCode === 403) {
      showPermissionError();
    } else {
      showGenericError(error.message);
    }
  } else if (error instanceof NetworkError) {
    showNetworkError();
  } else {
    showGenericError('An unexpected error occurred');
    console.error(error);
  }
}
```

**Global Error Handling:**

```javascript
// Глобальный handler для uncaught errors
window.addEventListener('error', (event) => {
  console.error('Global error:', event.error);
  
  // Отправка в error tracking
  logErrorToService({
    message: event.error.message,
    stack: event.error.stack,
    filename: event.filename,
    lineno: event.lineno,
    colno: event.colno
  });
  
  // Показать пользователю
  showErrorNotification('Something went wrong. Please try again.');
});

// Unhandled promise rejections
window.addEventListener('unhandledrejection', (event) => {
  console.error('Unhandled promise rejection:', event.reason);
  
  logErrorToService({
    type: 'unhandled_rejection',
    reason: event.reason,
    promise: event.promise
  });
  
  showErrorNotification('An error occurred. Please try again.');
});

// React 18+ - onRecoverableError
const root = ReactDOM.createRoot(
  document.getElementById('root'),
  {
    onRecoverableError: (error, errorInfo) => {
      console.error('Recoverable error:', error, errorInfo);
      logErrorToService({ error, errorInfo });
    }
  }
);
```

**Error Monitoring (Sentry):**

```javascript
// Sentry integration
import * as Sentry from '@sentry/react';
import { BrowserTracing } from '@sentry/tracing';

Sentry.init({
  dsn: 'YOUR_SENTRY_DSN',
  integrations: [new BrowserTracing()],
  tracesSampleRate: 1.0,
  environment: process.env.NODE_ENV,
});

// Error Boundary с Sentry
const SentryErrorBoundary = Sentry.withErrorBoundary(App, {
  fallback: <ErrorFallback />,
  showDialog: true,
});

// Manual error capture
try {
  riskyOperation();
} catch (error) {
  Sentry.captureException(error, {
    tags: {
      section: 'payment',
    },
    extra: {
      userId: user.id,
      orderId: order.id,
    },
  });
}

// Breadcrumbs для context
Sentry.addBreadcrumb({
  category: 'auth',
  message: 'User logged in',
  level: 'info',
});
```

**User-friendly Error Messages:**

```jsx
function ErrorMessage({ error }) {
  const getUserFriendlyMessage = (error) => {
    if (error instanceof NetworkError) {
      return {
        title: 'Connection Problem',
        message: 'Please check your internet connection and try again.',
        action: 'Retry'
      };
    }
    
    if (error instanceof APIError) {
      if (error.statusCode === 404) {
        return {
          title: 'Not Found',
          message: 'The requested resource was not found.',
          action: 'Go Back'
        };
      }
      
      if (error.statusCode === 401) {
        return {
          title: 'Authentication Required',
          message: 'Please log in to continue.',
          action: 'Log In'
        };
      }
      
      if (error.statusCode >= 500) {
        return {
          title: 'Server Error',
          message: 'Something went wrong on our end. Please try again later.',
          action: 'Retry'
        };
      }
    }
    
    return {
      title: 'Error',
      message: 'Something went wrong. Please try again.',
      action: 'Retry'
    };
  };
  
  const { title, message, action } = getUserFriendlyMessage(error);
  
  return (
    <div className="error-message">
      <h3>{title}</h3>
      <p>{message}</p>
      <button onClick={handleAction}>{action}</button>
    </div>
  );
}
```

**Для чего:**
- Предотвращение падения всего приложения
- User-friendly error messages
- Логирование и мониторинг ошибок
- Debugging в production

**Best Practices:**

```javascript
// ✅ Используйте Error Boundaries для React компонентов
// ✅ Try-catch для асинхронных операций
// ✅ Специфичные error messages для пользователей
// ✅ Логируйте ошибки в error monitoring service
// ✅ Обрабатывайте разные типы ошибок по-разному
// ✅ Предоставляйте recovery actions (retry, reload)
// ✅ Не показывайте stack traces пользователям
// ✅ Валидация данных до обработки
// ✅ Fallback UI для graceful degradation

// ❌ Не игнорируйте ошибки
// ❌ Не показывайте технические детали пользователям
// ❌ Не логируйте sensitive данные
```

---

### 25. RegExp (Регулярные выражения)

**Основы:**

```javascript
// Создание RegExp
const regex1 = /pattern/flags;
const regex2 = new RegExp('pattern', 'flags');

// Flags:
// g - global (все совпадения)
// i - case insensitive
// m - multiline (^ и $ работают для каждой строки)
// s - dotAll (. включает \n)
// u - unicode
// y - sticky (начинает поиск с lastIndex)

// Методы
const str = 'Hello World';

// test() - проверяет наличие совпадения
/world/i.test(str); // true

// exec() - возвращает массив с деталями
/(\w+)\s(\w+)/.exec(str); // ['Hello World', 'Hello', 'World']

// String методы с RegExp
str.match(/\w+/g); // ['Hello', 'World']
str.matchAll(/\w+/g); // Iterator
str.search(/world/i); // 6 (index)
str.replace(/world/i, 'Universe'); // 'Hello Universe'
str.split(/\s/); // ['Hello', 'World']
```

**Паттерны:**

```javascript
// Символы
. // любой символ (кроме \n)
\d // цифра [0-9]
\D // не цифра
\w // слово [a-zA-Z0-9_]
\W // не слово
\s // пробел [\t\n\r\f\v]
\S // не пробел
\b // граница слова
\B // не граница слова

// Квантификаторы
* // 0 или больше
+ // 1 или больше
? // 0 или 1
{n} // ровно n
{n,} // n или больше
{n,m} // от n до m

// Ленивые квантификаторы (добавить ?)
*? // минимум 0
+? // минимум 1
?? // минимум 0, максимум 1
{n,m}? // минимум n

// Наборы
[abc] // a, b или c
[^abc] // не a, b, c
[a-z] // от a до z
[A-Z0-9] // A-Z или 0-9

// Якоря
^ // начало строки
$ // конец строки
\b // граница слова

// Группы
(abc) // группа захвата
(?:abc) // не захватывающая группа
(?<name>abc) // именованная группа

// Lookahead/Lookbehind
(?=abc) // positive lookahead
(?!abc) // negative lookahead
(?<=abc) // positive lookbehind
(?<!abc) // negative lookbehind

// Альтернация
a|b // a или b
```

**Практические примеры:**

```javascript
// Email validation
const emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
emailRegex.test('user@example.com'); // true

// Phone number (US)
const phoneRegex = /^(\+1)?[-.\s]?\(?(\d{3})\)?[-.\s]?(\d{3})[-.\s]?(\d{4})$/;
phoneRegex.test('(123) 456-7890'); // true
phoneRegex.test('+1-123-456-7890'); // true

// URL validation
const urlRegex = /^https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)$/;
urlRegex.test('https://example.com/path'); // true

// Password (min 8 chars, 1 uppercase, 1 lowercase, 1 number, 1 special)
const passwordRegex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/;
passwordRegex.test('Password123!'); // true

// Hex color
const hexColorRegex = /^#([A-Fa-f0-9]{6}|[A-Fa-f0-9]{3})$/;
hexColorRegex.test('#FF5733'); // true
hexColorRegex.test('#F57'); // true

// Date (MM/DD/YYYY)
const dateRegex = /^(0[1-9]|1[0-2])\/(0[1-9]|[12][0-9]|3[01])\/\d{4}$/;
dateRegex.test('12/31/2024'); // true

// Credit card
const creditCardRegex = /^\d{4}[-\s]?\d{4}[-\s]?\d{4}[-\s]?\d{4}$/;
creditCardRegex.test('1234-5678-9012-3456'); // true

// Username (3-16 chars, alphanumeric + underscore)
const usernameRegex = /^[a-zA-Z0-9_]{3,16}$/;
usernameRegex.test('user_123'); // true

// IP Address
const ipRegex = /^((25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$/;
ipRegex.test('192.168.1.1'); // true

// HTML tags
const htmlTagRegex = /<([a-z]+)([^<]+)*(?:>(.*)<\/\1>|\s+\/>)/gi;
'<div>Hello</div>'.match(htmlTagRegex);

// Extract domain from URL
const domainRegex = /https?:\/\/(www\.)?([a-zA-Z0-9-]+\.[a-zA-Z]{2,})/;
const match = 'https://www.example.com/path'.match(domainRegex);
console.log(match[2]); // 'example.com'
```

**Группы захвата:**

```javascript
// Обычная группа
const nameRegex = /(\w+)\s(\w+)/;
const match = 'John Doe'.match(nameRegex);
console.log(match[0]); // 'John Doe' (полное совпадение)
console.log(match[1]); // 'John' (первая группа)
console.log(match[2]); // 'Doe' (вторая группа)

// Именованные группы
const dateRegex = /(?<month>\d{2})\/(?<day>\d{2})\/(?<year>\d{4})/;
const dateMatch = '12/31/2024'.match(dateRegex);
console.log(dateMatch.groups.month); // '12'
console.log(dateMatch.groups.day); // '31'
console.log(dateMatch.groups.year); // '2024'

// Не захватывающая группа
const urlRegex = /https?:\/\/(?:www\.)?example\.com/;
// (?:www\.)? не создает группу захвата

// Backreferences
const duplicateRegex = /(\w+)\s\1/;
duplicateRegex.test('hello hello'); // true (повтор слова)
duplicateRegex.test('hello world'); // false
```

**Replace с RegExp:**

```javascript
// Простая замена
'hello world'.replace(/world/, 'universe'); // 'hello universe'

// Global замена
'hello hello'.replace(/hello/g, 'hi'); // 'hi hi'

// Функция замены
const str = 'John Doe, Jane Smith';
str.replace(/(\w+)\s(\w+)/g, (match, firstName, lastName) => {
  return `${lastName}, ${firstName}`;
});
// 'Doe, John, Smith, Jane'

// Именованные группы в replace
const date = '2024-12-31';
date.replace(
  /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/,
  '$<month>/$<day>/$<year>'
);
// '12/31/2024'

// Capitalize первая буква каждого слова
const title = 'hello world from javascript';
title.replace(/\b\w/g, char => char.toUpperCase());
// 'Hello World From Javascript'

// Remove HTML tags
const html = '<div>Hello <strong>World</strong></div>';
html.replace(/<[^>]*>/g, ''); // 'Hello World'

// Format phone number
const phone = '1234567890';
phone.replace(/(\d{3})(\d{3})(\d{4})/, '($1) $2-$3');
// '(123) 456-7890'
```

**Lookahead/Lookbehind:**

```javascript
// Positive Lookahead (?=...)
// Проверяет что после паттерна идет определенный текст
const priceRegex = /\d+(?=\s*dollars)/;
'100 dollars'.match(priceRegex); // ['100']
'100 euros'.match(priceRegex); // null

// Negative Lookahead (?!...)
// Проверяет что после паттерна НЕ идет определенный текст
const notDollarRegex = /\d+(?!\s*dollars)/;
'100 euros'.match(notDollarRegex); // ['100']
'100 dollars'.match(notDollarRegex); // null

// Positive Lookbehind (?<=...)
// Проверяет что перед паттерном идет определенный текст
const afterDollarRegex = /(?<=\$)\d+/;
'$100'.match(afterDollarRegex); // ['100']
'€100'.match(afterDollarRegex); // null

// Negative Lookbehind (?<!...)
// Проверяет что перед паттерном НЕ идет определенный текст
const notAfterDollarRegex = /(?<!\$)\d+/;
'€100'.match(notAfterDollarRegex); // ['100']
'$100'.match(notAfterDollarRegex); // null

// Password validation с lookaheads
const strongPasswordRegex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&]).{8,}$/;
// Должен содержать: lowercase, uppercase, digit, special char, минимум 8 символов
```

**React валидация с RegExp:**

```jsx
function FormValidation() {
  const [formData, setFormData] = useState({
    email: '',
    phone: '',
    password: ''
  });
  
  const [errors, setErrors] = useState({});
  
  const validate = () => {
    const newErrors = {};
    
    // Email
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(formData.email)) {
      newErrors.email = 'Invalid email format';
    }
    
    // Phone
    const phoneRegex = /^\d{3}-\d{3}-\d{4}$/;
    if (!phoneRegex.test(formData.phone)) {
      newErrors.phone = 'Phone must be in format: 123-456-7890';
    }
    
    // Password
    const passwordRegex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&]).{8,}$/;
    if (!passwordRegex.test(formData.password)) {
      newErrors.password = 'Password must contain uppercase, lowercase, number, special char, min 8 chars';
    }
    
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    if (validate()) {
      console.log('Form is valid!', formData);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>Email</label>
        <input
          type="email"
          value={formData.email}
          onChange={(e) => setFormData({ ...formData, email: e.target.value })}
        />
        {errors.email && <span>{errors.email}</span>}
      </div>
      
      <div>
        <label>Phone</label>
        <input
          type="tel"
          value={formData.phone}
          onChange={(e) => setFormData({ ...formData, phone: e.target.value })}
          placeholder="123-456-7890"
        />
        {errors.phone && <span>{errors.phone}</span>}
      </div>
      
      <div>
        <label>Password</label>
        <input
          type="password"
          value={formData.password}
          onChange={(e) => setFormData({ ...formData, password: e.target.value })}
        />
        {errors.password && <span>{errors.password}</span>}
      </div>
      
      <button type="submit">Submit</button>
    </form>
  );
}
```

**Performance:**

```javascript
// ✅ Компилируйте RegExp заранее (если используется многократно)
const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/; // Вне функции

function validateEmail(email) {
  return emailRegex.test(email);
}

// ❌ Не создавайте RegExp в цикле
for (let i = 0; i < items.length; i++) {
  // ❌ Плохо
  if (/pattern/.test(items[i])) { }
}

// ✅ Хорошо
const regex = /pattern/;
for (let i = 0; i < items.length; i++) {
  if (regex.test(items[i])) { }
}

// ⚠️ Опасность catastrophic backtracking
// ❌ Плохо - может зависнуть на длинных строках
const badRegex = /(a+)+b/;
badRegex.test('aaaaaaaaaaaaaaaaaaaaaa'); // Долго!

// ✅ Хорошо - оптимизировано
const goodRegex = /a+b/;
```

**Для чего:**
- Валидация форм
- Парсинг текста
- Search and replace
- Извлечение данных из строк

---

Отлично! Все 7 частей готовы. Теперь давайте соберем все файлы в outputs директорию для пользователя.


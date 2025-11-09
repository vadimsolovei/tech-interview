# Подготовка к собеседованию: Strong Middle Frontend React Developer

## JavaScript (ES6+)

### 1. Что такое замыкания (closures) и как они работают?

**Как работает:**
Замыкание — это функция, которая имеет доступ к переменным из внешней области видимости, даже после того, как внешняя функция завершила выполнение. JavaScript создает замыкание каждый раз, когда функция создается.

```javascript
function createCounter() {
  let count = 0;
  return function() {
    return ++count;
  };
}

const counter = createCounter();
console.log(counter()); // 1
console.log(counter()); // 2
```

**Для чего:**
- Инкапсуляция данных (приватные переменные)
- Создание функций-фабрик
- Каррирование и частичное применение
- Колбэки с сохранением контекста

**Плюсы:**
- Позволяют создавать приватные переменные
- Мощный инструмент для функционального программирования
- Естественная работа с асинхронным кодом

**Минусы:**
- Могут приводить к утечкам памяти, если неправильно использовать
- Усложняют отладку
- Увеличивают потребление памяти

**Аналоги:**
- В других языках: private поля в классах, блоки с локальными переменными

---

### 2. В чем разница между var, let и const?

**Как работает:**

**var:**
- Function scope (область видимости — функция)
- Всплытие (hoisting) с инициализацией undefined
- Можно переопределять

**let:**
- Block scope (область видимости — блок)
- Всплытие без инициализации (Temporal Dead Zone)
- Можно переприсваивать

**const:**
- Block scope
- Всплытие без инициализации
- Нельзя переприсваивать (но объекты мутабельны)

```javascript
// var
console.log(x); // undefined (hoisting)
var x = 5;
if (true) {
  var x = 10; // та же переменная!
}
console.log(x); // 10

// let
console.log(y); // ReferenceError
let y = 5;
if (true) {
  let y = 10; // другая переменная
  console.log(y); // 10
}
console.log(y); // 5

// const
const z = { value: 5 };
z.value = 10; // OK, мутация объекта
z = {}; // Error, переприсвоение
```

**Для чего:**
- var — устаревший способ, используется для совместимости
- let — для переменных, которые будут изменяться
- const — для констант и ссылок на объекты (default choice)

**Плюсы let/const:**
- Предсказуемая область видимости
- Защита от случайных переопределений
- Явное выражение намерений

**Минусы var:**
- Непредсказуемое поведение из-за function scope
- Легко создать баги с переопределением

---

### 3. Что такое Event Loop и как работает асинхронность в JavaScript?

**Как работает:**

Event Loop — это механизм, который позволяет JavaScript выполнять неблокирующие операции, несмотря на то, что JavaScript однопоточный.

**Основные компоненты:**
1. **Call Stack** — стек вызовов, где выполняется синхронный код
2. **Web APIs** — браузерные API (setTimeout, fetch, DOM events)
3. **Callback Queue** — очередь макрозадач (callbacks, events)
4. **Microtask Queue** — очередь микрозадач (Promises, queueMicrotask)
5. **Event Loop** — проверяет Call Stack и перемещает задачи из очередей

**Порядок выполнения:**
1. Синхронный код из Call Stack
2. Все микрозадачи (Promises)
3. Одна макрозадача (setTimeout, setInterval)
4. Рендеринг (если необходимо)
5. Повтор

```javascript
console.log('1');

setTimeout(() => console.log('2'), 0);

Promise.resolve().then(() => console.log('3'));

console.log('4');

// Вывод: 1, 4, 3, 2
```

**Для чего:**
- Неблокирующее выполнение I/O операций
- Обработка событий пользователя
- Выполнение асинхронных запросов
- Плавная отрисовка UI

**Плюсы:**
- Простая модель программирования (однопоточность)
- Эффективная обработка I/O
- Отзывчивый UI

**Минусы:**
- Сложно отлаживать асинхронный код
- Блокирующие операции парализуют всё приложение
- Нет параллелизма (только конкурентность)

**Аналоги:**
- Web Workers для параллельных вычислений
- Service Workers для фоновых задач

---

### 4. Объясните Promise, async/await

**Как работает:**

**Promise** — объект, представляющий результат асинхронной операции (pending, fulfilled, rejected).

```javascript
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    const success = true;
    if (success) {
      resolve('Данные получены');
    } else {
      reject('Ошибка');
    }
  }, 1000);
});

promise
  .then(data => console.log(data))
  .catch(error => console.error(error))
  .finally(() => console.log('Завершено'));
```

**async/await** — синтаксический сахар над Promise для более читаемого асинхронного кода.

```javascript
async function fetchData() {
  try {
    const response = await fetch('/api/data');
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Ошибка:', error);
    throw error;
  }
}
```

**Для чего:**
- Работа с асинхронными операциями (API, файлы, таймеры)
- Избежание "callback hell"
- Композиция асинхронных операций
- Обработка ошибок в асинхронном коде

**Плюсы Promise:**
- Чейнинг операций
- Лучшая обработка ошибок чем callbacks
- Promise.all, Promise.race для параллелизма

**Плюсы async/await:**
- Синхронный стиль кода
- Более читаемый код
- Удобная обработка ошибок через try/catch

**Минусы:**
- async/await скрывает асинхронную природу
- Легко забыть await (получить Promise вместо значения)
- Последовательный await может быть медленнее параллельного Promise.all

**Аналоги:**
- Callbacks (устаревший подход)
- RxJS Observables (для потоков данных)
- Generators с co/yield (устаревший подход)

---

### 5. Что такое this и как он работает?

**Как работает:**

`this` — контекстная переменная, которая ссылается на объект, от имени которого вызвана функция. Значение определяется в момент вызова.

**Правила определения this:**

1. **Явное связывание (call, apply, bind)**
```javascript
function greet() {
  console.log(this.name);
}
const user = { name: 'Alice' };
greet.call(user); // 'Alice'
```

2. **Неявное связывание (метод объекта)**
```javascript
const obj = {
  name: 'Bob',
  greet() {
    console.log(this.name);
  }
};
obj.greet(); // 'Bob'
```

3. **new связывание**
```javascript
function User(name) {
  this.name = name;
}
const user = new User('Charlie'); // this = новый объект
```

4. **Стрелочные функции (лексический this)**
```javascript
const obj = {
  name: 'Dave',
  greet: () => {
    console.log(this.name); // this из внешней области
  }
};
```

5. **По умолчанию (глобальный объект или undefined в strict mode)**
```javascript
function test() {
  console.log(this); // window (или undefined в strict mode)
}
```

**Для чего:**
- Доступ к свойствам объекта в методах
- Переиспользование функций для разных объектов
- ООП паттерны

**Плюсы:**
- Гибкость в использовании функций
- Динамическое связывание контекста

**Минусы:**
- Частый источник багов
- Неинтуитивное поведение
- Легко потерять контекст при передаче методов

**Решения проблем:**
- Стрелочные функции
- bind для привязки контекста
- Использование переменных (const self = this)

---

### 6. Что такое прототипное наследование?

**Как работает:**

В JavaScript объекты наследуют свойства и методы от других объектов через прототипы. Каждый объект имеет внутреннюю ссылку `[[Prototype]]` на другой объект.

```javascript
// Прототип
const animal = {
  eats: true,
  walk() {
    console.log('Animal walks');
  }
};

// Создание объекта с прототипом
const rabbit = Object.create(animal);
rabbit.jumps = true;

console.log(rabbit.eats); // true (из прототипа)
console.log(rabbit.jumps); // true (собственное свойство)

// Цепочка прототипов
console.log(rabbit.__proto__ === animal); // true
console.log(animal.__proto__ === Object.prototype); // true
console.log(Object.prototype.__proto__); // null
```

**Функции-конструкторы:**
```javascript
function User(name) {
  this.name = name;
}

User.prototype.greet = function() {
  console.log(`Hello, ${this.name}`);
};

const user = new User('Alice');
user.greet(); // 'Hello, Alice'
```

**Классы (синтаксический сахар):**
```javascript
class User {
  constructor(name) {
    this.name = name;
  }
  
  greet() {
    console.log(`Hello, ${this.name}`);
  }
}
```

**Для чего:**
- Переиспользование кода
- Создание иерархий объектов
- Экономия памяти (методы в прототипе, а не в каждом экземпляре)

**Плюсы:**
- Динамическое наследование (можно изменять прототип)
- Гибкость
- Экономия памяти

**Минусы:**
- Сложнее для понимания чем классовое наследование
- Медленнее доступ к свойствам в длинной цепочке прототипов
- Легко создать циклические зависимости

**Аналоги:**
- Классовое наследование в других языках
- Композиция вместо наследования (современный подход)

---

### 7. Разница между == и ===

**Как работает:**

**== (нестрогое сравнение):**
- Выполняет приведение типов перед сравнением
- Сложные правила преобразования

**=== (строгое сравнение):**
- Сравнивает значение и тип без приведения
- Более предсказуемое поведение

```javascript
// == примеры
5 == '5'        // true (string приводится к number)
0 == false      // true
null == undefined // true
'' == 0         // true

// === примеры
5 === '5'       // false (разные типы)
0 === false     // false
null === undefined // false
'' === 0        // false

// Особые случаи
NaN === NaN     // false (используйте Number.isNaN)
Object.is(NaN, NaN) // true

+0 === -0       // true
Object.is(+0, -0)   // false
```

**Для чего:**
- == — когда нужно проверить "пустоту" (null/undefined)
- === — везде остальное (default choice)

**Плюсы ===:**
- Предсказуемое поведение
- Нет скрытых преобразований типов
- Рекомендуется ESLint

**Минусы ==:**
- Сложные правила приведения
- Частый источник багов
- Плохо читается

**Рекомендация:**
Всегда используйте === и !==, кроме проверки на null/undefined:
```javascript
if (value == null) { // проверка на null или undefined
  // ...
}
```

---

### 8. Что такое деструктуризация?

**Как работает:**

Синтаксис для извлечения значений из массивов или свойств из объектов в отдельные переменные.

**Деструктуризация объектов:**
```javascript
const user = {
  name: 'Alice',
  age: 30,
  email: 'alice@example.com'
};

// Базовая
const { name, age } = user;

// С переименованием
const { name: userName, age: userAge } = user;

// Со значениями по умолчанию
const { name, role = 'user' } = user;

// Вложенная
const { address: { city } } = user;

// Rest оператор
const { name, ...rest } = user;
```

**Деструктуризация массивов:**
```javascript
const numbers = [1, 2, 3, 4, 5];

// Базовая
const [first, second] = numbers;

// Пропуск элементов
const [first, , third] = numbers;

// Rest оператор
const [first, ...rest] = numbers;

// Со значениями по умолчанию
const [a = 0, b = 0] = [];

// Обмен значений
let x = 1, y = 2;
[x, y] = [y, x];
```

**В параметрах функций:**
```javascript
function greet({ name, age = 18 }) {
  console.log(`${name}, ${age}`);
}

function sum([a, b]) {
  return a + b;
}
```

**Для чего:**
- Более читаемый код
- Извлечение нужных свойств
- Работа с возвращаемыми значениями
- Параметры функций

**Плюсы:**
- Чистый и лаконичный синтаксис
- Меньше повторений кода
- Явное указание используемых свойств

**Минусы:**
- Может усложнить читаемость при глубокой вложенности
- Дополнительные переменные в scope

---

### 9. Что такое spread и rest операторы?

**Как работает:**

**Spread (...)** — разворачивает элементы массива/объекта:

```javascript
// Массивы
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const combined = [...arr1, ...arr2]; // [1, 2, 3, 4, 5, 6]

// Копирование массива
const copy = [...arr1]; // поверхностная копия

// Передача аргументов
Math.max(...arr1); // Math.max(1, 2, 3)

// Объекты
const user = { name: 'Alice', age: 30 };
const updated = { ...user, age: 31 }; // перезапись свойства
const merged = { ...user, role: 'admin' }; // добавление свойства

// Приоритет (последний выигрывает)
const result = { ...obj1, ...obj2 }; // obj2 перезапишет общие ключи
```

**Rest (...)** — собирает элементы в массив:

```javascript
// В параметрах функции
function sum(...numbers) {
  return numbers.reduce((a, b) => a + b, 0);
}
sum(1, 2, 3, 4); // 10

// В деструктуризации массива
const [first, ...rest] = [1, 2, 3, 4, 5];
// first = 1, rest = [2, 3, 4, 5]

// В деструктуризации объекта
const { name, ...otherProps } = { name: 'Alice', age: 30, role: 'admin' };
// name = 'Alice', otherProps = { age: 30, role: 'admin' }
```

**Для чего:**
- Spread: копирование, слияние, передача аргументов
- Rest: сбор оставшихся элементов, произвольное количество аргументов

**Плюсы:**
- Чистый и читаемый синтаксис
- Иммутабельные операции
- Гибкость

**Минусы:**
- Поверхностное копирование (shallow copy)
- Может быть неэффективно для больших объектов
- Порядок имеет значение при слиянии

---

### 10. Map, Set, WeakMap, WeakSet — в чем разница?

**Map:**
```javascript
const map = new Map();
map.set('key1', 'value1');
map.set({ id: 1 }, 'object key');
map.get('key1'); // 'value1'
map.has('key1'); // true
map.size; // 2
map.delete('key1');
map.clear();

// Итерация
for (const [key, value] of map) {
  console.log(key, value);
}
```

**Особенности:**
- Любые типы ключей (объекты, функции, примитивы)
- Сохраняет порядок вставки
- Итерируемый
- Есть size

**Set:**
```javascript
const set = new Set([1, 2, 3, 3]); // [1, 2, 3] - дубликаты удаляются
set.add(4);
set.has(2); // true
set.delete(1);
set.size; // 3

// Итерация
for (const value of set) {
  console.log(value);
}
```

**Особенности:**
- Уникальные значения
- Любые типы значений
- Итерируемый
- Есть size

**WeakMap:**
```javascript
const weakMap = new WeakMap();
let obj = { id: 1 };
weakMap.set(obj, 'metadata');
weakMap.get(obj); // 'metadata'

obj = null; // объект будет удален сборщиком мусора вместе с записью в WeakMap
```

**Особенности:**
- Только объекты как ключи
- Слабые ссылки (не препятствует garbage collection)
- Не итерируемый
- Нет size, clear()

**WeakSet:**
```javascript
const weakSet = new WeakSet();
let obj = { id: 1 };
weakSet.add(obj);
weakSet.has(obj); // true

obj = null; // объект будет удален сборщиком мусора
```

**Особенности:**
- Только объекты
- Слабые ссылки
- Не итерируемый
- Нет size

**Для чего:**
- **Map:** когда нужны не-строковые ключи, порядок важен, или частые добавления/удаления
- **Set:** уникальные значения, проверка принадлежности
- **WeakMap:** хранение метаданных об объектах без утечек памяти (private данные, кэши)
- **WeakSet:** отслеживание объектов без препятствия GC

**Плюсы Map/Set:**
- Производительность для частых операций
- Любые типы ключей/значений
- Встроенные методы

**Плюсы WeakMap/WeakSet:**
- Автоматическая очистка памяти
- Приватность данных

**Минусы WeakMap/WeakSet:**
- Нельзя итерировать
- Нет информации о размере
- Только объекты

---

## TypeScript

### 11. Основные типы в TypeScript

**Как работает:**

TypeScript добавляет статическую типизацию к JavaScript.

**Примитивные типы:**
```typescript
// Базовые
let name: string = 'Alice';
let age: number = 30;
let isActive: boolean = true;
let nothing: null = null;
let notDefined: undefined = undefined;

// Специальные
let anyValue: any = 'anything'; // отключает проверку типов
let unknownValue: unknown = 'something'; // безопасная версия any
let neverReturns: never; // функция никогда не возвращает значение

// Symbol и BigInt
let sym: symbol = Symbol('key');
let bigNum: bigint = 100n;
```

**Сложные типы:**
```typescript
// Массивы
let numbers: number[] = [1, 2, 3];
let strings: Array<string> = ['a', 'b', 'c'];

// Кортежи (Tuples)
let tuple: [string, number] = ['Alice', 30];

// Объекты
let user: { name: string; age: number } = {
  name: 'Alice',
  age: 30
};

// Функции
let greet: (name: string) => string;
greet = (name) => `Hello, ${name}`;

// Union типы
let value: string | number = 'text';
value = 42; // OK

// Intersection типы
type Person = { name: string };
type Employee = { employeeId: number };
type Staff = Person & Employee;

// Literal типы
let status: 'pending' | 'approved' | 'rejected';
status = 'pending'; // OK
```

**Для чего:**
- Ранее обнаружение ошибок
- Автодополнение в IDE
- Документирование кода
- Безопасный рефакторинг

**Плюсы:**
- Меньше runtime ошибок
- Лучшая поддержка IDE
- Самодокументирующийся код
- Упрощает рефакторинг

**Минусы:**
- Дополнительный код
- Кривая обучения
- Замедление разработки (на начальном этапе)

---

### 12. Interface vs Type

**Как работает:**

**Interface:**
```typescript
interface User {
  name: string;
  age: number;
}

// Расширение (extends)
interface Admin extends User {
  role: string;
}

// Declaration merging (слияние деклараций)
interface User {
  email: string; // добавляется к существующему User
}

// Только для объектов
interface Point {
  x: number;
  y: number;
}
```

**Type:**
```typescript
type User = {
  name: string;
  age: number;
};

// Пересечение (intersection)
type Admin = User & {
  role: string;
};

// Union типы
type Status = 'active' | 'inactive';

// Примитивы, кортежи, функции
type ID = string | number;
type Tuple = [string, number];
type Callback = (data: string) => void;

// Mapped types
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

// Conditional types
type NonNullable<T> = T extends null | undefined ? never : T;
```

**Для чего:**
- Interface: определение структуры объектов, классов, контрактов API
- Type: всё остальное (union, primitives, utility types)

**Когда использовать Interface:**
- Описание формы объектов
- Когда нужно расширение через extends
- Публичное API библиотек (благодаря declaration merging)

**Когда использовать Type:**
- Union или intersection типы
- Примитивы и literal типы
- Кортежи, функции
- Сложные типы с условиями

**Плюсы Interface:**
- Declaration merging
- Более понятный extends
- Лучше для ООП паттернов

**Плюсы Type:**
- Более гибкий
- Поддержка любых типов
- Более мощные возможности (conditional, mapped types)

**Рекомендация:**
Используйте interface для объектов по умолчанию, type для остального. Главное — консистентность в проекте.

---

### 13. Generics (дженерики)

**Как работает:**

Generics позволяют создавать переиспользуемые компоненты, которые работают с разными типами, сохраняя type safety.

```typescript
// Базовый пример
function identity<T>(value: T): T {
  return value;
}

identity<string>('hello'); // string
identity<number>(42); // number
identity('hello'); // TypeScript выведет тип автоматически

// Массивы
function firstElement<T>(arr: T[]): T | undefined {
  return arr[0];
}

const num = firstElement([1, 2, 3]); // number | undefined
const str = firstElement(['a', 'b']); // string | undefined

// Множественные типы
function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}

pair<string, number>('age', 30); // [string, number]

// С ограничениями (constraints)
interface HasLength {
  length: number;
}

function logLength<T extends HasLength>(item: T): void {
  console.log(item.length);
}

logLength('hello'); // OK
logLength([1, 2, 3]); // OK
logLength(42); // Error: number не имеет length

// Generic классы
class Box<T> {
  private content: T;
  
  constructor(content: T) {
    this.content = content;
  }
  
  getContent(): T {
    return this.content;
  }
}

const stringBox = new Box<string>('hello');
const numberBox = new Box<number>(42);

// Generic интерфейсы
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

const userResponse: ApiResponse<User> = {
  data: { name: 'Alice', age: 30 },
  status: 200,
  message: 'Success'
};
```

**Для чего:**
- Переиспользуемые компоненты
- Type-safe коллекции и структуры данных
- Generic функции утилиты
- API клиенты с типизированными ответами

**Плюсы:**
- Переиспользование кода с сохранением типов
- Type safety без дублирования
- Гибкость
- Самодокументирующийся код

**Минусы:**
- Сложность для понимания
- Может усложнить читаемость
- Verbose синтаксис

**Практические примеры:**
```typescript
// Утилиты
function map<T, U>(arr: T[], fn: (item: T) => U): U[] {
  return arr.map(fn);
}

// API клиент
class ApiClient {
  async get<T>(url: string): Promise<ApiResponse<T>> {
    const response = await fetch(url);
    return response.json();
  }
}

const api = new ApiClient();
const users = await api.get<User[]>('/users');
```

---

### 14. Utility Types в TypeScript

**Как работает:**

TypeScript предоставляет встроенные utility types для трансформации типов.

**Partial<T>** — делает все свойства опциональными:
```typescript
interface User {
  name: string;
  age: number;
  email: string;
}

type PartialUser = Partial<User>;
// { name?: string; age?: number; email?: string; }

function updateUser(user: User, updates: Partial<User>): User {
  return { ...user, ...updates };
}
```

**Required<T>** — делает все свойства обязательными:
```typescript
interface Config {
  host?: string;
  port?: number;
}

type RequiredConfig = Required<Config>;
// { host: string; port: number; }
```

**Readonly<T>** — делает все свойства readonly:
```typescript
type ReadonlyUser = Readonly<User>;
// { readonly name: string; readonly age: number; ... }

const user: ReadonlyUser = { name: 'Alice', age: 30, email: 'a@a.com' };
user.name = 'Bob'; // Error
```

**Pick<T, K>** — выбирает только указанные свойства:
```typescript
type UserPreview = Pick<User, 'name' | 'email'>;
// { name: string; email: string; }
```

**Omit<T, K>** — исключает указанные свойства:
```typescript
type UserWithoutEmail = Omit<User, 'email'>;
// { name: string; age: number; }
```

**Record<K, T>** — создает тип с ключами K и значениями T:
```typescript
type Roles = 'admin' | 'user' | 'guest';
type RolePermissions = Record<Roles, string[]>;
// { admin: string[]; user: string[]; guest: string[]; }

const permissions: RolePermissions = {
  admin: ['read', 'write', 'delete'],
  user: ['read', 'write'],
  guest: ['read']
};
```

**Exclude<T, U>** — исключает типы из union:
```typescript
type T1 = Exclude<'a' | 'b' | 'c', 'a'>; // 'b' | 'c'
type T2 = Exclude<string | number | boolean, boolean>; // string | number
```

**Extract<T, U>** — извлекает типы из union:
```typescript
type T1 = Extract<'a' | 'b' | 'c', 'a' | 'f'>; // 'a'
```

**NonNullable<T>** — исключает null и undefined:
```typescript
type T1 = NonNullable<string | number | null | undefined>; // string | number
```

**ReturnType<T>** — получает тип возвращаемого значения функции:
```typescript
function getUser() {
  return { name: 'Alice', age: 30 };
}

type User = ReturnType<typeof getUser>;
// { name: string; age: number; }
```

**Parameters<T>** — получает типы параметров функции:
```typescript
function createUser(name: string, age: number) {}

type Params = Parameters<typeof createUser>;
// [string, number]
```

**Awaited<T>** — разворачивает Promise:
```typescript
type A = Awaited<Promise<string>>; // string
type B = Awaited<Promise<Promise<number>>>; // number
```

**Для чего:**
- Трансформация типов без дублирования
- Работа с API и формами
- Создание производных типов
- Повышение переиспользуемости

**Плюсы:**
- DRY принцип для типов
- Мощные композиции типов
- Стандартные инструменты

**Минусы:**
- Может усложнить читаемость
- Требует понимания TypeScript механик

---

### 15. Type Guards и Type Narrowing

**Как работает:**

Type narrowing — процесс уточнения типа переменной в определенной области кода.

**typeof guards:**
```typescript
function process(value: string | number) {
  if (typeof value === 'string') {
    console.log(value.toUpperCase()); // value: string
  } else {
    console.log(value.toFixed(2)); // value: number
  }
}
```

**instanceof guards:**
```typescript
class Dog {
  bark() {}
}

class Cat {
  meow() {}
}

function makeSound(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    animal.bark(); // animal: Dog
  } else {
    animal.meow(); // animal: Cat
  }
}
```

**in operator:**
```typescript
interface Car {
  drive(): void;
}

interface Boat {
  sail(): void;
}

function move(vehicle: Car | Boat) {
  if ('drive' in vehicle) {
    vehicle.drive(); // vehicle: Car
  } else {
    vehicle.sail(); // vehicle: Boat
  }
}
```

**Custom type guards (предикаты):**
```typescript
interface User {
  name: string;
  role: 'user';
}

interface Admin {
  name: string;
  role: 'admin';
  permissions: string[];
}

// Type predicate
function isAdmin(user: User | Admin): user is Admin {
  return user.role === 'admin';
}

function doSomething(user: User | Admin) {
  if (isAdmin(user)) {
    console.log(user.permissions); // user: Admin
  } else {
    console.log(user.name); // user: User
  }
}
```

**Discriminated unions:**
```typescript
interface SuccessResponse {
  status: 'success';
  data: any;
}

interface ErrorResponse {
  status: 'error';
  error: string;
}

type ApiResponse = SuccessResponse | ErrorResponse;

function handleResponse(response: ApiResponse) {
  switch (response.status) {
    case 'success':
      console.log(response.data); // response: SuccessResponse
      break;
    case 'error':
      console.log(response.error); // response: ErrorResponse
      break;
  }
}
```

**Truthiness narrowing:**
```typescript
function printName(name: string | null | undefined) {
  if (name) {
    console.log(name.toUpperCase()); // name: string
  }
}
```

**Non-null assertion operator (!):**
```typescript
function processValue(value: string | null) {
  console.log(value!.toUpperCase()); // говорим TS что value точно не null
}
```

**Для чего:**
- Безопасная работа с union типами
- Уточнение типов в условиях
- Обработка разных типов данных

**Плюсы:**
- Type safety в runtime логике
- Избегание type assertions
- Читаемый код

**Минусы:**
- Дополнительные проверки
- Может быть verbose

---

## React

### 16. Жизненный цикл компонента (class components)

**Как работает:**

**Mounting (монтирование):**
1. **constructor()** — инициализация state, bind методов
2. **static getDerivedStateFromProps()** — синхронизация state с props
3. **render()** — возвращает JSX
4. **componentDidMount()** — компонент добавлен в DOM (запросы, подписки)

**Updating (обновление):**
1. **static getDerivedStateFromProps()**
2. **shouldComponentUpdate()** — оптимизация, нужен ли ререндер
3. **render()**
4. **getSnapshotBeforeUpdate()** — перед применением изменений к DOM
5. **componentDidUpdate()** — после обновления DOM

**Unmounting (размонтирование):**
1. **componentWillUnmount()** — очистка (отписки, таймеры)

```typescript
class UserProfile extends React.Component<Props, State> {
  timerId?: number;
  
  constructor(props: Props) {
    super(props);
    this.state = { user: null, loading: true };
  }
  
  componentDidMount() {
    // Запросы, подписки, таймеры
    this.fetchUser();
    this.timerId = setInterval(() => {
      this.fetchUser();
    }, 60000);
  }
  
  componentDidUpdate(prevProps: Props, prevState: State) {
    // Реакция на изменения props/state
    if (prevProps.userId !== this.props.userId) {
      this.fetchUser();
    }
  }
  
  componentWillUnmount() {
    // Очистка
    if (this.timerId) {
      clearInterval(this.timerId);
    }
  }
  
  shouldComponentUpdate(nextProps: Props, nextState: State) {
    // Оптимизация ререндеров
    return nextProps.userId !== this.props.userId ||
           nextState.user !== this.state.user;
  }
  
  render() {
    return <div>{/* JSX */}</div>;
  }
}
```

**Для чего:**
- Управление side-effects
- Оптимизация производительности
- Работа с внешними библиотеками

**Плюсы:**
- Явный контроль над жизненным циклом
- Предсказуемая последовательность вызовов

**Минусы:**
- Verbose код
- Легко сделать ошибку (забыть отписку)
- Дублирование логики между методами

**Аналоги:**
- Функциональные компоненты с хуками (современный подход)

---

### 17. Hooks: useState, useEffect, useContext

**useState:**
```typescript
const [count, setCount] = useState<number>(0);
const [user, setUser] = useState<User | null>(null);

// Функциональное обновление
setCount(prev => prev + 1);

// Ленивая инициализация (для дорогих вычислений)
const [data, setData] = useState(() => {
  return expensiveCalculation();
});
```

**Как работает:**
- Сохраняет состояние между ререндерами
- Возвращает пару [значение, функция обновления]
- Обновление асинхронное (батчится)

**useEffect:**
```typescript
// Mount + каждое обновление
useEffect(() => {
  console.log('После каждого рендера');
});

// Mount + когда меняются зависимости
useEffect(() => {
  fetchData(userId);
}, [userId]);

// Только mount (пустой массив зависимостей)
useEffect(() => {
  console.log('Только при монтировании');
}, []);

// С cleanup функцией
useEffect(() => {
  const subscription = api.subscribe(userId);
  
  return () => {
    subscription.unsubscribe(); // очистка
  };
}, [userId]);

// Множественные эффекты (разделение логики)
useEffect(() => {
  // Эффект 1
}, [dep1]);

useEffect(() => {
  // Эффект 2
}, [dep2]);
```

**Как работает:**
- Выполняется после рендера (не блокирует отрисовку)
- Cleanup функция вызывается перед следующим эффектом и при unmount
- Зависимости определяют когда эффект перезапускается

**useContext:**
```typescript
// Создание контекста
const ThemeContext = createContext<'light' | 'dark'>('light');

// Provider
function App() {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  
  return (
    <ThemeContext.Provider value={theme}>
      <Toolbar />
    </ThemeContext.Provider>
  );
}

// Потребление контекста
function Button() {
  const theme = useContext(ThemeContext);
  
  return (
    <button className={theme === 'dark' ? 'dark-btn' : 'light-btn'}>
      Click me
    </button>
  );
}
```

**Как работает:**
- Позволяет компонентам подписаться на контекст
- Ререндер при изменении значения контекста
- Не требует prop drilling

**Для чего:**
- useState: локальное состояние компонента
- useEffect: side-effects (запросы, подписки, DOM манипуляции)
- useContext: глобальное состояние, темы, локализация

**Плюсы:**
- Чище чем class components
- Легче переиспользовать логику (custom hooks)
- Меньше кода
- Функциональный подход

**Минусы useState:**
- Асинхронные обновления могут быть неочевидны
- Нужно помнить про функциональные обновления

**Минусы useEffect:**
- Легко забыть зависимости (ESLint помогает)
- Может вызывать лишние эффекты
- Сложно отлаживать

**Минусы useContext:**
- Ререндерит все компоненты использующие контекст
- Нет встроенной оптимизации

---

### 18. useMemo и useCallback

**useMemo:**
```typescript
const expensiveValue = useMemo(() => {
  return computeExpensiveValue(a, b);
}, [a, b]); // Пересчитывается только при изменении a или b

// Практический пример
function TodoList({ todos, filter }: Props) {
  const filteredTodos = useMemo(() => {
    console.log('Фильтруем todos');
    return todos.filter(todo => todo.status === filter);
  }, [todos, filter]);
  
  return (
    <ul>
      {filteredTodos.map(todo => (
        <TodoItem key={todo.id} todo={todo} />
      ))}
    </ul>
  );
}
```

**Как работает:**
- Мемоизирует (кэширует) результат вычисления
- Пересчитывается только при изменении зависимостей
- Возвращает мемоизированное значение

**useCallback:**
```typescript
const handleClick = useCallback(() => {
  console.log('Clicked with', userId);
  doSomething(userId);
}, [userId]); // Функция пересоздается только при изменении userId

// Практический пример
function Parent() {
  const [count, setCount] = useState(0);
  
  // Без useCallback — новая функция каждый рендер
  const handleClick = () => {
    console.log('Clicked');
  };
  
  // С useCallback — та же функция между рендерами
  const memoizedHandleClick = useCallback(() => {
    console.log('Clicked');
  }, []);
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <Child onClick={memoizedHandleClick} />
    </div>
  );
}

const Child = React.memo(({ onClick }: { onClick: () => void }) => {
  console.log('Child render');
  return <button onClick={onClick}>Click me</button>;
});
```

**Как работает:**
- Мемоизирует функцию
- Возвращает ту же ссылку на функцию между рендерами
- Эквивалентен `useMemo(() => fn, deps)`

**Для чего:**
- useMemo: оптимизация дорогих вычислений
- useCallback: предотвращение ненужных ререндеров дочерних компонентов

**Когда использовать:**
- Дорогие вычисления в useMemo
- Передача колбэков в React.memo компоненты
- Зависимости в useEffect, useMemo, useCallback
- Работа с референсиями в useEffect

**Когда НЕ использовать:**
- Для каждой функции (оверинжиниринг)
- Для простых вычислений (мемоизация дороже)
- Без замеров производительности

**Плюсы:**
- Оптимизация производительности
- Предотвращение лишних ререндеров
- Стабильные ссылки

**Минусы:**
- Дополнительная сложность
- Потребление памяти для кэша
- Может навредить если использовать везде
- Читаемость кода

**Важно:**
```typescript
// Плохо — преждевременная оптимизация
const value = useMemo(() => x + y, [x, y]); // Простое сложение

// Хорошо
const filteredList = useMemo(() => {
  return hugeList.filter(item => item.category === category);
}, [hugeList, category]); // Дорогая операция
```

---

### 19. useRef и forwardRef

**useRef:**
```typescript
// 1. Доступ к DOM элементам
function TextInput() {
  const inputRef = useRef<HTMLInputElement>(null);
  
  const focusInput = () => {
    inputRef.current?.focus();
  };
  
  return (
    <>
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>Focus</button>
    </>
  );
}

// 2. Хранение мутабельных значений (не вызывает ререндер)
function Timer() {
  const intervalRef = useRef<number | null>(null);
  const [count, setCount] = useState(0);
  
  const startTimer = () => {
    intervalRef.current = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
  };
  
  const stopTimer = () => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
    }
  };
  
  useEffect(() => {
    return () => stopTimer(); // cleanup
  }, []);
  
  return (
    <div>
      <div>Count: {count}</div>
      <button onClick={startTimer}>Start</button>
      <button onClick={stopTimer}>Stop</button>
    </div>
  );
}

// 3. Хранение предыдущего значения
function usePrevious<T>(value: T): T | undefined {
  const ref = useRef<T>();
  
  useEffect(() => {
    ref.current = value;
  }, [value]);
  
  return ref.current;
}

function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);
  
  return (
    <div>
      Now: {count}, Before: {prevCount}
    </div>
  );
}
```

**forwardRef:**
```typescript
// Передача ref в дочерний компонент
interface InputProps {
  label: string;
}

const FancyInput = forwardRef<HTMLInputElement, InputProps>((props, ref) => {
  return (
    <div>
      <label>{props.label}</label>
      <input ref={ref} />
    </div>
  );
});

// Использование
function Parent() {
  const inputRef = useRef<HTMLInputElement>(null);
  
  const focusInput = () => {
    inputRef.current?.focus();
  };
  
  return (
    <>
      <FancyInput ref={inputRef} label="Name" />
      <button onClick={focusInput}>Focus</button>
    </>
  );
}

// useImperativeHandle — кастомизация ref
interface CustomInputRef {
  focus: () => void;
  clear: () => void;
}

const CustomInput = forwardRef<CustomInputRef, InputProps>((props, ref) => {
  const inputRef = useRef<HTMLInputElement>(null);
  
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current?.focus();
    },
    clear: () => {
      if (inputRef.current) {
        inputRef.current.value = '';
      }
    }
  }));
  
  return <input ref={inputRef} />;
});

// Использование
function Parent() {
  const inputRef = useRef<CustomInputRef>(null);
  
  const handleClear = () => {
    inputRef.current?.clear();
  };
  
  return (
    <>
      <CustomInput ref={inputRef} label="Name" />
      <button onClick={handleClear}>Clear</button>
    </>
  );
}
```

**Как работает:**
- useRef возвращает мутабельный объект { current: value }
- Изменение .current не вызывает ререндер
- Сохраняется между рендерами
- forwardRef позволяет передавать ref дочерним компонентам

**Для чего:**
- Доступ к DOM элементам
- Хранение значений без ререндера (таймеры, подписки, предыдущие значения)
- Интеграция с неReact библиотеками
- Императивная работа с дочерними компонентами

**Плюсы:**
- Прямой доступ к DOM
- Не вызывает лишних ререндеров
- Хранение мутабельных значений

**Минусы:**
- Императивный подход (не декларативный)
- Можно злоупотребить (лучше использовать state где возможно)
- forwardRef усложняет код

---

### 20. React.memo, PureComponent

**React.memo:**
```typescript
// Базовое использование
const ExpensiveComponent = React.memo(({ data }: { data: Data }) => {
  console.log('Render');
  return <div>{/* JSX */}</div>;
});

// С кастомной функцией сравнения
const CustomMemoComponent = React.memo(
  ({ user }: { user: User }) => {
    return <div>{user.name}</div>;
  },
  (prevProps, nextProps) => {
    // Вернуть true если НЕ нужен ререндер
    return prevProps.user.id === nextProps.user.id;
  }
);

// Практический пример
interface ListItemProps {
  item: Item;
  onDelete: (id: string) => void;
}

const ListItem = React.memo(({ item, onDelete }: ListItemProps) => {
  console.log(`Render item ${item.id}`);
  
  return (
    <div>
      <span>{item.name}</span>
      <button onClick={() => onDelete(item.id)}>Delete</button>
    </div>
  );
});

function List() {
  const [items, setItems] = useState<Item[]>([]);
  const [filter, setFilter] = useState('');
  
  // Без useCallback — новая функция каждый рендер
  // ListItem будет ререндериться даже при изменении filter
  const handleDelete = useCallback((id: string) => {
    setItems(items => items.filter(item => item.id !== id));
  }, []);
  
  return (
    <div>
      <input value={filter} onChange={e => setFilter(e.target.value)} />
      {items.map(item => (
        <ListItem
          key={item.id}
          item={item}
          onDelete={handleDelete}
        />
      ))}
    </div>
  );
}
```

**PureComponent (class components):**
```typescript
class ListItem extends React.PureComponent<Props> {
  render() {
    console.log('Render');
    return <div>{/* JSX */}</div>;
  }
}

// Эквивалентно Component с shouldComponentUpdate
class ManualOptimization extends React.Component<Props> {
  shouldComponentUpdate(nextProps: Props) {
    return !shallowEqual(this.props, nextProps);
  }
  
  render() {
    return <div>{/* JSX */}</div>;
  }
}
```

**Как работает:**
- React.memo оборачивает компонент и мемоизирует результат
- Поверхностное сравнение props (shallow comparison)
- Ререндер только если props изменились
- PureComponent делает то же для class components

**Shallow comparison:**
```javascript
// Поверхностное сравнение
{ a: 1, b: 2 } === { a: 1, b: 2 } // false (разные ссылки)

// React.memo сравнивает так:
prevProps.a === nextProps.a && prevProps.b === nextProps.b // true

// Проблема с объектами/массивами
const obj = { value: 1 };
prevProps.data === { value: 1 } // false (новая ссылка)
```

**Для чего:**
- Оптимизация компонентов с дорогим рендерингом
- Большие списки
- Компоненты с частыми обновлениями родителя

**Когда использовать:**
- Компонент часто ререндерится с теми же props
- Рендеринг компонента дорогой
- Замерили и убедились в проблеме

**Когда НЕ использовать:**
- Для каждого компонента (преждевременная оптимизация)
- Если props почти всегда разные
- Для простых компонентов

**Плюсы:**
- Предотвращает лишние ререндеры
- Легко применить
- Автоматическая оптимизация

**Минусы:**
- Оверхэд на сравнение props
- Ложное чувство оптимизации
- Работает только с примитивами в props (проблема со ссылками)

**Частые ошибки:**
```typescript
// Плохо — новый объект каждый рендер
<Component style={{ color: 'red' }} />

// Хорошо
const style = { color: 'red' };
<Component style={style} />

// Плохо — новая функция каждый рендер
<Component onClick={() => console.log('click')} />

// Хорошо
const handleClick = useCallback(() => console.log('click'), []);
<Component onClick={handleClick} />

// Плохо — новый массив каждый рендер
<Component items={items.filter(x => x.active)} />

// Хорошо
const activeItems = useMemo(
  () => items.filter(x => x.active),
  [items]
);
<Component items={activeItems} />
```

---

### 21. Custom Hooks

**Как работает:**

Custom hooks — функции, которые используют встроенные хуки для инкапсуляции и переиспользования логики.

**Правила:**
- Название начинается с "use"
- Могут вызывать другие хуки
- Подчиняются Rules of Hooks

**Примеры:**

```typescript
// 1. useLocalStorage — синхронизация с localStorage
function useLocalStorage<T>(
  key: string,
  initialValue: T
): [T, (value: T) => void] {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });
  
  const setValue = (value: T) => {
    try {
      setStoredValue(value);
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error(error);
    }
  };
  
  return [storedValue, setValue];
}

// Использование
function App() {
  const [name, setName] = useLocalStorage('name', 'Guest');
  return <input value={name} onChange={e => setName(e.target.value)} />;
}

// 2. useFetch — загрузка данных
interface FetchState<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
}

function useFetch<T>(url: string): FetchState<T> {
  const [state, setState] = useState<FetchState<T>>({
    data: null,
    loading: true,
    error: null
  });
  
  useEffect(() => {
    let cancelled = false;
    
    const fetchData = async () => {
      try {
        const response = await fetch(url);
        const data = await response.json();
        
        if (!cancelled) {
          setState({ data, loading: false, error: null });
        }
      } catch (error) {
        if (!cancelled) {
          setState({ data: null, loading: false, error: error as Error });
        }
      }
    };
    
    fetchData();
    
    return () => {
      cancelled = true;
    };
  }, [url]);
  
  return state;
}

// Использование
function UserProfile({ userId }: { userId: string }) {
  const { data, loading, error } = useFetch<User>(`/api/users/${userId}`);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  return <div>{data?.name}</div>;
}

// 3. useDebounce — отложенное обновление
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);
  
  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);
  
  return debouncedValue;
}

// Использование
function SearchComponent() {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 500);
  
  useEffect(() => {
    if (debouncedSearchTerm) {
      // API запрос только после 500ms без изменений
      searchAPI(debouncedSearchTerm);
    }
  }, [debouncedSearchTerm]);
  
  return (
    <input
      value={searchTerm}
      onChange={e => setSearchTerm(e.target.value)}
    />
  );
}

// 4. useWindowSize — размеры окна
function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });
  
  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };
    
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  return size;
}

// 5. useOnClickOutside — клик вне элемента
function useOnClickOutside<T extends HTMLElement>(
  ref: React.RefObject<T>,
  handler: () => void
) {
  useEffect(() => {
    const listener = (event: MouseEvent | TouchEvent) => {
      if (!ref.current || ref.current.contains(event.target as Node)) {
        return;
      }
      handler();
    };
    
    document.addEventListener('mousedown', listener);
    document.addEventListener('touchstart', listener);
    
    return () => {
      document.removeEventListener('mousedown', listener);
      document.removeEventListener('touchstart', listener);
    };
  }, [ref, handler]);
}

// Использование
function Modal() {
  const [isOpen, setIsOpen] = useState(false);
  const modalRef = useRef<HTMLDivElement>(null);
  
  useOnClickOutside(modalRef, () => setIsOpen(false));
  
  return isOpen ? (
    <div ref={modalRef}>
      Modal content
    </div>
  ) : null;
}

// 6. useForm — управление формами
interface FormState<T> {
  values: T;
  errors: Partial<Record<keyof T, string>>;
  handleChange: (name: keyof T) => (e: React.ChangeEvent<HTMLInputElement>) => void;
  handleSubmit: (callback: (values: T) => void) => (e: React.FormEvent) => void;
  reset: () => void;
}

function useForm<T extends Record<string, any>>(
  initialValues: T,
  validate?: (values: T) => Partial<Record<keyof T, string>>
): FormState<T> {
  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});
  
  const handleChange = (name: keyof T) => (
    e: React.ChangeEvent<HTMLInputElement>
  ) => {
    setValues({ ...values, [name]: e.target.value });
  };
  
  const handleSubmit = (callback: (values: T) => void) => (
    e: React.FormEvent
  ) => {
    e.preventDefault();
    
    if (validate) {
      const validationErrors = validate(values);
      setErrors(validationErrors);
      
      if (Object.keys(validationErrors).length === 0) {
        callback(values);
      }
    } else {
      callback(values);
    }
  };
  
  const reset = () => {
    setValues(initialValues);
    setErrors({});
  };
  
  return { values, errors, handleChange, handleSubmit, reset };
}

// Использование
function LoginForm() {
  const { values, errors, handleChange, handleSubmit } = useForm(
    { email: '', password: '' },
    (values) => {
      const errors: any = {};
      if (!values.email) errors.email = 'Required';
      if (!values.password) errors.password = 'Required';
      return errors;
    }
  );
  
  const onSubmit = (values: typeof values) => {
    console.log('Submit:', values);
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        value={values.email}
        onChange={handleChange('email')}
      />
      {errors.email && <span>{errors.email}</span>}
      
      <input
        type="password"
        value={values.password}
        onChange={handleChange('password')}
      />
      {errors.password && <span>{errors.password}</span>}
      
      <button type="submit">Login</button>
    </form>
  );
}
```

**Для чего:**
- Переиспользование логики между компонентами
- Разделение concerns
- Упрощение компонентов
- Создание библиотек утилит

**Плюсы:**
- DRY принцип
- Композиция вместо наследования
- Легко тестировать
- Чистые компоненты

**Минусы:**
- Может усложнить отладку
- Абстракция может скрывать сложность
- Чрезмерное использование затрудняет понимание

---

### 22. Context API и управление состоянием

**Как работает:**

Context API позволяет передавать данные через дерево компонентов без prop drilling.

**Базовое использование:**
```typescript
// Создание контекста
interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

// Provider
function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// Custom hook для использования контекста
function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}

// Использование
function Button() {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <button
      className={theme === 'dark' ? 'dark-btn' : 'light-btn'}
      onClick={toggleTheme}
    >
      Toggle Theme
    </button>
  );
}

function App() {
  return (
    <ThemeProvider>
      <Button />
    </ThemeProvider>
  );
}
```

**Сложный пример с reducer:**
```typescript
// State и Actions
interface User {
  id: string;
  name: string;
  email: string;
}

interface AuthState {
  user: User | null;
  loading: boolean;
  error: string | null;
}

type AuthAction =
  | { type: 'LOGIN_START' }
  | { type: 'LOGIN_SUCCESS'; payload: User }
  | { type: 'LOGIN_ERROR'; payload: string }
  | { type: 'LOGOUT' };

// Reducer
function authReducer(state: AuthState, action: AuthAction): AuthState {
  switch (action.type) {
    case 'LOGIN_START':
      return { ...state, loading: true, error: null };
    case 'LOGIN_SUCCESS':
      return { user: action.payload, loading: false, error: null };
    case 'LOGIN_ERROR':
      return { user: null, loading: false, error: action.payload };
    case 'LOGOUT':
      return { user: null, loading: false, error: null };
    default:
      return state;
  }
}

// Context
interface AuthContextType extends AuthState {
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

// Provider
function AuthProvider({ children }: { children: React.ReactNode }) {
  const [state, dispatch] = useReducer(authReducer, {
    user: null,
    loading: false,
    error: null
  });
  
  const login = async (email: string, password: string) => {
    dispatch({ type: 'LOGIN_START' });
    
    try {
      const user = await api.login(email, password);
      dispatch({ type: 'LOGIN_SUCCESS', payload: user });
    } catch (error) {
      dispatch({ type: 'LOGIN_ERROR', payload: error.message });
    }
  };
  
  const logout = () => {
    dispatch({ type: 'LOGOUT' });
  };
  
  return (
    <AuthContext.Provider value={{ ...state, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

// Hook
function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}

// Использование
function LoginForm() {
  const { login, loading, error } = useAuth();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    await login(email, password);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input value={email} onChange={e => setEmail(e.target.value)} />
      <input type="password" value={password} onChange={e => setPassword(e.target.value)} />
      <button disabled={loading}>Login</button>
      {error && <div>{error}</div>}
    </form>
  );
}
```

**Оптимизация Context:**

```typescript
// Проблема: каждое изменение контекста ререндерит всех потребителей
// Решение 1: Разделение контекстов
const ThemeStateContext = createContext<'light' | 'dark'>('light');
const ThemeDispatchContext = createContext<(() => void) | undefined>(undefined);

function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  const toggleTheme = useCallback(() => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  }, []);
  
  return (
    <ThemeStateContext.Provider value={theme}>
      <ThemeDispatchContext.Provider value={toggleTheme}>
        {children}
      </ThemeDispatchContext.Provider>
    </ThemeStateContext.Provider>
  );
}

// Решение 2: useMemo для value
function AuthProvider({ children }: { children: React.ReactNode }) {
  const [state, dispatch] = useReducer(authReducer, initialState);
  
  const value = useMemo(() => ({
    ...state,
    login: (email: string, password: string) => {/* ... */},
    logout: () => dispatch({ type: 'LOGOUT' })
  }), [state]);
  
  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}

// Решение 3: Селекторы с дополнительными библиотеками
// use-context-selector, zustand, jotai
```

**Для чего:**
- Глобальное состояние (темы, локализация, авторизация)
- Избежание prop drilling
- Dependency injection
- Конфигурация приложения

**Плюсы:**
- Встроенный в React
- Простой API
- Не требует дополнительных библиотек
- Хорошо для простых случаев

**Минусы:**
- Ререндерит все компоненты использующие контекст
- Нет встроенной оптимизации
- Сложно масштабируется для большого стейта
- Нет DevTools

**Аналоги:**
- Redux, MobX, Zustand, Jotai, Recoil для более сложного state management

---

### 23. Redux и Redux Toolkit

**Как работает:**
Redux - библиотека для предсказуемого управления состоянием приложения. Основана на паттерне Flux от Facebook.

**Основные принципы:**

1. **Single Source of Truth** - всё состояние хранится в одном store
2. **State is Read-Only** - изменить state можно только dispatch action
3. **Changes are Made with Pure Functions** - reducers - чистые функции

**Core Redux:**

```javascript
// Actions - объекты описывающие ЧТО произошло
const increment = () => ({ type: 'INCREMENT' });
const decrement = () => ({ type: 'DECREMENT' });
const addTodo = (text) => ({
  type: 'ADD_TODO',
  payload: { id: Date.now(), text, completed: false }
});

// Action Creators - функции создающие actions
function fetchUserRequest() {
  return { type: 'FETCH_USER_REQUEST' };
}

// Reducers - чистые функции определяющие КАК изменить state
function counterReducer(state = { value: 0 }, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, value: state.value + 1 };
    case 'DECREMENT':
      return { ...state, value: state.value - 1 };
    default:
      return state;
  }
}

function todosReducer(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, action.payload];
    case 'TOGGLE_TODO':
      return state.map(todo =>
        todo.id === action.payload.id
          ? { ...todo, completed: !todo.completed }
          : todo
      );
    default:
      return state;
  }
}

// Root Reducer - комбинация всех reducers
import { combineReducers } from 'redux';

const rootReducer = combineReducers({
  counter: counterReducer,
  todos: todosReducer
});

// Store - единое хранилище state
import { createStore } from 'redux';

const store = createStore(
  rootReducer,
  window.__REDUX_DEVTOOLS_EXTENSION__?.()
);

// Использование
store.dispatch(increment());
console.log(store.getState()); // { counter: { value: 1 }, todos: [] }

// Подписка на изменения
const unsubscribe = store.subscribe(() => {
  console.log('State changed:', store.getState());
});
```

**Redux Toolkit (современный подход):**

```javascript
import { createSlice, configureStore, createAsyncThunk } from '@reduxjs/toolkit';

// createSlice - создает reducer и action creators автоматически
const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: (state) => {
      // Immer позволяет "мутировать" state
      state.value += 1;
    },
    decrement: (state) => {
      state.value -= 1;
    },
    incrementByAmount: (state, action) => {
      state.value += action.payload;
    }
  }
});

// Actions автоматически созданы
export const { increment, decrement, incrementByAmount } = counterSlice.actions;

// Async thunks для асинхронных операций
export const fetchUser = createAsyncThunk(
  'user/fetch',
  async (userId, { rejectWithValue }) => {
    try {
      const response = await fetch(`/api/users/${userId}`);
      if (!response.ok) throw new Error('Failed to fetch');
      return await response.json();
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

const userSlice = createSlice({
  name: 'user',
  initialState: {
    data: null,
    loading: false,
    error: null
  },
  reducers: {
    clearError: (state) => {
      state.error = null;
    }
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUser.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.loading = false;
        state.data = action.payload;
      })
      .addCase(fetchUser.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      });
  }
});

// Store configuration с Redux Toolkit
const store = configureStore({
  reducer: {
    counter: counterSlice.reducer,
    user: userSlice.reducer
  },
  // Middleware автоматически добавлены (thunk, devtools, immutability check)
  // Можно добавить кастомные
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(loggerMiddleware)
});

// React hooks для использования Redux
import { useSelector, useDispatch } from 'react-redux';

function Counter() {
  // useSelector - выбор данных из store
  const count = useSelector((state) => state.counter.value);
  const dispatch = useDispatch();

  return (
    <div>
      <span>{count}</span>
      <button onClick={() => dispatch(increment())}>+</button>
      <button onClick={() => dispatch(decrement())}>-</button>
      <button onClick={() => dispatch(incrementByAmount(5))}>+5</button>
    </div>
  );
}

function UserProfile({ userId }) {
  const { data, loading, error } = useSelector((state) => state.user);
  const dispatch = useDispatch();

  useEffect(() => {
    dispatch(fetchUser(userId));
  }, [userId, dispatch]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!data) return null;

  return (
    <div>
      <h1>{data.name}</h1>
      <p>{data.email}</p>
    </div>
  );
}
```

**Селекторы и мемоизация:**

```javascript
import { createSelector } from '@reduxjs/toolkit';

// Базовый селектор
const selectTodos = (state) => state.todos;
const selectFilter = (state) => state.filter;

// Мемоизированный селектор (пересчитывается только при изменении зависимостей)
const selectFilteredTodos = createSelector(
  [selectTodos, selectFilter],
  (todos, filter) => {
    console.log('Recalculating filtered todos');
    switch (filter) {
      case 'completed':
        return todos.filter(todo => todo.completed);
      case 'active':
        return todos.filter(todo => !todo.completed);
      default:
        return todos;
    }
  }
);

// Использование
function TodoList() {
  const filteredTodos = useSelector(selectFilteredTodos);
  
  return (
    <ul>
      {filteredTodos.map(todo => (
        <TodoItem key={todo.id} todo={todo} />
      ))}
    </ul>
  );
}

// Селектор с параметрами
const selectTodoById = createSelector(
  [selectTodos, (state, todoId) => todoId],
  (todos, todoId) => todos.find(todo => todo.id === todoId)
);

function TodoDetails({ todoId }) {
  const todo = useSelector(state => selectTodoById(state, todoId));
  return <div>{todo?.text}</div>;
}
```

**Middleware:**

```javascript
// Logger middleware (пример)
const loggerMiddleware = (store) => (next) => (action) => {
  console.log('Dispatching:', action);
  console.log('Previous state:', store.getState());
  const result = next(action);
  console.log('Next state:', store.getState());
  return result;
};

// Analytics middleware
const analyticsMiddleware = (store) => (next) => (action) => {
  // Отправляем аналитику для определенных actions
  if (action.type.startsWith('user/')) {
    analytics.track(action.type, action.payload);
  }
  return next(action);
};

// Error handling middleware
const errorMiddleware = (store) => (next) => (action) => {
  try {
    return next(action);
  } catch (error) {
    console.error('Error in action:', action, error);
    // Можно dispatch специальный error action
    store.dispatch({ type: 'ERROR_OCCURRED', payload: error });
    throw error;
  }
};

const store = configureStore({
  reducer: rootReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware()
      .concat(loggerMiddleware)
      .concat(analyticsMiddleware)
      .concat(errorMiddleware)
});
```

**RTK Query - для работы с API:**

```javascript
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

const api = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({ baseUrl: '/api' }),
  tagTypes: ['User', 'Post'],
  endpoints: (builder) => ({
    // Queries - для чтения данных
    getUser: builder.query({
      query: (id) => `/users/${id}`,
      providesTags: (result, error, id) => [{ type: 'User', id }]
    }),
    
    getPosts: builder.query({
      query: () => '/posts',
      providesTags: ['Post']
    }),
    
    // Mutations - для изменения данных
    updateUser: builder.mutation({
      query: ({ id, ...patch }) => ({
        url: `/users/${id}`,
        method: 'PATCH',
        body: patch
      }),
      invalidatesTags: (result, error, { id }) => [{ type: 'User', id }]
    }),
    
    createPost: builder.mutation({
      query: (body) => ({
        url: '/posts',
        method: 'POST',
        body
      }),
      invalidatesTags: ['Post']
    })
  })
});

export const {
  useGetUserQuery,
  useGetPostsQuery,
  useUpdateUserMutation,
  useCreatePostMutation
} = api;

// В store
const store = configureStore({
  reducer: {
    [api.reducerPath]: api.reducer
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(api.middleware)
});

// Использование
function UserProfile({ userId }) {
  const { data, error, isLoading } = useGetUserQuery(userId);
  const [updateUser, { isLoading: isUpdating }] = useUpdateUserMutation();

  const handleUpdate = async () => {
    try {
      await updateUser({ id: userId, name: 'New Name' }).unwrap();
      alert('Updated!');
    } catch (error) {
      alert('Failed to update');
    }
  };

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error!</div>;

  return (
    <div>
      <h1>{data.name}</h1>
      <button onClick={handleUpdate} disabled={isUpdating}>
        Update
      </button>
    </div>
  );
}
```

**Для чего:**
- Управление глобальным состоянием
- Предсказуемость изменений
- Централизованная логика
- Time-travel debugging
- Middleware для side effects

**Плюсы:**
- Предсказуемое управление состоянием
- Отличные DevTools
- Большая экосистема
- Middleware система
- Хорошо масштабируется

**Минусы:**
- Много boilerplate (решено в RTK)
- Кривая обучения
- Может быть избыточным для простых приложений
- Дополнительная библиотека

**Альтернативы:**
- **Zustand** - минималистичный, простой API
- **MobX** - observable state, меньше boilerplate
- **Jotai** - атомарный state
- **Recoil** - от Facebook, atom-based
- **Context API** - встроен в React, для простых случаев

---

## Алгоритмы и структуры данных

### 24. Основные алгоритмы сортировки

**Bubble Sort (Пузырьковая сортировка):**

```javascript
function bubbleSort(arr) {
  const n = arr.length;
  
  for (let i = 0; i < n - 1; i++) {
    let swapped = false;
    
    for (let j = 0; j < n - i - 1; j++) {
      if (arr[j] > arr[j + 1]) {
        [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
        swapped = true;
      }
    }
    
    // Если не было обменов, массив отсортирован
    if (!swapped) break;
  }
  
  return arr;
}

// Сложность:
// Время: O(n²) худший/средний, O(n) лучший
// Память: O(1)
// Стабильная: Да
```

**Quick Sort (Быстрая сортировка):**

```javascript
function quickSort(arr, left = 0, right = arr.length - 1) {
  if (left < right) {
    const pivotIndex = partition(arr, left, right);
    quickSort(arr, left, pivotIndex - 1);
    quickSort(arr, pivotIndex + 1, right);
  }
  return arr;
}

function partition(arr, left, right) {
  const pivot = arr[right];
  let i = left - 1;
  
  for (let j = left; j < right; j++) {
    if (arr[j] < pivot) {
      i++;
      [arr[i], arr[j]] = [arr[j], arr[i]];
    }
  }
  
  [arr[i + 1], arr[right]] = [arr[right], arr[i + 1]];
  return i + 1;
}

// Сложность:
// Время: O(n log n) средний, O(n²) худший
// Память: O(log n) из-за рекурсии
// Стабильная: Нет
// Используется: Array.sort() в V8 (для больших массивов)
```

**Merge Sort (Сортировка слиянием):**

```javascript
function mergeSort(arr) {
  if (arr.length <= 1) return arr;
  
  const mid = Math.floor(arr.length / 2);
  const left = mergeSort(arr.slice(0, mid));
  const right = mergeSort(arr.slice(mid));
  
  return merge(left, right);
}

function merge(left, right) {
  const result = [];
  let i = 0, j = 0;
  
  while (i < left.length && j < right.length) {
    if (left[i] < right[j]) {
      result.push(left[i]);
      i++;
    } else {
      result.push(right[j]);
      j++;
    }
  }
  
  return result.concat(left.slice(i)).concat(right.slice(j));
}

// Сложность:
// Время: O(n log n) всегда
// Память: O(n)
// Стабильная: Да
```

**Heap Sort (Пирамидальная сортировка):**

```javascript
function heapSort(arr) {
  const n = arr.length;
  
  // Построение max-heap
  for (let i = Math.floor(n / 2) - 1; i >= 0; i--) {
    heapify(arr, n, i);
  }
  
  // Извлечение элементов из heap
  for (let i = n - 1; i > 0; i--) {
    [arr[0], arr[i]] = [arr[i], arr[0]];
    heapify(arr, i, 0);
  }
  
  return arr;
}

function heapify(arr, n, i) {
  let largest = i;
  const left = 2 * i + 1;
  const right = 2 * i + 2;
  
  if (left < n && arr[left] > arr[largest]) {
    largest = left;
  }
  
  if (right < n && arr[right] > arr[largest]) {
    largest = right;
  }
  
  if (largest !== i) {
    [arr[i], arr[largest]] = [arr[largest], arr[i]];
    heapify(arr, n, largest);
  }
}

// Сложность:
// Время: O(n log n) всегда
// Память: O(1)
// Стабильная: Нет
```

**Insertion Sort (Сортировка вставками):**

```javascript
function insertionSort(arr) {
  for (let i = 1; i < arr.length; i++) {
    const key = arr[i];
    let j = i - 1;
    
    while (j >= 0 && arr[j] > key) {
      arr[j + 1] = arr[j];
      j--;
    }
    
    arr[j + 1] = key;
  }
  
  return arr;
}

// Сложность:
// Время: O(n²) худший/средний, O(n) лучший
// Память: O(1)
// Стабильная: Да
// Используется: Array.sort() в V8 (для малых массивов < 10 элементов)
```

**Counting Sort (Сортировка подсчетом):**

```javascript
function countingSort(arr) {
  if (arr.length === 0) return arr;
  
  const max = Math.max(...arr);
  const min = Math.min(...arr);
  const range = max - min + 1;
  
  const count = new Array(range).fill(0);
  const output = new Array(arr.length);
  
  // Подсчет элементов
  for (let i = 0; i < arr.length; i++) {
    count[arr[i] - min]++;
  }
  
  // Кумулятивные суммы
  for (let i = 1; i < count.length; i++) {
    count[i] += count[i - 1];
  }
  
  // Построение отсортированного массива
  for (let i = arr.length - 1; i >= 0; i--) {
    output[count[arr[i] - min] - 1] = arr[i];
    count[arr[i] - min]--;
  }
  
  return output;
}

// Сложность:
// Время: O(n + k) где k - диапазон значений
// Память: O(k)
// Стабильная: Да
// Подходит только для целых чисел в ограниченном диапазоне
```

**Сравнение алгоритмов:**

| Алгоритм | Время (средн.) | Время (худш.) | Память | Стабильная |
|----------|----------------|---------------|--------|------------|
| Bubble Sort | O(n²) | O(n²) | O(1) | Да |
| Quick Sort | O(n log n) | O(n²) | O(log n) | Нет |
| Merge Sort | O(n log n) | O(n log n) | O(n) | Да |
| Heap Sort | O(n log n) | O(n log n) | O(1) | Нет |
| Insertion Sort | O(n²) | O(n²) | O(1) | Да |
| Counting Sort | O(n + k) | O(n + k) | O(k) | Да |

**Когда использовать:**

**Bubble Sort:**
- Учебные цели
- Очень маленькие массивы
- Почти отсортированные данные

**Quick Sort:**
- Общего назначения
- Когда важна скорость
- Когда память ограничена

**Merge Sort:**
- Нужна стабильная сортировка
- Когда важна предсказуемость O(n log n)
- Сортировка связных списков

**Heap Sort:**
- Когда важна память O(1)
- Гарантированная O(n log n)

**Insertion Sort:**
- Маленькие массивы (< 10-50 элементов)
- Почти отсортированные данные
- Online сортировка (по мере поступления)

**Counting Sort:**
- Целые числа в ограниченном диапазоне
- Когда n >> k

**Как работает Array.sort() в JavaScript:**

```javascript
// V8 (Chrome, Node.js) использует Timsort
// - Гибрид Merge Sort и Insertion Sort
// - Для малых массивов (< 10): Insertion Sort
// - Для больших: Merge Sort с оптимизациями

[3, 1, 4, 1, 5].sort(); // Timsort
[3, 1, 4, 1, 5].sort((a, b) => a - b); // С компаратором

// ВАЖНО: sort() мутирует исходный массив
const arr = [3, 1, 2];
arr.sort(); // arr = [1, 2, 3]

// Для иммутабельности:
const sorted = [...arr].sort();
```

---

### 25. Поиск в массиве и строках

**Linear Search (Линейный поиск):**

```javascript
function linearSearch(arr, target) {
  for (let i = 0; i < arr.length; i++) {
    if (arr[i] === target) {
      return i;
    }
  }
  return -1;
}

// Сложность: O(n)
// Используется: когда массив не отсортирован
```

**Binary Search (Бинарный поиск):**

```javascript
function binarySearch(arr, target) {
  let left = 0;
  let right = arr.length - 1;
  
  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    
    if (arr[mid] === target) {
      return mid;
    } else if (arr[mid] < target) {
      left = mid + 1;
    } else {
      right = mid - 1;
    }
  }
  
  return -1;
}

// Рекурсивная версия
function binarySearchRecursive(arr, target, left = 0, right = arr.length - 1) {
  if (left > right) return -1;
  
  const mid = Math.floor((left + right) / 2);
  
  if (arr[mid] === target) return mid;
  if (arr[mid] < target) return binarySearchRecursive(arr, target, mid + 1, right);
  return binarySearchRecursive(arr, target, left, mid - 1);
}

// Сложность: O(log n)
// Требования: отсортированный массив
```

**Two Pointers (Два указателя):**

```javascript
// Найти пару чисел с заданной суммой (отсортированный массив)
function twoSum(arr, target) {
  let left = 0;
  let right = arr.length - 1;
  
  while (left < right) {
    const sum = arr[left] + arr[right];
    
    if (sum === target) {
      return [left, right];
    } else if (sum < target) {
      left++;
    } else {
      right--;
    }
  }
  
  return [-1, -1];
}

// Сложность: O(n)

// Удалить дубликаты из отсортированного массива
function removeDuplicates(arr) {
  if (arr.length === 0) return 0;
  
  let writeIndex = 1;
  
  for (let readIndex = 1; readIndex < arr.length; readIndex++) {
    if (arr[readIndex] !== arr[readIndex - 1]) {
      arr[writeIndex] = arr[readIndex];
      writeIndex++;
    }
  }
  
  return writeIndex; // новая длина
}
```

**Sliding Window (Скользящее окно):**

```javascript
// Максимальная сумма подмассива размера k
function maxSumSubarray(arr, k) {
  if (arr.length < k) return null;
  
  let windowSum = 0;
  
  // Первое окно
  for (let i = 0; i < k; i++) {
    windowSum += arr[i];
  }
  
  let maxSum = windowSum;
  
  // Скользим окно
  for (let i = k; i < arr.length; i++) {
    windowSum = windowSum - arr[i - k] + arr[i];
    maxSum = Math.max(maxSum, windowSum);
  }
  
  return maxSum;
}

// Сложность: O(n)

// Наименьший подмассив с суммой >= target
function minSubArrayLen(target, arr) {
  let minLen = Infinity;
  let windowSum = 0;
  let start = 0;
  
  for (let end = 0; end < arr.length; end++) {
    windowSum += arr[end];
    
    while (windowSum >= target) {
      minLen = Math.min(minLen, end - start + 1);
      windowSum -= arr[start];
      start++;
    }
  }
  
  return minLen === Infinity ? 0 : minLen;
}
```

**String Search (Поиск в строках):**

```javascript
// Naive pattern matching
function naiveSearch(text, pattern) {
  const results = [];
  
  for (let i = 0; i <= text.length - pattern.length; i++) {
    let match = true;
    
    for (let j = 0; j < pattern.length; j++) {
      if (text[i + j] !== pattern[j]) {
        match = false;
        break;
      }
    }
    
    if (match) results.push(i);
  }
  
  return results;
}

// Сложность: O(n * m) где n - длина текста, m - длина pattern

// KMP (Knuth-Morris-Pratt) Algorithm
function KMPSearch(text, pattern) {
  const lps = computeLPSArray(pattern);
  const results = [];
  
  let i = 0; // индекс для text
  let j = 0; // индекс для pattern
  
  while (i < text.length) {
    if (pattern[j] === text[i]) {
      i++;
      j++;
    }
    
    if (j === pattern.length) {
      results.push(i - j);
      j = lps[j - 1];
    } else if (i < text.length && pattern[j] !== text[i]) {
      if (j !== 0) {
        j = lps[j - 1];
      } else {
        i++;
      }
    }
  }
  
  return results;
}

function computeLPSArray(pattern) {
  const lps = new Array(pattern.length).fill(0);
  let len = 0;
  let i = 1;
  
  while (i < pattern.length) {
    if (pattern[i] === pattern[len]) {
      len++;
      lps[i] = len;
      i++;
    } else {
      if (len !== 0) {
        len = lps[len - 1];
      } else {
        lps[i] = 0;
        i++;
      }
    }
  }
  
  return lps;
}

// Сложность: O(n + m)
```

**Для чего:**
- Поиск элементов в данных
- Оптимизация запросов
- Текстовые редакторы
- Автодополнение

**Плюсы знания алгоритмов:**
- Оптимальные решения
- Лучшая производительность
- Прохождение собеседований

---

## Архитектура приложений

### 26. Clean Architecture и SOLID принципы

**SOLID Принципы:**

**S - Single Responsibility Principle (Принцип единственной ответственности):**

```javascript
// ❌ Плохо - класс делает слишком много
class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }
  
  save() {
    // Логика сохранения в БД
    fetch('/api/users', {
      method: 'POST',
      body: JSON.stringify(this)
    });
  }
  
  sendEmail(subject, body) {
    // Логика отправки email
    emailService.send(this.email, subject, body);
  }
  
  validate() {
    // Логика валидации
    return this.email.includes('@');
  }
}

// ✅ Хорошо - каждый класс отвечает за одно
class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }
}

class UserRepository {
  async save(user) {
    return fetch('/api/users', {
      method: 'POST',
      body: JSON.stringify(user)
    });
  }
}

class EmailService {
  send(email, subject, body) {
    // Логика отправки
  }
}

class UserValidator {
  validate(user) {
    return user.email.includes('@');
  }
}
```

**O - Open/Closed Principle (Принцип открытости/закрытости):**

```javascript
// ❌ Плохо - нужно модифицировать класс для добавления нового типа
class PaymentProcessor {
  processPayment(type, amount) {
    if (type === 'credit') {
      // Логика для кредитной карты
    } else if (type === 'paypal') {
      // Логика для PayPal
    } else if (type === 'crypto') {
      // Логика для крипто
    }
  }
}

// ✅ Хорошо - открыт для расширения, закрыт для модификации
interface PaymentMethod {
  process(amount: number): Promise<void>;
}

class CreditCardPayment implements PaymentMethod {
  async process(amount: number) {
    // Логика для кредитной карты
  }
}

class PayPalPayment implements PaymentMethod {
  async process(amount: number) {
    // Логика для PayPal
  }
}

class CryptoPayment implements PaymentMethod {
  async process(amount: number) {
    // Логика для крипто
  }
}

class PaymentProcessor {
  constructor(private paymentMethod: PaymentMethod) {}
  
  async processPayment(amount: number) {
    await this.paymentMethod.process(amount);
  }
}

// Использование
const processor = new PaymentProcessor(new CreditCardPayment());
await processor.processPayment(100);
```

**L - Liskov Substitution Principle (Принцип подстановки Барбары Лисков):**

```typescript
// ❌ Плохо - Square нарушает контракт Rectangle
class Rectangle {
  constructor(protected width: number, protected height: number) {}
  
  setWidth(width: number) {
    this.width = width;
  }
  
  setHeight(height: number) {
    this.height = height;
  }
  
  getArea() {
    return this.width * this.height;
  }
}

class Square extends Rectangle {
  setWidth(width: number) {
    this.width = width;
    this.height = width; // Нарушает ожидания Rectangle
  }
  
  setHeight(height: number) {
    this.width = height;
    this.height = height;
  }
}

// Тест ломается:
function testRectangle(rect: Rectangle) {
  rect.setWidth(5);
  rect.setHeight(4);
  expect(rect.getArea()).toBe(20); // Fail для Square!
}

// ✅ Хорошо - используем композицию
interface Shape {
  getArea(): number;
}

class Rectangle implements Shape {
  constructor(private width: number, private height: number) {}
  
  getArea() {
    return this.width * this.height;
  }
}

class Square implements Shape {
  constructor(private side: number) {}
  
  getArea() {
    return this.side * this.side;
  }
}
```

**I - Interface Segregation Principle (Принцип разделения интерфейса):**

```typescript
// ❌ Плохо - "толстый" интерфейс
interface Worker {
  work(): void;
  eat(): void;
  sleep(): void;
}

class Human implements Worker {
  work() { /* ... */ }
  eat() { /* ... */ }
  sleep() { /* ... */ }
}

class Robot implements Worker {
  work() { /* ... */ }
  eat() { throw new Error('Robots don't eat'); } // Вынуждены реализовать
  sleep() { throw new Error('Robots don't sleep'); }
}

// ✅ Хорошо - разделенные интерфейсы
interface Workable {
  work(): void;
}

interface Eatable {
  eat(): void;
}

interface Sleepable {
  sleep(): void;
}

class Human implements Workable, Eatable, Sleepable {
  work() { /* ... */ }
  eat() { /* ... */ }
  sleep() { /* ... */ }
}

class Robot implements Workable {
  work() { /* ... */ }
}
```

**D - Dependency Inversion Principle (Принцип инверсии зависимостей):**

```typescript
// ❌ Плохо - зависимость от конкретной реализации
class MySQLDatabase {
  connect() { /* ... */ }
  query(sql: string) { /* ... */ }
}

class UserService {
  private db = new MySQLDatabase(); // Жесткая зависимость
  
  getUser(id: number) {
    return this.db.query(`SELECT * FROM users WHERE id = ${id}`);
  }
}

// ✅ Хорошо - зависимость от абстракции
interface Database {
  connect(): void;
  query(sql: string): Promise<any>;
}

class MySQLDatabase implements Database {
  connect() { /* ... */ }
  query(sql: string) { /* ... */ }
}

class PostgreSQLDatabase implements Database {
  connect() { /* ... */ }
  query(sql: string) { /* ... */ }
}

class UserService {
  constructor(private db: Database) {} // Зависимость инжектится
  
  getUser(id: number) {
    return this.db.query(`SELECT * FROM users WHERE id = ${id}`);
  }
}

// Использование
const db = new MySQLDatabase(); // Можно легко заменить на PostgreSQLDatabase
const userService = new UserService(db);
```

**Clean Architecture (Чистая Архитектура):**

**Слои (от внутреннего к внешнему):**

1. **Entities (Domain)** - бизнес-логика
2. **Use Cases (Application)** - сценарии использования
3. **Interface Adapters** - контроллеры, презентеры, gateways
4. **Frameworks & Drivers** - UI, БД, внешние сервисы

**Пример:**

```typescript
// 1. Entities (Domain Layer)
class User {
  constructor(
    public readonly id: string,
    public readonly email: string,
    public readonly name: string
  ) {}
  
  isValid(): boolean {
    return this.email.includes('@') && this.name.length > 0;
  }
}

// 2. Use Cases (Application Layer)
interface UserRepository {
  findById(id: string): Promise<User | null>;
  save(user: User): Promise<void>;
}

class GetUserUseCase {
  constructor(private userRepository: UserRepository) {}
  
  async execute(userId: string): Promise<User> {
    const user = await this.userRepository.findById(userId);
    
    if (!user) {
      throw new Error('User not found');
    }
    
    return user;
  }
}

class CreateUserUseCase {
  constructor(private userRepository: UserRepository) {}
  
  async execute(email: string, name: string): Promise<User> {
    const user = new User(generateId(), email, name);
    
    if (!user.isValid()) {
      throw new Error('Invalid user data');
    }
    
    await this.userRepository.save(user);
    return user;
  }
}

// 3. Interface Adapters (Presentation Layer)
class UserController {
  constructor(
    private getUserUseCase: GetUserUseCase,
    private createUserUseCase: CreateUserUseCase
  ) {}
  
  async getUser(req: Request, res: Response) {
    try {
      const user = await this.getUserUseCase.execute(req.params.id);
      res.json(user);
    } catch (error) {
      res.status(404).json({ error: error.message });
    }
  }
  
  async createUser(req: Request, res: Response) {
    try {
      const user = await this.createUserUseCase.execute(
        req.body.email,
        req.body.name
      );
      res.status(201).json(user);
    } catch (error) {
      res.status(400).json({ error: error.message });
    }
  }
}

// 4. Frameworks & Drivers (Infrastructure Layer)
class MySQLUserRepository implements UserRepository {
  constructor(private db: Database) {}
  
  async findById(id: string): Promise<User | null> {
    const result = await this.db.query(
      'SELECT * FROM users WHERE id = ?',
      [id]
    );
    
    if (!result) return null;
    
    return new User(result.id, result.email, result.name);
  }
  
  async save(user: User): Promise<void> {
    await this.db.query(
      'INSERT INTO users (id, email, name) VALUES (?, ?, ?)',
      [user.id, user.email, user.name]
    );
  }
}

// Dependency Injection Container
const db = new MySQLDatabase();
const userRepository = new MySQLUserRepository(db);
const getUserUseCase = new GetUserUseCase(userRepository);
const createUserUseCase = new CreateUserUseCase(userRepository);
const userController = new UserController(getUserUseCase, createUserUseCase);

// Express routes
app.get('/users/:id', (req, res) => userController.getUser(req, res));
app.post('/users', (req, res) => userController.createUser(req, res));
```

**Для чего:**
- Поддерживаемый код
- Тестируемость
- Масштабируемость
- Независимость от фреймворков

**Плюсы:**
- Четкое разделение ответственности
- Легко тестировать
- Легко заменять компоненты
- Уменьшение связанности

**Минусы:**
- Больше кода
- Может быть избыточно для малых проектов
- Кривая обучения

---

Это первая часть гайда. Хотите, чтобы я продолжил с остальными разделами?

# Design Patterns

## Module Pattern

### Encapsulation with Modules

**Principle:** Use modules to encapsulate private state and expose a public API.

```typescript
// ✅ Good - module pattern with closures
const Counter = (() => {
  let count = 0;

  return {
    increment() { count++; },
    decrement() { count--; },
    getCount() { return count; },
    reset() { count = 0; }
  };
})();

Counter.increment();
Counter.increment();
Counter.getCount(); // 2

// ❌ Bad - exposed state
let count = 0;
function increment() { count++; }
function getCount() { return count; }
```

### ES Modules

```typescript
// ✅ Good - explicit exports
// math.ts
export function add(a: number, b: number): number {
  return a + b;
}

export function multiply(a: number, b: number): number {
  return a * b;
}

export const PI = 3.14159;

// ✅ Good - named imports
import { add, multiply, PI } from './math';

// ✅ Good - default exports
// logger.ts
export default class Logger {
  log(message: string): void {
    console.log(`[${new Date().toISOString()}] ${message}`);
  }
}

// ✅ Good - default + named exports
// user.ts
export class UserService { }
export const MAX_USERS = 1000;
export default UserService;
```

---

## Observer Pattern

### Simple Event Emitter

**Principle:** Decouple event producers from event consumers.

```typescript
// ✅ Good - simple event emitter
class EventEmitter<T = unknown> {
  private listeners = new Map<string, Set<(event: T) => void>>();

  on(event: string, callback: (event: T) => void): () => void {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, new Set());
    }
    this.listeners.get(event)!.add(callback);

    return () => this.off(event, callback);
  }

  off(event: string, callback: (event: T) => void): void {
    this.listeners.get(event)?.delete(callback);
  }

  emit(event: string, data: T): void {
    this.listeners.get(event)?.forEach(cb => cb(data));
  }
}

// Usage
const emitter = new EventEmitter<{ userId: string }>();

const unsubscribe = emitter.on('user:login', ({ userId }) => {
  console.log(`User ${userId} logged in`);
});

emitter.emit('user:login', { userId: '123' });
unsubscribe(); // remove listener
```

### Observer Interface

```typescript
// ✅ Good - explicit observer interface
interface Observer<T> {
  update(data: T): void;
}

class Subject<T> {
  private observers = new Set<Observer<T>>();

  attach(observer: Observer<T>): void {
    this.observers.add(observer);
  }

  detach(observer: Observer<T>): void {
    this.observers.delete(observer);
  }

  notify(data: T): void {
    this.observers.forEach(obs => obs.update(data));
  }
}

// Usage
class UserList implements Observer<{ type: string; user: User }> {
  update({ type, user }: { type: string; user: User }): void {
    switch (type) {
      case 'add': this.addUser(user); break;
      case 'remove': this.removeUser(user.id); break;
    }
  }

  private addUser(user: User): void { /* ... */ }
  private removeUser(id: string): void { /* ... */ }
}
```

---

## Factory Pattern

### Simple Factory

**Principle:** Centralize object creation logic.

```typescript
// ✅ Good - simple factory
interface Button {
  render(): string;
}

class PrimaryButton implements Button {
  render(): string {
    return '<button class="btn btn-primary">Submit</button>';
  }
}

class SecondaryButton implements Button {
  render(): string {
    return '<button class="btn btn-secondary">Cancel</button>';
  }
}

function createButton(type: 'primary' | 'secondary'): Button {
  switch (type) {
    case 'primary': return new PrimaryButton();
    case 'secondary': return new SecondaryButton();
  }
}

const button = createButton('primary');

// ❌ Bad - scattered creation logic
function renderForm() {
  if (type === 'primary') {
    html += '<button class="btn btn-primary">Submit</button>';
  } else {
    html += '<button class="btn btn-secondary">Cancel</button>';
  }
}
```

### Factory with Parameters

```typescript
// ✅ Good - factory with configuration
interface Config {
  baseUrl: string;
  timeout: number;
  retries: number;
}

class ApiClient {
  constructor(private config: Config) { }
}

function createApiClient(config: Partial<Config> = {}): ApiClient {
  const finalConfig: Config = {
    baseUrl: config.baseUrl ?? 'https://api.example.com',
    timeout: config.timeout ?? 5000,
    retries: config.retries ?? 3
  };
  return new ApiClient(finalConfig);
}

const client = createApiClient({ timeout: 10000 });
```

---

## Dependency Injection

### Constructor Injection

**Principle:** Pass dependencies explicitly rather than creating them inside.

```typescript
// ✅ Good - dependency injection via constructor
interface Logger {
  log(message: string): void;
}

interface UserRepository {
  findById(id: string): Promise<User | null>;
}

class UserService {
  constructor(
    private logger: Logger,
    private userRepo: UserRepository
  ) { }

  async getUser(id: string): Promise<User | null> {
    this.logger.log(`Fetching user ${id}`);
    return this.userRepo.findById(id);
  }
}

// Usage
const logger = new ConsoleLogger();
const userRepo = new SqlUserRepository();
const userService = new UserService(logger, userRepo);

// ❌ Bad - hard-coded dependencies
class UserService {
  private logger = new ConsoleLogger();
  private userRepo = new SqlUserRepository();

  async getUser(id: string): Promise<User | null> {
    this.logger.log(`Fetching user ${id}`);
    return this.userRepo.findById(id);
  }
}
```

### DI Container

```typescript
// ✅ Good - simple DI container
class Container {
  private services = new Map<string, unknown>();

  register<T>(token: string, factory: () => T): void {
    this.services.set(token, factory);
  }

  resolve<T>(token: string): T {
    const factory = this.services.get(token);
    if (!factory) throw new Error(`Service ${token} not registered`);
    return (factory as () => T)();
  }
}

const container = new Container();

container.register('Logger', () => new ConsoleLogger());
container.register('UserRepo', () => new SqlUserRepository());
container.register('UserService', () => new UserService(
  container.resolve<Logger>('Logger'),
  container.resolve<UserRepository>('UserRepo')
));

const userService = container.resolve<UserService>('UserService');
```

---

## Singleton Pattern

### When Appropriate

**Principle:** Use singletons sparingly; usually better to inject instances.

```typescript
// ✅ Good - module-level singleton
// database.ts
class Database {
  private static instance: Database;

  static getInstance(): Database {
    if (!Database.instance) {
      Database.instance = new Database();
    }
    return Database.instance;
  }

  query(sql: string): unknown { /* ... */ }
}

export const db = Database.getInstance();

// ✅ Good - explicit singleton when needed
class AppConfig {
  private static instance: AppConfig;

  static getInstance(): AppConfig {
    if (!AppConfig.instance) {
      AppConfig.instance = new AppConfig();
    }
    return AppConfig.instance;
  }

  readonly apiUrl = 'https://api.example.com';
  readonly maxRetries = 3;
}

export const config = AppConfig.getInstance();

// ❌ Bad - global state is often problematic
let globalState = { /* ... */ };
```

---

## Composition Over Inheritance

### Prefer Composition

**Principle:** Compose behavior from smaller pieces rather than inheriting.

```typescript
// ✅ Good - composition
class Logger {
  log(message: string): void { console.log(message); }
}

class Metrics {
  record(metric: string, value: number): void { /* ... */ }
}

class UserService {
  constructor(
    private logger: Logger,
    private metrics: Metrics
  ) { }

  async createUser(data: UserData): Promise<User> {
    this.metrics.record('user.create', 1);
    this.logger.log('Creating user');
    // ...
  }
}

// ❌ Bad - inheritance hierarchy
class BaseService {
  log(message: string): void { console.log(message); }
  metrics(record: string): void { /* ... */ }
}

class UserService extends BaseService {
  async createUser(data: UserData): Promise<User> {
    this.metrics('user.create', 1);
    this.log('Creating user');
    // ...
  }
}
```

### Mixins

```typescript
// ✅ Good - mixin pattern
function withLogging<T extends new (...args: any[]) => {}>(Base: T) {
  return class extends Base {
    log(message: string): void {
      console.log(`[${this.constructor.name}] ${message}`);
    }
  };
}

function withMetrics<T extends new (...args: any[]) => {}>(Base: T) {
  return class extends Base {
    record(metric: string, value: number): void {
      console.log(`Metrics: ${metric} = ${value}`);
    }
  };
}

class UserService extends withLogging(withMetrics(Object)) {
  createUser(data: UserData): User {
    this.log('Creating user');
    this.record('user.create', 1);
    return { id: '1', ...data };
  }
}
```

---

## Notes

1. **Patterns are tools, not rules** - don't force patterns where they don't fit
2. **Prefer composition over inheritance** - more flexible and easier to test
3. **Dependency injection improves testability** - makes mocking easy
4. **Singletons can cause hidden coupling** - use them sparingly
5. **Module pattern provides encapsulation** - useful for maintaining private state
# Clean Code Principles

## Single Responsibility Principle (SRP)

### One Function, One Purpose

**Principle:** Each function should do one thing and do it well.

```typescript
// ✅ Good - single responsibility
function validateEmail(email: string): boolean {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

function sendEmail(email: string, message: string): void {
  // Send email logic
}

// ❌ Bad - multiple responsibilities
function sendEmailWithValidation(email: string, message: string): boolean {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test(email)) {
    return false;
  }
  // Send email logic
  return true;
}
```

### Small Functions

**Principle:** Functions should be small. If a function is over 20 lines, consider splitting it.

```typescript
// ✅ Good - small, focused functions
function parseQuery(queryString: string): Record<string, string> {
  const params = queryString.slice(1).split('&');
  return params.reduce((acc, param) => {
    const [key, value] = param.split('=');
    acc[decodeURIComponent(key)] = decodeURIComponent(value || '');
    return acc;
  }, {});
}

function buildResponse(data: unknown, status = 200): Response {
  return new Response(JSON.stringify(data), {
    status,
    headers: { 'Content-Type': 'application/json' }
  });
}

// ❌ Bad - function trying to do too much
function handleRequest(request: Request): Response {
  // 50+ lines of parsing, validating, processing, and responding
}
```

---

## Don't Repeat Yourself (DRY)

### Extract Repeated Logic

**Principle:** If you see the same pattern twice, extract it.

```typescript
// ✅ Good - extracted common logic
function formatDate(date: Date): string {
  return date.toISOString().split('T')[0];
}

function formatTimestamp(date: Date): string {
  return date.toISOString();
}

// ❌ Bad - repeated date formatting
function processUser(user: User) {
  const created = user.createdAt.toISOString().split('T')[0];
  console.log(`User created: ${created}`);
}

function processPost(post: Post) {
  const created = post.createdAt.toISOString().split('T')[0];
  console.log(`Post created: ${created}`);
}
```

### Constants for Magic Values

```typescript
// ✅ Good - named constants
const MAX_FILE_SIZE = 10 * 1024 * 1024; // 10MB
const ALLOWED_EXTENSIONS = ['.jpg', '.png', '.gif'];
const API_TIMEOUT_MS = 5000;

function validateFile(file: File): boolean {
  if (file.size > MAX_FILE_SIZE) return false;
  const ext = '.' + file.name.split('.').pop()?.toLowerCase();
  return ALLOWED_EXTENSIONS.includes(ext);
}

// ❌ Bad - magic numbers
function validateFile(file: File): boolean {
  if (file.size > 10485760) return false; // what is this?
  const ext = '.' + file.name.split('.').pop()?.toLowerCase();
  return ['.jpg', '.png', '.gif'].includes(ext);
}
```

---

## You Aren't Gonna Need It (YAGNI)

### Don't Over-Engineer

**Principle:** Don't add functionality until you actually need it.

```typescript
// ✅ Good - implement what you need now
interface User {
  id: string;
  name: string;
}

// ❌ Bad - over-engineering for future
interface User {
  id: string;
  name: string;
  email?: string;        // not used yet
  phone?: string;        // not used yet
  address?: Address;     // not used yet
  preferences?: Preferences; // not used yet
  metadata?: Metadata;    // not used yet
}
```

### Prefer Simple Solutions

```typescript
// ✅ Good - simple solution that works
function isEmpty(value: string | null | undefined): boolean {
  return !value;
}

// ❌ Bad - over-engineered
class StringValidator {
  private trim: boolean;
  private caseSensitive: boolean;

  constructor(options?: { trim?: boolean; caseSensitive?: boolean }) {
    this.trim = options?.trim ?? true;
    this.caseSensitive = options?.caseSensitive ?? true;
  }

  isEmpty(value: string): boolean {
    const processed = this.trim ? value.trim() : value;
    return this.caseSensitive
      ? processed === ''
      : processed.toLowerCase() === '';
  }
}
```

---

## SOLID Principles

### Single Responsibility

See SRP section above.

### Open/Closed Principle

**Principle:** Open for extension, closed for modification.

```typescript
// ✅ Good - extend behavior without modifying existing code
interface Transformer<T, R> {
  transform(value: T): R;
}

class StringToUpperCase implements Transformer<string, string> {
  transform(value: string): string {
    return value.toUpperCase();
  }
}

class StringToLength implements Transformer<string, number> {
  transform(value: string): number {
    return value.length;
  }
}

// ❌ Bad - modify existing code to add new behavior
function processValue(value: string, format: 'upper' | 'length'): string | number {
  if (format === 'upper') return value.toUpperCase();
  if (format === 'length') return value.length;
}
```

### Liskov Substitution

**Principle:** Subtypes must be substitutable for their base types.

```typescript
// ✅ Good - proper inheritance
abstract class Shape {
  abstract area(): number;
}

class Rectangle extends Shape {
  constructor(private width: number, private height: number) {
    super();
  }
  area(): number { return this.width * this.height; }
}

class Circle extends Shape {
  constructor(private radius: number) {
    super();
  }
  area(): number { return Math.PI * this.radius ** 2; }
}

// ❌ Bad - violation
class Bird {
  fly(): void { }
}

class Penguin extends Bird {
  fly(): void {
    throw new Error('Penguins cannot fly');
  }
}
```

### Interface Segregation

**Principle:** Prefer small, specific interfaces over large, general ones.

```typescript
// ✅ Good - small, focused interfaces
interface Printable {
  print(): void;
}

interface Saveable {
  save(): void;
}

class Document implements Printable, Saveable {
  print(): void { }
  save(): void { }
}

// ❌ Bad - large, general interface
class Document implements Machine {
  print(): void { }
  scan(): void { }
  fax(): void { }
  copy(): void { }
}
```

### Dependency Inversion

**Principle:** Depend on abstractions, not on concretions.

```typescript
// ✅ Good - depend on abstraction
interface UserRepository {
  findById(id: string): Promise<User | null>;
  save(user: User): Promise<void>;
}

class UserService {
  constructor(private repository: UserRepository) { }

  async getUser(id: string): Promise<User | null> {
    return this.repository.findById(id);
  }
}

// ❌ Bad - depend on concretion
class UserService {
  constructor(private db: SqlDatabase) { } // hard dependency
}
```

---

## Meaningful Names

See [naming.md](naming.md) for detailed naming conventions.

---

## Comments

### Comment Why, Not What

```typescript
// ✅ Good - explains intent/reason
// Using bitwise OR to combine flags efficiently
const flags = FLAG_A | FLAG_B;

// Retry with exponential backoff to handle server load
await retryWithBackoff(() => api.call());

// ❌ Bad - states the obvious
// Increment counter
count++;

// Check if user is logged in
if (isLoggedIn) { }
```

### Document Complex Logic

```typescript
// ✅ Good - explains complex logic
// Kolmogorov's backward algorithm for dynamic programming
function levenshteinDistance(a: string, b: string): number {
  // Implementation
}

// ❌ Bad - no comment for complex code
function fuzzyMatch(str1: string, str2: string): number {
  // 50 lines of complex logic with no explanation
}
```

---

## Notes

1. **YAGNI doesn't mean don't plan** - it means don't implement things you *might* need
2. **DRY doesn't mean never repeat** - sometimes duplication is clearer than the abstraction
3. **Comments should explain intent** - why you did something, not what the code does
4. **Small functions are easier to test** - aim for functions that do one thing
5. **SOLID can be overkill for small scripts** - apply judgment based on project size
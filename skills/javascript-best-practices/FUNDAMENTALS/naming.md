# Naming Conventions

## Variables and Constants

### Use Descriptive Names

**Principle:** Names should reveal intent. Avoid single letters except in short loops or callbacks.

```typescript
// ✅ Good
const userAge = 25;
const isAuthenticated = true;
const totalPrice = 149.99;

// ❌ Bad
const a = 25;
const flag = true;
const tp = 149.99;
```

### Boolean Naming

**Principle:** Prefix boolean variables with `is`, `has`, `can`, `should`, `will`.

```typescript
// ✅ Good
const isActive = true;
const hasPermission = false;
const canEdit = true;
const shouldRedirect = false;

// ❌ Bad
const active = true;
const permission = false;
const edit = true;
const redirect = false;
```

### Constants

**Principle:** Use UPPER_SNAKE_CASE for true constants; camelCase for values that shouldn't mutate.

```typescript
// ✅ Good - true constants
const MAX_RETRY_COUNT = 3;
const API_BASE_URL = 'https://api.example.com';
const DEFAULT_TIMEOUT_MS = 5000;

// ✅ Good - don't mutate (const doesn't guarantee mutability)
const config = { theme: 'dark', lang: 'en' };

// ❌ Bad
const maxRetryCount = 3;  // looks mutable
const baseUrl = 'https://api.example.com';
```

---

## Functions

### Verb + Noun Pattern

**Principle:** Function names should start with a verb and describe the action.

```typescript
// ✅ Good
function getUserById(id: string): User { }
function calculateTotal(items: Item[]): number { }
function validateEmail(email: string): boolean { }
function fetchUserData(): Promise<User> { }

// ❌ Bad
function user(id: string): User { }
function total(items: Item[]): number { }
function email(email: string): boolean { }
function data(): Promise<User> { }
```

### Event Handlers

**Principle:** Prefix with `handle`; suffix with event type if ambiguous.

```typescript
// ✅ Good
function handleClick(event: MouseEvent): void { }
function handleInputChange(value: string): void { }
function handleSubmit(form: HTMLFormElement): void { }

// ❌ Bad
function onClick(event: MouseEvent): void { }
function click(event: MouseEvent): void { }
function submit(form: HTMLFormElement): void { }
```

### Boolean Return Functions

**Principle:** Use `is`, `has`, `can`, `should` prefixes.

```typescript
// ✅ Good
function isValidEmail(email: string): boolean { }
function hasPermission(user: User, action: string): boolean { }
function canAccess(resource: Resource): boolean { }

// ❌ Bad
function validEmail(email: string): boolean { }
function permission(user: User, action: string): boolean { }
function access(resource: Resource): boolean { }
```

---

## Classes and Types

### Class Naming

**Principle:** Use PascalCase, noun phrases describing what the class represents.

```typescript
// ✅ Good
class UserService { }
class PaymentProcessor { }
class EventEmitter { }
class ConfigStore { }

// ❌ Bad
class userService { }
class service { }
class processPayments { }
class store { }
```

### Interface and Type Naming

**Principle:** Use PascalCase. For interfaces, consider adding `I` prefix only if your codebase uses it consistently.

```typescript
// ✅ Good - no prefix (preferred)
interface UserRepository { }
type CreateUserDto = { name: string; email: string };
interface ApiResponse<T> { data: T; error: string | null; }

// ✅ Good - with I prefix (if consistent in codebase)
interface IUserRepository { }
interface IApiResponse<T> { }

// ❌ Bad
interface userRepository { }
type userDto { }
```

### Generic Type Parameters

**Principle:** Use descriptive names, usually single letters or very short words.

```typescript
// ✅ Good
function getFirst<T>(array: T[]): T | undefined { }
function mapObject<K, V>(obj: Record<K, V>, fn: (v: V) => V): Record<K, V> { }
interface Repository<T extends BaseEntity> { }

// ❌ Bad
function getFirst<MyType>(array: MyType[]): MyType | undefined { }
function mapObject<KeyType, ValueType>(obj: any, fn: any): any { }
```

---

## Files and Directories

### File Naming

**Principle:** Use kebab-case for files, PascalCase for class-containing files.

```typescript
// ✅ Good
// File: user-service.ts
export class UserService { }

// File: types.ts
export type User = { id: string; name: string };

// File: use-auth.ts
export function useAuth() { }

// ❌ Bad
// File: userService.ts (camelCase)
// File: UserService.ts (PascalCase file with kebab-case export)
```

### Directory Naming

**Principle:** Use kebab-case for directories.

```
// ✅ Good
src/
  user-management/
  payment-processing/
  auth/
  api-client/

// ❌ Bad
src/
  userManagement/
  PaymentProcessing/
  Auth/
```

---

## Enums and Constants

### Enum Naming

**Principle:** Use PascalCase for enum name, UPPER_SNAKE_CASE for values.

```typescript
// ✅ Good
enum HttpStatus {
  OK = 200,
  NOT_FOUND = 404,
  INTERNAL_ERROR = 500,
}

enum UserRole {
  ADMIN = 'admin',
  EDITOR = 'editor',
  VIEWER = 'viewer',
}

// ❌ Bad
enum httpStatus {
  ok = 200,
  notFound = 404,
}

enum user_role {
  admin = 'admin',
  editor = 'editor',
}
```

---

## Notes

1. **Consistency > Correctness** - If your codebase uses a convention, keep using it
2. **Context Matters** - `i`, `j` are fine for loop counters; `e` is fine for error catch blocks
3. **Team Agreements** - Document naming conventions in your team's style guide
4. **Avoid Abbreviations** - `config` over `cfg`, `description` over `desc`, `function` over `fn`
# TypeScript Essentials

## Type Inference

### Let TypeScript Infer When Obvious

**Principle:** Don't annotate when the type is obvious from the right side.

```typescript
// ✅ Good - inference works well
const name = 'John';           // string
const count = 42;               // number
const isActive = true;          // boolean
const items = [1, 2, 3];       // number[]
const user = { id: 1, name: 'John' };  // { id: number; name: string }

// ❌ Bad - redundant annotation
const name: string = 'John';
const count: number = 42;
const isActive: boolean = true;
```

### Explicit Annotations When Needed

**Principle:** Annotate function parameters, return types for public APIs, and when inference isn't clear.

```typescript
// ✅ Good - explicit for public API
function processUser(user: User): ProcessedUser {
  return { ...user, processed: true };
}

// ✅ Good - complex inferred type needs annotation
const fetchData = async (): Promise<ApiResponse<Data>> => {
  const response = await fetch('/api/data');
  return response.json();
};

// ❌ Bad - missing return type on public function
function processUser(user: User) {
  return { ...user, processed: true };
}
```

---

## Interfaces vs Type Aliases

### Use Interface for Object Shapes

**Principle:** Interfaces are more extensible and produce better error messages.

```typescript
// ✅ Good - use interface
interface User {
  id: string;
  name: string;
  email: string;
  createdAt: Date;
}

interface UserRepository {
  findById(id: string): Promise<User | null>;
  save(user: User): Promise<void>;
  delete(id: string): Promise<void>;
}

// ❌ Bad - use type for object shape
type User = {
  id: string;
  name: string;
};
```

### Use Type for Unions, Intersections, and Primitives

**Principle:** Types are better for unions, mapped types, and utility operations.

```typescript
// ✅ Good - use type
type Status = 'pending' | 'active' | 'deleted';
type Id = string | number;
type UserOrGuest = User | Guest;
type UserWithMeta = User & { meta: Metadata };

// ✅ Good - use type for computed/concatenated types
type UserKeys = keyof User;
type PartialUser = Partial<User>;
type RequiredUser = Required<User>;
```

### Combining Both

**Principle:** Use whichever makes the code clearer in context.

```typescript
// ✅ Good - interface for extensibility
interface Config {
  debug: boolean;
}

interface ConfigWithEndpoint extends Config {
  endpoint: string;
}

// ✅ Good - type for union
type Response = SuccessResponse | ErrorResponse;
```

---

## Generics

### Use Generics for Reusable Logic

**Principle:** Don't use `any` when you can use generics.

```typescript
// ✅ Good - generic function
function first<T>(array: T[]): T | undefined {
  return array[0];
}

const num = first([1, 2, 3]);       // number | undefined
const str = first(['a', 'b', 'c']); // string | undefined

// ❌ Bad - using any
function first(array: any[]): any {
  return array[0];
}
```

### Constrain Generics When Needed

**Principle:** Use `extends` to constrain generic types.

```typescript
// ✅ Good - constrained generic
function getId<T extends { id: string | number }>(entity: T): string {
  return String(entity.id);
}

interface HasId {
  id: string | number;
}

function getId<T extends HasId>(entity: T): string {
  return String(entity.id);
}

// ❌ Bad - unconstrained when constraint is known
function getId<T>(entity: T): string {
  return (entity as any).id;
}
```

### Generic Constraints

```typescript
// ✅ Good - multiple constraints
type Primitive = string | number | boolean;

function processValue<T extends Primitive>(value: T): string {
  return String(value);
}

// ✅ Good - keyof constraint
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const name = getProperty({ name: 'John', age: 30 }, 'name'); // string
```

---

## Union Types and Type Guards

### Discriminated Unions

**Principle:** Use discriminated unions for handling multiple related types.

```typescript
// ✅ Good - discriminated union
type LoadingState = { status: 'loading' };
type SuccessState = { status: 'success'; data: User[] };
type ErrorState = { status: 'error'; message: string };

type State = LoadingState | SuccessState | ErrorState;

function handleState(state: State): string {
  switch (state.status) {
    case 'loading': return 'Loading...';
    case 'success': return `Found ${state.data.length} users`;
    case 'error': return `Error: ${state.message}`;
  }
}

// ❌ Bad - no discriminated union
type BadState = {
  status: 'loading' | 'success' | 'error';
  data?: User[];
  message?: string;
};
```

### Type Guards

**Principle:** Use type guards to narrow types safely.

```typescript
// ✅ Good - custom type guard
function isUser(obj: unknown): obj is User {
  return typeof obj === 'object' && obj !== null && 'id' in obj && 'name' in obj;
}

if (isUser(data)) {
  console.log(data.name); // data is User here
}

// ✅ Good - built-in type guards
if (value instanceof Error) {
  console.log(value.message);
}

// ✅ Good - typeof guard
if (typeof value === 'string') {
  console.log(value.toUpperCase());
}
```

---

## Utility Types

### Common Utility Types

```typescript
// ✅ Good - Partial (all properties optional)
type PartialUser = Partial<User>;

// ✅ Good - Required (all properties required)
type RequiredConfig = Required<Config>;

// ✅ Good - Pick (select specific properties)
type UserPreview = Pick<User, 'id' | 'name'>;

// ✅ Good - Omit (exclude specific properties)
type UserWithoutPassword = Omit<User, 'password'>;

// ✅ Good - Record (key-value map)
type UserById = Record<string, User>;

// ✅ Good - Readonly
type FrozenConfig = Readonly<Config>;

// ❌ Bad - manually doing what utilities do
type PartialUserBad = { id?: string; name?: string; email?: string };
```

### Custom Utility Types

```typescript
// ✅ Good - custom utility type
type Nullable<T> = T | null;
type AsyncResult<T> = Promise<{ data: T; error: null } | { data: null; error: Error }>;

// ✅ Good - mapped types
type Stringify<T> = {
  [K in keyof T]: string;
};

type UserStrings = Stringify<User>; // all values are strings
```

---

## Notes

1. **Prefer interfaces for object shapes** that might be extended
2. **Use types for unions, intersections, and computed types**
3. **Don't overuse generics** - if it's hard to read, simplify
4. **TypeScript's strict mode** catches more errors - enable it in tsconfig.json
5. **Use `unknown` instead of `any`** when you don't know the type, then narrow it
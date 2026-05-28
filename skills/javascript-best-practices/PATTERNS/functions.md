# Function Patterns

## Parameters

### Use Default Parameters When Sensible

**Principle:** Defaults make functions more flexible and reduce the need for conditionals.

```typescript
// ✅ Good - sensible defaults
function createUser(name: string, role = 'viewer'): User {
  return { name, role, createdAt: new Date() };
}

createUser('John');              // role: 'viewer'
createUser('Jane', 'admin');     // role: 'admin'

// ❌ Bad - requires conditional logic
function createUser(name: string, role?: string): User {
  return {
    name,
    role: role ?? 'viewer',
    createdAt: new Date()
  };
}
```

### Rest Parameters for Variable Arguments

**Principle:** Use rest parameters for functions that accept any number of arguments.

```typescript
// ✅ Good - rest parameters
function sum(...numbers: number[]): number {
  return numbers.reduce((total, n) => total + n, 0);
}

sum(1, 2, 3);        // 6
sum(1, 2, 3, 4, 5); // 15

// ❌ Bad - array parameter
function sum(numbers: number[]): number {
  return numbers.reduce((total, n) => total + n, 0);
}

sum([1, 2, 3]);
```

### Destructuring for Object Parameters

**Principle:** Destructuring makes parameter intent clear and supports partial passing.

```typescript
// ✅ Good - destructured parameters
function createButton({
  label,
  variant = 'primary',
  size = 'medium',
  disabled = false
}: ButtonProps): HTMLButtonElement {
  // implementation
}

createButton({ label: 'Submit', variant: 'secondary' });

// ❌ Bad - unclear parameters
function createButton(label: string, variant?: string, size?: string, disabled?: boolean) {
  // implementation
}

createButton('Submit', 'secondary', undefined, undefined);
```

---

## Return Early

### Guard Clauses

**Principle:** Handle edge cases and errors first, reduce nesting.

```typescript
// ✅ Good - guard clauses, early returns
function processUser(user: User | null): Result {
  if (!user) {
    return { success: false, error: 'User not found' };
  }

  if (!user.email) {
    return { success: false, error: 'Email required' };
  }

  if (!user.isActive) {
    return { success: false, error: 'User inactive' };
  }

  // Main logic here with no nesting
  return { success: true, data: processUserData(user) };
}

// ❌ Bad - deeply nested
function processUser(user: User | null): Result {
  if (user) {
    if (user.email) {
      if (user.isActive) {
        return { success: true, data: processUserData(user) };
      } else {
        return { success: false, error: 'User inactive' };
      }
    } else {
      return { success: false, error: 'Email required' };
    }
  } else {
    return { success: false, error: 'User not found' };
  }
}
```

### Avoid else When Possible

```typescript
// ✅ Good - no else needed
function getStatus(status: number): string {
  if (status === 200) return 'ok';
  if (status === 404) return 'not found';
  if (status === 500) return 'error';
  return 'unknown';
}

// ❌ Bad - unnecessary else
function getStatus(status: number): string {
  if (status === 200) {
    return 'ok';
  } else {
    if (status === 404) {
      return 'not found';
    } else {
      if (status === 500) {
        return 'error';
      } else {
        return 'unknown';
      }
    }
  }
}
```

---

## Function Composition

### Small, Composable Functions

**Principle:** Build complex operations from simple, single-purpose functions.

```typescript
// ✅ Good - composable functions
const trim = (str: string) => str.trim();
const toLowerCase = (str: string) => str.toLowerCase();
const removeSpecialChars = (str: string) => str.replace(/[^a-z0-9]/g, '');
const normalize = (str: string) => trim(toLowerCase(removeSpecialChars(str)));

normalize('  Hello, World!  '); // 'helloworld'

// Pipeline version
const pipe = (...fns: Function[]) => (value: unknown) =>
  fns.reduce((acc, fn) => fn(acc), value);

const normalizePipe = pipe(trim, toLowerCase, removeSpecialChars);

// ❌ Bad - monolithic function
function normalize(str: string): string {
  const trimmed = str.trim();
  const lower = trimmed.toLowerCase();
  const cleaned = lower.replace(/[^a-z0-9]/g, '');
  return cleaned;
}
```

### Currying for Reusability

```typescript
// ✅ Good - curried function
const multiply = (a: number) => (b: number) => a * b;
const double = multiply(2);
const triple = multiply(3);

double(5); // 10
triple(5); // 15

// ✅ Good - practical currying
const withPrefix = (prefix: string) => (str: string) => `${prefix}${str}`;
const withSuffix = (suffix: string) => (str: string) => `${str}${suffix}`;

const addBrackets = withPrefix('[');
const addParens = withPrefix('(');
const addSuffixExclaim = withSuffix('!');

addBrackets('hello');    // '[hello'
addParens('world');      // '(world'
addSuffixExclaim('hi');  // 'hi!'
```

---

## Arrow Functions

### Implicit Returns

**Principle:** Use implicit return for simple expressions.

```typescript
// ✅ Good - implicit return for expressions
const double = (n: number) => n * 2;
const getName = (user: User) => user.name;
const sum = (...nums: number[]) => nums.reduce((a, b) => a + b, 0);

// ✅ Good - explicit return for statements
const doubleAndLog = (n: number) => {
  const result = n * 2;
  console.log(`Result: ${result}`);
  return result;
};

// ❌ Bad - unnecessary braces for simple return
const double = (n: number) => { return n * 2; };
```

### When NOT to Use Arrow Functions

```typescript
// ❌ Bad - don't use arrow for methods
const user = {
  name: 'John',
  // Wrong: 'this' doesn't refer to user
  getName: () => this.name,

  // Correct: use shorthand method syntax
  getNameAlt() { return this.name; }
};

// ❌ Bad - don't use arrow in constructors
class User {
  constructor() {
    // Arrow doesn't have own 'this'
  }
}

// ✅ Good - arrow functions for callbacks
[1, 2, 3].map(n => n * 2);
```

---

## Pure Functions

### Avoid Side Effects When Possible

**Principle:** Pure functions are easier to test and reason about.

```typescript
// ✅ Good - pure function
function add(a: number, b: number): number {
  return a + b;
}

function getFullName(first: string, last: string): string {
  return `${first} ${last}`;
}

// ❌ Bad - side effects
let total = 0;
function addToTotal(amount: number): void {
  total += amount; // modifies external state
}
```

### When Side Effects Are Necessary

```typescript
// ✅ Good - side effects are explicit in function name
function saveUserToDatabase(user: User): void {
  database.save(user);
}

function logMessage(message: string): void {
  console.log(message);
}

function sendAnalyticsEvent(event: string, data: unknown): void {
  analytics.track(event, data);
}
```

---

## Notes

1. **Aim for 1-3 parameters** - more than 3 suggests the function does too much
2. **Return consistent types** - don't sometimes return `null` and sometimes throw
3. **Pure functions are easier to test** - but some side effects are unavoidable
4. **Don't mutate parameters** - create new objects instead
5. **Consider function length** - if over 20 lines, consider splitting
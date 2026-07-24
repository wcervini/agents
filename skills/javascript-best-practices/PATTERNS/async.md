# Async JavaScript

## Promises

### Create Promises Judiciously

**Principle:** Don't wrap sync code in Promises unnecessarily.

```typescript
// ✅ Good - async operations deserve promises
function fetchUser(id: string): Promise<User> {
  return fetch(`/api/users/${id}`).then(res => res.json());
}

function delay(ms: number): Promise<void> {
  return new Promise(resolve => setTimeout(resolve, ms));
}

// ❌ Bad - wrapping sync code in promises
function getConfig(): Promise<Config> {
  return Promise.resolve({ debug: true });
}

function add(a: number, b: number): Promise<number> {
  return Promise.resolve(a + b);
}
```

### Promise Chaining

```typescript
// ✅ Good - clear promise chain
function getUserAndPosts(userId: string): Promise<{ user: User; posts: Post[] }> {
  return fetchUser(userId)
    .then(user =>
      fetchPosts(userId).then(posts => ({ user, posts }))
    );
}

// ❌ Bad - promise hell
function getUserAndPosts(userId: string): Promise<{ user: User; posts: Post[] }> {
  return fetchUser(userId).then(user => {
    return fetchPosts(userId).then(posts => {
      return fetchComments(userId).then(comments => {
        return fetchLikes(userId).then(likes => {
          return { user, posts, comments, likes };
        });
      });
    });
  });
}
```

---

## Async/Await

### Prefer Async/Await Over Then/Catch

**Principle:** Async/await produces cleaner, more readable code.

```typescript
// ✅ Good - async/await
async function getUserData(userId: string): Promise<UserData> {
  const user = await fetchUser(userId);
  const posts = await fetchPosts(userId);
  return { user, posts };
}

// ❌ Bad - promise chains
function getUserData(userId: string): Promise<UserData> {
  return fetchUser(userId)
    .then(user =>
      fetchPosts(userId).then(posts => ({ user, posts }))
    );
}
```

### Await Promises in Parallel When Possible

```typescript
// ✅ Good - parallel when independent
async function getDashboardData(): Promise<DashboardData> {
  const [user, stats, notifications] = await Promise.all([
    fetchUser(),
    fetchStats(),
    fetchNotifications()
  ]);
  return { user, stats, notifications };
}

// ❌ Bad - sequential when parallel is possible
async function getDashboardData(): Promise<DashboardData> {
  const user = await fetchUser();
  const stats = await fetchStats();    // waits for user
  const notifications = await fetchNotifications(); // waits for stats
  return { user, stats, notifications };
}
```

### Promise.allSettled for Partial Failures

```typescript
// ✅ Good - handle partial failures
async function processItems<T>(
  items: T[],
  processor: (item: T) => Promise<Result>
): Promise<{ succeeded: Result[]; failed: { item: T; error: Error }[] }> {
  const results = await Promise.allSettled(items.map(processor));

  const succeeded: Result[] = [];
  const failed: { item: T; error: Error }[] = [];

  results.forEach((result, index) => {
    if (result.status === 'fulfilled') {
      succeeded.push(result.value);
    } else {
      failed.push({ item: items[index], error: result.reason });
    }
  });

  return { succeeded, failed };
}

// ❌ Bad - one failure fails all
async function processItems<T>(
  items: T[],
  processor: (item: T) => Promise<Result>
): Promise<Result[]> {
  return Promise.all(items.map(processor)); // one rejection fails everything
}
```

---

## Error Handling

### Try/Catch Appropriately

**Principle:** Only catch what you can handle, rethrow unexpected errors.

```typescript
// ✅ Good - specific error handling
async function saveUser(user: User): Promise<void> {
  try {
    await validateUser(user);
    await database.save(user);
    await sendWelcomeEmail(user);
  } catch (error) {
    if (error instanceof ValidationError) {
      throw new UserInputError(error.message);
    }
    if (error instanceof DatabaseError) {
      throw new ServiceUnavailableError('Unable to save user');
    }
    throw error; // rethrow unexpected errors
  }
}

// ❌ Bad - catching everything
async function saveUser(user: User): Promise<void> {
  try {
    await saveToDatabase(user);
  } catch (error) {
    console.error('Error saving user:', error);
  }
}
```

### Error Propagation

```typescript
// ✅ Good - errors bubble up with context
async function processOrder(orderId: string): Promise<void> {
  const order = await fetchOrder(orderId); // throws if not found

  if (!order) {
    throw new NotFoundError(`Order ${orderId} not found`);
  }

  await chargeCustomer(order.paymentMethod, order.total);
  await fulfillOrder(order);
}

// ❌ Bad - swallowing errors
async function processOrder(orderId: string): Promise<void> {
  const order = await fetchOrder(orderId).catch(() => null);
  if (!order) return; // silently ignores network errors
}
```

### Custom Error Classes

```typescript
// ✅ Good - typed errors
class AppError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode: number = 500
  ) {
    super(message);
    this.name = this.constructor.name;
  }
}

class NotFoundError extends AppError {
  constructor(resource: string) {
    super(`${resource} not found`, 'NOT_FOUND', 404);
  }
}

class ValidationError extends AppError {
  constructor(message: string, public field?: string) {
    super(message, 'VALIDATION_ERROR', 400);
  }
}

// Usage
async function getUser(id: string): Promise<User> {
  const user = await db.findUser(id);
  if (!user) {
    throw new NotFoundError('User');
  }
  return user;
}
```

---

## Async Patterns

### Async Iteration

```typescript
// ✅ Good - async iterator for streaming
async function* fetchUsersInBatches(batchSize: number): AsyncGenerator<User[]> {
  let offset = 0;
  while (true) {
    const batch = await fetchUsers({ limit: batchSize, offset });
    if (batch.length === 0) break;
    yield batch;
    offset += batchSize;
  }
}

// Usage
for await (const batch of fetchUsersInBatches(100)) {
  processBatch(batch);
}

// ❌ Bad - loading all in memory
async function getAllUsers(): Promise<User[]> {
  const allUsers: User[] = [];
  let hasMore = true;
  while (hasMore) {
    const { data, hasMore: more } = await fetchUsers({ offset: allUsers.length });
    allUsers.push(...data);
    hasMore = more;
  }
  return allUsers;
}
```

### Async Pool / Concurrency Limit

```typescript
// ✅ Good - limit concurrency
async function processWithLimit<T, R>(
  items: T[],
  processor: (item: T) => Promise<R>,
  limit: number
): Promise<R[]> {
  const results: R[] = [];
  const executing = new Set<Promise<void>>();

  for (const item of items) {
    const promise = processor(item).then(result => {
      results.push(result);
      executing.delete(promise);
    });

    executing.add(promise);

    if (executing.size >= limit) {
      await Promise.race(executing);
    }
  }

  await Promise.all(executing);
  return results;
}

// ❌ Bad - unbounded concurrency
async function processAll(items: Item[]): Promise<Result[]> {
  return Promise.all(items.map(item => processExpensive(item)));
}
```

### Top-Level Await (ES2022)

```Principle:** Use top-level await for module initialization.

```typescript
// ✅ Good - module initialization
const config = await fetchConfig();

export const api = createApiClient(config);

export const userService = new UserService(api);

// ✅ Good - conditional loading
const maybeDatabase = process.env.DATABASE_URL
  ? await connectToDatabase(process.env.DATABASE_URL)
  : createMockDatabase();
```

---

## Notes

1. **Promise.all fails fast** - use `Promise.allSettled` when partial success is acceptable
2. **Don't forget to await** - forgetting await is a common bug
3. **Avoid async in constructors** - use factory functions or initialization methods instead
4. **Handle errors explicitly** - don't leave promises floating without error handling
5. **Consider cancellation** - use AbortController for cancellable operations
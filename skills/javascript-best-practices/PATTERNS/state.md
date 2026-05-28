# State Management

## Immutability

### Prefer Immutability

**Principle:** Don't mutate objects directly; create new copies with changes.

```typescript
// ✅ Good - immutable updates
function addItem(state: State, item: Item): State {
  return { ...state, items: [...state.items, item] };
}

function updateUser(state: State, userId: string, updates: Partial<User>): State {
  return {
    ...state,
    users: state.users.map(user =>
      user.id === userId ? { ...user, ...updates } : user
    )
  };
}

function removeItem(state: State, itemId: string): State {
  return {
    ...state,
    items: state.items.filter(item => item.id !== itemId)
  };
}

// ❌ Bad - mutating directly
function addItem(state: State, item: Item): State {
  state.items.push(item); // mutates original
  return state;
}
```

### Immutable Array Operations

```typescript
// ✅ Good - immutable array operations
const add = (arr: number[], item: number) => [...arr, item];
const remove = (arr: number[], index: number) => arr.filter((_, i) => i !== index);
const update = (arr: number[], index: number, value: number) =>
  arr.map((item, i) => i === index ? value : item);

const prepend = (arr: number[], item: number) => [item, ...arr];
const concat = (a: number[], b: number[]) => [...a, ...b];

// ❌ Bad - mutating array methods
const add = (arr: number[], item: number) => { arr.push(item); return arr; };
const remove = (arr: number[], index: number) => { arr.splice(index, 1); return arr; };
```

### Nested Object Immutability

```typescript
// ✅ Good - deep clone for nested updates
function updateNested(state: State, path: string[], value: unknown): State {
  return produce(state, draft => {
    let current = draft;
    for (let i = 0; i < path.length - 1; i++) {
      current = current[path[i]];
    }
    current[path[path.length - 1]] = value;
  });
}

// ✅ Good - using Immer (recommended for complex state)
// npm install immer
import { produce } from 'immer';

function updateUser(state: State, userId: string, fn: (user: User) => void): State {
  return produce(state, draft => {
    const user = draft.users.find(u => u.id === userId);
    if (user) fn(user);
  });
}

// ❌ Bad - manual deep clone
function updateNested(state: State, path: string[], value: unknown): State {
  const clone = JSON.parse(JSON.stringify(state));
  let current = clone;
  for (let i = 0; i < path.length - 1; i++) {
    current = current[path[i]];
  }
  current[path[path.length - 1]] = value;
  return clone;
}
```

---

## State Patterns

### Simple State Object

**Principle:** For simple apps, use a plain state object with update functions.

```typescript
// ✅ Good - simple state management
type State = {
  users: User[];
  loading: boolean;
  error: string | null;
};

const initialState: State = {
  users: [],
  loading: false,
  error: null
};

function createStore(reducer: (state: State, action: Action) => State) {
  let state = initialState;
  const listeners = new Set<() => void>();

  return {
    getState: () => state,
    dispatch: (action: Action) => {
      state = reducer(state, action);
      listeners.forEach(listener => listener());
    },
    subscribe: (listener: () => void) => {
      listeners.add(listener);
      return () => listeners.delete(listener);
    }
  };
}
```

### State Machine (XState-like)

**Principle:** Model state as a finite state machine for complex flows.

```typescript
// ✅ Good - simple state machine
type OrderState = 'pending' | 'processing' | 'shipped' | 'delivered' | 'cancelled';

type OrderContext = {
  orderId: string;
  items: Item[];
  trackingNumber?: string;
};

type OrderEvent =
  | { type: 'PROCESS' }
  | { type: 'SHIP'; trackingNumber: string }
  | { type: 'DELIVER' }
  | { type: 'CANCEL' };

const orderMachine = {
  initial: 'pending' as OrderState,

  states: {
    pending: {
      on: { PROCESS: 'processing' }
    },
    processing: {
      on: { SHIP: 'shipped', CANCEL: 'cancelled' }
    },
    shipped: {
      on: { DELIVER: 'delivered' }
    },
    delivered: {},
    cancelled: {}
  }
};

function transition(
  state: OrderState,
  event: OrderEvent,
  context: OrderContext
): OrderState {
  const nextState = orderMachine.states[state].on[event.type];
  if (!nextState) return state;

  // Side effects based on transition
  if (event.type === 'SHIP') {
    sendShippingNotification(context.orderId, event.trackingNumber);
  }

  return nextState;
}
```

### useState Pattern (React)

```typescript
// ✅ Good - separate state concerns
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);
  const [posts, setPosts] = useState<Post[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    setLoading(true);
    fetchUser(userId)
      .then(setUser)
      .catch(err => setError(err.message))
      .finally(() => setLoading(false));
  }, [userId]);

  // ...
}

// ❌ Bad - object state when separate makes sense
function UserProfile({ userId }: { userId: string }) {
  const [state, setState] = useState<{
    user: User | null;
    posts: Post[];
    loading: boolean;
    error: string | null;
  }>({ user: null, posts: [], loading: false, error: null });

  // Harder to update single fields
}
```

### useReducer for Complex State

```typescript
// ✅ Good - useReducer for related state transitions
type State = {
  count: number;
  history: number[];
};

type Action =
  | { type: 'INCREMENT' }
  | { type: 'DECREMENT' }
  | { type: 'RESET' };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1, history: [...state.history, state.count + 1] };
    case 'DECREMENT':
      return { count: state.count - 1, history: [...state.history, state.count - 1] };
    case 'RESET':
      return { count: 0, history: [] };
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0, history: [] });
  // ...
}
```

---

## State Updates

### Functional Updates

**Principle:** When new state depends on old state, use functional updates.

```typescript
// ✅ Good - functional update
const [count, setCount] = useState(0);

setCount(prev => prev + 1);      // when depending on previous
setCount(prev => prev * 2);
setCount(c => c + 1);           // shorthand

// ❌ Bad - direct reference
setCount(count + 1); // might use stale count in async context
```

### Batched Updates

```typescript
// ✅ Good - batch related state updates
function loadData() {
  setLoading(true);
  setError(null);

  fetchData()
    .then(data => {
      setData(data);
      setLoading(false);
    })
    .catch(error => {
      setError(error.message);
      setLoading(false);
    });
}

// ❌ Bad - multiple renders
function loadData() {
  setLoading(true);
  fetchData().then(data => {
    setData(data);
    setLoading(false); // second render
  });
}
```

---

## Derived State

### Don't Derive Unnecessarily

**Principle:** Compute values when needed, not always.

```typescript
// ✅ Good - derive on render (simple cases)
function UserList({ users }: { users: User[] }) {
  const activeUsers = users.filter(u => u.isActive);
  const adminCount = users.filter(u => u.role === 'admin').length;

  return (
    <div>
      <p>Active: {activeUsers.length}</p>
      <p>Admins: {adminCount}</p>
      {users.map(user => (
        <UserItem key={user.id} user={user} />
      ))}
    </div>
  );
}

// ✅ Good - useMemo for expensive computations
function ExpensiveList({ items, filter }: Props) {
  const filteredItems = useMemo(
    () => expensiveFilter(items, filter),
    [items, filter]
  );

  return filteredItems.map(item => <Item key={item.id} item={item} />);
}

// ❌ Bad - storing derived state
const [filteredItems, setFilteredItems] = useState<Item[]>([]);

useEffect(() => {
  setFilteredItems(expensiveFilter(items, filter));
}, [items, filter]);

// filteredItems is derived from items and filter - no need for separate state
```

---

## Notes

1. **Immutability enables time-travel debugging** - state history is easier with immutable updates
2. **Single source of truth** - don't duplicate state in multiple places
3. **Minimize derived state** - compute when possible, don't store what can be derived
4. **Consider normalized state** - flatten deeply nested objects for easier updates
5. **State machines prevent impossible states** - model explicit states and transitions
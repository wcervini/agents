# Actions

Actions provide type-safe form handling and server functions in Astro.

## Setup

Actions are defined in `src/actions/index.ts`:

```typescript
// src/actions/index.ts
import { defineAction, z } from 'astro:actions';

export const server = {
  // Actions go here
};
```

## Defining Actions

### Basic Action

```typescript
// src/actions/index.ts
import { defineAction, z } from 'astro:actions';

export const server = {
  subscribe: defineAction({
    input: z.object({
      email: z.string().email(),
      name: z.string().min(2),
    }),
    handler: async ({ email, name }) => {
      // Save to database, send email, etc.
      await db.subscribers.create({ email, name });
      return { success: true, message: 'Subscribed!' };
    },
  }),
};
```

### Action with Accept Header

```typescript
export const server = {
  // Accepts form data
  submitForm: defineAction({
    accept: 'form', // Parses FormData
    input: z.object({
      email: z.string().email(),
      message: z.string(),
    }),
    handler: async ({ email, message }) => {
      await sendEmail(email, message);
      return { sent: true };
    },
  }),

  // Accepts JSON (default)
  createPost: defineAction({
    accept: 'json', // Default
    input: z.object({
      title: z.string(),
      content: z.string(),
    }),
    handler: async ({ title, content }) => {
      const post = await db.posts.create({ title, content });
      return post;
    },
  }),
};
```

### Action Without Input

```typescript
export const server = {
  getCurrentUser: defineAction({
    handler: async (_, context) => {
      const user = context.locals.user;
      return user || null;
    },
  }),
};
```

## Using Actions

### In Forms (Progressive Enhancement)

```astro
---
import { actions } from 'astro:actions';
---

<form method="POST" action={actions.subscribe}>
  <input type="email" name="email" required />
  <input type="text" name="name" required />
  <button type="submit">Subscribe</button>
</form>
```

### With JavaScript

```astro
---
import { actions } from 'astro:actions';
---

<form id="subscribe-form">
  <input type="email" name="email" required />
  <input type="text" name="name" required />
  <button type="submit">Subscribe</button>
</form>

<script>
  import { actions } from 'astro:actions';

  const form = document.getElementById('subscribe-form');

  form.addEventListener('submit', async (e) => {
    e.preventDefault();
    const formData = new FormData(form);

    const { data, error } = await actions.subscribe({
      email: formData.get('email'),
      name: formData.get('name'),
    });

    if (error) {
      console.error(error.message);
      return;
    }

    console.log('Success:', data.message);
  });
</script>
```

### Server-Side Calls

```astro
---
import { actions } from 'astro:actions';

// Call action from server
const { data, error } = await actions.subscribe({
  email: 'user@example.com',
  name: 'John Doe',
});
---
```

## Handling Results

### getActionResult

Get the result of a form submission:

```astro
---
import { actions, getActionResult } from 'astro:actions';

const result = await getActionResult(actions.subscribe);

if (result?.error) {
  // Handle validation errors
}
---

{result?.data && (
  <p class="success">{result.data.message}</p>
)}

{result?.error && (
  <p class="error">{result.error.message}</p>
)}

<form method="POST" action={actions.subscribe}>
  <input type="email" name="email" required />
  <button type="submit">Subscribe</button>
</form>
```

### Error Handling

```typescript
import { defineAction, z, ActionError } from 'astro:actions';

export const server = {
  createUser: defineAction({
    input: z.object({
      email: z.string().email(),
    }),
    handler: async ({ email }) => {
      const existing = await db.users.findByEmail(email);

      if (existing) {
        throw new ActionError({
          code: 'CONFLICT',
          message: 'User already exists',
        });
      }

      return await db.users.create({ email });
    },
  }),
};
```

### Error Codes

Available error codes:
- `BAD_REQUEST` - Invalid input
- `UNAUTHORIZED` - Authentication required
- `FORBIDDEN` - Permission denied
- `NOT_FOUND` - Resource not found
- `CONFLICT` - Resource conflict
- `PRECONDITION_FAILED` - Condition not met
- `INTERNAL_SERVER_ERROR` - Server error

```astro
---
const result = await getActionResult(actions.createUser);

if (result?.error?.code === 'CONFLICT') {
  // Handle duplicate user
}
---
```

## Input Validation

### Zod Schema Validation

```typescript
import { defineAction, z } from 'astro:actions';

export const server = {
  updateProfile: defineAction({
    input: z.object({
      username: z.string()
        .min(3, 'Username must be at least 3 characters')
        .max(20, 'Username must be at most 20 characters')
        .regex(/^[a-z0-9_]+$/, 'Only lowercase letters, numbers, and underscores'),

      bio: z.string().max(500).optional(),

      birthdate: z.coerce.date()
        .min(new Date('1900-01-01'))
        .max(new Date()),

      tags: z.array(z.string()).max(5).default([]),
    }),
    handler: async (data) => {
      return await db.profiles.update(data);
    },
  }),
};
```

### Accessing Validation Errors

```astro
---
const result = await getActionResult(actions.updateProfile);

const fieldErrors = result?.error?.fields;
// { username: ['Too short'], bio: ['Too long'] }
---

<form method="POST" action={actions.updateProfile}>
  <input type="text" name="username" />
  {fieldErrors?.username && (
    <span class="error">{fieldErrors.username[0]}</span>
  )}
</form>
```

## Context Access

```typescript
import { defineAction, z } from 'astro:actions';

export const server = {
  protectedAction: defineAction({
    input: z.object({ data: z.string() }),
    handler: async ({ data }, context) => {
      // Access request
      const ip = context.request.headers.get('x-forwarded-for');

      // Access cookies
      const token = context.cookies.get('session')?.value;

      // Access locals (from middleware)
      const user = context.locals.user;

      if (!user) {
        throw new ActionError({
          code: 'UNAUTHORIZED',
          message: 'Must be logged in',
        });
      }

      return { processed: true };
    },
  }),
};
```

## File Uploads

```typescript
export const server = {
  uploadImage: defineAction({
    accept: 'form',
    input: z.object({
      image: z.instanceof(File),
      description: z.string().optional(),
    }),
    handler: async ({ image, description }) => {
      const buffer = await image.arrayBuffer();
      const path = await saveFile(buffer, image.name);
      return { url: path };
    },
  }),
};
```

```astro
<form method="POST" action={actions.uploadImage} enctype="multipart/form-data">
  <input type="file" name="image" accept="image/*" />
  <input type="text" name="description" />
  <button type="submit">Upload</button>
</form>
```

## Redirect After Action

```typescript
import { defineAction, z, ActionError } from 'astro:actions';

export const server = {
  login: defineAction({
    accept: 'form',
    input: z.object({
      email: z.string().email(),
      password: z.string(),
    }),
    handler: async ({ email, password }, context) => {
      const user = await authenticate(email, password);

      if (!user) {
        throw new ActionError({
          code: 'UNAUTHORIZED',
          message: 'Invalid credentials',
        });
      }

      context.cookies.set('session', user.token, { httpOnly: true });

      // Return redirect URL for client to handle
      return { redirect: '/dashboard' };
    },
  }),
};
```

```astro
---
const result = await getActionResult(actions.login);

if (result?.data?.redirect) {
  return Astro.redirect(result.data.redirect);
}
---
```

## Calling Actions via fetch (getActionPath)

`getActionPath()` (Astro 5.1+) returns the URL path for an action so you can call it with `fetch()` directly and provide custom headers:

```astro
<script>
  import { actions, getActionPath } from 'astro:actions';

  await fetch(getActionPath(actions.like), {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      Authorization: 'Bearer YOUR_TOKEN',
    },
    body: JSON.stringify({ id: 'YOUR_ID' }),
    keepalive: true,
  });
</script>
```

## ActionReturnType

`ActionReturnType` extracts the output type from an action handler (unwraps `Promise` and `ReturnType`):

```astro
---
import { actions, type ActionReturnType } from 'astro:actions';

type ContactResult = ActionReturnType<typeof actions.contact>;
---
```

## React: withState() and getActionState()

The `@astrojs/react` integration provides `withState()` and `getActionState()` for use with React's `useActionState()` hook.

`withState()` wraps an action so it works as a React form action while preserving progressive enhancement:

```tsx
// Like.tsx
import { actions } from 'astro:actions';
import { withState } from '@astrojs/react/actions';
import { useActionState } from 'react';

export function Like({ postId }: { postId: string }) {
  const [state, action, pending] = useActionState(
    withState(actions.like),
    { data: 0, error: undefined },
  );

  return (
    <form action={action}>
      <input type="hidden" name="postId" value={postId} />
      <button disabled={pending}>{state.data} ❤️</button>
    </form>
  );
}
```

`getActionState()` reads the state stored by `useActionState()` on the server inside your action handler:

```typescript
// src/actions/index.ts
import { defineAction } from 'astro:actions';
import { z } from 'astro/zod';
import { getActionState } from '@astrojs/react/actions';

export const server = {
  like: defineAction({
    input: z.object({ postId: z.string() }),
    handler: async ({ postId }, ctx) => {
      const { data: currentLikes = 0, error } = await getActionState(ctx);
      if (error) throw error;
      return currentLikes + 1;
    },
  }),
};
```

## Security When Using Actions

Actions are accessible as public endpoints based on their name (e.g. `blog.like()` → `/_actions/blog.like`). You **must** apply the same authorization checks you would for API endpoints.

### Authorize from the action handler

Add an authentication check in the handler and raise `ActionError` with `UNAUTHORIZED` when unauthorized:

```typescript
export const server = {
  getUserSettings: defineAction({
    handler: async (_input, context) => {
      if (!context.locals.user) {
        throw new ActionError({ code: 'UNAUTHORIZED' });
      }
      return { /* data on success */ };
    },
  }),
};
```

### Gate actions from middleware with getActionContext()

Use `getActionContext()` in middleware to inspect inbound action requests, including `calledFrom` (`'rpc'` for client-side function calls, `'form'` for HTML form actions):

```typescript
// src/middleware.ts
import { defineMiddleware } from 'astro:middleware';
import { getActionContext } from 'astro:actions';

export const onRequest = defineMiddleware(async (context, next) => {
  const { action } = getActionContext(context);

  // Gate client-side RPC calls behind a session token
  if (action?.calledFrom === 'rpc') {
    if (!context.cookies.has('user-session')) {
      return new Response('Forbidden', { status: 403 });
    }
  }

  context.cookies.set('user-session', 'session-token-value');
  return next();
});
```

`getActionContext()` also returns `setActionResult()` and `serializeActionResult()` to programmatically persist action results (e.g. the POST/Redirect/GET pattern with a session).

## Best Practices

1. **Use Zod for validation** - Type-safe input handling
2. **Throw ActionError for business logic errors** - Proper error codes
3. **Access context for auth** - Use `context.locals` from middleware
4. **Use `accept: 'form'`** - For form submissions without JS
5. **Handle errors gracefully** - Show user-friendly messages
6. **Use getActionResult** - For progressive enhancement
7. **Keep handlers focused** - Single responsibility

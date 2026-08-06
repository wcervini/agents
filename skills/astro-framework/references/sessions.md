# Sessions

Sessions store data on the server between requests for on-demand rendered pages. Unlike cookies, sessions have no size limits and are more secure since data never leaves the server.

**Added in:** Astro 5.7+

## Setup

Sessions require a storage driver. Some adapters (Node, Cloudflare, Netlify) configure a default driver automatically.

```javascript
// astro.config.mjs
import { defineConfig, sessionDrivers } from 'astro/config';
import vercel from '@astrojs/vercel';

export default defineConfig({
  adapter: vercel(),
  session: {
    driver: sessionDrivers.lruCache({ max: 800 }),
    cookie: { name: 'my-session-cookie', sameSite: 'lax', secure: true },
    ttl: 3600, // default expiration for session values, in seconds
  },
});
```

Any [unstorage driver](https://unstorage.unjs.io/drivers) can be used (Redis, filesystem, memory, etc.).

## Configuration Options

| Option | Type | Description |
|--------|------|-------------|
| `session.driver` | `SessionDriverConfig` | Storage driver (use `sessionDrivers.*` helpers from `astro/config`) |
| `session.cookie` | `string \| AstroCookieSetOptions` | Cookie name (string) or cookie options (default: `{ name: "astro-session", sameSite: "lax", httpOnly: true, secure: true }`) |
| `session.ttl` | `number` | Default time-to-live for session values, in seconds (default: Infinity). A per-entry `ttl` passed to `session.set()` overrides it |
| `session: false` | `false` | Disable sessions entirely (Astro 7.2+) — adapter skips its default driver and the session runtime is excluded from the SSR bundle |

### Disabling sessions (Astro 7.2+)

Set `session: false` to opt out of session support entirely. This is useful for serverless/edge runtimes where a smaller bundle reduces cold start time:

```javascript
// astro.config.mjs
export default defineConfig({
  session: false,
});
```

Adapters that normally provide a default session driver (Node, Cloudflare, Netlify) will not configure one, and `Astro.session` / `context.session` will be `undefined`.

### Overriding the configuration at runtime via `entrypoint`

Session drivers are configured at **build time** — environment variables used in the driver config are inlined and cannot be overridden at runtime. When you need runtime configuration (e.g. an external service), define it in a separate file and use it as the driver's `entrypoint`:

```typescript
// src/session-driver.ts
import type { SessionDriver } from 'astro';
import redisDriver from 'unstorage/drivers/redis';
import { REDIS_HOST, REDIS_PORT } from 'astro:env';

export default function (): SessionDriver {
  return redisDriver({
    host: REDIS_HOST,
    port: REDIS_PORT,
  });
}
```

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import vercel from '@astrojs/vercel';

export default defineConfig({
  adapter: vercel(),
  session: {
    driver: {
      entrypoint: new URL('./src/session-driver.ts', import.meta.url),
    },
  },
});
```

## Using Sessions

### In Astro Components and Pages

```astro
---
export const prerender = false; // Required in hybrid mode

const cart = await Astro.session?.get('cart');
---

<a href="/checkout">Cart: {cart?.length ?? 0} items</a>
```

### In API Endpoints

```typescript
// src/pages/api/addToCart.ts
export async function POST(context: APIContext) {
  const cart = await context.session?.get('cart') || [];
  const data = await context.request.json<{ item: string }>();

  if (!data?.item) {
    return new Response('Item is required', { status: 400 });
  }

  cart.push(data.item);
  await context.session?.set('cart', cart);
  return Response.json(cart);
}
```

### In Actions

```typescript
import { defineAction } from 'astro:actions';
import { z } from 'astro/zod';

export const server = {
  addToCart: defineAction({
    input: z.object({ productId: z.string() }),
    handler: async (input, context) => {
      const cart = await context.session?.get('cart') || [];
      cart.push(input.productId);
      await context.session?.set('cart', cart);
      return cart;
    },
  }),
};
```

### In Middleware

```typescript
import { defineMiddleware } from 'astro:middleware';

export const onRequest = defineMiddleware(async (context, next) => {
  context.session?.set('lastVisit', new Date());
  return next();
});
```

**Note:** Sessions are not supported in edge middleware.

## Session API

| Method | Description |
|--------|-------------|
| `session.get(key)` | Get a value by key |
| `session.set(key, value, options?)` | Set a value (optional `{ ttl }` in seconds overrides the global default) |
| `session.regenerate()` | Create a new session ID (use after login) |
| `session.destroy()` | Delete the entire session (use on logout) |

## Type Safety

Define session data types in `src/env.d.ts`:

```typescript
declare namespace App {
  interface SessionData {
    user: {
      id: string;
      name: string;
    };
    cart: string[];
    lastVisit: Date;
  }
}
```

This enables type-checking and autocomplete:

```astro
---
const cart = await Astro.session?.get('cart');
// const cart: string[] | undefined

Astro.session?.set('user', { id: 1, name: 'Houston' });
// Error: id should be string, not number
---
```

## Supported Data Types

Session values are serialized with [devalue](https://github.com/Rich-Harris/devalue). Supported types: strings, numbers, `Date`, `Map`, `Set`, `URL`, arrays, and plain objects.

## Best Practices

1. **Always use optional chaining** (`session?.get()`) — session may be undefined if not configured
2. **Regenerate after login** — prevents session fixation attacks
3. **Destroy on logout** — cleans up server-side data
4. **Keep session data small** — store IDs and references, not large objects
5. **Type your session data** — use `App.SessionData` for safety
6. **Don't rely on sessions for prerendered pages** — sessions only work with on-demand rendering
7. **Set `session.ttl`** for values that should expire, and use per-entry `ttl` in `session.set()` for exceptions
8. **Use a custom `entrypoint`** when you need to override driver config at runtime (env vars are inlined at build time)
9. **Set `session: false`** on serverless/edge runtimes that don't need sessions to shrink the bundle

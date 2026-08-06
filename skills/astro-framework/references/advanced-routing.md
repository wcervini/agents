# Advanced Routing

Astro 7 lets you replace the built-in request pipeline (trailing-slash normalization → redirects → sessions → actions → middleware → pages → i18n → caching) with your own custom pipeline using a special `src/fetch.ts` file.

**Added in:** `astro@7.0.0`

## The fetch entrypoint

Create `src/fetch.ts` that default-exports an object with a `fetch()` method. It receives a standard `Request` and must return a `Response`:

```typescript
// src/fetch.ts
import type { Fetchable } from 'astro';

export default {
  async fetch(request) {
    return new Response('Hello from advanced routing!');
  },
} satisfies Fetchable;
```

Supported formats: `.ts`, `.js`, `.mjs`, `.mts`.

> **Warning:** `src/fetch.ts` is a reserved file name in Astro 7 (like `src/middleware.ts`). If you already use `src/fetch.ts` for other purposes, rename it or configure `fetchFile`.

## fetchFile configuration

Change the entrypoint filename (or disable it) in `astro.config.mjs`:

```javascript
// astro.config.mjs
export default defineConfig({
  fetchFile: 'handler', // looks for src/handler.ts
  // fetchFile: null,   // disables advanced routing entirely
});
```

## Composing handlers with `astro/fetch`

The `astro/fetch` module exports composable handlers that cover the built-in pipeline:

```typescript
import {
  FetchState,
  astro,
  actions,
  cache,
  i18n,
  middleware,
  pages,
  redirects,
  sessions,
  trailingSlash,
} from 'astro/fetch';
```

### FetchState

The per-request state object. Create one at the start of your `fetch` method; all handlers require it as their first argument.

Key properties and methods:

| Member | Type | Description |
|--------|------|-------------|
| `state.request` | `Request` | The incoming request |
| `state.url` | `URL` | Normalized URL derived from the request |
| `state.pathname` | `string` | Base-stripped, decoded pathname (`/about`, `/blog/my-post`) |
| `state.routeData` | `RouteData \| undefined` | Matched route, resolved automatically |
| `state.cookies` | `AstroCookies` | Cookie read/set for this request |
| `state.locals` | `App.Locals` | Request-scoped data (same as middleware `locals`) |
| `state.params` | `Params \| undefined` | Route params from the matched route |
| `state.status` | `number` | Response status (set before rendering, e.g. `state.status = 404`) |
| `state.response` | `Response \| undefined` | Response produced by `pages()`/`middleware()` |
| `state.rewrite(payload)` | `Promise<Response>` | Rewrite to a different pathname, URL, or `Request` |

### The `astro()` handler

The all-in-one handler that runs the full pipeline in the default order. Use it to add logic before/after Astro without changing the internal order:

```typescript
// src/fetch.ts
import { FetchState, astro } from 'astro/fetch';

export default {
  async fetch(request: Request): Promise<Response> {
    const state = new FetchState(request);
    // custom pre-processing here...
    const response = await astro(state);
    // custom post-processing here...
    return response;
  },
};
```

### Composing individual handlers

To control the pipeline order, compose handlers manually. For example, middleware + pages + i18n:

```typescript
// src/fetch.ts
import { FetchState, middleware, pages, i18n } from 'astro/fetch';

export default {
  async fetch(request: Request): Promise<Response> {
    const state = new FetchState(request);
    const response = await middleware(state, (s) => pages(s));
    return i18n(state, response);
  },
};
```

Handle actions before rendering (RPC actions return a `Response`; form actions return `undefined`):

```typescript
// src/fetch.ts
import { actions, FetchState } from 'astro/fetch';

export default {
  async fetch(request: Request): Promise<Response> {
    const state = new FetchState(request);
    const actionResponse = await actions(state);
    if (actionResponse) return actionResponse;
    // otherwise continue to page rendering...
  },
};
```

Register sessions early (before middleware runs). It registers no provider when no session driver is configured or sessions are disabled:

```typescript
// src/fetch.ts
import { FetchState, sessions } from 'astro/fetch';

export default {
  async fetch(request: Request): Promise<Response> {
    const state = new FetchState(request);
    await sessions(state);
    // ...render pipeline...
  },
};
```

Use `state.rewrite()` to redirect the request to a different route:

```typescript
// src/fetch.ts
import { FetchState, middleware, pages } from 'astro/fetch';

export default {
  async fetch(request: Request): Promise<Response> {
    const state = new FetchState(request);
    const response = await state.rewrite('/other-page');
    return response;
  },
};
```

## Hono middleware with `astro/hono`

If you prefer Hono, `astro/hono` exports the same handlers as Hono middleware:

```typescript
// src/fetch.ts
import { Hono } from 'hono';
import { actions, middleware, pages, i18n } from 'astro/hono';

const app = new Hono();

app.use(actions());
app.use(middleware());
app.use(pages());
app.use(i18n());

export default app;
```

The Cloudflare adapter also provides `cf()` companion handlers for `astro/fetch` and `astro/hono` (see the Cloudflare adapter docs) when using a custom worker entrypoint.

## Best Practices

1. **Start from `astro(state)`** when you only need pre/post processing — keeps the default pipeline intact
2. **Compose individual handlers** (`sessions`, `actions`, `middleware`, `pages`, `i18n`) when you need full control over ordering
3. **Check `actions()` return value** — only continue rendering when it returns `undefined`
4. **Register `sessions()` early** — before middleware runs
5. **Set `state.status`** before rendering to control the response status
6. **Rename or disable via `fetchFile`** if `src/fetch.ts` is used for something else
# Route Caching

Astro 7 adds a platform-agnostic route caching API for SSR responses. It provides pluggable **cache providers** that adapters can configure automatically, plus a per-request API to control caching behavior.

**Added in:** `astro@7.0.0`

## Enabling the cache

Enable caching by configuring a `cache.provider` in `astro.config.mjs`. Astro ships a built-in in-memory LRU provider:

```javascript
// astro.config.mjs
import { defineConfig, memoryCache } from 'astro/config';

export default defineConfig({
  cache: {
    provider: memoryCache({ max: 500 }),
  },
});
```

`memoryCache()` options: `max` (max entries, default `1000`) and `query` (`{ sort, include, exclude }`) to control how query parameters affect cache keys. By default it sorts query params and excludes common tracking params (`utm_*`, `fbclid`, `gclid`, etc.).

## Route rules

Set default caching per route pattern with `routeRules`:

```javascript
export default defineConfig({
  cache: {
    provider: memoryCache(),
  },
  routeRules: {
    '/blog/[...path]': { maxAge: 300, swr: 60 },
    '/api/data': { maxAge: 3600, tags: ['api'] },
  },
});
```

## The cache object

The `cache` object is available as `Astro.cache` in `.astro` pages and as `context.cache` in API routes and middleware.

| Property | Type | Description |
|----------|------|-------------|
| `cache.enabled` | `boolean` | Whether caching is active. `false` when no provider is configured or in dev mode |
| `cache.set(options)` | `(CacheOptions \| false) => void` | Set cache options for the current request; pass `false` to opt out |
| `cache.options` | `Readonly<CacheOptions>` | Read-only snapshot of accumulated cache options |
| `cache.tags` | `string[]` | Read-only array of accumulated cache tags |
| `cache.invalidate(options)` | `Promise<void>` | Purge cached entries by tag or path (requires a provider) |

`CacheOptions`: `maxAge` (seconds fresh), `swr` (stale-while-revalidate window), `tags` (targeted invalidation), `lastModified`, `etag`.

### Setting cache options

```astro
---
// src/pages/index.astro
export const prerender = false; // Not needed in 'server' mode
Astro.cache.set({
  maxAge: 120,
  swr: 60,
  tags: ['home'],
});
---
<html><body>Cached page</body></html>
```

In API routes and middleware, use `context.cache`:

```ts
// src/pages/api/data.ts
export function GET(context) {
  context.cache.set({
    maxAge: 300,
    tags: ['api', 'data'],
  });
  return Response.json({ ok: true });
}
```

### Checking if caching is enabled

When no provider is configured, `cache.set()`, `cache.tags`, and `cache.options` log a warning, and `cache.invalidate()` throws. Always guard with `cache.enabled`:

```astro
---
if (Astro.cache.enabled) {
  const tags = await getProductTags(Astro.params.id);
  Astro.cache.set({ maxAge: 3600, tags });
}
---
```

### Opting out of caching

Pass `false` to `cache.set()` to explicitly opt a request out of caching (e.g. when a route rule would otherwise cache it):

```astro
---
if (isPersonalized) {
  Astro.cache.set(false);
}
---
```

### Invalidating cache entries

Purge by tag or by exact path (no glob/wildcard):

```ts
// src/pages/api/revalidate.ts
export async function POST(context) {
  await context.cache.invalidate({ tags: ['data'] });
  await context.cache.invalidate({ path: '/api/data' });
  return Response.json({ purged: true });
}
```

## CDN cache providers (per adapter)

Astro's first-party adapters for Netlify, Vercel, and Cloudflare provide CDN cache providers that map cache directives to the platform's native cache headers and invalidation API. Cache hits are served from the CDN edge without invoking your server function. During the experimental phase these must be enabled manually.

### Netlify

```js
import { defineConfig } from 'astro/config';
import netlify from '@astrojs/netlify';
import { cacheNetlify } from '@astrojs/netlify/cache';

export default defineConfig({
  adapter: netlify(),
  cache: {
    provider: cacheNetlify(),
  },
});
```

Sets `Netlify-CDN-Cache-Control` and `Netlify-Cache-Tag` headers. Uses Netlify's durable cache (shared across edge nodes). Supports tag- and path-based invalidation.

### Vercel

```js
import { defineConfig } from 'astro/config';
import vercel from '@astrojs/vercel';
import { cacheVercel } from '@astrojs/vercel/cache';

export default defineConfig({
  adapter: vercel(),
  cache: {
    provider: cacheVercel(),
  },
});
```

Sets `Vercel-CDN-Cache-Control` and `Vercel-Cache-Tag` headers. Tag invalidation is soft (marks stale, revalidates in background via SWR).

### Cloudflare

```js
import { defineConfig } from 'astro/config';
import cloudflare from '@astrojs/cloudflare';
import { cacheCloudflare } from '@astrojs/cloudflare/cache';

export default defineConfig({
  adapter: cloudflare(),
  cache: {
    provider: cacheCloudflare(),
  },
});
```

Sets `Cloudflare-CDN-Cache-Control` and `Cache-Tag` headers. Enables Cloudflare Workers Cache with default settings.

## Best Practices

1. **Guard all cache calls** with `cache.enabled` — avoids warnings/throws without a provider
2. **Use `routeRules`** for default per-route caching, then override per-request with `cache.set()`
3. **Tag responses** for targeted invalidation instead of clearing the whole cache
4. **Use CDN providers** on Netlify/Vercel/Cloudflare to serve from the edge
5. **Opt out with `cache.set(false)`** for personalized responses that route rules would otherwise cache
6. **Use `swr`** to serve stale content while revalidating in the background